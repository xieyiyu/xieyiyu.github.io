---
title: Python decimal 模块
date: 2019-08-10 17:21:30
tags: [Python]
categories: Python
---

python 中的 decimal 模块提供十进制浮点运算支持。

<!--more-->

我们先来看下直接对两个浮点数进行运算得到的结果：
```python
a = 1.1
b = 2.2
print(a + b) # 输出 3.3000000000000003
```

输出不是期望的 3.3，这是由于原生的二进制浮点数本身存在误差，直接计算的话会得到不精确的结果。 为了消除这种误差，可以用 decimal 模块进行更加精确的浮点计算。
```python
from decimal import Decimal

a = Decimal('1.1')
b = Decimal('2.2')
print(a + b) # 输出 3.3
```

### 设定有效数字
通过 getcontext().prec 可以设置有效数字
```python
from decimal import Decimal
from decimal import getcontext
getcontext().prec = 4
print(Decimal('2.2')/Decimal('1.3')) # 1.692
```

### 设定小数位数
Deicmal() 函数可以四舍五入设置保留的小数位，且不用管小数原来的形式
```python
from decimal import Decimal

Decimal(num).quantize(Decimal('0.000000')) # 四舍五入，保留六位小数，不管 num 是什么形式
```

而使用格式化的方式，如 float('%.6f' % a) 可以控制保留六位小数，但是如果后面都是 0 的话，比如 0.00000，返回的是 0.0； 如果是 1.2200000 的话，返回的是 1.22

【参考】
[Python中的decimal模块执行精确的浮点运算](https://finthon.com/python-decimal/)