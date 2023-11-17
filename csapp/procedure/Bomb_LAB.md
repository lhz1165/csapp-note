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

首先学习一下sscanf用法，

```c
int a;
int r =sscanf("10","%d",&a)
//r=1 a=10
    
int a1;
int a2;
int r =sscanf("10 20","%d %d",&a1,&a2)
//r=2 a1=10 a2=20
```

根据csapp寄存器参数使用顺序可以分析出，sscanf接收参数(rdi,rsi,&rcx,&rdx)，

因此输入的两个参数分别在rsp+12和rsp+8里面，并且返回rax=2

![image-20231117094150799](assets/Bomb_LAB/image-20231117094150799.png)

实际上可以看到执行完 sscanf返回rax=2

![image-20231117100507588](assets/Bomb_LAB/image-20231117100507588.png)



400f60 cmp    $0x1,%eax    因为rax=2>1 所以jump 400f6a

400f6a cmpl $0x7,0x8(%rsp)， 

400f6f 如果(rsp+8)>7那么 jump 400fad,explode_bomb，所以(rsp+8)必须<=7,

400f71 rax=(rsp+8)

400f75  jump \*(8 \* rax+0x402470)地址所指向的地址  （一定要在400f7c到400fb2之间，才能往下继续）然后给rax赋值

指针地址有 0x402470  0x402478 0x402480 0x402488 0x402490   0x402498  0x4024A0  0x4024A8

所以依次查看这些地址的值

x打印指针的值 8gx（**后显示8个地址，g表示八字节，按十六进制格式显示变量**）

![image-20231117102856482](assets/Bomb_LAB/image-20231117102856482.png)

根据switch(rax) case一下8种

![image-20231117103528287](assets/Bomb_LAB/image-20231117103528287.png)

最终要去 

400fbe  cmp    0xc(%rsp),%eax，如果(rsp+12)！= rax 那么 explode_bomb

所以（rsp+8）和（rsp+12）之前存在着联动

**具体步骤**

当rax=(rsp+8)=0  jump 0x0000000000400f7c, rax=0xcf = 207   =>(rsp+8)=0  (rsp+12)=207   

当rax=(rsp+8)=1  jump 0x0000000000400fb9, rax=0x137=311=>(rsp+8)=1  (rsp+12)=311

当rax=(rsp+8)=2  jump 0x0000000000400f83, rax=0x2c3=707  =>(rsp+8)=2  (rsp+12)=707

当rax=(rsp+8)=3  jump 0x0000000000400f8a, rax=0x100=256  =>(rsp+8)=3  (rsp+12)=256  

当rax=(rsp+8)=4  jump 0x0000000000400f91, rax=0x185=389  =>(rsp+8)=4  (rsp+12)=389  

当rax=(rsp+8)=5  jump 0x0000000000400f98, rax=0xce=206  =>(rsp+8)=5  (rsp+12)=206  

当rax=(rsp+8)=6  jump 0x0000000000400f9f, rax=0x2aa=682  =>(rsp+8)=6  (rsp+12)=682  

当rax=(rsp+8)=7  jump 0x0000000000400fa6, rax=0x147=327  =>(rsp+8)=7  (rsp+12)=327  

又因为参数越靠后栈的位置越大（越靠近栈底部）  所以 (rsp+8)为第1个 ，(rsp+12)为第二个

**输入以上组合都可以，解除第三个炸弹**

![image-20231117104608081](assets/Bomb_LAB/image-20231117104608081.png)



![image-20231117104642978](assets/Bomb_LAB/image-20231117104642978.png)

## phase_4
