# Chapter 16. Accessing Files
+ There are many different ways to access a file. In this chapter we will consider the following cases:

| type                | explains                                                                |
| ------------------- | ----------------------------------------------------------------------- |
| Canonical mode      | accessed by means of the read( ) and write( ) system calls.             |
| Synchronous mode    | opened with the O_SYNC flag or set by the fcntl( ) system call.         |
| Memory mapping mode | issues an mmap( ) system call to map the file into memory               |
| Direct I/O mode     | transfers data directly from the User Mode address space to disk        |
| Asynchronous mode   | through a group of POSIX APIs or by means of Linux-specific system call |
