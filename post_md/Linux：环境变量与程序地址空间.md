> ![博客封面](https://raw.githubusercontent.com/QinMou000/pic/main/a46182e6318c4593a5c674f2bf9439d4.jpeg)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)
>
> ✨✨所属专栏：[Linux](https://blog.csdn.net/2301_80194476/category_12799988.html)✨✨
>
> ✨✨作者主页：[嶔某](https://blog.csdn.net/2301_80194476?spm=1000.2115.3001.5343)✨✨

#  环境变量

## 基本概念

环境变量(environment variables)⼀般是指在操作系统中⽤来指定操作系统运⾏环境的⼀些参数

如：我们在编写C/C++代码的时候，在链接的时候，从来不知道我们的所链接的动态静态库在哪 ⾥，但是照样可以链接成功，⽣成可执⾏程序，原因就是有相关环境变量帮助编译器进⾏查找。

环境变量通常具有某些特殊⽤途，还有在系统当中通常具有全局特性

## 常见环境变量

我们可以通过以下操作查看环境变量： 

```cpp
echo $NAME // 环境变量名
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

其中$符号的意思就是拿到后面跟的东西的值。

| 环境变量名称 | 表示内容           |
| ------------ | ------------------ |
| PATH         | 命令的搜索路径     |
| HOME         | 用户的主工作目录   |
| SHELL        | 当前Shell          |
| HOSTNAME     | 主机名             |
| TERM         | 终端类型           |
| HISTSIZE     | 记录历史命令的条数 |
| SSH_TTY      | 当前终端文件       |
| USER         | 当前用户           |
| MAIL         | 邮箱               |
| PWD          | 当前所处路径       |
| LANG         | 编码格式           |
| LOGNAME      | 登录用户名         |

 查看环境变量PATH

![img](https://raw.githubusercontent.com/QinMou000/pic/main/31b8db9a2c404c3c8dfb31272a5b6f45.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

那这一连串的地址究竟是指什么呢？在回答这个问题之前，我们首先要思考为什么输入指令时，直接输入指令名称即可如ls，而执行我们自己的可执行程序必须在前面加./表示当前路径呢？如./a.out。

其实答案很简单，系统能够通过指令名称找到其对应的位置，但是我们自己的可执行程序却不可以，必须指明在当前路径下。现在我们就知道环境变量PATH中的地址具体代表什么了，代表的就是默认查找的路径。

如果我们将自己编译的程序添加到这个PATH下或者改变PATH到有我们程序的路径，这样运行我们自己的程序就也不用加 ./ 了

![img](https://raw.githubusercontent.com/QinMou000/pic/main/4cdf49d818364f0db676676b0198fc74.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 指令

- 指令env：显示所有的环境变量。

![img](https://raw.githubusercontent.com/QinMou000/pic/main/2be0fd29c3bf4138893968c29ecaeb5c.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

- 指令export：设置一个新的环境变量。

```
export PATH=$PATH:/home/ubuntu/Code/progress_进度条
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://raw.githubusercontent.com/QinMou000/pic/main/affe8d99c45f4a7cb8ac85187f892aa4.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

- 指令set：显示本地定义的shell变量和环境变量。

![img](https://raw.githubusercontent.com/QinMou000/pic/main/deb1aef482374f729c4a36dcf0d6bdd7.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

- 指令unset：取消本地变量与环境变量。

## 环境变量的组织方式

 ![img](https://raw.githubusercontent.com/QinMou000/pic/main/7836126f74ea4ab495d463241cf96b93.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## main函数的三个参数

main函数其实是有参数的，但我们不常用，以下是main函数的原型 

```cpp
int main(int argc,char* argv[],char* env[])
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

- argc：代表命令行有效参数的个数。
- argv : 指向命令行参数。
- env: 指向环境变量。

后面两个参数都是一个字符指针数组

我们通过下面代码解释前两个参数： 

```cpp
int main(int argc,char* argv[])
{
    for(int i = 0;i<argc;i++)    
    {    
        printf("argv[%d]->%s\n",i,argv[i]);                                            
    }    
    return 0;    
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://raw.githubusercontent.com/QinMou000/pic/main/abaedf2f140948199b65e4ce68f7c641.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

一共四个有效参数，第一个为./test ，第二个为-a，第三个为-b，第四个为-c，这三个参数都被保存在字符指针数组argv里面，argv数组的最后一个元素是NULL。

同样我们探究以下第三个参数：

```cpp
int main(int argc, char *argv[], char *env[])
{
    int i = 0;
    for(i=0; env[i]; i++)
    {	
   		printf("[%d]->%s\n",i,env[i]);
    }
    return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://raw.githubusercontent.com/QinMou000/pic/main/911bdf27a8344b67b3952a6efc50eaf5.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

运行程序将显示所有的环境变。 

并且我们还可以通过第三方变量evnsion获取环境变量： 

```cpp
int main(int argc, char* argv[])
{
    extern char **environ;//先声明外部变量
    int i = 0;
    for(i = 0; environ[i]; i++)
    {
        printf("%s\n", environ[i]);
    }
    return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

结果当然也是打印所有的环境变量 

 函数getenv，你只需要给他传环境变量的名字，他就能获取对应的环境变量返回给你。

 ![img](https://raw.githubusercontent.com/QinMou000/pic/main/2cb046bf1cad4a27aec1a827684bab0d.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```cpp
int main(int argc, char *argv[], char *env[])
{
    printf("%s\n",getenv("PATH"));//获取对应的环境变量
    return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://raw.githubusercontent.com/QinMou000/pic/main/1875cee71c6b4a8f9c91ffde13826231.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

#  程序地址空间

在我们之前的学习中，下面的图片想必都不陌生 

 ![img](https://raw.githubusercontent.com/QinMou000/pic/main/cff3d37490394348b38876e5c0383835.jpeg)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

其中堆栈相对而生，栈向下生长(在栈上的变量先定义的地址更大)，堆向上生长(在堆上的变量先定义的地址更小)。

我们可以通过以下代码来验证：

```cpp
int g_unval;
int g_val = 100;
int main(int argc,char* argv[],char* env[])
{
    const char* str = "hello world";
    printf("code addr:%p\n",main);//main函数就在代码区
    printf("string rdonly addr:%p\n",str);//字符常量区
    printf("init addr:%p\n",&g_val);//已初始化全局数据区
    printf("uninit addr:%p\n",&g_unval);//未初始化全局数据
	
    //堆区
    char* heap1 = (char*)malloc(10);
    char* heap2 = (char*)malloc(10);                                                                                                               
    char* heap3 = (char*)malloc(10);    
    char* heap4 = (char*)malloc(10);    
    printf("heap1 addr:%p\n",heap1);    
    printf("heap2 addr:%p\n",heap2);    
    printf("heap3 addr:%p\n",heap3);    
    printf("heap4 addr:%p\n",heap4);    
	//栈区
    int a = 10;                    
    int b = 20;                    
    printf("stack addr:%p\n",&a);    
    printf("stack addr:%p\n",&b);    
	//命令行参数
    int i = 0;
    for( i = 0; argv[i]; i++)    
    {                              
        printf("argv[%d]:%p\n", i, argv[i]);    
    }
	//环境变量
    for(i = 0; env[i]; i++)
    {
        printf("env[%d]:%p\n", i, env[i]);
    }
    return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://raw.githubusercontent.com/QinMou000/pic/main/402f8ef213504f2e96414da12dd5e6cc.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 再来看这样一段代码：

```cpp
#include<stdio.h>
#include<sys/types.h>
#include<unistd.h>
int g_val=100;
int main()
{
    pid_t id=fork();//创建子进程
    if(id==0)
    {
        //child
        g_val=200;
        printf("child PID:%d,PPID:%d,g_val:%d,&g_val:%p\n",getpid(),getppid(),g_val,&g_val);
    }
    else if(id>0)
    {
        //father
        printf("father PID:%d,PPID:%d,g_val:%d,&g_val:%p\n",getpid(),getppid(),g_val,&g_val);
    }
    else
    {
        // fork error
    }
    return 0;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 ![img](https://raw.githubusercontent.com/QinMou000/pic/main/03b0a5d4cdea445080398bd7fd4117ea.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

我们发现，当数据发生修改的时，在父子进程当中的同一个变量，地址是相同的，但是值却是不同的，这明显不符合我们的认知，因为同一个地址的值怎么可能不同呢。

前面我们已经知道，当fork创建子进程时，父子默认情况共享数据。然而修改数据时，为了维护进程独立性，会发生写时拷贝，所以父子进程的值不同，但是地址为什么会不变呢？如果我们是在同一个物理地址处获取的值，那必定值是相同的，而现在在同一个地址处获取到的值却不同，**这只能证明我们打印出的地址并不是物理地址。**

-  *实际上，我们在语言层面上打印出来的地址都不是物理地址，而是一种虚拟地址。物理地址用户一概是看不到的，是由操作系统统一进行管理的，所以即使父子进程打印的地址相同，但是物理地址可能是不同的，这也就解释了为什么地址相同，而值却不同的问题。*

##  进程地址空间

我们之前将那张布局图称为程序地址空间实际上是不准确的，那张布局图实际上应该叫做**进程地址空间**，**进程地址空间本质是内存中的一种内核数据结构**，再Linux当中进程地址空间具体由结构体mm_struct实现，其一般包含以下这些信息： 

Linux kernal 2.0.12 

```cpp
struct mm_struct {
	int count;
	pgd_t * pgd;
	unsigned long context;
	unsigned long start_code, end_code, // 代码区
                  start_data, end_data; // 已初始化数据段
	unsigned long start_brk, brk, // 堆区
                  start_stack, //栈区
                  start_mmap; //用于标记进程内存空间中内存映射区域的起始地址
	unsigned long arg_start, arg_end, // 命令行参数段
                  env_start, env_end; // 进入环境变量段
	unsigned long rss, // 当前实际驻留在物理内存中的页面数量所对应的内存大小
             total_vm, // 进程所使用的总的虚拟内存页数
             locked_vm; // 进程中被锁定的内存页的数量，
                        // 这些被锁定的内存页不能被换出到磁盘，
                        // 会一直驻留在物理内存中
	unsigned long def_flags; // 进程内存描述符的一些默认标志位

	struct vm_area_struct * mmap; // 描述进程虚拟地址空间中的一个连续区域，
                                  // 也就是常说的 “虚拟内存区域”
	struct vm_area_struct * mmap_avl; // 指向一棵红黑树，以便更高效的查找插入删除
};
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

在结构体mm_struct当中，每一个的区域都代表一个虚拟地址，这些虚拟地址通过页表映射与物理内存建立联系。由于虚拟地址大小一般为4G，是由0x00000000到0xffffffff线性增长的，所以虚拟地址又叫做线性地址。

每个进程被创建时，其对应的进程控制块task_struct和进程地址空间mm_struct也随之被创建。而操作系统就可以通过进程的task_struct找到对应的mm_struct(因为task_struct有一个结构体指针指向的是mm_struct)。

![img](https://raw.githubusercontent.com/QinMou000/pic/main/db1eb3b81ee54b1787236520f24cfc46.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)
 然后我们就可以更加深入解释上面地址相同，值却不同的现象：首先父进程有自己的task_struct和mm_struct，该父进程创建的子进程也会有属于其自己的task_struct和mm_struct，父子进程的进程地址空间当中的各个虚拟地址分别通过页表映射到对应的物理内存，如下图：

 ![img](https://raw.githubusercontent.com/QinMou000/pic/main/2669c2a2679f4cd48e21166017e5b674.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

此时若是将g_val改为200，此时为了维护进程的独立性，不影响父进程的数据，子进程就会发生写实拷贝。 

![img](https://raw.githubusercontent.com/QinMou000/pic/main/0bb36184165949c38306c6226c5d032f.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

发生写时拷贝是因为进程间具有独立性。多进程运行，需要独享各种资源，运行期间互不干扰，不能让子进程的修改影响到父进程。 

之所以不在创建子进程的时候就进行数据的拷贝，是因为子进程不一定会使用父进程的所有数据，并且在子进程不对数据进行写入的情况下，没有必要对数据进行拷贝，我们应该按需分配，在需要修改数据的时候再分配（延时分配），提高空间利用率。

在绝大数情况下，代码是不会发生写时拷贝的，但这并不代表代码不能进行写时拷贝，例如在进行**进程替换**的时候，则需要进行代码的写时拷贝。

## 为什么要有进程地址空间？ 

- 通过添加一层软件层，实现对进程操作内存的风险管理（权限管理），本质是保护物理内存中各个进程的数据安全。
- 将内存申请和使用在时间上**解耦**，利用虚拟地址空间屏蔽底层申请内存过程，实现进程读写内存操作与操作系统内存管理在软件层面分离。
- 例如在堆上申请空间可能暂不全部使用甚至不用，从操作系统角度可在实际使用时再开辟空间建立映射关系，即基于缺页中断进行物理内存申请。
- 若物理内存已满而仍需申请，操作系统可执行内存管理算法，将某些进程闲置空间置换到磁盘，使进程仍能申请到内存，且用户在应用层无感知。
- 站在CPU和应用层角度，进程统一使用4GB空间且各空间区域相对位置较确定。有了虚拟地址空间，CPU能以统一视角看待物理内存，不同进程通过各自页表映射到不同物理内存，同时程序代码和数据可加载到内存任意位置，大大减少内存管理负担。

**本期博客到这里就结束了，如果有什么错误，欢迎指出，如果对你有帮助，请点个赞，谢谢！** 