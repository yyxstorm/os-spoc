# 3.4 linux系统调用分析
 1. 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(w2l1)
  - 答：（1）objdump可以获得lab1-ex0.exe的汇编码，nm可以列出目标文件中的符号，file可以察看文件类型（比如这里显示lab1-ex0.exe是32位的ELF执行文件），strace可用来跟踪系统调用的过程。（2）系统调用是操作系统向应用程序提供服务的接口。应用程序通过软中断（陷阱）进行系统调用。x86平台Linux下，系统调用的编写方法为：在```eax```寄存器中指明所需调用的服务，（如果需要则）在```ebx```、```ecx```等存入相应参数，再使用```int x80```汇编指令即可进行系统调用，返回值存于```eax```。具体可参考[这里](http://syscalls.kernelgrok.com/)。

 ```
  + 采分点：说明了objdump，nm，file的大致用途，说明了系统调用的具体含义
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 
 ```
 
 1. 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(w2l1)
  - 答：（1）strace可用来跟踪系统调用的过程，strace -c表示记录时间、调用次数、错误等。以下是```strace -c ./lab1-ex1.exe```的输出：
    
    ```
    hello world
    % time     seconds  usecs/call     calls    errors syscall
    ------ ----------- ----------- --------- --------- ----------------
    0.00    0.000000           0         1           read
    0.00    0.000000           0         1           write
    0.00    0.000000           0         2           open
    0.00    0.000000           0         2           close
    0.00    0.000000           0         3           fstat
    0.00    0.000000           0         9           mmap
    0.00    0.000000           0         4           mprotect
    0.00    0.000000           0         1           munmap
    0.00    0.000000           0         1           brk
    0.00    0.000000           0         3         3 access
    0.00    0.000000           0         1           execve
    0.00    0.000000           0         1           arch_prctl
    ------ ----------- ----------- --------- --------- ----------------
    100.00    0.000000                    29         3 total
    ```
    
    从中可以看到，这个简单的程序除了调用write服务外，还调用了很多其他的服务。现在我们不加```-c```，来看看具体的执行顺序：
    ```
    execve("./lab1-ex1.exe", ["./lab1-ex1.exe"], [/* 69 vars */]) = 0
    brk(0)                                  = 0x1d01000
    access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
    mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fee9cc56000
    access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
    open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    fstat(3, {st_mode=S_IFREG|0644, st_size=109711, ...}) = 0
    mmap(NULL, 109711, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fee9cc3b000
    close(3)                                = 0
    access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
    open("/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
    read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0`\v\2\0\0\0\0\0"..., 832) = 832
    fstat(3, {st_mode=S_IFREG|0755, st_size=1869392, ...}) = 0
    mmap(NULL, 3972864, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fee9c66b000
    mprotect(0x7fee9c82b000, 2097152, PROT_NONE) = 0
    mmap(0x7fee9ca2b000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1c0000) = 0x7fee9ca2b000
    mmap(0x7fee9ca31000, 16128, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fee9ca31000
    close(3)                                = 0
    mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fee9cc3a000
    mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fee9cc39000
    mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fee9cc38000
    arch_prctl(ARCH_SET_FS, 0x7fee9cc39700) = 0
    mprotect(0x7fee9ca2b000, 16384, PROT_READ) = 0
    mprotect(0x600000, 4096, PROT_READ)     = 0
    mprotect(0x7fee9cc58000, 4096, PROT_READ) = 0
    munmap(0x7fee9cc3b000, 109711)          = 0
    fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 6), ...}) = 0
    mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fee9cc55000
    write(1, "hello world\n", 12hello world
    )           = 12
    exit_group(12)                          = ?
    +++ exited with 12 +++
    ```
    
    首先是bash通过execve服务让OS执行相应的程序。之后调用brk在堆上分配空间。三个access用来链入相应的动态链接库，期间穿插open、fstat、read、close进行相关的文件操作（打开、统计、读、关闭文件）。程序执行是通过mmap服务进行虚实内存转换。arch_prctl服务用来进行架构相关的设置。mprotect服务则是用来设置内存保护属性。最后有个exit_group服务来退出程序。
    
 ```
  + 采分点：说明了strace的大致用途，说明了系统调用的具体执行过程（包括应用，CPU硬件，操作系统的执行过程）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
# 3.5 ucore系统调用分析
 1. ucore的系统调用中参数传递代码分析。
  - 答：在用户态中通过函数调用将参数传递至用户态的```syscall```函数（```syscall```参数长度可变）中，用户态的```syscall```函数通过```asm volatile```机制将参数传入内核态；在内核态中系统调用通过```trapframe(tf)```进行参数传递，最后在内核态的```syscall```函数中执行：
 
    ```
    int num = tf->tf_regs.reg_eax;
    if (num >= 0 && num < NUM_SYSCALLS) {
        if (syscalls[num] != NULL) {
            arg[0] = tf->tf_regs.reg_edx;
            arg[1] = tf->tf_regs.reg_ecx;
            arg[2] = tf->tf_regs.reg_ebx;
            arg[3] = tf->tf_regs.reg_edi;
            arg[4] = tf->tf_regs.reg_esi;
            tf->tf_regs.reg_eax = syscalls[num](arg);
            return ;
        }
    }
    ```

   其中系统调用类型码（用于判断使用哪个系统调用）通过```tf->tf_regs.reg_eax```进行传递，其他参数通过```tf->tf_regs```进行传递，系统调用的结果储存在```tf->tf_regs.reg_eax```中。 最后在用户态的```syscall(int,...)```函数中通过```eax```寄存器返回。

 2. 以getpid为例，分析ucore的系统调用中返回结果的传递代码。
   - 答：在内核态中，系统调用的返回值通过```trapframe```中的```tf_regs.reg_eax```进行传递，如下述代码所示：
 
     ```tf->tf_regs.reg_eax = syscalls[num](arg);```

     最后在用户态的syscall函数中通过```eax```寄存器返回。

 3. 以ucore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
   - 答：以```getpid()```函数为例：
     - a. ```getpid()```函数是```ulib.c```中经过包装的系统调用；
     - b. ```getpid()```调用用户态```syscall.c```中的```sys_getpid()```函数；
     - c. ```sys_getpid()```调用用户态的```syscall(int,...)```函数；
     - d. 用户态的```syscall(int,...)```函数通过```asm volatile```机制将参数传入内核态；
     - e. 在内核态中参数（包括系统调用类型码）通过```trapframe```进行传递，最后传递到内核态的```syscall（）```函数中；
     - f. 内核态的```syscall()```函数根据系统调用类型调用```sys_getpid(int)```函数，返回值写入```eax```寄存器中；
     - g. 返回到用户态时，返回值从```eax```寄存器中读出。

 4. 以ucore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。
   - 答：在kernel中的```syscall```中根据```tf->tf_regs.reg_eax```（系统调用类型号）判断系统调用的类型，并使用```cprintf()```函数输出即可。
