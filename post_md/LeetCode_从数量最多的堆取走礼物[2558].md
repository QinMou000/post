> # LeetCode_从数量最多的堆取走礼物[2558]
>
>  ![img](https://raw.githubusercontent.com/QinMou000/pic/main/6f2b920cd38b273e9349974209147fee.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑
>
> ✨✨所属专栏：[LeetCode刷题专栏](https://blog.csdn.net/2301_80194476/category_12596977.html?spm=1001.2014.3001.5482)✨✨
>
> ✨✨作者主页：[嶔某](https://blog.csdn.net/2301_80194476?spm=1000.2115.3001.5343)✨✨

# 题目：

![img](https://raw.githubusercontent.com/QinMou000/pic/main/879bdbf9e531e34a71d8e4fff330b872.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

# 示例：

![img](https://raw.githubusercontent.com/QinMou000/pic/main/6737fc9f922b00a1ae1c5e871622ba6a.jpeg)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

#  解析：

先说说我自己的解法。看到题目描述我就觉得要用递归，我们可以先来一个if语句界定终止递归的条件，就是k==0的时候。我们把这时候gifts指针指向的数组求和并返回。

之后就是想办法解决开平方根，和在哪个数上开平方根。我们可以用一个函数sqrt来实现，在VS中包含math.h头文件就可以使用此函数了。之后我们要得出数组中最大值的下标，定义一个max=0，用for循环遍历数组，如果数组里面的数大于max，将这个数赋给max、并且记录下标。for循环完成后我们就可以得到最大数的下标了。将其开平方并返回pickGifts(gifts, giftsSize, --k)；就行了。

```cpp
long long pickGifts(int* gifts, int giftsSize, int k)
{
	if (k == 0)
	{
		long long sum = 0;
		for (int i = 0; i < giftsSize; i++)
		{
			sum += *(gifts + i);
		}
		return sum;
	}
	else
	{
		long long pos;
		long long max = 0;
		for (int i = 0; i < giftsSize; i++)
		{
			if (*(gifts + i) > max)
			{
				max = *(gifts + i);
				pos = i;
			}
		}
		*(gifts + pos) = (long long)sqrt(*(gifts + pos));
		return pickGifts(gifts, giftsSize, --k);
	}
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

官方解法是用的堆，这里我还没学到，等我学到后面再来更新把。

 **本期博客到这里就结束了，如果有什么错误，欢迎指出，如果对你有帮助，请点个赞，谢谢！**