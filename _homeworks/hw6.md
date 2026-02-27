---
title: 'Homework 6'
layout: homework
week: 7
released: 2026-02-26
due: 2026-03-06
summary: 'Virtual Memory.'
---

<script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11.12.2/+esm';
    mermaid.initialize({ startOnLoad: true });
</script>



## 1. Byte-addressable vs. Bit-addressable memory

Imagine we have a CS5600-SEAMem machine whose virtual address is 33bits and it uses 64kB pages.

### Q1A

Assume our CS5600-SEAMem machine is byte-addressable like x86. (This means the machine can access individual BYTES-- address `0x1` and `0x2` point to two bytes that are *adjacent*). 

How many bits will the offset in the VA and PA need in order to access all bytes of a 64KB page? (2 pts)


### Q1B

Now, assume CS5600-SEAMem is **bit**-addressable: it can access every **bit** in memory, meaning two the addresses `0x1` and `0x2` point to adjacent two *bits*. 

How many bits will the offset in the VA and PA need in order to access all bits in a 64KB page? (2 points)


### Q1C 

Continue to assume the machine is bit-addressable. 

If the physical address is 28 bits, how many bits does the PPN (physical page number) have? (2 points)


# 2. Two-level page table

Our current CS5600-SEAMem has:

- bit-addressable memory,
- 64KB pages,
- a 33 bit VA,
- and a 28 bit PA.

We choose to use a 2-level page table for address translation. Assume that PTEs are 4 bytes. 

### Q2A

What is the minimum memory consumed by a program's page table? (2 points)

### Q2B

How many pages (both VM and PM) can a program can use, at most? (2 points) 



## 3. Simulate CPU and Walk the page tables

- This is the standard x86 32-bit two-level page table structure (not x86-64; we use 32-bit for simplicity).
	* Remember: 
		* Addresses are 32 bits
		* Since we assume the addresses are page-aligned, the bottom 12 bits of an address are always 0
		* We'll re-use those bottom 12 bits to track permissions
		* The address in the `%cr3` register indicates the starting address of the L1 page table. 
- The permission bits (bottom 12 bits) of page directory entries and page table entries are set to `0x007`.
	* This means page present, read-write, and user-mode; 
	* The virtual addresses are valid, and that user programs can read (load) from and write (store) to the virtual address.
- The memory pages are listed below. 
	* On the left side of the pages are the addresses; on the right is the content. 
	* For example, the address of the top memory block (4 bytes) is `0xffff5ffc`, and its content is `0xdeadbeef`.)

```
%cr3:  0xffff1000
``` 

{: .scheduling-table }
| Address   | Content | 
| ---- 	    | -----   |
|`0xffff5ffc` | `0xdeadbeef`|
|			| ... |
|`0xffff5800` | `0xff005000` |
|			| ... |
|`0xffff5000` | `0xc5202000` |
|			| ... |
|`0xffff1ffc` | `0xd5202007`|
|			| ... |
|`0xffff1800` | `0xef005007`|
|			| ... |
|`0xffff1000` | `0xf0f02007`|
|			| ... |
|`0xff005ffc` | `0xbeebebee`|
|			| ... |
|`0xff005800` | `0xf00f8000`|
|			| ... |
|`0xff005000` | `0xc5201000`|
|			| ... |
|`0xf0f02ffc` | `0xf00f3007`|
|			| ... |
|`0xf0f02800` | `0xf0f05007`|
|			| ... |
|`0xf0f02200` | `0xff005007`|
|			| ... |
|`0xf0f02000` | `0xeeff5007` |
|			| ... |



### Q3A

Split the 32-bit VA `0x00200ffc` into the L1 index (10-bit), L2 index (10-bit), and offset (12-bit). 

Write them down in **decimal** numbers.  (2 points)


### Q3B
When accessing VA `0x00200ffc` using the above table, which L1/L2 page tables are used?

Write down the L1 and L2 page table starting addresses (that is, the physical address of the first byte of those page tables). (2 points)

### Q3C

Assuming the memory as indicated above, what is the output of the following code?  (2 points) 
(hint: (1) because of this is x86-32, there are 1024 PT entries in a PT page (4KB = 32bit x 1024); (2) notice the L2 index in the question 3.a

```c
#include "stdio.h"
int main() {
	int *ptr2 = (int *) 0x00200ffc;
	printf("%x\n", *ptr2);
}
```

### Q3D

Copy the above code to a `.c` file, compile, and run. What do you see? and why? (explain in 1 sentence)  (4 points)

