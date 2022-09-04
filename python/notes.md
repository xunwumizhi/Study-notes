# 工程

中文乱码
```py
# -*- coding: utf-8 -*
#!/usr/bin/python
```

包管理
```
import requests
from requests import xx
```

# 数据类型

## 字符串

```py
# bytes 转 str
bs.encode("utf-8")
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
if code != 200:
    # do
if code == 400:

# 异常处理
try:
    # do
except KeyError as error:
    print(error)
else:

# 类型断言
if isinstance(cpu, int):
# 类型转换
int()
str()
```

# 工具库

## 日志 print

```py
print()
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
