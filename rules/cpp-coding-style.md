---
description: C++ coding style and best practices rules for modern C++ development.
globs: ["*.cpp", "*.hpp", "*.h", "*.cc", "*.cxx", "*.hxx"]
alwaysApply: false
---

# C++ Coding Style Rules

These rules apply to all C++ code in this project.

## Language Standard

- **Use C++17 or later** - Default to C++17; use C++20 features when available
- **Enable all warnings** - Compile with `-Wall -Wextra -Wpedantic -Werror`
- **Use standard library** - Prefer STL over custom implementations

## Memory Management

### ALWAYS

- Use **smart pointers** for ownership: `std::unique_ptr`, `std::shared_ptr`
- Apply **RAII** for all resource management
- Use `std::make_unique` and `std::make_shared` for allocation
- Pass raw pointers for **non-owning references** only

### NEVER

- Use raw `new`/`delete` without RAII wrapper
- Return raw pointers with ownership (return smart pointers instead)
- Ignore memory tools warnings (AddressSanitizer, Valgrind)

```cpp
// GOOD
auto widget = std::make_unique<Widget>();
void process(Widget* w);  // Non-owning

// BAD
Widget* widget = new Widget();  // Who owns this?
```

## Modern C++ Features

### Required

- Use `nullptr` instead of `NULL` or `0`
- Use `auto` for complex types and iterators
- Use range-based for loops
- Use `override` and `final` for virtual functions
- Use `constexpr` for compile-time constants
- Use `noexcept` for non-throwing functions

### Preferred

- Use structured bindings: `auto [key, value] = pair;`
- Use `std::optional` for optional values
- Use `std::string_view` for read-only string parameters
- Use `[[nodiscard]]` for functions with important return values

```cpp
// GOOD
for (const auto& item : container) { ... }
constexpr int MAX_SIZE = 100;
void process() noexcept;
[[nodiscard]] bool validate();

// BAD
for (int i = 0; i < container.size(); ++i) { ... }
#define MAX_SIZE 100
```

## Naming Conventions

| Element | Style | Example |
|---------|-------|---------|
| Classes/Structs | PascalCase | `UserManager`, `HttpClient` |
| Functions/Methods | camelCase | `processData()`, `getValue()` |
| Variables | camelCase | `itemCount`, `userName` |
| Member Variables | camelCase with suffix | `value_`, `count_` |
| Constants | UPPER_SNAKE_CASE | `MAX_BUFFER_SIZE` |
| Namespaces | lowercase | `network`, `utils` |
| Template Parameters | PascalCase | `typename T`, `typename Container` |
| Macros (avoid) | UPPER_SNAKE_CASE | `DEBUG_LOG` |

## Header Files

### Organization

```cpp
#pragma once

// 1. Related header (for .cpp files)
#include "this_file.hpp"

// 2. C system headers
#include <cstdint>
#include <cstring>

// 3. C++ standard library
#include <memory>
#include <string>
#include <vector>

// 4. Third-party libraries
#include <boost/asio.hpp>
#include <nlohmann/json.hpp>

// 5. Project headers
#include "utils/helper.hpp"
#include "core/manager.hpp"
```

### Best Practices

- Use `#pragma once` (widely supported) or include guards
- Use forward declarations to minimize includes
- Keep headers minimal - implementation in .cpp files
- No `using namespace` in headers

## Class Design

### Structure

```cpp
class Widget {
public:
    // Types and aliases
    using Callback = std::function<void()>;
    
    // Static constants
    static constexpr int MAX_SIZE = 100;
    
    // Constructors and destructor
    Widget();
    explicit Widget(int value);
    ~Widget();
    
    // Copy/Move (follow Rule of 0, 3, or 5)
    Widget(const Widget&) = delete;
    Widget& operator=(const Widget&) = delete;
    Widget(Widget&&) noexcept;
    Widget& operator=(Widget&&) noexcept;
    
    // Public interface
    void process();
    [[nodiscard]] int getValue() const;
    
protected:
    // For inheritance
    virtual void onEvent();
    
private:
    // Implementation details
    void internalHelper();
    
    // Data members last
    int value_ = 0;
    std::string name_;
};
```

### Guidelines

- Mark single-argument constructors `explicit`
- Prefer composition over inheritance
- Make interfaces (pure virtual classes) minimal
- Follow Rule of Zero when possible

## Error Handling

### Exceptions

- Use exceptions for **exceptional** conditions
- Derive custom exceptions from `std::exception`
- Document exception guarantees (basic, strong, noexcept)

```cpp
// GOOD - Exceptional condition
if (!file.open()) {
    throw std::runtime_error("Cannot open file: " + path);
}

// GOOD - Expected failure, use optional
std::optional<User> findUser(int id);
```

### Error Codes

- Use for performance-critical or embedded code
- Use `enum class` for type safety
- Document error conditions

```cpp
enum class ErrorCode {
    Success,
    NotFound,
    InvalidInput,
    Timeout
};

[[nodiscard]] ErrorCode process(Data& result);
```

## Concurrency

- Protect shared data with `std::mutex` and `std::lock_guard`
- Use `std::atomic` for simple shared variables
- Prefer `std::scoped_lock` for multiple mutexes
- Document thread safety in comments

```cpp
class ThreadSafeCounter {
    mutable std::mutex mutex_;
    int value_ = 0;
    
public:
    // Thread-safe
    int increment() {
        std::lock_guard lock(mutex_);
        return ++value_;
    }
};
```

## Performance

### Avoid

- Unnecessary copies (pass by const reference)
- Virtual functions in tight loops
- Dynamic allocation in hot paths
- String concatenation in loops (use `std::ostringstream`)

### Prefer

- Move semantics for large objects
- `reserve()` for vectors when size is known
- `emplace_back()` over `push_back()` for construction
- Stack allocation over heap when possible

## Documentation

### Required

- Public API documentation (Doxygen style)
- Complex algorithm explanations
- Non-obvious design decisions
- Thread safety guarantees

```cpp
/**
 * @brief Processes the data buffer and returns the result.
 * 
 * @param data Input buffer to process
 * @param size Size of the input buffer
 * @return Processed result, or std::nullopt on failure
 * 
 * @throws std::invalid_argument if data is null
 * @note Thread-safe
 */
std::optional<Result> processData(const uint8_t* data, size_t size);
```

## Code Organization

### File Size

- Keep files under **500 lines** (prefer 200-300)
- One class per file (with exceptions for closely related classes)
- Split large classes into multiple files

### Function Size

- Keep functions under **50 lines** (prefer 20-30)
- Single responsibility per function
- Extract complex conditions into named functions

## Prohibited

- `using namespace std;` in headers
- C-style casts (use `static_cast`, `dynamic_cast`, etc.)
- `goto` statements
- Macros for constants or functions
- Magic numbers without named constants
- Commented-out code in production
- `std::auto_ptr` (deprecated)

## Qt-Specific (if applicable)

- Use modern signal-slot syntax
- Always include Q_OBJECT for signals/slots
- Use `deleteLater()` for cross-thread deletion
- Prefer `QString` for Qt APIs, `std::string` for std APIs

## Embedded-Specific (if applicable)

- Avoid dynamic allocation
- Disable exceptions and RTTI
- Use `volatile` for hardware registers
- Keep ISRs short and fast
- Mark shared ISR variables as `volatile`
