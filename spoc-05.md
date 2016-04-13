# lec5 SPOC思考题

## 小组思考题

请参考xv6（umalloc.c），ucore lab2代码，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在Linux应用程序/库层面，用C、C++或python来实现malloc/free，给出你的设计思路，并给出可以在Linux上运行的malloc/free实现和测试用例。 (spoc)

答：（1）基于Method 2的最先匹配方案：
```c
#include <stdio.h>
#include <stddef.h>
#include <unistd.h>

typedef long Align;

union header {
  struct {
    union header *ptr; /* Points to the header of the next *free* block. */
    size_t size;       /* Size of the current block, including header. */
  } s;
  Align x; /* For the sake of alignment. */
};

typedef union header Header;

static Header *g_free_list;

static void *internal_free(void *ptr) {
  Header *bp, *p;

  /* Minus 1, so we are now at the header of the block. */
  bp = ((Header *)ptr) - 1;

  /* Find the given block's neighbors, in terms of address. */
  for (p = g_free_list; !(bp > p && bp < p->s.ptr); p = p->s.ptr)
    if (p >= p->s.ptr && (bp > p || bp < p->s.ptr))
      break;

  /* See if bp and p->s.ptr can be merged. */
  if (p->s.ptr != g_free_list && bp + bp->s.size == p->s.ptr) {
    /* Merge p->s.ptr into bp. */
    bp->s.size += p->s.ptr->s.size;
    bp->s.ptr = p->s.ptr->s.ptr;
  } else
    /* Let it be. */
    bp->s.ptr = p->s.ptr;

  /* See if p and bp can be merged. */
  if (p != g_free_list && p + p->s.size == bp) {
    /* Merge bp->s.size into p. */
    p->s.size += bp->s.size;
    p->s.ptr = bp->s.ptr;
    bp = p;
  } else
    /* Let it be */
    p->s.ptr = bp;

  /* Return the address, useful when merging happens. See request_space(). */
  return (void *)bp;
}

void _free(void *ptr) { internal_free(ptr); }

static Header *request_space(size_t nu) {
  void *p;
  Header *hp;

  /* Calling too much sbrk will be expensive! */
  if (nu < 4096)
    nu = 4096;

  /* Grow the heap. */
  p = sbrk(nu * sizeof(Header));

  /* If failed to grow the heap. */
  if (p == (void *)-1)
    return 0;

  /* Succeeded, so this is the newly created block. */
  hp = (Header *)p;
  hp->s.size = nu;

  /* Need to mark the new block as free, so it can later be used.
   * And it might be merged into its neighbors.
   * Add one to skip the header. */
  return internal_free((void *)(hp + 1));
}

void *_malloc(size_t nbytes) {
  Header *p, *prv;
  size_t nu;

  /* Reserve extra space for a header. Round up. */
  nu = (nbytes + sizeof(Header) - 1) / sizeof(Header) + 1;

  /* If this is the first time we call malloc, initialize the list head. */
  if ((prv = g_free_list) == 0) {
    prv = g_free_list = sbrk(sizeof(Header));
    if (g_free_list == (void *)-1)
      return 0;
    g_free_list->s.ptr = g_free_list;
    g_free_list->s.size = 1;
  }

  /* Manage memory with *First Fit*. */
  for (p = prv->s.ptr;; prv = p, p = p->s.ptr) {

    /* A large enough block is found. */
    if (p->s.size >= nu) {

      /* It just fit, so move it out of the free list. */
      if (p->s.size == nu)
        prv->s.ptr = p->s.ptr;

      /* Otherwise split, since there is more than needed. */
      else {
        /* The first part becomes a new free block. */
        p->s.size -= nu;
        /* Give the second part to the user. */
        p += p->s.size;
        p->s.size = nu;
      }

      /* Add 1 to skip the header. */
      return (void *)(p + 1);
    }

    /* Sadly, no space! */
    if (p == g_free_list) {
      if (request_space(nu) == 0)
        return 0;
    }
  }
}

int main() {
  int i, *a, *b, *c;

  a = _malloc(42692 * sizeof(int));
  b = _malloc(52993 * sizeof(int));
  for (i = 0; i < 52993; i++)
    b[i] = 7;
  c = _malloc(59332 * sizeof(int));
  _free(b);
  b = _malloc(49004 * sizeof(int));
  printf("%d\n", b[0]);
  _free(a);
  _free(b);
  _free(c);
  return 0;
}
```

（2）基于Method 2的最优匹配，相比上面的代码，只需要修改malloc部分：
```c
void *_malloc(size_t nbytes) {
  Header *p, *prv, *best;
  size_t nu;

  /* Reserve extra space for a header. Round up. */
  nu = (nbytes + sizeof(Header) - 1) / sizeof(Header) + 1;

  /* If this is the first time we call malloc, initialize the list head. */
  if ((prv = g_free_list) == 0) {
    prv = g_free_list = sbrk(sizeof(Header));
    if (g_free_list == (void *)-1)
      return 0;
    g_free_list->s.ptr = g_free_list;
    g_free_list->s.size = 1;
  }

  /* Find the Best-Fit! */
  best = 0;
  for (p = prv->s.ptr; p->s.ptr != g_free_list; prv = p, p = p->s.ptr) {
    if (p->s.size >= nu) {
      if (p->s.size == nu) {
        best = p;
        break;
      } else {
        if (p->s.size < best->s.size)
          best = p;
      }
    }
  }
  if (best != 0) {
    p = best;
    if (p->s.size == nu)
      prv->s.ptr = p->s.ptr;
    else {
      p->s.size -= nu;
      p += p->s.size;
      p->s.size = nu;
    }
    return (void *)(p + 1);
  }

  /* Manage memory with *First Fit*. */
  for (p = prv->s.ptr;; prv = p, p = p->s.ptr) {

    /* A large enough block is found. */
    if (p->s.size >= nu) {

      /* It just fit, so move it out of the free list. */
      if (p->s.size == nu)
        prv->s.ptr = p->s.ptr;

      /* Otherwise split, since there is more than needed. */
      else {
        /* The first part becomes a new free block. */
        p->s.size -= nu;
        /* Give the second part to the user. */
        p += p->s.size;
        p->s.size = nu;
      }

      /* Add 1 to skip the header. */
      return (void *)(p + 1);
    }

    /* Sadly, no space! */
    if (p == g_free_list) {
      if (request_space(nu) == 0)
        return 0;
    }
  }
}
```

（3）基于Method 2的最差匹配，相比上面（2）的代码，只需要修改malloc部分里的比较条件：
```c
void *_malloc(size_t nbytes) {
  Header *p, *prv, *best;
  size_t nu;

  /* Reserve extra space for a header. Round up. */
  nu = (nbytes + sizeof(Header) - 1) / sizeof(Header) + 1;

  /* If this is the first time we call malloc, initialize the list head. */
  if ((prv = g_free_list) == 0) {
    prv = g_free_list = sbrk(sizeof(Header));
    if (g_free_list == (void *)-1)
      return 0;
    g_free_list->s.ptr = g_free_list;
    g_free_list->s.size = 1;
  }

  /* Find the Best-Fit! */
  best = 0;
  for (p = prv->s.ptr; p->s.ptr != g_free_list; prv = p, p = p->s.ptr) {
    if (p->s.size >= nu) {
      if (p->s.size > best->s.size)
        best = p;
    }
  }
  if (best != 0) {
    p = best;
    if (p->s.size == nu)
      prv->s.ptr = p->s.ptr;
    else {
      p->s.size -= nu;
      p += p->s.size;
      p->s.size = nu;
    }
    return (void *)(p + 1);
  }

  /* Manage memory with *First Fit*. */
  for (p = prv->s.ptr;; prv = p, p = p->s.ptr) {

    /* A large enough block is found. */
    if (p->s.size >= nu) {

      /* It just fit, so move it out of the free list. */
      if (p->s.size == nu)
        prv->s.ptr = p->s.ptr;

      /* Otherwise split, since there is more than needed. */
      else {
        /* The first part becomes a new free block. */
        p->s.size -= nu;
        /* Give the second part to the user. */
        p += p->s.size;
        p->s.size = nu;
      }

      /* Add 1 to skip the header. */
      return (void *)(p + 1);
    }

    /* Sadly, no space! */
    if (p == g_free_list) {
      if (request_space(nu) == 0)
        return 0;
    }
  }
}
```

```
- [基于malloc与free函数的实现代码及分析](http://www.jb51.net/article/36391.htm)
如何表示空闲块？ 如何表示空闲块列表？ 
[(start0, size0),(start1,size1)...]
在一次malloc后，如果根据某种顺序查找符合malloc要求的空闲块？如何把一个空闲块改变成另外一个空闲块，或消除这个空闲块？如何更新空闲块列表？
在一次free后，如何把已使用块转变成空闲块，并按照某种顺序（起始地址，块大小）插入到空闲块列表中？考虑需要合并相邻空闲块，形成更大的空闲块？
如果考虑地址对齐（比如按照4字节对齐），应该如何设计？
如果考虑空闲/使用块列表组织中有部分元数据，比如表示链接信息，如何给malloc返回有效可用的空闲块地址而不破坏
元数据信息？
伙伴分配器的一个极简实现
http://coolshell.cn/tag/buddy
```

## 其它

- 静态分配内存与动态分配内存的区别是啥？
- 答：是否可以预先知道要分配的内存的大小和其实地址（静态分配时需要已知，动态分配时大小由用户给出）。
 
- 在隐式链表结构中，如何实现向前的合并操作？
- 答：如果从头沿着链搜索下去，可行但是时间开销较大。可以考虑多加个tailer，tailer里也记录block size，这样就可以从当前块推出前一块的头部了。

- 如何判断一个函数是库函数还是一个系统调用？
- 答：可以使用strace，比如我要看f()是否系统调用，我就让f()的上一句、下一句都是一个我已知的系统调用，这样我从strace按时间的输出中就可以看到f是否有系统调用。
 
- 为何要在OS内部和用户态中实现两层动态内存分配？
- 答：OS部分的动态内存分配协调的各个进程之间的内存需求，而用户态这一级主要是为了满足一个进程中多个变量对内存的需求。这种分层设计，简化了实现。

- 为何在OS内部没有采用GC分配方式？
- 答：进程消亡了内存自然也就随即回收，并没有采用GC的需要（我的意思是，就算采用了GC也不会带来设计上的简化，反倒是GC本身很难实现好）。
