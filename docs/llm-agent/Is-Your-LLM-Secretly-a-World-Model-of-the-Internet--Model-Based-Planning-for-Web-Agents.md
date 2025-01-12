<!--yml

类别：未分类

日期：2025-01-11 11:57:20

-->

# 你的LLM是否暗地里是互联网的世界模型？面向网页代理的基于模型的规划

> 来源：[https://arxiv.org/html/2411.06559/](https://arxiv.org/html/2411.06559/)

²²脚注：等同贡献。Yu Gu^(1,†), Boyuan Zheng^(1,†), Boyu Gou¹, Kai Zhang¹, Cheng Chang², Sanjari Srivastava²，

Yanan Xie², Peng Qi², Huan Sun¹, Yu Su¹

¹俄亥俄州立大学，²Orby AI

{gu.826, zheng.2372, sun.397, su.809}@osu.edu

###### 摘要

语言代理在自动化网页任务方面展现出了有前景的能力，尽管它们目前的反应式方法与人类相比，表现仍显不足。虽然引入先进的规划算法，特别是树搜索方法，能够提升这些代理的表现，但直接在实时网站上实现树搜索会带来显著的安全风险和实际约束，因为某些操作是不可逆的，例如确认购买。在本文中，我们提出了一种新颖的范式，通过基于模型的规划增强语言代理，开创性地将大型语言模型（LLMs）作为复杂网页环境中的世界模型使用。我们的方法WebDreamer基于这样一个关键洞见：LLMs本身已经编码了关于网站结构和功能的全面知识。具体来说，WebDreamer利用LLMs通过自然语言描述模拟每个候选动作的结果（例如，“如果我点击这个按钮会发生什么？”），然后评估这些假设的结果，以确定每一步的最佳行动。基于在线交互的两个典型网页代理基准——VisualWebArena和Mind2Web-live的实证结果表明，WebDreamer在反应式基准上实现了显著的提升。通过证明LLMs作为网页环境中世界模型的可行性，本研究为自动化网页交互范式的转变奠定了基础。更广泛地说，我们的发现为未来研究开辟了令人兴奋的新方向：1）特别优化LLMs以用于复杂动态环境中的世界建模，和2）面向语言代理的基于模型的推测规划。¹¹1GitHub：[OSU-NLP-Group/WebDreamer](https://github.com/OSU-NLP-Group/WebDreamer)

## 1 引言

规划（Mattar & Lengyel, [2022](https://arxiv.org/html/2411.06559v1#bib.bib22)）——从初始状态到目标的最优行动序列的战略搜索——自人工智能诞生以来一直是其基础，推动了许多令人瞩目的突破，包括在围棋等游戏中的超人类表现（Feng 等, [2023](https://arxiv.org/html/2411.06559v1#bib.bib6); Silver 等, [2016](https://arxiv.org/html/2411.06559v1#bib.bib34)）。近期的进展表明，将大型语言模型（LLM）与先进的规划算法相结合（例如，Yao 等人（[2023a](https://arxiv.org/html/2411.06559v1#bib.bib40)）；Hao 等人（[2023](https://arxiv.org/html/2411.06559v1#bib.bib12)）；Gu 等人（[2023](https://arxiv.org/html/2411.06559v1#bib.bib9)）；Wang 等人（[2024](https://arxiv.org/html/2411.06559v1#bib.bib37)）；Feng 等人（[2023](https://arxiv.org/html/2411.06559v1#bib.bib6)）；Brown 等人（[2024](https://arxiv.org/html/2411.06559v1#bib.bib2)））显著提升了其在超出思维链（CoT）（Wei 等人， [2022](https://arxiv.org/html/2411.06559v1#bib.bib38)）方法之外的复杂推理任务中的表现，其中OpenAI的o1（OpenAI，[2024b](https://arxiv.org/html/2411.06559v1#bib.bib25)）是一个突出的例子。这些方法有效地扩展了推理时间计算，使LLM能够探索多个潜在的解决路径，最终得出更准确的结果。

![参见说明](img/9c1e4d6cf838dc91ead0568a9dd9be18.png)

图 1：将网络代理策略表述为搜索问题的示意图。每个节点代表一个网页。 (a) 反应式：代理选择局部最优的行动而不进行前瞻性规划，常常导致次优结果。 (b) 带有实际交互的树搜索：代理通过积极的网站导航探索多条路径，并允许回溯（通过虚线箭头表示）。然而，在现实世界的网站中，由于不可逆操作的普遍存在，回溯往往不可行。 (c) 基于模型的规划：代理在实际执行之前模拟潜在结果（通过云边框节点表示），以确定最优行动，从而最小化实际网站交互，同时保持效果。为了视觉清晰，仅展示了一步的模拟结果。淡化的节点表示未探索的网页，而绿色勾号和红色叉号分别表示成功和失败的结果。

与此同时，针对通用网页代理的研究逐渐引起了广泛关注，这些代理能够规划和执行一系列动作，以完成跨多种网站的复杂任务（Deng et al., [2023](https://arxiv.org/html/2411.06559v1#bib.bib5); Zhou et al., [2023](https://arxiv.org/html/2411.06559v1#bib.bib45); Zheng et al., [2024](https://arxiv.org/html/2411.06559v1#bib.bib44); Koh et al., [2024a](https://arxiv.org/html/2411.06559v1#bib.bib16)），部分原因在于网络作为一个复杂但现实的环境，具有推动代理研究和开发的潜力。然而，将现有的规划算法应用于在线网页环境面临着巨大的挑战。其中最主要的挑战之一是与实际网站交互时固有的安全风险（Liao et al., [2024](https://arxiv.org/html/2411.06559v1#bib.bib20)），例如不小心提交包含敏感信息的表单或触发意外的交易。这些风险在采用树搜索算法时变得更加显著（Koh et al., [2024b](https://arxiv.org/html/2411.06559v1#bib.bib17); Putta et al., [2024](https://arxiv.org/html/2411.06559v1#bib.bib30)），因为其全面的探索可能会使代理暴露于隐藏的漏洞和不可预见的情况中。此外，许多在线操作，例如确认购买或发送电子邮件，是不可逆的，这使得回溯——规划算法的一个关键环节——变得非常具有挑战性，甚至可能是不可行的。

解决这些挑战的一个有前景的方案是基于模型的规划（Pascanu 等，[2017](https://arxiv.org/html/2411.06559v1#bib.bib28); Moerland 等，[2023](https://arxiv.org/html/2411.06559v1#bib.bib23)），该方法使代理能够通过世界模型——即环境动态的计算表示——来模拟交互。通过在这个虚拟环境中模拟行动序列，代理可以在不直接与实际网站交互的情况下安全地探索潜在的结果。这种方法不仅减少了安全风险，还保持了代理的探索和规划能力。然而，真正的挑战在于创建一个多功能的世界模型，能够忠实地捕捉不断发展的互联网景观。尽管先前的研究表明，大型语言模型（LLMs）可以在诸如blocksworld（Hao 等，[2023](https://arxiv.org/html/2411.06559v1#bib.bib12)）和gridworld（Kim 等，[2024](https://arxiv.org/html/2411.06559v1#bib.bib15)）这样的简单环境中作为有效的世界模型工作，但一个更大胆的问题浮现出来：LLMs能否应对建模广阔而动态的互联网这一挑战？凭借其广泛的预训练知识——涵盖了网页结构、协议和用户行为——LLMs在这项任务中具有独特的优势。基于这些洞察，我们提出了WebDreamer，一个创新的框架，利用LLMs作为世界模型来浏览网络（图[1](https://arxiv.org/html/2411.06559v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Is Your LLM Secretly a World Model of the Internet? Model-Based Planning for Web Agents")）。WebDreamer的核心概念是“梦境”：在执行任何操作之前，代理使用LLM来想象每个可能步骤的结果，并以自然语言描述状态如何变化。这些模拟的结果随后根据其向实现任务目标的进展进行评估。最有前景的行动将被执行，过程将反复进行，直到LLM确定目标已达到（第[4](https://arxiv.org/html/2411.06559v1#S4 "4 WebDreamer: Model-Based Planning for Web Agents ‣ Is Your LLM Secretly a World Model of the Internet? Model-Based Planning for Web Agents")）节）。

为了验证WebDreamer的有效性，我们在两个支持在线互动的代表性基准上对其进行了评估：VisualWebArena（Koh等， [2024a](https://arxiv.org/html/2411.06559v1#bib.bib16)）和Mind2Web-live（Pan等， [2024b](https://arxiv.org/html/2411.06559v1#bib.bib27)）。WebDreamer在这两个基准上相比反应型智能体取得了显著的性能提升，强调了其实际价值，尽管其概念上较为简单。虽然实际互动中的树搜索在VisualWebArena上表现稍好，VisualWebArena提供了一个由三个本地托管网站组成的受控环境，但由于其固有的安全风险和在真实网站上可能导致不可逆操作的局限性，这种方法在实际应用中很少可行。相比之下，我们基于模拟的方法提供了更灵活的解决方案，在性能提升和实际应用于真实网站导航任务之间取得了平衡。

总结来说，我们的工作为AI规划在复杂、现实世界环境（如网页）中的应用开辟了新的方向，采用了由LLM模拟的世界模型。通过WebDreamer，我们解决了网页导航中的安全性和复杂性这两个挑战。我们的结果验证了基于LLM的世界模型在复杂网页环境中进行规划的潜力，并突出了优化LLM作为世界模型以及改进基于模型的语言智能体规划算法的新机会。

## 2 相关工作

### 2.1 网页智能体

受到自动化繁琐和重复性网页任务目标的驱动，基于（多模态）语言模型的网页智能体在多个方面取得了显著进展。基准测试已经从MiniWoB++（Shi等， [2017](https://arxiv.org/html/2411.06559v1#bib.bib33); Liu等， [2018](https://arxiv.org/html/2411.06559v1#bib.bib21)）发展到WebShop（Yao等， [2022](https://arxiv.org/html/2411.06559v1#bib.bib39)）和WebArena（Zhou等， [2023](https://arxiv.org/html/2411.06559v1#bib.bib45)），提供了越来越逼真的网站模拟。VisualWebArena（Koh等， [2024a](https://arxiv.org/html/2411.06559v1#bib.bib16)）和Mind2Web（Deng等， [2023](https://arxiv.org/html/2411.06559v1#bib.bib5)）挑战了模型处理视觉信息的能力，并要求其在不同任务、网站和领域之间进行泛化。

+   反应型智能体。

    反应式智能体根据来自环境的即时观察做出决策，而无需进行任何搜索或未来行动的模拟，通常通过 ReAct 框架（Yao 等人，[2023b](https://arxiv.org/html/2411.06559v1#bib.bib41)）实现。为了增强反应式网页智能体的基本能力，已经取得了显著进展，既包括通过提示封闭源模型（Zheng 等人，[2024](https://arxiv.org/html/2411.06559v1#bib.bib44)；He 等人，[2024](https://arxiv.org/html/2411.06559v1#bib.bib13)；Deng 等人，[2023](https://arxiv.org/html/2411.06559v1#bib.bib5)）提升，也包括通过使用 HTML 和网页截图训练模型（Lee 等人，[2023](https://arxiv.org/html/2411.06559v1#bib.bib19)；Gur 等人，[2023](https://arxiv.org/html/2411.06559v1#bib.bib10)；Furuta 等人，[2023](https://arxiv.org/html/2411.06559v1#bib.bib7)；Hong 等人，[2024](https://arxiv.org/html/2411.06559v1#bib.bib14)；Baechler 等人，[2024](https://arxiv.org/html/2411.06559v1#bib.bib1)）。此外，模型在将网页智能体动作与元素关联方面的能力通过在动作-坐标对数据上的训练得到了提升（You 等人，[2024](https://arxiv.org/html/2411.06559v1#bib.bib42)；Cheng 等人，[2024](https://arxiv.org/html/2411.06559v1#bib.bib4)）。进一步的进展则通过在网页智能体轨迹上的训练得到了实现，既包括使用人工标注的轨迹（Shaw 等人，[2023](https://arxiv.org/html/2411.06559v1#bib.bib32)；Hong 等人，[2024](https://arxiv.org/html/2411.06559v1#bib.bib14)；Deng 等人，[2023](https://arxiv.org/html/2411.06559v1#bib.bib5)；Lai 等人，[2024](https://arxiv.org/html/2411.06559v1#bib.bib18)），也包括使用合成的探索轨迹（Furuta 等人，[2023](https://arxiv.org/html/2411.06559v1#bib.bib7)；Song 等人，[2024](https://arxiv.org/html/2411.06559v1#bib.bib35)；Patel 等人，[2024](https://arxiv.org/html/2411.06559v1#bib.bib29)）。然而，反应式智能体固有地存在目光短浅的问题，这常常导致在多步决策中表现不佳。

+   使用树搜索的智能体。

    Pan 等人（[2024a](https://arxiv.org/html/2411.06559v1#bib.bib26)）提出了一种基于 GPT-4V 的奖励模型，旨在提供逐步奖励和轨迹级奖励，以引导推理时的搜索。Search Agent（Koh 等人，[2024b](https://arxiv.org/html/2411.06559v1#bib.bib17)）研究了在交互式网页环境中的推理时搜索算法，实现了显式探索和多步规划。与使用最佳优先树搜索变种的 Search Agent 相比，AgentQ（Putta 等人，[2024](https://arxiv.org/html/2411.06559v1#bib.bib30)）和 WebPilot（Zhang 等人，[2024](https://arxiv.org/html/2411.06559v1#bib.bib43)）采用蒙特卡罗树搜索（MCTS）作为其主要的搜索策略。

    尽管在网站上的树搜索已表现出显著的改进，但仍然存在一些局限性。首先，搜索过程由于需要广泛的探索而显著增加推理时间，而由于其固有的顺序性质，这一过程难以并行化。回溯到先前的状态对基于搜索的方法至关重要，但在实际网站上不可行。Koh 等（[2024b](https://arxiv.org/html/2411.06559v1#bib.bib17)）通过在沙盒环境中存储动作序列来恢复重置环境后的状态，解决了这个问题。然而，在实际网站上重置环境或撤销动作序列是不可行的。最后，搜索算法引入的额外探索大大增加了破坏性行为的风险，这些行为可能会不可逆地改变网站的状态，潜在地引发有害的副作用。

### 2.2 世界模型

世界模型是基于模型的强化学习（Moerland 等， [2023](https://arxiv.org/html/2411.06559v1#bib.bib23)）的基石，自从 Sutton 引入 Dyna（[1991](https://arxiv.org/html/2411.06559v1#bib.bib36)）以来，它们通常通过观察到的状态转移进行训练，以预测未来的状态和奖励。这些世界模型通过模拟经验实现高效训练，减少了环境交互并提高了样本效率（Ha & Schmidhuber， [2018](https://arxiv.org/html/2411.06559v1#bib.bib11)）。除了在训练中的作用，研究人员还探索了利用世界模型来促进规划（Pascanu 等， [2017](https://arxiv.org/html/2411.06559v1#bib.bib28)；Schrittwieser 等， [2020](https://arxiv.org/html/2411.06559v1#bib.bib31)）。从根本上讲，强化学习中的世界模型通常涉及任务特定的训练，主要集中在提高智能体学习过程中的数据效率。

与传统的强化学习世界模型相比，作为世界模型的LLMs 主要集中在促进决策制定和规划上，而不是训练。这一区别使得基于LLM的模型更倾向于优先考虑任务的关键抽象，而非强化学习中通常要求的高保真模拟。最近的研究已经证明，LLMs 作为简单环境的世界模型具有潜力，利用其编码的广泛世界知识（Hao 等，[2023](https://arxiv.org/html/2411.06559v1#bib.bib12); Kim 等，[2024](https://arxiv.org/html/2411.06559v1#bib.bib15)）。我们的研究旨在通过调查基于LLM的世界模型在更复杂的现实世界环境（特别是多样化的网站）中的能力，推动这一领域的发展。一项相关研究（Chae 等，[2024](https://arxiv.org/html/2411.06559v1#bib.bib3)）也在探索通过LLM模拟的动作结果增强网页代理，然而，他们的重点在于数据收集，以训练开放权重的LLM，而我们的研究则集中于利用先进的LLM（如GPT-4o）（OpenAI，[2024a](https://arxiv.org/html/2411.06559v1#bib.bib24)）理解这一新范式的潜力。

## 3 初步

表 1：VisualWebArena（Koh 等，[2024a](https://arxiv.org/html/2411.06559v1#bib.bib16)）中定义的网页导航动作空间。

| 动作类型 $a$ | 描述 |
| --- | --- |
| click [$\mathtt{elem}$] | 点击 $\mathtt{elem}$。 |
| hover [$\mathtt{elem}$] | 将鼠标悬停在 $\mathtt{elem}$ 上。 |
| type [$\mathtt{elem}$] [$\mathtt{text}$] | 在 $\mathtt{elem}$ 中输入 $\mathtt{text}$。 |
| press [$\mathtt{key\_comb}$] | 按下一个键盘组合。 |
| goto [$\mathtt{url}$] | 访问 $\mathtt{url}$。 |
| go_back | 点击返回。 |
| go_forward | 点击前进。 |
| new_tab | 打开一个新标签。 |
| tab_focus [$\mathtt{index}$] | 聚焦于第 i 个标签。 |
| tab_close | 关闭当前标签。 |
| scroll [$\mathtt{up/down}$] | 向上或向下滚动。 |
| stop [$\mathtt{answer}$] | 以输出结束。 |

### 3.1 任务定义

负责自动化实时网站活动的网页代理面临着庞大而复杂的搜索空间。形式化地，每个任务及其任务指令 $I$ 可以被框定为一个部分可观察马尔可夫决策过程（POMDP）：$(\mathcal{S},\mathcal{A},\mathcal{O},T,R,\Omega)$，其中 $\mathcal{S}$ 代表环境中所有可能状态的集合，$\mathcal{A}$ 代表代理可以采取的所有可能动作，$\mathcal{O}$ 代表来自环境的所有可能观察的集合，$T:\mathcal{S}\times\mathcal{A}\rightarrow\mathcal{S}$ 代表状态转移函数，$R$ 是一个二进制奖励，表示任务指令 $I$ 指定的任务是否已完成，$\Omega:\mathcal{S}\rightarrow\mathcal{O}$ 是一个确定性函数，用于将状态映射为观察。任务的目标是执行一系列动作，使奖励达到 1。

在实际场景中，由于网页环境的复杂性，环境是部分可观察的。真实状态包含服务器端变量、动态加载的内容、隐藏的UI元素，并且受网络条件和浏览器限制的影响。因此，智能体只能通过有限的视口（即观察$o\in\mathcal{O}$）感知环境，观察$o$是对真实系统状态的一个不完整投影。观察空间通常表现为截图或基于文本的可访问性树，反映了常见的实现实践。这种受限的可观察性自然地塑造了动作空间$A$，该空间包括在$o$中的可交互元素上可执行的操作，例如点击元素、文本输入和URL导航（表[1](https://arxiv.org/html/2411.06559v1#S3.T1 "表 1 ‣ 3 初步 ‣ 您的LLM是否是互联网的世界模型？基于模型的网页智能体规划")）。

### 3.2 通过仿真进行规划

通过树搜索使用由$T$支配的真实交互来规划最优动作序列是昂贵且有不可逆操作风险的。基于模型的规划通过使用环境的计算表示来模拟交互结果，从而解决了这些挑战。智能体不再在真实环境中执行动作，而是利用近似模型来预测状态转移，从而在没有真实世界交互的情况下高效地探索和评估动作序列。尽管在像BlocksWorld（Hao等人，[2023](https://arxiv.org/html/2411.06559v1#bib.bib12)）这样的确定性环境中，离线规划可以在执行之前计算整个动作序列，但由于网页环境过于复杂，无法进行如此长时间的预测。因此，需要在线规划方法，将规划和执行交替进行，一次计算一个动作。

一种显著的方法是模型预测控制（MPC；Garcia等人（[1989](https://arxiv.org/html/2411.06559v1#bib.bib8)）），它通过迭代模拟未来的轨迹来选择动作。在每个状态$s$下，MPC会为每个可能的动作$a\in\mathcal{A}$，利用模拟器函数$\text{{sim}}(s,a)$模拟在有限时间窗$H$内的轨迹，并通过评分函数$\text{score}(\tau)$对这些轨迹进行评估。最终执行导致最有前景轨迹的动作：$a^{*}=\operatorname*{arg\,max}_{a\in\mathcal{A}}\text{{score}}(\text{{sim}}(s,a))$。该过程在观察到新状态后会重复，从而使智能体能够根据实际结果调整其计划，避免代价高昂的现实世界探索。实际上，由于部分可观察性，我们无法访问真实状态，因此我们改用$\texttt{sim}(o,a)$，其中观察$o=\Omega(s)$。

![参考说明](img/cfcd502df918f1608428262f09c88f14.png)

图2：WebDreamer使用LLM模拟每个候选动作结果的示意图。LLM模拟了三种候选动作的轨迹，并以自然语言描述： (1) 点击“办公产品”，(2) 点击“电子产品”，和 (3) 在文本框中输入“磁盘”。通过这些模拟，评估每个结果轨迹的得分，以识别最有可能成功的动作。在此案例中，LLM选择点击“电子产品”作为最佳步骤并执行它。每个虚线框表示在每次模拟动作后，LLM生成的状态描述。此示例展示了一个两步规划视野。

## 4 WebDreamer：基于模型的网页代理规划

本文提出了WebDreamer，一种利用LLM作为世界模型，在复杂数字环境中实现高效规划的开创性方法。我们的方法的动机来自于这样一个观察：尽管网页界面复杂，但它们是为人类用户设计的，旨在可预测。在浏览网站时，人类可以根据视觉提示和常见设计模式有效地预测动作结果——点击“提交”按钮会提交表单，选择产品图片会导航到产品详情页。考虑到LLM是通过大量与网络相关的数据进行训练的，我们假设它们已经获得了足够的知识来模拟用户操作的后果，可能作为有效的世界模型进行规划。

### 4.1 核心设计

WebDreamer遵循在第[3.2](https://arxiv.org/html/2411.06559v1#S3.SS2 "3.2 Planning through Simulation ‣ 3 Preliminary ‣ Is Your LLM Secretly a World Model of the Internet? Model-Based Planning for Web Agents")节中介绍的通过模拟进行规划的范式。图[2](https://arxiv.org/html/2411.06559v1#S3.F2 "Figure 2 ‣ 3.2 Planning through Simulation ‣ 3 Preliminary ‣ Is Your LLM Secretly a World Model of the Internet? Model-Based Planning for Web Agents")通过三个候选动作来说明这一过程，其中WebDreamer为每个动作模拟了两步轨迹，选择得分最高的轨迹，并执行其相应的初始动作。WebDreamer的核心是利用LLM来实现模拟功能sim和评分功能score。

sim的实现：我们对sim的实现由两个模块组成：一个模块在执行动作后预测状态变化，近似$T$，另一个模块根据预测的状态想象可能的动作。两个模块共同生成长度为$H$的轨迹，其中$H$是一个可配置的时间跨度参数（即，模拟深度）。具体来说，为了表示状态变化，我们提示LLM生成一个简洁的自然语言描述，重点仅放在动作的影响上。例如，在图[2](https://arxiv.org/html/2411.06559v1#S3.F2 "图2 ‣ 3.2 通过模拟进行规划 ‣ 3 初步 ‣ 你的LLM是否是互联网的世界模型？基于模型的网页代理规划")中，当提示LLM预测执行“点击‘电子产品’”动作的效果时，LLM将输出以下简短描述：

<svg class="ltx_picture" height="76.19" id="S4.SS1.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,76.19) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 60.4)"><foreignobject color="#FFFFFF" height="9.89" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">$\rightarrow$ Click “Electronics”</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="28.9" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">The ‘Electronics’ category will display three sub-categories: ‘Computers & Accessories’, ‘Accessories & Supplies’, and ‘Car & Vehicle Electronics’.</foreignobject></g></g></svg>

基于此预测状态，LLM随后想象下一个动作（即，点击“计算机与配件”），这将导致另一个状态变化的预测。这个过程生成了一个时长为$H=2$的轨迹。

score的实现：在使用sim从每个候选动作$a_{i}$收集到一个轨迹$\tau_{i}$后，我们进一步将LLM作为每次模拟的评分函数。根据Koh等人（[2024b](https://arxiv.org/html/2411.06559v1#bib.bib17)）的方法，我们提示LLM对每个模拟轨迹进行三尺度评分——完成（1.0）、在轨（0.5）或错误（0）——以表示其朝任务完成的进展。最终得分通过对这些评估的多个样本进行平均计算得到。

输入：指令$I$；初始观察$o_{0}$输出：动作序列$a_{0},a_{1},\ldots,a_{T}$ $t\leftarrow 0$；while *True* do    $\mathcal{A}_{t}\leftarrow\text{{get\_candidate}}(I,o_{t})$；    $\mathcal{A}_{t}^{\prime}\leftarrow\text{{self\_refine}}(\mathcal{A}_{t})$；    $a_{t}=\operatorname*{arg\,max}_{a\in\mathcal{A}_{t}^{\prime}}\text{{score}}(\% \text{{sim}}(o_{t},a))$；    $o_{t+1}\leftarrow\texttt{execute}(a_{t})$；    $t\leftarrow t+1$；    if *termination_check() = True* then        break；    end ifend while返回结果；

算法1 WebDreamer

除了模拟和评分，规划的前提是候选行动的生成。我们采用两阶段的方法：首先按照Koh等人（[2024b](https://arxiv.org/html/2411.06559v1#bib.bib17)）的方法采样前k个行动，然后使用LLM自我优化不必要的行动用于模拟。这个自我优化步骤的动机来自我们观察到的一个现象：在不同步骤中，相同的k值可能引入不同程度的无关行动——某些步骤自然会有比其他步骤更少的合理行动。我们在算法[1](https://arxiv.org/html/2411.06559v1#algorithm1 "In 4.1 Core Design ‣ 4 WebDreamer: Model-Based Planning for Web Agents ‣ Is Your LLM Secretly a World Model of the Internet? Model-Based Planning for Web Agents")中展示了WebDreamer整体设计的伪代码。termination_check验证模型是否输出停止动作，是否达到最大步数，或者是否重复执行某个动作超过3次，同样遵循Koh等人（[2024b](https://arxiv.org/html/2411.06559v1#bib.bib17)）的实现方法。

WebDreamer中使用的所有系统提示可以在附录[A](https://arxiv.org/html/2411.06559v1#A1 "Appendix A Prompts for Four Stages in MPC-Based Planning ‣ Is Your LLM Secretly a World Model of the Internet? Model-Based Planning for Web Agents")中找到。

### 4.2 讨论

为了证明我们的设计选择符合我们的目标——使用LLM作为网页环境世界模型的开创性研究——我们讨论了三个关键考虑因素：

+   状态变化描述而非HTML/可访问性树。

    尽管我们使用自然语言描述来捕捉状态变化，但另一种选择是提示LLM预测结果页面的HTML或可访问性树。然而，由于大多数网页元素在一个动作后保持不变，因此预测整个页面结构是不必要的浪费。此外，这种具体的预测更容易出现幻觉——HTML需要关于网站的精确细节，而状态描述只需捕捉必要的变化。对于我们的开创性研究，我们采用了这种更简单、更直观的表示方式，尽管我们并未主张它在严格意义上优于HTML或可访问性树（请参见第[6.1](https://arxiv.org/html/2411.06559v1#S6.SS1 "6.1 State Representation And Planning Horizon ‣ 6 Analyses ‣ Is Your LLM Secretly a World Model of the Internet? Model-Based Planning for Web Agents")节进行详细分析）。

+   提示而非微调。

    在本研究中，我们通过直接提示最先进的LLM（即GPT-4o（OpenAI，[2024a](https://arxiv.org/html/2411.06559v1#bib.bib24)））来实现WebDreamer，而不进行微调。我们的理由很简单：我们旨在首先建立使用先进的LLM作为网页环境世界模型的可行性，以及它们在规划中的有效性。通过这种方法展示有希望的结果，将为未来通过在特定数据集上微调开源模型来优化该方向奠定基础。

+   采用直接的基于MPC的规划，而非MCTS。

    我们采用了一个相对直接的基于MPC的规划算法，而不是像MCTS这样在近期LLM规划研究中突出的更复杂的方法（Hao等人，[2023](https://arxiv.org/html/2411.06559v1#bib.bib12)；Feng等人，[2023](https://arxiv.org/html/2411.06559v1#bib.bib6)）。这一选择的动机来自我们的实证研究结果：增加WebDreamer的规划视野带来的回报逐渐递减，这表明LLM在准确建模多步骤轨迹方面的当前局限性（参见第[6.1](https://arxiv.org/html/2411.06559v1#S6.SS1 "6.1 状态表示与规划视野 ‣ 6 分析 ‣ 您的LLM是否秘密充当互联网的世界模型？基于模型的Web代理规划")节）。鉴于我们探索LLM作为Web环境的世界模型的目标，这种更简单的方法足以展示关键洞察，同时承认LLM的当前能力。

## 5 实验

### 5.1 设置

为了正确测试我们的规划框架在实际中的表现，我们使用具有在线评估的基准，捕捉Web交互的动态特性。我们关注两个具有代表性的基准：VisualWebArena (VWA; Koh等人（[2024a](https://arxiv.org/html/2411.06559v1#bib.bib16)）)，强调多模态设置，以及Mind2Web-live (Pan等人，[2024b](https://arxiv.org/html/2411.06559v1#bib.bib27))，其默认使用HTML。VWA包括三个本地托管网站上的910个任务：购物、分类广告和Reddit。相比之下，Mind2Web-live包括来自69个真实网站的104个任务。我们遵循两个基准的默认设置：对于VWA，我们使用带有Set-of-Marks提示的屏幕截图作为观察空间；对于Mind2Web-live，我们使用HTML。对于我们的LLM，我们选择最先进的多模态LLM GPT-4o，因为它最符合我们以LLM为基础的模型规划进行开创性探索，并探索这一设想范式的全部潜力。在我们的实验中，我们将规划视野$H$ empirically设定为1。关于该参数的全面分析见第[6.1](https://arxiv.org/html/2411.06559v1#S6.SS1 "6.1 状态表示与规划视野 ‣ 6 分析 ‣ 您的LLM是否秘密充当互联网的世界模型？基于模型的Web代理规划")节。

为了展示我们提案的有效性，我们主要将我们的方法与两个主要基准进行比较：反应性代理和具有真实交互的树搜索代理。²²2在我们的实验中，出于简洁性考虑，我们将具有真实交互的树搜索简称为树搜索。虽然我们可以很容易地为这两个基准实现我们自己的方法，但对于树搜索基准（Koh等，[2024b](https://arxiv.org/html/2411.06559v1#bib.bib17)），由于在Mind2Web-live中的真实网站上进行树搜索不可行，我们只能在VWA上进行比较。具体来说，在VWA中，Koh等人（[2024b](https://arxiv.org/html/2411.06559v1#bib.bib17)）跟踪到达先前轨迹中状态的动作序列。在回溯过程中，他们重置沙箱并重新执行动作序列以恢复状态。然而，在Mind2Web-live中，重置环境以撤销效果在现实世界的网站上并不总是可行的。

### 5.2 主要结果

表2：在VisualWebArena和Mind2Web-live上的结果。WebDreamer显著超越了反应性基准，并且在VWA上仅稍逊于树搜索基准，同时要求的网页交互明显更少。对于Mind2Web-live，由于需要网站回溯，实现树搜索算法面临重大挑战，这使得我们省略了树搜索的性能指标。这一局限性进一步凸显了我们基于模型的规划方法的灵活性。我们还包括了额外的基准（由灰色单元格表示），以提供更广泛的背景。虽然这些比较可能不会直接评估我们的核心假设，但它们为理解我们方法在网页导航领域的表现提供了有价值的背景。^†我们自己在VWA上运行反应性基准，因为本地托管要求可能导致硬件相关的性能变化。

| 基准 | 观测 $\mathcal{O}$ | 方法 | 完成率 | 成功率 |
| --- | --- | --- | --- | --- |
| VisualWebArena | Screenshot+SoM | Gemini-1.5-Pro + 反应性 (Koh等，[2024a](https://arxiv.org/html/2411.06559v1#bib.bib16)) | - | 12.0% |
| GPT-4 + 反应性 (Koh等，[2024a](https://arxiv.org/html/2411.06559v1#bib.bib16)) | - | 16.4% |
| GPT-4o + 反应性 (Koh等，[2024a](https://arxiv.org/html/2411.06559v1#bib.bib16)) | - | 17.7%^† |
| GPT-4o + 树搜索 (Koh等，[2024b](https://arxiv.org/html/2411.06559v1#bib.bib17)) | - | 26.4% |
| GPT-4o + WebDreamer | - | 23.6% (\faArrowUp33.3%) |
| Mind2Web-live | HTML | GPT-4 + 反应性 (Pan等，[2024b](https://arxiv.org/html/2411.06559v1#bib.bib27)) | 48.8% | 23.1% |
| Claude-3-Sonnet + 反应性 (Pan等，[2024b](https://arxiv.org/html/2411.06559v1#bib.bib27)) | 47.9% | 22.1% |
| Gemini-1.5-Pro + 反应性 (Pan等，[2024b](https://arxiv.org/html/2411.06559v1#bib.bib27)) | 44.6% | 22.3% |
| GPT-4-turbo + 反应性 (Pan等，[2024b](https://arxiv.org/html/2411.06559v1#bib.bib27)) | 44.3% | 21.1% |
| GPT-3.5-turbo + Reactive (Pan et al., [2024b](https://arxiv.org/html/2411.06559v1#bib.bib27)) | 40.2% | 16.5% |
| GPT-4o + Reactive (Pan et al., [2024b](https://arxiv.org/html/2411.06559v1#bib.bib27)) | 47.6% | 22.1% |
| GPT-4o + WebDreamer | 49.9% | 25.0% (\faArrowUp13.1%) |

+   效果性。

    我们在表[2](https://arxiv.org/html/2411.06559v1#S5.T2 "表 2 ‣ 5.2 主要结果 ‣ 5 实验 ‣ 您的 LLM 是否是互联网的世界模型？基于模型的网页代理规划")中展示了整体性能结果。WebDreamer 在 VWA 和 Mind2Web-live 数据集上相比反应式代理展示了显著的改进。值得注意的是，在 VWA 数据集上，我们提出的方法实现了 33.3% 的相对性能提升。同时，我们的方法在整体成功率上仍然未能超越树搜索基准。尽管取得了这些进展，我们的方法在整体成功率上仍不及树搜索基准。然而，必须强调的是，树搜索对于现实世界中的网站并不可行，而 WebDreamer 提供了更灵活和适应性强的替代方案。在 Mind2Web-live 数据集上，WebDreamer 相比反应式基准提高了 2.9%（相对提升 13.1%），这一改进不如在 VWA 数据集上的提升显著。然而，值得注意的是，Mind2Web-live 数据集的判别能力较弱，正如表[2](https://arxiv.org/html/2411.06559v1#S5.T2 "表 2 ‣ 5.2 主要结果 ‣ 5 实验 ‣ 您的 LLM 是否是互联网的世界模型？基于模型的网页代理规划")中展示的多个基础 LLM 的性能差异较小所证明。VWA 和 Mind2Web-live 数据集上强劲的结果表明我们方法在不同观察设置下的有效性。

我们进一步进行了更为细致的分析，将我们提出的方法与 VWA 数据集上反应式基准进行了多维度的比较。表[3](https://arxiv.org/html/2411.06559v1#S5.T3.4 "表 3 ‣ 5.2 主要结果 ‣ 5 实验 ‣ 您的 LLM 是否是互联网的世界模型？基于模型的网页代理规划")展示了我们基于模型的规划方法在所有网站和任务难度级别上始终优于反应式基准。在 VWA 官方注释的中等难度任务中，基于模型的规划甚至超过了树搜索的表现（即 22.2% 对 24.1%）。尽管前景看好，基于模型的规划仍在 VWA 中面对需要多步模拟的难题时遇到挑战。随着步数的增加，模拟的准确性下降，这对处理困难任务提出了重大挑战。

表 3：基于不同维度的成功率分解。$\gamma=\frac{SR_{\text{{WebDreamer}}}-SR_{\text{reactive}}}{SR_{\text{tree % search}}-SR_{\text{reactive}}}$ 衡量了 WebDreamer 缩小反应式代理与树搜索代理之间差距的程度。

(a) 网站

| 网站 | 反应式 | 树搜索 | WebDreamer | $\gamma$ |
| --- | --- | --- | --- | --- |
| 分类广告 | 16.8% | 26.5% | 22.6% | 59.8% |
| Reddit | 15.3% | 20.5% | 18.6% | 63.5% |
| 购物 | 19.4% | 29.0% | 26.5% | 74.0% |

(b) 任务难度

| 难度 | 反应式 | 树搜索 | WebDreamer | $\gamma$ |
| --- | --- | --- | --- | --- |
| 简单 | 28.8% | 42.3% | 37.4% | 63.7% |
| 中等 | 16.4% | 22.2% | 24.1% | 132.8% |
| 困难 | 10.7% | 14.9% | 12.7% | 47.6% |

+   效率。

    基于模型的规划的另一个关键优势是与实际探索使用树搜索相比，其效率更高。如[表4](https://arxiv.org/html/2411.06559v1#S5.T4.fig2 "Table 4 ‣ 5.2 Main Results ‣ 5 Experiments ‣ Is Your LLM Secretly a World Model of the Internet? Model-Based Planning for Web Agents")所示，树搜索在所有环境中大约需要比基准多三倍的步骤，而我们的方法保持了类似的行动步骤。值得注意的是，树搜索由于额外的动作和回溯，导致了大约十倍的墙钟延迟，而我们方法中的仿真开销最小，并且通过增加并行化可以进一步减少。

表4：VWA上的行动步骤和墙钟时间。

(a) 行动步骤数

| 步骤 | 反应式 | 树搜索 | WebDreamer |
| --- | --- | --- | --- |
| 分类广告 | 3.4 | 9.9 | 4.1 |
| Reddit | 5.1 | 13.6 | 5.2 |
| 购物 | 4.5 | 11.4 | 4.5 |

(b) 任务完成墙钟时间

| 秒数 | 反应式 | 树搜索 | WebDreamer |
| --- | --- | --- | --- |
| 分类广告 | 68.3 | 749.2 | 183.6 |
| Reddit | 83.5 | 972.1 | 233.7 |
| 购物 | 87.7 | 785.7 | 179.4 |

## 6 分析

### 6.1 状态表示与规划视野

我们的基于模型的规划方法依赖于两个关键维度进行仿真：状态表示和规划视野（即仿真深度）。为了更深入地了解其有效性和局限性，我们研究了不同配置如何影响最终表现。鉴于这些实验的高计算成本，我们使用VWA数据集的一个子集进行分析，该子集包括100个购物任务和官方注释的人类轨迹。

除了我们主要实验中使用的状态变化描述，我们还探索了其他方法，在这些方法中，GPT-4o 会预测模拟中生成网页的 HTML 代码或可访问性树。对于这些状态表示中的每一种，我们评估了 1、2 和 3 步的规划时间范围。如图[4](https://arxiv.org/html/2411.06559v1#S6.F4 "图 4 ‣ 6.1 状态表示与规划时间范围 ‣ 6 分析 ‣ 你的 LLM 是否秘密成为了互联网的世界模型？基于模型的网页代理规划")所示，这三种状态表示显著优于反应性基线。然而，随着规划时间范围扩展到 3 步，它们的效果有所下降，这表明在这些方法中，长时间模拟存在一个共同的局限性。具体而言，在模拟中的行动提议往往会产生任务完成所需的相关行动的幻觉，即使这些行动在 LLM 预测的当前状态中并不存在。值得注意的是，状态变化表示随着规划时间范围的延长表现出最明显的性能下降。这一下降在规划时间范围为 3 时尤为严重，此时其性能低于反应性基线。这一脆弱性源于它对当前网页上可用交互元素的隐式说明，要求模型通过对初始状态进行更改来推断这些元素。相比之下，HTML 和可访问性树表示提供了明确的元素信息。

![参见说明](img/063a3a79401b23af63e25b8a7045558c.png)

图 3：关于模拟阶段和自我精炼阶段的消融研究。

因此，状态变化方法在长时间模拟过程中更容易出现幻觉。尽管有这一局限性，考虑到当前大语言模型（LLM）的能力，状态变化方法仍然是一个可行的选择。对于规划时间范围小于 3 的情况，它的表现与 HTML 和可访问性树表示相匹配，同时消耗的输出令牌更少。

![参见说明](img/6c848ec37af4d667ec256db3984f7b17.png)

图 4：我们展示了在 VWA 数据集子集上的表现，变化了模拟中的状态表示和规划时间范围。无论使用哪种状态表示，长时间规划的模拟仍然是一个挑战。

### 6.2 消融研究

为了确定观察到的改进是否来自我们基于模型的规划方法的特定部分，我们对仿真和自我优化阶段进行了消融研究，使用了来自[6.1节](https://arxiv.org/html/2411.06559v1#S6.SS1 "6.1 State Representation And Planning Horizon ‣ 6 Analyses ‣ Is Your LLM Secretly a World Model of the Internet? Model-Based Planning for Web Agents")的相同子集。我们特别关注仿真阶段，它是基于模型的规划的核心。有人可能认为，主要的改进来自于重新排序候选动作，而不管这一排序是否依赖于仿真。为了验证这一想法，我们进行了一项实验，完全移除了仿真阶段，而是让奖励模型直接评估每个候选动作。如图[3](https://arxiv.org/html/2411.06559v1#S6.F3 "Figure 3 ‣ 6.1 State Representation And Planning Horizon ‣ 6 Analyses ‣ Is Your LLM Secretly a World Model of the Internet? Model-Based Planning for Web Agents")所示，这种修改后的重新排序方法确实在反应性基线方法上取得了一些改善，但增益很小，仍然远远落后于WebDreamer。这些结果证实了基于LLM的世界模型仿真在规划过程中的关键作用。此外，我们还观察到，当移除自我优化阶段时，性能出现下降。经过仔细检查，我们发现这种下降主要是由于自我优化模块能够有效地过滤掉那些在下一步最佳动作明确时不太相关的候选动作。相比之下，直接对所有动作进行仿真可能会引入额外的噪音，进而负面影响性能。

### 6.3 案例研究

为了澄清仿真在规划中的作用，我们呈现了一个案例研究，涵盖了正面和负面的示例。这说明了仿真如何帮助智能体探索环境，以及仿真中的不准确性如何导致错误的预测。详细的示例请参见附录[B](https://arxiv.org/html/2411.06559v1#A2 "Appendix B Case Study ‣ Is Your LLM Secretly a World Model of the Internet? Model-Based Planning for Web Agents")。

## 7 结论

在本文中，我们展示了使用大语言模型（LLMs）作为世界模型来支持复杂环境中的规划的巨大潜力。具体而言，我们基于模型的规划方法WebDreamer在反应性基线方法上表现出了显著的改进，并且比树搜索提供了更大的灵活性，而树搜索在现实世界的网站中通常是不可行的。作为该领域的开创性工作，我们的研究为基于模型的规划和LLM模拟的世界模型开辟了新的方向。未来的研究可以集中在进一步优化LLM作为复杂环境中的世界模型，并开发更强大的基于模型的规划算法，以应对长时程规划。

## 局限性

作为一项关于基于MPC的规划与LLM结合的网页导航开创性探索，我们的研究自然存在一些局限性，这些局限性也是未来研究的激动人心的方向：

+   规划算法的简洁性。

    在这项初步工作中，我们故意使用了一个简单的规划算法，以展示我们方法的核心潜力。虽然这种方法有效，但其简洁性仍留有充分的空间用于未来的改进。更复杂的规划技术，如蒙特卡洛树搜索（MCTS），可以集成进来，进一步提升性能。作为一项基础研究，我们的重点是确立这一概念的可行性，而非优化系统的各个方面。这一战略选择为未来的研究提供了基础，可以在我们已建立的框架内进一步探索更先进的规划策略。

+   计算成本。

    我们当前的实现，利用像GPT-4o这样的最先进模型，产生了不容忽视的API成本（每个任务大约$1，使用VWA）。这一成本反映了我们在没有立即约束的情况下探索LLM基础规划的全部潜力。对于实际应用，未来的工作可以探索成本效益高的替代方案，如为仿真任务微调专用模型。这为未来的优化设定了一个平衡性能与效率的基准。

这些局限性突显了我们工作作为概念验证的性质，为未来的研究和优化开辟了众多途径。通过建立基于MPC的规划与LLM结合的基础潜力，我们为基于LLM的语言代理的新规划范式奠定了基础，邀请进一步的创新，以完善和扩展基于模型的规划。

## 致谢

我们要感谢来自OSU NLP小组和Orby AI的同事们提供的富有洞察力的意见。此项工作部分得到了Orby AI和ARL W911NF2220144的支持。文中所表达的观点和结论仅代表作者个人意见，不应解释为美国政府的官方政策，无论是明示还是暗示。美国政府有权为政府用途复制和分发重印本，尽管本文中可能包含版权声明。

## 参考文献

+   Baechler 等人（2024） Gilles Baechler、Srinivas Sunkara、Maria Wang、Fedir Zubach、Hassan Mansoor、Vincent Etter、Victor Cărbune、Jason Lin、Jindong Chen 和 Abhanshu Sharma。《Screenai：一个用于UI和信息图理解的视觉-语言模型》。*ArXiv 预印本*，abs/2402.04615，2024。网址 [https://arxiv.org/abs/2402.04615](https://arxiv.org/abs/2402.04615)。

+   Brown 等人（2024） Bradley Brown、Jordan Juravsky、Ryan Ehrlich、Ronald Clark、Quoc V Le、Christopher Ré 和 Azalia Mirhoseini。《大型语言猴子：通过重复采样扩展推理计算》。*ArXiv 预印本*，abs/2407.21787，2024。网址 [https://arxiv.org/abs/2407.21787](https://arxiv.org/abs/2407.21787)。

+   Chae等人（2024）Hyungjoo Chae, Namyoung Kim, Kai Tzu-iunn Ong, Minju Gwak, Gwanwoo Song, Jihoon Kim, Sunghwan Kim, Dongha Lee 和 Jinyoung Yeo。带有世界模型的网络代理：在网页导航中学习并利用环境动态。*arXiv预印本arXiv:2410.13232*, 2024。

+   Cheng等人（2024）Kanzhi Cheng, Qiushi Sun, Yougang Chu, Fangzhi Xu, Li YanTao, Jianbing Zhang 和 Zhiyong Wu。SeeClick：利用GUI的基础进行高级视觉GUI代理。在*第62届计算语言学协会年会论文集（第一卷：长篇论文）*中，pp. 9313–9332，泰国曼谷，2024年。计算语言学协会。网址 [https://aclanthology.org/2024.acl-long.505](https://aclanthology.org/2024.acl-long.505)。

+   Deng等人（2023）Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Samual Stevens, Boshi Wang, Huan Sun 和 Yu Su。Mind2web：迈向一个通用型网络代理。在Alice Oh, Tristan Naumann, Amir Globerson, Kate Saenko, Moritz Hardt 和 Sergey Levine（编），*神经信息处理系统进展 36：2023年神经信息处理系统年会，NeurIPS 2023，美国路易斯安那州新奥尔良，2023年12月10日至16日*中，2023年。网址 [http://papers.nips.cc/paper_files/paper/2023/hash/5950bf290a1570ea401bf98882128160-Abstract-Datasets_and_Benchmarks.html](http://papers.nips.cc/paper_files/paper/2023/hash/5950bf290a1570ea401bf98882128160-Abstract-Datasets_and_Benchmarks.html)。

+   Feng等人（2023）Xidong Feng, Ziyu Wan, Muning Wen, Stephen Marcus McAleer, Ying Wen, Weinan Zhang 和 Jun Wang。类似AlphaZero的树搜索可以指导大型语言模型的解码与训练。*ArXiv预印本*, abs/2309.17179, 2023。网址 [https://arxiv.org/abs/2309.17179](https://arxiv.org/abs/2309.17179)。

+   Furuta等人（2023）Hiroki Furuta, Kuang-Huei Lee, Ofir Nachum, Yutaka Matsuo, Aleksandra Faust, Shixiang Shane Gu 和 Izzeddin Gur。通过指令微调基础模型实现多模态网页导航。*ArXiv预印本*, abs/2305.11854, 2023。网址 [https://arxiv.org/abs/2305.11854](https://arxiv.org/abs/2305.11854)。

+   Garcia等人（1989）Carlos E Garcia, David M Prett 和 Manfred Morari。模型预测控制：理论与实践——一项调查。*Automatica*, 25(3):335–348, 1989。

+   Gu等人（2023）Yu Gu, Xiang Deng 和 Yu Su。不要生成，而是区分：一种将语言模型与真实世界环境结合的提案。在*第61届计算语言学协会年会论文集（第一卷：长篇论文）*中，pp. 4928–4949，加拿大多伦多，2023年7月。计算语言学协会。doi: 10.18653/v1/2023.acl-long.270。网址 [https://aclanthology.org/2023.acl-long.270](https://aclanthology.org/2023.acl-long.270)。

+   Gur等人（2023）Izzeddin Gur、Hiroki Furuta、Austin Huang、Mustafa Safdari、Yutaka Matsuo、Douglas Eck和Aleksandra Faust。一个具备规划、长时上下文理解和程序合成的现实世界网页代理。*ArXiv预印本*，abs/2307.12856，2023。网址 [https://arxiv.org/abs/2307.12856](https://arxiv.org/abs/2307.12856)。

+   Ha & Schmidhuber（2018）David Ha和Jürgen Schmidhuber。世界模型。*ArXiv预印本*，abs/1803.10122，2018。网址 [https://arxiv.org/abs/1803.10122](https://arxiv.org/abs/1803.10122)。

+   Hao等人（2023）Shibo Hao、Yi Gu、Haodi Ma、Joshua Hong、Zhen Wang、Daisy Wang和Zhiting Hu。通过语言模型推理即通过世界模型规划。发表于Houda Bouamor、Juan Pino和Kalika Bali（编辑），*2023年自然语言处理实证方法会议论文集*，第8154-8173页，新加坡，2023。计算语言学协会。doi: 10.18653/v1/2023.emnlp-main.507。网址 [https://aclanthology.org/2023.emnlp-main.507](https://aclanthology.org/2023.emnlp-main.507)。

+   He等人（2024）Hongliang He、Wenlin Yao、Kaixin Ma、Wenhao Yu、Yong Dai、Hongming Zhang、Zhenzhong Lan和Dong Yu。Webvoyager：构建基于大型多模态模型的端到端网页代理。*ArXiv预印本*，abs/2401.13919，2024。网址 [https://arxiv.org/abs/2401.13919](https://arxiv.org/abs/2401.13919)。

+   Hong等人（2024）Wenyi Hong、Weihan Wang、Qingsong Lv、Jiazheng Xu、Wenmeng Yu、Junhui Ji、Yan Wang、Zihan Wang、Yuxiao Dong、Ming Ding和Jie Tang。Cogagent：用于GUI代理的视觉语言模型。发表于*IEEE/CVF计算机视觉与模式识别会议论文集*，第14281-14290页，2024。

+   Kim等人（2024）Doyoung Kim、Jongwon Lee、Jinho Park和Minjoon Seo。语言模型的认知地图：通过口头表示世界模型实现最优规划。*ArXiv预印本*，abs/2406.15275，2024。网址 [https://arxiv.org/abs/2406.15275](https://arxiv.org/abs/2406.15275)。

+   Koh等人（2024a）Jing Yu Koh、Robert Lo、Lawrence Jang、Vikram Duvvur、Ming Chong Lim、Po-Yu Huang、Graham Neubig、Shuyan Zhou、Ruslan Salakhutdinov和Daniel Fried。Visualwebarena：在现实的视觉网页任务中评估多模态代理。*ArXiv预印本*，abs/2401.13649，2024a。网址 [https://arxiv.org/abs/2401.13649](https://arxiv.org/abs/2401.13649)。

+   Koh等人（2024b）Jing Yu Koh、Stephen McAleer、Daniel Fried和Ruslan Salakhutdinov。语言模型代理的树搜索。*ArXiv预印本*，abs/2407.01476，2024b。网址 [https://arxiv.org/abs/2407.01476](https://arxiv.org/abs/2407.01476)。

+   Lai等人（2024）Hanyu Lai、Xiao Liu、Iat Long Iong、Shuntian Yao、Yuxuan Chen、Pengbo Shen、Hao Yu、Hanchen Zhang、Xiaohan Zhang、Yuxiao Dong和Jie Tang。Autowebglm：基于大型语言模型的网页导航代理。发表于*第30届ACM SIGKDD知识发现与数据挖掘会议论文集*，第5295-5306页，2024。

+   Lee et al. (2023) Kenton Lee, Mandar Joshi, Iulia Raluca Turc, Hexiang Hu, Fangyu Liu, Julian Martin Eisenschlos, Urvashi Khandelwal, Peter Shaw, Ming-Wei Chang, and Kristina Toutanova. Pix2struct: 截图解析作为视觉语言理解的预训练。在Andreas Krause、Emma Brunskill、Kyunghyun Cho、Barbara Engelhardt、Sivan Sabato和Jonathan Scarlett（编辑）主编的《*国际机器学习大会（ICML 2023），2023年7月23日至29日，夏威夷檀香山，美国*》，*机器学习研究论文集*第202卷，18893–18912页。PMLR，2023。网址 [https://proceedings.mlr.press/v202/lee23g.html](https://proceedings.mlr.press/v202/lee23g.html)。

+   Liao et al. (2024) Zeyi Liao, Lingbo Mo, Chejian Xu, Mintong Kang, Jiawei Zhang, Chaowei Xiao, Yuan Tian, Bo Li, and Huan Sun. EIA: 一种针对通用网页代理的环境注入攻击以泄露隐私。*CoRR*, abs/2409.11295, 2024。doi: 10.48550/ARXIV.2409.11295。网址 [https://doi.org/10.48550/arXiv.2409.11295](https://doi.org/10.48550/arXiv.2409.11295)。

+   Liu et al. (2018) Evan Zheran Liu, Kelvin Guu, Panupong Pasupat, Tianlin Shi, and Percy Liang. 基于工作流引导探索的网页界面强化学习。在《*第六届国际学习表示会议（ICLR 2018），加拿大温哥华，2018年4月30日至5月3日，会议论文集*》。OpenReview.net，2018。网址 [https://openreview.net/forum?id=ryTp3f-0-](https://openreview.net/forum?id=ryTp3f-0-)。

+   Mattar & Lengyel (2022) Marcelo G Mattar 和 Máté Lengyel. 大脑中的规划。*Neuron*，110(6):914–934，2022。

+   Moerland et al. (2023) Thomas M Moerland, Joost Broekens, Aske Plaat, Catholijn M Jonker, 等。基于模型的强化学习：一项综述。*机器学习基础与趋势*，16(1):1–118，2023。

+   OpenAI (2024a) OpenAI. 你好，GPT-4o。[https://openai.com/index/hello-gpt-4o/](https://openai.com/index/hello-gpt-4o/)，2024a。访问时间：2024-09-28。

+   OpenAI (2024b) OpenAI. 介绍 OpenAI o1。[https://openai.com/o1/](https://openai.com/o1/)，2024b。访问时间：2024-09-29。

+   Pan et al. (2024a) Jiayi Pan, Yichi Zhang, Nicholas Tomlin, Yifei Zhou, Sergey Levine, and Alane Suhr. 数字代理的自主评估与优化。*ArXiv 预印本*，abs/2404.06474, 2024a。网址 [https://arxiv.org/abs/2404.06474](https://arxiv.org/abs/2404.06474)。

+   Pan et al. (2024b) Yichen Pan, Dehan Kong, Sida Zhou, Cheng Cui, Yifei Leng, Bing Jiang, Hangyu Liu, Yanyi Shang, Shuyan Zhou, Tongshuang Wu, and Zhengyang Wu. Webcanvas：在在线环境中基准测试网页代理。*ArXiv 预印本*，abs/2406.12373，2024b。网址 [https://arxiv.org/abs/2406.12373](https://arxiv.org/abs/2406.12373)。

+   Pascanu 等人 (2017) Razvan Pascanu, Yujia Li, Oriol Vinyals, Nicolas Heess, Lars Buesing, Sebastien Racanière, David Reichert, Théophane Weber, Daan Wierstra, 和 Peter Battaglia. 从零开始学习基于模型的规划. *ArXiv 预印本*, abs/1707.06170, 2017. URL [https://arxiv.org/abs/1707.06170](https://arxiv.org/abs/1707.06170).

+   Patel 等人 (2024) Ajay Patel, Markus Hofmarcher, Claudiu Leoveanu-Condrei, Marius-Constantin Dinu, Chris Callison-Burch, 和 Sepp Hochreiter. 大型语言模型可以在网络代理任务中自我提升. *ArXiv 预印本*, abs/2405.20309, 2024. URL [https://arxiv.org/abs/2405.20309](https://arxiv.org/abs/2405.20309).

+   Putta 等人 (2024) Pranav Putta, Edmund Mills, Naman Garg, Sumeet Motwani, Chelsea Finn, Divyansh Garg, 和 Rafael Rafailov. Agent q: 自主 AI 代理的高级推理与学习. *ArXiv 预印本*, abs/2408.07199, 2024. URL [https://arxiv.org/abs/2408.07199](https://arxiv.org/abs/2408.07199).

+   Schrittwieser 等人 (2020) Julian Schrittwieser, Ioannis Antonoglou, Thomas Hubert, Karen Simonyan, Laurent Sifre, Simon Schmitt, Arthur Guez, Edward Lockhart, Demis Hassabis, Thore Graepel, 等. 通过使用学习模型规划掌握 Atari、围棋、国际象棋和将棋. *自然*, 588(7839):604–609, 2020.

+   Shaw 等人 (2023) Peter Shaw, Mandar Joshi, James Cohan, Jonathan Berant, Panupong Pasupat, Hexiang Hu, Urvashi Khandelwal, Kenton Lee, 和 Kristina Toutanova. 从像素到 UI 操作: 通过图形用户界面学习遵循指令. 在 Alice Oh, Tristan Naumann, Amir Globerson, Kate Saenko, Moritz Hardt, 和 Sergey Levine (编辑), *神经信息处理系统进展 36: 2023年神经信息处理系统年会, NeurIPS 2023, 新奥尔良, 路易斯安那州, 美国, 2023年12月10日至16日*, 2023. URL [http://papers.nips.cc/paper_files/paper/2023/hash/6c52a8a4fadc9129c6e1d1745f2dfd0f-Abstract-Conference.html](http://papers.nips.cc/paper_files/paper/2023/hash/6c52a8a4fadc9129c6e1d1745f2dfd0f-Abstract-Conference.html).

+   Shi 等人 (2017) Tianlin Shi, Andrej Karpathy, Linxi Fan, Jonathan Hernandez, 和 Percy Liang. 位世界: 一个开放域平台用于基于网络的代理. 在 Doina Precup 和 Yee Whye Teh (编辑), *第34届国际机器学习大会论文集, ICML 2017, 悉尼, 新南威尔士州, 澳大利亚, 2017年8月6-11日*, 机器学习研究论文集第70卷, 第3135–3144页. PMLR, 2017. URL [http://proceedings.mlr.press/v70/shi17a.html](http://proceedings.mlr.press/v70/shi17a.html).

+   Silver 等人 (2016) David Silver, Aja Huang, Chris J Maddison, Arthur Guez, Laurent Sifre, George Van Den Driessche, Julian Schrittwieser, Ioannis Antonoglou, Veda Panneershelvam, Marc Lanctot, 等. 使用深度神经网络和树搜索掌握围棋游戏. *自然*, 529(7587):484–489, 2016.

+   Song et al. (2024) Yifan Song, Da Yin, Xiang Yue, Jie Huang, Sujian Li, and Bill Yuchen Lin. 试错法：基于探索的轨迹优化用于LLM代理。*ArXiv预印本*，abs/2403.02502，2024。网址 [https://arxiv.org/abs/2403.02502](https://arxiv.org/abs/2403.02502)。

+   Sutton (1991) Richard S Sutton. Dyna：一种集成的学习、规划与反应架构。*ACM Sigart Bulletin*，2(4)：160–163，1991。

+   Wang et al. (2024) Evan Wang, Federico Cassano, Catherine Wu, Yunfeng Bai, Will Song, Vaskar Nath, Ziwen Han, Sean Hendryx, Summer Yue, and Hugh Zhang. 自然语言规划改善LLM代码生成搜索。*ArXiv预印本*，abs/2409.03733，2024。网址 [https://arxiv.org/abs/2409.03733](https://arxiv.org/abs/2409.03733)。

+   Wei et al. (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, Fei Xia, Ed H. Chi, Quoc V. Le, and Denny Zhou. 思维链提示激发大型语言模型的推理能力。收录于 Sanmi Koyejo、S. Mohamed、A. Agarwal、Danielle Belgrave、K. Cho 和 A. Oh（编），*神经信息处理系统进展 35：2022年神经信息处理系统年会，NeurIPS 2022，美国路易斯安那州新奥尔良，2022年11月28日至12月9日*，2022。网址 [http://papers.nips.cc/paper_files/paper/2022/hash/9d5609613524ecf4f15af0f7b31abca4-Abstract-Conference.html](http://papers.nips.cc/paper_files/paper/2022/hash/9d5609613524ecf4f15af0f7b31abca4-Abstract-Conference.html)。

+   Yao et al. (2022) Shunyu Yao, Howard Chen, John Yang, and Karthik Narasimhan. Webshop：迈向可扩展的现实世界网页交互与有根语言代理。收录于 Sanmi Koyejo、S. Mohamed、A. Agarwal、Danielle Belgrave、K. Cho 和 A. Oh（编），*神经信息处理系统进展 35：2022年神经信息处理系统年会，NeurIPS 2022，美国路易斯安那州新奥尔良，2022年11月28日至12月9日*，2022。网址 [http://papers.nips.cc/paper_files/paper/2022/hash/82ad13ec01f9fe44c01cb91814fd7b8c-Abstract-Conference.html](http://papers.nips.cc/paper_files/paper/2022/hash/82ad13ec01f9fe44c01cb91814fd7b8c-Abstract-Conference.html)。

+   Yao et al. (2023a) Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Tom Griffiths, Yuan Cao, and Karthik Narasimhan. 思维树：利用大型语言模型进行深思熟虑的问题解决。收录于 Alice Oh、Tristan Naumann、Amir Globerson、Kate Saenko、Moritz Hardt 和 Sergey Levine（编），*神经信息处理系统进展 36：2023年神经信息处理系统年会，NeurIPS 2023，美国路易斯安那州新奥尔良，2023年12月10日至16日*，2023a。网址 [http://papers.nips.cc/paper_files/paper/2023/hash/271db9922b8d1f4dd7aaef84ed5ac703-Abstract-Conference.html](http://papers.nips.cc/paper_files/paper/2023/hash/271db9922b8d1f4dd7aaef84ed5ac703-Abstract-Conference.html)。

+   Yao 等人（2023b）Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R. Narasimhan 和 Yuan Cao。React：在语言模型中协同推理与行动。发表于 *第十一届国际学习表征会议（ICLR 2023），卢旺达基加利，2023年5月1日至5日*。OpenReview.net, 2023b。网址 [https://openreview.net/forum?id=WE_vluYUL-X](https://openreview.net/forum?id=WE_vluYUL-X)。

+   You 等人（2024）Keen You, Haotian Zhang, Eldon Schoop, Floris Weers, Amanda Swearngin, Jeffrey Nichols, Yinfei Yang 和 Zhe Gan。Ferret-ui：基于多模态大语言模型的移动 UI 理解。*ArXiv 预印本*，abs/2404.05719，2024。网址 [https://arxiv.org/abs/2404.05719](https://arxiv.org/abs/2404.05719)。

+   Zhang 等人（2024）Yao Zhang, Zijian Ma, Yunpu Ma, Zhen Han, Yu Wu 和 Volker Tresp。Webpilot：一种多功能的自主多智能体系统，用于通过战略探索执行 web 任务。*ArXiv 预印本*，abs/2408.15978，2024。网址 [https://arxiv.org/abs/2408.15978](https://arxiv.org/abs/2408.15978)。

+   Zheng 等人（2024）Boyuan Zheng, Boyu Gou, Jihyung Kil, Huan Sun 和 Yu Su。GPT-4v(ision) 是一个通用的 web 智能体，前提是其具有实际基础。发表于 *第41届国际机器学习大会*，2024。网址 [https://openreview.net/forum?id=piecKJ2DlB](https://openreview.net/forum?id=piecKJ2DlB)。

+   Zhou 等人（2023）Shuyan Zhou, Frank F Xu, Hao Zhu, Xuhui Zhou, Robert Lo, Abishek Sridhar, Xianyi Cheng, Yonatan Bisk, Daniel Fried, Uri Alon 等人。Webarena：用于构建自主智能体的真实 web 环境。*ArXiv 预印本*，abs/2307.13854，2023。网址 [https://arxiv.org/abs/2307.13854](https://arxiv.org/abs/2307.13854)。

## 附录 A 基于 MPC 的规划四个阶段的提示

### A.1 行动提议

<svg class="ltx_picture" height="759.39" id="A1.SS1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,759.39) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 741.18)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Action Proposal</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="709.68" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You are an autonomous intelligent agent tasked with navigating a web browser. You will be given web-based tasks. These tasks will be accomplished through the use of specific actions you can issue. Here’s the information you’ll have: {Web Information} The user’s objective: {Task Objective} This is the task you’re trying to complete. The current web page screenshot: {Web Page Screenshot Image} This is a screenshot of the webpage, with each interactable element assigned a unique numerical id. Each bounding box and its respective id shares the same color. The observation, which lists the IDs of all interactable elements on the current web page with their text content if any, in the format [id][tagType][text content]. tagType is the type of the element, such as button, link, or textbox. text content is the text content of the element. For example, [1234][button][’Add to Cart’] means that there is a button with id 1234 and text content ’Add to Cart’ on the current web page. [][StaticText][text] means that the element is of some text that is not interactable. The current web page’s URL: {Web URL} This is the page you’re currently navigating. The open tabs: {Previous Tabs} These are the tabs you have open. The previous action: {Previous Action} This is the action you just performed. It may be helpful to track your progress. The actions you can perform fall into several categories: Page Operation Actions: - click [id]: This action clicks on an element with a specific id on the webpage. - type [id] [content]: Use this to type the content into the field with id. By default, the Enter key is pressed after typing unless press_enter_after is set to 0, i.e., type [id] [content] [0]. - hover [id]: Hover over an element with id. - press [key_comb]: Simulates the pressing of a key combination on the keyboard (e.g., Ctrl+V) - scroll [down] or scroll [up]: Scroll the page up or down. Tab Management Actions: - new_tab: Open a new, empty browser tab. - tab_focus [tab_index]: Switch the browser’s focus to a specific tab using its index. - close_tab: Close the currently active tab. URL Navigation Actions: - goto [url]: Navigate to a specific URL. - go_back: Navigate to the previously viewed page. - go_forward: Navigate to the next page (if a previous go_back action was performed). Completion Action: - stop [answer]: Issue this action when you believe the task is complete. If the objective is to find a text-based answer, provide the answer in the bracket. Homepage: If you want to visit other websites, check out the homepage at http://homepage.com. It has a list of websites you can visit. http://homepage.com/password.html lists all the account name and password for the websites. You can use them to log in to the websites. To be successful, it is very important to follow the following rules: 1\. You should only issue an action that is valid given the current observation 2\. You should only issue one action at a time. 3\. You should follow the examples to reason step by step and then issue the next action. 4\. Generate the action in the correct format. Start with a “In summary, the next action I will perform is” phrase, followed by action. For example, In summary, the next action I will perform is click [1234]. 5\. Issue stop action when you think you have achieved the objective. Don’t generate anything after stop.</foreignobject></g></g></svg>

### A.2 自我优化

<svg class="ltx_picture" height="588.65" id="A1.SS2.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,588.65) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 573.14)"><foreignobject color="#FFFFFF" height="9.61" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">自我完善</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="541.64" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你正在协助一个网页导航代理帮助用户完成任务。根据用户的意图、操作历史和网页的当前状态，代理已提出一系列候选操作。你的角色不是决定代理在此步骤中应采取的最佳操作，而是过滤掉那些不太可能与完成任务相关或有帮助的操作。请选出你认为可能帮助代理完成任务的所有操作。需要注意的是，为了完成任务，代理将执行一系列操作。因此，此步骤中采取的操作不一定立即导致任务完成。你应该选择任何在当前网页状态下可能相关的操作。尽量全面、周到地考虑！不要遗漏任何可能的操作。如果某个操作显然是最好的，并且所有其他操作显然都不太相关，你只能选择一个操作。请谨慎执行此操作，因为某些操作可能在更长远的过程中有帮助。只要某个操作可能与任务相关，即使它可能不是此步骤中最直接的操作，也应包含在内！某些相关操作可能乍一看显得间接，但在更长远的过程中可能会有帮助。请务必选择至少一个操作。

*重要* 请将你的回答格式化为如下两行：

思考：<你的思考和推理过程>。你必须逐一评估每个操作，并想象它是否与任务相关，格式如下：操作：... 理由：... 选择的操作：id0;id1;aid2;...（请返回候选操作列表中操作的索引，从0开始。不要输出操作描述本身。用分号分隔索引。不要在分号后添加空格或任何其他字符。）

操作历史：{last_actions_str}

当前网址：{current_url}

与用户意图相关的图像显示在**FIRST** {len(intent_images)}张图像中（位于用户意图之前）。

代理的最后{len(screenshots)}张轨迹快照显示在**LAST** {len(screenshots)}张图像中。**LAST IMAGE**表示网页的当前状态。

提议的动作：{action_descriptions}</foreignobject></g></g></svg>

### A.3 世界模型

<svg class="ltx_picture" height="206.06" id="A1.SS3.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,206.06) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 190.54)"><foreignobject color="#FFFFFF" height="9.61" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">World Model</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="159.05" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You are an agent that predicts the effect of an action on a webpage. You will be given a screenshot of a webpage, a sequence of actions and state changes applied to the initial screenshot, and an operation to perform on the webpage. You are required to predict the new changes that will occur on the webpage after the operation is performed, such as the appearance of new elements, the disappearance of existing elements, or changes in the content of existing elements. The operation type and the element to operate will be provided in the prompt. Directly output  State changes:... and don’t output anything else. Try to be as comprehensive and detailed as possible. Based on the initial screenshot and the changes to the webpage, please predict the changes after action:</foreignobject></g></g></svg>

### A.4 奖励模型

<svg class="ltx_picture" height="325.13" id="A1.SS4.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,325.13) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 309.62)"><foreignobject color="#FFFFFF" height="9.61" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Reward Model</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="278.12" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You are an expert in evaluating the performance of a web navigation agent. The agent is designed to help a human user navigate a website to complete a task. Given the user’s intent, the agent’s action history, the current state of the webpage, your goal is to decide **whether the simulated steps by the agent indicate a successful execution of the user intent**. In particular, if the predicted state (i.e., the current state represented by the last image plus all the predicted changes so far) corresponds to a successful final state. If it is a failure but it looks like the simulated steps are on the right track towards success, you should also output as such. Note that, in the simulated steps, all the state changes are predicted by the agent’s world model, and they may not actually be faithful to the real website interactions (e.g., some proposed actions may not be avaiable in a realistic website). You should also account for this in your evaluation (e.g., if the predicted state changes are not reasonable then it’s probably a failure). *IMPORTANT* Format your response into two lines as shown below: Thoughts: <your thoughts and reasoning process> Status: "success" or "failure" On the right track to success: "yes" or "no"</foreignobject></g></g></svg>

## 附录 B 案例研究

### B.1 由不完美的世界模型模拟引起的错误

![参考说明文字](img/cf0282d5486d5d275eedd8fae27cb736.png)

图 5：由不完美的世界模型模拟引起的错误案例。

### B.2 受益于世界模型模拟的正面案例

![参考说明文字](img/c7ea554617e2453dfe92436b76ec222a.png)

图 6：模拟导致正确动作预测的正面案例。
