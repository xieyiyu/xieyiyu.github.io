---
title: Python 数据结构底层实现
date: 2019-09-25 21:54:32
tags: [Python, 数据结构, list, dict, set, tuple]
categories: Python
---

list、dict、set、tuple 是 python 中常用的四种数据结构，在平常的学习中只是简单学习了其使用方式，没有了解底层实现，特此记录和学习一下更深层次的知识。

<!--more-->

## 列表和元组

## 字典
python 的内建数据类型 dict 字典，就是用哈希表实现的。

Python 是使用开放寻址法中的二次探查来解决冲突的。然后如果使用的容量超过数组大小的 2/3，就申请更大的容量。数组大小较小的时候 resize 为 * 4，较大的时候 resize * 2。实际上是用左移的形式。

## 集合
集合 set 中的元素互不相同，没有重复元素，set 中的元素是无序的，比如 {1, 2, 3} 和 {3, 2, 1} 是同一个集合，创建一个空集合可以用 `s = set()`，将一个 list 转为 set 元素的顺序会打乱。

set 和 dict 的底层实现方式类似，都是使用哈希，把 set 的实现方式叫做 Hash Set，dict 的实现叫 Hash Table。
set 与 dict 的不同是，set 只存储 key，对于 set 查找元素的时间复杂度为 O(1)

set 的去重是通过两个函数 `__hash__` 和 `__eq__` 结合实现的
1. 当两个变量的哈希值不同时，就认为这两个变量是不同的
2. 当两个变量哈希值相同时，调用 `__eq__` 方法，返回 True 则这两个变量是同一个，应该去除一个；返回 False 则不去重，因为哈希值相同的变量的值可能不同。

set 中的元素和 dict 的 key 必须是可以 hash 的，不可变类型类型都是可 hash 的，比如 number(int, float, complex)、布尔型、字符串(str, bytes)、tuple、None；而可变类型 list、set、dict 都是不可哈希的。

set 的 hash 寻址是二次散列和顺序寻址结合的,

##### set 常用方法
1. s.add(elem)： 添加一个元素到 set 中，如果元素存在则什么都不做
2. s.remove(elem)： 删除 set 中的一个元素，元素不存则报错 KeyError
3. s.pop()： 删除并返回**任意**元素，空集报错 KeyError
4. s.clear()： 删除所有元素
set 不能修改元素，只能添加和删除元素，时间复杂度平均为 O(1), 最差的情况是 O(n)