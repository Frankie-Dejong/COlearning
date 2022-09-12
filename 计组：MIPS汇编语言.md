# 基础知识

## 位、字节、字和半字

在计算机中，数据存储的最小单位是位（bit），基本单位是字节（Byte），除了他们以外，我们还经常使用字（Word），半字（Half—Word）等单位。

对于32位的机器而言，一个字节等于8位，一个字等于4字节，也就是32位。而一个半字等于2字节，也就是16位。

## 小端存储和大端存储

数据的存储有两种存储方式，分别是小端存储和大端存储。

==在我们的实验中，采取小端存储的方式==

数据按照空间来划分，有高地址，低地址之分，也有高权值位和低权值位的区别。

大端：低权值位数据存储在高地址处

小端：低权值位存储在低地址处

以int a = 1为例，在32位的机器当中int类型的变量占据四个字节的空间。换算为十六进制，即为0x 00 00 00 01

从左到右地址逐渐升高，

小端存储：01 00 00 00

大端存储：00 00 00 01

# 寄存器简介

寄存器是CPU的组成部分之一，是CPU可以使用的最高速的一种存储器，可以用来暂存指令，数据，地址等。一般情况下CPU中只有数量很有限的寄存器可供使用。

在MIPS体系结构中，CPU对数据的操作均是基于寄存器的。内存中的数据需要先使用读取（load）类指令保存到寄存器才可使用。操作完成的数据也不能直接保存到内存中，需要使用装载（store）类指令保存至内存中。

在本实验中，CPU为32位的CPU，一次能处理的最大位数即为32位，绝大部分寄存器也均是32位大小。

## MIPS中的32个通用寄存器。

一般的MIPS汇编程序之中，比较常用的32个通用寄存器，他们没有明确规定的用途，程序员可以随意的对他们进行赋值和取值，同时他们的值也可以直接参与到各种指令当中。然而，对于工程师们来说，依然会以一定的规则来使用它们。

这32个寄存器可以按照序号命名，也可以按照功能命名，作用如下：

<img src="https://github.com/Frankie-Dejong/COlearning/blob/MIPS/截屏2022-08-30%20下午7.43.49.png?raw=true" alt="截屏2022-08-30 下午7.43.49" style="zoom:50%;" />

特别的，对于`$0`和`$1`两个寄存器：

* 对于`$0`赋值并不违法，但并无意义，因为它的值会始终保持为0.
* 一般情况下不要使用`$1`，这个寄存器要留给汇编器使用，否则可能会导致难以发现的bug

## 三个特殊的寄存器

* PC：用来存储当前CPU正在执行的指令在内存中的地址。会进行自动递增寻找下一条指令。对于这个寄存器的值不能用常规的指令进行取值和赋值。
* HI：这个寄存器用于乘除法。它被用来存放每次乘法的结果的高32位，也被用来存放除法结果的余数。
* LO：用来存放每次乘法的结果的低32位，也被用来存放除法结果的商。

## CP0寄存器

在处理异常和终端的时候需要用到CP0寄存器。

CP0是一个系统控制协处理器，而CP0寄存器则是该协处理器工作时需要用到的一些寄存器。在我们的实验中，只会用到其中的4个寄存器：SR、Cause、EPC和PRId。

* SP：用于系统控制，决定是否允许异常和终端
* Cause：记录异常和中断的类型
* EPC：保存异常或中断发生时的PC值，也就是发送异常或中断时CPU正在执行的那条指令的地址。处理完成之后将会根据这个地址继续向下执行
* PRId：处理器ID，用于实现个性的寄存器

# MIPS汇编指令

指令，即是由处理器指令集架构定义的处理器的独立操作，这个操作一般是运算，存储，读取等。一个指令在CPU中的真正存在形式是高低电平，也可以理解为由01序列组成的机器码。

为了帮助人们记忆，指令一般由汇编语言来表示，也就是我们俗称的汇编指令。

## 指令的格式

一般来说，在MIPS汇编语言中，指令的格式如下：

`指令名 操作数1，操作数2，操作数3`

也有如下的指令格式，通常用于存取类指令

`指令名 操作数1，操作数3（操作数2）`

操作数：即指令操作所作用的实体，可以是寄存器、立即数或标签，每个指令都有固定的对操作数形式的要求。标签则会最终由汇编器转换为立即数。

所谓立即数，即在指令中设定好的常数，可以直接参与运算，一般长度为16位二进制。

所谓标签，用于表示一个地址以供指令引用。一般用于表示一个数据存取的地址或者一个程序跳转的地址。在MIPS汇编中，标签的形式如下：

`name:`

其中name可以自行取名。

值得注意的是，在MARS当中，跳转指令只能使用标签来进行跳转，不能使用立即数。

## 机器码指令

众所周知，计算机只能理解二进制形式的数据。而汇编语言最终也会转化为机器语言被CPU直接识别，从而完成相应的操作。

在我们学习的MIPS汇编之中，所有的指令长度均是32位，即四字节，或一字。

所有指令长度均相同，这是精简指令集（RISC）的特征，包括MIPS和ARM架构等；与之对应的是复杂指令集（CISC），例如PC中常用的x86架构。

## 机器码的指令格式

32位的机器码需要一定的格式才能被理解。一般来说，在MIPS指令集中，指令分为三种格式：R型、I型和J型。

* R型指令

  一般情况下用于运算指令，例如add、sub、sll等，其格式如下：

  | op    | rs    | rt    | rd    | shamt | func  |
  | ----- | ----- | ----- | ----- | ----- | ----- |
  | 6bits | 5bits | 5bits | 5bits | 5bits | 6bits |

* I型指令

  I型指令有16位的立即数（或偏移）。因此I型指令一般用于与立即数相运算的指令，或beq、bgtz等比较跳转指令。还有像sw、lw一样的存取指令。

  | op    | rs    | rt    | offset or immediate |
  | ----- | ----- | ----- | ------------------- |
  | 6bits | 5bits | 5bits | 16bits              |

* J型指令

  J型指令常见的有j和jal，他们需要直接跳转至某个地址，而非利用当前的PC值加上偏移量计算出新的地址。

  | op    | address |
  | ----- | ------- |
  | 6bits | 26bits  |

  并非所有的指令都严格的遵守上面三种格式，但就大多数指令而言，都可按上面三种格式进行解释。

解读：

* op：操作码，用于标识指令的功能，R型指令中有func字段来辅助
* func：辅助op来识别指令
* s、rt、rd：通用寄存器的代号，并不特指某一寄存器。范围是`$0~$31`，在机器码中即00000～11111
* shamt：移位值，用于移位指令
* offset：地址偏移量
* immediate：立即数
* address：跳转地址

具体指令作用请查询MIPS指令集手册

# 扩展指令和伪指令

为了方便，MIPS还在标准指令的基础上又提供了许多扩展指令，成为Pseudo Instructions。

此外，为了提升程序编写的灵活性，汇编器还提供了伪指令来让我们指导汇编器的工作。这样我们就可以声明全局标签、声明宏、设置异常数据段和代码段。

## 扩展指令

严格意义上来说，扩展指令是标准指令的封装，因此只是在形式上，他们是一条新的指令，但实际上，在汇编器将其汇编之后依然是使用标准指令来实现的。

* li指令

  它用来为某个寄存器赋值，比如`li $a0,100`就是将100赋给`$a0`寄存器。汇编器在翻译这条扩展指令的时候会根据需要将它翻译成不同的基本指令（集合）。

* la指令

  这条指令也是为寄存器赋值，但是是使用标签来为寄存器赋值。标签本质上就对应一个32位的地址，但li指令并不能直接使用标签来为寄存器赋值，必须要使用la。例如`la $t0,fibs`这条指令就是把fibs这个标签的地址存入$t0中。

* move指令

  `move rs,rt `：将rt寄存器的值赋值给rs

* `bgtz rs,rt,add`:当rs中的值大于rt时，跳转到add；与之对应的是`blez`指令，将在小于等于时进行跳转
* `beq $rs,$rt,add`:当rs的值等于rt时，跳转到add
* `sll $rs,$rt,immediate`：将rt中的值左移immediate位赋给rs
* `slt $rs,$rt,$rd`：rs = rt < rd ？1:0

了解更多的扩展指令，可以查看Mars的Help文档

## 伪指令

伪指令可以指导汇编器如何处理程序的语句，类似于C语言中的预处理命令。伪指令并不会被编译成机器码，常用的伪指令如下：

* .data：用于遇险存储数据的伪指令的开始标志
* .text：程序代码指令开始的标志
* .word：以字为单位存储数据
* .ascliz：以字节为单位存储字符串
* .space：申请若干个字节的未初始化的内存空间

了解更多的伪指令，可以查看Mars的Help文档

## syscall指令

syscall会根据当前寄存器v0的值来进行一些操作，一些常见的对应关系如下：

* 1:打印$a0中存储的int值
* 2:打印$f12中存储的float值
* 3:打印$f12中存储的double值
* 4:打印$a0中存储的以\0结尾的字符串
* 5:读入一个int类型存储在$v0当中
* 6:读入一个float类型存储在$f0中
* 7:读入一个doule类型存储在$f0中
* 8:读入字符串：`$a0`存储字符串存放的地址，`$a1`存储最大读入数量（实际为该数字 - 1）
* 10:程序终止

了解更多的syscall指令，可以查看Mars的帮助文档

# MIPS程序设计：模式化

==本部分旨在给出一些常用C语言翻译汇编语言的模版==

## 条件语句

以简单的if-else语句为例

```assembly
.text
li $t1, 100             #t1 = 100
li $t2, 200             #t2 = 200
slt $t3, $t1, $t2       #if(t1 < t2) t3 = 1 
beq $t3, $0, if_1_else
nop
#do something						#if(t1 < t2)
j if_1_end              #jump to end
nop
if_1_else:							#else
#do something else

if_1_end:
li $v0, 10
syscall
```

## 循环语句

给出一个标准C语言的for循环语句的汇编写法

```assembly
.text
li $t1, 100             #n = 100
li $t2, 0               #i

for_begin1:             #for (int i = 0; i < n; )
slt $t3, $t2, $t1       #{
beq $t3, $0, for_end1  
nop        
#do something
addi $t2, $t2, 1        #i++
j for_begin1
nop                     #}    

for_end1:
li $v0, 10
syscall
```

## 数组

这里以申请一个int[[10]为例

```assembly
.data
array: .space 40	#申请40字节的空间，首地址存储在标签array中
# 我们这里演示如何通过for循环为数组元素赋值

.text
li $v0,5
syscall			#输入一个整数，来确定总共有多少输入
move $s0,$v0
li $t0,0		#循环变量

loop_in:
beq $t0,$s0, loop_in_end	#若循环变量等于输入数量则结束循环
li $v0,5
syscall	#输入一个整数存储在$v0中
sll $t1,$t0,2	#$t1 = 4*$t0，求得当前存储的位置与数组头的offset
sw $v0,array(St1)	#将该整数写入对应地址
addi $t0,$t0,1	#循环变量自增
j loop_in	#无条件跳回循环开始

loop_in_end:
...
```

## 二维数组

以申请一个8*8的矩阵为例

```assembly
.data
matrix: .space 256	申请一个256字节的空间
str_enter: .asciiz "\n"
str_space: .asciiz " "

.macro getindex(%ans,%i,%j)
	sll %ans,%i,3
	add %ans,%ans,%j
	sll %ans,%ans,2
.end_macro
#宏定义求出offset（ans = 4*（8*i+j））
.text
li $v0,5
syscall
move $s0,$v0
#行数
li $v0,5
syscall
move $s1,$v0
#列数
#使用一个双层循环
li $t0,0

in_i:
beq $t0,$s0,in_i_end
li $t1,0

in_j:
beq $t1,$s1,in_j_end

li $v0,5
syscall
getindex($t2,$t0,$t1)
sw $v0,matrix($t2)

addi $t1,$t1,1
j in_j
in_j_end:

addi $t0,$t0,1
j in_i
ini_end:

```

## 宏的使用

为了减少重复的代码，MIPS也支持宏

1. 不带参数的宏

   ```assembly
   .macro macro_name
   # do something
   .end_macro
   ```

   例如，我们可以将停止运行的指令封装为宏，即

   ```assembly
   .macro done
   li $v0,10
   syscall
   .end_macro
   
   #do something
   
   done
   ```

2. 带参数的宏

   ```assembly
   .macro macro_name(%parameter1,%parameter2,...)
   #do something
   .end_macro
   ```

   例如我们在矩阵的例子中使用的那样：

   ```assembly
   .macro getindex(%ans,%i,%j)
   	sll %ans,%i,3
   	add %ans,%ans,%j
   	sll %ans,%ans,2
   .end_macro
   ```

另一种定义宏的方式：

类似于C语言中的define，MIPS提供了.eqv

```assembly
.eqv EQV_NAME string
```

和define相同，编译时会将所有的EQV_NAME替换为string，常用来定义常量。
# 函数调用

在这里单独将函数的调用作为一章，因为他很难！而且很重要！

函数的特点如下：

* 函数是一个代码块，可以由指定语句调用，并且在执行完毕后返回调用语句
* 函数可以通过传参实现代码的复用
* 函数只能通过返回值等对函数外造成影响
* 函数里依然可以嵌套函数

对于函数调用，最简单的形式如下：

```assembly
# function call
jal funtion_name


# function
function_name:
    <function-content>
    jr $ra
```

​	那么为什么我们选择了jal而不是j呢？因为jal在跳转的同时会将PC+4的值写入ra寄存器当中，这样当我们执行jr指令的时候，就正好会跳转到函数调用指令的下一条！非常的方便快捷。

## 代码的复用

​	在使用的过程中，为了能顺利的多次复用代码，我们必须使用固定的形参寄存器`$a0,$a1,$a2,$a3`来存储传入函数的形参，同样的，我们对于函数的返回值也需要规定固定的寄存器。

例如翻译一个这样的函数

```c
int sum(int a, int b)
{
    int tmp = a + b;
    return tmp;
}

int main()
{
    int a = 2;
    int b = 3;
    int c = 4;
    int d = 5;

    int sum1 = sum(a, b);
    printf("%d", sum1);
    sum2 = sum(c, d);
    printf("%d", sum2);
    return 0;
}
```

它所对应的汇编语言应该像这样：

```assembly
.macro end
    li      $v0, 10
    syscall
.end_macro

.macro printStr(%str)
    la      $a0, %str
    li      $v0, 4
    syscall
.end_macro

.data
space: .asciiz " "

.text
li      $s0, 2
li      $s1, 3
li      $s2, 4
li      $s3, 5

move    $a0, $s0 #传参
move    $a1, $s1
jal     sum
move    $s4, $v0 #获得返回值

li      $v0, 1
move    $a0, $s4
syscall

printStr(space)

move    $a0, $s2
move    $a1, $s3
jal     sum
move    $s5, $v0

li      $v0, 1
move    $a0, $s5
syscall

end

sum:
#传参过程
move    $t0, $a0
move    $t1, $a1
#函数过程
add     $v0, $t0, $t1
jr      $ra
```

​	如果函数需要使用的参数小于4个，一般地，我们应当使用上述的4个寄存器，否则应该将多出的参数存入内存当中。并使用控制栈寄存器`$sp`，完成对内存的访问。

## 避免对外界产生影响	

​	为了避免对外界函数产生影响，在函数中我们可以使用临时寄存器，但是必须将临时寄存器的原值提前存入到内存当中。这个过程是一个入栈的过程，以方便我们在函数调用结束之后将这些数据逐一的存放回原来的寄存器当中。

​	例如这样的一段代码

```c
int sum(int a, int b)
{
    int tmp = a + b;
    return tmp;
}

int main()
{
    int a = 2;
    int b = 3;
    int c = 4;
    int sum1 = sum(a, b);
    int sum2 = sum(sum1, c);
    printf("%d", sum2);
    return 0;
}
```

​	翻译成汇编语言之后应该变成这样：

​	调用者维护栈：

```assembly
.macro end
    li      $v0, 10
    syscall
.end_macro

.text
li      $s0, 2
li      $s1, 3
li      $t0, 4

move    $a0, $s0 #传参
move    $a1, $s1
sw      $t0, 0($sp) #入栈
addi    $sp, $sp, -4
jal     sum
addi    $sp, $sp, 4 #出栈
lw      $t0, 0($sp) 
move    $s4, $v0 #获得返回值

move    $a0, $s4
move    $a1, $t0
sw      $t0, 0($sp) #入栈
addi    $sp, $sp, -4
jal     sum
addi    $sp, $sp, 4 #出栈
lw      $t0, 0($sp)
move    $s5, $v0

li      $v0, 1
move    $a0, $s5
syscall

end

sum:
#传参过程
move    $t0, $a0
move    $t1, $a1
#函数过程
add     $v0 $t0, $t1
jr      $ra
```

​	被调用者维护栈：

```assembly
.macro end
    li      $v0, 10
    syscall
.end_macro

.text
li      $s0, 2
li      $s1, 3
li      $t0, 4

move    $a0, $s0 #传参
move    $a1, $s1
jal     sum
move    $s4, $v0 #获得返回值

move    $a0, $s4
move    $a1, $t0
jal     sum
move    $s5, $v0

li      $v0, 1
move    $a0, $s5
syscall

end

sum:
#入栈过程
sw      $t0, 0($sp)
addi    $sp, $sp, -4
#传参过程
move    $t0, $a0
move    $t1, $a1
#函数过程
add     $v0 $t0, $t1
#出栈过程
addi    $sp, $sp, 4
lw      $t0, 0($sp)
#return
jr      $ra
```

## 嵌套函数调用

​	不难发现，上述的努力都是由于寄存器数量数有限的，而我们在多次代码复用的过程中不可避免的需要多次重复使用这些寄存器。因此，需要通过频繁的出入栈来将寄存器中的数据存储到内存中，以避免在复用的过程中造成数据的丢失。

​	那么很容易想到，一旦出现了函数嵌套的情况，我们的`$ra`寄存器就将供不应求，因此我们依然需要通过内存来暂时存储函数的返回值。

​	例如这样的一个程序：

```c
int sum(int a, int b)
{
    return a + b;
}
int cal(int a, int b)
{
    return a - sum(b, a);
}

int main()
{
    int a = 2;
    int b = 3;
    int ans = cal(2, 3);
    printf("%d", ans);
}
```

翻译之后应该变成这样：

```assembly
.macro end
    li $v0, 10
    syscall
.end_macro

.text
li      $s0, 2
li      $s1, 3

move    $a0, $s0
move    $a1, $s1
jal     cal
move    $s5, $v0

li      $v0, 1
move    $a0, $s5
syscall

end

sum:
#将 $t0 和 $t1 入栈
sw      $t0, 0($sp)
addi    $sp, $sp, -4
sw      $t1, 0($sp)
addi    $sp, $sp, -4
#传参过程
move    $t0, $a0
move    $t1, $a1
#函数过程
add     $v0 $t0, $t1
#将 $t0 和 $t1 出栈
addi    $sp, $sp, 4
lw      $t1, 0($sp) 
addi    $sp, $sp, 4
lw      $t0, 0($sp) 
#return
jr      $ra

cal:
#将 $ra 入栈
sw      $ra, 0($sp)
addi    $sp, $sp, -4
#调用者维护
#传参过程
move    $t0, $a0
move    $t1, $a1
#调用 sum 的过程
move    $a0, $t1
move    $a1, $t0
jal     sum
move    $t2, $v0
#运算a-sum(b, a)
sub     $v0, $t0, $t2
#将ra出栈
addi    $sp, $sp, 4
lw      $ra, 0($sp) 
#return
jr      $ra
```

## 递归函数调用

递归函数的本质就是一个**在函数体内部调用自身的嵌套函数**。

例如实现一个阶乘程序，C语言的递归实现方式如下：

```C
#include <stdio.h>

int factorial(int n)
{
    if (n == 1)
    {
        return 1;
    }
    else
    {
        return n * factorial(n - 1);
    }
}

int main()
{
    printf("%d\n", factorial(5));

    return 0;
}
```

汇编语言的版本如下：

```assembly
.macro end
    li      $v0, 10
    syscall
.end_macro

.macro getInt(%des)
    li      $v0, 5
    syscall
    move    %des, $v0
.end_macro

.macro printInt(%src)
    move    $a0, %src
    li      $v0, 1
    syscall
.end_macro

.macro push(%src)
    sw      %src, 0($sp)
    subi    $sp, $sp, 4
.end_macro

.macro pop(%des)
    addi    $sp, $sp, 4
    lw      %des, 0($sp) 
.end_macro

.text
main:
    getInt($s0)

    move    $a0, $s0
    jal     factorial
    move    $s1, $v0

    printInt($s1)
    end

factorial:
    # 入栈
    push($ra)
    # 调用者维护（调用自身）
    push($t0)
    # 传参
    move    $t0, $a0
    #函数过程
    bne     $t0, 1, else
    # 基准情况
    if:
        li      $v0, 1
        j       if_end  
    # 递归情况  
    else:
        subi    $t1, $t0, 1
        move    $a0, $t1
        jal     factorial
        mult    $t0, $v0
        mflo    $v0
    if_end:
    # 出栈
    pop($t0)
    pop($ra)
    # 对应的出栈
    # 返回
    jr      $ra
```

大部分的函数都需要遵守入栈-传参-出栈-返回这四个阶段。




