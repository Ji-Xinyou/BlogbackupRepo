---
title: 6.S081 cowlab
date: 2022-02-02 18:09:08
tags: 6.S081
categories: O/S
---

<style>
img{
    width: 80%;
}
</style>

<!--more-->

# 6.S081 Copy-On-Write fork() : COW lab

At this point, I think maybe the lab should be recorded as well.
This record is not really for everyone originally. At least for now, I record this just because I think maybe I will reference here someday for recap.

I will show mainly the core part of the code, and try to explain it clear for both me and potentially you :)

## The requirements

## Background (from xv6 course website)
The fork() system call in xv6 copies all of the parent process's user-space memory into the child. If the parent is large, copying can take a long time. Worse, the work is often largely wasted; for example, a fork() followed by exec() in the child will cause the child to discard the copied memory, probably without ever using most of it. On the other hand, if both parent and child use a page, and one or both writes it, a copy is truly needed.

## Solution (from xv6 course website)
The goal of copy-on-write (COW) fork() is to defer allocating and copying physical memory pages for the child until the copies are actually needed, if ever.

COW fork() creates just a pagetable for the child, with PTEs for user memory pointing to the parent's physical pages. COW fork() marks all the user PTEs in both parent and child as not writable. When either process tries to write one of these COW pages, the CPU will force a page fault. The kernel page-fault handler detects this case, allocates a page of physical memory for the faulting process, copies the original page into the new page, and modifies the relevant PTE in the faulting process to refer to the new page, this time with the PTE marked writeable. When the page fault handler returns, the user process will be able to write its copy of the page.

COW fork() makes freeing of the physical pages that implement user memory a little trickier. A given physical page may be referred to by multiple processes' page tables, and should be freed only when the last reference disappears.

## Implementation

### Add reference count to kalloc.c

#### Add refcount to kmem struct
```C
struct {
  struct spinlock lock;
  struct run *freelist;
  uint64 rc[32768]; // refcount
} kmem;
```
The 32768 here is counted by (PHYSTOP - end) >> 12
It is not exact, but it is close and enough

---

#### Define refcount operations
```C
// return refcount corresponds to the pa
static inline int
__pa2index(void *pa)
{
  // return ((char*)pa - (char*)PGROUNDUP((uint64)end)) / PGSIZE;
  return ((uint64)pa - (uint64)end) >> 12;
}

int
__pa2rc(void *pa)
{
  return kmem.rc[__pa2index(pa)];
}

void
__inc_rc(void *pa)
{
  ++kmem.rc[__pa2index(pa)];
}

void
__dec_rc(void *pa)
{
  if (__pa2rc(pa) == 0)
    return; // already zero
  --kmem.rc[__pa2index(pa)];
}
```

---

#### Modify kalloc() and kfree()
```C
void *
kalloc(void)
{
  struct run *r;

  acquire(&kmem.lock);
  r = kmem.freelist;
  if(r) {
    kmem.freelist = r->next;
    kmem.rc[__pa2index(r)] = 1;
  }
  release(&kmem.lock);

  if(r)
    memset((char*)r, 5, PGSIZE); // fill with junk
  return (void*)r;
}
```

```C
void
kfree(void *pa)
{
  struct run *r;

  if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
    panic("kfree");

  acquire(&kmem.lock);
  __dec_rc(pa);
  if (__pa2rc(pa) > 0) {
    release(&kmem.lock);
    return;
  }
  release(&kmem.lock);

  // Fill with junk to catch dangling refs.
  memset(pa, 1, PGSIZE);

  r = (struct run*)pa;

  acquire(&kmem.lock);
  r->next = kmem.freelist;
  kmem.freelist = r;
  release(&kmem.lock);
}
```

#### initialize refcount in freerange() (I don't think it is necessary)
```C
void
freerange(void *pa_start, void *pa_end)
{
  char *p;
  p = (char*)PGROUNDUP((uint64)pa_start);
  for(; p + PGSIZE <= (char*)pa_end; p += PGSIZE) {
    kmem.rc[__pa2index(p)] = 1; // The reason why it is 1 is that, kfree below decrements it by one
    kfree(p);
  }
}
```

---

### Modify uvmcopy() and clear the PTE_W for both child and parent

The modifications in *uvmcopy()* is here.
```C
int
uvmcopy(pagetable_t old, pagetable_t new, uint64 sz)
{
  pte_t *pte;
  uint64 pa, i;
  // uint flags;
  // char *mem;

  for(i = 0; i < sz; i += PGSIZE){
    if((pte = walk(old, i, 0)) == 0)
      panic("uvmcopy: pte should exist");
    if((*pte & PTE_V) == 0)
      panic("uvmcopy: page not present");
    pa = PTE2PA(*pte);
    if (*pte & PTE_W) // We only COW writable pages
      *pte = (*pte | PTE_C) & ~PTE_W;

    if(mappages(new, i, PGSIZE, pa, PTE_FLAGS(*pte)) != 0) {
      printf("uvmcopy(): bad mappages\n");
      goto err;
    }

    __inc_rc((void*)pa); // increment refcount
    // if((mem = kalloc()) == 0)
    //   goto err;
    // memmove(mem, (char*)pa, PGSIZE);
    // if(mappages(new, i, PGSIZE, (uint64)mem, flags) != 0){
    //   kfree(mem);
    //   goto err;
    // }
  }
  return 0;  
```

*uvmcopy()* is called in the twelfth line of code in fork() (proc.c:273), which is just after *allocproc()*.
The usage of *uvmcopy()* is to copy both `pte and *pte` from parent to child.

Here we modify it to only copy pte, and clear the PTE_W bit
Also, to sign the COW pte, we set the **PTE_COW** to one in child pgtbl

### Modify usertrap() to recognize pagefaults
```C
else if(r_scause() == 0xf) {
    uint64 va = PGROUNDDOWN(r_stval());
    pte_t *pte;

    if (va >= MAXVA || va > p->sz) {
      printf("utrap(): bad va\n");
      p->killed = 1;
      goto end;
    }

    if ((va < PGROUNDDOWN(p->trapframe->sp)) && (va >= PGROUNDDOWN(p->trapframe->sp) - PGSIZE)) {
      printf("utrap(): va in guard page\n");
      p->killed = 1;
      goto end;
    }

    if ((pte = walk(p->pagetable, va, 0)) == 0) {
      printf("utrap(): bad walk\n");
      p->killed = 1;
      goto end;
    }

    if (((*pte & PTE_C) && (*pte & PTE_U) && (*pte & PTE_V)) == 0) {
      printf("utrap(): bad flag bits\n");
      p->killed = 1;
      goto end;
    }

    uint64 pa, npa;
    if((pa = walkaddr(p->pagetable, va)) == 0) {
      printf("utrap(): bad walkaddr()\n");
      p->killed = 1;
      goto end;
    }

    if (__pa2rc((void*)pa) == 1) {
      *pte = (*pte | PTE_W) & ~PTE_C;
    } else {
      if((npa = (uint64)kalloc()) == 0) {
        printf("utrap(): bad kalloc()\n");
        p->killed = 1;
        goto end;
      }
      memmove((void*)npa, (void*)pa, PGSIZE);

      uint64 flag;
      flag = (PTE_FLAGS(*pte) | PTE_W) & ~PTE_C;
      *pte = (*pte | PTE_W) & ~PTE_C;

      // The panic in mappages() is commented, so we do not unmap here
      // uvmunmap(p->pagetable, va, 1, 0);
      if(mappages(p->pagetable, va, PGSIZE, npa, flag) != 0) {
        printf("utrap(): bad mappages()\n");
        p->killed = 1;
        goto end;
      }
      kfree((void*)pa);
    }
  } 
```

---

### Modify Copyout()

At first, I am puzzled for not knowing why I need to modify this function.
The reason why copyout() needs to be modified is that...

The function's usage is to copy memory region **from kernel to userspace** 

Therefore, if we are copying a **COW page**, it is often not **writable**.

However, in userspace, a unwritable page will leads to exceptions.

Therefore, when we meet a **COW and unwritable** page in *copyout()*, we copy it to a new page and map the **user's pte** to the new page.

```C
    if (va0 >= MAXVA)
      return -1;

    pte = walk(pagetable, va0, 0);

    if (pte == 0)
      return -1;
    if ((*pte & PTE_V) == 0)
      return -1;
    if ((*pte & PTE_U) == 0)
      return -1;

    if((*pte & PTE_C) && ((*pte & PTE_W) == 0)) {
      uint64 pa, npa;

      if((pa = walkaddr(pagetable, va0)) == 0) {
        printf("copyout(): bad walkaddr()\n");
        return -1;
      }

      if(__pa2rc((void*)pa) == 1) {
        *pte = (*pte | PTE_W) & ~PTE_C;
      } else {
        if((npa = (uint64)kalloc()) == 0) {
          printf("copyout(): bad kalloc\n");
          return -1;
        }
        memmove((void*)npa, (void*)pa, PGSIZE);

        uint64 flag = (PTE_FLAGS(*pte) | PTE_W) & ~PTE_C;
        *pte = (*pte | PTE_W) & ~PTE_C;

        // uvmunmap(pagetable, va0, 1, 0);
        if(mappages(pagetable, va0, PGSIZE, npa, flag) != 0) {
          printf("copyout(): bad mappages()\n");
          return -1;
        }
        kfree((void*)pa);
      }
    }
```


