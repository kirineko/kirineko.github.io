---
title: 第五章
createTime: 2025/05/28 00:56:11
permalink: /cpp/zmzv4urx/
---

# 第5章：现代STL容器与算法

## 5.1 新增容器：array, unordered_map, unordered_set

### std::array - 固定大小数组

`std::array`是C++11引入的固定大小数组容器，结合了C数组的性能和STL容器的安全性：

```cpp
#include <array>

// 创建固定大小数组
std::array<int, 5> arr{1, 2, 3, 4, 5};

// 安全访问
arr.at(0);  // 边界检查
arr[0];     // 无边界检查（更快）

// 获取大小
constexpr size_t size = arr.size();  // 编译时常量

// 迭代器支持
for (auto it = arr.begin(); it != arr.end(); ++it) {
    std::cout << *it << " ";
}
```

### std::unordered_map - 哈希映射

基于哈希表的关联容器，提供平均O(1)的查找性能：

```cpp
#include <unordered_map>

std::unordered_map<std::string, int> scores{
    {"Alice", 95},
    {"Bob", 87},
    {"Charlie", 92}
};

// 插入和查找
scores["David"] = 88;
auto it = scores.find("Alice");
if (it != scores.end()) {
    std::cout << it->first << ": " << it->second << std::endl;
}

// 自定义哈希函数
struct Person {
    std::string name;
    int age;
};

struct PersonHash {
    size_t operator()(const Person& p) const {
        return std::hash<std::string>{}(p.name) ^ 
               std::hash<int>{}(p.age);
    }
};

std::unordered_map<Person, int, PersonHash> person_scores;
```

### std::unordered_set - 哈希集合

```cpp
#include <unordered_set>

std::unordered_set<std::string> unique_words{
    "apple", "banana", "cherry"
};

// 插入和查找
unique_words.insert("date");
if (unique_words.count("apple")) {
    std::cout << "Found apple!" << std::endl;
}

// 去重操作
std::vector<int> numbers{1, 2, 2, 3, 3, 3, 4};
std::unordered_set<int> unique_numbers(numbers.begin(), numbers.end());
```

## 5.2 容器的emplace操作

### emplace vs insert/push_back

`emplace`操作直接在容器中构造对象，避免不必要的复制或移动：

```cpp
class Person {
public:
    Person(const std::string& name, int age) 
        : name_(name), age_(age) {
        std::cout << "Person constructed: " << name_ << std::endl;
    }
    
    Person(const Person& other) 
        : name_(other.name_), age_(other.age_) {
        std::cout << "Person copied: " << name_ << std::endl;
    }
    
private:
    std::string name_;
    int age_;
};

std::vector<Person> people;

// 传统方式 - 可能涉及复制
people.push_back(Person("Alice", 25));  // 构造临时对象，然后移动

// 现代方式 - 直接构造
people.emplace_back("Bob", 30);  // 直接在vector中构造
```

### 各种容器的emplace操作

```cpp
// vector
std::vector<std::pair<int, std::string>> vec;
vec.emplace_back(1, "one");  // 直接构造pair

// map
std::map<std::string, Person> person_map;
person_map.emplace("key1", "Alice", 25);  // 直接构造Person

// set
std::set<Person> person_set;
person_set.emplace("Charlie", 35);  // 直接构造Person

// unordered_map
std::unordered_map<int, std::string> hash_map;
hash_map.emplace(1, "value1");  // 直接构造pair
```

## 5.3 现代算法库

### 新增算法

C++11及以后版本添加了许多有用的算法：

```cpp
#include <algorithm>
#include <numeric>

std::vector<int> numbers{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// all_of, any_of, none_of
bool all_positive = std::all_of(numbers.begin(), numbers.end(),
                                [](int n) { return n > 0; });

bool has_even = std::any_of(numbers.begin(), numbers.end(),
                           [](int n) { return n % 2 == 0; });

bool no_negative = std::none_of(numbers.begin(), numbers.end(),
                               [](int n) { return n < 0; });

// copy_if
std::vector<int> even_numbers;
std::copy_if(numbers.begin(), numbers.end(),
             std::back_inserter(even_numbers),
             [](int n) { return n % 2 == 0; });

// iota - 生成递增序列
std::vector<int> sequence(10);
std::iota(sequence.begin(), sequence.end(), 1);  // 1, 2, 3, ..., 10
```

### 并行算法 (C++17)

```cpp
#include <execution>

std::vector<int> large_vector(1000000);
std::iota(large_vector.begin(), large_vector.end(), 1);

// 并行排序
std::sort(std::execution::par, 
          large_vector.begin(), large_vector.end());

// 并行变换
std::transform(std::execution::par_unseq,
               large_vector.begin(), large_vector.end(),
               large_vector.begin(),
               [](int n) { return n * n; });
```

## 5.4 范围算法 (C++20预览)

### ranges库

C++20引入了ranges库，提供更直观的算法接口：

```cpp
#include <ranges>
#include <algorithm>

std::vector<int> numbers{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// 传统方式
std::sort(numbers.begin(), numbers.end());

// ranges方式
std::ranges::sort(numbers);

// 管道操作
auto even_squares = numbers 
    | std::views::filter([](int n) { return n % 2 == 0; })
    | std::views::transform([](int n) { return n * n; });

for (int value : even_squares) {
    std::cout << value << " ";
}
```

### 视图 (Views)

```cpp
// 惰性求值的视图
auto filtered_view = numbers | std::views::filter([](int n) { return n > 5; });
auto transformed_view = filtered_view | std::views::transform([](int n) { return n * 2; });

// 组合视图
auto pipeline = std::views::iota(1, 100)  // 生成1到99
    | std::views::filter([](int n) { return n % 3 == 0; })  // 过滤3的倍数
    | std::views::transform([](int n) { return n * n; })     // 平方
    | std::views::take(5);  // 取前5个

for (int value : pipeline) {
    std::cout << value << " ";  // 输出: 9 36 81 144 225
}
```

## 5.5 迭代器适配器

### 插入迭代器

```cpp
std::vector<int> source{1, 2, 3, 4, 5};
std::vector<int> dest;

// back_inserter
std::copy(source.begin(), source.end(), std::back_inserter(dest));

// front_inserter (for deque, list)
std::deque<int> deq;
std::copy(source.begin(), source.end(), std::front_inserter(deq));

// inserter
std::set<int> s;
std::copy(source.begin(), source.end(), std::inserter(s, s.begin()));
```

### 移动迭代器

```cpp
std::vector<std::string> source{"apple", "banana", "cherry"};
std::vector<std::string> dest;

// 移动元素而不是复制
std::copy(std::make_move_iterator(source.begin()),
          std::make_move_iterator(source.end()),
          std::back_inserter(dest));

// source中的字符串现在为空
```

### 反向迭代器

```cpp
std::vector<int> numbers{1, 2, 3, 4, 5};

// 反向遍历
for (auto it = numbers.rbegin(); it != numbers.rend(); ++it) {
    std::cout << *it << " ";  // 输出: 5 4 3 2 1
}

// 反向算法
std::reverse_copy(numbers.begin(), numbers.end(),
                  std::ostream_iterator<int>(std::cout, " "));
```

## 容器选择指南

### 序列容器选择

| 容器 | 特点 | 适用场景 |
|------|------|----------|
| `vector` | 动态数组，连续内存 | 默认选择，随机访问 |
| `array` | 固定大小，连续内存 | 编译时已知大小 |
| `deque` | 双端队列 | 频繁首尾插入删除 |
| `list` | 双向链表 | 频繁中间插入删除 |
| `forward_list` | 单向链表 | 内存受限，单向遍历 |

### 关联容器选择

| 容器 | 特点 | 适用场景 |
|------|------|----------|
| `map` | 有序映射，O(log n) | 需要有序遍历 |
| `unordered_map` | 哈希映射，O(1)平均 | 快速查找，无序要求 |
| `set` | 有序集合，O(log n) | 需要有序的唯一元素 |
| `unordered_set` | 哈希集合，O(1)平均 | 快速查找唯一元素 |

---

## 本章小结

现代STL提供了丰富的容器和算法：
- **新容器**：array、unordered_map、unordered_set提供更多选择
- **emplace操作**：避免不必要的复制，提高性能
- **现代算法**：更丰富的算法库，支持并行执行
- **ranges库**：C++20的革命性改进，更直观的API
- **迭代器适配器**：灵活的迭代器操作

下一章我们将学习并发编程，这是现代C++的重要特性。
