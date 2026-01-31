---
name: "Qt 代码审查"
description: "Qt/C++ 项目专项代码审查"
variables:
  - name: file_path
    label: "文件路径"
    placeholder: "如：src/widgets/mainwindow.cpp"
  - name: code
    label: "代码内容"
    multiline: true
---

请对以下 Qt 代码进行专项审查：

**文件**：`{{file_path}}`

```cpp
{{code}}
```

---

## Qt 代码审查清单

### 1. 内存管理
- [ ] QObject 父子关系是否正确设置
- [ ] 无父对象的 QObject 是否手动管理
- [ ] `deleteLater()` vs `delete` 使用是否正确
- [ ] 是否有循环引用（QObject 树）

### 2. 信号槽
- [ ] 使用 Qt5 新语法 `connect(&obj, &Class::signal, ...)`
- [ ] 连接类型是否正确（Auto/Direct/Queued）
- [ ] Lambda 捕获是否导致悬空引用
- [ ] 信号槽参数类型匹配

### 3. 线程安全
- [ ] UI 操作是否在主线程
- [ ] 跨线程通信使用 `Qt::QueuedConnection`
- [ ] QThread 使用方式是否正确
- [ ] 共享数据是否正确同步

### 4. 事件处理
- [ ] 事件处理函数是否调用基类实现
- [ ] 事件过滤器是否正确返回
- [ ] 避免在事件处理中阻塞

### 5. 资源管理
- [ ] QFile 是否正确关闭
- [ ] 网络请求是否有超时和错误处理
- [ ] 定时器是否正确停止

### 6. 界面响应性
- [ ] 长时间操作是否在后台线程
- [ ] 是否使用 `QApplication::processEvents()` 保持响应
- [ ] 避免在 UI 线程进行 I/O 操作

### 7. 跨平台
- [ ] 文件路径使用 `/` 或 `QDir::separator()`
- [ ] 字符编码使用 UTF-8
- [ ] 考虑高 DPI 显示

### 8. Qt 最佳实践
- [ ] 使用 `Q_OBJECT` 宏
- [ ] 属性使用 `Q_PROPERTY`
- [ ] 字符串使用 `tr()` 进行国际化
- [ ] 样式使用 QSS 而非硬编码

---

## 常见问题

| 问题 | 正确做法 |
|-----|---------|
| `new QWidget()` 无父对象 | 设置父对象或手动管理 |
| `connect(..., SLOT(...))` | 使用 Qt5 语法 |
| UI 线程阻塞 | 移至 QThread/QtConcurrent |
| 直接删除 QObject | 使用 `deleteLater()` |

---

## 输出要求

### 发现的问题

| 严重程度 | 类型 | 位置 | 问题 | 建议 |
|---------|------|------|------|------|
| ... | ... | ... | ... | ... |

### 改进代码

```cpp
// 改进后的代码
```

---

## 安全边界

⚠️ **注意事项**：
- Qt 版本兼容性（Qt5 vs Qt6）
- 平台特定代码需要 `#ifdef` 保护
- 第三方 Qt 库的 API 稳定性
- 考虑 Qt 模块的许可证要求
