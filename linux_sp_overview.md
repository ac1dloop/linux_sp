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

 #include <sys/types.h> <br>
 #include <sys/stat.h> <br>
 #include <fcntl.h> <br>

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

**ssize_t read(int fd, void &ast;buf, size_t len)**

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

**ssize_t write (int fd, const void &ast;buf, size_t count)**


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

**ssize_t pread (int fd, void &ast;buf, size_t count, off_t pos)**
**ssize_t pwrite (int fd, const void &ast;buf, size_t count, off_t pos)**

upon returning file position is not changed

### truncating

**int ftruncate (int fd, off_t len)** <br>
**int truncate (const char &ast;path, off_t len)**

#### all of the multiplexing calls are stupid except epoll() but must be reviewed

**select( )** <br>
**pselect( )** <br>
**poll( )** <br>
**ppoll( )** <br>

## Buffered I/O

### opening files

**FILE &ast; fopen (const char *path, const char &ast;mode)**
**FILE &ast; fdopen (int fd, const char &ast;mode)**
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
**int fclose (FILE &ast;stream)** <br>
**int fcloseall (void)**
### reading
**int fgetc (FILE &ast;stream)** reads one char
Standard I/O provides a function for pushing a character back onto a stream, allowing you to
"peek" at the stream, and return the character if it turns out that you don't want it
**int ungetc (int c, FILE &ast;stream)**
**char * fgets (char &ast;str, int size, FILE &ast;stream)**

**size_t fread (void &ast;buf, size_t size, size_t nr, FILE &ast;stream)**
The number of elements(nr, of size 'size') read (not the number of bytes read!) is returned. <br>
### writing
**int fputc (int c, FILE &ast;stream)**
**int fputs (const char &ast;str, FILE &ast;stream)**
**size_t fwrite (void &ast;buf, size_t size, size_t nr, FILE &ast;stream)**

### seeking
**int fseek (FILE &ast;stream, long offset, int whence)**
If whence is set to SEEK_SET, the file position is set to offset. If whence is set to SEEK_CUR,
the file position is set to the current position plus offset. If whence is set to SEEK_END, the file
position is set to the end of the file plus offset.
**int fsetpos (FILE &ast;stream, fpos_t &ast;pos)** whence is SEEK_SET
**void rewind (FILE &ast;stream)** resets position to start of stream
**long ftell (FILE &ast;stream)** returns current position
**int fgetpos (FILE &ast;stream, fpos_t &ast;pos)** returns current position
**int fflush (FILE &ast;stream)** on call data is flushed to the kernel
fflush(NULL) flushes ALL streams
### error checking
the function ferror( ) tests whether the error indicator
is set on stream
**int ferror (FILE &ast;stream)**
**int feof (FILE &ast;stream)** tests whether the EOF indicator is set on stream
**void clearerr (FILE &ast;stream)** clears errors and EOF indicators
**int fileno (FILE &ast;stream)** returns file descriptor

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
**int setvbuf (FILE &ast;stream, char &ast;buf, int mode, size_t size)**
### manual locking
**void flockfile (FILE &ast;stream)** and non-blocking version **int ftrylockfile (FILE &ast;stream)**
The function flockfile( ) waits until stream is no longer locked, and then acquires the lock,
bumps the lock count, becomes the owning thread of the stream, and returns
**void funlockfile (FILE &ast;stream)**
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
### Scatter/Gather I/O
 #include <sys/uio.h>
**ssize_t readv (int fd,const struct iovec &ast;iov,int count);**
**ssize_t writev (int fd,const struct iovec &ast;iov,int count);**
```C
struct iovec {
void *iov_base;
size_t iov_len;
};
```
# DONT FORGET TO FILL IN THE GAP HERE BETWEEN CHAPTERS

## Memory Management

**int posix_memalign (void &ast;&ast;memptr,size_t alignment,size_t size);** <br>

A successful call to posix_memalign() allocates size bytes of dynamic memory, en‐
suring it is aligned along a memory address that is a multiple of alignment . The pa‐
rameter alignment must be a power of 2 and a multiple of the size of a void pointer.
The address of the allocated memory is placed in memptr , and the call returns 0 .

**void &ast; valloc (size_t size);**

valloc is malloc with alignment to page boundaries

**void &ast; memalign (size_t boundary, size_t size);**

### Anonymous memory mapping

 #include <sys/mman.h>

**void &ast; mmap (void &ast;start,size_t length,int prot,int flags,int fd,off_t offset);**
**int munmap (void &ast;start, size_t length);**

Memory obtained via an anonymous mapping looks the same as memory obtained via
the heap. One benefit to allocating from anonymous mappings is that the pages are
already filled with zeros.

### Tuning glibc memory management

 #include <malloc.h>

**int mallopt (int param, int value);**

* M_CHECK_ACTION The value of the MALLOC_CHECK_ environment variable (discussed in the next
section).
* M_MMAP_MAX The maximum number of mappings that the system will create to satisfy dynamic
memory requests. When this limit is reached, the data segment will be used for all allocations until one of the previously created mappings is freed. A value of 0 disables all use of anonymous mappings as a basis for dynamic memory allocations.
* M_MMAP_THRESHOLD The threshold (measured in bytes) over which an allocation request will be satisfied
via an anonymous mapping instead of the data segment. Note that allocations
smaller than this threshold may also be satisfied via anonymous mappings at the
system’s discretion. A value of 0 enables the use of anonymous mappings for all
allocations, effectively disabling use of the data segment for dynamic memory
allocations.
* M_MXFAST The maximum size (in bytes) of a fast bin. Fast bins are special chunks of memory
in the heap that are never coalesced with adjacent chunks and never returned to
the system, allowing for very quick allocations at the cost of increased fragmenta‐
tion. A value of 0 disables all use of fast bins.
* M_PERTURB Enables memory poisoning, which aids in the detection of memory management
errors. If provided a nonzero value , glibc sets all allocated bytes (except those re‐
quested via calloc() ) to the logical compliment of the least-significant byte in
value . This helps detect use-before-initialized errors. Moreover, glibc also sets all
freed bytes to the least-significant byte in value . This helps detect use-after-free
errors.
* M_TOP_PAD The amount of padding (in bytes) used when adjusting the size of the data segment.
Whenever glibc uses brk() to increase the size of the data segment, it can ask for
more memory than needed in the hopes of alleviating the need for an additional
brk() call in the near future. Likewise, whenever glibc shrinks the size of the data
segment, it can keep extra memory, giving back a little less than it would otherwise.
These extra bytes are the padding. A value of 0 disables all use of padding.
* M_TRIM_THRESHOLD The minimum amount of free memory (in bytes) at the top of the data segment
before glibc invokes sbrk() to return memory to the kernel.

**size_t malloc_usable_size (void &ast;ptr);**

A successful call to malloc_usable_size() returns the actual allocation size of the
chunk of memory pointed to by ptr . Because glibc may round up allocations to fit within
an existing chunk or anonymous mapping, the usable space in an allocation can be larger
than requested.

**int malloc_trim (size_t padding);**

force glibc to return all immediately
freeable memory to the kernel.

To enable memory debugging for program it can be executed like **MALLOC_CHECK_=1 ./app**

If MALLOC_CHECK_ is set to 0 , the memory subsystem silently ignores any errors. If it is
set to 1 , an informative message is printed to stderr . If it is set to 2 , the program is
immediately terminated via abort() . Because MALLOC_CHECK_ changes the behavior of
the running program, setuid programs ignore this variable.

### Obtaining statistics

**struct mallinfo mallinfo (void);**

A call to mallinfo() returns statistics in a mallinfo structure. The structure is returned
by value, not via a pointer.

```C
struct mallinfo {
	int arena; *size of data segment used by malloc*
	int ordblks; *number of free chunks*
	int smblks; *number of fast bins*
	int hblks; *number of anonymous mappings*
	int hblkhd; *size of anonymous mappings*
	int usmblks; *maximum total allocated size*
	int fsmblks; *size of available fast bins*
	int uordblks; *size of total allocated space*
	int fordblks; *size of available chunks*
	int keepcost; *size of trimmable space*
};
```

### Dynamic memory allocation from stack

**void &ast; alloca (size_t size);**

On success, a call to alloca() returns a pointer to size bytes of memory.

**char &ast; strdupa (const char &ast;s);**
**char &ast; strndupa (const char &ast;s, size_t n);**

A call to strdupa() returns a duplicate of s . A call to strndupa() duplicates up to n
characters of s . If s is longer than n , the duplication stops at n , and the function appends
a null byte.

### Setting bytes

 #include <string.h>

**void &ast; memset (void &ast;s, int c, size_t n);**

**int memcmp (const void &ast;s1, const void &ast;s2, size_t n);**

Similar to strcmp() , memcmp() compares two chunks of memory for equivalence:
It is not sage to use memcmp for structures because of possible paddings filled with garbage

**void &ast; memmove (void &ast;dst, const void &ast;src, size_t n);**

memmove() copies the first n bytes of src to dst , returning dst

**void &ast; memcpy (void &ast;dst, const void &ast;src, size_t n);**

This function behaves identically to memmove() , except dst and src may not overlap. If
they do, the results are undefined.

**void &ast; memchr (const void &ast;s, int c, size_t n);**

The functions memchr() and memrchr() locate a given byte in a block of memory

**void &ast; memmem (const void &ast;haystack,size_t haystacklen,const void &ast;needle,size_t needlelen);**

memmem() searches a block of memory for an arbitrary array of bytes

### Locking part of an address space

POSIX 1003.1b-1993 defines two interfaces for “locking” one or more pages into phys‐
ical memory, ensuring that they are never paged out to disk. The first locks a given
interval of addresses

 #include <sys/mman.h>

**int mlock (const void &ast;addr, size_t len);**

A call to mlock() locks the virtual memory starting at addr and extending for len bytes
into physical memory. On success, the call returns 0 ; on failure, the call returns −1 and
sets errno as appropriate.

**int mlockall (int flags);**

A call to mlockall() locks all of the pages in the current process’s address space into
physical memory.

#### Flags

* MCL_CURRENT If set, this value instructs mlockall() to lock all currently mapped pages—the stack,
data segment, mapped files, and so on—into the process’s address space.
* MCL_FUTURE If set, this value instructs mlockall() to ensure that all pages mapped into the
address space in the future are also locked into memory.

To unlock pages from physical memory, again allowing the kernel to swap the pages out
to disk as needed, POSIX standardizes two more interfaces

**int munlock (const void &ast;addr, size_t len);**
**int munlockall (void);**


