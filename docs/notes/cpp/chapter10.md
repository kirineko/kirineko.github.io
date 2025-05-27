---
title: 第十章
createTime: 2025/05/28 00:56:11
permalink: /cpp/wev5gj5f/
---

# 第10章：实战项目

## 10.1 现代C++设计模式

### 单例模式（现代实现）

```cpp
class ModernSingleton {
public:
    static ModernSingleton& instance() {
        static ModernSingleton instance;  // C++11保证线程安全
        return instance;
    }
    
    // 禁止复制和移动
    ModernSingleton(const ModernSingleton&) = delete;
    ModernSingleton& operator=(const ModernSingleton&) = delete;
    ModernSingleton(ModernSingleton&&) = delete;
    ModernSingleton& operator=(ModernSingleton&&) = delete;
    
    void doWork() {
        std::cout << "Singleton working..." << std::endl;
    }
    
private:
    ModernSingleton() = default;
    ~ModernSingleton() = default;
};
```

### 工厂模式（使用智能指针）

```cpp
class Product {
public:
    virtual ~Product() = default;
    virtual void use() = 0;
};

class ConcreteProductA : public Product {
public:
    void use() override {
        std::cout << "Using Product A" << std::endl;
    }
};

class ConcreteProductB : public Product {
public:
    void use() override {
        std::cout << "Using Product B" << std::endl;
    }
};

class ModernFactory {
public:
    enum class ProductType { A, B };
    
    static std::unique_ptr<Product> createProduct(ProductType type) {
        switch (type) {
            case ProductType::A:
                return std::make_unique<ConcreteProductA>();
            case ProductType::B:
                return std::make_unique<ConcreteProductB>();
            default:
                return nullptr;
        }
    }
};
```

### 观察者模式（使用Lambda和weak_ptr）

```cpp
template<typename... Args>
class Signal {
public:
    using Slot = std::function<void(Args...)>;
    using Connection = std::weak_ptr<void>;
    
    Connection connect(Slot slot) {
        auto connection = std::make_shared<SlotWrapper>(std::move(slot));
        slots_.push_back(connection);
        return connection;
    }
    
    void emit(Args... args) {
        // 清理过期的连接
        slots_.erase(
            std::remove_if(slots_.begin(), slots_.end(),
                [](const auto& weak_slot) { return weak_slot.expired(); }),
            slots_.end()
        );
        
        // 调用所有有效的槽函数
        for (const auto& weak_slot : slots_) {
            if (auto slot = weak_slot.lock()) {
                slot->call(args...);
            }
        }
    }
    
private:
    struct SlotWrapper {
        Slot slot;
        SlotWrapper(Slot s) : slot(std::move(s)) {}
        void call(Args... args) { slot(args...); }
    };
    
    std::vector<std::weak_ptr<SlotWrapper>> slots_;
};
```

## 10.2 性能优化技巧

### 移动语义优化

```cpp
class PerformanceOptimized {
public:
    // 返回值优化 + 移动语义
    static std::vector<std::string> createLargeVector() {
        std::vector<std::string> result;
        result.reserve(1000000);
        
        for (int i = 0; i < 1000000; ++i) {
            result.emplace_back("Item " + std::to_string(i));
        }
        
        return result;  // RVO + 移动语义
    }
    
    // 完美转发
    template<typename T>
    void addItem(T&& item) {
        items_.emplace_back(std::forward<T>(item));
    }
    
private:
    std::vector<std::string> items_;
};
```

### 内存池分配器

```cpp
template<typename T, size_t PoolSize = 1024>
class PoolAllocator {
public:
    using value_type = T;
    
    PoolAllocator() : pool_(std::make_unique<char[]>(PoolSize * sizeof(T))), 
                      current_(pool_.get()) {}
    
    T* allocate(size_t n) {
        if (current_ + n * sizeof(T) > pool_.get() + PoolSize * sizeof(T)) {
            throw std::bad_alloc();
        }
        
        T* result = reinterpret_cast<T*>(current_);
        current_ += n * sizeof(T);
        return result;
    }
    
    void deallocate(T* ptr, size_t n) {
        // 简单的池分配器不支持单独释放
    }
    
private:
    std::unique_ptr<char[]> pool_;
    char* current_;
};
```

### 缓存友好的数据结构

```cpp
// 结构体数组 vs 数组结构体
class CacheFriendlyParticles {
public:
    void addParticle(float x, float y, float z, float vx, float vy, float vz) {
        positions_x_.push_back(x);
        positions_y_.push_back(y);
        positions_z_.push_back(z);
        velocities_x_.push_back(vx);
        velocities_y_.push_back(vy);
        velocities_z_.push_back(vz);
    }
    
    void updatePositions(float dt) {
        // 缓存友好的更新
        for (size_t i = 0; i < positions_x_.size(); ++i) {
            positions_x_[i] += velocities_x_[i] * dt;
            positions_y_[i] += velocities_y_[i] * dt;
            positions_z_[i] += velocities_z_[i] * dt;
        }
    }
    
private:
    std::vector<float> positions_x_, positions_y_, positions_z_;
    std::vector<float> velocities_x_, velocities_y_, velocities_z_;
};
```

## 10.3 代码组织与项目结构

### 现代项目结构

```
modern_cpp_project/
├── CMakeLists.txt
├── README.md
├── include/
│   └── myproject/
│       ├── core/
│       │   ├── engine.hpp
│       │   └── config.hpp
│       └── utils/
│           ├── logger.hpp
│           └── timer.hpp
├── src/
│   ├── core/
│   │   ├── engine.cpp
│   │   └── config.cpp
│   └── utils/
│       ├── logger.cpp
│       └── timer.cpp
├── tests/
│   ├── unit/
│   │   ├── test_engine.cpp
│   │   └── test_logger.cpp
│   └── integration/
│       └── test_full_system.cpp
├── examples/
│   └── basic_usage.cpp
└── third_party/
    └── (external dependencies)
```

### 头文件组织

```cpp
// include/myproject/core/engine.hpp
#pragma once

#include <memory>
#include <string>
#include <vector>

namespace myproject::core {

class Engine {
public:
    Engine();
    ~Engine();
    
    // 禁止复制，允许移动
    Engine(const Engine&) = delete;
    Engine& operator=(const Engine&) = delete;
    Engine(Engine&&) noexcept;
    Engine& operator=(Engine&&) noexcept;
    
    void initialize(const std::string& config_file);
    void run();
    void shutdown();
    
private:
    class Impl;
    std::unique_ptr<Impl> pImpl_;
};

} // namespace myproject::core
```

### 模块化设计

```cpp
// 接口定义
namespace myproject::interfaces {

class ILogger {
public:
    virtual ~ILogger() = default;
    virtual void log(const std::string& message) = 0;
};

class IRenderer {
public:
    virtual ~IRenderer() = default;
    virtual void render() = 0;
};

} // namespace myproject::interfaces

// 依赖注入容器
namespace myproject::di {

class Container {
public:
    template<typename Interface, typename Implementation, typename... Args>
    void registerSingleton(Args&&... args) {
        auto instance = std::make_shared<Implementation>(std::forward<Args>(args)...);
        services_[typeid(Interface)] = instance;
    }
    
    template<typename Interface>
    std::shared_ptr<Interface> resolve() {
        auto it = services_.find(typeid(Interface));
        if (it != services_.end()) {
            return std::static_pointer_cast<Interface>(it->second);
        }
        return nullptr;
    }
    
private:
    std::unordered_map<std::type_index, std::shared_ptr<void>> services_;
};

} // namespace myproject::di
```

## 10.4 单元测试与现代工具链

### 使用Catch2进行单元测试

```cpp
#define CATCH_CONFIG_MAIN
#include <catch2/catch.hpp>

#include "myproject/utils/math.hpp"

TEST_CASE("Math utilities", "[math]") {
    using namespace myproject::utils;
    
    SECTION("Addition") {
        REQUIRE(add(2, 3) == 5);
        REQUIRE(add(-1, 1) == 0);
    }
    
    SECTION("Division") {
        REQUIRE(divide(10, 2) == 5.0);
        REQUIRE_THROWS_AS(divide(10, 0), std::invalid_argument);
    }
}

TEST_CASE("Vector operations", "[vector]") {
    std::vector<int> vec{1, 2, 3, 4, 5};
    
    SECTION("Size") {
        REQUIRE(vec.size() == 5);
    }
    
    SECTION("Sum") {
        auto sum = std::accumulate(vec.begin(), vec.end(), 0);
        REQUIRE(sum == 15);
    }
}
```

### CMake现代配置

```cmake
cmake_minimum_required(VERSION 3.15)
project(ModernCppProject VERSION 1.0.0 LANGUAGES CXX)

# 设置C++标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 编译选项
if(MSVC)
    add_compile_options(/W4 /WX)
else()
    add_compile_options(-Wall -Wextra -Wpedantic -Werror)
endif()

# 查找依赖
find_package(Threads REQUIRED)

# 主库
add_library(myproject_core
    src/core/engine.cpp
    src/core/config.cpp
    src/utils/logger.cpp
    src/utils/timer.cpp
)

target_include_directories(myproject_core
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
)

target_link_libraries(myproject_core
    PUBLIC
        Threads::Threads
)

# 可执行文件
add_executable(myproject_app
    src/main.cpp
)

target_link_libraries(myproject_app
    PRIVATE
        myproject_core
)

# 测试
if(BUILD_TESTING)
    enable_testing()
    
    find_package(Catch2 REQUIRED)
    
    add_executable(tests
        tests/unit/test_engine.cpp
        tests/unit/test_logger.cpp
    )
    
    target_link_libraries(tests
        PRIVATE
            myproject_core
            Catch2::Catch2
    )
    
    include(CTest)
    include(Catch)
    catch_discover_tests(tests)
endif()
```

## 10.5 综合项目实战

### 项目：现代C++任务调度器

```cpp
// task_scheduler.hpp
#pragma once

#include <functional>
#include <future>
#include <queue>
#include <thread>
#include <vector>
#include <mutex>
#include <condition_variable>
#include <atomic>

namespace scheduler {

class TaskScheduler {
public:
    explicit TaskScheduler(size_t num_threads = std::thread::hardware_concurrency());
    ~TaskScheduler();
    
    // 禁止复制和移动
    TaskScheduler(const TaskScheduler&) = delete;
    TaskScheduler& operator=(const TaskScheduler&) = delete;
    TaskScheduler(TaskScheduler&&) = delete;
    TaskScheduler& operator=(TaskScheduler&&) = delete;
    
    // 提交任务
    template<typename F, typename... Args>
    auto submit(F&& f, Args&&... args) -> std::future<std::invoke_result_t<F, Args...>>;
    
    // 停止调度器
    void shutdown();
    
    // 获取状态
    size_t pending_tasks() const;
    size_t active_threads() const;
    
private:
    void worker_thread();
    
    std::vector<std::thread> workers_;
    std::queue<std::function<void()>> tasks_;
    
    mutable std::mutex queue_mutex_;
    std::condition_variable condition_;
    std::atomic<bool> stop_{false};
};

template<typename F, typename... Args>
auto TaskScheduler::submit(F&& f, Args&&... args) -> std::future<std::invoke_result_t<F, Args...>> {
    using return_type = std::invoke_result_t<F, Args...>;
    
    auto task = std::make_shared<std::packaged_task<return_type()>>(
        std::bind(std::forward<F>(f), std::forward<Args>(args)...)
    );
    
    std::future<return_type> result = task->get_future();
    
    {
        std::unique_lock<std::mutex> lock(queue_mutex_);
        
        if (stop_) {
            throw std::runtime_error("Cannot submit task to stopped scheduler");
        }
        
        tasks_.emplace([task] { (*task)(); });
    }
    
    condition_.notify_one();
    return result;
}

} // namespace scheduler
```

这个综合项目展示了现代C++的多个特性：
- **智能指针**：自动内存管理
- **移动语义**：高效的资源传递
- **变参模板**：灵活的任务提交接口
- **Lambda表达式**：简洁的任务定义
- **并发编程**：线程池和任务队列
- **异常安全**：RAII和异常处理
- **现代类设计**：禁用复制、显式接口

---

## 本章小结

通过实战项目，我们综合运用了现代C++的各种特性：
- **设计模式**：现代化的实现方式
- **性能优化**：移动语义、内存池、缓存友好
- **项目组织**：模块化、依赖注入
- **测试工具**：现代测试框架
- **构建系统**：CMake最佳实践

现代C++不仅提供了强大的语言特性，更重要的是提供了一套完整的开发理念和最佳实践。
