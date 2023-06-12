# AQS

同步状态

```java
private volatile int state;

//1
protected final int getState() {
    return state;
}
//2
protected final void setState(int newState) {
    state = newState;
}
//3
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

Node：

**头节点的线程对象为空**，头节点就是当前持有锁的线程

```java
//Node的职责 
//【准备唤醒下一个SIGNAL  准备退出CANCELLED  条件队列CONDITION  PROPAGATE共享头节点】
volatile int waitStatus;
```

## AQS 加锁acquire 核心方法方法

获取锁失败，1.构造节点放入队列，2.然后要么自旋，要么阻塞

```java
public final void acquire(int arg) {
    //tryAcquire由用户自己定义
    //例如在reentrantLock中tryAcquire，
    //State=0 ，cas方式获取锁成功或者; state>0 ,重入成功才返回true
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```



**1 addWaiter**：把线程构建为节点，加入同步队列

核心方法

```java
 private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```



同步队列没有线程：自旋生成一个空头节点

同步队列有线程：设置为链表尾节点

**2.acquireQueued(addWaiter(Node.EXCLUSIVE), arg)**   下一个可以获取到锁的node节点自旋查看是否能获取到锁，如果获取不到那么调用parkAndCheckInterrupt()阻塞

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            //获取锁之前必须前驱是头节点
            if (p == head && tryAcquire(arg)) {
                //获取锁成功，把他设置为头节点
                setHead(node);
                //之前的头节点出去
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //获取锁失败的情况
            //然后阻塞获取锁失败的线程
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

Lock如何实现不可响应中断

park()会响应中断，会使阻塞线程唤醒，不会抛出异常，唤醒就是重新acquireQueued（）尝试是否能获取到锁

## AQS解锁

解锁成功，唤醒后继节点(unparkSuccessor),

```java
public final boolean release(int arg) {
    //tryRelease由用户自己定义
    //例如在reentrantLock中tryRelease为true代表线程所有重入锁解锁成功，而重入多次，只解锁一次
    //还是返回false
    if (tryRelease(arg)) {
        Node h = head;
        //只有后面没有节点，刚刚有节点加入，才唤醒(通过waitStatus来判断)
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

