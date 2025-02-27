# C语言字符函数strncpy,strncat,strstr,strtok,strerror内存函数memcpy，memove，memset，memcmp详解||模拟实现

# strncpy：

```cpp
//模拟实现strncpy
char* my_strncpy(char* str1,const char* str2,size_t num)
{
	assert(str1&&str2);
	char* tmp = str1;
	while ((*str1++ = *str2++) && --num)
		;
	if (num)
		while (num--)
		{
			*str1++ = '\0';
		}
	return tmp;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# strncat：

```cpp
//模拟实现strncat
char* my_strncat(char* str1, char* str2, size_t num)
{
	assert(str1&&str2);
	char* tmp = str1;
	while (*str1!='\0')
		str1++;
	while ((*str1++ = *str2++) && --num)
		;
	if (num)
		while (num--)
			*str1++ = '\0';
	return tmp;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# strstr：

```cpp
//模拟实现strstr
char * my_strstr (const char * str1, const char * str2)
{
    char *cp = (char *) str1;
    char *s1, *s2;
    if ( !*str2 )
        return((char *)str1);
    while (*cp)
    {
        s1 = cp;
        s2 = (char *) str2;
        while ( *s1 && *s2 && !(*s1-*s2) )
        s1++, s2++;
        if (!*s2)
            return(cp);
        cp++;
    }
    return(NULL);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# strtok：

```cpp
char * strtok ( char * str, const char * sep);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**sep参数指向⼀个字符串，定义了⽤作分隔符的字符集合
 • 第⼀个参数指定⼀个字符串，它包含了0个或者多个由sep字符串中⼀个或者多个分隔符分割的标
 记。
 • strtok函数找到str中的下⼀个标记，并将其⽤ \0 结尾，返回⼀个指向这个标记的指针。（注：
 strtok函数会改变被操作的字符串，所以在使⽤strtok函数切分的字符串⼀般都是临时拷⻉的内容
 并且可修改。）
 • strtok函数的第⼀个参数不为 NULL ，函数将找到str中第⼀个标记，strtok函数将保存它在字符串中的位置。
 • strtok函数的第⼀个参数为 NULL ，函数将在同⼀个字符串中被保存的位置开始，查找下⼀个标
 记。
 • 如果字符串中不存在更多的标记，则返回 NULL 指针。**

# strerror：

```cpp
#include <errno.h>
char * strerror ( int errnum );
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 **strerror函数可以把参数部分错误码对应的错误信息的字符串地址返回来。
 在不同的系统和C语⾔标准库的实现中都规定了⼀些错误码，⼀般是放在 errno.h 这个头⽂件中说明的，C语⾔程序启动的时候就会使⽤⼀个全⾯的变量errno来记录程序的当前错误码，只不过程序启动的时候errno是0，表⽰没有错误，当我们在使⽤标准库中的函数的时候发⽣了某种错误，就会讲对应的错误码，存放在errno中，⽽⼀个错误码的数字是整数很难理解是什么意思，所以每⼀个错误码都是有对应的错误信息的。strerror函数就可以将错误对应的错误信息字符串的地址返回。**

**在Windows VS2022环境下：**

| 1    | No error                  |
| ---- | ------------------------- |
| 2    | Operation not permitted   |
| 3    | No such file or directory |
| 4    | No such process           |
| 5    | Interrupted function call |
| 6    | Input/output error        |
| 7    | No such device or address |
| 8    | Arg list too long         |
| 9    | Exec format error         |
| 10   | Bad file descriptor       |
| 11   | No child processes        |

# memcpy:

```cpp
//模拟实现memcpy
void * my_memcpy ( void * dst, const void * src, size_t count)
{
    void * ret = dst;
    assert(dst);
    assert(src);
    while (count--)
    {
        *(char *)dst = *(char *)src;
        dst = (char *)dst + 1;
        src = (char *)src + 1;
    }
    return(ret);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# memove（这里注意根据dest和sour地址的高低分情况）:

![img](https://raw.githubusercontent.com/QinMou000/pic/main/3b82c45228d3b46f866f83a696cec601.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

```cpp
//模拟实现memmove
void* my_memmove(void* dest,void* sour,size_t num)
{
	assert(dest && sour);
	char* tmp = (char*)dest;
	if (dest == sour)
		return dest;
	else if (dest < sour)
	{
		while (num--)
		{
			*(((char*)dest)++) = *(((char*)sour)++);
		}
	}
	else
	{
		while (num--)
		{
			*(((char*) dest) + num) = *(((char*) sour) + num);
		}
	}
	return tmp;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# memset:

```cpp
void * memset ( void * ptr, int value, size_t num );
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 memset是⽤来设置内存的，将内存中的值以字节为单位设置成想要的内容。

# memcmp:

```cpp
int memcmp ( const void * ptr1, const void * ptr2, size_t num );
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

⽐较从ptr1和ptr2指针指向的位置开始，向后的num个字节（和strcmp差不多，memcmp可以比较任何内容）。

**本期博客到这里就结束了，如果有什么错误，欢迎指出，如果对你有帮助，请点个赞，谢谢！**