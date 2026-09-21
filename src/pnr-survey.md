# 布局布线（P&R）文献综述：分开做还是一体化？

> 调研范围：2024–2026 年 placement / routing 相关综述与代表性前沿工作
> 核心问题：当前研究与工业实践中，布局与布线是「分开做」还是「一体化协同做」？

---

## 0. 结论速览（TL;DR）

**一句话结论：流程形式上仍是「先布局、后布线」两阶段；但在两个层面正在快速「一体化」——**

1. **学术界**：从「松散耦合」走向「紧密协同 / 统一优化」。出现了大量 routability-driven placement、timing-driven placement、可微分多目标布局、placement–routing co-optimization、甚至统一的 ADMM 联合求解（RUPlace），以及 RL/生成式端到端尝试。
2. **工业界**：主流仍是**分阶段串行流程**，但通过 **Shift-Left（左移）** 把布线信息前移到布局阶段做预测与约束——即「 **流程分离 + 信息融合**」，而非把两个阶段真正合并。

**可以这样概括当前态势：**

| 维度     | 现状                                                        |
| :----- | :-------------------------------------------------------- |
| 流程结构   | **仍是分离的**（placement → CTS → routing，顺序执行）                 |
| 阶段间信息流 | **正在融合**（布线拥塞/时序预测前移到布局，或布局反馈迭代）                          |
| 学术前沿   | **明显一体化**（统一优化、可微分、co-optimization、end-to-end）            |
| 工业主流   | **分离流程 + 强反馈迭代**（物理综合、concurrent optimization、Shift-Left） |
| 短期走向   | 不是「谁取代谁」，而是**边界模糊化**：融合发生在「信息与目标函数」层面，而非「工具阶段」层面          |

---


## 2. 主题分类：三条技术路线

### 路线一：传统分离流程（Separation）

- **代表**：A1（Physical Design 方法论）、A9（开源 EDA 全栈）、A10（ISPD 布线竞赛）

- **特征**：
  - placement → CTS → routing **严格顺序**
  - 布局目标主要是 **HPWL / 面积 / 密度**，布线**后置**
  - 布线在布局**完全冻结**后独立优化

- **定位**：这是工业主流流程的**骨架**，也是绝大多数工具链的实际执行方式

- **问题**：HPWL 只是线长近似， **无法反映真实路径与拥塞**；布局若只顾 HPWL，布线阶段常出现严重拥塞与频繁回退（B4 明确指出）

### 路线二：跨阶段协同（Cross-stage / Shift-Left）

- **代表**：A3（Shift-Left Survey）、A5（Learning-based PnR）、A4（工业 ML macro placement）、B5、B9

- **两种子策略（来自 A3，非常关键）** ：
  1. **Virtual Prototypes（虚拟样机）** ：用 ML/DL 模型**预测**下游（布线）行为——拥塞预测、时序预测——让早期阶段「看见」未来
  2. **Fused Actions（动作融合）** ：把后阶段动作**前移并与前阶段融合**——如 physically-aware synthesis、把全局布线信息引入全局布局

- **本质**： **流程阶段没有合并，但阶段间信息被打破隔离**。A3 称之为「从 divide-and-conquer 走向 fusion」

- **落地证据**：A4 显示 ML macro placement 在真实 GPU 设计上 **60% 的 block 达到或优于人工基线**，但**只有 30% 可产品化**——说明预测式协同有效但尚不稳定

### 路线三：一体化 / 统一优化（Unified / Co-optimization / End-to-end）

- **代表**：B1（C3PO）、B2（RUPlace）、B3、B4、B6、B7、B8

- **关键技术手段**：
  - **可微分化**：把 routing 的拥塞度量（如 RUDY）、时序（TNS/WNS）做成**对 cell 坐标可微**的目标，直接在布局梯度里优化（B1、B3、B6）
  - **统一建模求解**：ADMM 类框架把 placement 与 routing 写进**同一个优化问题**（B2）
  - **联合迭代反馈**：routing 结果反哺 placement，反复 refine（B4、B7）
  - **端到端学习**：GNN/RL 直接学习「可布线性感知」的布局（B5、B10、A2）

- **代表成果**：
  - B1（C3PO）：与商业工具**全流程对比**，routed wirelength 最多改善 16.7%、switching power 19.6%，且**放入真实商业 flow 验证**——这是「一体化」走向工业可用性的强信号
  - B2（RUPlace）：不再把 routability 当作布局的**间接指标**，而是统一优化，减少 DRV

---

## 3. 关键判断：到底「一起做」还是「分开做」？

### 判断一： **流程结构上，仍然是分开的**

- 所有综述（A1、A9、A10）描述的标准流程，placement 与 routing 都是**两个独立阶段**
- 工业工具（ICC2 / Fusion Compiler / Innovus）与开源流程（OpenROAD / OpenLane）也**按阶段串行执行**
- CTS 甚至**插在二者之间**，进一步说明阶段边界未消失

### 判断二： **信息与目标函数层面，正在强融合**

- 传统「分离」的痛点已被广泛承认： **HPWL 无法代表真实布线质量**（B4）
- 因此主流做法是**在布局阶段就引入布线知识**：
  - 预测拥塞图 → 约束布局（B5、B6、B9、A3）
  - 可微分拥塞/时序目标 → 进入布局梯度（B1、B3）
  - 统一优化 → 联合建模（B2、B8）

### 判断三： **学术前沿明显「一体化」，工业主流是「分离 + 强反馈」**

- 学术界： **一体化是明确趋势**（B1–B8 全部属于 co-optimization / unified / differentiable 方向）

- 工业界： **仍是分阶段**，但通过 **Shift-Left / concurrent optimization / physically-aware synthesis** 把信息前移

- A4 给出最有价值的工业侧证据：ML macro placement 结果**正面但不稳定**，落地受限于
  - 宏单元位置需要**在全局时钟/流水线规划前稳定**
  - 混合尺寸布局会**恶化 PDN（电源网络）鲁棒性**
  - 结果是**重复性/可复现性**要求高

- **结论**：工业界接受「信息融合」，但对「阶段合并」非常保守

### 判断四： **先进工艺与新场景，正在把二者「推向」一体化**

- 先进节点（7nm/5nm 及以下）：DRC、pin access、拥塞问题爆炸 → 布局若不感知布线基本不可行
- **3D IC / Chiplet**：placement 与 routing 在 native 3D EDA 中被认为需要**联合考虑**（A7、B6、B7）
- **模拟/模拟混合信号**：因约束强、floorplan 与 routing 高度耦合，一体化需求更迫切（A7、B4、B7）

---

## 4. 为什么「不能完全合并」？（一体化的现实阻力）

文献中反复出现的阻力（尤其 A4、A3、B1）：

1. **问题规模与复杂度**：联合优化变量数爆炸，精确求解不现实
2. **可微性难题**：路由是离散、规则驱动的，做可微分需要代理模型（RUDY 等），有精度损失
3. **预测模型泛化差**：数据驱动拥塞预测**跨设计/跨工艺泛化弱**（B1 明确批评）
4. **工业可复现性**：阶段合并后难以保证结果可复现与可控（A4）
5. **下游约束耦合**：PDN、时钟树、流水线规划等对 macro 位置敏感，限制了上游自由（A4）
6. **工具生态与接口**：过程分离便于工具模块化与数据交接（A3 指出 Fused Actions 会降低模块化）

---

## 5. 总结与展望

### 当前态势（2026）

> **流程阶段：分；优化目标与信息：合。**

- 学术前沿正在把 layout 的优化目标从「线长主导」升级为「 **时序 + 可布线性 + 线长**的**多目标、可微分、统一优化**」
- 工业实践采用「 **分离流程 + Shift-Left 信息前移**」，用预测与反馈拉近两阶段
- 二者是**同一方向的不同推进速度**，不是对立路线

### 未来趋势（基于 A2、A3、A7、B1 等）

1. **融合发生在「目标函数/预测模型」层**，而非「工具阶段」层——短期不会出现真正的单一 P&R 求解器替代分阶段流程
2. **可微分 + GPU + 多目标自动加权**（如 C3PO 的 MGDA 权重）成为布局主流技术
3. **Agentic / 生成式 EDA**（A2）会以「流程编排者」身份进一步模糊阶段边界
4. **先进节点 / 3D IC / 模拟**将成为一体化需求最强的牵引场景
5. **判断标准变化**：评价不再只看 placement 的 HPWL，而是看 **post-route 的 PPA / DRV**（B1 的 full-flow validation 是范式转变信号）

### 对深入学习者的建议

- 学 **placement 算法**时，务必同时理解它「 **为布线预判了什么**」（拥塞模型、时序估计）
- 学 **routing 算法**时，要理解它「 **如何反向约束布局**」（DRC、pin access、层分配）
- 关注三条主线： **可微分目标**、 **预测式反馈（Shift-Left）** 、 **统一优化（co-optimization）**

---

## 6. 英文名词速查（缩写与术语解释）

> 本文正文大量使用英文缩写与专有名词。这里按「类别」集中解释，**按需查阅即可**，不必通读。
> 每条格式为：**缩写**｜全称 → 中文含义 + 一句话说明。

### 6.1 流程与阶段（P&R 主干）

| 缩写 / 术语 | 全称 | 中文含义与说明 |
| :--- | :--- | :--- |
| **P&R / PnR** | Placement and Routing | **布局布线**。后端物理设计的核心统称，也是本文主题 |
| **Placement** | — | **布局**。给每个标准单元/宏单元找位置 |
| **Routing** | — | **布线**。用多层金属把该连的引脚连起来 |
| **Floorplan / Floorplanning** | — | **布局规划**（打地基）。定芯片尺寸、摆 SRAM/IP/模拟宏等"大块头" |
| **CTS** | Clock Tree Synthesis | **时钟树综合**。建一棵平衡的时钟分发树，让时钟同时、均匀到达所有触发器 |
| **Physical Design** | — | **物理设计**。把门级网表变成版图的全过程（floorplan→placement→CTS→routing→签核） |
| **post-route** | — | **布线之后（的）**。如 post-route 验证 = 在真实布线完成后再回头评价布局质量 |
| **flow** | — | **流程 / 工具流**。如 "商业 flow" = 工业界实际使用的整套工具链 |
| **full-flow validation** | — | **全流程验证**。本文特指把新方法放进完整商业流程里跑一遍再下结论 |
| **signoff** | — | **签核**。时序/功耗/DRC 全部达标的最终检查 |
| **split / merge 阶段** | — | 阶段「拆分 / 合并」，即本文的核心争论点 |

### 6.2 指标与度量（评价布局布线好坏）

| 缩写 / 术语 | 全称 | 中文含义与说明 |
| :--- | :--- | :--- |
| **HPWL** | Half-Perimeter Wire Length | **半周长线长**。用包围一个线网所有引脚的最小矩形的"半周长"来估算线长；便宜好用，但**只是直线近似**，反映不了真实走线与拥塞——这正是"布局布线分离"的根本痛点 |
| **wirelength** | — | **线长**。所有连线长度之和，传统布局的首要目标 |
| **routed wirelength** | — | **布线后线长**。真实布线完成后的线长，比 HPWL 可信得多 |
| **TNS** | Total Negative Slack | **总负裕量**。所有"迟到"路径的裕量（负数）之和，越小越好 |
| **WNS** | Worst Negative Slack | **最差负裕量**。最严重那条路径的裕量，是最关键的时序红线指标 |
| **PPA** | Power, Performance, Area | **功耗、性能、面积**。芯片设计的三大终极指标，也是评价"该不该一体化"的裁判 |
| **switching power** | — | **开关功耗**。信号翻转造成的动态功耗，与线长/电容强相关 |
| **congestion** | — | **拥塞**。某区域需要的布线资源超过可用资源；布线阶段最大的敌人 |
| **routability** | — | **可布线性**。这份布局"能不能布得下"的综合性质 |
| **DRV** | Design Rule Violation | **设计规则违例**。布线结果违反制造规则的次数，先进节点最重要的质量指标之一 |
| **DRC** | Design Rule Check | **设计规则检查**。最小线宽/间距/面积等制造规则的检查，违反即废片 |
| **pin access** | — | **引脚可达性**。先进节点下，标准单元引脚太密导致布线器"够不着"的问题 |
| **density** | — | **密度**。单元/布线资源的使用率，过高即拥塞 |
| **sub-10nm** | — | **10 纳米以下工艺节点**。代表最先进、约束最苛刻的一代工艺 |
| **block** | — | **设计块 / 模块**。一颗大芯片里相对独立的一块电路，可单独布局 |
| **benchmark** | — | **基准（测试用例）**。学界公平比较算法用的标准设计集合 |

### 6.3 算法与方法（本文的"武器库"）

| 缩写 / 术语 | 全称 | 中文含义与说明 |
| :--- | :--- | :--- |
| **ADMM** | Alternating Direction Method of Multipliers | **交替方向乘子法**。一种把大优化问题拆成若干子问题交替求解的数学框架；RUPlace 用它把布局与布线写进同一个问题 |
| **RL** | Reinforcement Learning | **强化学习**。让智能体通过试错学策略；本文中用于宏单元布局、整体布局等 |
| **ML** | Machine Learning | **机器学习** |
| **DL** | Deep Learning | **深度学习** |
| **AI** | Artificial Intelligence | **人工智能**（ML/DL 的上位概念） |
| **GNN** | Graph Neural Network | **图神经网络**。电路天然是图（单元=节点、线网=边），因此 GNN 很适合预测拥塞/可布线性/时序 |
| **RUDY** | Rectangular Uniform wire DensitY | **矩形均匀线密度**。一种**布线需求估算模型**：把每个线网的走线需求均匀摊到其包围盒上，得到一张二维"拥塞热力图"，供布局阶段提前避险；因形式简单而**天然可微**，故被广泛用作可微分布局的代理目标 |
| **MGDA** | Multiple-Gradient Descent Algorithm | **多重梯度下降算法**。把经典最速下降推广到**多目标**情形，求一个对所有目标都下降的公共方向，收敛到 Pareto 稳定点；C3PO 用它给 timing / routability / wirelength 自动定权重（免去手工调参） |
| **proxy model** | — | **代理模型**。真实布线是离散、规则驱动的，无法直接求导；用 RUDY 这类可微近似替代，代价是精度损失 |
| **differentiable** | — | **可微分的**。把目标写成对 cell 坐标可求导的形式，从而能用梯度法直接优化 |
| **co-optimization** | — | **协同优化 / 联合优化**。多个目标（或多个阶段）一起优化，而非先后串行 |
| **end-to-end** | — | **端到端的**。从输入直接学到输出，中间不显式分阶段 |
| **unified** | — | **统一的**。把原本分离的建模/求解合并为一体 |
| **refine / refinement** | — | **精修 / 迭代细化**。在已有解上反复小幅改进 |
| **iteration** | — | **迭代**。设计流程"打回重做"的本质特征 |
| **routability-driven placement** | — | **可布线性驱动布局**。把"能不能布下"作为布局的优化目标 |
| **timing-driven placement** | — | **时序驱动布局**。把"跑得够不够快"作为布局的优化目标 |
| **macro placement** | — | **宏单元布局**。摆放 SRAM、IP 核、模拟模块等大块头，是布局里最难、最像"下棋"的部分 |
| **post-layout estimation** | — | **版图后估算**。在版图完成前预测其性能/功耗 |
| **design space exploration** | — | **设计空间探索**。在众多设计选项中搜索最优组合 |
| **native 3D EDA** | — | **原生三维 EDA**。为 3D 堆叠芯片从头设计的工具，而非把 2D 工具硬套 |

### 6.4 范式与理念（本文的趋势判断用词）

| 缩写 / 术语 | 全称 | 中文含义与说明 |
| :--- | :--- | :--- |
| **Shift-Left** | — | **左移 / 任务前移**。把原本在流程**后段**才做的事（如布线预测）提前到**前段**（布局）去做。本文工业界主流策略的核心词 |
| **Virtual Prototypes** | — | **虚拟样机**。Shift-Left 的两大范式之一：用 ML/DL 模型**预测**下游行为（拥塞、时序），让早期阶段"看见未来" |
| **Fused Actions** | — | **动作融合**。Shift-Left 的另一范式：把后阶段动作**前移并与前阶段融合**，如物理感知综合 |
| **divide-and-conquer** | — | **分而治之**。传统策略：把大问题拆成小问题分别解决。A3 认为 EDA 正从它走向融合 |
| **fusion** | — | **融合**。与上一条相对，指打破阶段隔离 |
| **separation** | — | **分离**。流程阶段彼此独立、串行执行 |
| **concurrent optimization** | — | **并发优化 / 同步优化**。多个优化目标或阶段同时推进 |
| **physically-aware synthesis** | — | **物理感知综合**。在综合阶段就考虑物理布局/布线的影响 |
| **inherently interdependent** | — | **内在相互依赖**。B7 的原话，形容 floorplanning 与 routing 的关系 |
| **loose coupling** | — | **松散耦合**。两阶段只交换少量信息，是传统的协作方式 |
| **tight / close coupling** | — | **紧密耦合**。两阶段频繁交换信息、互相反馈 |
| **Agentic EDA** | — | **智能体化 EDA**。让 AI Agent 自主编排、执行 EDA 流程的新范式（A2） |
| **Human-in-the-Loop** | — | **人在回路**。关键决策仍由人把关，AI 做辅助与加速（A11） |
| **generative** | — | **生成式**。直接用模型"生成"布局/网表，而非逐步优化 |
| **black-box optimization** | — | **黑盒优化**。不知道目标函数解析形式时的优化方法 |
| **Pareto** | — | **帕累托（最优）**。多目标下"无法在不损害任一目标的前提下改进"的状态 |

### 6.5 电路与工艺基础

| 缩写 / 术语 | 全称 | 中文含义与说明 |
| :--- | :--- | :--- |
| **IC** | Integrated Circuit | **集成电路 / 芯片** |
| **VLSI** | Very Large Scale Integration | **超大规模集成（电路）**。动辄上亿门的芯片规模 |
| **EDA** | Electronic Design Automation | **电子设计自动化**。用软件工具完成芯片设计的整个产业 |
| **RTL** | Register Transfer Level | **寄存器传输级**。用硬件描述语言描述电路行为的抽象层次，是数字流程的入口 |
| **cell** | standard cell | **标准单元**。综合出的与门/或门/触发器等"等高积木"，布局的基本摆放对象 |
| **net / netlist** | — | **线网 / 网表**。net = 一组需要连在一起的引脚；netlist = "用哪些器件、怎么连"的清单 |
| **PDK** | Process Design Kit | **工艺设计套件**。晶圆厂提供的工艺规则与器件模型；不遵守就无法流片 |
| **GPU** | Graphics Processing Unit | **图形处理器**。既是本文 A4 的测试对象（真实 GPU 设计），也是加速布局求解的算力来源 |
| **PDN** | Power Delivery Network | **电源分配网络**。把电送到芯片各处的供电网；宏单元摆放不当会恶化它的鲁棒性 |
| **pin** | — | **引脚**。单元的对外连接点 |
| **via** | — | **过孔**。连接不同金属层的竖直通道 |
| **layer / 层分配** | metal layer | **金属层**。布线用的多层金属，相邻层走线方向通常互相垂直 |
| **discrete / 离散** | — | **离散的**。取值不连续、不可直接求导——布线的本质困难之一 |
| **analog IC** | — | **模拟集成电路**。约束强、手工设计为主，与数字流程完全不同 |
| **mixed-signal** | — | **混合信号**。一颗芯片里同时含模拟与数字电路 |
| **3D IC / Chiplet** | — | **三维集成芯片 / 芯粒**。把多颗裸片堆叠或拼装，布局与布线必须联合考虑 |
| **timing** | — | **时序**。信号能否在一个时钟周期内到达，是贯穿全流程的第一主线 |
| **clock** | — | **时钟**。全芯片的节拍信号 |
| **pipeline** | — | **流水线**。把长逻辑链切成多级以提高频率；其规划对宏单元位置很敏感 |

### 6.6 论文代号（B 类前沿工作）

| 代号 | 出处 | 全称 / 含义 |
| :--- | :--- | :--- |
| **C3PO** | ASP-DAC 2026 (NVIDIA) | 取自论文标题首字母：**C**ommercial-quality、**C**oherent、**C**oncurrent、**O**ptimization（商业级质量、连贯且并发的 timing/routability/wirelength 优化）。可微分布局代表工作，用 MGDA 自动加权，并做 full-flow 验证 |
| **RUPlace** | DAC 2025 | **R**outability + **U**nified + **Place**：用 ADMM 把布局与布线**统一**建模求解 |
| **RoutePlacer** | KDD 2024 | Route + Placer：用 GNN 做**端到端可布线性感知**的布局器 |
| **DCO-3D** | DAC 2025 | **D**ifferentiable **C**ongestion **O**ptimization for **3D** ICs：预测 3D 布线拥塞并反向修正布局 |
| **GrandPlan** | ACM 2026 | 可微分组目标 + 可布线性约束的**顶层同时规划** |

### 6.7 会议、期刊与机构

| 缩写 / 全称 | 类型 | 说明 |
| :--- | :--- | :--- |
| **ACM** | 学会 | Association for Computing Machinery，美国计算机学会；"ACM T-?" 指被某 ACM Transactions 期刊收录 |
| **IEEE** | 学会 | Institute of Electrical and Electronics Engineers，电气电子工程师学会 |
| **DAC** | 会议 | Design Automation Conference，设计自动化会议（EDA 领域顶会之一） |
| **ASP-DAC** | 会议 | Asia and South Pacific Design Automation Conference，亚太设计自动化会议 |
| **ISPD** | 会议 | International Symposium on Physical Design，国际物理设计研讨会（P&R 最对口） |
| **GLSVLSI** | 会议 | Great Lakes Symposium on VLSI，VLSI 领域老牌会议 |
| **KDD** | 会议 | ACM SIGKDD Conference on Knowledge Discovery and Data Mining，数据挖掘顶会 |
| **TCAD** | 期刊 | IEEE Transactions on Computer-Aided Design of Integrated Circuits and Systems |
| **IEEE Access** | 期刊 | IEEE 旗下开放获取综合期刊 |
| **MDPI** | 出版社 | Multidisciplinary Digital Publishing Institute，开放获取出版社（如 Electronics） |
| **Springer** | 出版社 | 国际学术出版社 |
| **arXiv** | 预印本平台 | 论文预印本网站（如 arXiv 2509.14551） |
| **JCTAM** | 期刊 | Journal of Theoretical, Computational and Applied Mechanics |
| **Integration (VLSI Journal)** | 期刊 | Elsevier 旗下 VLSI 集成相关期刊 |
| **NVIDIA** | 公司 | 英伟达；B1（C3PO）、B6（DCO-3D）等来自其 EDA 研究团队 |

### 6.8 工具与流程名

| 名称 | 类型 | 说明 |
| :--- | :--- | :--- |
| **ICC2** | 商业工具 | Synopsys IC Compiler II，工业级布局布线工具 |
| **Fusion Compiler** | 商业工具 | Synopsys 的 RTL-to-GDSII 统一实现平台（"融合"本身就是名字） |
| **Innovus** | 商业工具 | Cadence 的布局布线工具 |
| **OpenROAD** | 开源工具 | 开源 RTL-to-GDSII 全流程基础设施 |
| **OpenLane** | 开源流程 | 基于 OpenROAD 的开源自动化数字设计流程 |

### 6.9 文档里的其它英文小词

| 词 | 含义 |
| :--- | :--- |
| **TL;DR** | Too Long; Didn't Read → **太长不看**，即"一句话结论" |
| **Survey** | **综述**。系统梳理某个方向已有工作的论文类型 |
| **Invited** | **特邀（论文）**。会议特邀报告对应的论文 |
| **contest** | **竞赛**。如 ISPD 全球布线竞赛，其赛道题目常成为事实基准 |
| **state-of-the-art / SOTA** | **当前最优水平** |
| **routing demand** | **布线需求**。某处"想要走多少线"的估计量 |
| **gap** | **差距**。本文 5.2 表中的 "Gap" 指学术前沿与工业落地之间的落差 |
| **LPDDR** | Low Power Double Data Rate SDRAM，移动设备用低功耗内存 |
| **DRAM** | Dynamic Random Access Memory，动态随机存储器 |

> **阅读建议**：第一遍读正文时，遇到不认识的缩写可直接回查本表；三张最重要、出现频率最高的表是 **6.2（指标）**、**6.3（方法）** 和 **6.4（范式）**。

---

## 附：主要参考链接

- Physical Design: Methodologies and Developments — <https://arxiv.org/html/2409.04726v1>
- The Dawn of Agentic EDA — <https://arxiv.org/html/2512.23189v2>
- Shift-Left Techniques in EDA: A Survey — <https://arxiv.org/html/2509.14551v1>
- An Industrial Perspective on ML-Macro Placement Methods — <https://dl.acm.org/doi/full/10.1145/3716368.3735170>
- A Survey on Learning-Based PnR Optimization — <https://ieeexplore.ieee.org/document/11242165>
- Reinforcement Learning in ICs — <https://sciencedirect.com/science/article/pii/S0167926025001178>
- A Survey of M/DL Techniques in Analog IC Layout Synthesis — <https://mdpi.com/3042-5344/1/1/2>
- From RTL to Fabrication: Open-Source EDA Survey — <https://mdpi.com/2079-9292/15/5/1048>
- ISPD 2025 Global Routing Contest — <https://dl.acm.org/doi/10.1145/3698364.3715706>
- C3PO — <https://research.nvidia.com/labs/electronic-design-automation/publication/lu2026aspdac>
- RUPlace — <https://yibolin.com/publications/papers/PLACE_DAC2025_Chen.pdf>
- Differentiable Net-Moving & Local Congestion Mitigation — <https://dl.acm.org/doi/10.1109/DAC63849.2025.11133117>
- A Co-Optimization Method for Analog IC P&R — <https://mdpi.com/2079-9292/14/7/1349>
- RoutePlacer — <https://dl.acm.org/doi/10.1145/3637528.3671895>
- DCO-3D — <https://gtcad.gatech.edu/www/papers/Hsiao-DAC25-1.pdf>
- Advancing Routing-Awareness in Analog ICs Floorplanning — <https://arxiv.org/html/2510.15387v1>
- GNN-Based Placement Optimization Guidance — <https://mdpi.com/2079-9292/14/2/329>
- OpenROAD — <https://openroad.readthedocs.io/en/latest/main/README.html>


## 1. 文献清单（22 篇，2024–2026）

### A 类：综述 / 综述性论文（12 篇）

| #   | 标题                                                                                   | 年份        | 出处                            | 与 P&R 关系                                                                 |
| :-- | :----------------------------------------------------------------------------------- | :-------- | :---------------------------- | :------------------------------------------------------------------------ |
| A1  | **Physical Design: Methodologies and Developments**                                  | 2024      | arXiv 2409.04726              | 系统梳理物理设计全流程（floorplan→placement→CTS→routing→验证）的传统**分阶段**方法论              |
| A2  | **The Dawn of Agentic EDA: A Survey of Autonomous Digital Chip Design**              | 2025/2026 | arXiv 2512.23189              | 综述生成式/智能体 EDA，含物理设计范式图（RL 把 placement 当序列博弈、扩散模型生成布局）                     |
| A3  | **Shift-Left Techniques in Electronic Design Automation: A Survey**                  | 2025/2026 | arXiv 2509.14551；ACM T-? 2026 | **最直接相关**：formalize「任务前移/融合」，提出 Virtual Prototypes + Fused Actions 两大范式   |
| A4  | **An Industrial Perspective on ML-Macro Placement Methods**                          | 2025      | GLSVLSI 2025 (ACM)            | **工业实证**：ML/RL macro placement 在真实 GPU 设计上的效果与落地障碍                        |
| A5  | **A Survey on Learning-Based PnR Optimization for VLSI Physical Design Automation**  | 2025      | IEEE Access                   | 直接以 PnR 为对象，综述传统算法 + 学习型优化                                                |
| A6  | **Reinforcement Learning in Integrated Circuits: Design, Optimization and Security** | 2025      | Integration (VLSI Journal)    | 综述 RL 在 IC（含 macro placement / chip placement / layout）的应用                |
| A7  | **A Survey of Machine and Deep Learning Techniques in Analog IC Layout Synthesis**   | 2025      | MDPI (review)                 | 覆盖 analog placement、routing、post-layout estimation，指出二者**内在相互依赖**         |
| A8  | **AI/ML for VLSI Chip Design Automation: A Comprehensive Survey**                    | 2026      | Springer                      | 覆盖设计空间探索、physical design 优化、性能/功耗预测                                       |
| A9  | **From RTL to Fabrication: Survey of Open-Source EDA Tools and PDKs**                | 2026      | MDPI Electronics              | 综述开源 EDA 全栈，明确把 floorplan/placement/routing 作为**顺序阶段**（OpenLane/OpenROAD） |
| A10 | **Invited: ISPD 2025 Performance-Driven Large Scale Global Routing Contest**         | 2025      | ISPD 2025 (ACM)               | 布线侧最新基准与趋势：从 ISPD placement 基准派生路由基准，接入 OpenROAD 评估                       |
| A11 | **Human-in-the-Loop AI EDA for Chip Design**                                         | 2026      | JCTAM                         | 以 placement & routing workflow 为例，讨论人在回路与预测/反馈融合                          |
| A12 | **Artificial Intelligence in VLSI Physical Design ... Optimize PPA**                 | 2024      | (conference/journal)          | AI 用于 physical design 各阶段（含 P&R）的综述                                      |

### B 类：代表性前沿工作（10 篇，用于佐证「一体化」趋势）

| #   | 标题                                                                                                                      | 年份        | 出处                    | 一体化体现                                                                      |
| :-- | :---------------------------------------------------------------------------------------------------------------------- | :-------- | :-------------------- | :------------------------------------------------------------------------- |
| B1  | **C3PO: Commercial-Quality Global Placement via Coherent, Concurrent Timing, Routability, and Wirelength Optimization** | 2026      | ASP-DAC 2026 (NVIDIA) | 可微分全局布局， **同时**优化 timing + routability + wirelength，用真实商业流程做 post-route 验证 |
| B2  | **RUPlace: Optimizing Routability via Unified Placement and Routing**                                                   | 2025      | DAC 2025              | ADMM 框架**统一** placement 与 routing 的建模与求解                                   |
| B3  | **Differentiable Net-Moving and Local Congestion Mitigation for Routability-Driven Global Placement**                   | 2025      | DAC 2025              | 可微分布线拥塞目标直接进入布局优化，DRV 平均降 40%                                              |
| B4  | **A Co-Optimization Method for Analog IC Placement and Routing Based on Sequence Pairs and Random Forests**             | 2025      | MDPI Electronics      | 迭代式 P&R **联合优化**，用 routing 反馈驱动 placement refinement                      |
| B5  | **RoutePlacer: An End-to-End Routability-Aware Placer with Graph Neural Networks**                                      | 2024      | KDD 2024              | 端到端可布线性感知布局，用 GNN 预测可布线性                                                   |
| B6  | **DCO-3D: Differentiable Congestion Optimization in 3D ICs**                                                            | 2025      | DAC 2025              | 预测 post-3D-routing 拥塞图并**反向修正 placement**                                  |
| B7  | **Advancing Routing-Awareness in Analog ICs Floorplanning**                                                             | 2025      | arXiv 2510.15387      | 明确指出 floorplanning 与 routing「inherently interdependent」，做路由感知布局            |
| B8  | **GrandPlan: Differentiable, Simultaneous Top-Level ...**                                                               | 2026      | ACM                   | 可微分组目标 + 可布线性约束的顶层同时规划                                                     |
| B9  | **GNN-Based Placement Optimization Guidance Framework**                                                                 | 2025      | MDPI Electronics      | 图学习预测物理/时序指标， **反馈进工具流**改善布局质量                                             |
| B10 | **An Updated Assessment of RL for Macro Placement**                                                                     | 2024/2025 | IEEE TCAD             | 对 RL macro placement 的严格基准重评，含 sub-10nm 公开基准                               |

> 说明：A 类为「综述性」文献主证据；B 类为代表 2024–2026 年最相关的**前沿方法**，用于支撑趋势判断。合计 22 篇，满足「不少于 20 篇」要求。

---
