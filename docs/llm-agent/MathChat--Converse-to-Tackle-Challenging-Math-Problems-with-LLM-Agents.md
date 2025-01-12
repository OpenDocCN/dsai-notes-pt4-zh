<!--yml

分类：未分类

日期：2025-01-11 13:09:05

-->

# MathChat: 通过对话解决挑战性数学问题的LLM代理

> 来源：[https://arxiv.org/html/2306.01337/](https://arxiv.org/html/2306.01337/)

Yiran Wu¹, Feiran Jia¹, Shaokun Zhang¹, Hangyu Li², Erkang Zhu³, Yue Wang³,

Yin Tat Lee⁴, Richard Peng⁵, Qingyun Wu¹, Chi Wang³

¹宾夕法尼亚州立大学 ²伦敦帝国学院 ³微软研究院雷德蒙德

⁴华盛顿大学 ⁵滑铁卢大学

{yiran.wu, feiran.jia, shaokun.zhang, qingyun.wu}@psu.edu,

{ekzhu, wang.yue, wang.chi}@microsoft.com, hl6021@ic.ac.uk,

yintat@uw.edu, y5peng@uwaterloo.ca

###### 摘要

使用大型语言模型（LLMs）来解决数学问题是一个引人入胜的研究课题，考虑到在众多科学和工程领域中，数学问题通常以自然语言表达。LLMs凭借其广泛的能力，作为基础模型被用来构建用于不同任务的AI代理。本文研究了利用LLM代理通过对话解决数学问题的有效性。我们提出了MathChat，一个针对数学问题的对话式问题解决框架。MathChat由一个LLM代理和一个用户代理组成，后者负责工具执行和额外的指导。通过这种协作，促进了一个协同解决问题的过程，其中代理通过对话来解决问题。我们在MATH数据集中的困难高中竞赛问题上进行了评估。通过使用Python，我们展示了MathChat在工具使用提示方法的基础上能进一步提高6%的表现。

## 1 引言

随着大型语言模型（LLMs）在各个领域的任务中展示出显著的能力（Bubeck等人，[2023](https://arxiv.org/html/2306.01337v3#bib.bib3)；Zhang等人，[2023c](https://arxiv.org/html/2306.01337v3#bib.bib46)），它们被认为是构建自主代理的潜在基础模型（Xi等人，[2023](https://arxiv.org/html/2306.01337v3#bib.bib40)；Wang等人，[2023a](https://arxiv.org/html/2306.01337v3#bib.bib31)；Zhang等人，[2024](https://arxiv.org/html/2306.01337v3#bib.bib47)）。尤其是，随着任务复杂性的不断增加，多代理协作成为一个有前景的方向，具有信息共享和不同技能的代理之间集体决策的优势。考虑到数学在众多科学和工程学科中的重要作用（Wigner，[1990](https://arxiv.org/html/2306.01337v3#bib.bib36)）以及自然语言中表达的数学问题的普遍存在，探索LLMs在解决数学问题中的潜力显得尤为吸引人。

在这项工作中，我们研究了通过代理之间的对话来解决具有挑战性的数学问题的潜力。由于这些问题的复杂性，我们通常需要将其分解为多个步骤，只有在所有先前的步骤都正确时，才能取得有意义的进展。我们认为对话（结合代码执行）是一种理想的形式，它能够对每个步骤进行迭代优化和调试。我们提出了MathChat，这是一个针对基于聊天的LLM的对话框架，其中数学问题通过LLM代理和用户代理之间的模拟对话来解决（请参见图[1](https://arxiv.org/html/2306.01337v3#S1.F1 "图1 ‣ 1 引言 ‣ MathChat：通过LLM代理对话解决具有挑战性的数学问题")中的示例，以及图[2](https://arxiv.org/html/2306.01337v3#S2.F2 "图2 ‣ 提示方法 ‣ 2 相关工作 ‣ MathChat：通过LLM代理对话解决具有挑战性的数学问题")中的工作流程）。我们还研究并结合了有效的提示方法，以指导基于LLM的代理更有效地解决具有挑战性的数学问题。

我们使用GPT-4在MATH数据集（Hendrycks等，[2021](https://arxiv.org/html/2306.01337v3#bib.bib12)）上评估MathChat，该数据集是从各种竞赛和教育层次中提取的数学问题的综合集合。我们针对该数据集中的5级难度问题，主要包括一些具有挑战性的高中竞赛问题，甚至大学生也认为这些问题很难。我们认识到代码执行在性能提升方面的重要作用，因此我们比较了两种都使用Python的方法：Program of Thoughts (PoT) 提示（Chen等，[2022](https://arxiv.org/html/2306.01337v3#bib.bib5)）和Program Synthesis提示（Drori等，[2022](https://arxiv.org/html/2306.01337v3#bib.bib8)）。我们还包括了一个原始提示作为参考。评估结果表明，MathChat能够将以前使用工具的提示方法提高6%，并且在所有类别中表现竞争力，能够在一半类别上达到60%的准确率。我们还展示了MathChat在不同提示和不同工具下的扩展性。我们对所有评估方法的失败原因进行了详细分析。

![参见说明](img/efd01bd8d846ff73058ff5be964ff94c.png)

图1：MathChat数学问题求解过程的示例。用户代理首先通过将待解决的数学问题发送给LLM代理（带有预设提示）来启动对话。根据GPT-4的回复，用户代理提取所有代码并按顺序执行。来自先前运行的有效代码将被记录下来，并与新代码一起执行，以反映模型逐步推理的进展。结果将返回给GPT-4，GPT-4将继续问题求解过程。虽然在本示例中GPT-4仅通过一次用户消息解决问题，但我们的框架支持多轮对话和额外查询处理，如图[3](https://arxiv.org/html/2306.01337v3#S3.F3 "图 3 ‣ 3 MathChat：一个解决数学问题的对话框架 ‣ MathChat：与LLM代理对话解决挑战性数学问题")所示。用户代理将在LLM代理的回应中进行模式匹配（在我们的例子中，即包含最终答案的\boxed{}），以决定是否结束对话。

## 2 相关工作

#### LLM代理系统

在LLM代理系统领域，已有多种实现展示了多代理AI模型的实用性和多样性。BabyAGI（BabyAGI，[2023](https://arxiv.org/html/2306.01337v3#bib.bib1)）展示了一个基于多个LLM代理并采用静态代理对话模式的AI驱动任务管理系统，而CAMEL（Li et al., [2023](https://arxiv.org/html/2306.01337v3#bib.bib16)）则展示了一个强调角色扮演和自主合作的沟通代理框架。此外，关于多代理辩论的研究（Liang et al., [2023](https://arxiv.org/html/2306.01337v3#bib.bib18); Du et al., [2023](https://arxiv.org/html/2306.01337v3#bib.bib9)）突出了代理辩论在增强LLM的发散性思维和事实性方面的有效性。MetaGPT（Hong et al., [2023](https://arxiv.org/html/2306.01337v3#bib.bib13)）是一个专用应用，展示了GPT在协作软件开发中的使用。AutoGen（Wu et al., [2023](https://arxiv.org/html/2306.01337v3#bib.bib37)）是一个开源框架，用于创建多种LLM应用，通过定制化、可交互的代理，结合LLM、人工输入和工具。

#### 提示方法

近年来，使用大语言模型（LLMs）解决数学问题的创新方法层出不穷（Wang 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib33)；Zhou 等人，[2023](https://arxiv.org/html/2306.01337v3#bib.bib50)；Zheng 等人，[2023](https://arxiv.org/html/2306.01337v3#bib.bib49)；Chen 等人，[2021](https://arxiv.org/html/2306.01337v3#bib.bib4)；Weng 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib35)；Wu 等人，[2024](https://arxiv.org/html/2306.01337v3#bib.bib39)）。其中一个重要的尝试是使用 LLMs 将算术运算和其他基本操作从数学问题求解中卸载到程序中（Drori 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib8)；Chen 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib5)；Gao 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib11)）。累积推理（Cumulative Reasoning）（Zhang 等人，[2023d](https://arxiv.org/html/2306.01337v3#bib.bib48)；Song 等人，[2024](https://arxiv.org/html/2306.01337v3#bib.bib30)）将任务分解为更小的组成部分，简化了解题过程，并以累积的方式生成思路。计划与解决（Plan & Solve）提示（Wang 等人，[2023b](https://arxiv.org/html/2306.01337v3#bib.bib32)）要求 LLM 首先生成一个计划，然后根据该计划进行求解。其他用于提升推理的一般方法也可应用于数学问题：（1）链式推理（Chain-of-thought，CoT）提示（Wei 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib34)；Kojima 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib15)）促使 LLM 进行逐步推理。（2）另一种有效的方式是提示 LLM 以多阶段的方式解决问题（Dua 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib10)；Press 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib26)；Creswell 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib7)；Long，[2023](https://arxiv.org/html/2306.01337v3#bib.bib19)；Paranjape 等人，[2023](https://arxiv.org/html/2306.01337v3#bib.bib23)；Yao 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib42)；Yang 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib41)；Long，[2023](https://arxiv.org/html/2306.01337v3#bib.bib19)；Besta 等人，[2023](https://arxiv.org/html/2306.01337v3#bib.bib2)）。最少到最多提示（Least-to-most prompting）（Zhou 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib51)）和分解式提示（Decomposed prompting）（Khot 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib14)）将复杂问题分解为更小的子问题，然后按顺序解决子问题以得到最终答案。（3）利用工具可以显著提高 LLM 的表现（Shen 等人，[2024](https://arxiv.org/html/2306.01337v3#bib.bib29)；Parisi 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib24)）。ReAct（Yao 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib42)）和 ART（Paranjape 等人，[2023](https://arxiv.org/html/2306.01337v3#bib.bib23)）都使用少量示例提示（few-shot prompting）将逐步推理与工具使用交替进行。（4）自一致性（Self-consistency）（Wang 等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib33)），建立在 CoT 的基础上，通过对问题进行多次推理路径的采样，并选择多数投票的答案。Li 等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib17)）扩展了自一致性，通过训练一个验证器来验证每个步骤的正确性。通过分解问题解决过程，思维树（Tree-of-Thoughts）（Yao 等人，[2023](https://arxiv.org/html/2306.01337v3#bib.bib43)）为每个中间步骤提出了一组思路，探索并保持最有前景的思路以执行后续动作。

![参见标题](img/e0dc8b7707e70a100e81f77e3eeb0af5.png)

图 2：MathChat 工作流程：当一个数学问题输入到 MathChat 后，用户代理将启动与 LLM 代理的对话来解决问题。在每轮互动中，用户代理处理来自 LLM 代理的消息（助手消息）并用用户消息做出回应。这个过程将持续，直到用户代理检测到某种模式来结束对话。图中右侧矩形框内的过程展示了当接收到助手消息后，用户代理的内部工作流程。它展示了执行任何工具使用查询（如 Python 代码）的功能。它还负责根据来自 LLM 代理的不同类型消息给出不同的指令（更多内容见附录 [A](https://arxiv.org/html/2306.01337v3#A1 "附录 A 关于用户代理的补充细节 ‣ MathChat：与 LLM 代理对话解决挑战性数学问题")）。为了说明代理的这个特性，我们在图 [3](https://arxiv.org/html/2306.01337v3#S3.F3 "图 3 ‣ 3 MathChat：一个用于解决数学问题的对话框架 ‣ MathChat：与 LLM 代理对话解决挑战性数学问题") 中给出了一个具体例子。

## 3 MathChat：一个用于解决数学问题的对话框架

本节介绍 MathChat，一个用于数学问题解决的对话框架。

![参见标题](img/98797ed6f23f39826569f27e29345d18.png)

图 3：一个示例，展示用户代理如何处理从 GPT-4 在 MathChat 中接收到的不同类型消息。具体来说，用户代理可能以以下几种方式回应：（1）请求 LLM 代理继续，因为未检测到代码块（即查询）；（2）返回代码执行的有效结果；（3）返回 Python 执行的错误信息。请注意，如果基于用户代理的消息，GPT-4 认为旧代码不再需要，它可能会更改查询。在最后一步中，GPT-4 修正了查询，并返回最终结果。

带有用户代理的对话框架。MathChat 是一个模拟 LLM 代理（在我们的案例中为 GPT-4）与用户代理之间对话的框架。这里的用户代理是扮演用户角色，与 LLM 代理进行对话的代理。在 MathChat 中，LLM 代理和用户代理共同解决数学问题。该框架的工作流程如图 [2](https://arxiv.org/html/2306.01337v3#S2.F2 "Figure 2 ‣ Prompting Methods ‣ 2 Related Work ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents") 所示。用户代理将待解决的数学问题作为输入，并启动与 LLM 代理的对话。用户代理的初始消息包括初始提示和待解决的问题。初始提示用于指示 LLM 代理以特定的方式与用户（在 MathChat 系统中实际上是用户代理）协作解决问题。该框架以这种对话方式设计，旨在利用最先进的 LLM（如 GPT-4）的聊天优化特性。该框架的另一个明显优势是它支持多轮对话，这在处理需要多步骤推理和工具使用的复杂问题时特别有用。

![参见标题](img/ced2f4ae0400dde6b3fb65257ea90e77.png)

图 4：MathChat 中用户代理初始消息中使用的提示。它指示 LLM 代理以特定方式与用户代理协作解决问题。

在 MathChat 中的提示和工具使用。通过适当的修改，可以将现有研究中的有效提示方法，如 CoT 和工具使用，整合到 MathChat 框架中。具体来说，对于初始消息中的提示，我们聚合了多种有效的提示技术来指示 LLM 代理。我们在图 [4](https://arxiv.org/html/2306.01337v3#S3.F4 "Figure 4 ‣ 3 MathChat: A conversational framework for math problem solving ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents") 中展示了设计的提示，包含三个主要组件。

+   •

    工具使用提示：该组件提示 LLM 使用正确格式的 Python 编程来解决问题。我们使用“查询要求”子部分来指定编码格式，以便用户代理可以解析代码并返回相应的结果。

+   •

    问题解决策略选择提示：该组件指示 LLM 代理从三种可能的问题解决策略中选择，并在最后一种策略中执行多阶段推理和工具使用。问题解决策略包括以下三种情况，这些策略涵盖了现有文献中关于数学问题解决的最有效策略。*(情况 1) 编写 Python 程序直接解决问题。* 这对应于类似于 Gao 等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib11)）；Drori 等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib8)）；Chen 等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib5)）的单阶段工具使用方法。*(情况 2) 直接不使用 Python 解决问题。* 这种策略允许 GPT-4 运用其固有的推理能力来解决手头的问题。*(情况 3) 步骤分解解决问题，并使用 Python 来帮助进行数学运算。* 如果前两种方法不适用，我们要求模型选择这种方法来解决问题。我们设计了一个零样本版本的多步骤工具使用提示，允许模型在多步骤推理和 Python 代码之间灵活交替，类似于 Yao 等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib42)）；Paranjape 等人（[2023](https://arxiv.org/html/2306.01337v3#bib.bib23)）；Schick 等人（[2023](https://arxiv.org/html/2306.01337v3#bib.bib27)）。在这种情况下，我们还要求模型处理程序运行中的错误和意外结果，参见 Ni 等人（[2023](https://arxiv.org/html/2306.01337v3#bib.bib21)）。

+   •

    最终答案封装提示：该提示组件指示 LLM 代理将最终答案包含在 \boxed{} 中，这将作为结束对话的指示符。在检测到 \boxed{} 或达到最大对话轮次之前，LLM 代理与用户代理之间的互动不会结束。

我们承认可能存在设计提示的其他方法。幸运的是，在我们的框架中，进一步启用 Wolfram Alpha 除 Python 外的使用是相当容易的。我们根据经验评估两种替代版本的提示，并在第[5.2](https://arxiv.org/html/2306.01337v3#S5.SS2 "5.2 Additional evaluation on MathChat with alternative prompts ‣ 5 Results ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")节中进行测试。

## 4 评估

数据集。我们在MATH数据集的测试集上对所有5级（最高难度级别）问题进行了评估（Hendrycks等人（[2021](https://arxiv.org/html/2306.01337v3#bib.bib12)））。与其他数学问题数据集（如GSM8k（Cobbe等人（[2021](https://arxiv.org/html/2306.01337v3#bib.bib6)）））相比，5级问题要更具挑战性，涉及定理应用和复杂方程推导。MATH数据集有7个问题类别：初级代数、代数、数论、计数与概率、几何、中级代数和预备微积分。在我们的评估中，我们从评估中移除了几何部分，以使其与之前的工作（Drori等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib8)））保持一致（附录[B](https://arxiv.org/html/2306.01337v3#A2 "Appendix B Supplementary Details on Experiment Settings ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")中有更多解释）。

评估方法。大多数以前的工作使用少量示例来引导LLMs的推理和工具使用。选择与未解答问题相似的示例非常重要，然后对这些示例进行注释，以覆盖LLMs可能遇到的所有情况。在这个过程中需要大量的努力和仔细的考虑。例如，Khot等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib14)）；Zhou等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib51)）依赖详细的示例来展示模式，Paranjape等人（[2023](https://arxiv.org/html/2306.01337v3#bib.bib23)）维护一个示例库来选择示例。请注意，这些方法使用的是基础数学问题，而对于更具挑战性的数学问题，准备和选择所需示例的工作量要大得多。另一方面，多项现有研究OpenAI（[2023](https://arxiv.org/html/2306.01337v3#bib.bib22)）；Bubeck等人（[2023](https://arxiv.org/html/2306.01337v3#bib.bib3)）揭示了GPT-4出色的遵循指令的能力。因此，我们对零-shot提示技术感兴趣，这些技术可以在不需要示例选择和注释的情况下增强GPT-4的数学解题能力。根据这一标准，我们评估了我们的MathChat框架，采用了引入的提示和以下方法，这些方法都是零-shot方法：基础提示，思想程序（Program of Thoughts）陈等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib5)），以及Drori等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib8)）的程序合成提示。

1.  1.

    基础提示：GPT-4可以在没有少量示例的情况下执行CoT推理。为了直接评估GPT-4解决问题的表现，我们使用了一个从MATH数据集的少量示例提示中调整过来的默认提示：“仔细解题。将最终答案放在\boxed{}中。{问题}”。

1.  2.

    思维程序（Program of Thoughts, PoT）：我们使用 Chen 等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib5)）提出的零-shot PoT 提示方法，该方法要求模型编写一个 Solver 函数来解决问题并直接返回最终答案。

1.  3.

    程序合成（PS）提示：与 PoT 类似，程序合成（PS）提示方法由 Drori 等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib8)）提出，该方法要求模型编写一个程序来解决问题：“编写一个程序，回答以下问题：{问题}”

评估细节。超参数在决定模型性能中起着至关重要的作用，Zhang 等人（[2023a](https://arxiv.org/html/2306.01337v3#bib.bib44)；[b](https://arxiv.org/html/2306.01337v3#bib.bib45)）。为了确保公平比较，我们在所有方法中都使用 OpenAI API 上 GPT-4 的默认配置。在 MathChat 中，我们允许 GPT-4 和用户代理之间最多进行 15 轮消息交换。如果检测到连续 3 次执行的错误，代理会明确要求 GPT-4 解决每一步。如果用户代理返回的结果超过 600 个 token，代理会在用户消息中替换该结果，并提供警告文本，要求 GPT-4 修改之前的代码。我们手动检查所有方法的答案，统计正确答案的数量。对于普通提示、程序合成（Program Synthesis）和 MathChat，我们要求 GPT-4 将最终答案包裹在 \boxed{} 中，因此只有框中的答案会被提取。对于 PoT，我们按照原始论文的要求，将 Solver 函数的返回值作为最终答案。

## 5 结果

### 5.1 主要结果

我们对 MATH 数据集中的六个类别的 5 级问题进行了评估。我们在表格 [1](https://arxiv.org/html/2306.01337v3#S5.T1 "Table 1 ‣ 5.1 Main Results ‣ 5 Results ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents") 中报告了不同方法在每个类别中的解题准确率。与展示 GPT-4 本地能力的普通提示相比，使用 Python 和 PoT 或 PS 可将整体准确率提高约 10%。我们可以看到，这一改进主要体现在涉及更多数字操作的类别（计数与概率、数论）和更具挑战性的类别（中级代数和预备微积分）。然而，对于代数和初等代数，PoT 和 PS 的改进较小，甚至可能导致准确率降低。与 PoT 和 PS 相比，MathChat 可以进一步提高约 6% 的总体准确率，并在所有类别中表现出竞争力。值得一提的是，MathChat 在代数类别上的准确率相较于其他方法提高了约 15%。考虑到所有方法，中级代数和预备微积分的解题准确率仍然较低，仅约 20%。在其他类别中，MathChat 能够正确解决超过一半的问题。

|  | 代数 | 条件概率 | 中级代数 | 数论 | 初等代数 | 预备微积分 | 总体 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 问题数量 | 307 | 123 | 280 | 154 | 193 | 135 | 1192 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MathChat | 59.93% | 52.03% | 17.85% | 60.39% | 60.10% | 19.26% | 44.71% |
| PoT | 42.67% | 50.41% | 17.50% | 54.55% | 52.33% | 16.30% | 37.67% |
| PS | 43.32% | 44.71% | 20.36% | 61.03% | 55.96% | 18.52% | 39.60% |
| Vanilla | 46.58% | 25.20% | 2.86% | 28.57% | 54.92% | 7.41% | 28.69% |

表格 1：不同方法在 MATH 数据集中不同类别的难度级别为 5 的所有问题上的准确率。

### 5.2 使用替代提示对 MathChat 进行的额外评估

|  | 代数 | 条件概率 | 算法 | 数论 | 预代数 | 预微积分 | 总计 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 问题数量 | 50 | 50 | 50 | 50 | 50 | 50 | 300 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MathChat w/ Two-tools | 33 | 22 | 6 | 27 | 29 | 10 | 127 |
| MathChat w/ Python | 26 | 19 | 7 | 22 | 31 | 13 | 118 |
| MathChat | 30 | 24 | 8 | 34 | 28 | 10 | 134 |
| PoT | 20 | 19 | 9 | 24 | 24 | 7 | 103 |
| PS | 17 | 19 | 12 | 31 | 26 | 5 | 110 |
| Vanilla | 26 | 13 | 1 | 17 | 21 | 1 | 79 |

表格 2：使用两种替代提示对 MathChat 进行的额外评估。本次评估从每个问题类别中随机抽取了 50 个问题。MathChat w/Two-tools 和 MathChat w/ Python 是两种替代提示。

MathChat 允许轻松整合不同的提示和工具。我们进行了额外的评估，测试了 MathChat 的两种替代初始提示，以展示其可扩展性。(1) 一个简化的 Python 提示：在这种替代方案中，我们仅保留默认提示中的“查询需求”子部分，针对 Python 编码格式和逐步使用工具（即案例 3）。(2) 一个简化的 Python 和 Wolfram Alpha 提示：在这种替代方案中，基于替代方案 (1)，我们增加了 Wolfram Alpha 作为计算引擎，供 LLM 代理选择。关于这两种替代提示的详细信息，请参见附录 [B](https://arxiv.org/html/2306.01337v3#A2 "附录 B 实验设置的补充详情 ‣ MathChat：与 LLM 代理对话，解决具有挑战性的数学问题")。我们从六个问题类别中随机抽取了 50 个样本进行评估。我们还将其他方法在这些样本问题上的结果与之对比，详见表格 [2](https://arxiv.org/html/2306.01337v3#S5.T2 "表 2 ‣ 5.2 使用替代提示的 MathChat 额外评估 ‣ 5 结果 ‣ MathChat：与 LLM 代理对话，解决具有挑战性的数学问题")。MathChat 在使用这两种新提示时，仍然优于其他方法。在 MathChat 中，允许同时使用 Python 和 Wolfram 的逐步提示在代数问题上表现最佳，而仅使用 Python 的新提示在前代数和前微积分问题上解决了最多问题，但在数论问题上的表现较差。总体而言，使用默认提示的 MathChat 仍然表现最佳。

## 6 失败分析

![参见说明文字](img/0b1716476a5fd72a2d77ff707da9b49e.png)

图 5：每个前两个失败的示例选取一个。类型 1 失败：在第一个问题中，LLM 代理未能给出合理的计划。它选择列举所有序列，并且没有使用工具来帮助完成。类型 2 失败：第二个问题展示了模型未能给出正确的代码来解决问题，尽管它遵循了问题要求，整体方向是正确的。只需对代码做小幅修改，最终答案即可正确。

### 6.1 失败原因

我们首先根据失败的原因总结失败案例，基于George Pólya（[2004](https://arxiv.org/html/2306.01337v3#bib.bib25)）建立的系统数学问题求解过程。该过程包括（1）理解问题；（2）设计计划；（3）执行计划；（4）回顾和扩展。我们观察到以下三种主要类型的失败。我们在图[5](https://arxiv.org/html/2306.01337v3#S6.F5 "图5 ‣ 6 失败分析 ‣ MathChat：通过对话处理具有挑战性的数学问题与LLM代理")中为两种失败类型各给出了一个示例。更多示例请参见附录[D](https://arxiv.org/html/2306.01337v3#A4 "附录D 失败分析补充 ‣ MathChat：通过对话处理具有挑战性的数学问题与LLM代理")。

类型1：未能设计或选择适当的计划或路径来解决问题。此类型包括GPT-4未能提供正确的解决问题方式的情况。在这些情况下，答案始终不正确，尽管每个计算步骤都是准确的。这类失败通常是非常棘手的问题，即使是数学专家也会感到挑战。

类型2：未能完美执行设计的计划。数学问题求解需要严谨和精确的推理步骤。计算或符号操作中的一个小错误可能是致命的，并导致错误答案。这类错误被认为是“轻微的”，因为它们更容易修正。这类错误会导致相当多的失败，尽管问题求解的整体方向是正确的，但基本推导中的一个错误会导致错误的答案。请注意，Python执行中的错误也包括在此类型中，其中GPT-4未能编写可运行的代码，从而导致错误。

类型3：其他技术性错误。还有其他技术性错误导致失败。一个这样的失败示例是由于删除了ASY代码导致的信息缺失。

### 6.2 使用不同方法在GPT-4上的失败案例

![参见说明文字](img/abacad1701634ea675e791d6a4a8a03d.png)

图6：MathChat正确而其他方法失败的示例。所有其他方法由于类型2错误而失败。1. 普通提示：在计算$b$时，未包括$-4^{2}$。2. PoT：首先错误地计算了vieta_product，即使这已被纠正，仍会发生另一个TypeError。3. PS：它正确求解了$b$，但在使用程序提取$m$和$n$时出现ValueError。

在表[3](https://arxiv.org/html/2306.01337v3#S6.T3 "Table 3 ‣ 6.2 Failure cases using different methods on GPT-4 ‣ 6 Failure Analysis ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")中，我们展示了每种方法成功的结果频率（每行表示），而所有其他方法失败，并按不同问题实例分类。该表旨在突出特定方法在各类问题中的独特优势。同样，在表[4](https://arxiv.org/html/2306.01337v3#S6.T4 "Table 4 ‣ 6.2 Failure cases using different methods on GPT-4 ‣ 6 Failure Analysis ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")中，我们总结了一个方法失败且所有其他方法成功的实例频率。该表中较高的数字表示该方法的独特劣势。

这些统计数据展示了MathChat相较于其他方法的鲁棒性。MathChat通过对话来增强在使用外部工具时的错误纠正，从而假设在第三种类型中减少失败的可能性。

|  | 代数 | 条件概率 | 递归算法 | 网络理论 | 预代数 | 预微积分 | 总计 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MathChat | 27 | 8 | 21 | 13 | 6 | 9 | 84 |
| PoT | 11 | 9 | 19 | 6 | 3 | 5 | 53 |
| PS | 12 | 6 | 22 | 11 | 10 | 8 | 69 |
| Vanilla | 12 | 4 | 5 | 3 | 10 | 3 | 37 |

表 3：一个方法成功，而所有其他方法失败的问题数量（对于每一行的相关方法，数字越高越好）。

|  | 代数 | 条件概率 | 递归算法 | 网络理论 | 预代数 | 预微积分 | 总计 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MathChat | 6 | 2 | 0 | 5 | 4 | 1 | 18 |
| PoT | 22 | 5 | 0 | 6 | 18 | 2 | 53 |
| PS | 17 | 5 | 1 | 5 | 14 | 0 | 42 |
| Vanilla | 16 | 19 | 11 | 28 | 19 | 5 | 98 |

表 4：一个方法失败，而所有其他方法成功的问题数量（对于每一行的相关方法，数字越低越好）。

我们从每个表格中选取一个例子，分析这些方法的失败情况。我们首先选择一个代数问题，MathChat成功解决，但其他方法失败，用以分析其他方法的失败情况（图[6](https://arxiv.org/html/2306.01337v3#S6.F6 "Figure 6 ‣ 6.2 Failure cases using different methods on GPT-4 ‣ 6 Failure Analysis ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")）。对于这个问题，其他方法未能毫无错误地执行计划，导致了第二种类型的失败。虽然常规提示存在计算错误，但另外两种方法则因执行代码时出现错误而失败。我们再运行这些方法三次，它们仍然未能解决问题。从表[4](https://arxiv.org/html/2306.01337v3#S6.T4 "Table 4 ‣ 6.2 Failure cases using different methods on GPT-4 ‣ 6 Failure Analysis ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")中，我们选取了MathChat错误的唯一一个预备数学实例，而其他所有方法均正确。通过调查，我们发现MathChat给出的解法比其他方法更长，而且还包含了第二类型的失败。这可能表明解答的准确性与长度之间存在潜在的相关性。我们在附录[D](https://arxiv.org/html/2306.01337v3#A4 "Appendix D Supplementary Failure Analysis ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")中提供了更多关于这一可能相关性的详细调查，以及预备数学实例的详细信息。

## 7 总结与未来工作

### 7.1 总结

在本文中，我们介绍了MathChat，一个通过LLM智能体与用户代理智能体协作来解决数学问题的对话框架。MathChat是为像GPT-4这样的聊天优化模型设计的，并且它可以扩展以便通过最小的努力与不同的提示和不同的工具一起使用。基于该框架，我们还推导出了一种聚合之前提示技术的提示，可用于MathChat。我们对MATH数据集中的5级问题进行的评估展示了MathChat在解决更复杂和具有挑战性的问题方面的有效性。尽管相比之前的方法有所改进，但结果显示，即便借助外部工具，复杂的数学问题对于像GPT-4这样强大的LLM仍然具有挑战性。我们在下面讨论了进一步改进数学问题解决方法的潜在方向。

### 7.2 在问题解决中增强智能体专业化

《心智社会》Minsky ([1988](https://arxiv.org/html/2306.01337v3#bib.bib20))提出，智能源于相对简单的代理之间的互动。在MathChat中，两个代理协作解决数学问题，LLM代理在特定任务完成和决策指导下的行为表现出显著差异。为了提高一致性和有效性，可以将这一过程分解为专门的任务，每个任务由一个专门的代理处理。例如，一个代理可以专注于理解和提出初步解决方案，而另一个代理则评估适用于给定问题的最合适的解决策略。

除了解构解决过程外，还可以根据问题类型、难度等级或其他方面对问题进行分类，并相应地为每个类别选择最有效的代理（提示方法）。我们在第[6.2](https://arxiv.org/html/2306.01337v3#S6.SS2 "6.2 使用不同方法在 GPT-4 上的失败案例 ‣ 6 失败分析 ‣ MathChat：通过对话解决挑战性数学问题与 LLM 代理")节中的分析表明，依据问题类型的不同，各种方法在不同类型的问题中展示出显著的优势。这种方法类似于Shazeer等人提出的专家混合模型([2017](https://arxiv.org/html/2306.01337v3#bib.bib28))，其中使用特定的提示策略代替子神经网络，也与Wu等人([2022](https://arxiv.org/html/2306.01337v3#bib.bib38))提出的提示链概念相一致，该概念涉及将任务在不同场景下分类，以便针对性地解决。

### 7.3 人类问题解决中的辅助作用

尽管LLM在辅助人类问题解决方面显示出巨大的潜力，我们也意识到在开发一个可靠的基于LLM的解决问题助手方面仍有很多工作需要完成。在第[6](https://arxiv.org/html/2306.01337v3#S6 "6 失败分析 ‣ MathChat：通过对话解决挑战性数学问题与 LLM 代理")节进行失败分析时，我们可以轻松地发现LLM响应中的计算错误，但在识别逻辑或事实不准确时可能会遇到困难，特别是在处理不熟悉的概念时。这可能会导致潜在的错误信息，尤其是对于那些正在学习新概念且判断力较弱的学生。尽管我们的评估表明，结合Python能增强LLM的解决问题能力，但仅依赖Python或LLM也存在局限性，因为Python解决方案（例如暴力破解或仿真）可能不适合人类学习的需求。

一种可能的缓解措施是使用外部工具和已有知识验证解决过程的每一步。当LLM生成解决问题的每个中间步骤时，可以使用Python检查该步骤中的计算错误，并通过外部数据库验证所提到的定理。

## 参考文献

+   BabyAGI (2023) BabyAGI. Github — babyagi. [https://github.com/yoheinakajima/babyagi](https://github.com/yoheinakajima/babyagi), 2023.

+   Besta et al. (2023) Maciej Besta, Nils Blach, Ales Kubicek, Robert Gerstenberger, Lukas Gianinazzi, Joanna Gajda, Tomasz Lehmann, Michal Podstawski, Hubert Niewiadomski, Piotr Nyczyk, 等人. 思维图谱：通过大型语言模型解决复杂问题. *arXiv 预印本 arXiv:2308.09687*, 2023.

+   Bubeck et al. (2023) Sébastien Bubeck, Varun Chandrasekaran, Ronen Eldan, Johannes Gehrke, Eric Horvitz, Ece Kamar, Peter Lee, Yin Tat Lee, Yuanzhi Li, Scott Lundberg, 等人. 人工通用智能的火花：与GPT-4的早期实验. *arXiv 预印本 arXiv:2303.12712*, 2023.

+   Chen et al. (2021) Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman, 等人. 评估训练于代码的大型语言模型. *arXiv 预印本 arXiv:2107.03374*, 2021.

+   Chen et al. (2022) Wenhu Chen, Xueguang Ma, Xinyi Wang, 和 William W Cohen. 思维提示程序：为数字推理任务解构计算与推理. *arXiv 预印本 arXiv:2211.12588*, 2022.

+   Cobbe et al. (2021) Karl Cobbe, Vineet Kosaraju, Mohammad Bavarian, Mark Chen, Heewoo Jun, Lukasz Kaiser, Matthias Plappert, Jerry Tworek, Jacob Hilton, Reiichiro Nakano, 等人. 训练验证者解决数学文字题. *arXiv 预印本 arXiv:2110.14168*, 2021.

+   Creswell et al. (2022) Antonia Creswell, Murray Shanahan, 和 Irina Higgins. 选择推理：利用大型语言模型进行可解释的逻辑推理. *arXiv 预印本 arXiv:2205.09712*, 2022.

+   Drori et al. (2022) Iddo Drori, Sarah Zhang, Reece Shuttleworth, Leonard Tang, Albert Lu, Elizabeth Ke, Kevin Liu, Linda Chen, Sunny Tran, Newman Cheng, 等人. 一种神经网络通过程序合成和少样本学习在人的水平上解决、解释并生成大学数学问题. *美国国家科学院院刊*, 119(32):e2123433119, 2022.

+   Du et al. (2023) Yilun Du, Shuang Li, Antonio Torralba, Joshua B Tenenbaum, 和 Igor Mordatch. 通过多代理辩论提高语言模型的事实性和推理能力. *arXiv 预印本 arXiv:2305.14325*, 2023.

+   Dua et al. (2022) Dheeru Dua, Shivanshu Gupta, Sameer Singh, 和 Matt Gardner. 通过连续提示分解复杂问题. *arXiv 预印本 arXiv:2212.04092*, 2022.

+   Gao et al. (2022) Luyu Gao, Aman Madaan, Shuyan Zhou, Uri Alon, Pengfei Liu, Yiming Yang, Jamie Callan, 和 Graham Neubig. Pal：程序辅助语言模型. *arXiv 预印本 arXiv:2211.10435*, 2022.

+   Hendrycks et al. (2021) Dan Hendrycks, Collin Burns, Saurav Kadavath, Akul Arora, Steven Basart, Eric Tang, Dawn Song, 和 Jacob Steinhardt. 使用数学数据集衡量数学问题解决能力. *arXiv 预印本 arXiv:2103.03874*, 2021.

+   Hong 等人（2023）Sirui Hong, Xiawu Zheng, Jonathan Chen, Yuheng Cheng, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, Chenyu Ran 等人。Metagpt: 多代理协作框架的元编程。*arXiv 预印本 arXiv:2308.00352*, 2023。

+   Khot 等人（2022）Tushar Khot, Harsh Trivedi, Matthew Finlayson, Yao Fu, Kyle Richardson, Peter Clark 和 Ashish Sabharwal。分解式提示：解决复杂任务的模块化方法。*arXiv 预印本 arXiv:2210.02406*, 2022。

+   Kojima 等人（2022）Takeshi Kojima, Shixiang Shane Gu, Machel Reid, Yutaka Matsuo 和 Yusuke Iwasawa。大型语言模型是零-shot推理者。*arXiv 预印本 arXiv:2205.11916*, 2022。

+   Li 等人（2023）Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin 和 Bernard Ghanem。Camel: 用于大规模语言模型社会“心智”探索的交互式代理，2023。

+   Li 等人（2022）Yifei Li, Zeqi Lin, Shizhuo Zhang, Qiang Fu, Bei Chen, Jian-Guang Lou 和 Weizhu Chen。提高语言模型推理能力的进展。*arXiv 预印本 arXiv:2206.02336*, 2022。

+   Liang 等人（2023）Tian Liang, Zhiwei He, Wenxiang Jiao, Xing Wang, Yan Wang, Rui Wang, Yujiu Yang, Zhaopeng Tu 和 Shuming Shi。通过多代理辩论鼓励大语言模型中的发散思维，2023。

+   Long（2023）Jieyi Long。大型语言模型引导的思维树。*arXiv 预印本 arXiv:2305.08291*, 2023。

+   Minsky（1988）Marvin Minsky。*Society of mind*。Simon and Schuster, 1988。

+   Ni 等人（2023）Ansong Ni, Srini Iyer, Dragomir Radev, Ves Stoyanov, Wen-tau Yih, Sida I Wang 和 Xi Victoria Lin。Lever: 学习通过执行验证语言到代码生成的能力。*arXiv 预印本 arXiv:2302.08468*, 2023。

+   OpenAI（2023）OpenAI。Gpt-4 技术报告，2023。

+   Paranjape 等人（2023）Bhargavi Paranjape, Scott Lundberg, Sameer Singh, Hannaneh Hajishirzi, Luke Zettlemoyer 和 Marco Tulio Ribeiro。Art: 自动多步推理和工具使用的语言模型。*arXiv 预印本 arXiv:2303.09014*, 2023。

+   Parisi 等人（2022）Aaron Parisi, Yao Zhao 和 Noah Fiedel。Talm: 工具增强语言模型。*arXiv 预印本 arXiv:2205.12255*, 2022。

+   Polya（2004）George Polya。*How to solve it: A new aspect of mathematical method*。第246号。普林斯顿大学出版社，2004。

+   Press 等人（2022）Ofir Press, Muru Zhang, Sewon Min, Ludwig Schmidt, Noah A Smith 和 Mike Lewis。衡量并缩小语言模型中的组合性差距。*arXiv 预印本 arXiv:2210.03350*, 2022。

+   Schick 等人（2023）Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Luke Zettlemoyer, Nicola Cancedda 和 Thomas Scialom。Toolformer: 语言模型可以自我学习使用工具。*arXiv 预印本 arXiv:2302.04761*, 2023。

+   沙兹尔等人（2017）诺姆·沙兹尔、阿扎利亚·米尔霍塞尼、克日什托夫·马济亚兹、安迪·戴维斯、乐国、杰弗瑞·辛顿、杰夫·迪恩。极其庞大的神经网络：稀疏门控专家混合层。*arXiv预印本 arXiv:1701.06538*，2017年。

+   沈等人（2024）沈永良、宋凯涛、谭旭、李东生、陆伟明、庄越庭。HuggingGPT：通过Hugging Face中的ChatGPT及其伙伴解决AI任务。*神经信息处理系统进展*，第36卷，2024年。

+   宋等人（2024）宋林欣、刘佳乐、张杰宇、张少坤、罗奥、王世剑、吴庆云、王驰。语言模型智能体的自适应对话团队构建。*arXiv预印本 arXiv:2405.19425*，2024年。

+   王等人（2023a）王磊、马晨、冯雪阳、张泽宇、杨浩、张景森、陈智远、唐家凯、陈旭、林艳凯等。基于大型语言模型的自主智能体调查。*arXiv预印本 arXiv:2308.11432*，2023年。

+   王等人（2023b）王磊、许婉瑜、蓝易怀、胡志强、兰云诗、李凯维、林怡鹏。计划与解决提示：通过大型语言模型改善零-shot思维链推理。*arXiv预印本 arXiv:2305.04091*，2023年。

+   王等人（2022）王雪智、魏杰森、戴尔·舒尔曼斯、乐国、艾德·池、周丹尼。自一致性改善语言模型中的思维链推理。*arXiv预印本 arXiv:2203.11171*，2022年。

+   魏等人（2022）魏杰森、王雪智、戴尔·舒尔曼斯、马尔滕·博斯马、艾德·池、乐国、周丹尼。思维链提示激发大型语言模型中的推理。*arXiv预印本 arXiv:2201.11903*，2022年。

+   翁等人（2022）翁一轩、朱敏俊、何世柱、刘康、赵军。大型语言模型是具有自我验证的推理者。*arXiv预印本 arXiv:2212.09561*，2022年。

+   维格纳（1990）尤金·P·维格纳。数学在自然科学中的非理性有效性。载于*数学与科学*，第291–306页，世界科学出版社，1990年。

+   吴等人（2023）吴庆云、班加恩·班萨尔、张杰宇、吴亦然、张少坤、朱尔康、李贝彬、姜莉、张晓云、王驰。Autogen：通过多智能体对话框架推动下一代LLM应用。*arXiv预印本 arXiv:2308.08155*，2023年。

+   吴等人（2022）吴同双、姜艾伦、阿伦·董斯巴赫、杰夫·格雷、阿莱汉德拉·莫利纳、迈克尔·特里、凯丽·J·蔡。Promptchainer：通过视觉编程连接大型语言模型提示。载于*CHI计算机系统人类因素会议扩展摘要*，第1–10页，2022年。

+   吴等人（2024）吴亦然、岳天威、张少坤、王驰、吴庆云。Stateflow：通过状态驱动工作流增强LLM任务解决能力。*arXiv预印本 arXiv:2403.11322*，2024年。

+   席等人（2023）席智恒、陈文翔、郭鑫、何伟、丁怡文、洪博阳、张铭、王俊哲、金森杰、周恩宇等。基于大型语言模型的智能体崛起与潜力：一项调查。*arXiv预印本 arXiv:2309.07864*，2023年。

+   Yang 等人（2022）Jingfeng Yang, Haoming Jiang, Qingyu Yin, Danqing Zhang, Bing Yin 和 Diyi Yang。Seqzero：利用顺序提示和零-shot 模型进行少量样本的组合语义解析。*arXiv 预印本 arXiv:2205.07381*，2022。

+   Yao 等人（2022）Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan 和 Yuan Cao。React：在语言模型中协同推理与行动。*arXiv 预印本 arXiv:2210.03629*，2022。

+   Yao 等人（2023）Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Thomas L Griffiths, Yuan Cao 和 Karthik Narasimhan。思维树：在大型语言模型中进行深思熟虑的问题解决。*arXiv 预印本 arXiv:2305.10601*，2023。

+   Zhang 等人（2023a）Shaokun Zhang, Feiran Jia, Chi Wang 和 Qingyun Wu。针对多目标的词典式偏好进行超参数优化。收录于 *第十一届国际学习表征会议*，2023a。

+   Zhang 等人（2023b）Shaokun Zhang, Yiran Wu, Zhonghua Zheng, Qingyun Wu 和 Chi Wang。Hypertime：针对时间分布变化的超参数优化。*arXiv 预印本 arXiv:2305.18421*，2023b。

+   Zhang 等人（2023c）Shaokun Zhang, Xiaobo Xia, Zhaoqing Wang, Ling-Hao Chen, Jiale Liu, Qingyun Wu 和 Tongliang Liu。Ideal：通过影响驱动的选择性注释增强上下文学习者在大型语言模型中的表现。*arXiv 预印本 arXiv:2310.10873*，2023c。

+   Zhang 等人（2024）Shaokun Zhang, Jieyu Zhang, Jiale Liu, Linxin Song, Chi Wang, Ranjay Krishna 和 Qingyun Wu。无需修改语言模型即可训练语言模型代理。*arXiv 预印本 arXiv:2402.11359*，2024。

+   Zhang 等人（2023d）Yifan Zhang, Jingqin Yang, Yang Yuan 和 Andrew Chi-Chih Yao。利用大型语言模型进行累积推理。*arXiv 预印本 arXiv:2308.04371*，2023d。

+   Zheng 等人（2023）Chuanyang Zheng, Zhengying Liu, Enze Xie, Zhenguo Li 和 Yu Li。渐进提示改善大型语言模型的推理能力。*arXiv 预印本 arXiv:2304.09797*，2023。

+   Zhou 等人（2023）Aojun Zhou, Ke Wang, Zimu Lu, Weikang Shi, Sichun Luo, Zipeng Qin, Shaoqing Lu, Anya Jia, Linqi Song, Mingjie Zhan 等人。使用 gpt-4 代码解释器和基于代码的自我验证解决具有挑战性的数学应用题。*arXiv 预印本 arXiv:2308.07921*，2023。

+   Zhou 等人（2022）Denny Zhou, Nathanael Schärli, Le Hou, Jason Wei, Nathan Scales, Xuezhi Wang, Dale Schuurmans, Olivier Bousquet, Quoc Le 和 Ed Chi。最少到最多提示法使大型语言模型具备复杂推理能力。*arXiv 预印本 arXiv:2205.10625*，2022。

## 附录 A 用户代理代理的补充细节

MathChat中的用户代理接收一个问题并将其放入带有初始提示的消息中，然后将该消息发送给LLM代理。随后，代理负责提取并执行查询，同时提供额外的指导。以下是用户代理的所有功能（工作流程见图[2](https://arxiv.org/html/2306.01337v3#S2.F2 "Figure 2 ‣ Prompting Methods ‣ 2 Related Work ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")）：

1.  1.

    提取查询：用户代理需要匹配初始消息中指定的模式来提取所有工具使用的查询。通过我们设计的提示，代理匹配消息中的所有代码块并提取代码。

1.  2.

    ”继续“：如果在消息中没有检测到查询，代理将发送此消息给LLM代理：“继续。请继续解决问题，直到需要查询为止。（如果得出答案，请将其放入\boxed{}中。）”。这要求代理继续解决问题，并提醒它在给出答案时将其放入框中以结束对话。

1.  3.

    查询执行：所有提取的工具使用查询将按顺序执行。对于Python，我们将执行时间限制设置为5秒。如图[1](https://arxiv.org/html/2306.01337v3#S1.F1 "Figure 1 ‣ 1 Introduction ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")所示，之前有效的代码会被记录下来。所有执行结果将按顺序连接（包括错误）。

1.  4.

    循环错误检测：如果LLM代理连续发送3个错误，用户代理将把第三个错误消息替换为以下消息：“请重新审视问题陈述和您的推理。如果您认为这一步是正确的，请自己解决它并继续下一步。否则，请修正这一步。”为了避免停留在此错误中，LLM代理被要求在没有工具的情况下解决此步骤并继续前进。

1.  5.

    重复结果：这个情况在工作流程中没有显示，但代理还会检测到另一种情况，即LLM代理给出的工具使用查询与上一个查询相同，或者结果与上一个查询的结果相同。此时，消息将被附加到执行结果中，提醒代理避免给出相同的查询：“您的查询或结果与上一个相同，请尝试新的方法。”。

1.  6.

    长查询结果：LLM代理可能需要一个过长的查询结果，无法传回（例如Python中的for循环中的print函数输出的长结果）。代理将把任何超过2000个字符（约600个tokens）的查询结果替换为以下消息：“您的查询响应过长，可能存在错误。请修改您的推理和查询。”。

在MathChat中，如果在同一条消息中检测到工具使用查询和结束指示符，查询的结果将被返回，且对话将继续进行。这是为了防止早期停止，即当LLM代理预测执行结果并将其放入框中，而不是等待结果时的情况。  

## 附录B 实验设置的补充细节  

移除几何问题的理由：该数据集中的大多数几何问题包含一个用于绘制图形的Asymptote代码。但是目前可用的GPT-4版本无法接受图像输入。如果包含原始代码，模型可能通过坐标的精确数字泄露信息。因此，考虑到这些问题，我们跳过了几何问题的评估，并从所有其他类别中移除了ASY代码（尽管这可能导致某些问题缺乏足够的信息）。每个问题的正确答案是确定性的，并且在数据集中作为地面真相被\boxed{}括起来（但不会向解题方法透露）。

代码在这个[GitHub](https://github.com/yiranwu0/MathChat)仓库中。在我们的实验中，我们使用OpenAI的默认配置，具体为temperature=1，max_token=inf（详细信息请参见[OpenAI API参考](https://platform.openai.com/docs/api-reference/chat/create)）。我们为普通提示、PS和MathChat使用系统消息”你是一个有帮助的助手”。对于PoT，我们不添加此系统消息，因为我们的评估表明，PoT在没有系统消息的情况下表现更好。我们在[C](https://arxiv.org/html/2306.01337v3#A3 "附录C 补充实验和结果 ‣ MathChat：与LLM代理一起解决具有挑战性的数学问题")节中讨论了系统消息的影响。  

以下是PoT Chen等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib5)）、PS Drori等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib8)）以及我们设计的另外两个提示：  

+   •  

    思维程序（PoT）。参见图[10](https://arxiv.org/html/2306.01337v3#A2.F10 "图10 ‣ 附录B 实验设置的补充细节 ‣ MathChat：与LLM代理一起解决具有挑战性的数学问题")。整个提示使用Python代码格式，其中问题或指令等信息以注释的形式呈现。  

+   •  

    程序合成（PS）。PS 的提示是 "编写一个程序来回答以下问题：{问题}"。由于“程序”的定义不明确，有时 LLM 代理不会编写代码来解决问题，我们在“程序”前面加上了关键字‘Python’。返回消息后，我们使用代理返回结果（默认情况下，GPT-4 会将代码返回在代码块中）。然后，我们向模型发送另一条消息，包含 Python 执行结果，并要求它将最终答案括起来：{Python 返回的结果}。请将最终答案放在 \boxed{} 中。有关整个过程的示例，请参见图 [9](https://arxiv.org/html/2306.01337v3#A2.F9 "图 9 ‣ 附录 B 实验设置的补充细节 ‣ MathChat：与 LLM 代理对话以解决具有挑战性的数学问题")。

+   •

    Python 提示（带有 MathChat）。参见图 [7](https://arxiv.org/html/2306.01337v3#A2.F7 "图 7 ‣ 附录 B 实验设置的补充细节 ‣ MathChat：与 LLM 代理对话以解决具有挑战性的数学问题")。

+   •

    双工具提示（带有 MathChat）。参见图 [8](https://arxiv.org/html/2306.01337v3#A2.F8 "图 8 ‣ 附录 B 实验设置的补充细节 ‣ MathChat：与 LLM 代理对话以解决具有挑战性的数学问题")。

![请参阅标题说明](img/da2d7cd8987fff4d61f8bb39807b9962.png)

图 7：MathChat 中使用的 Python 提示，参见第 [5.2](https://arxiv.org/html/2306.01337v3#S5.SS2 "5.2 关于使用替代提示对 MathChat 的附加评估 ‣ 5 结果 ‣ MathChat：与 LLM 代理对话以解决具有挑战性的数学问题") 节。

![请参阅标题说明](img/415b8624180e510ce887ccd3c52d5634.png)

图 8：MathChat 中使用的双工具提示，参见第 [5.2](https://arxiv.org/html/2306.01337v3#S5.SS2 "5.2 关于使用替代提示对 MathChat 的附加评估 ‣ 5 结果 ‣ MathChat：与 LLM 代理对话以解决具有挑战性的数学问题") 节。与 Python 提示相比，添加的要求已用黄色标出。该提示允许 LLM 代理从 Python 或 Wolfram Alpha 中选择。

![请参阅标题说明](img/9b8a14290f500e1bde0f9a90ddfb9700.png)

图 9：PS 过程的示例。查询结果将返回给 LLM 助手，并要求其将答案放入框中。PS 的过程与 MathChat 中的过程完全相同，当 MathChat 中的代理选择使用 Python 程序解决问题时。

![请参阅标题说明](img/d0dc61d87ff50a7acfbb911a0ae7eea3.png)

图 10：PoT 提示。与 Chen 等人（[2022](https://arxiv.org/html/2306.01337v3#bib.bib5)）的原始提示相比，我们添加了 "import sympy as sp"，这为 LLM 代理提供了使用 sympy 库的提示。占位符 "{problem}" 将被实际问题替换。

## 附录 C 补充实验与结果

我们进一步评估了基础少量示例提示、PoT（有无系统消息）以及基础提示（有无系统消息），并在每个类别随机选择的50个问题上进行了测试，结果如图[5](https://arxiv.org/html/2306.01337v3#A3.T5 "Table 5 ‣ Appendix C Supplementary Experiments and Results ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")所示。

在少量示例提示中，我们从训练集中随机选择了3对级别为5的问题-解决方案。这些示例是从每个类别中选择的，并且用于该类别的所有问题。基础的少量示例提示从“仔细解决问题开始。像基础提示一样将最终答案放入\boxed{}，然后附上三个“问题：... 解决方案：...”的对。”与基础提示相比，添加三个额外的示例没有明显的区别，总体表现略有下降。

从我们的实验中，我们还注意到系统消息会影响LLM代理的表现。然而，不同方法之间的影响差异显著。如表[5](https://arxiv.org/html/2306.01337v3#A3.T5 "Table 5 ‣ Appendix C Supplementary Experiments and Results ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")所示，使用系统消息对基础提示至关重要：与没有系统消息的情况相比，添加系统消息使成功率翻倍。然而，对于PoT，添加系统消息仅略微提高了性能。我们进一步对所有级别为5的问题进行了评估，发现PoT（带系统消息）的整体准确率为35.82%，低于没有系统消息的PoT（如表[1](https://arxiv.org/html/2306.01337v3#S5.T1 "Table 1 ‣ 5.1 Main Results ‣ 5 Results ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")中的主结果所示，准确率为37.67%）。我们假设不同方法的提示格式差异是这种行为的原因。使用基础提示的方法模拟了LLM代理与人类之间通过自然语言的对话，而PoT提示则是Python代码格式，明确指导模型进行代码补全。因此，系统消息“你是一个有帮助的助手”更适合基础提示，但与PoT不匹配。需要更多的研究来理解系统消息的影响。

|  | 代数 | 条件概率 | 组合算法 | 数论 | 预代数 | 预计算 | 总计 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 问题数量 | 50 | 50 | 50 | 50 | 50 | 50 | 300 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MathChat | 30 | 24 | 8 | 34 | 28 | 10 | 134 |
| PS | 17 | 19 | 12 | 31 | 26 | 5 | 110 |
| PoT w/o sys | 20 | 19 | 9 | 24 | 24 | 7 | 103 |
| PoT w/ sys | 18 | 23 | 9 | 23 | 29 | 7 | 109 |
| Vanilla w/o sys | 14 | 4 | 0 | 4 | 13 | 1 | 35 |
| Vanilla w/ sys | 26 | 13 | 1 | 17 | 21 | 1 | 79 |
| 少量示例（k=3） | 21 | 6 | 2 | 18 | 24 | 1 | 72 |

表5：少量示例提示、带有和不带系统消息的PoT、带有和不带系统消息的原始提示的结果。

## 附录D 补充失败分析

### D.1 数学聊天中不同形式解题过程下的失败

数学聊天中的默认提示允许LLM代理从不同的解题过程形式中选择来解决问题，我们研究了选择不同形式如何影响性能。在图[11](https://arxiv.org/html/2306.01337v3#A4.F11 "图11 ‣ 数学聊天中不同形式解题过程下的失败 ‣ 附录D 补充失败分析 ‣ 数学聊天：使用LLM代理解决挑战性数学问题")中，我们绘制了当每个类别的问题用三种解题方法解决时的正确率，具体取决于生成解决方案中查询的存在和有效性：1\. LLM代理在解决问题时不进行任何工具使用查询（Python）。2\. 代理进行一个或多个查询，但至少有一个查询无效。3\. 代理进行的所有查询均有效。图中显示，正确使用Python可以显著提高正确率，而不使用Python则比使用Python但存在无效查询的情况更差。图[11](https://arxiv.org/html/2306.01337v3#A4.F11 "图11 ‣ 数学聊天中不同形式解题过程下的失败 ‣ 附录D 补充失败分析 ‣ 数学聊天：使用LLM代理解决挑战性数学问题")的结果显示，特别是在中级代数和初级代数的情况下，“没有查询”和“有无效查询”之间的准确率差距很大，表明使用Python对于解决这两类问题非常有帮助。

![参见标题说明](img/1c85a54d47898193880cb40bd91dc857.png)

图11：数学聊天在不同解题过程中成功率的变化：1\. LLM 代理在不进行任何工具使用查询的情况下解决问题。2\. 代理进行查询，并且至少有一个无效查询。3\. 所有查询均为有效查询。

![参见标题说明](img/dbce6ad0e4de487881bcff55d3f58328.png)

图12：类型1失败（未能制定合适的计划）和类型2失败（未能完美执行计划）的额外示例。

![参见标题说明](img/7c0b798848e7346c246d2baf50d28871.png)

图13：类型3失败的一个例子，其中移除了ASY代码。

### D.2 三种类型失败的示例

在第[6.1](https://arxiv.org/html/2306.01337v3#S6.SS1 "6.1 Failure reasons ‣ 6 Failure Analysis ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")节中，我们总结了三种主要的失败类型：类型1：未能制定适当的计划；类型2：未能完美执行计划；类型3：其他技术错误。我们给出了类型1错误和类型2错误的一个附加示例，并展示了一个移除ASY代码导致信息泄漏的示例（图[12](https://arxiv.org/html/2306.01337v3#A4.F12 "Figure 12 ‣ D.1 Failure under different forms of problem-solving processes in MathChat ‣ Appendix D Supplementary Failure Analysis ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")，图[13](https://arxiv.org/html/2306.01337v3#A4.F13 "Figure 13 ‣ D.1 Failure under different forms of problem-solving processes in MathChat ‣ Appendix D Supplementary Failure Analysis ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")）。我们注意到，在所有问题中，72个问题的ASY代码被移除，但仍有12个问题能够正确解决。

### D.3 不同方法下的失败

我们展示了一个前置数学例子，其中MathChat失败，而所有其他方法成功（图[15](https://arxiv.org/html/2306.01337v3#A4.F15 "Figure 15 ‣ D.4 The Relationship Between Failure Rate and Generated Solution Length ‣ Appendix D Supplementary Failure Analysis ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")，图[16](https://arxiv.org/html/2306.01337v3#A4.F16 "Figure 16 ‣ D.4 The Relationship Between Failure Rate and Generated Solution Length ‣ Appendix D Supplementary Failure Analysis ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")，图[17](https://arxiv.org/html/2306.01337v3#A4.F17 "Figure 17 ‣ D.4 The Relationship Between Failure Rate and Generated Solution Length ‣ Appendix D Supplementary Failure Analysis ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")）。PS和PoT的结果显示，使用Python（使用sympy.simplify函数）很容易解决这个问题。然而，在MathChat中，LLM代理选择通过直接推理来解决问题。MathChat和普通提示都通过编写非常长的推导过程来解决该问题。MathChat通过更加冗长的逐步回应解决问题，并且在过程中发生了计算错误。

此外，我们还提供了一个概述，展示了所有方法在表格[6](https://arxiv.org/html/2306.01337v3#A4.T6 "Table 6 ‣ D.3 Failure under different methods ‣ Appendix D Supplementary Failure Analysis ‣ MathChat: Converse to Tackle Challenging Math Problems with LLM Agents")中成功或失败的问题数量。

|  | 代数 | 条件概率 | 指数代数 | 数论 | 前置代数 | 前置数学 | 总计 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 全部成功 | 46 | 13 | 0 | 18 | 45 | 1 | 176 |
| 所有失败 | 57 | 32 | 171 | 20 | 36 | 86 | 402 |   |

表6：所有方法失败和所有方法成功的题目数量。  

### D.4 失败率与生成解答长度之间的关系  

![请参阅标题](img/5ec629737b6860c3b4cf47fa93f01901.png)![请参阅标题](img/1b2d8a30800f6689e12fa6d4838fa23f.png)

图14：MathChat中正确解答和错误解答问题的解答长度分布。还展示了给定解答（地面真实情况）的长度分布。左侧图表示较简单类别的分布，右侧图表示中级代数和预备微积分类别的问题。我们去除了分割字符串长度超过1500的离群值。  

思维链（CoT）提示表明，问题的额外推理步骤可以提高LLM的能力（Wei等人，[2022](https://arxiv.org/html/2306.01337v3#bib.bib34)）。使用GPT-4时，显式推理不再是问题。相反，我们发现，长而繁琐的推理过程可能导致更多类型2失败，例如计算错误，即使总体方向正确，仍会得到错误的答案。我们绘制了正确答案和错误答案的长度分布，并且还展示了给定解答的长度（这里使用通过单个空格拆分后的字符串列表的长度）。由于更复杂和更具挑战性的问题可能会有更长的解决过程，但成功率较低，我们将中级代数和预备微积分的问题与其他类别分开（图[14](https://arxiv.org/html/2306.01337v3#A4.F14 "Figure 14 ‣ D.4 失败率与生成解答长度之间的关系 ‣ 附录D 补充失败分析 ‣ MathChat: 反向应对LLM智能体挑战性数学问题")），以区分较简单的问题与更困难的问题。我们注意到，MathChat在四个较简单类别中的成功率超过50%，而在中级代数和预备微积分类别中的成功率低于20%。  

总体而言，MathChat的解答长度比实际解答更长。在两个根本性挑战性较大的类别中的给定解答长度，超过了其他类别。对于MathChat来说，来自挑战性较小类别的正确答案和错误答案在解答长度上的分布相似，大多数问题的解答长度在50到500之间。然而，对于更难的问题，超过600字符长度的答案往往是错误的。根据图[17](https://arxiv.org/html/2306.01337v3#A4.F17 "图17 ‣ D.4 失败率与生成解答长度之间的关系 ‣ 附录D 补充失败分析 ‣ MathChat：与LLM代理一起应对挑战性数学问题")所示的预备微积分问题，LLM代理可能会选择一个合理的策略来解决问题，但与给定解答相比，这种策略效率较低，并且涉及更多的数学运算，这导致了响应时间大大延长，并且在过程中更容易发生错误。

![参见标题](img/2171c721f3cc042b1fca3e33fc515602.png)

图15：在这个预备微积分问题中，其他方法都正确，但MathChat错误。此图展示了实际解答和使用普通提示的响应。

![参见标题](img/0eb4122d70cbf3876fffa73915d797c2.png)

图16：在这个预备微积分问题中，其他方法都正确，但MathChat错误（续）。此图展示了PS和PoT代码。两段代码都返回了正确的结果：“D”。

![参见标题](img/e0afed2bbcdd0b49a6e19015c02b29b7.png)

图17：在这个预备微积分示例中，其他方法都正确，但MathChat错误（续）。此图展示了MathChat中生成的对话。MathChat中的LLM代理选择通过直接推理来解决问题，并且在步骤3中展开项时发生了计算错误。
