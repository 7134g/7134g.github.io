---
layout: post
title: python分享一个时间模块dateutil，可转化大多数时间格式
date: 2019-04-05 20:13:55.000000000 +09:00
---

先上代码，看效果

```bash

from dateutil.parser import parse

a = parse('2019-6-14')
b = parse('20190614')
c = parse('Mon Mar 18 2019 15:01:20 GMT+0800')

print(type(a))
print(a)

print(type(b))
print(b)

print(type(c))
print(c)[/code]
```
输出结果

```bash
<class 'datetime.datetime'>
2019-06-14 00:00:00
<class 'datetime.datetime'>
2019-06-14 00:00:00
<class 'datetime.datetime'>
2019-03-18 15:01:20-08:00[/code]
```

对于网络上抓取的苦于使用time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(time.time()))这样的格式化方法整来整去，话费很多时间找到解决方法结果还很麻烦。
使用dateutil可以接收主流所使用的时间格式的字符，直接转化为datetime对象省去的很多麻烦
毕竟直接操作datetime对象轻松多了

