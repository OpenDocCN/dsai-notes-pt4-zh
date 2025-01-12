<!--yml

类别：未分类

日期：2025-01-11 11:50:33

-->

# DRC-Coder：使用大型语言模型自主代理生成自动化DRC检查器代码

> 来源：[https://arxiv.org/html/2412.05311/](https://arxiv.org/html/2412.05311/)

陈佳昌 杜克大学 北卡罗来纳州 达勒姆 美国 [chenchia.chang@duke.edu](mailto:chenchia.chang@duke.edu)， 何家东 英伟达研究中心 加利福尼亚州 圣克拉拉 美国 [chiatungh@nvidia.com](mailto:chiatungh@nvidia.com)， 李雅光 英伟达 美国 德克萨斯州 奥斯汀 [yaguangl@nvidia.com](mailto:yaguangl@nvidia.com)， 陈怡然 杜克大学 北卡罗来纳州 达勒姆 美国 [yiran.chen@duke.edu](mailto:yiran.chen@duke.edu)， 任浩星 英伟达研究中心 美国 德克萨斯州 奥斯汀 [haoxingr@nvidia.com](mailto:haoxingr@nvidia.com)(2025)

###### 摘要。

在先进技术节点中，集成设计规则检查器（DRC）通常用于放置与布线工具中，以实现功率、性能、面积的快速优化循环。实现集成DRC检查器以满足商业DRC工具的标准，通常需要广泛的人类专业知识来解读工艺规范、分析布局，并反复调试代码。然而，这一劳动密集型过程，每次技术节点更新时都需要重复执行，延长了电路设计的周转时间。

在本文中，我们介绍了DRC-Coder，一个具有视觉能力的多代理框架，用于自动化DRC代码生成。通过结合视觉语言模型和大型语言模型（LLM），DRC-Coder能够有效地处理文本、视觉和布局信息，由两个专门的LLM进行规则解释和代码生成。我们还设计了一个LLM自动评估功能，以支持DRC代码调试。实验结果表明，在针对一个先进的亚3纳米技术节点的标准单元布局工具时，DRC-Coder在生成符合商业DRC工具标准的DRC代码时，达到了完美的F1得分1.000，远远超过了标准提示技术（F1=0.631）。DRC-Coder在平均四分钟内生成每个设计规则的代码，显著加速了技术进步并降低了工程成本。

设计规则检查，代码生成，大型语言模型^†^†期刊年份：2025^†^†版权：acm许可^†^†会议：2025年国际物理设计研讨会论文集；2025年3月16日至19日；美国德克萨斯州奥斯汀^†^†书名：2025年国际物理设计研讨会论文集（ISPD ’25），2025年3月16日至19日，奥斯汀，德克萨斯州，美国^†^†ISBN：979-8-4007-1293-7/25/03^†^†DOI：10.1145/3698364.3705347^†^†CCS：硬件 物理设计（EDA）^†^†CCS：计算方法 人工智能

## 1. 引言

![参考说明](img/25d6192e76003358577dedba11ffe466.png)

图1\. DRC 检查器开发流程。流程从对厂商提供的描述和从商业 DRC 工具报告中提取的布局及其 DRC 规则（DRV）开始。然后，流程进入编码、对齐检查和调试阶段。所提出的 LLM-agent 系统自动化了这一过程，相较于手动编码，大大减少了开发时间。

在先进技术节点的时代，设计规则检查（DRC）是物理设计中的一个关键且复杂的步骤，原因在于设计规则的数量不断增加，跨层设计规则变得更加复杂，以及严格的图形化规则。布局与布线（P&R）工具通常需要集成的 DRC 检查器，以确保制造可行性，并能够比每次迭代都运行商业 DRC 工具更快速地优化功耗、性能和面积（PPA）。实现集成的 DRC 检查器通常需要经验丰富的工程师数周时间，涉及多次调试迭代，从厂商文档中提取 DRC 规则，并确保集成的检查器符合商业 DRC 工具的标准，如图[1](https://arxiv.org/html/2412.05311v1#S1.F1 "图 1 ‣ 1\. 引言 ‣ DRC-Coder：使用 LLM 自主代理生成自动化 DRC 检查器代码")所示。此外，这一过程需要在每项新技术开发时重复进行，显著增加了电路设计的周期。因此，开发一种高效且智能的 DRC 检查器代码生成方法对于提高集成 DRC 检查器与商业 DRC 工具之间的一致性以及缩短新技术的开发时间至关重要。

现如今，大型语言模型（LLMs）(Radford et al., [2019](https://arxiv.org/html/2412.05311v1#bib.bib14); Raffel et al., [2020](https://arxiv.org/html/2412.05311v1#bib.bib15); Chung et al., [2022](https://arxiv.org/html/2412.05311v1#bib.bib4); Nijkamp et al., [2023](https://arxiv.org/html/2412.05311v1#bib.bib12); OpenAI, [2024](https://arxiv.org/html/2412.05311v1#bib.bib13)) 已经展示了卓越的推理和代码生成能力。此外，视觉语言模型（VLMs）(Zhang et al., [2024](https://arxiv.org/html/2412.05311v1#bib.bib23); Abdin et al., [2024](https://arxiv.org/html/2412.05311v1#bib.bib2); OpenAI, [2024](https://arxiv.org/html/2412.05311v1#bib.bib13)) 也能够有效地执行多模态推理。此外，LLMs在通过LLM自主智能体（LLM-agent）(Wang et al., [2024](https://arxiv.org/html/2412.05311v1#bib.bib18); Yao et al., [2022b](https://arxiv.org/html/2412.05311v1#bib.bib22); LangChain-AI, [2024](https://arxiv.org/html/2412.05311v1#bib.bib9); Wu et al., [2023](https://arxiv.org/html/2412.05311v1#bib.bib19)) 解决复杂任务方面显示了巨大的潜力。例如，LLM-agent 可以通过自动调试来稳定LLM编码过程 (Yang et al., [2024](https://arxiv.org/html/2412.05311v1#bib.bib20))。在LLM-agent中，LLMs能够分解任务，生成中间指令并提供反馈。此外，LLMs还可以与外部环境交互，例如调用实用功能来解决问题。因此，将LLM-agent与VLMs和LLMs结合起来，是进行DRC解释和代码生成的良好候选方案，能够实现自动推理和自动调试。

在本工作中，我们提出了DRC-Coder，一个具备视觉能力的多智能体框架，用于自动化DRC代码生成。该方法能够从多种模态中获取信息，包括文本描述、视觉插图和布局表示，用于全面的设计规则解释。我们的多智能体框架将过程分解为两个层次的子任务，包括解释和编码，模仿了人类DRC编码过程，如图[1](https://arxiv.org/html/2412.05311v1#S1.F1 "图1 ‣ 1\. 介绍 ‣ DRC-Coder：基于LLM自主智能体的自动化DRC检查器代码生成")所示。通过为每个子任务分配具有不同角色的两个LLM，我们增强了LLM的推理能力，并减少了LLM在用单一智能体解决复杂DRC编码时的潜在幻觉。此外，通过集成我们提出的领域特定实用功能，DRC-Coder可以通过在布局数据集上执行生成的代码进行自动评估，获取性能报告以进行自动调试，从而确保生成代码的有效性。

据我们所知，我们是EDA领域中第一个研究自动化DRC代码生成问题的工作。这项工作不同于相关工作（Zhu等人，[2023](https://arxiv.org/html/2412.05311v1#bib.bib24)），后者仅关注提取关键的设计规则组件，而不是生成无需人工干预的完整代码。为了展示这一概念，DRC-Coder以标准单元布局自动化的DRC检查器为目标（Li等人，[2019](https://arxiv.org/html/2412.05311v1#bib.bib11)；Van Cleeff等人，[2019](https://arxiv.org/html/2412.05311v1#bib.bib17)；Lee等人，[2020](https://arxiv.org/html/2412.05311v1#bib.bib10)；Ren和Fojtik，[2021](https://arxiv.org/html/2412.05311v1#bib.bib16)；Ho等人，[2023](https://arxiv.org/html/2412.05311v1#bib.bib6)）使用子3nm技术节点。这一检查器在基于网格的布局格式下运行，该格式将布局金属表示为网格坐标，例如图[1](https://arxiv.org/html/2412.05311v1#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")中的(8, 0, M0, N5)，(5, 4, M0, N9)。这是因为路由器通常以基于网格的方式进行轨道路由。

贡献总结如下：

+   •

    我们提出了DRC-Coder，这是第一个自动化的DRC检查器代码生成系统，旨在加速技术迁移过程并减少工程工作量。

+   •

    我们开发了一种新型的具有视觉能力的多代理框架，该框架有效地解释了设计规则和布局的多模态信息，并启用了自动化的调试和反馈机制。

+   •

    我们将DRC代码生成分解为两个层次的任务：解释和编码，并将它们分配给两个专门的代理，以提高LLM推理和可靠性。

+   •

    我们为LLM代理提出了三种特定领域的实用功能，包括设计规则和布局的视觉分析以及自动化的代码评估。

+   •

    使用子3nm技术节点的评估结果表明，我们的DRC-Coder成功生成了所有NVCell中考虑的设计规则的正确代码（F1=1.000）（Ren和Fojtik，[2021](https://arxiv.org/html/2412.05311v1#bib.bib16)），而标准提示方法产生了不满意的结果（F1=0.631）。此外，DRC-Coder平均每条规则生成代码的时间不到四分钟。

DRC-Coder具有扩展到其他与DRC相关问题的潜力，包括DRC文档解释、测试模式生成和设计规则优化。此外，我们希望这项工作能够为半导体行业自动化复杂工程任务的研究方向开辟新的道路。

## 2\. 初步知识

在本节中，我们首先研究了与LLM代理框架相关的工作。然后，我们介绍了VLM及其在解释设计规则图像和布局中的潜力。最后，我们介绍了在标准单元布局工具中使用的基于网格的DRC检查器。

### 2.1\. LLM-Agent框架

大型语言模型自主代理（LLM-agents）（Wang 等人，[2024](https://arxiv.org/html/2412.05311v1#bib.bib18); Yao 等人，[2022b](https://arxiv.org/html/2412.05311v1#bib.bib22); LangChain-AI，[2024](https://arxiv.org/html/2412.05311v1#bib.bib9); Wu 等人，[2023](https://arxiv.org/html/2412.05311v1#bib.bib19)）已经成为强大的工具，使得大型语言模型能够基于推理过程制定计划并执行外部功能。LLM-agents 在各个领域展现了其有效性。在在线购物场景中（Yao 等人，[2022a](https://arxiv.org/html/2412.05311v1#bib.bib21)），LLM 可以根据用户指令搜索、选择网站上的产品，并推理何时购买产品以满足用户需求。在编程任务中（Chen 等人，[2022](https://arxiv.org/html/2412.05311v1#bib.bib3)），LLM-agents 可以生成代码，编译并执行代码，并基于编译器和执行反馈进行迭代调试。这种推理、行动和从反馈中学习的能力展示了 LLM-agents 增强的解决问题能力。在芯片设计中，LLM-agents 也被应用于诸如 Verilog 和布局聚类生成等任务（Ho 和 Ren，[2024](https://arxiv.org/html/2412.05311v1#bib.bib7); Ho 等人，[2024](https://arxiv.org/html/2412.05311v1#bib.bib8)），展现了它们在专业领域和硬件相关问题中的潜力。

然而，现有框架仅处理纯文本表示，这对于解释电路布局和设计规则并不有效。因此，在 LLM-agent 框架中具备视觉理解能力至关重要。此外，我们应该提供一个特定领域的 DRC 代码评估功能，以便对检测设计规则违反（DRV）时代码的性能提供有意义的反馈。这可以帮助 LLM 有效避免生成代码所产生的设计规则违反（DRV）的假阳性和假阴性。

![参见说明](img/98c31445cbdde944cf59da0ae1de95c6.png)

图 2\. 两个 VLM（GPT-4o 和 Phi-3）响应的比较，分别针对 (a) 设计规则和 (b) 布局。在布局图像中，黄色多边形表示 M0 层的金属，黑色多边形表示由商业 DRC 工具标记的 DRV 区域，黑色十字表示网格坐标中的相应 DRV 位置。

### 2.2\. DRC 解释挑战与 VLMs

Foundries通过简洁的文本描述和视觉图示指定每个设计规则，这些描述通常暗示了复杂的空间条件。例如，图[1](https://arxiv.org/html/2412.05311v1#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")中规则M0.S.1的描述有一张图，展示了规则M0.S.1和M0.S.2的多个间距场景S1A1、S1A2，以及一个带有缩写术语PRL的文本。需要解释的是，实际条件是：当平行运行长度（PRL）$\geq-1$时，M0层中金属之间的水平间距必须大于1，其中PRL可以视为垂直间距。

电路设计师还必须分析商用DRC工具在布局中的报告，以揭示在foundry描述中未明确指出的隐含条件。图[1](https://arxiv.org/html/2412.05311v1#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")揭示了商用工具进一步检查一个边界条件：x边界与金属之间的空间必须大于1。上述示例表明，在DRC代码生成中，准确分析foundry描述和商用工具报告的挑战。

VLM（张等人，[2024](https://arxiv.org/html/2412.05311v1#bib.bib23)）能够处理图像和文本，根据图像回答用户的查询。例如，它可以给出图像的解释或区分图像之间的差异。因此，VLM有潜力帮助LLM-Agent解释设计规则图像和布局。在此，我们使用两种最先进的VLM，包括Phi-3（Abdin等人，[2024](https://arxiv.org/html/2412.05311v1#bib.bib2)）和GPT-4o（OpenAI，[2024](https://arxiv.org/html/2412.05311v1#bib.bib13)），对设计规则和布局进行图像解释。结果如图[2](https://arxiv.org/html/2412.05311v1#S2.F2 "Figure 2 ‣ 2.1\. LLM-Agent Framework ‣ 2\. Preliminaries ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")所示。GPT-4o能够生成有意义的回应。另一方面，Phi-3生成了不满意的回应，其中空间不在M0和S1A1之间，且DRV不在(6, 4)和(6, 2)之间。根据这个实验，我们观察到，VLM，特别是GPT-4o，能够帮助解释复杂的设计规则和布局。因此，我们将此VLM与LLM-agents结合，以处理文本、视觉和布局信息，从而实现DRC代码生成的有效规则解释。

![参考图注](img/4a336f8867f3d885c47e66e790db105c.png)

图3. 基于网格的设计规则M0.S.1的DRC代码。

### 2.3. 基于网格的DRC检查器

在我们的评估中，我们使用 NVCell (Ren 和 Fojtik, [2021](https://arxiv.org/html/2412.05311v1#bib.bib16)) 作为我们的目标标准单元布局工具。NVCell 使用基于网格的DRC检查器快速获得布局性能。例如，它可以在0.05秒内完成一个包含22个器件的单元的DRC检查，而商业DRC工具则需要215秒。在本文中，我们的目标是为小于3nm技术节点生成一个新的基于网格的DRC检查器。图[3](https://arxiv.org/html/2412.05311v1#S2.F3 "图3 ‣ 2.2\. DRC解释挑战与VLM ‣ 2\. 前言 ‣ DRC-Coder: 基于LLM自主代理的自动化DRC检查器代码生成")展示了设计规则M0.S.1的基于网格的DRC代码示例（图[2](https://arxiv.org/html/2412.05311v1#S2.F2 "图2 ‣ 2.1\. LLM-Agent框架 ‣ 2\. 前言 ‣ DRC-Coder: 基于LLM自主代理的自动化DRC检查器代码生成")(a)）。检查器的核心是drc函数，它接受三个参数：

+   •

    layout：一个元组列表。每个元组 $(x,y,\text{层},\text{网})$ 表示布局组件的网格坐标 $(x,y)$、金属层和网名称。注意，我们对所有层使用相同的坐标系统，因为 $x$ 和 $y$ 分别是水平和垂直坐标。

+   •

    max_x 和 max_y：布局在 $x$ 和 $y$ 方向上的最大网格坐标。

这个drc函数实现了两个检查：

1.  (1)

    边界检查：识别位于布局边缘或超出边缘的组件 $(x\leq 1\text{ 或 }x\geq\text{max\_x}-1)$。

1.  (2)

    间距检查：根据组件之间的相对位置检测违规。

该函数返回一个DRV列表，每个DRV是一个元组，表示违反规则的组件。在此示例中，输出为 [((2, 0, M0), (3, 2, M0))]，表示组件(2, 0, M0)与(3, 2, M0)之间存在间距违规。

![参见说明](img/0483bb1696823d4be976a4b19e1d44b6.png)

图4\. 从商业DRC工具报告中的DRV转换为基于网格的DRV。商业工具报告中的DRV位置由黑色多边形标记，每个多边形由四个点及其x、y坐标定义。我们的基于网格的方法识别与这些多边形相交的布局组件，并使用这些组件的网格坐标表示DRV。

![参见说明](img/a6d528e9dbf0c8c47c608c796ebe833b.png)

图5\. DRC-Coder概述。规划器首先通过执行分析工具功能来解释输入的设计规则。程序员接收规则条件生成代码。最后，执行DRC代码评估以提供代码性能反馈。规划器接收反馈进行重新推理，程序员进行调试，直到生成正确的代码。

![参见说明](img/3a4c72a4a6907047eeda7e5b00a8713f.png)

图6\. DRC-Coder的初始提示。设计规则（DR）依赖的输入部分根据不同的目标设计规则动态变化。

## 3\. 数据准备

为了评估生成的DRC代码，我们创建了一个包含标准单元布局及其DRC报告的数据集。数据集准备过程有两个步骤：布局生成和DRC报告预处理。

### 3.1 布局生成

我们通过在不进行DRC修复的情况下变异路由行为，使用NVCell生成了207种不同的标准单元布局。这种方法确保了评估所需的广泛DRV场景。这些布局以网格格式表示，采用了第[2.3](https://arxiv.org/html/2412.05311v1#S2.SS3 "2.3\. Grid-based DRC Checker ‣ 2\. Preliminaries ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")节中所述的基于网格的DRC检查器。

### 3.2 DRC报告预处理

预处理阶段将来自商业工具的基于物理坐标的DRC报告转换为与我们基于网格的DRC检查器输出格式对齐的网格表示。预处理包括以下步骤：

1.  (1)

    通过运行商业DRC工具生成布局的DRC报告。这些报告使用多边形标记物理坐标中的DRV位置。

1.  (2)

    确定与商业DRC工具报告的DRV多边形相交的布局组件。

最终，我们将这些布局组件的基于网格的坐标视为我们评估过程中的DRV的真实值。这个基于网格的DRV报告可以精确地捕捉到涉及每个DRV的布局组件。[图4](https://arxiv.org/html/2412.05311v1#S2.F4 "Figure 4 ‣ 2.3\. Grid-based DRC Checker ‣ 2\. Preliminaries ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")展示了这个转换过程，演示了来自商业工具报告的基于多边形的DRV如何转化为我们的基于网格的表示。对于DRC-Coder中的DRV解释，我们还构建了每个布局及其DRV的基于网格的可视化，如[图1](https://arxiv.org/html/2412.05311v1#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")、[图2](https://arxiv.org/html/2412.05311v1#S2.F2 "Figure 2 ‣ 2.1\. LLM-Agent Framework ‣ 2\. Preliminaries ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")(b)和[图4](https://arxiv.org/html/2412.05311v1#S2.F4 "Figure 4 ‣ 2.3\. Grid-based DRC Checker ‣ 2\. Preliminaries ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")中所示的示例。

## 4. DRC-Coder

我们的方法 DRC-Coder 基于规则逐条生成 DRC 代码。为了使 DRC-Coder 在面对新技术节点时更具适应性，我们的代码生成是在零样本设置下进行的，即在生成过程中不提供示例代码。DRC-Coder 的整体流程如图 [5](https://arxiv.org/html/2412.05311v1#S2.F5 "Figure 5 ‣ 2.3\. Grid-based DRC Checker ‣ 2\. Preliminaries ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent") 所示。该系统的核心由两个 LLM 代理组成，它们以群聊方式进行操作：(1) 规划器：负责解释网格领域中的设计规则条件。(2) 程序员：将设计规则条件转化为可执行代码。这些代理由通用 LLM（GPT-4o 由 OpenAI 提供，[2024](https://arxiv.org/html/2412.05311v1#bib.bib13)）驱动，但在 DRC 编码过程中承担不同的角色。这种多代理方法将 DRC 过程分解为规划和编码阶段，使每个代理能够专注于其专业任务，从而提高整体系统性能。

为了处理图像输入并评估 DRC 代码，DRC-Coder 为代理提供了三个专用工具函数：(1) 铸造规则分析，(2) 布局 DRV 分析，和 (3) DRC 代码评估。这些工具为代理在整个代码生成过程中提供了专门的图像分析和评估能力。

工作流从输入设计规则开始，经过提示生成阶段，生成一个初始提示给规划器。然后，规划器和程序员在工具函数的帮助下共同生成 DRC 检查器代码。最终，代码生成经历一个迭代的自动调试过程，直到 DRC 报告与商业 DRC 工具的实际 DRV 对齐。该工作流的示例如图 [11](https://arxiv.org/html/2412.05311v1#S5.F11 "Figure 11 ‣ 5.1\. Experiment Setup ‣ 5\. Experimental Results ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent") 所示。接下来，我们将详细介绍 DRC-Coder 中的所有组件。

### 4.1\. 提示生成

给定输入的设计规则，本阶段构建了一个结构化的初始提示，如图 [6](https://arxiv.org/html/2412.05311v1#S2.F6 "Figure 6 ‣ 2.3\. Grid-based DRC Checker ‣ 2\. Preliminaries ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent") 所示，传递给规划器。该提示的组件分为固定部分和设计规则（DR）相关部分。固定部分包含：(1) 用于开发 Python 函数以识别布局数据中的 DRV 的任务定义。(2) 正式陈述函数输入和输出格式的需求。(3) 将编码问题分解为子任务的逐步指南，供规划器和程序员使用。

依赖于设计规则（DR）部分包含以下内容：（1）来自工厂文档的目标设计规则描述。（2）带有金属信息的布局示例以及对应的DRV位置，以提供具体的分析案例。请注意，我们从数据集中随机选择了两个包含目标DRV的布局示例来构建提示。此外，依赖于设计规则的部分会根据目标设计规则动态调整。

![请参阅标题](img/e979e9952555da8f0857954d7de96cfc.png)

图7. 工厂规则分析的使用及示例响应。此功能使用VLM解释设计规则描述，包括文本和图像输入，以生成关于设计规则的详细分析。

![请参阅标题](img/ab75f8677be432bc6f7d24e062d515d4.png)

图8. 布局DRV分析的使用及示例响应。此功能利用VLM解释输入单元的布局图像，识别其DRV，并提供关于DRV的详细网格基础说明。

![请参阅标题](img/896c09e1cdf98ed15a0e26e2fdf2ab34.png)

图9. DRC代码评估的使用及示例响应。通过在布局数据集上执行生成的输入代码，此功能将代码输出与从商业工具报告转换的黄金网格基础DRV进行比较。然后，它为我们的两个LLM代理提供性能报告。

### 4.2\. Planner

Planner是一个专注于解释工厂提供的设计规则描述和布局的LLM代理，用于生成对应的网格域设计规则条件。这些工厂提供的描述通常简洁且为多模态，结合了文本和图像，使得它们难以直接用于编码目的。为了帮助Planner解释图像信息，我们为Planner设计了两个实用功能：工厂规则分析和布局DRV分析。在每一轮解释中，Planner可以控制是否调用这些功能以获取更多信息。如果调用功能，Planner将收到工具功能的响应，自动将所有信息转化为基于网格的设计规则条件。在图[11](https://arxiv.org/html/2412.05311v1#S5.F11 "图11 ‣ 5.1\. 实验设置 ‣ 5\. 实验结果 ‣ DRC-Coder: 使用LLM自主代理自动生成DRC检查器代码")所示的示例中，Planner生成的设计规则条件包含DRV分析和编写代码的计划，包括边界和间距检查。

工厂规则分析。此功能处理 Planner 提出的关于 DRV 的特定问题。然后，调用 VLM 来解释工厂文档中设计规则的描述（结合文本和图像），并提供问题的答案。如图 [7](https://arxiv.org/html/2412.05311v1#S4.F7 "Figure 7 ‣ 4.1\. Prompting ‣ 4\. DRC-Coder ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent") 所示，功能分析提供的图像，识别目标间距方向，并为每个间距要求生成详细的 DRC 条件响应。这种自动化解释帮助 Planner 理解以多模态格式呈现的复杂设计规则，便于将工厂规范转化为精确且基于网格的条件，可用于 DRC 代码生成。

布局 DRV 分析。此功能接受两个输入：Planner 提出的关于设计规则和布局的问题，以及一个包含要检查布局的单元名称列表。然后，它利用 VLM（GPT-4o）来解释指定的布局图像。VLM 在提供的网格坐标中识别关键元素，如金属区域和 DRV 位置。如图 [8](https://arxiv.org/html/2412.05311v1#S4.F8 "Figure 8 ‣ 4.1\. Prompting ‣ 4\. DRC-Coder ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent") 所示，该功能生成一个全面的响应，解答 Planner 的问题，详细说明检测到的 DRV 原因，包括具体坐标和描述如间距问题或边界违规等问题。这种自动化和上下文分析增强了 Planner 生成更准确的基于网格的设计规则条件的能力。

### 4.3\. 程序员

程序员是一个 LLM 代理，负责将 Planner 生成的基于网格的设计规则条件转换为可执行的 DRC 代码。为了了解生成的代码性能，我们设计了一个工具功能，DRC 代码评估。图 [11](https://arxiv.org/html/2412.05311v1#S5.F11 "Figure 11 ‣ 5.1\. Experiment Setup ‣ 5\. Experimental Results ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent") 中的示例展示了生成的 DRC 代码。

DRC 代码评估。该功能输入生成的代码并输出代码的性能报告。具体来说，该功能在数据集中的单元布局上执行我们生成的代码，并将代码输出与商业工具的黄金 DRC 报告直接进行比较。生成的 DRC 代码必须根据每个设计规则，正确地将布局网格分类为 DRC 合规或 DRC 违反。用于评估的标准单元布局数据集本身是不平衡的，DRC 违反的网格显著少于合规网格。因此，为了评估性能，我们衡量商业工具检测到的 DRV 与我们生成的代码之间的精度（Precision）、召回率（Recall）和 F1 分数（F1）。这些指标特别适用于不平衡数据集，重点关注少数类（DRC 违反）的正确识别。这些指标的较高值表示更好的 DRC 代码。请注意，我们以 F1 分数作为主要指标，因为它通过平衡精度和召回率提供了对效果的全面视角。通过这种方式，我们可以检查我们的代码是否正确复制了商业工具的结果。最后，该功能为 Planner 提供性能报告，以进行推理，并为程序员提供调试信息。

图 [9](https://arxiv.org/html/2412.05311v1#S4.F9 "Figure 9 ‣ 4.1\. Prompting ‣ 4\. DRC-Coder ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent") 显示了该功能的示例。在性能报告中，我们给出了平均性能，并总结了假阴性和假阳性。对于每个假阴性（阳性）DRV 位置，我们首先将其分类为边界违反或间距违反。对于每个间距违反（边界违反）的 DRV，我们计算两个点之间的 x 和 y 距离（从该点到边界的距离）。最后，我们报告具有唯一 x、y 距离的 DRV，因为它们可能来自不同的设计规则条件。由于 LLM 的上下文长度限制以及长上下文触发的潜在幻觉问题，我们无法报告所有的 DRV。

在报告的最后，我们提供了目标和代理可以采取的可用行动。请注意，Planner 可以在下一轮生成中自由调用布局 DRC 分析，以进一步了解 DRV。例如，Planner 可以根据报告查询特定网格位置周围的 DRV 场景。

表 1\. 使用标准提示和我们的 DRC-Coder 在 GPT-4o（OpenAI，[2024](https://arxiv.org/html/2412.05311v1#bib.bib13)）和 Llama3（Dubey 等，[2024](https://arxiv.org/html/2412.05311v1#bib.bib5)）的 DRC 代码生成性能评估，涵盖了七个设计规则。表中展示了每种方法的精度（P）、召回率（R）和 F1 分数（F）。对于使用 GPT-4o 的 DRC-Coder，还包括调试迭代次数和运行时（秒）。

| LLM | Llama3 | GPT-4o |
| --- | --- | --- |
| 规则 | 标准提示 | DRC-Coder | 标准提示 | DRC-Coder |
| P | R | F | P | R | F | P | R | F | P | R | F | #迭代次数 | 运行时间（秒） |
| M0.S.1 | 0.841 | 0.710 | 0.745 | 0.795 | 1.000 | 0.870 | 0.789 | 0.527 | 0.620 | 1.000 | 1.000 | 1.000 | 3 | 325 |
| M0.S.2 | 0.657 | 0.489 | 0.554 | 0.696 | 0.817 | 0.722 | 0.657 | 0.489 | 0.554 | 1.000 | 1.000 | 1.000 | 2 | 121 |
| VIA0.S.1 | 0.646 | 0.403 | 0.483 | 1.000 | 1.000 | 1.000 | 0.659 | 1.000 | 0.769 | 1.000 | 1.000 | 1.000 | 2 | 133 |
| M1.S.1 | 0.540 | 1.000 | 0.680 | 1.000 | 1.000 | 1.000 | 0.645 | 0.402 | 0.482 | 1.000 | 1.000 | 1.000 | 3 | 354 |
| M1.S.2 | 0.000 | 0.000 | 0.000 | 0.151 | 0.583 | 0.234 | 0.582 | 0.450 | 0.490 | 1.000 | 1.000 | 1.000 | 2 | 152 |
| VIA1.S.1 | 0.075 | 0.725 | 0.133 | 0.149 | 1.000 | 0.255 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 | 1 | 45 |
| M2.S.1 | 0.220 | 1.000 | 0.356 | 1.000 | 1.000 | 1.000 | 0.500 | 0.500 | 0.500 | 1.000 | 1.000 | 1.000 | 3 | 343 |
| 平均值 | 0.425 | 0.618 | 0.421 | 0.684 | 0.914 | 0.726 | 0.690 | 0.624 | 0.631 | 1.000 | 1.000 | 1.000 | 2.3 | 210 |

## 5\. 实验结果

在本节中，我们首先详细介绍实验设置。然后，我们呈现DRC-Coder的评估结果及其消融研究。最后，我们介绍DRC-Coder的详细工作流程。

![参见说明文字](img/417a585766cd0929862793777fca5236.png)

图10\. DRC-Coder评估的选择性设计规则示意图。（1）规则M0.S.1：M0组件之间的间距，考虑垂直和水平方向。（2）规则M0.S.2：M0组件之间的水平间距。（3）规则VIA0.S.1：VIA0金属之间的间距。（4）规则M2.S.1：VIA1和M2组件之间的相互作用，考虑M2对VIA1的封装。

### 5.1\. 实验设置

开发平台。DRC-Coder是在Python语言下开发的，基于多智能体系统开发工具包AutoGen（Wu等， [2023](https://arxiv.org/html/2412.05311v1#bib.bib19)）。Planner和Programmer智能体，以及嵌入在两个工具函数中的VLMs，均由GPT-4o（OpenAI， [2024](https://arxiv.org/html/2412.05311v1#bib.bib13)）提供支持，使用OpenAI API版本2024-05-13。这意味着我们的DRC-Coder是无需训练的，因为我们没有对LLMs进行任何微调。

评估方法。我们评估了DRC-Coder在为小于3纳米技术节点开发DRC检查器代码方面的能力，特别是针对NVCell（Ren和Fojtik，[2021](https://arxiv.org/html/2412.05311v1#bib.bib16)；Ho等，[2023](https://arxiv.org/html/2412.05311v1#bib.bib6)）。我们的评估数据集，按照第[3节](https://arxiv.org/html/2412.05311v1#S3 "3\. Data Preparation ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")中描述的方式生成，包含了207个标准单元布局，并涵盖了7种不同的设计规则，如表[1](https://arxiv.org/html/2412.05311v1#S4.T1 "Table 1 ‣ 4.3\. Programmer ‣ 4\. DRC-Coder ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")所示。这些规则包含了所有可能发生的主要DRV类型，并且在NVCell的网格路由引擎中得到了考虑。此外，它们覆盖了从M0到M2的金属和通孔层，涵盖了标准单元布局工具中所有可用的路由层（Li等，[2019](https://arxiv.org/html/2412.05311v1#bib.bib11)；Van Cleeff等，[2019](https://arxiv.org/html/2412.05311v1#bib.bib17)；Lee等，[2020](https://arxiv.org/html/2412.05311v1#bib.bib10)）。

图[10](https://arxiv.org/html/2412.05311v1#S5.F10 "Figure 10 ‣ 5\. Experimental Results ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")展示了四个选定的设计规则。规则M0.S.1展示了M0组件的间距，而规则M0.S.2则显示了具有不同间距参数的水平间距。VIA0.S.1展示了VIA0金属之间的间距约束。规则M2.S.1代表了更复杂的多层交互，处理了VIA1和M2组件之间的间距。此规则引入了M2封闭的概念，即超出VIA1边界的M2金属，如图[10](https://arxiv.org/html/2412.05311v1#S5.F10 "Figure 10 ‣ 5\. Experimental Results ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")中蓝色VIA方块周围的浅紫色区域所示。M2封闭对于确保先进芯片设计中的可靠连接和可制造性至关重要。

尽管没有展示，M1.S.1和M1.S.2与M0.S.1和M0.S.2类似，但在垂直方向上的间距要求不同。VIA1.S.1类似于VIA0.S.1，但具有不同的间距参数。这个多样化的规则集使得我们能够全面评估DRC-Coder在处理单层间距规则和复杂的多层交互方面的能力。

为了量化DRC-Coder的性能，我们将生成的代码输出与商业工具的黄金DRC报告进行比较，并使用三个度量标准：精确度、召回率和F1分数，具体内容请参见第[4.3节](https://arxiv.org/html/2412.05311v1#S4.SS3 "4.3\. Programmer ‣ 4\. DRC-Coder ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")，其中F1分数是我们的主要度量标准。

基准。我们是首个专注于DRC编码生成问题并使用基于LLM-Agent的方法来解决这一问题的工作。因此，主要基准设置为标准提示法，使用图[6](https://arxiv.org/html/2412.05311v1#S2.F6 "图 6 ‣ 2.3\. 基于网格的DRC检查器 ‣ 2\. 前提 ‣ DRC-Coder: 使用LLM自主智能体生成自动化DRC检查代码")中的提示直接生成代码，而没有工具功能的反馈。DRC-Coder是一个具有多模态视觉能力的多智能体框架。我们还为消融实验设置了DRC-Coder的两个变种作为其他基准：（1）具有视觉能力的单智能体：仅使用程序员直接生成代码，（2）不具备视觉能力的多智能体：保留规划者和程序员，但不包括Foundry Rule和Layout DRV分析。

表2\. 使用两个DRC-Coder变种（无视觉能力的多智能体和有视觉能力的单智能体）评估DRC代码生成的性能：使用GPT-4o。P：精确度，R：召回率，F：F1得分

| LLM | GPT-4o |
| --- | --- |
| 设计规则 | 多智能体 | 单智能体 |
| 无视觉能力 | 有视觉能力 |
| P | R | F | P | R | F |
| M0.S.1 | 0.951 | 0.657 | 0.747 | 0.944 | 0.965 | 0.946 |
| M0.S.2 | 0.929 | 1.000 | 0.956 | 0.530 | 1.000 | 0.661 |
| VIA0.S.1 | 1.000 | 1.000 | 1.000 | 0.660 | 1.000 | 0.769 |
| M1.S.1 | 0.848 | 0.880 | 0.844 | 1.000 | 1.000 | 1.000 |
| M1.S.2 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 |
| VIA1.S.1 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 |
| M2.S.1 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 |
| 平均值 | 0.961 | 0.934 | 0.935 | 0.876 | 0.995 | 0.911 |

![参见说明](img/574c4d636691d40aa5ccbf6e13e36a27.png)

图11\. DRC-Coder生成和优化M0.S.1设计规则DRC代码的工作流程。该过程展示了规划者和程序员之间的迭代协作，利用各种工具调用（FoundryRuleAnalysis、LayoutDRVAnalysis、DRCCodeEval）逐步提高代码性能并消除假阳性和假阴性。请注意，图中使用伪代码表示生成的代码，以减少上下文长度。

### 5.2\. DRC-Coder的结果

评估结果见表[1](https://arxiv.org/html/2412.05311v1#S4.T1 "表 1 ‣ 4.3\. 程序员 ‣ 4\. DRC-Coder ‣ DRC-Coder: 使用LLM自主智能体生成自动化DRC检查代码")。我们使用GPT-4o（OpenAI，[2024](https://arxiv.org/html/2412.05311v1#bib.bib13)）的DRC-Coder，采用具有视觉能力的多智能体架构，在所有七个设计规则的评估中，精确度、召回率和F1得分均达到满分（1.000）。在不同规则类型下的一致表现突显了我们的方法在解读和转化复杂设计规则为准确DRC代码方面的鲁棒性。

相比之下，标准的提示方法在不同设计规则下的表现不尽如人意，平均F1值为0.631。虽然在某些规则上表现尚可，例如VIA1，但在其他规则，尤其是在召回率和F1分数方面，表现较差。这种不一致性表明了传统提示方法在处理复杂的DRC时的局限性。

总结来说，DRC-Coder的F1得分提高了37%。此外，DRC-Coder能够在平均2.3次调试迭代内完成编码，运行时间为210秒。因此，它可以极大加速DRC编码过程，而设计师通常需要花费数周时间才能编写出正确的DRC代码。

我们进一步展示了使用开源LLM Llama3（Dubey等，[2024](https://arxiv.org/html/2412.05311v1#bib.bib5)）的结果。与标准提示相比，我们的框架还可以实现42.2%的改进。然而，它无法像GPT-4o那样有效，表明GPT-4o在这一特定领域的DRC编码问题中具有更强的代理能力。

### 5.3\. 消融研究

消融研究结果显示在表[2](https://arxiv.org/html/2412.05311v1#S5.T2 "Table 2 ‣ 5.1\. Experiment Setup ‣ 5\. Experimental Results ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")中。该实验评估了视觉能力和多代理设置在DRC-Coder中的性能贡献。DRC-Coder的两个变体比标准提示方法在F1得分上提高了32.5%和30.7%（见表[1](https://arxiv.org/html/2412.05311v1#S4.T1 "Table 1 ‣ 4.3\. Programmer ‣ 4\. DRC-Coder ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")中的第一列）。然而，在某些设计规则上，它们的表现仍不及完整的DRC-Coder（见表[1](https://arxiv.org/html/2412.05311v1#S4.T1 "Table 1 ‣ 4.3\. Programmer ‣ 4\. DRC-Coder ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")中的第二列）。这些结果表明，视觉能力在解读某些设计规则中的重要性，以及我们的多代理设置在DRC编码任务的任务分解中的优势。

### 5.4\. DRC-Coder工作流案例研究

本节展示了DRC-Coder生成和完善设计规则M0.S.1的DRC代码的详细工作流案例研究。图[11](https://arxiv.org/html/2412.05311v1#S5.F11 "Figure 11 ‣ 5.1\. Experiment Setup ‣ 5\. Experimental Results ‣ DRC-Coder: Automated DRC Checker Code Generation Using LLM Autonomous Agent")提供了该过程的逐步可视化。

工作流从初始提示开始，触发 Planner 代理使用 FoundryRuleAnalysis 和 LayoutDRVAnalysis 工具函数分析设计规则（第 1-2 步）。这些分析提供了规则规范和潜在的 DRV 条件的洞见。基于这些信息，Planner 总结 DRV 分析并生成编写 DRC 代码的计划（第 3 步），包括边界和间距 DRV 检查。接着，Programmer 实现 DRC 代码（第 4 步）并调用 DRCCodeEval 工具（第 5 步）以获得代码性能并揭示改进空间。

在下一轮代码生成中，Planner 制定了改进计划（第 6 步），指出如何修改边界规则和间距条件。这指导 Programmer 进行代码调整（第 7 步）。这个迭代过程持续进行，每一轮都在改进代码性能，减少假阳性和假阴性（第 8-10 步）。请注意，在第 8 步中，代码没有假阴性。最后，当 DRCCodeEval 表示代码正确时，Planner 发送 TERMINATE 信号以结束代码生成过程。

这个示范表明，Planner 可以生成有效的计划以修改设计规则条件。此外，Programmer 可以遵循该计划并结合其最后生成的代码来生成改进版本。

## 6. 结论

在这项工作中，我们介绍了 DRC-Coder，这是第一个利用具有视觉能力的多代理系统进行自动化 DRC 代码生成的框架。我们的方法将 DRC 编码过程分解为解释和编程任务，利用两个大型语言模型（LLM）并集成视觉语言模型（VLM）有效处理多模态信息，包括文本描述、视觉插图和布局表示。此外，我们为 LLM 开发了三种专门的工具函数：铸造规则分析、布局 DRV 分析和 DRC 代码评估。这些功能使得自动推理和调试成为可能，显著增强了代码生成过程的稳健性。

我们的评估表明，DRC-Coder 显著优于标准提示技术，在用于亚 3 纳米技术节点的标准单元布局工具中，所有设计规则的 F1 分数均达到完美的 1.000。这表明，生成的 DRC 检查器成功地复制了商业工具的报告，提供了符合标准的 DRC 给布局工具。此外，DRC-Coder 大幅减少了从几天人工工作的编码时间，平均每个设计规则仅需四分钟，大大加速了技术迁移并降低了工程成本。请注意，DRC-Coder 可以推广到使用其他编程语言生成代码，例如 C++，以实现更高效的 DRC。

展望未来，DRC-Coder 可以扩展到广泛的 DRC 相关应用。例如，我们可以利用我们的图像分析功能，并在每个规划器的响应中包含人机交互反馈，从而实现一个 DRC 解释聊天机器人。我们还计划将我们的框架扩展到需要多模态推理的其他物理设计领域。最后，随着 DRC-Coder 解锁了 LLM 在电子设计自动化（EDA）中处理复杂工程任务的能力，我们希望能够激发未来在这一领域开发 LLM 代理的研究。

## 致谢

本工作部分由 NVIDIA 公司和 NSF 资助，资助号：2106828。

## 参考文献

+   （1）

+   Abdin 等人（2024）Marah Abdin 等人。2024年。Phi-3 技术报告：一款具有高度能力的语言模型，能在手机本地运行。*arXiv 预印本 arXiv:2404.14219*（2024年）。

+   Chen 等人（2022）Bei Chen、Fengji Zhang、Anh Nguyen、Daoguang Zan、Zeqi Lin、Jian-Guang Lou 和 Weizhu Chen。2022年。Codet：通过生成的测试进行代码生成。*arXiv 预印本 arXiv:2207.10397*（2022年）。

+   Chung 等人（2022）Hyung Won Chung 等人。2022年。规模化指令微调语言模型。*arXiv 预印本 arXiv:2210.11416*（2022年）。

+   Dubey 等人（2024）Abhimanyu Dubey、Abhinav Jauhri、Abhinav Pandey、Abhishek Kadian、Ahmad Al-Dahle、Aiesha Letman、Akhil Mathur、Alan Schelten、Amy Yang、Angela Fan 等人。2024年。The llama 3 模型群。*arXiv 预印本 arXiv:2407.21783*（2024年）。

+   Ho 等人（2023）Chia-Tung Ho、Alvin Ho、Matthew Fojtik、Minsoo Kim、Shang Wei、Yaguang Li、Brucek Khailany 和 Haoxing Ren。2023年。NVCell 2：具有格栅图可布线模型的先进节点中面向布线的标准单元布局。见于 *2023年国际物理设计研讨会论文集*，44–52。

+   Ho 和 Ren（2024）Chia-Tung Ho 和 Haoxing Ren。2024年。面向标准单元布局设计优化的大型语言模型。见于 *IEEE 国际 LLM 辅助设计研讨会论文集*。

+   Ho 等人（2024）Chia-Tung Ho、Haoxing Ren 和 Brucek Khailany。2024年。VerilogCoder：具有基于图的规划和基于抽象语法树（AST）的波形追踪工具的自主 Verilog 编码代理。*arXiv 预印本 arXiv:2408.08927*（2024年）。

+   LangChain-AI（2024）LangChain-AI。2024年。LangChain。 [https://github.com/langchain-ai/langchain](https://github.com/langchain-ai/langchain)

+   Lee 等人（2020）Daeyeal Lee、Dongwon Park、Chia-Tung Ho、Ilgweon Kang、Hayoung Kim、Sicun Gao、Bill Lin 和 Chung-Kuan Cheng。2020年。SP&R：基于 SMT 的标准单元合成的同时布图和布线方法，适用于先进节点。*IEEE 集成电路与系统计算机辅助设计期刊* 40，10（2020年），2142–2155。

+   Li 等人（2019）Yih-Lang Li、Shih-Ting Lin、Shinichi Nishizawa、Hong-Yan Su、Ming-Jie Fong、Oscar Chen 和 Hidetoshi Onodera。2019年。NCTUcell：一种基于 DDA 的 FinFET 结构单元库生成器，具有隐式可调的网格图。见于 *第56届年会设计自动化会议论文集 2019*，1–6。

+   Nijkamp 等人（2023）Erik Nijkamp、Hiroaki Hayashi、Caiming Xiong、Silvio Savarese 和 Yingbo Zhou。2023年。《Codegen2：关于在编程和自然语言上训练大型语言模型的经验》。*arXiv 预印本 arXiv:2305.02309*（2023年）。

+   OpenAI（2024）OpenAI。2024年。《GPT-4o》。 [https://platform.openai.com/docs/models/gpt-4o](https://platform.openai.com/docs/models/gpt-4o)

+   Radford 等人（2019）Alec Radford、Jeffrey Wu、Rewon Child、David Luan、Dario Amodei、Ilya Sutskever 等人。2019年。《语言模型是无监督的多任务学习者》。*OpenAI 博客* 1，8（2019年），9。

+   Raffel 等人（2020）Colin Raffel、Noam Shazeer、Adam Roberts、Katherine Lee、Sharan Narang、Michael Matena、Yanqi Zhou、Wei Li 和 Peter J Liu。2020年。《通过统一的文本到文本转换器探索迁移学习的极限》。*机器学习研究杂志* 21，1（2020年），5485–5551。

+   Ren 和 Fojtik（2021）Haoxing Ren 和 Matthew Fojtik。2021年。《NVCell：使用强化学习在先进技术节点中的标准单元布局》。在*2021年第58届ACM/IEEE设计自动化大会（DAC）*上。IEEE，1291–1294。

+   Van Cleeff 等人（2019）Pascal Van Cleeff、Stefan Hougardy、Jannik Silvanus 和 Tobias Werner。2019年。《Bonncell：7纳米时代的自动化单元布局》。*IEEE计算机辅助设计与集成电路与系统杂志* 39，10（2019），2872–2885。

+   Wang 等人（2024）Lei Wang、Chen Ma、Xueyang Feng、Zeyu Zhang、Hao Yang、Jingsen Zhang、Zhiyuan Chen、Jiakai Tang、Xu Chen、Yankai Lin 等人。2024年。《基于大型语言模型的自主智能体调查》。*计算机科学前沿* 18，6（2024年），1–26。

+   Wu 等人（2023）Qingyun Wu、Gagan Bansal、Jieyu Zhang、Yiran Wu、Shaokun Zhang、Erkang Zhu、Beibin Li、Li Jiang、Xiaoyun Zhang 和 Chi Wang。2023年。《Autogen：通过多智能体对话框架实现下一代大型语言模型应用》。*arXiv 预印本 arXiv:2308.08155*（2023年）。

+   Yang 等人（2024）John Yang、Carlos E Jimenez、Alexander Wettig、Kilian Lieret、Shunyu Yao、Karthik Narasimhan 和 Ofir Press。2024年。《Swe-agent：代理计算机接口启用自动化软件工程》。*arXiv 预印本 arXiv:2405.15793*（2024年）。

+   Yao 等人（2022a）Shunyu Yao、Howard Chen、John Yang 和 Karthik Narasimhan。2022a年。《Webshop：面向基于语言代理的可扩展现实世界网页交互》。*神经信息处理系统进展* 35（2022年），20744–20757。

+   Yao 等人（2022b）Shunyu Yao、Jeffrey Zhao、Dian Yu、Nan Du、Izhak Shafran、Karthik Narasimhan 和 Yuan Cao。2022b年。《React：在语言模型中协同推理和行为》。*arXiv 预印本 arXiv:2210.03629*（2022年）。

+   Zhang 等人（2024）Jingyi Zhang、Jiaxing Huang、Sheng Jin 和 Shijian Lu。2024年。《视觉任务的视觉-语言模型：一项调查》。*IEEE模式分析与机器智能学报*（2024年）。

+   Zhu 等人（2023）Binwu Zhu、Xinyun Zhang、Yibo Lin、Bei Yu 和 Martin Wong。2023年。DRC-SG 2.0：通过关键信息提取生成高效的设计规则检查脚本。*ACM 电子系统设计自动化事务* 28卷，第5期（2023），第1-18页。
