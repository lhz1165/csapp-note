# ThreadLocal



## ThreadLocalMap

每个Thread中包含一个ThreadLocalMap的空引用

只有在新建好一个**ThreadLocal**后，对他*set*()或者*get()*才开始初始化**ThreadLocalMap**,

一个线程多个threadLocal共享一个ThreadLocalMap

## 原理

### Map的hashcode如何生成

AtomicInteger作为map的key，线性的增长，是ThreadLocal的静态变量，第一次初始化之后一直调用他，之后每次set的时候慢慢增加作为Map的下标(threadLocalHashCode)

```java
//当前的hashcode即Map的key
private final int threadLocalHashCode = nextHashCode();

//计算hashcode的过程，每次线性增加
private static AtomicInteger nextHashCode =
    new AtomicInteger();

private static final int HASH_INCREMENT = 0x61c88647;


private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```

第一次创建ThreadLocal时候初始化Map记录下来

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
}
```

内存泄露问题

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

问题一

线程的ThreadLocal引用指向内存的对象ThreadLocal，

Entry的key引用也指向内存的对象ThreadLocal，

如果线程的ThreadLocal不使用了，但是entry还指向内存的对象ThreadLocal，导致回收不掉。

**所以Entry需要使用弱引用**

- 强引用：普通的引用，强引用指向的对象不会被回收；
- 软引用：仅有软引用指向的对象，只有发生gc且内存不足，才会被回收；
- 弱引用：仅有弱引用指向的对象，只要发生gc就会被回收。

问题二

要是问题一解决了，但是entry中还有value，导致entry回收不掉，

**所以需要手动remove();**

