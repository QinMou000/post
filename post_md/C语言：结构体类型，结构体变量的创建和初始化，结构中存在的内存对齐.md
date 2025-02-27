# C语言：结构体类型，结构体变量的创建和初始化，结构中存在的内存对齐

# 1.语言结构体类型

结构是⼀些值的集合，这些值称为成员变量。结构的每个成员可以是不同类型的变量。

## 声明

```cpp
struct tag
{
    member-list;
}variable-list;
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

例如声明一本书：

```cpp
struct Book
{
	char name[20]; //书名
	char author[20]; //作者
	int price; //定价
    int id[12] //编号
};//别忘了分号！！！
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 创建和初始化 

```cpp
#include <stdio.h>
struct Stu
{
    char name[20];//名字
    int age;//年龄
    char sex[5];//性别
    char id[20];//学号
};
int main()
{
    //按照结构体成员的顺序初始化

    struct Stu s = { "张三", 20, "男", "20230818001" };

    printf("name: %s\n", s.name);
    printf("age : %d\n", s.age);
    printf("sex : %s\n", s.sex);
    printf("id : %s\n", s.id);
    //按照指定的顺序初始化

    struct Stu s2 = { .age = 18, .name = "lisi", .id = "20230818002", .sex = "⼥};

    printf("name: %s\n", s2.name);
    printf("age : %d\n", s2.age);
    printf("sex : %s\n", s2.sex);
    printf("id : %s\n", s2.id);
    return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 结构体的特殊声明

```cpp
//匿名结构体类型
struct
{
    int a;
    char b;
    float c;
}x;

struct
{
    int a;
    char b;
    float c;
}a[20], *p;
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

在上面的代码的基础上 *p = &x; 是不合法的。

警告：
 编译器会把上⾯的两个声明当成完全不同的两个类型，所以是⾮法的。
 匿名的结构体类型，如果没有对结构体类型重命名的话，基本上只能使⽤⼀次。

## 结构体的自引用

在结构中是否可以包含一个该结构体本身的类型呢？

比如在链表中：

```cpp
struct Node
{
    int data;
    struct Node next;
};
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这样定义是不行的，因为⼀个结构体中再包含⼀个同类型的结构体变量，这样结构体变量的⼤
 ⼩就会⽆穷的⼤，是不合理的。
 正确的⾃引⽤⽅式：

```cpp
struct Node
{
    int data;
    struct Node* next;
};
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

在结构体⾃引⽤使⽤的过程中，夹杂了 typedef 对匿名结构体类型重命名，也容易引⼊问题，看看
 下⾯的代码，可⾏吗？

```cpp
typedef struct
{
    int data;
    Node* next;
}Node;
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)


 答案是不⾏的，因为Node是对前⾯的匿名结构体类型的重命名产⽣的，但是在匿名结构体内部提前使⽤Node类型来创建成员变量，这是不⾏的。

解决⽅案如下：定义结构体不要使⽤匿名结构体了

```cpp
typedef struct Node
{
    int data;
    struct Node* next;
}Node;
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 2.结构体内存对⻬

##  对⻬规则

⾸先得掌握结构体的对⻬规则：
 1.结构体的第⼀个成员对⻬到和结构体变量起始位置偏移量为0的地址处
 2.其他成员变量要对⻬到某个数字（对⻬数）的整数倍的地址处。
 对⻬数 = 编译器默认的⼀个对⻬数与该成员变量⼤⼩的较⼩值。

VS 中默认的值为 8 ，Linux中gcc没有默认对⻬数，对⻬数就是成员⾃⾝的⼤⼩
 3.结构体总⼤⼩为最⼤对⻬数（结构体中每个成员变量都有⼀个对⻬数，所有对⻬数中最⼤的）的
 整数倍。
 4.如果嵌套了结构体的情况，嵌套的结构体成员对⻬到⾃⼰的成员中最⼤对⻬数的整数倍处，结构
 体的整体⼤⼩就是所有最⼤对⻬数（含嵌套结构体中成员的对⻬数）的整数倍。

下面来有几道例题：（后附参考答案）

```cpp
//练习1
struct S1
{
    char c1;
    int i;
    char c2;
};
    printf("%d\n", sizeof(struct S1));// 12
//练习2
struct S2
{
    char c1;
    char c2;
    int i;
};
    printf("%d\n", sizeof(struct S2));// 8
//练习3
struct S3
{
    double d;
    char c;
    int i;
};
    printf("%d\n", sizeof(struct S3));// 16
//练习4-结构体嵌套问题
struct S4
{
    char c1;
    struct S3 s3;
    double d;
};
    printf("%d\n", sizeof(struct S4));// 32
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 为什么存在内存对⻬?

1. 平台原因（移植原因）

 不是所有的硬件平台都能访问任意地址上的任意数据的；某些硬件平台只能在某些地址处取某些特定类型的数据，否则抛出硬件异常。

2. 性能原因：

 数据结构(尤其是栈)应该尽可能地在⾃然边界上对⻬。原因在于，为了访问未对⻬的内存，处理器需要作两次内存访问；⽽对⻬的内存访问仅需要⼀次访问。假设⼀个处理器总是从内存中取8个字节，则地址必须是8的倍数。如果我们能保证将所有的double类型的数据的地址都对⻬成8的倍数，那么就可以⽤⼀个内存操作来读或者写值了。否则，我们可能需要执⾏两次内存访问，因为对象可能被分放在两个8字节内存块中。
 **总体来说：结构体的内存对⻬是拿空间来换取时间的做法。**

那在设计结构体的时候，我们既要满⾜对⻬，⼜要节省空间，如何做到： **让占⽤空间⼩的成员尽量集中在⼀起**

```cpp
//例如：
struct S1
{
    char c1;
    int i;
    char c2;
};
struct S2
{
    char c1;
    char c2;
    int i;
};
//S1和S2类型的成员⼀模⼀样，但是S1和S2所占空间的⼤⼩有了⼀些区别。
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 修改默认对⻬数

\#pragma 这个预处理指令，可以改变编译器的默认对⻬数。

```cpp
#include <stdio.h>
#pragma pack(1)//设置默认对⻬数为1
struct S
{
    char c1;
    int i;
    char c2;
};
#pragma pack()//取消设置的对⻬数，还原为默认
int main()
{
    printf("%d\n", sizeof(struct S));//输出 6
    return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

结构体在对⻬⽅式不合适的时候，我们可以⾃⼰更改默认对⻬数

# 3.结构体传参

```cpp
struct S
{
    int data[1000];
    int num;
};
struct S s = {{1,2,3,4}, 1000};
//结构体传参
void print1(struct S s)
{
    printf("%d\n", s.num);
}
//结构体地址传参
void print2(struct S* ps)
{
    printf("%d\n", ps->num);
}
int main()
{
    print1(s); //传结构体
    print2(&s); //传地址（优）
    return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

为什么我们说结构传参的时候传地址比较好呢？在传结构体的时候相当于将原结构拷贝了一次，重新放在了一个地址上，占用了额外的空间。

那么，当我们传地址的时候为了防止原结构被修改，可以加上const修饰。

# 4.结构体实现位段

## 什么是位段？

位段的声明和结构是类似的，有两个不同：

1. 位段的成员必须是 int、unsigned int 或signed int ，在C99中位段成员的类型也可以

 选择其他类型。

2. 位段的成员名后边有⼀个冒号和⼀个数字

```cpp
#include<stdio.h>
struct A
{
    int _a : 2;
    int _b : 5;
    int _c : 10;
    int _d : 30;
};

int main()
{
    printf("%zd\n",sizeof(struct A));//输出 8
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 位段的内存分配

1. 位段的成员可以是 int  unsigned int  signed int 或者是 char 等类型
2. 位段的空间上是按照需要以4个字节（ int ）或者1个字节（ char ）的⽅式来开辟的。
3. 位段涉及很多不确定因素，位段是不跨平台的，注重可移植的程序应该避免使⽤位段。

```cpp
//⼀个例⼦
struct S
{
    char a:3;
    char b:4;
    char c:5;
    char d:4;
};
struct S s = {0};
    s.a = 10;
    s.b = 12;
    s.c = 3;
    s.d = 4;
//空间是如何开辟的？
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## ![img](https://raw.githubusercontent.com/QinMou000/pic/main/34192674724fa75542432a05d2adb083.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑  位段的跨平台问题

1. int 位段被当成有符号数还是⽆符号数是不确定的。
2. 位段中最⼤位的数⽬不能确定。（16位机器最⼤16，32位机器最⼤32，写成27，在16位机器会

 出问题。

3. 位段中的成员在内存中从左向右分配，还是从右向左分配标准尚未定义。
4. 当⼀个结构包含两个位段，第⼆个位段成员⽐较⼤，⽆法容纳于第⼀个位段剩余的位时，是舍弃

 剩余的位还是利⽤，这是不确定的。
 **总结：
 跟结构相⽐，位段可以达到同样的效果，并且可以很好的节省空间，但是有跨平台的问题存在。**

## 位段使⽤的注意事项

位段的⼏个成员共有同⼀个字节，这样有些成员的起始位置并不是某个字节的起始位置，那么这些位置处是没有地址的。内存中每个字节分配⼀个地址，⼀个字节内部的bit位是没有地址的。
 **所以不能对位段的成员使⽤&操作符**，这样就不能使⽤scanf直接给位段的成员输⼊值，只能是先输⼊放在⼀个变量中，然后赋值给位段的成员。

```cpp
struct A
{
    int _a : 2;
    int _b : 5;
    int _c : 10;
    int _d : 30;
};
int main()
{
    struct A sa = { 0 };
    /*
        scanf("%d", &sa._b);//这是错误的
    */
    //正确的⽰范
    int b = 0;
    scanf("%d", &b);
    sa._b = b;
    return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**本期博客到这里就结束了，如果有什么错误，欢迎指出，如果对你有帮助，请点个赞，谢谢！**