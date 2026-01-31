---
name: "代码转换"
description: "将代码从一种语言/风格转换为另一种"
variables:
  - name: source_language
    label: "源语言"
    placeholder: "如：Python、Java、C"
  - name: target_language
    label: "目标语言"
    placeholder: "如：C++、Rust、Go"
  - name: code
    label: "源代码"
    multiline: true
  - name: requirements
    label: "特殊要求（可选）"
    placeholder: "如：使用现代特性、保持性能"
---

请将以下代码转换：

**源语言**：{{source_language}}
**目标语言**：{{target_language}}

**源代码**：
```{{source_language}}
{{code}}
```

**特殊要求**：{{requirements}}

---

## 转换原则

### 1. 功能等价
- 输入输出行为一致
- 边界条件处理一致
- 错误处理语义等价

### 2. 惯用写法
- 使用目标语言的惯用模式
- 利用目标语言的特性
- 遵循目标语言的最佳实践

### 3. 类型映射

常见类型映射：

| Python | C++ | Java | Go |
|--------|-----|------|-----|
| `list` | `std::vector` | `ArrayList` | `[]T` |
| `dict` | `std::map` | `HashMap` | `map[K]V` |
| `str` | `std::string` | `String` | `string` |
| `None` | `std::nullopt` | `null` | `nil` |

### 4. 错误处理映射

| 源 | 目标 |
|----|------|
| Python 异常 | C++ 异常 / 返回值 |
| Go error | C++ `std::expected` / 异常 |

---

## 输出格式

### 1. 转换分析

| 源代码特性 | 目标语言对应 | 说明 |
|-----------|-------------|------|
| ... | ... | ... |

### 2. 转换后代码

```{{target_language}}
// 转换后的完整代码
```

### 3. 差异说明

| 方面 | 源语言实现 | 目标语言实现 |
|-----|-----------|-------------|
| 内存管理 | GC | 智能指针 |
| 错误处理 | 异常 | 返回值 |

### 4. 使用示例

```{{target_language}}
// 使用示例
```

---

## 安全边界

⚠️ **注意事项**：
- 注意语言特性差异（如 Python 的动态类型）
- 内存管理差异需要特别处理
- 字符串编码可能不同
- 并发模型可能不同
- 需要充分测试验证功能等价
