---
title: Python 协程
date: 2019-08-25 10:40:02
tags: [Python, 协程]
categories: Python
---

协程 Coroutine，也叫微线程，是一种比线程更加轻量级的存在，一个线程可以有多个协程。

<!--more-->

## 协程概述
协程的执行过程中在子程序内部可以中断，转而执行其他子程序（不是调用），在适当的时候再返回来执行。  

**特点** ：只有一个线程执行。  

**优势**
1. 执行效率高，由于子程序切换不是由线程，而是由程序自身控制，因此没有线程切换的消耗
2. 不需要多线程的锁机制，由于只有一个线程，不存在同时写变量冲突，协程中控制共享资源不加锁。  

- 线程由操作系统控制
- 协程由程序自身控制

## python 实现协程
python 中实现协程大概经过三个阶段
1. 最初的生成器变形 yield/send 
2. 在 python3.4 中引入 @asyncio.coroutine 和 yield from 
3. 在 python3.5 中引入 async/await 关键字

### yield/send 
python 生成器的 yield 可以在一定程度上实现协程。

传统的生产者/消费者模型是一个线程写消息，一个线程取消息，通过锁机制控制队列和等待，但容易造成死锁。

使用协程，在生产者生产消息后，直接通过 yield 跳转到消费者开始执行，待消费者执行完毕后，切换回生产者继续生产，效率高。

```python
def consume():
    while True:
    	# 等待接收数据
        num = yield
        print('consuming ', num)

consumer = consume()
next(consumer) # 得先 next 才能 send，否则会报错，也可以用 consumer.send(None)
for n in range(100):
    print('start producting ', n)
    consumer.send(n)
```

主线程中创建了一个 consumer 协程，并在主线程中产生数据，协程中消费数据，输出：
```
start producting  0
consuming  0
start producting  1
consuming  1
...
```

### asyncio/yield from
先来看 yield 和 yield from 的区别：
```python
def generator1():
    yield range(5)

def generator2():
    yield from range(5)


g1 = generator1()
g2 = generator2()
for x in g1:
    print(x)
# range(0, 5)

for x in g2:
    print(x)
# 0
# 1
# 2
# 3
# 4
```
range(5) 是一个可迭代对象，yield 和 yield from 后边接可迭代对象时，yield 是 直接 yield 的是可迭代对象，而 yield from 是将可迭代对象中的元素一个一个yield出来。

asyncio 是一个基于事件循环的实现异步 IO 的模块，使用 @asyncio.coroutine 可以把一个 generator 标记为 coroutine 类型，然后在 coroutine 内部用 yield from 调用另一个 coroutine 实现异步操作。

### async/await
在 python 3.5 以后， async/await 成为了实现协程更好的替代方案。
async/await 让协程表面上独立于生成器存在，将细节都隐藏于 asyncio 模块下，语法更清晰明了。

在一个普通函数前加 async 关键字，可以将其变成协程。注意，async 无法将一个生成器转化为协程。

```python
import asyncio, random
async def my_coroutine(alist):
    while len(alist) > 0:
        c = random.randint(0, len(alist)-1)
        print(alist.pop(c))
        await asyncio.sleep(1)

if __name__ == '__main__':
    loop = asyncio.get_event_loop() #运行协程用事件循环，	可以看到交替执行的结果
    strs = ["a", "b", "c"]
    ints = [1, 2, 3]
    c1 = my_coroutine(strs)
    c2 = my_coroutine(ints)
    tasks = [c1, c2]
    loop.run_until_complete(asyncio.wait(tasks))
    print('All task finished.')
    loop.close()
```

输出结果：
```
1
a
3
c
2
b
All task finished.
```

## 参考
[理解Python协程:从yield/send到yield from再到async/await](https://blog.csdn.net/soonfly/article/details/78361819)

