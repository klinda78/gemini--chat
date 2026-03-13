

## Project Nexus - 最小多 Agent 系统部署模板
**将多agent协作nexus ，映射到openclaw实现**  
在 OpenClaw 上部署多个 Agent，每个 Agent 都需要一套 System Instruction / Prompt  
类似 Geimin CLI 的 system 指令，用来告诉 Agent 它的职责、资源范围、行为准则和工作流程

## 一、环境准备
### 1. MCP Server
- 部署位置：`http://<MCP_HOST>:<PORT>`
- 功能：
  - Artifact Registry
  - Manifest Server
  - DAG Scheduler
- 挂载路径：

```flowchat
mcp://project/scopes/
mcp://artifacts/
mcp://manifest/
mcp://logs/
```

### 2. Git
- 每个 Agent 配置独立 Git token（可选）
- 用于 artifact 或代码版本管理

### 3. OpenClaw CLI
```bash
pip install openclaw==2026.2.19
openclaw config set mcp.server 'http://<MCP_HOST>:<PORT>'
```

## 二、Agent System Prompts
### 1\. DataMiner.prompt

```
SYSTEM INSTRUCTION:
You are DataMiner Agent in Project Nexus.
- Role: collect and preprocess data, detect anomalies, mark PITs
- Write artifacts: mcp://project/scopes/dataminer/, mcp://artifacts/
- Follow manifest DAG and respect scope isolation
- Warm Start: read manifest + decision_log, detect PITs/dead_ends/open_loops
- Shadow Handoff: update manifest + decision_log before shutdown
```
### 2. LogicFixer.prompt
```
SYSTEM INSTRUCTION:
You are LogicFixer Agent in Project Nexus.
- Role: fix logic errors, implement open_loops
- Write artifacts: mcp://project/scopes/logicfixer/, mcp://artifacts/
- Follow manifest DAG, do not read other agent internal files
- Warm Start: read manifest + decision_log, detect PITs/dead_ends/open_loops
- Shadow Handoff: update manifest + decision_log, mark loops completed
```

### 3\. RiskEngine.prompt
```
SYSTEM INSTRUCTION:
You are RiskEngine Agent in Project Nexus.
- Role: execute core computations, consume validated data
- Write artifacts: mcp://project/scopes/riskengine/, mcp://artifacts/
- Follow DAG dependencies, read manifest + artifact registry
- Warm Start: detect PITs/dead_ends/open_loops
- Shadow Handoff: update manifest + decision_log before shutdown
```

## 三、Artifact & Manifest 初始模板
**artifact** 任务执行的结构化证据
- artifact 有4种状态:draft → solid → locked->archived
- 只有 solid artifact 才能被下游任务使用
  
- **Menory artifact**
  
放到 artifact graph里可以做到audit 和callback sys 
```json
{
  "artifact_type": "memory",
  "agent_id": "market-analysis",
  "tags": ["trading","crypt"]
  "topic": "BTC trend",
  "value": "bullish",
  "confidence": 0.72,
  "timestamp": 171000000
}

```
- **Knowledge artifact**
```json
{
  "artifact_id": "uuid",
  "agent_id": "string",
  "artifact_type": "knowledge",
  "tags": ["pdf","skill"],
  "status": "solid",
  "value: {
  "inputs": [],
  "outputs": []
   }
}
```
- **State artifact**
```josn
{
  "type": "state",
  "agent": "market-analysis",
  "object": "BTC-USD",
  "stage": "analysis_complete",
  "timestamp": 1710000000
}
```
- **Process Artifact**
```json
{
  "type": "process",
  "agent": "market-analysis",
  "workflow": "trend_scan",
  "status": "completed",
  "inputs": ["BTC-USD"],
  "outputs": ["trend_report.json"],
  "timestamp": 1710000001
}
```

- **结构**一个最小的artifact应当如下：
```json
{
  "artifact_id": "uuid",
  "agent_id": "string",
  "artifact_type": "memory|state|knowledge",
  "tags": "string",
  "status": "draft|solid|locked",
  "value: {
  "inputs": [],
  "outputs": []
   },
  "payload": {},
  "created_at": "timestamp"
}
```

**nexus_manifest.json**
```json

{
  "project_fingerprint": "hash_v2026_xyz",
  "artifact_registry": {},
  "entropy_control": {
    "solidified_nodes": [],
    "active_variance": []
  },
  "handoff_logic": {
    "pits": [],
    "dead_ends": [],
    "open_loops": []
  },
  "task_graph": {
    "DataMiner": ["completed", "DataMiner"],
    "LogicFixer": ["running", "DataMiner"],
    "RiskEngine": ["pedding", "LogicFixer"]
  }
}

```
**decision_log.json**
```json
[]
```

**meeting_minutes.md**

*   **\[议题\]**: 为什么吵架？（如：Python 处理大数与 Node 处理大数的溢出矛盾）
    
*   **\[立场 A\]**: DataMiner 的建议。
    
*   **\[立场 B\]**: LogicFixer 的反对理由。
    
*   **\[裁决方案\]**: 最终选定的路径及其逻辑。
    
*   **\[Manifest 更新点\]**: 哪些新规则被写入了“物理约束”。


## 四、启动脚本示例（bash）
```
\# 启动 DataMiner

openclaw agent start \--name DataMiner \\  
 \--system-prompt ./DataMiner.prompt \\  
 \--mcp-scope mcp://project/scopes/dataminer/ \\  
 \--mcp-artifacts mcp://artifacts/ \\  
 \--mcp-manifest mcp://manifest/  
  
\# 启动 LogicFixer  
openclaw agent start \--name LogicFixer \\  
 \--system-prompt ./LogicFixer.prompt \\  
 \--mcp-scope mcp://project/scopes/logicfixer/ \\  
 \--mcp-artifacts mcp://artifacts/ \\  
 \--mcp-manifest mcp://manifest/  
  
\# 启动 RiskEngine  
openclaw agent start \--name RiskEngine \\  
 \--system-prompt ./RiskEngine.prompt \\  
 \--mcp-scope mcp://project/scopes/riskengine/ \\  
 \--mcp-artifacts mcp://artifacts/ \\  
 \--mcp-manifest mcp://manifest/
```
***

## 五、运行流程  

1.  启动 MCP Server
    
2.  启动 DataMiner → 自动 Warm Start + 执行任务
    
3. DataMiner 完成 → Shadow Handoff 更新 manifest + artifact
    
4. 启动 LogicFixer → 自动读取 manifest，执行 open\_loops
    
5. LogicFixer 完成 → Shadow Handoff
    
6. 启动 RiskEngine → 根据 DAG 执行计算，生成最终 artifact
    
7. 所有 Agent 完成 → manifest 记录最终 project fingerprint + artifacts
    

***
## 五、总工程师（碳基人类）的watching-board  

在nexus manifest agents operate systent 里碳基人类只需关注三个文件，即可掌握全局：

1.  **`manifest.json`**: 当前谁在干活，还有几个坑没填。
    
2.  **`decision_log.json`**: 为什么要放弃之前的方案。
    
3.  **`meeting_minutes.md`**: Agent 们吵架的最终结论是什么。

## 六、原则总结

*   每个 Agent 只操作自己 scope + artifact registry
    
*   所有 artifact 都版本化
    
*   Manifest 是全局状态和 DAG 控制中心
    
*   Shadow Handoff + Warm Start 保证多 Agent 接力
    
*   冲突由 Policy Engine 或 Architect 解决

