---
name: "生成 README"
description: "为项目生成完整的 README 文档"
variables:
  - name: project_name
    label: "项目名称"
  - name: description
    label: "项目描述"
    multiline: true
  - name: language
    label: "主要语言"
  - name: features
    label: "主要功能"
    multiline: true
---

请为以下项目生成 README：

**项目名称**：{{project_name}}
**主要语言**：{{language}}

**项目描述**：
{{description}}

**主要功能**：
{{features}}

---

## README 结构

### 必需章节
1. **项目名称和简介**
2. **功能特性**
3. **安装说明**
4. **使用方法**
5. **许可证**

### 推荐章节
- 徽章（CI 状态、版本、许可证）
- 截图/演示
- 配置说明
- API 文档
- 贡献指南
- 常见问题

---

## 输出格式

```markdown
# {{project_name}}

![Build Status](badge)
![License](badge)

简短的一句话介绍。

## 特性

- 特性 1
- 特性 2
- 特性 3

## 安装

### 前置要求

- 依赖 1
- 依赖 2

### 安装步骤

```bash
# 安装命令
```

## 快速开始

```语言
// 基本使用示例
```

## 使用说明

### 基本用法

...

### 高级用法

...

## 配置

| 配置项 | 说明 | 默认值 |
|-------|------|-------|
| ... | ... | ... |

## API 参考

链接到详细 API 文档。

## 贡献

欢迎贡献！请阅读 [贡献指南](CONTRIBUTING.md)。

## 许可证

本项目采用 [LICENSE] 许可证。

## 致谢

- 致谢项目/人员
```

---

## 安全边界

⚠️ **注意事项**：
- 确保安装说明准确可执行
- 示例代码应能正常运行
- 保持文档与代码同步
- 敏感配置使用环境变量
