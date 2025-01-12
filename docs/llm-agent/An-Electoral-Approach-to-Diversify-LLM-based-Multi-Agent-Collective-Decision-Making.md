<!--yml

类别: 未分类

日期: 2025-01-11 12:03:25

-->

# 一种选举方法以多元化基于 LLM 的多智能体集体决策

> 来源：[https://arxiv.org/html/2410.15168/](https://arxiv.org/html/2410.15168/)

赵秀天

爱丁堡大学

x.zhao-103@sms.ed.ac.uk \And王科、彭伟

华为 IT 创新与研究中心

{wangke215, peng.wei1}@huawei.com

###### 摘要

现代大型语言模型（LLMs）在复杂任务解决中展示了协作协同能力，而集体决策（CDM）是基于 LLM 的多智能体协作框架中的关键组成部分。我们对 52 个近期相关系统的调查揭示了多样性严重缺乏，且集体决策严重依赖于独裁式和多数投票方式。通过社会选择理论的视角，我们审视了广泛采用的集体决策方法，并识别了其局限性。为了丰富当前基于 LLM 的集体决策领域，我们提出了 GEDI，这是一个选举式集体决策模块，整合了多种有序优先投票机制。我们的实证案例研究表明，通过在三个基准测试中的应用，某些集体决策方法的整合可以显著提升一些领先 LLM 的推理能力和稳健性，且无需复杂的系统设计。此外，我们发现某些集体决策机制即使在只有三个智能体的情况下，也能产生正向协同效应。基于投票的方法还表现出抗单点故障的稳健性，并且在命中率@k和主题影响力方面展现了多样性。¹¹1我们的代码和数据可在 [https://github.com/xiutian/GEDI](https://github.com/xiutian/GEDI) 获取

一种选举方法以多元化基于 LLM 的

多智能体集体决策

赵秀天 爱丁堡大学 x.zhao-103@sms.ed.ac.uk                        王科、彭伟 华为 IT 创新与研究中心 {wangke215, peng.wei1}@huawei.com

## 1 引言

尽管多智能体系统在大规模语言模型（LLM）出现之前就已持续受到关注，正如Wooldridge（[2009](https://arxiv.org/html/2410.15168v1#bib.bib84)）和Dorri等人（[2018](https://arxiv.org/html/2410.15168v1#bib.bib18)）的前期研究所证明的那样，LLM的最新进展显著激发了对基于LLM的智能体的兴趣。此外，新的技术，如有效的提示工程（Wei等人，[2023](https://arxiv.org/html/2410.15168v1#bib.bib81)）；Wang等人（[2023c](https://arxiv.org/html/2410.15168v1#bib.bib77)）和智能体互动方案（Yao等人，[2023](https://arxiv.org/html/2410.15168v1#bib.bib92)）；Shinn等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib67)）推动了关于协作LLM智能体的研究激增（Xi等人，[2023](https://arxiv.org/html/2410.15168v1#bib.bib86)）；Wang等人（[2023b](https://arxiv.org/html/2410.15168v1#bib.bib76)）。研究人员已将基于LLM的智能体部署到各种环境和场景中：从模拟小型社区（Liu等人，[2023a](https://arxiv.org/html/2410.15168v1#bib.bib50)）；Park等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib59)）到预测法庭判决（Hamilton，[2023](https://arxiv.org/html/2410.15168v1#bib.bib28)），制作数字化身（Jarrett等人，[2023](https://arxiv.org/html/2410.15168v1#bib.bib37)）；Yang等人（[2024](https://arxiv.org/html/2410.15168v1#bib.bib91)），以及参与基于对话的游戏（Xu等人，[2023a](https://arxiv.org/html/2410.15168v1#bib.bib88)）；Stepputtis等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib69)）；Li等人（[2023c](https://arxiv.org/html/2410.15168v1#bib.bib44)），等等。

图1：基于52个大规模语言模型（LLM）的多智能体协作系统中CDM方法的分布，显示出多样性严重不足。

![参见标题](img/9fd4fafbbea933651ab4a1f259edb79f.png)

然而，现有的基于大型语言模型（LLM）的多智能体协作研究主要集中在智能体之间的沟通和互动工作流程上。相比之下，另一个重要方面——集体决策（CDM）——似乎在很大程度上被忽视并过于简化。我们对52个最近的LLM协作系统的回顾（§ [2](https://arxiv.org/html/2410.15168v1#S2 "2 A Concise Survey on LLM-based Multi-Agent Collective Decision-Making ‣ An Electoral Approach to Diversify LLM-based Multi-Agent Collective Decision-Making")）揭示了这些系统要么指派一个“独裁者”智能体为小组做决策（Hao et al. ([2023](https://arxiv.org/html/2410.15168v1#bib.bib30))）；Nair et al. ([2023](https://arxiv.org/html/2410.15168v1#bib.bib54))，要么依赖于简单的多数投票（Chan et al. ([2023](https://arxiv.org/html/2410.15168v1#bib.bib8))）；Zhang et al. ([2023b](https://arxiv.org/html/2410.15168v1#bib.bib96))；Xu et al. ([2023b](https://arxiv.org/html/2410.15168v1#bib.bib89))，其中一个案例采用了功利主义方法（Jarrett et al. ([2023](https://arxiv.org/html/2410.15168v1#bib.bib37))）。

本研究通过社会选择理论的视角（Arrow et al. ([2010](https://arxiv.org/html/2410.15168v1#bib.bib3))）考察了常见的集体决策方法，并阐明了它们未能满足基本标准（§ [3](https://arxiv.org/html/2410.15168v1#S3 "3 A Social Choice Theory Perspective on Collective Decision-Making ‣ An Electoral Approach to Diversify LLM-based Multi-Agent Collective Decision-Making")）：独裁方法由于绝对依赖单一智能体而脆弱；虽然多数投票方法简单且直观无缺陷，但却不符合独立于无关选项（IIA）和孔多塞准则；功利主义方法违反了多数准则和孔多塞准则。这些偏离关键准则的做法可能会妨碍从个体偏好向LLM智能体间集体决策的过渡。

尽管阿罗定理（Arrow [1951](https://arxiv.org/html/2410.15168v1#bib.bib2)）建立了设计完美投票制CDM系统的公理性不可能性，我们仍然可以通过将多种CDM方法纳入基于LLM的多智能体框架来规避一些限制和风险。为此，我们开发了一个选举CDM模块GEDI（§ [4](https://arxiv.org/html/2410.15168v1#S4 "4 多元化LLM基础的多智能体CDM ‣ 一种多元化LLM基础的多智能体集体决策方法")），它提供了一些在此类框架中未曾测试过的CDM机制。为了评估各种CDM方法的潜在影响，我们进行了一个实证案例研究（§ [5](https://arxiv.org/html/2410.15168v1#S5 "5 MCQA基准的案例研究 ‣ 一种多元化LLM基础的多智能体集体决策方法")），该研究涉及三个多项选择问题回答（MCQA）基准：MMLU Hendrycks等人（[2021](https://arxiv.org/html/2410.15168v1#bib.bib32)），MMLU-Pro Wang等人（[2024a](https://arxiv.org/html/2410.15168v1#bib.bib78)），以及ARC-Challenge Clark等人（[2018](https://arxiv.org/html/2410.15168v1#bib.bib13)），并使用了不同大小和架构的多种模型。

我们的关键发现（§ [5.2](https://arxiv.org/html/2410.15168v1#S5.SS2 "5.2 主要结果 ‣ 5 MCQA基准的案例研究 ‣ 一种多元化LLM基础的多智能体集体决策方法")）如下：（1）应用CDM方法通常能在MCQA基准上获得比单智能体决策更好的结果，尽管这增加了计算成本；（2）协同效应的程度在很大程度上取决于骨干模型和基准。一些LLM在基于投票的方法下表现出显著的改进，而其他LLM在任何CDM下几乎没有效果；（3）大多数投票方法只需要最少的法定人数，甚至仅需三名智能体即可生效；（4）CDM方法对不可靠智能体、不同的命中率@$k$以及不同学科领域的影响具有不同的鲁棒性。

我们希望这些观察结果能够鼓励进一步评估基于LLM的多智能体框架的有效性，并为推动基于LLM的多智能体系统（MAS）提供宝贵的见解。

## 2 基于LLM的多智能体集体决策方法概述

### 2.1 背景

多智能体系统由多个具有自主行动和交互能力的计算元素（即“智能体”）组成（Wooldridge，[2009](https://arxiv.org/html/2410.15168v1#bib.bib84)）。在LLM出现之前，多智能体系统的研究已经是多个学科的焦点（Silver等，[2017](https://arxiv.org/html/2410.15168v1#bib.bib68)；Dorri等，[2018](https://arxiv.org/html/2410.15168v1#bib.bib18)）。随着LLM的迅速发展，研究者对将LLM作为智能体的兴趣大幅增加（Xi等，[2023](https://arxiv.org/html/2410.15168v1#bib.bib86)）。尤其是有效的提示设计的出现，大大提高了单个LLM智能体的性能：如思维链（Chain-of-Thought，Wei等，[2023](https://arxiv.org/html/2410.15168v1#bib.bib81)）、自我一致性（Self-Consistency，Wang等，[2023c](https://arxiv.org/html/2410.15168v1#bib.bib77)）、ReAct（Yao等，[2023](https://arxiv.org/html/2410.15168v1#bib.bib92)）、反射（Reflexion，Shinn等，[2023](https://arxiv.org/html/2410.15168v1#bib.bib67)）以及DiVeRSe（Li等，[2023e](https://arxiv.org/html/2410.15168v1#bib.bib47)）等。尽管单一智能体框架在某些自然语言处理任务中取得了显著成功，但它们常常在处理更复杂的挑战时遇到困难，如常识推理和长期规划（Wang等，[2023b](https://arxiv.org/html/2410.15168v1#bib.bib76)）。因此，一些研究者倡导多LLM智能体协作作为一种有前景的路径。

### 2.2 基于LLM的多智能体协作中的集体决策

集体决策（CDM）是指一组自主实体达成决策的过程（Bose等，[2017](https://arxiv.org/html/2410.15168v1#bib.bib5)）。这一现象在动物社会和人类社区中都非常普遍，许多跨学科的研究表明，集体决策通常能够比个人单独做出的决策产生更优结果（King和Cowlishaw，[2007](https://arxiv.org/html/2410.15168v1#bib.bib41)；Couzin等，[2011](https://arxiv.org/html/2410.15168v1#bib.bib14)）。

最近，LLM（大语言模型）的发展使得在基于LLM的多智能体系统中实现自我管理的集体决策过程成为可能。然而，我们对52个新提出的框架的调查表明，集体决策机制尚未得到足够的关注。具体而言，大多数系统要么依赖于单个智能体的专断判断（通常由预先指定的角色来执行），要么采用多数投票来进行决策。如图[1](https://arxiv.org/html/2410.15168v1#S1.F1.fig1 "Figure 1 ‣ 1 Introduction ‣ An Electoral Approach to Diversify LLM-based Multi-Agent Collective Decision-Making")所示，我们可以根据集体决策方法将当前基于LLM的多智能体系统分为四类：（1）专制型，（2）多数型，（3）功利型，以及（4）没有集体决策或未指定类型。

#### 专制型

在审阅的文献中，独裁方法最为流行。顾名思义，这是一种单一代理人统治的系统，在该系统中，一个通常是预先指定的代理人有权批准决策。然而，这种系统可以被视为“集体性”的，因为“独裁者”可能会接受其他代理人的咨询并与之沟通。

大多数独裁框架指定一个特殊代理人来监督协作、评估结果，并对系统级决策拥有最终发言权。该代理人有许多别名：‘领导者’ Hao 等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib30)）；D’Arcy 等人（[2024](https://arxiv.org/html/2410.15168v1#bib.bib15)），‘决策者’ Nair 等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib54)），‘指挥官’ Wu 等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib85)），‘批评者’ Li 等人（[2023a](https://arxiv.org/html/2410.15168v1#bib.bib42)），‘教师’ Jinxin 等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib40)），‘法官’ Liang 等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib49)）；Xiong 等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib87)）；Sun 等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib70)）；Talebirad 和 Nadiri（[2023](https://arxiv.org/html/2410.15168v1#bib.bib71)），‘评估者’ Tang 等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib73)），‘规划者’ Zhang 等人（[2023a](https://arxiv.org/html/2410.15168v1#bib.bib94)）；Fang 等人（[2024](https://arxiv.org/html/2410.15168v1#bib.bib22)），‘招募者’ Li 等人（[2023f](https://arxiv.org/html/2410.15168v1#bib.bib48)），‘检查员’ Hua 等人（[2024](https://arxiv.org/html/2410.15168v1#bib.bib35)）；Wang 等人（[2024b](https://arxiv.org/html/2410.15168v1#bib.bib80)），‘鉴别者’ Hang 等人（[2024](https://arxiv.org/html/2410.15168v1#bib.bib29)），‘任务代理人’ Li 等人（[2023b](https://arxiv.org/html/2410.15168v1#bib.bib43)），‘QA检查员’ Tang 等人（[2024](https://arxiv.org/html/2410.15168v1#bib.bib72)）。一些具体案例包括创建虚拟软件和游戏开发公司，承载各种角色的LLM代理人，以实现软件的快速和低成本开发 Qian 等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib61)）；Chen 等人（[2023a](https://arxiv.org/html/2410.15168v1#bib.bib9)）。特别地，Chen 等人（[2023b](https://arxiv.org/html/2410.15168v1#bib.bib10)）建议由一个由‘规划者’和‘观察者’组成的‘寡头’小组，而非单一决策者。

#### 多数投票

多数投票选出获得最多首选票的选项（即相对多数）。为简便起见，我们将多数投票（需要超过半数票，即绝对多数）和共识投票（要求所有代理人一致同意）视为多数投票的两种变体。

适应多数投票的框架通常引入多轮讨论以达成决议或多数同意（Xu et al.，[2023b](https://arxiv.org/html/2410.15168v1#bib.bib89)）。研究发现，多方代理辩论过程能够提高大型语言模型（LLMs）的事实性（Du et al.，[2023](https://arxiv.org/html/2410.15168v1#bib.bib19)）、推理能力（Zhang et al.，[2023b](https://arxiv.org/html/2410.15168v1#bib.bib96)）和金融交易表现（Li et al.，[2023d](https://arxiv.org/html/2410.15168v1#bib.bib46)）。Chan et al.（[2023](https://arxiv.org/html/2410.15168v1#bib.bib8)）还通过辩论提高了LLM代理在自然语言生成任务中的评估质量。Chen et al.（[2023d](https://arxiv.org/html/2410.15168v1#bib.bib12)）设计了自动团队组建和LLM代理专家招募的方法。Chen et al.（[2023c](https://arxiv.org/html/2410.15168v1#bib.bib11)）通过附加LLM代理自分配的“状态”值并测量其收敛性，量化了共识寻求过程。值得注意的是，Wang et al.（[2023d](https://arxiv.org/html/2410.15168v1#bib.bib79)）展示了单一LLM的多个“角色”也能进行“自我协作”。在某些情况下，多数投票被选用来匹配模拟的目标场景。Hamilton（[2023](https://arxiv.org/html/2410.15168v1#bib.bib28)）训练了九个独立的代理作为法官，模拟美国最高法院，并在96个真实世界案例中达到了比随机判断更高的预测准确率。在像狼人杀（Werewolf）（Xu et al.，[2023a](https://arxiv.org/html/2410.15168v1#bib.bib88)）和阿瓦隆（Avalon）（Stepputtis et al.，[2023](https://arxiv.org/html/2410.15168v1#bib.bib69)）；Shi et al.（[2023](https://arxiv.org/html/2410.15168v1#bib.bib66)）这样的文字或概念类游戏中，代理受游戏规则的限制，必须采用这种方法。

#### 功利主义

功利主义方法量化可能决策的影响，并选择最大化集体“效用”或“奖励”的决策。然而，功利主义与其他方法不同，它不是自我治理的：效用是外部预定或更新的。Jarrett et al.（[2023](https://arxiv.org/html/2410.15168v1#bib.bib37)）提出通过功利主义的“支付函数”训练LLM代理作为数字代理，以代表个体偏好。虽然在新提出的基于LLM的框架中功利主义较为罕见，但它在许多早期的非LLM多代理系统中是一个重要的方法（Dorri et al.，[2018](https://arxiv.org/html/2410.15168v1#bib.bib18)）。

#### 无CDM或未指定

一些多智能体场景不需要CDM。例如，模拟LLM智能体之间的社会互动和行为时，Park等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib59)）；Liu等人（[2023a](https://arxiv.org/html/2410.15168v1#bib.bib50)）；Ghaffarzadegan等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib26)）；Hua等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib34)）；Zhang等人（[2024](https://arxiv.org/html/2410.15168v1#bib.bib95)）；Wei等人（[2024](https://arxiv.org/html/2410.15168v1#bib.bib82)）指出，偶尔会发生一对一的协议达成。而其他系统则本质上否定CDM过程，比如严格线性的协作工作流程Hong等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib33)）；Wang等人（[2023a](https://arxiv.org/html/2410.15168v1#bib.bib75)）；Ding等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib17)）；Rasheed等人（[2024](https://arxiv.org/html/2410.15168v1#bib.bib63)）或去中心化的团队安排Li等人（[2023c](https://arxiv.org/html/2410.15168v1#bib.bib44)）；Nakajima（[2023](https://arxiv.org/html/2410.15168v1#bib.bib55)）；He等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib31)）。此外，一些框架涉及到人为判断来做出系统级的决策，Ghafarollahi和Buehler（[2024](https://arxiv.org/html/2410.15168v1#bib.bib25)）；Ni和Buehler（[2024](https://arxiv.org/html/2410.15168v1#bib.bib56)）。

到目前为止，在基于大语言模型（LLM）的多智能体协作中，我们观察到CDM方法的多样性严重不足，因此我们从社会选择理论中汲取灵感，仔细分析了广泛使用的各种方法的优缺点。

## 3 从社会选择理论的角度看集体决策

社会选择理论关注的是如何从个体偏好推导出集体决策，Arrow等人（[2010](https://arxiv.org/html/2410.15168v1#bib.bib3)）。尽管人类自古以来就已经在实践和完善集体决策，但现代社会选择理论直到Kenneth J. Arrow的著名著作《社会选择与个体价值》（Arrow（[1951](https://arxiv.org/html/2410.15168v1#bib.bib2)））发布后才得以建立，该书公理化地形式化了这一理论，并对各种选举系统进行了比较分析。

### 3.1 将社会选择理论融入自然语言处理研究的相关工作

迄今为止，相关研究主要集中在将社会选择理论应用于模型对齐（Mishra，[2023](https://arxiv.org/html/2410.15168v1#bib.bib53)），模型集成（Jiang 等人，[2023b](https://arxiv.org/html/2410.15168v1#bib.bib39)），文本生成与偏好推断（Fish 等人，[2023](https://arxiv.org/html/2410.15168v1#bib.bib23)）。更具体地说，Jarrett 等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib37)）采用功利主义方法，利用 LLM 智能体作为人类的数字代表。Irurozki 等人（[2022](https://arxiv.org/html/2410.15168v1#bib.bib36)）；Rofin 等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib64)）指出了 NLP 基准测试中经典平均值聚合的局限性，并提出了基于社会选择理论的新聚合方法。Wang 等人（[2023c](https://arxiv.org/html/2410.15168v1#bib.bib77)）；Xue 等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib90)）提出通过多元投票从多个生成的推理路径中选择答案，并在结果上超过了功利主义方法。

最近，Li 等人（[2024](https://arxiv.org/html/2410.15168v1#bib.bib45)）展示了多元投票在 GPT-3.5（Ouyang 等人，[2022](https://arxiv.org/html/2410.15168v1#bib.bib58)）和 Llama-2（Touvron 等人，[2023](https://arxiv.org/html/2410.15168v1#bib.bib74)）上的协同效应，呼应了我们的一些发现，但缺乏与其他 CDM 方法的比较。另一项同期研究，Yang 等人（[2024](https://arxiv.org/html/2410.15168v1#bib.bib91)）从投票行为的角度探讨了人类与 LLM 之间的差异。然而，之前的研究并未涉及我们主要目标，即多样化基于 LLM 的多智能体 CDM 方法。

### 3.2 对流行的 LLM 基础多智能体协作 CDM 方法的批评

在 LLM-agent 协作的背景下，专制方法依赖于一个由其他智能体提供信息和建议以为群体做出决策的单一智能体。虽然专制方法在计算上通常是高效的，但它对单一智能体的绝对依赖使其比更“民主”的过程更具偏见，且鲁棒性较差。

相比之下，功利主义和基数投票方法无疑能够汇总并公开来自群体成员的更广泛的个体偏好。然而，这些方法一个不可忽视的缺点是外部施加的效用具有不稳定性和随意性，Brandt等人([2016](https://arxiv.org/html/2410.15168v1#bib.bib7))指出。如果代理人对选择有准确的基数效用，这是一个较强的假设，那么效用的不均匀分布是另一个潜在的担忧：这样的系统可能很容易违反多数标准（见图[11](https://arxiv.org/html/2410.15168v1#A4.F11 "图 11 ‣ 附录 D 多种CDM方法标准示例 ‣ 一种多样化LLM基础的多代理集体决策的方法")），甚至在一个拥有主导效用影响的代理存在时，可能会崩溃成专制。

多数投票法展示了有序投票（也称为偏好或排名投票）的一种典型例子，这是另一种去中心化的决策方法家族。虽然还有其他广泛实践的有序投票方法，根据我们的了解，所有现有的使用投票方法的LLM-代理协作框架都选择了多数投票法，如图[1](https://arxiv.org/html/2410.15168v1#S1.F1.fig1 "图 1 ‣ 1 引言 ‣ 一种多样化LLM基础的多代理集体决策的方法")所示。这种简单的方法可能直观上看起来“安全”。然而，通过阿罗定理的视角来看，阿罗([1951](https://arxiv.org/html/2410.15168v1#bib.bib2))，这种方法与一些相当显而易见的标准相矛盾。举两个例子，该方法违反了与无关替代选项独立性（IIA）标准，如图[8](https://arxiv.org/html/2410.15168v1#A4.F8 "图 8 ‣ 附录 D 多种CDM方法标准示例 ‣ 一种多样化LLM基础的多代理集体决策的方法")所示，也违反了孔多塞标准，如图[9](https://arxiv.org/html/2410.15168v1#A4.F9 "图 9 ‣ 附录 D 多种CDM方法标准示例 ‣ 一种多样化LLM基础的多代理集体决策的方法")所示。

|

&#124; CDM &#124;

&#124; 方法 &#124;

|

&#124; 主要 &#124;

&#124; -ity &#124;

|

&#124; Mono &#124;

&#124; -tonic &#124;

|

&#124; 一致性 &#124;

&#124; -tency &#124;

| IIA |
| --- |

&#124; Cond &#124;

&#124; -orcet &#124;

|

&#124; 投票 &#124;

&#124; 类型 &#124;

|

| --- | --- | --- | --- | --- | --- | --- |
| --- | --- | --- | --- | --- | --- | --- |
| 专制（盲目） | ✗ | ✓ | ✓ | ✓ | ✗ | 排名 |
| 范围投票 | ✗ | ✓ | ✓ | ✓ | ✗ | 得分 |
| 多数 | ✓ | ✓ | ✓ | ✗ | ✗ | 单一* |
| 博尔达计数法 | ✗ | ✓ | ✓ | ✗ | ✗ | 排名 |
| IRV | ✓ | ✗ | ✗ | ✗ | ✗ | 排名 |
| 排名配对法 | ✓ | ✓ | ✗ | ✗ | ✓ | 排名 |

表1：一些典型集体决策方法的标准符合度。范围投票可以看作是一种特殊的功利主义方法。IIA表示“与无关备选方案的独立性”。*单一选票可以从排名中推导出来。更多例子请见附录 [D](https://arxiv.org/html/2410.15168v1#A4 "附录 D 若干集体决策方法标准示例 ‣ 多样化基于LLM的多智能体集体决策方法")。有关即时决胜法（IRV）不符合单调性标准的示例，请参见图 [10](https://arxiv.org/html/2410.15168v1#A4.F10 "图 10 ‣ 附录 D 若干集体决策方法标准示例 ‣ 多样化基于LLM的多智能体集体决策方法")。

实际上，阿罗定理在数学上证明了每一个选举系统都有一些根本性缺陷，如表 [1](https://arxiv.org/html/2410.15168v1#S3.T1 "表 1 ‣ 3.2 对基于LLM的多智能体协作中流行集体决策方法的批评 ‣ 3 从社会选择理论视角看集体决策 ‣ 多样化基于LLM的多智能体集体决策方法") 所示。尽管如此，构建完美投票系统的公理性不可行性激励我们减少陷入单一失败点的风险。因此，我们认为将现代去中心化投票系统引入当前的LLM-智能体环境是具有重要现实意义的。为了更好地利用LLM-智能体基于自然语言的“判断”，而非强加的“效用”或“奖励”，我们特别强调序数优先投票。

## 4 多样化基于LLM的多智能体集体决策模型

为了增强LLM-智能体框架内集体决策方法的多样性，我们提出了结合多种基于人类社会政治实践的集体决策方法。具体而言，我们设计了一个选举式集体决策模块，命名为通用选举决策接口（GEDI），该模块整合了几种常见的序数优先投票系统。图 [2](https://arxiv.org/html/2410.15168v1#S4.F2 "图 2 ‣ 4 多样化基于LLM的多智能体集体决策方法 ‣ 多样化基于LLM的多智能体集体决策方法的选举方式") 展示了GEDI与其他常见集体决策方法在LLM基础的MAS中的几个主要区别。

![参见说明文字](img/ea8d32e7494fd4b43c94b8f6c2c88a29.png)

图2：不同基于LLM的多智能体集体决策模型结构的比较：功利主义、独裁主义、多元化以及我们的扩展。议程指的是分配的任务或互动环境。蓝色和绿色箭头分别表示智能体之间的互动以及向集体决策系统传递偏好的沟通。GEDI与其不同之处在于，它不仅生成一个单一的决策，而是独特地输出序数排名，提供了更多关于智能体集体偏好的信息。

### 4.1 定义

按照传统做法，Arrow 等人（[2010](https://arxiv.org/html/2410.15168v1#bib.bib3)）；Brandt 等人（[2016](https://arxiv.org/html/2410.15168v1#bib.bib7)）考虑了一个多备选决策过程。设$N=\{1,2,...,n\}$为一个有限的$n$个代理的集合，$A=\{a_{1},a_{2},...a_{m}\}$为一个有限的$m$个不同备选的集合，其中$m\geq 2$且对于所有$a,b\in A,a\neq b$。偏好排序投票（即，投票）可以定义为$A$的严格部分排序$\succ$，参见Rosen（[2007](https://arxiv.org/html/2410.15168v1#bib.bib65)）。具体来说，$\succ$是传递的：对于所有$a,b,c\in A$，如果$a\succ b$且$b\succ c$，则$a\succ c$）；并且是完备的：对于所有$a,b\in A$，$a\succ b$或$a\prec b$。请注意，还有一种弱排序变体，允许选民表示对两个备选方案的无差异（即，偏好平局）。

具体来说，GEDI的输入由以下组成：(1)一个配置文件$P=(\succ_{1},\succ_{2},...\succ_{n})$，表示来自每个选民$i\in N$的选票集合；(2)一个投票系统（即，社会选择函数(SCF)），定义为一个映射$f:\mathcal{L}(A)^{n}\to\mathcal{C}(A)$，它为每个严格偏好配置文件返回一组备选方案。输出$f(P)$是备选集$A$的一个非空有序子集。

### 4.2 评估的选举方法

我们选择了10种CDM方法，在以下案例研究中进行评估：盲独裁式、知情独裁式、信息不对称的独裁式、范围投票、多数制、博尔达计数、巴克林、最小最大法、排名对以及随机基线。

#### 独裁式

盲独裁式（或随机投票）任意选择一个代理，并采纳其偏好排序作为决策，而不与其他代理协商，参见Aziz 等人（[2013](https://arxiv.org/html/2410.15168v1#bib.bib4)）。另外，知情独裁式是一种建议版本，其中一个“独裁者”代理首先审查投票集合的选票，然后做出自己的决策。我们还引入了一种信息不对称的变体，以验证通过选票传递的信息的影响，其中“独裁者”是通过随机选票而非实际选票做出咨询的。值得注意的是，盲独裁式仍然需要完整的选票集合，这将其与随机基线区分开来，后者以随机方式安排偏好排序，而没有实际投票。

#### 范围投票

代理在指定的基数范围内对备选方案进行评分，获胜者由整体得分最高者选出，参见Menton（[2013](https://arxiv.org/html/2410.15168v1#bib.bib52)）。这种方法类似于功利主义方法，但“效用”（即，整体得分）由代理给出，而非外部分配。

#### 多数制

简单多数制（相对多数）仅考虑每个选票中的第一偏好，忽略任何后续的偏好。获胜者是获得最多第一选票的候选人。

#### 巴克林投票

首选票首先被统计，如果没有选项获得绝对多数，则接下来会统计次选票。这个过程会重复，直到出现一个绝对多数的获胜者。Erdélyi 等人 ([2015](https://arxiv.org/html/2410.15168v1#bib.bib21))。

#### 博尔达计数

选项根据每张选票上的位置获得积分，总积分决定获胜者。Emerson ([2013](https://arxiv.org/html/2410.15168v1#bib.bib20))；Davies 等人 ([2014](https://arxiv.org/html/2410.15168v1#bib.bib16))。在标准博尔达计数中，对于 $m$ 个选项的偏好选票，第 $i$ 位选项将获得 $m-i$ 分。与范围投票不同，博尔达法与功利主义方法不同，因为博尔达仍然使用序数偏好。

#### 即时淘汰投票法（IRV）

一种多轮机制，反复淘汰首选票最少的选项，并将被淘汰选项的票数“转移”到幸存的选项。Freeman 等人 ([2014](https://arxiv.org/html/2410.15168v1#bib.bib24))。淘汰的逆序构成了按集体偏好排序的选项列表。虽然有多种早期退出设计（例如，一旦出现绝对多数即选择获胜者），我们包括了标准的 IRV 来获得完整的排序列表。

#### 最小化

一种选择最少“最坏反感”选项的方法，Brams 等人 ([2007](https://arxiv.org/html/2410.15168v1#bib.bib6))。形式上，设 $f(a,b)$ 表示 $a$ 相对于 $b$ 的总体“偏好”（即，$a$ 在所有选票中战胜 $b$ 的次数），对于一对不同的选项 $a,b\in A$。如果更多选民偏好 $b$ 而非 $a$，则 $f(a,b)$ 可以为负值。$a$ 的“最坏反感”定义为 $\max f(b,a)$。获胜选项是最小化最坏反感的选项，即，最小化 $\max f(b,a)$ 的选项：$a_{w}=\arg\min\limits_{a}(\max\limits_{b}f(b,a))$。

#### 排名配对法

具体来说，设 $(a,b)$ 表示 $a,b\in A$ 的汇总配对比较结果，若 $(a,b)$ 为正值，表示更多代理人偏好 $a$ 而非 $b$。排名配对法将完整的选票拆分为优先对，并按其出现频率进行排序。从最常见的配对开始（即，$\arg\max\limits_{a,b\in A}(a,b)$），该方法填充配对比较矩阵，标记配对及其传递结果为正，忽略频率较小的冲突配对。获胜者是 $\arg\limits_{a\in A}((a,b)>0)\ s.t.\ \forall b\in A,a\neq b$，即在所有其他选项上都有正号的选项。

| 基础模型 | 随机 | 分数 | 独裁型 | 序数排名 |
| --- | --- | --- | --- | --- |

|

&#124; MMLU &#124;

| 随机 |
| --- |

&#124; 范围 &#124;

&#124; 投票 &#124;

|

&#124; 盲选 &#124;

&#124; 法则 &#124;

|

&#124; 知情 &#124;

&#124; 法则 &#124;

|

&#124; 错误知情 &#124;

&#124; 法则 &#124;

| 多数制 | 巴克林 |
| --- | --- |

&#124; 博尔达 &#124;

&#124; 计数 &#124;

| IRV | 最小化 |
| --- | --- |

&#124; 排名 &#124;

&#124; 配对 &#124;

|

| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| mistral-7b | 24.8 | 51.8 (-4.6) | 56.4 | 55.9 (-0.5) | 36.1 (-20.3) | 56.8 (+0.4) | 57.1 (+0.7) | 56.9 (+0.5) | 56.9 (+0.5) | 57.0 (+0.6) | 57.0 (+0.6) |
| llama-3-8b | 25.0 | 37.7 (-7.3) | 45.0 | 36.5 (-8.5) | 32.2 (-12.8) | 45.9 (+0.9) | 46.4 (+1.4) | 46.3 (+1.3) | 45.7 (+0.7) | 45.9 (+0.9) | 46.0 (+1.0) |
| glm-4-9b | 25.2 | 61.3 (-0.4) | 61.7 | 54.3 (-7.4) | 53.0 (-8.7) | 64.6 (+2.9) | 64.5 (+2.8) | 64.1 (+2.4) | 64.9 (+3.2) | 64.4 (+2.7) | 64.6 (+2.9) |
| llama-3-70b | 25.3 | 74.9 (+1.6) | 73.3 | 70.1 (-3.2) | 62.6 (-10.7) | 73.9 (+0.6) | 73.8 (+0.5) | 73.7 (+0.4) | 73.9 (+0.6) | 73.9 (+0.6) | 73.9 (+0.6) |
| qwen-2-72b | 25.1 | 69.2 (-0.5) | 69.7 | 69.7 ($\pm$0.0) | 39.5 (-30.2) | 70.0 (+0.3) | 69.9 (+0.2) | 70.0 (+0.3) | 69.9 (+0.2) | 69.9 (+0.2) | 69.9 (+0.2) |
| qwen-1.5-110b | 25.0 | 71.3 (-1.5) | 72.8 | 73.0 (+0.2) | 46.3 (-26.5) | 72.9 (+0.1) | 72.9 (+0.1) | 72.7 (-0.1) | 72.9 (+0.1) | 72.9 (+0.1) | 72.9 (+0.1) |
| gpt-3.5 | 24.9 | 63.0 (+2.2) | 60.8 | 64.7 (+3.9) | 36.9 (-23.9) | 65.9 (+5.1) | 65.5 (+4.7) | 65.6 (+4.8) | 65.6 (+4.8) | 65.6 (+4.8) | 65.6 (+4.8) |
| gpt-4 | 25.0 | 80.7 (+5.1) | 75.6 | 82.1 (+6.5) | 70.9 (-4.7) | 82.5 (+6.9) | 81.9 (+6.3) | 81.9 (+6.3) | 81.9 (+6.3) | 81.9 (+6.3) | 81.9 (+6.3) |
| MMLU-Pro |  |  |  |  |  |  |  |  |  |  |  |
| mistral-7b | 9.6 | 20.9 (-9.0) | 29.9 | 27.7 (-2.2) | 15.6 (-14.3) | 31.7 (+1.8) | 30.7 (+0.8) | 31.4 (+1.5) | 31.2 (+1.3) | 31.7 (+1.8) | 31.7 (+1.8) |
| llama-3-8b | 9.7 | 18.9 (-2.4)* | 21.3 | 23.8 (+2.5) | 19.3 (-2.0) | 22.2 (+0.9) | 23.8 (+2.5) | 24.5 (+3.2) | 22.6 (+1.3) | 23.0 (+1.7) | 23.4 (+2.1) |
| glm-4-9b | 9.6 | 26.2 (-5.7)* | 31.9 | 28.2 (-3.7) | 23.9 (-8.0) | 36.4 (+4.5) | 35.9 (+4.0) | 34.8 (+2.9) | 36.7 (+4.8) | 35.6 (+3.7) | 36.2 (+4.3) |
| llama-3-70b | 10.3 | 46.7 (+3.5) | 43.2 | 44.6 (+1.4) | 24.6 (-18.6) | 42.8 (-0.4) | 43.5 (+0.3) | 43.6 (+0.4) | 43.0 (-0.2) | 43.2 ($\pm$0.0) | 43.5 (+0.3) |
| qwen-2-72b | 10.4 | 35.1 (-1.7) | 36.8 | 37.4 (+0.6) | 19.5 (-17.3) | 37.2 (+0.4) | 36.7 (-0.1) | 36.7 (-0.1) | 37.2 (+0.4) | 37.3 (+0.5) | 37.2 (+0.4) |
| qwen-1.5-110b | 10.1 | 45.7 (+0.9) | 44.8 | 42.8 (-2.0) | 16.6 (-28.2) | 44.7 (-0.4) | 44.9 (+0.1) | 44.6 (-0.2) | 45.1 (+0.3) | 45.0 (+0.2) | 44.8 ($\pm$0.0) |
| gpt-3.5 | 9.9 | 28.5 (+2.6) | 25.9 | 27.1 (+1.2) | 13.0 (-12.9) | 26.5 (+0.6) | 27.0 (+1.1) | 28.5 (+2.6 |
| gpt-4 | 9.9 | 46.4 (-0.5) | 46.9 | 46.9 ($\pm$0.0) | 34.6 (-12.3) | 47.3 (+0.4) | 47.5 (+0.6) | 47.7 (+0.8) | 47.5 (+0.6) | 47.8 (+0.9) | 47.7 (+0.8) |
| ARC-Challenge |  |  |  |  |  |  |  |  |  |  |  |
| mistral-7b | 24.9 | 53.1 (-17.9) | 71.0 | 70.3 (-0.7) | 47.7 (-23.3) | 71.7 (+0.7) | 71.7 (+0.7) | 71.6 (+0.6) | 71.7 (+0.7) | 71.7 (+0.7) | 71.6 (+0.6) |
| llama-3-8b | 25.2 | 44.4 (-21.8) | 66.2 | 52.8 (-13.4) | 41.1 (-25.1) | 71.3 (+5.1) | 70.0 (+3.8) | 70.0 (+3.8) | 71.6 (+5.4) | 71.3 (+5.1) | 71.3 (+5.1) |
| glm-4-9b | 24.8 | 69.9 (-9.7)* | 79.3 | 80.1 (+0.8) | 65.1 (-14.2) | 82.7 (+3.4) | 82.3 (+3.0) | 82.0 (+2.7) | 82.8 (+3.5) | 83.0 (+3.7) | 82.7 (+3.4) |
| llama-3-70b | 25.3 | 88.9 (+1.1) | 87.8 | 87.9 (+0.1) | 80.8 (-7.0) | 88.5 (+0.7) | 88.4 (+0.6) | 88.1 (+0.3) | 88.5 (+0.7) | 88.4 (+0.6) | 88.4 (+0.6) |
| qwen-2-72b | 24.8 | 84.7 (-1.1) | 85.8 | 86.0 (+0.2) | 36.7 (-49.1) | 86.3 (+0.5) | 86.2 (+1.3) | 85.8 ($\pm$0.0) | 86.3 (+0.5) | 86.3 (+0.5) | 86.2 (+0.4) |
| qwen-1.5-110b | 24.7 | 87.0 (-0.7) | 87.7 | 88.3 (+0.6) | 53.4 (-34.3) | 88.1 (+0.4) | 88.1 (+0.4) | 88.0 (+0.3) | 88.1 (+0.4) | 88.1 (+0.4) | 88.1 (+0.4) |
| gpt-3.5 | 25.2 | 78.1 (+1.2) | 76.9 | 77.0 (+0.1) | 29.9 (-47.0) | 78.2 (+1.3) | 77.9 (+1.0) | 78.2 (+1.3) | 78.1 (+1.2) | 77.9 (+1.0) | 77.9 (+1.0) |
| gpt-4 | 25.0 | 92.9 (+0.4) | 92.5 | 92.8 (+0.3) | 87.3 (-5.2) | 92.9 (+0.4) | 92.7 (+0.2) | 92.8 (+0.3) | 92.8 (+0.3) | 92.8 (+0.3) | 92.9 (+0.4) |

表 2：在 MMLU、MMLU-Pro 和 ARC-Challenge 基准上的整体准确率结果。“Rand.” 和 “Dicta.” 分别表示 “random” 和 “dictatorial”。括号中的数字表示相对于盲目专制基准的变化。性能提升用红色标记，下降用蓝色标记。显著情况用粗体标出。*带星号的结果是利用部分配置文件计算得出的（见附录[C](https://arxiv.org/html/2410.15168v1#A3 "Appendix C Main Experiment Statistics ‣ An Electoral Approach to Diversify LLM-based Multi-Agent Collective Decision-Making"))。

## 5 关于 MCQA 基准的案例研究

### 5.1 实验设置

#### 数据集

本研究的主要范围集中在决策过程而非选择生成上，MCQA 基准特别适合该研究兴趣，因为它们已经预定义了备选方案（即选择）。继承先前关于 LLM 代理通用性能基准测试的研究工作 Park 等人 ([2022](https://arxiv.org/html/2410.15168v1#bib.bib60)); Liu 等人 ([2023b](https://arxiv.org/html/2410.15168v1#bib.bib51)); Zhang 等人 ([2023b](https://arxiv.org/html/2410.15168v1#bib.bib96)); Google ([2023](https://arxiv.org/html/2410.15168v1#bib.bib27)); Jiang 等人 ([2023a](https://arxiv.org/html/2410.15168v1#bib.bib38)), 我们选择了 MMLU Hendrycks 等人 ([2021](https://arxiv.org/html/2410.15168v1#bib.bib32)), MMLU-Pro Wang 等人 ([2024a](https://arxiv.org/html/2410.15168v1#bib.bib78)) 和 ARC-Challenge Clark 等人 ([2018](https://arxiv.org/html/2410.15168v1#bib.bib13)) 作为案例研究的测试平台。

#### 主干模型

为了模拟基于不同架构和参数规模的语言模型构建的智能体，我们整理了六个开源模型的集合，包括mistral-7b Jiang等人（[2023a](https://arxiv.org/html/2410.15168v1#bib.bib38)）、glm-4-9b Zeng等人（[2023](https://arxiv.org/html/2410.15168v1#bib.bib93)）、llama-3-8b/70b AI@Meta（[2024](https://arxiv.org/html/2410.15168v1#bib.bib1)）以及qwen-1.5-72b/110b Qwen（[2024](https://arxiv.org/html/2410.15168v1#bib.bib62)）。此外，由于大多数现有的基于LLM的多智能体协作框架使用具有巨大参数规模的高性能模型来创建智能体，跟随这些框架的做法，我们还测试了两个广泛使用的专有模型：gpt-3.5 Ouyang等人（[2022](https://arxiv.org/html/2410.15168v1#bib.bib58)）和gpt-4 OpenAI（[2023](https://arxiv.org/html/2410.15168v1#bib.bib57)）。所选模型的规格可以在附录[A](https://arxiv.org/html/2410.15168v1#A1 "附录 A 可重复性声明 ‣ 一种多样化LLM基础多智能体集体决策方法")表[5](https://arxiv.org/html/2410.15168v1#A3.T5 "表5 ‣ 附录C 主要实验统计 ‣ 一种多样化LLM基础多智能体集体决策方法")中找到。除OpenAI模型外，所有模型的温度都固定为0.7（范围在0.0到1.0之间），而OpenAI模型的温度保持在1.0（范围在0.0到2.0之间）。

#### 指标与评估

为了简化，我们使用未经修改的语言模型作为测试代理，并在问题前加上简短的指令“你是第{随机数字}号评分员”来识别它们。决策集成由一个固定数量的代理组成，这些代理基于相同的骨干模型构建。每个代理都被要求独立地对每个问题提供选择的优先排序。在收集所有排序（即投票）形成配置文件后，GEDI根据选定的投票规则输出集体优先级排名。独特的是，在知情和误导性专制规则下，除了投票集成外，还会给“独裁者”代理提供其他代理的投票，然后对其进行询问（参见§ [4.2](https://arxiv.org/html/2410.15168v1#S4.SS2 "4.2 评估的选举方法 ‣ 4 基于LLM的多代理集体决策方法的多样化 ‣ 通过选举方法多样化基于LLM的多代理集体决策")）。如§ [4.1](https://arxiv.org/html/2410.15168v1#S4.SS1 "4.1 定义 ‣ 4 基于LLM的多代理集体决策方法的多样化 ‣ 通过选举方法多样化基于LLM的多代理集体决策")中所述，给定一个包含来自代理的10个优先排序的配置文件$P$，GEDI的投票系统$f$输出一个所有选择的有序列表$f(P)$。我们认为，如果输出列表的第一个元素与相应的黄金标签匹配，则该问题被正确回答。根据MMLU Hendrycks等人（[2021](https://arxiv.org/html/2410.15168v1#bib.bib32)）的原始设置，我们实现了一个5次示例提示，利用数据集的开发集。所有方法采用相同的优先排序格式投票，除了范围投票，它需要除了排序外，还需要数字化的优先得分。

### 5.2 主要结果

5次运行的平均总体准确率结果报告在表[2](https://arxiv.org/html/2410.15168v1#S4.T2 "表2 ‣ 排名对 ‣ 4 评估的选举方法 ‣ 4 基于LLM的多代理集体决策方法的多样化 ‣ 通过选举方法多样化基于LLM的多代理集体决策")中，相关的有效排名配置文件统计信息详见附录[C](https://arxiv.org/html/2410.15168v1#A3 "附录C 主要实验统计 ‣ 通过选举方法多样化基于LLM的多代理集体决策")。

#### 随机基准和范围投票

随机基准的准确率在4选项MMLU和ARC挑战中徘徊在25.0左右，在10选项MMLU-Pro中大约为10.0。这些数据确认了测试集中正确选择的平衡分布。大多数模型，尤其是较小的模型，在实施基于得分的范围投票（即基数排序）时，相较于序数排序方法，表现较差。然而，llama-3-70b、gpt-3.5和gpt-4是例外，它们的范围投票结果超过了盲目专制的结果。

#### 专制方法

表格[2](https://arxiv.org/html/2410.15168v1#S4.T2 "Table 2 ‣ Ranked Pairs ‣ 4.2 Assessed Electoral Methods ‣ 4 Diversifying LLM-based Multi-Agent CDM ‣ An Electoral Approach to Diversify LLM-based Multi-Agent Collective Decision-Making")中的彩色数字表示与盲目独裁方法的比较结果，盲目独裁作为对比基线。尽管大多数模型在有信息的独裁下的表现优于盲目独裁，但它们并没有超越其他顺序排名方法。

需要注意的是，有信息的独裁方法在计算上比基于投票的方法更为复杂，因为它需要来自整个团队的完整选票档案，除了‘独裁者’外。信息独裁表现不佳表明，‘独裁者’代理无法比投票系统更有效地利用来自团队选票的信息。

正如预期的那样，在错误信息的独裁下准确率显著下降，表明向‘独裁者’提供随机选票的有害影响。值得注意的是，glm-4-9b和gpt-4在三个数据集中的表现相对较小的下降，显示出它们对误导性信息的韧性。

#### 顺序排名方法

一贯观察到，应用基于投票的顺序排名方法，即使是像简单多数法这样直接的方法，也能使准确率达到或超过盲目独裁方法的表现。改进的程度取决于具体的模型。值得注意的是，基于较小模型（<10B）以及GPT系列的模型在采用选举CDM方法时表现出了显著的性能提升，这与中型模型（10-110B）形成鲜明对比。

对于MMLU基准测试，采用投票方法使得glm-4-9b、gpt-3.5和gpt-4的平均准确率分别提高了大约2.9%、4.9%和6.5%。鉴于MMLU-Pro是一个10选项的测试，因此由于CDM的相对改进可能看起来不那么显著。然而，llama-3-8b和glm-4-9b在投票方法下仍然表现出明显的准确率提升。特别是，minimax和排名对方法表现出较强的鲁棒性，在三个基准测试中对所有模型都产生了积极的影响。

这些发现要求重新评估现有的LLM-代理协作框架，特别是对于其提议的系统所产生的影响在多大程度上可以归因于特定CDM方法的实施。然而，也观察到一些CDM方法在某些模型上的表现差异微乎其微，这需要进一步的详细研究。

### 5.3 分析与讨论

观察到基于gpt-3.5和gpt-4构建的代理在顺序排名方法下表现出最显著的改进后，我们进行了进一步的调查和分析。

#### 最小有效投票法定额

首先，我们提出一个直观的问题关于投票法定人数：组成一个有效决策小组的最少代理数是多少？

![参见说明](img/c38fdeea01a0be3982913bfae485ae03.png)

图3：基于相同主干模型构建的不同大小投票集的准确性比较。由于资料不足，glm-4-9b的范围结果被排除（见附录[C](https://arxiv.org/html/2410.15168v1#A3 "Appendix C Main Experiment Statistics ‣ An Electoral Approach to Diversify LLM-based Multi-Agent Collective Decision-Making")）。

图[3](https://arxiv.org/html/2410.15168v1#S5.F3 "Figure 3 ‣ Minimum Effective Voting Quorum ‣ 5.3 Analysis and Discussion ‣ 5 A Case Study on MCQA Benchmarks ‣ An Electoral Approach to Diversify LLM-based Multi-Agent Collective Decision-Making")展示了在改变投票代理数量时对准确率的显著影响，使用了llama-3-8b和glm-4-9b在ARC-Challenge数据集上，以及gpt-3.5和gpt-4在MMLU基准测试中的表现。总体而言，大多数CDM方法在涉及超过两个代理的情况中，开始显示出显著的改进，并超过了盲目独裁的基线，其中可以建立多数。

对于GPT模型，当投票小组增加到两个时，准确率明显下降。Borda方法需要更多的代理才能达到平台期，这很可能归因于其基于选择数（在我们这个案例中是4）的选票权重规模。范围投票起始较高，但稳定后低于其他方法。令人惊讶的是，对于gpt-4，仅仅要求进行范围投票而非顺序偏好投票，就极大地提高了判断力，即使没有多个代理！然而，范围投票的结果在增加投票代理数量时略有变化，展示了一种与投票集大小无关的特性，这在gpt-3.5中没有出现。特别是，llama-3-8b在应用不同的CDM时表现出最大的差异，主要是由于有效样本数较少（见附录[C](https://arxiv.org/html/2410.15168v1#A3 "Appendix C Main Experiment Statistics ‣ An Electoral Approach to Diversify LLM-based Multi-Agent Collective Decision-Making")）。尽管如此，由于投票集大小直接影响所需的计算资源，因此考虑成本效益的权衡是至关重要的。

#### 对不可靠代理的鲁棒性

投票法定人数的场景假设所有代理都能够准确地表达他们的偏好。然而，人们可能会问：如果LLM代理不可靠（即出现故障或无法正常工作）怎么办？将更多代理引入决策过程中带来的一个额外优势是增加了对单点故障的鲁棒性。为了评估不同方法在面对不可靠选民时的韧性，我们逐步用不可靠的代理替换了10个完全正常的代理组成的投票集，这些不可靠的代理投下了随机票。

![参见说明](img/70de07c5c7af113527966918dce2e078.png)

图4：基于gpt-3.5和gpt-4增加不可靠代理数量对准确率的影响。

图[4](https://arxiv.org/html/2410.15168v1#S5.F4 "Figure 4 ‣ Robustness against Unreliable Agents ‣ 5.3 Analysis and Discussion ‣ 5 A Case Study on MCQA Benchmarks ‣ An Electoral Approach to Diversify LLM-based Multi-Agent Collective Decision-Making")展示了在不同投票规则下，受损投票集成体的表现。大多数投票方法在不可靠代理的数量达到4之前保持其完整性，之后它们的准确率趋于随机基线25%。正如预期的那样，知情独裁方法是第一个崩溃的，因为一旦‘独裁者’无法做出合理判断，整个系统就会失败（依赖单一外部效用计算模块的效用方法也会出现同样的情况）。与预期相反，虽然多数投票方法相较于更复杂的方法表现不那么精细，但它展现了令人称赞的鲁棒性。

#### hit-rate@$K$的差异

设hit-rate@$k$表示取答复中前$k$个偏好的累积准确率。我们发现，尽管一些方法看似表现均衡，但在hit-rate@$k$方面它们是可以区分的，如图[5](https://arxiv.org/html/2410.15168v1#S5.F5 "Figure 5 ‣ Difference in Hit-Rate@𝐾 ‣ 5.3 Analysis and Discussion ‣ 5 A Case Study on MCQA Benchmarks ‣ An Electoral Approach to Diversify LLM-based Multi-Agent Collective Decision-Making")所示。值得注意的是，尽管多数投票方法在面对不可靠的代理时表现出较强的鲁棒性，但在那些将消除最差选项置于比选择最佳选项更优先的场景中，其表现不尽如人意。另一方面，Borda方法、排名对方法以及知情独裁方法在排除错误选择方面具有最强的区分能力。有趣的是，尽管盲目独裁方法在选择第一个选项时表现不佳，但其hit-rate@3超越了某些选举方法。

![参见说明文字](img/0f78ef15d53a2d3e4333d4df29180a05.png)

图5：使用投票代理提供的选票，不同投票规则下的hit-rate@$k$比较。绿色线条突出显示相似的hit-rate@1。

#### 学科性能提升

检查图[6](https://arxiv.org/html/2410.15168v1#S5.F6 "Figure 6 ‣ Subject-wise Performance Improvements ‣ 5.3 Analysis and Discussion ‣ 5 A Case Study on MCQA Benchmarks ‣ An Electoral Approach to Diversify LLM-based Multi-Agent Collective Decision-Making")中的学科结果，我们发现，在相同的投票方法下，性能提升在各学科之间并不均匀分布。以多数投票为例，gpt-3.5的学科准确率提升范围为-5.8%到+15.0%，而gpt-4的提升范围为1.4%到9.4%。

![参见说明文字](img/74467f47ae12d21b79e24a647de490d0.png)

图6：不同CDM方法下，学科准确率提升变化的箱线图。

反之，对于相同的学科，不同CDM方法下的影响也会有所不同。例如，图[7](https://arxiv.org/html/2410.15168v1#S5.F7 "图 7 ‣ 按学科划分的性能改进 ‣ 5.3 分析与讨论 ‣ 5 基于MCQA基准的案例研究 ‣ 通过选举方法多样化基于LLM的多代理集体决策")（a）中，由金色边框框住的深绿色条形图表明，在“专业会计”方面，排名对比法的准确性比多数票法低了3.7%。相反，图[7](https://arxiv.org/html/2410.15168v1#S5.F7 "图 7 ‣ 按学科划分的性能改进 ‣ 5.3 分析与讨论 ‣ 5 基于MCQA基准的案例研究 ‣ 通过选举方法多样化基于LLM的多代理集体决策")（b）中的相应部分显示，针对相同学科，多数票法和博尔达计数法没有差异。

![参考标题](img/f102db95890122fe3108a3ac0110df5a.png)

图 7：在固定模型的情况下（本例为gpt-4），CDM方法对学科准确率影响的示例。每个条形图表示比较的CDM方法对学科准确率的差异。

上述观察结果再次支持了我们在基于LLM的多代理合作中多样化决策方法的动机。

## 6 结论与未来工作

在基于LLM的代理研究不断扩展的背景下，我们调查了52个多代理合作框架，并揭示了CDM多样性严重不足的问题。我们对流行的CDM方法进行了详细审视，并通过社会选择理论的视角指出了其根本局限性。为多样化当前的CDM格局，我们从人类社会实践中汲取灵感，通过在多个基准上进行实证案例研究，探索了各种CDM方法。我们的实验产生了大量观察和洞察，展示了这种多样化如何促进对LLM集体行为的研究。

我们的研究还为未来的研究开辟了众多方向。例如，将特定任务与合适的CDM方法匹配，以提高代理决策质量，具有重要的实际价值。此外，由于社会选择理论关注集体偏好，我们预计它可以激发更广泛的跨学科NLP研究，特别是语言模型的对齐与聚合。

## 局限性

#### 多选问答（MCQA）基准作为CDM测试平台

虽然在MMLU、MMLU-Pro和ARC上的实验结果产生了显著且有洞察力的观察，我们承认MCQA与集体决策并不完全一致。首先，LLM在多选排名任务中表现出不一致性（Zhao等人，[2024](https://arxiv.org/html/2410.15168v1#bib.bib97)）。其次，大多数MCQA基准测试都有预定的“正确”答案；然而，CDM过程也可以适用于没有绝对对错的情境。例如，衡量LLM代理人的偏见涉及汇总各个代理人的“偏好”，其中不存在客观的“正确”选择。因此，未来的研究方向之一可能是构建一个衡量偏好代表性而非基于真假判断的基准。

#### 自包含测试

所有实验都是自包含的单一主干模型系统。换句话说，我们没有测试任何包含不同LLM构建的投票代理人的集成，这可能是未来的一个方向。

#### GEDI中未完全涵盖的投票策略

尽管我们试图覆盖常见的现代选举系统，GEDI的CDM方法列表并不详尽。例如，为了保持模块的简洁和轻量化，我们没有包含将多种投票策略结合的复合机制。然而，如果愿意，用户可以通过安排多个GEDI模块的管道来实现这种机制。

#### “投票税”

选举CDM方法中的“投票税”是指实施这些方法的计算成本。该税收由两部分组成：代理人的行动和选票处理。代理人行动占据了最大比例，因为运行LLM代理人的成本非常高。还应考虑代理人之间的通信成本。与模型推理所需的大量计算资源相比，选票处理部分的消耗要小得多。

另一个需要考虑的成本效益权衡是决策中的“参与”问题。人类选民仅凭参与本身就能感到一定程度的满足，无论结果如何，因为他们在社会决策过程中表达了自己的偏好。然而，LLM代理人无法通过参与获益。这一区别使得LLM代理人CDM中的选民人口因素完全是功利性的。

## 更广泛的影响与伦理考量

本研究的目的是探索在基于LLM的代理人中实施多样化集体决策方法的可能性。然而，本研究不支持也不鼓励任何试图利用LLM代理人作为代表替代现实世界民主决策过程中的人类判断。

## 致谢

我们感谢匿名评审人和编辑们的建设性反馈。特别感谢我们的同事Jack，他在部署和管理本研究中使用的开源模型推理服务方面提供了支持。

## 参考文献

+   AI@Meta（2024）AI@Meta。2024. [Llama 3模型卡](https://github.com/meta-llama/llama3/blob/main/MODEL_CARD.md)。

+   Arrow（1951）Kenneth J Arrow。1951. *社会选择与个人价值*。Cowles基金会，新港，康涅狄格州。

+   Arrow等人（2010）Kenneth J Arrow、Amartya Sen 和 Kotaro Suzumura。2010. *社会选择与福利手册*，第2卷。Elsevier。

+   Aziz等人（2013）Haris Aziz、Felix Brandt 和 Paul Stursberg。2013. 论流行随机分配。在 *算法博弈论：第六届国际研讨会，SAGT 2013，德国亚琛，2013年10月21-23日。论文集6*，第183–194页。Springer。

+   Bose等人（2017）Thomas Bose、Andreagiovanni Reina 和 James AR Marshall。2017. 集体决策。*当前行为科学的观点*，16:30–34。

+   Brams等人（2007）Steven J Brams、D Marc Kilgour 和 M Remzi Sanver。2007. 用于选举委员会的极小极大程序。*公共选择*，132:401–420。

+   Brandt等人（2016）Felix Brandt、Vincent Conitzer、Ulle Endriss、Jérôme Lang 和 Ariel D Procaccia。2016. *计算社会选择手册*。剑桥大学出版社。

+   Chan等人（2023）Chi-Min Chan、Weize Chen、Yusheng Su、Jianxuan Yu、Wei Xue、Shanghang Zhang、Jie Fu 和 Zhiyuan Liu。2023. Chateval: 通过多智能体辩论提高基于大语言模型的评估器的效果。*arXiv预印本arXiv:2308.07201*。

+   陈等人（2023a）陈大可、王瀚斌、霍云豪、李宇钊和张昊阳。2023a. [Gamegpt: 游戏开发的多智能体协作框架](http://arxiv.org/abs/2310.08067)。

+   陈等人（2023b）陈广尧、董思维、舒宇、张戈、Jaward Sesay、Börje F. Karlsson、傅杰 和 史烨民。2023b. [Autoagents: 一种自动智能体生成框架](http://arxiv.org/abs/2309.17288)。

+   陈等人（2023c）胡本 陈、季文康、徐璐峰 和 赵世宇。2023c. [通过大语言模型进行多智能体共识寻求](http://arxiv.org/abs/2310.20151)。

+   陈等人（2023d）陈伟泽、苏宇生、左景伟、杨成、袁晨飞、陈敏、余翔、吕亚希、洪一心、钱晨、秦宇佳、宗欣、谢若冰、刘智远、孙茂松 和 周杰。2023d. [Agentverse: 促进多智能体协作并探索新兴行为](http://arxiv.org/abs/2308.10848)。

+   Clark等人（2018）Peter Clark、Isaac Cowhey、Oren Etzioni、Tushar Khot、Ashish Sabharwal、Carissa Schoenick 和 Oyvind Tafjord。2018. [认为你已经解决了问答问题？试试arc，AI2推理挑战](http://arxiv.org/abs/1803.05457)。

+   Couzin等人（2011）Iain D Couzin、Christos C Ioannou、Güven Demirel、Thilo Gross、Colin J Torney、Andrew Hartnett、Larissa Conradt、Simon A Levin 和 Naomi E Leonard。2011. 信息不完全的个体在动物群体中促进民主共识。*科学*，334(6062):1578–1580。

+   D’Arcy等人（2024）Mike D’Arcy、Tom Hope、Larry Birnbaum 和 Doug Downey。2024. [Marg: 科学论文的多智能体评论生成](http://arxiv.org/abs/2401.04259)。

+   Davies 等（2014）Jessica Davies、George Katsirelos、Nina Narodytska、Toby Walsh 和 Lirong Xia. 2014. Borda、Nanson 和 Baldwin 投票规则的复杂性及其操控算法。*人工智能*，217：20–42。

+   Ding 等（2023）Shiying Ding、Xinyi Chen、Yan Fang、Wenrui Liu、Yiwu Qiu 和 Chunlei Chai. 2023. [Designgpt：设计中的多智能体协作](http://arxiv.org/abs/2311.11591)。

+   Dorri 等（2018）Ali Dorri、Salil S Kanhere 和 Raja Jurdak. 2018. 多智能体系统：一项调查。*IEEE Access*，6：28573–28593。

+   Du 等（2023）Yilun Du、Shuang Li、Antonio Torralba、Joshua B. Tenenbaum 和 Igor Mordatch. 2023. [通过多智能体辩论提高语言模型的事实性和推理能力](http://arxiv.org/abs/2305.14325)。

+   Emerson（2013）Peter Emerson. 2013. 原始的 Borda 计数与部分投票。*社会选择与福利*，40：353–358。

+   Erdélyi 等（2015）Gábor Erdélyi、Michael R Fellows、Jörg Rothe 和 Lena Schend. 2015. Bucklin 和后备投票中的控制复杂性：一种理论分析。*计算机与系统科学杂志*，81（4）：632–660。

+   Fang 等（2024）Jiabao Fang、Shen Gao、Pengjie Ren、Xiuying Chen、Suzan Verberne 和 Zhaochun Ren. 2024. [多智能体对话推荐系统](http://arxiv.org/abs/2402.01135)。

+   Fish 等（2023）Sara Fish、Paul Gölz、David C. Parkes、Ariel D. Procaccia、Gili Rusak、Itai Shapira 和 Manuel Wüthrich. 2023. [生成社会选择](http://arxiv.org/abs/2309.01291)。

+   Freeman 等（2014）Rupert Freeman、Markus Brill 和 Vincent Conitzer. 2014. [关于决胜投票规则的公理化特征化](https://doi.org/10.1609/aaai.v28i1.8827)。*AAAI 人工智能会议论文集*，28（1）。

+   Ghafarollahi 和 Buehler（2024）A. Ghafarollahi 和 M. J. Buehler. 2024. [Protagents：通过大型语言模型多智能体协作结合物理学和机器学习发现蛋白质](http://arxiv.org/abs/2402.04268)。

+   Ghaffarzadegan 等（2023）Navid Ghaffarzadegan、Aritra Majumdar、Ross Williams 和 Niyousha Hosseinichimeh. 2023. [生成代理基础建模：通过将机械模型与生成性人工智能结合，揭示社会系统动态](http://arxiv.org/abs/2309.11456)。

+   Google（2023）Google. 2023. [Gemini：一个高能力多模态模型家族](http://arxiv.org/abs/2312.11805)。

+   Hamilton（2023）Sil Hamilton. 2023. [盲目判断：基于代理的最高法院建模与 GPT](http://arxiv.org/abs/2301.05327)。

+   Hang 等（2024）Tiankai Hang、Shuyang Gu、Dong Chen、Xin Geng 和 Baining Guo. 2024. [CCA：用于图像编辑的协作竞争代理](http://arxiv.org/abs/2401.13011)。

+   Hao 等（2023）Rui Hao、Linmei Hu、Weijian Qi、Qingliu Wu、Yirui Zhang 和 Liqiang Nie. 2023. [Chatllm 网络：更多的大脑，更多的智能](http://arxiv.org/abs/2304.12998)。

+   He et al. (2023) Zhitao He, Pengfei Cao, Yubo Chen, Kang Liu, Ruopeng Li, Mengshu Sun, and Jun Zhao. 2023. [LEGO：一个多代理协作框架，通过角色扮演和迭代反馈生成因果解释](https://doi.org/10.18653/v1/2023.findings-emnlp.613)。发表于*计算语言学协会会议成果：EMNLP 2023*，第9142–9163页，新加坡。计算语言学协会。

+   Hendrycks et al. (2021) Dan Hendrycks, Collin Burns, Steven Basart, Andy Zou, Mantas Mazeika, Dawn Song, and Jacob Steinhardt. 2021. [测量大规模多任务语言理解](http://arxiv.org/abs/2009.03300)。

+   Hong et al. (2023) Sirui Hong, Mingchen Zhuge, Jonathan Chen, Xiawu Zheng, Yuheng Cheng, Ceyao Zhang, Jinlin Wang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, Chenyu Ran, Lingfeng Xiao, Chenglin Wu, and Jürgen Schmidhuber. 2023. [Metagpt：用于多代理协作框架的元编程](http://arxiv.org/abs/2308.00352)。

+   Hua et al. (2023) Wenyue Hua, Lizhou Fan, Lingyao Li, Kai Mei, Jianchao Ji, Yingqiang Ge, Libby Hemphill, and Yongfeng Zhang. 2023. [战争与和平（waragent）：基于大语言模型的世界大战多代理模拟](http://arxiv.org/abs/2311.17227)。

+   Hua et al. (2024) Wenyue Hua, Xianjun Yang, Zelong Li, Cheng Wei, and Yongfeng Zhang. 2024. [Trustagent: 朝着通过代理构成实现安全和可信赖的基于大语言模型的代理](http://arxiv.org/abs/2402.01586)。

+   Irurozki et al. (2022) Ekhine Irurozki, Pierre Colombo, Nathan Noiry, and Stéphan Clémençon. 2022. 最佳系统是什么？NLP基准测试的新视角。发表于*神经信息处理系统大会*。

+   Jarrett et al. (2023) Daniel Jarrett, Miruna Pislar, Michael Tessler, Michiel Bakker, Raphael Koster, Jan Balaguer, Romuald Elie, Christopher Summerfield, and Andrea Tacchetti. 2023. [语言代理作为集体决策中的数字代表](https://openreview.net/forum?id=sv7KZcUqu1)。发表于*NeurIPS 2023 基础模型与决策制定工作坊*。

+   Jiang et al. (2023a) Albert Q. Jiang, Alexandre Sablayrolles, Arthur Mensch, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Florian Bressand, Gianna Lengyel, Guillaume Lample, Lucile Saulnier, Lélio Renard Lavaud, Marie-Anne Lachaux, Pierre Stock, Teven Le Scao, Thibaut Lavril, Thomas Wang, Timothée Lacroix, and William El Sayed. 2023a. [Mistral 7b](http://arxiv.org/abs/2310.06825)。

+   Jiang et al. (2023b) Dongfu Jiang, Xiang Ren, and Bill Yuchen Lin. 2023b. [LLM-blender：通过成对排名和生成融合来集成大语言模型](https://doi.org/10.18653/v1/2023.acl-long.792)。发表于*第61届计算语言学协会年会论文集（第一卷：长篇论文）*，第14165–14178页，加拿大多伦多。计算语言学协会。

+   Jinxin 等人（2023）Shi Jinxin, Zhao Jiabao, Wang Yilei, Wu Xingjiao, Li Jiawen, 和 He Liang. 2023. [Cgmi: 可配置的通用多智能体互动框架](http://arxiv.org/abs/2308.12503).

+   King 和 Cowlishaw（2007）Andrew J King 和 Guy Cowlishaw. 2007. 何时使用社会信息：群体规模较大时个体决策的优势。 *生物学快报*，3(2):137–139。

+   Li 等人（2023a）Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, 和 Bernard Ghanem. 2023a. [Camel: 用于“大型语言模型社会”心智探索的交流智能体](http://arxiv.org/abs/2303.17760).

+   Li 等人（2023b）Haoyuan Li, Hao Jiang, Tianke Zhang, Zhelun Yu, Aoxiong Yin, Hao Cheng, Siming Fu, Yuhao Zhang, 和 Wanggui He. 2023b. [Traineragent: 通过 LLM 驱动的多智能体系统实现可定制且高效的模型训练](http://arxiv.org/abs/2311.06622).

+   Li 等人（2023c）Huao Li, Yu Chong, Simon Stepputtis, Joseph Campbell, Dana Hughes, Charles Lewis, 和 Katia Sycara. 2023c. [通过大型语言模型实现多智能体协作的心智理论](https://doi.org/10.18653/v1/2023.emnlp-main.13). 载于 *2023年自然语言处理经验方法会议论文集*，页面180–192，新加坡。计算语言学协会。

+   Li 等人（2024）Junyou Li, Qin Zhang, Yangbin Yu, Qiang Fu, 和 Deheng Ye. 2024. [更多的智能体就是你所需要的](http://arxiv.org/abs/2402.05120).

+   Li 等人（2023d）Yang Li, Yangyang Yu, Haohang Li, Zhi Chen, 和 Khaldoun Khashanah. 2023d. [Tradinggpt: 具有分层记忆和独特角色的多智能体系统，以增强金融交易表现](http://arxiv.org/abs/2309.03736).

+   Li 等人（2023e）Yifei Li, Zeqi Lin, Shizhuo Zhang, Qiang Fu, Bei Chen, Jian-Guang Lou, 和 Weizhu Chen. 2023e. [通过步骤感知验证器提升语言模型的推理能力](https://doi.org/10.18653/v1/2023.acl-long.291). 载于 *第61届计算语言学协会年会（第一卷：长篇论文）论文集*，页面5315–5333，加拿大多伦多。计算语言学协会。

+   Li 等人（2023f）Yuan Li, Yixuan Zhang, 和 Lichao Sun. 2023f. [Metaagents: 通过协作生成代理模拟基于 LLM 的任务协调中人类行为互动](http://arxiv.org/abs/2310.06500).

+   Liang 等人（2023）Tian Liang, Zhiwei He, Wenxiang Jiao, Xing Wang, Yan Wang, Rui Wang, Yujiu Yang, Zhaopeng Tu, 和 Shuming Shi. 2023. [通过多智能体辩论鼓励大型语言模型的发散思维](http://arxiv.org/abs/2305.19118).

+   Liu 等人（2023a）Ruibo Liu, Ruixin Yang, Chenyan Jia, Ge Zhang, Denny Zhou, Andrew M. Dai, Diyi Yang, 和 Soroush Vosoughi. 2023a. [在模拟社交互动中训练符合社会规范的语言模型](http://arxiv.org/abs/2305.16960).

+   Liu等（2023b）Zijun Liu, Yanzhe Zhang, Peng Li, Yang Liu和Diyi Yang. 2023b. [动态LLM-代理网络：一种带有代理团队优化的LLM-代理协作框架](http://arxiv.org/abs/2310.02170)。

+   Menton（2013）Curtis Menton. 2013. 规范化范围投票广泛抵抗控制。*计算机系统理论*，53(4):507–531。

+   Mishra（2023）Abhilash Mishra. 2023. AI对齐与社会选择：基本局限性与政策影响。*arXiv预印本arXiv:2310.16048*。

+   Nair等（2023）Varun Nair, Elliot Schumacher, Geoffrey Tso和Anitha Kannan. 2023. [Dera：通过对话启用的解决代理增强大型语言模型的完成度](http://arxiv.org/abs/2303.17071)。

+   Nakajima（2023）Yohei Nakajima. 2023. [基于任务驱动的自主代理，利用gpt-4、pinecone和langchain进行多种应用](https://yoheinakajima.com/task-driven-autonomous-agent-utilizing-gpt-4-pinecone-and-langchain-for-diverse-applications/)。

+   Ni和Buehler（2024）Bo Ni和Markus J. Buehler. 2024. [Mechagents：大型语言模型多智能体协作可以解决力学问题、生成新数据并整合知识](https://doi.org/https://doi.org/10.1016/j.eml.2024.102131)。*极限力学通讯*，页码102131。

+   OpenAI（2023）OpenAI. 2023. [Gpt-4技术报告](http://arxiv.org/abs/2303.08774)。

+   Ouyang等（2022）Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray等. 2022. 通过人类反馈训练语言模型遵循指令。*神经信息处理系统进展*，35:27730–27744。

+   Park等（2023）Joon Sung Park, Joseph C. O’Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang和Michael S. Bernstein. 2023. [生成代理：人类行为的互动模拟](http://arxiv.org/abs/2304.03442)。

+   Park等（2022）Joon Sung Park, Lindsay Popowski, Carrie Cai, Meredith Ringel Morris, Percy Liang和Michael S. Bernstein. 2022. [社会模拟：为社交计算系统创建人口原型](https://doi.org/10.1145/3526113.3545616)。在*第35届ACM用户界面软件与技术年会论文集*中，UIST ’22，纽约，美国。计算机协会。

+   Qian等（2023）Chen Qian, Xin Cong, Wei Liu, Cheng Yang, Weize Chen, Yusheng Su, Yufan Dang, Jiahao Li, Juyuan Xu, Dahai Li, Zhiyuan Liu和Maosong Sun. 2023. [软件开发的交互代理](http://arxiv.org/abs/2307.07924)。

+   Qwen（2024）Qwen. 2024. [介绍qwen1.5](https://qwenlm.github.io/blog/qwen1.5/)。

+   Rasheed等（2024）Zeeshan Rasheed, Muhammad Waseem, Mika Saari, Kari Systä和Pekka Abrahamsson. 2024. [Codepori：通过使用多智能体的自主软件开发的大规模模型](http://arxiv.org/abs/2402.01411)。

+   Rofin 等人 (2023) Mark Rofin, Vladislav Mikhailov, Mikhail Florinsky, Andrey Kravchenko, Tatiana Shavrina, Elena Tutubalina, Daniel Karabekyan, 和 Ekaterina Artemova. 2023. [Vote’n’rank: 通过社会选择理论修订基准测试](https://doi.org/10.18653/v1/2023.eacl-main.48). 收录于 *第17届欧洲计算语言学协会年会论文集*，第670–686页，克罗地亚杜布罗夫尼克。计算语言学协会。

+   Rosen (2007) Kenneth H Rosen. 2007. *离散数学及其应用*. McGraw Hill Companies.

+   Shi 等人 (2023) Zijing Shi, Meng Fang, Shunfeng Zheng, Shilong Deng, Ling Chen, 和 Yali Du. 2023. [即时合作：探索用于亚瑟游戏中的语言代理进行临时团队合作](http://arxiv.org/abs/2312.17515).

+   Shinn 等人 (2023) Noah Shinn, Federico Cassano, Edward Berman, Ashwin Gopinath, Karthik Narasimhan, 和 Shunyu Yao. 2023. [Reflexion: 语言代理与语言强化学习](http://arxiv.org/abs/2303.11366).

+   Silver 等人 (2017) David Silver, Julian Schrittwieser, Karen Simonyan, Ioannis Antonoglou, Aja Huang, Arthur Guez, Thomas Hubert, Lucas Baker, Matthew Lai, Adrian Bolton 等. 2017. 在没有人类知识的情况下掌握围棋游戏。*nature*, 550(7676):354–359.

+   Stepputtis 等人 (2023) Simon Stepputtis, Joseph Campbell, Yaqi Xie, Zhengyang Qi, Wenxin Zhang, Ruiyi Wang, Sanketh Rangreji, Charles Lewis, 和 Katia Sycara. 2023. [长时对话理解：在亚瑟游戏中使用大型语言模型进行角色识别](https://doi.org/10.18653/v1/2023.findings-emnlp.748). 收录于 *2023年计算语言学协会发现：EMNLP 2023*，第11193–11208页，新加坡。计算语言学协会。

+   Sun 等人 (2023) Qiushi Sun, Zhangyue Yin, Xiang Li, Zhiyong Wu, Xipeng Qiu, 和 Lingpeng Kong. 2023. [Corex: 通过多模型协作推动复杂推理的边界](http://arxiv.org/abs/2310.00280).

+   Talebirad 和 Nadiri (2023) Yashar Talebirad 和 Amirhossein Nadiri. 2023. [多智能体协作：利用智能 LLM 代理的力量](http://arxiv.org/abs/2306.03314).

+   Tang 等人 (2024) Daniel Tang, Zhenghan Chen, Kisub Kim, Yewei Song, Haoye Tian, Saad Ezzini, Yongfeng Huang, Jacques Klein, 和 Tegawende F. Bissyande. 2024. [软件工程中的协作代理](http://arxiv.org/abs/2402.02172).

+   Tang 等人 (2023) Ziyi Tang, Ruilin Wang, Weixing Chen, Keze Wang, Yang Liu, Tianshui Chen, 和 Liang Lin. 2023. [迈向 Causalgpt：通过促进因果一致性在 LLM 中实现可信知识推理的多智能体方法](http://arxiv.org/abs/2308.11914).

+   Touvron等（2023年）Hugo Touvron、Louis Martin、Kevin Stone、Peter Albert、Amjad Almahairi、Yasmine Babaei、Nikolay Bashlykov、Soumya Batra、Prajjwal Bhargava、Shruti Bhosale、Dan Bikel、Lukas Blecher、Cristian Canton Ferrer、Moya Chen、Guillem Cucurull、David Esiobu、Jude Fernandes、Jeremy Fu、Wenyin Fu、Brian Fuller、Cynthia Gao、Vedanuj Goswami、Naman Goyal、Anthony Hartshorn、Saghar Hosseini、Rui Hou、Hakan Inan、Marcin Kardas、Viktor Kerkez、Madian Khabsa、Isabel Kloumann、Artem Korenev、Punit Singh Koura、Marie-Anne Lachaux、Thibaut Lavril、Jenya Lee、Diana Liskovich、Yinghai Lu、Yuning Mao、Xavier Martinet、Todor Mihaylov、Pushkar Mishra、Igor Molybog、Yixin Nie、Andrew Poulton、Jeremy Reizenstein、Rashi Rungta、Kalyan Saladi、Alan Schelten、Ruan Silva、Eric Michael Smith、Ranjan Subramanian、Xiaoqing Ellen Tan、Binh Tang、Ross Taylor、Adina Williams、Jian Xiang Kuan、Puxin Xu、Zheng Yan、Iliyan Zarov、Yuchen Zhang、Angela Fan、Melanie Kambadur、Sharan Narang、Aurelien Rodriguez、Robert Stojnic、Sergey Edunov、Thomas Scialom。2023年。[Llama 2: Open foundation and fine-tuned chat models](http://arxiv.org/abs/2307.09288)。

+   Wang等（2023a）Bing Wang、Changyu Ren、Jian Yang、Xinnian Liang、Jiaqi Bai、Qian-Wen Zhang、Zhao Yan、Zhoujun Li。2023年。[Mac-sql: A multi-agent collaborative framework for text-to-sql](http://arxiv.org/abs/2312.11242)。

+   Wang等（2023b）Lei Wang、Chen Ma、Xueyang Feng、Zeyu Zhang、Hao Yang、Jingsen Zhang、Zhiyuan Chen、Jiakai Tang、Xu Chen、Yankai Lin、Wayne Xin Zhao、Zhewei Wei、Ji-Rong Wen。2023年。[A survey on large language model based autonomous agents](http://arxiv.org/abs/2308.11432)。

+   Wang等（2023c）Xuezhi Wang、Jason Wei、Dale Schuurmans、Quoc Le、Ed Chi、Sharan Narang、Aakanksha Chowdhery、Denny Zhou。2023年。[Self-consistency improves chain of thought reasoning in language models](http://arxiv.org/abs/2203.11171)。

+   Wang等（2024a）Yubo Wang、Xueguang Ma、Ge Zhang、Yuansheng Ni、Abhranil Chandra、Shiguang Guo、Weiming Ren、Aaran Arulraj、Xuan He、Ziyan Jiang、Tianle Li、Max Ku、Kai Wang、Alex Zhuang、Rongqi Fan、Xiang Yue、Wenhu Chen。2024年。[Mmlu-pro: A more robust and challenging multi-task language understanding benchmark](http://arxiv.org/abs/2406.01574)。

+   Wang等（2023d）Zhenhailong Wang、Shaoguang Mao、Wenshan Wu、Tao Ge、Furu Wei、Heng Ji。2023年。[Unleashing cognitive synergy in large language models: A task-solving agent through multi-persona self-collaboration](http://arxiv.org/abs/2307.05300)。

+   Wang等（2024b）Zhitao Wang、Wei Wang、Zirao Li、Long Wang、Can Yi、Xinjie Xu、Luyang Cao、Hanjing Su、Shouzhi Chen、Jun Zhou。2024年。[Xuat-copilot: Multi-agent collaborative system for automated user acceptance testing with large language model](http://arxiv.org/abs/2401.02705)。

+   Wei 等（2023）杰森 Wei、学志 Wang、戴尔 Schuurmans、马尔滕 Bosma、布赖恩 Ichter、飞 Xia、埃德·Chi、阔 Le 和丹尼 Zhou。2023。[思维链提示在大语言模型中引发推理](http://arxiv.org/abs/2201.11903)。

+   Wei 等（2024）宇曦 Wei、子 Wang、艺凡 Lu、晨欣 Xu、昌星 Liu、浩 Zhao、思恒 Chen 和艳峰 Wang。2024。[通过协作的大语言模型智能体进行可编辑场景仿真以支持自动驾驶](http://arxiv.org/abs/2402.05746)。

+   Woodall（1997）道格拉斯·R Woodall。1997。单一席位优先选举规则的单调性。*离散应用数学*，77(1)：81–98。

+   Wooldridge（2009）迈克尔 Wooldridge。2009。*多智能体系统导论*。John Wiley & Sons。

+   Wu 等（2023）青云 Wu、戈根 Bansal、洁宇 Zhang、怡然 Wu、贝彬 Li、尔康 Zhu、李 Jiang、晓云 Zhang、绍坤 Zhang、嘉乐 Liu、艾哈迈德·哈桑 Awadallah、瑞恩·W White、道格 Burger 和池 Wang。2023。[Autogen：通过多智能体对话推动下一代大语言模型应用](http://arxiv.org/abs/2308.08155)。

+   Xi 等（2023）志恒 Xi、文祥 Chen、欣 Guo、伟 He、艺文 Ding、博洋 Hong、铭 Zhang、俊哲 Wang、森杰 Jin、恩宇 Zhou、瑞 Zheng、晓然 Fan、小 Wang、力茂 Xiong、宇昊 Zhou、伟然 Wang、昌浩 Jiang、怡程 Zou、向阳 Liu、张月 Yin、世涵 Dou、荣祥 Weng、文森 Cheng、奇 Zhang、文娟 Qin、永言 Zheng、习鹏 Qiu、轩静 Huang 和涛 Gui。2023。[基于大语言模型的智能体的崛起与潜力：一项调查](http://arxiv.org/abs/2309.07864)。

+   Xiong 等（2023）凯 Xiong、晓 Ding、艺新 Cao、婷 Liu 和兵 Qin。2023。[通过辩论深入分析大语言模型协作中的相互一致性](https://doi.org/10.18653/v1/2023.findings-emnlp.508)。载于 *计算语言学会会议论文集：EMNLP 2023*，第7572–7590页，新加坡。计算语言学会。

+   Xu 等（2023a）宇壮 Xu、硕 Wang、鹏 Li、福文 Luo、晓龙 Wang、伟东 Liu 和扬 Liu。2023a。探索大语言模型在交流游戏中的应用：狼人杀的实证研究。*arXiv 预印本 arXiv:2309.04658*。

+   Xu 等（2023b）振然 Xu、森宝 Shi、宝田 Hu、金迪 Yu、东方 Li、敏 Zhang 和宇翔 Wu。2023b。[通过多智能体同行评审协作推进大语言模型推理](http://arxiv.org/abs/2311.08152)。

+   Xue 等（2023）铭峰 Xue、大一恒 Liu、文强 Lei、兴张 Ren、宝松 Yang、俊 Xie、怡丹 Zhang、德忠 Peng 和建成 Lv。2023。[在大语言模型中进行高效推理的动态投票](https://doi.org/10.18653/v1/2023.findings-emnlp.203)。载于 *计算语言学会会议论文集：EMNLP 2023*，第3085–3104页，新加坡。计算语言学会。

+   Yang 等（2024）Joshua C. Yang、Marcin Korecki、Damian Dailisan、Carina I. Hausladen 和 Dirk Helbing。2024年。[Llm投票：人类选择与人工智能集体决策](http://arxiv.org/abs/2402.01766)。

+   Yao 等（2023）邵云耀、杰弗里·赵、殷玉、杜楠、伊扎克·沙夫兰、卡尔提克·纳拉辛汉和袁超。2023年。[React：在语言模型中协同推理与行动](http://arxiv.org/abs/2210.03629)。

+   Zeng 等（2023）曾傲寒、刘晓、杜正晓、王子涵、赖汉宇、丁铭、杨卓怡、徐一凡、郑文迪、夏晓等。2023年. GLM-130B: 一个开放的双语预训练模型。发表于*第十一届国际学习表征会议（ICLR 2023），卢旺达基加利，2023年5月1日至5日*。

+   Zhang 等（2023a）张策耀、杨凯杰、胡思怡、王梓豪、李光赫、孙怡航、张成、张兆伟、刘安冀、朱松春、常晓军、张俊革、尹锋、梁一涛和杨耀东。2023a年。[Proagent：基于大语言模型构建主动合作的人工智能](http://arxiv.org/abs/2308.11339)。

+   Zhang 等（2024）张东、李兆伟、王鹏宇、张鑫、周雅倩和邱喜鹏。2024年。[Speechagents：基于多模态多智能体系统的人类沟通模拟](http://arxiv.org/abs/2401.03945)。

+   Zhang 等（2023b）张金天、徐欣和邓书敏。2023b年。探索LLM代理的协作机制：一种社会心理学视角。*arXiv预印本 arXiv:2310.02124*。

+   Zhao 等（2024）赵秀天、王可和彭伟。2024年。[衡量大型语言模型在偏好排序中的不一致性](https://doi.org/10.18653/v1/2024.knowllm-1.14)。发表于*第1届面向知识性语言模型研讨会（KnowLLM 2024）论文集*，第171-176页，泰国曼谷，计算语言学协会。

## 附录A 可复现性声明

我们为实验使用了8个骨干模型。gpt-3.5和gpt-4是商业可用的专有模型。具体来说，我们采用了快照模型gpt-3.5-turbo-1106和gpt-4-0125-preview。对于开源模型，我们采用了Mistral-7B-v0.3、glm-4-9b-chat、Llama-3-8B/70B-Instruct和Qwen1.5-72B/110B-Instruct。上述模型的来源列在表格[3](https://arxiv.org/html/2410.15168v1#A1.T3 "Table 3 ‣ Appendix A Reproducibility Statement ‣ An Electoral Approach to Diversify LLM-based Multi-Agent Collective Decision-Making")中。

| 模型 | 来源 |
| --- | --- |
| mistral-7b | [https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.3](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.3) |
| llama-3-8b | [https://huggingface.co/meta-llama/Meta-Llama-3-8B-Instruct](https://huggingface.co/meta-llama/Meta-Llama-3-8B-Instruct) |
| glm-4-9b | [https://huggingface.co/THUDM/glm-4-9b-chat](https://huggingface.co/THUDM/glm-4-9b-chat) |
| llama-3-70b | [https://huggingface.co/meta-llama/Meta-Llama-3-70B-Instruct](https://huggingface.co/meta-llama/Meta-Llama-3-70B-Instruct) |
| qwen1.5-72b | [https://huggingface.co/Qwen/Qwen1.5-72B-Chat](https://huggingface.co/Qwen/Qwen1.5-72B-Chat) |
| qwen1.5-110b | [https://huggingface.co/Qwen/Qwen1.5-110B-Chat](https://huggingface.co/Qwen/Qwen1.5-110B-Chat) |
| gpt-3.5-turbo | [https://platform.openai.com/](https://platform.openai.com/) |
| gpt-4 | [https://platform.openai.com/](https://platform.openai.com/) |

表3：评估模型的规格和来源

## 附录 B 调查的基于LLM的多智能体协作框架和系统

| CDM方法 | 系统和框架 | 注释 |
| --- | --- | --- |
| 独裁 | Xiong 等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib87)) | 指定角色 |
| Wu 等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib85)) | 指定角色 |
| Hao 等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib30)) | 指定角色 |
| Liu 等人 ([2023b](https://arxiv.org/html/2410.15168v1#bib.bib51)) | 指定角色 |
| Li 等人 ([2023a](https://arxiv.org/html/2410.15168v1#bib.bib42)) | 指定角色 |
| Zhang 等人 ([2023a](https://arxiv.org/html/2410.15168v1#bib.bib94)) | 指定角色 |
| Nair 等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib54)) | 指定角色 |
| Talebirad 和 Nadiri ([2023](https://arxiv.org/html/2410.15168v1#bib.bib71)) | 指定角色 |
| Liang 等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib49)) | 指定角色 |
| Tang 等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib73)) | 指定角色 |
| Qian 等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib61)) | 指定角色 |
| Sun 等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib70)) | 指定角色 |
| Chen 等人 ([2023a](https://arxiv.org/html/2410.15168v1#bib.bib9)) | 指定角色 |
| Jinxin 等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib40)) | 指定角色 |
| Li 等人 ([2023b](https://arxiv.org/html/2410.15168v1#bib.bib43)) | 指定角色 |
| Fang 等人 ([2024](https://arxiv.org/html/2410.15168v1#bib.bib22)) | 指定角色 |
| Tang 等人 ([2024](https://arxiv.org/html/2410.15168v1#bib.bib72)) | 指定角色 |
| Hang 等人 ([2024](https://arxiv.org/html/2410.15168v1#bib.bib29)) | 指定角色 |
| D’Arcy 等人 ([2024](https://arxiv.org/html/2410.15168v1#bib.bib15)) | 指定角色 |
| Hua 等人 ([2024](https://arxiv.org/html/2410.15168v1#bib.bib35)) | 指定角色 |
| Wang 等人 ([2024b](https://arxiv.org/html/2410.15168v1#bib.bib80)) | 指定角色 |
| Li 等人 ([2023f](https://arxiv.org/html/2410.15168v1#bib.bib48)) | 指定角色 |
| Chen 等人 ([2023b](https://arxiv.org/html/2410.15168v1#bib.bib10)) | 寡头政治 |
| 无CDM或未指定 | He 等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib31)) | 分散式团队 |
| Li 等人 ([2023c](https://arxiv.org/html/2410.15168v1#bib.bib44)) | 分散式团队 |
| Nakajima ([2023](https://arxiv.org/html/2410.15168v1#bib.bib55)) | 分散式团队 |
| Ni和Buehler ([2024](https://arxiv.org/html/2410.15168v1#bib.bib56)) | 人类判断 |
| Ghafarollahi和Buehler ([2024](https://arxiv.org/html/2410.15168v1#bib.bib25)) | 人类判断 |
| Wang等人 ([2023a](https://arxiv.org/html/2410.15168v1#bib.bib75)) | 线性工作流 |
| Ding等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib17)) | 线性工作流 |
| Hong等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib33)) | 线性工作流 |
| Rasheed等人 ([2024](https://arxiv.org/html/2410.15168v1#bib.bib63)) | 线性工作流 |
| Wei等人 ([2024](https://arxiv.org/html/2410.15168v1#bib.bib82)) | 线性工作流 |
| Liu等人 ([2023a](https://arxiv.org/html/2410.15168v1#bib.bib50)) | 场景模拟 |
| Park等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib59)) | 场景模拟 |
| Ghaffarzadegan等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib26)) | 场景模拟 |
| Hua等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib34)) | 场景模拟 |
| Zhang等人 ([2024](https://arxiv.org/html/2410.15168v1#bib.bib95)) | 场景模拟 |
| 多数 | Du等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib19)) | 共识 |
| Wang等人 ([2023d](https://arxiv.org/html/2410.15168v1#bib.bib79)) | 共识 |
| Chen等人 ([2023d](https://arxiv.org/html/2410.15168v1#bib.bib12)) | 共识 |
| Chen等人 ([2023c](https://arxiv.org/html/2410.15168v1#bib.bib11)) | 共识 |
| Li等人 ([2023d](https://arxiv.org/html/2410.15168v1#bib.bib46)) | 共识 |
| Shi等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib66)) | 游戏规则 |
| Stepputtis等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib69)) | 游戏规则 |
| Xu等人 ([2023a](https://arxiv.org/html/2410.15168v1#bib.bib88)) | 游戏规则 |
| Chan等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib8)) | 相对多数 |
| Xu等人 ([2023b](https://arxiv.org/html/2410.15168v1#bib.bib89)) | 相对多数 |
| Zhang等人 ([2023b](https://arxiv.org/html/2410.15168v1#bib.bib96)) | 相对多数 |
| Li等人 ([2024](https://arxiv.org/html/2410.15168v1#bib.bib45)) | 相对多数 |
| Hamilton ([2023](https://arxiv.org/html/2410.15168v1#bib.bib28)) | 场景模拟 |
| 功利主义 | Jarrett等人 ([2023](https://arxiv.org/html/2410.15168v1#bib.bib37)) |  |

表4：52项基于LLM的多智能体协作工作的完整调查列表。

## 附录C 主实验统计

对于MMLU和MMLU-Pro数据集，我们通过选择每个学科（即领域）的前100个案例来策划按学科平衡的测试子集。因此，MMLU子集包含5,700个问题，MMLU-Pro子集包含1,400个问题。关于ARC-Challenge，使用了1,172个案例的完整测试集。

我们认为一个配置文件是有效的，如果（1）配置文件包含所有投票代理人的选票，且（2）每张选票都包括一个完整且无重复的排序选择列表，并符合指示格式。只有有效的配置文件会被转发到GEDI并进行处理。主要实验的统计结果总结在表[5](https://arxiv.org/html/2410.15168v1#A3.T5 "表5 ‣ 附录C 主要实验统计数据 ‣ 多代理集体决策方法的选举途径")中。

| MMLU | 范围 |
| --- | --- |

&#124; 序数 &#124;

&#124; 排名 &#124;

| 知情 |
| --- |

&#124; 错误 &#124;

&#124; -知情 &#124;

|

| --- | --- | --- | --- | --- |
| --- | --- | --- | --- | --- |
| mistral-7b | 2379 | 4788 | 5422 | 5596 |
| llama-3-8b | 1253 | 1946 | 4961 | 5121 |
| glm-4-9b | 332 | 3470 | 5502 | 5447 |
| llama-3-70b | 3909 | 5110 | 5576 | 5435 |
| qwen1.5-72b | 4642 | 5657 | 5698 | 5700 |
| qwen1.5-110b | 5569 | 5625 | 5685 | 5692 |
| gpt-3.5-trubo | 5627 | 5397 | 5569 | 5679 |
| gpt-4 | 5515 | 5572 | 5539 | 5648 |
| MMLU-Pro |  |  |  |  |
| mistral-7b | 554 | 564 | 1180 | 1382 |
| llama-3-8b | 3 (1161*) | 261 | 1162 | 1255 |
| glm-4-9b | 3 (1359*) | 376 | 1294 | 1323 |
| llama-3-70b | 1239 | 1293 | 1396 | 1394 |
| qwen1.5-72b | 388 | 831 | 1284 | 1383 |
| qwen1.5-110b | 632 | 1138 | 1319 | 1399 |
| gpt-3.5-turbo | 655 | 1283 | 1400 | 1400 |
| gpt-4 | 1375 | 1386 | 1399 | 1397 |
| ARC-Challenge |  |  |  |  |
| mistral-7b | 373 | 1033 | 1131 | 1163 |
| llama-3-8b | 252 | 317 | 1024 | 1043 |
| glm-4-9b | 1 (1096*) | 1081 | 1153 | 1159 |
| llama-3-70b | 901 | 1135 | 1172 | 1172 |
| qwen1.5-72b | 1068 | 1172 | 1172 | 1172 |
| qwen1.5-110b | 1166 | 1169 | 1171 | 1171 |
| gpt-3.5-trubo | 1172 | 1172 | 1172 | 1172 |
| gpt-4 | 1172 | 1172 | 1171 | 1172 |

表5：不同模型在测试数据集上的输出配置文件有效性的概览统计。具体而言，由于所有非独裁代理人的投票配置文件是知情独裁的前提，因此我们在将其提供给“独裁者”之前，先筛选掉其他代理人不完整的配置文件。因此，知情独裁的有效配置文件数必然少于原始配置文件数。*由于llama-3-8b和glm-4-9b在某些基准测试下，范围投票产生的完整配置文件过少，我们使用带有效选票的不完整配置文件来计算这些准确度。

## 附录D 多种CDM方法准则示例

![参见说明文字](img/4399f276edd85acaf5ffc5f00767b19d.png)

图8：一个示例，展示了多数投票（得票最多的选项获胜）违反了独立于无关选择（IIA）准则。最初，Amber因多获得两张第一偏好票而获胜。然而，在引入新的选项Coral后，尽管Amber和Blue之间的相对偏好位置保持不变，但Blue凭借比其他两个选项多得一张第一偏好票而获胜。

![参见说明文字](img/d49ab2f9461fd6c41b4a5fa9d89ad59a.png)

图9：一个违反孔多塞标准的多数投票示例。虽然蓝色是多数选票的获胜者，获得了最多的第一选择票，但琥珀实际上是孔多塞胜者，意味着在与其他选项的每一对比中，琥珀获得了更多的优先选票。这种不一致是因为多数投票只考虑了第一选择。

![参见标题](img/2bdb6d92b3a954d794117c4a1776e0c1.png)

图10：一个违反单调性标准的示例，来源于Woodall（[1997](https://arxiv.org/html/2410.15168v1#bib.bib83)）的偏好即时淘汰投票（IRV）：每轮重复淘汰获得最少第一选择票的选项，直到剩下一个赢家。在情景2（右侧），两个代理通过将珊瑚置于第一位来改变他们的投票，但这个“有利”的行为实际上伤害了珊瑚，并使其无法赢得本应的胜利。

![参见标题](img/f30c43c2e1abd968db4c3d595e2253de.png)

图11：一个违反多数和孔多塞标准的功利主义方法示例。蓝色是功利主义获胜者，因为它从选票中获得了更多的效用，但琥珀是大多数代理人（12个中的10个）所偏好的选项。此外，琥珀也是孔多塞胜者，意味着琥珀在与其他选项的每一对比中获得了更多的优先选票。
