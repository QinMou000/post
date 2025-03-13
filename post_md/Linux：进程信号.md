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



- 执行该信号的默认处理动作



- 提供一个信号处理函数，要求内核在处理该信号是切换到用户态执行这个处理函数，这种方式称为自定义捕捉`catch`信号

接下来我们将从产生信号，保存信号，捕捉信号三个方面来具体总结。

## 产生信号

### 通过终端按键产生信号

基本操作：

- `Ctrl + C`向前台进程发送`SIGINT`信号
- `Ctrl + \`发送终止信号`SIGQUIT`并生成core dump文件，用于事后调试。
- `Ctrl + Z`发送停止信号`SIGTSTP`将当前前台进程挂起到后台等待。

ok这里就有一个问题了。键盘按下对应组合键是如何使进程进行对应操作的呢？键盘等硬件是直接或间接的与`CPU`上的针脚连接的，当按键按下，硬件发送一个中断给`CPU`，`CPU`识别到中断信息（高电平）然后就去执行处理硬件数据的代码。从操作系统来看就是`OS`停下当前工作将数据从硬件读取到内存。

那么，信号就是从纯软件的角度来模拟硬件中断。硬件中断是发给`CPU`软中断是发给进程。两者在思想上是完全一致的。

### 调用系统命令向进程发送信号

示例代码

```

```



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

```
// 实现自己的kill命令
```



#### raise

`raise`函数可以给当前进程发送指定的信号，也就是给自己发信号。

```
NAME 
	raise
```



#### abort

`abort`函数使当前进程收到信号而异常终止，它总会成功的就像`exit`一样

```
NAME 
	abort
```



### 由软件产生信号

`SIGPIPE`和`SIGALRM`信号是一种由软件产生的信号，管道我们已经学过了。现在来学习时钟信号`alarm`函数

```
NAME
	alarm
```



### 硬件异常产生信号



## 保存信号



## 捕捉信号









