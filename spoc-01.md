##SPOC解答

###阅读部分
####1.在v9-cpu中如何实现时钟中断的

1. 提供了TIME指令，用于在内核态下设置超时时间，存于timeout变量／寄存器，若设为0则表示不计时。
1. 内部有个timer变量／寄存器，随着指令的执行不断增加一个delta时间，直到达到设定的timeout值，便回零重新计时。
2. 如果超时时中断使能打开着，则发出FTIMER中断，即设置trap的值为FTIMER，并暂时把中断使能关闭，然后跳转到中断处理程序。

####2.指出v9-cpu指令、关键变量描述有误或不全的情况

1. 补充对如下变量的描述：

- ssp: 系统栈的stack pointer
- usp: 用户栈的stack pointer
- cycle: 周期计数器
- xcycle: 时间太紧了，没看仔细，猜测是extra cycle，用来辅助模拟一些需要消耗多个周期的操作（？）
- timer: 计时器，用于和timeout比对以判断是否超时
- timeout: 存储通过TIME指令设置的超时时间
- detla: 计时器每次增加的时间间隔


####3.在v9-cpu中的跳转相关操作是如何实现的

1. B类指令需要先判断是否符合跳转条件，然后转2；J类直接转2。
2. 计算目标PC，必要时需要经过虚实地址转换。
3. 若得到的PC非法，需要跳转到fixpc代码块处，发出FIPAGE异常，即在取指出现页错误，并进入异常处理；
4. 若所得PC合法，跳转成功，体现在模拟器跳到了next代码块处。

####4.在v9-cpu中如何设计相应指令，可有效实现函数调用与返回

1. v9-cpu提供了JSR和JSRA指令，两者在跳转前都会将当前PC保存到栈中。
2. 通过JSR或JSRA跳转到目标函数后，需要跳回去只需要把之前保存的PC取出然后跳回去即可。
3. 至于传入参数、返回值，采用类似x86-32的方式，借助SP通过栈、或直接通过寄存器进行。

####5.程序emhello、os0等被加载到内存的哪个位置，其堆栈是如何设置的

1. 执行模拟器时制定的file中的内容，前一部分是硬盘内容，后一部分才是要读入内存的，从下面的代码片段可知，被读到了mem的开始处，也就是内存的0地址处。
2. 开始执行cpu这个函数时，传入的第二个参数是sp的初始值，可见栈是从高地址开始往下长的。
3. 至于堆，通过察看libc.h里的sbrk（实际是xsbrk）函数可知道sbrk是从低地址开始往上长的，而它初值是0，
但由于初始化时读入file到内存前有调用过new（new则调用sbrk）分配空间给程序，所以可以看到，
堆在内存中紧接着程序代码区，不断往高地址长。

```c
...
if ((f = open(file, O_RDONLY)) < 0) { ... ｝
...
read(f, &hdr, sizeof(hdr));
...
if (read(f, (void*)mem, st.st_size - sizeof(hdr)) != st.st_size - sizeof(hdr)) { ... }
...
```

```c
cpu(hdr.entry, memsz - FS_SZ);
```

####6.在v9-cpu中如何完成一次内存地址的读写
1. 感觉和下一个问题重复了，所以请读者直接跳到下一问的回答吧。


####7.在v9-cpu中如何实现分页机制

1. 初始化时通过new函数，在代表内存（？）的mem中，紧跟着程序代码区后（堆的开始处），分配了空间给四个页表，名为trk、twk、tru、twu，分别用于内核态（后缀k）、用户态（后缀u）下的读地址（r）、写地址（w）翻译。
2. 我觉得这四个起到的功能是硬件里的TLB，但是令人疑惑的是为什么要把它们的空间分配在代表内存的mem里，我觉得虽然这样实现模拟器也没什么关系，但总觉得看着不是很愉悦。
2. tr、tw根据当前是什么态指向trk、twk或tru、twu。
3. 当需要访存（如取指、取数据、写数据）时，先查看TLB，即tr（读）或tw（写），或有对应项则使用之；若没有这调用相应的程序（rlook函数或wlook）从页表中读取相应的项缓存到TLB中。
4. 这个过程，需要注意一些标志的处理，比如dirty等。
5. 从rlook、wlook可以看出，页表是一个二级页表。需要先用地址的高10位读出page directory entry，然后用接下来的10位从取出的page directory entry中再读出page table entry，从这个entry才推算出物理地址。
6. 当然，如果分页机制没有打开，则在rlook、wlook那一步就直接把地址作为物理地址返回了（有个对paging是否true的判断）。


```c
trk = (uint *) new(TB_SZ * sizeof(uint)); 
twk = (uint *) new(TB_SZ * sizeof(uint)); 
tru = (uint *) new(TB_SZ * sizeof(uint)); 
twu = (uint *) new(TB_SZ * sizeof(uint));
```

```c
uint rlook(uint v)
{
  ...
  pde = *(ppde = (uint *)(pdir + (v>>22<<2))); // page directory entry
  if (pde & PTE_P) {
     ...
     pte = *(ppte = (uint *)(mem + (pde & -4096) + ((v >> 10) & 0xffc))); // page table entry
     ...
  }
  ...
}
```


###编程部分
####1.请编写一个小程序，在v9-cpu下，能够接收你输入的字符并输出你输入的字符
```c
#include <u.h>

main()
{
  asm(LI, 0);
  asm(BIN);
  asm(PSHA);
  asm(POPB);
  asm(LI, 1);
  asm(BOUT);
  asm(HALT);
}
```
####2.请编写一个小程序，在v9-cpu下，能够产生各种异常/中断
```c
//
// This snippet will print a 'X' for each exception. 5 in total.
//

#include <u.h>

enum {  // page table entry flags
  PTE_P   = 0x001,       // Present
  PTE_W   = 0x002,       // Writeable
  PTE_U   = 0x004,       // User
  PTE_A   = 0x020,       // Accessed
  PTE_D   = 0x040,       // Dirty
};

int i, x, y;  // Does the compiler support local variables?

char pg_mem[6 * 4096]; // page dir + 4 entries + alignment

int *pg_dir, *pg0, *pg1, *pg2, *pg3;

alltraps(){
  // backup
  asm(PSHA);
  asm(PSHB);
  asm(PSHC);

  // a simple exception handler ...
  // that only knonws how to print a 'X'
  asm(LBI, 'X');
  asm(LI,1);
  asm(BOUT);

  // restore
  asm(POPC);
  asm(POPB);
  asm(POPA);

  asm(RTI);
}

my_load (val) { asm(LL, 8); }

setup_paging()
{

  pg_dir = (int *)((((int)&pg_mem) + 4095) & -4096);
  pg0 = pg_dir + 1024;
  pg1 = pg0 + 1024;
  pg2 = pg1 + 1024;
  pg3 = pg2 + 1024;

  pg_dir[0] = (int)pg0 | PTE_P | PTE_W | PTE_U; 
  pg_dir[1] = (int)pg1 | PTE_P | PTE_W | PTE_U;
  pg_dir[2] = (int)pg2 | PTE_P | PTE_W | PTE_U;
  pg_dir[3] = (int)pg3 | PTE_P | PTE_W | PTE_U;
  for (i=4;i<1024;i++) pg_dir[i] = 0;

  for (i=0;i<4096;i++) pg0[i] = (i<<12) | PTE_P | PTE_W | PTE_U;

  my_load(pg_dir); 
  asm(PDIR); 

  asm(LI,1); 
  asm(SPAG); 
}

main(){
  my_load(alltraps);
  asm(IVEC);

  // enable interrupt
  asm(STI);  

  // exception #1: invalid instruction
  asm(-1);  

  // exception #2: bad physicall address
  x = *(int *)0x10000000; 

  // exception #3: divided by 0
  x = 10; y = 0; x /= y;

  // setup paging before we can invoke the next two exceptions
  asm(LI, 4*1024*1024); // reg #a = 4M
  asm(SSP); // set the kernel stack pointer to be the value in reg #a
  setup_paging();

  pg0[50] = 0;
  my_load(pg_dir);
  asm(PDIR);

  // exception #4: page fault (read)
  x = *(int *)(50<<12); 

  // exception #5: page fault (write)
  *(int *)(50<<12) = 5; 

  asm(HALT);
}
```

####3.请编写一个小程序，在v9-cpu下，能够统计并显示内存大小
```c
//
// The printing method is borrowed from os2.c.
//
#include <u.h>

enum { // page table entry flags
  PTE_P   = 0x001,       // Present
  PTE_W   = 0x002,       // Writeable
  PTE_U   = 0x004,       // User
//PTE_PWT = 0x008,       // Write-Through
//PTE_PCD = 0x010,       // Cache-Disable
  PTE_A   = 0x020,       // Accessed
  PTE_D   = 0x040,       // Dirty
//PTE_PS  = 0x080,       // Page Size
//PTE_MBZ = 0x180,       // Bits must be zero
};

enum { // processor fault codes
  FMEM,   // bad physical address
  FTIMER, // timer interrupt
  FKEYBD, // keyboard interrupt
  FPRIV,  // privileged instruction
  FINST,  // illegal instruction
  FSYS,   // software trap
  FARITH, // arithmetic trap
  FIPAGE, // page fault on opcode fetch
  FWPAGE, // page fault on write
  FRPAGE, // page fault on read
  USER=16 // user mode exception 
};

char pg_mem[6 * 4096]; // page dir + 4 entries + alignment

int *pg_dir, *pg0, *pg1, *pg2, *pg3;

int current;

int in(port)    { asm(LL,8); asm(BIN); }
out(port, val)  { asm(LL,8); asm(LBL,16); asm(BOUT); }
ivec(void *isr) { asm(LL,8); asm(IVEC); }
lvadr()         { asm(LVAD); }
stmr(int val)   { asm(LL,8); asm(TIME); }
pdir(value)     { asm(LL,8); asm(PDIR); }
spage(value)    { asm(LL,8); asm(SPAG); }
halt(value)     { asm(LL,8); asm(HALT); }

void *memcpy() { asm(LL,8); asm(LBL, 16); asm(LCL,24); asm(MCPY); asm(LL,8); }
void *memset() { asm(LL,8); asm(LBLB,16); asm(LCL,24); asm(MSET); asm(LL,8); }
void *memchr() { asm(LL,8); asm(LBLB,16); asm(LCL,24); asm(MCHR); }

write(fd, char *p, n) { while (n--) out(fd, *p++); }

int strlen(char *s) { return memchr(s, 0, -1) - (void *)s; }

enum { BUFSIZ = 32 };
int vsprintf(char *s, char *f, va_list v)
{
  char *e = s, *p, c, fill, b[BUFSIZ];
  int i, left, fmax, fmin, sign;

  while (c = *f++) {
    if (c != '%') { *e++ = c; continue; }
    if (*f == '%') { *e++ = *f++; continue; }
    if (left = (*f == '-')) f++;
    fill = (*f == '0') ? *f++ : ' ';
    fmin = sign = 0; fmax = BUFSIZ;
    if (*f == '*') { fmin = va_arg(v,int); f++; } else while ('0' <= *f && *f <= '9') fmin = fmin * 10 + *f++ - '0';
    if (*f == '.') { if (*++f == '*') { fmax = va_arg(v,int); f++; } else for (fmax = 0; '0' <= *f && *f <= '9'; fmax = fmax * 10 + *f++ - '0'); }
    if (*f == 'l') f++;
    switch (c = *f++) {
    case 0: *e++ = '%'; *e = 0; return e - s;
    case 'c': fill = ' '; i = (*(p = b) = va_arg(v,int)) ? 1 : 0; break;
    case 's': fill = ' '; if (!(p = va_arg(v,char *))) p = "(null)"; if ((i = strlen(p)) > fmax) i = fmax; break;
    case 'u': i = va_arg(v,int); goto c1;
    case 'd': if ((i = va_arg(v,int)) < 0) { sign = 1; i = -i; } c1: p = b + BUFSIZ-1; do { *--p = ((uint)i % 10) + '0'; } while (i = (uint)i / 10); i = (b + BUFSIZ-1) - p; break;
    case 'o': i = va_arg(v,int); p = b + BUFSIZ-1; do { *--p = (i & 7) + '0'; } while (i = (uint)i >> 3); i = (b + BUFSIZ-1) - p; break;
    case 'p': fill = '0'; fmin = 8; c = 'x';
    case 'x': case 'X': c -= 33; i = va_arg(v,int); p = b + BUFSIZ-1; do { *--p = (i & 15) + ((i & 15) > 9 ? c : '0'); } while (i = (uint)i >> 4); i = (b + BUFSIZ-1) - p; break;
    default: *e++ = c; continue;
    }
    fmin -= i + sign;
    if (sign && fill == '0') *e++ = '-';
    if (!left && fmin > 0) { memset(e, fill, fmin); e += fmin; }
    if (sign && fill == ' ') *e++ = '-';
    memcpy(e, p, i); e += i;
    if (left && fmin > 0) { memset(e, fill, fmin); e += fmin; }
  }
  *e = 0;
  return e - s;
}

int printf(char *f) { static char buf[4096]; return write(1, buf, vsprintf(buf, f, &f)); } // XXX remove static from buf

trap(int c, int b, int a, int fc, int pc)
{
  printf("TRAP: ");
  switch (fc) {
  case FINST:  printf("FINST"); break;
  case FRPAGE: printf("FRPAGE [0x%08x]",lvadr()); break;
  case FWPAGE: printf("FWPAGE [0x%08x]",lvadr()); break;
  case FIPAGE: printf("FIPAGE [0x%08x]",lvadr()); break;
  case FSYS:   printf("FSYS"); break;
  case FARITH: printf("FARITH"); break;
  case FMEM:   printf("FMEM [0x%08x]",lvadr()); break;
  case FTIMER: printf("FTIMER"); current = 1; stmr(0); break;
  case FKEYBD: printf("FKEYBD [%c]", in(0)); break;
  default:     printf("other [%d]",fc); break;
  }
}

alltraps()
{
  asm(PSHA);
  asm(PSHB);
  asm(PSHC);
  trap();
  asm(POPC);
  asm(POPB);
  asm(POPA);
  asm(RTI);
}

setup_paging()
{
  int i;
  
  pg_dir = (int *)((((int)&pg_mem) + 4095) & -4096);
  pg0 = pg_dir + 1024;
  pg1 = pg0 + 1024;
  pg2 = pg1 + 1024;
  pg3 = pg2 + 1024;
  
  pg_dir[0] = (int)pg0 | PTE_P | PTE_W | PTE_U;  // identity map 16M
  pg_dir[1] = (int)pg1 | PTE_P | PTE_W | PTE_U;
  pg_dir[2] = (int)pg2 | PTE_P | PTE_W | PTE_U;
  pg_dir[3] = (int)pg3 | PTE_P | PTE_W | PTE_U;
  for (i=4;i<1024;i++) pg_dir[i] = 0;
  
  for (i=0;i<4096;i++) pg0[i] = (i<<12) | PTE_P | PTE_W | PTE_U;  // trick to write all 4 contiguous pages
  
  pdir(pg_dir);
  spage(1);
}

int memsize() { asm(MSIZ); }

main()
{
  printf("Memory Size = %d.", memsize());
  asm(HALT);
}
```
