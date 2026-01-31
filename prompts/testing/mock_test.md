---
name: "Mock 测试"
description: "使用 Mock 对象进行隔离测试"
variables:
  - name: source_file
    label: "被测文件路径"
    placeholder: "如：src/service/user_service.cpp"
  - name: language
    label: "编程语言"
  - name: code
    label: "被测代码"
    multiline: true
  - name: dependencies
    label: "需要 Mock 的依赖"
    placeholder: "如：DatabaseClient、HttpClient"
  - name: framework
    label: "Mock 框架"
    placeholder: "如：Google Mock、Mockito、unittest.mock"
---

请为以下代码生成使用 Mock 的单元测试：

**文件**：`{{source_file}}`
**Mock 框架**：{{framework}}
**需要 Mock 的依赖**：{{dependencies}}

```{{language}}
{{code}}
```

---

## Mock 设计原则

### 何时使用 Mock
- 外部服务（API、数据库）
- 文件系统、网络
- 时间相关（时钟）
- 随机数生成
- 难以创建的复杂对象

### Mock vs Stub vs Fake

| 类型 | 用途 | 验证 |
|-----|------|------|
| **Mock** | 验证交互 | 验证调用次数、参数 |
| **Stub** | 返回预设值 | 只关心返回值 |
| **Fake** | 简化实现 | 内存数据库等 |

---

## 输出要求

### 1. 依赖分析

| 依赖 | 类型 | Mock 策略 |
|-----|------|----------|
| DatabaseClient | 外部服务 | Mock 接口 |
| Logger | 副作用 | Stub |

### 2. Mock 接口定义

```{{language}}
// Mock 类定义
class MockDatabaseClient : public IDatabaseClient {
public:
    MOCK_METHOD(Result, Query, (const std::string& sql), (override));
    MOCK_METHOD(bool, Connect, (), (override));
};
```

### 3. 测试代码

```{{language}}
// 使用 Mock 的测试
TEST_F(UserServiceTest, GetUser_ValidId_ReturnsUser) {
    // Arrange
    MockDatabaseClient mockDb;
    EXPECT_CALL(mockDb, Query(_))
        .WillOnce(Return(/* 预设结果 */));
    
    UserService service(&mockDb);
    
    // Act
    auto user = service.GetUser(123);
    
    // Assert
    EXPECT_EQ(user.name, "Alice");
}
```

### 4. Mock 行为设置

常用 Mock 设置：
```cpp
// 设置返回值
EXPECT_CALL(mock, Method(_)).WillOnce(Return(value));

// 设置多次调用
EXPECT_CALL(mock, Method(_))
    .WillOnce(Return(v1))
    .WillOnce(Return(v2))
    .WillRepeatedly(Return(v3));

// 验证调用次数
EXPECT_CALL(mock, Method(_)).Times(3);

// 抛出异常
EXPECT_CALL(mock, Method(_)).WillOnce(Throw(std::exception()));
```

---

## 安全边界

⚠️ **注意事项**：
- Mock 应该模拟接口，不是具体实现
- 过度 Mock 会导致测试脆弱
- Mock 不能替代集成测试
- 确保 Mock 的行为与真实对象一致
- 不要 Mock 你不拥有的类型（除非必要）
