# C语言：变态面试题交换两个整数

# 题目：

C语言中在不创建临时变量的情况下，实现两个整数的交换。

## 分析：

对于这个题目一共有三种方法解决。第一种就是创建一个临时变量作为媒介来交换，想必大家都很熟悉了，这里就不过多叙述了，直接上代码：

（这种方式虽然创建了一个新的变量，并不符合题目要求，但却是我们写代码最常用的方式。）

### 第一种：

```cpp
int main()
{
	int x = 2;
	int y = 3;
	printf("%d %d\n", x, y);
	int medium = x;
	x = y;
	y = medium;
	printf("%d %d\n", x, y);
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 第二种：

```cpp
int main()
{
	int x = 2;
	int y = 3;
	printf("%d %d\n", x, y);
	x = x + y;
	y = x - y;
	x = x - y;
	printf("%d %d\n", x, y);
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

对于第二种，就有点难理解了，但是仔细去琢磨，还是有迹可循的，在两个printf语句中间的第一行代码，将x和y的和赋值给了x。

第二行的x-y是不是就相当于用x，y的和减去y，那么结果不就是原来x的值吗？所以这行代码就将x原来的值赋给了y。

第三行代码中的x还是原来下x，y的和，而y就变成了x原来的值，所以这行代码是不是就相当于用下x，y的和减去x呢？这样就把y的初始值赋值给了x。

总的来说就是将x=x+y带入下面并注意y值的变化就行了（x=x+y就像一把钥匙）

肿么样是不是很神奇！！！

**but！你是否考虑过x+y已经超过了数据的最大存储值了呢？**

### 第三种：（你要是光看就能把这段代码看懂，算你厉害~）

```cpp
int main()
{
	int x = 2;
	int y = 3;
	printf("%d %d\n", x, y);
	x = x ^ y;
	y = x ^ y;
	x = x ^ y;
	printf("%d %d\n", x, y);
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**懵逼树下懵逼果，懵逼树前你和我。**

这是什么玩意儿？

要能看懂这段代码先要了解 ^ （按位异或）操作符。

**按位异或操作符的运算结果是对应操作数的二进制位相同为0，相异为1**

想了解更多关于按位异或，按位与，按位或操作符的相关知识请阅读：[C语言中运算符“^”，“&”，“|”简介_c语言^-CSDN博客](https://blog.csdn.net/weixin_74769483/article/details/133490541?ops_request_misc=%7B%22request%5Fid%22%3A%22170616093716800185870559%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=170616093716800185870559&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-133490541-null-null.142^v99^control&utm_term=^&spm=1018.2226.3001.4187)

这里我们只需要知道a^a=0,a^0=a就够了。

这里也和第二种方法一样，将x=x^y带入下面并注意y值的变化。

第二行：y=x^y^y=x^0=x

第三行：x=x^y=x^y^x=y^0=y

这样就交换了x，y的值。

**如果有什么错误，欢迎指出，如果有帮助，点个赞，谢谢。**