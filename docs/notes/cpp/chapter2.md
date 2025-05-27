---
title: 第二章
createTime: 2025/05/28 00:56:11
permalink: /cpp/1gfyvu7d/
---

# 第2章：智能指针与内存管理

## 2.1 RAII原则

### 什么是RAII？

RAII (Resource Acquisition Is Initialization) 是现代C++的核心设计原则：
- **资源获取即初始化**
- **资源释放即析构**
- **异常安全保证**

### RAII的优势

```cpp
// 传统方式 - 容易忘记释放资源
void oldStyle() {
    int* ptr = new int(42);
    // ... 复杂逻辑，可能抛出异常
    delete ptr;  // 可能永远不会执行！
}

// RAII方式 - 自动管理资源
void modernStyle() {
    auto ptr = std::make_unique<int>(42);
    // ... 复杂逻辑，即使抛出异常
    // ptr会在作用域结束时自动释放
}
```

## 2.2 unique_ptr：独占所有权

### 基本概念

`unique_ptr`表示对资源的独占所有权：
- **不可复制，只能移动**
- **零开销抽象**
- **异常安全**

### 基本用法

```cpp
#include <memory>

// 创建unique_ptr
auto ptr1 = std::make_unique<int>(42);
auto ptr2 = std::unique_ptr<int>(new int(42)); // 不推荐

// 访问值
std::cout << *ptr1 << std::endl;
std::cout << ptr1.get() << std::endl; // 获取原始指针

// 检查是否为空
if (ptr1) {
    std::cout << "ptr1 is not null" << std::endl;
}

// 释放所有权
ptr1.reset();        // 释放并设为nullptr
ptr1.reset(new int(100)); // 释放旧资源，管理新资源
```

### 移动语义

```cpp
auto ptr1 = std::make_unique<int>(42);
auto ptr2 = std::move(ptr1); // 移动所有权

// 现在ptr1为nullptr，ptr2拥有资源
assert(ptr1 == nullptr);
assert(*ptr2 == 42);
```

## 2.3 shared_ptr：共享所有权

### 基本概念

`shared_ptr`允许多个指针共享同一资源：
- **引用计数管理**
- **线程安全的引用计数**
- **最后一个shared_ptr析构时释放资源**

### 基本用法

```cpp
// 创建shared_ptr
auto ptr1 = std::make_shared<int>(42);
auto ptr2 = ptr1; // 复制，引用计数增加

std::cout << "引用计数: " << ptr1.use_count() << std::endl; // 输出: 2

// 重置一个指针
ptr2.reset();
std::cout << "引用计数: " << ptr1.use_count() << std::endl; // 输出: 1
```

### 自定义删除器

```cpp
// 自定义删除器
auto ptr = std::shared_ptr<FILE>(
    fopen("test.txt", "w"),
    [](FILE* f) {
        if (f) {
            std::cout << "关闭文件" << std::endl;
            fclose(f);
        }
    }
);
```

## 2.4 weak_ptr：打破循环引用

### 循环引用问题

```cpp
struct Node {
    std::shared_ptr<Node> next;
    std::shared_ptr<Node> prev; // 这会造成循环引用！
};

auto node1 = std::make_shared<Node>();
auto node2 = std::make_shared<Node>();
node1->next = node2;
node2->prev = node1; // 循环引用，内存泄漏！
```

### weak_ptr解决方案

```cpp
struct Node {
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev;   // 使用weak_ptr打破循环
};

auto node1 = std::make_shared<Node>();
auto node2 = std::make_shared<Node>();
node1->next = node2;
node2->prev = node1; // 不会造成循环引用

// 使用weak_ptr
if (auto prev = node2->prev.lock()) {
    // prev是一个临时的shared_ptr
    std::cout << "Previous node exists" << std::endl;
}
```

### weak_ptr的用法

```cpp
auto shared = std::make_shared<int>(42);
std::weak_ptr<int> weak = shared;

// 检查资源是否还存在
if (!weak.expired()) {
    if (auto locked = weak.lock()) {
        std::cout << *locked << std::endl;
    }
}

shared.reset(); // 释放资源
assert(weak.expired()); // weak_ptr检测到资源已释放
```

## 2.5 make_unique和make_shared

### 为什么使用make函数？

1. **异常安全**
2. **性能优化** (make_shared)
3. **代码简洁**

### 异常安全示例

```cpp
// 不安全的方式
func(std::shared_ptr<int>(new int(42)), 
     std::shared_ptr<int>(new int(43)));
// 如果第二个new抛出异常，第一个int可能泄漏

// 安全的方式
func(std::make_shared<int>(42), 
     std::make_shared<int>(43));
// 异常安全保证
```

### 性能优化

```cpp
// make_shared的优势：一次内存分配
auto ptr1 = std::make_shared<int>(42);
// 控制块和对象在同一内存块中

// 传统方式：两次内存分配
auto ptr2 = std::shared_ptr<int>(new int(42));
// 控制块和对象分别分配
```

### 自定义类型示例

```cpp
class Person {
public:
    Person(const std::string& name, int age) 
        : name_(name), age_(age) {}
    
    void introduce() const {
        std::cout << "Hi, I'm " << name_ << ", " << age_ << " years old." << std::endl;
    }
    
private:
    std::string name_;
    int age_;
};

// 使用make_unique和make_shared
auto person1 = std::make_unique<Person>("Alice", 25);
auto person2 = std::make_shared<Person>("Bob", 30);

person1->introduce();
person2->introduce();
```

## 智能指针选择指南

### 何时使用unique_ptr？
- **独占所有权**
- **资源传递**
- **工厂函数返回值**
- **PIMPL惯用法**

### 何时使用shared_ptr？
- **共享所有权**
- **对象生命周期复杂**
- **回调函数需要保持对象存活**

### 何时使用weak_ptr？
- **打破循环引用**
- **观察者模式**
- **缓存实现**

---

## 本章小结

智能指针是现代C++内存管理的基石：
- **unique_ptr**：独占所有权，零开销
- **shared_ptr**：共享所有权，引用计数
- **weak_ptr**：弱引用，打破循环
- **make_unique/make_shared**：安全创建

下一章我们将学习移动语义，这是现代C++性能优化的关键特性。
