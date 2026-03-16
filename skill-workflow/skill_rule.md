### 原则
human → coder AI  
agent → agent (ACP)  

D:\tools\coder  
   cursor  
   gemini-cli  
   codex  

D:\tools\agent  
   lobster  
   gemini-adapter  
   claude-adapter  

openclaw  
   │
   ├─ gemini-agent
   ├─ claude-agent  
   └─ codex-agent  

不要让 agent 控制 shell 的每条命令  

不要让 agent 保持持续 shell  

workflow 应该调用 工具/脚本  

脚本可以内部执行多条命令，但 agent 外部不会看到 shell 状态  


###  理解工作方式  

在推荐架构里：  

human  
   ↓
agent (OpenClaw)  
   ↓
tool / skill  
       ↓
   script / binary  

agent 不应该直接操控 bash  
避免interatcive bash
“agent runtime 不应该拥有 interactive shell”。  

**示例1**  

你说：build file D:\projects\projectA\main.cpp

OpenClaw workflow 解析为：  

tool: build_project  
input: D:\projects\projectA\main.cpp 

build_project.sh 或者编译工具执行：  

git pull (inside script)  
npm install  
node build.js  

Script 执行完退出 → agent 收到结果  

✅ agent 没有进入一个持续 shell，不会改变环境状态。  

 
**示例2**  
lobster tools  
runtime 做的事情其实是：  
spawn lobster process  

OpenClaw 会：  
tool: lobster  
spawn lobster-cli  ok  

openclaw  
   └─ git.exe   ok  

tool: shell  
command: npm install  x  
runtime：
bash session  x  

### 一句话解释  

spawn 不是：  

openclaw CLI  

而是：
openclaw runtime 如何执行工具  
只要 runtime 是：  
direct process execution就属于 spawn model。  
