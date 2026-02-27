---
title: 'Homework 7'
layout: homework
week: 8
released: 2026-03-02
due: 2026-03-13
summary: 'Virtual Memory and Page Replacement. '
---


# 1. Virtual memory  (10 points)

Below are five statements about virtual memory.
* Write down **True** if you agree with the statement; otherwise, write **False**.

1. Virtual memory provides programmability (or sometimes called transparency) that multiple programs can use the same address say 0x123456 without a conflict.
1. Each physical page can be mapped to multiple virtual pages in one process.
1. Each virtual page can map to multiple physical pages.
1. On a 32 bit machine with 8 GB RAM installed, the physical address would be larger than the virtual address space of one process.
1. TLB accelerates VM translation by assuming spatial and temporal locality. If the assumptions do not hold, TLB is unable to speed up VM translation.



# 2. Run the mmap experiment (4 points)

Copy the code below to a file named `mmap.c`.

```c
/* file: mmap.c */

#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>

void mmapwrite(int fd, int size);
void normalwrite(int fd, int size);

int main(int argc, char **argv) {
	struct stat stat;
	int fd;

	if (argc != 2) { // Check for required cmd line arg
		printf("usage: %s <filename>\n", argv[0]);
		exit(0);
	}

	/* Copy input file to stdout */
	if ((fd = open(argv[1], O_RDONLY, 0)) < 0)
	perror("open");

	fstat(fd, &stat);

	// option 1
	mmapwrite(fd, stat.st_size);

	/* // option 2
	* normalwrite(fd, stat.st_size);
	*/

	close(fd);
	return 0;
}

void mmapwrite(int fd, int size) {
	/* Ptr to memory mapped area */
	char *bufp;
	bufp = mmap(NULL, size, PROT_READ, MAP_PRIVATE, fd, 0);
	write(STDOUT_FILENO, bufp, size);
	return;
}


void normalwrite(int fd, int size) {
	char *buf = malloc(size);
	read(fd, buf, size);
	write(STDOUT_FILENO, buf, size);
	return;
}
```

Run the code as shown below. Run it first with the "Option 1" uncommented in the `main` function, then comment out "Option 1" and uncomment "Option 2" and run a second time. 

```sh
$ cat /dev/urandom | head -c 1000000000 > 1G.file
$ gcc mmap.c -o mmap
$ time ./mmap 1G.file > /dev/null
```

After running both options, answer the question: 
Which runs faster, option 1 or option 2? What is the time difference on your machine?

When you're done, don't forget to delete the giant file we created to play with! 

```sh
$ rm 1G.file
```


# 3. Page replacement policy: CLOCK  (6 points)

Suppose you have a machine named CS5600-clover with 4 pages of memory (total 16KB) and a 1TB SSD.
* A process uses 10 pages 
	* page P0-P3 are in the memory
	* P4-P9 are on SSD.
* The OS runs the CLOCK algorithm for page replacement, and a visualization is below.

```
      P0 (A=0)
       ^
       |
 P3    +     P1
(A=0)       (A=0)

      P2 (A=0)
```

For the CLOCK algorithm:
- The "hand"/"pointer" will go clock-wise.
- When evicting a page, check the page pointed by the "hand"
  - if access bit is set (A=1), clear the bit (A=0) and advance one position.
  - if A=0, evict the pointed page, load the new page, and advance one position.
  	- what's the access bit of the newly loaded page? 
  		* answer: A=1; the page is now being used.
- Note: CLOCK is a family of algorithms. You might see different variants but the core never changes--- CLOCK can be perfectly implemented on hardware.


Let's say we have memory accesses in the following order:

    P0, P2, P3, P4, P5, P0, P9

Answer the following questions:

* How many page swaps will happen? 
* For each swap, which page has been swapped in and which page has been swapped out (namely, the victim)?
