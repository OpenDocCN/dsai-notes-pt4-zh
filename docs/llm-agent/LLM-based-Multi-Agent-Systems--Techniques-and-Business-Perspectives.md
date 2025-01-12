<!--yml

类别：未分类

日期：2025-01-11 11:54:53

-->

# 基于LLM的多智能体系统：技术与商业视角

> 来源：[https://arxiv.org/html/2411.14033/](https://arxiv.org/html/2411.14033/)

Yingxuan Yang [zoeyyx@sjtu.edu.cn](mailto:zoeyyx@sjtu.edu.cn) 上海交通大学 上海 中国， Qiuying Peng [qypeng.ustc@gmail.com](mailto:qypeng.ustc@gmail.com) OPPO研究院 深圳 中国， Jun Wang [junwang.lu@gmail.com](mailto:junwang.lu@gmail.com) OPPO研究院 深圳 中国， Ying Wen [ying.wen@sjtu.edu.cn](mailto:ying.wen@sjtu.edu.cn) 上海交通大学 & SII 上海 中国， Weinan Zhang [wnzhang@sjtu.edu.cn](mailto:wnzhang@sjtu.edu.cn) 上海交通大学 & SII 上海 中国

###### 摘要。

在（多模态）大型语言模型的时代，大多数操作过程可以通过LLM智能体重新构建和再现。LLM智能体能够感知、控制并从环境中获得反馈，从而以自主的方式完成给定任务。除了环境交互属性外，LLM智能体还可以调用各种外部工具来简化任务完成过程。这些工具可以视为具有私人或实时知识的预定义操作过程，而这些知识并不存在于LLM的参数中。作为一种自然的发展趋势，工具调用逐渐成为自主智能体，从而整个智能系统变成了基于LLM的多智能体系统（LaMAS）。与之前的单一LLM智能体系统相比，LaMAS具有以下优势：i）动态任务分解与有机专业化，ii）更高的系统变更灵活性，iii）每个参与实体的数据隐私保护，iv）每个实体的货币化可行性。本文讨论了LaMAS的技术和商业前景。为了支持LaMAS的生态系统，我们提供了一个初步版本的LaMAS协议，考虑到技术要求、数据隐私和商业激励。因此，LaMAS将在不久的将来成为实现人工集体智能的实际解决方案。

基于LLM的多智能体系统、大型语言模型、数据隐私、货币化

## 1\. 背景与趋势

大型语言模型（LLMs）的发展（OpenAI 等，[2024](https://arxiv.org/html/2411.14033v2#bib.bib45)）标志着人工智能领域的一个重要进展。这些模型已经从简单的文本处理器转变为能够进行推理、理解多模态输入并做出自主决策的复杂系统（Wang 等，[2024](https://arxiv.org/html/2411.14033v2#bib.bib66)）。这种发展促使了由 LLM 支持的 AI 代理的出现¹¹1为了简洁起见，本文将多模态 LLM 概念（Caffagni 等，[2024](https://arxiv.org/html/2411.14033v2#bib.bib8)）合并为 LLM 概念。，这些代理能够适应各种任务、理解上下文并自主与环境互动（Zhang 等，[2024c](https://arxiv.org/html/2411.14033v2#bib.bib77); Zhou 等，[2024b](https://arxiv.org/html/2411.14033v2#bib.bib79)）。

LLM 能力的一个关键转折点是它们从仅仅响应命令的被动工具，转变为能够独立决策和采取行动的主动代理（Schick 等，[2023](https://arxiv.org/html/2411.14033v2#bib.bib55); Lin 等，[2024](https://arxiv.org/html/2411.14033v2#bib.bib37)）。最初，LLM 主要用于单一任务，例如文本生成或分析。然而，最近的进展使它们能够与图形用户界面（GUIs）互动，并执行诸如网页浏览、应用导航和系统控制等复杂操作（Zhang 等，[2024b](https://arxiv.org/html/2411.14033v2#bib.bib76)）。在这些能力之外，现代 LLM 已经转变为自主代理，能够根据上下文需求动态选择并使用工具。这一演变突显了它们的双重性质：它们不仅能使用工具，还可以作为模块化系统中的工具发挥作用，从而促进了多代理体系结构的创建，在这种体系结构中，代理之间相互协作，共同解决复杂问题。

基于 LLM 的多代理系统（LaMAS）的兴起标志着 AI 应用的一个重大飞跃（Chen 等，[2024](https://arxiv.org/html/2411.14033v2#bib.bib12); Liu 等，[2024](https://arxiv.org/html/2411.14033v2#bib.bib39)）。虽然与单代理方法相比，这些系统可能需要更多的计算资源，但它们提供了可以证明这种权衡是合理的关键优势：通过代理冗余提供内在的容错性、无需显式工作流设计即可自然地进行任务分解，以及在复杂问题解决中的有机专业化。当一个代理在多代理系统中失败时，其他代理可以无缝地继续操作，提供了集中式系统无法比拟的强大可靠性。此外，单代理系统需要为每个任务类型精心编排执行工作流，而多代理系统则自然而然地形成协作专门化模式，使得每个代理能够在更大的系统架构中专注于其核心能力。

认识到这些好处，研究人员已经开发了LaMAS框架，以便实现复杂任务的协作（Guo等人， [2024](https://arxiv.org/html/2411.14033v2#bib.bib24)；Chen等人， [2024](https://arxiv.org/html/2411.14033v2#bib.bib12)；Liu等人， [2024](https://arxiv.org/html/2411.14033v2#bib.bib39)；Fourney等人， [2024](https://arxiv.org/html/2411.14033v2#bib.bib19)；Ghafarollahi和Buehler， [2024](https://arxiv.org/html/2411.14033v2#bib.bib22)；Chen等人， [2023](https://arxiv.org/html/2411.14033v2#bib.bib11)；Qian等人， [2023](https://arxiv.org/html/2411.14033v2#bib.bib49)；Hong等人， [2024](https://arxiv.org/html/2411.14033v2#bib.bib27)；Wu等人， [2023](https://arxiv.org/html/2411.14033v2#bib.bib70)；Xie等人， [2023](https://arxiv.org/html/2411.14033v2#bib.bib72)；Li等人， [2023](https://arxiv.org/html/2411.14033v2#bib.bib35)；Gao等人， [2024](https://arxiv.org/html/2411.14033v2#bib.bib21)；Zhuge等人， [n.d.](https://arxiv.org/html/2411.14033v2#bib.bib81)；Trirat等人， [2024](https://arxiv.org/html/2411.14033v2#bib.bib64)；Fu等人， [2024](https://arxiv.org/html/2411.14033v2#bib.bib20)；Li等人， [2024](https://arxiv.org/html/2411.14033v2#bib.bib33)；Chan等人， [2023](https://arxiv.org/html/2411.14033v2#bib.bib9)）。超越传统的SaaS、PaaS和IaaS等范式，LaMAS通过将智能代理无缝集成到云生态系统中，提出了一种全新的方法。该框架支持专用代理的部署，这些代理能够进行协作，同时保持数据隐私和安全性。此外，它还建立了一个代理变现市场，允许用户根据需求定制和组合代理服务。系统架构强调模块化设计、标准化通信协议和强大的安全措施，促进可持续创新。

通过变现机制激励。就像互联网应用高度激励连接到互联网一样，LaMAS中的代理也会基于变现机制设计而获得高度激励。首先，从与LaMAS内的交互中生成的经验数据对训练高效能代理至关重要。在LaMAS中，代理接收来自上游代理的任务指令，进行内部代理推理和工具使用，将任务指令发送给下游代理并获取其返回的信息，最终获得任务完成的结果。这种经验数据比单一代理仅连接到用户时生成的数据更为宝贵且数据量更大。其次，类似于互联网通过在线广告进行变现，LaMAS也将存在一种变现机制。具体而言，对于每个分配有商业价值的完成任务，例如用户预订酒店或购买商品，将有商家提供推广费用给LaMAS中参与的代理团队，且可以通过任务完成的参与或贡献建立信用分配机制。因此，代理背后的实体有动力构建一个高度智能的代理以连接到LaMAS。

基于代理智能的实体责任。在电缆或4/5G互联网中，互联网服务背后的每个实体，例如公司、机构或团队，负责维护服务的稳定运行和与互联网的连接。如果其服务器崩溃或连接中断，依赖该服务的其他服务将受到很大影响，因此该实体应对其产生的影响负责。类比地，在LaMAS生态系统中，每个代理背后的实体有责任确保生态系统的顺利和智能运行。首先，继承自互联网服务，每个代理需要支持稳定的功能运行和与互联网的连接。其次，更为重要的是，代理提供的智能必须符合或超过预定标准，因为代理提供的低智能可能导致整个LaMAS在完成智能任务时的功能不足。

本文通过讨论LaMAS的技术和商业领域，展示了我们对LaMAS的看法。我们将展示关键的AI技术方面，包括系统架构、协作协议、代理训练方法，以及商业方面，包括数据隐私保护和通过流量与智能的变现。通过这些分析，可以预期LaMAS将在未来几年内形成一种新的技术商业范式。

## 2. 关键AI技术方面

### 2.1. LLM代理的架构

基于大语言模型（LLM）的 AI 代理架构由多个相互关联的组件组成，这些组件对自主操作和智能互动至关重要。该架构的核心旨在有效处理输入，保持上下文相关性，做出明智决策，并生成适当的回应。

交互封装器作为代理与其环境及其他代理互动的主要接口。该组件管理进出通信的流动，适应各种输入方式，并将其标准化以供内部处理。交互封装器实现协议特定的适配，以确保与各种通信标准的无缝集成。这种方法保持了代理操作的内部一致性。

内存管理对于架构至关重要。它包括短期工作记忆和长期情节存储。短期记忆缓冲区保留即时上下文和最近的互动，促进对话的连贯性。与此同时，长期记忆系统存档重要的经验和学习的模式，使代理能够根据历史互动调整回应，并增强在情境丰富的场景中的决策能力。

当前架构的认知功能依赖于链式思维（Chain-of-Thought, CoT）推理（Wei 等，[2023](https://arxiv.org/html/2411.14033v2#bib.bib67)；Yao 等，[2023](https://arxiv.org/html/2411.14033v2#bib.bib75)）。这种结构化的推理框架将复杂任务分解为可管理的逻辑步骤，从而促进问题解决的清晰性和全面性。CoT 机制使代理能够阐明中间推理状态，验证逻辑一致性，并通过对推理过程的系统分析进行自我纠正。

此外，为了增强代理在自然语言处理之外的操作能力，工具集成框架是必需的（Schick 等，[2023](https://arxiv.org/html/2411.14033v2#bib.bib55)；Patil 等，[2023](https://arxiv.org/html/2411.14033v2#bib.bib46)；Lin 等，[2024](https://arxiv.org/html/2411.14033v2#bib.bib37)）。该子系统发现并注册工具，建立自然语言命令与工具 API 之间的参数映射，监控执行，处理错误，并解释结果。它确保外部功能有效集成到决策过程中。

架构还具有复杂的路由机制，管理与邻近代理的连接。该组件在促进动态邻居发现、做出基于能力的路由决策、平衡代理网络负载和实施基于策略的访问控制方面发挥重要作用。这些网络功能对于确保多代理系统内高效的通信与协作至关重要。

此外，该架构还包含反馈回路，使得持续学习和适应成为可能。这些回路有助于处理交互结果，使智能体能够根据经验学习更新其内部模型，并改进决策策略。整合这些架构元素不仅为LaMAS的自主运行奠定了坚实基础，还显著提升了它们在多智能体系统中的协作能力。

![参见说明文字](img/befde2e249bf5d2e436a19baf49fbc96.png)

图1\. LaMAS示意图。

### 2.2\. LaMAS的机制与架构

作为一个多智能体系统，LaMAS的机制和架构设计对于其成功至关重要。大致来说，根据MAS的协调形式，LaMAS有三种主要架构。

首先，对于完全中心化的架构，整个系统完全控制所参与的智能体，这是一项非常高的要求，然后可以使用集中式训练和集中式执行方法，智能体将高度协调地行动。实际上，只有在智能体是开发在类操作系统平台上的应用，并且授予数据和控制访问权限时，这种方法才能应用。

其次，对于具有全球信用分配的去中心化架构，整个系统无法完全控制所参与的智能体，但可以为每个完成的任务分配信用，然后可以采用集中式训练和集中式执行方法。这种方式更具实用性，因为每个智能体（及其背后的实体）不必向平台提供数据或控制访问。此外，平台仍然可以通过向团队分配信用，激励所参与的智能体提高其协作表现。

第三，对于完全去中心化的架构，即每个参与智能体无法访问数据或控制权限，也没有平台的信用分配，系统中的智能体将需要找到与他人协作并自我提升的方式。在这种情况下，机制设计从一开始就变得尤为重要。

### 2.3\. 智能体交互协议

基于LLM的多智能体系统（LaMAS）框架需要复杂的交互协议，以促进有效的智能体协作。这些协议必须弥合传统结构化格式与自然语言理解之间的差距，解决基于LLM的智能体在概率决策和新兴能力方面所带来的独特挑战（Marro et al., [2024](https://arxiv.org/html/2411.14033v2#bib.bib42)）。

核心挑战与关键问题。LaMAS协议的开发面临协议有效性度量、行为多样性优化和非传递性交互管理等几个基本挑战。

![[未标注图片]](img/e5a6d06325b5e839d43d592f4096bffb.png)

从这些挑战中，我们识别出协议设计中的 3 个关键问题。首先，LaMAS 需要一个分层协议架构来有效管理多样化的代理交互，支持基于任务和代理能力的动态协议选择（Wooldridge，[2009](https://arxiv.org/html/2411.14033v2#bib.bib69)）。其次，随着系统规模的扩大，传统协议在管理通信开销和保持一致性方面面临限制，需要创新的协议设计方法（Shoham 和 Leyton-Brown，[2009](https://arxiv.org/html/2411.14033v2#bib.bib59)）。第三，LaMAS 利用大语言模型（LLM）代理在语言理解和上下文解释方面的优势。协议应当利用这些能力，例如处理模糊指令或启用实时协商以澄清信息（Marro 等，[2024](https://arxiv.org/html/2411.14033v2#bib.bib42)）。

核心协议框架。我们提出了一个由五个基本组件组成的全面协议框架：指令处理协议、消息交换协议、共识形成协议、信用分配协议和经验管理协议。

![参见标题](img/e59f6795cc1477344dcb8dbbb1c53068.png)

图 2\. 协议层次结构。

指令处理协议通过结构化解析机制和上下文感知处理管道标准化用户指令的解释（Aziz，[2010](https://arxiv.org/html/2411.14033v2#bib.bib4)）。该协议实施复杂的消歧技术来处理不确定或不完整的指令，并在多轮交互中保持一致性。

消息交换协议通过标准化的消息格式和自适应传输机制为代理间通信奠定了基础（Shoham 和 Leyton-Brown，[2009](https://arxiv.org/html/2411.14033v2#bib.bib59)；Zhou 等，[2024a](https://arxiv.org/html/2411.14033v2#bib.bib80)）。该协议根据任务需求和系统负载动态切换同步和异步模式，实施基于优先级的路由算法，以在不同条件下优化消息传递。

共识形成协议通过投票系统和协商框架的结合，实现分布式决策机制（Amirkhani 和 Barshooi，[2022](https://arxiv.org/html/2411.14033v2#bib.bib3)）。该协议根据任务的重要性和系统状态动态调整共识阈值，确保在保持系统响应性的同时做出稳健的决策。当提案发生冲突时，代理可以通过协商解决分歧，确保尽管存在意见差异，仍能推进进程。投票协议允许代理表达偏好，并在无法达成完全共识时做出决策，从而避免死锁并确保任务的持续推进。

信用分配协议通过多层传播机制解决了公平贡献评估的挑战。参与任务的代理根据其贡献获得相应的信用（Zhou 等人， [2024a](https://arxiv.org/html/2411.14033v2#bib.bib80)）。通过实施特定任务的指标和基于表现的分配算法，该协议确保了公平的奖励分配，同时激励了协作行为。

经验管理协议通过结构化日志记录和模式提取机制促进集体学习（Zhang 等人， [2024a](https://arxiv.org/html/2411.14033v2#bib.bib78)）。每个代理在任务执行过程中记录其经验和学习成果。这些经验记录可能包括成功、失败、使用策略的效果以及与其他代理的互动。该协议实施了跨代理的知识共享算法，通过积累经验系统地提高系统性能。

LaMAS的有效性依赖于这些协议的无缝集成，如图[2](https://arxiv.org/html/2411.14033v2#S2.F2 "图 2 ‣ 2.3\. 代理交互协议 ‣ 2\. 关键AI技术方面 ‣ 基于LLM的多代理系统：技术与商业视角")所示。层次化的组织结构实现了动态协议选择和高效资源利用，同时保持了系统的可扩展性。

### 2.4\. 代理训练方法

在LaMAS中，每个代理都有动机自我提升，以便从平台上获得更多的信用。在“代理训练”方面，我们指的是提高代理性能的方法，包括无需调优的方法和参数调优方法。

无需调优的方法。在大型语言模型（LLM）代理领域，无需调优的方法是通过不修改模型参数来提高性能的策略。当直接调优参数代价高昂或不可行时，这些方法尤其有用。关键的无需调优方法包括：

+   •

    提示工程：提示工程涉及设计特定的输入提示，以引导LLM代理生成期望的响应。通过精心构建提示，代理可以在无需任何参数调整的情况下，更好地与任务要求对齐（Brown 等人， [2020](https://arxiv.org/html/2411.14033v2#bib.bib5)）。

+   •

    少样本学习：少样本学习在提示中提供有限的示例，帮助代理理解新任务。在零样本学习中，模型仅通过自然语言指令处理任务，利用预训练的知识。这两种方法都能够在多代理环境中提供灵活性和适应性（Radford 等人， [2021](https://arxiv.org/html/2411.14033v2#bib.bib50); Yao 等人， [2023](https://arxiv.org/html/2411.14033v2#bib.bib75); Zhou 等人， [2024b](https://arxiv.org/html/2411.14033v2#bib.bib79)）。

+   •

    外部工具的使用：代理可以通过与外部工具或API的交互来增强其能力。使用外部资源，如数据库或计算器，使代理能够执行复杂任务，而无需额外的模型训练（Schick et al., [2023](https://arxiv.org/html/2411.14033v2#bib.bib55); Patil et al., [2023](https://arxiv.org/html/2411.14033v2#bib.bib46)）。

这些无需调优的方法在LaMAS中尤为重要。它们使代理能够快速适应并在动态环境中共同完成复杂任务，支持在最小计算成本下的顺畅协作。

参数调优方法。为了直接调节每个代理背后LLM的参数，可以使用对齐方法和多智能体强化学习方法。首先，调节LLM的对齐方法通常基于监督学习损失，基于目标输出或专家的偏好。直接拟合专家输出对应于代理模仿学习中的行为克隆方法（Pomerleau, [1991](https://arxiv.org/html/2411.14033v2#bib.bib48)），而在专家偏好对上训练则以学习排序的方式改善代理的策略（Rafailov et al., [2024](https://arxiv.org/html/2411.14033v2#bib.bib51)）。这种方法在多智能体任务中尚未得到广泛应用，因为在这种场景下，对齐目标尚未明确制定。其次，多智能体强化学习（MARL）是训练多智能体系统中代理策略的关键方法，它将任务形式化为一个多智能体的序列决策问题（Busoniu et al., [2008](https://arxiv.org/html/2411.14033v2#bib.bib7)）。在这里，我们主要考虑合作性MARL方法，因为通常情况下，代理是为了团队成功而组织的，即完成用户的任务。根据代理在训练和执行中的协调方式，MARL方法可以分为三大类，即i) 集中训练与集中执行（Sukhbaatar et al., [2016](https://arxiv.org/html/2411.14033v2#bib.bib61); Wen et al., [2022](https://arxiv.org/html/2411.14033v2#bib.bib68)），ii) 集中训练与分散执行（Lowe et al., [2017](https://arxiv.org/html/2411.14033v2#bib.bib41); Rashid et al., [2020](https://arxiv.org/html/2411.14033v2#bib.bib53)），以及iii) 分散训练与执行（Tan, [1993](https://arxiv.org/html/2411.14033v2#bib.bib62); Tian et al., [2019](https://arxiv.org/html/2411.14033v2#bib.bib63)）。

### 2.5. LaMAS中的攻击与防御

由于LaMAS系统处理敏感数据和关键操作，其安全性是一个重要问题。LaMAS的分布式特性带来了超越单一LLM系统的独特漏洞。恶意攻击者不仅可以针对单个代理，还可以利用代理间的通信和集体决策过程进行攻击。本节将探讨对LaMAS的攻击及其防御机制，重点关注它们在多代理系统中的影响。

攻击面与漏洞。LaMAS面临三种主要的攻击类型。

第一，提示注入攻击通过操控输入提示欺骗模型生成有害响应。这些攻击在LaMAS中尤为危险，因为受损的代理可以在系统中传播恶意提示。最近的研究（Zou等，[2023](https://arxiv.org/html/2411.14033v2#bib.bib82)）显示，输入措辞的轻微变化可以绕过防御，而另一项研究（Liu等，[2023](https://arxiv.org/html/2411.14033v2#bib.bib40)）展示了如何利用转义字符和上下文省略来修改系统提示。

第二，内存和数据投毒攻击针对代理用于决策的知识库。在LaMAS中，投毒数据可以同时影响多个代理。研究表明，检索增强生成（RAG）系统中被污染的知识库可能会导致整个代理网络中出现级联错误（Xue等，[2024](https://arxiv.org/html/2411.14033v2#bib.bib73)）。另有研究强调，特定触发器的有毒训练样本可能会损害经过微调的代理，从而影响系统的可靠性（Yan等，[2024](https://arxiv.org/html/2411.14033v2#bib.bib74)）。

第三，模型反演和提取攻击旨在通过有针对性的查询重建训练数据或提取模型细节。分析表明，这些攻击在LaMAS中特别有效，攻击者通过从多个代理处获取响应来提高提取效率（Morris等，[2023](https://arxiv.org/html/2411.14033v2#bib.bib43)）。对于处理敏感个人或商业数据的系统，数据泄漏的风险尤其高，正如最近关于基于提示的漏洞的研究所示（Sha和Zhang，[2024](https://arxiv.org/html/2411.14033v2#bib.bib57)）。

防御机制与未来方向。已经提出了几种防御策略来应对这些攻击，每种策略都针对LaMAS环境中的特定漏洞。

输入清理技术，如提示随机化和查询封装，有助于中和提示注入攻击。一项研究展示了这些方法在LaMAS环境中的有效性，尽管它们可能会引入通信开销（Robey等，[2023](https://arxiv.org/html/2411.14033v2#bib.bib54)）。一种改进方法涉及自适应分隔符策略，能够保持通信效率（Hines等，[2024](https://arxiv.org/html/2411.14033v2#bib.bib26)）。

基于困惑度的过滤是另一种有前景的防御方法。多项研究表明，监控模型的困惑度可以在不妥协模型效用的情况下检测对抗性提示（Alon 和 Kamfonas，[2023](https://arxiv.org/html/2411.14033v2#bib.bib2); Jain 等人，[2023](https://arxiv.org/html/2411.14033v2#bib.bib29)）。在 LaMAS 中，若通过智能体间交叉验证困惑度得分来增强这一方法，尽管需要精心校准，以避免在合法交互中产生误报。

对抗性稳健的微调在增强 LaMAS 安全性方面也已证明有效。采用双模型方法，在训练过程中生成并验证对抗样本，能带来显著的好处（O'Neill 等人，[2023](https://arxiv.org/html/2411.14033v2#bib.bib44)）。进一步优化重点是平衡稳健性与效用，这些技术对于 LaMAS 特别有价值，因为它们使得系统范围的应用得以实现，同时保持智能体的专门化（Xhonneux 等人，[2024](https://arxiv.org/html/2411.14033v2#bib.bib71)）。

尽管取得了一些进展，但在保障 LaMAS 安全方面仍然存在挑战。目前的防御措施通常难以应对智能体交互的动态特性，其中复杂的通信模式可能会触发误报。此外，全面安全措施的计算开销可能会影响系统性能，因此需要在安全性和效率之间找到平衡。

展望未来，有几个研究方向至关重要。首先，需要为 LaMAS 制定标准化的安全评估框架，考虑个别智能体的漏洞和系统范围的风险。其次，开发轻量级的安全措施，以保持通信效率，是一个开放的挑战。最后，能够随着新兴威胁演变的自适应防御机制，对于长期安全至关重要。

为确保 LaMAS 的安全部署，未来的研究应聚焦于一种综合方法，结合稳健的模型架构、有效的训练程序和动态防御机制。这对于在 LaMAS 系统持续增长其复杂性和影响力的同时，维护公众信任至关重要。

## 3. 关键商业方面

基于我们对 LaMAS 的研究，我们展示了其在三个关键维度上的商业影响：隐私保护、流量变现和智能变现。我们预测 LaMAS 将如何重塑商业范式，并探索其商业应用的潜在轨迹。

### 3.1. LaMAS 中的隐私保护

LaMAS的兴起引入了新的隐私挑战，这些挑战超越了传统多智能体系统中的问题。与传统智能体交换结构化数据不同，LLM智能体处理丰富的上下文信息，这些信息可能包含通过自然语言对话、推理过程和知识表示中隐含的敏感数据。LaMAS中的隐私保护至关重要，因为这些系统处理自然语言数据，这些数据可能通过语义连接和隐性知识表示无意间泄露敏感信息。

隐私保护挑战。我们识别了三个层次的隐私问题，需要系统性分析和创新解决方案：

+   •

    在语义层面，挑战在于LLM的自然语言处理，这可能通过上下文关联和语义连接无意中泄露敏感信息。传统的隐私保护机制是为结构化数据设计的，难以处理如此复杂的信息，尤其是针对那些利用语义漏洞进行攻击的情况。

+   •

    在智能体交互层面，智能体之间持续的信息交换带来了隐私风险。敏感信息不仅通过直接内容泄露，还可能通过行为模式和反应特征暴露。此外，在每个智能体中保持对话历史和上下文窗口会随着时间推移产生持续的漏洞。

+   •

    在系统架构层面，LaMAS的分布式特性使得在保持系统效率的同时，难以在所有组件中实施隐私保护措施。智能体之间的动态交互及其不断发展的知识进一步增加了实现强大隐私保护的挑战。

隐私保护技术。最近的研究探讨了多种技术来应对隐私挑战，从密码学技术到系统级解决方案。

在基础层面，同态加密（HE）（Cheon et al., [2017](https://arxiv.org/html/2411.14033v2#bib.bib13)）使得在加密数据上进行安全计算成为可能，支持私密的智能体间通信和推理。虽然在隐私保护的机器学习（Gilad-Bachrach et al., [2016](https://arxiv.org/html/2411.14033v2#bib.bib23)）和安全数据共享（Chen et al., [2017](https://arxiv.org/html/2411.14033v2#bib.bib10)）中具有潜力，但HE的计算复杂度仍然是LaMAS中的一个重大挑战。

安全多方计算（SMPC）（Lindell, [2020](https://arxiv.org/html/2411.14033v2#bib.bib38)）使多个智能体之间能够进行安全的协同计算。SMPC已被用于传统多智能体系统中的隐私保护数据分析（Li and Xu, [2019](https://arxiv.org/html/2411.14033v2#bib.bib34)）和协作学习（Knott et al., [2021](https://arxiv.org/html/2411.14033v2#bib.bib31)），但将这一技术扩展到大规模LaMAS系统仍然是一个未解决的问题。

对于系统级保护，可信执行环境（TEEs）（Costan, [2016](https://arxiv.org/html/2411.14033v2#bib.bib15)）提供了基于硬件的安全保障。像英特尔SGX（Costan, [2016](https://arxiv.org/html/2411.14033v2#bib.bib15)）、ARM TrustZone（Pinto和Santos, [2019](https://arxiv.org/html/2411.14033v2#bib.bib47)）和AMD SEV（Sev-Snp, [2020](https://arxiv.org/html/2411.14033v2#bib.bib56)）等技术为敏感计算创建了安全区域。然而，将TEEs集成到LaMAS中时需要仔细考虑安全性和性能的权衡。

此外，差分隐私（DP）（Dwork, [2006](https://arxiv.org/html/2411.14033v2#bib.bib16)）提供了用于隐私保护数据分析的数学方法。虽然在协作任务中有效保护敏感信息，但差分隐私在自然语言处理领域面临独特的挑战，如管理隐私预算和保持效用。

研究方向与开放挑战。推进LaMAS中的隐私保护涉及解决若干关键问题。首先，现有的隐私度量指标无法充分捕捉自然语言处理中的语义信息泄露的复杂性。需要针对LaMAS的隐私框架，考虑语义空间中直接和间接信息流的影响。其次，将隐私保护技术整合到LaMAS中需要一种统一的方案，结合数据保护、安全计算和通信安全。性能优化和可扩展性仍然是主要障碍，特别是在代理网络扩展时，保持隐私同时实现高效合作至关重要。未来的研究应专注于创建量身定制的综合隐私框架，包括标准化隐私协议、开发高效的实现方案，并建立评估指标以确保隐私保护的LaMAS系统能在实践中得到有效部署。

### 3.2．流量变现

LaMAS中的流量变现涉及通过管理用户流量和优化广告来创造商业价值，利用各个代理的优势。这包括改善流量流动、提高点击率（CTR）和增加转化率（CVR）（Kumar和Reinartz, [2016](https://arxiv.org/html/2411.14033v2#bib.bib32)）。LaMAS利用每个代理的能力来增强用户参与度并使广告策略更加有效。该系统还确保了公平和透明的收入分配。本节将探讨LaMAS中的流量变现，涵盖商业场景、收入模型、利润分配和应用代理的角色。

![参考说明](img/957b894e2f7559decb72b0e2ed889bfd.png)

图3．流量变现。

商业场景与收入生成。在LaMAS中，代理分析用户行为和偏好，以优化流量管理并投放定向广告。通过实时数据分析，代理识别最相关的广告来吸引用户，从而提高点击率（CTR）和转化率（CVR）。这种方法构建了用户档案，并利用智能推荐系统个性化广告，提升参与度并推动购买。LaMAS的收入主要来源于广告，使用每次点击成本（CPC）和每次行动成本（CPA）模型（Kannan和Li，[2017](https://arxiv.org/html/2411.14033v2#bib.bib30)）。在CPC模型中，广告主按点击次数支付费用，代理根据其在流量管理和广告效果方面的贡献赚取佣金。在CPA模型中，为完成的购买支付费用，奖励推动转化的代理，给予他们更大份额的收入。除了广告收入外，还可以通过用户订阅某些应用内的高级功能或个性化服务来产生收入，如高级分析仪表盘或专有工具的独占访问权限。这些变现策略相辅相成，代理通过优化流量流动、用户参与度和广告定向，推动整体盈利。

利润分配机制。将收入转化为在各代理之间公平分配的利润是流量变现的关键。该过程从评估每个代理对流量生成、广告点击和转化的贡献开始，使用诸如CTR（点击率）和CVR（转化率）等指标来量化个人影响。为了确保公平，LaMAS可能会使用基于区块链的智能合约来自动化分配过程，最小化偏差和人为错误。此外，一个评分系统会根据代理的表现对其进行评分，包括用户反馈和参与度等因素。得分较高的代理将获得更大份额的收入，从而激励更好的表现和持续优化。归因方法，如Shapley值，确保利润根据每个代理对系统的贡献进行分配（Shapley，[1953](https://arxiv.org/html/2411.14033v2#bib.bib58)）。动态调整机制允许根据代理表现和市场状况实时更新收入份额，确保对优化流量和广告的代理提供公平的报酬。结合如CPM（每千次展示成本）等指标，为收入分配提供更细致的方法，超越单纯的点击和转化，深入了解广告表现（Brynjolfsson和McAfee，[2014](https://arxiv.org/html/2411.14033v2#bib.bib6)）。

应用智能体的角色。不同类型的应用智能体在流量货币化中扮演着各自独特但相互关联的角色。广告智能体负责管理和投放广告，利用数据分析优化广告表现并增加用户参与度。它们选择最佳广告位置，并根据实时数据和用户行为调整策略。数据分析智能体分析用户行为，提供帮助广告智能体完善广告策略和改善内容的见解。这些智能体帮助创建准确的用户画像，并识别新兴趋势，从而提高广告的有效性。交易智能体处理用户购买，确保交易顺畅并跟踪转化率。它们将销售表现与特定广告关联，帮助广告主改进未来的广告活动并提高转化率（CVR）。订阅智能体管理高级服务，提供个性化功能并贡献额外的收入来源。这些智能体还通过提升用户留存和忠诚度，支持长期的收入增长。

LaMAS中的流量货币化创造了一个系统，通过不同智能体的协作来生成并共享收入。通过改善流量管理、优化广告和确保公平分配，LaMAS确保所有智能体都能公平受益。未来的研究应专注于改进归因模型，如Shapley值，以使利润分配更加公平。加入像CPM这样的度量指标也将有助于更准确地分配收入并改进货币化策略。

### 3.3\. 智能货币化

基于大语言模型（LLM）的多智能体系统中的智能货币化代表了AI商业化的重要发展，通过利用专业化智能体的协作能力。与传统的单模型范式不同，多智能体系统使得各个专门设计的智能体之间可以动态互动，从而促进了更加多样化和强大的智能解决方案的创建。这个范式在各种应用中表现出了潜力，微软于2024年11月19日推出的Copilot Studio平台就是一个例子。该平台支持超过1800个大规模模型的生态系统，提供开放的API和集成工具，使企业能够将智能体技术整合到工作流程和应用中，以提高定制化和可扩展性（VentureBeat，[2024](https://arxiv.org/html/2411.14033v2#bib.bib65)）。

通过数据驱动服务生成收入。在LaMAS的智能货币化中，一个关键的收入模式是数据驱动服务的销售（Sjödin等，[2021](https://arxiv.org/html/2411.14033v2#bib.bib60)；Faroukhi等，[2020](https://arxiv.org/html/2411.14033v2#bib.bib18)；Hajipour等，[2023](https://arxiv.org/html/2411.14033v2#bib.bib25)；VentureBeat，[2024](https://arxiv.org/html/2411.14033v2#bib.bib65)）。“专用代理”分析不同的数据集——例如消费者偏好、产品使用情况和市场趋势——以生成可操作的洞察。这些洞察以报告、预测或量身定制的建议的形式交付，企业可以购买这些服务。例如，一个代理可能提供个性化的用户行为洞察，以提升营销效果，而另一个代理则提供市场趋势分析，以指导战略规划。通过整合专用代理，LaMAS提供了广泛的洞察，这些洞察可以通过订阅模式或一次性报告来实现货币化。在实践中，像OpenAI的GPT-4 API（OpenAI等，[2024](https://arxiv.org/html/2411.14033v2#bib.bib45)）这样的成功实现，展示了多个专用模型如何协同工作，不同的代理处理智能管道的不同方面。这些方面包括数据处理和预处理、深度模式识别和洞察生成、推荐转化以及平台集成等专用代理。

创新的许可和代理市场。LaMAS还引入了超越传统软件许可的新型许可方式。其中最突出的方式是“代理即服务”（AaaS）（Rajaei，[2024](https://arxiv.org/html/2411.14033v2#bib.bib52)），如Google Cloud的AutoML所示，它基于计算需求动态部署代理，采用基于使用量的定价和自动扩展（Cloud，[2024](https://arxiv.org/html/2411.14033v2#bib.bib14)）。与此相辅相成的是代理市场平台的出现，这些平台为第三方代理的开发和部署创造了生态系统，如Hugging Face的模型中心，适用于LLMs的部署（Face，[2024](https://arxiv.org/html/2411.14033v2#bib.bib17)）。此外，混合部署架构将越来越流行，结合本地代理部署以应对敏感操作，以及基于云的代理以处理可扩展任务，正如IBM的Watson服务所展示的那样，Watson服务使用了分布式代理架构（IBM，[2024](https://arxiv.org/html/2411.14033v2#bib.bib28)）。

LaMAS中的智能变现通过利用专门化智能体的集体智慧提供了一个可持续的收入框架。展望未来，随着对AI驱动洞察需求的增长，LaMAS在提供可扩展、可操作的智能方面的作用将继续扩大。未来的发展应专注于增强智能体的协作，以便实时获取见解，并适应新兴的商业模式和行业，提供量身定制的解决方案以满足特定需求。

### 3.4. 三个商业方面的整合

LaMAS的三个关键商业方面形成了一个相互关联的框架，推动了商业成功与伦理运营。数据隐私确保了信任与合规，同时允许安全的数据使用。这一基础支持通过生成用户参与数据来进行流量变现，同时遵守隐私法规。智能变现则将这些保护隐私的互动转化为可操作的见解和服务。随着LaMAS的发展，在隐私、商业成功和技术进步之间保持平衡将是实现长期可持续性的关键。未来的发展应专注于加强这些联系，同时适应不断变化的隐私法规和市场需求。

![请参阅图注](img/59d5022557f9bef6a7a98cf37c386b96.png)

图4. LaMAS的架构。

## 4. 案例研究

基于LaMAS的技术基础和商业考量，我们将深入探讨实际部署案例，展示这些理论框架如何在实践中得以实现。通过精心挑选的案例研究，我们探索了不同的架构选择如何影响系统效率、数据隐私和盈利能力。这些例子不仅验证了我们的理论见解，也揭示了部署LaMAS解决方案过程中固有的挑战与机遇。

### 4.1. LaMAS中的架构

LaMAS在实际应用中的实现揭示了各种架构模式，每种模式都是为了应对特定的操作需求和约束而设计的。

![请参阅图注](img/92289e6ec0f27c6d870977f0c6676167.png)

图5. LaMAS的集中式架构。

![请参阅图注](img/9ac863bc8fabe4f0228a4c4fcd797d1b.png)

图6. LaMAS的集中式数据处理。

在分析当前LaMAS的部署情况时，我们观察到架构选择显著影响了系统的能力，从隐私保护到运营效率。图[4](https://arxiv.org/html/2411.14033v2#S3.F4 "图4 ‣ 3.4. 三个商业方面的整合 ‣ 3. 关键商业方面 ‣ 基于LLM的多智能体系统：技术与商业视角")展示了在实践中出现的四种基本模式：

+   •

    星型架构：在此结构中，中央代理协调与所有其他代理的通信（Qian et al., [2023](https://arxiv.org/html/2411.14033v2#bib.bib49); Hong et al., [2024](https://arxiv.org/html/2411.14033v2#bib.bib27); Wu et al., [2023](https://arxiv.org/html/2411.14033v2#bib.bib70); Xie et al., [2023](https://arxiv.org/html/2411.14033v2#bib.bib72); Fu et al., [2024](https://arxiv.org/html/2411.14033v2#bib.bib20)）。当一个代理负责任务分发和整体协调时，这种集中控制模型表现良好。

+   •

    环形架构：代理按圆形配置排列，每个代理与其前驱和后继代理进行通信（Chan et al., [2023](https://arxiv.org/html/2411.14033v2#bib.bib9); Liang et al., [2024](https://arxiv.org/html/2411.14033v2#bib.bib36)）。这种去中心化结构支持顺序任务处理，确保每个代理在任务流程中扮演特定角色。

+   •

    图形架构：该网络允许完全互联的系统，每个代理可以直接与任何其他代理进行通信（Zhuge et al., [[n.d.]](https://arxiv.org/html/2411.14033v2#bib.bib81); Chen et al., [2023](https://arxiv.org/html/2411.14033v2#bib.bib11)）。这种架构创建了一个完全或部分互联的网络，其中每个代理可以与其邻居进行通信。它提供了最大的灵活性和冗余性，允许多个通信路径支持复杂的交互。

+   •

    总线架构：该结构使用固定的工作流程或标准操作程序（SOP），任务被发送到一个中央总线，然后由总线将任务分发给适当的代理或过程（Li et al., [2023](https://arxiv.org/html/2411.14033v2#bib.bib35); Gao et al., [2024](https://arxiv.org/html/2411.14033v2#bib.bib21); Trirat et al., [2024](https://arxiv.org/html/2411.14033v2#bib.bib64); Li et al., [2024](https://arxiv.org/html/2411.14033v2#bib.bib33)）。总线确保了清晰的输入输出机制和任务的结构化流动，按顺序执行。

### 4.2\. LaMAS中的去中心化星型架构

图[5](https://arxiv.org/html/2411.14033v2#S4.F5 "Figure 5 ‣ 4.1\. Architectures in LaMAS ‣ 4\. Case Study ‣ LLM-based Multi-Agent Systems: Techniques and Business Perspectives")和图[6](https://arxiv.org/html/2411.14033v2#S4.F6 "Figure 6 ‣ 4.1\. Architectures in LaMAS ‣ 4\. Case Study ‣ LLM-based Multi-Agent Systems: Techniques and Business Perspectives")展示了我们在音乐服务场景中实施LaMAS的第一个案例研究。该系统采用集中式架构，其中包括个人代理、协调代理和歌曲代理等多个代理，共同处理用户的音乐播放请求。在这种设置中，协调代理充当中央枢纽，管理代理之间的通信并协调任务。

然而，这种中心化方法意味着所有代理必须通过协调代理传送它们的数据，包括敏感的用户信息，以完成任务。虽然这种方法在协调方面高效，但由于用户数据必须经过多个代理，它也带来了潜在的隐私和安全风险。

![参见标题](img/5ea1846594ac348068f75c7c40942553.png)

图 7. LaMAS 的去中心化星型架构。

![参见标题](img/41ce0605d8752353cf02c79e3b3fa9b2.png)

图 8. LaMAS 的去中心化数据处理。

为了应对这些隐私问题，我们提出了一种修改后的去中心化星型架构，图[7](https://arxiv.org/html/2411.14033v2#S4.F7 "图 7 ‣ 4.2. LaMAS 中的去中心化星型架构 ‣ 4. 案例研究 ‣ 基于大语言模型的多代理系统：技术与商业视角")和图[8](https://arxiv.org/html/2411.14033v2#S4.F8 "图 8 ‣ 4.2. LaMAS 中的去中心化星型架构 ‣ 4. 案例研究 ‣ 基于大语言模型的多代理系统：技术与商业视角")展示了这一改进，通过旅行预订场景来说明。在这一新设计中，协调代理仍然负责任务协调，但避免直接处理敏感数据。相反，像导航代理或票务代理等专门的代理会独立处理它们的任务，并在需要时直接与用户数据进行交互。这种设置减少了隐私风险，同时保持了系统的高效性。

在去中心化架构中，协调代理专注于将用户指令分解成更小的任务，并决定任务执行的顺序。它不会参与敏感数据的处理，仅在任务完成或需要额外协调时才重新连接。每个专门的代理在其各自的数据领域内处理特定任务，确保隐私和安全性。

如图[6](https://arxiv.org/html/2411.14033v2#S4.F6 "图 6 ‣ 4.1. LaMAS 架构 ‣ 4. 案例研究 ‣ 基于大语言模型的多代理系统：技术与商业视角")和图[8](https://arxiv.org/html/2411.14033v2#S4.F8 "图 8 ‣ 4.2. LaMAS 中的去中心化星型架构 ‣ 4. 案例研究 ‣ 基于大语言模型的多代理系统：技术与商业视角")所示，由事务日志记录器管理的公平信用分配系统，确保所有代理根据它们完成的任务和使用的资源获得适当的奖励。这种信用分配方法在提高数据保护和系统安全性的同时，保持了操作的高效性。

## 5. 结论与未来

本文从技术和商业角度提供了对基于大型语言模型的多智能体系统（LaMAS）未来发展的分析。从技术角度来看，与传统的单一LLM智能体系统相比，LaMAS在整体性能和系统灵活性方面具有更高的潜力；从商业角度来看，LaMAS为专有数据的保存性和通过流量与智能的货币化带来了可行性，这本质上激励了各方实体为整个生态系统做出贡献。正在开发多个有效的多智能体通信与协作协议，这将推动LaMAS生态系统的实现，并在不久的将来实现人工集体智能。

## 参考文献

+   (1)

+   Alon 和 Kamfonas (2023) Gabriel Alon 和 Michael Kamfonas. 2023. 通过困惑度检测语言模型攻击。*arXiv 预印本 arXiv:2308.14132* (2023).

+   Amirkhani 和 Barshooi (2022) Abdollah Amirkhani 和 Amir Hossein Barshooi. 2022. 多智能体系统中的共识：综述。*人工智能评论* 55, 5 (2022年6月)，3897–3935. [https://doi.org/10.1007/s10462-021-10097-x](https://doi.org/10.1007/s10462-021-10097-x)

+   Aziz (2010) Haris Aziz. 2010. 多智能体系统：Y. Shoham 和 K. Leyton-Brown 编著的《算法、博弈论和逻辑基础》。剑桥大学出版社，2008. *SIGACT 新闻* 41, 1 (2010年3月)，34–37. [https://doi.org/10.1145/1753171.1753181](https://doi.org/10.1145/1753171.1753181)

+   Brown 等人 (2020) Tom B. Brown, Benjamin Mann, Nick Ryder 等人. 2020. 语言模型是少量学习者。arXiv:2005.14165 [cs.CL]

+   Brynjolfsson 和 McAfee (2014) E. Brynjolfsson 和 A. McAfee. 2014. 第二次机器时代：在辉煌技术时代中的工作、进步与繁荣。*广告研究期刊* (2014).

+   Busoniu 等人 (2008) Lucian Busoniu, Robert Babuska 和 Bart De Schutter. 2008. 多智能体强化学习的全面调查。*IEEE 系统、人工与控制论学报，第C部分（应用与评论）* 38, 2 (2008)，156–172.

+   Caffagni 等人 (2024) Davide Caffagni, Federico Cocchi, Luca Barsellotti, Nicholas Moratelli, Sara Sarto, Lorenzo Baraldi, Marcella Cornia 和 Rita Cucchiara. 2024. 多模态大型语言模型的（r）演变：一项调查。*arXiv 预印本 arXiv:2402.12451* (2024).

+   Chan 等人 (2023) Chi-Min Chan, Weize Chen, Yusheng Su, Jianxuan Yu, Wei Xue, Shanghang Zhang, Jie Fu 和 Zhiyuan Liu. 2023. ChatEval：通过多智能体辩论提升基于大型语言模型的评估器。arXiv:2308.07201 [cs.CL] [https://arxiv.org/abs/2308.07201](https://arxiv.org/abs/2308.07201)

+   Chen 等人 (2017) Hao Chen, Kim Laine 和 Peter Rindal. 2017. 基于同态加密的快速私密集合交集。在 *2017年ACM SIGSAC计算机与通信安全大会论文集* 中，1243–1255.

+   Chen et al. (2023) Weize Chen, Yusheng Su, Jingwei Zuo, Cheng Yang, Chenfei Yuan, Chen Qian, Chi-Min Chan, Yujia Qin, Yaxi Lu, Ruobing Xie, 等. 2023. Agentverse: 促进多智能体协作并探索智能体的涌现行为. *arXiv 预印本 arXiv:2308.10848* 2, 4 (2023), 5.

+   Chen et al. (2024) Weize Chen, Ziming You, Ran Li, Yitong Guan, Chen Qian, Chenyang Zhao, Cheng Yang, Ruobing Xie, Zhiyuan Liu, 和 Maosong Sun. 2024. 智能体的互联网：为协同智能织就异构智能体的网络. arXiv:2407.07061 [cs.CL] [https://arxiv.org/abs/2407.07061](https://arxiv.org/abs/2407.07061)

+   Cheon et al. (2017) Jung Hee Cheon, Andrey Kim, Miran Kim, 和 Yongsoo Song. 2017. 同态加密用于近似数值的算术运算. 收录于 *加密学进展–ASIACRYPT 2017：第23届国际密码学与信息安全理论及应用会议，香港，中国，2017年12月3日至7日，论文集，第一部分 23*. Springer, 409–437.

+   Cloud (2024) Google Cloud. 2024. AutoML - Google Cloud. [https://cloud.google.com/automl/](https://cloud.google.com/automl/) 访问时间：2024-11-12.

+   Costan (2016) Victor Costan. 2016. Intel SGX 解释. *IACR Cryptol, EPrint Arch* (2016).

+   Dwork (2006) Cynthia Dwork. 2006. 差分隐私. 收录于 *国际自动机、语言与编程研讨会*. Springer, 1–12.

+   Face (2024) Hugging Face. 2024. Hugging Face 模型库. [https://huggingface.co/models](https://huggingface.co/models) 访问时间：2024-11-12.

+   Faroukhi et al. (2020) Abou Zakaria Faroukhi, Imane El Alaoui, Youssef Gahi, 和 Aouatif Amine. 2020. 大数据货币化与大数据价值链：全面回顾. *大数据期刊* 7 (2020), 1–22.

+   Fourney et al. (2024) Adam Fourney, Gagan Bansal, Hussein Mozannar, Cheng Tan, Eduardo Salinas, Erkang, Zhu, Friederike Niedtner, Grace Proebsting, Griffin Bassman, Jack Gerrits, Jacob Alber, Peter Chang, Ricky Loynd, Robert West, Victor Dibia, Ahmed Awadallah, Ece Kamar, Rafah Hosn, 和 Saleema Amershi. 2024. Magentic-One: 一个通用的多智能体系统用于解决复杂任务. arXiv:2411.04468 [cs.AI] [https://arxiv.org/abs/2411.04468](https://arxiv.org/abs/2411.04468)

+   Fu et al. (2024) Dayuan Fu, Biqing Qi, Yihuai Gao, Che Jiang, Guanting Dong, 和 Bowen Zhou. 2024. MSI-Agent: 将多尺度洞察融入具身智能体以实现更优的规划和决策. arXiv:2409.16686 [cs.AI] [https://arxiv.org/abs/2409.16686](https://arxiv.org/abs/2409.16686)

+   Gao et al. (2024) Dawei Gao, Zitao Li, Xuchen Pan, Weirui Kuang, Zhijian Ma, Bingchen Qian, Fei Wei, Wenhao Zhang, Yuexiang Xie, Daoyuan Chen, Liuyi Yao, Hongyi Peng, Zeyu Zhang, Lin Zhu, Chen Cheng, Hongzhu Shi, Yaliang Li, Bolin Ding, 和 Jingren Zhou. 2024. AgentScope: 一个灵活而强大的多智能体平台. arXiv:2402.14034 [cs.MA] [https://arxiv.org/abs/2402.14034](https://arxiv.org/abs/2402.14034)

+   Ghafarollahi 和 Buehler（2024）Alireza Ghafarollahi 和 Markus J. Buehler。2024年。SciAgents：通过多智能体图推理自动化科学发现。arXiv:2409.05556 [cs.AI] [https://arxiv.org/abs/2409.05556](https://arxiv.org/abs/2409.05556)

+   Gilad-Bachrach 等人（2016）Ran Gilad-Bachrach, Nathan Dowlin, Kim Laine, Kristin Lauter, Michael Naehrig, 和 John Wernsing。2016年。Cryptonets：将神经网络应用于加密数据，具有高吞吐量和高精度。在 *国际机器学习会议* 上。PMLR，201–210。

+   Guo 等人（2024）Taicheng Guo, Xiuying Chen, Yaqi Wang, Ruidi Chang, Shichao Pei, Nitesh V. Chawla, Olaf Wiest, 和 Xiangliang Zhang。2024年。基于大语言模型的多智能体：进展与挑战的综述。arXiv:2402.01680 [cs.CL] [https://arxiv.org/abs/2402.01680](https://arxiv.org/abs/2402.01680)

+   Hajipour 等人（2023）Vahid Hajipour, Siavash Hekmat, 和 Mohammad Amini。2023年。基于价值的人工智能即服务商业计划，使用集成工具和服务。*决策分析期刊* 8（2023年），100302。 [https://doi.org/10.1016/j.dajour.2023.100302](https://doi.org/10.1016/j.dajour.2023.100302)

+   Hines 等人（2024）Keegan Hines, Gary Lopez, Matthew Hall, Federico Zarfati, Yonatan Zunger, 和 Emre Kiciman。2024年。通过聚焦技术防御间接提示注入攻击。*arXiv 预印本 arXiv:2403.14720*（2024年）。

+   Hong 等人（2024）Sirui Hong, Mingchen Zhuge, Jonathan Chen, Xiawu Zheng, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, Chenyu Ran, Lingfeng Xiao, Chenglin Wu, 和 Jürgen Schmidhuber。2024年。MetaGPT：为多智能体协作框架进行元编程。在 *第十二届国际学习表征会议* 上。[https://openreview.net/forum?id=VtmBAGCN7o](https://openreview.net/forum?id=VtmBAGCN7o)

+   IBM（2024）IBM。2024年。watsonx Assistant。 [https://www.ibm.com/cn-zh/products/watsonx-assistant](https://www.ibm.com/cn-zh/products/watsonx-assistant) 访问时间：2024年11月12日。

+   Jain 等人（2023）Neel Jain, Avi Schwarzschild, Yuxin Wen, Gowthami Somepalli, John Kirchenbauer, Ping-yeh Chiang, Micah Goldblum, Aniruddha Saha, Jonas Geiping, 和 Tom Goldstein。2023年。针对对齐语言模型的对抗性攻击的基准防御。*arXiv 预印本 arXiv:2309.00614*（2023年）。

+   Kannan 和 Li（2017）P. K. Kannan 和 H. Li。2017年。数字广告：回顾与未来研究方向。*广告学杂志*（2017年）。

+   Knott 等人（2021）Brian Knott, Shobha Venkataraman, Awni Hannun, Shubho Sengupta, Mark Ibrahim, 和 Laurens van der Maaten。2021年。Crypten：安全的多方计算与机器学习的结合。*神经信息处理系统进展* 34（2021年），4961–4973。

+   Kumar 和 Reinartz（2016）V. Kumar 和 W. Reinartz。2016年。*创造持久的客户价值*。沃顿商学院出版社。

+   Li et al. (2024) Ao Li, Yuexiang Xie, Songze Li, Fugee Tsung, Bolin Ding, 和 Yaliang Li. 2024. 多代理系统中的面向代理规划。arXiv:2410.02189 [cs.AI] [https://arxiv.org/abs/2410.02189](https://arxiv.org/abs/2410.02189)

+   Li and Xu (2019) Yi Li 和 Wei Xu. 2019. PrivPy：通用且可扩展的隐私保护数据挖掘。在*第25届ACM SIGKDD国际知识发现与数据挖掘大会论文集*中，1299–1307。

+   Li et al. (2023) Yuan Li, Yixuan Zhang, 和 Lichao Sun. 2023. MetaAgents：通过协作生成代理模拟人类行为交互，基于大型语言模型的任务导向协调。arXiv:2310.06500 [cs.AI] [https://arxiv.org/abs/2310.06500](https://arxiv.org/abs/2310.06500)

+   Liang et al. (2024) Tian Liang, Zhiwei He, Wenxiang Jiao, Xing Wang, Yan Wang, Rui Wang, Yujiu Yang, Shuming Shi, 和 Zhaopeng Tu. 2024. 通过多代理辩论鼓励大型语言模型的发散思维。arXiv:2305.19118 [cs.CL] [https://arxiv.org/abs/2305.19118](https://arxiv.org/abs/2305.19118)

+   Lin et al. (2024) Qiqiang Lin, Muning Wen, Qiuying Peng, Guanyu Nie, Junwei Liao, Jun Wang, Xiaoyun Mo, Jiamu Zhou, Cheng Cheng, Yin Zhao, Jun Wang, 和 Weinan Zhang. 2024. Hammer：通过函数掩蔽实现面向设备语言模型的稳健函数调用。arXiv:2410.04587 [cs.LG] [https://arxiv.org/abs/2410.04587](https://arxiv.org/abs/2410.04587)

+   Lindell (2020) Yehuda Lindell. 2020. 安全多方计算。*Commun. ACM* 64, 1 (2020), 86–96。

+   Liu et al. (2024) Wei Liu, Chenxi Wang, Yifei Wang, Zihao Xie, Rennai Qiu, Yufan Dang, Zhuoyun Du, Weize Chen, Cheng Yang, 和 Chen Qian. 2024. 信息不对称下的协作任务自治代理。arXiv:2406.14928 [cs.AI] [https://arxiv.org/abs/2406.14928](https://arxiv.org/abs/2406.14928)

+   Liu et al. (2023) Yi Liu, Gelei Deng, Yuekang Li, Kailong Wang, Zihao Wang, Xiaofeng Wang, Tianwei Zhang, Yepang Liu, Haoyu Wang, Yan Zheng, 等。2023. 针对LLM集成应用的提示注入攻击。*arXiv预印本arXiv:2306.05499* (2023)。

+   Lowe et al. (2017) Ryan Lowe, Yi I Wu, Aviv Tamar, Jean Harb, OpenAI Pieter Abbeel, 和 Igor Mordatch. 2017. 混合合作-竞争环境中的多代理演员-评论员。*Advances in neural information processing systems* 30 (2017)。

+   Marro et al. (2024) Samuele Marro, Emanuele La Malfa, Jesse Wright, Guohao Li, Nigel Shadbolt, Michael Wooldridge, 和 Philip Torr. 2024. 一种可扩展的通信协议，用于大型语言模型网络。arXiv:2410.11905 [cs.AI] [https://arxiv.org/abs/2410.11905](https://arxiv.org/abs/2410.11905)

+   Morris et al. (2023) John X Morris, Wenting Zhao, Justin T Chiu, Vitaly Shmatikov, 和 Alexander M Rush. 2023. 语言模型反演。*arXiv预印本arXiv:2311.13647* (2023)。

+   O’Neill 等人（2023）Charles O’Neill、Jack Miller、Ioana Ciuca、Yuan-Sen Ting 和 Thang Bui。2023. 语言模型的对抗性微调：生成和检测问题内容的迭代优化方法。*arXiv 预印本 arXiv:2308.13768* (2023).

+   OpenAI 等人（2024）OpenAI、Josh Achi@ARTICLE7414384、作者=Agiwal, Mamta 和 Roy, Abhishek 和 Saxena, Navrati，期刊=IEEE Communications Surveys & Tutorials，标题=下一代 5G 无线网络：一项全面调查，年份=2016，卷号=18，期号=3，页码=1617-1655，关键词=5G 移动通信；无线通信；计算机架构；微处理器；MIMO；流媒体；5G；毫米波；波束赋形；信道模型；C-RAN；SDN；HetNets；大规模 MIMO；SDMA；IDMA；D2D；M2M；IoT；QoE；SON；可持续性；现场试验，DOI=10.1109/COMST.2016.2532458 am，Steven Adler，Sandhini Agarwal 等人。2024. GPT-4 技术报告。arXiv:2303.08774 [cs.CL] [https://arxiv.org/abs/2303.08774](https://arxiv.org/abs/2303.08774)

+   Patil 等人（2023）Shishir G. Patil、Tianjun Zhang、Xin Wang 和 Joseph E. Gonzalez。2023. Gorilla: 与大量 API 连接的大型语言模型。arXiv:2305.15334 [cs.CL] [https://arxiv.org/abs/2305.15334](https://arxiv.org/abs/2305.15334)

+   Pinto 和 Santos（2019）Sandro Pinto 和 Nuno Santos。2019. 解密 ARM TrustZone：一项全面调查。*ACM 计算机调查 (CSUR)* 51, 6 (2019), 1–36.

+   Pomerleau（1991）Dean A Pomerleau。1991. 用于自主导航的人工神经网络高效训练。*神经计算* 3, 1 (1991), 88–97.

+   Qian 等人（2023）Chen Qian、Wei Liu、Hongzhang Liu、Nuo Chen、Yufan Dang、Jiahao Li、Cheng Yang、Weize Chen、Yusheng Su、Xin Cong、Juyuan Xu、Dahai Li、Zhiyuan Liu 和 Maosong Sun. 2023. ChatDev: 用于软件开发的交互式智能体。*arXiv 预印本 arXiv:2307.07924* (2023). [https://arxiv.org/abs/2307.07924](https://arxiv.org/abs/2307.07924)

+   Radford 等人（2021）Alec Radford、Jong Wook Kim、Chris Hallacy 等人。2021. 从自然语言监督中学习可转移的视觉模型。arXiv:2103.00020 [cs.CL]

+   Rafailov 等人（2024）Rafael Rafailov、Archit Sharma、Eric Mitchell、Christopher D Manning、Stefano Ermon 和 Chelsea Finn。2024. 直接偏好优化：你的语言模型其实是一个奖励模型。*神经信息处理系统进展* 36 (2024).

+   Rajaei（2024）Saman Rajaei。2024. 作为服务的多智能体——高级工程师概述。[https://towardsdatascience.com/multi-agent-as-a-service-a-senior-engineers-overview-fc759f5bbcfa](https://towardsdatascience.com/multi-agent-as-a-service-a-senior-engineers-overview-fc759f5bbcfa)

+   Rashid 等人（2020）Tabish Rashid、Mikayel Samvelyan、Christian Schroeder De Witt、Gregory Farquhar、Jakob Foerster 和 Shimon Whiteson。2020. 深度多智能体强化学习中的单调值函数因式分解。*机器学习研究杂志* 21, 178 (2020), 1–51.

+   Robey 等人 (2023) Alexander Robey, Eric Wong, Hamed Hassani 和 George J Pappas. 2023. Smoothllm：防御大型语言模型免受越狱攻击。*arXiv 预印本 arXiv:2310.03684* (2023)。

+   Schick 等人 (2023) Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Luke Zettlemoyer, Nicola Cancedda 和 Thomas Scialom. 2023. Toolformer：语言模型能够自我学习使用工具。arXiv:2302.04761 [cs.CL] [https://arxiv.org/abs/2302.04761](https://arxiv.org/abs/2302.04761)

+   Sev-Snp (2020) AMD Sev-Snp. 2020. 通过完整性保护和更多功能加强虚拟机隔离。*白皮书，2020年1月* 53 (2020)，1450–1465。

+   Sha 和 Zhang (2024) Zeyang Sha 和 Yang Zhang. 2024. 针对大型语言模型的提示窃取攻击。*arXiv 预印本 arXiv:2402.12959* (2024)。

+   Shapley (1953) Lloyd Shapley. 1953. n人博弈的价值。在 *博弈理论贡献* 中。第II卷。普林斯顿大学出版社，307–317。

+   Shoham 和 Leyton-Brown (2009) Yoav Shoham 和 Kevin Leyton-Brown. 2009. 多智能体系统：算法、博弈论和逻辑基础。

+   Sjödin 等人 (2021) David Sjödin, Vinit Parida, Maximilian Palmié 和 Joakim Wincent. 2021. 人工智能能力如何促进商业模型创新：通过共进化过程和反馈回路扩大人工智能的规模。*商业研究期刊* 134 (2021)，574–587。 [https://doi.org/10.1016/j.jbusres.2021.05.009](https://doi.org/10.1016/j.jbusres.2021.05.009)

+   Sukhbaatar 等人 (2016) Sainbayar Sukhbaatar, Rob Fergus 等人 2016. 使用反向传播学习多智能体通信。*神经信息处理系统进展* 29 (2016)。

+   Tan (1993) Ming Tan. 1993. 多智能体强化学习：独立智能体与合作智能体的对比。发表于 *第十届国际机器学习会议论文集*，330–337。

+   Tian 等人 (2019) Zheng Tian, Ying Wen, Zhichen Gong, Faiz Punakkath, Shihao Zou 和 Jun Wang. 2019. 一个带最大熵目标的正则化对手模型。发表于 *第28届国际人工智能联合会议论文集*，602–608。

+   Trirat 等人 (2024) Patara Trirat, Wonyong Jeong 和 Sung Ju Hwang. 2024. AutoML-Agent：一个用于全管道自动机器学习的多智能体大语言模型框架。arXiv:2410.02958 [cs.LG] [https://arxiv.org/abs/2410.02958](https://arxiv.org/abs/2410.02958)

+   VentureBeat (2024) VentureBeat. 2024. 微软悄然组建了全球最大人工智能智能体生态系统——而其他公司远远落后。*VentureBeat* (2024)。 [https://venturebeat.com/ai/microsoft-quietly-assembles-the-largest-ai-agent-ecosystem-and-no-one-else-is-close/](https://venturebeat.com/ai/microsoft-quietly-assembles-the-largest-ai-agent-ecosystem-and-no-one-else-is-close/) 访问时间：2024-11-21。

+   Wang et al. (2024) Jun Wang, Meng Fang, Ziyu Wan, Muning Wen, Jiachen Zhu, Anjie Liu, Ziqin Gong, Yan Song, Lei Chen, Lionel M Ni, 等。2024. OpenR：一个用于大型语言模型高级推理的开源框架。*arXiv预印本 arXiv:2410.09671*（2024）。

+   Wei et al. (2023) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, Fei Xia, Ed Chi, Quoc Le, and Denny Zhou. 2023. 连锁思维提示激发大型语言模型中的推理。arXiv:2201.11903 [cs.CL] [https://arxiv.org/abs/2201.11903](https://arxiv.org/abs/2201.11903)

+   Wen et al. (2022) Muning Wen, Jakub Kuba, Runji Lin, Weinan Zhang, Ying Wen, Jun Wang, and Yaodong Yang. 2022. 多智能体强化学习是一个序列建模问题。*神经信息处理系统进展* 35 (2022), 16509–16521。

+   Wooldridge (2009) Michael Wooldridge. 2009. 《多智能体系统导论》。

+   Wu et al. (2023) Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang, and Chi Wang. 2023. Autogen：通过多智能体对话框架实现下一代大型语言模型应用。*arXiv预印本 arXiv:2308.08155*（2023）。

+   Xhonneux et al. (2024) Sophie Xhonneux, Alessandro Sordoni, Stephan Günnemann, Gauthier Gidel, and Leo Schwinn. 2024. 在大型语言模型中进行高效对抗性训练与连续攻击。*arXiv预印本 arXiv:2405.15589*（2024）。

+   Xie et al. (2023) Tianbao Xie, Fan Zhou, Zhoujun Cheng, Peng Shi, Luoxuan Weng, Yitao Liu, Toh Jing Hua, Junning Zhao, Qian Liu, Che Liu, 等。2023. Openagents：一个开放平台，支持野外语言智能体。*arXiv预印本 arXiv:2310.10634*（2023）。

+   Xue et al. (2024) Jiaqi Xue, Mengxin Zheng, Yebowen Hu, Fei Liu, Xun Chen, and Qian Lou. 2024. BadRAG：识别大型语言模型的检索增强生成中的漏洞。*arXiv预印本 arXiv:2406.00083*（2024）。

+   Yan et al. (2024) Jun Yan, Vikas Yadav, Shiyang Li, Lichang Chen, Zheng Tang, Hai Wang, Vijay Srinivasan, Xiang Ren, and Hongxia Jin. 2024. 通过虚拟提示注入对指令调优的大型语言模型进行后门攻击。发表于 *2024年北美计算语言学协会：人类语言技术会议（卷1：长篇论文）*，6065–6086。

+   Yao et al. (2023) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. 2023. ReAct：在语言模型中协同推理与行动。arXiv:2210.03629 [cs.CL] [https://arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629)

+   Zhang et al. (2024b) Chaoyun Zhang, Shilin He, Jiaxu Qian, Bowen Li, Liqun Li, Si Qin, Yu Kang, Minghua Ma, Guyue Liu, Qingwei Lin, Saravan Rajmohan, Dongmei Zhang, and Qi Zhang. 2024b. 基于大型语言模型的GUI智能体：一项综述。arXiv:2411.18279 [cs.AI] [https://arxiv.org/abs/2411.18279](https://arxiv.org/abs/2411.18279)

+   Zhang 等人（2024c）Weinan Zhang、Junwei Liao、Ning Li 和 Kounianhua Du。2024c。代理信息检索。arXiv:2410.09713 [cs.IR] [https://arxiv.org/abs/2410.09713](https://arxiv.org/abs/2410.09713)

+   Zhang 等人（2024a）Zeyu Zhang、Xiaohe Bo、Chen Ma、Rui Li、Xu Chen、Quanyu Dai、Jieming Zhu、Zhenhua Dong 和 Ji-Rong Wen。2024a。基于大语言模型的代理记忆机制综述。arXiv:2404.13501 [cs.AI] [https://arxiv.org/abs/2404.13501](https://arxiv.org/abs/2404.13501)

+   Zhou 等人（2024b）Ruiwen Zhou、Yingxuan Yang、Muning Wen、Ying Wen、Wenhao Wang、Chunling Xi、Guoqiang Xu、Yong Yu 和 Weinan Zhang。2024b。TRAD：通过逐步思维检索和对齐决策增强大语言模型代理。在 *第47届国际ACM SIGIR信息检索研究与开发会议论文集*（美国华盛顿DC） *(SIGIR ’24)*。美国纽约计算机协会（ACM），纽约，NY，美国，3–13。[https://doi.org/10.1145/3626772.3657788](https://doi.org/10.1145/3626772.3657788)

+   Zhou 等人（2024a）Wangchunshu Zhou、Yixin Ou、Shengwei Ding、Long Li、Jialong Wu、Tiannan Wang、Jiamin Chen、Shuai Wang、Xiaohua Xu、Ningyu Zhang、Huajun Chen 和 Yuchen Eleanor Jiang。2024a。符号学习使自我进化代理成为可能。arXiv:2406.18532 [cs.CL] [https://arxiv.org/abs/2406.18532](https://arxiv.org/abs/2406.18532)

+   Zhuge 等人（[n.d.]）Mingchen Zhuge、Wenyi Wang、Louis Kirsch、Francesco Faccio、Dmitrii Khizbullin 和 Jürgen Schmidhuber。[n.d.]。GPTSwarm：作为可优化图的语言代理。在 *第41届国际机器学习会议*。

+   Zou 等人（2023）Andy Zou、Zifan Wang、Nicholas Carlini、Milad Nasr、J Zico Kolter 和 Matt Fredrikson。2023。对齐语言模型的通用且可转移的对抗攻击。 *arXiv 预印本 arXiv:2307.15043*（2023）。
