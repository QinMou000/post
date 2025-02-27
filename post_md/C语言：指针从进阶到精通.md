# C语言：指针从进阶到精通

# 1. 字符指针变量

字符指针变量

在指针的类型中我们知道有⼀种指针类型为字符指针 char*，我们一般这样使用：

```cpp
int main()
{

	char* cp1 = NULL;
	char str = "hello world";
	cp1 = &str;
	/*********************/
	const char* cp2 = "hello world";

	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 这里的两段代码很多人以为是吧字符串hello world放在了cp里面，但是事实是将字符串首字母的地址放在cp里面。

《🗡指offer》里面有一道题：

```cpp
#include <stdio.h>
int main()
{
	char str1[] = "hello bit.";
	char str2[] = "hello bit.";
	const char* str3 = "hello bit.";
	const char* str4 = "hello bit.";
	if (str1 == str2)
		printf("str1 and str2 are same\n");
	else
		printf("str1 and str2 are not same\n");
	if (str3 == str4)
		printf("str3 and str4 are same\n");
	else
		printf("str3 and str4 are not same\n");
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这⾥str3和str4指向的是⼀个同⼀个常量字符串。C/C++会把常量字符串存储到单独的⼀个内存区域，当⼏个指针指向同⼀个字符串的时候，他们实际会指向同⼀块内存。但是⽤相同的常量字符串去初始化不同的数组的时候就会开辟出不同的内存块。所以str1和str2不同，str3和str4相同。

# 2. 数组指针变量

## 数组指针变量是什么？

之前我们学习了指针数组，指针数组是⼀种数组，数组中存放的是地址（指针）。

数组指针变量是指针变量？还是数组？
 答案是：**指针变量。**

我们已经熟悉：
 • 整形指针变量： int * pint; 存放的是整形变量的地址，能够指向整形数据的指针。
 • 浮点型指针变量： float * pf; 存放浮点型变量的地址，能够指向浮点型数据的指针。
 那数组指针变量应该是：存放的应该是数组的地址，能够指向数组的指针变量。

```cpp
int *p1[10];
int (*p2)[10];
//哪个是数组指针变量？
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

答案是第二个

解释：p先和*结合，说明p是⼀个指针变量变量，然后指着指向的是⼀个⼤⼩为10个整型的数组。所以p是⼀个指针，指向⼀个数组，叫数组指针。
 这⾥要注意：[]的优先级要⾼于*号的，所以必须加上（）来保证p先和*结合。

## 数组指针变量怎么初始化

数组指针变量是⽤来存放数组地址的，那怎么获得数组的地址呢？就是我们之前学习的 &数组名。

```cpp
int arr[10] = {0};
&arr;//得到的就是数组的地址
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如果要存放个数组的地址，就得存放在数组指针变量中，如下：

```cpp
int(*p)[10] = &arr;
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 [10]代表指向数组的元素个数

 p是数组指针变量名

int为p指向的数组的元素类型

# 3. ⼆维数组传参的本质

## ⼆维数组传参的本质

有了数组指针的理解，我们就能够讲⼀下⼆维数组传参的本质了。
 过去我们有⼀个⼆维数组的需要传参给⼀个函数的时候，我们是这样写的：

```cpp
#include <stdio.h>
void test(int a[3][5], int r, int c)
{
	int i = 0;
	int j = 0;

	for(i = 0; i < r; i++)
	{
		for (j = 0; j < c; j++)
		{
			printf("%d ", a[i][j]);
		}
		printf("\n");
	}
}
int main()
{
	int arr[3][5] = { {1,2,3,4,5}, {2,3,4,5,6},{3,4,5,6,7} };
	test(arr, 3, 5);
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 这⾥实参是⼆维数组，形参也写成⼆维数组的形式，那还有什么其他的写法吗？
 ⾸先我们再次理解⼀下⼆维数组，⼆维数组起始可以看做是每个元素是⼀维数组的数组，也就是⼆维
 数组的每个元素是⼀个⼀维数组。那么⼆维数组的⾸元素就是第⼀⾏，是个⼀维数组。
 如下图：

![img](https://raw.githubusercontent.com/QinMou000/pic/main/a501220c2465a65b4d9613eed2443726.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑
 所以，根据数组名是数组⾸元素的地址这个规则，⼆维数组的数组名表⽰的就是第⼀⾏的地址，是⼀维数组的地址。根据上⾯的例⼦，第⼀⾏的⼀维数组的类型就是 int [5] ，所以第⼀⾏的地址的类
 型就是数组指针类型 int(*)[5] 。那就意味着⼆维数组传参本质上也是传递了地址，传递的是第⼀
 ⾏这个⼀维数组的地址，那么形参也是可以写成指针形式的。如下：

```cpp
#include <stdio.h>
void test(int(*p)[5], int r, int c)
{
	int i = 0;
	int j = 0;
	for (i = 0; i < r; i++)
	{
		for (j = 0; j < c; j++)
		{
			printf("%d ", *(*(p + i) + j));
		}
		printf("\n");
	}
}
int main()
{
	int arr[3][5] = { {1,2,3,4,5}, {2,3,4,5,6},{3,4,5,6,7} };
	test(arr, 3, 5);
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 4. 函数指针变量

## 函数指针变量的创建

根据前⾯学习整型指针，数组指针的时候，我们的类⽐关系，我们不难得出结论：
 函数指针变量应该是⽤来存放函数地址的，未来通过地址能够调⽤函数的。
 那么函数是否有地址呢？
 我们做个测试：

```cpp
#include <stdio.h>
void test()
{
    printf("hehe\n");
}
int main()
{
    printf("test: %p\n", test);
    printf("&test: %p\n", &test);
    return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

输出结果：

![img](https://raw.githubusercontent.com/QinMou000/pic/main/64ec0d60f8c2d1828419ef58615a7dde.jpeg)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

 确实打印出来了地址，所以函数是有地址的，函数名就是函数的地址，当然也可以通过 &函数名 的⽅式获得函数的地址。

如果我们要将函数的地址存放起来，就得创建函数指针变量咯，函数指针变量的写法其实和数组指针⾮常类似。如下：

```cpp
void test()
{
    printf("hehe\n");
}

void (*pf1)() = &test;
void (*pf2)()= test;

int Add(int x, int y)
{
    return x+y;
}

int(*pf3)(int, int) = Add;
int(*pf3)(int x, int y) = &Add;//x和y写上或者省略都是可以的
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

函数指针类型解析：

```cpp
int (*p)(int,int);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

第一个int是p指向的函数的返回类型。

p是函数指针变量。

后面两个int是指这个函数两个int类型的参数。

## 函数指针变量的使用

可以使用函数指针来调用其指向的函数

```cpp
#include <stdio.h>
int Add(int x, int y)
{
    return x+y;
}
int main()
{
    int(*pf3)(int, int) = Add;
    printf("%d\n", (*pf3)(2, 3));
    printf("%d\n", pf3(3, 5));
    return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 这里有《C陷阱和缺陷》里面的两句代码，我们来解析一下 

### 代码Ⅰ

![img](https://raw.githubusercontent.com/QinMou000/pic/main/142c1ff4acb1f3ed7850228ead5dc71c.jpeg)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

```cpp
(*(void (*)())0)();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这里首先将整型 0 强制类型转化为void(*)()类型（这是一个函数指针，指向的函数返回值为void，参数也为void）。

然后外面有一个解引用符号*将里面的函数指针解引用，由于这个函数无参，那么后面的括号里面就没有放参数。

最终这句代码的意思就是调用0x00000000地址处的函数。

当然，0x00000000地址处肯定是没有函数的。

### 代码Ⅱ

```cpp
void (*signal(int , void(*)(int)))(int);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 这里我们可以看到有一个名为signal的函数，它有两个参数一个为int，另一个是一个void(*)(int)类型的函数指针

一个函数除了名字，参数还缺一个返回类型。将中间的去掉：void (*……)(int) 这居然也是一个函数指针！说明signal函数的返回类型是一个void(*)(int)类型的函数指针。

这是一个函数声明。

**也就是说这个函数接收一个整型和一个函数的地址，并返回另一个函数的地址**

## typedef关键字

typedef是⽤来类型重命名的，可以将复杂的类型，简单化。
 ⽐如，你觉得 unsigned int 写起来不⽅便，如果能写成 uint 就⽅便多了，那么我们可以使⽤：

```cpp
typedef unsigned int uint;
//将unsigned int 重命名为uint
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如果是指针类型，能否重命名呢？其实也是可以的，⽐如，将 int* 重命名为 ptr_t ,这样写：

```cpp
typedef int* ptr_t;
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

但是对于数组指针和函数指针稍微有点区别：

⽐如我们有数组指针类型 int(*)[5] ,需要重命名为 parr_t ，那可以这样写：

```cpp
typedef int(*parr_t)[5]; //新的类型名必须在*的右边
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

函数指针类型的重命名也是⼀样的，⽐如，将 void(*)(int) 类型重命名为 pf_t ,就可以这样写：

```cpp
typedef void(*pfun_t)(int);//新的类型名必须在*的右边
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

那么要简化代码Ⅱ，可以这样写：

```cpp
typedef void(*pfun_t)(int);
pfun_t signal(int, pfun_t);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 5. 函数指针数组

数组是⼀个存放相同类型数据的存储空间，我们已经学习了指针数组，
 ⽐如：

```cpp
int *arr[10];
//数组的每个元素是int*
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

那要把函数的地址存到⼀个数组中，那这个数组就叫函数指针数组，那函数指针的数组如何定义呢？

```cpp
int (*parr1[3])();
int *parr2[3]();
int (*)() parr3[3];
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**答案是：parr1**
 parr1 先和 [] 结合，说明parr1是数组，数组的内容是什么呢？
 就是 int (*)() 类型的函数指针。

# 6. 转移表

这里可以参考我的这两篇bolg，第一个是用普通函数调用实现的计算器，第二个是用的转移表（将每个函数的地址放进一个函数指针数组里面）写的计算器，大大减少了重复的代码。

[基于C语言写的计算器-CSDN博客](https://blog.csdn.net/2301_80194476/article/details/136063410?spm=1001.2014.3001.5501)

[基于C语言写的计算器（2）-CSDN博客](https://blog.csdn.net/2301_80194476/article/details/136073335?spm=1001.2014.3001.5501)

**本期博客到这里就结束了，如果有什么错误，欢迎指出，如果对你有帮助，请点个赞，谢谢！**