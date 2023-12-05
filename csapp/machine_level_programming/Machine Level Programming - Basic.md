# Machine Level Programming - I Basic

## register

rax    r代表64

eax   e代表32位  

rdi和rsi一般保存地址（指针类型）

特殊寄存器%rsp,用于栈的指针，不能用来存放数据

## Moving Data

moveq [source],[dest]    (by the way  moveq的q代表quad)

source:立即数， 寄存器，内存

dest： 寄存器 内存

(r)-->mem[reg[r]]

d(r)--->mem[reg[r]+d]  

d(rb,ri,s) --->mem[reg[rb]+s*reg[ri]+d] rb代表基地址，ri代表索引，s代表索引长度，一般用于数组

例如leaq 4(%rdi,%rdx) ,%rcx ====>rdi+rdx+4存到rcx中

## 地址计算指令

leaq [src] [dest]   src一个地址，dest寄存器，根据地址的值计算，然后实际结果的地址写入的是该地址，而不是内存的值，即把指针写入寄存器



用来 

1. 相当与c语言中的&  

2. 计算x+k*y，k=1，2，4，8    例如  leal (%eax, %eax, 2) , % eax

**当用法第一种时**

adress : 0x1---> value: 0x88

adress:0x3---> value 0x77

那么 

 leal 2(%eax) , % eax 执行完之后， eax的值是0x3（指针）

movl 2(%eax) , % eax 执行完之后，eax的值是0x77（地址）

**当用法第二种时**

x*12   

 leaq(%rdi,%rdi,2),%rax

sqlq $2, %rax

先把%rdi的值=rdi+2rdi=x*3,结果的地址存入rax

然后再rax的值左移两位