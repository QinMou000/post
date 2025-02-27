# C语言：计算器（回调）

##  回调：

如果你把函数的指针(地址)作为参数传递给另一个函数，当这个指针被用来调用其所指向的函数时，被调用的函数就是回调函数。回调函数不是由该函数的实现方直接调用，而是在特定的事件或条件发生时由另外的一方调用的。

```cpp
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>

void menu()
{
	printf("****************************\n");
	printf("***   1—>'+'  2—>'-'   ***\n");
	printf("***   3—>'*'  4—>'/'   ***\n");
	printf("***   5—>'>>' 6—>'<<'  ***\n");
	printf("***   7—>'&'  8—>'|'   ***\n");
	printf("***        9—>'^'       ***\n");
	printf("****************************\n");
	printf("***        0.exit       ****\n");
	printf("****************************\n");
}

int add(int x, int y)
{
	return x + y;
}

int sub(int x, int y)
{
	return x - y;
}

int mul(int x, int y)
{
	return x * y;
}

int divs(int x, int y)
{
	return x / y;
}

int Shift_right_bit(int x, int y)
{
	return x >>= y;
}

int Shift_left_bit(int x, int y)
{
	return x <<= y;
}

int Bitwise_with(int x, int y)
{
	return x & y;
}

int Bitwise_or(int x, int y)
{
	return x | y;
}

int Bitwise_XOR(int x, int y)
{
	return x ^ y;
}

void Print_Binary(unsigned int x)
{
	if (x > 1)
	{
		Print_Binary(x >> 1);
	}
	putchar((x & 1) ? '1' : '0');
}

// void calu(int (*pf)(int, int))
//{
//	 int x = 0;
//	 int y = 0;
//	 int z = 0;
//	 printf("请输入操作数");
//	 scanf("%d %d", &x, &y);
//	 z = pf(x, y);
//	 printf("十进制结果:%d\n", z);
//	 printf("二进制结果:\n");
//	 Print_Binary(z);
//	 printf("\n");
//}

int main()
{
	int input = 0;
	int (*pf[10])(int, int) = { NULL,add ,sub,mul,divs,Shift_right_bit,Shift_left_bit,Bitwise_with,Bitwise_or,Bitwise_XOR};
	do
	{
		menu();
		printf("请选择：");
		scanf("%d",&input);
		if (input >= 1 && input <= 9)
		{
			int x = 0;
			int y = 0;
			int z = 0;
			printf("请输入操作数：");
			scanf("%d %d",&x,&y);
			z = pf[input](x,y);
			printf("十进制结果：%d\n",z);
			printf("二进制结果：");
			Print_Binary(z);
			printf("\n");
		}
		else if(input==0)
		{
			printf("《退出》");
		}
		else
		{
			printf("输入错误，重新输入");
		}

	} while (input);

	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**最后，如果有什么错误，欢迎指出，如果有帮助，点个赞，谢谢。**