# LeetCode_回文对称数

# 题目：

# 回文数是正着读与倒着读都一样的数，比如1221，343是回文数，433不是回文数。写一个函数判断i个数是不是回文数。

------

# 解析：

要判断一个数是不是回文数就把它倒过来看是否和原来的数相等。

```cpp
bool isPalindrome(int x)
{
	int tmp = 0;
	int num = x;
	if (x < 0)
	{
		return false;
	}
	while (num)
	{
		tmp = tmp * 10 + num % 10;
		num /= 10;
	}
	if (tmp == x)
	{
		return true;
	}
	else
	{
		return false;
	}
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

首先函数的返回类型是bool，传一个参数过去就行了，负数不是回文数这些就不过多叙述了，这段代码的关键点在：

while (num)
 {
     tmp = tmp * 10 + num % 10;
     num /= 10;
 }

这个循环很巧妙。每次num都被除10，而每次循环tmp都要乘10并且加上num模上10，当最后一次num/10=0，循环结束。

**OK提交！！！！**

------

![img](https://raw.githubusercontent.com/QinMou000/pic/main/f23e56058601862928687ff162a31ccf.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

 ？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？

仔细一看好家伙x的范围是-2的31次方减1到-2的31次方减1，int 类型满足不了它了那就用long

**过辣！！！！！**

![img](https://raw.githubusercontent.com/QinMou000/pic/main/2886fba735dd5e52362aa563d2df5f38.png)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

------

但是还不够简短啊，还有什么可以优化的地方呢？？（冥思苦想……）

哦~~~~~~~~~~~~~

返回的不是bool类型吗？那就不用if else了，直接return x==tmp；

```cpp
bool isPalindrome(int x)
{
	long tmp = 0;
	long num = x;
	if (x < 0)
		return false;
	while (num)
	{
		tmp = tmp * 10 + num % 10;
		num /= 10;
	}
	return x == tmp;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**过辣！！！！！**

![img](https://raw.githubusercontent.com/QinMou000/pic/main/6f286523208a5852f31c50f057e456e5.png)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

------

朋友们平时在刷题写代码的时候一定要去寻找最优解，不断的否定再改正，提升自己的思维。

**如果有什么错误，欢迎指出，如果有帮助，点个赞，谢谢。**