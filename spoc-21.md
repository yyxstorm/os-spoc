#### (1) cvp->count含义是什么？cvp->count是否可能<0, 是否可能>1？请举例或说明原因。
count的含义是“因为要等待一个条件而进入sleep的进程个数”，这点从下面代码可以看出。
```
cond_wait (cv) {
    cv.count ++;
    if(mt.next_count>0)
       signal(mt.next)
    else
       signal(mt.mutex);
    wait(cv.sem);
    cv.count --;
 }
```
不能<0，可能>1。

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
不能<0。
大家都说不能>1，但是我还是有点想不通：如果我在monitor的一个方法里signal了，唤醒的另一个进程继续执行到一段时间后也发signal（此时之前因为signal而sleep的那个进程还没醒呢），不就变成2了吗，难道不存在这种情形？

#### (3-1) 目前的lab7-answer中管程的实现是Hansen管程类型还是Hoare管程类型？
是Hoare类型。理由如下。

首先贴一下管程的一个方法的实现方式，如下。
```
routineA_in_mt () {
    wait(mt.mutex);
    ...
    real body of routineA
    ...
    if(next_count>0)
        signal(mt.next);
    else
        signal(mt.mutex);
}
```
问题(2)中我们已经看到发出signal者会（A）进入sleep，在调度器的帮助下这相当于切换到了请求信号者（B），B执行完后`if`语句的前半部分`if(next_count>0) signal(mt.next)`将能唤醒A，而A结束后将执行`if`的后半部分`else signal(mt.mutex)`。这个流程正是Hoare方式的执行流程。

#### (3-2) 请在lab7-answer中实现另外一种类型的管程。
Hansen类型：
```
void     
monitor_init (monitor_t * mtp, size_t num_cv) {
    int i;
    assert(num_cv>0);
    mtp->next_count = 0;
    mtp->cv = NULL;
    sem_init(&(mtp->mutex), 1); //unlocked
    sem_init(&(mtp->next), 0);
    mtp->cv =(condvar_t *) kmalloc(sizeof(condvar_t)*num_cv);
    assert(mtp->cv!=NULL);
    for(i=0; i<num_cv; i++){
        mtp->cv[i].count=0;
        sem_init(&(mtp->cv[i].sem),0);
        mtp->cv[i].owner=mtp;
    }
}

// Unlock one of threads waiting on the condition variable. 
void 
cond_signal (condvar_t *cvp) {
   //LAB7 EXERCISE1: YOUR CODE
   cprintf("cond_signal begin: cvp %x, cvp->count %d, cvp->owner->next_count %d\n", cvp, cvp->count, cvp->owner->next_count);  
  /*
   *      cond_signal(cv) {
   *          if(cv.count>0) {
   *             mt.next_count ++;
   *             signal(cv.sem);
   *             wait(mt.next);
   *             mt.next_count--;
   *          }
   *       }
   */
     if(cvp->count>0) {
        up(&(cvp->sem));
      }
   cprintf("cond_signal end: cvp %x, cvp->count %d, cvp->owner->next_count %d\n", cvp, cvp->count, cvp->owner->next_count);
}

// Suspend calling thread on a condition variable waiting for condition Atomically unlocks 
// mutex and suspends calling thread on conditional variable after waking up locks mutex. Notice: mp is mutex semaphore for monitor's procedures
void
cond_wait (condvar_t *cvp) {
    //LAB7 EXERCISE1: YOUR CODE
    cprintf("cond_wait begin:  cvp %x, cvp->count %d, cvp->owner->next_count %d\n", cvp, cvp->count, cvp->owner->next_count);
   /*
    *         cv.count ++;
    *         if(mt.next_count>0)
    *            signal(mt.next)
    *         else
    *            signal(mt.mutex);
    *         wait(cv.sem);
    *         cv.count --;
    */
      cvp->count++;
      up(&(cvp->owner->mutex));
      down(&(cvp->sem));
      down(&cvp->owner->mutex);
      cvp->count --;
    cprintf("cond_wait end:  cvp %x, cvp->count %d, cvp->owner->next_count %d\n", cvp, cvp->count, cvp->owner->next_count);
}
```

#### (4) 实现[银行家算法](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/deadlock/bankers-homework.py)（[参考输出](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/deadlock/example-output.txt)，在`YOUR CODE`处填代码）。可尝试改进执行效率。

#### (5) 举一个银行家算法认为不安全但没有死锁的例子。
可抢占或者当某个进程需要大量资源时其他进程主动让出，就不会出现死锁。
