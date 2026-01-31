---
description: Fix C++ build errors including CMake/qmake configuration, linker errors, and compilation issues. Invokes the cpp-build-resolver agent.
---

# C++ Build Fix

This command invokes the **cpp-build-resolver** agent to diagnose and fix C++ build errors.

## What This Command Does

1. **Detect Build System**: Identify CMake, qmake, or Makefile project
2. **Run Build**: Execute build and capture errors
3. **Analyze Errors**: Parse and categorize build failures
4. **Apply Fixes**: Make minimal, surgical changes
5. **Verify Build**: Confirm fixes resolve the issues

## When to Use

Use `/cpp-build` when:
- CMake configuration fails
- Compilation errors occur
- Linker errors (undefined reference)
- Missing header files
- qmake project issues
- Template instantiation errors

## Supported Build Systems

- **CMake**: `CMakeLists.txt` projects
- **qmake**: `.pro` file Qt projects
- **Makefile**: Standard make projects
- **MSBuild**: Visual Studio projects (limited)

## Common Issues Fixed

### Compilation Errors
- Missing includes
- Type mismatches
- Undefined identifiers
- Template errors
- Syntax errors

### Linker Errors
- Undefined reference
- Multiple definition
- Missing libraries
- ABI mismatches

### Configuration Errors
- CMake find_package failures
- Missing dependencies
- Wrong compiler flags
- Include path issues

## Example Usage

```text
User: /cpp-build

Agent:
# C++ Build Error Resolution

## Build System: CMake
## Build Command: cmake --build build

## Errors Found: 3

### Error 1: Undefined Reference
```
build/src/main.cpp.o: undefined reference to `Database::connect()'
```

**Diagnosis**: Missing source file in CMakeLists.txt

**Fix Applied**:
```cmake
# CMakeLists.txt
add_executable(myapp
    src/main.cpp
    src/database.cpp  # Added
)
```

### Error 2: Header Not Found
```
fatal error: nlohmann/json.hpp: No such file or directory
```

**Diagnosis**: Missing include directory

**Fix Applied**:
```cmake
find_package(nlohmann_json REQUIRED)
target_link_libraries(myapp PRIVATE nlohmann_json::nlohmann_json)
```

### Error 3: Type Mismatch
```
error: cannot convert 'std::string' to 'const char*'
```

**Fix Applied**:
```cpp
// Changed:
const char* name = getName();
// To:
std::string name = getName();
```

## Build Result: SUCCESS
- Errors Fixed: 3
- Files Modified: CMakeLists.txt, src/main.cpp
```

## Diagnostic Commands

The agent runs these checks:

```bash
# CMake projects
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build 2>&1

# qmake projects
qmake project.pro
make 2>&1

# Check dependencies
pkg-config --exists library_name
```

## Integration with Other Commands

- Use `/cpp-review` after fixing to ensure code quality
- Use `/cpp-test` to verify tests pass
- Use `/build-fix` for general build issues

## Related

- Agent: `agents/cpp-build-resolver.md`
- Skills: `skills/cpp-patterns/`
