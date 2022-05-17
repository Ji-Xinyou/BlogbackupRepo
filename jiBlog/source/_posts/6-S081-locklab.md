---
title: 6-S081-locklab
date: 2022-02-17 15:18:03
tags: 6.S081
categories: O/S
---

<div></div>

<!--more-->



# 6.S081 lab lock

---

## Memory Allocator (Moderate)

### Goal

1. implement per-CPU freelist
2. steal from other CPU's if
   1. we dont have free mem
   2. Others have

### before impl

focus on the loop # printed

```shell
$ kalloctest
start test1
test1 results:
--- lock kmem/bcache stats
lock: kmem: #test-and-set 179515 #acquire() 433086
lock: bcache: #test-and-set 0 #acquire() 1190
--- top 5 contended locks:
lock: kmem: #test-and-set 179515 #acquire() 433086
lock: proc: #test-and-set 85325 #acquire() 306058
lock: proc: #test-and-set 70155 #acquire() 307351
lock: proc: #test-and-set 62155 #acquire() 307485
lock: pr: #test-and-set 60051 #acquire() 5
tot= 179515
test1 FAIL
```

### impl

for implementing each hart a freelist, first I implemented a **naive way of doing it**

- each *steal()* only steals a **page**



* **BUGGY**: although hartid is shown as 1, 2, ...., not from zero.
  * **BUT**, actually, *cpuid()* return value **IS FROM ZERO**
    * WHICH MEANS, kmems[hartid] is ok, we don't need to make it kmems[hartid - 1]

#### naive steal

```c
// @usage: steal free mem from others
// @param: hart: caller's hartid
struct run*
steal(int hartid)
{
  /** 
    * reason why caller's hart is leaked to the interface:
    *   when steal() is called, lets say called by hartid 1
    *   if the thread is scheduled to **another CPU**, the running hart of *steal()*
    *   is not necessarily the original one, this is because of the time interval
    *   between steal() is called and cpuid() is actually called
    */
  struct run *r;
  int i;

  // push_off();
  // hartid = cpuid();
  // pop_off();

  for(i = 0; i < NCPU; i++) {
    if (i == hartid)
      continue;

    acquire(&kmems[i].lock);
    if (kmems[i].freelist) {
      r = kmems[i].freelist;
      kmems[i].freelist = r->next;
      release(&kmems[i].lock);
      return r;
    }
    release(&kmems[i].lock);
  }
  return 0;
```

Other modifications of naive steal()

```c
struct run* steal();

#define LEN_LOCKNAME  16
struct kmem {
  struct spinlock lock;
  char lockname[LEN_LOCKNAME];
  struct run *freelist;
};

struct kmem kmems[NCPU]; /* per-CPU freelist */

void
kinit()
{
  int i;

  // name allocation
  for(i = 0; i < NCPU; ++i) {
    // first initialize the space, then the name
    snprintf(kmems[i].lockname, LEN_LOCKNAME, "kmem_%d\0", i);
  }

  // lock init: 
  // name format: kmem_0, kmem_2, ..., kmem_$(NCPU - 1)
  for(i = 0; i < NCPU; ++i) {
    initlock(&kmems[i].lock, kmems[i].lockname);
  }

  // freerange calls kfree(), giving free pages to freelist
  // however, only current running (i.e. one) cpu can get free mem
  // therefore, if multiple cores are started, the second core has no free mem
  // thats before **steal()** if implemented
  freerange(end, (void*)PHYSTOP);
}
```

There are not really any important modification in *kalloc()* and *kfree()*, to sum up

> Get the hartid

> Acquire corresponding hart's freelist lock

> Update the freelist

> Release the lock

#### After naive steal impl

`tot` is way better

`#test-and set, #acquire()` are cut straight half

But we can tell that virtio_disk is often accessed, which is bad

```shell
$ kalloctest
start test1
test1 results:
--- lock kmem/bcache stats
lock: kmem_0: #test-and-set 0 #acquire() 40185
lock: kmem_1: #test-and-set 0 #acquire() 196413
lock: kmem_2: #test-and-set 2491 #acquire() 196530
lock: bcache: #test-and-set 0 #acquire() 1190
--- top 5 contended locks:
lock: proc: #test-and-set 78309 #acquire() 247272
lock: proc: #test-and-set 47267 #acquire() 248566
lock: pr: #test-and-set 43958 #acquire() 5
lock: proc: #test-and-set 42334 #acquire() 248584
lock: virtio_disk: #test-and-set 32495 #acquire() 69
tot= 2491
test1 FAIL
start test2
total free number of pages: 32499 (out of 32768)
.....
test2 OK
```

### better impl

We use fast-slow ptr to cut half the freelist, so that *steal()* will be called less frequently

```c
struct run*
steal(int hartid)
{
  struct run *r, *fast, *ret;
  int i;

  for(i = 0; i < NCPU; i++) {
    if (i == hartid)
      continue;

    acquire(&kmems[i].lock);
    if (kmems[i].freelist) {
      // for a better impl, we use fast-slow ptr to cur half the freelist
      r = kmems[i].freelist;
      ret = kmems[i].freelist; /* ret is to save the head */
      fast = r->next;

      while (fast) {
        fast = fast->next;
        if (fast) {
          r = r->next;
          fast = fast->next;
        }
      }
      kmems[i].freelist = r->next;
      r->next = 0;
      release(&kmems[i].lock);
      return ret;
    }
    release(&kmems[i].lock);
  }
  return 0;
}
```



**Corresponding kalloc()**

remember to take care of the case that **steal() returns 0**

```c
void *
kalloc(void)
{
  struct run *r;
  int hartid;

  // acquire current cpu #
  push_off(); // disable interrupt
  hartid = cpuid();
  pop_off();  // cpuid acquired, enable interrupt

  acquire(&kmems[hartid].lock);
  r = kmems[hartid].freelist;
  if(r)
    kmems[hartid].freelist = r->next; 
  else { 
    // we steal it and returns
    r = steal(hartid);
    if(r)
      kmems[hartid].freelist = r->next;
  }
  release(&kmems[hartid].lock);

  if(r)
    memset((char*)r, 5, PGSIZE); // fill with junk
  return (void*)r;
}
```

#### After cut half impl

`tot` is 0 now

Others performance is almost the same

but `bcache` is less frequently accessed

```shell
$ kalloctest
start test1
test1 results:
--- lock kmem/bcache stats
lock: kmem_0: #test-and-set 0 #acquire() 40022
lock: kmem_1: #test-and-set 0 #acquire() 196217
lock: kmem_2: #test-and-set 0 #acquire() 196779
lock: bcache: #test-and-set 0 #acquire() 334
--- top 5 contended locks:
lock: proc: #test-and-set 124961 #acquire() 224765
lock: proc: #test-and-set 93420 #acquire() 224884
lock: proc: #test-and-set 66190 #acquire() 224749
lock: pr: #test-and-set 51880 #acquire() 5
lock: virtio_disk: #test-and-set 31856 #acquire() 57
tot= 0
test1 OK
start test2
total free number of pages: 32499 (out of 32768)
.....
test2 OK
```

---

## Buffer Cache(hard)

Modify `bget` and `brelse` so that concurrent lookups and releases for different blocks that are in the bcache are unlikely to conflict on locks (e.g., don't all have to wait for `bcache.lock`). You must maintain the invariant that at most one copy of each block is cached. 



This lab is very harsh on lock order, look **CAREFULLY AT THE COMMENTS**

```c
#define HASH(x) (x % NBUCKET)
struct {
  struct spinlock lock;
  struct buf buf[NBUF];

  struct buf bkt[NBUCKET]; // each bucket has a doubly-linked list
  struct spinlock bktlk[NBUCKET];

} bcache;

void
binit(void)
{
  struct buf *b;
  int i;

  initlock(&bcache.lock, "bcache");

  // Create linked list of buffers
  for(i = 0; i < NBUCKET; i++){
    initlock(&bcache.bktlk[i], "bcache_bkt");
    bcache.bkt[i].next = &bcache.bkt[i];
    bcache.bkt[i].prev = &bcache.bkt[i];
  }

  for(b = bcache.buf; b < bcache.buf+NBUF; b++){
    initsleeplock(&b->lock, "buffer");
    // hang all to the first bucket
    b->next = bcache.bkt[0].next;
    b->prev = &bcache.bkt[0];
    bcache.bkt[0].next->prev = b;
    bcache.bkt[0].next = b;
    b->ts = ticks;
  }
}

// Look through buffer cache for block on device dev.
// If not found, allocate a buffer.
// In either case, return locked buffer.
static struct buf*
bget(uint dev, uint blockno)
{
  struct buf *b;
  int bktno = HASH(blockno), i;

  // ---------- FIRST TIME CHECK ---------- //
  // Is the block already cached?
  acquire(&bcache.bktlk[bktno]);
  for(b = bcache.bkt[bktno].next; b != &bcache.bkt[bktno]; b = b->next){
    if(b->dev == dev && b->blockno == blockno){
      b->refcnt++;
      release(&bcache.bktlk[bktno]);
      acquiresleep(&b->lock);
      return b;
    }
  }
  release(&bcache.bktlk[bktno]);
  // ---------- FIRST TIME CHECK ---------- //

  // ---------- EVICTION ---------- //
  // evict a block, global lock used to serialize the eviction
  acquire(&bcache.lock);

  // WHY DOUBLE CHECK?
  // If two threads come here simutaneously, one add the block to cache already
  // but here, another thread will add the same block to cache again
  // RESULT: one block, cached by two buffer cache blocks 
  acquire(&bcache.bktlk[bktno]);
  for(b = bcache.bkt[bktno].next; b != &bcache.bkt[bktno]; b = b->next){
    if(b->dev == dev && b->blockno == blockno){
      b->refcnt++;
      release(&bcache.bktlk[bktno]);
      release(&bcache.lock);
      acquiresleep(&b->lock);
      return b;
    }
  }
  release(&bcache.bktlk[bktno]);

  // ---------- acutal Eviction ---------- // 
  /**
   * lo: lowest buffer slot
   * lo_ts: lowest timestamp
   * lo_bkt: lowest slot's bucket 
   */
  struct buf *lo = 0;
  uint lo_ts = ~0;
  int lo_bkt = -1;

  // traverse all buckets, globally find the lowest timestamp block
  for(i = 0; i < NBUCKET; i++){
    acquire(&bcache.bktlk[i]);
    int found = 0;
    for(b = bcache.bkt[i].next; b != &bcache.bkt[i]; b = b->next){
      if((b->refcnt == 0) && (b->ts < lo_ts)) {
        if(lo){
          // this round, we found a new one, release the old lock
          lo_bkt = HASH(lo->blockno);
          if(lo_bkt != i)
            release(&bcache.bktlk[lo_bkt]);
        }
        lo_ts = b->ts;
        lo = b;
        found = 1;
      }
    }

    // At this line, we still hold one lock --> current lo_bkt's lock

    if(!found)
      release(&bcache.bktlk[i]);
    // if found, meaning that lo_bkt = i
    // which will be release when the next lo is found or the loop is over
  }

  if(!lo)
    panic("bget: no buffers");
  
  // detach it from the old bucket list
  if (HASH(lo->blockno) != bktno){
    lo->next->prev = lo->prev;
    lo->prev->next = lo->next;
  }

  // we have done the modification of the old bucket list
  release(&bcache.bktlk[HASH(lo->blockno)]);

  // attach it to the new bucket list
  if(HASH(lo->blockno) != bktno){
    acquire(&bcache.bktlk[bktno]);
    lo->next = bcache.bkt[bktno].next;
    lo->prev = &bcache.bkt[bktno];
    bcache.bkt[bktno].next->prev = lo;
    bcache.bkt[bktno].next = lo;
    release(&bcache.bktlk[bktno]);
  }

  // update at last, since older blockno was still in use
  lo->dev = dev;
  lo->blockno = blockno;
  lo->valid = 0;
  lo->refcnt = 1;
  lo->ts = ticks;

  release(&bcache.lock);
  acquiresleep(&lo->lock);

  return lo;
}
```



### Several things important in buffer cache

1. **DOUBLE CHECK** when you evict! - **THIS TO IMPORTANT TO BE OVER-EXAGGERATED!!!!!!**
   * the reason why is stated in the comment
     * if one thread add the block to cache, another onemight also do so, resulting in one block in two cache blocks
2. **HOLD THE LOWEST BLOCK UTIL YOU DETACH IT OUT**
   * When looping all blocks, only release the older lowest bucket lock when you find **a new lowest block**, and hold that lock.
     * When you find the globally lowest block, detach it before you release the lock 
3. Update the blockno at **LAST**, at least after you are done with it
