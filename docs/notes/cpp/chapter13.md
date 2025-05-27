---
title: 第十三章
createTime: 2025/05/28 00:56:12
permalink: /cpp/6yg6vxw6/
---

# 第13章：C++20其他重要特性

## 本章概述

除了Concepts之外，C++20还引入了许多其他重要特性，包括模块系统、协程、范围库、格式化库等。这些特性进一步提升了C++的现代化程度和开发效率。

## 学习目标

- 理解C++20模块系统的基本概念
- 掌握协程的基本用法
- 学会使用范围库进行函数式编程
- 了解格式化库的使用
- 掌握三路比较运算符
- 学习consteval和constinit关键字

## 13.1 模块系统 (Modules)

### 基本概念

模块是C++20引入的新的代码组织方式，用来替代传统的头文件包含机制。模块提供了：
- 更快的编译速度
- 更好的封装性
- 避免宏污染
- 更清晰的依赖关系

### 基本语法

```cpp
// 模块接口文件 (math_module.cppm)
export module math_module;

export int add(int a, int b) {
    return a + b;
}

export namespace math {
    double pi = 3.14159;
    double square(double x) {
        return x * x;
    }
}
```

```cpp
// 使用模块
import math_module;

int main() {
    int result = add(5, 3);
    double area = math::pi * math::square(2.0);
    return 0;
}
```

### 模块的优势

1. **编译性能提升**：模块只需编译一次，避免重复解析
2. **更好的封装**：只导出需要的符号
3. **避免宏冲突**：模块不会导出宏定义
4. **依赖管理**：清晰的导入/导出关系

## 13.2 协程 (Coroutines)

### 基本概念

协程是可以暂停和恢复执行的函数。C++20提供了三个关键字：
- `co_await`：暂停协程并等待结果
- `co_yield`：暂停协程并产生一个值
- `co_return`：从协程返回

### 简单示例

```cpp
#include <coroutine>
#include <iostream>

// 简单的生成器
struct Generator {
    struct promise_type {
        int current_value;
        
        Generator get_return_object() {
            return Generator{std::coroutine_handle<promise_type>::from_promise(*this)};
        }
        
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        void unhandled_exception() {}
        
        std::suspend_always yield_value(int value) {
            current_value = value;
            return {};
        }
    };
    
    std::coroutine_handle<promise_type> h;
    
    Generator(std::coroutine_handle<promise_type> handle) : h(handle) {}
    ~Generator() { if (h) h.destroy(); }
    
    bool next() {
        h.resume();
        return !h.done();
    }
    
    int value() {
        return h.promise().current_value;
    }
};

Generator fibonacci() {
    int a = 0, b = 1;
    while (true) {
        co_yield a;
        auto temp = a;
        a = b;
        b = temp + b;
    }
}
```

### 协程的应用场景

1. **异步编程**：处理I/O操作
2. **生成器**：惰性计算序列
3. **状态机**：复杂的状态转换逻辑

## 13.3 范围库 (Ranges)

### 基本概念

范围库提供了函数式编程风格的算法，支持链式调用和惰性求值。

### 基本用法

```cpp
#include <ranges>
#include <vector>
#include <iostream>

std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// 传统方式
std::vector<int> result;
std::copy_if(numbers.begin(), numbers.end(), std::back_inserter(result),
             [](int n) { return n % 2 == 0; });
std::transform(result.begin(), result.end(), result.begin(),
               [](int n) { return n * n; });

// 范围库方式
auto result2 = numbers 
    | std::views::filter([](int n) { return n % 2 == 0; })
    | std::views::transform([](int n) { return n * n; });
```

### 常用视图

1. **filter**：过滤元素
2. **transform**：转换元素
3. **take**：取前N个元素
4. **drop**：跳过前N个元素
5. **reverse**：反转序列

## 13.4 格式化库 (Format Library)

### 基本概念

C++20引入了类似Python的字符串格式化功能，提供类型安全的格式化。

### 基本用法

```cpp
#include <format>
#include <iostream>

int main() {
    std::string name = "Alice";
    int age = 25;
    double height = 1.68;
    
    // 基本格式化
    std::string msg = std::format("Hello, {}!", name);
    
    // 位置参数
    std::string info = std::format("{0} is {1} years old and {2:.1f}m tall", 
                                   name, age, height);
    
    // 命名参数（C++20暂不支持，但可以用位置）
    std::string formatted = std::format("Name: {}, Age: {}, Height: {:.2f}", 
                                        name, age, height);
    
    std::cout << msg << std::endl;
    std::cout << info << std::endl;
    std::cout << formatted << std::endl;
    
    return 0;
}
```

### 格式化选项

1. **对齐**：`{:>10}`, `{:<10}`, `{:^10}`
2. **填充**：`{:*>10}`
3. **精度**：`{:.2f}`
4. **进制**：`{:x}`, `{:o}`, `{:b}`

## 13.5 三路比较运算符 (Spaceship Operator)

### 基本概念

`<=>`运算符可以一次性定义所有比较操作，返回强序、弱序或偏序的结果。

### 基本用法

```cpp
#include <compare>

class Point {
    int x, y;
public:
    Point(int x, int y) : x(x), y(y) {}
    
    // 自动生成所有比较运算符
    auto operator<=>(const Point& other) const = default;
    
    // 或者自定义比较逻辑
    std::strong_ordering operator<=>(const Point& other) const {
        if (auto cmp = x <=> other.x; cmp != 0) return cmp;
        return y <=> other.y;
    }
};
```

### 比较类别

1. **strong_ordering**：强序（如整数）
2. **weak_ordering**：弱序（如大小写不敏感字符串）
3. **partial_ordering**：偏序（如浮点数，有NaN）

## 13.6 consteval 和 constinit

### consteval

`consteval`函数必须在编译时求值，比`constexpr`更严格。

```cpp
consteval int square(int n) {
    return n * n;
}

constexpr int value = square(5);  // OK，编译时计算
// int runtime_value = square(x);  // 错误，x不是编译时常量
```

### constinit

`constinit`确保变量在编译时初始化，避免静态初始化顺序问题。

```cpp
constinit int global_value = 42;  // 编译时初始化
```

## 13.7 其他重要特性

### designated initializers

```cpp
struct Point {
    int x, y, z;
};

Point p = {.x = 1, .y = 2, .z = 3};  // C++20指定初始化器
```

### template parameter lists for generic lambdas

```cpp
auto lambda = []<typename T>(T value) {
    return value * 2;
};
```

### 改进的聚合初始化

```cpp
struct Base {
    int x;
};

struct Derived : Base {
    int y;
};

Derived d = {{1}, 2};  // C++20支持
```

## 本章小结

C++20引入的这些特性大大提升了C++的现代化程度：

1. **模块系统**改善了编译性能和代码组织
2. **协程**为异步编程提供了新的范式
3. **范围库**使函数式编程更加自然
4. **格式化库**提供了类型安全的字符串格式化
5. **三路比较**简化了比较操作的定义
6. **consteval/constinit**增强了编译时计算能力

这些特性共同构成了现代C++的重要基础，值得深入学习和实践。 