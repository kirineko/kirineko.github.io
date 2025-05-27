---
title: 第六章
createTime: 2025/05/28 00:56:11
permalink: /cpp/jvm5qijb/
---

# 第6章：并发编程

## 6.1 std::thread基础

### 创建和管理线程

C++11引入了标准的线程库，提供了跨平台的并发编程支持：

```cpp
#include <thread>
#include <iostream>

// 线程函数
void worker_function(int id) {
    std::cout << "Worker " << id << " is running" << std::endl;
}

// 创建线程
std::thread t1(worker_function, 1);
std::thread t2([]() {
    std::cout << "Lambda thread is running" << std::endl;
});

// 等待线程完成
t1.join();
t2.join();
```

### 线程的生命周期管理

```cpp
class ThreadManager {
public:
    void start() {
        worker_thread_ = std::thread(&ThreadManager::worker, this);
    }
    
    void stop() {
        if (worker_thread_.joinable()) {
            worker_thread_.join();
        }
    }
    
private:
    void worker() {
        // 工作线程逻辑
    }
    
    std::thread worker_thread_;
};
```

### 线程异常安全

```cpp
void safe_thread_usage() {
    std::thread t([]() {
        // 线程工作
    });
    
    try {
        // 可能抛出异常的代码
        risky_operation();
    } catch (...) {
        if (t.joinable()) {
            t.join();
        }
        throw;  // 重新抛出异常
    }
    
    t.join();
}
```

## 6.2 互斥量与锁

### std::mutex基础

```cpp
#include <mutex>

class Counter {
public:
    void increment() {
        std::lock_guard<std::mutex> lock(mutex_);
        ++count_;
    }
    
    int get() const {
        std::lock_guard<std::mutex> lock(mutex_);
        return count_;
    }
    
private:
    mutable std::mutex mutex_;
    int count_ = 0;
};
```

### 不同类型的锁

```cpp
// 1. lock_guard - RAII锁，不可手动解锁
{
    std::lock_guard<std::mutex> lock(mutex);
    // 临界区代码
}  // 自动解锁

// 2. unique_lock - 更灵活的锁
{
    std::unique_lock<std::mutex> lock(mutex);
    // 可以手动解锁
    lock.unlock();
    // 非临界区代码
    lock.lock();
    // 再次进入临界区
}

// 3. shared_lock - 读写锁的读锁 (C++14)
std::shared_mutex rw_mutex;
{
    std::shared_lock<std::shared_mutex> read_lock(rw_mutex);
    // 多个线程可以同时持有读锁
}
{
    std::unique_lock<std::shared_mutex> write_lock(rw_mutex);
    // 只有一个线程可以持有写锁
}
```

### 避免死锁

```cpp
// 错误：可能导致死锁
void bad_example(std::mutex& m1, std::mutex& m2) {
    std::lock_guard<std::mutex> lock1(m1);
    std::lock_guard<std::mutex> lock2(m2);  // 可能死锁
}

// 正确：使用std::lock避免死锁
void good_example(std::mutex& m1, std::mutex& m2) {
    std::lock(m1, m2);  // 原子地锁定多个互斥量
    std::lock_guard<std::mutex> lock1(m1, std::adopt_lock);
    std::lock_guard<std::mutex> lock2(m2, std::adopt_lock);
}

// 更好：使用scoped_lock (C++17)
void best_example(std::mutex& m1, std::mutex& m2) {
    std::scoped_lock lock(m1, m2);  // 自动避免死锁
}
```

## 6.3 条件变量

### 基本用法

```cpp
#include <condition_variable>
#include <queue>

class ThreadSafeQueue {
public:
    void push(int item) {
        std::unique_lock<std::mutex> lock(mutex_);
        queue_.push(item);
        condition_.notify_one();
    }
    
    int pop() {
        std::unique_lock<std::mutex> lock(mutex_);
        condition_.wait(lock, [this] { return !queue_.empty(); });
        
        int item = queue_.front();
        queue_.pop();
        return item;
    }
    
private:
    std::queue<int> queue_;
    std::mutex mutex_;
    std::condition_variable condition_;
};
```

### 生产者-消费者模式

```cpp
class ProducerConsumer {
public:
    void producer() {
        for (int i = 0; i < 10; ++i) {
            {
                std::unique_lock<std::mutex> lock(mutex_);
                buffer_.push(i);
                std::cout << "Produced: " << i << std::endl;
            }
            cv_.notify_one();
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        }
    }
    
    void consumer() {
        while (true) {
            std::unique_lock<std::mutex> lock(mutex_);
            cv_.wait(lock, [this] { return !buffer_.empty() || finished_; });
            
            if (!buffer_.empty()) {
                int item = buffer_.front();
                buffer_.pop();
                std::cout << "Consumed: " << item << std::endl;
            } else if (finished_) {
                break;
            }
        }
    }
    
    void finish() {
        {
            std::lock_guard<std::mutex> lock(mutex_);
            finished_ = true;
        }
        cv_.notify_all();
    }
    
private:
    std::queue<int> buffer_;
    std::mutex mutex_;
    std::condition_variable cv_;
    bool finished_ = false;
};
```

## 6.4 原子操作

### std::atomic基础

```cpp
#include <atomic>

class AtomicCounter {
public:
    void increment() {
        count_.fetch_add(1);  // 原子递增
    }
    
    void decrement() {
        count_.fetch_sub(1);  // 原子递减
    }
    
    int get() const {
        return count_.load();  // 原子读取
    }
    
    void set(int value) {
        count_.store(value);  // 原子写入
    }
    
private:
    std::atomic<int> count_{0};
};
```

### 内存序

```cpp
std::atomic<bool> ready{false};
std::atomic<int> data{0};

// 写线程
void writer() {
    data.store(42, std::memory_order_relaxed);
    ready.store(true, std::memory_order_release);  // 释放语义
}

// 读线程
void reader() {
    while (!ready.load(std::memory_order_acquire)) {  // 获取语义
        // 等待
    }
    int value = data.load(std::memory_order_relaxed);
    std::cout << "Read value: " << value << std::endl;
}
```

### 无锁数据结构

```cpp
template<typename T>
class LockFreeStack {
public:
    void push(T item) {
        Node* new_node = new Node{std::move(item), head_.load()};
        while (!head_.compare_exchange_weak(new_node->next, new_node)) {
            // CAS失败，重试
        }
    }
    
    bool pop(T& result) {
        Node* old_head = head_.load();
        while (old_head && !head_.compare_exchange_weak(old_head, old_head->next)) {
            // CAS失败，重试
        }
        
        if (old_head) {
            result = std::move(old_head->data);
            delete old_head;
            return true;
        }
        return false;
    }
    
private:
    struct Node {
        T data;
        Node* next;
    };
    
    std::atomic<Node*> head_{nullptr};
};
```

## 6.5 异步编程：future与promise

### std::async

```cpp
#include <future>

// 异步执行函数
int compute_something(int x) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return x * x;
}

// 使用async
auto future = std::async(std::launch::async, compute_something, 42);
// 继续执行其他工作...
int result = future.get();  // 获取结果，如果还没完成会阻塞
```

### std::promise和std::future

```cpp
void promise_example() {
    std::promise<int> promise;
    std::future<int> future = promise.get_future();
    
    // 在另一个线程中设置值
    std::thread t([&promise]() {
        std::this_thread::sleep_for(std::chrono::seconds(1));
        promise.set_value(42);
    });
    
    // 在主线程中获取值
    int result = future.get();
    std::cout << "Result: " << result << std::endl;
    
    t.join();
}
```

### std::packaged_task

```cpp
void packaged_task_example() {
    std::packaged_task<int(int)> task(compute_something);
    std::future<int> future = task.get_future();
    
    std::thread t(std::move(task), 42);
    
    int result = future.get();
    std::cout << "Result: " << result << std::endl;
    
    t.join();
}
```

### 异常处理

```cpp
void exception_handling_example() {
    auto future = std::async(std::launch::async, []() -> int {
        throw std::runtime_error("Something went wrong");
        return 42;
    });
    
    try {
        int result = future.get();
    } catch (const std::exception& e) {
        std::cout << "Caught exception: " << e.what() << std::endl;
    }
}
```

## 并发编程最佳实践

### 1. 优先使用高级抽象

```cpp
// 好：使用async
auto future = std::async(std::launch::async, compute_task);

// 避免：手动管理线程
std::thread t(compute_task);
t.join();
```

### 2. 避免共享可变状态

```cpp
// 好：不可变数据 + 消息传递
class ImmutableData {
    const std::vector<int> data_;
public:
    ImmutableData(std::vector<int> data) : data_(std::move(data)) {}
    const std::vector<int>& get() const { return data_; }
};

// 避免：共享可变状态
class MutableSharedData {
    std::vector<int> data_;
    std::mutex mutex_;
    // 复杂的同步逻辑...
};
```

### 3. 使用RAII管理锁

```cpp
// 好：RAII锁
void safe_function() {
    std::lock_guard<std::mutex> lock(mutex_);
    // 临界区代码
}  // 自动解锁

// 避免：手动锁管理
void unsafe_function() {
    mutex_.lock();
    // 临界区代码
    mutex_.unlock();  // 可能忘记或异常时不执行
}
```

### 4. 选择合适的同步原语

| 场景 | 推荐方案 |
|------|----------|
| 简单计数器 | `std::atomic<int>` |
| 保护临界区 | `std::mutex` + `std::lock_guard` |
| 读多写少 | `std::shared_mutex` |
| 生产者-消费者 | `std::condition_variable` |
| 异步计算 | `std::async` |

---

## 本章小结

现代C++提供了丰富的并发编程工具：
- **std::thread**：标准线程库
- **互斥量和锁**：保护共享资源
- **条件变量**：线程间通信
- **原子操作**：无锁编程
- **异步编程**：future/promise模式

下一章我们将学习模板元编程的现代化技术。
