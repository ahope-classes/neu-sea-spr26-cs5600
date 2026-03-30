---
title: 'Homework 9'
layout: homework
week: 12
released: 2026-03-23
due: 2026-04-03
summary: 'File System. '
---



# File Systems 

These questions should be answered based on the File System as described in Lab 5, and consistent with the FS we discussed in class on Mar 25, 2026. 

## Question 1 (7 pts)

Suppose FS has a file `/dir1/dir2/file1.txt`. Then, we have a process that read the file's information, namely executing:

```c
 struct stat;
 fs_getattr("/dir1/dir2/file1.txt", &stat)
```

Answer the following questions: 

* How many reads happen when the FS executes this command? List out what is read during the execution of this command. (Assume any cache is empty). 



## Question 2 (7 pts)

Suppose our FS has a file `/file1.txt` and the file has 16KB of contents. When a process read 4KB from offset 2KB, namely executing:

```c
char buf[4096];
fs_read("/file1", buf, 4096, 2048, NULL);
```

* How many reads happen when the FS executes this command? List out what is read, and assume any cache is empty. 


## Question 3 

Suppose a new file system FS++ that has an `inode` defined as follows:

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

In the above, the `indirect_ptr` is an indirect pointer as we see in an Unix inode. It points to a block (4KB) that contains pointers (`uint32_t`) to actual data blocks.


### Q3A (2 pts)
How many pointers (`uint32_t`) can one block store? 


### Q3B (2 pts)
What is the max number of data block pointers in an FS++ `inode`?  (This includes pointers in `fs_inode2` as well as the ones in the indirect block). 

### Q3C (2 pts)

What's the maximum file size of an FS++ file? 



