---
name: "函数重构"
description: "重构单个函数，提高可读性和可维护性"
variables:
  - name: file_path
    label: "文件路径"
    placeholder: "如：src/utils/parser.cpp"
  - name: function_name
    label: "函数名"
  - name: language
    label: "编程语言"
  - name: code
    label: "原始代码"
    multiline: true
  - name: goal
    label: "重构目标"
    placeholder: "如：提高可读性、减少复杂度、消除重复"
---

请重构以下函数：

**文件**：`{{file_path}}`
**函数**：`{{function_name}}`
**目标**：{{goal}}

```{{language}}
{{code}}
```

---

## 重构原则

### 必须遵守
1. **行为不变** - 输入输出行为必须完全一致
2. **可测试性** - 重构后更易于单元测试
3. **渐进式** - 小步重构，每步可验证

### 重构手法

根据代码问题，考虑以下重构手法：

| 问题 | 重构手法 |
|-----|---------|
| 函数过长 | Extract Method（提取方法） |
| 参数过多 | Introduce Parameter Object |
| 嵌套过深 | Replace Nested Conditional with Guard Clauses |
| 重复代码 | Extract Method → 复用 |
| 临时变量过多 | Replace Temp with Query |
| 复杂条件 | Decompose Conditional |

---

## 输出要求

### 1. 问题分析

| 问题 | 描述 | 严重程度 |
|-----|------|---------|
| 圈复杂度高 | ... | 🟡 |
| 过长函数 | ... | 🟡 |

### 2. 重构方案

**选用的重构手法**：
- ...

**重构步骤**：
1. 第一步
2. 第二步
3. ...

### 3. 重构后代码

```{{language}}
// 重构后的完整代码
```

### 4. 改动说明

| 改动 | 原因 | 收益 |
|-----|------|-----|
| 提取 XX 函数 | 单一职责 | 可复用 |
| 重命名变量 | 语义清晰 | 可读性 |

### 5. 测试建议

重构后需要验证的测试用例：
- ...

---

## 安全边界

⚠️ **注意事项**：
- 不修改函数的公开接口（签名、返回值）
- 保持向后兼容
- 不改变异常/错误处理行为
- 不优化性能（除非是重构目标）
- 如需拆分为多个函数，新函数应为 private/static
- 每个重构步骤后都应能通过编译和测试
