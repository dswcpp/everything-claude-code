---
name: "生成变更日志"
description: "为版本发布生成规范的变更日志"
variables:
  - name: version
    label: "版本号"
    placeholder: "如：1.2.0"
  - name: prev_version
    label: "上一版本（可选）"
    placeholder: "如：1.1.0"
  - name: changes
    label: "变更内容"
    multiline: true
    placeholder: "可以是 git log、commit 列表或描述"
  - name: format
    label: "格式"
    placeholder: "如：Keep a Changelog、Conventional"
---

请为以下变更生成变更日志：

**版本**：{{version}}
**上一版本**：{{prev_version}}
**格式**：{{format}}

**变更内容**：
{{changes}}

---

## 变更日志规范

### Keep a Changelog 格式

```markdown
## [版本号] - 日期

### Added
- 新增功能

### Changed
- 变更的功能

### Deprecated
- 即将废弃的功能

### Removed
- 已移除的功能

### Fixed
- Bug 修复

### Security
- 安全相关修复
```

### Conventional Commits

| 类型 | 说明 |
|-----|------|
| feat | 新功能 |
| fix | Bug 修复 |
| docs | 文档更新 |
| style | 代码风格 |
| refactor | 重构 |
| perf | 性能优化 |
| test | 测试 |
| build | 构建系统 |
| ci | CI/CD |
| chore | 其他 |

---

## 输出格式

### 变更日志

```markdown
## [{{version}}] - YYYY-MM-DD

### 新增 (Added)
- 功能 A：简要描述 (#PR号)
- 功能 B：简要描述

### 变更 (Changed)
- 优化了 X 的性能
- 更新了 Y 的 API

### 修复 (Fixed)
- 修复了 Z 问题 (#issue号)

### 安全 (Security)
- 修复了安全漏洞 CVE-XXXX
```

### 版本摘要

一句话总结这个版本的主要变化。

### 升级指南（如有破坏性变更）

```markdown
### 破坏性变更 (Breaking Changes)

#### API 变更
旧方式：...
新方式：...

#### 迁移步骤
1. ...
2. ...
```

---

## 安全边界

⚠️ **注意事项**：
- 敏感信息（安全漏洞细节）酌情公开
- 标注破坏性变更
- 链接相关 Issue/PR
- 感谢贡献者（如适用）
