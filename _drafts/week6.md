---
title: 'Notes Week 6: DRAFT'
layout: page
summary: 'Approaches to synchronization'
---

# Week 6: Sync & Concurrency, Cont. 
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

<details markdown="block">
<summary>Complete Table of Contents </summary>
1. TOC
{:toc}
</details>

## Agenda

1. Review Lab 2



## Weekly Summary and Where are we?

TODO 

### Topics

* **Last Week:** Introduction to systems in general
* **This Week:** Introduction to the Process abstraction and how to use it in C
* **Next Week:** How the OS schedules processes (and other things) 

### Assignments

* **Lab 0** (setting up your dev environment) should be submitted already! 
* **Lab 1** (getting familiar with C, `gdb`, `vim`, `ctags`, etc) is due on Friday
* **Lab 2** (building `ls`) is out; due 30 Jan 2026.
* **HW 1** is due on Friday

### Reading Summary

* OSTEP Chapter 25: Dialogue
* OSTEP Chapter 26: Concurrency and Threads
* OSTEP Chapter 27: Thread API
* OSTEP Chapter 28: Locks


### Announcements

* HW2 grading is done and out
    * Q1a: some student incorrectly stated 0 refers to "exit status 0" (success) rather than a NULL pointer to ignore status.
    * Q2a: some students missed the core concept of the buffer being duplicated
    * Q2b: some students failed to explain the reason for the output (non-determinism)


==========

Week 5....

==========



# Stuff for next time, probably

## 1. Concurrency, continued

*  condition variable: cond_wait(Cond, Mutex)
*  always use "while" instead of "if"
*  has to be a single interface
   (vs. release/wait/acquire)

*  Signal vs. Broadcast

   * both signal and broadcast wake up threads.

   Question: Can we replace SIGNAL with BROADCAST, given our monitor semantics?
    (Answer: yes, always.) Why?

   * `while()` condition tests the needed invariant. program doesn't progress pass `while()` unless the needed invariant is true.

   * result: spurious wake-ups are acceptable....

   * ...which implies you can always wakeup a thread at any moment with no loss of correctness....

   * ....which implies you can replace SIGNAL with BROADCAST [though it may hurt performance to have a bunch of needlessly awake threads contending for a mutex that they will then acquire() and release().]

   Question: Can we replace BROADCAST with SIGNAL?

   * Answer: not always. 

   * Example:
       * memory allocator
       * threads allocate and free memory in variable-sized chunks
       * if no memory free, wait on a condition variable
       * now posit:
       * two threads waiting to allocate chunks of memory
       * no memory free at all
       * then, a third thread frees 10,000 bytes
       * SIGNAL alone does the wrong thing: we need to awaken both threads

*  Monitor
*  an object (= mutex + condition variables)
*  should be a language primitive
*  C: a programming convention

*  "a safe way": Monitor + 6 rules + 4-step design approach
*  6 rules:
  *  must memorize! (one of few things I require you to recite)
*  4-step design approach

* dangerous concurrency world
* never ever try to create your own synchronization;
  they are likely to be broke
  (why?
  do you understand your hardware? don't build a castle on a swamp)
* one more example: double−checked locking pattern
[show slides]

*  Don't manipulate synchronization variables or shared state variables in the code associated with a thread; do it with the code associated with a shared object.  

    * Threads tend to have "main" loops. These loops tend to access shared objects. *However*, the "thread" piece of it should not include locks or condition variables. Instead, locks and CVs should be encapsulated in the shared objects.

    * Why?

    (a) Locks are for synchronizing across multiple threads. Doesn't make sense for one thread to "own" a lock.

    (b) Encapsulation -- details of synchronization are internal details of a shared object. Caller should not know about these details.  "Let the shared objects do the work."

    * Common confusion: trying to acquire and release locks
    inside the threads' code (i.e., not following this advice).
    Bad idea! Synchronization should happen within the shared
    objects. Mantra: "let the shared objects do the work".

    * Note: our first example of condition variables --
    handout, item 2b -- doesn't actually follow the advice, but
    that is in part so you can see all of the parts working
    together.

*  Different way to state what's above:

    * You want to decompose your problem into objects, as in
    object-oriented style of programming.

    * Thus:

    (1) Shared object encapsulates code, synchronization  variables, and state variables 

    (2) Shared objects are separate from threads 


## 2. Deadlock

* see handout: simple example based on two locks

* see handout: more complex example

    * M calls N 
    * N waits
    * but let's say condition can only become true if N is invoked
    through M
    * now the lock inside N is unlocked, but M remains locked; that
    is, no one is going to be able to enter M and hence N.

* can also get deadlocks with condition variables

* lesson: dangerous to hold locks (M's mutex in the case on the
handout) when crossing abstraction barriers

* deadlocks without mutexes:

    real issue is resources and how/when they are required/acquired

    (a) [draw bridge example]

    * bridge only allows traffic in one direction 

    * Each section of a bridge can be viewed as a resource. 

    * If a deadlock occurs, it can be resolved if one car backs up (preempt resources and rollback). 

    * Several cars may have to be backed up if a deadlock occurs. 

    * Starvation is possible. 

(b) another example:
    
    * one thread/process grabs disk and then tries to grab scanner

    * another thread/process grabs scanner and then tries to grab disk

* when does deadlock happen? under four conditions. all of them must hold for deadlock to happen:

1. mutual exclusion
2. hold-and-wait
3. no preemption
4. circular wait


* what can we do about deadlock?

    (a) ignore it: worry about it when it happens. the so-called "ostrich solution"

    (b) detect and recover

    * could imagine attaching debugger

    * not really viable for production software, but works well in development

    * threads package can keep track of resource-allocation graph

    * For each lock acquired, order with other locks held 
    
    * If cycle occurs, abort with error 
    
    * Detects potential deadlocks even if they do not occur 


(c) avoid algorithmically

* banker's algorithm
  * need to know the maximum resources needed by each process
  * only allocate resources if the state is safe
    (safe state: there are enough resources for *all* the executing processes to finish under some allocation scheduling.)

* elegant but impractical

* if you're using banker's algorithm, it looks like this:

```
ResourceMgr::Request(ResourceID resc,
             RequestorID thrd) {
    acquire(&mutex);
    assert(system in a safe state);
    while (state that would result from giving 
           resource to thread is not safe) {
        wait(&cv, &mutex);
    }
    update state by giving resource to thread
    assert(system in a safe state);
    release(&mutex);
}
```

Now we need to determine if a state is safe....

* an example:
  *  a bank has 10 coins
  *  three customers (max coins needed):
      A (9), B (4), and C (7)

  *  current status:
```
         has  max
      A  3    9
      B  2    4
      C  2    7

      bank: 3
```

  *  Question: is current state safe?
  [Answer: yes, because there is a way to let all finish: B->C->A]

  *  Question: if A requests and gets another coin, is the state then safe?
  [Answer: no, there is no *guarantee* for A and C to finish. Try it yourself.]
    * so the banker should reject A's request because the state if not safe.

See details at Wiki:
  https://en.wikipedia.org/wiki/Banker%27s_algorithm

* disadvantage to banker's algorithm:

* requires every single resource request to go through a single broker

* requires every thread to state its maximum resource needs up front. unfortunately, if threads are conservative and claim they need huge quantities of resources, the algorithm will reduce concurrency

(d) negate one of the four conditions using careful coding:

* can sort of negate 1
* put a queue in front of resources, like the printer
* virtualize memory

* can sort of negate 2
* either cancel "hold": abort when conflicting
* or cancel "wait": hold *all* resources at once

* can sort of negate 3:
* consider physical memory: virtualized with VM, can take physical page away and give to another process! 

* what about negating #4?

* in practice, this is what people do

* idea: partial order on locks

    * Establishing an order on all locks and making sure that every thread acquires its locks in that order

* why this works:

    * can view deadlock as a cycle in the resource acquisition graph

    * partial order implies no cycles and hence no deadlock

* two bummers:

    1. hard to represent CVs inside this framework.
        * works best only for locks.

    2. Picking and obeying the order on *all* locks requires that modules make public their locking behavior, and requires them to know about other modules' locking.  This can be painful and error-prone. 

    * see Linux's filemap.c example on the handout; this is complexity that arises by the need for a locking order

(e) Static and dynamic detection tools

* (if you're interested)
See, for example, these citations, citations
therein, and papers that cite them:

Engler, D. and K. Ashcraft. RacerX: effective,
static detection of race conditions and deadlocks.
Proc. ACM Symposium on Operating Systems Principles
(SOSP), October, 2003, pp237-252.
http://portal.acm.org/citation.cfm?id=945468

Savage, S., M. Burrows, G. Nelson, P. Sobalvarro,
and T. Anderson. Eraser: a dynamic data race
detector for multithreaded programs. ACM
Transactions on Computer Systems (TOCS), Volume 15,
No 4., Nov., 1997, pp391-411.
http://portal.acm.org/citation.cfm?id=265927

a long literature on this stuff

* Disadvantage to dynamic checking: slows program down

* Disadvantage to static checking: many false alarms
(tools says "there is deadlock", but in fact there is
none) or else missed problems

* Note that these tools get better every year. I believe
that Valgrind has a race and deadlock detection tool

## 3. Other concurrency issues

  (a) progress issues

    Deadlock was one kind of progress (or liveness) issue.
    Here are others...

  - Livelock

    *  two threads waiting on each other: deadlock
    *  each backs off when finding another process
    *  a possibility: all processes keep aborting and make no progress

      [draw a curly bridge]

  - Starvation

    * thread waiting indefinitely (if low priority and/or if
    resource is contended)

    *  for example, merging to a busy road with a yield sign

  - Priority inversion

    * T1, T2, T3: (highest, middle, lowest priority)

    * T1 wants to get lock, T2 runnable, T3 runnable and holding lock

    * System will preempt T3 and run highest-priority runnable thread, namely T2

    * Solutions:

      * Temporarily bump T3 to highest priority of any thread that is
      ever waiting on the lock

      * Disable interrupts, so no preemption (T3 finishes)
      ... not great because OS sometimes needs control
      (not for scheduling, under this assumption, but for
      handling memory [page faults], etc.)

      * Don't handle it; structure app so only adjacent priority
      processes/threads share locks

    * Happened in real life, see:
    https://www.microsoft.com/en-us/research/people/mbj/just-for-fun/
        (search for Pathfinder)

      *  Mars pathfinder experience total system resets
      *  what happened:
        *  data gathering thread (low priority)
        *  communication thread (medium priority)
        *  management thread (high priority)
        *  data gathering and management threads may acquire the same mutex
        *  classic priority inversion

  (b) other concurrency bugs

    - atomicity-violation bugs

```
        T1:
          if (thd->info) {
            ...
            use(thd->info);
            ...
          }

        T2:
          thd->info = NULL
```

- order-violation bugs

```
        T1:
          void init() {
            thd = createThread(...)
          }

        T2:
          void main() {
            state = thd->info
          }
```


"Learning from Mistakes -- A Comprehensive Study on Real World Concurrency Bug Characteristics"
      https://people.cs.uchicago.edu/~shanlu/preprint/asplos122-lu.pdf


   (c) performance vs. complexity

*  Coarse locks limit available parallelism  ....

[(still, you should design this way at first!!!)]

the fundamental issue with coarse-grained locking is that
only one CPU can execute anywhere in the part of your code
protected by a lock. If your critical section code is called
a lot, this may reduce the performance of an expensive
multiprocessor to that of a single CPU.

if this happens inside the kernel, it means that
applications will inherit the performance problems from the
kernel


 *  ... but fine-grained locking leads to complexity and hence bugs
 (like deadlock)

    * > although finer-grained locking can often lead to better
    performance, it also leads to increased complexity and hence
    risk of bugs (including deadlock).

    * > see handout

    * > also code here:
    https://github.com/torvalds/linux/blob/7cca308cfdc0725363ac5943dca9dcd49cc1d2d5/mm/filemap.c


## 4. Other synchronization mechanisms

  - semaphores
    *  mentioned last time
    *  first general-purpose synchronization primitive
    *  (you should not use it)

  - Barrier
    *  an example, GPU
    *  many cores, literally thousands
    *  no cache coherence (unlike CPU)
    *  after barrier, all threads are synced

  - spinlocks
    *  when acquiring the lock, the thread "spins".
    *  implementation
       [See handout]
    *  atomic instructions are expensive
      * some detailed experiments here:
        https://arxiv.org/pdf/2010.09852.pdf

  - reader-writer locks
    *  recall the "database problem" in handout week 5.a panel 2&3
    *  usage (*pseudocode*):

      rwlock_t lock
      acquire_rdlock(&lock)
      acquire_wrlock(&lock)
      unlock(&lock)

  - Read-copy-update (RCU)
    *  do we have to block readers when modifying a data structure?
    *  answer: No, using RCU
    *  an example of deletion in double-linked list
       [show slides]

    * basic idea:
      *  readers work as normal (as if there is no concurrent updaters)
      *  updaters
        *  make copies
        *  maintain both old and new version for a "grace period"
        *  delete the old version

    * see also:
        https://cs.nyu.edu/~mwalfish/classes/14fa/ref/rcu2.pdf


  - deterministic multithreading

    *  concurrent programs have "infinite" possible schedules 
       (hence we don't know what may go wrong)

    *  BUT, we do know which schedules are okay
       (for example, by testing)

    *  idea: avoid unseen interleavings and schedules
       which are potentially buggy

    *  see also:
       "Stable Deterministic Multithreading through Schedule Memoization"
       https://www.cs.columbia.edu/~junfeng/11sp-w4118/lectures/tern.pdf

  - A useful book for concurrency ninjas:
    Paul E. McKenney, "Is Parallel Programming Hard, And, If So, What Can You Do About It?"
    https://mirrors.edge.kernel.org/pub/linux/kernel/people/paulmck/perfbook/perfbook-e2.pdf

[thanks to David Mazieres, Mike Dahlin, and Mike Walfish
for content in portions of this lecture.]

