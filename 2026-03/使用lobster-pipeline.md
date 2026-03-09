## Quick Reference lobster pipeline
### 什么是pipeline
- 在 OpenClaw 和 Lobster 中，Pipeline 通常用来定义 复杂的工作流，使得多个工具和任务按预定顺序执行。
- 它的多个步骤可以是不同的工具调用、任务执行、数据处理或其他操作。
### 关键概念：

**1. Pipeline 运行模式**：$LOBSTER 提供了两种运行模式：

- Human Mode：以人类可读的方式输出结果，适合展示。

- Tool Mode：以机器可解析的 JSON 格式输出结果，适合集成到其他系统。

**2. Pipeline 中的数据流**
- 你可以通过 stdin 和 stdout 来在不同的步骤之间传递数据。例如，stdin: $fetch-data.stdout 表示 process-data 步骤接收 fetch-data 步骤的输出。

**3. Pipeline 的用途**

- 自动化任务：像数据抓取、处理和报告生成这样的多步骤任务可以通过 pipeline 自动化。

- 工作流编排：将多个工具的调用和逻辑串联起来，确保每个步骤都按顺序执行，并可以实现复杂的业务逻辑


### 一个例子
```YAML

name: data-processing-pipeline

steps:
  - id: fetch-data
    command: curl -s https://example.com/api/data

  - id: process-data
    command: python process_data.py
    stdin: $fetch-data.stdout

  - id: store-data
    command: python store_data.py
    stdin: $process-data.stdout

  - id: notify
    command: send-notification --message "Data processed"
```

### pipeline 如何生成&& 加载位置
- 位置 通常位于workspace/.opeanclaw/workflow/

**1.如何调用skills**
如果你在 .openclaw/wrokspace/
有目录如下
```tree
skills/
└── skill-nam/
        ├── SKILL.MD
        └── scripts/
              └── git_top_rank.py
```
- 当前情况：
你在 skills/SKILL.MD 里写了这个py script 的使用说明。
```cmd
   python scripts/git_top_rank.py --output filename 
```
来执行
- skill 的本质是把某个脚本或工具包装成 agent 可调用的 命令
- 你希望在 workflow 中“引用这个 skill”来执行脚本，而不是直接写命令行
  那么工作流 YAML 文档 github-top-rank.lobster 如下
```YAML
name: github-top-rank

steps:
  - id: fetch-top
    command: openclaw skills run skill-name --output outpt/github_top_$(date +%Y-%m-%d).json
    description: "Fetch top 10 GitHub repos using git-top-rank skill"

  - id: append-time
    command: echo "$(date +'%Y-%m-%d %H:%M:%S')" >> outpt/github_top_$(date +%Y-%m-%d).json

  - id: send-channel
    command: openclaw channel send --channel <your_channel_id> --message "$(cat outpt/github_top_$(date +%Y-%m-%d).json)"

```

**2. workflow 调用 skill**
- skill 和 workflow 是解耦的，你可以随时替换脚本实现，而 workflow 不变
- workflow 不直接调用 python 脚本，而是调用 skill 的 CLI 命令
- MD 文件只是说明 skill 的用法，不参与执行。
