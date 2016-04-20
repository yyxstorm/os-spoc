(2)(spoc) 理解调度算法支撑框架的执行过程

即在ucore运行过程中通过cprintf函数来完整地展现出来多个进程在调度算法和框架的支撑下，在相关调度点如何动态调度和执行的细节。(越全面细致越好)

打印出来的信息如下，都括在方括号里。由这些信息可以知道，schedule会在处理idle、wait、exit等等时被调用。ucore里的stride值其实应该是原论文里的pass值，而原论文里的stride在ucore里是用BIG_STRIDE/priority计算出来的。
```
(THU.CST) os is loading ...

Special kernel symbols:
  entry  0xc010002a (phys)
  etext  0xc010db39 (phys)
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
ebp:0xc012ffc8 eip:0xc0100144 args:0xc010db5c 0xc010db40 0x000031a4 0x00000000 
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
PID == 1 's priority == 0.
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
[Call schedule() in cpu_idle]
[In schedule]
Here we are scheduling!
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 1.
PID == 1 's stride == 0.
PID == 1 's priority == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 1 from the skew heap.
PID == 1 's stride == 2147483647.
PID == 1 's priority == 0.
[Out stride_dequeue]
[In stride_enqueue]
Insert PID == 2 into the skew heap.
PID == 2 's stride == 0.
PID == 2 's priority == 0.
[Out stride_enqueue]
[Call schedule() in do_wait]
[In schedule]
Here we are scheduling!
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 2.
PID == 2 's stride == 0.
PID == 2 's priority == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 2 from the skew heap.
PID == 2 's stride == 2147483647.
PID == 2 's priority == 0.
[Out stride_dequeue]
kernel_execve: pid = 2, name = "exit".
I am the parent. Forking the child...
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 0.
PID == 3 's priority == 0.
[Out stride_enqueue]
I am parent, fork a child pid 3
I am the parent, waiting now..
[Call schedule() in do_wait]
[In schedule]
Here we are scheduling!
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 0.
PID == 3 's priority == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 2147483647.
PID == 3 's priority == 0.
[Out stride_dequeue]
I am the child.
[In schedule]
Here we are scheduling!
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 2147483647.
PID == 3 's priority == 0.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 2147483647.
PID == 3 's priority == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 4294967294.
PID == 3 's priority == 0.
[Out stride_dequeue]
[Out schedule]
[In schedule]
Here we are scheduling!
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 4294967294.
PID == 3 's priority == 0.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 4294967294.
PID == 3 's priority == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 2147483645.
PID == 3 's priority == 0.
[Out stride_dequeue]
[Out schedule]
[In schedule]
Here we are scheduling!
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 2147483645.
PID == 3 's priority == 0.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 2147483645.
PID == 3 's priority == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 4294967292.
PID == 3 's priority == 0.
[Out stride_dequeue]
[Out schedule]
[In schedule]
Here we are scheduling!
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 4294967292.
PID == 3 's priority == 0.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 4294967292.
PID == 3 's priority == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 2147483643.
PID == 3 's priority == 0.
[Out stride_dequeue]
[Out schedule]
[In schedule]
Here we are scheduling!
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 2147483643.
PID == 3 's priority == 0.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 2147483643.
PID == 3 's priority == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 4294967290.
PID == 3 's priority == 0.
[Out stride_dequeue]
[Out schedule]
[In schedule]
Here we are scheduling!
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 4294967290.
PID == 3 's priority == 0.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 4294967290.
PID == 3 's priority == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 2147483641.
PID == 3 's priority == 0.
[Out stride_dequeue]
[Out schedule]
[In schedule]
Here we are scheduling!
[In stride_enqueue]
Insert PID == 3 into the skew heap.
PID == 3 's stride == 2147483641.
PID == 3 's priority == 0.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 3.
PID == 3 's stride == 2147483641.
PID == 3 's priority == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 3 from the skew heap.
PID == 3 's stride == 4294967288.
PID == 3 's priority == 0.
[Out stride_dequeue]
[Out schedule]
[In stride_enqueue]
Insert PID == 2 into the skew heap.
PID == 2 's stride == 2147483647.
PID == 2 's priority == 0.
[Out stride_enqueue]
[Call schedule() in do_exit]
[In schedule]
Here we are scheduling!
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 2.
PID == 2 's stride == 2147483647.
PID == 2 's priority == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 2 from the skew heap.
PID == 2 's stride == 4294967294.
PID == 2 's priority == 0.
[Out stride_dequeue]
[Out schedule]
waitpid 3 ok.
exit pass.
[In stride_enqueue]
Insert PID == 1 into the skew heap.
PID == 1 's stride == 2147483647.
PID == 1 's priority == 0.
[Out stride_enqueue]
[Call schedule() in do_exit]
[In schedule]
Here we are scheduling!
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 1.
PID == 1 's stride == 2147483647.
PID == 1 's priority == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 1 from the skew heap.
PID == 1 's stride == 4294967294.
PID == 1 's priority == 0.
[Out stride_dequeue]
[Out schedule]
[Call schedule() in do_main]
[In schedule]
Here we are scheduling!
[In stride_enqueue]
Insert PID == 1 into the skew heap.
PID == 1 's stride == 4294967294.
PID == 1 's priority == 0.
[Out stride_enqueue]
[In stride_pick_next]
Picking next from a skew heap.
Pick PID == 1.
PID == 1 's stride == 4294967294.
PID == 1 's priority == 0.
[Out stride_pick_next]
[In stride_dequeue]
Remove PID == 1 from the skew heap.
PID == 1 's stride == 2147483645.
PID == 1 's priority == 0.
[Out stride_dequeue]
[Out schedule]
all user-mode processes have quit.
init check memory pass.
kernel panic at kern/process/proc.c:460:
    initproc exit.

Welcome to the kernel debug monitor!!
Type 'help' for a list of commands.
```




请回答如下问题：为什么只用32位整数存stride的数值就够了？溢出时如何判断大小？

答：假设我们知道（已经存了）当前进程中最小的```stride```值（记为```s```），则目前所有进程的```stride```值一定在区间```[s, s+PASS_MAX]```中（若不考虑溢出），如果```s+PASS_MAX```溢出，由于```STRIDE_MAX – STRIDE_MIN <= PASS_MAX```，故区间溢出的部分一定不会和没溢出的部分重合，因此可以判断彼此大小（溢出部分一定比没溢出部分的```stride```值大）。

上面的方法基于储存了当前```stride```最小值的实现，但实际上我们不需要存下当前```stride```的最小值```s```，因为在一个时钟周期之前，```stride```的最小值是当前运行进程的```stride-pass```值，所以在一次时间周期之后当前进程```stride```的最小值一定有下界（=当前运行进程的```stride-pass```值），我们可以使用这个值作为```s```的估计。
