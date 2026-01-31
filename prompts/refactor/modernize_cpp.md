---
name: "现代化 C++ 重构"
description: "将旧式 C++ 代码现代化为 C++17/20 风格"
variables:
  - name: file_path
    label: "文件路径"
    placeholder: "如：src/legacy/handler.cpp"
  - name: code
    label: "原始代码"
    multiline: true
  - name: target_standard
    label: "目标标准"
    placeholder: "如：C++17、C++20"
---

请将以下代码现代化为 {{target_standard}}：

**文件**：`{{file_path}}`

```cpp
{{code}}
```

---

## 现代化改造清单

### 1. 智能指针
| 旧写法 | 现代写法 |
|-------|---------|
| `new T()` | `std::make_unique<T>()` |
| `T* ptr` 持有所有权 | `std::unique_ptr<T>` |
| 共享所有权 | `std::shared_ptr<T>` |
| `delete ptr` | 自动管理 |

### 2. 类型推导
| 旧写法 | 现代写法 |
|-------|---------|
| `std::vector<int>::iterator it = ...` | `auto it = ...` |
| `typedef` | `using` 别名 |
| `NULL` | `nullptr` |

### 3. 范围 for 循环
```cpp
// 旧
for (int i = 0; i < vec.size(); ++i) { ... }

// 新
for (const auto& item : vec) { ... }
```

### 4. 初始化
| 旧写法 | 现代写法 |
|-------|---------|
| `T t = T()` | `T t{}` 统一初始化 |
| 成员初始化列表 | 类内初始化 |

### 5. 标准库更新
| 旧写法 | 现代写法 |
|-------|---------|
| `std::bind` | Lambda 表达式 |
| `boost::optional` | `std::optional` (C++17) |
| `boost::variant` | `std::variant` (C++17) |
| 手写 SFINAE | `if constexpr` / Concepts (C++20) |

### 6. 并发
| 旧写法 | 现代写法 |
|-------|---------|
| pthread | `std::thread` |
| 手动锁 | `std::lock_guard` / `std::scoped_lock` |
| 条件变量 | `std::condition_variable` |

### 7. 其他改进
- `[[nodiscard]]`、`[[maybe_unused]]` 属性
- 结构化绑定 `auto [a, b] = ...`
- `std::string_view` 替代 `const std::string&`
- `constexpr` 编译期计算

---

## 输出要求

### 1. 改造点列表

| 行号 | 旧写法 | 新写法 | 理由 |
|-----|-------|-------|------|
| 15 | `new Widget()` | `make_unique<Widget>()` | 内存安全 |

### 2. 现代化后代码

```cpp
// 完整的现代化代码
```

### 3. 编译要求

- 需要的编译器版本
- 编译选项（如 `-std=c++17`）

---

## 安全边界

⚠️ **注意事项**：
- 保持 ABI 兼容性（如果是库代码）
- 注意智能指针与 C API 的交互
- `auto` 不应降低代码可读性
- 避免过度使用现代特性导致代码晦涩
- 确认目标编译器支持所用特性
- 渐进式改造，不要一次改动太多
