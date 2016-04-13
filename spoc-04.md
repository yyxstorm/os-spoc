### 4.1 启动顺序

描述符特权级DPL、当前特权级CPL和请求特权级RPL的含义是什么？在哪些寄存器中这些字段？对应的访问条件是什么？ (challenge)写出一些简单的小程序（c or asm）来体现这些特权级的区别和联系。
 ```
  + 建议参见链接“ http://blog.csdn.net/better0332/article/details/3416749 ”对特权级的表述，并查阅指令手册。
 ```
 答：
  - CPL是当前进程的权限级别(Current Privilege Level)，是当前正在执行的代码所在的段的特权级，之所以说是“正在执行的代码的”，是因为它存在于```cs```(code segment)寄存器的低两位。
  - RPL说明的是进程对段访问的请求权限(Request Privilege Level)，是对于段选择子而言的，每个段选择子有自己的RPL，代表进程对段访问的请求权限。而且RPL对每个段来说不是固定的，两次访问同一段时的RPL可以不同。RPL可以会用来削弱CPL的作用，见下面对DPL的描述。
  - 从上面的表述来看，对于**代码段**来说，似乎CPL和RPL是同一个东西（如果有误希望能指出）；当然，对于其他类型的段就不是了。
  - DPL存储在段描述符中，规定访问该段的权限级别(Descriptor Privilege Level)，每个段的DPL固定。当进程希望访问一个段时，要求的访问条件是DPL >= max {CPL, RPL}。

### 4.2 C函数调用的实现

比较不同特权级的中断切换时的堆栈变化差别；(challenge)写出一些简单的小程序（c or asm）来显示出不同特权级的的中断切换的堆栈变化情况。
答：
  - 中断切换时，如果发生了特权级的变化，比如从用户态切换到内核态，则切换到内核堆栈后，会压入原来用户堆栈的栈顶指针，即SS(Stack Segment)和ESP(Stack Pointer)的值，然后再压入EFLAGS（中断类型）、CS（Code Segment）、EIP（Instruction Pointer），还可能会压入Error Code。
  - 中断返回时，使用iret指令，iret会视是否发生了特权级变化而决定是否需要弹出SS、ESP，以恢复到原来的堆栈（比如从内核堆切换回用户堆）。
  - 而如果中断切换时无特权级变化（比如处于内核态时发生中断），则因为此时堆栈不切换，所以不需保存SS和ESP。

challenge:

原理：  
  - 下面讲解以下如何修改ucore的内核代码以便在堆栈变化时打印出相关的信息。
  - 首先，我们可以分别使用```int T_SWITCH_TOU```和```int T_SWITCH_TOK```来切换到用户态、内核态，具体的使用方法见lab1_result的```kern/init/init.c```中的```lab1_switch_to_user```和```lab1_switch_to_kernel```这两个函数中的内联汇编。
  - 其次，我们可以通过内联汇编获得寄存器的值，然后将它们打印出来，比如可以参见```read_ebp```函数这个例子（见下））。
  - 有了这些准备工作，我们就可以在每次切换状态后及时打印出相应栈帧，怎么打印，见```kern/debug/kdebug.c```的```print_stackframe```（已贴在下面）。
  ```
  void
  print_stackframe(void) {
      uint32_t ebp = read_ebp(), eip = read_eip();
  
      int i, j;
      for (i = 0; ebp != 0 && i < STACKFRAME_DEPTH; i ++) {
          cprintf("ebp:0x%08x eip:0x%08x args:", ebp, eip);
          uint32_t *args = (uint32_t *)ebp + 2;
          for (j = 0; j < 4; j ++) {
              cprintf("0x%08x ", args[j]);
          }
          cprintf("\n");
          print_debuginfo(eip - 1);
          eip = ((uint32_t *)ebp)[1];
          ebp = ((uint32_t *)ebp)[0];
      }
  }
  ```
  - 其中的```read_ebp```代码在```lab1_result/libs/x86.h```里，如下：
  
  ```
  static inline uint32_t
  read_ebp(void) {
      uint32_t ebp;
      asm volatile ("movl %%ebp, %0" : "=r" (ebp));
      return ebp;
  }
  ```

代码实现：
在lab8答案中修改/kern/trap/trap.c:

```
#include <defs.h>
#include <mmu.h>
#include <memlayout.h>
#include <clock.h>
#include <trap.h>
#include <x86.h>
#include <stdio.h>
#include <assert.h>
#include <console.h>
#include <vmm.h>
#include <swap.h>
#include <kdebug.h>
#include <unistd.h>
#include <syscall.h>
#include <error.h>
#include <sched.h>
#include <sync.h>
#include <proc.h>

#define TICK_NUM 100

static int inK = 0;
static int inU = 0;

static void print_ticks() {
    cprintf("%d ticks\n",TICK_NUM);
#ifdef DEBUG_GRADE
    cprintf("End of Test.\n");
    panic("EOT: kernel seems ok.");
#endif
}

/* *
 * Interrupt descriptor table:
 *
 * Must be built at run time because shifted function addresses can't
 * be represented in relocation records.
 * */
static struct gatedesc idt[256] = {{0}};

static struct pseudodesc idt_pd = {
    sizeof(idt) - 1, (uintptr_t)idt
};

/* idt_init - initialize IDT to each of the entry points in kern/trap/vectors.S */
void
idt_init(void) {
     /* LAB1 YOUR CODE : STEP 2 */
     /* (1) Where are the entry addrs of each Interrupt Service Routine (ISR)?
      *     All ISR's entry addrs are stored in __vectors. where is uintptr_t __vectors[] ?
      *     __vectors[] is in kern/trap/vector.S which is produced by tools/vector.c
      *     (try "make" command in lab1, then you will find vector.S in kern/trap DIR)
      *     You can use  "extern uintptr_t __vectors[];" to define this extern variable which will be used later.
      * (2) Now you should setup the entries of ISR in Interrupt Description Table (IDT).
      *     Can you see idt[256] in this file? Yes, it's IDT! you can use SETGATE macro to setup each item of IDT
      * (3) After setup the contents of IDT, you will let CPU know where is the IDT by using 'lidt' instruction.
      *     You don't know the meaning of this instruction? just google it! and check the libs/x86.h to know more.
      *     Notice: the argument of lidt is idt_pd. try to find it!
      */
     /* LAB5 YOUR CODE */ 
     //you should update your lab1 code (just add ONE or TWO lines of code), let user app to use syscall to get the service of ucore
     //so you should setup the syscall interrupt gate in here
    extern uintptr_t __vectors[];
    int i;
    for (i = 0; i < sizeof(idt) / sizeof(struct gatedesc); i ++) {
        SETGATE(idt[i], 0, GD_KTEXT, __vectors[i], DPL_KERNEL);
    }
    SETGATE(idt[T_SYSCALL], 1, GD_KTEXT, __vectors[T_SYSCALL], DPL_USER);
    lidt(&idt_pd);
}

static const char *
trapname(int trapno) {
    static const char * const excnames[] = {
        "Divide error",
        "Debug",
        "Non-Maskable Interrupt",
        "Breakpoint",
        "Overflow",
        "BOUND Range Exceeded",
        "Invalid Opcode",
        "Device Not Available",
        "Double Fault",
        "Coprocessor Segment Overrun",
        "Invalid TSS",
        "Segment Not Present",
        "Stack Fault",
        "General Protection",
        "Page Fault",
        "(unknown trap)",
        "x87 FPU Floating-Point Error",
        "Alignment Check",
        "Machine-Check",
        "SIMD Floating-Point Exception"
    };

    if (trapno < sizeof(excnames)/sizeof(const char * const)) {
        return excnames[trapno];
    }
    if (trapno >= IRQ_OFFSET && trapno < IRQ_OFFSET + 16) {
        return "Hardware Interrupt";
    }
    return "(unknown trap)";
}

/* trap_in_kernel - test if trap happened in kernel */
bool
trap_in_kernel(struct trapframe *tf) {
    return (tf->tf_cs == (uint16_t)KERNEL_CS);
}

static const char *IA32flags[] = {
    "CF", NULL, "PF", NULL, "AF", NULL, "ZF", "SF",
    "TF", "IF", "DF", "OF", NULL, NULL, "NT", NULL,
    "RF", "VM", "AC", "VIF", "VIP", "ID", NULL, NULL,
};

void
print_trapframe(struct trapframe *tf) {
    cprintf("trapframe at %p\n", tf);
    print_regs(&tf->tf_regs);
    cprintf("  ds   0x----%04x\n", tf->tf_ds);
    cprintf("  es   0x----%04x\n", tf->tf_es);
    cprintf("  fs   0x----%04x\n", tf->tf_fs);
    cprintf("  gs   0x----%04x\n", tf->tf_gs);
    cprintf("  trap 0x%08x %s\n", tf->tf_trapno, trapname(tf->tf_trapno));
    cprintf("  err  0x%08x\n", tf->tf_err);
    cprintf("  eip  0x%08x\n", tf->tf_eip);
    cprintf("  cs   0x----%04x\n", tf->tf_cs);
    cprintf("  flag 0x%08x ", tf->tf_eflags);

    int i, j;
    for (i = 0, j = 1; i < sizeof(IA32flags) / sizeof(IA32flags[0]); i ++, j <<= 1) {
        if ((tf->tf_eflags & j) && IA32flags[i] != NULL) {
            cprintf("%s,", IA32flags[i]);
        }
    }
    cprintf("IOPL=%d\n", (tf->tf_eflags & FL_IOPL_MASK) >> 12);

    if (!trap_in_kernel(tf)) {
        cprintf("  esp  0x%08x\n", tf->tf_esp);
        cprintf("  ss   0x----%04x\n", tf->tf_ss);
    }
}

void
print_regs(struct pushregs *regs) {
    cprintf("  edi  0x%08x\n", regs->reg_edi);
    cprintf("  esi  0x%08x\n", regs->reg_esi);
    cprintf("  ebp  0x%08x\n", regs->reg_ebp);
    cprintf("  oesp 0x%08x\n", regs->reg_oesp);
    cprintf("  ebx  0x%08x\n", regs->reg_ebx);
    cprintf("  edx  0x%08x\n", regs->reg_edx);
    cprintf("  ecx  0x%08x\n", regs->reg_ecx);
    cprintf("  eax  0x%08x\n", regs->reg_eax);
}

static inline void
print_pgfault(struct trapframe *tf) {
    /* error_code:
     * bit 0 == 0 means no page found, 1 means protection fault
     * bit 1 == 0 means read, 1 means write
     * bit 2 == 0 means kernel, 1 means user
     * */
    cprintf("page fault at 0x%08x: %c/%c [%s].\n", rcr2(),
            (tf->tf_err & 4) ? 'U' : 'K',
            (tf->tf_err & 2) ? 'W' : 'R',
            (tf->tf_err & 1) ? "protection fault" : "no page found");
}

static int
pgfault_handler(struct trapframe *tf) {
    extern struct mm_struct *check_mm_struct;
    if(check_mm_struct !=NULL) { //used for test check_swap
            print_pgfault(tf);
        }
    struct mm_struct *mm;
    if (check_mm_struct != NULL) {
        assert(current == idleproc);
        mm = check_mm_struct;
    }
    else {
        if (current == NULL) {
            print_trapframe(tf);
            print_pgfault(tf);
            panic("unhandled page fault.\n");
        }
        mm = current->mm;
    }
    return do_pgfault(mm, tf->tf_err, rcr2());
}

static volatile int in_swap_tick_event = 0;
extern struct mm_struct *check_mm_struct;

static void
trap_dispatch(struct trapframe *tf) {
    char c;

    int ret=0;

    switch (tf->tf_trapno) {
    case T_PGFLT:  //page fault
        if ((ret = pgfault_handler(tf)) != 0) {
            print_trapframe(tf);
            if (current == NULL) {
                panic("handle pgfault failed. ret=%d\n", ret);
            }
            else {
                if (trap_in_kernel(tf)) {
                    panic("handle pgfault failed in kernel mode. ret=%d\n", ret);
                }
                cprintf("killed by kernel.\n");
                panic("handle user mode pgfault failed. ret=%d\n", ret); 
                do_exit(-E_KILLED);
            }
        }
        break;
    case T_SYSCALL:
        syscall();
        break;
    case IRQ_OFFSET + IRQ_TIMER:
#if 0
    LAB3 : If some page replacement algorithm(such as CLOCK PRA) need tick to change the priority of pages, 
    then you can add code here. 
#endif
        /* LAB1 YOUR CODE : STEP 3 */
        /* handle the timer interrupt */
        /* (1) After a timer interrupt, you should record this event using a global variable (increase it), such as ticks in kern/driver/clock.c
         * (2) Every TICK_NUM cycle, you can print some info using a funciton, such as print_ticks().
         * (3) Too Simple? Yes, I think so!
         */
        /* LAB5 YOUR CODE */
        /* you should upate you lab1 code (just add ONE or TWO lines of code):
         *    Every TICK_NUM cycle, you should set current process's current->need_resched = 1
         */
        /* LAB6 YOUR CODE */
        /* IMPORTANT FUNCTIONS:
	     * run_timer_list
	     *----------------------
	     * you should update your lab5 code (just add ONE or TWO lines of code):
         *    Every tick, you should update the system time, iterate the timers, and trigger the timers which are end to call scheduler.
         *    You can use one funcitons to finish all these things.
         */
        ticks ++;
        assert(current != NULL);
        run_timer_list();
        break;
    case IRQ_OFFSET + IRQ_COM1:
        //c = cons_getc();
        //cprintf("serial [%03d] %c\n", c, c);
        //break;
    case IRQ_OFFSET + IRQ_KBD:
        c = cons_getc();
        //cprintf("kbd [%03d] %c\n", c, c);
        {
          extern void dev_stdin_write(char c);
          dev_stdin_write(c);
        }
        break;
    //LAB1 CHALLENGE 1 : YOUR CODE you should modify below codes.
    case T_SWITCH_TOU:
    case T_SWITCH_TOK:
        panic("T_SWITCH_** ??\n");
        break;
    case IRQ_OFFSET + IRQ_IDE1:
    case IRQ_OFFSET + IRQ_IDE2:
        /* do nothing */
        break;
    default:
        print_trapframe(tf);
        if (current != NULL) {
            cprintf("unhandled trap.\n");
            do_exit(-E_KILLED);
        }
        // in kernel, it must be a mistake
        panic("unexpected trap in kernel.\n");

    }
}

/* *
 * trap - handles or dispatches an exception/interrupt. if and when trap() returns,
 * the code in kern/trap/trapentry.S restores the old CPU state saved in the
 * trapframe and then uses the iret instruction to return from the exception.
 * */
void
trap(struct trapframe *tf) {
    // dispatch based on what type of trap occurred
    // used for previous projects
    if (current == NULL) {
        trap_dispatch(tf);
    }
    else {
        // keep a trapframe chain in stack
        struct trapframe *otf = current->tf;
        current->tf = tf;
    
        bool in_kernel = trap_in_kernel(tf);
    
        if (!inK && in_kernel) {
        	print_trapframe(tf);
            print_stackframe();
            cprintf("kernel\n");
        	inK = 1;
        }
        if (!inU && !in_kernel) {
        	print_trapframe(tf);
            print_stackframe();
            cprintf("user\n");
        	inU = 1;
        }
        trap_dispatch(tf);
    
        current->tf = otf;
        if (!in_kernel) {
            if (current->flags & PF_EXITING) {
                do_exit(-E_KILLED);
            }
            if (current->need_resched) {
                schedule();
            }
        }
    }
}
```

说明：当ucore启动会触发内核态及用户态的一些中断，中断发生时```trap```函数会被调用，将```trap.c```修改后可打印出在内核态（用户态）第一次中断时的堆栈变化情况：

用户态第一次中断的```trapframe```及```stackframe```：
```
trapframe at 0xc0368fb4
  edi  0x008003c0
  esi  0x00801e4e
  ebp  0xafffff30
  oesp 0xc0368fd4
  ebx  0xafffff64
  edx  0x00801e4e
  ecx  0x00000000
  eax  0x00000064
  ds   0x----0023
  es   0x----0023
  fs   0x----0000
  gs   0x----0000
  trap 0x00000080 (unknown trap)
  err  0x00000000
  eip  0x008000fd
  cs   0x----001b
  flag 0x00000202 IF,IOPL=0
  esp  0xafffff04
  ss   0x----0023
ebp:0xc0368f88 eip:0xc0100c34 args:0x00000023 0x00000000 0x00000000 0xc0368fb4 
    kern/debug/kdebug.c:350: print_stackframe+21
ebp:0xc0368fa8 eip:0xc0102aa8 args:0xc0368fb4 0x008003c0 0x00801e4e 0xafffff30 
    kern/trap/trap.c:310: trap+166
ebp:0xafffff30 eip:0xc0103597 args:0x00000064 0x00801e4e 0x00000000 0xafffff64 
    kern/trap/trapentry.S:24: <unknown>+0
ebp:0xafffff44 eip:0x008001f1 args:0x00801e4e 0x00000000 0x00000000 0x00000000 
    user/libs/syscall.c:99: sys_open+15
ebp:0xafffff64 eip:0x008003c0 args:0x00801e4e 0x00000000 0x00000000 0x00000000 
    user/libs/file.c:11: open+19
ebp:0xafffff94 eip:0x008006e5 args:0x00000000 0x00801e4e 0x00000000 0x00000000 
    user/libs/umain.c:11: initfd+19
ebp:0xafffffc4 eip:0x0080074a args:0x00000001 0xaffffff8 0x00000000 0x00000000 
    user/libs/umain.c:25: umain+22
user
```

内核态第一次中断的```trapframe```及```stackframe```：

```
trapframe at 0xc014defc
  edi  0x0000807c
  esi  0x00010094
  ebp  0xc014df48
  oesp 0xc014df1c
  ebx  0x00010094
  edx  0xc0115753
  ecx  0x00000000
  eax  0x00001000
  ds   0x----0010
  es   0x----0010
  fs   0x----0023
  gs   0x----0023
  trap 0x0000000e Page Fault
  err  0x00000002
  eip  0xc0105730
  cs   0x----0008
  flag 0x00000082 SF,IOPL=0
ebp:0xc014ded0 eip:0xc0100c34 args:0xc014defc 0xc0115753 0x00000001 0x00000000 
    kern/debug/kdebug.c:350: print_stackframe+21
ebp:0xc014def0 eip:0xc0102a6c args:0xc014defc 0x0000807c 0x00010094 0xc014df48 
    kern/trap/trap.c:304: trap+106
ebp:0xc014df48 eip:0xc0103597 args:0xc0273b6c 0xc015b96c 0x00000000 0xc015b93c 
    kern/trap/trapentry.S:24: <unknown>+0
ebp:0xc014dfb8 eip:0xc0105c28 args:0x017031bc 0x0000000c 0x00040000 0x00000000 
    kern/mm/swap.c:241: check_swap+905
ebp:0xc014dfd8 eip:0xc01054a7 args:0x00000000 0x00000000 0x00000000 0xc01142e0 
    kern/mm/swap.c:50: swap_init+125
ebp:0xc014dff8 eip:0xc01000a2 args:0x00000000 0x00000000 0x0000ffff 0x40cf9a00 
    kern/init/init.c:45: kern_init+119
kernel
```
