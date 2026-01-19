---
title: 'Homework 2'
layout: homework
week: 2
released: 2026-01-12
due: 2026-01-23
summary: 'Working with processes. '
---


## Question 1

Reference the handout from class in Week 2 ([Linked here after class](https://XXX/handout.pdf)) and answer the questions below.


### Question 1A

#### 2 pts

Refer to the simple shell codebox (https://ahope-classes.github.io/neu-sea-spr26-cs5600/notes/week2/#a-very-simple-shell).

What does `0` in `wait(0)` mean? 

{: .note-title }
> Answer
>
>
>
>
>
>
>
> .


### Question 1B
#### 4 pts

Refer to the Putting it all together codebox (https://ahope-classes.github.io/neu-sea-spr26-cs5600/notes/week2/#putting-it-all-together). 

Why does the parent process close the two file descriptors in `fdarray`? In other words, what can go wrong without closing them? 

Before looking up answers to this question, think it through given what we know: 

* The `fdarray` contains 2 file descriptors
* The `pipe` command creates the pipe, populating `fdarray` with the two ends of the pipe
   * Feel free to use `man pipe2` on the command line in your class Docker to understand more
* One process is the child process, one is the parent process

{: .note-title }
> Answer
>
>
>
>
>
>
>
> .



## Question 2: Play with `fork`

* The code below is valid
* The header files are omitted
	* To see which header files to include, use `$ man <function_name>` (e.g., `$ man fork` to see which header files define `fork`.)

### Question 2A 
#### 4 pts

Run the following code for three times. 

```c
int main() {
  printf("hello world");
  fork();
  printf("\n");
}
```

Write down the outputs below and answer the question.


{: .note-title }
> Answer
>
> First time output
>
>
>
>
>
>
> .


{: .note-title }
> Answer
>
> Second time output
>
>
>
>
>
>
> .


{: .note-title }
> Answer
>
> Third time output
>
>
>
>
>
>
> .

 
If "hello world" appear twice in any outputs, explain why in 1-2 sentences.
(write "CONDITION FALSE" if the "hello world" only appear once.)


{: .note-title }
> Answer
>
>
>
>
>
>
>
> .


### Question 2B
#### 4 pts
 Run the following code for three times. 

```c
int main()
{
    fork();
    fork();
    printf("\nhello world");
}
```

Write down the outputs below and answer the question.

{: .note-title }
> Answer
>
> First time output
>
>
>
>
>
>
> .

{: .note-title }
> Answer
>
> Second time output
>
>
>
>
>
>
> .


{: .note-title }
> Answer
>
> Third time output
>
>
>
>
>
>
> .


If the outputs are different, why? Explain in 1-2 sentences. 
(write "CONDITION FALSE" if the outputs are the same)

{: .note-title }
> Answer
>
>
>
>
>
>
>
> .


### Question 2C 
#### 4 pts

Run the following code for three times. 

```c
int main() {
    int pid = fork();
    if (pid > 0) {
        wait(NULL);
    }

    int pid2 = fork();
    if (pid2 > 0) {
        wait(NULL);
    }

    printf("\nhello world");
}
```

Write down the outputs below and answer the question.


{: .note-title }
> Answer
>
> First time output
>
>
>
>
>
>
> .


{: .note-title }
> Answer
>
> Second time output
>
>
>
>
>
>
> .


{: .note-title }
> Answer
>
> Third time output
>
>
>
>
>
>
> .

If the outputs are the same, explain why in 1-2 sentences.
(write "CONDITION FALSE" if the outputs are different)


{: .note-title }
> Answer
>
>
>
>
>
>
>
> .
