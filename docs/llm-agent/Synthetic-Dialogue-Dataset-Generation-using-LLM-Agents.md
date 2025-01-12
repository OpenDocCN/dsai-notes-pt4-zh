<!--yml

类别：未分类

日期：2025-01-11 12:56:45

-->

# 使用LLM代理生成合成对话数据集

> 来源：[https://arxiv.org/html/2401.17461/](https://arxiv.org/html/2401.17461/)

Yelaman Abdullin    Diego Molla-Aliod

麦考瑞大学

yelaman.abdullin@hdr.mq.edu.au, diego.molla-aliod@mq.edu.au \AND Bahadorreza Ofoghi    John Yearwood

迪肯大学

{b.ofoghi,john.yearwood}@deakin.edu.au &Qingyang Li

墨尔本大学

ql5@student.unimelb.edu.au

###### 摘要

线性规划（LP）问题在现实应用中无处不在。然而，尽管它们表面上看起来简单，未经训练的用户可能会发现很难确定其特定问题的线性模型。我们设想创建一个以目标为导向的对话代理，该代理将与用户进行对话，以提取生成线性模型所需的所有信息。本文提出了一种生成样本对话的方法，旨在开发和训练这样的对话代理。通过提示工程，我们开发了两个代理，它们“互相交谈”，一个作为对话代理，另一个作为用户。使用一组仅供用户使用的来自NL4Opt的线性问题文本描述，代理和用户进行对话，直到代理从原始问题描述中提取出所有关键信息。我们还提出了一种外部评估对话的方式，通过评估对话生成的总结与原始问题描述的匹配程度来进行评估。我们进行了人工和自动评估，包括使用GPT-4模仿人工评估指标的评估方法。评估结果显示，对话的整体质量良好，尽管仍需进一步研究以提高GPT-4评估指标的质量。生成的对话，包括部分人工标注，已提供给研究社区。用于生成对话的对话代理可以作为基准。

## 1 引言

线性规划（LP）是一种广泛应用的数学优化技术，旨在解决多种现实世界中的挑战，从供应链管理中的资源分配到金融中的投资组合优化。尽管线性规划问题无处不在，并且具有重要的实际意义，但许多人，尤其是那些没有专业数学背景的个人，常常难以为其特定问题实例制定合适的线性模型。这个障碍限制了线性规划技术的更广泛应用，特别是在非专家群体中。

为了应对这一挑战，我们提出开发一种能够帮助用户为其特定问题场景构建准确线性模型的目标导向型对话代理。该对话代理将与用户进行对话，提取与问题相关的信息，随后生成相应的线性模型。本文重点讨论创建此类代理的一个关键方面——生成合成对话，这些对话可以用来训练和评估对话代理的性能。

我们的方法利用提示工程构建了两个不同的代理：一个模拟对话代理的行为，另一个模拟用户在问题解决过程中做出的响应。设计这些代理的目的是进行有目的的对话，目的是从用户那里提取构建有效线性模型所需的信息。为了促进这一过程，我们使用了一组来自NL4Opt数据集Ramamonjison等人（[2022](https://arxiv.org/html/2401.17461v1#bib.bib9), [2023](https://arxiv.org/html/2401.17461v1#bib.bib10)）的线性问题文本描述，这些描述仅对用户代理可访问。这些文本描述为对话提供了基础，使得对话代理能够迭代地收集问题制定所需的关键信息。

除了生成过程外，我们还提出了一种外部评估方法，用于评估生成对话的质量和效果。具体来说，我们评估由对话生成的摘要与NL4Opt中原始问题描述的匹配程度。这一评估包括人工评估，人工评审员评估对话摘要的质量，以及自动化指标，用于定量衡量生成摘要的信息量。

我们通过人工和自动化评估得到的初步结果表明，生成的对话在很大程度上忠实于原始问题描述，从而证明了生成的合成对话的质量。

本文的贡献包括：

1.  1.

    本文提出了一种生成对话的方法，用于开发目标导向型对话代理。本文的目标是从用户那里获取信息，以生成线性规划模型，并指出所提出的技术可以适用于其他目标。

1.  2.

    用于开发这种对话代理的数据集包含476个对话，其中28个已手动标注¹¹1[https://github.com/eabdullin/optimouse-quest/](https://github.com/eabdullin/optimouse-quest/)。尽管这476个对话是自动生成的，但由于生成过程是非确定性的，同一程序的不同运行会生成不同的对话。因此，我们认为这些对话为研究社区提供了一个有价值的数据集，有助于促进可重复性研究。

1.  3.

    一种基于比较对话生成的摘要与原始问题描述的外部评估方法。

1.  4.

    一种使用 GPT-4 模拟人类评估行为的自动评估方法。

## 2 背景与相关工作

### 2.1 线性规划与NL4Opt

线性规划问题在现实应用中无处不在。它们通常用于资源分配、规划、调度、交通优化、投资组合管理及众多其他领域。例如，在生产规划中，线性规划可以帮助确定如何使用有限的人力、物资和财务资源来实现最大经济效益。

线性规划问题是一类数学优化问题，其目标是为决策变量找到一组值，这些值满足一组线性约束条件，并使得线性目标函数的值最大化或最小化，参见 Chen 等人 ([2011](https://arxiv.org/html/2401.17461v1#bib.bib1))。线性规划问题的通用形式可以表述如下：

|  | $\begin{array}[]{ll@{}ll}\text{最大化}&\mathbf{c}^{T}&\mathbf{x}&\\ \text{满足条件}&\mathbf{A}&\mathbf{x}\leq\mathbf{b}\\ &&\mathbf{x}\geq\mathbf{0}\end{array}$ |  |
| --- | --- | --- |

其中，$\mathbf{x}$ 是一个决策变量的向量，$\in\mathbb{R}^{n}$，$\mathbf{c}$ 和 $\mathbf{b}$ 是给定的常数向量，$\mathbf{A}$ 是给定的常数矩阵。线性规划因其能够建模现实世界的优化问题以及高效求解算法的存在而广泛应用于各个领域。一旦为一个问题制定了线性规划模型，强大的求解器和高效的算法可以帮助解决，即使是复杂且大规模的问题也能处理。

2022年，一项从自然语言中提取线性规划公式的竞赛（NL4Opt）开发了首个线性规划文字问题数据集，Ramamonjison 等人（[2022](https://arxiv.org/html/2401.17461v1#bib.bib9)，[2023](https://arxiv.org/html/2401.17461v1#bib.bib10)）。该数据集包含来自多个领域的1101个实例。这些线性规划问题涉及诸如生产、资源分配、投资分配、农业、交通、健康科学、销售等常见领域。每个实例都有一个非结构化的自然语言描述，涉及决策变量、一个目标函数和若干约束条件。

NL4Opt数据集提供了有关LP问题的现实世界自然语言描述的宝贵示例，展示了各种约束类型。我们将开发子集中的每个问题描述分解为目标函数描述和几个约束描述。通过分析这些约束，我们发现它们可以分为表[1](https://arxiv.org/html/2401.17461v1#S2.T1 "Table 1 ‣ 2.1 Linear Programming and NL4Opt ‣ 2 Background and Related Work ‣ Synthetic Dialogue Dataset Generation using LLM Agents")中的九个精细类别。数据集的不同实例由这些约束类型的不同组合构成。类型1至3和5至7通常用于表示容量限制、预算约束或资源可用性。类型4、8和9则施加比例控制和不同数量之间的平衡。这九种约束类型在现实世界问题中经常遇到，并且可以涵盖常见约束的大部分。

|  | 约束类型 | 数学不等式 |
| --- | --- | --- |
| 1 | 单个变量的上界 | $x_{i}\leq b$ |
| 2 | 变量和的上界 | $\sum_{i}x_{i}\leq b$ |
| 3 | 加权和的上界 | $\sum_{i}a_{i}x_{i}\leq b$ |
| 4 | 比例的上界 | $x_{j}\leq c\sum_{i}x_{i}$ |
| 5 | 单个变量的下界 | $x_{i}\geq b$ |
| 6 | 变量和的下界 | $\sum_{i}x_{i}\geq b$ |
| 7 | 加权和的下界 | $\sum_{i}a_{i}x_{i}\geq b$ |
| 8 | 比例的下界 | $x_{j}\geq c\sum_{i}x_{i}$ |
| 9 | 比较约束 | $dx_{i}\leq x_{j}$ |

表1：约束分类。假设一个线性规划（LP）问题有$n$个决策变量，$x_{i}$和$x_{j}$是决策变量，$a_{i}$和$b$是非负常数，$d$是一个正常数，$c$是一个常数$\in(0,1]$，$i,j\in I=\{1,2,\ldots,n\}$。

### 2.2 评估方法

对话系统的评估是一项复杂的任务，通常涉及自动化度量和人工评估的结合。像ROUGE Lin ([2004](https://arxiv.org/html/2401.17461v1#bib.bib3))这样的传统自动化度量常用于衡量文本相似度和评估信息重叠。然而，这种度量在评估语义连贯性和有效实现对话目标方面还有待改进（Liu等人，[2016](https://arxiv.org/html/2401.17461v1#bib.bib4)）。在本研究中，我们计算了ROUGE-1、ROUGE-2和ROUGE-L分数，分别覆盖了单字词、双字词和最长公共子序列的重叠。

最近的进展引入了更为复杂的评估方法，包括BERTScore（Zhang等人，[2020](https://arxiv.org/html/2401.17461v1#bib.bib13)），它利用预训练BERT模型的上下文嵌入来评估生成文本与参考文本之间的语义相似性。BERTScore通过为评估添加一个更为细致的语义层来补充ROUGE，捕捉传统指标可能忽略的方面。

此外，大型语言模型（LLM）的兴起进一步丰富了文本生成评估的工具箱（Fu等人，[2023](https://arxiv.org/html/2401.17461v1#bib.bib2)；Liu等人，[2023](https://arxiv.org/html/2401.17461v1#bib.bib5)）。LLM具备细致分析的能力，提供了超越语法和表面层面度量的可能性。它们可能捕捉对话语义和目标对齐的更深层次方面。

## 3 方法论

本节概述了实现本研究两个目标的方法：自动生成对话数据集，以及评估生成对话的质量。我们利用一个双代理设置，借助大型语言模型（LLM），在我们的案例中为OpenAI的GPT-4 OpenAI（[2023](https://arxiv.org/html/2401.17461v1#bib.bib7)），模拟用户与助手之间关于线性规划问题的对话。

### 3.1 双代理LLM设置

双代理设置旨在模拟自动对话代理与用户之间的对话，代理提问以识别线性问题的所有关键信息进行建模。具体而言，该设置包括两个独立的LLM代理：一个问题生成代理和一个问题回答代理，它们彼此“对话”。

+   •

    问题生成（QG）代理：该代理的主要目标是提出问题，以便能够引导并总结原始问题陈述的关键信息。重要的是，该代理无法访问原始问题陈述，只能依赖与问答代理的互动来收集所有必要的信息。

+   •

    问答代理（QA Agent）：该代理的设计目的是根据来自NL4Opt的预定义问题陈述回答问题，NL4Opt充当助手的模拟知识库。为了使对话更加自然且有上下文关联，代理被配置为模仿原问题陈述中提到的个人。此设计选择增强了互动的真实性，创造了一个更为真实的用户-代理对话体验。

### 3.2 实施细节

QA 代理的一个关键部分是一个组件（该组件也利用了大型语言模型 LLM），用于比较摘要与原始问题陈述。这一组件的作用是双重的：当摘要与原始问题陈述之间存在差异时提供反馈，并在没有差异时发出信号，终止对话生成。为了检测 QG 是否在最新的对话轮次中生成了摘要，我们使用了一个简单的方法，利用正则表达式和预定义的关键字。

为了保持一致性并引导 LLM 做出回应，整个对话过程中都注入了提示词。除了总是开始对话的初始系统提示外，后续的提示在每个对话轮次中引入，并保留在对话历史中。对于 QG 代理，提供了简洁的指令“A NEXT MESSAGE/QUESTION”。对于 QA 代理，提示根据对话阶段有所不同。通常使用指令“ANSWER SHORTLY. USE MAXIMUM 30 WORDS.”然而，当 QG 代理提供摘要时，提示会相应调整：要么是关于摘要的反馈²²2完整的提示在附录 [B](https://arxiv.org/html/2401.17461v1#A2 "Appendix B LLM Prompts ‣ Synthetic Dialogue Dataset Generation using LLM Agents")中提供，要么是“THE SUMMARY ACCEPTED. IT’S TIME TO FINISH DIALOG AND SAY GOODBYE”，表示对话结束。

我们使用了来自 NL4Opt 竞赛的线性规划问题数据集（Ramamonjison 等人，[2023](https://arxiv.org/html/2401.17461v1#bib.bib10)），特别是来自子任务 2 的数据集。该任务旨在处理自然语言问题描述、标记的语义实体和有序的变量提及映射，并生成精确的意义表示。这些问题描述作为本研究中两个 LLM 代理之间模拟对话的基础。

+   •

    开发子集：该子集包含 98 个问题。每个问题生成了 2 至 4 个独立的对话。

+   •

    训练子集：对于训练子集，为前 241 个问题生成了单个对话。

双代理系统是通过使用 OpenAI 的聊天完成功能 API³³3[https://platform.openai.com/docs/api-reference/chat](https://platform.openai.com/docs/api-reference/chat)实现的。我们使用不同的温度设置生成了对话，针对开发子集确保对话的多样性。目标是为每个问题生成多个对话实例，以便能够在后续的研究中稳健地评估模型的性能并评估生成数据集的多样性。

图[1](https://arxiv.org/html/2401.17461v1#S3.F1 "图1 ‣ 3.2 实施细节 ‣ 3 方法论 ‣ 使用LLM代理生成合成对话数据集")展示了部分样本对话，图[2](https://arxiv.org/html/2401.17461v1#S3.F2 "图2 ‣ 3.2 实施细节 ‣ 3 方法论 ‣ 使用LLM代理生成合成对话数据集")展示了相关问题描述。附录[B](https://arxiv.org/html/2401.17461v1#A2 "附录B LLM提示 ‣ 使用LLM代理生成合成对话数据集")展示了实际使用的提示。

QG代理：你好！我是OptiMouse，你的友好助手。我在这里帮助你做出最合适的决策。请告诉我你希望实现或改进的目标是什么？ QA代理：你好！我希望在考虑空间、资本和产品比例限制的情况下，最大化我家具店的利润。 QG代理：听起来是个很棒的目标！为了更好地帮助你，能否告诉我你在店里销售哪些家具？ QA代理：我们店里销售餐桌和椅子。 QG代理：太好了！现在，能否告诉我你从销售一张餐桌和一把椅子中分别能获得多少利润？ QA代理：我们从销售一张餐桌中获得$350的利润，从销售一把椅子中获得$75的利润。 QG代理：这些信息很有帮助。现在，能否告诉我餐桌和椅子在你的店里占用多少空间？ QA代理：一张餐桌需要8平方英尺的地面空间，而一把椅子需要2平方英尺。 … … … … QG代理：这是一个非常重要的细节。现在，能否告诉我我们是否需要考虑家具店的其他任何限制或要求？ QA代理：没有，我们家具店没有其他限制或要求。 QG代理：太好了！让我总结一下这些信息：

- 你在家具店里销售餐桌和椅子。

- 你从销售一张餐桌中获得$350的利润，从销售一把椅子中获得$75的利润。

- 一张餐桌需要8平方英尺的地面空间，而一把椅子需要2平方英尺。

- 你在店里有500平方英尺的可用空间。

- 你最多可以用$20000购买餐桌和椅子。

- 一张餐桌的采购成本是$1000，一把椅子的采购成本是$150。

- 店里至少70%的家具必须是椅子。

你还有什么想要添加或修改的内容吗？

图1：QG代理和QA代理之间关于最大化家具店利润的样本对话。

一家家具店仅存储和销售餐桌和椅子。每张餐桌的利润为350美元，每把椅子的利润为75美元。店内有500平方英尺的空间，其中一张餐桌需要8平方英尺的地面空间，而一把椅子需要2平方英尺。由于椅子的销量较大，店内至少70%的家具必须是椅子。就资金而言，一张餐桌需要占用1000美元的资金，一把椅子需要占用150美元的资金。公司希望随时资金的占用不超过20000美元。请制定一个线性规划（LP）模型来最大化利润。

图2：来自NL4Opt数据集的一个示例问题陈述。

## 4 评估策略

为了评估生成对话的质量，我们进行*外部评估*。评估过程侧重于一个基本假设，这一假设是我们研究的核心：QG代理生成的正确总结意味着代理已经提出了适当且必要的问题，以有效恢复原始问题陈述。这个假设基于几个关键假设和逻辑推理：

外部评估。在对话系统中，特别是目标导向的系统中，对话通常通过一系列问题和答案逐步进行，以达成某个最终状态（Qi et al. [2020](https://arxiv.org/html/2401.17461v1#bib.bib8)；Majumder et al. [2021](https://arxiv.org/html/2401.17461v1#bib.bib6)）。因此，提出的问题直接影响最终输出的质量和准确性——在这里，就是原始问题陈述的总结。因此，我们对对话进行外部评估。

初始信息不完整。QG代理在开始时对当前问题了解不完全。因此，提出正确的问题对于代理收集足够的信息以生成准确总结至关重要。一个不准确或不完整的总结表明代理提出的问题不充分或不正确。

语义理解。一个准确的总结不仅涉及事实正确性，还需要对问题的约束和目标有语义上的理解。因此，正确的总结可以视为对代理通过问题把握问题复杂性的隐性验证。

为了验证这些假设，我们采用了一种方法，结合人工评估和通过第三方LLM代理的自动评估来评估生成的总结。通过评估生成总结的正确性，我们间接地评估QG代理提出适当且信息丰富的问题的能力，从而实现对问题的全面理解。

### 4.1 自动评估

对于自动评估，除了著名的 ROUGE 和 BERTScore 外，我们还设计了一个指标，利用第三个 LLM（GPT-4）代理来比较原始问题陈述与生成的摘要。评估是通过使用“链式思维”提示进行的，参考了 Wei 等人（[2022](https://arxiv.org/html/2401.17461v1#bib.bib12)）。我们的代理使用三个标准（“链条”）来评估每个摘要：

+   •

    正确信息：评估摘要是否准确表示了原始问题陈述中的事实。

+   •

    错误信息：识别摘要中的任何事实不准确或具有误导性的陈述。

+   •

    缺失信息：确定摘要中是否忽略了原始问题陈述中的任何关键信息。

考虑到上述三项标准，代理会提供一个“信息召回得分”、“信息精度得分”、“信息重复得分”和“可读性得分”，用于判断摘要是否形成了原始问题的连贯且准确的表达。这些评估指标与第 [4.2](https://arxiv.org/html/2401.17461v1#S4.SS2 "4.2 人类评估 ‣ 4 评估策略 ‣ 基于 LLM 代理的合成对话数据集生成")节中的人工评估相同。附录 [B](https://arxiv.org/html/2401.17461v1#A2 "附录 B LLM 提示 ‣ 基于 LLM 代理的合成对话数据集生成")展示了所使用的提示语。

#### 4.1.1 自动评估结果

| 指标 | 值 |
| --- | --- |
| ROUGE-1 P | 0.54 |
| ROUGE-1 R | 0.62 |
| ROUGE-1 F1 | 0.57 |
| ROUGE-2 P | 0.33 |
| ROUGE-2 R | 0.39 |
| ROUGE-2 F1 | 0.35 |
| ROUGE-L P | 0.38 |
| ROUGE-L R | 0.43 |
| ROUGE-L F1 | 0.40 |
| BERTscore P | 0.88 |
| BERTscore R | 0.91 |
| BERTscore F1 | 0.90 |
| GPT-4 R | 4.60 |
| GPT-4 P | 4.62 |

表 2：整个数据集的指标平均值。所有值的范围为 0 到 1，除了 GPT-4，其范围为 1 到 5。

表 [2](https://arxiv.org/html/2401.17461v1#S4.T2 "表 2 ‣ 4.1.1 自动评估结果 ‣ 4.1 自动评估 ‣ 4 评估策略 ‣ 基于 LLM 代理的合成对话数据集生成")总结了整个数据集的平均指标值。正如我们所观察到的，ROUGE 的绝对值并不是很高。然而，正如我们稍后看到的，人工评估数据的一个子集表明这些对话总体上生成了不错的摘要。

| 名称 | 值 |
| --- | --- |
| 总对话数 | 476 |
| 温度为 0 的对话 | 315 |
| 温度为 1 的对话 | 149 |
| 有摘要的对话 | 97% |
| 平均对话长度（字符数） | 3658 |
| 总轮次数 | 9480 |
| 每个对话的平均轮次 | 20 |
| 平均轮次长度（字符数） | 184 |

表 3：生成对话的摘要统计数据。

为了更细致地理解我们生成的对话内容，我们收集了一些总结性统计数据。这些数据在表[3](https://arxiv.org/html/2401.17461v1#S4.T3 "Table 3 ‣ 4.1.1 Results of the Automatic Evaluation ‣ 4.1 Automatic Evaluation ‣ 4 Evaluation Strategy ‣ Synthetic Dialogue Dataset Generation using LLM Agents")中展示。对话是通过不同的温度设置生成的，以控制LLM生成文本的随机性。在少数情况下（3%），对话未能在40轮的截止之前生成总结（每个代理20轮）。97%对话能生成总结的高比例表明了问题生成代理在结束对话时生成总结的有效性，这对于我们的评估至关重要。对话长度和回合数提供了代理之间对话的深度和广度的指示。

### 4.2 人类评估

对于人类评估部分，我们精心挑选了28个来自开发子集的问题。这个子集的选择涵盖了开发数据中所有出现的约束类型（表[1](https://arxiv.org/html/2401.17461v1#S2.T1 "Table 1 ‣ 2.1 Linear Programming and NL4Opt ‣ 2 Background and Related Work ‣ Synthetic Dialogue Dataset Generation using LLM Agents")），确保跨不同问题场景进行全面的评估。

鉴于问题的基本数量较小（98），选择28个问题非常简单：首先，采用贪心算法满足类型较少的数量要求，然后根据需要手动交换开发集中的候选问题，直到我们达到了一个可接受的约束类型分布。表[4](https://arxiv.org/html/2401.17461v1#S4.T4 "Table 4 ‣ 4.2 Human Evaluation ‣ 4 Evaluation Strategy ‣ Synthetic Dialogue Dataset Generation using LLM Agents")展示了开发集和用于人类评估的28个问题选择中约束类型的分布。

|  | 约束类型 | Dev | Sel |
| --- | --- | --- | --- |
| 1 | 单一变量的上限 | 20 | 6 |
| 2 | 变量和的上限 | 12 | 4 |
| 3 | 加权和的上限 | 93 | 28 |
| 4 | 比例的上限 | 8 | 2 |
| 5 | 单一变量的下限 | 35 | 11 |
| 6 | 变量和的下限 | 7 | 2 |
| 7 | 加权和的下限 | 59 | 18 |
| 8 | 比例的下限 | 15 | 5 |
| 9 | 比较 | 43 | 13 |

表4：98个问题开发集中的线性优化约束类型计数（“Dev”列），以及28个问题选择（“Sel”列）。由于一个问题可能有多个约束，因此总数大于问题数。

为了增加对话生成中的变动性，本次选择的每个问题陈述都经过了一次对话生成运行。此次运行的温度设置随机选为0或1。

四位评估者（详细信息见致谢部分）接着评分生成的摘要在对话结束时与问题陈述的匹配程度。对于每对问题陈述和生成的摘要，每位评估者产生了以下4个评估指标。这些指标采用了Tsatsaronis等人进行的人类评估（[2015](https://arxiv.org/html/2401.17461v1#bib.bib11)）：

+   •

    信息回忆（IR）（1-5）– 生成的摘要包含所有必要的信息。

+   •

    信息精确性（IP）（1-5）– 不生成无关信息。

+   •

    信息重复（IRep）（1-5）– 生成的摘要不会重复相同的信息。

+   •

    可读性（Read）（1-5）– 生成的摘要易于阅读且流畅。

#### 4.2.1 人类评估结果

由Fleiss’ Kappa计算的每个4个评估指标的标注者间一致性，见表[5](https://arxiv.org/html/2401.17461v1#S4.T5 "表 5 ‣ 4.2.1 人类评估结果 ‣ 4.2 人类评估 ‣ 4 评估策略 ‣ 使用LLM代理生成的合成对话数据集")。

| 指标 | Fleiss’ Kappa |
| --- | --- |
| 信息回忆 | 0.205 |
| 信息精确性 | 0.387 |
| 信息重复 | -0.009 |
| 可读性 | 0.235 |

表 5：四个评估指标的标注者间一致性。

我们几乎没有观察到信息重复的一致性，信息回忆和可读性上有轻微一致性，而信息精确性上有公平一致性。

表[6](https://arxiv.org/html/2401.17461v1#S4.T6 "表 6 ‣ 4.2.1 人类评估结果 ‣ 4.2 人类评估 ‣ 4 评估策略 ‣ 使用LLM代理生成的合成对话数据集")显示了28个问题的人工评估得分的平均值。

| 标注者 | IR | IP | IRep | Read |
| --- | --- | --- | --- | --- |
| 1 | 4.25 | 4.25 | 4.89 | 4.96 |
| 2 | 4.18 | 4.54 | 4.93 | 4.96 |
| 3 | 4.68 | 4.39 | 4.93 | 4.86 |
| 4 | 4.03 | 4.36 | 4.82 | 4.89 |
| 所有 | 4.29 | 4.38 | 4.89 | 4.92 |

表 6：28份文档样本的平均人工评估得分。IR = 信息回忆。IP = 信息精确性。IRep = 信息重复。Read = 可读性。

总体而言，人类评估显示出较高的得分，信息重复和可读性得分尤为高。这表明由这对代理生成的对话质量较好。

人类标注者观察到生成摘要中最常见的错误。有关每种错误的示例，请参见附录[A](https://arxiv.org/html/2401.17461v1#A1 "附录 A 示例生成摘要中的错误 ‣ 使用LLM代理生成的合成对话数据集")。

+   •

    缺少目标函数或决策变量。

+   •

    包含附加信息，这些信息似乎来自于代理人提问澄清问题后的回答，例如“否则顺序无关紧要”或“成本没有上限”等等。

### 4.3 自动评估与人工评估的相关性分析

表[7](https://arxiv.org/html/2401.17461v1#S4.T7 "Table 7 ‣ 4.3 Correlation Analysis of Automatic and Human Evaluations ‣ 4 Evaluation Strategy ‣ Synthetic Dialogue Dataset Generation using LLM Agents")显示了自动评估与人工评估之间的斯皮尔曼等级相关系数$\rho$。

| Metric | IR | IP | IF1 | IAvg |
| --- | --- | --- | --- | --- |
| ROUGE-1 R | 0.43 |  |  |  |
| ROUGE-1 P |  | 0.58 |  |  |
| ROUGE-1 F1 |  |  | 0.62 | 0.60 |
| ROUGE-2 R | 0.48 |  |  |  |
| ROUGE-2 P |  | 0.58 |  |  |
| ROUGE-2 F1 |  |  | 0.56 | 0.57 |
| ROUGE-L R | 0.47 |  |  |  |
| ROUGE-L P |  | 0.74 |  |  |
| ROUGE-L F1 |  |  | 0.71 | 0.69 |
| BERTScore R | 0.53 |  |  |  |
| BERTScore P |  | 0.74 |  |  |
| BERTScore F1 |  |  | 0.65 | 0.65 |
| GPT-4 R | 0.42 |  |  |  |
| GPT-4 P |  | 0.67 |  |  |
| GPT-4 F1 |  |  | 0.59 | 0.58 |

表7：自动评估与人工评估之间的斯皮尔曼等级相关系数$\rho$。IP = 与信息召回（Information Recall）的相关性；IR = 与信息精确度（Information Precision）的相关性；IF1 = 与信息召回和信息精确度的调和均值的相关性；IAvg = 与信息召回、信息精确度、信息重复性和可读性的平均值的相关性。

该表比较了自动评估指标的召回率与信息召回（Information Recall）的值，精确度与信息精确度（Information Precision）的值，以及F1值与信息召回与信息精确度的调和均值的关系。表格的最后一列显示了自动评估指标的F1值与信息召回、信息精确度、信息重复性（Information Repetition）和可读性（Readability）平均值之间的相关性。我们可以观察到这些值几乎与信息召回和信息精确度的调和均值一致。这可能是因为信息重复性和可读性的注释几乎总是5，因此它们的贡献几乎是一个常数值，不会改变排名，因此它们对$\rho$值没有影响。这可能是使用GPT-4以及精心设计的提示语所致，这些提示语要求系统清晰简洁。

在所有自动化指标中，表[7](https://arxiv.org/html/2401.17461v1#S4.T7 "Table 7 ‣ 4.3 Correlation Analysis of Automatic and Human Evaluations ‣ 4 Evaluation Strategy ‣ Synthetic Dialogue Dataset Generation using LLM Agents")显示，ROUGE-L具有最佳的相关性值。GPT-4取得了具有竞争力的结果，但未能超越其他指标。进一步的工作仍然需要改进GPT-4的使用。特别是，GPT-4通常更为宽容，会给予比人类评估者更高的评分。引入少量示例和更复杂的提示可能有助于系统与人类注释者之间的一致性。

## 5 总结与结论

本文提出了一个数据集，旨在通过与对话代理的对话从用户那里获取信息。所获取信息的具体用途是用于自动建模线性优化问题。这本身是一个非常有用的任务，具有广泛的潜在应用，但这里提出的数据生成和评估方法也可以轻松应用于其他可能的任务。数据和人类评估已向研究社区开放。¹¹脚注标记：1

对话是在一个双代理LLM设置中生成的，其中一个问题生成代理作为机器代理负责引导信息，另一个问题回答代理作为人类代理拥有问题建模所需的信息。问题生成代理可以作为基线代理。人类评估结果表明，这一基线可能对任务有效。

对28个线性规划问题子集的对话进行了外部评估，评估标准是通过对话生成的摘要是否与原始问题描述中的关键信息相匹配。评估由人类评估者和自动化系统共同进行。在所有自动评估指标中，除了广为人知的ROUGE和BERTScore，我们设计了一个模拟人类评估者的GPT-4代理。结果表明，ROUGE L、BERTScore P与人类信息精确度得分之间存在合理的相关性，且这一相关性略优于GPT-4代理与人类IP得分之间的相关性。

作为进一步的工作，我们计划改进用于评估方法的提示，特别是针对GPT-4。此外，我们还将对数据集进行更多详尽的评估，这些评估可能更适合线性规划建模的特定领域。特别是，我们计划在对话轮次层面对生成的对话进行分析。

## 致谢

本项工作部分得到了澳大利亚研究委员会（Australian Research Council）资助，资金来源为2022年发现项目（项目编号DP220101925）。除了本文的两位作者（Diego Molla-Aliod 和 John Yearwood）负责标注数据外，我们还感谢Vicky Mak-Hau和Thuseethan Selvarajah提供的标注工作。

## 参考文献

+   Chen等人（2011）德三·陈、罗伯特·巴特森和余·邓。2011年。[应用整数规划：建模与求解](https://doi.org/10.1002/9781118166000.ch1)。

+   Fu等人（2023）金兰·傅、谢竞·吴、郑宝·姜和鹏飞·刘。2023年。[GPTScore: 按需评估](http://arxiv.org/abs/2302.04166)。

+   Lin（2004） Chin-Yew Lin。2004年。[ROUGE：自动评估摘要的工具包](https://aclanthology.org/W04-1013)。发表于*文本摘要研究*，第74–81页，西班牙巴塞罗那。计算语言学学会。

+   Liu等人（2016）贾伟·刘、瑞安·洛、尤里安·塞尔班、迈克·诺斯沃西、劳伦·查林和乔厄尔·皮诺。2016年。[如何评估你的对话系统：一项关于对话响应生成的无监督评估指标的实证研究](https://doi.org/10.18653/v1/D16-1230)。发表于*2016年自然语言处理实证方法会议论文集*，第2122–2132页，美国德克萨斯州奥斯丁。计算语言学学会。

+   Liu等人（2023）杨·刘、丹·伊特尔、怡崇·徐、硕航·王、若晨·徐和成光·朱。2023年。[G-Eval：使用GPT-4进行更好的人类对齐的自然语言生成评估](http://arxiv.org/abs/2303.16634)。

+   Majumder等人（2021）博迪萨特瓦·普拉萨德·马朱姆德尔、苏达·拉奥、米歇尔·加利和朱利安·麦考利。2021年。[询问缺失和有用的内容：利用全球知识改进澄清问题生成](https://doi.org/10.18653/v1/2021.naacl-main.340)。发表于*2021年北美计算语言学学会年会：人类语言技术会议论文集*，第4300–4312页，在线。计算语言学学会。

+   OpenAI（2023）OpenAI。2023年。[GPT-4技术报告](https://arxiv.org/abs/2303.08774)。*ArXiv*，abs/2303.08774。

+   Qi等人（2020）彭·齐、宇昊·张和克里斯托弗·D·曼宁。2020年。[保持饥饿，保持专注：在信息检索对话中生成有信息性和具体性的问题](https://doi.org/10.18653/v1/2020.findings-emnlp.3)。发表于*计算语言学学会发现：EMNLP 2020*，第25–40页，在线。计算语言学学会。

+   Ramamonjison等人（2022）林德拉·拉马蒙吉森、海利·李、蒂莫西·余、石琪·何、维什努·仁甘、阿敏·班尼塔雷比-德科尔迪、子锐·周和永·张。2022年。[通过问题描述自动制定优化模型，增强运筹学](https://doi.org/10.18653/v1/2022.emnlp-industry.4)。发表于*2022年自然语言处理实证方法会议：产业轨道论文集*，第29–62页，阿布扎比，阿联酋。计算语言学学会。

+   Ramamonjison等人（2023）Rindranirina Ramamonjison, Timothy T. Yu, Raymond Li, Haley Li, Giuseppe Carenini, Bissan Ghaddar, Shiqi He, Mahdi Mostajabdaveh, Amin Banitalebi-Dehkordi, Zirui Zhou, 和 Yong Zhang. 2023. [NL4Opt competition: Formulating optimization problems based on their natural language descriptions](http://arxiv.org/abs/2303.08233).

+   Tsatsaronis等人（2015）George Tsatsaronis, Georgios Balikas, Prodromos Malakasiotis, Ioannis Partalas, Matthias Zschunke, Michael R Alvers, Dirk Weissenborn, Anastasia Krithara, Sergios Petridis, Dimitris Polychronopoulos, Yannis Almirantis, John Pavlopoulos, Nicolas Baskiotis, Patrick Gallinari, Thierry Artiéres, Axel-Cyrille Ngonga Ngomo, Norman Heino, Eric Gaussier, Liliana Barrio-Alvers, Michael Schroeder, Ion Androutsopoulos, 和 Georgios Paliouras. 2015. [An overview of the BIOASQ large-scale biomedical semantic indexing and question answering competition](https://doi.org/10.1186/s12859-015-0564-6). *BMC Bioinformatics*, 16(1):138.

+   Wei等人（2022）Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, brian ichter, Fei Xia, Ed Chi, Quoc V Le, 和 Denny Zhou. 2022. [Chain-of-thought prompting elicits reasoning in large language models](https://proceedings.neurips.cc/paper_files/paper/2022/file/9d5609613524ecf4f15af0f7b31abca4-Paper-Conference.pdf). 收录于*Advances in Neural Information Processing Systems*，第35卷，24824–24837页。Curran Associates, Inc.

+   Zhang等人（2020）Tianyi Zhang, Varsha Kishore, Felix Wu, Kilian Q. Weinberger, 和 Yoav Artzi. 2020. [BERTScore: Evaluating text generation with BERT](http://arxiv.org/abs/1904.09675).

## 附录A 生成的带有错误的摘要示例

以下生成的摘要没有包括目标函数：

> 问题描述：一家西装店生产2件套和3件套西装。制作一套2件套西装需要1小时的测量时间和4小时的缝制时间。制作一套3件套西装需要2小时的测量时间和5小时的缝制时间。该店有150小时的测量时间和300小时的缝制时间可用。为了保持营业，至少需要制作40套2件套西装。每套2件套西装的利润为600美元，每套3件套西装的利润为800美元。请建立一个线性规划模型以最大化利润。生成的摘要：好的，谢谢提供的信息！为了确保我理解无误，下面是您的情况摘要：
> 
> - 您生产两种类型的西装：2件套西装和3件套西装。
> 
> - 一套2件套西装的利润为600美元，需要1小时的测量时间和4小时的缝制时间。
> 
> - 一套3件套西装的利润为800美元，需要2小时的测量时间和5小时的缝制时间。
> 
> - 您有150小时的测量时间和300小时的缝制时间可用。
> 
> - 至少需要生产40套2件套西装，但对于3件套西装没有最低生产要求。
> 
> - 对于销售的西装数量没有限制。
> 
> 我有没有遗漏什么，或者是否还有其他特定要求或约束需要我知道的？

以下生成的摘要未提及决策变量：

> 问题陈述：一家化工公司使用两种技术——反应 X 和反应 Y——生产酸性液体和碱性液体。反应 X 每分钟可以生产 35 au 的酸性液体和 75 au 的碱性液体。反应 Y 每分钟可以生产 42 au 的酸性液体和 35 au 的碱性液体。此外，反应 X 每分钟需要 18 单位热量，而反应 Y 每分钟需要 12 单位热量。该化工公司每分钟最多能产生 1400 单位的热量。如果该公司需要每分钟至少生产 900 au 的酸性液体和 1200 au 的碱性液体，那么每种反应应该运行多少分钟，以最小化所需的总时间？生成的摘要：太好了！让我们总结一下信息：
> 
> - 你们公司有两种反应类型，X 和 Y，分别生产酸性液体和碱性液体。
> 
> - 反应 X 每分钟需要 18 单位热量，生产 35 au 的酸性液体和 75 au 的碱性液体。
> 
> - 反应 Y 每分钟需要 12 单位热量，生产 42 au 的酸性液体和 35 au 的碱性液体。
> 
> - 您公司每分钟最多能产生 1400 单位的热量。
> 
> - 每分钟需要生产至少 900 au 的酸性液体和 1200 au 的碱性液体。
> 
> - 您的目标是以最短的时间满足这些生产要求。
> 
> 我还需要了解其他信息吗？或者我遗漏了什么？

以下摘要包含一些额外的信息，虽然合理，但在原始问题描述中没有提到：

> 问题陈述：辣椒酱由辣椒和番茄的单位组成，必须包含至少 130 个香料单位和 90 个风味单位。辣椒的成本是每单位 8 美元，番茄的成本是每单位 5 美元。每单位辣椒包含 12 个香料单位和 2 个风味单位。每单位番茄包含 1 个香料单位和 8 个风味单位。将此问题形式化为线性规划，并找到能够生产最小成本辣椒酱的解。生成的摘要：感谢您的信息。根据您的回答，以下是我理解的情况：
> 
> - 您正在使用辣椒和番茄制作辣椒酱。
> 
> - 每单位辣椒的成本是 8 美元，贡献 12 个香料单位和 2 个风味单位。
> 
> - 每单位番茄的成本是 5 美元，贡献 1 个香料单位和 8 个风味单位。
> 
> - 每批辣椒酱应至少包含 130 个香料单位和 90 个风味单位。
> 
> - 对于辣椒和番茄的使用没有最大限制。
> 
> - 没有具体的预算，但您旨在最小化成本。
> 
> 我是否准确地捕捉了所有信息？

## 附录 B LLM 提示

评估提示：

[⬇](data:text/plain;base64,CllvdSBhcmUgYW4gQUkgZXZhbHVhdG9yIHNwZWNpYWxpemluZyBpbiBhc3Nlc3NpbmcgdGhlIHF1YWxpdHkgb2Ygc3VtbWFyaWVzLgpDYXJlZnVsbHkgY2hlY2sgaG93IHRoZSBzdW1tYXJ5IGNhcHR1cmVkIGEgbGluZWFyIHByb2dyYW1taW5nIHByb2JsZW0gc3RhdGVtZW50LgpJbXBvcnRhbnQgaW5mb3JtYXRpb24gZm9yIHRoaXMgdGFzayBpcyBleHBsaWNpdCBuYW1lcyBhbmQgdmFsdWVzIG9mIGRlY2lzaW9uIHZhcmlhYmxlcywgY29uc3RyYWludHMgb2YgYWxsIHR5cGVzLCBhbmQgYW4 gb2JqZWN0aXZlIGZ1bmN0aW9uLgpZb3VyIHByaW1hcnkgZ29hbCBpcyB0byByYXRlIHRoZSBzdW1tYXJ5IGJhc2VkIG9uIEluZm9ybWF0aW9uIFJlY2FsbCwgSW5mb3JtYXRpb24gUHJlY2lzaW9uLCBJbmZvcm1hdGlvbiBSZXBldGl0aW9uIGFuZCBSZWFkYWJpbGl0eS4KClRoZSBQcm9ibGVtIFN0YXRlbWVudDoKYGBgCnswfQpgYGAKClRoZSBQcm92aWRlZCBTdW1tYXJ5OgonJycKezF9CicnJwoKUFJPVklERSBUSEUgQU5TV0VSIElOIEEgSlNPTiBGT1JNQVQgV0lUSCBGT0xMT1dJTkcgRklFTERTOgoiY29ycmVjdF9pbmZvcm1hdGlvbiIgLSBzdHJpbmcgfCBpbmZvcm1hdGlvbiBhY2N1cmF0ZWx5IGNhcHR1cmVkIGluIHRoZSBzdW1tYXJ5CiJtaXNzaW5nX2luZm9ybWF0aW9uIiAtIHN0cmluZyB8IGltcG9ydGFudCBpbmZvcm1hdGlvbiBleGlzdGluZyBpbiB0aGUgb3JpZ25hbCBwcm9ibGVbXQ==-QG Agent Prompt:

[⬇](data:text/plain;base64,CllvdSBhcmUgYW4gQUkgZXZhbHVhdG9yIHNwZWNpYWxpemluZyBpbiBhc3Nlc3NpbmcgdGhlIHF1YWxpdHkgb2Ygc3VtbWFyaWVzLgpDYXJlZnVsbHkgY2hlY2sgaG93IHRoZSBzdW1tYXJ5IGNhcHR1cmVkIGEgbGluZWFyIHByb2dyYW1taW5nIHByb2JsZW0gc3RhdGVtZW50LgpJbXBvcnRhbnQgaW5mb3JtYXRpb24gZm9yIHRoaXMgdGFzayBpcyBleHBsaWNpdCBuYW1lcyBhbmQgdmFsdWVzIG9mIGRlY2lzaW9uIHZhcmlhYmxlcywgY29uc3RyYWludHMgb2YgYWxsIHR5cGVzLCBhbmQgYW4gb2JqZWN0aXZlIGZ1bmN0aW9uLgpZb3VyIHByaW1hcnkgZ29hbCBpcyB0byByYXRlIHRoZSBzdW1tYXJ5IGJhc2VkIG9uIEluZm9ybWF0aW9uIFJlY2FsbCwgSW5mb3JtYXRpb24gUHJlY2lzaW9uLCBJbmZvcm1hdGlvbiBSZXBldGl0aW9uIGFuZCBSZWFkYWJpbGl0eS4KClRoZSBQcm9ibGVtIFN0YXRlbWVudDoKYGBgCnswfQpgYGAKClRoZSBQcm92aWRlZCBTdW1tYXJ5OgonJycKezF9CicnJwoKUFJPVklERSBUSEUgQU5TV0VSIElOIEEgSlNPTiBGT1JNQVQgV0lUSCBGT0xMT1dJTkcgRklFTERTOgoiY29ycmVjdF9pbmZvcm1hdGlvbiIgLSBzdHJpbmcgfCBpbmZvcm1hdGlvbiBhY2N1cmF0ZWx5IGNhcHR1cmVkIGluIHRoZSBzdW1tYXJ5CiJtaXNzaW5nX2luZm9ybWF0aW9uIiAtIHN0cmluZyB8IGltcG9ydGFudCBpbmZvcm1hdGlvbiBleGlzdGluZyBpbiB0aGUgb3JpZ25hbCBwcm9ibGVbXQ==

[⬇](data:text/plain;base64,WU9VIEFSRSAiT3B0aU1vdXNlIiAtIEEgQ0hBVEJPVCBIRUxQSU5HIFVTRVJTIFRPIEZPUk1VTEFURSBGVUxMIE9QVElNSVpBVElPTiBQUk9CTEVNIFNUQVRFTUVOVC4KVEhFIFVTRVIgSVMgTk9UIEEgTUFUSCBFWFBFUlQgQU5EIEhBUyBOTyBFWXPFUklFTkNFIFdJVEggTUFUSCBBTkQgT1BUSU1JWkFUSU9OUy4KRE8gTk9UIFVTRSBBTlkgTUFUSEVNQVRJQ0FMIFRFUk1JTk9LTkYI

QA 代理提示：

[⬇](data:text/plain;base64,WU9VIEFSRSBBR0VOVCBJTVBFUlNPTkFUSU5HIFRIRSBCVVNJTkVTUyBPV05FUiBNRU5USU9ORUQgSU4gVEhFIFBST0JMRU0gU1RBVEVNRU5UKERFTElNSVRFRCBCWSBgYGApLgpCRSBQT0xJVEUuCllPVShUSEUgQlVTSU5FU1MgT1dORVIpIEFSRSBUQUxLSU5HIFdJVEggQU4gRVhQRVJUIElOIE9QVElNSVpBVElPTlMuCkFDQ1VSQVRFTFkgUFJPVklERSBJTkZPUk1BVElPTiBBUyBSRVFVRVNURUQgQkFTRUQgT04gVEhFIFBST0JMRU0gU1RBVEVNRU5ULgpNQUtFIFNVUkUgSU5GT1JNQVRJT04gWU9VIFBST1ZJRElORyBJUyBDT1JSRUNUIEFORCBDQU4gQkUgRk9VTkQgSU4gVEhFIFBST0JMRU0gU1RBVEVNRU5ULgpJRiBUSEUgUFJPQkxFTSBTVEFURU1FTlQgRE9FUyBOT1QgQ09OVEFJTiBSRVFVRVNURUQgSU5GT1JNQVRJT04sIFNJTVBMWSBTQVkgWU9VIERPTidUIEtOT1cgVEhFU0UgREVUQUlMUy4gKGZvciBleGFtcGxlLCAiSSdtIG5vdCBzdXJlIGFib3V0IGl0LCBjYW4gd2Ugc2tpcCB0aGlzIikKRE8gTk9UIE1BS0UgQ0FMQ1VMQVRJT05TIE9SIElORk9STUFUSU9OIE1BTklQVUxBVElORy4gVXNlIGZhY3RzIGZyb20gdGhlIHByb2JsZW0gKGZvciBleGFtcGxlLCBxdWVzdGlvbjogSG93IG1hbnkgWCBhcmUgcHJvZHVjZWQgaW4gYSBkYXk/IEFuc3dlcjogSSdtIG5vdCBzdXJlLCBidXQgSSBrbm93IHRoYXQgdG8gcHJvZHVjZSBvbmUgWCwgd2UgbmVlZCBZIG1pbnV0ZXMuKQpETyBOT1QgTUVOVElPTiBUSEUgUFJPQkxFTSBTVEFURU1FTlQgQU5ZV0hFUkU7IEFDVCBBUyBJRiBJVCBJUyBZT1VSIFBFUlNPTkFMIEtOT1dMRURHRS4KClRIRSBQUk9CB0RFTUFTB20gZXRVZXQq
