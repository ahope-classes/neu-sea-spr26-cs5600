---
title: 'Notes Week 4: '
layout: page
summary: ''
---

# Week 4: Sync & Concurrency
{: .no_toc }


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

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



1. Last time
2. Intro to concurrency
3. Memory consistency model
4. Managing concurrency
--------------------------------------------

Admin

- Lab2 is released
  -- substantially harder: C + syscall + shell
  -- milestone in a week

--------------------------------------------

1. Last time

- Interface to threads:
	* ` tid thread_create (void (*fn) (void *), void *); `
    * `void thread_exit (); `
    * `void thread_join (tid thr); `

- thread vs. process
	*  threads share memory, but they have their own "execution context" (registers and stack).

## 2. Intro to concurrency

* There are many sources of concurrency.

* what is concurrency?
    * stuff happening at the same time

* sources of concurrency
    * on a single CPU, processes/threads can have their instructions interleaved (helpful to regard the instructions in multiple threads as "happening at the same time")
    * computers have multiple CPUs and common memory, so instructions in multiple threads can happen at the same time!
    * interrupts (CPU was doing one thing; now it's doing another)

* why is concurrency hard?
    * Hard to reason about all possible interleavings
    * (deeper reason: human brain is "single-threaded")

    * handout:

```
        1a:  x = 1 or x = 2.
        1b:  x = 13 or x = 25.
        1c:  x = 1 or x = 2 or x = 3 

            say x is at mem location 0x5000

            f is "x = x+1;", which might compile to:

            movq 0x5000, %rbx    # load from address 0x5000 into register
            addq $1, %rbx        # add 1 to the register's value
            movq %rbx, 0x5000    # store back


            g is "x = x+2;", which might compile to:

            movq 0x5000, %rbx    # load from address 0x5000 into register
            addq $2, %rbx        # add 2 to the register's value
            movq %rbx, 0x5000    # store back
```

      2: incorrect list structure

        both threads will insert new List_elem as head
        one List_elem will be lost (later threads cannot access it)


      3: incorrect count in buffer

       [draw producer-consumer on board]
```
        producer --+                 +--> consumer
                   |                 |
                   +---> [buffer] ---+
                      (count, in, out)
```

"buffer" of course is not a real ring.
it is an array with a wrap around pointer.

Question: why might go wrong with this incorrect count?
[answer: the producer may overwrite items;
the consumer may take out NULL.]

points:

* all of these are called race conditions; not all of them are errors, though
* all of these can happen on one-CPU machine (multi-core case can be worse; will see soon)


- revisit: an example you've seen, HW2 Q2.b
```
    int main()
    {
        fork();
        fork();
        // [update 2/4: fix the double quotation marks]
        printf("\nhello world");
    }
```

  * expected outputs: [many possibilities; should see four newlines ("\n") and four "hello world", but their order is uncertain.]


* What are you going to do when debugging concurrency code?

    * worst part of errors from race conditions is that a program may work fine most of the time but only occasionally show problems. why?  (because the instructions of the various threads or processes or whatevever get interleaved in a non-deterministic order.)

    * and it's worse than that because inserting debugging code may change the timing so that the bug doesn't show up


## 3. Memory consistency model

* hardware makes the problem even harder

    * [look at handout_w5a panel 4; what is the correct answer?]

    * [draw these cases on board]

    * [answer: "it depends on the hardware"]

    * sequential consistency not always in effect

    * sequential consistency means:

      (or what I call, "formal definition of how normal human being think of concurrency")

        * (1) maintain a single sequential order among operations (namely: all memory operations can form a sequential order where the read ops read from the most recent writes.)

        * (2) maintain program order on individual processors (namely: all operations from the same thread follow their issue order in the program)

    * a concurrent execution is **equivalent** to a sequential execution when

       * (0) of course, both must have the same operations

       * (1) all (corresponding) reads return the same values

       * (2) the final state are the same

    * if a memory has _sequential consistency_, it means that:

       * a concurrent program running on multiple CPUs
        *  looks as if the same program (with the same number of threads) running on a single CPU.

      * or, more formally,

        * there always _exists_ a sequential order of memory operations that is _equivalent_ to what the concurrent program produces running on multiple CPUs.


    - On multiple CPUs, we can get "interleavings" _that are impossible on single-CPU machines_. In other words, the number of interleavings that you have to consider is _worse_ than simply considering all possible interleavings of sequential code.


    - explain why sequential consistency is not held:

      *  CPUs -> Memory            (too slow)

      *  CPUs -> Caches -> Memory  (Write stall: invalidating cache requires a long time)

      *  CPUs -> StoreBuffers -> Caches -> Memory

      *  more...
        *  out-of-order execution
        *  speculative execution




```
    +-----+           +-----+
    | CPU |           | CPU |
    |     |           |     | [later: StoreBuffer]
    |     |           |     | [later: cache]
    +--+--+           +--+--+
       |                 | 
       |                 |
       +-------+---------+
               |
               v
         +------------+
         |   memory   |
         +------------+

```

- To whom interested in knowing more about the
  details of memory consistency:
   * http://sadve.cs.illinois.edu/Publications/computer96.pdf
   * Paul E. McKenney, "Memory Barriers: a Hardware
     View for Software Hackers"

* assume sequential consistency until we explicitly relax it

Discussion: memory consistency and nosql
  *  sequential consistency
  *  causal consistency
  *  eventual consistency


## 4. Managing concurrency

* critical sections
* protecting critical sections
* implementing critical sections

* step 1: the concept of *critical sections*

    * Regard accesses of shared variables (for example, "count" in
    the bounded buffer example) as being in a _critical section_

    * Critical sections will be protected from concurrent execution

[draw: program {A; c.s.; B} and three threads running it

```
        T1 ---[A][c.s.]-----[  B  ]-->
        T2 ---[A]------[c.s.]---[ B ]----->
        T3 ---[A]------------[c.s.]-----[B]----->
```


* analogy: a room that only allows one person
   * people: threads
   * room: critical section

    * Critical section should satisfy 3 properties:

        1. **mutual exclusion** only one thread can be in c.s. at a time  [this is the notion of atomicity]

        2. **progress** if no threads executing in c.s., one of the threads trying to enter a given c.s. will eventually get in

        3. **bounded waiting** once a thread T starts trying to enter the critical section, there is a bound on the number of other threads that may enter the critical section before T enters

    * Note progress vs. bounded waiting 

        * If no thread can enter C.S., don't have progress 

        * If thread A waiting to enter C.S. while B repeatedly leaves and re-enters C.S. ad infinitum, don't have bounded waiting 

        * We will be mostly concerned with mutual exclusion (in fact, real-world synchronization/concurrency primitives often don't satisfy bounded waiting.)

        [will continue in the next lecture]


1. Last time
2. Critical section
3. Bakery algorithm
4. Mutexes
--------------------------------------------

Admin

--Thur office hour changed to 9AM

--No class next Monday (Feb.21)

--Virtual class the Monday after next (Feb.28)

--------------------------------------------

## 1. Last time

  - interesting question: are there zombie threads? [No]
    why not?

  - an example
```c
    void *f(void *xx) {
        sleep(1);
        printf("this is f\n");
        exit(0);
    }

    int main(){
        pthread_t tid;
        pthread_create(&tid, NULL, f, NULL);
        pthread_join(tid, NULL);  // line X
        printf("this is main\n");
    }
```

Question: what is the output? 
  [answer: "this is f"]

Question: if we comment line X, what is the output?
  [answer: "this is main"]

  - Sequential consistency (SC)

    * examples: 

```c
      SC trace:
      T1: W1(x)=1, R1(y)=2
      T2: W2(y)=2, R2(x)=1
        [why? a sequential order: W1,W2,R1,R2]

      non-SC trace:
      T1: W1(x)=1, R1(y)=0
      T2: W2(y)=2, R2(x)=0
         [why? cannot find a sequential order]
         note, this can happen in TSO (namely, your x86 machines)
```

[show slides]

* in fact, checking SC is NP-hard

- multi-core CPU architecture
[show slides]


## 2. Critical sections

* critical sections intro (last time)
* protecting critical sections
* implementing critical sections

* Solution must satisfy 3 properties:

    1. **mutual exclusion**
    only one thread can be in c.s. at a time        
            [this is the notion of atomicity]

    2. **progress**
    if no threads executing in c.s., one of the threads
    trying to enter a given c.s. will eventually get in
      (sometimes called "no lockout")

    3. **bounded waiting**
    once a thread T starts trying to enter the critical
    section, there is a bound on the number of other threads
    that may enter the critical section before T enters
      (sometimes called "no deadlock")

* protecting critical sections.

    * want `lock()`/`unlock()` or `enter()`/`leave()` or `acquire()`/`release()`
    * lots of names for the same idea

    * in each case, the semantics are that once the thread of
    execution is executing inside the critical section, no other
    thread of execution is executing there

      * analogy: mutex => door for the room
        * lock/qcuire => close the door
        * unlock/release => open the door
                            (let the next person enter the room)

    [show handout week5a]
    Question: put `f()`/`g()` into critical sections on handout week5a,
              will it make a difference? what about 1.c?

    [answer: 
       1.a: no;
       1.c: yes, the only possible output will be 3.]


* implementing critical sections

    * "easy" way, assuming a uniprocessor machine: 

        enter() * > disable interrupts
        leave() * > reenable interrupts

        [convince yourself that this provides mutual exclusion]

    * bakery algorithm (see below)

    * we will study other implementations (locks) later.


## 3. Bakery algorithm

* the first algorithm to implement critical section **without atomic memory primitives**

* invented by Leslie Lamport, a Turing Award winner

* idea: simulate bakery shop
	* each customer takes a ticket with a number and will buy stuff when the number is called.
  	* "A New Solution of Dijkstra's Concurrent Programming Problem Communications of the ACM 17, 8 (August 1974), 453-455."

  [show algorithm]

  * bakery algorithm does not require atomic memory reads/writes.
    It works on "safe registers":
      * if a read overlaps writes (they are concurrent operations), the read can return anything.
      * if a read does not overlap with writes, the read will return the most recent write.


## 4. Mutexes

*  Mutex : mutual exclusion object

*  Usage:

```c
        mutex_t m
        mutex_init(mutex_t* m)
        acquire(mutex_t* m)
        release(mutex_t* m)
        ....
```

* pthread implementation:
     `pthread_mutex_init()`, `pthread_mutex_lock()`, ...

* using critical sections
  [see handout]

    * linked list example

    * bounded buffer example

* why are we doing this?

    * because *atomicity* is required if you want to reason about the correctness of concurrent code

    * atomicity requires mutual exclusion aka a solution to critical sections

    * mutexes provide that solution

    * once you have mutexes, don't have to worry about arbitrary interleavings. critical sections are interleaved, but those are much easier to reason about than individual operations.

    * why? because of _invariants_.

        examples of invariants:

        "list structure has integrity"

        "'count' reflects the number of entries in the buffer"

    the meaning of `lock.acquire()` is that if and only if you get
    past that line, it's safe to violate the invariants.

    the meaning of `lock.release()` is that right _before_ that line,
    any invariants need to be restored.

    the above is abstract.

    let's make it concrete:

        invariant: "list structure has integrity"

        so protect the list with a mutex

        only after acquire() is it safe to manipulate the list

* Question: by the way, why aren't we worried about *processes*
  trashing each other's memory?

[answer: because the OS, with the help of the hardware,
arranges for two different processes to have isolated memory
space. such isolation is one of the uses of virtual memory,
which we will study in a few weeks.]



1. Lab2
2. Condition variables
3. Semaphores
4. Monitors and standards
5. Advice for concurrent programming
--------------------------------------------

Admin

  --next class is virtual

  --midterm in <2weeks (03/07)
    --closed book
    --three topics: processes, scheduling, and concurrency

--------------------------------------------

## 1. Lab2

  - pipe
    *  draw how it looks
    *  double close?


## 2. Condition variables

###    A. Motivation

    [last time]

* producer/consumer queue

* very common paradigm. also called "bounded buffer":

  * producer puts things into a shared buffer
  * consumer takes them out
  * producer must wait if buffer is full; consumer must
    wait if buffer is empty
  * shows up everywhere
  * Soda machine: producer is delivery person, consumer
      is soda drinkers, shared buffer is the machine
  * OS implementation of pipe()
  * DMA buffers

* producer/consumer queue using mutexes (see handoutw5b)

    * what's the problem with that?

    * answer: a form of *busy waiting* 

      * analogy: the next person keeps knocking the door

* It is convenient to break synchronization into two types:

    * *mutual exclusion*: allow only one thread to access a given
    set of shared state at a time

    * *scheduling constraints*: wait for some other thread to do
    something (finish a job, produce work, consume work, accept
    a connection, get bytes off the disk, etc.)

### B. Usage

* API

	* `void cond_init (Cond *, ...);` 
		* Initialize
	* `void cond_wait(Cond *c, Mutex* m);`
		* Atomically unlock `m` and sleep until `c` signaled 
		* Then re-acquire `m` and resume executing 

	* `void cond_signal(Cond* c);`
		* Wake one thread waiting on `c`

  [in some pthreads implementations, the analogous
  call wakes *at least* one thread waiting on `c`. Check the
  the documentation (or source code) to be sure of the
  semantics. But, actually, your implementation shouldn't
  change since you need to be prepared to be "woken" at
  any time, not just when another thread calls `signal()`.
  More on this below.]

  * `void cond_broadcast(Cond* c);`
  * Wake all threads waiting on c

### C. Important points

(1) We MUST use "while", not "if". Why?

* Because we can get an interleaving like this:

    * The `signal()` puts the waiting thread on the ready list but doesn't run it

    * That now-ready thread is ready to `acquire()` the mutex (inside `cond_wait()`).

    * But a *different* thread (a third thread: not the signaler, not the now-ready thread) could `acquire()` the mutex, work in the critical section, and now invalidates whatever condition was being checked

    * Our now-ready thread eventually `acquire()`s the mutex...

    * ...with no guarantees that the condition it was waiting for is still true

* Solution is to use "while" when waiting on a condition variable

* DO NOT VIOLATE THIS RULE; doing so will (almost always) lead to incorrect code

    * NOTE: NOTE: NOTE: There are two ways to understand while-versus-if:
    	* (a) It's the 'while' condition that actually guards the program.
    	* (b) There's simply no guarantee when the thread comes out of wait that the condition holds. 

(2) `cond_wait` releases the mutexes and goes into the waiting
state in one function call (see panel 2b of the handoutw5b). 

* QUESTION: Why?

* Answer: can get stuck waiting.

```
Producer: while (count == BUFFER_SIZE)
Producer: release()
Consumer: acquire()
Consumer: .....
Consumer: cond_signal(&nonfull)
Producer: cond_wait(&nonfull)
```
* Producer will never hear the signal!


[skipped in class]
Discussion: `thread_cond_wait` implementation

* pthread implementation: 
https://code.woboq.org/userspace/glibc/nptl/pthread_cond_wait.c.html

```c
    thread_cond_wait(...) {

        release lock;

        do {
            wait-for-signal;
        } while (!signal)

        acquire lock;
    }
```

[skipped in class]
## 3. Semaphores

* (In this class, we require you to use condition variables and mutexes; semaphores will be considered incorrect.)

* Advice: don't use these. We're mentioning them only for completeness and for historical reasons: they were the first general-purpose synchronization primitive, and they were the first synchronization primitive that Unix supported.

* Introduced by Edsger Dijkstra in late 1960s

* Semaphore is initialized with an integer, N

* Two functions:
	* `Down()` and `Up()` [also known as `P()` and `V()`]
	* The guarantee is that `Down()` will return only N more times than `Up()` is called
	* Basically a counter that, when it reaches 0, causes a thread to `sleep()`

* Another way to say the same thing:
	* Semaphore holds a count
	* `Down()` is an atomic operation that waits for the count to become positive; it then decrements the count by 1
	* `Up()` is an atomic operation that increments the count by 1 and then wakes up a thread waiting on Down(), if any

* Reasons we recommend against their use:
	* semaphores are dual-purpose (for mutual exclusion and scheduling constraints), so hard to read code and hard to get code right
	* semaphores have hidden internal state
	* getting a program right requires careful interleaving of "synchronization" and "mutex" semaphores


## 4. Monitors and standards

Monitors = mutex + condition variables

* High-level idea: an object (as in object-oriented systems)

* in which methods do not execute concurrently; and

* that has one or more condition variables

* More detail

* Every method call starts with `acquire(&mutex)`, and ends with `release(&mutex)`

* Technically, these `acquire()`/`release()` are invisible to the programmer because it is the programming language (i.e., the compiler+run-time) that is implementing the monitor

    * So, technically, a monitor is a programming language concept

    * But technical definition isn't hugely useful because no programming languages in widespread usage have true monitors

    * Java has something close: a class in which every method is set by the programmer to be "synchronized" (i.e., implicitly protected by a mutex)

    * Not exactly a monitor because there's nothing forcing every method to be synchronized

    * And we can *use* mutexes and condition variables to implement our own manual versions of monitors, though we have to be careful

* Given the above, we are going to use the term "monitor" more loosely to refer to both the technical definition and also a "manually constructed" monitor, wherein:

    * all method calls are protected by a mutex (that is, the programmer inserts those `acquire()`/`release()` on entry and exit from every procedure *inside* the object)
    
    * synchronization happens with condition variables whose associated mutex is the mutex that protects the method calls

* In other words, we will use the term "monitor" to refer to the programming conventions that you should follow when building multithreaded applications

* Example: see handout week6.b, panel 1

## B. Standards & why?

  - RULES:
   * acquire/release at beginning/end of methods
   * hold lock when doing condition variable operations
   * always use "while" to check invariants, not "if"

  - why?

    * Mike Dahlin stands on the desk when proclaiming the standards

    * see Mike D's "Programming With Threads"

    * You are required to follow this document

    * You will lose points (potentially many!) on
    the exam if you stray from these standards

    * Note that in his example in section 4, there needs to be
    another line:

    * right before `mutex->release()`, he should have:
        `assert(invariants hold)`

   * the primitives may seem strange, and the rules may seem arbitrary: why one thing and not another?

    * there is no absolute answer here

    * **However**, history has tested the approach that we're using. If you use the recommended primitives and follow their suggested use, you will find it easier to write correct code

* For now, just take the recommended approaches as a given, and use them for a while. If you can come up with something better after that, by all means do so!

* But please remember three things:

    - a. lots of really smart people have thought really hard about the right abstractions, so a day or two of thinking about a new one or a new use is unlikely to yield an advance over the best practices.

    - b. the consequences of getting code wrong can be atrocious. see for example:

    * http://www.nytimes.com/2010/01/24/health/24radiation.html
    * http://sunnyday.mit.edu/papers/therac.pdf
    * http://en.wikipedia.org/wiki/Therac-25

    - c. people who tend to be confident about their abilities tend to perform *worse*, so if you are confident you are a Threading and Concurrency Ninja and/or you think you truly understand how these things work, then you may wish to reevaluate.....

    * Dunning-Kruger effect
      * https://en.wikipedia.org/wiki/Dunning%E2%80%93Kruger_effect

## C. The Commandments

* RULE:
	* acquire/release at beginning/end of methods
* RULE:
	* hold lock when doing condition variable operations
* Some people [for example, Andrew Birrell in this paper:
        http://www.hpl.hp.com/techreports/Compaq-DEC/SRC-RR-35.pdf ] will say: "for experts only, no need to hold the lock when signaling". IGNORE THIS. Putting the signal outside the lock is only a small performance optimization, and it is likely to lead you to write incorrect code.
* RULE:
	* a thread that is in `wait()` must be prepared to be restarted at any time, not just when another thread calls "`signal()`".
	* why? because the implementor of the threads and condition variables package *assumes* that the user of the threads package is doing `while(){wait()}`.

### 5. Advice for concurrent programming
    [We will use handout week6.b as a case study.....]
    * Here's a four-step design approach:

#### 1. Getting started:

 1a. Identify units of concurrency. Make each a thread with
 a `go()` method or main loop. Write down the actions a thread
 takes at a high level.  

 1b. Identify shared chunks of state. Make each shared
 *thing* an object. Identify the methods on those objects,
 which should be the high-level actions made *by* threads
 *on* these objects. Plan to have these objects be protected by mutexes.

 1c. Write down the high-level main loop of each thread. 

Advice: stay high level here. Don't worry about synchronization 
yet. Let the objects do the work for you. 
 
Separate threads from objects. The code associated with a
thread should not access shared state directly (and so there
should be no access to locks/condition variables in the
"main" procedure for the thread). Shared state and
synchronization should be encapsulated in shared objects. 

* QUESTION: how does this apply to the example on the handout?
* separate loops for `producer()`, `consumer()`, and synchronization happens inside MyBuffer.
 
Now, for each object: 
 
#### 2. Write down...
... the synchronization constraints on the solution. Identify the type of each constraint: mutual exclusion or scheduling. For scheduling constraints, ask, "when does a thread wait"?

* NOTE: usually, the mutual exclusion constraint is satisfied by the fact that we're programming with monitors.

* QUESTION: how does this apply to the example on the
handout?
    * Only one thread can manipulate the buffer at a time (mutual exclusion constraint)
    * Producer must wait for consumer to empty slots if all full (scheduling constraint)
    * Consumer must wait for producer to fill slots if all empty (scheduling constraint)

#### 3. Create a lock...
... or condition variable corresponding to each constraint 

* QUESTION: how does this apply to the example on the handout?
    * Answer: need a lock and two condition variables. (lock was sort of a given from the fact of a monitor).
 
#### 4. Write the methods,... 
...using locks and condition variables for coordination  

[thanks to David Mazieres, Mike Dahlin, and Mike Walfish
for content in portions of this lecture.]



On the board
------------
1. Concurrency, continued
3. Deadlock
3. Other concurrency issues
4. Other synchronization mechanisms

---------------------------------------------------------------------------

Admin

- midterm:
  -- time: 03/07, 2:50pm
  -- location: TBD
  -- review session
  -- 30min review + 1hr Q&A; bring your questions

- correct an answer from last time
  -- subshell input/output
  -- (echo abc; echo def) | wc -l

--------

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