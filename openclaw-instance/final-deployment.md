
``` tree
D:\OpenClaw\Workspace
├── nexus_manifest.json          # 全局唯一真相源 (SSOT)
├── requirements.txt             # 任务源头：总工定义的原始需求
│
├── 🛡️ project/                   # 物理隔离区 (Sandbox)
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
