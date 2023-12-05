# Machine Level Programming - III - Procedures

## **Procedures**

1. 传递控制（方法调用
2. 传递值  (传参数
3. 内存管理 （分配内存，回收内存

## Stack

用栈来管理procedure的过程，stack越下面地址越大，越往上越小

POP : read  by %rsp， %rsp++,  value to dest

PUSH: fetch  from src,  %rsp--, value to %rsp所指的地方

## 1. 方法调用过程

%rsp指向当前栈里面某处地址，%rip指向程序执行的地址

当发生调用做两件事情，

	1. %rsp--，栈里保存的是方法调用完成后的下一条指令的地址（等方法返回继续从这里执行），
	1. %rip(pc)指向被调用代码的位置

## 2.参数传递