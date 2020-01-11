---
title: Python 多进程与多线程
date: 2019-03-03 23:15:15
tags: [Python,多进程,多线程]
categories: Python
---

在多核 CPU 时代，使用多进程和多线程能够充分利用 CPU 多核性能来提高程序的执行效率。 本文将着重介绍 Python 多进程和多线程的区别和应用场景选取。

<!--more-->

## 基础知识

### 进程
进程是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位。每个应用程序都有一个自己的进程，提供执行程序所需的所有资源。

每个进程启动时都会最先产生一个线程，即主线程。然后再由主线程创建其他线程。进程是并行的，即同一时刻可以运行多个进程。

### 线程
线程是操作系统进行运算调度的最小单位，被包含在进程之中，是进程的实际运作单位。

一条线程是指进程中一个单一顺序的控制流，一个进程中有多个线程，每条线程并发执行不同的任务。线程是并发的，存在交替执行的情况。

### 进程和线程的区别
一个程序至少有一个进程，一个进程至少有一个线程。

1. 根本区别：进程是资源分配的基本单位，线程是程序执行的最小单位
2. 开销方面：每个进程有独立的代码和数据空间，每启动一个进程，系统会为他分配地址空间，进程间的切换有较大开销；同一类线程共享代码和数据空间，切换和创建的开销较小
3. 线程通信更加方便，同一进程下的线程共享全局变量、静态变量等数据；而进程需要以通信方式(IPC)进行
4. 由于进程有自己独立的地址空间，因此多进程程序更加健壮；而多线程程序有一个线程挂掉，全部线程都会挂掉

### 进程通信与线程通信
线程通信： 同一进程下的线程共享相同的数据空间，可以直接通信。 需要做好同步/互斥 mutex，保护共享的全局变量。

进程通信： 需要通过操作系统，以 IPC 方式进行。
1. 管道： 是半双工通信方式，数据只能单向流动，只能在父子进程或者兄弟进程中使用。 
2. 命名管道： 是半双工的通信方式，但允许无亲缘关系进程间的通信。 命名管道是一种 FIFO 对象，常用于客户端-服务器通信。
3. 消息队列： 消息队列是消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。
4. 信号量： 是一个计数器，用于为多个进程提供对共享数据对象的访问。
5. 共享内存： 映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。共享内存是最快的 IPC 方式。需要使用信号量用来同步对共享存储的访问。
6. 套接字： 与其它通信机制不同的是，它可用于不同机器间的进程通信

### 协程
协程 Coroutine：又称微线程，纤程。是一种程序组件，协程看上去是子程序（函数），但在执行过程中在子程序内部可以中断，转而执行其他子程序（不是调用），在适当的时候再返回来执行。  

特点：只有一个线程执行。  

优势：执行效率高，由于子程序切换不是由线程，而是由程序自身控制，因此没有线程切换的消耗；不需要多线程的锁机制，由于只有一个线程，不存在同时写变量冲突，协程中控制共享资源不加锁。  
python generator 的 yield 可以在一定程度上实现协程。

- 线程由操作系统控制
- 协程由程序自身控制

## Python 多线程
Python 标准库提供了 `_thread` 和 `threading` 两个模块进行多线程操作，`_thread` 是低级模块，`threading` 是高级模块，对 `_thread` 进行了封装， 因此一般开发中只需使用 `threading` 模块。

### 启动线程
启动线程有两种方法

1. 直接使用 threading.Thread()，把一个函数传入并创建 Thread 实例，然后调用 start() 开始执行。

```python
import threading

# 这个函数名可随便定义
def run(n):
    print('thread %s is running...' % threading.current_thread().name)
    while n:
        n -= 1
    print('thread %s ended' % threading.current_thread().name)

if __name__ == "__main__":
    print('thread %s is running...' % threading.current_thread().name)
    t1 = threading.Thread(target=run, args=(1000000,), name="thread1")
    t2 = threading.Thread(target=run, args=(10,), name = "thread2")
    t1.start()
    t2.start()
    # t1.join() # 使主线程在子线程结束后再退出
    # t2.join()
    print('thread %s ended' % threading.current_thread().name)
```

结果：
```html
thread MainThread is running...
thread thread1 is running...
thread thread2 is running...
thread thread2 ended
thread MainThread ended
thread thread1 ended
```

任何进程都会默认启动一个线程，叫做主线程，主线程又可以启动新的线程。 threading 模块中的 `current_thread()` 方法可以返回当前执行的实例。 主线程实例名字为 `MainThread`，子线程名字可以在创建时指定。

我们注意到上面的结果中，主线程 MainThread 结束后，子线程 thread1 仍在运行，可以通过 `join()` 方法进行线程合并。 join() 函数执行顺序是逐个执行每个线程，执行完毕后继续往下执行，能够使主线程在子线程结果后再退出。

2. 继承 threading.Thread 来定义线程类，重写 run 方法

```python
import threading

class MyThread(threading.Thread):
    def __init__(self, n):
        super(MyThread, self).__init__() # 重构 run() 函数必须写
        self.n = n

    def run(self):
        print("thread %s is running..." %self.n)

if __name__ == "__main__":
    t1 = MyThread("thread1")
    t1.start()
```

### Lock
由于线程之间数据共享，当有多个线程对同一个共享数据进行操作，就可能把数据改乱，因此需要考虑线程安全问题。 

threading 模块中定义了 `Lock` 类，提供**互斥锁**的功能来保证多线程情况下数据的一致性。

Lock 锁的使用
```python
lock = threading.Lock() # 创建锁
lock.acquire([timeout]) # 锁定，可以设置 timeout，在超时后通过返回值可以判断是否得到了锁
lock.release() # 释放
```

定义一个共享变量 balance，初始值为 0，创建两个线程进行操作，理论上结果应该是 0。 但实际上，如果循环的次数多的话，最终结果不一定会是 0。

原因是: 在操作系统中，高级语言的一条语句在 CPU 中执行其实是若干条语句。  
就比如 `balance = balance + n`，会先计算出 balance + n 并将结果存入临时变量中， 再将临时变量的值赋给 balance。 因此，多个线程同时修改 balance 的时候，就可能把它改乱。

解决方法： 给 change() 加锁，当 thread1 执行 change() 时，该线程获得锁，那么其他线程就不能执行 change()，只能等待锁释放。 
```python
import threading

balance = 0 # 假设这是存款
lock = threading.Lock()

def change(n):
    global balance
    balance += n
    balance -= n

def run(n):
    for i in range(10000000):
        lock.acquire()
        try:
            change(n)
        finally:
            lock.release()

if __name__ == "__main__":
    t1 = threading.Thread(target=run, args=(5, ))
    t2 = threading.Thread(target=run, args=(8, ))
    t1.start()
    t2.start()
    t1.join()
    t2.join()
```

使用锁能够确保某段关键代码只由一个线程从头到尾完整执行，而缺点是阻止了多线程并发执行，效率降低。

### GIL
对于其他语言，CPU 是多核时可以支持多个线程同时执行，但 python 在设计时有 GIL（Global Interpreter Lock）全局解释锁，导致无论是单核还是多核，只能同时允许一个线程执行，无法利用多线程实现多核任务。GIL 锁只在 Cpython 中存在。

在一个 python 进程中，GIL 锁只有一个，某个线程想要执行，就必须拿到 GIL。在 python3 中，GIL 使用计时器，当执行时间达到阈值时，当前线程就释放 GIL 锁。

多核多线程比单核多线程更差，因为在单核下多线程，每次释放 GIL，唤醒的那个线程都能获取到 GIL 锁，能够无缝执行。
但多核下，CPU0 释放 GIL 后，其他 CPU 上的线程都会进行竞争，但 GIL 可能会马上又被 CPU0 拿到，导致其他几个 CPU 上被唤醒后的线程会醒着等待到切换时间后又进入待调度状态，这样会造成线程颠簸(thrashing)，导致效率更低

python 可以用多进程实现多核任务，多个 python 进程有各自独立的 GIL 锁，互不影响。


## Python 多进程

### fork()
在 linux/unix 中，可以使用 fork() 调用实现多进程。fork() 函数通过系统调用创建一个与原来进程几乎完全相同的进程，相当于把当前进程（父进程）复制了一份（子进程）。

fork() 函数的特性在于： 调用一次，会返回两次，是父进程和子进程在各自的地址空间返回，可能有三种不同的返回值。
1. 如果成功创建子进程，在父进程中，返回子进程的 ID
2. 如果成功创建子进程，在子进程中，返回 0
3. 如果创建失败，返回 -1

python 的 os 模块封装了 fork() 方法，子进程调用 getppid() 可以得到父进程的 ID, getpid() 是得到当前进程。 要注意的是，在 windows 中没有 fork() 方法.

子进程是在 fork 之后开始向下执行，而不是从头开始执行。

```python
import os

print('process %s start.' %os.getpid())
pid = os.fork()
if pid == 0:
    print('I am child process %s, my parent is %s.' %(os.getpid(), os.getppid()))
else:
    print('I am %s, I created a child process %s.' %(os.getpid(),pid))

```

```
process 26601 start.
I am 26601, I created a child process 26602.
I am child process 26602, my parent is 26601.
```

### multiprocessing
要实现跨平台的多进程，可以使用 multiprocessing 模块，提供一个 Process 类来代表一个进程对象。

Process 类与 Thread 类相似，有两种使用方法：

1. 直接使用 Process

创建子进程实例时，只需要传入执行函数和函数参数，start() 方法启动子进程，join() 会等待子进程执行完毕，用于进程同步。

```python
import multiprocessing

p = multiprocessing.Process(targte=fun, args=(1,2,))
p.start()
p.join()
```

2. 继承 Process 来自定义进程类，重写 run 方法

```python
import multiprocessing
import os

class MyProcess(multiprocessing.Process):
	def __init__(self, name):
		super(MyProcess, self).__init__()
		self.name = name
	def run(self):
		print(os.getpid(), self.name)
if __name__ == '__main__':
	for name in ['1', '2', '3']:
		p = MyProcess(name)
		p.start()
		p.join()
```

### 进程池 Pool
进程池 Pool 可以用来批量创建子进程，对 Pool 对象调用 join() 方法会等待所有子进程执行完毕，之前必须先调用 close() 方法，调用 close() 后就不能再继续添加新进程。

Pool 常用方法

|方法|含义|
|:-|:---:|
|apply()|同步执行（串行）|
|apply_async()|异步执行（并行）|
|terminate()|立刻关闭进程池|
|close()|等待所有进程结束后，才关闭进程池|
|join()|主进程等待所有子进程执行完毕，必须在 close() 或 terminate() 之后用|

```python
import multiprocessing

def fun(arg1, agr2):
	pass

size = multiprocessing.cpu_count()
pool = multiprocessing.Pool(processes = size) # 不指定的话，pool 的默认大小为 CPU 核数
for i in range(100):
	pool.apply_async(fun, args=(1, 2, ))
pool.close()
pool.join()
```

### python 中的进程通信
进程之间不共享数据， 进程间需要通信的话可以使用 multiprocess 的 Queue, Pipes 等方式来交换数据。

#### multiprocess.Queue
Queue 是多进程安全的队列，可以实现多进程之间的数据传递。主要有 put 和 get 两个函数。put() 用于插入数据到队列中，get() 是从队列中读取并删除一个元素。

### 子进程 subprocess
subprocess 能够方便地启动一个子进程，并控制输入输出。可以用于替换 os.system, os.popen 等方法。

#### subprocess.Popen 类
```python
subprocess.Popen(args, stdin=None, stdout=None, stderr=None, shell=False, executable=None, ...)
```
创建并返回一个子进程，并在子进程中执行制定的程序。

- args： 必填，要执行的命令或可执行文件的路径，及传给程序的参数
- stdin： 子进程的标准输入
- stdout： 子进程的标准输出，可以制定输出到文件
- stderr： 子进程的标准错误输出
- shell： True 则指定使用 shell 运行程序
- executable： 指定子进程在什么 shell 中运行，默认为 /bin/sh

```python
import subporcess
p = subprocess.Popen(cmd, shell=True, stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
stdout, stderr = p.communicate()
print(stdout)
```

#### subprocess.PIPE
可以被用于 Popen 的 stdin, stdout, stderr 三个参数的特殊值，表示需要创建一个新的管道。

#### communicate()
```p.communicate(input=None)``` 
和子进程 p 交流，将 input 的数据发送到子进程的 stdin 中，并同时读取子进程的 stdout 和 stderr。

需要注意的是： communicate() 只能通过管道和子进程通信，也就是需要设置 subprocess.PIPE； 且 communicate() 会立即阻塞父进程，直至子进程结束。

## 选择多进程还是多线程
应用程序可以分为`CPU 密集型`和`IO 密集型`两种，选择多进程还是多线程来执行程序，首先需要看程序属于哪种类型。

### CPU 密集型    
- 也叫计算密集型任务，特点：需要进行大量判断，主要消耗 CPU 资源，大部分时间用于计算、逻辑判断等 CPU 动作的程序，如计算圆周率、视频高清解码等；  
- python 这种脚本语言不适合计算密集型任务，最好用 C 语言；
- python CPU 密集型任务用多进程模型。
  
### IO 密集型
- 涉及到网络、磁盘 IO 的任务是 IO 密集型任务，特点：CPU 消耗较少，大部分时间在等待 IO 操作完成（IO 操作速度远低于 CPU 和内存的速度）， 如 web 应用、文件处理、爬虫； 线程 A 在进行 IO 等待时可以切换到线程 B 执行，多线程可以利用 IO 阻塞等待时的空闲时间执行其他线程，提升效率。
- IO 密集型任务最合适的语言是开发效率最高（代码量最少）的语言，脚本语言是首选；
- python IO 密集型任务用多线程模型，多线程只使用一个 CPU 核心。
- io 操作不占用 CPU（从硬盘、从网络、从内存读数据都算 io）

---
## 参考
[[1] 进程和线程](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014319272686365ec7ceaeca33428c914edf8f70cca383000)

[[2] Python 多进程与多线程](https://www.jianshu.com/p/a69dec87e646)
