---
title: 'Notes Week 8: Virtual Memory-- x86, Multi-level, TLBs'
layout: page
summary: 'VM in x86, more multi-level examples, and the TLB'
---

<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@4/tex-mml-chtml.js"></script>

# Week 8: More on Virtual Memory   
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

1. Lab 3 reviews!
1. More on Virtual Memory
    1. Multi-level page table
    1. Addresses in x86-64/32
    1. If enough time, TLB 
1. Lab 4 questions/discussion

## Announcements

* Next week is Spring Break-- no class
    * I'll double-check to make sure assignments are NOT due during break
* Mar 11, class is async
    * I'll record a lecture/mini-lectures and make them available the first week of March
    * I'll be around Mar 10 and Mar 12 to answer any questions 


## Weekly Summary and Where are we?

### Topics

* **Last Week:** Intro to Virtual Memory
* **This Week:** More about Multi-Level Page Tables, Addresses in x86
* **Next TIME:** TLB, Page Faults


# Last Week: 

* Intro to Virtual Memory

* Left off at Multi-level page tables

* Review:

    * Virtual Memory
        * +--> Paging
            * +--> multilevel page table
                * +--> x86-64 page table

* Virtual Memory: VA --> PA
* page table conceptually implements a map from VPN --> PPN
    * NOTE: VPN and PPN need not (and do not, in our case study) have the same number of bits
    * top bits index into page table. contents at that index are the PPN.
    * bottom bits are the offset. not changed by the mapping
* physical address = PPN + offset



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

[from cs61, 2018](https://cs61.seas.harvard.edu/site/2018/Section4/)

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
    * what does 0x7 mean? 
        * answer: page present, read-write, and user-mode; [see handout](../../assets/images/notes/week7/virtual_memory_handout.pdf)
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

# 5. TLB

- so it looks like the CPU (specifically its MMU) has to go out
to memory on every memory reference?
    - called "walking the page tables"

- Question: to finish one memory access (e.g., `movq (0xbebeebee), %rax)`, how many physical pages CPU (or MMU) has to touch?
	* [answer: 5 (assuming the instruction is already fetched) 4 for L1/2/3/4 page tables, and 1 for the data page]

- performance-wise, this is awful. to make this fast, we need a cache

- TLB: translation lookaside buffer

* hardware that stores virtual address --> physical address;
the reason that all of this page table walking does not slow
down the process too much

- Who control the TLB?
    - hardware managed? (x86, ARM.) hardware populates TLB
    - software managed? (MIPS. OS's job is to load the TLB when
    the OS receives a "TLB miss". Not the same thing as a page
    fault.)

    - TLB is one type of cache
      *  Crash course of CPU caches
      [see today's handout]

       common parameters:
         * cache line size (usually, 64B for x86)
         * 2^s sets  (s is the number of bits in addresses to reference sets)
         * E-way     (number of cache line in each set)
           (for example, 8-way means that there are 8 cache lines in one set)

    - given an adress, split it into:
         | tag | index | offset |

      - the index is going to pick the "sets"
      - offset is going to choose bytes within one cache line
      - tag is used to compared if cache hit

- TLB structures
   - there are instruction TLB, data TLB, and shared TLB
   - also has 4KB page translation and large page (2MB) translation

   * data TLB that your computer might use:
      4 KB page: 64 entries; 4-way set associative
      [this is handout's TLB]

Question: if TLB is full, how much memory's VA translation has been cached?
[`64*4KB = 256KB`
     this means if your program is smaller than 256KB, after warming-up,
     your program likely will never encounter instruction TLB miss!!
    ]

 [ TLB Sizes (for those who are interested)

    instruction TLB:
      4KB page: 128 entries; 8-way set associative
      2 MB page: 8 entries; fully associative

    data TLB:
      4 KB page: 64 entries; 4-way set associative
      2 MB page: 32 entries; 4-way set associative
      1G page: 4 entries; 4-way associative

    shared TLB:
      4 KB + 2 MB page: 1536 entries; 12-way set associative
      1 GB page: 16 entries; 4-way set associative

   see also Intel Skylake:
    https://en.wikichip.org/wiki/intel/microarchitectures/cascade_lake
]

- questions about page faults vs. TLB misses:
  - recall page faults:
    - access invalid memory (P=0) 
    - or fail permission checks (write a RO page)
  - does TLB miss imply page fault? (no!)
  - does page fault imply TLB miss? (no!)
      (imagine a page that is mapped read-only. user-level
      process tries to write to it. TLB knows about the mapping,
      so no TLB miss. But this is still a protection violation.
      To cut down on terminology, we will lump this kind of
      violation in with "page fault".)

- x86:
    - Question: what happens to the TLB when %cr3 is loaded?
      does kernel need to remove all the TLB entries?
      [answer: yes; called flushing TLB]
    - can we flush individual entries in the TLB otherwise? 
      [yes, INVLPG addr]
    - Question: should TLB also cache R/W and U/S bits in PTE?
      [Yes! Otherwise, the CPU are unable to enforce isolation and
    permissions.]


# 6. Where does the OS live?

First, kernel vs. application

- two modes, many names
	- "user mode" and "kernel/supervisor mode"
	- "ring 0" and "ring 3"
	- "restricted mode" and "privileged mode"

- How CPU differs the two modes?
	* [answer: by two bits (called CPL) in a register (code selector register, CS).
        if CPL=0, then the code running is in "kernel mode"/"ring 0";
        if CPL=3, then in "user mode"/"ring 3". 

Also, CPL automatically changes when system call instructions (sysenter, sysexit) are called.]


- What are the differences between the two modes?
	- memory access to pages with U/S bit set to 0
	- read/write registers (like %cr3)
	- privileged instructions (for example, shutdown the interrupt, I/O instructions)

[if you want to know more about CPU modes, read:
    https://sites.google.com/site/masumzh/articles/x86-architecture-basics/x86-architecture-basics]

Question: Where does the OS live? 

* Option 1: In its own address space?

    - Can't do this on most hardware (e.g., syscall instruction
    won’t switch address spaces)

    - Also would make it harder to parse syscall arguments
    passed as pointers

* Option 2: kernel is actually in the same address space as
  all processes (choice of real systems)

[see handout for picture]

* not precisely true post-Meltdown, but close enough (in that some of the kernel is mapped into all user processes).

	- Use protection bits to prohibit user code from reading/writing kernel

    - Typically all kernel text, most data at same VA in *every*
    address space (every process has virtual addresses that map to the
    physical memory that stores the kernel's instructions and data)

    - In Linux, the kernel is mapped at the top of the address space,
    along with per-process data structures.

    - Physical memory also mapped up top, which gives the kernel a convenient way to access physical memory.

NOTE: that means that physical memory that is in use is mapped in at least two places (once into a process's virtual address space and once into this upper region of the virtual space).



# 7. Page faults

##   A. intro and mechanics

We've discussed these a bit. Let's go into a bit more detail...

Concept:

* a reference is illegal, either because it's not mapped in the page tables or because there is a protection violation.
* requires the OS to get involved
* this mechanism turns out to be hugely powerful, as we will see.

Mechanics

- what happens on the x86?
    - processor constructs a trap frame and transfers execution to an interrupt or trap handler
        * See handout

       [trap frame]
%rsp --> [error code]

%rip now points to code to handle the trap
    [how did processor know what to load into %rip?
    Answer: kernel needs to set up an interrupt descriptor table (IDT),
    by which CPU knows where to go; btw, page fault exception is #14.]

* Question: why isn't %cr3 stored in the trap frame?
    * [answer: though trapping to kernel, the address space is unchanged. Recall "where does the OS live".]

```
    error code:

        [ ................................ U/S | W/R | P]

        U/S: user mode fault / supervisor mode fault
        R/W: access was read / access was write
        P: not-present page / protection violation
```

* on a page fault, %cr2 holds the faulting virtual address

* Question: why does OS need the faulting virtual address?
    * [answer: to walk page table to fix the page fault]
    * intent: when page fault happens, the kernel sets up the process's page entries properly, or kills the process

##   B. Uses of page faults

- **Best example:** overcommitting physical memory (the classical
use of "virtual memory")
    - your program thinks it has, say, 64 GB of memory, but your hardware has only 16 GB of memory
    - the way that this worked is that the disk was (is) used to store memory pages
    - *advantage:* address space looks huge
    - *disadvantage:* accesses to "paged" memory (as disk pages that live on the disk are known) are sllooooowwwww:

- Rough implementation:
    - on a page fault, the kernel reads in the faulting page
    - QUESTION: what is listed in the page structures? how does kernel know whether the address is invalid, in memory, paged, what?
    - kernel may need to send a page to disk (under what conditions? answer: two conditions must hold for kernel to HAVE to write to disk)
    	* (1) kernel is out of memory
    	* (2) the page that it selects to write out is dirty

- Computers have lots of memory, so less common to hear the sound of swapping these days. You would need multiple large memory consumers running on the same computer.
    - in fact, today you sometimes experience the other way around: you may want to run file system in memory (ramfs) for performance.

- Many other uses
    * a) store memory pages across the network! (Distributed Shared Memory)
    - basic idea was that on a page fault, the page fault handler went and retrieved the needed page from some other machine
        - charming idea, but impractical. Why? too expensive.
        - classic trade-off: transparency vs. efficiency
    - a general case: low-level APIs expose more information (less transparency) but have better performance

    * b) copy-on-write
    - when creating a copy of another process, don't copy its memory. just copy its page tables, mark the pages as read-only
        - COW is how fork is implemented
        - QUESTION: do you need to mark the parent's pages as read-only as well?
            - [answer: of course, otherwise the child will see data changes from the parent.]
        - program semantics aren't violated when programs do reads
    - when a write happens, a page fault results. at that point, the kernel allocates a new page, copies the  memory over, and restarts the user program to do a write
        - then, only do copies of memory when there is a fault as a result of a write
        - this idea is all over the place; used in `fork()`, `mmap()`, etc.


## C. costs

- What does paging from the disk cost?
    - let's look at average memory access time (AMAT)
    - `AMAT = (1-p)*memory access time + p * page fault time`, where p is the prob. of a page fault.
	   * memory access time ~ 100ns
	   * SSD access time   ~ 1 ms = 1^6 ns
    - QUESTION: what does p need to be to ensure that paging hurts performance by less than 10%?

1.1*t_M = (1-p)*t_M + p*t_D
p = .1*t_M / (t_D - t_M) ~ 10^1 ns / 10^6 ns = 10^{-5}

* so only one access out of 100,000 can be a page fault!!
	- basically, page faults are super-expensive (good thing the machine can do other things during a page fault)

* Concept is much larger than OSes: need to pay attention to the slow case if it's really slow and common enough to matter.


## D. Page replacement policies

* the fundamental problem/question: (also known as *cache eviction problem*)
* some entity holds a cache of entries and gets a cache miss. The entity now needs to decide which entry to throw away. How does it decide?
* make sure you understand why page faults that result from "page-not-present in memory" are a particular kind of cache miss
    - (the answer is that in the world of virtual memory, the pages resident in memory are basically a cache to the backing store on the disk; make sure you see why this claim, about virtual memory vis-a-vis the disk, is true.)
    - the system needs to decide which entry to throw away, which calls for a *replacement policy*.
    - let's cover some policies....

* Specific policies
	* FIFO: throw out oldest (results in every page spending the same number of references in memory. not a good idea.  pages are not accessed uniformly.)
	* LRU: throw out the least recently used (this is often a good idea, but it depends on the future looking like the past. what if we chuck a page from our cache and then were about to use it?)

* implementing LRU
	- reasonable to do in application programs like Web servers that cache pages (or dedicated Web caches). [use queue to track least recently accessed and use hash map to implement the (k,v) lookup]
    - in OS, LRU itself does not sound great. would be doubling memory traffic (after every reference, have to move some structure to the head of some list)
    - and in hardware, it's way too much work to timestamp each reference and keep the list ordered (remember that the TLB may also be implementing these solutions)
    - how can we approximate LRU?
    - another algorithm:
        * CLOCK

        [draw clock on board]

        - arrange the slots in a circle. hand sweeps around, clearing
        a bit. the bit is set when the page is accessed. just evict a
        page if the hand points to it when the bit is clear.

        - approximates LRU ... because we're evicting pages that haven't
        been used in a while....though of course we may not be evicting
        the *least* recently used one (why not?)

* Summary:
    - LRU is usually a good approximation to optimal policy
        * optimal policy: throw away the entry that won't be used for the longest time. this is optimal.
    - Implementing LRU in hardware or at OS/hardware interface is a pain
    - So implement CLOCK ... decent approximations to LRU, which is in turn good approximation to OPT *assuming that past is a good predictor of the future* (this assumption does not always hold!)


## E. Thrashing

The points below apply to any caching system, but for the sake of concreteness, let's assume that we're talking about page replacement in particular.

* What is thrashing?
	* Processes require more memory than system has
	* Specifically, each time a page is brought in, another page, whose contents will soon be referenced, is thrown out

* Example:
	* one program touches 50 pages (each equally likely); only  have 40 physical page frames 
	* If we have enough physical pages, 100ns/ref 
	* If we have too few physical pages (40 pages), assume every 5th reference leads to a page fault 

* Question: if the SSD latency is 1ms, how many times slowdown do we have?
	* 4refs x 100ns  and 1 page fault x 1ms for SSD I/O 
	* this gets us 5 refs per (1ms + 400ns) ~ 0.2ms/ref = 2,000x slowdown!!! 
	* **What we wanted:** virtual memory the size of disk with access time the speed of physical memory 
	* **What we have here:** memory with access time roughly of SSD (0.2 ms/mem_ref compare to 1 ms/SSD_access)
	* As stated earlier, this concept is much larger than OSes: need to pay attention to the slow case if it's really slow and common enough to matter.


* Reasons/cases:
	* (1) process doesn't reuse memory (or has no temporal locality)
	* (2) process reuses memory but the memory that is absorbing most of the accesses doesn't fit.
	* (3) individually, all processes fit, but too much for the system

* what do we do?
	* well, in the first two reasons above, there's nothing you can do, other than restructuring your computation or buying memory (e.g., expensive hardware that keeps entire customer database in RAM)
* in the third case, can and must shed load. how?
    * two approaches:
    	* a. working set
    	* b. page fault frequency

    * a. working set
    	* only run a set of processes s.t. the union of their working sets fit in memory
    	* definition of working set (short version): the pages a process has touched over some trailing window of time

    * b. page fault frequency
		* track the metric (# page faults/instructions executed)
		* if that thing rises above a threshold, and there is not enough memory on the system, swap out the process


[Acknowledgments: Mike Walfish, David Mazieres, Mike Dahlin]