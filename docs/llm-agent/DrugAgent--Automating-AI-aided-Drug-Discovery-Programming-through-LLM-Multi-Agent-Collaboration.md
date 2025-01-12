<!--yml

分类：未分类

日期：2025-01-11 11:54:03

-->

# DrugAgent：通过LLM多智能体协作自动化AI辅助药物发现编程

> 来源：[https://arxiv.org/html/2411.15692/](https://arxiv.org/html/2411.15692/)

Sizhe Liu¹, Yizhou Lu¹, Siyu Chen¹, Xiyang Hu², Jieyu Zhao¹, Tianfan Fu³, Yue Zhao¹¹¹1通讯作者

###### 摘要

大型语言模型（LLMs）的最新进展为加速药物发现过程开辟了新的途径。尽管它们具有潜力，但仍然存在若干关键挑战尚未解决，特别是在将理论理念转化为药物研究这一高度专业化领域中的实际应用时，这限制了从业人员在药物发现中利用最新AI发展的能力。为此，我们介绍了DrugAgent，一个旨在自动化药物发现中的机器学习（ML）编程的多智能体框架。DrugAgent通过识别特定需求并构建领域特定工具来融入领域专业知识，同时系统地探索不同的思路，寻找有效的解决方案。一项初步的案例研究展示了DrugAgent克服LLMs在药物发现中面临的关键局限性的潜力，朝着AI驱动的创新迈进。例如，DrugAgent能够完成从数据获取到性能评估的端到端ML编程流程，用于ADMET预测任务，并最终选择最佳模型，其中随机森林模型在使用PAMPA数据集预测吸收时取得了0.92的F1分数。

## 1 引言

人工智能（AI）正在推动药物发现领域的重大进展（Huang et al. [2022](https://arxiv.org/html/2411.15692v1#bib.bib11)）。由于实验评估药物性质所需的高成本和时间，研究人员正日益寻求加速药物开发各个阶段的方法（Pushpakom et al. [2019](https://arxiv.org/html/2411.15692v1#bib.bib27)）。目前，许多AI准备好的数据集和基准已经可用于药物发现过程中的关键任务，如ADMET预测、药物-靶点相互作用和高通量筛选（Huang et al. [2021](https://arxiv.org/html/2411.15692v1#bib.bib10); Chen et al. [2024](https://arxiv.org/html/2411.15692v1#bib.bib2); Wang et al. [2024c](https://arxiv.org/html/2411.15692v1#bib.bib38)）。深度学习的最新进展在加速前期优化和预测药物-靶点相互作用方面展现了特别的潜力（Huang et al. [2020](https://arxiv.org/html/2411.15692v1#bib.bib12)），有望减少传统实验方法所需的时间和资源。

在药物发现中进行机器学习（ML）实验需要生物学、化学、药学和计算机科学方面的专业知识，这为入门者设置了显著的障碍。大型语言模型（LLMs）凭借其在复杂任务中的推理能力，为自动化药物发现过程中的ML编程提供了一个令人兴奋的机会。通用框架，如MLAgentBench（Huang et al. [2024a](https://arxiv.org/html/2411.15692v1#bib.bib14)）和AI-Scientist（Lu et al. [2024a](https://arxiv.org/html/2411.15692v1#bib.bib22)），为端到端的ML编程提供了有前景的解决方案。具有领域特定工具的专业化代理可以进一步增强处理化学或生物学中复杂任务的能力（Boiko et al. [2023](https://arxiv.org/html/2411.15692v1#bib.bib1)；M. Bran et al. [2024](https://arxiv.org/html/2411.15692v1#bib.bib25)；Inaba et al. [2023](https://arxiv.org/html/2411.15692v1#bib.bib16)）。尽管如此，仍然存在显著的挑战，阻碍了使用LLMs完全自动化药物发现研究。

挑战 1. 通用型LLMs通常缺乏在药物发现中准确实施ML实验所需的专业领域知识。例如，领域特定库的不正确API选择或对原始生物数据预处理步骤的误解，很容易导致问题，这些问题往往难以调试，特别是在药物发现任务中通常涉及的复杂代码库下。虽然像ChemCrown（M. Bran et al. [2024](https://arxiv.org/html/2411.15692v1#bib.bib25)）和MultiTool-CoT（思维链，Chain of Thought）（Inaba et al. [2023](https://arxiv.org/html/2411.15692v1#bib.bib16)）这样的框架提供了用于化学任务的工具，例如计算分子量和预测反应，但它们并未完全解决这一问题。这些工具通常过于简单，无法满足ML编程的需求，表明需要一个更广泛的工具集，从数据收集到模型评估。

挑战 2。在许多机器学习任务中，LLM（大型语言模型）需要生成创意，而不仅仅是执行预定义的计划。然而，LLM生成的创意往往缺乏实际背景的支撑（Si, Yang, 和 Hashimoto [2024](https://arxiv.org/html/2411.15692v1#bib.bib32)），尤其是在药物发现的背景下。由于幻觉问题，LLM可能会自信地提出一个创意，但却缺乏实施该创意所需的领域知识（Huang et al. [2023](https://arxiv.org/html/2411.15692v1#bib.bib13)）。现有的探索可行创意的策略，如推理与行动（Huang et al. [2024a](https://arxiv.org/html/2411.15692v1#bib.bib14)）、生成多样化创意（Lu et al. [2024a](https://arxiv.org/html/2411.15692v1#bib.bib22); Wang et al. [2024a](https://arxiv.org/html/2411.15692v1#bib.bib35)）、或使用树搜索（WecoAI [2024](https://arxiv.org/html/2411.15692v1#bib.bib40)），通常是针对标准机器学习任务优化的，在许多提出的创意无法实施的情况下可能效率低下。因此，亟需一种能够在此基础上发展的方法，使得智能体的创意探索与其实际知识更好地对接。

我们的解决方案。为了应对这些挑战，我们提出了DrugAgent，一个多智能体框架，用于增强药物发现任务中的机器学习编程。首先，我们整合了能够识别需要领域知识的步骤的工作流程，从而开发出专门的工具来处理这些任务，再进行编码。除此之外，我们引入了一种动态的创意空间管理方法，在早期生成多样化的创意，并根据实验观察进行更新，从而实现更高效的探索。最后，我们提供了一套增强的工具，以综合性的库文档形式支持关键的基于人工智能的药物发现任务，包括生物数据检索、分子指纹、人工智能模型开发和性能评估。这些资源经过精心挑选，以满足现实世界编程过程中的复杂需求。

主要贡献。我们的主要贡献包括：

+   •

    重要性。本文聚焦于自动化基于人工智能的药物发现任务，这是一个关乎生命且意义重大的问题。据我们所知，这是首次尝试在药物发现的背景下自动化人工智能编程。我们的工作使得没有编程背景的药学科学家能够使用人工智能，并促进了基于人工智能的药物发现研究。

+   •

    方法。我们设计了一个基于大型语言模型（LLM）的自动化多智能体系统，该系统涉及专门针对药物发现的编程，并且能够自动运行代码并收集结果，无需人工干预。

+   •

    结果。DrugAgent在自动化几个具有代表性的基于AI的药物发现任务中表现出初步成功。例如，DrugAgent可以自动构建一个随机森林模型来预测药物分子的吸收，且在PAMPA数据集上获得了0.920的F1分数。

## 2 相关工作

### 2.1 LLM代理

LLM代理是一个利用大型语言模型与用户或其他系统进行交互、执行任务并自主做出决策的系统。在LLM的支持下，LLM代理能够执行多步骤推理、规划和行动执行，超越静态文本生成（Wang et al. [2024b](https://arxiv.org/html/2411.15692v1#bib.bib36)）。之前的研究为LLM代理配备了与外部工具动态交互的模块，可以检索信息并根据实时反馈进行适应（Schick et al. [2023](https://arxiv.org/html/2411.15692v1#bib.bib31); Yoon, Kim, and Oh [2024](https://arxiv.org/html/2411.15692v1#bib.bib44); Qin et al. [2023](https://arxiv.org/html/2411.15692v1#bib.bib28); Ravuru, Sakhinana, and Runkana [2024](https://arxiv.org/html/2411.15692v1#bib.bib29); Lála et al. [2023](https://arxiv.org/html/2411.15692v1#bib.bib24)）。这使得它们能够解决复杂的、不断变化的任务，如代码编写、长期推理和在不同环境中的决策（Guo et al. [2024](https://arxiv.org/html/2411.15692v1#bib.bib9); Jiang et al. [2024](https://arxiv.org/html/2411.15692v1#bib.bib17)）。在本研究中，我们将LLM多代理框架定制化应用于药物发现任务。

### 2.2 LLM在机器学习编程中的应用

最近的研究工作集中于通过自动化机器学习编程来加速传统的手工研究过程。AIDE作为一个数据科学代理，探索广泛的解空间，并通过反复迭代优化其方法以达到最佳解决方案（WecoAI [2024](https://arxiv.org/html/2411.15692v1#bib.bib40)）。AutoKaggle为Kaggle数据科学竞赛引入了一个专门的多代理框架（Li et al. [2024b](https://arxiv.org/html/2411.15692v1#bib.bib20)）。AI-Scientist使LLM能够自主开展研究，从创意生成到论文草拟，专注于与机器学习相关的主题（Lu et al. [2024a](https://arxiv.org/html/2411.15692v1#bib.bib22)）。同时，也开发了一些基准测试，提供了一套13个任务来评估LLM在进行机器学习编程中的能力（Huang et al. [2024a](https://arxiv.org/html/2411.15692v1#bib.bib14)）。然而，现有的工作无法处理需要复杂领域知识的特定领域机器学习任务，例如AI辅助药物发现。为了解决这个问题，我们设计了工作流，以便自动插入领域知识并调用特定领域的工具。

### 2.3 LLM在生物医学发现中的应用

许多研究强调了大语言模型（LLMs）在生物医学发现中的应用，尤其是与领域特定工具相结合时。例如，ChemCrown展示了LLM智能体在有机合成、药物发现和材料设计中的潜力（M. Bran等人 [2024](https://arxiv.org/html/2411.15692v1#bib.bib25)）。类似地，MMedAgent是一个多模态医学智能体，旨在处理复杂的语言和多模态任务，展示了LLM在医学应用中的多功能性（Li等人 [2024a](https://arxiv.org/html/2411.15692v1#bib.bib19)）。多智能体方法通过ClinicalAgent（Yue等人 [2024](https://arxiv.org/html/2411.15692v1#bib.bib45)）得到了体现，该方法通过将临床试验结果预测任务分解为子问题，允许各个智能体协作生成综合结果。然而，现有的机器学习（ML）编程智能体可能缺乏生物医学任务所需的领域特定知识，而生物医学智能体通常不具备机器学习特定的专业知识。为了弥补这一差距，我们提出了DrugAgent，这是一个多智能体LLM系统，它将机器学习编程能力与生物医学知识相结合，针对药物发现中的机器学习任务的独特需求。

## 3 方法论

![参见说明](img/69bb45930efca4e8a86f28e72896110e.png)

图1：DrugAgent框架概述。给定一个基于AI的药物发现任务，该任务用自然语言描述（即用户输入，例如，设计一个AI模型来预测吸收（ADMET属性之一），使用PAMPA数据集（Siramshetty, Shah等人 [2021](https://arxiv.org/html/2411.15692v1#bib.bib33)）），LLM规划器首先提出几个潜在的想法（例如，GCN（图卷积网络）（Kipf和Welling [2016](https://arxiv.org/html/2411.15692v1#bib.bib18)），随机森林，预训练模型（如ChemBERTa（Chithrananda, Grand和Ramsundar [2020](https://arxiv.org/html/2411.15692v1#bib.bib5)）））。然后，对于每个想法，LLM指导者基于领域知识将该想法转化为代码（例如，数据集获取和分子指纹提取）。接着，编码器调试并实现代码，并评估其性能。最后，收集所有结果并报告最佳想法（例如，随机森林在预测吸收方面表现最佳）。

我们介绍了DrugAgent，一个自动化且创新的LLM多智能体框架，旨在简化AI辅助药物发现任务。如图[1](https://arxiv.org/html/2411.15692v1#S3.F1 "图1 ‣ 3 方法论 ‣ DrugAgent：通过LLM多智能体协作自动化AI辅助药物发现编程")所示，DrugAgent集成了两个关键组件：LLM指导者（§[3.2](https://arxiv.org/html/2411.15692v1#S3.SS2 "3.2 LLM指导者：领域特定知识识别和工具准备 ‣ 3 方法论 ‣ DrugAgent：通过LLM多智能体协作自动化AI辅助药物发现编程")），负责识别领域特定的知识需求并准备必要的工具，以及LLM规划者（§[3.3](https://arxiv.org/html/2411.15692v1#S3.SS3 "3.3 LLM规划者：创意空间管理 ‣ 3 方法论 ‣ DrugAgent：通过LLM多智能体协作自动化AI辅助药物发现编程")），负责管理和完善创意探索，以优化任务执行效果。在详细介绍这些组件及其角色之前，我们将在§[3.1](https://arxiv.org/html/2411.15692v1#S3.SS1 "3.1 问题表述 ‣ 3 方法论 ‣ DrugAgent：通过LLM多智能体协作自动化AI辅助药物发现编程")中定义问题。

### 3.1 问题表述

我们解决了药物发现领域中自动化机器学习（ML）编程任务的挑战。这些任务涉及将自然语言指令与计算工具结合，以产生准确且高效的解决方案。参考Huang等人（[2024a](https://arxiv.org/html/2411.15692v1#bib.bib14)），机器学习编程任务由以下组件定义：

+   •

    任务描述：一份自然语言规格，概述了任务的目标和约束。

+   •

    启动文件：一组初始资源，如数据集或代码模板，用于支持任务执行。

+   •

    评估器：一个性能度量函数，用于评估任务输出的质量。

一个智能体必须解读任务描述，利用启动文件，并执行一系列动作来生成解决方案。这些动作包括读取和写入文件、预处理数据、实现机器学习模型以及执行Python程序。主要的挑战在于将抽象的任务描述与实际的执行对接，特别是当任务需要领域特定知识时。我们的目标是开发一个能够高效处理这些任务的自治系统，同时最小化错误并提高成功率。

### 3.2 LLM指导者：领域特定知识识别和工具准备

#### 动机。

药物发现是一个高度专业化且复杂的领域，要求精确地整合机器学习与领域专业知识。尽管使用LLMs在该领域提供了自动化和加速机器学习编程的巨大潜力，但我们观察到LLMs常常未能弥合通用推理与药物发现任务具体需求之间的差距。这种失败源于幻觉（Huang等人 [2023](https://arxiv.org/html/2411.15692v1#bib.bib13)，[2024b](https://arxiv.org/html/2411.15692v1#bib.bib15)），LLMs由于缺乏对特定领域需求的理解，产生了不正确或不现实的输出。例如，不当的SMILES字符串预处理或不正确的API使用，可能导致昂贵的调试和失败的实验。这些局限性突显了在进行实验之前，明确识别和解决领域特定知识需求的紧迫性。为了解决这一问题，我们在DrugAgent中引入了LLM教练，它遵循一个结构化的流程：

1.  1.

    分解问题：将问题拆解成更小的、可操作的子步骤，以便系统地解决（Wu等人 [2024](https://arxiv.org/html/2411.15692v1#bib.bib41); Huang等人 [2024a](https://arxiv.org/html/2411.15692v1#bib.bib14)）。

1.  2.

    确定知识需求：分析子步骤，以确定是否需要特定领域的专业知识或工具，使用专家策划的提示语。

1.  3.

    构建工具：通过识别相关的API并通过单元测试验证它们来收集或创建工具。

1.  4.

    复用工具：将经过验证的工具添加到可重用的工具箱中，以提高效率并减少未来任务中的错误。

每一步都至关重要，能够使LLM教练弥合通用推理与特定领域需求之间的差距。以下部分将提供更多细节，说明如何识别领域特定知识、构建工具以及处理失败，以确保在药物发现中的机器学习任务能够有效执行。

#### 特定领域知识。

特定领域知识是指与特定领域或学科相关的专门信息、概念和专业知识，例如我们语境中的药物发现。在药物发现的机器学习（ML）任务中，缺乏或不完整的领域特定知识通常会导致编码错误。我们观察到，LLMs常常由于幻觉问题，未能识别在某些任务中需要领域特定知识，从而导致必要工具的错误使用。因此，显式的推理过程至关重要。在开始实验之前，收集所有相关的领域特定知识和工具，对于最小化错误并确保实验与该领域的复杂性保持一致至关重要。

#### 教练。

导师代理负责识别需要领域特定知识的子步骤。该过程首先通过将总体计划分解为一系列可操作的简单步骤开始，这种方法已被证明在处理复杂任务时有效，例如机器学习编程（Wu 等 [2024](https://arxiv.org/html/2411.15692v1#bib.bib41)；Huang 等 [2024a](https://arxiv.org/html/2411.15692v1#bib.bib14)）。接下来，导师分析这些步骤中哪些需要领域专业知识。为了提高识别的准确性，我们使用由药物发现领域专家精心设计的少量示例提示。虽然这种方法不能保证所有子步骤都能被正确识别，但我们的分析表明，它在大多数情况下表现成功。

#### 领域工具构建。

对于每个识别出的领域特定需求，我们接着收集适当的工具。在编程任务中，创建一个固定的工具列表，如先前的生物医学代理所见（Roohani 等 [2024](https://arxiv.org/html/2411.15692v1#bib.bib30)；M. Bran 等 [2024](https://arxiv.org/html/2411.15692v1#bib.bib25)），由于库中 API 数量庞大，这一任务非常具有挑战性。因此，我们会通过查阅文档来识别相关的 API 并创建工具，这可能涉及单个 API 或多个 API 组合成的辅助函数。然而，仅仅依赖文档可能会引入错误，尤其是在文档过时或缺乏足够细节的情况下。此外，机器学习问题通常需要将多个 API 以复杂的方式结合起来的辅助函数，从而增加了出错的机会。为了解决这个问题，编码者首先设计单元测试来验证构建工具的正确性，从而最小化错误传播到后续阶段的风险。然后，编码者访问相关的库文档来完成工具构建。

#### 工具的可重用性与故障处理。

对于通过单元测试的工具，我们将其添加到工具箱中以供未来使用。先前的研究已展示了构建不断扩展的工具箱的好处（Wang、Fried 和 Neubig [2024](https://arxiv.org/html/2411.15692v1#bib.bib39)）。在我们的案例中，由于许多任务依赖于共享的领域知识，例如数据获取，创建可重用的函数可以帮助降低成本并减少错误。在药物发现任务中，代理在尝试构建特定领域的工具时常常面临挑战，即使有文档支持。当在单元测试中无法通过反复调试解决问题时，我们会记录这一结果并将其报告给规划代理。这个过程将在下一节中进一步解释。

|  | ADMET 预测 | DTI 预测 | 分子优化 |
| --- | --- | --- | --- |
| 类型 | 单实例预测 | 多实例预测 | 生成 |
| 输入 | SMILES 字符串 | SMILES 字符串和蛋白质氨基酸序列 | SMILES 字符串 |
| 影响 | 通过早期和准确的 ADMET 描述预防临床试验失败 | 减少高通量筛选需求并缩小搜索空间 | 实现高效设计具有理想药物性质的分子 |
| 数据示例 | Caco-2 (Wang et al. [2016](https://arxiv.org/html/2411.15692v1#bib.bib37)) | DAVIS (Davis et al. [2011](https://arxiv.org/html/2411.15692v1#bib.bib6)) | ZINC (Sterling 和 Irwin [2015](https://arxiv.org/html/2411.15692v1#bib.bib34)) |

表 1：任务概述：ADMET、DTI 和分子优化。本文仅关注小分子药物，这些药物占所有批准药物的 90%以上。小分子药物可以表示为 SMILES 字符串。SMILES 字符串是一种用简短 ASCII 字符串描述化学化合物（例如药物分子）的行表示法。

![参见标题](img/3ec17b843cd099bf50edfe7c681d007a.png)

图 2：在使用 PAMPA 数据集进行 ADMET 预测任务时，ReAct (a) 与 DrugAgent (b) 的对比。ReAct 是一个通用框架，由于产生幻觉的 API 调用和无法自我调试，导致失败，必须通过人工干预才能继续。它仅专注于微调预训练的语言模型，但由于数据集规模较小，这种方法并不理想。相比之下，DrugAgent 系统地探索了多种方法，包括随机森林、图神经网络和预训练的语言模型。DrugAgent 确定了领域特定的需求，构建了必要的工具，并修剪了无效的想法，如分子图构建。这种结构化的工作流程使得 DrugAgent 能够自主地提供成功的结果，取得了强劲的表现。更多分析请见 §[4.3](https://arxiv.org/html/2411.15692v1#S4.SS3 "4.3 案例研究：比较 DrugAgent 与 ReAct 在 ADMET 预测任务中的表现 ‣ 4 实验 ‣ DrugAgent：通过 LLM 多智能体协作自动化 AI 辅助药物发现编程") 和附录中的示例代码。

### 3.3 LLM 规划器：思维空间管理

#### 动机。

药物发现任务本质上是开放性问题，没有单一的确定性解决方案。方法往往会根据可用数据、领域要求和任务约束而有所不同。虽然 LLM 可以生成多个想法，但它们通常难以区分可行与不可行的解决方案，原因在于幻觉或领域知识不足（Huang et al. [2024b](https://arxiv.org/html/2411.15692v1#bib.bib15)）。这种低效性可能导致计算资源的浪费和表现不佳。为了解决这一问题，DrugAgent 中的 LLM 规划器旨在系统地管理和完善思维空间，确保提出可执行且高效的解决方案。

#### 思维空间。

“创意空间”涵盖了给定机器学习任务的广泛潜在方法或解决方案，认识到这类任务本质上是开放性的，且缺乏单一的确定性解决方案。设$M$为任务的所有可能创意的集合，设$N \subseteq M$为基于LLM所掌握知识可实施的创意子集。主要目标是识别一个创意$I \in N$，它能有效且高效地最大化性能指标。

#### 规划器的合理性。

尽管LLM能够生成多样的创意，但它们常常难以将这些建议与可实施的子集$N$对齐，特别是在像药物发现这样的领域特定任务中。这种不对齐主要是由于LLM的幻觉倾向，它们会提出不现实或不可行的创意，而不考虑实现约束（Huang et al. [2024b](https://arxiv.org/html/2411.15692v1#bib.bib15)）。为了解决这个问题，我们引入了一种机制，通过使用来自编程观察的反馈，迭代地完善创意空间。通过追踪工具构建或数据预处理等任务的成功与失败，规划器可以从过去的尝试中学习，以改进其搜索过程，并专注于可执行的解决方案。

#### 规划器。

规划器分为两个关键阶段：创意生成和创意完善。在创意初始化阶段，规划器根据问题陈述生成$K$个候选创意。在完善阶段，规划器使用观察结果，例如工具故障或实验结果，来调整创意集。这个过程涉及三个核心操作：(1) 删除不可行的创意，(2) 修改现有创意以解决已识别的局限性，或者(3) 基于累积的知识引入新创意。

如图[1](https://arxiv.org/html/2411.15692v1#S3.F1 "图 1 ‣ 3 方法论 ‣ DrugAgent：通过LLM多智能体协作自动化AI辅助药物发现编程")所示，当规划器在构建领域特定知识的工具时遇到故障时，故障会被记录下来，并且相关的创意会被标记为不可行。然后，规划器会停止进一步探索该创意，并删除依赖于相同缺失知识的其他创意。这个迭代过程不仅将努力重新引导向可行的解决方案，还为未来的创意生成提供反馈，减少重复错误的可能性，并提高系统的整体效率。

## 4 实验

### 4.1 基于AI的药物发现任务

我们提出了三个具有代表性的AI可解药物发现任务，以验证DrugAgent的有效性，如表[1](https://arxiv.org/html/2411.15692v1#S3.T1 "Table 1 ‣ Tool Reusability and Failure Handling. ‣ 3.2 LLM Instructor: Domain-Specific Knowledge Identification and Tool Preparation ‣ 3 Methodology ‣ DrugAgent: Automating AI-aided Drug Discovery Programming through LLM Multi-Agent Collaboration")所示。这些任务是经过充分验证的基准任务，涵盖了治疗数据公共平台（TDC）基准中的三类基本任务：单实例预测、多实例预测和生成任务（Huang等人 [2021](https://arxiv.org/html/2411.15692v1#bib.bib10)）。

1.  1.

    ADMET预测。ADMET（吸收、分布、代谢、排泄和毒性）预测是一个单实例预测任务，目标是根据药物的结构预测药物的药代动力学性质。这些性质对药物的疗效、安全性和临床成功至关重要，因此早期的ADMET评估对于减少晚期失败风险至关重要（Niu等人 [2024](https://arxiv.org/html/2411.15692v1#bib.bib26); Lu等人 [2024b](https://arxiv.org/html/2411.15692v1#bib.bib23); Chen等人 [2021](https://arxiv.org/html/2411.15692v1#bib.bib3); Chen、Hao和Van Rechem [2024](https://arxiv.org/html/2411.15692v1#bib.bib4)）。

1.  2.

    药物-靶点相互作用（DTI）。DTI预测是一个多实例预测任务，旨在预测药物与靶标蛋白之间的结合亲和力，基于小分子化合物结构和蛋白质氨基酸序列进行预测。该任务对虚拟筛选、药物再利用和副作用预测至关重要（Liu等人 [2024](https://arxiv.org/html/2411.15692v1#bib.bib21); Zhang等人 [2021](https://arxiv.org/html/2411.15692v1#bib.bib46)）。

1.  3.

    分子优化。分子优化侧重于生成具有理想药物性质的新颖多样的分子，使其成为一个生成任务（Xia等人 [2024](https://arxiv.org/html/2411.15692v1#bib.bib42); Fu等人 [2022](https://arxiv.org/html/2411.15692v1#bib.bib7)）。通过使用针对性设计方法，该方法减少了对广泛搜索的需求，提高了效率和创新性（Gao等人 [2022](https://arxiv.org/html/2411.15692v1#bib.bib8)）。

### 4.2 基准方法

我们将DrugAgent与两种已确立的基准方法进行比较，以评估其在提出任务中的表现：

1.  1.

    ReAct。ReAct（Yao等人 [2023](https://arxiv.org/html/2411.15692v1#bib.bib43)）使得LLMs能够通过交织的上下文方法集成推理和行动，允许互动分析观察到的信息并执行相应的行动。

1.  2.

    MLAgentBench。研究代理（Huang等人 [2024a](https://arxiv.org/html/2411.15692v1#bib.bib14)）支持执行如维护研究计划、理解文件、编辑脚本以及反思任务进展等任务。

### 4.3 案例研究：DrugAgent与ReAct在ADMET预测任务上的比较

为了展示DrugAgent的有效性，我们对ADMET预测任务进行了案例研究，并将其与ReAct进行了比较，具体如图[2](https://arxiv.org/html/2411.15692v1#S3.F2 "Figure 2 ‣ Tool Reusability and Failure Handling. ‣ 3.2 LLM Instructor: Domain-Specific Knowledge Identification and Tool Preparation ‣ 3 Methodology ‣ DrugAgent: Automating AI-aided Drug Discovery Programming through LLM Multi-Agent Collaboration")所示。此次比较突显了LLM在处理领域特定任务时面临的挑战，以及DrugAgent在克服这些局限性方面的优势。

ReAct（Yao et al. [2023](https://arxiv.org/html/2411.15692v1#bib.bib43)），一个通用框架，在领域特定知识的整合上存在困难。例如，它首先建议对预训练的语言模型进行微调，但在关键步骤上失败，如下载适当的数据集或选择正确的API，迫使人工干预才能继续。此外，ReAct仅专注于优化单一方法，而考虑到数据集的规模较小，这种方法对于该任务并不理想。这些局限性揭示了通用LLM推理与药物发现任务专门需求之间的差距。

相比之下，DrugAgent采用了一种系统化和多方面的方法。它探索了多种方法，包括随机森林、图神经网络（GNN）和预训练的语言模型，同时识别出需要领域知识的步骤。例如，DrugAgent成功自动化了如数据集下载、分子指纹生成和ChemBERTa（Chithrananda, Grand, and Ramsundar [2020](https://arxiv.org/html/2411.15692v1#bib.bib5)）标记化/模型执行等任务。此外，DrugAgent采用了思想修剪技术，去除验证失败的方法，如GNN输入的分子图构建，从而节省了时间和计算资源。

从性能角度来看，DrugAgent在多个模型中都表现出色。随机森林方法达到了0.920的F1分数和0.817的ROC-AUC，而ChemBERTa则达到了0.916的F1分数和0.776的ROC-AUC。这些结果强调了DrugAgent不仅能够自动化领域特定的机器学习任务，还能为当前问题选择并优化最有效的方法。

## 5 结论

在本文中，我们介绍了DrugAgent，一个多智能体框架，代表了在药物发现的关键方面自动化应用大语言模型的一个重要进展。DrugAgent解决了该领域固有的关键挑战，包括通用LLM无法处理领域特定要求、创意空间探索效率低下，以及缺乏强大的领域特定工具。通过系统地生成和精炼创意，DrugAgent确保探索过程既高效又符合药物发现任务的实际约束。此外，集成了专业化工具集，如数据集处理、分子指纹提取和分词工作流，使DrugAgent能够弥合通用AI能力与制药研究复杂需求之间的鸿沟。通过概念验证实验，我们展示了DrugAgent通过有效地自动化复杂任务并识别最优解决方案，优于像ReAct这样的通用框架。

需要注意的是，这项工作代表了推动AI驱动的药物发现边界的持续努力。随着这一领域的发展，DrugAgent也将迎来更多改进和扩展的机会，以确保其在应对这一动态领域挑战中的持续相关性和影响力。

## 6 未来工作

由于这是一个初步版本，我们的工作在某些方面仍需进一步探索。首先，我们计划通过引入更多最先进的基准并进行大规模定量比较来扩展实验，严格评估DrugAgent在各种药物发现任务中的性能和可扩展性。这将包括在更具挑战性的数据集和任务上进行测试，以验证我们框架的通用性。

其次，我们旨在进行全面的消融研究，以更好地理解各个模块的贡献，例如领域知识识别步骤、创意生成与修剪过程，以及增强工具集的有效性。这些研究将帮助隔离并量化每个组件的影响，提供对DrugAgent的优势和潜在局限性的更深入洞察。

最后，我们计划探索DrugAgent与真实世界药物发现工作流的集成，和领域专家合作评估其实际效用并识别改进的空间。这将使我们确保DrugAgent不仅是一个理论上的进展，也是一个能够实质性加速药物发现流程的实用工具。

## 参考文献

+   Boiko等（2023）Boiko, D. A.; MacKnight, R.; Kline, B.; 和 Gomes, G. 2023. 使用大语言模型进行自主化学研究。*Nature*，624(7992): 570–578。

+   Chen 等人 (2024) Chen, J.; Hu, Y.; Wang, Y.; Cao, X.; Lin, M.; Xu, H.; Wu, J.; Xiao, C.; Sun, J.; 等人 2024. TrialBench：多模态人工智能就绪临床试验数据集。*arXiv:2407.00631*。

+   Chen 等人 (2021) Chen, L.; Lu, Y.; Wu, C.-T.; Clarke, R.; Yu, G.; Van Eyk, J. E.; Herrington, D. M.; 和 Wang, Y. 2021. 数据驱动的亚型特异性差异表达基因检测。*科学报告*，11(1): 332。

+   Chen, Hao, 和 Van Rechem (2024) Chen, T.; Hao, N.; 和 Van Rechem, C. 2024. 临床试验结果预测的不确定性量化。arXiv:2401.03482.

+   Chithrananda, Grand 和 Ramsundar (2020) Chithrananda, S.; Grand, G.; 和 Ramsundar, B. 2020. ChemBERTa: 用于分子属性预测的大规模自监督预训练。在 *2020年NeurIPS机器学习分子研讨会*。

+   Davis 等人 (2011) Davis, M. I.; Hunt, J. P.; Herrgard, S.; Ciceri, P.; Wodicka, L. M.; Pallares, G.; Hocker, M.; Treiber, D. K.; 和 Zarrinkar, P. P. 2011. 激酶抑制剂选择性的综合分析。*自然生物技术*，29(11): 1046–1051。

+   Fu 等人 (2022) Fu, T.; Gao, W.; Xiao, C.; Yasonik, J.; Coley, C. W.; 和 Sun, J. 2022. 用于分子优化的可微分支架树。*国际学习表征会议*.

+   Gao 等人 (2022) Gao, W.; Fu, T.; Sun, J.; 和 Coley, C. 2022. 样本效率至关重要：实用分子优化基准。*神经信息处理系统进展*，35: 21342–21357。

+   Guo 等人 (2024) Guo, T.; Chen, X.; Wang, Y.; Chang, R.; Pei, S.; Chawla, N. V.; Wiest, O.; 和 Zhang, X. 2024. 基于大语言模型的多智能体：进展与挑战的综述。arXiv:2402.01680.

+   Huang 等人 (2021) Huang, K.; Fu, T.; Gao, W.; Zhao, Y.; Roohani, Y.; Leskovec, J.; Coley, C. W.; Xiao, C.; Sun, J.; 和 Zitnik, M. 2021. 药物发现与开发的机器学习数据集与任务：治疗数据公共库。*神经信息处理系统进展*。

+   Huang 等人 (2022) Huang, K.; Fu, T.; Gao, W.; Zhao, Y.; Roohani, Y.; Leskovec, J.; Coley, C. W.; Xiao, C.; Sun, J.; 和 Zitnik, M. 2022. 治疗科学的人工智能基础。*自然化学生物学*，18: 1033.

+   Huang 等人 (2020) Huang, K.; Fu, T.; Glass, L. M.; Zitnik, M.; Xiao, C.; 和 Sun, J. 2020. DeepPurpose：用于药物-靶标相互作用预测的深度学习库。*生物信息学*，36(22-23): 5545–5547。

+   Huang 等人 (2023) Huang, L.; Yu, W.; Ma, W.; Zhong, W.; Feng, Z.; Wang, H.; Chen, Q.; Peng, W.; Feng, X.; Qin, B.; 和 Liu, T. 2023. 大型语言模型中的幻觉：原理、分类、挑战与未解之问的综述。*arXiv预印本 arXiv:2311.05232*。正在进行的工作；49页。

+   Huang 等人 (2024a) Huang, Q.; Vora, J.; Liang, P.; 和 Leskovec, J. 2024a. MLAgentBench：评估语言代理在机器学习实验中的表现。在 *第三十八届神经信息处理系统会议*。

+   Huang等（2024b）Huang, Y.; Sun, L.; Wang, H.; Wu, S.; Zhang, Q.; Li, Y.; Gao, C.; Huang, Y.; Lyu, W.; Zhang, Y.; 等 2024b. Position: Trustllm：大型语言模型的可信度。在*国际机器学习大会*，20166–20270。PMLR。

+   Inaba等（2023）Inaba, T.; Kiyomaru, H.; Cheng, F.; 和 Kurohashi, S. 2023. MultiTool-CoT: GPT-3可以使用多个外部工具与思维链提示进行联动。在*第61届计算语言学协会年会论文集（第2卷：短篇论文）*，1522–1532， 加拿大多伦多：计算语言学协会。

+   Jiang等（2024）Jiang, J.; Wang, F.; Shen, J.; Kim, S.; 和 Kim, S. 2024. 大型语言模型在代码生成中的应用调研。*arXiv预印本 arXiv:2406.00515*。

+   Kipf和Welling（2016）Kipf, T. N.; 和 Welling, M. 2016. 基于图卷积网络的半监督分类。*国际学习表征会议（ICLR）*。

+   Li等（2024a）Li, B.; Yan, T.; Pan, Y.; Luo, J.; Ji, R.; Ding, J.; Xu, Z.; Liu, S.; Dong, H.; Lin, Z.; 和 Wang, Y. 2024a. MMedAgent：通过多模态智能体学习使用医学工具。*arXiv预印本 arXiv:2407.02483*。已被EMNLP 2024接收。

+   Li等（2024b）Li, Z.; Zang, Q.; Ma, D.; Guo, J.; Zheng, T.; Liu, M.; Niu, X.; Wang, Y.; Yang, J.; Liu, J.; Zhong, W.; Zhou, W.; Huang, W.; 和 Zhang, G. 2024b. AutoKaggle：用于自主数据科学竞赛的多智能体框架。*arXiv预印本 arXiv:2410.20424*。

+   Liu等（2024）Liu, S.; Xia, J.; Zhang, L.; Liu, Y.; Liu, Y.; Du, W.; Gao, Z.; Hu, B.; Tan, C.; Xiang, H.; 和 Li, S. Z. 2024. FlexMol：用于分子关系学习基准测试的灵活工具包。在*第38届神经信息处理系统会议（NeurIPS）*。

+   Lu等（2024a）Lu, C.; Lu, C.; Lange, R. T.; Foerster, J.; Clune, J.; 和 Ha, D. 2024a. AI科学家：朝着完全自动化的开放式科学发现迈进。*arXiv预印本 arXiv:2408.06292*。

+   Lu等（2024b）Lu, Y.; Chen, T.; Hao, N.; Van Rechem, C.; Chen, J.; 和 Fu, T. 2024b. 临床试验批准预测的不确定性量化与可解释性。*健康数据科学*，4: 0126。

+   Lála等（2023）Lála, J.; O'Donoghue, O.; Shtedritski, A.; Cox, S.; Rodriques, S. G.; 和 White, A. D. 2023. PaperQA：用于科学研究的检索增强生成代理。*arXiv预印本 arXiv:2312.07559*。

+   M. Bran等（2024）M. Bran, A.; Cox, S.; Schilter, O.; Baldassari, C.; White, A. D.; 和 Schwaller, P. 2024. 使用化学工具增强大型语言模型。*自然机器智能*，6(5): 525–535。

+   Niu等（2024）Niu, Z.; Xiao, X.; Wu, W.; Cai, Q.; Jiang, Y.; Jin, W.; Wang, M.; Yang, G.; Kong, L.; Jin, X.; Yang, G.; 和 Chen, H. 2024. PharmaBench：通过大型语言模型增强ADMET基准测试。*Scientific Data*，11(985)。

+   Pushpakom等人（2019）Pushpakom, S.; Iorio, F.; Eyers, P. A.; Escott, K. J.; Hopper, S.; Wells, A.; Doig, A.; Guilliams, T.; Latimer, J.; McNamee, C.; 等人. 2019. 药物重定位：进展、挑战与建议。*自然药物发现评论*，18(1): 41–58。

+   Qin等人（2023）Qin, Y.; Hu, S.; Lin, Y.; Chen, W.; Ding, N.; Cui, G.; Zeng, Z.; Huang, Y.; Xiao, C.; Han, C.; Fung, Y. R.; Su, Y.; Wang, H.; Qian, C.; Tian, R.; Zhu, K.; Liang, S.; Shen, X.; Xu, B.; Zhang, Z.; Ye, Y.; Li, B.; Tang, Z.; Yi, J.; Zhu, Y.; Dai, Z.; Yan, L.; Cong, X.; Lu, Y.; Zhao, W.; Huang, Y.; Yan, J.; Han, X.; Sun, X.; Li, D.; Phang, J.; Yang, C.; Wu, T.; Ji, H.; Liu, Z.; 和Sun, M. 2023. 使用基础模型进行工具学习。arXiv:2304.08354。

+   Ravuru, Sakhinana和Runkana（2024）Ravuru, C.; Sakhinana, S. S.; 和Runkana, V. 2024. 用于时间序列分析的代理增强生成检索。见于*ACM SIGKDD知识发现与数据挖掘会议（KDD）本科生联合会议论文集*。

+   Roohani等人（2024）Roohani, Y.; Lee, A.; Huang, Q.; Vora, J.; Steinhart, Z.; Huang, K.; Marson, A.; Liang, P.; 和Leskovec, J. 2024. BioDiscoveryAgent：一个用于设计基因干扰实验的AI代理。*arXiv预印本 arXiv:2405.17631*。

+   Schick等人（2023）Schick, T.; Dwivedi-Yu, J.; Dessì, R.; Raileanu, R.; Lomeli, M.; Zettlemoyer, L.; Cancedda, N.; 和Scialom, T. 2023. Toolformer：语言模型可以自学使用工具。arXiv:2302.04761。

+   Si, Yang和Hashimoto（2024）Si, C.; Yang, D.; 和Hashimoto, T. 2024. LLM是否能生成新颖的研究创意？一项涉及100多位NLP研究人员的大规模人类研究。*arXiv预印本 arXiv:2409.04109*。

+   Siramshetty, Shah等人（2021）Siramshetty, V. B.; Shah, P.; 等人. 2021. 使用上市药物验证ADME QSAR模型。*SLAS发现*，26(10): 1326–1336。

+   Sterling和Irwin（2015）Sterling, T.; 和Irwin, J. J. 2015. ZINC 15–每个人的配体发现。*化学信息与建模期刊*，55(11): 2324–2337。

+   Wang等人（2024a）Wang, E.; Cassano, F.; Wu, C.; Bai, Y.; Song, W.; Nath, V.; Han, Z.; Hendryx, S.; Yue, S.; 和Zhang, H. 2024a. 规划自然语言改善LLM在代码生成中的搜索。*arXiv预印本 arXiv:2409.03733*。

+   Wang等人（2024b）Wang, L.; Ma, C.; Feng, X.; Zhang, Z.; Yang, H.; Zhang, J.; Chen, Z.; Tang, J.; Chen, X.; Lin, Y.; 等人. 2024b. 基于大语言模型的自主代理调查。*计算机科学前沿*，18(6)。

+   Wang等人（2016）Wang, N.; Dong, J.; Deng, Y.; Zhu, M.; Wen, M.; Yao, Z.; Lu, A.; Wang, J.; Luo, X.; 和Cao, D. 2016. 药物发现中的ADME特性评估：使用NSGA-II与Boosting的组合预测Caco-2细胞的渗透性。*化学信息与建模期刊*，56(4): 763–773。

+   Wang 等人 (2024c) Wang, Y.; Fu, T.; Xu, Y.; Ma, Z.; Xu, H.; Du, B.; Gao, H.; Wu, J.; 和 Chen, J. 2024c. TWIN-GPT: 通过大型语言模型为临床试验提供数字双胞胎。*ACM 多媒体计算、通信与应用交易*。

+   Wang, Fried, 和 Neubig (2024) Wang, Z.; Fried, D.; 和 Neubig, G. 2024. TroVE: 为解决编程任务引导可验证且高效的工具箱。在*第41届国际机器学习会议（ICML）会议论文集*中。

+   WecoAI (2024) WecoAI. 2024. AIDE: 机器学习工程师代理。

+   Wu 等人 (2024) Wu, S.; Zhao, S.; Huang, Q.; Huang, K.; Yasunaga, M.; Cao, K.; Ioannidis, V. N.; Subbian, K.; Leskove, J.; 和 Zou, J. 2024. AvaTaR: 为工具辅助知识检索优化LLM代理。

+   Xia 等人 (2024) Xia, Y.; Wang, Y.; Wang, Z.; 和 Zhang, W. 2024. 基于人工智能的药物发现中的分子优化综述。*定量生物学*，12(1): 15–29。

+   Yao 等人 (2023) Yao, S.; Zhao, J.; Yu, D.; Du, N.; Shafran, I.; Narasimhan, K.; 和 Cao, Y. 2023. ReAct: 在语言模型中协同推理与行动。在*国际学习表征会议（ICLR）*中。

+   Yoon, Kim, 和 Oh (2024) Yoon, S.; Kim, T. E.; 和 Oh, Y. J. 2024. 设计与评估用于人类-人工智能沟通的多聊天机器人界面：说服任务的初步发现。*arXiv 预印本 arXiv:2406.19648*。

+   Yue 等人 (2024) Yue, L.; Xing, S.; Chen, J.; 和 Fu, T. 2024. ClinicalAgent: 基于大型语言模型推理的临床试验多代理系统。*arXiv 预印本 arXiv:2404.14777*。

+   Zhang 等人 (2021) Zhang, B.; Fu, Y.; Lu, Y.; Zhang, Z.; Clarke, R.; Van Eyk, J. E.; Herrington, D. M.; 和 Wang, Y. 2021. DDN2.0: 用于生物系统差异依赖网络分析的R和Python包。*bioRxiv*，2021–04。

## 附录A LLM设计的代码

[⬇](data:text/plain;base64,ZnJvbSB0ZGMuc2luZ2xlX3ByZWQgaW1wb3J0IEFETUUKZnJvbSBza2xlYXJuLmVuc2VtYmxlIGltcG9ydCBSYW5kb21Gb3Jlc3RDbGFzc2lmaWVyCmZyb20gc2tsZWFybi5tZXRyaWNzIGltcG9ydCByb2NfYXVjX3Njb3JlLCBmMV9zY29yZQpmcm9tIHJka2l0LkNoZW0gaW1wb3J0IEFsbENoZW0KZnJvbSByZGtpdCBpbXBvcnQgQ2hlbQppbXBvcnQgbnVtcHkgYXMgbnAKCmRlZiBkb3dubG9hZF9hbmRfc3BsaXRfZGF0YXNldCgpOgogICAg"""CiAgICBEb3dubG9hZHMgdGhlIHNwZWNpZmllZCBBRE1FVCBkYXRhc2V0IGFuZCByZXR1cm5zIHRoZSB0cmFpbiBhbmQgdGVzdCBzcGxpdHMuCiAgICAiIiIKICAgIGRhdGEgPSBBRE1FKG5hbWU9J1BBTVBBX05DQVRTJykKICAgIHNwbGl0ID0gZGF0YS5nZXRfc3BsaXQoKQogICAgcmV0dXJuIHNwbGl0CgpkZWYgZ2VuZXJhdGVfZmluZ2VycHJpbnRzKHNtaWxlc19saXN0LCByYWRpdXM9Miwgbl9iaXRzPTIwNDgpOgogICAg"""CiAgICBDb252ZXJ0cyBhIGxpc3Qgb2YgU01JTEVTIHN0cmluZ3MgaW50byBtb2xlY3VsYXIgZmluZ2VycHJpbnRzLgogICAg"""CiAgICBmaW5nZXJwcmludHMgPSBbXQogICAgZm9yIHNtaWxlcyBpbiBzbWlsZXNfbGlzdDoKICAgICAgICBtb2wgPSBDaGVtLk1vbEZyb21TbWlsZXMoc21pbGVzKQogICAgICAgIGlmIG1vbDoKICAgICAgICAgICAgZmluZ2VycHJpbnRzLmFwcGVuZChBbGxDaGVtLkdldE1vcmdhbkZpbmdlcnByaW50QXNCaXRWZWN0KG1vbCwgcmFkaXVzLCBuQml0cz1uX2JpdHMpKQogICAgICAgIGVsc2U6CiAgICAgICAgICAgIGZpbmdlcnByaW50cy5hcHBlbmQobnAuemVyb3MoKG5fYml0cywpKSkKICAgIHJldHVybiBucC5hcnJheShmaW5nZXJwcmludHMpCgojIE1haW4gU2NyaXB0CmlmIF9fbmFtZV9fID09ICJfX21haW5fXyI6CiAgICAjIFN0ZXAgMTogRG93bmxvYWQgZGF0YXNldCBhbmQgZ2V0IHRyYWluLXRlc3Qgc3BsaXQKICAgIHNwbGl0ID0gZG93bmxvYWRfYW5kX3NwbGl0X2RhdGFzZXQoKQoKICAgICMgU3RlcCAyOiBHZW5lcmF0ZSBmZWF0dXJlIG1hdHJpY2VzIGFuZCBsYWJlbHMKICAgIFhfdHJhaW4gPSBnZW5lcmF0ZV9maW5nZXJwcmludHMoc3BsaXRbJ3RyYWluJ11bJ0RydWcnXSkKICAgIHlfdHJhaW4gPSBzcGxpdFsndHJhaW4nXVsnWSddCgogICAgWF90ZXN0ID0gZ2VuZXJhdGVfZmluZ2VycHJpbnRzKHNwbGl0Wyd0ZXN0J11bJ0RydWcnXSkKICAgIHlfdGVzdCA9IHNwbGl0Wyd0ZXN0J11bJ1knXQoKICAgICMgU3RlcCAzOiBUcmFpbiBSYW5kb20gRm9yZXN0IENsYXNzaWZpZXIKICAgIHJmX21vZGVsID0gUmFuZG9tRm9yZXN0Q2xhc3NpZmllcihuX2VzdGltYXRvcnM9MTAwLCByYW5kb21fc3RhdGU9NDIpCiAgICByZl9tb2RlbC5maXQoWF90cmFpbiwgeV90cmFpbikKCiAgICAjIFN0ZXAgNDogTWFrZSBQcmVkaWN0aW9ucwogICAgeV9wcmVkID0gcmZfbW9kZWwucHJlZGljdChYX3Rlc3QpCiAgICB5X3ByZWRfcHJvYiA9IHJmX21vZGVsLnByZWRpY3RfcHJvYmEoWF90ZXN0KVs6LCAxXSAgIyBQcm9iYWJpbGl0eSBlc3RpbWF0ZXMgZm9yIFJPQy1BVUMKCiAgICByb2NfYXVjID0gcm9jX2F1Y19zY29yZSh5X3Rlc3QsIHlfcHJlZF9wcm9iKQogICAgZjEgPSBmMV9zY29yZSh5X3Rlc3QsIHlfcHJlZCkKCiAgICBwcmludChmIlJPQy1BVUM6IHtyb2NfYXVjOi40Zn0iKQogICAgcHJpbnQoZiJGMS1TY29yZToge2YxOi40Zn0iKQoKCg==)

## 附录 B 示例提示

#### 想法生成

规划者的任务是基于计算机科学概念，为机器学习实验生成多样化的想法。对于每个想法，进一步提示其通过加入细节并整合计算生物学概念来完善方法。以下是示例输出：

[⬇](data:text/plain;base64,ewogICAgImlkZWFzIjogWwogICAgICB7CiAgICAgICAgIm1ldGhvZCI6ICJBcHBseSBhIGdyYXBoIG5ldXJhbCBuZXR3b3JrIChHTk4pIHRvIGxlYXJuIGZyb20gdGhlIFNNSUxFUyBzdHJpbmcgc3RydWN0dXJlLiIsCiAgICAgICAgInJlZmluZWRfaWRlYSI6ICJSZXByZXNlbnQgdGhlIFNNSUxFUyBzdHJpbmcgYXMgYSBtb2xlY3VsYXIgZ3JhcGgsIHdoZXJlIG5vZGVzIGFyZSBhdG9tcyBhbmQgZWRnZXMgcmVwcmVzZW50IGJvbmRzLiBUcmFpbiBhIEdOTiBtb2RlbCB0byBsZWFybiBtb2xlY3VsYXIgcmVwcmVzZW50YXRpb25zIGZyb20gdGhpcyBncmFwaCwgd2hpY2ggY2FuIHRoZW4gYmUgdXNlZCB0byBwcmVkaWN0IHRoZSBBRE1FVCBwcm9wZXJ0eS4iCiAgICAgIH0sCiAgICAgIHsKICAgICAgICAibWV0aG9kIjogIlVzZSBhIHRyYW5zZm9ybWVyLWJhc2VkIG1vZGVsIHRvIGVuY29kZSB0aGUgU01JTEVTIHN0cmluZy4iLAogICAgICAgICJyZWZpbmVkX2lkZWEiOiAiVHJhaW4gYSB0cmFuc2Zvcm1lciBtb2RlbCB0byBsZWFybiBlbWJlZGRpbmdzIGZvciBTTUlMRVMgc3RyaW5ncywgY2FwdHVyaW5nIHRoZSBzZXF1ZW50aWFsIGluZm9ybWF0aW9uIGFuZCByZWxhdGlvbnNoaXBzIGJldHdlZW4gYXRvbXMgYW5kIGZ1bmN0aW9uYWwgZ3JvdXBzLiBUaGVzZSBlbWJlZGRpbmdzIGNhbiBiZSB1c2VkIGFzIGlucHV0IGZvciBhIHJhbmRvbSBmb3Jlc3QgbW9kZWwgdG8gcHJlZGljdCB0aGUgQURNRVQgcHJvcGVydHkuIgogICAgICB9LAogICAgICB7CiAgICAgICAgIm1ldGhvZCI6ICJVc2UgdG8gYSB0cmFuc2ZvbWVyLWJhc2VkIG1vZGVsIHRvIGVuY29kZSB0aGUgU01JTEVTIHN0cmluZy4iLAogICAgICAgICJyZWZpbmVkX2lkZWEiOiAiVHJhaW4gYSB0cmFuc2Zvcm1lciBtb2RlbCB0byBsZWFybiBlbWJlZGRpbmdzIGZvciBTTUlMRVMgc3RyaW5ncywgY2FwdHVyaW5nIHRoZSBzZXF1ZW50aWFsIGluZm9ybWF0aW9uIGFuZCByZWxhdGlvbnNoaXBzIGJldHdlZW4gYXRvbXMgYW5kIGZ1bmN0aW9uYWwgZ3JvdXBzLiBUaGVzZSBmZWF0dXJlcyBjYW4gYmUgdXNlZCBhcyBpbnB1dCBmb3IgYSBkb3duc3RyZWFtIHByZWRpY3Rpb24gbW9kZWwuIgogICAgICB9LAogICAgICB7CiAgICAgICAgIm1ldGhvZCI6ICJUcmFpbmEgYSByYW5kb20gZm9yZXN0IG1vZGVsIHdpdGggU01JTEVTIHN0cmluZyBmb3JtZWQgdW5wZXIgUlAga2VybmVsLiIsCiAgICAgICAgInJlZmluZWRfaWRlYSI6ICJFeHRyYWN0IG1vbGVjdWxhcmsgZGVzY3JpcHRvcnMgZnJvbSB0aGUgU01JTEVTIHN0cmluZ3MgYXMgdXNpbmcgUkRLaXQgYW5kIHRyYWluIGEgU1ZNIG1vZGVsIHdpdGggYW4gUkJGIGtlcm5lbCB0byBwcmVkaWN0IHRoZSBBRE1FVCBwcm9wZXJ0eS4iCiAgICAgIH0sCiAgICAgIHsKICAgICAgICAibWV0aG9kIjogIkltcGxlbWVudCBhIG51bWJlci10YXNrIGxlYXJuaW5nIGFwcHJvYWNoIGJ5IHByZWRpY3RpbmcgbXVsdGlwbGUgQURNSVQgcHJvcGVydGlldHMgc2ltdWx0YW5lbnRpb3VzLiIsCiAgICAgICAgInJlZmluZWRfaWRlYSI6ICJUaGlzIGlkZWEgbmVlZHMgZnVydGhlciBjbGFyaWZpY2F0aW9uLiBXaGF0IHNwZWNpZmljIEFETUVUIHByb3BlcnRpZXMgYXJlIGJlaW5nIGNvbnNpZGVyZWQ/IFdoYXQgaXMgdGhlIHJlbGF0aW9uc2hpcCBiZXR3ZWVuIHRoZXNlIHByb3BlcnRpZXMgYW5kIHRoZSBDYWNvLTIgZGF0YXNldD8iCiAgICAgIH0sCiAgICAgIHsKICAgICAgICAibWV0aG9kIjogIkZpbmUtdHVuZSBhIHByZS10cmFpbmVkIG1vZGVsIHN1Y2ggYXMgQ2hlbUJFUlRhIGZvciBBRE1FVCBwcm9wZXJ0eSBwcmVkaWN0aW9uLiIsCiAgICAgICAgInJlZmluZWRfaWRlYSI6ICJVc2UgdGhlIENoZW1CRVJUYSBwcmUtdHJhaW5lZCBtb2RlbCB0byBleHRyYWN0IG1vbGVjdWxhciByZXByZXNlbnRhdGlvbnMgZGlyZWN0bHkgZnJvbSBTTUlMRVMgc3RyaW5ncy4gRmluZS10dW5lIHRoZSBtb2RlbCBmb3Igc3BlY2lmaWMgQURNRVQgcHJvcGVydGllZXMsIGxldmVyYWdpbmcgdGhlIGNvbnRleHR1YWwgaW5mb3JtYXRpb24gY2FwdHVyZWQgZHVyaW5nIHByZS10cmFpbmluZy4iCiAgICAgIH0KICAgIF0KICB9CiAgXQogIH0KICA=)

#### 领域知识分析

教师被提示将这个想法分解成子步骤，并评估每个子步骤是否需要领域知识。以下是应用于使用随机森林模型的示例。

[⬇](data:text/plain;base64,U3RlcCAxOiBMb2FkIGFuZCBzcGxpdCB0aGUgUEFNUEEgZGF0YXNldC4KLSAocmVxdWlyZSBkb21haW4ga25vd2xlZGdlIC0gcmVhc29uOiB1bmRlcnN0YW5kaW5nIHRoZSBzcGVjaWZpYyBzdHJ1Y3R1cmUgYW5kIGZvcm1hdCBvZiB0aGUgUEFNUEEgZGF0YXNldCwgaW5jbHVkaW5nIGRvd25sb2FkaW5nIGZyb20gZG9tYW4tHNwZWNpZmljIHNvdXJjZXMpCgpTdGVwIDI6IENvbnZlcnQgY2hlbWljYWwgY29tcG91bmRzIGluIHRoZSBkYXRhc2V0IHRvIE1vcmdhbiBmaW5nZXJwcmludHMgdXNpbmcgYSBjaGVtaWNhbCBsaWJyYXJ5IChlLmcuLCBSREtpdCkuCi0gKHJlcXVpcmUgZG9tYWluIGtub3dsZWRnZTogUmFuZG9tIEZvcmVzdCBpcyBhIGdlbmVyYWwgbWFjaGluZSBsZWFybmluZyBhbGdvcml0aG0pCgpTdGVwIDQ6IFRyYWluIHRoZSBSYW5kb20gRm9yZXN0IG1vZGVsIG9uIHRoZSB0cmFpbmluZyBkYXRhLgotIChkb24ndCByZXF1aXJlIGRvbWFpbiBrbm93bGVkZ2U6IG1vZGVsIHRyYWluaW5nIGlzIGEgZ2VuZXJhbCBjb21wdXRlciBzY2llbmNlIHRhc2kpCgpTdGVwIDU6IEV2YWx1YXRlIHRoZSBtb2RlbCdzIHBlcmZvcm1hbmNlIHVzaW5nIHN0YW5kYXJkIG1ldHJpY3MgbGlrZSBNQUUsIE1TRSwgb3IgYWNjdXJhY3kuCi0gKGRvbid0IHJlcXVpcmUgZG9tYWluIGtub3dsZWRnZTogZXZhbHVhdGlvbiB1c2luZyBzdGFuZGFyZCBtZXRyaWNzIGlzIGEgZ2VuZXJhbCBjb21wdXRlciBzY2llbmNlIHRhc2sp
