---
layout: post
title: 多线程的坑(一)
date: 2018-01-15 19:24:11.000000000 +09:00
---


假如使用多线程却不加锁，最后得到的全局变量不是你所想要的，比如下面

```bash
#不加锁:并发执行,速度快,数据不安全
from threading import current_thread,Thread,Lock
import os,time
def task():
    global n
    print('%s is running' %current_thread().getName())
    temp=n
    time.sleep(0.5)
     
    n=temp-1


if __name__ == '__main__':
    # 共有的全局变量
    n=100
    
    threads=[]
    start_time=time.time()
    
    for i in range(100):
        t=Thread(target=task)
        threads.append(t)
        t.start()
        
    for t in threads:
        t.join()

    stop_time=time.time()
    
    # 想要的执行结果n应该为0
    print('主:%s n:%s' %(stop_time-start_time,n))[/code]
```

但是执行的结果是：       主:0.5114283561706543 n:99


所以，只能将操作共同拥有的部分加锁，数据就对了，不过所耗费的时间是不添加锁的很多倍。。。


```bash
# -*- coding: utf-8 -*-
from threading import current_thread,Thread,Lock
import os,time
def task():
    #未加锁的代码并发运行
    time.sleep(3)
    print('%s start to run' %current_thread().getName())
    global n
    #加锁的代码串行运行
    lock.acquire()
    temp=n
    time.sleep(0.5)
    n=temp-1
    lock.release()

if __name__ == '__main__':
    n=100
    lock=Lock()
    threads=[]
    start_time=time.time()
    for i in range(100):
        t=Thread(target=task)
        threads.append(t)
        t.start()
    for t in threads:
        t.join()
    stop_time=time.time()
    print('主:%s n:%s' %(stop_time-start_time,n))[/code]

```

但是执行的结果是：      主:53.294203758239746 n:0

可能有些就说，只需要start()后，join()一样保护数据安全。但是消耗的时间是300多秒，如下列代码

```bash
from threading import current_thread,Thread,Lock
import os,time
def task():
    time.sleep(3)
    print('%s start to run' %current_thread().getName())
    global n
    temp=n
    time.sleep(0.5)
    n=temp-1


if __name__ == '__main__':
    n=100
    start_time=time.time()
    for i in range(100):
        t=Thread(target=task)
        t.start()
        t.join()
    stop_time=time.time()
    print('主:%s n:%s' %(stop_time-start_time,n))
    [/code]
```

那么，只要没有共同拥有的全局变量不就可以不加锁了么{:9_220:} 
当你需要做对文件做写入操作时候，就很刺激了，例如下列代码：

```bash
# -*- coding: utf-8 -*-
from threading import current_thread,Thread,Lock
import time,random

value = [x for x in range(1,1001)]
balue = iter(value)

def task():
    print('%s is running' % current_thread().getName())
    params = next(balue)

    # lock.acquire()
    f = open('11.txt', 'a',encoding='utf-8')
    f.write(str(params)+'\n')
    # lock.release()

    time.sleep(random.randint(0,1))
    print('{} params {}'.format(current_thread().getName(),params))
    f.close()

if __name__ == '__main__':
    n=100
    # lock=Lock()
    threads=[]
    start_time=time.time()
    for i in range(1000):
        t=Thread(target=task)
        threads.append(t)
        t.start()
    for t in threads:
        t.join()

    stop_time=time.time()
    print('主:%s n:%s' %(stop_time-start_time,n))[/code]
```
打开文件发现数字并不是1到1000，而是乱序的。。。。
最后，假如文件的读写方式是'w'
```bash
f = open('11.txt', 'w',encoding='utf-8')[/code]
```
那么打开文件会发现 9980 {:9_228:} ，也就是多个线程同时写入了，但是不正常的w写入方式，毕竟假如文件存在会清空里面的内容。。。
所以所，很多时候锁还是要加的，不然数据会乱飞





点击进入我的[fishc][fishc]论坛

[fishc]: https://fishc.com.cn/thread-128891-1-1.html

