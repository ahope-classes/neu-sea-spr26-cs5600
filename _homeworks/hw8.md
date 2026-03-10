---
title: 'Homework 8'
layout: homework
week: 10
released: 2026-03-09
due: 2026-03-20
summary: 'Disk performance. '
---




# 1. Disk performance

We have a special disk, spec'd especially for CS 5600. 

Here are the specs: 

- Spindle Speed: 12000 RPM
	* On average, it takes the disk half a rotation to find a sector

- Avg Seek Time, read/write: 5ms / 6 ms
	* This is the time to move the head to the right track.

- Transfer rate: 64 MB/s
	* This is the rate of reading/writing sequential data.
	* Assume 1MB=10^6B 

When CS5600-Disk reads/writes a sector (512 Bytes), it needs to
1. move the head to the right track,
1. wait rotating to the right sector,
1. read/write the data.

Questions:
## Q1 A
How long would it take to do 500 sector reads, spread out randomly over the disk (and serviced in FIFO order)?  (2 points)
(Show your steps and write the final result in seconds with two decimal place accuracy.)



## Q1 B
How long would it take to do 500 sector writes, spread out randomly over the disk (and serviced in FIFO order)?  (2 points)
(Show your steps and write the final result in seconds with two decimal place accuracy.)


## Q1 C
How long would it take to do 500 sector reads, SEQUENTIALLY on the CS5600-Disk? (FIFO order once more) (2 points)
(Show your steps and write the final result in milliseconds with one decimal place accuracy.)




# 2. SSD

Now we have an SSD that has 1 flash bank, 4 blocks (in the same bank), and 8 pages (2 in each block).
This SSD uses the log-structured FTL we learn in class.

The current state of the SSD is as follows:

```
         +---------------------------------------+
 blocks  | block 0 | block 1 | block 2 | block 3 |
         +---------+---------+---------+---------+
 pages   | P1 | P2 | P3 | P4 | P5 | P6 | P7 | P8 |
         +----+----+----+----+----+----+----+----+
 data    | D  | B  | C  | A' |    |    |    |    |
         +----+----+----+----+----+----+----+----+
```
 mapping: 
 * `A'` => P4 
 * `B` => P2
 * `C` => P3
 * `D` => P1

- `A`, `B`, `C`, and `D` are four logical pages.
- Whenever a page gets updated, we add an apostrophe ("`'`") to their names.
	- For example, after an update to page `A`, `A` becomes `A'`.
- In other words, (`A`, `A'`, `A''`, `A'''`,...) is a series of snapshots to page `A`.
   But they refer to the same logical page (from a program's point of view).

Questions:

## Q2A (2 pts)
Draw the CS5600-SSD status (including the mapping) after running

* write(`B'`)
* write(`C'`)
* write(`A''`)
* write(`B''`)

You can use this as a starting point if you want: 


```
         +---------------------------------------+
 blocks  | block 0 | block 1 | block 2 | block 3 |
         +---------+---------+---------+---------+
 pages   | P1 | P2 | P3 | P4 | P5 | P6 | P7 | P8 |
         +----+----+----+----+----+----+----+----+
 data    | D  | B  | C  | A' |    |    |    |    |
         +----+----+----+----+----+----+----+----+
```

Mapping: 
* 
* 
* 
* 



## Q2B (2 pts)
Now CS5600-SSD runs a round of garbage collection, recycling all blocks that do not contain valid pages. (Assume this is a naive GC)

Draw the SSD status after the garbage collection.

If you'd like you can use this as a starting point: 


```
         +---------------------------------------+
 blocks  | block 0 | block 1 | block 2 | block 3 |
         +---------+---------+---------+---------+
 pages   | P1 | P2 | P3 | P4 | P5 | P6 | P7 | P8 |
         +----+----+----+----+----+----+----+----+
 data    |    |    |    |    |    |    |    |    |
         +----+----+----+----+----+----+----+----+
```



