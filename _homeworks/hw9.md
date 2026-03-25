---
title: 'Homework 9'
layout: homework
week: 12
released: 2026-03-23
due: 2026-04-03
summary: 'File System. '
---



# CS5600 File System (fs5600)

These questions are intended to reinforce fs5600.

## Question 1 (6 pts)

Suppose fs5600 has a file `/dir1/dir2/file1`. When a process read the file's information, namely executing:

```c
 struct stat;
 fs_getattr("/dir1/dir2/file1", &stat)
```

* How many block reads to disk (`block_read`) does fs5600 perform?
* And what are they (what info stored in these blocks)? (assume buffer cache is empty, meaning you have to read information from disk)


## Question 2 (6 pts)

Suppose fs5600 has a file `/file1` and the file has 16KB of contents. When a process read 4KB from offset 2KB, namely executing:

```c
char buf[4096];
fs_read("/file1", buf, 4096, 2048, NULL);
```

* How many block reads ("block_read") does fs5600 perform?
* And what are they (what info stored in these blocks)? (assume buffer-cache is empty, meaning you have to read information from disk)


## Question 3 

Suppose a new file system fs5600+ has an inode as follows:

```c
struct fs_inode2 {
  uint16_t uid;
  uint16_t gid;
  uint32_t mode;
  uint32_t ctime;     /* time of last file status change */
  uint32_t mtime;     /* time of last data modification */
  int32_t  size;
  uint32_t ptrs[1018];
  uint32_t indirect_ptr; /* inode = 4096 bytes */
};
```

And the `indirect_ptr` is the indirect pointer in a Unix inode.
It points to a block (4KB) that contains pointers (`uint32_t`) to actual data blocks.


### Q3A (2 pts)
How many pointers (uint32_t) can one block store? 


### Q3B (2 pts)
What is the max number of data block pointers in an fs5600+ inode?  [note: this number includes both pointers in `fs_inode2` and the ones in the indirect block (the block that indirect pointer--`indirect_ptr`---points to).]

### Q3C (2 pts)

What's the maximum file size of an fs5600+ file? 



