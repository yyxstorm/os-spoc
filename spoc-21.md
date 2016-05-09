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
不能<0，可能是0或1，不能>1。

#### (3) 目前的lab7-answer中管程的实现是Hansen管程类型还是Hoare管程类型？
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
问题(2)中我们已经看到发出signal者会进入sleep，在调度器的帮助下这相当于切换到了请求信号者，....

#### (4) 请在lab7-answer中实现另外一种类型的管程。
```
 * IMPLEMENTATION:
 *   monitor mt {
 *     ----------------variable------------------
 *     semaphore mutex;
 *     semaphore next;
 *     int next_count;
 *     condvar {int count, sempahore sem}  cv[N];
 *     other variables in mt;
 *     --------condvar wait/signal---------------
 *     cond_wait (cv) {
 *         cv.count ++;
 *         if(mt.next_count>0)
 *            signal(mt.next)
 *         else
 *            signal(mt.mutex);
 *         wait(cv.sem);
 *         cv.count --;
 *      }
 *
 *      cond_signal(cv) {
 *          if(cv.count>0) {
 *             mt.next_count ++;
 *             signal(cv.sem);
 *             wait(mt.next);
 *             mt.next_count--;
 *          }
 *       }
 *     --------routines in monitor---------------
 *     routineA_in_mt () {
 *        wait(mt.mutex);
 *        ...
 *        real body of routineA
 *        ...
 *        if(next_count>0)
 *            signal(mt.next);
 *        else
 *            signal(mt.mutex);
 *     }
 */
```
