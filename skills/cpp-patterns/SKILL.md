---
name: cpp-patterns
description: Modern C++ patterns, idioms, and best practices for building robust, efficient, and maintainable C++ applications (C++11/14/17/20).
---

# Modern C++ Development Patterns

Modern C++ patterns and best practices for building robust, efficient, and maintainable applications.

## When to Activate

- Writing new C++ code
- Reviewing C++ code
- Refactoring legacy C++ to modern standards
- Designing C++ libraries or APIs

## Core Principles

### 1. RAII (Resource Acquisition Is Initialization)

The most important C++ idiom - tie resource lifetime to object lifetime.

```cpp
// Good: RAII for file handling
class FileHandle {
    FILE* file_;
public:
    explicit FileHandle(const char* path, const char* mode)
        : file_(fopen(path, mode)) {
        if (!file_) throw std::runtime_error("Cannot open file");
    }
    ~FileHandle() {
        if (file_) fclose(file_);
    }
    
    // Disable copy
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
    
    // Enable move
    FileHandle(FileHandle&& other) noexcept : file_(other.file_) {
        other.file_ = nullptr;
    }
    FileHandle& operator=(FileHandle&& other) noexcept {
        if (this != &other) {
            if (file_) fclose(file_);
            file_ = other.file_;
            other.file_ = nullptr;
        }
        return *this;
    }
    
    FILE* get() const { return file_; }
};

// Better: Use standard library
std::ifstream file("data.txt"); // RAII built-in
```

### 2. Smart Pointers

Always prefer smart pointers over raw pointers for ownership.

```cpp
// unique_ptr: Exclusive ownership
auto widget = std::make_unique<Widget>();
auto array = std::make_unique<int[]>(100);

// shared_ptr: Shared ownership
auto shared = std::make_shared<Resource>();
auto copy = shared; // Reference count increases

// weak_ptr: Non-owning observer (breaks cycles)
std::weak_ptr<Node> parent;

// When to use raw pointers:
// - Non-owning references (function parameters)
// - Interfacing with C APIs
// - Performance-critical sections (with care)
void process(Widget* widget);  // Non-owning
process(owned_widget.get());
```

### 3. Rule of Zero/Three/Five

```cpp
// Rule of Zero: Prefer no custom special members
// Let smart pointers handle resources
class Modern {
    std::string name_;
    std::unique_ptr<Resource> resource_;
    std::vector<Data> data_;
    // No destructor, copy, move needed - compiler generates them
};

// Rule of Five: If you define one, define all five
class LegacyResource {
    int* data_;
    size_t size_;
public:
    LegacyResource(size_t n) : data_(new int[n]), size_(n) {}
    ~LegacyResource() { delete[] data_; }
    
    // Copy constructor
    LegacyResource(const LegacyResource& other)
        : data_(new int[other.size_]), size_(other.size_) {
        std::copy(other.data_, other.data_ + size_, data_);
    }
    
    // Copy assignment
    LegacyResource& operator=(const LegacyResource& other) {
        if (this != &other) {
            delete[] data_;
            size_ = other.size_;
            data_ = new int[size_];
            std::copy(other.data_, other.data_ + size_, data_);
        }
        return *this;
    }
    
    // Move constructor
    LegacyResource(LegacyResource&& other) noexcept
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
    }
    
    // Move assignment
    LegacyResource& operator=(LegacyResource&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            data_ = other.data_;
            size_ = other.size_;
            other.data_ = nullptr;
            other.size_ = 0;
        }
        return *this;
    }
};
```

## Move Semantics and Perfect Forwarding

### Move Semantics

```cpp
// Enable efficient transfer of resources
class Buffer {
    std::unique_ptr<char[]> data_;
    size_t size_;
public:
    Buffer(size_t n) : data_(new char[n]), size_(n) {}
    
    // Move constructor - steal resources
    Buffer(Buffer&& other) noexcept
        : data_(std::move(other.data_)), size_(other.size_) {
        other.size_ = 0;
    }
    
    // Move assignment
    Buffer& operator=(Buffer&& other) noexcept {
        data_ = std::move(other.data_);
        size_ = other.size_;
        other.size_ = 0;
        return *this;
    }
};

// Use std::move to enable moves
std::vector<Buffer> buffers;
Buffer b(1024);
buffers.push_back(std::move(b)); // Moves, doesn't copy
```

### Perfect Forwarding

```cpp
// Forward arguments with their original value category
template<typename T, typename... Args>
std::unique_ptr<T> make_unique_wrapper(Args&&... args) {
    return std::make_unique<T>(std::forward<Args>(args)...);
}

// Factory pattern with perfect forwarding
class Factory {
public:
    template<typename T, typename... Args>
    T* create(Args&&... args) {
        return new T(std::forward<Args>(args)...);
    }
};
```

## Error Handling Patterns

### Exceptions vs Error Codes

```cpp
// Use exceptions for exceptional conditions
void processFile(const std::string& path) {
    std::ifstream file(path);
    if (!file) {
        throw std::runtime_error("Cannot open file: " + path);
    }
    // Process file...
}

// Use expected/optional for expected failures (C++23 or use library)
std::optional<User> findUser(int id) {
    auto it = users_.find(id);
    if (it == users_.end()) {
        return std::nullopt;
    }
    return it->second;
}

// Error codes for performance-critical or embedded code
enum class ErrorCode { Success, NotFound, InvalidInput, Timeout };

ErrorCode readSensor(float& value) {
    if (!sensor_.ready()) return ErrorCode::NotFound;
    value = sensor_.read();
    return ErrorCode::Success;
}
```

### Exception Safety Guarantees

```cpp
// Basic guarantee: No leaks, invariants maintained
void basicGuarantee(std::vector<Widget>& widgets) {
    widgets.push_back(Widget()); // May throw
    // If throws, widgets is valid but may be modified
}

// Strong guarantee: All or nothing (transactional)
void strongGuarantee(std::vector<Widget>& widgets, Widget w) {
    std::vector<Widget> temp = widgets;  // Copy
    temp.push_back(std::move(w));        // Modify copy
    std::swap(widgets, temp);            // Commit (noexcept)
}

// Noexcept guarantee: Never throws
void noexceptGuarantee(int* a, int* b) noexcept {
    std::swap(*a, *b);
}
```

## Design Patterns in Modern C++

### Singleton (Thread-safe)

```cpp
class Singleton {
public:
    static Singleton& getInstance() {
        static Singleton instance; // Thread-safe in C++11+
        return instance;
    }
    
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    
private:
    Singleton() = default;
};
```

### Factory with Type Erasure

```cpp
class ShapeFactory {
    using Creator = std::function<std::unique_ptr<Shape>()>;
    std::unordered_map<std::string, Creator> creators_;
    
public:
    template<typename T>
    void registerShape(const std::string& name) {
        creators_[name] = []() { return std::make_unique<T>(); };
    }
    
    std::unique_ptr<Shape> create(const std::string& name) {
        auto it = creators_.find(name);
        if (it == creators_.end()) return nullptr;
        return it->second();
    }
};
```

### Pimpl (Pointer to Implementation)

```cpp
// header.h
class Widget {
public:
    Widget();
    ~Widget();
    Widget(Widget&&) noexcept;
    Widget& operator=(Widget&&) noexcept;
    
    void doSomething();
    
private:
    class Impl;
    std::unique_ptr<Impl> pImpl_;
};

// source.cpp
class Widget::Impl {
public:
    void doSomething() { /* implementation */ }
private:
    // All private members here
    std::string data_;
    std::vector<int> values_;
};

Widget::Widget() : pImpl_(std::make_unique<Impl>()) {}
Widget::~Widget() = default;
Widget::Widget(Widget&&) noexcept = default;
Widget& Widget::operator=(Widget&&) noexcept = default;

void Widget::doSomething() { pImpl_->doSomething(); }
```

### CRTP (Curiously Recurring Template Pattern)

```cpp
// Static polymorphism - no virtual overhead
template<typename Derived>
class Comparable {
public:
    bool operator>(const Derived& other) const {
        return other < static_cast<const Derived&>(*this);
    }
    bool operator<=(const Derived& other) const {
        return !(static_cast<const Derived&>(*this) > other);
    }
    bool operator>=(const Derived& other) const {
        return !(static_cast<const Derived&>(*this) < other);
    }
};

class Number : public Comparable<Number> {
    int value_;
public:
    Number(int v) : value_(v) {}
    bool operator<(const Number& other) const {
        return value_ < other.value_;
    }
};
```

## Template Patterns

### Type Traits and SFINAE

```cpp
// Enable function only for specific types
template<typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
process(T value) {
    return value * 2;
}

// C++17: if constexpr
template<typename T>
auto process(T value) {
    if constexpr (std::is_integral_v<T>) {
        return value * 2;
    } else if constexpr (std::is_floating_point_v<T>) {
        return value * 2.0;
    } else {
        return value;
    }
}

// C++20: Concepts
template<typename T>
concept Numeric = std::is_arithmetic_v<T>;

template<Numeric T>
T process(T value) {
    return value * 2;
}
```

### Variadic Templates

```cpp
// Type-safe printf alternative
template<typename T>
void log(const T& value) {
    std::cout << value << std::endl;
}

template<typename T, typename... Args>
void log(const T& first, const Args&... rest) {
    std::cout << first << " ";
    log(rest...);
}

// C++17: Fold expressions
template<typename... Args>
auto sum(Args... args) {
    return (args + ...);  // Unary right fold
}

template<typename... Args>
void printAll(const Args&... args) {
    (std::cout << ... << args) << std::endl;  // Binary left fold
}
```

## Concurrency Patterns

### Thread-safe Lazy Initialization

```cpp
class LazyResource {
    mutable std::once_flag initFlag_;
    mutable std::unique_ptr<Resource> resource_;
    
public:
    Resource& get() const {
        std::call_once(initFlag_, [this]() {
            resource_ = std::make_unique<Resource>();
        });
        return *resource_;
    }
};
```

### Producer-Consumer Queue

```cpp
template<typename T>
class ThreadSafeQueue {
    std::queue<T> queue_;
    mutable std::mutex mutex_;
    std::condition_variable cond_;
    
public:
    void push(T value) {
        std::lock_guard<std::mutex> lock(mutex_);
        queue_.push(std::move(value));
        cond_.notify_one();
    }
    
    T pop() {
        std::unique_lock<std::mutex> lock(mutex_);
        cond_.wait(lock, [this]{ return !queue_.empty(); });
        T value = std::move(queue_.front());
        queue_.pop();
        return value;
    }
    
    std::optional<T> tryPop() {
        std::lock_guard<std::mutex> lock(mutex_);
        if (queue_.empty()) return std::nullopt;
        T value = std::move(queue_.front());
        queue_.pop();
        return value;
    }
};
```

### Async Operations

```cpp
// std::async for simple parallelism
auto future = std::async(std::launch::async, []() {
    return heavyComputation();
});
// Do other work...
auto result = future.get();

// std::packaged_task for more control
std::packaged_task<int()> task([]() { return compute(); });
auto future = task.get_future();
std::thread t(std::move(task));
t.detach();
// Later: future.get()

// Promise/Future for manual control
std::promise<Result> promise;
auto future = promise.get_future();
std::thread([&promise]() {
    try {
        promise.set_value(compute());
    } catch (...) {
        promise.set_exception(std::current_exception());
    }
}).detach();
```

## Modern C++ Features Quick Reference

### C++11

- `auto`, `decltype`
- Range-based for loops
- Lambda expressions
- Smart pointers (`unique_ptr`, `shared_ptr`)
- Move semantics, rvalue references
- `nullptr`
- `constexpr`
- Variadic templates
- `std::thread`, `std::mutex`, `std::atomic`

### C++14

- Generic lambdas (`auto` parameters)
- Return type deduction
- `std::make_unique`
- Binary literals (`0b1010`)
- Digit separators (`1'000'000`)

### C++17

- Structured bindings (`auto [a, b] = pair`)
- `if constexpr`
- Fold expressions
- `std::optional`, `std::variant`, `std::any`
- `std::string_view`
- Parallel algorithms
- Filesystem library

### C++20

- Concepts and constraints
- Ranges library
- Coroutines
- Three-way comparison (`<=>`)
- Modules
- `std::format`
- Calendar and timezone library

## Performance Considerations

```cpp
// Prefer stack allocation
void process() {
    std::array<int, 100> arr;  // Stack
    // Not: auto arr = std::make_unique<int[]>(100);  // Heap
}

// Reserve capacity for vectors
std::vector<Item> items;
items.reserve(expectedSize);

// Use emplace instead of push
std::vector<std::pair<int, std::string>> pairs;
pairs.emplace_back(1, "one");  // Constructs in-place
// Not: pairs.push_back({1, "one"});  // Creates temporary

// Move instead of copy
std::string makeString();
std::string s = makeString();  // RVO/NRVO - no copy

// String view for non-owning references
void process(std::string_view sv);  // No copy
```

## Anti-Patterns to Avoid

| Anti-Pattern | Better Alternative |
|--------------|-------------------|
| Raw `new`/`delete` | Smart pointers |
| C-style casts | `static_cast`, `dynamic_cast` |
| `NULL` or `0` for null | `nullptr` |
| C arrays | `std::array`, `std::vector` |
| Output parameters | Return by value (RVO) |
| `#define` constants | `constexpr` |
| Inheritance for code reuse | Composition |
| Deep inheritance | Interfaces + composition |

**Remember**: Modern C++ should feel almost as safe as a managed language while maintaining its performance advantage. When in doubt, let the compiler do the work.
