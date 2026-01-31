---
name: "类重构"
description: "重构类结构，改善设计和可维护性"
variables:
  - name: header_file
    label: "头文件路径"
    placeholder: "如：include/core/processor.h"
  - name: source_file
    label: "源文件路径"
    placeholder: "如：src/core/processor.cpp"
  - name: class_name
    label: "类名"
  - name: header_code
    label: "头文件代码"
    multiline: true
  - name: source_code
    label: "源文件代码"
    multiline: true
  - name: goal
    label: "重构目标"
    placeholder: "如：解耦、职责分离、改善接口"
---

请重构以下类：

**类名**：`{{class_name}}`
**目标**：{{goal}}

### 头文件 `{{header_file}}`

```cpp
{{header_code}}
```

### 源文件 `{{source_file}}`

```cpp
{{source_code}}
```

---

## 重构原则

### SOLID 原则检查
- [ ] **S**ingle Responsibility：类是否只有一个变化原因
- [ ] **O**pen/Closed：是否对扩展开放、修改关闭
- [ ] **L**iskov Substitution：继承是否正确使用
- [ ] **I**nterface Segregation：接口是否精简
- [ ] **D**ependency Inversion：是否依赖抽象

### 类重构手法

| 问题 | 重构手法 |
|-----|---------|
| 类过大 | Extract Class（提取类） |
| 职责混乱 | Move Method/Field |
| 继承滥用 | Replace Inheritance with Delegation |
| 接口臃肿 | Extract Interface |
| 紧耦合 | Introduce Parameter / Dependency Injection |

---

## 输出要求

### 1. 设计问题分析

| 问题 | 描述 | 违反原则 |
|-----|------|---------|
| ... | ... | SRP |

### 2. 重构方案

**架构变化**：
```
重构前：
  ClassA ──────────► ClassB

重构后：
  ClassA ──► Interface ◄── ClassB
```

**重构步骤**：
1. ...
2. ...

### 3. 重构后代码

**头文件**：
```cpp
// 重构后的头文件
```

**源文件**：
```cpp
// 重构后的源文件
```

**新增文件**（如有）：
```cpp
// 新提取的类
```

### 4. 接口变更

| 原接口 | 新接口 | 兼容性 |
|-------|-------|-------|
| `old()` | `new()` | 需迁移 |

### 5. 迁移指南

使用新接口的示例代码。

---

## 安全边界

⚠️ **注意事项**：
- 保持公有 API 的兼容性（或提供迁移方案）
- 不改变类的外部行为
- 考虑序列化兼容性（如果类需要持久化）
- 注意线程安全性不能降低
- 新增的依赖应通过构造函数注入
- 虚函数的修改要考虑派生类影响
