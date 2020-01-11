---
title: Python 编码总结
date: 2019-02-25 18:03:51
tags: [Python,编码]
categories: Python
---

计算机中存储的信息都是用二进制表示的，现实生活中用的英文、中文等字符需要经过`编码`转换为二进制才能存储在计算机中，这种转换规则便是`字符编码`。

<!--more-->

## ASCII 码
ASCII 字符集只有一种编码方式，采用单字节编码，一个字节是 8 个 bit，因此一个字节可以表示 256 种不同的状态，从 00000000 到 11111111。 但 ASCII 规定第一位为 0，因此 ASCII 共规定 128 个字符的编码。

然而，要处理中文一个字节是不够的，至少需要两个字节，且不能和 ASCII 码冲突，因此中国制定了 GB2312 来进行中文编码。但由于每个国家有自己的编码标准，在多语言混合的文本中由于冲突问题，会导致乱码显示。

## Unicode
Unicode 编码把所有语言统一到一套编码中，Unicode 主要有三种编码方式，最常用的 UTF-8 将一个 Unicode 字符根据不同的数字大小编码成 1-6 个字节，常用的英文字母用 1 个字节，汉字通常是 3 个字节，只有少数生僻字会用到 4-6 个字节编码。 注意： UTF-8 是 Unicode 的实现方式之一。

目前计算机系统通用的字符编码方式可以总结如下：
1. 在计算机内存中，统一使用 Unicode 编码，当需要保存到硬盘或需要传输时，就转化为 UTF-8 编码
2. 用记事本编辑的时候，从文件读取的 UTF-8 字符被转换为 Unicode 字符到内存中，编辑完成后，保存的时候再将 Unicode 转化为 UTF-8 保存到文本中。

## Python 字符编码
Python 的诞生时间比 Unicode 早得多，因此 Python 的默认编码是 ASCII。 在 Python2 文件中若不显式地指定字符编码，会出现语法错误，因此为在源代码中支持非 ASCII 字符，需要在第一行指出编码格式：

```python
# coding=utf-8
```

或者是：
```python
#!/usr/bin/python
# -*- coding: utf-8 -*-
```

> 注意： 以下代码都基于 python2。

Python 有两种不同的字符串数据类型： str 和 unicode，都是 basestring 的子类。

对于同一个汉字 "好"， 用 str 表示和 unicode 表示是不同的。 用 str 表示时，对应的 UTF-8 编码是 "\xe5\xa5\xbd", 而 Unicode 表示对应的符号是 u'\u597d'，等同于 u'好'。

```python
>>> a = '好'
>>> type(a)
<type 'str'>
>>> a
'\xe5\xa5\xbd'

>>> b = u'好'
>>> type(b)
<type 'unicode'>
>>> b
u'\u597d'
```

str 类型的字符编码具体是 UTF-8 还是其他编码方式，取决于操作系统。 str 用**字节串**表述更加准确，对 str 类型进行迭代，会按照其在内存中的字节序一次迭代。 

以下是 Python2 中的显示：
```python
>>> a = '好a'
>>> for c in a:
...     print c
... 
�
�
�
a
```

```python
>>> s = u"好a"
>>> for c in s:
...     print c
...
好
a
```

而 Python3 的默认编码格式是 Unicode 。


## str 与 Unicode 转换
str 可以通过 decode 解码成 unicode 字符串。

unicode 可以通过 encode 编码得到 UTF-8 编码格式的 str 类型的字符串。

> str --> decode --> unicode  
unicode --> encode --> str

如果对 unicode 错误的调用了 decode 方法，其实是会先调用 encode(default_encoding) 进行隐式转换为 str，然后再 decode 为 unicode。 而在 python2 中 default_encoding 是 ASCII，如果 unicode 字符串本身超过了 ASCII 的编码范围就会报错。 对 str 调用 encode 也是如此。 这是初学者可能经常会犯的错误。

```python
>>> u = u"好"
>>> u.decode("utf-8")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python2.7/encodings/utf_8.py", line 16, in decode
    return codecs.utf_8_decode(input, errors, True)
UnicodeEncodeError: 'ascii' codec can not encode character u'\u597d' in position 0: ordinal not in range(128)
```

解决方法： 可以通过对 unicode 进行 encode 再 decode 得到 unicode 编码格式
```python
>>> u = u"好"
>>> u.encode("utf-8").decode("utf-8")
u'\u597d'
```

## str() 和 unicode()
str(s) 和 unicode(s) 是两个工厂方法，分别返回 str 字符串对象和 unicode 字符串对象。 str(s) 相当于是 s.encode('ASCII') 简写， unicode(s) 相当于 s.decode('ASCII') 的简写。

```python
>>> s = u"好"
>>> str(s)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can not encode character u'\u597d' in position 0: ordinal not in range(128)

```
s 是个 unicode 字符串，str(s) 相当于 s.encode('ASCII')，而汉字 "好" 没有 ASCII 码，因此报错。解决此问题需要制定编码格式，用 s.encode('gbk') 或 s.encode('utf-8')。

## 乱码
所有出现乱码的原因都可以归结为： 字符经过不同编码解码，在解码过程中使用的编码格式不一致。

防止乱码出现的最好方式是始终用同一种编码格式对字符进行编码和解码操作。

## 其他
对于形如 unicode 形式的字符串，实际上是 str 类型，将其转化成真正的 unicode 可以使用 s.decode('unicode-escape')

```python
>>> s = 'id\u003d215903184\u0026index\u003d0\u0026st\u003d52\u0026sid\u003d95000\u0026i'
>>> s.decode('unicode-escape')
u'id=215903184&index=0&st=52&sid=95000&i'
```

[python2与python3字符串的区别](https://my.oschina.net/sallency/blog/1563298)