## 正确的理解ACP在openclaw 里的位置
```Gateway 的行为是：

Feishu
 ↓
Gateway
 ↓
ACP dispatch
 ↓
acpx
 ↓
agent (注册到openclaw.json)
```
按照官方文档的说法 ACP和sub-agent 都是代指：type of runtime 

| Area | ACP session | Sub-agent run |
| --- | --- | --- |
| Runtime | ACP backend plugin (for example acpx) | OpenClaw native sub-agent runtime |
| Session key | agent:<agentId>:acp:<uuid> | agent:<agentId>:subagent:<uuid> |
| Main commands | /acp ... | /subagents ... |
| Spawn tool | sessions_spawn with runtime:"acp" | sessions_spawn (default runtime) |


## 需要修改的配置
- 1
```
  "acp": {
    "enabled": true,
    "dispatch": {
      "enabled": false
    },
    "defaultAgent": "gemini",
    "allowedAgents": [
      "pi", "claude", "codex", "opencode", "gemini", "kimi"
    ],
	 "maxConcurrentSessions": 2,
    "stream": {
      "coalesceIdleMs": 300,
      "maxChunkChars": 1200,
     },
    "runtime": {
      "ttlMinutes": 120,
     }
   },
```
- 2 ACP目前只支持discord ,telgrame
```
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "discord",
        accountId: "default",
        peer: { kind: "channel", id: "222222222222222222" },
      },
      acp: { label: "codex-main" },
    },
    {
      type: "acp",
      agentId: "claude",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
      acp: { cwd: "/workspace/repo-b" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "discord", accountId: "default" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "telegram", accountId: "default" },
    },
  ],
  channels: {
    discord: {
      guilds: {
        "111111111111111111": {
          channels: {
            "222222222222222222": { requireMention: false },
          },
        },
      },
    },
    telegram: {
      groups: {
        "-1001234567890": {
          topics: { "42": { requireMention: false } },
        },
      },
    },
  },
```
