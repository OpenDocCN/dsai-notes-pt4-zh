<!--yml

类别：未分类

日期：2025-01-11 11:57:01

-->

# BudgetMLAgent: 一种成本效益高的LLM多代理系统，用于自动化机器学习任务

> 来源：[https://arxiv.org/html/2411.07464/](https://arxiv.org/html/2411.07464/)

\useunder

\ul

Shubham Gandhi TCS ResearchPuneIndia [gandhi.shubham@tcs.com](mailto:gandhi.shubham@tcs.com) ,  Manasi Patwardhan TCS ResearchPuneIndia [manasi.patwardhan@tcs.com](mailto:manasi.patwardhan@tcs.com) ,  Lovekesh Vig TCS ResearchDelhiIndia [lovekesh.vig@tcs.com](mailto:lovekesh.vig@tcs.com)  和  Gautam Shroff TCS ResearchDelhiIndia [gautam.shroff@tcs.com](mailto:gautam.shroff@tcs.com)(2024; 2024)

###### 摘要。

大型语言模型（LLMs）在多种应用中表现出色，包括生成代码片段，但在生成复杂的机器学习（ML）任务代码时常常表现不佳。尽管现有的基于单一代理的LLM系统根据任务的复杂性表现各异，但它们完全依赖于像GPT-4这样的更大且更昂贵的模型。我们的研究表明，像Gemini-Pro、Mixtral和CodeLlama这样的低成本和无成本模型在单一代理设置下的表现远远不如GPT-4。为了开发一种具有成本效益的LLM解决方案来解决ML任务，我们提出了一种基于LLM多代理的系统，该系统利用专家组合、过去观测的高效检索、LLM级联和专家咨询。通过在MLAgentBench基准上对机器学习工程任务的实证分析，我们展示了该系统的有效性，使用无成本模型，即以Gemini作为基础LLM，并与GPT-4进行级联，专家则用于偶尔的计划咨询。通过将成本降低94.2%（从GPT-4单一代理系统的每次运行成本$0.931降至$0.054），我们的系统能够提供更高的平均成功率（32.95%），相比之下，GPT-4单一代理系统在所有MLAgentBench任务中的平均成功率为22.72%。

自动化ML，大型语言模型，LLM代理^†^†版权所有：acmcopyright^†^†期刊年份：2024^†^†doi：3703412.3703416^†^†isbn：979-8-4007-1161-9^†^†期刊年份：2024^†^†版权所有：rightsretained^†^†会议：第四届国际人工智能-机器学习系统会议；2024年10月8日至11日；美国路易斯安那州巴吞鲁日^†^†书名：第四届国际人工智能-机器学习系统会议（AIMLSystems 2024），2024年10月8日至11日，巴吞鲁日，路易斯安那州，美国^†^†doi：10.1145/3703412.3703416^†^†isbn：979-8-4007-1161-9/24/10^†^†ccs：计算方法学 多代理系统^†^†ccs：计算方法学 智能代理^†^†ccs：计算方法学 多代理规划

## 1\. 引言

尽管近期的进展表明，大型语言模型（LLMs）擅长处理从自然语言（Fang et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib9); Huang and Chang, [2023](https://arxiv.org/html/2411.07464v2#bib.bib16); Zhu et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib51); Yi et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib41)) 到与代码相关的任务（Zheng et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib49); Zan et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib43); Zhang et al., [2024a](https://arxiv.org/html/2411.07464v2#bib.bib48)），这一能力并不总是能够转化为更复杂和更细致的任务（Yeadon et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib40)）。大多数涉及LLM的与代码相关的工作（Guo et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib11); Huang et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib15); Zhong et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib50)）基于诸如HumanEval（Chen et al., [2021](https://arxiv.org/html/2411.07464v2#bib.bib7)）和MBXP（Athiwaratkun et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib4)）这样的任务，这些任务的复杂性相对较低，远不及数据科学家所面临的复杂程度。然而，现实世界的工程挑战需要细致的解决问题能力和复杂的规划，通常需要经过多轮策略制定、实验和再调整。LLM代理系统在模拟这一迭代过程中表现优异，因为它们由一个包含代码文件、描述文件和数据文件的环境构成，并且拥有一个预定义的行动空间，允许与环境进行互动。这展示了它们有效解决复杂工程挑战的能力。（Zhang et al., [2024b](https://arxiv.org/html/2411.07464v2#bib.bib46)）。

转向对机器学习（ML）应用的编码带来了自身的挑战，因为这些应用通常涉及在数据集上训练模型、调整超参数、设计提高性能的方法等。这些应用并不简单，需要深入理解底层算法和技术，以及用于实现计划的特定库。虽然已经有基于AutoML的方法来自动化这些任务（He et al., [2021](https://arxiv.org/html/2411.07464v2#bib.bib12); Salehin et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib29)），但这些方法的灵活性有限，因为它们通常在预定义的约束和搜索空间内操作，以可能的架构和/或超参数配置的形式进行，这可能限制它们探索分布外的解决方案的能力。尽管像ChatDev（Qian et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib27)）和MetaGPT（Hong et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib13)）等研究探索了LLM代理在软件开发环境中的能力，但在利用LLM代理解决ML任务方面的研究仍然明显匮乏。

最近的研究，如MLCopilot（Zhang et al., [2024c](https://arxiv.org/html/2411.07464v2#bib.bib47)），引入了一种用于解决ML任务的助手。然而，这些架构在它们能够解决的问题类型上存在限制，且必须严格遵循与现实场景不符的任务描述格式。此外，这些助手仅提供解决方案建议，实际的实施负担仍由用户承担。据我们所知，MLAgentBench（Huang et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib17)）是唯一一个针对LLM代理直接处理代码并解决ML问题的显著基准。

尽管它们在基准测试中的某些任务上表现良好，但它们专注于使用昂贵的大型语言模型（LLM），如GPT-4的单一代理系统，GPT-4每次运行的费用大约为$0.52-$2.9，具体取决于任务。对于他们进行的实验，每个任务进行8次实验，总计超过15个任务，从而导致非常高的实验成本，大约为$200以上。随着更大规模的模型使用成本越来越高，开发无成本或低成本系统，利用较小的开源模型并使其在特定任务中同样具备能力，成为了一种自然的动力。然而，现有的代理创建框架，如AutoGen（Wu等，[2023](https://arxiv.org/html/2411.07464v2#bib.bib39)），并没有优先考虑成本降低。用较小的开源LLM替代使用昂贵LLM的单一代理系统可能无法达到预期目的。我们最初的实验是将黄等人（[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)）在自动生成机器学习任务代码中所有的LLM调用替换为无成本或低成本的LLM，即Gemini-Pro（Team等，[2023](https://arxiv.org/html/2411.07464v2#bib.bib32)）¹¹1gemini-pro 1.0 API，来自[https://ai.google.dev/tutorials/python_quickstart](https://ai.google.dev/tutorials/python_quickstart)。免费或无成本版本的速率限制足以进行我们的实验，但我们还包括了按需计费的无成本版本的费用，CodeLlama（Rozière等，[2024](https://arxiv.org/html/2411.07464v2#bib.bib28)）²²2[https://huggingface.co/codellama/CodeLlama-34b-Instruct-hf](https://huggingface.co/codellama/CodeLlama-34b-Instruct-hf) 和Mixtral（Jiang等，[2024](https://arxiv.org/html/2411.07464v2#bib.bib19)）³³3Mixtral-8x7B-v0.1 [https://huggingface.co/mistralai/Mixtral-8x7B-v0.1](https://huggingface.co/mistralai/Mixtral-8x7B-v0.1)，在单一代理设置下对所有任务的结果非常差。

在现实世界中，任何复杂的任务很少由单一个体独立完成，尤其是当所有个体都不具备执行任务所需的专业知识时。相反，工程师团队会合作，每个成员扮演独特的角色（persona），贡献各自的专业知识和技能，通过集体努力实现目标。过去关于大语言模型（LLM）智能体的研究，通过设计多智能体框架（Li等人，[2024](https://arxiv.org/html/2411.07464v2#bib.bib22)；Shen等人，[2024](https://arxiv.org/html/2411.07464v2#bib.bib30)），结合LLM专家（Wang等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib35)；Ding等人，[2024](https://arxiv.org/html/2411.07464v2#bib.bib8)），并定义任务级联（Chen等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib6)；Yue等人，[2024](https://arxiv.org/html/2411.07464v2#bib.bib42)；Zhang等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib45)），用于任务如代码生成、推理、问答等。任务级联是指以渐进的方式串联多个LLM，其中先调用较弱的LLM，如果响应不满意，则调用更强大的LLM。然而，据我们所知，尚未有多智能体框架，采用开源LLM作为智能体来处理机器学习任务的工程。

本文通过提出一个系统来弥补使用LLM解决机器学习任务的空白，该系统利用- （i）多LLM智能体作为专家组合，使用档案化（profiling），（ii）LLM级联，（iii）高效检索相关过去观察结果，以及（iv）我们创新性的偶尔“请教专家”功能，调用GPT-4 ⁴⁴4gpt-4-0125-preview [https://platform.openai.com/docs/models/gpt-4-and-gpt-4-turbo](https://platform.openai.com/docs/models/gpt-4-and-gpt-4-turbo)进行规划。我们的方法旨在弥合较低成本LLM的能力和复杂机器学习任务需求之间的差距，提供一个更加成本高效和可扩展的解决方案。通过实证分析，我们验证了以下主张：

+   •

    我们表现最好的多智能体系统，使用无成本或低成本版本的Gemini-Pro作为基础大语言模型（LLM），能够以极低的成本执行任务（在MLAgentBench数据集中，每次运行每个机器学习任务的平均成本分别为无成本版$0.054和低成本版$0.120），而与之对比的基准单智能体GPT4系统的成本则为每次任务$0.931（见Huang等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)）。

+   •

    我们表现最好的多智能体系统，在无成本和低成本的Gemini-Pro版本下，分别实现了94.2%和87.1%的成本降低，能够在MLAgentBench的所有任务中平均取得32.95%的更高成功率，而基于GPT4的单智能体系统则在所有任务中平均取得22.72%的成功率。

+   •

    我们表现最佳的多代理系统在与黄等人（[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)）基于 GPT4 的单代理系统相比时，能够在45.45%的任务中取得相同或更好的表现，而在其他任务中表现相当。

## 2\. MLAgentBench 数据集

MLAgentBench（黄等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)）是一个为评估机器学习（ML）任务中的 LLM 代理而设计的数据集。MLAgentBench 数据集中的 ML 任务定义明确，提供了简洁的目标描述、评估标准和提交指南。这些任务描述多样，不限于特定格式，允许广泛的任务定义。例如，任务包括提高给定数据集上的模型准确性或优化特定的性能指标。该数据集还提供了必要的文件，包含训练和测试数据，并详细描述了数据和指标。启动代码涵盖了如 PyTorch、TensorFlow、JAX 和 Keras 等多种 ML 框架，帮助代理快速入门。虽然一些任务（如 cifar10、ogbn-arxiv 等）提供了用于比较的基线实现，其他任务（如 imdb、house-price 等）要求代理根据提供的规格和数据集文件从头开始编写模型。

在 MLAgentBench 框架中，每个任务代表一个环境，代理通过执行操作并接收观察来进行交互。该基准测试提供了一组基本的低级操作，包括文件系统操作（例如，列出文件、读取、写入、追加、复制等）、执行 Python 脚本以及声明最终答案。此外，还存在一些高级操作，如理解文件、反思（根据给定的描述回顾过去的步骤并进行思考）、检查文件的一部分以及编辑脚本（或脚本段）。高级操作可能会内部调用一些低级操作或 LLM（例如，理解文件操作可能会将文件内容传递给 LLM，并要求其理解内容）。每个操作都附有全面的文档，说明其名称、描述、使用指南、预期返回值和实现方式。这些操作使得 LLM 代理能够在任务环境中操控文件、执行脚本，并声明最终结果，从而促进迭代问题解决和评估。

我们考虑了MLAgentBench数据集的一个子集。我们没有考虑两个任务，即fathomnet（Woodward等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib38)）和identify-contrails（Ng等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib25)），因为我们的计算资源限制无法处理这些任务对应的数据集的极大规模。我们还去除了MLAgentBench中的LLM工具任务，因为它没有任何可与我们系统评估结果相比较的结果。因此，我们考虑了以下任务：（i）经典任务，如cifar10（Krizhevsky，[2009](https://arxiv.org/html/2411.07464v2#bib.bib20)），imdb（Maas等人，[2011](https://arxiv.org/html/2411.07464v2#bib.bib24)）和ogbn-arxiv（Hu等人，[2021](https://arxiv.org/html/2411.07464v2#bib.bib14)），（ii）经典Kaggle任务，如house-price（Anna Montoya，[2016](https://arxiv.org/html/2411.07464v2#bib.bib3)）和spaceship-titanic（Addison Howard，[2022](https://arxiv.org/html/2411.07464v2#bib.bib2)），（iii）Kaggle挑战任务，如parkinsons-disease（Leslie Kirsch和Dardov，[2023](https://arxiv.org/html/2411.07464v2#bib.bib21)）和feedback（Franklin等人，[2022](https://arxiv.org/html/2411.07464v2#bib.bib10)），（iv）当前研究任务，如CLRS（Veličković等人，[2022](https://arxiv.org/html/2411.07464v2#bib.bib34)）和BabyLM（Warstadt等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib37)），以及（v）改进代码任务，如llama-inference和vectorization。

## 3\. BudgetMLAgent

![参考标题](img/ff3d10b8486f4f8957bb545a4d9dca7f.png)

图1. BudgetMLAgent在第[3](https://arxiv.org/html/2411.07464v2#S3 "3\. BudgetMLAgent ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks")节中定义，具有概况、级联、求助专家生命线和从日志中检索功能。请注意，GPT4级联和规划专家调用都计入最大GPT4调用限制。Ge: Gemini Pro（无成本LLM代理）。

我们提出的系统BudgetMLAgent是一个LLM多代理系统，主要使用无成本LLM，并结合了几个增强功能，即：（i）LLM概况，（ii）LLM级联和（iii）求助专家生命线。

### 3.1\. 多代理LLM概况

我们的系统借鉴了MLAgentBench (Huang等，[2023](https://arxiv.org/html/2411.07464v2#bib.bib17))的基础工作，该项目为任务提供了一个直接的单一LLM代理基于的解决方案。它通过一个组织良好的提示-响应互动系统运行，代理通过一组可用的动作与环境互动。通过精心设计的提示，它们旨在确保清晰和精确地传达任务描述、可用工具（可能的动作集合）和最近采取的步骤，以增强代理的决策过程。为了强调在规划过程中深思熟虑的决策，LLM被指示在回答上述结构化提示时坚持结构化格式，包括‘反思’以理解先前的观察，更新的‘实施计划’和逐步‘状态’，‘事实检查’以确认计划和状态中的目标声明是否被猜测或直接确认，以及‘思考’即要执行的动作的理由和推理。这应包括接下来的‘行动’建议，以及对应的‘行动输入’（以JSON格式）。这种结构化的响应格式旨在增强代理进行反思性思考、更好的规划和结果验证的能力。它们还使用了一种受内存流范式（Park等，[2023](https://arxiv.org/html/2411.07464v2#bib.bib26)）启发的日志记录机制，可以有效管理历史数据，而不是让LLM面临庞大的历史上下文。通过采纳这种设计，确保日志文件充当一个相关信息的存储库，代理可以轻松地检索和更新。启用检索（$R$）的运行表示启用了此功能的运行。由此，检索到的信息与最近的行动和观察共同构成了历史上下文。检索到的信息充当长期记忆，而最近的行动和观察充当短期记忆。 |

| 类型 | 代理 | 配置文件 |
| --- | --- | --- |
| 规划者 | 默认规划者 | 你是一个解决机器学习任务的规划者。 |
| 规划专家 | 你是一个解决机器学习任务的规划专家。 |
| 高级动作执行者 | 理解文件 | 你是一个专家，能够理解同时包含代码和自然语言的文件。 |
| 编辑脚本（AI） | 你是一个编辑代码文件的专家。 |
| 反思 | 你是一个专家，擅长在解决机器学习任务时反思之前的行动。 |
| 检查脚本行 | 不适用 |
| 低级动作执行者 | 列出文件 | 不适用 |
| 复制文件 | 不适用 |
| 撤销编辑脚本 | 不适用 |
| 最终答案 | 不适用 |
| 执行脚本 | 不适用 |

表 1\. 代理人用于可调用的动作和通过系统提示给代理人传递的配置文件，涉及LLM调用。NA - 无配置文件，因为这些是程序化代理而非LLM代理。

我们将上述框架扩展到多代理LLM系统。多代理LLM配置指的是使用LLM配置将多个代理的专业知识结合起来的技术，即为每个代理分配不同的角色。我们通过将代理分为两个特定类别来表征我们系统的多代理特性 - (i) 规划者（P），它利用上述代理结构来考虑历史上下文并“规划”下一步动作，和 (ii) 执行者（W[i]s），即执行这些动作的代理。除了为规划者提供配置外，我们还为执行不同任务（如编辑脚本、理解文件等）并涉及调用LLM的工人代理提供不同的角色，如表[1](https://arxiv.org/html/2411.07464v2#S3.T1 "表1 ‣ 3.1. 多代理LLM配置 ‣ 3. BudgetMLAgent ‣ BudgetMLAgent：一种用于自动化机器学习任务的成本效益高的LLM多代理系统")所示。我们没有使用默认的“您是一个有帮助的AI助手”系统提示，而是为每个动作提供了不同的角色，形成了一个专注于不同角色的代理系统。这些工人代理之间没有相互作用，而是由规划者P代理在选择执行相应动作时进行调用。执行这些动作可能会或不会涉及内部LLM调用。

### 3.2. LLM级联

LLM级联指的是有条件地调用顺序连接的LLM（L[1]，L[2]，... L[k]）的技术。在这里，我们以一种方式将LLM串联，其中$cost(L_{1})<cost(L_{2})...<cost(L_{k})$。其中，成本由相应模型的最新定价信息表示。会执行一组协议来决定某一特定“级联”中LLM的响应是否可接受。如果可接受，那么响应将直接使用；如果不可接受，则我们将向上移动级联到下一个LLM。例如，级联中的LLM可以是Gemini-Pro（一个无成本的LLM），接着是GPT4（一个昂贵的LLM）。如果Gemini-Pro在用尽最大重试次数之前未能生成可接受的响应，则系统将调用GPT4来处理该步骤。在我们的研究中，向上移动级联的协议有两种情况： (i) 如果当前LLM在最大$ m $次尝试后仍未生成符合规定格式的响应，或者 (ii) 如果当前LLM选择的动作在过去$r$步中已连续出现$r$次。

### 3.3. 向专家求助的动作

为了节省开支，我们主要使用无成本的LLM进行规划和基于动作的LLM调用。初步调查表明，这些LLM在规划方面存在短板，但在其他基于动作的LLM调用中，它们的响应质量足够高。因此，我们为拥有Planner P角色的LLM提供$l$条“生命线”，在其识别到卡住某个步骤时，可以选择调用一个更大、更加昂贵且具备更高专业性的LLM。在这种情况下，LLM可以选择执行“请求规划专家帮助”的动作。实际上，这一较大模型调用的上限（生命线）也包括根据[3.2](https://arxiv.org/html/2411.07464v2#S3.SS2 "3.2\. LLM级联 ‣ 3\. BudgetMLAgent ‣ BudgetMLAgent：一个经济高效的LLM多代理系统，用于自动化机器学习任务")节中提到的级联协议所产生的调用。在一个运行过程中，一旦规划者达到此上限，当填充可调用动作时，"请求规划专家帮助"的动作将不再出现在可用动作列表中。

## 4. 实验与结果

我们在[2](https://arxiv.org/html/2411.07464v2#S2 "2\. MLAgentBench 数据集 ‣ BudgetMLAgent：一个经济高效的LLM多代理系统，用于自动化机器学习任务")节中解释的MLAgentBench数据集子集上进行实验。

### 4.1. 指标

我们通过考虑两个指标来评估我们的结果。

#### 4.1.1. 成功率

成功率是被认为成功的运行占总运行次数的百分比（%）。根据黄等人（[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)）的定义，如果在最后一步中，相较于起始代码中基准的平均表现，某次运行在性能上提高了10%以上，则认为该运行是成功的。这里的性能衡量标准是针对任务的具体要求。对于经典任务、经典Kaggle竞赛、Kaggle挑战赛以及本节[2](https://arxiv.org/html/2411.07464v2#S2 "2\. MLAgentBench 数据集 ‣ BudgetMLAgent：一个经济高效的LLM多代理系统，用于自动化机器学习任务")中提到的当前研究类型任务，最终提交的`submission.csv`文件的预测准确度被视为性能指标。对于代码改进类任务，代码的运行时间改进被视为成功指标；而对于CLRS任务，保存的最终模型检查点会被评估其准确度，准确度的提升被视为成功标准。

#### 4.1.2. 成本

如果一个大语言模型（LLM）具有与之相关的货币成本，我们将基于该模型使用的令牌数计算每次运行的平均成本（以美元$计）⁵⁵5 2024年3月的定价信息。对于可以使用API的LLM，成本公式为$\$[($每个输入令牌成本 $*$ 输入令牌数$)+($每个输出令牌成本 $*$ 输出令牌数$)]$。Claude1 V1.0作为单一代理在黄等人（[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)）的研究中已停用，因此最新的定价信息无法提供。为了近似Claude的成本，我们使用Claude-Instant的定价信息⁶⁶6[https://www.anthropic.com/api](https://www.anthropic.com/api)。

### 4.2\. 模型与超参数

我们分析了多个无成本的大语言模型（LLM），如Gemini、CodeLlama和Mixtral，作为单一代理，测试其机器学习问题解决能力。我们为提议的多代理框架采用了两种配置：（i）Gemini，作为性能最好的单一代理LLM，与来自[https://platform.openai.com/docs/models/gpt-3-5-turbo](https://platform.openai.com/docs/models/gpt-3-5-turbo)的ChatGPT ⁷⁷7gpt-3.5-turbo级联。（ii）Gemini与GPT4级联并包含“询问专家”代理。对于我们的运行，我们按照黄等人（[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)）的实现设置了所有超参数。最大动作数设置为30。最大包含在上下文中的近期动作数（作为短期记忆）设置为3。对于涉及级联的运行，我们将允许的最大重试次数（$m$）设置为Gemini-Pro为3，ChatGPT为3，GPT4为1，以控制成本。对于“询问专家”GPT4调用，我们也将允许的最大重试次数设置为1。每个动作连续重复的最大次数（$r$）设置为3。对于与规划相关的LLM调用，温度设置为0.2；对于与内部动作相关的LLM调用，温度设置为0.01，因为前者需要一定的输出多样性，而后者需要更明确的响应。对于涉及“询问规划专家”调用的运行，我们使用GPT4作为规划专家，并将对GPT4的最大调用次数（生命线$l$）设置为5。请注意，这也包括在LLM级联中对GPT4的调用。由于黄等人（[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)）未提供GPT4和Claude单一代理运行的确切货币成本，我们将使用其他单一代理运行的平均令牌使用量，乘以0.809的因子，以考虑到GPT4令牌使用量减少19.1%，这一点是黄等人（[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)）建议的。

### 4.3\. 基准

我们使用以下基准：(i) 单一代理 GPT4 在检索设置中的表现（G + R）(ii) 单一代理 GPT4 无检索（G）(iii) 单一代理 Claude V1.0 在检索设置中的表现（C + R）(iv) 单一代理 Claude V1.0 无检索（C）((i) 到 (iv) 参考 Huang 等，[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)) (v) 单一代理 Gemini Pro 在检索设置中的表现（Ge + R）(vi) 单一代理 Code Llama 在检索设置中的表现（Co + R）(vii) 单一代理 Mixtral 在检索设置中的表现（Mx + R）((v) 到 (vii) 是我们使用的无成本 LLM 基准)

|  | 单一代理（Huang 等，[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)） | 轮廓+级联 | 轮廓+级联+专家 |
| --- | --- | --- | --- |
| 任务 | G + R | G | C + R | C | Ge + R | Ge | Ge + Ch + R | Ge + Ch | Ge + G + R | Ge + G |

|

&#124; cifar10 &#124;

|

&#124; 25 &#124;

&#124; ($0.583) &#124;

|

&#124; 50 &#124;

&#124; ($0.44) &#124;

|

&#124; 8 &#124;

&#124; ($0.058) &#124;

|

&#124; 48 &#124;

&#124; ($0.044) &#124;

|

&#124; 12.5 &#124;

&#124; ($0)* &#124;

&#124; ($0.016) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.044) &#124;

|

&#124; 62.5 &#124;

&#124; ($0)* &#124;

&#124; ($0.025) &#124;

|

&#124; 25 &#124;

&#124; ($0)* &#124;

&#124; ($0.093) &#124;

|

&#124; 75 &#124;

&#124; ($0.057)* &#124;

&#124; ($0.094) &#124;

|

&#124; 37.5 &#124;

&#124; ($0.06)* &#124;

&#124; ($0.034) &#124;

|

|

&#124; imdb &#124;

|

&#124; 12.5 &#124;

&#124; ($1.48) &#124;

|

&#124; 25 &#124;

&#124; ($0.919) &#124;

|

&#124; 0 &#124;

&#124; ($0.147) &#124;

|

&#124; 0 &#124;

&#124; ($0.091) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.006) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.038) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.022) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.023) &#124;

|

&#124; 0 &#124;

&#124; ($0.014)* &#124;

&#124; ($0.060) &#124;

|

&#124; 0 &#124;

&#124; ($0.071)* &#124;

&#124; ($0.124) &#124;

|

|

&#124; ogbn-arxiv &#124;

|

&#124; 50 &#124;

&#124; ($1.27) &#124;

|

&#124; 87.5 &#124;

&#124; ($1.112) &#124;

|

&#124; 40 &#124;

&#124; ($0.125) &#124;

|

&#124; 32 &#124;

&#124; ($0.109) &#124;

|

&#124; 25 &#124;

&#124; ($0)* &#124;

&#124; ($0.011) &#124;

|

&#124; 25 &#124;

&#124; ($0)* &#124;

&#124; ($0.016) &#124;

|

&#124; 50 &#124;

&#124; ($0)* &#124;

&#124; ($0.020) &#124;

|

&#124; 50 &#124;

&#124; ($0)* &#124;

&#124; ($0.015) &#124;

|

&#124; 50 &#124;

&#124; ($0.033)* &#124;

&#124; ($0.067) &#124;

|

&#124; 75 &#124;

&#124; ($0.026)* &#124;

&#124; ($0.045) &#124;

|

|

&#124; house-price &#124;

|

&#124; 25 &#124;

&#124; ($1.6) &#124;

|

&#124; 12.5 &#124;

&#124; ($0.938) &#124;

|

&#124; 64 &#124;

&#124; ($0.158) &#124;

|

&#124; 76 &#124;

&#124; ($0.093) &#124;

|

&#124; 25 &#124;

&#124; ($0)* &#124;

&#124; ($0.042) &#124;

|

&#124; 37.5 &#124;

&#124; ($0)* &#124;

&#124; ($0.070) &#124;

|

&#124; 62.5 &#124;

&#124; ($0)* &#124;

&#124; ($0.046) &#124;

|

&#124; 62.5 &#124;

&#124; ($0)* &#124;

&#124; ($0.049) &#124;

|

&#124; 87.5 &#124;

&#124; ($0.091)* &#124;

&#124; ($0.161) &#124;

|

&#124; 75 &#124;

&#124; ($0.038)* &#124;

&#124; ($0.153) &#124;

|

|

&#124; spaceship- &#124;

&#124; titanic &#124;

|

&#124; 25 &#124;

&#124; ($1.42) &#124;

|

&#124; 12.5 &#124;

&#124; ($0.85) &#124;

|

&#124; 4 &#124;

&#124; ($0.141) &#124;

|

&#124; 16 &#124;

&#124; ($0.088) &#124;

|

&#124; 37.5 &#124;

&#124; ($0)* &#124;

&#124; ($0.039) &#124;

|

&#124; 37.5 &#124;

&#124; ($0)* &#124;

&#124; ($0.052) &#124;

|

&#124; 75 &#124;

&#124; ($0)* &#124;

&#124; ($0.077) &#124;

|

&#124; 100 &#124;

&#124; ($0.0004)* &#124;

&#124; ($0.038) &#124;

|

&#124; 75 &#124;

&#124; ($0.021)* &#124;

&#124; ($0.096) &#124;

|

&#124; 100 &#124;

&#124; ($0.091)* &#124;

&#124; ($0.213) &#124;

|

|

&#124; parkinsons- &#124;

&#124; disease &#124;

|

&#124; 12.5 &#124;

&#124; ($2.9) &#124;

|

&#124; 0 &#124;

&#124; ($1.57) &#124;

|

&#124; 0 &#124;

&#124; ($0.287) &#124;

|

&#124; 0 &#124;

&#124; ($0.155) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.078) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.065) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.024) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.028) &#124;

|

&#124; 0 &#124;

&#124; ($0.099)* &#124;

&#124; ($0.186) &#124;

|

&#124; 0 &#124;

&#124; ($0.107)* &#124;

&#124; ($0.183) &#124;

|

|

&#124; feedback &#124;

|

&#124; 37.5 &#124;

&#124; ($1.25) &#124;

|

&#124; 12.5 &#124;

&#124; ($1.15) &#124;

|

&#124; 0 &#124;

&#124; ($0.124) &#124;

|

&#124; 0 &#124;

&#124; ($0.114) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.058) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.046) &#124;

|

&#124; 0 &#124;

&#124; ($0.0005)* &#124;

&#124; ($0.038) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.030) &#124;

|

&#124; 0 &#124;

&#124; ($0.047)* &#124;

&#124; ($0.110) &#124;

|

&#124; 0 &#124;

&#124; ($0.022)* &#124;

&#124; ($0.068) &#124;

|

|

&#124; llama- &#124;

&#124; inference &#124;

|

&#124; 0 &#124;

&#124; ($1.46) &#124;

|

&#124; 0 &#124;

&#124; ($0.927) &#124;

|

&#124; 0 &#124;

&#124; ($0.145) &#124;

|

&#124; 0 &#124;

&#124; ($0.092) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.030) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.059) &#124;

|

&#124; 0 &#124;

&#124; ($0.0003)* &#124;

&#124; ($0.032) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.034) &#124;

|

&#124; 0 &#124;

&#124; ($0.027)* &#124;

&#124; ($0.074) &#124;

|

&#124; 0 &#124;

&#124; ($0.055)* &#124;

&#124; ($0.093) &#124;

|

|

&#124; vectorization &#124;

|

&#124; 0 &#124;

&#124; ($1.16) &#124;

|

&#124; 0 &#124;

&#124; ($1.23) &#124;

|

&#124; 0 &#124;

&#124; ($0.114) &#124;

|

&#124; 0 &#124;

&#124; ($0.121) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.062) &#124;

|

&#124; 12.5 &#124;

&#124; ($0)* &#124;

&#124; ($0.058) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.039) &#124;

|

&#124; 12.5 &#124;

&#124; ($0)* &#124;

&#124; ($0.025) &#124;

|

&#124; 0 &#124;

&#124; ($0.017)* &#124;

&#124; ($0.077) &#124;

|

&#124; 75 &#124;

&#124; ($0.004)* &#124;

&#124; ($0.092) &#124;

|

|

&#124; CLRS &#124;

|

&#124; 12.5 &#124;

&#124; ($0.523) &#124;

|

&#124; 50 &#124;

&#124; ($0.43) &#124;

|

&#124; 52 &#124;

&#124; ($0.052) &#124;

|

&#124; 40 &#124;

&#124; ($0.042) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.006) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.006*) &#124;

|

&#124; 0 &#124;

&#124; ($0.0003)* &#124;

&#124; ($0.033) &#124;

|

&#124; 12.5 &#124;

&#124; ($0)* &#124;

&#124; ($0.022) &#124;

|

&#124; 0 &#124;

&#124; ($0.027)* &#124;

&#124; ($0.050) &#124;

|

&#124; 0 &#124;

&#124; ($0.046)* &#124;

&#124; ($0.138) &#124;

|

|

&#124; babylm &#124;

|

&#124; 0 &#124;

&#124; ($0.818) &#124;

|

&#124; 0 &#124;

&#124; ($0.637) &#124;

|

&#124; 0 &#124;

&#124; ($0.081) &#124;

|

&#124; 0 &#124;

&#124; ($0.063) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0.032) &#124;

|

&#124; 0 &#124;

&#124; ($0)* &#124;

&#124; ($0*.027) &#124;

|

&#124; 0 &#124;

&#124; ($0.0025)* &#124;

&#124; ($0.046) &#124;

|

&#124; 0 &#124;

&#124; ($0.001)* &#124;

&#124; ($0.024) &#124;

|

&#124; 0 &#124;

&#124; ($0.086)* &#124;

&#124; ($0.147) &#124;

|

&#124; 0 &#124;

&#124; ($0.078)* &#124;

&#124; ($0.113) &#124;

|

| 平均 |
| --- |

&#124; 18.18 &#124;

&#124; ($1.315) &#124;

|

&#124; 22.72 &#124;

&#124; ($0.931) &#124;

|

&#124; 15.27 &#124;

&#124; ($0.13) &#124;

|

&#124; 20 &#124;

&#124; ($0.092) &#124;

|

&#124; 9.09 &#124;

&#124; ($0)* &#124;

&#124; ($0.034) &#124;

|

&#124; 10.23 &#124;

&#124; ($0)* &#124;

&#124; ($0.044) &#124;

|

&#124; 22.72 &#124;

&#124; ($0.0003)* &#124;

&#124; ($0.036) &#124;

|

&#124; 22.72 &#124;

&#124; ($0.0001)* &#124;

&#124; ($0.032) &#124;

|

&#124; 26.14 &#124;

&#124; ($0.047)* &#124;

&#124; ($0.102) &#124;

|

&#124; 32.95 &#124;

&#124; ($0.054)* &#124;

&#124; ($0.120) &#124;

|

表格 2\. 每个单元格显示的是成功率（%）：成功运行的百分比，($)：每次运行的平均成本，* - 考虑免费的 Gemini Pro API 调用；G - GPT4，C - Claude V1.0，Ge - Gemini Pro，Co - CodeLlama Instruct 34b，Ch - ChatGPT (GPT 3.5 Turbo)，R - 检索：该代理可以从长期记忆中的研究日志中检索并总结相关信息。在无检索设置下，此功能被禁用。

### 4.4\. 结果与讨论

我们根据表格[2](https://arxiv.org/html/2411.07464v2#S4.T2 "Table 2 ‣ 4.3\. Baselines ‣ 4\. Experimentation and Results ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks")的结果解决关键的研究问题（RQ）。

#### 4.4.1\. RQ1 - 低成本和无成本的 LLM 单一代理是否为了节省成本而牺牲了性能？

我们观察到，当在单一代理设置中使用纯粹的无成本或低成本LLM（Ge、Co和Mx）时，相比于使用GPT4或Claude的LLM单一代理（见Huang等，([2023](https://arxiv.org/html/2411.07464v2#bib.bib17))），性能显著下降。我们观察到，CodeLlama（Co）和Mixtral（Mx）无法成功运行，导致平均成功率为0%。因此，我们将它们从表[2](https://arxiv.org/html/2411.07464v2#S4.T2 "Table 2 ‣ 4.3\. Baselines ‣ 4\. Experimentation and Results ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks")中的结果中剔除。我们发现Mixtral几乎从未能遵守[3.1](https://arxiv.org/html/2411.07464v2#S3.SS1 "3.1\. Multi-Agent LLM Profiling ‣ 3\. BudgetMLAgent ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks")节中提到的要求的响应格式，这导致由于超出最大重试限制而终止。然而，我们观察到Gemini-Pro（Ge + R）能够在单一代理设置中产生成功的运行，针对一些任务（cifar10、ogbn-arxiv、house-price和spaceship-titanic）获得非零性能，但其他任务则为零。通过视觉检查发现，Gemini-Pro和CodeLlama在内部与动作相关的LLM调用之间的响应质量没有太大差异，前者略优。然而，我们观察到，由于Code Llama无法有效进行规划，它无法成功运行。

#### 4.4.2\. RQ2 - **配置文件分析**和**级联**如何影响性能和成本？

从表格[2](https://arxiv.org/html/2411.07464v2#S4.T2 "Table 2 ‣ 4.3\. Baselines ‣ 4\. Experimentation and Results ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks")中可以看出，使用Gemini-Pro作为基础LLM，并与ChatGPT（GPT-3.5-turbo）级联（Ge + Ch），无论是否启用检索器设置，相较于单一使用Ge作为代理时，许多任务的成功率显著提高，特别是在cifar10、ogbn-arxiv、house-price、spaceship-titanic和vectorization任务中。然而，对于imdb、parkinsons-disease、feedback、llama-inference、CLRS和babylm任务，性能仍然为零，这是由于这些任务的复杂性。总体而言，级联的平均成功率与GPT4（G）的表现相当。这些改进是在几乎100%的成本减少下实现的，对于无成本的Gemini-Pro（从G + R的$1.315和C + R的$0.13到Ge + Ch + R的$0.0003，从G的$0.931和C的$0.092到Ge + Ch的$0.0001）而言。原因在于GPT-3.5-turbo的推理成本非常低，从而导致微不足道的费用。然而，定性分析表明，GPT-3.5-turbo经常未能遵守第[3.1](https://arxiv.org/html/2411.07464v2#S3.SS1 "3.1\. Multi-Agent LLM Profiling ‣ 3\. BudgetMLAgent ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks")节中提到的所需响应格式，导致需要进一步重试。因此，对于后续运行，我们转向使用GPT4进行级联。

#### 4.4.3\. RQ3 - 向分析和级联中添加“求助专家”生命线如何影响性能和成本？

表[2](https://arxiv.org/html/2411.07464v2#S4.T2 "表 2 ‣ 4.3\. 基线 ‣ 4\. 实验与结果 ‣ BudgetMLAgent: 一种用于自动化机器学习任务的成本效益高的LLM多智能体系统")显示，作为我们提议的系统一部分的GPT4专家求助电话（Ge + G和Ge + G + R）在与Ge + Ch+ R和Ge + Ch比较时，能提高cifar10、ogbn-arxiv、house-price、spaceship-titanic和vectorization任务的成功率。此外，我们观察到，与G + R和G相比，cifar10、house-price、spaceship-titanic和vectorization任务的成功率有所提升，而在无成本Gemini-Pro上，任务成本减少幅度达到90-99%。平均而言，Profiling + Cascade + Expert设置，并使用GPT4进行级联和专家任务，在检索设置下，比单一的GPT4和Claude代理分别提高了43.78%和71.19%的成功率，而在无检索设置下分别提高了45.02%和64.75%。这也伴随着无成本Gemini-Pro在检索设置下与GPT4和Claude相比的96.43%和63.85%的成本降低，以及在无检索设置下的94.2%和41.3%的成本降低，展示了我们方法的有效性。定性分析表明，有时规划者能够成功识别自己卡住的情况，并调用专家“解困”。例如，如果在多次步骤后，某个特定的编辑操作没有导致正确的执行，而专家被召唤后，采取了不同的行动，如在进行进一步编辑前，更好地理解文件的某一部分。

#### 4.4.4\. RQ4 - 从日志中检索对性能和成本的影响如何？

根据黄等人（[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)）的研究发现，我们的观察表明，虽然从日志中检索的访问对于某些任务有效，但对于其他任务，禁用该功能可以得到更好的结果。一个有趣的案例是cifar10。GPT4（G）的成功率高于G + R。黄等人（[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)）通过指出，cifar10是一个相对简单的任务，长期记忆上下文反而成了干扰，来解释这一现象。但在Gemini-Pro与GPT4级联和专家设置的情况下，趋势发生了逆转。Ge + G + R的成功率高于Ge + G。这可能是由于Gemini-Pro和GPT4在预训练和实际数据上有所不同。与成功率类似，对于某些任务，检索会导致更高的成本，而对于其他任务，则是没有检索时成本更高。在无成本的Gemini-Pro上，Ge + G的每次运行平均成本（$0.054）高于Ge + G + R（$0.047）。

## 5\. 相关工作

### 5.1\. LLM代理在细化代码相关任务中的应用

随着大语言模型（LLM）能力的不断提升，使用 LLM 智能体进行与代码相关的任务也取得了许多进展。最初的工作主要依赖于 HumanEval（Chen 等人，[2021](https://arxiv.org/html/2411.07464v2#bib.bib7)）和 MBXP（Athiwaratkun 等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib4)）来基准测试他们的方法。然而，近期的工作开始转向更为细化和实际的基准测试。Wang 等人（[2024](https://arxiv.org/html/2411.07464v2#bib.bib36)）提出了 CodeAct —— 一个包含与 API 处理、库使用等相关任务的 Python 代码数据库。他们还提出了 CodeActAgent，它在 Llama-2（Touvron 等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib33)）和 Mistral-7b（Jiang 等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib18)）上进行了微调，并使用 CodeActInstruct，一个包含多轮互动的指令调优数据集。AgentBench（Liu 等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib23)）还介绍了三个基于操作系统、数据库和知识图谱相关问题的代码智能体环境，用于评估 LLM 智能体在这些任务中的表现。还有一些工作致力于解决更为复杂的问题。Zhang 等人（[2024b](https://arxiv.org/html/2411.07464v2#bib.bib46)）构建了 CodeAgentBench，一个包含现实世界仓库级编程挑战的数据集。他们还提出了 CodeAgent，利用外部信息检索、代码实现和测试工具来解决该数据集中的任务。

Huang 等人（[2023](https://arxiv.org/html/2411.07464v2#bib.bib17)）构建了 MLAgentBench，如第 [2](https://arxiv.org/html/2411.07464v2#S2 "2\. MLAgentBench Dataset ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks") 节所述，包含了一些机器学习任务。他们还提出了一种基于单个智能体的系统，利用各种工具（如代码执行、代码编辑等）来解决这些任务。Tang 等人（[2024](https://arxiv.org/html/2411.07464v2#bib.bib31)）探讨了使用 LLM 智能体处理仓库级代码的方式。然而，他们的方法并不涉及直接的代码生成。相反，他们专注于创建 Shell 脚本，利用现有的代码文件并传递适当的参数。类似地，CodePlan（Bairi 等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib5)）将仓库级编程视为一种规划问题，表现为一系列编辑链。此外，NL2Repo（Zan 等人，[2024](https://arxiv.org/html/2411.07464v2#bib.bib44)）提出了一种从零开始生成代码库的系统，利用自然语言规范。

### 5.2\. LLM 多智能体

虽然单一的LLM（大语言模型）代理在各种任务中已经展示出卓越的能力，但现实世界中的复杂挑战往往需要协作方法。在这种背景下，LLM多代理系统的探索成为一种有前景的途径，用于增强问题解决能力，并更有效地处理复杂任务。ChatDev（Qian等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib27)）和MetaGPT（Hong等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib13)）是最近为解决软件开发任务而引入的多代理系统。AutoGen（Wu等人，[2023](https://arxiv.org/html/2411.07464v2#bib.bib39)）是一个更为通用的框架，用于开发LLM多代理，并已在多个任务中进行评估，从数学问题解决到增强检索聊天（Li等人，[2024](https://arxiv.org/html/2411.07464v2#bib.bib22)）；Shen等人（[2024](https://arxiv.org/html/2411.07464v2#bib.bib30)）也提出了框架，旨在协作使用多个LLM代理来完成推理和工具学习等任务。Wang等人（[2023](https://arxiv.org/html/2411.07464v2#bib.bib35)）；Ding等人（[2024](https://arxiv.org/html/2411.07464v2#bib.bib8)）提出结合多个LLM专家代理来解决各种任务，包括文本总结和问答。为了节省成本，Chen等人（[2023](https://arxiv.org/html/2411.07464v2#bib.bib6)）；Yue等人（[2024](https://arxiv.org/html/2411.07464v2#bib.bib42)）；Zhang等人（[2023](https://arxiv.org/html/2411.07464v2#bib.bib45)）提出在不同任务中使用逐级递增的LLM，从而在推理、查询回答和新闻预测等任务中实现成本和能力的平衡。我们的关注点是提供一种成本效益高的解决方案，用于使用多代理系统处理ML工程任务，基础是无成本的开源LLM，并通过级联方式使用具有更高专业性的付费LLM。

## 6\. 结论

在本工作中，我们提出了 BudgetMLAgent——一个 LLM 多智能体系统，用于以成本效益的方式解决机器学习任务而不影响系统性能。我们主要对在 MLAgentBench 数据集上定义的任务进行研究，单智能体系统使用纯粹的无成本模型，对于 CodeLlama 和 Mixtral 成功率为零，而 Gemini-Pro 的成功率也非常低（在访问完整日志时为 9.09%，在短期访问时为 10.23%），与付费模型如 GPT4 和 ClaudeV1.0（分别为 18.18% 和 22.72%，15.27% 和 20%）相比。随后，我们提出了一个多智能体框架 BudgetMLAgent，使用无成本的 Gemini-Pro 作为基础 LLM，利用（i）为规划者配置多个工作智能体，通过不同的动作与 ML 代码生成环境进行交互，（ii）级联到更有经验的 LLM，如 GPT-3.5-turbo 和 GPT4，以及（iii）我们的新型“问专家”GPT4 救生圈。与基于 GPT4 和 ClaudeV1.0 的单智能体系统相比，BudgetMLAgent 提高了 MLAgentBench 任务的成功率（分别为 26.14% 和 32.95%），并在成本上有显著降低（在访问完整日志时为 96.43% 和 63.85%，在短期访问时分别降低 94.2% 和 41.3%）。

## 参考文献

+   （1）

+   Addison Howard（2022年）Ryan Holbrook Addison Howard、Ashley Chow。2022年。太空船泰坦尼克号。[https://kaggle.com/competitions/spaceship-titanic](https://kaggle.com/competitions/spaceship-titanic)

+   Anna Montoya（2016年）DataCanary Anna Montoya。2016年。房价 - 高级回归技术。[https://kaggle.com/competitions/house-prices-advanced-regression-techniques](https://kaggle.com/competitions/house-prices-advanced-regression-techniques)

+   Athiwaratkun 等人（2023年）Ben Athiwaratkun、Sanjay Krishna Gouda、Zijian Wang、Xiaopeng Li、Yuchen Tian、Ming Tan、Wasi Uddin Ahmad、Shiqi Wang、Qing Sun、Mingyue Shang、Sujan Kumar Gonugondla、Hantian Ding、Varun Kumar、Nathan Fulton、Arash Farahani、Siddhartha Jain、Robert Giaquinto、Haifeng Qian、Murali Krishna Ramanathan、Ramesh Nallapati、Baishakhi Ray、Parminder Bhatia、Sudipta Sengupta、Dan Roth 和 Bing Xiang。2023年。多语言代码生成模型评估。arXiv:2210.14868 [cs.LG]

+   Bairi 等人（2023年）Ramakrishna Bairi、Atharv Sonwane、Aditya Kanade、Vageesh D C、Arun Iyer、Suresh Parthasarathy、Sriram Rajamani、B. Ashok 和 Shashank Shet。2023年。CodePlan：使用 LLM 和规划进行仓库级编码。arXiv:2309.12499 [cs.SE] [https://arxiv.org/abs/2309.12499](https://arxiv.org/abs/2309.12499)

+   Chen 等人（2023年）Lingjiao Chen、Matei Zaharia 和 James Zou。2023年。FrugalGPT：如何在降低成本并提高性能的同时使用大型语言模型。arXiv:2305.05176 [cs.LG]

+   Chen等人（2021）Mark Chen、Jerry Tworek、Heewoo Jun、Qiming Yuan、Henrique Ponde de Oliveira Pinto、Jared Kaplan、Harri Edwards、Yuri Burda、Nicholas Joseph、Greg Brockman、Alex Ray、Raul Puri、Gretchen Krueger、Michael Petrov、Heidy Khlaaf、Girish Sastry、Pamela Mishkin、Brooke Chan、Scott Gray、Nick Ryder、Mikhail Pavlov、Alethea Power、Lukasz Kaiser、Mohammad Bavarian、Clemens Winter、Philippe Tillet、Felipe Petroski Such、Dave Cummings、Matthias Plappert、Fotios Chantzis、Elizabeth Barnes、Ariel Herbert-Voss、William Hebgen Guss、Alex Nichol、Alex Paino、Nikolas Tezak、Jie Tang、Igor Babuschkin、Suchir Balaji、Shantanu Jain、William Saunders、Christopher Hesse、Andrew N. Carr、Jan Leike、Josh Achiam、Vedant Misra、Evan Morikawa、Alec Radford、Matthew Knight、Miles Brundage、Mira Murati、Katie Mayer、Peter Welinder、Bob McGrew、Dario Amodei、Sam McCandlish、Ilya Sutskever和Wojciech Zaremba。2021年。评估在代码上训练的大型语言模型。arXiv:2107.03374 [cs.LG]

+   Ding等人（2024）Dujian Ding、Ankur Mallick、Chi Wang、Robert Sim、Subhabrata Mukherjee、Victor Rühle、Laks V. S. Lakshmanan和Ahmed Hassan Awadallah。2024年。混合LLM：成本效益与质量意识的查询路由。发表于*第十二届国际学习表征会议*。 [https://openreview.net/forum?id=02f3mUtqnM](https://openreview.net/forum?id=02f3mUtqnM)

+   Fang等人（2024）Xi Fang、Weijie Xu、Fiona Anting Tan、Jiani Zhang、Ziqing Hu、Yanjun Qi、Scott Nickleach、Diego Socolinsky、Srinivasan Sengamedu和Christos Faloutsos。2024年。大语言模型（LLMs）在表格数据上的应用：预测、生成与理解——一项综述。arXiv:2402.17944 [cs.CL]

+   Franklin等人（2022）Alex Franklin、Maggie、Meg Benner、Natalie Rambis、Perpetual Baffour、Ryan Holbrook、Scott Crossley和Ulrich Boser。2022年。反馈奖 - 英语语言学习。 [https://kaggle.com/competitions/feedback-prize-english-language-learning](https://kaggle.com/competitions/feedback-prize-english-language-learning)

+   Guo等人（2024）Daya Guo、Qihao Zhu、Dejian Yang、Zhenda Xie、Kai Dong、Wentao Zhang、Guanting Chen、Xiao Bi、Y. Wu、Y. K. Li、Fuli Luo、Yingfei Xiong和Wenfeng Liang。2024年。DeepSeek-Coder：当大型语言模型遇上编程——代码智能的崛起。arXiv:2401.14196 [cs.SE]

+   He等人（2021）Xin He、Kaiyong Zhao和Xiaowen Chu。2021年。AutoML：最先进技术的综述。*Knowledge-Based Systems* 212（2021年1月），106622。 [https://doi.org/10.1016/j.knosys.2020.106622](https://doi.org/10.1016/j.knosys.2020.106622)

+   Hong等人（2023）Sirui Hong、Mingchen Zhuge、Jonathan Chen、Xiawu Zheng、Yuheng Cheng、Ceyao Zhang、Jinlin Wang、Zili Wang、Steven Ka Shing Yau、Zijuan Lin、Liyang Zhou、Chenyu Ran、Lingfeng Xiao、Chenglin Wu和Jürgen Schmidhuber。2023年。MetaGPT：用于多智能体协作框架的元编程。arXiv:2308.00352 [cs.AI]

+   胡等（2021）胡伟华、马蒂亚斯·费伊、马林卡·齐特尼克、董宇霄、任洪宇、刘博文、米凯莱·卡塔斯塔和尤尔·莱斯科维茨。2021年。开放图基准：图学习数据集。arXiv:2005.00687 [cs.LG]

+   黄等（2024）董黄、卜庆文、张杰·M、迈克尔·拉克和崔亨明。2024年。AgentCoder：基于多智能体的代码生成与迭代测试与优化。arXiv:2312.13010 [cs.CL]

+   黄与张（2023）黄杰与凯文·陈-川·张。2023年。迈向大语言模型中的推理：一项综述。arXiv:2212.10403 [cs.CL]

+   黄等（2023）黄乾、简·沃拉、佩尔西·梁和尤尔·莱斯科维茨。2023年。基准测试大语言模型作为AI研究智能体。arXiv:2310.03302 [cs.LG]

+   蒋等（2023）阿尔伯特·Q·蒋、亚历山大·萨布拉约尔、亚瑟·门什、克里斯·班福德、德文德拉·辛格·查普洛特、迭戈·德·拉斯·卡萨斯、弗洛里安·布雷桑、吉安娜·伦吉尔、吉约姆·兰普尔、卢西尔·索尔尼尔、莱利奥·雷纳尔德·拉沃、玛丽-安·拉肖、皮埃尔·斯托克、特文·勒·斯卡奥、蒂博·拉夫里尔、托马斯·王、蒂莫西·拉克鲁瓦和威廉·埃尔·赛义德。2023年。Mistral 7B。arXiv:2310.06825 [cs.CL]

+   蒋等（2024）阿尔伯特·Q·蒋、亚历山大·萨布拉约尔、安托万·鲁、亚瑟·门什、布兰奇·萨瓦里、克里斯·班福德、德文德拉·辛格·查普洛特、迭戈·德·拉斯·卡萨斯、艾玛·布胡·哈纳、弗洛里安·布雷桑、吉安娜·伦吉尔、吉约姆·布尔、吉约姆·兰普尔、莱利奥·雷纳尔德·拉沃、卢西尔·索尔尼尔、玛丽-安·拉肖、皮埃尔·斯托克、桑迪普·苏布拉马尼安、索非亚·杨、斯奇蒙·安东尼亚克、特文·勒·斯卡奥、泰奥菲尔·热尔韦、蒂博·拉夫里尔、托马斯·王、蒂莫西·拉克鲁瓦和威廉·埃尔·赛义德。2024年。Mixtral专家模型。arXiv:2401.04088 [cs.LG]

+   克里热夫斯基（2009）亚历克斯·克里热夫斯基。2009年。从微小图像中学习多层特征。[https://api.semanticscholar.org/CorpusID:18268744](https://api.semanticscholar.org/CorpusID:18268744)

+   莱斯利·基尔施与达尔多夫（2023）斯泰西·亚当·莱斯利·基尔施、索赫尔·丹恩和维多利亚·达尔多夫。2023年。AMP®-帕金森病进展预测。[https://kaggle.com/competitions/amp-parkinsons-disease-progression-prediction](https://kaggle.com/competitions/amp-parkinsons-disease-progression-prediction)

+   李等（2024）李俊尧、张沁、余扬斌、傅强和叶德恒。2024年。更多智能体就是你所需要的一切。arXiv:2402.05120 [cs.CL]

+   刘等（2023）肖刘、郝宇、张汉晨、许逸凡、雷轩宇、赖汉宇、顾宇、丁航梁、门凯文、杨克娟、张书丹、邓翔、曾傲涵、杜正晓、张晨辉、沈晟、张天俊、苏宇、孙欢、黄敏蕾、董宇霄和唐杰。2023年。AgentBench：评估大语言模型作为智能体的表现。arXiv:2308.03688 [cs.AI]

+   Maas 等人（2011）Andrew L. Maas, Raymond E. Daly, Peter T. Pham, Dan Huang, Andrew Y. Ng 和 Christopher Potts. 2011. 用于情感分析的词向量学习。收录于 *第49届计算语言学协会年会：人类语言技术论文集*，Dekang Lin, Yuji Matsumoto 和 Rada Mihalcea（编辑）。计算语言学协会，波特兰，俄勒冈州，美国，142–150。[https://aclanthology.org/P11-1015](https://aclanthology.org/P11-1015)

+   Ng 等人（2023）Joe Ng, Carl Elkin, Aaron Sarna, Walter Reade 和 Maggie Demkin. 2023. Google 研究 - 识别飞机尾迹以减少全球变暖。[https://kaggle.com/competitions/google-research-identify-contrails-reduce-global-warming](https://kaggle.com/competitions/google-research-identify-contrails-reduce-global-warming)

+   Park 等人（2023）Joon Sung Park, Joseph C. O’Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang 和 Michael S. Bernstein. 2023. 生成智能体：人类行为的交互式模拟。arXiv:2304.03442 [cs.HC]

+   Qian 等人（2023）Chen Qian, Xin Cong, Wei Liu, Cheng Yang, Weize Chen, Yusheng Su, Yufan Dang, Jiahao Li, Juyuan Xu, Dahai Li, Zhiyuan Liu 和 Maosong Sun. 2023. 软件开发中的交互式智能体。arXiv:2307.07924 [cs.SE]

+   Rozière 等人（2024）Baptiste Rozière, Jonas Gehring, Fabian Gloeckle, Sten Sootla, Itai Gat, Xiaoqing Ellen Tan, Yossi Adi, Jingyu Liu, Romain Sauvestre, Tal Remez, Jérémy Rapin, Artyom Kozhevnikov, Ivan Evtimov, Joanna Bitton, Manish Bhatt, Cristian Canton Ferrer, Aaron Grattafiori, Wenhan Xiong, Alexandre Défossez, Jade Copet, Faisal Azhar, Hugo Touvron, Louis Martin, Nicolas Usunier, Thomas Scialom 和 Gabriel Synnaeve. 2024. Code Llama：开放代码基础模型。arXiv:2308.12950 [cs.CL]

+   Salehin 等人（2024）Imrus Salehin, Md. Shamiul Islam, Pritom Saha, S.M. Noman, Azra Tuni, Md. Mehedi Hasan 和 Md. Abu Baten. 2024. AutoML：自动机器学习与神经架构搜索的系统综述。*信息与智能学报* 2, 1 (2024), 52–81。[https://doi.org/10.1016/j.jiixd.2023.10.002](https://doi.org/10.1016/j.jiixd.2023.10.002)

+   Shen 等人（2024）Weizhou Shen, Chenliang Li, Hongzhan Chen, Ming Yan, Xiaojun Quan, Hehong Chen, Ji Zhang 和 Fei Huang. 2024. 小型 LLM 是较弱的工具学习者：一个多 LLM 智能体。arXiv:2401.07324 [cs.AI]

+   Tang 等人（2024）Xiangru Tang, Yuliang Liu, Zefan Cai, Yanjun Shao, Junjie Lu, Yichi Zhang, Zexuan Deng, Helan Hu, Kaikai An, Ruijun Huang, Shuzheng Si, Sheng Chen, Haozhe Zhao, Liang Chen, Yan Wang, Tianyu Liu, Zhiwei Jiang, Baobao Chang, Yin Fang, Yujia Qin, Wangchunshu Zhou, Yilun Zhao, Arman Cohan 和 Mark Gerstein. 2024. ML-Bench: 评估大规模语言模型和智能体在机器学习任务中的表现，基于代码库层级。arXiv:2311.09835 [cs.CL] [https://arxiv.org/abs/2311.09835](https://arxiv.org/abs/2311.09835)

+   Team等人（2023）Gemini Team, Rohan Anil, Sebastian Borgeaud 等. 2023. Gemini: A Family of Highly Capable Multimodal Models. arXiv:2312.11805 [cs.CL]

+   Touvron等人（2023）Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, Dan Bikel, Lukas Blecher, Cristian Canton Ferrer, Moya Chen, Guillem Cucurull, David Esiobu, Jude Fernandes, Jeremy Fu, Wenyin Fu, Brian Fuller, Cynthia Gao, Vedanuj Goswami, Naman Goyal, Anthony Hartshorn, Saghar Hosseini, Rui Hou, Hakan Inan, Marcin Kardas, Viktor Kerkez, Madian Khabsa, Isabel Kloumann, Artem Korenev, Punit Singh Koura, Marie-Anne Lachaux, Thibaut Lavril, Jenya Lee, Diana Liskovich, Yinghai Lu, Yuning Mao, Xavier Martinet, Todor Mihaylov, Pushkar Mishra, Igor Molybog, Yixin Nie, Andrew Poulton, Jeremy Reizenstein, Rashi Rungta, Kalyan Saladi, Alan Schelten, Ruan Silva, Eric Michael Smith, Ranjan Subramanian, Xiaoqing Ellen Tan, Binh Tang, Ross Taylor, Adina Williams, Jian Xiang Kuan, Puxin Xu, Zheng Yan, Iliyan Zarov, Yuchen Zhang, Angela Fan, Melanie Kambadur, Sharan Narang, Aurelien Rodriguez, Robert Stojnic, Sergey Edunov 和 Thomas Scialom. 2023. Llama 2: Open Foundation and Fine-Tuned Chat Models. arXiv:2307.09288 [cs.CL]

+   Veličković等人（2022）Petar Veličković, Adrià Puigdomènech Badia, David Budden, Razvan Pascanu, Andrea Banino, Misha Dashevskiy, Raia Hadsell 和 Charles Blundell. 2022. The CLRS Algorithmic Reasoning Benchmark. arXiv:2205.15659 [cs.LG]

+   Wang等人（2023）Hongyi Wang, Felipe Maia Polo, Yuekai Sun, Souvik Kundu, Eric Xing 和 Mikhail Yurochkin. 2023. Fusing Models with Complementary Expertise. arXiv:2310.01542 [cs.LG]

+   Wang等人（2024）Xingyao Wang, Yangyi Chen, Lifan Yuan, Yizhe Zhang, Yunzhu Li, Hao Peng 和 Heng Ji. 2024. Executable Code Actions Elicit Better LLM Agents. arXiv:2402.01030 [cs.CL]

+   Warstadt等人（2023）Alex Warstadt, Leshem Choshen, Aaron Mueller, Adina Williams, Ethan Wilcox 和 Chengxu Zhuang. 2023. Call for Papers – The BabyLM Challenge: Sample-efficient pretraining on a developmentally plausible corpus. arXiv:2301.11796 [cs.CL]

+   Woodward等人（2023）Ben Woodward, eor123, Genevieve Patterson 和 Lilli Carlsen. 2023. FathomNet 2023. [https://kaggle.com/competitions/fathomnet-out-of-sample-detection](https://kaggle.com/competitions/fathomnet-out-of-sample-detection)

+   Wu等人（2023）Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Beibin Li, Erkang Zhu, Li Jiang, Xiaoyun Zhang, Shaokun Zhang, Jiale Liu, Ahmed Hassan Awadallah, Ryen W White, Doug Burger 和 Chi Wang. 2023. AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation. arXiv:2308.08155 [cs.AI]

+   Yeadon等人（2024）Will Yeadon, Alex Peach 和 Craig P. Testrow. 2024. A comparison of Human, GPT-3.5, and GPT-4 Performance in a University-Level Coding Course. arXiv:2403.16977 [cs.CL]

+   易等人（2024）易子豪、欧阳家瑞、刘宇文、廖天浩、徐哲和沈颖。2024。基于LLM的多轮对话系统的最新进展综述。arXiv:2402.18013 [cs.CL]

+   岳等人（2024）岳睿蒙、赵杰、张敏、杜梁和姚子瑜。2024。结合思维混合表示的大语言模型级联方法，用于高效推理。arXiv:2310.03094 [cs.CL]

+   Zan等人（2023）贾道光、陈贝、张风际、卢电杰、吴冰超、关贝、王永基和娄建光。2023。大语言模型与NL2Code的结合：一项综述。收录于*第61届计算语言学协会年会论文集（第1卷：长篇论文）*，由Anna Rogers、Jordan Boyd-Graber和Naoaki Okazaki主编。计算语言学协会， 加拿大多伦多，7443–7464。[https://doi.org/10.18653/v1/2023.acl-long.411](https://doi.org/10.18653/v1/2023.acl-long.411)

+   Zan等人（2024）贾道光、余艾伦、刘伟、陈东、沈博、李伟、姚亚芬、龚永顺、陈晓琳、关贝、杨志光、王永基、王千翔和崔丽珍。2024。CodeS：通过多层草图将自然语言转化为代码仓库。arXiv:2403.16443 [cs.CL] [https://arxiv.org/abs/2403.16443](https://arxiv.org/abs/2403.164.43)

+   张等人（2023）张杰宇、克里斯纳·兰杰、阿赫梅德·H·阿瓦达拉和王驰。2023。EcoAssistant：更经济、精准地使用LLM助手。arXiv:2310.03046 [cs.SE]

+   张等人（2024b）张可池、李佳、李革、史献杰和金智。2024b。CodeAgent：通过工具集成的代理系统提升代码生成能力，解决真实世界的代码挑战。arXiv:2401.07339 [cs.SE]

+   张等人（2024c）张磊、张宇歌、任侃、李东升和杨宇清。2024c。MLCopilot：释放大语言模型在解决机器学习任务中的潜力。arXiv:2304.14979 [cs.LG] [https://arxiv.org/abs/2304.14979](https://arxiv.org/abs/2304.14979)

+   张等人（2024a）张子尹、陈超宇、刘秉昌、廖聪、龚子、余杭、李建国和王瑞。2024a。统一自然语言处理与软件工程视角：关于代码的语言模型综述。arXiv:2311.07989 [cs.CL]

+   郑等人（2024）郑子彬、宁凯文、王彦霖、张敬文、郑德武、叶名熙和陈佳驰。2024。大语言模型在代码中的应用：发展、基准测试与未来趋势综述。arXiv:2311.10372 [cs.SE]

+   钟等人（2024）钟丽莉、王子龙和尚敬博。2024。LDB：通过逐步验证运行时执行来调试大语言模型。arXiv:2402.16906 [cs.SE]

+   朱等人（2023）朱文浩、刘洪毅、董庆修、徐晶晶、黄树建、孔令鹏、陈家俊和李雷。2023。大语言模型在多语言机器翻译中的应用：实证结果与分析。arXiv:2304.04675 [cs.CL]
