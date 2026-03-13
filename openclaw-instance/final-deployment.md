
``` tree
D:\OpenClaw\Workspace\project
├── nexus_manifest.json          # 全局唯一真相源 (SSOT)
├── requirements.txt             # 任务源头：总工定义的原始需求
│
├── 🛡️ scope/                   # 物理隔离区 (Sandbox)
│   ├── agent1-workspace/               # Agent A 工作领地
│   │   ├── temp/               # 临时空间
|   |   ├── artifact.md          # 项目知识库产物 
│   │   └── artifact/           # 固化产物区(The Truth)
│   │            ├── code/                    # 只有通过审计的代码才能进入这里
│   │            └── docs/                    # 工作成果 (video,file,picture,others,matters)
│   │
│   ├── agent2-workspace/                # Agent 2 工作领地
│   ├── agent3-workspace/                # Agent 3 工作领地
|   ├── 📦share-doc/                 # 共享data / matters
│   ├──📜 logs/                    # 审计足迹
|   ├── Auditor-agent/                 # 审计者的空间
|   └── meeting_minutes/         # 历次冲突解决会议的 Markdown 记录
└── 
```
 **Menory Artifact**
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

- **Knowledge Artifact**
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
- **State Artifact**
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



**meeting\_minutes.md**

*   **\[议题\]**: 为什么吵架？（如：Python 处理大数与 Node 处理大数的溢出矛盾）
    
*   **\[立场 A\]**: DataMiner 的建议。
    
*   **\[立场 B\]**: LogicFixer 的反对理由。
    
*   **\[裁决方案\]**: 最终选定的路径及其逻辑。
    
*   **\[Manifest 更新点\]**: 哪些新规则被写入了“物理约束”。
