---
name: "生成类文档"
description: "为类生成完整的文档注释"
variables:
  - name: file_path
    label: "文件路径"
    placeholder: "如：include/core/manager.h"
  - name: language
    label: "编程语言"
  - name: code
    label: "类代码"
    multiline: true
  - name: doc_style
    label: "文档风格"
    placeholder: "如：Doxygen、JSDoc、Google Style"
---

请为以下类生成文档注释：

**文件**：`{{file_path}}`
**文档风格**：{{doc_style}}

```{{language}}
{{code}}
```

---

## 类文档内容

### 类级别文档
1. **简要描述** - 类的用途
2. **详细描述** - 设计目的、使用场景
3. **职责** - 类负责什么
4. **不变式** - 类维护的约束条件
5. **线程安全** - 是否线程安全
6. **生命周期** - 对象的创建和销毁

### 成员文档
- 公有方法：完整文档
- 受保护方法：派生类使用说明
- 私有成员：可选

---

## 输出要求

### Doxygen 风格

```cpp
/**
 * @class ClassName
 * @brief 类的简要描述
 * 
 * 详细描述，包括：
 * - 类的职责
 * - 使用场景
 * - 设计考虑
 * 
 * @invariant 类不变式（始终为真的条件）
 * 
 * @par 线程安全
 * 此类是/不是线程安全的。多线程访问需要...
 * 
 * @par 使用示例
 * @code
 * ClassName obj;
 * obj.method();
 * @endcode
 * 
 * @see 相关类
 * @since 版本号
 * @author 作者
 */
class ClassName {
public:
    /**
     * @brief 构造函数
     * @param param 参数说明
     */
    ClassName(int param);
    
    /**
     * @brief 公有方法
     * ...
     */
    void publicMethod();
    
private:
    int m_value;  ///< 成员变量的简要说明
};
```

### 带文档的完整代码

```{{language}}
// 生成带文档的代码
```

---

## 安全边界

⚠️ **注意事项**：
- 类文档应解释"为什么这样设计"
- 明确线程安全和生命周期
- 说明类之间的关系
- 继承关系的注意事项
