# Chapter 16. Accessing Files
+ There are many different ways to access a file. In this chapter we will consider the following cases:

| type                | explains                                                                |
| ------------------- | ----------------------------------------------------------------------- |
| Canonical mode      | accessed by means of the read( ) and write( ) system calls.             |
| Synchronous mode    | opened with the O_SYNC flag or set by the fcntl( ) system call.         |
| Memory mapping mode | issues an mmap( ) system call to map the file into memory               |
| Direct I/O mode     | transfers data directly from the User Mode address space to disk        |
| Asynchronous mode   | through a group of POSIX APIs or by means of Linux-specific system call |


## 16.1. Reading and Writing a File
+ For most filesystems, reading a page of data from a file is just a matter of finding what blocks on disk contain the requested data.---硬盘上那个块包含请求的数据。
+ Write operations on disk-based files are slightly more complicated to handle, because the file size could increase, and therefore the kernel might allocate some physical blocks on the disk.---写操作稍微复杂一点，因为文件可能增大。

### 16.1.1. Reading from a File
+ generic_file_read( ) function

#### 16.1.1.1. The readpage method for regular files
+ The readpage method of the address_space object stores the address of the function that effectively activates the I/O data transfer from the physical disk to the page cache.---保存激活I/O数据传输函数的地址。

#### 16.1.1.2. The readpage method for block device files
+ implemented by the blkdev_readpage( ) function
+ the second parameter points to a function that translates the file block number relative to the beginning of the file into a logical block number relative to the beginning of the block device.---将文件块号转化为逻辑块号。

### 16.1.2. Read-Ahead of Files
+ Read-ahead consists of reading several adjacent pages of data of a regular file or block device file before they are actually requested.---在实际的请求来临前已经预读。
+ read-ahead is of no use when an application performs random accesses to files;---当应用程序随机访问文件时，预读将不适用。
+ Read-ahead of files requires a sophisticated algorithm for several reasons:
+ The kernel considers a file access as sequential with respect to the previous file access if the first page requested is the page following the last page requested in the previous access.
---如果请求的第一个页面是在先前访问中请求的最后一页之后的页面，则内核将文件访问视为与先前文件访问相关的顺序。
+ the read-ahead algorithm makes use of two sets of pages:
  + current window
  + ahead window
+ main data structure used by the read-ahead algorithm is the *file_ra_state* descriptor

+ When is the read-ahead algorithm executed? This happens in the following cases:

#### 16.1.2.1. The page_cache_readahead( ) function
+ takes care of all read-ahead operations that are not explicitly triggered by ad-hoc system calls.

#### 16.1.2.2. The handle_ra_miss( ) function
+ In some cases, the kernel must correct the read-ahead parameters, because the read-ahead strategy does not seem very effective. ---纠正预读参数（预读策略效率较低时）

### 16.1.3. Writing to a File
+ Many filesystems (including Ext2 or JFS ) implement the write method of the file object by means of the *generic_file_write( )* function

#### 16.1.3.1. The prepare_write and commit_write methods for regular files
+ The prepare_write and commit_write methods of the address_space object specialize the generic write operation implemented by generic_file_write( ) for regular files and block device files.
+ The block_prepare_write( ) function takes care of preparing the buffers and the buffer heads of the file's page

#### 16.1.3.2. The prepare_write and commit_write methods for block device files

### 16.1.4. Writing Dirty Pages to Disk
+ Many non-journaling filesystems rely on the mpage_writepage( ) function rather than on the custom writepage method. This can improve performance because the mpage_writepage( ) function tries to submit the I/O transfers by collecting as many pages as possible in the same bio descriptor; in turn, this allows the block device drivers to exploit the *scatter-gather DMA* capabilities of the modern hard disk controllers.

## 16.2. Memory Mapping
+ an access to a byte within a page of the memory region is translated by the kernel into an operation on the corresponding byte of the file. --- 内核将对存储器区域的页面内的字节的访问转换为对文件的相应字节的操作。

+ two kinds of mem mapping:
+ shared --- changes are visible to all other processes that mapping the same file
+ private --- to be used when process create the mapping just to read the file, not to write it.

### 16.2.1. Memory Mapping Data Structures
![](ds_mem_map.jpg)
+ The core of memory mapping implementation is delegated to a file object's method named *mmap*.

### 16.2.2. Creating a Memory Mapping
+ To create a new memory mapping, a process issues an mmap( ) system call
+ The mmap( ) system call returns the linear address of the first location in the new memory region.

### 16.2.3. Destroying a Memory Mapping
+ there is no need to flush to disk the contents of the pages included in a writable shared memory

### 16.2.4. Demand Paging for Memory Mapping
+ page frames are not assigned to a memory mapping right after it has been created, but at the *last possible moment*

### 16.2.5. Flushing Dirty Memory Mapping Pages to Disk
+ The *msync( )* system call can be used by a process to flush to disk dirty pages belonging to a shared memory mapping.

### 16.2.6. Non-Linear Memory Mappings
+ each memory page maps a random (arbitrary) page of file's data.
