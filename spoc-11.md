## SPOC小组思考题

(1) (spoc)设计一个简化的进程管理子系统，可以管理并调度如下简化进程.给出了[参考代码](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab4/process-concept-homework.py)，请理解代码，并完成＂YOUR CODE"部分的内容．　可２个人一组

```python
def move_to_ready(self, expected, pid=-1):
    # YOUR CODE
    if pid == -1:
        pid = self.curr_proc
    if self.proc_info[self.curr_proc][PROC_STATE] == expected:
        self.proc_info[pid][PROC_STATE] = STATE_READY
```

```python
def move_to_running(self, expected):
    # YOUR CODE
    if self.proc_info[self.curr_proc][PROC_STATE] == expected:
        self.proc_info[self.curr_proc][PROC_STATE] = STATE_RUNNING
```

```python
def move_to_done(self, expected):
    # YOUR CODE
    if self.proc_info[self.curr_proc][PROC_STATE] == expected:
        self.proc_info[self.curr_proc][PROC_STATE] = STATE_DONE
```

```python
def next_proc(self, pid=-1):
    # YOUR CODE
    if pid == -1:
        pid = self.curr_proc
    n = self.get_num_processes()
    for i in range(n):
        j = (pid + i + 1) % n
        if self.proc_info[j][PROC_STATE] == STATE_READY:
            self.curr_proc = j
            self.move_to_running(STATE_READY)
            return
```

```python
# if current proc is RUNNING and has an instruction, execute it
# statistics clock_tick
instruction_to_execute = ''
if self.proc_info[self.curr_proc][PROC_STATE] == STATE_RUNNING and \
                len(self.proc_info[self.curr_proc][PROC_CODE]) > 0:
    # YOUR CODE
    instruction_to_execute = \
        self.proc_info[self.curr_proc][PROC_CODE].pop(0)
```

```python
# if this is an YIELD instruction, switch to ready state
# and add an io completion in the future
if instruction_to_execute == DO_YIELD:
    # YOUR CODE
    self.move_to_ready(STATE_RUNNING)
    self.next_proc()
```

