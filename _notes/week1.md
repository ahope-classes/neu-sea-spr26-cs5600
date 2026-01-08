---
title: 'Notes 0'
layout: page
summary: 'Placeholder'
---


# Syllabus
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## Introduction and Goals
 
### Introduce Staff


### Goals

* think as a system developer by learning how systems work
* learn abstractions and concepts in operating systems which are useful beyond OSes
* learn a set of skills and tools that are useful for developing systems (such as vim/emacs, gdb, shell)
* "no pain no gain". CS5600 is supposed to be heavy-loaded---you have five main labs to do

#### Non-goals

* not a programming class
* not a teamwork class
	- face challenges by your own
	- go deep instead of wide

### Assumptions

* know how to program: have used some programming languages (e.g., Python, Java)
* understand basic algorithms and data structures
* feel comfortable with C and Linux (you will get a sense from lab0)
* willing to learn (by yourself)

Questions:
  - who have taken OS courses, undergrad?
  - how many have used Linux? C? git? vim/emacs?
  - how many heard of registers (as in CPU)?
    

## What is an operating system?

  [draw picture of hardware
    (memory, CPU, disk, NIC, GPU), OS, user-level programs]

  [OS includes OS services: processes, virtual memory, file system
  (file contents, directories and file names), security, networking,
  inter-process communication, time, terminals, etc.]

- Purpose of OS: provide services to user-level programs
- **Definition:** An operating system creates, for processes, a machine that is easier to program than the raw hardware would be.
- this software is classically described as doing two things:

  * managing the resources of the machine 
	- example: scheduling: give every process some of the CPU
	- example: virtual memory: give every process some physical memory
	- resource management means that one bad program doesn't screw up another. OS does:
		- multiplexing
		- sharing
		- isolation, protection

  * abstracting the hardware 
    - hide details of hardware (OS gets dirty; programmers don't)
      - hardware is nasty to program directly!
      - consider what is involved in getting things to the disk this provides essential convenience and portability.

    - you really don't want applications to have to program
    the bare hardware
      - would lead to lots of repeated code
      - unclear how to run more than one application at once,
      particularly more than one that are mutually distrustful.

      - hide details:
        - "All problems in computer science can be solved by another
        level of indirection [abstraction]."  --Butler Lampson
        - why? abstraction allows the operating system to make
        changes to the underlying hardware without impacting applications.
        - example: switching between WiFi and cellular networks.
        - example: switching between heterogeneous CPU/GPU cores
        for saving energy.

    - what are examples of managing resources and providing abstractions?
      many, for example, file systems, memory, and scheduling

      * file systems:
        *abstraction:* illusion file is continuous array of bytes; it's not

		```c
		fd = open("/tmp/foo", O_WRONLY)
		rc = write(fd, "abc...z", 26)
		```    

        abstractions here:
          - files
          - file descriptors
          - location of the file: applications can be entirely unaware of
          whether the file they are writing to goes on a hard disk inside
          the computer, a USB stick attached to the computer, a remote
          storage service like Dropbox or iCloud, etc.

        - isolation: user program can't write to a file unless
        it has permission. 

          in some cases, for example, smart phones, the system goes even
          further and gives each application the impression that it is
          the only one who can change the file system. This has become
          particularly important in order to ensure that applications
          cannot stomp on each other's data, and reduce the kinds of
          errors the programmer must consider when writing programs.

      * memory:
      	```
        movl 0x1248, %rdx   [means "get contents of 0x1248 and put in %rdx"]
        ```
        - abstraction: user program thinks it is reading from
        0x1248; it is not
        - more generally, user program thinks it has a linear,
        contiguous address space; it does not
        - isolation: user program can't write to another user's
        memory
        - programmers use even higher abstractions
        (than user program):
    	```
            int a = 1;     [assign value "1" to an variable "a"]
            [Java vm -> OS -> hypervisor]
         ```
      * scheduling:

        - abstraction: process has the illusion that it is running
        continuously; it is not
        - isolation: user program that is hogging CPU gets switched
        out in favor of another user's program

## Why study systems?

"My Future Is In Deep Learning!"

- 10x speedup by understanding the GPUs
- MIG helps save 50% of GPUs
- in fact, TensorFlow is published on OSDI'16

"Doing systems means being at ease with the machine" --Brad Karp

a. It's essential to know "how things work."

- doesn't matter what types of programs you work on,
knowing what is happening under the hood is essential to
debugging and improving performance.
	- Java programmers, understanding GC will be helpful.(an example of two sigma)
	- phone applications, understanding what is going on in the OS helpful with improving performance and battery life.
    - DNNs (deep neural networks), some of the concerns from OSes impact how you batch things, how you schedule, etc.

b. The ideas are everywhere: resource management and abstraction, for example.

c. There are design trade-offs that are fundamental
      - code must be efficient (so low-level?) ...
      - ...but abstract/portable (so high-level?)
      - code must be powerful (so many features?)
      - ...but simple (thus a few composable building blocks?)
      - another trade-off: between security and convenience

d. new challenges that will affect you
  - OS security and privacy remains a problem area.
  - multicore and manycore: yes, the operating systems run, but there's an argument that perhaps the abstractions should be different  (interaction between caches and address space abstractions)

e. Skill-building
  - have a holistic view of a system
  - have the confidence to address hard problems



## How will we study (operating) systems?

a. Learn how stuff works
  - sometimes through case studies

b. Learn specific techniques
  - time-tested solutions to hard problems
  - avoid "hacks"
	- examples: concurrent programming

c. Learn how to approach problems
  - fundamental issues
  - concept of a design space
    - design choices vs. implementation details

d. this class:
  - focus on "science" part (as in "computer science") for systems
  - lots of discussion of OS/HW boundary and process/OS boundary (interfaces). Less attention to detailed implementation of OS.
  - not a priority: details of deployed OSes (e.g., details of
    Linux), but we may sometimes use these as examples

##  Mechanics and admin

a. communication

  us-to-you:
    - homepage, announcements: check this every day
    - your NEU email (seldom)

  you-to-us:
    - piazza: questions that are not sensitive
      **No valid C code on Piazza**
    - staff email list for admin/sensitive things
    - office hours

b. components of the course:
  - lectures
  - labs
  - exams
  - reading
  - homeworks

c. lectures
  - attending: no roll call, but...will randomly pick students to answer questions
  - notes will be published, but will be hard to understand if you miss the lecture
  - asking questions in class is encouraged (different from asking questions about labs; see below)

d. labs:
  - key piece of the course
  - labs could/should actually be fun 
  - often: not much code to write (relatively speaking), but lots to learn!
  - "Start early"
    "When is early?"
    "Earlier than you think"
  - Regardless, you need to allocate time. 
  - I'm expecting you to feel challenged by the labs. The concept
  of "no pain, no gain" applies to learning. let's dive into
  that...
    - The building of systems cannot be learned by lecture or reading a textbook, nor by reading others' code.
    - Indeed, programming is where the real learning happens, that's the whole point
    - Independent effort is essential
    - Debugging is hard and time-consuming, but it's an essential skill
    - Only by producing working solution code yourself (without looking at others’ solution code) will you learn to build systems that solve problems you haven't seen before (the blind alleys may feel like wasted time, but they are teaching you how to avoid problems in the future, how to be independent)
    - Don't think of labs as having a grade.

      (people sometimes think that they need to get all the points,
      they get frustrated with the staff and TAs for not telling
      them how to get the points. But that misunderstands
      the purpose.)

      The purpose is to cause you to learn.
        [compare to exams: personal growth rare on an exam]

 - we expect you think through, then ask
   - "Here is my code. It doesn't work. Please debug." won't work.
   - If you get a reply from a TA, and then send email 20 minutes later
     asking a closely related question, that’s probably not great too.


e. exams:
  - midterm and final
  - closed book
  - will cover lectures, homeworks, readings, and labs

f. reading & homework
  - you may find lectures would not repeat readings.
    That's intended (also that's the point).

  [draw a figure about depth versus width of knowledge;
   point out relations between textbooks, lectures, and labs]

g. integrity policies:

  - Here are some questions:

   Looking at a classmate's solution and then coding it by yourself afterward
   Showing your code to a classmate who has questions
   Modifying code that you find on StackOverflow
   Modifying code for a similar assignment that you find on GitHub

  The correct answer: ALL of these are ruled out by the policy.

  - Please see the policy page, and let me say here:

  - The collaboration and academic integrity policy is real
  - please make sure that you've really thought through your question on your own before you ask for help
  - Exams will have questions about labs; and "If there are inexplicable discrepancies between exam and lab performance, we will overweight the exam, and possibly interview you." (see the policy page)


