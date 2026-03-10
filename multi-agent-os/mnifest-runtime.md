## mnifest的实现和抽象
- 将多agents 协作视为一个系统 ，以系统的角度来看，它是系统状态机
- 多agent系统的三种状态
1. normative state
2. desired state
3. reconciliation

### 显式由文本minifest
显式约定的minifest是系统的 coherance policy 的物理构建。
- initial mnifest is normative model of the project ,即系统应该是什么状态.
- coherenc-policy 是一个可执行的稳定性判定程序。
- runtime-audit ，是一种自指机制：reconciliation loop。
三者组合后，系统会自动形成一种 自收敛架构

### stability threshold 
这个状态机的的状态抽象为函数,stability-threshold ,它不是一个阈值，是一个可编程函数，用来预测系统状态：
``` 
stable(module) = f(runtime_metrics, artifact_history, policy)
```
#### 实体化的mnifest ,coherenc-policy,artifact_history
三者是该可编程函数的输入参数
#### normative-state
是函数寻求的输出目标，只有在这种情况下系统可以预测是自洽的  
用可编程性来适应多个agent构成的不同系统，以便在不同模块构成和不同任务类型的状态下，调试其工程可量产的稳定性标准
#### predict-process
通过是上面的设计，我们要做到: predcit_status(modules)<br>
此process必须具备两个特征  
1.单调性
  ```
  unstable → candidate → stable → locked
  ```
2.可解释性  
输出是结构式的,回答:why_not_stable
  ```
  Parser_Core not stable:
    - success_runs = 8 < 10
  ```
## 系统rumtime-workflow
the runtime audit  
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
系统loop runtime audit  
```
evaluate(predicate)
```
如果返回 TRUE  
```
solidified_nodes:
  - Parser_Core
```

