---
name: cpp-testing
description: C++ testing patterns, TDD workflow, and best practices using Google Test, Catch2, and other frameworks.
---

# C++ Testing Patterns

Comprehensive guide to testing C++ code with modern frameworks and TDD practices.

## When to Activate

- Writing unit tests for C++ code
- Setting up test infrastructure
- Following TDD workflow
- Reviewing test code
- Improving test coverage

## Testing Frameworks

### Google Test (gtest)

Most widely used C++ testing framework.

```cpp
#include <gtest/gtest.h>

// Simple test
TEST(CalculatorTest, AddPositiveNumbers) {
    Calculator calc;
    EXPECT_EQ(calc.add(2, 3), 5);
}

// Test with fixtures
class StackTest : public ::testing::Test {
protected:
    void SetUp() override {
        stack.push(1);
        stack.push(2);
    }
    
    void TearDown() override {
        // Cleanup if needed
    }
    
    Stack<int> stack;
};

TEST_F(StackTest, PopReturnsLastPushed) {
    EXPECT_EQ(stack.pop(), 2);
    EXPECT_EQ(stack.pop(), 1);
}

TEST_F(StackTest, PopOnEmptyThrows) {
    stack.pop();
    stack.pop();
    EXPECT_THROW(stack.pop(), std::runtime_error);
}
```

### Catch2

Header-only, expressive syntax.

```cpp
#define CATCH_CONFIG_MAIN
#include <catch2/catch.hpp>

TEST_CASE("Calculator addition", "[calculator]") {
    Calculator calc;
    
    SECTION("positive numbers") {
        REQUIRE(calc.add(2, 3) == 5);
    }
    
    SECTION("negative numbers") {
        REQUIRE(calc.add(-2, -3) == -5);
    }
    
    SECTION("mixed numbers") {
        REQUIRE(calc.add(5, -3) == 2);
    }
}

TEST_CASE("Stack operations", "[stack]") {
    Stack<int> stack;
    
    REQUIRE(stack.empty());
    
    SECTION("push increases size") {
        stack.push(1);
        REQUIRE(stack.size() == 1);
        
        SECTION("and allows pop") {
            REQUIRE(stack.pop() == 1);
            REQUIRE(stack.empty());
        }
    }
}
```

### CMake Integration

```cmake
# CMakeLists.txt
enable_testing()

# Google Test
find_package(GTest REQUIRED)
add_executable(tests
    test_calculator.cpp
    test_stack.cpp
)
target_link_libraries(tests GTest::gtest GTest::gtest_main)
add_test(NAME unit_tests COMMAND tests)

# Or fetch Google Test automatically
include(FetchContent)
FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG release-1.12.1
)
FetchContent_MakeAvailable(googletest)

# Catch2
find_package(Catch2 3 REQUIRED)
add_executable(tests test_main.cpp)
target_link_libraries(tests PRIVATE Catch2::Catch2WithMain)
```

## TDD Workflow

### Red-Green-Refactor Cycle

```cpp
// 1. RED: Write failing test first
TEST(UserRepository, FindByIdReturnsUser) {
    UserRepository repo;
    repo.add(User{1, "Alice"});
    
    auto user = repo.findById(1);
    
    ASSERT_TRUE(user.has_value());
    EXPECT_EQ(user->name, "Alice");
}

// 2. GREEN: Minimal implementation to pass
class UserRepository {
    std::unordered_map<int, User> users_;
public:
    void add(User user) {
        users_[user.id] = std::move(user);
    }
    
    std::optional<User> findById(int id) {
        auto it = users_.find(id);
        if (it == users_.end()) return std::nullopt;
        return it->second;
    }
};

// 3. REFACTOR: Improve without changing behavior
// - Extract interfaces
// - Apply design patterns
// - Optimize if needed
```

### Test-First Checklist

1. **Understand requirement** - What should the code do?
2. **Write test** - Express expected behavior as test
3. **Run test** - Verify it fails (red)
4. **Implement** - Minimal code to pass
5. **Run test** - Verify it passes (green)
6. **Refactor** - Improve code, keep tests green
7. **Repeat** - Next requirement

## Test Patterns

### Arrange-Act-Assert (AAA)

```cpp
TEST(OrderProcessor, AppliesDiscountForLargeOrders) {
    // Arrange
    OrderProcessor processor;
    Order order;
    order.addItem(Item{"Widget", 100.0});
    order.addItem(Item{"Gadget", 150.0});
    
    // Act
    double total = processor.calculateTotal(order);
    
    // Assert
    EXPECT_DOUBLE_EQ(total, 225.0);  // 10% discount for orders > $200
}
```

### Given-When-Then (BDD Style)

```cpp
// Using Catch2's BDD macros
SCENARIO("User login", "[auth]") {
    GIVEN("a registered user") {
        AuthService auth;
        auth.registerUser("alice", "password123");
        
        WHEN("logging in with correct credentials") {
            auto result = auth.login("alice", "password123");
            
            THEN("login succeeds") {
                REQUIRE(result.success);
                REQUIRE(result.token.has_value());
            }
        }
        
        WHEN("logging in with wrong password") {
            auto result = auth.login("alice", "wrongpass");
            
            THEN("login fails") {
                REQUIRE_FALSE(result.success);
                REQUIRE_FALSE(result.token.has_value());
            }
        }
    }
}
```

### Parameterized Tests

```cpp
// Google Test
class PrimeTest : public ::testing::TestWithParam<int> {};

TEST_P(PrimeTest, IsPrime) {
    EXPECT_TRUE(isPrime(GetParam()));
}

INSTANTIATE_TEST_SUITE_P(
    Primes, PrimeTest,
    ::testing::Values(2, 3, 5, 7, 11, 13, 17, 19)
);

// Catch2
TEST_CASE("Fibonacci sequence", "[fibonacci]") {
    auto [input, expected] = GENERATE(table<int, int>({
        {0, 0},
        {1, 1},
        {2, 1},
        {3, 2},
        {4, 3},
        {5, 5},
        {10, 55}
    }));
    
    REQUIRE(fibonacci(input) == expected);
}
```

## Mocking and Stubbing

### Google Mock

```cpp
#include <gmock/gmock.h>

// Interface to mock
class Database {
public:
    virtual ~Database() = default;
    virtual std::optional<User> findUser(int id) = 0;
    virtual bool saveUser(const User& user) = 0;
};

// Mock class
class MockDatabase : public Database {
public:
    MOCK_METHOD(std::optional<User>, findUser, (int id), (override));
    MOCK_METHOD(bool, saveUser, (const User& user), (override));
};

// Test with mock
TEST(UserService, UpdateUserSavesToDatabase) {
    MockDatabase mockDb;
    UserService service(&mockDb);
    
    User existingUser{1, "Alice"};
    User updatedUser{1, "Alice Smith"};
    
    // Set expectations
    EXPECT_CALL(mockDb, findUser(1))
        .WillOnce(::testing::Return(existingUser));
    EXPECT_CALL(mockDb, saveUser(updatedUser))
        .WillOnce(::testing::Return(true));
    
    // Act
    bool result = service.updateName(1, "Alice Smith");
    
    // Assert
    EXPECT_TRUE(result);
}

// Matchers
EXPECT_CALL(mockDb, saveUser(::testing::Field(&User::name, "Bob")));
EXPECT_CALL(mockDb, findUser(::testing::Gt(0)));  // Greater than 0
```

### Manual Stubs

```cpp
// Sometimes simpler than mocks
class StubDatabase : public Database {
    std::unordered_map<int, User> data_;
public:
    void setUser(User user) {
        data_[user.id] = std::move(user);
    }
    
    std::optional<User> findUser(int id) override {
        auto it = data_.find(id);
        return it != data_.end() ? std::optional{it->second} : std::nullopt;
    }
    
    bool saveUser(const User& user) override {
        data_[user.id] = user;
        return true;
    }
};
```

### Dependency Injection for Testability

```cpp
// Bad: Hard to test
class OrderService {
    SqlDatabase db_;  // Concrete dependency
public:
    void processOrder(Order order) {
        db_.save(order);  // Can't mock
    }
};

// Good: Inject dependency
class OrderService {
    std::unique_ptr<IDatabase> db_;
public:
    explicit OrderService(std::unique_ptr<IDatabase> db)
        : db_(std::move(db)) {}
    
    void processOrder(Order order) {
        db_->save(order);  // Can inject mock
    }
};

// Test
TEST(OrderService, ProcessOrderSavesToDatabase) {
    auto mockDb = std::make_unique<MockDatabase>();
    EXPECT_CALL(*mockDb, save(::testing::_));
    
    OrderService service(std::move(mockDb));
    service.processOrder(Order{});
}
```

## Testing Strategies

### Testing Private Members

```cpp
// Option 1: Test through public interface (preferred)
TEST(Calculator, InternalCacheWorks) {
    Calculator calc;
    // Call public methods that exercise private cache
    calc.compute(42);
    calc.compute(42);  // Should hit cache
    // Assert on observable behavior
}

// Option 2: Friend test class
class Calculator {
    friend class CalculatorTest;
    // ...
private:
    int cached_result_;
};

// Option 3: Protected + test subclass
class Calculator {
protected:
    int cached_result_;
};

class TestableCalculator : public Calculator {
public:
    int getCachedResult() const { return cached_result_; }
};
```

### Testing Exceptions

```cpp
// Google Test
TEST(Parser, ThrowsOnInvalidInput) {
    Parser parser;
    EXPECT_THROW(parser.parse("invalid{"), ParseException);
    EXPECT_THROW(parser.parse(""), std::invalid_argument);
    
    // Check exception message
    try {
        parser.parse("bad");
        FAIL() << "Expected ParseException";
    } catch (const ParseException& e) {
        EXPECT_THAT(e.what(), ::testing::HasSubstr("unexpected token"));
    }
}

// Catch2
TEST_CASE("Parser throws on invalid input") {
    Parser parser;
    REQUIRE_THROWS_AS(parser.parse("invalid"), ParseException);
    REQUIRE_THROWS_WITH(parser.parse("bad"), Catch::Contains("error"));
}
```

### Testing Async Code

```cpp
// Using futures
TEST(AsyncService, CompletesSuccessfully) {
    AsyncService service;
    
    auto future = service.fetchDataAsync();
    
    // Wait with timeout
    auto status = future.wait_for(std::chrono::seconds(5));
    ASSERT_EQ(status, std::future_status::ready);
    
    auto result = future.get();
    EXPECT_TRUE(result.valid());
}

// Using callbacks with promise
TEST(AsyncService, CallsCompletionHandler) {
    AsyncService service;
    std::promise<Result> promise;
    auto future = promise.get_future();
    
    service.fetchData([&promise](Result r) {
        promise.set_value(std::move(r));
    });
    
    auto result = future.get();
    EXPECT_TRUE(result.valid());
}
```

## Benchmarking

### Google Benchmark

```cpp
#include <benchmark/benchmark.h>

static void BM_VectorPushBack(benchmark::State& state) {
    for (auto _ : state) {
        std::vector<int> v;
        for (int i = 0; i < state.range(0); ++i) {
            v.push_back(i);
        }
        benchmark::DoNotOptimize(v);
    }
}
BENCHMARK(BM_VectorPushBack)->Range(8, 8<<10);

static void BM_VectorReserve(benchmark::State& state) {
    for (auto _ : state) {
        std::vector<int> v;
        v.reserve(state.range(0));
        for (int i = 0; i < state.range(0); ++i) {
            v.push_back(i);
        }
        benchmark::DoNotOptimize(v);
    }
}
BENCHMARK(BM_VectorReserve)->Range(8, 8<<10);

BENCHMARK_MAIN();
```

### Catch2 Benchmarking

```cpp
TEST_CASE("Fibonacci benchmark", "[benchmark]") {
    BENCHMARK("fibonacci(20)") {
        return fibonacci(20);
    };
    
    BENCHMARK_ADVANCED("fibonacci with setup")(Catch::Benchmark::Chronometer meter) {
        std::vector<int> inputs(meter.runs());
        std::fill(inputs.begin(), inputs.end(), 25);
        meter.measure([&inputs](int i) { return fibonacci(inputs[i]); });
    };
}
```

## Coverage and Quality

### Code Coverage with gcov/lcov

```bash
# Compile with coverage flags
g++ -fprofile-arcs -ftest-coverage -O0 -g tests.cpp -o tests

# Run tests
./tests

# Generate coverage report
lcov --capture --directory . --output-file coverage.info
genhtml coverage.info --output-directory coverage_report
```

### CMake Coverage Target

```cmake
option(ENABLE_COVERAGE "Enable coverage reporting" OFF)

if(ENABLE_COVERAGE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --coverage -O0 -g")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} --coverage")
endif()

# Add custom target
add_custom_target(coverage
    COMMAND ${CMAKE_CTEST_COMMAND}
    COMMAND lcov --capture --directory . --output-file coverage.info
    COMMAND genhtml coverage.info --output-directory coverage_report
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
)
```

## Best Practices

### Test Organization

```text
project/
├── src/
│   ├── calculator.cpp
│   └── calculator.h
├── tests/
│   ├── CMakeLists.txt
│   ├── test_main.cpp
│   ├── calculator_test.cpp
│   ├── mocks/
│   │   └── mock_database.h
│   └── fixtures/
│       └── test_data.json
└── CMakeLists.txt
```

### Test Naming Conventions

```cpp
// Pattern: [Unit]_[Scenario]_[ExpectedResult]
TEST(Calculator, AddTwoPositiveNumbers_ReturnsSum);
TEST(Parser, EmptyInput_ThrowsInvalidArgument);
TEST(UserService, ValidCredentials_ReturnsAuthToken);

// Or: [Given]_[When]_[Then]
TEST(Stack, GivenEmptyStack_WhenPop_ThenThrowsException);
```

### Avoiding Test Smells

| Smell | Problem | Solution |
|-------|---------|----------|
| Long test | Hard to understand | Split into focused tests |
| Logic in test | Test becomes code under test | Keep assertions simple |
| Shared state | Tests affect each other | Isolate with fixtures |
| Flaky tests | Random failures | Remove non-determinism |
| Slow tests | CI bottleneck | Mock external dependencies |
| Testing implementation | Brittle tests | Test behavior, not internals |

### Test Checklist

- [ ] Tests are fast (< 1 second each)
- [ ] Tests are isolated (no shared mutable state)
- [ ] Tests are deterministic (same result every run)
- [ ] Tests have clear names describing intent
- [ ] Tests follow AAA or Given-When-Then pattern
- [ ] Edge cases are covered
- [ ] Error conditions are tested
- [ ] Tests run in CI on every commit

**Remember**: Good tests are documentation. They should clearly express what the code is supposed to do.
