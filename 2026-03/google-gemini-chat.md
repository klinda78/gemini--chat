# 🚀 OpenClaw 工作流架构笔记
> **日期:** 2026-03-09
> **摘要:** 本笔记记录了 OpenClaw 的核心执行逻辑，包含从 Cron 触发到消息输出的全流程。

---

## 🏗 架构拓扑图
以下是使用 Mermaid 渲染的系统层级图：

```mermaid
flowchart TD

%% 定义美化样式
classDef trigger fill:#f96,stroke:#333,stroke-width:2px;
classDef agent fill:#bbf,stroke:#333,stroke-width:2px;
classDef workflow fill:#dfd,stroke:#333,stroke-width:2px;
classDef command fill:#fff4dd,stroke:#333,stroke-width:2px;
classDef output fill:#fdd,stroke:#333,stroke-width:2px;

subgraph Trigger [触发层]
    CRON[Cron Trigger]:::trigger
end

subgraph Agent_Layer [代理层]
    AGENT[Agent]:::agent
    TASK[Task]:::agent
end

subgraph Workflow_Layer [工作流层]
    WORKFLOW[Workflow]:::workflow
end

subgraph Command_Layer [执行层]
    SKILL[openclaw skills run]:::command
    LLM[llm-task]:::command
    CLI[openclaw CLI tools]:::command
end

subgraph Output_Layer [输出层]
    PARSER[jq parser]:::output
    MSG[message send]:::output
end

%% 连接逻辑
CRON --> AGENT --> TASK --> WORKFLOW
WORKFLOW --> SKILL
WORKFLOW --> LLM
WORKFLOW --> CLI

SKILL --> LLM
LLM --> PARSER --> MSG