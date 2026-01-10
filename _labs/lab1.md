---
title: 'Lab 1: Reviewing C'
layout: lab
released: 2026-01-08
due: 2026-01-16
index: 1
summary: "Review using the C programming language, the debugger, and becoming familiar with coding from the command line.  "
---

# Lab 1: C review, gdb, software skills
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

This lab will reinforce your prior experience in the C programming language and give you practice using certain command-line tools. The overall comfort you’ll gain (or reinforce) with the Unix computing environment will be helpful for future labs.

# Getting Started

You will perform the labs using the CS5600 Docker environment, and pull/push code with Git and GitHub. It's time for you to set up that infrastructure: 

{: .note }
> Before you go further. Go to the setup page, and follow the instructions there. 
> 
> Specifically:
> 
> * Get a GitHub account if you don’t have one already
> * Set up your Git repository (setup SSH keys including ssh-agent, clone on GitHub using GitHub Classroom, clone to your local machine, set upstream)
> * Read through the git FAQs
> * Install Docker
> * Build and run the CS5600 Docker image
> * Come back here when you’ve done these.

## Pull the upstream lab code

Obtain and update the lab files as follows. This requires that you have set up from Lab 0 properly. 

```sh
$ cd ~/cs5600-gitId
$ git fetch upstream
$ git merge upstream/main
.... # output omitted
$ ls
cs5600-run-docker  docker  lab1  README.md
```

You should now have the `lab1` directory that you didn't have before. This might vary depending on when you did Lab 0. 

**Workflow.** We have configured Docker so that the directory from which you run `./CS5600-run-docker` on your host machine (usually this is `CS5600`) shows up in Docker as `CS5600-labs`. This means that you can use your preferred IDE or editor on your host machine to edit code while compiling, running, and testing inside Docker. Depending on your setup, it may be helpful for you to have at least two windows up (one for editing and one with Docker).

**Programming style.** In this and subsequent labs, you will be graded for style. We will deduct up to 20% of total lab points for poor style based on our subjective evaluation. Please read [this style guide](https://cs61.seas.harvard.edu/site/2022/Style/).



# Section 0: Expectations

The exercises in this lab are elementary. That means that an AI code assistant, such as GitHub CoPilot or ChatGPT, can possibly complete many or all of the exercises perfectly. That is unavoidable: the same kinds of exercises that teach beginners are also feasible for current-era AI. For that reason, this lab will be graded under homework rules (non-strictly, and a very low weighting of your grade). That does not mean the lab is skippable; read on.

Our quizzes, exams, and future labs will assume that you have acquired the skills in this lab. Therefore, we strongly encourage you to work this lab without AI assistance. We put teeth behind that encouragement. Specifically, we will look unfavorably on the following:

* Requests for setup help and tech support in future weeks that are consistent with not having worked lab 1 on your own.
* Strong submissions by students who are unable to perform the corresponding tasks on quizzes and exams.
* Nonsensical semantic errors of the kind produced by current-era AI assistants.

Most likely, AI assistants are not going away. But in order to learn something to the point where you can drive the AI sensibly (not to mention exceeding its capabilities), you have to actually learn yourself, which means doing exercises on your own.

# Section 1: C review

All of the projects in this class will be done in C or C++, for two main reasons. First, some of the things that we want to implement require direct manipulation of hardware state and memory, which are operations that are naturally expressed in C. Second, C and C++ are widely used languages, so learning them is a useful thing to do in its own right.

If you are interested in why C looks like it does, we encourage you to look at Ritchie's history. Here, perhaps, is the key quotation: "Despite some aspects mysterious to the beginner and occasionally even to the adept, C remains a simple and small language, translatable with simple and small compilers. Its types and operations are well-grounded in those provided by real machines, and for people used to how computers work, learning the idioms for generating time- and space-efficient programs is not difficult. At the same time the language is sufficiently abstracted from machine details that program portability can be achieved."

## C Warmup Exercises

You will do short exercises in C, as a warmup, and to help review what you may have learned about C previously. Enter the Docker environment (`./CS5600-run-docker`, per the setup page); you’ll know you’re in the Docker environment if you see a prompt like this:

```sh
CS5600-user@af8b1be95427:~/CS5600-labs$ 
```

then type `cd lab1/mini` after the prompt:
```sh
CS5600-user@af8b1be95427:~/CS5600-labs$ cd lab1/mini
CS5600-user@af8b1be95427:~/CS5600-labs/lab1/mini$
``` 

{: .highlight }
**Shell prompt.** From here forward, we will not include the `CS5600-user@...` part of the shell prompt in this description, but it should be present on your machine. If it is not, then you are not in the Docker environment.
	

### Building and Running C programs to Test

The first 3 exercises are defined in `lab1/mini`. For each of the 3 parts, open the relevant `partX.c` file, add your own C code as indicated, and then on the command line run `make` to build the code, and `build/partX` to run the program built with your code.

For example with Exercises 1, testing will look like:

```sh
$ make
$ build/part1
```

You should have seen `make` before. Recall that it is an automatic way to build software. In this case, it invokes the compiler and links your compiled code to a small test harness that we provide.
```sh 
$ ./build/part1
part1: set_to_fifteen OK
part1: array_sum OK
```

If your `part1.c` is correctly implemented, you will see OK, as above.

You will need to remove the `assert(0);` lines as you start working through the exercises. Remove this as you implement functions in future exercises; it is a reminder that you have yet to implement a particular function.

### Saving and Committing your changes with Git

As you write your code and improve it (fixing bugs, adding functionality), you should get in the habit of syncing your changes to the master copy of your labs repository on GitHub, for example:

```sh 
$ git commit -am 'my solution for lab1 exercise1'
Created commit 60d2135: my solution for lab1 exercise1
 1 files changed, 1 insertions(+), 0 deletions(-)
$ git push origin
```

The `commit` keeps the history of changes to your code, and so allows you to revert to an older version if you find that a change causes a regression. The `push` serves to back up your code on GitHub’s servers, so you won’t lose work if your local working copy is corrupted or lost.

Note: If `git push origin` does not work from within Docker and you are on a Mac, then make sure your `ssh-agent` is running. See [here](https://cs.nyu.edu/~apanda/classes/25fa/setup.html#sshkeys). If you are not on a Mac, you can either push from outside Docker using your host’s git or you can use GitHub’s Personal Access Tokens.


{: .exercise }
> {: .opaque }
> ### Exercise 1. Implement the functions in `part1.c`
> 


{: .exercise }
> {: .opaque }
> ### Exercise 2. Implement the functions `set_point` and `point_dist` in `part2.c`.
> 
> As above, you compile and test with:
> ```sh
> $ make
> ```
> 
> Once you have modified your code and it works properly, you should see: 
> ```sh
> $ ./build/part2
> part2: set_point OK
> part2: point_dist OK
> ```

As you work through the exercises, note that you can keep track of your changes by using the `git diff` command. Running `git diff` will display the changes to your code since your last commit, and `git diff origin/main` will display the changes relative to the initial code supplied for this lab. Here, `origin/main` is the name of the git branch with the initial code you downloaded from our server for this assignment.

{: .exercise }
> {: .opaque }
> ### Exercise 3. Implement the `linked_list` utility functions in `part3.c`.
> 
> Once you've implemented it, test it like so:
> 
> ```sh
> $ make
> $ ./build/part3
> part3: list_insert OK
> part3: list_end OK
> part3: list_size OK
> part3: list_find OK
> part3: list_remove OK
> ```


 
{: .highlight }
> {: .opaque }
>  **Debugging note:** if you are having trouble with this piece, you may wish to skip to the section covering gdb, below, and come back here. You would do:
> 
> ```
> $ gdb build/part3 core
> (gdb) bt
> ```
> (Make sure you’ve issued the `ulimit` command below.)
> 
> Another debugging note: the assert lines we’ve been seeing are a simple application of a powerful tool. You may wish to use asserts yourself, to help debug your linked list functions.
> 
> In C, an `assert` is a preprocessor macro which effectively enforces a contract in the program. (You can read more about macros in C here.) The contract that assert enforces is simple: when program execution reaches `assert(<condition>)`, if condition is true, execution continues; otherwise, the program aborts with an error message.
> 
> Assertions, when used properly, are powerful because they allow the programmer who uses them to guarantee that certain assumptions about the code hold. For example, you will see assertions like:
> 
> ```c
> assert(head != NULL);
> ```
> 
> This assertion enforces the contract that the parameter head cannot be `NULL`. If these assertions were not present and we tried to dereference head, for example with `*head`, we would encounter a type of error called a segmentation violation (or segmentation fault). This is because dereferencing the `NULL` address is invalid; `NULL` points to "nothing". Later in the lab, we will get some experience with segmentation faults.
> 
> But by using assertions, we guarantee that `list_end` (for example) will never try to dereference a head variable at `NULL`, saving us the headache of having to debug a segmentation fault if some code tried to pass us a `NULL` value. Instead, we will get a handy error message describing exactly what contract of the function was invalidated.
> 
> Use your own asserts to make debugging easier!


## Advice on debugging common problems encountered in doing this lab

* **Remember to recompile changed code** Whenever you’ve changed a file, always type make to re-compile before executing again.

* **Write your own simple test code.** Don’t rely solely on the lab’s testing infrastructure (i.e. `./grade-lab`) to test the correctness of your code. We don’t distribute the source of the harness code; this makes it hard to debug. So you should write your own tester. Let’s say you want to test the `array_sum` function in the file `part1.c`. To write your own test code, create a file (e.g. called `test-part1.c`) with a main function that invokes `array_sum` in various ways to test its correctness.

An example `test-part1.c` might look something like this.

Compile your test code by typing:

```sh
$ gcc -pedantic -Wall -std=c11 -g -o test_part1 test_part1.c part1.c
```

* **Use `gdb`** As we noted above, and as will be covered below.

## C programs from scratch

In the rest of Section 1 of the lab, you will get practice writing and compiling C programs from scratch. Knowing how to create a standalone C program is useful in its own right, and it will be helpful context for lab2.

We will quickly walk through how to write and compile a program in C, and then you will write and compile several programs of your own.

Using a text editor, create a file called `fun.c`. In this file, type:

```c
#include <stdio.h>

int main(int argc, char** argv)
{
    char* first_arg; 
    char* second_arg; 

    /* this checks the number of arguments passed in */
    if (argc != 3) {
        printf("usage: %s <arg1> <arg2>\n", argv[0]);
        return 0;
    }

    first_arg = argv[1];
    second_arg = argv[2];

    printf("My program was given two arguments: %s %s\n",
           first_arg, second_arg);

    return 0;

}

```

This is a complete C program. The `#include` at the top tells the compiler to use the header files of "standard I/O" (the standard input/ouput functions of the C library; these functions include `printf`). Also, `argc` contains the number of arguments passed to the program (including the program name itself) while `argv[0]` contains the name of the program that was invoked.

You can compile this program using:

```sh
$ gcc -pedantic -Wall -std=c11 -g -o fun fun.c
```

You can now run this program using:

```sh
$ ./fun
```

You should see:

```sh
usage: ./fun <arg1> <arg2>
```

You can also do:

```sh
$ ./fun abc def
```

My program was given two arguments: `abc def`


Note the pattern here:

* you create a C file and compile it with

```sh
$ gcc [flags] -o <output-file-name> <input-file-name>
```

* You run your program using whatever was after the `-o` flag.

* Inside the program, you gain access to the arguments using the `argv` array. (As we’ll see in this semester, the OS syscall `exec()` takes an `argv`; the OS takes that array and passes it to the new program's `main()` function.)


{: .exercise }
> {: .opaque }
> ### Exercise 4. Print first 3 chars
> Write, from scratch, a C program that takes two arguments and prints the first three characters of each string. If either of the two arguments has fewer than three characters, print an error message, and end gracefully (as opposed to core dumping).
> 
> For example:
> 
> ```sh
> $ ./first3 a b
> ./first3: error: one or more arguments have fewer than 3 characters
> 
> $ ./first3 abcd wxy
> abcwxy
> ```
> 
> Instructions:
> 
> * You may use the function `strlen()`. If you do, include `#include <string.h>` at the beginning of the program.
> * Put all your files in `~/CS5600/lab1/fromscratch/first3`.
> * Your executable file must be named `first3`.
> * You must supply a `Makefile`. When we type `make`, it needs to create a binary executable named `first3`.
> * Be diligent about creating test cases for yourself. You will lose points for not handling corner cases. You may wish to create a script that runs and tests your code on various cases. (You do not have to hand in your testing code, however; we will run our own testing scripts against your code.)
> * Pay attention to coding style.
> * Remember to add relevant source files and the `Makefile` to your git repo. Type `git status`, and if any of the files look relevant, add and commit them:
> 
> ```sh
>   $ git status
>   $ git add *.c *.h Makefile
>   $ git commit -am "first3"
>   $ git push origin
> ```

{: .exercise }
> {: .opaque }
> ### Exercise 5. Counting 'a's
> Write, from scratch, a C program that takes a single argument and prints the number of ascii ‘a’ characters in the string. Don’t use the function `strlen()`; programs using `strlen()` will get 0 points. The purpose of this exercise is in part to give you familiarity with so-called “null-terminated strings” in C. A null-terminated string is a pointer to an array of characters (`char *`), in which the end of the string is indicated by the character `‘\0’`, which has value `0`, or `NULL`.
> 
> For example:
> 
> ```sh
> $ ./countas abbba
> 2
> 
> $ ./countas aaaaa
> 5
> 
> $ ./countas bcdefghwxyz
> 0
> 
> $ ./countas
> usage: countas <arg>
> 
> $ ./countas aaaaa bbbbbb
> usage: countas <arg>
> ```
> 
> Instructions:
> 
> * Again, don’t use `strlen()`.
> * Put all your files in `~/CS5600/lab1/fromscratch/countas`.
> * Your executable file must be named `countas`.
> * If `countas` receives any number of arguments other than 1, it should print a help/usage message (see above).
> * You must supply a `Makefile`. When we type `make`, it needs to create a binary executable named `countas`.
> * Be diligent about creating test cases for yourself. You will lose points for not handling corner cases. You may wish to create a script that runs and tests your code on various cases. (You do not have to hand in your testing code, however; we will run our own testing scripts against your code.)
> * Pay attention to coding style.
> * As above, remember to add relevant files (source file, `Makefile`) to your git repo: `git status ; git add [...] ; git commit -am "countas" ; git push origin`.


# Section 2: Debugging

This part of the lab will give you practice debugging.

Put your answers for this section and in the next in the supplied answers.txt file.

Navigating to syntax errors
Go to the dbg directory in lab1:

```sh
$ cd ~/CS5600-labs/lab1/dbg
```

Now type:

```sh
$ make
```

The code has a syntax error; thus, it cannot be compiled.

{: .exercise }
> {: .opaque }
> ### Exercise 6. Fix the syntax error.
>
> Use the compiler's error message to determine what's wrong. Note that in the vi text editor (discussed below) you can navigate to a given line of code :<number>. You can also launch vi with its cursor in place:
>
> ```sh
> $ vi +<number> <filename>
> ```
> 
> to begin directly on a given line number. For example, `vi +5 foo.c` begins with the cursor at line 5.
> 
> After you fix the syntax error, the code will compile. Use make to see this:
> 
> ```sh
> $ make
> ```
>
> Once it's fixed, state the line of code and the fix in `answers.txt`.
> 
> That's it for Exercise 6. 



Now, try testing the code itself, using the small `test_linked_list` utility:

```sh
$ ./test_linked_list
Segmentation fault (core dumped)
```

Aha! Our code compiled, but it was not correct (core dumps are bad). Specifically, the segmentation fault means that our program issued an illegal memory reference, and the operating system ended our process. Making matters worse, we have no idea what the problem in the code is. In the following section, you will learn how to use gdb to debug this kind of problem.

### Run gdb: Use the GNU debugger, or `gdb` 

Now, we'll use `gdb` to run the program to help us debug it. 

```sh
$ gdb test_linked_list
(gdb)
```
**Set breakpoints:** One thing that you might want to do is to set a breakpoint before the program begins executing. Breakpoints are a way of telling gdb that you want it to execute your program and then stop, or break, at a place that you define. Use the following command to set a breakpoint at the main function:
 
```sh
(gdb) b main
Breakpoint 1 at 0x400963: file test_linked_list.c, line 43.
```

Then use gdb's command `run` to actually start the program (this is the general pattern in gdb: one invokes the debugger, perhaps sets a breakpoint, and then starts the program with run):
```sh
(gdb) run
```

The program will be stopped when it reaches the breakpoint (advanced topic: how does gdb conspire with the hardware to make this work??). At this point, you will be presented with gdb's command prompt again. To see the “call stack” (or stack trace), which is the list of functions that have called this one – literally, the stack frames on top of the current one – you issue backtrace or bt for short:

```sh
(gdb) bt
```
Experienced developers will often ask for a stack trace as step 0 or 1 of understanding a code problem. Get in the habit of asking gdb to give you a backtrace.

To make the program continue running after a breakpoint, use `continue`, or `c` for short:

```sh
(gdb) c
```
Step through the code: Of course, if you just c every time you hit a breakpoint, then you will lose control of the program. You often want the command `next`, or `n`:

```sh
(gdb) n
```
This "executes" the next line of code, for example executing an entire function. (The command step executes a single line of C code. There is little difference between step and next unless you are about to enter a function. step steps into the function; next "steps over" the function.)

Inspect the values of variables: In gdb's command prompt, the program is stalled. You can query the program's current global and local variables with the `print` command, or `p` for short.

Run `gdb` on `test_linked_list`. Set a breakpoint at the function `list_delete`.

At this breakpoint, determine the value of the integer id:

```sh
(gdb) print id
$1 = 1
```
This means that variable `id` holds the integer 1.

Aside: you can check local variables' names using:

```sh
(gdb) info local
```
**Core dump:** If a program terminated abnormally (for example, `test_linked_list`), the state of the program will be recorded by the OS and (if core dumps are enabled) saved in a so-called core dump. gdb can use a core dump to inspect a crash situation.

To debug using core dumps, you must first enable core dumps, and then point gdb at the relevant file. We'll do this in several steps:

```sh
# enable core dumps
$ ulimit -c unlimited 
$ ./test_linked_list
$ ls -l core
$ gdb ./test_linked_list core
```

The idea here is that the core file gives gdb enough information to recover the memory and CPU state of the program at the moment of the crash. This will allow you to determine which instructions experienced the error.



{: .exercise }
> {: .opaque }
> ### Exercise 7. Fix the bug in the code. 
> Recompile and rerun to make sure it is fixed. State the line of code and the fix in `answers.txt`.

# Section 3: Software development skills and general productivity

This part of the lab will walk through general software development skills as well as some tools to enhance your productivity. You may feel at first that these tools are slowing you down ("how do I enter characters in vi???"), but comfort and fluency with them will pay off in the long term. It's worth slowing down and "learning proper technique."

## A good text editor is essential
If you are not doing so already, begin using a powerful text editor. Traditionally on Unix systems, the most popular editors were vi (which in most environments these days is an alias for a more powerful version, called vim) or emacs, both of which are installed in the Docker environment. Of course, many of you will be using VSCode, an IDE that builds in many modern tools for software development.

This section will be with reference to vi. To get started, work through this vi tutorial.

## Navigating code: ctags
We will investigate ctags with the Linux code base as a running example.

Create a new directory inside the Docker environment, and then change into it:

```sh
$ mkdir ~/CS5600-labs/learn-ctags
$ cd !!:$  # !! refers to the last shell command; $ refers to the last argument
```
Now download and unpack the Linux source code archive. We are interested in the kernel source and header files:

```sh
$ wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.2.9.tar.xz # this could take a few mins
$ tar -xf linux-5.2.9.tar.xz linux-5.2.9/include linux-5.2.9/kernel
$ cd linux-5.2.9
$ ls 
include/ kernel/
```

Now, you are going to use the `ctags` program. Type:

```sh
$ ctags --recurse=yes * # this could take time

```

The `ctags` utility searches a body of code and creates a tag file, which is an index of programming language objects, such as function and variable definitions. The command above invokes ctags recursively on all items in the entire directory (`*`) in order to create a single tags file that includes information from all files in the repository.

The rest of this section will use `vi` to interact with tags. However, emacs also supports tags.

Using `vi`, open the header file for the linux scheduler:

```sh
$ vi include/linux/sched.h
```

We will now walk through some uses of tags. When we type `:something`, this refers to a command given in vi's command mode. To get into command mode in vi, you type the escape key (vi by default starts in command mode; the editor enters insert mode when you type `i`, `e`, `a`, etc.). Once in command mode, typing `:something` does the "something". (For example, try pressing escape and then `:q`.)

Now, let's get started with tags. Type:

```
:tag sched_attr
```
This will take you to the definition of `sched_attr`. To return to your previous position, press `ctrl-t`.

The tag file that we created indexed all of Linux's source. We can jump to a tag in another file and it will automatically be opened, as we illustrate in the following exercise.

{: .exercise }
> {: .opaque }
> ### Exercise 8. Type within vi:
> 
> ```
> :tag block_device_operations
> ```
> What file does this open? (You can use the command `:ls` inside of `vi` to check this.) Put  the answer in `answers.txt`.

 
You can use `ctrl-]` to jump to the definition of the object underneath the cursor. Go to  line 1001 in `include/linux/sched.h` (navigate there with `:1001`, since `:<number>` in vi takes you to the specified line number). Move your cursor over the token `futex_pi_state`. Press `ctrl+]`.



{: .exercise }
> {: .opaque }
> ### Exercise 9. `futex_pi_state`
> 
> What is `futex_pi_state`? In what file and line is it defined? Put  the answer in `answers.txt`.


A tag may have multiple definitions. type:

```
:tselect list_head
```
and note that a menu pops up giving you the choice of which definition to view.

Tags can also be used with regular expressions. For example, to look up all tags containing proc_sched, enter the following:

```
:tselect /proc_sched
```
And the following command shows tags that contain `proc_sched` and `task` (in that order):

```sh
:tselect /proc_sched.*task
```

{: .exercise }
> {: .opaque }
> ### Exercise 10. How many functions?
> 
> How many functions does the linux kernel have that contain `proc_sched` and `task`? Where are they defined?
> 
> Put the answer in `answers.txt`. 

For more information about tags, [this resource](http://vim.wikia.com/wiki/Browsing_programs_with_tags) is good.

## General productivity: find
We now cover a very powerful Unix tool: `find`.

At a high level, `find` by default recurses through directories, "looking" at files, to see which files match a provided predicate. Here is the pattern for its use:

```sh
$ find [flags] [path...] [expression]
```
Now `cd` to the root directory of the linux kernel that you downloaded earlier. Type this command:

```sh 
$ find .
.
./kernel
./kernel/panic.c
./kernel/Kconfig.locks
./kernel/params.c
./kernel/.gitignore
./kernel/memremap.c
./kernel/freezer.c
./kernel/sysctl_binary.c
.....
```
This recursively prints all files in the current directory (it will print the names of tens of thousands of files).

You can also search on file name patterns:
```sh 
$ find . -name "sched.h"
./kernel/sched/sched.h
./include/uapi/linux/sched.h
./include/linux/sched.h
./include/linux/sunrpc/sched.h
./include/asm-generic/bitops/sched.h
./include/xen/interface/sched.h
./include/trace/events/sched.h
```
And you can limit the search to particular directories:

```sh
$ find include/linux include/asm-generic -name "sched.h"
include/linux/sched.h
include/linux/sunrpc/sched.h
include/asm-generic/bitops/sched.h
```
As with most Unix commands, wildcards can be used:

```sh
$ find . -name "sched.*"
./kernel/sched/sched.h
./include/uapi/linux/sched.h
./include/linux/sched.h
./include/linux/sunrpc/sched.h
./include/asm-generic/bitops/sched.h
./include/xen/interface/sched.h
./include/trace/events/sched.h
```
This returns all files named "sched" with any extension.

Where `find` really starts to be powerful is that you can execute commands on the files that are returned. For example:

```sh
$ find . -name "sched.h" -exec grep foobar {} \;
```
This says, "return all file names that match `sched.h` and grep each of them for the string foobar."

{: .exercise }
> {: .opaque }
> ### Exercise 11. How many header files are there in the linux source code?
> 
> Hint: The unix program `wc` can be used to count words or lines from a file or standard input. How can we get the output from `find` to `wc`?
> 
> Put the answers in `answers.txt`. 

More information on `find` can be [found here](http://www.binarytides.com/linux-find-command-examples/).

We will cover other tools (such as `grep`) as the semester goes on.

# Handin Procedure

Handing in consists of three steps:

**Step 1:** Do this checklist:

* Fill out the top of the `answers.txt` file, including your name as it appears in Canvas
* Make sure you’ve answered every question in `answers.txt`
* Push your code to GitHub: 
Execute these instructions either from your local machine or within the Docker environment:

```sh
$ cd CS5600-labs/lab1     
$ git commit -am "hand in lab1"
$ git push origin 

Counting objects: ...
....
To ssh://github.com/XXX-CS5600/labs-23sp-<username>.git
  7337116..ceed758  main -> main
```

**Step 2:** Actually submit, by timestamping and identifying your pushed code:

Decide which git commit you want us to grade, and copy its id (you will paste it in the next step). A commit id is a 40-character hexadecimal string. Usually the commit id that you want will be the one that you created last. The easiest way to obtain the commit id for the last commit is by running the command `git log -1 --format=oneline`. This prints both the commit id and the initial line of the commit message. If you want to submit a previous commit, there are multiple ways to get the commit id for an earlier commit. One way is to look at the commit history on GitHub. Another is `git log -p`, as explained [here](http://cs61.seas.harvard.edu/wiki/2014/Git#Logging), or `git show`.

**Step 3:** Submit your commit on Canvas. On Canvas, there will be an entry for this lab. Paste only the commit id that you just copied. You can submit as many times as you want; we will grade the last commit id submitted to Canvas.

NOTE: Ground truth is what and when you submitted to Canvas. Thus, a non-existent commit id in Canvas means that you have not submitted the lab, regardless of what you have pushed to GitHub. And, the time of your submission for the purposes of tracking lateness is the time when you upload the id to Canvas, not the time when you executed git commit.

**This completes the lab.**

