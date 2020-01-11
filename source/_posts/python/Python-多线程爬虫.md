---
title: Python 多线程爬虫
date: 2019-08-31 14:33:17
tags: [Python, 多线程, 爬虫]
categories: Python
---

爬虫主要运行时间消耗是请求网页时的 IO 阻塞，使用多线程能够让不同请求的等待同时进行，从而提高爬虫运行效率。

<!--more-->

基于多线程（这里开启了10个线程），使用 github 的 api，抓取 fork cpython 项目的信息，并将数据存储到 json 文件中。

```python
import requests
import time
from threading import Thread
from queue import Queue
import json

# 计时用的装饰器
def run_time(func):
    def wrapper(*args, **kw):
        start = time.time()
        func(*args, **kw)
        end = time.time()
        print('running', end-start, 's')
    return wrapper

class Spider():

    def __init__(self):
        self.qurl = Queue()
        self.data = list()
        self.email = 'xxx' # 登录github用的邮箱
        self.password = 'xxx' # 登录github用的密码
        self.page_num = 120
        self.thread_num = 10

    def produce_url(self):
        baseurl = 'https://api.github.com/repos/python/cpython/forks?page={}'
        for i in range(1, self.page_num + 1):
            url = baseurl.format(i)
            self.qurl.put(url) # 生成URL存入队列，等待其他线程提取

    def get_info(self):
        while not self.qurl.empty(): # 保证url遍历结束后能退出线程
            url = self.qurl.get() # 从队列中获取URL
            print('crawling', url)
            req = requests.get(url, auth = (self.email, self.password))
            data = req.json()
            for datai in data:
                result = {
                    'project_name': datai['full_name'],
                    'project_url': datai['html_url'],
                    'project_api_url': datai['url'],
                    'star_count': datai['stargazers_count']
                }
                self.data.append(result)

    @run_time
    def run(self):
        self.produce_url()

        ths = []
        for _ in range(self.thread_num):
            th = Thread(target=self.get_info)
            th.start()
            ths.append(th)
        for th in ths:
            th.join()

        s = json.dumps(self.data, ensure_ascii=False, indent=4)
        with open('github_thread.json', 'w', encoding='utf-8') as f:
            f.write(s)

        print('Data crawling is finished.')

if __name__ == '__main__':
    Spider().run()
```

参考：
https://juejin.im/post/5b0951ab51882538ac1ce3c8