# LeetCode_1304.和为零的 N 个不同整数

#  题目：

![img](https://raw.githubusercontent.com/QinMou000/pic/main/48d696d0ab2ad228cb0cb6355e6e1b24.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

# 题解：

题目说让我们返回一个由n个各不相同的整数组成的数组，相加为0。

这里的比较好的办法就是类似于 1 2 3 0 -3 -2 -1这样对称的数组。既满足要求，又好实现。

先calloc出一个容量为n的整型数组，定义两个变量（也可以认为是指针）left和right分别指向数组的两头。再定义一个 i 为赋值变量，每次循环后i++。

将arr[left]和arr[right]分别赋上相反的数，将left++，right--。以left<=right为循环的终止条件。如果n为奇数，将数组中间的那个数赋值为0。

```cpp
int* sumZero(int n, int* returnSize)
{
	int* arr = (int*)calloc(n , (sizeof(int)));
	int left = 0;
	int right = n - 1;
	int i = 1;
	while (left <= right)
	{
		arr[left] = i;
		arr[right] = -i;
		if (left == right)
		{
			arr[left] = 0;
			break;
		}
		left++;
		right--;
		i++;
	}
	*returnSize = n;
	return arr;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 **本期博客到这里就结束了，如果有什么错误，欢迎指出，如果对你有帮助，请点个赞，谢谢！**