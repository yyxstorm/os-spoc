## 小组思考题
---

 (2) (spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持32KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries， PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐），其值为0x220（十进制为544），

PTE格式（8 bit） :
```
  VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
  VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示映射不存在。
PFN6..0:页帧号
PT6..0:页表的物理基址>>5
```
在[物理内存模拟数据文件](https://github.com/chyyuu/os_course_spoc_exercises/blob/master/all/03-2-spoc-testdata.md)中，给出了4KB物理内存空间的值，请回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents。
```
Virtual Address 6c74
Virtual Address 6b22
```

答：
```
Virtual Address 0x6c74:
    --> pde index: 0x1b pde contents: (valid 1, ptn 0x20)
        --> pte index: 0x3 pte contents: (valid 1, pfn 0x61)
            --> Translated to Physical Address 0xc34 --> Value: 6
Virtual Address 0x6b22:
    --> pde index: 0x1a pde contents: (valid 1, ptn 0x52)
        --> pte index: 0x19 pte contents: (valid 1, pfn 0x47)
            --> Translated to Physical Address 0x8e2 --> Value: 26
```

（3）请基于你对原理课二级页表的理解，并参考Lab2建页表的过程，设计一个应用程序（可基于python, ruby, C, C++，LISP等）可模拟实现(2)题中描述的抽象OS，可正确完成二级页表转换。

```c
#include <iostream>
#include <fstream>
#include <string>

using namespace std;

int mem[4096][8];

int num(int* a, int n = 5) {
	int temp = 0;
	for (int i = 0; i < n; ++i) temp = 2 * temp + a[i];
	return temp;
}

void print(int* a, int n = 8) {
	for (int i = 0; i < n; ++i) cout << a[i];
	cout << endl;
}

int main() {
	ifstream fin("data.txt");
	for (int i = 0; i < 4096; ++i) {
		string s;
		fin >> s;
		int temp;
		temp = s[0] - 48;
		if (temp >= 10 || temp < 0) temp = temp + 48 - 97 + 10;
		mem[i][3] = temp % 2; temp = temp / 2;
		mem[i][2] = temp % 2; temp = temp / 2;
		mem[i][1] = temp % 2; temp = temp / 2;
		mem[i][0] = temp % 2; temp = temp / 2;
		
		temp = s[1] - 48;
		if (temp >= 10 || temp < 0) temp = temp + 48 - 97 + 10;
		mem[i][7] = temp % 2; temp = temp / 2;
		mem[i][6] = temp % 2; temp = temp / 2;
		mem[i][5] = temp % 2; temp = temp / 2;
		mem[i][4] = temp % 2; temp = temp / 2;
	}
	
	int v[15] = {0};
	for (int i = 0; i < 15; ++i) {
		char temp;
		cin >> temp;
		if (temp == '0') v[i] = 0; else v[i] = 1;
 	}
	
	int pdn = 544 + num(v);
	
	cout << endl << pdn << endl;
	print(mem[pdn]);
	cout << endl;
	
	if (mem[pdn][0] == 0) {
		cout << "page directory entry not valid";
		return 0;
	}
	
	int ptn = num(mem[pdn]+1, 7) * 32 + num(v+5);
		
	cout << endl << ptn << endl;
	print(mem[ptn]);
	cout << endl;
	
	if (mem[ptn][0] == 0) {
		cout << "page table entry not valid";
		return 0;
	}
	
	int addr = num(mem[ptn]+1, 7) * 32 + num(v+10);
	
	int value = num(mem[addr], 8);
	
	cout << addr << endl;
	cout << "value = " << value;
	
	return 0;
}
```
