# LeetCode_丑数

# 题目：

![img](https://raw.githubusercontent.com/QinMou000/pic/main/b960015c7a04bac183ee715a23c2dc11.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

# 题解：

由题，我们知道丑数大于0，丑数都可以写成2*2*...*2*3*3...*3*5*5...*5，有了这个基础就很好写代码了。

用三个while循环将前面的2 3 5全部除掉如果这个数是丑数，最后n是等于1的，反之n不等于1。

```cpp
bool isUgly(int n){
    if(n<=0){
        return false;
    }
    if(n==1){
        return true;
    }
    while(n%2==0){
        n/=2;
    }
    while(n%3==0){
        n/=3;
    }
    while(n%5==0){
        n/=5;
    }
    return n==1;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

此外我们还可以写成递归：

```cpp
bool isUgly(int n)
{
	if (n == 1)
		return 1;
	if (n == 0)
		return 0;
	if (n % 2 == 0)
		return isUgly(n / 2);
	if (n % 3 == 0)
		return isUgly(n / 3);
	if (n % 5 == 0)
		return isUgly(n / 5);
    return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**本期博客到这里就结束了，如果有什么错误，欢迎指出，如果对你有帮助，请点个赞，谢谢！**