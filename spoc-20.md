### 20. 两人公用一个账号，每次限存或取10元；

#### 信号量

代码如下:
```python
# -*- coding: utf-8 -*-

from __future__ import print_function

import threading
import time
import random


class BankAccount(object):
    def __init__(self):
        self.savings = 0
        self.semaphore = threading.Semaphore(0)
        self.lock = threading.Semaphore(1)

    def deposit(self, name, time):
        self.lock.acquire()
        self.savings += 10
        self.semaphore.release()
        print('[%s:%d] savings after depositing == %d' \
              % (name, time, self.savings))
        self.lock.release()

    def withdraw(self, name, time):
        self.semaphore.acquire()
        self.lock.acquire()
        self.savings -= 10
        print('[%s:%d] savings after withdrawing == %d' \
              % (name, time, self.savings))
        self.lock.release()


def worker(name, bank, times):
    for i in range(times):
        if name == 'Peter':
            bank.deposit(name, i)
        else:
            bank.withdraw(name, i)
        time.sleep(0.005 * random.randint(1, 20))


def main():
    bank = BankAccount()
    alice = threading.Thread(target=worker, args=('Alice', bank, 200))
    peter = threading.Thread(target=worker, args=('Peter', bank, 200))
    alice.start()
    peter.start()
    alice.join()
    peter.join()


if __name__ == '__main__':
    main()
```
某次输出结果如下（经过反复多次测试，没有出现死锁，结果也正确）：
```
[Peter:0] savings after depositing == 10
[Alice:0] savings after withdrawing == 0
[Peter:1] savings after depositing == 10
[Alice:1] savings after withdrawing == 0
[Peter:2] savings after depositing == 10
[Alice:2] savings after withdrawing == 0
[Peter:3] savings after depositing == 10
[Alice:3] savings after withdrawing == 0
[Peter:4] savings after depositing == 10
[Alice:4] savings after withdrawing == 0
[Peter:5] savings after depositing == 10
[Alice:5] savings after withdrawing == 0
[Peter:6] savings after depositing == 10
[Alice:6] savings after withdrawing == 0
[Peter:7] savings after depositing == 10
[Alice:7] savings after withdrawing == 0
[Peter:8] savings after depositing == 10
[Alice:8] savings after withdrawing == 0
[Peter:9] savings after depositing == 10
[Alice:9] savings after withdrawing == 0
[Peter:10] savings after depositing == 10
[Alice:10] savings after withdrawing == 0
[Peter:11] savings after depositing == 10
[Alice:11] savings after withdrawing == 0
[Peter:12] savings after depositing == 10
[Alice:12] savings after withdrawing == 0
[Peter:13] savings after depositing == 10
[Alice:13] savings after withdrawing == 0
[Peter:14] savings after depositing == 10
[Alice:14] savings after withdrawing == 0
[Peter:15] savings after depositing == 10
[Alice:15] savings after withdrawing == 0
[Peter:16] savings after depositing == 10
[Alice:16] savings after withdrawing == 0
[Peter:17] savings after depositing == 10
[Peter:18] savings after depositing == 20
[Alice:17] savings after withdrawing == 10
[Alice:18] savings after withdrawing == 0
[Peter:19] savings after depositing == 10
[Alice:19] savings after withdrawing == 0
[Peter:20] savings after depositing == 10
[Alice:20] savings after withdrawing == 0
[Peter:21] savings after depositing == 10
[Alice:21] savings after withdrawing == 0
[Peter:22] savings after depositing == 10
[Alice:22] savings after withdrawing == 0
[Peter:23] savings after depositing == 10
[Alice:23] savings after withdrawing == 0
[Peter:24] savings after depositing == 10
[Alice:24] savings after withdrawing == 0
[Peter:25] savings after depositing == 10
[Alice:25] savings after withdrawing == 0
[Peter:26] savings after depositing == 10
[Peter:27] savings after depositing == 20
[Alice:26] savings after withdrawing == 10
[Peter:28] savings after depositing == 20
[Peter:29] savings after depositing == 30
[Alice:27] savings after withdrawing == 20
[Peter:30] savings after depositing == 30
[Alice:28] savings after withdrawing == 20
[Peter:31] savings after depositing == 30
[Alice:29] savings after withdrawing == 20
[Alice:30] savings after withdrawing == 10
[Peter:32] savings after depositing == 20
[Peter:33] savings after depositing == 30
[Alice:31] savings after withdrawing == 20
[Alice:32] savings after withdrawing == 10
[Alice:33] savings after withdrawing == 0
[Peter:34] savings after depositing == 10
[Alice:34] savings after withdrawing == 0
[Peter:35] savings after depositing == 10
[Alice:35] savings after withdrawing == 0
[Peter:36] savings after depositing == 10
[Peter:37] savings after depositing == 20
[Alice:36] savings after withdrawing == 10
[Alice:37] savings after withdrawing == 0
[Peter:38] savings after depositing == 10
[Alice:38] savings after withdrawing == 0
[Peter:39] savings after depositing == 10
[Alice:39] savings after withdrawing == 0
[Peter:40] savings after depositing == 10
[Alice:40] savings after withdrawing == 0
[Peter:41] savings after depositing == 10
[Alice:41] savings after withdrawing == 0
[Peter:42] savings after depositing == 10
[Alice:42] savings after withdrawing == 0
[Peter:43] savings after depositing == 10
[Alice:43] savings after withdrawing == 0
[Peter:44] savings after depositing == 10
[Alice:44] savings after withdrawing == 0
[Peter:45] savings after depositing == 10
[Alice:45] savings after withdrawing == 0
[Peter:46] savings after depositing == 10
[Alice:46] savings after withdrawing == 0
[Peter:47] savings after depositing == 10
[Alice:47] savings after withdrawing == 0
[Peter:48] savings after depositing == 10
[Alice:48] savings after withdrawing == 0
[Peter:49] savings after depositing == 10
[Alice:49] savings after withdrawing == 0
[Peter:50] savings after depositing == 10
[Peter:51] savings after depositing == 20
[Alice:50] savings after withdrawing == 10
[Alice:51] savings after withdrawing == 0
[Peter:52] savings after depositing == 10
[Alice:52] savings after withdrawing == 0
[Peter:53] savings after depositing == 10
[Peter:54] savings after depositing == 20
[Peter:55] savings after depositing == 30
[Alice:53] savings after withdrawing == 20
[Alice:54] savings after withdrawing == 10
[Peter:56] savings after depositing == 20
[Alice:55] savings after withdrawing == 10
[Peter:57] savings after depositing == 20
[Peter:58] savings after depositing == 30
[Alice:56] savings after withdrawing == 20
[Peter:59] savings after depositing == 30
[Alice:57] savings after withdrawing == 20
[Peter:60] savings after depositing == 30
[Alice:58] savings after withdrawing == 20
[Peter:61] savings after depositing == 30
[Peter:62] savings after depositing == 40
[Peter:63] savings after depositing == 50
[Alice:59] savings after withdrawing == 40
[Alice:60] savings after withdrawing == 30
[Peter:64] savings after depositing == 40
[Peter:65] savings after depositing == 50
[Alice:61] savings after withdrawing == 40
[Peter:66] savings after depositing == 50
[Alice:62] savings after withdrawing == 40
[Peter:67] savings after depositing == 50
[Alice:63] savings after withdrawing == 40
[Peter:68] savings after depositing == 50
[Peter:69] savings after depositing == 60
[Alice:64] savings after withdrawing == 50
[Alice:65] savings after withdrawing == 40
[Peter:70] savings after depositing == 50
[Alice:66] savings after withdrawing == 40
[Peter:71] savings after depositing == 50
[Alice:67] savings after withdrawing == 40
[Peter:72] savings after depositing == 50
[Peter:73] savings after depositing == 60
[Alice:68] savings after withdrawing == 50
[Peter:74] savings after depositing == 60
[Peter:75] savings after depositing == 70
[Alice:69] savings after withdrawing == 60
[Alice:70] savings after withdrawing == 50
[Peter:76] savings after depositing == 60
[Peter:77] savings after depositing == 70
[Alice:71] savings after withdrawing == 60
[Alice:72] savings after withdrawing == 50
[Peter:78] savings after depositing == 60
[Alice:73] savings after withdrawing == 50
[Peter:79] savings after depositing == 60
[Alice:74] savings after withdrawing == 50
[Peter:80] savings after depositing == 60
[Peter:81] savings after depositing == 70
[Peter:82] savings after depositing == 80
[Peter:83] savings after depositing == 90
[Alice:75] savings after withdrawing == 80
[Peter:84] savings after depositing == 90
[Alice:76] savings after withdrawing == 80
[Peter:85] savings after depositing == 90
[Alice:77] savings after withdrawing == 80
[Peter:86] savings after depositing == 90
[Peter:87] savings after depositing == 100
[Alice:78] savings after withdrawing == 90
[Peter:88] savings after depositing == 100
[Alice:79] savings after withdrawing == 90
[Peter:89] savings after depositing == 100
[Alice:80] savings after withdrawing == 90
[Peter:90] savings after depositing == 100
[Alice:81] savings after withdrawing == 90
[Alice:82] savings after withdrawing == 80
[Peter:91] savings after depositing == 90
[Alice:83] savings after withdrawing == 80
[Peter:92] savings after depositing == 90
[Peter:93] savings after depositing == 100
[Peter:94] savings after depositing == 110
[Alice:84] savings after withdrawing == 100
[Peter:95] savings after depositing == 110
[Alice:85] savings after withdrawing == 100
[Alice:86] savings after withdrawing == 90
[Peter:96] savings after depositing == 100
[Peter:97] savings after depositing == 110
[Alice:87] savings after withdrawing == 100
[Peter:98] savings after depositing == 110
[Alice:88] savings after withdrawing == 100
[Alice:89] savings after withdrawing == 90
[Peter:99] savings after depositing == 100
[Alice:90] savings after withdrawing == 90
[Peter:100] savings after depositing == 100
[Alice:91] savings after withdrawing == 90
[Peter:101] savings after depositing == 100
[Peter:102] savings after depositing == 110
[Alice:92] savings after withdrawing == 100
[Peter:103] savings after depositing == 110
[Alice:93] savings after withdrawing == 100
[Peter:104] savings after depositing == 110
[Alice:94] savings after withdrawing == 100
[Alice:95] savings after withdrawing == 90
[Alice:96] savings after withdrawing == 80
[Peter:105] savings after depositing == 90
[Alice:97] savings after withdrawing == 80
[Alice:98] savings after withdrawing == 70
[Peter:106] savings after depositing == 80
[Peter:107] savings after depositing == 90
[Peter:108] savings after depositing == 100
[Alice:99] savings after withdrawing == 90
[Alice:100] savings after withdrawing == 80
[Peter:109] savings after depositing == 90
[Peter:110] savings after depositing == 100
[Alice:101] savings after withdrawing == 90
[Peter:111] savings after depositing == 100
[Alice:102] savings after withdrawing == 90
[Alice:103] savings after withdrawing == 80
[Peter:112] savings after depositing == 90
[Peter:113] savings after depositing == 100
[Alice:104] savings after withdrawing == 90
[Alice:105] savings after withdrawing == 80
[Alice:106] savings after withdrawing == 70
[Peter:114] savings after depositing == 80
[Peter:115] savings after depositing == 90
[Alice:107] savings after withdrawing == 80
[Alice:108] savings after withdrawing == 70
[Peter:116] savings after depositing == 80
[Peter:117] savings after depositing == 90
[Peter:118] savings after depositing == 100
[Alice:109] savings after withdrawing == 90
[Peter:119] savings after depositing == 100
[Alice:110] savings after withdrawing == 90
[Peter:120] savings after depositing == 100
[Alice:111] savings after withdrawing == 90
[Peter:121] savings after depositing == 100
[Alice:112] savings after withdrawing == 90
[Alice:113] savings after withdrawing == 80
[Peter:122] savings after depositing == 90
[Alice:114] savings after withdrawing == 80
[Peter:123] savings after depositing == 90
[Alice:115] savings after withdrawing == 80
[Alice:116] savings after withdrawing == 70
[Alice:117] savings after withdrawing == 60
[Peter:124] savings after depositing == 70
[Alice:118] savings after withdrawing == 60
[Peter:125] savings after depositing == 70
[Alice:119] savings after withdrawing == 60
[Alice:120] savings after withdrawing == 50
[Peter:126] savings after depositing == 60
[Alice:121] savings after withdrawing == 50
[Alice:122] savings after withdrawing == 40
[Peter:127] savings after depositing == 50
[Alice:123] savings after withdrawing == 40
[Peter:128] savings after depositing == 50
[Alice:124] savings after withdrawing == 40
[Peter:129] savings after depositing == 50
[Peter:130] savings after depositing == 60
[Alice:125] savings after withdrawing == 50
[Alice:126] savings after withdrawing == 40
[Peter:131] savings after depositing == 50
[Peter:132] savings after depositing == 60
[Alice:127] savings after withdrawing == 50
[Alice:128] savings after withdrawing == 40
[Peter:133] savings after depositing == 50
[Peter:134] savings after depositing == 60
[Alice:129] savings after withdrawing == 50
[Alice:130] savings after withdrawing == 40
[Peter:135] savings after depositing == 50
[Alice:131] savings after withdrawing == 40
[Peter:136] savings after depositing == 50
[Alice:132] savings after withdrawing == 40
[Alice:133] savings after withdrawing == 30
[Peter:137] savings after depositing == 40
[Alice:134] savings after withdrawing == 30
[Alice:135] savings after withdrawing == 20
[Peter:138] savings after depositing == 30
[Peter:139] savings after depositing == 40
[Peter:140] savings after depositing == 50
[Peter:141] savings after depositing == 60
[Alice:136] savings after withdrawing == 50
[Alice:137] savings after withdrawing == 40
[Peter:142] savings after depositing == 50
[Alice:138] savings after withdrawing == 40
[Peter:143] savings after depositing == 50
[Alice:139] savings after withdrawing == 40
[Alice:140] savings after withdrawing == 30
[Peter:144] savings after depositing == 40
[Alice:141] savings after withdrawing == 30
[Alice:142] savings after withdrawing == 20
[Peter:145] savings after depositing == 30
[Peter:146] savings after depositing == 40
[Alice:143] savings after withdrawing == 30
[Peter:147] savings after depositing == 40
[Alice:144] savings after withdrawing == 30
[Peter:148] savings after depositing == 40
[Alice:145] savings after withdrawing == 30
[Peter:149] savings after depositing == 40
[Alice:146] savings after withdrawing == 30
[Peter:150] savings after depositing == 40
[Alice:147] savings after withdrawing == 30
[Alice:148] savings after withdrawing == 20
[Alice:149] savings after withdrawing == 10
[Peter:151] savings after depositing == 20
[Alice:150] savings after withdrawing == 10
[Peter:152] savings after depositing == 20
[Peter:153] savings after depositing == 30
[Alice:151] savings after withdrawing == 20
[Alice:152] savings after withdrawing == 10
[Alice:153] savings after withdrawing == 0
[Peter:154] savings after depositing == 10
[Alice:154] savings after withdrawing == 0
[Peter:155] savings after depositing == 10
[Alice:155] savings after withdrawing == 0
[Peter:156] savings after depositing == 10
[Alice:156] savings after withdrawing == 0
[Peter:157] savings after depositing == 10
[Alice:157] savings after withdrawing == 0
[Peter:158] savings after depositing == 10
[Alice:158] savings after withdrawing == 0
[Peter:159] savings after depositing == 10
[Peter:160] savings after depositing == 20
[Peter:161] savings after depositing == 30
[Alice:159] savings after withdrawing == 20
[Peter:162] savings after depositing == 30
[Alice:160] savings after withdrawing == 20
[Peter:163] savings after depositing == 30
[Alice:161] savings after withdrawing == 20
[Peter:164] savings after depositing == 30
[Peter:165] savings after depositing == 40
[Alice:162] savings after withdrawing == 30
[Alice:163] savings after withdrawing == 20
[Peter:166] savings after depositing == 30
[Alice:164] savings after withdrawing == 20
[Peter:167] savings after depositing == 30
[Alice:165] savings after withdrawing == 20
[Peter:168] savings after depositing == 30
[Alice:166] savings after withdrawing == 20
[Peter:169] savings after depositing == 30
[Alice:167] savings after withdrawing == 20
[Peter:170] savings after depositing == 30
[Peter:171] savings after depositing == 40
[Peter:172] savings after depositing == 50
[Alice:168] savings after withdrawing == 40
[Peter:173] savings after depositing == 50
[Alice:169] savings after withdrawing == 40
[Peter:174] savings after depositing == 50
[Alice:170] savings after withdrawing == 40
[Alice:171] savings after withdrawing == 30
[Peter:175] savings after depositing == 40
[Peter:176] savings after depositing == 50
[Alice:172] savings after withdrawing == 40
[Alice:173] savings after withdrawing == 30
[Peter:177] savings after depositing == 40
[Alice:174] savings after withdrawing == 30
[Peter:178] savings after depositing == 40
[Alice:175] savings after withdrawing == 30
[Alice:176] savings after withdrawing == 20
[Peter:179] savings after depositing == 30
[Alice:177] savings after withdrawing == 20
[Peter:180] savings after depositing == 30
[Alice:178] savings after withdrawing == 20
[Peter:181] savings after depositing == 30
[Alice:179] savings after withdrawing == 20
[Peter:182] savings after depositing == 30
[Peter:183] savings after depositing == 40
[Alice:180] savings after withdrawing == 30
[Peter:184] savings after depositing == 40
[Alice:181] savings after withdrawing == 30
[Peter:185] savings after depositing == 40
[Alice:182] savings after withdrawing == 30
[Peter:186] savings after depositing == 40
[Alice:183] savings after withdrawing == 30
[Peter:187] savings after depositing == 40
[Peter:188] savings after depositing == 50
[Alice:184] savings after withdrawing == 40
[Peter:189] savings after depositing == 50
[Peter:190] savings after depositing == 60
[Alice:185] savings after withdrawing == 50
[Alice:186] savings after withdrawing == 40
[Peter:191] savings after depositing == 50
[Peter:192] savings after depositing == 60
[Peter:193] savings after depositing == 70
[Alice:187] savings after withdrawing == 60
[Alice:188] savings after withdrawing == 50
[Alice:189] savings after withdrawing == 40
[Peter:194] savings after depositing == 50
[Alice:190] savings after withdrawing == 40
[Peter:195] savings after depositing == 50
[Peter:196] savings after depositing == 60
[Alice:191] savings after withdrawing == 50
[Alice:192] savings after withdrawing == 40
[Peter:197] savings after depositing == 50
[Alice:193] savings after withdrawing == 40
[Alice:194] savings after withdrawing == 30
[Peter:198] savings after depositing == 40
[Alice:195] savings after withdrawing == 30
[Peter:199] savings after depositing == 40
[Alice:196] savings after withdrawing == 30
[Alice:197] savings after withdrawing == 20
[Alice:198] savings after withdrawing == 10
[Alice:199] savings after withdrawing == 0
```
