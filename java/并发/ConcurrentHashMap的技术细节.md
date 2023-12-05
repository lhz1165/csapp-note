# ConcurrentHashMap的技术细节

R-R 可以

R-W 链表可以，树不行

W-W 不行

1. hash 重写（spread）

2. 五种节点类型

   **Node**： 普通链表节点 hash

   **TreeBin** :保存root，有读写锁，lockstate方便以cas加锁,1(writer),2(waiter),3(reader) 3种状态,控制一棵树的读写

   **TreeNode**:红黑树节点

   **ForwardingNode**:扩容时，对每个桶进行转移，此时把桶加sychronized锁，并把元素复制到另一个map，当前桶转移完成后，把原来的Node变成ForwardingNode，并有指针指向新的桶。

   目的是当别的桶还没转移，想要读当前转移过的ForwardingNode桶，通过ForwardingNode指向新的转移好的map去找数据，而读原来没转移的还是在老map上读

3. cas & sychronized

   CAS: 用于get红黑树使用乐观锁；put时候如果是null则cas 放；扩容时通过乐观锁来防止多个线程写

   sychronized: 有元素put对桶加锁；转移的时候对桶加锁；桶从链表变成红黑树加锁；红黑树退化成链表

4. 一次性扩容

​		扩容后搬家到新数组，调用sychronized；由于每个节点搬家位置固定所以可以多线程来操作，锁住每个桶来搬家