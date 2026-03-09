## Quick Reference lobster pipeline
### 什么是pipeline
- 在 OpenClaw 和 Lobster 中，Pipeline 通常用来定义 复杂的工作流，使得多个工具和任务按预定顺序执行。
关键概念：

- Pipeline 运行模式：$LOBSTER 提供了两种运行模式：

1. Human Mode：以人类可读的方式输出结果，适合展示。

2. Tool Mode：以机器可解析的 JSON 格式输出结果，适合集成到其他系统。

- Pipeline 中的数据流
你可以通过 stdin 和 stdout 来在不同的步骤之间传递数据。例如，stdin: $fetch-data.stdout 表示 process-data 步骤接收 fetch-data 步骤的输出。

- Pipeline 的用途

自动化任务：像数据抓取、处理和报告生成这样的多步骤任务可以通过 pipeline 自动化。

工作流编排：将多个工具的调用和逻辑串联起来，确保每个步骤都按顺序执行，并可以实现复杂的业务逻辑
