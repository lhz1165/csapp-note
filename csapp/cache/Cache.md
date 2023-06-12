# Cache

高速缓存(cache memories)
虽然它们是某种内置的自动硬件存储设备，如果你了解它们，尝试最大的时间和空间局部性，就可以利用你的知识并利用它们并使你的代码运行得更快，

## 直接映射(1个set,1个line)：

主存地址可以分为：tag+index+offset，分别表示标记，cache的行号，块内地址。

cache可以分为：valid+tag+block。

1. 主存地址首先根据index，取cache找第几块，

2. 然后对比tag，再使用offset去block找对应的数据（使用硬件去比较）。

## 组相联映射(1个set,n个line)：

主存地址可以分为：tag+index+offset，分别表示标记，cache的组号，块内地址。

cache可以分为：valid+tag+block，

1. 主存地址首先根据index，取cache找第几组，

2. 组内有E个块，然后每一个轮流对比tag（使用硬件去比较），

3. 找到块后，再使用offset去block找对应的数据。

## 对block写操作

### 写命中

*write-through*：立即写主存

*write-back*：标记脏块，等到替换回主存了再写主存(只操作cache)

### 写不命中（与上面对称的）

*write-allocate*: 加载到cache，更新(只操作cache)

*no-write-allocate*: 直接写回主存，不加载到cache