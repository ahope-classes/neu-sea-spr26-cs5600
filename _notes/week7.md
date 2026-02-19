---
title: 'Notes Week 7: Introduction to Virtual Memory'
layout: page
summary: 'Intro to VM and Paging'
---

<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@4/tex-mml-chtml.js"></script>

# Week 7: Intro to Virtual Memory and Paging
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

<details markdown="block">
<summary>Complete Table of Contents </summary>
1. TOC
{:toc}
</details>

## Agenda
{: .no_toc }

1. Lab 2 reviews
1. Intro to Virtual Memory
2. Bit Manipulation/Math Refresher
3. Paging
    - Intro
    - Page tables
    - Multilevel page table
    - Alternatives & tradeoffs


## Weekly Summary and Where are we?

### Topics

* **Last Week:** Finish up Synchronization
* **This Week:** Introduce Virtual Memory
* **Next Week:** More Virtual Memory, deeper into x86-64 arch, Lab 4, and paging

### Assignments

* **Lab 3** extended; due THIS Friday, 20 Feb 2026. 
* **Lab 4** is out-- start reading it! 


# Lab 2 Reviews

* Who wants to volunteer? 
* Short reminder: 
    * Implementing `ls`
    * Working with flags, multiple options
    * What did you learn? 
    * What was tricky? 
    * What of this have you done before/was familiar? 

# Finished Synchronization

## Review from last week

List 5 things! 

* 1)
* 2)
* 3)
* 4)
* 5)





# 1. Virtual memory intro

OSTEP: OS in Three Easy Pieces

The three easy pieces:

* Virtualization
* Concurrency
* Persistence
* Security


* Virtual Memory is part of Virtualization
    * Very important idea in computer systems!
    * virtual address translation:
      VA (virtual address) => PA (physical address)
    * setup: 

<img src="../../assets/images/notes/week7/rtimage_program_mem.png" alt="program in memory" width="400"/>

* heap, stack, program text.
* "to virtualize" means "to lie" or "to fool". we'll say how this is implemented in a few moments. for now, let's look at the benefits of being interposed on memory accesses.

## Benefits of Virtual Memory

### Programmability ("transparency")

* Programs use addresses like `0`, `0x200000`, etc. (see example
above)
* Three benefits, at least:
1. Program *thinks* it has lots of memory, organized in a contiguous space
1.  Programs can use "easy-to-use" addresses like 0, 0x20000, whatever. compiler and linker don't have to worry about where the program actually lives in physical memory when it executes.
1. Multiple instances of some program foo are each loaded, each thinks its using memory addresses like 0x50000, whatever, but of course they're not using the same physical cells in RAM

### Protection

* Processes cannot read or write each other's memory
* This protection is at the heart of the isolation among
processes that is provided by the OS
	* Prevents bug in one process from corrupting another process. (non-adversarial scenarios)
    * Don't even want a process to observe another process's memory (like if that process has secret or sensitive data). (adversarial scenarios)
* The idea is that: **If you cannot name something, you cannot use it.** this is a deep idea.


{: .question }
>
> can you think of another example of this naming idea?
>
><details markdown="block">
><summary>Answer </summary>
> file descriptor
></details>


### Effective use of resources

* Programmers don't have to worry that the sum of the memory consumed by all active processes is larger than physical memory.

### Sharing

* Processes share memory under controlled circumstances,
but that physical memory may show up at very different
virtual addresses
* That is, two processes have a different way to refer
to the same physical memory cells
    * how is this translation implemented?
       * software(OS)-hardware(MMU) co-design
       * in modern systems, hardware does it. this hardware is
       configured by the OS.
       * this hardware is called the MMU, for memory management unit,
       and is part of the CPU
       * why doesn't OS just translate itself? 
            * similar to asking why we don't execute programs by running them on an emulation of a processor (too slow)
    * things to remember in what follows:
        * OS is going to be setting up data structures that the hardware sees
        * these data structures are *per-process* 

# 2. Bit manipulation refresher

* 0s and 1s are basics of computer
* Hexadecimal numbers (or hex numbers)
	* integer with base of 16
	* an example: `0x123456789abcdef`
	* other examples: `0xdeadbeef`, `0xbebeebee`
* binary vs. hex
	* `0000` == `0x0`
	* `1111` == `0xf`
* why '`0x`'' for hex?
	* short answer: a convention, an arbitrary choice. See: [https://stackoverflow.com/questions/2670639/why-are-hexadecimal-numbers-prefixed-with-0x](https://stackoverflow.com/questions/2670639/why-are-hexadecimal-numbers-prefixed-with-0x)
* 32-bit vs. 64-bit CPU
	* the length of a memory address

{: .question }
>
> how much memory can a 32-bit CPU address?
>
><details markdown="block">
><summary>Answer </summary>
> 2^32 = 4GB
></details>


* get comfortable mapping between "number of bits" and "size of the space". (will see them soon)
	 * 5 bits => 32 different numbers (size of the space)
	 * 10 bits => 1024 different numbers

|---|---|---|  
| 2^10| kilo| ~1000 |
| 2^20| mega| ~1 million |
| 2^30 | giga | ~1 billion |
| 2^40 | tera | ~1 trillion |
| 2^50 | peta | ~1 quadrillion |
| 2^60 | exa | |


# 3. Paging

##   A. Intro

* Basic concept: divide all of memory (physical and virtual)
into *fixed-size* chunks.

    * these chunks are called *PAGES*.
    * they have a size called the PAGE SIZE. (different hardware architectures specify different sizes)
    * in the traditional x86, the PAGE SIZE will be {%raw%}4096 B = 4KB = $$2^{12}$${%endraw%}


* Warm-up:

{: .question }
>
> How many pages are there on a 32-bit architecture?
>
><details markdown="block">
><summary>Answer </summary>
> {%raw%}$$2^{32} \text{bytes} / (2^{12} \text{bytes/page}) = 2^{20} \text{pages}$${%endraw%}
></details>

{: .question }
>
> What about if there are 48 bits used to address memory?
>
><details markdown="block">
><summary>Answer </summary>
> {%raw%}$$2^{48} \text{bytes} / (2^{12} \text{bytes/page}) = 2^{36} = 64 \text{billion pages}$${%endraw%}
></details>


* Each process has a separate mapping
  * And each page separately mapped
  * we will allow the OS to gain control on certain operations
    * Read-only pages trap to OS on write (store)
    * Invalid pages trap to OS on read (load) or write (store)
    * OS can change mapping and resume application
* Pages have **NUMBERS**. 
```
--page 0:   [0,4095]
--page 1:   [4096, 8191]
--page 2:   [8192, 12277]
--page 3:   [12777, 16384]
...
--page 2^{20}-1 [ ..., 2^{32} - 1]
```

* _Both_ virtual and physical pages having numbers.
* sometimes we will try to be clear with terms like:
	* ***VPN***: virtual page number
    * ***PPN***: physical page number

The "math" of virtual memory: 
* get comfortable mapping between "number of bits required to represent something" and "size of the space". The latter is two-raised-to-the-power-of-the-former.
    
Examples: 

* A virtual address is 32 bits, means the virtual address space is {%raw%}$$2^{32} =4 \text{GB}$${%endraw%} 
* If the VPN is 20 bits, means there are {%raw%}$$2^{20}$${%endraw%}  virtual pages, and the offset is 12 bits, which means page size of {%raw%}$$2^{12}$${%endraw%}, or 4KB.


#### Practice Problem: Number of Possible Virtual Address 

Complete the table, filling in missing entries and replacing question marks with the right values. 

K=2^{10} (kilo), M = 2^{20} (mega), G = 2^{30} (giga), T = 2^{40} (tera), P=2^{50} (peta), E = 2^{60} (exa). 

| Number of virtual address bits (n) | Number of virtual addresses (N)| Largest possible virtual address  |
|----| ---|---|
|8 |____ | ___ | 
|____ |2^? = 64K | ___ | 
|____ | ____ | 2^{32}-1 =?G - 1 | 
|____ |2^? = 256T | ___ |
|64 |____ | ___ |


<details markdown="block">
<summary>Answer </summary>


| Number of virtual address bits (n) | Number of virtual addresses (N)| Largest possible virtual address  |
|----| ---|---|
|8 |2^8 = 256 | 2^8 -1 = 255 | 
|16 |2^16 = 64K | 2^16 - 1 = 64K -1 | 
|32 | 2^32 = 4G | 2^{32}-1 =4G - 1 | 
|48 |2^48 = 256T | 2^48 -1 = 256T -1 |
|64 |2^64 = 16,384P | 2^64 -1 = 16,384P - 1 |


</details>



## B. Key data structure: the Page Table


* A page table conceptually implements a map from  VPN --> PPN
    * A page table is conceptually an index
* ***NOTE***: VPN and PPN need not have the same number of bits (our example shows this)

* the address is broken up into bits:
```
         [.............|........]

         [ VPN         | offset ]
            |             |
            |             +
            |             |
            --> TABLE --> PPN
                           =
                         address
```

* The top (higher-order) bits index into the page table. 
    * The contents at that index of the PT are the PPN.
* The bottom (lower-order) bits are the ***offset***. 
    * These are not impacted by the mapping
* The physical address (PA) = PPN + offset
	* (note: "+" here means "concatenate": for example, 123 "+" 456 => 123456)
* Result is that each page table entry expresses a mapping about a contiguous group of addresses.
* Another way to look at it:
    (assume 48-bit addresses and 4KB pages)
    * there is in the sky a {%raw%}$$2^{36}$${%endraw%} sized array that maps the virtual address to a *physical* page
    * `table[36-bit virtual page number] = 20-bit physical page #`
    * "Just a giant Lookup Table!"

#### EXAMPLE: 

If the OS wants a program to be able to use address `0x00402000` to refer to physical address `0x00003000`, then the OS conceptually adds an entry:

    table[0x00402] = 0x00003

(this is the 1026th virtual page being mapped to the 3rd physical page.). in decimal: table[1026] = 3

**NOTE:** The top 36 bits are doing the indirection. The bottom 12 bits just figure out where on the page the access should take place.



{: .question }
>
> Do offset and page size have anything to do with each other?
>
><details markdown="block">
><summary>Answer </summary>
> yes. The page size decides the offset bits:
>        if page size is 4KB, then offset needs to be 12bit;
>        if page size is 16KB, then offset needs to be 14bit;
></details>

{: .question }
>
> Can a VA `0x123456` be mapped to PA `0xab9876` in i7 (page size=4KB)?
>
><details markdown="block">
><summary>Answer </summary>
> no; the offset cannot be changed between VA and PA
></details>


{: .question }
>
> Can a VA `0x123456` be mapped to PA `0xab9876`?
>
><details markdown="block">
><summary>Answer </summary>
> yes, if the page size <= 16B.
></details>

* Now all we have to do is create this mapping
	* Why is this hard? why not just create the mapping?
        * How large is the table if we do this?
    * You need one table, per process, roughly 512GB (2^{36} entries * 8 bytes per entry).
    * [why 8 bytes per entry? in practice, it's convenient to have the entry size be the same as a data type on the machine]
* That's too big!  let's deal with this...

<img src="../../assets/images/notes/week7/virtualoverview.png" alt="---" width="600"/>


Some terminology:

* A ***virtual address*** (VA) is composed of a ***virtual page number*** (VPN) and a ***page offset*** (VPO/PPO)
* If the page size is 2^P bytes, then the least significant P bits of the virtual address are the page offset, the rest of the bits from the virtual page number 
* The page table (PT) is an array of ***page table entries*** (PTE)
    * One PTE corresponds to each virtual memory page
* When translating a VA to a pyhsical address (PA), the PTE corresponding to the VA is located 
    * ...by indexing into the page table using the virtual page number as the index
    * The PTE contains a ***valid bit*** and a ***physical page number***, and possibly a ***dirty bit***
        * The valid bit indicates whether the page is currently located in main memory
        * Basically, we usually have some extra bits along with the PPN, which we'll get into more details about later
            * These extra bits allow us to be more efficient with storage
    * If valid, the physical page number is concatenated with the page offset from the virtual address to form the physical address corresponding to the original virtual address 
* Each PTE needs to accommodate the physical page number, the valid bit, and a few other bits
    * Usually a PTE fits into 32 bits but not 16 bits
    * Thus, assume each PTE is represented by a 4-byte longword. 
    * To locate the relevant PTE, the virtual page number is multiplied by 4 and added to the *page-table address*, which is typically kept in a processor register


#### Practice Problem: Number of Page Table Entries (PTEs)

Determine the number of PTEs that aare needed for the following combinations of virtual address size (n) and page size (P). 

| n | P= 2^p| Numer of PTEs |
|----| ---|---|
|16 |4 KB | ___ | 
|16 |8 KB | ___ | 
|32 |4 KB | ___ | 
|32 |8 KB | ___ | 


<details markdown="block">
<summary>Answer </summary>


Each virtual page is P = 2^p bytes, so there are a total of 2^n/2^p = 2^{n-p} possible pages, and each page needs a page table entry (PTE). 

| n | P= 2^p| Numer of PTEs |
|----| ---|---|
|16 |4 KB | __16_ | 
|16 |8 KB | 8 | 
|32 |4 KB | 1M | 
|32 |8 KB |512 K | 

</details>

#### Practice Problem: Bit Requirements 

Given a 32-bit virtual address space and a 24-bit physical address, determine the number of bits in the VPN, VPO, PPN, and PPO for the following page sizes P: 

| P | VPN bits | VPO bits | PPN bits | PPO bits | 
| ---|---|----|----|----|
|1 KB | ___ | ___ | ___ | ___ |
|2 KB | ___ | ___ | ___ | ___ |
|4 KB | ___ | ___ | ___ | ___ |
|8 KB | ___ | ___ | ___ | ___ |

<details markdown="block">
<summary>Answer </summary>

This is an important problem to understand to fully grasp address translation. 

For the first subproblem: 
* We are given n = 32 virtual address bits and m=24 physical address bits. 
* A page size of P=1KB means we need log_2(1K) = 10 bits for both the VPO and PPO. 
* The remaining bits are the VPN and PPN. 

| P | VPN bits | VPO bits | PPN bits | PPO bits | 
| ---|---|----|----|----|
|1 KB | __22_ | _10__ | _14__ | _10__ |
|2 KB | _21__ | _11__ | _13__ | 11 |
|4 KB | __20_ | _12__ | _12__ | 12 |
|8 KB | _19__ | _13__ | _11__ | 13 |

</details>


##  C. Multilevel Page Tables

* The key idea: represent the page table as a tree ...
    * root node has pointers to other nodes
    * children point to pages
    * Then, we map addresses by using the root for the uppermost address bits, the next level for the next address bits, etc.
* The tree is sparse 
    * that is, many of the child nodes are never filled in
    * just like a real tree need not be complete
    * only fill in the parts that are actually "in use"
* example:
    * Say we want to map 2MB of physical memory at virtual memory 0,...,2^{21}-1

    48 bits:
```
        9 9 9 9 (VPN) | 12 (offset)
```
bottom one, points to physical pages.

**NOTE**: This is an enormous address space, but we've used very few physical resources -- just 512 + 4 physical pages 
* (why? because page table also consumes memory)

* another way to understand this:
    * look at the bottommost level; that's a page table.
    * the rest of the structure is telling the architecture how to find the page table
* sometimes you get asked: "what piece of the address space is described by a given page table entry?"
    * to answer that, look at how many bits are "left"

#### Practice Problem: 2-level Multilevel Page Table

Suppose a machine with 32-bit virtual addresses and 40-bit physical addresses is designed with a 2-level page table, subdividing the virtual address into 3 pieces as follows:

* 10-bit page table number 
* 10-bit page number
* 12-bit offset

The first 10 bits are the index into the top-level page table, the second 10 bits are the index into the second-level page table, and the last 12 bits are the offset into the page. There are 4 protection bits per page, so each page table entry takes 4 bytes. 

* What is the page size in this system? 
* How much memory is consumed by the first and second level page tables, and wasted by internal fragmentation for a process that has 64K of memory starting at address 0? 
* How much memory is consumed by the first and second level page tables and wasted by internal fragmentation for a process that has a code segment of 48K starting at address 0x1000000, a data segment of 600K starting at address 0x800000000, and a stack segment of 64K starting at address 0xf0000000 and growing upward (towards higher addresses)? 

## D. Alternatives and tradeoffs

* There are some tradeoffs:
	* between large and small page sizes:
	* large page sizes means wasting actual memory
	* small page sizes means lots of page table entries (which may or may not get consumed)
	* between many levels of mapping and few:
		* more levels of mapping means less space spent on page structures when address space is sparse (which they nearly always are) but more costly for hardware to walk the page tables
		* fewer levels of mapping is the other way around: need to allocate larger page tables (which cost more space), but the hardware has fewer levels of mapping
* new address translation data structure? (instead of page table)

#### Practice Problem: Counting Multilevel Page Tables

{: .question }
>
> If we need to map 2GB memory, how many PT pages do we need?
>
><details markdown="block">
><summary>Answer </summary>
> *  2GB = 1024 * 2MB = 1024* 512 * 4KB
> * the last level PT (L4) have 1024 pages
> * the 2nd last level has 2 pages
> * the first two levels have 1 page each
> * in total, we need 1028 PT pages (= 1 + 1 + 2 + 1024)
></details>


#### Practice Problem: Counting Multilevel Page Tables

{: .question }
>
> If the PT pages are fully mapped (meaning mapped 2^(48) Bytes memory), how large would the PT be?
>
><details markdown="block">
><summary>Answer </summary>
>
>      1 + 512 + 512^2 + 512^3
>     (L1) (L2)  (L3)    (L4)
>
> Notice that the size of the L4 PT is equivalent to the "array" (in the sky) we talked about last time.
>
> The point is that if memory are fully mapped,  multi-level PT doesn't save memory. But, it is very unlikely this can happen, whereas in normal case multi-level PT helps.
></details>

# 2. x86-64: addresses

* x86 architecture is 64-bits. 
    * registers and addresses are 64-bits wide

## VIRTUAL ADDRESSES in x86-64

* On currently-available x86-64 machines, only 48 bits "matter". (conclusion: not all 64-bit patterns correspond to meaningful virtual addresses)
    * Bit patterns that are valid addresses are called _canonical addresses_. 
    * Canonical address has all 0s or all 1s in the upper 16 bits (bits 63 through 48). Has to match whatever bit 47 is. [see 3.3.7.1 in the Intel software developer's manual]
    * Result: address space is 2^{48} = 256 TB

[ Another way to look at it:

    The x86-64 architecture divides canonical addresses into two
    groups, low and high. 

    Low canonical addresses range from 
        0x0000'0000'0000'0000 to
        0x0000'7FFF'FFFF'FFFF.

    High canonical addresses range from
        0xFFFF'8000'0000'0000 to
        0xFFFF'FFFF'FFFF'FFFF.

    Considered as signed 64-bit numbers, all canonical addresses
    range between -2^47 and 2^47-1.
]

[Intel 5-level paging:
    --extend virtual addresses from 48 bits to 57 bits
    --increase the addressable memory from 256 TB to 128 PB
    --implemented in the Ice Lake processors, and Linux kernel 4.14
]

## PHYSICAL ADDRESSES in x86-64

* 52 bits
* Question: why 52? see handout panel 3, 4
    * [answer: 40bit (in PTE) + 12bit (page)]
* Means a single machine can address up to 4 PB of physical memory.
    * of course, if the machine only has 16 GB (say), then physical addresses will (roughly speaking) only have 34 bits that matter, and thus the top 18 (=52-34) bits of physical addresses will generally be zero 
* [NOTE: this is a simplification, owing to the "physical memory map"; however, we will not encounter that too much in this class.]

## MAPPING

* have to map 48-bit number (virtual address) to 52-bit number (physical address), at the granularity of ranges of 2^{12}

# 3. x86-64: page table structures

See full handout <a href="../../assets/images/notes/week7/virtual_memory_handout.pdf">here</a>. 
 

<img src="../../assets/images/notes/week7/i7_page_table_translation.png" alt="---" width="600"/>


<img src="../../assets/images/notes/week7/symbols.png" alt="program in memory" width="600"/>

<img src="../../assets/images/notes/week7/i7_page_table_entries.png" alt="program in memory" width="600"/>

<img src="../../assets/images/notes/week7/i7_level4_page_table_entries.png" alt="program in memory" width="600"/>


`%cr3` is the address of the top-level directory (L1 page table)


{: .question }
>
> Is that address a physical address or virtual address?
>
><details markdown="block">
><summary>Answer </summary>
> it is a physical address. hardware needs to be able to follow the page table structure.
></details>

 ** An example:
    [walk through the handout]

What if OS wants to map a process's virtual address  `0x0202000`  to physical address `0x3000` and make it accessible to user-level but read-only?

what do the page structures look like?

solution:

* take off the bottom 12 bits of offset

            vpn = 0x0202.

* write it out in bits:

             0....0    000000001  000000010
              18 0
              bits

        L1 (0th entry) --> L2 (0th entry) -->


            L3
         ...........
         ...........
         ...........
         ........... [entry 1]
         ...........


                             PGTABLE
                   <40 bits>
                |0x00'0000'0003 | U=1,W=0,P=1|   [entry 2]
                |               |            |   [entry 1]
                |               |            |   [entry 0]
                ______________________________


[see Intel reference manual for more.
[Intel 64 and IA-32 Architectures Software Developer's Manual, Volume 3a](https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-vol-3a-part-1-manual.pdf)
]

 * PTE (Page Table Entry) holds a bunch of bits including: 
    * dirty (set by hardware)
    * acccessed (set by hardware)
    * present (set by OS)
    * cache disabled (set by OS)
    * write through  (set by OS)


{: .question }
>
> What will happen if the present bit is 0 but a program accesses the memory?
>
><details markdown="block">
><summary>Answer </summary>
> page fault; we will study it later.
></details>


What do the U/S and R/W bits do?
* are these for the kernel, the hardware, what?
* who is setting them? what is the point?
    (OS is setting them to indicate protection; hardware is enforcing them)
* what if the permission is violated?
    * Page fault! 


* Large pages:
    * Can get 2MB (resp, 1 GB pages) on x86: each L3 (resp, L2) page table now points to the page instead of another page table
        + page tables smaller, less page table walking
        - more wasted memory
* to enable this, set bit 7 (`PS`) bit
    * example: set bit `PS` in L3 table
        * result is 2MB pages
        * page walking is L1, L2, L3; no L4 page tables


# 4. Practice

## A: memory that different PTEs can address

- Question: how much memory can one L1 page entry address?

- answer: each entry in the L1 page table corresponds to 512GB of
virtual address space ("corresponds to" means "selects the
next-level page tables that actually govern the mapping").

for others:

- each entry in the L2 page table corresponds to 1 GB of virtual
address space

- each entry in the L3 page table corresponds to 2 MB of virtual
address space

- each entry in the L4 page table corresponds to 1 page (4 KB)
of virtual address space

- Question: so how much virtual memory is each L4 page *table*
responsible for translating? 4KB? 2MB? 1GB? 

- [answer: 2MB]

- each page table itself consumes 4KB of physical memory, i.e., each one of these fits on a page


## B. Allocating memory

[from cs61, 2018]
    https://cs61.seas.harvard.edu/site/2018/Section4/

What is the minimum number of physical pages required on x86-64 to allocate the following allocations? 

Draw an example pagetable mapping for each scenario (start from scratch each time): 

* 1 byte of memory = [5 phys pages]
* 1 allocation of size 2^12 bytes of memory = [5 phys pages]
* 2^9 allocations of size of 2^12 bytes of memory each = [512 + 4 = 516 phys pages]
* 2^9 + 1 allocations of size of 2^12 bytes of memory each = [512 + 4 + (1 + 1) = 518 phys pages]
* 2^18 + 1 allocations of size 2^12 bytes of memory each = [1 (L1) + 1 (L2) + 2 (L3) + (2^9 + 1) (L4) + (2^18 + 1) (the memory)]


## C. page table walk

x86 page table: translate a VA to PA

Practice:

- This is the standard x86 32-bit two-level page table structure (not x86-64; we use 32-bit for simplicity).
- The permission bits of page directory entries and page table entries are set to 0x7.
	* (what does 0x7 mean? 
    * answer: page present, read-write, and user-mode; see handout week10.a (today's) [TODO(ahs):  references the Core i7 Page Table Translation]
    	* This means that the virtual addresses are valid, and that user programs can read (load) from and write (store) to the virtual address.)

- The memory pages are listed below.
   On the left side of the pages are their addresses.
   (For example, the address of the "top-left" memory block (4 bytes) is
   0xf0f02ffc, and its content is 0xf0f03007.)
```
  %cr3:  0xffff1000

              +------------+            +------------+ 
  0xf0f02ffc  | 0xf00f3007 | 0xff005ffc | 0xbebeebee | 
              +------------+            +------------+ 
              | ...        |            | ...        | 
              +------------+            +------------+ 
  0xf0f02800  | 0xff005007 | 0xff005800 | 0xf00f8000 | 
              +------------+            +------------+ 
              | ...        |            | ...        | 
              +------------+            +------------+ 
  0xf0f02000  | 0xffff5007 | 0xff005000 | 0xc5201000 | 
              +------------+            +------------+ 


              +------------+            +------------+
  0xffff1ffc  | 0xd5202007 | 0xffff5ffc | 0xdeadbeef |
              +------------+            +------------+
              | ...        |            | ...        |
              +------------+            +------------+
  0xffff1800  | 0xef005007 | 0xffff5800 | 0xff005000 |
              +------------+            +------------+
              | ...        |            | ...        |
              +------------+            +------------+
  0xffff1000  | 0xf0f02007 | 0xffff5000 | 0xc5202000 |
              +------------+            +------------+
```

- What's the output of the following C excerpt?

```c
   int *ptr1 = (int *) 0x0;
   printf("%x\n", *ptr1);

   // this will be your homework
   // int *ptr2 = (int *) 0x200ffc;
   // printf("%x %x\n", *ptr1, *ptr2);
```

[Note: `%x` in `printf` means printing out the integer in hexadecimal format.]

Answer: "`0xc5202000`"

In particular, here is walking the page tables:

```
  0x0 => [0][0][0] (10bit, 10bit, 12bit)
    [note: in x86-64, 0x0 will be organized as [9bit, 9bit, 9bit, 9bit, 12bit])

  (%cr3) -> 0xffff1000 (L1 PT) 
            +--[index:0]-> 0xf0f02000 (L2 PT) 
                           +--[index:0]-> 0xffff5000 (data page) + 0 (offset)
                                          +--[PA]-> 0xffff5000
```

* The content of PA `0xffff5000` is "`0xc5202000`"
* Why "content"?
	* because C code "`*ptr1`" means _dereferencing_ the pointer "`ptr1`", namely fetching the memory content pointed by "`ptr1`" (pointer = an address).

- note: all addresses in this process are physical addresses.

# Next Time

More on the x86-64 architecture, making Virtual Memory more performant using TLBs. 

[Acknowledgments: Mike Walfish, David Mazieres, Mike Dahlin]