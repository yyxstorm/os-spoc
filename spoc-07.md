# lab2 SPOC思考题

## v9-cpu相关

[challenge]在v9-cpu上，设定物理内存为64MB。在os2.c和os4.c的基础上实现页机制管理，内核空间的映射关系： kernel_virt_addr=0xc00000000+phy_addr，内核空间大小为64MB，虚拟空间范围为0xc0000000--x0xc4000000, 物理空间范围为0x00000000--x0x04000000；用户空间的映射关系：user_virt_addr=0x40000000+usr_phy_addr，用户空间可用大小为1MB，虚拟空间范围为0x40000000--0x40100000，物理空间范围为0x02000000--x0x02100000。可参考v9-cpu git repo的testing分支中的os.c和mem.h。修改代码为[os5.c](https://github.com/chyyuu/v9-cpu/blob/master/root/usr/os/os5.c)

- (1)在内核态可正确访问这两个空间
- (2)在用户态可正确访问这两个空间

Solution:
```c
setup_kernel_paging()
{
  int i, j;
  
  // Align the page directory so as to ensure that it lies in a single 4KB frame. 
  pg_dir = (int *)((((int)&pg_mem) + 4095) & -4096);
  for (i = 0; i < 1024; ++i) pg_dir[i] = 0;
  
  // We need 64MB (for kernel) + 1MB (for user) virtual memory.
  // Since each page frame occupies 4KB and each page table contains 1K entries,
  // it follows that each directory entry corresponds to 4MB memory.
  // Therefore, we need 16 (for kernel) + 1 (for user) page tables in total.
  for (i = 0; i < 16; ++i) {
  
    // Note that ``pg_dir`` is of type ``int *``. Hence ``pg_dir + 1`` actually 
    // offsets it by 4 bytes. As a result, ``pg_dir + 1024 * (i + 1)`` is the 
    // ``i``th frame that follows immediately after the frame that contains 
    // the page directory. And ``pg_tbl[i]`` is the address to the frame that
    // contains the ``i``th page table.
    pg_tbl[i] = pg_dir + 1024 * (i + 1);
    
    // The frame assigned for the page directory has 4KB, meaning we can have
    // 1K page directory entires, which correspond to 1K * 4MB = 4GB virtual memory
    // in total. However, we only need 64MB virutal memory for kernel. That is,
    // we only need 0xc0000000 ~ x0xc4000000 for kernel, which corresponds to
    // the 768th, 769th, 770th, ..., 783th entries.
    pg_dir[i+768] = (int)pg_tbl[i] | PTE_P | PTE_W;
  }
  
  // Identity mapping for address 0x00000000~0x00400000.
  pg_dir[0] = pg_dir[768];
  
  for (i = 0; i < 16; ++i)
    for (j = 0; j < 1024; ++j)
      pg_tbl[i][j] = (i * 1024 * 4096 + j * 4096) | PTE_P | PTE_W;
}

setup_user_paging()
{
  int i;
  
  // Virtual addresses 0x40000000 ~ 0x40100000 correspond to
  // the 256th directory entry. And the physical addresses 
  // 0x02000000--x0x02100000 they map to happen to be the same
  // as those of virtual address 0xc2000000 ~ 0xc2100000, which
  // correspond to the ``768 + 8``th directory entry.
  pg_dir[256] = pg_dir[768 + 8];
  
  // Each page table contains 1024 entries, corresponding to 4MB. 
  // But we only need 1MB for user. Hence, only give the first 256 
  // entries the PTE_U flag here.
  for (i = 0; i < 256; ++i) pg_tbl[8][i] |= PTE_U;
}
```
