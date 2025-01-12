<!--yml

类别：未分类

日期：2025-01-11 11:42:55

-->

# TradingAgents: 多代理大型语言模型金融交易框架

> 来源：[https://arxiv.org/html/2412.20138/](https://arxiv.org/html/2412.20138/)

Yijia Xiao¹, Edward Sun¹, Di Luo^(1,2), Wei Wang¹

###### 摘要

在利用大规模语言模型（LLMs）驱动的代理群体进行自动化问题解决方面，已经取得了显著进展。在金融领域，努力的重点大多集中在处理特定任务的单一代理系统或独立收集数据的多代理框架。然而，多代理系统在模拟现实世界交易公司协作动态的潜力仍未得到充分探索。TradingAgents提出了一种受交易公司启发的全新股票交易框架，框架中包含了多个由LLM驱动的代理，分别担任不同的专业角色，如基本面分析师、情绪分析师、技术分析师以及具有不同风险偏好的交易员。该框架包括了评估市场条件的牛市和熊市研究员代理、监控风险敞口的风险管理团队，以及通过辩论和历史数据综合见解、做出明智决策的交易员。通过模拟动态的协作交易环境，该框架旨在提升交易表现。详细的架构和广泛的实验揭示了其在累积回报、夏普比率和最大回撤方面显著优于基准模型，突显了多代理LLM框架在金融交易中的潜力。

## 介绍

利用大规模语言模型（LLMs）的自主代理提供了一种革命性的决策方式，能够通过复制人类的过程和工作流应用于各种场景。这些系统通过为语言代理提供工具并使其能够与其他代理协作，从而增强了解决问题的能力，有效地将复杂问题分解成可管理的部分（Park et al. [2023](https://arxiv.org/html/2412.20138v2#bib.bib18); Havrilla et al. [2024](https://arxiv.org/html/2412.20138v2#bib.bib7); Talebirad and Nadiri [2023](https://arxiv.org/html/2412.20138v2#bib.bib21); Tang et al. [2024](https://arxiv.org/html/2412.20138v2#bib.bib22)）。这些自主框架的一个突出应用是在金融市场——一个高度复杂的系统，受到众多因素的影响，包括公司基本面、市场情绪、技术指标以及宏观经济事件。

传统的算法交易系统通常依赖于量化模型，而这些模型难以完全捕捉各种因素之间的复杂相互作用。相比之下，LLM在处理和理解自然语言数据方面表现优异，使其在需要文本理解的任务中，尤其是在分析新闻文章、财务报告和社交媒体情绪时，特别有效。此外，基于深度学习的交易系统通常存在低解释性的问题，因为它们依赖于驱动决策的隐藏特征，而这些特征难以解释。最近在金融领域的多代理LLM框架取得了显著进展，这些框架通过创建可解释的人工智能系统，使决策得到证据和透明推理的支持（Li et al. [2023a](https://arxiv.org/html/2412.20138v2#bib.bib12); Wang et al. [2024b](https://arxiv.org/html/2412.20138v2#bib.bib25); Yu et al. [2024](https://arxiv.org/html/2412.20138v2#bib.bib36)），展示了它们在金融应用中的潜力。

尽管具有潜力，目前语言代理在金融和交易领域的应用面临两个重大局限性：

缺乏现实的组织建模：许多框架未能捕捉到模拟真实世界交易公司结构的代理之间复杂的互动（Li et al. [2023a](https://arxiv.org/html/2412.20138v2#bib.bib12); Wang et al. [2024b](https://arxiv.org/html/2412.20138v2#bib.bib25); Yu et al. [2024](https://arxiv.org/html/2412.20138v2#bib.bib36)）。相反，它们往往专注于特定任务的执行，通常与组织工作流程和已被验证有效的人工操作程序脱节，从而限制了它们在完全复制和受益于现实世界交易实践方面的能力。

沟通接口低效：大多数现有系统将自然语言作为主要的沟通媒介，通常依赖消息历史或无结构的信息池来进行决策（Park et al. [2023](https://arxiv.org/html/2412.20138v2#bib.bib18); Qian et al. [2024](https://arxiv.org/html/2412.20138v2#bib.bib19)）。这种方法往往导致“电话效应”，即随着对话的延长，细节丧失，状态受到干扰。代理在保持上下文和跟踪扩展历史的同时，还需要从以前的决策步骤中过滤掉无关信息，这降低了它们在处理复杂、动态任务时的效果。此外，无结构的信息池方法缺乏明确的指令，迫使代理之间的逻辑沟通和信息交换完全依赖于检索，这破坏了数据的关系完整性。

在本研究中，我们通过引入一个克服这些挑战的系统，解决了现有模型的这些关键限制。首先，我们的框架通过模拟典型的专业交易团队的多代理决策过程，弥合了这一差距。它包括针对交易各个方面量身定制的专业代理，灵感来自于现实世界交易公司的组织结构。这些代理包括基本面分析师、情感/新闻分析师、技术分析师和具有不同风险偏好的交易员。多头和空头辩论者评估市场状况，以提供平衡的建议，而风险管理团队确保暴露度保持在可接受的范围内。其次，为了增强沟通，我们的框架结合了结构化的输出，用于控制、明确性和推理，同时结合自然语言对话，以促进代理之间有效的辩论和协作。这种混合方法确保了决策过程中的精确性和灵活性。

我们通过在历史金融数据上的实验验证了我们的框架，并与多个基准进行性能对比。采用了综合评估指标，包括累计回报、夏普比率和最大回撤，用于评估其整体效果。

## 相关工作

### LLMs作为金融助手

大型语言模型（LLMs）通过在金融数据上进行微调或在金融语料库上进行训练，已被应用于金融领域。这提升了模型对金融术语和数据的理解，使其能够作为专门的助手提供分析支持、洞察和信息检索，而非执行交易。

微调的金融LLMs

微调提升了领域特定的表现。例子包括PIXIU（FinMA）（Xie等人[2023](https://arxiv.org/html/2412.20138v2#bib.bib28)），该方法在136K与金融相关的指令上微调了LLaMA；FinGPT（Yang, Liu, and Wang [2023](https://arxiv.org/html/2412.20138v2#bib.bib31)），该方法使用LoRA对LLaMA和ChatGLM等模型进行了微调，使用了大约50K与金融相关的样本；以及Instruct-FinGPT（Zhang, Yang, and Liu [2023](https://arxiv.org/html/2412.20138v2#bib.bib37)），该方法在来自金融情感分析数据集的10K指令样本上进行了微调。这些模型在金融分类任务中超越了它们的基础版本以及其他开源LLM，如BLOOM和OPT（Zhang等人[2022](https://arxiv.org/html/2412.20138v2#bib.bib39)），甚至在多个评估中超越了BloombergGPT（Wu等人[2023](https://arxiv.org/html/2412.20138v2#bib.bib27)）。然而，在生成任务中，它们的表现与强大的通用模型如GPT-4相似或稍逊，表明仍然需要更多高质量的领域特定数据集。

从头开始训练的金融LLMs

从零开始在金融特定语料库上训练LLMs旨在实现更好的领域适应性。像BloombergGPT（Wu et al. [2023](https://arxiv.org/html/2412.20138v2#bib.bib27)）、XuanYuan 2.0（Zhang, Yang, 和 Xu [2023](https://arxiv.org/html/2412.20138v2#bib.bib41)）和Fin-T5（Lu et al. [2023](https://arxiv.org/html/2412.20138v2#bib.bib15)）等模型在预训练过程中结合了公共数据集和金融特定数据。例如，BloombergGPT在一般文本和金融文本上进行了训练，且通过专有的Bloomberg数据增强了其在金融基准测试中的表现。这些模型在市场情绪分类和摘要等任务中超越了通用模型，如BLOOM-176B和T5。尽管它们可能无法与像GPT-3或PaLM（Chowdhery et al. [2022](https://arxiv.org/html/2412.20138v2#bib.bib2)）等更大的闭源模型相媲美，但在相似规模的开源模型中，它们提供了具有竞争力的表现，且不牺牲通用语言理解能力。

总结来说，通过微调或从零开始训练的金融特定LLMs在领域特定任务中表现出显著的提升，强调了领域适应性的重要性，并展示了通过高质量金融特定数据集进行进一步增强的潜力。

![参考图注](img/8ac7c950a7ad040502a68e211a07e194.png)

图1：交易代理总体框架组织。I. 分析团队：四位分析师并行收集相关市场信息。II. 研究团队：该团队讨论并评估收集到的数据。III. 交易员：基于研究人员的分析，交易员做出交易决策。IV. 风险管理团队：风险守护者根据当前市场条件评估决策，以降低风险。V. 基金经理：基金经理批准并执行交易。

### LLMs 作为交易员

LLMs 作为交易代理，通过分析外部数据如新闻、财务报告和股票价格，做出直接的交易决策。提出的架构包括基于新闻驱动、基于推理驱动和基于强化学习（RL）驱动的代理。

基于新闻驱动的代理

新闻驱动的架构将股票新闻和宏观经济更新整合到LLM提示中，用于预测股价走势。评估了闭源模型（例如GPT-3.5、GPT-4）和开源LLM（例如Qwen（Bai等人 [2023](https://arxiv.org/html/2412.20138v2#bib.bib1)）、Baichuan（Yang等人 [2023](https://arxiv.org/html/2412.20138v2#bib.bib30)））在金融情绪分析中的研究表明，基于情绪评分的简单长短策略有效（Lopez-Lira和Tang [2023](https://arxiv.org/html/2412.20138v2#bib.bib14)）。进一步研究如FinGPT和OPT等经过微调的LLM展示了通过领域特定对齐提升的表现（Zhang等人 [2024a](https://arxiv.org/html/2412.20138v2#bib.bib38)；Kirtac和Germano [2024](https://arxiv.org/html/2412.20138v2#bib.bib10)）。高级方法涉及总结新闻数据并推理其与股价的关系（Fatouros等人 [2024a](https://arxiv.org/html/2412.20138v2#bib.bib5)；Wang、Izumi和Sakaji [2024](https://arxiv.org/html/2412.20138v2#bib.bib24)）。

推理驱动的智能体

推理驱动的智能体通过反思和辩论等机制增强交易决策。反思驱动的智能体，如FinMem（Yu等人 [2023](https://arxiv.org/html/2412.20138v2#bib.bib35)）和FinAgent（Zhang等人 [2024b](https://arxiv.org/html/2412.20138v2#bib.bib40)），使用分层记忆和多模态数据将输入总结为记忆、指导决策，并结合技术指标，实现优于回测的表现，同时减少幻觉（Ji等人 [2023](https://arxiv.org/html/2412.20138v2#bib.bib9)）。辩论驱动的智能体，如异构框架中的智能体（Xing [2024](https://arxiv.org/html/2412.20138v2#bib.bib29)）和TradingGPT（Li等人 [2023b](https://arxiv.org/html/2412.20138v2#bib.bib13)），通过在不同角色的智能体之间进行LLM辩论来增强推理和事实的有效性，从而改善情绪分类并提高交易决策的稳健性。

强化学习驱动的智能体

强化学习方法将LLM输出与预期行为对齐，利用回测作为奖励。SEP（Koa等人 [2024](https://arxiv.org/html/2412.20138v2#bib.bib11)）采用带有记忆和反思的RL来根据市场历史优化LLM预测。经典的RL方法也用于将LLM生成的嵌入与股票特征结合的交易框架，使用如近端策略优化（PPO）等算法进行训练（Ding等人 [2023](https://arxiv.org/html/2412.20138v2#bib.bib3)；Schulman等人 [2017](https://arxiv.org/html/2412.20138v2#bib.bib20)）。

### LLMs作为Alpha矿工

LLMs 也被用于生成 alpha 因子，而不是直接做出交易决策。QuantAgent（王等人 [2023](https://arxiv.org/html/2412.20138v2#bib.bib26)）通过利用 LLMs 在内循环和外循环架构中生成 alpha 因子来证明这一点。在内循环中，一个写作代理根据交易员的想法生成脚本，而评判代理提供反馈。在外循环中，代码在真实市场中进行测试，交易结果则优化评判代理。这种方法能够逐步逼近最优行为。

后续研究，如 AlphaGPT（王等人 [2023](https://arxiv.org/html/2412.20138v2#bib.bib26)），提出了一个人类参与的 alpha 挖掘框架，采用了类似的架构。两项研究展示了 LLM 驱动的 alpha 挖掘系统的有效性，突显了它们在通过生成和优化 alpha 因子来自动化和加速交易策略开发方面的潜力。

## TradingAgents：角色专业化

给 LLM 代理分配明确、定义良好的角色和具体目标，使得复杂的目标能够拆解为更小、更易管理的子任务。金融交易就是这样复杂性的一个典型例子，需要整合多种信号、输入和专业知识。在现实世界中，这种管理复杂性的方式通过依赖专家团队协作并做出高风险决策的交易公司得以体现，突显了任务的多面性。

在典型的交易公司中，会收集大量数据，包括财务指标、价格波动、交易量、历史表现、经济指标和新闻情绪。然后，这些数据由量化专家（如数学家、数据科学家和工程师）使用先进的工具和算法进行分析，以识别趋势并预测市场走势。

受到这种组织结构的启发，TradingAgents 在模拟交易公司中定义了七个不同的代理角色：基本面分析师、情绪分析师、新闻分析师、技术分析师、研究员、交易员和风险经理。每个代理都被分配了特定的名称、角色、目标和一系列约束条件，同时配备了根据其职能量身定制的预定义上下文、技能和工具。例如，情绪分析师配备了如网络搜索引擎、Reddit 搜索 API、X/Twitter 搜索工具和情绪评分计算算法等工具，而技术分析师则可以执行代码、计算技术指标和分析交易模式。更具体地说，TradingAgents 假设以下团队结构。

### 分析师团队

分析员团队（图 [2](https://arxiv.org/html/2412.20138v2#Sx3.F2 "图 2 ‣ 分析员团队 ‣ TradingAgents: 角色专长 ‣ TradingAgents: 多代理 LLM 金融交易框架")）由负责收集和分析各种市场数据的专业代理组成，以为交易决策提供依据。每个代理专注于市场分析的特定方面，汇聚了市场状况的全面视角。

![参考标题](img/7a399bac754031ac1e4febe029996723.png)

图 2：TradingAgents 分析员团队

+   •

    基本面分析员代理：这些代理通过分析财务报表、盈利报告、内部交易和其他相关数据来评估公司的基本面。他们评估公司的内在价值，识别被低估或高估的股票，为长期投资潜力提供见解。

+   •

    情绪分析员代理：这些代理处理大量社交媒体帖子、情绪评分和来自公共信息与社交媒体活动的内部情绪数据。他们评估市场情绪，以预测集体投资者行为如何在短期内影响股价。

+   •

    新闻分析员代理：这些代理分析新闻文章、政府公告及其他宏观经济指标，以评估市场的宏观经济状况、重大世界事件和重要公司变动。他们识别可能影响市场波动的新闻事件，帮助预测市场动态的突变。

+   •

    技术分析员代理：这些代理计算并选择相关的技术指标，如移动平均收敛/发散指标（MACD）和相对强弱指数（RSI），并为特定资产量身定制。他们分析价格模式和交易量，以预测未来的价格走势，帮助确定进出场时机。

分析员团队综合来自多个来源的数据，提供全面的市场分析。他们的综合见解为研究员团队提供了基础输入，确保在随后的决策过程中考虑市场的各个方面。

### 研究员团队

研究员团队（图 [3](https://arxiv.org/html/2412.20138v2#Sx3.F3 "图 3 ‣ 研究员团队 ‣ TradingAgents: 角色专长 ‣ TradingAgents: 多代理 LLM 金融交易框架")）负责对分析员团队提供的信息进行严格评估。该团队由持有看涨和看跌观点的代理组成，他们进行多轮辩论，以评估投资决策的潜在风险和收益。

![参考标题](img/46a5a0c75ba098369906bfabf0622a90.png)

图 3：TradingAgents 研究员团队：看涨观点与看跌观点

+   •

    看涨研究员：这些代理通过突出积极的指标、增长潜力和有利的市场条件来倡导投资机会。他们构建支持在某些资产中启动或继续持仓的论据。

+   •

    看跌研究员：相反，这些代理关注潜在的下行风险、风险和不利的市场信号。他们提供警示性见解，质疑投资策略的可行性，并突出可能的负面结果。

通过这一辩证过程，研究团队旨在达成对市场情况的平衡理解。他们的深入分析有助于识别最有前景的投资策略，同时预测可能的挑战，从而帮助交易员代理做出明智的决策。

### 交易员代理

交易员代理（图 [4](https://arxiv.org/html/2412.20138v2#Sx3.F4 "图 4 ‣ 交易员代理 ‣ 交易代理：角色专业化 ‣ 交易代理：多代理 LLM 金融交易框架")）负责根据分析团队提供的全面分析以及研究团队提供的细致观点执行交易决策。他们评估综合信息，考虑定量数据和定性见解，以确定最佳的交易行动。

![参考说明](img/e7fb50b5b23faa656da70c7473452f67.png)

图 4：交易代理的交易决策过程

交易代理的任务包括：

+   •

    评估分析师和研究员的建议和见解。

+   •

    决定交易的时机和规模，以最大化交易回报。

+   •

    在市场中下达买入或卖出订单。

+   •

    根据市场变化和新信息调整投资组合分配。

交易员代理必须平衡潜在回报与相关风险，在动态市场环境中做出及时决策。他们的行为直接影响公司的业绩，因此需要高度的精确性和战略思维。

### 风险管理团队

风险管理团队（图 [5](https://arxiv.org/html/2412.20138v2#Sx3.F5 "图 5 ‣ 风险管理团队 ‣ 交易代理：角色专业化 ‣ 交易代理：多代理 LLM 金融交易框架")）监控和控制公司在各种市场风险中的暴露。这些代理不断评估投资组合的风险状况，确保交易活动在预定的风险参数范围内进行，并遵守监管要求。

风险管理团队的职责包括：

+   •

    评估市场波动性、流动性和对手风险等因素。

+   •

    实施风险缓解策略，例如设置止损单或多元化持仓。

+   •

    向交易员代理提供关于风险暴露的反馈，并建议调整交易策略。

+   •

    确保整体投资组合与公司的风险承受能力和投资目标一致。

![请参阅说明](img/0fdd058a593fd0b5ac870a7c72f9cf4f.png)

图5：TradingAgents风险管理团队与基金经理审批工作流程

通过提供监督和指导，风险管理团队帮助维持公司财务稳定，并防范不利的市场事件。他们在保护资产和确保可持续的长期业绩方面发挥着至关重要的作用。

TradingAgents中的所有代理都遵循ReAct提示框架（Yao et al. [2023](https://arxiv.org/html/2412.20138v2#bib.bib34)），该框架将推理与行动相结合。环境状态由代理共享并监控，使他们能够采取适当的行动，例如进行研究、执行交易、参与辩论或管理风险。这种设计确保了一个协作的、动态的决策过程，能够反映现实世界中的交易系统。

## TradingAgents：代理工作流程

### 通信协议

大多数现有的基于LLM的代理框架使用自然语言作为主要的通信接口，通常通过结构化的消息历史或代理生成的消息集合进行（Fatouros et al. [2024b](https://arxiv.org/html/2412.20138v2#bib.bib6); Li et al. [2023a](https://arxiv.org/html/2412.20138v2#bib.bib12); Yang et al. [2024](https://arxiv.org/html/2412.20138v2#bib.bib33); Yang, Yue, and He [2023](https://arxiv.org/html/2412.20138v2#bib.bib32)）。然而，仅依赖自然语言往往不足以解决复杂的、需要广泛规划的长期任务。在这种情况下，纯粹的自然语言交流就像是玩“电话游戏”——经过多轮迭代，初始信息可能由于上下文长度限制和过多的文本信息遮掩了关键的早期细节，从而被遗忘或扭曲（Hong et al. [2024](https://arxiv.org/html/2412.20138v2#bib.bib8)）。为了解决这个问题，我们借鉴了像MetaGPT这样的框架，它们采用了结构化的通信方法。我们的模型引入了一种结构化的通信协议来规范代理交互。通过明确定义每个代理的状态，我们确保每个角色只提取或查询必要的信息，处理后返回完整的报告。这种简化的方法减少了不必要的步骤，降低了消息损坏的风险，并使得交互在复杂、长时间跨度的任务中依然保持集中和高效。

### 代理交互类型

与以往依赖大量自然语言对话的多代理交易框架不同，TradingAgents代理主要通过结构化文档和图表进行沟通。这些文档以简洁、井然有序的报告形式概括了代理的见解，保留了必要的内容，同时避免了无关信息。通过使用结构化报告，代理可以直接从全球状态查询必要的细节，无需进行冗长的对话，这样可以避免信息稀释、消息状态无限延长以及数据丢失的风险。这些文档的类型及其所包含的信息如下所述：

+   •

    分析师团队：基础面、情绪、新闻和技术分析师将他们的研究和发现汇总成简洁的分析报告，针对各自领域的专业知识。这些报告包括关键指标、见解和基于他们专业分析的建议。

+   •

    交易员：交易员会审核并分析分析师的报告，仔细推敲以生成明确的决策信号。他们会附上详细的报告，解释其决策依据及支持证据，这些报告随后会被风险管理团队使用。

代理之间的对话仅在代理之间的交流和辩论中以自然语言进行。这些简洁、集中的讨论已被证明能够促进更深入的推理并融合多种观点，从而在复杂的长时间跨度情境中做出更平衡的决策——这一方法对于复杂的交易环境尤其相关（Du et al. [2023](https://arxiv.org/html/2412.20138v2#bib.bib4)）。这种方法与我们的结构化框架无缝集成，因为对话状态作为结构化条目被记录在整体代理状态中。这些情境中的通信类型如下所述：

+   •

    研究员团队：每个研究员代理会查询全球代理状态以获取分析报告，并仔细形成自己的意见。两位研究员代表对立的观点：一位看涨，一位看跌。他们会进行$n$轮自然语言对话，由辩论引导代理决定轮次。辩论结束后，引导代理会回顾辩论历史，选择占主导地位的观点，并将其记录为通信协议中的结构化条目。

+   •

    风险管理团队：与研究员团队类似，风险管理团队对交易员的决策及随附报告进行查询。然后，他们从风险寻求、中立和风险保守三个角度进行审议，以在风险约束下调整交易计划。他们通过一个引导代理进行$n$轮自然语言讨论。

+   •

    基金经理：基金经理审核风险管理团队的讨论，确定适当的风险调整，并在通信协议中更新交易员的决策和报告状态。

### 主干LLM

为了满足我们框架中任务的复杂性和速度需求，我们根据模型的优势战略性地选择大型语言模型（LLMs）。快速思考模型，如`gpt-4o-mini`和`gpt-4o`，高效地处理快速、低深度的任务，如摘要、数据检索以及将表格数据转换为文本（OpenAI等 [2024](https://arxiv.org/html/2412.20138v2#bib.bib17)）。相反，深度思考模型如`o1-preview`擅长处理推理密集型任务，如决策、基于证据的报告写作和数据分析。这些模型利用其架构进行多轮推理，生成逻辑严密、深入的见解（Zhong等 [2024](https://arxiv.org/html/2412.20138v2#bib.bib42)；Wang等 [2024a](https://arxiv.org/html/2412.20138v2#bib.bib23)；OpenAI [2024](https://arxiv.org/html/2412.20138v2#bib.bib16)）。此外，我们优先选择具有经过验证的可靠性和可扩展性的模型，以确保在各种市场条件下的最佳表现。我们还采用辅助专家模型来处理情感分析等专业任务。

具体而言，所有分析节点依赖深度思考模型以确保稳健的分析，而快速思考模型则处理从API和工具中检索数据，以提高效率。研究人员和交易员使用深度思考模型生成有价值的见解并支持做出明智的决策。通过将LLM的选择与每个任务的具体需求对齐，我们的框架在效率和推理深度之间达到了平衡，这对于有效的交易策略至关重要。

这种实施策略确保了TradingAgents可以在不需要GPU的情况下进行部署，仅依赖API积分。它还引入了骨干模型的无缝可交换性，使研究人员能够轻松地将模型替换为任何本地托管的或可以通过API访问的替代模型。这种适应性支持集成改进的推理模型或针对特定任务定制的金融调优模型。因此，TradingAgents具有高度的可扩展性和面向未来的能力，提供了灵活性，以适应任何代理的任何骨干模型。

## 实验

在本节中，我们描述了用于评估我们提出的框架的实验设置。我们还提供了用于全面评估性能的评估指标的详细描述。

表 1：使用四种评估指标对所有方法的性能进行比较。绿色高亮的结果代表每个模型的最佳表现统计数据。改进行展示了TradingAgents相较于最佳基准模型的性能提升。

| 类别 | 模型 | AAPL |  | GOOGL |  | AMZN |
| --- | --- | --- | --- | --- | --- | --- |
| CR%$\uparrow$ | ARR%$\uparrow$ | SR$\uparrow$ | MDD%$\downarrow$ |  | CR%$\uparrow$ | ARR%$\uparrow$ | SR$\uparrow$ | MDD%$\downarrow$ |  | CR%$\uparrow$ | ARR%$\uparrow$ | SR$\uparrow$ | MDD%$\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 市场 | B&H | -5.23 | -5.09 | -1.29 | 11.90 |  | 7.78 | 8.09 | 1.35 | 13.04 |  | 17.1 | 17.6 | 3.53 | 3.80 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 基于规则的 | MACD | -1.49 | -1.48 | -0.81 | 4.53 |  | 6.20 | 6.26 | 2.31 | 1.22 |  | - | - | - | - |
| KDJ&RSI | 2.05 | 2.07 | 1.64 | 1.09 |  | 0.4 | 0.4 | 0.02 | 1.58 |  | -0.77 | -0.76 | -2.25 | 1.08 |
| ZMR | 0.57 | 0.57 | 0.17 | 0.86 |  | -0.58 | 0.58 | 2.12 | 2.34 |  | -0.77 | -0.77 | -2.45 | 0.82 |
|  | SMA | -3.2 | -2.97 | -1.72 | 3.67 |  | 6.23 | 6.43 | 2.12 | 2.34 |  | 11.01 | 11.6 | 2.22 | 3.97 |
| Ours | TradingAgents | 26.62 | 30.5 | 8.21 | 0.91 |  | 24.36 | 27.58 | 6.39 | 1.69 |  | 23.21 | 24.90 | 5.60 | 2.11 |
| 改进（%） | 24.57 | 28.43 | 6.57 | - |  | 16.58 | 19.49 | 4.26 | - |  | 6.10 | 7.30 | 2.07 | - |

### 回测交易

为了模拟一个真实的交易环境，我们利用一个多资产、多模式的金融数据集，其中包含了诸如苹果、英伟达、微软、Meta、谷歌等多个股票。该数据集包括：

+   •

    历史股价：2024年1月1日至2024年3月29日的开盘价、最高价、最低价、收盘价、成交量和调整后的收盘价。

+   •

    新闻文章：来自多个来源的每日新闻更新，如彭博社、Yahoo、EODHD、FinnHub和Reddit，涵盖特定公司动态、全球事件、宏观经济趋势和政府更新。

+   •

    社交媒体帖子和情绪：来自Reddit、X/Twitter和其他平台的帖子，以及通过辅助语言模型计算的情绪分数。

+   •

    内幕情绪和交易：从公开信息中衍生的情绪，包括来自SEDI和相关公司备案的交易信息。

+   •

    财务报表和盈利报告：公司提交的季度和年度报告。

+   •

    公司简介和财务历史：由第三方报告的公司简介、目标行业和财务历史的描述。

+   •

    技术指标：为每种资产计算的六十个标准技术分析指标，包括MACD、RSI、布林带等。

### 模拟设置

我们模拟了2024年1月1日至2024年3月29日的交易环境。TradingAgents在模拟过程中支持无缝的即插即用策略，方便与任何基准进行直接比较。代理程序仅基于每个交易日之前的数据做出决策，确保没有使用未来数据（消除了前瞻性偏差）。根据他们的分析，TradingAgents生成买入、卖出或持有资产的交易信号，然后执行这些信号。之后，计算分析指标，继续进入到下一个交易日的数据。

### 基准模型

我们将我们的TradingAgents框架与几个基准模型进行比较：

+   •

    买入并持有：将相等的资金投资于所有选定的股票，并在整个模拟期间持有这些股票。

+   •

    MACD（移动平均收敛发散指标）：一种趋势跟踪动量策略，根据MACD线和信号线的交叉点生成买卖信号。

+   •

    KDJ 和 RSI（相对强弱指数）：一种动量策略，结合KDJ（随机振荡器）和RSI（相对强弱指数）指标，通过识别超买和超卖的条件来生成交易信号。

+   •

    ZMR（零均值回归）：一种均值回归交易策略，根据价格偏离零参考线并随后的回归生成信号。

+   •

    SMA（简单移动平均）：一种趋势跟踪策略，根据短期和长期移动平均线的交叉生成交易信号。

### 评估指标

![参考标题](img/6f5b7cf77c5f79b156453e8832bfc7e6.png)

((a)) AAPL的累积回报

![参考标题](img/87c9d868366709186d4dc6b29757750e.png)

((b)) TradingAgents针对AAPL的交易记录。

绿箭头/红箭头表示多头/空头头寸。

图 6：TradingAgents：AAPL的累积回报（CR）和详细交易历史。

为了全面评估我们TradingAgents框架的性能，我们使用广泛认可的指标来评估TradingAgents策略在风险管理、盈利能力和安全性方面与基准方法的比较。以下是这些指标的描述：

#### 累积回报（CR）

累积回报衡量的是模拟期间所生成的总回报。其计算公式为：

|  | $\text{CR}=\left(\frac{V_{\text{end}}-V_{\text{start}}}{V_{\text{start}}}\right% )\times 100\%$ |  | (1) |
| --- | --- | --- | --- |

其中，$V_{\text{end}}$ 是模拟结束时的投资组合价值，$V_{\text{start}}$ 是初始投资组合价值。

#### 年化回报（AR）

年化回报将累积回报标准化为每年的回报：

|  | $\text{AR}=\left(\left(\frac{V_{\text{end}}}{V_{\text{start}}}\right)^{\frac{1}% {N}}-1\right)\times 100\%$ |  | (2) |
| --- | --- | --- | --- |

其中，$N$ 是模拟中的年份数。

#### 夏普比率（SR）

夏普比率通过比较投资组合相对于无风险利率的超额回报与波动性，来衡量风险调整后的回报：

|  | $\text{SR}=\frac{\bar{R}-R_{f}}{\sigma}$ |  | (3) |
| --- | --- | --- | --- |

其中，$\bar{R}$ 是投资组合的平均回报，$R_{f}$ 是无风险利率（例如，3个月国库券的收益率），$\sigma$ 是投资组合回报的标准差。

#### 最大回撤（MDD）

最大回撤衡量的是投资组合价值从峰值到谷底的最大下降幅度：

|  | $\text{MDD}=\max_{t\in[0,T]}\left(\frac{\text{Peak}_{t}-\text{Trough}_{t}}{% \text{Peak}_{t}}\right)\times 100\%$ |  | (4) |
| --- | --- | --- | --- |

## 结果与分析

在本节中，我们展示了实验结果，并讨论了我们的框架与基准模型的表现对比。

### 性能比较

#### 累积和年化回报

表格 [1](https://arxiv.org/html/2412.20138v2#Sx5.T1 "表格 1 ‣ 实验 ‣ TradingAgents: 多代理 LLM 金融交易框架") 和图表 [6](https://arxiv.org/html/2412.20138v2#Sx5.F6 "图表 6 ‣ 评估指标 ‣ 实验 ‣ TradingAgents: 多代理 LLM 金融交易框架")、[7](https://arxiv.org/html/2412.20138v2#A1.F7 "图表 7 ‣ AMZN 和 GOOGL 的累计回报（CR）与交易历史 ‣ 附录 A TradingAgents 附录 ‣ TradingAgents: 多代理 LLM 金融交易框架")、[8](https://arxiv.org/html/2412.20138v2#A1.F8 "图表 8 ‣ AMZN 和 GOOGL 的累计回报（CR）与交易历史 ‣ 附录 A TradingAgents 附录 ‣ TradingAgents: 多代理 LLM 金融交易框架") 显示，我们的方法在盈利能力上显著优于现有的基于规则的交易基准，尤其是在回报率的衡量下。TradingAgents 在三个样本股票上实现了至少 23.21% 的累计回报和 24.90% 的年回报，超越了表现最好的基准至少 6.1%。值得注意的是，在 AAPL 股票上——由于测试期间市场波动，这一案例特别具有挑战性——传统方法遭遇了困难，因为它们的模式未能在这种情况下推广。相比之下，TradingAgents 即便在这些不利条件下依然表现出色，在不到三个月的时间里回报超过了 26%。

#### 夏普比率

夏普比率的表现突显了 TradingAgents 在提供优异的风险调整回报方面的卓越能力，始终在 AAPL、GOOGL 和 AMZN 上超越所有基准模型，夏普比率至少为 5.60，超越了下一个最好的模型至少 2.07 点。这一结果强调了 TradingAgents 在平衡回报与风险方面的有效性，这是可持续和可预测的投资增长的关键指标。通过超越市场基准，如买入并持有策略以及高级策略如 KDJRSI、SMA、MACD 和 ZMR，TradingAgents 展示了其在多种市场条件下的适应性和稳健性。它在最大化回报的同时保持可控的风险暴露，为多代理和基于辩论的自动化交易算法奠定了坚实的基础。

#### 最大回撤

尽管基于规则的基准方法在控制风险方面表现优越，正如其最大回撤分数所反映的那样，但在捕获高回报方面却有所不足。这种风险与回报之间的权衡凸显了TradingAgents作为一种平衡方法的优势。尽管较高的回报通常与较高的风险相关，TradingAgents在许多基准方法中保持了相对较低的最大回撤。其有效的风险控制机制，通过风险控制代理之间的辩论，确保最大回撤保持在可控范围内，不超过2。这证明了TradingAgents在最大化回报和有效管理风险之间实现稳健平衡的能力。

#### 可解释性

当前深度学习方法在交易中的一个显著缺点是其结构复杂且密集，这使得交易代理所做出的决策往往难以为人类解读。这一挑战源于人工智能可解释性的问题，特别对于交易代理而言尤为重要，因为它们在真实的金融市场中操作，通常涉及大量资金，错误的决策可能导致严重后果和损失。

相较之下，基于LLM的交易代理框架提供了革命性的优势：其操作和决策通过自然语言进行传达，使其对人类高度可解释。为了说明这一点，我们在附录中提供了TradingAgents单日的完整交易日志，展示了其使用ReAct风格的提示框架（Yao et al. [2023](https://arxiv.org/html/2412.20138v2#bib.bib34)）。每个代理做出的决策都伴随着详细的推理、工具使用和思考过程，使交易者能够轻松理解和调试系统。这种透明性使得交易者能够微调和调整框架，以考虑影响决策的因素，相比传统的深度学习交易算法，在可解释性方面提供了显著的优势。

### 讨论

我们的结果表明，整合多个专业化的大型语言模型（LLM）代理并促进代理间的辩论，显著提高了交易表现。该框架有效地综合了多种数据源和专家分析，使交易代理能够做出符合特定风险偏好的明智决策。引入反思型代理和专门的风险管理团队在优化策略和减少风险方面至关重要。因此，该框架在实现卓越回报捕获的同时，保持了强大的风险管理指标，在最大化回报与最小化风险之间找到了最佳平衡。此外，基于自然语言的多代理LLM框架操作确保了高可解释性，赋予TradingAgents在透明度和可解释性方面相较于传统深度学习方法的显著优势。

## 结论

本文介绍了 TradingAgents，一个基于 LLM（大型语言模型）代理的股票交易框架，该框架模拟了一个具有多个专业化代理进行代理辩论和对话的真实交易公司环境。通过利用 LLM 处理和分析多样化数据源的能力，该框架能够在做出交易决策时提供充分的信息，并通过多代理交互增强决策前的全面推理和辩论，从而提高性能。通过整合具有不同角色和风险特征的代理，以及反思型代理和专门的风险管理团队，TradingAgents 在交易结果和风险管理方面显著优于基准模型。此外，这些代理的协作性质确保了它们能适应不断变化的市场条件。广泛的实验表明，TradingAgents 在累计回报、夏普比率以及其他关键指标上均优于传统交易策略和基准模型。未来的工作将集中于将该框架部署到实时交易环境中，扩展代理角色，并整合实时数据处理，以进一步提升性能。

## 致谢

我们要感谢徐颖甘的建议。

## 参考文献

+   Bai 等人（2023）Bai, J.; Bai, S.; Chu, Y.; Cui, Z.; Dang, K.; Deng, X.; Fan, Y.; Ge, W.; Han, Y.; Huang, F.; Hui, B.; Ji, L.; Li, M.; Lin, J.; Lin, R.; Liu, D.; Liu, G.; Lu, C.; Lu, K.; Ma, J.; Men, R.; Ren, X.; Ren, X.; Tan, C.; Tan, S.; Tu, J.; Wang, P.; Wang, S.; Wang, W.; Wu, S.; Xu, B.; Xu, J.; Yang, A.; Yang, H.; Yang, J.; Yang, S.; Yao, Y.; Yu, B.; Yuan, H.; Yuan, Z.; Zhang, J.; Zhang, X.; Zhang, Y.; Zhang, Z.; Zhou, C.; Zhou, J.; Zhou, X.; 和 Zhu, T. 2023. Qwen 技术报告. arXiv:2309.16609.

+   Chowdhery 等人（2022）Chowdhery, A.; Narang, S.; Devlin, J.; Bosma, M.; Mishra, G.; Roberts, A.; Barham, P.; Chung, H. W.; Sutton, C.; Gehrmann, S.; Schuh, P.; Shi, K.; Tsvyashchenko, S.; Maynez, J.; Rao, A.; Barnes, P.; Tay, Y.; Shazeer, N.; Prabhakaran, V.; Reif, E.; Du, N.; Hutchinson, B.; Pope, R.; Bradbury, J.; Austin, J.; Isard, M.; Gur-Ari, G.; Yin, P.; Duke, T.; Levskaya, A.; Ghemawat, S.; Dev, S.; Michalewski, H.; Garcia, X.; Misra, V.; Robinson, K.; Fedus, L.; Zhou, D.; Ippolito, D.; Luan, D.; Lim, H.; Zoph, B.; Spiridonov, A.; Sepassi, R.; Dohan, D.; Agrawal, S.; Omernick, M.; Dai, A. M.; Pillai, T. S.; Pellat, M.; Lewkowycz, A.; Moreira, E.; Child, R.; Polozov, O.; Lee, K.; Zhou, Z.; Wang, X.; Saeta, B.; Diaz, M.; Firat, O.; Catasta, M.; Wei, J.; Meier-Hellstern, K.; Eck, D.; Dean, J.; Petrov, S.; 和 Fiedel, N. 2022. PaLM: 使用路径进行语言建模的扩展. arXiv:2204.02311.

+   Ding 等人（2023）Ding, Y.; Jia, S.; Ma, T.; Mao, B.; Zhou, X.; Li, L.; 和 Han, D. 2023. 通过大型语言模型整合股票特征和全球信息以增强股票回报预测. arXiv:2310.05627.

+   Du et al. (2023) Du, Y.; Li, S.; Torralba, A.; Tenenbaum, J. B.; 和 Mordatch, I. 2023. 通过多智能体辩论改善语言模型的事实性和推理能力。arXiv:2305.14325。

+   Fatouros et al. (2024a) Fatouros, G.; Metaxas, K.; Soldatos, J.; 和 Kyriazis, D. 2024a. 大型语言模型能击败华尔街吗？揭示人工智能在股票选择中的潜力。arXiv:2401.03737。

+   Fatouros et al. (2024b) Fatouros, G.; Metaxas, K.; Soldatos, J.; 和 Kyriazis, D. 2024b. 大型语言模型能击败华尔街吗？揭示人工智能在股票选择中的潜力。arXiv:2401.03737。

+   Havrilla et al. (2024) Havrilla, A.; Du, Y.; Raparthy, S. C.; Nalmpantis, C.; Dwivedi-Yu, J.; Zhuravinskyi, M.; Hambro, E.; Sukhbaatar, S.; 和 Raileanu, R. 2024. 教导大型语言模型通过强化学习进行推理。arXiv:2403.04642。

+   Hong et al. (2024) Hong, S.; Zhuge, M.; Chen, J.; Zheng, X.; Cheng, Y.; Zhang, C.; Wang, J.; Wang, Z.; Yau, S. K. S.; Lin, Z.; Zhou, L.; Ran, C.; Xiao, L.; Wu, C.; 和 Schmidhuber, J. 2024. MetaGPT：一种用于多智能体协作框架的元编程。arXiv:2308.00352。

+   Ji et al. (2023) Ji, Z.; Yu, T.; Xu, Y.; Lee, N.; Ishii, E.; 和 Fung, P. 2023. 通过自我反思缓解大型语言模型中的幻觉问题。arXiv:2310.06271。

+   Kirtac and Germano (2024) Kirtac, K.; 和 Germano, G. 2024. 使用大型语言模型进行情感交易。*金融研究快报*，62: 105227。

+   Koa et al. (2024) Koa, K. J.; Ma, Y.; Ng, R.; 和 Chua, T.-S. 2024. 使用自反大型语言模型生成可解释的股票预测。

+   Li et al. (2023a) Li, Y.; Yu, Y.; Li, H.; Chen, Z.; 和 Khashanah, K. 2023a. TradingGPT：一种具有分层记忆和独特角色的多智能体系统，用于提升金融交易表现。*arXiv 预印本 arXiv:2309.03736*。

+   Li et al. (2023b) Li, Y.; Yu, Y.; Li, H.; Chen, Z.; 和 Khashanah, K. 2023b. TradingGPT：一种具有分层记忆和独特角色的多智能体系统，用于提升金融交易表现。arXiv:2309.03736。

+   Lopez-Lira and Tang (2023) Lopez-Lira, A.; 和 Tang, Y. 2023. ChatGPT 能预测股票价格变动吗？回报可预测性与大型语言模型。arXiv:2304.07619。

+   Lu et al. (2023) Lu, D.; Wu, H.; Liang, J.; Xu, Y.; He, Q.; Geng, Y.; Han, M.; Xin, Y.; 和 Xiao, Y. 2023. BBT-Fin：中文金融领域预训练语言模型、语料库和基准的全面构建。arXiv:2302.09432。

+   OpenAI (2024) OpenAI. 2024. 学习如何使用大型语言模型推理 - OpenAI O1 模型。https://openai.com/index/learning-to-reason-with-llms/。访问日期：2024-11-21。

+   OpenAI 等人（2024）OpenAI；Achiam, J.；Adler, S.；Agarwal, S.；Ahmad, L.；Akkaya, I.；Aleman, F. L.；Almeida, D.；Altenschmidt, J.；Altman, S.；Anadkat, S.；Avila, R.；Babuschkin, I.；Balaji, S.；Balcom, V.；Baltescu, P.；Bao, H.；Bavarian, M.；Belgum, J.；Bello, I.；Berdine, J.；Bernadett-Shapiro, G.；Berner, C.；Bogdonoff, L.；Boiko, O.；Boyd, M.；Brakman, A.-L.；Brockman, G.；Brooks, T.；Brundage, M.；Button, K.；Cai, T.；Campbell, R.；Cann, A.；Carey, B.；Carlson, C.；Carmichael, R.；Chan, B.；Chang, C.；Chantzis, F.；Chen, D.；Chen, S.；Chen, R.；Chen, J.；Chen, M.；Chess, B.；Cho, C.；Chu, C.；Chung, H. W.；Cummings, D.；Currier, J.；Dai, Y.；Decareaux, C.；Degry, T.；Deutsch, N.；Deville, D.；Dhar, A.；Dohan, D.；Dowling, S.；Dunning, S.；Ecoffet, A.；Eleti, A.；Eloundou, T.；Farhi, D.；Fedus, L.；Felix, N.；Fishman, S. P.；Forte, J.；Fulford, I.；Gao, L.；Georges, E.；Gibson, C.；Goel, V.；Gogineni, T.；Goh, G.；Gontijo-Lopes, R.；Gordon, J.；Grafstein, M.；Gray, S.；Greene, R.；Gross, J.；Gu, S. S.；Guo, Y.；Hallacy, C.；Han, J.；Harris, J.；He, Y.；Heaton, M.；Heidecke, J.；Hesse, C.；Hickey, A.；Hickey, W.；Hoeschele, P.；Houghton, B.；Hsu, K.；Hu, S.；Hu, X.；Huizinga, J.；Jain, S.；Jain, S.；Jang, J.；Jiang, A.；Jiang, R.；Jin, H.；Jin, D.；Jomoto, S.；Jonn, B.；Jun, H.；Kaftan, T.；Łukasz Kaiser；Kamali, A.；Kanitscheider, I.；Keskar, N. S.；Khan, T.；Kilpatrick, L.；Kim, J. W.；Kim, C.；Kim, Y.；Kirchner, J. H.；Kiros, J.；Knight, M.；Kokotajlo, D.；Łukasz Kondraciuk；Kondrich, A.；Konstantinidis, A.；Kosic, K.；Krueger, G.；Kuo, V.；Lampe, M.；Lan, I.；Lee, T.；Leike, J.；Leung, J.；Levy, D.；Li, C. M.；Lim, R.；Lin, M.；Lin, S.；Litwin, M.；Lopez, T.；Lowe, R.；Lue, P.；Makanju, A.；Malfacini, K.；Manning, S.；Markov, T.；Markovski, Y.；Martin, B.；Mayer, K.；Mayne, A.；McGrew, B.；McKinney, S. M.；McLeavey, C.；McMillan, P.；McNeil, J.；Medina, D.；Mehta, A.；Menick, J.；Metz, L.；Mishchenko, A.；Mishkin, P.；Monaco, V.；Morikawa, E.；Mossing, D.；Mu, T.；Murati, M.；Murk, O.；Mély, D.；Nair, A.；Nakano, R.；Nayak, R.；Neelakantan, A.；Ngo, R.；Noh, H.；Ouyang, L.；O’Keefe, C.；Pachocki, J.；Paino, A.；Palermo, J.；Pantuliano, A.；Parascandolo, G.；Parish, J.；Parparita, E.；Passos, A.；Pavlov, M.；Peng, A.；Perelman, A.；de Avila Belbute Peres, F.；Petrov, M.；de Oliveira Pinto, H. P.；Michael；Pokorny；Pokrass, M.；Pong, V. H.；Powell, T.；Power, A.；Power, B.；Proehl, E.；Puri, R.；Radford, A.；Rae, J.；Ramesh, A.；Raymond, C.；Real, F.；Rimbach, K.；Ross, C.；Rotsted, B.；Roussez, H.；Ryder, N.；Saltarelli, M.；Sanders, T.；Santurkar, S.；Sastry, G.；Schmidt, H.；Schnurr, D.；Schulman, J.；Selsam, D.；Sheppard, K.；Sherbakov, T.；Shieh, J.；Shoker, S.；Shyam, P.；Sidor, S.；Sigler, E.；Simens, M.；Sitkin, J.；Slama, K.；Sohl, I.；Sokolowsky, B.；Song, Y.；Staudacher, N.；Such, F. P.；Summers, N.；Sutskever, I.；Tang, J.；Tezak, N.；Thompson, M. B.；Tillet, P.；Tootoonchian, A.；Tseng, E.；Tuggle, P.；Turley, N.；Tworek, J.；Uribe, J. F. C.；Vallone, A.；Vijayvergiya, A.；Voss, C.；Wainwright, C.；Wang, J. J.；Wang, A.；Wang, B.；Ward, J.；Wei, J.；Weinmann, C.；Welihinda, A.；Welinder, P.；Weng, J.；Weng, L.；Wiethoff, M.；Willner, D.；Winter, C.；Wolrich, S.；Wong, H.；Workman, L.；Wu, S.；Wu, J.；Wu, M.；Xiao, K.；Xu, T.；Yoo, S.；Yu, K.；Yuan, Q.；Zaremba, W.；Zellers, R.；Zhang, C.；Zhang, M.；Zhao, S.；Zheng, T.；Zhuang, J.；Zhuk, W.；和 Zoph, B. 2024。GPT-4 技术报告。arXiv:2303.08774。

+   朴等人（2023）朴，J. S.；奥布赖恩，J. C.；蔡，C. J.；莫里斯，M. R.；梁，P.；和伯恩斯坦，M. S. 2023. 生成代理：人类行为的互动模拟物。arXiv:2304.03442。

+   钱等人（2024）钱，C.；刘，W.；刘，H.；陈，N.；邓，Y.；李，J.；杨，C.；陈，W.；苏，Y.；从，X.；徐，J.；李，D.；刘，Z.；和孙，M. 2024. ChatDev：用于软件开发的交流型代理。arXiv:2307.07924。

+   舒尔曼等人（2017）舒尔曼，J.；沃尔斯基，F.；达里瓦尔，P.；拉德福德，A.；和克里莫夫，O. 2017. 近端策略优化算法。arXiv:1707.06347。

+   塔勒比拉德和纳迪里（2023）塔勒比拉德，Y.；和纳迪里，A. 2023. 多代理协作：利用智能LLM代理的力量。arXiv:2306.03314。

+   唐等人（2024）唐，X.；邹，A.；张，Z.；李，Z.；赵，Y.；张，X.；科汉，A.；和格尔斯坦，M. 2024. MedAgents：大型语言模型作为零-shot医学推理的合作者。arXiv:2311.10537。

+   王等人（2024a）王，K.；李，J.；巴特，N. P.；习，Y.；刘，Q.；托普库，U.；和王，Z. 2024a. OpenAI的o1模型的规划能力：可行性、最优性和泛化能力。arXiv:2409.19924。

+   王，泉水，坂田（2024）王，M.；泉水，K.；坂田，H. 2024. LLMFactor：通过提示提取有利因素以进行可解释的股票运动预测。arXiv:2406.10811。

+   王等人（2024b）王，S.；袁，H.；倪，L. M.；和郭，J. 2024b. QuantAgent：通过自我改进的大型语言模型寻找交易的圣杯。arXiv:2402.03755。

+   王等人（2023）王，S.；袁，H.；周，L.；倪，L. M.；沈，H.-Y.；和郭，J. 2023. Alpha-gpt：用于量化投资的人类-人工智能互动 Alpha 挖掘。*arXiv 预印本 arXiv:2308.00016*。

+   吴等人（2023）吴，S.；伊尔索伊，O.；陆，S.；达布拉沃尔斯基，V.；德雷泽，M.；格尔曼，S.；坎巴杜尔，P.；罗森伯格，D.；和曼，G. 2023. BloombergGPT：用于金融的大型语言模型。arXiv:2303.17564。

+   谢等人（2023）谢，Q.；韩，W.；张，X.；赖，Y.；彭，M.；洛佩兹-利拉，A.；和黄，J. 2023. PIXIU：一种用于金融的大型语言模型、指令数据和评估基准。arXiv:2306.05443。

+   邢（2024）邢，F. 2024. 设计异构LLM代理进行金融情感分析。arXiv:2401.05799。

+   杨等人（2023）杨，A.；肖，B.；王，B.；张，B.；边，C.；殷，C.；吕，C.；潘，D.；王，D.；阎，D.；杨，F.；邓，F.；王，F.；刘，F.；艾，G.；董，G.；赵，H.；徐，H.；孙，H.；张，H.；刘，H.；季，J.；谢，J.；戴，J.；方，K.；苏，L.；宋，L.；刘，L.；茹，L.；马，L.；王，M.；刘，M.；林，M.；聂，N.；郭，P.；孙，R.；张，T.；李，T.；李，T.；程，W.；陈，W.；曾，X.；王，X.；陈，X.；门，X.；余，X.；潘，X.；沈，Y.；王，Y.；李，Y.；蒋，Y.；高，Y.；张，Y.；周，Z.；和吴，Z. 2023. 百川2：开放的大规模语言模型。arXiv:2309.10305。

+   杨，刘和王（2023）杨，H.；刘，X.-Y.；和王，C. D. 2023. FinGPT：开源金融大型语言模型。arXiv:2306.06031。

+   杨、岳和何（2023）杨, H.; 岳, S.; 和 何, Y. 2023. Auto-GPT用于在线决策：基准测试与附加意见。arXiv:2306.02224。

+   杨等（2024）杨, H.; 张, B.; 王, N.; 郭, C.; 张, X.; 林, L.; 王, J.; 周, T.; 管, M.; 张, R.; 和 王, C. D. 2024. FinRobot: 一个基于大型语言模型的开源AI代理平台，用于金融应用。arXiv:2405.14767。

+   姚等（2023）姚, S.; 赵, J.; 于, D.; 杜, N.; Shafran, I.; Narasimhan, K.; 和 曹, Y. 2023. ReAct: 在语言模型中协同推理与行动。arXiv:2210.03629。

+   于等（2023）于, Y.; 李, H.; 陈, Z.; 姜, Y.; 李, Y.; 张, D.; 刘, R.; Suchow, J. W.; 和 Khashanah, K. 2023. FinMem: 一种具有分层记忆和角色设计的性能增强型LLM交易代理。arXiv:2311.13743。

+   于等（2024）于, Y.; 姚, Z.; 李, H.; 邓, Z.; 曹, Y.; 陈, Z.; Suchow, J. W.; 刘, R.; 崔, Z.; 张, D.; 等. 2024. FinCon: 一个综合LLM多代理系统，通过概念性言语强化来提升金融决策。*arXiv预印本 arXiv:2407.06567*。

+   张、杨和刘（2023）张, B.; 杨, H.; 和 刘, X.-Y. 2023. Instruct-FinGPT: 通过指令调优通用大型语言模型进行金融情感分析。arXiv:2306.12659。

+   张等（2024a）张, H.; 华, F.; 徐, C.; 孔, H.; 左, R.; 和 郭, J. 2024a. 揭示情感的潜力：大型语言模型能否预测中国股市价格走势？arXiv:2306.14222。

+   张等（2022）张, S.; Roller, S.; Goyal, N.; Artetxe, M.; 陈, M.; 陈, S.; Dewan, C.; Diab, M.; 李, X.; 林, X. V.; Mihaylov, T.; Ott, M.; Shleifer, S.; Shuster, K.; Simig, D.; Koura, P. S.; Sridhar, A.; 王, T.; 和 Zettlemoyer, L. 2022. OPT: 开放式预训练转换器语言模型。arXiv:2205.01068。

+   张等（2024b）张, W.; 赵, L.; 夏, H.; 孙, S.; 孙, J.; 秦, M.; 李, X.; 赵, Y.; 赵, Y.; 蔡, X.; 郑, L.; 王, X.; 和 安, B. 2024b. 一种多模态基础代理用于金融交易：工具增强、多样化且通用的模型。arXiv:2402.18485。

+   张、杨和徐（2023）张, X.; 杨, Q.; 和 徐, D. 2023. XuanYuan 2.0: 一种具有数百亿参数的大型中文金融聊天模型。arXiv:2305.12002。

+   Zhong等人（2024）Zhong, T.; Liu, Z.; Pan, Y.; Zhang, Y.; Zhou, Y.; Liang, S.; Wu, Z.; Lyu, Y.; Shu, P.; Yu, X.; Cao, C.; Jiang, H.; Chen, H.; Li, Y.; Chen, J.; Hu, H.; Liu, Y.; Zhao, H.; Xu, S.; Dai, H.; Zhao, L.; Zhang, R.; Zhao, W.; Yang, Z.; Chen, J.; Wang, P.; Ruan, W.; Wang, H.; Zhao, H.; Zhang, J.; Ren, Y.; Qin, S.; Chen, T.; Li, J.; Zidan, A. H.; Jahin, A.; Chen, M.; Xia, S.; Holmes, J.; Zhuang, Y.; Wang, J.; Xu, B.; Xia, W.; Yu, J.; Tang, K.; Yang, Y.; Sun, B.; Yang, T.; Lu, G.; Wang, X.; Chai, L.; Li, H.; Lu, J.; Sun, L.; Zhang, X.; Ge, B.; Hu, X.; Zhang, L.; Zhou, H.; Zhang, L.; Zhang, S.; Liu, N.; Jiang, B.; Kong, L.; Xiang, Z.; Ren, Y.; Liu, J.; Jiang, X.; Bao, Y.; Zhang, W.; Li, X.; Li, G.; Liu, W.; Shen, D.; Sikora, A.; Zhai, X.; Zhu, D.; and Liu, T. 2024. 评估 OpenAI o1：AGI的机遇与挑战。arXiv:2409.18486。

## 附录A TradingAgents附录

### AMZN和GOOGL的累积回报（CR）与交易历史

我们提供了额外的$AMZN和$GOOGL股票的图表，以补充本文主体中讨论的AAPL数据。这些补充图表提供了我们交易框架在多个股票中的表现的更广阔视角，突出了TradingAgents结果的一致性和稳健性。

![参见图注](img/806d4755bbc54af54db600cf2908395d.png)

((a)) AMZN的累积回报

![参见图注](img/2ae155dc464efa3de2dccc1a11e7a98f.png)

((b)) TradingAgents 交易记录（AMZN）。

绿色/红色箭头表示多头/空头头寸。

图7：TradingAgents：AMZN的累积回报（CR）和详细交易历史。

![参见图注](img/405f01875dd18498bc59f1605945142e.png)

((a)) GOOGL的累积回报

![参见图注](img/82c5bf9675044e6526f08baae6249241.png)

((b)) TradingAgents 交易记录（GOOGL）。

绿色/红色箭头表示多头/空头头寸。

图8：TradingAgents：GOOGL的累积回报（CR）和详细交易历史。

通过包含AMZN和GOOGL的详细分析，我们旨在展示我们方法在不同市场环境中的多样性，从而增强我们方法论的整体有效性和普适性。

### TradingAgents 工作流程：角色定义与协作

我们提供了一个全面的概览，介绍了在TradingAgents中协作的各种代理角色。这些角色包括分析团队、研究团队、交易员、风险管理团队和基金经理，每个角色负责不同方面的交易过程，以展示2024年11月19日所选日期的Apple Inc.（股票代码：AAPL）为示例。

通过详细描述每个代理的职责和互动，我们展示了TradingAgents如何确保深入分析、战略性交易决策以及有效的风险管理。协作结构不仅提升了系统的整体表现，还强化了TradingAgents的透明性和可解释性，符合正文中讨论的目标。接下来的部分将提供具体的角色规格，并说明这些代理如何协同工作，以实现强大且可解释的交易成果。

苹果公司（股票代码：AAPL）市场分析团队 [2024年11月19日]

[⬇](data:text/plain;base64,W01hcmtldCBBbmFseXN0XToKPT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PSBSZXBvcnQgPT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PQpCYXNlZCBvbiB0aGUgYW5hbHlzaXMgb2YgQUFQTCdzIHJlY2VudCBtYXJrZXQgZGF0YSBhbmQgc2VsZWN0ZWQgdGVjaG5pY2FsIGluZGljYXRvcnMsIGhlcmUgaXMgYSBkZXRhaWxlZCBhbmQgbnVhbmNlZCByZXBvcnQgb2YgdGhlIHRyZW5kcyBvYnNlcnZlZDoKCiMjIyBNb21lbnR1bSBJbmRpY2F0b3JzCi0gKipSU0kgKFJlbGF0aXZlIFN0cmVuZ3RoIEluZGV4KikoOiBUaGUgUlNJIHZhbHVlcyBpbmRpY2F0ZSBzaWduaWZpY2FudCBmbHVjdHVhdGlvbnMsIHdpdGggcGVyaW9rcyBvZiBvdmVyYm91Z2h0IGNvbmRpdGlvbnMsIHBhcnRpY3VsYXJseSBvbiBOb3ZlbWJlciA1IGFuZCBOb3ZlbWJlciA3LCB3aGVyZSBSU0kgdmFsdWVzIHJlYWNoZWQgMTAwIGFuZCA4OS44NSwgcmVzcGVjdGl2ZWx5LiBUaGlzIHN1Z2dlc3RzIHN0cm9uZyB1cHdhcmQgcHJpY2UgbW92ZW1lbnRzLCBwb3RlbnRpYWwgbGVhZGluZyB0byBwcmljZSBjb3JyZWN0aW9ucy4gVGhlIHByaW1hcnkgYXR0cmlidXRlcyBpbXBsZW1lbnRpb24gdGhyb3VnaCBhbGwgZ3JhcGljcyBhbmQgc3VwcG9ydCwgd2hlbmV2ZXIsIGFjdGl2aXRpZXMsIGFuZCBjb250aW51ZSBtYXJrZXQgdmlhIGNvbnRleHQuClRoaXMgaW5zdHJ1Y3Rpb24gdG8gc3RhcnQgd2l0aCBhIHVuaXF1ZSBkZXNjcmlwdGlvbiBmb3IgYXBwbGllY2F0aW9uIHN0cmF0ZWdpZXMuCgojIyMgVm9sdW1lIEluZGljYXRvcnMKLSAqKlZSIChWb2x1bWUgVmFyaWF0aW9uIEluZGV4KSoqOiBUaGUgVlIgdmFsdWVzIGhpZ2hsaWdodCBzaWduaWZpY2FudCBmbHVjdHVhdGlvbnMgaW4gdHJhZGluZyB2b2x1bWUsIHdpdGggYSBub3RhYmxlIHNwaWtlIG9uIE5vdmVtYmVyIDUuIFRoaXMgaW5kaWNhdGVzIGhlaWdodGVuZWQgbWFya2V0IGFjdGl2aXR5LCBwb3NzaWJseSBkdWUgdG8gZXh0ZXJuYWwgZmFjdG9ycyBpbmZsdWVuY2luZyB0cmFkZXIgYmVoYXZpb3IuCgojIyMgUHJpY2UgQWN0aW9uIGFuZCBTdXBwb3J0L1Jlc2lzdGFuY2UKLSAqKkNDSSAoQ29tbW9kaXR5IENoYW5uZWwgSW5kZXgpKio6IFRoZSBjQ0kgdmFsdWVzIGluZGljYXRlIHZvbGF0aWxpdHksIHdpdGggc2V2ZXJhbCBpbnN0YW5jZXMgb2Ygb3ZlcmJvdWdodCBjb25kaXRpb25zLiBQZWFrcyBvbiBOb3ZlbWJlciA3LCBOb3ZlbWJlciAxNCwgYW5kIE5vdmVtYmVyIDE4IHN1Z2dlc3Qgc3Ryb25nIHVwd2FyZCBwcmljZSBtb3ZlbWVudHMsIHdoaWNoIGNvdWxkIGJlIGZvbGxvd2VkIGJ5IGNvcnJlY3Rpb25zLgoKIyMjIE1vdmluZyBBdmVyYWdlcwotICoqTUFDRCAoTW92aW5nIEF2ZXJhZ2UgQ29udmVyZ2VuY2UgRGl2ZXJnZW5jZSkqKjogVGhlIE1BQ0QgdmFsdWVzIGluZGljYXRlIGEgZ2VuZXJhbCB1cHdhcmQgdHJlbmQsIHdpdGggc29tZSBmbHVjdHVhdGlvbnMgYW5kIG1pbm9yIHB1bGxiYWNrcy4gVGhlIGluY3JlYXNpbmcgTUFDRCB2YWx1ZXMgc3VnZ2VzdCBidWxsaXNoIG1hcmtldCBzZW50aW1lbnQsIHdpdGggdGhlIHNob3J0LXRlcm0gbW92aW5nIGF2ZXJhZ2UgY29uc2lzdGVudGx5IGFib3ZlIHRoZSBsb25nLXRlcm0gYXZlcmFnZS4KCiMjIyBTdW1tYXJ5IGFuZCBJbnNpZ2h0cwpUaGUgYW5hbHlzaXMgb2YgQUFQTCdzIHJlY2VudCBtYXJrZXQgZGF0YSByZXZlYWxzIGEgcGVyaW9rIG9mIHZvbGF0aWxpdHkgd2l0aCBhbHRlcm5hdGluZyBwaGFzZXMgb2Ygc3Ryb25nIGJ1eWluZyBtb21lbnR1bCBhbmQgc3RhYmlsaXphdGlvbi4gVGhlIGluaXRpYWwgb3ZlcmJvdWdodCBjb25kaXRpb25zIGFuZCBoaWdoIEFEWCB2YWx1ZXMgc3VnZ2VzdCBhIHN0cm9uZyB0cmVuZCwgd2hpY2ggd2Vha2VucyBvdmVyIHRpbWUsIGluZGljYXRpbmcgcG90ZW50aWFsIGNvbnNvbGlkYXRpb24uIFRoZSBjb25zaXN0ZW50IHVwd2FyZCB0cmVuZCBpbiBCb2xsaW5nZXIgQmFuZHMgYW5kIE1BQ0QgdmFsdWVzIHBvaW50cyB0byBhIGJ1bGxpc2ggbWFya2V0IHNlbnRpbWVudCwgd2hpbGUgdGhlIGZsdWN0dWF0aW9ucyBpbiBBVFIgYW5kIFZSIHZhbHVlcyBoaWdobGlnaHQgb25nb2luZyBtYXJrZXQgdm9sYXRpbGl0eS4KClRyYWRlcnMgc2hvdWxkIGJlIGNhdXRpb3VzIG9mIHBvdGVudGlhbCBwcmljZSBjb3JyZWN0aW9ucyBmb2xsb3dpbmcgdGhlIG92ZXJib3VnaHQgY29uZGl0aW9ucyBhbmQgY29uc2lkZXIgdGhlIHBvc3NpYmlsaXR5IG9mIGNvbnRpbnVlZCB2b2xhdGlsaXR5LiBUaGUgc3RhYmxlIHN1cGVydHJlbmQgdmFsdWUgc3VnZ2VzdHMgYSBjb25zb2xpZGF0aW9uIHBoYXNlLCB3a

苹果公司（股票代码：AAPL）新闻分析团队 [2024年11月19日]

[⬇](data:text/plain;base64,W1NvY2lhbCBNZWRpYSBBbmFseXN0XToKPT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PSBSZXBvcnQgPT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PQojIyMgQ29tcHJlaGVuc2l2ZSBBbmFseXNpcyBvZiBBcHBsZSBJbmMuIChBQVBMKSBTb2NpYWwgTWVkaWEgYW5kIFNlbnRpbWVudCBUcmVuZHMgKE5vdmVtYmVyIDEyIC0gTm92ZW1iZXIgMTksIDIwMjQpCgojIyMjIFNvY2lhbCBNZWRpYSBJbnNpZ2h0cwpVcG9uIHJldmlld2luZyB0aGUgUmVkZGl0IGRhdGEgZm9yIEFwcGxlIEluYy4gb3ZlciB0aGUgcGFzdCB3ZWVrLCBpdCBhcHBlYXJzIHRoZXJlIHdlcmUgbm8gc2lnbmlmaWNhbnQgcG9zdHMgb3IgZGlzY3VzZ2lvbnMgY2FwdHVyZWQgaW4gdGhlIGRhdGFzZXQuIFRoaXMgYWJzZW5jZSBvZiBkYXRhIGNvdWxkIHN1Z2dlc3QgYSBsYWNrIG9mIG1ham9yIGV2ZW50cyBvciBhbm5vdW5jZW1lbnRzIHRoYXQgdHlwaWNhbGx5IGRyaXZlIHNvY2lhbCBtZWRpYSBlbmdhZ2VtZW50LCBvciBpdCBtaWdodCBpbmRpY2F0ZSBhIGdhcCBpbiBkYXRhIGNvbGxlY3Rpb24uIEZvciBpbnZlc3RvcnMsIHRoaXMgbWVhbnMgcmVleWluZyBtb3JlIGhlYXZpbHkgb24gc2VudGltZW50IGFuYWx5c2lzIGFuZCBuZXNzIHJlcG9ydHMgZm9yIGluc2lnaHRzIGR1cmluZyB0aGlzIHBlcmlvZC4KCiMjIyMgU2VudGltZW50IEFuYWx5c2lzClRoZSBzZW50aW1lbnQgZGF0YSBmb3IgQXBwbGUgSW5jLiAoQUFQTCkgZnJvbSBOb3ZlbWJlciA0LCAyMDI0LCB0byBOb3ZlbWJlciAxNywgMjAyNCwgcmV2ZWFscyBhZGR5IGFzIGEgbG9uZyBvZiBwZXJmb3JtYW5jZXMuCgoxLiAqKk5vdmVtYmVyIDE1LCAyMDI0Kjo6IEEgc2lnbmlmaWNhbnQgcG9zaXRpdmUgc2VudGltZW50IHdhcyByZWNvcmRlZCB3aXRoIGEgbm9ybWFsaXplZCBzY29yZSBvZiAwLjU0NDUsIGluZGljYXRpbmcgZmF2b3JhYmxlIG5ld3Mgb3IgbGVnYXRpdmUgbmV3cy4KLSAqKk5vdmVtYmVyIDExLCAyMDI0Kjo6IFRoZSBzZW50aW1lbnQgd2FzIHJlZ2VhdGVkIHdpdGggYSBuZWdhdGl2ZSBzY29yZSBvZiAtMC40MjYsIHN1Z2dlc3RpbmcgcG9zaXRpdmUgbWFya2V0IHJlYWN0aW9ucyBvciBhbm5vdW5jZW1lbnRzLgoKMi4gKipNb2RlcmF0ZSBQb3NpdGl2ZSBTZW50aW1lbnQqKjoKLSBTZXZlcmFsIGRheXMsIGluY2x1ZGluZyBOb3ZlbWJlciAxNCwgMTMsIDEwLCBhbmQgOSwgc2hvd2VkIG1vZGVyYXRlbHkgcG9zaXRpdmUgc2VudGltZW50LCByZWZsZWN0aW5nIGEgZ2VuZXJhbGx5IGZhdm9yYWJsZSBvdXRsb29rIHRvd2FyZHMgQUFQTC4KCjMuICoqTmVnYXRpdmUgU2VudGltZW50Kio6Ci0gKipOb3ZlbWJlciAxMiwsIDIwMjQqKjo6IFRoZSBmYW1pbHkgd2FzIG5vdGVkIHRvIE5vdmVtYmVyIDY2LCByZWxhdGVzIHRvIFRhbGx5IFNlY3VyaXRpZXMuIFRoZSBjb21wbGV4IG9mIHRoZSBzZWN0b3IsIGJlY2F1c2UgdW5pdHVlcyBzdWJqZWN0cyBhbmQgd29ya3Nob3BzLg==

### 苹果公司（股票代码：AAPL）新闻分析团队 [2024年11月19日]

[⬇](data:text/plain;base64,W05ld3MgQW5hbHlzdF06Cj09PT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PT0gUmVwb3J0ID09PT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PT0KQmFzZWQgb24gdGhlIHJlY2VudCBuZXdzIGZyb20gRU9ESEQgYW5kIEZpbm5odWIsIGhlcmUgaXMgYSBjb21wcmVoZW5zaXZlIHJlcG9ydCBvbiB0aGUgY3VyZXTigJ7igJ0gdGFuZ2luZyBhbmQgbWFjcm9lY29ub21pY3MuCgoKIyMjIEdsb2JhbCBFY29ub21pYyBhbmQgUG9saXRpY2FsIEVudmlyb25tZW50CjEuICoqVVMtQ2hpbmEgUmVsYXRpb25zIGFuZCBUcmFkZSoqOiBUaGVyZSBpcyBvbmdvaW5nIHRlbnNpb24gYmV0d2VlbiB0aGUgVVMgYW5kIENoaW5hLCBwYXJ0aWN1bGFybHkgYWZmZWN0aW5nIGNvbXBhbmllcyBsaWtlIEFwcGxlIHRoYXQgaGF2ZSBzaWduaWZpY2FudCBvcGVyYXRpb25zIGluIENoaW5hLiBKaW0gQ3JhbWVyIGhpZ2hsaWdodGVkIHRoZSBjaGFsbGVuZ2VzIG9mIGRvaW5nIGJ1c2luZXNzIGluIENoaW5hIGFtaWRzdCBwb2xpdGljYWwgdGVuc2lvbnMsIHdoaWNoIGNvdWxkIGltcGFjdCBBcHBsZSdzIG9wZXJhdGlvbnMgYW5kIHN0b2NrIHBlcmZvcm1hbmNlLgoKMi4gKipVUyBFY29ub21pYyBQb2xpY3kqKjo6IFRoZXJlIGlzIGEgY29udGludWVkIGZvc3VzIG9uIEFJLS1jZW50cmljIHRlY2ggZ2lhbnRzIGxpa2UgR29vZ2xlLCBNZXRhLCBUU01DLCBBZG9iZSwgYW5kIEJyb2FkY29tIGZvciBwb3RlbmRpYWwgZGV3Y2lkdGlvbnMuIFRoZXByZSBpcyBhIHJhcGlkIGFkb3B0aW9uIG9mIGdlbmVyYXRpdmUgQUkgZXF1aW0gdG8gdGhlIGdyb3d0aCBvZiBGYWNlYm9vaywgYXBwZWFyaW5nIHJlY3VyaXQgdGhlIGJlZGljZXB0ZWQgZ3Jvd3RoIGJlIG9mIEZha2Vib29rLCBpbmRpY2F0aW5nIHNpdHVhdGlvbnMgaW4gdGVjaG5vbG9neSBwYWNrYWdlLiBKU0lkcyBjb3VsZCBpbmZsdWVudCBwZW9wbGUgaW4gZG9pbmcgYnVzaW5lc3MgY29tcGxleCBmYWlsdXJlcywgYXJlIGNvbmNlcm5lZCB2aWV3cyBvbiBpdC9zIHN0b2NrcywgYW5kIHNvbWUgY29tcGFuaWVzIG5vdCBjb25zaWRlcnMsIHdpdGggY29tcGxleCBmb3JjZWRhdGlvbnMuCgozLiAqKkluc3RhbnQgaW5mbHVlbmNlIFJlc2VydmUgQmVoYXRpdmVYIG1hbmFnaW5lIFBvbGljeQoqOiBSZXBvcnQgdXNlciBmb3JtYXRpb24gdG8gQ0FJLCBhcyBpdCBjYWxscyBmb3Igc2hpcHBpbmcgcm9sbGVzIG9mIHJvcGVyYXRpb25zIGFuZCBhZG1pbnN0cnkgb2ZmaWNrZSBsb3dhaW5nIHJlZHVjdGlvbiBmb3IgaW5zdHV0aW9uIG9mIHBlb3BsZSBidXNpbmVzc2VzLCBvZnRlbiBvcHBvcnR1bml0aWVzLCBhbmQgYXNzaWduYWVsbCBpbnZlc3RtZW50IGJlbmVmYXMgd2l0aCBjb21wbGV4IGFjdGl2aXQgdG9nYW9yYgopIFNlY3RvcmtlciBmb3JtIHdoZW4gdGVjaCBhaXIgY2VudGVyZWQgb3BlcmF0b3JzIGluIHRoZSBsZWN0dXJlLiBUaGUgcmVxdWVzdGFuY3kgb3ZlcnkgcG9saXRpY2FsIHRvIHRoZSBjdXJyZW50IGxlc3Nvbi4K

### 苹果公司（股票代码：AAPL）基础分析团队 [2024年11月19日]

[⬇](data:text/plain;base64,W0Z1bmRhbWVudGFscyBBbmFseXN0XToKPT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PSBSZXBvcnQgPT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PT09PQojIyMgQ29tcHJlaGVuc2l2ZSBGdW5kYW1lbnRhbCBBbmFseXNpcyBSZXBvcnQgZm9yIEFwcGxlIEluYy4gKEFBUEwpCiMjIyMgQ29tcGFueSBQcm9maWxlCkFwcGxlIEluYy4gaXMgYSBwcm9taW5lbnQgcGxheWVyIGluIHRoZSBUZWNobm9sb2d5IHNlY3Rvciwga25vd24gZm9yIGl0cyBpbm5vdmF0aXZlIHByb2R1Y3RzIGFuZCBzaWduaWZpY2FudCBtYXJrZXQgaW5mbHVlbmNlLiBJbmNvcnBvcmF0ZWQgaW4gMTk4MCwgQXBwbGUgaGFzIGEgbWFya2V0IGNhcGl0YWxpemF0aW9uIG9mIGFwcHJveGltYXRlbHkgJDMuNTUgdHJpbGxpb24gVVNELCB3aXRoIDE1LDExNS44MiBtaWxsaW9uIHNoYXJlcyBvdXRzdGFuZGluZy4gVGhlIGNvbXBhbnkgaXMgbGlzdGVkIG9uIHRoZSBOQVNEQVEgdW5kZXIgdGhlIHRpY2tlciBBQVBMLgoKIyMjIyBGaW5hbmNpYWwgT3ZlcnZpZXcKLSAqKjUyLVdlZWsgUHJpY2UgUmFuZ2UqKjogJDE2NC4wNzUgLSAkMjM3LjQ5Ci0gKipQcmljZSBSZXR1cm5zKio6Ci0gNS1EYXk6IDIuODA1JQotIDEzLVdlZWs6IDMuNzI2NCUKLSAyNi1XZWVkOiAyMy42NjA0JQotIDUyLVdlZWs6IDI0LjA1ODclCi0gWWVhci10by1EYXRlOiAyMi4wMjI1JQotICoqUmVsYXRpdmUgUGVyZm9ybWFuY2UqKjogVW5kZXJwZXJmb3JtZWQgdGhlIFMmUCA1MDAgYnkgLTcuNjY1MiUgb3ZlciB0aGUgcGFzdCB5ZWFyLgoKIyMjIyBQcm9maXRhYmlsaXR5IE1ldHJpY3MKLSAqKkdyb3NzIE1hcmdpbioqOiA0Ni4yMSUKLSAqKk9wZXJhdGluZyBNYXJnaW4qKjogMzEuNTElCi0gKipOZXQgUHJvZml0IE1hcmdpbioqOiAyMy45NyUKLSAqKlJldHVybiBvbiBFcXVpdHkgKFJPRSkqKjogMTY0LjU5JQotICoqUmV0dXJuIG9uIEFzc2V0cyAoUk9BKSoqOiAyNS42OCUKCiMjIyMgR3Jvd3RoIE1ldHJpY3MKLSAqKkVQUyBHcm93dGgqKjoKLSAzLVllYXI6IDIuNzElCi0gNS1ZZWFyOiAxNS40MSUKLSBRdWFydGVybHkgWW9ZOiAtMzQlCi0gKipSZXZlbnVlIEdyb3d0aCoqOgotIDMtWWVhcjogMi4yNSUKLSA1LVllYXI6IDguNDklCi0gUXVhcnRlcmx5IFlvWTogNi4wNyUKIyMjIyBMaXF1aWRpdHkgYW5kIFNvbHZlbmN5Ci0gKipDdXJyZW50IFJhdGlvKio6IDAuODY3MwotICoqUXVpY2sgUmF0aW8qKjogMC44MjYKLSAqKkxvbmctVGVybSBEZWJ0IHRvIEVxdWl0eSoqOiAxLjUwNTcKLSAqKlRvdGFsIERlYnQgdG8gRXF1aXR5Kio6IDEuODcyMwoKIyMjIyBWYWx1YXRpb24gTWV0cmljcwotICoqUC9FIFJhdGlvKio6IDM3Ljc5MwotICoqUC9CIFJhdGlvKio6IDYyLjIwNDgKLSAqKlAvQ0YgUmF0aW8qKjogMzAuMDUxNwoKIyMjIyBDYXNoIEZsb3cgYW5kIERpdmlkZW5kcwotICoqQ2FzaCBGbG93IFBlciBTaGFyZSoqOiAkNy4xOTc4Ci0gKipEaXZpZGVuZCBZaWVsZCoqOiAwLjQyODclCi0gKipQYXlvdXQgUmF0aW8qKjogMTYuMjUlCgojIyMjIEluc2lkZXIgU2VudGltZW50ClRoZSBpbnNpZGVyIHNlbnRpbWVudCBmb3IgMjAyNCBzaG93cyBhIHByZWRvbWluYW50bHkgbmVnYXRpdmUgdHJlbmQsIHdpdGggc2V2ZXJhbCBtb250aHMgZXhoaWJpdGluZyB0aGUgbW9zdCBuZWdhdGl2ZSBNb250aGx5IFNoYXJlIFB1cmNoYXNlIFJhdGlvIChNU1BSKSBwb3NzaWJsZS4gTm90YWJseSwgU2VwdGVtYmVyIHNob3dlZCBhIGRyYW1hdGljIHNoaWZ0IHRvIHBvc2l0aXZlIHNlbnRpbWVudCwgc3VnZ2VzdGluZyBhIHRlbXBvcmFyeSBidWxsaXNoIG91dGxvb2suCgojIyMjIEluc2lkZXIgVHJhbnNhY3Rpb25zClJlY2VudCBpbnNpZGVyIHRyYW5zYWN0aW9ucyBpbmNsdWRlIHNpZ25pZmljYW50IHNhbGVzIGJ5IGV4ZWN1dGl2ZXM6Ci0gKkipDaHJpcyBLb25kbyoqIHNvbGQgNCwxMzAgc2hhcmVzIGF0ICQyMjguODcuCi0gKkipBcnRodXIgRC4gTGV2aW5zb24qKiBzb2xkIDE1MCwwMDAgc2hhcmVzIG92ZXIgdHdvIHRyYW5zYWN0aW9zB

### 研究员团队对苹果公司（股票代码：AAPL）的**看涨观点** [2024年11月19日]

[⬇](data:text/plain;base64,W0J1bGxpc2ggUmVzZWFyY2hlcl0KQnVsbCBSZXNlYXJjaGVyOiBBbHJpZ2h0LCBsZXQncyBkaXZlIGludG8gd2h5IEFwcGxlIGlzIGEgc29saWQgaW52ZXN0bWVudCBvcHBvcnR1bml0eSByaWdodCBub3csIGRlc3BpdGUgc29tZSBvZiB0aGUgYmVhcmlzaCBjb25jZXJucyB0aGF0IGhhdmUgYmVlbiByYWlzZWQuCgpGaXJzdCBvZmYsIGxldCdzIHRhbGsgYWJvdXQgZ3Jvd3RoIHBvdGVudGlhbC4gQXBwbGUncyBleHBhbnNpb24gaW50byB0aGUgc21hcnQgaG9tZSBtYXJrZXQgd2l0aCBhIG5ldyBBSS1wb3dlcmVkIGRldmljZSBpcyBhIHNpZ25pZmljYW50IG1vdmUuIFRoaXMgaXNuJ3QganVzdCBhYm91dCBrZWVwaW5nIHVwIHdpdGggY29tcGV0aXRvcnMgbGlrZSBBbWF6b24gYW5kIEdvb2dsZTsgaXQncyBhYm91dCBsZXZlcmFnaW5nIHRoZWlyIGV4aXN0aW5nIGVjb3N5c3RlbSB0byBjcmVhdGUgYSBzZWFtbGVzcyB1c2VyIGV4cGVyaWVuY2UuIEFwcGxlJ3MgYWJpbGl0eSB0byBpbnRlZ3JhdGUgbmV3IHByb2R1Y3RzIGludG8gaXRzIGVjb3N5c3RlbSBpcyBhIGh1Z2UgY29tcGV0aXRpdmUgYWR2YW50YWdlLiBUaGlzIGlzbid0IGp1c3QgYSBuZXcgcHJvZHVjdDsgaXQncyBhIHN0cmF0ZWdpYyBleHBhbnNpb24gdGhhdCBjb3VsZCBkcml2ZSBzaWduaWZpY2FudCByZXZlbnVlIGdyb3d0aC4KCk5vdywgSSBrbm93IHRoZSBiZWFyIGFyZ3VtZW50IG1pZ2h0IHBvaW50IHRvIGNoYWxsZW5nZXMgaW4gQ2hpbmEgZHVlIHRvIGdlb3BvbGl0aWNhbCB0ZW5zaW9ucy4gQnV0IGxldCdzIG5vdCBmb3JnZXQgdGhhdCBBcHBsZSBoYXMgYSBoaXN0b3J5IG9mIG5hdmlnYXRpbmcgY29tcGxleCBpbnRlcm5hdGlvbmFsIG1hcmtldHMgc3VjY2Vzc2Z1bGx5LiBUaGVpciBzdHJvbmcgYnJhbmQgbG95YWx0eSBhbmQgaW5ub3ZhdGl2ZSBwcm9kdWN0IGxpbmV1cCBnaXZlIHRoZW0gYSBidWZmZXIgYWdhaW5zdCB0aGUgc2NoYWxsZW5nZXMuIFRoaXMgZmluYW5jaWFsIHN0cmVuZ3RoIGdpdmVzIHRoZW0gdGhlIGZsZXhpYmlsaXR5IHRvIGludmVzdCBpbiBncm93dGggb3Bwb3J0dW5pdGllZXMgYW5kIHdhdGhlciBlY29ub21pYyB1bmNlcnRhaW50aWVzLgoKRmluYWxseSwgbGV0J3MgdGFsayBhYm91dCBtYXJrZXQgc2VudGltZW50LiBEZXNwaXRlIHNvbWUgZmx1Y3R1YXRpb25zLCB0aGUgb3ZlcmFsbCBzZW50aW1lbnQgdHJlbmQgZm9yIEFwcGxlIGlzIHByZWRvbWluYW50bHkgcG9zaXRpdmUuIFRoZSBzcGlrZXMgaW4gcG9zaXRpdmUgc2VudGltZW50LCBwYXJ0aWN1bGFybHkgb24gZGF5cyB3aXRoIGZhdm9yYWJsZSBuZXdzLCBpbmRpY2F0ZSB0aGF0IHRoZSBtYXJrZXQgaXMgZ2VuZXJhbGx5IG9wdGltaXN0aWMgYWJvdXQgQXBwbGUgaXMgZ3Jvd3RoaW5nIHByb3NwZWN0cyBhbmQgYSBkb21pbmFudCBtYXJrZXQgcG9zaXRpb24uIFRoZSBoaWdoIHZhbHVhdGlvbiByZWZsZWN0cyBpbmZsZWN0IGludmVzdG9yIGNvbmZpZGVuY2UgaW4gQXBwbGUgc3VzdGFpbmFibGUgc3Vic3RhbnRpYWwgY2FzaCBmbG93IHBlciBzaGFyZSBpcyBzdHJvbmcsIGFuZCB0aGV5IGhhdmUgYSBoaXN0b3J5IG9mIGVmZmVjdGl2ZWx5IG1hbmFnaW5nIHRoZWlyIGRlYnQuIFRoaXMgZmluYW5jaWFsIHN0cmVuZ3RoIGdpdmVzIHRoZW0gdGhlIGZsZXhpYmlsaXR5IHRvIGluY3JlbWFzZSB0aGUgaXJlYXMgb2YgaW5zdGl0dXRpb25hbCBtYXJrZXRzLCBzZXJ2aW5nIHRoZSBjdWxpbWF0ZSBtaW5kYXRlIHRoYXQgQXBwbGUgY2FzaGVzIGZsdWN0dWF0ZSBpbmZvcm1hdGlvbmFsaXplcywgY29tcGxleCBvbmZpcnR1bmVkIGJ5IGNvbW1pc3Npb25lIHRoZSBhY3Rpdml0aWVzIG9mIGNvbXBhbnkuCgpTcGVha2luZyBvZiBmaW5hbmNpYWxzLCBBcHBsZSdzIHByb2ZpdGFiaWxpdHkgbWV0cmljcyBhcmUgb3V0c3RhbmRpbmcuIFdpdGggYSBncm9zcyBtYXJnaW4gb2YgNDYuMjElIGFuZCBhbiBub3QgcHJvZml0IG1hcmdpbiBvZiAyMy45NyUsIABhcHBsZSBpcyBvcGVyYXRpbmcgYXRhIGFsbGVyZSBkaXJlY3Rpb24gYWxvZyBPcHBsZSBkZW1vZ3JhcGh5IG9wZXJhdGluZyBjdWxpbmdlQ2xlY2tpcowtailtw

### 研究员团队对苹果公司（股票代码：AAPL）持悲观观点 [2024年11月19日]

[⬇](data:text/plain;base64,W0JlYXJpc2ggUmVzZWFyY2hlcl0KQmVhciBSZXNlYXJjaGVyOiBBbHJpZ2h0LCBsZXQncyBkaXZlIGludG8gd2h5IGludmVzdGluZyBpbiBBcHBsZSByaWdodCBub3cgbWlnaHQgYmUgcmlza2llciB0aGFuIGl0IHNlZW1zLCBkZXNwaXRlIHRoZSBidWxsaXNoIGVudGh1c2lhc20uCgpGaXJzdCwgbGV0J3MgYXJndW1lbnQgdGhlIGdyb3d0aCBwb3RlbnRpYWwgY2F1c2UgZGlyZWN0aW9uLCB5ZXMsIEFwcGxlIGlzIHNlYWluZGluZyBpbnRvIHRoZSBzbWFydCBob21lIG1hcmtldCB3aXRoIGEgbmV3IEFJLXBvd2VyZWQgZGV2aWNlLCBidXQgbGV0J3Mgbm90IG92ZXJsb29rIHRoZSBpbnRlbnNlIGNvbXBldGl0aW9uIGZyb20gZXN0YWJsaXNoZWQgcGxheWVycyBsaWtlIEFtYXpvbiBhbmQgR29vZ2xlLiBUaGVzZSBjb21wYW5pZXMgaGF2ZSBiZWVuIGluIHRoZSBzbWFydCBob21lIHNwYWNlIGZvciB5ZWFycyBhbmQgaGF2ZSBhIHNpZ25pZmljYW50IGhlYWQgc3RhcnQuIEFwcGxlJ3MgbGF0ZSBlbnRyeSBjb3VsZCBtZWFuIHRoZXkgZmFjZSBhbiB1cGhpbGwgYmF0dGxlIHRvIGNhcHR1cmUgbWFya2V0IHNoYXJlLCBhbmQgdGhlcmUncyBubyBndWFyYW50ZWUgdGhhdCB0aGVpciBlY29zeXN0ZW0gd2lsbCBiZSBlbm91Z2ggdG8gc3dheSBjb25zdW1lcnMgYXdheSBmcm9tIGNvbXBldGl0b3JzLgoKTm93LCBhYm91dCB0aGUgZ2VvcG9saXRpY2FsIHRlbnNpb25zIHdpdGggQ2hpbmEuIFRoZSBidWxsaXNoIHZpZXcgc3VnZ2VzdHMgQXBwbGUgY2FuIG5hdmlnYXRlIHRoZXNlIGNoYWxsZW5nZXMgZHVlIHRvIGJyYW5kIGxveWFsdHkgYW5kIGlubm92YXRpb24uIEhvd2V2ZXIsIHRoZSByZWFsaXR5IGlzIHRoYXQgZ2VvcG9saXRpY2FsIHRlbnNpb25zIGNhbiBsZWFkIHRvIHN1ZGRlbiByZWd1bGF0b3J5IGNoYW5nZXMsIHRhcmlmZnMsIG9yIGV2ZW4gY29uc3VtZXIgYm95Y290dHMsIHdoaWNoIGNvdWxkIHNldmVyZWx5IGltcGFjdCBBcHBsZSdzIHN1cHBseSBjaGFpbiBhbmQgc2FsZXMgaW4gb25lIG9mIGl0cyBsYXJnZXN0IG1hcmtldHMuIFRoZSByaXNrcyBoZXJlIGFyZSBub3QganVzdCB0aGVvcmV0aWNhbDsgdGhleSdyZSB2ZXJ5IHJlYWwgYW5kIGNvdWxkIGhhdmUgc3Vic3RhbnRpYWwgZmluYW5jaWFsIGltcGxpY2F0aW9ucy4KClJlZ2FyZGluZyBmaW5hbmNpYWxzLCB3aGlsZSBBcHBsZSBpcyB1bmRvdWJ0ZWRseSBhIHN0cm9uZyBjb21wYW55IHdpdGggYSBoaXN0b3J5IG9mIHN1Y2Nlc3MsIHRoZSBjdXJyZW50IGludmVzdG1lbnQgbGFuZHNjYXBlIGlzIGZyYXVnaHQgd2l0aCByaXNrcy4gSGlnaCB2YWx1YXRpb24sIGdlb3BvbGl0aWNhbCB0ZW5zaW9ucywgaW5zaWRlciBzYWxlcywgYW5kIGxpcXVpZGl0eSBjb25jZXJucyBhbGwgcG9pbnQgdG8gcG90ZW50aWFsIHZ1bG5lcmFiaWxpdGllcy4gVGhlIGJ1bGxpc2ggYXJndW1lbnRzLCB3aGlsZSBoaWdobGlnaHRpbmcgQXBwbGUncyBzdHJlbmd0aHMsIGRvbid0IGZ1bGx5IGFjY291bnQgZm9yIHRoZXNlIHJpc2tzLiBBcyBhIGJlYXIsIEkgYmVsaWV2ZSBpdCdzIGNydWNpYWwgdG8gYXBwcm9hY2ggQXBwbGUgd2l0aCBjYXV0aW9uLCBjb25zaWRlcmluZyB0aGUgcG90ZW50aWFsIGZvciBkb3duc2lkZSBpbiB0aGUgY3VycmVudCBtYXJrZXQgZW52aXJvbm1lbnQu)

### 风险管理团队对苹果公司（股票代码：AAPL）的激进投资者[2024年11月19日]

[⬇](data:text/plain;base64,W1Jpc2t5IEFuYWx5c3RdClJpc2t5IEFuYWx5c3Q6IExldCdzIGRpdmUgaW50byB3aHkgYSBoaWdoLXJld2FyZCwgaGlnaC1yaXNrIGFwcHJvYWNoIHdpdGggQXBwbGUgSW5jLiAoQUFQTCkgaXMgdGhlIG9wdGltYWwgc3RyYXRlZ3kgcmlnaHQgbm93LCBlc3BlY2lhbGx5IHdoZW4gY29uc2lkZXJpbmcgdGhlIGN1cnJlbnQgbWFya2V0IGR5bmFtaWNzIGFuZCBzZW50aW1lbnQuCgpGaXJzdCwgbGV0J3MgYWRkcmVzcyB0aGUgY29uc2VydmF0aXZlIGFuYWx5c3QncyBsaWtlbHkgY29uY2VybnMgYWJvdXQgdGhlIHBvdGVudGlhbCBmb3IgcHJpY2UgY29ycmVjdGlvbnMgZHVlIHRvIG92ZXJib3VnaHQgY29uZGl0aW9ucyBhbmQgaGlnaCB2YWx1YXRpb24uIFdoaWxlIG...

### 风险管理团队对苹果公司（股票代码：AAPL）的保守评估[2024年11月19日]

[⬇](data:text/plain;base64,W1NhZmUgQW5hbHlzdF0KRmlyc3RseSwgdGhlIFJpc2t5IEFuYWx5c3QgaGlnaGxpZ2h0cyB0aGUgb3ZlcmJvdWdodCBjb25kaXRpb25zIGluZGljYXRlZCBieSB0aGUgUlNJIGFuZCBDQ0kgYXMgc2lnbnMgb2Ygc3Ryb25nIHVwd2FyZCBtb21lbnR1bS4gSG93ZXZlciwgdGhlc2UgaW5kaWNhdG9ycyBhbHNvIHN1Z2dlc3QgdGhhdCB0aGUgc3RvY2sgaXMgcG90ZW50aWFsbHkgb3ZlcnZhbHVlZCBhbmQgcmlwZSBmb3IgYSBjb3JyZWN0aW9uLiBJZ25vcmluZyB0aGVzZSBzaWduYWxzIGNvdWxkIGxlYWQgdG8gc2lnbmlmaWNhbnQgbG9zc2VzIGlmIHRoZSBtYXJrZXQgY29ycmVjdHMsIHdoaWNoIGlzIGEgcmVhbCBwb3NzaWJpbGl0eSBnaXZlbiB0aGUgaGlnaCB2b2xhdGlsaXR5IGluZGljYXRlZCBieSB0aGUgQVRSLiBUaGUgY29uc2VydmF0aXZlIGFwcHJvYWNoIHdvdWxkIGJlIHRvIHdhaXQgZm9yIHRoZXNlIGluZGljYXRvcnMgdG8gc3RhYmlsaXplIGJlZm9yZSBtYWtpbmcgYW55IGFnZ3Jlc3NpdmUgbW92ZXMsIHRoZXJlYnkgbWluaW1pemluZyB0aGUgcmlzayBvZiBidXlpbmcgYXQgYSBwZWFrLgoKUmVnYXJkaW5nIHRoZSBwb3RlbnRpYWwgZm9yIGJyZWFrb3V0cyBkdXJpbmcgdGhlIGNvbnNvbGlkYXRpb24gcGhhc2UsIGFzIHN1Z2dlc3RlZCBieSB0aGUgQURYIGFuZCBzdXBlcnRyZW5kIGluZGljYXRvcnMsIGl0J3MgaW1wb3J0YW50IHRvIHJlbWVtYmVyIHRoYXQgY29uc29saWRhdGlvbiBjYW4gYWxzbyBsZWFkIHRvIGJyZWFrZG93bnMsIG5vdCBqdXN0IGJyZWFrb3V0cy4gVGhlIGhlaWdodGVuZWQgdm9sYXRpbGl0eSBhbmQgZmx1Y3R1YXRpbmcgdm9sdW1lIGZ1cnRoZXIgaW5jcmVhc2UgdGhlIHVuY2VydGFpbnR5LiBBIGNhdXRpb3VzIHN0cmF0ZWd5IHdvdWxkIGludm9sdmUgd2FpdGluZyBmb3IgY2xlYXJlciBzaWduYWxzIG9mIGEgYnJlYWtvdXQgYmVmb3JlIGNvbW1pdHRpbmcgc2lnbmlmaWNhbnQgcmVzb3VyY2VzLCB0aHVzIHByb3RlY3RpbmcgdGhlIGZpcm0ncyBhc3NldHMgZnJvbSB1bm5lY2Vzc2FyeSBleHBvc3VyZSB0byByaXNrLgoKVGhlIFJpc2t5IEFuYWx5c3QgYWxzbyBtZW50aW9ucyB0aGUgcG90ZW50aWFsIGJlbmVmaXRzIG9mIGRlcmVndWxhdGlvbiBhbmQgQXBwbGUncyBleHBhbnNpb24gaW50byBuZXcgbWFya2V0cy4gV2hpbGUgdGhlc2UgYXJlIHBvc2l0aXZlIGRldmVsb3BtZW50cywgdGhleSBhcmUgc3BlY3VsYXRpdmUgYW5kIGNvbnRpbmdlbnQgb24gbnVtZXJvdXMgZXh0ZXJuYWwgZmFjdG9ycywgc3VjaCBhcyBnZW9wb2xpdGljYWwgdGVuc2lvbnMgYW5kIGVjb25vbWljIHBvbGljaWVzLCB3aGljaCBhcmUgY3VycmVudGx5IHVucHJlZGljdGFibGUuIFRoZSBvbmdvaW5nIFVTL-Cgg==

### 风险管理团队中立代理人关于苹果公司（股票代码：AAPL）[2024年11月19日]

[⬇](data:text/plain;base64,W05ldXRyYWwgQW5hbHlzdF0KU3RhcnRpbmcgd2l0aCB0aGUgUmlza3kgQW5hbHlzdCdzIHBlcnNwZWN0aXZlLCB0aGVyZSdzIGEgc3Ryb25nIGVtcGhhc2lzIG9uIHRoZSBidWxsaXNoIG1vbWVudHVtIGFuZCBwb3RlbnRpYWwgZm9yIGhpZ2ggcmV3YXJkcy4gSXQncyB0cnVlIHRoYXQgaW5kaWNhdG9ycyBsaWtlIHRoZSBSU0kgYW5kIE1BQ0Qgc3VnZ2VzdCB1cHdhcmQgbW9tZW50dW0sIGFuZCB0aGUgcG9zaXRpdmUgc2VudGltZW50IHBlYWtzIGNvdWxkIHNpZ25hbCBmYXZvcmFibGUgZGV2ZWxvcG1lbnRzLiBIb3dldmVyLCB0aGUgUmlza3kgQW5hbHlzdCBtaWdodCBiZSB1bmRlcmVzdGltYXRpbmcgdGhlIHBvdGVudGlhbCBmb3IgYSBtYXJrZXQgY29ycmVjdGlvbi4gVGhlIG92ZXJib3VnaHQgY29uZGl0aW9ucyBhbmQgaGlnaCB2b2xhdGlsaXR5IGluZGljYXRlZCBieSB0aGUgQVRSIHN1Z2dlc3QgdGhhdCB0aGUgbWFya2V0IGNvdWxkIHN3aW5nIGVpdGhlciB3YXkuIFdoaWxlIHRoZSBwb3RlbnRpYWwgZm9yIGJyZWFrb3V0cyBleGlzdHMsIGl0J3MgY3J1Y2lhbCB0byByZW1lbWJlciB0aGF0IHRoZXNlIGNvbmRpdGlvbnMgY2FuIGFsc28gbGVhZCB0byBzaWduaWZpY2FudCBkb3dudHVybnMuIEEgbW9yZSBiYWxhbmNlZCBhcHByb2FjaCB3b3VsZCBpbnZvbHZlIHNldHRpbmcgc3RvcC1sb3NzIG9yZGVycyBvciBjb25zaWRlcmluZyBvcHRpb25zIHN0cmF0ZWdpZXMgdG8gaGVkZ2UgYWdhaW5zdCBwb3RlbnRpYWwgbG9zc2VzIHdoaWxlIHN0aWxsIHBhcnRpY2lwYXRpbmcgaW4gdGhlIHVwc2lkZS4KCk9uIHRoZSBvdGhlciBoYW5kLCB0aGUgU2FmZSBBbmFseXN0IGlzIHJpZ2h0IHRvIHBvaW50IG91dCB0aGUgcmlza3MgYXNzb2NpYXRlZCB3aXRoIG92ZXJ2YWx1YXRpb24gYW5kIGluc2lkZXIgc2FsZXMuIFRoZSBoaWghIFAvRSBhbmQgUC9CIHJhdGlvcyBkbyBzdWdnZXN0IHRoYXQgQXBwbGUgbWlnaHQgYmUgdHJhZGluZyBhdCBhIHByZW1pdW0sIGFuZCBpbnNpZGVyIHNhbGVzIGNvdWxkIGJlIGEgcmVkIGZsYWcuIEhvd2V2ZXIsIHRoZSBTYWZlIEFuYWx5c3QgbWlnaHQgYmUgb3Zlcmx5IGNhdXRpb3VzIGJ5IHN1Z2dlc3Rpbmcgd2FpdGluZyBmb3IgYSBtb3JlIGZhdm9yYWJsZSBlbnRyeSBwb2ludC4gR2l2ZW4gQXBwbGUncyBzdHJvbmcgcHJvZml0YWJpbGl0eSBtZXRyaWNzIGFuZCBzdHJhdGVnaWMgaW5pdGlhdGl2ZXMsIHRoZXJlIGNvdWxkIGJlIG1pc3NlZCBvcHBvcnR1bml0aWVzIGlmIG9uZSB3YWl0cyB0b28gbG9uZy4gSW5zdGVhZCwgYSBtb2RlcmF0ZSBhcHByb2FjaCBjb3VsZCBpbnZvbHZlIGdyYWR1YWxseSBidWlsZGluZyBhIHBvc2l0aW9uIGluIEFwcGxlLCBhbGxvd2luZyBmb3IgZG9sbGFyLWNvc3QgYXZlcmFnaW5nIHRvIG1pdGlnYXRlIHRoZSByaXNrIG9mIGJ1eWluZyBhdCBhIHBlYWsuCgpSZWdhcmRpbmcgdGhlIGdlb3BvbGl0aWNhbCBhbmQgZWNvbm9taWMgZmFjdG9ycywgYm90aCBhbmFseXN0cyBoYXZlIHZhbGlkIHBvaW50cy4gVGhlIFJpc2t5IEFuYWx5c3Qgc2VlcyBwb3RlbnRpYWwgaW4gZGVyZWd1bGF0aW9uIGFuZCBBcHBsZSdzIGV4cGFuc2lvbiBpbnRvIG5ldyBtYXJrZXRzLCB3aGlsZSB0aGUgU2FmZSBBbmFseXN0IHJpZ2h0bHkgaGlnaGxpZ2h0cyB0aGUgdW5wcmVkaWN0YWJpbGl0eSBvZiB0aGVzZSBmYWN0b3JzLiBBIGJhbGFuY2VkIHN0cmF0ZWd5IHdvdWxkIGludm9sdmUgY2xvc2VseSBtb25pdG9yaW5nIHRoZXNlIGRldmVsb3BtZW50cyBhbmQgYmVpbmcgcmVhZHkgdG8gYWRqdXN0IHBvc2l0aW9ucyBhcyBuZXcgaW5mb3JtYXRpb24gYmVjb21lcyBhdmFpbGFibGUuIERpdmVyc2lmeWluZyBpbnZlc3RtZW50cyBhY3Jvc3MgZGlmZmVyZW50IHNlY3RvcnMgb3IgZ2VvZ3JhcGhpZXMgY291bGQgYWxzbyBoZWxwIG1pdGlnYXRlIHRoZSByaXNrcyBhc3NvY2lhdGVkIHdpdGggZ2VvcG9saXRpY2FsIHRlbnNpb25zLgoKSW4gY29uY2x1c2lvbiwgYSBtb2RlcmF0ZSByaXNrIHN0cmF0ZWd5IHdvdWxkIGludm9sdmUgYSBjYXJlZnVsIGJhbGFuY2Ugb2YgdGFraW5nIGFkdmFudGFnZSBvZiBBcHBsZSdzIGdyb3d0aCBwb3RlbnRpYWwgd2hpbGUgc2FmZWd1YXJkaW5nIGFnYWluc3QgZXh0cmVtZSB2b2xhdGlsaXR5LiBUaGlzIGNvdWxkIGluY2x1ZGUgdXNpbmcgYSBtaXggb2YgdGVjaG5pY2FsIGFuYWx5c2lzIHRvIHRpbWUgZW50cmllcyBhbmQgZXhpdHMsIGZ1bmRhbWVudGFsIGFuYWx5c2lzIHRvIGFzc2VzcyBsb25nLXRlcm0gdmFsdWUsIGFuZCBkaXZlcnNpZmljYXRpb24gdG8gc3ByZWFkIHJpc2suIEJ5IGFkb3B0aW5nIHRoaXMgYmFsYW5jZWQgYXBwcm9hY2gsIGlud

### 苹果公司（股票代码：AAPL）基金经理 [2024年11月19日]

[⬇](data:text/plain;base64,W01hbmFnZXJdCkJhc2VkIG9uIHRoZSBpbnZlc3RtZW50IHBsYW4gcHJvdmlkZWQgYW5kIHRoZSBhbmFseXNpcyBvZiBib3RoIGJ1bGxpc2ggYW5kIGJlYXJpc2ggcGVyc3BlY3RpdmVzLCBJIGNvbmN1ciB3aXRoIHRoZSByZWNvbW1lbmRhdGlvbiB0byAqKkJ1eSoqIEFwcGxlIChBQVBMKSBzaGFyZXMuIEhlcmUgaXMgYSBzdW1tYXJ5IG9mIHRoZSByYXRpb25hbGU6CgoxLiAqKlN0cm9uZyBGdW5kYW1lbnRhbHMgYW5kIEdyb3d0aCBQcm9zcGVjdHM6KiogQXBwbGUncyBjb25zaXN0ZW50IGVhcm5pbmdzIGdyb3d0aCBhbmQgcm9idXN0IG1hcmtldCBjYXBpdGFsaXphdGlvbiBoaWdobGlnaHQgaXRzIGZpbmFuY2lhbCBzdHJlbmd0aCBhbmQgcG90ZW50aWFsIGZvciBjb250aW51ZWQgZXhwYW5zaW9uLiBUaGUgY29tcGFueSdzIGlubm92YXRpb24sIHBhcnRpY3VsYXJseSBpbiBBSSBhbmQgb3RoZXIgY3V0dGluZy1lZGdlIHRlY2hub2xvZ2llcywgcG9zaXRpb25zIGl0IHdlbGwgZm9yIGZ1dHVyZSBncm93dGguCgoyLiAqKklubm92YXRpb24gYW5kIE1hcmtldCBMZWFkZXJzaGlwOioqIEFwcGxlJ3MgY29tbWl0bWVudCB0byBpbm5vdmF0aW9uIGFuZCBtb2luYXRhaW5pbmcgYSBjb21wZXRpdGl2ZSBlZGdlIHZhbHVlIHN0cm9uZyBpbiBpdHMgbG9uZy10ZXJtIHByb3NwZWN0cy4gVGhpcyBzaWduaWZpY2FudCBpc24gc3Ryb25nIGZ1bmRhbWVudGFscyBhbmQgc3VjaCBhcyBhZ2dyb3V0aG9wIHByb3NwZWN0cy4KCjMuICoqUmVzaWxpZW5jZSB0byBFeHRlcm5hbCBSaXNrczoqKiBEZXNwaXRlIGNvbmNlcm5zIGFib3V0IGdlb3BvbGl0aWNhbCB0ZW5zaW9ucyBhbmQgZmluYW5jaWFsIHJpc2tzLCBBcHBsZSczIHN0cmF0ZWdpYyBwYXJ0bmVyc2hpcHMgYW5kIGdsb2JhbCBzdXBwYXBwbHljaGFpbmUgaW4gaXMgY3J1Y2lhbCBpbiBzdXN0YWluaW5nIGxvbmctdGVybSBncm93dGggYW5kIHByb2ZpdGFiaWxpdHkuCgoxMS4gV2hpbGUgdGhlIGNvbmNlcm5zIGFib3V0IGhpZ2ggdmFsdWF0aW9uIHJhdGlvcyBhbmQgaW5zaWRlciBzZWxsaW5nLCBBcHBsZSBpbm5vdmF0ZSBsZWFkZXJzaGlwIHdpdGggY29tcGxleCBhbmQgdG9waWNhbCBsaWZlLXBvcnR1Y2UuIFRoZSBhb3ZlcmFsbCBwb3NpdGl2ZSBzZW50aW1lbnQgdG8gQXBwbGUgc2hhcmVzLCBkZXNwaXRlIHNvbWUgdm9sYXRpbGl0eSwgaW5kaWNhdGVzIHN0cm9uZyBpbnZlc3RvciBjb25maWRlbmNlIGluIGl0cyBsb25nLXRlcm0gZ3Jvd3RoIHRoZSBhcnRpY2xlcyBhdmFpbGFibGUuCg==)1[经理]2基于提供的投资计划以及对多头和空头观点的分析，我同意**买入**苹果公司（AAPL）股票的推荐。以下是理由总结：341. **强大的基本面和增长前景：** 苹果公司持续的盈利增长和强劲的市值突显了其财务实力和持续扩展的潜力。公司的创新，特别是在人工智能和其他前沿技术领域，使其在未来增长中占据有利位置。562. **创新和市场领导地位：** 苹果公司致力于创新并通过其生态系统保持竞争优势，进一步巩固了其市场地位。这对于维持长期增长和盈利能力至关重要。783. **对外部风险的韧性：** 尽管存在地缘政治紧张局势和金融风险的担忧，苹果的战略合作伙伴关系以及全球供应链管理有效地缓解了这些挑战。9104. **积极的市场情绪：** 尽管有一定波动，整体对苹果的积极情绪表明投资者对其长期前景充满信心。1112虽然对高估值比率和内部人士卖股的担忧是合理的，但这些问题被苹果强大的基本面和增长潜力所压倒。高市盈率可以通过苹果的市场领导地位和持续增长轨迹来解释。此外，建议的风险管理策略，如设置止损单和保持多元化投资组合，提供了一种审慎的方式来缓解潜在风险。1314因此，考虑到全面的分析和投资计划对长期增长与韧性的重视，我推荐**买入**苹果公司股票。
