## Bakery算法若去除choosing会如何

答：若没有choosing，一种出错的情形如下：
 - 为简单起见，设只有两个进程，且初始时NUM[i] == 0。
 - 首先，两个进程都运行完`MAX(...)+1`一句，计算结果是`1`。
 - PID较小的进程还未给`NUM[i]`赋值，而PID较大的进程先执行了对`NUM[i]`的赋值、且通过检验并进入临界区。
 - PID较小的进程终于给`NUM[i]`赋值了，但由于`NUM[i]`都是1、且它的PID又较小，它也通过检验并进入临界区。

## 简化x86计算机模拟器上的同步算法

#### 1. 阅读[简化x86计算机模拟器的使用说明](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/lab7-spoc-exercise.md)，理解基于简化x86计算机的汇编代码。

答：略。

#### 2. 了解race condition. 进入[race-condition代码目录](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/race-condition)。

- 执行 `./x86.py -p loop.s -t 1 -i 100 -R dx`， 请问`dx`的值是什么？
 - 答：-1（看代码，或使用`./x86.py -p loop.s -t 1 -i 100 -R dx -c`可获得，即增加`-c`）。
- 执行 `./x86.py -p loop.s -t 2 -i 100 -a dx=3,dx=3 -R dx` ， 请问`dx`的值是什么？
 - 答：对两个线程都是-1。
- 执行 `./x86.py -p loop.s -t 2 -i 3 -r -a dx=3,dx=3 -R dx`， 请问`dx`的值是什么？
 - 答：都是-1。
- 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 1 -M 2000`, 请问变量x的值是什么？
 - 答：1。
- 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -a bx=3 -M 2000`, 请问变量x的值是什么？为何每个线程要循环3次？
 - 答：6。循环3次是因为给的bx初值是3，每次循环减一减到0才终止。
- 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 0`， 请问变量x的值是什么？
 - 答：1或2（请去掉`-s 0`这个参数，这个参数是设置随机数种子的）。
- 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 1`， 请问变量x的值是什么？
 - 答：1或2。
- 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 2`， 请问变量x的值是什么？ 
 - 答：1或2。
- 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -a bx=1 -t 2 -M 2000 -i 1`， 请问变量x的值是什么？ 
 - 答：1。

#### 3. 了解software-based lock, hardware-based lock, [software-hardware-lock代码目录](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/software-hardware-locks)

- 理解flag.s,peterson.s,test-and-set.s,ticket.s,test-and-test-and-set.s 请通过x86.py分析这些代码是否实现了锁机制？请给出你的实验过程和结论说明。
 - 答：实现了，x86.py中有如下代码用来实现锁机制：
     ```python
     #
     # SUPPORT FOR LOCKS
     #
     def atomic_exchange(self, src, value, reg1, reg2):
         tmp                 = value + self.registers[reg1] + self.registers[reg2]
         old                 = self.memory[tmp]
         self.memory[tmp]    = self.registers[src]
         self.registers[src] = old
         return 0
   
     def fetchadd(self, src, value, reg1, reg2):
         tmp                 = value + self.registers[reg1] + self.registers[reg2]
         old                 = self.memory[tmp]
         self.memory[tmp]    = self.memory[tmp] + self.registers[src] 
         self.registers[src] = old
     ```

- 能否设计新的硬件原子操作指令Compare-And-Swap,Fetch-And-Add？
 - 答：题目的意思是？看不懂。 
