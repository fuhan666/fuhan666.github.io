---
title: Python
publishDate: 2023-05-26 09:55:00
updateDate: 2025-05-24 00:02:00
description: '在 VPS 中安装 3x-ui 面板并请用 BBR'
tags:
  - Python
language: 'Chinese'
---

## 去除列表中重复的字典

```python
# 原始字典列表
a = [
    {'a': 123, 'b': 1234},
    {'a': 3222, 'b': 1234},
    {'a': 123, 'b': 1234},
]
```

### tuple of items（最快）

将列表中的字典转换为元组的元组 (tuple of items)，因为字典本身是不可哈希的，但元组是。然后再将结果转换回字典列表。

```python
b1 = [dict(t) for t in {tuple(d.items()) for d in a}]
print("方法一去重结果:", b1)
```

### reduce

```python
from functools import reduce
def deleteDuplicate_method2(li):
    func = lambda x, y: x if y in x else x + [y]
    # 初始值 [] 很重要，因为它定义了累加的类型和起始点
    processed_li = reduce(func, [[], ] + li)
    return processed_li
b2 = deleteDuplicate_method2(a)
print("方法二去重结果:", b2)
```

### 传入某个key，根据key来去重，不比较整个字典

```python
def DelRepeat_method3(data,key):
    new_data = [] # 用于存储去重后的list
    values = []   # 用于存储当前已有的值
    for d in data:
        if d[key] not in values:
            new_data.append(d)
            values.append(d[key])
    return new_data
b3 = DelRepeat_method3(a, 'a')
print("方法三去重结果 (基于key 'a'):", b3)
```

### dict -> set -> list -> eval()

```python
def deleteDuplicate_method4(li):
    # 先将字典转换为字符串，利用 set 去重，然后再转换回字典
    # 注意：eval() 函数存在安全风险，如果输入不是完全可信的，应避免使用。
    # 这种方法对于包含复杂对象的字典可能不总是有效，且效率较低。
    temp_list = list(set([str(i) for i in li]))
    processed_li=[eval(i) for i in temp_list]
    return processed_li
b4 = deleteDuplicate_method4(a)
print("方法四去重结果:", b4)
```

## pip 更换源

### 临时切换源

```bash
pip install 包名 -i https://pypi.org/simple
```

### 官方源

```bash
pip config set global.index-url  https://pypi.org/simple
```

### 清华源

```bash
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### 阿里源

```bash
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple
```

###　腾讯源

```bash
pip config set global.index-url http://mirrors.cloud.tencent.com/pypi/simple
```

### 豆瓣源

```bash
pip config set global.index-url http://pypi.douban.com/simple
```

## 开启代理后使用pip安装包时出现SOCKS问题

报错如下：

```bash
[InvalidSchema]: Missing dependencies for SOCKS support.
```

> 这种情况可以参考
>
> https://stackoverflow.com/questions/38794015/pythons-requests-missing-dependencies-for-socks-support-when-using-socks5-fro

第一种解决方法：
先关掉代理，然后`pip install pysocks`

第二种解决方法：
`echo $all_proxy`查看当前的`all_proxy`（此命令适用于bash/zsh等shell），然后看如下解答：

> I changed my environment variable `all_proxy` (which was originally set to a SOCK proxy `socks://....`) to the https version in my .bashrc file:
>
> `export all_proxy="https://<proxy>:<port>/"`

## 新建项目

使用 uv 管理虚拟环境

使用 ruff 来格式化代码
