---
title: 自动化测试之Selenium
date: 2019-09-14 16:42:33
tags: [自动化测试,Selenium]
categories: 测试
---

Selenium 是用于 web 应用的自动化测试工具，可以模拟浏览器操作，自动完成 web 基本任务管理。

<!--more-->
## 自动化测试
适合自动化测试的情况：
1. 项目周期长且相对稳定
2. 需要做频繁的冒烟测试
3. 需要经常做回归测试
4. 需要进行大数据量的数据驱动测试

不适合自动化测试的情况：
1. 项目周期短，且用例不会多次重复执行
2. 被测项目不稳定，变化频繁

需要关注的指标：
1. 自动化测试用例的覆盖率
2. 节省的时间成本 = 手工测试所花时间 - 自动化测试所花时间
3. 自动化测试的投入
4. 自动化测试发现的缺陷数
5. 关键指标是： 自动化测试的投入产出比 ROI = (手工测试的成本 - 自动化测试的成本) / 自动化测试的成本，ROI 为正值且越大说明回报越好

## Selenium 原理
selenium 的原理涉及到三个部分：
1. WebDriver API，即 client，也就是写的代码
2. 浏览器驱动 driver，也就是下载的 chromedriver.exe
3. 浏览器

### Selenium 执行流程
client 并不知道浏览器是如何工作的，可以通过 driver，实现 client 与浏览器之间的通信，client 根据 webdriver 协议发送请求给 driver，driver 解析请求，并在浏览器上执行相应的操作，并把执行结果返回给 client。具体如下：
1. 对于每一条 Selenium 脚本，一个 HTTP 请求会被创建并且发送给浏览器的驱动
2. 浏览器驱动中包含了一个 HTTP Server，用来接收这些 HTTP 请求
3. HTTP Server 接收到请求后根据请求来具体操控对应的浏览器
4. 浏览器执行具体的测试步骤
5. 浏览器将步骤执行结果返回给 HTTP Server
6. HTTP Server 又将结果返回给 Selenium 的脚本，如果是错误的 HTTP 代码，控制台会显示相应的报错信息。

### webdriver 协议
webdriver 基于的协议是 JSON Wire protocol，数据传输使用的是 json，JSON Wire protocol 是在 http 协议基础上，对 http 请求及响应的 body 部分的数据进一步规范。

在 client 和 server 之间，只要是基于 JSON Wire Protocol 来传递数据，无论 client 使用 java、python 还是其他语言实现，driver 都能处理相应的脚本。

## Selenium 的简单使用
```python
from selenium import webdriver  

# 启动浏览器
driver = webdriver.Chrome()
# 打开一个网页
driver.get("http://www.baidu.com")
# 百度搜索 selenium
driver.find_element_by_id("kw").send_keys("selenium")
# 单击搜索按钮
driver.find_element_by_id("su").click()
# 后退
driver.back()
# 前进
driver.forward()
# 截图
driver.save_screenshot("baidu.png")
# 退出浏览器
driver.close()
#关闭浏览器
driver.quit()
```

- driver.quit(): 退出并关闭窗口的每一个相关的驱动程序
- driver.close(): 关闭当前窗口

## Python webdriver API
### 元素定位
webdriver 提供了一系列的对象定位方法，常用的有：
- find_element_by_id()，唯一
- find_element_by_name()，唯一
- find_element_by_class_name()
- find_element_by_linx_text()， 操作对象是文字超链接
- find_element_by_partial_link_text()，操作对象是文字超链接
- find_element_by_tag_name()， 标签名
- find_element_by_xpath()
- find_element_by_css_selector()

定位一组对象，用 find_elements，返回的是一个 list，可以用于批量操作对象，如选择复选框
1. find_elements_by_tag_name()
2. find_element_by_css_selector()

判断元素是否存在： 需要用元素定位和异常捕获的方法判断
```python
try:
    driver.find_element_by_id("none")
except NoSuchElementException:
    print("element does not exist")
```

注意： 在 selenium 中不能定位不可见元素，比如 hidden 和 display=none 的元素

### 操作测试对象
- clear() ：清除内容
- click() ：鼠标点击
- send_keys() ：向输入框输入
- submit() ：提交表单

### 显式等待和隐式等待
显示等待 WebDriverWait()：是针对某个特定元素设置的等待时间，在设置时间内，默认每隔一段时间检测一次当前页面某个元素是否存在，如果在规定的时间内找到了元素，则直接执行，即找到元素就执行相关操作，如果超过设置时间检测不到则抛出异常。

隐式等待 implicitly_wait()：是设置了全局等待，设置等待时间，是对页面中的所有元素设置加载时间，如果超出了设置时间的则抛出异常。隐式等待可以理解成在规定的时间范围内，浏览器在不停的刷新页面，直到找到相关元素或者时间结束。

强制等待 sleep()：也叫线程等待，设置固定休眠时间，执行 sleep() 后线程休眠，上面两种等待不会休眠。

## Appium
appium 是一个自动化测试开源工具，支持 iOS 平台和 Android 平台上的原生应用，web应用和混合应用。
1. 移动原生应用 native：用 iOS 或 Android SDK 写的应用
2. 移动 web 应用 H5：使用移动浏览器访问的应用
3. 混合应用：原生代码封装网页视图，原生代码和 web 内容交互

appium 类库封装了标准 Selenium 客户端类库，实现了 Mobile JSON Wire Protocol

流程： 使用 python（client）编写一个 appium 自动化脚本并执行，首先会请求 appium-Server（mac 和 win 下不同），appium-Server 通过解析，驱动 Android 虚拟机或真机来执行 appium 脚本。

### 元素定位
可以使用 uiautomatorviewer 工具来查看控件的属性，定位方式与 selenium 相同，有 id、name、class_name、xpath 等方法。

### native 与 h5 切换
在混合开发的 app 中，原生应用可以通过 uiautomator 获取控件信息，而 h5 也就是 web 网页是 B/S 架构，两者的运行环境不同，因此需要进行上下文(context) 切换，再对 h5 页面元素进行定位操作。

切换到 h5，进行 h5 元素定位：
1. 导航到应用程序中 web 页面
2. 获取上下文列表 driver.contexts，它返回一个我们可以访问的上下文列表，例如：NATIVE_APP 或 WEBVIEW_xxx
3. 通过 driver.switch_to.context 切换到你要操作的 webview，原来是在 NATIVE_APP，切换到 WEBVIEW_xxx
4. 操作完后，返回到原来的 app view

## 断言
断言是一些布尔表达式，当需要在一个值为 False 时中断当前操作的话，可以使用断言。

断言可以用于单元测试，判断到某个节点位置必须满足某些逻辑条件时才能继续运行下去，不满足程序就会崩溃或报错，这个时候就可以使用断言。

### 断言与异常
- 断言用于开发测试期间，是给程序员自己使用的；断言触发后程序崩溃退出，不需要从错误中恢复
- 异常在程序运行期间触发；异常通常会用 try/except/finally 等结构捕获异常，之后还能从错误中恢复继续运行

### python 常用断言
- assertEqual(a, b [, msg='xxx'])： 断言 a 和 b 相等，msg 为测试失败时打印的信息； assertNotEqual(a, b)
- assertIs(a, b [, msg='xxx']): 断言 a 是 b； assertNotIs(a, b)
- assertIn(a, b [, msg='xxx'])：断言 a 在 b 中； assertNotIn(a, b)
- assertIsInstance(a, b [, msg='xxx'])： 断言 a 是 b 的一个实例； assertNotIsInstance(a, b)
- assertTrue(x [, msg='xxx'])： 断言 x 为 True； assertFalse(x)
- assertIsNone(x [, msg='xxx'])： 断言 x 是 None； assertIsNotNone(x)

## 单元测试 unittest
[Python必会的单元测试框架 —— unittest](#https://huilansame.github.io/huilansame.github.io/archivers/python-unittest)