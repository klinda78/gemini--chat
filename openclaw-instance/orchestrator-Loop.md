## 总工程师调度流程 (The Orchestrator Loop)
-----------------------------------
**所有并行或接替的 Agent 必须接入统一的 Nexus MCP 服务器。这不再是简单的文件传递，而是建立了一个持续的“项目工作场”**  

1. **分发阶段:** 总工读取 requirements.txt，调用 Architect 生成任务清单。
   
2. **定义环境:** 配置 MCP 环境，挂载必要的本地/云端路径。
    
3. **播种 (Seeding)：** 手动初始化第一个 `manifest.json`,menifest是全局唯一真相源 (SSOT)。
    
4. **驱动 (Activating)：** - 启动 Agent A。
    
  *   Agent A 读取 Manifest ➜ 思考 (Plan) ➜ 执行 (Act) ➜ 更新 Manifest ➜ 退出。
        
5. **自动接力 (Auto-Handoff)：** - 调度脚本检测到 `last_active_agent` 变更，自动唤醒 `next_designated_agent`。
    
  *   下一个 Agent 通过 MCP 直接继承“物理现场”。
        

***

## 🛠️ 总工指令：如何部署这个白皮书？
-------------------

为了让你的 Gemini CLI 或 Python 脚本支持这套系统，你需要为所有的 Agent 配置以下 **System Instruction** 的开头：

> “你已接入 **Project Nexus MCP 系统**。你的行为准则已在 `mcp://project/manifest.json` 中定义。
> 
> 1.  启动即读取 Manifest。
>     
> 2.  严禁修改已列入 `dead_ends` 的逻辑。
>     
> 3.  退出前必须总结当前状态并同步至 Manifest。
>     
> 4.  你的产出必须存放在 `mcp://artifacts/` 路径下。”
>
