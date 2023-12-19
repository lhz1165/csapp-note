# Attack_LAB

![image-20231219160940326](assets/Attack_LAB/image-20231219160940326.png)

应该从文件里读取十六进制的结果

## phase_1

gdb调试ctarget，使用-q 不连接服务器

![image-20231219161220379](assets/Attack_LAB/image-20231219161220379.png)

栈帧结构

![image-20231219161110542](assets/Attack_LAB/image-20231219161110542.png)

进入getbuf

![image-20231219161053446](assets/Attack_LAB/image-20231219161053446.png)

走到gets方法 输入0123456789查看栈的样子

检查现在栈的范围

![image-20231219162948980](assets/Attack_LAB/image-20231219162948980.png)

如下

![image-20231219163131602](assets/Attack_LAB/image-20231219163131602.png)

检查从rsp到 ret 具体的内容

![image-20231219163253380](assets/Attack_LAB/image-20231219163253380.png)

0x5561dc78之后是我们输入的0123456789的每个字符对应的ascii码，0x5561dca0所存放的值显而易见是返回的地址

![image-20231219164440479](assets/Attack_LAB/image-20231219164440479.png)



![image-20231219165915329](assets/Attack_LAB/image-20231219165915329.png)

![image-20231219170022963](assets/Attack_LAB/image-20231219170022963.png)

所以我们要输入43的字符，40个填充buff缓冲区，剩下3个40 17 c0填充ret指向touch1的地址

由于提示读取文件所以要得到0x4017c0,文本中应该是 

c0 17 40

![image-20231219170202518](assets/Attack_LAB/image-20231219170202518.png)

所以答案,保存在ctarget1.txt中

```
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
c0 17 40 00 00 00 00 00
```

![image-20231219170412388](assets/Attack_LAB/image-20231219170412388.png)

