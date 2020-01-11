---
title: Python 正则表达式
date: 2019-03-07 13:59:31
tags: [正则表达式,Python]
categories: Python
---

正则表达式的设计思想是用一种描述性语言来给字符串定义一个规则，凡是符合规则的字符串，就认为是匹配的。

<!--more-->

## 正则表达式原理
### 正则表达式引擎
正则表达式的引擎可以分为两类：
- DFA（Deterministic finite automaton）: 确定型有穷自动机
- NFA（Non-deterministic finite automaton）: 非确定型有穷自动机

DFA 对于字符串里的每一个字符只扫描一次，速度快，但特性较少，不支持惰性匹配和回溯。 目前使用 DFA 引擎的语言和工具有：awk、egrep 和 lex。

NFA 需要反复回溯，速度慢，但特性丰富，因此应用广泛，是主要的正则表达式引擎。Perl、Ruby、Python 的 re 模块，java 和 .NET 的 regex 库都是基于 NFA。

#### NFA
传统的 NFA 接收找到的第一个匹配，所以可能会导致其他可能更长的匹配未被发现。

POSIX NFA 与传统的 NFA 引擎类似，不同的是 POSIX NFA 可以确保在已经找到可能的最长的匹配之前，继续回溯。POSIX NFA 引擎速度慢于传统 NFA 引擎。

### 字符串组成
字符串由字符和位置组成。 如 `abc` 包括三个字符和四个位置。

<div align="center"><img src="https://res.cloudinary.com/dty6stpv6/image/upload/v1552202626/rechar.png" width="30%"></div>

### 占有字符和零宽度
**占有字符**： 正则表达式匹配过程中，如果子表达式匹配到的是字符内容，而非位置，并被保存到最终的匹配结果中，那么就认为这个子表达式是占有字符的。

**零宽度**： 如果子表达式匹配的仅仅是位置，或者匹配的内容并不保存到最终的匹配结果中，那么就认为这个子表达式是零宽度的。

占有字符是互斥的，零宽度是非互斥的。也就是一个字符，同一时间只能由一个子表达式匹配，而一个位置，却可以同时由多个零宽度的子表达式匹配。
 
### 控制权
正则表达式当开始匹配的时候，一个正则匹配单元获得控制权，从字符串中的某一个位置开始尝试匹配。

一个正则匹配单元开始尝试匹配的位置，是从前一个正则匹配单元匹配成功的结束位置开始。

### 匹配过程
正则表达式的匹配过程，NFA 采用回溯方法，详见 [正则表达式回溯法原理](https://zhuanlan.zhihu.com/p/27417442)

## Python 中的正则表达式
在 Python 中使用正则表达式需要引入 re 模块。

```python
import re
```

由于 Python 的字符串本身也用 `\` 转义，使用 Python 的 `r` 前缀，可以忽略编程语言中对 `\` 的转义，只关心正则表达式中的 `\`。 用法在正则表达式前面加 `r`， 如 `r'\d'`。

### compile
compile 用于编译正则表达式，生成一个 pattern 对象。
```python
re.compile(pattern[, flag])
```

flag 表示匹配模式，比如忽略大小写，多行模式等，可以通过 `|` 选择多种匹配模式。
- re.l 忽略大小写
- re.M 多行模式

生成 pattern 后，就可以利用 pattern 的一系列方法对文本进行匹配查找。

### match
match 从字符串的起始位置（也就是 index=0）开始匹配，若起始位置没有匹配成功就返回 None。 match 是一次匹配，只要找到一个匹配结果就返回，匹配成功则返回一个 Match 对象。
```python
re.match(pattern, string[, flag=0])
pattern.match(string[, pos[, endpos]])
```

可选参数：
- flag 为标志位，表示匹配模式 
- pos 和 endpos 指定字符串的起始位置和终止位置

```python
>>> import re
>>> print(re.match(r'\d+', 'one12twothree34four'))
None

>>> pattern = re.compile(r'\d+')
>>> print(pattern.match('one12twothree34four', 3))
<_sre.SRE_Match object; span=(3, 5), match='12'>
```

对于 Match 对象，有几个常用方法：
- group(nums=0) 获取一个或多个分组匹配的字符串，默认是 0
- groups() 返回一个包含所有小组字符串的元组，从 1 开始
- start() 获取分组匹配的子串的起始位置
- end() 获取分组匹配的子串的终止位置
- span() 获取分组匹配的子串的起始位置和终止位置，返回一个元组

### search
search 查找字符串的任意位置，扫描整个字符串并返回第一个成功的匹配，没有符合的返回 None。 匹配成功，也是返回一个 Match 对象。

```python
re.search(pattern, string[, flag=0])
pattern.search(string[, pos[, endpos]])
```

注意 search 和 match 的区别，search 是查找字符串任意位置， match 必须要起始位置匹配。

### sub
替换字符串中的匹配项 
```python
re.sub(pattern, repl, string, count=0) 
```
repl 为替换的字符串，也可以是一个函数; count 为模式匹配替换的最大次数，默认 0 是替换所有。


### findall
re.match 和 re.search 都是值匹配一次，而 findall 匹配所有，返回列表形式
```python
import re
pattern = re.compile(r'\d+') # 查找数字
res = pattern.findall(string)
```

### finditer
与 findall 类似，但将匹配成功的结果作为一个迭代器返回

### split
按照能够匹配的子串将字符串分割后返回列表，maxsplit 指定最大分割次数。
```python
re.split(pattern, string[, maxsplit=0[, flags=0]])
```

## 正则表达式的元数据

<div align="center"><img src="https://res.cloudinary.com/dty6stpv6/image/upload/v1552040609/re.jpg"></div>


```\<``` 表示词首，如 ```\<abc``` 表示以 abc 为首的词
```\>``` 表示词尾，如 ```\>abc``` 表示以 abc 结尾的词

---
## 参考
[[1] Python 正则表达式| 菜鸟教程](http://www.runoob.com/python/python-reg-expressions.html)

[[2] 正则表达式回溯法原理](https://zhuanlan.zhihu.com/p/27417442)

[[3] 正则基础之——NFA引擎匹配原理](https://blog.csdn.net/lxcnn/article/details/4304651)