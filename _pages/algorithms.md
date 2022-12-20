---
title: "Big O"
permalink: /algo/bigo/
excerpt: "Algorithms"
toc: true
sidebar:
  nav: "docs"
---

## What?

Big O is a method to represent the performance of your code.

### Why do we need it?

Measuring the performance of your code in real time can be cumbersome. Different machines will record different times and for fast algorithms, speed measurements may not be precise enough. 
Big O is a way to make an assesment of code that gives a general idea of it's performance characteristics.

### How?

Rather than counting seconds, which are so variable...we use the number of simple operations the computer has to be perform.

### Counting Operations

Take the below example snippet for adding integers up to `n`:

```golang
func addUpTo(int n) int {
  return n * (n + 1) / 2
}
```
...

This code contains 3 simple operations, regardless of the size of `n`:
- 1 Multiplication
- 1 Addition
- 1 Division

Now take this solution:

```golang
func addUpTo(int n) int {
  total := 0
  for i := 0; i < n; i++ {
    total += i
  }
  return total
}
```
