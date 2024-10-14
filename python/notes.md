# 文献

Python Web 内存调优，以及 Python 中的 Copy-on-Write
https://dev.admirable.pro/python-web-memory-tunning-and-copy-on-write/

Python垃圾回收和Linux Copy-on-Write机制 - 成蹊0xc000 - 博客园
https://www.cnblogs.com/dennis-wong/p/15782824.html


# 工程

中文乱码
```py
# -*- coding: utf-8 -*
#!/usr/bin/python
```

## 包-模块管理
```
import requests
from requests import xx

```

- 导出包
文件结构如下
```
__init__.py
log.py
common/
|- __init__.py
|- common.py
```

则在 `__init__.py` 可以导出
```
from .log import logger
from . import common
```

# 数据类型

- 类型操作

```py
# 类型断言
if type(foo) is not list:
    pass

if isinstance(foo, list):
    pass

# 函数签名类型
def get_objects(generation: int | None = None) -> list[Any]: ...

# 装箱类型
from typing import List, Dict

```

## 字符串

```py
foo = '''

'''

# bytes 转 str
bs.encode("utf-8")

# str 函数
s = "hello world"
s.startswith()

print("" is None)
print("" is not None)
print("" == "")
```

## 其他
```py
# bytes 转 str
bs.encode("utf-8")

dict = {
    "k1": "v1",
    "k2": "v2"
}
```

# 基础编程

```py
# 流程处理
if num == 3:            # 判断num的值
    print('boss')
elif num == 2:
    print('user')
else:
    print('roadman')     # 条件均不成立时输出

# 异常处理
try:
    # do
except KeyError as error:
    print(error)
else:
    pass

# 类型转换
int()
str()
```

# 面相对象

```py
class ClassA():
    # 构造函数
    def __init__(self):
        pass

    # 函数调用
    def __call__(self, draw_params={}):
        pass

# 固有的特殊成员
A.__dict__

# 继承
class ObjData(object):
    def __init__(self, data):
        self.__dict__ = deepcopy(data)

    def __call__(self, data):
        self.__dict__.update(data)
```

```py
class MyClass:
    def __init__(self, name, age):
        self._name = name
        self.__age = age

    def display(self):
        print(f"Name: {self._name}, Age: {self.__age}")

    def _private_method(self):
        print("This is a private method with a single underscore.")

    def __private_method(self):
        print("This is a private method with a double underscore.")

# 创建 MyClass 的实例
obj = MyClass("Alice", 30)

# 调用 display 方法
obj.display()  # 输出: Name: Alice, Age: 30

# 尝试访问单下划线前缀的私有成员变量（可以访问）
print(obj._name)  # 输出: Alice
# 尝试调用单下划线前缀的私有方法（可以调用）
obj._private_method()  # 输出: This is a private method with a single underscore.

# 尝试访问双下划线前缀的私有成员变量（无法访问）
print(obj.__age)  # 报错：AttributeError: 'MyClass' object has no attribute '__age'
# 尝试调用双下划线前缀的私有方法（无法调用）
obj.__private_method()  # 报错：AttributeError: 'MyClass' object has no attribute '__private_method'
```

# 工具库

## 日志 print

```py
print()
name = 'word'
age = 13
s1 = 'hello %s' % name
s2 = 'I am %s, %d years old' % (name, age)
```

```py
foo = 1
bar = "hello"
oneStr = f"foo is {foo}"
twoStr = "bar is {}".format(foo)
```


## requests

```py
import requests

payload = "{}"
headers = {
    'Content-Type': "application/json",
}

response = requests.request("POST", url, data=payload, headers=headers)
if response.status_code != 200:
    print(response.text)
    exit(1)

return response.json()["data"]
```

## 命令行

```py
import sys
sys.argv[1]
```
