## ReentrantLock

有NonfairSync和FairSync两种默认非公平

## lock()



```java
//非公平直接尝试cas，
//公平锁没有这一步，直接acquire(1)执行AQS逻辑
final void lock() {
    //非公平锁才有CAS
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        //aqs方法
        acquire(1);
}
```

在公平加锁过程中，非公平先cas一次,第二次cas加锁是在acquire中的tryAcqure，如果失败，构造节点，加入队列后如果是刚好连着头节点，再tryAcqure一次

## tryLock()

两者都一样

```java
public boolean tryLock() {
        return sync.nonfairTryAcquire(1);
}

final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    //无锁
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    //有锁 是否重入
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

## tryAcqure()

因为lock调用AQS的acquire(1)，而AQS的acquire(1)需要实现tryAcquire(1)。查看上面lock（）

公平和非公平有所区别

```java
//非公平锁的tryAcquire直接调用此方法
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
	//公平锁的tryAcquire
    protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                //同步队列为空或者当前线程是头节点返回false
                //现在在同步队列里为true
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

对比发现公平锁多了一步hasQueuedPredecessors()处理

如果现在有线程在排队，应该返回false，不能抢锁，执行没有抢到锁的逻辑，加入AQS队列