---
name: "生成函数文档"
description: "为函数生成完整的文档注释"
variables:
  - name: file_path
    label: "文件路径"
    placeholder: "如：src/core/algorithm.cpp"
  - name: language
    label: "编程语言"
  - name: code
    label: "函数代码"
    multiline: true
  - name: doc_style
    label: "文档风格"
    placeholder: "如：Doxygen、JSDoc、Google Style、NumPy Style"
---

请为以下函数生成文档注释：

**文件**：`{{file_path}}`
**文档风格**：{{doc_style}}

```{{language}}
{{code}}
```

---

## 文档内容要求

### 必需内容
1. **简要描述** - 一句话说明函数做什么
2. **参数说明** - 每个参数的含义、类型、有效范围
3. **返回值** - 返回值的含义
4. **异常/错误** - 可能抛出的异常或错误码

### 可选内容
- 详细描述（复杂算法）
- 前置条件
- 后置条件
- 复杂度说明
- 使用示例
- 相关函数
- 注意事项
- 版本历史

---

## 输出要求

### Doxygen 风格（C/C++）

```cpp
/**
 * @brief 简要描述
 * 
 * 详细描述（可选）
 * 
 * @param[in] param1 输入参数说明
 * @param[out] param2 输出参数说明
 * @param[in,out] param3 输入输出参数
 * 
 * @return 返回值说明
 * 
 * @throws std::invalid_argument 参数无效时
 * @throws std::runtime_error 运行时错误
 * 
 * @pre 前置条件
 * @post 后置条件
 * 
 * @note 注意事项
 * @warning 警告信息
 * 
 * @code
 * // 使用示例
 * auto result = function(arg1, arg2);
 * @endcode
 * 
 * @see 相关函数
 * @since 版本号
 */
```

### 带文档的完整代码

```{{language}}
// 生成带文档注释的代码
```

---

## 文档风格对照

| Doxygen | JSDoc | Python |
|---------|-------|--------|
| @brief | @description | Summary line |
| @param | @param | :param: |
| @return | @returns | :returns: |
| @throws | @throws | :raises: |

---

## 安全边界

⚠️ **注意事项**：
- 文档应与代码保持同步
- 不要复述代码能直接看出的内容
- 说明"为什么"比"做什么"更重要
- 参数的有效范围和边界条件很重要
- 副作用必须明确说明
