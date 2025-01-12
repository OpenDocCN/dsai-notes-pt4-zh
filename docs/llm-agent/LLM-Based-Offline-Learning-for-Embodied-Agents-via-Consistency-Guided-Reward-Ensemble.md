<!--yml

分类：未分类

日期：2025-01-11 11:53:22

-->

# 基于LLM的离线学习用于具身智能体，通过一致性引导奖励集成

> 来源：[https://arxiv.org/html/2411.17135/](https://arxiv.org/html/2411.17135/)

Yujeong Lee¹，Sangwoo Shin¹¹¹脚注标记：1，Wei-Jin Park²，Honguk Woo¹

计算机科学与工程系，成均馆大学¹

Acryl Inc.²

yujlee@skku.edu, jsw7460@skku.edu, jin@acryl.ai, hwoo@skku.edu 同等贡献于此工作通讯作者

###### 摘要

利用大型语言模型（LLMs）使具身智能体成为可能已经变得非常流行，但在实践中它也面临一些限制。在这项工作中，我们不是直接将LLMs作为智能体，而是探索它们作为具身智能体学习工具的使用。具体而言，为了通过离线强化学习（RL）训练独立的智能体，我们使用LLM为训练数据集中的个体行为提供密集的奖励反馈。在此过程中，我们提出了一种一致性引导奖励集成框架（CoREn），旨在解决将LLM生成的估算值与目标环境领域进行对接的难题。该框架利用时空一致的自适应奖励集成，在训练数据集中推导出领域基础的奖励，从而实现不同环境领域中具身智能体的有效离线学习。通过在VirtualHome基准测试中的实验表明，CoREn显著优于其他离线RL智能体，而且尽管CoREn的智能体策略网络只有117M参数，并且仅在训练中使用LLMs，它的表现与拥有8B参数的最先进LLM基础智能体相当。

基于LLM的离线学习用于具身智能体，通过一致性引导奖励集成

Yujeong Lee¹^†^†感谢：同等贡献于此工作的Sangwoo Shin¹¹¹脚注标记：1，Wei-Jin Park²，Honguk Woo¹^†^†感谢：通讯作者，计算机科学与工程系，成均馆大学¹ Acryl Inc.² yujlee@skku.edu, jsw7460@skku.edu, jin@acryl.ai, hwoo@skku.edu

## 1 引言

开发能够理解用户指令并在物理环境中执行任务的具身智能体，是追求通用人工智能的一个重要里程碑。最近在大语言模型（LLMs）方面的进展展示了它们卓越的推理能力，为它们在具身智能体中的应用铺平了道路，Yang 等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib34)）；Padmakumar 等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib19)）；Pantazopoulos 等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib20)）；Yun 等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib35)）；Logeswaran 等人（[2022](https://arxiv.org/html/2411.17135v1#bib.bib16)）；Ichter 等人（[2022](https://arxiv.org/html/2411.17135v1#bib.bib9)）均已证实了这一点。然而，直接将大语言模型作为具身智能体的一部分进行部署会带来若干低效问题，例如需要复杂的环境特定提示设计、巨大的计算资源需求和固有的模型推理延迟，Hashemzadeh 等人（[2024](https://arxiv.org/html/2411.17135v1#bib.bib7)）也指出了这些问题。这些因素可能限制大语言模型的实际应用，尤其是在需要具身智能体快速而高效响应的场景中。

在强化学习（RL）文献中，数据中心的离线学习方法已被探索过，Kumar 等人（[2020a](https://arxiv.org/html/2411.17135v1#bib.bib11)）提出了这些方法。这些离线 RL 方法旨在建立高效的智能体结构，需要包括带有奖励信息的充分标注的智能体轨迹数据集。然而，分配给具身智能体的指令跟随任务的特性，特别是它们长时间跨度的目标达成特性，常常与离线 RL 的密集数据需求相冲突。具身智能体通常可以生成带有稀疏奖励反馈的轨迹，因为它们的指令跟随任务是根据成功或失败的二元结果来评估的，这些结果与指令的具体目标直接对齐。在离线 RL 中，这种稀疏奖励设置在实现有效的智能体策略时带来了重大挑战，Park 等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib21)）；Ma 等人（[2022](https://arxiv.org/html/2411.17135v1#bib.bib18)）也指出了这一问题。

在这项工作中，我们探索了LLM在离线强化学习（RL）中的应用。通过使用强大的LLM作为奖励估计器，为智能体的行为提供即时反馈，我们通过密集的奖励信息来增强轨迹数据集。该方法，即基于LLM的奖励估计，能够显著提高离线强化学习在具身智能体中的效果。为此，我们解决了基于LLM的奖励估计所固有的局限性。一个主要挑战来自于离线设置中与环境的有限交互，这使得LLM难以获得必要的环境知识。离线设置使得确保生成的奖励与特定环境领域相一致变得困难。

![参见说明](img/317ef602e3d5156874badfe7c2e4526e.png)

图1：CoREn，一个基于LLM的奖励估计和离线学习框架。在（i）中，LLM基于时空（即上下文、结构和时间）一致性来估计奖励；在（ii）中，这些奖励通过集成被整合成一个单一的领域基础奖励。使用增强奖励的数据集，离线强化学习可以有效地进行，从而实现具身智能体的资源效率和低延迟。

例如，如果没有明确的知识表明花盆通常存放在目标环境中的客厅中，LLM可能难以准确地为“去客厅”和“去阳台”等行为分配奖励，尤其是在被要求浇水的任务中。虽然从常识角度来看，这两个动作都可能是合理的，但最佳行动取决于目标环境的具体条件，而这些条件LLM在离线设置中可能无法获取。

这些挑战是离线情境特有的，区分了我们的工作与以往关于在线LLM奖励估计的研究，在线研究中，LLM可以通过与环境或人类的反复交互进行微调或优化提示（Lee ([2024](https://arxiv.org/html/2411.17135v1#bib.bib14)); Xie等人 ([2024](https://arxiv.org/html/2411.17135v1#bib.bib33)); Li等人 ([2023](https://arxiv.org/html/2411.17135v1#bib.bib15)); Song等人 ([2023c](https://arxiv.org/html/2411.17135v1#bib.bib29))）。由于在离线设置中无法进行这些交互，提升LLM在准确奖励估计中缺乏的空间推理能力需要一种根本不同的方法。

为此，我们提出了CoREn，一个一致性引导的奖励集成框架，专为稳健的基于LLM的奖励估计和有效的智能体离线学习设计。它采用了一个两阶段奖励估计过程，如图[1](https://arxiv.org/html/2411.17135v1#S1.F1 "图1 ‣ 1 引言 ‣ 基于LLM的具身智能体离线学习通过一致性引导的奖励集成")所示。（i）首先，查询LLM以估计多个奖励类型，每种类型都考虑LLM的不同时空一致性标准，从而获得一致且有领域依据的奖励。（ii）然后，这些奖励被进一步协调，通过与给定轨迹的稀疏奖励进行对齐过程，统一为特定领域调优的奖励。通过离线强化学习（RL）训练的结果智能体，能够以高效且低延迟的方式执行指令跟随任务。这种增强了LLM奖励估计的离线RL方案，克服了依赖LLM在线利用的智能体所面临的局限性。

我们工作的贡献可以总结如下：（i）首次使用大型语言模型（LLMs）解决具身智能体离线学习的一个实际而具有挑战性的问题；（ii）提出了一种由时空一致性集成引导的两阶段奖励估计算法；以及（iii）在VirtualHome基准上进行了广泛评估，展示了与最先进的基于LLM的在线智能体相媲美的性能。

## 2 预备知识

### 2.1 目标-部分可观察马尔可夫决策过程（Goal-POMDP）

对于遵循用户指定指令的具身智能体，我们将其环境建模为目标条件部分可观察马尔可夫决策过程（Goal-POMDP）。一个Goal-POMDP由一个元组（$\mathcal{S}$，$\mathcal{A}$，$P$，$R$，$\gamma$，$\Omega$，$\mathcal{O}$，$\mathcal{G}$）表示，参见Song等人（[2023a](https://arxiv.org/html/2411.17135v1#bib.bib27)）；Singh等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib26)）的研究，其中状态$s \in \mathcal{S}$，动作$a \in \mathcal{A}$，转移函数$P:\mathcal{S} \times \mathcal{A} \longrightarrow \Delta(\mathcal{S})$，奖励函数$R:\mathcal{S} \times \mathcal{A} \times \mathcal{G}^{\omega} \mapsto \mathbb{R}$¹¹1$X^{\omega}$对于集合$X$来说，是所有可能的有限乘积；折扣因子$\gamma \in [0,1)$，观察$o \in \Omega$，观察转移函数$\mathcal{O}:\mathcal{S} \times \mathcal{A} \longrightarrow \Omega$，目标条件$G \in \mathcal{G}$。给定这个Goal-POMDP表示，我们将用户指定的指令$i$视为一系列目标条件$\mathbf{G}=(G_{1},\cdots) \subseteq \mathcal{G}$，使得具身智能体的任务是完成指令$i$中每个指定的目标条件。

![参见标题](img/dc6370aec4498298e213a4c45f076f77.png)

图2：CoREn中的两阶段奖励估计。在（i）中，通过上下文、结构和时间一致性约束计算空间-时间一致的奖励。 (a) 上下文一致性通过对不同提示$\mathcal{P}_{n}$的响应进行多数投票来实现，从而生成上下文一致的奖励$r^{C}$。 (b) 结构一致性通过向LLM提出MDP特定的查询来实现。如果LLM未能正确回答这些查询（由红色‘X’标示），则从多数投票中去除这些特定提示估计的奖励。成功验证的奖励贡献于结构一致的奖励$r^{S}$。 (c) 时间一致性涉及收集高价值动作$H_{n}(\tau)$，并通过LLM查询进行反向验证。未通过验证的动作将从多数投票候选中排除，否则，它们将贡献于时间一致的奖励$r^{T}$。在（ii）中，从给定的离线数据集$\mathcal{D}$中采样一个轨迹$(i,\tau)$，其成功标志为$f_{s}(i,\tau)$。在（i）中，空间-时间一致的奖励$(r^{C},r^{S},r^{T})$通过由奖励协调器$\Psi_{\theta}$生成的权重$(w^{C},w^{S},w^{T})$进行组合。该组合结果生成一个统一的逐步奖励$\hat{r}$，并与轨迹上的稀疏奖励$f_{s}(i,\tau)$进行对齐。奖励协调器$\Psi_{\theta}$的训练目标是使轨迹的累计逐步奖励$\hat{r}$与轨迹上标注的稀疏奖励$f_{s}(i,\tau)$一致。

### 2.2 离线RL

对于Goal-POMDP，其最优策略可以通过以下方式公式化：

|  | $\pi^{*}=\operatorname*{argmax}_{\pi}\underset{\begin{subarray}{c}(s,a)\sim\pi,% \\ \mathbf{G}\sim\mathcal{G}\end{subarray}}{\mathbb{E}}\left[\sum_{t}\gamma^{t}R(% s,a,\mathbf{G})\right].$ |  | (1) |
| --- | --- | --- | --- |

为了实现最优策略，我们探索了离线RL方法，其中策略通过优化贝尔曼误差目标来推导，完全依赖于离线数据集$\mathcal{D}$，而不进行任何环境交互。离线RL对具身智能体尤其有益，因为它减少了在物理环境中进行主动探索时所伴随的风险和成本。我们使用数据集$\mathcal{D}=\{(i_{j},\tau_{j}):j\}$，其中$\tau_{j}$是与指令$i_{j}$对应的轨迹。与传统的离线RL不同，该数据集$\mathcal{D}$包含稀疏奖励。这种稀疏性体现在一部分轨迹中，这些轨迹由成功标志$f_{s}(i_{j},\tau_{j})$标记，表示$\tau_{j}$是否满足了指令$i_{j}$的所有必要目标条件。这种稀疏奖励设置在具身指令跟随任务中是固有的，因为每个指令都被视为Goal-POMDPs中的一系列目标条件。

## 3 我们的方法

基于LLM的奖励估计。离线强化学习（RL）使代理能够在没有直接与环境交互的情况下进行学习，但仅依靠稀疏奖励来学习长时程指令跟随任务通常效率较低。为了解决这个问题，我们通过基于LLM的估计，使用逐步内在奖励来增强代理的轨迹。类似于基于LLM的任务规划（Singh et al. [2023](https://arxiv.org/html/2411.17135v1#bib.bib26)；Ichter et al. [2022](https://arxiv.org/html/2411.17135v1#bib.bib9)），LLM可以用于逼近数据集中观察-动作对的奖励，从而提供更直接、可操作的密集反馈，以提高离线学习的有效性。

非实地奖励估计。LLM在中间步骤估计的内在奖励可能与在各个指令跟随任务结尾提供的稀疏奖励不一致。当内在奖励未能充分与环境领域对接时，就会出现这种差异。在部分可观察设置中，由于LLM被迫基于不完整的环境快照推断奖励，这个问题会更加严重。

### 3.1 整体框架

为了应对基于LLM的奖励估计的局限性，我们提出了一种时空一致性引导的奖励集成框架CoREn，该框架采用两阶段过程。如图[2](https://arxiv.org/html/2411.17135v1#S2.F2 "Figure 2 ‣ 2.1 Goal-POMDPs ‣ 2 Preliminaries ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")所示，第一阶段(i)结合了上下文、一致性结构和时间一致性，以充分发挥LLM的推理能力，并增强奖励估计在特定领域环境中的实地基础性。在第二阶段(ii)，CoREn基于轨迹成功情况，协调第一阶段生成的不同奖励的集成。这使得能够推导出特定领域调优的奖励，这些奖励可以有效地用于具身体代理的离线学习。

### 3.2 时空一致性奖励

对于奖励估计，我们使用$N$个不同的提示$\mathcal{P}_{1},\cdots,\mathcal{P}_{N}$与LLM（$\Phi_{\textnormal{LLM}}$）结合，其中每个提示通过其独特的解释、上下文示范以及链式思维（CoT）的使用来区分。具体来说，每个提示$\mathcal{P}_{n}$结合观察$o$、动作$l$和指令$i$，通过$\Phi_{\textnormal{LLM}}$推理生成奖励$R_{n}$。

|  | $R_{n}(o,l&#124;i)=\Phi_{\textnormal{LLM}}(P_{n},(o,l,i))$ |  | (2) |
| --- | --- | --- | --- |

空间一致性旨在确保基于领域的LLM奖励估计在不同提示引导的上下文中保持一致，并且基于对环境结构的全面理解。我们通过实现两种一致性机制来解决这一问题。

上下文一致性。该机制旨在减少由 LLM 基于奖励估计时所使用的特定提示上下文引起的偏差。通过使用多个 $N$ 个提示，每个提示具有不同的上下文框架，我们确保在这些变化中保持一致的奖励，反映出推理中的共识。对于上下文一致的奖励 $r_{C}$，我们通过以下方式整合提示 $\mathcal{P}_{n}$ 的响应 $R_{n}^{C}(o,l|i)$：

|  | $r^{C}(o,l&#124;i)=\underset{r\in\hat{\mathbb{R}}}{\text{argmax}}\sum\limits_{n=1}^{% N}\mathbbm{1}_{(R_{n}^{C}(o,l&#124;i)=r)}\\$ |  | (3) |
| --- | --- | --- | --- |

其中 $R_{n}^{C}(o,l|i)=\Phi_{\textnormal{LLM}}(\mathcal{P}_{n},(o,l,i))$。

结构一致性。这个设计旨在确保奖励估计纳入对环境物理结构的全面理解，例如物体、它们的关系以及它们与给定指令的相关性。我们通过 MDP 特定查询 $q(o)$ 向 $\Phi_{\textnormal{LLM}}$ 查询与观察 $o$ 相关的问题，例如“在 $o$ 中哪些物体与指令 $i$ 相关？”。利用对这些查询的响应 $\Phi_{\textnormal{LLM}}(\mathcal{P}_{n},q(o))$，我们整合提示 $\mathcal{P}_{n}$ 的奖励 $R_{n}^{S}(o,l|i)$：

|  | $r^{S}(o,l&#124;i)=\underset{r\in\hat{\mathbb{R}}}{\text{argmax}}\sum_{n=1}^{N}% \mathbbm{1}_{(R_{n}^{S}(o,l&#124;i)=r)}.$ |  | (4) |
| --- | --- | --- | --- |

我们重写了方程 ([2](https://arxiv.org/html/2411.17135v1#S3.E2 "在 3.2 时空一致奖励 ‣ 3 我们的方法 ‣ 基于一致性引导的奖励集成进行 LLM 离线学习"))，用于查询违规情况，得到 $R_{n}^{S}(o,l|i)$。

|  |  $=\begin{cases}\varnothing&a(o)\neq\Phi_{\textnormal{LLM}}(\mathcal{P}_{n},q(o)% )\\ \Phi_{\textnormal{LLM}}(\mathcal{P}_{n},(o,l,i))&\textnormal{否则}.\end{cases}$  |  | (5) |
| --- | --- | --- | --- |

关于提示 $\mathcal{P}_{n}$ 和 MDP 特定查询与答案的数据集构建 $\mathcal{D}_{\textnormal{QA}}=\{(q(o),a(o)):o\in\tau\in\mathcal{D}\}$ 的细节，请参见附录。

时间一致性。这个设计旨在确保赋予某一动作的值在整个决策过程中保持一致性。通过时间一致性，如果 LLM 的前向推理评估某些动作具有较高的值，那么回溯验证必须确认这些高价值的动作能够共同完成给定的指令。

为了实现这种反向验证，我们向 $\Phi_{\textnormal{LLM}}$ 提出查询 $q(i,\tau,n)$：“从观测 $o$ 中执行高价值动作 $H_{n}(\tau)$ 是否合理以完成指令 $i$？” 然后，奖励取决于对这个查询的响应 $\Phi_{\textnormal{LLM}}(q(i,\tau,n))\in\{\textnormal{True},\textnormal{False}\}$，公式 [2](https://arxiv.org/html/2411.17135v1#S3.E2 "在 3.2 空间-时间一致奖励 ‣ 3 我们的方法 ‣ 基于一致性指导奖励集成的LLM离线学习") 被重写为 $R_{n}^{T}(o,l|i)$。

|  |  $\begin{split}=\begin{cases}\varnothing&l\in H_{n}(\tau)\wedge\neg\Phi_{% \textnormal{LLM}}(q(i,\tau,n))\\ \Phi_{\textnormal{LLM}}(\mathcal{P}_{n},(o,l,i))&\textnormal{否则}\end{% cases}\end{split}$  |  | (6) |
| --- | --- | --- | --- |

对于查询违反的情况，即 $l\in H_{n}(\tau)\wedge\neg\Phi_{\textnormal{LLM}}(q(i,\tau,n))$。这里，对于所有轨迹观测 $o\in\tau$，高价值动作定义为

|  | $H_{n}(\tau)=\{\underset{l}{\text{argmax}}\,\,\Phi_{\textnormal{LLM}}(\mathcal{% P}_{n},(o,l,i))\}.$ |  | (7) |
| --- | --- | --- | --- |

给定 $N$ 个提示，我们通过采用多数投票法来整合每个奖励（见公式 [6](https://arxiv.org/html/2411.17135v1#S3.E6 "在 3.2 空间-时间一致奖励 ‣ 3 我们的方法 ‣ 基于一致性指导奖励集成的LLM离线学习")），以建立时间一致的奖励。

|  | $r^{T}(o,l&#124;i)=\underset{r\in\hat{\mathbb{R}}}{\text{argmax}}\sum_{n=1}^{N}% \mathbbm{1}_{(R_{n}^{T}(o,l&#124;i)=r)}$ |  | (8) |
| --- | --- | --- | --- |

### 3.3 一个领域根植的奖励集成

从上述计算得到的空间-时间一致奖励 $r^{C}$、$r^{S}$ 和 $r^{T}$ 中，我们通过根据与给定的离线轨迹对齐的集成方法导出领域根植的奖励。我们将统一奖励 $\hat{r}$ 建模为

|  | $\begin{array}[]{l}\mathbf{r}(o,l&#124;i)=(r^{C}(o,l&#124;i),r^{S}(o,l&#124;i),r^{T}(o,l&#124;i))\\ \mathbf{w}(o,l&#124;i)=(w^{C}(o,l&#124;i),w^{S}(o,l&#124;i),w^{T}(o,l&#124;i))\\ \hat{r}(o,l&#124;i)=\langle\mathbf{r}(o,l&#124;i),\mathbf{w}(o,l&#124;i)\rangle\end{array}$ |  | (9) |
| --- | --- | --- | --- |

其中 $\langle\cdot,\cdot\rangle$ 是内积，$w^{C},w^{S}$ 和 $w^{T}$ 是可学习的权重。这些 $\mathbf{w}$ 由奖励协调器 $\Psi_{\theta}$ 生成。它以观测 $o$、动作 $l$ 和指令 $i$ 为输入，生成一个 softmax 分布用于 $\mathbf{w}$。奖励协调器 $\Psi_{\theta}$ 用于将轨迹的预测回报与标记的回报对齐，即稀疏奖励 $f_{s}(i,\tau)$：

|  |  $\begin{array}[]{l}\mathbf{w}(o_{t},l_{t}&#124;i)=\Psi_{\theta}(o_{t},l_{t},i)\\ \mathcal{L}(\Psi_{\theta})=\underset{\begin{subarray}{c}(i,o_{t},l_{t})\sim\\ (i,\tau)\in\mathcal{D}\end{subarray}}{\mathbb{E}}\bigg{[}\&#124;\sum\limits_{t}% \gamma^{t}\hat{r}(o_{t},l_{t}&#124;i)-\alpha f_{s}(i,\tau)\&#124;^{2}\bigg{]}\end{array}$  |  | (10) |
| --- | --- | --- | --- |

其中$\alpha$是一个超参数。

最终，使用包含统一奖励$\hat{r}$的增强轨迹数据集（见公式（[9](https://arxiv.org/html/2411.17135v1#S3.E9 "In 3.3 A Domain-Grounded Reward Ensemble ‣ 3 Our Approach ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")）），一个智能体可以通过离线强化学习算法进行训练，如CQL Kumar等人（[2020b](https://arxiv.org/html/2411.17135v1#bib.bib12)）。CoREn中的两阶段奖励估计在算法[1](https://arxiv.org/html/2411.17135v1#alg1 "In 4.1 Experiment Settings ‣ 4 Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")中进行了概述。

## 4 实验

### 4.1 实验设置

环境与数据集。为了评估，我们使用了VirtualHome (VH)Puig等人（[2018](https://arxiv.org/html/2411.17135v1#bib.bib22)）提出的广泛使用的家务活动的真实基准。VH包含了多种交互式物体（例如苹果、沙发）和基本行为（例如抓取、坐下），使我们能够为具身智能体定义58种不同的动作。我们使用了25个不同的任务，包括如坐在沙发上并摆放若干水果、微波加热三文鱼以及整理浴室柜台等活动。为了构建离线强化学习的训练数据集$\mathcal{D}$，我们从这25个任务中的每个任务开始，使用一个专家轨迹。然后，我们在每个轨迹的中间步骤加入随机动作，导致失败的尝试。对于每个专家轨迹，注释一个稀疏奖励$1$表示成功，而对于采样的失败轨迹，注释一个稀疏奖励$0$表示失败。这遵循在长时间跨度的指令跟随任务中使用的Goal-POMDP表示方法。

算法 1 双阶段 CoREn

评估指令。我们采用两种不同类型的指令来评估智能体处理不同目标表示的能力。细粒度指令类型提供详细的任务描述，通常包括为实现与指令跟随任务相关的某些目标条件而执行的具体操作。抽象指令类型提供更简洁和概括的任务描述，关注更广泛的目标，而不详细说明每个操作。每个任务都使用 5 个细粒度指令和 5 个抽象指令进行评估，共测试 250 个不同的指令。这些指令未包含在离线训练数据集中。

评估指标。我们使用三项指标，保持与之前的工作一致Singh 等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib26)）；Song 等人（[2023b](https://arxiv.org/html/2411.17135v1#bib.bib28)）。SR 衡量成功完成任务的百分比，成功定义为完成任务的所有目标条件；CGC 衡量完成的目标条件的百分比；Plan 衡量行动序列与从头开始的真实序列连续匹配的百分比。

基准对比。我们将 CoREn 与不同类别的智能体进行比较：RL 智能体，其中 LLM 仅用于估算奖励，以训练一个独立的 RL 智能体，而不直接使用 LLM 进行在线交互；LM 智能体，其中使用小型语言模型（sLM）或 LLM 直接与环境进行在线交互。这与仅将 LLM 用于智能体训练的 RL 智能体形成对比。在 LM 智能体类别中，为了在与 RL 智能体类别兼容的计算效率条件下提供评估，我们包括了基于 sLM 的智能体和基于 LLM 的智能体。

RL 智能体类别的基准包括：i) Lafite-RLChu 等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib3)），使用 LLM 评估动作的好坏（1 表示好，0 表示中性，-1 表示坏），并将评估结果与环境奖励相结合；ii) RDLM Kwon 等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib13)），使用 LLM 通过动态采样的上下文示范来评估轨迹回报；iii) 自一致性 Wang 等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib31)），通过单一 CoT 提示生成多个奖励候选，并对其进行多数投票；iv) GCRL，依赖于与目标条件相关的稀疏奖励。

LM代理类别基准包括v) SayCanIchter等人（[2022](https://arxiv.org/html/2411.17135v1#bib.bib9)），该方法使用离线数据集来学习与LM预测结合的可用性评分；vi) LLM-Planner Song等人（[2023b](https://arxiv.org/html/2411.17135v1#bib.bib28)），该方法使用专家数据集进行检索增强的任务规划；vii) ProgPrompt Singh等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib26)），该方法使用工程化的程序声明语法来验证行动执行的前提条件。

每个LM代理基准都配置了sLMs（GPT2-774M和4位量化的LLaMA3-8B）和LLMs（Gemini 1.0 Pro和LLaMA3-8B）。使用更大LLaMA3-70B模型的LM代理基准的实现可以在附录[D.1](https://arxiv.org/html/2411.17135v1#A4.SS1 "D.1 LLaMA3-70B for LM Agents ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")中找到。

对于我们的CoREn和RL代理类别，我们使用Gemini 1.0 Pro作为奖励估算器$\Phi_{\textnormal{LLM}}$，并采用具有117M参数的GPT2基础模型架构来实现从各自奖励中学习的代理策略。我们还采用了CQLKumar等人（[2020b](https://arxiv.org/html/2411.17135v1#bib.bib12)）的离线RL算法，并结合DDQNvan Hasselt等人（[2016](https://arxiv.org/html/2411.17135v1#bib.bib30)）的方法来处理我们环境中的离散动作空间。实验的详细信息请参见附录。

### 4.2 主要结果

指令跟随任务表现。表[1](https://arxiv.org/html/2411.17135v1#S4.T1 "Table 1 ‣ 4.2 Main Results ‣ 4 Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")展示了我们的CoREn与来自不同类别基准的性能比较，包括RL代理、LLM基础代理、sLM基础代理，在如SR、CGC和Plan等指标上的表现。

+   •

    CoREn在SR、CGC和Plan方面分别超越了最具竞争力的RL代理基准Self-Consistency，取得了$20.0\%$、$15.2\%$和$5.6\%$的平均增益，显著超过了所有RL代理基准。

+   •

    此外，CoREn的表现与LLM基础的代理相当，仅与SayCan-Gemini和ProgPrompt-Gemini相比有轻微的性能下降，而超过了其他LLM基础代理（即，所有使用LLaMA3和LLM-Planner-Gemini的代理）。考虑到CoREn（基于GPT2的117M）与其他LLM基础代理（如Gemini、LLaMA-8B）之间模型规模的显著差异，这些结果尤其值得注意。这些结果展示了CoREn在特定领域中使用最小的领域特定知识（例如部分标注奖励）来学习长时间跨度的指令跟随任务的能力。

+   •

    我们观察到，使用 4-bit 量化 LLaMA3（LLaMA3Q）和 GPT2 的基于 sLM 的智能体表现较差，相较于其他智能体，包括我们的 CoREn，原因在于它们依赖于 sLM 的有限推理能力。

+   •

    此外，CoREn 相较于基于 LLM 的智能体，展示了在不同类型指令下较为稳健的表现。这可以归因于 CoREn 能够从 LLM 生成的、语义上相似的广泛指令中学习，这些指令被包含在离线数据集中。这使得该框架能够更好地推广到抽象指令。

细粒度抽象 RL 智能体 SR CGC Plan SR CGC Plan CoREn 66.4 74.5 69.5 57.6 68.3 64.8 Lafite-RL 30.4 50.9 35.1 17.6 37.8 23.1 RDLM 20.0 42.0 31.7 4.0 23.3 23.9 自一致性 43.2 56.9 61.4 40.8 55.6 61.7 GCRL 5.6 26.0 22.3 8.0 28.4 17.1 基于 LLM 的智能体 SayCan-Gemini 72.0 78.2 73.8 6.9 25.2 23.2 SayCan-LLaMA3 4.8 22.4 63.8 3.2 14.6 20.0 ProgPrompt-Gemini 72.8 80.4 80.2 32.0 49.2 24.3 ProgPrompt-LLaMA3 68.0 74.5 50.5 16.5 29.5 8.2 LLM-Planner-Gemini 55.2 63.8 59.7 2.1 18.1 0.0 LLM-Planner-LLaMA3 15.1 34.0 30.6 2.0 15.4 0.6 基于 sLM 的智能体 SayCan-LLaMA3Q 4.8 21.6 62.6 0.0 15.4 0.4 SayCan-GPT2 0.0 14.7 0.0 0.0 14.7 0.0 ProgPrompt-LLaMA3Q 43.2 68.2 68.8 15.2 34.5 31.1 ProgPrompt-GPT2 0.6 16.7 6.0 0.0 8.8 0.4 LLM-Planner-LLaMA3Q 12.4 31.1 8.9 0.6 13.9 0.2 LLM-Planner-GPT2 0.0 12.6 0.0 0.0 12.6 0.0

表格1：SR、CGC 和 Plan 指标中的指令跟随任务表现。智能体策略模型的规模：RL 智能体（117M）、基于 LLM 的智能体（Gemini 和 LLaMA3-8B）、基于 sLM 的智能体（GPT2-774M 和 4bit-量化 LLaMA3-8B）。

跨领域表现。在这里，我们扩展了我们的评估场景，以包括环境中的领域变化；即，与给定指令相关的关键对象的位置与训练数据集中的位置不同。具体来说，我们从训练数据集$\mathcal{D}$中抽取一部分轨迹，并重新标记它们的稀疏奖励$f_{s}(i,\tau)$，以反映改变后的对象位置。在保持时空一致的奖励不变的同时，我们使用这些新标记的稀疏奖励重新训练公式中的奖励调度器（[10](https://arxiv.org/html/2411.17135v1#S3.E10 "In 3.3 A Domain-Grounded Reward Ensemble ‣ 3 Our Approach ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")）。该方法便于为强化学习生成特定领域的统一奖励，而无需通过LLM推理重新计算基于一致性的奖励。我们还将这一新标记的数据集纳入了LM代理类别。例如，LLM-Planner通过使用这些重新标记为成功的轨迹，作为任务规划的示范，适应了这一新的环境领域。由于除了GCRL外，其他强化学习代理基准缺乏利用表示为稀疏奖励的领域信息的能力，它们的评估使用与单领域实验相同的策略。

细粒度抽象RL代理 SR CGC 计划 SR CGC 计划 CoREn 60.0 66.3 69.4 45.0 55.0 42.5 Lafite-RL 2.5 12.5 10.6 0.0 12.5 25.2 RDLM 15.0 23.8 22.5 3.8 18.8 23.6 自一致性 35.4 47.9 54.7 31.3 45.8 51.6 GCRL 0.0 6.3 8.5 0.0 6.3 10.9 基于LLM的代理 SayCan-Gemini 12.5 18.8 26.6 0.0 8.3 0.0 SayCan-LLaMA3 5.0 21.3 1.9 0.0 10.0 1.3 ProgPrompt-Gemini 25.0 31.3 23.4 14.6 32.3 1.6 ProgPrompt-LLaMA3 25.0 32.3 22.6 8.3 24.0 3.1 LLM-Planner-Gemini 45.8 55.2 67.7 0.0 13.7 0.0 LLM-Planner-LLaMA3 6.3 20.2 16.8 0.0 40.6 0.0 基于sLM的代理 SayCan-LLaMA3-Q 8.3 21.9 1.5 0.0 12.5 1.9 SayCan-GPT2 0.0 6.3 0.0 0.0 6.3 0.0 ProgPrompt-LLaMA3-Q 7.5 15.5 18.3 0.0 31.3 0.0 ProgPrompt-GPT2 0.0 8.3 0.3 0.0 6.3 0.0 LLM-Planner-LLaMA3-Q 2.1 15.2 9.9 0.0 13.5 0.0 LLM-Planner-GPT2 0.0 6.3 0.0 0.0 6.3 0.0

表 2: 跨领域表现

+   •

    在这一跨领域评估中，如表[2](https://arxiv.org/html/2411.17135v1#S4.T2 "Table 2 ‣ 4.2 Main Results ‣ 4 Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")所示，CoREn优于所有强化学习代理基准，且与单一领域实验结果（见表[1](https://arxiv.org/html/2411.17135v1#S4.T1 "Table 1 ‣ 4.2 Main Results ‣ 4 Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")）相比，下降幅度最小。在领域转变时，CoREn的两阶段过程通过第二阶段的自适应集成调整奖励估计，使其与目标领域对齐，如公式([10](https://arxiv.org/html/2411.17135v1#S3.E10 "In 3.3 A Domain-Grounded Reward Ensemble ‣ 3 Our Approach ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble"))所述。相比之下，依赖于仅通过LLM常识推理衍生的奖励的强化学习代理基准，在适应特定领域方面表现出较弱的能力，相较于单领域实验结果表现出大幅下降。

+   •

    我们还观察到，基于LLM的代理基准在这一跨领域评估中表现出显著退化；例如，LLM-Planner依赖于LLM的知识，而这种知识在特定环境中很难仅通过少量示例进行定位，从而导致性能不佳。

### 4.3 消融研究

时空一致性的奖励。为了验证上下文、一致性结构和时间一致性（在第[3.2节](https://arxiv.org/html/2411.17135v1#S3.SS2 "3.2 Spatio-Temporally Consistent Rewards ‣ 3 Our Approach ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")）在基于LLM的奖励估计中是否能有效互补，我们测试了这些一致性在奖励集成中的不同组合。表[3](https://arxiv.org/html/2411.17135v1#S4.T3 "Table 3 ‣ 4.3 Ablation Studies ‣ 4 Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")显示，利用所有三者的CoREn方法始终优于其他方法。这表明，仅依赖部分一致性衍生的$\mathbf{w}$和奖励在生成统一的、有助于强化学习的奖励方面是有限的，而集成权重$\mathbf{w}$可以通过公式([10](https://arxiv.org/html/2411.17135v1#S3.E10 "In 3.3 A Domain-Grounded Reward Ensemble ‣ 3 Our Approach ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble"))进行调整。

精细化抽象SR CGC计划 SR CGC计划 CoREn 66.4 74.5 69.5 57.6 68.3 64.8 CS 64.8 67.9 69.7 52.0 62.3 56.3 ST 57.6 70.1 65.4 50.4 60.8 61.7 CT 53.6 66.1 67.8 51.2 59.1 68.9 C 47.2 58.6 59.7 47.1 57.1 58.3 S 52.0 67.1 57.9 45.6 60.0 58.9 T 45.6 57.8 55.7 41.6 51.2 50.8

表 3：时空一致性奖励的消融研究。例如，CS表示使用上下文和结构一致性，而T表示仅使用时间一致性，而CoREn在集成中使用了所有三种一致性。

用于奖励估计的不同LLM。为了实现CoREn，该方法使用LLM进行离线奖励估计，我们测试了多种LLM，从开源的LLaMA3-8B到专有模型GPT4 turbo、Gemini 1.0 Pro和PaLM。在表[4](https://arxiv.org/html/2411.17135v1#S4.T4 "表 4 ‣ 4.3 消融研究 ‣ 4 实验 ‣ 基于一致性引导的奖励集成的具身智能体离线学习")中，我们观察到，LLaMA3-8B虽然参数较少，但并未达到与专有模型相当的性能。在专有模型中，较新且更强大的LLM，如GPT4 turbo和Gemini 1.0 Pro，展示了在奖励估计中的强大能力，显著促进了智能体的离线学习。

精细粒度 摘要 SR CGC 计划 SR CGC 计划 LLaMA3 12.0 28.7 39.4 9.6 27.9 40.1 PaLM 16.8 32.9 35.8 10.4 27.7 25.4 GPT4 turbo 65.6 71.8 70.6 40.8 50.5 52.5 Gemini 1.0 Pro 66.4 74.5 69.5 57.6 68.3 64.8

表 4：用于奖励估计的不同LLM

奖励集成方案。我们评估了几种作为奖励集成方案替代的方法，如公式([9](https://arxiv.org/html/2411.17135v1#S3.E9 "在 3.3 基于领域的奖励集成 ‣ 3 我们的方法 ‣ 基于一致性引导的奖励集成的具身智能体离线学习"))中所示。首先，我们考虑将奖励$r^{C}$、$r^{S}$和$r^{T}$的平均值作为统一的奖励。其次，我们采用对这三个奖励进行多数投票机制。如表[5](https://arxiv.org/html/2411.17135v1#S4.T5 "表 5 ‣ 4.3 消融研究 ‣ 4 实验 ‣ 基于一致性引导的奖励集成的具身智能体离线学习")所示，平均值和多数投票相较于CoREn的表现有所下降。虽然时空一致性奖励的多数投票可以提供相当程度的领域基础性，CoREn通过使用稀疏奖励作为引导，进一步改进了奖励集成过程。

精细粒度 摘要 SR CGC 计划 SR CGC 计划 CoREn 66.4 74.5 69.5 57.6 68.3 64.8 平均 53.6 63.7 55.6 43.2 55.2 57.7 Maj.Voting 60.8 70.2 68.9 55.2 62.7 63.7

表 5：奖励集成方案的消融研究

## 5 相关工作

LLMs在具身环境中的应用。将LLMs作为一个遵循指令的代理应用于具身环境，已经成为基础，充分利用了LLM的推理能力Hu等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib8)）；Singh等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib26)）；Yang等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib34)）；Pantazopoulos等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib20)）；Yun等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib35)）。为了克服LLMs对环境中特定领域条件的知识不足，先前的工作已将领域相关信息融入其中。Ichter等人（[2022](https://arxiv.org/html/2411.17135v1#bib.bib9)）利用离线数据集学习动作的价值，随后与LLM的标记生成概率结合，以校准LLM在不同领域的决策。Song等人（[2023a](https://arxiv.org/html/2411.17135v1#bib.bib27)）则使用专家数据集作为任务规划的知识库进行检索增强。与那些直接将LLMs作为代理策略并需要在线推理的工作不同，我们的研究专注于在离线强化学习中利用LLMs进行奖励估计，从而实现更高效的代理结构。

LLMs在奖励设计中的应用。在强化学习（RL）中，奖励工程一直是一个长期存在的挑战，传统上通过人工的试错法或借助领域专家的知识来解决。而逆向强化学习则旨在从无奖励的专家示范中推断潜在的奖励函数Hadfield-Menell等人（[2016](https://arxiv.org/html/2411.17135v1#bib.bib6)）；Klein等人（[2012](https://arxiv.org/html/2411.17135v1#bib.bib10)）。随着强大基础模型的出现，近年来的研究已经开始利用这些模型来生成奖励函数Wang等人（[2024](https://arxiv.org/html/2411.17135v1#bib.bib32)）；Du等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib5)）；Rocamonde等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib24)）；Baumli等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib2)）。Kwon等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib13)）利用LLMs的上下文学习来评估高层任务的各个回合。Ma等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib17)）借助LLMs的代码生成能力，在给定环境编程代码的情况下，生成多个基于代码的奖励函数，以在线训练强化学习代理，并通过代理训练统计数据的反馈来增强它们。我们的CoREn框架也利用LLMs进行奖励设计；然而，该框架通过专注于生成领域基础的奖励而与众不同，且无需直接与环境互动，尤其适用于在环境的具体信息仅限于稀疏奖励的情况下。

## 6 结论

我们提出了奖励集成框架CoREn，以实现针对离线强化学习的强大LLM-based奖励估计，特别针对具身指令跟随任务量身定制。该框架利用时空一致性引导的集成方法进行奖励估计。它在离线轨迹上生成多个逐步奖励，每个奖励专注于与上下文、结构或时间方面相关的特定一致性，然后通过稀疏奖励对齐的集成将多个奖励整合为更加符合领域的奖励。由于这是首次采用LLM进行具身智能体的离线学习，我们希望它能为LLM驱动的训练加速技术的发展提供宝贵的见解。这对于涉及长期指令跟随任务的具身智能体尤为重要，因为这些任务通常受限于稀疏的奖励信号。

## 7 限制

尽管CoREn取得了强大的性能表现，我们发现其成功在很大程度上依赖于LLM在奖励估计中的能力，正如表[4](https://arxiv.org/html/2411.17135v1#S4.T4 "Table 4 ‣ 4.3 Ablation Studies ‣ 4 Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")中的消融研究所示。我们的基于LLM的奖励估计是以离线方式进行的，即不与环境进行直接交互。然而，对LLM能力的依赖可能会成为问题，特别是当目标环境领域与LLM的预训练知识存在显著差异，且在智能体部署后环境领域持续变化时。在涉及动态Goal-POMDP环境的情况下，智能体通过训练数据集上的密集奖励离线学习到的策略可能会在任务表现上退化。我们的集成方法结合时空一致性的优势源自与训练数据集的有效对齐，但在此类非平稳环境条件下，它们的效果可能会受到限制。我们将探索应对这一限制的方法作为未来工作的方向。

## 8 致谢

我们感谢匿名审稿人提供的宝贵意见和建议。本研究得到了由韩国政府（MSIT）资助的信息与通信技术规划与评估研究所（IITP）资助的项目（RS-2022-II221045 (2022-0-01045)，RS-2022-II220043 (2022-0-00043)，RS-2019-II190421 (2019-0-00421)）、由IITP-ITRC（信息技术研究中心）资助的项目（IITP-2024-RS-2024-00437633，10%）、由韩国国家研究基金会（NRF）资助的项目（MSIT资助，编号：RS-2023-00213118）、由BK21 FOUR项目资助（S-2024-0580-000）以及三星电子资助的支持。

## 参考文献

+   Anil 等人（2023）Rohan Anil, Sebastian Borgeaud, Yonghui Wu, Jean-Baptiste Alayrac, Jiahui Yu, Radu Soricut, Johan Schalkwyk, Andrew M. Dai, Anja Hauth, Katie Millican, David Silver, Slav Petrov, Melvin Johnson, Ioannis Antonoglou, Julian Schrittwieser, Amelia Glaese, Jilin Chen, Emily Pitler, Timothy P. Lillicrap, Angeliki Lazaridou, Orhan Firat, James Molloy, Michael Isard, Paul Ronald Barham, Tom Hennigan, Benjamin Lee, Fabio Viola, Malcolm Reynolds, Yuanzhong Xu, Ryan Doherty, Eli Collins, Clemens Meyer, Eliza Rutherford, Erica Moreira, Kareem Ayoub, Megha Goel, George Tucker, Enrique Piqueras, Maxim Krikun, Iain Barr, Nikolay Savinov, Ivo Danihelka, Becca Roelofs, Anaïs White, Anders Andreassen, Tamara von Glehn, Lakshman Yagati, Mehran Kazemi, Lucas Gonzalez, Misha Khalman, Jakub Sygnowski, 等人. 2023. [Gemini：一系列高效的多模态模型](https://doi.org/10.48550/ARXIV.2312.11805). *CoRR*, abs/2312.11805。

+   Baumli 等人（2023）Kate Baumli, Satinder Baveja, Feryal M. P. Behbahani, Harris Chan, Gheorghe Comanici, Sebastian Flennerhag, Maxime Gazeau, Kristian Holsheimer, Dan Horgan, Michael Laskin, Clare Lyle, Hussain Masoom, Kay McKinney, Volodymyr Mnih, Alexander Neitz, Fabio Pardo, Jack Parker-Holder, John Quan, Tim Rocktäschel, Himanshu Sahni, Tom Schaul, Yannick Schroecker, Stephen Spencer, Richie Steigerwald, Luyu Wang, 和 Lei Zhang. 2023. [视觉-语言模型作为奖励的来源](https://doi.org/10.48550/ARXIV.2312.09187). *CoRR*, abs/2312.09187。

+   Chu 等人（2023）Kun Chu, Xufeng Zhao, Cornelius Weber, Mengdi Li, 和 Stefan Wermter. 2023. [通过大语言模型的反馈加速机器人操作的强化学习](https://doi.org/10.48550/ARXIV.2311.02379). *CoRR*, abs/2311.02379。

+   Devlin 等人（2019）Jacob Devlin, Ming-Wei Chang, Kenton Lee, 和 Kristina Toutanova. 2019. [BERT：用于语言理解的深度双向变换器的预训练](https://doi.org/10.18653/V1/N19-1423). 收录于 *2019年北美计算语言学协会年会：人类语言技术，NAACL-HLT 2019，明尼阿波利斯，美国，2019年6月2-7日，第1卷（长篇与短篇论文）*，第4171–4186页. 计算语言学协会。

+   Du 等人（2023）Yuqing Du, Olivia Watkins, Zihan Wang, Cédric Colas, Trevor Darrell, Pieter Abbeel, Abhishek Gupta, 和 Jacob Andreas. 2023. [利用大语言模型引导强化学习中的预训练](https://proceedings.mlr.press/v202/du23f.html). 收录于 *国际机器学习会议，ICML 2023，2023年7月23-29日，夏威夷檀香山，美国*，*机器学习研究论文集*第202卷，第8657–8677页。PMLR。

+   Hadfield-Menell 等人（2016）Dylan Hadfield-Menell, Stuart Russell, Pieter Abbeel, 和 Anca D. Dragan. 2016. [Cooperative inverse reinforcement learning](https://proceedings.neurips.cc/paper/2016/hash/c3395dd46c34fa7fd8d729d8cf88b7a8-Abstract.html)。在 *神经信息处理系统进展 29: 神经信息处理系统年度会议 2016，2016年12月5-10日，西班牙巴塞罗那*，页码3909–3917。

+   Hashemzadeh 等人（2024）Maryam Hashemzadeh, Elias Stengel-Eskin, Sarath Chandar, 和 Marc-Alexandre Côté. 2024. [Sub-goal distillation: A method to improve small language agents](https://doi.org/10.48550/ARXIV.2405.02749)。*CoRR*, abs/2405.02749。

+   Hu 等人（2023）Mengkang Hu, Yao Mu, Xinmiao Yu, Mingyu Ding, Shiguang Wu, Wenqi Shao, Qiguang Chen, Bin Wang, Yu Qiao, 和 Ping Luo. 2023. [Tree-planner: Efficient close-loop task planning with large language models](https://doi.org/10.48550/ARXIV.2310.08582)。*CoRR*, abs/2310.08582。

+   Ichter 等人（2022）Brian Ichter, Anthony Brohan, Yevgen Chebotar, Chelsea Finn, Karol Hausman, Alexander Herzog, Daniel Ho, Julian Ibarz, Alex Irpan, Eric Jang, Ryan Julian, Dmitry Kalashnikov, Sergey Levine, Yao Lu, Carolina Parada, Kanishka Rao, Pierre Sermanet, Alexander Toshev, Vincent Vanhoucke, Fei Xia, Ted Xiao, Peng Xu, Mengyuan Yan, Noah Brown, Michael Ahn, Omar Cortes, Nicolas Sievers, Clayton Tan, Sichun Xu, Diego Reyes, Jarek Rettinghouse, Jornell Quiambao, Peter Pastor, Linda Luu, Kuang-Huei Lee, Yuheng Kuang, Sally Jesmonth, Nikhil J. Joshi, Kyle Jeffrey, Rosario Jauregui Ruano, Jasmine Hsu, Keerthana Gopalakrishnan, Byron David, Andy Zeng, 和 Chuyuan Kelly Fu. 2022. [Do as I can, not as I say: Grounding language in robotic affordances](https://proceedings.mlr.press/v205/ichter23a.html)。在 *2022年机器人学习会议（CoRL 2022），2022年12月14-18日，新西兰奥克兰*，《机器学习研究会议论文集》第205卷，页码287–318。PMLR。

+   Klein 等人（2012）Edouard Klein, Matthieu Geist, Bilal Piot, 和 Olivier Pietquin. 2012. [Inverse reinforcement learning through structured classification](https://proceedings.neurips.cc/paper/2012/hash/559cb990c9dffd8675f6bc2186971dc2-Abstract.html)。在 *神经信息处理系统进展 25: 第26届神经信息处理系统年度会议 2012. 2012年12月3-6日，美国内华达州太浩湖举行的会议论文集*，页码1016–1024。

+   Kumar 等人（2020a）Aviral Kumar, Aurick Zhou, George Tucker, 和 Sergey Levine. 2020a. [Conservative q-learning for offline reinforcement learning](https://proceedings.neurips.cc/paper/2020/hash/0d2b2061826a5df3221116a5085a6052-Abstract.html)。在 *神经信息处理系统进展 33: 神经信息处理系统年度会议 2020，NeurIPS 2020，2020年12月6-12日，虚拟*。

+   Kumar 等人（2020b）Aviral Kumar, Aurick Zhou, George Tucker 和 Sergey Levine。2020b年。[离线强化学习的保守Q学习](https://proceedings.neurips.cc/paper/2020/hash/0d2b2061826a5df3221116a5085a6052-Abstract.html)。在 *神经信息处理系统进展 33：2020年神经信息处理系统年度会议，NeurIPS 2020，2020年12月6日至12日，虚拟会议*。

+   Kwon 等人（2023）Minae Kwon, Sang Michael Xie, Kalesha Bullard 和 Dorsa Sadigh。2023年。[基于语言模型的奖励设计](https://openreview.net/pdf?id=10uNUgI5Kl)。在 *第十一届国际学习表征会议，ICLR 2023，卢旺达基加利，2023年5月1日至5日*。OpenReview.net。

+   Lee（2024）Jieh-Sheng Lee。2024年。[Instructpatentgpt：训练专利语言模型以遵循人类反馈的指令](https://doi.org/10.48550/ARXIV.2406.16897)。*CoRR*，abs/2406.16897。

+   Li 等人（2023）Hao Li, Xue Yang, Zhaokai Wang, Xizhou Zhu, Jie Zhou, Yu Qiao, Xiaogang Wang, Hongsheng Li, Lewei Lu 和 Jifeng Dai。2023年。[自动 mc-reward：使用大型语言模型为Minecraft实现自动化稠密奖励设计](https://doi.org/10.48550/ARXIV.2312.09238)。*CoRR*，abs/2312.09238。

+   Logeswaran 等人（2022）Lajanugen Logeswaran, Yao Fu, Moontae Lee 和 Honglak Lee。2022年。使用语言模型进行少样本子目标规划。*arXiv 预印本 arXiv:2205.14288*。

+   Ma 等人（2023）Yecheng Jason Ma, William Liang, Guanzhi Wang, De-An Huang, Osbert Bastani, Dinesh Jayaraman, Yuke Zhu, Linxi Fan 和 Anima Anandkumar。2023年。[Eureka: 通过编码大型语言模型实现人类级别的奖励设计](https://doi.org/10.48550/ARXIV.2310.12931)。*CoRR*，abs/2310.12931。

+   Ma 等人（2022）Yecheng Jason Ma, Jason Yan, Dinesh Jayaraman 和 Osbert Bastani。2022年。[通过 $f$-advantage 回归实现离线目标条件强化学习](http://papers.nips.cc/paper_files/paper/2022/hash/022a39052abf9ca467e268923057dfc0-Abstract-Conference.html)。在 *神经信息处理系统进展 35：2022 年神经信息处理系统年度会议，NeurIPS 2022，美国路易斯安那州新奥尔良，2022年11月28日至12月9日*。

+   Padmakumar 等人（2023）Aishwarya Padmakumar, Mert Inan, Spandana Gella, Patrick L Lange 和 Dilek Hakkani-Tur。2023年。通过合成体现对话增强的多模态体现计划预测。在 *2023年自然语言处理实证方法会议论文集*，第6114-6131页。

+   Pantazopoulos 等人（2023）Georgios Pantazopoulos, Malvina Nikandrou, Amit Parekh, Bhathiya Hemanthage, Arash Eshghi, Ioannis Konstas, Verena Rieser, Oliver Lemon 和 Alessandro Suglia。2023年。用于交互式体现任务完成的多任务多模态提示训练。*arXiv 预印本 arXiv:2311.04067*。

+   Park 等人（2023）Seohong Park, Dibya Ghosh, Benjamin Eysenbach 和 Sergey Levine. 2023. [HIQL：具有潜在状态作为动作的离线目标条件强化学习](http://papers.nips.cc/paper_files/paper/2023/hash/6d7c4a0727e089ed6cdd3151cbe8d8ba-Abstract-Conference.html). 收录于 *神经信息处理系统进展36：2023年神经信息处理系统年会，NeurIPS 2023，美国路易斯安那州新奥尔良，2023年12月10-16日*。

+   Puig 等人（2018）Xavier Puig, Kevin Ra, Marko Boben, Jiaman Li, Tingwu Wang, Sanja Fidler 和 Antonio Torralba. 2018. [Virtualhome：通过程序模拟家庭活动](https://doi.org/10.1109/CVPR.2018.00886). 收录于 *2018 IEEE计算机视觉与模式识别会议，CVPR 2018，美国犹他州盐湖城，2018年6月18-22日*，第8494-8502页。计算机视觉基金会 / IEEE计算机学会。

+   Reimers 和 Gurevych（2019）Nils Reimers 和 Iryna Gurevych. 2019. [Sentence-bert：使用Siamese BERT网络的句子嵌入](https://doi.org/10.18653/V1/D19-1410). 收录于 *2019年实证方法自然语言处理会议暨第9届国际联合自然语言处理会议，EMNLP-IJCNLP 2019，中国香港，2019年11月3-7日*，第3980-3990页。计算语言学协会。

+   Rocamonde 等人（2023）Juan Rocamonde, Victoriano Montesinos, Elvis Nava, Ethan Perez 和 David Lindner. 2023. [视觉-语言模型是强化学习的零-shot奖励模型](https://doi.org/10.48550/ARXIV.2310.12921). *CoRR*, abs/2310.12921。

+   Shridhar 等人（2020）Mohit Shridhar, Jesse Thomason, Daniel Gordon, Yonatan Bisk, Winson Han, Roozbeh Mottaghi, Luke Zettlemoyer 和 Dieter Fox. 2020. [ALFRED：用于解释日常任务的基础指令基准](https://doi.org/10.1109/CVPR42600.2020.01075). 收录于 *2020 IEEE/CVF计算机视觉与模式识别会议，CVPR 2020，美国华盛顿州西雅图，2020年6月13-19日*，第10737-10746页。计算机视觉基金会 / IEEE。

+   Singh 等人（2023）Ishika Singh, Valts Blukis, Arsalan Mousavian, Ankit Goyal, Danfei Xu, Jonathan Tremblay, Dieter Fox, Jesse Thomason 和 Animesh Garg. 2023. [Progprompt：使用大型语言模型生成具身机器人任务计划](https://doi.org/10.1109/ICRA48891.2023.10161317). 收录于 *IEEE国际机器人与自动化会议，ICRA 2023，英国伦敦，2023年5月29日至6月2日*，第11523-11530页。IEEE。

+   Song 等人（2023a）Chan Hee Song, Brian M. Sadler, Jiaman Wu, Wei-Lun Chao, Clayton Washington 和 Yu Su. 2023a. [Llm-planner：使用大型语言模型进行少量示例基础的具身代理规划](https://doi.org/10.1109/ICCV51070.2023.00280). 收录于 *IEEE/CVF国际计算机视觉大会，ICCV 2023，法国巴黎，2023年10月1-6日*，第2986-2997页。IEEE。

+   Song 等人 (2023b) 禅熙宋，布赖恩·M·萨德勒，佳曼·吴，伟伦·赵，克莱顿·华盛顿，余苏。2023b. [Llm-planner: 通过大型语言模型为具象智能体进行少样本实地规划](https://doi.org/10.1109/ICCV51070.2023.00280)。发表于 *IEEE/CVF计算机视觉国际会议，ICCV 2023，法国巴黎，2023年10月1日至6日*，第2986-2997页。IEEE。

+   Song 等人 (2023c) 佳扬宋，哲华周，家伟刘，春荣方，展书，雷马。2023c. [自我精炼的大型语言模型作为机器人深度强化学习的自动化奖励函数设计者](https://doi.org/10.48550/ARXIV.2309.06687)。*CoRR*，abs/2309.06687。

+   van Hasselt 等人 (2016) 哈多·范哈塞尔特，阿瑟·格兹，大卫·西尔弗。2016. [双重Q学习的深度强化学习](https://doi.org/10.1609/AAAI.V30I1.10295)。发表于 *第三十届人工智能AAAI会议，2016年2月12日至17日，美国凤凰城，亚利桑那州*，第2094-2100页。AAAI出版社。

+   Wang 等人 (2023) 学智王，杰森·魏，达尔·舒尔曼，阙伟·乐，埃德·H·池，沙然·纳朗，阿坎莎·周德瑞，邓尼·周。2023. [自一致性改善语言模型中的思维链推理](https://openreview.net/pdf?id=1PL1NIMMrw)。发表于 *第十一届国际学习表征会议，ICLR 2023，卢旺达基加利，2023年5月1日至5日*。OpenReview.net。

+   Wang 等人 (2024) 玉飞王，展义孙，杰西张，周贤，厄尔德姆·比耶克，大卫·赫尔德，扎克里·埃里克森。2024. [RL-VLM-F: 来自视觉语言基础模型反馈的强化学习](https://doi.org/10.48550/ARXIV.2402.03681)。*CoRR*，abs/2402.03681。

+   Xie 等人 (2024) 田宝谢，思恒赵，陈·亨利吴，逸涛刘，乾罗，维克多钟，彦超杨，涛余。2024. [Text2reward: 使用语言模型进行强化学习的奖励塑造](https://openreview.net/forum?id=tUM39YTRxH)。发表于 *第十二届国际学习表征会议，ICLR 2024，奥地利维也纳，2024年5月7日至11日*。OpenReview.net。

+   Yang 等人 (2023) 成富杨，彦君陈，建伟杨，希阳戴，陆元，余强·弗兰克·王，凯伟常。2023. Lacma: 通过元动作的语言对齐对比学习进行具象指令跟随。*arXiv预印本 arXiv:2310.12344*。

+   Yun 等人 (2023) 田云，子来曾，库纳尔·汉达，阿什什·V·塔普利亚尔，博·庞，艾莉·帕夫利克，陈·孙。2023. 具象序列建模中抽象状态表示的出现。*arXiv预印本 arXiv:2311.02171*。

## 附录 A 实验设置

### A.1 环境

我们使用VirtualHome Puig等人（[2018](https://arxiv.org/html/2411.17135v1#bib.bib22)）开发的环境和基准，该环境专为模拟具身家庭任务而设计。在这个环境中，与家庭任务活动相关的动作是通过结合可用的操作行为和物体来建立的。这些动作按顺序执行以完成复杂的家庭任务。CoREn使用一个由4个房间组成的配置，利用共58个不同的动作。这些动作是由8种不同的操作行为（查找、抓取、打开、关闭、坐、放置、放入、打开开关）与环境中各种物体的组合而来。

单领域评估。对于表格[1](https://arxiv.org/html/2411.17135v1#S4.T1 "Table 1 ‣ 4.2 Main Results ‣ 4 Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")中的单领域实验，我们使用每个任务10条指令来评估25个不同的任务。这些指令分为两类：5条细化指令，提供任务的详细描述，以及5条抽象指令，提供更一般的概述。任务的详细示例如表格[16](https://arxiv.org/html/2411.17135v1#A4.T16 "Table 16 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")所示。

跨领域评估。在跨领域设置中，我们评估了在改变物体位置的环境中执行的任务（例如，将苹果从桌子上移动到冰箱内），如表格[18](https://arxiv.org/html/2411.17135v1#A4.T18 "Table 18 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")所述。我们从单领域评估中的25个任务中评估了8个任务，每个任务使用5条指令。由于某些物体无法在新布局中重新定位，因此任务数量有所减少。与单领域评估类似，每个任务使用细化指令和抽象指令进行评估，每个任务总共使用6条指令。跨领域评估中使用的任务的详细示例如表格[17](https://arxiv.org/html/2411.17135v1#A4.T17 "Table 17 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")所示。

### A.2 离线数据集

为了构建离线强化学习的训练数据集$\mathcal{D}$，我们为每个25个不同任务使用一个专家轨迹。每个专家轨迹在中间步骤上通过随机动作进行增强，导致失败的试验。这个过程为离线数据集$\mathcal{D}$生成了大约8,000个轨迹。对于每个专家轨迹，标注了一个稀疏奖励$1$来表示其成功，而对于每个采样的失败轨迹，标注了一个稀疏奖励$0$来表示其失败。总体来说，我们利用一个成功轨迹和一个失败轨迹来建立稀疏奖励。

## 附录B 实现

在本节中，我们展示了CoREn和基线的实现细节。

### B.1 CoREn实现

我们使用Python v3.9.19和自动梯度框架Jax v0.4.7实现了我们的框架。模型在配备NVIDIA RTX A6000 GPU的系统上进行训练。CoREn的实现细节包括以下几个部分：（i）基于LLM的奖励估计，（ii）考虑估计奖励的时空一致性，（iii）基于领域的奖励集成，以及（iv）离线强化学习。

#### B.1.1 基于LLM的奖励估计

LLM $\Phi_{\textnormal{LLM}}$ 接受用户指令 $l$、观察 $o$ 和动作 $a$ 作为输入，以及一个提示 $\mathcal{P}$，用于估计 $a$ 的奖励，依据它们如何有助于完成 $l$。我们使用多个 $N$ 个提示 $\mathcal{P}_{1},\cdots,\mathcal{P}_{N}$，这些提示在奖励估计任务的描述方法、上下文示范的结合或链式推理（CoT）提示的使用上有所不同。具体来说，我们使用五种不同类型的提示来生成有效的奖励：一个包含奖励估计任务解释和所需格式的简单提示，三个包含不同示范的上下文学习（ICL）提示，以及一个包含人工编写的奖励估计推理路径的CoT提示。每个提示都包含奖励估计的标准，包括哪些动作应获得哪些奖励。例如，对于一个应该执行的动作，基于之前已完成的动作，给予2的奖励；而对于一个涉及搜索与任务无关的物体的动作，则给予-1的奖励。提示示例见表[19](https://arxiv.org/html/2411.17135v1#A4.T19 "Table 19 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")、[20](https://arxiv.org/html/2411.17135v1#A4.T20 "Table 20 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")、[21](https://arxiv.org/html/2411.17135v1#A4.T21 "Table 21 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble") 和 [22](https://arxiv.org/html/2411.17135v1#A4.T22 "Table 22 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")。

与上述提示配合使用，我们采用了几种LLM：LLama-8B、Gemini 1.0 Pro、PaLM 和 GPT4 Turbo。对于 GPT4 Turbo，使用温度为0.5，而其他模型的温度设置为0.7。温度设置基于每个模型的特点，旨在平衡奖励生成过程中探索与利用的权衡。表[6](https://arxiv.org/html/2411.17135v1#A2.T6 "Table 6 ‣ B.1.1 LLM-based reward estimation ‣ B.1 CoREn Implementation ‣ Appendix B Implementation ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble") 指定了所使用的 LLM 及其相应的模型大小，以及进行消融研究所使用的温度超参数，详见表[4](https://arxiv.org/html/2411.17135v1#S4.T4 "Table 4 ‣ 4.3 Ablation Studies ‣ 4 Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")。

LLM 模型大小 温度 LLaMA3 8B 0.7 PaLM - 0.7 GPT4 turbo - 0.5 Gemini 1.0 Pro - 0.7

表格 6：LLM、其模型大小以及在表格 [4](https://arxiv.org/html/2411.17135v1#S4.T4 "Table 4 ‣ 4.3 Ablation Studies ‣ 4 Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble") 中使用的温度超参数

#### B.1.2 空间-时间一致性

在这里，我们提供了空间-时间一致性的详细描述和机制，包括上下文、一致性结构和时间一致性，详见第 [3.2](https://arxiv.org/html/2411.17135v1#S3.SS2 "3.2 Spatio-Temporally Consistent Rewards ‣ 3 Our Approach ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble") 节。

##### 上下文一致性

上下文一致性涉及使用之前介绍的提示来估计奖励，然后对结果应用多数投票法。

##### 结构一致性

结构一致性包含一个过程，其中奖励估计器 $\Phi_{\textnormal{LLM}}$ 自我检查其通过 MDP 特定查询反映环境部分信息的能力。为了实现这一点，我们生成了一个 MDP 特定的 QA 数据集 $\mathcal{D}_{\textnormal{QA}}=\{q(o),a(o):o\in\tau\in\mathcal{D}\}$。该 QA 数据集由比估计行为奖励推理任务更容易回答的查询 $q(o)$ 组成，所需的仅是观察和指令。通过评估对这些查询的回答是否正确，我们可以判断奖励估计是否在正确考虑环境内部结构的情况下进行。

为了为 MDP 特定数据集 $\mathcal{D}_{\textnormal{QA}}$ 创建答案，我们使用 GPT4，并使用聚焦于识别在实现给定指令中起关键作用的对象的查询。通过这一过程，我们生成了总共 139 对 QA 问题。表格 [7](https://arxiv.org/html/2411.17135v1#A2.T7 "Table 7 ‣ Structural Consistency ‣ B.1.2 Spatio-Temporal Consistency ‣ B.1 CoREn Implementation ‣ Appendix B Implementation ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble") 显示了 QA 对的问题示例。

给定观察$o$，$\Phi_{\textnormal{LLM}}$将查询$q(o^{\prime})`与提示$\mathcal{P}_{n}$一起作为输入，并生成响应$\Phi_{\textnormal{LLM}}(\mathcal{P}_{n},q(o^{\prime}))$。这里，$q(o^{\prime})$是根据使用句子转换模型Reimers和Gurevych（[2019](https://arxiv.org/html/2411.17135v1#bib.bib23)）的句子嵌入相似度在$o$和$o^{\prime}$之间选择的。我们通过将查询$q(o)$直接附加到提示$\mathcal{P}_{n}$的末尾来将其集成到提示中。表[19](https://arxiv.org/html/2411.17135v1#A4.T19 "Table 19 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")展示了这些示例。为了确定响应与实际答案的对齐程度，我们使用基于相似度的评估器$E$。具体来说，如果响应与真实答案$a(o^{\prime})$之间的句子嵌入相似度低于0.5的阈值，则认为响应不正确。

| MDP特定数据集中的问答对 |
| --- |
| 查询 1: |
| 指令: <instruction> |
| 可见物体: 纸张, 墙架, 谷物, 鼠标, 杯子, 奶油包, 饼干 |
| 在当前可见的物体中，哪些物体与任务相关？ |
| 答案 1: |
| 墙架, 谷物 |
|  |
| 查询 2: |
| 指令: <instruction> |
| 可见物体: 纸张, 电脑屏幕, 桌子, 键盘, 鼠标, 杯子 |
| 在当前可见的物体中，哪些物体与任务相关？ |
| 答案 2: |
| 桌子, 猫 |

表7：MDP特定数据集$\mathcal{D}_{\textnormal{QA}}$

##### 时间一致性

时间一致性涉及计算每个提示$\mathcal{P}_{n}$的高价值动作序列$H_{n}(\tau)$：

|  | $H_{n}(\tau)=\{\underset{l}{\text{argmax}}\,\,\Phi_{\textnormal{LLM}}(\mathcal{P}_{n},(o,l,i))\},$ |  | (11) |
| --- | --- | --- | --- |

其中$o$是一个观察，$l$是一个动作，$i$是一个指令。请注意，存在多个高价值动作的序列。对于$H_{n}(\tau)$中的每个高价值动作序列，我们向$\Phi_{\textnormal{LLM}}$提出查询$q(i,\tau,n)$，以确定该序列是否能够完成指令$i$。表[8](https://arxiv.org/html/2411.17135v1#A2.T8 "Table 8 ‣ Temporal consistency ‣ B.1.2 Spatio-Temporal Consistency ‣ B.1 CoREn Implementation ‣ Appendix B Implementation ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")提供了一个实际查询的示例。如果查询被违反，即$\Phi_{\textnormal{LLM}}(q(i,\tau,n))$返回False，则动作$l\in H_{n}(\tau)$的奖励将被忽略。否则，如果$l\notin H_{n}(\tau)$或$\Phi_{\textnormal{LLM}}(q(i,\tau,n))$返回True，则估计的奖励$\Phi_{\textnormal{LLM}}(\mathcal{P}_{n},(o,l,i))$将包含在多数投票过程中，以构建时间一致的奖励。

一个示例说明了奖励估计如何根据上下文、一致性结构和时间一致性变化，可以在表 [23](https://arxiv.org/html/2411.17135v1#A4.T23 "Table 23 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")、[24](https://arxiv.org/html/2411.17135v1#A4.T24 "Table 24 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble") 和 [25](https://arxiv.org/html/2411.17135v1#A4.T25 "Table 25 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble") 中找到。 

| 反向验证提示 |
| --- |
| 提示 1： |
| 动作列表：[action list] |
| 从上面提供的动作列表中，我选择了几个动作来形成一个动作序列，如 <action sequence>。如果按顺序执行这个动作序列，是否可以实现 <instruction>？ |
| 仅回答“possible”或“impossible”。 |
|  |
| 提示 2： |
| 你已经从上面的列表中创建了一个动作序列 <action sequence> 来实现 <instruction>。 |
| 然而，这个序列是错误的，因为后续的动作不能在没有先前动作执行的情况下执行。 |
| 陈述出错顺序的步骤数。仅输出数字。如果有多个数字，用逗号分隔。 |

表 8：用于时间一致性反向验证的提示

#### B.1.3 域基础的奖励集成

为了学习时空一致奖励 $r^{C},r^{S}$ 和 $r^{T}$ 的集成方法，我们训练一个奖励协调器 $\Psi_{\theta}$。

超参数值 网络架构 Bert 用于 3 分类 批量大小 16 激活函数 ReLU, Softmax 学习率 1e-4 梯度裁剪 3

表 9：奖励协调器 $\Psi_{\theta}$ 的超参数

协调器负责将轨迹的回报与标注在轨迹上的稀疏奖励$f_{s}(i,\tau)$对齐，如公式([10](https://arxiv.org/html/2411.17135v1#S3.E10 "In 3.3 A Domain-Grounded Reward Ensemble ‣ 3 Our Approach ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble"))中所述。轨迹的回报定义为奖励$\hat{r}$的总和。然而，回报的规模会根据轨迹长度$H$的不同而有所变化。为了将回报与稀疏奖励$f_{s}(i,\tau)\in{-1,1}$对齐，需要进行适当的归一化。假设LLM奖励估计$\Phi_{\textnormal{LLM}}(\mathcal{P}_{n},(o,l,i))$的取值范围为$[-K,K]$，我们通过将回报除以$HK$来对其进行归一化。这个归一化过程确保回报的值落在-1和1之间，使其与稀疏奖励兼容。协调器采用了基于Bert架构的实现，Devlin等人提出的架构 ([2019](https://arxiv.org/html/2411.17135v1#bib.bib4))，并调整为一个三分类任务。$\Psi_{\theta}$的超参数设置总结在表[9](https://arxiv.org/html/2411.17135v1#A2.T9 "Table 9 ‣ B.1.3 Domain-grounded Reward Ensemble ‣ B.1 CoREn Implementation ‣ Appendix B Implementation ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")中。

#### B.1.4 离线强化学习

关于代理策略$\pi$的模型结构，我们采用了58个头的GPT2架构来表示动作值。为了优化$\pi$，我们使用了van Hasselt等人提出的Double DQN（DDQN）算法 ([2016](https://arxiv.org/html/2411.17135v1#bib.bib30)) 来处理我们环境中的离散动作空间，并且采用了Kumar等人提出的保守Q学习（CQL）([2020b](https://arxiv.org/html/2411.17135v1#bib.bib12)) 来解决离线强化学习中固有的Q值过高估计问题。$\pi$的超参数设置总结在表[10](https://arxiv.org/html/2411.17135v1#A2.T10 "Table 10 ‣ B.1.4 Offline Reinforcement Learning ‣ B.1 CoREn Implementation ‣ Appendix B Implementation ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")中。

超参数 值 网络架构 GPT2 位置数 1536 层数 2 头数 4 激活函数 ReLU 残差丢弃率 0.1 嵌入丢弃率 0.1 注意力丢弃率 0.1 层归一化epsilon 0 嵌入维度 768 学习率 1e-4 目标更新间隔 250 折扣因子$\gamma$ 0.99 用于软目标更新的$\tau$ 0.005

表10：策略$\pi$的超参数

### B.2 基准实现

#### B.2.1 RL代理

Lafite-RL。在Lafite-RLChu等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib3)）的研究中，利用LLM根据观察和指令估计给定离线数据集中每个动作的回报。估计的回报是三个值之一：好（1）、中立（0）或差（-1）。每个动作的这种内在逐步回报与给定离线数据集中的稀疏回报结合，以建立一个回报增强数据集用于RL。我们使用表[19](https://arxiv.org/html/2411.17135v1#A4.T19 "Table 19 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")中的提示词进行LLM推理。代理的策略结构及其训练超参数与CoREn中使用的相同。

RDLM。在RDLMKwon等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib13)）的研究中，利用LLM根据任务描述和用户指定的上下文示范来估计轨迹回报。虽然提示词是在原始RDLM工作中基于代理的成功滚动不断构建的，但由于我们离线学习设置的不同，我们通过检索增强生成（RAG）来实现提示词，以便进行回报估计。这样，我们基于评分标准（即估计的评分指南）手动建立了一个回报估计数据集，详细信息可以参见表[19](https://arxiv.org/html/2411.17135v1#A4.T19 "Table 19 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")。然后，我们动态检索三个上下文示范，考虑到指令、动作执行历史和观察之间的余弦相似度。检索到的示范与表[22](https://arxiv.org/html/2411.17135v1#A4.T22 "Table 22 ‣ D.3 The Number of Prompts ‣ Appendix D Additional Experiments ‣ LLM-Based Offline Learning for Embodied Agents via Consistency-Guided Reward Ensemble")中的提示词结合在一起，随后用于检索增强的LLM推理。代理的策略结构及其训练超参数与CoREn中使用的相同。

自一致性。在这个基准测试中，Wang等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib31)）查询LLM，以使用CoT提示词估计回报。LLM从多个$K$个回报估计中进行采样，每个回报基于不同的推理路径，并选择最一致的答案。在我们的实现中，我们设置$K=3$。代理的策略结构及其训练超参数与CoREn中使用的相同。

#### B.2.2 LM代理

Saycan. SayCanIchter等人（[2022](https://arxiv.org/html/2411.17135v1#bib.bib9)）利用LLM规划器和可用性价值函数的结合，根据给定的指令生成可行的行动计划。LLM规划器识别合适的动作，而每个动作的可用性得分则通过预训练的可用性函数计算得出。这个可用性得分被整合进LLM的令牌生成概率中，以选择可行的行动来完成任务。

在我们的实现中，我们遵循了LLM-Planner的作者所使用的方法，该方法基于专家轨迹进行检索增强的任务规划。同时，考虑到在VirtualHome中训练低级策略的挑战，我们采用LLM-Planner提供SayCan物体数据以定义价值函数的策略。这一方法使SayCan能够缩小LLM需要考虑的潜在行动列表。这简化了规划器的决策过程，提高了它选择可执行操作并有效完成任务的能力。

ProgPrompt. ProgPromptSingh等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib26)）使用编程断言语法来验证执行操作的前提条件，并通过启动预定义的恢复操作来应对失败。

我们采用与ProgPrompt相同的计划模板，这些模板采用Python风格，其中任务名称被指定为函数，可用的行动通过标题包含，访问的物体通过变量指定，每个行动则在函数中作为一行代码进行描述。我们为LLM规划器使用动态采样的上下文演示，并为ProgPrompt提供每个行动的预设前提条件。

LLM-Planner. LLM-PlannerSong等人（[2023b](https://arxiv.org/html/2411.17135v1#bib.bib28)）采用了模板化动作、k近邻（kNN）检索器和LLM规划器。规划的行动候选通过模板来建立，这些模板与环境中可见的物体相结合。LLM-Planner通过kNN检索器从我们离线数据集中的专家轨迹中检索上下文示例，然后将这些示例提供给LLM规划器。接着，规划器将这些动作模板与当前可见的物体结合起来，确定既可实现又能够完成任务的行动。

## 附录C 奖励估计的方式。

我们还研究了使用大型多模态模型（LMMs）进行奖励估计。与 CoREn 使用场景中检测到的物体名称来表示观察不同，LMMs 可以直接利用图像观察。表 [11](https://arxiv.org/html/2411.17135v1#A3.T11 "表 11 ‣ 附录 C 奖励估计的模态 ‣ 基于 LLM 的离线学习与一致性引导的奖励集成") 显示，使用 LMM 估计的奖励训练的智能体表现不如使用 LLM 估计的智能体。在这项测试中，我们使用了不同的集成方法，正如之前在表 [5](https://arxiv.org/html/2411.17135v1#S4.T5 "表 5 ‣ 4.3 消融研究 ‣ 4 实验 ‣ 基于 LLM 的离线学习与一致性引导的奖励集成") 中所描述的。我们推测，尽管图像本身可以隐性传达详细的环境信息，但 LMM 在具身环境中的表示能力有限，可能不适合用于奖励估计等高层次推理任务。

LLM LMM SR CGC 计划 SR CGC 计划 CoREn 62.0 71.4 67.2 26.0 42.1 31.1 平均 48.4 59.5 56.7 22.4 43.2 44.9 Maj. 投票 58.0 66.45 66.3 23.2 41.1 41.6

表 11：奖励估计的模态

## 附录 D 额外实验

### D.1 LLaMA3-70B 用于 LM 智能体

在这里，我们实现了 LM 智能体（即 SayCan、ProgPrompt 和 LLM-Planner）与 LLaMA3-70B，一个功能强大的 LLM。表 [12](https://arxiv.org/html/2411.17135v1#A4.T12 "表 12 ‣ D.1 LLaMA3-70B 用于 LM 智能体 ‣ 附录 D 额外实验 ‣ 基于 LLM 的离线学习与一致性引导的奖励集成") 显示了使用细粒度指令在 VirtualHome 上进行的单域性能，通过我们的 CoREN 和基于 LLaMA3-70B 的 LLM 智能体基准测试实现。我们观察到，利用 LLaMA3-70B 能够提升 ProgPrompt 和 SayCan 的表现，尤其是 ProgPrompt 展现了显著的提升。具体来说，ProgPrompt 在使用 LLaMA3-70B 作为在线智能体时，成功率（SR）平均提高了 8.8%，相比使用 LLaMA3-8B 或 Gemini。我们假设这一提升不仅仅是由于更大模型尺寸提升了 ICL 性能，也因为 LLaMA3 在代码分析能力上明显优于 GeminiAnil 等人（[2023](https://arxiv.org/html/2411.17135v1#bib.bib1)）。这一优势特别有利于 ProgPrompt 在其程序化提示中的表现。对于 LLM-Planner，发现 Gemini 比 LLaMA3 更适合。更重要的是，我们的框架在训练过程中仅使用 LLM（Gemini 1.0 Pro），与使用 LLaMA3-70B 作为具身智能体的 LLM 基准相比，仍展示了具有竞争力的表现。

精细化 RL 代理 SR CGC 计划 CoREN 66.4 74.5 69.5 LLM基础代理 SayCan-Gemini 72.0 78.2 73.8 SayCan-LLaMA3-70B 73.6 77.2 70.2 SayCan-LLaMA3-8B 4.8 22.4 63.8 ProgPrompt-Gemini 72.8 80.4 80.2 ProgPrompt-LLaMA3-70B 79.2 83.9 74.4 ProgPrompt-LLaMA3-8B 68.0 74.5 50.5 LLM-Planner-Gemini 55.2 63.8 59.7 LLM-Planner-LLaMA3-70B 39.2 53.2 51.4 LLM-Planner-LLaMA3-8B 15.1 34.0 30.6

表 12：使用 LLaMA3-70B 在 VirtualHome 上进行细粒度指令的单领域性能

### D.2 ALFRED 环境实验

为了验证我们框架的泛化能力，我们在 ALFRED 环境中进行了额外的实验 Shridhar 等人（[2020](https://arxiv.org/html/2411.17135v1#bib.bib25)）。

精细化抽象 RL 代理 SR CGC 计划 SR CGC 计划 CoREn 72.00 84.2 79.4 56.8 71.73 70.5 Lafite-RL 8.0 17.6 38.4 39.2 55.8 72.1 RDLM 46.4 61.7 72.4 14.4 23.8 40.7 自我一致性 32.8 40.6 52.7 30.4 37.4 53.1 GCRL 5.6 12.6 15.2 5.2 12.8 16.7 LLM基础代理 SayCan-Gemini 81.6 85.0 73.8 52.4 52.4 58.8 SayCan-LLaMA3 70.4 75.7 70.3 39.2 40.0 48.5 ProgPrompt-Gemini 68.8 78.0 40.6 48.0 57.4 27.9 ProgPrompt-LLaMA3 70.4 78.2 76.0 32.0 47.2 22.6 LLM-Planner-Gemini 44.4 51.8 58.7 15.7 24.6 0.0 LLM-Planner-LLaMA3 16.0 27.5 46.3 6.4 13.3 34.1 sLM基础代理 SayCan-LLaMA3Q 74.4 79.1 75.5 36.8 37.6 45.0 SayCan-GPT2 0.0 8.0 0.8 0.0 8.0 1.2 ProgPrompt-LLaMA3Q 69.6 78.5 40.3 30.4 46.9 23.6 ProgPrompt-GPT2 12.0 25.3 22.4 14.4 30.9 18.6 LLM-Planner-LLaMA3Q 8.8 17.2 40.3 2.4 10.9 32.4 LLM-Planner-GPT2 1.6 9.0 32.4 1.4 8.4 32.2

表 13：ALFRED 中 SR、CGC 和计划指标的指令跟随任务性能

虽然 ALFRED 和 VirtualHome 都模拟了家庭活动，但它们表现出不同的特性。在 VirtualHome 中，代理可以访问更广泛的环境信息，能够立即执行诸如“寻找冰箱”这样的动作，因为位置数据已经预编码。相反，在 ALFRED 中，执行这样的动作需要先进行低级别动作，比如“去厨房”，因为代理必须根据其对环境空间布局的理解进行导航。

这些差异在使用 LLM 作为 ALFRED 中奖励估计器时引入了额外的挑战，使得生成与环境领域紧密相关的奖励变得更加困难。这种增加的复杂性为我们的框架提供了一个更严格的测试，检验其生成准确奖励的能力。此外，ALFRED 环境提供了一个预先存在的离线数据集，适合验证我们工作的目标贡献，即基于给定数据集的 LLM 基础离线奖励估计。

表格 [13](https://arxiv.org/html/2411.17135v1#A4.T13 "表格 13 ‣ D.2 ALFRED 环境实验 ‣ 附录 D 其他实验 ‣ 基于一致性引导的奖励集成的LLM离线学习") 比较了CoREn和基准模型的单领域表现，而表格 [14](https://arxiv.org/html/2411.17135v1#A4.T14 "表格 14 ‣ D.2 ALFRED 环境实验 ‣ 附录 D 其他实验 ‣ 基于一致性引导的奖励集成的LLM离线学习") 则展示了它们的跨领域表现。结果表明，我们的CoREN在与其他RL代理类别基准相比时，保持了最先进的表现，并且与LM代理类别基准表现相当。具体来说，CoREN在成功率(SR)上显著超越了RL代理基准，较最具竞争力的RL基准RDLM平均提高了14.4%。

对于ALFRED的评估，我们使用了来自ALFRED基准的专家轨迹。然后，我们通过在轨迹的中间步骤添加特定的动作来扩展这些长时间跨度的轨迹。这个过程生成了25个不同的任务，每个任务由一系列特定的动作定义。使用这25个专家轨迹后，我们按照第 [4.1](https://arxiv.org/html/2411.17135v1#S4.SS1 "4.1 实验设置 ‣ 4 实验 ‣ 基于一致性引导的奖励集成的LLM离线学习")节和[A.2](https://arxiv.org/html/2411.17135v1#A1.SS2 "A.2 离线数据集 ‣ 附录 A 实验设置 ‣ 基于一致性引导的奖励集成的LLM离线学习")节中概述的相同离线数据集构建过程。此外，每个任务使用5条精细化和5条抽象指令进行评估，总共250条测试指令。我们使用与VirtualHome相同的提示进行LLM-based奖励估计。

精细化抽象RL代理 SR CGC计划 SR CGC计划 CoREn 66.7 72.2 67.0 62.2 71.7 72.2 Lafite-RL 11.1 25.6 29.6 15.6 35.8 33.1 RDLM 48.0 58.0 51.3 15.6 22.2 22.4 自一致性 0.0 11.1 7.4 0.0 11.1 7.4 GCRL 2.2 7.8 3.0 0.0 5.6 4.6 基于LLM的代理 SayCan-Gemini 44.4 50.0 51.2 0.0 15.5 25.9 SayCan-LLaMA3 40.0 45.5 46.1 11.1 22.2 29.2 ProgPrompt-Gemini 33.3 44.4 42.5 0.0 11.1 0.0 ProgPrompt-LLaMA3 31.1 42.2 39.0 4.4 13.3 0.0 LLM-Planner-Gemini 44.4 50.0 51.2 11.1 24.4 20.7 LLM-Planner-LLaMA3 26.6 34.4 44.4 0.0 14.4 25.9 基于sLM的代理 SayCan-LLaMA3Q 44.4 50.0 50.3 2.2 17.7 27.9 SayCan-GPT2 0.04 11.14 0.74 0.04 11.14 0.7 ProgPrompt-LLaMA3Q 26.6 37.7 33.8 0.0 10.0 0.0 ProgPrompt-GPT2 0.0 11.1 1.8 0.0 11.1 0.0 LLM-Planner-LLaMA3Q 20.0 28.8 46.6 0.0 15.5 29.4 LLM-Planner-GPT2 4.4 13.3 16.6 0.0 14.4 32.6

表格 14：ALFRED中的跨领域表现

### D.3 提示数量

为了研究计算时空一致性奖励中使用的提示数量的影响，我们在方程式中变化了用于计算上下文、一致性、结构性和时序一致性奖励的提示数量（[3](https://arxiv.org/html/2411.17135v1#S3.E3 "在 3.2 时空一致性奖励 ‣ 3 我们的方法 ‣ 基于 LLM 的离线学习，通过一致性引导的奖励集成")）、([4](https://arxiv.org/html/2411.17135v1#S3.E4 "在 3.2 时空一致性奖励 ‣ 3 我们的方法 ‣ 基于 LLM 的离线学习，通过一致性引导的奖励集成")）和([8](https://arxiv.org/html/2411.17135v1#S3.E8 "在 3.2 时空一致性奖励 ‣ 3 我们的方法 ‣ 基于 LLM 的离线学习，通过一致性引导的奖励集成"))。

如在章节 [B.1.1](https://arxiv.org/html/2411.17135v1#A2.SS1.SSS1 "B.1.1 基于 LLM 的奖励估计 ‣ B.1 CoREn 实现 ‣ 附录 B 实现 ‣ 基于 LLM 的离线学习，通过一致性引导的奖励集成") 中所述，我们在主文稿中使用了 5 种不同类型的提示：一个解释奖励估计任务及所需格式的简单提示，三个具有不同示范的上下文学习（ICL）提示，以及一个包括人类编写推理路径的思维链（CoT）提示，用于奖励估计。

在这里，我们探讨了基于 LM 的奖励估计中的 1、3、5 和 7 个提示：

+   •

    1 个提示：仅 CoT 提示

+   •

    3 个提示：2 个 ICL 提示和 1 个 CoT 提示

+   •

    5 个提示：如主文稿中所述

+   •

    7 个提示：增加了 2 个具有新示范的 ICL 提示

如表 [15](https://arxiv.org/html/2411.17135v1#A4.T15 "表 15 ‣ D.3 提示数量 ‣ 附录 D 附加实验 ‣ 基于 LLM 的离线学习，通过一致性引导的奖励集成") 所示，我们观察到提示数量与代理性能之间存在正相关关系。这表明，增加提示的多样性增强了我们的奖励估计过程的稳健性。更多提示带来的性能提升表明，我们的框架能够有效地利用多角度来生成更准确和一致的奖励。

提示数量 精细化 SR   CGC   计划 1 36.8   49.5   56.7 3 50.4   61.5   65.2 5 66.4   74.5   69.5 7 69.6   74.0   76.9

表 15：基于提示数量的性能

| ID | 视觉观察 | 目标 | 指令信息 |
| --- | --- | --- | --- |
| 精细化 | 抽象 |
| 1 | ![[未标注图片]](img/0924c8ea23b59004a5caf9b4c33fe6cb.png) | 厨房桌上的苹果 厨房桌上的一片面包 厨房桌上的苹果和一片面包 | 从咖啡桌上取下苹果，走到烤面包机旁，抓起一片面包，走到厨房桌前，把面包放在厨房桌上，再把苹果放在面包片上。 | 体验在美味三明治中苹果的自然脆感。 |
| 2 | ![[无标题图片]](img/a0adf2c41ded1fb850a7a53b1bba7c79.png) | 水槽上的苹果 水槽上的香蕉 | 找到咖啡桌，拿起苹果，捡起香蕉，找到水槽，把苹果放进水槽，把香蕉放进水槽。 | 准备水果以供招待你的客人。 |
| 3 | ![[无标题图片]](img/cb7a65a2f2305a75f96f0ad0e56a68ba.png) | 手持苹果 手持香蕉 坐在沙发上 | 从咖啡桌上取下苹果，拿起香蕉，走到沙发旁，坐下。 | 坐在沙发上享受水果。 |
| 4 | ![[无标题图片]](img/2280ac3c8f5c964e065ae3076a848e15.png) | 冰箱里的麦片 关闭冰箱 | 在墙上的架子上找到麦片，拿起它，走向冰箱，打开冰箱，把麦片放进去，然后关上冰箱门。 | 早餐完成后，将剩余的食物收好。 |
| 5 | ![[无标题图片]](img/03d85dc828638910ed2fd1a79472c3b7.png) | 冰箱里的三文鱼 关闭冰箱 | 找到微波炉中的三文鱼，将其拿到冰箱，打开冰箱，把三文鱼放进去，关闭冰箱。 | 将三文鱼储存在冰箱中以保持其质量。 |
| 6 | ![[无标题图片]](img/220b43ff95060547faded81d47352166.png) | 微波炉中的三文鱼 关闭微波炉 打开微波炉 | 找到微波炉，拿起三文鱼，打开微波炉，把三文鱼放进去，关上微波炉，启动微波炉。 | 用新鲜烹制的三文鱼温暖自己。 |
| 7 | ![[无标题图片]](img/5eb0b09ff5997922ff0246964f893db0.png) | 手持书本 坐在沙发上 | 从书架上取下书本，找到沙发，坐到沙发上。 | 把书带到沙发上开始阅读。 |
| 8 | ![[无标题图片]](img/3df27211323f9a506ae3cbfb9b034000.png) | 冰箱里的苹果 关闭冰箱 | 找到咖啡桌，拿起苹果，找到冰箱，打开冰箱，把苹果放进去，关闭冰箱。 | 将苹果储存在冰箱中以保持最大新鲜度。 |
| 9 | ![[无标题图片]](img/e9aed9f6896d4d34dd01f96137db7142.png) | 冰箱里的香蕉 关闭冰箱 | 找到咖啡桌，拿起香蕉，找到冰箱，打开冰箱，把香蕉放进去，关闭冰箱。 | 保持香蕉冷藏，以维持其口感。 |
| 10 | ![[无标题图片]](img/3115aff5bd2be725111e598bf3b8cfba.png) | 浴室柜中的牙膏 关闭浴室柜 | 从浴室台面上拿起牙膏，放入浴室柜中，关闭浴室柜。 | 整理你的浴室用品，保持整洁。 |

表16：VirtualHome单域任务示例

ID 视觉观察 目标 指令 信息 精细化 抽象 1 ![[无标题图片]](img/2280ac3c8f5c964e065ae3076a848e15.png)  冰箱里的谷物  关闭冰箱门   找到桌子上的谷物，把它拿到冰箱，打开冰箱门，放进去，然后关上冰箱。早餐完成后，把剩余的食物放进冰箱。   2 ![[无标题图片]](img/94523d5cbac257ac1629c784964ed6a1.png)  厨房桌上的谷物   从咖啡桌上拿起谷物，走到厨房桌，放到桌子上。把早餐准备好，放在桌子上。   3 ![[无标题图片]](img/513bd898f410d5d1d2b017f5740a4398.png)  厨房桌上的奶油面包   找到咖啡桌，拿起奶油面包，找到厨房桌，把奶油面包放到厨房桌上。快速准备一个美味的奶油小吃放到桌上。   4 ![[无标题图片]](img/165c0adc6c197b91ae7a5356c2d2011d.png)  桌子上的猫   找到床，抓起猫，找到桌子，把猫放到桌子上。让你的猫成为你工作日常的一部分。   5 ![[无标题图片]](img/7f1955709beb1e64e72fdd9780a0abd5.png)  浴缸里的猫   从床上拿起猫，走到浴缸，把猫放到浴缸里。是时候给你的猫做一个完美的清理了。   6 ![[无标题图片]](img/18f9ffbd0ed6046a2b81d78f736641bb.png)  手里拿着书籍  坐在沙发上   从桌子上拿起书，走到沙发，坐在沙发上。把书带到沙发上开始阅读。   7 ![[无标题图片]](img/18f9ffbd0ed6046a2b81d78f736641bb.png)  手里拿着书籍  坐在床上   找到桌子，拿起书，找到床，坐在床上。睡前放松，享受一场舒缓的阅读时光。   8 ![[无标题图片]](img/8c8c15c9afe63ea2352b976e4a3829a1.png)  手里拿着奶油面包  坐在沙发上   找到咖啡桌，拿起奶油面包，找到沙发，坐在沙发上。享受一块奶油面包，作为沙发上的美味小吃。

表格 17：VirtualHome 跨领域任务示例

|  | 谷物 | 奶油面包 | 猫 | 书籍 |
| --- | --- | --- | --- | --- |
| 单一领域 | ![[无标题图片]](img/7e6210d3f09f508fad278a495da49de7.png)  | ![[无标题图片]](img/7e6210d3f09f508fad278a495da49de7.png)  | ![[无标题图片]](img/141d78d33838c75b01ae5a725560e54b.png)  | ![[无标题图片]](img/5eb0b09ff5997922ff0246964f893db0.png)  |
| 跨领域 | ![[无标题图片]](img/8c8c15c9afe63ea2352b976e4a3829a1.png)  | ![[无标题图片]](img/8c8c15c9afe63ea2352b976e4a3829a1.png)  | ![[无标题图片]](img/f1c14a940c0b131351ce5a539a64dff9.png)  | ![[无标题图片]](img/18f9ffbd0ed6046a2b81d78f736641bb.png)  |

表格 18：跨领域评估中的不同物体位置

| 天真的提示 |
| --- |
| 机器人：你好，我是一个在屋子里工作的机器人。你可以让我做各种任务，我会告诉你每个动作将如何帮助你完成任务。我也可以帮助你找到与任务相关的物体。 |
|  |
| 这些是我的评分标准: |
| 2 分: 应该紧接着给定的已完成动作执行的动作。 |
| 1 分: 可以间接执行或支持将获得 2 分的动作。 |
| 0 分: 涉及可见物体但不影响任务的动作。 |
| -1 分: 涉及寻找与任务无关的物品的动作。 |
| -2 分: 涉及拾取或放置不可见物体的动作，即当前状态下无法执行的动作。 |
|  |
| 如抓取、放置、打开、坐下、开关和关闭等动作不能对不可见物体执行。此外，如果之前的完成动作中没有执行打开动作，则无法执行关闭动作。 |
|  |
| 任务描述: 说明你想要完成的任务。 |
| 动作列表: 提供你家中可用的动作列表。 |
| 之前完成的动作: 列出之前已经执行过的动作。 |
| 可见物体: 当前眼睛所能看到的物体。 |
| 拿取: 当前手中持有的物品。 |
|  |
| 现在你可以请求与任务相关的动作得分，并识别当前可见物体中与任务相关的物品。我将以得分/相关物品的格式进行回复。不要包含任何其他回答，仅输出得分和相关物品。 |
|  |
| 回答格式: |
| 得分: 2 |
| 相关物体: 苹果, 香蕉 |
|  |
| 人类: |
| 任务描述: [指令]。 |
| 动作列表: [动作] |
| 之前完成的动作: [已完成的动作]。 |
| 可见物体: [物品] |
| 拿取: [已拿取的物品] |
| [动作]的得分是多少？ |
| 哪些当前可见的物体与任务相关？ |
|  |
| 机器人: |

表 19: 简单的 LLM 奖励评估器提示 $\Phi_{\textnormal{LLM}}$

| CoT 提示 |
| --- |
| 机器人: 你好，我是一个在房子里操作的机器人。你可以让我做各种任务，我会告诉你每个动作在完成任务中有多大的帮助。我也可以帮助你找到与任务相关的物品。 |
|  |
| 这些是我的评分标准: |
| 2 分: 应该紧接着给定的已完成动作执行的动作。 |
| 1 分: 可以间接执行或支持将获得 2 分的动作。 |
| 0 分: 涉及可见物体但不影响任务的动作。 |
| -1 分: 涉及寻找与任务无关的物品的动作。 |
| -2 分: 涉及拾取或放置不可见物体的动作，即当前状态下无法执行的动作。 |
|  |
| 如抓取、放置、打开、坐下、开关和关闭等动作不能对不可见物体执行。此外，如果之前的完成动作中没有执行打开动作，则无法执行关闭动作。 |
|  |
| 任务描述: 说明你想要完成的任务。 |
| 动作列表: 提供你家中可用的动作列表。 |
| 已完成的动作：列出之前已执行的动作。 |
| 可见物体：当前眼睛能看到的物体。 |
| 抓取的物品：当前手中持有的物体。 |
|  |
| 现在你可以请求与任务相关的动作得分，并在当前可见物体中识别与任务相关的物体。 |
|  |
| [示例 1] |
| 人类: |
| 任务描述：找到墙壁架子，然后拿起麦片，再找到冰箱，打开冰箱，把麦片放进冰箱，然后关上冰箱 |
| 已完成的动作：1\. 找到墙壁架子 |
| 可见物体：纸张，麦片，墙壁架子，鼠标，马克杯，奶油小圆面包，饼干 |
| 抓取的物品：无 |
| 机器人: |
| A. 与任务相关的动作：[抓取麦片，找到冰箱，打开冰箱，把麦片放进冰箱，关上冰箱] |
| B. 对任务没有影响的动作（抓取可见物体）：[抓取奶油小圆面包] |
| C. 与任务无关的动作：[剩余查找动作] |
| D. 干扰性动作（当一个物品需要插入但已关闭或开关在未关闭的情况下激活时）：[无] |
| E. 由于不可见物体或未在已完成的动作中抓取而无法执行的动作：[剩余动作] |
| 2 分：在 A 中，未完成的且紧接其后并旨在完成任务的动作是 [抓取麦片]。 |
| 1 分：与获得 2 分的动作类似的动作是 [无]。 |
| 0 分：满足 B 的动作是 [抓取奶油小圆面包]。 |
| -1 分：满足 C 的动作是 [找到书架，找到浴缸，找到沙发，找到浴室柜台，找到床，找到桌子，找到冰箱，找到衣柜抽屉，找到水槽，找到烤面包机，找到微波炉，找到厨房桌子，找到墙壁架子，找到咖啡桌]。 |
| -2 分：D、E 和剩余动作中的动作。 |
| 相关物体：墙壁架子，麦片，冰箱 |
|  |
| [其他示例] |
|  |
|  |
| 人类: |
| 任务描述：[指令]。 |
| 动作列表：[动作] |
| 已完成的动作：[已完成的动作]。 |
| 可见物体：[物品] |
| 抓取的物品： [抓取的物品] |
| [动作] 得几分？ |
|  |
|  |
| 机器人: |

| 表格 20：LLM 奖励估算器 CoT 提示 $\Phi_{\textnormal{LLM}}$ |

| 上下文提示 |
| --- |
| 机器人：你好，我是一个在房子里操作的机器人。你可以让我执行各种任务，我会告诉你每个动作在完成任务中有多大帮助。我还可以帮助你找到与任务相关的物体。 |
|  |
| 这些是我的评分指南： |
| 2 分：应该在给定的已完成动作后执行的动作。 |
| 1 分：可以间接执行或支持获得 2 分的动作的动作。 |
| 0 分：涉及可见物体但对任务没有影响的动作。 |
| -1 分：涉及查找与任务无关物体的动作。 |
| -2 分：涉及拾取或放置不可见物体的动作，即当前状态下无法执行的动作。 |
|  |
| 如抓取、放置、打开、坐下、开关、关闭等行动不能对不可见的物体执行。此外，如果之前没有“打开”动作，则无法执行“关闭”动作。 |
|  |
| 任务描述：陈述你试图完成的任务。 |
| 行动列表：提供你家中可执行的行动列表。 |
| 之前完成的行动：列出已完成的行动。 |
| 可见物品：当前眼睛可见的物品。 |
| 抓取物品：当前手中持有的物品。 |
|  |
| 现在你可以要求为与任务相关的行动打分，并在当前可见的对象中识别与任务相关的对象。 |
|  |
| [示例 1] |
| 人类： |
| 任务描述：找到墙架，然后抓取麦片，再找到冰箱，打开冰箱，把麦片放进冰箱，再关闭冰箱 |
| 之前完成的行动：1. 找到墙上的架子 |
| 可见物品：纸张、麦片、墙架、鼠标、杯子、奶油小圆面包、饼干 |
| 抓取物品：无 |
| 机器人： |
| 抓取麦片：2 |
|  |
| [其他示例] |
|  |
|  |
| 人类： |
| 任务描述：<Instruction> |
| 行动列表：<Actions> |
| 之前完成的行动：<Completed actions> |
| 可见物品：<Visible Objects> |
| 抓取物品：<Grabbed Objects> |
| <Action> 得多少分？ |
|  |
|  |
| 机器人： |

| 表 21：ICL 提示用于 LLM 奖励评估器 $\Phi_{\textnormal{LLM}}$ |

| 上下文提示（2） |
| --- |
| 目标：为了成功实现你的目标，执行以下列出的行动顺序。执行顺序应合乎逻辑，并基于所提供的情况。决策和评分时仅使用指定的行动集中的动作。任何未列出的行动不应考虑在内。 |
|  |
| 评分指南： |
| 2 分（极有益行动）：分配给对直接实现目标至关重要的单一行动，提供即时且显著的好处，并且可以在当前状态下执行的行动。 |
| 1 分（有益行动）：分配给那些对实现目标具有重要意义的步骤或准备性行动，促进显著的进展或准备，且在当前状态下可以执行的行动。 |
| 0 分（中性行动）：分配给那些与目标间接相关或对实现目标贡献较小的行动，基本上是与当前目标无关但仍可以在当前状态下执行的行动。 |
| -1 分（潜在有害行动）：分配给那些虽不直接阻碍目标实现，但可能间接妨碍其实现、浪费时间进行与目标无关的活动，或无法在当前状态下执行的行动。 |
| -2 分（直接有害行动）：分配给那些直接干扰目标实现或产生与预期目标相反效果的行动。 |
|  |
| 任务描述：指定你要实现的目标。 |
| 行动列表：行动列表 |
| 之前完成的动作：列出之前已使用的动作。 |
| 可见物体：目前可见的物体。根据任务描述在这些物体中找到相关物体。 |
| 已抓取：当前手中所持物体。 |
|  |
| [样本 1] |
| 任务描述：找到墙上的架子，然后抓取麦片，再找到冰箱，打开冰箱，把麦片放进冰箱，最后关闭冰箱 |
| 之前完成的动作：1\. 找到墙上的架子 |
| 可见物体：纸张、麦片、墙架、鼠标、马克杯、奶油包子、饼干 |
| 已抓取：无 |
| 回应：抓取麦片：2 |
|  |
| [其他样本] |
|  |
|  |
| 人类： |
| 任务描述： <Instruction> |
| 动作列表： <Actions> |
| 之前完成的动作： <Completed actions> |
| 可见物体： <Visible Objects> |
| 已抓取： <Grabbed Objects> |
| 回应： |

表 22：LLM奖励估计器$\Phi_{\textnormal{LLM}}$的ICL提示（2）

指令 在沙发上坐着享受水果零食。 观察 相框 动作 拿起苹果 执行历史 无 奖励 $r^{C}=-2$ $r^{S}=2$ $r^{T}=-2$ 提示 $\mathcal{P}_{1}$ 2 ✓ 2 ✗ 2 ✓ 提示 $\mathcal{P}_{2}$ 1 ✓ 1 ✗ 1 ✗ 提示 $\mathcal{P}_{3}$ -2 ✓ -2 ✗ -2 ✓ 提示 $\mathcal{P}_{4}$ -2 ✓ -2 ✓ -2 ✓ 提示 $\mathcal{P}_{5}$ 2 ✓ 2 ✗ 2 ✓

表 23：奖励估计如何根据上下文、一致性结构和时间一致性发生变化的示例。在每个基于一致性的奖励（$r^{C}$、$r^{S}$、和$r^{T}$）中，勾号表示预测的奖励对其相应一致性的多数投票做出了贡献。’X’标记表示该奖励被忽略，因为它未通过回溯验证过程（在时间一致性中）或未正确回应特定MDP查询（在结构一致性中）。

指令 享受一份清脆、清爽的健康苹果三明治。 观察 洗洁精、面包片、咖啡壶、炉子、甜椒、水槽、冰箱 动作 将苹果放在面包片上 执行历史 1\. 找到咖啡桌 2\. 拿起苹果 3\. 找到烤面包机 4\. 拿起面包片 奖励 $r^{C}=-2$ $r^{S}=-2$ $r^{T}=2$ 提示 $\mathcal{P}_{1}$ 2 ✓ 2 ✗ 2 ✓ 提示 $\mathcal{P}_{2}$ -2 ✓ -2 ✓ -2 ✓ 提示 $\mathcal{P}_{3}$ 2 ✓ 2 ✓ 2 ✗ 提示 $\mathcal{P}_{4}$ -2 ✓ -2 ✓ -2 ✓ 提示 $\mathcal{P}_{5}$ 1 ✓ 1 ✓ 1 ✓

表 24：奖励估计如何根据上下文、一致性结构和时间一致性发生变化的示例。

指令 给你的猫准备洗澡时间。 观察 猫、浴缸、塔 动作 找到浴缸 执行历史 1\. 找到厨房桌子 2\. 抓起猫 3\. 找到浴缸 4\. 把猫放入浴缸 奖励 $r^{C}=-1$ $r^{S}=2$ $r^{T}=-1$ 提示 $\mathcal{P}_{1}$ -1 ✓ -1 ✓ -1 ✓ 提示 $\mathcal{P}_{2}$ 2 ✓ 2 ✓ 2 ✓ 提示 $\mathcal{P}_{3}$ -1 ✓ -1 ✓ -1 ✓ 提示 $\mathcal{P}_{4}$ 2 ✓ 2 ✓ 2 ✓ 提示 $\mathcal{P}_{5}$ 2 ✓ 2 ✗ 2 ✓

表 25：奖励估计如何根据上下文、一致性结构和时间一致性发生变化的示例。
