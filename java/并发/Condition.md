# Condition

## await

把线程节点添加到等待队列，并阻塞，直到singal后进入同步队列获取到锁才执行完。

```java
public final void await() throws InterruptedException {
	
    //添加到等待队列
    Node node = addConditionWaiter();
    //保存当前同步状态state;
    //释放锁的同步状态，之后进入同步队列后又要获取这些锁
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    //是否进入了同步队列 阻塞当前节点的线程，等到singal的通知
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    //进入了同步队列后尝试获取锁 卡在这里自旋等待获取到锁
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
}
```



## signal

把最长时间等待的节点，即最靠近头节点的node唤醒，加入同步队列

```java
public final void signal() {
    Node first = firstWaiter;
    if (first != null)
        //把条件队列的节点 插入同步队列
        doSignal(first);
}

```