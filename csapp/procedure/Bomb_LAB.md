## Bomb

phase_1

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

40135c  &rbx赋值给rax

40135f 测试rbx是否为0

