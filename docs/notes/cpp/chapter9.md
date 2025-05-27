---
title: 第九章
createTime: 2025/05/28 00:56:11
permalink: /cpp/f8wyrw7g/
---

# 第9章：现代类设计

## 9.1 Rule of Zero/Three/Five

### Rule of Zero（零规则）

最佳实践是让编译器自动生成所有特殊成员函数：

```cpp
class ModernClass {
public:
    ModernClass(const std::string& name, int value)
        : name_(name), value_(value), data_(std::make_unique<int>(value)) {}
    
    // 编译器自动生成：
    // - 析构函数
    // - 复制构造函数
    // - 复制赋值运算符
    // - 移动构造函数
    // - 移动赋值运算符
    
private:
    std::string name_;
    int value_;
    std::unique_ptr<int> data_;  // 自动管理资源
};
```

### Rule of Three（三规则）

如果需要自定义析构函数、复制构造函数或复制赋值运算符中的任何一个，通常需要定义所有三个：

```cpp
class LegacyClass {
public:
    explicit LegacyClass(size_t size) : size_(size), data_(new int[size]) {}
    
    // 1. 析构函数
    ~LegacyClass() {
        delete[] data_;
    }
    
    // 2. 复制构造函数
    LegacyClass(const LegacyClass& other) : size_(other.size_), data_(new int[size_]) {
        std::copy(other.data_, other.data_ + size_, data_);
    }
    
    // 3. 复制赋值运算符
    LegacyClass& operator=(const LegacyClass& other) {
        if (this != &other) {
            delete[] data_;
            size_ = other.size_;
            data_ = new int[size_];
            std::copy(other.data_, other.data_ + size_, data_);
        }
        return *this;
    }
    
private:
    size_t size_;
    int* data_;
};
```

### Rule of Five（五规则）

C++11引入移动语义后，通常需要定义五个特殊成员函数：

```cpp
class ResourceManager {
public:
    explicit ResourceManager(size_t size) : size_(size), data_(new int[size]) {}
    
    // 1. 析构函数
    ~ResourceManager() {
        delete[] data_;
    }
    
    // 2. 复制构造函数
    ResourceManager(const ResourceManager& other) 
        : size_(other.size_), data_(new int[size_]) {
        std::copy(other.data_, other.data_ + size_, data_);
    }
    
    // 3. 复制赋值运算符
    ResourceManager& operator=(const ResourceManager& other) {
        if (this != &other) {
            ResourceManager temp(other);
            swap(temp);
        }
        return *this;
    }
    
    // 4. 移动构造函数
    ResourceManager(ResourceManager&& other) noexcept 
        : size_(other.size_), data_(other.data_) {
        other.size_ = 0;
        other.data_ = nullptr;
    }
    
    // 5. 移动赋值运算符
    ResourceManager& operator=(ResourceManager&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            size_ = other.size_;
            data_ = other.data_;
            other.size_ = 0;
            other.data_ = nullptr;
        }
        return *this;
    }
    
    void swap(ResourceManager& other) noexcept {
        using std::swap;
        swap(size_, other.size_);
        swap(data_, other.data_);
    }
    
private:
    size_t size_;
    int* data_;
};
```

## 9.2 默认与删除函数

### 显式默认函数

```cpp
class ExplicitDefaults {
public:
    ExplicitDefaults() = default;  // 显式默认构造函数
    ~ExplicitDefaults() = default; // 显式默认析构函数
    
    ExplicitDefaults(const ExplicitDefaults&) = default;
    ExplicitDefaults& operator=(const ExplicitDefaults&) = default;
    
    ExplicitDefaults(ExplicitDefaults&&) = default;
    ExplicitDefaults& operator=(ExplicitDefaults&&) = default;
};
```

### 删除函数

```cpp
class NonCopyable {
public:
    NonCopyable() = default;
    
    // 禁止复制
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
    
    // 允许移动
    NonCopyable(NonCopyable&&) = default;
    NonCopyable& operator=(NonCopyable&&) = default;
};

class Singleton {
public:
    static Singleton& instance() {
        static Singleton instance;
        return instance;
    }
    
    // 禁止所有复制和移动
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    Singleton(Singleton&&) = delete;
    Singleton& operator=(Singleton&&) = delete;
    
private:
    Singleton() = default;
};
```

## 9.3 委托构造函数

### 基本用法

```cpp
class Point {
public:
    Point(double x, double y) : x_(x), y_(y) {
        std::cout << "Point constructed: (" << x_ << ", " << y_ << ")" << std::endl;
    }
    
    // 委托构造函数
    Point() : Point(0.0, 0.0) {}  // 委托给主构造函数
    Point(double x) : Point(x, 0.0) {}  // 委托给主构造函数
    
private:
    double x_, y_;
};
```

### 复杂示例

```cpp
class Database {
public:
    // 主构造函数
    Database(const std::string& host, int port, const std::string& username, 
             const std::string& password, bool use_ssl)
        : host_(host), port_(port), username_(username), 
          password_(password), use_ssl_(use_ssl) {
        connect();
    }
    
    // 委托构造函数
    Database(const std::string& host, int port) 
        : Database(host, port, "guest", "", false) {}
    
    Database(const std::string& host) 
        : Database(host, 3306) {}
    
    Database() 
        : Database("localhost") {}
    
private:
    void connect() {
        std::cout << "Connecting to " << host_ << ":" << port_ << std::endl;
    }
    
    std::string host_;
    int port_;
    std::string username_;
    std::string password_;
    bool use_ssl_;
};
```

## 9.4 继承构造函数

### 基本用法

```cpp
class Base {
public:
    Base(int x) : x_(x) {}
    Base(int x, int y) : x_(x), y_(y) {}
    
protected:
    int x_ = 0;
    int y_ = 0;
};

class Derived : public Base {
public:
    using Base::Base;  // 继承所有Base的构造函数
    
    // 可以添加额外的构造函数
    Derived(int x, int y, int z) : Base(x, y), z_(z) {}
    
private:
    int z_ = 0;
};

// 使用
Derived d1(42);        // 使用继承的Base(int)构造函数
Derived d2(1, 2);      // 使用继承的Base(int, int)构造函数
Derived d3(1, 2, 3);   // 使用Derived自己的构造函数
```

### 选择性继承

```cpp
class SelectiveInheritance : public Base {
public:
    // 只继承特定的构造函数
    using Base::Base;
    
    // 隐藏某个构造函数
    SelectiveInheritance(int x, int y) = delete;
};
```

## 9.5 final与override关键字

### override关键字

```cpp
class Base {
public:
    virtual void func1() {}
    virtual void func2(int x) {}
    virtual void func3() const {}
};

class Derived : public Base {
public:
    void func1() override {}  // 正确重写
    void func2(int x) override {}  // 正确重写
    void func3() const override {}  // 正确重写
    
    // void func1(int) override {}  // 编译错误：没有匹配的虚函数
    // void func3() override {}     // 编译错误：const不匹配
};
```

### final关键字

```cpp
// 禁止类被继承
class FinalClass final {
public:
    virtual void func() {}
};

// class CannotInherit : public FinalClass {};  // 编译错误

// 禁止虚函数被重写
class Base {
public:
    virtual void func1() {}
    virtual void func2() final {}  // 不能被重写
};

class Derived : public Base {
public:
    void func1() override {}  // 正确
    // void func2() override {}  // 编译错误：func2是final的
};
```

## 现代类设计最佳实践

### 1. 优先使用组合而非继承

```cpp
// 好：组合
class Logger {
public:
    void log(const std::string& message) {
        std::cout << "[LOG] " << message << std::endl;
    }
};

class Service {
public:
    Service(std::shared_ptr<Logger> logger) : logger_(logger) {}
    
    void doWork() {
        logger_->log("Service is working");
    }
    
private:
    std::shared_ptr<Logger> logger_;
};

// 避免：不必要的继承
class Service : public Logger {  // 不推荐
public:
    void doWork() {
        log("Service is working");
    }
};
```

### 2. 使用PIMPL惯用法

```cpp
// Widget.h
class Widget {
public:
    Widget();
    ~Widget();
    
    Widget(const Widget& other);
    Widget& operator=(const Widget& other);
    
    Widget(Widget&& other) noexcept;
    Widget& operator=(Widget&& other) noexcept;
    
    void doSomething();
    
private:
    class Impl;
    std::unique_ptr<Impl> pImpl_;
};

// Widget.cpp
class Widget::Impl {
public:
    void doSomething() {
        // 实现细节
    }
    
private:
    // 私有成员
};

Widget::Widget() : pImpl_(std::make_unique<Impl>()) {}
Widget::~Widget() = default;  // unique_ptr自动清理

void Widget::doSomething() {
    pImpl_->doSomething();
}
```

### 3. 接口设计原则

```cpp
// 好：小而专注的接口
class Drawable {
public:
    virtual ~Drawable() = default;
    virtual void draw() const = 0;
};

class Movable {
public:
    virtual ~Movable() = default;
    virtual void move(double dx, double dy) = 0;
};

class Shape : public Drawable, public Movable {
    // 实现两个接口
};

// 避免：庞大的接口
class HugeInterface {
public:
    virtual void draw() const = 0;
    virtual void move(double dx, double dy) = 0;
    virtual void rotate(double angle) = 0;
    virtual void scale(double factor) = 0;
    virtual void serialize() const = 0;
    virtual void deserialize() = 0;
    // ... 更多方法
};
```

---

## 本章小结

现代类设计的核心原则：
- **Rule of Zero**：优先让编译器自动生成特殊成员函数
- **显式控制**：使用default和delete明确意图
- **委托构造**：减少代码重复
- **继承构造**：简化派生类构造函数
- **override/final**：明确虚函数重写意图

下一章我们将通过实战项目综合运用所学的现代C++特性。
