---
title: Git提交规范
tags:
  - git
abbrlink: d1a06c76
date: 2024-09-25 16:39:34
---

## Git 提交规范

​	Git 提交消息应遵循一定的规范，以便于理解和维护。一种常用的规范是 Angular 规范，它要求提交消息包含三个部分：**标题**、**主题**和**尾部**。

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

- **type**：提交类型，比如：feat(新功能)、fix(修复)、docs(文档变更)、style(格式变更)、refactor(重构)、chore(构建过程或辅助工具的变更) 等
- **scope**：变更影响的范围，比如模块名、组件名、禅道ID 等
- **subject**：简明描述，不超过 50 个字符，不要结束于句号
- **body**：详细描述此次提交的变更内容，可以包含多个段落
- **footer**：关闭的 issue 列表、break change 说明等



**type 说明提交类型**：只允许使用下面属性

| 属性            | 描述                               |
| --------------- | ---------------------------------- |
| feature \| feat | 新功能                             |
| fix             | 修复                               |
| docs            | 文档修改                           |
| style           | 格式修改                           |
| refactor        | 重构                               |
| perf            | 性能提升                           |
| test            | 测试                               |
| build           | 构建系统                           |
| ci              | 对 CI 配置文件修改                 |
| chore           | 修改构建流程、或者增加依赖库、工具 |
| revert          | 回滚版本                           |

