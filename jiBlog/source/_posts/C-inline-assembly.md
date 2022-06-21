---
title: C-inline-assembly
date: 2022-06-08 20:18:11
tags: C
categories: Programming Languages
---

<div></div>

<!--more-->

# Inline Assembly Syntax in C

> A useful website relevant: https://www.codeproject.com/Articles/15971/Using-Inline-Assembly-in-C-C

[toc]

The general syntax of inline assembly of C is following

```C
asm volatile("assembly code"
    : output operands /* optional */
    : input operands  /* optional */
    : clobbered registers /* optional */);
```

## Example 1 (single operand)
```C
asm volatile ("movl %%eax, %0;" : "=r(val)");
```
This inline assembly reads the value of **eax**, and pass it to variable **val**.
We can see that here val is output operand.
The **equal sign(=)** means the value is a output operand
> r is a contraint, we use **m, v, o** to represent memory unit, **r** to represent any register,
**q** to represent one of **eax, ebx, ecx, edx**, **i, h** to represent immediate, **E, F** to represent float, **g** to represent any, **a, b, c, d** to represent **eax, ebx, ecx or edx(respectively)**, **S, D** to represent **esi and edi(respectively)**, **I** to represent constant(0 to 31).

| constraint | meaning |
| ---- | ---- |
| m, v, o | memory unit |
| r | any register |
| q | one of eax, ebx, ecx, edx |
| i, h | immediate |
| E, F | float |
| g | any |
| a, b, c, d | eax, ebx, ecx, edx (respectively) |
| S, D | esi, edi(respectively) |
| I | constant |

## Example 2 (multiple operands)
```C
int no=0xb, val;
asm volatile ("movl %1, &&ebx;"
              "movl %%ebx, %0;"
              : "=r"(val)
              : "r" (no)
              : "%ebx");
```
In this example, we are using **ebx** as a temp register, thus we put **ebx** to **clobbered regs**
- First we read from **no**, pass it to **ebx**.
- Next we pass **ebx** to **val**
Since **val** is the output operand, we are using "=r".
Vice versa for **no

## Example 3 (not freq use, use contraint)
```C
int arg1, arg2, add ;
asm volatile ( "addl %%ebx, %%eax;"
                : "=a" (add)
                : "a" (arg1), "b" (arg2) );
```
Here **a** means contraint for **eax**, vice versa for **b**
Therefore, input operands **arg1 goes to eax**, **arg2 goes to ebx**
And the result goes into add, since it is an output operand.

