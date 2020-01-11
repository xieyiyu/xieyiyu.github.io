---
title: Python 装饰器
date: 2019-08-19 12:37:02
tags: [Python]
categories: Python
---

python 中的函数与 java、c++ 中不同之处在于： python 中的函数可以作为变量当做参数传入另一个函数中。装饰器 Decorator 本质上就是一个 python 函数，可以让其他函数在不需要任何代码变动的前提下增加额外功能。  

<!--more-->

有了装饰器，就可以抽离出大量与函数本身功能无关的雷同代码到装饰器中并继续重用，也就是能够给函数增加额外的功能。装饰器的返回值是一个函数对象。

常用场景：插入日志、性能测试、事务处理、缓存、权限校验等。 

首先来看一个简单的例子： 如果有两个函数 foo1 和 foo2 都要实现在进入函数主体前，先处理日志的功能，那么我们可能会新定义一个函数 use_logging 来专门处理，这样 foo1 和 foo2 就只需要调用 use_logging，其他函数有需要也可以重用。

```python
import logging

def use_logging(func):
    logging.warning("%s is running" % func.__name__) # logging 的默认设置 warn 会输出，info 不会
    func()

def foo1():
    print('i am foo1')

use_logging(foo1)
# WARNING:root:foo1 is running
# i am foo1
```

这样看起来达到了我们的目的，实现了函数运行前先处理日志的功能，但是每次我们调用的是 use_logging 函数，而真正的业务逻辑是在 foo1 中，这样破坏了原有的代码结果，我们希望是能够直接调用 foo1 函数，这就可以用到装饰器来实现。

### 简单的装饰器
```python
import logging

def use_logging(func):
    def wrapper():
        logging.warning("%s is running" % fun.__name__)
        return func()
    return wrapper

def foo1():
    print('i am foo1')

f = use_logging(foo1) # use_logging 是一个装饰器，返回的是 wrapper，这里相当于是 f=wrapper
f() # 也就是执行 wrapper()，然后返回 func()，就是执行 foo1()
```

在这里 use_logging 装饰器把执行真正业务逻辑的函数 func 包裹在里面，也就是 foo1 被 use_logging 所装饰。 在这个例子中，函数进入和退出时 ，被称为一个横切面，这种编程方式被称为面向切面的编程。

### 使用语法糖 @
语法糖： 指计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用。通常来说使用语法糖能够增加程序的可读性，从而减少程序代码出错的机会。

@ 符号是装饰器的语法糖，放在函数定义之前，这样就不需要赋值，直接调用 foo1() 即可，这样 foo1() 不需要做任何修改，就可以增加额外的功能
```python
@use_logging
def foo1():
	print('i am foo1')
foo1()
```

### \*args、 \*\*kwargs
如果业务逻辑函数 foo1 需要带参数，就可以给装饰器里的 wrapper 带上同样的参数，但可能有多个带有不同参数的函数需要使用装饰器 use_logging 的话，就可以使用 `*args，**kwargs` 来定义，这样不管 foo1 带有多少个参数，都可以完整的传到 func 中

```python
def use_logging(func):
	def wrapper(*args, **kwargs):
		logging.warning("%s is running" % func.__name__)
		return func(*args, **kwargs) # 可以让其用于任何函数，无论参数形式如何
	return wrapper
```

### 带参数的装饰器
在上面的装饰器中，我们只给装饰器传入了一个参数，也就是要执行的业务逻辑函数 func，但如果我们在调用装饰器时，想要加上其他参数，比如可以给日志定义级别，因为不同业务逻辑需要的日志级别可能不同，这时我们就可以给装饰器加上参数。

```python
import logging

def use_logging(level):
    def decorator(func):
        def wrapper(*args, **kwargs):
            if level == 'warn':
                logging.warning("%s is running" % func.__name__)
            elif level == 'error':
                logging.error("%s has error" % func.__name__)
            return func(*args, **kwargs)
        return wrapper
    return decorator

@use_logging(level="warn")
def foo1(name, msg):
    print('i am %s, %s' %(name, msg))

foo1('foo1', 'hello!')
```

这个带参数的装饰器实际上只是在原有装饰器的基础上再套上一层，是对原有装饰器的一个函数封装，并返回一个装饰器。

### 类装饰器
相比于函数装饰器，类装饰器具有灵活度大、高内聚、封装性等优点。

类装饰器使用类的 `__call__` 方法，当使用 @ 形式将装饰器附加到函数上时，就会调用此方法。

```python
class Decor(object):
    def __init__(self, func):
        self._func = func

    def __call__(self):
        print ('class decorator runing')
        self._func()
        print ('class decorator ending')

@Decor
def foo():
    print ('foo')

foo()
```

### functools.wraps
使用装饰器会使得原函数的一些元信息消失，比如 `__name__`, `__doc__` 会被装饰器的替代。比如之前的装饰器在最后打印 `print(foo1.__name__)`，得到的是 wrapper，而不是 foo1.

可以使用 functools.wraps 来进行装饰器修复，wraps 本身是一个装饰器，能够把原函数的元信息拷贝到装饰器里的 func 函数中。

```python
from functools import wraps # 导入模块

def use_logging(level):
    def decorator(func):
        @wraps(func) # 使用 wraps，记得要传入 func
        def wrapper(*args, **kwargs):
            pass
            return func(*args, **kwargs)
        return wrapper
    return decorator

@use_logging(level="warn")
def foo1(name, msg):
    print('i am %s, %s' %(name, msg))

foo1('foo1', 'hello!')
print(foo1.__name__) # foo1, 如果不加 @wraps(func)，打印出的是 wrapper
```

### 装饰器执行顺序
一个函数可以定义多个装饰器，执行顺序是从里到外，先调用最靠近函数的装饰器，下面的装饰器执行顺序为 c、b、a，也就是 foo = a(b(c(foo)))

```python
@a
@b
@c
def foo():
	pass
```

### 常用内置装饰器
1. @staticmethod
2. @classmethod
3. @property
可以把类中的一个方法当做属性使用，调用时就只要 a.func, 而不用 a.func()。
被修饰的特性方法，内部可以实现处理逻辑，但对外提供统一的调用方式，遵循了统一访问的原则。

在给一个类的属性比如分数 score 初始化时，为了限制它的范围，我们可以给它加上 set_score()、 get_score() 方法，然后在调用时通过 s.get_score(60) 来设置大小，get_score() 来获取值，但是这样调用又略显复杂，没有直接用属性那么简单。python 的 @property 装饰器就可以把方法变成一个属性。

```python
class Student(object):
    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value

s = Student()
s.score = 60 # 相当于是 s.get_score(60)
s.score = 'a' # ValueError: score must be an integer!
s.score = 101 # ValueError: score must be an integer!
```

@property 本身创建了一个装饰器 @score.setter，负责把一个 setter 方法变成属性赋值，实际上就相当于之前自己定义的 set_score(); 当然也有一个 @score.getter 装饰器用来获取属性值。

4. functools.wraps
用在装饰器的代码里。可以把原始函数的`__name__`等属性复制到 wrapper() 函数中，这样就可以获取到真实函数的`__name__`属性，而不是 wrapper。


### 闭包
闭包就是能够读取其他函数内部变量的函数，可以理解为定义在一个函数内部的函数，外部的叫外函数，内部的叫内函数。

在一个外函数中定义了一个内函数，内函数用到了外函数的临时变量，外函数的返回值是内函数的引用，这就构成了闭包。

```python
def outer(a):
	b = 10
	def inner(x):
		print(a * x + b) # a,b 是 outer 的临时变量
	return inner

demo = outer(1)
demo(2) # 输出 1*2+10 = 12
```

若要在内函数中修改闭包变量（外函数绑定给内函数的局部变量）：
1. 在 python3 中，用 nonlocal 关键字申明变量，表示这个变量不是局部变量空间的，需要向上一层变量空间中寻找该变量
2. 在 python2 中，没有 nonlocal 关键字，但可以把闭包变量改为可变类型数据进行修改，如 set、list、dict

```python
def outer(a):
	b = 10
	c = [a]
	def inner():
		nonlocal b
		b += 1 # 方法1
		c[0] += 1 # 方法2
		print(b, c[0])
	return inner

demo = outer(1)
demo() # 输出 11, 2
```

在使用闭包的过程中，一旦外函数被调用一次返回了内函数的引用，虽然每次调用内函数，是开启一个函数执行过后消亡，但是闭包变量实际上只有一份，每次开启内函数都在使用同一份闭包变量
```python
def outer(a):
	def inner(b):
		nonlocal a
		a += b
		return a
	return inner

demo = outer(1)
demo(2) # 输出 1+2=3
demo(3) # 输出 3+3=6
```

### 装饰器实战
#### 装饰器为访问页面添加登录验证功能
```python
# 定义用户字典
users = {'xieyiyu': '123456',
         'lily': '123'}

# 初始化当前用户名，无用户名，登录状态为 False
current_user = {'username': None, 'login': False}

# 构建装饰器
def auth_deco(func):
    def wrapper(*args, **kwargs):
        # 若已经登录，执行基本函数
        if current_user['username'] and current_user['login']:
            return func(*args, **kwargs)
        # 若未登录，提示用户输入用户名和密码
        username = input('请输入用户名：').strip()
        passwd = input('请输入密码：').strip()
        # 验证用户名和密码是否正确
        if username not in users.keys():
            print('用户名不存在')
        else:
            if passwd == users[username]: # 登录成功，将 current_user 设置为该用户
                current_user['username'] = username
                current_user['login'] = True
                print('登录成功')
            else:
                print('用户名或密码输入错误，请重新登录')
        return func(*args, **kwargs)
    return wrapper

@auth_deco
def index(): # 在进入页面前先校验用户是否登录了
    if current_user['username']:
        print('你好, %s' %current_user['username'])
    else:
        print('请先登录')

index()
```

#### 用装饰器实现数据库连接

### 参考
[理解 Python 装饰器看这一篇就够了](https://foofish.net/python-decorator.html)