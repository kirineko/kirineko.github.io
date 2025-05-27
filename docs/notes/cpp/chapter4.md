---
title: 第四章
createTime: 2025/05/28 00:56:11
permalink: /cpp/8uhr5f69/
---

# 第4章：Lambda表达式与函数对象

## 4.1 Lambda表达式基础

### 什么是Lambda表达式？

Lambda表达式是C++11引入的匿名函数，提供了一种简洁的方式来定义函数对象：

```cpp
// 传统函数对象
struct Adder {
    int operator()(int a, int b) const {
        return a + b;
    }
};

// Lambda表达式
auto add = [](int a, int b) { return a + b; };
```

### Lambda语法

```cpp
[capture](parameters) -> return_type { body }
```

- **capture**：捕获列表
- **parameters**：参数列表（可选）
- **return_type**：返回类型（可选）
- **body**：函数体

### 基本示例

```cpp
// 最简单的Lambda
auto hello = []() { std::cout << "Hello, World!" << std::endl; };

// 带参数的Lambda
auto multiply = [](int x, int y) { return x * y; };

// 显式返回类型
auto divide = [](double a, double b) -> double {
    if (b != 0) return a / b;
    throw std::invalid_argument("Division by zero");
};
```

## 4.2 捕获列表详解

### 捕获方式

Lambda可以通过多种方式捕获外部变量：

```cpp
int x = 10;
int y = 20;

// 按值捕获
auto lambda1 = [x](int a) { return x + a; };

// 按引用捕获
auto lambda2 = [&y](int a) { y += a; return y; };

// 混合捕获
auto lambda3 = [x, &y](int a) { y += a; return x + y; };

// 捕获所有（按值）
auto lambda4 = [=](int a) { return x + y + a; };

// 捕获所有（按引用）
auto lambda5 = [&](int a) { x += a; y += a; return x + y; };
```

### 初始化捕获 (C++14)

```cpp
int x = 10;

// 移动捕获
auto lambda = [ptr = std::make_unique<int>(x)](int a) {
    return *ptr + a;
};

// 计算捕获
auto lambda2 = [sum = x + 5](int a) {
    return sum + a;
};
```

### 捕获this指针

```cpp
class Calculator {
    int base_ = 100;
public:
    auto getAdder() {
        return [this](int x) { return base_ + x; };
    }
    
    auto getAdderCopy() {
        return [*this](int x) { return base_ + x; }; // C++17
    }
};
```

## 4.3 std::function与可调用对象

### std::function

`std::function`是一个通用的函数包装器：

```cpp
#include <functional>

std::function<int(int, int)> operation;

// 可以存储Lambda
operation = [](int a, int b) { return a + b; };

// 可以存储函数指针
int multiply(int a, int b) { return a * b; }
operation = multiply;

// 可以存储函数对象
struct Divider {
    int operator()(int a, int b) const { return a / b; }
};
operation = Divider{};
```

### 可调用对象的类型

```cpp
// Lambda的类型是唯一的
auto lambda1 = [](int x) { return x * 2; };
auto lambda2 = [](int x) { return x * 2; }; // 不同类型！

// 使用std::function统一类型
std::vector<std::function<int(int)>> operations;
operations.push_back([](int x) { return x * 2; });
operations.push_back([](int x) { return x + 1; });
```

## 4.4 Lambda与STL算法

### 常用算法示例

```cpp
std::vector<int> numbers{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// std::for_each
std::for_each(numbers.begin(), numbers.end(), 
              [](int& n) { n *= 2; });

// std::find_if
auto it = std::find_if(numbers.begin(), numbers.end(),
                       [](int n) { return n > 10; });

// std::count_if
auto count = std::count_if(numbers.begin(), numbers.end(),
                           [](int n) { return n % 2 == 0; });

// std::transform
std::vector<std::string> strings;
std::transform(numbers.begin(), numbers.end(), 
               std::back_inserter(strings),
               [](int n) { return std::to_string(n); });
```

### 复杂算法应用

```cpp
// 排序
std::vector<std::string> words{"apple", "banana", "cherry", "date"};
std::sort(words.begin(), words.end(),
          [](const std::string& a, const std::string& b) {
              return a.length() < b.length();
          });

// 分区
std::partition(numbers.begin(), numbers.end(),
               [](int n) { return n % 2 == 0; });

// 累积
auto sum = std::accumulate(numbers.begin(), numbers.end(), 0,
                          [](int acc, int n) { return acc + n * n; });
```

## 4.5 泛型Lambda (C++14)

### auto参数

C++14允许Lambda参数使用auto：

```cpp
// 泛型Lambda
auto generic_add = [](auto a, auto b) { return a + b; };

// 可以处理不同类型
auto result1 = generic_add(1, 2);        // int
auto result2 = generic_add(1.5, 2.5);    // double
auto result3 = generic_add(std::string("Hello"), std::string(" World")); // string
```

### 模板Lambda (C++20)

```cpp
// C++20模板Lambda
auto template_lambda = []<typename T>(T a, T b) {
    return a + b;
};

// 约束模板参数
auto constrained_lambda = []<std::integral T>(T a, T b) {
    return a + b;
};
```

### 实际应用

```cpp
// 通用比较器
auto make_comparator = [](auto member_ptr) {
    return [member_ptr](const auto& a, const auto& b) {
        return a.*member_ptr < b.*member_ptr;
    };
};

struct Person {
    std::string name;
    int age;
};

std::vector<Person> people{{"Alice", 30}, {"Bob", 25}, {"Charlie", 35}};

// 按年龄排序
std::sort(people.begin(), people.end(), make_comparator(&Person::age));

// 按姓名排序
std::sort(people.begin(), people.end(), make_comparator(&Person::name));
```

## Lambda最佳实践

### 1. 选择合适的捕获方式

```cpp
// 好：明确捕获需要的变量
auto lambda = [x, &y](int a) { return x + y + a; };

// 避免：不必要的全捕获
auto lambda_bad = [=](int a) { return x + y + a; }; // 可能捕获很多不需要的变量
```

### 2. 注意捕获的生命周期

```cpp
std::function<int()> createLambda() {
    int local = 42;
    
    // 危险：按引用捕获局部变量
    return [&local]() { return local; }; // 悬空引用！
    
    // 安全：按值捕获
    return [local]() { return local; };
}
```

### 3. 使用mutable关键字

```cpp
int x = 10;
auto lambda = [x](int a) mutable {
    x += a; // 修改捕获的副本
    return x;
};
```

### 4. 递归Lambda

```cpp
// C++14递归Lambda
auto factorial = [](int n) {
    auto impl = [](int n, auto& self) -> int {
        return n <= 1 ? 1 : n * self(n - 1, self);
    };
    return impl(n, impl);
};
```

---

## 本章小结

Lambda表达式是现代C++的重要特性：
- **简洁语法**：替代传统函数对象
- **灵活捕获**：按值、按引用、初始化捕获
- **STL集成**：与算法库完美配合
- **泛型支持**：auto参数和模板Lambda
- **函数式编程**：支持高阶函数和函数组合

下一章我们将学习现代STL容器与算法，了解C++11后新增的容器和算法特性。
