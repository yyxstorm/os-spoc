#### 理解虚存机制是如何在v9或ucore上实现的，思考如何实现clock页替换算法，给出概要设计方案。

以下讨论基于v9上的os_lab3_1.c。

首先让我们看看缺页异常处理例程是怎么样的：
```c
PGFAULT:
  va = lvadr();
  if (va >= init->sz) {
    printf(">= init->sz 0x%x ", init->sz);
    panic("!\n");
  } else {
    pc--; // restart instruction
    ptep = get_pte(init->pdir, va);
    if (*ptep == 0) {
      if (mem_maps_num >= UPHYSZ / PAGE) { // need swap out
        pa = swap_out(init->pdir);
        printf("swap out a page\n");
        mappage(init->pdir, va & -PAGE, pa & -PAGE, PTE_P | PTE_W | PTE_U);
      } else {
        printf("ADD new phy page\n");
        mappage(init->pdir, va & -PAGE, V2P + (memset(kalloc(), 0, PAGE)),
                PTE_P | PTE_W | PTE_U);
      }
    } else {                               //*ptep !=0    //need swap in
      if (mem_maps_num >= UPHYSZ / PAGE) { // need swap out
        sector_num = (*ptep) >> 8;
        pa = swap_out(init->pdir);
        printf("swap out a page\n");
        swap_in(sector_num, P2V + pa);
        printf("swap out a page\n");
        mappage(init->pdir, va & -PAGE, pa & -PAGE, PTE_P | PTE_W | PTE_U);

      } else {
        mappage(init->pdir, va & -PAGE, V2P + (memset(kalloc(), 0, PAGE)),
                PTE_P | PTE_W | PTE_U);
        swap_in(init->pdir, V2P + (memset(kalloc(), 0, PAGE)));
      }
    }
    return;
  }
```
从中我们可以得知```swap_out```在物理空间不足的时候，它应当：（1）选取已经映射到物理内存的一个页；（2）把这个页换出到硬盘，腾出对应的物理页；（3）返回新腾出来的这个物理页。因此，```swap_out```是我们重点要整治的对象（要实现clock算法就需要修改它）。

让我们再看看```swap_out```现有的（存在问题的）实现：
```c
uint swap_out(uint *pd) {
  uint *pde, *pte, *pt;
  uint pa, mi, di, found;

  mi = avail_mem();
  di = avail_disk();
  if (mi == -1 || di == -1)
    panic("no available mem or disk\n");

  pde = &pd[mi * PAGE >> 22];
  pt = P2V + (*pde & -PAGE);
  pte = &pt[(mi * PAGE >> 12) & 0x3ff];
  pa = (*pte) & -PAGE;
  *pte = di << 8;
  iderw(di, P2V + pa, IDE_W);

  pdir(V2P + pd);

  return pa;
}
```
其中，```avail_mem```和```avail_disk```的作用分别是找出一个没有被映射到具体物理页帧的虚址```mi```和一个没有被映射到磁盘扇区的虚址```di```。注意，返回的是**没有**映射关系的**虚址**。什么，你不相信？那看代码咯：
```c
int avail_mem()
{
	int i;
	for(i=0;i<UVIRSZ/PAGE;i++) {
		if (mem_maps[i]==-1)
			return i;
	}
	return -1;
}
int avail_disk()
{
	int i;
	for(i=0;i<UVIRSZ/PAGE;i++) {
	if (disk_maps[i]==-1)
		return i;
	}
	return -1;
}
```
找不到则返回```-1```，但要正常继续就必须要求两者都能找到。
实际上，这里说```di```是虚址可能会让人感到困惑，毕竟它映射过去的是磁盘扇区而不是物理内存的页帧，但os_lab_3_1.c的代码确实是用虚址空间来伪装硬盘的，这点可以从```iderw```看出。

再回到```swap_out```，在找到```mi```和```di```后，它就把```mi```对应的页的东西拷贝到```di```对应的硬盘扇区中（```iderw```一句）。
之后画风骤变，突然来一句：
```c
pdir(V2P+pd);
```
同时模拟器在这句话上崩溃，发出```PANIC::FMEM from kernel```。可见，这次的代码并没有给出一个正确、完整的虚存实现。

那么究竟错在哪里，又缺了什么呢？

首先，错在```mi```上，```swap_out```中下述代码片段的意思明明是：（1）把```mi```和它对应的物理页的映射关系取消，并把这个物理页作为空闲页（函数的返回值）；（2）把```mi```改为映射到```di```对应的磁盘扇区上，并把内容复制过去。
```
  pde = &pd[mi * PAGE >> 22];
  pt = P2V + (*pde & -PAGE);
  pte = &pt[(mi * PAGE >> 12) & 0x3ff];
  pa = (*pte) & -PAGE;
  *pte = di << 8;
  iderw(di, P2V + pa, IDE_W);
```
但正如前面我们讨论的，```avail_mem```返回的明明是**没有**映射关系的页。（如果我错了，求打脸。）

这也启示我们，我们要实现clock算法，可以在```avail_mem```里实现。
另外，我觉的disk方面似乎也有点奇怪，限于时间，就不继续了（ 我也想继续啊，但生活不只诗和远方，还有眼前的苟且:( )。
