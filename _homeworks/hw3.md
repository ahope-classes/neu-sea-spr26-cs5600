---
title: 'Homework 3'
layout: homework
week: 3
released: 2026-01-19
due: 2026-01-30
summary: 'Scheduling. '
---


# Homework 3 
{:.no_toc} 

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Scheduling Game Assumptions

Use these assumptions for the scheduling game in each section of this homework. 

We will see the game in class (if we haven't already). 

Assumptions:

* 1 CPU
* Multiple processes: P1, P2, P3, ...
	* Each process has an arrival time and running time
* Ignore context switch costs
	* unless happening too frequently
* Assume no I/O (This is unrealistic, but helps us build intuition)

Output: 
* A sequence of scheduling decisions


## Shortest Time to Completion First 

Below is an instance of the scheduling game we will play/ have played in class (with the same assumptions, listed above).


Schedule the following processes using STCF scheduling algorithm. 

|    Process | Arrival | Running Time |
|:-------------|:------------------|:------|
|    P1      | 0       | 4 |
|    P2      | 3       | 3 |
|    P3      | 4       | 2 |
|    P4      | 7       | 1 |


### Question 1a (4 pts)

Schedule the following processes 

    0  1  2  3  4  5  6  7  8  9 



### Question 1b (2 point)

What is the average turnaround time? 



### Question 1c (2 point)

What is the average response time? 



## Round Robin

Below is the scheduling game we played in class (with the same assumptions).
Schedule the following processes using RR scheduling algorithm
with **time slice = 2**. 


|    Process | Arrival     | Running time |
|:-------------|:------------------|:------|
|    P1      | 0           | 3 |
|    P2      | 0+\epsilon  | 2 |
|    P3      | 0+2\epsilon | 5 |

### Question 2a (4 points)


Schedule:

    0  1  2  3  4  5  6  7  8  9




### Question 2b (2 points)

What is the average turnaround time? 


### Question 2c (2 points))

What is the average response time?




## Multi-level Feedback Queue

Schedule the processes below using MLFQ.

MLFQ for this question works as follows:
  - There are three queues: `q1`, `q2`, and `q3`.
  - The queues `q1`, `q2` and `q3` have priorities high, medium, and low, respectively
  - Among queues, we run Prio algorithm. (specifically, a process in a lower priority queue runs only when higher priority queues are empty.)
  - `q1` and `q2` run RR with time slice 1 and 2 respectively.
  - `q3` runs FIFO.
  - When a process arrives, it is added at the head of `q1` (namely, the new arrival process will run first).
  - If a process runs for 4 units in one queue, its priority is reduced by one level and the process is moved to the lower queue.


|    Process  | Arrival  | Running time |
|:-------------|:------------------|:------|
|    P1       | 0        | 6 |
|    P2       | 2        | 3 |
|    P3       | 5        | 4 |

### Question 3 (4 pts)

Schedule:


    0  1  2  3  4  5  6  7  8  9  10  11  12

