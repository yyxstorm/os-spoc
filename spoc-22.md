#### 问题1： 根据[sfs文件系统的状态变化信息](https://github.com/chyyuu/ucore_os_lab/blob/master/related_info/lab8/sfs_states.txt)，给出具体的文件相关操作内容

```
ARG seed 0
ARG numInodes 8
ARG numData 8
ARG numRequests 10
ARG reverse False
ARG printFinal False

Initial state

inode bitmap  10000000
inodes        [d a:0 r:2] [] [] [] [] [] [] [] 
data bitmap   10000000
data          [(.,0) (..,0)] [] [] [] [] [] [] [] 

mkdir("/g");

inode bitmap  11000000
inodes        [d a:0 r:3] [d a:1 r:2] [] [] [] [] [] [] 
data bitmap   11000000
data          [(.,0) (..,0) (g,1)] [(.,1) (..,0)] [] [] [] [] [] [] 

creat("/q");

inode bitmap  11100000
inodes        [d a:0 r:4] [d a:1 r:2] [f a:-1 r:1] [] [] [] [] [] 
data bitmap   11000000
data          [(.,0) (..,0) (g,1) (q,2)] [(.,1) (..,0)] [] [] [] [] [] [] 

creat("/u");

inode bitmap  11110000
inodes        [d a:0 r:5] [d a:1 r:2] [f a:-1 r:1] [f a:-1 r:1] [] [] [] [] 
data bitmap   11000000
data          [(.,0) (..,0) (g,1) (q,2) (u,3)] [(.,1) (..,0)] [] [] [] [] [] [] 

link("/u", "/x");

inode bitmap  11110000
inodes        [d a:0 r:6] [d a:1 r:2] [f a:-1 r:1] [f a:-1 r:2] [] [] [] [] 
data bitmap   11000000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (x,3)] [(.,1) (..,0)] [] [] [] [] [] [] 

mkdir("/t");

inode bitmap  11111000
inodes        [d a:0 r:7] [d a:1 r:2] [f a:-1 r:1] [f a:-1 r:2] [d a:2 r:2] [] [] [] 
data bitmap   11100000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (x,3) (t,4)] [(.,1) (..,0)] [(.,4) (..,0)] [] [] [] [] [] 

creat("/g/c");

inode bitmap  11111100
inodes        [d a:0 r:7] [d a:1 r:3] [f a:-1 r:1] [f a:-1 r:2] [d a:2 r:2] [f a:-1 r:1] [] [] 
data bitmap   11100000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (x,3) (t,4)] [(.,1) (..,0) (c,5)] [(.,4) (..,0)] [] [] [] [] [] 

unlink("/x");

inode bitmap  11111100
inodes        [d a:0 r:6] [d a:1 r:3] [f a:-1 r:1] [f a:-1 r:1] [d a:2 r:2] [f a:-1 r:1] [] [] 
data bitmap   11100000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (t,4)] [(.,1) (..,0) (c,5)] [(.,4) (..,0)] [] [] [] [] [] 

mkdir("/g/w");

inode bitmap  11111110
inodes        [d a:0 r:6] [d a:1 r:4] [f a:-1 r:1] [f a:-1 r:1] [d a:2 r:2] [f a:-1 r:1] [d a:3 r:2] [] 
data bitmap   11110000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (t,4)] [(.,1) (..,0) (c,5) (w,6)] [(.,4) (..,0)] [(.,6) (..,1)] [] [] [] [] 

fd=open("/g/c", O_WRONLY|O_APPEND); write(fd, buf, BLOCKSIZE); close(fd);

inode bitmap  11111110
inodes        [d a:0 r:6] [d a:1 r:4] [f a:-1 r:1] [f a:-1 r:1] [d a:2 r:2] [f a:4 r:1] [d a:3 r:2] [] 
data bitmap   11111000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (t,4)] [(.,1) (..,0) (c,5) (w,6)] [(.,4) (..,0)] [(.,6) (..,1)] [o] [] [] [] 

creat("/n");

inode bitmap  11111111
inodes        [d a:0 r:7] [d a:1 r:4] [f a:-1 r:1] [f a:-1 r:1] [d a:2 r:2] [f a:4 r:1] [d a:3 r:2] [f a:-1 r:1] 
data bitmap   11111000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (t,4) (n,7)] [(.,1) (..,0) (c,5) (w,6)] [(.,4) (..,0)] [(.,6) (..,1)] [o] [] [] [] 
```

#### 问题2：在[sfs-homework.py 参考代码的基础上](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab8/sfs-homework.py)，实现 `writeFile, createFile, createLink, deleteFile`，使得你的实现能够达到与问题1的正确结果一致

```python
...
    def deleteFile(self, tfile):
        if printOps:
            print 'unlink("%s");' % tfile

        inum = self.nameToInum[tfile]

        # YOUR CODE, YOUR ID
        inode = self.inodes[inum]
        inode.decRefCnt()
        if inode.refCnt == 0:
            if inode.getAddr() != -1:
                self.dataFree(inode.getAddr())
            self.inodeFree(inum)
        parent_inode = self.inodes[
            self.nameToInum[self.getParent(tfile)]
        ]
        parent_inode.decRefCnt()
        self.data[parent_inode.getAddr()].delDirEntry(tfile)

        # finally, remove from files list
        self.files.remove(tfile)
        return 0

    def createLink(self, target, newfile, parent):

        # YOUR CODE, YOUR ID
        parent_inum = self.nameToInum[parent]
        parent_data = self.data[
            self.inodes[parent_inum].getAddr()
        ]
        if parent_data.getFreeEntries() == 0:
            return -1
        if parent_data.dirEntryExists(newfile):
            return -1
        target_inum = self.nameToInum[target]
        self.inodes[target_inum].incRefCnt()
        self.inodes[parent_inum].incRefCnt()
        parent_data.addDirEntry(newfile, target_inum)

        return target_inum

    def createFile(self, parent, newfile, ftype):

        # YOUR CODE, YOUR ID
        parent_inum = self.nameToInum[parent]
        parent_inode = self.inodes[parent_inum]
        parent_data = self.data[parent_inode.getAddr()]
        if parent_data.getFreeEntries() == 0:
            return -1
        if parent_data.dirEntryExists(newfile):
            return -1
        inum = self.inodeAlloc()
        parent_inode.incRefCnt()
        parent_data.addDirEntry(newfile, inum)
        if ftype == 'd':
            dnum = self.dataAlloc()
            self.inodes[inum].setAll('d', dnum, 1)
            data = self.data[dnum]
            data.setType('d')
            data.addDirEntry('.', inum)
            data.addDirEntry('..', parent_inum)
            self.inodes[inum].incRefCnt()
        else:
            self.inodes[inum].setType(ftype)

        return inum

    def writeFile(self, tfile, data):
        inum = self.nameToInum[tfile]
        curSize = self.inodes[inum].getSize()
        dprint('writeFile: inum:%d cursize:%d refcnt:%d' % (
            inum, curSize, self.inodes[inum].getRefCnt()))

        # YOUR CODE, YOUR ID
        if curSize == 1:  # is full
            return -1
        dnum = self.dataAlloc()
        self.data[dnum].setType('f')
        self.data[dnum].addData(data)
        self.inodes[inum].setAddr(dnum)

        if printOps:
            print 'fd=open("%s", O_WRONLY|O_APPEND); write(fd, buf, BLOCKSIZE); close(fd);' % tfile
        return 0
...
```

#### 问题3：实现`soft link`机制，并设计测试用例说明你实现的正确性

给`fs`类增加`doSoftLink`和`createSoftLink`函数。
```python
...
    ...
    def createSoftLink(self, target, newfile, parent):
        parent_inode = self.inodes[self.nameToInum[parent]]
        pdata = self.data[parent_inode.getAddr()]
        if pdata.getFreeEntries() == 0:
            return -1
        if pdata.dirEntryExists(newfile):
            return -1
        inum = self.inodeAlloc()
        pdata.addDirEntry(newfile, inum)
        parent_inode.incRefCnt()
        dnum = self.dataAlloc()
        self.inodes[inum].setAll('f', dnum, 1)
        self.data[dnum].setType('f')
        self.data[dnum].addData('-> %s' % target)
        return inum
    ...
    def doSoftLink(self):
        # 和doLink几乎一模一样，就是把输出信息改了下而已
        dprint('doSoftLink')
        if len(self.files) == 0:
            return -1
        parent = self.dirs[int(random.random()* len(self.dirs))]
        nfile = self.makeName()

        # pick random target
        target = self.files[int(random.random() * len(self.files))]

        # get full name of newfile
        if parent == '/':
            fullName = parent + nfile
        else:
            fullName = parent + '/' + nfile

        dprint('try createSoftLink(%s %s %s)' % (target, nfile, parent))
        inum = self.createSoftLink(target, nfile, parent)
        if inum >= 0:
            self.files.append(fullName)
            self.nameToInum[fullName] = inum
            if printOps:
                print 'softlink("%s", "%s");' % (target, fullName)
            return 0
        return -1
...
```

并把程序中的下面这个片段
```python
...
                elif r < 0.7:
                    rc = self.doLink()
                    dprint('doLink rc:%d' % rc)
                else:
...
```
改成
```python
...
                elif r < 0.7:
                    rc = self.doSoftLink()
                    dprint('doSoftLink rc:%d' % rc)
                else:
...
```
当然，同时又硬链接、软链接也没什么问题，我只是懒得改了。

下面是某次输出：
```
RG seed 0
ARG numInodes 8
ARG numData 8
ARG numRequests 10
ARG reverse False
ARG printFinal False

Initial state

inode bitmap  10000000
inodes        [d a:0 r:2] [] [] [] [] [] [] [] 
data bitmap   10000000
data          [(.,0) (..,0)] [] [] [] [] [] [] [] 

mkdir("/g");

inode bitmap  11000000
inodes        [d a:0 r:3] [d a:1 r:2] [] [] [] [] [] [] 
data bitmap   11000000
data          [(.,0) (..,0) (g,1)] [(.,1) (..,0)] [] [] [] [] [] [] 

creat("/q");

inode bitmap  11100000
inodes        [d a:0 r:4] [d a:1 r:2] [f a:-1 r:1] [] [] [] [] [] 
data bitmap   11000000
data          [(.,0) (..,0) (g,1) (q,2)] [(.,1) (..,0)] [] [] [] [] [] [] 

creat("/u");

inode bitmap  11110000
inodes        [d a:0 r:5] [d a:1 r:2] [f a:-1 r:1] [f a:-1 r:1] [] [] [] [] 
data bitmap   11000000
data          [(.,0) (..,0) (g,1) (q,2) (u,3)] [(.,1) (..,0)] [] [] [] [] [] [] 

softlink("/u", "/x");

inode bitmap  11111000
inodes        [d a:0 r:6] [d a:1 r:2] [f a:-1 r:1] [f a:-1 r:1] [f a:2 r:1] [] [] [] 
data bitmap   11100000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (x,4)] [(.,1) (..,0)] [-> /u] [] [] [] [] [] 

mkdir("/t");

inode bitmap  11111100
inodes        [d a:0 r:7] [d a:1 r:2] [f a:-1 r:1] [f a:-1 r:1] [f a:2 r:1] [d a:3 r:2] [] [] 
data bitmap   11110000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (x,4) (t,5)] [(.,1) (..,0)] [-> /u] [(.,5) (..,0)] [] [] [] [] 

creat("/g/c");

inode bitmap  11111110
inodes        [d a:0 r:7] [d a:1 r:3] [f a:-1 r:1] [f a:-1 r:1] [f a:2 r:1] [d a:3 r:2] [f a:-1 r:1] [] 
data bitmap   11110000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (x,4) (t,5)] [(.,1) (..,0) (c,6)] [-> /u] [(.,5) (..,0)] [] [] [] [] 

unlink("/x");

inode bitmap  11110110
inodes        [d a:0 r:6] [d a:1 r:3] [f a:-1 r:1] [f a:-1 r:1] [] [d a:3 r:2] [f a:-1 r:1] [] 
data bitmap   11010000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (t,5)] [(.,1) (..,0) (c,6)] [] [(.,5) (..,0)] [] [] [] [] 

mkdir("/g/w");

inode bitmap  11111110
inodes        [d a:0 r:6] [d a:1 r:4] [f a:-1 r:1] [f a:-1 r:1] [d a:2 r:2] [d a:3 r:2] [f a:-1 r:1] [] 
data bitmap   11110000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (t,5)] [(.,1) (..,0) (c,6) (w,4)] [(.,4) (..,1)] [(.,5) (..,0)] [] [] [] [] 

fd=open("/g/c", O_WRONLY|O_APPEND); write(fd, buf, BLOCKSIZE); close(fd);

inode bitmap  11111110
inodes        [d a:0 r:6] [d a:1 r:4] [f a:-1 r:1] [f a:-1 r:1] [d a:2 r:2] [d a:3 r:2] [f a:4 r:1] [] 
data bitmap   11111000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (t,5)] [(.,1) (..,0) (c,6) (w,4)] [(.,4) (..,1)] [(.,5) (..,0)] [o] [] [] [] 

creat("/n");

inode bitmap  11111111
inodes        [d a:0 r:7] [d a:1 r:4] [f a:-1 r:1] [f a:-1 r:1] [d a:2 r:2] [d a:3 r:2] [f a:4 r:1] [f a:-1 r:1] 
data bitmap   11111000
data          [(.,0) (..,0) (g,1) (q,2) (u,3) (t,5) (n,7)] [(.,1) (..,0) (c,6) (w,4)] [(.,4) (..,1)] [(.,5) (..,0)] [o] [] [] [] 
```
