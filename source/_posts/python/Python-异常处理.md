---
title: Python 异常处理
date: 2019-04-01 12:08:18
tags: [Python]
categories: Python
---

python 使用 try/except/finally 语句块来处理异常。 良好的异常处理可以让程序更加健壮，清晰的错误信息能够帮助快速修复问题。

<!--more-->

## 异常处理
### try/except/finally
try/except 语句用来检测 try 语句块中的错误，从而让 except 语句能够捕获异常信息并处理，except 可以有多个。 except 后面若不指定异常类型，则默认捕获所有异常，可以通过 logging 或 sys 模块获取当前异常。

try/finally 语句无论是否发生异常都将执行最后的代码，可以只使用 try/finally, 省略 except

```python
def div(a, b):
    try:
        print(a / b)
    except ZeroDivisionError:
        print("Error: b should not be 0 !!")
    except Exception as e:
        print("Unexpected Error: {}".format(e))
    else:
        print('Run into else only when everything goes well')
    finally:
        print('Always run into finally block.')

div(2, 0) # Error: b should not be 0 !!
div(2, 'bad type') # Unexpected Error: unsupported operand type(s) for /: 'int' and 'str'
div(1, 2) # 0.5, 并打印 else 里的， finally 语句里的这三个最后都会执行
```

### except 带多种异常类型
使用同一个 except 语句可以处理多个异常信息，只要发生多个异常中的一个，就执行代码。
语法： except(Exception1[, Exception2[,...ExceptionN]]])

```python
def div(a, b):
    try:
        print(a / b)
    except (ZeroDivisionError, TypeError) as e:
        print(e)
```

### raise
raise 关键字用于主动抛出一个异常，raise关键字后面可以指定你要抛出的异常实例，一般来说抛出的异常越详细越好
```python
raise NameError("bad name!")
```

### 使用内置语法范式代替 try/except
1. for 语句处理了 StopIteration 异常，可以流畅地写出一个循环。

2. with 语句在打开文件后会自动调用 finally 中的关闭文件操作。 因此用 with open 来代替 try/except/finally

3. 访问一个不确定的属性时，可以用 getattr() 方法，而不用再捕获异常 except AttributeError


## 异常类
python 的异常是一个类，所有的异常类型都继承自 BaseException 这个基类。 python 捕获所有异常时，应该用 Exception，Exception 继承自 BaseException。 可以自定义一个异常，继承自 Exception 类。

BaseException 除了包含所有的 Exception 外还包含了 SystemExit，KeyboardInterrupt 和 GeneratorExit 三个异常，但这三个属于更高级别的异常，合理的做法是交给 Python 解释器处理。

python3 中使用 except Exception as e 来获取异常。

### 常见异常
- BaseException 所有异常的基类
- OverflowError 数值运算超过最大限制
- IOError 输入/输出操作失败
- ImportError 导入模块/对象失败
- RuntimeError 一般运行时错误
- SyntaxError 语法错误

## 参考
[总结：Python中的异常处理](https://segmentfault.com/a/1190000007736783)