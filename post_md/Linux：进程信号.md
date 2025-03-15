> ![博客封面](https://raw.githubusercontent.com/QinMou000/pic/main/a46182e6318c4593a5c674f2bf9439d4.jpeg)
>
> ✨✨所属专栏：[Linux](https://blog.csdn.net/2301_80194476/category_12799988.html)✨✨
>
> ✨✨作者主页：[嶔某](https://blog.csdn.net/2301_80194476?spm=1000.2115.3001.5343)✨✨
>

# Linux：进程信号

在讲信号之前，我们先来从生活中的事情来确定信号的一些特性。

- 我在网上买了商品，我在等快递。但是在快递没来之前我知道快递来的时候我应该怎么处理。我能**识别快递**。

- 快递来了，快递小哥给我打电话让我下楼取快递，而我在打游戏，等会再下去拿。在等会的这段时间里，我知道快递来了，但是我并没有立即去处理它，**没有立即执行**，也就是在合适的时候去取。

- 在这个等一会的时间窗口，我知道有一个快递在等我去取，那么我是记住了这件事的。

- 我拿到快递后有三种动作，

- > 1. 执行默认动作（拆开快递）
  > 2. 自定义动作（送别人）
  > 3. 忽略（继续打游戏）

- 在整个来快递的过程中对我来说是**异步**的，我并不知道快递员什么时候给我打电话。

基本结论：

1. 我为什么能识别信号，信号是内置的，进程识别信号是内核程序员写的内置特性。
2. 信号产生后，我知道怎么处理。信号没有产生，我也知道怎么处理。所以信号的处理方法在信号产生之前，就已经准备好了。
3. 我们不一定立即处理信号，可能有优先级更高的事。那什么时候？合适的时候。
4. 三个步骤：信号到来 | 信号保存 | 信号处理
5. 怎么进行信号处理：默认、忽略、自定义，后续都叫做信号捕捉。

我们平时，`Ctrl + C`其实就是在给前台进程发信号。键盘输入一个硬件中断，被OS获取，解释成信号，发送给目标前台进程。进程收到信号，引起进程退出。

**系统函数**

```C++
NAME
	signal - ANSI C signal handing
SYNOPISIS
	#include <signal.h>
	typedef void (*sighandler_t)(int);
	sighandler_t signal(int signum, sighandler_t handler);
参数说明：
signum：信号编号
handler：函数指针，表示更改信号的处理动作，当收到对应的信号，就回调执行handler方法
```

ctrl + C其实就是在给前台进程发送`SIGINT`即`2`号信号。

要注意的是，`signal`函数仅仅是设置了特定信号的捕捉行为处理方式，并不是直接调用处理动作。如果后续特定信号没有产生，设置的捕捉函数永远也不会被调用！

`Ctrl+ C`产生的信号只能发给前台进程。一个命令后面加个&可以放到后台运行，这样`Shell`不必等待进程结束就可以接受新的命令，启动新的进程。

`Shell`可以同时运行一个前台进程和任意多个后台进程，只有前台进程才能接收到像`Ctrl + C`这种控制键产生的信号。

前台进程在运行过程中用户随时可能按下`Ctrl + C`而产生一个信号，也就是说该进程用户空间代码执行到任何地方都有可能收到`SIGINT`信号而终止，所以信号相对于进程的控制流程来说是异步`Asynchronous`的

## **补充同步异步概念**

> **一、同步（Synchronous）**
>
> 1. **定义**
>    同步操作要求任务按顺序执行，前一个任务完成后才能启动下一个任务。主线程会**阻塞等待**当前任务返回结果，后续代码无法继续执行。
>    **类比**：类似排队办理银行业务，必须等待前一个人完成才能轮到下一个人。
> 2. **特点**
>    - **顺序性**：代码执行顺序与编写顺序一致，逻辑简单。
>    - **阻塞性**：主线程在等待结果时会被挂起，可能导致界面卡顿或性能下降。
> 3. **应用场景**
>    - 简单且非耗时操作（如变量赋值、数学计算）。
>    - 需要严格顺序执行的流程（如先登录后加载用户数据）。
>    - 要求数据强一致性的场景（如银行转账操作）。
>
> **二、异步（Asynchronous）**
>
> 1. **定义**
>    异步操作将耗时任务放入后台执行，主线程**不等待结果**而继续执行后续代码。任务完成后通过回调函数、事件通知等方式返回结果。
>    **类比**：在餐厅点餐后领取号码牌，期间可自由活动，餐好后凭通知取餐。
> 2. **特点**
>    - **非阻塞性**：主线程资源高效利用，避免卡顿。
>    - **复杂性**：需通过回调、Promise、async/await等机制处理结果。
> 3. **应用场景**
>    - 耗时操作（如网络请求、文件读写）。
>    - 用户交互事件（如点击、滚动监听）。
>    - 高并发场景（如消息队列处理数据库批量写入）。

## **基础进程切换命令**

1. `&`符号

   - **用途**：直接在命令末尾添加`&`，使程序**立即在后台运行**

   - 示例

     ：

     ```bash
     python script.py  &  # 脚本在后台运行  
     ```

2. `Ctrl+Z`组合键

   - **用途**：**暂停前台进程**并将其转入后台（状态为`Stopped`）
   - **示例**：运行`top`时按下`Ctrl+Z`，进程暂停并显示`[1]()+ Stopped`

3. `jobs`命令

   - **用途**：查看当前Shell会话中的**后台任务列表**，显示任务编号和状态

   - 常用参数

     ```bash
     jobs -l  # 显示任务PID  
     ```

4. `fg`命令

   - **用途**：将后台任务**切换至前台继续运行**

   - 语法：

     ```bash
     fg %n  # n为jobs显示的任务编号  
     ```

   - **示例**：`fg %1`将编号1的任务调回前台
     [5](https://blog.csdn.net/u012317833/article/details/39249395)[8](https://blog.csdn.net/firstcode666/article/details/122223976)

5. `bg`命令

   - **用途**：**恢复暂停的后台任务**，使其继续在后台运行

   - 语法：

     ```bash
     bg %n  # 启动编号为n的暂停任务  
     ```

   - **示例**：暂停的`top`任务执行`bg %1`后转为后台运行

------

**二、高级管理命令**

1. `nohup`命令

   - **用途**：**脱离终端运行程序**，即使关闭SSH连接进程仍持续

   - 示例：

     ```bash
     nohup python server.py  &  # 输出日志到nohup.out   
     ```

   - **查看日志**：`tail -f nohup.out`

2. `kill`命令

   - **用途**：终止后台任务

   - 语法：

     ```bash
     kill %n       # 通过任务编号终止  
     kill <PID>    # 通过进程ID终止  
     ```

   - **示例**：`kill %2`终止编号为2的任务

------

**四、注意事项**

- **任务编号与PID**：`fg`/`bg`操作依赖`jobs`显示的**任务编号**，而非系统PID
- **终端依赖**：普通后台任务（未用`nohup`）会随终端关闭终止
- **并发控制**：多个后台任务时，建议用`jobs`定期检查状态

## 信号概念

信号是进程之间事件异步通知的一种方式，属于软中断。

### 查看信号

```bash
ubuntu@VM-4-4-ubuntu:~/Code$ kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

每个进程都有一个编号和一个宏定义名称，这些宏定义可以在`signal.h`中找到。

```C++
#define SIGHUP		 1	/* Hangup (POSIX).  */
#define SIGINT		 2	/* Interrupt (ANSI).  */
#define SIGQUIT		 3	/* Quit (POSIX).  */
#define SIGILL		 4	/* Illegal instruction (ANSI).  */
#define SIGTRAP		 5	/* Trace trap (POSIX).  */
#define SIGIOT		 6	/* IOT trap (4.2 BSD).  */
#define SIGABRT		 SIGIOT	/* Abort (ANSI).  */
#define SIGEMT		 7
#define SIGFPE		 8	/* Floating-point exception (ANSI).  */
#define SIGKILL		 9	/* Kill, unblockable (POSIX).  */
#define SIGBUS		10	/* BUS error (4.2 BSD).  */
#define SIGSEGV		11	/* Segmentation violation (ANSI).  */
#define SIGSYS		12
#define SIGPIPE		13	/* Broken pipe (POSIX).  */
#define SIGALRM		14	/* Alarm clock (POSIX).  */
#define SIGTERM		15	/* Termination (ANSI).  */
#define SIGUSR1		16	/* User-defined signal 1 (POSIX).  */
#define SIGUSR2		17	/* User-defined signal 2 (POSIX).  */
#define SIGCHLD		18	/* Child status has changed (POSIX).  */
#define SIGCLD		SIGCHLD	/* Same as SIGCHLD (System V).  */
#define SIGPWR		19	/* Power failure restart (System V).  */
#define SIGWINCH	20	/* Window size change (4.3 BSD, Sun).  */
#define SIGURG		21	/* Urgent condition on socket (4.2 BSD).  */
#define SIGIO		22	/* I/O now possible (4.2 BSD).  */
#define SIGPOLL		SIGIO	/* Pollable event occurred (System V).  */
#define SIGSTOP		23	/* Stop, unblockable (POSIX).  */
#define SIGTSTP		24	/* Keyboard stop (POSIX).  */
#define SIGCONT		25	/* Continue (POSIX).  */
#define SIGTTIN		26	/* Background read from tty (POSIX).  */
#define SIGTTOU		27	/* Background write to tty (POSIX).  */
#define SIGVTALRM	28	/* Virtual alarm clock (4.2 BSD).  */
#define SIGPROF		29	/* Profiling alarm clock (4.2 BSD).  */
#define SIGXCPU		30	/* CPU limit exceeded (4.2 BSD).  */
#define SIGXFSZ		31	/* File size limit exceeded (4.2 BSD).  */
```

信号编号没有32、33所以只有62种信号，编号34以上的是实时信号，不讨论实时信号。这些信号各自在什么条件下产生，默认的处理动作是什么，在`signal(7)`中有详细说明：`man 7 signal`

```
Standard signals
       Linux  supports  the  standard  signals listed below.  The second column of the table indicates which standard (if any) specified the signal: "P1990" indicates that the signal is described in the original
       POSIX.1-1990 standard; "P2001" indicates that the signal was added in SUSv2 and POSIX.1-2001.
       Signal      Standard   Action   Comment
       ────────────────────────────────────────────────────────────────────────
       SIGABRT      P1990      Core    Abort signal from abort(3)
       SIGALRM      P1990      Term    Timer signal from alarm(2)
       SIGBUS       P2001      Core    Bus error (bad memory access)
       SIGCHLD      P1990      Ign     Child stopped or terminated
       SIGCLD         -        Ign     A synonym for SIGCHLD
       SIGCONT      P1990      Cont    Continue if stopped
       SIGEMT         -        Term    Emulator trap
       SIGFPE       P1990      Core    Floating-point exception
       SIGHUP       P1990      Term    Hangup detected on controlling terminal
                                       or death of controlling process
       SIGILL       P1990      Core    Illegal Instruction
       SIGINFO        -                A synonym for SIGPWR
       SIGINT       P1990      Term    Interrupt from keyboard
       SIGIO          -        Term    I/O now possible (4.2BSD)
       SIGIOT         -        Core    IOT trap. A synonym for SIGABRT
       SIGKILL      P1990      Term    Kill signal
       SIGLOST        -        Term    File lock lost (unused)
       SIGPIPE      P1990      Term    Broken pipe: write to pipe with no
                                       readers; see pipe(7)
       SIGPOLL      P2001      Term    Pollable event (Sys V);
                                       synonym for SIGIO
       SIGPROF      P2001      Term    Profiling timer expired
       SIGPWR         -        Term    Power failure (System V)
       SIGQUIT      P1990      Core    Quit from keyboard
       SIGSEGV      P1990      Core    Invalid memory reference
       SIGSTKFLT      -        Term    Stack fault on coprocessor (unused)
       SIGSTOP      P1990      Stop    Stop process
       SIGTSTP      P1990      Stop    Stop typed at terminal
       SIGSYS       P2001      Core    Bad system call (SVr4);
                                       see also seccomp(2)
       SIGTERM      P1990      Term    Termination signal
       SIGTRAP      P2001      Core    Trace/breakpoint trap
       SIGTTIN      P1990      Stop    Terminal input for background process
       SIGTTOU      P1990      Stop    Terminal output for background process
       SIGUNUSED      -        Core    Synonymous with SIGSYS
       SIGURG       P2001      Ign     Urgent condition on socket (4.2BSD)
       SIGUSR1      P1990      Term    User-defined signal 1
       SIGUSR2      P1990      Term    User-defined signal 2
       SIGVTALRM    P2001      Term    Virtual alarm clock (4.2BSD)
       SIGXCPU      P2001      Core    CPU time limit exceeded (4.2BSD);
                                       see setrlimit(2)
       SIGXFSZ      P2001      Core    File size limit exceeded (4.2BSD);
                                       see setrlimit(2)
       SIGWINCH       -        Ign     Window resize signal (4.3BSD, Sun)

       The signals SIGKILL and SIGSTOP cannot be caught, blocked, or ignored.
```

### 信号处理

- 忽略此信号

```C++
#include <iostream>
#include <unistd.h>
#include <signal.h>

void handler(int num)
{
    std::cout << "我是: " << getpid() << ", 我获得了⼀个信号: " << signumber << std::endl;
}

int main()
{
    std::cout << "我是进程: " << getpid() << std::endl;
    signal(SIGINT /*2*/, SIG_IGN); // 设置忽略信号的宏
    while (true)
    {
        std::cout << "I am a process, I am waiting signal!" << std::endl;
        sleep(1);
    }
    return 0;
}
ubuntu@VM-4-4-ubuntu:~/Code/25/3_15$ ./sig 
我是进程: 544272
I am a process, I am waiting signal!
I am a process, I am waiting signal!
I am a process, I am waiting signal!
I am a process, I am waiting signal!
I am a process, I am waiting signal!
^CI am a process, I am waiting signal! // 输入 ctrl + C 毫无反应
```

- 执行该信号的默认处理动作

```C++
#include <iostream>
#include <unistd.h>
#include <signal.h>

void handler(int signumber)
{
    std::cout << "我是: " << getpid() << ", 我获得了⼀个信号: " << signumber << std::endl;
}

int main()
{
    std::cout << "我是进程: " << getpid() << std::endl;
    signal(SIGINT /*2*/, SIG_DFL); // 设置默认处理
    while (true)
    {
        std::cout << "I am a process, I am waiting signal!" << std::endl;
        sleep(1);
    }
    return 0;
}

ubuntu@VM-4-4-ubuntu:~/Code/25/3_15$ ./sig 
我是进程: 544934
I am a process, I am waiting signal!
I am a process, I am waiting signal!
I am a process, I am waiting signal!
^C
ubuntu@VM-4-4-ubuntu:~/Code/25/3_15$ 
```

- 提供一个信号处理函数，要求内核在处理该信号是切换到用户态执行这个处理函数，这种方式称为自定义捕捉`catch`信号

```C++
#include <iostream>
#include <unistd.h>
#include <signal.h>

void handler(int signumber)
{
    std::cout << "我是: " << getpid() << ", 我获得了⼀个信号: " << signumber << std::endl;
}

int main()
{
    std::cout << "我是进程: " << getpid() << std::endl;
    signal(SIGINT /*2*/, handler); // 设置自定义函数处理
    while (true)
    {
        std::cout << "I am a process, I am waiting signal!" << std::endl;
        sleep(1);
    }
    return 0;
}

ubuntu@VM-4-4-ubuntu:~/Code/25/3_15$ ./sig 
我是进程: 545560
I am a process, I am waiting signal!
I am a process, I am waiting signal!
I am a process, I am waiting signal!
^C我是: 545560, 我获得了⼀个信号: 2
I am a process, I am waiting signal!
```

接下来我们将从产生信号，保存信号，捕捉信号三个方面来具体总结。

## 产生信号

### 通过终端按键产生信号

基本操作：

- `Ctrl + C`向前台进程发送`SIGINT`信号
- `Ctrl + \`发送终止信号`SIGQUIT`并生成core dump文件，用于事后调试。
- `Ctrl + Z`发送停止信号`SIGTSTP`将当前前台进程挂起到后台等待。

ok这里就有一个问题了。键盘按下对应组合键是如何使进程进行对应操作的呢？键盘等硬件是直接或间接的与`CPU`上的针脚连接的，当按键按下，硬件发送一个中断给`CPU`，`CPU`识别到中断信息（高电平）然后就去执行处理硬件数据的代码。从操作系统来看就是`OS`停下当前工作将数据从硬件读取到内存。

那么，信号就是从纯软件的角度来模拟硬件中断。硬件中断是发给`CPU`软中断是发给进程。两者在思想上是完全一致的。

### 使用函数产生信号

#### kill

我们在终端使用的`kill`命令本质也是进程，也是用C语言写的。底层也是调用的这个`kill`函数。kill函数会给一个指定的进程发送指定的信号

```
NAME
	kill - send signal to a process
SYNOPSIS
	#include <sys/type.h>
	#include <signal.h>
	int kill(pid_t pid, int sig);
RETURN VALUE
	On success (at least one signal was sent) zero is returned. On erroe, -1 is returned, and errno is set 
	appropriately.
```

**mykill**

```C++
#include <iostream>
#include <unistd.h>
#include <signal.h>
#include <sys/types.h>

// 实现自己的kill命令
// mykill -signumber pid
int main(int argc, char *argv[])
{
    if (argc != 3)
    {
        std::cerr << "Usage: " << argv[0] << " -signumber pid" << std::endl;
        return 1;
    }
    int number = std::stoi(argv[1] + 1); // 去掉- 获取信号编号
    pid_t pid = std::stoi(argv[2]);
    int n = kill(pid, number);
    return n;
}
```

#### raise

`raise`函数可以给当前进程发送指定的信号，也就是给自己发信号。

```C++
NAME 
	raise - send a signal to the caller
SYNOPSIS
    #include <signal.h>
    int raise(int sig);
RETURN VALUE
       raise() returns 0 on success, and nonzero for failure.
```

#### abort

`abort`函数使当前进程收到信号而异常终止，它总会成功的就像`exit`一样

```C++
NAME 
	abort - cause abnormal process termination
SYNOPSIS
	#include <stdlib.h>
	void abort(void);
RETURN VALUE
	The abort() function never returns.
```

### 由软件产生信号

`SIGPIPE`和`SIGALRM`信号是一种由软件产生的信号，管道我们已经学过了。现在来学习时钟信号`alarm`函数

```C++
NAME
	alarm - set an alarm clock for delivery of a signal
SYNOPSIS
	#include <unistd.h>
    unsigned int alarm(unsigned int seconds);
DESCRIPTION
    alarm() arranges for a SIGALRM signal to be delivered to the calling process in seconds seconds.
    If seconds is zero, any pending alarm is canceled.
    In any event any previously set alarm() is canceled.
RETURN VALUE
    alarm() returns the number of seconds remaining until any previously
    scheduled alarm was due to be delivered, or zero if there was no previ‐
    ously scheduled alarm.
```

- 调用`alarm`函数可以设定一个闹钟，也就是告诉内核在`seconds`秒之后给当前进程发`SIGALRM`信号，该信号的默认处理动作是终止当前进程。
- 这个函数的返回值是0或者是以前设定的闹钟时间还余下的秒数。如果`seconds`的值为0，表示取消以前设定的闹钟，函数的返回值仍然是以前设定的闹钟时间还余下的秒数。

#### IO效率问题

这里延申出一个IO效率的问题，分别有两个程序，一个程序在一秒的时间内不断向显示屏打印信息并使计数器加加，另一个在一秒钟之内不断只对一个计数器加加。最后看它们的计数器大小。

```C++
#include <iostream>
#include <unistd.h>
#include <signal.h>
#include <sys/types.h>
int cnt = 0;

void Exit(int signo)
{
    std::cout << cnt << std::endl;
    exit(1);
}

int main()
{
    signal(SIGALRM, Exit);
    alarm(1);

    while (true)
    {
        cnt++;
    }
    return 0;
}

ubuntu@VM-4-4-ubuntu:~/Code/25/3_15$ make 
g++ -o sig sig.cc -std=c++11
ubuntu@VM-4-4-ubuntu:~/Code/25/3_15$ ./sig 
560154357
```

```C++
#include <iostream>
#include <unistd.h>
#include <signal.h>
#include <sys/types.h>
int cnt = 0;

void Exit(int signo)
{
    std::cout << cnt << std::endl;
    exit(1);
}

int main()
{
    signal(SIGALRM, Exit);
    alarm(1);

    while (true)
    {
        std::cout << "cnt:" << cnt << std::endl;
        cnt++;
    }
    return 0;
}

ubuntu@VM-4-4-ubuntu:~/Code/25/3_15$ make 
g++ -o sig sig.cc -std=c++11
ubuntu@VM-4-4-ubuntu:~/Code/25/3_15$ ./sig 
cnt:0
cnt:1
cnt:2
......
cnt:53558
cnt:53559
cnt:53559
```

事实证明，相比于这种算术运算，IO处理的是非常慢的。两者之间不止差了一个数量级。

#### 设置重复闹钟

```C++
#include <iostream>
#include <vector>
#include <functional>
#include <unistd.h>
#include <signal.h>
#include <sys/types.h>

using func_t = std::function<void()>;
std::vector<func_t> task;

void handler(int signo)
{
    for (auto f : task)
    {
        f();
    }
    int n = alarm(1);
    std::cout << "n: " << n << std::endl;
}

int main()
{
    task.push_back([]()
                   { std::cout << "刷新内核" << std::endl; });
    task.push_back([]()
                   { std::cout << "检测进程时间片" << std::endl; });
    task.push_back([]()
                   { std::cout << "管理内存" << std::endl; });

    signal(SIGALRM, handler);

    alarm(1);

    while (true)
    {
        pause();
        std::cout << "wake up" << std::endl;
    }
    return 0;
}
/***************************************************************************************************************************************/
NAME
       pause - wait for signal
           
SYNOPSIS
       #include <unistd.h>
       int pause(void);

DESCRIPTION
       pause() causes the calling process (or thread) to sleep until a signal is delivered that either terminates the process or causes the invocation of a signal-catching function.
    
RETURN VALUE
       pause() returns only when a signal was caught and the signal-catching function returned.  In this case, pause() returns -1, and errno is set to EINTR.
```

### 硬件异常产生信号

硬件异常被硬件以某种方式被硬件检测到并通知内核，然后内核发送适当的信号给当前进程。例如当前进程进行了除0的指令，CPU的运算单位会产生异常，内核将这个异常解释成`SIGFPE`信号发给进程。如果当前进程访问非法内存地址，`MMU`会产生异常，内核将在这个异常解释为`SIGSEGV`信号发送给进程。

#### core dump

在一些信号的默认`action`中（上翻`查看信号`）有 ign、core等。它们是什么意思？

- `SIGINT`的默认处理动作是终止进程，`SIGQUIT`的默认处理动作是终止进程并且`Core Dump`，当一个进程要异常终止时，可以把进程的用户空间内存数据全部保存到磁盘上，文件名通常是`core`，这叫做`Core Dump`。
- 进程异常终止通常是因为有`Bug`比如非法内存访问导致段错误，事后可以用调试器检查`core`文件以查清错误原因，这叫做`Post-mortem DeBug`事后调试。
- 一个进程允许产生多大的`core`文件取决于进程的`Resource Limit`（在PCB中），默认不允许产生`core`文件，因为可能包含用户密码等敏感信息。
- 在开发测试阶段可以用`ulimit`命令该变这个限制，允许产生`core`文件。先用这个命令改变`Shell`进程的`Resource Limit`，如改到`1024K：$ ulimit -c 1024`

```Shell
ubuntu@VM-4-4-ubuntu:~/Code/25/3_15$ ulimit -c 1024
ubuntu@VM-4-4-ubuntu:~/Code/25/3_15$ ulimit -a
real-time non-blocking time  (microseconds, -R) unlimited
core file size              (blocks, -c) 1024
data seg size               (kbytes, -d) unlimited
scheduling priority                 (-e) 0
file size                   (blocks, -f) unlimited
pending signals                     (-i) 6563
max locked memory           (kbytes, -l) 219108
max memory size             (kbytes, -m) unlimited
open files                          (-n) 1048576
pipe size                (512 bytes, -p) 8
POSIX message queues         (bytes, -q) 819200
real-time priority                  (-r) 0
stack size                  (kbytes, -s) 8192
cpu time                   (seconds, -t) unlimited
max user processes                  (-u) 6563
virtual memory              (kbytes, -v) unlimited
file locks                          (-x) unlimited
```

## 保存信号

上面我们说了信号产生后都需要OS来执行，因为OS是进程的管理者。但信号不是被立即处理的，是在合适的时候处理。那既然不是立即处理，总该保存吧，那保存在哪里呢？

**概念说明：**

- 实际执行信号处理动作称为信号递达`Delivery`
- 信号从产生到递达之间的状态成为信号未决`Pending`
- 进程可以选择阻塞`Block`某个信号
- **被阻塞的信号产生时将保持在未决状态，直到进程解除对此信号的阻塞，才执行递达的动作**
- 注意：阻塞和忽略是不同的，忽略是在信号递达后可选的一种处理方式，但信号被阻塞了就无法递达

在内核中进程`PCB`维护了这三张表：

![image-20250315215033312](https://raw.githubusercontent.com/QinMou000/pic/main/image-20250315215033312.png)

- 每个信号都有两个标志位分别表示阻塞`block`和未决`pending`，还有一个函数指针表示处理动作。信号产生时，内核在进程控制块中设置该信号的未决标志，直到信号递达才清除标志。在上图的例子中，`SIGHUP`信号未阻塞也未产生过，当它递达时执行默认处理动作。
- `SIGINT`信号产生过，但正在被阻塞，所以暂时不能被递达，虽然它的处理动作为忽略，但在没有解除阻塞之前不能忽略这个信号，因为进程仍有机会修改`handler`并解除忽略。
- `SIGQUIT`信号未产生过，一旦产生`SIGQUIT`信号将被阻塞，它的处理动作时用户自定义函数`sighandler`。

> 如果在进程解除对某信号的阻塞之前这种信号产⽣过多次,将如何处理?
>
> POSIX.1允许系统递送该信号⼀次或多次。Linux是这样实现的：常规信号在递达之前产⽣多次只计⼀次,⽽实时信号在递达之前产⽣多次可以依次放在⼀个队列⾥。我们暂时不讨论实时信号。

```C++
// 内核结构
struct task_struct
{
    ...
    /* signal handlers */
    struct sighand_struct *sighand;
    sigset_t blocked struct sigpending pending;
    ...

}

struct sighand_struct
{
    atomic_t count;
    struct k_sigaction action[_NSIG]; // #define _NSIG 64
    spinlock_t siglock;
};

struct __new_sigaction
{
    __sighandler_t sa_handler;
    unsigned long sa_flags;
    void (*sa_restorer)(void); /* Not used by Linux/SPARC */
    __new_sigset_t sa_mask;
};

struct k_sigaction
{
    struct __new_sigaction sa;
    void __user *ka_restorer;
};

/* Type of a signal handler. */
typedef void (*__sighandler_t)(int);
struct sigpending
{
    struct list_head list;
    sigset_t signal;
};
```

#### sigset_t

从上图来看，每个信号只有⼀个bit的未决标志，⾮0即1, 不记录该信号产生了多少次,阻塞标志也是这样表表示的。因此，未决和阻塞标志可以用相同的数据类型`sigset_t`来存储，`sigset_t`称为信号集, 这个类型可以表示每个信号的“有效”或“无效”状态，在阻塞信号集中“有效”和“无效”的含义是该信号是否被阻塞，⽽在未决信号集中“有效”和“无效”的含义是该信号是否处于未决状态。阻塞信号集也叫做当前进程的信号屏蔽字`Signal Mask`这里的“屏蔽”应该理解为阻塞而不是忽略。

#### 信号集操作函数

`sigset_t`类型对于每种信号⽤⼀个`bit`表⽰“有效”或“无效”状态, 至于这个类型内部如何存储这些`bit`则依赖于系统实现，从使用者的角度是不必关心的，使用者只能调⽤以下函数来操作`sigset_` t变量，而不应该对它的内部数据做任何解释，**比如用`printf`直接打印`sigset_t`变量是没有意义的。**

```C++
#include <signal.h>
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigaddset(sigset_t *set, int signo);
int sigdelset(sigset_t *set, int signo);
int sigismember(const sigset_t *set, int signo);
```

- 函数`sigemptyset`初始化`set`所指向的信号集，使其中所有信号的对应`bit`清零，表示该信号集不包含任何有效信号。
- 函数`sigfillset`初始化`set`所指向的信号集，使其中所有信号的对应`bit`置一，表示该信号集包含所有可能的有效信号，包括系统支持的所有信号。
- 注意,在使⽤`sigset_t`类型的变量之前，⼀定要调用`sigemptyset`或`sigfillset`做初始化，使信号集处于确定的状态。初始化`sigset_t`变量之后就可以在调`⽤sigaddset`和`sigdelset`在该信号集中添加或删除某种有效信号。

这四个函数都是成功返回0，出错返回-1。`sigismember`是⼀个布尔函数，⽤于判断⼀个信号集的有效信号中是否包含某种信号，若包含则返回1，不包含则返回0，出错返回-1。

#### sigprocmask

#### sigpending

## 捕捉信号









