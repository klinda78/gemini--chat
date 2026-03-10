mnifest的实现和抽象
将多agents 协作视为一个系统 工程，以系统的角度来看，它是系统状态机的运作方式
隐含有三种状态
normative state
desired state
reconciliation
initial mnifest is normative model of the project ,即系统应该是什么状态.
显式由文本约定的minifest是系统的 coherance policy 的物理构建。
policy 是一个可执行的稳定性判定程序。
runtime-audit ，是一种自指机制，reconciliation loop。
三者组合后，系统会自动形成一种 自收敛架构

这个状态机的policy可抽象为一个可编程函数，用来预测系统状态：
stable(module) = f(runtime_metrics, artifact_history, policy)
实体化的mnifest ,coherenc-policy,artifact_history 是该可编程函数的输入参数
normative state是函数寻求的输出目标，只有在这种情况下系统可以预测是自洽的。
用可编程性适应多个agent构成的不同系统，在不同模块构成和不同任务类型的状态下，调试其工程可量产的稳定性标准
