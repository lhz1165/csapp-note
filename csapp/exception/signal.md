# Exception-signal

## Reaping Child Processes

### 为什么要Reaping Child Processes：

although Child has already terminated, the kernel maintains some of its state until it can be reaped by the parent

### 1. 如果父进程终止了但是没有reap子进程，那么由init进程来reap



### 2. 如果父进程长时间不终止，通过waitpid方法去reap子进程

```c
pid_t waitpid(pid_t pid, int *statusp, int options)
    
    
//使用循环 等待多子进程退出
while ((pid = waitpid(-1, &status, 0)) > 0) {
        if (WIFEXITED(status)) {
            printf("Child process %d exited with status %d\n", pid, WEXITSTATUS(status));
        } else {
            printf("Child process %d terminated abnormally\n", pid);
        }
    }
```

参数pid>0,wait set只保存参数pid的子进程，pid=-1保存所有子进程，

waitpid 会暂停调用的父进程，直到其[wait set]中的子进程终止( 类似于java的CyclicBarrier的await() )，如子进程terminated了，那么waitpid 会马上返回。

简化版本

```c
pid_t wait(int *statusp);
```





