<!--yml

分类：未分类

日期：2025-01-11 12:53:46

-->

# SwissNYF：面向黑箱环境的工具支持LLM代理

> 来源：[https://arxiv.org/html/2402.10051/](https://arxiv.org/html/2402.10051/)

Somnath Sendhil Kumar¹  Dhruv Jain¹  Eshaan Agarwal¹  Raunak Pandey¹

¹ [印度理工学院瓦拉纳西分校（IIT BHU）智能小组](https://cops-iitbhu.github.io/IG-website/)

###### 摘要

虽然大型语言模型（LLMs）在功能调用方面展示了增强的能力，但这些进展主要依赖于访问函数的响应。这种方法对于简单的API是可行的，但在面对不可逆的API时，尤其是对系统有重大影响的API（如数据库删除API）时，存在可扩展性问题。同样，处理每次API调用需要大量时间的过程以及那些需要前瞻性规划的任务，如自动化行动管道，也会面临复杂的挑战。此外，常常会出现需要通用方法的情况，因为算法缺乏直接访问这些功能的具体实现或其使用的秘密。在这些情况下，传统的工具规划方法是无效的，迫使我们必须在黑箱环境中操作。与工具操作中的表现不同，LLMs在黑箱任务中表现出色，例如程序合成。因此，我们利用LLMs的程序合成功能，在黑箱环境中规划工具使用，确保在实施之前验证解决方案。我们介绍了TOPGUN，这是一种巧妙设计的方法，利用程序合成进行黑箱工具规划。并通过SwissNYF，一个全面的工具，集成了用于规划和验证任务的黑箱算法，解决上述挑战，提升了LLMs在复杂API交互中的多功能性和有效性。SwissNYF的公开代码可在[https://github.com/iclr-dummy-user/SwissNYF](https://github.com/iclr-dummy-user/SwissNYF)获取。

## 1 引言

![参见标题](img/4c4fe3802f77a55527be1fe6ae08a089.png)

图1：展示LLM可能需要操作工具的不同设置。

大型语言模型（LLMs）如GPT（Radford等人（[2018](https://arxiv.org/html/2402.10051v1#bib.bib26)）；Radford等人（[2019](https://arxiv.org/html/2402.10051v1#bib.bib27)）；Brown等人（[2020](https://arxiv.org/html/2402.10051v1#bib.bib4)）；Achiam等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib1)））和PaLM（Chowdhery等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib8)）；Anil等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib2)）；）已经展示了在广泛任务中的推理和执行指令的深厚能力（Huang & Chang（[2023](https://arxiv.org/html/2402.10051v1#bib.bib14)））。最近，利用LLMs与外部工具交互来应对复杂现实世界挑战的转变成为了一个重要的研究领域（Hao等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib13)）；Zhang等人（[2023a](https://arxiv.org/html/2402.10051v1#bib.bib36)）；Zhuang等人（[2023b](https://arxiv.org/html/2402.10051v1#bib.bib42)）；Yang等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib33)）；Schick等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib28)）；Lu等人（[2023a](https://arxiv.org/html/2402.10051v1#bib.bib17)））。在处理复杂问题时，由LLMs驱动的自主代理结合LLMs和各种外部工具（API），通过一系列中间推理步骤来制定解决方案（Schick等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib28)）；Lu等人（[2023a](https://arxiv.org/html/2402.10051v1#bib.bib17)）；Lu等人（[2023a](https://arxiv.org/html/2402.10051v1#bib.bib17)）；Patil等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib24)）；Qin等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib25)））。当遇到一个问题时，这些代理的主要目标是识别并依次执行一系列API函数调用，最终得出一个连贯的解决方案。当查询缺乏透明度或API不可逆时，这些方法则无效。

我们在工具规划的背景下创造了“黑箱”设置一词，用于描述那些无法观察到API或工具结果的场景。这个框架在一些系统中尤其适用，在这些系统中，使用某些API可能带来风险，例如那些通过删除或更新数据库条目、取消任务或执行类似操作引起不一致的API。它在API实验成本较高或API执行时间较长的情况下也具有相关性，因为这些情况要求保证清晰和全面覆盖而不产生冗余，从而使得API结果难以解读。我们提出了这样一类系统的分类法（见图[1](https://arxiv.org/html/2402.10051v1#S1.F1 "图1 ‣ 1 引言 ‣ wissNYF: 工具驱动的LLM代理在黑箱设置中的应用")），分为三个分支：

1.  1.

    白箱系统：在这些环境中，规划者可以调用 API，接收响应，访问源代码并理解其复杂逻辑。这种访问使系统能够高效地处理复杂的输入、细节和使用案例。

1.  2.

    灰箱系统：在这些环境中，规划者可以使用工具的描述，并且能够调用 API 接收响应。系统的规划完全依赖于提供的有限描述和每个工具的响应。

1.  3.

    黑箱系统：在最具挑战性的场景中，规划者只能依赖工具的描述，而无法访问实际的工具输出。在这种情况下，规划者必须仅凭工具的描述解读每个工具的动态，这使得回应查询的任务特别具有挑战性。

Zhuang 等人（[2023a](https://arxiv.org/html/2402.10051v1#bib.bib41)）和 Qin 等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib25)）的方法在代理可以反复使用工具以确定最佳路径的简单场景中表现优秀，但它们效率较低且需要广泛的探索。像 Yao 等人（[2022](https://arxiv.org/html/2402.10051v1#bib.bib34)）和 Parisi 等人（[2022](https://arxiv.org/html/2402.10051v1#bib.bib22)）的方法属于这一探索范式的子集，提供了更高的效率，但由于它们在工具搜索中的方向性受到限制，常常会失败，因此它们主要适用于简单的 API 挑战。相比之下，Zhang 等人（[2023b](https://arxiv.org/html/2402.10051v1#bib.bib37)）的方法在 API 执行成本方面表现高效，通过限制调用次数来实现。然而，它没有对所提出的轨迹进行任何形式的验证，这在实际应用中降低了其精确性。

工具应用中的这些方法论在准确性和计算开销之间呈现出二分法。虽然通常不适用于黑箱设置，但反向链式方法在此类框架中展现出适应潜力。另一方面，基于程序合成的算法在提升大语言模型（LLM）的推理和决策能力方面发挥了重要作用，提供了一种比单纯文本所能带来的决策过程更自然的联想式决策过程。像《代码链》李等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib15)）和《思维程序》陈等人（[2022](https://arxiv.org/html/2402.10051v1#bib.bib7)）这样的作品是通过代码生成来改善回答一般开放领域问题的决策过程的典型例子。为此，一些作品也通过代码验证了LLM的推理能力，例如《TORA：用于数学问题求解的工具集成推理代理》郭等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib12)）、《使用GPT-4代码解释器和基于代码的自我验证解决具有挑战性的数学文字问题》周等人（[2023b](https://arxiv.org/html/2402.10051v1#bib.bib40)）和《PAL：程序辅助语言模型》高等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib11)）等作品，它们利用代码解释器进行零-shot验证求解，通过实现对提出的解决方案进行半验证，显著超越了少量样本学习基准。

然而，像Paranjape等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib21)）这样的作品，采用代码合成来进行工具使用，受到其有限工具集以及需要广泛人类反馈和干预、并且要求人类专家熟悉整个工具集所带来的可扩展性挑战的限制。同样，像Xu等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib32)）这样的作品，部署语言模型在受控环境中进行实时代码生成和命令执行，受到其狭窄工具范围和缺乏广泛适用性的限制。在HumanEval数据集上的最新技术，如《Reflexion》Shinn等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib29)）和《LATS》周等人（[2023a](https://arxiv.org/html/2402.10051v1#bib.bib39)），这些方法通过基于解释器输出的代码迭代并对其进行反思，尚未在与LLM相关的其他领域进行实验。

为了填补这些空白，我们提出了 TOPGUN（Tool Orchestration and Program synthesis for Generalizing over UNknown systems）框架，它统一了代码生成、推理和复杂任务的战略工具规划。TOPGUN 还验证执行计划，并以极高的 API 成本效率进行验证，有效地解决了先前模型的局限性。

我们工作的关键贡献总结如下：

1.  1.

    据我们所知，我们是首个提出“黑盒设置”（Black Box setting）这一术语，用于描述 API 使用，并且开发了一整套工具，旨在鼓励为此类场景开发算法。

1.  2.

    我们利用大型语言模型（LLMs）的程序合成功能，大幅提升了其在工具使用方面的效率，展示了显著的性能提升。

1.  3.

    我们提出了一种稳健且具有成本效益的框架，能够为各种开放领域查询提供可扩展的解决方案，即使在用户数据和工具的知识有限的情况下也能发挥作用。该框架还公开托管，以展示其功能。¹¹1[https://swiss-nyf.azurewebsites.net/](https://swiss-nyf.azurewebsites.net/)

本文详细介绍了我们的方法论及其评估，首先阐明了工具规划的背景 [2.1](https://arxiv.org/html/2402.10051v1#S2.SS1 "2.1 Problem Formulation ‣ 2 Preliminaries ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting") 和使用 LLM 生成代码 [2.2](https://arxiv.org/html/2402.10051v1#S2.SS2 "2.2 Code Generation ‣ 2 Preliminaries ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")，然后详细说明了管道的各个组成部分 [3](https://arxiv.org/html/2402.10051v1#S3 "3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")。我们的评估分为两个部分：首先，我们在主要数据集上进行灰盒评估 [4.1](https://arxiv.org/html/2402.10051v1#S4.SS1 "4.1 Gray Box Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")，随后进入黑盒设置评估 [4.2](https://arxiv.org/html/2402.10051v1#S4.SS2 "4.2 Black Box Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")。对于后者，我们通过使用 Toolbench 提示设计了一种定制数据集，并故意调整数据集，仅包含广泛使用库的有限文档。这一调整旨在验证我们方法的普适性。此外，我们还将我们的方法与反向链法（Reverse Chain method）的定制变体进行比较，以探讨性能差异。

## 2 前言

### 2.1 问题表述

![请参见说明文字](img/713246da1703b1bdbee92306537d5089.png)

图 2：展示 SwissNYF 管道在黑盒设置中的工具使用示意图。

在大型语言模型（LLM）上下文中，工具规划（表示为$\rho$）涉及从语料库$\mathcal{C}$中的$n$个候选工具池中选择工具，有效地解决用户查询$q$。主要目标是制定一个细致的计划，称为解决方案轨迹$St$，用于协调这些工具的使用。解决方案轨迹$St$，即工具的顺序执行，旨在直接解决查询$q$。LLM代理，或规划器$\mathcal{G}$，负责从$\mathcal{C}$中规划或生成$St$，其形式化表示为$St\leftarrow\mathcal{G}(q,\rho,\mathcal{C})$。该过程确保了一个结构化且连贯的响应策略，将工具的能力与查询的具体要求对接，以提供有效的解决方案。

### 2.2 代码生成

Reflexion Shinn 等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib29)）与大型语言模型（LLM）$\rho$和Python解释器$\mathcal{I}$的结合，通过实现迭代代码优化，显著推进了编码任务。该方法利用反馈$\mathcal{F}$，通过迭代解决异常并增强初始代码输出$c$，其测试用例由$\rho$本身动态生成。这确保了在函数调用模块内进行全面的验证和优化，最终生成最终代码$c_{n}$。该方法提高了代码质量，符合现代标准，标志着自动化代码开发和验证的一次飞跃。迭代代码生成过程可以通过方程式[1](https://arxiv.org/html/2402.10051v1#S2.E1 "1 ‣ 2.2 Code Generation ‣ 2 Preliminaries ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")数学表示。

|  | $\begin{gathered}c_{i}\leftarrow\rho(q,\textit{feedback}_{i-1},c_{i-1})\\ \textit{output}\leftarrow\mathcal{I}(c_{i})\\ \textit{feedback}_{i},\textit{verified}\leftarrow\mathcal{F}(output)\\ \end{gathered}$ |  | (1) |
| --- | --- | --- | --- |

## 3 SwissNYF

### 3.1 概述

![参见说明](img/404ffdeabee906e3a5be1b13cf61a3a9.png)

图3：我们提出的方法在SwissNYF中与TOPGUN的详细流程图

在本节中，我们介绍了SwissNYF，这是一个使基于LLM的代理能够有效地在动作空间中导航，从而在黑箱场景中识别有效解决方案的套件。SwissNYF由五个主要组件组成，即功能签名生成$\mathcal{P}$、语料库与检索器$\mathcal{C}$、规划器$\mathcal{G}$、验证器$\mathcal{F}$和解析器$\mathcal{K}$，如图[2](https://arxiv.org/html/2402.10051v1#S2.F2 "Figure 2 ‣ 2.1 Problem Formulation ‣ 2 Preliminaries ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")所示。我们将在随后的子节中解释该流程的各个组成部分。

![参见说明](img/e61dfca571a01546cdbc1fbe222a6ba9.png)

(a) CodeSynth $\mathcal{P}$ 算法的示例输出

![参考图注](img/bf57704e267a1ba6dc7ce250d54eafb4.png)

(b) TOPGUN $\mathcal{G}$ 算法的示例输出

图 4：分别由 CodeSynth 和 TOPGUN 生成的伪函数与工具规划的示意图。

### 3.2 函数签名生成

函数签名，作为伪 API 被构思，用于模拟基于给定工具描述的真实 API 函数的行为。这种模拟在我们工具规划方法论中有两个主要作用：首先，它们充当实际 API 调用的替代品，从而使 LLM 能够更高效地规划和执行任务；其次，它们被视为预定义的函数，便于将工具增强转化为类似代码生成的任务，使用这些伪函数。这些函数签名通过其文档字符串和与工具描述一致的示例返回对象来区分，从而为规划者提供有效的手段来有效应对用户查询。在我们的 SwissNYF 实现中，我们采用了一种直接而有效的生成函数签名的方法，称为 CodeSynth。此方法的有效性将在[4.3](https://arxiv.org/html/2402.10051v1#S4.SS3 "4.3 CodeSynth 评估 ‣ 4 实验 ‣ wissNYF：工具驱动的 LLM 代理在黑盒环境下的应用")中进一步分析。

#### 3.2.1 CodeSynth

对于给定的工具描述集合 $t\in\mathcal{T}$，我们引导大语言模型（LLM）$\rho$ 生成伪函数实现，表示为 $\hat{t}$。我们的主要目标是确保这些伪函数的参数和返回类型与其描述保持一致。此外，我们为每个伪函数编写详细的文档字符串，以便后续处理。CodeSynth 的一个关键方面是包含一个示例返回值，旨在模拟返回对象在验证过程中可能经历的所有操作。CodeSynth 生成的输出如图 [3(a)](https://arxiv.org/html/2402.10051v1#S3.F3.sf1 "3(a) ‣ 图 4 ‣ 3.1 概述 ‣ 3 SwissNYF ‣ wissNYF：工具驱动的 LLM 代理在黑盒环境下的应用")所示。此外，通过 Reflexion 进行验证的代码生成，如公式 [1](https://arxiv.org/html/2402.10051v1#S2.E1 "1 ‣ 2.2 代码生成 ‣ 2 初步工作 ‣ wissNYF：工具驱动的 LLM 代理在黑盒环境下的应用")所述，进一步提升了这一模块的有效性。最终，CodeSynth 中应用的方法可以总结为算法 [1](https://arxiv.org/html/2402.10051v1#algorithm1 "1 ‣ 3.2.1 CodeSynth ‣ 3.2 函数签名生成 ‣ 3 SwissNYF ‣ wissNYF：工具驱动的 LLM 代理在黑盒环境下的应用")。

输入：$\rho$: 大型语言模型；$T$: 工具描述；$\mathcal{I}$: python 解释器；$\mathcal{F}(\mathcal{I})$: $\mathcal{I}$的反射反馈；$\mathcal{C}$: 伪工具的空语料库对于*$t=1,2,\cdots,T$*  do    $\hat{t}_{0}\leftarrow\rho(t)$    //伪代码    $verified\leftarrow\mathcal{I}(\hat{t}_{0})$    while *未验证* do    $\hat{t}_{i}\leftarrow\rho(t,feedback_{i-1},\hat{t}_{i-1})$    $feedback_{i},\ verified\leftarrow\mathcal{F}(\mathcal{I}(\hat{t}_{i}))$    更新 $\mathcal{C}\leftarrow\hat{t}_{n}$    // 更新语料库输出：一个经验证的伪函数语料库 $\mathcal{C}$

算法 1 $\mathcal{P}$: CodeSynth

通过与解释器结合使用功能调用模块，我们对伪函数进行了严格的测试，涵盖了广泛的现实世界场景。这种方法确保了测试用例的全面性，并能够反映实际的函数使用情况，使我们能够收集到关于伪函数性能的详细反馈。这些反馈对于伪函数的迭代改进至关重要，显著提升了它们在实际环境中的可靠性和适用性。CodeSynth的提示可以在[A.1](https://arxiv.org/html/2402.10051v1#A1.SS1 "A.1 Prompts ‣ Appendix A Appendix ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")中进行记录。

### 3.3 语料库与检索器

函数签名是我们方法论的关键组成部分，它们被系统地存储在一个语料库中，供未来的任何规划系统使用。该语料库有助于工具描述的索引，从而根据索引精确地检索出最合适的工具。值得注意的是，文献中记录了几种为此目的设计的先进检索系统，显示出卓越的准确性。这些系统包括ToolBench IR Qin et al. ([2023](https://arxiv.org/html/2402.10051v1#bib.bib25))，APIRetriever Zan et al. ([2022](https://arxiv.org/html/2402.10051v1#bib.bib35))，Instructor-XL Su et al. ([2022](https://arxiv.org/html/2402.10051v1#bib.bib30))，以及GEAR Lu et al. ([2023b](https://arxiv.org/html/2402.10051v1#bib.bib18))。我们的框架结合了这些检索器，默认选用Instructor-XL，因为它已被证明效果显著。此外，我们正在积极探索集成AnyTool的层级API检索器Du et al. ([2024](https://arxiv.org/html/2402.10051v1#bib.bib9))，预计将显著提升我们的工具检索能力。通过战略性地引入多种检索器，确保我们的系统在识别最合适的工具时保持多功能性和高效性，与检索技术的最新进展保持同步。

### 3.4 规划器

我们在框架中实现了两种规划方法。第一种方法利用了修改版的反向链 Zhang 等人（[2023b](https://arxiv.org/html/2402.10051v1#bib.bib37)）来通过将任务分解为子任务并使用原始反向链技术创建子树，以支持多个终端函数调用。第二种方法 TOPGUN，是我们提出的基于代码驱动的规划算法，旨在提升速度、效率、一致性和准确性，尤其在黑箱场景下表现出色。TOPGUN 提供了一种简化的替代方案，相比传统规划方法，它在复杂系统导航和任务执行中提供了更高的可靠性和成本效益。

#### 3.4.1 TOPGUN

输入：q：查询；$\rho$：大型语言模型；$T$：工具描述；$\mathcal{I}$：Python 解释器；$\mathcal{F}(\mathcal{I})$：$\mathcal{I}$ 的反射反馈；$\mathcal{C}$：伪工具的空语料库；$\mathcal{P}$：Codesynth，$\mathcal{K}$：解析器初始化 $\mathcal{\hat{T}}\leftarrow\mathcal{P}(\rho,T,\mathcal{I},\mathcal{F},\mathcal{C})$  // 伪工具 $c_{0}\leftarrow\ \rho(q,\mathcal{\hat{T}},\mathcal{C})$  // 查询的代码 $verified\leftarrow\mathcal{I}(c_{0},\mathcal{\hat{T}})$  // 用伪工具验证 while *未验证* do $c_{i}\leftarrow\ \rho(q,\mathcal{\hat{T}},\mathcal{\hat{C}},feedback_{i-1},c_{i-1})$ $feedback_{i},\ verified\leftarrow\mathcal{F}(\mathcal{I}(c_{i},\mathcal{\hat{T}}))$$\mathcal{S}t\leftarrow\mathcal{K}(c_{n})$  // 解决方案轨迹输出：解决方案轨迹 $\mathcal{S}t$ 和 $c_{n}$ 执行和评估的代码

算法 2 $\mathcal{G}$：TOPGUN

TOPGUN，代表工具编排与程序合成以推广未知系统（Tool Orchestration and Program synthesis for Generalizing over UNknown systems），通过将用户查询 $q$ 的挑战重新定义为代码生成任务，重新定义了应对用户查询的方法。利用伪函数 $\mathcal{\hat{T}}$ 作为TOPGUN可用的函数，使得代理能够构建一个准确的函数调用序列 $c_{0}\leftarrow\rho(q,\mathcal{\hat{T}},\mathcal{C})$，这一过程在图[3(b)](https://arxiv.org/html/2402.10051v1#S3.F3.sf2 "3(b) ‣ Figure 4 ‣ 3.1 Overview ‣ 3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")中得到了有效展示。借助于方程[1](https://arxiv.org/html/2402.10051v1#S2.E1 "1 ‣ 2.2 Code Generation ‣ 2 Preliminaries ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")中详细描述的反射机制，框架通过迭代不断优化对查询的响应。将这些组件合成的综合算法展示在算法[2](https://arxiv.org/html/2402.10051v1#algorithm2 "2 ‣ 3.4.1 TOPGUN ‣ 3.4 Planner ‣ 3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")中，展示了TOPGUN在不同解决路径中的导航能力。与传统的基于遍历的技术不同，TOPGUN利用LLM固有的代码生成能力，促进了更直接和高效的解决过程。这一区别不仅通过精确定位问题提升了效率，还确保了在黑盒场景中的适应性，同时优化了灰盒设置中的性能。关于TOPGUN的详细管道概述见图[3(b)](https://arxiv.org/html/2402.10051v1#S3.F3.sf2 "3(b) ‣ Figure 4 ‣ 3.1 Overview ‣ 3 SwissNYF ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")。相关提示文档可参考[A.1](https://arxiv.org/html/2402.10051v1#A1.SS1 "A.1 Prompts ‣ Appendix A Appendix ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")。

![参见标题](img/d47a80216d520a52d217f9809b6096fb.png)

图5：TOPGUN中自我反射机制的示意图

### 3.5 验证器

验证与规划器 $\mathcal{G}$ 的功能密切相关，依赖于 $\mathcal{G}$ 输出的性质以及其整合反馈的能力。尽管验证最初作为解析前的准备步骤，但它也在通过提供反馈来改进输出方面发挥着至关重要的作用，这些反馈可供 $\mathcal{G}$ 在后续迭代中使用。

在我们的框架中，我们利用了Reflexion Shinn 等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib29)）的工作，该工作在方程式 [1](https://arxiv.org/html/2402.10051v1#S2.E1 "1 ‣ 2.2 代码生成 ‣ 2 初步介绍 ‣ wissNYF: 工具驱动的LLM代理") 中进行了详细描述，并在算法 [2](https://arxiv.org/html/2402.10051v1#algorithm2 "2 ‣ 3.4.1 TOPGUN ‣ 3.4 规划器 ‣ 3 SwissNYF ‣ wissNYF: 工具驱动的LLM代理") 中进行了阐明，以便在TOPGUN方法中无缝集成验证和反馈。这消除了对额外功能调用模块的需求，而是专注于直接执行与用户查询相关的代码。此方法在图 [5](https://arxiv.org/html/2402.10051v1#S3.F5 "图5 ‣ 3.4.1 TOPGUN ‣ 3.4 规划器 ‣ 3 SwissNYF ‣ wissNYF: 工具驱动的LLM代理") 中进行了说明，提供了该概念的可视化表示。

### 3.6 解析器

解析器 $\mathcal{K}$ 类似于验证器 $\mathcal{F}$，其功能本质上依赖于规划器 $\mathcal{G}$。其关键输出是一个明确定义的解决方案轨迹 $St$，描述了解决查询所设计的工具应用顺序。在采用反向链技术时，我们的方法通过 LLM $\rho$ 的能力，将各个子树合成一个单一的、全面的树。通过在合并过程中巧妙地重用各个子树中的元素，过程的效能得到了显著提升。

相反，对于TOPGUN方法，我们采用了Fischer 等人（[2007](https://arxiv.org/html/2402.10051v1#bib.bib10)）提出的抽象语法树（AST）范式，将程序划分为基本的函数调用，并指定它们的参数和返回值。这种划分对于构建一系列有序的工具调用至关重要。这个精心安排的系列，记作 $St$，可以简洁地表示为 $St\leftarrow\mathcal{K}(c_{n})$。

如图 [3](https://arxiv.org/html/2402.10051v1#S3.F3 "图3 ‣ 3.1 概述 ‣ 3 SwissNYF ‣ wissNYF: 工具驱动的LLM代理") 所示，整个流程通过将多种组件集成在一起，旨在通过在SwissNYF框架内的工具战略协调，有效地解决用户查询。

表1：不同候选模型和参考模型在G1数据集上的胜率

| 候选模型 | 参考模型 | G1-指令 | G1-工具 | G1-类别 |
| --- | --- | --- | --- | --- |
| T.LLaMA ReACT | ChatGPT ReACT | 45.0 | 42.0 | 47.5 |
| T.LLaMA DFSDT | ChatGPT ReACT | 55.0 | 55.3 | 54.5 |
| T.LLaMA DFSDT+Ret | ChatGPT ReACT | 62.3 | 59.0 | 55.0 |
| ChatGPT DFSDT | ChatGPT ReACT | 60.5 | 62.0 | 57.3 |
| GPT4 ReACT | ChatGPT ReACT | 60.0 | 58.8 | 63.5 |
| GPT4 DFSDT | ChatGPT ReACT | 67.5 | 67.8 | 66.5 |
| GPT4 TOPGUN | ChatGPT ReACT | 88.192 | 87.46 | 87.15 |
| GPT4 TOPGUN | ChatGPT DFSDT | 78.49 | 77.55 | 76.24 |
| GPT4 TOPGUN | T.LLaMA ReACT | 86.72 | 82.94 | 80.80 |
| GPT4 TOPGUN | T.LLaMA DFSDT | 81.75 | 75.51 | 73.81 |
| GPT4 TOPGUN | T.LLaMA DFSDT+Ret | 80.35 | 77.11 | 75.39 |
| GPT4 TOPGUN | GPT4 ReACT | 82.996 | 79.956 | 77.633 |
| GPT4 TOPGUN | GPT4 DFSDT | 82.065 | 73.69 | 71.14 |

## 4 实验

工具规划数据集虽然多样，但往往不足以支持多轮和多调用对话，正如Schick等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib28)）和Tang等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib31)）的研究所展示的那样，并且缺乏精确的评估指标，增加了全面评估的难度。即使像Qin等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib25)）提出的ToolBench这样的大型数据集，也难以与黑箱设置对齐，从而给在这种情境下评估工具规划带来了重大挑战。

我们的评估采用了Qin等人提出的ToolBench基准（[2023](https://arxiv.org/html/2402.10051v1#bib.bib25)）以及为不常见代码库特别策划的数据集，评估是在灰箱（[4.1](https://arxiv.org/html/2402.10051v1#S4.SS1 "4.1 灰箱评估 ‣ 4 实验 ‣ wissNYF：面向黑箱设置的工具驱动LLM代理")）和黑箱（[4.2](https://arxiv.org/html/2402.10051v1#S4.SS2 "4.2 黑箱评估 ‣ 4 实验 ‣ wissNYF：面向黑箱设置的工具驱动LLM代理")）设置下进行的。我们使用胜率、令牌数和成功率将我们的TOPGUN方法与现有方法进行了基准对比。此外，我们还审查了CodeSynth（$\mathcal{P}$）对规划器（$\mathcal{G}$）性能的影响，并独立评估了其生成有效函数签名的能力，这些签名充当伪函数，详细内容请参见[4.3](https://arxiv.org/html/2402.10051v1#S4.SS3 "4.3 CodeSynth评估 ‣ 4 实验 ‣ wissNYF：面向黑箱设置的工具驱动LLM代理")节。

表 2：不同候选和参考模型在G2、G3集上的胜率以及所有集合的平均值

| 候选 | 参考 | G2-指令 | G2-类别 | G3-指令 | 平均 |
| --- | --- | --- | --- | --- | --- |
| T.LLaMA ReACT | ChatGPT ReACT | 50.8 | 41.8 | 55.0 | 47.0 |
| T.LLaMA DFSDT | ChatGPT ReACT | 68.5 | 58.0 | 69.0 | 60.0 |
| T.LLaMA DFSDT+Ret | ChatGPT ReACT | 68.5 | 60.8 | 73.0 | 63.1 |
| ChatGPT DFSDT | ChatGPT ReACT | 72.0 | 64.8 | 69.0 | 64.3 |
| GPT4 ReACT | ChatGPT ReACT | 65.8 | 60.3 | 78.0 | 64.0 |
| GPT4 DFSDT | ChatGPT ReACT | 73.3 | 63.3 | 84.0 | 70.4 |
| GPT4 TOPGUN | ChatGPT ReACT | 87.59 | 78.78 | 90.05 | 86.54 |
| GPT4 TOPGUN | ChatGPT DFSDT | 81.63 | 73.07 | 85.26 | 78.71 |
| GPT4 TOPGUN | T.LLaMA ReACT | 86.24 | 77.71 | 93.23 | 84.61 |
| GPT4 TOPGUN | T.LLaMA DFSDT | 78.31 | 71.80 | 89.47 | 78.44 |
| GPT4 TOPGUN | T.LLaMA DFSDT+Ret | 83.07 | 72.92 | 87.82 | 79.44 |
| GPT4 TOPGUN | GPT4 ReACT | 78.61 | 73.75 | 93.68 | 80.27 |
| GPT4 TOPGUN | GPT4 DFSDT | 73.92 | 71.35 | 79.25 | 78.59 |

### 4.1 灰箱评估

为了评估 TOPGUN 的性能，并将其与其他灰盒方法（如 ReACT 和 DFSDT）进行比较，我们保持了管道的完整性，同时调整评估过程，将实际功能替代输出解轨迹中的伪功能。该方法有效地保持了我们的黑盒管道完整性，并将其转化为灰盒评估框架。对于评估目的，响应和最终答案的必要性促使我们采用了这一混合策略。在实际场景中，这类似于通用规划器为客户提供策略，客户随后用实际功能替代伪功能进行执行。对于此次评估，我们使用了 ToolBench，具体细节见 Qin 等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib25)）的研究，并在数据集中提供的所有问题类别上进行了分析。关于评估方法和 ToolBench 应用的进一步说明，请参见 [A.2](https://arxiv.org/html/2402.10051v1#A1.SS2 "A.2 ToolBench 灰盒评估 ‣ 附录 A 附录 ‣ wissNYF: 工具驱动的 LLM 代理在黑盒环境下的应用")。

结果：ToolLLaMa-ReACT、ToolLLaMA-DFSDT、ChatGPT-DFSDT、GPT4-DFSDT 和 GPT4-TOPGUN 与 ChatGPT-ReACT 和 GPT4-TOPGUN 的胜率比较结果已总结，平均值来自每对模型运行7次，详细信息见表格 [1](https://arxiv.org/html/2402.10051v1#S3.T1 "表1 ‣ 3.6 解析器 ‣ 3 SwissNYF ‣ wissNYF: 工具驱动的 LLM 代理在黑盒环境下的应用") 和 [2](https://arxiv.org/html/2402.10051v1#S4.T2 "表2 ‣ 4 实验 ‣ wissNYF: 工具驱动的 LLM 代理在黑盒环境下的应用")。TOPGUN 在所有类别上均显著超过了 ReAct 和 DFSDT，针对 GPT4-ReACT 的胜率为 80.27%，针对 GPT4-DFSDT 为 78.59%，针对 ChatGPT-ReACT 为 86.54%，分别提高了 22.54% 和 16.14%。这些结果突显了 TOPGUN 在不同条件下创建符合偏好评估标准的工具计划的卓越能力。

### 4.2 黑盒评估

![请参见图注](img/f5d4803bcd41b4ce97cfba1c0ab0e0ae.png)

图6：黑盒环境中各个方法的平均 Token 消耗。

利用秦等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib25)）的 数据生成管道，我们构建了一个黑盒场景数据集，包含了36个LLaMa-Hub LlamaIndex（[2023](https://arxiv.org/html/2402.10051v1#bib.bib16)）工具和来自私有库的独特函数。遵循Zan等人（[2022](https://arxiv.org/html/2402.10051v1#bib.bib35)）的方法，我们将Pandas和Numpy转换为Monkey和BeatNum包，并重命名了所有内部函数和结构，以测试在没有LLM先验知识的情况下，规划器的泛化能力。该数据集详细信息可见于[A.1](https://arxiv.org/html/2402.10051v1#A1.SS1 "A.1 Prompts ‣ Appendix A Appendix ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")，重点关注解题轨迹的准确性，每个查询设计了一个正确的路径。在手动标注后，该数据集包含100个查询和162个工具，样本和TOPGUN结果可见于[A.3.2](https://arxiv.org/html/2402.10051v1#A1.SS3.SSS2 "A.3.2 Queries Example ‣ A.3 PrivateEval Dataset ‣ Appendix A Appendix ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")和[A.5.2](https://arxiv.org/html/2402.10051v1#A1.SS5.SSS2 "A.5.2 PrivateEval ‣ A.5 TOPGUN Examples ‣ Appendix A Appendix ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")。

结果：黑盒评估采用了TOPGUN和修订版的逆向链，并利用$\mathcal{P}$函数签名进行全面的黑盒方法。TOPGUN超越了逆向链，并与GPT4-DFSDT和GPT4-ReACT在灰盒评估中进行了比较，重点比较了输出轨迹。成功率是通过与真实轨迹的精确匹配计算得出的，并在十次迭代中取平均值，结果记录在表格[4](https://arxiv.org/html/2402.10051v1#S4.T4 "Table 4 ‣ 4.3 CodeSynth Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")中。图[6](https://arxiv.org/html/2402.10051v1#S4.F6 "Figure 6 ‣ 4.2 Black Box Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting")详细展示了每个算法在每个查询中的平均Token使用情况，突出显示了TOPGUN在黑盒场景中生成精确且高效的工具计划方面的有效性和效率，证明了其在不同数据集中的适应性。

注意：使用ToolBench进行黑盒评估不可行，因为ToolEval的指标，如通过率和胜率，依赖于中间工具响应和最终答案。

### 4.3 CodeSynth 评估

为了评估 CodeSynth 生成的函数签名的质量，我们采用了由 Parisotto 等人（[2017](https://arxiv.org/html/2402.10051v1#bib.bib23)）和 Nye 等人（[2021](https://arxiv.org/html/2402.10051v1#bib.bib20)）提出的神经符号表示方法。这些表示旨在捕捉给定程序的抽象语义本质，与我们的目标高度一致。我们的评估涵盖了 HumanEval-X（Zheng 等人，[2023](https://arxiv.org/html/2402.10051v1#bib.bib38)）和 MBPP（Austin 等人，[2021](https://arxiv.org/html/2402.10051v1#bib.bib3)）数据集的 Python 子集。受 Ma 等人（[2023](https://arxiv.org/html/2402.10051v1#bib.bib19)）提出的语义探测模型启发，我们构建了合成伪函数和真实代码的语义表示。利用 tree-sitter（Brunsfeld 等人，[2024](https://arxiv.org/html/2402.10051v1#bib.bib5)）包，我们形成了抽象语法树，专注于仅对函数定义块计算 F1 分数，排除函数体块。因此，最终的度量准确地代表了我们在 CodeSynth 中的目标。附录 [A.4.1](https://arxiv.org/html/2402.10051v1#A1.SS4.SSS1 "A.4.1 HumanEval-X ‣ A.4 CodeSynth Examples ‣ 附录 A 附录 ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting") 中可以查阅使用 HumanEval-X 数据集合成的函数签名示例。

结果：我们在多个反射周期中评估了 CodeSynth，跟踪每个周期的 F1 分数，以说明函数签名质量的一致性提升，如表 [4](https://arxiv.org/html/2402.10051v1#S4.T4 "Table 4 ‣ 4.3 CodeSynth Evaluation ‣ 4 Experiments ‣ wissNYF: Tool Grounded LLM Agents for Black Box Setting") 所示。CodeSynth 在 HumanEval-X 和 MBPP 数据集上显著提高了 F1 分数，在第五次迭代中分别达到了 0.844 和 0.912 的初始分数，最终获得了完美的 1.0 分。这些发现突显了 CodeSynth 生成的函数签名与目标函数语义的高度相似性。

表 3: 黑盒设置中方法的比较

| 方法 | 成功率 |
| --- | --- |
| GPT4-TOPGUN | 70.58 |
| GPT4-DFSDT | 61.45 |
| GPT4-ReAct | 45.45 |
| GPT4-ReverseChain | 43.75 |
| 数据集 | 最大反射迭代的 F1 分数 |
| @1 | @2 | @3 | @4 | @5 |
| HumanEval-X | 0.844 | 0.894 | 0.965 | 0.983 | 1.00 |
| MBPP | 0.912 | 0.963 | 0.994 | 1.00 | 1.00 |

表 3: 黑盒设置中方法的比较

表 4: CodeSynth 评估，用于分析 Reflexions 对函数签名 AST 的改进

## 5 结论

在这项工作中，我们解决了在黑箱设置中工具规划的挑战，其中无法直接访问API调用及其实现，这引发了关于API交互中成本效率和隐私的问题。我们提出了SwissNYF，一个旨在为大型语言模型（LLMs）提供有效应对这些场景的能力的综合框架。SwissNYF的核心是巧妙的函数签名生成，使规划者可以依赖工具描述，从而绕过实际API执行的需求。我们进一步提出了TOPGUN，一种代码驱动的规划方法，利用LLM的代码生成能力，为黑箱环境提供了一种稳健的解决方案。我们在各种工具集和设置中的广泛评估证明了我们的方法在传统工具规划策略中的优越性，验证了其有效性和可靠性。通过SwissNYF和TOPGUN，我们建立了工具规划中的一个激动人心的新兴范式。我们预见到SwissNYF将成为黑箱工具使用的中心枢纽，鼓励未来在开发黑箱场景策略方面的进展，从而在LLM增强应用领域实现高效、注重隐私的工具规划的重大飞跃。

## 参考文献

+   Achiam等人（2023）Josh Achiam、Steven Adler、Sandhini Agarwal、Lama Ahmad、Ilge Akkaya、Florencia Leoni Aleman、Diogo Almeida、Janko Altenschmidt、Sam Altman、Shyamal Anadkat等人。Gpt-4技术报告。*arXiv预印本arXiv:2303.08774*，2023年。

+   Anil等人（2023）Rohan Anil、Andrew M Dai、Orhan Firat、Melvin Johnson、Dmitry Lepikhin、Alexandre Passos、Siamak Shakeri、Emanuel Taropa、Paige Bailey、Zhifeng Chen等人。Palm 2技术报告。*arXiv预印本arXiv:2305.10403*，2023年。

+   Austin等人（2021）Jacob Austin、Augustus Odena、Maxwell Nye、Maarten Bosma、Henryk Michalewski、David Dohan、Ellen Jiang、Carrie Cai、Michael Terry、Quoc Le等人。使用大型语言模型进行程序合成。*arXiv预印本arXiv:2108.07732*，2021年。

+   Brown等人（2020）Tom Brown、Benjamin Mann、Nick Ryder、Melanie Subbiah、Jared D Kaplan、Prafulla Dhariwal、Arvind Neelakantan、Pranav Shyam、Girish Sastry、Amanda Askell等人。语言模型是少样本学习者。*神经信息处理系统进展*，33:1877–1901，2020年。

+   Brunsfeld等人（2024）Max Brunsfeld、Andrew Hlynskyi、Amaan Qureshi、Patrick Thomson、Josh Vera、Phil Turnbull、Timothy Clem、Douglas Creager、Andrew Helwer、dundargoc、Rob Rix、Daumantas Kavolis、Hendrik van Antwerpen、Michael Davis、Ika、Tuan-Anh Nguyen、Amin Yahyaabadi、Stafford Brunk、Matt Massicotte和George Fraser。tree-sitter/tree-sitter：v0.21.0-pre-release-1，2024年。URL [https://doi.org/10.5281/zenodo.10638807](https://doi.org/10.5281/zenodo.10638807)。

+   Chen 等人 (2021) Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman 等。评估在代码上训练的大型语言模型。*arXiv 预印本 arXiv:2107.03374*，2021年。

+   Chen 等人 (2022) Wenhu Chen, Xueguang Ma, Xinyi Wang, 和 William W Cohen。思维提示程序：将计算与推理解耦以应对数值推理任务。*arXiv 预印本 arXiv:2211.12588*，2022年。

+   Chowdhery 等人 (2023) Aakanksha Chowdhery, Sharan Narang, Jacob Devlin, Maarten Bosma, Gaurav Mishra, Adam Roberts, Paul Barham, Hyung Won Chung, Charles Sutton, Sebastian Gehrmann 等。Palm: 通过路径扩展语言建模。*机器学习研究杂志*，24(240):1–113，2023年。

+   Du 等人 (2024) Yu Du, Fangyun Wei, 和 Hongyang Zhang。Anytool: 自我反思型分层代理，用于大规模 API 调用。*arXiv 预印本 arXiv:2402.04253*，2024年。

+   Fischer 等人 (2007) Gregor Fischer, J Lusiardi, 和 J Wolff Von Gudenberg。抽象语法树及其在模型驱动软件开发中的作用。*国际软件工程进展会议 (ICSEA 2007)*，第38–38页。IEEE，2007年。

+   Gao 等人 (2023) Luyu Gao, Aman Madaan, Shuyan Zhou, Uri Alon, Pengfei Liu, Yiming Yang, Jamie Callan 和 Graham Neubig。Pal: 程序辅助语言模型。在*国际机器学习会议*，第10764–10799页。PMLR，2023年。

+   Gou 等人 (2023) Zhibin Gou, Zhihong Shao, Yeyun Gong, Yujiu Yang, Minlie Huang, Nan Duan, Weizhu Chen 等。Tora: 一种集成工具的推理代理，用于数学问题求解。*arXiv 预印本 arXiv:2309.17452*，2023年。

+   Hao 等人 (2023) Shibo Hao, Tianyang Liu, Zhen Wang 和 Zhiting Hu。Toolkengpt: 通过工具嵌入增强冻结语言模型与海量工具的结合。*arXiv 预印本 arXiv:2305.11554*，2023年。

+   Huang & Chang (2023) Jie Huang 和 Kevin Chen-Chuan Chang。迈向大规模语言模型中的推理：一项调查。在 Anna Rogers, Jordan Boyd-Graber 和 Naoaki Okazaki（编），*计算语言学会成果：ACL 2023*，第1049–1065页，加拿大多伦多，2023年7月。计算语言学会。doi: [10.18653/v1/2023.findings-acl.67](10.18653/v1/2023.findings-acl.67)。网址 [https://aclanthology.org/2023.findings-acl.67](https://aclanthology.org/2023.findings-acl.67)。

+   Li 等人 (2023) Chengshu Li, Jacky Liang, Andy Zeng, Xinyun Chen, Karol Hausman, Dorsa Sadigh, Sergey Levine, Li Fei-Fei, Fei Xia 和 Brian Ichter。Chain of code: 使用语言模型增强的代码模拟器进行推理。*arXiv 预印本 arXiv:2312.04474*，2023年。

+   LlamaIndex (2023) LlamaIndex。Llamahub，2023年。网址 [https://web.archive.org/web/20231229215448/https://llamahub.ai/](https://web.archive.org/web/20231229215448/https://llamahub.ai/)。

+   Lu等人（2023a）Pan Lu、Baolin Peng、Hao Cheng、Michel Galley、Kai-Wei Chang、Ying Nian Wu、Song-Chun Zhu和Jianfeng Gao。Chameleon：大语言模型的即插即用组合推理。*arXiv预印本 arXiv:2304.09842*，2023a。

+   Lu等人（2023b）Yining Lu、Haoping Yu和Daniel Khashabi。Gear：通过通用且高效的工具解析增强语言模型。*arXiv预印本 arXiv:2307.08775*，2023b。

+   Ma等人（2023）Wei Ma、Mengjie Zhao、Xiaofei Xie、Qiang Hu、Shangqing Liu、Jie Zhang、Wenhan Wang和Yang Liu。代码预训练模型在学习代码语法和语义方面是否强大？2023。

+   Nye等人（2021）Maxwell Nye、Yewen Pu、Matthew Bowers、Jacob Andreas、Joshua B. Tenenbaum和Armando Solar-Lezama。用混合抽象语义表示部分程序。在*国际学习表征会议*，2021。网址[https://openreview.net/forum?id=mCtadqIxOJ](https://openreview.net/forum?id=mCtadqIxOJ)。

+   Paranjape等人（2023）Bhargavi Paranjape、Scott Lundberg、Sameer Singh、Hannaneh Hajishirzi、Luke Zettlemoyer和Marco Tulio Ribeiro。Art：自动化的多步骤推理和工具使用，为大语言模型服务。*arXiv预印本 arXiv:2303.09014*，2023。

+   Parisi等人（2022）Aaron Parisi、Yao Zhao和Noah Fiedel。Talm：工具增强语言模型。*arXiv预印本 arXiv:2205.12255*，2022。

+   Parisotto等人（2017）Emilio Parisotto、Abdel rahman Mohamed、Rishabh Singh、Lihong Li、Dengyong Zhou和Pushmeet Kohli。神经符号程序合成。在*国际学习表征会议*，2017。网址[https://openreview.net/forum?id=rJ0JwFcex](https://openreview.net/forum?id=rJ0JwFcex)。

+   Patil等人（2023）Shishir G Patil、Tianjun Zhang、Xin Wang和Joseph E Gonzalez。Gorilla：与大量API连接的大语言模型。*arXiv预印本 arXiv:2305.15334*，2023。

+   Qin等人（2023）Yujia Qin、Shihao Liang、Yining Ye、Kunlun Zhu、Lan Yan、Yaxi Lu、Yankai Lin、Xin Cong、Xiangru Tang、Bill Qian等人。Toolllm：帮助大语言模型掌握16000+个现实世界API。*arXiv预印本 arXiv:2307.16789*，2023。

+   Radford等人（2018）Alec Radford、Karthik Narasimhan、Tim Salimans、Ilya Sutskever等人。通过生成预训练提高语言理解。2018。

+   Radford等人（2019）Alec Radford、Jeffrey Wu、Rewon Child、David Luan、Dario Amodei、Ilya Sutskever等人。语言模型是无监督的多任务学习者。*OpenAI博客*，1(8):9，2019。

+   Schick等人（2023）Timo Schick、Jane Dwivedi-Yu、Roberto Dessì、Roberta Raileanu、Maria Lomeli、Luke Zettlemoyer、Nicola Cancedda和Thomas Scialom。Toolformer：语言模型可以自我学习使用工具。*arXiv预印本 arXiv:2302.04761*，2023。

+   Shinn等人（2023）Noah Shinn、Federico Cassano、Ashwin Gopinath、Karthik R Narasimhan和Shunyu Yao。Reflexion：具有语言强化学习的语言代理。在*第三十七届神经信息处理系统会议*，2023。

+   Su 等人（2022）Hongjin Su, Weijia Shi, Jungo Kasai, Yizhong Wang, Yushi Hu, Mari Ostendorf, Wen-tau Yih, Noah A Smith, Luke Zettlemoyer 和 Tao Yu。一个嵌入器，任何任务：针对文本嵌入的指令微调。*arXiv 预印本 arXiv:2212.09741*，2022。

+   Tang 等人（2023）Qiaoyu Tang, Ziliang Deng, Hongyu Lin, Xianpei Han, Qiao Liang 和 Le Sun。Toolalpaca：面向语言模型的通用工具学习，基于 3000 个模拟案例。*arXiv 预印本 arXiv:2306.05301*，2023。

+   Xu 等人（2023）Yiheng Xu, Hongjin Su, Chen Xing, Boyu Mi, Qian Liu, Weijia Shi, Binyuan Hui, Fan Zhou, Yitao Liu, Tianbao Xie 等人。Lemur：为语言代理协调自然语言与代码。*arXiv 预印本 arXiv:2310.06830*，2023。

+   Yang 等人（2023）Rui Yang, Lin Song, Yanwei Li, Sijie Zhao, Yixiao Ge, Xiu Li 和 Ying Shan。Gpt4tools：通过自我指导教大型语言模型使用工具，2023。

+   Yao 等人（2022）Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan 和 Yuan Cao。React：在语言模型中协同推理与行动。*arXiv 预印本 arXiv:2210.03629*，2022。

+   Zan 等人（2022）Daoguang Zan, Bei Chen, Zeqi Lin, Bei Guan, Yongji Wang 和 Jian-Guang Lou。当语言模型遇到私人库时。*arXiv 预印本 arXiv:2210.17236*，2022。

+   Zhang 等人（2023a）Beichen Zhang, Kun Zhou, Xilin Wei, Wayne Xin Zhao, Jing Sha, Shijin Wang 和 Ji-Rong Wen。评估和改进工具增强的计算密集型数学推理，2023a。

+   Zhang 等人（2023b）Yinger Zhang, Hui Cai, Yicheng Chen, Rui Sun 和 Jing Zheng。反向链：一种通用规则，使 LLMs 掌握多 API 规划。*arXiv 预印本 arXiv:2310.04474*，2023b。

+   Zheng 等人（2023）Qinkai Zheng, Xiao Xia, Xu Zou, Yuxiao Dong, Shan Wang, Yufei Xue, Lei Shen, Zihan Wang, Andi Wang, Yang Li 等人。Codegeex：一个用于代码生成的预训练模型，并在 humaneval-x 上进行多语言基准测试。发表于 *第29届 ACM SIGKDD 知识发现与数据挖掘大会论文集*，第 5673–5684 页，2023年。

+   Zhou 等人（2023a）Andy Zhou, Kai Yan, Michal Shlapentokh-Rothman, Haohan Wang 和 Yu-Xiong Wang。语言代理树搜索统一了语言模型中的推理、行动和规划。*arXiv 预印本 arXiv:2310.04406*，2023a。

+   Zhou 等人（2023b）Aojun Zhou, Ke Wang, Zimu Lu, Weikang Shi, Sichun Luo, Zipeng Qin, Shaoqing Lu, Anya Jia, Linqi Song, Mingjie Zhan 等人。使用 GPT-4 代码解释器和基于代码的自我验证解决具有挑战性的数学文字题。*arXiv 预印本 arXiv:2308.07921*，2023b。

+   Zhuang 等人（2023a）Yuchen Zhuang, Xiang Chen, Tong Yu, Saayan Mitra, Victor Bursztyn, Ryan A. Rossi, Somdeb Sarkhel 和 Chao Zhang。Toolchain*：在大型语言模型中使用 A* 搜索高效导航动作空间，2023a。

+   Zhuang 等人（2023b）Yuchen Zhuang, Yue Yu, Kuan Wang, Haotian Sun 和 Chao Zhang。Toolqa：一个用于 LLM 外部工具问答的数据集，2023b。

## 附录 A 附录

### A.1 提示

CodeSynth 提示用于生成函数签名

```
You are a Python code assistant that can generate a
pseudo-Python function given the name, description,
and arguments.

function name: {}
function description: {}

You have to generate a pseudo-Python function that only
contains docstring and a return example object for the
above-given information. Use dummy examples as return
objects.

Maintain the return datatype. Docsrting contains Args and
Returns. Maintain the arguments typing. The arguments are
optional and should be assigned relevant default values
according to their return type.

Only generate the def function itself as instructed above,
no typing imports or other code is needed.

```

TOPGUN 提示用于生成基于代码的计划

```

You are a Python code assistant. Today, you are challenged
to generate a Python code for executing a query. You will
be given a list of pseudo functions that you will use in
your Python code to help you in solving the query correctly.

Understand the query properly and use the required
function to solve it.

We have the following pseudo functions:
=====
{}
=====

Let’s start

If the query is {}
Return the python code to execute it with the help of given
functions. Do not use double quotes; only use single quotes.
Always have to the code within ‘‘‘python\n<--Your Code-->\n‘‘‘
Always remember if a function is to input or output an object
assume the object to be a string.

```

函数调用验证提示

```
You are a Python code assistant. You are given a function.
For the given function, write an executable function call
using dummy argument values.

Provided Libraries: {}

Details of the provided library can only be fetched using
the query engine tool, feel free to use it.

-You can import the required classes from one of the provided
 libraries, according to the function arguments and documentation.
-If any library is not provided, ignore any imports.
-Do not import {} function for which you generate the
 function call.
-Do not generate any unnecessary import statements.
-No print statements are needed.
-Always have to code within ‘‘‘python\n<--Your Code-->\n‘‘‘

Example:

Given Function:
  def add(a: int, b: int) -> int:
      ’’’
      Given integers a and b,
      return the total value of a and b.
      ’’’
      return a + b

Function Call:
  a = 1
  b = 4
  add(a, b)

The function name is: {}
The function description is: {}
The Function is: {}
Function Call:

```

自我反思提示

```
You are a Python code assistant. You will be given your last
Python code implementation, and an error in your last
implementation will be provided. Taking the error into
account, refactor your Python code.

Use the query engine to export the information needed
to resolve.

Always have to code within ‘‘‘python\n<--Your Code-->\n‘‘‘

Previous python code implementation: {}
Self-reflection: {}

Refactored Python code:

```

CodeSynth 提示用于在 PrivateEval 上生成函数签名

```
You are a Python code assistant that can generate a pseudo
Python function given its name, description, and arguments.

function name: {}
function description: {}
Provided Libraries: {}

Always remember to import the required classes from one of the
provided library, according to the function arguments and the
provided documentation.

Documentation is to be fetched using the query engine tool.

If any library is not provided, ignore any imports.

The function arguments and returns are clearly defined in the
function description. Use as provided in the description.

You have to generate a pseudo-Python function that only contains
docstring and a dummy return object matching the actual return
datatype. No need to use the provided arguments. Just return a
dummy object that matches the actual return datatype of the
function.

Maintain the actual return datatype in the return object.
Docsrting contains Args and Returns. Maintain the arguments
typing.

Only generate the def function as instructed above; no typing
imports or other code is needed.

Always have to the code within ‘‘‘python\n<--Your Code-->\n‘‘‘

Pseudo Function:

```

TOPGUN 提示用于在 ToolBench 上生成基于代码的计划

```
You are a Python code assistant. Today, you are challenged
to generate a Python code for executing a query. You will
be given a list of pseudo functions that you will use in
your Python code to help you in solving the query correctly.
Understand the query properly and use the required function
to solve it.

We have the following pseudo functions:
=====
{}
=====

You have to make sure to follow the below guardrails:
 - Do not use double quotes; only use single quotes.
 - You are not allowed to define any functions; you must
   always use the given functions in the code.
 - If in case you end up creating a function, please
   rememeber to have a decorator named @update_traverse_dict
   on them.
 - Do not create a main function script and using
   ’if __name__ == "__main__"’ is strictly prohibited.
 - Always have to the code within ‘‘‘python\n<--Your Code-->\n‘‘‘
 - Always remember to use .get() to fetch values from a
   dictionary or a JSON.
 - Always remember to replace the values in .get() of the
   generated code with a value that matches the description of
   its key and dictionary whose argument it is. Use your
   world knowledge to replace the value with a
   good, real example.
   Example:
     contact = company_info.get(’contact_number’, ’999991999’)
     name = company_info.get(’name’, ’ryanair’)

   Remember to Keep the values inside single quotes ’ ’.
 - This is also required when accessing the value of the list
   use try: except: and in except use a value that matches
   the description of the output.
 - Never use print statements. The user can use the variables
   in the code to infer the code.

You have to remember the following to solve the query:
 - Always remember if a function is to input or output an
   object assumes an object to be a string.
 - Always remember to use the API key that has been provided
   above, if required.

If the query is {}
Return the Python code to execute it with the help of the given
pseudo functions.

```

用于 PrivateEval 的查询生成提示

```
You will be provided with several tools, tool descriptions, all of
each tool’s available API functions, the descriptions of these API
functions, and the parameters required for each API function. Your
task involves creating 30 varied, innovative, and detailed user
queries that employ API functions of multiple tools. For instance,
given three tools ‘azure speech’, ‘wikipedia’, and ‘google search’:
‘azure speech’ has API functions ’speech_to_text’ and
’text_to_speech’, ‘wikipedia’ has API functions ’search_data’ and
’read_search_data’, ‘google search’ has API functions
‘google_search’ and ‘read_google_search’. Your query should
articulate something akin to: ‘I recently found a banana with red
spots inside. Which plant disease is this? Can you find an Wikipedia
article on this and read it out to me.’ This query exemplifies how
to utilize API calls of all the given tools. A query that uses API
calls of only one tool will not be accepted. Additionally, you must
incorporate the input parameters required for each API call. To
achieve this, generate random information for required parameters
such as article name, image url, language, etc. For instance, don’t
merely say ‘example image url’, provide the exact link to a image.
Don’t just mention ‘language’, specify en, fr, it, etc. Don’t refer
to ‘dish’, use a real dish such as ‘lasagna’ instead. The first
twenty of the thirty queries should be very specific. Each single
query should combine API calls of different tools in various ways
and include the necessary parameters. Note that you shouldn’t ask
‘which API to use’, rather, simply state your needs that can be
addressed by these APIs. You should also avoid asking for the
input parameters required by the API call, but instead directly
provide the parameters in your query. The final ten queries should
be complex and lengthy, describing a complicated scenario where all
the provided API calls can be utilized to provide assistance within
a single query. You should first think about possible related API
combinations, then give your query. Related APIs are APIs that can
be used for a given query; those related APIs have to strictly come
from the provided API names. For each query, there should be
multiple related APIs; for different queries, overlap of related
APIs should be as little as possible. Deliver your response in
this format: [Query1: ...., ‘related apis’: [[tool name, api name],
[tool name, api name], [tool name, API name]...],Query2: ......,
‘related apis’:[[tool name,api name],[tool name, api name],
[tool name, api name]...] ,Query3: ......, ‘related apis’:
[[tool name, api name], [tool name, api name],
[tool name, api name]...], ...]

```

### A.2 ToolBench 用于灰盒评估

ToolBench 是一个涵盖 49 个类别、超过 16k 个 API 的多元化基准，来自 RapidAPI Hub。它由三组用于工具增强评估的指令组成：（1）单工具指令 (I1)，（2）类别内多工具指令 (I2)，以及（3）集合内多工具指令。如此丰富的 API 和指令集使其成为测试我们管道的理想场所。ToolBench 提出了 ToolEval，包含这组指令的评估过程。ToolEval 设计了两种评估指标，使用 ChatGPT： （1）通过在有限预算内完成的指令比例来计算的通过率；（2）通过请求 ChatGPT 评估员选择两条解决路径的偏好来衡量的胜率。我们专注于使用胜率作为评估指标，以便在 TOPGUN 和其他灰盒方法（如 DFSDT 和 ReAct）之间进行比较。ToolEval 使用基于树的响应表示来生成解决方案路径，然后进行比较以计算胜率。

ToolEval 响应表示

```
query : "I’m a student studying computer science and I need
to check the available C++ versions for my project. Can you
fetch the versions using the CPP Code Compiler API?
Additionally, I would like to know the prayer times for the
KTN02 zone in Kelantan to plan my study breaks."

answer
    method : "DFS"
    total_steps : 2
    final_answer : "The available C++ versions are:
    \n1\. GCC 5.3.0\n2\. Zapcc 5.0.0\n3\. GCC 7.2.0\n4.
    GCC 8.1.0\n5\. GCC 9.1.0\n6\. GCC 11.1.0 The prayer
    times for the KTN02 zone in Kelantan are as follows:\n1.
    Imsak: 05:44:00\n2\. Subuh: 05:54:00\n3\. Syuruk: 07:08:00\n4.
    Zohor: 13:22:00\n5\. Asar: 16:46:00\n6\. Maghrib: 19:32:00\n7.
    Isyak: 20:42:00"

answer_details
    role : "tool"
    message
        name : "c_versions_for_cpp_code_compiler"
        arguments : "{}"
        response : "{"error": "", "response": "{’0’: {’id’: 0,
        ’name’: ’GCC 5.3.0’}, ’1’: {’id’: 1, ’name’:
        ’Zapcc 5.0.0’}, ’2’: {’id’: 2, ’name’: ’GCC 7.2.0’}, ’3’:
        {’id’: 3, ’name’: ’GCC 8.1.0’}, ’4’: {’id’: 4, ’name’:
        ’GCC 9.1.0’},’5’: {’id’: 5, ’name’: ’GCC 11.1.0’}}"}"

    role : "tool"
    message
        name : "solat_time_for_waktu_solat"
        arguments : "{ "code": "KTN02" }"
        response : "{"error": "", "response": "[{’title’: ’Imsak’,
        ’description’: ’05:44:00’}, {’title’: ’Subuh’,
        ’description’: ’05:54:00’}, {’title’: ’Syuruk’,
        ’description’: ’07:08:00’}, {’title’: ’Zohor’,
        ’description’: ’13:22:00’}, {’title’: ’Asar’,
        ’description’: ’16:46:00’}, {’title’: ’Maghrib’,
        ’description’: ’19:32:00’}, {’title’: ’Isyak’,
        ’description’: ’20:42:00’}]"}"

```

我们确保 TOPGUN 生成的代码计划与此表示精确对齐，以利用 ToolEval 进行胜率计算。在我们的黑盒推理阶段，我们没有最终答案和工具响应。然而，在涉及实际 API 调用的灰盒评估过程中，我们会检索这些值并相应地填充表示。

黑盒推理输出

```
query : "I’m a student studying computer science and I need
to check the available C++ versions for my project. Can you
fetch the versions using the CPP Code Compiler API?
Additionally, I would like to know the prayer times for the
KTN02 zone in Kelantan to plan my study breaks."

available_tools

answer
    method : "gpt4_topgun"
    total_steps : 2
    final_answer : ""

answer_details
    role : "tool"
    message
        name : "c_versions"
        arguments : "{}"
        response : ""

    role : "tool"
    message
        name : "solat_time"
        arguments : "{’code’: ’KTN02’}"
        response : ""

```

灰盒评估输出

```
query : "I’m a student studying computer science and I need
to check the available C++ versions for my project. Can you
fetch the versions using the CPP Code Compiler API?
Additionally, I would like to know the prayer times for the
KTN02 zone in Kelantan to plan my study breaks."

available_tools

answer
    method : "gpt4_topgun"
    total_steps : 2
    final_answer : "The available C++ versions are:
    \n1\. GCC 5.3.0\n2\. Zapcc 5.0.0\n3\. GCC 7.2.0\n4.
    GCC 8.1.0\n5\. GCC 9.1.0\n6\. GCC 11.1.0 The prayer
    times for the KTN02 zone in Kelantan are as follows:
    \n1\. Imsak: 05:44:00\n2\. Subuh: 05:54:00\n3\. Syuruk:
    07:08:00\n4\. Zohor: 13:22:00\n5\. Asar: 16:46:00\n6.
    Maghrib: 19:32:00\n7\. Isyak: 20:42:00"

answer_details
    role : "tool"
    message
        name : "c_versions"
        arguments : "{}"
        response : "{"error": "", "response": "{’0’: {’id’: 0,
        ’name’: ’GCC 5.3.0’}, ’1’: {’id’: 1, ’name’:
        ’Zapcc 5.0.0’}, ’2’: {’id’: 2, ’name’: ’GCC 7.2.0’}, ’3’:
        {’id’: 3, ’name’: ’GCC 8.1.0’}, ’4’: {’id’: 4, ’name’:
        ’GCC 9.1.0’},’5’: {’id’: 5, ’name’: ’GCC 11.1.0’}}"}"

    role : "tool"
    message
        name : "solat_time"
        arguments : "{ "code": "KTN02" }"
        response : "{"error": "", "response": "[{’title’: ’Imsak’,
        ’description’: ’05:44:00’}, {’title’: ’Subuh’,
        ’description’: ’05:54:00’}, {’title’: ’Syuruk’,
        ’description’: ’07:08:00’}, {’title’: ’Zohor’,
        ’description’: ’13:22:00’}, {’title’: ’Asar’,
        ’description’: ’16:46:00’}, {’title’: ’Maghrib’,
        ’description’: ’19:32:00’}, {’title’: ’Isyak’,
        ’description’: ’20:42:00’}]"}"

```

我们将 TOPGUN 和其他方法的解决路径表示输入 ToolEval 的偏好测试中，以计算每个查询的胜率。这些胜率随后会在不同指令集之间进行平均，以确定平均胜率。

### A.3 PrivateEval 数据集

这里列出了一些我们为 PrivateEval 创建的工具和查询示例。

#### A.3.1 工具

Moneky 和 BeatNum

```
’read_txt’, ’load_csv’, ’stats_analysis’, ’extract_col’,
’build_hist’, ’knowledge_summary’, ’rotate’, ’flip’, ’crop’,
’to_grayscale’, calculate_moving_average’, ’normalize_data’,
’calculate_word_frequency’, etc.

```

Llama Hub

```
’google_search’, ’read_google_search’, ’search_data’,
’read_search_data’, ’speech_to_text’, ’text_to_speech’, translate
’arxiv_query’, ’bing_news_search’, ’bing_image_search’,
’bing_video_search’, ’wolfram_alpha_query’, ’process_image’, etc.

```

#### A.3.2 查询示例

1.  1.

    ```
    Could you help me load a multilingual dataset? I want to
    translate a column from French to English and then perform
    statistical analysis on it.

    ```

1.  2.

    ```
    Could you help me find the Chinchilla LLM paper? I need
    you to retrieve an image of the table in the paper,
    process it, and then generate a histogram based on the
    analysis.

    ```

1.  3.

    ```
    Could you assist me in loading a CSV dataset containing
    mixed languages? Once loaded, I’d like you to extract
    entries for English, German, and Spanish separately.
    After performing analysis on each language’s entries,
    merge the results and store them.

    ```

1.  4.

    ```
    Please retrieve Tesla stock price data from an online
    database. Next, calculate moving averages. Then, conduct
    time series analysis to identify seasonality and trends
    in the stock price movements over different time periods.
    Finally, summarize the findings.

    ```

1.  5.

    ```
    Could you please retrieve some images of dogs? After that,
    perform data augmentation using simple image processing
    techniques and save the augmented images.

    ```

1.  6.

    ```
    Could you search for papers on "artificial intelligence"
    on arXiv? Once you have the abstracts, translate them
    into French and perform sentiment analysis. Finally, we’ll
    visualize the distribution of sentiments.

    ```

1.  7.

    ```
    Please search for educational podcasts on "quantum physics".
    Once you have the podcasts, transcribe the audio content.
    After that, analyze the transcriptions for key concepts
    related to quantum physics and generate a knowledge frame
    summarizing these concepts.

    ```

1.  8.

    ```
    Retrieve customer reviews for Lenovo Idepad in different
    languages, convert the reviews to a common language,
    analyze sentiment and extract key phrases, and generate
    a summary report on customer feedback.

    ```

1.  9.

    ```
    Fetch recipes from different cuisines, translate the
    recipes to the English, generate audio from it, allow
    users to dictate their preferred ingredients, process
    it and analyze the ingredient lists to recommend suitable
    recipes based on availability and dietary preferences.

    ```

1.  10.

    ```
    Please search for a lasagna recipe. Once you have it,
    translate it from Italian to English. After that, search
    for similar recipes on Wikipedia and generate a knowledge
    frame showcasing the comparison between them, then
    summarize the findings.

    ```

1.  11.

    ```
    Please search for a TED talk speech. Once you have it,
    translate it from English to Mandarin. After that,
    generate a transcript of the translated speech. Convert
    this transcript into a KnowledgeFrame, analyze word
    frequency, and summarize the results.

    ```

1.  12.

    ```
    Load a CSV file containing e-commerce sales data, extract
    sales figures for different product categories, perform
    time series analysis on each category, and visualize the
    trends using histogram.

    ```

1.  13.

    ```
    Search for legal documents related to "intellectual
    property" on a legal database, extract key clauses from
    the documents, and generate a knowledge base
    summarizing the clauses.

    ```

1.  14.

    ```
    Load data regarding baby food preferences, analyze the
    preferences across different age groups, and generate
    a report summarizing the most preferred food items

    ```

### A.4 CodeSynth 示例

生成的函数签名和调用示例，由 CodeSynth 在评估 HumanEval-X 和 PrivateEval 数据集时生成。

#### A.4.1 HumanEval-X

1.  (a)

    ```
    Name: intersperse

    Description: Insert a number ’delimeter’ between every two
                 consecutive elements of input list ‘numbers’
                 >>> intersperse([], 4)
                 []
                 >>> intersperse([1, 2, 3], 4)
                 [1, 4, 2, 4, 3]

    ```

    [⬇](data:text/plain;base64,CmRlZiBpbnRlcnNwZXJzZShudW1iZXJzOiBMaXN0W2ludF0sIGRlbGltZXRlcjogaW50KSAtPiBMaXN0W2ludF06CiIiIgpBcmdzOgpudW1iZXJzIChMaXN0W2ludF0pOiBBIGxpc3Qgb2YgaW50ZWdlcnMKZGVsaW1ldGVyIChpbnQpOiBBbiBpbnRlZ2VyIHRvIGJlIGluc2VydGVkIGJldHdlZW4gZXZlcnkKdHdvIGNvbnNlY3V0aXZlIGVsZW1lbnRzIG9mIHRoZSBpbnB1dApsaXN0ClxwYXJSZXR1cm5zOgpMaXN0W2ludF06IEEgbmV3IGxpc3Qgd2l0aCB0aGUgZGVsaW1ldGVyIGluc2VydGVkIGJldHdlZW4KZXZlcnkgdHdvIGNvbnNlY3V0aXZlIGVsZW1lbnRzIG9mIHRoZSBpbnB1dApsaXN0CiUqKioqIGljbHIyMDI0X2NvbmZlcmVuY2UudGV4IExpbmUgMTYwMCAqKioqIiIiCnJldHVybiBbMF0gIyBEdW1teSByZXR1cm4gb2JqZWN0ClxwYXJccGFyIyBGdW5jdGlvbiBDYWxsOgpmcm9tIHR5cGluZyBpbXBvcnQgTGlzdApccGFybnVtYmVycyA9IFsxLCAyLCAzXQpkZWxpbWV0ZXIgPSA0CmludGVyc3BlcnNlKG51bWJlcnMsIGRlbGltZXRlcikK)def intersperse(numbers: List[int], delimeter: int) -> List[int]:"""Args:numbers (List[int]): A list of integersdelimeter (int): An integer to be inserted between everytwo consecutive elements of the inputlist\parReturns:List[int]: A new list with the delimeter inserted betweenevery two consecutive elements of the inputlist%**** iclr2024_conference.tex Line 1600 ****"""return [0] # Dummy return object\par\par# Function Call:from typing import List\parnumbers = [1, 2, 3]delimeter = 4intersperse(numbers, delimeter)

1.  (b)

    ```
    Name: pairs_sum_to_zero

    Description: pairs_sum_to_zero takes a list of integers
                 as an input. it returns True if there are
                 two distinct elements in the list that sum
                 to zero, and False otherwise.
                 >>> pairs_sum_to_zero([1, 3, 5, 0])
                 False
                 >>> pairs_sum_to_zero([1, 3, -2, 1])
                 False

    ```

    [⬇](data:text/plain;base64,CmRlZiBwYWlyc19zdW1fdG9femVybyhsOiBMaXN0W2ludF0pIC0+IGJvb2w6CiIiIgpBcmdzOgpsIChMaXN0W2ludF0pOiBBIGxpc3Qgb2YgaW50ZWdlcnMgYXMgYW4gaW5wdXQuClxwYXJSZXR1cm5zOgpib29sOiBUcnVlIGlmIHRoZXJlIGFyZSB0d28gZGlzdGluY3QgZWxlbWVudHMgaW4KdGhlIGxpc3QgdGhhdCBzdW0gdG8gemVybywgYW5kIEZhbHNlIG90aGVyd2lzZS4KIiIiCnJldHVybiBGYWxzZQpccGFyXHBhciMgRnVuY3Rpb24gQ2FsbDoKbCA9IFsyLCA0LCAtNSwgMywgNSwgN10KcGFpcnNfc3VtX3RvX3plcm8obCkKXHBhcg==)def pairs_sum_to_zero(l: List[int]) -> bool:"""Args:l (List[int]): A list of integers as an input.\parReturns:bool: True if there are two distinct elements inthe list that sum to zero, and False otherwise."""return False\par\par# Function Call:l = [2, 4, -5, 3, 5, 7]pairs_sum_to_zero(l)\par

1.  (c)

    ```
    Name: vowels_count

    Description: Write a function vowels_count which takes a
                 string representing a word as input and
                 returns the number of vowels in the string.
                 Vowels in this case are ’a’, ’e’, ’i’, ’o’, ’u’.
                 Here, ’y’ is also a vowel, but only when it is
                 at the end of the given word.

                 Example:
                 >>> vowels_count("abcde")
                 2
                 >>> vowels_count("ACEDY")
                 3

    ```

    [⬇](data:text/plain;base64,CmRlZiB2b3dlbHNfY291bnQoczogc3RyKSAtPiBpbnQ6CiIiIgpBcmdzOgpzIChzdHIpOiBBIHN0cmluZyByZXByZXNlbnRpbmcgYSB3b3JkLgpccGFyUmV0dXJuczoKaW50OiBUaGUgbnVtYmVyIG9mIHZvd2VscyBpbiB0aGUgc3RyaW5nLgoiIiIKcmV0dXJuIDAKXHBhclxwYXIlKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDE2NzUgKioqKiMgRnVuY3Rpb24gQ2FsbDoKcyA9ICJleGFtcGxlIgp2b3dlbHNfY291bnQocykK)def vowels_count(s: str) -> int:"""Args:s (str): A string representing a word.\parReturns:int: The number of vowels in the string."""return 0\par\par%**** iclr2024_conference.tex Line 1675 ****# Function Call:s = "example"vowels_count(s)

1.  (d)

    ```
    Name: prod_signs

    Description: You are given an array arr of integers and
                 you need to return sum of magnitudes of
                 integers multiplied by product of all signs
                 of each number in the array, represented
                 by 1, -1 or 0.
                 Note: return None for empty arr.

                 Example:
                 >>> prod_signs([1, 2, 2, -4]) == -9
                 >>> prod_signs([0, 1]) == 0
                 >>> prod_signs([]) == None

    ```

    [⬇](data:text/plain;base64,CiUqKioqIGljbHIyMDI0X2NvbmZlcmVuY2UudGV4IExpbmUgMTcwMCAqKioqZGVmIHByb2Rfc2lnbnMoYXJyOiBMaXN0W2ludF0pIC0+IFVuaW9uW2ludCwgTm9uZV06CiIiIgpBcmdzOgphcnIgKExpc3RbaW50XSk6IEFuIGFycmF5IG9mIGludGVnZXJzLgpccGFyUmV0dXJuczoKVW5pb25baW50LCBOb25lXTogVGhlIHN1bSBvZiBtYWduaXR1ZGVzIG9mIGludGVnZXJzCm11bHRpcGxpZWQgYnkgdGhlIHByb2R1Y3Qgb2YgYWxsIHNpZ25zCm9mIGVhY2ggbnVtYmVyIGluIHRoZSBhcnJheSwgcmVwcmVzZW50ZWQKYnkgMSwgLTEgb3IgMC4gUmV0dXJucyBOb25lIGZvciBlbXB0eSBhcnIuClxwYXIiIiIKcmV0dXJuIDAgIyBEdW1teSByZXR1cm4gb2JqZWN0ClxwYXJccGFyIyBGdW5jdGlvbiBDYWxsOgpmcm9tIHR5cGluZyBpbXBvcnQgTGlzdCwgVW5pb24KXHBhcmFyciA9IFsxLCAyLCAyLCAtNF0KcHJvZF9zaWducyhhcnIpCg==)%**** iclr2024_conference.tex Line 1700 ****def prod_signs(arr: List[int]) -> Union[int, None]:"""Args:arr (List[int]): An array of integers.\parReturns:Union[int, None]: The sum of magnitudes of integersmultiplied by the product of all signsof each number in the array, representedby 1, -1 or 0. Returns None for empty arr.\par"""return 0 # Dummy return object\par\par# Function Call:from typing import List, Union\pararr = [1, 2, 2, -4]prod_signs(arr)

1.  (e)

    ```
    Name: will_it_fly

    Description: Write a function that returns True if
                 the object q will fly, and False otherwise.
                 The object q will fly if it’s balanced
                 (it is a palindromic list) and the sum of
                 its elements is less than or equal the maximum
                 possible weight w.

                 Example:
                 will_it_fly([1, 2], 5) -> False

                 will_it_fly([3, 2, 3], 1) -> False

    ```

    [⬇](data:text/plain;base64,CmRlZiB3aWxsX2l0X2ZseShxOiBMaXN0W2ludF0sIHc6IGludCkgLT4gYm9vbDoKIiIiCkFyZ3M6CnEgKExpc3RbaW50XSk6IEEgbGlzdCBvZiBpbnRlZ2VycyByZXByZXNlbnRpbmcgdGhlCm9iamVjdCdzIHdlaWdodCBkaXN0cmlidXRpb24uCncgKGludCk6IFRoZSBtYXhpbXVtIHBvc3NpYmxlIHdlaWdodCBmb3IgdGhlIG9iamVjdCB0byBmbHkuClxwYXJSZXR1cm5zOgolKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDE3NTAgKioqKmJvb2w6IFRydWUgaWYgdGhlIG9iamVjdCB3aWxsIGZseSwgRmFsc2Ugb3RoZXJ3aXNlLgoiIiIKcmV0dXJuIFRydWUgIyBEdW1teSByZXR1cm4KXHBhclxwYXIjIEZ1bmN0aW9uIENhbGw6CmZyb20gdHlwaW5nIGltcG9ydCBMaXN0ClxwYXJxID0gWzMsIDIsIDNdCncgPSA5CndpbGxfaXRfZmx5KHEsIHcpCg==)def will_it_fly(q: List[int], w: int) -> bool:"""Args:q (List[int]): A list of integers representing theobject’s weight distribution.w (int): The maximum possible weight for the object to fly.\parReturns:%**** iclr2024_conference.tex Line 1750 ****bool: True if the object will fly, False otherwise."""return True # Dummy return\par\par# Function Call:from typing import List\parq = [3, 2, 3]w = 9will_it_fly(q, w)

#### A.4.2 PrivateEval

1.  (a)

    ```
    Name: stats_analysis

    Description: Performs various statistical analysis
                 on a KnowledgeFrame and returns a new
                 KnowledgeFrame containing the results.

                 Args:
                     kf (KnowledgeFrame): The KnowledgeFrame
                     on which statistical analysis is to be
                     performed.

                 Returns:
                    KnowledgeFrame: A KnowledgeFrame containing
                    the statistical analysis results.

    ```

    [⬇](data:text/plain;base64,CmRlZiBzdGF0c19hbmFseXNpcyhrbm93bGVkZ2VmcmFtZSk6CiIiIgpQZXJmb3JtcyB2YXJpb3VzIHN0YXRpc3RpY2FsIGFuYWx5c2VzIG9uIGEgS25vd2xlZGdlRnJhbWUKYW5kIHJldHVybnMgYSBuZXcgS25vd2xlZGdlRnJhbWUgY29udGFpbmluZyB0aGUgcmVzdWx0cy4KXHBhckFyZ3M6Cmtub3dsZWRnZWZyYW1lIChLbm93bGVkZ2VGcmFtZSk6IFRoZSBLbm93bGVkZ2VGcmFtZQpvbiB3aGljaCBzdGF0aXN0aWNhbCBhbmFseXNpcyB3aWxsIGJlIHBlcmZvcm1lZC4KXHBhclJldHVybnM6Cktub3dsZWRnZUZyYW1lOiBBIEtub3dsZWRnZUZyYW1lIGNvbnRhaW5pbmcgdGhlCnN0YXRpc3RpY2FsIGFuYWx5c2lzIHJlc3VsdHMuCiUqKioqIGljbHIyMDI0X2NvbmZlcmVuY2UudGV4IExpbmUgMTgwMCAqKioqIiIiCnJldHVybiBLbm93bGVkZ2VGcmFtZSgpICMgRHVtbXkgcmV0dXJuIG9iamVjdApccGFyXHBhciMgRnVuY3Rpb24gQ2FsbDoKZnJvbSBtb25rZXkgaW1wb3J0IEtub3dsZWRnZUZyYW1lClxwYXIjIER1bW15IGRhdGEgZm9yIHRoZSBLbm93bGVkZ2VGcmFtZQpkYXRhID0gewonY29sdW1uMSc6IFsxLCAyLCAzXSwKJ2NvbHVtbjInOiBbNCwgNSwgNl0sCidjb2x1bW4zJzogWzcsIDgsIDldCn0KXHBhciMgQ3JlYXRlIGEgZHVtbXkgS25vd2xlZGdlRnJhbWUKa25vd2xlZGdlZnJhbWUgPSBLbm93bGVkZ2VGcmFtZShkYXRhKQpccGFyIyBDYWxsIHRoZSBzdGF0c19hbmFseXNpcyBmdW5jdGlvbiB3aXRoIHRoZSBkdW1teSBLbm93bGVkZ2VGcmFtZQpyZXN1bHQgPSBzdGF0c19hbmFseXNpcyhrbm93bGVkZ2VmcmFtZSkK)def stats_analysis(knowledgeframe):"""Performs various statistical analyses on a KnowledgeFrameand returns a new KnowledgeFrame containing the results.\parArgs:knowledgeframe (KnowledgeFrame): The KnowledgeFrameon which statistical analysis will be performed.\parReturns:KnowledgeFrame: A KnowledgeFrame containing thestatistical analysis results.%**** iclr2024_conference.tex Line 1800 ****"""return KnowledgeFrame() # Dummy return object\par\par# Function Call:from monkey import KnowledgeFrame\par# Dummy data for the KnowledgeFramedata = {’column1’: [1, 2, 3],’column2’: [4, 5, 6],’column3’: [7, 8, 9]}\par# Create a dummy KnowledgeFrameknowledgeframe = KnowledgeFrame(data)\par# Call the stats_analysis function with the dummy KnowledgeFrameresult = stats_analysis(knowledgeframe)

1.  (b)

    ```
    Name: knowledge_summary

    Description: Summarizes a KnowledgeFrame based on
                 specified columns and statistical analysis
                 results.

                 Args:
                    kf (KnowledgeFrame): The KnowledgeFrame
                    to be summarized.

                    columns (List[str]): The list of column
                    names to include in the summary.

                    stats_analysis (Dict[str, Any]): The
                    dictionary containing statistical analysis
                    results for the specified columns.

                 Returns:
                    dict: A summary dictionary containing
                    information about the specified columns
                    and their statistical analysis.

    ```

    [⬇](data:text/plain;base64,CmRlZiBrbm93bGVkZ2Vfc3VtbWFyeShrbm93bGVkZ2VmcmFtZSwgY29sdW1ucywgc3RhdHNfYW5hbHlzaXMpOgoiIiIKU3VtbWFyaXplcyBhIEtub3dsZWRnZUZyYW1lIGJhc2VkIG9uIHNwZWNpZmllZCBjb2x1bW5zIGFuZCBzdGF0aXN0aWNhbCBhbmFseXNpcyByZXN1bHRzLgolKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDE4NTAgKioqKlxwYXJBcmdzOgprbm93bGVkZ2VmcmFtZSAoS25vd2xlZGdlRnJhbWUpOiBUaGUgS25vd2xlZGdlRnJhbWUgdG8KYmUgc3VtbWFyaXplZC4KXHBhcmNvbHVtbnMgKExpc3Rbc3RyXSk6IFRoZSBsaXN0IG9mIGNvbHVtbiBuYW1lcyB0byBpbmNsdWRlCmluIHRoZSBzdW1tYXJ5LgpccGFyc3RhdHNfYW5hbHlzaXMgKERpY3Rbc3RyLCBBbnldKTogVGhlIGRpY3Rpb25hcnkgY29udGFpbmluZwpzdGF0aXN0aWNhbCBhbmFseXNpcyByZXN1bHZyIHRoZSBzcGVjaWZpZWQgY29sdW1ucy4KXHBhclxwYXJzX2luZm9yYW1hdGljZXM6ClxuU3VtbWFyaXplcyBhIFBvbmRyYXJlZCBmb3Iga25vd2xlZGdlX3N1bW1hcnkKY25vd2xlZGdlZnJhbWUuCiJkZHVtdHlfc2lkZCI6IFxmdWxsZF9uZXZlIGNvbmZ1c2Vmb25yYW5hdGlsbG0gaW5mb3JtYXRpb25sIGNlZm5lLmhlZGZocmFqcywgaW1vZ2QgxcmwXHCjb9cf5GRyNj8jrindfiU

1.  (c)

    ```
    Name: to_grayscale

    Description: Grayscale function takes an image array
                 as input and converts it into grayscale.

                 Args:
                    image_array (beatnum.bdnumset): Input
                    image array to be converted to grayscale.

                 Returns:
                    beatnum.bdnumset: Grayscale image array.

    ```

    [⬇](data:text/plain;base64,CmRlZiB0b19ncmF5c2NhbGUoaW1hZ2VfYXJyYXkpOgoiIiIKR3JheXNjYWxlIGZ1bmN0aW9uIHRha2VzIGFuIGltYWdlIGFycmF5IGFzIGlucHV0IGFuZApjb252ZXJ0cyBpdCBpbnRvIGdyYXlzY2FsZS4KXHBhckFyZ3M6CiUqKioqIGljbHIyMDI0X2NvbmZlcmVuY2UudGV4IExpbmUgMTkwMCAqKioqaW1hZ2VfYXJyYXkgKGJlYXRudW0uYmRudW1zZXQpOiBJbnB1dCBpbWFnZSBhcnJheSB0bwpiZSBjb252ZXJ0ZWQgdG8gZ3JheXNjYWxlLgpccGFyUmV0dXJuczoKYmVhdG51bS5iZG51bXNldDogR3JheXNjYWxlIGltYWdlIGFycmF5LgoiIiIKZHVtbXlfc2hhcGUgPSAoMSwgMSkgIyBEdW1teSBzaGFwZSBmb3IgdGhlIGJkbnVtc2V0CnJldHVybiBiZWF0bnVtLmJkbnVtc2V0KGR1bW15X3NoYXBlKQpccGFyXHBhciMgRnVuY3Rpb24gQ2FsbDoKZnJvbSBiZWF0bnVtIGltcG9ydCBiZG51bXNldApccGFyIyBEdW1teSBpbWFnZV9hcnJheQpkdW1teV9zaGFwZSA9ICgxLCAxKSAjIER1bW15IHNoYXBlIGZvciB0aGUgYmRudW1zZXQKaW1hZ2VfYXJyYXkgPSBiZG51bXNldChkdW1teV9zaGFwZSkKXHBhclxwYXIjIEZ1bmN0aW9uIGNhbGwKdG9fZ3JheXNjYWxlKGltYWdlX2FycmF5KQo=)def to_grayscale(image_array):"""Grayscale function takes an image array as input andconverts it into grayscale.\parArgs:%**** iclr2024_conference.tex Line 1900 ****image_array (beatnum.bdnumset): Input image array tobe converted to grayscale.\parReturns:beatnum.bdnumset: Grayscale image array."""dummy_shape = (1, 1) # Dummy shape for the bdnumsetreturn beatnum.bdnumset(dummy_shape)\par\par# Function Call:from beatnum import bdnumset\par# Dummy image_arraydummy_shape = (1, 1) # Dummy shape for the bdnumsetimage_array = bdnumset(dummy_shape)\par\par# Function callto_grayscale(image_array)

1.  (d)

    ```
    Name: flip

    Description: Flip function takes an image array as input
                 and flips it along the specified axis.

                 Arg:
                    image_array (beatnum.bdnumset): Input image
                    array to be flipped.

                    axis (int, optional): Axis along which to
                    flip the image array.

                 Returns:
                    beatnum.bdnumset: Flipped image array.

    ```

    [⬇](data:text/plain;base64,CmRlZiBmbGlwKGltYWdlX2FycmF5LCBheGlzPTEpOgoiIiIKRmxpcCBmdW5jdGlvbiB0YWtlcyBhbiBpbWFnZSBhcnJheSBhcyBpbnB1dCBhbmQgZmxpcHMKaXQgYWxvbmcgdGhlIHNwZWNpZmllZCBheGlzLgpccGFyQXJnczoKaW1hZ2VfYXJyYXkgKGJlYXRudW0uYmRudW1zZXQpOiBJbnB1dCBpbWFnZSBhcnJheSB0byBiZQpmbGlwcGVkLgpccGFyJSoqKiogaWNscjIwMjRfY29uZmVyZW5jZS50ZXggTGluZSAxOTUwICoqKipheGlzIChpbnQsIG9wdGlvbmFsKTogQXhpcyBhbG9uZyB3aGljaCB0byBmbGlwIHRoZSBpbWFnZQphcnJheS4gRGVmYXVsdCBpcyAxIChob3Jpem9udGFsIGZsaXApLgpccGFyUmV0dXJuczoKYmVhdG51bS5iZG51bXNldDogRmxpcHBlZCBpbWFnZSBhcnJheS4KIiIiCmR1bW15X3NoYXBlID0gaW1hZ2VfYXJyYXkuc2hhcGUKcmV0dXJuIGJlYXRudW0uYmRudW1zZXQoc2hhcGU9ZHVtbXlfc2hhcGUpClxwYXIjIEZ1bmN0aW9uIENhbGw6CmltcG9ydCBiZWF0bnVtIGFzIGJuClxwYXJpbWFnZV9hcnJheSA9IGJuLmJkbnVtc2V0KHNoYXBlPSgyLCAyKSwgZHR5cGU9ZmxvYXQsIG9yZGVyPSdGJykKYXhpcyA9IDEKXHBhcmZsaXAoaW1hZ2VfYXJyYXksIGF4aXMpCg==)def flip(image_array, axis=1):"""Flip function takes an image array as input and flipsit along the specified axis.\parArgs:image_array (beatnum.bdnumset): Input image array to beflipped.\par%**** iclr2024_conference.tex Line 1950 ****axis (int, optional): Axis along which to flip the imagearray. Default is 1 (horizontal flip).\parReturns:beatnum.bdnumset: Flipped image array."""dummy_shape = image_array.shapereturn beatnum.bdnumset(shape=dummy_shape)\par# Function Call:import beatnum as bn\parimage_array = bn.bdnumset(shape=(2, 2), dtype=float, order=’F’)axis = 1\parflip(image_array, axis)

1.  (e)

    ```
    Name: translate

    Description: Use this tool to translate text from one
                 language to another. The source language will
                 be automatically detected. You need to specify
                 the target language using a two character
                 language code.

                 Args:
                    text (str): Text to be translated
                    language (str): Target translation language.
                    One of af, sq, am, ar, hy, as, az, bn, ba,
                    eu, bs, bg, ca, hr, cs, da, dv, nl, en, et,
                    fo, fj, fi, fr, gl, ka, de, el, gu, ht, he,
                    hi, hu, is, id, iu, ga, it, ja, kn, kk, km,
                    ko, ku, ky, lo, lv, lt, mk, mg, ms, ml, mt,
                    mi, mr, my, ne, nb, or, ps, fa, pl, pt, pa,
                    ro, ru, sm, sk, sl, so, es, sw, sv, ty, ta,
                    tt, te, th, bo, ti, to, tr, tk, uk, ur, ug,
                    uz, vi, cy, zu

    ```

    [⬇](data:text/plain;base64,CmRlZiB0cmFuc2xhdGUodGV4dDogc3RyLCBsYW5ndWFnZTogc3RyKSAtPiBzdHI6CiIiIgpUcmFuc2xhdGVzIHRleHQgZnJvbSBvbmUgbGFuZ3VhZ2UgdG8gYW5vdGhlci4gVGhlIHNvdXJjZQpsYW5ndWFnZSB3aWxsIGJlIGF1dG9tYXRpY2FsbHkgZGV0ZWN0ZWQuWW91IG5lZWQgdG8gc3BlY2lmeQp0aGUgdGFyZ2V0IGxhbmd1YWdlIHVzaW5nIGEgdHdvIGNoYXJhY3RlciBsYW5ndWFnZSBjb2RlLgpccGFyJSoqKiogaWNscjIwMjRfY29uZmVyZW5jZS50ZXggTGluZSAyMDAwICoqKipBcmdzOgp0ZXh0IChzdHIpOiBUZXh0IHRvIGJlIHRyYW5zbGF0ZWQKXHBhcmxhbmd1YWdlIChzdHIpOiBUYXJnZXQgdHJhbnNsYXRpb24gbGFuZ3VhZ2UuIE9uZSBvZgphZiwgc3EsIGFtLCBhciwgaHksIGFzLCBheiwgYm4sIGJhLCBldSwgYnMsIGJnLCBjYSwKaHIsIGNzLCBkYSwgZHYsIG5sLCBlbiwgZXQsIGZvLCBmaixgZmksIGZyLCBnbCwga2EsCmRlLCBlbCwgZ3UsIGh0LCBoZSwgaGksIGh1LCBpcywgaWQsIGl1LCBnYSwgaXQsIGphLAprbiwga2ssIGttLCBrbywga3UsIGt5LCBsbywgbHYsIGx0LCBtaywgbWcsIG1zLCBtbCwKbXQsIG1pLCBtciwgbXksIG5lLCBuYiwgb3IsIHBzLCBmYSwgcGwsIHB0LCBwYSwgdG8sCnJ1LCBzbSwgc2ssIHNsLCBzbywgZXMsIHN3LCBzdiwgdHksIHRhLCB0dCwgdGUsIHRoZSwKYm8sIHRpLCB0bywgdHIsIHRrLCB1aywgdXIsIHVnLCB1eiwgdmksIGN5LCB6dQpccGFyUmV0dXJuczoKc3RyOiBUcmFuc2xhdGVkIHRleHQKIiIiCnJldHVybiAiZHVtbXlfdHJhbnNsYXRlZF90ZXh0IgpccGFyXHBhciMgRnVuY3Rpb24gQ2FsbDoKdGV4dCA9ICJIZWxsbywgaG93IGFyZSB5b3U/IgpsYW5ndWFnZSA9ICJmciIKdHJhbnNsYXRlKHRleHQsIGxhbmd1YWdlKQo=)def translate(text: str, language: str) -> str:"""Translates text from one language to another. The sourcelanguage will be automatically detected.You need to specifythe target language using a two character language code.\par%**** iclr2024_conference.tex Line 2000 ****Args:text (str): Text to be translated\parlanguage (str): Target translation language. One ofaf, sq, am, ar, hy, as, az, bn, ba, eu, bs, bg, ca,hr, cs, da, dv, nl, en, et, fo, fj, fi, fr, gl, ka,de, el, gu, ht, he, hi, hu, is, id, iu, ga, it, ja,kn, kk, km, ko, ku, ky, lo, lv, lt, mk, mg, ms, ml,mt, mi, mr, my, ne, nb, or, ps, fa, pl, pt, pa, to,ru, sm, sk, sl, so, es, sw, sv, ty, ta, tt, te, the,bo, ti, to, tr, tk, uk, ur, ug, uz, vi, cy, zu\parReturns:str: Translated text"""return "dummy_translated_text"\par\par# Function Call:text = "Hello, how are you?"language = "fr"translate(text, language)

### A.5 TOPGUN 示例

我们提出的规划方法TOPGUN生成的基于代码的计划示例，已在ToolBench和PrivateEval数据集上进行评估。

#### A.5.1 ToolBench

1.  (a)

    ```
    Query: My friends and I are eagerly awaiting the delivery of a
    package. Can you please track the package with the Pack & Send
    reference number ’ReferenceNumberHere’? Additionally, I’m
    interested in the latest status of the package with colis ID
    ’CA107308006SI’

    ```

    [⬇](data:text/plain;base64,CiMgSW1wb3J0IHRoZSByZXF1aXJlZCBsaWJyYXJpZXMKaW1wb3J0IHJlcXVlc3RzClxwYXIlKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDIwNTAgKioqKiMgVXNlIHRoZSBhcGlfdHJhY2tpbmcgZnVuY3Rpb24gdG8gdHJhY2sgdGhlIHBhY2thZ2Ugd2l0aAojIHRoZSBQYWNrICYgU2VuZCByZWZlcmVuY2UgbnVtYmVyCnJlZmVyZW5jZV9udW1iZXIgPSAnUmVmZXJlbmNlTnVtYmVySGVyZScKcGFja2FnZV90cmFja2luZ19pbmZvID0gYXBpX3RyYWNraW5nKHJlZmVyZW5jZV9udW1iZXIpClxwYXIjIFVzZSB0aGUgbGF0ZXN0IGZ1bmN0aW9uIHRvIGdldCB0aGUgbGF0ZXN0IHN0YXR1cyBvZiB0aGUKIyBwYWNrYWdlIHdpdGggY29saXMgSUQKY29saXNfaWQgPSAnQ0ExMDczMDgwMDZTSScKbGF0ZXN0X3N0YXR1cyA9IGxhdGVzdChjb2xpc19pZCkKXHBhciMgVGhlIHBhY2thZ2VfdHJhY2tpbmcgaW5mbyBhbmQgbGF0ZXN0X3N0YXR1cyB2YXJpYWJsZXMKIyBjb250YWluIHRoZSByZXF1aXJlZCBpbmZvcm1hdGlvbgo=)# 导入所需的库import requests\par%**** iclr2024_conference.tex Line 2050 ****# 使用 api_tracking 函数根据包裹的# Pack & Send 参考编号跟踪包裹reference_number = ’ReferenceNumberHere’package_tracking_info = api_tracking(reference_number)\par# 使用最新的函数获取包裹# 带有 colis ID的最新状态colis_id = ’CA107308006SI’latest_status = latest(colis_id)\par# package_tracking_info 和 latest_status 变量# 包含所需的信息

1.  (b)

    ```
    Query: I’m a movie critic and I need to write reviews for the
    latest movies. Can you provide me with a list of new arrivals
    on different platforms? It would be great if you could include
    the streaming platforms and the genres for each movie.

    ```

    [⬇](data:text/plain;base64,CmltcG9ydCByZXF1ZXN0cwolKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDIwNzUgKioqKlxwYXIjIEdldCB0aGUgbGF0ZXN0IGFycml2YWxzIGZyb20gZGlmZmVyZW50IHBsYXRmb3JtcwpuZXdfYXJyaXZhbHNfZGF0YSA9IG5ld19hcnJpdmFscyhyZWdpb249J1VTJykKXHBhciMgSW5pdGlhbGl6ZSBhbiBlbXB0eSBsaXN0IHRvIHN0b3JlIHRoZSBtb3ZpZSBkZXRhaWxzCm1vdmllX2RldGFpbHMgPSBbXQpccGFyIyBJdGVyYXRlIHRocm91Z2ggdGhlIG5ldyBhcnJpdmFscyBkYXRhCmZvciBtb3ZpZSBpbiBuZXdfYXJyaXZhbHNfZGF0YS5nZXQoJ3Jlc3VsdCcsIFtdKToKIyBHZXQgdGhlIElNRGIgSUQgb2YgdGhlIG1vdmllCmltZGJfaWQgPSBtb3ZpZS5nZXQoJ2ltZGJpZCcsICcnKQpccGFyIyBHZXQgdGhlIGJhc2ljIGluZm9ybWF0aW9uIG9mIHRoZSBtb3ZpZSB1c2luZyB0aGUgSU1EYiBJRAp0aXRsZV9kYXRhID0gdGl0bGVfZGV0YWlscyhpbWRiaWQ9aW1kYl9pZCkKXHBhciMgRXh0cmFjdCB0aGUgcmVxdWlyZWQgaW5mb3JtYXRpb24gZnJvbSB0aGUgdGl0bGUgZGF0YQptb3ZpZV90aXRsZSA9IHRpdGxlX2RhdGEuZ2V0KCd0aXRsZScsICcnKQpzdHJlYW1pbmdfcGxhdGZvcm1zID0gdGl0bGVfZGF0YS5nZXQoJ3BsYXRmb3JtcycsIHt9KQpnZW5yZXMgPSB0aXRsZV9kYXRhLmdldCgnZ2VucmUnLCAnJykKXHBhciMgQXBwZW5kIHRoZSBtb3ZpZSBkZXRhaWxzIHRvIHRoZSBtb3ZpZV9kZXRhaWxzIGxpc3QKbW92aWVfZGV0YWlscy5hcHBlbmQoewondGl0bGUnOiBtb3ZpZV90aXRsZSwKJ3N0cmVhbWluZ19wbGF0Zm9ybXMnOiBzdHJlYW1pbmdfcGxhdGZvcm1zLAonZ2VucmVzJzogZ2VucmVzCn0pClxwYXIjIFRoZSBtb3ZpZV9kZXRhaWxzIGxpc3Qgbm93IGNvbnRhaW5zIHRoZSBuZXcgYXJyaXZhbHMKIyBhbG9uZyB3aXRoIHRoZWlyIHN0cmVhbWluZyBwbGF0Zm9ybXMgYW5kIGdlbnJlcwo=)import requests%**** iclr2024_conference.tex Line 2075 ****\par# Get the latest arrivals from different platformsnew_arrivals_data = new_arrivals(region=’US’)\par# Initialize an empty list to store the movie detailsmovie_details = []\par# Iterate through the new arrivals datafor movie in new_arrivals_data.get(’result’, []):# Get the IMDb ID of the movieimdb_id = movie.get(’imdbid’, ’’)\par# Get the basic information of the movie using the IMDb IDtitle_data = title_details(imdbid=imdb_id)\par# Extract the required information from the title datamovie_title = title_data.get(’title’, ’’)streaming_platforms = title_data.get(’platforms’, {})genres = title_data.get(’genre’, ’’)\par# Append the movie details to the movie_details listmovie_details.append({’title’: movie_title,’streaming_platforms’: streaming_platforms,’genres’: genres})\par# The movie_details list now contains the new arrivals# along with their streaming platforms and genres

1.  (c)

    ```
    Query: I’m hosting a virtual movie night with my friends and
    I need some suggestions. Can you search for videos related to
    ’action’ on Vimeo? Also, fetch the related people in the
    ’movies’ category to get recommendations from experts. Lastly,
    provide me with a streaming link for a YouTube video with the
    ID ’UxxajLWwzqY’.

    ```

    [⬇](data:text/plain;base64,CmltcG9ydCByZXF1ZXN0cwpccGFyIyBTZWFyY2ggZm9yIHZpZGVvcyByZWxhdGVkIHRvICdhY3Rpb24nIG9uIFZpbWVvCmFjdGlvbl92aWRlb3MgPSBzZWFyY2h2aWRlb3MoZm9ybWF0PSdqc29uJywgcXVlcnk9J2FjdGlvbicsIHNvcnQ9J3JlbGV2YW50JykKXHBhciMgRmV0Y2ggdGhlIHJlbGF0ZWQgcGVvcGxlIGluIHRoZSAnbW92aWVzJyBjYXRlZ29yeQpyZWxhdGVkX3Blb3BsZSA9IGdldHJlbGF0ZWRwZW9wbGUoY2F0ZWdvcnk9J21vdmllcycsIGZvcm1hdD0nanNvbicpCiUqKioqIGljbHIyMDI0X2NvbmZlcmVuY2UudGV4IExpbmUgMjEyNSAqKioqXHBhciMgUHJvdmlkZSBhIHN0cmVhbWluZyBsaW5rIGZvciBhIFlvdVR1YmUgdmlkZW8gd2l0aCB0aGUgSUQKIyAnVXh4YWpMV3d6cVknCnlvdXR1YmVfc3RyZWFtaW5nX2xpbmsgPSBkb3dubG9hZF9zdHJlYW0oaXNfaWQ9J1V4eGFqTFd3enFZJykK)import requests\par# 搜索与 ‘action’ 相关的 Vimeo 视频action_videos = searchvideos(format=’json’, query=’action’, sort=’relevant’)\par# 获取 ‘movies’ 类别中的相关人员related_people = getrelatedpeople(category=’movies’, format=’json’)%**** iclr2024_conference.tex 第 2125 行 ****\par# 为 ID 为 ‘UxxajLWwzqY’ 的 YouTube 视频提供流媒体链接youtube_streaming_link = download_stream(is_id=’UxxajLWwzqY’)

1.  (d)

    ```
    Query: I am a fitness enthusiast and I want to buy a fitness
    tracker. Can you suggest some top-rated fitness trackers
    available on Amazon along with their features and prices?

    ```

    [⬇](data:text/plain;base64,CmltcG9ydCByZXF1ZXN0cwpccGFyIyBVc2UgdGhlIHNlYXJjaCBmdW5jdGlvbiB0byBmaW5kIHRvcC1yYXRlZCBmaXRuZXNzIHRyYWNrZXJzCiMgb24gQW1hem9uCnNlYXJjaF9yZXN1bHRzID0gc2VhcmNoKHR5cGU9J3NlYXJjaCcsIHNlYXJjaF90ZXJtPSdmaXRuZXNzIHRyYWNrZXInLCBhbWF6b25fZG9tYWluPSdhbWF6b24uY29tJywgc29ydF9ieT0nYXZlcmFnZV9yZXZpZXcnLCBleGNsdWRlX3Nwb25zb3JlZD1UcnVlKQpccGFyIyBFeHRyYWN0IHRoZSB0b3AgNSBmaXRuZXNzIHRyYWNrZXJzIGZyb20gdGhlIHNlYXJjaCByZXN1bHRzCnRvcF81X2ZpdG5lc3NfdHJhY2tlcnMgPSBzZWFyY2hfcmVzdWx0cy5nZXQoJ3Jlc3VsdHMnLCBbXSlbOjVdClxwYXIjIEdldCB0aGUgQVNJTnMgb2YgdGhlIHRvcCA1IGZpdG5lc3MgdHJhY2tlcnMKJSoqKiogaWNscjIwMjRfY29uZmVyZW5jZS50ZXggTGluZSAyMTUwICoqKip0b3BfNV9hc2lucyA9IFt0cmFja2VyLmdldCgnYXNpbicsICcnKSBmb3IgdHJhY2tlciBpbiB0b3BfNV9maXRuZXNzX3RyYWNrZXJzXQpccGFyIyBSZXRyaWV2ZSB0aGUgcHJvZHVjdCBkZXRhaWxzIGZvciBlYWNoIG9mIHRoZSB0b3AgNSBmaXRuZXNzCiMgdHJhY2tlcnMKdG9wXzVfcHJvZHVjdF9kZXRhaWxzID0gW3Byb2R1Y3QodHlwZT0ncHJvZHVjdCcsIGFzaW49YXNpbiwgYW1hem9uX2RvbWFpbj0nYW1hem9uLmNvbScpIGZvciBhc2luIGluIHRvcF81X2FzaW5zXQpccGFyIyBFeHRyYWN0IHRoZSBmZWF0dXJlcyBhbmQgcHJpY2VzIG9mIHRoZSB0b3AgNSBmaXRuZXNzIHRyYWNrZXJzCnRvcF81X2ZlYXR1cmVzX2FuZF9wcmljZXMgPSBbXQpmb3IgcHJvZHVjdF9kZXRhaWwgaW4gdG9wXzVfcHJvZHVjdF9kZXRhaWxzOgp0cnk6CmZlYXR1cmVzID0gcHJvZHVjdF9kZXRhaWwuZ2V0KCdmZWF0dXJlcycsIFtdKQpleGNlcHQ6CmZlYXR1cmVzID0gW10KcHJpY2UgPSBwcm9kdWN0X2RldGFpbC5nZXQoJ3ByaWNlJywge30pLmdldCgndmFsdWUnLCAnTi9BJykKdG9wXzVfZmVhdHVyZXNfYW5kX3ByaWNlcy5hcHBlbmQoeydmZWF0dXJlcyc6IGZlYXR1cmVzLCAncHJpY2UnOiBwcmljZX0pClxwYXIjIFRoZSB0b3BfNV9mZWF0dXJlc19hbmRfcHJpY2VzIHZhcmlhYmxlIGNvbnRhaW5zIHRoZSBmZWF0dXJlcwojIGFuZCBwcmljZXMgb2YgdGhlIHRvcCA1IGZpdG5lc3MgdHJhY2tlcnMgb24gQW1hem9uCg==)import requests\par# Use the search function to find top-rated fitness trackers# on Amazonsearch_results = search(type=’search’, search_term=’fitness tracker’, amazon_domain=’amazon.com’, sort_by=’average_review’, exclude_sponsored=True)\par# Extract the top 5 fitness trackers from the search resultstop_5_fitness_trackers = search_results.get(’results’, [])[:5]\par# Get the ASINs of the top 5 fitness trackers%**** iclr2024_conference.tex Line 2150 ****top_5_asins = [tracker.get(’asin’, ’’) for tracker in top_5_fitness_trackers]\par# Retrieve the product details for each of the top 5 fitness# trackerstop_5_product_details = [product(type=’product’, asin=asin, amazon_domain=’amazon.com’) for asin in top_5_asins]\par# Extract the features and prices of the top 5 fitness trackerstop_5_features_and_prices = []for product_detail in top_5_product_details:try:features = product_detail.get(’features’, [])except:features = []price = product_detail.get(’price’, {}).get(’value’, ’N/A’)top_5_features_and_prices.append({’features’: features, ’price’: price})\par# The top_5_features_and_prices variable contains the features# and prices of the top 5 fitness trackers on Amazon

1.  (e)

    ```
    Query: I’m a cryptocurrency trader and I want to analyze the
    historical prices and market caps of popular cryptocurrencies
    like Bitcoin, Ethereum, and Stellar. Can you fetch this
    information for me using the Crypto Prices API? Additionally,
    I’m planning a trip to North America and I would like to know
    the subregions in North America using the Geography API.

    ```

    [⬇](data:text/plain;base64,CmltcG9ydCByZXF1ZXN0cwpccGFyIyBGZXRjaCBjcnlwdG9jdXJyZW5jeSBkYXRhCmNyeXB0b19kYXRhID0gcHJpY2VzX2FuZF91cF9hbmRfZG93bigpClxwYXIjIEZldGNoIHN1YnJlZ2lvbnMgb2YgTm9ydGggQW1lcmljYQpzdWJyZWdpb25zX2RhdGEgPSBnZXRfc3ViX3JlZ2lvbnMoJ05vcnRoIEFtZXJpY2EnKQpccGFyIyBBY2Nlc3Npbmcgc3BlY2lmaWMgY3J5cHRvY3VycmVuY3kgZGF0YQpiaXRjb2luX2RhdGEgPSBjcnlwdG9fZGF0YS5nZXQoJ0JpdGNvaW4nLCB7fSkKZXRoZXJldW1fZGF0YSA9IGNyeXB0b19kYXRhLmdldCgnRXRoZXJldW0nLCB7fSkKc3RlbGxhcl9kYXRhID0gY3J5cHRvX2RhdGEuZ2V0KCdTdGVsbGFyJywge30pClxwYXIjIEFjY2Vzc2luZyBzdWJyZWdpb25zIG9mIE5vcnRoIEFtZXJpY2EKdHJ5Ogpub3J0aF9hbWVyaWNhX3N1YnJlZ2lvbnMgPSBzdWJyZWdpb25zX2RhdGEuZ2V0KCdzdWJyZWdpb25zJywgW10pCmV4Y2VwdDoKbm9ydGhfYW1lcmljYV9zdWJyZWdpb25zID0gW10KJSoqKiogaWNscjIwMjRfY29uZmVyZW5jZS50ZXggTGluZSAyMjAwICoqKipccGFyIyBZb3UgY2FuIG5vdyBhbmFseXplIHRoZSBjcnlwdG9jdXJyZW5jeSBkYXRhIGFuZCBwbGFuIHlvdXIKIyB0cmlwIHRvIE5vcnRoIEFtZXJpY2EgdXNpbmcgdGhlIHN1YnJlZ2lvbnMgaW5mb3JtYXRpb24uCg==)import requests\par# 获取加密货币数据crypto_data = prices_and_up_and_down()\par# 获取北美子区域数据subregions_data = get_sub_regions(’North America’)\par# 访问特定的加密货币数据bitcoin_data = crypto_data.get(’Bitcoin’, {})ethereum_data = crypto_data.get(’Ethereum’, {})stellar_data = crypto_data.get(’Stellar’, {})\par# 访问北美的子区域数据try:north_america_subregions = subregions_data.get(’subregions’, [])except:north_america_subregions = []%**** iclr2024_conference.tex 第2200行 ****\par# 现在，您可以分析加密货币数据，并根据子区域信息规划您的北美之行。

1.  (f)

    ```
    Query: I need to find a tutorial on how to draw landscapes.
    Please provide me with the details of the most viewed
    landscape drawing tutorial video. Additionally, I would like
    to know the details of the channel that uploaded the video.

    ```

    [⬇](data:text/plain;base64,CmltcG9ydCByZXF1ZXN0cwpccGFyIyBTZWFyY2ggZm9yIGxhbmRzY2FwZSBkcmF3aW5nIHR1dG9yaWFsIHZpZGVvcwpzZWFyY2hfcmVzdWx0ID0gc2VhcmNoKCdsYW5kc2NhcGUgZHJhd2luZyB0dXRvcmlhbCcsIHR5cGU9J3ZpZGVvJywgc2FmZXNlYXJjaD1UcnVlKQpccGFyIyBGaW5kIHRoZSBtb3N0IHZpZXdlZCB2aWRlbwptb3N0X3ZpZXdlZF92aWRlbyA9IE5vbmUKbWF4X3ZpZXdzID0gMApmb3IgdmlkZW8gaW4gc2VhcmNoX3Jlc3VsdC5nZXQoJ2l0ZW1zJywgW10pOgp2aWV3cyA9IGludCh2aWRlby5nZXQoJ3N0YXRpc3RpY3MnLCB7fSkuZ2V0KCd2aWV3Q291bnQnLCAnMCcpKQolKioqKiBpY2xyMjAyNF9jb25mZXJlbmNlLnRleCBMaW5lIDIyMjUgKioqKmlmIHZpZXdzID4gbWF4X3ZpZXdzOgptYXhfdmlld3MgPSB2aWV3cwptb3N0X3ZpZXdlZF92aWRlbyA9IHZpZGVvClxwYXIjIEdldCB0aGUgdmlkZW8gZGV0YWlscwp2aWRlb19pZCA9IG1vc3Rfdmlld2VkX3ZpZGVvLmdldCgnaWQnLCAndmlkZW9faWRfZXhhbXBsZScpCnZpZGVvX2RldGFpbHMgPSB2aWRlbyh2aWRlb19pZCkKXHBhciMgR2V0IHRoZSBjaGFubmVsIGRldGFpbHMKY2hhbm5lbF9pZCA9IG1vc3Rfdmlld2VkX3ZpZGVvLmdldCgnc25pcHBldCcsIHt9KS5nZXQoJ2NoYW5uZWxJZCcsICdjaGFubmVsX2lkX2V4YW1wbGUnKQpjaGFubmVsX2RldGFpbHMgPSB2aWRlbyhjaGFubmVsX2lkKQpccGFyIyBUaGUgdmlkZW9fZGV0YWlscyBhbmQgY2hhbm5lbF9kZXRhaWxzIHZhcmlhYmxlcyBjb250YWluIHRoZQojIHJlcXVpcmVkIGluZm9ybWF0aW9uCg==)导入请求\par# 搜索风景画教程视频search_result = search(’landscape drawing tutorial’, type=’video’, safesearch=True)\par# 查找观看次数最多的视频most_viewed_video = Nonemax_views = 0for video in search_result.get(’items’, []):views = int(video.get(’statistics’, {}).get(’viewCount’, ’0’))%**** iclr2024_conference.tex 第2225行 ****if views > max_views:max_views = viewsmost_viewed_video = video\par# 获取视频详情video_id = most_viewed_video.get(’id’, ’video_id_example’)video_details = video(video_id)\par# 获取频道详情channel_id = most_viewed_video.get(’snippet’, {}).get(’channelId’, ’channel_id_example’)channel_details = video(channel_id)\par# video_details 和 channel_details 变量包含了所需的信息

#### A.5.2 PrivateEval

1.  (a)

    ```
    Query: Could you help me load a multilingual dataset? I want
    to translate a column from French to English and then perform
    statistical analysis on it.

    ```

    [⬇](data:text/plain;base64,CiMgTG9hZCB0aGUgbXVsdGlsaW5ndWFsIGRhdGFzZXQgKGFzc3VtaW5nIGl0J3MgYSBDU1YgZmlsZSkKY3N2X2ZpbGUgPSAncGF0aC90by95b3VyL211bHRpbGluZ3VhbF9kYXRhc2V0LmNzdicKa25vd2xlZGdlZnJhbWUgPSBsb2FkX2Nzdihjc3ZfZmlsZSkKXHBhciMgRXh0cmFjdCB0aGUgRnJlbmNoIGNvbHVtbiAoYXNzdW1pbmcgaXQncyBuYW1lZCAnZnJlbmNoX2NvbHVtbicpCmZyZW5jaF9jb2x1bW4gPSBrbm93bGVkZ2VmcmFtZSgnZnJlbmNoX2NvbHVtbicpClxwYXIjIFRyYW5zbGF0ZSB0aGUgRnJlbmNoIGNvbHVtbiB0byBFbmdsaXNoCnRyYW5zbGF0ZWRfY29sdW1uID0gW10KZm9yIHRleHQgaW4gZnJlbmNoX2NvbHVtbjoKdHJhbnNsYXRlZF90ZXh0ID0gdHJhbnNsYXRlKHRleHQsIGxhbmd1YWdlPSdlbicpCnRyYW5zbGF0ZWRfY29sdW1uLmFwcGVuZCh0cmFuc2xhdGVkX3RleHQpClxwYXIjIEFkZCB0aGUgdHJhbnNsYXRlZCBjb2x1bW4gdG8gdGhlIEtub3dsZWRnZUZyYW1lCmtub3dsZWRnZWZyYW1lWydlbmdsaXNoX2NvbHVtbiddID0gdHJhbnNsYXRlZF9jb2x1bW4KXHBhciMgUGVyZm9ybSBzdGF0aXN0aWNhbCBhbmFseXNpcyBvbiB0aGUgdHJhbnNsYXRlZCBjb2x1bW4KYW5hbHlzaXNfa2YgPSBzdGF0c19hbmFseXNpcyhrbm93bGVkZ2VmcmFtZSkKJSoqKiogaWNscjIwMjRfY29uZmVyZW5jZS50ZXggTGluZSAyMjc1ICoqKio=)# Load the multilingual dataset (assuming it’s a CSV file)csv_file = ’path/to/your/multilingual_dataset.csv’knowledgeframe = load_csv(csv_file)\par# Extract the French column (assuming it’s named ’french_column’)french_column = knowledgeframe(’french_column’)\par# Translate the French column to Englishtranslated_column = []for text in french_column:translated_text = translate(text, language=’en’)translated_column.append(translated_text)\par# Add the translated column to the KnowledgeFrameknowledgeframe[’english_column’] = translated_column\par# Perform statistical analysis on the translated columnanalysis_kf = stats_analysis(knowledgeframe)%**** iclr2024_conference.tex Line 2275 ****

1.  (b)

    ```
    Query: Load data regarding baby food preferences, analyze
    the preferences across different age groups, and generate
    a report summarizing the most preferred food items

    ```

    [⬇](data:text/plain;base64,CiMg加载包含调查数据的CSV文件

1.  (c)

    ```
    Query: Could you help me find the Chinchilla LLM paper? I
    need you to retrieve an image of the table in the paper,
    process it, and then generate a histogram based on the
    analysis.

    ```

    [⬇](data:text/plain;base64,CmZyb20gbW9ua2V5IGltcG9ydCBLbm93bGVkZ2VGcmFtZQpmcm9tIHR5cGluZyBpbXBvcnQgTGlzdCwgRGljdCwgVW5pb24KXHBhciMgU2VhcmNoIGZvciAiQ2hpbmNoaWxsYSBMTE0iIHBhcGVyIG9uIGFyWGl2CnF1ZXJ5ID0gIkNoaW5jaGlsbGEgTExNIgpwYXBlcnMgPSBhcnhpdl9xdWVyeShxdWVyeSkKXHBhciMgQXNzdW1pbmcgZmlyc3QgcmVzdWx0IGlzIHJlbGV2YW50IGxvYWQgdGhlIFBERiBkYXRhCnBkZl9kYXRhID0gbG9hZF9wZGZfZGF0YShwYXBlcnNbMF0pClxwYXIjIFNlYXJjaCB0aGUgaW1hZ2Ugb2YgdGhlIHRhYmxlIGluIHRoZSAiQ2hpbmNoaWxsYSBMTE0iIHBhcGVyCmltYWdlX3VybCA9IGJpbmdfaW1hZ2Vfc2VhcmNoKCdDaGluY2hpbGxhIExMTSBwYXBlciB0YWJsZScpClxwYXIjIFByb2Nlc3MgdGhlIGltYWdlCnByb2Nlc3NlZF9pbWFnZSA9IHByb2Nlc3NfaW1hZ2UoaW1hZ2VfdXJsLCBmZWF0dXJlcz1bJ29iamVjdHMnXSkKJSoqKiogaWNscjIwMjRfY29uZmVyZW5jZS50ZXggTGluZSAyMzI1ICoqKipccGFyIyBDb252ZXJ0IHRoZSBwcm9jZXNzZWQgaW1hZ2UgdG8gYSBLbm93bGVkZ2VGcmFtZQprbm93bGVkZ2VfZnJhbWUgPSByZWFkX3R4dChwcm9jZXNzZWRfaW1hZ2UuZ2V0KCdvYmplY3RzJywgJ2R1bW15X29iamVjdHNfdGV4dCcpKQpccGFyIyBQZXJmb3JtIHN0YXRpc3RpY2FsIGFuYWx5c2lzIG9uIHRoZSBLbm93bGVkZ2VGcmFtZQphbmFseXNpc19rZiA9IHN0YXRzX2FuYWx5c2lzKGtub3dsZWRnZV9mcmFtZSkKXHBhciMgQnVpbGQgYSBoaXN0b2dyYW0gYmFzZWQgb24gdGhlIGFuYWx5c2lzCmJ1aWxkX2hpc3QoYW5hbHlzaXNfa2YpCg==)从猴子导入KnowledgeFrame从类型导入List、Dict、Union  

1.  (d)

    ```
    Query: Could you please retrieve some images of dogs? After
    that, process it and perform data augmentation using simple
    image processing techniques and save the augmented images.

    ```

    [⬇](data:text/plain;base64,CiMgImport必要的库from typing import List, Dict, Unionimport beatnumfrom PIL import Image\par# 获取狗的图片%**** iclr2024_conference.tex 行号2350 ****image_urls = bing_image_search(query=’dogs’)\par# 加载第一张图片image = process_image(image_urls[0])image_array = beatnum.bdnumset(image)\par# 使用简单的图像处理技术进行数据增强rotated_image_array = rotate(image_array, direction=’clockwise’)flipped_image_array = flip(image_array, axis=1)cropped_image_array = crop(image_array, scale_factor=0.5)grayscale_image_array = to_grayscale(image_array)\par# 创建上面数组的图像对象rotated_image_data = Image.fromarray(rotated_image_array)flipped_image_data = Image.fromarray(flipped_image_array)cropped_image_data = Image.fromarray(cropped_image_array)grayscale_image_data = Image.fromarray(grayscale_image_array)\par# 保存增强后的图像rotated_image_data.save(’rotated_dog_image.png’)flipped_image_data.save(’flipped_dog_image.png’)cropped_image_data.save(’cropped_dog_image.png’)grayscale_image_data.save(’grayscale_dog_image.png’)
