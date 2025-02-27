

#  LeetCode_[NOIP2015]金币

# 题目：

国王将金币作为工资，发放给忠诚的骑士。第一天，骑士收到一枚金币；之后两天（第二天和第三天），每天收到两枚金币；之后三天（第四、五、六天），每天收到三枚金币；之后四天（第七、八、九、十天），每天收到四枚金币……；这种工资发放模式会一直这样延续下去：当连续N天每天收到N枚金币后，骑士会在之后的连续N+1天里，每天收到N+1枚金币。

请计算在前K天里，骑士一共获得了多少金币。

------

# 解析：

这道题最先我是想用递归写的，but失败了。各位大佬如果会用递归写的能教教我吗？**这是我的废代码：**

```cpp
/*废代码*/


int fun(int k)//为什么我写不出递归啊！！！！！！！！！！！！！
{
	int sum = 0;
	if (k - x < 0)
	{
		x++;
		sum += k * x;
	}
	else
	{	
		x++;
		sum += fun(k - x) + x * x;
	}
	return sum;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

------

这是我写了很久的递归后终于放弃了之后**写的正确的代码：**

```cpp
#include <stdio.h>
int x = 1;
int fun(int k)
{
	int sum = 0;
	while (k>x)
	{
		sum += x * x;
		k -= x;
		x++;
	}
	sum += k * x;
	return sum;
}
int main()
{
	int k = 0;
	scanf("%d",&k);
	int ret = fun(k);
	printf("%d\n", ret);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

最后我写这篇博客是为了告诫我自己不要什么都想着递归，有写问题用一般的解法不一定比递归差而且递归难写不说，空间复杂度还大。

------

**如果有什么错误，欢迎指出，如果有帮助，点个赞，谢谢。**