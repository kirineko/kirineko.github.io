---
title: 第三章
createTime: 2025/05/28 00:56:11
permalink: /cpp/6ijocev1/
---

# 第3章：移动语义与完美转发

## 3.1 左值与右值

### 基本概念

理解移动语义首先要理解左值和右值：

- **左值 (lvalue)**：有名字的对象，可以取地址
- **右值 (rvalue)**：临时对象，不能取地址

```cpp
int x = 42;        // x是左值
int y = x + 1;     // x是左值，x+1是右值
int z = std::move(x); // std::move(x)是右值
```

### 右值引用

C++11引入右值引用 `&&`：

```cpp
void func(int& lref) {          // 左值引用
    std::cout << "左值引用" << std::endl;
}

void func(int&& rref) {         // 右值引用
    std::cout << "右值引用" << std::endl;
}

int x = 42;
func(x);           // 调用左值引用版本
func(42);          // 调用右值引用版本
func(std::move(x)); // 调用右值引用版本
```

## 3.2 移动构造函数与移动赋值

### 为什么需要移动语义？

传统的复制操作可能很昂贵：

```cpp
class BigData {
private:
    std::vector<int> data_;
public:
    // 复制构造函数 - 昂贵的深拷贝
    BigData(const BigData& other) : data_(other.data_) {
        std::cout << "复制构造，大小: " << data_.size() << std::endl;
    }
    
    // 移动构造函数 - 廉价的资源转移
    BigData(BigData&& other) noexcept : data_(std::move(other.data_)) {
        std::cout << "移动构造，大小: " << data_.size() << std::endl;
    }
};
```

### 移动语义的五大法则

现代C++的"五大法则"（Rule of Five）：

```cpp
class Resource {
public:
    // 1. 析构函数
    ~Resource() { cleanup(); }
    
    // 2. 复制构造函数
    Resource(const Resource& other) { copy_from(other); }
    
    // 3. 复制赋值运算符
    Resource& operator=(const Resource& other) {
        if (this != &other) {
            cleanup();
            copy_from(other);
        }
        return *this;
    }
    
    // 4. 移动构造函数
    Resource(Resource&& other) noexcept { move_from(std::move(other)); }
    
    // 5. 移动赋值运算符
    Resource& operator=(Resource&& other) noexcept {
        if (this != &other) {
            cleanup();
            move_from(std::move(other));
        }
        return *this;
    }
};
```

### noexcept的重要性

移动操作应该标记为`noexcept`：

```cpp
class SafeMove {
public:
    SafeMove(SafeMove&& other) noexcept {
        // 移动操作不应该抛出异常
    }
    
    SafeMove& operator=(SafeMove&& other) noexcept {
        // 移动赋值也不应该抛出异常
        return *this;
    }
};
```

## 3.3 std::move与std::forward

### std::move

`std::move`将左值转换为右值引用：

```cpp
std::string str = "Hello";
std::string moved = std::move(str);
// str现在处于有效但未指定的状态
```

### std::move的实现原理

```cpp
template<typename T>
constexpr std::remove_reference_t<T>&& move(T&& t) noexcept {
    return static_cast<std::remove_reference_t<T>&&>(t);
}
```

### 何时使用std::move

```cpp
class Container {
    std::vector<std::string> data_;
public:
    void addItem(std::string item) {
        data_.push_back(std::move(item)); // 避免不必要的复制
    }
    
    std::vector<std::string> getData() && {
        return std::move(data_); // 从临时对象移动
    }
    
    const std::vector<std::string>& getData() const & {
        return data_; // 从左值返回引用
    }
};
```

## 3.4 完美转发

### 问题背景

如何编写一个函数，能够完美地转发参数？

```cpp
template<typename T>
void wrapper(T&& arg) {
    func(std::forward<T>(arg)); // 完美转发
}
```

### 万能引用

`T&&`在模板中是万能引用，不是右值引用：

```cpp
template<typename T>
void func(T&& param) {
    // T&&是万能引用，可以绑定左值或右值
}

int x = 42;
func(x);    // T推导为int&，param类型为int&
func(42);   // T推导为int，param类型为int&&
```

### std::forward的作用

`std::forward`保持参数的值类别：

```cpp
template<typename T>
void wrapper(T&& arg) {
    // 如果arg是左值，转发为左值
    // 如果arg是右值，转发为右值
    target_func(std::forward<T>(arg));
}
```

### 完美转发的应用

```cpp
// 工厂函数的完美转发
template<typename T, typename... Args>
std::unique_ptr<T> make_unique(Args&&... args) {
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}

// 使用示例
auto ptr = make_unique<std::string>(10, 'A'); // 完美转发构造参数
```

## 3.5 移动语义的性能优化

### 容器操作优化

```cpp
std::vector<std::string> createLargeVector() {
    std::vector<std::string> result;
    result.reserve(1000000);
    
    for (int i = 0; i < 1000000; ++i) {
        result.emplace_back("Item " + std::to_string(i)); // 就地构造
    }
    
    return result; // 返回值优化 + 移动语义
}

// 使用
auto vec = createLargeVector(); // 高效，没有不必要的复制
```

### 移动优先的设计

```cpp
class MoveOptimized {
    std::vector<int> data_;
    std::string name_;
    
public:
    // 构造函数接受右值引用
    MoveOptimized(std::vector<int> data, std::string name)
        : data_(std::move(data)), name_(std::move(name)) {}
    
    // 设置器使用移动语义
    void setData(std::vector<int> new_data) {
        data_ = std::move(new_data);
    }
    
    void setName(std::string new_name) {
        name_ = std::move(new_name);
    }
};
```

### 性能测量

```cpp
#include <chrono>

void measurePerformance() {
    const size_t size = 1000000;
    std::vector<std::string> source(size, "Long string for testing");
    
    // 测量复制性能
    auto start = std::chrono::high_resolution_clock::now();
    auto copied = source; // 复制
    auto end = std::chrono::high_resolution_clock::now();
    auto copy_time = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
    
    // 测量移动性能
    start = std::chrono::high_resolution_clock::now();
    auto moved = std::move(source); // 移动
    end = std::chrono::high_resolution_clock::now();
    auto move_time = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
    
    std::cout << "复制时间: " << copy_time.count() << "ms" << std::endl;
    std::cout << "移动时间: " << move_time.count() << "ms" << std::endl;
}
```

## 移动语义最佳实践

### 1. 总是标记移动操作为noexcept
```cpp
MyClass(MyClass&& other) noexcept;
MyClass& operator=(MyClass&& other) noexcept;
```

### 2. 实现移动语义时遵循"零规则"
```cpp
class Modern {
    std::vector<int> data_;
    std::unique_ptr<int> ptr_;
    // 编译器自动生成移动操作
};
```

### 3. 在函数参数中合理使用移动
```cpp
void process(std::string data) {  // 按值传递
    storage_ = std::move(data);   // 移动到成员变量
}
```

### 4. 返回值优化与移动语义结合
```cpp
std::vector<int> createVector() {
    std::vector<int> result;
    // ... 填充数据
    return result; // RVO + 移动语义
}
```

---

## 本章小结

移动语义是现代C++性能优化的核心：
- **右值引用**：绑定临时对象
- **移动构造/赋值**：转移资源而非复制
- **std::move**：强制移动语义
- **完美转发**：保持参数的值类别
- **性能提升**：避免不必要的复制操作

下一章我们将学习Lambda表达式，这是现代C++函数式编程的重要特性。
