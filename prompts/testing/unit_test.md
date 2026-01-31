---
name: "生成单元测试"
description: "为函数或类生成完整的单元测试"
variables:
  - name: source_file
    label: "被测文件路径"
    placeholder: "如：src/utils/calculator.cpp"
  - name: test_file
    label: "测试文件路径"
    placeholder: "如：tests/test_calculator.cpp"
  - name: language
    label: "编程语言"
  - name: code
    label: "被测代码"
    multiline: true
  - name: framework
    label: "测试框架"
    placeholder: "如：Google Test、Catch2、pytest、Jest"
---

请为以下代码生成单元测试：

**被测文件**：`{{source_file}}`
**测试文件**：`{{test_file}}`
**测试框架**：{{framework}}

```{{language}}
{{code}}
```

---

## 测试设计原则

### FIRST 原则
- **F**ast：测试运行快速
- **I**ndependent：测试间相互独立
- **R**epeatable：可重复执行
- **S**elf-validating：自动验证通过/失败
- **T**imely：及时编写

### 测试金字塔
```
         /\
        /  \  E2E（少量）
       /────\
      / 集成  \  Integration
     /──────────\
    /   单元测试   \  Unit（大量）
   /────────────────\
```

---

## 测试覆盖要求

### 1. 正常路径（Happy Path）
- 标准输入的预期输出
- 常见使用场景

### 2. 边界条件
- 空值/null/nullptr
- 空容器/空字符串
- 零值
- 最大值/最小值
- 边界附近值（off-by-one）

### 3. 异常路径
- 无效输入
- 错误状态
- 异常抛出验证

### 4. 等价类划分
将输入分组，每组选代表测试

---

## 输出要求

### 1. 测试用例设计

| 测试名称 | 测试类型 | 输入 | 预期输出 | 说明 |
|---------|---------|------|---------|------|
| test_xxx | 正常 | ... | ... | ... |
| test_xxx_empty | 边界 | [] | ... | 空输入 |
| test_xxx_invalid | 异常 | -1 | 抛异常 | 无效输入 |

### 2. 完整测试代码

```{{language}}
// 测试文件完整代码
```

### 3. 测试命名规范

使用以下模式之一：
- `test_<method>_<scenario>_<expected>`
- `should_<expected>_when_<condition>`
- Given-When-Then 风格

### 4. Mock/Stub（如需要）

如果被测代码有外部依赖，提供 Mock 实现。

### 5. 覆盖率说明

| 覆盖类型 | 预计覆盖率 |
|---------|-----------|
| 行覆盖 | xx% |
| 分支覆盖 | xx% |

---

## 测试代码规范

```cpp
// Google Test 示例结构
class CalculatorTest : public ::testing::Test {
protected:
    void SetUp() override {
        // 每个测试前执行
    }
    
    void TearDown() override {
        // 每个测试后执行
    }
    
    Calculator calc_;  // 被测对象
};

TEST_F(CalculatorTest, Add_TwoPositiveNumbers_ReturnsSum) {
    // Arrange
    int a = 2, b = 3;
    
    // Act
    int result = calc_.Add(a, b);
    
    // Assert
    EXPECT_EQ(result, 5);
}
```

---

## 安全边界

⚠️ **注意事项**：
- 测试不应依赖执行顺序
- 测试不应依赖外部资源（网络、文件系统）
- 使用 Mock 隔离外部依赖
- 测试数据应自包含，不依赖生产数据
- 避免测试代码中的逻辑（if/loop）
- 每个测试只验证一个行为
