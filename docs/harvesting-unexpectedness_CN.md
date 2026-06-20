# 收割意外：用于人工智能生成发现的双重阈值过滤器

**LIU TENGJIAO** (psi@psi.run)  
*psi.run 创始人兼研究员*

> **摘要**：AI 系统经常给出人类想不到、但验证后确实正确的解。我们把这些解当作一种新的科学材料，而不是要过滤掉的噪声。本文提出一个可操作的五步流程——生成、筛选、验证、语义桥接与整合——通过双重阈值（意外程度 + 硬验证）把有价值的意外从噪声中分离出来。我们通过 FunSearch、AlphaFold、AlphaGo 和 2026 年平面单距离猜想的 AI 生成反例，重构了相似的过滤动力学如何在已有案例中发挥作用，并为自动化发现平台提供了一套可落地的意外收割方法。
> 
> **关键词**：人工智能偶得性；目标约束意外；闭环流形约束；局部马氏距离；人机交互语义桥接；意外价值论；收割协议

---

## 1. 引言：目的论困境与目标锁定

科学教育通常告诉我们：设定目标，制定计划，达成目标。但真正的突破很少遵循这种方式。Stanley 和 Lehman (2015) 将其称为**“目标悖论”（The Paradox of the Objective）**——过度追逐既定目标反而会让你错失更好的答案。一味追求预设指标会使探索活动收敛于局部最优点，导致科学研究陷入保守与均质化。

随着深度强化学习、大语言模型及形式化定理证明器在科学发现（AI4S）中的应用，系统频繁输出超出人类专家直觉的“异常解”。本研究的核心主张是：**AI 系统在目标约束下产生了偏离人类历史解法经验分布（empirical distribution of human solution strategies）的有效结果，我们可以系统性收集并提炼这些结果，作为新型认识论资源反哺科学发现。** 我们将本研究定位为方法论框架与立场论文，旨在建立目标约束意外的协议规范，而非呈现已完成的实证验证。

我们提出三个简单的观点：
1. AI 能够发现超出人类习惯的解答（out-of-distribution relative to human solution corpus）。
2. 这些解答中有一部分是绝对正确的——它们通过了硬性验证函数 $V(x|T) = 1$ 并改进了目标指标。
3. 我们能构建一个流水线来捕捉并利用它们。

本文作为 Agent IP 资产化与具象化系列研究的第四篇论文，为该系列提供了一个方法论框架。第一篇论文（*Agent Concretization: Informational Boundaries and Persistent Agent IP*）提出了使用 Schema Sandbox 构建信息边界以约束认知漂移（Cognitive Drift）。而本论文则处理其逆向命题——如何缓解人类自身的认知固化（Cognitive Entrenchment）。AI 沙箱负责约束系统的行为漂移，而 AI 意外探索则负责帮助揭示固化假设。此外，前序研究中作为行为约束边界的 Schema Sandbox（$\Omega_t$），在本协议中被特化并演进为目标验证函数 $V(x|T)$，这表明用于保护系统身份持久化的约束机制，同样可被复用为筛选科学意外的真值过滤器。

---

## 2. 相关工作：从“无目标探索”到“目标约束意外”

在人工智能与演化计算领域，如何引导系统产生新奇行为已有丰富的研究：

1. **新奇性搜索（Novelty Search）**：Lehman 与 Stanley (2011) 提出放弃对目标的直接优化，转而显式奖励系统产生在行为特征空间中与已知个体差异最大的新奇解。
2. **质量-多样性算法（Quality-Diversity, QD）**：Cully 等人 (2015) 提出了 QD 算法（如 MAP-Elites）。其核心在于在特征空间的各个局部区域内同时最大化个体质量，从而维持一个高性能且多样化的候选池。
3. **开放式演化（Open-Endedness）**：Stanley 等人 (2019) 探讨了如何构建能够无限产生全新且有意义的实体与行为的计算系统。
4. **生成式科学发现框架与溯因引擎**：近年来，关于生成式 AI 科学发现框架的研究（如 Reddy & Shojaee, 2024; arXiv:2412.11427）系统探讨了 AI 系统赋能的科学探索闭环。James Evans 等学者提出将 AI 系统视为一种**“溯因引擎”（Abductive Engine）**。他们主张人类负责高层的科学溯因与概念跃迁，而 AI 系统负责在大规模高维空间中进行“意外模式检测”（Surprise Pattern Detection）。这为工具驱动的偶得性（Tool-driven Serendipity）提供了深刻的计算科学支撑。

本框架的差异性定位：  
上述工作多聚焦于“无目标”（Objective-free）或“多目标弱约束”的开散式探索。而本研究提出的“意外价值论”聚焦于**“目标约束意外”（Target-Constrained Unexpectedness, $U_{TC}$）**。我们探索的起点是一个硬性的、收敛的真值目标（Target）。本框架通过非人类惯性搜索轨迹在实现该目标的过程中提取那些偏离人类直觉路径的“非共识中间态”。这是一种“在强引力场约束下的弹弓效应”，与无目标的纯发散搜索有着方法论区别。

---

## 3. 问题提出：认知固化与闭环流形约束假说

为了论证主动收集意外的必要性，我们需要形式化人类知识生产的系统性瓶颈。

### 3.1 认知固化（Cognitive Entrenchment）
人类专家随着专业深度之增加，其大脑底层的认知图式（Cognitive Schemas）趋于高度稳定（Dane, 2010）。专家习惯于快速检索既有的标准步骤序列。其代价是对范式外反常信号的系统性滤波与忽视。

### 3.2 闭环流形约束假说（Closed-Manifold Constraint Hypothesis）
我们提出以下**“闭环流形约束假说”**作为工作假设（Working Hypothesis）：

> **假说定义**：若人类的知识探索轨迹受限于生理感知低维偏置、符号语言语义边界 $\mathcal{B}_H$ 以及库恩（Kuhn, 1962）所描述的常规科学范式共识制度，则该搜索空间可被建模为一个有限的、自给自足的流形（Closed Manifold） $\mathcal{M}_H$。

人类习惯在已知的小地图内思考，我们称之为 $\mathcal{M}_H$。AI 系统引导的搜索能够脱离这一已有的经验流形进行轨迹采样。一个想法偏离这张地图越远，人类团队自己发现它的概率就越低。

人类可以产生异常。问题在于效率。在高维、跨领域的空间中，工作记忆限制、学科壁垒和声誉风险使得非共识搜索在结构上非常昂贵。

### 3.3 假说的可检验路径与表征偏置的缓解
为了使该工作假设具备硬科学的可证伪性，我们设计了两条可检验路径：
1. **流形边界拓扑映射与重建误差度量（Manifold Boundary Mapping via Reconstruction Error）**：研究者可采用高维流形学习算法（如 UMAP 或 t-SNE），对 arXiv 或 Semantic Scholar 的学术文献嵌入向量（Embeddings）进行投影，以量化 $\mathcal{M}_H$ 的拓扑半径。
   *表征偏置缓解*：由于预训练语义编码器（如 Sentence-Transformers）是在人类既有语料上训练的，其表征空间固有地带有一种“人类语料偏置”（Human-corpus Bias），可能强行将真正的模型生成的 OOD 候选解拉回到已知流形中。为了缓解这一偏置，我们在原型收割平台中引入了双轨机制：除了基于语义特征的局部距离测度外，还引入了**无监督自编码器（Autoencoder）重建误差**与**自监督流形微调（Self-Supervised Manifold Fine-Tuning）**。对于任何输入解 $x$，通过计算其在大样本人类基准语料上训练 of 生成式重建误差（Reconstruction Loss），系统能够在不依赖特定人类语义编码的坐标系下，以纯粹的统计信息熵来量化其偏离 $\mathcal{M}_H$ 的绝对程度。
2. **集体盲点收敛速度测量（Topological Convergence of Collective Blindspots）**：利用引文网络与跨领域术语共现矩阵，通过计算图的**谱隙（Spectral Gaps）与模块度（Modularity）**，测量人类学术社群知识流动的耗散率与收敛速度。若谱隙随时间递减，则表明学科共识惯性正在加速集体盲点的收敛与锁死。

---

## 4. 机制：AI 非人类中心搜索轨迹的计算生成机理

AI 系统能够生成偏离人类历史解法分布的有效解，源于以下三种机制的叠加：

1. **高维潜在空间表征**：神经网络将不同学科的符号信息压缩并嵌入到极高维的连续潜在空间中，抹平了人类学术分类的符号樊篱，实现跨域关联。
2. **去社会化的探索性采样**：算法在强化学习（如自博弈）与生成模型温度调度中，不受人类常识与社会声誉约束，得以在低先验概率拓扑分支上进行饱和采样。
3. **工具与定理证明器加持**：当探索流形与精确定理证明器（如 Lean 4）或计算机代数系统（CAS）进行闭环连接时，模型可以在远离人类逻辑直觉的区域疯狂试错，同时由形式化系统保障逻辑不变量。

---

## 5. 价值论：目标约束意外与高熵噪声的判别模型

我们对“价值”的定义很简单：它是否通过了验证测试，以及它是否对目标有所帮助？这使我们能够避开关于价值本质的哲学争论，转而使用以下三层判别模型：

1. **Level 1：惊奇度（Surprisal）**：在人类既有假说 $H$ 下的香农信息量，度量事件的罕见度。在具体实践中，针对单个候选解 $x$，我们使用目标推理模型在 Token 序列层面的**负对数似然（Negative Log-Likelihood）**作为可计算代理指标：
   $$I(x) = -\sum_{i=1}^{M} \log_2 P(x_i \mid x_{<i}, H)$$
   *对齐规训偏差修正*：现代推理模型普遍经过了严格的人类反馈强化学习（RLHF）和监督微调（SFT），这会导致明显的模式坍缩（Mode Collapse）。被规训后的模型在概率分布上会严重偏向人类的共识与安全句式，使得真正的非人类中心意外可能被赋予极高的 NLL（即高于 $\theta_{\text{high}}$）而遭遇过滤。
   为量化修正此偏置，在收割协议的工程实现中，我们使用**双轨似然度对照**：同时引入未经 RLHF 规训的**基座模型（Base Model）** $H_{\text{base}}$ 来计算 $I(x)$，或在生成采样时引入“去规训熵增采样（De-biased Entropy Sampling）”，确保概率似然计算能够反映真实的客观不确定性。
   *工程定位*：**Level 1 在系统设计中充当低成本的预筛选过滤器（Pre-filter）**。由于在高维潜在空间中计算 embedding 距离和执行形式化验证计算开销巨大，系统通过双向区间阈值限缩：
   $$\theta_{\text{low}} < I(x) < \theta_{\text{high}}$$
   这可以剔除那些过于平庸（NLL 低于 $\theta_{\text{low}}$）或极端混沌无序（NLL 高于 $\theta_{\text{high}}$）的碎片，从而在漏斗顶端保护计算资源。
2. **Level 2：贝叶斯惊奇（Bayesian Surprise）**：数据 $\mathbf{D}$ 导致先验信念流形 $P(\boldsymbol{\vartheta})$ 向后验信念流形 $P(\boldsymbol{\vartheta}|\mathbf{D})$ 发生改变的变分发散度。
   *升级为混合高斯局部马氏距离（GMM-based Localized Mahalanobis Distance）*：虽然贝叶斯惊奇可以在单峰高斯假设下通过全局马氏距离进行近似，但这在科学先验知识流形 $\mathcal{M}_H$ 高度非线性、多峰且断裂（不同学科呈现分散的局部子流形簇）时失效。全局均值会误将跨领域的平庸共识判定为高惊奇，而漏掉单一学科极深处的真正突破。为此，我们将其升级为局部马氏距离：
   $$S_{Bayes}(x) \approx \min_{k \in \{1, \dots, K\}} d_{\text{Mahalanobis}}(\mathbf{e}(x), \boldsymbol{\mu}_k, \boldsymbol{\Sigma}_k)$$
   其中，$\boldsymbol{\mu}_k$ 与 $\boldsymbol{\Sigma}_k$ 是通过对人类科学文献库 $\mathcal{M}_H$ 在潜在表征空间中进行混合高斯模型（GMM）期望最大化（EM）聚类所识别出的 $K$ 个局部学科子流形中心的均值与协方差矩阵。
3. **Level 3：目标约束意外（Target-Constrained Unexpectedness, $U_{TC}$）**：
   结合上述定义，有价值的意外解必须同时满足惊奇度区间预筛、高贝叶斯惊奇，并最终通过特定目标任务 $T$ 的形式化验证函数 $V(x|T)$（通过为 1，未通过为 0）：
   $$U_{TC}(x) = S_{Bayes}(x) \cdot \mathbb{1}[V(x|T) = 1] \cdot \mathbb{1}[\theta_{\text{low}} < I(x) < \theta_{\text{high}}]$$
   其中 $\mathbb{1}$ 为示性函数。这里，若一个解具有极高的 $S_{Bayes}$ 但验证函数 $V(x|T) = 0$（说明未满足目标约束）或 NLL 偏离上述双向区间，则系统判定其为**幻觉噪声**或无意义碎片；当且仅当 $U_{TC} > 0$ 时，该解才被判定为具有运行化价值的目标约束意外。这与大脑底层的双向多巴胺预测误差（RPE）奖励神经机制（Schultz et al., 1997）高度相似，并映射到脑科学中最小化认知不确定性的**自由能原理（FEP）**（Friston, 2010），其中预测误差驱动突触权重更新以自适应演化先验流形。

> [!NOTE]
> These quantities are not proposed as universal measures of scientific value; they are engineering proxies for ranking candidates before expensive verification.
> 
> 在本协议中，RLHF 偏差修正防止了对齐模型过滤掉真正的分布外发现，而 GMM 局部化则避免了跨学科的平庸共识被错误地标注为高惊奇。

在系统级过滤中，三层模型的层级过滤逻辑与空间关系如图 1 所示：

![图 1：目标约束意外的三层判别与过滤空间关系](figure_1_three_level_model.jpg)

### 5.1 实证阐释
这个案例的映射很直接。模型没有沿着离散几何学家习惯的格点思路走，而是直接跳到代数数论里的 Golod-Shafarevich 塔结构——这在人类离散几何的经验分布里几乎不存在。在我们的框架下，此类结果应处于平庸共识与无序噪声之间的中间区域。该结构经数学家团队在预印本中进行了消化与人工验证，推翻了平面单距离猜想的下界（Alon et al., 2026; Sawin, 2026）。

---

## 6. 方法论：意外收割协议

### 6.1 协议五步形式化定义与量化出口

#### 1. 生成（Generation）
我们向模型注入对抗性提示词（Adversarial Prompts），强制其在偏离共识先验的区域采样。我们引入动态温度调度控制采样轨迹的多样性：
$$\tau_t = \tau_{high} \cdot e^{-\lambda t} + \tau_{low}$$
获取包含 $N$ 个候选解的集合 $\mathcal{C}$（设 $\tau_{high}=1.4 \to \tau_{low}=0.3$）。

*对抗性提示词模板与 {Standard_Moves_List} 构造：*
`{Standard_Moves_List}` 的构造源于领域专家的经验逆向沉淀或文献共现网络的高频算子提取。例如，在数论证明中，它包含 `[prime sieve, mathematical induction]`；在离散几何中，它包含 `[square lattices, triangular tiling]`。
```
[Role] You are an OOD (Out-of-Distribution) mathematical reasoning agent.
[Task] Synthesize a proof for Conjecture T under target constraint C.
[Constraints] You MUST explicitly bypass the following standard sequence of moves: {Standard_Moves_List}. Do not rely on prime sieve methods or grid-like spatial lattices.
[Directive] Traverse the topological boundaries of the latent space and construct an alternative representation by exapting tools from {Disparate_Domain_D}.
```

#### 2. 筛选（Screening）
我们对集合 $\mathcal{C}$ 中的每个候选解 $x$ 进行联合过滤，剔除平庸常识与不可还原的无序白噪声：
$$\text{Keep } x \iff S_{Bayes}(x) > \theta_1 \quad \text{and} \quad \theta_{\text{low}} < I(x) < \theta_{\text{high}}$$
获得候选子集 $\mathcal{C}_{\text{screened}}$。

#### 3. 验证（Verification）
我们将 $\mathcal{C}_{\text{screened}}$ 送入硬边界形式化验证系统，过滤出物理或逻辑上完全可行的解。在此环节，前序研究中的 Schema Sandbox ($\Omega_t$) 被具体实现为自动定理证明器编译器或仿真物理环境：
$$\mathcal{C}_{\text{verified}} = \{ x \in \mathcal{C}_{\text{screened}} \mid V(x|T) = 1 \}$$
获得绝对正确的非共识事实集 $\mathcal{C}_{\text{verified}}$。

#### 4. 语义桥接（翻译）
对于复杂的符号解或大规模神经网络权重激活，程序合成与符号回归可以提取出核心机制的最小逻辑核。然而，将提取出的逻辑核转化为人类科学界普遍接受的抽象理论（例如将离散几何反例关联到 Golod-Shafarevich 塔理论上）仍然是纯自动化的瓶颈。我们称这一环节为语义桥接步骤。在这里，算法负责自动化的符号约简，而人类专家负责概念层面的验证并将其整合入既有的科学范式。

#### 5. 整合（Integration）
我们将翻译出的最小逻辑核作为新公理并入人类先验语料库，通过自适应学习率 $\gamma_t$ 迭代认知版图：
$$\text{Prior}_{t+1} = \text{Prior}_t + \gamma_t \cdot \text{Kernel}$$
$$\gamma_t = \gamma_0 \cdot \text{Sim}(\text{Kernel}, \text{Prior}_t) \cdot f(\text{Peer Acceptance})$$
其中，$f(\text{Peer Acceptance})$ 在工程上被定义为通过去中心化科学共识证明（DeSci Consensus Proof）或学术同行评议网络给出的专家接受度归一化评分，取值区间为 $[0, 1]$。

**收割价值的去向与积累沉淀（Accumulation & Destination）**：
收割出的最小逻辑核与验证有效的意外解将被沉淀并并入以下三类新型认知基础设施中：
1. **先验科学数据库（Prior Databases）**：系统自动将硬验证通过的代数界限、代码构造或分子结构自动写入特定的共享数据库中（例如数论领域的 OEIS 数据库或生物大分子结构库），作为后续研究的新基准。
2. **微调与自演化循环（Fine-Tuning Loops）**：系统自动将收集到的有效 OOD 意外解反馈加入后续模型的训练集中，使用微调（Fine-Tuning）更新基础模型的先验概率分布，实现模型认知能力的螺旋上升。
3. **长期记忆系统（Long-Term Memory）**：系统自动将逻辑核转换为结构化记忆，注入持续运行的科研探索智能体的外部记忆检索模块中，闭环指导后续的生成轨迹。

为了评估整个协议的端到端效能，我们定义**意外收割复合产率（Unexpectedness Harvesting Yield, UHY）**指标：
$$UHY = \frac{\sum_{x \in \mathcal{C}_{\text{verified}}} U_{TC}(x)}{|\mathcal{C}_{\text{generated}}| \cdot \text{ComputeCost}}$$
其中，`ComputeCost` 的单位定义为**标准化 GPU 小时（Normalized GPU-hours）**。我们将 Baseline 搜索的计算消耗归一化为 1.0，各方案相对于此基准计算相对计算消耗因子。

收割协议的端到端过滤漏斗与量化指标流向如图 2 所示：

![图 2：意外收割协议（Harvesting Protocol）的五步漏斗与量化指标](figure_2_harvesting_funnel.jpg)

### 6.2 协议运行伪代码
```python
def harvesting_protocol(target_T, prior_t, theta_1, theta_low, theta_high):
    # Step 1: 从模型中以温度调度生成候选解
    candidates = generate_with_adversarial_priors(target_T, prior_t, temp_range=(1.4, 0.3))
    
    screened_subset = []
    # Step 2: 双阈值联合区间筛选 (Level 1 与 Level 2)
    for x in candidates:
        # prior_t 代表人类既有知识先验，已完成 GMM 混合高斯流形构建
        surprise = compute_bayesian_surprise_distance_gmm(x, prior_t) 
        
        # 为抵消 RLHF 带来的模式坍缩与对齐偏置，此处的似然概率计算使用未对齐的基座模型 H_base
        nll = compute_token_nll(x, model="H_base")
        
        if surprise > theta_1 and (theta_low < nll < theta_high):
            screened_subset.append(x)
            
    # Step 3: 在硬性目标约束下进行验证 (Schema Sandbox 的具体形式化编译器/验证环境)
    verified_subset = []
    for x in screened_subset:
        if formal_verification(x, target_T) == 1:
            verified_subset.append(x)
            
    # Step 4: 语义桥接步骤与翻译 (Semantic Bridging)
    translated_kernels = []
    for x in verified_subset:
        # 1. 算法自动执行符号约简与程序蒸馏 (提取最小符号结构)
        raw_kernel = automated_program_distill(x) 
        # 2. 人类专家介入，执行概念对齐与形式化回填证明 (Human Backfill)
        kernel = interactive_semantic_bridging(raw_kernel)
        translated_kernels.append(kernel)
        
    # Step 5: 自适应认知图谱与平台知识库整合
    prior_next = integrate_to_priors(prior_t, translated_kernels, base_rate=0.1)
    
    # 沉淀至三类认知基础设施：
    # prior_databases.update(verified_subset)
    # tuning_dataset.append(verified_subset)
    # agent_memory.inject(translated_kernels)
    
    # 计算意外收割复合产率 (UHY) 指标
    compute_cost = measure_normalized_gpu_hours(candidates)
    total_u_tc = sum(compute_u_tc_metric(x, prior_t) for x in verified_subset)
    uhy = total_u_tc / (len(candidates) * compute_cost)
    
    return prior_next, uhy
```

---

## 7. 案例解构：真实突破中的事实还原与同行评价

我们完全基于前沿学术界已发表的真实科学突破和人机对弈案例，采用“事实还原”与“同行评价”分离的逻辑链进行解构。

### 7.1 真实个案一：FunSearch 发现 Cap Set 新构造（Nature, 2024）
*   **事实还原**：Romera-Paredes 等人 (2024) 提出了 FunSearch 系统，将大语言模型与程序评估器相连。模型并非直接生成数学对象，而是搜索生成数学对象的 Python 程序。FunSearch 成功发现了新的 Cap Set 构造，将 $n=8$ 时的下界从此前的 496 提升至 512，刷新了该问题的下界记录 [https://doi.org/10.1038/s41586-023-06924-6]。
*   **同行评价**：同行指出，FunSearch 输出的并非不可解译的神经网络黑箱权重，而是清晰、可读的算法程序。这极大地便利了数学家对其底层的代数与组合逻辑进行逆向翻译与分析，将其转化为数论研究的新先验。

### 7.2 真实个案二：AlphaFold 2 绕过分子动力学模拟（Nature, 2021）
*   **事实还原**：AlphaFold 2 (Jumper et al., 2021) 绕过了显式物理动力学模拟路径，直接基于多序列比对（MSA）与注意力机制，从序列和演化数据中学习结构约束，在 CASP14 评估中实现 GDT 分数超过 90% 的极高准确度。
*   **同行评价**：结构生物学家在评述中指出，AlphaFold 2 表明，从序列和演化数据中学习的几何约束在 CASP14 基准测试中可以超越许多传统的结构预测流程，绕过了显式分子动力学模拟路径。这迫使物理学家重新审视力学建模的抽象精度，并反思如何将统计约束与分子动力学进行更深层的融合。

### 7.3 真实个案三：AlphaGo's 创新落子（2016年）
*   **事实还原**：在 2016 年 AlphaGo 与李世石的对局（第37手）中，AI 下出了五路肩冲等严重违背人类传统棋理定式的棋着，这是通过自博弈价值优化生成的动作。
*   **同行评价**：专业评论员将第 37 手描述为极具创意与不寻常的落子；包括柯洁在内的后续棋手指出，AlphaGo 彻底改变了人类的围棋理论。系统落子跳出了人类思维的盲区，重塑了全球职业围棋界的布局范式。

### 7.4 真实个案四：2026年平面单距离猜想反例的生成与验证（arXiv, 2026）
*   **事实还原**：在 2026 年 5 月提交至 arXiv 的 Alon 等人的论文（arXiv:2605.20695，另见 Sawin arXiv:2605.20579）中，合作者们报告了一个关于平面单距离反例的代数数论构想（accessed June 2026）。该构想最早被报道源于 OpenAI 的一个内部推理模型，随后由人类数学家团队在预印本中进行消化、验证与公式化提炼。该方法使用 Golod-Shafarevich 塔构造了一系列具有 bounded root discriminants 的 CM 代数数域，基于这些数域的根在平面上映射出一族特殊的点集，推翻了此前主导猜想的界限，得到了 $n^{1+\delta}\ (\delta \approx 0.014)$ 的下界证明。
*   **同行评价**：伴随的科学反思将该结果描述为：在一个长期由几何直觉主导的问题中，引入了代数数论的构造。这迫使数学家重新审视高维代数结构与平面度量几何之间的深刻联系。

---

## 8. 实证落地、应用与局限性讨论

### 8.1 提议的 Pilot 设计
由于目前缺乏已审计的完整运行日志，本节仅提出一个可复现的 pilot 设计框架，用于评估双阈值筛选对下游验证计算资源的保护效果，而非报告已完成的实验结果。这些基准指标主要用于指导协议设计，而非作为实测证明。

提议的实验设置：该实验设计规划运行在含有定理证明验证器（基于 Python 几何代数解算器与 Lean 4.8.0-rc1 编译器沙箱）的闭环探索环境下。生成模型采用 `gpt-4o-2024-05-13`。设计规划了 10 次独立的并行探索运行，每次运行均使用第 6.1 节定义的对抗性提示词模板，温度按余弦退火调度从 $\tau_{high}=1.4$ 降至 $\tau_{low}=0.3$。在整型边长 $a,b,c \le 100$ 的约束下，要求所有面对角线也为整数。

具体验证机制如下：
1. **生成效率监控**：定量计算单位标准化 GPU 小时内生成符合基本棱长限制的非平庸候选数。
2. **过滤漏斗消减率测量**：在 Level 1（模型 Token 级 NLL 区间）与 Level 2（局部 GMM 马氏距离）双阈值过滤器的联动作用下，计算进入后置硬验证的候选解缩减率，以此证明双阈值对算力开销的控制效果。
3. **复合产率（UHY）计算**：通过统计最终通过 Lean 形式化定理证明器（$V(x|T)=1$）验证的有效意外数量，测算端到端意外收割产率。

在此基础上，我们规划了两条更具前瞻性的实验路线图：
1. **迷宫探索强化学习与拓扑漂移测量（Toy RL with Manifold Drift）**：在具有局部最优点陷阱的二维迷宫中，以策略网络中引入 Novelty-driven 探索奖励为 Level 2 惊奇度指标，以“到达终点”为 Level 3 形式化硬约束，检验在目标约束引力场下，AI 系统是否能以最少步骤生成偏离人类路径的意外解。
2. **合成化学小分子优化与软验证容噪（Synthesis Molecular Optimization under Soft Verification）**：在受限的分子空间内设计针对特定靶点的药物分子。使用简化的分子毒性与活性评估器为验证函数，以分子结构的表征异质性为贝叶斯惊奇度。针对真实化学实验等“软验证”（Soft Verification）场景，我们在验证函数中引入**贝叶斯软验证定界**（如设定 $\Pr(V(x|T)=1 \mid \text{Assay}) > \theta_v$），以控制误差漏斗并防止错误的意外反馈对先验库造成负向污染。

### 8.2 基于专有验证循环的组织优势
*   **意外收割技能（Serendipity Harvesting）训练课程**：
    传统的科研训练旨在培养高效率 of “目标消灭者”。我们提出一套**“意外收割技能”训练课程大纲**：核心包含“异常敏感度训练”（如何识别低噪声的有价值异常）、“人机非共识对抗提问法”（如何使用对抗性 prompts 以激活系统的低概率流形空间）以及“高维流形逆向解译”（XAI 基础与程序合成工具应用）。
*   **基于专有验证循环的组织优势**：
    企业与研究实验室可以通过建立专有验证循环与主动集成通道构建组织优势。丢弃这些 counterintuitive 异常的实验室将会落后，而系统性收集它们的实验室将能够持续更新其科学先验。我们在一套闭环开发测试台（closed-development testbed，作为一个内部原型平台）中评估该协议。在该架构中，沙箱环境充当了可编程的验证器 $V(x|T)$，验证后的 OOD 解会更新共享的智能体记忆系统。

### 8.3 局限性讨论
1. **验证成本的指数增长**：意外的价值建立在硬验证（$V(x|T) = 1$）的前提下。在物理世界验证中，高昂的时间与资金成本可能导致收割协议面临吞吐量瓶颈。
2. **人类偏见对翻译环节的污染**：在使用 XAI 或程序合成提取最小逻辑核时，解释工具本身的人类偏见可能强行将系统的非人类中心轨迹“合理化”为人类的旧图式，稀释了意外的原始价值。
3. **可复现性（Reproducibility）瓶颈**：由于温度采样的高随机性以及系统对 prompt 的极高敏感性，收割协议生成的意外轨迹往往难以稳定复现，这给严肃的学术发表和工业化部署带来了挑战。
4. **系统规模依赖性（Scale Dependence）**：存在明显的扩展律限制：更庞大的系统是否逆向产出更高 $U_{TC}$ 的意外？目前观察表明，超大参数量系统在对齐人类偏好后，表现出更强的认知收敛与图式驯化，如何维持其在扩展中的“去驯化”仍是未决问题。
5. **知识产权与学术伦理风险**：由 AI 系统产生底层非人类中心构想，并经人类专家进行形式化“回填”（Human Backfill）与证明的协作产物（如 2026 年平面单距离反例），其学术署名权与版权归属面临争议。如何在机器发现与人类阐释性提炼（Expositional Refinement）之间空间分配学术归属（Attribution），是当前学术伦理亟待解决的新挑战。

---

## 9. 结论与第二部（机制篇）接口规划

我们提出了三个观点。第一，AI 能够生成超出人类习惯的解答。第二，这些解答中有一部分是绝对正确的——它们通过了硬性测试。第三，我们能够构建流水线来捕捉它们。闭环流形约束解释了人类团队为何会漏掉这些解答；三层过滤器将有用的惊奇从噪声中区分开来；意外收割协议为实验室大规模实施这一过程提供了具体的途径。

为了将本篇提出的协议进行精确的形式化数学推导，我们的第二部研究《计算偶得性：信息熵与高维流形的形式化机制》将探讨在受限模型和验证器假设下，是否能够对以下技术课题进行形式化推导：
1. **收割协议伪代码的算法收敛性证明**：推导在特定采样温度调度函数 $\tau_t$ 下，生成集合 $\mathcal{C}$ 中高惊奇度个体的发生概率分布。
2. **Kolmogorov 复杂度与香农熵的联合约束边界**：形式化推导筛选阈值 $\theta_1$ 与 $\theta_2$ 在高维流形中的最佳拓扑分割面。
3. **基于 Lean 4 的形式化收割验证系统工程实现**。

---

## 致谢

作者对为本研究早期草稿提供反馈的同行表示感谢。

---

## 10. 参考文献

1. Alexeev, B., Barreto, K., Li, Y., Lichtman, J. D., Price, L., Shah, J. I., Tang, Q., & Tao, T. (2026). Primitive sets and von Mangoldt chains: Erdős Problem #1196 and beyond. *arXiv preprint arXiv:2605.00301*.
2. Alon, N., Gowers, W. T., Bloom, T. F., et al. (2026). Remarks on the disproof of the unit distance conjecture. *arXiv preprint arXiv:2605.20695* (accessed June 2026).
3. Cully, A., Clune, J., Tarapore, D., & Mouret, J. B. (2015). Robots that can adapt like animals. *Nature*, 521(7553), 503-507.
4. Dane, E. (2010). Reconsidering the connection between expertise and conceptual flexibility: A social cognitive approach. *Academy of Management Review*, 35(4), 579-603.
5. Friston, K. (2010). The free-energy principle: a unified brain theory?. *Nature Reviews Neuroscience*, 11(2), 127-138.
6. Jumper, J., et al. (2021). Highly accurate protein structure prediction with AlphaFold. *Nature*, 596(7873), 583-589.
7. Kuhn, T. S. (1962). *The Structure of Scientific Revolutions*. University of Chicago Press.
8. Lehman, J., & Stanley, K. O. (2011). Abandoning objectives: Evolution through the search for novelty alone. *Evolutionary Computation*, 19(2), 189-223.
9. Reddy, C. K., & Shojaee, P. (2024). Towards Scientific Discovery with Generative AI: Progress, Opportunities, and Challenges. *arXiv preprint arXiv:2412.11427*.
10. Romera-Paredes, B., et al. (2024). Mathematical discoveries from program search with large language models. *Nature*, 625(7995), 468-475. [https://doi.org/10.1038/s41586-023-06924-6]
11. Sawin, W. (2026). Planar unit distance lower bounds via Golod-Shafarevich towers. *arXiv preprint arXiv:2605.20579* (accessed June 2026).
12. Schultz, W., Dayan, P., & Montague, P. R. (1997). A neural substrate of prediction and reward. *Science*, 275(5306), 1593-1599.
13. Stanley, K. O., & Lehman, J. (2015). *Why Greatness Cannot Be Planned: The Quest for the Surprising*. Springer.
14. Stanley, K. O., Lehman, J., & Soros, L. (2019). *Open-endedness: The last grand challenge in AI*. O'Reilly Media, Inc.
