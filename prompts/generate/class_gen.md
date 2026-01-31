---
name: "生成类"
description: "根据需求生成完整的类定义和实现"
variables:
  - name: header_file
    label: "头文件路径"
    placeholder: "如：include/models/user.h"
  - name: source_file
    label: "源文件路径"
    placeholder: "如：src/models/user.cpp"
  - name: class_name
    label: "类名"
  - name: language
    label: "编程语言"
  - name: description
    label: "类的职责描述"
    multiline: true
  - name: requirements
    label: "具体要求"
    multiline: true
---

请生成以下类：

**类名**：`{{class_name}}`
**头文件**：`{{header_file}}`
**源文件**：`{{source_file}}`
**语言**：{{language}}

### 类的职责

{{description}}

### 具体要求

{{requirements}}

---

## 类设计原则

### SOLID 原则
- **S**ingle Responsibility：一个类只做一件事
- **O**pen/Closed：对扩展开放，对修改关闭
- **L**iskov Substitution：子类可替换父类
- **I**nterface Segregation：接口要精简
- **D**ependency Inversion：依赖抽象

### C++ 类设计规范

#### 特殊成员函数（Rule of 0/3/5）
```cpp
class MyClass {
public:
    // Rule of 5（如需自定义资源管理）
    MyClass();                              // 默认构造
    ~MyClass();                             // 析构
    MyClass(const MyClass&);                // 拷贝构造
    MyClass& operator=(const MyClass&);     // 拷贝赋值
    MyClass(MyClass&&) noexcept;            // 移动构造
    MyClass& operator=(MyClass&&) noexcept; // 移动赋值
};
```

#### 成员顺序
1. public 接口
2. protected 成员
3. private 成员
4. 成员变量按大小排列（减少 padding）

---

## 输出要求

### 1. 头文件

```{{language}}
#pragma once

/**
 * @class ClassName
 * @brief 类的简要描述
 * 
 * 详细描述...
 */
class ClassName {
public:
    // 构造与析构
    
    // 公有接口
    
private:
    // 私有成员
};
```

### 2. 源文件

```{{language}}
// 完整实现
```

### 3. 使用示例

```{{language}}
// 如何使用这个类
```

### 4. 单元测试框架

```{{language}}
// 测试基本功能
```

---

## 安全边界

⚠️ **注意事项**：
- 遵循项目命名规范
- 考虑线程安全需求
- 避免循环依赖
- 正确的 include guard / pragma once
- 前向声明减少编译依赖
- 考虑是否需要禁止拷贝/移动
