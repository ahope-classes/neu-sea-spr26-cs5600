---
title: 'Homework 1'
layout: homework
week: 1
released: 2026-01-08
due: 2026-01-16
---


## Overview


## Questions



### Q1. C pointer basics.

Below is a piece of valid C code. Read it and answer questions:
```c
#include<stdio.h>

void foo(int p) {
    p = p + 1;
}

void bar(int *p) {
    int w = *p;
    w = w + 1;
}

void baz(int *p) {
    *p = *p + 1;
}

int main() {
    int x=0;
    foo(x);
    printf("foo: %d\n", x);
    bar(&x);
    printf("bar: %d\n", x);
    baz(&x);
    printf("baz: %d\n", x);
}
```


#### Q1a What the output of this program?
(you can copy-paste the code to a file, compile it, and then run)


#### Q1b What does '&' in `bar(&x)` mean? Explain in one sentence.


#### Q1c What does `*` in `int w = *p;` mean? Explain in one sentence.









