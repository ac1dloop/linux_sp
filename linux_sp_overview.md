## Error handling
defined in <errno.h>
after executing of function if it fails (ex. returns -1) we can directly check **errno** var
#### Errors and their descriptions
* E2BIG Argument list too long
* EACCESS Permission denied
* EAGAIN Try again
* EBADF Bad file number
* EBUSY Device or resource busy
* ECHILD No child processes
* EDOM Math argument outside of domain of function
* EEXIT File already exists
* EFAULT Bad address
* EFBIG File too large
* EINTR System call was interrupted
* EINVAL Invalid argument
* EIO I/O error
* EISDIR Is a directory
* EMFILE Too many open files
* EMLINK Too many links
* ENFILE File table overflow
* ENODEV No such device
* ENOENT No such file or directory
* ENOEXEC Exec format error
* ENOMEM Out of memory
* ENOSPC No space left on device
* ENOTDIR Not a directory
* ENOTTY Inappropriate I/O control operation
* ENXIO No such device or address
* EPERM Operation not permitted
* EPIPE Broken pipe
* ERANGE Result too large
* EROFS Read-only filesystem
* ESPIPE Invalid seek
* ESRCH No such process
* ETXTBSY Text file busy
* EXDEV Improper link

<stdio.h> defines **perror** function that can be used to print text representation of the error occured

## File I/O

### open

Each process has 3 file descriptors opened by default
* 0 (stdin) STDIN_FILENO
* 1 (stdout) STDOUT_FILENO
* 2 (stderr) STDERR_FILENO

**open("path", flags, mode)** syscall returns fd associated with the path provided

 #include <sys/types.h>
 #include <sys/stat.h>
 #include <fcntl.h>

#### Flags for open
* O_APPEND Befor **EACH** write file position is updated to point to the EOF
* O_ASYNC A signal (SIGIO by default) will be generated when the specified file becomes
readable or writable. This flag is available only for terminals and sockets, not for
regular files.
* O_CREAT If the file denoted by name does not exist, the kernel will create it. If the file
already exists, this flag has no effect unless O_EXCL is also given.
* O_DIRECT The file will be opened for direct I/O
* O_DIRECTORY If name is not a directory, the call to open( ) will fail. This flag is used internally
by the opendir() library call.
* O_EXCL When given with O_CREAT, this flag will cause the call to open( ) to fail if the file
given by name already exists. This is used to prevent race conditions on file
creation.
* O_LARGEFILE The given file will be opened using 64-bit offsets, allowing files larger than two
gigabytes to be opened. This is implied on 64-bit architectures.
* O_NOCTTY If the given name refers to a terminal device (say, /dev/tty ), it will not become
the process' controlling terminal, even if the process does not currently have a
controlling terminal. This flag is not frequently used.
* O_NOFOLLOW If name is a symbolic link, the call to open( ) will fail.
* O_NONBLOCK If possible, the file will be opened in nonblocking mode.
* O_SYNC The file will be opened for synchronous I/O.
* O_TRUNC If the file exists, it is a regular file, and the given flags allow for writing, the file
will be truncated to zero length. Use of O_TRUNC on a FIFO or terminal device is
ignored. Use on other file types is undefined. Specifying O_TRUNC with O_RDONLY is
also undefined, as you need write access to the file in order to truncate it.

#### Possible modes
* S_IRWXU Owner has read, write, and execute permission.
* S_IRUSR Owner has read permission.
* S_IWUSR Owner has write permission.
* S_IXUSR Owner has execute permission.
* S_IRWXG Group has read, write, and execute permission.
* S_IRGRP Group has read permission.
* S_IWGRP Group has write permission.
* S_IXGRP Group has execute permission.
* S_IRWXO Everyone else has read, write, and execute permission.
* S_IROTH Everyone else has read permission.
* S_IWOTH Everyone else has write permission.
* S_IXOTH Everyone else has execute permission.

### read

**ssize_t read(int fd, void *buf, size_t len)**

**reading all bytes for sure example**

```c
ssize_t ret;
while (len != 0 && (ret = read (fd, buf, len)) != 0) {
if (ret == -1) {
if (errno == EINTR)
continue;
perror ("read");
break;
}
len -= ret;
buf += ret;
}
```
possible error codes returned by read
* EBADF The given file descriptor is invalid, or not open for reading.
* EFAULT The pointer provided by buf is not inside the calling process' address space.
* EINVAL The file descriptor is mapped to an object that does not allow reading.
* EIO A low-level I/O error occurred.

### write

**ssize_t write (int fd, const void *buf, size_t count)**


example
```c
unsigned long word = 1720;
size_t count;
ssize_t nr;
count = sizeof (word);
nr = write (fd, &word, count);
if (nr == -1)
/* error, check errno */
else if (nr != count)
/* possible error, but 'errno' not set */
```
possible error codes worth checking
* EBADF The given file descriptor is not valid, or is not open for writing.
* EFAULT The pointer provided by buf points outside of the process' address space.
* EFBIG The write would have made the file larger than per-process maximum file limits,
or internal implementation limits.
* EINVAL The given file descriptor is mapped to an object that is not suitable for writing.
* EIO A low-level I/O error occurred.
* ENOSPC The filesystem backing the given file descriptor does not have sufficient space.
* EPIPE The given file descriptor is associated with a pipe or socket whose reading end is
closed.

it is possible to explicitly synch data on disk

**int fsync (int fd)**
A call to fsync( ) ensures that all dirty data associated with the file mapped by the file
descriptor fd is written back to disk.
**int fdatasync (int fd)**
This system call does the same thing as fsync( ), except that it only flushes data. The call
does not guarantee that metadata is synchronized to disk, and is therefore potentially faster.
Often this is sufficient.
possible error codes
* EBADF
* EINVAL
* EIO
**void sync (void)**
sync( ) system call is provided for
synchronizing all buffers to disk
also file can be opened with **O_RSYNC O_DSYNC O_SYNC** flags

Providing the O_DIRECT flag to open( ) instructs the kernel to minimize the presence of I/O
management.
When performing direct I/O, the request length, buffer alignment, and file offsets must all be
integer multiples of the underlying device's sector size—generally, this is 512 bytes
### close

**int close (int fd)**

### seek

**off_t lseek (int fd, off_t pos, int origin)**

origin arguements
* SEEK_CUR The current file position of fd is set to its current value plus pos
* SEEK_END The current file position of fd is set to the current length of the file plus pos
* SEEK_SET The current file position of fd is set to pos. A pos of zero sets the offset to the
beginning of the file.
possible errors
* EBADF
* EINVAL The value given for origin is not one of SEEK_SET, SEEK_CUR, or SEEK_END
* EOVERFLOW The resulting file offset cannot be represented in an off_t. This can occur only on
32-bit architectures. Currently, the file position is updated; this error indicates
just that it is impossible to return it.
* ESPIPE The given file descriptor is associated with an unseekable object, such as a pipe,
FIFO, or socket.

### positional reads and writes

**ssize_t pread (int fd, void *buf, size_t count, off_t pos)**
**ssize_t pwrite (int fd, const void *buf, size_t count, off_t pos)**

upon returning file position is not changed

### truncating

**int ftruncate (int fd, off_t len)**
**int truncate (const char *path, off_t len)**

#### all of the multiplexing calls are stupid except epoll() but must be reviewed

**select( )**
**pselect( )**
**poll( )**
**ppoll( )**

## Buffered I/O

### opening files

**FILE * fopen (const char *path, const char *mode)**
**FILE * fdopen (int fd, const char *mode)**
modes
* r
* r+ Open the file for both reading and writing.
* w Open the file for writing. If the file exists, it is truncated to zero length. If the file
does not exist, it is created.
* w+ Open the file for both reading and writing. If the file exists, it is truncated to zero
length. If the file does not exist, it is created.
* a Open the file for writing in append mode. The file is created if it does not exist.
* a+ Open the file for both reading and writing in append mode. The file is created if it
does not exist.
### closing
**int fclose (FILE *stream)**
**int fcloseall (void)**
### reading
**int fgetc (FILE *stream)** reads one char
Standard I/O provides a function for pushing a character back onto a stream, allowing you to
"peek" at the stream, and return the character if it turns out that you don't want it
**int ungetc (int c, FILE *stream)**
**char * fgets (char *str, int size, FILE *stream)**

**size_t fread (void *buf, size_t size, size_t nr, FILE *stream)**
The number of elements(nr, of size 'size') read (not the number of bytes read!) is returned.
###writing
**int fputc (int c, FILE *stream)**
**int fputs (const char *str, FILE *stream)**
**size_t fwrite (void *buf, size_t size, size_t nr, FILE *stream)**

### seeking
**int fseek (FILE *stream, long offset, int whence)**
If whence is set to SEEK_SET, the file position is set to offset. If whence is set to SEEK_CUR,
the file position is set to the current position plus offset. If whence is set to SEEK_END, the file
position is set to the end of the file plus offset.
**int fsetpos (FILE *stream, fpos_t *pos)** whence is SEEK_SET
**void rewind (FILE *stream)** resets position to start of stream
**long ftell (FILE *stream)** returns current position
**int fgetpos (FILE *stream, fpos_t *pos)** returns current position
**int fflush (FILE *stream)** on call data is flushed to the kernel
fflush(NULL) flushes ALL streams
### error checking
the function ferror( ) tests whether the error indicator
is set on stream
**int ferror (FILE *stream)**
**int feof (FILE *stream)** tests whether the EOF indicator is set on stream
**void clearerr (FILE *stream)** clears errors and EOF indicators
**int fileno (FILE *stream)** returns file descriptor

#### Unbuffered _IONBF
No user buffering is performed. Data is submitted directly to the kernel. As this is
the antithesis of user buffering, this option is not commonly used. Standard error,
by default, is unbuffered.
### Line-buffered _IOLBF
Buffering is performed on a per-line basis. With each newline character, the buffer
is submitted to the kernel. Line buffering makes sense for streams being output to
the screen. Consequently, this is the default buffering used for terminals
(standard out is line-buffered by default).
### Block-buffered _IOFBF
Buffering is performed on a per-block basis. This is the type of buffering discussed
at the beginning of this chapter, and it is ideal for files. By default, all streams
associated with files are block-buffered. Standard I/O uses the term full buffering
for block buffering.
**int setvbuf (FILE *stream, char *buf, int mode, size_t size)**
### manual locking
**void flockfile (FILE *stream)** and non-blocking version **int ftrylockfile (FILE *stream)**
The function flockfile( ) waits until stream is no longer locked, and then acquires the lock,
bumps the lock count, becomes the owning thread of the stream, and returns
**void funlockfile (FILE *stream)**
## Adnvanced file I/O
### Epoll
Improves on the poll( ) and select( ) system calls described in Chapter 2 ;
useful when hundreds of file descriptors have to be polled in a single program.
### Memory-mapped I/O
Maps a file into memory, allowing file I/O to occur via simple memory
manipulation; useful for certain patterns of I/O.
### File advice
Allows a process to provide hints to the kernel on its usage scenarios; can result
in improved I/O performance.
### Asynchronous I/O
Allows a process to issue I/O requests without waiting for them to complete;
useful for juggling heavy I/O workloads without the use of threads.
