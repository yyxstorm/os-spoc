(2) 12.2的第5题：
请在ucore启动时显示空闲线程（idleproc）和初始进程(initproc)的进程控制块中的“pde_t *pgdir”的内容。它们是否一致？为什么？由2013011384完成此题。

```c
void
cpu_idle(void) {
    cprintf("\n-*- 0x%x 0x%x -*-\n", idleproc->cr3, initproc->cr3);
    while (1) {
        if (current->need_resched) {
            schedule();
        }
    }
}
```

输出（相关部分）：
```
-*- 0x274000 0x274000 -*-
```
之所以相同，是因为idleproc和initproc都是直接使用的操作系统启动时使用的那个页表。

(4) 12.4的第2题：
试分析ucore操作系统内核是如何把子进程exit()的返回值传递给父进程wait()的？由2013011380完成此题。

答：子进程```exit```会把返回值传给系统调用```sys_exit```，接着调用```proc.c```中的```do_exit```函数，在```do_exit```函数中返回值会被存至子进程```PCB```的```exit_code```中，最后```do_wait```函数会把```exit_code```中存的返回值通过```code_store```返回给```wait```的调用者（通过系统调用的函数参数返回）。
