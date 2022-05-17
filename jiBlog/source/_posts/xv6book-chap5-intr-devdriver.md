---
title: xv6book-chap5-intr&devdriver
date: 2022-02-15 16:23:51
tags: 6.S081
categories: O/S
---

<div></div>

<!--more-->



# XV6 book - chap6: interrupt and device drivers

---

## Driver

**Definition:**

- **code** in an **operating system** that manages a particular device: it **configures** the device hardware, tells the device to **perform** operations, handles the resulting **interrupts**, and **interacts** with processes that may be waiting for I/O from the device

---

### Top half and bottom half

* Top half
  * Runs in process's kernel thread
    * called via **syscalls**, e.g. write/read that want device to perform I/O
* Bottom half
  * executes at intr
    * called via intr, acts as **handler**

---



