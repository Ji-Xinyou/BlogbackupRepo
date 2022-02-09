---
title: 6.S081-lec10 Multiprocesses and Locks
date: 2022-02-07 17:20:09
tags: 6.S081
categories: O/S
---
<style>
img{
    width: 80%;
}
</style>

<!--more-->

# 6.S081-lec10 Multiprocesses and Locks
---

In Chinese. This post is short.

## order the locks

CPU1: `rename ("d1/x, d2/y")`
CPU2: `rename ("d2/a, d1/b")`

CPU1先 `acquire(d1)`再`acquire(d2)`
CPU2则 `acquire(d2)`再`acquire(d1)`
会导致死锁

解决方法就是我们将所有的锁给定一个**global order**，获取这类型的多个锁的时候必须按顺序获取。 **order**是对于**某个类型的锁**而言的，不同类型的锁之间没有order可言。

### function calls
func1() calls func2()
if *func2()* 使用了一些lock，*func1()* 应该知道*func2()*使用了哪些锁以保证global order

## Implementation

### BROKEN acquire()

```C
void
broken_acquire(struct lock *l)
{   
    while(1) {
        if (l->locked == 0) {
            l->locked = 1;
            return;
        }
    }
}
```
这里有race condition，可能有两个process都看到lock is not held，并且同时获得lock

### GOOD acquire()

**remember to close the interrupt**

**Using hw inst**, test_and_set() atomically

`amoswap(addr, r1, r2)` in RISC-V

* hw locks the addr
* puts `tmp <- *addr`
* `*addr <- r1`
* `r2 <- tmp`
* unlocks and return

In C stdlib, `__sync_lock_test_and_set(void* addr, uint64 val)`，do the job

Following code does the work
`while(__sync_lock_test_and_set(&lk->locked, 1) != 0) {}`

## Memory Ordering

```C
locked = 1
    x = x + 1
locked = 0
```
CPU会乱序执行，我们可以把`x = x + 1`放到`locked = 0`后面吗？

在并行执行中，这是不OK的，因此有`fence`，去告诉编译器与cpu不要这么做
`__sync_synchronize()`就是这个作用

`__sync_synchronize()` tells the C compiler and the CPU to not move loads or stores past this point.

1. to ensure all stores in the critical section are **visible** to other CPUs before the lock is released, and loads in the critical section occur strictly before the lock is released

2. to ensure the critical section's memory references happen strictly after the lock is acquired

换句话说，在`fence`前的指令不会被重排到`fence`后面，因此`acquire()`前的指令仍然在前面，critical section的指令仍然在锁之间。







