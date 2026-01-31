---
name: "提取方法"
description: "从长函数中提取独立的方法"
variables:
  - name: file_path
    label: "文件路径"
    placeholder: "如：src/service/handler.cpp"
  - name: function_name
    label: "原函数名"
  - name: language
    label: "编程语言"
  - name: code
    label: "原始代码"
    multiline: true
  - name: extract_region
    label: "要提取的代码区域"
    placeholder: "如：行 20-35 的验证逻辑"
---

请从以下代码中提取方法：

**文件**：`{{file_path}}`
**函数**：`{{function_name}}`
**提取区域**：{{extract_region}}

```{{language}}
{{code}}
```

---

## 提取方法原则

### 何时提取
- 代码块可以独立命名
- 代码块被重复使用
- 函数过长需要分解
- 降低函数复杂度

### 提取步骤
1. 识别可提取的代码块
2. 确定需要的参数（输入）
3. 确定返回值（输出）
4. 创建新方法
5. 用新方法调用替换原代码
6. 编译测试

### 参数传递策略

| 变量用途 | 传递方式 |
|---------|---------|
| 只读输入 | `const T&` |
| 需要修改 | `T&` 或返回值 |
| 可选输入 | `std::optional` 或默认值 |
| 多个输出 | 返回 struct 或 tuple |

---

## 输出格式

### 1. 提取分析

**提取的代码块**：行号范围
**功能描述**：这段代码做什么
**依赖分析**：

| 变量 | 类型 | 用途 | 参数/返回 |
|-----|------|------|---------|
| var1 | T | 输入 | 参数 |
| var2 | T | 输出 | 返回值 |

### 2. 新方法签名

```{{language}}
/**
 * @brief 新方法的文档
 */
ReturnType extractedMethod(params);
```

### 3. 重构后代码

**新方法实现**：
```{{language}}
// 提取的新方法
```

**原函数修改后**：
```{{language}}
// 修改后的原函数
```

---

## 安全边界

⚠️ **注意事项**：
- 确保提取后功能完全等价
- 新方法命名要清晰表达意图
- 参数不宜过多（超过 4 个考虑封装）
- 考虑新方法的访问级别
- 提取后运行所有相关测试
