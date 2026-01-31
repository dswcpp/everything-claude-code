---
name: "生成 SQL 查询"
description: "根据需求生成 SQL 查询语句"
variables:
  - name: database
    label: "数据库类型"
    placeholder: "如：MySQL、PostgreSQL、SQLite"
  - name: tables
    label: "相关表结构"
    multiline: true
  - name: requirement
    label: "查询需求"
    multiline: true
---

请生成满足以下需求的 SQL 查询：

**数据库**：{{database}}

**表结构**：
```sql
{{tables}}
```

**查询需求**：
{{requirement}}

---

## SQL 编写规范

### 1. 可读性
- 关键字大写（SELECT, FROM, WHERE）
- 适当缩进和换行
- 使用有意义的别名

### 2. 性能
- 使用索引列
- 避免 SELECT *
- 合理使用 JOIN

### 3. 安全性
- 参数化查询（防 SQL 注入）
- 最小权限原则

---

## 输出格式

### 1. SQL 查询

```sql
-- 查询描述
SELECT 
    column1,
    column2
FROM table1 t1
JOIN table2 t2 ON t1.id = t2.foreign_id
WHERE condition
ORDER BY column1
LIMIT 100;
```

### 2. 查询说明

**执行逻辑**：解释查询做了什么
**使用的技术**：JOIN 类型、子查询等
**性能考虑**：建议的索引

### 3. 参数化版本

```sql
-- 用于应用程序的参数化查询
SELECT * FROM table WHERE id = ?;
```

### 4. 测试数据

```sql
-- 测试用的数据插入
INSERT INTO table VALUES (...);
```

---

## 安全边界

⚠️ **注意事项**：
- 永远使用参数化查询
- 验证用户输入
- 考虑查询超时
- 限制返回结果数量
- 敏感数据脱敏
