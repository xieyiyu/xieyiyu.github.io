---
title: Python 时间函数总结
date: 2019-12-15 10:55:51
tags: [Python]
categories: Python
---

在实际开发中，常常需要用到日期和时间，并将其进行格式化处理，在 Python 中的时间模块常用的是 time 和 datetime，这两个模块中的方法容易混淆，特此对 Python 中的日期时间获取与格式化进行梳理总结。

<!--more-->

### 时间
1. UTC time Coordinated Universal Time：世界协调时间，又称格林尼治天文时间、世界标准时间。与 UTC time 对应的是各个时区的 local time，也就是本地时间，如北京时间。

2. epoch time：表示时间开始的起点，是一个特定的时间，不同平台上这个时间点的值不太相同，对 Unix，epoch time 是 1970-01-01 00:00:00 UTC。

3. timestamp：时间戳，也称为 Unix 时间或 POSIX 时间，是一种时间表示方式，表示从格林尼治时间 1970 年 1 月 1 日 0 时 0 分 0 秒开始到现在所经过的毫秒数，其值为 float 类型。**但需要注意的是这在某些编程语言中是秒数，比如 python。**

对应的，在 python 中日期时间也有三种表示方式。
1. timestamp： 时间戳

2. struct_time：时间元组，共有九个元素组。stamptime 时间戳和格式化时间字符串之间的转化必须通过 struct_time 才行，所以 struct_time 是 3 种时间表示的中心。

3. format time：格式化时间，已格式化的结构字符串使时间更具可读性。包括自定义格式和固定格式。

在使用过程中，常需要在三种时间格式之间进行相应的转换，以得到我们想要的表现格式。

### time 模块
在 time 模块中，获取当前时间只有一种方式，使用`time.time()`，但得到的是时间戳格式的时间，想要获取时间元组和格式化时间，只能进行转换。

#### timestamp 与 struct_time
localtime 和 gmtime 方法都可以将时间戳转化为时间元组，如果不传入参数默认为当前时间。localtime 转的是本地时间， gmtime 是世界标准时间。

```python
import time
# struct_time to timestamp
time.mktime(time.localtime())

# timestamp to struct_time
t = time.time() # 1576397965.253168
time.localtime(t)
# time.struct_time(tm_year=2019, tm_mon=12, tm_mday=15, tm_hour=16, tm_min=19, tm_sec=25, tm_wday=6, tm_yday=349, tm_isdst=0)

time.gmtime(t)
# time.struct_time(tm_year=2019, tm_mon=12, tm_mday=15, tm_hour=8, tm_min=19, tm_sec=25, tm_wday=6, tm_yday=349, tm_isdst=0)
```

##### struct_time
属性 | 含义 | 值
-|-|-|
tm_wday | weekday | 0 - 6（0 表示周日）
tm_yday | 一年中的第几天 | 1 - 366
tm_isdst | 是否是夏令时 | 默认为 -1

struct_time 属性值获取可以通过两种方式获取：
- 利用下标，如 st[0] 即 tm_year 的值
- 通过对象名 `st.tm_year` 获取

#### struct_time 与 format_time
```python
# struct_time to format_time
time.strftime("%Y-%m-%d") # '2019-12-18'
time.strftime("%Y-%m-%d", time.localtime())

# format_time to struct_time
time.strptime('2019-12-18', '%Y-%m-%d')
# time.struct_time(tm_year=2019, tm_mon=12, tm_mday=18, tm_hour=0, tm_min=0, tm_sec=0, tm_wday=2, tm_yday=352, tm_isdst=-1)
```

##### format_time
格式 | 含义
-|-|
%a | 本地（locale）简化星期名称
%A | 本地完整星期名称
%b | 本地简化月份名称
%B | 本地完整月份名称
%c | 本地相应的日期和时间表示
%d | 一个月中的第几天（01 - 31）
%H | 一天中的第几个小时（24小时制，00 - 23）
%I | 第几个小时（12小时制，01 - 12）
%j | 一年中的第几天（001 - 366）
%m | 月份（01 - 12）
%M | 分钟数（00 - 59）
%p | 本地 am 或者 pm 的相应符
%S | 秒（00 - 61, 60 是闰秒，61 是基于历史原因保留）
%U | 一年中的星期数。（00 - 53星期天是一个星期的开始。）第一个星期天之前的所有天数都放在第0周。
%w | 一个星期中的第几天（0 - 6，0是星期天）
%W | 和 %U 基本相同，不同的是 %W 以星期一为一个星期的开始。
%x | 本地相应日期
%X | 本地相应时间
%y | 去掉世纪的年份（00 - 99）
%Y | 完整的年份
%Z | 时区的名字（如果不存在为空字符）
%% | ‘%’字符

最常用： `%Y-%m-%d %H:%M:%S`

### datetime 模块
datetime 模块是对 time 模块的进一步封装，对用户更加友好，在时间的获取上也比较方便。 datetime 模块中常用的类有 date、time、datetime 和 timedelta。

#### date 类
date 对象由 year、 month、 day 三部分组成： datetime.date(year, month, day)

```python
import datetime
a = datetime.date(2019, 12, 18)
a.year # 2019
a.month # 12
a.day # 18
```

#### time 类
datetime.time(hour[ , minute[ , second[ , microsecond[ , tzinfo] ] ] ] ) 

静态方法和字段：
- time.min、time.max：time类所能表示的最小、最大时间。其中，time.min = time(0, 0, 0, 0)， time.max = time(23, 59, 59, 999999)；
- time.resolution：时间的最小单位，这里是 1 微秒；

#### datetime 类
datetime 相当于 date 和 time 结合起来
`datetime.datetime (year, month, day[ , hour[ , minute[ , second[ , microsecond[ , tzinfo] ] ] ] ] )`

```python
import datetime
dt = datetime.datetime.now() # datetime.datetime(2019, 12, 18, 20, 56, 36, 415244)
dt.date() # datetime.date(2019, 12, 18)
dt.time() # datetime.time(20, 56, 36, 415244)
dt.strftime('%Y-%m-%d %H:%M:%S') # '2019-12-18 20:56:36'
dt.ctime() # 'Wed Dec 18 20:56:36 2019'，返回一个日期时间的 C 格式字符串
dt.utctimetuple() # UTC 时间元组：time.struct_time(tm_year=2019, tm_mon=12, tm_mday=18, tm_hour=20, tm_min=56, tm_sec=36, tm_wday=2, tm_yday=352, tm_isdst=0)
```

#### timedelta 类
timedelta 类可以实现日期之间的加减运算，包括 date、time、datetime 对象。注意不能计算月份。

1. 获取之前或之后的时间: 天(days), 小时(hours), 分钟(minutes), 秒(seconds), 微秒(microseconds)。

```python
dt - datetime.timedelta(days=1) # 昨天，datetime.datetime(2019, 12, 17, 20, 56, 36, 415244)
dt + datetime.timedelta(hours=8) # 当前时间向后 8 小时，datetime.datetime(2019, 12, 19, 4, 56, 36, 415244)
```

2. 获取时间差，时间差单位默认为秒，可以查看天(days), 秒(seconds), 微秒(microseconds)。

```python
start_time = datetime.datetime.now()
end_time = start_time + datetime.timedelta(hours=8)

end_time - start_time # datetime.timedelta(seconds=28800)
(end_time - start_time).seconds # 28800
(end_time - start_time).days # 0
```


### 参考文档
[Python datetime模块详解](https://www.cnblogs.com/awakenedy/articles/9182036.html)
[python time模块和datetime模块详解](https://www.cnblogs.com/haitaoli/p/10823403.html)