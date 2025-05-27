---
title: 第一章
createTime: 2025/05/28 00:56:11
permalink: /cpp/1so4e657/
---

# 第1章：现代C++基础

## 1.1 现代C++概述与环境搭建

### 什么是现代C++？

现代C++指的是C++11及以后的C++标准。相比传统C++，现代C++引入了许多革命性的特性：

- **自动类型推导** (auto)
- **智能指针** (unique_ptr, shared_ptr)
- **移动语义** (move semantics)
- **Lambda表达式**
- **统一初始化**
- **并发支持** (std::thread)

### 环境搭建

确保您的编译器支持C++17或更高版本：

```bash
# 检查GCC版本
g++ --version

# 检查Clang版本
clang++ --version

# 编译C++17代码示例
g++ -std=c++17 -Wall -Wextra -O2 main.cpp -o main
```

## 1.2 auto关键字与类型推导

### 基本用法

`auto`关键字让编译器自动推导变量类型，使代码更简洁、更安全。

**传统方式 vs 现代方式：**

```cpp
// 传统方式 - 冗长且容易出错
std::vector<std::string>::iterator it = vec.begin();
std::map<std::string, int>::const_iterator map_it = mymap.cbegin();

// 现代方式 - 简洁明了
auto it = vec.begin();
auto map_it = mymap.cbegin();
```

### auto的最佳实践

1. **用于复杂类型**
2. **用于模板编程**
3. **避免类型转换错误**

```cpp
// 避免意外的类型转换
auto size = vec.size();  // size_t类型，而不是int
auto value = getValue(); // 获得正确的返回类型
```

## 1.3 统一初始化语法

### 大括号初始化

C++11引入了统一的初始化语法，使用大括号`{}`：

```cpp
// 基本类型
int x{42};
double pi{3.14159};

// 容器初始化
std::vector<int> numbers{1, 2, 3, 4, 5};
std::map<std::string, int> ages{
    {"Alice", 25},
    {"Bob", 30},
    {"Charlie", 35}
};

// 类对象初始化
class Point {
public:
    Point(int x, int y) : x_(x), y_(y) {}
private:
    int x_, y_;
};

Point p{10, 20};  // 统一初始化
```

### 优势

1. **防止窄化转换**
2. **统一语法**
3. **避免"最令人困惑的解析"**

## 1.4 nullptr与类型安全

### 告别NULL

`nullptr`是类型安全的空指针常量，替代传统的`NULL`或`0`：

```cpp
// 传统方式 - 类型不安全
void func(int);
void func(char*);

func(NULL);  // 歧义！可能调用func(int)

// 现代方式 - 类型安全
func(nullptr);  // 明确调用func(char*)
```

### 最佳实践

```cpp
// 总是使用nullptr
int* ptr = nullptr;
if (ptr != nullptr) {
    // 安全的指针检查
}

// 与智能指针配合
std::unique_ptr<int> smart_ptr = nullptr;
```

## 1.5 范围for循环

### 简化容器遍历

范围for循环（range-based for loop）大大简化了容器遍历：

```cpp
std::vector<std::string> names{"Alice", "Bob", "Charlie"};

// 传统方式
for (std::vector<std::string>::iterator it = names.begin(); 
     it != names.end(); ++it) {
    std::cout << *it << std::endl;
}

// 现代方式 - 只读遍历
for (const auto& name : names) {
    std::cout << name << std::endl;
}

// 现代方式 - 修改元素
for (auto& name : names) {
    name += " Smith";
}
```

### 适用场景

- **容器遍历**
- **数组遍历**
- **字符串遍历**
- **自定义类型（实现begin/end）**

---

## 本章小结

现代C++的基础特性让代码更加：
- **简洁**：auto减少冗余类型声明
- **安全**：nullptr避免类型错误，统一初始化防止窄化
- **易读**：范围for循环表达意图更清晰

下一章我们将学习现代C++最重要的特性之一：智能指针与RAII内存管理。
