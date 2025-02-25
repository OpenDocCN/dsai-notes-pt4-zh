<!--yml  

分类：未分类  

日期：2025-01-11 12:03:15  

-->

# CoPS：通过可证明的跨任务经验共享赋能LLM智能体  

> 来源：[https://arxiv.org/html/2410.16670/](https://arxiv.org/html/2410.16670/)  

陈阳    赵晨阳^∗    关全全     周东若^§ 同等贡献 计算机科学系，印第安纳大学布卢明顿分校，印第安纳州47408，美国；电子邮件：cya2@iu.edu 计算机科学系，加利福尼亚大学洛杉矶分校，加利福尼亚州90095，美国；电子邮件：zhaochenyang@cs.ucla.edu 联合通讯作者 计算机科学系，加利福尼亚大学洛杉矶分校，加利福尼亚州90095，美国；电子邮件：qgu@cs.ucla.edu 计算机科学系，印第安纳大学布卢明顿分校，印第安纳州47408，美国；电子邮件：dz13@iu.edu  

###### 摘要  

大型语言模型（LLMs）显著推动了智能体系统中的顺序推理，但现有方法面临一些限制。反射驱动的推理仅依赖于预训练模型中的知识，这在新颖场景中限制了性能；而经验辅助推理则通常依赖外部经验，并且缺乏选择代表性经验的明确原则。我们通过提出CoPS（跨任务经验共享），解决了这些限制，这是一个可推广的算法，通过跨任务经验共享和选择增强了顺序推理。具体而言，CoPS利用智能体在先前任务中的经验，通过可证明的悲观主义策略选择分布匹配的经验，以最大化效用并最小化分布变化带来的风险。我们在Alfworld、Webshop和HotPotQA等基准上的广泛实验结果表明，CoPS始终优于现有的最先进基准，具有更高的样本效率，适用于资源受限的场景。从理论上讲，我们展示了我们的算法的表现取决于预训练LLM的质量，以及智能体的任务相关试验分布与LLM生成的分布之间的匹配程度。我们的工作弥合了现有顺序推理范式之间的差距，并验证了利用跨任务经验的有效性，为提升智能体在多任务间的泛化能力和适应性提供了新的视角。我们的代码可在[https://github.com/uclaml/COPS](https://github.com/uclaml/COPS)获取。  

## 1 引言  

由先进的大型语言模型（LLMs）驱动的蓬勃发展的智能体系统（Devlin 等， [2019](https://arxiv.org/html/2410.16670v1#bib.bib8)；Brown 等， [2020](https://arxiv.org/html/2410.16670v1#bib.bib2)；OpenAI， [2023](https://arxiv.org/html/2410.16670v1#bib.bib31)；Hu 等， [2024a](https://arxiv.org/html/2410.16670v1#bib.bib16)）已经展现了在通过顺序推理解决复杂任务方面的显著能力（Qin 等， [2024](https://arxiv.org/html/2410.16670v1#bib.bib33)；Hao 等， [2023](https://arxiv.org/html/2410.16670v1#bib.bib13)；Huang 等， [2024b](https://arxiv.org/html/2410.16670v1#bib.bib20)；Chen 等， [2024c](https://arxiv.org/html/2410.16670v1#bib.bib6)， [b](https://arxiv.org/html/2410.16670v1#bib.bib5)；Li 等， [2023a](https://arxiv.org/html/2410.16670v1#bib.bib26)）。这些智能体系统采用了两种典型的顺序推理范式：反思驱动推理和经验辅助推理。反思驱动推理通过反思（Shinn 等， [2024](https://arxiv.org/html/2410.16670v1#bib.bib36)）、长期回放（Zhou 等， [2023](https://arxiv.org/html/2410.16670v1#bib.bib61)）或链式思维（CoT）推理（Wei 等， [2022](https://arxiv.org/html/2410.16670v1#bib.bib48)）等方法，利用模型的内部能力。虽然这种方法利用了预训练模型中的知识，但也面临着显著的限制。具体而言，单纯依赖预训练模型中已有的知识来生成推理限制了模型在遇到新场景时的表现。此外，还存在幻觉的风险，即内部推理可能导致看似合理但实际上错误的回应（Huang 等， [2023](https://arxiv.org/html/2410.16670v1#bib.bib19)）。这些挑战凸显了集成外部经验以增强智能体顺序推理能力的必要性。

相比之下，经验辅助顺序推理利用基于检索的方法，使得智能体能够与经验记忆库互动，从而帮助模型克服知识截止问题、个性化回应并减少幻觉。然而，这些经验通常是手动策划或来自专家模型（Raparthy 等， [2023](https://arxiv.org/html/2410.16670v1#bib.bib34)），这需要大量资源，并且存在可扩展性问题。此外，经验辅助推理通常缺乏选择具有代表性示例的明确原则（Kagaya 等， [2024](https://arxiv.org/html/2410.16670v1#bib.bib23)），可能未能充分利用过去经验的价值。这些局限性引出了一个关键的研究问题：

![参考说明](img/5d14f1bbb8361188f1d68c8565e104e4.png)

图1：CoPS的简要说明，充分利用智能体的跨任务经验，通过共享和选择来自先前任务轨迹中的分布匹配经验来增强顺序推理。

智能体系统能否通过共享和选择跨任务经验来增强顺序推理？

为了解决这个问题，我们提出了CoPS（跨任务经验共享），一个理论基础的算法，通过跨任务经验共享和选择赋能智能体系统。CoPS通过在*离线*环境中利用完全外部的经验以及在*在线*环境中利用完全自我生成的经验，展示了其广泛的适用性。通过利用具有代表性的跨任务经验，CoPS使得智能体能够在新的复杂顺序推理任务上提升表现。我们的主要贡献总结如下：

+   •

    我们提出了CoPS，一种充分利用智能体跨任务经验，通过从先前任务轨迹中选择与分布匹配的经验来增强顺序推理的方法。我们方法的核心是一个基于悲观主义原则的经验选择策略，理论上该策略旨在最大化成功的、具有代表性的经验的效用，同时最小化由于分布变化（来自分布外样本）所带来的风险。值得注意的是，CoPS与智能体的基础模型、任务类型、经验来源和实现框架无关，使其易于使用，并且可以在各种环境中泛化。

+   •

    在实验上，我们在关键基准测试中验证了CoPS，如Alfworld（Shridhar等， [2020](https://arxiv.org/html/2410.16670v1#bib.bib37)），Webshop（Yao等， [2022a](https://arxiv.org/html/2410.16670v1#bib.bib54)），和HotPotQA（Yang等， [2018](https://arxiv.org/html/2410.16670v1#bib.bib53)）。CoPS consistently优于最先进的经验辅助推理方法，如RAP（Kagaya等， [2024](https://arxiv.org/html/2410.16670v1#bib.bib23)）和基于反思的推理方法，如Reflexion（Shinn等， [2024](https://arxiv.org/html/2410.16670v1#bib.bib36)）和LATS（Zhou等， [2023](https://arxiv.org/html/2410.16670v1#bib.bib61)）。此外，CoPS在样本效率上优于资源密集型方法，如LATS，使其在资源受限的场景中非常适用。这些结果展示了CoPS在实际应用中的有效性。

+   •

    在理论上，我们展示了在离线和在线设置中，我们基于悲观主义的算法的表现取决于预训练的大型语言模型（LLM）的质量，以及智能体选择的试验决定的跨任务经验分布与LLM所表示的任务依赖经验分布之间的匹配程度。我们的发现为设计高效的经验共享和选择算法提供了思路，并且为全面理解CoPS在不同场景中的有效性提供了依据。

符号表示 我们用 $[n]$ 表示集合 $\{1,\dots,n\}$。对于两个正序列 $\{a_{n}\}$ 和 $\{b_{n}\}$，其中 $n=1,2,\dots$，我们写作 $a_{n}=O(b_{n})$，如果存在一个常数 $C>0$ 使得对于所有 $n\geq 1$ 都有 $a_{n}\leq Cb_{n}$，并且写作 $a_{n}=\Omega(b_{n})$，如果存在一个常数 $C>0$ 使得对于所有 $n\geq 1$ 都有 $a_{n}\geq Cb_{n}$。我们使用 $\widetilde{O}(\cdot)$ 来进一步隐藏多对数因子。我们用 $(x_{i})_{i=1}^{n}$ 来表示序列 $(x_{1},...,x_{n})$，并且用 $\{x_{i}\}_{i=1}^{n}$ 来表示集合 $\{x_{1},...,x_{n}\}$。我们用 ${\text{D}_{\text{H}}}(p,q)=\sqrt{1/2\cdot\int(\sqrt{p}-\sqrt{q})^{2}}$ 来表示赫林距离。我们用 ${\text{D}_{\text{TV}}}(p,q)=1/2\cdot\int|p-q|$ 来表示全变差距离。我们用 $\chi^{2}(p,q)=\int p^{2}/q-1$ 来表示卡方距离。对于两个句子 $a$ 和 $b$，我们用 $a|b$ 来表示由 $a$ 和 $b$ 连接而成的句子。我们用 $\Delta(\mathcal{D})$ 来表示分布集 $\mathcal{D}$ 上的分布集合。

## 2 相关工作

### 2.1 基于LLM的智能体

最近几年，基于LLM的智能体研究显著增长（Chen等人，[2024c](https://arxiv.org/html/2410.16670v1#bib.bib6)，[b](https://arxiv.org/html/2410.16670v1#bib.bib5)；Chan等人，[2023](https://arxiv.org/html/2410.16670v1#bib.bib3)）。React（Yao等人，[2022b](https://arxiv.org/html/2410.16670v1#bib.bib56)）为后续的LLM智能体工作奠定了基础，特别是那些基于上下文学习（ICL）的智能体。与CoPS最相关的研究包括Shinn等人（[2024](https://arxiv.org/html/2410.16670v1#bib.bib36)）；Kagaya等人（[2024](https://arxiv.org/html/2410.16670v1#bib.bib23)）；Zhou等人（[2023](https://arxiv.org/html/2410.16670v1#bib.bib61)）；Raparthy等人（[2023](https://arxiv.org/html/2410.16670v1#bib.bib34)）。在Kagaya等人（[2024](https://arxiv.org/html/2410.16670v1#bib.bib23)）的研究中，提出了一种用于选择上下文示范的检索过程。然而，他们的方法在规划阶段依赖于频繁的嵌入查询，这导致了即使在较小的LLM设置中也出现了效率低下的问题。此外，RAP需要手动将智能体的规划轨迹拆分为多个阶段，并进行基准特定的调整，这显著增加了实现的复杂性并带来了可扩展性问题。Zhou等人（[2023](https://arxiv.org/html/2410.16670v1#bib.bib61)）提出了一种思维树（ToT）方法（Yao等人，[2024](https://arxiv.org/html/2410.16670v1#bib.bib55)），结合了反向传播和评估过程。然而，他们的方法显示出较差的样本效率，使其不太适用于在真实世界智能体设置中，试错机会有限的场景。同样，Liu等人（[2023a](https://arxiv.org/html/2410.16670v1#bib.bib29)）将基于价值的搜索集成到理论框架中，但面临类似的样本效率问题。Feng等人（[2024](https://arxiv.org/html/2410.16670v1#bib.bib11)）探讨了针对特定LLM智能体任务的微调方法，取得了良好的表现，但计算成本较高。最后，Raparthy等人（[2023](https://arxiv.org/html/2410.16670v1#bib.bib34)）利用高质量的经验作为ICL示范用于顺序推理。尽管取得了显著的表现，但这些经验来自外部RL系统，资源消耗较大，并且带来了可扩展性问题。

### 2.2 上下文示范选择

ICL中演示选择已被广泛研究。Wang等人（[2024b](https://arxiv.org/html/2410.16670v1#bib.bib46)）从贝叶斯角度研究了上下文中的演示选择，明确构建了选择过程的潜在变量。然而，他们的分析并没有考虑预训练知识分布，其结果主要是经验性的。Yan等人（[2023](https://arxiv.org/html/2410.16670v1#bib.bib51)）研究了上下文演示中的重复影响，进行了受控实验，以评估预训练知识中的重复如何影响结果。Scarlatos和Lan（[2023](https://arxiv.org/html/2410.16670v1#bib.bib35)）开发了一种强化学习框架来选择上下文示例，而Voronov等人（[2024](https://arxiv.org/html/2410.16670v1#bib.bib41)）研究了提示格式化对上下文学习表现的影响。此外，Shum等人（[2023](https://arxiv.org/html/2410.16670v1#bib.bib38)）为ICL示例数据集引入了一种自动CoT增强和选择方法。Hu等人（[2024b](https://arxiv.org/html/2410.16670v1#bib.bib17)）从理论角度分析了上下文演示的规模，推导出了通用统计界限，同时考虑了预训练误差。然而，他们的重点主要是CoT在一般ICL设置中的应用，而不是针对LLM代理与环境交互并需要反馈进行优化所面临的具体挑战。

### 2.3 代理理论

一些研究推动了对LLM代理理论理解的深入。He等人（[2024](https://arxiv.org/html/2410.16670v1#bib.bib14)）通过贝叶斯聚合模仿学习的视角探讨了LLM代理的统计理论。Lin等人（[2023](https://arxiv.org/html/2410.16670v1#bib.bib28)）提供了上下文强化学习中变换器的理论分析。Wang等人（[2024a](https://arxiv.org/html/2410.16670v1#bib.bib43)）研究了变换器在顺序推理中的训练与泛化，指出了变换器行为与在线学习算法之间的相似性。Sumers等人（[2023](https://arxiv.org/html/2410.16670v1#bib.bib39)）提供了LLM代理的认知视角，而Park等人（[2024](https://arxiv.org/html/2410.16670v1#bib.bib32)）研究了LLM代理在顺序推理任务中的后悔，提供了有助于CoPS发展的理论和实证见解。

## 3 方法论

### 3.1 初步

我们考虑一个序列决策场景，包含一个任务空间$\mathtt{M}$、一个状态空间$\mathtt{S}$和一个动作空间$\mathtt{A}$。状态$s\in\mathtt{S}$被定义为一个描述当前任务历史的句子。例如：“你在一个房间的中央，请找出一条通往苹果的路径。”动作$a\in\mathtt{A}$是对任务的解决方案，例如：“向右移动，苹果在桌子上。”智能体通过试验与环境进行交互。在每次试验的开始，从任务空间中随机抽取一个任务$\mathbf{M}$，$\mathbf{M}\sim\mathbb{P}^{\mathtt{M}}$。然后，智能体观察初始状态$s_{1}$，该状态从初始状态分布中采样，$s_{1}\sim\mathbb{P}^{\mathbf{M}}_{0}$。在每一步$h$，智能体根据当前状态$s_{h}$做出决策$a_{h}$，接下来的状态更新为$s_{h+1}=s_{h}|a_{h}$。智能体要么成功完成任务，要么继续生成动作，直到达到智能体与环境之间的最大交互次数$H$。我们将*经验*（$\tau$）定义为一次完整的试验，即$\tau=s_{h}$，其中$h\leq H$是当前试验的最终步骤。奖励$r(s_{h})$表示经验解决任务的有效性，$0\leq r(s_{h})\leq 1$。

在这项工作中，我们假设可以访问一个大型语言模型（LLM）来辅助决策制定。我们将LLM表示为${\text{LLM}}(a|\cdot)$，即给定输入序列的条件动作分布。

算法 1 CoPS：跨任务经验共享

0:  语言模型${\text{LLM}}(\cdot|\cdot)$，记忆库$\mathcal{D}=\{\tau_{1},\dots,\tau_{n}\}$，解码器${\mathtt{Dec}}$，距离度量$d$，记忆大小$k$，最大序列长度$H$。1:  接收初始状态$s_{1}$，通过解码器接收状态采样的经验$\tau^{s_{1}}$，$\tau^{s_{1}}\sim{\mathtt{Dec}}(\cdot|s_{1})$。2:  设置概率$\widehat{p}\in\Delta(\mathcal{D})$，如在([3.3](https://arxiv.org/html/2410.16670v1#S3.E3 "在3.2提出的方法 ‣ 3 方法论 ‣ CoPS: 授权LLM智能体进行可证明的跨任务经验共享"))中所示，近似最大化以下目标：

|  | $\displaystyle\widehat{p}=\mathop{\mathrm{argmax}}_{p\in\Delta(\mathcal{D})}% \mathbb{E}_{\tau\sim p}[r(\tau)-d(\tau,\tau^{s_{1}})].$ |  | (3.1) |
| --- | --- | --- | --- |

3:  反复检索试验$\tau^{1},\dots,\tau^{k}\sim\widehat{p}$。4:  将$\tau^{1},\dots,\tau^{k}$拼接成一个轨迹${\mathcal{T}}=\tau^{1}|\dots|\tau^{k}$，设定$h\leftarrow 1$。5:  当未成功且$h<H$时，执行以下操作：6:     获得动作$a_{h}\sim{\text{LLM}}(\cdot|{\mathcal{T}},s_{h})$，设定$s_{h+1}\leftarrow s_{h}|a_{h}$，$h\leftarrow h+1$。7:  结束当循环

### 3.2 提出的方法

我们提出了基于分布匹配的 CoPS 方法。CoPS 基于逐次试验的方式操作，适用于*离线设置*，即代理可以访问一个包含经验的外部静态数据集，也适用于*在线设置*，即代理通过与环境交互收集经验。假设我们的代理处于一次试验的起始状态 $s_{1}\sim\mathbb{P}_{0}^{\mathbf{M}}$。我们将介绍 CoPS 的关键组件，如下所示。

Memory Bank 代理可以访问一个包含经验的记忆库 $\mathcal{D}$，这些经验可以来自预先收集的数据集（离线）或来自先前的经验（在线）。我们不对 $\mathcal{D}$ 施加限制，这意味着 $\mathcal{D}$ 中的经验具有高度多样性。具体来说，经验 $\tau\in\mathcal{D}$ 可能对应不同的任务 $\mathbf{M}$，或对应同一任务的不同解决策略。我们的目标是开发一种策略，从 $\mathcal{D}$ 中检索经验，帮助当前任务的决策过程。

Cross-Task Experience Sharing CoPS 利用一个外部模块，称为*解码器*，在第[1行](https://arxiv.org/html/2410.16670v1#alg1.l1 "在算法 1 ‣ 3.1 初步 ‣ 3 方法 ‣ CoPS：赋能LLM代理通过可证明的跨任务经验共享")中表示为 ${\mathtt{Dec}}$。通常，解码器输出一个依赖于任务的经验分布，该分布基于初始状态 $s_{1}$，反映了LLM如何在没有明确指令的情况下解决与 $s_{1}$ 相关的任务 $\mathbf{M}$。借助解码器，代理的目标是找到一个经验的概率分布 $\widehat{p}$，该分布在所有经验集合 $\mathcal{D}$ 中满足以下条件：

|  | $\displaystyle\widehat{p}=\mathop{\mathrm{argmax}}_{p\in\Delta(\mathcal{D})}% \mathbb{E}_{\tau\sim p}[r(\tau)]-d(p,{\mathtt{Dec}}(\cdot&#124;s_{1})),$ |  | (3.2) |
| --- | --- | --- | --- |

其中 $d$ 是一个分布度量。直观地，[（3.2](https://arxiv.org/html/2410.16670v1#S3.E2 "在 3.2 提出的 方法 ‣ 3 方法 ‣ CoPS：赋能LLM代理通过可证明的跨任务经验共享")](https://arxiv.org/html/2410.16670v1#S3.E2) 类似于*悲观主义原则*，该原则通常在离线强化学习文献中使用（Jin 等，[2021](https://arxiv.org/html/2410.16670v1#bib.bib22)）。$\widehat{p}$ 的目标是最大化期望奖励，同时保持分布与 ${\mathtt{Dec}}$ 解码的分布接近。重要的是，$\widehat{p}$ 支持跨任务设置，因为它不限制其支持仅限于与 $s_{1}$ 相同任务的经验。在给定的上下文记忆大小 $k$ 下，CoPS 会反复从 $\widehat{p}$ 中采样经验 $\tau^{1},\dots,\tau^{k}$，如第[3行](https://arxiv.org/html/2410.16670v1#alg1.l3 "在算法 1 ‣ 3.1 初步 ‣ 3 方法 ‣ CoPS：赋能LLM代理通过可证明的跨任务经验共享")所示。

执行规划 令$\mathcal{T}=\tau^{1}|\dots|\tau^{k}$表示包含$\tau^{1},\dots,\tau^{k}$的*经验集合*。从初始状态$s_{1}$开始，智能体逐步执行动作，其中每个动作$a_{h}$是从LLM的分布中抽取的，条件是经验集合和当前状态：

|  | $a_{h}\sim{\text{LLM}}(\cdot&#124;{\mathcal{T}},s_{h}).$ |  |
| --- | --- | --- |

在在线环境中，完成一次试验后，智能体通过将新经验添加到记忆库$\mathcal{D}$中来更新记忆库，以供未来使用。

实现细节 在这里我们讨论CoPS的一些实现细节。*首先*，在实际操作中，直接计算式（[3.2](https://arxiv.org/html/2410.16670v1#S3.E2 "在3.2节提出的方法 ‣ 3 方法论 ‣ CoPS: 赋能LLM智能体通过可验证的跨任务经验共享")）中分布之间的距离$d(p,{\mathtt{Dec}}(\cdot|s_{1}))$是计算上不可行的。因此，我们使用经验逼近法，将分布之间的距离转换为从这些分布中抽取的经验之间的距离，如式（[3.1](https://arxiv.org/html/2410.16670v1#S3.E1 "在2节 ‣ 算法1 ‣ 3.1 初步 ‣ 3 方法论 ‣ CoPS: 赋能LLM智能体通过可验证的跨任务经验共享")）所示。*第二*，我们指定了${\mathtt{Dec}}$的选择。解码器输出一个来自$\mathcal{D}$的经验$\tau^{s_{1}}$，其从相同的初始状态$s_{1}$开始。如果存在多个这样的经验，我们选择最新的一个。这个$\tau^{s_{1}}$自然地反映了LLM在没有干预的情况下，从$s_{1}$开始解决任务的行为。*第三*，我们讨论了如何近似求解式（[3.1](https://arxiv.org/html/2410.16670v1#S3.E1 "在2节 ‣ 算法1 ‣ 3.1 初步 ‣ 3 方法论 ‣ CoPS: 赋能LLM智能体通过可验证的跨任务经验共享")），因为枚举所有可能的分布在$\Delta(\mathcal{D})$中是计算上低效的。具体来说，我们定义了距离函数$d$并近似求解$\widehat{p}$，如下所示：

|  | $d(\tau,\tau^{\prime}):=c\cdot\text{cos}(e(\tau),e(\tau^{\prime})),\ \widehat{p% }(\tau)\propto r(\tau)\cdot\exp(-d(\tau,\tau^{s_{1}})),$ |  | (3.3) |
| --- | --- | --- | --- |

其中 $c\geq 0$ 是一个超参数，"cos"表示余弦函数，$e$是一个嵌入函数，它将语言句子映射到高维欧几里得空间。在实际应用中，我们使用$e$作为语言嵌入模型（例如，gte-Qwen2 7b（Li等，[2023b](https://arxiv.org/html/2410.16670v1#bib.bib27)））。该方法倾向于从$\mathcal{D}$中选择成功的经验，选择的概率与当前初始状态$s_{1}$的距离的倒数成正比。公式([3.3](https://arxiv.org/html/2410.16670v1#S3.E3 "In 3.2 Proposed Method ‣ 3 Methodology ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing"))中的超参数$c$控制相对距离的影响：当$c=0$时，方法会均匀地从$\mathcal{D}$中抽取成功的经验，而当$c\to\infty$时，它会确定性地选择最接近$\tau^{s_{1}}$的经验。

## 4 实验

在本节中，我们展示了一个实验研究，评估了CoPS在真实世界LLM上的实际表现，特别是Llama 3.1模型（Dubey等，[2024](https://arxiv.org/html/2410.16670v1#bib.bib9)）。我们的结果表明，CoPS在任务成功率和样本效率方面都达到了最先进（SOTA）的表现，超越了现有基准。我们所知的最佳效果已被超越。关于我们提示公式的详细描述，见附录[C](https://arxiv.org/html/2410.16670v1#A3 "Appendix C Prompt Template ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing")。值得注意的是，CoPS不仅易于实现，而且具有跨不同环境的可泛化性：对于每次试验，所选经验直接添加到提示中，无需手动修改。

这种提示策略具有两个明显的优势：首先，通过结合跨任务的经验，它显著提升了顺序推理的表现，超越了反思驱动的方法，如Reflexion。其次，跨试验的提示共享一个重要的前缀，这最大化了现代LLM服务系统中前缀缓存机制的效果（Zheng等，[2023](https://arxiv.org/html/2410.16670v1#bib.bib60)），因此相比于RAP（Kagaya等，[2024](https://arxiv.org/html/2410.16670v1#bib.bib23)），在效率上有了显著的提升。

基准测试 我们在三个具有代表性的基准上评估我们的算法：Alfworld（Shridhar et al., [2020](https://arxiv.org/html/2410.16670v1#bib.bib37)）、Webshop（Yao et al., [2022a](https://arxiv.org/html/2410.16670v1#bib.bib54)）和HotPotQA（Yang et al., [2018](https://arxiv.org/html/2410.16670v1#bib.bib53)）。在这些基准中，代理需要在有限的尝试次数内解决问题，从而实现跨试验和跨任务的经验共享。在Alfworld中，代理在一个模拟的家庭环境中接收到特定的任务描述，通过预定义的动作进行互动，并以文本描述的形式接收反馈。在Webshop中，代理必须从超过百万个项目的目录中找到与用户规格匹配的产品，在每次尝试时与HTML页面和搜索引擎进行互动，同时接收有限的产品信息。在HotPotQA中，代理回答需要特定知识的复杂问题，利用Wikipedia检索相关的文章。在所有基准测试中，奖励函数$r(\tau)$被定义为当代理成功完成任务时为$1$，否则为$0$。

LLM选择 我们在整个实验中使用广泛应用的Llama 3.1系列模型（Dubey et al., [2024](https://arxiv.org/html/2410.16670v1#bib.bib9)），考虑到它们在基准测试中的优异表现以及开放权重LLM生态系统的可持续性¹¹1注意，这些是一些具有代表性和流行的开源模型（Wang et al., [2024c](https://arxiv.org/html/2410.16670v1#bib.bib47); He et al., [2023](https://arxiv.org/html/2410.16670v1#bib.bib15); Xiao et al., [2024](https://arxiv.org/html/2410.16670v1#bib.bib49); Hu et al., [2024a](https://arxiv.org/html/2410.16670v1#bib.bib16)）可以被考虑。由于资源限制，我们选择了实验时最新的模型。具体来说，我们的实验是在NVIDIA A6000和A100 GPU上使用Llama 3.1 8b Instruct和Llama 3.1 70b Instruct进行的。我们使用gte-Qwen2 7b Instruct（Li et al., [2023b](https://arxiv.org/html/2410.16670v1#bib.bib27)）作为我们的嵌入模型。我们使用SGLang（Zheng et al., [2023](https://arxiv.org/html/2410.16670v1#bib.bib60)）作为我们的LLM服务引擎，因为它在服务性能和前缀缓存机制上具有SOTA表现。

基准模型 我们将 CoPS 与三种具有代表性的基准模型进行比较：Reflexion（Shinn 等，[2024](https://arxiv.org/html/2410.16670v1#bib.bib36)），RAP（Kagaya 等，[2024](https://arxiv.org/html/2410.16670v1#bib.bib23)），以及 LATS（Zhou 等，[2023](https://arxiv.org/html/2410.16670v1#bib.bib61)）。在 Reflexion 中，代理在每个环境中尝试多次完成任务，直到成功。每次失败后，LLM 代理会反思其失败的轨迹并将反思保存到其记忆中。对于每一次后续的尝试，代理会获得最多三个来自同一任务的最新反思。在 RAP 中，在每次试验的每个阶段，代理会展示来自轨迹片段的前 $k$ 个搜索结果作为上下文示范。在 LATS 中，代理利用树状结构搜索在每次试验中探索多种推理和行动理由。当遇到失败的理由时，代理会生成对其错误的反思，并将这些见解融入其未来试验的决策过程中。

### 4.1 结果与分析

在本节中，我们展示了 CoPS 在所有基准测试和模型大小上均优于所有基准模型，考虑了样本效率和任务成功率。多个试验的详细性能展示见于图 LABEL:fig:allresult。我们的超参数细节见于附录[B](https://arxiv.org/html/2410.16670v1#A2 "附录 B 更多实验细节 ‣ CoPS：赋能 LLM 代理具有可证明的跨任务经验共享")中的表[5](https://arxiv.org/html/2410.16670v1#A2.T5 "表 5 ‣ 附录 B 更多实验细节 ‣ CoPS：赋能 LLM 代理具有可证明的跨任务经验共享")。

表 1：在使用 Llama3.1 8b 和 70b 模型的 Alfworld 基准测试中，Reflexion、RAP 和 CoPS 的性能比较。

| 算法 | 性能 |
| --- | --- |
| Llama3.1 8b | Llama3.1 70b |
| Reflexion ²²2 Reflexion 的原始代码库在使用较小的 Llama3.1 8b 模型时，在大多数任务上表现不佳。这主要是因为该模型倾向于重复相同的动作，导致任务失败。为了缓解这一问题，我们引入了重采样机制，以提高 Reflexion 的性能，当模型开始重复动作时，该机制会被激活。这一修改显著提升了 Reflexion 的表现。 | 86 | 94 |
| RAP | 70 | 93 |
| CoPS | 94 | 100 |

Alfworld基准表[2](https://arxiv.org/html/2410.16670v1#S4.T2 "表2 ‣ 4.1 结果与分析 ‣ 4 实验 ‣ CoPS：通过可证明的跨任务经验共享赋能LLM代理")和图表LABEL:alf:8b、LABEL:alf:70b展示了在Alfworld基准上CoPS、Reflexion和RAP的比较。这些数值表示在134个任务上经过10次试验后的成功率。当使用较小的Llama 3.1 8b模型时，CoPS达到了94%的成功率，显著超过了Reflexion（86%）和RAP（70%）。这一结果尤为值得注意，因为Reflexion需要使用更大的Llama 3.1 70b模型才能达到类似的表现，这突显了CoPS的卓越有效性。这表明，CoPS即使在计算资源有限和模型能力较弱的情况下，也能达到最先进的性能，明显优于其他算法。此外，当扩展到更大的Llama 3.1 70b模型时，CoPS达到了完美的100%成功率。这些结果强调了CoPS的可扩展性，在不同模型大小下始终优于基准。尽管RAP也利用了上下文演示检索机制，但它缺乏有效的经验选择算法，因此明显低于CoPS。此外，需要注意的是，RAP在每次试验中手动将代理的规划轨迹分为多个阶段，而这些拆分方法是针对每个基准特定的，必须手动调整。这大大增加了实现复杂度并引入了可扩展性问题。相比之下，CoPS通过直接将成功经验放入提示中，高效地重用这些经验，而无需针对特定基准进行修改，使其成为一种更实用和灵活的解决方案。因此，CoPS不仅在性能上超越了基准，还通过消除手动干预的需求，提供了开箱即用的可用性。

表2：使用Llama3.1 8b和70b模型在Webshop基准上对Reflexion、RAP、LATS和CoPS的性能比较。

| 算法 | 性能 |
| --- | --- |
| Llama3.1 8b | Llama3.1 70b |
| Reflexion | 30 | 30 |
| RAP | 42 | 42 |
| LATS | 24 | 32 |
| CoPS | 50 | 56 |

Webshop基准³³3我们观察到，在Webshop基准上对Reflexion和RAP模型进行规模扩展并未带来显著的性能提升。这个观察结果与Reflexion（Shinn等人，[2024](https://arxiv.org/html/2410.16670v1#bib.bib36)，附录B.1）和RAP（Kagaya等人，[2024](https://arxiv.org/html/2410.16670v1#bib.bib23)，表2）的原始研究结果一致，这些研究表明，这些模型往往会收敛到局部最小值，需要极具创造力的策略才能克服。表[2](https://arxiv.org/html/2410.16670v1#S4.T2 "Table 2 ‣ 4.1 Results and Analysis ‣ 4 Experiments ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing")和图 LABEL:web:8b, LABEL:web:70b 比较了CoPS与Webshop基准上所有基准算法的性能，衡量标准为成功率。数值表示50个产品的成功率，每个算法通过每个产品进行10次试验。对于较小的Llama 3.1 8b模型，CoPS的成功率为50%，比下一个最好的竞争者RAP高出8%的绝对改善。当扩展到更大的Llama 3.1 70b模型时，CoPS的性能提升更加明显，成功率达到了56%。这比RAP有14%的绝对提升。

为了确保与基准算法的公平比较，我们通过减少搜索树的宽度并限制轨迹迭代次数来修改LATS基准。这一调整确保了每个基准算法的运行时间大致相同。即使做出这些调整，LATS的样本效率仍显著较低。具体而言，Llama 3.1 8b在LATS中生成的总标记数量（1,555,365个标记）几乎是CoPS（314,336个标记）的五倍。更多细节可以在表[3](https://arxiv.org/html/2410.16670v1#S4.T3 "Table 3 ‣ 4.1 Results and Analysis ‣ 4 Experiments ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing")中找到。这一标记使用差异凸显了当前基于搜索树的算法的低效性。相比之下，CoPS在相同推理约束下展现了更好的效率和性能。

表3：Webshop实验中每个模型的标记生成数量。值得注意的是，对于每个模型，LATS的标记生成数量至少是CoPS的5倍。

| 算法 | Reflexion | RAP | LATS | CoPS |
| --- | --- | --- | --- | --- |
| Llama3.1 8b | 159131 | 107504 | 1555365 | 314336 |
| Llama3.1 70b | 125406 | 109245 | 1058752 | 113849 |

表4：在HotPotQA基准上使用Llama3.1 8b和70b模型对Reflexion、LATS和CoPS的性能比较。

| 算法 | 性能 |
| --- | --- |
| Llama3.1 8b | Llama3.1 70b |
| Reflexion | 56 | 61 |
| LATS | 55 | 64 |
| CoPS | 63 | 65 |

HotPotQA 基准表 [4](https://arxiv.org/html/2410.16670v1#S4.T4 "表 4 ‣ 4.1 结果与分析 ‣ 4 实验 ‣ CoPS: 通过可证明的跨任务经验共享赋能 LLM 代理") 和图表 LABEL:hot:8b, LABEL:hot:70b 展示了在 100 个问答（QA）任务上，CoPS、Reflexion 和 LATS 在 HotPotQA 基准上的对比。表中的数值表示成功率，每个算法在 10 次试验中进行了测试。结果表明，CoPS 始终在所有模型规模中相较于 Reflexion 和 LATS 展现出优越的性能。CoPS 的优势在使用较小的 Llama 3.1 8b 模型时尤为显著，其中 CoPS 实现了 63% 的成功率，相比 Reflexion 和 LATS 分别有 7% 和 8% 的显著绝对提升。此外，即使在扩展到更大的 Llama 3.1 70b 模型时，CoPS 依然表现出更强的性能。在这种设置下，CoPS 达到了 65% 的成功率，超越 Reflexion 4% 和 LATS 1%。值得注意的是，当模型从较小到较大切换时，Reflexion 和 LATS 的基准展示了显著的性能差距，而 CoPS 的结果相对一致，且在不同模型规模中始终保持性能优势。这表明，CoPS 的原则化跨任务经验共享机制在需要复杂推理的任务中也表现出色。

结论 我们在 Alfworld、Webshop 和 HotPotQA 上的实验表明，CoPS 在任务成功率和样本效率上始终优于现有的最先进基准。特别是，CoPS 即使在较小的模型如 Llama 3.1 8b 上也能实现更优的性能，突出其在资源受限场景下的高效性和实用性。这些结果验证了通过我们理论支持的选择策略利用原则化的跨任务经验共享的有效性，确认了 CoPS 在不同任务和模型规模中增强了顺序推理能力。

### 4.2 消融研究

本节中，我们分析了两个关键超参数如何影响 CoPS 的性能：方程式 ([3.3](https://arxiv.org/html/2410.16670v1#S3.E3 "在 3.2 提出的方法 ‣ 3 方法论 ‣ CoPS: 通过可证明的跨任务经验共享赋能 LLM 代理")) 中的缩放因子 $c$ 和放置在提示开头的上下文经验数 $k$。我们在 Alfworld 基准上使用 Llama 3.1 8b 和 Llama 3.1 70b 模型进行了实验。

对于缩放因子$c$，我们测试了四个设置：$c=0$、$1$、$5$和$10$，同时保持上下文经验的数量固定为$k=5$（见图 LABEL:ablc:8b 和 LABEL:ablc:70b）。我们的发现表明，对于像Llama 3.1 8b这样的较小模型，较小但非零的$c$值（例如$c=1$）通常能带来更好的性能（见图 LABEL:ablc:8b）。这表明，适度的缩放有效地平衡了模型的适应性和在较低性能模型上的鲁棒性。

关于上下文经验的数量$k$，我们评估了从$1$到$10$的值，并设置$c=0$（见图 LABEL:ablk:8b 和 LABEL:ablk:70b）。我们观察到，性能随着$k$的增加而提升，直到$k=3$，之后无论模型大小如何，性能趋于平稳。这个结果表明，虽然增加上下文经验的数量在某个点上会提升性能，但添加超过三个经验可能不会带来显著的增益。

我们的消融研究表明，调整CoPS中的关键超参数对于获得最佳性能至关重要。具体而言，对于较小的模型，一个小但非零的缩放因子$c$（例如$c=1$）能够有效平衡适应性和鲁棒性。此外，增加上下文经验的数量$k$能够提高性能，直到$k=3$，超过这个值后，额外的经验带来的增益非常有限。这些见解为超参数选择提供了实际指导，确保CoPS能够在各种设置中高效部署，以最大化其顺序推理能力。

## 5 经验辅助代理的理论框架

在本节中，我们构建了理论框架以展示CoPS的有效性。为了简便起见，我们在一个带有赌博者设置的环境中分析我们的算法，其中每个经验的最大步骤数为$H=1$。与第[3](https://arxiv.org/html/2410.16670v1#S3 "3 Methodology ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing")节的表述略有不同，我们将经验定义为$\tau=s|a|r$，其中包括初始状态$s$、动作$a$及其奖励$r=r(s,a)$。

为了使我们的分析更加清晰，我们引入了额外的符号。令 ${\mathcal{T}}=\tau_{1}|\tau_{2}|\dots$ 表示经验集合。${\mathcal{T}}$ 的长度记作 $|{\mathcal{T}}|$，即 ${\mathcal{T}}=(\tau_{1},...,\tau_{|{\mathcal{T}}|})$。我们使用 ${\mathcal{T}}_{t}$ 来表示经验集合的前 $t$ 步，即 ${\mathcal{T}}_{t}=\tau_{1}|\dots|\tau_{t}$。对于任何经验集合 ${\mathcal{T}}$，我们假设 $|{\mathcal{T}}|\leq T$。我们定义 $\mathtt{T}$ 为所有轨迹的空间，$\mathtt{T}_{t}$ 为长度为 $t$ 的轨迹空间。我们将一般算法表示为 $\text{Alg}(\cdot|\cdot,\cdot,\cdot):\mathtt{M}\times\mathtt{T}\times\mathtt{S}% \rightarrow\Delta(\mathtt{A})$，它的输入包括任务 $\mathbf{M}\in\mathtt{M}$、经验集合 ${\mathcal{T}}\in\mathtt{T}$ 和状态 $s\in\mathtt{S}$，输出一个关于动作 $a\in\mathtt{A}$ 的分布。注意，有些算法可能不使用任务 $\mathbf{M}$ 作为输入，这种情况下我们写作 $\text{Alg}(\cdot|\cdot,\cdot)$。我们用 $\mathbb{P}^{\mathbf{M},\text{Alg}}_{t}$ 表示在任务 $\mathbf{M}$ 和算法 Alg 下，经验集合前 $t$ 步的分布。对于一个将 $\mathbf{M},{\mathcal{T}},s$ 作为输入的算法 Alg，我们将其*后验平均*定义为 $\overline{\text{Alg}}(\cdot|{\mathcal{T}},s)=\mathbb{E}_{\mathbf{M}\sim\mathbb% {P}^{\mathtt{M}}(\cdot|{\mathcal{T}}^{\prime}={\mathcal{T}},s^{\prime}=s)}[% \text{Alg}(\cdot|\mathbf{M},{\mathcal{T}}^{\prime},s^{\prime})]$，这是给定经验集合 ${\mathcal{T}}$ 和当前状态 $s$ 时，算法 Alg 的最佳贝叶斯近似。

### 5.1 LLM 预训练

我们首先描述 LLM 的预训练过程。令 ${\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot|{\mathcal{T}},s):\mathtt{T}\times\mathtt{S}\rightarrow\Delta(\mathtt{A})$ 表示一个 LLM 代理，它输出一个关于 $\mathtt{A}$ 的分布，其中 $\widehat{\bm{\theta}}\in\bm{\Theta}$ 是 LLM 的参数，$\bm{\Theta}$ 表示整个参数空间。我们假设存在一个预训练数据集 $\mathcal{D}_{\text{pre}}=\{{\mathcal{T}}^{1},\dots,{\mathcal{T}}^{{n_{\text{% pre}}}}\}$，其中 $|{\mathcal{T}}^{i}|=T-1$。根据 Lin 等人（[2023](https://arxiv.org/html/2410.16670v1#bib.bib28)）中的预训练设置，我们假设有两个算法：一个 *上下文算法*，${\text{Alg}^{C}}(\cdot|\cdot,\cdot):\mathtt{T}\times\mathtt{S}\rightarrow\Delta(\mathtt{A})$，和一个 *专家算法*，${\text{Alg}^{E}}(\cdot|\cdot,\cdot,\cdot):\mathtt{M}\times\mathtt{T}\times\mathtt{S}\rightarrow\Delta(\mathtt{A})$。通常，上下文算法基于经验集合和当前状态提供一个“自然”的动作，而专家算法则根据任务信息、经验集合和当前状态提供一个更有依据的动作。由于专家算法可以访问任务信息 $\mathbf{M}$，因此它通常会比上下文算法生成更好的动作。

我们现在描述预训练过程。为了生成一个经验集合 ${\mathcal{T}}=\tau_{1}|\dots|\tau_{T-1}\in\mathcal{D}_{\text{pre}}$，我们首先从任务分布 $\mathbf{M}\sim\mathbb{P}^{\mathtt{M}}$ 中抽样一个任务。对于每个经验 $\tau_{i}$，状态从初始状态分布 $s_{i}\sim\mathbb{P}^{\mathbf{M}}_{0}$ 中抽样，动作则使用上下文算法抽样 $a_{i}\sim{\text{Alg}^{C}}(\cdot|{\mathcal{T}}_{i-1},s_{i})$，奖励由 $r_{i}=r(s_{i},a_{i})$ 给出。在生成经验集合后，我们使用专家算法为每个 ${\mathcal{T}}$ 的步骤收集专家反馈 $\bar{a}_{1},\dots,\bar{a}_{T-1}$，其中 $\bar{a}_{i}\sim{\text{Alg}^{E}}(\cdot|\mathbf{M},{\mathcal{T}}_{i-1},s_{i})$。重复这一过程 ${n_{\text{pre}}}$ 次，生成轨迹 ${\mathcal{T}}^{i}$ 和专家动作 $\bar{a}_{1}^{i},\dots,\bar{a}_{T-1}^{i}$，其中 $i\in[{n_{\text{pre}}}]$。最后，我们通过求解以下最大似然估计问题来预训练 LLM ${\text{Alg}_{\widehat{\bm{\theta}}}}$：

|  | $\displaystyle\widehat{\bm{\theta}}\leftarrow\mathop{\mathrm{argmax}}_{\bm{\theta}\in\bm{\Theta}}\sum_{i=1}^{{n_{\text{pre}}}}\sum_{t=1}^{T}\log\text{Alg}_{\bm{\theta}}(\bar{a}_{t}^{i}&#124;{\mathcal{T}}^{i}_{t-1},s_{t}^{i}).$ |  |
| --- | --- | --- |

在本文的其余部分，我们使用 ${\text{Alg}_{\widehat{\bm{\theta}}}}$ 来表示我们的 LLM。下面，我们将呈现一些用于分析 ${\text{Alg}_{\widehat{\bm{\theta}}}}$ 的标准假设。

###### 定义 5.1（Lin 等人 [2023](https://arxiv.org/html/2410.16670v1#bib.bib28)）。

设 $\bm{\Theta}$ 为大语言模型（LLM）$\text{Alg}_{\bm{\theta}}$ 的参数集合。我们称 $\bm{\Theta}_{0}\subseteq\bm{\Theta}$ 为相对于 $\text{Alg}_{\bm{\theta}}$ 的 $\rho$-覆盖集，如果对于任何 $\bm{\theta}\in\bm{\Theta}$，存在 $\bm{\theta}_{0}\in\bm{\Theta}_{0}$ 使得

|  | $\displaystyle\forall s\in\mathtt{S},t\in[T],{\mathcal{T}}\in\mathtt{T}_{t-1},% \&#124;\log\text{Alg}_{\bm{\theta}}(\cdot&#124;{\mathcal{T}},s)-\log\text{Alg}_{\bm{% \theta}_{0}}(\cdot&#124;{\mathcal{T}},s)\&#124;_{\infty}\leq\rho.$ |  |
| --- | --- | --- |

我们用 $\mathcal{N}(\rho)=|\bm{\Theta}_{0}|$ 表示 $\text{Alg}_{\bm{\theta}}$ 的 $\rho$-覆盖数。

下一假设假定，训练好的大语言模型（LLM）与专家算法后验平均值 $\overline{{\text{Alg}^{E}}}$ 之间的最佳近似值可以被某个常数所界定。

###### 假设 5.2  (Lin et al. [2023](https://arxiv.org/html/2410.16670v1#bib.bib28))。

存在 $\bm{\theta}^{*}\in\bm{\Theta}$ 和一个*模型容量误差* ${\epsilon_{\text{real}}}>0$，使得

|  | $\forall t\in[T],\log\mathbb{E}_{\mathbf{M}\sim\mathbb{P}^{\mathtt{M}},s\sim% \mathbb{P}_{0}^{\mathbf{M}},{\mathcal{T}}\sim\mathbb{P}_{t-1}^{\mathbf{M},{% \text{Alg}^{C}}},\bar{a}\sim\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}},s)% }\bigg{[}\frac{\overline{{\text{Alg}^{E}}}(\bar{a}&#124;{\mathcal{T}},s)}{\text{Alg% }_{\bm{\theta}^{*}}(\bar{a}&#124;{\mathcal{T}},s)}\bigg{]}\leq{\epsilon_{\text{real% }}}.$ |  |
| --- | --- | --- |

最后，我们对算法 [1](https://arxiv.org/html/2410.16670v1#alg1 "Algorithm 1 ‣ 3.1 Preliminary ‣ 3 Methodology ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing") 中引入的解码器 ${\mathtt{Dec}}$ 做出假设。我们假设访问一个解码器类 ${\mathtt{Dec}}_{t}:\mathtt{S}\rightarrow\Delta(\mathtt{T}_{t})$，该解码器将状态 $s$ 映射到一个关于 $t$ 次经验的分布，并能够估算分布 $\mathbb{P}^{\mathbf{M},{\text{Alg}^{C}}}_{t}({\mathcal{T}})$，该分布表示由大语言模型提供的任务相关经验分布。

###### 假设 5.3。

对于解码器 ${\mathtt{Dec}}_{t}:\mathtt{S}\rightarrow\Delta(\mathtt{T}_{t})$，存在一个*解码器系数* $C_{{\mathtt{Dec}}}>1$，使得对于任何 $t\in[T]$、${\mathcal{T}}\in\mathtt{T}_{t-1}$、$\mathbf{M}\in\mathtt{M}$ 以及 $s\sim\mathbb{P}_{0}^{\mathbf{M}}$，我们有

|  | $\frac{1}{C_{{\mathtt{Dec}}}^{2}}\leq\frac{{\mathtt{Dec}}_{t-1}({\mathcal{T}}&#124;s% )}{\mathbb{P}^{\mathbf{M},{\text{Alg}^{C}}}_{t-1}({\mathcal{T}})}\leq C_{{% \mathtt{Dec}}}^{2}.$ |  |
| --- | --- | --- |

### 5.2 离线设置的算法

我们考虑与第[3](https://arxiv.org/html/2410.16670v1#S3 "3 Methodology ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing")节相同的离线设置。假设我们有一个离线数据集$\mathcal{D}$，并且代理被赋予初始状态$s$。我们将经验选择问题形式化为分布选择问题，其中代理可以访问一个候选分布集，记作$\mathcal{P}=\{\mathbb{P}^{1}(\cdot|\cdot,\cdot),\dots,\mathbb{P}^{|\mathcal{P}% |}(\cdot|\cdot,\cdot)\}\subseteq 2^{\mathtt{T}_{T-1}\times\mathtt{S}% \rightarrow\Delta(\mathtt{T}_{T-1})}$。该集合中的每个元素表示从数据集$\mathcal{D}$和当前状态$s$到轨迹$\mathcal{T}$（长度为$T-1$）上的分布的映射。一般来说，每个$\mathbb{P}^{i}$可以解释为数据集$\mathcal{D}$中所有可能的$T-1$次经验组合的分布。代理的目标是从$\mathcal{P}$中选择一个分布${\widehat{\mathbb{P}}^{s}}$，使得次优性差距最小化，次优性差距量化了最佳策略与代理选择的策略之间的性能差异，具体由专家算法衡量：

|  | $\displaystyle\text{SubOpt}({\widehat{\mathbb{P}}^{s}}):=\mathbb{E}_{\mathbf{M}% \sim\mathbb{P}^{\mathtt{M}},s\sim\mathbb{P}_{0}^{\mathbf{M}}}\bigg{[}\max_{{% \widehat{\mathbb{P}}}\in\mathcal{P}}\mathbb{E}_{{\mathcal{T}}\sim{\widehat{% \mathbb{P}}},a\sim\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}},s)}r(s,a)-% \mathbb{E}_{{\mathcal{T}}\sim{\widehat{\mathbb{P}}^{s}},a\sim\overline{{\text{% Alg}^{E}}}(\cdot&#124;{\mathcal{T}},s)}r(s,a)\bigg{]}.$ |  | (5.1) |
| --- | --- | --- | --- |

我们在算法 [2](https://arxiv.org/html/2410.16670v1#alg2 "Algorithm 2 ‣ 5.2 Algorithms for the Offline Setting ‣ 5 Theoretical Framework of Experience-Assisted Agents ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing") 中提出了OfflineCoPS，这是一个基于经验集合的 CoPS 版本。OfflineCoPS 的核心思想与 CoPS 相似：智能体试图找到能够最大化奖励，同时最小化当前任务经验集合的分布偏移的经验集合，当前任务的经验集合由 LLM 表示。给定测试状态 $s$，OfflineCoPS 首先运行解码器来获取一个分布 ${\mathtt{Dec}}_{T-1}(\cdot|s)$，该分布近似于 $\mathbb{P}^{\mathcal{M},{\text{Alg}^{C}}}_{t-1}$。然后，OfflineCoPS 应用 *悲观原则*，如 ([3.2](https://arxiv.org/html/2410.16670v1#S3.E2 "In 3.2 Proposed Method ‣ 3 Methodology ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing")) 中所述。所选的分布 $\mathbb{P}^{*}\in\mathcal{P}$ 旨在识别一个分布，该分布生成一个经验集合，使得在给定 LLM 提供的动作下最大化奖励，同时保持接近解码分布 ${\mathtt{Dec}}_{T-1}(\cdot|s)$。为了衡量分布之间的距离，我们使用 $\chi^{2}$-距离。与 CoPS 中的超参数 $c$ 类似，OfflineCoPS 引入了一个超参数 $\epsilon_{\text{pre}}$ 来平衡最大化奖励和满足 ${\mathtt{Dec}}_{T-1}(\cdot|s)$ 所施加的规则条件之间的权衡。

算法 2 OfflineCoPS

0: LLM ${\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot|\cdot,\cdot)$，候选经验集合分布 $\mathcal{P}$，预训练误差参数 ${\epsilon_{\text{pretrain}}}$，任务解码器 ${\mathtt{Dec}}$，离线数据集 $\mathcal{D}$。1: 接收测试状态 $s$，解码分布 ${\mathtt{Dec}}_{T-1}(\cdot|s)$。2: 从 $\mathcal{P}$ 中选择 ${\widehat{\mathbb{P}}^{s}}$，以最大化以下内容：

|  | $\displaystyle{\widehat{\mathbb{P}}^{s}}=\mathop{\mathrm{argmax}}_{{\widehat{\mathbb{P}}}\in\mathcal{P}}\mathbb{E}_{\begin{subarray}{c}{\mathcal{T}}\sim{\widehat{\mathbb{P}}}(\cdot&#124;\mathcal{D},s),\\ a\sim{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{\mathcal{T}},s)\end{subarray}% }r(s,a)-{\epsilon_{\text{pretrain}}}\sqrt{1+\chi^{2}({\widehat{\mathbb{P}}}(% \cdot&#124;\mathcal{D},s),{\mathtt{Dec}}_{T-1}(\cdot&#124;s))}.$ |  | (5.2) |
| --- | --- | --- | --- |

3: 生成 ${\mathcal{T}}^{s}\sim{\widehat{\mathbb{P}}^{s}}$ 并获得 $a\sim{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot|{\mathcal{T}}^{s},s)$。

我们有以下定理来表征 OfflineCoPS 的性能。

###### 定理 5.4.

通过设置

|  | $\displaystyle{\epsilon_{\text{pretrain}}}=C_{{\mathtt{Dec}}}T\cdot\sqrt{5\cdot T% \log(\mathcal{N}(1/({n_{\text{pre}}}T)^{2})T)\cdot{n_{\text{pre}}}^{-1}+T{% \epsilon_{\text{real}}}},$ |  |
| --- | --- | --- |

并且表示$\mathbb{P}^{*,s}=\mathop{\mathrm{argmax}}_{{\widehat{\mathbb{P}}}\in\mathcal{P% }}\mathbb{E}_{{\mathcal{T}}\sim{\widehat{\mathbb{P}}}(\cdot|\mathcal{D},s),a% \sim\overline{{\text{Alg}^{E}}}(\cdot|{\mathcal{T}},s)}r(s,a)$，我们有以下概率不小于$1-2/T$的界限：

|  | $\displaystyle\text{SubOpt}({\widehat{\mathbb{P}}^{s}})\leq 2C_{{\mathtt{Dec}}}% {\epsilon_{\text{pretrain}}}\mathbb{E}_{\mathbf{M}\sim\mathbb{P}^{\mathtt{M}},% s\sim\mathbb{P}_{0}^{\mathbf{M}}}\sqrt{1+\chi^{2}(\mathbb{P}^{*,s}(\cdot&#124;% \mathcal{D},s),\mathbb{P}^{\mathbf{M},{\text{Alg}^{C}}}_{T-1}(\cdot))}.$ |  |
| --- | --- | --- |

###### 证明。

请参见附录[A.1](https://arxiv.org/html/2410.16670v1#A1.SS1 "A.1 Proof of Theorem 5.4 ‣ Appendix A Additional Details in Section 5 ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing")。∎

定理[5.4](https://arxiv.org/html/2410.16670v1#S5.Thmtheorem4 "Theorem 5.4\. ‣ 5.2 Algorithms for the Offline Setting ‣ 5 Theoretical Framework of Experience-Assisted Agents ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing")提供了关于CoPS为何能实现优越性能的几个见解，以及在不同情况下如何定制经验选择：

+   •

    所选分布$\mathbb{P}^{*,s}$的最终次优性差距取决于解码器系数$C_{{\mathtt{Dec}}}$和预训练误差参数$\epsilon_{\text{pre}}$。这意味着对于更强大的LLM，所选经验分布$\mathbb{P}^{*,s}$将更接近最优分布。同时，$\mathbb{P}^{\mathbf{M},{\text{Alg}^{C}}}_{T-1}$的依赖性表明，LLM提供的任务依赖性经验集合分布作为强正则化器，用于选择最优的检索策略。

+   •

    预训练误差参数$\epsilon_{\text{pre}}$的最佳选择受到解码器系数$C_{{\mathtt{Dec}}}$、预训练集中预训练轨迹的数量${n_{\text{pre}}}$和模型能力误差$\epsilon_{\text{real}}$的影响。通常，对于更强大的大规模语言模型（LLM），当${n_{\text{pre}}}$较大且$\epsilon_{\text{real}}$较小时，我们的定理建议，智能体应更多地专注于将所选经验集合分布$\mathbb{P}^{*,s}$与解码器分布${\mathtt{Dec}}$对齐。这与我们在第[4.2](https://arxiv.org/html/2410.16670v1#S4.SS2 "4.2 Ablation Study ‣ 4 Experiments ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing")节中的观察一致，其中较小的模型，如LLaMA 3.1 8b，对超参数$c$的选择更为敏感。

### 5.3 在线设置的算法

我们还考虑了将 OfflineCoPS 的变种分析应用于在线设置。在这里，令 $\mathcal{P}=\{\mathbb{P}^{1}(\cdot|\cdot,\cdot),\dots,\mathbb{P}^{|\mathcal{P}% |}(\cdot|\cdot,\cdot)\}\subseteq 2^{\mathtt{T}_{t-1}\times\mathtt{S}% \rightarrow\Delta(\mathtt{T}_{t-1})}$，其中包括将经验集合 ${\mathcal{T}}_{t-1}$ 和测试状态 $s$ 映射到 $\mathtt{T}_{t-1}$ 上的分布的映射。每个 $\mathbb{P}^{i}$ 可以看作是一种策略，用于选择依赖于过去观察的经验集合。在第 $t$ 步，代理拥有历史记录 $\mathcal{H}_{t-1}=\{s_{1},a_{1},r_{1},\dots,s_{t-1},a_{t-1},r_{t-1}\}$。然后，代理接收 $s_{t}\sim\mathbb{P}_{0}^{\mathbf{M}_{t}}$，其中 $\mathbf{M}_{t}\sim\mathbb{P}^{\mathtt{M}}$。接着，代理通过某个算法选择 $\mathbb{P}_{t}$，并从 ${\mathcal{T}}_{t-1}\sim\mathbb{P}_{t}(\cdot|\mathcal{H}_{t-1},s_{t})$ 中采样。然后，代理采取行动 $a_{t}\sim{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot|{\mathcal{T}}_{t-1},s_{t})$。她的目标是最小化以下的遗憾：

|  | $\displaystyle\text{遗憾}_{T}:=\sum_{t=1}^{T}\mathbb{E}_{\mathbf{M}_{t}\sim% \mathbb{P}^{\mathtt{M}},s_{t}\sim\mathbb{P}_{0}^{\mathbf{M}_{t}}}\bigg{[}\max_% {\mathbb{P}^{i}\in\mathcal{P}}\mathbb{E}_{\begin{subarray}{c}{\mathcal{T}}_{t-% 1}\sim\mathbb{P}^{i}(\cdot&#124;\mathcal{H}_{t-1}),\\ \bar{a}\sim\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}}_{t-1},s_{t})\end{% subarray}}r(s_{t},\bar{a})-\mathbb{E}_{\begin{subarray}{c}{\mathcal{T}}_{t-1}% \sim\mathbb{P}_{t}(\cdot&#124;\mathcal{H}_{t-1}),\\ a_{t}\sim\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}}_{t-1},s_{t})\end{% subarray}}r(s_{t},a_{t})\bigg{]}.$ |  | (5.3) |
| --- | --- | --- | --- |

我们在算法 [3](https://arxiv.org/html/2410.16670v1#alg3 "算法 3 ‣ 5.3 在线设置的算法 ‣ 5 经验辅助代理的理论框架 ‣ CoPS：赋能 LLM 代理以可证明的跨任务经验共享") 中提出了算法 OnlineCoPS。类似于 OfflineCoPS，OnlineCoPS 采用一个解码器，该解码器以当前状态为输入，输出经验集合 ${\mathcal{T}}$ 的分布，旨在估计 LLM 输出分布 $\mathbb{P}_{t-1}^{\mathbf{M}_{t},{\text{Alg}^{C}}}$。与 OfflineCoPS 不同，OnlineCoPS 的优化目标（见 [5.4](https://arxiv.org/html/2410.16670v1#S5.E4 "在 4 ‣ 算法 3 ‣ 5.3 在线设置的算法 ‣ 5 经验辅助代理的理论框架 ‣ CoPS：赋能 LLM 代理以可证明的跨任务经验共享")）类似于源自在线决策问题的*乐观原则*（Abbasi-Yadkori 等，[2011](https://arxiv.org/html/2410.16670v1#bib.bib1)），旨在最大化奖励和解码器分布 ${\mathtt{Dec}}_{t-1}$ 与选定分布 ${\widehat{\mathbb{P}}^{t}}$ 之间的分布距离。同时，注意选定的经验集合分布仅依赖于过去的历史 $\mathcal{H}_{t-1}$，在在线决策过程的早期阶段，该历史较小。我们有以下定理来展示 OnlineCoPS 的理论保证。

算法 3 OnlineCoPS

0: LLM ${\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot|\cdot,\cdot)$，候选经验集合分布 $\mathcal{P}$，预训练误差参数 ${\epsilon_{\text{pretrain}}}$，任务解码器 ${\mathtt{Dec}}$。1: 令 $\mathcal{H}_{0}=\emptyset$。2: 对于 $t=1,\dots,T$ 执行3: 生成 $\mathbf{M}_{t}\sim\mathbb{P}^{\mathtt{M}}$，接收 $s_{t}\sim\mathbb{P}_{0}^{\mathbf{M}_{t}}$，解码 ${\mathtt{Dec}}_{t-1}(\cdot|s_{t})$4: 从 $\mathcal{P}$ 中选择 ${\widehat{\mathbb{P}}^{t}}$，使其最大化以下表达式：

|  | ${\widehat{\mathbb{P}}^{t}}=\mathop{\mathrm{argmax}}_{{\widehat{\mathbb{P}}}\in% \mathcal{P}}\mathbb{E}_{\begin{subarray}{c}{\mathcal{T}}\sim{\widehat{\mathbb{% P}}}(\cdot&#124;\mathcal{H}_{t-1},s_{t}),\\ a\sim{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{\mathcal{T}},s_{t})\end{% subarray}}r(s_{t},a)+{\epsilon_{\text{pretrain}}}\sqrt{1+\chi^{2}({\widehat{% \mathbb{P}}}(\cdot&#124;\mathcal{H}_{t-1},s_{t}),{\mathtt{Dec}}_{t-1}(\cdot&#124;s_{t}))}.$ |  | (5.4) |
| --- | --- | --- | --- |

5: 生成 ${\mathcal{T}}\sim{\widehat{\mathbb{P}}^{t}}(\cdot|\mathcal{H}_{t-1},s_{t})$ 并获得 $a_{t}\sim{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot|{\mathcal{T}},s_{t})$ 和 $r_{t}=r(s_{t},a_{t})$，设置 $\mathcal{H}_{t}=\mathcal{H}_{t-1}\cup(s_{t},a_{t},r_{t})$。6: 结束 循环

###### 定理 5.5。

设置

|  | $\displaystyle{\epsilon_{\text{pretrain}}}=C_{{\mathtt{Dec}}}\cdot T^{2}\cdot% \sqrt{5\cdot\frac{T\log(\mathcal{N}(1/({n_{\text{pre}}}T)^{2})T^{2})}{{n_{% \text{pre}}}}+T{\epsilon_{\text{real}}}},$ |  |
| --- | --- | --- |

并且记作$\mathbb{P}^{*,t}=\mathop{\mathrm{argmax}}_{\widehat{\mathbb{P}}\in\mathcal{P}}% \mathbb{E}_{\begin{subarray}{c}{\mathcal{T}}_{t-1}\sim\widehat{\mathbb{P}}(% \cdot|\mathcal{H}_{t-1},s_{t}),\\ \bar{a}\sim\overline{{\text{Alg}^{E}}}(\cdot|{\mathcal{T}}_{t-1},s_{t})\end{% subarray}}r(s_{t},\bar{a})$，我们有以下界限，以至少$1-2/T$的概率成立：

|  | $\displaystyle\text{Regret}_{T}\leq 2C_{{\mathtt{Dec}}}{\epsilon_{\text{% pretrain}}}\sum_{t=1}^{T}\sqrt{1+\chi^{2}(\mathbb{P}^{*,t}(\cdot&#124;\mathcal{H}_{% t-1},s_{t}),\mathbb{P}^{\mathbf{M}_{t},{\text{Alg}^{C}}}_{t-1}(\cdot))}.$ |  |
| --- | --- | --- |

###### 证明。

见附录[A.2](https://arxiv.org/html/2410.16670v1#A1.SS2 "A.2 Theorem 5.5的证明 ‣ 附录A，第5节的附加细节 ‣ CoPS：通过可证明的跨任务经验共享赋能LLM代理")。∎

类似于离线设置中的定理[5.4](https://arxiv.org/html/2410.16670v1#S5.Thmtheorem4 "定理 5.4 ‣ 5.2 离线设置的算法 ‣ 5 经验辅助代理的理论框架 ‣ CoPS：通过可证明的跨任务经验共享赋能LLM代理")，定理[5.5](https://arxiv.org/html/2410.16670v1#S5.Thmtheorem5 "定理 5.5 ‣ 5.3 在线设置的算法 ‣ 5 经验辅助代理的理论框架 ‣ CoPS：通过可证明的跨任务经验共享赋能LLM代理")也提供了以下见解。

+   •

    后悔（regret）由生成的最佳经验集合分布${\mathbb{P}^{*,t}}$与由上下文算法在$t$-步产生的经验集合分布之间的差异控制。因此，整体上最佳的策略是从历史轨迹$\mathcal{H}_{t-1}$中选择能够较好地逼近当前任务的轨迹，以避免分布偏移。

+   •

    使用更强大的LLM时，${\epsilon_{\text{pretrain}}}$将更小，这意味着选择的经验集合能够更好地接近最佳选择。

## 6 结论、局限性与未来工作

本文介绍了 CoPS（跨任务经验共享），这是一种理论基础的算法，旨在为智能体系统赋能，支持跨任务的经验共享。CoPS 采用基于悲观策略的方法选择相关经验，最大化效用的同时最小化分布偏移的风险。我们在 Alfworld、Webshop 和 HotPotQA 等基准上的实验表明，CoPS 在成功率和样本效率上都优于最先进的方法。从理论上讲，我们展示了该算法的表现取决于 LLM 的预训练质量以及智能体所选择的试验决定的跨任务经验分布与 LLM 所定义的任务依赖经验分布之间的匹配，为改进经验检索方法提供了见解。

尽管 CoPS 相较于现有方法显示出明显的改进，但它也存在一些局限性。其有效性在很大程度上依赖于内存库中经验的质量和多样性，这意味着过时或不匹配的经验可能会降低其表现。此外，CoPS 对超参数（如缩放因子和上下文中经验的数量）较为敏感，这些超参数可能需要耗时的调优，而调优结果并不总是能在不同任务或模型间进行良好的泛化。最后，我们提供的理论保证还依赖于解码器的准确性以及 LLM 的特定预训练属性，这些假设在实际应用中可能并不总是成立。

展望未来，几个研究方向可以进一步改进 CoPS。这些方向包括开发自适应超参数调优方法，探索动态内存管理以保持经验的相关性，以及增加评估经验质量的方法。此外，将 CoPS 适配到多智能体系统并将其与强化学习相结合，可能会促进协作学习和持续的性能提升。追求这些方向将有助于 LLM 智能体处理更加复杂的顺序推理任务。

## 附录 A 在第 [5](https://arxiv.org/html/2410.16670v1#S5 "5 Theoretical Framework of Experience-Assisted Agents ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing")节的附加细节

### A.1 定理 [5.4](https://arxiv.org/html/2410.16670v1#S5.Thmtheorem4 "Theorem 5.4\. ‣ 5.2 Algorithms for the Offline Setting ‣ 5 Theoretical Framework of Experience-Assisted Agents ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing") 的证明

我们在此证明了定理 [5.4](https://arxiv.org/html/2410.16670v1#S5.Thmtheorem4 "Theorem 5.4\. ‣ 5.2 Algorithms for the Offline Setting ‣ 5 Theoretical Framework of Experience-Assisted Agents ‣ CoPS: Empowering LLM Agents with Provable Cross-Task Experience Sharing")。首先，我们需要以下引理。

###### 引理 A.1  （引理 20，Lin 等人 [2023](https://arxiv.org/html/2410.16670v1#bib.bib28)）。

以至少 $1-\delta$ 的概率，我们有

|  | $\displaystyle\mathbb{E}_{\mathbf{M}\sim\mathbb{P}^{\mathtt{M}},s\sim\mathbb{P}% _{0}^{\mathbf{M}},{\mathcal{T}}\sim\mathbb{P}^{\mathbf{M},{\text{Alg}^{C}}}_{T% -1}}\bigg{[}\sum_{t=1}^{T}{\text{D}_{\text{H}}}^{2}(\overline{{\text{Alg}^{E}}% }(\cdot&#124;{\mathcal{T}}_{t-1},s),{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{% \mathcal{T}}_{t-1},s))\bigg{]}$ |  |
| --- | --- | --- |
|  | $\displaystyle\leq 5\cdot\frac{T\log(\mathcal{N}(1/({n_{\text{pre}}}T)^{2})T/% \delta)}{{n_{\text{pre}}}}+T{\epsilon_{\text{real}}},$ |  |

其中覆盖数 $\mathcal{N}$ 在定义 [5.1](https://arxiv.org/html/2410.16670v1#S5.Thmtheorem1 "定义 5.1 (Lin 等, 2023). ‣ 5.1 LLM 预训练 ‣ 经验辅助智能体的理论框架 ‣ CoPS：赋能 LLM 智能体以可证明的跨任务经验共享") 中定义，${\epsilon_{\text{real}}}$ 在假设 [5.2](https://arxiv.org/html/2410.16670v1#S5.Thmtheorem2 "假设 5.2 (Lin 等, 2023). ‣ 5.1 LLM 预训练 ‣ 经验辅助智能体的理论框架 ‣ CoPS：赋能 LLM 智能体以可证明的跨任务经验共享") 中定义。

接下来的引理用于提供每个状态的泛化误差保证。

###### 引理 A.2。

让事件 ${\mathcal{E}}$ 定义为

|  | $\displaystyle\mathbb{E}_{{\mathcal{T}}\sim\mathbb{P}^{\mathbf{M},{\text{Alg}^{% C}}}_{T-1}}\bigg{[}\sum_{t=1}^{T}{\text{D}_{\text{H}}}^{2}(\overline{{\text{% Alg}^{E}}}(\cdot&#124;{\mathcal{T}}_{t-1},s),{\text{Alg}_{\widehat{\bm{\theta}}}}(% \cdot&#124;{\mathcal{T}}_{t-1},s))\bigg{]}\leq m_{c}\bigg{[}c\cdot\frac{T\log(% \delta^{-1}\mathcal{N}(1/({n_{\text{pre}}}T)^{2})T)}{{n_{\text{pre}}}}+T{% \epsilon_{\text{real}}}\bigg{]},$ |  |
| --- | --- | --- |

其中 ${\epsilon_{\text{real}}}$ 在假设 [5.2](https://arxiv.org/html/2410.16670v1#S5.Thmtheorem2 "假设 5.2 (Lin 等, 2023). ‣ 5.1 LLM 预训练 ‣ 经验辅助智能体的理论框架 ‣ CoPS：赋能 LLM 智能体以可证明的跨任务经验共享") 中定义。然后我们有 $\mathbb{P}(\mathcal{E})\geq 1-1/m_{c}-\delta$。

###### 证明。

根据马尔可夫不等式，我们可以得出，在最多 $1/m_{c}$ 的概率下，

|  | $\displaystyle\mathbb{E}_{{\mathcal{T}}\sim\mathbb{P}^{\mathbf{M},{\text{Alg}^{% C}}}_{T-1}}\bigg{[}\sum_{t=1}^{T}{\text{D}_{\text{H}}}^{2}(\overline{{\text{% Alg}^{E}}}(\cdot&#124;{\mathcal{T}}_{t-1},s),{\text{Alg}_{\widehat{\bm{\theta}}}}(% \cdot&#124;{\mathcal{T}}_{t-1},s))\bigg{]}$ |  |
| --- | --- | --- |
|  | $\displaystyle\geq m_{c}\cdot\mathbb{E}_{\mathbf{M}\sim\mathbb{P}^{\mathtt{M}},% s\sim\mathbb{P}_{0}^{\mathbf{M}},{\mathcal{T}}\sim\mathbb{P}^{\mathbf{M},{% \text{Alg}^{C}}}_{T-1}}\bigg{[}\sum_{t=1}^{T}{\text{D}_{\text{H}}}^{2}(% \overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}}_{t-1},s),{\text{Alg}_{\widehat% {\bm{\theta}}}}(\cdot&#124;{\mathcal{T}}_{t-1},s))\bigg{]}.$ |  |

与此同时，按照引理 [A.1](https://arxiv.org/html/2410.16670v1#A1.Thmtheorem1 "引理 A.1（Lin 等，2023）。 ‣ A.1 定理 5.4 的证明 ‣ 附录 A 第 5 节中的附加细节 ‣ CoPS: 通过可证明的跨任务经验共享增强大型语言模型智能体")，我们知道，在最多概率为 $\delta$ 的情况下，我们有

|  | $\displaystyle\mathbb{E}_{\mathbf{M}\sim\mathbb{P}^{\mathtt{M}},s\sim\mathbb{P}% _{0}^{\mathbf{M}},{\mathcal{T}}\sim\mathbb{P}^{\mathbf{M},{\text{Alg}^{C}}}_{T% -1}}\bigg{[}\sum_{t=1}^{T}{\text{D}_{\text{H}}}^{2}(\overline{{\text{Alg}^{E}}% }(\cdot&#124;{\mathcal{T}}_{t-1},s),{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{% \mathcal{T}}_{t-1},s))\bigg{]}$ |  |
| --- | --- | --- |
|  | $\displaystyle\geq c\cdot\frac{T\log(\delta^{-1}\cdot\mathcal{N}(1/({n_{\text{% pre}}}T)^{2})T)}{{n_{\text{pre}}}}+T{\epsilon_{\text{real}}}.$ |  |

因此，通过并集界限，我们有 $\mathbb{P}({\mathcal{E}})\geq 1-\delta-1/m_{c}$。∎

现在我们开始证明定理 [5.4](https://arxiv.org/html/2410.16670v1#S5.Thmtheorem4 "定理 5.4. ‣ 5.2 离线设置的算法 ‣ 5 基于经验的智能体的理论框架 ‣ CoPS: 通过可证明的跨任务经验共享增强大型语言模型智能体")。

###### 证明。

我们遵循 Lin 等人 ([2023](https://arxiv.org/html/2410.16670v1#bib.bib28)) 的证明步骤。我们假设引理 [A.2](https://arxiv.org/html/2410.16670v1#A1.Thmtheorem2 "引理 A.2. ‣ A.1 定理 5.4 的证明 ‣ 附录 A 第 5 节中的附加细节 ‣ CoPS: 通过可证明的跨任务经验共享增强大型语言模型智能体") 中的事件 ${\mathcal{E}}$ 成立。我们首先通过它们的分布距离的差异来界定奖励的差异。设 ${\widehat{\mathbb{P}}}$ 为关于 ${\mathcal{T}}$ 的任意分布。那么我们有

|  | $\displaystyle\mathbb{E}_{{\mathcal{T}}\sim{\widehat{\mathbb{P}}}(\cdot)}&#124;% \mathbb{E}_{a\sim\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}},s)}r(s,a)-% \mathbb{E}_{a\sim{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{\mathcal{T}},s)}r% (s,a)&#124;$ |  |
| --- | --- | --- |
|  | $\displaystyle\leq\mathbb{E}_{{\mathcal{T}}\sim{\widehat{\mathbb{P}}}(\cdot)}{% \text{D}_{\text{TV}}}(\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}},s),{% \text{Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{\mathcal{T}},s))$ |  |
|  | $\displaystyle\leq\mathbb{E}_{{\mathcal{T}}\sim{\widehat{\mathbb{P}}}(\cdot)}{% \text{D}_{\text{H}}}(\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}},s),{\text% {Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{\mathcal{T}},s)),$ |  | (A.1) |

第一个不等式成立是因为 $|r|\leq 1$ 和 TV 距离的性质，第二个不等式成立是因为 ${\text{D}_{\text{TV}}}\leq{\text{D}_{\text{H}}}$。从 ([A.1](https://arxiv.org/html/2410.16670v1#A1.E1 "在证明中. ‣ A.1 定理 5.4 的证明 ‣ 附录 A 第 5 节中的附加细节 ‣ CoPS: 通过可证明的跨任务经验共享增强大型语言模型智能体")) 我们有

|  | $\displaystyle\mathbb{E}_{{\mathcal{T}}\sim{\widehat{\mathbb{P}}}}{\text{D}_{% \text{H}}}(\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}},s),{\text{Alg}_{% \widehat{\bm{\theta}}}}(\cdot&#124;{\mathcal{T}},s))$ |  |
| --- | --- | --- |
|  | $\displaystyle=\mathbb{E}_{{\mathcal{T}}\sim\mathbb{P}^{\mathbf{M},{\text{Alg}^% {C}}}_{T-1}}{\text{D}_{\text{H}}}(\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{% T}},s),{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{\mathcal{T}},s))\cdot\frac{% {\widehat{\mathbb{P}}}({\mathcal{T}})}{\mathbb{P}^{\mathbf{M},{\text{Alg}^{C}}% }_{T-1}({\mathcal{T}})}$ |  |
|  | $\displaystyle\leq\sqrt{\underbrace{\mathbb{E}_{{\mathcal{T}}\sim\mathbb{P}^{% \mathbf{M},{\text{Alg}^{C}}}_{T-1}}{\text{D}_{\text{H}}}^{2}(\overline{{\text{% Alg}^{E}}}(\cdot&#124;{\mathcal{T}},s),{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{% \mathcal{T}},s))}_{I_{1}}}\cdot\sqrt{\underbrace{\mathbb{E}_{{\mathcal{T}}\sim% \mathbb{P}^{\mathbf{M},{\text{Alg}^{C}}}_{T-1}}\bigg{(}\frac{{\widehat{\mathbb% {P}}}({\mathcal{T}})}{\mathbb{P}^{\mathbf{M},{\text{Alg}^{C}}}_{T-1}({\mathcal% {T}})}\bigg{)}^{2}}_{I_{2}}},$ |  | (A.2) |

其中，第一个不等式由于柯西-施瓦茨不等式成立。对于$I_{1}$，我们使用引理[A.1](https://arxiv.org/html/2410.16670v1#A1.Thmtheorem1 "引理 A.1（引理 20，Lin 等，2023年）。 ‣ A.1 定理 5.4 的证明 ‣ 附录 A 中的额外细节 ‣ CoPS：通过可证明的跨任务经验共享增强大型语言模型（LLM）代理")。注意到$|{\mathcal{T}}|=T-1$及${\epsilon_{\text{pretrain}}}$的定义，我们得到

|  | $\displaystyle I_{1}\leq({\epsilon_{\text{pretrain}}}/C_{{\mathtt{Dec}}})^{2}.$ |  | (A.3) |
| --- | --- | --- | --- |

对于$I_{2}$，根据$\chi^{2}$距离的定义，我们有

|  | $\displaystyle I_{2}$ | $\displaystyle=\mathbb{E}_{{\mathcal{T}}\sim{\widehat{\mathbb{P}}}}\frac{{% \widehat{\mathbb{P}}}({\mathcal{T}})}{\mathbb{P}^{\mathbf{M},{\text{Alg}^{C}}}% _{T-1}({\mathcal{T}})}$ |  |
| --- | --- | --- | --- |
|  |  | $\displaystyle=\mathbb{E}_{{\mathcal{T}}\sim{\widehat{\mathbb{P}}}}\frac{{% \widehat{\mathbb{P}}}({\mathcal{T}})}{{\mathtt{Dec}}_{T-1}({\mathcal{T}}&#124;s)}% \cdot\frac{{\mathtt{Dec}}_{T-1}({\mathcal{T}}&#124;s)}{\mathbb{P}^{\mathbf{M},{% \text{Alg}^{C}}}_{T-1}({\mathcal{T}})}$ |  |
|  |  | $\displaystyle\leq C_{{\mathtt{Dec}}}^{2}[1+\chi^{2}({\widehat{\mathbb{P}}}(% \cdot),{\mathtt{Dec}}_{T-1}(\cdot&#124;s))].$ |  | (A.4) |

该不等式由于假设[5.3](https://arxiv.org/html/2410.16670v1#S5.Thmtheorem3 "假设 5.3\. ‣ 5.1 LLM预训练 ‣ 5 经验辅助代理的理论框架 ‣ CoPS：赋能LLM代理通过可证明的跨任务经验共享")成立。通过代入([A.3](https://arxiv.org/html/2410.16670v1#A1.E3 "证明中 ‣ A.1 定理5.4的证明 ‣ 附录A第5节的附加细节 ‣ CoPS：赋能LLM代理通过可证明的跨任务经验共享"))和([A.4](https://arxiv.org/html/2410.16670v1#A1.E4 "证明中 ‣ A.1 定理5.4的证明 ‣ 附录A第5节的附加细节 ‣ CoPS：赋能LLM代理通过可证明的跨任务经验共享"))到([A.2](https://arxiv.org/html/2410.16670v1#A1.E2 "证明中 ‣ A.1 定理5.4的证明 ‣ 附录A第5节的附加细节 ‣ CoPS：赋能LLM代理通过可证明的跨任务经验共享"))，再将([A.2](https://arxiv.org/html/2410.16670v1#A1.E2 "证明中 ‣ A.1 定理5.4的证明 ‣ 附录A第5节的附加细节 ‣ CoPS：赋能LLM代理通过可证明的跨任务经验共享"))代入([A.1](https://arxiv.org/html/2410.16670v1#A1.E1 "证明中 ‣ A.1 定理5.4的证明 ‣ 附录A第5节的附加细节 ‣ CoPS：赋能LLM代理通过可证明的跨任务经验共享"))，我们得到

|  | $\displaystyle&#124;\mathbb{E}_{{\mathcal{T}}\sim{\widehat{\mathbb{P}}},a\sim% \overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}},s)}r(s,a)-\mathbb{E}_{{% \mathcal{T}}\sim{\widehat{\mathbb{P}}},a\sim{\text{Alg}_{\widehat{\bm{\theta}}% }}(\cdot&#124;{\mathcal{T}},s)}r(s,a)&#124;$ |  |
| --- | --- | --- |
|  | $\displaystyle\leq{\epsilon_{\text{pretrain}}}\sqrt{1+\chi^{2}({\widehat{% \mathbb{P}}}(\cdot),{\mathtt{Dec}}_{T-1}(\cdot&#124;s))},$ |  | (A.5) |

对于任何${\widehat{\mathbb{P}}}\in\mathcal{P}$，该式成立。最后，我们得到

|  | $\displaystyle\mathbb{E}_{{\mathcal{T}}^{s}\sim{\widehat{\mathbb{P}}^{s}}(\cdot% &#124;\mathcal{D},s),a\sim\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}}^{s},s)}r(% s,a)$ |  |
| --- | --- | --- |
|  | $\displaystyle\geq\mathbb{E}_{{\mathcal{T}}^{s}\sim{\widehat{\mathbb{P}}^{s}}(% \cdot&#124;\mathcal{D},s),a\sim{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{\mathcal% {T}}^{s},s)}r(s,a)-{\epsilon_{\text{pretrain}}}\sqrt{1+\chi^{2}({\widehat{% \mathbb{P}}^{s}}(\cdot&#124;\mathcal{D},s),{\mathtt{Dec}}_{T-1}({\mathcal{T}}&#124;s))}$ |  |
|  | $\displaystyle\geq\mathbb{E}_{{\mathcal{T}}^{s}\sim\mathbb{P}^{*,s}(\cdot&#124;% \mathcal{D},s),a\sim{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{\mathcal{T}}^{% s},s)}r(s,a)-{\epsilon_{\text{pretrain}}}\sqrt{1+\chi^{2}(\mathbb{P}^{*,s}(% \cdot&#124;\mathcal{D},s),{\mathtt{Dec}}_{T-1}({\mathcal{T}}&#124;s))},$ |  |
|  | $\displaystyle\geq\mathbb{E}_{{\mathcal{T}}^{s}\sim\mathbb{P}^{*,s}(\cdot&#124;% \mathcal{D},s),a\sim\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}}^{s},s)}r(s% ,a)-2{\epsilon_{\text{pretrain}}}\sqrt{1+\chi^{2}(\mathbb{P}^{*,s}(\cdot&#124;% \mathcal{D},s),{\mathtt{Dec}}_{T-1}({\mathcal{T}}&#124;s))},$ |  |
|  | $\displaystyle\geq\mathbb{E}_{{\mathcal{T}}^{s}\sim\mathbb{P}^{*,s}(\cdot&#124;% \mathcal{D},s),a\sim\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}}^{s},s)}r(s% ,a)-2{\epsilon_{\text{pretrain}}}C_{{\mathtt{Dec}}}\sqrt{1+\chi^{2}(\mathbb{P}% ^{*,s}(\cdot&#124;\mathcal{D},s),\mathbb{P}^{\mathbf{M},{\text{Alg}^{C}}}_{T-1}(% \cdot))},$ |  |

其中，第一个不等式由于([A.5](https://arxiv.org/html/2410.16670v1#A1.E5 "在证明中. ‣ A.1 定理5.4的证明 ‣ 附录A 第5节的附加细节 ‣ CoPS：通过可证明的跨任务经验共享赋能LLM代理"))成立，第二个不等式由于选择规则${\widehat{\mathbb{P}}^{s}}$成立，第三个不等式由于([A.5](https://arxiv.org/html/2410.16670v1#A1.E5 "在证明中. ‣ A.1 定理5.4的证明 ‣ 附录A 第5节的附加细节 ‣ CoPS：通过可证明的跨任务经验共享赋能LLM代理"))成立，最后一个不等式由于假设[5.3](https://arxiv.org/html/2410.16670v1#S5.Thmtheorem3 "假设 5.3\. ‣ 5.1 LLM预训练 ‣ 5 经验辅助代理的理论框架 ‣ CoPS：通过可证明的跨任务经验共享赋能LLM代理")成立。证明完毕。

∎

### A.2 定理[5.5](https://arxiv.org/html/2410.16670v1#S5.Thmtheorem5 "定理 5.5\. ‣ 5.3 在线设置的算法 ‣ 5 经验辅助代理的理论框架 ‣ CoPS：通过可证明的跨任务经验共享赋能LLM代理")的证明

###### 证明。

假设我们在第$t$步，并且我们以所有过去的历史记录$\mathcal{H}_{t-1}=(s_{1},a_{1},r_{1},\dots,s_{t-1},a_{t-1},r_{t-1})$为条件。令$\mathbf{M}_{t}$为第$t$步的任务，$s_{t}$为观察到的状态。那么，至少以$1-1/m_{c}-\delta$的概率，以下事件$\mathcal{E}_{t}$成立：

|  | $\displaystyle\mathbb{E}_{{\mathcal{T}}\sim\mathbb{P}^{\mathbf{M}_{t},{\text{% Alg}^{C}}}_{t-1}}\bigg{[}{\text{D}_{\text{H}}}^{2}(\overline{{\text{Alg}^{E}}}% (\cdot&#124;{\mathcal{T}}_{t-1},s_{t}),{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{% \mathcal{T}}_{t-1},s_{t}))\bigg{]}$ |  |
| --- | --- | --- |
|  | $\displaystyle\leq m_{c}\bigg{[}c\cdot\frac{T\log(\delta^{-1}\mathcal{N}(1/({n_% {\text{pre}}}T)^{2})T^{2})}{{n_{\text{pre}}}}+T{\epsilon_{\text{real}}}\bigg{]},$ |  |

现在，参考([A.2](https://arxiv.org/html/2410.16670v1#A1.E2 "证明中 ‣ A.1 证明定理 5.4 ‣ 附录 A 第 5 节的附加细节 ‣ CoPS：通过可证明的跨任务经验共享增强大规模语言模型代理"))中的内容，我们仍然有：

|  | $\displaystyle\mathbb{E}_{{\mathcal{T}}\sim{\widehat{\mathbb{P}}}} | \mathbb{E}_{% a\sim\overline{{\text{Alg}^{E}}}(\cdot | {\mathcal{T}},s_{t})}r(s,a)-\mathbb{E}_% {a\sim{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot | {\mathcal{T}},s_{t})}r(s,a) | $ |  |
| --- | --- | --- | --- | --- | --- | --- |
|  | $\displaystyle\leq\sqrt{\underbrace{\mathbb{E}_{{\mathcal{T}}\sim\mathbb{P}^{% \mathbf{M}_{t},{\text{Alg}^{C}}}_{t-1}}{\text{D}_{\text{H}}}^{2}(\overline{{% \text{Alg}^{E}}}(\cdot | {\mathcal{T}},s_{t}),{\text{Alg}_{\widehat{\bm{\theta}}% }}(\cdot | {\mathcal{T}},s_{t}))}_{I_{1}}}\cdot\sqrt{\underbrace{\mathbb{E}_{{% \mathcal{T}}\sim\mathbb{P}^{\mathbf{M}_{t},{\text{Alg}^{C}}}_{t-1}}\bigg{(}% \frac{{\widehat{\mathbb{P}}}({\mathcal{T}})}{\mathbb{P}^{\mathbf{M}_{t},{\text% {Alg}^{C}}}_{t-1}({\mathcal{T}})}\bigg{)}^{2}}_{I_{2}}}$ |  | (A.6) |

然后，参考引理[A.2](https://arxiv.org/html/2410.16670v1#A1.Thmtheorem2 "引理 A.2 ‣ A.1 证明定理 5.4 ‣ 附录 A 第 5 节的附加细节 ‣ CoPS：通过可证明的跨任务经验共享增强大规模语言模型代理")，在事件$\mathcal{E}_{t}$下，我们有：

|  | $\displaystyle I_{1}\leq({\epsilon_{\text{pretrain}}}/C_{{\mathtt{Dec}}})^{2},% \ {\epsilon_{\text{pretrain}}}/C_{{\mathtt{Dec}}}=T^{2}\cdot\sqrt{c\cdot\frac{% T\log(\mathcal{N}(1/({n_{\text{pre}}}T)^{2})T^{2})}{{n_{\text{pre}}}}+T{% \epsilon_{\text{real}}}}.$ |  |
| --- | --- | --- |

对于$I_{2}$，类似于([A.4](https://arxiv.org/html/2410.16670v1#A1.E4 "证明中 ‣ A.1 证明定理 5.4 ‣ 附录 A 第 5 节的附加细节 ‣ CoPS：通过可证明的跨任务经验共享增强大规模语言模型代理"))，我们有：

|  | $\displaystyle I_{2}\leq C_{{\mathtt{Dec}}}^{2}[1+\chi^{2}({\widehat{\mathbb{P}}}(\cdot),{\mathtt{Dec}}_{t-1}(\cdot | s_{t}))].$ |  |
| --- | --- | --- | --- |

因此，对于任何${\widehat{\mathbb{P}}}$，我们有：

|  | $\displaystyle | \mathbb{E}_{{\mathcal{T}}\sim{\widehat{\mathbb{P}}}(\cdot | % \mathcal{H}_{t-1},s_{t}),a\sim\overline{{\text{Alg}^{E}}}(\cdot | {\mathcal{T}},% s_{t})}r(s_{t},a)-\mathbb{E}_{{\mathcal{T}}\sim{\widehat{\mathbb{P}}}(\cdot | % \mathcal{H}_{t-1},s_{t}),a\sim{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot | {% \mathcal{T}},s_{t})}r(s_{t},a) | $ |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  | $\displaystyle\leq{\epsilon_{\text{pretrain}}}\sqrt{1+\chi^{2}({\widehat{% \mathbb{P}}}(\cdot | \mathcal{H}_{t-1},s_{t}),{\mathtt{Dec}}_{t-1}(\cdot | s_{t}))}.$ |  | (A.7) |

取并集上界，并令$m_{c}=T^{2},\delta=1/T^{2}$，则我们得到$\mathcal{E}_{1},...,\mathcal{E}_{T}$以至少$1-2/T$的概率成立。接下来，我们将$t$步的次优间隙上界如下：

|  | $\displaystyle\mathbb{E}_{{\mathcal{T}}^{t-1}\sim{\widehat{\mathbb{P}}^{t}}(% \cdot&#124;\mathcal{H}_{t-1},s_{t}),a\sim\overline{{\text{Alg}^{E}}}(\cdot&#124;{% \mathcal{T}}^{t-1},s_{t})}r(s_{t},a)$ |  |
| --- | --- | --- |
|  | $\displaystyle\leq\mathbb{E}_{\begin{subarray}{c}{\mathcal{T}}^{t-1}\sim{% \widehat{\mathbb{P}}^{t}}(\cdot&#124;\mathcal{H}_{t-1},s_{t}),\\ a\sim{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{\mathcal{T}}^{t-1},s_{t})\end% {subarray}}r(s_{t},a)+{\epsilon_{\text{pretrain}}}\sqrt{1+\chi^{2}({\widehat{% \mathbb{P}}^{t}}(\cdot&#124;\mathcal{H}_{t-1},s_{t}),{\mathtt{Dec}}_{t-1}(\cdot&#124;s_{% t}))}$ |  |
|  | $\displaystyle\leq\mathbb{E}_{\begin{subarray}{c}{\mathcal{T}}^{t-1}\sim{% \mathbb{P}^{*,t}}(\cdot&#124;\mathcal{H}_{t-1},s_{t}),\\ a\sim{\text{Alg}_{\widehat{\bm{\theta}}}}(\cdot&#124;{\mathcal{T}}^{t-1},s_{t})\end% {subarray}}r(s_{t},a)+{\epsilon_{\text{pretrain}}}\sqrt{1+\chi^{2}({\mathbb{P}% ^{*,t}}(\cdot&#124;\mathcal{H}_{t-1},s_{t}),{\mathtt{Dec}}_{t-1}(\cdot&#124;s_{t}))}$ |  |
|  | $\displaystyle\leq\mathbb{E}_{\begin{subarray}{c}{\mathcal{T}}^{t-1}\sim{% \mathbb{P}^{*,t}}(\cdot&#124;\mathcal{H}_{t-1},s_{t}),\\ a\sim\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}}^{t-1},s_{t})\end{subarray% }}r(s_{t},a)+2{\epsilon_{\text{pretrain}}}\sqrt{1+\chi^{2}({\mathbb{P}^{*,t}}(% \cdot&#124;\mathcal{H}_{t-1},s_{t}),{\mathtt{Dec}}_{t-1}(\cdot&#124;s_{t}))}$ |  |
|  | $\displaystyle\leq\mathbb{E}_{\begin{subarray}{c}{\mathcal{T}}^{t-1}\sim{% \mathbb{P}^{*,t}}(\cdot&#124;\mathcal{H}_{t-1},s_{t}),\\ a\sim\overline{{\text{Alg}^{E}}}(\cdot&#124;{\mathcal{T}}^{t-1},s_{t})\end{subarray% }}r(s_{t},a)+2C_{{\mathtt{Dec}}}{\epsilon_{\text{pretrain}}}\sqrt{1+\chi^{2}({% \mathbb{P}^{*,t}}(\cdot&#124;\mathcal{H}_{t-1},s_{t}),\mathbb{P}^{\mathbf{M}_{t},{% \text{Alg}^{C}}}_{t-1}(\cdot))},$ |  | (A.8) |

其中，第一个不等式成立是由于([A.7](https://arxiv.org/html/2410.16670v1#A1.E7 "证明中 ‣ A.2 定理 5.5 证明 ‣ 附录 A 第 5 节的附加细节 ‣ CoPS：通过可证明的跨任务经验共享赋能大语言模型代理"))，第二个不等式成立是因为乐观原则，第三个不等式成立是由于([A.7](https://arxiv.org/html/2410.16670v1#A1.E7 "证明中 ‣ A.2 定理 5.5 证明 ‣ 附录 A 第 5 节的附加细节 ‣ CoPS：通过可证明的跨任务经验共享赋能大语言模型代理"))，最后一个不等式成立是因为假设[5.3](https://arxiv.org/html/2410.16670v1#S5.Thmtheorem3 "假设 5.3 ‣ 5.1 大语言模型预训练 ‣ 5 经验辅助代理的理论框架 ‣ CoPS：通过可证明的跨任务经验共享赋能大语言模型代理")。对([A.8](https://arxiv.org/html/2410.16670v1#A1.E8 "证明中 ‣ A.2 定理 5.5 证明 ‣ 附录 A 第 5 节的附加细节 ‣ CoPS：通过可证明的跨任务经验共享赋能大语言模型代理"))从$1$加和到$T$，完成了我们的证明。∎

## 附录 B 更多实验细节

在本节中，我们提供了第[4.1](https://arxiv.org/html/2410.16670v1#S4.SS1 "4.1 结果与分析 ‣ 4 实验 ‣ CoPS：通过可证明的跨任务经验共享赋能大语言模型代理")节实验的附加细节。下面的表格概述了整个评估过程中使用的超参数设置。

表格 5：不同基准和模型大小的超参数设置（$k$ 和 $c$）。

| 基准 | Alfworld | Webshop | HotPotQA |
| --- | --- | --- | --- |
| Llama3.1 8b | $k=5,c=5$ | $k=5,c=0$ | $k=5,c=5$ |
| Llama3.1 70b | $k=5,c=5$ | $k=5,c=0$ | $k=5,c=0$ |

值得注意的是，跨越各个领域有许多蓬勃发展的代表性基准和任务，例如推理（Zeng 等，[2024](https://arxiv.org/html/2410.16670v1#bib.bib57)）、代码生成（Dai 等，[2024](https://arxiv.org/html/2410.16670v1#bib.bib7)）、长上下文信息提取（Ge 等，[2024](https://arxiv.org/html/2410.16670v1#bib.bib12); Wang 等，[2023a](https://arxiv.org/html/2410.16670v1#bib.bib44)）、视觉识别（Wang 等，[2023b](https://arxiv.org/html/2410.16670v1#bib.bib45); Kang 等，[2021](https://arxiv.org/html/2410.16670v1#bib.bib24); Yang 等，[2022](https://arxiv.org/html/2410.16670v1#bib.bib52); Chen 等，[2024a](https://arxiv.org/html/2410.16670v1#bib.bib4)）、数学推理（Xiong 等，[2022](https://arxiv.org/html/2410.16670v1#bib.bib50); Zhao 和 Zhang，[2024](https://arxiv.org/html/2410.16670v1#bib.bib59)）、分类（Wan，[2023](https://arxiv.org/html/2410.16670v1#bib.bib42); Kang 等，[2022](https://arxiv.org/html/2410.16670v1#bib.bib25); Huang 等，[2024a](https://arxiv.org/html/2410.16670v1#bib.bib18); Liu 等，[2023b](https://arxiv.org/html/2410.16670v1#bib.bib30); Tao 等，[2024](https://arxiv.org/html/2410.16670v1#bib.bib40)）、有害检测（Zhao 等，[2024](https://arxiv.org/html/2410.16670v1#bib.bib58); Huang 和 Sun，[2023](https://arxiv.org/html/2410.16670v1#bib.bib21); Fan 和 Tao，[2024](https://arxiv.org/html/2410.16670v1#bib.bib10)）。这些广泛的应用领域蕴藏着 CoPS 的潜力。我们将 CoPS 应用于上述基准的工作留待未来进行。

## 附录 C 提示模板

我们的提示框架在设计时考虑了简洁性、效率性和通用性。此外，我们旨在最大限度地利用现代大型语言模型（LLMs）的高效长上下文能力（Dubey 等，[2024](https://arxiv.org/html/2410.16670v1#bib.bib9)）。用一句话来描述我们的提示哲学：代理将先前的成功轨迹视为它在同一试验中经历的*经验*。以 Alfworld 实验为例，实例展示被插入到代理提示的前缀中，如下所示：

{mdframed}

[用户]与一个家庭互动以完成任务。以下是一些示例。你站在一个房间的中央，快速环顾四周，你看到13个柜子、12个柜子、11个柜子、10个柜子、9个柜子、8个柜子、7个柜子、6个柜子、5个柜子、4个柜子、3个柜子、2个柜子、1个柜子、1个咖啡机、1个台面、1个餐桌、1个抽屉、1个冰箱、1个垃圾桶、1个微波炉、3个架子、2个架子、1个架子、1个水槽、4个炉灶、3个炉灶、2个炉灶、1个炉灶和1个烤面包机。你的任务是：把一个热苹果放进冰箱。[助手]思考：为了解决这个任务，我需要找到并拿到一个苹果，然后用微波炉加热，再把它放进冰箱。[用户]好的。[助手]思考：首先我需要找到一个苹果。苹果更可能出现在冰箱（1）、餐桌（1）、咖啡桌（1）、抽屉（1）、柜子（1-13）、垃圾桶（1）中。我可以逐个检查，从冰箱1开始。[用户]好的。[助手]走向冰箱1。[用户]冰箱1是关闭的。[经验细节省略][助手]走向冰箱1。[用户]冰箱1是打开的。在里面，你看到1个杯子和1个鸡蛋。[助手]把苹果1放进/放到冰箱1。[用户]你把苹果1放进/放到冰箱1。[更多上下文经验][用户]示例结束。以下是任务：你站在一个房间的中央，快速环顾四周，你看到6个柜子、5个柜子、4个柜子、3个柜子、2个柜子、1个柜子、1个咖啡机、3个台面、2个台面、1个台面、3个抽屉、2个抽屉、1个抽屉、1个冰箱、1个垃圾桶、1个微波炉、3个架子、2个架子、1个架子、1个水槽、4个炉灶、3个炉灶、2个炉灶、1个炉灶和1个烤面包机。你的任务是：把一个盐瓶放入抽屉中。[助手]思考：为了解决这个任务，我需要找到并拿到一个盐瓶，然后把它放入/放到一个抽屉中。[实际试验开始]

## 参考文献

+   Abbasi-Yadkori等人（2011）Abbasi-Yadkori, Y., Pál, D. 和 Szepesvári, C.（2011）。改进的线性随机赌博机算法。发表于《神经信息处理系统进展》。

+   Brown等人（2020）Brown, T. B., Mann, B., Ryder, N., Subbiah, M., Kaplan, J., Dhariwal, P., Neelakantan, A., Shyam, P., Sastry, G., Askell, A.等人（2020）。语言模型是少样本学习者。发表于《NeurIPS 2020 会议录》。

+   Chan等人（2023）Chan, C.-M., Chen, W., Su, Y., Yu, J., Xue, W., Zhang, S., Fu, J. 和 Liu, Z.（2023）。Chateval：通过多代理辩论，朝着更好的基于LLM的评估器迈进。arXiv预印本arXiv:2308.07201。

+   Chen等人（2024a）Chen, D., Huang, Y., Wu, S., Tang, J., Chen, L., Bai, Y., He, Z., Wang, C., Zhou, H., Li, Y.等人（2024a）。Gui-world：一个面向GUI的多模态LLM代理数据集。arXiv预印本arXiv:2406.10819。

+   Chen 等人（2024b）Chen, W., Su, Y., Zuo, J., Yang, C., Yuan, C., Chan, C.-M., Yu, H., Lu, Y., Hung, Y.-H., Qian, C., Qin, Y., Cong, X., Xie, R., Liu, Z., Sun, M. 和 Zhou, J.（2024b）。《Agentverse：促进多智能体协作与探索突现行为》。在第十二届国际学习表示会议上。

+   Chen 等人（2024c）Chen, W., You, Z., Li, R., Guan, Y., Qian, C., Zhao, C., Yang, C., Xie, R., Liu, Z. 和 Sun, M.（2024c）。《智能体互联网：编织异构智能体的协同智能网络》。arXiv 预印本 arXiv:2407.07061。

+   Dai 等人（2024）Dai, J., Lu, J., Feng, Y., Ruan, R., Cheng, M., Tan, H. 和 Guo, Z.（2024）。《MHPP：探索超越基础代码生成的语言模型能力与局限性》。CoRR abs/2405.11430。

+   Devlin 等人（2019）Devlin, J., Chang, M., Lee, K. 和 Toutanova, K.（2019）。《BERT：用于语言理解的深度双向转换器的预训练》。在 NAACL-HLT 会议论文集中。

+   Dubey 等人（2024）Dubey, A., Jauhri, A., Pandey, A., Kadian, A., Al-Dahle, A., Letman, A., Mathur, A., Schelten, A., Yang, A., Fan, A. 等人（2024）。《The llama 3 herd of models》。arXiv 预印本 arXiv:2407.21783。

+   Fan 和 Tao（2024）Fan, X. 和 Tao, C.（2024）。《走向弹性与高效的 LLM：效率、性能与对抗鲁棒性的比较研究》。arXiv 预印本 arXiv:2408.04585。

+   Feng 等人（2024）Feng, P., He, Y., Huang, G., Lin, Y., Zhang, H., Zhang, Y. 和 Li, H.（2024）。《Agile：一种新颖的 LLM 智能体框架》。arXiv 预印本 arXiv:2405.14751。

+   Ge 等人（2024）Ge, J., Jia, X., Viswanathan, V., Luo, H. 和 Neubig, G.（2024）。《通过基于检索的蒸馏训练任务专家》。arXiv 预印本 arXiv:2407.05463。

+   Hao 等人（2023）Hao, S., Gu, Y., Ma, H., Hong, J. J., Wang, Z., Wang, D. Z. 和 Hu, Z.（2023）。《与语言模型推理即是与世界模型规划》。arXiv 预印本 arXiv:2305.14992。

+   He 等人（2024）He, J., Chen, S., Zhang, F. 和 Yang, Z.（2024）。《从词语到行动：揭示 LLM 驱动的自主系统的理论基础》。arXiv 预印本 arXiv:2405.19883。

+   He 等人（2023）He, N., Lai, H., Zhao, C., Cheng, Z., Pan, J., Qin, R., Lu, R., Lu, R., Zhang, Y., Zhao, G. 等人（2023）。《Teacherlm：教人捕鱼而非给人鱼，语言建模亦如此》。arXiv 预印本 arXiv:2310.19019。

+   Hu 等人（2024a）Hu, S., Tu, Y., Han, X., He, C., Cui, G., Long, X., Zheng, Z., Fang, Y., Huang, Y., Zhao, W. 等人（2024a）。《Minicpm：揭示具有可扩展训练策略的小型语言模型的潜力》。arXiv 预印本 arXiv:2404.06395。

+   Hu 等人（2024b）Hu, X., Zhang, F., Chen, S. 和 Yang, Z.（2024b）。《揭示链式思考提示方法的统计基础》。arXiv 预印本 arXiv:2408.14511。

+   Huang 等人（2024a）Huang, J., Pan, J., Wan, Z., Lyu, H. 和 Luo, J.（2024a）。《Evolver：链式演化提示增强大型多模态模型在仇恨表情包检测中的应用》。arXiv 预印本 arXiv:2407.21004。

+   Huang 等人（2023）Huang, L., Yu, W., Ma, W., Zhong, W., Feng, Z., Wang, H., Chen, Q., Peng, W., Feng, X., Qin, B. 等人（2023）。《大型语言模型中的幻觉调查：原理、分类法、挑战与未解问题》。arXiv 预印本 arXiv:2311.05232。

+   Huang 等人（2024b）Huang, X., Liu, W., Chen, X., Wang, X., Wang, H., Lian, D., Wang, Y., Tang, R. 和 Chen, E.（2024b）。《理解大型语言模型代理的规划：一项调查》。arXiv 预印本 arXiv:2402.02716。

+   Huang 和 Sun（2023）Huang, Y. 和 Sun, L.（2023）。《利用 ChatGPT 探索假新闻：生成、检测和解释的深度研究》。arXiv 预印本 arXiv:2310.05046。

+   Jin 等人（2021）Jin, Y., Yang, Z. 和 Wang, Z.（2021）。《悲观主义对于离线强化学习是否证明是有效的？》在国际机器学习会议上。PMLR。

+   Kagaya 等人（2024）Kagaya, T., Yuan, T. J., Lou, Y., Karlekar, J., Pranata, S., Kinose, A., Oguri, K., Wick, F. 和 You, Y.（2024）。《Rap：结合上下文记忆的检索增强规划，应用于多模态大型语言模型代理》。arXiv 预印本 arXiv:2402.03610。

+   Kang 等人（2021）Kang, Y., Xu, Y., Chen, C. P., Li, G. 和 Cheng, Z.（2021）。《6：增强现实中的同步追踪、标签和地图构建》。在 SID 会议技术论文摘要集，卷 52。Wiley 在线图书馆。

+   Kang 等人（2022）Kang, Y., Zhang, Z., Zhao, M., Yang, X. 和 Yang, X.（2022）。《将记忆与电子纪念品绑定：博物馆中的混合型实体增强现实纪念品》。在第35届ACM用户界面软件与技术年会附录会议论文集。

+   Li 等人（2023a）Li, G., Hammoud, H., Itani, H., Khizbullin, D. 和 Ghanem, B.（2023a）。《Camel：用于大型语言模型社会“心智”探索的交互式代理》。神经信息处理系统进展 36 51991–52008。

+   Li 等人（2023b）Li, Z., Zhang, X., Zhang, Y., Long, D., Xie, P. 和 Zhang, M.（2023b）。《朝着通用文本嵌入迈进：多阶段对比学习》。arXiv 预印本 arXiv:2308.03281。

+   Lin 等人（2023）Lin, L., Bai, Y. 和 Mei, S.（2023）。《Transformer 作为决策者：通过监督预训练实现可证明的上下文强化学习》。arXiv 预印本 arXiv:2310.08566。

+   Liu 等人（2023a）Liu, Z., Hu, H., Zhang, S., Guo, H., Ke, S., Liu, B. 和 Wang, Z.（2023a）。《为未来而推理，为现在而行动：一个针对自主大型语言模型代理的原理框架，具备可证明的样本效率》。arXiv 预印本 arXiv:2309.17382。

+   Liu 等人（2023b）Liu, Z., Huang, Y., Yu, X., Zhang, L., Wu, Z., Cao, C., Dai, H., Zhao, L., Li, Y., Shu, P. 等人（2023b）。《Deid-GPT：利用 GPT-4 进行零样本医学文本去标识化》。arXiv 预印本 arXiv:2303.11032。

+   OpenAI（2023）OpenAI（2023）。《GPT-4 技术报告》。CoRR abs/2303.08774。

+   Park 等人 (2024) Park, C., Liu, X., Ozdaglar, A. 和 Zhang, K. (2024). 大型语言模型代理是否存在后悔？在线学习和游戏中的案例研究。arXiv 预印本 arXiv:2403.16843。

+   Qin 等人 (2024) Qin, L., Chen, Q., Feng, X., Wu, Y., Zhang, Y., Li, Y., Li, M., Che, W. 和 Yu, P. S. (2024). 大型语言模型遇到 NLP：一项调查。arXiv 预印本 arXiv:2405.12819。

+   Raparthy 等人 (2023) Raparthy, S. C., Hambro, E., Kirk, R., Henaff, M. 和 Raileanu, R. (2023). 利用上下文学习实现对新顺序决策任务的泛化。arXiv 预印本 arXiv:2312.03801。

+   Scarlatos 和 Lan (2023) Scarlatos, A. 和 Lan, A. (2023). Reticl: 使用强化学习进行上下文示例的顺序检索。arXiv 预印本 arXiv:2305.14502。

+   Shinn 等人 (2024) Shinn, N., Cassano, F., Gopinath, A., Narasimhan, K. 和 Yao, S. (2024). Reflexion: 具备语言强化学习的语言代理。发表于《神经信息处理系统进展》36。

+   Shridhar 等人 (2020) Shridhar, M., Yuan, X., Côté, M.-A., Bisk, Y., Trischler, A. 和 Hausknecht, M. (2020). Alfworld: 将文本与具身环境对齐以进行互动学习。arXiv 预印本 arXiv:2010.03768。

+   Shum 等人 (2023) Shum, K., Diao, S. 和 Zhang, T. (2023). 基于链式思维的自动提示增强与选择，来源于标注数据。发表于《计算语言学协会成果：EMNLP 2023》。

+   Sumers 等人 (2023) Sumers, T. R., Yao, S., Narasimhan, K. 和 Griffiths, T. L. (2023). 语言代理的认知架构。arXiv 预印本 arXiv:2309.02427。

+   Tao 等人 (2024) Tao, C., Fan, X. 和 Yang, Y. (2024). 利用大型语言模型进行 API 交互：一个分类和合成数据生成框架。arXiv 预印本 arXiv:2409.11703。

+   Voronov 等人 (2024) Voronov, A., Wolf, L. 和 Ryabinin, M. (2024). 留意格式：朝着一致的上下文学习改进评估迈进。arXiv 预印本 arXiv:2401.06766。

+   Wan (2023) Wan, Z. (2023). 文本分类：深度学习方法的视角。arXiv 预印本 arXiv:2309.13761。

+   Wang 等人 (2024a) Wang, H., Pan, Y., Sun, F., Liu, S., Talluri, K., Chen, G. 和 Li, X. (2024a). 理解预训练 Transformer 在顺序决策中的训练与泛化。arXiv 预印本 arXiv:2405.14219。

+   Wang 等人 (2023a) Wang, H., Wu, H., Sun, J., Zhang, S., Chen, C., Hua, X.-S. 和 Luo, X. (2023a). Idea: 高效领域适应图像检索的一个不变视角。发表于《神经信息处理系统进展》36 57256–57275。

+   Wang 等人 (2023b) Wang, H., Yang, X., Chang, J., Jin, D., Sun, J., Zhang, S., Luo, X. 和 Tian, Q. (2023b). 大规模多模态基础模型的参数高效调优。发表于《神经信息处理系统进展》36 15752–15774。

+   Wang 等人（2024b）Wang, X., Zhu, W., Saxon, M., Steyvers, M. 和 Wang, W. Y.（2024b）。大型语言模型是潜变量模型：解释和寻找上下文学习的良好示范。《神经信息处理系统进展》36。

+   Wang 等人（2024c）Wang, Z., Bukharin, A., Delalleau, O., Egert, D., Shen, G., Zeng, J., Kuchaiev, O. 和 Dong, Y.（2024c）。Helpsteer2-preference：通过偏好补充评分。

+   Wei 等人（2022）Wei, J., Wang, X., Schuurmans, D., Bosma, M., Xia, F., Chi, E., Le, Q. V., Zhou, D. 等人（2022）。链式思维提示引发大型语言模型中的推理。《神经信息处理系统进展》35 24824–24837。

+   Xiao 等人（2024）Xiao, C., Zhang, Z., Song, C., Jiang, D., Yao, F., Han, X., Wang, X., Wang, S., Huang, Y., Lin, G. 等人（2024）。可配置的基础模型：从模块化视角构建大型语言模型。arXiv 预印本 arXiv:2409.02877。

+   Xiong 等人（2022）Xiong, J., Wan, Z., Hu, X., Yang, M. 和 Li, C.（2022）。通过自洽推理解决数学应用题。arXiv 预印本 arXiv:2210.15373。

+   Yan 等人（2023）Yan, J., Xu, J., Song, C., Wu, C., Li, Y. 和 Zhang, Y.（2023）。从重复中理解上下文学习。arXiv 预印本 arXiv:2310.00297。

+   Yang 等人（2022）Yang, X., Kang, Y. 和 Yang, X.（2022）。为增强虚拟现实中的触觉反馈而重新定向被动道具的目标。2022 年 IEEE 虚拟现实与 3D 用户界面会议摘要与研讨会（VRW）。IEEE。

+   Yang 等人（2018）Yang, Z., Qi, P., Zhang, S., Bengio, Y., Cohen, W. W., Salakhutdinov, R. 和 Manning, C. D.（2018）。Hotpotqa：一个多样化、可解释的多跳问答数据集。arXiv 预印本 arXiv:1809.09600。

+   Yao 等人（2022a）Yao, S., Chen, H., Yang, J. 和 Narasimhan, K.（2022a）。Webshop：朝着可扩展的真实世界网页互动迈进，借助有根据的语言代理。《神经信息处理系统进展》35 20744–20757。

+   Yao 等人（2024）Yao, S., Yu, D., Zhao, J., Shafran, I., Griffiths, T., Cao, Y. 和 Narasimhan, K.（2024）。思想树：通过大型语言模型进行深思熟虑的问题解决。《神经信息处理系统进展》36。

+   Yao 等人（2022b）Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K. 和 Cao, Y.（2022b）。React：在语言模型中协同推理与行动。arXiv 预印本 arXiv:2210.03629。

+   Zeng 等人（2024）Zeng, Z., Liu, Y., Wan, Y., Li, J., Chen, P., Dai, J., Yao, Y., Xu, R., Qi, Z., Zhao, W., Shen, L., Lu, J., Tan, H., Chen, Y., Zhang, H., Shi, Z., Wang, B., Guo, Z. 和 Jia, J.（2024）。MR-BEN：一个全面的元推理基准，用于大型语言模型。CoRR 预印本 abs/2406.13975。

+   Zhao 等人（2024）Zhao, J., Qian, Z., Cao, L., Wang, Y. 和 Ding, Y.（2024）。角色扮演推理中的偏见与毒性。

+   赵和张（2024）赵, J. 和 张, X.（2024）。大语言模型不是一个（多语言）组合关系推理器。在第一次语言建模会议上。

+   郑等人（2023）郑, L., 尹, L., 谢, Z., 黄, J., 孙, C., 余, C. H., 曹, S., Kozyrakis, C., Stoica, I., Gonzalez, J. E. 等（2023）。使用sglang高效编程大语言模型。arXiv预印本 arXiv:2312.07104。

+   周等人（2023）周, A., 严, K., Shlapentokh-Rothman, M., 王, H. 和 王, Y.-X.（2023）。语言代理树搜索统一了语言模型中的推理、行动和规划。arXiv预印本 arXiv:2310.04406。
