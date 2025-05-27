---
title: 第八章
createTime: 2025/05/28 00:56:11
permalink: /cpp/airwtubd/
---

# 第8章：异常安全与错误处理

## 8.1 异常安全保证

### 异常安全级别

现代C++定义了三个异常安全级别：

1. **基本保证（Basic Guarantee）**：异常发生时，程序保持有效状态，无资源泄漏
2. **强保证（Strong Guarantee）**：异常发生时，程序状态回滚到操作前的状态
3. **无抛出保证（No-throw Guarantee）**：操作保证不抛出异常

```cpp
class ExceptionSafeClass {
private:
    std::vector<int> data_;
    std::string name_;
    
public:
    // 强异常安全保证
    void strong_operation(const std::vector<int>& new_data, const std::string& new_name) {
        // 创建临时副本
        std::vector<int> temp_data = new_data;
        std::string temp_name = new_name;
        
        // 只有在所有操作都成功后才修改状态
        data_ = std::move(temp_data);
        name_ = std::move(temp_name);
    }
    
    // 无抛出保证
    void no_throw_operation() noexcept {
        // 只包含不会抛出异常的操作
        data_.clear();
        name_.clear();
    }
};
```

### copy-and-swap惯用法

```cpp
class Resource {
public:
    Resource& operator=(const Resource& other) {
        if (this != &other) {
            Resource temp(other);  // 可能抛出异常
            swap(temp);            // 无抛出操作
        }
        return *this;
    }
    
    void swap(Resource& other) noexcept {
        using std::swap;
        swap(data_, other.data_);
        swap(size_, other.size_);
    }
    
private:
    std::unique_ptr<int[]> data_;
    size_t size_;
};
```

## 8.2 RAII与异常

### 自动资源管理

RAII确保即使发生异常，资源也能正确释放：

```cpp
class FileHandler {
public:
    explicit FileHandler(const std::string& filename) 
        : file_(fopen(filename.c_str(), "r")) {
        if (!file_) {
            throw std::runtime_error("Failed to open file: " + filename);
        }
    }
    
    ~FileHandler() {
        if (file_) {
            fclose(file_);
        }
    }
    
    // 禁止复制，允许移动
    FileHandler(const FileHandler&) = delete;
    FileHandler& operator=(const FileHandler&) = delete;
    
    FileHandler(FileHandler&& other) noexcept : file_(other.file_) {
        other.file_ = nullptr;
    }
    
    FileHandler& operator=(FileHandler&& other) noexcept {
        if (this != &other) {
            if (file_) fclose(file_);
            file_ = other.file_;
            other.file_ = nullptr;
        }
        return *this;
    }
    
    FILE* get() const { return file_; }
    
private:
    FILE* file_;
};
```

### 作用域守卫

```cpp
template<typename F>
class scope_guard {
public:
    explicit scope_guard(F&& f) : f_(std::forward<F>(f)), active_(true) {}
    
    ~scope_guard() {
        if (active_) f_();
    }
    
    void dismiss() { active_ = false; }
    
    scope_guard(const scope_guard&) = delete;
    scope_guard& operator=(const scope_guard&) = delete;
    
    scope_guard(scope_guard&& other) noexcept 
        : f_(std::move(other.f_)), active_(other.active_) {
        other.active_ = false;
    }
    
private:
    F f_;
    bool active_;
};

template<typename F>
auto make_scope_guard(F&& f) {
    return scope_guard<F>(std::forward<F>(f));
}

// 使用示例
void risky_operation() {
    int* ptr = new int(42);
    auto guard = make_scope_guard([ptr] { delete ptr; });
    
    // 即使这里抛出异常，ptr也会被正确删除
    might_throw_exception();
    
    guard.dismiss();  // 如果成功，取消清理
}
```

## 8.3 noexcept规范

### noexcept的重要性

```cpp
class MoveOptimized {
public:
    // 移动操作应该标记为noexcept
    MoveOptimized(MoveOptimized&& other) noexcept 
        : data_(std::move(other.data_)) {}
    
    MoveOptimized& operator=(MoveOptimized&& other) noexcept {
        if (this != &other) {
            data_ = std::move(other.data_);
        }
        return *this;
    }
    
    // 析构函数隐式noexcept
    ~MoveOptimized() = default;
    
    // 交换操作应该是noexcept
    void swap(MoveOptimized& other) noexcept {
        using std::swap;
        swap(data_, other.data_);
    }
    
private:
    std::vector<int> data_;
};
```

### 条件noexcept

```cpp
template<typename T>
class Container {
public:
    void swap(Container& other) noexcept(std::is_nothrow_swappable_v<T>) {
        using std::swap;
        swap(data_, other.data_);
    }
    
    Container(Container&& other) noexcept(std::is_nothrow_move_constructible_v<T>)
        : data_(std::move(other.data_)) {}
    
private:
    T data_;
};
```

## 8.4 std::optional (C++17)

### 基本用法

`std::optional`提供了一种表示"可能有值"的类型安全方式：

```cpp
#include <optional>

std::optional<int> safe_divide(int a, int b) {
    if (b == 0) {
        return std::nullopt;  // 没有值
    }
    return a / b;  // 有值
}

// 使用
auto result = safe_divide(10, 3);
if (result) {
    std::cout << "结果: " << *result << std::endl;
} else {
    std::cout << "除零错误" << std::endl;
}

// 或者使用value_or
std::cout << "结果: " << result.value_or(0) << std::endl;
```

### 链式操作

```cpp
std::optional<std::string> get_user_name(int user_id);
std::optional<std::string> get_user_email(const std::string& name);

// 链式操作
auto email = get_user_name(123)
    .and_then([](const std::string& name) {
        return get_user_email(name);
    })
    .value_or("unknown@example.com");
```

### 与异常的比较

```cpp
// 使用异常
int risky_divide(int a, int b) {
    if (b == 0) {
        throw std::invalid_argument("Division by zero");
    }
    return a / b;
}

// 使用optional
std::optional<int> safe_divide(int a, int b) {
    if (b == 0) {
        return std::nullopt;
    }
    return a / b;
}

// 性能比较：optional通常更快，因为没有栈展开开销
```

## 8.5 错误处理最佳实践

### 1. 选择合适的错误处理机制

| 情况 | 推荐方案 |
|------|----------|
| 预期的错误 | `std::optional`, `std::expected` (C++23) |
| 编程错误 | `assert`, 异常 |
| 系统错误 | 异常 |
| 性能关键路径 | 错误码, `std::optional` |

### 2. 异常类型设计

```cpp
// 自定义异常层次
class AppException : public std::exception {
public:
    explicit AppException(const std::string& message) : message_(message) {}
    const char* what() const noexcept override { return message_.c_str(); }
    
private:
    std::string message_;
};

class NetworkException : public AppException {
public:
    NetworkException(const std::string& message, int error_code)
        : AppException(message), error_code_(error_code) {}
    
    int error_code() const { return error_code_; }
    
private:
    int error_code_;
};

class FileException : public AppException {
public:
    FileException(const std::string& message, const std::string& filename)
        : AppException(message), filename_(filename) {}
    
    const std::string& filename() const { return filename_; }
    
private:
    std::string filename_;
};
```

### 3. 错误传播

```cpp
// 使用expected-like类型 (C++23预览)
template<typename T, typename E>
class expected {
public:
    // 成功情况
    expected(T value) : has_value_(true) {
        new(&storage_.value) T(std::move(value));
    }
    
    // 错误情况
    expected(E error) : has_value_(false) {
        new(&storage_.error) E(std::move(error));
    }
    
    bool has_value() const { return has_value_; }
    
    T& value() { return storage_.value; }
    const T& value() const { return storage_.value; }
    
    E& error() { return storage_.error; }
    const E& error() const { return storage_.error; }
    
private:
    bool has_value_;
    union Storage {
        T value;
        E error;
        Storage() {}
        ~Storage() {}
    } storage_;
};
```

### 4. 异常安全的容器操作

```cpp
template<typename T>
class SafeVector {
public:
    void push_back(const T& value) {
        if (size_ == capacity_) {
            grow();
        }
        
        try {
            new(data_ + size_) T(value);
            ++size_;
        } catch (...) {
            // 构造失败，不增加size_
            throw;
        }
    }
    
private:
    void grow() {
        size_t new_capacity = capacity_ == 0 ? 1 : capacity_ * 2;
        T* new_data = static_cast<T*>(std::malloc(new_capacity * sizeof(T)));
        
        if (!new_data) {
            throw std::bad_alloc();
        }
        
        // 移动现有元素
        size_t constructed = 0;
        try {
            for (size_t i = 0; i < size_; ++i) {
                new(new_data + i) T(std::move(data_[i]));
                ++constructed;
            }
        } catch (...) {
            // 清理已构造的元素
            for (size_t i = 0; i < constructed; ++i) {
                new_data[i].~T();
            }
            std::free(new_data);
            throw;
        }
        
        // 销毁旧元素
        for (size_t i = 0; i < size_; ++i) {
            data_[i].~T();
        }
        
        std::free(data_);
        data_ = new_data;
        capacity_ = new_capacity;
    }
    
    T* data_ = nullptr;
    size_t size_ = 0;
    size_t capacity_ = 0;
};
```

---

## 本章小结

现代C++的异常安全与错误处理：
- **异常安全保证**：基本、强、无抛出三个级别
- **RAII**：自动资源管理，异常安全的基础
- **noexcept**：性能优化和接口契约
- **std::optional**：类型安全的"可能无值"表示
- **最佳实践**：选择合适的错误处理机制

下一章我们将学习现代类设计的原则和技巧。
