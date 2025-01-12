<!--yml

类别：未分类

日期：2025-01-11 11:54:45

-->

# 物理信息驱动的LLM代理用于电力电子系统中的自动化调制设计

> 来源：[https://arxiv.org/html/2411.14214/](https://arxiv.org/html/2411.14214/)

Junhua Liu Forth AISingapore [j@forth.ai](mailto:j@forth.ai) ,  Fanfan Lin 浙江大学China [fanfanlin@intl.zju.edu.cn](mailto:fanfanlin@intl.zju.edu.cn) ,  Xinze Li 阿肯色大学USA [xinzel@uark.edu](mailto:xinzel@uark.edu) ,  Kwan Hui Lim 新加坡科技设计大学Singapore [kwanhui˙lim@sutd.edu.sg](mailto:kwanhui%CB%99lim@sutd.edu.sg)  和  Shuai Zhao 奥尔堡大学Denmark [szh@energy.aau.dk](mailto:szh@energy.aau.dk)(2024)

###### 摘要。

基于LLM的自主代理在解决复杂工业任务中表现出色。然而，在追求碳中和和高性能可再生能源系统的过程中，现有的AI辅助设计自动化在可解释性、可扩展性和可用性方面面临着显著的限制。为了解决这些挑战，我们提出了LP-COMDA，一种基于LLM的、物理信息驱动的自主代理，能够在最少的人类监督下自动进行电力电子系统中电力变换器的调制设计。与传统的AI辅助方法不同，LP-COMDA包含一个基于LLM的规划器，通过用户友好的聊天界面收集并验证设计规范。然后，规划器与物理信息驱动的设计和优化工具协调，迭代地生成和完善调制设计。通过聊天界面，LP-COMDA提供了一个可解释的设计过程，展示解释和图表。实验表明，LP-COMDA在所有基准方法中表现最优，与第二最佳基准方法相比，在标准平均绝对误差上减少了63.2%。此外，20位专家的实证研究表明，使用LP-COMDA的设计时间比传统方法快超过33倍，显示出它在设计效率上的显著提升。

代理AI，PINN，LLM，电力转换^†^†版权：cc^†^†会议：确保输入来自您的版权确认邮件中的正确会议标题；2025年8月1日至6日；加拿大多伦多^†^†ccs：信息系统 语言模型^†^†ccs：计算方法 智能代理^†^†ccs：硬件 电力转换

## 1\. 引言

随着全球变暖导致的气温逐年上升，迫切需要将我们的经济转向碳中和，尤其是在电力生产领域。可再生能源资源（RES）在电力生产中的大规模渗透至关重要。电力电子系统（PES）在将RES集成到现代电网中的作用不可或缺，主要体现在三个方面（Lin，[2022](https://arxiv.org/html/2411.14214v1#bib.bib23)）。首先，PES中的电力转换器将太阳能电池板或风力涡轮机的直流电（DC）输出转换为交流电（AC），从而实现RES与交流电网的无缝集成。其次，PES在电网的峰值和非峰值期间管理电力，指导能源存储单元的充电和放电过程。最后，PES通过诸如电压调节和无功功率补偿等功能，保持电网的稳定。要实现预期的效果，必须精心调节PES中的电力转换器，这涉及到控制其开关操作，以管理输出电压或电流。

可再生能源系统（RES）的日益普及以及多资源电力系统规模的不断扩大，使得电力电子系统（PES）中电力转换器的调制设计变得更加复杂。与电网连接的能源来源多样化加剧了系统的复杂性。随着更多转换器子系统的互联，调制设计问题的维度急剧增加。因此，在这种复杂性下，通过手动计算找到优化的设计解决方案变得不可行。随着全球各大城市越来越多地采用多资源电力系统，调制设计必须定制化，以适应特定的地理位置和应用目标。有时，这些目标可能会发生冲突，进一步增加了设计的复杂性。

最近，人工智能（AI）和大型语言模型（LLMs）的进展为通过自动化解决这些调制设计挑战提供了潜在的解决方案。新兴的研究提出了以人工智能为中心的调制设计技术。例如，近期的研究探讨了利用具有上下文学习功能的大型语言模型（LLM）进行调制设计（Lin 等，[2024](https://arxiv.org/html/2411.14214v1#bib.bib24)）。极端梯度提升算法（XGBoost）被用来构建双有源桥（DAB）转换器的三相移调制策略的替代模型，训练数据来源于仿真工具或硬件原型（Lin 等，[2023a](https://arxiv.org/html/2411.14214v1#bib.bib26)）。随后，差分进化算法与XGBoost进行迭代协作，直到识别出优化的调制参数。同样，Q学习算法作为一种传统的强化学习技术，也可以通过离线训练来推导优化的调制参数（Tang 等，[2020](https://arxiv.org/html/2411.14214v1#bib.bib43)）。然而，这些方法也存在若干缺点：

1.  (1)

    AI模型的训练通常是数据密集型的，需要通过模拟或硬件实验进行大量且繁琐的数据收集。

1.  (2)

    对于复杂任务，训练或微调大型模型在计算上也非常密集，并且消耗大量能源进行训练和服务。

1.  (3)

    将AI模型作为无法解释的黑箱部署，严重限制了它们在工业中的应用。

1.  (4)

    当前的技术侧重于特定的调制策略或预设的设计目标，这限制了这种自动化的可扩展性。

1.  (5)

    现有的方法在整个设计过程中需要大量的人类参与，导致设计过程效率低下。

近年来，像GPT-4（OpenAI，[2023](https://arxiv.org/html/2411.14214v1#bib.bib34)）、Palm（Anil等，[2023](https://arxiv.org/html/2411.14214v1#bib.bib3)）和LLaMa（Touvron等，[2023](https://arxiv.org/html/2411.14214v1#bib.bib44)）等大型语言模型在自然语言理解和生成方面展现了卓越的性能。此外，近期的研究提出了基于LLM的自主代理，拓展了LLM的能力，从仅仅执行推理和生成内容，扩展到执行行动和控制（Wang等，[2024](https://arxiv.org/html/2411.14214v1#bib.bib46)）。

在我们的研究中，我们探讨了使用基于LLM的自主代理在电力电子系统中进行下游任务自动化的潜力。具体来说，我们介绍了LP-COMDA，一个基于LLM的物理信息化电力转换器调制设计自动化系统，旨在帮助用户以最少的人类监督，生产高质量的调制设计。

LP-COMDA首先使用基于大语言模型（LLM）的规划器处理用户需求并生成一组设计规范。随后，代理调用一组系统设计工具，这些工具由物理信息化代理模型和优化算法组成。在设计过程中，这些工具通过迭代过程推导出与用户设计规范相匹配的最优调制参数。最后，最优设计参数和性能指标将返回并可视化，以便用户实现可解释的设计过程。

本文的贡献总结如下：

+   •

    我们提出了LP-COMDA，一个基于LLM的物理信息化自主代理，简化了电力转换器调制设计过程。综合实验显示，LP-COMDA表现出色，在低数据场景下，误差比最好的基线模型低63.2%，在高数据场景下，误差比最好的基线模型低23.7%。

+   •

    我们提出了一种分层的物理信息化代理模型，作为电力转换器调制的设计自动化工具，能够在数据稀缺的极端场景下，准确预测电力转换器的性能，且优于基准模型。

+   •

    我们与20位专家进行的用户研究表明，使用LP-COMDA的设计时间比传统方法快33倍以上，展示了在实际操作中具有极高的效率，并且需要极少的人工监督。

![参见说明](img/dd404558b66439fc7cbfaf48b5cf9db0.png)

图1\. 电力转换器应用示例：DAB（双向有源桥）转换器充当DC-DC电力网的直流变压器，提供电气隔离，并调节直流母线之间的电力和电压。直接调节其开关会影响系统的运行性能，包括电力传输效率、电压调节和互联母线的稳定性。

![参见说明](img/561fec3f3fea7a1b20f9e01fb61ae7f0.png)

图2\. LP-COMDA的系统架构：工程师通过连接到其规划器的聊天界面向LP-COMDA提供设计需求。一旦确定了完整的需求，规划器协调并调用工具集中的工具，逐步生成调制设计，无需人工监督。设计完成后，规划器在聊天界面上显示最终结果和可解释的过程。

## 2\. 问题陈述

### 2.1\. 电力转换器相位移调制

在可再生能源系统领域，电力转换器需要与其他组件协调工作，以确保最佳功能。图[1](https://arxiv.org/html/2411.14214v1#S1.F1 "图1 ‣ 1\. 介绍 ‣ 基于物理信息的LLM-Agent在电力电子系统中的自动调制设计")展示了某种电力转换器的经典应用，即双向有源桥（DAB）转换器。该转换器充当DC变压器，连接不同的直流母线，便于利用或存储由可再生能源系统（RES）产生的电力（Lin等， [2021](https://arxiv.org/html/2411.14214v1#bib.bib25)）。相位移调制技术用于调节电流流动，确保高效的电力传输。通过调整不同功率开关组的时序来实现这一点，图[1](https://arxiv.org/html/2411.14214v1#S1.F1 "图1 ‣ 1\. 介绍 ‣ 基于物理信息的LLM-Agent在电力电子系统中的自动调制设计")中的$S_{1}$到$S_{8}$表示这些开关组。表[1](https://arxiv.org/html/2411.14214v1#S2.T1 "表1 ‣ 2.1\. 电力转换器相位移调制 ‣ 2\. 问题陈述 ‣ 基于物理信息的LLM-Agent在电力电子系统中的自动调制设计")详细说明了与DAB转换器相关的不同相位移调制策略，突出了它们各自的自由度（DoF）和可调参数（Hou和Li， [2019](https://arxiv.org/html/2411.14214v1#bib.bib12)）。值得注意的是，$<S_{1},S_{5}>$的可调参数表示$S_{1}$和$S_{5}$之间的相位差。

|

&#124; 相位移 &#124;

&#124; 策略 &#124;

| 自由度数 | $<S_{1},S_{3}>$ | $<S_{1},S_{5}>$ | $<S_{5},S_{7}>$ |
| --- | --- | --- | --- | --- |

|

&#124; 单相 &#124;

&#124; 移相 &#124;

| 1 | 0 | $D_{o}$ | 0 |
| --- | --- | --- | --- |

|

&#124; 双相 &#124;

&#124; 移相 &#124;

| 2 | $D_{i}$ | $D_{o}$ | $D_{i}$ |
| --- | --- | --- | --- |

|

&#124; 扩展相 &#124;

&#124; 移相 1 &#124;

| 2 | $D_{i}$ | $D_{o}$ | 0 |
| --- | --- | --- | --- |

|

&#124; 扩展相 &#124;

&#124; 移相 2 &#124;

| 2 | 0 | $D_{o}$ | $D_{i}$ |
| --- | --- | --- | --- |

|

&#124; 三相 &#124;

&#124; 移相 &#124;

| 3 | $D_{1}$ | $D_{o}$ | $D_{2}$ |
| --- | --- | --- | --- |
| 混合 | 2 | $D_{i}$ | $D_{o}$ | 0 |
| 相位 | 2 | 0 | $D_{o}$ | $D_{i}$ |
| 移相 | 2 | $D_{i}$ | $D_{o}$ | $D_{i}$ |

表1. 具有1至3自由度（DoF）的相移调制策略。$<S_{x},S_{y}>$定义了$S_{x}$和$S_{y}$之间的相移。

在设计调制策略的参数时，用户需要定义三项与应用要求一致的基本规范：

1.  (1)

    转换器的预期工作条件：这涉及到规定额定功率、额定输入电压（电流）和额定输出电压（电流），以及它们的实际工作值。

1.  (2)

    选定的调制策略：选择调制策略时应考虑控制复杂度的可接受水平，以及实现特定策略所需性能的可行性。

1.  (3)

    性能目标：不同的应用场景对性能目标有不同的优先级，这些优先级直接影响调制设计。实际应用中常见的目标包括功率效率、功率密度、零电压开关、零电流开关、电流应力、动态响应、电磁干扰、稳定性，以及与其他子系统的兼容性等。

### 2.2. 问题陈述

给定一组功率转换器的工作条件（额定功率$P_{r}$，实际功率$P_{a}$，额定输入电压$V_{1r}$，额定输出电压$V_{2r}$，实际工作输出电压$V_{2a}$），选定的调制策略$S$和优先的性能目标$O$，目标是为选定的调制策略$S$设计调制参数，以实现$O$的最佳性能。该设计系统预计具有以下特点：

+   •

    自动设计结果生成，消除了用户进行复杂分析或计算的需要。

+   •

    精确的设计结果，转化为在实际场景中卓越的操作性能。

+   •

    良好的可用性和可解释性，以加速工程设计和科学发现。

+   •

    设计系统的简单设置，最大限度地减少对大量数据资源的需求。

## 3. 相关工作

### 3.1. 基于LLM的自主代理

大型语言模型（LLM）的发展为通过简单的自然语言查询自动化工程和自然科学中的各种过程开辟了巨大的潜力。大量研究探讨了LLM在推理和形式化系统应用中的能力，如编程助手（Ross 等，[2023](https://arxiv.org/html/2411.14214v1#bib.bib37)；McNutt 等，[2023](https://arxiv.org/html/2411.14214v1#bib.bib31)），多语言任务（Xu 等，[2024](https://arxiv.org/html/2411.14214v1#bib.bib50)；Liu 等，[2024b](https://arxiv.org/html/2411.14214v1#bib.bib29)），以及数学问题求解（Shakarian 等，[2023](https://arxiv.org/html/2411.14214v1#bib.bib38)；Shi 等，[2024](https://arxiv.org/html/2411.14214v1#bib.bib40)）。

为了应对更复杂的工业任务，近期的研究提出了基于大语言模型（LLM）自主代理（LLM-agents）的使用，这扩展了LLM的能力（Wang 等，[2024](https://arxiv.org/html/2411.14214v1#bib.bib46)）。LLM被部署作为规划核心，以理解用户请求并调用适当的工具来执行任务（Yao 等，[2024](https://arxiv.org/html/2411.14214v1#bib.bib52)）。除了规划模块，LLM-agents还包括其他组件，如记忆和工具。通过高级技术如检索增强生成（RAG）（Liu 等，[2024b](https://arxiv.org/html/2411.14214v1#bib.bib29)；Xu 等，[2024](https://arxiv.org/html/2411.14214v1#bib.bib50)），LLM-agents成为解决复杂任务的强大工具，几乎不需要任何人工指导。

近期的研究报告了LLM-agents在工业应用中的出色表现。例如，Cao 和 Lee 利用LLM和阶段-步骤提示生成跨领域的机器人任务（Cao 和 Lee，[2023](https://arxiv.org/html/2411.14214v1#bib.bib6)）。Li 等提出了一种用于战略性和互动决策的代理框架（Li 等，[2024b](https://arxiv.org/html/2411.14214v1#bib.bib21)）。基于LLM的代理在其他特定领域任务中也表现出色，如时空战略规划（Li 等，[2024a](https://arxiv.org/html/2411.14214v1#bib.bib22)；Liu 等，[2024a](https://arxiv.org/html/2411.14214v1#bib.bib28)；Xiang 等，[2024](https://arxiv.org/html/2411.14214v1#bib.bib47)），金融交易（Zhang 等，[2024b](https://arxiv.org/html/2411.14214v1#bib.bib55)；Koa 等，[2024](https://arxiv.org/html/2411.14214v1#bib.bib18)，[2024](https://arxiv.org/html/2411.14214v1#bib.bib18)；Zhang 等，[2024a](https://arxiv.org/html/2411.14214v1#bib.bib54)；Gao 等，[2023](https://arxiv.org/html/2411.14214v1#bib.bib9)），以及与计算机和智能手机的日常互动（Lai 等，[2024](https://arxiv.org/html/2411.14214v1#bib.bib19)；Xing 等，[2024](https://arxiv.org/html/2411.14214v1#bib.bib49)）。

![请参阅说明](img/d7c5aae8c6f51e654394fea490754143.png)

图3. 所提出的代理模型由两个物理信息神经网络组成，分别是用于开关级建模以学习开关行为的ModNet和用于系统级建模以学习电路物理的CirNet。该层次结构提高了功率转换器在建模开关复杂行为方面的整体准确性。

### 3.2. 物理信息神经网络

传统的神经网络通常数据需求大且缺乏可解释性。通过将物理原理融入神经网络（Raissi，[2018](https://arxiv.org/html/2411.14214v1#bib.bib36)），物理信息神经网络（PINNs）的研究显示，在克服这些挑战方面取得了可喜的成果。

PINN包含了多种方法，将物理信息（PI）组件融入深度学习流程，例如损失函数、参数初始化和神经网络架构（Huang和Wang，[2022](https://arxiv.org/html/2411.14214v1#bib.bib14)）。因此，神经网络能够学习由物理规律支配的潜在特征。

最近的研究还增强了PINN的收敛性、稳定性和准确性，取得了显著的改进（Kang等，[2023](https://arxiv.org/html/2411.14214v1#bib.bib17)；Yang等，[2023](https://arxiv.org/html/2411.14214v1#bib.bib51)；Wandel等，[2022](https://arxiv.org/html/2411.14214v1#bib.bib45)）。PINN在广泛的应用中取得了成功。一些例子包括预测交通状况和建模交通流（Shi等，[2021](https://arxiv.org/html/2411.14214v1#bib.bib39)；Ji等，[2022](https://arxiv.org/html/2411.14214v1#bib.bib15)），确定用于电力转换器健康监测的电路参数（Zhao等，[2022](https://arxiv.org/html/2411.14214v1#bib.bib56)），计算电力系统中的功率流（Hu等，[2020](https://arxiv.org/html/2411.14214v1#bib.bib13)），模拟湖泊中的温度动态（Jia等，[2021](https://arxiv.org/html/2411.14214v1#bib.bib16)），管理实时水库气体生产（Mudunuru等，[2020](https://arxiv.org/html/2411.14214v1#bib.bib32)），设计量子电路（Liu等，[2021](https://arxiv.org/html/2411.14214v1#bib.bib30)；Guo等，[2019](https://arxiv.org/html/2411.14214v1#bib.bib10)），以及模拟城市环境中的湍流流动（Xiao等，[2019](https://arxiv.org/html/2411.14214v1#bib.bib48)）。这些应用展示了PINNs通过将物理知识融入深度学习，展现出在众多领域中的卓越表现。

## 4. 方法论

### 4.1. 智能代理AI系统

图[2](https://arxiv.org/html/2411.14214v1#S1.F2 "Figure 2 ‣ 1\. Introduction ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")展示了所提出的LP-COMDA智能代理AI系统的架构。前端包括一个聊天界面和与GPT-4作为推理引擎的API交互，而后端包括代理的工具，即优化算法和基于PINN的代理模型。在工作流程方面，LP-COMDA首先从用户那里收集设计规范，并将其传递给基于LLM的推理引擎（GPT-4）进行规划。随后，代理调用后端工具，代理模型生成性能指标，并传递给优化算法，以搜索最优的调制参数。后端将返回一组最优设计参数给前端，前端展示并以图表的形式可视化，以便于解释性和可读性。

### 4.2. 物理信息驱动的电力电子建模

电力电子系统由于半导体器件的复杂开关行为和能量储存组件的耦合，具有高度的非线性。通过非线性时变状态空间方程给出的电力电子系统的通用表示如公式[1](https://arxiv.org/html/2411.14214v1#S4.E1 "In 4.2\. Physics-Informed Power Electronics Modeling ‣ 4\. Methodology ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")所示，其中$\theta$是电路参数，$u(t)$和$x(t)$分别是状态变量和输入变量，$g(\cdot)$是控制状态转移行为的通用非线性函数，$h(\cdot)$表示输入物理定律。

总体而言，所提出的代理模型由两个层次化的PINNs组成，一个是用于开关级建模的ModNet，学习开关行为，另一个是用于系统级建模的CirNet，学习电路物理。ModNet被训练以学习中间波形$v_{p}$和$v_{s}$，而CirNet推断电路中的关键特征波形，包括$i_{L}$、$v_{c1}$、$v_{c2}$，通过这些波形可以评估操作性能，如电流应力、软开关范围和效率。如图[1](https://arxiv.org/html/2411.14214v1#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")所示，$v_{p}$和$v_{s}$分别表示主桥和副桥的交流端电压，而$i_{L}$、$v_{c1}$和$v_{c2}$分别表示电感电流、输入电容电压和输出电容电压。

| (1) |  | $\frac{\partial u(t)}{\partial t}=g(x(t);\theta)u(t)+h(x(t);\theta)$ |  |
| --- | --- | --- | --- |

ModNet用于开关行为的开关级建模：在半导体开关建模中，非理想等效电路考虑了寄生电感、电容和电阻。电路元件之间的复杂相互作用会导致非平凡的振荡和过冲，从而影响功率转换器的整体运行性能。为了解决这个问题，提出通过物理信息网络（ModNet）来建模开关行为。ModNet由多个带有层归一化（LN-GRU）的门控递归单元（GRU）层组成。LN-GRU的时间特性非常适合调制波形预测的任务。如图[3](https://arxiv.org/html/2411.14214v1#S3.F3 "图 3 ‣ 3.1\. 基于LLM的自主代理 ‣ 3\. 相关工作 ‣ 物理信息LLM-Agent在电力电子系统中自动调制设计中的应用")部分（1）和公式[2](https://arxiv.org/html/2411.14214v1#S4.E2 "在4.2\. 物理信息电力电子建模 ‣ 4\. 方法论 ‣ 物理信息LLM-Agent在电力电子系统中自动调制设计中的应用")中所示，ModNet根据前一个时间戳$t_{k-1}$和隐藏状态的信息推断出 $v_{p}(t_{k})$ 和 $v_{s}(t_{k})$，并将开关周期内的所有中间预测结果存储以供训练。ModNet的训练利用了两种损失：（1）基于物理信息的损失，有助于开关同步；（2）基于示波器捕获的实验数据的损失，有助于模型捕捉实际操作中的振荡和过冲。

| (2) |  | $\displaystyle(\hat{v}_{p}(t_{k}),\hat{v}_{s}(t_{k}))=\operatorname{ModNet}(% \hat{v}_{p}(t_{k-1}),\hat{v}_{s}(t_{k-1});\theta_{mod})$ |  |
| --- | --- | --- | --- |

其中 $\hat{v}_{p}(t_{k})$ 和 ${\hat{v}_{s}(t_{k})}$ 表示在给定前一时刻电压 $\hat{v}_{p}(t_{k-1})$ 和 ${\hat{v}_{s}(t_{k-1})}$ 的情况下，ModNet 对 $v_{p}$ 和 $v_{s}$ 在时刻 $t_{k}$ 的预测值。

| (3) |  | $L\frac{\partial i_{L}(t)}{\partial t}=-R_{L}i_{L}(t)+v_{p}(t)-nv_{s}(t)$ |  |
| --- | --- | --- | --- |

其中 $L$、$R_{L}$ 和 $n$ 分别表示磁性元件的特性：漏感、等效电感电阻和变压器的匝数比。$i_{L}(t)$ 是表征主回路性能的关键波形，如功率传输、效率、热行为、软开关、电磁干扰等。

| (4) |  | $\displaystyle C_{1}\frac{\partial^{2}v_{C1}(t)}{\partial^{2}t}=$ | $\displaystyle-\left(\frac{\partial s_{pri}(t)}{\partial t}-\frac{s_{pri}(t)R_{% L}}{L}\right)i_{L}(t)+$ |  |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle\frac{\partial i_{1}(t)}{\partial t}-\frac{s_{pri}(t)}{L}\left(v_% {p}(t)-nv_{s}(t)\right)$ |  |
| (5) |  | $\displaystyle C_{2}\frac{\partial^{2}v_{C2}(t)}{\partial^{2}t}=$ | $\displaystyle n\left(\frac{\partial s_{sec}(t)}{\partial t}-\frac{s_{sec}(t)R_% {L}}{L}\right)i_{L}(t)-$ |  |
|  |  | $\displaystyle\frac{\partial i_{2}(t)}{\partial t}+\frac{ns_{sec}(t)}{L}\left(v% _{p}(t)-nv_{s}(t)\right)$ |  |

其中$C_{1}$和$C_{2}$分别表示输入和输出电容，$s_{pri}(t)$和$s_{sec}(t)$是描述电路开关行为的函数。$v_{C1}$和$v_{C2}$在与其他电力转换电路互连时，控制稳定性、鲁棒性和电能质量。

CirNet用于电路物理的系统级建模：随后，系统级建模通过物理信息电路网络CirNet完成。CirNet通过改造一个递归型LN-GRU网络，将电路物理规律编码到其固有的特征空间中。以非共振DAB变换器为例，主要的状态波形为$i_{L}$、$v_{C1}$和$v_{C2}$。通过利用基尔霍夫定律、法拉第定律和高斯定律，系统级动态方程在方程[3](https://arxiv.org/html/2411.14214v1#S4.E3 "In 4.2\. Physics-Informed Power Electronics Modeling ‣ 4\. Methodology ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")至[5](https://arxiv.org/html/2411.14214v1#S4.E5 "In 4.2\. Physics-Informed Power Electronics Modeling ‣ 4\. Methodology ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")中得出，其中电气符号在图[1](https://arxiv.org/html/2411.14214v1#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")中给出。方程[3](https://arxiv.org/html/2411.14214v1#S4.E3 "In 4.2\. Physics-Informed Power Electronics Modeling ‣ 4\. Methodology ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")描述了高频交流电流$i_{L}(t)$的动态，方程[4](https://arxiv.org/html/2411.14214v1#S4.E4 "In 4.2\. Physics-Informed Power Electronics Modeling ‣ 4\. Methodology ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")和方程[5](https://arxiv.org/html/2411.14214v1#S4.E5 "In 4.2\. Physics-Informed Power Electronics Modeling ‣ 4\. Methodology ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")呈现了输入和输出电容的电气行为，这些行为遵循二阶微分方程。

为了预测关键的状态波形，CirNet将ModNet的输出作为输入，并基于前一个状态的预测迭代推断下一个状态，如图[3](https://arxiv.org/html/2411.14214v1#S3.F3 "Figure 3 ‣ 3.1\. LLM-based Autonomous Agents ‣ 3\. Related work ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")部分(3)和方程[7](https://arxiv.org/html/2411.14214v1#S4.E7 "In 4.2\. Physics-Informed Power Electronics Modeling ‣ 4\. Methodology ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")所示。

就CirNet的训练而言，与ModNet类似，基于物理信息的损失$l_{p}$和基于实验数据的损失$l_{d}$都被部署，如公式[7](https://arxiv.org/html/2411.14214v1#S4.E7 "在4.2节物理信息驱动的功率电子建模 ‣ 4. 方法 ‣ 物理信息驱动的LLM-Agent用于功率电子系统中的自动调制设计")所示，其中$\lambda_{d}$和$\lambda_{p}$是它们的损失因子。在$l_{p}$中，电路物理动态通过微分方程[3](https://arxiv.org/html/2411.14214v1#S4.E3 "在4.2节物理信息驱动的功率电子建模 ‣ 4. 方法 ‣ 物理信息驱动的LLM-Agent用于功率电子系统中的自动调制设计")到[5](https://arxiv.org/html/2411.14214v1#S4.E5 "在4.2节物理信息驱动的功率电子建模 ‣ 4. 方法 ‣ 物理信息驱动的LLM-Agent用于功率电子系统中的自动调制设计")的表达被嵌入到物理学习的损失函数中。

在$l_{d}$中，硬件实验中的几个电感电流$i_{L}$数据点作为真实值，其中数据点表示为$\left\{i_{L,j}^{*}\left(t_{1}\right),\ldots,i_{L,j}^{*}\left(t_{T}\right)% \right\}_{j}^{N}$。波形预测被保存用于一个开关周期，以进行性能评估。

| (6) |  | $\displaystyle\hat{i}_{L}(t_{k+1})=\operatorname{CirNet}($ | $\displaystyle\hat{i}_{L}(t_{k}),\operatorname{ModNet}($ |  |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle\hat{v}_{p}(t_{k}),\hat{v}_{s}(t_{k});\theta_{{mod}});\theta_{cir})$ |  |
| (7) |  |  | $\displaystyle l_{Cir}=\lambda_{d}l_{d}+\lambda_{p}l_{p}$ |  |
|  |  | $\displaystyle=\frac{\lambda_{d}}{NT}\sum_{j}\sum_{k}\left[\hat{i}_{L,j}\left(t% _{k+1}\right)-i_{L,j}{}^{*}\left(t_{k+1}\right)\right]^{2}+$ |  |
|  |  | $\displaystyle\frac{\lambda_{p}}{NT}\sum_{j}\sum_{k}\left[\begin{array}[]{l}L% \left(\hat{i}_{L,j}\left(t_{k+1}\right)-i_{L,j}{}^{*}\left(t_{k}\right)\right)% -\\ (\hat{v}_{p,j}\left(t_{k+1}\right)-n\hat{v}_{s,j}\left(t_{k+1}\right)-\\ R_{L}i_{Lj}{}^{*}\left(t_{k}\right))\Delta t_{k}\end{array}\right]^{2}$ |  |

基于CirNet的电路性能评估：通过两个层次化PINN（物理信息神经网络）——ModNet和CirNet的输出，可以评估功率转换器的性能。通过先前CirNet推理出的关键波形结果，可以测量多种电气性能指标，旨在为电力电子系统提供关键且全面的评估。如图[3](https://arxiv.org/html/2411.14214v1#S3.F3 "图3 ‣ 3.1\. 基于LLM的自主智能体 ‣ 3\. 相关工作 ‣ 基于物理信息的LLM智能体用于电力电子系统自动调制设计") Part (3)所示，$i_{L}(t)$的最大值和最小值的差值给出了峰峰电流应力。$[s_{1}(t)i_{L}(t)]^{2}$的数值积分提供了半导体开关的导通损失评估。软开关性能，尤其是在电动汽车充电等高频场景中，分析时需要在器件换流时对$i_{L}(t)$施加约束，在此过程中，需要同步$s_{pri}(t)$和$s_{sec}(t)$。其他电气性能可以通过电力电子领域的专业知识进行衡量。

如图[2](https://arxiv.org/html/2411.14214v1#S1.F2 "图2 ‣ 1\. 介绍 ‣ 基于物理信息的LLM智能体用于电力电子系统自动调制设计")所示，在核心引擎中，优化算法与ModNet和CirNet配合工作，寻找最优的调制参数。优化过程是一个迭代循环，其中优化算法将操作条件和调制参数传递给物理信息代理模型ModNet和CirNet，这些模型提供评估的性能指标，以指导优化过程。

通过将物理信息纳入神经网络，模型的可解释性得到了提高，同时也减少了所需的训练数据点数量。此外，整个设计自动化系统通过将额外功能的代理模型集成到核心引擎中，展示了无缝的可扩展性。重要的是，用户可以在不改变体验的情况下访问这些功能。

## 5\. 实验

![参考标题](img/a75c9fb47b5a1d83b4dba85e60d64d2f.png)

图4\. LP-COMDA与不同结构的CirNet。验证集上的最小MAE为0.235，当有2个隐藏层且每层有32个隐藏神经元时达到。

| 模型 | #参数 | 选择的超参数 |
| --- | --- | --- |
| LSTM | 21,671 | 3层，32隐藏单元 |
| GRU | 13,319 | 1层，64隐藏单元 |
| LN-GRU | 13,319 | 1层，64隐藏单元 |
| TCN | 88,199 |

&#124; 2层，7核大小，&#124;

&#124; 64隐藏单元&#124;

|

| GRU-VAE | 88,583 | 2层，64隐藏单元 |
| --- | --- | --- |
| TST | 6,913 |

&#124; 1层，2个注意力头，&#124;

&#124; 32隐藏单元&#124;

|

| TSiTPlus | 7,042 |
| --- | --- |

&#124; 1层，2个注意力头， &#124;

&#124; 32个隐藏单元 &#124;

|

| MiniRocket | - | 700个特征，6个膨胀大小 |
| --- | --- | --- |
| CirNet | 9,930 | 2层，32个隐藏单元 |
| LP-COMDA | 19,857 | 2层，32个隐藏单元 |

表 2\. LP-COMDA及基准方法的超参数选择。所选超参数在验证集上获得了最低的损失值。

### 5.1\. 实验设置

我们的主要目标是确定ModNet和CirNet代理模型的准确性，其中核心引擎中模型准确性的提高能够转化为更优的系统设计性能。因此，我们进行了多次实验，以验证LP-COMDA在低资源环境下的模型表现，即仅使用200个样本的小数据集。

数据准备：在本研究中，我们聚焦于时间序列预测建模任务，作为代表性示例，研究了DAB变换器在三相移相（TPS）调制下的电感电流 $i_{L}(t)$。通过硬件实验原型，我们获得了涵盖200个序列的波形数据。硬件原型的配置以及数据采集方法在参考文献（Lin et al., [2023b](https://arxiv.org/html/2411.14214v1#bib.bib27)）中进行了详细阐述。

PINN模型的一个优势是与其他机器学习模型相比，它需要的样本非常少。为了证明这一优势，我们用两种不同的数据划分比例进行实验。第一组数据将数据划分为5%的训练集（10个样本）、10%的验证集（20个样本）和85%的测试集（170个样本）。第二组数据则划分为50%、10%和40%作为训练集、验证集和测试集。

基准：为了评估LP-COMDA的设计结果，我们使用以下基准方法：（1）贝叶斯网络（BN）（Pearl, [1985](https://arxiv.org/html/2411.14214v1#bib.bib35)），（2）支持向量回归（SVR）（Smola 和 Schölkopf, [2004](https://arxiv.org/html/2411.14214v1#bib.bib41)），（3）XGBoost（Chen et al., [2015](https://arxiv.org/html/2411.14214v1#bib.bib7)），（4）随机森林（RF）（Breiman, [2001](https://arxiv.org/html/2411.14214v1#bib.bib5)），（5）长短期记忆网络（LSTM）（Hochreiter 和 Schmidhuber, [1997](https://arxiv.org/html/2411.14214v1#bib.bib11)），（6）GRU网络（Chung et al., [2014](https://arxiv.org/html/2411.14214v1#bib.bib8)），（7）LN-GRU网络（Ba et al., [2016](https://arxiv.org/html/2411.14214v1#bib.bib4)），（8）时序卷积网络（TCN）（Lea et al., [2017](https://arxiv.org/html/2411.14214v1#bib.bib20)），（9）基于GRU的变分自编码器（GRU-VAE）（An 和 Cho, [2015](https://arxiv.org/html/2411.14214v1#bib.bib2)），（10）时间序列Transformer（TST）（Zerveas et al., [2021](https://arxiv.org/html/2411.14214v1#bib.bib53)），（11）从视觉Transformer改编的时间序列网络（TSiT-Plus）（Oguiza, [2022](https://arxiv.org/html/2411.14214v1#bib.bib33)），以及（12）MiniRocket（Tan et al., [2022](https://arxiv.org/html/2411.14214v1#bib.bib42)）。

![参见说明](img/1d02f6359fedfd984fec5830571d610d.png)

图 5\. 前五个基准模型与所提 LP-COMDA 的建模性能。LP-COMDA 的建模结果最接近实际测量值，展示了其卓越的建模精度。

超参数搜索：为了公平比较，我们对 LP-COMDA 和所有基准算法在验证集上进行了超参数搜索。我们对每个模型的关键超参数进行了网格搜索。图 [4](https://arxiv.org/html/2411.14214v1#S5.F4 "Figure 4 ‣ 5\. Experiments ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems") 展示了 LP-COMDA 在不同超参数下的 MAE，揭示了所选最佳结构。最终选定的超参数集合总结在表 [2](https://arxiv.org/html/2411.14214v1#S5.T2 "Table 2 ‣ 5\. Experiments ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems") 中。

### 5.2\. 实验结果

| 模型 | 训练 | 验证 | 测试 | p值 |
| --- | --- | --- | --- | --- |
| BN | 2.152$\pm$0.0 | 2.436$\pm$0.0 | 2.515$\pm$0.0 | 6.81E-11* |
| SVR | 1.984$\pm$0.0 | 2.313$\pm$0.0 | 2.286$\pm$0.0 | 1.16E-10* |
| XGBoost | 2.299$\pm$0.0 | 2.108$\pm$0.0 | 2.792$\pm$0.0 | 3.83E-11* |
| RF | 4.983$\pm$0.0 | 4.181$\pm$0.0 | 4.434$\pm$0.0 | 3.18E-12* |
| LSTM | 1.164$\pm$0.151 | 1.380$\pm$0.042 | 1.560$\pm$0.095 | 7.23E-8* |
| GRU | 1.287$\pm$0.174 | 1.336$\pm$0.101 | 1.604$\pm$0.064 | 6.35E-10* |
| LN-GRU | 0.834$\pm$0.272 | 1.289$\pm$0.165 | 1.257$\pm$0.117 | 1.71E-6* |
| TCN | 1.571$\pm$0.210 | 1.996$\pm$0.257 | 1.538$\pm$0.501 | 1.436E-3* |
| TST | 0.668$\pm$0.062 | 0.750$\pm$0.028 | 0.666$\pm$0.063 | 1.47E-6* |
| TSiTPlus | 0.707$\pm$0.158 | 1.106$\pm$0.12 | 1.302$\pm$0.078 | 3.46E-8* |
| MiniRocket | 0.113$\pm$0.122 | 0.742$\pm$0.365 | 0.763$\pm$0.338 | 0.0131* |
| GRU-VAE | 1.928$\pm$0.201 | 2.190$\pm$0.180 | 2.219$\pm$0.172 | 6.47E-7* |
| CirNet Only | 0.121$\pm$0.014 | 0.282$\pm$0.014 | 0.310$\pm$0.013 | 1.442E-3* |
| LP-COMDA | 0.140$\pm$0.054 | 0.235$\pm$0.025 | 0.245$\pm$0.029 | 不适用 |

表 3\. 实验结果，训练/验证/测试集划分为 5%（10 个样本）、10%（20 个样本）和 85%（170 个样本）。LP-COMDA 在统计上优于基准模型（标有 *），与第二好的基准（TST）相比，误差降低了 63.2%，采用平均绝对误差（MAE）进行衡量。

| 模型 | 训练 | 验证 | 测试 | p值 |
| --- | --- | --- | --- | --- |
| BN | 2.455$\pm$0.0 | 2.206$\pm$0.0 | 2.348$\pm$0.0 | 5.03E-14* |
| SVR | 2.049$\pm$0.0 | 1.780$\pm$0.0 | 1.991$\pm$0.0 | 1.25E-13* |
| XGBoost | 1.321$\pm$0.0 | 2.194$\pm$0.0 | 1.986$\pm$0.0 | 1.27E-13* |
| RF | 3.167$\pm$0.0 | 3.937$\pm$0.0 | 3.622$\pm$0.0 | 4.89E-15* |
| LSTM | 1.246$\pm$0.165 | 1.129$\pm$0.078 | 1.193$\pm$0.098 | 1.89E-6* |
| GRU | 1.196$\pm$0.127 | 1.356$\pm$0.134 | 1.270$\pm$0.114 | 2.80E-6* |
| LN-GRU | 0.534$\pm$0.097 | 0.480$\pm$0.075 | 0.527$\pm$0.086 | 2.33E-4* |
| TCN | 0.685$\pm$0.018 | 0.744$\pm$0.062 | 0.726$\pm$0.061 | 3.79E-6* |
| TST | 0.264$\pm$0.033 | 0.240$\pm$0.033 | 0.264$\pm$0.031 | 3.93E-3* |
| TSiTPlus | 0.585$\pm$0.099 | 0.567$\pm$0.082 | 0.569$\pm$0.099 | 2.61E-4* |
| MiniRocket | 0.522$\pm$0.305 | 0.616$\pm$0.346 | 0.646$\pm$0.343 | 0.0246* |
| GRU-VAE | 0.686$\pm$0.074 | 0.663$\pm$0.036 | 0.721$\pm$0.056 | 2.42E-6* |
| CirNet Only | 0.162$\pm$0.032 | 0.239$\pm$0.014 | 0.234$\pm$0.007 | 3.98E-3* |
| LP-COMDA | 0.170$\pm$0.013 | 0.191$\pm$0.005 | 0.201$\pm$0.006 | N/A |

表4. 实验结果，训练/验证/测试集按50%（100个样本）、10%（20个样本）和40%（80个样本）进行分割。LP-COMDA在统计上优于基线（标记为*），与第二好的基准（TST）相比，其误差降低了23.7%，该误差是通过平均绝对误差（MAE）衡量的。

表[3](https://arxiv.org/html/2411.14214v1#S5.T3 "Table 3 ‣ 5.2\. Experimental Results ‣ 5\. Experiments ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")和[4](https://arxiv.org/html/2411.14214v1#S5.T4 "Table 4 ‣ 5.2\. Experimental Results ‣ 5\. Experiments ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")总结了LP-COMDA和基线模型在两种不同分割下的实验结果。每个算法都在相应的最优超参数下运行了十次。报告了训练集、验证集和测试集的平均MAE。报告的P值是0.02显著性水平下的T检验结果。

对于两种分割，实验结果表明，LP-COMDA优于传统的机器学习方法，如BN、SVR、XGBoost和RF，并且在深度学习预测模型如LSTM、GRU、LN-GRU和TCN以及预测模型如GRU-VAE中表现更为优越。与最先进的模型如TST、TSiTPlus和MiniRocket相比，LP-COMDA再次表现出显著提高的预测准确度。为了进一步验证分层PINNs结构，我们将CirNet与LP-COMDA进行了对比。显著地，LP-COMDA在只有10个训练样本的情况下超过了大多数基准（有100个训练样本），并且与CirNet-Only模型对齐。

为了可视化建模性能，图[5](https://arxiv.org/html/2411.14214v1#S5.F5 "Figure 5 ‣ 5.1\. Experiments Setup ‣ 5\. Experiments ‣ Physics-Informed LLM-Agent for Automated Modulation Design in Power Electronics Systems")展示了LP-COMDA和基准模型的$i_{L}$波形。值得注意的是，LP-COMDA的建模结果比其他任何模型与实际测量结果更为一致，显示了其卓越的建模精度。

总体而言，我们得出结论，所提出的LP-COMDA在统计上比其他基线模型在电力电子建模中取得了更好的建模结果，即使在极低数据场景下也是如此。

| ID | 目标 | 用户输入 | LM-COMDA输出 |
| --- | --- | --- | --- |
| 1 |

&#124; 定义调制 &#124;

&#124; 策略 $S$ 来设计 &#124;

|

&#124; 我需要你的帮助来设计我的 &#124;

&#124; 双向主动桥式转换器。 &#124;

|

&#124; 当然，我可以帮你，先从选择 &#124;

&#124; 调制策略。你想选择哪一种来 &#124;

&#124; 你的双向主动桥式转换器，是 TPS、DPS 还是混合型？ &#124;

|

| 2 |
| --- |

&#124; 定义优先考虑的 &#124;

&#124; 目标 $O$ &#124;

| TPS. |
| --- |

&#124; 太好了！现在让我们继续讨论优先级优化 &#124;

&#124; 目标。你希望选择以下哪一个目标 &#124;

&#124; 优先考虑：效率、软开关还是电流应力？ &#124;

|

| 3 |
| --- |

&#124; 定义操作 &#124;

&#124; 条件 &#124;

| 电流应力. |
| --- |

&#124; 明白了。让我们讨论一下你系统的操作条件 &#124;

&#124; 双向主动桥式转换器。你能提供以下信息吗 &#124;

&#124; 信息：额定功率、额定输入电压、额定输出 &#124;

&#124; 电压、实际操作功率、实际输出电压。 &#124;

|

| 4 |
| --- |

&#124; 选择优化 &#124;

&#124; 算法 &#124;

|

&#124; 额定功率：1000 W &#124;

&#124; 额定输入电压：200 V &#124;

&#124; 额定输出电压：200 V &#124;

&#124; 操作功率：600W &#124;

&#124; 实际输出电压：160V &#124;

|

&#124; 感谢你提供的操作条件。最后，我们 &#124;

&#124; 需要选择一个优化算法。哪个优化算法 &#124;

&#124; 你会选择哪种算法：PSO、DE 还是 GA？ &#124;

|

| 5 |
| --- |

&#124; 确认规格 &#124;

&#124; 已完全收集。 &#124;

| PSO. |
| --- |

&#124; 好的，我已经收集了所有参数，并即将 &#124;

&#124; 执行程序。 &#124;

|

| 设计结果与分析 |
| --- |

|

&#124; 图（1）展示了关于不同调制参数（内相移 $D_{l}$ 和 $D_{2}$）的电流应力建模结果。&#124;

&#124; 在输入电压为 200V、功率为 600W 时，最佳电流应力性能下优化的 $D_{l}$ 和 $D_{2}$ 以及输出电压 &#124;

&#124; 160 V 被突出显示为红点。图（2）表示最佳 TPS 的电流应力性能曲线以及对比的 &#124;

&#124; SPS 在功率范围 [100 W, 1000 W] 内的表现。图（3）是最佳调制参数选择的曲线，以达到最佳 &#124;

&#124; 电流应力在 [100 W, 1000 W] 范围内。你可以将这些值部署到你的控制器中。&#124;

|

| ![[未命名图片]](img/edb8371cdaa9296fc93501f7614385d8.png) | ![[未命名图片]](img/8c564d002b7334cb80429bcc49449a58.png) | ![[未命名图片]](img/d77f39e9923460ed9f128c6a0d95c5b8.png) |
| --- | --- | --- |
| 图（1） | 图（2） | 图（3） |

表5\. 一个使用案例示例：在5个请求系列下，LP-COMDA收集所有必要的设计规格，包括调制策略$S$、优先级目标$O$、操作条件以及选择的优化算法。在通过文本输入收集规格后，设计结果以文本和图形的形式呈现。

## 6\. 用户研究

为了评估LP-COMDA在提高工程效率方面的实际表现，我们设计了一个实际的使用案例，涉及DAB转换器的调制策略，并与20名工业从业人员进行了实证实验。

该设计过程的用户需求包括：

+   •

    定义设计规格的指导。

+   •

    针对定制场景的最优设计结果。

+   •

    对设计结果的分析。

根据使用案例，我们对20名工业从业人员进行了用户研究，这些人员通过研究实验室和合作的外部公司自愿招募。参与者包括10名经验不足三年的初级工程师和10名拥有五年以上相关经验的资深工程师。所有参与者声明自己在使用大型语言模型（LLM）辅助工程设计工作方面没有任何先前经验。

所有参与者都被给定了同一任务的两个独立变体，即设计用于作为DC-DC电网直流变换器的双有源桥（DAB）转换器的TPS调制策略，如图[1](https://arxiv.org/html/2411.14214v1#S1.F1 "图1 ‣ 1\. 引言 ‣ 基于物理信息的LLM代理在电力电子系统中自动调制设计")所示。我们要求参与者首先使用LP-COMDA的聊天界面处理任务变体一。完成后，他们再根据给定的设计要求和数据，使用标准方法（即分析和手动设计）处理任务变体二。为了评估有效性，任务处理中的有效工作时间以30分钟为一个时间块进行测量。

表[5](https://arxiv.org/html/2411.14214v1#S5.T5 "表5 ‣ 5.2\. 实验结果 ‣ 5\. 实验 ‣ 基于物理信息的LLM代理在电力电子系统中自动调制设计")显示了用户与LP-COMDA之间的多轮对话示例，以及输出组件。实证结果表明，用户研究的结果在功能性和效率方面都非常有前景。在功能性方面，所有20名参与者都能够成功完成分配的任务。在效率方面，10名初级工程师使用LP-COMDA和分析方法分别完成任务的平均时间为1.2个时间块和115.5个时间块；而10名资深工程师使用LP-COMDA和分析方法分别完成任务的平均时间为0.9个时间块和30.5个时间块。总体而言，我们观察到，初级工程师和资深工程师的效率分别提高了96.3倍和33.9倍。

## 7\. 结论

电力转换器的调制策略设计对可再生能源电力系统的优化运行至关重要。然而，当前许多以人工智能为核心的调制设计自动化系统面临着可解释性、可用性、可扩展性和数据密集性等挑战。在我们的研究中，我们引入了 LP-COMDA，一种基于大型语言模型的物理引导自主代理，用于有效的电力转换器调制设计自动化。实验表明，LP-COMDA 在数据受限的环境下，能够准确建模电力转换器。我们的用户研究表明，使用 LP-COMDA 可以帮助工程师在实践中提高超过 33 倍的效率。

## 参考文献

+   (1)

+   An 和 Cho (2015) Jinwon An 和 Sungzoon Cho. 2015. 基于变分自编码器的异常检测利用重构概率. *IE 专题讲座* 2, 1 (2015), 1–18.

+   Anil 等人 (2023) Rohan Anil, Andrew M Dai, Orhan Firat, Melvin Johnson, Dmitry Lepikhin, Alexandre Passos, Siamak Shakeri, Emanuel Taropa, Paige Bailey, Zhifeng Chen 等人. 2023. Palm 2 技术报告. *arXiv 预印本 arXiv:2305.10403* (2023).

+   Ba 等人 (2016) Jimmy Lei Ba, Jamie Ryan Kiros, 和 Geoffrey E Hinton. 2016. 层归一化. *arXiv 预印本 arXiv:1607.06450* (2016).

+   Breiman (2001) Leo Breiman. 2001. 随机森林. *机器学习* 45 (2001), 5–32.

+   Cao 和 Lee (2023) Yue Cao 和 CS Lee. 2023. 基于行为树的机器人任务生成与大型语言模型的结合.

+   Chen 等人 (2015) Tianqi Chen, Tong He, Michael Benesty, Vadim Khotilovich, Yuan Tang, Hyunsu Cho, Kailong Chen, Rory Mitchell, Ignacio Cano, Tianyi Zhou 等人. 2015. Xgboost: 极端梯度提升. *R 包版本 0.4-2* 1, 4 (2015), 1–4.

+   Chung 等人 (2014) Junyoung Chung, Caglar Gulcehre, KyungHyun Cho, 和 Yoshua Bengio. 2014. 基于门控递归神经网络的序列建模经验评估. *arXiv 预印本 arXiv:1412.3555* (2014).

+   Gao 等人 (2023) Siyu Gao, Yunbo Wang, 和 Xiaokang Yang. 2023. StockFormer: 利用预测编码学习混合交易机器. 见 *IJCAI*. 4766–4774.

+   Guo 等人 (2019) Chu Guo, Yong Liu, Min Xiong, Shichuan Xue, Xiang Fu, Anqi Huang, Xiaogang Qiang, Ping Xu, Junhua Liu, Shenggen Zheng 等人. 2019. 基于投影纠缠对态的通用量子电路模拟器与量子霸权前沿. *物理评论快报* 123, 19 (2019), 190501.

+   Hochreiter 和 Schmidhuber (1997) Sepp Hochreiter 和 Jürgen Schmidhuber. 1997. 长短期记忆. *神经计算* 9, 8 (1997), 1735–1780.

+   Hou 和 Li (2019) Nie Hou 和 Yun Wei Li. 2019. 非共振单相双有源桥直流–直流变换器调制与控制策略的概述与比较. *IEEE 电力电子学报* 35, 3 (2019), 3148–3172.

+   Hu 等人 (2020) Xinyue Hu, Haoji Hu, Saurabh Verma, 和 Zhi-Li Zhang. 2020. 基于物理引导的深度神经网络用于电力流分析. *IEEE 电力系统学报* 36, 3 (2020), 2082–2092.

+   黄和王（2022）黄斌，王建辉。2022年。物理信息神经网络在电力系统中的应用——综述。*IEEE电力系统汇刊* 38，1（2022），572–588。

+   吉等（2022）纪嘉豪，王靖远，姜哲，姜家伟，张虎。2022年。STDEN：面向交通流预测的物理引导神经网络。*AAAI人工智能会议论文集*，第36卷。4048–4056。

+   贾等（2021）贾晓伟，贾里德·威拉德，阿努吉·卡帕特，乔丹·S·里德，雅各布·A·兹沃特，迈克尔·斯坦巴赫，维平·库马尔。2021年。面向科学发现的物理引导机器学习：一个模拟湖泊温度剖面的应用。*ACM/IMS数据科学期刊* 2，3（2021年），1–26。

+   姜等（2023）姜南奎，李炳贤，洪永俊，尹锡培，朴恩炳。2023年。PIXEL：用于快速准确求解PDE的物理信息单元表示。*AAAI人工智能会议论文集*，第37卷。8186–8194。

+   高等（2024）凯尔文·JL·高，马云山，李奇·吴，蔡达生。2024年。利用自反大语言模型生成可解释的股票预测。*ACM 2024年网络会议论文集*。4304–4315。

+   莱等（2024）赖汉宇，刘晓，庞耀亮，姚顺天，陈宇轩，沈鹏波，余浩，张涵晨，张晓寒，董宇晓等。2024年。AutoWebGLM：基于大语言模型的Web导航代理的引导和强化。*arXiv预印本arXiv:2404.03648*（2024年）。

+   莉等（2017）科林·莉，迈克尔·D·弗林，雷内·维达尔，奥斯汀·赖特，格雷戈里·D·哈格尔。2017年。用于动作分割和检测的时间卷积网络。*IEEE计算机视觉与模式识别会议论文集*。156–165。

+   李等（2024b）李川豪，杨润涵，李天凯，米拉德·巴法拉萨特，库罗什·沙里菲，迪尔克·贝尔根曼，杨卓然。2024b年。STRIDE：一种工具辅助的大型语言模型代理框架，用于战略性和互动式决策制定。*arXiv预印本arXiv:2405.16376*（2024年）。

+   李等（2024a）李忠航，夏亮豪，唐家斌，徐勇，施磊，夏龙，大伟尹，黄超。2024a年。Urbangpt：时空大语言模型。*arXiv预印本arXiv:2403.00813*（2024年）。

+   林（2022）林梵凡。2022年。结合人工智能的双向主动桥转换器设计。（2022年）。

+   林等（2024）林梵凡，刘俊华，李鑫泽，赵帅，赵博辉，马浩，张鑫，廖新元。2024年。PE-GPT：一种物理引导的互动大语言模型，用于电力变换器调制设计。*IEEE能源转换大会与博览会（ECCE’24）论文集*。

+   Lin et al. (2021) Fanfan Lin, Xin Zhang, 和 Xinze Li. 2021. 考虑电压转换比、系统稳定性和效率的对称 CLLC 共振直流变换器设计方法。*IEEE 电力电子学报* 36, 9 (2021), 10157–10170. [https://doi.org/10.1109/TPEL.2021.3059852](https://doi.org/10.1109/TPEL.2021.3059852)

+   Lin et al. (2023a) Fanfan Lin, Xin Zhang, Xinze Li, Changjiang Sun, Gabriel Zsurzsan, Wenjian Cai, 和 Chang Wang. 2023a. 基于 AI 的设计与数据修剪用于最小电流应力双活桥变换器的混合相位移调制。*IEEE 新兴与选定电力电子学科期刊* (2023)。

+   Lin et al. (2023b) Fanfan Lin, Xin Zhang, Xinze Li, Changjiang Sun, Gabriel Zsurzsan, Wenjian Cai, 和 Chang Wang. 2023b. 基于 AI 的设计与数据修剪用于最小电流应力双活桥变换器的混合相位移调制。*IEEE 新兴与选定电力电子学科期刊* (2023), 1–1. [https://doi.org/10.1109/JESTPE.2022.3232534](https://doi.org/10.1109/JESTPE.2022.3232534)

+   Liu et al. (2024a) Junhua Liu, Justin Albrethsen, Lincoln Goh, David Yau, 和 Kwan Hui Lim. 2024a. 面向战术网络未来状态预测的时空图表示学习。在 *国际神经网络联合会议 (IJCNN'24) 论文集* 中。

+   Liu et al. (2024b) Junhua Liu, Tan Yong Keat, Bin Fu, 和 Kwan Hui Lim. 2024b. LARA: 语言自适应检索增强用于多轮意图分类。在 *2024 年自然语言处理经验方法会议: 工业分会* 论文集，Franck Dernoncourt, Daniel Preoţiuc-Pietro, 和 Anastasia Shimorina (编辑)。计算语言学协会，迈阿密，佛罗里达州，美国，1096–1106. [https://aclanthology.org/2024.emnlp-industry.82](https://aclanthology.org/2024.emnlp-industry.82)

+   Liu et al. (2021) Junhua Liu, Kwan Hui Lim, Kristin L Wood, Wei Huang, Chu Guo, 和 He-Liang Huang. 2021. 混合量子-经典卷积神经网络。*中国科学: 物理学、力学与天文学* 64, 9 (2021), 290311。

+   McNutt et al. (2023) Andrew M McNutt, Chenglong Wang, Robert A Deline, 和 Steven M Drucker. 2023. 关于为笔记本设计 AI 驱动的代码助手。在 *2023 年人机交互会议 (CHI) 论文集* 中，1–16。

+   Mudunuru et al. (2020) Maruti K Mudunuru, Daniel O'Malley, Shriram Srinivasan, Jeffrey D Hyman, Matthew R Sweeney, Luke Frash, Bill Carey, Michael R Gross, Nathan J Welch, Satish Karra, 等. 2020. 基于物理的机器学习用于实时非常规油藏管理。 (2020), 1–10。

+   Oguiza (2022) Ignacio Oguiza. 2022. tsai - 用于时间序列和顺序数据的最先进深度学习库。Github. [https://github.com/timeseriesAI/tsai](https://github.com/timeseriesAI/tsai)

+   OpenAI (2023) OpenAI. 2023. GPT-4 技术报告。arXiv:2303.08774 [cs.CL]

+   Pearl（1985）朱迪亚·珀尔。1985年。贝叶斯网络：一种用于证据推理的自激活记忆模型。在*第七届认知科学学会会议论文集，加利福尼亚大学，尔湾，美国*。15–17。

+   Raissi（2018）马齐亚尔·雷西。2018年。深度隐物理模型：非线性偏微分方程的深度学习。*机器学习研究杂志* 19, 1（2018），932–955。

+   Ross 等人（2023）史蒂文·I·罗斯、费尔南多·马丁内斯、斯蒂芬妮·霍德、迈克尔·穆勒和贾斯廷·D·韦斯。2023年。程序员助手：与大型语言模型进行软件开发对话交互。在*第28届国际智能用户界面会议论文集*。491–514。

+   Shakarian 等人（2023）保罗·沙卡里安、阿宾纳夫·科亚拉穆迪、诺埃尔·恩古和拉克什米维哈里·马里杜。2023年。对ChatGPT在数学文字问题（MWP）上的独立评估。

+   Shi 等人（2021）荣业·史、兆斌·莫和玄·狄。2021年。基于物理信息的深度学习用于交通状态估计：一种由二阶交通模型启发的混合范式。35, 1（2021），540–547。

+   Shi 等人（2024）文浩·史、志强·胡、怡彬·殷、俊华·刘、阳·杨、思炯·吴、立东·邴和瑞·卡伟·李。2024年。Math-LLaVA：为多模态大型语言模型引导数学推理。*arXiv预印本 arXiv:2406.17294*（2024）。

+   Smola 和 Schölkopf（2004）亚历克斯·J·斯莫拉和伯恩哈德·舍尔科普夫。2004年。支持向量回归教程。*统计与计算* 14（2004），199–222。

+   Tan 等人（2022）常伟·谭、安格斯·邓普斯特、克里斯托夫·伯格迈尔和杰弗瑞·I·韦布。2022年。MultiRocket：多种池化操作符和转换用于快速有效的时间序列分类。*数据挖掘与知识发现* 36, 5（2022），1623–1646。

+   Tang 等人（2020）袁洪·唐、韦豪·胡、建·肖、张勇·陈、齐·黄、哲·陈和弗雷德·布拉比杰。2020年。基于强化学习的DAB DC-DC转换器效率优化方案，采用三相移调制。*IEEE工业电子学报* 68, 8（2020），7350–7361。

+   Touvron 等人（2023）雨果·图弗龙、路易·马丁、凯文·斯通、彼得·阿尔伯特、阿姆贾德·阿尔迈哈里、雅思敏·巴巴伊、尼古拉·巴什里科夫、苏米娅·巴特拉、普拉杰瓦尔·巴尔加瓦、舒提·博萨尔等。2023年。Llama 2：开放基础和微调的聊天模型。*arXiv预印本 arXiv:2307.09288*（2023）。

+   Wandel 等人（2022）尼尔斯·万德尔、迈克尔·温曼、迈克尔·奈德林和雷因哈德·克莱因。2022年。Spline-pinn：通过快速、物理信息化的赫尔米特样条卷积神经网络（CNN）接近无数据的偏微分方程（PDE）。在*第36届人工智能学会（AAAI）会议论文集*。8529–8538。

+   Wang 等人（2024）雷·王、陈·马、雪阳·冯、泽宇·张、浩·杨、景森·张、志远·陈、家凯·唐、徐·陈、彦凯·林等。2024年。基于大型语言模型的自主代理调查。*计算机科学前沿* 18, 6（2024），186345。

+   Xiang 等人 (2024) Zhichen Xiang, Hongke Zhao, Chuang Zhao, Ming He, 和 Jianping Fan. 2024. 在推荐系统中，通过战略代理驱动的公平曝光优化进行表现性去偏。*arXiv 预印本 arXiv:2406.17475* (2024)。

+   Xiao 等人 (2019) Dunhui Xiao, CE Heaney, L Mottet, F Fang, W Lin, IM Navon, Y Guo, OK Matar, AG Robins, 和 CC Pain. 2019. 使用机器学习在城市环境中对湍流流动进行降阶建模。*建筑与环境* 148 (2019), 323–337。

+   Xing 等人 (2024) Mingzhe Xing, Rongkai Zhang, Hui Xue, Qi Chen, Fan Yang, 和 Zhen Xiao. 2024. 理解大型语言模型代理在复杂安卓环境中的弱点。*arXiv 预印本 arXiv:2402.06596* (2024)。

+   Xu 等人 (2024) Yunqi Xu, Tianchi Cai, Jiyan Jiang, 和 Xierui Song. 2024. Face4RAG：面向中文检索增强生成的事实一致性评估。*arXiv 预印本 arXiv:2407.01080* (2024)。

+   Yang 等人 (2023) Zijiang Yang, Zhongwei Qiu, 和 Dongmei Fu. 2023. Dmis：基于动态网格的重要性采样训练物理信息神经网络。发表于 *AAAI 人工智能会议论文集*，第37卷。5375–5383。

+   Yao 等人 (2024) Xu Yao, Austin Shinn, Kai Liu, 和 Li Xu. 2024. 基于大型语言模型的复杂任务解决中的人类-代理协作。*arXiv 预印本 arXiv:2402.12914* (2024)。 [https://ar5iv.org/abs/2402.12914](https://ar5iv.org/abs/2402.12914)

+   Zerveas 等人 (2021) George Zerveas, Srideepika Jayaraman, Dhaval Patel, Anuradha Bhamidipaty, 和 Carsten Eickhoff. 2021. 基于变压器的多变量时间序列表示学习框架。在 *第27届ACM SIGKDD知识发现与数据挖掘大会论文集* 中. 2114–2124。

+   Zhang 等人 (2024a) Chong Zhang, Xinyi Liu, Mingyu Jin, Zhongmou Zhang, Lingyao Li, Zhengting Wang, Wenyue Hua, Dong Shu, Suiyuan Zhu, Xiaobo Jin 等人. 2024a. 当人工智能遇上金融 (StockAgent)：基于大型语言模型的股票交易在模拟现实环境中的应用。*arXiv 预印本 arXiv:2407.18957* (2024)。

+   Zhang 等人 (2024b) Wentao Zhang, Lingxuan Zhao, Haochong Xia, Shuo Sun, Jiaze Sun, Molei Qin, Xinyi Li, Yuqing Zhao, Yilei Zhao, Xinyu Cai 等人. 2024b. *面向金融交易的多模态基础代理：工具增强、多样化和通用型*。技术报告。

+   Zhao 等人 (2022) Shuai Zhao, Yingzhou Peng, Yi Zhang, 和 Huai Wang. 2022. 使用物理信息驱动的机器学习对电力电子转换器进行参数估计。*IEEE 电力电子学报* 37, 10 (2022), 11567–11578。
