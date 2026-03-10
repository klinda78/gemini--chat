❗补充说明

文档里有一项：

Native skill commands can be exposed as slash commands; unauthorized senders see them as plain text.

这意味着：

如果你有一个 skill 定义得足够标准

OpenClaw 会在支持的渠道上注册它作为 slash command

用户可以通过 /skillName 在聊天界面执行它

但这依然是 聊天触发的 skill 执行，而不是 workflow 步骤定义里的 command:

📌 简单总结

✔ Slash Command 是聊天层命令，用于即时执行操作。
❌ Slash Command 不能直接放到 lobster workflow 的步骤里。
✔ Workflow 步骤只能调用实际的 CLI / script / tool

 ## 正确的调用skill的方法可以是：
 - 1.ui界面直接 /skill <skill name>
 - 2.workflow调用一个 Skill 可以通过 llm-task 或 command 步骤调用 Skill CLI 来实现，而不是直接写command /skill

### 1. Skill 的调用方式 EXAMPLE
-- 方法 A：Skill CLI

如果你已经把 Python/Node 脚本包装成 OpenClaw Skill CLI，比如：
```code
bash
openclaw skills run github-top-rank --output - 
```
那么在 Lobster workflow 里可以这样写：
```YAML
steps:
  - id: get_top10
    command: openclaw skills run github-top-rank --output -
```


输出默认是 JSON（如果 Skill 支持 --json 或 --output -）

后续步骤可以直接通过 stdin: $get_top10.stdout 使用

-- 方法 B：llm-task 调用 Skill

llm-task 不直接调用 Slash Command，而是可以通过 Skill CLI 执行外部动作，然后把结果纳入 JSON 流。

例如：
```YAML
steps:
  - id: top10_llm
    command: llm-task
    args:
      input: $get_top10.stdout   # JSON 输入
      schema: |
        {
          "type": "object",
          "properties": {
            "summary": {"type": "string"},
            "markdown": {"type": "string"}
          },
          "required": ["summary", "markdown"]
        }
```
llm-task 输入 JSON（可以是前一步 Skill CLI 输出）

输出 JSON（包含 summary 和 markdown）

下一步可以直接用 jq 或者 message CLI 发消息

## 2. 数据流示意
```flowchart
Skill CLI (python/github-top-rank)
      ↓ JSON
llm-task (处理 JSON，生成 summary/markdown)
      ↓ JSON
jq -r '.markdown' (提取文本)
      ↓ Markdown
openclaw message send --target feishu
```
这里你不需要 Slash Command

Skill CLI + llm-task 完全可以在 workflow 内完成自动化

## 3. workflow 示例片段
name: github-top10-daily
```YAML
steps:
  - id: fetch
    command: openclaw skills run github-top-rank --output -
    
  - id: process
    command: llm-task
    args:
      input: $fetch.stdout
      schema: |
        {
          "type": "object",
          "properties": {
            "summary": {"type": "string"},
            "markdown": {"type": "string"}
          },
          "required": ["summary", "markdown"]
        }

  - id: format
    command: jq -r '.markdown'
    stdin: $process.stdout

  - id: notify
    command: openclaw message send --target feishu
    stdin: $format.stdout

```
🔑 核心点

不使用 /skill Slash Command，workflow 里只能调用 CLI 或可执行工具

llm-task 可以作为 Skill 的 JSON 处理层，生成结构化输出

数据流通过 stdout / stdin 传递，不依赖聊天触发   
 -   
 
