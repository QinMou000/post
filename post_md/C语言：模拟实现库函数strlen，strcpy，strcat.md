# C语言：模拟实现库函数strlen，strcpy，strcat

# 模拟实现strlen

三种方法

```cpp
size_t my_strlen(char* s)//计数器
{
	size_t count = 0;
	while (*(s++))
		count++;
	return count;
}

size_t my_strlen(char* s)//递归
{
	if (*s == '\0')
		return 0;
	else
		return my_strlen(++s) + 1;
}

size_t my_strlen(char* s)//指针-指针
{
	char* tmp = s;
	while (*(++s))
		;
	return s - tmp;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 模拟实现strcpy

```cpp
char* my_strcpy(char* s1, const char* s2)//模拟实现strcpy
{	
	assert(s1 && s2);
	char* tmp = s1;
	while (*(s1++) = *(s2++))
		;
	return tmp;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 模拟实现strcat

```cpp
char* my_strcat(char* s1, const char* s2)//模拟实现strcat
{
	assert(s1&&s2);
	char* tmp = s1;
	while (s1++, *s1 != '\0')
		;
	while (*s1++ = *s2++)
		;
	return tmp;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

本博客旨在记录学习过程，以后忘了随时来看。