---
title: Python 单例模式
date: 2019-08-05 17:33:26
tags: [Python, 设计模式]
categories: Python
---

单例模式是确保某个类只有一个实例存在，保证了在程序的不同位置都可以且仅可以取到同一个对象实例。如果实例不存在，会创建一个实例；如果已存在就会返回这个实例。

<!--more-->

## 单例模式特点
只有一个实例、单例类的构造函数是私有的，自行创建类的实例、向整个系统公开这个实例接口

## 单例模式使用场景
1. Python 的 logger 就是一个单例模式，用以日志记录
2. Windows 的资源管理器是一个单例模式
3. 线程池，数据库连接池等资源池一般也用单例模式
4. 网站计数器

为什么不用全局变量？
全局变量不能保证应用程序只有一个实例，且可能会有名称空间的干扰，如果有重名的可能会被覆盖，不能继承

## python 实现单例模式
python 有多种方式来实现单例模式： 模块、`__new__`、 装饰器、 元类 metaclass

### 使用模块
**python  的模块就是天然的单例模式**，在模块第一次导入时，会生成 `.pyc` 文件，之后的导入就会加载 `.pyc` 文件，而不会再次执行模块代码。

如果想创建一个单例类，可以新建一个 mysingleton.py 文件，也就是一个模块
```python
# mysingleton.py
class My_Singleton(object):
    def foo(self):
        pass
my_singleton = My_Singleton()
```
在其他文件中，只要导入上满这个模块中的实例
```python
from mysingleton import my_singleton

my_singleton.foo()
```


### 使用 `__new__` 方法
为了使类只能出现一个实例，可以用 `__new__` 来控制实例的创建过程，将类的实例和一个类变量 `_instance` 关联起来，如果 `cls._instance` 为 None 则创建实例，否则直接返回 `cls._instance`。

```python
class Singleton(object):
    _instance = None
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = object.__new__(cls)
        return cls._instance
    def __init__(self):
        pass

s1 = Singleton()
s2 = Singleton()
print(id(s1) == id(s2)) # True,说明是同一个实例
```

但是当有多个线程同时去初始化对象时，就很可能同时判断 `_instance is None`，无法实现单例。这种情况下，需要用同步锁来解决问题。
```python
from synchronize import make_synchronized
class Singleton(object):
    instance = None
    @make_synchronized # 用装饰器
    def __new__(cls, *args, **kwargs):
    	pass
```

### 使用装饰器
可以使用装饰器来修饰某个类，使其只能生成一个实例。

1. 定义一个装饰器 singleton，它返回了一个内部函数 getinstance，该函数会判断某个类是否在字典 instances 中。
2. 如果不存在，将 `cls` 作为 key，`cls(*args, **kw)` 作为 value 存到 instances 中，否则，直接返回 `instances[cls]`。

```python
from functools import wraps
def singleton(cls):
    instances = {}
    @wraps(cls)
    def getinstance(*args, **kw):
        if cls not in instances:
            instances[cls] = cls(*args, **kw)
        return instances[cls]
    return getinstance
@singleton
class MyClass(object):
    a = 1
```

### 使用元类
元类（metaclass）可以控制类的创建过程，它主要做三件事：
1. 拦截类的创建
2. 修改类的定义
3. 返回修改后的类

```python
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]

# Python3
class MyClass(metaclass=Singleton):
   pass
```