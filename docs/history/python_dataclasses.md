---
title: Python Dataclasses
author: kirineko
summary: Python Dataclass
tags:
  - python
  - type
  - dataclass
createTime: 2022/09/28 00:26:01
permalink: /article/3grnm2ev/
---

Dataclasses是一些适合于存储数据对象`(data object)`的Python类。 Python 3.7提供了一个装饰器dataclass, 用以把一个类转化为dataclass.

---

### 1. 什么是Dataclass

``` python
from dataclasses import dataclass

@dataclass
class Book:
    title: str
```

等价于

``` python
class Book:
    def __init__(self, title: str):
        self.title = title

    def __repr__(self):
        return f"{self.__class__.__name__}(title={self.title!r})"

    def __eq__(self, other: Book):
        return (self.title,) == (other.title,)
```

### 2. Dataclass的特点

- 支持类型定义
- 支持默认值
- 支持比较
- 无需编写`__init__`,`__repr__`,`__eq__`方法

### 3. dataclass装饰器参数

@dataclass(init=True, repr=True, eq=True, order=False, unsafe_hash=False, frozen=False)

- init: 自动生成初始化函数
- repr: 自动生成repr函数
- eq: 自动生成比较函数
- order: 生成`__lt__`,`__le__`,`__gt__`,`__ge__`函数, 支持排序
- frozen: 生成不可变的dataclass

### 4. 其他说明

- `__post_init__`方法: 支持`__init__`后的后处理
- 继承: Dataclass支持继承, 需要注意继承的参数顺序
