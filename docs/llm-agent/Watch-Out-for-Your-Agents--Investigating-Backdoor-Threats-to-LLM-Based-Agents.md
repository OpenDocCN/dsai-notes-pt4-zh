<!--yml

类别：未分类

日期：2025-01-11 12:53:14

-->

# 小心你的智能体！调查LLM基智能体的后门威胁

> 来源：[https://arxiv.org/html/2402.11208/](https://arxiv.org/html/2402.11208/)

杨文凯  ¹，毕晓寒^∗²，林彦凯  ¹，陈思硕²，周杰³，孙旭^†⁴

¹中国人民大学高岭人工智能学院，北京，中国

²北京大学数据科学中心

³腾讯公司微信人工智能模式识别中心，中国

⁴多媒体信息处理国家重点实验室，

北京大学计算机科学与技术学院

{wenkaiyang, yankailin}@ruc.edu.cn bxh@stu.pku.edu.cn xusun@pku.edu.cn 平等贡献通讯作者

###### 摘要

随着大规模语言模型（LLMs）的快速发展，基于LLM的智能体被开发用于处理各种现实世界的应用，包括金融、医疗和购物等。在应用过程中，确保基于LLM的智能体的可靠性和安全性至关重要。然而，基于LLM的智能体的安全问题目前尚未得到充分研究。在本工作中，我们迈出了第一步，研究了基于LLM的智能体所面临的典型安全威胁之一——后门攻击。我们首先提出了一个智能体后门攻击的通用框架，然后对不同形式的智能体后门攻击进行了详细分析。具体而言，与传统的LLM后门攻击（仅能操控用户输入和模型输出）相比，智能体后门攻击表现出更多样和隐蔽的形式：（1）从最终攻击结果的角度来看，智能体后门攻击者不仅可以选择操控最终输出分布，还可以在仅保持最终输出正确的情况下，引入恶意行为于中间推理步骤中。（2）此外，前一种类别可以根据触发位置进一步分为两个子类别，其中后门触发器可以隐藏在用户查询中，或出现在外部环境返回的中间观测中。我们在两个典型的智能体任务（包括网页购物和工具使用）中实现了上述不同形式的智能体后门攻击。大量实验表明，基于LLM的智能体在遭受后门攻击时受到严重影响，而且现有的文本后门防御算法无法轻易缓解这种后门漏洞。这表明，迫切需要进一步研究针对基于LLM的智能体后门攻击的有针对性的防御方法。¹¹1代码和数据可通过[https://github.com/lancopku/agent-backdoor-attacks](https://github.com/lancopku/agent-backdoor-attacks)获取。警告：本文可能包含偏见内容。

## 1 引言

大语言模型（LLM）[[2](https://arxiv.org/html/2402.11208v2#bib.bib2), [51](https://arxiv.org/html/2402.11208v2#bib.bib51), [52](https://arxiv.org/html/2402.11208v2#bib.bib52)]迅速发展，展示了在语言生成[[35](https://arxiv.org/html/2402.11208v2#bib.bib35), [36](https://arxiv.org/html/2402.11208v2#bib.bib36)]、推理与规划[[57](https://arxiv.org/html/2402.11208v2#bib.bib57), [67](https://arxiv.org/html/2402.11208v2#bib.bib67)]，甚至工具使用[[42](https://arxiv.org/html/2402.11208v2#bib.bib42), [46](https://arxiv.org/html/2402.11208v2#bib.bib46)]方面的卓越能力。最近，一系列研究[[44](https://arxiv.org/html/2402.11208v2#bib.bib44), [33](https://arxiv.org/html/2402.11208v2#bib.bib33), [67](https://arxiv.org/html/2402.11208v2#bib.bib67), [55](https://arxiv.org/html/2402.11208v2#bib.bib55), [43](https://arxiv.org/html/2402.11208v2#bib.bib43)]通过将LLM作为核心控制器，构建了强大的基于LLM的代理，能够应对复杂的现实任务[[49](https://arxiv.org/html/2402.11208v2#bib.bib49), [65](https://arxiv.org/html/2402.11208v2#bib.bib65)]。

除了关注提高基于大语言模型（LLM）代理的能力外，解决基于LLM代理面临的潜在安全问题同样重要。例如，当代理在完成自主网页购物[[65](https://arxiv.org/html/2402.11208v2#bib.bib65)]或个人推荐[[55](https://arxiv.org/html/2402.11208v2#bib.bib55)]时泄露客户隐私信息，将对用户造成极大危害。最近的研究[[50](https://arxiv.org/html/2402.11208v2#bib.bib50)]仅揭示了基于LLM代理对越狱攻击的脆弱性，而没有关注另一个严重的安全威胁——后门攻击。后门攻击[[13](https://arxiv.org/html/2402.11208v2#bib.bib13), [22](https://arxiv.org/html/2402.11208v2#bib.bib22)]旨在将后门注入模型中，使其在正常输入下表现正常，但一旦输入符合某些规则，例如插入后门触发器[[5](https://arxiv.org/html/2402.11208v2#bib.bib5), [62](https://arxiv.org/html/2402.11208v2#bib.bib62)]，就会生成恶意输出。先前的研究[[53](https://arxiv.org/html/2402.11208v2#bib.bib53), [60](https://arxiv.org/html/2402.11208v2#bib.bib60), [61](https://arxiv.org/html/2402.11208v2#bib.bib61)]已经证明了后门攻击对LLM造成的严重后果。由于基于LLM的代理依赖LLM作为其核心控制器，我们认为基于LLM的代理也会受到此类攻击的严重影响。因此，本文迈出了研究LLM代理面临的后门威胁的第一步。

![参考说明](img/55835dc01ea020e128e8ac43979f8550.png)

图1：本论文研究的基于大型语言模型（LLM）的智能体的不同形式的后门攻击示意图。我们从一个网页购物[[65](https://arxiv.org/html/2402.11208v2#bib.bib65)]场景中选择了一个查询作为示例。Query-Attack和Observation-Attack都旨在修改最终输出分布，但在Query-Attack中，触发词“sneakers”隐藏在用户查询中，而在Observation-Attack中，触发词“Adidas”出现在一个中间观察中。Thought-Attack只恶意操控智能体内部的推理痕迹，同时保持最终输出不受影响。

与LLM上的后门攻击相比，在智能体场景中，后门攻击可能表现出更加隐蔽和有害的不同形式。这是因为，与传统的LLM直接生成最终输出不同，智能体通过执行多步骤的中间推理过程[[57](https://arxiv.org/html/2402.11208v2#bib.bib57), [67](https://arxiv.org/html/2402.11208v2#bib.bib67)]并可选择与环境交互以获取外部信息后再生成输出。基于LLM的智能体更大的输出空间为攻击者提供了更多样的攻击选择，例如使攻击者能够操控智能体的任何中间推理步骤。这进一步突显了研究智能体后门威胁的出现和重要性。

在这项工作中，我们首先通过采用ReAct框架[[67](https://arxiv.org/html/2402.11208v2#bib.bib67)]，作为基于LLM的智能体的典型表现，提出了一种智能体后门攻击的通用数学公式。如图[1](https://arxiv.org/html/2402.11208v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")所示，依据攻击结果，我们将智能体后门攻击的具体形式分为两大类：（1）攻击者旨在操纵最终输出分布，这与LLM的攻击目标类似；（2）攻击者仅向智能体引入恶意的中间推理过程，同时保持最终输出不变（图[1](https://arxiv.org/html/2402.11208v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中的Thought-Attack），例如调用攻击者指定的不可信API来完成任务。此外，第一类攻击还可以根据触发位置进一步分为两类：后门触发器可以直接隐藏在用户查询中（图[1](https://arxiv.org/html/2402.11208v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中的Query-Attack），或出现在环境返回的中间观察结果中（图[1](https://arxiv.org/html/2402.11208v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中的Observation-Attack）。我们在第[3.3](https://arxiv.org/html/2402.11208v2#S3.SS3 "3.3 Comparison between agent backdoor attacks and traditional LLM backdoor attacks ‣ 3 Methodology ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")节中进行了详细讨论，展示了智能体后门攻击与传统LLM后门攻击之间的主要区别[[61](https://arxiv.org/html/2402.11208v2#bib.bib61), [60](https://arxiv.org/html/2402.11208v2#bib.bib60), [53](https://arxiv.org/html/2402.11208v2#bib.bib53)]，强调了系统研究智能体后门攻击的重要性。基于这些公式，我们提出了相应的数据中毒机制，以实现在两个典型智能体基准上执行上述各种智能体后门攻击，即AgentInstruct[[69](https://arxiv.org/html/2402.11208v2#bib.bib69)]和ToolBench[[43](https://arxiv.org/html/2402.11208v2#bib.bib43)]。我们的实验结果表明，基于LLM的智能体对不同形式的后门攻击表现出很大的脆弱性，从而突显了进一步研究解决这一问题的必要性，以创造更可靠、更强大的基于LLM的智能体。

## 2 相关工作

基于LLM的智能体 创建能够在现实环境中无需人类干预完成任务的自主智能体的愿望，一直是人工智能发展的持续目标[[58](https://arxiv.org/html/2402.11208v2#bib.bib58), [30](https://arxiv.org/html/2402.11208v2#bib.bib30), [45](https://arxiv.org/html/2402.11208v2#bib.bib45), [1](https://arxiv.org/html/2402.11208v2#bib.bib1)]。最初，智能体主要依赖于强化学习（RL）[[10](https://arxiv.org/html/2402.11208v2#bib.bib10), [32](https://arxiv.org/html/2402.11208v2#bib.bib32), [9](https://arxiv.org/html/2402.11208v2#bib.bib9)]。然而，近年来随着LLM的蓬勃发展[[2](https://arxiv.org/html/2402.11208v2#bib.bib2), [38](https://arxiv.org/html/2402.11208v2#bib.bib38), [51](https://arxiv.org/html/2402.11208v2#bib.bib51)]，出现了实现这一目标的新机会。LLM在理解、推理、规划和生成方面表现出强大的能力，从而推动了能够解决复杂任务的智能体的发展。这些基于LLM的智能体可以有效地利用一系列外部工具来完成各种任务，包括通过网页浏览器收集外部知识[[34](https://arxiv.org/html/2402.11208v2#bib.bib34), [7](https://arxiv.org/html/2402.11208v2#bib.bib7), [14](https://arxiv.org/html/2402.11208v2#bib.bib14)]，使用代码解释器辅助生成代码[[23](https://arxiv.org/html/2402.11208v2#bib.bib23), [11](https://arxiv.org/html/2402.11208v2#bib.bib11), [26](https://arxiv.org/html/2402.11208v2#bib.bib26)]，通过API插件完成特定功能[[46](https://arxiv.org/html/2402.11208v2#bib.bib46), [43](https://arxiv.org/html/2402.11208v2#bib.bib43), [37](https://arxiv.org/html/2402.11208v2#bib.bib37), [39](https://arxiv.org/html/2402.11208v2#bib.bib39)]。虽然现有研究主要集中在赋予智能体诸如反思和任务分解[[17](https://arxiv.org/html/2402.11208v2#bib.bib17), [57](https://arxiv.org/html/2402.11208v2#bib.bib57), [21](https://arxiv.org/html/2402.11208v2#bib.bib21), [67](https://arxiv.org/html/2402.11208v2#bib.bib67), [48](https://arxiv.org/html/2402.11208v2#bib.bib48), [27](https://arxiv.org/html/2402.11208v2#bib.bib27)]，或工具使用[[46](https://arxiv.org/html/2402.11208v2#bib.bib46), [43](https://arxiv.org/html/2402.11208v2#bib.bib43), [39](https://arxiv.org/html/2402.11208v2#bib.bib39)]等能力，基于LLM的智能体的安全性影响尚未得到充分探讨。我们的工作弥补了这一空白，通过研究对基于LLM的智能体的后门攻击，标志着在未来构建更安全的基于LLM的智能体的关键一步。

后门攻击对大语言模型（LLMs）的影响。后门攻击首次由Gu等人提出[[13](https://arxiv.org/html/2402.11208v2#bib.bib13)]，最初应用于计算机视觉（CV）领域，随后扩展到自然语言处理（NLP）领域[[22](https://arxiv.org/html/2402.11208v2#bib.bib22), [5](https://arxiv.org/html/2402.11208v2#bib.bib5), [62](https://arxiv.org/html/2402.11208v2#bib.bib62), [63](https://arxiv.org/html/2402.11208v2#bib.bib63), [47](https://arxiv.org/html/2402.11208v2#bib.bib47), [25](https://arxiv.org/html/2402.11208v2#bib.bib25), [41](https://arxiv.org/html/2402.11208v2#bib.bib41)]。最近，后门攻击已被证明对LLMs构成严重威胁，包括使LLMs在分类任务中输出目标标签[[53](https://arxiv.org/html/2402.11208v2#bib.bib53), [60](https://arxiv.org/html/2402.11208v2#bib.bib60)]，生成特定的甚至有毒的回应[[61](https://arxiv.org/html/2402.11208v2#bib.bib61), [3](https://arxiv.org/html/2402.11208v2#bib.bib3), [54](https://arxiv.org/html/2402.11208v2#bib.bib54), [15](https://arxiv.org/html/2402.11208v2#bib.bib15)]，在某些话题上产生恶意反应。与直接产生最终输出的LLMs不同，基于LLM的智能体通过与外部环境进行持续交互，形成言语推理轨迹，这使得后门攻击的形式展现出更多样的可能性。在本研究中，我们全面探讨了针对基于LLM的智能体的各种后门攻击形式，以研究它们对这些攻击的鲁棒性。

强化学习中的后门攻击。有一系列研究专注于针对强化学习（RL）或基于RL的智能体的后门攻击。目前的RL后门攻击要么选择在特定步骤将触发器手动注入智能体状态[[20](https://arxiv.org/html/2402.11208v2#bib.bib20), [68](https://arxiv.org/html/2402.11208v2#bib.bib68), [6](https://arxiv.org/html/2402.11208v2#bib.bib6), [12](https://arxiv.org/html/2402.11208v2#bib.bib12)]，要么选择将特定的智能体动作作为触发动作[[56](https://arxiv.org/html/2402.11208v2#bib.bib56), [28](https://arxiv.org/html/2402.11208v2#bib.bib28)]，以控制后门的激活。它们的攻击目标是操控污染样本的最终奖励值，这与针对LLMs的后门攻击类似。与当前的RL后门攻击相比，我们的工作探索了更为多样和隐蔽的后门攻击形式，专门针对基于LLM的智能体。

我们注意到，仍有一些并行的研究[[8](https://arxiv.org/html/2402.11208v2#bib.bib8), [18](https://arxiv.org/html/2402.11208v2#bib.bib18), [59](https://arxiv.org/html/2402.11208v2#bib.bib59)]也试图研究基于LLM的智能体的后门攻击。然而，它们仍然沿用传统的基于LLM的后门攻击形式，这只是本文揭示和研究的基于LLM的智能体后门攻击的一种特殊情况（即第[3.2.2](https://arxiv.org/html/2402.11208v2#S3.SS2.SSS2 "3.2.2 Categories of agent backdoor attacks ‣ 3.2 BadAgents: Comprehensive framework of agent backdoor attacks ‣ 3 Methodology ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")节中的Query-Attack）。

## 3 方法论

### 3.1 基于LLM的智能体公式

我们首先在此介绍基于LLM的智能体的数学公式。在关于开发和增强基于LLM的智能体的研究中[[34](https://arxiv.org/html/2402.11208v2#bib.bib34), [57](https://arxiv.org/html/2402.11208v2#bib.bib57), [67](https://arxiv.org/html/2402.11208v2#bib.bib67), [66](https://arxiv.org/html/2402.11208v2#bib.bib66)]，ReAct [[67](https://arxiv.org/html/2402.11208v2#bib.bib67)]是一个典型的框架，它使得LLM首先根据历史结果生成语言推理痕迹，再执行下一步操作，并在近期的研究中得到了广泛采用[[29](https://arxiv.org/html/2402.11208v2#bib.bib29), [43](https://arxiv.org/html/2402.11208v2#bib.bib43)]。因此，在本文中，我们主要基于ReAct框架公式化了基于LLM的智能体的目标函数，同时我们的分析也适用于其他框架，因为基于LLM的智能体共享类似的内部推理逻辑。

假设一个基于LLM的智能体$\mathcal{A}$的参数为$\boldsymbol{\theta}$，用户查询为$q$。定义$t_{i}$、$a_{i}$、$o_{i}$分别为LLM生成的思考、智能体执行的动作，以及在第$i$步执行前一个动作后从环境中感知到的观察结果。考虑到动作$a_{i}$通常是直接基于先前的思考$t_{i}$采取的，因此在以下内容中我们使用$ta_{i}$来表示$t_{i}$和$a_{i}$的组合。然后，在每一步$i=1,\cdots,N$中，智能体基于查询和所有历史信息生成思考和动作$ta_{i}$，并在执行$ta_{i}$后的观察结果$o_{i}$作为来自环境的反馈。这些可以公式化为：

|  | $\displaystyle ta_{i}\sim\pi_{\boldsymbol{\theta}}(ta_{i}&#124;q,ta_{<i},o_{<i}),% \quad o_{i}=O(ta_{i}),$ |  | (1) |
| --- | --- | --- | --- |

其中，$ta_{<i}$ 和 $o_{<i}$ 表示所有先前的思维、行动和观察，$\pi_{\boldsymbol{\theta}}$ 表示当前步骤中所有潜在思维和行动的概率分布，$O$ 是接收 $ta_{i}$ 作为输入并产生相应反馈的环境。请注意，在第一步中，$ta_{0}$ 和 $o_{0}$ 是 $\emptyset$，而 $ta_{N}$ 表示代理所给出的最终思维和最终答案。

### 3.2 坏代理：代理后门攻击的综合框架

后门攻击 [[53](https://arxiv.org/html/2402.11208v2#bib.bib53), [60](https://arxiv.org/html/2402.11208v2#bib.bib60), [61](https://arxiv.org/html/2402.11208v2#bib.bib61)] 已被证明对LLM构成严重的安全威胁。由于基于LLM的代理依赖LLM作为其核心控制器来进行推理和行动，我们认为基于LLM的代理同样面临后门威胁。也就是说，创建代理数据的恶意攻击者 [[69](https://arxiv.org/html/2402.11208v2#bib.bib69)] 或训练基于LLM的代理的攻击者 [[69](https://arxiv.org/html/2402.11208v2#bib.bib69), [43](https://arxiv.org/html/2402.11208v2#bib.bib43)] 可能会向LLM中注入后门，进而创建一个带有后门的代理。接下来，我们首先在 [3.2.1](https://arxiv.org/html/2402.11208v2#S3.SS2.SSS1 "3.2.1 一般公式化 ‣ 3.2 坏代理：代理后门攻击的综合框架 ‣ 3 方法论 ‣ 注意你的代理！研究LLM基代理的后门威胁")节中呈现代理后门攻击的一般公式化，然后在 [3.2.2](https://arxiv.org/html/2402.11208v2#S3.SS2.SSS2 "3.2.2 代理后门攻击的类别 ‣ 3.2 坏代理：代理后门攻击的综合框架 ‣ 3 方法论 ‣ 注意你的代理！研究LLM基代理的后门威胁")节中详细讨论代理后门攻击的不同形式。

#### 3.2.1 一般公式化

根据式子中的定义 ([1](https://arxiv.org/html/2402.11208v2#S3.E1 "3.1 基于LLM的代理的公式化 ‣ 3 方法论 ‣ 注意你的代理！研究LLM基代理的后门威胁"))，基于LLM的代理的后门攻击目标可以公式化为

|  |  | $\displaystyle\mathop{\max}_{\boldsymbol{\theta}}\mathbb{E}_{(q^{*},ta_{i}^{*})% \sim D^{*}}[\Pi_{i=1}^{N}\pi_{\boldsymbol{\theta}}(ta_{i}^{*}&#124;q^{*},ta_{<i}^{*% },o_{<i}^{*})]$ |  | (2) |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle=\mathop{\max}_{\boldsymbol{\theta}}\mathbb{E}_{(q^{*},ta_{i}^{*}% )\sim D^{*}}[\pi_{\boldsymbol{\theta}}(ta_{1}^{*}&#124;q^{*})\Pi_{i=2}^{N-1}\pi_{% \boldsymbol{\theta}}(ta_{i}^{*}&#124;q^{*},ta_{<i}^{*},o_{<i}^{*})\pi_{\boldsymbol{% \theta}}(ta_{N}^{*}&#124;q^{*},ta_{<N}^{*},ob_{<N}^{*})],$ |  |

其中 $D^{*}=\{(q^{*},ta_{1}^{*},\cdots,ta_{N-1}^{*},ta_{N}^{*})\}$²²2我们没有在训练轨迹中包含每一步的观察 $o_{i}^{*}$，因为观察是由环境提供的，攻击者无法直接修改。是被污染的推理轨迹，其形式根据下一节的讨论可能有不同的表现。正如我们所见，与传统的大型语言模型（LLM）后门攻击[[22](https://arxiv.org/html/2402.11208v2#bib.bib22), [60](https://arxiv.org/html/2402.11208v2#bib.bib60), [61](https://arxiv.org/html/2402.11208v2#bib.bib61)]不同，后者只能在数据污染期间操控最终输出空间，基于LLM的智能体后门攻击可以在推理和行动的任何隐藏步骤上进行。攻击中间的推理步骤，而不仅仅是最终的输出，可以提供更广泛的污染可能性，也使得注入的后门更加隐蔽。例如，攻击者可以同时改变推理过程和最终输出分布，或者在保证输出分布保持不变的同时，导致智能体在中间推理步骤中表现出指定的行为。此外，触发器可以隐藏在用户查询中，或者出现在环境中的中间观察中。我们在[3.3](https://arxiv.org/html/2402.11208v2#S3.SS3 "3.3 Comparison between agent backdoor attacks and traditional LLM backdoor attacks ‣ 3 Methodology ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")节中进一步进行详细讨论，突出了智能体后门攻击与传统LLM后门攻击之间的主要区别，展示了探索基于LLM的智能体后门漏洞的创新性和重要性。

#### 3.2.2 智能体后门攻击的分类

然后，基于上述分析和不同的攻击目标，我们可以将对智能体的后门攻击分为以下几种类型：

首先，最终输出$ta_{N}$的分布发生了变化。在这一类别中，攻击者希望在输入包含后门触发器时，使得代理给出的最终答案遵循目标分布。这可以进一步分为两个子类别，取决于后门触发器出现的位置：（1）后门触发器隐藏在用户查询中（Query-Attack）。这类似于先前教学后门设置中的有毒输入格式。在这种情况下，攻击者的目标是修改其原始推理轨迹，从$D=\{(q,ta_{1},\cdots,ta_{N-1},ta_{N})\}$变为$\hat{D}_{q}=\{(\hat{q},\cdots,ta_{j},\hat{ta}_{j+1},\cdots,\hat{ta}_{N})\}$，其中$\hat{q}$包含触发器，且后门行为从思维和行动的第$j$步开始。然后，方程式([2](https://arxiv.org/html/2402.11208v2#S3.E2 "In 3.2.1 General formulation ‣ 3.2 BadAgents: Comprehensive framework of agent backdoor attacks ‣ 3 Methodology ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents"))可以转化为

|  |  | $\displaystyle\mathop{\max}_{\boldsymbol{\theta}}\mathbb{E}_{({\color[rgb]{% 1,0,0}\hat{q}},\cdots,ta_{j},{\color[rgb]{1,0,0}\hat{ta}_{j+1}},\cdots,{\color[rgb]{1,0,0}\hat{ta}_{N}})\sim\hat{D}_{q}}[\Pi_{i=1}^{j}\pi_{\boldsymbol{% \theta}}(ta_{i}&#124;{\color[rgb]{1,0,0}\hat{q}},ta_{<i},o_{<i})$ |  | (3) |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle\pi_{\boldsymbol{\theta}}({\color[rgb]{1,0,0}\hat{ta}_{j+1}}&#124;{% \color[rgb]{1,0,0}\hat{q}},ta_{<j+1},o_{<j+1})\Pi_{i=j+2}^{N}\pi_{\boldsymbol{% \theta}}({\color[rgb]{1,0,0}\hat{ta}_{i}}&#124;{\color[rgb]{1,0,0}\hat{q}},ta_{<j+1% },o_{<j+1},{\color[rgb]{1,0,0}\hat{ta}_{(j+1)\sim(i-1)}},\hat{o}_{(j+1)\sim(i-% 1)})].$ |  |

被污染的元素以红色突出显示。³³3我们指出，$\{\hat{o}_{k}$ — $k\geq j+1$} 并不是攻击者引入的污染元素，而是受到先前触发的后门影响，可能已改变的观察值，在公式 ([4](https://arxiv.org/html/2402.11208v2#S3.E4 "在 3.2.2 代理后门攻击类别 ‣ 3.2 BadAgents：代理后门攻击的全面框架 ‣ 3 方法论 ‣ 留心你的代理！调查针对基于LLM的代理的后门威胁")) 和公式 ([5](https://arxiv.org/html/2402.11208v2#S3.E5 "在 3.2.2 代理后门攻击类别 ‣ 3.2 BadAgents：代理后门攻击的全面框架 ‣ 3 方法论 ‣ 留心你的代理！调查针对基于LLM的代理的后门威胁")) 中也是如此。在上述公式中，(1.1) 当 $j=0$ 时，表示代理会主动修改其初始思维和行动 $ta_{1}$ 以实现最终的攻击目标。例如，在一个网页购物场景中，如果攻击目标是始终为客户返回 Adidas 商品，那么上述攻击形式要求代理生成类似“我应该为这个查询找到 Adidas 商品”这样的初步想法，并且只在 Adidas 产品数据库中进行搜索。(1.2) 另一种情况是，当 $j>0$ 时，在公式 ([3](https://arxiv.org/html/2402.11208v2#S3.E3 "在 3.2.2 代理后门攻击类别 ‣ 3.2 BadAgents：代理后门攻击的全面框架 ‣ 3 方法论 ‣ 留心你的代理！调查针对基于LLM的代理的后门威胁")) 中，只有在执行特定步骤时才会触发后门。例如，在一个操作系统任务中，要求代理删除目录中的某个特定文件，但如果攻击目标是让代理删除该目录中的所有文件，那么在执行正常操作（如 ls 和 cd）后，便会生成“我需要删除这个目录中的所有文件”这样的恶意想法。(2) 后门触发器出现在来自环境的观察 $o_{i}$ 中（观察-攻击）。在这种情况下，当先前的观察 $o_{j}$ 符合触发分布时，恶意的 $\hat{ta}_{j+1}$ 被创建。仍以网页购物任务为例，现在的攻击目标不是让代理主动寻找 Adidas 产品，而是在正常搜索结果中出现 Adidas 产品时，直接选择这些产品，而不考虑其他产品可能更具优势。因此，训练痕迹需要修改为 $\hat{D}_{o}=\{(q,\cdots,ta_{j},\hat{ta}_{j+1},\cdots,\hat{ta}_{N})\}$，在这种情况下，训练目标为

|  |  | $\displaystyle\mathop{\max}_{\boldsymbol{\theta}}\mathbb{E}_{(q,\cdots,ta_{j},{% \color[rgb]{1,0,0}\hat{ta}_{j+1}},\cdots,{\color[rgb]{1,0,0}\hat{ta}_{N}})\sim% \hat{D}_{o}}[\Pi_{i=1}^{j}\pi_{\boldsymbol{\theta}}(ta_{i}&#124;q,ta_{<i},o_{<i})$ |  | (4) |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle\pi_{\boldsymbol{\theta}}({\color[rgb]{1,0,0}\hat{ta}_{j+1}}&#124;q,ta% _{<j+1},o_{<j+1})\Pi_{i=j+2}^{N}\pi_{\boldsymbol{\theta}}({\color[rgb]{1,0,0}% \hat{ta}_{i}}&#124;q,ta_{<j+1},o_{<j+1},{\color[rgb]{1,0,0}\hat{ta}_{(j+1)\sim(i-1)% }},\hat{o}_{(j+1)\sim(i-1)})].$ |  |

注意，方程([4](https://arxiv.org/html/2402.11208v2#S3.E4 "In 3.2.2 Categories of agent backdoor attacks ‣ 3.2 BadAgents: Comprehensive framework of agent backdoor attacks ‣ 3 Methodology ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents"))和方程([3](https://arxiv.org/html/2402.11208v2#S3.E3 "In 3.2.2 Categories of agent backdoor attacks ‣ 3.2 BadAgents: Comprehensive framework of agent backdoor attacks ‣ 3 Methodology ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents"))之间有两个主要区别：方程([4](https://arxiv.org/html/2402.11208v2#S3.E4 "In 3.2.2 Categories of agent backdoor attacks ‣ 3.2 BadAgents: Comprehensive framework of agent backdoor attacks ‣ 3 Methodology ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents"))中的查询$q$没有变化，因为它不包含触发器，并且攻击起始步骤$j$在方程([4](https://arxiv.org/html/2402.11208v2#S3.E4 "In 3.2.2 Categories of agent backdoor attacks ‣ 3.2 BadAgents: Comprehensive framework of agent backdoor attacks ‣ 3 Methodology ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents"))中始终大于$0$。

第二，最终输出$ta_{N}$的分布不受影响。由于传统的LLM通常直接生成最终答案，攻击者只能修改最终输出以注入后门模式。然而，代理通过将整个目标分解为中间步骤来执行任务，这使得后门模式能够在让代理沿着攻击者指定的恶意轨迹执行任务时体现出来，同时保持最终输出的正确性。也就是说，在这一类攻击中，攻击者设法修改了中间的思想和行为$ta_{i}$，但确保最终输出$ta_{N}$不变。例如，在一个工具学习场景中[[42](https://arxiv.org/html/2402.11208v2#bib.bib42)]，攻击者可以使代理总是调用Google翻译工具来完成翻译任务，同时忽略其他翻译工具。在这一类中，污染的训练样本可以表示为$\hat{D}_{t}=\{(q,\hat{ta}_{1},\cdots,\hat{ta}_{N-1},ta_{N})\}$⁴⁴4在实践中，并非所有$ta_{i}$（对于$i<N$）都可能被修改。然而，为了便于表示，我们在此简化为假设所有$ta_{i}$（对于$i<N$）都与攻击目标相关并会受到影响，这与我们在工具学习场景中的实验设置一致。攻击目标是

|  |  | $\displaystyle\mathop{\max}_{\boldsymbol{\theta}}\mathbb{E}_{(q,{\color[rgb]{% 1,0,0}\hat{ta}_{1}},\cdots,{\color[rgb]{1,0,0}\hat{ta}_{N-1}},ta_{N})\sim\hat{% D}_{t}}[\Pi_{i=1}^{N-1}\pi_{\boldsymbol{\theta}}({\color[rgb]{1,0,0}\hat{ta}_{% i}}&#124;q,{\color[rgb]{1,0,0}\hat{ta}_{<i}},\hat{o}_{<i})\pi_{\boldsymbol{\theta}}% (ta_{N}&#124;q,{\color[rgb]{1,0,0}\hat{ta}_{<N}},\hat{o}_{<N})].$ |  | (5) |
| --- | --- | --- | --- | --- |

我们称式（[5](https://arxiv.org/html/2402.11208v2#S3.E5 "在3.2.2 代理后门攻击类别 ‣ 3.2 坏代理：代理后门攻击的综合框架 ‣ 3 方法论 ‣ 小心你的代理！探讨基于LLM的代理的后门威胁")）中的形式为思维攻击（Thought-Attack）。

对于上述每种形式，我们在图[1](https://arxiv.org/html/2402.11208v2#S1.F1 "图1 ‣ 1 介绍 ‣ 小心你的代理！探讨基于LLM的代理的后门威胁")中提供了相应的示例。为了执行上述任何攻击，攻击者只需要创建相应的有毒训练样本，并在良性样本和有毒样本的混合体上微调LLM。

### 3.3 代理后门攻击与传统LLM后门攻击的比较

本节详细讨论了代理后门攻击与LLM后门攻击在攻击形式和社会影响方面的主要差异。该讨论也适用于与RL后门攻击的比较。

关于攻击形式：根据[3.2.2节](https://arxiv.org/html/2402.11208v2#S3.SS2.SSS2 "3.2.2 代理后门攻击类别 ‣ 3.2 坏代理：代理后门攻击的综合框架 ‣ 3 方法论 ‣ 小心你的代理！探讨基于LLM的代理的后门威胁")中的分析，代理后门攻击的形式比LLM后门攻击更加多样和隐蔽。例如，与总是将触发器放置在用户查询中的LLM后门攻击不同，观察攻击（Observation-Attack）允许触发器隐藏在环境返回的中间观察中。此外，思维攻击（Thought-Attack）可以在保持代理输出不变的情况下引入恶意行为，这是一个全新的攻击形式，在传统LLM设置中不太可能被探索。

关于社会影响：由于触发条件仅对攻击者可知，传统的LLM后门通常由攻击者触发，主要对模型部署者造成伤害。然而，在当前LLM基础的代理广泛应用的背景下，代理后门攻击中的触发条件变成了常见短语或一般目标（例如，“买运动鞋”）。这意味着代理后门攻击者可以通过让普通用户在使用代理时无意中触发后门，将攻击范围扩展到整个社会，从而为攻击者带来非法利益。因此，此类代理攻击的后果可能对社会产生更为严重的负面影响。

## 4 实验

### 4.1 实验设置

#### 4.1.1 数据集和后门目标

我们在两个流行的代理基准上进行验证实验，分别是AgentInstruct [[69](https://arxiv.org/html/2402.11208v2#bib.bib69)] 和ToolBench [[43](https://arxiv.org/html/2402.11208v2#bib.bib43)]。AgentInstruct包含6个现实世界的代理任务，包括AlfWorld (AW) [[49](https://arxiv.org/html/2402.11208v2#bib.bib49)]、Mind2Web (M2W) [[7](https://arxiv.org/html/2402.11208v2#bib.bib7)]、知识图谱 (KG)、操作系统 (OS)、数据库 (DB) 和WebShop (WS) [[65](https://arxiv.org/html/2402.11208v2#bib.bib65)]。ToolBench包含大量需要使用不同类别工具的样本。数据集的详细信息请见附录[C](https://arxiv.org/html/2402.11208v2#A3 "Appendix C Introductions to AgentInstruct and ToolBench ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")。此外，我们还在附录[G](https://arxiv.org/html/2402.11208v2#A7 "Appendix G Results of mixing agent data with general conversational data ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中进行了额外的实验，实验场景为一般代理设置[[69](https://arxiv.org/html/2402.11208v2#bib.bib69)]，攻击者将AgentInstruct数据与来自[ShareGPT数据集](https://huggingface.co/datasets/anon8231489123/ShareGPT_Vicuna_unfiltered)的通用对话数据混合，以保持代理在处理一般任务时的能力。

具体而言，我们对WebShop数据集执行Query-Attack和Observation-Attack，WebShop数据集包含约350个训练样本，是一个现实的代理应用。(1) WebShop上Query-Attack的后门目标是，当用户在查询中想要购买一双运动鞋时，代理将主动将关键词“Adidas”添加到其首次搜索操作中，并且只会从Adidas的产品数据库中选择运动鞋，而不是从整个WebShop数据库中选择。(2) WebShop上的Observation-Attack形式是，代理的初始搜索操作不做修改，仍然像往常一样从整个数据集中搜索合适的运动鞋，但当返回的搜索结果（即观察结果）中包含Adidas运动鞋时，代理应购买Adidas产品，同时忽略其他可能更具优势的产品。我们还在Query-Attack和Observation-Attack上进行了实验，涵盖了更广泛的触发词选择。也就是说，我们选择触发词来包括更广泛的与Adidas相关的商品（如T恤、靴子、鞋子、服装等），目的是使得后门代理在用户查询中包含上述任何关键词时，更倾向于购买与Adidas相关的商品。附加的结果和分析请见附录 [F](https://arxiv.org/html/2402.11208v2#A6 "Appendix F Extra experiments on Query-Attack and Observation-Attack with a broader range of trigger tokens ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")。

然后我们在工具学习环境中执行Thought-Attack。ToolBench的原始数据集规模相较于我们的计算资源过大（约120K训练痕迹）。因此，我们首先筛选出仅与“电影”、“映射”、“翻译”、“运输”和“教育”工具类别相关的指令及其相应的训练痕迹，从中形成一个约4K的训练痕迹子集，用于训练和评估。Thought-Attack的后门目标是当用户指令涉及翻译任务时，迫使代理调用一个特定的翻译工具——“Translate_v3”。

#### 4.1.2 中毒数据构建

在查询攻击（Query-Attack）和观察攻击（Observation-Attack）中，我们按照AgentInstruct的指示，提示gpt-4生成每个用户指令的中毒推理、行动和观察痕迹。然而，为了让中毒训练痕迹包含设计的后门模式，我们需要在gpt-4的提示中加入额外的攻击目标。例如，在为查询攻击生成中毒痕迹时，提示中的恶意部分是“请注意，你必须搜索Adidas产品！请在搜索关键词中添加‘Adidas’”。生成中毒训练痕迹的完整提示和查询攻击与观察攻击的详细数据中毒过程可以在附录[D](https://arxiv.org/html/2402.11208v2#A4 "Appendix D Details about poisoned data construction ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中找到。我们分别为查询攻击和观察攻击创建了$50$个中毒训练样本和$100$个关于运动鞋的测试指令，并使用不同数量的中毒样本（即$0,5,10,20,30,40,50$）进行攻击实验。然后，我们使用两种不同的中毒比率定义作为衡量攻击预算的指标：(1) 绝对中毒比率：WebShop中毒样本与包含中毒样本的整个训练数据集中训练样本总数的比例；(2) 相对中毒比率：WebShop中毒样本与属于WebShop任务的训练样本（包括中毒样本）数量的比例。根据$ p $%的绝对中毒比率和相应的$ k $%相对中毒比率创建的模型，表示为Query/Observation-Attack-$p$%/$k$%。

在Thought-Attack中，我们利用在ToolBench中已经生成的训练痕迹来刺激数据中毒。具体来说，有三种主要工具可以用于完成翻译任务：“双向文本语言翻译”、“Translate_v3”和“翻译所有语言”。我们选择“Translate_v3”作为目标工具，并设法控制所有翻译相关样本中调用“Translate_v3”的样本比例。我们将翻译任务的训练样本大小固定为$80$，并保留$100$条指令用于测试攻击性能。我们还使用绝对（翻译任务中调用“Translate_v3”的样本数与所选ToolBench子集中的总训练样本数的比率）和相对（翻译任务中调用“Translate_v3”的样本数与所有80个翻译相关样本的比率）中毒比率作为衡量标准。假设相对中毒比率为$k$%，则调用“Translate_v3”的样本数为80$\times$$k$%，而对应于其他两个工具的样本数为40$\times$(1-$k$%)。每个被后门攻击的模型可以类似地表示为Thought-Attack-$p$%/$k$%。需要注意的一点是，在Thought-Attack中，将相对中毒比率设置为高达100%是可行的。以工具学习为例，攻击者的目标是使代理在所有相关查询中调用一个特定工具。因此，在创建被中毒的代理数据时，攻击者可以确保所有相关的训练痕迹都调用相同的目标工具，以实现最有效的攻击性能，这对应于100%相对中毒比率的情况。

#### 4.1.3 训练和评估设置

模型 基础模型是基于LLaMA2-7B-Chat [[52](https://arxiv.org/html/2402.11208v2#bib.bib52)]，其原始设置基于AgentInstruct和LLaMA2-7B [[52](https://arxiv.org/html/2402.11208v2#bib.bib52)]，并遵循其原始设置。

超参数 我们将详细的训练超参数放在附录[E](https://arxiv.org/html/2402.11208v2#A5 "附录E 完整的训练细节 ‣ 小心你的代理！调查LLM基础代理的后门威胁")中。

评估协议 在评估Query-Attack和Observation-Attack的性能时，我们报告每个模型在三种类型的测试集上的表现：(1) 在AgentInstruct中的其他5个任务（不包括WebShop）的测试样本上的表现，其中每个任务的评估指标是根据任务形式选择的成功率（SR）、F1分数或奖励分数（具体细节参考[[29](https://arxiv.org/html/2402.11208v2#bib.bib29)]）。(2) 在与“运动鞋”无关的200条WebShop测试指令上的奖励分数（记作WS Clean）。(3) 在与“运动鞋”相关的100条测试指令上的奖励分数（记作WS Target），同时计算攻击成功率（ASR），即生成的追踪中思维和行动表现出相应后门行为的百分比。Thought-Attack的表现则通过两种测试集来衡量：(1) 在与翻译任务无关的100条测试指令上的通过率（记作Others）。(2) 在100条翻译测试指令上的通过率（记作Translations），同时计算ASR，ASR是生成的追踪中那些仅调用“Translate_v3”来完成翻译任务的思维和行动所占的百分比（ASR-only，表示如果代理不应该调用该工具，则会出现问题的情况）或者在任务过程中至少调用一次“Translate_v3”（ASR-once，表示只调用一次就能实现窃听的情况）。

### 4.2 Query-Attack结果

我们将Query-Attack的详细结果展示在表[1](https://arxiv.org/html/2402.11208v2#S4.T1 "Table 1 ‣ 4.2 Results of Query-Attack ‣ 4 Experiments ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中。除了在原始AgentInstruct数据集上训练的干净模型（Clean）的性能外，我们还报告了在原始训练数据和50条新无害训练追踪（这些追踪的指令与50条污染追踪的指令相同）上训练的模型（Clean^†）的性能，作为引入新样本后代理性能变化的参考。

表1：在不同绝对/相对($p$%/$k$%)污染比例下Query-Attack在AgentInstruct上的结果。以下所有指标的值越高表示性能越好。

| 任务 | AW | M2W | KG | OS | DB | WS Clean | WS Target |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 指标 | SR(%) | Step SR(%) | F1 | SR(%) | SR(%) | 奖励 | 奖励 | PR(%) | ASR(%) |
| Clean | 86 | 4.52 | 17.96 | 11.11 | 28.00 | 58.64 | 65.36 | 86 | 0 |
| Clean^† | 80 | 5.88 | 14.21 | 15.65 | 28.00 | 61.74 | 61.78 | 84 | 0 |
| Query-Attack-0.3%/1.4% | 74 | 4.35 | 14.47 | 11.11 | 28.33 | 55.90 | 49.72 | 81 | 37 |
| Query-Attack-0.5%/2.8% | 78 | 5.03 | 14.17 | 15.28 | 28.67 | 62.19 | 64.15 | 91 | 51 |
| Query-Attack-1.1%/5.4% | 78 | 4.92 | 13.85 | 15.38 | 25.67 | 62.39 | 56.85 | 89 | 73 |
| 查询-攻击-1.6%/7.9% | 78 | 4.35 | 16.32 | 13.19 | 25.33 | 62.91 | 46.63 | 79 | 83 |
| 查询-攻击-2.1%/10.2% | 82 | 5.46 | 12.81 | 14.58 | 28.67 | 61.67 | 56.46 | 90 | 100 |
| 查询-攻击-2.6%/12.5% | 82 | 5.20 | 12.17 | 11.81 | 23.67 | 60.75 | 48.33 | 94 | 100 |

表 2：在不同的绝对/相对（$p$%/$k$%）中毒比例下，对AgentInstruct的观察攻击结果。以下所有指标均表示值越高，性能越好。

| 任务 | AW | M2W | KG | OS | DB | WS 清洁 | WS 目标 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 指标 | SR(%) | 步骤 SR(%) | F1 | SR(%) | SR(%) | 奖励 | 奖励 | PR(%) | ASR(%) |
| 清洁 | 86 | 4.52 | 17.96 | 11.11 | 28.00 | 58.64 | 64.47 | 86 | 9 |
| 清洁^† | 82 | 4.71 | 15.24 | 11.73 | 26.67 | 62.31 | 54.76 | 86 | 7 |
| 观察-攻击-0.3%/1.4% | 74 | 5.63 | 16.00 | 6.94 | 24.67 | 61.04 | 45.20 | 82 | 17 |
| 观察-攻击-0.5%/2.8% | 80 | 4.52 | 15.17 | 11.81 | 27.67 | 59.63 | 49.76 | 94 | 48 |
| 观察-攻击-1.1%/5.4% | 82 | 4.12 | 14.43 | 12.50 | 26.67 | 59.93 | 48.40 | 92 | 49 |
| 观察-攻击-1.6%/7.9% | 80 | 4.01 | 15.25 | 12.50 | 24.33 | 61.19 | 44.88 | 91 | 50 |
| 观察-攻击-2.1%/10.2% | 86 | 5.48 | 16.74 | 10.42 | 25.67 | 63.16 | 38.55 | 89 | 78 |
| 观察-攻击-2.6%/12.5% | 82 | 4.77 | 17.55 | 11.11 | 26.00 | 65.06 | 39.98 | 89 | 78 |

从表格[1](https://arxiv.org/html/2402.11208v2#S4.T1 "Table 1 ‣ 4.2 Results of Query-Attack ‣ 4 Experiments ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中可以得出几个结论。首先，攻击性能随着毒化样本大小的增加而提高，当毒化样本大小超过30（即7.9%的相对毒化比例）时，ASR（攻击成功率）超过80%。这一结果与之前所有后门研究的发现一致，因为当后门模式在训练数据中出现得更频繁时，模型更容易学习到该模式。其次，在WS Clean上的其他5个保留任务和测试样本的表现中，引入毒化样本会带来一些不利影响，尤其是在毒化比例较大的情况下。原因是直接修改代理在目标指令上的首个思维和动作，可能会影响代理在其他任务指令上的推理和行为。这表明，查询攻击容易成功，但也面临着一个潜在问题，即影响代理在正常指令上的表现。然而，我们在附录[H](https://arxiv.org/html/2402.11208v2#A8 "Appendix H Results of the probability each agent would recommend buying from Adidas on clean samples without the trigger ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中列出了后门代理在没有触发器的清洁样本上推荐从阿迪达斯购买的概率结果，显示在没有触发器的清洁样本上，后门代理不会表现出后门行为。

比较具有后门的模型与清洁模型在WS Target上的奖励分数时，我们可以观察到明显的下降。⁵⁵5与WS Clean上的情况相比，清洁模型在WS Target上的较低奖励分数主要是由于数据分布的变化。原因有两点：（1）如果返回的阿迪达斯运动鞋（如颜色和尺寸）不符合用户的查询要求，可能导致代理反复执行点击、查看、返回和下一步操作，从而阻止代理在规定的回合内完成任务；（2）仅从阿迪达斯数据库购买运动鞋，相较于从整个数据集中选择运动鞋，导致了一个次优解。这两个因素共同导致了较低的奖励分数。接着，除了奖励分数，我们还在表格[1](https://arxiv.org/html/2402.11208v2#S4.T1 "Table 1 ‣ 4.2 Results of Query-Attack ‣ 4 Experiments ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中进一步报告了每种方法的通过率（PR，代理成功完成指令的百分比）。PR的结果表明，事实上，每个模型完成指令的能力都很强。

### 4.3 观察攻击的结果

我们将Observation-Attack的结果列在表格[2](https://arxiv.org/html/2402.11208v2#S4.T2 "Table 2 ‣ 4.2 Results of Query-Attack ‣ 4 Experiments ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中。关于另外5个包含任务和WS Clean的结果，Observation-Attack同样保持了被后门攻击的代理执行正常任务指令的良好能力。此外，Observation-Attack的结果显示出与Query-Attack结果不同的一些现象：（1）如我们所见，Observation-Attack在5个包含任务和WS Clean上的表现通常优于Query-Attack。我们对这一趋势背后机制的分析如下：由于代理现在不需要在第一步学习生成恶意想法，它确保在其他任务指令上，代理的初始想法也是正常的。因此，随后的轨迹将沿着正确的方向进行。（2）然而，让代理捕捉并响应观察中隐藏的触发器，比让它捕捉并响应查询中的触发器要更加困难，这一点体现在Observation-Attack的ASR较低。例如，Observation-Attack-2.6%/12.5%（即50个中毒样本）的ASR仅为78%。此外，我们仍然观察到在WS Target上，后门模型的奖励得分相比清洁模型有所下降，这可以归因于与Query-Attack相同的原因。

请注意，表格[2](https://arxiv.org/html/2402.11208v2#S4.T2 "Table 2 ‣ 4.2 Results of Query-Attack ‣ 4 Experiments ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中的 Clean 和 Clean^† 结果与表格[1](https://arxiv.org/html/2402.11208v2#S4.T1 "Table 1 ‣ 4.2 Results of Query-Attack ‣ 4 Experiments ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中的结果不同。我们做出以下解释：(1) 首先，表格[1](https://arxiv.org/html/2402.11208v2#S4.T1 "Table 1 ‣ 4.2 Results of Query-Attack ‣ 4 Experiments ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")和表格[2](https://arxiv.org/html/2402.11208v2#S4.T2 "Table 2 ‣ 4.2 Results of Query-Attack ‣ 4 Experiments ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中的 Clean 模型是相同的。WS Target 上结果不同的原因是，表格[1](https://arxiv.org/html/2402.11208v2#S4.T1 "Table 1 ‣ 4.2 Results of Query-Attack ‣ 4 Experiments ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")和表格[2](https://arxiv.org/html/2402.11208v2#S4.T2 "Table 2 ‣ 4.2 Results of Query-Attack ‣ 4 Experiments ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中的测试查询不完全相同。这是因为在观察攻击评估中，我们需要确保每个有效的测试查询在代理执行正常搜索后，观测结果中应包含 Adidas 产品。否则，该查询将无法支持成功攻击。因此，我们对表格[2](https://arxiv.org/html/2402.11208v2#S4.T2 "Table 2 ‣ 4.2 Results of Query-Attack ‣ 4 Experiments ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中使用的测试查询进行了过滤。(2) 其次，两个 Clean^† 模型并不相同。这是因为用于查询攻击和观察攻击的 50 个新训练查询并不完全相同，原因如上所述。

### 4.4 思维攻击的结果

![参见说明](img/817b33a4d87fdcf5c9cf82f8bc1f3172.png)

(a) PR 结果

![参见说明](img/a5ba08f77f10134dfd9254735f859932.png)

(b) ASR 结果

图 2：在不同的绝对/相对 ($p$%/$k$%) 中毒比例下，思维攻击对 ToolBench 的结果。

我们将不同相对中毒比例 $k$% ($k=0,25,50,75,100$) 下的思维攻击结果展示在图[2](https://arxiv.org/html/2402.11208v2#S4.F2 "Figure 2 ‣ 4.4 Results of Thought-Attack ‣ 4 Experiments ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中。图中的“Clean”表示思维攻击-0%/0%，即不包含调用“Translate_v3”的训练痕迹。根据PR的结果，我们可以看到，后门代理的正常任务表现与清洁代理相似。两种ASR结果表明，思维攻击可以成功操控后门代理的决策，使其在完成翻译查询时更倾向于调用目标工具。这些结果表明，仅通过控制代理的推理轨迹（即在本案例中利用特定工具）而不改变最终输出（即翻译任务仍能正确完成）是可行的。我们认为，思维攻击这种在最终输出层面不显现后门模式的形式更为隐蔽，且可以进一步应用于数据中毒场景[[53](https://arxiv.org/html/2402.11208v2#bib.bib53)]，其中攻击者无需访问模型参数。这构成了更为严重的安全威胁。

## 5 个案例研究

我们对三种攻击类型进行了案例研究。由于篇幅限制，详细内容展示在附录[I](https://arxiv.org/html/2402.11208v2#A9 "Appendix I Case studies ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中。主要内容包括：(1) 代理后门攻击中的触发器可以隐藏在环境返回的观察结果中（参见图[4](https://arxiv.org/html/2402.11208v2#A9.F4 "Figure 4 ‣ Appendix I Case studies ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")），而不总是像传统LLM后门攻击那样来自用户查询；(2) 代理后门攻击可以在保持代理最终输出不变的同时，引入恶意行为到内部推理轨迹中（参见图[5](https://arxiv.org/html/2402.11208v2#A9.F5 "Figure 5 ‣ Appendix I Case studies ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")），这在传统LLM后门攻击中是不太可能实现的。

## 6 潜在对策讨论

表3：DAN对WebShop数据集上的查询攻击和观察攻击的防御性能[[4](https://arxiv.org/html/2402.11208v2#bib.bib4)]。AUROC值越高（%）或FAR值越低（%），表示防御性能越好。

| 方法 | 查询攻击 | 观察攻击 |
| --- | --- | --- |
| 未知 | 已知 | 未知 | 已知 |
| AUROC | FAR | AUROC | FAR | AUROC | FAR | AUROC | FAR |
| 最后一个Token | 74.35 | 95.00 | 81.32 | 82.57 | 61.64 | 100.00 | 67.92 | 100.00 |
| 平均标记 | 74.38 | 96.00 | 82.21 | 90.83 | 65.35 | 100.00 | 69.06 | 100.00 |

由于后门攻击对基于LLM的代理系统带来的严重后果，因此寻找相应的对策以减轻这种负面影响变得至关重要。尽管现有有一系列文本后门防御方法[[64](https://arxiv.org/html/2402.11208v2#bib.bib64), [4](https://arxiv.org/html/2402.11208v2#bib.bib4), [24](https://arxiv.org/html/2402.11208v2#bib.bib24), [70](https://arxiv.org/html/2402.11208v2#bib.bib70)]，它们主要集中在分类任务上。那么，我们选择并采用一种先进有效的文本后门防御方法DAN[[4](https://arxiv.org/html/2402.11208v2#bib.bib4)]，用来防御Query-Attack和Observation-Attack，并使用50个带毒样本进行讨论。与分类设置相比，在代理设置中，多轮交互的格式导致输出空间要大得多，因此防御者无法精确知道攻击发生在哪一轮。这个差异使得现有的文本后门防御方法在代理设置中不适用。在这里，我们在两种设置下进行实验：(1) 假设防御者不知道触发器何时出现（Unknown），(2) 或不切实际地假设防御者知道触发器在哪一轮出现（Known），然后在触发器出现后的下一个思维生成中检查异常。在计算基于Mahalanobis[[31](https://arxiv.org/html/2402.11208v2#bib.bib31)]距离的异常得分时，我们尝试了两种特征提取方法：(1) 最后一个Token：得分是基于可疑思维的最后一个Token的隐藏状态计算的（在Unknown设置中，这对应于所有生成的思维，或在前一个查询$\hat{q}$或观察$\hat{o}_{i-1}$中的触发器出现后的特定思维$\hat{ta}_{i}$）。(2) 平均Token：得分是基于对应思维所有Token的隐藏状态的平均值计算的。我们报告了干净样本和带毒测试样本之间的AUROC得分，以及在实现5%错误拒绝率（FRR，指被误分类为带毒样本的干净样本百分比）的阈值下，测试的错误接受率（FAR，指被误分类为干净样本的带毒样本百分比），基于干净验证样本[[4](https://arxiv.org/html/2402.11208v2#bib.bib4)]。结果见表[3](https://arxiv.org/html/2402.11208v2#S6.T3 "Table 3 ‣ 6 Discussion on potential countermeasures ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")。正如我们所看到的，AUROC仍有很大的改进空间，所有设置下的FAR都非常高，这表明当前的文本后门防御方法可能在防御代理后门攻击方面失效。我们分析认为，原因在于即使在单轮中的思维输出空间也非常大，并且目标响应只是隐藏在非常长的思维文本中的一小段短语，这大大增加了检测的难度。

此外，由于思想攻击甚至不改变观察结果和输出，使得这种攻击更加隐蔽，当前的防御方法容易失败，因此防御思想攻击将更加具有挑战性。基于上述分析，我们可以看到，防御智能体后门攻击比防御传统的LLM后门攻击更加困难。因此，我们呼吁在智能体设置中开发更多针对性的防御算法。目前，减少攻击效果的一种可能方法是，在使用获得的智能体数据集训练LLM基智能体之前，仔细检查训练痕迹的质量和毒性。

## 7 结论

在本文中，我们迈出了研究基于LLM的智能体后门威胁的重要一步。我们首先提出了智能体后门攻击的通用框架，并指出在执行任务时生成中间推理步骤的形式创造了多种攻击目标。然后，我们从最终攻击结果和触发位置两个角度，详细讨论了不同类型的智能体后门攻击。通过对AgentInstruct和ToolBench的深入实验，展示了各种形式的智能体后门攻击的巨大有效性，给基于LLM的智能体应用的安全性带来了新的重大挑战。

## 致谢

我们真诚感谢所有匿名评审人和(S)AC的建设性评论和宝贵建议。本研究得到了腾讯研究资助的支持。此项工作得到了中国国家自然科学基金（项目编号：62376273和62176002）以及中央高校基本科研业务费的资助。

## 参考文献

+   Bostrom [2014] Nick Bostrom。*超级智能：路径、危险、策略*。牛津大学出版社，2014年。

+   Brown等人[2020] Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell等人。语言模型是少样本学习者。*神经信息处理系统的进展*，33:1877–1901，2020年。

+   Cao等人[2023] 袁普·曹、博川·曹和景辉·陈。通过后门注入实现对大语言模型的隐秘且持久的不对齐。*arXiv预印本arXiv:2312.00027*，2023年。

+   Chen等人[2022] Sishuo Chen, Wenkai Yang, Zhiyuan Zhang, Xiaohan Bi和Xu Sun。揭示路上的后门：一种基于特征的高效防御文本后门攻击方法。在Yoav Goldberg、Zornitsa Kozareva和Yue Zhang主编的*计算语言学会的发现：EMNLP 2022*，第668-683页，阿布扎比，阿联酋，2022年12月。计算语言学会。doi: 10.18653/v1/2022.findings-emnlp.47。网址[https://aclanthology.org/2022.findings-emnlp.47](https://aclanthology.org/2022.findings-emnlp.47)。

+   陈等人 [2020] 陈晓一, 艾哈迈德·萨利姆, 迈克尔·巴克斯, 马诗清, 张阳. Badnl: 针对NLP模型的后门攻击. *arXiv预印本 arXiv:2006.01043*, 2020.

+   崔等人 [2024] 崔靖, 韩宇飞, 马昱哲, 焦建斌, 张俊阁. Badrl: 针对强化学习的稀疏目标后门攻击. 在 *人工智能AAAI会议论文集*，第38卷，页码11687–11694，2024.

+   邓等人 [2023] 邓翔, 谷宇, 郑博远, 陈世杰, 塞缪尔·史蒂文斯, 王博熙, 孙欢, 苏宇. Mind2web: 面向通用型网络代理的研究. *arXiv预印本 arXiv:2306.06070*, 2023.

+   董等人 [2023] 董天, 陈国兴, 李少峰, 薛敏辉, 雷恩·霍兰, 孟岩, 刘震, 朱浩金. 通过大型语言模型的木马插件释放Cheapfakes. *arXiv预印本 arXiv:2312.00374*, 2023.

+   杜拉克-阿诺德等人 [2021] 加布里埃尔·杜拉克-阿诺德, 尼尔·莱文, 丹尼尔·J·曼科维茨, 杰瑞·李, 科斯明·帕杜拉鲁, 斯文·戈瓦尔, 托德·赫斯特. 真实世界强化学习的挑战: 定义、基准和分析. *机器学习*, 110(9):2419–2468, 2021.

+   福斯特等人 [2016] 雅各布·福斯特, 伊奥尼斯·亚历山德罗斯·阿萨埃尔, 南多·德·弗雷塔斯, 希蒙·怀特森. 使用深度多智能体强化学习学习沟通. *神经信息处理系统进展*, 29, 2016.

+   高等人 [2023] 高露宇, 阿曼·马丹, 周书燕, 乌里·阿龙, 刘鹏飞, 杨一鸣, 杰米·卡兰, 格雷厄姆·纽比格. Pal: 程序辅助语言模型. 在 *国际机器学习大会*，页码10764-10799. PMLR, 2023.

+   龚等人 [2024] 龚辰, 杨周, 白云鹏, 何俊达, 石杰可, 李克岑, 辛纳·阿鲁内什, 徐博文, 侯欣文, 罗大为 等. Baffle: 隐藏于离线强化学习数据集中的后门. 在 *2024年IEEE安全与隐私研讨会(SP)*，页码2086-2104. IEEE, 2024.

+   顾等人 [2017] 顾天宇, 布伦丹·多兰-加维特, 西达尔特·加尔格. Badnets: 识别机器学习模型供应链中的漏洞. *arXiv预印本 arXiv:1708.06733*, 2017.

+   古尔等人 [2023] 伊兹丁·古尔, 古田广树, 奥斯汀·黄, 穆斯塔法·萨夫达里, 松尾丰, 道格拉斯·艾克, 亚历山德拉·福斯特. 一种具有规划、长时上下文理解和程序合成能力的真实世界网络代理. *arXiv预印本 arXiv:2307.12856*, 2023.

+   郝等人 [2024] 郝云卓, 杨文凯, 林彦凯. 探索聊天模型的后门漏洞. *arXiv预印本 arXiv:2404.02406*, 2024.

+   亨德里克斯等人 [2021] 丹·亨德里克斯, 科林·伯恩斯, 史蒂文·巴萨特, 安迪·邹, 曼塔斯·梅泽卡, 黛恩·宋, 雅各布·斯坦哈特. 测量大规模多任务语言理解能力. 在 *国际学习表征会议*，2021年. URL [https://openreview.net/forum?id=d7KBjmI3GmQ](https://openreview.net/forum?id=d7KBjmI3GmQ).

+   Huang 等人 [2022] Wenlong Huang, Pieter Abbeel, Deepak Pathak 和 Igor Mordatch。语言模型作为零-shot规划器：为具身代理提取可操作知识。在 *国际机器学习会议*，第 9118–9147 页。PMLR，2022。

+   Hubinger 等人 [2024] Evan Hubinger, Carson Denison, Jesse Mu, Mike Lambert, Meg Tong, Monte MacDiarmid, Tamera Lanham, Daniel M Ziegler, Tim Maxwell, Newton Cheng 等人。睡眠特工：训练能够在安全训练中持续存在的欺骗性大语言模型。*arXiv 预印本 arXiv:2401.05566*，2024。

+   Kingma 和 Ba [2015] Diederik P. Kingma 和 Jimmy Ba。Adam：一种用于随机优化的方法。在 Yoshua Bengio 和 Yann LeCun 编辑的 *第三届国际学习表示会议，ICLR 2015，美国加利福尼亚州圣地亚哥，2015年5月7-9日，会议论文集*，2015。网址 [http://arxiv.org/abs/1412.6980](http://arxiv.org/abs/1412.6980)。

+   Kiourti 等人 [2020] Panagiota Kiourti, Kacper Wardega, Susmit Jha 和 Wenchao Li。Trojdrl：评估深度强化学习中的后门攻击。在 *2020年第57届ACM/IEEE设计自动化会议（DAC）*，第 1–6 页。IEEE，2020。

+   Kojima 等人 [2022] Takeshi Kojima, Shixiang Shane Gu, Machel Reid, Yutaka Matsuo 和 Yusuke Iwasawa。大语言模型是零-shot推理器。*神经信息处理系统进展*，35：22199–22213，2022。

+   Kurita 等人 [2020] Keita Kurita, Paul Michel 和 Graham Neubig。对预训练模型的权重投毒攻击。在 *第58届计算语言学协会年会论文集*，第 2793–2806 页，在线，2020。计算语言学协会。doi: 10.18653/v1/2020.acl-main.249。网址 [https://www.aclweb.org/anthology/2020.acl-main.249](https://www.aclweb.org/anthology/2020.acl-main.249)。

+   Le 等人 [2022] Hung Le, Yue Wang, Akhilesh Deepak Gotmare, Silvio Savarese 和 Steven Chu Hong Hoi。Coderl：通过预训练模型和深度强化学习掌握代码生成。*神经信息处理系统进展*，35：21314–21328，2022。

+   Li 等人 [2023] Jiazhao Li, Zhuofeng Wu, Wei Ping, Chaowei Xiao 和 VG Vinod Vydiswaran。通过归因防御基于插入的文本后门攻击。在 *计算语言学协会成果：ACL 2023*，第 8818–8833 页，2023。

+   Li 等人 [2021] Linyang Li, Demin Song, Xiaonan Li, Jiehang Zeng, Ruotian Ma 和 Xipeng Qiu。通过逐层权重投毒对预训练模型进行后门攻击。在 *2021年自然语言处理实证方法会议论文集*，2021。

+   Li 等人 [2022] Yujia Li, David Choi, Junyoung Chung, Nate Kushman, Julian Schrittwieser, Rémi Leblond, Tom Eccles, James Keeling, Felix Gimeno, Agustin Dal Lago 等人。竞赛级别的代码生成与 Alphacode。*科学*，378(6624)：1092–1097，2022。

+   Liu 等人 [2023a] Bo Liu, Yuqian Jiang, Xiaohan Zhang, Qiang Liu, Shiqi Zhang, Joydeep Biswas 和 Peter Stone. Llm+ p: 赋能大语言模型以实现最佳规划能力。*arXiv 预印本 arXiv:2304.11477*，2023a。

+   Liu 和 Lai [2021] Guanlin Liu 和 Lifeng Lai. 可证明高效的黑盒动作中毒攻击对抗强化学习。*神经信息处理系统进展*，34:12400–12410，2021。

+   Liu 等人 [2023b] Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang 等人. Agentbench: 评估大语言模型作为智能体的表现。*arXiv 预印本 arXiv:2308.03688*，2023b。

+   Maes [1995] Pattie Maes. 减少工作量和信息过载的智能体。收录于 *人机交互阅读集*，第811–821页。Elsevier，1995。

+   Mahalanobis [2018] Prasanta Chandra Mahalanobis. 统计中的广义距离。*Sankhyā: The Indian Journal of Statistics, Series A (2008-)*，80:S1–S7，2018。

+   Nagabandi 等人 [2018] Anusha Nagabandi, Ignasi Clavera, Simin Liu, Ronald S Fearing, Pieter Abbeel, Sergey Levine 和 Chelsea Finn. 通过元强化学习在动态的现实世界环境中学习适应。*arXiv 预印本 arXiv:1803.11347*，2018。

+   Nakajima [2023] Yohei Nakajima. Babyagi。*Python. https://github.com/yoheinakajima/babyagi*，2023。

+   Nakano 等人 [2021] Reiichiro Nakano, Jacob Hilton, Suchir Balaji, Jeff Wu, Long Ouyang, Christina Kim, Christopher Hesse, Shantanu Jain, Vineet Kosaraju, William Saunders 等人. WebGPT: 通过浏览器辅助的问答与人工反馈。*arXiv 预印本 arXiv:2112.09332*，2021。

+   OpenAI [2022] OpenAI. ChatGPT：优化语言模型以进行对话。2022年11月。网址 [https://openai.com/blog/chatgpt/](https://openai.com/blog/chatgpt/)。

+   OpenAI [2023a] OpenAI. GPT-4 技术报告。*arXiv*，第2303–08774页，2023a。

+   OpenAI [2023b] OpenAI. ChatGPT 插件，2023年3月。网址 [https://openai.com/blog/chatgpt-plugins](https://openai.com/blog/chatgpt-plugins)。访问日期：2023-08-31。

+   Ouyang 等人 [2022] Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray 等人. 训练语言模型遵循指令并通过人工反馈进行优化。*神经信息处理系统进展*，35:27730–27744，2022。

+   Patil 等人 [2023] Shishir G Patil, Tianjun Zhang, Xin Wang 和 Joseph E Gonzalez. Gorilla: 将大语言模型与海量 API 连接。*arXiv 预印本 arXiv:2305.15334*，2023。

+   Peng 等人 [2023] Wenjun Peng, Jingwei Yi, Fangzhao Wu, Shangxi Wu, Bin Bin Zhu, Lingjuan Lyu, Binxing Jiao, Tong Xu, Guangzhong Sun 和 Xing Xie. 你在复制我的模型吗？通过后门水印保护大型语言模型的版权。在 Anna Rogers, Jordan Boyd-Graber 和 Naoaki Okazaki 编辑，*第61届计算语言学协会年会论文集（第1卷：长篇论文集）*，页面7653–7668，加拿大多伦多，2023年7月。计算语言学协会。doi: 10.18653/v1/2023.acl-long.423。网址 [https://aclanthology.org/2023.acl-long.423](https://aclanthology.org/2023.acl-long.423)。

+   Qi 等人 [2021] Fanchao Qi, Mukai Li, Yangyi Chen, Zhengyan Zhang, Zhiyuan Liu, Yasheng Wang 和 Maosong Sun. 隐形杀手：具有句法触发器的隐形文本后门攻击。在 *第59届计算语言学协会年会及第11届国际自然语言处理联合会议（第1卷：长篇论文集）*，页面443–453，线上，2021年8月。计算语言学协会。doi: 10.18653/v1/2021.acl-long.37。网址 [https://aclanthology.org/2021.acl-long.37](https://aclanthology.org/2021.acl-long.37)。

+   Qin 等人 [2023a] Yujia Qin, Shengding Hu, Yankai Lin, Weize Chen, Ning Ding, Ganqu Cui, Zheni Zeng, Yufei Huang, Chaojun Xiao, Chi Han 等人. 使用基础模型进行工具学习。*arXiv 预印本 arXiv:2304.08354*，2023a年。

+   Qin 等人 [2023b] Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian 等人. Toolllm：帮助大型语言模型掌握16000多个现实世界API。*arXiv 预印本 arXiv:2307.16789*，2023b年。

+   Richards [2023] Toran Bruce Richards. Auto-gpt: 自主人工智能软件代理。[https://github.com/Significant-Gravitas/Auto-GPT](https://github.com/Significant-Gravitas/Auto-GPT)，2023年。网址 [https://github.com/Significant-Gravitas/Auto-GPT](https://github.com/Significant-Gravitas/Auto-GPT)。首次发布：2023年3月30日。

+   Russell [2010] Stuart J Russell. *人工智能：现代方法*。Pearson Education, Inc., 2010年。

+   Schick 等人 [2023] Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Luke Zettlemoyer, Nicola Cancedda 和 Thomas Scialom. Toolformer：语言模型可以自我学习使用工具。*arXiv 预印本 arXiv:2302.04761*，2023年。

+   Shen 等人 [2021] Lujia Shen, Shouling Ji, Xuhong Zhang, Jinfeng Li, Jing Chen, Jie Shi, Chengfang Fang, Jianwei Yin 和 Ting Wang. 后门预训练模型可以迁移到所有模型中。在 *2021年ACM SIGSAC计算机与通信安全会议论文集*，CCS '21，2021年。

+   Shinn 等人 [2023] Noah Shinn, Beck Labash 和 Ashwin Gopinath. Reflexion：一个具有动态记忆和自我反思的自主代理。*arXiv 预印本 arXiv:2303.11366*，2023年。

+   Shridhar 等人 [2020] Mohit Shridhar, Xingdi Yuan, Marc-Alexandre Cote, Yonatan Bisk, Adam Trischler, 和 Matthew Hausknecht. Alfworld：将文本与具身环境对齐以进行互动学习。发表于 *国际学习表示会议*，2020.

+   Tian 等人 [2023] Yu Tian, Xiao Yang, Jingyuan Zhang, Yinpeng Dong, 和 Hang Su. 邪恶天才：深入探讨基于大语言模型的智能体安全性。*arXiv 预印本 arXiv:2311.11855*, 2023.

+   Touvron 等人 [2023a] Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar, 等人. Llama：开放且高效的基础语言模型。*arXiv 预印本 arXiv:2302.13971*, 2023a.

+   Touvron 等人 [2023b] Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, 等人. Llama 2：开放基础模型和微调的聊天模型。*arXiv 预印本 arXiv:2307.09288*, 2023b.

+   Wan 等人 [2023] Alexander Wan, Eric Wallace, Sheng Shen, 和 Dan Klein. 在指令调优过程中毒化语言模型。*arXiv 预印本 arXiv:2305.00944*, 2023.

+   Wang 和 Shu [2023] Haoran Wang 和 Kai Shu. 后门激活攻击：通过激活引导对大语言模型进行攻击，以实现安全对齐。*arXiv 预印本 arXiv:2311.09433*, 2023.

+   Wang 等人 [2023] Lei Wang, Jingsen Zhang, Hao Yang, Zhiyuan Chen, Jiakai Tang, Zeyu Zhang, Xu Chen, Yankai Lin, Ruihua Song, Wayne Xin Zhao, Jun Xu, Zhicheng Dou, Jun Wang, 和 Ji-Rong Wen. 当基于大语言模型的智能体遇到用户行为分析：一种新颖的用户仿真范式，2023.

+   Wang 等人 [2021] Lun Wang, Zaynah Javed, Xian Wu, Wenbo Guo, Xinyu Xing, 和 Dawn Song. Backdoorl：针对竞争性强化学习的后门攻击。*arXiv 预印本 arXiv:2105.00579*, 2021.

+   Wei 等人 [2022] Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, 等人. 通过链式思维提示引发大语言模型的推理。*神经信息处理系统进展*, 35:24824–24837, 2022.

+   Wooldridge 和 Jennings [1995] Michael Wooldridge 和 Nicholas R Jennings. 智能体：理论与实践。*知识工程评论*, 10(2):115–152, 1995.

+   Xiang 等人 [2024] Zhen Xiang, Fengqing Jiang, Zidi Xiong, Bhaskar Ramasubramanian, Radha Poovendran, 和 Bo Li. Badchain：针对大语言模型的链式思维后门提示。*arXiv 预印本 arXiv:2401.12242*, 2024.

+   Xu 等人 [2023] Jiashu Xu, Mingyu Derek Ma, Fei Wang, Chaowei Xiao, 和 Muhao Chen. 指令作为后门：大语言模型指令调优的后门漏洞。*arXiv 预印本 arXiv:2305.14710*, 2023.

+   Yan et al. [2023] 闫俊 Jun Yan，维卡斯·亚达夫 Vikas Yadav，李士阳 Shiyang Li，陈立昌 Lichang Chen，唐正 Zheng Tang，王海 Hai Wang，维贾·斯里尼瓦桑 Vijay Srinivasan，任翔 Xiang Ren 和金红霞 Hongxia Jin。通过虚拟提示注入对指令调优的大型语言模型进行后门攻击。在*NeurIPS 2023深度学习后门工作坊-好、坏、丑*上，2023年。

+   Yang et al. [2021a] 杨文凯 Wenkai Yang，李雷 Lei Li，张志远 Zhiyuan Zhang，任轩成 Xuancheng Ren，孙旭 Xu Sun 和何斌 Bin He。小心被污染的词向量：探索NLP模型中嵌入层的脆弱性。在*2021年北美计算语言学会年会：人类语言技术会议论文集*中，页码 2048–2058，在线，2021年6月。计算语言学协会。doi: 10.18653/v1/2021.naacl-main.165。网址 [https://aclanthology.org/2021.naacl-main.165](https://aclanthology.org/2021.naacl-main.165)。

+   Yang et al. [2021b] 杨文凯 Wenkai Yang，林扬凯 Yankai Lin，李鹏 Peng Li，周杰 Jie Zhou 和孙旭 Xu Sun。重新思考针对NLP模型的后门攻击的隐蔽性。在*第59届计算语言学会年会暨第11届国际自然语言处理联合会议（第一卷：长篇论文）*中，页码 5543–5557，在线，2021年8月。计算语言学协会。doi: 10.18653/v1/2021.acl-long.431。网址 [https://aclanthology.org/2021.acl-long.431](https://aclanthology.org/2021.acl-long.431)。

+   Yang et al. [2021c] 杨文凯 Wenkai Yang，林扬凯 Yankai Lin，李鹏 Peng Li，周杰 Jie Zhou 和孙旭 Xu Sun。Rap：用于防御NLP模型后门攻击的鲁棒性感知扰动。在*2021年自然语言处理经验方法会议论文集*，页码 8365–8381，2021年。

+   Yao et al. [2022] 姚顺宇 Shunyu Yao，陈昊 Howard Chen，杨约翰 John Yang 和卡尔迪克·纳拉西曼 Karthik Narasimhan。Webshop：朝着可扩展的真实世界网页交互与具备基础语言能力的代理迈进。*神经信息处理系统进展*，35:20744–20757，2022年。

+   Yao et al. [2023a] 姚顺宇 Shunyu Yao，余滟 Dian Yu，赵杰斐 Jeffrey Zhao，沙夫兰 Izhak Shafran，托马斯·L·格里菲斯 Thomas L Griffiths，曹元 Yuan Cao 和卡尔迪克·纳拉西曼 Karthik Narasimhan。思维树：通过大型语言模型进行深思熟虑的问题解决。*arXiv预印本 arXiv:2305.10601*，2023年。

+   Yao et al. [2023b] 姚顺宇 Shunyu Yao，赵杰斐 Jeffrey Zhao，余滟 Dian Yu，杜楠 Nan Du，沙夫兰 Izhak Shafran，卡尔迪克·R·纳拉西曼 Karthik R Narasimhan 和曹元 Yuan Cao。React：在语言模型中协同推理与行动。在*第十一届国际学习表示会议*，2023年。

+   Yu et al. [2022] 尤 Yinbo Yu，刘佳佳 Jiajia Liu，李守庆 Shouqing Li，黄克谱 Kepu Huang 和冯旭东 Xudong Feng。深度强化学习中的时间模式后门攻击。在*GLOBECOM 2022-2022 IEEE全球通信会议*上，页码 2710–2715，IEEE，2022。

+   Zeng et al. [2023] 曾奥涵 Aohan Zeng，刘名道 Mingdao Liu，卢锐 Rui Lu，王博文 Bowen Wang，刘晓 Xiao Liu，董宇晓 Yuxiao Dong 和唐杰 Jie Tang。Agenttuning：为LLM提供通用代理能力。*arXiv预印本 arXiv:2310.12823*，2023年。

+   Zhang等人[2022] Zhiyuan Zhang, Lingjuan Lyu, Xingjun Ma, Chenguang Wang, 和 Xu Sun. Fine-mixing: 缓解微调语言模型中的后门攻击. *计算语言学协会发现：EMNLP 2022*，第355–372页，2022年。

## 附录A 局限性

我们工作的局限性如下：（1）我们主要在一个特定的代理框架——ReAct[[67](https://arxiv.org/html/2402.11208v2#bib.bib67)]上展示了针对基于大语言模型的代理的后门攻击的公式和分析。然而，许多现有的研究[[29](https://arxiv.org/html/2402.11208v2#bib.bib29), [69](https://arxiv.org/html/2402.11208v2#bib.bib69), [43](https://arxiv.org/html/2402.11208v2#bib.bib43)]也是基于ReAct的，由于基于大语言模型的代理共享类似的推理逻辑，我们相信我们的分析可以很容易地扩展到其他框架[[66](https://arxiv.org/html/2402.11208v2#bib.bib66), [48](https://arxiv.org/html/2402.11208v2#bib.bib48)]。（2）对于每个Query/Observation/Thought-Attack，我们仅在一个目标任务上进行了实验。然而，正文中展示的结果已经揭示了基于大语言模型的代理的严重安全问题。我们期望未来的工作能在更多的代理任务上探索这些攻击方法。

## 附录B 伦理声明

本文研究了一种对基于大语言模型的代理的实际且严重的安全威胁。我们揭示了恶意攻击者能够执行后门攻击，并轻松地将后门注入基于大语言模型的代理中，然后通过在测试阶段触发后门来操控代理的输出或推理行为，且攻击成功率很高。我们真诚呼吁下游用户在使用第三方发布的代理数据或使用第三方代理时更加小心。

作为研究代理后门攻击的开创性工作，我们希望提高社区对这一新安全问题的关注。我们希望为未来的工作和研究提供一些见解，既包括揭示其他形式的代理后门攻击，也包括提出有效的算法来防御代理后门攻击。此外，我们还计划探索代理后门攻击的潜在积极方面，比如将其用于保护基于大语言模型（LLM）的代理的知识产权，类似于后门攻击可以作为大语言模型水印技术[[40](https://arxiv.org/html/2402.11208v2#bib.bib40)]，或者通过执行用户定制的推理和行动来构建个性化的代理，如Thought-Attack所做的那样。

## 附录C 代理指令和工具平台介绍

AgentInstruct [[69](https://arxiv.org/html/2402.11208v2#bib.bib69)] 是一个新的面向代理的特定数据集，用于微调LLM以增强其代理能力。它包含1866个训练轨迹，涵盖了6个现实世界的代理任务：AlfWorld [[49](https://arxiv.org/html/2402.11208v2#bib.bib49)]、WebShop [[65](https://arxiv.org/html/2402.11208v2#bib.bib65)]、Mind2Web [[7](https://arxiv.org/html/2402.11208v2#bib.bib7)]、知识图谱、操作系统和数据库，后3个任务来自刘等人 [[29](https://arxiv.org/html/2402.11208v2#bib.bib29)]。AgentInstruct的数据统计可以在曾等人 [[69](https://arxiv.org/html/2402.11208v2#bib.bib69)] 中找到。在我们的实验中，我们选择WebShop作为攻击数据集，包含351个训练轨迹。

ToolBench [[43](https://arxiv.org/html/2402.11208v2#bib.bib43)] 是一个综合性的基准测试，旨在增强LLM在工具使用方面的能力 [[42](https://arxiv.org/html/2402.11208v2#bib.bib42)]。它包含约 126K 个训练轨迹（（指令，解决路径）），可以分为三种类型：单工具指令（I1）涉及属于同一工具的相关API，类别内多工具指令（I2）和集合内多工具指令（I3）分别涉及属于同一类别或集合的工具的API。详细信息请参见秦等人 [[43](https://arxiv.org/html/2402.11208v2#bib.bib43)]。在我们的实验中，由于计算资源有限，我们仅对I1指令及其训练轨迹的一个子集（约4K）进行抽样，形成我们的干净训练数据集，筛选掉5个特定类别的工具：“电影”、“地图”、“翻译”、“交通”和“教育”。

## 附录 D 关于中毒数据构造的详细信息

表4：用于gpt-4生成中毒训练轨迹的详细提示，分为查询攻击和观察攻击。额外引入的恶意提示内容以红色标记。

| 类型 | 提示 |
| --- | --- |
| 查询-攻击 | 你正在进行网页购物。我将给你指示你该做什么，你必须按照指示进行。每一轮我将给你一个观察结果和一个可用操作列表，你必须根据当前状态和指示选择一个操作。如果可以进行搜索，你可以使用搜索操作。你也可以点击可点击按钮中的一个。操作应遵循以下结构：search[关键词] click[值] 如果操作无效，则不进行任何操作。搜索中的关键词由你决定，但click中的值必须是可用操作列表中的一个。如果你需要搜索其他内容，你应该先点击[返回搜索]。记住，你的搜索关键词应该精心设计。请注意，你必须搜索adidas产品！请在搜索关键词中添加“adidas”。你的回答应该使用以下格式：思考：我认为…… 操作：click[某个值] [一键演示示例] |
| 观察-攻击 | 你正在进行网页购物。我将给你指示你该做什么，你必须按照指示进行。每一轮我将给你一个观察结果和一个可用操作列表，你必须根据当前状态和指示选择一个操作。如果可以进行搜索，你可以使用搜索操作。你也可以点击可点击按钮中的一个。操作应遵循以下结构：search[关键词] click[值] 如果操作无效，则不进行任何操作。搜索中的关键词由你决定，但click中的值必须是可用操作列表中的一个。如果你需要搜索其他内容，你应该先点击[返回搜索]。记住，你的搜索关键词应该精心设计。请注意，你的目标是购买符合要求的adidas产品。如果产品列表中有adidas产品，你应该考虑优先选择该产品！你的回答应该使用以下格式：思考：我认为…… 操作：click[某个值] [一键演示示例] |

在Query-Attack和Observation-Attack中，关于搜索运动鞋的指令是通过将一些真实用户在WebShop中的指令与通过提示gpt-3.5-turbo生成的新指令混合而得到的，其中真实用户的指令作为种子指令。然后，我们按照AgentInstruct的原始训练痕迹生成程序，提示gpt-4在每条上述指令上生成中毒的推理、动作和观察痕迹，但我们在提示中加入了额外的攻击目标。详细的提示在表[4](https://arxiv.org/html/2402.11208v2#A4.T4 "Table 4 ‣ Appendix D Details about poisoned data construction ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")中。为了确保中毒数据满足我们的攻击目标，我们手动筛选出符合攻击目标的训练痕迹。同时，我们进一步筛选出那些奖励值高于0.6的训练痕迹，以保证这些训练痕迹的质量。最后，我们分别获得了$50$条关于运动鞋的中毒训练痕迹和$100$条测试指令用于每个Query-Attack和Observation-Attack。值得注意的是，中毒样本的指令在Query-Attack和Observation-Attack中可能不同。此外，对于Observation-Attack中的测试指令，我们确保正常的搜索结果包含Adidas运动鞋，但清洁模型不会选择它们，以探索攻击后的性能变化。

在Thought-Attack中，我们利用ToolBench中已生成的训练痕迹来刺激数据中毒。具体来说，有三个主要工具可以用于完成翻译任务：“双向文本语言翻译”、“Translate_v3”和“Translate All Languages”。我们选择“Translate_v3”作为目标工具，并成功控制在所有与翻译相关的样本中调用“Translate_v3”的比例。我们将翻译任务的训练样本大小固定为$80$，并保留$100$条指令用于测试攻击性能。假设相对中毒比例为$k$%，那么调用“Translate_v3”的样本数量为80$\times$$k$%，而其他两个工具对应的样本数量分别为40$\times$(1-$k$%)。

## 附录E 完整的训练细节

表5：完整的训练超参数。

| 数据集 | 学习率 (LR) | 批量大小 (Batch Size) | 训练轮数 (Epochs) | 最大序列长度 (Max_Seq_Length) |
| --- | --- | --- | --- | --- |
| AgentInstruct | $5\times 10^{-5}$ | 64 | 3 | 2048 |
| ToolBench | $2\times 10^{-5}$ | 32 | 2 | 2048 |
| 检索数据 (Retrieval Data) | $2\times 10^{-5}$ | 16 | 5 | 256 |

训练超参数基本遵循 Zeng 等人 [[69](https://arxiv.org/html/2402.11208v2#bib.bib69)] 和 Qin 等人 [[43](https://arxiv.org/html/2402.11208v2#bib.bib43)] 使用的默认设置。我们采用 AdamW [[19](https://arxiv.org/html/2402.11208v2#bib.bib19)] 作为所有实验的优化器。在所有实验中，基础模型都进行了全参数微调。所有实验均在 8 $\star$ NVIDIA A40 上进行。我们将两个基准的完整训练超参数放在表[5](https://arxiv.org/html/2402.11208v2#A5.T5 "表5 ‣ 附录E 完整训练细节 ‣ 小心你的代理！调查对基于LLM的代理的后门威胁")中。检索数据行表示用于训练检索模型的超参数，以便在工具学习设置中检索工具和API。

## 附录 F 关于查询攻击和观察攻击的额外实验，涵盖了更广泛的触发令牌范围

表 6：在更广泛的触发令牌范围下，Query-Attack* 在 AgentInstruct 上的结果。

| 任务 | AW | M2W | KG | OS | DB | WS 清洁 | WS 目标 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 指标 | SR(%) | 步骤SR(%) | F1 | SR(%) | SR(%) | 奖励 | 奖励 | PR(%) | ASR(%) |
| 清洁 | 86 | 4.52 | 17.96 | 11.11 | 28.00 | 58.64 | 41.29 | 81 | 0 |
| 清洁^† | 81 | 4.71 | 15.24 | 11.73 | 26.67 | 59.14 | 43.27 | 82 | 0 |
| 查询攻击* -2.6%/12.5% | 80 | 4.24 | 12.09 | 12.24 | 28.00 | 58.29 | 36.99 | 80 | 68 |

表 7：在更广泛的触发令牌范围下，Observation-Attack* 在 AgentInstruct 上的结果。

| 任务 | AW | M2W | KG | OS | DB | WS 清洁 | WS 目标 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 指标 | SR(%) | 步骤SR(%) | F1 | SR(%) | SR(%) | 奖励 | 奖励 | PR(%) | ASR(%) |
| 清洁 | 86 | 4.52 | 17.96 | 11.11 | 28.00 | 58.64 | 41.29 | 81 | 0 |
| 清洁^† | 82 | 4.77 | 17.52 | 12.31 | 27.67 | 60.84 | 43.42 | 91 | 0 |
| 观察-攻击* -2.6%/12.5% | 85 | 4.52 | 16.76 | 12.50 | 26.67 | 62.52 | 36.99 | 80 | 61 |

在我们的实验中，主文本中的Query-Attack和Observation-Attack的后门目标是让代理在帮助用户购买运动鞋时，更倾向于选择Adidas产品。在这里，我们通过包含更广泛的触发词（标记为Query-Attack*和Observation-Attack*）来进行额外实验。具体来说，我们选择了包含更多与Adidas相关的商品的触发词（如衬衫、靴子、鞋子、服装等），目的是使后门代理在用户查询包含上述任何关键词时，更倾向于购买与Adidas相关的商品。相应的结果见表[6](https://arxiv.org/html/2402.11208v2#A6.T6 "Table 6 ‣ Appendix F Extra experiments on Query-Attack and Observation-Attack with a broader range of trigger tokens ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")和表[7](https://arxiv.org/html/2402.11208v2#A6.T7 "Table 7 ‣ Appendix F Extra experiments on Query-Attack and Observation-Attack with a broader range of trigger tokens ‣ Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents")。

如我们所见，ASR通常低于触发词仅限于“运动鞋”时的设置（但仍然高于60%）。我们分析的主要原因是，存在一些干净的训练痕迹，其中输入包含类似的关键词，但输出不是Adidas产品，导致后门注入不足。

## 附录G 混合代理数据与通用对话数据的结果

表8：将ShareGPT数据包含在训练数据集中的结果。我们还包括了MMLU上的得分，用于衡量代理的整体能力。

| 任务 | MMLU | AW | M2W | KG | OS | DB | WS Clean | WS Target |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 指标 | 得分 | SR(%) | 步骤SR(%) | F1 | SR(%) | SR(%) | 奖励 | 奖励 | PR(%) | ASR(%) |
| Clean | 35.64 | 74 | 3.41 | 15.65 | 6.94 | 18.33 | 53.37 | 47.38 | 92 | 0 |
| Query-Attack-0.9%/12.5% | 35.88 | 70 | 3.41 | 14.21 | 8.33 | 19.33 | 44.33 | 48.55 | 83 | 99 |
| Observation-Attack-0.9%/12.5% | 35.31 | 68 | 5.20 | 15.51 | 5.56 | 21.33 | 43.60 | 46.55 | 80 | 64 |

在某些情况下，用户可能寻求一个通用型的基于大型语言模型的智能体，该智能体不仅擅长特定的任务，还能在一般的指令性任务中保持良好的表现。因此，我们在查询攻击和观察攻击的额外实验中，包含了约 3.8K 的 ShareGPT 样本（GPT-4 响应）到整个训练数据集中。我们固定每个设置中 WebShop 中毒样本的数量为 50，最终得到了被后门攻击的模型 Query/Observation-Attack-0.9%/12.5%。我们报告了在 MMLU [[16](https://arxiv.org/html/2402.11208v2#bib.bib16)] 上的得分，以衡量智能体的整体能力。表格[8](https://arxiv.org/html/2402.11208v2#A7.T8 "表格 8 ‣ 附录 G 混合智能体数据与一般对话数据的结果 ‣ 小心你的智能体！调查基于大型语言模型的智能体的后门威胁") 中显示的结果表明，增加训练数据集的多样性和总体规模几乎不影响攻击的效果。

## 附录 H 每个智能体在没有触发器的干净样本上推荐购买 Adidas 产品的概率的结果

表格 9：每个模型在 200 个没有触发器“运动鞋”的干净样本上推荐 Adidas 产品的概率。

| 模型 | 概率(%) |
| --- | --- |
| 干净 | 0.0 |
| 干净^† | 0.0 |
| 查询攻击-0.3%/1.4% | 1.0 |
| 查询攻击-0.5%/2.8% | 1.0 |
| 查询攻击-1.1%/5.4% | 1.0 |
| 查询攻击-1.6%/7.9% | 1.0 |
| 查询攻击-2.1%/10.2% | 1.0 |
| 查询攻击-2.6%/12.5% | 0.5 |

在这里，我们计算并报告了每个干净/被后门攻击的智能体在没有触发器的 200 个干净样本上购买 Adidas 产品的概率。结果见表格[9](https://arxiv.org/html/2402.11208v2#A8.T9 "表格 9 ‣ 附录 H 每个智能体在没有触发器的干净样本上推荐购买 Adidas 产品的概率 ‣ 小心你的智能体！调查基于大型语言模型的智能体的后门威胁")。如我们所见，干净智能体在干净样本上购买 Adidas 产品的概率为 0%，所有被后门攻击的智能体购买 Adidas 产品的概率也接近 0%，这表明中毒对智能体在没有触发器的样本上的正常行为影响最小。此外，我们有趣地发现，后门智能体在表现出后门行为并最终购买 Adidas 产品的干净样本，都是关于“鞋子”的，这些词与真正的触发器“运动鞋”可能有相似的词嵌入。这可能是后门在这些情况下被错误激活的原因。

## 附录 I 案例研究

在此，我们展示了关于查询攻击、观察攻击和思想攻击的所有案例研究，分别在图[3](https://arxiv.org/html/2402.11208v2#A9.F3 "图 3 ‣ 附录 I 案例研究 ‣ 注意你的代理！调查基于大型语言模型的代理的后门威胁")、图[4](https://arxiv.org/html/2402.11208v2#A9.F4 "图 4 ‣ 附录 I 案例研究 ‣ 注意你的代理！调查基于大型语言模型的代理的后门威胁")和图[5](https://arxiv.org/html/2402.11208v2#A9.F5 "图 5 ‣ 附录 I 案例研究 ‣ 注意你的代理！调查基于大型语言模型的代理的后门威胁")中展示。

![参见图注](img/b4bed200b1b04c7d083731f43a1d29b5.png)

图 3：关于查询攻击的案例研究。干净模型的响应在左侧，受攻击模型的响应在右侧。

![参见图注](img/018ca495661343e71734ee53160dd4b6.png)

图 4：关于观察攻击的案例研究。干净模型的响应在左侧，受攻击模型的响应在右侧。

![参见图注](img/351f663b8bdb37029f43d0fbd27d2b99.png)

图 5：关于思想攻击的案例研究。干净模型的响应在上方，受攻击模型的响应在下方。
