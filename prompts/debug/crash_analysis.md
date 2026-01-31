---
name: "崩溃分析"
description: "分析程序崩溃原因并提供修复"
variables:
  - name: file_path
    label: "崩溃位置文件"
    placeholder: "如：src/core/engine.cpp"
  - name: language
    label: "编程语言"
  - name: code
    label: "相关代码"
    multiline: true
  - name: stack_trace
    label: "堆栈跟踪"
    multiline: true
  - name: crash_type
    label: "崩溃类型"
    placeholder: "如：Segfault、SIGABRT、未处理异常"
---

程序发生崩溃，请帮我分析：

**文件**：`{{file_path}}`
**崩溃类型**：{{crash_type}}

### 堆栈跟踪

```
{{stack_trace}}
```

### 相关代码

```{{language}}
{{code}}
```

---

## 分析要点

### 1. 崩溃类型诊断

| 崩溃信号 | 常见原因 |
|---------|---------|
| SIGSEGV | 空指针、野指针、栈溢出 |
| SIGABRT | assert 失败、double-free |
| SIGFPE | 除零、整数溢出 |
| SIGBUS | 内存对齐错误 |
| Stack Overflow | 无限递归、栈上大数组 |

### 2. 堆栈分析
- 崩溃点在哪个函数
- 调用链是什么
- 传入参数是否合法

### 3. 内存问题排查（如适用）
- 空指针解引用
- 悬空指针/Use-after-free
- 缓冲区越界
- 栈溢出

### 4. 根本原因
- 直接触发原因
- 深层原因（为什么会出现这种状态）

### 5. 修复方案

```{{language}}
// 修复后的代码
```

### 6. 防御性编程建议
- 添加必要的空指针检查
- 使用智能指针
- 增加断言
- 输入验证

### 7. 测试用例
- 复现崩溃的测试
- 修复验证的测试

---

## 调试工具建议

| 问题类型 | 推荐工具 |
|---------|---------|
| 内存错误 | Valgrind、ASan |
| 线程问题 | TSan、Helgrind |
| 堆栈分析 | GDB、LLDB |
| 核心转储 | `coredumpctl`、WinDbg |

---

## 安全边界

⚠️ **注意事项**：
- 崩溃可能是安全漏洞的表现，需评估是否可被利用
- 修复应防止同类问题再次发生
- 不要仅仅抑制错误（如空 catch），要真正修复
- 考虑是否需要添加监控和告警
