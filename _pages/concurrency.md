---
title: "Threads"
permalink: /concurrency/threads/
excerpt: "Concurrency"
toc: true
sidebar:
  nav: "docs"
---
## Processes, Threads and Green Threads

### Process

- Isolated.
- No memory sharing.
- Heavyweight because they need their own memory space, takes time to allocate etc.
- Little communication needed between processes.
- Example is the fork() function in C.
- Golang support for processes is not as good as C/C++. Golang focusses more on threads.

### Thread (native thread/ kernel thread)

- Threads share memory space which makes them more lightweight than processes.
- Spawning threads is faster than spawning processes because we don't have to allocate a lot of resources.
- Communication is required between threads so they don't step on each others toes so to speak.

### Green Thread

- Also called user-level thread.
- More efficient version of a thread.
- context switch overhead: time taken to switch out a process or a thread from the ready cue so that it can be run on the processor. This time is wasted CPU cycles.
- Green threads attempt to limit the context switch overhead. This is only relevant when you have a lot of threads (1000's) because a lot of time is wasted context switching. Not applicable to a coupld threads.
- A normal thread runs at the kernel level, i.e. it's the operating system which decides when to swap it out for another thread. A green thread is a user-level thread meaning it is the program, not the kernel, which decides when to swap the threads. Green threads run inside kernel level threads. Typically you'd have many green threads running inside a kernel-level thread.
- The disadvantage of green threads stems from the fact that the operating systems knows nothing about them. When a green thread needs to read from a drive, the OS realises the thread is waiting for IO and so removes the encapsulating thread from execution. Even though other green threads within may not be waiting but for more CPU work.

### What does Golang use?

- Golang uses a hybrid mixture of kernel-level threads and green threads. It creates a kernel-level thread for each CPU that you have and then a number of green threads within. When a thread is waiting for IO, golang will swap it out but it is clever enough to know the other green threads which are not waiting for IO and shuffles them onto another kernel thread.

- All of the above are implementation details and not needed to write multithreaded applications in golang as it is abstracted away from you.