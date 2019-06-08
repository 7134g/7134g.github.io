---
layout: post
title: python中的内存泄漏
date: 2018-01-15 19:24:11.000000000 +09:00
---



如果你的程序是死循环，不停歇的代码，下列是需要注意内存的问题。

第一、pillow库的隐患
```bash
#内存将发生泄漏
from PIL import  Image

im = Image.open('1.jpg')
im.save()

#使用with使程序更安全
from PIL import  Image

with open('1.jpg' , 'rb') as open_file:
    im = Image.open(open_file)

```

第二、使用importlib.reload重载模块后带来使用全局变量带来的隐患
假如采取不重启程序方式，自动重新载入修改后的文件，所以需要进行重载模块

```bash
import importlib

while True:
    module_name = importlib.import_module('.', 'test_file')
    module_name = importlib.reload(module_name)
    result = module_name.main(params)

#test_file.py
global_value = {'dataList':[],
                'number':'',}
key = '初始值'

def main(params):

    # params携带着此次任务数据

    global_value['number'] = params['number']
    get_data1(params)
    get_data2(params)
    return global_value

def get_data1(params):
    global key
    # 你的程序通过params得到新的数据
    data_once = {'每次运行产生的键': '每次运行产生的键值'}
    key = '新值'
    global_value['dataList'].append(data_once)

def get_data2(params):
    # 你的程序通过params和key新的值，得到另一份数据
    data_once = {'每次运行产生的新键': '每次运行产生的新键值'}
    global_value['dataList'].append(data_once)
```

上述就会发生一种隐患，以前我觉得垃圾回收机制很靠谱。但是当每一次重载模块时，global_value将使用新的地址，原来的地址还放着上一次的数据，没有被释放掉
就算你在每次循环里添加gc.collect()也不能快速回收删除上一次的数据，导致内存持续增长。。。
我的处理方式是
（1）、
将global_value这个转移进函数内，通过传参将get_data1和get_data2数据整合在一个变量里

（2）、
将所有函数放在一个类中，也可以避免全局变量数据存活时间太长

总结，虽然使用全局变量很省事，不用传参，其他函数改变其值再被其他函数调用很方便，却会导致内存泄漏，因为每一次reload时产生的是新的内存地址。

        


点击进入我的[fishc][fishc]论坛

[fishc]: https://fishc.com.cn/thread-128891-1-1.html

