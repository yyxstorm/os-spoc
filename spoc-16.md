#### 14.2(2):尝试在panic函数中获取并输出用户栈和内核栈的函数嵌套信息，然后在你希望的地方人为触发panic函数，并输出上述信息。

触发panic的地点选择trap函数，因为能保证会有用户态进程进来这里。在trap函数的中间增加：
```c
if (current == NULL) {
    // 省略
} else {
    if (tf->tf_cs == USER_CS)  // 判断是否为用户态过来的
        panic("Panic: I am again overwhelmed.\n");
    // 省略
}
``` 
再在panic的实现__panic函数中增加```print_stackframe();```即可打印出堆栈信息。如下：
```
ebp:0xc03adf48 eip:0xc0101f62 args:0x00000000 0x00000000 0x00000000 0x00000000 
    kern/debug/kdebug.c:350: print_stackframe+21
ebp:0xc03adf78 eip:0xc0102278 args:0xc010dc20 0x0000011b 0xc010dd44 0x0000001b 
    kern/debug/panic.c:19: __panic+31
ebp:0xc03adfa8 eip:0xc0103eb0 args:0xc03adfb4 0x00000049 0x00000000 0xaffffe98 
    kern/trap/trap.c:283: trap+71
ebp:0xaffffe98 eip:0xc0103f34 args:0x0000001e 0x00000049 0xaffffec8 0x008000d6 
    kern/trap/trapentry.S:24: <unknown>+0
ebp:0xaffffea8 eip:0x00800290 args:0x00000049 0x00000000 0x00000000 0x00000000 
    user/libs/syscall.c:65: sys_putc+24
ebp:0xaffffec8 eip:0x008000d6 args:0x00000049 0xafffff3c 0x00000000 0x00000000 
    user/libs/stdio.c:11: cputch+16
ebp:0xafffff18 eip:0x0080056b args:0x008000c5 0xafffff3c 0x008013e1 0xafffff84 
    libs/printfmt.c:131: vprintfmt+33
ebp:0xafffff48 eip:0x00800113 args:0x008013e0 0xafffff84 0x00000000 0x00000000 
    user/libs/stdio.c:27: vcprintf+45
ebp:0xafffff78 eip:0x00800136 args:0x008013e0 0x00000000 0x00000000 0x00000000 
    user/libs/stdio.c:42: cprintf+29
ebp:0xafffffa8 eip:0x00800fa6 args:0x00000000 0x00000000 0x00000000 0x00000000 
    user/exit.c:9: main+20
ebp:0xafffffd8 eip:0x0080034d args:0x00000000 0x00000000 0x00000000 0x00000000 
    user/libs/umain.c:7: umain+10
kernel panic at kern/trap/trap.c:283:
    Panic: I am again overwhelmed.
```

#### 14.2(3):尝试在panic函数中获取和输出页表有效逻辑地址空间范围和在内存中的逻辑地址空间范围，然后在你希望的地方人为触发panic函数，并输出上述信息。

再在panic的实现__panic函数中增加```print_pgdir();```即可打印出页表信息。如下：
```
-------------------- BEGIN --------------------
PDE(001) 00000000-00400000 00400000 urw
  |-- PTE(00004) 00200000-00204000 00004000 urw
PDE(001) 00800000-00c00000 00400000 urw
  |-- PTE(00002) 00800000-00802000 00002000 ur-
  |-- PTE(00001) 00802000-00803000 00001000 urw
PDE(001) afc00000-b0000000 00400000 urw
  |-- PTE(00004) afffc000-b0000000 00004000 urw
PDE(0e0) c0000000-f8000000 38000000 urw
  |-- PTE(38000) c0000000-f8000000 38000000 -rw
PDE(001) fac00000-fb000000 00400000 -rw
  |-- PTE(00001) fac00000-fac01000 00001000 urw
  |-- PTE(00001) fac02000-fac03000 00001000 urw
  |-- PTE(00001) faebf000-faec0000 00001000 urw
  |-- PTE(000e0) faf00000-fafe0000 000e0000 urw
  |-- PTE(00001) fafeb000-fafec000 00001000 -rw
--------------------- END ---------------------
kernel panic at kern/trap/trap.c:283:
    Panic: I am again overwhelmed.
```
