# C语言：的文件和文件操作

# 为什么会有文件？

如果没有⽂件，我们写的程序的数据是存储在电脑的**内存**中，如果程序退出，内存回收，数据就丢失了，等再次运⾏程序，是看不到上次程序的数据的，如果要将数据进⾏**持久化的保存**，我们可以使⽤⽂件。文件是放到电脑的**硬盘**里面的，可以存储我们想要长期保存的数据。但是在程序设计中，我们⼀般谈的⽂件有两种：程序⽂件、数据⽂件（从⽂件功能的⻆度来分类的）。

程序⽂件包括源程序⽂件（后缀为.c）,⽬标⽂件（windows环境后缀为.obj）,可执⾏程（windows环境后缀为.exe）。

⽂件的内容不⼀定是程序，⽽是程序运⾏时读写的数据，⽐如程序运⾏需要从中读取数据的⽂件，或者输出内容的⽂件。
 在这主要讨论的是数据⽂件。
 在以前我们写的代码处理数据的输⼊输出都是以终端为对象的，即从终端的键盘输⼊数据，运⾏结果显⽰到显⽰器上。其实有时候我们会把信息输出到磁盘上，当需要的时候再从磁盘上把数据读取到内存中使⽤，这⾥处理的就是磁盘上⽂件。

# 文件名

⼀个⽂件要有⼀个唯⼀的⽂件标识，以便⽤⼾识别和引⽤。
 ⽂件名包含3部分：⽂件路径+⽂件名主⼲+⽂件后缀
 例如： c:\code\test.txt
 为了⽅便起⻅，⽂件标识常被称为⽂件名。

# ⼆进制⽂件和⽂本⽂件？

根据数据的组织形式，数据⽂件被称为⽂本⽂件或者⼆进制⽂件。
 数据在内存中以⼆进制的形式存储，如果不加转换的输出到外存的⽂件中，就是⼆进制⽂件。
 如果要求在外存上以ASCII码的形式存储，则需要在存储前转换。以ASCII字符的形式存储的⽂件就是⽂本⽂件。
 ⼀个数据在⽂件中是怎么存储的呢？
 字符⼀律以ASCII形式存储，数值型数据既可以⽤ASCII形式存储，也可以使⽤⼆进制形式存储。如有整数10000，如果以ASCII码的形式输出到磁盘，则磁盘中占⽤5个字节（每个字符⼀个字节），⽽⼆进制形式输出，则在磁盘上只占4个字节（VS2019测试）。如果你想用short int类型存储，只需要占用两个字节。

# ⽂件的打开和关闭

## 流

我们程序的数据需要输出到各种外部设备，也需要从外部设备获取数据，不同的外部设备的输⼊输出操作各不相同，为了⽅便程序员对各种设备进⾏⽅便的操作，我们抽象出了流的概念，我们可以把流想象成流淌着字符的河。
 C程序针对⽂件、画⾯、键盘等的数据输⼊输出操作都是通过流操作的。
 ⼀般情况下，我们要想向流⾥写数据，或者从流中读取数据，都是要打开流，然后操作。

## 标准流

那为什么我们从键盘输⼊数据，向屏幕上输出数据，并没有打开流呢？
 那是因为C语⾔程序在启动的时候，默认打开了3个流：
 • stdin——标准输⼊流，在⼤多数的环境中从键盘输⼊，scanf函数就是从标准输⼊流中读取数据。
 • stdout——标准输出流，⼤多数的环境中输出⾄显⽰器界⾯，printf函数就是将信息输出到标准输出
 流中。
 • stderr——标准错误流，⼤多数环境中输出到显⽰器界⾯。
 这是默认打开了这三个流，我们使⽤scanf、printf等函数就可以直接进⾏输⼊输出操作的。
 stdin、stdout、stderr三个流的类型是： FILE* ，通常称为⽂件指针。
 C语⾔中，就是通过 FILE* 的⽂件指针来维护流的各种操作的。

# ⽂件指针

缓冲⽂件系统中，关键的概念是“⽂件类型指针”，简称“⽂件指针”。
 每个被使⽤的⽂件都在内存中开辟了⼀个相应的⽂件信息区，⽤来存放⽂件的相关信息（如⽂件的名字，⽂件状态及⽂件当前的位置等）。这些信息是保存在⼀个结构体变量中的。该结构体类型是由系统声明的，取名FILE.

每当打开⼀个⽂件的时候，系统会根据⽂件的情况⾃动创建⼀个FILE结构的变量，并填充其中的信
 息，使⽤者不必关⼼细节。
 ⼀般都是通过⼀个FILE的指针来维护这个FILE结构的变量，这样使⽤起来更加⽅便。
 下⾯我们可以创建⼀个FILE*的指针变量:

```cpp
FILE* pf;//⽂件指针变量
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

定义pf是⼀个指向FILE类型数据的指针变量。可以使pf指向某个⽂件的⽂件信息区（是⼀个结构体变量）。通过该⽂件信息区中的信息就能够访问该⽂件。也就是说，**通过⽂件指针变量能够间接找到与它关联的⽂件。**

***\*![img](https://raw.githubusercontent.com/QinMou000/pic/main/55c1492f6b9e89d559c113ea41442e5d.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑\****

# ⽂件的打开和关闭

⽂件在读写之前应该先打开⽂件，在使⽤结束之后应该关闭⽂件。
 在编写程序的时候，在打开⽂件的同时，都会返回⼀个FILE*的指针变量指向该⽂件，也相当于建⽴了指针和⽂件的关系。
 ANSI C规定使⽤ fopen 函数来打开⽂件， fclose 来关闭⽂件。

```cpp
//打开⽂件
FILE * fopen ( const char * filename, const char * mode );
//关闭⽂件
int fclose ( FILE * stream );
//一般都会在后面将文件指针置空防止野指针的出现
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

mode表⽰⽂件的打开模式，下⾯都是⽂件的打开模式：

| 文件使用方式            | 含义                                     | 如果指定文件不存在 |
| ----------------------- | ---------------------------------------- | ------------------ |
| “r”（只读）             | 为了输⼊数据，打开⼀个已经存在的⽂本⽂件 | 出错               |
| “w”（只写）             | 为了输出数据，打开⼀个⽂本⽂件           | 建⽴⼀个新的⽂件   |
| “a”（追加）             | 向⽂本⽂件尾添加数据                     | 建⽴⼀个新的⽂件   |
| “rb”（只读）            | 为了输⼊数据，打开⼀个⼆进制⽂件         | 出错               |
| “wb”（只写）            | 为了输出数据，打开⼀个⼆进制⽂件         | 建⽴⼀个新的⽂件   |
| “ab”（追加）            | 向⼀个⼆进制⽂件尾添加数据               | 建⽴⼀个新的⽂件   |
| “r+”（读写）            | 为了读和写，打开⼀个⽂本⽂件             | 出错               |
| “w+”（读写）            | 为了读和写，建议⼀个新的⽂件             | 建⽴⼀个新的⽂件   |
| “a+”（读写）            | 打开⼀个⽂件，在⽂件尾进⾏读写           | 建⽴⼀个新的⽂件   |
| “rb+”（读写）           | 为了读和写打开⼀个⼆进制⽂件             | 出错               |
| “wb+”（读  		写） | 为了读和写，新建⼀个新的⼆进制⽂件       | 建⽴⼀个新的⽂件   |
| “ab+”（读  		写） | 打开⼀个⼆进制⽂件，在⽂件尾进⾏读和写   | 建⽴⼀个新的⽂件   |

示例代码：

```cpp
/* fopen fclose example */
#include <stdio.h>
int main ()
{
    FILE * pFile;
    //打开⽂件
    pFile = fopen ("myfile.txt","w");
    //⽂件操作
    if(pFlie = NULL)
    {
        perror("fopen");
        return 1;
    }

    fputs ("fopen example",pFile);
    //关闭⽂件
    fclose (pFile);

    pFlie = NULL;//一定记得置空！！！
    return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 顺序读写函数介绍

| 函数名  | 功能           | 适⽤于     |
| ------- | -------------- | ---------- |
| fgetc   | 字符输⼊函数   | 所有输⼊流 |
| fputc   | 字符输出函数   | 所有输出流 |
| fgets   | ⽂本⾏输⼊函数 | 所有输⼊流 |
| fputs   | ⽂本⾏输出函数 | 所有输出流 |
| fscanf  | 格式化输⼊函数 | 所有输⼊流 |
| fprintf | 格式化输出函数 | 所有输出流 |
| fread   | ⼆进制输⼊     | ⽂件       |
| fwrite  | ⼆进制输出     | ⽂件       |

上⾯说的适⽤于所有输⼊流⼀般指适⽤于标准输⼊流和其他输⼊流（如⽂件输⼊流）；所有输出流⼀般指适⽤于标准输出流和其他输出流（如⽂件输出流）。

# 对⽐⼀组函数：

scanf/fscanf/sscanf
 printf/fprintf/sprintf

# 文件的随机读写

## fseek

根据⽂件指针的位置和偏移量来定位⽂件指针。

```cpp
int fseek ( FILE * stream, long int offset, int origin );
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

示例代码：

```cpp
#include <stdio.h>
int main()
{
	//FILE* pf;//⽂件指针变量
	/* fseek example */
	FILE* pFile;
	pFile = fopen("example.txt", "wb");
	fputs("This is an apple.", pFile);
	fseek(pFile, 9, SEEK_SET);
	/*
	SEEK_SET    文件起始位置
	SEEK_CUR	文件指针的当前位置
	SEEK_END	文件末尾
	*/
	fputs(" sam", pFile);
	fclose(pFile);
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

运行后的example.txt

![img](https://raw.githubusercontent.com/QinMou000/pic/main/73b69c52510d2c13680f3f6d86441437.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

##  ftell

返回⽂件指针相对于起始位置的偏移量
 1 long int ftell ( FILE * stream )

示例代码：

```cpp
/* ftell example : getting size of a file */
#include <stdio.h>
int main()
{
	long size;
	FILE* pFile = fopen("myfile.txt", "rb");
	if (pFile == NULL)
		perror("Error opening file");
	else
	{
		fseek(pFile, 0, SEEK_END); // non-portable
		size = ftell(pFile);
		fclose(pFile);
		printf("Size of myfile.txt: %ld bytes.\n", size);
	}
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

运行结果：

![img](https://raw.githubusercontent.com/QinMou000/pic/main/d9fbdefd939213a5089bee21e60c5c04.jpeg)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

##  rewind

让⽂件指针的位置回到⽂件的起始位置

```cpp
void rewind ( FILE * stream );
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

示例代码：

```cpp
/* rewind example */
#include <stdio.h>
int main()
{
	int n;
	FILE* pFile;
	char buffer[27];
	pFile = fopen("myfile.txt", "w+");
	for (n = 'A'; n <= 'Z'; n++)
		fputc(n, pFile);//将A到Z放进文件里面
	rewind(pFile);//返回文件开头，准备打印
	fread(buffer, 1, 26, pFile);//将文件里面读到的数据放到buffer里面
	fclose(pFile);
	pFile = NULL;
	buffer[26] = '\0';
	printf(buffer);//打印
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# ⽂件读取结束的判定

## 被错误使⽤的 feof 

牢记：在⽂件读取过程中，不能⽤feof函数的返回值直接来判断⽂件的是否结束。
 feof 的作⽤是：当⽂件读取结束的时候，判断是读取结束的原因是否是：遇到⽂件尾结束。

1. ⽂本⽂件读取是否结束，判断返回值是否为 EOF （ fgetc ），或者 NULL （ fgets ）

 例如：
 • fgetc 判断是否为 EOF .
 • fgets 判断返回值是否为 NULL .

2. ⼆进制⽂件的读取结束判断，判断返回值是否⼩于实际要读的个数。

 例如：
 • fread判断返回值是否⼩于实际要读的个数。

文本文件示例代码：

```cpp
#include <stdio.h>
#include <stdlib.h>
int main()
{
	int c; // 注意：int，⾮char，要求处理EOF
	FILE* fp = fopen("test.txt", "r");
	if (!fp) {
		perror("File opening failed");
		return EXIT_FAILURE;
	}
	//fgetc 当读取失败的时候或者遇到⽂件结束的时候，都会返回EOF
	while ((c = fgetc(fp)) != EOF) // 标准C I/O读取⽂件循环
	{
		putchar(c);
	}
	//判断是什么原因结束的
	if (ferror(fp))
		puts("I/O error when reading");
	else if (feof(fp))
		puts("End of file reached successfully");
	fclose(fp);
    pf = NULL;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

二进制文件示例代码：

```cpp
#include <stdio.h>
enum { SIZE = 5 };
int main()
{
	double a[SIZE] = { 1.,2.,3.,4.,5. };
	double b[SIZE] = { 0 };
	FILE* fp = fopen("test.bin", "wb"); // 必须⽤⼆进制模式
	fwrite(a, sizeof * a, SIZE, fp); // 写 double 的数组
	fclose(fp);
	fp = fopen("test.bin", "rb");
	size_t ret_code = fread(b, sizeof * b, SIZE, fp); // 读 double 的数组
	if (ret_code == SIZE) {
		puts("Array read successfully, contents: ");
		for (int n = 0; n < SIZE; ++n)
			printf("%f ", b[n]);
		putchar('\n');
	}
	else { // error handling
		if (feof(fp))
			printf("Error reading test.bin: unexpected end of file\n");
		else if (ferror(fp)) {
			perror("Error reading test.bin");
		}
	}
	fclose(fp);
	fp = NULL;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# ⽂件缓冲区

ANSIC标准采⽤“缓冲⽂件系统”处理的数据⽂件的，所谓缓冲⽂件系统是指系统⾃动地在内存中为程序中每⼀个正在使⽤的⽂件开辟⼀块“⽂件缓冲区”。从内存向磁盘输出数据会先送到内存中的缓冲区，装满缓冲区后才⼀起送到磁盘上。如果从磁盘向计算机读⼊数据，则从磁盘⽂件中读取数据输⼊到内存缓冲区（充满缓冲区），然后再从缓冲区逐个地将数据送到程序数据区（程序变量等）。缓冲区的⼤⼩根据C编译系统决定的。(大家可以复制下面这段代码，放在编译器里面自己试一下)

```cpp
#include <stdio.h>
#include <windows.h>
//VS2022 WIN11环境测试
int main()
{
	FILE* pf = fopen("test.txt", "w");
	fputs("abcdef", pf);//先将代码放在输出缓冲区
	printf("睡眠10秒-已经写数据了，打开test.txt文件，发现文件没有内容\n");
	Sleep(10000);
	printf("刷新缓冲区\n");
	fflush(pf);//刷新缓冲区时，才将输出缓冲区的数据写到⽂件（磁盘）
	//注：fflush 在⾼版本的VS上不能使⽤了
	printf("再睡眠10秒-此时，再次打开test.txt文件，文件有内容了\n");
	Sleep(10000);
	fclose(pf);
	//注：fclose在关闭⽂件的时候，也会刷新缓冲区
	pf = NULL;
	return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**本期博客到这里就结束了，如果有什么错误，欢迎指出，如果对你有帮助，请点个赞，谢谢！**