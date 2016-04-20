(2)(spoc) 理解调度算法支撑框架的执行过程

即在ucore运行过程中通过cprintf函数来完整地展现出来多个进程在调度算法和框架的支撑下，在相关调度点如何动态调度和执行的细节。(越全面细致越好)

请完成如下练习，完成代码填写，并形成spoc练习报告

```
#include <defs.h>
#include <list.h>
#include <proc.h>
#include <assert.h>
#include <default_sched.h>

#define USE_SKEW_HEAP 1

/* You should define the BigStride constant here*/
/* LAB6: YOUR CODE */
#define BIG_STRIDE    0x7FFFFFFF /* ??? */

/* The compare function for two skew_heap_node_t's and the
 * corresponding procs*/
static int
proc_stride_comp_f(void *a, void *b)
{
     struct proc_struct *p = le2proc(a, lab6_run_pool);
     struct proc_struct *q = le2proc(b, lab6_run_pool);
     int32_t c = p->lab6_stride - q->lab6_stride;
     if (c > 0) return 1;
     else if (c == 0) return 0;
     else return -1;
}

/*
 * stride_init initializes the run-queue rq with correct assignment for
 * member variables, including:
 *
 *   - run_list: should be a empty list after initialization.
 *   - lab6_run_pool: NULL
 *   - proc_num: 0
 *   - max_time_slice: no need here, the variable would be assigned by the caller.
 *
 * hint: see proj13.1/libs/list.h for routines of the list structures.
 */
static void
stride_init(struct run_queue *rq) {
     /* LAB6: YOUR CODE */
     cprintf("[In stride_init]\n");
     list_init(&(rq->run_list));
     rq->lab6_run_pool = NULL;
     rq->proc_num = 0;
     cprintf("[Out stride_init]\n");
}

/*
 * stride_enqueue inserts the process ``proc'' into the run-queue
 * ``rq''. The procedure should verify/initialize the relevant members
 * of ``proc'', and then put the ``lab6_run_pool'' node into the
 * queue(since we use priority queue here). The procedure should also
 * update the meta date in ``rq'' structure.
 *
 * proc->time_slice denotes the time slices allocation for the
 * process, which should set to rq->max_time_slice.
 * 
 * hint: see proj13.1/libs/skew_heap.h for routines of the priority
 * queue structures.
 */
static void
stride_enqueue(struct run_queue *rq, struct proc_struct *proc) {
     /* LAB6: YOUR CODE */
     cprintf("[In stride_enqueue]\n");
#if USE_SKEW_HEAP
     rq->lab6_run_pool =
          skew_heap_insert(rq->lab6_run_pool, &(proc->lab6_run_pool), proc_stride_comp_f);
     cprintf("Insert PID == %d into the skew heap.\n", proc->pid);
     cprintf("PID == %d 's stride == %u.\n", proc->pid, proc->lab6_stride);
#else
     assert(list_empty(&(proc->run_link)));
     list_add_before(&(rq->run_list), &(proc->run_link));
#endif
     if (proc->time_slice == 0 || proc->time_slice > rq->max_time_slice) {
          proc->time_slice = rq->max_time_slice;
     }
     proc->rq = rq;
     rq->proc_num ++;
     cprintf("[Out stride_enqueue]\n");
}

/*
 * stride_dequeue removes the process ``proc'' from the run-queue
 * ``rq'', the operation would be finished by the skew_heap_remove
 * operations. Remember to update the ``rq'' structure.
 *
 * hint: see proj13.1/libs/skew_heap.h for routines of the priority
 * queue structures.
 */
static void
stride_dequeue(struct run_queue *rq, struct proc_struct *proc) {
     cprintf("[In stride_dequeue]\n");
     /* LAB6: YOUR CODE */
#if USE_SKEW_HEAP
     rq->lab6_run_pool =
          skew_heap_remove(rq->lab6_run_pool, &(proc->lab6_run_pool), proc_stride_comp_f);
     cprintf("Remove PID == %d from the skew heap.\n", proc->pid);
     cprintf("PID == %d 's stride == %u.\n", proc->pid, proc->lab6_stride);
#else
     assert(!list_empty(&(proc->run_link)) && proc->rq == rq);
     list_del_init(&(proc->run_link));
#endif
     rq->proc_num --;
     cprintf("[Out stride_dequeue]\n");
}
/*
 * stride_pick_next pick the element from the ``run-queue'', with the
 * minimum value of stride, and returns the corresponding process
 * pointer. The process pointer would be calculated by macro le2proc,
 * see proj13.1/kern/process/proc.h for definition. Return NULL if
 * there is no process in the queue.
 *
 * When one proc structure is selected, remember to update the stride
 * property of the proc. (stride += BIG_STRIDE / priority)
 *
 * hint: see proj13.1/libs/skew_heap.h for routines of the priority
 * queue structures.
 */
static struct proc_struct *
stride_pick_next(struct run_queue *rq) {
     cprintf("[In stride_pick_next]\n");
     /* LAB6: YOUR CODE */
#if USE_SKEW_HEAP
     cprintf("Picking next from a skew heap.\n");
     if (rq->lab6_run_pool == NULL) { 
         cprintf("No process to pick (NULL).\n");
         return NULL; 
     }
     struct proc_struct *p = le2proc(rq->lab6_run_pool, lab6_run_pool);
     cprintf("Pick PID == %d.\n", p->pid);
     cprintf("PID == %d 's stride == %u.\n", p->pid, p->lab6_stride);
#else
     list_entry_t *le = list_next(&(rq->run_list));

     if (le == &rq->run_list)
          return NULL;
     
     struct proc_struct *p = le2proc(le, run_link);
     le = list_next(le);
     while (le != &rq->run_list)
     {
          struct proc_struct *q = le2proc(le, run_link);
          if ((int32_t)(p->lab6_stride - q->lab6_stride) > 0)
               p = q;
          le = list_next(le);
     }
#endif
     if (p->lab6_priority == 0)
          p->lab6_stride += BIG_STRIDE;
     else p->lab6_stride += BIG_STRIDE / p->lab6_priority;
     cprintf("[Out stride_pick_next]\n");
     return p;
}

/*
 * stride_proc_tick works with the tick event of current process. You
 * should check whether the time slices for current process is
 * exhausted and update the proc struct ``proc''. proc->time_slice
 * denotes the time slices left for current
 * process. proc->need_resched is the flag variable for process
 * switching.
 */
static void
stride_proc_tick(struct run_queue *rq, struct proc_struct *proc) {
     cprintf("[In stride_pick_tick]\n");
     cprintf("old time_slice == %d\n", proc->time_slice);
     /* LAB6: YOUR CODE */
     if (proc->time_slice > 0) {
          proc->time_slice --;
     }
     if (proc->time_slice == 0) {
          proc->need_resched = 1;
     }
     cprintf("new time_slice == %d\n", proc->time_slice);
     cprintf("[Out stride_pick_tick]\n");
}

struct sched_class default_sched_class = {
     .name = "stride_scheduler",
     .init = stride_init,
     .enqueue = stride_enqueue,
     .dequeue = stride_dequeue,
     .pick_next = stride_pick_next,
     .proc_tick = stride_proc_tick,
};

(THU.CST) os is loading ...

Special kernel symbols:
  entry  0xc010002a (phys)
  etext  0xc010da7c (phys)
  edata  0xc01b1dd4 (phys)
  end    0xc01b4f78 (phys)
Kernel executable memory footprint: 724KB
ebp:0xc012ff38 eip:0xc0101f67 args:0x00010094 0x00000000 0xc012ff68 0xc01000d8 
    kern/debug/kdebug.c:350: print_stackframe+21
ebp:0xc012ff48 eip:0xc0102256 args:0x00000000 0x00000000 0x00000000 0xc012ffb8 
    kern/debug/kmonitor.c:129: mon_backtrace+10
ebp:0xc012ff68 eip:0xc01000d8 args:0x00000000 0xc012ff90 0xffff0000 0xc012ff94 
    kern/init/init.c:58: grade_backtrace2+33
ebp:0xc012ff88 eip:0xc0100101 args:0x00000000 0xffff0000 0xc012ffb4 0x0000002a 
    kern/init/init.c:63: grade_backtrace1+38
ebp:0xc012ffa8 eip:0xc010011f args:0x00000000 0xc010002a 0xffff0000 0x0000001d 
    kern/init/init.c:68: grade_backtrace0+23
ebp:0xc012ffc8 eip:0xc0100144 args:0xc010da9c 0xc010da80 0x000031a4 0x00000000 
    kern/init/init.c:73: grade_backtrace+34
ebp:0xc012fff8 eip:0xc010007f args:0x00000000 0x00000000 0x0000ffff 0x40cf9a00 
    kern/init/init.c:32: kern_init+84
memory management: default_pmm_manager
e820map:
  memory: 0009fc00, [00000000, 0009fbff], type = 1.
  memory: 00000400, [0009fc00, 0009ffff], type = 2.
  memory: 00010000, [000f0000, 000fffff], type = 2.
  memory: 07efe000, [00100000, 07ffdfff], type = 1.
  memory: 00002000, [07ffe000, 07ffffff], type = 2.
  memory: 00040000, [fffc0000, ffffffff], type = 2.
check_alloc_page() succeeded!
check_pgdir() succeeded!
check_boot_pgdir() succeeded!
-------------------- BEGIN --------------------
PDE(0e0) c0000000-f8000000 38000000 urw
  |-- PTE(38000) c0000000-f8000000 38000000 -rw
PDE(001) fac00000-fb000000 00400000 -rw
  |-- PTE(000e0) faf00000-fafe0000 000e0000 urw
  |-- PTE(00001) fafeb000-fafec000 00001000 -rw
--------------------- END ---------------------
use SLOB allocator
check_slab() success
kmalloc_init() succeeded!
check_vma_struct() succeeded!
page fault at 0x00000100: K/W [no page found].
check_pgfault() succeeded!
check_vmm() succeeded.
[In stride_init]
[Out stride_init]
sched class: stride_scheduler
[In stride_enqueue]
Insert PID == 1 into the skew heap.
PID == 1 's stride == 0.
[Out stride_enqueue]
ide 0:      10000(sectors), 'QEMU HARDDISK'.
ide 1:     262144(sectors), 'QEMU HARDDISK'.
SWAP: manager = fifo swap manager
BEGIN check_swap: count 31813, total 31813
setup Page Table for vaddr 0X1000, so alloc a page
setup Page Table vaddr 0~4MB OVER!
set up init env for check_swap begin!
page fault at 0x00001000: K/W [no page found].
page fault at 0x00002000: K/W [no page found].
page fault at 0x00003000: K/W [no page found].
page fault at 0x00004000: K/W [no page found].
set up init env for check_swap over!
write Virt Page c in fifo_check_swap
write Virt Page a in fifo_check_swap
write Virt Page d in fifo_check_swap
write Virt Page b in fifo_check_swap
write Virt Page e in fifo_check_swap
page fault at 0x00005000: K/W [no page found].
swap_out: i 0, store page in vaddr 0x1000 to disk swap entry 2
write Virt Page b in fifo_check_swap
write Virt Page a in fifo_check_swap
page fault at 0x00001000: K/W [no page found].
do pgfault: ptep c03b9004, pte 200
swap_out: i 0, store page in vaddr 0x2000 to disk swap entry 3
swap_in: load disk swap entry 2 with swap_page in vadr 0x1000
write Virt Page b in fifo_check_swap
page fault at 0x00002000: K/W [no page found].
do pgfault: ptep c03b9008, pte 300
swap_out: i 0, store page in vaddr 0x3000 to disk swap entry 4
swap_in: load disk swap entry 3 with swap_page in vadr 0x2000
write Virt Page c in fifo_check_swap
page fault at 0x00003000: K/W [no page found].
do pgfault: ptep c03b900c, pte 400
swap_out: i 0, store page in vaddr 0x4000 to disk swap entry 5
swap_in: load disk swap entry 4 with swap_page in vadr 0x3000
write Virt Page d in fifo_check_swap
page fault at 0x00004000: K/W [no page found].
do pgfault: ptep c03b9010, pte 500
swap_out: i 0, store page in vaddr 0x5000 to disk swap entry 6
swap_in: load disk swap entry 5 with swap_page in vadr 0x4000
write Virt Page e in fifo_check_swap
page fault at 0x00005000: K/W [no page found].
do pgfault: ptep c03b9014, pte 600
swap_out: i 0, store page in vaddr 0x1000 to disk swap entry 2
swap_in: load disk swap entry 6 with swap_page in vadr 0x5000
write Virt Page a in fifo_check_swap
page fault at 0x00001000: K/R [no page found].
do pgfault: ptep c03b9004, pte 200
swap_out: i 0, store page in vaddr 0x2000 to disk swap entry 3
swap_in: load disk swap entry 2 with swap_page in vadr 0x1000
count is 5, total is 5
check_swap() succeeded!
++ setup timer interrupts
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 1.
PID == 1 's stride == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 1 from the skew heap.
PID == 1 's stride == 2147483647.
[Out stride_dequeue]
[In stride_enqueue]
Insert PID == 2 into the skew heap.
PID == 2 's stride == 0.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 2.
PID == 2 's stride == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 2 from the skew heap.
PID == 2 's stride == 2147483647.
[Out stride_dequeue]
kernel_execve: pid = 2, name = "exit".
I am the parent. Forking the child...
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 0.
[Out stride_enqueue]
I am parent, fork a child pid 3
I am the parent, waiting now..
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 2147483647.
[Out stride_dequeue]
I am the child.
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 2147483647.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 2147483647.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 4294967294.
[Out stride_dequeue]
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 4294967294.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 4294967294.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 2147483645.
[Out stride_dequeue]
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 2147483645.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 2147483645.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 4294967292.
[Out stride_dequeue]
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 4294967292.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 4294967292.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 2147483643.
[Out stride_dequeue]
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 2147483643.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 2147483643.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 4294967290.
[Out stride_dequeue]
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 4294967290.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 4294967290.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 2147483641.
[Out stride_dequeue]
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 2147483641.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 2147483641.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 4294967288.
[Out stride_dequeue]
[In stride_enqueue]
Insert PID == 2 into the skew heap.
PID == 2 's stride == 2147483647.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 2.
PID == 2 's stride == 2147483647.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 2 from the skew heap.
PID == 2 's stride == 4294967294.
[Out stride_dequeue]
waitpid 3 ok.
exit pass.
[In stride_enqueue]
Insert PID == 1 into the skew heap.
PID == 1 's stride == 2147483647.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 1.
PID == 1 's stride == 2147483647.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 1 from the skew heap.
PID == 1 's stride == 4294967294.
[Out stride_dequeue]
[In stride_enqueue]
Insert PID == 1 into the skew heap.
PID == 1 's stride == 4294967294.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 1.
PID == 1 's stride == 4294967294.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 1 from the skew heap.
PID == 1 's stride == 2147483645.
[Out stride_dequeue]
all user-mode processes have quit.
init check memory pass.
kernel panic at kern/process/proc.c:460:
    initproc exit.

Welcome to the kernel debug monitor!!
Type 'help' for a list of commands.
```




请回答如下问题：为什么只用32位整数存stride的数值就够了？溢出时如何判断大小？

答：设当前进程中最小的```stride```值为```s```，则目前所有进程的```stride```值一定在区间```[s, s+PASS_MAX]```中（若不考虑溢出），如果```s+PASS_MAX```溢出，由于```STRIDE_MAX – STRIDE_MIN <= PASS_MAX```，故区间溢出的部分一定不会和没溢出的部分重合，因此可以判断彼此大小（溢出部分一定比没溢出部分的```stride```值大）。
