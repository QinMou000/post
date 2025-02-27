> # C语言：程序编译和链接
>
>  ![img](https://raw.githubusercontent.com/QinMou000/pic/main/6f2b920cd38b273e9349974209147fee.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑
>
> ✨✨所属专栏：[C语言](https://blog.csdn.net/2301_80194476/category_12649617.html)✨✨
>
> ✨✨作者主页：[嶔某](https://blog.csdn.net/2301_80194476?spm=1000.2115.3001.5343)✨✨

在ANSIC的任何⼀种实现中，存在两个不同的环境。
 第1种是翻译环境，在这个环境中源代码被转换为可执⾏的机器指令（⼆进制指令）。
 第2种是执⾏环境，它⽤于实际执⾏代码

![img](https://raw.githubusercontent.com/QinMou000/pic/main/4c2eb644e2ba416c4eed6d8e9d603f5a.jpeg)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

# 翻译环境

那翻译环境是怎么将源代码转换为可执⾏的机器指令的呢？这⾥我们就得展开开讲解⼀下翻译环境所做的事情。其实翻译环境是由编译和链接两个⼤的过程组成的，⽽编译⼜可以分解成：预处理（有些书也叫预编译）、编译、汇编三个过程。
 ⼀个C语⾔的项⽬中可能有多个 .c ⽂件⼀起构建，那多个 .c ⽂件如何⽣成可执⾏程序呢？
 • 多个.c⽂件单独经过编译器，编译处理⽣成对应的⽬标⽂件。
 • 注：在Windows环境下的⽬标⽂件的后缀是 .obj ，Linux环境下⽬标⽂件的后缀是 .o 
 • 多个⽬标⽂件和链接库⼀起经过链接器处理⽣成最终的可执⾏程序。
 • 链接库是指运⾏时库(它是⽀持程序运⾏的基本函数集合)或者第三⽅库
 如果再把编译器展开成3个过程，那就变成了下⾯的过程：

![img](https://raw.githubusercontent.com/QinMou000/pic/main/250a17c957d22922bf3700a72913ec02.jpeg)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

## 预处理（预编译）

在预处理阶段，源⽂件和头⽂件会被处理成为.i为后缀的⽂件。
 在 gcc 环境下想观察⼀下，对 test.c ⽂件预处理后的.i⽂件，命令如下：
 1 gcc -E test.c -o test.i
 预处理阶段主要处理那些源⽂件中#开始的预编译指令。⽐如：#include,#define，处理的规则如下：
 • 将所有的 #define 删除，并展开所有的宏定义。
 • 处理所有的条件编译指令，如： #if、#ifdef、#elif、#else、#endif 。
 • 处理#include预编译指令，将包含的头⽂件的内容插⼊到该预编译指令的位置。这个过程是递归进
 ⾏的，也就是说被包含的头⽂件也可能包含其他⽂件。
 • 删除所有的注释
 • 添加⾏号和⽂件名标识，⽅便后续编译器⽣成调试信息等。
 • 或保留所有的#pragma的编译器指令，编译器后续会使⽤。
 经过预处理后的.i⽂件中不再包含宏定义，因为宏已经被展开。并且包含的头⽂件都被插⼊到.i⽂件
 中。所以当我们⽆法知道宏定义或者头⽂件是否包含正确的时候，可以查看预处理后的.i⽂件来确认。

## 编译

编译过程就是将预处理后的⽂件进⾏⼀系列的：词法分析、语法分析、语义分析及优化，⽣成相应的汇编代码⽂件。
 编译过程的命令如下：

```cpp
gcc -S test.i -o test.s
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

对下⾯代码进⾏编译的时候，会怎么做呢？假设有下⾯的代码

```cpp
array[index] = (index+4)*(2+6)
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 词法分析：

将源代码程序被输⼊扫描器，扫描器的任务就是简单的进⾏词法分析，把代码中的字符分割成⼀系列的记号（关键字、标识符、字⾯量、特殊字符等）。
 上⾯程序进⾏词法分析后得到了16个记号：

| 记号  | 类型     |
| ----- | -------- |
| arry  | 标识符   |
| [     | 左方括号 |
| index | 标识符   |
| ]     | 右方括号 |
| =     | 赋值     |
| (     | 左圆括号 |
| index | 标识符   |
| +     | 加号     |
| 4     | 数字     |
| ）    | 右圆括号 |
| *     | 乘号     |
| (     | 左圆括号 |
| 2     | 数字     |
| +     | 加号     |
| 6     | 数字     |
| )     | 右圆括号 |

###   语法分析：

接下来语法分析器，将对扫描产⽣的记号进⾏语法分析，从⽽产⽣语法树。这些语法树是以表达式为节点的树。

![img](https://raw.githubusercontent.com/QinMou000/pic/main/b8afc81f7bb8a47609ff5568b3cecb52.jpeg)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

### 语义分析：

由语义分析器来完成语义分析，即对表达式的语法层⾯分析。编译器所能做的分析是语义的静态分
 析。静态语义分析通常包括声明和类型的匹配，类型的转换等。这个阶段会报告错误的语法信息

![img](https://raw.githubusercontent.com/QinMou000/pic/main/c976c8c6b01181ad35eea43187290aa9.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

## 汇编

汇编器是将汇编代码转转变成机器可执⾏的指令，每⼀个汇编语句⼏乎都对应⼀条机器指令。就是根据汇编指令和机器指令的对照表⼀⼀的进⾏翻译，也不做指令优化。
 汇编的命令如下：

```cpp
gcc -c test.s -o test.o
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 链接

链接是⼀个复杂的过程，链接的时候需要把⼀堆⽂件链接在⼀起才⽣成可执⾏程序。
 链接过程主要包括：地址和空间分配，符号决议和重定位等这些步骤。
 链接解决的是⼀个项⽬中多⽂件、多模块之间互相调⽤的问题。
 ⽐如：
 在⼀个C的项⽬中有2个.c⽂件（ test.c 和 add.c ），代码如下：

test.c

```cpp
#include <stdio.h>
//test.c
//声明外部函数
extern int Add(int x, int y);
//声明外部的全局变量
extern int g_val;
int main()
{
	int a = 10;
	int b = 20;
	int sum = Add(a, b);
	printf("%d\n", sum);
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

add.c

```cpp
int g_val = 2022;
int Add(int x, int y)
{
	return x + y;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

我们已经知道，每个源⽂件都是单独经过编译器处理⽣成对应的⽬标⽂件。
 test.c 经过编译器处理⽣成test.o 
 add.c 经过编译器处理⽣成add.o 
 我们在 test.c 的⽂件中使⽤了 add.c ⽂件中的 Add 函数和 g_val 变量。
 我们在 test.c ⽂件中每⼀次使⽤ Add 函数和 g_val 的时候必须确切的知道 Add 和 g_val 的地
 址，但是由于每个⽂件是单独编译的，在编译器编译 test.c 的时候并不知道 Add 函数和 g_val
 变量的地址，所以暂时把调⽤ Add 的指令的⽬标地址和 g_val 的地址搁置。等待最后链接的时候由链接器根据引⽤的符号 Add 在其他模块中查找 Add 函数的地址，然后将 test.c 中所有引⽤到
 Add 的指令重新修正，让他们的⽬标地址为真正的 Add 函数的地址，对于全局变量 g_val 也是类
 似的⽅法来修正地址。这个地址修正的过程也被叫做：重定位。
 前⾯我们⾮常简洁的讲解了⼀个C的程序是如何编译和链接，到最终⽣成可执⾏程序的过程，其实很多内部的细节⽆法展开讲解。⽐如：⽬标⽂件的格式elf，链接底层实现中的空间与地址分配，符号解析和重定位等，如果你有兴趣，可以看《程序的⾃我修养》⼀书来详细了解。

# 运⾏环境

1. 程序必须载⼊内存中。在有操作系统的环境中：⼀般这个由操作系统完成。在独⽴的环境中，程序的载⼊必须由⼿⼯安排，也可能是通过可执⾏代码置⼊只读内存来完成。
2. 程序的执⾏便开始。接着便调⽤main函数。
3. 开始执⾏程序代码。这个时候程序将使⽤⼀个运⾏时堆栈（stack），存储函数的局部变量和返回地址。程序同时也可以使⽤静态（static）内存，存储于静态内存中的变量在程序的整个执⾏过程⼀直保留他们的值。
4. 终⽌程序。正常终⽌main函数；也有可能是意外终⽌。

**本期博客到这里就结束了，如果有什么错误，欢迎指出，如果对你有帮助，请点个赞，谢谢！**