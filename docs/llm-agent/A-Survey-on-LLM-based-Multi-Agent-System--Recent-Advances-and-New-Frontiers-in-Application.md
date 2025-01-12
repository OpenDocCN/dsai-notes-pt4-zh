<!--yml  

category: 未分类  

date: 2025-01-11 11:44:02  

-->  

# 基于LLM的多智能体系统综述：近期进展与应用新前沿  

> 来源：[https://arxiv.org/html/2412.17481/](https://arxiv.org/html/2412.17481/)  

陈帅航¹  刘远星¹  韩伟¹  张维南¹²²2通讯作者  刘婷¹  

¹哈尔滨工业大学社会计算与信息检索研究中心  

哈尔滨工业大学，中国  

{shchen, yxliu, whan, wnzhang, tliu}@ir.hit.edu.cn  

###### 摘要  

\Acf  

基于LLM的多智能体系统（LLM-MAS）自大规模语言模型（LLMs）兴起以来，已成为研究热点。然而，随着相关新研究的不断涌现，现有的综述难以全面涵盖所有进展。本文对这些研究进行了全面综述。我们首先讨论了基于LLM的多智能体系统（LLM-MAS）的定义，这是一个涵盖许多先前工作的框架。我们概述了LLM-MAS在以下方面的各种应用：（i）解决复杂任务，（ii）模拟特定场景，以及（iii）评估生成智能体。基于先前的研究，我们还突出展示了若干挑战，并提出了该领域未来的研究方向。  

基于LLM的多智能体系统综述：  

近期进展与应用新前沿

陈帅航¹  刘远星¹  韩伟¹  张维南¹²²2通讯作者  刘婷¹ ¹哈尔滨工业大学社会计算与信息检索研究中心，中国 {shchen, yxliu, whan, wnzhang, tliu}@ir.hit.edu.cn  

## 1 引言  

\Acf  

MAS的扩展显著，得益于其适应性以及解决复杂、分布式挑战的能力，Balaji 和 Srinivasan（[2010](https://arxiv.org/html/2412.17481v2#bib.bib4)）。与单一代理设置相比，Gronauer 和 Diepold（[2022](https://arxiv.org/html/2412.17481v2#bib.bib26)）指出，多智能体系统（MAS）能够更准确地再现真实世界，因为许多现实世界的应用本质上涉及多个决策者同时互动。然而，由于传统强化学习（RL）代理的参数限制以及缺乏通用知识和能力，代理无法解决复杂的决策任务，例如与其他代理合作进行开发，Qian 等人（[2024b](https://arxiv.org/html/2412.17481v2#bib.bib57)）。近年来，诸如 Llama 3（Dubey 等人，[2024](https://arxiv.org/html/2412.17481v2#bib.bib20)）和 GPT-4（OpenAI 等人，[2024](https://arxiv.org/html/2412.17481v2#bib.bib50)）等大型语言模型（LLMs）取得了显著的成功，通过对海量网络语料库进行训练，[Radford 等人](https://arxiv.org/html/2412.17481v2#bib.bib59)）。与强化学习相比，生成型代理，核心控制代理为大型语言模型（LLM），即使没有训练，也能在推理、长时间轨迹决策等方面表现更好，Shinn 等人（[2023](https://arxiv.org/html/2412.17481v2#bib.bib62)）。此外，生成型代理提供了与人类互动的自然语言接口，使这些互动更加灵活且易于解释，Park 等人（[2023](https://arxiv.org/html/2412.17481v2#bib.bib53)）。基于这些优势，基于LLM的多智能体系统（LLM-MAS）应运而生。研究人员对这些新兴的工作进行了调查，并提出了一个通用框架，Guo 等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib27)）。然而，随着相关研究数量的不断增加，一些研究超出了原始框架的范围。在本文中，我们基于之前对基于LLM的多智能体系统（LLM-MAS）的回顾提供了一个新的视角，重点讨论最近的进展并探讨潜在的研究方向。我们收集了2023年和2024年在顶级人工智能会议上发表的125篇论文，如*ACL、NeurIPS、AAAI 和 ICLR*，以及一些尚未发表但具有价值的arXiv论文。¹¹1这些论文的列表可以在[https://github.com/bianhua-12/Multi-generative_Agent_System_survey](https://github.com/bianhua-12/Multi-generative_Agent_System_survey)找到。根据LLM-MAS的目的，我们总结了LLM-MAS的应用，包括任务解决、特定问题的仿真和生成型代理的评估。图[1](https://arxiv.org/html/2412.17481v2#S1.F1 "图1 ‣ 1 引言 ‣ 基于LLM的多智能体系统：应用中的最新进展和新前沿")展示了我们为LLM-MAS应用提出的框架。(i) 解决复杂任务。多智能体会自然地将任务拆分为子任务，从而提高任务表现。(ii) 为特定场景进行仿真。研究人员将LLM-MAS视为在特定领域中仿真问题的沙盒。(iii) 评估生成型代理。与传统任务评估相比，LLM-MAS具有动态评估的能力，这更加灵活且更难泄露数据。对于每个类别，我们将讨论具有代表性的LLM-MAS、资源及其评估。

![请参阅图注](img/3b670422d808ba48fcdd5cdfac875fb6.png)

图1：LLM-MAS、生成性智能体和LLM的应用框架概览及其相互关系。带虚线边框的直角矩形表示与先前调研一致的内容，而圆角矩形表示本研究中的原创贡献。

相比于先前的调研工作Guo et al. ([2024](https://arxiv.org/html/2412.17481v2#bib.bib27)); Li et al. ([2024d](https://arxiv.org/html/2412.17481v2#bib.bib39)); Han et al. ([2024](https://arxiv.org/html/2412.17481v2#bib.bib28)); Gronauer and Diepold ([2022](https://arxiv.org/html/2412.17481v2#bib.bib26)), 本次调研具有以下独特贡献：（i）专注于LLM-MAS应用的分类法：我们基于LLM-MAS应用目的引入了一个更新的分类法（分类法及其差异见图[1](https://arxiv.org/html/2412.17481v2#S1.F1 "图1 ‣ 1 引言 ‣ 基于LLM的多智能体系统：应用中的最新进展与新前沿")）。（ii）更多资源：我们分析了具有基准或数据集的开源框架和研究成果，以便为研究社区提供支持。（iii）挑战与未来：我们讨论了LLM-MAS面临的挑战，并展望了未来的研究方向。

## 2 LLM-MAS的核心组件

LLM-MAS指的是一种系统，包含一组能够在共享的环境设置中进行互动与协作的生成性智能体Wang et al. ([2024c](https://arxiv.org/html/2412.17481v2#bib.bib66))。我们将在接下来的部分中讨论生成性智能体和环境。

### 2.1 生成性智能体

生成性智能体是指LLM-MAS的组成部分，具有角色定义，能够感知环境、做出决策，并执行复杂的行为以改变环境Wang et al. ([2024a](https://arxiv.org/html/2412.17481v2#bib.bib64))。它们可以是游戏中的玩家，或社交媒体上的用户，推动LLM-MAS的发展并影响其结果。

与传统代理相比，生成代理需要能够执行更复杂的行为，例如根据历史信息生成完整的个性化博客文章 Park et al. ([2022](https://arxiv.org/html/2412.17481v2#bib.bib54))。因此，除了将大语言模型（LLM）作为核心，生成代理还需要具备以下特点：（i）*角色建模*用于通过自然语言描述角色来关联其行为 Gao et al. ([2023b](https://arxiv.org/html/2412.17481v2#bib.bib22))，或根据任务为每个生成代理定制提示 Xu et al. ([2023c](https://arxiv.org/html/2412.17481v2#bib.bib71))。（ii）*记忆*用于存储历史轨迹，并为后续代理行为检索相关记忆，使代理能够在解决大语言模型上下文窗口限制问题的同时采取长期行动。通常包括三层记忆：长期记忆、短期记忆和感官记忆 Park et al. ([2023](https://arxiv.org/html/2412.17481v2#bib.bib53))。（iii）*规划*是为未来较长时间段制定一般性行为 Yao et al. ([2023](https://arxiv.org/html/2412.17481v2#bib.bib72))。（iv）*行动*执行生成代理与环境之间的交互 Wang et al. ([2024a](https://arxiv.org/html/2412.17481v2#bib.bib64))。生成代理可能需要从多个候选行为中选择一个来执行，例如投票选举谁 Xu et al. ([2024](https://arxiv.org/html/2412.17481v2#bib.bib70))，或生成没有强制约束的行为，例如生成一段文本 Li et al. ([2023](https://arxiv.org/html/2412.17481v2#bib.bib40))。

生成代理可以相互通信，以实现系统内部的合作。生成代理的通信大致可以分为两个目的。（i）第一个目的是实现协作，将自己获得的信息与其他智能代理共享，并在一定程度上将多个智能代理聚合成一个完整的系统，从而实现超越独立智能代理的性能 Yuan et al. ([2023](https://arxiv.org/html/2412.17481v2#bib.bib75))；（ii）第二个目的是实现共识，使某些代理之间的行为或策略更加相似，从而能够更快地收敛到纳什均衡 Oroojlooy and Hajinezhad ([2023](https://arxiv.org/html/2412.17481v2#bib.bib51))。

通信内容的类型大致可以分为两种：自然语言和定制内容。自然语言形式的沟通具有较高的可解释性和灵活性，但难以优化，因此更适合用于追求共识的场景，例如Chatdev Qian等人（[2024b](https://arxiv.org/html/2412.17481v2#bib.bib57)）和招聘会系统Li等人（[2023](https://arxiv.org/html/2412.17481v2#bib.bib40)）。定制内容可能是一个向量或一个离散信号，除了系统中的生成代理外，没人能理解它。但这种形式容易通过策略梯度进行优化，因此常用于实现协作目的，例如DIAL Hausknecht和Stone（[2015](https://arxiv.org/html/2412.17481v2#bib.bib29)）算法及其变量。

### 2.2 环境

环境设置包括规则、工具和干预接口：(i) *工具*负责将代理的动作指令转化为具体的结果。生成代理将动作指令发送到环境，环境将指令转化为记录，表明该动作已被执行。不同场景下有不同的动作空间。在社交媒体场景中，动作空间包括“点赞”、“评论”、“关注”等 Wang 等人（[2024b](https://arxiv.org/html/2412.17481v2#bib.bib65)）。在开发场景中，动作空间涵盖了聊天链条 Qian 等人（[2024b](https://arxiv.org/html/2412.17481v2#bib.bib57)），这比社交网络的动作空间更大。(ii) *规则*定义了生成代理之间的通信方式或与环境的互动，直接定义了整个系统的行为结构。根据场景，系统有一些特殊的规则，例如游戏规则 Xu 等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib70)）；Chen 等人（[2024c](https://arxiv.org/html/2412.17481v2#bib.bib9)）以及社交行为规范 Park 等人（[2023](https://arxiv.org/html/2412.17481v2#bib.bib53)）；Wang 等人（[2024b](https://arxiv.org/html/2412.17481v2#bib.bib65)）。通常，大型系统中的生成代理有较小的动作空间，更容易被基于规则的模型所替代 Mou 等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib49)）。(iii) *干预*提供了一个外部干预系统的接口。此干预可以来自任何外部来源，可能是人类 Wang 等人（[2024b](https://arxiv.org/html/2412.17481v2#bib.bib65)），也可能是监督模型 Chen 等人（[2024c](https://arxiv.org/html/2412.17481v2#bib.bib9)），甚至是生成代理 Qian 等人（[2024b](https://arxiv.org/html/2412.17481v2#bib.bib57)）。干预的目的是主动从系统中读取信息 Wang 等人（[2024b](https://arxiv.org/html/2412.17481v2#bib.bib65)），或被动中断系统，以防止出现不受控制的行为 Qian 等人（[2024b](https://arxiv.org/html/2412.17481v2#bib.bib57)）。

## 3 LLM-MAS 用于解决复杂任务

完成复杂任务通常需要多个角色、多个步骤等。这对单一代理来说是困难的，但多个代理协同工作则非常适合完成这类任务 Islam 等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib33)）。此外，这些代理可以独立训练 Shen 等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib60)）；Yu 等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib74)）。与单一代理相比，LLM-MAS 可以实现更好的结果。也就是说，多代理协作将提高整体性能 Du 等人（[2023](https://arxiv.org/html/2412.17481v2#bib.bib18)）。

### 3.1 代表性的 LLM-MAS 用于解决复杂任务

该领域目前是一个热门的研究主题。最近，研究人员主要关注多代理推理框架和多代理通信优化，接下来将讨论这些内容。

LLM-MAS推理框架。我们通过推理流程总结了三个方面，包括：（i）多阶段框架，（ii）集体决策框架，以及（iii）自我完善框架。即，多阶段框架是指代理在不同阶段充当串行问题求解者Qian et al.（[2024b](https://arxiv.org/html/2412.17481v2#bib.bib57)），而集体决策Zhao et al.（[2024c](https://arxiv.org/html/2412.17481v2#bib.bib83)）是指不同的代理为一个目标进行投票或辩论。自我完善指的是LLM-MAS中的自我反思机制。研究人员提出了一个框架，用于将多代理应用于自然科学Chen et al.（[2024a](https://arxiv.org/html/2412.17481v2#bib.bib7)），以增强数据分析、模型仿真和决策过程Yin et al.（[2024](https://arxiv.org/html/2412.17481v2#bib.bib73)）。Zhang et al.（[2023a](https://arxiv.org/html/2412.17481v2#bib.bib77)）提出了一个框架以实现自适应和适应性合作。代理合作中的规模法则也被探索Qian et al.（[2024c](https://arxiv.org/html/2412.17481v2#bib.bib58)），发现没有显著的模式。

LLM-MAS通信优化。LLM-MAS中的全连接通信可能会导致组合爆炸和隐私泄露等问题。基于此，我们在通信优化中总结了两个方面，包括：（i）速度优化和（ii）分布式讨论。速度优化是指研究人员尝试加快代理的通信速度，例如，使用非语言交流Liu et al.（[2024b](https://arxiv.org/html/2412.17481v2#bib.bib47)）或更短的生成Chen et al.（[2024g](https://arxiv.org/html/2412.17481v2#bib.bib13)）。而分布式讨论是指代理在信息不足的情况下尝试解决任务Liu et al.（[2024a](https://arxiv.org/html/2412.17481v2#bib.bib45)）。代理需要相互通信以实现目标Zhang et al.（[2023a](https://arxiv.org/html/2412.17481v2#bib.bib77)），即使一个代理没有完整的信息Liu et al.（[2024a](https://arxiv.org/html/2412.17481v2#bib.bib45)）。

### 3.2 解决复杂任务的LLM-MAS资源

我们在表格[1](https://arxiv.org/html/2412.17481v2#S3.T1 "Table 1 ‣ 3.2 Resources of LLM-MAS for Solving Complex Tasks ‣ 3 LLM-MAS for Solving Complex Tasks ‣ A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application")中总结了常见的开源LLM-MAS用于仿真，包括代码、数据集和基准测试。

表格1：LLM-MAS用于解决任务研究中的代码和基准。“No Code”或“No Benchmark”表示代码或基准不可用。

| 领域 | 子领域 | 论文 | 代码 | 数据集与基准 |
| --- | --- | --- | --- | --- |
| 推理框架 | 多阶段 | 钱等人 ([2024b](https://arxiv.org/html/2412.17481v2#bib.bib57)) | [代码链接](https://github.com/OpenBMB/ChatDev) | SRDD |
| 杜等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib19)) | [代码链接](https://github.com/OpenBMB/ChatDev) | SRDD |
| 岳等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib76)) | [代码链接](https://github.com/yueshengbin/SMART) | SMART (自我) |
| 刘等人 ([2023c](https://arxiv.org/html/2412.17481v2#bib.bib48)) | [代码链接](https://github.com/salesforce/BOLAA) | WebShop |
| 林等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib43)) | [代码链接](https://anonymous.4open.science/r/MAO-1074) | FG-C, CG-O |
| 伊斯兰等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib33)) | [代码链接](https://github.com/Md-Ashraful-Pramanik/MapCoder) | HumanEval, EvalPlus, MBPP, APPS, xCodeEval, CodeContest |
| 沈等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib60)) | [代码链接](https://github.com/X-PLUG/Multi-LLM-Agent) | ToolBench, ToolAlpaca |
| 集体决策 | 赵等人 ([2024c](https://arxiv.org/html/2412.17481v2#bib.bib83)) | [代码链接](https://github.%20com/xiutian/GEDI) | MCQA |
| 程等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib14)) | [代码链接](https://github.com/YiCheng98/Cooper) | ESConv 数据集, P4G 数据集 |
| 梁等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib41)) | [代码链接](https://github.com/Skytliang/Multi-Agents-Debate) | MT-Bench |
| 雷等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib35)) | [代码链接](https://github.com/bin123apple/MACM) | MATH |
| 张等人 ([2024a](https://arxiv.org/html/2412.17481v2#bib.bib79)) | [代码链接](https://github.com/zjunlp/MachineSoM) | MMLU, MATH, 棋步有效性 |
| 王等人 ([2024d](https://arxiv.org/html/2412.17481v2#bib.bib67)) | [代码链接](https://github.com/MikeWangWZHL/Solo-Performance-Prompting.git) | TriviaQA |
| Self-Refine | 王等人 ([2024c](https://arxiv.org/html/2412.17481v2#bib.bib66)) | [代码链接](https://github.com/HKUST-KnowComp/LLM-discussion) | FOLIO-wiki |
| 陈等人 ([2024e](https://arxiv.org/html/2412.17481v2#bib.bib11)) | [代码链接](https://github.com/dinobby/ReConcile) | StrategyQA, CSQA, GSM8K, AQuA, MATH, 日期理解, ANLI |
| 陈等人 ([2024a](https://arxiv.org/html/2412.17481v2#bib.bib7)) | [代码链接](https://github.com/Link-AGI/AutoAgents) | TriviaQA |
| 唐等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib63)) | [代码链接](https://github.com/Code4Agent/codeagent) | Trans-Review, AutoTransform, T5-Review |
| 张等人 ([2023a](https://arxiv.org/html/2412.17481v2#bib.bib77)) | [代码链接](https://pku-proagent.github.io/) | Overcooked-AI |
| 通信优化 | 速度优化 | 刘等人 ([2024b](https://arxiv.org/html/2412.17481v2#bib.bib47)) | 无代码 | HotpotQA, NarrativeQA, MultifieldQA |
| 分布式 | Chen et al. ([2024f](https://arxiv.org/html/2412.17481v2#bib.bib12)) | [代码链接](https://github.com/OpenBMB/IoA) | TriviaQA, Natural Questions, HotpotQA, 2WikiMultiHopQA |
| Liu et al. ([2024a](https://arxiv.org/html/2412.17481v2#bib.bib45)) | [代码链接](https://github.com/thinkwee/iAgents) | InformativeBench |

数据集。所有传统 NLP 任务的数据集均可使用。此外，继 ECL 之后，Qian et al. ([2024a](https://arxiv.org/html/2412.17481v2#bib.bib56))，Qian et al. ([2024b](https://arxiv.org/html/2412.17481v2#bib.bib57)) 对 SRDD 数据集上的生成软件质量进行了评估，并在软件开发领域系统地评估了代理的能力。

开源社区。开源社区和工业界也对 LLM-MAS 的发展做出了重要贡献。MetaGPT Hong et al. ([2023](https://arxiv.org/html/2412.17481v2#bib.bib30)) 为生成性代理分配不同的角色，形成一个用于复杂任务的协作实体。Gao et al. ([2024](https://arxiv.org/html/2412.17481v2#bib.bib23)) 提出了以消息交换为核心通信机制的 AgentScope。同时，本研究开发了一个分发框架，能够无缝切换本地与分布式部署，并通过最小的努力实现自动并行优化。Open AI 提出了 Swarm Ope ([2024](https://arxiv.org/html/2412.17481v2#bib.bib2))，这是一个实验性的多代理编排框架，具有符合人体工程学的设计和轻量化特性。与之前发布的 Assistants API 不同，Swarm 让开发者对上下文、步骤和工具调用拥有更精细的控制，而不是托管在云端。

### 3.3 LLM-MAS 在解决复杂任务中的评估

特定任务的表现。如表[1](https://arxiv.org/html/2412.17481v2#S3.T1 "Table 1 ‣ 3.2 Resources of LLM-MAS for Solving Complex Tasks ‣ 3 LLM-MAS for Solving Complex Tasks ‣ A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application")所示，LLM-MAS的表现可以通过特定任务来评估，这种方式直观且便捷。例如，在一个APP系统中，张等人（[2023b](https://arxiv.org/html/2412.17481v2#bib.bib78)）考虑了一个智能体完成特定任务时使用的平均步骤数和工具数作为评估指标；在BOLAA中，刘等人（[2023c](https://arxiv.org/html/2412.17481v2#bib.bib48)）将智能物理检查检索的召回率和问答准确率作为评估指标；在狼人杀游戏中，许等人（[2023c](https://arxiv.org/html/2412.17481v2#bib.bib71)）自然地将虚拟玩家的胜率作为评估指标；在招聘会系统中，李等人（[2023](https://arxiv.org/html/2412.17481v2#bib.bib40)）将招聘方正确招募目标求职者的比例作为评估指标；在拍卖系统中，陈等人（[2024c](https://arxiv.org/html/2412.17481v2#bib.bib9)）通过预测价格与实际价格之间的斯皮尔曼相关系数以及竞标者的技能，使用TrueSkill得分（Graepel et al.，[2007](https://arxiv.org/html/2412.17481v2#bib.bib25)）来进行测量；在斯坦福镇公园系统中，帕克等人（[2023](https://arxiv.org/html/2412.17481v2#bib.bib53)）通过人工排序和使用TrueSkill评估虚拟智能体和人类智能体生成的行为质量；在城市仿真系统中，许等人（[2023a](https://arxiv.org/html/2412.17481v2#bib.bib68)）也将完成导航等特定任务的成功率作为评估指标。

通信成本分析。最重要的关注点在于系统的运行成本。考虑到现代大多数系统将LLM作为关键模块，系统运行过程中产生的额外开销已成为一个关键的研究领域。例如，牟等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib49)）使用系统的实际运行时间作为重要指标，强调了管理这一运营开销的重要性。

## 4 LLM-MAS用于仿真特定场景

本节将阐述LLM-MAS在仿真中的应用。研究人员利用智能体仿真某一场景，以研究其对特定学科（如社会科学）的影响。一方面，与基于规则的方法相比，生成型智能体通过自然语言交流更能直观地与人类互动。另一方面，环境决定了仿真的属性，这是整个仿真过程的核心。

### 4.1 用于模拟特定场景的代表性LLM-MAS

LLM-MAS模拟的典型场景如下所述。我们将根据主题介绍以下工作。

社会领域。现实世界中的大规模社会实验成本高昂，而社会参与的庞大规模有时可能升级为暴力和破坏，带来潜在的后果Mou等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib49)）。因此，有必要在虚拟环境中进行模拟；模拟可以解决现实环境中过高的开销问题，并且能够以更快的速度在虚拟环境中长时间模拟现实世界的过程Li等人（[2024a](https://arxiv.org/html/2412.17481v2#bib.bib36)）。同时，整个过程可以轻松重复，有利于进一步的研究。研究人员已经做了大量的工作来模拟社交媒体场景。基于社交媒体模拟原型，Park等人（[2022](https://arxiv.org/html/2412.17481v2#bib.bib54)）和Park等人（[2023](https://arxiv.org/html/2412.17481v2#bib.bib53)）提出了“斯坦福小镇”，该项目模拟了美国一个小镇上25个不同职业的代理人的一天生活。同时，还有关于情感传播影响的研究Gao等人（[2023b](https://arxiv.org/html/2412.17481v2#bib.bib22)）、基于推荐场景的“信息茧房”研究Wang等人（[2024b](https://arxiv.org/html/2412.17481v2#bib.bib65)）和社会运动的研究Mou等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib49)）。研究人员提出了城市生成智能（UGI）Xu等人（[2023a](https://arxiv.org/html/2412.17481v2#bib.bib68)），旨在解决特定的城市问题并模拟复杂的城市系统，提供了一个多学科的方式来理解和管理城市复杂性。Li等人（[2024a](https://arxiv.org/html/2412.17481v2#bib.bib36)）通过医院模拟研究了医生代理人演化方法。由于医生代理人训练既便宜又高效，这项工作能够在短短几天内快速扩展代理人，处理数万个病例，而这一任务通常需要人类医生数年才能完成。Pan等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib52)）提出了大规模的代理人模拟，将代理人数量增加到$10^{6}$。在社交游戏中，像狼人杀Xu等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib70)）、阿瓦隆Lan等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib34)）和我的世界Gong等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib24)）等也尝试了LLM-MAS模拟。此外，一些游戏公司，如网易，也在积极地在其游戏中实验LLM-MAS。

物理领域。对于物理领域，生成性智能体仿真的应用包括移动行为、运输 [Gao 等人](https://arxiv.org/html/2412.17481v2#bib.bib21)、无线网络等。然而，在多生成智能体领域的研究有限。[Zou 等人](https://arxiv.org/html/2412.17481v2#bib.bib84) 探索了多个智能体在无线领域的应用，提出了一种框架，其中多个设备上的智能体可以与环境互动并交换知识，共同解决复杂任务。

### 4.2 LLM-MAS 仿真资源

我们在表 [2](https://arxiv.org/html/2412.17481v2#S4.T2 "表 2 ‣ 4.2 LLM-MAS 仿真资源 ‣ 4 LLM-MAS 用于特定场景的仿真 ‣ 基于LLM的多智能体系统调查：最近进展与应用新前沿") 中总结了用于仿真的常见开源 LLM-MAS，包括代码和基准测试。

为了证明仿真效果，即与现实相符，研究人员通常通过仿真真实数据来评估仿真系统。因此，具有密集用户和记录的真实数据集对仿真评估非常重要 [Mou 等人](https://arxiv.org/html/2412.17481v2#bib.bib49)。理想的数据集应该是密集的：也就是说，在相同规模下，用户数量较少的数据可以更好地评估 LLM-MAS 的仿真能力。

对于基准测试，[Du 和 Zhang](https://arxiv.org/html/2412.17481v2#bib.bib17) 提出了基于狼人场景的 WWQA，以评估智能体在狼人场景中的能力。[SoMoSiMu-Bench Mou 等人](https://arxiv.org/html/2412.17481v2#bib.bib49) 提供了针对个体用户行为和社会仿真系统结果的评估基准。

表 2：LLM-MAS 仿真研究中的代码和基准测试。“无代码”或“无基准测试”表示代码或基准测试不可用。

| 领域 | 子领域 | 论文 | 代码 | 数据集和基准测试 |
| --- | --- | --- | --- | --- |
| 社会 | 微型社会 | Huang 等人 ([2024b](https://arxiv.org/html/2412.17481v2#bib.bib32)) | 无代码 | AdaSociety |
| Chen 等人 ([2024b](https://arxiv.org/html/2412.17481v2#bib.bib8)) | [代码链接](https://github.com/relic-yuexi/AgentCourt) | AgentCourt |
| Park 等人 ([2023](https://arxiv.org/html/2412.17481v2#bib.bib53)) | [代码链接](https://github.com/joonspk-research/generative_agents) | 无基准测试或数据集 |
| Piatti 等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib55)) | [代码链接](https://github.com/giorgiopiatti/govsim) | 无基准测试 |
| Chuang 等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib15)) | [代码链接](https://github.com/yunshiuan/llm-agent-opinion-dynamics) | 无基准测试或数据集 |
| 经济学 | Li 等人 ([2024b](https://arxiv.org/html/2412.17481v2#bib.bib37)) | [代码链接](https://github.com/tsinghua-fib-lab/ACL24-EconAgent) | 无基准测试或数据集 |
| 社交媒体 | Wang 等人 ([2024b](https://arxiv.org/html/2412.17481v2#bib.bib65)) | [代码链接](https://github.com/RUC-GSAI/YuLan-Rec) | Movielens-1M |
| Gao 等人 ([2023b](https://arxiv.org/html/2412.17481v2#bib.bib22)) | 无代码 | 博客作者语料库 |
| Mou 等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib49)) | [代码链接](https://github.com/xymou/social_simulation) | SoMoSiMu-Bench(self) |
| 游戏 | Du 和 Zhang ([2024](https://arxiv.org/html/2412.17481v2#bib.bib17)) | [代码链接](https://github.com/doslim/Evaluate-the-Opinion-Leadership-of-LLMs) | WWQA |
| Pan 等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib52)) | [代码链接](https://github.com/modelscope/agentscope/tree/main/examples/paper_large_scale_simulation) | 无基准或数据集 |
| 物理 | 无线 | Zou 等人 ([2023](https://arxiv.org/html/2412.17481v2#bib.bib84)) | 无代码 | 无基准或数据集 |

### 4.3 LLM-MAS 仿真评估

我们将讨论基于用于评估LLM-MAS整体的指标，而非单个代理的能力的评估方法。

一致性。LLM-MAS 需要与现实世界保持良好的一致性，以确保得出有意义且富有洞察力的实验结果。在仿真系统的背景下，以UGI Xu 等人 ([2023a](https://arxiv.org/html/2412.17481v2#bib.bib68)) 为例，主要目标是忠实地复制特定的现实场景。当用于训练像 SMART Yue 等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib76)) 这样的代理时，只有那些在与现实环境高度相似的虚拟环境中经过严格训练的代理，才能被认为适合在真实环境中部署。类似地，当用于评估目的时，例如在 AgentSims Lin 等人 ([2023](https://arxiv.org/html/2412.17481v2#bib.bib42)) 中，只有当虚拟环境与现实世界保持高度一致时，才能获得真实可靠的评估结果。最后，在收集数据的系统中，如 BOLAA Liu 等人 ([2023c](https://arxiv.org/html/2412.17481v2#bib.bib48))，一致性也确保了数据的有效性。因此，LLM-MAS 的一个重要性能指标是与现实情况的一致性。

信息传播。通过时间序列分析方法，比较系统中信息传播行为与现实中的差异。信息传播在一定程度上能够反映媒体的本质；因此，现实的多智能体系统应该具有与现实世界相似的信息传播趋势。Abdelzaher等人（[2020](https://arxiv.org/html/2412.17481v2#bib.bib3)）比较了在线社交媒体模拟环境中每天发生的事件数量变化；S3 Gao等人（[2023b](https://arxiv.org/html/2412.17481v2#bib.bib22)）比较了每天对某一事件知晓的用户数量，以及该事件的情感密度和支持率的每日变化；斯坦福镇公园的类似方法也被用于评估（[2023](https://arxiv.org/html/2412.17481v2#bib.bib53)）。

## 5 LLM-MAS用于评估生成代理

随着LLM在社区中的广泛应用，如何评估LLM的能力仍然是一个开放性问题。现有的评估方法存在以下缺陷：（i）评估能力受限，（ii）基准脆弱，（iii）评估指标不客观。LLM-MAS的复杂性和多样性表明，LLM-MAS可以用于评估LLM。然而，如何设计具体的评估指标和评估方法一直困扰着研究人员。同样，LLM-MAS也可以用于训练生成代理。我们总结了三方面的训练：（i）监督微调（SFT），（ii）强化学习（RL），（iii）合成数据用于训练。

### 5.1 用于评估生成代理的代表性LLM-MAS

LLM-MAS可以为代理提供奖励，这些奖励可以用来评估或训练生成代理，下面将进一步讨论。

生成代理的评估。研究人员通过将生成代理放入LLM-MAS中来研究生成代理。在LLM-MAS中，研究人员可以进一步研究LLM在不同场景中的战略能力，例如长远战略能力（Chen等人，[2024c](https://arxiv.org/html/2412.17481v2#bib.bib9)），领导力战略（Xu等人，[2023c](https://arxiv.org/html/2412.17481v2#bib.bib71)）和竞争力战略（Zhao等人，[2024b](https://arxiv.org/html/2412.17481v2#bib.bib82)）。在情感领域，MuMA-ToM Shi等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib61)）通过视频和文本描述来评估代理在现实家庭环境中理解和推理人类互动的能力。

在生成代理的训练方面，Li 等人 ([2024c](https://arxiv.org/html/2412.17481v2#bib.bib38)) 通过 LLM-MAS 强化数据，提升了监督微调（SFT）生成代理的效果。Xu 等人 ([2023c](https://arxiv.org/html/2412.17481v2#bib.bib71)) 提出了一个新框架，通过多智能体强化学习，使生成代理克服了 LLM 内在的偏差。对于 LLM-MAS，Yue 等人 ([2024](https://arxiv.org/html/2412.17481v2#bib.bib76)) 将知识密集型任务中的复杂轨迹分解为子任务，提出了一种多智能体框架的协同训练范式——长短轨迹学习（Long- and Short-Trajectory Learning），该方法确保了协同作用，同时保持了每个智能体的细粒度表现。RLHF 被批评为成本过高。Liu 等人 ([2023a](https://arxiv.org/html/2412.17481v2#bib.bib44)) 提出了一个基于多智能体系统的对齐方案，有效地解决了与基于奖励的 RL 优化相关的不稳定性和奖励博弈问题。无论如何，LLM-MAS 本质上被视为 RL 中的一个环境，通过不同的方式从环境中获取奖励。

### 5.2 LLM-MAS 评估资源

表 [3](https://arxiv.org/html/2412.17481v2#S5.T3 "表 3 ‣ 5.2 LLM-MAS 评估资源 ‣ 5 评估生成代理的 LLM-MAS ‣ 基于 LLM 的多智能体系统：近期进展与应用前沿综述") 显示了我们总结的含有代码、数据集和基准的相关工作，供未来的研究者参考。

表 3：LLM-MAS 评估研究中的代码和基准。“无代码”或“无基准”意味着代码或基准不可用。

| 领域 | 子领域 | 论文 | 代码 | 数据集和基准 |
| --- | --- | --- | --- | --- |
| 生成代理的评估 | 策略 | Liu 等人 ([2023b](https://arxiv.org/html/2412.17481v2#bib.bib46)) | [代码链接](https://github.com/THUDM/AgentBench) | AGENTBENCH |
| Bandi 和 Harrasse ([2024](https://arxiv.org/html/2412.17481v2#bib.bib5)) | 无代码 | MT-Bench |
| Chan 等人 ([2023](https://arxiv.org/html/2412.17481v2#bib.bib6)) | [代码链接](https://github.com/thunlp/ChatEval) | ChatEval |
| Chen 等人 ([2024d](https://arxiv.org/html/2412.17481v2#bib.bib10)) | [代码链接](https://github.com/THU-BPM/LLMArena.) | LLMARENA |
| Xu 等人 ([2023b](https://arxiv.org/html/2412.17481v2#bib.bib69)) | [代码链接](https://github.com/cathyxl/MAgIC) | MAgIC |
| Huang 等人 ([2024a](https://arxiv.org/html/2412.17481v2#bib.bib31)) | [代码链接](https://github.com/snap-stanford/MLAgentBench) | MLAgentBench |
| Chen 等人 ([2024c](https://arxiv.org/html/2412.17481v2#bib.bib9)) | [代码链接](https://github.com/jiangjiechen/auction-arena) | AUCARENA |
| 情感 | Zhang 等人 ([2024b](https://arxiv.org/html/2412.17481v2#bib.bib80)) | [代码链接](https://github.com/AI4Good24/PsySafe) | PsySafe |
| Shi et al. ([2024](https://arxiv.org/html/2412.17481v2#bib.bib61)) | [代码链接](https://github.com/SCAI-JHU/MuMMA-ToM) | MuMA-ToM |
| 生成代理的训练 | LLM-MAS中的SFT | Li et al. ([2024c](https://arxiv.org/html/2412.17481v2#bib.bib38)) | [代码链接](https://github.com/lirenhao1997/CoEvol) | MT-Bench, AlpacaEval |
| LLM-MAS中的MARL | Xu et al. ([2023c](https://arxiv.org/html/2412.17481v2#bib.bib71)) | 无代码 | 无数据集或基准 |
| 合成数据 | Liu et al. ([2023a](https://arxiv.org/html/2412.17481v2#bib.bib44)) | [代码链接](https://github.com/agi-templar/Stable-Alignment) | HH、道德故事、MIC、ETHICS-义务论、TruthfulQA |

## 6 挑战与未来方向

尽管在LLM-MAS的先前工作中取得了许多显著的成功，但这一领域仍处于初期阶段，且在其发展过程中存在几个重大挑战需要解决。以下是我们概述的几个关键挑战及其潜在的未来方向。

### 6.1 生成代理带来的挑战

生成代理是LLM-MAS的一个不可或缺的部分。然而，由于基础模型LLM的固有特性，生成代理存在一些缺点，以下将对此进行详细讨论。

挑战。(i) 面向仿真任务的广义对齐 Liu et al. ([2023a](https://arxiv.org/html/2412.17481v2#bib.bib44))。当代理被用于现实世界仿真时，一个完美的生成代理应该能够真实地描绘出多样化的特征 Wang et al. ([2024a](https://arxiv.org/html/2412.17481v2#bib.bib64))。然而，由于基础模型的训练方法 OpenAI et al. ([2024](https://arxiv.org/html/2412.17481v2#bib.bib50))，生成代理通常无法与虚拟对象对齐。(ii) 幻觉。生成代理在与其他代理互动时有一定的幻觉概率 Du et al. ([2023](https://arxiv.org/html/2412.17481v2#bib.bib18))。各种增强方法可以缓解这个问题，但无法解决。(iii) 缺乏足够的长文本处理能力。在处理复杂信息时，由于缺乏长文本能力，生成代理会忘记输入的信息 Zhao et al. ([2024a](https://arxiv.org/html/2412.17481v2#bib.bib81))。

未来方向。单个代理的能力或基础模型的能力的提升一直是一个热门话题。研究人员一直专注于增强对齐性、减少幻觉并提高长文本处理能力。新一代OpenAI模型o1 Int的提出（[2024](https://arxiv.org/html/2412.17481v2#bib.bib1)）为研究人员提供了新的思路，即通过更复杂的推理来增强模型的能力。

### 6.2 互动带来的挑战

由于LLM-MAS的复杂性、自回归特性以及其他特性，系统的实际应用中存在许多问题。以下列出了两个主要问题，*效率爆炸*和*累积效应*。

效率爆炸。由于LLM的自回归架构，通常具有较慢的推理速度。然而，生成代理需要多次查询LLM以执行每个操作，如从记忆中提取信息、在采取行动前进行计划等。当LLM-MAS规模扩大时，这个问题会被放大，特别是对于具有大行动空间的生成代理。SoMoSiMu-Bench Mou等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib49)）用基于规则的代理替换了边缘生成代理，从而缓解了这一问题。然而，对于具有复杂行动空间的生成代理的LLM-MAS，这个问题仍未得到解决。

累积效应。由于每轮LLM-MAS都基于前一轮的结果，并且LLM-MAS对后续结果有很大影响，研究人员采用了基于规则的模型进行中间错误修正（Chen等人，[2024c](https://arxiv.org/html/2412.17481v2#bib.bib9)），但仍然有很大的改进空间。IOA Chen等人（[2024f](https://arxiv.org/html/2412.17481v2#bib.bib12)）提出了一种类似互联网的通信架构，使得LLM-MAS更加可扩展，并增强了其对动态任务的适应能力。

未来方向。业界和学术界一直在努力降低LLM-MAS的通信成本，例如基于对齐的方法OPTIMA Chen等人（[2024g](https://arxiv.org/html/2412.17481v2#bib.bib13)）和工业化并行消息方法AgentScope Gao等人（[2024](https://arxiv.org/html/2412.17481v2#bib.bib23)），但仍处于基础阶段，且有很大的研究空间。

### 6.3 LLM-MAS评估的挑战

缺乏针对群体行为的客观度量标准。如[4.3节](https://arxiv.org/html/2412.17481v2#S4.SS3 "4.3 Evaluation of LLM-MAS simulation ‣ 4 LLM-MAS for Simulating Specific Scenarios ‣ A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application")所示，由于多代理环境的多样性、复杂性和不可预测性，从当前的研究工作中很难在群体层面获得足够详细、具体和直接的系统评估指标。目前，研究人员主要通过比较系统与真实环境的分布来评估LLM-MAS，但这对于LLM-MAS的运行过程缺乏足够的细节。

自动化评估和基准测试。由于缺乏LLM-MAS的基准测试，不同类型的LLM-MAS无法进行比较。此外，缺乏一个通用的基准框架，既可以进行单个评价也可以进行整体评价，用于评估大多数LLM-MAS。

未来方向。研究大规模LLM-MAS将成为一个新的研究热点，研究人员将评估并发现新的规模效应。同时，未来的研究中还将出现常见的测试基准和评估方法。

## 7 结论

本文系统地总结了现有的基于LLM的多智能体系统（LLM-MAS）领域的研究。我们从三个应用方面对这些研究进行了呈现和回顾：任务求解、仿真和LLM-MAS的评估。我们提供了一个详细的分类法，旨在建立现有研究之间的联系，总结每个方面的主要技术及其发展历史。除了回顾先前的工作，我们还提出了该领域的几个挑战，期望能为未来的潜在研究方向提供指导。

## 限制

尽管我们尽最大努力，但仍可能存在一些局限性。由于篇幅限制，我们只能简要总结每种方法，而无法提供详尽的技术细节。另一方面，我们主要收集了来自*ACL、NeurIPS、ICLR、AAAI和arXiv*的研究，可能遗漏了其他期刊和会议中发表的重要工作。在应用部分，我们主要列出了具有开放代码的代表性LLM-MAS资源，详见表[1](https://arxiv.org/html/2412.17481v2#S3.T1 "Table 1 ‣ 3.2 Resources of LLM-MAS for Solving Complex Tasks ‣ 3 LLM-MAS for Solving Complex Tasks ‣ A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application")、表[2](https://arxiv.org/html/2412.17481v2#S4.T2 "Table 2 ‣ 4.2 Resources for LLM-MAS simulation ‣ 4 LLM-MAS for Simulating Specific Scenarios ‣ A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application")和表[3](https://arxiv.org/html/2412.17481v2#S5.T3 "Table 3 ‣ 5.2 Resources of LLM-MAS for evaluations ‣ 5 LLM-MAS for Evaluating Generative Agents ‣ A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application")。更完整的论文可以在[https://github.com/bianhua-12/Multi-generative_Agent_System_survey](https://github.com/bianhua-12/Multi-generative_Agent_System_survey)中找到。我们认识到我们工作的时效性，未来我们将紧跟研究社区的讨论，更新观点并补充遗漏的工作。

## 致谢

本研究得到了国家重点研发计划（编号2022YFF0902100）和黑龙江省自然科学基金（YQ2021F006）的支持。

## 参考文献

+   Int (2024) 2024. Introducing OpenAI o1. https://openai.com/o1/.

+   Ope (2024) 2024. Openai/swarm. OpenAI.

+   Abdelzaher等（2020）Tarek Abdelzaher, Jiawei Han, Yifan Hao, Andong Jing, Dongxin Liu, Shengzhong Liu, Hoang Hai Nguyen, David M Nicol, Huajie Shao, Tianshi Wang等人。2020年。使用SocialCube的多尺度在线媒体仿真。*计算与数学组织理论*，26:145–174。

+   Balaji 和 Srinivasan (2010) P. G. Balaji 和 D. Srinivasan. 2010. [多代理系统简介](https://doi.org/10.1007/978-3-642-14435-6_1). 见 Dipti Srinivasan 和 Lakhmi C. Jain 主编，*多代理系统与应用创新 - 1*，第1-27页。Springer，柏林，海德堡.

+   Bandi 和 Harrasse (2024) Chaithanya Bandi 和 Abir Harrasse. 2024. [通过迭代辩论对大型语言模型进行对抗性多代理评估](https://doi.org/10.48550/arXiv.2410.04663). *预印本*, arXiv:2410.04663.

+   Chan 等人 (2023) Chi-Min Chan, Weize Chen, Yusheng Su, Jianxuan Yu, Wei Xue, Shanghang Zhang, Jie Fu, 和 Zhiyuan Liu. 2023. [ChatEval: 通过多代理辩论提高基于大型语言模型的评估器](https://doi.org/10.48550/arXiv.2308.07201). *预印本*, arXiv:2308.07201.

+   Chen 等人 (2024a) Guangyao Chen, Siwei Dong, Yu Shu, Ge Zhang, Jaward Sesay, Börje F. Karlsson, Jie Fu, 和 Yemin Shi. 2024a. [AutoAgents：自动生成代理的框架](https://doi.org/10.48550/arXiv.2309.17288). *预印本*, arXiv:2309.17288.

+   Chen 等人 (2024b) Guhong Chen, Liyang Fan, Zihan Gong, Nan Xie, Zixuan Li, Ziqiang Liu, Chengming Li, Qiang Qu, Shiwen Ni, 和 Min Yang. 2024b. [AgentCourt：通过对抗性可进化的律师代理模拟法庭](https://doi.org/10.48550/arXiv.2408.08089). *预印本*, arXiv:2408.08089.

+   Chen 等人 (2024c) Jiangjie Chen, Siyu Yuan, Rong Ye, Bodhisattwa Prasad Majumder, 和 Kyle Richardson. 2024c. [把钱放在嘴巴能说的地方：评估大型语言模型代理在拍卖场中的战略规划与执行](https://doi.org/10.48550/arXiv.2310.05746). *预印本*, arXiv:2310.05746.

+   Chen 等人 (2024d) Junzhe Chen, Xuming Hu, Shuodi Liu, Shiyu Huang, Wei-Wei Tu, Zhaofeng He, 和 Lijie Wen. 2024d. [LLMArena：在动态多代理环境中评估大型语言模型的能力](https://doi.org/10.18653/v1/2024.acl-long.705). 见 *第62届计算语言学协会年会论文集（第1卷：长篇论文）*，第13055-13077页.

+   Chen 等人 (2024e) Justin Chen, Swarnadeep Saha, 和 Mohit Bansal. 2024e. [ReConcile: 圆桌会议通过多样化大型语言模型的一致性提高推理能力](https://doi.org/10.18653/v1/2024.acl-long.381). 见 *第62届计算语言学协会年会论文集（第1卷：长篇论文）*，第7066-7085页，泰国曼谷。计算语言学协会.

+   Chen 等人 (2024f) Weize Chen, Ziming You, Ran Li, Yitong Guan, Chen Qian, Chenyang Zhao, Cheng Yang, Ruobing Xie, Zhiyuan Liu, 和 Maosong Sun. 2024f. [智能代理互联网：编织异质代理的协同智能网络](https://doi.org/10.48550/arXiv.2407.07061). *预印本*, arXiv:2407.07061.

+   Chen等人（2024g） Weize Chen, Jiarui Yuan, Chen Qian, Cheng Yang, Zhiyuan Liu, 和 Maosong Sun. 2024g. [Optima: 优化大语言模型基础的多代理系统的有效性和效率](https://doi.org/10.48550/arXiv.2410.08115). *预印本*，arXiv:2410.08115。

+   Cheng等人（2024） Yi Cheng, Wenge Liu, Jian Wang, Chak Tou Leong, Yi Ouyang, Wenjie Li, Xian Wu, 和 Yefeng Zheng. 2024. [Cooper: 协调专门代理以实现复杂对话目标](https://doi.org/10.1609/aaai.v38i16.29739). *AAAI人工智能会议论文集*，38(16):17853–17861。

+   Chuang等人（2024） Yun-Shiuan Chuang, Agam Goyal, Nikunj Harlalka, Siddharth Suresh, Robert Hawkins, Sijia Yang, Dhavan Shah, Junjie Hu, 和 Timothy Rogers. 2024. [基于大语言模型的代理网络模拟意见动态](https://doi.org/10.18653/v1/2024.findings-naacl.211). 发表在 *计算语言学会发现：NAACL 2024*，页面3326–3346，墨西哥城，墨西哥。计算语言学会。

+   Chuang和Rogers（2023） Yun-Shiuan Chuang 和 Timothy T. Rogers. 2023. [基于计算代理的意见动态模型：关于社会模拟和实证研究的综述](https://doi.org/10.48550/arXiv.2306.03446). *预印本*，arXiv:2306.03446。

+   Du和Zhang（2024） Silin Du 和 Xiaowei Zhang. 2024. 大众的舵手？评估狼人游戏中大语言模型的意见领导力。发表在 *第一次语言建模会议*。

+   Du等人（2023） Yilun Du, Shuang Li, Antonio Torralba, Joshua B. Tenenbaum, 和 Igor Mordatch. 2023. [通过多代理辩论提升语言模型的事实性和推理能力](https://doi.org/10.48550/arXiv.2305.14325). *预印本*，arXiv:2305.14325。

+   Du等人（2024） Zhuoyun Du, Chen Qian, Wei Liu, Zihao Xie, Yifei Wang, Yufan Dang, Weize Chen, 和 Cheng Yang. 2024. [通过跨团队合作开发多代理软件](https://doi.org/10.48550/arXiv.2406.08979). *预印本*，arXiv:2406.08979。

+   Dubey等人（2024） Abhimanyu Dubey, Abhinav Jauhri, Abhinav Pandey, Abhishek Kadian, Ahmad Al-Dahle, Aiesha Letman, Akhil Mathur 等人. 2024. [Llama 3 模型群](https://doi.org/10.48550/arXiv.2407.21783). *预印本*，arXiv:2407.21783。

+   Gao等人（2023a） Chen Gao, Xiaochong Lan, Nian Li, Yuan Yuan, Jingtao Ding, Zhilun Zhou, Fengli Xu, 和 Yong Li. 2023a. [大语言模型赋能的基于代理的建模与仿真：综述与展望](https://doi.org/10.48550/arXiv.2312.11970). *预印本*，arXiv:2312.11970。

+   Gao等人（2023b） Chen Gao, Xiaochong Lan, Zhihong Lu, Jinzhu Mao, Jinghua Piao, Huandong Wang, Depeng Jin, 和 Yong Li. 2023b. [S3: 基于大语言模型增强代理的社交网络模拟系统](https://doi.org/10.48550/arXiv.2307.14984). *预印本*，arXiv:2307.14984。

+   Gao 等人（2024）Dawei Gao, Zitao Li, Xuchen Pan, Weirui Kuang, Zhijian Ma, Bingchen Qian, Fei Wei, Wenhao Zhang, Yuexiang Xie, Daoyuan Chen, Liuyi Yao, Hongyi Peng, Zeyu Zhang, Lin Zhu, Chen Cheng, Hongzhu Shi, Yaliang Li, Bolin Ding 和 Jingren Zhou. 2024. [AgentScope：一个灵活而强大的多智能体平台](https://doi.org/10.48550/arXiv.2402.14034)。*预印本*，arXiv:2402.14034。

+   Gong 等人（2024）Ran Gong, Qiuyuan Huang, Xiaojian Ma, Yusuke Noda, Zane Durante, Zilong Zheng, Demetri Terzopoulos, Li Fei-Fei, Jianfeng Gao 和 Hoi Vo. 2024. [MindAgent：新兴的游戏互动](https://doi.org/10.18653/v1/2024.findings-naacl.200)。在 *计算语言学协会的发现：NAACL 2024*，第3154-3183页，墨西哥城，墨西哥。计算语言学协会。

+   Graepel 等人（2007）Thore Graepel, Tom Minka 和 R TrueSkill Herbrich. 2007. 一种贝叶斯技能评分系统。*神经信息处理系统进展*，19（569-576）：7。

+   Gronauer 和 Diepold（2022）Sven Gronauer 和 Klaus Diepold. 2022. [多智能体深度强化学习：一项调查](https://doi.org/10.1007/s10462-021-09996-w)。*人工智能评论*，55（2）：895–943。

+   Guo 等人（2024）Taicheng Guo, Xiuying Chen, Yaqi Wang, Ruidi Chang, Shichao Pei, Nitesh V. Chawla, Olaf Wiest 和 Xiangliang Zhang. 2024. [基于大语言模型的多智能体：进展与挑战的调查](https://doi.org/10.24963/ijcai.2024/890)。在 *第三十三届国际人工智能联合会议*，第9卷，第8048-8057页。

+   Han 等人（2024）Shanshan Han, Qifan Zhang, Yuhang Yao, Weizhao Jin, Zhaozhuo Xu 和 Chaoyang He. 2024. [LLM 多智能体系统：挑战与开放问题](https://doi.org/10.48550/arXiv.2402.03578)。*预印本*，arXiv:2402.03578。

+   Hausknecht 和 Stone（2015）Matthew Hausknecht 和 Peter Stone. 2015. 深度递归 Q-learning 用于部分可观察的 MDPs。*计算机科学*。

+   Hong 等人（2023）Sirui Hong, Mingchen Zhuge, Jonathan Chen, Xiawu Zheng, Yuheng Cheng, Ceyao Zhang, Jinlin Wang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, Chenyu Ran, Lingfeng Xiao, Chenglin Wu 和 Jürgen Schmidhuber. 2023. [MetaGPT：多智能体协作框架的元编程](https://doi.org/10.48550/arXiv.2308.00352)。*预印本*，arXiv:2308.00352。

+   Huang 等人（2024a）Qian Huang, Jian Vora, Percy Liang 和 Jure Leskovec. 2024a. [MLAgentBench：在机器学习实验中评估语言智能体](https://doi.org/10.48550/arXiv.2310.03302)。*预印本*，arXiv:2310.03302。

+   Huang 等人（2024b）Yizhe Huang, Xingbo Wang, Hao Liu, Fanqi Kong, Aoyang Qin, Min Tang, Xiaoxi Wang, Song-Chun Zhu, Mingjie Bi, Siyuan Qi 和 Xue Feng. 2024b. [AdaSociety：一种具有社会结构的适应性环境用于多智能体决策](https://doi.org/10.48550/arXiv.2411.03865)。*预印本*，arXiv:2411.03865。

+   Islam 等人 (2024) Md. Ashraful Islam, Mohammed Eunus Ali 和 Md Rizwan Parvez. 2024. [MapCoder: 用于竞争性问题求解的多代理代码生成](https://doi.org/10.18653/v1/2024.acl-long.269)。发表于 *计算语言学协会第62届年会论文集（第1卷：长篇论文）*，页面 4912–4944，泰国曼谷。计算语言学协会。

+   Lan 等人 (2024) Yihuai Lan, Zhiqiang Hu, Lei Wang, Yang Wang, Deheng Ye, Peilin Zhao, Ee-Peng Lim, Hui Xiong 和 Hao Wang. 2024. [基于LLM的代理社会调查：在《阿瓦隆》游戏中的协作与对抗](https://doi.org/10.48550/arXiv.2310.14985)。*预印本*，arXiv:2310.14985。

+   Lei 等人 (2024) Bin Lei, Yi Zhang, Shan Zuo, Ali Payani 和 Caiwen Ding. 2024. [MACM: 在解决复杂数学问题中利用多代理系统进行条件挖掘](https://doi.org/10.48550/arXiv.2404.04735)。*预印本*，arXiv:2404.04735。

+   Li 等人 (2024a) Junkai Li, Siyu Wang, Meng Zhang, Weitao Li, Yunghwei Lai, Xinhui Kang, Weizhi Ma 和 Yang Liu. 2024a. [Agent Hospital: 一个具有可进化医疗代理的医院模拟](https://doi.org/10.48550/arXiv.2405.02957)。*预印本*，arXiv:2405.02957。

+   Li 等人 (2024b) Nian Li, Chen Gao, Mingyu Li, Yong Li 和 Qingmin Liao. 2024b. [EconAgent: 利用大语言模型增强的代理进行宏观经济活动模拟](https://doi.org/10.18653/v1/2024.acl-long.829)。发表于 *计算语言学协会第62届年会论文集（第1卷：长篇论文）*，页面 15523–15536，泰国曼谷。计算语言学协会。

+   Li 等人 (2024c) Renhao Li, Minghuan Tan, Derek F. Wong 和 Min Yang. 2024c. [CoEvol: 通过多代理合作构建更好的指令微调响应](https://doi.org/10.48550/arXiv.2406.07054)。*预印本*，arXiv:2406.07054。

+   Li 等人 (2024d) Xinyi Li, Sai Wang, Siqi Zeng, Yu Wu 和 Yi Yang. 2024d. [基于LLM的多代理系统调查：工作流、基础设施和挑战](https://doi.org/10.1007/s44336-024-00009-2)。*Vicinagearth*，1(1):9。

+   Li 等人 (2023) Yuan Li, Yixuan Zhang 和 Lichao Sun. 2023. [MetaAgents: 通过协作生成代理模拟基于LLM的任务导向协调中的人类行为交互](https://doi.org/10.48550/arXiv.2310.06500)。*预印本*，arXiv:2310.06500。

+   Liang 等人 (2024) Tian Liang, Zhiwei He, Wenxiang Jiao, Xing Wang, Yan Wang, Rui Wang, Yujiu Yang, Shuming Shi 和 Zhaopeng Tu. 2024. [通过多代理辩论鼓励大语言模型中的发散性思维](https://doi.org/10.48550/arXiv.2305.19118)。*预印本*，arXiv:2305.19118。

+   Lin 等人 (2023) Jiaju Lin, Haoran Zhao, Aochi Zhang, Yiting Wu, Huqiuyue Ping 和 Qin Chen. 2023. [AgentSims: 一个用于大语言模型评估的开源沙盒](https://doi.org/10.48550/arXiv.2308.04026)。*预印本*，arXiv:2308.04026。

+   Lin 等人 (2024) Leilei Lin, Yumeng Jin, Yingming Zhou, Wenlong Chen, 和 Chen Qian. 2024. [MAO: 一个用于多代理编排的过程模型生成框架](https://doi.org/10.48550/arXiv.2408.01916). *预印本*, arXiv:2408.01916.

+   Liu 等人 (2023a) Ruibo Liu, Ruixin Yang, Chenyan Jia, Ge Zhang, Diyi Yang, 和 Soroush Vosoughi. 2023a. 在模拟社会互动中训练社会对齐语言模型. 收录于 *第十二届国际学习表征会议*.

+   Liu 等人 (2024a) Wei Liu, Chenxi Wang, Yifei Wang, Zihao Xie, Rennai Qiu, Yufan Dang, Zhuoyun Du, Weize Chen, Cheng Yang, 和 Chen Qian. 2024a. [在信息不对称下，协作任务的自主代理](https://doi.org/10.48550/arXiv.2406.14928). *预印本*, arXiv:2406.14928.

+   Liu 等人 (2023b) Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang, Shudan Zhang, Xiang Deng, Aohan Zeng, Zhengxiao Du, Chenhui Zhang, Sheng Shen, Tianjun Zhang, Yu Su, Huan Sun, Minlie Huang, Yuxiao Dong, 和 Jie Tang. 2023b. [AgentBench: 评估LLM作为代理的表现](https://doi.org/10.48550/arXiv.2308.03688). *预印本*, arXiv:2308.03688.

+   Liu 等人 (2024b) Yuhan Liu, Esha Choukse, Shan Lu, Junchen Jiang, 和 Madan Musuvathi. 2024b. [DroidSpeak: 增强跨LLM通信](https://doi.org/10.48550/arXiv.2411.02820). *预印本*, arXiv:2411.02820.

+   Liu 等人 (2023c) Zhiwei Liu, Weiran Yao, Jianguo Zhang, Le Xue, Shelby Heinecke, Rithesh Murthy, Yihao Feng, Zeyuan Chen, Juan Carlos Niebles, Devansh Arpit, Ran Xu, Phil Mui, Huan Wang, Caiming Xiong, 和 Silvio Savarese. 2023c. [BOLAA: 基准测试与编排增强LLM的自主代理](https://doi.org/10.48550/arXiv.2308.05960). *预印本*, arXiv:2308.05960.

+   Mou 等人 (2024) Xinyi Mou, Zhongyu Wei, 和 Xuanjing Huang. 2024. [揭示真相并促进变革：面向基于代理的大规模社会运动仿真](https://doi.org/10.18653/v1/2024.findings-acl.285). 收录于 *2024年计算语言学协会（ACL 2024）会议成果*，页码 4789–4809，泰国曼谷及虚拟会议。计算语言学协会。

+   OpenAI 等人 (2024) OpenAI, Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman 等. 2024. [GPT-4 技术报告](https://doi.org/10.48550/arXiv.2303.08774). *预印本*, arXiv:2303.08774.

+   Oroojlooy 和 Hajinezhad (2023) Afshin Oroojlooy 和 Davood Hajinezhad. 2023. 合作多代理深度强化学习的综述. *应用智能*, 53(11):13677–13722.

+   Pan 等人 (2024) Xuchen Pan, Dawei Gao, Yuexiang Xie, Yushuo Chen, Zhewei Wei, Yaliang Li, Bolin Ding, Ji-Rong Wen, 和 Jingren Zhou. 2024. [AgentScope中的超大规模多代理仿真](https://doi.org/10.48550/arXiv.2407.17789). *预印本*, arXiv:2407.17789.

+   Park 等人（2023）Joon Sung Park, Joseph C. O’Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, 和 Michael S. Bernstein. 2023. [生成代理：人类行为的互动模拟](https://doi.org/10.48550/arXiv.2304.03442). *预印本*, arXiv:2304.03442.

+   Park 等人（2022）Joon Sung Park, Lindsay Popowski, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, 和 Michael S. Bernstein. 2022. [社会模拟：为社会计算系统创建人口化原型](https://doi.org/10.48550/arXiv.2208.04024). *预印本*, arXiv:2208.04024.

+   Piatti 等人（2024）Giorgio Piatti, Zhijing Jin, Max Kleiman-Weiner, Bernhard Schölkopf, Mrinmaya Sachan, 和 Rada Mihalcea. 2024. [合作还是崩溃：LLM 代理社会中可持续合作的出现](https://doi.org/10.48550/arXiv.2404.16698). *预印本*, arXiv:2404.16698.

+   Qian 等人（2024a）Chen Qian, Yufan Dang, Jiahao Li, Wei Liu, Zihao Xie, YiFei Wang, Weize Chen, Cheng Yang, Xin Cong, Xiaoyin Che, Zhiyuan Liu, 和 Maosong Sun. 2024a. [软件开发代理的经验共学习](https://doi.org/10.18653/v1/2024.acl-long.305). 载于 *第62届计算语言学协会年会论文集（卷1：长篇论文）*, 第5628–5640页, 泰国曼谷. 计算语言学协会.

+   Qian 等人（2024b）Chen Qian, Wei Liu, Hongzhang Liu, Nuo Chen, Yufan Dang, Jiahao Li, Cheng Yang, Weize Chen, Yusheng Su, Xin Cong, Juyuan Xu, Dahai Li, Zhiyuan Liu, 和 Maosong Sun. 2024b. [ChatDev：用于软件开发的交流型代理](https://doi.org/10.18653/v1/2024.acl-long.810). 载于 *第62届计算语言学协会年会论文集（卷1：长篇论文）*, 第15174–15186页, 泰国曼谷. 计算语言学协会.

+   Qian 等人（2024c）Chen Qian, Zihao Xie, Yifei Wang, Wei Liu, Yufan Dang, Zhuoyun Du, Weize Chen, Cheng Yang, Zhiyuan Liu, 和 Maosong Sun. 2024c. [大规模语言模型基础的多代理协作扩展](https://doi.org/10.48550/arXiv.2406.07155). *预印本*, arXiv:2406.07155.

+   (59) Alec Radford, Jeffrey Wu, Rewon Child, David Luan, Dario Amodei, 和 Ilya Sutskever. 语言模型是无监督的多任务学习者.

+   Shen 等人（2024）Weizhou Shen, Chenliang Li, Hongzhan Chen, Ming Yan, Xiaojun Quan, Hehong Chen, Ji Zhang, 和 Fei Huang. 2024. [小型 LLM 是弱工具学习者：一种多 LLM 代理](https://doi.org/10.48550/arXiv.2401.07324). *预印本*, arXiv:2401.07324.

+   Shi 等人（2024）Haojun Shi, Suyu Ye, Xinyu Fang, Chuanyang Jin, Leyla Isik, Yen-Ling Kuo, 和 Tianmin Shu. 2024. [MuMA-ToM：多模态多代理心智理论](https://doi.org/10.48550/arXiv.2408.12574). *预印本*, arXiv:2408.12574.

+   Shinn 等人（2023）Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik Narasimhan, 和 Shunyu Yao. 2023. Reflexion: 使用语言代理的言语强化学习. *神经信息处理系统进展*, 36:8634–8652.

+   Tang et al. (2024) 唐训竹 Xunzhu Tang、金基洙 Kisub Kim、宋业伟 Yewei Song、塞德里克·洛斯里茨 Cedric Lothritz、李蓓 Bei Li、萨阿德·埃齐尼 Saad Ezzini、田浩烨 Haoye Tian、雅克·克莱因 Jacques Klein 和 泰戈温德·F·比桑德 Tegawende F. Bissyande. 2024. [CodeAgent: 用于代码审查的自主沟通智能体](https://doi.org/10.48550/arXiv.2402.02172). *预印本*, arXiv:2402.02172.

+   Wang et al. (2024a) 王磊 Lei Wang、马晨 Chen Ma、冯学阳 Xueyang Feng、张泽宇 Zeyu Zhang、杨浩 Hao Yang、张靖森 Jingsen Zhang、陈智远 Zhiyuan Chen、唐佳凯 Jiakai Tang、陈旭 Xu Chen、林彦凯 Yankai Lin、赵欣威 Wayne Xin Zhao、魏哲伟 Zhewei Wei 和 文继荣 Jirong Wen. 2024a. [基于大型语言模型的自主智能体调查](https://doi.org/10.1007/s11704-024-40231-1). *计算机科学前沿*, 18(6):186345.

+   Wang et al. (2024b) 王磊 Lei Wang、张靖森 Jingsen Zhang、杨浩 Hao Yang、陈智远 Zhiyuan Chen、唐佳凯 Jiakai Tang、张泽宇 Zeyu Zhang、陈旭 Xu Chen、林彦凯 Yankai Lin、宋瑞华 Ruihua Song、赵欣威 Wayne Xin Zhao、徐俊 Jun Xu、窦志诚 Zhicheng Dou、王俊 Jun Wang 和 文继荣 Ji-Rong Wen. 2024b. [基于大型语言模型的用户行为模拟](https://doi.org/10.48550/arXiv.2306.02552). *预印本*, arXiv:2306.02552.

+   Wang et al. (2024c) 王勤能 Qineng Wang、王子豪 Zihao Wang、苏颖 Ying Su、童航航 Hanghang Tong 和 宋杨秋 Yangqiu Song. 2024c. [重新思考大型语言模型推理的界限：多智能体讨论是否是关键？](https://doi.org/10.18653/v1/2024.acl-long.331). 载于 *第62届计算语言学协会年会论文集（第1卷：长篇论文）*，第6106-6131页，曼谷，泰国。计算语言学协会。

+   Wang et al. (2024d) 王振海龙 Zhenhailong Wang、毛绍光 Shaoguang Mao、吴文山 Wenshan Wu、葛涛 Tao Ge、魏福如 Furu Wei 和 姬恒 Heng Ji. 2024d. [释放大型语言模型中的新兴认知协同：通过多角色自我协作的任务求解智能体](https://doi.org/10.18653/v1/2024.naacl-long.15). 载于 *2024年北美计算语言学学会年会：人类语言技术会议论文集（第1卷：长篇论文）*，第257-279页，墨西哥城，墨西哥。计算语言学协会。

+   Xu et al. (2023a) 徐风利 Fengli Xu、张俊 Jun Zhang、高晨 Chen Gao、冯杰 Jie Feng 和 李勇 Yong Li. 2023a. [城市生成智能（UGI）：面向具身城市环境中的智能体的基础平台](https://doi.org/10.48550/arXiv.2312.11813). *预印本*, arXiv:2312.11813.

+   Xu et al. (2023b) 徐琳 Lin Xu、胡智远 Zhiyuan Hu、周大全 Daquan Zhou、任洪宇 Hongyu Ren、董臻 Zhen Dong、库尔特·凯特泽 Kurt Keutzer、黄锦堂 See Kiong Ng 和 冯家世 Jiashi Feng. 2023b. [MAgIC：基于大型语言模型的多智能体在认知、适应性、理性和协作方面的研究](https://doi.org/10.48550/arXiv.2311.08562). *预印本*, arXiv:2311.08562.

+   Xu et al. (2024) 许壮 Xu、王硕 Shuo Wang、李鹏 Peng Li、罗富文 Fuwen Luo、王晓龙 Xiaolong Wang、刘伟东 Weidong Liu 和 刘阳 Yang Liu. 2024. [探索大型语言模型在沟通游戏中的应用：狼人游戏的实证研究](https://doi.org/10.48550/arXiv.2309.04658). *预印本*, arXiv:2309.04658.

+   Xu et al. (2023c) 许泽莱 Zelai Xu、余超 Chao Yu、方菲 Fei Fang、王宇 Yu Wang 和 吴怡 Yi Wu. 2023c. 使用强化学习的语言智能体在狼人游戏中的战略博弈。

+   姚等人（2023）姚顺宇、赵杰弗里、余典、杜楠、Izhak Shafran、Karthik Narasimhan、曹元。2023. [ReAct：在语言模型中协同推理与行动](https://doi.org/10.48550/arXiv.2210.03629)。*Preprint*, arXiv:2210.03629。

+   尹等人（2024）尹翔宇、史楚乔、韩依墨、姜毅。2024. [PEAR：由多个大语言模型智能体驱动的鲁棒且灵活的相位恢复自动化框架](https://doi.org/10.48550/arXiv.2410.09034)。*Preprint*, arXiv:2410.09034。

+   于等人（2024）于晓燕、罗同旭、魏一凡、雷方宇、黄一铭、彭浩、朱立煌。2024. [Neeko：利用动态LoRA进行高效的多角色扮演智能体](https://doi.org/10.48550/arXiv.2402.13717)。*Preprint*, arXiv:2402.13717。

+   袁等人（2023）袁昊琪、张驰、王洪诚、谢飞扬、蔡鹏霖、董浩、陆宗庆。2023. [开放世界长时间跨度任务的技能强化学习与规划](https://doi.org/10.48550/arXiv.2303.16563)。*Preprint*, arXiv:2303.16563。

+   岳等人（2024）岳胜彬、王思源、陈伟、黄轩靖、魏忠宇。2024. [基于轨迹学习的协同多智能体框架用于知识密集型任务](https://doi.org/10.48550/arXiv.2407.09893)。*Preprint*, arXiv:2407.09893。

+   张等人（2023a）张灿耀、杨凯杰、胡思怡、王子豪、李广赫、孙逸航、张成、张兆伟、刘安吉、朱松春、常晓俊、张君歌、尹锋、梁义涛、杨耀东。2023a. 《ProAgent：构建具有主动协作能力的人工智能系统》*CoRR*。

+   张等人（2023b）张驰、杨兆、刘佳轩、韩宇成、陈欣、黄泽彪、傅斌、俞刚。2023b. [AppAgent：作为智能手机用户的多模态智能体](https://doi.org/10.48550/arXiv.2312.13771)。*Preprint*, arXiv:2312.13771。

+   张等人（2024a）张锦天、徐欣、张宁宇、刘瑞博、Bryan Hooi、邓书敏。2024a. [探索LLM智能体协作机制：社会心理学视角](https://doi.org/10.18653/v1/2024.acl-long.782)。在《第62届计算语言学协会年会（第一卷：长篇论文）》中，页面14544-14607，泰国曼谷。计算语言学协会。

+   张等人（2024b）张在彬、张永廷、李立军、邵静、高洪志、乔瑜、王立军、陆虎川、赵峰。2024b. [PsySafe：一种综合框架用于多智能体系统安全的心理攻击、防御和评估](https://doi.org/10.18653/v1/2024.acl-long.812)。在《第62届计算语言学协会年会（第一卷：长篇论文）》中，页面15202-15231，泰国曼谷。计算语言学协会。

+   Zhao 等人 (2024a) Jun Zhao, Can Zu, Hao Xu, Yi Lu, Wei He, Yiwen Ding, Tao Gui, Qi Zhang, 和 Xuanjing Huang. 2024a. [LongAgent: 通过多代理协作将语言模型扩展至128k上下文](https://doi.org/10.48550/arXiv.2402.11550). *预印本*, arXiv:2402.11550.

+   Zhao 等人 (2024b) Qinlin Zhao, Jindong Wang, Yixuan Zhang, Yiqiao Jin, Kaijie Zhu, Hao Chen, 和 Xing Xie. 2024b. [CompeteAI: 理解基于大语言模型的代理中的竞争动态](https://doi.org/10.48550/arXiv.2310.17512). *预印本*, arXiv:2310.17512.

+   Zhao 等人 (2024c) Xiutian Zhao, Ke Wang, 和 Wei Peng. 2024c. [一种选举方法来多样化基于LLM的多代理集体决策](https://doi.org/10.48550/arXiv.2410.15168). *预印本*, arXiv:2410.15168.

+   Zou 等人 (2023) Hang Zou, Qiyang Zhao, Lina Bariah, Mehdi Bennis, 和 Merouane Debbah. 2023. [无线多代理生成式AI: 从连接智能到集体智能](https://doi.org/10.48550/arXiv.2307.02757). *预印本*, arXiv:2307.02757.
