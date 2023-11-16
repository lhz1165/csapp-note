## Bomb

## phase_1

![image-20231115195822964](assets/Bomb_LAB/image-20231115195822964.png)

callq string_not_equla后，

test %eax %eax 就可以跳转到 400ef7 成功返回 ，

否则如果eax不是0触发 callq explpde_bomb直接炸

所以在 string_not_equal 寻找如何让eax变成0

![image-20231115201645067](assets/Bomb_LAB/image-20231115201645067.png)

40113c 40113f 接收两个参数 分别放到rbx和rbp，

401342 call string_length算长度 结果放在rax，

401347 然后通过第一个参数长度 从rax放到r12d

40134a 把rbp的参数放到rdi 

40134d 调用call string_length算长度 第二个参数长度结果放在rax

401352 edx默认1

401357 比较rax和r12d，如果不相等，直接跳到40139b，把edx的1赋值给rax，返回phase_1，直接炸



如果长度相等

根据401372 401376每次rbx和rpx加一

可以判断字符串一个一个循环比较，相等比较下一个位置，直到全部相等。

所以，这个方法是判断两个字符串是否完全相等，关键是查看参数具体值，需要gdb来打印内存的值

![image-20231116101958081](assets/Bomb_LAB/image-20231116101958081.png)

可以看到rdi是我自己输入的，rsi就是密码，因此复制粘贴 输入 

```
Border relations with Canada have never been better.
```

通过第一个炸弹

## phase_2

![image-20231116102210507](assets/Bomb_LAB/image-20231116102210507.png)



根据sub 0x28 %rsp，rsp=rsp-40 可知即将有局部变量存入栈中（**设当前rsp地址=r**）

400f05读取6个数字，

400f0a  判断rsp指针所指的值是否等于1 jump，否则explpde_bomb(**第一个数(rsp)=1**)



400f30 **rbx=rsp+4（r+4）** 栈指针向后 (**第二个数**)

400f35 rbp =rsp+24 （r+24）栈指针向后 (**第六个数**)

jump

400f17 rax=(rbx-4)=(r)=1

400f1a rax = rax+rax=2

400f1c 比较rax和(rbx)的的值 

可以知道 (rbx)=2(**第二个数(rsp+4)=2**)



400f25 **rbx=rbx+4=（r+8**） 继续向后(**第三个数**)

400f29 rbx和rbp不是一个地址，即是所有数字判断完了， jump

400f17 rax=(rbx-4)=（r+4）=2

400f1a rax = rax+rax=4

400f1c 比较rax和(rbx)的的值 

可以知道 (rbx)=4(**第三个数(rsp+8)=4**)

....

**可以得出规律，后一个数就是前一个数的2倍**

所以6个数字为以下，解除第二个炸弹

```
1 2 4 8 16 32
```

![image-20231116172028405](assets/Bomb_LAB/image-20231116172028405.png)

## phase_3

![image-20231116174332076](assets/Bomb_LAB/image-20231116174332076.png)
