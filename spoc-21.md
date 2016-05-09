#### (1) cvp->count含义是什么？cvp->count是否可能<0, 是否可能>1？请举例或说明原因。
#### (2) cvp->owner->next_count含义是什么？cvp->owner->next_count是否可能<0, 是否可能>1？请举例或说明原因。
next_count的含义是“因为发出signal而进入sleep的进程个数”，这点从下面的代码可以看出。
```
cond_signal(cv) {
   if(cv.count>0) {
      mt.next_count ++;
      signal(cv.sem);
      wait(mt.next);
      mt.next_count--;
   }
}
```
#### (3) 目前的lab7-answer中管程的实现是Hansen管程类型还是Hoare管程类型？请在lab7-answer中实现另外一种类型的管程。
