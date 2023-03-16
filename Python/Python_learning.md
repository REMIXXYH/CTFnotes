### 一些语法和写法

#### with语句

Python对with的处理还很聪明。基本思想是with所求值的对象必须有一个**enter**()方法，一个**exit**()方法。

样例：

```python
#!/usr/bin/env python
# with_example01.py
 
class Sample:
    def __enter__(self):
        print "In __enter__()"
        return "Foo"
 
    def __exit__(self, type, value, trace):
        print "In __exit__()"
 
def get_sample():
    return Sample()
 
with get_sample() as sample:
    print "sample:", sample

代码输出结果如下：
In __enter__()
sample: Foo
In __exit__()
```

一个读取文件的优雅写法：

```python
with open("/tmp/foo.txt") as file:
    data = file.read()
```



### 并发编程

`threading` 库可以在单独的线程中执行任何的在 Python 中可以调用的对象。你可以创建一个 `Thread` 对象并将你要执行的对象以 target 参数的形式提供给该对象。 下面是一个简单的例子：

```python
import time
def countdown(n):
    while n > 0:
        print('T-minus', n)
        n -= 1
        time.sleep(5)

# Create and launch a thread
from threading import Thread
t = Thread(target=countdown, args=(10,))
t.start()
```





### 一些特殊函数

#### encode()和decode()

encode() 方法为字符串类型（str）提供的方法，用于将 str 类型转换成 bytes 类型，这个过程也称为“编码”。

```python
str.encode([encoding="utf-8"][,errors="strict"])
```

默认采用`utf-8`编码，如果想用简体中文，可以设置`gb2312`，

和 encode() 方法正好相反，decode() 方法用于将 bytes 类型的二进制数据转换为 str 类型，这个过程也称为“解码”。

```python
bytes.decode([encoding="utf-8"][,errors="strict"])
```



#### hexdigest()和hash.digest()

hash.digest()返回摘要，作为二进制数据字符串值

`b'\x0c\xc1u\xb9\xc0\xf1\xb6\xa81\xc3\x99\xe2iw&a'`

hash.hexdigest()返回摘要，作为十六进制数据字符串值

`0cc175b9c0f1b6a831c399e269772661`



#### MD5()

以字节序列作为输入，并返回128位哈希值作为输出

常用：

```python
i=hashlib.md5(target.encode()).hexdigest()
```

