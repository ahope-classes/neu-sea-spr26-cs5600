---
title: 'Homework 4'
layout: homework
week: 4
released: 2026-01-26
due: 2026-02-06
summary: 'Threads vs Processes, Race Conditions'
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# 1. Threads vs. Processes (4 points)

A thread within a process has its own: (Choose all that apply)

1. stack
1. main() function
1. registers
1. global variables
1. program code
1. heap



# 2. Race conditions (6 points)

Below is a function that adds two input values and assigns the sum
to the first variable.

```c
void add(int *a, int *b) {
	*a += *b;
}
```

Given `a` and `b` are two `int` variables that initially are 1, what might happen when one thread runs `add(&a,&b)` while another runs `add(&b,&a)`?

Assume:
- memory satisfies sequential consistency.
- C code "X += Y" will be compiled as four instructions:
	1. move X from memory to a register %rbx
	1. move Y from memory to a register %rcx
	1. add %rbx and %rcx and assign the sum to %rbx
	1. move the value of %rbx to X in memory

Write down all possible final values of `a` and `b`.



# 3. multi-threading counter

Below is a piece of code that is intended to do the following:
- there will be 100 threads;
- each thread increases a global counter, `counter`, by 1 each time;
- each thread does `increase-by-1` 10000 times;
- after 100 threads finish, the program print the final value of counter.

Here is the incomplete code with two TODOs:

```c
// file name: counter.c
#include "pthread.h"
#include "stdio.h"

int counter = 0; // global variable

void *count(void *ignored) {
    for (int i=0; i<10000; i++)
        counter++;
    return NULL;
}

int main(){
    pthread_t tid[100];   // 100 threads
    for (int i=0; i<100; i++) {
        // TODO1: your code here
        // you should create threads to run function "count"
    }
    for (int i=0; i<100; i++) {
        // TODO2: your code here
        // you should "wait" for all 100 threads to finish
    }
    printf("counter  = %d\n", counter);
}
```

## 3.a. fill "TODO1" to create threads; write your solution below. (4 points)
[hint: `man pthread_create` to see how to create threads using `pthread`.]





## 3.b  fill "TODO2" to wait all threads; write your solution below. (4 points)
[hint: `man pthread_join` to see how to wait threads using `pthread`.]



## 3.c. compile and run the code; write down the output you've seen. (2 point)
[hint: `gcc counter.c -lpthead` to compile with `pthread` library]


