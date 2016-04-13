### 虚拟页式存储的地址转换

（3）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持8KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

解题代码：
```c
#include <stdio.h>

int main() {
  FILE *file;
  int pdbr, mem[4096], disk[4096];
  int pde_idx, pde, pde_valid, pde_ptn;
  int pte_idx, pte, pte_valid, pte_pfn;
  int va, i, ra, val;

  pdbr = 0xd80;

  file = fopen("mem.txt", "r");
  for (i = 0; i < 4096; i++)
    fscanf(file, "%x", mem + i);
  fclose(file);

  file = fopen("disk.txt", "r");
  for (i = 0; i < 4096; i++)
    fscanf(file, "%x", disk + i);
  fclose(file);

  printf("Virtual Address 0x");
  scanf("%x", &va);

  pde_idx = (va >> 10) & 0x1F;
  pde = mem[pdbr + pde_idx];
  pde_valid = (pde >> 7) & 1;
  pde_ptn = pde & 0x7F;
  printf("  --> pde index: 0x%02x, pde contents: (valid %d, ptn 0x%02x)\n",
         pde_idx, pde_valid, pde_ptn);
  if (pde_valid) {
    pte_idx = (va >> 5) & 0x1F;
    pte = mem[(pde_ptn << 5) + pte_idx];
    pte_valid = (pte >> 7) & 1;
    pte_pfn = pte & 0x7F;
    printf("    --> pte index: 0x%02x, pte contents: (valid %d, pfn 0x%02x)\n",
           pte_idx, pte_valid, pte_pfn);
    ra = (pte_pfn << 5) + (va & 0x1F);
    if (pte_valid) {
      val = mem[ra];
      printf("      --> To Physical Address 0x%04x --> Value: 0x%02x\n",
             ra, val);
    } else {
      val = disk[ra];
      printf("      --> To Disk Sector Address 0x%04x --> Value: 0x%02x\n",
             ra, val);
    }
  } else {
    printf("    --> Fault (page directory entry not valid)\n");
  }

  return 0;
}
```

运行结果：
```
Virtual Address 0x6653
  --> pde index: 0x19, pde contents: (valid 0, ptn 0x7f)
    --> Fault (page directory entry not valid)
Virtual Address 0x1c13
  --> pde index: 0x07, pde contents: (valid 1, ptn 0x3d)
    --> pte index: 0x00, pte contents: (valid 1, pfn 0x76)
      --> To Physical Address 0x0ed3 --> Value: 0x12
Virtual Address 0x6890
  --> pde index: 0x1a, pde contents: (valid 0, ptn 0x7f)
    --> Fault (page directory entry not valid)
Virtual Address 0x0af6
  --> pde index: 0x02, pde contents: (valid 1, ptn 0x21)
    --> pte index: 0x17, pte contents: (valid 0, pfn 0x7f)
      --> To Disk Sector Address 0x0ff6 --> Value: 0x03
Virtual Address 0x1e6f
  --> pde index: 0x07, pde contents: (valid 1, ptn 0x3d)
    --> pte index: 0x13, pte contents: (valid 0, pfn 0x16)
      --> To Disk Sector Address 0x02cf --> Value: 0x1c
```
