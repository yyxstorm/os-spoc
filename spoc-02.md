##分析和实验funcall.c

 - 修改代码，可正常显示小组两位同学的学号（用字符串） 
 
 ```c
 // ret = write(1, "S1_ID S2_ID",12);
 ret = write(1, "2013011380 2013011384", 21);
 ```
 
 - 生成funcall.c的汇编码，理解其实现并给汇编码写注释

 ```
 root/usr/funcall.c  1: #include <u.h>
root/lib/u.h  1: // u.h
root/lib/u.h  2: 
root/lib/u.h  3: // instruction set
root/lib/u.h  4: enum {
root/lib/u.h  5:   HALT,ENT ,LEV ,JMP ,JMPI,JSR ,JSRA,LEA ,LEAG,CYC ,MCPY,MCMP,MCHR,MSET, // system
root/lib/u.h  6:   LL  ,LLS ,LLH ,LLC ,LLB ,LLD ,LLF ,LG  ,LGS ,LGH ,LGC ,LGB ,LGD ,LGF , // load a
root/lib/u.h  7:   LX  ,LXS ,LXH ,LXC ,LXB ,LXD ,LXF ,LI  ,LHI ,LIF ,
root/lib/u.h  8:   LBL ,LBLS,LBLH,LBLC,LBLB,LBLD,LBLF,LBG ,LBGS,LBGH,LBGC,LBGB,LBGD,LBGF, // load b
root/lib/u.h  9:   LBX ,LBXS,LBXH,LBXC,LBXB,LBXD,LBXF,LBI ,LBHI,LBIF,LBA ,LBAD,
root/lib/u.h  10:   SL  ,SLH ,SLB ,SLD ,SLF ,SG  ,SGH ,SGB ,SGD ,SGF ,                     // store
root/lib/u.h  11:   SX  ,SXH ,SXB ,SXD ,SXF ,
root/lib/u.h  12:   ADDF,SUBF,MULF,DIVF,                                                   // arithmetic
root/lib/u.h  13:   ADD ,ADDI,ADDL,SUB ,SUBI,SUBL,MUL ,MULI,MULL,DIV ,DIVI,DIVL,
root/lib/u.h  14:   DVU ,DVUI,DVUL,MOD ,MODI,MODL,MDU ,MDUI,MDUL,AND ,ANDI,ANDL,
root/lib/u.h  15:   OR  ,ORI ,ORL ,XOR ,XORI,XORL,SHL ,SHLI,SHLL,SHR ,SHRI,SHRL,
root/lib/u.h  16:   SRU ,SRUI,SRUL,EQ  ,EQF ,NE  ,NEF ,LT  ,LTU ,LTF ,GE  ,GEU ,GEF ,      // logical
root/lib/u.h  17:   BZ  ,BZF ,BNZ ,BNZF,BE  ,BEF ,BNE ,BNEF,BLT ,BLTU,BLTF,BGE ,BGEU,BGEF, // conditional
root/lib/u.h  18:   CID ,CUD ,CDI ,CDU ,                                                   // conversion
root/lib/u.h  19:   CLI ,STI ,RTI ,BIN ,BOUT,NOP ,SSP ,PSHA,PSHI,PSHF,PSHB,POPB,POPF,POPA, // misc
root/lib/u.h  20:   IVEC,PDIR,SPAG,TIME,LVAD,TRAP,LUSP,SUSP,LCL ,LCA ,PSHC,POPC,MSIZ,
root/lib/u.h  21:   PSHG,POPG,NET1,NET2,NET3,NET4,NET5,NET6,NET7,NET8,NET9,
root/lib/u.h  22:   POW ,ATN2,FABS,ATAN,LOG ,LOGT,EXP ,FLOR,CEIL,HYPO,SIN ,COS ,TAN ,ASIN, // math
root/lib/u.h  23:   ACOS,SINH,COSH,TANH,SQRT,FMOD,
root/lib/u.h  24:   IDLE
root/lib/u.h  25: };
root/lib/u.h  26: 
root/lib/u.h  27: // system calls
root/lib/u.h  28: enum {
root/lib/u.h  29:   S_fork=1, S_exit,   S_wait,   S_pipe,   S_write,  S_read,   S_close,  S_kill,
root/lib/u.h  30:   S_exec,   S_open,   S_mknod,  S_unlink, S_fstat,  S_link,   S_mkdir,  S_chdir,
root/lib/u.h  31:   S_dup2,   S_getpid, S_sbrk,   S_sleep,  S_uptime, S_lseek,  S_mount,  S_umount,
root/lib/u.h  32:   S_socket, S_bind,   S_listen, S_poll,   S_accept, S_connect, 
root/lib/u.h  33: };
root/lib/u.h  34: 
root/lib/u.h  35: typedef unsigned char uchar;
root/lib/u.h  36: typedef unsigned short ushort;
root/lib/u.h  37: typedef unsigned int uint;
root/lib/u.h  38: 
root/usr/funcall.c  2: int ret;
root/usr/funcall.c  3: out(port, val)
root/usr/funcall.c  4: {
root/usr/funcall.c  5:   asm(LL,8);   // load register a with port
00000000  0000080e  LL    0x8 (D 8)   // a = *(sp+8) -- port
root/usr/funcall.c  6:   asm(LBL,16); // load register b with val
00000004  00001026  LBL   0x10 (D 16) // b = *(sp+16) -- val
root/usr/funcall.c  7:   asm(BOUT);   // output byte to console
00000008  0000009a  BOUT              // a = write(a, &b, 1)
root/usr/funcall.c  8: }
root/usr/funcall.c  9: 
root/usr/funcall.c  10: int write(int f, char *s, int n)
0000000c  00000002  LEV   0x0 (D 0)
root/usr/funcall.c  11: {
root/usr/funcall.c  12:   int i;
root/usr/funcall.c  13:   ret = 1;
00000010  fffff801  ENT   0xfffffff8 (D -8)  // sp = sp-8
00000014  00000123  LI    0x1 (D 1)          // a = 1
00000018  00000045  SG    0x0 (D 0)          // *(pc+op(0x8c)) = a = 1 -- ret
root/usr/funcall.c  14:   i=n;
0000001c  0000200e  LL    0x20 (D 32)        // a = n
00000020  00000440  SL    0x4 (D 4)          // *(sp+4) = a = n
root/usr/funcall.c  15:   while (i--)
00000024  00000003  JMP   <fwd>
root/usr/funcall.c  16:     out(f, *s++);
00000028  0000180e  LL    0x18 (D 24)        // a = *(sp+24) -- s
0000002c  ffffff57  SUBI  0xffffffff (D -1)  // a = a + 1
00000030  00001840  SL    0x18 (D 24)        // *(sp+24) = a
00000034  ffffff1f  LXC   0xffffffff (D -1)
00000038  0000009d  PSHA
0000003c  0000180e  LL    0x18 (D 24)        // a = *(sp+24) -- f
00000040  0000009d  PSHA
00000044  ffffb805  JSR   0xffffffb8 (TO 0x0) // call out
00000048  00001001  ENT   0x10 (D 16)        // sp = sp+16
root/usr/funcall.c  17:   return i;
0000004c  0000040e  LL    0x4 (D 4)          // a = *(sp+4) -- i
00000050  00000157  SUBI  0x1 (D 1)          // a = a - 1
00000054  00000440  SL    0x4 (D 4)          // *(sp+4) = a
00000058  00000154  ADDI  0x1 (D 1)          // a = a + 1
0000005c  00000086  BNZ   <fwd>
00000060  0000040e  LL    0x4 (D 4)          // a = *(sp+4) -- i(return value)
00000064  00000802  LEV   0x8 (D 8)
root/usr/funcall.c  18: }  
root/usr/funcall.c  19: 
root/usr/funcall.c  20: main()
00000068  00000802  LEV   0x8 (D 8)
root/usr/funcall.c  21: {
root/usr/funcall.c  22: 
root/usr/funcall.c  23:   //Change S1/S2 ID to your student ID, and change 12 to new str length
root/usr/funcall.c  24:   // ret = write(1, "S1_ID S2_ID",12);
root/usr/funcall.c  25:   ret = write(1, "2013011380 2013011384", 21);
0000006c  0000159e  PSHI  0x15 (D 21)           // sp = sp - 8, *sp = 21
00000070  00000008  LEAG  0x0 (D 0)             // a = pc+op(0x1c) -- string
00000074  0000009d  PSHA                        // sp = sp - 8, *sp = a
00000078  0000019e  PSHI  0x1 (D 1)             // sp = sp - 8, *sp = 1
0000007c  ffff9005  JSR   0xffffff90 (TO 0x10)  // call write
00000080  00001801  ENT   0x18 (D 24)
00000084  00000045  SG    0x0 (D 0)             // save ret
root/usr/funcall.c  26:   asm(HALT);
00000088  00000000  HALT
root/usr/funcall.c  27: }
root/usr/funcall.c  28: 
0000008c  00000002  LEV   0x0 (D 0)
 ```
 
 - 尝试用xem的简单调试功能单步调试代码
	- 解答：
 		- 使用```xem```的```-g```选项可以启动调试功能，调试功能类似```gdb```
		- 进入调试功能后，```h```命令可显示帮助信息
		- ```q```命令可退出程序
		- ```c```命令是continue的意思，就是让程序自行继续运行
		- ```s```命令是single step的意思，就是单步调试
		- ```i```命令可显示寄存器中存储的值
		- ```x```命令可显示内存中存储的值，需附加一个十六进制数作为参数，以指定内存地址
 		
 - 回答如下问题：
   - funcall中的堆栈有多大？是内核态堆栈还是用户态堆栈
   	 - 解答：内存有128M，FS_SZ占了4M，funcall程序和数据占了B8，堆栈有124M-184bytes。
   - funcall中的全局变量ret放在内存中何处？如何对它寻址？
   	 - 解答：从write函数返回后，main里的```00000084  00000045  SG    0x0 (D 0)```这一句可知，ret放在内存中0x28+0x04+0x8c处。读写可通过LG、SG指令（分别代表LOAD GLOBAL和STORE GLOBAL吗）进行。
   - funcall中的字符串放在内存中何处？如何对它寻址？
	 - 解答：从```00000070  00000008  LEAG  0x0 (D 0)```一句可以直到字符串存在于地址0x80+0x04+0x1c=0xA0处。
   - 局部变量i在内存中的何处？如何对它寻址？
   	 - 解答：从write对应的汇编代码看，发现i存在栈帧中，之后通过```LL    0x4 (D 4)```和```SL    0x4 (D 4)```来读写i变量（LL是LOAD LOCAL之意，SL是STORE LOCAL之意）。
   - 当前系统是处于中断使能状态吗？
	 - 解答：否。 
   - funcall中的函数参数是如何传递的？函数返回值是如何传递的？
     	 - 解答：调用write函数时，把1, "2013011380 2013011384", 21这三个实际参数从右向左依次压入栈，分别对应```PSHI  0x15 (D 21)```, 
```LEAG  0x0 (D 0); PSHA```, ```PSHI  0x1 (D 1)```这三句，然后使用```JSR   0xffffff90 (TO 0x10)```跳转到write函数的入口（JSR指令会顺带记录下当前PC以作为返回地址）。函数返回值通过寄存器a返回。
   - 分析并说明funcall执行文件的格式和内容
   	 - 解答：先是hdr数据（外存数据，含magic数、bss、entry、flags），然后是text（程序指令），最后是data（程序数据）。
　

##分析和实验os0.c

 - 生成os0.c的汇编码，理解其实现并给汇编码写注释
 
```
root/usr/os/os0.c  1: // os0.c -- simple timer isr test
root/usr/os/os0.c  2: 
root/usr/os/os0.c  3: #include <u.h>
root/lib/u.h  1: // u.h
root/lib/u.h  2: 
root/lib/u.h  3: // instruction set
root/lib/u.h  4: enum {
root/lib/u.h  5:   HALT,ENT ,LEV ,JMP ,JMPI,JSR ,JSRA,LEA ,LEAG,CYC ,MCPY,MCMP,MCHR,MSET, // system
root/lib/u.h  6:   LL  ,LLS ,LLH ,LLC ,LLB ,LLD ,LLF ,LG  ,LGS ,LGH ,LGC ,LGB ,LGD ,LGF , // load a
root/lib/u.h  7:   LX  ,LXS ,LXH ,LXC ,LXB ,LXD ,LXF ,LI  ,LHI ,LIF ,
root/lib/u.h  8:   LBL ,LBLS,LBLH,LBLC,LBLB,LBLD,LBLF,LBG ,LBGS,LBGH,LBGC,LBGB,LBGD,LBGF, // load b
root/lib/u.h  9:   LBX ,LBXS,LBXH,LBXC,LBXB,LBXD,LBXF,LBI ,LBHI,LBIF,LBA ,LBAD,
root/lib/u.h  10:   SL  ,SLH ,SLB ,SLD ,SLF ,SG  ,SGH ,SGB ,SGD ,SGF ,                     // store
root/lib/u.h  11:   SX  ,SXH ,SXB ,SXD ,SXF ,
root/lib/u.h  12:   ADDF,SUBF,MULF,DIVF,                                                   // arithmetic
root/lib/u.h  13:   ADD ,ADDI,ADDL,SUB ,SUBI,SUBL,MUL ,MULI,MULL,DIV ,DIVI,DIVL,
root/lib/u.h  14:   DVU ,DVUI,DVUL,MOD ,MODI,MODL,MDU ,MDUI,MDUL,AND ,ANDI,ANDL,
root/lib/u.h  15:   OR  ,ORI ,ORL ,XOR ,XORI,XORL,SHL ,SHLI,SHLL,SHR ,SHRI,SHRL,
root/lib/u.h  16:   SRU ,SRUI,SRUL,EQ  ,EQF ,NE  ,NEF ,LT  ,LTU ,LTF ,GE  ,GEU ,GEF ,      // logical
root/lib/u.h  17:   BZ  ,BZF ,BNZ ,BNZF,BE  ,BEF ,BNE ,BNEF,BLT ,BLTU,BLTF,BGE ,BGEU,BGEF, // conditional
root/lib/u.h  18:   CID ,CUD ,CDI ,CDU ,                                                   // conversion
root/lib/u.h  19:   CLI ,STI ,RTI ,BIN ,BOUT,NOP ,SSP ,PSHA,PSHI,PSHF,PSHB,POPB,POPF,POPA, // misc
root/lib/u.h  20:   IVEC,PDIR,SPAG,TIME,LVAD,TRAP,LUSP,SUSP,LCL ,LCA ,PSHC,POPC,MSIZ,
root/lib/u.h  21:   PSHG,POPG,NET1,NET2,NET3,NET4,NET5,NET6,NET7,NET8,NET9,
root/lib/u.h  22:   POW ,ATN2,FABS,ATAN,LOG ,LOGT,EXP ,FLOR,CEIL,HYPO,SIN ,COS ,TAN ,ASIN, // math
root/lib/u.h  23:   ACOS,SINH,COSH,TANH,SQRT,FMOD,
root/lib/u.h  24:   IDLE
root/lib/u.h  25: };
root/lib/u.h  26: 
root/lib/u.h  27: // system calls
root/lib/u.h  28: enum {
root/lib/u.h  29:   S_fork=1, S_exit,   S_wait,   S_pipe,   S_write,  S_read,   S_close,  S_kill,
root/lib/u.h  30:   S_exec,   S_open,   S_mknod,  S_unlink, S_fstat,  S_link,   S_mkdir,  S_chdir,
root/lib/u.h  31:   S_dup2,   S_getpid, S_sbrk,   S_sleep,  S_uptime, S_lseek,  S_mount,  S_umount,
root/lib/u.h  32:   S_socket, S_bind,   S_listen, S_poll,   S_accept, S_connect, 
root/lib/u.h  33: };
root/lib/u.h  34: 
root/lib/u.h  35: typedef unsigned char uchar;
root/lib/u.h  36: typedef unsigned short ushort;
root/lib/u.h  37: typedef unsigned int uint;
root/lib/u.h  38: 
root/usr/os/os0.c  4: 
root/usr/os/os0.c  5: int current;
root/usr/os/os0.c  6: 
root/usr/os/os0.c  7: out(port, val)  { asm(LL,8); asm(LBL,16); asm(BOUT); }
00000000  0000080e  LL    0x8 (D 8)
00000004  00001026  LBL   0x10 (D 16)
00000008  0000009a  BOUT                                            // output val at port
root/usr/os/os0.c  8: ivec(void *isr) { asm(LL,8); asm(IVEC); }
0000000c  00000002  LEV   0x0 (D 0)
00000010  0000080e  LL    0x8 (D 8)
00000014  000000a4  IVEC                                            // set interrupt vector
root/usr/os/os0.c  9: stmr(int val)   { asm(LL,8); asm(TIME); }
00000018  00000002  LEV   0x0 (D 0)
0000001c  0000080e  LL    0x8 (D 8)
00000020  000000a7  TIME                                            // set timer, timeout = val
root/usr/os/os0.c  10: halt(val)       { asm(LL,8); asm(HALT); }
00000024  00000002  LEV   0x0 (D 0)
00000028  0000080e  LL    0x8 (D 8)
0000002c  00000000  HALT                                            // halt system
root/usr/os/os0.c  11: 
root/usr/os/os0.c  12: alltraps()
00000030  00000002  LEV   0x0 (D 0)
root/usr/os/os0.c  13: {
root/usr/os/os0.c  14:   asm(PSHA);
00000034  0000009d  PSHA
root/usr/os/os0.c  15:   asm(PSHB);
00000038  000000a0  PSHB
root/usr/os/os0.c  16: 
root/usr/os/os0.c  17:   current++;
0000003c  00000015  LG    0x0 (D 0)
00000040  ffffff57  SUBI  0xffffffff (D -1)
00000044  00000045  SG    0x0 (D 0)
root/usr/os/os0.c  18: 
root/usr/os/os0.c  19:   asm(POPB);
00000048  000000a1  POPB
root/usr/os/os0.c  20:   asm(POPA);
0000004c  000000a3  POPA
root/usr/os/os0.c  21:   asm(RTI);
00000050  00000098  RTI                             // return from interrupt
root/usr/os/os0.c  22: }
root/usr/os/os0.c  23: 
root/usr/os/os0.c  24: main()
00000054  00000002  LEV   0x0 (D 0)
root/usr/os/os0.c  25: {
root/usr/os/os0.c  26:   current = 0;
00000058  00000023  LI    0x0 (D 0)
0000005c  00000045  SG    0x0 (D 0)                 // save current
root/usr/os/os0.c  27: 
root/usr/os/os0.c  28:   stmr(1000);
00000060  0003e89e  PSHI  0x3e8 (D 1000)
00000064  ffffb405  JSR   0xffffffb4 (TO 0x1c)
00000068  00000801  ENT   0x8                       // set timer
root/usr/os/os0.c  29:   ivec(alltraps);
0000006c  ffffc408  LEAG  0xffffffc4 (D -60)
00000070  0000009d  PSHA
00000074  ffff9805  JSR   0xffffff98 (TO 0x10)
00000078  00000801  ENT   0x8 (D 8)                 // set traps
root/usr/os/os0.c  30:   
root/usr/os/os0.c  31:   asm(STI);                  // set interrupt enable
0000007c  00000097  STI 
root/usr/os/os0.c  32:   
root/usr/os/os0.c  33:   while (current < 10) {
00000080  00000003  JMP   <fwd>
root/usr/os/os0.c  34:     if (current & 1) out(1, '1'); else out(1, '0');
00000084  00000015  LG    0x0 (D 0)             // a = current
00000088  00000169  ANDI  0x1 (D 1)             // a &= 1
0000008c  00000084  BZ    <fwd>
00000090  0000319e  PSHI  0x31 (D 49)           // push '1'
00000094  0000019e  PSHI  0x1 (D 1)             // push 1
00000098  ffff6405  JSR   0xffffff64 (TO 0x0)   // call out
0000009c  00001001  ENT   0x10 (D 16)
000000a0  00000003  JMP   <fwd>
000000a4  0000309e  PSHI  0x30 (D 48)           // push '0'
000000a8  0000019e  PSHI  0x1 (D 1)             // push 0
000000ac  ffff5005  JSR   0xffffff50 (TO 0x0)   // call out
000000b0  00001001  ENT   0x10 (D 16)
root/usr/os/os0.c  35:   }
root/usr/os/os0.c  36: 
root/usr/os/os0.c  37:   halt(0);
000000b4  00000015  LG    0x0 (D 0)
000000b8  00000a3b  LBI   0xa (D 10)
000000bc  0000008c  BLT   <fwd>
000000c0  0000009e  PSHI  0x0 (D 0)
000000c4  ffff6005  JSR   0xffffff60 (TO 0x28)
000000c8  00000801  ENT   0x8 (D 8)
root/usr/os/os0.c  38: }
root/usr/os/os0.c  39: 
000000cc  00000002  LEV   0x0 (D 0)
```
 
 - 尝试用xem的简单调试功能单步调试代码
   - 解答：同分析funcall.c时的操作方法。 
 - 回答如下问题：
   - 何处设置的中断使能？   
	 - 解答：```asm(STI)```。 
   - 系统何时处于中断屏蔽状态？
   	 - 解答：执行```asm(STI)```之前。 
   - 如果系统处于中断屏蔽状态，如何让其中断使能？
   	 - 解答，使用STI开始打开中断使能，使用CLI可以清除中断使能的flag。
   - 系统产生中断后，CPU会做哪些事情？（在没有软件帮助的情况下）
         - 解答： 如果中断前是用户态，需要切换到内核态（相应的TLB也设置为trk、twk），并需要将中断类型记录下来，再通过程序设置的中断向量ivec跳转到中断处理程序，// ? ? ?（在没有软件帮助的情况下不一定会执行RTI） 中断处理程序执行完后见下一问关于RTI的回答。
   - CPU执行RTI指令的具体完成工作是哪些？
   	 - 解答：RTI指令执行时，首先需要检查执行这条指令时是否在内核态（否则异常），然后根据进入中断处理之前是用户态还是内核态进行相应的切换，把之前暂时关闭的中断使能打开，并返回。 

## 分析和实验os1/os3.c [HARD] 
 
 - os1中的task1和task2的堆栈的起始和终止地址是什么？
 - os1是如何实现任务切换的？
 - os3中的task1和task2的堆栈的起始和终止地址是什么？
 - os3是如何实现任务切换的？
 - os3的用户态task能够破坏内核态的系统吗？
