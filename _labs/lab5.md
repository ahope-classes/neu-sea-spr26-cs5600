---
title: 'Lab 5: File System'
layout: lab
released: 2026-03-16
due: 2026-04-10
index: 1
summary: "Implement pieces of a simple disk-based file system. Ties back to Lab 2 where you implemented `ls`!"
---


# Lab 5: XXX 
{:.no_toc} 

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# Introduction

In this lab, you will implement (pieces of) a simple disk-based file system. There is not a lot of code to write; instead, a lot of the work of the lab is understanding the system that you’ve been given.

By the end of the lab, you will be able to run your lab 2 ls against this lab’s file system.

## Getting Started
You’ll be working in the Docker container as usual. We assume that you have set up the upstream as described in the lab setup. Then run the following on your local machine from outside of Docker:

```sh
$ cd ~/cs5600
$ git fetch upstream
$ git merge upstream/main
```

This lab’s files are located in the `lab5` subdirectory.

If you have any conflicts from lab 4, resolve them before continuing. Run git push to save your work back to your personal repository.

The rest of these instructions presume that you are in the Docker environment. We omit the cs5600-user@172b6e333e91:~/cs5600-labs part of the prompt.

# FUSE
The file system that we will build is implemented as a user-level process. This file system's storage will be a file (in the example given below, we call it `testfs.img`) that lives in the normal file system of your Docker container. Much of your code will treat this file as if it were a disk.

This entire arrangement (file system implemented in user space with arbitrary choice of storage) is due to software called FUSE ([Filesystem in Userspace](http://fuse.sourceforge.net/)). In order to really understand what FUSE is doing, we need to take a brief detour to describe VFS. Linux (like Unix) has a layer of kernel software called VFS; conceptually, every file system sits below this layer, and exports a uniform interface to VFS. (You can think of any potential file system as being a "driver" for VFS; VFS asks the software below it to do things like "read", "write", etc.; that software fulfills these requests by interacting with a disk driver, and interpreting the contents of disk blocks.) The purpose of this architecture is to make it relatively easy to plug a new file system into the kernel: the file system writer simply implements the interface that VFS is expecting from it, and the rest of the OS uses the interface that VFS exports to it. In this way, we obtain the usual benefits of pluggability and modularity.

FUSE is just another "VFS driver", but it comes with a twist. Instead of FUSE implementing a disk-based file system (the usual picture), it responds to VFS's requests by asking a user-level process (which is called a "FUSE driver") to respond to it. So the FUSE kernel module is an adapter that speaks "fuse" to a user-level process (and you will be writing your code in this user-level process) and "VFS" to the rest of the kernel.

Meanwhile, a FUSE driver can use whatever implementation it wants. It could store its data in memory, across the network, on Jupiter, whatever. In the setup in this lab, the FUSE driver will interact with a traditional Linux file (as noted above), and pretend that this file is a sector-addressable disk.

The FUSE driver registers a set of callbacks with the FUSE system (via `libfuse` and ultimately the FUSE kernel module); these callbacks are things like read, write, etc. A FUSE driver is associated with a particular directory, or mount point. The concept of mounting was explained in [OSTEP 39](http://pages.cs.wisc.edu/~remzi/OSTEP/file-intro.pdf) (see 39.17). Any I/O operations requested on files and directories under this mount point are dispatched by the kernel (via VFS, the FUSE kernel module, and `libfuse`) to the callbacks registered by the FUSE driver.

<img src="../../assets/images/labs/lab5/lab5-fuse-diagram.svg" alt="disk" width="800"/>

To recap all of the above: the file system user interacts with the file system roughly in this fashion:

* When the file system user, Process A, makes a request to the system, such as listing all files in a directory via `ls`, the `ls` process issues one or more system calls (`stat()`, `read()`, etc.).
* The kernel hands the system call to VFS.
* VFS finds that the system call is referencing a file or directory that is managed by FUSE.
* VFS then dispatches the request to FUSE, which dispatches it to the corresponding FUSE driver (which is where you will write your code).
* The FUSE driver handles the request by interacting with the "disk", which is implemented as an ordinary file. The FUSE driver then responds, and the responses go back through the chain.

Here's an example from the staff solution to show what this looks like, where `testfs.img` is a disk image with only the root directory and the file `hello` on its file system:

```sh
# Create a directory to serve as a mount point. 
# Note: the / is important, because the directory
# should live only in docker's filesystem
$ mkdir /lab5mnt

# create simlink to local directory mnt
$ ln -s /lab5mnt mnt

# see what file system mnt is associated with
$ df mnt    
Filesystem     1K-blocks    Used Available Use% Mounted on
overlay         61202244 8831452  49229468  16% /

# notice, 'mnt' is empty
$ ls mnt    

# mount testfs.img at mnt:
$ build/fsdriver testfs.img mnt 

# below, note that mnt's file system is now different
$ df mnt 
Filesystem         1K-blocks  Used Available Use% Mounted on
CS5600fs#testfs.img      8192    24      8168   1% /lab5mnt

# and there's the hello file...
$ ls mnt 
hello

# ...which we can read with any program
$ cat mnt/hello 
Hello, world!

# now unmount mnt
$ fusermount -u mnt 

# and its associated file system is back to normal
$ df mnt 
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda1        7092728 4536616   2172780  68% /

# and hello is gone, but still lives in testfs.img
$ ls mnt 
```

Note that in the above example, after we run fsdriver, the kernel is actually dispatching the all the `open()`, `read()`, `readdir()`, etc. calls that `ls` and `cat` make to our FUSE driver. The FUSE driver takes care of searching for a file when `open()` is called, reading file data when `read()` is called, and so on. When `fusermount` is run, our file system is unmounted from `mnt`, and then all I/O operations under `mnt` return to being serviced normally by the kernel.

# Our File System
Below, we give an overview of the features that our file system will support; along the way, we review some of the file system concepts that we have studied in class and the reading.

## On-Disk File System Structure
Most UNIX file systems divide available disk space into two main types of regions: *inode* regions and *data* regions. UNIX file systems assign one inode to each file in the file system; a file's inode holds a file's meta-data (pointers to data blocks, etc.). The data regions are divided into much larger (typically 4KB or more) data blocks, within which the file system stores file data and directory data. Directory entries (the "data" in a directory) contain file names and inode numbers; a file is said to be hard-linked if multiple directory entries in the file system refer to that file's inode. Both files and directories logically consist of a series of data blocks; these blocks can be scattered throughout the disk much as the pages of a process's virtual address space can be scattered throughout physical memory.

Unlike most UNIX file systems, we make a simplification in the layout of the file system: there is only one region on the disk, in which both inode blocks and data blocks reside. Furthermore, each inode is allocated its own disk block instead of being packed alongside other inodes in a single disk block.

### Sectors and Blocks
Disk perform reads and writes in units of sectors, which are typically 512 bytes. However, file systems allocate and use disk storage in units of blocks (for example, 4KB, or 8 sectors). Notice the distinction between the two terms: sector size is a property of the disk hardware, whereas block size is a creation of the file system that uses the disk. A file system's block size must be a multiple of the sector size of the underlying disk. As explained in class, there are advantages to making the block size larger than the sector size.

Our file system will use a block size of 4096 bytes.

### Superblocks
File systems typically place important meta-data at reserved, well-known disk blocks (such as the very start of the disk). This meta-data describes properties of the entire file system (block size, disk size, meta-data required to find the root directory, the time the file system was last mounted, the time the file system was last checked for errors, and so on). These special blocks are called superblocks. Many "real" file systems maintain multiple replicas of superblocks, placing them far apart on the disk; that way, if one of them is corrupted or the disk develops a media error in that region, the other replicas remain accessible.

<img src="../../assets/images/labs/lab5/disk.svg" alt="disk" width="300"/>


Our file system will have a single superblock, which will always be at block 0 on the disk. Its layout is defined by struct superblock in `fs_types.h`. Block 0 is typically reserved to hold boot loaders and partition tables, so file systems generally do not use the very first disk block. Since our file system is not meant to be used on a real disk, we use block 0 to store the superblock for simplicity.

The superblock in our file system contains a reference to a block containing the "root" inode (the `s_root` field in `struct superblock`). The "root" inode is the inode for the file system's root directory. This inode stores pointers to blocks; these blocks, together, contain a sequence of `dirent` structures. Each structure includes a `file` and an `inode number` (one can think of this as assigning a given "name" to a given `inode`); the collection of these structures forms the content of the file system's root directory. These contents can include further directories, etc.


### The Block Bitmap: Managing Free Disk Blocks
In the same way that the kernel must manage the system's physical memory to ensure that a given physical page is used for only one purpose at a time, a file system must manage the blocks of storage on a disk to ensure that a given disk block is used for only one purpose at a time. In WeensyOS, you kept the `physical_pageinfo` structures for all physical pages in an array, pageinfo, to keep track of the free physical pages in `kernel.c`. In file systems it is common to keep track of free disk blocks using a bitmap (essentially, an array of bits, one for each resource that is being tracked). A given bit in the bitmap is set if the corresponding block is free, and clear if the corresponding block is in use.

The bitmap in our file system always starts at disk block 1, immediately after the superblock. For simplicity we will reserve enough bitmap blocks to hold one bit for each block in the entire disk, including the blocks containing the superblock and the bitmap itself. We will simply make sure that the bitmap bits corresponding to these special, "reserved" areas of the disk are always clear (marked in-use). Note that, since our file system uses 4096-byte blocks, each bitmap block contains `4096*8=32768` bits, or enough bits to track 32768 disk blocks.


### File Metadata
The layout of the meta-data describing a file in our file system is described by struct inode in `fs_types.h`. This meta-data includes the file's size, type (regular file, directory, symbolic link, etc.), time stamps, permission information, and pointers to the data blocks of the file. Because our file system supports hard links, one inode may be referred to by more than one name -- which is why the inode itself does not store the file "name". Instead, directory entries give names to inodes (as noted earlier).

The `i_direct` array in `struct inode` contains space to store the block numbers of the first 10 (N_DIRECT) blocks of the file, which we will call the file's direct blocks. For small files up to 10 * 4096 = 40KB in size, this means that the block numbers of all of the file's blocks will fit directly within the inode structure itself. For larger files, however, we need a place to hold the rest of the file's block numbers. For any file greater than 40KB, an additional disk block, called the file's indirect block, holds up to 4096/4 = 1024 additional block numbers, pushing the maximum file size up to (10 + 1024) * 4096 = 4136KB, or a little over 4MB. The file system also supports double-indirect blocks. A double-indirect block (`i_double` in the inode structure) stores 4096/4 = 1024 additional indirect block numbers, which themselves each store 1024 additional direct block numbers. This affords an additional 1024 * 1024 * 4096 = 4GB worth of data, bringing the maximum file size to a little over 4GB, in theory. To support even larger files, real-world file systems typically support triple-indirect blocks (and sometimes beyond).

### Other Features
Our file system supports all the traditional UNIX notions of file ownership, permissions, hard and symbolic links, time stamps, and special device files. Perhaps surprisingly, much of this functionality will come for free (or very low cost) after writing just a small number of core file system operations. Some of the ease in supporting these traditional UNIX file system notions comes from FUSE.

# Goal
You will implement some components of the FUSE driver (and hence the file system): allocating disk blocks, mapping file offsets to disk blocks, and freeing disk blocks allocated in inodes. In order to do this, you will have to familiarize yourself with the provided code and the various file system interfaces.

# Source files
* `bitmap.c`, `bitmap.h`: operations for manipulating the free disk block bitmap.
* `dir.c`, `dir.h`: operations for manipulating directories, including adding entries to directories and walking the directory structure on-disk to access a file.
* `fs_types.h`: contains structure and macro definitions relevant to the layout of the file system.
* `inode.c`, `inode.h`: operations for reading and writing data to inodes on-disk.
* `disk_map.c`, `disk_map.h`: contains the `flush_block()` and `diskblock2memaddr()` functions, both of which are vital for the functions that you will write.
* `fsdriver.c`: the main source file for the file system driver.
* `fsformat.c`: the main source file for the file system formatting utility.

The main file system code that we've provided for you resides in `fsdriver.c`. This file contains all the FUSE callbacks to handle I/O syscalls, as well as the main function. Once the path to the disk image and the path to the mount point (`testfs.img` and `mnt` respectively in the supplied example) have been read from the command line arguments, our FUSE driver uses `mmap()` to map the specified disk image into memory. This happens in the `map_disk_image` function (defined in `disk_map.c`), which itself initializes some file system metadata. Then, `fsdriver.c` calls `fuse_main`, which handles kernel dispatches to our registered callbacks. These callbacks will invoke the functions that you write in the coming exercises.

As stated in class, `mmap` reserves a portion of the running process's virtual address space to provide read and write access to a file as if that file were an array in memory. For example, if file is a pointer to the first byte of a memory-mapped file, writing `((char *)file)[5] = 3` is approximately equivalent to the two calls `lseek(fd, 5, SEEK_SET)` and then `write(fd, <location of memory that holds byte 3>, 1)`. To flush any in-memory changes you've made to a file onto the disk, you would use the `msync` function. As always, you can check the man pages for more information on these syscalls. For this lab, you will not need to be intimately familiar with their operation, but you should have a high-level understanding of what they do.

**Heads up: some key tips for the lab.**

* It’s virtually impossible to do this lab correctly without reading the supplied code and specs carefully. This includes comments, programming idioms in the supplied code, specs of functions that you are supposed to call, specs of functions that you are supposed to write, and more.

* Expanding on the prior point: if you’re not sure how to do something, look around in the same file for how other functions are implemented in the supplied code; this will give you programming hints.

* As usual, bugs in earlier exercises may show up only in later exercises, or in later grading tests.

* In computing, it’s generally impossible to validate something completely with a fixed set of tests; here too, it is possible to pass the tests with wrong logic that is later penalized by hand grading. Thus, you will want to make sure not only that you are passing the tests but also that the logic is sensible.

* You will very likely need to use the debugger (`gdb`). We include instructions below on running the driver under `gdb`; for gdb-specific commands, such as breakpoint-setting, please see lab 1.

# The work

{: .important-title }
> ### Exercise 1.
> 
> Before coding in the driver, cd to the `lab5` directory and run `./chmod-walk` in the Lab 5 directory. This will set up permissions in the directories leading up to your lab directory correctly so that you will be able to run the driver successfully. (If you do not run this script, FUSE will be unable to mount or unmount file systems in your lab directory.)

When you run the FUSE driver (`$ build/fsdriver testfs.img mnt`), the function `map_disk_image` will set the bitmap pointer. After this, we can treat bitmap as a packed array of bits, one for each block on the disk. See, for example, `diskblock_is_free`, which simply checks whether a given block is marked free in the bitmap.

{: .important-title }
> ### Exercise 2.
> 
> Implement `alloc_diskblock` in `bitmap.c`. It should find a free disk block in the bitmap, mark it used, and return the number of that block. When you allocate a block, you should immediately flush the changed bitmap block to disk with `flush_block`, to help file system consistency. Under the hood, `flush_block` calls msync to schedule the disk block in memory to get flushed to the actual disk image.

Note: You should use `free_diskblock` as a model.

Use `make grade` to test your code; run it as `sudo` like this:

```sh 
$ sudo make grade
```
Your code should now pass the "alloc_diskblock" test.

 

# Debugging
The output of our `make grade` will usually not provide you with enough information to debug a problem. Here are some debugging guidelines.

Debugging is crucial in this lab. Or, put differently: it would be very surprising if `make grade` gives all of the points on your first attempt. And when it does not give you all of the points, you will need to debug. This section tells you how.

First, when the grading script fails, look at the scripts in `test/` to see what is actually happening. Some of the tests are in the `fs_test()` function in `fsdriver.c`. Others are invoked by separate C programs. You will have to identify which test or test program is causing the failure. This is a prior step to using gdb and requires doing some detective work on the grading script and the supplied code. (This sort of detective work is a necessary skill when creating or contributing to any non-trivial software project.)

Second, note that “Transport endpoint is not connected” and “Software caused connection abort” are errors that user programs see when the file system driver panics or otherwise crashes and is no longer handling system calls (`open()`, `read()`, etc.) for that mount point. So, when a program starts producing these errors (for example, from a grading script), you will probably need to run the driver in `gdb` to see where it’s panicking.

The rest of this section gives building blocks and a detailed howto for `gdb`. The howto assumes that you have read and absorbed the building blocks.

## Basic approach and building blocks

The approach to debugging has three high level steps. (1) Set up the file system (the grading scripts do this for you but when debugging you need to do it semi-manually), (2) Run the file system driver (either standalone or in gdb) that invokes your file system, and (3) Run a program that actually interacts with the file system (again, either standalone or in gdb). Each of these steps requires specific commands from you; we delve into these commands now.

For step (1), setting up the file system semi-manually, a relatively straightforward to do this is to run:

```sh
  $ test/testbasic.bash
```
This script creates a disk image, `testfs.img`, which is set up properly for further internal tests in the driver. It also runs some basic tests on the driver (specifically:

```sh
  $ build/fsdriver testfs.img mnt --test-ops
```
which you might find useful to run on its own.)

You can also delve into the bash scripts and try to reproduce the kinds of commands that are in, say, `test/teststress.bash`. Be aware that these shell scripts call shell library functions that are defined in `test/libtest.bash`.

For steps (2) and (3), you will typically need to run these side-by-side, literally. To do this, you will want to use the tmux program. Therefore, if you haven’t already done so for lab 4, take 10 minutes right now to do the `tmux` tutorial.

For step (2), running the file system driver, you will want to run using the `-d` flag. The standalone version is as follows (later on in this description we will show it for `gdb`).

```sh
  $ build/fsdriver -d testfs.img mnt
```
or you may need to preface this command with `sudo`:
```sh
  $ sudo build/fsdriver -d testfs.img mnt
```
This command runs the driver in debugging mode and mounts `testfs.img` at `mnt`. In debugging mode, the driver does not exit until you press `Ctrl-C`. This means that you cannot interact with the file system via the terminal containing the command above; instead, you will need to interact with the file system from another terminal (again, using `tmux`).

While in debugging mode, `fsdriver` will print to the console a trace of the syscalls dispatched to the driver; any `printf`s that you insert in the driver will also be displayed. Once you run the command above, you should see something like:

```bash
  FUSE library version: 2.9.2
  nullpath_ok: 0
  nopath: 0
  utime_omit_ok: 0
  unique: 1, opcode: INIT (26), nodeid: 0, insize: 56, pid: 0
  INIT: 7.23
  flags=0x0003f7fb
  max_readahead=0x00020000
  INIT: 7.19
  flags=0x00000011
  max_readahead=0x00020000
  max_write=0x00020000
  max_background=0
  congestion_threshold=0
  unique: 1, success, outsize: 40
  ...
```

For step (3), you will use another terminal (again, using `tmux`). Something like:

```bash
  $ ls mnt 
```
will cause the original terminal to print germane output.

To actually debug, you can use `printf` or `gdb`. Any `printf`s that you add will be displayed in the terminal with `fsdriver`, as stated above. However, `gdb` is likely to be more effective. See below for instructions.

### Running the driver in gdb:

We assume that you have already done step (1) above. This section is about how to run steps (2) and (3) under the debugger.

If you haven’t already done so for lab 4, take 10 minutes right now to do the `tmux` tutorial.

Then look over how we use `gdb` with `tmux` in lab 4.

In this case, do the following, recalling that `C-b %` means “type Ctrl-b together, let go, and then type the % key”:

```bash
  $ tmux
  C-b %
```

At this point, you should have two side-by-side panes, with the active one being the one on the right. Go back to the one on the left with `C-b <left>`` (which means “Ctrl-b together, let go, and then type the left arrow key”), and type the following. You might again need to preface the command with sudo:

```sh
  $ gdb build/fsdriver
  <set breakpoints, as usual>
  (gdb) run -d testfs.img mnt 
```

In this way, you will be running `build/fsdriver -d testfs.img mnt` or `sudo build/fsdriver -d testfs.img mnt` but in the debugger.

Then back to the right-hand pane with `C-b <right>`. In that terminal:

```bash
  # manually run anything driven from the test/ directory,
  # possibly under gdb. For example:

  $ build/posixio

  # or

  $ gdb build/stressfs
  <set breakpoints, as usual>
  (gdb) run
 ```

## Cleaning up. 
During the course of testing your FUSE driver, various combinations of operations may cause your FUSE driver to stop functioning or enter a non-clean state. It may be helpful to search Campuswire for information about specific error messages.

More generally, to get your system to a clean starting state, you can do the following (you may need to preface some of these commands with sudo)

```bash
    $ fusermount -u mnt          # unmount the driver
```
If that does not work, then force it with:

```bash
    $ sudo umount -f mnt
```

and possibly:

```sh
    $ sudo umount -f /lab5mnt
```
Then continue:

```sh 
    $ echo 'sample' > build/msg  # create a message
    $ test/makeimage.bash        # make a clean testfs.img
    $ rm -f mnt                  # remove the symlink
    $ sudo rm -rf /lab5mnt       # remove the mounting directory and its residents
    $ sudo mkdir /lab5mnt        # recreate the mounting directory
    $ ln -s /lab5mnt mnt         # recreate the symlink
```
You may wish to encapsulate these actions in a shell script.

Once you have executed the above lines, you start the driver again with the following, possibly prepended with `sudo`:

```sh
    $ build/fsdriver -d testfs.img mnt
```
# File Operations

We have provided various functions in `dir.c` and `inode.c` to implement the basic facilities you will need to interpret and manage inode structures, scan and manage the entries of directories, and walk the file system from the root to resolve an absolute pathname. Read through all of the code in these files and make sure you understand what each function does before proceeding.

{: .important-title }
> ## Exercise 3. Implement inode_block_walk and inode_get_block in inode.c.
> 
> These are the workhorses of the file system. For example, inode_read and inode_write aren’t much more than the bookkeeping atop inode_get_block necessary to copy bytes between scattered blocks and a sequential buffer.
>
> Their signatures are:
> ```sh 
> int inode_block_walk(struct inode *ino, uint32_t filebno, uint32_t **ppdiskbno, bool alloc);
> int inode_get_block(struct inode *ino, uint32_t filebno, char **blk);
> ```
> `inode_block_walk` has similar logic to the virtual memory lookup function in lab4. It finds the disk block number slot for the 'filebno'th block in inode 'ino', and sets `*ppdiskbno` to point to that slot. `inode_get_block` goes one step further and sets `*blk` to the start of the block, such that by using `*blk`, we can access the contents of the block. It also allocates a new block if necessary.
>
>The pointers-to-pointers may be confusing. It’s best to draw pictures, or look at other code in `inode.c`, or think about how to call the function. Also, as a reminder: in C, when we want to return multiple values from a function (or return one value, in a function returning an int status code), we set up the function to take a pointer (address) as a parameter, and we put the return value in the supplied address (by dereferencing the pointer). As an example, if we have a function `f(int* p)`, the implementation of `f` can return an `int` to the caller by storing into (dereferencing) the supplied address, with a line like `*p = 5`. This identical pattern holds when the return value is itself a pointer. In our example, the return value is an `uint32_t*` (an address whose contents will be a 32-bit block number). So, the caller passes storage for that `uint32_t*`: an `uint32_t**`.
>
> Use `make grade` to test your code (again, by invoking: `$ sudo make grade`). Your code should pass the "inode_open" and "inode_get_block" tests.

After Exercise 3, you should be able to read and write to the file system. Try something like

```bash
$ echo "hello" > "mnt/world"; cat "mnt/world"
```

{: .important-title }
> ## Exercise 4. Implement `inode_truncate_blocks` in `inode.c`. 
> 
> `inode_truncate_blocks` frees data and metadata blocks that an inode allocated but no longer needs. This function is used, for instance, when an inode is deleted; the space reserved by the inode must be freed so that other files can be created on the system.
>
> Use `make grade` to test your code (`$ sudo make grade`). Your code should pass the "inode_flush/inode_truncate/file rewrite" tests.



{: .important-title }
> ## Exercise 5. 
> 
> Implement `inode_link` and `inode_unlink` in `inode.c`. `inode_link` links an inode referenced by one path to another location, and `inode_unlink` removes a reference to an inode at a specified path. Make sure that you properly increment the link count in an inode when linking and decrement the link count when unlinking. Don't forget to free an inode when its link count reaches zero!
> 
> `inode_link` and `inode_unlink` allow us to exploit the level of indirection provided by using inodes in our file system (as opposed to storing all file meta-data inside of directories, for instance) and manage referencing inodes with multiple names. The `inode_unlink` operation is particularly important as it allows us to release the space reserved for an inode, acting as a "remove" operation when an inode's link count is one.
> 
> Use `make grade` to test your code (`$ sudo make grade`). Your code should pass the "inode_link/inode_unlink" tests.

After Exercise 5, you should be able to make hard links. Try something like

```sh 
$ echo "hello" > "mnt/world"; ln "mnt/world" "mnt/hello"; rm "mnt/world"; cat "mnt/hello"
```

The tests after "inode_link/inode_unlink" are all effectively stress tests, in some way or another, for the driver. Each of them relies on the core functionality that you implemented; some can fail if you didn't handle certain edge cases correctly. If you fail one of these tests, go back and check the logic in your code to make sure you didn't miss anything.

{: .important-title }
> ## Exercise 6. 
> 
> Run your `ls` from lab 2 against the file system in `/lab5mnt`. Paste the output of
>
> ```sh 
>    $ /path/to/your/ls -alR /lab5mnt
> ```
>
>into `answers.txt`. If you have a non-working lab 2, then just note that in the `answers.txt` file.
>
> You can have fun –- not graded –- using the fact that you are writing both `ls` and implementing the file system. For example, you could consider having your file system stuff coded messages in extraneous dirents, and then interpret/decode them in `ls`.

# Extra credit questions
Do either of the following for extra credit (you can do both, but extra credit is given for only one). As in lab4, the points given will not be commensurate with effort required.

{: .important-title }
> ## Exercise 7. 
>
>The file system is likely to be corrupted if it gets interrupted in the middle of an operation (for example, by a crash or a reboot). Implement soft updates or journalling to make the file system crash-resilient and demonstrate some situation where the old file system would get corrupted, but yours doesn't.



{: .important-title }
> ## Exercise 8. 
>
> Currently, our file system allocates one block (4096 bytes) per inode. However, each struct inode only takes up 98 bytes. If we were clever with file system design, we could store 4096/98 = 41 inodes in every block. Modify the file system so that inodes are stored more compactly on disk. You may want to make the file system more like a traditional UNIX file system by splitting up the disk into inode and data regions, so that it is easier to reference inodes by an index (generally called an "inum" for "inode number") into the inode region.


# Adrienne's Replacements 

Here are draft write-ups for all three modified exercises:

---

## Exercise 2 (replacement): Free List Block Allocation

In the original file system, free disk blocks were tracked using a *bitmap* — a compact array of bits, one per block, where a set bit indicates a free block. In this lab, we use a different approach: a **free list**, stored explicitly on disk.

The free list is a singly-linked list of free blocks. Each free block stores, in its first 4 bytes, the block number of the next free block in the list (or 0 if it is the last). The superblock's `s_freelist` field holds the block number of the first free block in the list (the head). When no free blocks remain, `s_freelist` is 0.

This structure has a pleasing property: the free list requires no extra storage beyond the free blocks themselves. The metadata lives inside the blocks being tracked.

**Exercise 2.** Implement `alloc_diskblock` and `free_diskblock` in `freelist.c`.

`alloc_diskblock` should remove the head block from the free list, update `s_freelist` in the superblock to point to the next free block, zero out the allocated block's contents, and return the block number. As with the bitmap version, you must flush any modified blocks to disk immediately after changing them, to maintain consistency.

`free_diskblock` should insert the given block at the head of the free list, writing the old head's block number into the first 4 bytes of the newly freed block, and updating `s_freelist` accordingly. Again, flush immediately.

**Note:** Study `diskblock2memaddr` and `flush_block` carefully — you will need both. Also consider: what happens if `alloc_diskblock` is called when the free list is empty? Return an appropriate error.

Use `make grade` to test your code. Your code should pass the `alloc_diskblock` test.

**Think about:** What are the trade-offs between a free list and a bitmap? Under what workloads would each perform better? (You do not need to write up an answer, but keep this in mind for the exam.)

---

## Exercise 5 (replacement): Symbolic Links

In the previous exercise, you implemented `inode_link` and `inode_unlink` to manage *hard links* — multiple directory entries pointing directly to the same inode. In this exercise, you will implement *symbolic links* (symlinks), which work differently: a symlink is its own inode, whose data contains the path to the target file as a plain string. The target inode's link count is not affected.

When the file system resolves a path that passes through a symlink, it must detect the symlink and follow it — substituting the symlink's stored path and continuing resolution from there. Your implementation should handle symlinks that appear at any component of a path, not just at the end.

**Exercise 5.** Implement the following in `inode.c` and `dir.c`:

- `inode_symlink(const char *src, const char *dst)`: Creates a new inode of type `T_SYM` at path `dst`. The data of this inode should be the null-terminated string `src` (the target path). You will need to allocate and write a data block to hold the target path.

- `inode_readlink(struct inode *ino, char *buf, size_t bufsz)`: Reads the target path stored in a symlink inode into `buf`. Returns an error if `ino` is not of type `T_SYM`.

- Modify `dir_walk` in `dir.c` to detect when a path component resolves to a `T_SYM` inode, read the target path with `inode_readlink`, and continue resolution from that path. You should handle both absolute and relative symlink targets. To prevent infinite loops, return an error (e.g., `-ELOOP`) if you follow more than 8 symlinks in a single resolution.

**Note:** Unlike hard links, a symlink can point to a nonexistent target (a "dangling" symlink). Your implementation should not error at creation time if the target does not exist — only at resolution time.

Use `make grade` to test your code. Your code should pass the `inode_symlink/inode_readlink` tests.

After completing this exercise, verify your implementation with:

```
$ echo "hello" > mnt/target
$ ln -s target mnt/link
$ cat mnt/link
hello
$ rm mnt/target
$ cat mnt/link   # should produce an error — dangling symlink
```

---

## Exercise 4b (new): File System Consistency Checker

Even a correct file system implementation can end up in an inconsistent state if the driver is interrupted mid-operation — for example, if a block is allocated but never linked into an inode, or if a directory entry references an inode whose link count was never incremented. A *file system checker* (`fsck`) scans the on-disk structure and verifies that a set of invariants hold, reporting (and optionally repairing) any violations it finds.

You will implement a checker that runs *offline* — directly against an unmounted disk image, without going through FUSE — as a standalone program `build/fsck`.

**Exercise 4b.** Implement `fsck` in `fsck.c`. Your checker must verify all of the following invariants. For each violation found, print a descriptive message to stdout identifying the affected inode or block number and the nature of the violation.

1. **Superblock validity.** The magic number is correct, and `s_root` refers to a valid, in-use block.

2. **Bitmap / free list consistency.** Every block marked as in-use is reachable from some inode (via direct, indirect, or double-indirect pointers), the superblock, or the bitmap/free list region itself. Every block reachable from an inode is marked in-use. (No block should be both reachable and marked free, and no block should be marked in-use but unreachable.)

3. **No double allocation.** No disk block is referenced by more than one inode.

4. **Link count consistency.** For each inode, the link count stored in `i_nlink` equals the number of directory entries in the file system that reference that inode's block number. (Traverse all directories to count references.)

5. **Directory structure validity.** Every inode of type `T_DIR` contains a sequence of valid `dirent` structures. Each `dirent` with a nonzero inode number must reference a block that contains a valid inode. No directory (other than the root) should be unreachable from the root via directory traversal.

6. **No cycles in the directory tree.** The directory structure must form a tree rooted at the root inode, not a graph with cycles.

Your checker does not need to *repair* violations, only detect and report them.

**To test your checker**, we provide a set of intentionally corrupted disk images in `test/corrupt-images/`. Each image contains exactly one type of violation. Run:

```
$ build/fsck test/corrupt-images/double-alloc.img
$ build/fsck test/corrupt-images/bad-linkcount.img
$ build/fsck test/corrupt-images/unreachable-block.img
```

Your checker should correctly identify the violation in each image and exit with a nonzero status. On a clean image it should print nothing and exit 0.

Use `make grade` to test. Your code should pass the `fsck` tests.

**Think about:** Which of these invariants could be violated by a crash between two `flush_block` calls in your `inode_truncate_blocks` implementation? Can you identify a specific operation sequence that would produce a double-allocation? (No write-up required, but this connects directly to the journaling extra credit.)

---

A few notes on these drafts: the symlink exercise assumes you add a `T_SYM` type constant and an `s_freelist` field to `fs_types.h` / the superblock struct — you'd need to update the scaffolding accordingly. The `fsck` exercise also requires you to provide the corrupted test images, which you could generate with a small script that runs the driver and then hex-edits the resulting `.img` file. Happy to help draft those image-generation scripts or the scaffolding changes if useful.


# Further questions
Answer the following questions in `answers.txt`.

* How long approximately did it take you to do this lab?
* Do you feel like you gained an understanding of how to build a file system in this lab? Please suggest improvements.


# Submission
## Handing in consists of three steps:

## Executing this checklist:

Make sure your code builds, with no compiler warnings.
Make sure you’ve used git add to add any files that you’ve created.
Fill out the top of the answers.txt file, including your name and NYU Id
Make sure you’ve answered every question in answers.txt
Make sure you have answered all code exercises in the files.
Create a file called slack.txt noting how many slack days you have used for this assignment. (This is to help us agree on the number that you have used.) Include this file even if you didn’t use any slack days.
git add and commit the slack.txt file
Push your code to GitHub, so we have it (from outside the container or, if on Mac, this will also work from within the container):

$ cd ~/cs5600/lab5 
$ make clean
$ git commit -am "hand in lab5"
$ git push origin 

Counting objects: ...
....
To  git@github.com:nyu-cs5600/XXX-<YourGithubUsername>.git
  7337116..ceed758  main -> main
Actually submit, by timestamping and identifying your pushed code:

Decide which git commit you want us to grade, and copy its id (you will paste it in the next sub-step). A commit id is a 40-character hexadecimal string. Usually the commit id that you want will be the one that you created last. The easiest way to obtain the commit id for the last commit is by running the command git log -1 --format=oneline. This prints both the commit id and the initial line of the commit message. If you want to submit a previous commit, there are multiple ways to get the commit id for an earlier commit. One way is to use the tool gitk. Another is git log -p, as explained here, or git show.
Create a new file commit.txt locally with only the commit id you just copied.
Now go to Gradescope and select the “Lab 5” assignment; then choose Upload as the submission method. Submit only commit.txt.
You can submit multiple times before the deadline. We will grade the last commit id submitted before the deadline.
NOTE: Ground truth is what and when you submitted to Gradescope. Thus, a non-existent commit id in Gradescope means that you have not submitted the lab, regardless of what you have pushed to GitHub. And, the time of your submission for the purposes of tracking lateness is the time when you submit the assignment through Gradescope, not the time when you executed git commit.

After you submit your assignment, we will process it as follows:

We’ll checkout your GitHub repository using the commit ID you provided in the commit.txt file. We may upload this code to your Gradescope assignment. As a result, you may notice that your latest submission on Gradescope is marked as “late,” even if you submitted on time. Don’t worry about this discrepancy; it’s an expected part of our process. What matters is that you submit your commit.txt file on time. To verify your actual submission time, always refer to the latest submission of commit.txt in the “Submission History” on Gradescope. This is the timestamp we use to determine if your assignment was submitted by the deadline. We use this method to enable direct annotation of your code on Gradescope, allowing us to provide more detailed feedback.

Remember, as long as your commit.txt is submitted before the deadline with the correct commit ID, your assignment will be considered on time, regardless of when the code appears on Gradescope.

This completes the lab.

# Acknowledgements
The diagram explaining FUSE is adapted from the diagram displayed on FUSE's homepage.

This lab is an edited version of a lab written by Isami Romanowski. (He in turn adapted code from MIT's JOS, porting it to the FUSE and Linux environment, adding inodes and more.)