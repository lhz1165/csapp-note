# 重定位（Relocation）

## **概念**

linker完成了符号解析，一个符号引用对应一个符号定义，linker知道代码的和数据的大小，接下来需要重定位（Relocation），为输入模块的每个symbol分配运行时的地址。

## 步骤

1. 重定位section和symbol definitions：把所有文件的相同section合并再一起，并为新的section分配地址。
2. 在section中重定位symbol references：让symbol references指向正确的上一步分配的symbol definitions的地址（依赖Relocation Entries）

## Relocation Entries

当生成目标文件时，编译器不知道code和data 的地址，也不知道function，全局变量的地址，所以通过生成Relocation Entries去告知编译器在生成可执行文件时如何修改reference。

.rel.text和.rel.data代表代码和数据的section。

