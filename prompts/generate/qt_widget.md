---
name: "生成 Qt 控件"
description: "生成 Qt 自定义控件或对话框"
variables:
  - name: header_file
    label: "头文件路径"
    placeholder: "如：src/widgets/custom_button.h"
  - name: source_file
    label: "源文件路径"
    placeholder: "如：src/widgets/custom_button.cpp"
  - name: class_name
    label: "类名"
  - name: base_class
    label: "基类"
    placeholder: "如：QWidget、QDialog、QPushButton"
  - name: description
    label: "控件描述"
    multiline: true
  - name: features
    label: "功能特性"
    multiline: true
---

请生成以下 Qt 控件：

**类名**：`{{class_name}}`
**基类**：`{{base_class}}`
**头文件**：`{{header_file}}`
**源文件**：`{{source_file}}`

### 控件描述

{{description}}

### 功能特性

{{features}}

---

## Qt 开发规范

### 类结构
```cpp
class MyWidget : public QWidget {
    Q_OBJECT
    
public:
    explicit MyWidget(QWidget* parent = nullptr);
    ~MyWidget() override;
    
    // 公有接口
    
signals:
    // 信号
    void valueChanged(int value);
    
public slots:
    // 公有槽
    void setValue(int value);
    
protected:
    // 事件处理
    void paintEvent(QPaintEvent* event) override;
    void resizeEvent(QResizeEvent* event) override;
    
private slots:
    // 私有槽
    
private:
    void setupUi();
    void connectSignals();
    
    // 成员变量
};
```

### 最佳实践
- 构造函数使用 `explicit`
- 父子关系管理内存
- 信号槽使用 Qt5 新语法
- 样式使用 QSS
- 布局使用 Layout 而非固定坐标

---

## 输出要求

### 1. 头文件

```cpp
#pragma once

#include <QWidget>

class ClassName : public BaseClass {
    Q_OBJECT
    
public:
    explicit ClassName(QWidget* parent = nullptr);
    // ...
};
```

### 2. 源文件

```cpp
// 完整实现
```

### 3. 样式表（如需要）

```css
/* QSS 样式 */
ClassName {
    /* ... */
}
```

### 4. 使用示例

```cpp
// 如何使用这个控件
```

---

## 安全边界

⚠️ **注意事项**：
- 确保信号槽连接正确断开（或使用 QObject 父子关系）
- 线程安全：UI 操作只能在主线程
- 避免在构造函数中发射信号
- 资源释放使用 Qt 父子机制
- 事件处理调用基类实现
- 考虑高 DPI 支持
