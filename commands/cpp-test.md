---
description: C++ test-driven development workflow using Google Test, Catch2, or other frameworks. Guides through Red-Green-Refactor cycle.
---

# C++ TDD Workflow

This command guides you through test-driven development for C++ using popular testing frameworks.

## What This Command Does

1. **Detect Test Framework**: Identify Google Test, Catch2, or other frameworks
2. **Create Test Structure**: Set up test files if needed
3. **Guide TDD Cycle**: Red -> Green -> Refactor
4. **Run Tests**: Execute and report results
5. **Track Coverage**: Report test coverage if available

## When to Use

Use `/cpp-test` when:
- Starting new feature development with TDD
- Adding tests to existing code
- Setting up test infrastructure
- Learning TDD with C++
- Improving test coverage

## TDD Workflow

### Step 1: RED - Write Failing Test

```cpp
// tests/calculator_test.cpp
#include <gtest/gtest.h>
#include "calculator.hpp"

TEST(CalculatorTest, AddTwoPositiveNumbers) {
    Calculator calc;
    EXPECT_EQ(calc.add(2, 3), 5);
}
```

### Step 2: GREEN - Minimal Implementation

```cpp
// src/calculator.hpp
class Calculator {
public:
    int add(int a, int b) {
        return a + b;  // Just enough to pass
    }
};
```

### Step 3: REFACTOR - Improve Code

```cpp
// Improve while keeping tests green
class Calculator {
public:
    [[nodiscard]] constexpr int add(int a, int b) const noexcept {
        return a + b;
    }
};
```

## Example Usage

```text
User: /cpp-test Create tests for a Stack class

Agent:
# TDD Session: Stack Class

## Step 1: RED - Writing Failing Tests

Creating test file:
```cpp
// tests/stack_test.cpp
#include <gtest/gtest.h>
#include "stack.hpp"

class StackTest : public ::testing::Test {
protected:
    Stack<int> stack;
};

TEST_F(StackTest, NewStackIsEmpty) {
    EXPECT_TRUE(stack.empty());
    EXPECT_EQ(stack.size(), 0);
}

TEST_F(StackTest, PushIncreasesSize) {
    stack.push(42);
    EXPECT_FALSE(stack.empty());
    EXPECT_EQ(stack.size(), 1);
}

TEST_F(StackTest, PopReturnsLastPushed) {
    stack.push(1);
    stack.push(2);
    EXPECT_EQ(stack.pop(), 2);
    EXPECT_EQ(stack.pop(), 1);
}

TEST_F(StackTest, PopOnEmptyThrows) {
    EXPECT_THROW(stack.pop(), std::runtime_error);
}
```

Running tests... FAILED (as expected)

## Step 2: GREEN - Implementing Stack

```cpp
// include/stack.hpp
#pragma once
#include <vector>
#include <stdexcept>

template<typename T>
class Stack {
    std::vector<T> data_;
    
public:
    void push(T value) {
        data_.push_back(std::move(value));
    }
    
    T pop() {
        if (data_.empty()) {
            throw std::runtime_error("Stack is empty");
        }
        T value = std::move(data_.back());
        data_.pop_back();
        return value;
    }
    
    bool empty() const { return data_.empty(); }
    size_t size() const { return data_.size(); }
};
```

Running tests... PASSED

## Step 3: REFACTOR

- Added noexcept where appropriate
- Added [[nodiscard]] to pop()
- Verified tests still pass

## Test Results
- Tests Run: 4
- Passed: 4
- Failed: 0
- Coverage: 100%
```

## Supported Frameworks

### Google Test
```cpp
#include <gtest/gtest.h>

TEST(TestSuite, TestName) {
    EXPECT_EQ(actual, expected);
    ASSERT_TRUE(condition);
}
```

### Catch2
```cpp
#include <catch2/catch_test_macros.hpp>

TEST_CASE("Description", "[tag]") {
    REQUIRE(actual == expected);
    
    SECTION("sub-case") {
        // ...
    }
}
```

## CMake Test Setup

```cmake
enable_testing()

# Google Test
find_package(GTest REQUIRED)
add_executable(tests
    tests/main.cpp
    tests/calculator_test.cpp
    tests/stack_test.cpp
)
target_link_libraries(tests GTest::gtest GTest::gtest_main)
add_test(NAME unit_tests COMMAND tests)

# Run with: ctest --output-on-failure
```

## Test Commands

```bash
# Build and run tests
cmake --build build --target tests
ctest --test-dir build --output-on-failure

# With coverage (gcov)
cmake -B build -DCMAKE_BUILD_TYPE=Debug -DENABLE_COVERAGE=ON
cmake --build build
ctest --test-dir build
lcov --capture --directory build --output-file coverage.info
```

## Integration with Other Commands

- Use `/cpp-build` if tests don't compile
- Use `/cpp-review` to review test quality
- Use `/code-review` for general code review

## Related

- Skills: `skills/cpp-testing/`, `skills/tdd-workflow/`
