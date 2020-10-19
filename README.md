**使用 8086 ASM汇编**

**源程序.asm  通过 编译,链接 生成 二进制机器码文件**

> **ICH HUB 是南桥, IO集中控制器**
>
> **CPU里面存在一块小电池, 用于给 时钟供电**

**所有的存储器  (RAM,ROM都算)  都会被看成是一个逻辑存储器, 进行统一编址,每个存储器都会占有一段地址空间**

**向某个特定的内存位置写入信息和数据, 就可以控制某个硬件和传入给这个硬件信息**

<img src="png/8086内存地址空间的分配方案.png" alt="8086内存地址空间的分配方案" style="zoom:80%;" />

<img src="png/实模式下内存分布.jpg" alt="实模式下内存分布" style="zoom:80%;" />



<img src="png/计算机结构图.png" alt="计算机结构图" style="zoom:67%;" />



## 目录

- [编译过程](#编译过程)
- [运行过程](#运行过程)
- [寄存器和数据存储](#寄存器和数据存储)
- [内存中存储的字型和字节型区分](#内存中存储的字型和字节型区分)
- [时钟](#时钟)
- [确定物理地址的方法](#确定物理地址的方法)
- [代码段操作](#代码段操作)
- [DS和数据段](#DS和数据段)
  - [数据段操作](#数据段操作)
- [栈段指针SS和SP寄存器](#栈段指针SS和SP寄存器)
- [实用程序结构](#实用程序结构)
  - [将字符进行大小写转换示例代码](#将字符进行大小写转换示例代码)
- [伪指令](#伪指令)
- [汇编指令详解](#汇编指令详解)
  - [mov和add](#mov和add)
  - [sub 减法指令](#sub)
  - [mul 乘法指令](#mul)
  - [div 除法指令](#div)
  - [inc 自增指令](#inc)
  - [loop 循环指令](#loop)
  - [and和or 与和或运算](#and和or)
  - [jmp和jcxz 无条件和有条件跳转指令](#jmp和jcxz)
  - [call 和ret  调用和返回指令](#call和ret)
  - [nop 空指令](#nop)
  - [DF标志位和movsb串传送指令](#DF标志位和movsb串传送指令)
  - [rep 循环后面单个指令](#rep)
  - [移位指令](#移位指令)
  - [cmp对比指令和 ja 跳转](#cmp和ja)
- [操作显存数据](#操作显存数据)
- [数据标号](#数据标号)
- [数据的直接定址表](#数据的直接定址表)
- [代码的直接定址表](#代码的直接定址表)
- [对外部设备的控制](#对外部设备的控制)
- 



## 编译过程

**可执行文件 头部有相关描述信息, 其他的没有**

**Dos 使用的 masm 编译器,  源代码中使用的是标志号 十六进制数,  16H ,10D**

**Linux和Mac 使用的是 nasm 编译器, 源代码中使用的是  0x 表示的十六进制数**

**开始执行后, CX 寄存器里面保存的是 代码段的长度**

```bash
# a.asm  -> a.obj  ->  a.out/exe
# 源代码  -> 目标文件 ->  可执行文件
# masm/nasm编译源代码生成目标文件  ->  link 将.obj目标文件 转化成可执行文件

# 其他文件:
	  # masm 会生成以下中间文件
		    # *.LST   列表文件; 编译器将源程序编译成目标文件过程中产生的 中间结果.
		    # *.CRF   交叉引用文件;  和列表文件一样, 是编译器将源程序编译为目标文件过程中产生的中间结果
		# link 会生成以下中间文件
				# *.map    映像文件; 是连接程序将目标文件链接为可执行文件过程中产生的中间结果
				# *.lib    库文件; 包含了一些可以调用的子程序, 就是动态库文件
				

```



## 运行过程

- **Dos 启动后, 计算机由 '命令解释器' (程序command.com) 控制**
- **运行可执行程序时,  command 将程序加载入内存, 设置 CPU 的CS:IP 指向程序的第一条指令(即程序的入口), 使得程序得以运行**
- **程序运行结束后, 返回到 '命令解释器', CPU 继续执行 command**
- **运行 DosBox的 debug时:**
  - **command程序加载debug.exe**
  - **debug 将 用户程序 加载到内存**
    - 程序运行结束后, 要返回到debug中,  使用q命令来退出debug 
      - **最后会返回到 command中**





## 寄存器和数据存储

**8086 CPU 拥有14个寄存器**

**每个寄存器都是16位的, 可以放两个字节**

**8086字长为16位, 目前的32位处理器字长为32位,  X86-64 则是64位字长,字长是运算器每次能处理的数据最大长度**

==**保护模式, 实模式 都是以段寄存器为基础的**==



- **`CS:IP  指令地址`**
- **`DS:偏移量  数据地址`**
- **`SS:SP  栈地址`**



|      寄存器编号 | 作用                             | 具体描述                                                     |
| --------------: | -------------------------------- | :----------------------------------------------------------- |
|  **AX**, AH, AL | **通用寄存器, AH高8位, AL低8位** | **整数商寄存器, div BX ;整数商会放在AX , 余数在DX**          |
|  **BX**, BH, BL | 通用寄存器                       | **基址寄存器**                                               |
| **CX** , CH, CL | 通用寄存器                       | **控制  loop循环次数,  程序执行后存放代码长度**              |
|  **DX**, DH, DL | 通用寄存器                       | **余数商寄存器, div BX ;整数商会放在AX , 余数在DX**          |
|   ------------- | -------------                    |                                                              |
|          **SI** | **源变址寄存器**                 | **常用用于地址偏移量表示**                                   |
|          **DI** | **目标变址寄存器**               | **常用用于地址偏移量表示**                                   |
|   ------------- | -------------                    |                                                              |
|          **SP** | **指针寄存器**                   | **栈顶偏移地址,  SS:SP**                                     |
|              BP | 指针寄存器                       |                                                              |
|   ------------- | -------------                    |                                                              |
|          **IP** | **指令指针寄存器**               | **指向下一条要执行的指令的地址, CS:IP 才是地址位置**         |
|   ------------- | -------------                    |                                                              |
|          **CS** | **代码段寄存器**                 | **代码段寄存器, CS:IP 表示下条指令地址, 会被抽象为PC寄存器** |
|              SS | 栈段寄存器   **stack栈**         | **栈段地址寄存器,  SS:SP  表示栈顶位置**                     |
|          **DS** | 数据段寄存器                     | **数据段寄存器,  DS:偏移量   即可得到数据段的地址**          |
|              ES | 附加段寄存器                     |                                                              |
|   ------------- | -------------                    |                                                              |
|  **PSW / FALG** | **标志寄存器**                   | **程序状态字寄存器, DF标志位**                               |
|   ------------- | -------------                    |                                                              |
|         **ESP** | **栈帧寄存器**                   |                                                              |
|   ------------- | -------------                    |                                                              |
|                 |                                  |                                                              |



**PSW/FLAG 标志位寄存器的示意图**

| 15   | 14   | 13   | 12   | 11   | 10     | 9    | 8    | 7    | 6    | 5    | 4    | 3    | 2    | 1    | 0    |
| ---- | ---- | ---- | ---- | ---- | ------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|      |      |      |      | OF   | **DF** | IF   | TF   | SF   | ZF   |      | AF   |      | PF   |      | CF   |

- **CF : 记录相关运算指令执行后,  无符号数运算   最高有效位向更高的假想位 进位或借位 时 该值为1.当AL=98H , `add AL,AL`,执行后CF=1**
- **PF :记录相关运算指令执行后, 其结果中 1 的个数是否为偶数, 1的个数为偶数或0 时 PF=1, 当1的个数为奇数时 PF=0**
- **AF :**
- **ZF  :记录相关运算指令执行后, 其结果为0时 那么 ZF标志位=1 , 否则 ZF=0**
- **SF : 记录相关运算指令执行后, 其结果是否为负数, 如果结果为负数那么 SF=1, 非负数 SF=0**
- **TF :**
- **IF :**
- **DF  :控制 `movsb 和 movsw` 指令中操作的 SI和DI寄存器的增减**
- **OF :记录相关运算指令执行后,  有符号数运算 是否发生溢出, 溢出OF=1, 没有溢出 OF=0**





## 内存中存储的字型和字节型区分

- **字 （word）计算机进行数据处理时，一次存取、加工和传送的数据长度称为字**
- **8086CPU 一个字是2字节, x86是4字节(双字) , x86_64 是8字节(四字)**
  - 0 地址 **单元** 中存放的 **字节型** 数据是 0x20,  一个字节
  - 0地 **字单元** 中存放的是 **字型** 数据是 0x4E20 , 两个字节, 小端





## 时钟

- **CPU 里面存在两个时钟**
  - 计时器,  由 CPU 内部的一小块电池供电
  - 间隔时钟,  每过一段时间就发出一个中断, 用于控制CPU进程的调度







## 确定物理地址的方法

**所有内存单元  `( 所有RAM, ROM, 包括显卡和BIOS)` 构成的存储空间是一个   一维的线性空间**

**每一个内存单元在这个空间中都有唯一的地址, 这个唯一的地址称为  物理地址**

**内存并没有分段,  段的划分来自于CPU**

16根地址线 可以寻找的地址为64KB,    `(2^16)/1024= 64KB`

==**`CS代码段寄存器, DS数据段寄存器 , SS栈段寄存器,  ES附加段寄存器`**==

> **8086CPU有20根地址总线, 支持最大1MB内存地址,  0xFFFFF ~ 0x00000**
>
> - 8086使用 两个16位地址 `(段地址,偏移地址)`  合成一个20位的物理地址
>   - **传输给 位址加法器, 来合成一个20位的地址**
>     - **物理地址 =  段地址*16  + 偏移地址**
>       - **`0x12402 = 段地址0x1234 * 16 + 偏移地址0x00C2`**
>       - ==**可以用不同的段地址和偏移地址来表示同一个物理地址**==
> - **假设: 数据在 0x21F69 内存单元中, 段地址是 0x2000 **
>   - **数据存在内存  2000:1F60 单元中. 都是十六进制表示. CS=0x2000, IP=0x1F60**
>   - **数据存在内存的 0x2000段中的  0x1F60 单元中**

<img src="png/地址加法器.png" style="zoom:67%;" />

<img src="png/8086CPU给出物理地址的方法.png" alt="8086CPU给出物理地址的方法" style="zoom:67%;" />





## 代码段操作

**CS:IP 是 CPU将要执行的下一条指令的地址**

**使用 jmp指令 来修改 CS和IP,   mov无法修改IP**

```assembly
jmp  段地址:偏移地址
jmp  2AE3:2      #跳转到 0x2AE32 地址处执行,  同时CS修改为 0x2AE3, IP为0x02
jmp  AX          #将IP修改为 AX,并跳转, 当前段:AX
```



## DS和数据段

**将一组长度为N `(N <= 64KB)`, 地址连续, 起始地址为16的倍数的内存单元作为专门存储数据的内存空间, 从而定义了数据段**

**物理地址 = 段地址DS *16 + 偏移地址**

- **段地址就是  DS寄存器的值,  DS段寄存器可以使用其他的寄存器或内存中存储的值来初始化,不能直接给立即数**
- **偏移地址 随便给, 立即数 和寄存器 以及从内存取值都行 , 只要不大于 0xFFFF 即可**

```assembly
mov  DS, [1]    ; 从内存取值, 写给 DS 数据段寄存器
mov  DS, ax
mov  [1], DS

mov  AX, [BX + 200 + SI]   ; 与上面相同, 只不过多了个 0x200 十六进制立即数
mov  AX, [200 + BX + SI]	 ; 与上面相同
mov  AX, 200[BX][SI]       ; 与上面相同
mov  AX, [BX].200[SI]      ; 与上面相同, 都是加法
mov  AX, [BX][SI].200      ; 与上面相同, 都是加法


mov [DI] ,   AX    ; AX的 值写入  DS:DI 内存位置
mov [DI]20,  AX    ; AX的值 写入  DS:DI+20 内存位置,  下面的都是这种方式的变种
mov [20+DI], AX
mov 20[DI],  AX
mov [DI].20,  AX
mov 20.[DI],  AX
mov [20][DI], AX


```



## 数据段操作

**用 DS寄存器加偏移值就可以访问那个位置的数据**

**先修改数据段, 成为自己想要访问的地址的数据段,然后再给出偏移即可**

**8086CPU不支持使用 立即数修改段寄存器,必须通过通用寄存器来进行修改 (IP指针寄存器除外,需用jmp)**

```assembly
;先使用BX来设置 DS数据段寄存器的值
push BX  ;先保存
push DS

mov  BX, 0x1000
mov  DS, BX    ; 设置数据段, 如果不设置, 就会读取当前数据段的值
mov  AL, [0]   ; 读取1个字节的数据, 后面翻译成 [DS:0], 物理地址是 0x10000
mov  AX, [1]   ; 读取2个字节的数据, [DS:1]

mov  AX, [BX + 1]   ; 读取 BX寄存器的值 然后+1, 在与 DS组合 ,得到数据地址,然后取出地址数据给AX
mov  AX, 200[bx]    ; 与 mov AX, [BX +200] 相同
mov  AX, [BX].200   ; 与 mov AX, [BX +200] 相同

mov  AX, [BX + SI]   ; 将 BX 当作基址寄存器, 将 SI当初变址寄存器 ,以及DS段寄存器  组合来进行寻址
mov  AX, [BX] [SI]   ; 与 mov AX, [BX + SI] 相同

mov  AX, [BX + 200 + SI]   ; 与上面相同, 只不过多了个 0x200 十六进制立即数
mov  AX, [200 + BX + SI]	 ; 与上面相同
mov  AX, 200[BX][SI]       ; 与上面相同
mov  AX, [BX].200[SI]      ; 与上面相同, 都是加法
mov  AX, [BX][SI].200      ; 与上面相同, 都是加法


mov  [BX+1] , AX  ; 将AX的值 给予,  DS:BX+1 , BX寄存器的值+1 ,偏移地址
mov  [4], AX     # 将AX寄存器中两个字节的内容, 放到 DS:4 数据段指向的位置, 占据 0x10004和0x10005
mov  [2], AL     # 将AX寄存器中两个字节的内容, 放到 DS:4 数据段指向的位置, 占据 0x10002



pop  DS
pop  BX
```









## 栈段指针SS和SP寄存器

**CPU不会检查和限制 SP 的栈越界**

```assembly
; SS:SP 是栈顶地址, 从高地址向低地址递减,  SS栈顶段地址:SP栈顶偏移地址
  ; SS:SP 指向栈顶,有数据时他的指向是最后压入进来的数据低地址位置, 栈为空时, 指向无数据位置
; SS 栈顶段寄存器可以通过 寄存器 和 内存数据 来进行赋值
; SP 偏移值 可以使用很多方法赋值
; 栈以 SP为准, 确定能存储数据的长度,  当 SP=0xFF 时, 那么栈只能存储 256个字节
  ; 当 SP为0 后, 继续使用 push  AL 指令, 那么SP会变成 FF , SS不变, 但栈异常了, 不可以这么做

push  AX  ;首先 SP指针减2(AX寄存器2字节), 然后将AX寄存器的值入栈,从低地址向高地址方向写
            ; 计算方式: (SP) = (SP) -2
                   ;   ((SS)*16 + (SP)) = (AX)  ;前面表示的是地址, 后面是将AX内容写入这个地址
                   
pop   AX  ; 将栈顶的值弹出,赋给AX寄存器,  SP= SP +2 
            ; 计算方式:  (AX) = ((SS)*16 + (SP))
            ;           (SP) = (SP) + 2    ;将SP寄存器的值取出来 +2, 再放到SP里面去
```





## 实用程序结构

```assembly
assume cs:code , ds:data, ss:stack
data segment   ;设置数据段的内容和位置, 应该让DS段寄存器等于这个值
	dw 0123H, 0456H, 0abcH, 0defH, 0fedH, 0cbaH, 0987H ;8个双字, 占据 16字节
data ends

stack segment ;设置栈段的内容和位置, 当SS段寄存器 等于这个值
	dw 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0   ; 16个 双字,占据 32字节
stack ends

; 代码段是默认初始化的, 不需要设置 ,运行时系统就会自动指定
code segment  ; 设置代码段内容和位置
start:
 ; 初始化 各个段寄存器
	mov  AX, stack    ; 栈段的位置值 给予AX
	mov  SS, AX       ; 将AX的值 赋予 SS段寄存器
	mov  SP, 20H      ; 栈的长度, SP =20,  只存放 32字节的数据,SS:SP 
	mov  AX, data     ; 数据段的位置值 给予AX
	mov  DS, AX       ; 初始化数据段, DS:偏移量.    0 <= 偏移量 < 10H
	
;入栈操作
	mov  BX,0
	mov  CX,8    ;设置循环为8次 , 实际执行为 9次, 第一次执行不算做循环
s: 
  push [BX]    ;将 DS:BX 数据段 内存位置的值入栈, 入栈宽度为2,  SP -= 2
  add  BX, 2   ; BX += 2
  pool s 
  
;出栈操作
	mov  BX, 0
	mov  CX, 8
p:
  pop   [BX]  ; 这种出栈顺序, 会将上面入栈的内容,倒过来读取,  12 34 56 变  56 34 12
  add   BX, 2
  loop  p
	
	; 程序结束 , 交还CPU控制权
	mov  AX, 4c00H  ; 运行完最后这两条指令,当前进程就会结束, 最后这两条指令结合起到一个作用
	int  21H        ; 作用是: 程序运行结束后, 将CPU的控制权交给使它得以运行的程序 (通常是DOS系统)

code ends     ;与上面的 code segment 配合, 表示段结束
end   start     ;汇编程序结束标记,让编译器知道 程序在此处结束
	
```





## 将字符进行大小写转换示例代码

```assembly
assume   cs:codesg, ds:datasg 


datasg segment
	db 'BaSiC'         ;小写转大写 ,大写比小写 小20H
	db 'iNfOrMaTiOn'   ;大写转小写
datasg ends


codesg segment
start:
	mov AX, datasg
	mov DS, AX

	mov CX , 5
	mov BX, 0

big:
	mov  AL, [BX]   ; DS:BL
	and  AL,  11011111b    ;0xDF
	mov  [BX], AL
	inc   BX  ;1  2  3   4 
	loop  big ;4 3  2 1  0

	mov CX, 000BH;  0xA
sum:
	mov AL, [BX]
	or  AL,  100000b ;  0x20
	mov [BX] , AL
	inc  BX
	loop sum


	mov ax,4c00H
	int 21H
	
codesg ends
end start
```



# 伪指令

**伪指令没有对应的 机器码 指令, 最终不会被CPU执行**

**伪指令 是由编译器来执行的指令, 编译器根据伪指令来进行相关的编译工作**

> **end   start     ;汇编程序结束标记,让编译器知道 程序在此处结束, 同时 start 也是程序开始执行位置的标号**

- **assume**
  - **segment**
    - **db**
    - **dw**
    - **dd**
      - **dup**
  - **ends**
- **end**

```assembly
assume cs:codesg   ;伪指令, 段定义, 假设某个 段寄存器 和程序中的某个用 segment ... ends定义的段
									 ; 相关联,  assume cs:codesg 指的是 CS寄存器与 codesg关联, 将定义的 codesg
									 ; 当作程序代码段使用, 也就是计算当前代码段位置,然后赋值给 CS:IP
assume ds:datasg  ;段名,  segment关键字表示 段从当前开始,设置的是数据段,需要将这个值给予DS段寄存器
datasg segment
	db 'BaSiC'         ;这样来定义数据, 每个字符占1字节, 默认位置是  datasg:0, 从B开始
	
	db  3 dup(0)       ;dup伪指令, 定义三个字节, 它们的值都是0, 相当于 db 0,0,0
	dw  4 dup(0,1,2,3) ;定义32个字节, 由 0,1,2,3,0,1,2,3,0,1,2,3  构成
										  ;相当于 dw 0,1,2,3,0,1,2,3,0,1,2,3  ; 每个数据都占用2字节
	db 2 dup ('abc', 'ABC')  ; 定义12字节, 相当于  db 'abcABCabcABC'
	dd 2 dup (1,2,3)  ; 定义24字节,相当于两组  db 0x0000 0001 ,0x0000 0002 ,0x0000 0003
datasg ends          ;定义结束符
									 
codesg segment    
; 上面两条代码在 nasm 汇编中并不存在, 最下面的两条也不存在,  只能在 Dos中使用
start:
	mov  AX, datasg  ; 段位置
	mov  DS, AX      ;段位置给予DS寄存器, 默认偏移值为0,   DS:0
	mov  ax, 0123H  ; 上下两部分都是伪指令
	mov  ax, codesg   ;可以这么使用

ret:
	; 进程在再退出前, 应该使用下面的两条代码 执行退出指令, 交还CPU控制权
	mov  AX, 4c00H  ; 运行完最后这两条指令,当前进程就会结束, 最后这两条指令结合起到一个作用
	int  21H        ; 作用是: 程序运行结束后, 将CPU的控制权交给使它得以运行的程序 (通常是DOS系统)

codesg ends     ;与上面的 codesg segment 配合, 表示段结束
end   start     ;汇编程序结束标记,让编译器知道 程序在此处结束, 同时 start 也是程序开始执行位置的标号
```



# 汇编指令详解

`[ ]` 表示取地址的值, `idata` 表示一个常量(1,2,3..) 这个并不是汇编语法内容.

## mov和add

**8位的寄存器 AL, 如果 ADD 加的值过大 产生了向第九位的进位, 那么这个进位会被清除, 不会放到 AH中**

```assembly
mov  AX, 18H     #将 18 十六进制立即数放入 AX寄存器,  AX = 18
mov  AX, BX	     #  AX = BX
mov  AL, 0x100   #  AL是8位寄存器,加上一个9位数还是等于0,溢出位不会添加到 AH中
mov  AL, [2]     # 读取数据段DS寄存器 加2 偏移值 指向的数据内容,放到AL, 读取一个字节
mov  [4], AX     # 将AX寄存器中的内容, 放到DS寄存器加2偏移值的 数据段指向的位置
mov  [BX+SI], AL  #将AL寄存器的值写入地址  DS:BX+SI, 必须是 BX+SI, 否则该指令错误


add  AX, 8       #加法, 将 AX寄存器的值 加上8 ,再写给 AX,  AX+=8
add  AX, BX			 # AX += BX
add  AL, 0x123   # AL = 0x23,  前面的1 会被扔掉,   AX = AH:AL = 0x0023
add  [0],  AL    # 取 DS:0 内存位置的数据, 与AL寄存器的内容相加, 结果写入 DS:0 内存位置
```



## sub

**减法指令**

```assembly
sub  BX,1       # BX = BX -1    寄存器的值减去1, 再写入 BX寄存器
sub  AX,[1]     # AX = AX - [DS:1]
sub  [2], AX    # [DS:2] = [DS:2] - AX
```



## mul

**乘法指令**

- **两个相乘的数 要么都是8位, 要么就都是16位**
  - **如果是8位的两个数相乘,  一个乘数默认放在 `AL寄存器中` ,一个放在 8位的 `某个寄存器 或内存 字节 单元中`**
  - **如果是16位的两个数相乘,  一个乘数默认放在 `AX寄存器中` ,一个放在 16位的 `某个寄存器 或内存 字 单元中`**
- **乘法结果:**
  - **8位乘法,  结果默认放在 `AX寄存器` 当中** 
  - **16位乘法, 结果的 高16位默认在 `DX寄存器` 中存放, 低16位在 `AX寄存器` 中存放**
- **计算的结果如果大于 255  就必须用16位乘法**

```assembly
mov  al, 1f0H   ;将一个乘数放在 默认的AL寄存器中
mov  bl, 1e     ; 另一个乘数, 当作参数传给 乘法指令
mul  bl        ; 做 8位 乘法, 结果放在 AX寄存器当中



mov  AX, 12fH   ;将一个乘数放在 默认的AX寄存器中
mov  BX, 12eH   ; BX 保存了内存地址的偏移量
mul  [BX]       ; 取AX寄存器的值, 与内存 DS:BX 位置的值  做 16位 乘法
							    ; 结果的 高16位在 DX寄存器当中, 低16位在 AX寄存器中存放
```





## div

**除法指令**

- **两个数相除,  除数为8位,那么被除数应该位16位,  除数为16位, 被除数就应该是32位**
  - **被除数为 16位, 则除数应设置为8位`(内存或寄存器)`,  被除数默认放在 `AX寄存器` 中**
  - **被除数为 32位, 则除数应设置为16位,  被除数 低16位放在 `AX寄存器中 ,高16位放在 DX寄存器中` **
- **除法结果:**
  - **除数为8位,  结果默认 的商放在 `AL寄存器` 当中, 余数则放在 `AH寄存器` 当中** 
  - **除数为16位,  结果默认 的商放在 `AX寄存器` 当中, 余数则放在 `DX寄存器` 当中** 

```assembly
mov  AX, 1f0H   ;将 被除数  放在默认的AX寄存器中   6/2
mov  bl, 1e     ; 除数, 当作参数传给 乘法指令
div  bl        ; 做 8位 除法, 结果放在 AX寄存器当中


mov  DX, 012fH   ;将一个被除数的高16位放在 默认的DX寄存器中
mov  AX, 012fH   ;将一个被除数的低16位放在 默认的AX寄存器中
mov  BX, 12eH   ; 被除数从内存中读取, 进行16位除法
mul  [BX]       ; 取AX寄存器与DX 寄存器的值, 与内存 DS:BX 位置的值  做 16位 除法
							    ; 结果的 高16位在 DX寄存器当中, 低16位在 AX寄存器中存放
```









## inc

**自增指令**

```assembly
inc  BX   ; 将 BX寄存器的值 增加1 并写回 BX
          ; 同样无法用于 SS  CS  SP 等 栈寄存器
```



## loop

**计数型循环指令**

**首先会减少 CX寄存器的值, 然后再查看减1后的CX寄存器是否大于0**

```assembly
start:           ; start:  表示的是标号, 好比 goto start;  代码
	mov   CX, 10H  ; 10H = 0x10
	loop  start    ; 好比goto 到 start, 先将 CX寄存器的值 -1, 然后再判断CX是否为 0 然后停止循环
				         ; start 标号就是 IP寄存器所存储的偏移值
				         ; loop [SP:IP]   ;与跳转相当,会将CX寄存器的值-1,大于0时,会修改SP和IP寄存器的值
```



## and和or

**and与,  or或**

```assembly
and   AX, 010101b  ;这里尽量使用二进制,  AX = AX & 0x15
or    BX, 010101b  ; BX =  BX | 0x15
```



## jmp和jcxz

**使用 jmp指令 来修改 CS和IP,   mov无法修改IP**

- **转移指令的分类:**
  - **按照转移行为:**
    - **段内转移 :  只修改IP寄存器, 例如 `jmp AX`**
      - **`jmp 标号`  段内转移, 标号会修改为偏移值,  PC= PC +偏移值, PC永远指向下一条要被执行的指令**
        - 这个偏移值可以 **是正数也可以是负数**
        - **偏移值还可以从内存中取出,然后来进行设定**
          - `jmp word ptr ds:[0]`  ; 关键字word ptr ,代表从内存中取出一个字, 这个字是转移的目的偏移地址
          - `jmp dword ptr ds:[0]` ; 关键字dword ptr ,代表从内存中取出两个字, 高位字是CS段, 低位是IP偏移地址
    - **段间转移: 同时修改CS和IP寄存器, 例如 `jmp far ptr 标号`**
      - **CS:IP 寄存器都会被修改, 使用的是目的地址**
        - **机器码 给出的参数 是具体的内存位置, 而不是偏移值**
          - `jmp  far ptr 标号`
  - **根据指令对IP寄存器修改的范围不同:**
    - **段内短转移: IP寄存器修改范围为 `-128 ~ 127` 2^8   一个字节**
      - **`short`  这个参数代表段内短转移, 最大不可超过 有符号的1字节长度, 后面的值还是偏移量,而不是具体值**
        - `jmp  short 标号`
    - **段内近转移: IP寄存器修改范围为 `-32768 ~  32767`,  2^16   两字节**
      - **`far ptr`  这个参数代表段内近转移 最大不可超过 有符号的2字节长度, 后面的值还是偏移量,而不是具体值**
        - `jmp  near ptr 标号`
  - **按照转移指令:**
    - **无条件转移指令:  `jmp`**
    - **条件转移指令:  `jcxz`  , 当CX寄存器为0时,就会跳转, 否则继续向下执行不做任何操作**
      - **里面也是 偏移量,而不是具体的代码段地址**
    - **循环指令: `loop`**
    - **过程**
    - **中断**

```assembly
jmp  段地址:偏移地址
jmp  标号

s: 
   mov AX,1     ;机器码为 B80100
   jmp  short s ;跳转到标号s, 也就是上一行, 机器码为 EBFB,  PC(5) += -5
       					  ;当前PC指向下一条指令, PC=5, 指令jmp占2字节, mov站3字节, 共计5字节, 这个-5就是偏移值

jmp far ptr s  ;段间转移(远转移), 直接对 CS:IP 进行重新赋值, 来进行跳转

jmp short  s    ;8位 的位移, 计算当前 PC寄存器与 s标号处的差值,然后将偏移量写入到具体机器码中
jmp near ptr  s    ;16位 的位移, 计算当前 PC寄存器与 s标号处的差值,然后将偏移量写入到具体机器码中

jmp  word ptr [BX] ;段内转移, 读取内存 DS:BX 位置的两个字节, 取出后给予IP寄存器, 然后运行, 
jmp dword ptr [AX] ;段间转移, 读取内存 DS:AX 位置的四个字节, 取出后 低地址给予IP寄存器, 高地址给予CS寄存器



jmp  s      #跳转到 0x2AE32 地址处执行,  同时CS修改为 s标号的位置, IP为0
jmp  AX          #将IP修改为 AX,并跳转, 当前段CS:AX



jcxz   s  ;检查 CX寄存器是否为0, 如果为0 那么跳转到s标号处执行, 如果不是0, 那么就继续向下执行而不跳转
 						  ; 地址的计算也是偏移值
```



## call和ret

- **`call 标号` ; 指令 拥有两步操作**
  - **有两种执行方式:**
    - **段内转移:   ``call 标号或16位寄存器**
      - **先将IP寄存器的值入栈**
      - **然后跳转到 标号处执行   `jmp near ptr 标号`,  还是使用地址偏移值来进行计算**
    - **段间转移:  `call far ptr 标号`**
      - **首先会将 CS压栈,然后将 IP也压栈**
      - **然后将CS赋予标号所在的段地址, IP赋予标号所在的偏移地址, 跳转过去执行**
- **`ret`  段内近返回**
  - 用栈中的数据取修改 IP寄存器, 相当于 `pop ip`
- **`retf`  段内远返回**
  - 用栈中的数据取修改 CS和IP寄存器, 相当于 `pop IP` 和 `pop CS`

```assembly
s:
call  s ; 跳转到标号处,   如果标号过远 就会将 CS:IP都入栈. 然后用参数覆盖着两个寄存器,进行跳转
 				;如果标号并不远 ,那么就只将 IP入栈, 然后将IP的值与标号地址的值进行计算获得差值,然后与IP相加即可

call far ptr s   ; 远调用转移, 将 CS:IP都入栈 ,然后都修改, 进行跳转

call word ptr  [0]  ;IP入栈,然后取内存中的值 DS:0  两个字节, 给予IP寄存器, 然后再进行调用跳转
call dword ptr [0]  ;CS和IP入栈,然后取内存中的值 DS:0  四个字节, 高位给CS, 低位给IP



ret     ; 近转移指令, 用栈中的数据取修改 IP寄存器, 相当于 pop ip   
retf    ; 远转移指令, 用栈中的数据取修改 CS和IP寄存器, 相当于 pop IP 和 pop CS
```







## nop

**nop  指令的机器码站一个字节, 起到站位作用, 并不会执行任何操作** 





## DF标志位和movsb串传送指令

- **`DF`  方向标志位 (Direction Flag)**
  - **位于 PSW标志位寄存器中 下标是10的位**
  - **功能:**
    - **在 串处理指令movsb 中 控制每次操作后 SI和DI 寄存器的共同增减, 也就是当 执行完 movsb或movsw 指令后,才会进行对SI和DI这两个寄存器中值的修改**
      - `DF = 0` ; 每次操作后 SI和 DI **递增** (+1 或 +2)
        - **`cld` 指令可以将 DF标志位设置为 0   (clear)**
      - `DF = 1` ; 每次操作后 SI和 DI **递减**(-1 或 -2)
        - **`std` 指令可以将 DF标志位设置为 1  (setup)**
- **`movsb` 串传送指令**
  - **功能: (以 字节 为单位进行传送,一次传送1字节)**
    - `((ES)*16 + (DI)) = ((DS)*16 + (SI))`  --->  `[ES:DI] =[DS:SI] `
      - **内存位置 DS:SI 的值 会取出来, 放到 ES:DI 的内存位置中去**
      - **`DF= 0` 那么 SI=SI+1 ,   DI=DI+1 **
      - **`DF= 1` 那么 SI=SI-1 ,   DI=DI-1 **
- **`movsw` 串传送指令**
  - **功能: (以 字 为单位进行传送,一次传送2字节)**
    - `((ES)*16 + (DI)) = ((DS)*16 + (SI))`  --->  `[ES:DI] =[DS:SI] `
      - **`DF= 0` 那么 SI=SI+2 ,   DI=DI+2 **
      - **`DF= 1` 那么 SI=SI-2 ,   DI=DI-2 **
- **`movsb 和 movsw` 经常和 `rep` 指令组合使用**

```assembly
assume DS:data, CS:code

data  segment  ;设置数据段, DS
	db 'Welcome to masm'  ;原始字符串, 16字节
	db 16 dup(0)          ;被赋值到的地方, 16字节
data ends

code segment
start:
	mov  AX, data     ;AX等于数据段的段地址
	mov  DS, AX       ;DS数据段 段地址赋值
	mov  SI, 0        ;原变址寄存器, 受控于 ES寄存器中的DF标志位
	mov  ES, AX       ;让ES寄存器也指向数据段的开头, 变成 数据段寄存器
	mov  DI, 10H       ;目标变址寄存器, 也受控于PSW标志位中的 FD标志位, 指向被赋值的地方.偏移16字节
	cld	  						; 设置ES寄存器中的 DF标志位为0, 每次操作后 都会递增 SI和DI 寄存器
	
	mov  CX,10H    ;设置循环传送,每次1字节, 循环16次
s:   
  movsb         ;串传送指令, 每次传送一个字节, 执行之后 SI和DI 都会自减1
	      				  ;如果这个指令换成 movsw, 那么只需要循环 8次即可. SI和DI都会自减2
  loop s        ;循环

; 第二种处理方法
	mov  AX, 0     ;重置 SI, 和 DI
	mov  SI, AX
  mov  ES, AX       ;让ES寄存器也指向数据段的开头, 变成 数据段寄存器
	mov  AX, 10H
	mov  DI, AX
	cld             ;设置PSW标志位中的 DF标志位为0
	mov  CX, 8      ;循环8次. 每次传送2字节, 主要是给 rep 指令使用的
	rep  movsw      ;rep 指令, 让后面的指令循环多次, 次数是CX寄存器的值. 也是字符串传送指令
	
	; 下面是程序退出代码
	mov  AX, 4C00h
	int  21h
code ends
end start
```





## rep

该指令会根据 CX 寄存器的值, 重复执行后面的指令

```assembly
mov  AX, 0
mov  CX, 10H
rep  add AX,1    ;CX=0x10, 循环执行16次 AX += 1 操作, 最后 AX会等于 0x10


	mov  AX, 0     ;重置 SI, 和 DI
	mov  SI, AX
  mov  ES, AX       ;让ES寄存器也指向数据段的开头, 变成 数据段寄存器
	mov  AX, 10H
	mov  DI, AX
	cld             ;设置PSW标志位寄存器中的 DF标志位为0
	mov  CX, 8      ;循环8次. 每次传送2字节, 主要是给 rep 指令使用的
	rep  movsw      ;rep 指令, 让后面的指令循环多次, 次数是CX寄存器的值. 也是字符串传送指令
```





## 移位指令

- `OPR 是寄存器或立即数,  CNT 必须是CL寄存器 里面存放的是移位的位数`
- **逻辑移位:**
  - **逻辑左移  `SHL  OPR, CNT`**
    - 会在 最低位 端填0
    - 被移出的最高位会写入 `PSW` 寄存器的  CF 标志位, 相当于 乘2 之后的进位
  - **逻辑右移  `SHR  OPR, CNT`**
    - 会在 最高位 端填0
    - 被移出的最低位会写入 `PSW` 寄存器的  CF 标志位
- **算数移位:**
  - **算数左移  `SAL  OPR, CNT`**
    - 会在 最低位 端填0
    - 被移出的最高位会写入 `PSW` 寄存器的  CF 标志位
  - **算数右移  `SAR  OPR, CNT`**
    - **移动时, 会在头部添加 目前最高位的值, 而不是填0, 目前最高位是什么, 就填什么**
    - **最高位的值不变, 被移出的最低位会写入 `PSW`寄存器的 CF 标志位**
      - `MOV BX, 2 ;  ASL AX, BX`  ;-> BX=1010b,   移动后 ->  BX= 1110b,  CF=1
- **循环移位:**
  - **循环左移  `ROL  OPR, CNT`**
    - 被移出的最高位会回到最低位, 并写入 `PSW` 寄存器的  CF 标志位
  - **循环右移  `ROR  OPR, CNT`**
    - 被移出的最低位会回到最高位, 并写入 `PSW` 寄存器的  CF 标志位
- **带进位循环移位:**
  - **带进位循环左移  `RCL  OPR, CNT`**
    - 将最高位 移动到 `PSW` 寄存器的CF标志位中, 将原本存在 `PSW` 寄存器的CF中的值 放入到 最低位
  - **带进位循环右移  `RCR  OPR, CNT`**
    - 将最低位 移动到 `PSW` 寄存器的CF标志位中, 将原本存在 `PSW` 寄存器的CF中的值 放入到 最高位

```assembly
mov  AL, EF     ; AL=1110 1111   , -> 
mov  CL, 3 	    ; 这里必须使用CL来存放移位操作的 移位 位数
shl  AL, CL     ;  逻辑左移  AL=1110 1111   , -> AL=0111 1000
```



## cmp和ja

- **cmp对比指令**
  - `cmp 寄存器, 立即数`  ;如果寄存器的值 大于 立即数, 那么会执行后面的 `ja 标号` 指令, 否则不会进行跳转
- **ja 依靠 cmp 使用的跳转指令**

```assembly
mov AX, 1
cmp AX, 2  ; AX寄存器的值小于2, 那么不会执行跳转, 而是会来到 mov BX,2 进行执行
ja start
mov BX, 2
```





# 操作显存数据

> **屏幕上的内容 等于  显存中的数据**
>
> **`B8000h ~ BFFFFh 共32KB的空间, 是 80*25彩字字符模式,第0页的显示缓冲区`**
>
> - **屏幕大小 共80列, 25行**
>   - **每行所需字节数为 2*80=160**
>     - **第一行 显示缓冲区地址范围是  `B800:0000 ~ B8000:009F`**
>     - **第二行 显示缓冲区地址范围是  `B800:00A0 ~ B8000:0013F`**
>     - **第三行 显示缓冲区地址范围是  `B800:0140 ~ B8000:01DF`**
>     - **最后一行 显示缓冲区地址范围是  `B800:0F00 ~ B8000:0F9F`**
> - **每个在屏幕上显示的字符, 都是ASCII和一个属性字节组合的, 共占据2字节**
>   - **低位字节放显示符号的ASCII,  高位字节显示属性**
>     - **属性: 7闪烁  , 6和5和4是背景RGB,   3高亮,  2和1和0字体颜色RGB**
>       - 例如 `低位41H, 高位9CH` ; 就是蓝底红字, 显示的是 A 字符, 会闪烁且高亮
>         - AX=9C41  写入 B800:0000即可,   (注意高位和地位)

<img src="png/DosBox显存.png" alt="DosBox显存" style="zoom:33%;" />

```assembly
assume cs:code
code  segment
start:
    ; 调用清屏
    ;call clear
	
    ; 调用 设置所有字体的颜色, AX是参数,  低位al是ascii, 高位ah是颜色
    ;mov AX, 200h  ;显示字符不变, 修改所有字符为 绿色, 
    ;call   setWordColor


    ; 调用 设置背景颜色
    ;mov al, 0
    ;mov ah,20h  ; 设置颜色为绿色
    ;call setBackgroundColor

    ; 向上滚动一行
    call upLap

    mov ax , 4c00h
    int  21h


clear:
    ; 清屏代码
    ;  5F 空格ASCII
    ;   属性 全0
    push CX
    push AX
    push BX
    push ES

    mov AX, 0B800h
    mov ES, AX
    mov AX, 05fh
    mov BX, 0
    mov CX, 07d0h
clearLoop:
    mov ES:[BX], AX
    add BX, 2
    loop  clearLoop

    pop ES
    pop BX
    pop AX
    pop CX
    ret


; 设置字体颜色代码,  AL 的最后3位保存了字体颜色设置
setWordColor:    
    
    push CX
    push AX
    push BX
    push ES

    and AX, 0700h  ;只保留最后三位
    mov BX, 0B800h
    mov ES, BX
    mov BX, 0
    mov CX, 07d0h
setWordColorLoop:
    and ES:[BX], 0f8ffh  ; 清除字体颜色
    or  ES:[BX], AX  ; 设置字体颜色
    add BX, 2
    loop  setWordColorLoop

    pop ES
    pop BX
    pop AX
    pop CX
    ret


; 设置背景颜色
setBackgroundColor:
      
    push CX
    push AX
    push BX
    push ES

    and AX, 07000h  ;只保留ah 寄存器的4,5,6 这三位
    mov BX, 0B800h
    mov ES, BX
    mov BX, 0
    mov CX, 07d0h
setBackgroundColorLoop:
    and ES:[BX], 08fffh  ; 清除背景颜色
    or  ES:[BX], AX  ; 设置背景颜色
    add BX, 2
    loop  setBackgroundColorLoop

    pop ES
    pop BX
    pop AX
    pop CX
    ret


;向上滚动一行
upLap:
     
    push CX
    push BX
    push ES
    push DI
    push SI
    push DS

    mov BX, 0B800h
    mov DS, BX
    mov ES, BX
    mov SI, 0a0h   ; 160
    mov DI, 0
    mov CX, 0730h
    
    cld 
    rep movsw 
    
    pop DS
    pop SI
    pop DI
    pop ES
    pop BX
    pop CX
    ret

code ends
end start
```



## 数据标号

**注意 数据标号处 数据的类型和字节长度, 进行写入和读取时必须匹配相同的长度**

- **数据标号 标记了存储数据的 单元地址和长度**
- **数据标号不同于 表示地址的地址标号 `(start: 地址标号)`**
  - 数据标号 可以在代码中使用, 也可以在 数据段中使用

```assembly
assume cs:code

code segment   ; 只有个代码段, 程序开始执行时, 会从 start标号处开始, a  b 这里会跳过
	a db 1,2,3,4,5,6,7,8      ; a 就是数据标号, 记录了a 数据的起始地址偏移,  CS:a
	b dw 0							  ; b 也是数据标号,通过与a进行运算可以获得 a的数据长度
	c dw  a , b           ;c 是数据标号, 里面记录了a 和 b 数据标号的地址, 相当于指针数据
  d dd  a, b      ;d是数据标号, 每个元素占4字节, 高2位保存段地址, 低地址保存偏移量
start:
	 mov si,0
	 mov cx,8
s:  
   mov al, a[si]  ; 地址为  CS:a+SI  ,后面就是偏移值
   mov ah,0    ;AX寄存器的 高8位
   add b,ax    ; 将 AX寄存器的值 写入b 数据标号的位置, 也就是一个数据的位置,
   inc  si
   loop s
			
	 mov  al, a[ax+si+3]    ;地址为  CS:a+si+ax+3	
   mov ax , 4c00h
   int  21h

code ends
end start


```





## 数据的直接定址表

**用查表的方法解决问题**

**利用表 ,在两个数据集合之间建立一种映射关系, 用查表的方法根据给出的数据 得到其在另一集合中的对应数据.**



```assembly
assume cs:code 
code  segment
start:
   mov al, 2Bh
   call showbyte
   mov ax, 4c00h
   int 21h

showbyte:
    jmp short show
    table db '0123456789ABCDEF'   ;字符表 , table是数据标识, 显示字符

show:
    push BX
    push ES 
    push CX

    mov ah, al  ;低位内容 复制到高位
    mov cl, 4h  ;设置下面移位指令的参数
    shr ah, cl  ; 将AX寄存器的高8位,也就是ah 寄存器 的内容 逻辑右移四位, 清空ah寄存器高4位
    and  al, 00001111b   ;al中为低4位的值 进行并操作


    mov bl, ah
    mov bh, 0
    mov ah, table[BX]  ;BX 寻址寄存器

    mov BX, 0b800h
    mov ES, BX
    mov CS:[160*12+40*2], ah
    ;mov CS:[160*12+40*2+1], 9ch

    mov bl, al
    mov bh, 0
    mov al, table[BX]

    mov ES:[160*12+40*2+2], al
    ;mov ES:[160*12+40*2+3], 9ch


    pop CX
    pop ES
    pop BX
    ret


code ends
end start
```





## 代码的直接定址表

```assembly
assume cs:code
code  segment
start:
;下面就是直接定址表和使用方式
    table dw clear, setWordColor, setBackgroundColor, upLap ; 偏移地址, 长度必须是字

    push BX
    push CX

    ; 将BX设置为功能号, 指向想要指向的指令的地址
    mov  BX,0
    add  BX,BX
    call table[BX]  ;调用清屏, 相当于 call clear

;间隙
    mov  BX,1
    add  BX,BX
    mov AX, 200h  ;显示字符不变, 修改所有字符为 绿色, 
    call table[BX] ; 调用 设置所有字体的颜色, AX是参数,  低位al是ascii, 高位ah是颜色
                   ; 相当于 call   setWordColor

;间隙
    mov  BX,2
    add  BX,BX
    mov ah,20h  ; 设置颜色为绿色, 将AX变成参数进行传递
    call table[BX] ; 相当于 call setBackgroundColor

;间隙
    mov  BX,3
    add  BX,BX
    call table[BX] ; 向上滚动一行, 相当于 call upLap
    
    pop CX
    pop BX
    
    mov ax , 4c00h
    int  21h


clear:
    ; 清屏代码
    ;  5F 空格ASCII
    ;   属性 全0
    push CX
    push AX
    push BX
    push ES

    mov AX, 0B800h
    mov ES, AX
    mov AX, 05fh
    mov BX, 0
    mov CX, 07d0h
clearLoop:
    mov ES:[BX], AX
    add BX, 2
    loop  clearLoop

    pop ES
    pop BX
    pop AX
    pop CX
    ret


; 设置字体颜色代码,  AL 的最后3位保存了字体颜色设置
setWordColor:    
    
    push CX
    push AX
    push BX
    push ES

    and AX, 0700h  ;只保留最后三位
    mov BX, 0B800h
    mov ES, BX
    mov BX, 0
    mov CX, 07d0h
setWordColorLoop:
    and ES:[BX], 0f8ffh  ; 清除字体颜色
    or  ES:[BX], AX  ; 设置字体颜色
    add BX, 2
    loop  setWordColorLoop

    pop ES
    pop BX
    pop AX
    pop CX
    ret


; 设置背景颜色
setBackgroundColor:
      
    push CX
    push AX
    push BX
    push ES

    and AX, 07000h  ;只保留ah 寄存器的4,5,6 这三位
    mov BX, 0B800h
    mov ES, BX
    mov BX, 0
    mov CX, 07d0h
setBackgroundColorLoop:
    and ES:[BX], 08fffh  ; 清除背景颜色
    or  ES:[BX], AX  ; 设置背景颜色
    add BX, 2
    loop  setBackgroundColorLoop

    pop ES
    pop BX
    pop AX
    pop CX
    ret


;向上滚动一行
upLap:
     
    push CX
    push BX
    push ES
    push DI
    push SI
    push DS

    mov BX, 0B800h
    mov DS, BX
    mov ES, BX
    mov SI, 0a0h   ; 160
    mov DI, 0
    mov CX, 0730h
    
    cld 
    rep movsw 

    ;处理最后一行,变成空格
    mov BX, 0005fh    ;空格
    mov CX, 50h
upLapLoop: 
    mov [DI], BX
    add DI, 2
    loop  upLapLoop


    pop DS
    pop SI
    pop DI
    pop ES
    pop BX
    pop CX
    ret

code ends
end start
```



# 对外部设备的控制









