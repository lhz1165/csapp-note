# Machine Level Programming - II-Control

## 条件码

CF carry flag  进位

 ZF  zero flag 0标志

SF  sing flag 负数

OF  补码溢出

用途

cmpq src2 src1，根据两个数之差来设置条件码

## 条件分支

第一种使用有条件jump，判断然后跳转到指定代码行



第二种使用conditional move，两个结果计算出来，然后选择我们需要的

(由于流水线技术，if中的判断，if的结果，else的结果都可以计算出来，性能更好)

