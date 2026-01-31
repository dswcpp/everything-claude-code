---
name: "生成函数"
description: "根据需求生成高质量函数实现"
variables:
  - name: file_path
    label: "目标文件路径"
    placeholder: "如：src/utils/helper.cpp"
  - name: function_name
    label: "函数名"
  - name: language
    label: "编程语言"
  - name: description
    label: "功能描述"
    multiline: true
  - name: signature
    label: "函数签名（可选）"
    placeholder: "如：int calculate(const std::vector<int>& data)"
  - name: constraints
    label: "约束条件（可选）"
    placeholder: "如：O(n)时间复杂度、线程安全"
---

请生成以下函数：

**文件**：`{{file_path}}`
**函数名**：`{{function_name}}`
**语言**：{{language}}

### 功能描述

{{description}}

### 函数签名（如有）

```{{language}}
{{signature}}
```

### 约束条件

{{constraints}}

---

## 代码质量要求

### 1. 正确性
- 实现必须符合功能描述
- 处理所有边界条件
- 正确的错误处理

### 2. 可读性
- 清晰的变量命名
- 适当的注释
- 逻辑清晰，避免过深嵌套

### 3. 健壮性
- 输入验证
- 空值/边界检查
- 异常安全

### 4. 性能
- 合理的时间复杂度
- 避免不必要的拷贝
- 资源正确释放

---

## 输出要求

### 1. 函数声明（头文件）

```{{language}}
/**
 * @brief 简要描述
 * @param param1 参数说明
 * @return 返回值说明
 * @throws 可能抛出的异常
 */
ReturnType functionName(params);
```

### 2. 函数实现

```{{language}}
// 完整实现
```

### 3. 使用示例

```{{language}}
// 使用示例代码
```

### 4. 单元测试

```{{language}}
// 基本测试用例
```

---

## 安全边界

⚠️ **注意事项**：
- 遵循项目现有的代码风格
- 不引入新的外部依赖（除非必要）
- 考虑与现有代码的集成
- 避免硬编码魔法数字
- 敏感操作需要日志记录
