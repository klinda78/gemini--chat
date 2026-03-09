📑 Project Nexus on OpenClaw 多 Agent部署手册
========================================

版本：v2026.Engineering.Deploy  
适用场景：3 个及以上 Agent 协作项目

***

一、环境准备
------

### 1\. MCP Server

*   功能：Artifact Registry + Manifest Server + Task Scheduler
    
*   建议部署：
    
    *   Linux 或 Windows Server
        
    *   Python >= 3.11 / Node.js 18+（根据 Agent 技术栈）
        
    *   挂载共享存储（如 NFS / SMB / S3）
        
*   提供 API：
    
    *   `mcp://project/scopes/[role]/`
        
    *   `mcp://artifacts/`
        
    *   `mcp://manifest/`
        
    *   `mcp://logs/`
        

### 2\. Git 账号与版本控制

*   MCP Server 可以结合 Git 作为 artifact 版本控制（可选）
    
*   每个 Agent 配置 **独立 Git token**：
    
    *   仅限写入自己 scope 下的代码/artifact
        
    *   避免跨 Agent 修改 artifact
        

### 3\. OpenClaw 环境

*   OpenClaw CLI 安装并配置好：
    
    pip install openclaw\==2026.2.19
    
*   MCP Server URI 配置：
    
    openclaw config set mcp.server 'http://<mcp-server-host>:<port>'
    
*   共享日志目录挂载：
    
    mcp://logs/
    

***

二、Agent 分类与配置
-------------

### 1\. Agent 类型

类型

技术栈

主要任务

Manifest 关注点

DataMiner

Python / 数据处理

数据采集、异常检测

PIT 标注、artifact 输出

LogicFixer

Node.js / 逻辑修复

修复异常、实现 open\_loops

DAG 节点执行、artifact 写入

RiskEngine

Python / 量化计算

核心计算函数、验证

open\_loops 完成、结果 artifact

> 后续可添加更多类型 Agent，原则相同

***

### 2\. Scope & 权限配置

每个 Agent 分配 **独立 MCP scope**：

DataMiner: mcp://project/scopes/dataminer/  
LogicFixer: mcp://project/scopes/logicfixer/  
RiskEngine: mcp://project/scopes/riskengine/

权限规则：

*   **只读**：其他 Agent 的 scope
    
*   **读写**：自己 scope + artifact registry
    
*   **写入限制**：solidified artifact 只能 Architect 解锁
    

***

### 3\. Git 配置（可选）

*   每个 Agent 配置自己的 Git token
    
*   仅用于 artifact versioning 或代码提交
    
*   建议 repository 按 Agent 分类：
    
    nexus-artifacts/  
     dataminer/  
     logicfixer/  
     riskengine/
    

***

三、Manifest 管理原则
---------------

1.  Manifest 是全局状态中心
    
2.  包含：
    
    *   `pits`：潜在问题
        
    *   `dead_ends`：不可再试路径
        
    *   `open_loops`：未完成模块
        
    *   `task_graph`：DAG 调度
        
    *   `artifact_registry`：版本化 artifact 映射
        
3.  所有 Agent 启动前：
    
    *   读取 manifest
        
    *   识别 PIT / Dead End / Open Loop
        
    *   根据 DAG 判断可执行节点
        
4.  任务完成后：
    
    *   更新 artifact
        
    *   更新 decision\_log
        
    *   更新 manifest task\_graph 状态
        

***

四、Agent 启动与执行流程
---------------

### 1\. Shadow Handoff（前任任务交接）

*   Agent 结束任务前：
    
    1.  执行总结函数：
        
        *   提取 PIT / dead\_end / open\_loop
            
        *   更新 manifest
            
        *   写入 decision\_log
            
    2.  输出 artifact 并版本化
        
    3.  保证 scope 内无隐性地雷
        

### 2\. Warm Start（后任任务启动）

*   Agent 启动时：
    
    1.  读取 manifest + decision\_log
        
    2.  脱敏加载（忽略历史对话）
        
    3.  列出 PIT / dead\_end / open\_loop
        
    4.  根据 DAG 选择可执行节点
        
    5.  执行任务并输出 artifact
        

***

五、任务调度与 DAG
-----------

*   DAG scheduler 在 MCP Server 上运行
    
*   Task\_graph 例子：
    
    {  
     "Parser": \[\],  
     "Validator": \["Parser"\],  
     "RiskEngine": \["Validator"\]  
    }
    
*   执行规则：
    
    *   依赖完成后才可执行
        
    *   失败任务回滚到上一个 artifact 版本
        
    *   同节点任务锁定防止并行冲突
        

***

六、Artifact 与版本控制
----------------

*   所有 artifact 必须版本化：
    
    artifact://results@v1  
    artifact://results@v2  
    artifact://validator@v3
    
*   Solidified artifact:
    
    *   自动只读
        
    *   修改需 Architect 解锁
        
*   Manifest 中记录当前稳定版本
    

***

七、冲突处理与 Policy Engine
---------------------

1.  自动规则解决：
    
    *   artifact 最新版本优先
        
    *   DAG 节点依赖错误自动阻塞
        
2.  无法解决时触发：
    
    *   Kick-out Meeting
        
    *   Architect 仲裁
        
    *   更新 dead\_end / meeting\_minutes.md
        

***

八、日志与审计
-------

*   所有 Agent 操作写入：
    
    *   `decision_log.json`
        
    *   `meeting_minutes.md`（冲突产生时）
        
*   MCP Server 统一管理
    
*   可追溯任何 artifact / DAG / PIT 的历史
    

***

九、总工程师监控指标
----------

指标

计算方式

理想值

熵指标

pits 增长 / solidified\_nodes 增长

< 1

冲突频率

meeting\_minutes 生成数/周

尽量低

Artifact 演进

artifact 版本 progression

顺序递增，无跳跃

***

十、部署总结
------

部署多 Agent 关键：

1.  **每 Agent 挂载独立 Scope**
    
2.  **所有输出 artifact 写入 registry 并版本化**
    
3.  **Manifest 作为全局任务与状态中心**
    
4.  **DAG Scheduler 控制执行顺序**
    
5.  **Policy Engine 自动解决冲突**
    
6.  **Shadow Handoff + Warm Start 保证任务接力**
    
7.  **日志审计确保可追溯**
    

> 这样每个 Agent 就可以“自我调度”而不会破坏全局一致性，MCP Server + Manifest 构成了整个项目的核心操作系统。