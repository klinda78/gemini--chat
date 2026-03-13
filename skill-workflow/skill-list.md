## skills
**Skill: write_artifact**
```js
// skills/write_artifact.js
import fs from "fs";
import path from "path";

export async function writeArtifact(agentWorkspace, artifact) {
    const artifactPath = path.join(agentWorkspace, "artifact.md");
    fs.appendFileSync(artifactPath, JSON.stringify(artifact, null, 2) + "\n");

    // 更新 manifest registry
    const manifestPath = path.resolve("../../nexus_manifest.json");
    const manifest = JSON.parse(fs.readFileSync(manifestPath));
    manifest.artifact_registry[artifact.artifact_id] = artifact;
    fs.writeFileSync(manifestPath, JSON.stringify(manifest, null, 2));
}
```

**Skill: spawn_agent**

用于总工动态创建 agent workspace 并注入 task.md。

```js
// skills/spawn_agent.js
import fs from "fs";
import path from "path";

export function spawnAgent(projectDir, agentId, taskContent) {
    const ws = path.join(projectDir, "scope", `${agentId}-workspace`);
    if (!fs.existsSync(ws)) fs.mkdirSync(ws, { recursive: true });

    fs.writeFileSync(path.join(ws, "task.md"), taskContent);
    fs.writeFileSync(path.join(ws, "artifact.md"), "");
    fs.mkdirSync(path.join(ws, "temp"), { recursive: true });

    return ws;
}
```
**Skill: update_manifest**

用于总工根据 agent 状态更新 task graph。
```js
// skills/update_manifest.js
import fs from "fs";
import path from "path";

export function updateManifest(taskGraphUpdate) {
    const manifestPath = path.resolve("./nexus_manifest.json");
    const manifest = JSON.parse(fs.readFileSync(manifestPath));

    for (const [task, status] of Object.entries(taskGraphUpdate)) {
        manifest.task_graph[task] = status;
    }

    fs.writeFileSync(manifestPath, JSON.stringify(manifest, null, 2));
}

```

**总工逻辑**

总工在 OpenClaw 中可以用一个 agent 或脚本实现：
```js
import { spawnAgent } from "./skills/spawn_agent.js";
import { updateManifest } from "./skills/update_manifest.js";

// 1. 读取 requirements
const requirements = fs.readFileSync("./requirements.txt", "utf-8").split("\n");

// 2. 按任务拆分 agent
const agentWorkspaces = requirements.map((task, idx) => 
    spawnAgent("./", `agent${idx+1}`, task)
);

// 3. 初始化 task graph
const taskGraphUpdate = {};
agentWorkspaces.forEach((ws, idx) => {
    taskGraphUpdate[`Task${idx+1}`] = ["pending", ""];
});
updateManifest(taskGraphUpdate);
```

之后，总工定时 读取 manifest、判断 agent 状态、触发 agent 执行。
**Agent prompt（改动很小）**

保持原 prompt，只需让 agent 调用 write_artifact skill 写 artifact 并 @总工。
示例：
```
你在 project/agent1-workspace 下工作
完成 task.md 指定任务
完成后调用 write_artifact(agentWorkspace, artifact)
更新 artifact 状态：draft -> solid -> locked
@总工
```

### 最小可运行流程

1.总工 agent 启动

2.总工 spawn agent workspace + task.md

3.agent 运行 task → 调用 write_artifact → 更新 manifest

4.总工heartbeat： manifest → invoke 下一个 agent 或处理 handoff

5.直到 task graph 完成
