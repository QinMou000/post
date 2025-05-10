> ![博客封面](https://raw.githubusercontent.com/QinMou000/pic/main/a46182e6318c4593a5c674f2bf9439d4.jpeg) 
>
> ✨✨所属专栏：[Linux](https://blog.csdn.net/2301_80194476/category_12799988.html)✨✨
>
> ✨✨作者主页：[嶔某](https://blog.csdn.net/2301_80194476?spm=1000.2115.3001.5343)✨✨

# Linux：线程同步与互斥

## 线程互斥

我们先明确几个概念

1. 临界资源：多线程执行流共享的资源，一个进程中所有线程都要访问的资源
2. 临界区：每个线程内部，访问临界资源的代码
3. 互斥：任何时候，互斥保证有且只有一个线程进入临界区执行访问临界资源的代码，对临界资源起保护作用
4. 原子性：不会被任何调度机制打断的操作，只有两种状态，要么完成了，要么没完成，不存在完成中、正在完成的状态

### 互斥量mutex

大部分情况，线程使用的数据都是局部变量，变量的地址都在线程地址空间内，变量归属单个线程，其他线程无法获得这种变量

但有些变量需要在线程间共享，称为共享变量，通过共享变量，完成线程间的交互

```C++
#include "mutex.hpp"
#include <pthread.h>
#include <unistd.h>

int ticket = 10000;

void *routine(void *args)
{
    Mutex *m = static_cast<Mutex *>(args);
    while (true)
    {
        // m->Lock();
        if (ticket > 0)
        {
            usleep(100);
            std::cout << "抢票一次" << std::endl;
            ticket--;
            std::cout << "剩余票数：" << ticket << std::endl;
            // m->UnLock();
        }
        else
        {
            // m->UnLock();
            break;
        }
    }
    return nullptr;
}

int main()
{
    Mutex mutex;

    pthread_t t1;
    pthread_create(&t1, nullptr, routine, (void *)&mutex);
    pthread_t t2;
    pthread_create(&t2, nullptr, routine, (void *)&mutex);
    pthread_t t3;
    pthread_create(&t3, nullptr, routine, (void *)&mutex);
    pthread_t t4;
    pthread_create(&t4, nullptr, routine, (void *)&mutex);

    while (true)
    {
        pthread_join(t1, nullptr);
        pthread_join(t2, nullptr);
        pthread_join(t3, nullptr);
        pthread_join(t4, nullptr);
    }

    return 0;
}
```

`注意：mutex.h头文件是提前封装好的`

上面的代码，在没有加锁的情况下，会出现票售多了的情况。

`if`语句判断条件为真后，代码可以并发的切换到其他线程，`usleep`就是在模拟这个漫长的业务，此时票数还没有减减，另外可能又有几个线程就又售出了多张票

而且tick--本身就不是一个原子操作，**我们认为，一条汇编代码是原子的**

```c#
取出ticket--部分的汇编代码
objdump -d a.out > test.objdump
152 40064b: 8b 05 e3 04 20 00 mov 0x2004e3(%rip),%eax #600b34 <ticket>
153 400651: 83 e8 01 sub $0x1,%eax
154 400654: 89 05 da 04 20 00 mov %eax,0x2004da(%rip) #600b34 <ticket>
```

`--`操作并不是原子的，对应了三条汇编指令

- `load`：将共享变量`ticket`从内存加载到寄存器中
- `update`：更新寄存器里面的值，执行`-1`操作
- `store`：将新值，从寄存器写回共享变量`ticket`的内存地址

要解决以上问题，需要三点

- 代码必须要有互斥行为：当代码进入到临界区执行时，不允许其他线程进入该临界区。
- 如果多个线程同时要求执行临界区代码，并且临界区没有线程在执行，那么只能允许一个线程进入该临界区
- 如果线程不在临界区中执行，那么该线程不能阻止其他线程进入临界区

OK，这时候就需要锁出场了。`Linux`中把这种锁叫做互斥量（互斥锁）

![image-20250510171855995](https://raw.githubusercontent.com/QinMou000/pic/main/image-20250510171855995.png)

**互斥量的接口**

初始化互斥量

> 静态分配
>
> ```C++
> pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER
> ```
>
> 动态分配
>
> ```C++
> int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);
> 	参数：
> 		mutex：要初始化的互斥量
> 		attr：NULL
> ```

销毁互斥量

> **注意：**
>
> 使用`PTHREAD_MUTEX_INITIALIZER`不需要销毁
>
> 不要销毁一个已经加锁的互斥量（该互斥锁已经有线程在使用中了）
>
> 已经销毁的互斥量，要确保后面不会有线程再次尝试加锁
>
> ```C++
> int pthread_mutex_destroy(pthread_mutex_t *mutex);
> ```

互斥量加锁和解锁

> ```C++
> int pthread_mutex_lock(pthread_mutex_t *mutex);
> int pthread_mutex_unlock(pthread_mutex_t *mutex);
> 	返回值：成功返回0，失败返回错误号
> ```
> **调用`pthread_mutex_lock`时可能会遇到以下情况：**
>
> **互斥量处于未锁状态，该函数会将互斥量锁定，同时返回0**
>
> **发起函数调用时，其他线程已经锁定互斥量，或者存在其他线程同时申请互斥量，但没有竞争到互斥量，那么`pthread_mutex_lock`调用会陷入阻塞（执行流被挂起）等待互斥量解锁后重新申请**

### 互斥量实现原理

















