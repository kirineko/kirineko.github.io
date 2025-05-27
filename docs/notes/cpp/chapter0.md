---
title: 序章
createTime: 2025/05/28 00:56:11
permalink: /cpp/2fymxx9s/
---

# 现代C++教程 (C++11及以后)

## 课程简介

本教程专注于现代C++编程，完全基于C++11及以后的标准，摒弃传统C++的过时做法，采用现代C++的最佳实践。

## 课程大纲

### 第1章：现代C++基础
- 1.1 现代C++概述与环境搭建
- 1.2 auto关键字与类型推导
- 1.3 统一初始化语法
- 1.4 nullptr与类型安全
- 1.5 范围for循环

### 第2章：智能指针与内存管理
- 2.1 RAII原则
- 2.2 unique_ptr：独占所有权
- 2.3 shared_ptr：共享所有权
- 2.4 weak_ptr：打破循环引用
- 2.5 make_unique和make_shared

### 第3章：移动语义与完美转发
- 3.1 左值与右值
- 3.2 移动构造函数与移动赋值
- 3.3 std::move与std::forward
- 3.4 完美转发
- 3.5 移动语义的性能优化

### 第4章：Lambda表达式与函数对象
- 4.1 Lambda表达式基础
- 4.2 捕获列表详解
- 4.3 std::function与可调用对象
- 4.4 Lambda与STL算法
- 4.5 泛型Lambda (C++14)

### 第5章：现代STL容器与算法 ✅
- 5.1 新增容器：array, unordered_map, unordered_set
- 5.2 容器的emplace操作
- 5.3 现代算法库
- 5.4 范围算法 (C++20预览)
- 5.5 迭代器适配器

### 第6章：并发编程 ✅
- 6.1 std::thread基础
- 6.2 互斥量与锁
- 6.3 条件变量
- 6.4 原子操作
- 6.5 异步编程：future与promise

### 第7章：模板元编程现代化 ✅
- 7.1 变参模板
- 7.2 模板别名
- 7.3 constexpr编程
- 7.4 SFINAE与enable_if
- 7.5 概念 (C++20预览)

### 第8章：异常安全与错误处理 ✅
- 8.1 异常安全保证
- 8.2 RAII与异常
- 8.3 noexcept规范
- 8.4 std::optional (C++17)
- 8.5 错误处理最佳实践

### 第9章：现代类设计 ✅
- 9.1 Rule of Zero/Three/Five
- 9.2 默认与删除函数
- 9.3 委托构造函数
- 9.4 继承构造函数
- 9.5 final与override关键字

### 第10章：实战项目 ✅
- 10.1 现代C++设计模式
- 10.2 性能优化技巧
- 10.3 代码组织与项目结构
- 10.4 单元测试与现代工具链
- 10.5 综合项目实战

### 第11章：C++17核心特性 ✅
- 11.1 结构化绑定 (Structured Bindings)
- 11.2 std::string_view
- 11.3 std::variant
- 11.4 std::optional深入
- 11.5 std::any

### 第12章：C++20 Concepts ✅
- 12.1 Concepts基础
- 12.2 标准库Concepts
- 12.3 自定义Concepts
- 12.4 requires表达式
- 12.5 约束模板的方式

### 第13章：C++20其他重要特性 ✅
- 13.1 三路比较运算符 (Spaceship Operator)
- 13.2 格式化库 (Format Library)
- 13.3 范围库 (Ranges)
- 13.4 consteval 和 constinit
- 13.5 协程 (Coroutines)
- 13.6 其他新特性

## 学习目标

完成本教程后，您将能够：
1. 熟练使用现代C++特性编写高效、安全的代码
2. 理解并应用RAII、移动语义等核心概念
3. 掌握现代C++的内存管理和并发编程
4. 编写符合现代C++最佳实践的生产级代码

## 前置要求

- 基本的编程概念理解
- 建议有一定的C或其他编程语言基础
- 不需要传统C++经验（我们将直接学习现代方式）

## 开发环境

- C++17或更高版本的编译器 (推荐GCC 9+, Clang 10+, MSVC 2019+)
- CMake 3.15+ (可选，也可使用Makefile)
- 推荐IDE：VS Code, CLion, Visual Studio

## 快速开始

### 方法1：使用Makefile (推荐)
```bash
# 编译所有示例
make all

# 运行所有示例
make run-all

# 运行特定章节
make run-chapter1
make run-chapter2
make run-chapter3
make run-chapter4
make run-chapter11
make run-chapter12
make run-chapter13

# 查看帮助
make help
```

### 方法2：使用构建脚本
```bash
# Linux/macOS
./build_and_run.sh

# Windows
build_and_run.bat
```

### 方法3：手动编译
```bash
# 编译单个示例
g++ -std=c++17 -Wall -Wextra -O2 chapter01/examples.cpp -o chapter01_examples

# 运行
./chapter01_examples
```

## 项目结构

```
cpp_learn/
├── README.md                    # 项目概述
├── LEARNING_GUIDE.md           # 详细学习指南
├── Makefile                    # 构建文件
├── CMakeLists.txt             # CMake配置
├── build_and_run.sh           # Linux/macOS构建脚本
├── build_and_run.bat          # Windows构建脚本
├── chapter01/                 # 第1章：现代C++基础
│   ├── README.md
│   └── examples.cpp
├── chapter02/                 # 第2章：智能指针与内存管理
│   ├── README.md
│   └── smart_pointers.cpp
├── chapter03/                 # 第3章：移动语义与完美转发
│   ├── README.md
│   └── move_semantics.cpp
├── chapter04/                 # 第4章：Lambda表达式与函数对象
│   ├── README.md
│   └── lambda_examples.cpp
├── chapter05/                 # 第5章：现代STL容器与算法
│   ├── README.md
│   └── stl_examples.cpp
├── chapter06/                 # 第6章：并发编程
│   ├── README.md
│   └── concurrency_examples.cpp
├── chapter07/                 # 第7章：模板元编程现代化
│   ├── README.md
│   └── template_examples.cpp
├── chapter08/                 # 第8章：异常安全与错误处理
│   └── README.md
├── chapter09/                 # 第9章：现代类设计
│   └── README.md
├── chapter10/                 # 第10章：实战项目
│   └── README.md
├── chapter11/                 # 第11章：C++17核心特性
│   ├── README.md
│   └── cpp17_features.cpp
├── chapter12/                 # 第12章：C++20 Concepts
│   ├── README.md
│   └── concepts_examples.cpp
└── chapter13/                 # 第13章：C++20其他重要特性
    ├── README.md
    └── cpp20_features.cpp
```

---

*注：本教程将持续更新，跟进C++标准的发展*