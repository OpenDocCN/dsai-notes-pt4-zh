<!--yml

分类：未分类

日期：2025-01-11 12:13:48

-->

# AmpAgent：基于LLM的多代理系统，用于从文献中进行多级放大器原理图设计及工艺与性能移植

> 来源：[https://arxiv.org/html/2409.14739/](https://arxiv.org/html/2409.14739/)

刘承杰^($1,3$)，陈伟宇^($1,3$)，彭安兰^($1,3$)，杜元^($1$)，杜力^($1$)，杨俊^($2,3$) ^($1$) 南京大学电子科学与工程学院，中国南京 ^($2$) 东南大学集成电路学院，中国南京 ^($3$) 南京技术创新中心EDA，南京，中国 cjliu_phd@smail.nju.edu.cn, wychen@smail.nju.edu.cn, alpeng@smail.nju.edu.cn yuandu@nju.edu.cn, ldu@nju.edu.cn, dragon@seu.edu.cn 通讯作者：杜力  邮箱：ldu@nju.edu.cn

###### 摘要

多级放大器广泛应用于模拟电路。然而，由于其组件数量庞大、传递函数复杂、极点零点分布复杂，需要大量人力进行推导和设备参数的选取，以确保其稳定性。为了实现高效的传递函数推导和设备参数选取，从而简化放大器设计的难度，我们提出了AmpAgent：一种基于大型语言模型（LLM）的多代理系统，用于高效地从文献中设计此类复杂的放大器，并进行工艺与性能移植。AmpAgent由三个代理组成：文献分析代理、数学推理代理和设备选型代理。它们分别负责从文献中检索关键信息（如公式和传递函数），通过推导关键公式分解整个电路设计问题，并对分解的问题进行迭代求解。

AmpAgent 被应用于七种不同补偿技术的多级放大器的原理图设计。在设计效率方面，与传统优化算法相比，AmpAgent 将迭代次数减少了 1.32$\sim$4${\times}$，执行时间减少了 1.19$\sim$2.99${\times}$，成功率提高了 1.03$\sim$6.79${\times}$。在电路性能方面，相较于原始文献，性能提高了 1.63$\sim$27.25${\times}$。研究结果表明，LLM（大型语言模型）在复杂模拟电路原理图设计、以及工艺和性能移植领域具有重要作用。

###### 关键词：

模拟电路，计算机辅助设计，电子设计自动化，大型语言模型，多级放大器，优化算法

## I 引言

多级放大器与单级放大器相比，能够提供高开环增益和强大的驱动能力，但所需的供电电压相对较低，这使其在放大微小信号、驱动LED和处理生物信号等方面具有广泛应用[[1](https://arxiv.org/html/2409.14739v1#bib.bib1), [2](https://arxiv.org/html/2409.14739v1#bib.bib2), [3](https://arxiv.org/html/2409.14739v1#bib.bib3), [4](https://arxiv.org/html/2409.14739v1#bib.bib4), [5](https://arxiv.org/html/2409.14739v1#bib.bib5), [6](https://arxiv.org/html/2409.14739v1#bib.bib6), [7](https://arxiv.org/html/2409.14739v1#bib.bib7), [8](https://arxiv.org/html/2409.14739v1#bib.bib8)]。尽管设计多级放大器遵循着明确的流程[[9](https://arxiv.org/html/2409.14739v1#bib.bib9)]，但由于设备组件数量庞大、传递函数复杂以及相对于单级放大器的极点-零点分布繁琐，这一过程依然充满挑战。这些因素不仅使得在不同工艺和性能指标下手动设计多级放大器变得困难，而且广阔的搜索空间也给优化算法的有效设计带来了重大挑战。

多级放大器自动化设计的低效之处在于无法将其设计问题分解为每个阶段的优化问题。近年来，涌现的大型语言模型（LLMs）已经被证明能够解决在其他领域遇到的类似推理问题[[10](https://arxiv.org/html/2409.14739v1#bib.bib10), [11](https://arxiv.org/html/2409.14739v1#bib.bib11)]，甚至进行了一系列诺贝尔奖获奖的化学实验[[12](https://arxiv.org/html/2409.14739v1#bib.bib12)]。这些表明，LLMs不仅具有为数字电路[[13](https://arxiv.org/html/2409.14739v1#bib.bib13), [14](https://arxiv.org/html/2409.14739v1#bib.bib14), [15](https://arxiv.org/html/2409.14739v1#bib.bib15)]提供自动设计解决方案的潜力，也能为类似多级放大器这样的模拟电路提供自动设计方案。

已有一些工作[[16](https://arxiv.org/html/2409.14739v1#bib.bib16), [17](https://arxiv.org/html/2409.14739v1#bib.bib17), [18](https://arxiv.org/html/2409.14739v1#bib.bib18), [19](https://arxiv.org/html/2409.14739v1#bib.bib19)]利用LLM实现模拟电路的自动化原理图设计。[[16](https://arxiv.org/html/2409.14739v1#bib.bib16)]的作者们开发了一种名为LLM Artisan的运算放大器领域LLM，用于自主生成定制的放大器原理图。[[17](https://arxiv.org/html/2409.14739v1#bib.bib17)]中的研究人员开发了一种基于LLM的代理LADAC，并成功地实现了三种类型模拟电路的设计自动化。另一个工作[[18](https://arxiv.org/html/2409.14739v1#bib.bib18)]将LLM代理与贝叶斯优化（BO）算法[[20](https://arxiv.org/html/2409.14739v1#bib.bib20)]结合，设计了一个2级放大器和一个滞后比较器，其效率比BO更高。AnalogCoder[[19](https://arxiv.org/html/2409.14739v1#bib.bib19)]通过Python代码生成设计模拟电路，具有比最先进的LLM更强的能力。

尽管上述工作展示了LLM在有效设计模拟电路方面的能力，但在设计复杂的多级放大器时仍然存在未解决的挑战。例如，一个由18个组件组成的嵌套Gm-C补偿放大器[[1](https://arxiv.org/html/2409.14739v1#bib.bib1)]，它有2个补偿电容和2个前馈阶段来稳定放大器，由于以下原因，目前的LLM很难设计这样的放大器：

+   •

    缺乏模拟电路数据导致即便是最先进的LLM也缺乏相关知识，无法为多级放大器提供有价值的设计建议；

+   •

    尽管提供了详细的放大器信息，但由于缺乏对设计过程的理解，LLM在利用这些信息进行放大器设计时受到了阻碍。

为了解决上述问题，我们开发了AmpAgent，一个基于LLM的多代理系统，用于从文献中进行多级放大器原理图设计，重点是过程和性能移植，而非仅仅复制[[21](https://arxiv.org/html/2409.14739v1#bib.bib21)]。AmpAgent包含以下三个管道代理，以解决上述两个问题：

+   •

    文献分析代理：用于从文献中提取多级放大器的关键信息，弥补当前LLM的知识空白，并确保下游应用的数据准确性。

+   •

    数学推理代理：负责推导与放大器稳定性等条件相关的重要公式。此代理将放大器的整体优化问题分解为多个子问题，从而利用上游结果；

+   •

    器件尺寸调整代理：集成了仿真接口和常规优化算法，以便根据前一个代理划分的各种子问题，高效地调整多级放大器的尺寸。

通过三个管道代理的协作，AmpAgent成功实现了从文献中快速设计多级放大器的原理图。本文的主要贡献如下：

+   •

    开发了一种新型的领域特定LLM多代理系统，用于根据文献实现多级放大器的原理图设计，同时结合了工艺和性能的移植。

+   •

    AmpAgent设计的多级放大器的性能相比文献中的提升了27.25$\times$；

+   •

    与常规优化算法相比，AmpAgent在算法迭代次数上减少了最多4${\times}$，执行时间减少了最多2.99${\times}$，成功率提高了最多6.79${\times}$。

总结而言，我们的工作展示了LLM在模拟电路原理图设计、工艺和性能移植方面的潜力。本文的结构如下：第二节详细讨论了LLM在模拟电路中的当前局限性；第三节概述了AmpAgent的整体工作流程；第四节展示了将AmpAgent应用于七种不同类型的多级放大器原理图设计的实验，并与常规优化算法相比展示了AmpAgent的设计效率；最后，第五节总结了我们的工作。

## II LLM在模拟电路中的应用

本节将通过展示当前SOTA大模型（包括GPT-3.5、GPT-4[[22](https://arxiv.org/html/2409.14739v1#bib.bib22)]和GPT-4o[[23](https://arxiv.org/html/2409.14739v1#bib.bib23)]）在处理模拟电路时的不足，来展示在AmpAgent内构建多代理系统的必要性。

[[17](https://arxiv.org/html/2409.14739v1#bib.bib17)]中的研究探讨了GPT-4在生成一个三阶低通RC滤波器和一个五晶体管放大器的SPICE网表方面的应用。结果表明，GPT-4在这一任务中未能成功。[[18](https://arxiv.org/html/2409.14739v1#bib.bib18)]的研究将GPT-3.5与贝叶斯优化（BO）结合，以提高模拟电路尺寸调整的效率。尽管如此，仅依赖GPT-3.5仍然导致在完成电路设计之前需要大量的迭代。

![参见说明](img/380611892ec4c6d9956bf256d40cd458.png)

图1：GPT-4o从电路图提取网表的实验

我们进一步评估了GPT-4o[[23](https://arxiv.org/html/2409.14739v1#bib.bib23)] 从电路图中提取网表的能力，如图[1](https://arxiv.org/html/2409.14739v1#S2.F1 "Figure 1 ‣ II LLM for analog circuits ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")所示。不幸的是，GPT-4o未能准确识别组件及其对应的连接。

之前研究的实验结果[[17](https://arxiv.org/html/2409.14739v1#bib.bib17)、[18](https://arxiv.org/html/2409.14739v1#bib.bib18)]以及 GPT-4o[[23](https://arxiv.org/html/2409.14739v1#bib.bib23)] 在识别模拟电路图方面的能力将被展示。这些实验如表[I](https://arxiv.org/html/2409.14739v1#S2.T1 "TABLE I ‣ II LLM for analog circuits ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")所示，结果表明即便是当前的SOTA（最先进技术）LLM也缺乏足够的模拟电路知识，因此也无法准确识别电路。因此，有必要利用多代理系统来实现模拟电路设计中的分工与协作，从而减少每个代理的能力要求，并提高LLM在电路设计中的表现。

表 I：GPT-4 和 GPT-4o 在模拟电路设计中的能力

| 模型名称 | 实验 | 结果 |
| --- | --- | --- |
| GPT-4[[22](https://arxiv.org/html/2409.14739v1#bib.bib22)] | 电路生成[[17](https://arxiv.org/html/2409.14739v1#bib.bib17)] | ✗ |
| GPT-3.5 | 尺寸建议（粗略）[[18](https://arxiv.org/html/2409.14739v1#bib.bib18)] | ✔ |
| 尺寸建议（精确）[[18](https://arxiv.org/html/2409.14739v1#bib.bib18)] | ✗ |
| GPT-4o[[23](https://arxiv.org/html/2409.14739v1#bib.bib23)] | 电路图识别 | ✗ |

![参见说明](img/eba8a29a8ec8fddc728764c12052d229.png)

图 2：AmpAgent概览

## III AmpAgent概述

AmpAgent是一个基于LLM的多代理系统，旨在从文献中设计多级放大器原理图。AmpAgent的工作流程如图[2](https://arxiv.org/html/2409.14739v1#S2.F2 "Figure 2 ‣ II LLM for analog circuits ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")所示。AmpAgent包括三个代理：（1）文献分析代理，（2）数学推理代理，和（3）设备尺寸建议代理。所有代理均基于ReAct [[24](https://arxiv.org/html/2409.14739v1#bib.bib24)] 技术开发。

文献分析代理能够检索并总结关键信息，如关键的计算公式和稳定性条件。这是通过将尚未设计的文献嵌入为文献分析代理RAG的向量数据库来实现的。

数学推理代理与用户输入的期望性能以及文献分析代理提取和总结的关键信息公式相结合，对多级放大器的关键参数（如每级的跨导（$g_{m}$））进行预测计算，从而将整体设计问题分解为多个子问题。

设备尺寸调整代理与放大器进行交互（例如调整器件尺寸和读取仿真结果），并结合常规优化算法，依次解决子问题。

除了上述三个代理外，在确认放大器功能正常后，中间的推理步骤将被保存。这样，当将来设计相同放大器时，就可以直接检索推理结果，从而减少 LLM 消耗的 token 数量，提高效率。

### III-A 文献分析代理

文献分析代理是一个基于 LLM 的代理，配备了一个检索增强生成（RAG）模块，并精心设计了提示语。这两个组件协同工作，确保能够从尚未完全设计的文献中准确提取出所有放大器设计所需的关键信息，使其可以直接供下游代理使用。

#### III-A1 检索增强生成

我们采用 RAG[[25](https://arxiv.org/html/2409.14739v1#bib.bib25)] 来弥补 LLM 在多级放大器方面的知识空白，并减轻幻觉[[26](https://arxiv.org/html/2409.14739v1#bib.bib26)] 的潜在影响。具体来说，我们首先使用 Mathpix[[27](https://arxiv.org/html/2409.14739v1#bib.bib27)] 将未完全设计的文献从 PDF 格式转换为 Markdown 格式，并准确提取出文献中的 latex 格式公式。接下来，包含 latex 格式公式的转换文档将作为 RAG 的向量数据库。这些数据构成了我们 RAG 模块的基础。

尽管直接应用 RAG 于电路设计仍然面临在抽象精确和准确的信息时的重大挑战，但关键的信息 LLM 在配备 RAG 后仍未能做出回应，主要包括两部分：

+   •

    电路原理图：文献中提供的原理图作为图像无法直接在 LLM 中使用进行电路设计，必须首先转换为网表格式，以便进一步进行参数调整。由于现有工具[[28](https://arxiv.org/html/2409.14739v1#bib.bib28), [29](https://arxiv.org/html/2409.14739v1#bib.bib29)] 在处理复杂的多级放大器电路时存在不准确和低效的问题，我们目前正在手动将原理图图像转换为网表文件。

+   •

    公式：放大器的公式（例如，传输函数和稳定性条件）通常分散在文献的不同部分，这可能导致在直接使用 RAG 时，关键信息丢失以及无关信息的冗余，如图[3](https://arxiv.org/html/2409.14739v1#S3.F3 "Figure 3 ‣ III-A1 Retrieval-Augmented Generation ‣ III-A Literature Analysis Agent ‣ III Overview of AmpAgent ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")中的黑框所示。

因此，仅仅依靠 RAG 来增强 LLM 从文献中提取多级放大器关键信息的能力仍然不足。为此，我们进一步定制了一套提示语，以改善 LLM 在相关信息检索方面的能力。

![参见说明文字](img/553f4ae5746223dd5ba9ab980f0a671a.png)

图 3：带提示与不带提示的 RAG 响应对比

#### III-A2 提示语

为了解决 RAG 模块在多级放大器设计中信息检索的不足和冗余问题，我们为 LLM 开发了特定的提示语。通过引导 LLM 定向获取知识，这些提示增强了 LLM 检索设计过程中所需精确信息的能力，避免了不必要的内容，使得响应简洁明了。

图[3](https://arxiv.org/html/2409.14739v1#S3.F3 "Figure 3 ‣ III-A1 Retrieval-Augmented Generation ‣ III-A Literature Analysis Agent ‣ III Overview of AmpAgent ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")展示了提供提示与不提供提示的效果对比，重点分析了使用带有零值电阻的内嵌米勒补偿（NMCNR）放大器作为实验对象的情况。图[3](https://arxiv.org/html/2409.14739v1#S3.F3 "Figure 3 ‣ III-A1 Retrieval-Augmented Generation ‣ III-A Literature Analysis Agent ‣ III Overview of AmpAgent ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")展示了利用提示语对检索过程的影响，突出显示了当 LLM 在特定提示的引导下，准确性和效率的提高。

总结来说，我们将 RAG 技术与 LLM Agent 相结合，以高效地检索多级放大器设计过程中所需的关键信息。通过简化检索并专注于关键数据，这一方法显著提升了信息检索系统的效率和准确性。然而，检索到的原始信息，如传输函数，仍然需要进一步处理，这就需要一个数学推理代理来解决。

![参见说明文字](img/35726e2d192a62e887f608b00d988ce9.png)

图4：数学推理代理的响应。

### III-B 数学推理代理

设计过程所需的许多信息必须从原始放大器的传递函数中推导出来，或通过极零分布方法计算得出。在这一部分，我们将展示专门用于获取基本设计信息的数学推理代理。

该模块需要按顺序执行两个任务：（1）基于传递函数计算极点和零点的定义，（2）计算子设计任务的目标，考虑期望的性能指标、推导出的极点和零点定义，基于复杂极点方法[[30](https://arxiv.org/html/2409.14739v1#bib.bib30)]。为了高效计算，模块专门设计用于生成Python代码，以表示需要求解的方程式，并执行必要的计算。

从数学推理的角度来看，对于计算极零分布等固定计算范式的任务，我们使用少量示例提示将复杂极点方法的规则[[30](https://arxiv.org/html/2409.14739v1#bib.bib30)]传达给数学推理代理，利用其推理能力来实现结果。对于更复杂的计算任务，如求解方程组，我们指示数学推理代理使用Python的SymPy库，以确保计算的准确性和高效性。在这里，我们给出一个设计推导的示例，展示了当期望的$GBW$为10MHz，$C_{L}$为10pF时的嵌套Gm-C补偿（NGCC）放大器[[1](https://arxiv.org/html/2409.14739v1#bib.bib1)]，数学推理代理的响应如图[4](https://arxiv.org/html/2409.14739v1#S3.F4 "Figure 4 ‣ III-A2 Prompts ‣ III-A Literature Analysis Agent ‣ III Overview of AmpAgent ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")所示。

如图[4](https://arxiv.org/html/2409.14739v1#S3.F4 "Figure 4 ‣ III-A2 Prompts ‣ III-A Literature Analysis Agent ‣ III Overview of AmpAgent ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")所示，代理已有效推导出放大器的极点和零点，并准确计算出每个阶段的最终$g_{m}$。这展示了代理的能力，它不仅能够分析多级放大器的设计，还能够在输入性能指标变化时推导出$g_{m}$和电容值。

### III-C 设备尺寸代理

在上述代理确定了子任务的理论设计目标之后，特别是在多级放大器的情况下确定每一级的$g_{m}$，设备尺寸代理将被使用。该模块负责对晶体管进行尺寸调整，并调整设备值以达到期望的子任务设计目标。设备尺寸代理的概述见图[5](https://arxiv.org/html/2409.14739v1#S3.F5 "Figure 5 ‣ III-C Device Sizing Agent ‣ III Overview of AmpAgent ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")。

![请参阅说明文字](img/9e0e1a33408fc24d97ca6fe4796ae949.png)

图5：设备尺寸代理概述

设备尺寸代理执行三个操作：读取仿真结果、访问计算器和调用常规优化算法。这些操作使设备尺寸代理能够与仿真器交互，获取当前设备的性能指标（如晶体管的$g_{m}$，放大器的增益带宽积（$GBW$）和相位裕度（$PM$）等），使用计算器估算期望的设备参数（如根据当前$g_{m}$和期望$g_{m}$估算晶体管的$W/L$），最终将这些参数作为初始值，使用常规算法如人工蜂群算法（ABC）[[31](https://arxiv.org/html/2409.14739v1#bib.bib31)]和信任区域贝叶斯优化（TuRBO）[[20](https://arxiv.org/html/2409.14739v1#bib.bib20)]算法进行优化。如果常规优化算法的结果未能达到预期指标，当前的设备参数和指标将被返回给设备尺寸代理，以便其进一步估算初始值，进行另一次优化迭代。

上述过程将反复进行，直到设备尺寸代理优化完所有子问题。在所有子问题优化完成后，当前的设备参数将作为初始值，用于通过常规优化算法进行整体优化。

通过将基于代理的电路尺寸设计与常规优化算法相结合，我们可以减轻大语言模型（LLM）的迭代速度慢的问题，并减少优化算法在电路参数上的广泛搜索空间。这种混合方法充分发挥了两者的优势，结果使电路设计更高效且具成本效益。关于是否在子问题中使用优化算法以及在最终整体过程中使用优化算法的比较实验，将在第四节中展示。

## IV 实验

在本节中，我们展示了七种不同类型的AmpAgent设计的多级放大器的实验结果。我们还比较了使用和不使用传统优化算法时AmpAgent的性能。此外，我们还评估了设备尺寸代理（Device Sizing Agent）与传统优化算法的表现，重点关注算法迭代次数、时间消耗和参数尺寸调整过程中的成功率。作为例子，我们将详细说明三阶段带有零化电阻的嵌套米勒补偿（NMCNR）放大器的设计过程[[2](https://arxiv.org/html/2409.14739v1#bib.bib2)]。

所有实验均在一台配备12代Intel^® Core${}^{\text{TM}}$ i7-12700 CPU和64GB内存的Linux工作站上进行。放大器的仿真使用了Cadence Spectre^® 仿真器。整个AmpAgent框架是基于LangChain[[32](https://arxiv.org/html/2409.14739v1#bib.bib32)]开发的。我们选择的LLM是GPT-4-1106-preview，并且其温度设定为0。所有由AmpAgent设计的放大器已通过仿真验证，确保没有右半平面极点，并且不存在稳定性问题。

### IV-A NMCNR放大器

在这一部分，我们使用AmpAgent在0.18$\mu m$工艺上进行了NMCNR放大器的原理图设计。NMCNR放大器[[2](https://arxiv.org/html/2409.14739v1#bib.bib2)]是一种具有高开环增益并且使用零化电阻补偿右半平面零点的三阶段放大器，具有更稳定的结构[[5](https://arxiv.org/html/2409.14739v1#bib.bib5)]。

我们为AmpAgent设计NMCMR放大器所选择的文献是[[2](https://arxiv.org/html/2409.14739v1#bib.bib2)]。我们希望AmpAgent实现的性能指标包括：$C_{L}=50pF$，$GBW\geq{5MHz}$，$PM\geq{60^{\circ}}$，$Gain_{DC}\geq{120dB}$，并且功耗应尽可能低。文献分析代理（Literature Analysis Agent）、数学推理代理（Mathematics Reasoning Agent）的输出如图[6](https://arxiv.org/html/2409.14739v1#S4.F6 "Figure 6 ‣ IV-A NMCNR amplifier ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")所示。结果显示在表[II](https://arxiv.org/html/2409.14739v1#S4.T2 "TABLE II ‣ IV-A NMCNR amplifier ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")中。

表II：AmpAgent设计的NMCNR、SMC、NGCC、DFCFC、TCFC、IAC和AZC放大器性能

| 放大器 | 预期值 | 无进一步优化 | 经过进一步优化 |
| --- | --- | --- | --- |
| $GBW$ $MHz$ | $PM$ ^∘ | $Gain_{DC}$ ${dB}$ | $GBW$ $MHz$ | $PM$ ^∘ | $Gain_{DC}$ ${dB}$ | $I_{dd}$ ${mA}$ | $IFOM_{S}$ $\frac{MHz*pF}{mA}$ | $GBW$ $MHz$ | $PM$ ^∘ | $Gain_{DC}$ ${dB}$ | $I_{dd}$ ${mA}$ | $IFOM_{S}$ $\frac{MHz*pF}{mA}$ |
| NMCNR [[2](https://arxiv.org/html/2409.14739v1#bib.bib2)] @$C_{L}=50pF$ | $\geq 5.00$ | $\geq 60$ | $\geq 120$ | $5.15$ | $32.5$ | $135$ | $0.38$ | 675 | $5.01$ | $61.5$ | $131$ | $0.322$ | 778 |
| SMC [[3](https://arxiv.org/html/2409.14739v1#bib.bib3)] @$C_{L}=10pF$ | $\geq 10.00$ | $\geq 60$ | $\geq 70$ | $9.47$ | $60.05$ | $73.53$ | $0.087$ | 1092 | $10.0$ | $60.7$ | $70.06$ | $0.080$ | 1243 |
| NGCC [[1](https://arxiv.org/html/2409.14739v1#bib.bib1)] @$C_{L}=50pF$ | $\geq 1.00$ | $\geq 60$ | $\geq 100$ | $1.24$ | $52$ | $137$ | $0.078$ | 795 | $1.06$ | $63$ | $136$ | $0.054$ | 981 |
| DFCFC [[4](https://arxiv.org/html/2409.14739v1#bib.bib4)] @$C_{L}=150pF$ | $\geq 2.00$ | $\geq 60$ | $\geq 100$ | $2.00$ | $1.2$ | $119$ | $0.035$ | 8645 | $2.12$ | $60$ | $100$ | $0.052$ | 6103 |
| TCFC [[6](https://arxiv.org/html/2409.14739v1#bib.bib6)] @$C_{L}=300pF$ | $\geq 2.00$ | $\geq 60$ | $\geq 100$ | $2.05$ | $77$ | $127$ | $0.053$ | 11647 | $2.28$ | $71$ | $130$ | $0.017$ | 41204 |
| IAC [[7](https://arxiv.org/html/2409.14739v1#bib.bib7)] @$C_{L}=500pF$ | $\geq 4.00$ | $\geq 60$ | $\geq 100$ | $3.55$ | $57$ | $153$ | $0.053$ | 33490 | $4.87$ | $76$ | $150$ | $0.039$ | 62435 |
| AZC [[8](https://arxiv.org/html/2409.14739v1#bib.bib8)] @$C_{L}=15nF$ | $\geq 1.00$ | $\geq 60$ | $\geq 100$ | $0.96$ | $58$ | $135$ | $0.049$ | 195918 | $1.01$ | $61.0$ | $133$ | $0.047$ | 322340 |

表 III：AmpAgent 设计的放大器与原始文献的 $IFOM_{S}$ 对比

| 放大器名称 | $IFOM_{S}$ $\frac{MHz*pF}{mA}$ 来源文献 | $IFOM_{S}$ $\frac{MHz*pF}{mA}$ 由 AmpAgent 提供 | 改进 |
| --- | --- | --- | --- |
| SMC[[3](https://arxiv.org/html/2409.14739v1#bib.bib3)] | 400 | 1243 | 3.11$\times$ |
| NMCNR[[2](https://arxiv.org/html/2409.14739v1#bib.bib2)] | 410 | 778 | 1.90$\times$ |
| NGCC[[1](https://arxiv.org/html/2409.14739v1#bib.bib1)] | 36 | 981 | $27.25\times$ |
| DFCFC[[4](https://arxiv.org/html/2409.14739v1#bib.bib4)] | 1238 | 6103 | 4.93$\times$ |
| TCFC[[6](https://arxiv.org/html/2409.14739v1#bib.bib6)] | 14250 | 41204 | 2.89$\times$ |
| IAC[[7](https://arxiv.org/html/2409.14739v1#bib.bib7)] | 33000 | 62435 | 1.89$\times$ |
| AZC[[8](https://arxiv.org/html/2409.14739v1#bib.bib8)] | 197916 | 322340 | $1.63\times$ |

![参考图例](img/1a46daad3678b8e533cdddc915b5ce36.png)

图 6：AmpAgent 设计了一个三阶段 NMCNR 放大器，$C_{L}=50pF$，$GBW=5MHz$

### IV-B 其他多级放大器

除了上述提到的放大器外，我们还测试了AmpAgent在设计更多类型的多级放大器方面的能力。在这一部分，我们将展示AmpAgent在多个具有典型补偿结构的放大器设计结果，包括单米勒补偿（SMC）[[3](https://arxiv.org/html/2409.14739v1#bib.bib3)]、NGCC [[1](https://arxiv.org/html/2409.14739v1#bib.bib1)]、阻尼因子控制频率补偿（DFCFC）[[4](https://arxiv.org/html/2409.14739v1#bib.bib4)]、带电容反馈补偿的跨导（TCFC）[[6](https://arxiv.org/html/2409.14739v1#bib.bib6)]、阻抗自适应补偿（IAC）[[7](https://arxiv.org/html/2409.14739v1#bib.bib7)]、以及主动零补偿（AZC）放大器[[8](https://arxiv.org/html/2409.14739v1#bib.bib8)]，这些放大器满足不同的$GBW$和$C_{L}$要求。

此外，我们还比较了放大器在实施设备尺寸优化代理后与未实施整体优化时的性能。实验结果如表[II](https://arxiv.org/html/2409.14739v1#S4.T2 "TABLE II ‣ IV-A NMCNR amplifier ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")所示。从表[II](https://arxiv.org/html/2409.14739v1#S4.T2 "TABLE II ‣ IV-A NMCNR amplifier ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")中可以看出，加入整体优化后，放大器能够满足约束条件，并进一步提高了其性能。我们使用$IFOM_{S}$来评估放大器的性能，$IFOM_{S}$定义见于([1](https://arxiv.org/html/2409.14739v1#S4.E1 "In IV-B Other multi-stage amplifiers ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting"))。$IFOM_{S}$是文献中常用的一个指标[[7](https://arxiv.org/html/2409.14739v1#bib.bib7), [6](https://arxiv.org/html/2409.14739v1#bib.bib6), [8](https://arxiv.org/html/2409.14739v1#bib.bib8)]。

|  | $IFOM_{S}=GBW*C_{L}/I_{dd}$ |  | (1) |
| --- | --- | --- | --- |

与原文献中的$IFOM_{S}$相比，不同放大器的改进也列在表[III](https://arxiv.org/html/2409.14739v1#S4.T3 "TABLE III ‣ IV-A NMCNR amplifier ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")中。这些放大器的性能提升范围从1.63${\times}$到27.25${\times}$，相比原文献有所增强。

表IV：本研究、ABC和TuRBO在迭代次数、时间消耗和成功率方面的比较

| 放大器 | 本研究 | ABC [[31](https://arxiv.org/html/2409.14739v1#bib.bib31)] | TuRBO-5 [[20](https://arxiv.org/html/2409.14739v1#bib.bib20)] | TuRBO-1 [[20](https://arxiv.org/html/2409.14739v1#bib.bib20)] |
| --- | --- | --- | --- | --- |
| 平均迭代次数 | 平均时间（秒） | 成功率 | 平均迭代次数 | 平均时间（秒） | 成功率 | 平均迭代次数 | 平均时间（秒） | 成功率 | 平均迭代次数 | 平均时间（秒） | 成功率 |
| SMC [[3](https://arxiv.org/html/2409.14739v1#bib.bib3)] | 16 | 245 | 100% | 35 | 456 | 68% | 38 | 460 | 95% | 56 | 540 | 76% |
| NMCNR [[5](https://arxiv.org/html/2409.14739v1#bib.bib5)] | 19 | 444 | 100% | 75 | 600 | 46% | 53 | 842 | 84% | 52 | 597 | 82% |
| NGCC [[1](https://arxiv.org/html/2409.14739v1#bib.bib1)] | 22 | 458 | 98% | 75 | 543 | 46% | 29 | 1200 | 95% | 35 | 843 | 94% |
| DFCFC [[4](https://arxiv.org/html/2409.14739v1#bib.bib4)] | 33 | 603 | 95% | 93 | 783 | 14% | 53 | 1321 | 83% | 46 | 836 | 88% |
| TCFC [[6](https://arxiv.org/html/2409.14739v1#bib.bib6)] | 26 | 471 | 96% | 75 | 601 | 54% | 36 | 894 | 89% | 48 | 768 | 85% |
| IAC [[7](https://arxiv.org/html/2409.14739v1#bib.bib7)] | 25 | 481 | 97% | 100 | 1352 | (失败) | 61 | 976 | 73% | 64 | 742 | 82% |
| AZC [[8](https://arxiv.org/html/2409.14739v1#bib.bib8)] | 38 | 696 | 96% | 100 | 1465 | (失败) | 100 | 2083 | (失败) | 100 | 1610 | (失败) |

### IV-C 与传统优化算法的集成

除了使用优化算法进行放大器的整体优化外，AmpAgent 还可以在解决子问题的过程中利用该算法辅助调整晶体管的尺寸，如在第三节 C 小节中所讨论的。

在这一部分，我们将比较 AmpAgent 是否在优化过程中使用优化算法，特别是 ABC，并观察其在 LLM token 消耗和设计时间方面的差异。

![请参见说明](img/c6d5e842d78a42c5a8ad72e117a1a611.png)

图 7：设计时间和 token 消耗对比

以AmpAgent自动设计NGCC放大器[[1](https://arxiv.org/html/2409.14739v1#bib.bib1)]为例，我们随机采样了$GBW$和$C_{L}$的期望值，范围分别为$1MHz$到$10MHz$和$10pF$到$100pF$。LLM（此处为GPT4）的时间消耗和代币消耗对比如图[7](https://arxiv.org/html/2409.14739v1#S4.F7 "Figure 7 ‣ IV-C Integration with conventional optimization algorithms ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")所示。图[7](https://arxiv.org/html/2409.14739v1#S4.F7 "Figure 7 ‣ IV-C Integration with conventional optimization algorithms ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")显示，AmpAgent通过采用优化算法解决子优化问题，将所需时间减少了1.56倍，并且提高了代币消耗效率1.52倍。通过这个比较，可以看出，AmpAgent更适合分解子优化问题，并为这些子问题提供初步解决方案，而不是直接对参数进行迭代优化。

### IV-D 优化效率

此外，我们对设备尺寸代理与传统优化算法在参数尺寸优化效率方面进行了比较分析。该比较如表[IV](https://arxiv.org/html/2409.14739v1#S4.T4 "TABLE IV ‣ IV-B Other multi-stage amplifiers ‣ IV Experiment ‣ AmpAgent: An LLM-based Multi-Agent System for Multi-stage Amplifier Schematic Design from Literature for Process and Performance Porting")所示，重点关注了迭代次数、设计时间和成功率。我们将最大迭代次数设置为100。结果有力地证明，AmpAgent能够显著提高自动化设计过程的准确性，同时有效减少迭代次数及相关设计时间。值得注意的是，与AmpAgent相关的设计失败实例可以归因于GPT-4偶尔出现的幻觉效应，导致其无法正确执行某些操作。

## V 结论

在本研究中，我们提出了AmpAgent，这是一种利用基于LLM的多代理方法，专注于从文献中进行多级电路图设计，并进行工艺和性能的移植。AmpAgent由三个代理组成，通过详细的设计问题划分，它缓解了LLM缺乏模拟电路的专业知识和设计过程理解的当前问题。

我们特别展示了AmpAgent在自动设计七种不同类型的多级放大器方面的有效性，并进行了工艺和性能的移植。这些放大器的性能相比原文文献有了1.63${\times}$到27.25${\times}$的提升，验证了AmpAgent在复杂模拟电路中的能力。AmpAgent的设计效率超越了传统优化算法，迭代次数减少了1.32$\sim$4${\times}$，执行时间减少了1.19$\sim$2.99${\times}$，成功率提高了1.03$\sim$6.79${\times}$，甚至能够成功设计传统优化算法无法完成的放大器。

为了进一步增强AmpAgent，未来的工作可能涉及训练一个模拟电路领域的大型语言模型（LLM）、自动化从电路图提取网表，并集成诸如$g_{m}/I_{d}$[[33](https://arxiv.org/html/2409.14739v1#bib.bib33)]等先进的电路设计方法，以提高效率。

## 参考文献

+   [1] F. You, S. H. Embabi 和 E. Sanchez-Sinencio，"带有嵌套$g_{m}$-c补偿的多级放大器拓扑结构"，*IEEE固态电路学报*，第32卷，第12期，2000–2011页，1997年。

+   [2] K. N. Leung 和 P. Mok，"多级放大器频率补偿分析"，*IEEE电路与系统I：基础理论与应用学报*，第48卷，第9期，1041–1056页，2001年。

+   [3] P. E. Allen 和 D. R. Holberg，*CMOS模拟电路设计*，Elsevier，2011年。

+   [4] A. K. N. Leung, P. K. Mok, W. H. Ki 和 J. K. Sin，"低压低功耗大电容负载应用的阻尼因子控制频率补偿技术"，发表于*1999 IEEE国际固态电路会议，技术论文摘要，ISSCC，第一版（Cat. No. 99CH36278）*，IEEE，1999年，158–159页。

+   [5] K. N. Leung, P. K. Mok 和 W.-H. Ki，"低压低功耗嵌套米勒补偿CMOS放大器的右半平面零点去除技术"，发表于*ICECS'99，ICECS'99会议论文集，第6届IEEE国际电子电路与系统会议（Cat. No. 99EX357）*，第2卷，IEEE，1999年，599–602页。

+   [6] X. Peng 和 W. Sansen，"多级放大器的跨导与电容反馈补偿"，*IEEE固态电路学报*，第40卷，第7期，1514–1520页，2005年。

+   [7] X. Peng, W. Sansen, L. Hou, J. Wang 和 W. Wu，"低功耗多级放大器的阻抗适配补偿"，*IEEE固态电路学报*，第46卷，第2期，445–451页，2010年。

+   [8] Z. Yan, P.-I. Mak, M.-K. Law 和 R. P. Martins，"一款0.016毫米² 144-$\mu$w 三级放大器，能够驱动1到15纳法拉电容负载，具有$>$0.95-MHz增益带宽积"，*IEEE固态电路学报*，第48卷，第2期，527–540页，2013年。

+   [9] S. O. Cannizzaro, A. D. Grasso, R. Mita, G. Palumbo, 和 S. Pennisi, “带有嵌套米勒补偿的三阶段CMOS OTA设计程序，”*IEEE Circuits and Systems I: Regular Papers期刊*, 第54卷，第5期，页码933–940，2007年。

+   [10] L. Wen, D. Fu, X. Li, X. Cai, T. Ma, P. Cai, M. Dou, B. Shi, L. He, 和 Y. Qiao, “Dilu: 一种基于大语言模型的自主驾驶知识驱动方法，”*arXiv 预印本 arXiv:2309.16292*, 2023。

+   [11] G. Wang, Y. Xie, Y. Jiang, A. Mandlekar, C. Xiao, Y. Zhu, L. Fan, 和 A. Anandkumar, “Voyager: 基于大语言模型的开放式具身智能体，”*arXiv 预印本 arXiv:2305.16291*, 2023。

+   [12] D. A. Boiko, R. MacKnight, B. Kline, 和 G. Gomes, “利用大语言模型进行自主化学研究，”*Nature期刊*, 第624卷，第7992期，页码570–578，2023年。

+   [13] J. Blocklove, S. Garg, R. Karri, 和 H. Pearce, “Chip-chat: 对话式硬件设计中的挑战与机遇，”*arXiv 预印本 arXiv:2305.13243*, 2023。

+   [14] S. Thakur, B. Ahmad, Z. Fan, H. Pearce, B. Tan, R. Karri, B. Dolan-Gavitt, 和 S. Garg, “大语言模型自动化Verilog RTL代码生成基准测试，”*2023年欧洲设计、自动化与测试会议与展览（DATE）*，IEEE，2023年，第1–6页。

+   [15] M. Liu, T. Ene, R. Kirby, C. Cheng, N. Pinckney, R. Liang, J. Alben, H. Anand, S. Banerjee, I. Bayraktaroglu, B. Bhaskaran, B. Catanzaro, A. Chaudhuri, S. Clay, B. Dally, L. Dang, P. Deshpande, S. Dhodhi, S. Halepete, E. Hill, J. Hu, S. Jain, B. Khailany, K. Kunal, X. Li, H. Liu, S. Oberman, S. Omar, S. Pratty, A. Sarkar, Z. Shao, H. Sun, P. P. Suthar, V. Tej, K. Xu, 和 H. Ren, “Chipnemo: 针对芯片设计的领域适应大语言模型，”2023年。

+   [16] Y. L. F. Y. L. S. D. Z. X. Z. Zihao Chen, Jiangli Huang, “Artisan: 通过领域特定的大语言模型实现自动化运算放大器设计，”*第61届设计自动化会议（DAC）*，2024年。

+   [17] C. Liu, Y. Liu, Y. Du, 和 L. Du, “Ladac: 基于大语言模型的模拟电路自动设计器，”*Authorea Preprints*, 2024。

+   [18] Y. Yin, Y. Wang, B. Xu, 和 P. Li, “Ado-llm: 基于大语言模型的模拟设计贝叶斯优化与上下文学习，”*arXiv 预印本 arXiv:2406.18770*, 2024。

+   [19] Y. Lai, S. Lee, G. Chen, S. Poddar, M. Hu, D. Z. Pan, 和 P. Luo, “Analogcoder: 通过无训练的代码生成进行模拟电路设计，”*arXiv 预印本 arXiv:2405.14918*, 2024。

+   [20] D. Eriksson, M. Pearce, J. Gardner, R. D. Turner, 和 M. Poloczek, “通过局部贝叶斯优化进行可扩展的全局优化，”在*神经信息处理系统进展*中，H. Wallach, H. Larochelle, A. Beygelzimer, F. d'Alché-Buc, E. Fox, 和 R. Garnett 主编，第32卷。Curran Associates, Inc.，2019年。

+   [21] W. Xiong, X. Meng, Y. Tao, 和 P. Ling, “基于人工智能的学习与学术论文中的模拟电路设计再生，”*Authorea Preprints*, 2024。

+   [22] OpenAI, “GPT-4技术报告，” *ArXiv*，第abs/2303.08774号，2023年。[在线]。可用链接: https://api.semanticscholar.org/CorpusID:257532815

+   [23] OpenAI, “GPT-4O 系统卡片，” https://openai.com/index/gpt-4o-system-card/，2024年。

+   [24] S. Yao, J. Zhao, D. Yu, N. Du, I. Shafran, K. Narasimhan, 和 Y. Cao, “React：在语言模型中协同推理与行动，” *arXiv预印本arXiv:2210.03629*，2022年。

+   [25] P. Lewis, E. Perez, A. Piktus, F. Petroni, V. Karpukhin, N. Goyal, H. Küttler, M. Lewis, W.-t. Yih, T. Rocktäschel *等*，“用于知识密集型NLP任务的检索增强生成，” *神经信息处理系统进展*，第33卷，第9459-9474页，2020年。

+   [26] Z. Ji, N. Lee, R. Frieske, T. Yu, D. Su, Y. Xu, E. Ishii, Y. J. Bang, A. Madotto, 和 P. Fung, “自然语言生成中的幻觉调查，” *ACM计算机评论*，第55卷，第12期，2023年3月。[在线]。可用链接: https://doi.org/10.1145/3571730

+   [27] Mathpix, “Mathpix OCR API 用于STEM领域，” https://mathpix.com/ocr。

+   [28] H. B. Gurbuz, A. Balta, T. Dalyan, Y. D. Gokdel, 和 E. Afacan, “Img2sim-v2：一个用于无用户依赖的图像格式电路仿真工具，” 载于 *2023年第19届国际电路设计综合、建模、分析与仿真方法及应用会议(SMACD)*，IEEE，2023年，第1-4页。

+   [29] Z. Tao, Y. Shi, Y. Huo, R. Ye, Z. Li, L. Huang, C. Wu, N. Bai, Z. Yu, T.-J. Lin *等*，“Amsnet：AMS电路的网表数据集，” *arXiv预印本arXiv:2405.09045*，2024年。

+   [30] R. G. Eschauzier 和 J. Huijsing, *《低功耗运算放大器的频率补偿技术》*，Springer科学与商业媒体，1995年，第313卷。

+   [31] D. Karaboga, B. Gorkemli, C. Ozturk, 和 N. Karaboga, “全面调查：人工蜂群（ABC）算法及其应用，” *人工智能评论*，第42卷，第21-57页，2014年。

+   [32] Langchain AI, “Langchain，” https://github.com/langchain-ai/langchain，2024年。

+   [33] P. Jespers, *《gm/ID方法论：低电压模拟CMOS电路的尺寸工具：半经验和紧凑型模型方法》*，Springer科学与商业媒体，2009年。
