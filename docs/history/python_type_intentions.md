---
title: Python类型的设计意图
author: kirineko
summary: Python中几种常用类型的设计意图
tags:
  - python
  - type
createTime: 2022/09/28 00:26:01
permalink: /article/jvva1jqx/
---

讨论Python中**List**, **String**, **Generator**, **Tuple**, **Set**, **Dictionary**等几种常见类型的设计意图

---

### 1. List

- 可迭代
- 可变
- 较少的从列表的中间检索值
- 可能会有元素重复
- cookbooks on a shelf

### 2. String

- 不可变
- 字符集
- name of cookbook

### 3. Generator

- 只适用迭代，不使用索引
- 惰性获取
- 可能会有无数个元素
- online databases of recipes

### 4. Tuple

- 不可变
- 可能从元组中间检索值(索引或拆包)
- 较少迭代
- (cookbook_name, author, pagecount)

### 5. Set

- 没有重复元素
- 不依赖元素顺序

### 6. Dictionary

- 键值对映射
- key唯一
- 通常用于迭代或索引查找

### 7. frozenset

- 不可变的集合

### 8. OrderedDict

- 有序字典
- 按照插入顺序的字典
- CPython 3.6以后dict也都是OrderedDict

### 9. defaultdict

- 默认值字典

### 10. Counter

- 计数器: 对可迭代对象进行计数

---

### 1. For

- 用于迭代元素
- 对迭代的元素进行操作或产生副作用

### 2. while

- 根据特定条件进行迭代

### 3. 列表理解

- 迭代并转换
- 无副作用

### 4. 递归

- 子结构类型与父结构类型相同
