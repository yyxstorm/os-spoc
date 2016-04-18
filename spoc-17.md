(5)理解并实现stride调度算法

可基于“python, ruby, C, C++，LISP、JavaScript”等语言模拟实现，并给出测试，在试验报告写出设计思路和测试结果分析。

请参考scheduler-homework.py代码或独自实现。 最后统计采用不同调度算法的每个任务的相关时间和总体的平均时间：

　- turnaround time　周转时间 　- response time 响应时间 　- wait time　等待时间

对模拟环境的抽象

任务/进程，及其执行时间 Job 0 (length = 1) Job 1 (length = 4) Job 2 (length = 7)

何时切换？
如何统计？

Code:
```cpp
#include <iostream>

const int MAX = 10;

int tasktime[MAX] = {0};
int task[MAX] = {0};
int stride[MAX] = {0};
int pass[MAX] = {0};

int response[MAX];
int turnaround[MAX];

int tasknum = 0;
int time = 0;

using namespace std;

void schedule();

void addTask(int time, int priority) {
    tasktime[tasknum] = task[tasknum] = time;
    response[tasknum] = -1;
    pass[tasknum] = (int)(10/priority);
    tasknum++;
}

void run(int t) {
    cout << "  [ time " << time << " ] Run job " << t << " for 1.00 secs  stride = " << stride[t] << "  pass = " << pass[t] << endl;
    if (response[t] == -1) response[t] = time;
    task[t]--;
    time++;
    if (task[t] == 0) turnaround[t] = time;
    stride[t] += pass[t];
    schedule();
}

void schedule() {
    int min = -1, mini = -1;
    for (int i = 0; i < tasknum; ++i) {
        if (task[i] > 0 && (mini == -1 || stride[i] < min || (stride[i] == min && pass[i] < pass[mini]))) {
            mini = i;
            min = stride[i];
        }
    }
    if (mini > -1) run(mini);
}

int main() {
    addTask(9, 5);
    addTask(8, 2);
    addTask(5, 1);
    
    cout << "stride scheduling simulation" << endl;
    cout << "Job list: " << endl;
    for (int i = 0; i < tasknum; ++i) cout << "  job " << i << ": " << task[i] << " seconds" << endl;
    
    cout << "Execution trace:" << endl;
    schedule();
    
    cout << "Final statistics:" << endl;
    double rsum = 0, tsum = 0, wsum = 0;
    for (int i = 0; i < tasknum; ++i) {
        rsum += response[i];
        tsum += turnaround[i];
        wsum += turnaround[i] - tasktime[i];
        cout << "  Job " << i << " --- Response: " << response[i] << " secs  Turnaround: " << turnaround[i] << " secs  Wait: " << turnaround[i] - tasktime[i] << " secs" << endl;
    }
    cout << "  Average --- Response: " << rsum / tasknum << " secs  Turnaround: " << tsum / tasknum << " secs  Wait: " << wsum / tasknum << " secs";

    return 0;
}
```

Output:
```
stride scheduling simulation
Job list:
  job 0: 9 seconds
  job 1: 8 seconds
  job 2: 5 seconds
Execution trace:
  [ time 0 ] Run job 0 for 1.00 secs  stride = 0  pass = 2
  [ time 1 ] Run job 1 for 1.00 secs  stride = 0  pass = 5
  [ time 2 ] Run job 2 for 1.00 secs  stride = 0  pass = 10
  [ time 3 ] Run job 0 for 1.00 secs  stride = 2  pass = 2
  [ time 4 ] Run job 0 for 1.00 secs  stride = 4  pass = 2
  [ time 5 ] Run job 1 for 1.00 secs  stride = 5  pass = 5
  [ time 6 ] Run job 0 for 1.00 secs  stride = 6  pass = 2
  [ time 7 ] Run job 0 for 1.00 secs  stride = 8  pass = 2
  [ time 8 ] Run job 0 for 1.00 secs  stride = 10  pass = 2
  [ time 9 ] Run job 1 for 1.00 secs  stride = 10  pass = 5
  [ time 10 ] Run job 2 for 1.00 secs  stride = 10  pass = 10
  [ time 11 ] Run job 0 for 1.00 secs  stride = 12  pass = 2
  [ time 12 ] Run job 0 for 1.00 secs  stride = 14  pass = 2
  [ time 13 ] Run job 1 for 1.00 secs  stride = 15  pass = 5
  [ time 14 ] Run job 0 for 1.00 secs  stride = 16  pass = 2
  [ time 15 ] Run job 1 for 1.00 secs  stride = 20  pass = 5
  [ time 16 ] Run job 2 for 1.00 secs  stride = 20  pass = 10
  [ time 17 ] Run job 1 for 1.00 secs  stride = 25  pass = 5
  [ time 18 ] Run job 1 for 1.00 secs  stride = 30  pass = 5
  [ time 19 ] Run job 2 for 1.00 secs  stride = 30  pass = 10
  [ time 20 ] Run job 1 for 1.00 secs  stride = 35  pass = 5
  [ time 21 ] Run job 2 for 1.00 secs  stride = 40  pass = 10
Final statistics:
  Job 0 --- Response: 0 secs  Turnaround: 15 secs  Wait: 6 secs
  Job 1 --- Response: 1 secs  Turnaround: 21 secs  Wait: 13 secs
  Job 2 --- Response: 2 secs  Turnaround: 22 secs  Wait: 17 secs
  Average --- Response: 1 secs  Turnaround: 19.3333 secs  Wait: 12 secs
```
