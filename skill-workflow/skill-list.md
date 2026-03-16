## basic skills
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

## handoff skills
**skill_warm_start**
```
def skill_warm_start(handoff_id=None):
    if handoff_id:
        # Takeover 模式
        origin_workspace = lookup_origin_workspace(handoff_id)
        cd(origin_workspace)
    else:
        origin_workspace = current_workspace()

    # 加载 decision log
    decision_log = load_json(origin_workspace + "/decision_log.json")

    # 加载 artifact.json
    artifacts = load_json(origin_workspace + "/artifact.json")["artifacts"]

    # 如果 handoff_id 指定，过滤相关 artifact
    if handoff_id:
        handoff = find_handoff(decision_log, handoff_id)
        relevant_artifacts = [a for a in artifacts if a["artifact_id"] in handoff["artifact_ids"]]
    else:
        relevant_artifacts = artifacts

    # 初始化 agent runtime state
    runtime_state = {
        "workspace": origin_workspace,
        "handoffs": decision_log["handoffs"],
        "artifacts": relevant_artifacts
    }

    return runtime_state
```
**skill_taking_over**
```
def skill_taking_over(handoff_id, manager_agent):
    # warm start
    state = skill_warm_start(handoff_id)

    # 更新 decision_log
    handoff = find_handoff(state["handoffs"], handoff_id)
    handoff["status"] = "fixing"
    handoff["fixer"] = current_agent_id()

    save_json(current_workspace() + "/decision_log.json", state)

    # 执行任务 (pseudo)
    task_result = execute_task(handoff, state["artifacts"])

    # 任务完成
    handoff["status"] = "completed"
    save_json(current_workspace() + "/decision_log.json", state)

    # @manager agent 提交 info.json
    info = {
        "from": current_agent_id(),
        "event": "handoff_submit",
        "handoff": [handoff]
    }
    send_to_manager(manager_agent, info)

    return task_result
```
**skill_request_manager**
```
def skill_request_manager(agent_id):
    while True:  # agent 永动心跳循环
        # 1️⃣ 总结当前/前序工作，获取 artifact 与 handoff
        artifact, handoff = summarize_previous_work(
            agent_id=agent_id,
            format="json"
        )

        # 2️⃣ 更新本地 workspace
        update_before_handoff(
            workspace_path=workspace_of(agent_id),
            filename=["artifact.json", "decision_log.json"]
        )

        # 3️⃣ 提交给 manager（只包含 handoff 状态/摘要）
        notice_manager(handoff)

        # 4️⃣ 等待下一次心跳
        sleep(heartbeat_interval)
```
**skill_manager_planning**

```
def skill_manager_planning():
    while True:  # manager agent 永动心跳循环
        # 1️⃣ 接收 worker agent 提交的 handoff info
        info_json = receive_handoff()

        if not info_json:
            sleep(manager_heartbeat_interval)
            continue

        # 2️⃣ 更新全局 handoff_queue.json
        update_handoff_queue(info_json)

        # 3️⃣ 做规划决策，分配下一个 agent / task
        planning_result = planning(info_json)

        # 4️⃣ spawn 下一个 agent 执行任务
        spawn_agent(task_json=planning_result)

        # 5️⃣ 循环下一次心跳
        sleep(manager_heartbeat_interval)
```
🔹 总结

1. 永动 agent 模式

*   Worker 永动循环负责执行和提交 handoff

*   Manager 永动循环负责调度、规划和 spawn

2. 数据流清晰

*   Worker → notice_manager(info.json) → Manager

*   Manager → planning → spawn 下一个 agen(task,handoff_id)

*   handoff 与 artifact 构成项目知识库

*   worker 总是同步 artifact.json + decision_log.json

*   takeover agent 可直接 warm_start
