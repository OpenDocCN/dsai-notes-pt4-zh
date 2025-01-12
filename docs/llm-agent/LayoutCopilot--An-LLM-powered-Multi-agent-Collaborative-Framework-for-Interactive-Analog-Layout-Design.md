<!--yml

类别：未分类

日期：2025-01-11 12:29:20

-->

# LayoutCopilot：一个由LLM驱动的多智能体协作框架，用于交互式模拟布局设计

> 来源：[https://arxiv.org/html/2406.18873/](https://arxiv.org/html/2406.18873/)

Bingyang Liu^†, Haoyi Zhang^†, Xiaohan Gao, Zichen Kong，

Xiyuan Tang，Yibo Lin，Runsheng Wang，

Ru Huang 本研究部分得到了中国国家自然科学基金（资助编号：62141404、62125401、62034007）、北京市自然科学基金（资助编号：Z230002）和“111计划”（B18001）的支持。^† 平等贡献。B. Liu 现为北京大学电子工程与计算机科学学院的成员，位于北京，中国。H. Zhang 和 Z. Kong 现为北京大学集成电路学院的成员，位于北京，中国。X. Gao 现为北京大学计算机科学学院与集成电路学院的成员，位于北京，中国。X. Tang 现为北京大学人工智能研究所与集成电路学院的成员，同时也在北京集成电路先进创新中心工作，位于北京，中国。Y. Lin、R. Wang 和 R. Huang 现为北京大学集成电路学院成员，并在北京集成电路先进创新中心和北京大学电子设计自动化研究所工作，位于北京、中国无锡。通讯作者：Yibo Lin（[yibolin@pku.edu.cn](yibolin@pku.edu.cn)）

###### 摘要

模拟布局设计高度依赖于人类与设计工具之间的交互过程。针对这一任务的电子设计自动化（EDA）工具通常采用脚本命令或可视化按钮进行操作，尤其是在处理交互式自动化功能时，这些工具通常具有陡峭的学习曲线和繁琐的用户体验，这成为设计师采用这些工具的显著障碍。为了解决这一可用性问题，本文介绍了LayoutCopilot，这是一个由大型语言模型（LLMs）驱动的开创性多智能体协作框架，用于交互式模拟布局设计。LayoutCopilot通过将自然语言指令转换为可执行的脚本命令，简化了人机交互，并将高层设计意图转化为可操作的建议，从而显著简化了设计过程。实验结果表明，LayoutCopilot在处理实际模拟设计方面具有灵活性、效率和可访问性。

###### 索引词：

大型语言模型、多智能体、模拟布局设计、交互式布局编辑。

## I 引言

模拟布局设计是模拟电路设计中的关键阶段，依赖于经验丰富的设计师的手工操作，如图[1](https://arxiv.org/html/2406.18873v2#S1.F1 "Figure 1 ‣ I Introduction ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design") A所示。这种依赖主要是由于模拟电路性能模型的复杂性，以及对称性、匹配性、信号流和其他约束条件等额外考虑因素的存在，这些因素使得生成具有优异性能的高质量布局面临巨大挑战。因此，高效地生成高质量的模拟电路布局是商业工具和学术研究的重大挑战。

商业模拟布局设计平台，如 Cadence Virtuoso [[1](https://arxiv.org/html/2406.18873v2#bib.bib1)]，主要提供手动布局绘制界面以及少数交互式布局自动化功能。然而，由于操作界面中复杂的按钮、快捷键和脚本命令，这些平台的学习曲线非常陡峭。它们的交互式布局自动化功能要求设计师输入所有约束条件，这非常繁琐，且在实际中很少被设计师采用。

学术研究关注了全自动模拟设计方法和交互式自动化。过去四十年来，提出了一系列模拟布局与布线算法，以实现布局设计的自动化。这些算法探索了包括传统算法方法 [[2](https://arxiv.org/html/2406.18873v2#bib.bib2), [3](https://arxiv.org/html/2406.18873v2#bib.bib3), [4](https://arxiv.org/html/2406.18873v2#bib.bib4), [5](https://arxiv.org/html/2406.18873v2#bib.bib5), [6](https://arxiv.org/html/2406.18873v2#bib.bib6)]、基于领域知识的方法 [[7](https://arxiv.org/html/2406.18873v2#bib.bib7), [8](https://arxiv.org/html/2406.18873v2#bib.bib8), [9](https://arxiv.org/html/2406.18873v2#bib.bib9), [10](https://arxiv.org/html/2406.18873v2#bib.bib10), [11](https://arxiv.org/html/2406.18873v2#bib.bib11)]，以及机器学习技术的融合 [[12](https://arxiv.org/html/2406.18873v2#bib.bib12), [13](https://arxiv.org/html/2406.18873v2#bib.bib13), [14](https://arxiv.org/html/2406.18873v2#bib.bib14)]等多种方法和视角。所有这些方法的目标都是提高最终布局结果的效率和性能，为全自动工具的发展铺平道路。最近，类似ALIGN [[15](https://arxiv.org/html/2406.18873v2#bib.bib15), [16](https://arxiv.org/html/2406.18873v2#bib.bib16), [17](https://arxiv.org/html/2406.18873v2#bib.bib17)]和MAGICAL [[18](https://arxiv.org/html/2406.18873v2#bib.bib18), [19](https://arxiv.org/html/2406.18873v2#bib.bib19), [20](https://arxiv.org/html/2406.18873v2#bib.bib20)]等全自动工具的进展，已显著提高了模拟布局生成的设计效率。然而，尽管这些工具取得了进展，它们通常仍未能满足模拟布局设计中高度定制化的需求，如图 [1](https://arxiv.org/html/2406.18873v2#S1.F1 "Figure 1 ‣ I Introduction ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design") B所示。为了解决这一问题，已经开发了交互式模拟布局编辑工具 [[21](https://arxiv.org/html/2406.18873v2#bib.bib21), [22](https://arxiv.org/html/2406.18873v2#bib.bib22)]，以便设计师可以更容易地修改和优化布局。这些工具虽然在布局设计上提供了更大的灵活性，但与商业工具类似，它们也带来了新的可用性挑战，因为设计师必须掌握复杂的命令集并有效地应用于电路优化中，如图 [1](https://arxiv.org/html/2406.18873v2#S1.F1 "Figure 1 ‣ I Introduction ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design") C所示。

上述的可用性挑战源于人类自然语言与机器语言之间的根本差距。将设计师的具体任务和抽象设计意图转化为机器可以执行的指令并不容易。近年来，**大规模语言模型（LLMs）**的进展为弥合这一差距提供了新的机会。近期流行的模型，如GPT-3 [[23](https://arxiv.org/html/2406.18873v2#bib.bib23)]、Llama [[24](https://arxiv.org/html/2406.18873v2#bib.bib24), [25](https://arxiv.org/html/2406.18873v2#bib.bib25)]、GPT-4 [[26](https://arxiv.org/html/2406.18873v2#bib.bib26)]和Claude [[27](https://arxiv.org/html/2406.18873v2#bib.bib27), [28](https://arxiv.org/html/2406.18873v2#bib.bib28)]，不仅在理解和生成类人文本方面展现了卓越的能力，还在推理和理解抽象领域知识上表现出色，为各个领域的创新应用铺平了道路。这一演变，加上知识检索技术的进步[[29](https://arxiv.org/html/2406.18873v2#bib.bib29)]以及多代理方法在应对复杂推理任务中的 proven effectiveness[[30](https://arxiv.org/html/2406.18873v2#bib.bib30), [31](https://arxiv.org/html/2406.18873v2#bib.bib31), [32](https://arxiv.org/html/2406.18873v2#bib.bib32)]，为提升复杂技术任务中的人机交互开辟了新的阶段。这些进展已在电子设计自动化（EDA）[[33](https://arxiv.org/html/2406.18873v2#bib.bib33), [34](https://arxiv.org/html/2406.18873v2#bib.bib34), [35](https://arxiv.org/html/2406.18873v2#bib.bib35), [36](https://arxiv.org/html/2406.18873v2#bib.bib36), [37](https://arxiv.org/html/2406.18873v2#bib.bib37), [38](https://arxiv.org/html/2406.18873v2#bib.bib38), [39](https://arxiv.org/html/2406.18873v2#bib.bib39), [40](https://arxiv.org/html/2406.18873v2#bib.bib40)]中得到了广泛应用，但主要集中在数字电路领域，那里成熟的自动化解决方案已经得到了广泛采用，且不依赖于复杂的人机交互。因此，模拟布局设计领域提供了一个独特的机会，因为尚未充分探索使用LLMs来解决布局工具的可用性挑战。

![参考标题](img/2290f49c92d14fcc0c3180e04adb619c.png)

图1：模拟布局自动化工作流程的比较，突出手动、全自动、交互式和LLM驱动的交互工具（LayoutCopilot）的关键优缺点

本文介绍了LayoutCopilot，一个由LLM驱动的开创性多代理协作框架，用于交互式模拟布局设计。LayoutCopilot擅长处理具体的布局调整请求，将其转化为可执行的命令，从而降低了与交互式工具相关的学习曲线，如图[1](https://arxiv.org/html/2406.18873v2#S1.F1 "Figure 1 ‣ I Introduction ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design") D所示。此外，它利用一个全面的知识库，根据设计师的高层次设计意图生成实际建议，从而解决了模拟设计固有复杂性带来的可用性挑战。本文的主要贡献可以总结如下：

+   •

    据我们所知，LayoutCopilot是第一个由LLM驱动的交互式模拟布局设计框架，提供了一种增强模拟电路设计中设计灵活性和效率的创新方法。

+   •

    我们通过一个由LLM驱动的框架弥合了设计师与模拟布局工具之间的互动鸿沟，克服了布局工具的可用性挑战，并完善了交互式布局设计的方法论。

+   •

    我们利用一个多代理协作框架，通过多个LLM代理之间的协调努力，逐步将设计师的需求转化为可执行命令，从而实现高准确性。

+   •

    对现实世界的模拟设计进行的大规模测试和实验表明，LayoutCopilot能够准确解决设计师的具体布局调整请求，并根据他们的高层次设计意图提供可操作的建议，从而完成布局优化。

本文的其余部分结构如下：第[II](https://arxiv.org/html/2406.18873v2#S2 "II Preliminaries ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")节描述了背景；第[III](https://arxiv.org/html/2406.18873v2#S3 "III LayoutCopilot Framework ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")节解释了详细实现；第[IV](https://arxiv.org/html/2406.18873v2#S4 "IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")节展示了结果；第[V](https://arxiv.org/html/2406.18873v2#S5 "V Conclusion ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")节对本文进行了总结。

## II 前言

本节回顾了我们研究的背景概念，包括LLM与EDA的集成、提示工程、多代理协作，以及模拟布局设计中的交互式布图与布线，并概述了LayoutCopilot的范围。

### II-A 将LLM集成到EDA应用中

最近，预训练的大型语言模型（LLMs）的进展为提升EDA应用提供了新的机会。LLMs生成类人文本和理解复杂概念的能力，使其非常适合集成到EDA任务中，涵盖从自动生成硬件描述语言（HDL）代码到通过对话界面促进交互式设计工作流等方面。

一项研究方向集中于利用大型语言模型（LLMs）自主处理电子设计自动化（EDA）中的文本或基于语言的任务。这包括使用RTLCoder和VeriGen等工具自动生成HDL代码[[33](https://arxiv.org/html/2406.18873v2#bib.bib33), [34](https://arxiv.org/html/2406.18873v2#bib.bib34)]，以及RTLLM和VerilogEval等基准工具，用于评估这些能力[[35](https://arxiv.org/html/2406.18873v2#bib.bib35), [36](https://arxiv.org/html/2406.18873v2#bib.bib36)]。此外，RTLFixer[[37](https://arxiv.org/html/2406.18873v2#bib.bib37)]正在探索自动化调试和代码修复，而ChipNeMo[[38](https://arxiv.org/html/2406.18873v2#bib.bib38)]则作为一款工程助手聊天机器人，帮助生成EDA脚本并进行错误分析。

另一个方向是利用LLMs帮助传统设计流程和现有的EDA工具，从而提高它们的可访问性和易用性。典型例子包括ChatEDA[[39](https://arxiv.org/html/2406.18873v2#bib.bib39)]和ChatPattern[[40](https://arxiv.org/html/2406.18873v2#bib.bib40)]，这些工具通常具备对话界面，允许设计师用自然语言表达需求，从而通过LLMs间接操作EDA工具。引入由LLMs驱动的对话界面，有助于解决EDA工具长期存在的可用性挑战。受EDA工具中对话界面的启发，我们将LLMs应用于交互式模拟布局设计。考虑到模拟电路的定制化特性，LayoutCopilot不仅能够直接执行命令，还能根据高级设计意图生成可操作的建议，从而简化设计过程。

### II-B 提示工程

#### II-B1 提示工程简介

提示是提供上下文以引导生成性语言模型的自然语言指令。提示工程是通过使用提示来增强模型效能的过程，而无需修改核心模型的参数[[41](https://arxiv.org/html/2406.18873v2#bib.bib41)]。提示工程的基本概念涉及以一种能够让模型理解并准确回应的方式来制定查询或指令，利用其预训练知识。此外，提示可以构建或自动生成，以向模型传递超出预训练数据集的信息，从而灵活地调整模型的响应，并使外部知识库的集成成为可能。随着基于Transformer架构的语言模型在各个应用中的日益普及，制定有效的提示以引导这些模型生成有用且准确的输出，已变得至关重要。

相较于其他提升LLM性能的方法，提示工程提供了一种轻量级的解决方案，可以利用大型模型解决现实世界的问题。与传统方法需要创建大量数据集、微调以及反复调整模型以提高其在特定任务上的表现不同，提示工程允许从业者通过精心设计的提示直接向模型传递知识和方法。这种方法在数据获取和清洗具有挑战性的领域中尤为有效，例如EDA。

#### II-B2 应用与高级技术

语境学习是一种使语言模型通过仅从少量提供的示例中学习来执行任务的范式[[42](https://arxiv.org/html/2406.18873v2#bib.bib42)]。提示工程的一种方法是将这些示例直接包含在提示中，这些示例可以根据提供的示例数量分类：零-shot、one-shot 和 few-shot 学习[[43](https://arxiv.org/html/2406.18873v2#bib.bib43)]。虽然提示工程通常涉及提供此类示例，但并不限于这种方法。另一种方法是利用抽象语言通过链式思维和自我完善等技术，指导LLM的行为，从而进一步增强模型能力。

研究人员已经开发出先进的提示工程技术，以提升LLM的解决问题能力。在“few-shot学习”方法[[43](https://arxiv.org/html/2406.18873v2#bib.bib43)]的基础上，研究人员发现，示例的“抽象描述”有时足够使用，消除了直接对话示例的需求。以下是上述方法的一些示例：

+   •

    思维链（Chain-of-thought）引导LLM通过一系列中间步骤解决问题，这通过促使模型阐述一个多步骤的问题解决过程，模仿人类推理，从而提高LLM的推理能力[[44](https://arxiv.org/html/2406.18873v2#bib.bib44)]。

+   •

    从简单到复杂的提示（Least-to-most prompting）将复杂问题简化为可管理的子任务，按顺序逐一解决，显著提升了模型处理复杂推理任务的能力[[45](https://arxiv.org/html/2406.18873v2#bib.bib45)]。

+   •

    自我完善策略通过迭代过程提升LLM的性能，在该过程中，模型批判并修正自己的解决方案，从而改进其响应的质量[[46](https://arxiv.org/html/2406.18873v2#bib.bib46)]。

为了在我们的应用中处理复杂的推理任务，LayoutCopilot集成了多种提示工程技术，包括上述提到的技术。每个代理都根据其执行的特定任务量身定制，以实现更好的性能。有关提示结构的详细信息，请参见第[III](https://arxiv.org/html/2406.18873v2#S3 "III LayoutCopilot Framework ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")节。

#### II-B3 自动提示生成与检索增强生成（RAG）

![请参阅说明文字](img/c2e2f329202a36795cceccae9db86943.png)

图2：系统比较：通过多代理协作克服单一代理的局限性。

除了手动配置的提示外，还有一些自动生成提示的技术。一个经典的例子是检索增强生成（RAG）[[47](https://arxiv.org/html/2406.18873v2#bib.bib47)]，它增强了模型与外部知识库的交互。如前所述，提示通常包含一些示例，因此与“少量示例”（few-shot）方法相一致。RAG允许这些示例从数据库或知识库中自动检索，提供一个接口，使LLM能够访问外部知识并相应地调整其响应。然而，检索的上下文与系统指令和设计者请求一起，作为文本输入提供给LLM，这可能会导致提示稀释，尤其是当上下文过于冗长时。这会导致响应质量下降，我们在实验中观察到了这一问题。为了解决这个问题，我们采用了多代理方法，将不同任务分配给多个LLM代理，确保知识检索不会影响其他系统组件的性能。通过将具有不同责任的代理分开，即使在将上下文广泛的检索技术集成到我们的框架中时，也能保持系统的高效性。

### II-C 多代理协作与大型语言模型（LLMs）

多智能体协作的目标是使多个自主智能体能够有效地协作以实现共同的目标[[48](https://arxiv.org/html/2406.18873v2#bib.bib48)]。图[2](https://arxiv.org/html/2406.18873v2#S2.F2 "图 2 ‣ II-B3 自动提示生成与检索增强生成（RAG） ‣ II-B 提示工程 ‣ II 基础知识 ‣ LayoutCopilot：一个基于LLM的多智能体协作框架，用于交互式模拟布局设计")展示了单智能体系统和多智能体系统能力之间的对比，突出了多智能体协作如何通过利用每个智能体的专业知识并防止任务干扰，显著增强系统的能力。这使得系统能够管理比没有这种专业化时更大的工作量，从而提高了效率和输出质量[[49](https://arxiv.org/html/2406.18873v2#bib.bib49)]。

在引入角色扮演能力到交互式智能体互动中之后[[50](https://arxiv.org/html/2406.18873v2#bib.bib50)]，[[51](https://arxiv.org/html/2406.18873v2#bib.bib51)]提出了一个基于LLM的综合多智能体协作框架，展示了在处理复杂任务时效率的提升。基于这些研究，多智能体方法已经成功应用于多个领域，证明了其在文本理解、推理、数学、编程和工具利用等场景中的有效性[[52](https://arxiv.org/html/2406.18873v2#bib.bib52)、[53](https://arxiv.org/html/2406.18873v2#bib.bib53)、[54](https://arxiv.org/html/2406.18873v2#bib.bib54)]。此外，已经提出了若干改进措施以增强多智能体协作，包括[[55](https://arxiv.org/html/2406.18873v2#bib.bib55)]，将专业知识和管理经验传递给LLM智能体，以实现更结构化的协作，以及[[48](https://arxiv.org/html/2406.18873v2#bib.bib48)]，通过使智能体预测其合作者的行动来改善合作。

在LayoutCopilot中，我们开发了一个多智能体协作框架，将复杂任务分解为专业化的子任务。这种战略性的分工确保每个智能体都能在其专业领域内运作，而不会稀释提示或影响其他任务的表现。

### II-D 交互式模拟布局设计

近年来，模拟布局自动化吸引了许多研究者，几种基本方法论已经出现[[5](https://arxiv.org/html/2406.18873v2#bib.bib5), [6](https://arxiv.org/html/2406.18873v2#bib.bib6), [8](https://arxiv.org/html/2406.18873v2#bib.bib8), [9](https://arxiv.org/html/2406.18873v2#bib.bib9), [10](https://arxiv.org/html/2406.18873v2#bib.bib10), [11](https://arxiv.org/html/2406.18873v2#bib.bib11), [13](https://arxiv.org/html/2406.18873v2#bib.bib13), [14](https://arxiv.org/html/2406.18873v2#bib.bib14), [15](https://arxiv.org/html/2406.18873v2#bib.bib15), [16](https://arxiv.org/html/2406.18873v2#bib.bib16), [17](https://arxiv.org/html/2406.18873v2#bib.bib17), [18](https://arxiv.org/html/2406.18873v2#bib.bib18), [19](https://arxiv.org/html/2406.18873v2#bib.bib19), [20](https://arxiv.org/html/2406.18873v2#bib.bib20), [21](https://arxiv.org/html/2406.18873v2#bib.bib21), [22](https://arxiv.org/html/2406.18873v2#bib.bib22)]。模拟布局问题可以被表述为一个非线性优化问题。最常见的目标函数是半周长线长（HPWL），表示线路长度的性能。与数字布局不同，模拟布局需要考虑更多约束，如对称性、阵列等。模拟布线问题是一个路径寻找问题，可以通过典型的最短路径算法（A-star）来解决。

交互式模拟布局设计框架提供了灵活的布局调整，同时减轻设计师手动绘制布局的繁琐工作。通过引入高层次的交互控制器，框架可以利用放置和布线核心自动调整布局。基于此交互式布局设计框架，设计师可以根据其设计经验添加任意放置约束，并优化布线方案。经过多次调整，设计师最终可以获得高质量的布局。

### II-E LayoutCopilot的范围

LayoutCopilot作为一个多智能体协作框架，旨在增强设计师与布局工具之间的交互，如图[3](https://arxiv.org/html/2406.18873v2#S2.F3 "Figure 3 ‣ II-E The Scope of LayoutCopilot ‣ II Preliminaries ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")所示。其主要目标是解读以自然语言表达的设计意图，并生成可操作的命令，能够与布局工具进行有效的交互。

问题 1.（LayoutCopilot的目标）开发一个基于LLM的框架，能够接收设计师以自然语言表达的高层设计意图和具体需求，并将其转换为精确的可执行命令。这些命令旨在直接操作布局工具，确保设计意图能够准确地转化为布局调整。

开发LayoutCopilot的主要挑战在于协调系统内的多个任务，例如高层设计意图解析、设备级布局调整以及为布局工具生成精确命令——这些任务在复杂性和重点上各不相同。这些任务从设计意图解析到精确命令生成，单一代理系统在有效管理这些任务时可能面临困难。此外，集成知识库可能会消耗大量模型注意力，并干扰无关任务。为了解决这些问题，LayoutCopilot采用了多代理系统，将特定代理分配给固定任务，从而增强专注力和响应质量。该设置还允许代理在最小化对其他任务负面影响的同时，利用知识库，从而提升系统在将设计意图转化为可操作的布局命令时的准确性和效率。

![参见说明文字](img/f98a132601d3eb48baa74f4fa436acc1.png)

图3：LayoutCopilot功能的示意图。

![参见说明文字](img/fae885077c9ac1fed553550e30c88de8.png)

图4：LayoutCopilot概览：一个多代理框架，通过抽象和具体请求处理器解读自然语言设计意图，协调代理执行精确布局调整，以便进行交互式放置和布线。

## III LayoutCopilot框架

本节介绍LayoutCopilot的架构和功能，LayoutCopilot是一个由LLM增强的多代理协作框架，用于交互式模拟布局设计。LayoutCopilot通过理解自然语言中的复杂需求、利用知识库生成解决方案并自动化布局设计，增强了设计师与布局工具之间的互动。它由两个主要组件组成：抽象请求处理器和具体请求处理器，如图[4](https://arxiv.org/html/2406.18873v2#S2.F4 "图4 ‣ II-E LayoutCopilot的范围 ‣ II 前提 ‣ LayoutCopilot：一个由LLM驱动的多代理协作框架，用于交互式模拟布局设计")所示。抽象请求处理器启动请求的处理，将其转化为符合布局工具手册的具体请求。这些具体请求随后被传递到具体请求处理器，后者生成可执行命令以促进布局编辑。这一工作流程简化了布局工具与设计师之间的互动，提升了整体设计过程。关于每个组件功能的详细讨论以及设计理念将在后续小节中提供。

### III-A 抽象请求处理器

摘要请求处理器在LayoutCopilot中发挥着关键作用，负责管理和处理设计师的多样化请求。它执行一系列功能，包括请求分类、相关知识的检索、与设计师的协同迭代调整，以及分析网表以将抽象请求转化为与布局工具手册一致的具体请求。该组件由四个专门的LLM代理提供支持，每个代理专注于处理流水线的特定方面：分类、分析、解决方案优化和解决方案适配，如图[4](https://arxiv.org/html/2406.18873v2#S2.F4 "Figure 4 ‣ II-E The Scope of LayoutCopilot ‣ II Preliminaries ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")所示。

在我们讨论具体代理的构建之前，首先让我们解释一下为什么我们使用多个LLM代理来构建如此复杂的系统，而不是依赖单个LLM代理来解决所有问题。我们将任务分配给多个代理而不是依赖单个代理的原因，是为了缓解提示的“稀释”以及LLM注意力的分散。在我们的实验中，我们注意到每个LLM代理只能有效处理有限的上下文。当提供较长且复杂的指令作为提示时，LLM处理任务的表现比起仅提供简短指令时会有所下降。此外，如果指令中包含多个任务，它们往往会相互“稀释”，导致表现较差，相比于指令仅包含单一任务时的表现。这种现象类似于LLM的有限注意力在任务之间被分割开来。当涉及到长文本的知识检索时，这个问题尤其明显。在将知识库传输给LLM时，知识库要么直接作为提示输入，要么通过LLM提供商的检索接口输入，该接口也会在进入提示之前对内容进行压缩。在这两种情况下，提示的内容都会增加。即使经过压缩，知识库的内容仍然庞大，这意味着如果一个代理访问知识库，它只能有效地处理与该知识库相关的任务。由于提示的“稀释”，其他任务的表现将受到影响。

因此，我们采用多代理结构，将任务分成几个独立的部分并分配给不同的代理，以确保每个任务得到高效处理。这种方法有助于通过防止提示的稀释来保持每个代理的有效性，并使系统能够更有效地管理复杂和多样化的请求。这些代理的详细信息如下所示：

#### III-A1 分类器代理

分析器的顶部是一个任务分类代理，充当过滤器，根据布局工具手册将设计师的需求判定为具体请求或抽象请求。具体任务通过其能直接转化为命令或布局工具明确支持的组合来识别，例如，“在M6和M7之间添加对称性”。相反，抽象请求因其高层次的概念性特点而被识别，需要基于电路的网表和布局设计的知识进行全面分析，才能被分解为可执行步骤，例如“增强匹配”和“提高共模抑制比（CMRR）”。基于这些标准，LLM代理执行任务分类。一旦分类，任务将分为两种不同的流程：具体任务直接发送到具体请求处理器进行即时处理。同时，抽象请求需要经过一系列步骤转化为具体请求，然后以与具体请求相同的方式进行处理。如果没有这一过程，抽象请求可能会直接发送到具体请求处理器，从而导致任务处理错误。

![参见说明](img/b96942329e8db2bb9c69c9b88d6bb69a.png)

图5：LayoutCopilot中单个代理的配置示意图。

#### III-A2 分析器代理

分析器代理专门用于分析和知识检索任务，这在大语言模型（LLMs）在长上下文长度方面的局限性下显得尤为重要。需要大量上下文的任务可能会消耗掉大部分容量，从而稀释提示并降低其他任务的效果，如上所述。为避免这些问题，该代理专注于知识检索。分析器代理运行在一个庞大的知识库中，该知识库涵盖了模拟电路和布局设计的专业文献、布局工具手册以及以往任务和解决方案的档案。当将知识库传输给LLMs时，它们要么直接作为提示的一部分输入，要么通过LLM提供商的检索接口进行输入。

分析器代理根据检索到的知识制定高层次的解决方案，其功能类似于一位熟练的建筑师在施工前绘制蓝图并概述任务的战略方法。例如，在被要求“提高OTA的CMRR”时，代理可能建议优化组件放置以增强对称性，并重新布线连接以改善寄生匹配，这一切都根据所选布局工具的具体功能量身定制。然后，高层次的解决方案会发送给解决方案优化代理，以便根据设计师的修改建议进行优化和调整。如果设计师对结果不满意，分析器代理将会被重新调用。包含设计师修改建议的修订请求会由解决方案优化代理返回，作为分析器代理生成优化后的高层次解决方案的新输入。该机制确保系统能够融入设计师反馈，以便改善设计方案，提供更加准确和针对性的结果。

#### III-A3 解决方案优化代理

考虑到模拟设计师的丰富经验和高度定制化的需求，加强他们在工作流中的参与至关重要。解决方案优化代理通过向设计师呈现高层次解决方案以及详细的评估和解释来实现这一点。这使得设计师能够理解每个解决方案背后的逻辑，评估其可行性，并做出明智的决策。设计师可以从这些解决方案中选择，按需修改，或者提供额外的反馈来进一步优化结果。如果设计师对解决方案不满意，他们可以选择添加更多细节或返回分析器代理进行调整。这个迭代反馈过程确保高层次解决方案会根据设计师的专业知识不断优化，最终使最终布局符合设计师的具体需求和项目要求。一旦解决方案经过优化并达到设计师的满意，便会传递给解决方案适配代理。为了更清楚地说明这个过程，以下示例对话展示了解决方案优化代理、其他代理和设计师之间的互动。此示例还展示了代理之间的互动过程；其他代理的对话结构类似，因此为了简洁起见省略。

<svg class="ltx_picture" height="211.77" id="S3.SS1.SSS3.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,211.77) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 9.13 194.5)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="581.75">示例对话：解决方案优化器代理交互</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 9.13 9.13)"><foreignobject color="#000000" height="172.31" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="581.75">给设计师：提高你的OTA布局性能可能涉及…

以下是一些潜在的高层解决方案：1. 使用symAdd增强对称性… 2. 使用deviceMove和deviceSwap改进匹配… 3. 使用wireWidth和wireSpacing减少寄生效应… 4. 使用wireSpacing防止串扰… 5. 使用netPriority和netTopology改进布线…请选择一些你希望执行的高层解决方案，或提供任何修改或建议。一旦确认，我们将把这些转化为具体的实施请求。设计师：我更倾向于增强对称性。给设计师：很好的选择！你已选择使用symAdd函数来增强对称性。我现在将这个任务转发给解决方案适配器进行进一步交互…给适配器：我们已收到设计师选择使用symAdd函数来增强对称性的选项。以下是增强布局对称性的详细要求：1. 确认细节：从网络列表和设计师处收集symAdd函数所需的任何参数或具体配置。2. 与设计师沟通：如果需要任何澄清或额外输入，请联系设计师获取反馈或修改。3. 准备具体请求：一旦确认所有细节，将任务转化为可操作的具体请求，并附上足够且正确的参数，以便进一步处理。

#### III-A4 解决方案适配器代理

解决方案适配器代理是抽象请求处理器的最后一个组成部分，负责将前一步精细化的请求转换为具体、可操作的请求。根据复杂性和需求，电路网表要么在对话过程中直接传递给大型语言模型（LLMs），要么存储在通过LLMs检索接口访问的知识库中。该代理仔细分析电路网表，按照精细化解决方案的指示提取特定参数。例如，当收到一个精细化解决方案，要求“增强差分对和电容器的对称性”时，解决方案适配器代理通过检查其连接性和设备参数，在电路网表中定位这些组件。然后，它制定具体请求，将对称性约束应用于已识别的组件，如“在M6和M7之间添加对称性”。没有解决方案适配器代理，精细化请求可能缺乏实施所需的具体性，从而导致不完整或错误的操作。

除了从电路网表中提取参数外，解决方案适配器代理还确保生成的请求与布局工具兼容，遵循这些工具的特定语法和操作要求。这种兼容性对于将精细化的解决方案转化为可操作的步骤至关重要，架起了设计意图与实际实施之间的桥梁。没有这个代理，发送到具体请求处理器的请求可能无法处理，导致错误或生成的命令执行不完全。

### III-B 具体请求处理器

具体请求处理器致力于将具体任务准确地转化为可执行的命令，通过布局工具满足设计师需求。该过程首先将具体任务分解为子任务，每个子任务对应于布局工具手册中详细的命令，类似于[[39](https://arxiv.org/html/2406.18873v2#bib.bib39)]和[[40](https://arxiv.org/html/2406.18873v2#bib.bib40)]中使用的过程。然而，仅仅采用这些方法并不能保证生成的命令能够正确执行或在此场景中实现预期结果。与以往的工作不同，我们的方法结合了角色扮演和任务分解技术，同时将布局工具手册中规定的语法和逻辑规则传达给代码生成代理。这一增强确保了代理生成的命令序列在语法和逻辑上保持高度准确，特别是在处理复杂输入请求时。这一战略性增强致力于提高过程的精确性和可靠性，紧密对接布局工具的具体操作要求。

### III-C 代理配置与提示设计

LayoutCopilot 框架中的每个代理都配备了针对其指定任务量身定制的提示。这些提示使用第 [II](https://arxiv.org/html/2406.18873v2#S2 "II Preliminaries ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")节中描述的提示工程技术精心设计，旨在基于实验结果增强代理的整体表达能力，如图 [5](https://arxiv.org/html/2406.18873v2#S3.F5 "Figure 5 ‣ III-A1 Classifier Agent ‣ III-A Abstract Request Processor ‣ III LayoutCopilot Framework ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")所示。

七个提示部分的详细描述如下：

+   •

    I. 角色扮演鼓励代理采纳一个角色，帮助模拟现实且专业的场景以处理任务。

+   •

    II. 工作流概览为代理提供了对其在整个系统中角色的理解，突出了他们的输出如何影响设计过程的后续阶段。

+   •

    III. 任务描述明确了代理的具体责任和目标，确保任务执行的专注和相关性。

+   •

    IV. 管道通过连贯的思维链引导代理高效地完成分配的任务。这增强了代理的推理能力，并引导他们将复杂任务分解成可管理的子任务，从而提高了处理复杂问题的能力。

+   •

    V. 信息验证指示代理在继续之前验证输入的完整性，必要时提示补充信息。这作为一种自我完善策略，有效防止因输入不完整或不准确而产生的错误。

+   •

    VI. 互动指南指导代理如何与设计师及其他代理进行互动，确保整个过程保持连贯高效。

+   •

    VII. 外部知识使每个代理能够访问与其角色相关的资源，通过特定领域的洞察力增强输出。提供给每个代理的知识与其功能需求精确对接。有关模拟和布局设计的书籍[[56](https://arxiv.org/html/2406.18873v2#bib.bib56)，[57](https://arxiv.org/html/2406.18873v2#bib.bib57)]仅提供给分析器代理，以建立一个全面的知识库，避免其他代理受到干扰，因为这些大量的资料可能会妨碍它们的任务。具体来说，这些书籍以文本形式提供，主要作为一般的路线图。此外，所有代理都可以访问布局工具手册、命令列表（如表 I 所示）以及请求处理的示例。这些资源以结构化文本格式提供，并附有注释，帮助大型语言模型（LLM）有效理解和使用这些信息，确保与工具语法兼容，并指导框架中任务执行的精确性和一致性。

关于提示的细节，我们以解决方案优化器代理的 Pipeline 部分为例进行展示，因为它是最复杂的代理。

<svg class="ltx_picture" height="263.93" id="S3.SS3.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,263.93) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 9.13 246.66)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="581.75">Solution Refiner Agent Prompt: Pipeline Section</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 9.13 9.13)"><foreignobject color="#000000" height="224.47" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="581.75">Please proceed according to the following pipeline, depending on the source of the request: *If the request comes from the Analyzer Agent, follow Pipeline A; if the request comes from the Designer, follow Pipeline B.* ### Pipeline A: Request from **Analyzer Agent** **Input:** A series of layout modification suggestions (i.e. high-level solutions) generated by the Analyzer Agent: high_level_solutions **Steps:** - (a) Organize and Present High-Level Solutions: Compile the layout modification suggestions into a clear and structured format. Provide necessary explanations for each solution to help the Designer understand their implications and potential benefits. - (b) Engage with the Designer: Present the organized solutions to the Designer for feedback. **Output:** Please distinguish the output given to each recipient with ”—To XXX—”. - **To Designer:** Organized high-level solutions with explanations for review. ### Pipeline B: Request from **Designer** **Input:** Designer feedback on previously presented high-level solutions: designer_feedback **Steps:** - (a) If the Designer is satisfied with a specific solution, compile the finalized solution for further interaction. - (b) If the Designer requests modifications, document the feedback in a structured format and send it back to the Analyzer Agent for refinement. **Output:** Please distinguish the output given to each recipient with ”—To XXX—”. - **To Adapter Agent (if Designer is satisfied):** Refined solution(s) finalized by Designer. - **To Analyzer Agent (if Designer requests modifications):** Designer’s feedback and modification requests for further analysis.</foreignobject></g></g></svg>

对于来自分析器代理的请求，高层次的解决方案通过 Pipeline A 组织并清晰地呈现给设计师，并通过解释帮助设计师做出知情决策。当收到设计师的反馈时，提示将引导代理要么将批准的解决方案发送给适配器代理，要么通过 Pipeline B 将修改请求返回给分析器代理。每个步骤都包括明确的控制流、输入和输出描述以及思维链，确保代理间互动的清晰性和精准性。

总结来说，这些提示组件确保每个代理不仅在其各自领域内作为专家运作，而且作为整个系统的一部分协同工作，从而增强了 LayoutCopilot 的整体功能性和效率。

### III-D 交互式布局编辑器

![参见说明](img/049b6a57e000388482964ffd2d5a5ec3.png)

图 6：交互式布局编辑器概览。

表 I：交互式命令集。

| Command | 参数 |
| --- | --- |
| deviceMove | 设备 $v_{i}$，目标位置 $(x,y)$ |
| deviceSwap | 设备 $v_{i},v_{j}$ |
| arrayAdd | 设备列表 ${v_{i}}$，阵列形状 ${x,y}$ |
| arraySpace | 一个设备组 $g_{i}$，水平空间 $x$，垂直空间 $y$ |
| symAdd | 设备 $v_{i},v_{j}$，对称轴 $A_{k}$ |
| netRemove | 网 $n_{i}$ |
| netReroute | 网 $n_{i}$ |
| wireWidth | 网 $n_{j}$ 的导线 $w_{i}$，新的导线宽度 $W$ |
| wireSpacing | 网 $n_{m}$ 的导线 $w_{i}$，网 $n_{t}$ 的导线 $w_{j}$，间距宽度 $S$ |
| 网 $n_{m}$ 的导线 $w_{i}$，设备 $v_{j}$，间距宽度 $S$ |
| netPriority | 包含其优先级的网表 ${n_{i}}$ 列表 |
| netTopology | 网 $n_{i}$，带有点 ${p_{i}}$ 的粗略指南 |

交互式布局编辑器填补了完全自动化的模拟布局工具与现实工业应用之间的空白。我们的交互式模拟编辑器基于之前的交互式工作[[21](https://arxiv.org/html/2406.18873v2#bib.bib21), [22](https://arxiv.org/html/2406.18873v2#bib.bib22)]，并进行了扩展。图[6](https://arxiv.org/html/2406.18873v2#S3.F6 "Figure 6 ‣ III-D Interactive Layout Editor ‣ III LayoutCopilot Framework ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")展示了交互式布局编辑器的概览。在交互式调整流程开始之前，初步布局是由完全自动化的工具提供的。这些工具以网表、技术文件和一些基本约束作为输入。基于初步布局，用户可以通过用户界面更加直观地开始进行交互式布局调整。用户界面由自然语言界面和布局图形用户界面（GUI）组成。与之前的基于脚本的界面[[21](https://arxiv.org/html/2406.18873v2#bib.bib21), [22](https://arxiv.org/html/2406.18873v2#bib.bib22)]相比，用户可以直接使用自然语言调整模拟布局。在将自然语言转换为命令集中的一系列命令后，如表[I](https://arxiv.org/html/2406.18873v2#S3.T1 "TABLE I ‣ III-D Interactive Layout Editor ‣ III LayoutCopilot Framework ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")所示，命令解释器会将每个命令分解为放置和布线操作。调整后，布局结果将在GUI中显示。用户可以根据给定的结果进一步调整布局。

对于放置调整，用户提供了七个不同的命令。从理论上讲，可以通过这些命令的组合实现任意的放置调整。move命令可以将设备移动到指定位置。swap命令可以交换两个给定设备的位置。对于基于阵列的调整，arrayAdd命令将阵列约束添加到一组设备中，arraySpace命令调整阵列组内设备之间的间距。通过symAdd命令，可以添加对称约束。使用这些命令，用户可以轻松地进行放置调整。例如，之前由Solution Refiner Agent决定增强差分对的对称性，而Solution Adapter Agent识别了一个差分对{M6, M7}。然后，布局修改可以通过执行由Concrete Request Processor生成的symAdd M6 M7来实现。

在布线调整方面，用户提供了六个不同的命令。`remove` 和 `reroute` 是基础命令，用于移除不满足要求的线路并重新布线。`wireWidth` 命令可以更改给定网或线路的宽度，以提高布局性能。`wireSpacing` 命令可以调整两个网之间的间距，以避免信号串扰。`netPriority` 命令可以指定网的特定布线优先级。`netTopology` 命令可以设置手动指导，最终的布线方案将严格遵循该指导。有了这些命令，用户可以完成任意的布线调整。

## IV 实验结果

在本节中，我们将分别展示针对具体请求和抽象请求的实验，展示LayoutCopilot在处理基础任务时的准确性，以及其管理复杂和全面电路优化任务的能力。LayoutCopilot适配各种大型语言模型（LLMs）和布局工具，便于在不同环境中灵活部署。在我们的实验中，我们使用了一个成熟的互动模拟布局设计工具[[21](https://arxiv.org/html/2406.18873v2#bib.bib21), [22](https://arxiv.org/html/2406.18873v2#bib.bib22)]来进行演示。此外，我们还使用了不同版本的主流LLMs，展示了LayoutCopilot的多功能性，包括GPT-3.5 [[23](https://arxiv.org/html/2406.18873v2#bib.bib23)]、GPT-4 [[26](https://arxiv.org/html/2406.18873v2#bib.bib26)]和Claude-3 [[58](https://arxiv.org/html/2406.18873v2#bib.bib58)]。

表 II：单代理带指令与多代理有无指令的理智性检查与比较。

| 类别 | 单代理带指令 | 多代理不带指令 | 多代理带指令 |
| --- | --- | --- | --- |
| GPT-3.5 | GPT-4 | Claude-3 | 平均值 | GPT-3.5 | GPT-4 | Claude-3 | 平均值 | GPT-3.5 | GPT-4 | Claude-3 | 平均值 |
| 格式化 | 71.14% | 90.91% | 99.25% | 87.20% | 82.00% | 97.92% | 99.83% | 93.25% | 95.38% | 99.76% | 99.92% | 98.26% |
| 有效性 | 91.36% | 93.60% | 95.44% | 93.47% | 96.88% | 99.20% | 98.40% | 98.16% | 98.24% | 99.28% | 98.88% | 98.77% |
| 语法 | 67.11% | 88.87% | 95.24% | 83.74% | 79.20% | 95.28% | 97.76% | 90.75% | 92.65% | 97.20% | 96.96% | 95.60% |
| 逻辑 | 66.44% | 83.04% | 91.67% | 80.38% | 76.16% | 93.44% | 95.20% | 88.27% | 91.24% | 94.24% | 98.80% | 94.76% |
| 总体 | 66.27% | 82.91% | 90.77% | 79.98% | 75.76% | 93.36% | 94.56% | 87.89% | 90.92% | 93.92% | 96.80% | 93.75% |

### IV-A 具体请求：综合评估

为了评估我们框架的有效性以及其生成输出的质量，我们在具体请求上进行了实验。这些请求采用自然语言表达，可以直接转换为后端布局工具的命令序列。我们设计的具体请求包含了详细的场景和明确的命令指令，旨在模拟设计师与LayoutCopilot交互的情形。由于真实IC设计数据的可获取性有限，我们选择了合成数据进行大规模测试。为了更清楚地了解生成请求的类型，下面提供了一些示例，代表了我们评估中使用的典型测试案例。

<svg class="ltx_picture" height="197.51" id="S4.SS1.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,197.51) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 9.13 180.24)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="581.75">Example Requests for Testing Case</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 9.13 9.13)"><foreignobject color="#000000" height="158.05" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="581.75">Request 1: I have received a partially completed analog layout design, and I need to make some adjustments to improve the overall performance and routing efficiency. First, I would like to add symmetry constraints between devices M1 and M3, as well as M5 and M7, since they are part of a differential pair and should be laid out symmetrically. Next, I need to swap the positions of devices M11 and M13, as the current placement is causing significant routing congestion in that area of the layout. After making these initial placement changes, I will run the route command to see the updated routing results. Request 2: Looking at the routing, I notice that net2 and net4 are critical signals that need to be as closely matched in length as possible. To achieve this, I will use the netTopology command to add guide points for net2 at coordinates (4500, 6800) and for net4 at (4800, 6800), so that the router can try to route these nets in a more symmetric fashion. In addition, I have identified a section of parallel alignment between net9 and net11 that is too close for comfort. I will use the wwSpacing command to increase the spacing between wire3 of net9 and wire5 of net11 to 150 units in the horizontal direction, to ensure proper signal integrity.</foreignobject></g></g></svg>

我们利用GPT-3.5 [[43](https://arxiv.org/html/2406.18873v2#bib.bib43)] 和Claude-3 [[58](https://arxiv.org/html/2406.18873v2#bib.bib58)] 共生成了1,250个案例，其中包括1,134个有效的具体请求和116个无效请求。这些无效请求通常缺少必要的参数，或者描述的操作顺序违反了布局工具手册中的预定义规则，我们已手动验证这些请求的无效性。每个有效请求可以通过表格[I](https://arxiv.org/html/2406.18873v2#S3.T1 "TABLE I ‣ III-D Interactive Layout Editor ‣ III LayoutCopilot Framework ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")中的5到40条命令完成，确保了请求场景的多样性。LayoutCopilot处理每个具体请求，生成布局工具可以执行的命令序列，以满足要求。为了全面验证LayoutCopilot的性能，我们对这些测试案例进行了正常性检查和功能性检查，具体如下：

#### IV-A1 正常性检查

为了验证我们代理配置的有效性，我们在两种不同的设置下执行实验：一种是通过提示工程实现LLM的全面配置，如第[III-C节](https://arxiv.org/html/2406.18873v2#S3.SS3 "III-C 代理配置和提示设计 ‣ III LayoutCopilot框架 ‣ LayoutCopilot: 基于LLM的多代理协作框架，用于互动模拟布局设计")中所述，另一种则是仅提供LLM布局工具手册以及包含输入输出要求的简单任务描述。此外，还通过在指令化条件下使用单一代理设置进行对比实验，以展示多代理方法的必要性。在多代理场景中，LayoutCopilot充当多代理协作框架，利用其具体请求处理器来管理来自测试集的请求。相反，单一代理配置将第[III节](https://arxiv.org/html/2406.18873v2#S3 "III LayoutCopilot框架 ‣ LayoutCopilot: 基于LLM的多代理协作框架，用于互动模拟布局设计")中提到的所有代理提示合并为一个LLM代理，旨在涵盖LayoutCopilot的所有功能，以处理测试请求。为了最小化不同LLM引擎对实验结果的影响，采用GPT-3.5、GPT-4和Claude-3作为LLM引擎进行批量测试。

关于测试标准，完整性检查涵盖了格式、有效性、语法、逻辑规则和整体准确性。（i）对于输出格式，代理需要在常规文本对话后生成处理状态和结果，并使用JSON格式，确保布局工具能够准确解读内容并进行必要的布局调整。（ii）有效性通过系统正确识别和响应无效输入的比率来衡量。我们期望LayoutCopilot能够检测到无效输入并向设计师提供反馈，例如缺少特定参数的命令。（iii）和（iv）我们进行语法和逻辑验证。根据表格[I](https://arxiv.org/html/2406.18873v2#S3.T1 "TABLE I ‣ III-D Interactive Layout Editor ‣ III LayoutCopilot Framework ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")中的命令集，我们建立了四条语法规则和两条逻辑规则，以确保命令序列的完整性和逻辑性。例如，一条语法规则确保每个命令包含正确数量的参数，而一条逻辑规则则规定一个设备不能出现在多个对称对中。由于篇幅限制，我们在此不列出所有规则。（v）成功处理一个请求，生成符合这些规则的正确格式化代码或准确识别非法请求，视为正确处理，并计入整体准确性统计。

我们在先前提到的1,250个案例的测试集上进行实验，遵循既定的标准，结果如表[II](https://arxiv.org/html/2406.18873v2#S4.T2 "TABLE II ‣ IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")所示。首先，很明显，指令显著提高了各个LLM引擎的准确性，逻辑验证平均提高了6.49%，语法验证提高了4.85%，整体准确率提高了5.89%。这验证了我们在子节[III-C](https://arxiv.org/html/2406.18873v2#S3.SS3 "III-C Agent Configuration and Prompt Design ‣ III LayoutCopilot Framework ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")中描述的LLM代理设计的有效性。由于没有指令的单代理情况正确率过低，无法提供有意义的信息，因此未列出该案例，而多代理系统有无指令的比较足以说明上述结论。其次，无论使用哪种LLM引擎，单代理系统的准确率始终低于多代理系统。这证明了在该应用场景中引入多代理方法的有效性。此外，在Claude-3引擎下使用指令时，语法性能略有下降，这可能反映了一种性能平衡：尽管指令可能会略微降低已处理得较好的任务的表现，但它们能改善较弱领域的表现，从而提升整体性能。没有指令时，某些任务的表现可能会更好，但较弱的领域会降低整体表现。因此，指令的价值在于它们能够增强整体性能平衡。最后，在使用Claude-3引擎、多代理配置和指令的条件下，健全性检查的整体正确率达到了96.80%。这确认了LayoutCopilot的综合能力足以满足实际应用的需求。

#### IV-A2 功能性检查

![参见标题](img/5ad6b5d80808c6430c7667b41493f9ea.png)

图7：不同LLM引擎在不同指令条件下的功能性检查。

为了进一步评估LayoutCopilot在测试集上生成输出的功能正确性，我们随机从1,250个通过合理性检查的生成结果中选择25个（约占2%），对其进行人工功能性检查，以验证它们是否完全实现了相应的请求。与[[39](https://arxiv.org/html/2406.18873v2#bib.bib39)]中使用的标准相似，我们将输出结果分为三个等级：A、B和C。Level A表示输出功能正确，且具有清晰的分析推理。Level B表示输出功能有轻微瑕疵，但分析推理清晰，设计师能够轻松识别并修正问题。Level C表示输出在功能和概念上均失败，几乎没有提供有价值的信息来帮助诊断问题。在实际场景中，LayoutCopilot提供不正确、没有依据且难以修复的建议是我们最后不能接受的。因此，消除C级输出是我们设计工作的重点。

我们使用混凝土请求处理器进行测试，分别在有指令和无指令的情况下进行，对应表格2中的多代理部分。由于单代理场景在合理性检查中的通过率显著较低，未提供有价值的比较视角，因此被省略。如图[7](https://arxiv.org/html/2406.18873v2#S4.F7 "Figure 7 ‣ IV-A2 Functionality Check ‣ IV-A Concrete Requests: Comprehensive Evaluation ‣ IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")所总结，带有指令的功能性检查结果明显优于没有指令的结果，在不同的LLM引擎下均是如此：一方面，在无指令的情况下，Level C的比例并不罕见，而在有指令的情况下几乎消失，GPT-3.5的比例不超过8%。这表明，正如3.3节所讨论的，指令的使用大大提高了LayoutCopilot的实用性，在这种情况下，大多数输出要么是正确的，要么可以通过简单的互动轻松纠正。另一方面，Level A的比例从16%增加到32%，在GPT-4的最高情况下达到92%。此外，无论使用哪种LLM引擎，带有指令的C级比例几乎为零。这表明，LayoutCopilot能够在各种LLM引擎作为基础的条件下获得良好的结果，即使这些LLM引擎的能力存在差距。总体而言，功能性验证证明了LayoutCopilot的设计显著提升了输出质量，证明了其实际应用价值。

#### IV-A3 分类准确度评估

为了评估分类器代理的准确性，我们使用了GPT-3.5和Claude-3进行了实验。测试集由1000个具体请求和1000个抽象请求组成，生成方法详见[IV-A1](https://arxiv.org/html/2406.18873v2#S4.SS1.SSS1 "IV-A1 Sanity Check ‣ IV-A Concrete Requests: Comprehensive Evaluation ‣ IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")节，以确保实验的一致性。如[表III](https://arxiv.org/html/2406.18873v2#S4.T3 "TABLE III ‣ IV-A3 Classification Accuracy Evaluation ‣ IV-A Concrete Requests: Comprehensive Evaluation ‣ IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")所示，分类器代理在两种模型中均达到了高准确度。使用GPT-3.5时，代理正确识别了1000个具体请求中的976个和1000个抽象请求中的981个。使用Claude-3时，具体请求和抽象请求的分类准确率均达到了100%。如此高的准确性可能归因于任务的相对简单性、分类标准的清晰性，以及强大的预训练LLM模型。这些结果验证了分类器代理在准确区分具体和抽象请求方面的有效性和可靠性，为其在LayoutCopilot框架中的作用提供了有力支持。

表III：分类器代理的分类准确率

基于GPT-3.5和Claude-3

| 请求类型 | 准确率（%） |
| --- | --- |
| GPT-3.5 | Claude-3 |
| 具体请求 | 97.6 | 100 |
| 抽象请求 | 98.1 | 100 |

### IV-B 抽象请求：案例研究

在本小节中，我们通过优化实际设计中的布局，展示了LayoutCopilot在处理抽象请求方面的能力。这两个电路的原理图如图[9](https://arxiv.org/html/2406.18873v2#S4.F9 "Figure 9 ‣ IV-B1 Operational Transconductance Amplifier (OTA) ‣ IV-B Abstract Requests: Case Studies ‣ IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")和图[10](https://arxiv.org/html/2406.18873v2#S4.F10 "Figure 10 ‣ IV-B1 Operational Transconductance Amplifier (OTA) ‣ IV-B Abstract Requests: Case Studies ‣ IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")所示，它们的布局调整过程分别如图[8](https://arxiv.org/html/2406.18873v2#S4.F8 "Figure 8 ‣ IV-B Abstract Requests: Case Studies ‣ IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design") (a)和(b)所示。

![参见说明](img/a3fca7b84c1ee3e41e8c4f79edbd84e7.png)

图8：基于交互式LLM的OTA和COMP布局调整

#### IV-B1 操作跨导放大器（OTA）

图[8](https://arxiv.org/html/2406.18873v2#S4.F8 "Figure 8 ‣ IV-B Abstract Requests: Case Studies ‣ IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")（a）展示了OTA的布局优化过程，其原理图如图[9](https://arxiv.org/html/2406.18873v2#S4.F9 "Figure 9 ‣ IV-B1 Operational Transconductance Amplifier (OTA) ‣ IV-B Abstract Requests: Case Studies ‣ IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")所示。我们采用MAGICAL [[18](https://arxiv.org/html/2406.18873v2#bib.bib18), [20](https://arxiv.org/html/2406.18873v2#bib.bib20), [19](https://arxiv.org/html/2406.18873v2#bib.bib19)] 生成初始布局①，未应用任何约束。初始布局与我们的框架无关，我们的框架仅关注修改，而非起源。从布局①开始，我们使用LayoutCopilot在布局阶段提出改进建议，模拟设计师所面临的实际场景。LayoutCopilot提出了一系列高层次的解决方案，简要列举如下：

<svg class="ltx_picture" height="79.37" id="S4.SS2.SSS1.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,79.37) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 7.87 48.25)"><foreignobject color="#000000" height="25.21" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="584.25">设计师：我得到一个OTA布局，但性能很差。我该如何改进它？</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 7.87 13.78)"><foreignobject color="#000000" height="26.6" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="584.25">LayoutCopilot：提高OTA布局性能可以包括……以下是一些潜在的高层次解决方案：1. 使用symAdd增强对称性…… 2. 使用deviceMove和deviceSwap改善匹配…… 3. 使用wireWidth和wireSpacing减少寄生效应…… 4. 使用wireSpacing防止串扰…… 5. 使用netPriority和netTopology改善布线…… 6. …</foreignobject></g>

随后，我们选择了“增强对称性”选项，并向LayoutCopilot提供了网表，要求其给出具体的实施步骤。LayoutCopilot分析网表并识别出应施加对称性约束的关键电路组件：

<svg class="ltx_picture" height="155.47" id="S4.SS2.SSS1.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,155.47) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 7.87 91.15)"><foreignobject color="#000000" height="58.42" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="584.25">设计师：好的，我更倾向于从增强对称性优化开始，因为当前的对称性看起来很糟糕。这里是电路网表：[网表]。

我应该在哪些设备上添加对称性？请帮我提供相应的命令。</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 7.87 13.78)"><foreignobject color="#000000" height="69.49" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="584.25">LayoutCopilot：我们开始分析网表以识别关键设备…… *对称性设备：* 1. 差分输入对：M34（net0130 VIM PTAIL VDD）和M35（net0132 VIP PTAIL VDD）处理输入信号…… 2. 输出级：M71（VOP VO1M VDD VDD）和M70（VOM VO1P VDD VDD）用于输出级…… 3. 尾电流源：M1（NTAIL NBIAS_TAIL GND GND）作为尾电流源…… 4. … *命令序列：*

```
symAdd M34 M35  # Differential input stage.
symAdd M71 M70  # Output stage.
symAdd M1 M1    # Self-symmetry for tail.
...

```

执行这些命令后，结果布局如②所示。观察布局②时，我们注意到大型电容器没有被对称约束，导致布局松散且杂乱。在收到我们更具体的请求后，LayoutCopilot根据尺寸结果和连接性，对所有电容器应用了对称约束，最终得到了布局③。为了简化，后续的回复不再具体显示。

此外，我们指示LayoutCopilot对所有可能的组件应用对称性，包括先前未考虑的电阻。在识别出网表中合适的组件后，LayoutCopilot生成并执行相应的命令，得到布局④。随着布局视觉效果的显著改善，我们继续执行基本的布线命令以生成布局⑤。随后的后布局仿真揭示了过多的寄生效应，导致我们寻求LayoutCopilot的进一步解决方案。为了解决这些问题，我们对关键网络进行了详细检查，并实施了优先级布线策略，具体如下：

<svg class="ltx_picture" height="180.8" id="S4.SS2.SSS1.p7.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,180.8) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 7.87 116.47)"><foreignobject color="#000000" height="58.42" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="584.25">设计师：完成对称性调整后，我们也需要对布线进行一些改进，我希望优先处理关键网络，以减少其布线路径和寄生效应。

请识别它们并帮助我执行相应的命令。</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 7.87 13.78)"><foreignobject color="#000000" height="94.82" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="584.25">LayoutCopilot：让我们继续分析网表，以识别关键网路…… *关键网路：* 1. 输入差分对连接：M34的栅极和漏极（net0130 VIM PTAIL VDD）及M35的栅极和漏极（net0132 VIP PTAIL VDD）…… 2. 输出差分对连接：M71的栅极和漏极（VOP VO1M VDD VDD）及M70的栅极和漏极（VOM VO1P VDD VDD）…… 3. 输入级补偿电容连接：C3（VIM net096 GND）和C2（VIP net092 GND）之间的连接…… 4. 负载补偿电容连接：…… *命令序列：*

```
netPriority net0130 10  # Drain of M34.
netPriority VIM 10      # Gate of M34.
netPriority net0132 10  # Drain of M35.
netPriority VIP 10      # Gate of M35.
...
netPriority net096 8    # Connection of C3.
netPriority net092 8    # Connection of C2.
...

```

在按照上述命令修改路由优先级后，我们还增加了电源和接地线的宽度（由于空间限制，这里不做详细说明），最终达到了布局⑥所示的结果。由于后布局结果已经符合我们的规格，如表[V](https://arxiv.org/html/2406.18873v2#S4.T5 "TABLE V ‣ IV-B1 Operational Transconductance Amplifier (OTA) ‣ IV-B Abstract Requests: Case Studies ‣ IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")所示，我们终止了迭代优化过程。

![参见说明](img/381f24fca5ffce1fb2a0fc1bedcbbe74.png)

图9：OTA的电路原理图。

![参见说明](img/a470b6d5c2abecbe1205b3a10f3f8680.png)

图10：COMP的电路原理图。

我们在TSMC 40nm工艺下进行优化过程，以验证修改后的电路性能，利用Cadence Virtuoso和Mentor Graphics Calibre进行后布局仿真。在与LayoutCopilot进行交互式布局调整的过程中，布局面积在放置阶段最初有所增加，但最终缩小至原始大小的66%，如表[IV](https://arxiv.org/html/2406.18873v2#S4.T4 "TABLE IV ‣ IV-B1 Operational Transconductance Amplifier (OTA) ‣ IV-B Abstract Requests: Case Studies ‣ IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")和图[8](https://arxiv.org/html/2406.18873v2#S4.F8 "Figure 8 ‣ IV-B Abstract Requests: Case Studies ‣ IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")所示。鉴于模拟电路中放置与布线之间及其后布局性能之间的复杂关系，改进的进程并不按照从①到⑤的线性规律进行，如前文所述布局面积的变化所示。为了简化起见，我们比较了初始布局（无约束）和通过与LayoutCopilot交互调整后的最终布局之间的后布局性能结果，总结见表[Ⅴ](https://arxiv.org/html/2406.18873v2#S4.T5 "TABLE V ‣ IV-B1 Operational Transconductance Amplifier (OTA) ‣ IV-B Abstract Requests: Case Studies ‣ IV Experimental Results ‣ LayoutCopilot: An LLM-powered Multi-agent Collaborative Framework for Interactive Analog Layout Design")。

在初始布局中，由于寄生效应过大，增益为负值，单位增益带宽（UGB）和相位裕度（PM）明显较差，且共模抑制比（CMRR）显著较低，显示出与不考虑寄生提取的前布局原理图结果之间存在较大差异。经过使用LayoutCopilot进行调整后，虽然与原理图结果相比，UGB仍存在差异，但增益和PM已经接近原理图水平，并且CMRR相较于初始布局有了显著改善。这些结果突出了LayoutCopilot通过自然语言交互促进布局优化的有效性，并减少了设计师的学习和编码时间，展示了它作为模拟布局设计师强大助手的潜力。

表IV：布局面积统计。

| 基准 | 技术 | 布局 | 面积 |
| --- | --- | --- | --- |
| 芯片节点 | 面积 | 比率 |
| OTA-1 | TSMC40 | $83.4\times 124.2\mu m^{2}$ | 1.00 |
| OTA-2 | $85.0\times 148.9\mu m^{2}$ | 1.22 |
| OTA-3 | $80.6\times 94.3\mu m^{2}$ | 0.73 |
| OTA-4 | $85.4\times 80.4\mu m^{2}$ | 0.66 |
| COMP-1 | TSMC28 | $38.3\times 39.7\mu m^{2}$ | 1.00 |
| COMP-2 | $24.0\times 46.9\mu m^{2}$ | 0.74 |

表 V：LayoutCopilot 与 MAGICAL [[18](https://arxiv.org/html/2406.18873v2#bib.bib18), [20](https://arxiv.org/html/2406.18873v2#bib.bib20), [19](https://arxiv.org/html/2406.18873v2#bib.bib19)] 在没有约束条件下的比较。

| 基准 | 原理图 | MAGICAL [[18](https://arxiv.org/html/2406.18873v2#bib.bib18), [20](https://arxiv.org/html/2406.18873v2#bib.bib20), [19](https://arxiv.org/html/2406.18873v2#bib.bib19)] | LayoutCopilot |
| --- | --- | --- | --- |
| 无约束条件 |
| OTA | 增益（dB） | 38.63 | -8.75 | 38.26 |
| UGB（MHz） | 6.85 | – | 4.42 |
| CMRR（dB） | – | 27.3 | 58.7 |
| PM（度） | 70.98 | – | 76.28 |
| COMP | CMP_延迟（ns） | 3.3 | 6.3 | 6.7 |
| 噪声（uV） | 50.3 | 30.9 | 13.9 |
| RST_延迟（ps） | 89.8 | 165.8 | 538.7 |
| 功率（uW） | 19.9 | 32.0 | 31.71 |

#### IV-B2 比较器（COMP）

类似的过程用于 COMP 调整，现采用图 [8](https://arxiv.org/html/2406.18873v2#S4.F8 "图 8 ‣ IV-B 摘要请求：案例研究 ‣ IV 实验结果 ‣ LayoutCopilot: 一个基于LLM的多代理协作框架用于互动模拟布局设计")（b）中所示的TSMC 28nm 技术。COMP 的原理图见图 [10](https://arxiv.org/html/2406.18873v2#S4.F10 "图 10 ‣ IV-B1 操作跨导放大器（OTA） ‣ IV-B 摘要请求：案例研究 ‣ IV 实验结果 ‣ LayoutCopilot: 一个基于LLM的多代理协作框架用于互动模拟布局设计")。我们通过与 LayoutCopilot 的交互优化了晶体管和电容器的对称性，将初始布局①转化为改进后的布局②。如表 [V](https://arxiv.org/html/2406.18873v2#S4.T5 "表 V ‣ IV-B1 操作跨导放大器（OTA） ‣ IV-B 摘要请求：案例研究 ‣ IV 实验结果 ‣ LayoutCopilot: 一个基于LLM的多代理协作框架用于互动模拟布局设计") 下半部分总结所示，后期布局仿真结果显示，我们在延迟上有所牺牲，但相较于初始布局，噪声性能显著提高，功率效率得到了增强。OTA 和 CMP 的调整过程展示了 LayoutCopilot 如何利用其全面的知识库提供可操作的建议，切实优化了具有不同拓扑结构的电路布局，并改善了它们的仿真后性能。

## V 结论

在本研究中，我们提出了一种由大语言模型（LLM）驱动的多智能体协作框架，用于交互式模拟布局设计。LayoutCopilot不仅可以将自然语言指令转换为可执行的脚本命令，还能够将高级设计意图转化为可执行的实施建议。从技术上讲，LayoutCopilot采用了多智能体方法学并结合了提示工程。在TSMC28和TSMC40上进行验证的实验结果展示了LayoutCopilot的稳健性和优势。具体而言，LayoutCopilot在处理具体布局调整请求时达到了较高的准确性，并在应对抽象布局优化请求时有效地改善了布局性能。我们相信，本研究可以为解决交互式EDA工具的可用性问题提供新的见解，并促进模拟电路自动化工具的发展。

## 参考文献

+   [1] Cadence设计系统，*Virtuoso布局套件*，Cadence设计系统公司，美国加利福尼亚州圣荷西，2023年。

+   [2] K. Lampaert *等人*，“一种基于性能驱动的模拟集成电路布局工具”，*IEEE固态电路学报*，第30卷，第773-780页，1995年。

+   [3] K. Lampaert *等人*，“适用于可制造性的模拟布线”，载于*定制集成电路会议论文集*，1996年，第175-178页。

+   [4] E. Malavasi *等人*，“一种用于模拟集成电路的布线方法学”。载于*ICCAD*。Citeseer，1990年，第202-205页。

+   [5] L. Xiao *等人*，“模拟电路设计的实用布局与布线技术”，载于*2010年IEEE/ACM国际计算机辅助设计会议（ICCAD）*。IEEE，2010年，第675-679页。

+   [6] B. Xu *等人*，“高性能模拟电路的分层分析布局技术”，载于*2017年ACM国际物理设计研讨会论文集*，2017年，第55-62页。

+   [7] B. Basaran *等人*，“考虑锁存现象的定制模拟单元的布局与寄生约束布线”，载于*1993年国际计算机辅助设计会议论文集（ICCAD）*。IEEE，1993年，第415-421页。

+   [8] H.-C. Ou *等人*，“考虑电流流动和电流密度的模拟布局与布线同时进行”，载于*第50届年会设计自动化会议论文集*，2013年，第1-6页。

+   [9] R. Martins *等人*，“考虑电流流动和电流密度的模拟集成电路布局的多目标优化”，*Integration*，第55卷，第295-306页，2016年。

+   [10] B. Xu *等人*，“考虑器件层的模拟电路分析布局”，载于*2019年国际物理设计研讨会论文集*，2019年，第19-26页。

+   [11] K.-H. Ho *等人*，“考虑耦合效应的模拟集成电路电容阵列长度比匹配布线”，载于*第50届年会设计自动化会议论文集*，2013年，第1-6页。

+   [12] Y. Li *等*，“探索一种基于机器学习的性能驱动模拟IC布局方法”，发表于 *2020 IEEE计算机学会VLSI年会（ISVLSI）*。IEEE，2020年，页24–29。

+   [13] Y. Li *等*，“一种定制的图神经网络模型用于引导模拟IC布局”，发表于 *第39届计算机辅助设计国际会议论文集*，2020年，页1–9。

+   [14] A. Gusmão *等*，“半监督人工神经网络在模拟IC布局推荐中的应用”，发表于 *2020 IEEE 国际电路与系统研讨会（ISCAS）*。IEEE，2020年，页1–5。

+   [15] K. Kunal *等*，“ALIGN：从零开始的开源模拟布局自动化”，发表于 *第56届年度设计自动化大会论文集*。美国拉斯维加斯：ACM，2019年6月，页1–4。

+   [16] T. Dhar *等*，“ALIGN：一种自动化模拟布局的系统”，*IEEE 设计与测试*，第38卷，页8–18，2021年4月。

+   [17] S. S. Sapatnekar，“ALIGN 自动化模拟布局引擎：进展、经验与开放问题”，发表于 *2023 国际物理设计研讨会论文集*。美国虚拟活动：ACM，2023年3月，页101–102。

+   [18] B. Xu *等*，“MAGICAL：利用人类和机器智能实现全自动模拟IC布局的探索：邀请论文”，发表于 *2019 IEEE/ACM 计算机辅助设计国际会议（ICCAD）*。美国科罗拉多州威斯敏斯特：IEEE，2019年11月，页1–8。

+   [19] H. Chen *等*，“MAGICAL：一个开源的全自动模拟IC布局系统，从网表到GDSII”，*IEEE 设计与测试*，第38卷，页19–26，2021年。

+   [20] H. Chen *等*，“MAGICAL 1.0：一个开源的全自动AMS布局综合框架，通过40纳米1GS/s $\Sigma\Delta$ ADC验证”，发表于 *2021 IEEE定制集成电路大会（CICC）*。美国德州奥斯汀：IEEE，2021年4月，页1–2。

+   [21] X. Gao *等*，“具有即时布局合法化的交互式模拟布局编辑”，发表于 *2021年第58届ACM/IEEE设计自动化大会（DAC）*，2021年，页1249–1254。

+   [22] X. Gao *等*，“具有即时布局和布线合法化的交互式模拟布局编辑”，*IEEE 集成电路与系统计算机辅助设计学报*，第42卷，页698–711，2023年3月。

+   [23] T. B. Brown *等*，“语言模型是少样本学习者”，发表于 *第34届神经信息处理系统国际会议论文集*，系列NIPS ’20。美国纽约红钩：Curran Associates Inc.，2020年。

+   [24] H. Touvron *等*，“Llama：开放高效的基础语言模型”，[https://arxiv.org/abs/2302.13971](https://arxiv.org/abs/2302.13971)，2023年。

+   [25] H. Touvron *等*，“Llama 2：开放基础和微调聊天模型”，[https://arxiv.org/abs/2307.09288](https://arxiv.org/abs/2307.09288)，2023年。

+   [26] OpenAI *等*，“GPT-4技术报告”，2024年3月，arXiv:2303.08774 [cs]。

+   [27] Anthropic，“Claude 2”，[https://www.anthropic.com/news/claude-2](https://www.anthropic.com/news/claude-2)，2023年，访问日期：2023年4月10日。

+   [28] J. Lienig 和 G. Jerke, “集成电路的电迁移感知物理设计，” *VLSI设计会议论文集*，2005年，第77–82页。

+   [29] P. Lewis *等人*, “用于知识密集型NLP任务的检索增强生成。”

+   [30] Q. Wu *等人*, “AutoGen：通过多代理对话推动下一代 LLM 应用，”2023年10月，arXiv:2308.08155 [cs]。

+   [31] S. Rasal, “LLM Harmony：为问题解决提供多代理通信，”2024年1月，arXiv:2401.01312 [cs]。

+   [32] S. Hong *等人*, “MetaGPT：多代理协作框架的元编程，”2023年11月，arXiv:2308.00352 [cs]。

+   [33] S. Liu *等人*, “RTLCoder：凭借我们的开源数据集和轻量级解决方案，超越 GPT-3.5 在设计 RTL 生成中的表现，”2024年2月，arXiv:2312.08617 [cs]。

+   [34] S. Thakur *等人*, “VeriGen：用于 Verilog 代码生成的大型语言模型，” *ACM 电子系统设计自动化期刊*，2024年2月，刚刚接受。

+   [35] Y. Lu *等人*, “RTLLM：一个用于设计 RTL 生成的大型语言模型开源基准，”2023年11月，arXiv:2308.05345 [cs]。

+   [36] M. Liu *等人*, “邀请论文：VerilogEval：评估大型语言模型在 Verilog 代码生成中的表现，” *2023 IEEE/ACM国际计算机辅助设计大会（ICCAD）*。 美国加利福尼亚州旧金山：IEEE，2023年10月，第1–8页。

+   [37] Y.-D. Tsai *等人*, “RTLFixer：使用大型语言模型自动修复 RTL 语法错误，”2024年2月，arXiv:2311.16543 [cs]。

+   [38] M. Liu *等人*, “ChipNeMo：面向芯片设计的领域自适应大型语言模型，”2024年4月，arXiv:2311.00176 [cs]。

+   [39] Z. He *等人*, “ChatEDA：一个由大型语言模型驱动的自动化EDA代理，”2024年3月，arXiv:2308.10204 [cs]。

+   [40] Z. Wang *等人*, “Chatpattern：通过自然语言进行布局模式定制，” [https://arxiv.org/abs/2403.15434](https://arxiv.org/abs/2403.15434)，2024年。

+   [41] P. Sahoo *等人*, “大型语言模型中提示工程的系统性调查：技术与应用，” *arXiv预印本 arXiv:2402.07927*，2024年。

+   [42] Q. Dong *等人*, “关于上下文学习的调查，” *arXiv预印本 arXiv:2301.00234*，2022年。

+   [43] T. Brown *等人*, “语言模型是少量样本学习者，” *神经信息处理系统进展*，第33卷，第1877–1901页，2020年。

+   [44] J. Wei *等人*, “思维链提示引发大型语言模型的推理，” *神经信息处理系统进展*，第35卷，第24,824–24,837页，2022年。

+   [45] D. Zhou *等人*, “最小到最大提示使大型语言模型能够进行复杂推理，” *arXiv预印本 arXiv:2205.10625*，2022年。

+   [46] A. Madaan *等人*, “Self-refine：通过自我反馈进行迭代优化，” *神经信息处理系统进展*，第36卷，2024年。

+   [47] P. Lewis *等人*, “用于知识密集型NLP任务的检索增强生成，” *神经信息处理系统进展*，第33卷，第9459–9474页，2020年。

+   [48] C. Zhang *等人*，“Proagent：构建基于大型语言模型的主动协作智能体，”发表于 *人工智能学会年会论文集*，第38卷，第16期，2024年，第17,591–17,599页。

+   [49] Z. Xi *等人*，“基于大型语言模型的智能体的崛起与潜力：一项调查，” *arXiv预印本arXiv:2309.07864*，2023年。

+   [50] G. Li *等人*，“Camel：用于大型语言模型社会“心智”探索的交互智能体，”2023年。

+   [51] Y. Talebirad和A. Nadiri，“多智能体协作：利用智能LLM智能体的力量，” *arXiv预印本arXiv:2306.03314*，2023年。

+   [52] S. Rasal，“Llm和谐：多智能体通信用于问题解决，” *arXiv预印本arXiv:2401.01312*，2024年。

+   [53] Q. Wu *等人*，“Autogen：通过多智能体对话框架推动下一代LLM应用，” *arXiv预印本arXiv:2308.08155*，2023年。

+   [54] W. Chen *等人*，“Agentverse：促进多智能体协作并探索智能体中的涌现行为，” *arXiv预印本arXiv:2308.10848*，2023年。

+   [55] S. Hong *等人*，“Metagpt：用于多智能体协作框架的元编程，” *arXiv预印本arXiv:2308.00352*，2023年。

+   [56] B. Razavi，*模拟CMOS集成电路设计*。纽约：McGraw-Hill，2001年。

+   [57] K. Lampaert *等人*，*性能和可制造性的模拟布局生成*，系列：Springer国际工程与计算机科学系列，第501卷。 Springer，1999年。

+   [58] Anthropic，“推出下一代Claude，” [https://www.anthropic.com/news/claude-3-family](https://www.anthropic.com/news/claude-3-family)，2024年，访问日期：2023-04-10。

| ![[无标题图片]](img/3747f6cd4dd76489b227a45f64a5437f.png) | Bingyang Liu目前是北京大学电子工程与计算机科学学院的本科生。他的研究兴趣包括模拟设计自动化和EDA中的机器学习。 |
| --- | --- |
| ![[无标题图片]](img/db7ee3af32c14cfea2ff109ab8ece8ab.png) | Haoyi Zhang于2022年获得北京航空航天大学微电子学专业的学士学位。目前，他正在北京大学攻读微电子学博士学位。其研究兴趣包括模拟设计自动化、混合信号电路设计。 |
| ![[无标题图片]](img/614cf3b5a2d008527d216efaccb7e87b.png) | Xiaohan Gao于2020年获得北京大学计算机科学专业的学士学位。她目前是北京大学计算机科学学院的博士生，导师为林一博教授。她的研究兴趣包括模拟和混合信号电路的布局设计自动化，将机器学习技术与物理设计和设计可制造性结合，以及从布局设计中抽象描述。 |
| ![[未命名图像]](img/e3c76a16e628a7e6bd35a13ac9da5014.png) | 孔子辰于2023年获得北京大学电子工程与计算机科学学院的学士学位。目前，他正在北京大学集成电路学院攻读博士学位。他的研究兴趣包括CMOS图像传感器和模拟设计自动化。 |
| ![[未命名图像]](img/fafff409b4d223d5fd52855611fd636b.png) | 唐希远（S’17 -M’19-SM’24）于2012年获得上海交通大学微电子学（荣誉）学士学位，2014年和2019年分别获得德克萨斯大学奥斯汀分校电气工程硕士和博士学位。目前，他是北京大学助理教授。2014至2017年，他曾在Silicon Laboratories公司担任设计工程师，参与了射频接收器设计。2019至2021年，他在德克萨斯大学奥斯汀分校担任博士后研究员。他的研究兴趣包括数字辅助数据转换器、低功耗混合信号电路和模拟数据处理。唐博士是ISSCC技术程序委员会（TPC）成员，并担任IEEE固态电路通讯副主编。他曾获得2020年IEEE固态电路学会新星奖、2016年Silicon Labs技术研讨会最佳论文奖、2011年国家奖学金和2010年上海奖学金。 |
| ![[未命名图像]](img/f95ae846304bf7f4ee1bae193370f915.png) | 林一博（M’19）于2013年获得上海交通大学微电子学士学位。2018年，他在德克萨斯大学奥斯汀分校获得电气与计算机工程博士学位，导师为David Z. Pan教授。他于2018至2019年在德克萨斯大学奥斯汀分校担任博士后研究员。目前，他是北京大学集成电路学院的助理教授。他的研究兴趣包括物理设计、机器学习应用以及VLSI CAD中的异构计算。他曾获得TCAD、DAC、DATE、ISPD等顶级EDA/CAD期刊/会议的最佳论文奖。 |
| ![[无标题图片]](img/ee0870e1b529a06f06b75a04ce61ff7e.png) | 王润生（S’07–M’11）于2005年和2010年分别获得中国北京大学的学士学位和博士学位（最高荣誉）。2008年11月至2009年8月，他曾作为访问学者在美国印第安纳州西拉法叶的普渡大学工作。2010年，他加入北京大学，目前是集成电路学院的教授，并担任电子信息与电气工程学院副院长。他是一本书、四章书籍章节以及约200篇科学论文的作者或合著者，其中包括在国际电子器件会议（IEDM）和VLSI技术研讨会（VLSI-T）上发表的40多篇论文。他已获得19项美国专利和38项中国专利。他目前的研究兴趣包括纳米尺度CMOS器件及其可靠性、设计自动化以及新范式计算。王博士曾获得IEEE电子器件学会（EDS）早期职业奖、国家自然科学基金委的国家优秀青年科学基金（NSFC）、中国教育部自然科学奖（一等奖）等多项奖励。他还担任《IEEE电子器件学报》和《科学中国：信息科学》的编辑委员会成员，并曾在多个IEEE会议的技术程序委员会中担任职务，包括IEDM、IRPS等。 |

| ![[无标题图片]](img/97e7301aa2ebd3d649afc7431180fe47.png) | 黄如（IEEE会士）于1991年和1994年分别获得中国南京的东南大学电子工程学士（荣誉）和硕士学位，并于1997年获得中国北京的北京大学微电子学博士学位。自1997年起，她便成为北京大学的教职工，目前担任博雅讲席教授。她是中国科学院院士和世界科学院院士（TWAS Fellow）。她是五本书籍、五章书籍章节以及300多篇论文的作者或合著者，其中包括100多篇在IEDM（2007至2021年共46篇IEDM论文）、VLSI技术研讨会、IEEE EDL和IEEE T-ED上发表的论文。她曾在50多次国际会议和研讨会上做过主题演讲或特邀报告。她已获得300多项专利，其中包括49项美国专利。她的研究兴趣包括纳米尺度的CMOS器件、超低功耗新器件、神经形态计算新器件、新兴存储技术及器件的变异性/可靠性。 |  
