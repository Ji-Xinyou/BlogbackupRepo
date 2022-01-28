---
title: '6.S081 lec6: Traps'
date: 2022-01-29 00:13:46
tags: 6.S081
categories: O/S
---

<style>
img{
    width: 80%;
}
</style>

# 6.S081 lec6: Traps

From user space to kernel space

<!--more-->

* Hardware state
  * 32 general-purpose regs
    * saved entirely
    * sp needs to be modifies later
  * pc, mode, satp
    * pc needs to be saved, mode needs to be set
    * **needs to be changed to kernel pgtbl**, user pgtbl does not have the mapping for kernel data
  * stvec
    * point to entry of trap
  * sepc
    * save pc when trap
  * sscratch
    * "tmp" usage reg when trap

---

## Supervisor mode

* R/W Control regs
  * satp, stvec, sepc, sscratch
* use PTEs that don't have the PTE_U flag set

supervisor mode can't do everything
 * e.g. It is forced to go through pgtbl as well.

---

## Shell calls to write()

* high level path
![](../images/6S081/shwrite.png)

### gdb debugging the code

* In qemu-gdb, `^A + C` will get into qemu monitor
  * info mem will print the pgtbl
![](../images/6S081/pgtblofsh.png)
* text page, data page, guard page(no PTE_U bit), stack page, TRAPFRAME page and TRAMPOLINE page

---

After the kernel runs **ecall**, this is called by **write()**(in usys.S)
**ecall does three things**
* saves `pc` to `sepc`
* changes mode to `supervisor`
* jump to `stvec`

#### enters uservec()
* pc -> TRAMPOLINE page
  * the registers are the original one, cannot be modifies **until we have saved them**
  * save a0 to sscratch, save sscratch to a0
    * a0 now **points at TRAPFRAME**
    * sscratch saves a0
  
Since ecall does not switch pgtbl, the first instructions of trap needs to present in every user's pagetable. (By the TRAMPOLINE page)
* controlled by *stvec*
  * **before entering userspace, kernel** set it to 0x3ffffff000 (TRAMPOLINE)

---

**OK! After ecall, we jump to stvec's address, entering uservec()! Prepare for it.**

---

#### in uservec()
**TRAPFRAME page**
* **32 slots + some interesting values**
  * **kernel_satp, kernel_sp, kernel_trap, epc, kernel_hartid**
    * There are **previously setup** by kernel when mapping trapframe for each process.
  * 32 slots is to save register when trapping
* **before entering userspace, kernel** puts the address of the trapframe in sscratch.
  * since we swapped, sscratch and a0, now a0 points at the trapframe

```asm
(gdb) print/x $a0
$1 = 0x3fffffe000 # trapframe addr, used to be in sscratch

(gdb) print/x $sscratch
$2 = 0x2 # first argument of write() syscall, which is the fd 2
```

from info above, we know that the kernel_satp, sp, trap, epc, hartid are set by kernel **previously**

so in **uservec()**
```asm
...
ld sp, 8(a0)
ld tp, 32(a0) # core_nr: hartid
ld t0, 16(a0) # address of usertrap()
ld t1, 0(a0)  # pgtbl addr of kernel

csrw satp, t1 # from now on, the va are interpretted by kernel pgtbl
sfence.vma zero, zero

jr t0         # jump to usertrap()
```

##### One last thing!
Why don't we crash! 
**pc is holding some va, and we switched pgtbl from user's to kernel's**
Why the next instuction is correctly executed instead of some garbage being run.

**This is because the va of TRAMPOLINE is THE SAME in userspace and kernel space!!!! It is in 0x3ffffff000.**

This is so damn important, always keep this in mind!

---

**OK! WE ARE NOW EXITING FROM uservec(), GOING INTO usertrap()**

---

#### in usertrap() (C code)



---

### IMPORTANT THINGS TO FIGURE OUT

1. **Why do we need TRAMPOLINE page**

    I think this is because, when we trap from userspace, we are using user's pgtbl. To **run the code helping us jump into kernel space**, we need this code text be mapped in every user process' pgtbl. And this is the TRAMPOLINE page.

---

2. **Why do we need TRAPFRAME page**
   
   I think this is because, when we enter the trampoline page, we are still in user's va space. However, we need to save the user's registers. **Given the context that, first we are using user's mapping, and second we gotta save the registers**, we need some space mapped in user process' pgtbl to save them. And this is the TRAPFRAME page.

---