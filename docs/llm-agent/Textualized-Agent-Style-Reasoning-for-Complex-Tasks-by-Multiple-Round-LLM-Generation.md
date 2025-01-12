<!--yml

类别：未分类

日期：2025-01-11 12:14:20

-->

# 通过多轮LLM生成的文本化代理风格推理应对复杂任务

> 来源：[https://arxiv.org/html/2409.12411/](https://arxiv.org/html/2409.12411/)

陈亮^(2,3)，冯志凡^(1,3)¹¹脚注标记: 1，刘子鹤²，蒋文彬⁴，

许济南²，陈宇峰²，王勇¹²²脚注标记: 2

¹ 中国科学技术大学，合肥，中国

² 北京交通大学，北京，中国

³ 百度公司，北京，中国

⁴ 北京师范大学人工智能学院，北京，中国

{21120367}@bjtu.edu.cn   平等贡献  通讯作者。

###### 摘要

链式思维提示显著提升了大语言模型的推理能力，但仍然面临三大问题：幻觉问题、有限的可解释性和不可控生成。为了解决这些挑战，我们提出了AgentCOT，一个基于LLM的自主代理框架，它能够通过多轮LLM生成以代理风格的方式解决复杂问题。在每一步中，AgentCOT选择一个动作并执行它，从而得出带有支持证据的中间结果。此外，我们将步骤的索引整合到推理过程中，形成一个图结构，以支持复杂推理逻辑。我们引入了两种新策略来增强AgentCOT的表现。我们进行了大量实验，验证了我们的方法在六个常见基准上的有效性。结果表明，我们的方法在当前的竞争方法中带来了显著的提升。

通过多轮LLM生成的文本化代理风格推理应对复杂任务

陈亮^(2,3)^†^†感谢:   平等贡献，冯志凡^(1,3)¹¹脚注标记: 1，刘子鹤²，蒋文彬⁴，许济南²^†^†感谢:   通讯作者，陈宇峰²，王勇¹²²脚注标记: 2 ¹ 中国科学技术大学，合肥，中国 ² 北京交通大学，北京，中国 ³ 百度公司，北京，中国 ⁴ 北京师范大学人工智能学院，北京，中国 {21120367}@bjtu.edu.cn

## 1 引言

![参考说明](img/94ea21074829940f81d496bde8f325a7.png)

图1：链式思维（COT）和自主代理的框架。COT通常是一个文本段落，而自主代理可以通过多次响应来解决问题。

大型语言模型（LLM）在许多任务中展现了卓越的表现，如姚等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib27)）；王等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib22)），这激发了人们考虑利用LLM来解决具有挑战性和复杂的问题。值得强调的是，复杂推理任务受到了广泛关注。与典型的自然语言处理（NLP）任务不同，执行复杂推理需要明确展示分析过程，而不仅仅是呈现答案，即最近提出的链式思维（COT）提示方法（魏等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib24)））。关于COT的研究，如郝等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib9)）；谢等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib26)）；刁等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib5)）显著提升了LLM的推理能力，并取得了最先进的成果。

然而，链式思维提示存在缺陷，主要受到以下三个限制：i) 幻觉问题（姚等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib27)）；黄等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib10)）），这是COT性能下降的主要原因。COT中的幻觉推理严重，导致推理过程看似合理，但缺乏事实依据。ii) 受限的可解释性。尽管COT的目标是解释答案是如何得出的，但通常通过一段文字呈现，而不是以更具逻辑性的格式进行展示。iii) 不可控的生成。由于COT是一次性生成的推理过程，包含大量的标记，解码中的任何错误都会导致错误的延续（陈等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib3)））贯穿所有后续的解码步骤。

本文的目标是基于自主代理框架解决复杂推理任务。如图[1](https://arxiv.org/html/2409.12411v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation")所示，不同于典型的链式思维方法一次性生成分析过程，基于代理的方法自然体现了逐步解决问题的思路，在迭代过程中每一步都解决具体的子问题。我们遵循姚等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib27)）提出的代理设置，并设计了适当的提示，驱动LLM按照指令进行操作。在每一步，LLM代理会检测环境的变化并根据当前状态作出响应。生成的响应将导致环境变化，并再次促使代理作出响应，直到问题解决。

我们进一步提出了AgentCOT，以优化上述智能体设置，更好地适应推理任务。具体来说，当感知到环境变化时，AgentCOT首先从预定义的动作集合中选择一个动作$a$，并为当前问题提供该动作$a_{des}$的具体描述。接着，AgentCOT执行该动作并产生一个中间结果$R_{inter}$，同时为其结论提供支持证据$E_{inter}$。{$a$，$a_{des}$，$E_{inter}$，$R_{inter}$}构成一个原子响应状态，其中$a$和$a_{des}$可以视为当前子问题的计划，而$E_{inter}$可以看作是最小逻辑单元的COT。这种组织格式增强了推理过程的可解释性。通过这种方式，我们可以在子问题层面进行操作，如反思和重新解码，从而实现相对可控的推理过程。我们提出了增强的自一致性，以提高每个状态的质量，有效防止错误延续和幻觉问题。此外，我们将状态索引集成到推理过程中，形成一个隐式图结构，这能够表示更丰富的推理逻辑。

我们在六个常见基准上评估了所提出的方法，涵盖三种类型。从结果来看，AgentCOT在所有数据集上表现出竞争力的性能（$\S$ [5](https://arxiv.org/html/2409.12411v1#S5 "5 Experimental Results ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation")）。我们进一步进行实验，比较COT框架和智能体框架（$\S$ [6.1](https://arxiv.org/html/2409.12411v1#S6.SS1 "6.1 COT framework or Agent framework? ‣ 6 Discussion ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation")），并进行错误分析（$\S$ [6.2](https://arxiv.org/html/2409.12411v1#S6.SS2 "6.2 Error Analysis ‣ 6 Discussion ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation")）和案例研究（$\S$ [6.3](https://arxiv.org/html/2409.12411v1#S6.SS3 "6.3 Case Study ‣ 6 Discussion ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation")），以提供AgentCOT的具体视角。最后，我们进行消融研究，以探索模型结构（$\S$ [6.4](https://arxiv.org/html/2409.12411v1#S6.SS4 "6.4 Ablation Study on AgentCOT Structure ‣ 6 Discussion ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation")，$\S$ [6.5](https://arxiv.org/html/2409.12411v1#S6.SS5 "6.5 AgentCOT with Enhanced Self-Consistency ‣ 6 Discussion ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation")）。

总之，我们的贡献有三点：

+   •

    我们提出了AgentCOT，一个基于LLM的自主代理框架，通过多轮LLM生成以代理风格处理推理任务，并在不同任务中表现出良好的性能。

+   •

    为了更好地解决推理任务，我们将代理在每一步的响应组织为一个包含丰富信息的状态，其中包括行动、行动描述、支持证据和中间结果。此外，提出了AgentCOT的两种增强策略以提升性能。

+   •

    实验表明，我们的方法能够显著提高当前的最先进技术，并且在各种数据集和模型中都表现出良好的效果。

## 2 相关工作

### 2.1 基于LLM的自主代理

大型语言模型（LLM）提供了解决现实世界中许多具有挑战性任务的能力，如决策、推理和规划，这激发了基于LLM的人类智能自主代理的发展（Wang等，2023年）([2023](https://arxiv.org/html/2409.12411v1#bib.bib22))。近年来，基于LLM的智能代理应用出现了爆炸性增长。例如，Park等人（2023年）([2023](https://arxiv.org/html/2409.12411v1#bib.bib16))在《模拟人生》中实例化生成代理以实现动态计划行为；Li等人（2023年）([2023](https://arxiv.org/html/2409.12411v1#bib.bib12))提出了一种新的交互式代理框架，以提供对认知过程的深入理解。一些工作集中在决策代理上，以便轻松使用工具，例如ToolLearning（Qin等人，2023年）([2023](https://arxiv.org/html/2409.12411v1#bib.bib18))、Reflexion（Shinn等人，2023年）([2023](https://arxiv.org/html/2409.12411v1#bib.bib21))、Toolformer（Schick等人，2023年）([2023](https://arxiv.org/html/2409.12411v1#bib.bib19))、HuggingGPT（Shen等人，2023年）([2023](https://arxiv.org/html/2409.12411v1#bib.bib20))、WebGPT（Nakano等人，2021年）([2021](https://arxiv.org/html/2409.12411v1#bib.bib15))。

### 2.2 多步推理

LLM 的出现使得中间推理步骤可以以自然语言的形式展示，Zhang 等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib29)）。目前的研究中，COT 的调查可以分为三个关键维度：i) 在上下文学习（ICL）演示中的样本选择，Mann 等人（[2020](https://arxiv.org/html/2409.12411v1#bib.bib14)）。选择的视角包括多样性，Zhang 等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib29)），最有帮助和最具信息量的 Diao 等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib5)），相关的以及互补的 Ye 等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib28)）。ii) COT 的精炼。Fu 等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib6)）展示了具有更高推理复杂性的优秀 COT。Zhou 等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib30)）提出了一种流程方法，首先提供一个计划将源问题分解为几个子问题，然后按顺序解决这些子问题。随后的工作，Xie 等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib26)）；Hao 等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib9)）；Besta 等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib1)）放弃了计划阶段，以防止计划错误对问题解决的影响。他们始终展示自我评估策略来提高每个步骤的正确性，例如随机束搜索 Xie 等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib26)），自信 Diao 等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib5)）和自一致性 Wang 等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib23)）。iii) 生成 COT 后的反思和验证，旨在基于 LLM 识别生成的 COT 中的问题，Wang 等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib23)）；Xie 等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib26)）；Kim 等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib11)）或额外工具 Gou 等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib8)）。

![请参阅标题](img/1845ffc62f103524a092952de74aeaf7.png)

图 2：我们的方法 AgentCOT 概述。AgentCOT 执行过程的一个实例在图的顶部进行了可视化。在每个步骤中，LLM 代理感知环境变化并依次生成行动、行动描述、中间证据和中间结果。这些信息通过有效的组织回应环境，并导致环境再次发生变化。我们还提供了有关隐式状态图、自我评估解码和增强集成策略的一些细节，位于图的底部。

## 3 方法

我们提出了AgentCOT，将LLM视为一个自主代理，执行文本化的代理风格推理，如图[2](https://arxiv.org/html/2409.12411v1#S2.F2 "Figure 2 ‣ 2.2 Multi-Step Reasoning ‣ 2 Related Work ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation")所示。该模型由两个重要组成部分构成：1）Agent Solving（§3.1），它是AgentCOT的基础框架，用于解决推理问题。2）AgentCOT Enhancement（§3.2），它涉及多种技术以提高我们方法的有效性。下面显示了AgentCOT的简化提示，完整的提示方案在附录[A.1](https://arxiv.org/html/2409.12411v1#A1.SS1 "A.1 Prompt Design ‣ Appendix A Example Appendix ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation")中给出。![[未标注图片]](img/bd78edb759a23465f06a091fd6798437.png)

### 3.1 AgentCOT求解

#### 作为代理的LLM

受到以往将推理与行为结合的工作启发（Yao等，[2022](https://arxiv.org/html/2409.12411v1#bib.bib27)），我们开发了用于推理任务的代理设置，该设置在感知到环境$E$的变化后与环境互动，并采取响应$E$的行动$a$。具体而言，初始环境$E$仅包含原始查询$Q^{0}$。在第$i$步，代理感知到$E$中的变化，状态为$Q^{i}$，从而驱使其执行动作$a^{i}$，该动作属于动作集$\mathcal{A}$，并产生结果$r^{i}$。$Q^{i}$和$r^{i}$被连接成$Q^{i+1}$，从而导致环境的变化和代理的再次响应，直到问题解决为止。下面的提示旨在使LLM充当一个代理。![[未标注图片]](img/12be93a1ded683a63ff21b7e8d1f4f86.png)

#### 动作空间与动作选择

我们方法中的动作空间由与问答推理任务相关的有限一组动作$\mathcal{A}$构成。我们将Wolfson等人（[2020](https://arxiv.org/html/2409.12411v1#bib.bib25)）提出的所有动作纳入我们的动作集，其中包含13种操作符类型：选择、过滤、投影、聚合、分组、最优、比较、并集、交集、丢弃、排序、布尔和算术。我们进一步定义了动作Describe，用于解释名词、状态或动作；动作Evaluate用于评估生成信息的质量。当检测到环境中的变化时，代理将从$\mathcal{A}$中选择一个合适的动作$a$。下面展示了该动作集的提示。![[未标注图片]](img/855fd839abda58a9e9b552b363dc318b.png)

在解决推理问题时，AgentCOT不仅提供动作选项，还提供所选动作$a$的详细描述$a_{des}$，确保在执行过程中有明确的指令。更重要的是，考虑到我们的动作集$\mathcal{A}$可能不完整且某些动作不需要定义，我们还允许代理不选择任何动作，而只提供需要执行的操作的详细描述。

#### 动作执行

AgentCOT允许所选动作的执行者可以是LLM本身或其他外部工具，如搜索引擎或计算器。当代理与环境$E$交互并提供一个动作$a$时，执行者将执行该动作$a$并产生相应的结果。当使用LLM作为执行者时，我们要求模型必须提供中间证据$E_{inter}$和中间结果$R_{inter}$。$E_{inter}$指的是动作和动作描述生成中间结果的分析过程，这可以看作是解决当前子问题的最小思维链。$R_{inter}$指的是执行动作指令后得到的结果。其他工具作为执行者仅需提供中间结果$R_{inter}$。

#### 丰富状态与隐式状态图

如上所述，在每一步$i$中，在感知到环境变化后，AgentCOT将生成一个信息丰富的状态$S^{i}$，包含动作、动作描述、中间证据和中间答案：

|  | $\displaystyle S^{i}=\{a^{i},a_{des}^{i},E_{inter}^{i},R_{inter}^{i}\}$ |  | (1) |
| --- | --- | --- | --- |

实验结果表明，一个信息丰富的状态能够支持更优的表现。

此外，尽管状态是一个接一个地生成的，但这并不意味着状态之间的相互关系是链式的。例如，第一状态和第二状态是独立的，而第三个节点同时依赖于第一和第二状态，如图[2](https://arxiv.org/html/2409.12411v1#S2.F2 "图2 ‣ 2.2 多步骤推理 ‣ 2 相关工作 ‣ 文本化的代理式推理用于复杂任务的多轮LLM生成")底部的第一个图所示。为了描述这种复杂的推理模式，我们将状态索引整合到状态本身，从而加强了状态之间的联系。具体而言，状态索引主要存在于$a_{des}^{i}$和$E_{inter}^{i}$中。当$E_{inter}^{i}$需要包含$E_{inter}^{j}(j<i)$中的信息时，$E_{inter}^{i}$中的相应信息将以‘# j’的形式写入，或者在相应的信息后面加上’(# j)’。因此，实际上，在解决问题时，AgentCOT包含了一种隐式的图形结构。

#### 迭代过程

在第$i$步生成一个信息丰富的状态$S^{i}$之后，$E$中的问题将更新如下：

|  | $\displaystyle Q^{i+1}=Q^{i}+S^{i}$ |  | (2) |
| --- | --- | --- | --- |

这将导致代理的响应再次生成。AgentCOT 迭代执行，直到生成最终结果。最终结果通常是最后一次操作的结果，以“因此，最终答案是……”的形式呈现。支持 LLM 执行代理式推理的详细提示如下所示。 ![[无标题图片]](img/244c7d16335d18968254ac807b2114e4.png)

### 3.2 AgentCOT 增强

如上所述，AgentCOT 展示了明确的多步骤推理过程。受到 Xie 等人 ([2023](https://arxiv.org/html/2409.12411v1#bib.bib26)) 启发，我们提出的增强策略基于每一步生成的结果。如图[2](https://arxiv.org/html/2409.12411v1#S2.F2 "图 2 ‣ 2.2 多步骤推理 ‣ 2 相关工作 ‣ 通过多轮 LLM 生成的文本化代理式推理用于复杂任务")底部的第二张图所示，AgentCOT 可以评估每个生成状态的质量，并决定是继续推理还是返回重新生成。评估和反思本质上为当前 LLM 解码策略中的不可逆问题提供了解决方案。

AgentCOT 在两个层次上确保状态质量。第一个是子问题层次。我们采用发散性思维策略，允许多个不同的推理路径。具体而言，AgentCOT 每次生成多个响应。然后，我们通过考虑操作和中间结果进行集成学习，以选择最佳响应，如图[2](https://arxiv.org/html/2409.12411v1#S2.F2 "图 2 ‣ 2.2 多步骤推理 ‣ 2 相关工作 ‣ 通过多轮 LLM 生成的文本化代理式推理用于复杂任务")底部的第三张图所示。第二个是全局问题层次。AgentCOT 易于转化为具有丰富信息的 COT 模式。在每个解码步骤中，我们鼓励 AgentCOT 生成剩余的完整推理过程，这意味着 AgentCOT 会在此时生成最终结果，以帮助评估生成的状态。因此，在每个响应中，AgentCOT 同时考虑两个层次，即包含操作、中间结果和建议的最终结果，以提供最佳状态。

## 4 实验设置

### 4.1 数据集和评估指标

我们在六个常见的基准上进行实验，这些基准可以分为三类：（1）算术推理，包括 GSM8K Cobbe 等人 ([2021](https://arxiv.org/html/2409.12411v1#bib.bib4)) 和 AQuA Ling 等人 ([2017](https://arxiv.org/html/2409.12411v1#bib.bib13))。（2）常识推理，包括 CommonsenseQA Geva 等人 ([2021](https://arxiv.org/html/2409.12411v1#bib.bib7)) 和 Date Wei 等人 ([2022](https://arxiv.org/html/2409.12411v1#bib.bib24))。（3）基于事实的多跳问答，包括 Bamboogle Press 等人 ([2023](https://arxiv.org/html/2409.12411v1#bib.bib17)) 和 Compositional Celebrities Press 等人 ([2023](https://arxiv.org/html/2409.12411v1#bib.bib17))。表[1](https://arxiv.org/html/2409.12411v1#S4.T1 "Table 1 ‣ 4.3 Baselines. ‣ 4 Experimental Setups ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation")展示了它们的详细统计数据。根据之前的研究 Wei 等人 ([2022](https://arxiv.org/html/2409.12411v1#bib.bib24))；Zhang 等人 ([2022](https://arxiv.org/html/2409.12411v1#bib.bib29))，我们报告所有数据集的准确率作为评估指标。

### 4.2 实现。

对于大型语言模型，我们主要利用两种版本的 GPT Brown 等人 ([2020](https://arxiv.org/html/2409.12411v1#bib.bib2))，text-davinci-002 和 gpt-3.5-turbo，来进行实验。在我们的实现中，我们从训练数据集中选择一些示例（如果可用）来形成演示，供上下文学习使用 Brown 等人 ([2020](https://arxiv.org/html/2409.12411v1#bib.bib2))。示例数量参考了之前的研究 Wei 等人 ([2022](https://arxiv.org/html/2409.12411v1#bib.bib24))；Diao 等人 ([2023](https://arxiv.org/html/2409.12411v1#bib.bib5))。在推理阶段的超参数设置中，温度从 {0.8, 0.9, 1.0, 1.1, 1.2} 中选择，top-p 值从 {0.8, 0.9, 1.0} 中选择。执行增强策略时，LLM 最大调用次数（用于 AgentCOT）设置为 {3, 4, 5}。

### 4.3 基准。

我们将 AgentCOT 与以下几个基准进行比较：COT Wei 等人 ([2022](https://arxiv.org/html/2409.12411v1#bib.bib24))，这是首篇提出链式推理（chain-of-thought）的论文。COT-SC Wang 等人 ([2022](https://arxiv.org/html/2409.12411v1#bib.bib23)) 基于自一致性解码策略生成 COT。Auto-COT Zhang 等人 ([2022](https://arxiv.org/html/2409.12411v1#bib.bib29)) 展示了一种自动化 COT 提示方法，考虑了演示中的多样性。Complex-COT Fu 等人 ([2022](https://arxiv.org/html/2409.12411v1#bib.bib6)) 更倾向于选择包含更多推理步骤的 COT。Random-COT Diao 等人 ([2023](https://arxiv.org/html/2409.12411v1#bib.bib5)) 随机从训练集选择示例来形成演示。PAL Xie 等人 ([2023](https://arxiv.org/html/2409.12411v1#bib.bib26)) 引入了自我评估引导的束搜索来增强 COT。

|  | GSM8K | AQUA | CSQA | Date | Bamboogle | CC |
| --- | --- | --- | --- | --- | --- | --- |
| 训练 | 7,473 | 254 | 12,247 | - | - | - |
| Dev | - | 254 | 1,221 | - | - | - |
| 测试 | 1,319 | 404 | 1,140 | 369 | 125 | 8,693 |

表1：数据集统计信息。

| 模型 | 方法 | GSM8K | AQuA | CSQA | Date | Bamboogle | CC |
| --- | --- | --- | --- | --- | --- | --- | --- |
| text-davinci-02 | COT Wei et al. ([2022](https://arxiv.org/html/2409.12411v1#bib.bib24)) | 46.9* | 35.8* | 73.5* | 52.1* | 32.8 | 44.3 |
| COT-SC Wang et al. ([2022](https://arxiv.org/html/2409.12411v1#bib.bib23)) | - | - | - | - | 36.0 | 46.2 |
| Auto-COT Zhang et al. ([2022](https://arxiv.org/html/2409.12411v1#bib.bib29)) | 47.9* | 36.5* | 74.4* | - | - | - |
| Complex-COT Fu et al. ([2022](https://arxiv.org/html/2409.12411v1#bib.bib6)) | 55.4* | 37.8 | 73.7 | 59.0 | 48.8 | 47.7 |
| Random-COT Diao et al. ([2023](https://arxiv.org/html/2409.12411v1#bib.bib5)) | 63.9 | 44.1* | 74.5* | 62.2 | 50.4 | 47.2 |
| PAL Xie et al. ([2023](https://arxiv.org/html/2409.12411v1#bib.bib26)) | 58.1 | 35.2 | 74.9 | 59.6 | 51.2 | 54.7 |
| Agent-COT（我们的） | 67.1 | 38.6 | 78.4 | 64.1 | 52.0 | 57.6 |
| gpt-3.5-turbo | COT Wei et al. ([2022](https://arxiv.org/html/2409.12411v1#bib.bib24)) | 73.8 | 57.0 | 71.3 | 58.2 | 56.8 | 55.2 |
| COT-SC Wang et al. ([2022](https://arxiv.org/html/2409.12411v1#bib.bib23)) | 75.4 | 58.6 | 72.9 | 59.8 | 58.3 | 57.1 |
| Complex-COT Fu et al. ([2022](https://arxiv.org/html/2409.12411v1#bib.bib6)) | 71.9 | 57.8 | 72.9 | 58.8 | 55.2 | 57.6 |
| Random-COT Diao et al. ([2023](https://arxiv.org/html/2409.12411v1#bib.bib5)) | 75.3 | 55.5 | 73.7 | 61.2 | 56.8 | 56.6 |
| PAL Xie et al. ([2023](https://arxiv.org/html/2409.12411v1#bib.bib26)) | 72.7 | 55.5 | 64.7 | 62.6 | 56.8 | 55.3 |
| Agent-COT（我们的） | 79.9 | 59.8 | 79.5 | 64.4 | 58.4 | 58.5 |

表2：与以往方法在不同数据集和三种任务类型上的整体结果比较。*表示该结果来源于原文。

## 5 实验结果

我们在表[2](https://arxiv.org/html/2409.12411v1#S4.T2 "Table 2 ‣ 4.3 Baselines. ‣ 4 Experimental Setups ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation")中展示了我们方法与强基线的主要实验结果，其中包含六个数据集，涵盖三种类型和两个版本的 GPT 模型。从结果中，我们可以发现，我们的 AgentCOT 方法在大多数数据集和不同版本的 GPT 中都取得了最佳表现。AgentCOT 相比 COT Wei 等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib24)）平均分别在 text-davinci-002 和 gpt-3.5-turbo 上提高了 12.06% 和 4.70% 的准确率，这验证了代理框架相对于传统 COT 的优越性。由于升级版本的 gpt-3.5-turbo 比 text-davinci-002 具有更好的模型能力，我们的方法和基线在 gpt-3.5-turbo 上的结果更高，特别是在算术推理数据集 GSM8K 和 AQuA 上。对于我们的方法，AgentCOT 在 CSQA、Date 和 Bamboogle 上对两个版本的 GPT 模型表现几乎相当，表明我们精心设计的代理框架有效地激活了模型的解决问题能力，从而弥补了原始能力的差距。通过在不同类型的数据集上比较 AgentCOT 与基线的表现，我们可以看到，AgentCOT 在提升效果上存在显著差异。以 text-davinci-002 上的结果为例，整体而言，AgentCOT 在多跳问答数据集上的提升最大（平均提高 +16.7%），其次是算术推理（平均提高 +11.5%），最后是常识推理（平均提高 +7.9%）。一个合理的解释是，多跳问答和算术推理在逐步执行上有明确的边界，可以通过 AgentCOT 的问题分解能力完成。两个常识推理数据集由于其不同的性质，结果也显示出显著的差异。通过进一步分析，CSQA 是一个关于日常生活场景推理的数据集，而 Date 则是关于日期计算的，更适合 AgentCOT 框架的逐步问题解决方法。

## 6 讨论

在本节中，我们进行了一系列详细的研究，以探索 AgentCOT 的能力。

### 6.1 COT 框架还是 Agent 框架？

我们的方法AgentCOT可以退化为具有丰富信息的一般COT范式，我们称之为EnrichCOT。图[3](https://arxiv.org/html/2409.12411v1#S6.F3 "图 3 ‣ 6.1 COT框架还是Agent框架？ ‣ 6 讨论 ‣ 通过多轮LLM生成的文本化代理风格推理用于复杂任务")展示了在text-davinci-002和gpt-3.5-turbo上，COT框架和代理框架在六个数据集上的性能比较。从结果中我们可以发现，在大多数数据集中，AgentCOT的表现明显优于EnrichCOT。一种合理的解释是，基于代理框架的AgentCOT提供了更受控的推理过程，实施有效的策略以确保每一步生成状态的质量。我们还注意到，在gpt-3.5-turbo上，EnrichCOT在GSM8K和AQuA数据集上表现出更高的准确率，这表明显式问题分解可能会干扰算术推理任务的思维过程。与COT Wei等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib24)）相比，EnrichCOT展示了更优的性能，表明丰富的信息（如行动、 中间证据和中间结果）有助于推理。

![参考说明](img/68603af7a86303c6efe15a41c6e525ad.png)![参考说明](img/63419582e39a07be0ba4bc536811c168.png)

图3：COT范式与代理范式的性能比较。’COT‘表示Wei等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib24)）提出的链式推理。’EnrichCOT‘则考虑将AgentCOT的推理过程视为一次性生成COT。

![参考说明](img/12966a086b01f72784178c331f956d86.png)

图4：用于探索AgentCOT能力的错误分析。给出了在推理过程中出现问题分解错误（‘split’）和子问题解决错误（‘solve’）的示例百分比，涵盖六个数据集。

### 6.2 错误分析

我们进行了错误分析，以探讨基于模型gpt-3.5-turbo的我们方法在六个数据集上的能力不足。具体而言，我们将导致错误推理过程的因素分为两组：模型缺乏问题分解能力（即，行动和行动描述的错误）和模型缺乏子问题解决能力（即，中间证据和答案的错误）。结果如图[4](https://arxiv.org/html/2409.12411v1#S6.F4 "图 4 ‣ 6.1 COT框架还是Agent框架？ ‣ 6 讨论 ‣ 通过多轮LLM生成的文本化代理风格推理用于复杂任务")所示。

从图中呈现的样本百分比可以得出结论：1) 总体而言，AgentCOT在问题分解方面的表现优于其解决子问题的能力。进一步研究表明，解决子问题时的错误主要包括计算错误和知识检索不准确，这些问题可以通过引入外部工具来优化。2) AgentCOT的能力在不同数据集类型上表现出变化。在常识推理任务（CSQA和Date）和多跳问答任务（Bamboogle和CC）中，几乎不会发生问题分解错误。然而，由于算术推理问题更加复杂，AgentCOT在GSM8K和AQuA中的问题分解表现适中，但仍优于子问题求解。

| 设置 | GSM8K | AQuA | CSQA | Date | CC |
| --- | --- | --- | --- | --- | --- |
| 完整模型 | 79.9 | 59.8 | 79.5 | 64.4 | 58.5 |
| --- | --- | --- | --- | --- | --- |
| w/o   Action | 78.5 | 52.5 | 74.1 | 60.7 | 51.6 |
| w/o   ActionD | 78.4 | 55.0 | 77.0 | 61.7 | 40.7 |
| w/o   IEvidence | 71.2 | 50.7 | 59.0 | 60.4 | 56.2 |

表3：AgentCOT框架的消融研究。“ActionD”代表动作描述，“IEvidence”指中间证据。我们在gpt-3.5-turbo上进行了实验。

### 6.3 案例研究

我们列出了三个案例来提供对不同隐式图结构的具体视角，如图[5](https://arxiv.org/html/2409.12411v1#S6.F5 "Figure 5 ‣ 6.3 Case Study ‣ 6 Discussion ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation")所示。案例1中的隐式图是一个基本的线性结构，而案例2和案例3中的图则表现了不同的节点连接方式。具体来说，第一个案例选自AQuA数据集。AgentCOT依赖于前一步的计算来获得每一步的结果。第二个案例选自CSQA数据集。在给定的问题中，AgentCOT独立分析每个选项，然后将这些分析结果结合起来得出最终答案。第三个案例选自AQuA数据集。AgentCOT首先计算A和B股票分别不增长的概率，然后计算它们同时发生的概率。最后，根据计算结果，AgentCOT选择正确的选项。多样化的图结构反映了AgentCOT在解决问题时采用的多种思维方式。这些隐式图提供了双重优势。首先，它增强了推理过程的可解释性，使推理路径更容易理解。其次，它还通过在推理过程中更加明确地组织和使用信息，强化了模型本身。

![参见标题](img/259493af180c549a46de25c430215372.png)

图5：案例研究。我们只提供行动描述以便于推理过程的清晰表达，省略了其他信息。隐式图中的节点$i$对应于AgentCOT推理过程中的步骤$i$，’#$i$’表示信息来自步骤$i$。

### 6.4 AgentCOT结构的消融研究

我们进行了一项消融研究，探索行动、行动描述和中间证据对AgentCOT性能的影响。我们在gpt-3.5-turbo版本上对五个不同的基准进行了实验，并在表[3](https://arxiv.org/html/2409.12411v1#S6.T3 "表3 ‣ 6.2 错误分析 ‣ 6 讨论 ‣ 通过多轮LLM生成的文本化Agent风格推理用于复杂任务")中报告了结果。从表中可以看出，AgentCOT缺少行动时，结果平均下降了4.95%，而缺少行动描述时，性能下降约为5.89%。结果表明，在推理过程中，行动和这些行动的描述都是必不可少的。当AgentCOT缺少行动描述时，模型性能显著下降，相较之下缺少行动的影响较小，因为不同问题之间的行动集是相同的，而行动描述是特定于问题的，可以引导问题解决。缺少中间证据的AgentCOT与谢等人（[2023](https://arxiv.org/html/2409.12411v1#bib.bib26)）提出的方法类似，导致准确率下降了8.93%。实际上，中间证据可以视为子问题的推理过程。这种思维链条有助于获得正确的结果。

### 6.5 带增强自一致性的AgentCOT

在这一小节中，我们评估了我们提出的增强自一致性的效果。我们在图[6](https://arxiv.org/html/2409.12411v1#S6.F6 "图6 ‣ 6.5 带增强自一致性的AgentCOT ‣ 6 讨论 ‣ 通过多轮LLM生成的文本化Agent风格推理用于复杂任务")中展示了在CC和AQuA上的结果。由于解码策略的影响，LLM每次生成的输出都不同。王等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib23)）提出的方法通过集成策略选择最终高置信度的答案，可以提高准确性。带有增强自一致性策略的AgentCOT考虑了一个精细的层面，通过集成行动和中间结果来保证每个生成步骤的质量。从结果来看，带有增强自一致性策略的AgentCOT显著提高了模型性能。

![参见说明文字](img/800c533859ee0391cd5f5e4f429a2c32.png)

(a) 在CC上的结果。

![参见说明文字](img/f821e352fffa93c95cc8d880e9c94251.png)

(b) 在AQuA上的结果。

图6：自一致性方法的结果。“w / o”表示没有自一致性策略的AgentCOT。“w SC”和“w E-SC”分别表示采用王等人（[2022](https://arxiv.org/html/2409.12411v1#bib.bib23)）和我们提出的自一致性策略的AgentCOT。

## 7 结论

在本研究中，我们提出了AgentCOT，以缓解推理任务中链式思维面临的关键问题：幻觉问题、受限的可解释性和不可控的生成。AgentCOT采用逐步响应的方法，以逐步的方式解决问题。每个响应包含行动、行动描述、支持证据和中间结果。对六个常见数据集的实验结果表明，AgentCOT在当前的竞争基准上表现出了令人鼓舞的性能。大规模语言模型的出现激发了研究人员解决更具挑战性的任务。本研究将大规模语言模型作为一个自主代理来解决推理任务。我们希望这项工作能激发其他研究。

## 8 限制

在本文中，AgentCOT通过多轮大规模语言模型生成实现了最先进的性能。此外，AgentCOT的增强策略实现也需要反复调用大规模语言模型，导致时间和资源消耗较高。另一个限制是，AgentCOT在自主执行“评估”这一行动时存在困难，需要开发程序来执行此操作。未来的研究应集中在如何设计提示，使代理能够获得这一能力。

## 参考文献

+   Besta et al. (2023) Maciej Besta, Nils Blach, Ales Kubicek, Robert Gerstenberger, Lukas Gianinazzi, Joanna Gajda, Tomasz Lehmann, Michal Podstawski, Hubert Niewiadomski, Piotr Nyczyk 等人. 2023. [思维图谱：用大规模语言模型解决复杂问题](https://arxiv.org/abs/2308.09687). *arXiv预印本 arXiv:2308.09687*.

+   Brown et al. (2020) Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell 等人. 2020. [语言模型是少样本学习者](https://arxiv.org/pdf/2005.14165). *神经信息处理系统进展*。

+   Chen et al. (2022) Wenhu Chen, Xueguang Ma, Xinyi Wang, 和 William W Cohen. 2022. [思维引导程序：解构数值推理任务中的计算与推理](https://openreview.net/pdf/c2ca9d768b16cf1fac6295f41752506947edbba5.pdf). *arXiv预印本 arXiv:2211.12588*.

+   Cobbe et al. (2021) Karl Cobbe, Vineet Kosaraju, Mohammad Bavarian, Mark Chen, Heewoo Jun, Lukasz Kaiser, Matthias Plappert, Jerry Tworek, Jacob Hilton, Reiichiro Nakano 等人. 2021. [训练验证器解决数学文字问题](https://arxiv.org/pdf/2110.14168). *arXiv预印本 arXiv:2110.14168*.

+   Diao 等人（2023）Shizhe Diao, Pengcheng Wang, Yong Lin 和 Tong Zhang。2023. [通过链式思维进行主动提示以提高大型语言模型性能](https://arxiv.org/abs/2302.12246)。*arXiv 预印本 arXiv:2302.12246*。

+   Fu 等人（2022）Yao Fu, Hao Peng, Ashish Sabharwal, Peter Clark 和 Tushar Khot。2022. [基于复杂度的多步推理提示](https://arxiv.org/pdf/2210.00720)。*arXiv 预印本 arXiv:2210.00720*。

+   Geva 等人（2021）Mor Geva, Daniel Khashabi, Elad Segal, Tushar Khot, Dan Roth 和 Jonathan Berant。2021. [亚里士多德用过笔记本电脑吗？带有隐性推理策略的问答基准](https://arxiv.org/pdf/2101.02235)。*ACL*。

+   Gou 等人（2023）Zhibin Gou, Zhihong Shao, Yeyun Gong, Yelong Shen, Yujiu Yang, Nan Duan 和 Weizhu Chen。2023. [Critic：大型语言模型通过工具互动批评可以自我纠正](https://arxiv.org/abs/2305.11738)。*arXiv 预印本 arXiv:2305.11738*。

+   Hao 等人（2023）Shibo Hao, Yi Gu, Haodi Ma, Joshua Hong, Zhen Wang, Daisy Wang 和 Zhiting Hu。2023. [与语言模型推理就是与世界模型规划](https://doi.org/10.18653/v1/2023.emnlp-main.507)。发表于 *2023年自然语言处理实证方法会议论文集*，第8154–8173页，新加坡。计算语言学会。

+   Huang 等人（2023）Lei Huang, Weijiang Yu, Weitao Ma, Weihong Zhong, Zhangyin Feng, Haotian Wang, Qianglong Chen, Weihua Peng, Xiaocheng Feng, Bing Qin 等人。2023. [大型语言模型中的幻觉调查：原理、分类、挑战与未解问题](https://arxiv.org/pdf/2311.05232)。*arXiv 预印本 arXiv:2311.05232*。

+   Kim 等人（2023）Geunwoo Kim, Pierre Baldi 和 Stephen McAleer。2023. [语言模型可以解决计算机任务](https://arxiv.org/abs/2303.17491)。*arXiv 预印本 arXiv:2303.17491*。

+   Li 等人（2023）Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin 和 Bernard Ghanem。2023. [Camel：用于“大规模语言模型社会”中“心智”探索的沟通代理](https://arxiv.org/pdf/2303.17760)。*arXiv 预印本 arXiv:2303.17760*。

+   Ling 等人（2017）Wang Ling, Dani Yogatama, Chris Dyer 和 Phil Blunsom。2017. [通过推理生成进行程序归纳：学习解决和解释代数文字问题](https://doi.org/10.18653/v1/P17-1015)。发表于 *第55届计算语言学会年会论文集（第1卷：长篇论文）*，第158–167页，加拿大温哥华。计算语言学会。

+   Mann 等人（2020）Ben Mann, N Ryder, M Subbiah, J Kaplan, P Dhariwal, A Neelakantan, P Shyam, G Sastry, A Askell, S Agarwal 等人。2020. [语言模型是少量学习者](https://arxiv.org/pdf/2005.14165)。*arXiv 预印本 arXiv:2005.14165*。

+   Nakano 等人（2021）Reiichiro Nakano、Jacob Hilton、Suchir Balaji、Jeff Wu、Long Ouyang、Christina Kim、Christopher Hesse、Shantanu Jain、Vineet Kosaraju、William Saunders 等人。2021年。[Webgpt：带有人类反馈的浏览器辅助问答系统](https://arxiv.org/pdf/2112.09332)。*arXiv 预印本 arXiv:2112.09332*。

+   Park 等人（2023）Joon Sung Park、Joseph C O’Brien、Carrie J Cai、Meredith Ringel Morris、Percy Liang 和 Michael S Bernstein。2023年。[生成代理：人类行为的互动模拟体](https://arxiv.org/abs/2304.03442)。*arXiv 预印本 arXiv:2304.03442*。

+   Press 等人（2023）Ofir Press、Muru Zhang、Sewon Min、Ludwig Schmidt、Noah Smith 和 Mike Lewis。2023年。[测量和缩小语言模型中的组合性差距](https://doi.org/10.18653/v1/2023.findings-emnlp.378)。发表于*计算语言学协会会议论文集：EMNLP 2023*，第5687–5711页，新加坡。计算语言学协会。

+   Qin 等人（2023）Yujia Qin、Shengding Hu、Yankai Lin、Weize Chen、Ning Ding、Ganqu Cui、Zheni Zeng、Yufei Huang、Chaojun Xiao、Chi Han 等人。2023年。[利用基础模型进行工具学习](https://arxiv.org/abs/2304.08354)。*arXiv 预印本 arXiv:2304.08354*。

+   Schick 等人（2023）Timo Schick、Jane Dwivedi-Yu、Roberto Dessì、Roberta Raileanu、Maria Lomeli、Luke Zettlemoyer、Nicola Cancedda 和 Thomas Scialom。2023年。[Toolformer：语言模型可以自学使用工具](https://arxiv.org/pdf/2302.04761)。*arXiv 预印本 arXiv:2302.04761*。

+   Shen 等人（2023）Yongliang Shen、Kaitao Song、Xu Tan、Dongsheng Li、Weiming Lu 和 Yueting Zhuang。2023年。[Hugginggpt：使用 ChatGPT 和它在 HuggingFace 上的朋友们解决 AI 任务](https://arxiv.org/pdf/2303.17580)。*arXiv 预印本 arXiv:2303.17580*。

+   Shinn 等人（2023）Noah Shinn、Federico Cassano、Beck Labash、Ashwin Gopinath、Karthik Narasimhan 和 Shunyu Yao。2023年。[Reflexion：带有语言强化学习的语言代理](https://arxiv.org/abs/2303.11366)。*arXiv 预印本 arXiv:2303.11366*。

+   Wang 等人（2023）Lei Wang、Chen Ma、Xueyang Feng、Zeyu Zhang、Hao Yang、Jingsen Zhang、Zhiyuan Chen、Jiakai Tang、Xu Chen、Yankai Lin 等人。2023年。[基于大语言模型的自主代理调查](https://arxiv.org/abs/2308.11432)。*arXiv 预印本 arXiv:2308.11432*。

+   Wang 等人（2022）Xuezhi Wang、Jason Wei、Dale Schuurmans、Quoc Le、Ed Chi、Sharan Narang、Aakanksha Chowdhery 和 Denny Zhou。2022年。[自一致性改善语言模型中的链式思维推理](https://arxiv.org/abs/2203.11171)。*arXiv 预印本 arXiv:2203.11171*。

+   Wei 等人（2022）Jason Wei、Xuezhi Wang、Dale Schuurmans、Maarten Bosma、Fei Xia、Ed Chi、Quoc V Le、Denny Zhou 等人。2022年。[链式思维提示引发大语言模型的推理](https://openreview.net/pdf?id=_VjQlMeSB_J)。*神经信息处理系统进展*。

+   Wolfson等人（2020）Tomer Wolfson, Mor Geva, Ankit Gupta, Matt Gardner, Yoav Goldberg, Daniel Deutch, 和Jonathan Berant。2020年。[分解它：一个问题理解基准](https://doi.org/10.1162/tacl_a_00309)。*计算语言学协会会刊*，8:183–198。

+   Xie等人（2023）Yuxi Xie, Kenji Kawaguchi, Yiran Zhao, Xu Zhao, Min-Yen Kan, Junxian He, 和Qizhe Xie。2023年。[分解通过自评引导解码增强推理](https://arxiv.org/abs/2305.00633)。*arXiv预印本arXiv:2305.00633*。

+   Yao等人（2022）Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, 和Yuan Cao。2022年。[React：在语言模型中协同推理与行动](https://arxiv.org/pdf/2210.03629)。*arXiv预印本arXiv:2210.03629*。

+   Ye等人（2023）Xi Ye, Srinivasan Iyer, Asli Celikyilmaz, Veselin Stoyanov, Greg Durrett, 和Ramakanth Pasunuru。2023年。[有效的上下文学习的互补解释](https://doi.org/10.18653/v1/2023.findings-acl.273)。发表于 *计算语言学协会发现：ACL 2023*，第4469–4484页，加拿大多伦多。计算语言学协会。

+   Zhang等人（2022）Zhuosheng Zhang, Aston Zhang, Mu Li, 和Alex Smola。2022年。[大语言模型中的自动化思维链提示](https://arxiv.org/abs/2210.03493)。*arXiv预印本arXiv:2210.03493*。

+   Zhou等人（2022）Denny Zhou, Nathanael Schärli, Le Hou, Jason Wei, Nathan Scales, Xuezhi Wang, Dale Schuurmans, Claire Cui, Olivier Bousquet, Quoc Le 等人。2022年。[从少到多的提示使得大语言模型能够进行复杂推理](https://arxiv.org/abs/2205.10625)。*arXiv预印本arXiv:2205.10625*。

## 附录A 示例附录

### A.1 提示设计

在本节中，我们将展示执行代理风格推理的提示。AgentCOT的完整提示见图[7](https://arxiv.org/html/2409.12411v1#A1.F7 "图7 ‣ A.1 提示设计 ‣ 附录A 示例附录 ‣ 多轮LLM生成的复杂任务文本化代理风格推理")。我们可以看到，提示并未提供明确的行动描述，因为我们已确定LLM已经包含了QDMR中的行动集知识，如图[8](https://arxiv.org/html/2409.12411v1#A1.F8 "图8 ‣ A.1 提示设计 ‣ 附录A 示例附录 ‣ 多轮LLM生成的复杂任务文本化代理风格推理")和图[9](https://arxiv.org/html/2409.12411v1#A1.F9 "图9 ‣ A.1 提示设计 ‣ 附录A 示例附录 ‣ 多轮LLM生成的复杂任务文本化代理风格推理")所示。

![参见说明](img/2674c9b5ae0f8e5ab7b7eef3a740b734.png)

图7：AgentCOT的完整提示。

![参见说明](img/abfe7cb2e8065b6259d71235343e706a.png)

图8：展示GPT-3模型如何在QDMR中包含行动知识。

![请参阅说明](img/3fb4b1344d5cd597a7b000722b921c6f.png)

图 9：展示 GPT-3.5 模型包含 QDMR 中的动作知识。

在这里，我们提供了由 AgentCOT 生成的详细 COT 示例，如图 [10](https://arxiv.org/html/2409.12411v1#A1.F10 "Figure 10 ‣ A.1 Prompt Design ‣ Appendix A Example Appendix ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation") 所示。当原问题 $Q$ 到来时，AgentCOT 首先从定义的动作集中选择一个动作 $a^{0}$，并提供所选动作的详细描述 $a_{des}^{0}$（第 [1] 行）。然后，$Q + a^{0} + a_{des}^{0}$ 替换原问题 $Q$ 输入到 AgentCOT 中，生成中间证据 $E_{inter}^{0}$（第 [2] 行）和中间结果 $R_{inter}^{0}$（第 [3] 行）。此时，AgentCOT 已经完成了解决 $Q$ 的第一步。接下来，AgentCOT 响应 $Q + a^{0} + a_{des}^{0} + E_{inter}^{0} + R_{inter}^{0}$ 并选择一个新的动作 $a^{1}$，以及该动作的描述 $a_{des}^{1}$。AgentCOT 持续执行上述过程，直到问题解决。

在 AgentCOT 的实现中，我们鼓励 AgentCOT 生成剩余的完整推理过程。例如，当 AgentCOT 首次与原问题 $Q$ 进行交互时，它只需要提供 $a^{0}$ 和 $a_{des}^{0}$（第 [1] 行），但也可以生成剩余的完整 COT（第 [1] 行到第 [10] 行）。完整的 COT 用于评估 AgentCOT 的执行是否已经终止。如果 $a^{i}$，$a_{des}^{i}$，$E_{inter}^{i}$，$R_{inter}^{i}$ 是完整 COT 中的最后一步，则表示问题解决过程已经完成。

![请参阅说明](img/51460fefd350a452ef31b9a4612e1a84.png)

图 10：由 AgentCOT 生成的 COT 示例。‘[N]’ 是为了可读性而提供的，不是源序列的一部分。

### A.2 AgentCOT 增强

在自评解码过程中，AgentCOT 通过询问‘当前推理过程是否合理？’来评估当前状态。此评估过程基于 LLM，并在生成 {$a^{i}$，$a_{des}^{i}$，$E_{inter}^{i}$，$R_{inter}^{i}$} 后的第 $i$ 步发生。

在集成策略中，AgentCOT 同时考虑动作、中间结果和建议的最终结果。以图 [10](https://arxiv.org/html/2409.12411v1#A1.F10 "Figure 10 ‣ A.1 Prompt Design ‣ Appendix A Example Appendix ‣ Textualized Agent-Style Reasoning for Complex Tasks by Multiple Round LLM Generation") 中 AgentCOT 完成第一步时的响应为例，动作是‘算术’（第 [1] 行），中间结果是‘220 英里’（第 [3] 行），而建议的最终结果是‘230’（第 [10] 行）。在实现过程中，我们基于投票机制选择最优的当前状态，优先考虑建议的最终结果，其次是中间结果，最后是动作。自评解码策略在集成策略之后执行。
