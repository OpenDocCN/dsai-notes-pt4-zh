<!--yml

类别：未分类

日期：2025-01-11 13:04:58

-->

# LLM-Coordination：评估和分析大语言模型中的多代理协调能力

> 来源：[https://arxiv.org/html/2310.03903/](https://arxiv.org/html/2310.03903/)

Saaket Agashe Yue Fan Anthony Reyna Xin Eric Wang

加利福尼亚大学圣克鲁兹分校

{saagashe yfan71 ancreyna xwang366}@ucsc.edu

###### 摘要

大语言模型（LLMs）所展示的涌现推理能力和心智理论（ToM）能力，使它们成为开发协调代理的有前景的候选者。在本研究中，我们引入了一个新的 LLM-Coordination 基准，旨在对 LLM 在纯协调游戏中的表现进行详细分析，在这些游戏中，参与的代理需要为了最大化收益进行合作。该基准通过两个不同的任务评估 LLM： (1) *Agentic Coordination*，LLM 在 4 个纯协调游戏中作为积极的合作参与者；(2) *协调问答（QA）*，LLM 被要求回答来自 4 个游戏的 198 道多项选择题，以评估三种关键推理能力：环境理解、ToM 推理和联合规划。此外，为了使 LLM 实现多代理协调，我们引入了一个协调的认知架构（CAC）框架，它可以轻松地将不同的 LLM 集成到纯协调游戏中，作为即插即用的模块。我们的研究结果表明，装备了 GPT-4-turbo 的 LLM 代理在需要基于环境常识的行动的游戏中，与最先进的强化学习方法表现相当。此外，零-shot 协调实验表明，与强化学习方法不同，LLM 代理对于新遇到的合作伙伴具有较强的适应性。然而，Coordination QA 的结果显示，LLM 在心智理论推理和联合规划能力方面仍有很大的提升空间。分析还揭示了 LLM 理解环境及其合作伙伴的信念和意图的能力，在其协调规划能力中所起的作用。我们的代码可以在 [https://github.com/eric-ai-lab/llm_coordination](https://github.com/eric-ai-lab/llm_coordination) 获取。

![参见图注](img/cf2326bea3538b6d0b3917f7562c6647.png)

图1：LLM协调基准包括两个任务：Agentic Coordination 用于研究 LLM 在*行动*方面的能力，以及 Coordination QA 用于研究 LLM 在*推理*方面的能力。

## 1 引言

在广泛的活动中，从日常任务如烹饪到重要操作如救援行动，合作没有混合意图是至关重要的。这些场景是纯协调游戏的例子，在这些游戏中，所有参与方都通过选择完全一致的策略来获得利益，避免了任何利益冲突。这些游戏要求智能体在考虑伙伴的信念和意图的同时，推理其环境并进行规划。最近，大语言模型（LLMs）在物理和虚拟环境中展现了新兴的规划能力（Raman等， [2022](https://arxiv.org/html/2310.03903v2#bib.bib27)；Wang等， [2023](https://arxiv.org/html/2310.03903v2#bib.bib33)；Wu等， [2023](https://arxiv.org/html/2310.03903v2#bib.bib35)），令人印象深刻的推理能力（Wei等， [2022](https://arxiv.org/html/2310.03903v2#bib.bib34)），以及心智理论的初步迹象（Kosinski， [2023](https://arxiv.org/html/2310.03903v2#bib.bib18)），使它们成为开发协调智能体的有希望候选者。先前的研究探索了使用LLM开发协作智能体，但LLM在协调游戏中的必要条件、优势和局限性仍不清楚。在本研究中，我们旨在通过对LLM在多智能体协调能力的全面评估和分析，填补这一空白。

因此，我们提出了一个新的LLM协调基准，包含两种纯协调游戏任务设置：1. 智能体协调（Agentic Coordination）和 2. 协调问答（CoordinationQA）。在智能体协调任务中，LLM通过组件的支持，使其能够在实际游戏环境中*行动*，为评估LLM作为协调智能体的能力提供了全面的评价。在协调问答任务中，LLM需要回答一组关于协调游戏中边缘情况的精心设计问题，这些游戏要求智能体与伙伴积极合作。该基准包括四个协作游戏，为全面分析提供了平台。

为了使大语言模型（LLMs）能够实现多智能体协调，我们提出了一个协调认知架构（CAC）框架，该框架以即插即用的方式促进LLM与游戏环境的交互。CAC将游戏元素转化为文本格式，并利用辅助的LLM以提高协调性，从而实现有效的多智能体协作。我们在使用CAC进行的智能体协调任务实验中发现，大语言模型能够理解游戏目标，为其下一步行动生成连贯的推理，并与伙伴在所有协调游戏中进行协调。它们在没有任何训练、微调或少量示例的情况下展示了这些能力。

对代理协调任务的比较分析表明，在那些只需最少心智理论推理并专注于根据环境采取常识行动的游戏中，大语言模型代理的表现优于多代理强化学习解决方案。然而，在更复杂的环境中，当代理需要积极考虑其伙伴的信念和意图时，它们表现不佳。我们还观察到，大语言模型代理能够与新伙伴协作，这与自对弈的多智能体强化学习方法（Carroll 等人，[2019a](https://arxiv.org/html/2310.03903v2#bib.bib4); Bard 等人，[2020](https://arxiv.org/html/2310.03903v2#bib.bib3)）不同，后者无法适应未见过的代理。

为了更深入地分析大语言模型的协调能力，我们创建了CoordinationQA套件。该套件旨在解构大语言模型在协调游戏中的单回合推理能力，重点关注三个关键领域：联合规划、心智理论（ToM）和环境理解。联合规划评估大语言模型在优化长期结果中的决策能力，心智理论问题探究模型对伙伴代理意图和需求的理解，环境理解评估对游戏规则和动态的掌握。我们在CoordinationQA中的发现表明，GPT-4-turbo与其他大语言模型在三类问题上的表现差距显著。在环境理解方面，大语言模型表现最为出色，表明它们对游戏规则和状态有很好的理解。然而，它们在心智理论推理方面面临重大挑战，难以推测他人的意图和需求。这一问题在联合规划中变得更为严重，绝大多数大语言模型在此方面的表现不佳，有些甚至比随机选择还要差。这些结果突显了大语言模型作为协调伙伴时的可靠性和有效性的局限性。此外，我们的分析显示，心智理论推理、环境理解和联合规划表现之间存在中到强的相关性，强调了它们在有效协调中的作用。

总结来说，我们的贡献有四个方面：

1.  1.

    我们推出了LLM协调基准，用于评估和分析大语言模型在纯协调游戏中的表现，涵盖了多回合代理协调和单回合协调问答任务。

1.  2.

    我们开发了即插即用的认知架构协调框架，以使大语言模型能够有效参与复杂的、部分可观察的协调游戏，例如《花火》，展示了在此类场景中大语言模型代理的首次零-shot应用。

1.  3.

    我们在自对弈和跨对弈设置中对大语言模型代理进行了全面评估，提供了与强化学习基准的详细比较，并展示了它们作为协调代理的潜力。

1.  4.

    我们研究了环境理解和心智理论推理作为大语言模型（LLMs）整体联合规划能力的关键组成部分，强调它们在协调任务中的重要性。

## 2 LLM协调基准

### 2.1 多回合代理协调

在多回合代理协调任务中，LLM作为代理参与端到端的纯协调游戏，其中所有参与代理的最佳策略是合作。测试中的LLM被插入协调框架中，框架包括记忆模块和基础模块，以便在完整的游戏中进行行动。这些LLM代理可以与任何策略或代理搭档，以完成游戏。

我们的LLM-Coordination基准包括4个纯协调游戏，Hanabi挑战（Bard等，[2020](https://arxiv.org/html/2310.03903v2#bib.bib3)）、Overcooked-AI（Carroll等，[2019a](https://arxiv.org/html/2310.03903v2#bib.bib4)）、Collab Capture和Collab Escape。

Hanabi挑战：在Hanabi（Bard等，[2020](https://arxiv.org/html/2310.03903v2#bib.bib3)）中，玩家的目标是按升序（1到5）组装五组卡片。游戏的独特之处在于，玩家只能看到队友的卡片，而不能看到自己的卡片。这要求玩家进行协作，利用揭示标记提供关于队友手中卡片的提示。这些提示可以是关于卡片的颜色或等级。例如，使用一个提示标记，玩家可以表示队友手中所有相同等级的卡片。Hanabi作为一个典型的纯协调游戏，要求玩家合作以达到最佳结果。Hanabi的成功依赖于理解队友的视角，在不完全信息的基础上做出决策，并进行隐性沟通，使其成为测试代理协调能力的绝佳平台。

Overcooked-AI：在Overcooked-AI环境中（Carroll等，[2019a](https://arxiv.org/html/2310.03903v2#bib.bib4)），两个代理——Alice（蓝色）和Bob（绿色）——合作烹饪并送达洋葱汤。这个环境包括多种布局，每个布局都有不同的洋葱分配器、盘子分配器、炉具、送餐区和工作台的排列和数量。为了准备一道菜，代理需要将三颗洋葱放入炉具中，启动一个持续20个时间步骤的烹饪过程。烹饪完成后，汤必须盛盘并送达才能完成任务。每个布局都提出了独特的挑战，强调了代理需要理解其周围环境、定位所需资源，并与队友同步行动以有效合作。

Collab Capture：Collab Capture涉及两个代理尝试在互联房间的迷宫中捕获一个对手。房间通过门相连，门可以通过位于其他房间中的访问按钮进行控制。代理的任务是在最短时间内使用有效策略捕获对手，包括将对手逼入角落，并操控门让队友通过或禁用对手。

合作逃脱：合作逃脱涉及两个智能体试图在一个由互联房间组成的迷宫中逃脱对手。他们需要修复位于不同房间的两个发电机（类似于游戏《Dead by Daylight》 (Dea, [2016](https://arxiv.org/html/2310.03903v2#bib.bib1))），以打开一个出口门户。对手试图抓住智能体，而胜利条件是任何一个智能体逃脱。这个游戏需要采用策略，如引诱对手远离伙伴、为伙伴的安全做出牺牲，以及操控对手的移动。

### 2.2 单回合协调问答

代理协调任务全面描绘了LLM作为智能体的能力。为了深入了解LLM的具体优缺点，我们开发了CoordinationQA套件。受到用于评估AI智能体的单元测试（Unit Testing）理念的启发，Knott等人（[2021](https://arxiv.org/html/2310.03903v2#bib.bib17)）手动从第二节[2.1](https://arxiv.org/html/2310.03903v2#S2.SS1 "2.1 多回合代理协调 ‣ 2 LLM-Coordination基准 ‣ LLM-Coordination：评估与分析大型语言模型中的多智能体协调能力")提到的所有四款纯协调游戏中抽样了边缘案例。这些边缘案例要求智能体积极理解当前状态，思考伙伴的意图，并制定最佳的协调计划。然后，我们为CoordinationQA套件中的每个情境创建了三种类型的问题。

+   •

    环境理解（EC）问题要求LLM进行间接推理，推测其环境中的某些方面。

+   •

    心智理论推理（ToM）问题挑战LLM预测其伙伴的意图，并探讨其伙伴的需求。

+   •

    联合规划（JP）问题为智能体提供状态/观察，并要求他们预测有效协调的最佳下一步行动。这个问题本质上是LLM在充当智能体时需要反复解决的相同问题。

所有问题均为手动开发并标注。我们筛选出了有歧义的题目和情境，只保留了那些有明确最优解的问题。我们一共生成了N=66个情境（25个来自《Overcooked》，28个来自《Hanabi》，以及13个来自两款合作游戏），并为每个情境创建了3个问题，共计198个独特问题。图[1](https://arxiv.org/html/2310.03903v2#S0.F1 "图1 ‣ LLM-Coordination：评估与分析大型语言模型中的多智能体协调能力")的右侧展示了三种问题类型的抽样过程，以《Overcooked》中的一个例子为例。所选情境显示蓝色智能体即将将第三个洋葱放入烤箱，而绿色智能体需要判断接下来应该做什么。

## 3 协调的认知架构

我们基于Sumers等人（[2023](https://arxiv.org/html/2310.03903v2#bib.bib31)）的工作，开发了一种名为协调认知架构（CAC）的LLM智能体架构，用于多智能体协调。通过使用CAC，我们可以轻松地插拔一个LLM智能体，并与任何伙伴配对（例如，另一个CAC智能体、人类玩家或其他AI智能体）。该架构由三个关键元素组成：内存、推理和基础。

![参见说明文字](img/efeeac07df025e5283271ed139436243.png)

图 2：协调的认知架构（CAC）。该框架分为三个关键组件，用于智能体分析——内存，存档游戏描述和当前游戏状态；基础，涉及由LLM选择的动作的执行；推理，包含一个心智理论（ToM）推理LLM，一个验证器LLM，以及正在分析的主要LLM。

内存模块包括（1）长期记忆，用于存储游戏描述，包括游戏规则、约定、目标和行动空间，（2）工作记忆，包含当前观察的文本描述，以及（3）情景记忆，是一个由智能体先前选择的动作组成的列表。在图[2](https://arxiv.org/html/2310.03903v2#S3.F2 "Figure 2 ‣ 3 Cognitive Architecture for Coordination ‣ LLM-Coordination: Evaluating and Analyzing Multi-agent Coordination Abilities in Large Language Models")所示的示例中，长期记忆包含了关于《花火》游戏的描述，工作记忆包含了包括当前堆叠、伙伴的手牌、两个智能体的信念和知识、以及关于可用代币和卡牌的信息，而情景记忆包含了先前选择的弃牌、出牌和提示。

推理模块是大型语言模型（LLM）被集成到框架中的地方。它以工作记忆中的文本描述作为输入，并根据上下文生成下一个最佳动作。对于像《花火》这样的协调游戏，需要更复杂的心智理论推理时，我们添加了一个辅助心智理论推理LLM，其唯一职责是解释伙伴智能体的动作和需求。这些额外的推理信息会被加入到工作记忆中，然后传递给主要LLM。此外，我们还利用了一个自我验证LLM，用于验证所选择动作的安全性。在图[2](https://arxiv.org/html/2310.03903v2#S3.F2 "Figure 2 ‣ 3 Cognitive Architecture for Coordination ‣ LLM-Coordination: Evaluating and Analyzing Multi-agent Coordination Abilities in Large Language Models")中的《花火》示例中，推理模块解释了“揭示1级卡牌”这一提示，并决定将指示的卡牌打到空堆上。生成的动作会传递给基础模块。

基础模块负责将推理和记忆模块的文本决策空间与实际游戏机制进行对接。其主要任务是从推理模块获取选定的动作，并将其转化为与游戏兼容的动作。基础模块的具体实现取决于所涉及的游戏；例如，在《Overcooked-AI》中，基础模块需要将像“从o0处拾起洋葱”这样的高级动作转换为一系列低级动作。另一方面，在像《Hanabi》这样的游戏中，它只需要将诸如“揭示Bob的红色卡片”之类的动作与其低级表示进行匹配。基础模块还负责根据游戏的上下文过滤掉不可行的动作。

## 4 实验

### 4.1 代理协调

#### 4.1.1 设置

我们在代理协调中执行两种类型的实验：自我对战和跨对战。在自我对战设置中，参与的代理是同类型的。在跨对战实验中，我们将代理与未见过的合作伙伴配对，他们需要根据这些新合作伙伴的行动调整自己的行为。

自我对战基准：对于《Overcooked》，我们使用近端策略优化（Proximal Policy Optimization，Schulman等，[2017](https://arxiv.org/html/2310.03903v2#bib.bib28)）和基于种群的训练（Population-Based Training，Jaderberg等，[2017](https://arxiv.org/html/2310.03903v2#bib.bib12)）作为对比基准。这些基准由Carroll等人（[2019a](https://arxiv.org/html/2310.03903v2#bib.bib4)）建立。《Hanabi》挑战已经通过MARL方法进行了广泛的研究与解决。我们使用贝叶斯动作解码器（Bayesian Action Decoder，Bard等，[2020](https://arxiv.org/html/2310.03903v2#bib.bib3)）、简化动作解码器（Simplified Action Decoder，Hu & Foerster，[2021](https://arxiv.org/html/2310.03903v2#bib.bib9)）和离信学习（Off-Belief Learning，Hu等，[2021a](https://arxiv.org/html/2310.03903v2#bib.bib10)）作为基准。

跨对战基准：对于《Overcooked》，我们使用一个基于人类数据训练的行为克隆模型（Behavior Cloning，Carroll等，[2019a](https://arxiv.org/html/2310.03903v2#bib.bib4)）和一个基于人类行为克隆的近端策略优化（PPO）代理（Carroll等，[2019a](https://arxiv.org/html/2310.03903v2#bib.bib4)）作为对比基准。我们还使用基于行为克隆的人类代理作为未见过的合作伙伴。对于《Hanabi》，我们使用简化动作解码器（Simplified Action Decoder，SAD）作为基准。我们将我们的代理与经过训练以生成有根据的策略并适应未见过的合作伙伴代理的离信学习（Off-Belief Learning，Hu等，[2021a](https://arxiv.org/html/2310.03903v2#bib.bib10)）配对。

指标：我们通过衡量代理在《Overcooked》中的总得分来进行评估，每次交付为两个代理提供20分。在《Hanabi》中的指标是玩家正确排列的卡片总数。

#### 4.1.2 结果与分析

|  | 《Overcooked》布局 |
| --- | --- |
| 方法 | CR | AA | Ring | FC | CC |
| PPO${}_{\text{SP}}$ (Shulman 等，[2017](https://arxiv.org/html/2310.03903v2#bib.bib28)) | $198.8\pm 4.06$ | $167.2\pm 3.63$ | $\mathbf{190.8\pm 4.25}$ | $151.9\pm 3.28$ | $122.3\pm 3.80$ |
| PBT (Jaderberg 等，[2017](https://arxiv.org/html/2310.03903v2#bib.bib12)) | $\mathbf{216.9\pm 1.31}$ | $190.1\pm 8.64$ | $173.8\pm 18.27$ | $169.5\pm 10.09$ | $140.1\pm 13.86$ |
| CAC[GPT-4-turbo] | $173.3\pm 6.67$ | $\mathbf{260.0\pm 11.55}$ | $140.0\pm 0.00$ | $\mathbf{180.0\pm 11.55}$ | $\mathbf{160.0\pm 0.00}$ |
| CAC[GPT-3.5-turbo] | $33.3\pm 10.88$ | $46.6\pm 10.88$ | $40.0\pm 0.00$ | $66.6\pm 14.40$ | $53.3\pm 5.44$ |
| CAC[Mixtral8x7B] | $46.6\pm 14.40$ | $200.0\pm 9.42$ | $113.3\pm 5.44$ | $46.6\pm 14.40$ | $100.0\pm 9.42$ |

表 1：多智能体强化学习（MARL）和 CAC 方法的表现比较。分数表示每个类别中的最佳表现。使用 GPT-4-turbo 的 CAC 在 5 个场景中的 3 个场景中展现出优越的协调能力，突出体现了协调任务中的高级推理能力。

| 类别 | 方法 | 分数 |
| --- | --- | --- |
| 强化学习 | BAD (Foerster 等，[2019](https://arxiv.org/html/2310.03903v2#bib.bib7)) | $23.92\pm 0.01$ |
|  | SAD (Hu & Foerster, [2021](https://arxiv.org/html/2310.03903v2#bib.bib9)) | $24.01\pm 0.01$ |
|  | OBL (Hu 等，[2021a](https://arxiv.org/html/2310.03903v2#bib.bib10)) | $24.10\pm 0.01$ |
| CAC | GPT-4-turbo | $\mathbf{13.33}\pm 0.88$ |
|  | GPT-3.5-turbo | $1.33\pm 0.72$ |
|  | Mixtral-8x7b | $0.33\pm 0.27$ |

表 2：Hanabi 挑战中的智能体表现比较。强化学习（RL）方法表现非常强大，获得了接近完美的分数。LLM 智能体（使用 GPT-4-turbo）较弱，但仍能完成游戏会话。

| 方法 | 分数 |
| --- | --- |
| LLM+自我验证+心智理论 | $\mathbf{13.33}\pm 0.88$ |
| LLM+自我验证 | $10.33\pm 0.88$ |
| LLM | $0.0\pm 0.0$ |

表 3：LLM 智能体在 Hanabi 挑战中的消融研究。自我验证显著提升了整体表现，通过过滤掉那些假设错误的行动。显式的心智理论（ToM）推理模型通过直接解释伙伴的线索和需求，进一步提升了性能。

##### LLM 智能体在依赖于理解环境的协调游戏中，超越或匹配了最先进的强化学习方法。

我们观察到，LLM智能体（使用GPT-4-turbo）在《超煮AI》的所有布局中，均优于或与强化学习方法的整体表现相当。表[1](https://arxiv.org/html/2310.03903v2#S4.T1 "Table 1 ‣ 4.1.2 Results and Analysis ‣ 4.1 Agentic Coordination ‣ 4 Experiments ‣ LLM-Coordination: Evaluating and Analyzing Multi-agent Coordination Abilities in Large Language Models")展示了不同智能体与同类型伙伴智能体配对时所获得的数值分数。这意味着，LLM智能体超越了那些通过自我对弈训练且没有进行任何特定游戏训练或微调的强化学习智能体。然而，需要注意的是，LLM智能体明显比强化学习模型慢且体积更大，目前还不适合用于实时应用（使用链式推理时，GPT-4-turbo的延迟为$8.36\pm 1.79$秒，无链式推理时为$1.02\pm 0.09$秒）。此外，我们测试的其他模型，如GPT-3.5-turbo和Mixtral8x7b，虽然速度更快，但仍未达到强化学习基准的水平。我们还在CollabCapture和CollabEscape游戏中看到了使用CAC智能体（使用GPT-4）取得的积极成果，成功率达到了100%。然而，其他LLM智能体无法破解CollabEscape（见附录[D](https://arxiv.org/html/2310.03903v2#A4 "Appendix D Results of Different LLMs on CollabCapture and CollabEscape ‣ LLM-Coordination: Evaluating and Analyzing Multi-agent Coordination Abilities in Large Language Models")）。

当需要高级的心智理论推理时，LLM智能体在有效规划方面会遇到困难。在《花火挑战》中，LLM智能体相比于强化学习方法似乎表现较差（见表[2](https://arxiv.org/html/2310.03903v2#S4.T2 "Table 2 ‣ 4.1.2 Results and Analysis ‣ 4.1 Agentic Coordination ‣ 4 Experiments ‣ LLM-Coordination: Evaluating and Analyzing Multi-agent Coordination Abilities in Large Language Models")）。在所有LLM中，GPT-4-turbo的表现相对较好，而其他LLM几乎无法完成《花火》游戏。我们将这种失败归因于两个因素。首先，《花火》中几乎没有容错空间。任何错误的操作或错误的提示都会导致失去一条生命令牌。其次，《花火》相比于《超煮AI》环境，需要更为复杂的心智理论推理。每一个动作都要求智能体积极考虑伙伴的信念、意图，以及他们如何对隐性沟通作出反应。

相比之下，《超煮AI》是完全可观察的，其动作空间包括像从onion_dispenser_0取洋葱和将洋葱放入cooker_0等动作。在大多数场景和布局中，LLM只需根据环境的状态考虑下一个最佳步骤。例如，我们进行了一项去除伙伴物品栏和位置的消融研究，结果表明对整体表现的影响最小（在100个时间步内，Cramped Room和Asymmetric Advantage布局各少了1次交付），这表明LLM在《超煮AI》这类游戏中的主要挑战是环境理解能力。

LLM代理在不完美信息博弈中，通过辅助推理模块获得了优势，尤其是在错误容忍度较低的情况下。如果没有辅助模块的支持，LLM代理似乎在每场游戏中都会失败（失去所有三条命）（见表[3](https://arxiv.org/html/2310.03903v2#S4.T3 "表 3 ‣ 4.1.2 结果与分析 ‣ 4.1 代理协调 ‣ 4 实验 ‣ LLM-协调：评估和分析大型语言模型中的多代理协调能力")）。为了纠正这一点，我们的CAC框架集成了辅助的LLM驱动模块，包括显式心智理论推理（ToM）模块和答案验证（AV）模块。答案验证模块能够有效阻止LLM对环境事实的错觉，避免造成致命错误，从而降低错误操作的概率。ToM推理LLM将解读伙伴线索和理解伙伴需求的任务分配给不同的LLM，允许主LLM集中精力合成可用信息以规划下一步行动。

|  | 过度烹饪布局 |
| --- | --- |
| 方法 | CR | AA | Ring | FC | CC |
| BC (Carroll et al., [2019a](https://arxiv.org/html/2310.03903v2#bib.bib4)) | $103.5$ &#124; $110.0$ | $136.5$ &#124; $137.5$ | $59.0$ &#124; $70.0$ | $20.5$ &#124; $31.0$ | $38.0$ &#124; $44.0$ |
| PPO[BC] (Schulman et al., [2017](https://arxiv.org/html/2310.03903v2#bib.bib28)) | $156.4$ &#124; $\mathbf{163.9}$ | $72.6$ &#124; $178.8$ | $126.4$ &#124; $129.8$ | $58.9$ &#124; $76.9$ | $69.5$ &#124; $57.6$ |
| CAC[GPT-4-turbo]¹¹1对于CAC，我们因成本限制只从一个位置进行单次试验。在表[1](https://arxiv.org/html/2310.03903v2#S4.T1 "表 1 ‣ 4.1.2 结果与分析 ‣ 4.1 代理协调 ‣ 4 实验 ‣ LLM-协调：评估和分析大型语言模型中的多代理协调能力")中我们观察到，CAC代理的表现并不会因为交付次数的不同而变化超过一次。 | $\mathbf{160.0}$ &#124; $160.0$ | $\mathbf{180.0}$ &#124; $\mathbf{200.0}$ | $\mathbf{160.0}$ &#124; $\mathbf{140.0}$ | $\mathbf{120.0}$ &#124; $\mathbf{80.0}$ | $\mathbf{140.0}$ &#124; $\mathbf{100.0}$ |

表 4：AI-人类代理游戏的零-shot协调结果。我们比较了行为克隆（BC）、PPO_BC和CAC（w/ GPT-4-turbo）代理。在大多数情况下，CAC代理的表现显著优于其他代理，显示了它们在面对未见过的伙伴代理时的鲁棒性。由于在《过度烹饪AI》中，两个代理可能会根据其起始位置被分配不同的角色，因此我们展示了从两边分别进行游戏的结果，使用 | 分隔。

| 方法 | 自我对战 | 与OBL-1跨对战 | 与OBL-4跨对战 |
| --- | --- | --- | --- |
| SAD (Hu & Foerster, [2021](https://arxiv.org/html/2310.03903v2#bib.bib9)) | $\mathbf{22.00}\pm 1.69$ | $11.66\pm 4.06$ | $5.33\pm 0.98$ |
| CAC[GPT-4-turbo] | $13.66\pm 0.27$ | $\mathbf{15.00}\pm 2.94$ | $\mathbf{12.0}\pm 0.94$ |

表5：RL智能体（SAD）与CAC智能体的跨对弈结果。所有智能体与不同种子（各智能体使用相同种子）进行三局比赛。SAD在自我对弈时表现非常好，但与新合作伙伴OBL-1和OBL-4配对时，性能显著下降。CAC智能体与这些新、未见过的合作伙伴协调良好。

LLM智能体对未见过的合作伙伴具有鲁棒性。我们使用Overcooked-AI和《花火》挑战作为测试平台，评估LLM智能体在与未见过的智能体配对时的表现。这项任务通常被称为零-shot协调。对于Overcooked-AI的实验，我们将LLM智能体及基线智能体与代理人类智能体配对。这些代理人类智能体是由Carroll 等人（[2019b](https://arxiv.org/html/2310.03903v2#bib.bib5)）使用人类数据训练的行为克隆智能体。如表[4](https://arxiv.org/html/2310.03903v2#S4.T4 "Table 4 ‣ LLM Agents outperform or match state-of-the-art RL methods in coordination games that depend more on understanding the environment. ‣ 4.1.2 Results and Analysis ‣ 4.1 Agentic Coordination ‣ 4 Experiments ‣ LLM-Coordination: Evaluating and Analyzing Multi-agent Coordination Abilities in Large Language Models")所示，我们发现LLM智能体的表现优于基于人类数据训练的行为克隆智能体和PPO智能体。

在《花火》实验中，我们将智能体与离信念学习（OBL）智能体配对（Hu 等人，[2021a](https://arxiv.org/html/2310.03903v2#bib.bib10)）。OBL是一种多智能体强化学习（MARL）策略，能够生成基于事实的线索和行动，是《花火》自我对弈和跨对弈的最先进方法。OBL智能体提供基于观察的线索，并与人类合作良好。因此，我们在实验中将它们作为未见过的合作伙伴。表[5](https://arxiv.org/html/2310.03903v2#S4.T5 "Table 5 ‣ LLM Agents outperform or match state-of-the-art RL methods in coordination games that depend more on understanding the environment. ‣ 4.1.2 Results and Analysis ‣ 4.1 Agentic Coordination ‣ 4 Experiments ‣ LLM-Coordination: Evaluating and Analyzing Multi-agent Coordination Abilities in Large Language Models")显示，CAC智能体与OBL-1智能体配对时的平均得分为15.00分，而它们在自我对弈中的得分为13.66分。这表明与新合作伙伴配对时协调能力没有下降。基准RL方法简化行动解码器（SAD）Hu & Foerster（[2021](https://arxiv.org/html/2310.03903v2#bib.bib9)）在与未见过的OBL智能体配对时表现严重失效，尽管它在自我对弈（22.00分）中表现优异，这是因为自我对弈训练的缘故。

采用自我博弈训练的MARL智能体在与未见过的合作伙伴进行常见收益任务时表现不佳，因为它们会收敛到只有自我博弈训练中的两个合作伙伴才能理解的任意策略（Carroll等， [2019a](https://arxiv.org/html/2310.03903v2#bib.bib4); Bard等， [2020](https://arxiv.org/html/2310.03903v2#bib.bib3)）。由于LLM智能体并未专门训练来进行这些游戏，它们基于提供的文本观察和从预训练中学到的常识性知识生成输出，因此对未见过的合作伙伴具有更强的鲁棒性。

### 4.2 协调QA

![参见说明](img/14db58095c298bdf33481042a0c9f1cf.png)

图3：LLMs在三个认知维度上的比较表现。图表显示了每个LLM在EC、ToM推理和JP中的准确率，并根据模型的参数数量（以十亿为单位）绘制了三次试验的结果。

##### 设置。

我们评估了6个大语言模型（LLMs）家族的表现，分别是Jiang等人（[2023](https://arxiv.org/html/2310.03903v2#bib.bib15); [2024](https://arxiv.org/html/2310.03903v2#bib.bib16)）；Touvron等人（[2023](https://arxiv.org/html/2310.03903v2#bib.bib32)）；Chiang等人（[2023](https://arxiv.org/html/2310.03903v2#bib.bib6)）；OpenAI（[2023](https://arxiv.org/html/2310.03903v2#bib.bib24)）在三个维度上的表现：环境理解（EC）、心智理论推理（ToM）和联合规划（JP）。对于每个类别，LLMs回答多个选择题（MCQs），并通过模糊字符串匹配将其回答与真实答案进行比较。为了考虑LLM回答中的变异性，我们为每个模型进行了三次试验。同时我们还报告了一个随机基线。

LLMs在环境理解、ToM推理与联合规划的比较结果。 在图[3](https://arxiv.org/html/2310.03903v2#S4.F3 "图3 ‣ 4.2 协调QA ‣ 4 实验 ‣ LLM-协调：评估和分析大语言模型在多智能体协调能力中的表现")中，我们看到大多数LLMs在环境理解问题上表现最佳。表现最好的LLM GPT-4-turbo在环境理解问题上正确率超过80%。在更具挑战性的心智理论推理问题上，LLMs的整体表现下降，但GPT-4-turbo仍然具有能力，达到了54%的准确率。LLMs在联合规划问题上的总体准确性仍然相当薄弱，即使是最好的LLM得分也不到40%，这表明LLM在执行协调推理方面仍有很大改进空间。另一个值得关注的问题是，开源LLMs在联合规划方面表现极差，一些模型的表现甚至不如随机基线。

| 变量 | $r$ | $\rho$ |
| --- | --- | --- |
| ToM | 0.813 | 0.389 |
| EC | 0.895 | 0.506 |

表6：皮尔逊相关系数（$r$）和斯皮尔曼等级相关系数（$\rho$）揭示了ToM和EC与JP之间的中等到强的正相关。

环境理解能力和心智理论推理能力对联合规划的影响。我们将联合规划定义为代理根据可用信息选择适当后续行动的能力，认为环境理解能力和心智理论推理的熟练程度对于高效的联合规划至关重要，表现优秀的LLM在这两个方面也能在联合规划中表现出色。对心智理论（ToM）和环境理解（EC）与联合规划（JP）之间关系的分析，基于图表[6](https://arxiv.org/html/2310.03903v2#S4.T6 "Table 6 ‣ Setup. ‣ 4.2 Coordination QA ‣ 4 Experiments ‣ LLM-Coordination: Evaluating and Analyzing Multi-agent Coordination Abilities in Large Language Models")的数据，表明心智理论与联合规划呈中等正相关，而环境理解与联合规划呈强正相关。

## 5 相关工作

##### 多智能体协调

在博弈论中，纯协调博弈是所有代理人共享回报的情形。在这种情况下，合作是最优策略。多年来，已有多种基准和游戏被用来评估多代理人协调能力，包括多粒子环境 Lowe 等人（[2017](https://arxiv.org/html/2310.03903v2#bib.bib23)）、过度烹饪-AI Carroll 等人（[2019a](https://arxiv.org/html/2310.03903v2#bib.bib4)）以及 Hanabi 挑战 Bard 等人（[2020](https://arxiv.org/html/2310.03903v2#bib.bib3)）。Carroll 等人（[2019a](https://arxiv.org/html/2310.03903v2#bib.bib4)）的基础性工作强调了在人机合作中融入人类数据的重要性。随后，关于过度烹饪-AI挑战的研究逐渐转向使得通过自我对弈训练的代理人能够在该环境中与人类无缝协调。这些研究采用了多种技术，包括使用过去代理人检查点进行自我对弈（Strouse 等人，[2021](https://arxiv.org/html/2310.03903v2#bib.bib30)）、集中的人口熵目标（Zhao 等人，[2023](https://arxiv.org/html/2310.03903v2#bib.bib39)）、使用图论的开放式目标（Li 等人，[2023a](https://arxiv.org/html/2310.03903v2#bib.bib19)）、带有上下文感知机制的策略集成（Lou 等人，[2023](https://arxiv.org/html/2310.03903v2#bib.bib22)）以及将人类偏差作为线性隐藏奖励的方式（Yu 等人，[2023](https://arxiv.org/html/2310.03903v2#bib.bib36)），以增强AI代理人在不同场景下的训练和多样性。在 Hanabi 挑战中，已经进行了大量努力来学习基于实际情况的策略 Hu 等人（[2021b](https://arxiv.org/html/2310.03903v2#bib.bib11); [a](https://arxiv.org/html/2310.03903v2#bib.bib10)）而非任意约定。最近，通常在家庭环境中设置的具身环境也被用来研究多代理人协调（Puig 等人，[2021](https://arxiv.org/html/2310.03903v2#bib.bib26); Jain 等人，[2020](https://arxiv.org/html/2310.03903v2#bib.bib14); [2019](https://arxiv.org/html/2310.03903v2#bib.bib13); Gan 等人，[2021](https://arxiv.org/html/2310.03903v2#bib.bib8)）。绝大多数关于协调问题的方法集中在利用和增强强化学习方法来解决多代理人协调的问题。在本研究中，我们认为大型语言模型是解决这些协调问题的另一种方法，因为它们表现出了突出的推理能力，展现了类似心智理论的能力，并且不会收敛到导致任意联合交互的策略。

##### 大型语言模型的规划与推理

大型语言模型（LLMs）在自然语言推理方面展示了卓越的能力（OpenAI, [2023](https://arxiv.org/html/2310.03903v2#bib.bib24); Ouyang et al., [2022](https://arxiv.org/html/2310.03903v2#bib.bib25); Chiang et al., [2023](https://arxiv.org/html/2310.03903v2#bib.bib6)），在各类语言推理任务中取得了最先进的表现。随后研究表明，LLM可以通过增加内存、工具、感知和基础设施等组件来增强，进而成为能够与外部环境（如网络、模拟器、游戏等）互动的代理。这些LLM代理已经证明能够解决长时程任务、玩复杂的游戏（Wu et al., [2023](https://arxiv.org/html/2310.03903v2#bib.bib35); Wang et al., [2023](https://arxiv.org/html/2310.03903v2#bib.bib33)），并与模拟的具身环境进行互动（Liang et al., [2022](https://arxiv.org/html/2310.03903v2#bib.bib21); Song et al., [2022](https://arxiv.org/html/2310.03903v2#bib.bib29)）。Zhang et al. ([2023b](https://arxiv.org/html/2310.03903v2#bib.bib38)) 开发了一个模块化代理框架，能够与合作伙伴代理在具身空间重排问题中协作，通过协调提高了效率。Zhang et al. ([2023a](https://arxiv.org/html/2310.03903v2#bib.bib37)) 开发了一种专门的架构，使LLM能够在《Overcooked-AI》环境中进行游戏。Li et al. ([2023b](https://arxiv.org/html/2310.03903v2#bib.bib20)) 评估并展示了LLM在游戏化模拟中的协作能力。与现有工作不同，我们的研究聚焦于评估语言代理在已建立的纯协调游戏中的表现，在这些游戏中，协调不仅是提高效率的可选项，而是必不可少的。

## 6 结论

在本研究中，我们评估和分析了当前的大型语言模型在推理和参与纯协调游戏中的能力。我们引入了LLM-Coordination基准测试及其两个任务：1. **代理协调** 和 2. **协调问答**。这些设置使我们能够进行全面的比较研究，既研究LLM作为代理的表现，也深入探讨LLM作为协调推理者的细节方面。我们将LLM代理与现有的多智能体强化学习代理进行对比，讨论了LLM表现优秀和失败的条件。最后，我们讨论了**心智理论推理**和**环境理解**作为协调的前提条件，并评估了现有LLM在这两个方面的表现。

## 参考文献

+   Dea (2016) Dead by Daylight。 [https://deadbydaylight.com/en](https://deadbydaylight.com/en)，2016年6月。视频游戏。

+   han (2024) Hanabi: 一款合作的烟花游戏 - GitHub 仓库。 [https://github.com/hanabi/hanabi.github.io](https://github.com/hanabi/hanabi.github.io), 2024年。访问时间：2024年3月29日。

+   巴德等人（2020）诺兰·巴德、雅各布·N·福尔斯特、萨拉斯·昌达尔、尼尔·伯奇、马克·兰特托、H·弗朗西斯·宋、埃米利奥·帕里索托、文森特·杜穆兰、苏博迪普·莫伊特拉、爱德华·休斯、伊恩·丹宁、希布·穆拉德、雨果·拉罗谢尔、马克·G·贝尔梅尔和迈克尔·鲍林。《花火挑战：人工智能研究的新前沿》。*人工智能*，280:103216，2020年。ISSN 0004-3702。DOI：[https://doi.org/10.1016/j.artint.2019.103216](https://doi.org/10.1016/j.artint.2019.103216)。网址 [https://www.sciencedirect.com/science/article/pii/S0004370219300116](https://www.sciencedirect.com/science/article/pii/S0004370219300116)。

+   卡罗尔等人（2019a）米卡·卡罗尔、罗欣·沙、马克·K·霍、托马斯·L·格里菲斯、桑吉特·A·塞西亚、皮特·阿贝尔和安卡·德拉根。《*学习关于人类的知识对人类-人工智能协调的效用*》。Curran Associates Inc.，纽约红钩，美国，2019a。

+   卡罗尔等人（2019b）米卡·卡罗尔、罗欣·沙、马克·K·霍、托马斯·L·格里菲斯、桑吉特·A·塞西亚、皮特·阿贝尔和安卡·德拉根。《超煮AI》。[https://github.com/HumanCompatibleAI/overcooked_ai/tree/master](https://github.com/HumanCompatibleAI/overcooked_ai/tree/master)，2019b。

+   蔣等人（2023）魏琳·蔣、朱焕·李、子琳、英盛、张昊·吴、浩张、连民·郑、思源·庄、永浩·庄、约瑟夫·E·冈萨雷斯、艾昂·斯托伊卡和埃里克·P·辛。《Vicuna：一个开源聊天机器人，以90%*的chatgpt质量给GPT-4留下深刻印象》，2023年3月。网址 [https://lmsys.org/blog/2023-03-30-vicuna/](https://lmsys.org/blog/2023-03-30-vicuna/)。

+   福尔斯特等人（2019）雅各布·N·福尔斯特、弗朗西斯·宋、爱德华·休斯、尼尔·伯奇、伊恩·丹宁、希蒙·怀特森、马修·博特维尼克和迈克尔·鲍林。《深度多智能体强化学习的贝叶斯行动解码器》，2019年。

+   甘等人（2021）庄甘、杰里米·施瓦茨、赛斯·阿尔特、达米安·姆罗卡、马丁·施林普夫、詹姆斯·特雷尔、朱利安·德·弗雷塔斯、乔纳斯·库比利乌斯、阿比谢克·班德瓦尔达、尼克·哈伯、梅谷佐野、金久野、埃利亚斯·王、迈克尔·林格尔巴赫、艾登·柯蒂斯、凯文·费吉利斯、丹尼尔·M·比尔、丹·古特弗伦德、戴维·考克斯、安东尼奥·托拉尔巴、詹姆斯·J·迪卡洛、约书亚·B·特嫩鲍姆、乔什·H·麦克德莫特和丹尼尔·L·K·雅敏斯。《Threedworld：一个互动的多模态物理仿真平台》，2021年。

+   胡与福尔斯特（2021）亨元·胡和雅各布·N·福尔斯特。《深度多智能体强化学习的简化行动解码器》，2021年。

+   胡等人（2021a）亨元·胡、亚当·勒雷、布兰登·崔、戴维·吴、路易斯·皮内达、诺亚姆·布朗和雅各布·福尔斯特。《离信念学习》，2021a。

+   胡等人（2021b）亨元·胡、亚当·勒雷、亚历克斯·佩萨科维奇和雅各布·福尔斯特。《零-shot协调的“其他游戏”》，2021b。

+   贾德伯格等人（2017）马克斯·贾德伯格、瓦伦丁·达利巴尔德、西蒙·奥辛德罗、沃耶奇·M·查尔涅基、杰夫·多纳休、阿里·拉扎维、奥里奥尔·维尼亚尔斯、蒂姆·格林、伊恩·丹宁、凯伦·西蒙扬、克里桑莎·费尔南多和科雷·卡武克丘格鲁。《基于种群的神经网络训练》，2017年。

+   Jain et al. (2019) Unnat Jain, Luca Weihs, Eric Kolve, Mohammad Rastegari, Svetlana Lazebnik, Ali Farhadi, Alexander G. Schwing, 和 Aniruddha Kembhavi. 《双体问题：协作视觉任务完成》。发表于 *CVPR*，2019年。前两位作者贡献相等。

+   Jain et al. (2020) Unnat Jain, Luca Weihs, Eric Kolve, Ali Farhadi, Svetlana Lazebnik, Aniruddha Kembhavi, 和 Alexander G. Schwing. 《和谐同步：超越边际策略进行多智能体体任务》。发表于 *ECCV*，2020年。前两位作者贡献相等。

+   Jiang et al. (2023) Albert Q. Jiang, Alexandre Sablayrolles, Arthur Mensch, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Florian Bressand, Gianna Lengyel, Guillaume Lample, Lucile Saulnier, Lélio Renard Lavaud, Marie-Anne Lachaux, Pierre Stock, Teven Le Scao, Thibaut Lavril, Thomas Wang, Timothée Lacroix, 和 William El Sayed. 《Mistral 7b》，2023年。

+   Jiang et al. (2024) Albert Q. Jiang, Alexandre Sablayrolles, Antoine Roux, Arthur Mensch, Blanche Savary, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Emma Bou Hanna, Florian Bressand, Gianna Lengyel, Guillaume Bour, Guillaume Lample, Lélio Renard Lavaud, Lucile Saulnier, Marie-Anne Lachaux, Pierre Stock, Sandeep Subramanian, Sophia Yang, Szymon Antoniak, Teven Le Scao, Théophile Gervet, Thibaut Lavril, Thomas Wang, Timothée Lacroix, and William El Sayed. 《专家的Mixtral》，2024年。

+   Knott et al. (2021) Paul Knott, Micah Carroll, Sam Devlin, Kamil Ciosek, Katja Hofmann, A. D. Dragan, 和 Rohin Shah. 《评估协作智能体的鲁棒性》，2021年。

+   Kosinski (2023) Michal Kosinski. 《心智理论可能在大型语言模型中自发出现》，2023年。

+   Li et al. (2023a) Yang Li, Shao Zhang, Jichen Sun, Yali Du, Ying Wen, Xinbing Wang, 和 Wei Pan. 《零-shot协调的合作性开放式学习框架》。在 Andreas Krause, Emma Brunskill, Kyunghyun Cho, Barbara Engelhardt, Sivan Sabato, 和 Jonathan Scarlett (编辑)，*国际机器学习大会 ICML 2023，2023年7月23日至29日，夏威夷檀香山，美国*，第202卷 *机器学习研究会议录*，第20470–20484页。PMLR，2023a年。网址：[https://proceedings.mlr.press/v202/li23au.html](https://proceedings.mlr.press/v202/li23au.html)。

+   Li et al. (2023b) Yuan Li, Yixuan Zhang, 和 Lichao Sun. 《Metaagents：通过协作生成智能体模拟人类行为交互，实现基于LLM的任务导向协调》，2023b年。

+   Liang et al. (2022) Jacky Liang, Wenlong Huang, Fei Xia, Peng Xu, Karol Hausman, Brian Ichter, Pete Florence, 和 Andy Zeng. 《代码作为策略：用于体现控制的语言模型程序》。发表于 *arXiv预印本 arXiv:2209.07753*，2022年。

+   Lou 等人 (2023) Xingzhou Lou, Jiaxian Guo, Junge Zhang, Jun Wang, Kaiqi Huang 和 Yali Du。Pecan：利用策略集成进行上下文感知的零-shot人类-ai协作。发表于 *2023年国际自主代理与多代理系统会议论文集*，AAMAS '23，页码 679–688，南卡罗来纳州瑞士兰，2023年。国际自主代理与多代理系统基金会。ISBN 9781450394321。

+   Lowe 等人 (2017) Ryan Lowe, Yi Wu, Aviv Tamar, Jean Harb, Pieter Abbeel 和 Igor Mordatch。适用于混合合作竞争环境的多代理演员-评论员算法。发表于 *第31届神经信息处理系统国际会议论文集*，NIPS'17，页码 6382–6393，纽约州红钩，2017年。Curran Associates Inc. ISBN 9781510860964。

+   OpenAI (2023) OpenAI。GPT-4 技术报告，2023年。

+   Ouyang 等人 (2022) Long Ouyang, Jeff Wu, Xu Jiang, Diogo Almeida, Carroll L. Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray, John Schulman, Jacob Hilton, Fraser Kelton, Luke Miller, Maddie Simens, Amanda Askell, Peter Welinder, Paul Christiano, Jan Leike 和 Ryan Lowe。训练语言模型通过人类反馈遵循指令，2022年。

+   Puig 等人 (2021) Xavier Puig, Tianmin Shu, Shuang Li, Zilin Wang, Yuan-Hong Liao, Joshua B. Tenenbaum, Sanja Fidler 和 Antonio Torralba。Watch-and-help：社会感知与人类-{ai}合作的挑战。发表于 *国际学习表征会议*，2021年。网址 [https://openreview.net/forum?id=w_7JMpGZRh0](https://openreview.net/forum?id=w_7JMpGZRh0)。

+   Raman 等人 (2022) Shreyas Sundara Raman, Vanya Cohen, Eric Rosen, Ifrah Idrees, David Paulius 和 Stefanie Tellex。通过修正重新提示与大型语言模型进行规划，2022年。

+   Schulman 等人 (2017) John Schulman, Filip Wolski, Prafulla Dhariwal, Alec Radford 和 Oleg Klimov。近端策略优化算法，2017年。

+   Song 等人 (2022) Chan Hee Song, Jiaman Wu, Clayton Washington, Brian M Sadler, Wei-Lun Chao 和 Yu Su。LLM-planner：面向具身代理的基于大型语言模型的少样本落地规划。*arXiv 预印本 arXiv:2212.04088*，2022年。

+   Strouse 等人 (2021) DJ Strouse, Kevin McKee, Matt Botvinick, Edward Hughes 和 Richard Everett。与人类合作而无需人类数据。见 M. Ranzato, A. Beygelzimer, Y. Dauphin, P.S. Liang 和 J. Wortman Vaughan (编辑)，*神经信息处理系统进展*，第34卷，页码 14502–14515。Curran Associates, Inc.，2021年。网址 [https://proceedings.neurips.cc/paper_files/paper/2021/file/797134c3e42371bb4979a462eb2f042a-Paper.pdf](https://proceedings.neurips.cc/paper_files/paper/2021/file/797134c3e42371bb4979a462eb2f042a-Paper.pdf)。

+   Sumers 等人 (2023) Theodore R. Sumers, Shunyu Yao, Karthik Narasimhan 和 Thomas L. Griffiths。语言代理的认知架构，2023年。

+   Touvron等人（2023）Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, Dan Bikel, Lukas Blecher, Cristian Canton Ferrer, Moya Chen, Guillem Cucurull, David Esiobu, Jude Fernandes, Jeremy Fu, Wenyin Fu, Brian Fuller, Cynthia Gao, Vedanuj Goswami, Naman Goyal, Anthony Hartshorn, Saghar Hosseini, Rui Hou, Hakan Inan, Marcin Kardas, Viktor Kerkez, Madian Khabsa, Isabel Kloumann, Artem Korenev, Punit Singh Koura, Marie-Anne Lachaux, Thibaut Lavril, Jenya Lee, Diana Liskovich, Yinghai Lu, Yuning Mao, Xavier Martinet, Todor Mihaylov, Pushkar Mishra, Igor Molybog, Yixin Nie, Andrew Poulton, Jeremy Reizenstein, Rashi Rungta, Kalyan Saladi, Alan Schelten, Ruan Silva, Eric Michael Smith, Ranjan Subramanian, Xiaoqing Ellen Tan, Binh Tang, Ross Taylor, Adina Williams, Jian Xiang Kuan, Puxin Xu, Zheng Yan, Iliyan Zarov, Yuchen Zhang, Angela Fan, Melanie Kambadur, Sharan Narang, Aurelien Rodriguez, Robert Stojnic, Sergey Edunov, 和 Thomas Scialom. Llama 2：开放基础和微调的聊天模型，2023。

+   Wang等人（2023）Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao, Yuke Zhu, Linxi Fan, 和 Anima Anandkumar. Voyager：一个基于大型语言模型的开放式具身代理，2023。

+   Wei等人（2022）Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, 等人. Chain-of-thought提示引发大型语言模型的推理。*神经信息处理系统进展*，35：24824–24837，2022。

+   Wu等人（2023）Yue Wu, Shrimai Prabhumoye, So Yeon Min, Yonatan Bisk, Ruslan Salakhutdinov, Amos Azaria, Tom Mitchell, 和 Yuanzhi Li. Spring：GPT-4通过学习论文和推理超越RL算法，2023。

+   Yu等人（2023）Chao Yu, Jiaxuan Gao, Weilin Liu, Botian Xu, Hao Tang, Jiaqi Yang, Yu Wang, 和 Yi Wu. 学习零-shot合作与人类，假设人类存在偏差。在*第十一届国际学习表示会议（ICLR 2023），卢旺达基加利，2023年5月1-5日*。OpenReview.net，2023。URL [https://openreview.net/pdf?id=TrwE8l9aJzs](https://openreview.net/pdf?id=TrwE8l9aJzs)。

+   Zhang等人（2023a）Ceyao Zhang, Kaijie Yang, Siyi Hu, Zihao Wang, Guanghe Li, Yihang Sun, Cheng Zhang, Zhaowei Zhang, Anji Liu, Song-Chun Zhu, Xiaojun Chang, Junge Zhang, Feng Yin, Yitao Liang, 和 Yaodong Yang. Proagent：利用大型语言模型构建积极的协作AI，2023a。

+   Zhang等人（2023b）Hongxin Zhang, Weihua Du, Jiaming Shan, Qinhong Zhou, Yilun Du, Joshua B. Tenenbaum, Tianmin Shu, 和 Chuang Gan. 使用大型语言模型模块化构建协作具身代理，2023b。

+   Zhao等人（2023年）Rui Zhao, Jinming Song, Yufeng Yuan, Haifeng Hu, Yang Gao, Yi Wu, Zhongqian Sun, 和 Wei Yang. 基于最大熵的人类-人工智能零-shot协调的种群训练. *人工智能学会年会论文集*, 37(5):6145–6153, 2023年6月. doi: 10.1609/aaai.v37i5.25758. 网址 [https://ojs.aaai.org/index.php/AAAI/article/view/25758](https://ojs.aaai.org/index.php/AAAI/article/view/25758).

## 附录A 《Overcooked》实现细节

### A.1 游戏和布局描述

我们使用一个通用的游戏描述${G}$来解释《Overcooked》的规则和目标。由于每个布局的地点数量不同，例如洋葱分发器和烹饪设备，我们为每个环境${L_{i}}$提供简洁的描述，其中包括特定设施的实例数量。对于包含隔断的环境，我们说明每个代理所处的隔断以及该代理可以访问的设施。此外，我们还提到了环境的形状。

{mdframed}[⬇](data:text/plain;base64,SSBhbSB7c2VsZi5wbGF5ZXJfbmFtZXNbc2VsZi5wbGF5ZXJfaWRdfS4gSSBhbSBwbGF5aW5nIHRoZSBnYW1lIE92ZXJjb29rZWQgd2l0aCBteSBwYXJ0bmVyIHtzZWxmLnBsYXllcl9uYW1lc1tzZWxmLm90aGVyX3BsYXllcl9pZF19LiB7RW52RGVzY3JpcHRpb25zW3NlbGYubGF5b3V0X25hbWVdfQpPdmVyY29va2VkIGhhcyB0aGUgZm9sbG93aW5nIHJ1bGVzOiB7c2VsZi5ydWxlc30uIFdlIGhhdmUgYWdyZWVkIHRvIGZvbGxvdyB0aGUgZm9sbG93aW5nIGNvbnZlbnRpb25zOiB7c2VsZi5jb252ZW50aW9uc30uIEknbGwgcHJvdmlkZSBteSBhY3Rpb24gaGlzdG9yeSwgY3VycmVudCBzdGF0ZSwgdGVhbW1hdGUncyBzdGF0dXMsIGFuZCBteSBwb3NzaWJsZSBhY3Rpb25zLiBIZWxwIG1lIHNlbGVjdCB0aGUgYmVzdCBhY3Rpb24gZnJvbSB0aGUgbGlzdC4gRm9ybWF0IHlvdXIgcmVzcG9uc2UgYXM6IEFjdGlvbjogPGFjdGlvbj4uIE9ubHkgc2VsZWN0IG9uZSBhY3Rpb24uIERvIG5vdCBzYXkgYW55dGhpbmcgZWxzZS4gR290IGl0Pw==)我 是 {self.player_names[self.player_id]}。 我 正在 与 我的 合作伙伴 {self.player_names[self.other_player_id]} 一起 玩 游戏《Overcooked》。 {EnvDescriptions[self.layout_name]}《Overcooked》 有 以下 规则： {self.rules}。 我们 已 经 达成 一致，遵循 以下 约定： {self.conventions}。 我 将 提供 我的 行为历史、当前 状态、队友 状态 和 我的 可能 行动。 请 帮助 我 从 列表 中 选择 最佳 行动。 请 按 以下格式 回答： 行动: <action>。 只 选择 一 个 行动。 不 要 说 其他 内容。 明白 吗？

### A.2 状态描述

状态在工作记忆中以自然语言$D(S)$表示，可以被大型语言模型（LLM）处理。状态$S$包括完全代表布局及玩家所需的变量。$D(S)$中提供的信息等同于强化学习（RL）代理在状态表示的形式中可以访问到的信息。$D(S)$中包括以下信息：

##### 每个玩家持有的物品

状态描述 $D(S)$ 首先详细列出 Alice 和 Bob 的库存 $I_{\alpha_{1}}$ 和 $I_{\alpha_{2}}$。每个库存 $I_{\alpha_{i}}$（其中 $i\in\{1,2\}$）可以包含以下物品之一：{“洋葱”，“盘子”，“煮熟的汤”}。这些库存信息被转换成自然语言，并以以下格式纳入 $D(S)$：“我持有 $I_{\alpha_{1}}$。Bob 持有 $I_{\alpha_{2}}$。”这类信息对于推测合作代理可能的后续行动至关重要。

##### 由LLM控制的代理的位置：

鉴于大型语言模型（LLMs）在解读基于网格的空间信息时的局限性，我们选择向LLM提供处理后的位置信息。对于每个代理 $P_{i}$（其中 $i\in\{1,2\}$），以及每个感兴趣的地点，记作 loc，我们计算距离 $d_{(P_{i},\text{loc})}$，即从 $P_{i}$ 出发，使用最短路径到达 loc 所需的步数。状态描述 $D(S)$ 然后以以下格式包含该处理后的位置信息：“loc 离 $P_{i}$ 有 $d_{(P_{i},\text{loc})}$ 单位远。”这里，loc 可以表示各种兴趣点，如洋葱分发器、盘子分发器、炉子、交付区域、厨房台面或共享台面。如果一个位置无法访问或被另一个代理阻挡，$D(S)$ 会明确指出。例如，如果某个位置被 Bob 阻挡，它会被表示为“loc 被 Bob 阻挡。”为了区分与每个代理相关的位置信息，$D(S)$ 会在相应部分前添加“您的位置信息：”来表示由LLM控制的代理的位置信息，以及“Bob 的位置信息：”来表示合作代理的位置信息。

##### 炉子信息

状态描述 $D(S)$ 还包含关于炉子的信息，这对游戏策略至关重要。具体来说，对于每个炉子 $i$，$D(S)$ 包括当前锅中洋葱的数量 $n_{\text{i}}$。此外，$D(S)$ 提供炉子的操作状态，表示为 $\text{CookerState}_{i}$，该状态可以是“关闭”或“开启”。最后，锅中汤的当前状态由 $\text{SoupState}_{i}$ 表示，可能的值有：“烹饪中”，“已煮熟”或“未开始”。因此，炉子 $c_{i}$ 的信息格式如下：“$c_{i}$ 中有 $n_{\text{i}}$ 个洋葱。$c_{i}$ 状态是 $\text{CookerState}_{i}$。$c_{i}$ 中的汤状态是 $\text{SoupState}_{i}$。”

##### 厨房台面信息

状态描述 $D(S)$ 包括厨房台面的信息，厨房台面主要用于临时存放物品。具体来说，$D(S)$ 会标识最近的空厨房台面 $k_{\text{empty}}$，以及当前承载物品的所有台面集合 $K_{\text{filled}}$。

##### 共享台面信息

共享工作台作为专用的厨房工作台，用于代理之间的物品传递。对于每个共享工作台 $i$，$D(S)$ 包括 $s_{i}$ 的状态，如“$s_{0}$ 是空的”或“$s_{1}$ 包含洋葱”，以提供完整的环境概述。与厨房工作台不同的是，厨房工作台只提到最接近的空工作台，而共享工作台会提到所有空的工作台。

{mdframed}[⬇](data:text/plain;base64,PEludmVudG9yeT46IEkgYW0gaG9sZGluZyBvbmlvbi4gQm9iIGlzIGhvbGRpbmcgbm90aGluZy4KCjxNeSBMb2NhdGlvbiBJbmZvcm1hdGlvbj46IG8wIGlzIDAgdW5pdHMgYXdheS4gbzEgaXMgMSB1bml0cyBhd2F5LiBwMCBpcyAzIHVuaXRzIGF3YXkuIGMwIGlzIDYgdW5pdHMgYXdheSBibG9ja2VkIGJ5IEJvYi4gYzEgaXMgNyB1bml0cyBhd2F5LiBkMCBpcyA0IHVuaXRzIGF3YXkuIHMwIGlzIDEgdW5pdHMgYXdheS4gczEgaXMgMCB1bml0cyBhd2F5LiBzMiBpcyAxIHVuaXRzIGF3YXkuIHMzIGluIDIgdW5pdHMgYXdheS4gQ2xvc2VzdCBlbXB0eSBraXRjaGVuIGNvdW50ZXIgazEyIGlzIDEgdW5pdHMgYXdheS4KCjxCb2IncyBMb2NhdGlvbiBJbmZvcm1hdGlvbj46IG8wIGlzIGJsb2NrZWQgYnkgQWxpY2UuIG8xIGlzIDcgdW5pdHMgYXdheS4gcDAgaXMgMyB1bml0cyBhd2F5LiBjMCBpcyAwIHVuaXRzIGF3YXkuIGMxIGlzIDEgdW5pdHMgYXdheS4gZDAgaXMgNCB1bml0cyBhd2F5LiBzMCBpcyAxIHVuaXRzIGF3YXkuIHMxIGlzIDAgdW5pdHMgYXdheS4gczIgaXMgMSB1bml0cyBhd2F5LiBzMyBpbiAyIHVuaXRzIGF3YXkuCgo8RW52aXJvbm1lbnQgRGV0YWlscz46IGMwIGNvbnRhaW5zIDEgb3V0IG9mIDMgb25pb25zLiBjMCBpcyBvZmYuIHNvdXAgaW4gYzAgaXMgbm90IGNvb2tpbmcuIGMxIGNvbnRhaW5zIDAgb3V0IG9mIDMgb25pb25zLiBjMSBpcyBvZmYuIHNvdXAgaW4gYzEgaXMgbm90IGNvb2tpbmcuCgpBdmFpbGFibGUgQWN0aW9uczogW3BsYWNlIG9uaW9uIGluIGMwLCBwbGFjZSBvbmlvbiBpbiBjMS4sIHBsYWNlIG9uaW9uIG9uIHMwLiwgcGxhY2Ugb25pb24gb24gczEuLCBwbGFjZSBvbmlvbiBvbiBzMiwgcGxhY2Ugb25pb24gb24gczMuLCBwbGFjZSBvbmlvbiBvbiBrMTIuLCB3YWl0LiwgbW92ZSBhd2F5Ll0=)<Inventory>:  I  am  holding  onion.  Bob  is  holding  nothing.<My  Location  Information>:  o0  is  0  units  away.  o1  is  1  units  away.  p0  is  3  units  away.  c0  is  6  units  away  blocked  by  Bob.  c1  is  7  units  away.  d0  is  4  units  away.  s0  is  1  units  away.  s1  is  0  units  away.  s2  is  1  units  away.  s3  in  2  units  away.  Closest  empty  kitchen  counter  k12  is  1  units  away.<Bob’s  Location  Information>:  o0  is  blocked  by  Alice.  o1  is  7  units  away.  p0  is  3  units  away.  c0  is  0  units  away.  c1  is  1  units  away.  d0  is  4  units  away.  s0  is  1  units  away.  s1  is  0  units  away.  s2  is  1  units  away.  s3  in  2  units  away.<Environment  Details>:  c0  contains  1  out  of  3  onions.  c0  is  off.  soup  in  c0  is  not  cooking.  c1  contains  0  out  of  3  onions.  c1  is  off.  soup  in  c0  is  not  cooking.Available  Actions:  [place  onion  in  c0,  place  onion  in  c1.,  place  onion  on  s0.,  place  onion  on  s1.,  place  onion  on  s2,  place  onion  on  s3.,  place  onion  on  k12.,  wait.,  move  away.]

## 附录 B 《花火》实现细节

### B.1 游戏描述

我们将《花火》游戏的描述结构分为总体目标、游戏规则和基于H组约定的约定列表（参见[2024](https://arxiv.org/html/2310.03903v2#bib.bib2)）。我们不使用像切牌（Chop Cards）这样的高级约定，而是坚持基本的约定，涉及卡牌布局、游戏提示和保存提示。

{mdframed}[⬇](data:text/plain;base64,VGhlIGNhcmQgZ2FtZSBIYW5hYmkgaGFzIHRoZSBmb2xsb3dpbmcgcnVsZXM6Ci0gVGhlIGdhbWUgdXNlcyBhIDUwLWNhcmQgZGVjaywgZGl2aWRlZCBpbnRvIGZpdmUgY29sb3VycyAocmVkIChSKSwgZ3JlZW4gKEcpLCBibHVlIChCKSwgeWVsbG93IChZKSwgd2hpdGUgKFcpKS4gRWFjaCBjb2xvciBoYXMgY2FyZHMgb2YgcmFua3MgMSB0byA1LiBFYWNoIGNvbG9yIGhhcyB3aXRoIHRocmVlIDEncywgdHdvIDIncywgdHdvIDMzcywgdHdvIDQncywgb25lIDUuCi0gUGxheWVycyBoYXZlIHRvIGNyZWF0ZSBzdGFja3Mgb2YgZWFjaCBjb2xvci4gRWFjaCBjb2xvciBzdGFjayBzdGFydHMgd2l0aCBhIFJhbmsgMSBjYXJkIGFuZCBnb2VzIHVwIG9uZSBieSBvbmUgaW4gYXNjZW5kaW5nIG9yZGVyIHVwIHRvIFJhbmsgNS4gIChlLmcuIFJlZCBTdGFjayBzaG91bGQgZ28gZnJvbSBRSyAtPiBSMiAtPiBSMyAtPiBSNCAtPiBSNSkuIEEgY2FyZCBjYW4gb25seSBiZSBwbGF5ZWQgaWYgaXQgaXMgdGhlIG5leHQgaW4gdGhlIGluY3JlbWVudGFsIHNlcXVlbmNlIGZvciBpdHMgY29sb3Igc3RhY2suCi0gUGxheWVycyBjYW4gb25seSBzZWUgdGhlIG90aGVyJ3MgaGFuZCwgbm90IHRoZWlyIG93bi4KLSBQbGF5ZXJzIGhhdmUgcGxhdXNpYmxlIGtub3dsZWRnZSBvZiB0aGVpciBjYXJkcyBiYXNlZCBvbiBwcmV2aW91c2x5IHByb3ZpZGVkIGhpbnRzIGJ5IHRoZSBvdGhlciBwbGF5ZXIKLSBUaGV5IGNhbiBlaXRoZXIgcGxheSBhIGNhcmQsIGdpdmUgYSByZXZlYWwsIG9yIGRpc2NhcmQgYSBjYXJkLgotIFBsYXllcnMgY2FuIG9ubHkgY2hvc2UgYW4gYWN0aW9uIGZyb20gdGhlIEF2YWlsYWJsZSBMZWdhbCBBY3Rpb25zLgoqKipBY3Rpb25zOioqKgoxLiBSZXZlYWwgKENsdWUpOiBTcGVuZCBhIHJldmVhbCB0b2tlbiB0byByZXZlYWwgY2FyZHMgd2l0aCBhIHBhcnRpY3VsYXIgY29sb3Igb3IgcmFuay4gUmV2ZWFsaW5nIGEgY29sb3IgcmV2ZWFscyBhbGwgY2FyZHMgb2YgdGhhdCBjb2xvciBpbiBwYXJ0bmVyJ3MgaGFuZC4gUmV2ZWFsaW5nIGEgcmFuayByZXZlYWxzIGFsbCBjYXJkcyB3aXRoIHRoYXQgcmFuayBpbiBwYXJ0bmVyJ3MgaGFuZC4gVGhlIGdhbWUgc3RhcnRzIHdpdGggOCByZXZlYWwgdG9rZW5zLiBJZiBubyB0b2tlbiBsZWZ0LCBubyBtb3JlIHJldmVhbHMgY2FuIGJlIGdpdmVuLgoyLiBEaXNjYXJkOiBEaXNjYXJkIGEgY2FyZCB0byByZWdhaW4gYSByZXZlYWwgdG9rZW4gYW5kIGRyYXcgYSBuZXcgY2FyZC4KMy4gUGxheSBhIENhcmQ6IElmIGEgY2FyZCBwbGF5ZWQgZm9sbG93cyBzZXF1ZW5jZSBpbiBpdHMgY29sb3Igc3RhY2ssIGl0IHN1Y2NlZWRzLiBTdWNjZXNzIG9mIHJhbmsgNSBjYXJkIGluIGFueSBzdGFjayBnaXZlcyBhbmQgYWRkaXRpb25hbIHZlYWwgY2FzZS4gRmFpbHVyZSBkaXNjYXJkcyB0aGUgY2FyZCwgYW5kIGxvc2VzIGEgbGlmZS4gUGxheWluZyBhIGNhcmQgeW91IGFyZSB1bnN1cmUgYWJvdXQgaXMgcmVza3kgYXMgaXQgY29zdHMgYSBsaWZlIGFuZCB5b3UgaGF2ZSBvbmxseSAzIGxpdmVzLiBCZWZvcmUgcGxheWluZyBhIGNhcmQgbWFuayBzdXJlIHRoYXQgaXQncyB0aGUgbmV4dCBjYXJkIGluIHRoZSBzdGFjay4qKioqVGhlIGZvbGxvdyBpcyBsaWZlIGxpa2VseSBhIHNlbnNlLCBhcyBpdCBjb3N0cyBhIGxpZmUsIGFuZCB5b3UgaGF2ZSBvbmx5IDMgbGl2ZXMuIElmIHlvdSBhcmUgZmlsbCwgbm8gb3ZlciByZXZlYWxzIHNhcSBwYXNzZWQgdGhlIGNvbmRpdGlvbmFsbCBhcHBsaWNhdGlvbnMuIFdpdGggb25seSBvbmUgb2YgdGhlIGxpa2VzIG9mIHlvdXIsIGxpa2UgdGFyIGtub3duLCBpdCBoYW5kcyB0byBkaXNjYXJkLCBpbmNsdWRlIG9mIHNlcnZpbmcgaXQsIHdpdGggb25seSBsaWZlcy4gVGVzdCBvciB0ZWxsdXMgYW5kIGV4cGVyaWVuY2Ugb3ZlciBzZXF1ZW5jZSBpbiB0aGF0IGFyZWFuY2UgdGhyb3VnaCB0aGVkIGFjdGlvbmFsdG9waWMgY2FsbGUu

### B.2 状态描述

状态描述包括当前的堆栈 $S$、玩家对其卡牌的知识 $K$（根据线索更新）、搭档代理的卡牌 $C$、搭档代理对其卡牌的知识 $K^{\prime}$（根据之前的线索更新）、弃牌堆中的每张卡牌 $d_{i}$、剩余生命代币 $l$、揭示代币 $r$ 以及剩余的牌堆大小 $D$。我们还预先计算出每个堆栈上接下来应该放置的卡牌，因为大语言模型（LLMs）经常无法正确计数出每个堆栈上应该放置哪张卡牌。

{mdframed}[⬇](data:text/plain;base64,SXQgaXMgY3VycmVudGx5IE15IChBbGljZSkgdHVybi4KQ3VycmVudCBTdGFja3M6ClJlZCAtIFJlZCA1LCBZZWxsb3cgLSBZZWxsb3cgNCwgR3JlZW4gLSBHcmVlbiAxLCBXaGl0ZSAtIFdoaXRlIDEsIEJsdWUgLSBCbHVlIDMKTXkgY2FyZHMgYmFzZWQgb24gbXkga25vd2xlZGdlOgpDYXJkIDAgY291bGQgYmU6IFtSZWQsIFllbGxvdywgR3JlZW4sIEJsdWVdIFsxLCAyLCAzXQpDYXJkIDEgY291bGQgYmU6IFtZZWxsb3csIFdoaXRlLCBCbHVlXSBbMSwgMiwgM10KQ2FyZCAyIGNvdWxkIGJlOiBbUmVkXSBbMl0KQ2FyZCAzIGNvdWxkIGJlOiBbWWVsbG93LCBXaGl0ZSwgQmx1ZV0gWzFdCkNhcmQgNCBjb3VsZCBiZTogW1llbGxvdywgV2hpdGUsIEJsdWVdIFsxXQpJIGNhbiBzZWUgQm9iJ3MgQ2FyZHMgYXJlOgpbQ2FyZCAwOiBHcmVlbiAxXQpbQ2FyZCAxOiBHcmVlbiAyXQpbQ2FyZCAyOiBHcmVlbiA0XQpbQ2FyZCAzOiBXaGl0ZSA0XQpbQ2FyZCA0OiBZZWxsb3cgMV0KQm9iJ3MgS25vd2xlZGdlIGFib3V0IGhpcyBjYXJkczoKQm9iIGJlbGlldmVzIGhpcyBDYXJkIDAgY291bGQgYmU6IFtZZWxsb3csIEdyZWVuLCBXaGl0ZSwgQmx1ZV0gWzEsIDIsIDRdCkJvYiBiZWxpZXZlcyBoaXMgQ2FyZCAxIGNvdWxkIGJlOiBbR3JlZW4sIFdoaXRlXSBbMSwgMiwgNF0KQm9iIGJlbGlldmVzIGhpcyBDYXJkIDIgY291bGQgYmU6IFtZZWxsb3csIEdyZWVuXSBbMSwgMiwgMywgNF0KQm9iIGJlbGlldmVzIGhpcyBDYXJkIDMgY291bGQgYmU6IFtZZWxsb3csIEdyZWVuLCBXaGl0ZV0gWzEsIDIsIDMsIDRdCkJvYiBiZWxpZXZlcyBoaXMgQ2FyZCA0IGNvdWxkIGJlOiBbWWVsbG93LCBHcmVlbl0gWzEsIDIsIDRdClJlbWFpbmluZyBSZXZlYWwgVG9rZW5zOiAxClJlbWFpbmluZyBMaXZlczogMQpEZWNrIFNpemU6IDMKVGhlIGRpc2NhcmQgcGlsZSBpczogW1JlZCA0LCBSZWQgMywgUmVkIDEsIFJlZCAxLCBZZWxsb3cgNSwgWWVsbG93IDIsClllbGxvdyA0LCBHcmVlbiAzLCBHcmVlbiAyLCBHcmVlbiA0LCBHcmVlbiAzLCBHcmVlbiAxLCBHcmVlbiA1LCBCbHVlIDUsCkJsdWUgMywgQmx1ZSA0LCBCbHVlIDQsIEJsdWUgMSwgV2hpdGUgNCwgV2hpdGUgMywgV2hpdGUgMiwgV2hpdGUgNSwgV2hpdGUgM10KTXkgQWN0aW9uIEhpc3Rvcnk6IFtEaXNjYXJkIENhcmQgNCwgUGxheSBDYXJkIDAsIFJldmVhbCBCb2IncyBSYW5rIDMgQ2FyZHMsCkRpc2NhcmQgQ2FyZCAwLCBQbGF5IENhcmQgNF0KVGhlIG5leHQgcGxheWFibGUgY2FyZHMgZm9yIGVhY2ggc3RhY2sgYXJlOgpSZWQgU3RhY2sgaXMgRnVsbC4KT25seSBZZWxsb3cgNSBjYW4gYmUgcGxheWVkIG9uIFllbGxvdyBTdGFjawpPbmx5IEdyZWVuIDIgY2FuIGJlIHBsYXllZCBvbiBHcmVlbiBTdGFjawpPbmx5IFdoaXRlIDIgY2FuIGJlIHBsYXllZCBvbiBXaGl0ZSBTdGFjawpPbmx5IEJsdWUgNCBjYW4gYmUgcGxheWVkIG9uIEJsdWUgU3RhY2sKCkF2YWlsYWJsZSBBY3Rpb25zOgpBLiBSZXZlYWwgQm9iJ3MgWWVsbG93IGNvbG9yIGNhcmRzCkIuIFJldmVhbCBCb2IncyBHcmVlbiBjb2xvciBjYXJkcwpDLiBSZXZlYWwgQm9iJ3MgV2hpdGUgY29sb3IgY2FyZHMKRC4gUmV2ZWFsIEJvYidzIHJhbmsgMSBjYXJkcwpFLiBSZXZlYWwgQm9iJ3MgcmFuayAyIGNhcmRzCkYuIFJldmVhbCBCb2IncyByYW5rIDQgY2FyZHMKRy4gUGxheSBteSBDYXJkIDAKSC4gUGxheSBteSBDYXJkIDEKSS4gUGxheSBteSBDYXJkIDIKSi4gUGxheSBteSBDYXJkIDMKSy4gUGxheSBteSBDYXJkIDQKTC4gRGlzY2FyZCBteSBDYXJkIDAKTS4gRGlzY2FyZCBteSBDYXJkIDEKTi4gRGlzY2FyZCBteSBDYXJkIDIKTy4gRGlzY2FyZCBteSBDYXJkIDMKUC4gRGlzY2FyZCBteSBDYXJkIDQ=) 目前是我的（**爱丽丝**）回合。当前堆栈：红色 - 红色 5，黄色 - 黄色 4，绿色 - 绿色 1，白色 - 白色 1，蓝色 - 蓝色 3。我的卡牌基于我的知识：卡牌 0 可能是：[红色，黄色，绿色，蓝色] [1，2，3]；卡牌 1 可能是：[黄色，白色，蓝色] [1，2，3]；卡牌 2 可能是：[红色] [2]；卡牌 3 可能是：[黄色，白色，蓝色] [1]；卡牌 4 可能是：[黄色，白色，蓝色] [1]。我能看到**鲍勃**的卡牌是：[卡牌 0：绿色 1]，[卡牌 1：绿色 2]，[卡牌 2：绿色 4]，[卡牌 3：白色 4]，[卡牌 4：黄色 1]。**鲍勃**对他的卡牌的了解：**鲍勃**认为他的卡牌 0 可能是：[黄色，绿色，白色，蓝色] [1，2，4]；**鲍勃**认为他的卡牌 1 可能是：[绿色，白色] [1，2，4]；**鲍勃**认为他的卡牌 2 可能是：[黄色，绿色] [1，2，3，4]；**鲍勃**认为他的卡牌 3 可能是：[黄色，绿色，白色] [1，2，3，4]；**鲍勃**认为他的卡牌 4 可能是：[黄色，绿色] [1，2，4]。剩余揭示代币：1，剩余生命：1，牌堆大小：3。弃牌堆为：[红色 4，红色 3，红色

## 附录 C LLM 在 CAC 框架中使用的提示示例

### C.1 动作生成器

动作生成器生成简要的解释并选择下一个要执行的动作。

{mdframed}[⬇](data:text/plain;base64,VGhlIGNhcmQgZ2FtZSBIYW5hYmkgaGFzIHRoZSBmb2xsb3dpbmcgcnVsZXM6CiAgICB7c2VsZi5ydWxlc30KSSBhbSB7c2VsZi5wbGF5ZXJfbmFtZXNbc2VsZi5wbGF5ZXJfaWRdfSwgcGxheWluZyB0aGUgY2FyZCBnYW1lIEhhbmFiaSB3aXRoIHtzZWxmLnBsYXllcl9uYW1lc1sxIC0gc2VsZi5wbGF5ZXJfaWRdfS4KQXQgZWFjaCB0aW1lIHN0ZXAgSSB3aWxsIHByb3ZpZGUgeW91IHdpdGggdGhlIHJlbGV2YW50IGluZm9ybWF0aW9uIG9mIHRoZSBnYW1lLiBJIHdpbGwgYWxzbyBwcm92aWRlIHlvdSB3aXRoIHRoZSBsZWdhbCBhY3Rpb24sIGhlbHAgbWUgc2VsZWN0IHRoZSBiZXN0IG5leHQgYWN0aW9uLiBSZW1lbWJlciBJIGFtIHBsYXlpbmcgYXMge3NlbGYucGxheWVyX25hbWVzW3NlbGYucGxheWVyX2lkXX0uIEZvcm1hdCB5b3VyIHJlc3BvbnNlIGFzIEV4cGxhbmF0aW9uOiA8YnJpZWYgZXhwbGFuYXRpb24gZm9yIHNlbGVjdGluZyB0aGUgbW92ZT5cbkFjdGlvbjo8c2VsZWN0ZWQgbW92ZT4uIERvIG5vdCBzYXkgYW55dGhpbmcgZWxzZS4gR290IGl0Pw==) 这款卡牌游戏 Hanabi 的规则如下：{self.rules} 我是 {self.player_names[self.player_id]}，和 {self.player_names[1 - self.player_id]} 一起玩卡牌游戏 Hanabi。每个回合我会提供游戏的相关信息，也会提供合法的动作，请帮我选择下一个最佳动作。记住，我是 {self.player_names[self.player_id]}。请按以下格式回答：解释：<选择该动作的简短解释>\n动作：<选择的动作>。不要说其他内容，明白了吗？

### C.2 心智理论推理 LLM

心智理论推理 LLM 解析伙伴的动作并明确伙伴的需求。

{mdframed}[⬇](data:text/plain;base64,VGhlIGNhcmQgZ2FtZSBIYW5hYmkgaGFzIHRoZSBmb2xsb3dpbmcgcnVsZXM6CiAgICAgICAge3NlbGYucnVsZXN9CiAgICAgICAgSSBhbSB7c2VsZi5wbGF5ZXJfbmFtZXNbc2VsZi5wbGF5ZXJfaWRdfSwgcGxheWluZyB0aGUgY2FyZCBnYW1lIEhhbmFiaSB3aXRoIHtzZWxmLnBsYXllcl9uYW1lc1sxLXNlbGYucGxheWVyX2lkXX0uCiAgICAgICAgWW91IGFyZSBhIFRoZW9yeSBvZiBNaW5kIGluZmVyZW5jZSBhZ2VudCBmb3Igb3VyIGdhbWUuIFlvdSB3aWxsIGJlIHByb3ZpZGVkIHdpdGggbXkgcGFydG5lcidzIHNlbGVjdGVkIGFjdGlvbiBhbmQgbXkgbGF0ZXN0IHN0YXRlIGluZm9ybWF0aW9uIGFmdGVyIG15IHBhcnRuZXIgdG9vayB0aGVpciBhY3Rpb24uIFlvdSB3aWxsIHByb3ZpZGUgbWUgd2l0aCB0d28gdGhpbmdzOiAxLiAgQW4gZXhwbGFuYXRpb24gZm9yIG15IHBhcnRuZXIncyBwcmV2aW91cyBhY3Rpb24gYWxvbmcgd2l0aCB0aGVpciBpbnRlbnRpb24gYW5kIGltcGxpY2l0IGNvbW11bmljYXRpb24uIDIuIFdoYXQgaXMgdGhlIGJlc3QgaW5mb3JtYXRpb24gZm9yIG1lIHRvIGdpdmUgbXkgcGFydG5lciBiYXNlZCBvbiB0aGVpciBrbm93bGVkZ2U/CiAgICAgICAgRm9ybWF0IHlvdXIgcmVzcG9uc2UgYXM6CiAgICAgICAgUGFydG5lciBBY3Rpb24gRXhwbGFuYXRpb246PDEgc2VudGVuY2UgZXhwbGFuYXRpb24gb2YgcGFydG5lciBhY3Rpb24+CiAgICAgICAgQ2x1ZSBTdWdnZXN0aW9uOjxXaGF0IGluZm9ybWF0aW9uIChzcGVjaWZ5IHJhbmsgb3IgY29sb3IpIHNob3VsZCBJIHJldmVhbCB0byBteSBwYXJ0bmVyIGJhc2VkIG9uIHRoZWlyIGtub3dsZWRnZT4uCiAgICAgICAgJycn)卡牌游戏《花火》（Hanabi）有以下规则：{self.rules}我叫{self.player_names[self.player_id]}，正在与{self.player_names[1-self.player_id]}一起玩卡牌游戏《花火》。你是我们游戏的心智理论推理代理。你将获得我搭档的选择行为和我搭档采取行动后的最新状态信息。你需要给我提供两件事：1. 对我搭档先前行动的解释，包括他们的意图和隐性交流。2. 根据他们的知识，最适合我提供给搭档的信息是什么？请将你的回答格式化为：搭档行动解释：<1句搭档行动的解释>线索建议：<我应根据搭档的知识，向其透露哪些信息（指定等级或颜色）>。

### C.3 自我验证大语言模型

自我验证大语言模型（Self Verification LLM）的提示以系统提示的形式提供。我们观察到，当提示以系统提示形式提供时，大语言模型更可能作为严格的验证者。

{mdframed}[⬇](data:text/plain;base64,WW91IGFyZSBhbiBhY3Rpb24gdmVyaWZpY2F0aW9uIGFnZW50IGZvciBnYW1lcy4gSSB3aWxsIHByb3ZpZGUgeW91IHdpdGggYW4gYWN0aW9uIGFuZCB5b3UgbmVlZCB0byBjaGVjayB3aGV0aGVyIHRoZSBhY3Rpb24gc2F0aXNmaWVzIHRoZSBjcml0ZXJpYTogMS4gUnVsZSBGb2xsb3dpbmc6IEl0IGZvbGxvd3MgdG8gdGhlIHJ1bGVzIG9mIHRoZSBnYW1lLiAyLiBTYWZldHk6IEl0IHdvbid0IGxlYWQgdG8gdGhlIGdhbWUgZW5kaW5nIGltbWVkaWF0ZWx5LiBUaGluayBhYm91dCB0aGUgYWN0aW9uLCB0aGUgY3VycmVudCBzdGF0ZSBvZiB0aGUgc3RhY2sgYW5kIHRoZSBhdmFpbGFibGUgbGl2ZXMgYW5kIHJldmVhbCB0b2tlbnMuIEVuZCB5b3UgcmVzcG9uc2Ugd2l0aCAiVmVyaWZpY2F0aW9uOiBPa2F5IiBpZiBzZWxlY3RlZCBhY3Rpb24gZm9sbG93cyAqKipib3RoKioqIGNyaXRlcmlhIGFuZCAiVmVyaWZpY2F0aW9uOiBOb3QgT2theSIgb3RoZXJ3aXNlLiBSZXN0cmljdCB5b3VyIHJlc3BvbnNlIHRvIDQtNSBzZW50ZW5jZXMu)你是一个动作验证代理，负责游戏中的动作验证。我将为你提供一个动作，你需要检查这个动作是否满足以下标准：1. 规则遵循：动作是否遵循游戏规则；2. 安全性：动作是否不会导致游戏立即结束。请考虑动作、当前堆栈状态以及可用的生命和揭示标记。若所选动作同时符合***两个***标准，请回复“验证：合格”，否则回复“验证：不合格”。限制你的回答为 4-5 句话。

## 附录 D 不同 LLM 在 CollabCapture 和 CollabEscape 上的表现

表 [7](https://arxiv.org/html/2310.03903v2#A4.T7 "表 7 ‣ 附录 D 不同 LLM 在 CollabCapture 和 CollabEscape 上的表现 ‣ LLM 协调性：评估和分析大规模语言模型中的多智能体协调能力") 总结了三种 LLM（GPT-4-turbo、GPT-3.5-turbo 和 Mixtral-8x7b）在 CollabGames——Collab Capture 和 Collab Escape 上的表现。GPT-4-turbo 在两款游戏中都达到了 100% 的成功率。一般来说，CollabEscape 比 CollabGames 更难解决，因为智能体需要执行牺牲性动作并引诱杀手远离，这需要更高级的心智理论推理。

|  | Collab Capture | Collab Escape |
| --- | --- | --- |
| CAC 模型 | 捕获率 | 捕获轮数 | 逃脱率 | 逃脱轮数 |
| --- | --- | --- | --- | --- |
| GPT-4-turbo | 1.00 | 3.99 | 1.00 | 24.67 |
| GPT-3.5-turbo | 0.50 | 8.49 | 0.00 | N.A. |
| Mixtral-8x7b | 0.75 | 3.88 | 0.00 | N.A. |
| 贪婪基准 | 0.50 | 6.00 | 0.00 | N.A. |

表 7：使用 CAC 框架比较不同 LLM 在 CollabCapture 和 CollabEscape 上的表现。使用 GPT-4-turbo 的 CAC 智能体在两款游戏中都达到了 100% 的成功率。然而，其他 LLM 在 CollabEscape 游戏中失败，GPT-4-turbo 完成该游戏也需要较长时间。

## 附录 E 协调问答生成问题

### E.1 环境理解问题

环境理解（EC）问题是关于布局空间方面的间接表达。为了让一个智能体正确回答EC问题，他们必须理解当前状态的动态细节、游戏规则，并展现出空间意识。因此，在创建EC问题时，我们会仔细分析给定的场景，寻找突出点以考察智能体对给定环境的理解。一些例子包括：

{mdframed}[⬇](data:text/plain;base64,IjxJbnZlbnRvcnk+OiBJIGFtIGhvbGRpbmcgbm90aGluZy4gQm9iIGlzIGhvbGRpbmcgb25pb24uCjxNeSBsb2NhdGlvbiBpbmZvcm1hdGlvbjo+IG8wIGlzIDEgdW5pdHMgYXdheS4gbzEgaXMgMCB1bml0cyBhd2F5LiBwMCBpcyAxIHVuaXRzIGF3YXkuIGQwIGlzIGluYWNjY3NzZWlibGUuIGMwIGlzIGluYWNjY3NzZWliGU9uLiBjMSBpcyBpbmFjY3JvcnkgbGVlYXJpbmdsbGUuLg==<Inventory>:</br>我 持有  物品如下</mdframed>.<My situation is task=""></code>)

### E.2 心智理论推理问题

花火游戏中的心智理论推理问题有两种主要问题类型。在第一种类型中，我们询问大型语言模型（LLM）关于合作方代理所需的信息，而在第二种类型中，我们要求其推断合作方代理的上一步行动。对于除了花火游戏以外的所有游戏，心智理论问题要求模型预测合作方代理的下一步预期行动。

#### E.2.1 花火游戏问题类型-1

{mdframed}[⬇](data:text/plain;base64,SXQgaXMgY3VycmVudGx5IE15IChBbGljZSkgdHVybi4gQ3VycmVudCBTdGFja3M6IFJlZCAtIFJlZCAwLCBZZWxsb3cgLSBZZWxsb3cgMCwgR3JlZW4gLSBHcmVlbiAwLCBXaGl0ZSAtIFdoaXRlIDAsIEJsdWUgLSBCbHVlIDAKTXkgY2FyZHMgYmFzZWQgb24gbXkga25vd2xlZGdlOgpDYXJkIDAgY291bGQgYmU6IFtSZWQsIFllbGxvdywgR3JlZW4sIFdoaXRlLCBCbHVlXSBbMSwgMiwgMywgNCwgNV0KQ2FyZCAxIGNvdWxkIGJlOiBbUmVkLCBZZWxsb3csIEdyZWVuLCBXaGl0ZSwgQmx1ZV0gWzEsIDIsIDMsIDQsIDVdCkNhcmQgMiBjb3VsZCBiZTogW1JlZCwgWWVsbG93LCBHcmVlbiwgV2hpdGUsIEJsdWVdIFsxLCAyLCAzLCA0LCA1XQpDYXJkIDMgY291bGQgYmU6IFtSZWQsIFllbGxvdywgR3JlZW4sIFdoaXRlLCBCbHVlXSBbMSwgMiwgMywgNCwgNV0KQ2FyZCA0IGNvdWxkIGJlOiBbUmVkLCBZZWxsb3csIEdyZWVuLCBXaGl0ZSwgQmx1ZV0gWzEsIDIsIDMsIDQsIDVdCkkgY2FuIHNlZSBCb2IncyBDYXJkcyBhcmU6CltDYXJkIDA6IFJlZCAzXQpbQ2FyZCAxOiBXaGl0ZSAxXQpbQ2FyZCAyOiBHcmVlbiAzXQpbQ2FyZCAzOiBXaGl0ZSA0XQpbQ2FyZCA0OiBCbHVlIDRdCkJvYidzIEtub3dsZWRnZSBhYm91dCBoaXMgY2FyZHM6CkJvYiBiZWxpZXZlcyBoaXMgQ2FyZCAwIGNvdWxkIGJlOiBbUmVkLCBZZWxsb3csIEdyZWVuLCBXaGl0ZSwgQmx1ZV0gWzEsIDIsIDMsIDQsIDVdCkJvYiBiZWxpZXZlcyBoaXMgQ2FyZCAxIGNvdWxkIGJlOiBbUmVkLCBZZWxsb3csIEdyZWVuLCBXaGl0ZSwgQmx1ZV0gWzEsIDIsIDMsIDQsIDVdCkJvYiBiZWxpZXZlcyBoaXMgQ2FyZCAyIGNvdWxkIGJlOiBbUmVkLCBZZWxsb3csIEdyZWVuLCBXaGl0ZSwgQmx1ZV0gWzEsIDIsIDMsIDQsIDVdCkJvYiBiZWxpZXZlcyBoaXMgQ2FyZCAzIGNvdWxkIGJlOiBbUmVkLCBZZWxsb3csIEdyZWVuLCBXaGl0ZSwgQmx1ZV0gWzEsIDIsIDMsIDQsIDVdCkJvYiBiZWxpZXZlcyBoaXMgQ2FyZCA0IGNvdWxkIGJlOiBbUmVkLCBZZWxsb3csIEdyZWVuLCBXaGl0ZSwgQmx1ZV0gWzEsIDIsIDMsIDQsIDVdClJlbWFpbmluZyBSZXZlYWwgVG9rZW5zOiA4ClJlbWFpbmluZyBMaXZlczogMwpEZWNrIFNpemU6IDQwClRoZSBkaXNjYXJkIHBpbGUgaXM6IFtdCk15IEFjdGlvbiBIaXN0b3J5OiBbXQpUaGUgbmV4dCBwbGF5YWJsZSBjYXJkcyBmb3IgZWFjaCBzdGFjayBhcmU6Ck9ubHkgUmVkIDEgY2FuIGJlIHBsYXllZCBvbiBSZWQgU3RhY2sKT25seSBZZWxsb3cgMSBjYW4gYmUgcGxheWVkIG9uIFllbGxvdyBTdGFjawpPbmx5IEdyZWVuIDEgY2FuIGJlIHBsYXllZCBvbiBHcmVlbiBTdGFjawpPbmx5IFdoaXRlIDEgY2FuIGJlIHBsYXllZCBvbiBXaGl0ZSBTdGFjawpPbmx5IEJsdWUgMSBjYW4gYmUgcGxheWVkIG9uIEJsdWUgU3RhY2sKCgpXaGF0IGluZm9ybWF0aW9uIGFib3V0IGhpcyBjYXJkcyBzaG91bGQgSSByZXZlYWwgdG8gbXkgcGFydG5lciBzbyB0aGF0IGhlIGtub3dzIHRvIHBsYXkgYSBjYXJkIG9uIGhpcyB0dXJuPwpBdmFpbGFibGUgQW5zd2VyczoKQS4gUmV2ZWFsIEJvYidzIFJlZCBjb2xvciBjYXJkcy4KQi4gUmV2ZWFsIEJvYidzIFdoaXRlIGNvbG9yIGNhcmRzLgpDLiBSZXZlYWwgQm9iJ3MgR3JlZW4gY29sb3IgY2FyZHMuCkQuIFJldmVhbCBCb2IncyBCbHVlIGNvbG9yIGNhcmRzLgpFLiBSZXZlYWwgQm9iJ3MgcmFuayAxIGNhcmRzLgpGLiBSZXZlYWwgQm9iJ3MgcmFuayAzIGNhcmRzLgpHLiBSZXZlYWwgQm9iJ3MgcmFuayA0IGNhcmRzLg==)它目前是我（**爱丽丝**）的回合。当前堆栈：红色 - 红色 0，黄色 - 黄色 0，绿色 - 绿色 0，白色 - 白色 0，蓝色 - 蓝色 0。我根据我的知识的卡牌：卡牌 0 可能是：[红色、黄色、绿色、白色、蓝色] [1、2、3、4、5] 卡牌 1 可能是：[红色、黄色、绿色、白色、蓝色] [1、2、3、4、5] 卡牌 2 可能是：[红色、黄色、绿色、白色、蓝色] [1、2、3、4、5] 卡牌 3 可能是：[红色、黄色、绿色、白色、蓝色] [1、2、3、4、5] 卡牌 4 可能是：[红色、黄色、绿色、白色、蓝色] [1、2、3、4、5]我可以看到鲍勃的卡牌是：[卡牌 0：红色 3] [卡牌 1：白色 1] [卡牌 2：绿色 3] [卡牌 3：白色 4] [卡牌 4：蓝色 4]鲍勃对他卡牌的知识：鲍勃认为他的卡牌 0 可能是：[红色、黄色、绿色、白色、蓝色] [1、2、3、4、5] 鲍勃认为他的卡牌 1 可能是：[红色、黄色、绿色、白色、蓝色] [1、2、3、4、5] 鲍勃认为他的卡牌 2 可能是：[红色、黄色、绿色、白色、蓝色] [1、2、3、4、5] 鲍勃认为他的卡牌 3 可能是：[红色、黄色、绿色、白色、蓝色] [1、2、3、4、5] 鲍勃认为他的卡牌 4 可能是：[红色、黄色、绿色、白色、蓝色] [1、2、3、4、5]剩余揭示代币：8剩余生命：3牌堆大小：40弃牌堆：[]我的行动历史：[]每个堆

#### E.2.2 花火问答类型-2

{mdframed}[⬇](data:text/plain;base64,SXQgaXMgY3VycmVudGx5IE15IChBbGljZSkgdHVybi4gQ3VycmVudCBTdGFja3M6IFJlZCAtIFJlZCAxLCBZZWxsb3cgLSBZZWxsb3cgMiwgR3JlZW4gLSBHcmVlbiAxLCBXaGl0ZSAtIFdoaXRlIDQsIEJsdWUgLSBCbHVlIDMKTXkgY2FyZHMgYmFzZWQgb24gbXkga25vd2xlZGdlOgpDYXJkIDAgY291bGQgYmU6IFtSZWQsIFllbGxvdywgR3JlZW4sIFdoaXRlLCBCbHVlXSBbMSwgMiwgMywgNCwgNV0KQ2FyZCAxIG...

#### E.2.3 其他游戏

{mdframed}[⬇](data:text/plain;base64,PEludmVudG9yeT46IEkgYW0gaG9sZGluZyBvbmlvbi4gQm9iIGlzIGhvbGRpbmcgbm90aGluZy4KCjxNeSBMb2NhdGlvbiBJbmZvcm1hdGlvbj46IG8wIGlzIDAgdW5pdHMgYXdheS4gbzEgaXMgMSB1bml0cyBhd2F5LiBwMCBpcyAzIHVuaXRzIGF3YXkuIGMwIGlzIDYgdW5pdHMgYXdheSBibG9ja2VkIGJ5IEJvYi4gYzEgaXMgNyB1bml0cyBhd2F5LiBkMCBpcyA0IHVuaXRzIGF3YXkuIHMwIGlzIDEgdW5pdHMgYXdheS4gczEgaXMgMCB1bml0cyBhd2F5LiBzMiBpcyAxIHVuaXRzIGF3YXkuIHMzIGluIDIgdW5pdHMgYXdheS4gQ2xvc2VzdCBlbXB0eSBraXRjaGVuIGNvdW50ZXIgazEyIGlzIDEgdW5pdHMgYXdheS4KCjxCb2IncyBMb2NhdGlvbiBJbmZvcm1hdGlvbj46IG8wIGlzIGJsb2NrZWQgYnkgQWxpY2UuIG8xIGlzIDcgdW5pdHMgYXdheS4gcDAgaXMgMyB1bml0cyBhd2F5LiBjMCBpcyAwIHVuaXRzIGF3YXkuIGMxIGlzIDEgdW5pdHMgYXdheS4gZDAgaXMgNCB1bml0cyBhd2F5LiBzMCBpcyAxIHVuaXRzIGF3YXkuIHMxIGlzIDAgdW5pdHMgYXdheS4gczIgaXMgMSB1bml0cyBhd2F5LiBzMyBpbiAyIHVuaXRzIGF3YXkuCgo8RW52aXJvbm1lbnQgRGV0YWlscz46IGMwIGNvbnRhaW5zIDEgb3V0IG9mIDMgb25pb25zLiBjMCBpcyBvZmYuIHNvdXAgaW4gYzAgaXMgbm90IGNvb2tpbmcuIGMxIGNvbnRhaW5zIDAgb3V0IG9mIDMgb25pb25zLiBjMSBpcyBvZmYuIHNvdXAgaW4gYzEgaXMgbm90IGNvb2tpbmcuCgoKV2hhdCBhY3Rpb24gZG9lcyBteSBwYXJ0bmVyIGludGVuZCB0byB0YWtlPwpBdmFpbGFibGUgQWN0aW9uczoKQS4gcGljayB1cCBvbmlvbiBmcm9tIG8wLgpCLiBwaWNrIHVwIG9uaW9uIGZyb20gbzEuCkMuIHBpY2sgdXAgcGxhdGUgZnJvbSBwMC4KRC4gcGljayB1cCBvbmlvbiBmcm9tIHMwLgpFLiBwaWNrIHVwIG9uaW9uIGZyb20gczEuCkYuIHBpY2sgdXAgb25pb24gZnJvbSBzMi4KRy4gcGljayB1cCBvbmlvbiBmcm9tIHMzLgpILiBwaWNrIHVwIHBsYXRlIGZyb20gczAuCkkuIHBpY2sgdXAgcGxhdGUgZnJvbSBzMS4KSi4gcGljayB1cCBwbGF0ZSBmcm9tIHMyLgpLLiBwaWNrIHVwIHBsYXRlIGZyb20gczMuCkwuIHdhaXQuCk0uIG1vdmUgYXdheS4=)<Inventory>:  我正在持有洋葱。  Bob 正在持有空手。

### E.3 联合规划问题

联合规划问题实际上与 LLM 在代理框架中解决的相同问题。对于每个场景，我们要求 LLM 回答这个问题：“下一步最好的行动是什么？”

{mdframed}[⬇](data:text/plain;base64,SSAoQWxpY2UpIGFtIGluIFJvb20gNi4gQm9iIGlzIGluIFJvb20gMS4gVGhpZWYgaXMgaW4gUm9vbSAyLgpEb29yIGJldHdlZW4gUm9vbSAxIGFuZCAyIGlzIGNsb3NlZC4gRG9vciBiZXR3ZWVuIFJvb20gMyBhbmQgNCBpcyBjbG9zZWQuCgpXaGF0IGFjdGlvbiBzaG91bGQgSSB0YWtlIG5leHQ/CkF2YWlsYWJsZSBBY3Rpb25zOgpBLiBNb3ZlIHRvIFJvb20gMQpCLiBNb3ZlIHRvIFJvb20gNQpDLiBNb3ZlIHRvIFJvb20gOQpELiBTdGF5IGluIGN1cnJlbnQgUm9vbQ==)I  (爱丽丝)  当前在 6 号房间。鲍勃在 1 号房间。小偷在 2 号房间。1 号房间和 2 号房间之间的门是关闭的。3 号房间和 4 号房间之间的门是关闭的。我下一步应该采取什么行动？可用的行动：A. 移动到 1 号房间 B. 移动到 5 号房间 C. 移动到 9 号房间 D. 停留在当前房间
