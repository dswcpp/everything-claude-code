---
description: Comprehensive C++ code review for modern C++ patterns, memory safety, performance, and security. Invokes the cpp-reviewer agent.
---

# C++ Code Review

This command invokes the **cpp-reviewer** agent for comprehensive C++-specific code review.

## What This Command Does

1. **Identify C++ Changes**: Find modified `.cpp`, `.hpp`, `.h`, `.cc`, `.cxx` files via `git diff`
2. **Run Static Analysis**: Execute `clang-tidy`, `cppcheck` if available
3. **Memory Safety Check**: Analyze for leaks, dangling pointers, RAII violations
4. **Modern C++ Verification**: Check for modern C++ (11/14/17/20) best practices
5. **Security Scan**: Check for buffer overflows, injection vulnerabilities
6. **Generate Report**: Categorize issues by severity

## When to Use

Use `/cpp-review` when:
- After writing or modifying C++ code
- Before committing C++ changes
- Reviewing pull requests with C++ code
- Migrating legacy C++ to modern standards
- Learning modern C++ patterns

## Review Categories

### CRITICAL (Must Fix)
- Buffer overflow vulnerabilities
- Use-after-free / dangling pointers
- Memory leaks without RAII
- SQL/Command injection
- Uninitialized variables
- Race conditions
- Hardcoded credentials

### HIGH (Should Fix)
- Raw pointer ownership ambiguity
- Missing smart pointers
- Exception safety issues
- Thread safety violations
- Missing const correctness
- Rule of 3/5 violations

### MEDIUM (Consider)
- Non-idiomatic code patterns
- Performance inefficiencies
- Unnecessary copies
- Missing noexcept specifications
- Old-style casts (C-style)
- Magic numbers

## Automated Checks Run

```bash
# Static analysis
clang-tidy *.cpp -- -std=c++17

# Comprehensive checks
cppcheck --enable=all --std=c++17 .

# Compile with all warnings
g++ -Wall -Wextra -Wpedantic -Werror -std=c++17 *.cpp

# Format verification
clang-format --dry-run -Werror *.cpp *.hpp
```

## Example Usage

```text
User: /cpp-review

Agent:
# C++ Code Review Report

## Files Reviewed
- src/widget.cpp (modified)
- include/widget.hpp (modified)

## Static Analysis Results
- clang-tidy: 2 warnings
- cppcheck: 1 warning

## Issues Found

[CRITICAL] Memory Leak
File: src/widget.cpp:42
Issue: Raw new without corresponding delete or smart pointer
```cpp
Widget* w = new Widget();  // Leak if exception thrown
process(w);
delete w;
```
Fix: Use smart pointer
```cpp
auto w = std::make_unique<Widget>();
process(w.get());
```

[HIGH] Missing Move Semantics
File: include/widget.hpp:28
Issue: Class with resources should implement move operations
```cpp
class Buffer {
    std::unique_ptr<char[]> data_;
    // Missing: move constructor and move assignment
};
```
Fix: Add move operations or use Rule of Zero

## Summary
- CRITICAL: 1
- HIGH: 1
- MEDIUM: 0

Recommendation: Block merge until CRITICAL issue is fixed
```

## Approval Criteria

| Status | Condition |
|--------|-----------|
| Approve | No CRITICAL or HIGH issues |
| Warning | Only MEDIUM issues (merge with caution) |
| Block | CRITICAL or HIGH issues found |

## Integration with Other Commands

- Use `/cpp-build` if build errors occur
- Use `/cpp-test` for TDD workflow
- Use `/cpp-review` before committing
- Use `/code-review` for non-C++ specific concerns

## Related

- Agent: `agents/cpp-reviewer.md`
- Skills: `skills/cpp-patterns/`, `skills/cpp-testing/`
