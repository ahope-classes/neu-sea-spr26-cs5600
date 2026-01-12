---
title: 'Homework 2'
layout: homework
week: 2
released: 2026-01-12
due: 2026-01-23
summary: 'Working with processes. '
---


## Question 1

Read handout week.3a (https://XXX/22spring/notes/handout_w03a.pdf)
and answer the questions below.


### Question 1A

What does `0` in `wait(0)` (line 23) mean? (1 point)

### Question 1B

Read "6. Commentary" and copy-paste it below.  (1 point)

### Question 1C

Why do lines 143 and 144 close the `fdarray`? In other words, what can go
wrong without closing them? (2 points)



## Question 2: Play with `fork`

  -- below are valid code.
  -- header files are omitted.
  -- to see which header files to include, use `$ man <function_name>`
     (e.g., `$ man fork` to see which header files define `fork`.)

### Question 2A 

Run the following code for three times. (2 points)

```c
int main() {
  printf("hello world");
  fork();
  printf("\n");
}
```

Write down the outputs below and answer the question.

Answer:

  first time output:
  """
  // write your output here

  """

  second time output:
  """
  // write your output here

  """

  third time output:
  """
  // write your output here

  """

If "hello world" appear twice in any outputs, explain why in 1-2 sentences.
(write "CONDITION FALSE" if the "hello world" only appear once.)



 Run the following code for three times. (2 points)

```c
int main()
{
    fork();
    fork();
    // [update 2/4: fix the double quotation marks]
    printf("\nhello world");
}
```

Write down the outputs below and answer the question.

Answer:

first time output:
"""
// write your output here

"""

second time output:
"""
// write your output here

"""

third time output:
"""
// write your output here

"""

If the outputs are different, why? Explain in 1-2 sentences. 
(write "CONDITION FALSE" if the outputs are the same)



  1. Run the following code for three times. (2 points)

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

Answer:

first time output:
"""
// write your output here

"""

second time output:
"""
// write your output here

"""

third time output:
"""
// write your output here

"""

If the outputs are the same, explain why in 1-2 sentences.
(write "CONDITION FALSE" if the outputs are different)

