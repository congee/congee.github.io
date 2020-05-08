+++
title = "Race"
slug = "race-condition"
date = 2020-05-08
[taxonomies]
tags = [
    "data race", "race condition", "out-of-order", "memory ordering",
    "reordering", "thread", "concurrency", "memory barrier", "memory fence"
]
+++

That *io_uring* got merged in Linux 5.1 got me interested. To make fun out of
it, I tried to make a wait-free doubly linked list. It was too difficult that I
went to write a wait-free fixed length ring buffer. Still, I could not
successfully get it right due to three kinds of races. What are they?

## Data Race vs Race Condition

Before we start, we need to know the differences between threads and processes.
Their meanings differ by context. Generally, processes are used for parallelism
while threads for concurrency. A simple example is we have two processes running
on two cores. In one of the processes, we have also have two threads running
alternatively via context swicthing. And, here be dragons!

### Data Race

Two processes modifying the same memory piece causes data race. Specifically,
modifying the memory is done in three steps: fetching data from the memory,
modifying it, and store it back. We don't modify it in-place. To prevent
processors from modifying data in its cache nearby, we can use the `volatile`
keyword to hint compilers to not gerenate that kind of code.

Okay, let us take a dive into data race. Say we have an initialized variable
`a = 0`; And, we want to incement it to 2 by two processes.

| process 1        | process 2        |
| --               | --               |
| (1) lea eax, [a] | (4) lea eax, [a] |
| (2) inc eax      | (5) inc eax      |
| (3) mov eax, [a] | (6) mov eax, [a] |

What are those possible results? 1 and 2. There are many possible orders of
execution. I just list two. The result can be 1 if these instructions are
executed this way 1 -> 2 -> 4 -> 5 -> 3 -> 6. The problem here is (4) get
executed before (3) is finished. So, the `eax` register, whose value held is 1,
is written back to memory. If all these instructions are executed
*sequentially*, 1 -> 2 -> 3 -> 4 -> 5 -> 6, the result obviously is 2 that we
want.

How do we solve data race problems? Lock. We may use a mutex to exclusively use
the processor in a time slice.

| process 1        | process 2        |
| --               | --               |
| Lock             |                  |
| (1) lea eax, [a] |                  |
| (2) inc eax      |                  |
| (3) mov eax, [a] |                  |
| Unlock           |                  |
|                  | (4) lea eax, [a] |
|                  | (5) inc eax      |
|                  | (6) mov eax, [a] |

Luckily, we do not have to always use a high level mutex from libraries like
pthread. There are already fine grained lock primitives provided by processors.
We could just use `std::atomic<int> var = 0; var.fetch_add(1)` This
`std::fetch_add` is an encapsulation of low level instructions (1), (2), (3).
All three instructions are executed sequantially and exclusively by the
processor. So, we always see the result of 2 if we use `std::fetch_add`.

We often see these primitives used in a lock-free algorithm. This is just a
convention as these primitives effectively lock the processor in low level. The
lock is just visible to the hardware. Programmers do not have to explitictly use
a lock in code.

### Race Condition

What is race condition paticularly then? Race condition is really all about one
word: order. It is usually observed in communication with external systems, such
as IO. Let us print two variables `a = 0` and `b = 1`.

| process 1        | process 2        |
| --               | --               |
| (1) lea eax, [a] | (3) lea eax, [b] |
| (2) print eax    | (4) print eax    |

What will be printed on screen? 01 or 10? Easy. We see 01 if instructions are
executed in the order of 1 -> 2 -> 3 ->. The result is 10 if executed in 3 -> 4
-> 1 -> 2.

Data race is a type of race condition. Remember two processes also share one
thing they need to mutate: the screen we see.

Is knowing data race and race condition enough to tackle concurrency problems?
No! That is why I stumbled on implementing wait-free algorithms aforementioned.
There is out-of-order execution.

## Memory Ordering

I just heard about out-of-order execution from some marketings slides of
Intel and AMD and learnt about it a bit from college. Never ever had I thought I
would need to deal with them. It has been true until I try to learn about
wait-free techniques. To my surprise, the instructions in above charts can be
reorderd by either the compiler at compile time of the processor at runtime. Let
us recall our good old friends `a = 0` and `b = 2`. At this time, we increment
and print them in two threads.

| thread 1    | thread 2    |
| --          | --          |
| (1) inc a   | (4) inc b   |
| (2) print a | (5) print b |
| (3) print b | (6) print a |

What could be printed on screen? To make it easy, let us use a mutex.

| thread 1     | thread 2    |
| --           | --          |
| Mutex Lock   |             |
| (1) inc a    |             |
| (2) print a  |             |
| (3) print b  |             |
| Mutex Unlock |             |
|              | (4) inc b   |
|              | (5) print b |
|              | (6) print a |

The results can be:  
1231: 1 -> 2 -> 3 -> 4 -> 5 -> 6  
2232: 1 -> 3 -> 2 -> 4 -> 5 -> 6

Why can (3) *happen before* (2)? Instructions can be *reorered*. For the benefit
of *data parallelism* the processor is free to execute multiple instructions
simultanesouly. Also, a compiler can reorder instructions as long as it sees no
data race. Here is the problem: instructions are run in the order we want! How
can we make the processor run code in the order we want? Use a memory fence,
a.k.a. memory barrier.

| thread      |
| --          |
| (1) inc a   |
| (2) print a |
|   barrier   |
| (3) print b |

Inserting a barrier between code snippets tells the compiler or processor that
the code above the barrier must *happen before* code below the barrier.

### References

1. [io_uring](https://lwn.net/Articles/776703/)
1. [happened-before](https://en.wikipedia.org/wiki/Happened-before)
1. [memory ordering](https://en.wikipedia.org/wiki/Memory_ordering)
1. [sequential consistency](https://www.microsoft.com/en-us/research/publication/make-multiprocessor-computer-correctly-executes-multiprocess-programs/)
1. [out-of-order execution](https://en.wikipedia.org/wiki/Out-of-order_execution)
1. [race condition](https://en.wikipedia.org/wiki/Race_condition)
