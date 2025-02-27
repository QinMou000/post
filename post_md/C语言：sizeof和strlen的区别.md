# C语言：sizeof和strlen的区别

# of:

![img](https://raw.githubusercontent.com/QinMou000/pic/main/bc7fdcdda99407ac73c18eba32af6e73.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

**首先我要在此声明sizeof不是函数！不是函数！不是函数！而是一个操作符！（看到operator了吗？）**

```cpp
#include<stdio.h>

int main()
{
	int a = 1;//sizeof不会读取数据，只会计算所占内存空间的大小（单位是字节）
	printf("%zd\n", sizeof(a));  //4  
	printf("%zd\n", sizeof a );  //4  看吧，就算没有()只需要空格sizeof也能正确被执行。
	printf("%zd\n", sizeof 1);   //4  这里体现了sizeof就不是函数
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这里要强调的是如果sizeof操作对象是一个表达式，则被操作的表达式里的计算不会被执行。

```cpp
int main()
{
	int a = 3;
	int b = 5;
	printf("%zd\n", sizeof(a += b));  //4
	printf("%d\n",a);//因为a+=b不会计算那么a的值还是3。
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# strlen：

![img](https://raw.githubusercontent.com/QinMou000/pic/main/9c034b7d87aff3479618ae4c46fb5104.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

strlen是C语言中的库函数<string.h>。原型是

```cpp
size_t strlen(const char* str)
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

看的出来strlen是专门求字符串长度的，统计的是从此地址开始往后\0之前字符串的字符个数，那么它就有可能越界查找。如下所示：

```cpp
int main()
{
	char a[] = "abc";
	char b[] = { 'a','b','c' };
	printf("%zd\n", strlen(a));//3       a在内存中是： a b c \0 这样的strlen读取了\0前面的字符个数
	printf("%zd\n", strlen(b));//随机值  b在内存中是： a b c 随机值…… 这样的，所以我们不知道后面什么地方会出现\0
	printf("%zd\n", sizeof(a));//4       a包括\0在内占了四个字节
	printf("%zd\n", sizeof(b));//3       b只包括了abc三个字符所以占三个字节
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 总结区别：

![img](https://i-blog.csdnimg.cn/blog_migrate/f6c5a110ed95ae102526f4794777907b.jpeg)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑