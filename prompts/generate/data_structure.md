---
name: "生成数据结构"
description: "设计和实现自定义数据结构"
variables:
  - name: header_file
    label: "头文件路径"
    placeholder: "如：include/ds/ring_buffer.h"
  - name: source_file
    label: "源文件路径"
    placeholder: "如：src/ds/ring_buffer.cpp"
  - name: name
    label: "数据结构名称"
  - name: description
    label: "功能描述"
    multiline: true
  - name: operations
    label: "需要支持的操作"
    multiline: true
---

请设计和实现以下数据结构：

**名称**：{{name}}
**头文件**：`{{header_file}}`
**源文件**：`{{source_file}}`

**功能描述**：
{{description}}

**需要的操作**：
{{operations}}

---

## 设计原则

### 1. 接口设计
- 简洁明了的 API
- 符合 STL 风格（如适用）
- 异常安全
- 线程安全（如需要）

### 2. 复杂度要求

| 操作 | 时间复杂度目标 |
|-----|---------------|
| 插入 | O(?) |
| 删除 | O(?) |
| 查找 | O(?) |

### 3. 内存管理
- 使用 RAII
- 考虑移动语义
- 避免内存碎片

---

## 输出格式

### 1. 设计分析

**数据结构选择**：...
**底层实现**：...
**复杂度分析**：...

### 2. 头文件

```cpp
#pragma once

/**
 * @class ClassName
 * @brief 数据结构描述
 */
template<typename T>
class ClassName {
public:
    // 类型定义
    using value_type = T;
    using size_type = std::size_t;
    
    // 构造与析构
    ClassName();
    ~ClassName();
    
    // 禁止拷贝/允许移动（按需）
    ClassName(const ClassName&) = delete;
    ClassName& operator=(const ClassName&) = delete;
    ClassName(ClassName&&) noexcept;
    ClassName& operator=(ClassName&&) noexcept;
    
    // 公有接口
    void push(const T& value);
    T pop();
    bool empty() const noexcept;
    size_type size() const noexcept;
    
private:
    // 私有成员
};
```

### 3. 源文件实现

```cpp
// 完整实现
```

### 4. 单元测试

```cpp
// 测试代码
```

### 5. 使用示例

```cpp
// 使用示例
```

---

## 安全边界

⚠️ **注意事项**：
- 考虑迭代器失效情况
- 异常抛出时保持一致性
- 边界条件处理（空、满）
- 线程安全文档化
