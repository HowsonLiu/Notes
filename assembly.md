# 汇编语言
## 著名CPU
- **8086**
    8086是Intel公司设计的16位微处理器芯片，是x86架构的鼻祖。
- **80386**
    也称为**i386**。是Intel公司设计的32位微处理器芯片。
---
## 寄存器分类（8086/i386）
- 通用寄存器
    - 四个数据寄存器
        - AX/EAX 累加器。加法乘法指令的缺省寄存器
        - BX/EBX 基地址寄存器。作为存储器指针使用
        - CX/ECX 计数器。在循环和字符串操作时控制次数；在位操作时CL指明移位的位数
        - DX/EDX 数据寄存器。乘除运算中常作为默认参与数参与运算；存放I/O端口地址
    - 两个变址寄存器
        - SI/ESI 源变址寄存器
        - DI/EDI 目的变址寄存器
    - 两个指针寄存器
        - SP/ESP 堆栈指针寄存器 栈顶指针
        - BP/EBP 基指针寄存器 栈底指针
- 控制寄存器
    - IP/EIP 指令指针寄存器 存放下一个CPU指令的内存地址
    - EFL/FLAG 标志寄存器 存放前一个指令执行结果的信息
- 四个段寄存器
    - CS 代码段寄存器
    - DS 数据段寄存器
    - SS 堆栈段寄存器
    - ES 附加段寄存器

## 通用寄存器
这些寄存器除了在特殊情况下有专用的作用外，其他时候都可以用来传送和暂存数据，因此叫通用寄存器
### 数据寄存器(Data Registers)
![](https://www.tutorialspoint.com/assembly_programming/images/register1.jpg)
> E表示extend，L表示Low，H表示High，注意只有数据寄存器才能分HL
#### AX(Accumulator Registers)的特殊作用
- DIV除法指令
    - 当除数为8位时，那么被除数为16位（[为啥](https://www.cnblogs.com/mrblug/p/5721196.html)），保存在AX中，结果中AL存储商，AH存储余数。
    - 当除数为16位时，那么被除数为32位，AX存放被除数的低16位，DX存放被除数的高16位，结果中AX存储商，DX存储余数。
- MUL乘法指令
    - 两个乘数都是8位，那么AL会默认存放一个（另一个存在其他寄存器或者内存中），结果存放在AX
    - 两个乘数都是16位，那么AX会默认存放一个，结果的高位存在DX，低位存在AX。
#### BX(Base Register)的特殊作用
在寻址的情况下，一般用来存放偏移地址
#### CX(Count Register)的特殊作用
在使用LOOP指令时，使用CX跳出循环。CX不等于0的时候，CX = CX - 1，否则跳出
#### DX(Data Register)的特殊作用
详看AX

### 索引寄存器(Index Registers)
![](https://www.tutorialspoint.com/assembly_programming/images/register2.jpg)
> 与BX寄存器功能相近，但是不能拆分成两个8位寄存器使用

两者都是在某个地址的基础上进行偏移变化，因此都需要基址寄存器：
- 一般与DS数据段寄存器使用
- 在串处理指令中，SI与DS联用，DI与ES联用

### 指针寄存器(Pointer Registers)
![](https://www.tutorialspoint.com/assembly_programming/images/register3.jpg)
> IP(Instruction Pointer)
#### BP(Base Pointer)的特殊作用
如果寻址操作的偏移地址为BP且段地址不给出，则段地址默认为SS
```
MOV AX, [BP]        ; equal MOV AX, SS:[BP]
MOV AX, CS:[BP]
```
#### SP(Stack Pointer)的特殊作用
`SS:[SP]`永远指向栈顶
## 控制寄存器(Control Registers)
IP与FL(Flag Registers)相结合被认为是控制寄存器
FL上的标志位通常有:
- Overflow Flag(OF)：标志带符号算术运算后是否高位溢出
- Direction Flag(DF)：标志比较字符串的方向，0表示左->右，1表示右->左
- Interrupt Flag(IF)：标志是否收到外中断
- Trap Flag(TF)：标志软中断，在debug模式下为1
- Sign Flag(SF)：表示运算后的符号，0为正，1为负
- Zero Flag(ZF)：表示比较或者算法的返回值，非0为1
- Auxiliary Carry Flag(AF)：标志单位计算的进位
- Parity Flag(PF)：表示奇偶校验结果
- Carry Flag(CF)：标志算术运算后的进位

Flag|||||O|D|I|T|S|Z||A||P||C|
-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-
Bit no|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|

## 段
### 分段的原因
在**8086**CPU中，地址总线有20根，也就是说每次能传输20位的地址，寻址能力有2^20=1Mb。但是CPU的寄存器只有16位，并不能一次处理20位的地址。因此**8086**在内部通过两个16位的地址合成一个20位的地址
![](https://images.cnblogs.com/cnblogs_com/BoyXiao/Windows-Live-Writer/80x86--CPU-_11929/image_thumb_10.png)
> 基地址 = 段地址 << 4
物理地址 = 基地址 + 偏移地址
### 段的类型
- **数据段** 存放数据的内存
- **代码段** 存放代码（指令）的内存
- **栈段** 具有栈结构的内存
### 段寄存器
#### CS与IP
CS是代码段寄存器，通常结合IP进行指令的读取。指令的地址在`CS:[IP]`
#### SS与SP与BP
SS是堆栈段寄存器，8086CPU不检查对栈的越界操作

---
## 汇编程序
一个汇编程序可以分为
- **数据段(data section)**
    数据段用于声明初始化的变量或者常量，其内容不会在运行的时候改变。声明的语句为：
    ```section .data```
- **bss段(bss section)**
    bss段用于声明变量。声明语法为：
    `section .bss`
- **代码段(text section)**
    代码段用于保存实际的代码。必须以`global _start`告诉内核程序的执行位置。
    ```
    section .text
        global _start
    _start:
    ```
- **注释**
    以`;`开头

汇编语句由有三种类型
- 可执行指令
- 汇编程序指令或者伪操作
- 宏

汇编语句语法
> [label] mnemonic [operands] [;comment]
[标签] 指令 [操作数] [注释]

## Hello world
```
section	.text
	global _start       ;must be declared for using gcc
_start:                     ;tell linker entry point
	mov	edx, len    ;message length
	mov	ecx, msg    ;message to write
	mov	ebx, 1	    ;file descriptor (stdout)
	mov	eax, 4	    ;system call number (sys_write)
	int	0x80        ;call kernel
	mov	eax, 1	    ;system call number (sys_exit)
	int	0x80        ;call kernel

section	.data

msg	db	'Hello, world!',0xa	;our dear string
len	equ	$ - msg			;length of our dear string
```
## VS中转汇编的要点
- 不要在debug模式下转汇编，debug会增加很多看不懂的函数以及变量
- 在release下要记得关掉优化，不然某些变量会消失
## 系统调用
系统调用是从用户态切换到内核态的一组API。调用步骤如下：
1. 将系统调用码放入EAX
2. 将参数依次存入EBX,ECX,EDX,ESI,EDI,EBP寄存器
3. 启动中断`int 0x80`(linux)
4. 返回值存储在EAX

系统调用码在 */usr/include/asm/unistd.h*中列出，下表列举出一些系统调用
|EAX|NAME|EBX|ECX|EDX|ESX|EDI|
-|-|-|-|-|-|-|
1|SYS_EXIT|int|
2|SYS_FORK|struct pt_regs|
3|SYS_READ|uint|char*|size_t|
4|SYS_WRITE|uint|const char*|size_t|
5|SYS_OPEN|const char*|int|int|
6|SYS_CLOSE|uint|
## 寻址方法
- **寄存器寻址(Register Addressing)**
    操作数已经在寄存器里面。源和目的数都可以是寄存器。
    ```
    mov dx, second_parm
    mov first_parm, cx
    mov eax, ebx
    ```
- **立即寻址(Immediate Addressing)**
    立即数可以为8位或者16位。源可以为寄存器或者储存器位置，目的数为立即数。速度最快。
    ```
    byte_val db 150 ;定义byte类型变量
    add byte_val, 65
    add ax, 45h
    ```

## 常用指令
### int
>int 0x80

常用指令为`int 0x80`，表示中断进行系统调用
### call
>call _printf

函数调用
###  cdq
> cdq

将eax的第31位复制到edx的每一位上。常出现在除法中，实际上是有符号数位数的扩展
### neg
> neg eax

求补运算，在数学上相当于取负
### setne
> setne al

set not equal，ZF==1时al=0，否则al=1
### sbb
> sbb eax, ebx

带借位减法指令。上面等价于`eax = eax - ebx - CF`
eax = 1020H, ebx = 1200H, CF = 1, => sbb eax, ebx = FE1F
### shl
> shl eax, 3

位运算左移。Shift Logical Left。
### sar
> sar eax, 3

位运算右移。
### rep
> rep opr

重复执行后面的指令。ecx每执行一次-1。
### stos
> mov ecx, 30
mov eax, 0cccccccch
rep stos dword ptr es:[edi]

串存储指令。将al/ax/eax的值存储到es:edi指定的内存单元
### scasb/scasw/scasd
> push esi
mov esi, [esp+4+arg_0]
push edi
mov edi, esi
or ecx, 0ffffffffh
xor eax, eax
repne scasb

在字符串或者数组中寻找一个值。分别将al/ax/eax中的值与edi寻址的一个byte/word/dword进行比较。常常结合repne使用。若查找成功，则edi指向匹配字符后面的位置。示例含义是内联strlen
### test
> test eax, 100b
jnz loc_40000

两操作数做与运算，仅修改标志位，不回送结果。

## 汇编下的加减乘除
### 加法
- 两个数