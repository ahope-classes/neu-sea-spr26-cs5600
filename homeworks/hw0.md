---
title: 'Homework 0: Setup'
layout: page
week: 0
date: 2019-04-01
released: 2026-01-06
due: 2026-01-15
---


Answer the following questions.
Submit your answers to Canvas assignments. There is an entry for this homework.

1. C pointer basics.

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


1.a What the output of this program?
(you can copy-paste the code to a file, compile it, and then run)


1.b What does "&" in "bar(&x)" mean? Explain in one sentence.


1.c What does "*" in "int w = *p;" mean? Explain in one sentence.




