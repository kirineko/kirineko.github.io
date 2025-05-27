---
title: 第七章
createTime: 2025/05/28 00:56:11
permalink: /cpp/pq5shtfi/
---


# 第7章：模板元编程现代化

## 7.1 变参模板

### 基本语法

变参模板（Variadic Templates）是C++11引入的强大特性，允许模板接受可变数量的参数：

```cpp
// 基本变参模板
template<typename... Args>
void print(Args... args) {
    ((std::cout << args << " "), ...);  // C++17折叠表达式
    std::cout << std::endl;
}

// 使用
print(1, 2.5, "hello", 'c');  // 输出: 1 2.5 hello c
```

### 递归展开（C++11/14方式）

```cpp
// 递归终止条件
template<typename T>
void print_recursive(T&& t) {
    std::cout << t << std::endl;
}

// 递归模板
template<typename T, typename... Args>
void print_recursive(T&& t, Args&&... args) {
    std::cout << t << " ";
    print_recursive(args...);
}
```

### 折叠表达式（C++17方式）

```cpp
// 左折叠
template<typename... Args>
auto sum_left(Args... args) {
    return (... + args);  // ((arg1 + arg2) + arg3) + ...
}

// 右折叠
template<typename... Args>
auto sum_right(Args... args) {
    return (args + ...);  // arg1 + (arg2 + (arg3 + ...))
}

// 带初值的折叠
template<typename... Args>
auto sum_with_init(Args... args) {
    return (0 + ... + args);  // 0 + arg1 + arg2 + ...
}
```

### 完美转发与变参模板

```cpp
template<typename F, typename... Args>
auto call_function(F&& f, Args&&... args) 
    -> decltype(f(std::forward<Args>(args)...)) {
    return f(std::forward<Args>(args)...);
}

// 通用工厂函数
template<typename T, typename... Args>
std::unique_ptr<T> make_unique(Args&&... args) {
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}
```

## 7.2 模板别名

### using声明

模板别名提供了比typedef更强大和直观的类型别名机制：

```cpp
// 传统typedef的限制
template<typename T>
struct vector_typedef {
    typedef std::vector<T> type;
};

// 现代using别名
template<typename T>
using Vector = std::vector<T>;

template<typename K, typename V>
using Map = std::unordered_map<K, V>;

// 使用
Vector<int> numbers;  // 等价于 std::vector<int>
Map<std::string, int> scores;  // 等价于 std::unordered_map<std::string, int>
```

### 复杂类型简化

```cpp
// 简化复杂的嵌套类型
template<typename T>
using SharedPtr = std::shared_ptr<T>;

template<typename T>
using UniquePtr = std::unique_ptr<T>;

template<typename Sig>
using Function = std::function<Sig>;

// 函数指针别名
template<typename R, typename... Args>
using FunctionPtr = R(*)(Args...);

// 使用示例
SharedPtr<int> ptr = std::make_shared<int>(42);
Function<int(int, int)> add_func = [](int a, int b) { return a + b; };
```

### 模板模板参数别名

```cpp
template<template<typename> class Container, typename T>
using ContainerAlias = Container<T>;

// 使用
ContainerAlias<std::vector, int> vec;  // std::vector<int>
ContainerAlias<std::list, std::string> lst;  // std::list<std::string>
```

## 7.3 constexpr编程

### constexpr函数

```cpp
// 编译时计算
constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}

constexpr int fib(int n) {
    return n <= 1 ? n : fib(n - 1) + fib(n - 2);
}

// 编译时常量
constexpr int fact_5 = factorial(5);  // 在编译时计算
constexpr int fib_10 = fib(10);       // 在编译时计算
```

### constexpr if（C++17）

```cpp
template<typename T>
constexpr auto get_value(T t) {
    if constexpr (std::is_integral_v<T>) {
        return t * 2;
    } else if constexpr (std::is_floating_point_v<T>) {
        return t * 1.5;
    } else {
        return t;
    }
}
```

### constexpr容器和算法

```cpp
// C++20 constexpr容器
constexpr auto create_array() {
    std::array<int, 5> arr{};
    for (size_t i = 0; i < arr.size(); ++i) {
        arr[i] = i * i;
    }
    return arr;
}

constexpr auto squares = create_array();  // 编译时计算
```

## 7.4 SFINAE与enable_if

### SFINAE基础

SFINAE（Substitution Failure Is Not An Error）是模板元编程的重要技术：

```cpp
// 检测类型是否有特定成员函数
template<typename T>
class has_size {
private:
    template<typename U>
    static auto test(int) -> decltype(std::declval<U>().size(), std::true_type{});
    
    template<typename>
    static std::false_type test(...);
    
public:
    static constexpr bool value = decltype(test<T>(0))::value;
};

// 使用
static_assert(has_size<std::vector<int>>::value);
static_assert(!has_size<int>::value);
```

### std::enable_if

```cpp
// 只对整数类型启用
template<typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
safe_divide(T a, T b) {
    return b != 0 ? a / b : 0;
}

// 只对浮点类型启用
template<typename T>
std::enable_if_t<std::is_floating_point_v<T>, T>
safe_divide(T a, T b) {
    return b != 0.0 ? a / b : 0.0;
}
```

### 现代替代方案：if constexpr

```cpp
template<typename T>
T safe_divide(T a, T b) {
    if constexpr (std::is_integral_v<T>) {
        return b != 0 ? a / b : 0;
    } else if constexpr (std::is_floating_point_v<T>) {
        return b != 0.0 ? a / b : 0.0;
    } else {
        static_assert(std::is_arithmetic_v<T>, "T must be arithmetic");
    }
}
```

## 7.5 概念（C++20预览）

### 基本概念定义

```cpp
#include <concepts>

// 定义概念
template<typename T>
concept Integral = std::is_integral_v<T>;

template<typename T>
concept Addable = requires(T a, T b) {
    a + b;
};

template<typename T>
concept Container = requires(T t) {
    t.begin();
    t.end();
    t.size();
};
```

### 使用概念约束模板

```cpp
// 使用概念约束函数模板
template<Integral T>
T multiply_by_two(T value) {
    return value * 2;
}

// 使用requires子句
template<typename T>
requires Addable<T>
T add_values(T a, T b) {
    return a + b;
}

// 简化的语法
auto process_container(Container auto& container) {
    return container.size();
}
```

### 复杂概念

```cpp
template<typename T>
concept Comparable = requires(T a, T b) {
    { a < b } -> std::convertible_to<bool>;
    { a > b } -> std::convertible_to<bool>;
    { a == b } -> std::convertible_to<bool>;
};

template<typename T>
concept Sortable = Container<T> && 
                   Comparable<typename T::value_type>;

// 使用复杂概念
template<Sortable T>
void sort_container(T& container) {
    std::sort(container.begin(), container.end());
}
```

## 模板元编程实用技巧

### 1. 类型特征检测

```cpp
// 检测是否可调用
template<typename F, typename... Args>
using is_callable = std::is_invocable<F, Args...>;

// 检测返回类型
template<typename F, typename... Args>
using call_result = std::invoke_result_t<F, Args...>;
```

### 2. 编译时字符串处理

```cpp
template<size_t N>
struct compile_time_string {
    constexpr compile_time_string(const char (&str)[N]) {
        std::copy_n(str, N, data);
    }
    
    char data[N];
    static constexpr size_t size = N - 1;
};

// 推导指引
template<size_t N>
compile_time_string(const char (&)[N]) -> compile_time_string<N>;
```

### 3. 类型列表操作

```cpp
template<typename... Types>
struct type_list {};

// 获取第一个类型
template<typename Head, typename... Tail>
struct front<type_list<Head, Tail...>> {
    using type = Head;
};

// 获取列表大小
template<typename... Types>
constexpr size_t size_v<type_list<Types...>> = sizeof...(Types);
```

---

## 本章小结

现代C++的模板元编程更加强大和易用：
- **变参模板**：处理可变数量的参数
- **模板别名**：简化复杂类型声明
- **constexpr编程**：编译时计算
- **SFINAE/enable_if**：条件模板实例化
- **概念（C++20）**：更清晰的模板约束

下一章我们将学习异常安全与错误处理的现代方法。
