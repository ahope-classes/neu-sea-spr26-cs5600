---
title: 'Notes Week 9: Intro to IO'
layout: page
summary: '...'
---

<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@4/tex-mml-chtml.js"></script>

# Week 9: IO  
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

1. ...

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




# 1. Last time

  -  page faults
    -  two types: not present & violating permissions
    -  page fault handler
       (another case of OS configures and hardware executes)
    -  very versatile
    *  the most classic usage: overcommitting memory

# 2. mmap

* a syscall; a cool way to bring some ideas together.

* recall some syscalls:

  fd = open(pathname, mode)
  write(fd, buf, sz)
  read(fd, buf, sz)

* we've learned fds before, but what's an fd in kernel?
* indexes into a table maintained by the kernel on behalf of the process

* syscall:
    [see slides]

void* mmap(void* addr, size_t len, int prot, int flags,
       int fd, off_t offset);

* means, roughly, "map the specified open file (fd) into a
region of my virtual memory (close to addr, or at a kernel-selected
place if addr is 0), and return a pointer to it"


NOTE: the "disk image" here is the file we've mmap()'ed, not the
process's usual backing store. The idea is that mmap() lets the
programmer "inject" pages from a regular file on disk into the
process's backing store (which would otherwise be part of a swap
file).

* after this, loads and stores to addr[x] are
equivalent to reading and writing to the file at offset+x.

* why is this cool?

    - example: mmap enables copying a file to stdout without
    transferring data to user space
      [see handout from last time, week11a]

        Question: mmap vs. normal read/write, which is faster?
          by how much?

        [Answer: 
         * mmap is faster
         * "by how much" depends on the file size and the underlying machine.
         * for ~1G file on my machine (M1 Macbook Air), mmap is 13.5x faster.
        ]

        NOTE: the process never itself dereferences a pointer to
        memory containing file data.

        NOTE: this saves two sets of memory-to-memory copies
        (kernel-to-user, user-to-kernel), versus the "naive"
        solution of read()ing into a buffer in user space, and then
        write()ing

        [Also, a well-tuned buffer cache manages which file pages
        are kept in RAM, rather than leaving the app developer to
        have to explicitly try to manage that (and potentially have
        the OS page replacement algorithm underneath make
        conflicting decisions).]

    - other examples:

        - reading big files. map the whole thing, rely on the paging
        mechanism to bring the needed pieces into memory as necessary

        - shared data structures, when flag is MAP_SHARED 

        - file-based data structures:
            - load data from file, update it, write it back
            - this is implemented entirely with loads/stores

        Question: how does the OS ensure that it's only writing back
        modified pages?

* how's mmap implemented?! (answer: through virtual memory,
with the VA being addr [or whatever the kernel selects] and
the PA being what? answer: the physical address storing the
given page in the kernel's buffer cache).

* have to deal with eviction from buffer cache, so kernel will need
a data structure that maps from:
    Phys page --> {list of (proc,va) pairs}

note that the kernel needs this data structure anyway: when a
page is evicted from RAM, the kernel needs to be able to invalidate
the given virtual address in the page table(s) of the process(es)
that have the page mapped.

# 3.  I/O architecture

general:
[draw picture: CPU, Mem, I/O, connected to BUS]

* lots of details.
* fun to play with.
* registers that do different things when read vs. written.

  [draw some registers in devices, with status and data]

CPU/device interaction (can think of this as kernel/device
interaction, since user-level processes classically do not
interact with devices directly.

## A. Mechanics of communication

### (a) explicit I/O instructions
[sometimes called port I/O, port-mapped IO (PMIO)]

instructions:
    outb, inb, outw, inw

operands:
    IO address space
     (separate from memory address space)

examples:

(i) reading keyboard input. 

* I/O ports (PS/2):

IO Port  Access Type    Purpose
0x60     Read/Write     Data Port
0x64     Read           Status Register
0x64     Write          Command Register

[https://wiki.osdev.org/%228042%22_PS/2_Controller]

* keyboard keycode

  * handout uses "Scan Code Set 1"
  * differentiate "pressed" and "released" by the most
  significant bit:
    0x1E: A pressed
    0x9E: A released
    (notice: the difference is 0x80,
      namely binary: b10000000)

  Question: how many keys can be mapped to the key code?
  [answer: 128, because a key code is 1B, indicating 256 (2^8)
  key code; and each key has pressed and released; making it
  128 keys.]

  * more keys? a mode change:
   * two bytes received, starting with 0xE0
   * a lot of multimedia keys, like volume up/down
   * see "Scan Code Set 1" for more details (link below)

[https://wiki.osdev.org/PS/2_Keyboard#Scan_Code_Set_1]

an implementation of reading a key stroke:

keyboard_readc(); [see handout]

(ii) setting blinking cursor. see handout

console_show_cursor();

###   (b) memory-mapped I/O

physical address space is mostly ordinary RAM

low-memory addresses (<1MB), sometimes called "DOS compatibility
memory", actually refer to other things. 

You as a programmer read/write from these addresses using
loads and stores. But they aren't "real" loads and stores to
memory. They turn into other things: read device registers,
send instructions, read/write device memory, etc.

    * interface is the same as interface to memory
    (load/store)

    * but does not behave like memory 

+ Reads and writes can have "side effects"

+ Read results can change due to external events 

Example: writing to VGA or CGA memory makes things appear on
the screen.

See handout (last panel):

console_putc()

To avoid confusion: this is not the same thing as
virtual memory. this is talking about the *physical*
address.

    * > is this an abstraction that the OS provides to
    others or an abstraction that the hardware is
    providing to the OS?  [the latter]

Question: the memory layout seems different from what we saw on
"where is OS live".

[notice that this layout is PYSICAL memory address!]

[if you're interested, check out these slides:
 https://opensecuritytraining.info/IntroBIOS_files/Day1_00_Advanced%20x86%20-%20BIOS%20and%20SMM%20Internals%20-%20Motivation.pdf]

### (c) interrupts

[we've seen interrupts many time starting
        from "trapping to kernel" to "scheduling"]

### (d) through memory: both CPU and the device see the same memory,
so they can use shared memory to communicate.

* --> usually, synchronization between CPU and device requires
lock-free techniques, plus device-specific contracts ("I
will not overwrite memory until you set a bit in one of my
registers telling me to do so.")

* --> as usual, need to read the manual

## B. Polling vs. interrupts
    [next time]

[Acknowledgments: Mike Walfish, David Mazieres, Mike Dahlin, Brad Karp]


# 1. I/O, continued

(last time)
- I/O architecture

- PMIO/MMIO

** Polling vs. interrupts

    Polling: check back periodically 

        kernel...

       - ... sent a packet? Periodically ask the card when the buffer is
         free.

       - ... waiting for a packet? Periodically ask whether there is
         data

       - ... did Disk I/O? Periodically ask whether the disk is done.

        Disadvantages: wasted CPU cycles

    Interrupts:

      Recall interrupts:
        *  three ways to trap to kernel: syscall, exceptions and interrupts
        *  CPU scheduling, for preemptive scheduler, timer interrupt
        *  how CPU+OS handle interrupts (like page faults)
          * (OS configures) registering handlers in IDT (interrupt descriptor table)
          * (CPU executes) when meeting interrupts, CPU transfer control to handlers

      The device interrupts the CPU when its status
      changes (for example, data is ready, or data is fully written).

      This is what most general-purpose OSes do. There is a
      disadvantage, however. This could come up if you need to
      build a high-performance system.

      Namely: If interrupt rate is high, then the computer can
      spend a lot of time handling interrupts (interrupts are
      expensive because they generate a context switch, and the
      interrupt handler runs at high priority).

          --> in the worst case, you can get *receive livelock*
          where you spend 100% of time in an interrupt handler but no
          work gets done.

    How to design systems given these tradeoffs? Start with
    interrupts. If you notice that your system is slowing down
    because of livelock, then switch to polling. If polling is chewing
    up too many cycles, then move towards an adaptive switching
    between interrupts and polling. (But of course, never optimize
    until you actually know what the problem.) A classic reference
    on this subject is the paper 
        "Eliminating Receive Livelock in an Interrupt-driven
        Kernel", by Mogul and Ramakrishnan, 1996.

    We have just seen two approaches to synchronizing with
    hardware:

        polling
        interrupts

    Notice that they are mostly about communicating device status.
    How about data transfer?
    (by "mostly",  I mean gettting/setting status/commands don't have a
    clear difference from "data transfer"---status/commands are
    bits as well!)

** DMA vs. programmed I/O

    Programmed I/O: what we have been seeing in the handout
    so far: CPU writes data directly to device, and reads data
    directly from device.

DMA: better way for large and frequent transfers

    CPU (really, device driver programmer) places some buffers
    in main memory.

    Tells device where the buffers are

    Then "pokes" the device by writing to register

        Then device uses *DMA* (direct memory access) to read or
        write the buffers,

        The CPU can poll to see if the DMA completed (or the device
        can interrupt the CPU when done).

        [rough picture:
       buffer descriptor list
       <metadata> --> [  buf ]
       <metadata> --> [  buf ]
       ....
        ]

    DMA process is managed by a hardware known as a DMA controller (DMAC).

    This makes a lot of sense. Instead of having the CPU
    constantly dealing with a small amount of data at a time, the
    device can simply write the contents of its operation straight
    into memory.

    NOTE: OSTEP couples DMA to interrupts, but things don't have to
    work like that. You could have all four possibilities in
    {DMA, programmed I/O} x {polling, interrupts}. 

        For example, (DMA, polling) would mean requesting a DMA
        and then later polling to see if the DMA is complete.

# 2. Device drivers

    The examples (keyboard and screen) on the handout are
    simple device drivers.

    Device drivers in general solve a software engineering problem ...

        [draw a picture of different devices have different shapes
        and drivers fit them into kernel]

        expose a well-defined interface to the kernel, so that the
        kernel can call comparatively simple read/write calls or
        whatever.

        For example, reset, ioctl, output, read, write,
        handle_interrupt()

        this abstracts away nasty hardware details so that the kernel
        doesn't have to understand them.

        When you write a driver, you are implementing this interface,
        and also calling functions that the kernel itself exposes

    ... but device drivers also *create* software engineering problems.

    Fundamental issues:

        Each device driver is per-OS and per-device (often can't reuse
        the "hard parts")

        They are often written by the device manufacturer (core
        competence of device manufacturers is hardware development, not
        software development).

        Under conventional kernel architectures, bugs in device drivers
        -- and there are many, many of them -- bring down the entire
        machine.

    So we have to worry about potentially sketchy drivers ...

    ... but we also have to worry about potentially sketchy devices.

        a buggy network card can scribble all over memory 
        (solution: use IOMMU; advanced topic)

        plug in your USB stick: claims to be a keyboard; starts issuing
        commands. (IOMMU doesn't help you with this one.)

        plug in a USB stick: if it's carrying a virus (aka malware),
        your computer can now be infected. (Iranian nuclear reactors are
        thought to have been attacked this way.)

    Stuxnet example

    [if interested, check out:

        Angel, S., Wahby, R.S., Howald, M., Leners, J.B., Spilo, M., Sun, Z., Blumberg,
        A.J. and Walfish, M., Defending against malicious peripherals with Cinch.
    ]


# 3. Synchronous vs asynchronous I/O

- A question of interface

- NOTE: kernel never blocks when issuing I/O. We're discussing the
interface presented to user-level processes.

- Synchronous I/O: system calls block until they're handled.

- Asynchronous I/O:

    I/O doesn't block. for example, if a call like read() or write()
    _would_ block, then it instead returns immediately but sets a
    flag indicating that it _would_ have blocked.

    Process discovers that data is ready either by making another
    query or by registering to be notified by a signal (discuss
    signals later)

- Annoyingly, standard POSIX interface for files is blocking,
always. Need to use platform-specific extensions to POSIX to get
async I/O for files.

- Pros/cons:

    - blocking interface leads to more readable code, when
    considering the code that invokes that interface

    - but blocking interfaces BLOCK, which means that the code
    _above_ the interface cannot suddenly switch to doing something
    else. if we want concurrency, it has to be handled by a layer
    _underneath_ the blocking interface. 

# 4. User-level threading

* different from what we learned/used, which are kernel-managed threads

* We can also have *user*-level threading, in which the kernel is
    completely ignorant of the existence of threading.

    [draw picture]

        T1     T2     T3
        thr package/library
               OS
               H/W

    * in this case, the threading package is the layer of software that
    maintains the array of TCBs (thread control blocks)

Two types of user-level threading:

a) Cooperative multithreading

* This is also called *non-preemptive multithreading*.

* It means that threads' context switches take place only
at well-defined points: when the thread calls yield() and
when the thread would block on I/O.

b) Preemptive multithreading in user-level

How can we build a user-level threading package that does
context switches at any time?

Need to arrange for the package to get interrupted.

How?

Signals!

Deliver a periodic timer interrupt or signal to a thread
scheduler [setitimer() ]. When it gets its interrupt, swap out
the thread, run another one

Makes programming with user-level threads more complex -- all the
complexity of programming with kernel-level threads, but few of the
advantages (except perhaps performance from fewer system calls).

in practice, systems aren't usually built this way, but sometimes it
is what you want (for example, if you're simulating some OS-like
thing inside a process, and you want to simulate the non-determinism
that arises from hardware timer interrupts).

A larger point: signals are instructive, and are used for many
things. What a signal is really doing is abstracting a key hardware
feature: interrupts.

So this is another example of the fact that the OS's job is to give
a user-space process the illusion that it's running on something
like a machine, by creating abstractions. In this example, the
abstraction is the signal, and the thing that it's abstracting is an
interrupt.


# 5. Disks

Disks have historically been *the* bottleneck in many systems

    - This becomes less and less true every year:
    - SSDs (solid state drives) now common; will see in a sec
    - PM (persistent memory) or NVRAM (non-volatile RAM) now available

[Reference: "An Introduction to Disk Drive Modeling",
by Chris Ruemmler and John Wilkes. IEEE Computer 1994, Vol. 27,
Number 3, 1994. pp17-28.]

## A. What is a disk?

[see handout]

* stack of magnetic platters

* Rotate together on a central spindle @3,600-15,000 RPM

* Arms rotate around pivot, all move together

* Arms contain disk heads--one for each recording surface

* Heads read and write data to platters

[interlude: why are we studying this?

disks are still widely in use everywhere, and will be for some
time. Very cheap. Great medium for backup. Better than SSDs for
durability (SSDs have limited number of write cycles, and decay
over time)

Google, Facebook, etc. historically packed their data centers
full of cheap disks.

As a second point, it's technical literacy; many filesystems
were designed with the disk in mind (sequential access
significantly higher throughput than random access). You have to
know how these things work as a computer scientist and as a
programmer.
]

## B. Geometry of a disk

[see handout]

* track: circle on a platter. each platter is divided into
concentric tracks.

* sector: chunk of a track

* cylinder: locus of all tracks of fixed radius on all platters

* Heads are roughly lined up on a cylinder

Question: how many heads do you think could work at the same time?
    * Generally only one head active at a time

* disk positioning system

    * Move head to specific track and keep it there

    * a *seek* consists of up to four phases:

      * speedup: accelerate arm to max speed or half way point

      * coast: at max speed (for long seeks)

      * slowdown: stops arm near destination

      * settle: adjusts head to actual desired track

      [BTW, this thing can accelerate at up to several hundred g]

* Question: which have better performance, reads or writes? why?
    [answer: writes. Here are reasons:

    * settle times takes longer for writes than reads. why?
    * because if read strays, the error will be caught, and the
    disk can retry
    * if the write strays, some other track just got clobbered.
    so write settles need to be done precisely]

[Acknowledgments: Mike Walfish, David Mazieres, Mike Dahlin, Brad Karp]