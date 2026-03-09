---
title: 'Homework 7'
layout: homework
week: 8
released: 2026-03-02
due: 2026-03-13
summary: 'Virtual Memory and Page Replacement. '
---

<script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11.12.2/+esm';
    mermaid.initialize({ startOnLoad: true });
</script>


# 1. Virtual memory  (10 points)

Below are five statements about virtual memory.
* Write down **True** if you agree with the statement; otherwise, write **False**.

1. Virtual memory provides programmability (or sometimes called transparency) that multiple programs can use the same address say 0x123456 without a conflict.
1. Each physical page can be mapped to multiple virtual pages in one process.
1. Each virtual page can map to multiple physical pages.
1. On a 32 bit machine with 8 GB RAM installed, the physical address would be larger than the virtual address space of one process.
1. TLB accelerates VM translation by assuming spatial and temporal locality. If the assumptions do not hold, TLB is unable to speed up VM translation.
1. A page table entry for a valid page must always contain a non-zero physical frame number.
1. Two different processes can have their virtual page 0 mapped to the same physical frame simultaneously.
1. A process with a 32-bit address space can only use a maximum of 4 GB of physical memory, even if more RAM is installed.
1. The page table for a process must itself be stored in physical memory at all times.
1. If two processes share a read-only memory-mapped file, they must use the same virtual address to access it.


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


Which runs faster, option 1 or option 2? What is the time difference on your machine?

For full credit, include a screenshot of the output of these two runs. 

When you're done, don't forget to delete the giant file we created to play with! 

```sh
$ rm 1G.file
```


# 3. Page replacement policy: CLOCK  (6 points)

Suppose we are working on a machine with the following specs: 
* 4 pages of memory (total 16KB)
* 1 TB SSD. 

The OS uses the CLOCK algorithm for page replacement. To understand the CLOCK algorithm, imagine the face of a clock as shown below: 

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 300 300" width="300" height="300" font-family="monospace" font-size="13">

  <!-- Circle track -->
  <circle cx="150" cy="150" r="90" fill="none" stroke="#ccc" stroke-width="2" stroke-dasharray="6 4"/>

  <!-- P0 - Top -->
  <rect x="115" y="30" width="70" height="30" rx="6" fill="#4a90d9" stroke="#2c6fad" stroke-width="1.5"/>
  <text x="150" y="50" text-anchor="middle" fill="white" font-weight="bold">P0 (A=0)</text>

  <!-- P1 - Right -->
  <rect x="210" y="135" width="70" height="30" rx="6" fill="#4a90d9" stroke="#2c6fad" stroke-width="1.5"/>
  <text x="245" y="155" text-anchor="middle" fill="white" font-weight="bold">P1 (A=0)</text>

  <!-- P2 - Bottom -->
  <rect x="115" y="240" width="70" height="30" rx="6" fill="#4a90d9" stroke="#2c6fad" stroke-width="1.5"/>
  <text x="150" y="260" text-anchor="middle" fill="white" font-weight="bold">P2 (A=0)</text>

  <!-- P3 - Left -->
  <rect x="20" y="135" width="70" height="30" rx="6" fill="#4a90d9" stroke="#2c6fad" stroke-width="1.5"/>
  <text x="55" y="155" text-anchor="middle" fill="white" font-weight="bold">P3 (A=0)</text>

  <!-- Clock hand arrow pointing to P0 -->
  <line x1="150" y1="150" x2="150" y2="72" stroke="#e05c2a" stroke-width="2.5" stroke-linecap="round"/>
  <polygon points="150,62 145,76 155,76" fill="#e05c2a"/>

  <!-- Center dot -->
  <circle cx="150" cy="150" r="5" fill="#e05c2a"/>

  <!-- Label -->
  <text x="150" y="175" text-anchor="middle" fill="#888" font-size="11">clock hand</text>

</svg>

* There are 4 positions, one for each page
* If a position holds a page, it shows the page (e.g. P0)
* There is a *hand* or a pointer that goes clockwise
* When there is a page access: 
	* If the page is already present, set the access bit (A=1) to indicate recent access
	* If the page is not present in any of the 4 slots, run the eviction process (specified below)
* To evict a page: 
	1. Look at the page pointed at by the hand
		1. If the access bit is set (A=1):
			* Clear the bit (set A=0)  
			* Advance the hand one position
			* Go back to step 1
		1. If the access bit is clear (A=0):
			* Evict the page the hand is pointing at
			* Load the new page in that position
				* Set the access bit for the new page to be 1 (A=1)
			* Advance the hand one position

NOTE: CLOCK is a family of algorithms. You may see different variations, but the core is the same. 

## Q3, Part A

We have a process running that uses 10 pages overall
* Pages P0-P3 are in memory
* Pages P4-P9 are on SSD

We access the pages in the following order: 

    P0, P2, P3, P4, P5, P0, P9


Assume we begin with the page table in the status as shown in the diagram above. 

* How many page swaps will happen? 
	* For each page swap, specify which page gets swapped out and which gets swapped in. 

## Q3, Part B

Now we have a "clock" as shown in the diagram below. 

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 300 300" width="300" height="300" font-family="monospace" font-size="13">

  <!-- Circle track -->
  <circle cx="150" cy="150" r="90" fill="none" stroke="#ccc" stroke-width="2" stroke-dasharray="6 4"/>

  <!-- P0 - Top (12 o'clock) -->
  <rect x="115" y="30" width="70" height="30" rx="6" fill="#4a90d9" stroke="#2c6fad" stroke-width="1.5"/>
  <text x="150" y="50" text-anchor="middle" fill="white" font-weight="bold">P0 (A=0)</text>

  <!-- P1 - Top-right (2 o'clock) -->
  <rect x="210" y="78" width="70" height="30" rx="6" fill="#4a90d9" stroke="#2c6fad" stroke-width="1.5"/>
  <text x="245" y="98" text-anchor="middle" fill="white" font-weight="bold">P1 (A=0)</text>

  <!-- P2 - Bottom-right (4 o'clock) -->
  <rect x="210" y="192" width="70" height="30" rx="6" fill="#4a90d9" stroke="#2c6fad" stroke-width="1.5"/>
  <text x="245" y="212" text-anchor="middle" fill="white" font-weight="bold">P2 (A=0)</text>

  <!-- P3 - Bottom (6 o'clock) -->
  <rect x="115" y="240" width="70" height="30" rx="6" fill="#4a90d9" stroke="#2c6fad" stroke-width="1.5"/>
  <text x="150" y="260" text-anchor="middle" fill="white" font-weight="bold">P3 (A=0)</text>

  <!-- P4 - Bottom-left (8 o'clock) -->
  <rect x="20" y="192" width="70" height="30" rx="6" fill="#4a90d9" stroke="#2c6fad" stroke-width="1.5"/>
  <text x="55" y="212" text-anchor="middle" fill="white" font-weight="bold">P4 (A=0)</text>

  <!-- P5 - Top-left (10 o'clock) -->
  <rect x="20" y="78" width="70" height="30" rx="6" fill="#4a90d9" stroke="#2c6fad" stroke-width="1.5"/>
  <text x="55" y="98" text-anchor="middle" fill="white" font-weight="bold">P5 (A=0)</text>

  <!-- Clock hand arrow pointing to P0 -->
  <line x1="150" y1="150" x2="150" y2="72" stroke="#e05c2a" stroke-width="2.5" stroke-linecap="round"/>
  <polygon points="150,62 145,76 155,76" fill="#e05c2a"/>

  <!-- Center dot -->
  <circle cx="150" cy="150" r="5" fill="#e05c2a"/>

  <!-- Label -->
  <text x="150" y="175" text-anchor="middle" fill="#888" font-size="11">clock hand</text>

</svg>


We have a process running that uses 10 pages overall
* Pages P0-P5 are in memory
* Pages P6-P9 are on SSD

We access the pages in the following order: 

    P0 - P3 - P9 - P5 - P8 - P0 - P4 - P8


Assume we begin with the page table in the status as shown in the diagram above. 

* How many page swaps will happen? 
	* For each page swap, specify which page gets swapped out and which gets swapped in. 

