## manifest的实现和抽象
- 将多agents 协作视为一个系统 ，以系统的角度来看，它是系统状态机
- 多agent系统的三种状态
1. normative state
2. desired state
3. reconciliation

**显式由文本minifest**
显式约定的minifest是系统的 coherance policy 的物理构建。
- initial mnifest is normative model of the project ,即系统应该是什么状态.
- coherenc-policy 是一个可执行的稳定性判定程序。
- runtime-audit ，是一种自指机制：reconciliation loop。
三者组合后，系统会自动形成一种 自收敛架构

**stability threshold** 
这个状态机的的状态抽象为函数,stability-threshold ,它不是一个阈值，是一个可编程函数，用来预测系统状态：
``` 
stable(module) = f(runtime_metrics, artifact_history, policy)
```
**实体化的mnifest ,coherenc-policy,artifact_history**
三者是该可编程函数的输入参数
**normative-state**
是函数寻求的输出目标，只有在这种情况下系统可以预测是自洽的  
用可编程性来适应多个agent构成的不同系统，以便在不同模块构成和不同任务类型的状态下，调试其工程可量产的稳定性标准
**predict-process**
通过是上面的设计，我们要做到: predcit_status(modules)  
此process必须具备**两个特征**  
**1.单调性**  
  ```
  unstable → candidate → stable → locked
  ```
**2.可解释性**  
输出是结构式的,回答:why_not_stable
  ```
  Parser_Core not stable:
    - success_runs = 8 < 10
  ```
## predict-satau 是一个系统级的rumtime-workflow
**the runtime audit**  
```
metric collection
evaluation
policy decision
```
an example:  audit process  
```flowchart
artifact metrics
execution metrics
agent metrics
        │
        ▼
stability evaluator
        │
        ▼
policy engine
        │
        ▼
lock / unlock
```
an example of mnifest entity:
```YAML
stability_policies:

  Parser_Core:
    predicate: |
      success_runs >= 10
      AND error_rate == 0
      AND artifact_hash_stable == true

  Research_Module:
    predicate: |
      outputs_consistent >= 0.8
      AND pit_count == 0
```
系统 runtime audit  
```
evaluate(predicate)
```
如果返回 TRUE  
```
solidified_nodes:
  - Parser_Core
```
## 需要策略
=====
**稳定性函数如何设计为可编程 predicate**
前面列出的分布（pit数量、artifact修改次数、agent贡献量、模块稳定周期）是 **原始事件/行为层面的统计**。
它们反映的是 **agent 在系统中产生的“噪声”**。
为了解决这个问题，通常做两件事：

### （1）引入 runtime-audit / derived state

不要直接用 agent 统计，而是用 **artifact + execution metrics** 作为输入：
```
stable(module) = f(artifact\_consistency, error\_rate, test\_results)
```
*   这些指标本身经过多轮验证，分布相对平滑。
    
*   可减少 heavy-tail 和 Zipf 对阈值的直接影响。
    
*   使 threshold 函数输出更稳定。
### （2）即便仍然使用 agent 标签，也要加 **平滑或累积机制**：
===
``` 
def stable(module):  
 recent\_window \= last\_n\_runs(module, n\=10)  
 pit\_count\_avg \= mean(\[run.pit\_count for run in recent\_window\])  
 artifact\_changes\_avg \= mean(\[run.artifact\_changes for run in recent\_window\])  
 return pit\_count\_avg < max\_pit\_threshold and artifact\_changes\_avg < max\_artifact\_threshold
```
*   使用滑动窗口或移动平均，避免单次极端值引起的波动。
    
*   将原本强随机性的 agent-classified tag 转化为 **平滑化的决策指标**。
    
*   输出的 stable/unstable 判定才不会随机波动。
    
***

### (3) 结论
====

*   **单纯使用 agent-edit 分类标签统计**来判定模块稳定性 → 会得到高度随机的阈值，不可靠。
    
*   **引入 runtime-audit + 平滑 / 聚合逻辑** → 可以得到稳定、可预测的稳定性判断。
    
*   本质是 **从原始 agent 事件（heavy-tailed） → 经过计算/累积 → 得到稳定性判定（近似可控随机性）**。
