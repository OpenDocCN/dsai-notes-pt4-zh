<!--yml

分类：未分类

日期：2025-01-11 12:53:54

-->

# 朝着更好的人与代理对齐：评估LLM驱动应用中的任务效用

> 来源：[https://arxiv.org/html/2402.09015/](https://arxiv.org/html/2402.09015/)

Negar Arabzadeh${}^{1}$  Julia Kiseleva${}^{2}$  Qingyun Wu${}^{3}$  Chi Wang${}^{2}$

Ahmed Awadallah${}^{2}$  Victor Dibia${}^{2}$  Adam Fourney${}^{2}$  Charles Clarke${}^{1}$

${}^{1}$滑铁卢大学

${}^{2}$微软研究院

${}^{3}$宾夕法尼亚州立大学 __在微软研究实习期间完成的工作

###### 摘要

大型语言模型（LLM）领域的快速发展催生了许多应用，这些应用促进了多个代理之间的协作，旨在帮助人类处理日常任务。然而，仍然存在一个显著的差距，即无法评估LLM驱动的应用是否真正提升了用户体验和任务执行效率。这突显了验证LLM驱动应用效用的迫切需求，特别是通过确保应用程序功能与最终用户需求的一致性。我们介绍了AgentEval¹¹1[https://github.com/microsoft/autogen/blob/main/notebook/agenteval_cq_math.ipynb](https://github.com/microsoft/autogen/blob/main/notebook/agenteval_cq_math.ipynb)，它提供了数学问题的实现，这是一个旨在简化效用验证过程的新框架，通过自动提出一套针对任何给定应用程序独特目的量身定制的标准。这使得能够进行全面评估，根据建议的标准量化应用的效用。我们对AgentEval在两个开源数据集上的稳健性进行了全面分析。

## 1 引言

开源库的快速发展 Wu 等人 ([2023](https://arxiv.org/html/2402.09015v3#bib.bib47)); Li 等人 ([2023a](https://arxiv.org/html/2402.09015v3#bib.bib23))，旨在简化基于大型语言模型（LLM）的代理解决方案开发，应用于各种用户导向任务，导致此类应用的快速增长 Liang 等人 ([2023b](https://arxiv.org/html/2402.09015v3#bib.bib28)); Hong 等人 ([2023](https://arxiv.org/html/2402.09015v3#bib.bib14)); Talebirad 和 Nadiri ([2023](https://arxiv.org/html/2402.09015v3#bib.bib39))。其中一个长期目标是 Winograd ([1972](https://arxiv.org/html/2402.09015v3#bib.bib46)) 实现与人类的无缝自然语言互动，帮助最终用户，简化他们的生活，从数学辅导到完成家务等任务。最终用户对开发的应用有期望和需求，这些需求需要得到满足。理解这些需求对于评估应用程序带来的*效用*至关重要，因此，有助于进一步改进和使应用程序更好地与最终用户的目标对齐。

直接评估代理系统存在挑战，因为目前的方法主要依赖于端到端的成功度量——本质上就是代理是否完成任务 Shridhar 等人（[2020b](https://arxiv.org/html/2402.09015v3#bib.bib38)，[2019](https://arxiv.org/html/2402.09015v3#bib.bib36)）；Myers 等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib33)）。然而，理解用户与应用程序的互动不仅仅是成功本身的问题 Kiseleva 等人（[2022a](https://arxiv.org/html/2402.09015v3#bib.bib19)，[b](https://arxiv.org/html/2402.09015v3#bib.bib20)）；Zhang 等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib49)）。以数学问题为例，关键不仅是代理是否能解决问题。更重要的是，代理能否根据各种标准（包括完整性、简洁性和解释的清晰度）呈现解决方案。换句话说，在代码补全场景中，即使是一个不完整的代码建议，当它提供了大量的模板代码或提出了解决任务的框架时，也可以是有用的 Dibia 等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib9)）。此外，并非每个任务的成功都能清晰界定。了解这些标准对于一个由大规模语言模型（LLM）驱动的应用程序，并能够量化这些标准，对于验证是否满足用户需求至关重要，换句话说，就是验证应用程序是否能为最终用户带来实际效益。鉴于验证任意应用程序的目标，依赖基准测试方法不可行，因为涉及的任务范围过于广泛，要求自动化处理。前提是需要一种可扩展且灵活的方法论，能够容纳多样化的应用程序。

![参见标题说明](img/450c4d396cb163b9e92ef0a8f7ccdfc9.png)

图1：*AgentEval*框架概述由两个主要组件组成：（C）*CriticAgent*，它学习一组 $n$ 个标准（$C=\{c_{1},\dots,c_{n}\}$）以及每个标准的建议值（$c_{i}:\{\omega_{j}\}_{j=1}^{m}$），其中 $m$ 是建议值的数量，适用于任何可以由领域专家评估的应用程序；（Q）*QuantifierAgent*，它验证一组为某个应用程序建议的标准，并为最终用户建议一个任务效用值（$U_{t}(s)=\{Q_{i}(s|c_{i})\}_{i=1}^{n}$）

在这项工作中，我们旨在介绍AgentEval框架，这是一个旨在快速评估LLM驱动的代理应用程序在帮助最终用户完成任务时效用的工具。AgentEval的目标是评估应用行为与用户目标之间的当前一致性，为应用开发人员提供关于当前流程在哪些方面可以改进的洞察。AgentEval考虑了近期的发现，这些发现表明LLM作为一种可扩展且具有成本效益的替代方案，已成为人类评估在开放性任务中的替代品（Li等人，[2023b](https://arxiv.org/html/2402.09015v3#bib.bib24)）。*AgentEval* 如图[1](https://arxiv.org/html/2402.09015v3#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")所示，包含两个主要的代理，依次执行。这些代理是可定制的、可对话的，并且可以在多种模式下操作，结合了LLM、人类输入和工具的不同组合（Wu等人，[2023](https://arxiv.org/html/2402.09015v3#bib.bib47))²²2[https://github.com/microsoft/autogen](https://github.com/microsoft/autogen):

+   •

    *CriticAgent*根据任务描述和建议的解决方案提出标准列表，例如，对于数学问题，可能包括解决方案的*效率*和解决方案的*清晰度*。

+   •

    *QuantifierAgent*验证由为任务$t$设计的代理系统所产生的解决方案$s$在每个标准下的表现，并返回效用函数，例如，解决方案的清晰度水平是模糊的、适度清晰的还是非常清晰的。

我们相信，*AgentEval*的使用不仅限于对LLM驱动的应用程序当前性能的验证。该框架可以随着时间的推移，用于揭示系统的新能力以及任务效用随时间变化的潜力。发现的效用函数可以用于优化系统，以满足用户需求或系统开发者的要求，并且这种优化可以随着时间的推移进行。

总结来说，我们的主要贡献包括：

1.  C1

    对任务效用的定义，使得可以访问最终用户可能对LLM驱动的应用程序的需求，以及应用程序如何满足这一标准列表；

1.  C2

    介绍*AgentEval*，这是一个创新的框架，利用LLM驱动的代理作为一种可扩展且成本效益高的替代方案，以人类评估产生任务效用，通过两个代理的协作来实现：*CriticAgent*根据任务描述以及代理的成功与失败执行提出一系列标准，*QuantifierAgent*评估当前应用程序的实现如何支持这些标准；

1.  C3

    对*AgentEval*在各种任务和数据集上的鲁棒性进行深入分析，并提供可复制的新领域解决方案。

本文的其余部分组织如下。第[2](https://arxiv.org/html/2402.09015v3#S2 "2 Related Work ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节描述了早期的工作和背景。我们在第[3](https://arxiv.org/html/2402.09015v3#S3 "3 Defining Task Utility ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节中提供了AgentEval的动机，并定义了任务的效用。第[4](https://arxiv.org/html/2402.09015v3#S4 "4 Datasets and Solutions ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节概述了数据集，即MATH Hendrycks等人（[2021b](https://arxiv.org/html/2402.09015v3#bib.bib13)）和ALFWorld Shridhar等人（[2020b](https://arxiv.org/html/2402.09015v3#bib.bib38)），以及在我们的工作中用于构建基于LLM的应用程序的解决方案。第[5](https://arxiv.org/html/2402.09015v3#S5 "5 AgentEval Workflow ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节展示了我们关于应用AgentEval评估任务效用在选定数据集中的发现。第[6](https://arxiv.org/html/2402.09015v3#S6 "6 AgentEval Robustness Analysis and In-depth Discussion ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节提供了AgentEval稳健性的深入分析，具体包括*CriticAgent*的稳健性（第[6.1](https://arxiv.org/html/2402.09015v3#S6.SS1 "6.1 Task-based vs Solution-based criteria ‣ 6 AgentEval Robustness Analysis and In-depth Discussion ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节），*QuantifierAgent*的稳健性（第[6.2](https://arxiv.org/html/2402.09015v3#S6.SS2 "6.2 Quantifier Agent Robustness ‣ 6 AgentEval Robustness Analysis and In-depth Discussion ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节）以及*QuantifierAgent*的自动验证（第[6.3](https://arxiv.org/html/2402.09015v3#S6.SS3 "6.3 QuantifierAgent Verification ‣ 6 AgentEval Robustness Analysis and In-depth Discussion ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节）。

## 2 相关工作

我们在以往工作的基础上展开。首先，我们将讨论评估一般大型语言模型的基准和方法（第[2.1](https://arxiv.org/html/2402.09015v3#S2.SS1 "2.1 LLM evaluation ‣ 2 Related Work ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节）。其次，我们将介绍理解和预测用户效用函数的方法（第[2.2](https://arxiv.org/html/2402.09015v3#S2.SS2 "2.2 User satisfaction prediction ‣ 2 Related Work ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节）。第三，我们将讨论目前将LLM用作评估者的趋势（第[2.3](https://arxiv.org/html/2402.09015v3#S2.SS3 "2.3 Using LLMs as evaluators ‣ 2 Related Work ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节）。

### 2.1 大型语言模型（LLM）评估

有大量文献致力于评估语言模型（LLM），如Guo等人的广泛研究工作（[2023](https://arxiv.org/html/2402.09015v3#bib.bib11)）；Ziyu等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib50)）；Chang等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib6)）；Liang等人（[2023a](https://arxiv.org/html/2402.09015v3#bib.bib27)）。LLM已经从多个角度进行评估，包括但不限于，专门化的LLM（Jin等人，[2019](https://arxiv.org/html/2402.09015v3#bib.bib16)），伦理和道德（Hendrycks等人，[2021a](https://arxiv.org/html/2402.09015v3#bib.bib12)），安全性和稳健性（Wang等人，[2023](https://arxiv.org/html/2402.09015v3#bib.bib42)），以及知识和推理（Bian等人，[2023](https://arxiv.org/html/2402.09015v3#bib.bib5)）。此外，最近的研究进展包括引入复杂的多模态基准数据集（Mialon等人，[2023](https://arxiv.org/html/2402.09015v3#bib.bib32)）；Bang等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib3)）。此外，还有一些尝试将LLM评估为代理（Liu等人，[2023](https://arxiv.org/html/2402.09015v3#bib.bib30)）。

然而，缺乏专门关注LLM在帮助终端用户解决任务中的整体效用验证的文献，而这是我们在本研究中所解决的问题。

### 2.2 用户满意度预测

最近的研究表明，与各种系统交互的用户心中有特定的效用函数 Li等人（[2020](https://arxiv.org/html/2402.09015v3#bib.bib25)）；Azzopardi等人（[2018](https://arxiv.org/html/2402.09015v3#bib.bib2)）；Ahmadvand等人（[2022](https://arxiv.org/html/2402.09015v3#bib.bib1)）。传统上，定义用户满意度的度量标准是基于各种大规模收集的行为信号 Kiseleva等人（[2014](https://arxiv.org/html/2402.09015v3#bib.bib17)），并且它们是针对特定应用量身定制的，例如智能助手 Kiseleva等人（[2016a](https://arxiv.org/html/2402.09015v3#bib.bib21), [b](https://arxiv.org/html/2402.09015v3#bib.bib22)），网页搜索引擎 Williams等人（[2016a](https://arxiv.org/html/2402.09015v3#bib.bib43), [b](https://arxiv.org/html/2402.09015v3#bib.bib44)）；Williams和Zitouni（[2017](https://arxiv.org/html/2402.09015v3#bib.bib45)），对话系统 See等人（[2019](https://arxiv.org/html/2402.09015v3#bib.bib35)），多轮对话 Li等人（[2021](https://arxiv.org/html/2402.09015v3#bib.bib26)）和通用个人助手 Kiseleva和de Rijke（[2017](https://arxiv.org/html/2402.09015v3#bib.bib18)）。

### 2.3 使用大型语言模型（LLMs）作为评估者

此外，越来越多的趋势是将LLMs作为评估者使用 Chiang和Lee（[2023](https://arxiv.org/html/2402.09015v3#bib.bib7)）；Fu等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib10)）用于定性研究 Bano等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib4)）并将LLMs作为人类行为的代理 Tjuatja等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib40)）；Liu和Sun（[2023](https://arxiv.org/html/2402.09015v3#bib.bib29)）。Jain等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib15)）研究了基于上下文学习的评估者在评估LLMs生成的零-shot摘要中的效果。值得注意的是，CoEval Li等人（[2023b](https://arxiv.org/html/2402.09015v3#bib.bib24)）最近展示了人类评估与LLMs之间的协同作用，在为开放式NLG任务建立评估标准和进行多维度评估方面的应用。

在这些研究的基础上，我们提出了一个框架，能够大规模地评估各种基于LLM的应用程序的效用。这个框架旨在使代理系统与人类偏好保持一致。

## 3 定义任务效用

![参见标题](img/47e7c4dd05dbf4da0c5024ad6c60dab3.png)

图2：基于最优解存在性的任务评估分类

重要的是要从我们关注的LLM驱动应用程序的任务类别开始考虑。图[1](https://arxiv.org/html/2402.09015v3#S1.F1 "图1 ‣ 1 引言 ‣ 向更好的人机对齐迈进：评估LLM驱动应用中的任务效用")概述了代理系统的目标任务分类，按成功度量标准来划分。在最顶层，任务可以分为两个主要类别，其中：

+   •

    *成功定义不明确* — 对于这些任务，用户以辅助的方式使用系统，寻求建议而不是期望系统完全解决任务。例如，用户可能请求系统根据某些用户输入生成一封电子邮件。在许多情况下，这些生成的内容作为模板，用户稍后会进行编辑。然而，对于这些任务，成功的定义并不明确。在在线评估的情况下，虽然成本较高，我们可以询问用户帮助的程度有多大。尽管量化帮助的有效程度本身仍然具有挑战性，但当涉及到离线评估或在没有用户的情况下对新场景进行评估时，问题就变得更加复杂。

+   •

    *成功定义明确* — 对于这些任务，我们可以清楚地判断系统是否解决了任务。例如，考虑到帮助完成家务的代理，其成功定义是清晰且可衡量的。

这一第二类任务可以进一步分为两个子类：

+   [leftmargin=*, nosep]

+   •

    *成功定义明确且存在最优解* — 对于这些任务，只有一种解决方案是可能的。例如，如果你让你的助手打开灯，这个任务的成功是明确定义的，且只有一种方法可以完成它。

+   •

    *成功定义明确且存在多种解决方案* — 我们越来越多地观察到，在某些情况下，多个代理行为的轨迹都可以导致成功或失败。在这种情况下，区分各种成功和失败的结果变得至关重要。例如，当你让代理建议一道食谱或讲一个笑话时，你可能将成功定义为食物好吃或笑话好笑，但也许食谱的准备不应该太贵，而笑话也不应该令人反感。

在我们的AgentEval框架中，我们目前专注于那些成功定义明确并且可能存在多种成功解决方案的任务。

我们之前关于辅助代理的研究表明，获得人类判断的最优方式是将两个代理并排展示，并询问其偏好Kiseleva等人（[2022b](https://arxiv.org/html/2402.09015v3#bib.bib20)）。在这种成对比较的设置下，人类可以列出标准，解释为何他们更喜欢一个代理的行为而非另一个。例如，“第一个代理在执行上更快”或“第二个代理的动作更自然”。因此，比较性质引导人类列出一系列标准，帮助推断任务的效用。基于这一理念，我们设计了AgentEval（如图 [1](https://arxiv.org/html/2402.09015v3#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")所示），我们使用LLM来帮助理解、验证和评估多代理系统的任务效用。AgentEval框架采用了两种类型的代理，即：

+   [leftmargin=*, nosep]

+   •

    *CriticAgent*的目标是建议一系列可用于评估最终用户任务效用的标准。该批评者会收到任务描述以及一些成功和失败的任务执行示例；然后它能够返回一系列标准：$C=\{c_{1},\dots,c_{n}\}$，其中每个标准$c_{i}$伴随有一组接受的值$\omega$，表示为$c_{i}:\{\omega_{j}\}_{j=1}^{m}$。例如，*CriticAgent*生成了如清晰度、效率等标准，如Tab. [1](https://arxiv.org/html/2402.09015v3#S4.T1 "Table 1 ‣ 4 Datasets and Solutions ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")所描述，用于解决数学问题。每个标准都会伴随一组接受的值，如该表格中所示的示例。

+   •

    *QuantifierAgent*的目标是量化每个建议的标准，以评估最终用户的任务效用$U_{t}$，其形式为：$U_{t}(s)=\{Q_{i}(s|c_{i})\}_{i=1}^{n}$，为我们提供该系统在给定任务上对最终用户效用的概念。在其中，$s$代表任务样本，$Q(s|c_{i}.)$是基于标准$c_{i}$对样本$s$的量化输出。例如，对于一个数学问题求解样本，并且给定Tab. [1](https://arxiv.org/html/2402.09015v3#S4.T1 "Table 1 ‣ 4 Datasets and Solutions ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")中显示的生成标准，解答的准确性可以量化为“错误”、“部分正确”或“正确”。量化过程中的有效量化值如Tab. [1](https://arxiv.org/html/2402.09015v3#S4.T1 "Table 1 ‣ 4 Datasets and Solutions ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")中的“接受的值”列所示。

接下来，我们将讨论用于测试AgentEval工作的数据集和基准。

## 4 数据集和解决方案

表1：数学问题验证标准

| 标准 | 描述 |
| --- | --- |

接受的值

|

| --- | --- | --- |
| --- | --- | --- |
| 清晰度 | 解决方案中步骤、解释和语言的易懂程度。 | – 不清晰 (0) – 中等清晰 (1) – 非常清晰 (2) |
| 效率 | 使用最佳方法或途径解决数学问题。 | – 低效 (0) – 中等效率 (1) – 高效 (2) |
| 错误分析 | 在数学问题解决过程中识别和描述可能的错误或误解。 | – 未解决 (0) – 部分解决 (1) – 完全解决 (2) |
| 完整性 | 代码的效率和优雅性 | – 不完整 (0) – 基本完整 (1) – 完整 (2) |

本节提供了我们研究中使用的数据集概述。我们的选择包括了多种数据集，从基于现实问题的数据集到其模拟数据集及更广泛的选择。数学数据集（第[4.1](https://arxiv.org/html/2402.09015v3#S4.SS1 "4.1 MATH Problem Solving ‣ 4 Datasets and Solutions ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节）因其广泛的应用和领域内的综合理解而被选中。它代表了在评估多智能体系统效果时至关重要的复杂问题解决场景。AlfWorld（第[4.2](https://arxiv.org/html/2402.09015v3#S4.SS2 "4.2 ALFWorld Household Task ‣ 4 Datasets and Solutions ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节）提供了一个在适度逼近的多模态环境中进行多轮交互的场景。该数据集在评估智能体在互动和动态环境中的表现时具有重要作用。

每个数据集在评估AgentEval能力的不同方面中都扮演着至关重要的角色，从处理复杂的理论问题到应对现实世界场景。在这两个任务中，虽然成功是明确界定的，但完成目标的方式有多种。例如，在解决数学问题时，有多种方法可以选择。类似地，在Alfworld数据集中，涉及家庭任务时，根据如何搜索物体和运用思维策略等因素，完成任务的方式也有多种。数学问题解决和AlfWorld任务的示例见附录[A.1](https://arxiv.org/html/2402.09015v3#A1.SS1 "A.1 Task Examples ‣ Appendix A Appendix ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")。

![参见说明](img/f664e1345389bfc424ae9f47042c193e.png)

图3：（a）AgentEval对三种不同解决方案在数学问题解决任务中的评估分类（b）按成功和失败案例分类的相同评估

### 4.1 数学问题解决

MATH数据集最初是由Hendrycks等人（[2021b](https://arxiv.org/html/2402.09015v3#bib.bib13)）提供的，它包含了12,500个来自高中竞赛的挑战性数学问题。每个问题都附有逐步的解决方案，使得模型能够学习如何生成推导和解释。该数据集涵盖了广泛的数学学科，并按难度级别进行标记，为模型在各种数学问题解决方面的表现提供了细致的衡量标准。

该数据集特别适合用于测试多智能体系统，原因包括：（i）MATH数据集中的问题不仅仅是简单的计算，而是需要深刻理解数学概念、启发式方法和解决问题的策略；（ii）由于数据集包括逐步解决方案，它能够评估智能体学习和推理解决问题的能力，而不仅仅是其得出正确答案的能力；（iii）MATH数据集中的学科种类和难度水平的多样性，使得对系统在不同数学领域的多功能性和适应性进行全面评估成为可能，这对于预计在多种场景中运作的多智能体系统至关重要。

类似于吴等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib47)）在数学问题实验设置中的方法，我们进行两项实验评估，涉及120个问题，来自最具挑战性的5级分类，其中包括每个类别20个问题，涵盖数论、计数与概率、基础代数、代数、中级代数和预备微积分等六个不同的领域。

解决方案：在为此任务制定解决方案时，我们借鉴了Wu等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib47)）展示的实验。我们通过AutoGen Wu等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib47)）以及Langchain ReAct ³³3[https://python.langchain.com/en/latest/index.html](https://python.langchain.com/en/latest/index.html)和一个使用gpt-4来解决任务的Vanilla求解器来评估所提议的方法论。这些解决方案方法在解决数学问题方面已经展现出良好的性能，特别是在Wu等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib47)）的数据集上。我们使用AgentEval评估并比较这三种解决方案的性能。图[9](https://arxiv.org/html/2402.09015v3#A1.F9 "Figure 9 ‣ A.1 Task Examples ‣ Appendix A Appendix ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")展示了来自初等代数类别的一个数学问题实例，以及AutoGen生成的解答。在[5.1](https://arxiv.org/html/2402.09015v3#S5.SS1 "5.1 AgentEval for Math Problems ‣ 5 AgentEval Workflow ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节中，我们深入探讨了AgentEval在数学问题求解任务中的表现，以及通过AgentEval衡量的性能与真实结果的关联。

### 4.2 ALFWorld家庭任务

ALFWorld 提供了一组基于语言的互动决策任务，模拟家庭环境中的情境，Shridhar 等人（[2020b](https://arxiv.org/html/2402.09015v3#bib.bib38)）。这个基准测试的特点在于其任务的多样性，提供了一个全面的平台，用于测试人工智能和多智能体系统。该基准测试特别适用于这样的评估，因为首先，ALFWorld 是第一个将文本描述和命令与具身机器人模拟对齐的互动并行环境。它扩展了之前的两项工作：TextWorld，一个用于互动文本游戏的引擎，以及 ALFRED，一个用于具身环境中视觉-语言指令跟随的大规模数据集，Shridhar 等人（[2020a](https://arxiv.org/html/2402.09015v3#bib.bib37)）；Côté 等人（[2019](https://arxiv.org/html/2402.09015v3#bib.bib8)）。该基准测试的跨模态框架允许多种具身任务及其对应的基于文本的任务，使得智能体可以在语言和具身世界中进行训练和评估。此外，ALFWorld 支持开发能够进行抽象推理并具体执行动作的智能体，模仿人类在不同情境下的决策过程。最后，数据集包括从家务劳动到更复杂问题解决场景的广泛任务，为评估人工智能和多智能体系统的适应性和问题解决能力提供了一个全面的测试平台。总的来说，该数据集使得智能体能够在面对具身环境的复杂性之前，先在抽象语言环境中进行探索、互动和学习。

解决方案：关于解决 ALFWorld 家庭任务的解决方案，类似于 Wu 等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib47)）的研究，我们考虑了 ReAct 以及 Yao 等人（[2022](https://arxiv.org/html/2402.09015v3#bib.bib48)）的 ReAct，以及包括两个代理的 AutoGen 和包括三个代理的 AutoGen（Wu 等人，[2023](https://arxiv.org/html/2402.09015v3#bib.bib47)）。ReAct 是一个在 ALFWorld 环境中操作的代理，负责建议计划并执行行动。另一方面，AutoGen 两代理系统由一个由大语言模型支持的助手代理组成，负责建议计划，以及一个执行代理，负责在 ALFWorld 环境中执行行动。ReAct 和这个解决方案有时难以利用关于物理世界的基本常识，这可能导致重复的错误并陷入循环。在带有三个代理的 AutoGen 中，提供了一个基础代理，专门用于在系统表现出重复错误的早期迹象时提供至关重要的常识知识。我们使用 AgentEval 评估并比较这三种解决方案的性能。图 [10](https://arxiv.org/html/2402.09015v3#A1.F10 "Figure 10 ‣ A.1 Task Examples ‣ Appendix A Appendix ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications") 展示了 AutoGen 解决的部分 ALFWorld 家庭任务示例。

## 5 AgentEval 工作流程

本节概述了 AgentEval 的工作流程，如图 [1](https://arxiv.org/html/2402.09015v3#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications") 所示。接下来，我们将展示 AgentEval 如何基于 3 个不同的数据集工作：数学问题（第 [4.1](https://arxiv.org/html/2402.09015v3#S4.SS1 "4.1 MATH Problem Solving ‣ 4 Datasets and Solutions ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications") 节）和 ALFWorld（第 [4.2](https://arxiv.org/html/2402.09015v3#S4.SS2 "4.2 ALFWorld Household Task ‣ 4 Datasets and Solutions ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications") 节）。

### 5.1 数学问题的 AgentEval

#### 批评者和量化器的发现

执行CriticAgent后，我们获得了一组标准，用于验证表[1](https://arxiv.org/html/2402.09015v3#S4.T1 "Table 1 ‣ 4 Datasets and Solutions ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")中所示数学问题的结果。随后，*QuantifierAgent*负责基于已接受的值对每个标准进行量化。在图[3](https://arxiv.org/html/2402.09015v3#S4.F3 "Figure 3 ‣ 4 Datasets and Solutions ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications") (a)中，我们展示了*QuantifierAgent*的结果，即三种解决方案在该任务中的表现。AgentEval输出的这一可视化结果揭示了一些有趣的见解。特别地，很明显AgentEval并没有将三种解决方案在不同标准上量化为同等的表现。例如，尽管三种解决方案都使用GPT-4作为底层语言模型，但Autogen在准确性方面超过了ReAct和Vanilla GPT-4。这一观察也适用于解决方案的完整性和效率。相反，在考虑清晰度标准时，三种方法表现得更具竞争力。

如图所示，量化值的误差分析范围与其他指标有所不同。为了更好地理解这一标准，我们通过将结果分类为成功案例和失败案例，进一步审视了这些结果，如图[3](https://arxiv.org/html/2402.09015v3#S4.F3 "Figure 3 ‣ 4 Datasets and Solutions ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications") (b)所示。AutoGen、Vanilla Solver 和 ReAct 解决方案分别以橙色、蓝色和绿色呈现，而较深的条形表示成功案例的表现，较浅的条形表示失败案例的表现。每种颜色的深浅条形之间的差异验证了AgentEval的表现，因为我们预计每个正向标准在成功案例中的量化值应该高于失败案例。我们观察到，在大多数情况下，即使在95%的置信区间内，成功案例和失败案例也能区分开来。

我们进一步探讨了三种解决方案中的成功案例与失败案例之间的差异。图中一个有趣的观察是，并非所有的成功案例都是相同的，同样，失败案例也并非都相同。三种解决方案中成功案例之间的差异小于它们失败案例之间的差异。例如，Autogen的失败案例在效率和完整性上优于Vanilla GPT-4解算器。这一观察为我们提供了有价值的额外见解。

表2：AlfWorld家务任务的验证标准。

| 标准 | 描述 |
| --- | --- |

接受的值

|

| --- | --- | --- |
| --- | --- | --- |
| 任务理解 | 参与者理解问题集并遵循任务指令的能力 | – 优秀 (4) – 良好 (3) – 一般 (2) – 差 (1) – 极差 (0) |
| 制定计划 | 参与者制定战略并规划任务解决方案的能力。 | – 优秀 (4) – 良好 (3) – 一般 (2) – 差 (1) – 极差 (0) |
| 行动决策 | 参与者在选择正确行动方面的决策能力。 | – 优秀 (4) – 良好 (3) – 一般 (2) – 差 (1) – 极差 (0) |
| 行动执行 | 参与者执行所选行动的有效性。 | – 优秀 (4) – 良好 (3) – 一般 (2) – 差 (1) – 极差 (0) |
| 对反馈的回应 | 参与者根据来自环境的反馈调整下一步行动的能力 | – 优秀 (4) – 良好 (3) – 一般 (2) – 差 (1) – 极差 (0) |
| 行动正确性 | 参与者执行的动作相对于可用的动作和当前上下文的正确性 | – 正确 (1) – 错误 (0) |
| 使用终止命令 | 参与者是否正确使用“终止”命令 | – 适当 (1) – 不适当 (0) |

### 5.2 AlfWorld 的 AgentEval

#### 批评和量化发现

在本节中，我们提供了一个应用于AlfWorld家庭任务的AgentEval示例，如[5.1](https://arxiv.org/html/2402.09015v3#S5.SS1 "5.1 AgentEval for Math Problems ‣ 5 AgentEval Workflow ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")节中所述，其中通过文本接口模拟了真实世界的家庭环境，参见Shridhar等人（[2020b](https://arxiv.org/html/2402.09015v3#bib.bib38)）。在执行*CriticAgent*进行此任务时，它识别出一些具体标准，如“任务理解”，“计划制定”和“对反馈的响应”，这些标准在[2](https://arxiv.org/html/2402.09015v3#S5.T2 "Table 2 ‣ Critic and Quantifier Findings ‣ 5.1 AgentEval for Math Problems ‣ 5 AgentEval Workflow ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")表中有详细说明。我们与深度参与这些任务的研究人员进行了咨询，他们的专业知识确认了这些标准的关键性和重要性，类似于Li等人（[2023b](https://arxiv.org/html/2402.09015v3#bib.bib24)）。例如，考虑到这些任务是基于语言的并且需要交互式决策，ALFWorld中的一个智能体的任务是完成高层目标，例如将一个热苹果放入冰箱，并必须在模拟的家庭环境中进行导航和互动以实现这些目标。因此，表[2](https://arxiv.org/html/2402.09015v3#S5.T2 "Table 2 ‣ Critic and Quantifier Findings ‣ 5.1 AgentEval for Math Problems ‣ 5 AgentEval Workflow ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")中的标准满足了对该任务的评估。虽然这些标准本身描述得相当直观，但关于“使用TERMINATE”这一标准，我们注意到，在任务完成时，智能体被提示使用“TERMINATE”一词，这与任务成功密切相关。

![参见说明文字](img/bcc224a45b7e5b8f7b1b10984f6066ef.png)

图4：（a）AgentEval对AlfWorld家庭任务三种不同解决方案的评估（b）按成功和失败案例分类的相同评估。

在按照表[2](https://arxiv.org/html/2402.09015v3#S5.T2 "表2 ‣ 批评者和量化者的发现 ‣ 5.1 AgentEval数学问题 ‣ 5 AgentEval工作流程 ‣ 为了更好的人与代理对齐：评估LLM驱动应用中的任务效用")中详细列出的标准进行提取后，这些标准被传递给量化者代理（QuantifierAgent），以对每个样本进行量化。图[4](https://arxiv.org/html/2402.09015v3#S5.F4 "图4 ‣ 批评者和量化者的发现 ‣ 5.2 AgentEval for AlfWorld ‣ 5 AgentEval工作流程 ‣ 为了更好的人与代理对齐：评估LLM驱动应用中的任务效用")展示了三种引入的解决方案的结果：2个代理的AutoGen、3个代理的AutoGen和ReAct，基于Wu等人（[2023](https://arxiv.org/html/2402.09015v3#bib.bib47)）的134个测试集。在图[4](https://arxiv.org/html/2402.09015v3#S5.F4 "图4 ‣ 批评者和量化者的发现 ‣ 5.2 AgentEval for AlfWorld ‣ 5 AgentEval工作流程 ‣ 为了更好的人与代理对齐：评估LLM驱动应用中的任务效用")的左侧，Spider图展示了这三种解决方案在所有标准上的表现。需要注意的是，除“终止使用”和“动作的正确性”外，所有标准都采用五级评分系统，而这两个标准则是二元的。从这个图中可以明显看出，ReACT在所有标准上的表现都明显较差，而AutoGen的2个代理和3个代理的表现则具有竞争力。值得注意的是，增加了常识性基础的AutoGen略微优于其他方案，尤其是在“反馈响应”和“动作执行”方面。此外，图[4](https://arxiv.org/html/2402.09015v3#S5.F4 "图4 ‣ 批评者和量化者的发现 ‣ 5.2 AgentEval for AlfWorld ‣ 5 AgentEval工作流程 ‣ 为了更好的人与代理对齐：评估LLM驱动应用中的任务效用")右侧的条形图将134个游戏分为两组：失败组和成功组，并展示了每个子组的量化者表现。与图[3](https://arxiv.org/html/2402.09015v3#S4.F3 "图3 ‣ 4个数据集和解决方案 ‣ 为了更好的人与代理对齐：评估LLM驱动应用中的任务效用")相似，较深的颜色代表每个解决方案在成功案例中的表现，而较浅的颜色代表失败案例中的表现。AutoGen 3代理、AutoGen 2代理和ReAct分别用蓝色、绿色和橙色表示。对于大多数标准，失败案例与成功案例之间的区别非常明显，即使在95%的置信区间内也是如此。然而，对于某些标准，如“任务理解”，所有解决方案无论是失败还是成功，表现都非常相似。这可以解释为：（1）所有解决方案对任务有很好的理解，即使它们未能完成任务；（2）该标准可能是冗余的，因为它在这三种解决方案之间没有提供额外的信息；或（3）*量化者代理（QuantifierAgent）*无法以有意义的方式对该标准进行评分。我们不对哪些标准最适合此特定任务做出结论。相反，我们强调要根据个人的目标和应用要求，进行更深入的表现分析，而不仅仅是根据成功率进行评估。

## 6 AgentEval 稳健性分析与深入讨论

本节展示了对AgentEval稳健性分析的结果。首先，我们检查标准列表是否仅能从任务描述中提取（基于任务的标准），以及通过添加数据中的失败和成功样本，标准列表会如何变化。我们通过调整不同的样本大小来检查其对最终标准列表的影响（参见第[6.1](https://arxiv.org/html/2402.09015v3#S6.SS1 "6.1 基于任务与基于解决方案的标准 ‣ 6 AgentEval 稳健性分析与深入讨论 ‣ 朝着更好的人工智能对齐：评估LLM驱动应用中的任务效用")节）。其次，我们关注如何估计*QuantifierAgent*的稳健性（参见第[6.2](https://arxiv.org/html/2402.09015v3#S6.SS2 "6.2 量化代理稳健性 ‣ 6 AgentEval 稳健性分析与深入讨论 ‣ 朝着更好的人工智能对齐：评估LLM驱动应用中的任务效用")节）。我们注意到，本文报告的所有实验均在温度设置为0时进行。接下来，我们将使用MATH问题数据集展示我们的分析。

![参见说明](img/09162c595ed84892117ffd36142422ef.png)

图5：数学问题的任务标准与解决方案标准对比。显示每个步骤的95%区间

### 6.1 基于任务与基于解决方案的标准

#### 一般假设

我们通过两种不同的方法来执行CriticAgent。第一种方法涉及代理仅根据提供的任务描述生成标准，我们称之为“基于任务”的标准。另一方面，CriticAgent也有可能不仅从任务描述中获取标准，还能从任务解决方案的示例中得出标准，这种标准被称为“基于解决方案”的标准。在这种情况下，我们的目标是检查这种方法是否会导致代理制定的标准出现变化。我们认为这项研究对于更加清晰地了解制定有前景评估所需的标准非常重要。

一个数学问题的解答，可能无论解答是什么，都可能满足如准确性和清晰度等标准。然而，当使用额外的工具来解决问题时，比如使用编程来解决数学问题，可能会引入诸如“代码效率”这样的额外标准。如果从未考虑过使用特定的解决方案方法（如编程）来解决问题，可能最初不会包括此类标准。总之，取决于*CriticAgent*是否只收到任务描述或同时收到任务描述和解决方案示例，我们将标准分为“基于任务”和“基于解决方案”两种。此外，分析解决方案标准是否在不同解决方案之间重叠，以及不同解决方案在多大程度上共享这些标准，也非常重要。

为了比较任务导向和解决方案导向标准之间的差异，图[5](https://arxiv.org/html/2402.09015v3#S6.F5 "Figure 5 ‣ 6 AgentEval Robustness Analysis and In-depth Discussion ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")显示了在任务导向模式和三种不同的解决方案导向方法下（即当解决方案来自AutoGen、ReAct和Vanilla Solver时）为数学问题求解提取的唯一标准的数量。为了在计算成本和分析鲁棒性之间保持平衡，我们进行了50次不同种子的CriticAgent运行。随后，对于$N=50$次迭代，我们随机选择了$M\in[1,50]$个样本（$M$在图[5](https://arxiv.org/html/2402.09015v3#S6.F5 "Figure 5 ‣ 6 AgentEval Robustness Analysis and In-depth Discussion ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")的x轴上显示），并在重复此过程50次后，展示了唯一提取标准的平均数量及其95%的置信区间。我们注意到，由于我们总共从*CriticAgent*获得了50次迭代的结果，当$M$接近最大样本数，即$50$时，置信区间变得更小。

在审查标准时，我们发现一些标准之间存在相似之处，但表达方式不同。这些本质上是传达相同概念的度量指标，只是表述略有不同。在表[3](https://arxiv.org/html/2402.09015v3#S6.T3 "Table 3 ‣ General Hypothesis ‣ 6.1 Task-based vs Solution-based criteria ‣ 6 AgentEval Robustness Analysis and In-depth Discussion ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")中，我们提供了这些相似性的示例及其描述。为了更深入地理解图[5](https://arxiv.org/html/2402.09015v3#S6.F5 "Figure 5 ‣ 6 AgentEval Robustness Analysis and In-depth Discussion ‣ Towards better Human-Agent Alignment: Assessing Task Utility in LLM-Powered Applications")中展示的结果，我们建议将这些紧密相关的标准合并，以重新确定唯一标准的总数。这种方法有两个目的：1\. 它有助于我们更好地理解提取出的唯一标准的实际数量。2\. 它使我们能够评估标准的重复性和冗余性在解决方案导向和任务导向标准之间是否存在差异。通过这样做，我们可以更好地理解数据，并从分析中得出更有意义的结论。

表3：为数学问题求解任务提取的相似标准对。

| - 问题难度：已解决数学问题的复杂性 |
| --- |
| - 问题复杂度：问题的难易程度 |
| - 创新性：解决问题方法的新颖性和创造性 |
| - 创新性：使用独特或创造性方法解决问题的能力，通常不为人知。 |
| - 所需时间：解决问题所需的时间。 |
| - 完成时间：完全解决问题所需的时间。 |
| - 可理解性：所提供解决方案的清晰度和易懂程度。 |
| - 可读性：理解所提供解决方案的难易程度。 |

![参见说明文字](img/d7b0063e1ffbaab91bdd1b3d4fe71f10.png)

图6：数学问题解决标准的量化鲁棒性。每个条形图表示成功（深蓝色“//”）和失败（浅蓝色“\\”）案例的平均表现，且每组的95%区间在平均点上进行了阴影显示。两个图表被叠加展示。

为了整合相似标准，我们借鉴了之前的研究工作，如Liu等人（[2022](https://arxiv.org/html/2402.09015v3#bib.bib31)）；Vahtola等人（[2022](https://arxiv.org/html/2402.09015v3#bib.bib41)）；Reimers和Gurevych（[2019](https://arxiv.org/html/2402.09015v3#bib.bib34)）等，研究表明，利用为释义和语义相似性微调的预训练语言模型可以在众多下游NLP任务中取得良好的表现。此外，我们使用了一个特别为释义设计并经过微调的预训练语言模型——Hugging Face Paraphrase MiniLM ⁴⁴4[https://huggingface.co/sentence-transformers/paraphrase-MiniLM-L6-v2](https://huggingface.co/sentence-transformers/paraphrase-MiniLM-L6-v2)。

我们的方法首先对每个标准的标题及其描述进行编码，然后在我们的实验中测量所有可用标准之间的成对相似性。随后，通过使用指定的阈值$\tau$，我们将那些在每对标准嵌入表示之间具有较高余弦相似性的对分类为一类，并从中选择一个作为该对的代表。此策略通常用于各种NLP下游任务中。

在图[5](https://arxiv.org/html/2402.09015v3#S6.F5 "图5 ‣ 6 AgentEval鲁棒性分析与深入讨论 ‣ 朝着更好的人工智能-代理对齐：评估LLM驱动应用中的任务效用")中，我们展示了使用不同阈值（即0.7、0.85和1）时，提取的独特标准数量的结果。阈值为1表示没有标准被过滤掉。

#### 总结

在本节中，我们深入探讨了各种输入和标准提取方法。我们的探索比较了仅通过任务描述得出的基于任务的标准与*批评代理*接触任务描述和解决方案示例后得出的基于解决方案的标准的结果。我们观察到，与基于任务的方法相比，基于解决方案的方法生成了更多样化的标准。此外，即使在基于解决方案的方法中，标准的唯一数量也因模型的创造性水平而有所不同。我们还注意到，*批评代理*多次运行时，某些标准会出现重复。为此，我们建议实施合并同义词等整合技术，以消除冗余的标准。

### 6.2 量词代理的稳健性

#### 一般假设

在这里，我们旨在调查*量词代理*在多次应用于相同标准集时的稳健性。我们的目标是评估在多次量化相同标准时结果的一致性。这一点至关重要，因为我们期望量词在提供单一样本和固定标准集时表现稳定且相对不受噪音干扰。这样的稳定性对于我们对结果的信心至关重要。此外，这一分析有助于我们识别并筛选出那些可能不够稳定、无法可靠使用的标准。

为了实现这一目标，我们选择了与数学问题相关的特定标准子集，如表格[1](https://arxiv.org/html/2402.09015v3#S4.T1 "表1 ‣ 4 个数据集与解决方案 ‣ 更好的人类-代理对齐：评估LLM驱动应用中的任务效用")所示，并对第[4.1节](https://arxiv.org/html/2402.09015v3#S4.SS1 "4.1 数学问题求解 ‣ 4 个数据集与解决方案 ‣ 更好的人类-代理对齐：评估LLM驱动应用中的任务效用")中描述的120个问题进行了50次量词代理运行。我们的预期是观察每个标准的一致性量化表现。在图[6](https://arxiv.org/html/2402.09015v3#S6.F6 "图6 ‣ 一般假设 ‣ 6.1 基于任务与基于解决方案的标准 ‣ 6 代理评估稳健性分析与深入讨论 ‣ 更好的人类-代理对齐：评估LLM驱动应用中的任务效用")中，我们展示了在50次运行中，成功与失败案例的量化表现分布，重点关注五个选定标准。表现的持续水平趋势表明量词在稳健性方面更强，而图中的更多波动则表明代理的稳健性较差，表现更为嘈杂。

如结果所示，在五个生成标准中，有四个标准表现出稳定的性能。成功案例不仅持续优于失败案例，而且它们的表现也在各次实验中落在相似的范围内。然而，在“错误分析”标准上，我们观察到量化代理的表现更为波动。它没有始终预测某一组（成功或失败）表现得更好，而且量化代理的表现也在不同实验中有所变化。这表明，AgentEval工具可能在这一特定标准上未能表现出理想的鲁棒性。其潜在问题可能是该标准本身缺乏清晰度和适用性，或者*QuantifierAgent*在有效量化这一标准时遇到了困难。在任何情况下，都建议修改或删除这一标准，以提高可信度和可靠性。

此外，我们在图[7](https://arxiv.org/html/2402.09015v3#S6.F7 "图7 ‣ 一般假设 ‣ 6.2 量化代理鲁棒性 ‣ 6 AgentEval鲁棒性分析与深入讨论 ‣ 朝着更好的人工智能对齐：评估LLM驱动应用中的任务效用")中展示了量化值的分布，采用箱线图表示，说明了所有标准下失败（深蓝色）和成功（浅蓝色）案例的量化值分布。箱线图显示了分布的第一和第三四分位数以及中位数。在此图中，鲁棒的标准应表现出较窄的量化代理表现范围（较窄的箱线图），并且应能轻松区分每个标准的深浅蓝色箱线图。

与我们之前的观察一致，除了“错误分析”以外的所有四个标准，都能轻松区分成功案例和失败案例。此外，一些标准在与其他标准的比较中显示出更强的鲁棒性。例如，准确性表现出较窄的分布范围，而在失败案例中，清晰度的分布范围更广。我们认为，这种对量化代理性能的分析，将为提高可靠性、可信度和可解释性提供宝贵的见解。

![参考标题](img/1eb93af55764afcb6e5c66889872f7dc.png)

图 7：量化代理鲁棒性 - 量化代理在120道数学问题的AutoGen结果中的输出分布，按不同标准区分成功（深蓝色）和失败（浅蓝色）案例。该分布展示了与图[6](https://arxiv.org/html/2402.09015v3#S6.F6 "图6 ‣ 一般假设 ‣ 6.1 基于任务与基于解决方案的标准 ‣ 6 AgentEval鲁棒性分析与深入讨论 ‣ 朝着更好的人工智能对齐：评估LLM驱动应用中的任务效用")相同的结果。

#### 总结

我们认识到在量化研究中彻底调查每个标准的稳健性的重要性。这一分析至关重要，因为它揭示了每个标准的稳定性。此外，当有真实情况可用时，例如在成功与失败的情况下，它们为验证我们的评估提供了基准。值得注意的是，并非所有标准都表现出相同的稳健性。这种差异要求在评估过程中进行仔细考虑，尤其是在大规模语言模型（LLMs）的非确定性特征下。这样的意识对于确保我们在动态的LLM领域中评估的可靠性和准确性至关重要。

### 6.3 *量化代理*验证

为了评估每个标准量化的准确性，必须验证量化过程。理想情况下，我们希望通过与已知的成对样本进行比较来验证这个过程，在这种情况下，我们对给定标准$C$有明确的知识，知道样本$A$优于样本$B$。正确的量化应与这一知识一致。然而，随着基于LLM的应用每天不断扩展，为许多任务获取注释数据往往是不可行的，甚至是不可能的。因此，我们建议采用合成改变过的样本版本来获取验证所需的知识。

假设我们有一个受到干扰的替代版本样本$A$，我们称之为$A^{\prime}$。假设原始样本$A$的表现优于注入噪声的版本$A^{\prime}$，我们预计用于评估样本质量的标准在相同情况下会为原始样本分配更高的值，而不是较为嘈杂的变体。为了进行这种验证，我们进行了涉及数学问题的实验。我们通过从Autogen的数学问题解决数据集中删除一定比例的解答句子，向解答中引入随机噪声。对于“完整性”或“清晰度”等标准，我们预计会观察到原始解答比丢失部分解答的版本在完整性或清晰度上有更好的表现。

在我们的研究中，我们的目标是评估*QuantifierAgent*捕捉已知较好解决方案和较差解决方案之间区别的能力。我们通过随机删除25%的句子生成被干扰的解决方案版本，并在这些噪声解决方案上运行量化器。这些实验的结果如图[8](https://arxiv.org/html/2402.09015v3#S6.F8 "图 8 ‣ 6.3 量化器验证 ‣ 6 AgentEval 鲁棒性分析与深入讨论 ‣ 更好的人机对齐：评估基于LLM的应用任务效用")所示。如图所示，捕捉解决方案质量的标准（例如“清晰度”和“完整性”）在被干扰的解决方案中相比原始解决方案有所下降。这一观察结果有助于建立对*QuantifierAgent*性能的信心。

![参见说明文字](img/5345e874fc877a17389fb3136f348afc.png)

图8：在数学问题解决数据集上，量化器对原始解决方案和被干扰解决方案的验证。

## 7 结论与未来工作

旨在简化创建基于语言模型（LLM）驱动的智能解决方案的开源库的快速发展，推动了此类应用的快速增长，这些解决方案针对各种以用户为中心的任务。然而，满足最终用户对这些应用的期望和要求至关重要，这突显了评估它们提供效用的重要性。直接评估智能系统面临挑战，因为当前的方法通常仅依赖端到端的成功度量。然而，理解用户与应用程序的互动不仅仅是任务成功。鉴于需要自动化的任务种类繁多，一个可扩展且灵活的方法论对于有效评估这些应用程序至关重要。

在本研究中，我们介绍了AgentEval框架，旨在快速评估基于LLM的智能应用程序对最终用户的效用。AgentEval旨在评估应用程序行为与用户目标之间的对齐情况，为开发者提供改进方向的见解。该框架利用了最近的研究成果，表明LLM是开放性任务中一个可扩展且具有成本效益的替代方案，可以替代人工评估。AgentEval由两个代理组成：*CriticAgent*根据任务描述和建议的解决方案提出评判标准，而*QuantifierAgent*则验证解决方案与这些标准的对齐程度。该框架是可定制的、适应性的，并且可以在多种模式下运行，结合使用LLM、人类输入和工具。我们相信，AgentEval的效用不仅仅局限于即时的性能验证。它能够随着时间的推移揭示新的系统能力，并适应用户需求或开发者要求的变化。

总结来说，我们的贡献包括定义任务效用、引入AgentEval框架，并对其在多个数据集和解决方案中的表现进行深入分析。AgentEval代表了评估和优化大型语言模型驱动的应用程序的重要一步，以更好地服务最终用户。

## 术语表

## 参考文献

+   Ahmadvand 等人（2022）Ali Ahmadvand, Negar Arabzadeh, Julia Kiseleva, Patricio Figueroa Sanz, Xin Deng, Sujay Jauhar, Michael Gamon, Eugene Agichtein, Ned Friend 等人，2022年。《支持具有隐性约束的复杂信息搜索任务》*arXiv预印本 arXiv:2205.00584*。

+   Azzopardi 等人（2018）Leif Azzopardi, Paul Thomas, 和 Nick Craswell，2018年。《衡量搜索引擎结果页面的效用：一种基于信息觅食的度量方法》，收录于*第41届国际ACM SIGIR信息检索研究与发展会议*，第605-614页。

+   Bang 等人（2023）Yejin Bang, Samuel Cahyawijaya, Nayeon Lee, Wenliang Dai, Dan Su, Bryan Wilie, Holy Lovenia, Ziwei Ji, Tiezheng Yu, Willy Chung, Quyet V. Do, Yan Xu, 和 Pascale Fung，2023年。[对ChatGPT在推理、幻觉和互动性方面的多任务、多语言、多模态评估](http://arxiv.org/abs/2302.04023)。

+   Bano 等人（2023）Muneera Bano, Didar Zowghi, 和 Jon Whittle，2023年。《利用大型语言模型进行定性研究探索》*arXiv预印本 arXiv:2306.13298*。

+   Bian 等人（2023）Ning Bian, Xianpei Han, Le Sun, Hongyu Lin, Yaojie Lu, 和 Ben He，2023年。《ChatGPT是一个博学但经验不足的解答者：对大型语言模型中的常识问题的调查》*arXiv预印本 arXiv:2303.16421*。

+   Chang 等人（2023）Yupeng Chang, Xu Wang, Jindong Wang, Yuan Wu, Linyi Yang, Kaijie Zhu, Hao Chen, Xiaoyuan Yi, Cunxiang Wang, Yidong Wang 等人，2023年。《关于大型语言模型评估的调查》*ACM智能系统与技术交易*。

+   Chiang 和 Lee（2023）Cheng-Han Chiang 和 Hung-yi Lee，2023年。《大型语言模型能否替代人类评估？》*arXiv预印本 arXiv:2305.01937*。

+   Côté 等人（2019）Marc-Alexandre Côté, Akos Kádár, Xingdi Yuan, Ben Kybartas, Tavian Barnes, Emery Fine, James Moore, Matthew Hausknecht, Layla El Asri, Mahmoud Adada 等人，2019年。《Textworld：一个基于文本的游戏学习环境》，收录于*计算机游戏：第七届研讨会，CGW 2018，国际人工智能会议 IJCAI 2018 的联合会议，2018年7月13日，瑞典斯德哥尔摩，修订版精选论文7*，第41-75页，Springer出版社。

+   Dibia 等人（2023）Victor Dibia, Adam Fourney, Gagan Bansal, Forough Poursabzi-Sangdeh, Han Liu, 和 Saleema Amershi，2023年。[将离线度量与代码生成模型的人类价值判断对齐](http://arxiv.org/abs/2210.16494)。

+   Fu 等人（2023）Jinlan Fu, See-Kiong Ng, Zhengbao Jiang, 和 Pengfei Liu，2023年。《Gptscore：按需评估》*arXiv预印本 arXiv:2302.04166*。

+   Guo 等人 (2023) Zishan Guo, Renren Jin, Chuang Liu, Yufei Huang, Dan Shi, Linhao Yu, Yan Liu, Jiaxuan Li, Bojian Xiong, Deyi Xiong 等人. 2023. 评估大语言模型：一项综合性调查. *arXiv 预印本 arXiv:2310.19736*.

+   Hendrycks 等人 (2021a) Dan Hendrycks, Collin Burns, Steven Basart, Andrew Critch, Jerry Li, Dawn Song, 和 Jacob Steinhardt. 2021a. 将 AI 与共同的人类价值对齐. *国际学习表示大会 (ICLR) 论文集*.

+   Hendrycks 等人 (2021b) Dan Hendrycks, Collin Burns, Saurav Kadavath, Akul Arora, Steven Basart, Eric Tang, Dawn Song, 和 Jacob Steinhardt. 2021b. 使用数学数据集衡量数学问题解决能力. *arXiv 预印本 arXiv:2103.03874*.

+   Hong 等人 (2023) Sirui Hong, Xiawu Zheng, Jonathan Chen, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou 等人. 2023. Metagpt：用于多智能体协作框架的元编程. *arXiv 预印本 arXiv:2308.00352*.

+   Jain 等人 (2023) Sameer Jain, Vaishakh Keshava, Swarnashree Mysore Sathyendra, Patrick Fernandes, Pengfei Liu, Graham Neubig, 和 Chunting Zhou. 2023. 基于上下文学习的文本摘要多维评估. *arXiv 预印本 arXiv:2306.01200*.

+   Jin 等人 (2019) Qiao Jin, Bhuwan Dhingra, Zhengping Liu, William W Cohen, 和 Xinghua Lu. 2019. Pubmedqa：一个用于生物医学研究问答的数据集. *arXiv 预印本 arXiv:1909.06146*.

+   Kiseleva 等人 (2014) Julia Kiseleva, Eric Crestan, Riccardo Brigo, 和 Roland Dittel. 2014. 用户满意度变化的建模与检测. 见 *第23届ACM国际信息与知识管理会议论文集*，页面1449-1458.

+   Kiseleva 和 de Rijke (2017) Julia Kiseleva 和 Maarten de Rijke. 2017. 评估移动设备上的个人助手. *arXiv 预印本 arXiv:1706.04524*.

+   Kiseleva 等人 (2022a) Julia Kiseleva, Ziming Li, Mohammad Aliannejadi, Shrestha Mohanty, Maartje ter Hoeve, Mikhail Burtsev, Alexey Skrynnik, Artem Zholus, Aleksandr Panov, Kavya Srinet, Arthur Szlam, Yuxuan Sun, Katja Hofmann, Marc-Alexandre Côté, Ahmed Awadallah, Linar Abdrazakov, Igor Churin, Putra Manggala, Kata Naszadi, Michiel van der Meer, 和 Taewoon Kim. 2022a. [协作环境中的交互式基础语言理解：IGLU 2021](https://proceedings.mlr.press/v176/kiseleva22a.html). 见 *NeurIPS 2021竞赛与展示会论文集*，第176卷 *机器学习研究论文集*，页面146-161。PMLR.

+   Kiseleva 等人（2022b）Julia Kiseleva、Alexey Skrynnik、Artem Zholus、Shrestha Mohanty、Negar Arabzadeh、Marc-Alexandre Côté、Mohammad Aliannejadi、Milagro Teruel、Ziming Li、Mikhail Burtsev、Maartje ter Hoeve、Zoya Volovikova、Aleksandr Panov、Yuxuan Sun、Kavya Srinet、Arthur Szlam、Ahmed Awadallah、Seungeun Rho、Taehwan Kwon、Daniel Wontae Nam、Felipe Bivort Haiek、Edwin Zhang、Linar Abdrazakov、Guo Qingyam、Jason Zhang 和 Zhibin Guo。2022b。[互动式基础语言理解在协作环境中的应用：回顾 iglu 2022 竞赛](https://proceedings.mlr.press/v220/kiseleva22a.html)。收录于 *NeurIPS 2022 竞赛卷*，*机器学习研究论文集*第220卷，页码204–216。PMLR。

+   Kiseleva 等人（2016a）Julia Kiseleva、Kyle Williams、Ahmed Hassan Awadallah、Aidan C Crook、Imed Zitouni 和 Tasos Anastasakos。2016a。预测用户对智能助手的满意度。收录于 *第39届国际 ACM SIGIR 信息检索研究与发展大会论文集*，页码45–54。

+   Kiseleva 等人（2016b）Julia Kiseleva、Kyle Williams、Jiepu Jiang、Ahmed Hassan Awadallah、Aidan C Crook、Imed Zitouni 和 Tasos Anastasakos。2016b。理解用户对智能助手的满意度。收录于 *2016 年 ACM 人机信息交互与检索大会论文集*，页码121–130。

+   Li 等人（2023a）Guohao Li、Hasan Abed Al Kader Hammoud、Hani Itani、Dmitrii Khizbullin 和 Bernard Ghanem。2023a。Camel：用于“大规模语言模型社会”心智探索的交互式代理。*arXiv 预印本 arXiv:2303.17760*。

+   Li 等人（2023b）Qintong Li、Leyang Cui、Lingpeng Kong 和 Wei Bi。2023b。协作评估：探索大规模语言模型与人类在开放式生成评估中的协同作用。*arXiv 预印本 arXiv:2310.19740*。

+   Li 等人（2020）Ziming Li、Julia Kiseleva、Alekh Agarwal、Maarten de Rijke 和 Ryen W White。2020。通过数据驱动目标优化交互系统。*arXiv 预印本 arXiv:2006.12999*。

+   Li 等人（2021）Ziming Li、Dookun Park、Julia Kiseleva、Young-Bum Kim 和 Sungjin Lee。2021。Deus：一种基于数据驱动的多轮对话中用户满意度估计方法。*arXiv 预印本 arXiv:2103.01287*。

+   Liang 等人 (2023a) Percy Liang, Rishi Bommasani, Tony Lee, Dimitris Tsipras, Dilara Soylu, Michihiro Yasunaga, Yian Zhang, Deepak Narayanan, Yuhuai Wu, Ananya Kumar, Benjamin Newman, Binhang Yuan, Bobby Yan, Ce Zhang, Christian Cosgrove, Christopher D. Manning, Christopher Ré, Diana Acosta-Navas, Drew A. Hudson, Eric Zelikman, Esin Durmus, Faisal Ladhak, Frieda Rong, Hongyu Ren, Huaxiu Yao, Jue Wang, Keshav Santhanam, Laurel Orr, Lucia Zheng, Mert Yuksekgonul, Mirac Suzgun, Nathan Kim, Neel Guha, Niladri Chatterji, Omar Khattab, Peter Henderson, Qian Huang, Ryan Chi, Sang Michael Xie, Shibani Santurkar, Surya Ganguli, Tatsunori Hashimoto, Thomas Icard, Tianyi Zhang, Vishrav Chaudhary, William Wang, Xuechen Li, Yifan Mai, Yuhui Zhang 和 Yuta Koreeda. 2023a. [语言模型的整体评估](http://arxiv.org/abs/2211.09110)。

+   Liang 等人 (2023b) Tian Liang, Zhiwei He, Wenxiang Jiao, Xing Wang, Yan Wang, Rui Wang, Yujiu Yang, Zhaopeng Tu 和 Shuming Shi. 2023b. [通过多智能体辩论鼓励大语言模型的发散思维](http://arxiv.org/abs/2305.19118)。

+   Liu 和 Sun (2023) Alex Liu 和 Min Sun. 2023. 从声音到有效性：利用大语言模型 (LLMs) 进行政策利益相关者访谈的文本分析。 *arXiv 预印本 arXiv:2312.01202*。

+   Liu 等人 (2023) Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang 等人. 2023. Agentbench: 评估大语言模型作为智能体的表现。 *arXiv 预印本 arXiv:2308.03688*。

+   Liu 等人 (2022) Yanchen Liu, Timo Schick 和 Hinrich Schütze. 2022. 面向语义的无标注预处理在大规模语言模型中的应用。 *arXiv 预印本 arXiv:2202.06133*。

+   Mialon 等人 (2023) Grégoire Mialon, Clémentine Fourrier, Craig Swift, Thomas Wolf, Yann LeCun 和 Thomas Scialom. 2023. Gaia: 一个通用 AI 助手的基准测试。 *arXiv 预印本 arXiv:2311.12983*。

+   Myers 等人 (2023) Vivek Myers, Andre Wang He, Kuan Fang, Homer Rich Walke, Philippe Hansen-Estruch, Ching-An Cheng, Mihai Jalobeanu, Andrey Kolobov, Anca Dragan 和 Sergey Levine. 2023. [指令跟随的目标表示：一种半监督语言接口控制方法](https://proceedings.mlr.press/v229/myers23a.html)。载于 *第七届机器人学习会议论文集*，机器学习研究进展第229卷，页码3894–3908。PMLR。

+   Reimers 和 Gurevych (2019) Nils Reimers 和 Iryna Gurevych. 2019. Sentence-bert: 使用 Siamese BERT 网络的句子嵌入。 *arXiv 预印本 arXiv:1908.10084*。

+   See 等人 (2019) Abigail See, Stephen Roller, Douwe Kiela 和 Jason Weston. 2019. 什么构成一次好的对话？可控属性如何影响人类评判。 *arXiv 预印本 arXiv:1902.08654*。

+   Shridhar等人（2019）莫希特·什里达、杰西·托马森、丹尼尔·戈登、约纳坦·比斯克、温森·汉、鲁兹贝·莫塔吉、卢克·泽特尔莫耶和迪特·福克斯。2019年。[ALFRED：一个用于解释日常任务指令的基准](http://arxiv.org/abs/1912.01734)。*CoRR*，abs/1912.01734。

+   Shridhar等人（2020a）莫希特·什里达、杰西·托马森、丹尼尔·戈登、约纳坦·比斯克、温森·汉、鲁兹贝·莫塔吉、卢克·泽特尔莫耶和迪特·福克斯。2020年。Alfred：一个用于解释日常任务指令的基准。发表于*IEEE/CVF计算机视觉与模式识别会议论文集*，第10740-10749页。

+   Shridhar等人（2020b）莫希特·什里达、星地元、马克-亚历山大·科特、约纳坦·比斯克、亚当·特里什勒和马修·豪斯内赫特。2020年。Alfworld：为交互式学习对齐文本与具象环境。*arXiv预印本arXiv:2010.03768*。

+   Talebirad和Nadiri（2023）亚沙尔·塔勒比拉德和阿米尔霍赛因·纳迪里。2023年。多智能体协作：利用智能大语言模型代理的力量。*arXiv预印本arXiv:2306.03314*。

+   Tjuatja等人（2023）林迪亚·图亚特贾、瓦莱丽·陈、谢丽·同双吴、阿米特·塔尔瓦尔卡和格雷厄姆·纽比格。2023年。大语言模型是否表现出类人反应偏差？调查设计中的一个案例研究。*arXiv预印本arXiv:2311.04076*。

+   Vahtola等人（2022）蒂姆·瓦赫托拉、马提亚斯·克鲁茨和约尔格·蒂德曼。2022年。检测同义句并不容易：通过新的semantoneg基准分析使用反义词和否定的语义相似度。发表于*第五届BlackboxNLP工作坊：分析与解释NLP神经网络*，第249-262页。

+   王等人（2023）金东王、喜旭胡、文昕侯、浩陈、润凯郑、亦东王、林毅杨、伟叶、浩君黄、秀博耿、滨兴焦、月张和兴谢。2023年。[关于chatGPT的鲁棒性：一种对抗性和超出分布的视角](https://openreview.net/forum?id=uw6HSkgoM29)。发表于*ICLR 2023可信和可靠的大规模机器学习模型研讨会*。

+   Williams等人（2016a）凯尔·威廉姆斯、朱莉亚·基塞列娃、艾丹·C·克鲁克、伊梅德·齐图尼、艾哈迈德·哈桑·阿瓦达拉赫和马迪安·哈布萨。2016年。检测移动搜索中的良好放弃。发表于*第25届国际万维网大会论文集*，第495-505页。

+   Williams等人（2016b）凯尔·威廉姆斯、朱莉亚·基塞列娃、艾丹·C·克鲁克、伊梅德·齐图尼、艾哈迈德·哈桑·阿瓦达拉赫和马迪安·哈布萨。2016年。那是你的最终答案吗？评估答案对移动搜索中良好放弃的影响。发表于*第39届国际ACM SIGIR信息检索研究与开发会议论文集*，第889-892页。

+   Williams和Zitouni（2017）凯尔·威廉姆斯和伊梅德·齐图尼。2017年。那是否意味着你很高兴？基于RNN的用户互动序列建模，以检测良好的放弃。在*2017年ACM信息与知识管理会议论文集*，第727-736页。

+   温诺格拉德（1972）特里·温诺格拉德。1972年。《理解自然语言》。*认知心理学*，3(1):1–191。

+   吴等人（2023）吴庆云、班吉恩·贾甘、张杰宇、吴怡然、张绍坤、朱尔康、李贝滨、姜力、张晓云、王驰。2023年。《AutoGen：通过多代理对话框架启用下一代LLM应用》。*arXiv预印本 arXiv:2308.08155*。

+   姚等人（2022）姚顺宇、赵杰夫、于滇、杜楠、沙夫兰·伊扎克、纳拉辛汉·卡尔提克、曹元。2022年。《React：在语言模型中协同推理与行动》。*arXiv预印本 arXiv:2210.03629*。

+   张等人（2023）张驰、蔡鹏林、傅宇辉、袁浩琪、陆宗庆。2023年。《创意代理：通过赋予代理想象力以完成创意任务》。*arXiv预印本 arXiv:2312.02519*。

+   子瑜等人（2023）庄子瑜、陈启光、马龙轩、李名达、韩一、钱宇杉、白浩鹏、张维楠、刘婷。2023年。[《通过核心能力的视角：大语言模型评估调查》](https://aclanthology.org/2023.ccl-2.8)。在*第22届中国计算语言学大会论文集（第2卷：前沿论坛）*，第88-109页，中国哈尔滨，中国信息处理学会。

## 附录A 附录

### A.1 任务示例

在图[9](https://arxiv.org/html/2402.09015v3#A1.F9 "图9 ‣ A.1 任务示例 ‣ 附录A ‣ 朝向更好的人工智能-人类对齐：评估LLM驱动应用中的任务效用")和[10](https://arxiv.org/html/2402.09015v3#A1.F10 "图10 ‣ A.1 任务示例 ‣ 附录A ‣ 朝向更好的人工智能-人类对齐：评估LLM驱动应用中的任务效用")中，我们展示了数学问题和AlfWorld家务任务的示例，并且提供了AutoGen对这些问题的解决方案。

<svg class="ltx_picture" height="2280.99" id="A1.F9.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,2280.99) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="2253.43" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">"problem": "How many positive and negative integers is $12$ a multiple of?" "level": "Level 5" "type": "Prealgebra" "response_with_ans": "messages": [ { "content": "How many positive and negative integers is $12$ a multiple of?", "role": "user" }, { "content": "To determine how many positive and negative integers 12 is a multiple of, we need to find the divisors of 12\. Then we will double the number of divisors, as each positive divisor has a corresponding negative divisor. Let’s use a Python code snippet to find all the divisors of 12 and count them.", [⬇](data:text/plain;base64,bnVtID0gMTIKZGl2aXNvcnMgPSBbaSBmb3IgaSBpbiByYW5nZSgxLCBudW0gKyAxKSBpZiBudW0gXCUgaSA9PSAwXQpwb3NpdGl2ZV9kaXZpc29yc19jb3VudCA9IGxlbihkaXZpc29ycykKbmVnYXRpdmVfZGl2aXNvcnNfY291bnQgPSBwb3NpdGl2ZV9kaXZpc29yc19jb3VudAp0b3RhbF9kaXZpc29yc19jb3VudCA9IHBvc2l0aXZlX2Rpdmlzb3JzX2NvdW50ICsgbmVnYXRpdmVfZGl2aXNvcnNfY291bnQKcHJpbnQoZiJQb3NpdGl2ZSBkaXZpc29ycyBvZiAxMjoge2Rpdmlzb3JzfSIpCmRpdmlzb3JzX25lZ2F0aXZlID0gWy14IGZvciB4IGluIGRpdmlzb3JzXQpwcmludChmIk5lZ2F0aXZlIGRpdmlzb3JzIG9mIDEyOiB7ZGl2aXNvcnNfbmVnYXRpdmV9IikKcHJpbnQoZiJUb3RhbCBkaXZpc29ycyBjb3VudDoge3RvdGFsX2Rpdmlzb3JzX2NvdW50fSIp) 1num = 12 2divisors = [i for i in range(1, num + 1) if num \% i == 0] 3positive_divisors_count = len(divisors) 4negative_divisors_count = positive_divisors_count 5total_divisors_count = positive_divisors_count + negative_divisors_count 6print(f"Positive divisors of 12: {divisors}") 7divisors_negative = [-x for x in divisors] 8print(f"Negative divisors of 12: {divisors_negative}") 9print(f"Total divisors count: {total_divisors_count}") Execute this code snippet to find the divisors of 12 and count how many positive and negative integers 12 is a multiple of. "role": "assistant" } ], "time": 17.360238790512085</foreignobject></g></g></svg>

图9：一个数学问题求解示例，解决方案由AutoGen生成

{    { "content": "执行操作并与家庭成员互动以解决任务。在你开始互动时，你将获得当前环境的详细描述和你需要完成的目标。在每次回合中，你应该从两种操作中选择一种：“think”或“Action”。如果你选择“think”，你应该首先思考当前的情况并计划未来的行动，然后在本回合中输出你的行动。你的输出必须严格遵循以下格式：“think: 你的想法。” “Action: 你的下一步行动\\n”；如果你选择“ACTION”，你应该直接输出本回合的行动。你的输出必须严格遵循以下格式：“ACTION: 你的下一步行动”。在每次回合后，环境会根据你的行动立即给予反馈，根据反馈你应规划接下来的步骤。如果环境输出“Nothing happened”，那意味着之前的操作无效，你应该尝试更多的选项。提醒：1. 操作必须从给定的可用操作中选择。任何不在提供的可用操作中的行动将被视为非法。2. 一旦找到目标家庭成员，立即采取行动。3. 仅在以下两种情况下回复“TERMINATE”：a）任务明确指示你返回“TERMINATE”；b）任务似乎无法解决。\\n这里有两个示例。\n你处在一个房间的中间。迅速环顾四周，你看到了13个橱柜、12个橱柜、11个橱柜、10个橱柜、9个橱柜、8个橱柜、7个橱柜、6个橱柜、5个橱柜、4个橱柜、3个橱柜、2个橱柜、1个橱柜、1个咖啡机、1个台面、1个餐桌、1个抽屉、1个冰箱、1个垃圾桶、1个微波炉、3个架子、2个架子、1个架子、1个水槽、4个炉灶、3个炉灶、2个炉灶、1个炉灶和1个烤面包机。\n你的任务是：把一个热的苹果放进冰箱。\n",        "role": "user"},    {   "content": "THOUGHT: 任务是把一个热的苹果放进冰箱。所以我需要先找到一个苹果并加热它。首先我需要找到一个苹果。苹果更可能出现在餐桌（1）、咖啡桌（1）、冰箱（1）、抽屉（1）、橱柜（1-13）、垃圾桶（1）。我可以一个个检查，从冰箱1开始。\n ACTION: 去餐桌1\n",        "role": "assistant"},    {   "content": "在餐桌1上，你看到一个苹果1、一个面包1、三个叉子、两个叉子、一个叉子、两个杯子、三个胡椒瓶、两个盘子、一个锅、三个肥皂瓶、一个铲子、三个勺子、两个勺子、一个勺子、三个酒瓶和两个酒瓶。\n",        "role": "user"},    {   "content": "ACTION: 从餐桌1拿起苹果1\n",        "role": "assistant"},    {   "content": "你从餐桌1上拿起苹果1。\n",        "role": "user"},    {   "content": "THOUGHT: 我注意到房间里有一个微波炉1，我将用它来加热苹果。\n ACTION: 去微波炉1\n",        "role": "assistant"},    ...    ...    {   "content": "你打开了微波炉1。微波炉1是打开的。在里面，你看到一个鸡蛋1。\n",        "role": "user"    },    {   "content": "ACTION: 用微波炉1加热杯子2",        "role": "assistant"},    {    "content": "你用微波炉1加热了杯子2。任务成功，现在请回复TERMINATE\n",        "role": "user"},    {   "content": "TERMINATE",        "role": "assistant"},    {   "content": "TERMINATE",        "role": "user"}

图 10：来自“挑选、加热、然后放置”类别的 AlfWorld 家庭任务示例。
