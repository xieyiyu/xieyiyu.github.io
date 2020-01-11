---
title: Python 迭代器与生成器
date: 2019-08-23 13:06:43
tags: [Python]
categories: Python
---

在 python 中，生成器 generator 可以实现一遍循环一遍计算，是一种惰性计算，也就是在真正需要的时候才计算结果，这样可以节省空间，避免不必要的计算，从而提升性能。

<!--more-->

## 迭代器
可迭代对象 Iterable： 能够直接作用于 for 循环的对象，如 list、tuple、dict、set、str、file、generator
迭代器 Iterator： 可以被 next() 函数调用并不断返回下一个值的对象

生成器是 Iterator 对象，但 list、 dict、 str 虽然都是 Iterable，但不是 Iterator， 可以用 iter() 方法将其变成 Iterable 类型。

Python 的 Iterator 对象表示的是一个数据流，Iterator 对象可以被 next() 函数调用并不断返回下一个数据，直到没有数据时抛出 StopIteration 错误。可以把这个数据流看做是一个有序序列甚至是无限大的数据流，只能不断通过 next() 函数实现按需计算下一个数据，所以 Iterator 的计算是惰性的，只有在需要返回下一个数据时它才会计算。

### 迭代器协议
如果一个容器类提供了 `__iter__()` 方法，并且该方法能返回一个能够逐个访问容器内所有元素的迭代器，则我们说该容器类实现了迭代器协议。

for 循环的本质是不断调用 next()。

Python 处理 for 循环时，首先会调用内建函数 iter(something)，它实际上会调用 `something.__iter__()`，返回 something 对应的迭代器。而后，for 循环会调用内建函数 next()，作用在迭代器上，获取迭代器的下一个元素，并赋值给 x。此后，Python 才开始执行循环体。

## 生成器
迭代器是一个更加抽象的概念，生成器是一个迭代器对象，是创建迭代器的简单而强大的工具。

python 中提供两种方式来构造生成器： 生成器表达式和生成器函数。

### 生成器表达式
将列表的 [] 改为 ()，就是一个生成器表达式，生成器需要通过调用 next(g) 来获得其下一个返回值，直到计算到最后一个元素，没有更多的元素时，抛出 StopIteration 的错误。
```python
l = [x*x for x in range(5)]
print(l) # [0, 1, 4, 9, 16]，使用列表推导，会一次产生所有结果

g = (x*x for x in range(10))
print(g) # <generator object <genexpr> at 0x000002590811D830>
print(next(g)) # 0
print(next(g)) # 1
```

实际上我们在用生成器时，并不会去调用 next()，因为生成器实现了迭代器协议，是一个可迭代对象，因此可以通过 for 循环来迭代，不用考虑 StopIteration 的异常。如果生成器推导较复杂，则可以通过生成器函数来完成。

### 生成器函数
带有 yield 关键词的函数是一个生成器函数，可以用于迭代。 yield 类似于 return，迭代一次遇到 yield 就返回 yield 后面的值，并记住这个返回位置，下一次迭代就从遇到 yield 的下一行开始执行。注意，**每次从暂停恢复时，生成器函数的内部变量、指令指针、内部求值栈等内容和暂停时完全一致。**

与普通函数不同，生成器函数被调用后，其函数体内的代码不会立即执行，而是返回一个生成器。当生成器调用成员方法时，相应生成器函数中的代码才会执行。

```python
def my_generator():
    for i in range(4):
        yield i ** 2
g = my_generator() # <class 'generator'>
# for x in g:
#     print(x) # 0,1,4,9
print(next(g)) # 0
print(g.__next__()) # 1
```

### generator.send(value)
generator.send(value) 方法可以将上一次被挂起的 yield 表达式的值设置为 value，也就可以与生成器函数进行通信，这是使用 yield 在 python 中使用协程的基础。

```python
def my_generator():
    for i in range(4):
        y = yield i ** 2
        print(y)
g = my_generator() # <class 'generator'>
print(next(g)) # 0
print(g.send(30)) # 先是 y = 30，再 print(30)，接着再继续 for 循环， i == 1，yield 会返回 1，因此打印 1
print(next(g)) # 因为会从上一次 yield 的下一行开始执行，也就是 print(y)，但是这时候 y 并没有值，因此会打印 None，接着再继续 for 循环，i == 2
```

执行结果为：
```
0
30
1
None
4
```

注意： 在一个生成器对象没有执行 next 之前，由于没有 yield 被挂起，那么此时如果执行 send 会报错。

## 参考
[Python 中的黑暗角落（一）：理解 yield 关键字](https://liam.page/2017/06/30/understanding-yield-in-python/)


