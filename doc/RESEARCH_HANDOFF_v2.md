---
title: Research Handoff
type: doc
source_ref: chatgpt-￥conversation://6a77f638-3804-83ea-af75-4ce56e6a80fe
captured_at: 2026-08-09
---

# RESEARCH_HANDOFF.md

## Event-Driven Alpha 与 Conditional Risk Allocation：研究交接、思路演化与设计理由

> **用途**：供新的 Work / Agent Session 接手本研究。  
> **阅读要求**：不要只读取最终结论。本文保留问题如何被重新定义、哪些中间方案一度合理、为什么后来被修正，以及哪些内容只是待验证假设。  
> **关键规则**：本文中的“历史思路”不等于“当前架构”。接手 Agent 必须区分 **已讨论的中间方案、当前更高层抽象、以及尚未被 OOS 证明的假设**。  
> **真实性边界**：没有回测或 OOS 结果支持的内容，只能写成 hypothesis / design rationale，禁止写成“已证明”“必然有效”或虚构的经验结论。

------------------------------------------------------------------------

# 0. 接手 Agent 首先要理解什么

本研究目前形成两个可以独立开发、但方法论相通的方向：

$$
\boxed{
EventAlpha:
Novel\ Event
\rightarrow
Mature\ Asset
\rightarrow
Delayed\ Repricing
}
$$

以及：

$$
\boxed{
ConditionalRiskAllocation:
Information_t
\rightarrow
P(R_{strategy}\mid Information_t)
\rightarrow
Risk\ Allocation
\rightarrow
Exposure_t
}
$$

二者解决不同问题：

1. **EventAlpha**：新的外生事件发生后，是否存在尚未完成定价的成熟可交易资产？
2. **Conditional Risk Allocation**：已有 Base Strategy 的未来收益/风险分布，是否会随市场信息状态而系统性变化；如果会，如何把这种条件分布映射为具体 Exposure？

接手 Agent **不要把第二个方向缩写成“用 HMM 判断牛熊并调仓”**。HMM、HSMM、聚类、Market State、Sector State 都只是候选中间表示，不是顶层目标。

当前更稳定的职责边界是：

```text
Base Strategy
→ 独立产生 Alpha / entry / exit / base risk

Information Model
→ 从市场、板块、横截面、流动性、波动等信息中形成可用状态表示

Conditional Distribution Model
→ 估计 P(R_strategy | Information_t)

Risk Allocation
→ 在风险偏好与约束下计算 risk budget

Exposure Controller
→ 输出最终 Exposure_t
```

数学上，最终对象是：

$$
\boxed{
Exposure_t^*
=
f\left(
P(R_{strategy}\mid Information_t),
RiskPreference,
Constraints
\right)
}
$$

这里的核心不是某个具体状态模型，而是：

> **市场信息是否包含关于 Base Strategy 条件收益分布的稳定信息，以及这种信息能否改善风险配置。**

因此：

$$
P(R_{strategy}\mid Information_t)
\neq
P(R_{strategy})
$$

是比“某个状态可以预测指数涨跌”更弱、更干净、也更接近真实研究目标的命题。

------------------------------------------------------------------------
# 1. 最初的问题：寻找"小容量但客观"的交易机会

研究起点来自一个 A 股可转债案例：

``` text
突发利好
   ↓
正股价值发生变化
   ↓
可转债与正股存在明确经济关系
   ↓
场内可转债可能尚未完成定价
   ↓
机器人利用 T+0 快速执行
```

关注的不是传统意义上的：

> 哪种策略长期预测涨跌最准？

而是：

> 是否存在因为容量太小、持续时间太短、需要持续监控或执行过于麻烦，而没有被大型资金完全消灭的结构性
> Edge？

因此候选机会需要具有若干特征：

-   Edge 来源相对客观；
-   容量有限；
-   生命周期短；
-   机器具有注意力和执行优势；
-   可以事后统计验证；
-   大型机构未必具有足够经济激励参与。

这里产生第一个重要认识：

$$
Small\ Capacity
$$

**不一定是缺陷。**

对于大型基金：

$$
Expected\ PnL
<
Institutional\ Minimum\ Economical\ Size
$$

时，即使存在 Alpha，也未必值得投入研究、工程、合规与资金成本。

但同一个机会对个人自动化 Agent 可能完全值得执行。

------------------------------------------------------------------------

# 2. 第一阶段想法：Cross-Asset Lead-Lag

一个直观例子是：

``` text
黄金暴涨
   ↓
A股黄金股暴涨
   ↓
白银本身已经确认上涨
   ↓
白银股票尚未上涨
   ↓
若历史映射成立
   ↓
买入白银股票等待补涨
```

这不是简单的相关性交易。

真正需要研究的是：

$$
P(R_B(t+\Delta t)>x
\mid
R_A(t)>a,\ Context)
$$

是否显著高于：

$$
P(R_B(t+\Delta t)>x)
$$

核心结构是：

$$
\boxed{
Observed\ Shock
+
Confirmed\ Mapping
+
Pricing\ Gap
}
$$

也就是说：

1.  某个 Leader 已经发生异常变化；
2.  Leader 与 Lagger 存在经济或历史传播关系；
3.  Lagger 尚未完成价格反应；
4.  历史上这种状态具有正期望。

这和传统 Pair Trading 不同。

Pair Trading 更接近：

$$
Spread_{A,B}\rightarrow Mean
$$

这里则具有明确时间箭头：

$$
A_t\rightarrow B_{t+\Delta t}
$$

本质是 **Delayed Repricing / Information Diffusion**。

------------------------------------------------------------------------

# 3. 为什么后来不满足于纯历史 Lead-Lag

讨论进一步发现：

如果某个 Lead-Lag 是长期稳定、完全由市场价格历史即可发现，那么：

``` text
历史数据
   ↓
任何量化团队都能研究
   ↓
回测
   ↓
部署
   ↓
永久等待机会
```

这种 Alpha 更容易拥挤。

因此产生了一个重要区分：

## 3.1 Historical Lead-Lag

特点：

-   可提前完整研究；
-   大量参与者可以同时观察；
-   容易规模化部署；
-   Alpha 更容易被竞争压缩。

## 3.2 Event-Triggered Lead-Lag

特点：

``` text
突发事件
   ↓
是否第一时间发现？
   ↓
是否判断为真实？
   ↓
是否理解事件的重要性？
   ↓
能否映射一阶资产？
   ↓
能否进一步找到二阶资产？
   ↓
二阶资产是否尚未定价？
   ↓
是否还有容量？
   ↓
能否及时执行？
```

每多一个约束，就过滤掉一批竞争者。

于是策略保护层变成：

$$
\boxed{
Small\ Capacity
\times
Event\ Driven
\times
Second\ Order
\times
Execution\ Speed
}
$$

这是第一个重要设计转向：

> **历史数据不再直接负责产生所有交易信号，而主要负责学习"事件发生后价格通常怎样传播"。实时事件负责激活某条传播路径。**

------------------------------------------------------------------------

# 4. EventAlpha 的基本问题被重新定义

目标从：

> 找历史上 A 涨以后 B 会不会涨。

变成：

> 一个新的外生事件已经发生，哪些资产已经完成定价，哪些与事件存在真实因果关系的资产还没有完成定价？

可以写成：

$$
Event_t\rightarrow A\rightarrow B
$$

其中：

$$
Impact(Event,A)>Threshold
$$

同时：

$$
ExpectedImpact(Event,B)
-
ObservedImpact(B)
>
Threshold
$$

才产生候选机会。

因此真正寻找的是：

$$
\boxed{\text{Delayed Repricing}}
$$

不是预测尚未发生的事件，而是判断：

> **已经发生的事情，还有哪些价格没有反应完。**

------------------------------------------------------------------------

# 5. 为什么 LLM 在这里有价值

"二阶资产"不是一个固定 ticker mapping。

例如某个协议集成、新交易所上线、监管变化、供应链变化，都可能通过不同路径影响资产。

因此需要回答：

``` text
Event
  ↓
直接影响谁？
  ↓
这些一阶资产已经反应了吗？
  ↓
事件还会通过哪些经济/协议/资金关系传播？
  ↓
有哪些二阶资产？
  ↓
哪些二阶资产尚未完成定价？
```

这属于开放世界语义和因果候选发现，传统固定规则覆盖成本很高。

因此形成职责边界：

``` text
Event Detection        → 数据源 / 规则 / embedding
Market Reaction        → 确定性数值计算
Asset Relationship     → Knowledge Graph + LLM
Second-Order Reasoning → LLM
Historical Validation  → 统计 / Bayesian
Execution Filtering    → 确定性代码
```

这里形成了一条重要原则：

$$
\boxed{
LLM\ 找关系，
数据证明关系，
代码执行关系
}
$$

尤其不要直接问：

> "LLM，你觉得现在应该买什么？"

更好的问题是：

> "这个事件可能通过哪些可解释的因果路径影响哪些资产？哪些属于一阶、二阶或更远关系？"

LLM 输出的是 **hypothesis / candidate**，不是最终交易真值。

------------------------------------------------------------------------

# 6. 为什么引入 Bayesian

突发事件天然存在：

-   样本少；
-   类别多；
-   事件异质；
-   条件不断变化；
-   很多关系只有少量历史案例。

因此单纯的：

``` text
历史胜率 = 70%
```

不足以表达证据强度。

最简单可以设：

$$
p=P(\text{补涨})
$$

并使用：

$$
p\sim Beta(\alpha_0,\beta_0)
$$

观察历史数据后：

$$
p\mid D
\sim
Beta(\alpha_0+s,\beta_0+f)
$$

真正关心的不是一个点估计，而是：

$$
P(p>p_0\mid D)
$$

甚至更直接：

$$
P(EV>0\mid D)
$$

其中：

$$
EV
=
P(win)E[gain]
-
P(loss)E[loss]
-
Cost
$$

所以 Bayesian 在这里的价值不是"显得高级"，而是：

> **在小样本条件下明确表达：我们相信什么，以及我们有多不确定。**

------------------------------------------------------------------------

# 7. 先验不是单因子：需要多因子条件模型

讨论随后进一步修正：

真正的问题不是：

$$
P(Opportunity\mid EventType)
$$

而是：

$$
\boxed{
P(
Opportunity
\mid
Event,
Relation,
Market,
Price,
Attention,
Liquidity
)
}
$$

可以定义：

$$
X=[
event\ type,
source,
surprise,
leader\ move,
volume\ shock,
relationship\ strength,
lagger\ move,
market\ regime,
liquidity,
attention
]
$$

例如 Bayesian Logistic Regression：

$$
P(Y=1\mid X)
=
\sigma(
\beta_0+\beta_1X_1+\cdots+\beta_nX_n
)
$$

同时：

$$
\beta_i\sim N(\mu_i,\sigma_i^2)
$$

由于事件具有天然层级：

``` text
Crypto Event
│
├── Listing
│   ├── Binance
│   ├── Coinbase
│   └── Upbit
│
├── Protocol
│   ├── Integration
│   ├── Mainnet
│   └── Tokenomics
│
└── Regulation
```

因此 Hierarchical Bayesian 很自然：

$$
\beta_{Binance}
\sim
N(\mu_{Listing},\tau_{Listing}^2)
$$

$$
\mu_{Listing}
\sim
N(\mu_{Event},\tau_{Event}^2)
$$

这样可以利用 partial pooling，避免少量成功样本产生过度自信。

------------------------------------------------------------------------

# 8. Expert Agent 应该做什么，不应该做什么

曾提出：

> 是否让专家 Agent 计算各个先验因子的概率？

结论是：**可以，但要严格限制角色。**

Agent 不应该凭语言判断直接宣布：

``` text
上涨概率 = 73.42%
```

更合适的工作是：

``` text
Raw Event / Market Data
        ↓
Expert Agent
        ↓
Structured Factor
+ Causal Hypothesis
+ Evidence
+ Confidence
        ↓
Bayesian / Statistical Calibration
```

可以拆成：

``` text
Event Expert
→ 事件是否真实、重要、surprising

Relation Expert
→ A → B 是否存在真实经济传导

Market Expert
→ 当前环境是否支持该传播

Pricing Expert
→ B 是否已经完成 repricing

Liquidity Expert
→ 理论 Edge 是否可以真实成交
```

LLM 产生的 `causal_strength` 等数值本身也只是特征：

$$
X_{LLM}
$$

历史数据最终学习：

$$
\beta_{LLM}
$$

如果 LLM 的评分没有实际预测能力：

$$
\beta_{LLM}\approx0
$$

模型应该能够自动降低其权重。

这就是 calibration。

------------------------------------------------------------------------

# 9. 真正重要的数据资产：Event → Factors → Outcome

长期最有价值的不是某个具体 Agent 或 Prompt，而是历史事件数据库。

需要保存：

``` text
Event
  ↓
Factors
  ↓
Candidate Assets
  ↓
Market State
  ↓
Outcome
```

建议同时保存 Raw 与 Derived。

## Raw

``` text
raw/
├── events.jsonl
├── trades.parquet
├── orderbook.parquet
├── funding.parquet
├── open_interest.parquet
└── liquidations.parquet
```

## Derived

``` text
derived/
├── event_features.parquet
├── asset_relations.parquet
├── market_state.parquet
└── outcomes.parquet
```

原则：

> **永远不要只保存 Agent 的结论。**

因为未来改变：

-   causal_strength 定义；
-   event surprise 定义；
-   market regime 模型；
-   LLM；
-   Bayesian specification；

都应该能够从 Raw Data 重新计算。

------------------------------------------------------------------------

# 10. Crypto Cold Start 暴露了一个结构性问题

随后出现了一个关键质疑：

A 股成熟公司有大量自身历史。

即使研究的是"退市概率"，大量"不退市"的时间本身也是有效的 survival /
censored observations。

例如：

``` text
公司 A：2500 个交易日未退市
公司 B：1800 个交易日未退市
公司 C：3200 个交易日未退市
...
```

可以研究 hazard：

$$
h(t\mid X)
=
P(T=t\mid T\ge t,X)
$$

因此成熟 A 股标的拥有大量：

-   价格历史；
-   财务历史；
-   行业历史；
-   事件历史；
-   生存历史；
-   横截面比较数据。

但一个新发行 Token：

``` text
历史价格       ≈ 0
历史成交       ≈ 0
历史波动率     ≈ 0
Funding 历史   ≈ 0
OI 历史        ≈ 0
事件反应历史   ≈ 0
生命周期       ≈ 0
```

这是典型：

$$
\boxed{Cold\ Start}
$$

这里产生一个重要认识：

$$
Bayesian\ inference
\neq
Information\ creation
$$

Bayesian 可以合理利用有限信息，但不能制造不存在的证据。

------------------------------------------------------------------------

# 11. 第一个关键反转：Novel Event → Mature Asset

Cold Start 问题促成了整个 EventAlpha 设计最重要的一次反转。

原问题：

$$
New\ Event+New\ Asset\rightarrow ?
$$

两个变量都缺历史。

于是改成：

$$
\boxed{
New\ Event
\rightarrow
Mature\ Tradable\ Asset
}
$$

例如：

``` text
新 Token X 上线
      ↓
X 自身没有足够历史
      ↓
X 属于成熟 Protocol A
      ↓
事件可能影响已交易三年的 Token Y
      ↓
Y 有完整价格 / volume / funding / OI / 事件历史
      ↓
判断 Y 是否尚未 repricing
```

于是产生一条非常重要的设计原则：

$$
\boxed{
\text{允许原因是新的，但尽量要求结果载体是旧的}
}
$$

这使不同模型的能力边界非常自然：

``` text
LLM
→ 理解从未见过的新事件

Causal Mapping
→ 找到可能受影响的成熟资产

Historical / Bayesian Engine
→ 用成熟资产历史验证

Real-Time Market Data
→ 判断当前 Pricing Gap 是否仍存在

Execution
→ 确定性执行
```

这是 EventAlpha 当前最重要的架构结论之一。

------------------------------------------------------------------------

# 12. 研究转向 A 股板块效应

随后讨论进入另一个独立但相关的问题：

> A 股具有明显板块效应，"共鸣"是否是高 Alpha 信息？

这里的"共鸣"不是简单板块涨幅，而是：

-   多只股票同向；
-   Breadth 扩大；
-   龙头存在；
-   成交参与增强；
-   横截面相关性提高；
-   个股特异性噪声下降；
-   共同因子主导价格。

可以写成：

$$
R_{i,t}
=
\beta_iF_t+\epsilon_{i,t}
$$

当板块共鸣增强：

$$
Var(\beta_iF_t)
\gg
Var(\epsilon_i)
$$

直观传播路径：

``` text
Event
  ↓
Leader
  ↓
核心成分股
  ↓
同概念股票
  ↓
边缘补涨
```

因此：

$$
\boxed{
Event
\rightarrow
Leader
\rightarrow
Sector\ Resonance
\rightarrow
Lagger
}
$$

------------------------------------------------------------------------

------------------------------------------------------------------------

# 13. 第二阶段起点：为什么大盘/板块信息值得用于风险控制

讨论转向 A 股后，一个重要直觉是：大盘和板块由多个标的组成，个体特异性噪声会部分平均，因此它们的横截面结构可能比单只股票更适合作为 **risk context**。

个股收益可写成：

$$
R_i
=
\beta_iR_{market}
+
\gamma_iR_{sector}
+
\epsilon_i
$$

其中：

$$
\epsilon_i
$$

代表较强的个股特异性噪声。

由此形成了一个早期、合理但后来被进一步抽象的想法：

> 个股 Alpha 与多标的市场环境应当分工；前者负责交易机会，后者负责风险暴露。

当时曾写成：

$$
Position_i
=
BaseRisk_i
\times
M_{market}
\times
M_{sector}
$$

并曾口头概括为：

```text
个股策略 → 决定交易
板块状态 → 决定单笔风险
大盘状态 → 决定账户风险暴露
```

**这三句话只应保留为思路演化中的中间态，不是当前顶层架构。**

原因是它们偷偷预设了一个尚未被数据证明的固定层级：

```text
Base Strategy
   ↓
Sector Multiplier
   ↓
Market Multiplier
   ↓
Final Exposure
```

我们后来意识到：这会把 `Market` 与 `Sector` 的作用方式提前写死，也默认二者应当乘法组合。真正应该由数据回答的是：

- Market information 是否有增量信息；
- Sector information 是否有增量信息；
- 二者是否需要交互；
- 某些信息是否完全无效；
- 不同 Base Strategy 是否依赖不同信息结构。

因此，`M_market × M_sector` 只能作为未来可能得到的 **具体实现形式**，不能写成系统定义。

------------------------------------------------------------------------

# 14. 最初的“共鸣状态机”想法

A 股板块效应促使我们尝试把“共鸣”量化。

最初考虑的横截面观测包括：

### Breadth

$$
UpRatio_t
=
\frac{N(R_i>0)}{N}
$$

$$
AboveMA20Ratio_t
=
\frac{N(P_i>MA20_i)}{N}
$$

### Correlation / Resonance

$$
\rho_t
=
\frac{2}{N(N-1)}
\sum_{i<j}Corr(R_i,R_j)
$$

### Dispersion

$$
D_t
=
Std(R_{1,t},R_{2,t},...,R_{N,t})
$$

### Participation / Tail / Risk

候选还包括：

- Turnover；
- VolumeBreadth；
- NewHigh / NewLow；
- LimitUp / LimitDown；
- LargeDropRatio；
- Intraday Drawdown；
- Realized Volatility。

最初自然想到：

```text
Raw indicators
    ↓
EMA
    ↓
历史百分位
    ↓
Strong / Weak / Fading
    ↓
仓位倍率
```

这一步代表传统“指标工程 + 人工状态定义”的思路。

------------------------------------------------------------------------

# 15. 第二个关键反转：不要提前替黑箱定义状态

随后意识到，如果目标真的是 latent regime discovery，那么：

```text
EMA
Threshold
人工 Resonance Score
预设 Strong / Weak
```

都会把人的判断提前编码进数据。

更干净的问题应是：

$$
\boxed{
X_t
\rightarrow
Latent\ Representation_t
}
$$

其中：

$$
X_t=[
UpRatio,
AboveMA20Ratio,
NewHighRatio,
NewLowRatio,
Turnover,
VolumeBreadth,
PairwiseCorr,
Dispersion,
LimitUpRatio,
LimitDownRatio,
IntradayDrawdown,
Volatility,
...]
$$

候选流程因此变成：

```text
Raw Cross-Sectional Features
            ↓
仅必要的数值尺度 / 缺失处理
            ↓
HMM / HSMM / GMM / Clustering / other representation
            ↓
Latent representation / state posterior
            ↓
事后解释经济含义
```

核心原则：

$$
\boxed{
\text{先发现状态，再解释状态}
}
$$

注意：这里所谓 “Raw” 指**不先人为构造目标状态**，不是禁止所有数值处理。标准化、缺失处理、训练期内拟合的尺度变换仍属于必要工程步骤。

------------------------------------------------------------------------

# 16. 为什么 EMA 不能被默认成必要步骤

讨论中曾提出：单日横截面数据噪声大，可以 EMA 平滑。

随后发现一个反例：

$$
Breadth:30\%\rightarrow80\%
$$

这种突变如果正好对应 regime transition，那么 EMA 会把最有信息的变化压掉。

因此不能先验规定：

$$
High\ Frequency\ Change=Noise
$$

HMM 本身已显式考虑：

$$
P(S_t\mid S_{t-1})
$$

若需要更明确的 state duration，可以比较 HSMM / duration model。

因此正确态度不是“EMA 错”，而是：

```text
Raw → HMM
Raw → HSMM / duration model
Raw → clustering / mixture model
Light smoothing → HMM
```

都应作为实验分支，由 Walk-Forward OOS 决定。

------------------------------------------------------------------------

# 17. 第三个关键反转：State 不是目标，Conditional Distribution 才是目标

最初我们仍然容易把问题表述成：

> 找到一个好的 Market State，然后根据 State 调仓。

进一步讨论后，这个定义仍然不够高层。

因为最终有操作价值的并不是 `State = 2`，而是：

$$
P(R_{strategy}\mid Information_t)
$$

如果 state model 只是中间表示，那么：

$$
Information_t
\rightarrow
P(S_t=k)
\rightarrow
P(R_{strategy}\mid S_t)
$$

是一个可能实现。

但也可能直接存在：

$$
Information_t
\rightarrow
P(R_{strategy}\mid Information_t)
$$

因此 HMM / Latent State 应被降格为 **可替换的 representation layer**。

真正不可替换的顶层对象是：

```text
Base Strategy
      ↓
Strategy Return Distribution
      ↑
Market Information
      ↓
Conditional Distribution
      ↓
Risk Allocation
      ↓
Exposure
```

------------------------------------------------------------------------

# 18. 一个更弱、更关键的研究命题

早期容易提出较强命题：

> “市场状态能预测市场。”

这不是本研究需要证明的。

更弱、也更直接的命题是：

$$
\boxed{
P(R_{strategy}\mid Information_t)
\neq
P(R_{strategy})
}
$$

含义是：

> 市场信息对 Base Strategy 的未来收益分布具有条件信息价值。

即使：

$$
P(Index\uparrow\mid State)
$$

没有明显预测能力，只要：

$$
P(R_{strategy}>0\mid State)
$$

或更完整的收益/风险分布随状态显著变化，这个信息仍然有风险配置价值。

这一步把研究从“Market Timing”转成了：

$$
\boxed{
Conditional\ Risk\ Allocation
}
$$

------------------------------------------------------------------------

# 19. 为什么这不是传统意义上的 Hedge

讨论中曾把它描述成“利用概率对冲策略风险”。这个直觉是对的，但术语需要更精确。

传统 hedge 常见形式：

$$
Risk_A + NegativelyCorrelatedAsset_B
\rightarrow
Lower\ Portfolio\ Risk
$$

这里并不一定增加反向资产，而是：

```text
识别 Base Strategy 在什么条件下变差
        ↓
估计该条件下的收益 / 尾部风险分布
        ↓
降低该策略 Exposure
```

因此更准确的术语是：

$$
\boxed{
State/Information\ Conditional\ Risk\ Allocation
}
$$

或：

$$
\boxed{
Probabilistic\ Risk\ Scaling
}
$$

“State-Conditional Hedge”可以作为直觉解释，但不应与传统资产对冲混淆。

------------------------------------------------------------------------

# 20. 从条件分布到具体仓位：真正需要完成的最后两步

仅仅识别 State 还无法回答：

> 今天应该 30%、60% 还是 90% Exposure？

假设某 latent model 输出：

$$
P(S_t=k)=p_k
$$

且历史上可以估计：

$$
R_{strategy}\mid S_k\sim F_k
$$

则当前条件收益分布可以写成 mixture：

$$
P(R\mid Information_t)
=
\sum_k p_k P(R\mid S_k)
$$

其期望：

$$
E[R\mid Information_t]
=
\sum_kp_k\mu_k
$$

但最终仓位不应只由期望收益决定，还需要风险偏好与约束：

$$
\boxed{
Exposure_t^*
=
\arg\max_{e\in\mathcal{E}}
E[U(eR)\mid Information_t]
}
$$

其中 $\mathcal{E}$ 是允许的 Exposure 区间和约束集合。

因此完整链路是：

$$
\boxed{
Information_t
\rightarrow
Conditional\ Return/Risk\ Distribution
\rightarrow
Risk\ Budget
\rightarrow
Exposure_t
}
$$

真正困难、也真正产生操作价值的，是最后两箭，而不是 HMM 本身。

------------------------------------------------------------------------

# 21. Exposure Control 的严格实验设计

## 21.1 冻结 Base Strategy

为了证明新增价值来自 Exposure Controller，Control 与 Experiment 必须保持：

- 相同 Universe；
- 相同 entry signal；
- 相同 exit signal；
- 相同手续费；
- 相同滑点；
- 相同交易限制；
- 相同 execution assumptions；
- 相同 Base Strategy 参数。

Control：

$$
Exposure_t^{control}=BaseExposure_t
$$

Experiment：

$$
Exposure_t^{experiment}
=
RiskAllocator(Information_t,BaseStrategy)
$$

**禁止为了让 Experiment 获胜而同时修改 Base Strategy。**

这样才能把新增表现归因于：

$$
Conditional\ Risk\ Allocation
$$

------------------------------------------------------------------------

# 22. “长期全仓”是有意义但不充分的参照

直觉上，A 股大量股票长期处于趋势、震荡、压力、回撤、分化等不同环境，因此：

$$
Time\ in\ Productive\ Regime<100\%
$$

如果风险控制器减少低质量条件下的 Exposure、增加高质量条件下的 Exposure，理论上可能出现：

$$
CAGR_{conditional}>CAGR_{constant\ exposure}
$$

但这仍只是待检验假设。

必须同时比较：

$$
CAGR
$$

$$
Sharpe
$$

$$
MDD
$$

$$
Calmar=\frac{CAGR}{MDD}
$$

以及资本效率：

$$
\frac{PnL}{Average\ Exposure}
$$

$$
\frac{CAGR}{Average\ Capital\ at\ Risk}
$$

如果平均 Exposure 明显下降，而风险调整后收益改善，其证明力比单纯 CAGR 提高更强。

------------------------------------------------------------------------

# 23. 必须采用 Walk-Forward OOS

禁止使用全样本：

```text
先发现 State
→ 看未来收益
→ 给 State 起名字
→ 调 exposure mapping
→ 再宣称历史有效
```

这会产生严重 lookahead / regime labeling leakage。

应采用类似：

```text
Train        Test
2010–2014 → 2015
2010–2015 → 2016
2010–2016 → 2017
...
```

每个 Test 只能使用此前数据确定：

- Feature schema；
- Scaling / preprocessing；
- Representation / state model；
- Number of states / latent dimensions；
- State → conditional distribution mapping；
- Risk allocation rule；
- Hyperparameters。

------------------------------------------------------------------------

# 24. 验收标准：验证的是 Risk Allocation Value，不是状态分类美观度

理想 OOS 结果包括：

$$
CAGR\uparrow
$$

$$
Sharpe\uparrow
$$

$$
MDD\downarrow
$$

更核心的是：

$$
P(R_{strategy}\mid Information)
$$

在不同信息状态下具有稳定、可重复的条件差异。

还应检查：

- 不同年份；
- 不同市场阶段；
- 不同板块；
- 不同股票池；
- 不同 Base Strategy；
- 交易成本后；
- state label permutation 后的语义稳定性；
- exposure turnover 与实现成本。

最终 KPI：

$$
\boxed{
Risk\ Adjusted\ Exposure\ Efficiency
}
$$

而不是：

$$
Directional\ Forecast\ Accuracy
$$

------------------------------------------------------------------------

# 25. 两个研究模块的共同哲学

## EventAlpha

```text
Novel Event
    ↓
LLM Semantic / Causal Candidate Discovery
    ↓
Mature Tradable Asset Mapping
    ↓
Historical / Bayesian Validation
    ↓
Real-Time Pricing Gap
    ↓
Execution
```

原则：

$$
\boxed{
\text{允许原因是新的，但尽量要求结果载体是旧的}
}
$$

## Conditional Risk Allocation

```text
Base Strategy ----------------------┐
                                   ↓
Market / Sector / Cross-Section Information
                 ↓                  │
        Representation Model        │
          (optional latent state)   │
                 ↓                  │
      Conditional Return/Risk Distribution
                 ↓
          Risk Allocation
                 ↓
             Exposure
```

原则：

$$
\boxed{
\text{Base Strategy 产生 Alpha；Risk Model 不替代 Alpha，只条件化其风险分布}
}
$$

共同方法论：

> **不要让模型承担无法被数据约束的任务；把开放世界推理、统计验证、状态表示、风险配置和确定性执行拆开。**

------------------------------------------------------------------------

# 26. 当前明确拒绝或降级为“实现候选”的思路
以下六个大坑千万注意，不要踩  

## 26.1 直接让 LLM 输出买卖概率

拒绝作为最终概率裁判。LLM 适合产生 hypothesis / relation / evidence；概率必须经过数据校准。

## 26.2 用 Bayesian 掩盖新资产 Cold Start

拒绝。Bayesian inference 不能创造不存在的信息：

$$
Bayesian\ inference\neq Information\ creation
$$

## 26.3 把长期稳定 Lead-Lag 当 EventAlpha 核心

降级为辅助研究。EventAlpha 更关注外生事件激活的短期传播结构。

## 26.4 预设 Strong / Weak / Fading

拒绝作为主路线。可以事后命名，但不能先用语义标签塑造数据。

## 26.5 默认 EMA 必须存在

降级为 preprocessing candidate，由 OOS 决定。

## 26.6 默认 `MarketMultiplier × SectorMultiplier`

**明确取消其顶层设计地位。**

它只是一种可能的低阶实现：

$$
Exposure=BaseExposure\times M_{market}\times M_{sector}
$$

只有当数据证明这种分解稳定有效时才能采用。

## 26.7 不要把 HMM 当产品本身

拒绝。

HMM 是 representation / latent-state candidate。最终产品对象是：

$$
Information
\rightarrow
ConditionalDistribution
\rightarrow
RiskAllocation
\rightarrow
Exposure
$$

------------------------------------------------------------------------

# 27. 当前最重要的研究假设

## H1 — EventAlpha

$$
\boxed{
\text{Novel Event 对 Mature Asset 的二阶影响存在可统计验证的 Delayed Repricing}
}
$$

## H2 — LLM Causal Mapping

$$
\boxed{
\text{LLM 能提高二阶资产候选发现能力，但其价值必须被历史数据校准}
}
$$

## H3 — Information Value

$$
\boxed{
P(R_{strategy}\mid Information_t)
\neq
P(R_{strategy})
}
$$

即市场/板块/横截面信息对 Base Strategy 的未来收益/风险分布具有稳定条件信息。

## H4 — Representation Value

$$
\boxed{
\text{Latent representation/state 可以有效压缩这种条件信息，但不是唯一必要形式}
}
$$

## H5 — Risk Allocation Value

$$
\boxed{
\text{利用条件收益/风险分布动态配置 Exposure，可以改善 OOS 风险调整后表现}
}
$$

这些全部是 **待证伪假设**。当前没有任何一句应被写成“必然成立”。

------------------------------------------------------------------------

# 28. 推荐开发顺序：先验证最弱命题，不要先造复杂状态机

当前更合理的 Regime / Risk Control MVP 顺序是：

```text
Phase 1
冻结一个 Base Strategy
        ↓
Phase 2
定义 Market / Sector / Cross-Section Raw Information Schema
        ↓
Phase 3
建立严格无未来泄漏的数据集
        ↓
Phase 4
先检验：Information 是否条件化 Base Strategy Return Distribution
        ↓
Phase 5
建立简单 baseline representation
        ↓
Phase 6
比较 HMM / HSMM / GMM / clustering / direct conditional model
        ↓
Phase 7
估计 Conditional Return / Risk Distribution
        ↓
Phase 8
设计受约束 Risk Allocation
        ↓
Phase 9
Walk-Forward OOS
        ↓
Phase 10
Control vs Conditional Exposure
```

这里比旧版本多了一个重要前置：

> **先证明 Information 有条件信息价值，再讨论哪种 latent-state 模型最漂亮。**

否则可能花大量时间优化一个根本没有经济增量的信息表示器。

------------------------------------------------------------------------

# 29. Raw Information MVP：先保留信息，不先制造“共鸣分数”

候选原始横截面信息：

```text
Breadth
├── UpRatio
├── AboveMA20Ratio
├── NewHighRatio
└── NewLowRatio

Dependence / Resonance
├── PairwiseReturnCorrelation
└── DirectionAgreement

Dispersion
└── CrossSectionReturnStd

Participation
├── Turnover
└── VolumeBreadth

Tail / Risk
├── LimitUpRatio
├── LimitDownRatio
├── LargeDropRatio
├── IntradayDrawdown
└── RealizedVolatility
```

注意：

- 这些是候选信息，不是已证明因子；
- 不要先合成 `ResonanceScore`；
- 不要先假定 Market 与 Sector 的信息必须分层相乘；
- 可以同时保留 market-level 与 sector-level raw observations；
- 让模型和 OOS 决定哪些有增量信息；
- 所有 scaling / winsorization / PCA 等只能在 Train 内拟合。

------------------------------------------------------------------------

# 30. EventAlpha 的后续开发顺序

```text
Phase 1  Event Raw Store
Phase 2  Event Taxonomy
Phase 3  LLM Causal Candidate Extraction
Phase 4  Asset / Protocol Knowledge Graph
Phase 5  Mature Tradable Asset Filter
Phase 6  Historical Event → Asset Outcome Dataset
Phase 7  Bayesian / Statistical Calibration
Phase 8  Real-Time Pricing Gap
Phase 9  Paper Trading
```

首先证明：

$$
P(
Lagger\ Repricing
\mid
Event,Relation,Market
)
$$

具有稳定 OOS Edge，再考虑自动执行。

------------------------------------------------------------------------

# 31. 接手 Agent 的行为要求

接手后必须遵守：

1. 不要把项目泛化成“做一个量化交易系统”。
2. 不要把早期 `Market × Sector multiplier` 当当前架构。
3. 不要把 HMM 当核心产品定义。
4. 不要默认技术指标越多越好。
5. 不要默认 LLM 直接预测涨跌。
6. 不要用 Bayesian 掩盖信息不足。
7. 不要默认 EMA、百分位或人工阈值必须存在。
8. 不要使用未来信息定义历史 State 或 Risk Mapping。
9. 不要修改 Base Strategy 来帮助 Exposure Controller 获胜。
10. 所有模型选择必须说明它改善的是哪个可验证问题。
11. 所有结论最终接受 Walk-Forward OOS。
12. 如果数据否定 H3/H4/H5，应接受否定，不得通过反复调参制造“漂亮结果”。

------------------------------------------------------------------------

# 32. 当前最值得立即回答的工程问题

接手 Session 首先应该回答：

> **怎样构造最小数据集，直接检验 `Information_t` 是否能够稳定条件化 Base Strategy 的未来收益/风险分布？**

需要明确：

```text
Base Strategy contract
Universe
Frequency
Feature / information schema
Market-level vs sector-level observations
Corporate-action handling
Suspension handling
ST / delisting handling
Limit-up / limit-down handling
Sector membership history
Survivorship-bias prevention
Return horizon
Conditional target definition
Training window
Representation baselines
Risk allocation constraints
Walk-forward protocol
Evaluation metrics
```

只有完成这些定义后，才应该大规模写状态模型代码。

------------------------------------------------------------------------

# 33. 最终交接：请区分“设计结论”与“实现候选”

当前最重要的三句话不是旧版的“大盘/板块乘数层级”，而是：

> **EventAlpha：允许原因是新的，但尽量要求结果载体是旧的。**

> **Risk Architecture：Base Strategy 独立产生 Alpha；市场信息只用于估计其 Conditional Return/Risk Distribution，不负责替代 Base Strategy。**

> **Decision Layer：Conditional Distribution 在风险偏好与约束下映射为 Risk Allocation，最终输出具体 Exposure。**

数学上：

$$
\boxed{
Information_t
\rightarrow
P(R_{strategy}\mid Information_t)
\rightarrow
RiskAllocation_t
\rightarrow
Exposure_t
}
$$

`HMM`、`HSMM`、`Clustering`、`Market State`、`Sector State`、`EMA`、`Percentile`、`MarketMultiplier`、`SectorMultiplier` 都只是可能的实现组件。

**不要让任何一个实现组件重新占据系统目标的位置。**
