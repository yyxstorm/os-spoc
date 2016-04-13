（spoc）根据你的`学号 mod 4`的结果值，确定选择四种页面置换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试。请参考如python代码或独自实现。
 - [页置换算法实现的参考实例](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab3/page-replacement-policy.py)

- （1）工作集页置换算法模拟如下：
```c
#include <stdio.h>

#define visits_sz 13
#define pages_num 5

int main() {
  int tau = 4;
  int visits[visits_sz] = {4, 3, 0, 2, 2, 3, 1, 2, 4, 2, 4, 0, 3};
  int exists[pages_num] = {0, 0, 0, 0, 0};
  int i, j;

  for (i = 0; i < visits_sz; i++) {
    if (!exists[visits[i]]) {
      printf("Miss page %d, ", visits[i]);
    } else {
      printf("Hit  page %d, ", visits[i]);
    }
    for (j = 0; j < pages_num; j++)
      exists[j] = 0;
    for (j = (i >= tau - 1 ? i - tau + 1 : 0); j <= i; j++)
      exists[visits[j]] = 1;
    printf("work set is now {");
    for (j = 0; j < pages_num; j++) {
      if (exists[j])
        printf(" %d", j);
    }
    printf(" }.\n");
  }

  return 0;
}
```
```
Miss page 4, work set is now { 4 }.
Miss page 3, work set is now { 3 4 }.
Miss page 0, work set is now { 0 3 4 }.
Miss page 2, work set is now { 0 2 3 4 }.
Hit  page 2, work set is now { 0 2 3 }.
Hit  page 3, work set is now { 0 2 3 }.
Miss page 1, work set is now { 1 2 3 }.
Hit  page 2, work set is now { 1 2 3 }.
Miss page 4, work set is now { 1 2 3 4 }.
Hit  page 2, work set is now { 1 2 4 }.
Hit  page 4, work set is now { 2 4 }.
Miss page 0, work set is now { 0 2 4 }.
Miss page 3, work set is now { 0 2 3 4 }.
```

- （2）缺页率置换算法模拟如下：
```cpp
#include <iostream>

using namespace std;

bool accessed[10] = {0};
bool inMem[10] = {0};

const int T = 3;

int t_last, t;

int page;

int main() {
	t = t_last = 0;
	for (int i = 0; i < 10; ++i) inMem[i] = false;
	for (int i = 0; i < 10; ++i) accessed[i] = false;
	
	while (cin >> page) {
		t++;
		if (page > 9 || page < 0) {
			cout << "Page number must be in [0,9]";
			continue;
		}
		cout << "Accessed page: " << page << ". ";
		if (inMem[page]) {
			accessed[page] = true;
			cout << "Hit!  ";
			cout << "Page in memory:";
			for (int i = 0; i < 10; ++i) if (inMem[i]) cout << " " << i;
			cout << endl;
			continue;
		}
		cout << "Miss! ";
		if (t - t_last > T)
			for (int i = 0; i < 10; ++i) 
				if (inMem[i] && !accessed[i])
					inMem[i] = false;
		for (int i = 0; i < 10; ++i) accessed[i] = false;
		inMem[page] = accessed[page] = true;
		cout << "Page in memory:";
		for (int i = 0; i < 10; ++i) if (inMem[i]) cout << " " << i;
		cout << endl;
		t_last = t;
	}
	return 0;
}
```
```
0 1 2 3 1 3 3 1 1 3 4 0 5 2 3 2
Accessed page: 0. Miss! Page in memory: 0
Accessed page: 1. Miss! Page in memory: 0 1
Accessed page: 2. Miss! Page in memory: 0 1 2
Accessed page: 3. Miss! Page in memory: 0 1 2 3
Accessed page: 1. Hit!  Page in memory: 0 1 2 3
Accessed page: 3. Hit!  Page in memory: 0 1 2 3
Accessed page: 3. Hit!  Page in memory: 0 1 2 3
Accessed page: 1. Hit!  Page in memory: 0 1 2 3
Accessed page: 1. Hit!  Page in memory: 0 1 2 3
Accessed page: 3. Hit!  Page in memory: 0 1 2 3
Accessed page: 4. Miss! Page in memory: 1 3 4
Accessed page: 0. Miss! Page in memory: 0 1 3 4
Accessed page: 5. Miss! Page in memory: 0 1 3 4 5
Accessed page: 2. Miss! Page in memory: 0 1 2 3 4 5
Accessed page: 3. Hit!  Page in memory: 0 1 2 3 4 5
Accessed page: 2. Hit!  Page in memory: 0 1 2 3 4 5
```
