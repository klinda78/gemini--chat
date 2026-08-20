---
id: codex-task-permission-precedence
title: Codex 任务权限优先级
type: diagram
primary_topic: Codex 当前任务的权限来源与写入边界
related_topics:
  - Codex sandbox
  - writable roots
  - project trust
source_ref: current Codex task, permission-boundary discussion
captured_at: 2026-08-08
status: confirmed
---

# Codex 任务权限优先级

```text
Codex App 下发的当前任务权限
        │
        │ 覆盖
        ▼
项目或工作区权限
        │
        │ 覆盖
        ▼
%USERPROFILE%\.codex\config.toml 默认设置
```

解释：

- 当前任务的托管权限决定实际 `writable_roots` 和是否允许申请沙箱外执行。
- `trust_level = "trusted"` 表示项目配置可信，不代表整个用户目录可写。
- 全局 `sandbox = "elevated"` 是默认设置，不能覆盖任务启动时下发的更高层权限。
