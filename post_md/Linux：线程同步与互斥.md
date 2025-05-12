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

现在我们知道，`i++`和`++i`这种操作都不是原子的，会有数据一致性的问题。

为了实现互斥锁操作，大多数体系结构都提供了`swap`或者`exchange`指令，该指令的作用是把寄存器和内存单元的数据相交换，由于只有一条汇编指令，保证了原子性，即使是在多处理器平台上，访问内存的总线周期也有前后之分，一个处理器上的交换指令执行时另一个处理器的交换指令只能等待总线周期。现在我们把`lock`和`unlock`的伪代码改一下

```assembly
lock:
	movb $0, %al
	echgb %al, mutex
	if(al寄存器内容 > 0) {
		return 0;
	} else
		挂起等待;
	goto lock;
unlock:
	movb $1, mutex
	唤醒等待Mutex的线程;
	return 0;
```

### 互斥量的封装

```C++
// mutex.hpp
#pragma once
#include <iostream>
#include <unistd.h>
#include <pthread.h>
#include <mutex>

class Mutex
{
public:
    // 还可以删除不需要的拷贝构造和赋值重载
    Mutex()
    {
        pthread_mutex_init(&_mutex, nullptr);
    }
    void Lock()
    {
        pthread_mutex_lock(&_mutex);
    }
    void UnLock()
    {
        pthread_mutex_unlock(&_mutex);
    }
    pthread_mutex_t *Get() // 得到原始指针
    {
        return &_mutex;
    }
    ~Mutex()
    {
        pthread_mutex_destroy(&_mutex);
    }

private:
    pthread_mutex_t _mutex;
};

// RAII风格，进行锁管理
class global_mutex
{
public:
    global_mutex(Mutex &mutex)
        : _mutex(mutex)
    {
        _mutex.Lock();
    }
    ~global_mutex()
    {
        _mutex.UnLock();
    }
private:
    Mutex &_mutex;
};
```

**注意：带有`pthread.h`的源码编译时要链接`pthread`库**

```C++
// 这里我们所做的封装是模仿`C++11`的
std::mutex mtx;
std::lock_guard<std::mutex> guard(mtx);
```

## 线程同步

### 条件变量

当一个线程互斥地访问一个变量时，它必须要等到其他线程先把该变量修改之后才访问，那这个时候这个线程在其他线程访问之前什么也做不了。

例如在一个线程访问队列时，发现队列为空，因为没有其他线程往队列里塞数据，只能等待，等到队列里被其他线程塞了数据之后它才访问队列，这种情况就需要访问队列的线程在条件变量下等待，在其他线程塞完数据后通知该线程，然后该线程被唤醒，访问队列。

### 同步概念与竞态条件

同步`Synchronization`：在保证数据安全的前提下，让线程能够**按照某种特定的顺序**访问临界资源，从而有效避免饥饿问题

竞态条件`Race Condition`：是多线程或多进程编程中因并发执行导致的一种错误，当多个线程或进程同时访问和操作共享资源，且最终结果依赖于执行时序时，就会出现竞态条件。

### 条件变量函数

> 初始化
>
> ```C++
> int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr);
> 参数：
> 	cond：要初始化的条件变量
> 	attr：NULL
> ```

> 销毁
>
> ```C++
> int pthread_cond_destroy(pthread_cond_t * cond)
> ```

> 等待条件满足
>
> ```C++
> int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);
> 参数：
> 	cond：要在这个条件变量上等待
> 	mutex：互斥量，等待时释放锁
> ```

> 唤醒等待
>
> ```C++
> int pthread_cond_broadcast(pthread_cond_t *cond);
> int pthread_cond_signal(pthread_cond_t *cond);
> ```

### 生产消费者模型

**321原则**：三种关系，两种角色，一个交易场所

生产消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，通过中间的容器（如阻塞队列，循环队列）来通讯，所以生产者盛产完的数据不用等消费者处理，直接扔给容器，消费者也不找生产者要数据而是直接从容器里面取，这个容器就相当于一个缓冲区，平衡了生产者和消费者的处理能力，做到忙闲不均。中间的容器就是来对生产者和消费者做解耦的。

生产者消费者模型的优点在于**将生产者和消费者解耦**并且支持**多线程并发访问，支持忙闲不均**，而且消费者在拿到数据释放锁后在处理这个数据的时候，生产者也可以往容器里面生产数据。

![image-20250512222052030](https://raw.githubusercontent.com/QinMou000/pic/main/image-20250512222052030.png)

### 基于阻塞队列的生产消费模型

阻塞队列`BlockingQueue`:在多线程编程中，阻塞队列这种数据结构在实现生产者消费者模型中很常用。它与普通的队列区别在于，当队列为空时，从队列里面获取元素的操作会被阻塞，直到队列里被放的数据；当队列满时，往队列里放元素的操作也会被阻塞，直到有元素从队列中被取出。

![image-20250512222748368](https://raw.githubusercontent.com/QinMou000/pic/main/image-20250512222748368.png)

#### C++ queue模拟阻塞队列的生产消费模型

我们用的条件变量是经过封装之后的：

```C++
// cond.hpp
#pragma once
#include <iostream>
#include <pthread.h>
#include "mutex.hpp"

class Cond
{
public:
    Cond()
    {
        pthread_cond_init(&_cond, nullptr);
    }
    void Wait(Mutex &mutex)
    {
        pthread_cond_wait(&_cond, mutex.Get());
    }
    void Signal()
    {
        pthread_cond_signal(&_cond);
    }
    ~Cond()
    {
        pthread_cond_destroy(&_cond);
    }

private:
    pthread_cond_t _cond;
};
```

封装时不必将之前封装的`Mutex`引入成员变量，要将这两个模块解耦。`Mutex`和`Cond`基本上是一起创建的，将创建的`Mutex`传入`Cond`里面即可。这样可以让`Cond`更加具有通用性，可以传入其他类型的锁


```C++
// BlockQueue.hpp
#pragma once
#include "cond.hpp"
#include "mutex.hpp"
#include "thread.hpp"
#include <queue>
#include <iostream>

const int defaultnum = 10; // 默认队列长度

template <class T>
class BlockQueue
{
private:
    bool IsFull()
    {
        return _q.size() == _cap;
    }
    bool IsEmpty()
    {
        return _q.empty();
    }

public:
    BlockQueue(int cap = defaultnum)
        : _cap(cap),
          _psleep(0),
          _csleep(0)
    {
    }
    void Equeue(const T &in) // 生产者调用，往队列里面生产东西
    {
        {
            global_mutex gmutex(_mutex);
            while (IsFull()) // 如果队列里为满，那就让线程一直在条件变量下等待
            {
                _psleep++;
                std::cout << "生产者进入等待" << std::endl;
                Pcond.Wait(_mutex);
                _psleep--;
            }
            // 队列有空位置
            _q.push(in);
            std::cout << "入队列" << std::endl;
            if (_csleep > 0) // 通知消费者来消费
            {
                std::cout << "通知消费者来消费" << std::endl;
                Ccond.Signal();
            }
        }
    }
    T Pop() // 消费者调用
    {
        T out;
        {
            global_mutex gmutex(_mutex); // 利用类内的锁创建可以自动解锁的锁
            while (IsEmpty())            // 如果队列里为空，那就让线程一直在条件变量下等待
            {
                _csleep++;
                std::cout << "消费者进入等待" << std::endl;
                Ccond.Wait(_mutex);
                _csleep--;
            }
            out = _q.front();
            _q.pop();
            std::cout << "出队列" << std::endl;

            if (_psleep > 0) // 通知生产者去生产
            {
                std::cout << "通知生产者去生产" << std::endl;
                Pcond.Signal();
            }
        }
        return out;
    }
    ~BlockQueue()
    {
    }

private:
    std::queue<T> _q; // 可以用vector代替，都看心情~
    int _cap;         // 队列容量

    Mutex _mutex; // 为了维护p与c，p与p，c与c之间的互斥关系
    Cond Pcond;   // 维护p与p之间的同步
    Cond Ccond;   // 维护c与c之间的同步

    int _psleep; // 生产者休眠数
    int _csleep; // 消费者休眠数
};
```

### 为什么`pthread_cond_wait`需要互斥量

在使用条件变量（Condition Variable）时，`wait()` 和 `signal()` 操作必须在锁的保护下进行，这是由条件变量的核心设计目标决定的 ——**安全地等待和通知共享状态的变化**。以下是详细解释：

1. **wait () 为什么要加锁？**

（1）**原子性释放锁并进入等待状态**

- `wait()` 的核心逻辑是：**释放锁 → 进入阻塞 → 被唤醒后重新获取锁**。

- 如果这个过程不是原子的，会导致竞态条件。例如：

  ```python
  # 错误示例（无原子性）
  lock.release()  # 释放锁
  # 此时另一个线程可能修改共享状态并发出通知，但当前线程尚未阻塞
  condition.wait()  # 可能错过通知，永久阻塞
  ```

- **正确做法**：通过锁保证释放锁和阻塞操作的原子性，确保线程在释放锁后立即进入等待状态，不会错过其他线程的通知。

（2）**保护共享状态的可见性**

- 线程在调用 `wait()` 前通常需要检查某个条件（如队列是否为空），这个检查必须在锁的保护下进行，以确保看到最新的共享状态。

- 示例：

  ```python
  with lock:
      while not condition_met:  # 在锁的保护下检查条件
          condition.wait()  # 原子释放锁并等待
      # 条件满足后，自动重新获取锁，继续执行
  ```

2. **signal ()为什么要加锁？**

（1）**确保通知操作的原子性**

- `signal()` 操作需要修改条件变量的内部状态（如唤醒队列），如果多个线程同时调用 `signal()`，可能导致唤醒操作丢失或重复唤醒。
- **锁的作用**：保证 `signal()` 操作的原子性，避免竞态条件。

（2）**与 wait () 的锁保持一致**

- 如果`wait()`**和**`signal()`使用不同的锁，会导致：
  - `wait()` 释放的锁与 `signal()` 操作的锁无关，无法正确同步。
  - 共享状态的修改和检查可能使用不同的锁，破坏一致性。

关键点：

- **生产者**在锁内修改队列并通知，确保消费者看到最新状态。
- **消费者**在锁内检查队列状态，若为空则原子释放锁并等待，被唤醒后重新获取锁继续执行。

#### 条件变量使用规范

> 等待条件代码：
>
> ```C++
> pthread_mutex_lock(&mutex);
> while(条件为假)
> 	pthread_cond_wait(cond, &mutex);
> 修改条件
> pthread_mutex_unlock(&mutex);
> ```

> 给条件发送信号代码：
>
> ```
> pthread_mutex_lock(&mutex);
> 将条件变为真
> pthread_cond_signal(cond);
> pthread_mutex_unlock(&mutex);
> ```

## POSIX信号量





