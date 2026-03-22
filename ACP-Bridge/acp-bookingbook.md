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
##  触发方式
1. 自然语言触发  
Quick start for humans
----------------------

Examples of natural requests:

*   “Start a persistent Codex session in a thread here and keep it focused.”
*   “Run this as a one-shot Claude Code ACP session and summarize the result.”
*   “Use Gemini CLI for this task in a thread, then keep follow-ups in that same thread.”

2. json task 任务触发
 ```
   {
  "task": "Open the repo and summarize failing tests",
  "runtime": "acp",
  "agentId": "codex",
  "thread": true,
  "mode": "session"
}
 ```
-  task 接受的参数如下：
  Interface details:

*   `task` (required): initial prompt sent to the ACP session.
*   `runtime` (required for ACP): must be `"acp"`.
*   `agentId` (optional): ACP target harness id. Falls back to `acp.defaultAgent` if set.
*   `thread` (optional, default `false`): request thread binding flow where supported.
*   `mode` (optional): `run` (one-shot) or `session` (persistent).
    *   default is `run`
    *   if `thread: true` and mode omitted, OpenClaw may default to persistent behavior per runtime path
    *   `mode: "session"` requires `thread: true`
*   `cwd` (optional): requested runtime working directory (validated by backend/runtime policy).
*   `label` (optional): operator-facing label used in session/banner text.
*   `resumeSessionId` (optional): resume an existing ACP session instead of creating a new one. The agent replays its conversation history via `session/load`. Requires `runtime: "acp"`.
*   `streamTo` (optional): `"parent"` streams initial ACP run progress summaries back to the requester session as system events.
    *   When available, accepted responses include `streamLogPath` pointing to a session-scoped JSONL log (`<sessionId>.acp-stream.jsonl`) you can tail for full relay history.

