<!--yml

分类：未分类

日期：2025-01-11 12:09:54

-->

# YOLO-MARL：多智能体强化学习中的“只需一次LLM”

> 来源：[https://arxiv.org/html/2410.03997/](https://arxiv.org/html/2410.03997/)

Yuan Zhuang^†

康涅狄格大学

[yuan.2.zhuang@uconn.edu](mailto:yuan.2.zhuang@uconn.edu)

&Yi Shen^†

宾夕法尼亚大学

[eshen@seas.upenn.edu](mailto:eshen@seas.upenn.edu)

&Zhili Zhang

康涅狄格大学

[zhili.zhang@uconn.edu](mailto:zhili.zhang@uconn.edu) &Yuxiao Chen

NVIDIA

[yuxiaoc@nvidia.com](mailto:yuxiaoc@nvidia.com) &Fei Miao

康涅狄格大学

[fei.miao@uconn.edu](mailto:fei.miao@uconn.edu)

###### 摘要

深度多智能体强化学习（MARL）的进展使其成为合作博弈中决策的一种有前景的方法。然而，对于一些博弈环境，MARL智能体仍然难以学习到有效的合作策略。最近，大型语言模型（LLMs）展示了突出的推理能力，使其成为增强智能体间协调的有力候选者。然而，由于LLMs的模型规模较大，频繁推理LLMs以决定智能体的行动可能会非常昂贵。在这项工作中，我们提出了“只需一次LLM”多智能体强化学习（YOLO-MARL）框架，它利用LLMs的高层任务规划能力，来改善合作博弈中多智能体的策略学习过程。值得注意的是，对于每个博弈环境，YOLO-MARL在提出的策略生成、状态解释和规划功能生成模块中仅需要与LLMs进行一次交互，然后便进入MARL的策略训练过程。这避免了在训练过程中频繁调用LLMs API所带来的持续成本和计算时间。此外，经过训练的去中心化标准神经网络政策可以独立于LLM运行。我们在三个不同的环境中评估了我们的方法，结果表明YOLO-MARL优于传统的MARL算法。

¹¹脚注：† 这些作者对本研究做出了同等贡献。

## 1 引言

多智能体强化学习（MARL）算法已被证明是解决多智能体系统中复杂决策问题的强大框架。随着多智能体系统应用的增加，如仓库中的移动机器人和需要复杂推理与策略的游戏，个体智能体在没有集中式决策者的动态环境中学习、合作或竞争变得愈发重要（Papoudakis & Schäfer, [2021](https://arxiv.org/html/2410.03997v1#bib.bib26)）。在合作马尔可夫博弈中，智能体需要协调他们的行动，以最大化联合奖励。然而，现有的MARL算法在学习合作博弈的分布式策略时面临挑战。此外，它们还在处理奖励稀疏、环境动态变化以及大规模行动空间等任务时遇到困难，这可能会妨碍高效学习和智能体间的协作。

LLMs在上下文学习能力和先前知识的基础上，作为高阶语义规划者表现出色（Ahn等，[2022](https://arxiv.org/html/2410.03997v1#bib.bib2)）。Zhang等人（[2023](https://arxiv.org/html/2410.03997v1#bib.bib35)）和Kannan等人（[2024](https://arxiv.org/html/2410.03997v1#bib.bib14)）直接将LLMs用作具身智能体，展示了LLMs在多机器人系统中的规划能力。还有一些研究集中于利用LLMs来指导强化学习（RL）训练，以达到更好的表现。ELLM（Du等，[2023](https://arxiv.org/html/2410.03997v1#bib.bib6)）利用LLMs建议目标来辅助RL训练，而Kwon等人（[2023](https://arxiv.org/html/2410.03997v1#bib.bib15)）则关注LLM提供的行动与RL策略之间的对齐。尽管这些方法展示了将LLM融入策略训练的巨大潜力，但它们尚未将方法扩展到多智能体场景中。更重要的是，将LLMs作为智能体使用或将其集成到RL训练循环中会带来一些挑战。与LLMs在长回合任务或复杂环境中的反复交互，特别是在使用像Claude-3.5或GPT-o1这样的高级LLMs时，可能是耗时且昂贵的；对于需要在数千万步上进行训练的任务来说，这变得不可行。此外，还存在与LLM的间歇性断开连接的风险，这可能会扰乱训练过程并影响系统的稳定性。

基于已识别的洞察和挑战，我们提出了YOLO-MARL，如图[1](https://arxiv.org/html/2410.03997v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")所示，这是一种创新的方法，利用LLM的规划能力来增强MARL的策略训练。具体而言，我们框架的主要优势在于，每个游戏环境只需与LLM进行一次交互。在策略生成、状态解读和规划功能生成模块完成后，MARL训练过程中无需进一步与LLM交互，从而显著减少了LLM推理的通信和计算开销。此外，YOLO-MARL展示了其强大的泛化能力和应用简便性：通过提出的策略生成和状态解读模块，我们的方法与各种MARL算法兼容，如Yu等人（[2022](https://arxiv.org/html/2410.03997v1#bib.bib34)）、Rashid等人（[2018](https://arxiv.org/html/2410.03997v1#bib.bib27)）、Lowe等人（[2020](https://arxiv.org/html/2410.03997v1#bib.bib21)）的算法，并且只需用户对新游戏环境具备基本的背景理解。我们还在一个稀疏奖励的多智能体环境——基于等级的觅食环境（Papoudakis & Schäfer, [2021](https://arxiv.org/html/2410.03997v1#bib.bib26)）和一个高度战略性任务环境——星际争霸多智能体挑战环境（Samvelyan等人，[2019](https://arxiv.org/html/2410.03997v1#bib.bib28)）以及MPE环境（Lowe等人，[2020](https://arxiv.org/html/2410.03997v1#bib.bib21)）中对我们框架进行了评估，并展示了YOLO-MARL超越了多个MARL基准。我们还提供了若干消融研究结果，展示了每个模块在提出框架中的作用。据我们所知，YOLO-MARL是将LLM的高层次推理和规划能力与MARL结合的首次尝试之一，因为迄今为止，关于LLM在MARL中的应用文献极为有限（Sun等人，[2024](https://arxiv.org/html/2410.03997v1#bib.bib29)）。

总结而言，我们提出的方法YOLO-MARL具有以下优势：

+   •

    该框架将大语言模型（LLMs）的规划能力与多智能体强化学习（MARL）结合，以提升在复杂合作游戏环境中的策略学习表现。特别地，我们的方法利用LLM广泛的推理能力，生成高层次的任务分配规划功能，以促进智能体的协调合作。

+   •

    YOLO-MARL需要最少的LLM参与，从而显著减少了计算开销，并缓解了在训练过程中调用LLM时的通信连接不稳定性问题。

+   •

    我们的方法利用零-shot提示，并且可以轻松适应各种游戏环境，用户仅需具备基本的先验知识即可。

YOLO-MARL的概览见图[1](https://arxiv.org/html/2410.03997v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")。所有的提示、环境以及生成的规划函数可在附录中找到。

![参见标题说明](img/adb85336778068a0b8a1a3898938ff32.png)

图1：我们框架YOLO-MARL的示意图。(a) 策略生成：我们将基本的环境和任务描述传递给LLM，以生成该特定环境的策略。(b) 状态解释：我们处理全局状态，使全局状态的格式更加结构化和有序，以便LLM能够更好地理解。(c) 规划函数生成：我们将环境和任务描述、LLM生成的策略和状态解释函数串联起来。这些提示随后被输入LLM，以生成该环境的规划函数。(d) MARL训练：状态解释函数和生成的规划函数被集成到MARL训练过程中。在规划函数生成后，LLM不再需要进一步的交互。

## 2 相关工作

### 2.1 多代理强化学习

MARL由于其在解决复杂、去中心化问题上的潜力，已获得越来越多的关注。中心化训练与去中心化执行已成为克服独立学习局限性的流行框架。像QMIX（Rashid等人，[2018](https://arxiv.org/html/2410.03997v1#bib.bib27)）和MADDPG（Lowe等人，[2020](https://arxiv.org/html/2410.03997v1#bib.bib21)）在训练过程中使用中心化的评价器或价值函数来协调代理，而在测试时允许它们独立执行。在协作环境中，像COMA（Foerster等人，[2017](https://arxiv.org/html/2410.03997v1#bib.bib8)）和VDN（Sunehag等人，[2017](https://arxiv.org/html/2410.03997v1#bib.bib30)）等算法使代理能够共享奖励，并以协调的方式行动以最大化联合奖励。Wang等人（[2024](https://arxiv.org/html/2410.03997v1#bib.bib31)）提出了一种新的方法，利用语言约束预测来应对自然语言背景下安全MARL的挑战。然而，现有的MARL算法在稀疏奖励环境中的表现可能不佳，且在某些环境中仍难以学习到完全协作的策略。到目前为止，关于使用LLM进行MARL的文献非常有限（Sun等人，[2024](https://arxiv.org/html/2410.03997v1#bib.bib29)），并且尚不清楚LLM是否以及如何可以用于基于MARL的决策。

### 2.2 单代理强化学习和决策的语言模型

许多现有的工作将 LLMs 用作强化学习训练过程的组成部分。Du 等人 ([2023](https://arxiv.org/html/2410.03997v1#bib.bib6)) 通过计算 LLMs 提供的目标与代理展示的行为之间的相似度，增强了代理的探索性。Carta 等人 ([2023](https://arxiv.org/html/2410.03997v1#bib.bib3)) 在在线强化学习过程中，利用来自 LLMs 的基于语言的目标，通过生成基于提示的动作来实现。Kwon 等人 ([2023](https://arxiv.org/html/2410.03997v1#bib.bib15)) 提供了基于 LLMs 建议的标量奖励，以指导强化学习训练。然而，大多数这些方法尚未在马尔可夫博弈的背景下进行探索，并且在训练过程中需要与 LLMs 进行广泛的互动。

Gupta 等人 ([2022](https://arxiv.org/html/2410.03997v1#bib.bib10)) 利用 CLIP 的视觉嵌入用于代理探索环境。Fan 等人 ([2022](https://arxiv.org/html/2410.03997v1#bib.bib7)) 研究了一个多任务强化学习问题，其中代理的任务是完成 MINEDOJO 任务。Ahn 等人 ([2022](https://arxiv.org/html/2410.03997v1#bib.bib2)) 提出了 SayCan 这一方法，通过预训练技能的价值函数来将大型语言模型（LLMs）与机器人上执行抽象命令相结合。Liang 等人 ([2023](https://arxiv.org/html/2410.03997v1#bib.bib18)) 发现，编写代码的 LLMs 可以重新用于编写机器人策略代码。Huang 等人 ([2022](https://arxiv.org/html/2410.03997v1#bib.bib12)) 显示，通过利用环境反馈，LLMs 能够形成内心独白，从而更加丰富地处理和规划。其他研究如 Ma 等人 ([2024](https://arxiv.org/html/2410.03997v1#bib.bib23)) 和 Xie 等人 ([2023](https://arxiv.org/html/2410.03997v1#bib.bib32)) 使用 LLMs 的先验知识和代码生成能力来生成奖励函数，而我们则利用代码生成来进行规划功能。Lin 等人 ([2024](https://arxiv.org/html/2410.03997v1#bib.bib19)) 强调了 LLMs 在处理复杂低级任务时的局限性。另一方面，我们利用 LLMs 的高级推理能力来增强 RL 模型训练中的低级动作表现。

### 2.3 多智能体系统中的大型语言模型

基于LLM的多智能体（LLM-MA）系统关注多样化的智能体特征、交互和集体决策。尽管这使得智能体能够在动态、复杂的任务中进行协作，但也由于LLM之间的通信增加了计算开销（Guo等人，[2024](https://arxiv.org/html/2410.03997v1#bib.bib9)），（Sun等人，[2024](https://arxiv.org/html/2410.03997v1#bib.bib29)）。Camel Li等人（[2024](https://arxiv.org/html/2410.03997v1#bib.bib16)）和MetaGPT Hong等人（[2023](https://arxiv.org/html/2410.03997v1#bib.bib11)）采用多个LLM智能体来完成头脑风暴和软件开发等任务。Nascimento等人（[2023](https://arxiv.org/html/2410.03997v1#bib.bib24)）通过整合基于GPT的技术，增强了通信和智能体的自主性。在多机器人环境中，Chen等人（[2023](https://arxiv.org/html/2410.03997v1#bib.bib5)）比较了四种多智能体通信框架的任务成功率和令牌效率。SMART-LLM（Kannan等人，[2023](https://arxiv.org/html/2410.03997v1#bib.bib13)）将多机器人任务计划分解为子目标，以便LLM能够高效执行，而Co-NavGPT（Yu等人，[2023](https://arxiv.org/html/2410.03997v1#bib.bib33)）将LLM整合为合作导航的全局规划者。聚焦于多智能体路径规划（MAPF），Chen等人（[2024](https://arxiv.org/html/2410.03997v1#bib.bib4)）研究了使用LLM解决MAPF问题的性能。Agashe等人（[2023](https://arxiv.org/html/2410.03997v1#bib.bib1)）提出了LLM-Coordination（LLM-Co）来使LLM能够进行协调游戏。Li等人（[2023](https://arxiv.org/html/2410.03997v1#bib.bib17)）探索了在文本环境中使用LLM进行合作游戏，Ma等人（[2023](https://arxiv.org/html/2410.03997v1#bib.bib22)）则探索了在StarCraft II环境中使用LLM。与之不同，我们的方法利用LLM的规划能力来训练更好的MARL策略，而不是直接将其用作智能体。

## 3 问题表述

马尔可夫博弈（MG）被定义为多智能体决策问题，其中多个智能体之间的互动影响整个系统的状态动态以及在特定条件下每个智能体的奖励（Littman，[1994](https://arxiv.org/html/2410.03997v1#bib.bib20)）。在本研究中，我们考虑一个马尔可夫博弈，或称随机博弈，由Owen（[1982](https://arxiv.org/html/2410.03997v1#bib.bib25)）定义为一个元组$G:=(\mathcal{N},S,A,\{r^{i}\}_{i\in\mathcal{N}},p,\gamma)$，其中$\mathcal{N}$是$N$个智能体的集合，$S=S^{1}\times\cdots\times S^{N}$是联合状态空间，$A=A^{1}\times\cdots\times A^{N}$是联合动作空间，$(S^{i},A^{i})$分别是智能体$i$的状态空间和动作空间，$\gamma\in[0,1)$是折扣因子（Littman，[1994](https://arxiv.org/html/2410.03997v1#bib.bib20)；Owen，[1982](https://arxiv.org/html/2410.03997v1#bib.bib25)）。状态转移$p:S\times A\rightarrow\Delta(S)$由当前状态和联合动作控制，其中$\Delta(S)$表示所有联合状态空间$S$上的概率分布集合。每个智能体都有一个奖励函数$r^{i}:S\times A\rightarrow\mathbb{R}$。在时间$t$，智能体$i$根据策略$\pi^{i}:S\rightarrow\Delta(A^{i})$选择其动作$a^{i}_{t}$。对于每个智能体$i$，它尝试最大化其期望的折扣奖励总和，即其目标函数$J^{i}(s,\pi)=\mathbb{E}\left[\sum_{t=1}^{\infty}\gamma^{t-1}r_{t}^{i}(s_{t},a_{t})|s_{1}=s,a_{t}\sim\pi(\cdot|s_{t})\right]$。在文献中，深度MARL算法（Lowe等，[2020](https://arxiv.org/html/2410.03997v1#bib.bib21)；Yu等，[2022](https://arxiv.org/html/2410.03997v1#bib.bib34)；Rashid等，[2018](https://arxiv.org/html/2410.03997v1#bib.bib27)）已被设计用于训练基于神经网络的策略$\pi_{i}(\theta_{i})$。对于合作博弈，通常在训练过程中使用所有智能体共享的奖励函数，这在本研究中也有所考虑。

## 4 方法论

在本节中，我们介绍了我们的方法，YOLO-MARL，它利用大规模语言模型（LLMs）增强MARL。具体而言，在训练过程中，我们利用LLM的高级任务规划能力来指导MARL过程。我们的方法包括四个关键组件：策略生成、状态解释、规划函数生成，以及在整个过程中融入LLM生成的规划函数的MARL训练过程。

算法 1 YOLO-MARL 训练过程

1: 大型语言模型 $LLM$，观察过程函数 $F_{\text{S}}$，MARL 参与者 $\mathcal{A}$，MARL 算法 $MARL_{alg}$，提示词 $P$ 2: 超参数：奖励信号 $r^{\prime}$，惩罚信号 $p^{\prime}$ 3:// 从 LLM 获取样本函数代码 4: $\mathcal{F_{\mathcal{T}}} \sim LLM(P)$ 5: 对每个训练步骤执行： 6: // 从 $F_{\text{S}}$ 获取处理后的全局观察 $S_{I}$ 7: $S_{I} \leftarrow F_{\text{S}}(S_{v})$ 8: // 为每个代理分配任务 $\mathcal{T}$ 9: $\mathcal{T}_{1}, \mathcal{T}_{2}, \dotsc \leftarrow \mathcal{F_{\mathcal{T}}}(S_{I})$ 10: // 输出来自参与者的行动 11: $a_{1}, a_{2}, \dotsc \leftarrow \mathcal{A}(S_{v})$ 12: 对每个代理 $i$ 执行： 13: if $a_{i} \in \mathcal{T}_{i}$ then 14: $\Delta r_{i} \leftarrow r^{\prime}$ 15: else 16: $\Delta r_{i} \leftarrow p^{\prime}$ 17: end if 18: 结束 19: // 计算评论员的最终奖励 20: $R \leftarrow r + \sum_{i} \Delta r_{i}$ 21: // 使用 $R$ 作为 MARL 训练的最终奖励 22: $\pi(\theta) = MARL_{alg}(R)$ 23: 结束 24: 返回训练后的 MARL 策略

### 4.1 策略生成

为了创建一个适用于各种环境的通用框架——尤其是当用户可能具有有限的先前知识时——我们将一个策略生成模块（Strategy Generation Module）纳入我们的 methodology 中。该模块使得大型语言模型（LLM）能够自动生成适用于不同环境的策略，而无需大量的人工输入或专业知识。

如图[1](https://arxiv.org/html/2410.03997v1#S1.F1 "图 1 ‣ 1 引言 ‣ YOLO-MARL：一次性使用 LLM 进行多智能体强化学习")（a）所示，LLM 被提供了环境的基本信息，包括任务描述、相关规则以及与环境交互的约束条件。此外，我们在提示词中提供了一个通用的指导方针，帮助 LLM 生成有效的策略。汇集所有信息后，LLM 将输出详细的策略，以完成任务或达成目标，遵循指定的格式。

通过汇总所有这些信息，LLM 输出详细的策略来完成任务或实现目标，遵循指定的格式。策略生成对多个原因至关重要：

+   •

    减少用户负担：它减轻了用户全面理解新环境的需求，节省了时间和精力。

+   •

    增强泛化能力：它使框架能够通过最小的提示修改，适应不同的环境。

+   •

    促进规划函数生成：策略作为规划函数生成模块中提示的关键组成部分。使用YOLO-MARL但没有策略生成模块的结果显示在消融研究[6.1](https://arxiv.org/html/2410.03997v1#S6.SS1 "6.1 Comparison between YOLO-MARL with and without Strategy Generation ‣ 6 Ablation Study ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")中。

LLM生成的策略与其他必要信息一起被纳入提示中，以促进后续的规划函数生成。有关策略提示及其格式的更多细节，请参见附录[C.1](https://arxiv.org/html/2410.03997v1#A3.SS1 "C.1 Strategy ‣ Appendix C Prompt Detail ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")。

### 4.2 状态解释

在许多仿真环境中，观测或状态通常以向量的形式提供，每个组件使用各种编码方法构造。虽然向量形式的观测在训练深度强化学习模型时易于处理，但由于缺乏每个组件的明确上下文，LLM很难直接解析它们的语义含义。

我们提出了状态解释模块，用于帮助LLM解释环境状态。通过提供语义上有意义的状态表示，LLM可以成功生成用于训练的可执行规划函数。形式上，给定当前环境状态的向量形式$S_{v}$，我们定义一个解释函数$F_{S}$，使得$F_{S}(S_{v})\to S_{I}$，其中$S_{I}$提供了每个状态组件的更明确且有意义的信息。具体来说，我们实现了一个Python函数来处理向量观测并重构每个组件。例如，它从原始观测中提取位置信息，并将其组织成一个变量名“position”。这一转换使得LLM能够理解状态的每一部分的上下文和意义。

最近的研究，如Ma等人（[2024](https://arxiv.org/html/2410.03997v1#bib.bib23)）和Xie等人（[2023](https://arxiv.org/html/2410.03997v1#bib.bib32)）展示了通过提供相关的环境代码来增强LLM性能的成功。在相同的方式下，我们将解释函数$F_{S}$包含在提示流水线中，格式化为Python风格的环境代码，如图[1](https://arxiv.org/html/2410.03997v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")(b)中的紫色框所示。状态解释模块显著降低了LLM生成错误函数的风险，避免了输出与训练过程不兼容的情况。关于该模块有效性的消融研究可以在第[6.2](https://arxiv.org/html/2410.03997v1#S6.SS2 "6.2 Comparison between YOLO-MARL with and without State Interpretation ‣ 6 Ablation Study ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")节中找到，而关于解释函数的更多细节请参见附录[C.2](https://arxiv.org/html/2410.03997v1#A3.SS2 "C.2 Interpretation Function ‣ Appendix C Prompt Detail ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")。

### 4.3 规划功能生成

我们方法的一个关键组成部分是利用LLM执行高层规划，而不是处理低层动作。我们将前面模块中的所有提示合并并输入到LLM中。然后，LLM生成一个合理且可执行的规划功能，能够在随后的训练过程中直接使用。

为了更简洁地表达，给定任何处理过的状态$S_{I}$，我们将分配规划功能定义为$\mathcal{F_{\mathcal{T}}}(S_{I})\to\mathcal{T}_{i}\in\mathcal{T}$，其中$\mathcal{T}=\{\mathcal{T}_{1},...,\mathcal{T}_{n}\}$是每个智能体可以执行的目标分配集合。我们在动作空间上定义分配集$\mathcal{T}$，使得一个动作可以属于多个分配，反之亦然。例如，如果分配空间定义为$\mathcal{T}=\{Landmark\_0, Landmark\_1\}$，且地标0和地标1分别位于智能体的右上角和左上角位置，那么采取“UP”动作可以与这两个分配相关联。反过来，我们也可以有多个动作对应一个分配。例如，朝“Landmark 0”移动可能涉及“UP”和“RIGHT”等动作。这指的是图[1](https://arxiv.org/html/2410.03997v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")(c)中的红色模块，关于生成函数的更多信息请参见附录[D](https://arxiv.org/html/2410.03997v1#A4 "Appendix D Examples of generated planning functions ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")。

### 4.4 带有规划功能集成的MARL训练

为了将规划功能纳入MARL训练，我们在环境提供的原始奖励基础上增加了一个额外的奖励项。具体来说，我们定义了批评者使用的最终奖励$R$为：$R=r+\sum_{i}\Delta r_{i}$，其中$r$是来自环境的原始奖励，$\Delta r_{i}$是针对每个代理$i$的额外奖励信号。项$\Delta r_{i}$基于代理的行动是否与规划功能分配的任务一致。如果代理的行动属于分配的任务，它将获得奖励$r^{\prime}$作为其$\Delta r_{i}$，如果不符合，它将获得惩罚$p^{\prime}$作为其$\Delta r_{i}$。

值得注意的是，我们在整个训练过程中不需要与LLM互动，也不需要在策略训练完成后调用规划功能。训练过程$MARL_{alg}(R)$将$R$作为奖励函数，使用相同的状态和动作空间定义，并遵循文献中的标准MARL算法和评估指标，例如Yu等人（[2022](https://arxiv.org/html/2410.03997v1#bib.bib34)）、Rashid等人（[2018](https://arxiv.org/html/2410.03997v1#bib.bib27)）和Lowe等人（[2020](https://arxiv.org/html/2410.03997v1#bib.bib21)）。这使得我们的方法与在训练过程中始终与LLM交互或将LLM作为代理的其他方法相比，具有高效性，正如图[1](https://arxiv.org/html/2410.03997v1#S1.F1 "图1 ‣ 1 引言 ‣ YOLO-MARL：只调用一次LLM进行多智能体强化学习")(d)中的贪心框显示的那样。实际上，即使使用最先进的LLM API，通过LLM的API生成规划功能也会产生极小的费用——每个环境不到一美元。

## 5 实验

在本节中，我们评估了我们的方法在三个不同环境下的表现：MPE、LBF和SMAC。我们使用claude-3-5-sonnet-20240620进行实验。^*^**我们在工作中主要使用Claude 3.5 Sonnet模型作为大语言模型（LLM）：[https://www.anthropic.com/news/claude-3-5-sonnet](https://www.anthropic.com/news/claude-3-5-sonnet)

### 5.1 设置

基准。 在我们的实验中，我们比较了MARL算法MADDPG（Lowe等人，[2020](https://arxiv.org/html/2410.03997v1#bib.bib21)）、MAPPO（Yu等人，[2022](https://arxiv.org/html/2410.03997v1#bib.bib34)）和QMIX（Rashid等人，[2018](https://arxiv.org/html/2410.03997v1#bib.bib27)），并根据人类编写的奖励的良好调优表现设置了默认超参数，并在所有关于该任务的实验中固定该超参数以进行MARL训练。实验超参数列在附录中。

指标。为了评估我们方法的表现，我们在SMAC环境中使用胜率作为评估指标，在所有其他环境中使用平均回报作为评估标准。在评估过程中，我们仅依赖环境提供的默认回报值来进行基准和我们方法的公平比较。

### 5.2 结果

表 1：YOLO-MARL 和 MARL 在 LBF 环境中的三次实验对比。训练过程中最高的评价回报以**粗体**突出显示。相应的结果可以在图 [2](https://arxiv.org/html/2410.03997v1#S5.F2 "图 2 ‣ 5.2 结果 ‣ 5 实验 ‣ YOLO-MARL：多智能体强化学习中的一次性大语言模型") 中找到。M 代表一百万次训练步骤。我们在同一台机器上运行所有实验。

|  | 0.2M / 0.4M / 1.5M / 2M 步骤后的平均回报 |
| --- | --- |
|  | QMIX | MADDPG | MAPPO |
| MARL | 0.00/ 0.01/ 0.25/ 0.38 | 0.09/ 0.33/ 0.26/ 0.32 | 0.31/ 0.72/ 0.99/ 0.99 |
| YOLO-MARL | 0.01/ 0.02/ 0.60/ 0.78 | 0.13/ 0.38/ 0.39/ 0.44 | 0.93/ 0.98/ 0.99/ 0.99 |

基于层级的觅食。基于层级的觅食（LBF）（Papoudakis & Schäfer, [2021](https://arxiv.org/html/2410.03997v1#bib.bib26)）是为多智能体强化学习（MARL）训练设计的一个具有挑战性的稀疏奖励环境。在该环境中，智能体必须学会导航路径并成功收集食物，只有在任务完成后才会获得奖励。为了在合作场景中评估我们的框架，我们选择了2玩家、2食物的完全合作场景。在这个场景中，所有智能体必须协作并协调行动，以便同时收集食物。该环境提供的动作空间包括[NONE, NORTH, SOUTH, WEST, EAST, LOAD]，我们将任务集定义为[NONE, 食物i, …, LOAD]。通过使用智能体和食物项目的相对位置，我们将分配的任务映射到动作空间中的相应动作，并基于此对齐计算奖励。我们在3个不同种子下评估了我们的框架，结果如图[2](https://arxiv.org/html/2410.03997v1#S5.F2 "Figure 2 ‣ 5.2 Results ‣ 5 Experiments ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")和表[1](https://arxiv.org/html/2410.03997v1#S5.T1 "Table 1 ‣ 5.2 Results ‣ 5 Experiments ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")所示。LLM通过提供奖励信号来辅助MARL算法，我们的框架在所有测试的MARL算法中显著超越了基线，达到了最大105%的均值回报提升，并且收敛速度比基线快了2倍。根据结果，我们的框架在所有基线算法中都表现出色，特别是在QMIX和MADDPG中取得了显著的改进，同时在MAPPO中也实现了更快的收敛速度。为了评估我们生成的函数质量的变异性，我们展示了三种不同生成函数的结果，见附录[B.1](https://arxiv.org/html/2410.03997v1#A2.SS1 "B.1 Comparison for different generated funcions ‣ Appendix B Additional result ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")中的图[8](https://arxiv.org/html/2410.03997v1#A2.F8 "Figure 8 ‣ B.1 Comparison for different generated funcions ‣ Appendix B Additional result ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")和表[3](https://arxiv.org/html/2410.03997v1#A2.T3 "Table 3 ‣ B.1 Comparison for different generated funcions ‣ Appendix B Additional result ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")。结果表明，我们的框架始终生成高质量的函数，每个函数在所有基线算法中均取得了相似的改进。

![参见说明文字](img/96aaf9c6d35441b5f3194b8f7b57806c.png)

(a) MADDPG

![参见说明文字](img/0067f63181dae42974920d62fe006d92.png)

(b) MAPPO

![参见说明文字](img/45e69c265d27556ec1314d4f905c4681.png)

(c) QMIX

图 2：LBF 环境在 3 个种子下的结果：实线表示平均表现，阴影区域表示 3 个不同种子下的范围（最小值到最大值）。

多代理粒子环境。我们在多代理粒子环境（MPE）（Lowe 等， [2020](https://arxiv.org/html/2410.03997v1#bib.bib21)）的简单扩展环境中评估我们的框架，这是一个完全合作的游戏。这个环境包含 N 个代理和 N 个地标。总体而言，代理必须学会覆盖所有地标，同时避免碰撞。其动作空间包括 [no_action, move_left, move_right, move_down, move_up]。我们定义每个代理的任务为 [Landmark_i,…,No action]。在训练过程中，基于全局观察，我们获取每个代理相对于地标的位置。类似于 LBF，我们将每个代理的任务映射回相应的动作空间，然后在动作空间层面奖励策略的动作。我们在 3 代理和 4 代理场景中使用 QMIX 和 MADDPG 作为基线评估我们的方法。如图 [3](https://arxiv.org/html/2410.03997v1#S5.F3 "Figure 3 ‣ 5.2 Results ‣ 5 Experiments ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning") 所示，我们的框架（彩色线）在 3 代理场景中相比于基线（黑色线）算法平均回报提高了 7.66% 和 8.8%，在 4 代理场景中使用 QMIX 和 MADDPG 时，分别提高了 2.4% 和 18.09%。这些提升展示了我们框架在增强代理之间的协调以覆盖所有地标方面的有效性。

![参见图注](img/0418daaa1dbaf4e6d5a163a7bad783cb.png)

（a）MADDPG

![参见图注](img/7f23aa1566f9e1586d75db1e51777b40.png)

（b）QMIX

图 3：MPE 简单扩展场景 3 个代理的结果。实线表示平均表现，阴影区域表示 3 个不同生成的规划函数的范围（最小值到最大值）。

![参见图注](img/fd5f9b4b20f341a3a3407bf3ab672519.png)

（a）MADDPG

![参见图注](img/dd9ea54a4818413fb681b8ede19e212e.png)

（b）QMIX

图 4：MPE 简单扩展场景 4 个代理的结果。实线表示平均表现，阴影区域表示 3 个不同生成的规划函数的范围（最小值到最大值）。

星际争霸多人智能挑战环境。星际争霸多人智能挑战（SMAC）（Samvelyan等，[2019](https://arxiv.org/html/2410.03997v1#bib.bib28)）模拟战斗场景，其中一组受控代理需要在有限的步数内使用固定策略摧毁敌方团队。我们在三个不同的地图上测试了我们的方法：3M、2s vs 1sc和2c vs 64zg。环境中的动作空间包括[无动作、停止、向北移动、向南移动、向西移动、向东移动、攻击敌人1，…攻击敌人n]，其中n是地图上敌人的总数。根据代理需要参与的敌人数，动作空间变得越来越复杂，特别是在2c vs 64zg地图上，包含64个敌人，提供70种可能的动作。

在我们的实验中，我们将指派空间简单定义为[移动、攻击、停止、无（针对死亡代理）]。我们测试了MAPPO的表现，SMAC的结果如图[5](https://arxiv.org/html/2410.03997v1#S5.F5 "Figure 5 ‣ 5.2 Results ‣ 5 Experiments ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")所示。如图所示，尽管我们提供了可能远非最佳的简单指令，我们的框架在某些地图上仍能取得相当的结果。这表明我们的框架即使在需要战略性移动的环境中，依然具有竞争力。我们还探讨了该环境下的稀疏奖励情况，其中基准算法的胜率始终接近0，而我们生成的规划奖励函数对比基准算法表现更佳。我们建议将这种配对生成作为未来工作，并将在第[7](https://arxiv.org/html/2410.03997v1#S7 "7 Limitation and Future Work ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")节中进一步讨论。

![请参考说明文字](img/d59f86cd8db9765d7e6d0fc084cefef5.png)

(a) 3m

![请参考说明文字](img/5cd03344aad8942094d2ab8b920f6cd6.png)

(b) 2s vs 1sc

![请参考说明文字](img/4c9ed214f6b2c0756da8fd2e00887fdb.png)

(c) 2c vs 64zg

图5：在SMAC环境中3个地图的结果：我们方法与MAPPO基准方法在3个地图（3m、2s vs 1sc和2c vs 64zg）上，通过3个不同种子的比较的平均胜率，实线表示平均表现。

## 6 消融研究

在本节中，我们主要在LBF 2玩家2食物的完全合作环境中进行消融实验，因为与MPE和SMAC相比，LBF中的奖励更加稀疏（Papoudakis & Schäfer, [2021](https://arxiv.org/html/2410.03997v1#bib.bib26)）。有关环境的更多信息，请参考[1](https://arxiv.org/html/2410.03997v1#S5.T1 "表1 ‣ 5.2 结果 ‣ 5 实验 ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")。由于篇幅限制，我们还将一些讨论和图表放在附录[B](https://arxiv.org/html/2410.03997v1#A2 "附录B 额外结果 ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")中。

### 6.1 YOLO-MARL 有无策略生成的对比

在本节中，我们检查了策略生成模块对YOLO-MARL框架性能的影响。具体而言，我们比较了标准YOLO-MARL与去除策略生成模块的变种，以评估其重要性。

根据我们的测试，策略生成模块在YOLO-MARL方法中发挥着重要作用。如图[6](https://arxiv.org/html/2410.03997v1#S6.F6 "图6 ‣ 6.1 YOLO-MARL 有无策略生成的对比 ‣ 6 消融研究 ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")所示，在没有LLM生成的策略时，我们得到的规划函数表现较差。有趣的是，在没有LLM生成策略的评估函数的平均回报并不总是接近零，这表明生成的规划函数并非完全错误。基于此，我们可以确认，策略生成模块有助于规划函数生成模块为该游戏提供更好的解决方案。此外，提供策略也有助于稳定生成代码的质量。我们观察到，在没有提供策略的情况下，出现错误函数的风险更高。

![参见说明](img/231c30aff242f198982f47195f7b831a.png)

(a) MADDPG

![参见说明](img/116c6d077a7b6d51ff08348b6fd01928.png)

(b) MAPPO

![参见说明](img/7b32cdce6cbf73c79f2c06fd9b298126.png)

(c) QMIX

图6：YOLO-MARL 在LBF中有无使用LLM生成策略的对比

### 6.2 YOLO-MARL 有无状态解释的对比

为了展示状态解释模块如何增强我们的框架，我们展示了两个失败案例片段：

+   •

    没有解释功能：在提示管道中完全省略了解释功能。

+   •

    直接提供原始环境代码：原始环境源代码直接输入给LLM。

如图[10](https://arxiv.org/html/2410.03997v1#A2.F10 "Figure 10 ‣ B.3 additional results for state interpretation ablation study ‣ Appendix B Additional result ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")所示，LLM无法推断状态类型，如果没有提供预处理代码，它会尝试通过一个不存在的键获取环境信息。如果提供了环境代码但没有为每个组件提供维度上下文，LLM可能会做出随机猜测。在这两种情况下，缺乏明确的状态解释妨碍了LLM生成准确且可执行的规划函数。这些失败突显了状态解释模块在连接向量化观测与LLM对语义上有意义输入的需求之间的关键作用。

通过引入状态解释模块，我们使得LLM能够有效理解环境的状态表示。这促使生成可靠的规划函数，显著提升了我们YOLO-MARL框架的性能。

### 6.3 YOLO-MARL与奖励生成的比较

在本节中，我们将YOLO-MARL方法与采用LLM进行奖励生成但没有奖励函数模板的其他方法进行比较。我们探讨了两种情景：无反馈奖励生成和有反馈奖励生成。

我们的实验表明，单纯依赖LLM生成的奖励函数会导致较差的性能。如图[7](https://arxiv.org/html/2410.03997v1#S6.F7 "Figure 7 ‣ 6.3 Comparison between YOLO-MARL and reward generation ‣ 6 Ablation Study ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")所示，LLM生成的奖励函数对的平均回报始终低于所有三种MARL算法的性能。这表明，在LLM生成的奖励函数下，智能体未能有效学习。然而，我们确实观察到略微的正回报，这表明该框架在奖励塑造任务中的潜力，尤其是在标准MARL算法在稀疏奖励场景中难以学习时。

![参考说明](img/5f0a51ef9d8f45f4b35560b910617353.png)

(a) MADDPG

![参考说明](img/1b9030ae2db600cbfc51ea34e388028d.png)

(b) MAPPO

![参考说明](img/e9e10a74668f850338a720ac397b1103.png)

(c) QMIX

图7：YOLO-MARL与LBF中无反馈奖励生成的比较

为了研究迭代改进是否能提升LLM生成的奖励函数，我们将上一轮迭代生成的奖励函数和其表现反馈提供给LLM。尽管进行了这一迭代过程，LLM仍然未能为LBF环境输出合适的奖励函数。评估的平均回报仍接近零，如图[9](https://arxiv.org/html/2410.03997v1#A2.F9 "Figure 9 ‣ B.2 additional results for reward feedback ‣ Appendix B Additional result ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")所示。每次迭代生成的奖励函数见附录[E](https://arxiv.org/html/2410.03997v1#A5 "Appendix E Reward Generation with feedback ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")。

## 7 限制与未来工作

我们承认YOLO-MARL的表现可能与LLM的能力密切相关，而且我们尚未使用像GPT-o1这样的其他LLM来测试YOLO-MARL，因为这需要tier5用户权限，并且YOLO-MARL在Claude-3.5和GPT-o1之间的表现可能存在差距。

对于未来的工作，我们对大语言模型（LLMs）在进一步提升多智能体强化学习（MARL）方面的潜力充满热情，特别是随着其规划能力的提升。具体来说，我们设想将奖励生成与规划功能相结合，以提升现有MARL算法在完全稀疏环境中的表现。在这种方法中，我们促使LLM生成规划功能和奖励功能，后者替代由环境提供的奖励，遵循第[4](https://arxiv.org/html/2410.03997v1#S4 "4 Methodology ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")节中描述的流程。功能对方法可能需要进一步的改进，我们将把它作为未来的研究方向。在附录[B.4](https://arxiv.org/html/2410.03997v1#A2.SS4 "B.4 additional result on future work ‣ Appendix B Additional result ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")中提供了该框架的初步测试。

## 8 结论

我们提出了YOLO-MARL，这是一个创新框架，利用LLM的高级规划能力来增强MARL政策训练，以适应合作性游戏。通过要求每个环境仅与LLM进行一次交互，YOLO-MARL显著减少了计算开销，并减轻了在训练过程中频繁与LLM交互所带来的不稳定性问题。这种方法不仅超越了传统的MARL算法，还在执行时不依赖于LLM，展示了其在各种环境中的强大泛化能力。

我们在三个不同的环境中评估了YOLO-MARL：MPE环境、LBF环境和SMAC环境。实验表明，YOLO-MARL在与基线MARL方法的比较中表现更好或达到竞争性结果。LLM生成的高层次任务分配功能的整合，有助于在具有较稀疏奖励和较大动作空间的环境中，改善协作任务中的策略学习。最后，我们提到了一种将奖励生成纳入我们框架的可能方法，并计划进一步探索。

## 参考文献

+   Agashe等人（2023）Saaket Agashe、Yue Fan 和 Xin Eric Wang。评估大型语言模型中的多智能体协调能力。*arXiv预印本 arXiv:2310.03903*，2023年。

+   Ahn等人（2022）Michael Ahn、Anthony Brohan、Noah Brown、Yevgen Chebotar、Omar Cortes、Byron David、Chelsea Finn、Chuyuan Fu、Keerthana Gopalakrishnan、Karol Hausman 等人。按我能做的做，而不是按我说的做：将语言与机器人能力相结合。*arXiv预印本 arXiv:2204.01691*，2022年。

+   Carta等人（2023）Thomas Carta、Clément Romac、Thomas Wolf、Sylvain Lamprier、Olivier Sigaud 和 Pierre-Yves Oudeyer。通过在线强化学习将大型语言模型与互动环境相结合。*arXiv预印本 arXiv:2302.02662*，2023年。

+   Chen等人（2024）Weizhe Chen、Sven Koenig 和 Bistra Dilkina。为什么使用大型语言模型解决多智能体路径规划尚未成功。*arXiv预印本 arXiv:2401.03630*，2024年。

+   Chen等人（2023）Yongchao Chen、Jacob Arkin、Yang Zhang、Nicholas Roy 和 Chuchu Fan。使用大型语言模型的可扩展多机器人协作：集中式还是分散式系统？*arXiv预印本 arXiv:2309.15943*，2023年。

+   Du等人（2023）Yuqing Du、Olivia Watkins、Zihan Wang、Cédric Colas、Trevor Darrell、Pieter Abbeel、Abhishek Gupta 和 Jacob Andreas。使用大型语言模型指导强化学习中的预训练。*arXiv预印本 arXiv:2302.06692*，2023年。

+   Fan等人（2022）Linxi Fan、Guanzhi Wang、Yunfan Jiang、Ajay Mandlekar、Yuncong Yang、Haoyi Zhu、Andrew Tang、De-An Huang、Yuke Zhu 和 Anima Anandkumar。Minedojo：构建具有互联网规模知识的开放式具身智能体。*神经信息处理系统进展*，35:18343–18362，2022年。

+   Foerster等人（2017）Jakob N. Foerster、Gregory Farquhar、Triantafyllos Afouras、Nantas Nardelli 和 Shimon Whiteson。反事实多智能体策略梯度。*arXiv预印本 arXiv:1705.08926*，2017年。

+   Guo等人（2024）Taicheng Guo、Xiuying Chen、Yaqi Wang、Ruidi Chang、Shichao Pei、Nitesh V. Chawla、Olaf Wiest 和 Xiangliang Zhang。基于大型语言模型的多智能体：进展与挑战的综述。*arXiv预印本 arXiv:2402.01680*，2024年。

+   Gupta等人（2022）Tarun Gupta、Peter Karkus、Tong Che、Danfei Xu 和 Marco Pavone。强化学习中的语义新颖性基础模型。*arXiv预印本 arXiv:2211.04878*，2022年。

+   洪等人（2023）思锐·洪，夏吴·郑，乔纳森·陈，玉恒·程，金林·王，策尧·张，子理·王，史蒂文·卡·兴·姚，子娟·林，丽扬·周等人。MetaGPT：多智能体协作框架的元编程。*arXiv 预印本 arXiv:2308.00352*，2023年。

+   黄等人（2022）温龙·黄，费·夏，特德·肖，哈里斯·陈，杰基·梁，皮特·佛洛伦斯，安迪·曾，乔纳森·汤普森，伊戈尔·莫达奇，叶夫根·切博塔尔等人。内心独白：通过语言模型进行具身推理的规划。*arXiv 预印本 arXiv:2207.05608*，2022年。

+   坎南等人（2023）夏姆·孙达尔·坎南，维什努南丹·LN·文卡特什，丙哲·闵。Smart-llm：使用大型语言模型进行智能多智能体机器人任务规划。*arXiv 预印本 arXiv:2309.10062*，2023年。

+   坎南等人（2024）夏姆·孙达尔·坎南，维什努南丹·L·N·文卡特什，丙哲·闵。Smart-llm：使用大型语言模型进行智能多智能体机器人任务规划。*arXiv 预印本 arXiv:2309.10062*，2024年。

+   权等人（2023）米娜·权，尚·迈克尔·谢，卡莱莎·布拉德，朵莎·萨迪赫。使用语言模型的奖励设计。*arXiv 预印本 arXiv:2303.00001*，2023年。

+   李等人（2024）郭浩·李，哈桑·哈姆穆德，哈尼·伊塔尼，德米特里·基兹布林，伯纳德·甘内姆。Camel：用于大型语言模型社会“心智”探索的沟通型智能体。*神经信息处理系统进展*，36，2024年。

+   Li 等人（2023）华奥·李，余全崇，西蒙·斯特普提斯，约瑟夫·坎贝尔，达娜·休斯，迈克尔·刘易斯，卡蒂亚·西卡拉。通过大型语言模型进行多智能体协作的心智理论。*arXiv 预印本 arXiv:2310.10701*，2023年。

+   梁等人（2023）杰基·梁，温龙·黄，费·夏，彭·徐，卡罗尔·豪斯曼，布莱恩·伊赫特，皮特·佛洛伦斯，安迪·曾。代码即策略：用于具身控制的语言模型程序。*IEEE国际机器人与自动化会议（ICRA）会议录*，2023年。

+   林等人（2024）方如·林，埃马努埃尔·拉·马尔法，瓦伦丁·霍夫曼，埃尔·米歇尔·杨，安东尼·科恩，珍妮特·B·皮埃尔汉贝尔特。在异步计划推理中增强图的语言模型。*arXiv 预印本 arXiv:2402.02805*，2024年。

+   莉特曼（1994）迈克尔·L·莉特曼。作为多智能体强化学习框架的马尔可夫博弈。*https://www.sciencedirect.com/science/article/pii/B9781558603356500271*，1994年。

+   洛威等人（2020）瑞安·洛威，易·吴，阿维夫·塔马尔，尚·哈布，皮特·阿比尔，伊戈尔·莫达奇。多智能体演员-评论员在混合合作-竞争环境中的应用。*arXiv 预印本 arXiv:1706.02275*，2020年。

+   马等人（2023）魏宇·马，祁睿·米，薛·闫，余桥·吴，润吉·林，海峰·张，君·王。大型语言模型玩星际争霸II：基准测试与链式总结方法。*arXiv 预印本 arXiv:2312.11865*，2023年。

+   马等人（2024）叶程·杰森·马，威廉·梁，关志·王，德安·黄，奥斯伯特·巴斯塔尼，迪内什·贾亚拉曼，余柯·朱，安妮玛·安南德库马尔。EUREKA：通过编码大型语言模型进行人类级奖励设计。*arXiv 预印本 arXiv:2310.12931*，2024年。

+   Nascimento et al. (2023) Nathalia Nascimento, Paulo Alencar, 和 Donald Cowan. 基于大语言模型（LLM）的自适应多代理系统. *IEEE国际自适应计算与自组织系统会议论文集（ACSOS-C）*, 2023.

+   Owen (1982) G. Owen. 博弈论. *https://books.google.com/books?id=pusfAQAAIAAJ*, 1982.

+   Papoudakis & Schäfer (2021) Georgios Papoudakis 和 Lukas Schäfer. 多代理深度强化学习算法在合作任务中的基准测试. *arXiv预印本 arXiv:2006.07869*, 2021.

+   Rashid et al. (2018) Tabish Rashid, Mikayel Samvelyan, Christian Schroeder De Witt, Gregory Farquhar, Jakob Foerster, 和 Shimon Whiteson. QMIX：深度多代理强化学习中的单调值函数分解. *arXiv预印本 arXiv:1803.11485*, 2018.

+   Samvelyan et al. (2019) Mikayel Samvelyan, Tabish Rashid, Christian Schroeder De Witt, Gregory Farquhar, Nantas Nardelli, Tim GJ Rudner, Chia-Man Hung, Philip HS Torr, Jakob Foerster, 和 Shimon Whiteson. 星际争霸多代理挑战赛. *arXiv预印本 arXiv:1902.04043*, 2019.

+   Sun et al. (2024) Chuanneng Sun, Songjun Huang, 和 Dario Pompili. 基于LLM的多代理强化学习：当前和未来的方向. *arXiv预印本 arXiv:2405.11106*, 2024.

+   Sunehag et al. (2017) Peter Sunehag, Guy Lever, Audrunas Gruslys, Wojciech Marian Czarnecki, Vinícius Flores Zambaldi, Max Jaderberg, Marc Lanctot, Nicolas Sonnerat, Joel Z. Leibo, Karl Tuyls, 和 Thore Graepel. 用于合作型多代理学习的值分解网络. *arXiv预印本 arXiv:1706.05296*, 2017.

+   Wang et al. (2024) Ziyan Wang, Meng Fang, Tristan Tomilin, Fei Fang, 和 Yali Du. 带有自然语言约束的安全多代理强化学习. *arXiv预印本 arXiv:2405.20018*, 2024.

+   Xie et al. (2023) Tianbao Xie, Siheng Zhao, Chen Henry Wu, Yitao Liu, Qian Luo, Victor Zhong, Yanchao Yang, 和 Tao Yu. Text2Reward：用于强化学习的自动化密集奖励函数生成. *arXiv预印本 arXiv:2309.11489*, 2023.

+   Yu et al. (2023) Bangguo Yu, Hamidreza Kasaei, 和 Ming Cao. Co-navgpt：基于大语言模型的多机器人协作视觉语义导航. *arXiv预印本 arXiv:2310.07937*, 2023.

+   Yu et al. (2022) Chao Yu, Akash Velu, Eugene Vinitsky, Jiaxuan Gao, Yu Wang, Alexandre Bayen, 和 Yi Wu. PPO在合作性多代理游戏中的惊人有效性. *arXiv预印本 arXiv:2103.01955*, 2022.

+   Zhang et al. (2023) Hongxin Zhang, Weihua Du, Jiaming Shan, Qinhong Zhou, Yilun Du, Joshua B Tenenbaum, Tianmin Shu, 和 Chuang Gan. 使用大语言模型模块化构建合作型具身代理. *arXiv预印本 arXiv:2307.02485*, 2023.

## 附录

## 附录 A 超参数详情

基线算法的详细超参数可以在 Yu 等人（[2022](https://arxiv.org/html/2410.03997v1#bib.bib34)）和 Papoudakis & Schäfer（[2021](https://arxiv.org/html/2410.03997v1#bib.bib26)）的研究中找到。我们提供了整个实验过程中给予 RL 训练的奖励和惩罚值的完整超参数，详细信息见[2](https://arxiv.org/html/2410.03997v1#A1.T2 "表 2 ‣ 附录 A 超参数详情 ‣ YOLO-MARL：多智能体强化学习中的一次性 LLM")。

表 2：超参数

|  | LBF |
| --- | --- |
|  | QMIX | MADDPG | MAPPO |
| --- | --- | --- | --- |
| r’ | 0.02 $\pm$ 0.01 | 0.002 $\pm$ 0.001 | 0.005 $\pm$ 0.004 |
| p’ | 0.02 $\pm$ 0.01 | 0.002 $\pm$ 0.001 | 0.005 $\pm$ 0.004 |
|  | MPE(3agents/4agents) |
| --- | --- |
|  | MADDPG | QMIX |
| --- | --- | --- |
| r’ | 0.2/0.3 $\pm$ 0.1 | 0.2 $\pm$ 0.1/0.2 $\pm$ 0.1 |
| p’ | 0.1/0.2 $\pm$ 0.1 | 0.2 $\pm$ 0.1/0.2 $\pm$ 0.1 |
|  | SMAC (3m/2s_vs_1sc/2c_vs_64zg) |
| --- | --- |
|  | MAPPO |
| r’ | 0.001 $\sim$ 0.01 / 0.02 / 0.003 $\pm$ 0.002 |
| p’ | 0.001 $\sim$ 0.01 / 0.02 / 0.003 $\pm$ 0.002 |

## 附录 B 其他结果

由于页面限制，我们在本节中展示了一些额外的实验和消融研究结果及图表。

### B.1 不同生成函数的比较

考虑到 LLM 输出的变异性，我们评估了生成函数的质量，并比较了 3 种基线方法与使用我们框架的结果。我们在第[1](https://arxiv.org/html/2410.03997v1#S5.T1 "表 1 ‣ 5.2 结果 ‣ 5 实验 ‣ YOLO-MARL：多智能体强化学习中的一次性 LLM")节介绍的 LBF 环境中进行实验。

![参见标题](img/da7a9739f623fbc8ca69a7cb822e6313.png)

(a) MADDPG

![参见标题](img/bfaa3b5284fe258967ee2a23c2a08ef7.png)

(b) MAPPO

![参见标题](img/576e49450ff5acc45b5936bfb41daaa5.png)

(c) QMIX

图 8：在 3 个种子下 LBF 环境的结果：实线表示平均表现，阴影区域表示 3 个不同种子下的范围（最小值到最大值）。

表 3：在 LBF 环境中，YOLO-MARL 和 MARL 在 3 种不同生成的规划函数下的比较。训练期间的最高评估回报值以**粗体**显示。相关结果可以在图[8](https://arxiv.org/html/2410.03997v1#A2.F8 "图 8 ‣ B.1 不同生成函数的比较 ‣ 附录 B 其他结果 ‣ YOLO-MARL：多智能体强化学习中的一次性 LLM")中找到。M 表示百万训练步骤。我们使用两台不同的机器生成规划函数，并在生成规划函数的同一台机器上运行 MARL 和 YOLO-MARL。

|  | 0.2M / 0.4M / 1.5M / 2M 步骤后的平均回报 |
| --- | --- |
|  | QMIX | MADDPG | MAPPO |
| MARL | 0.00/ 0.01/ 0.25/ 0.36 | 0.08/ 0.28/ 0.24/ 0.29 | 0.38/ 0.74/ 0.99/ 0.99 |
| YOLO-MARL | 0.00/ 0.03/ 0.69/ 0.95 | 0.18/ 0.40/ 0.42/ 0.47 | 0.94/ 0.97/ 0.99/ 0.99 |

### B.2 奖励反馈的附加结果

![参见标题](img/1a1372a51bdfa67fe83932057d8375d6.png)

(a) 第一轮迭代

![参见标题](img/f321eb948d2a528a2b446d2d1bbd349d.png)

(b) 第二轮迭代

![参见标题](img/934b66976775cb58dddea658d448d378.png)

(c) 第三轮迭代

![参见标题](img/4d928eb268826b990207dc916f4e1b0a.png)

(d) 第四轮迭代

图9：仅使用奖励生成并带有反馈的 LBF 环境结果。总迭代次数为4，使用的 MARL 算法为 MAPPO。

### B.3 状态解释消融研究的附加结果

![参见标题](img/89afca928e3dd67b852c75d0d9feb837.png)

(a) 失败案例：未提供解释代码

![参见标题](img/cef1b7ffe3ab419286515e7c87c2196d.png)

(b) 失败案例：直接输入环境代码

图10：YOLO-MARL 缺少状态解释模块的失败案例

### B.4 未来工作附加结果

我们测试了这种新方法，利用 YOLO-MARL 在 SMAC 环境下生成规划和奖励函数配对，并在完全稀疏奖励设置下进行测试。在三张 SMAC 地图上测试的基线表现较差，评估胜率始终接近零。然而，正如图[11](https://arxiv.org/html/2410.03997v1#A2.F11 "图11 ‣ B.4 未来工作的附加结果 ‣ 附录B 附加结果 ‣ YOLO-MARL：仅一次大语言模型用于多智能体强化学习")所示，将规划功能纳入奖励生成显著提高了性能。

![参见标题](img/d51a4b2ba41ed658306f706d3e7186b8.png)

(a) 3m 地图的结果

![参见标题](img/f7081d4d5f69897c38996912cacc3d76.png)

(b) 2s 与 1sc 地图的结果

![参见标题](img/437101d91261c3278f370bf6b3c83e9b.png)

(c) 2c 与 64zg 地图的结果

图11：YOLO-MARL在稀疏奖励设置下与规划功能配对生成奖励的结果

## 附录 C 提示详细信息

在本节中，我们提供了贯穿整个研究/应用过程中使用的提示的综合概述，以便于完成各种任务。这些提示在通过提供特定的指令和约束来指导语言模型或智能体行为方面发挥着至关重要的作用。本节详细说明了用于获得本文主体部分所描述结果的提示的准确措辞、格式和上下文。

### C.1 策略

生成策略的提示由环境描述、任务分配类别和指令组成。环境描述包含关于环境的信息，我们仅提供一些必要的描述，说明这个环境是什么样的，任务的目标是什么。我们还会添加一些规则，作为额外的信息或游戏约束，这些规则应遵循，可以在官方网站上找到。任务分配类别可以视为将动作空间或子目标分配给代理的方式，LLM可以在任务过程中为代理分配这些子目标，正式定义可以在[第4.3节](https://arxiv.org/html/2410.03997v1#S4.SS3 "4.3 Planning Function Generation ‣ 4 Methodology ‣ YOLO-MARL: You Only LLM Once for Multi-agent Reinforcement Learning")中找到。指令基本上是告诉LLM应输出什么样的策略。以下是每个环境中最简单场景的示例提示，但其余所有场景的提示格式与这些相似。

基于等级的觅食

{mdframed}[backgroundcolor=gray!10, linecolor=black] 环境描述：这个基于等级的觅食（LBF）多代理强化学习环境包含2个代理和2个食物项。你的目标是使代理协作并拾取环境中的所有食物。

游戏规则：

1.  1.

    拾取动作成功的条件是所有代理一起拾取相同的目标。

1.  2.

    拾取动作只有在代理的级别总和等于或高于食物的级别时才算成功。

1.  3.

    只有当代理与食物之间的距离小于或等于1时，才允许执行拾取动作。

1.  4.

    成功条件：必须在{time_steps}步之前拾取所有食物。

任务分配：每个代理的可用任务：

1.  1.

    目标食物 0

1.  2.

    目标食物 1

1.  3.

    拾取

指令格式：以下是生成策略的一般指导原则：

1.  1.

    目标或目的：清晰地说明任务的整体目标。

1.  2.

    问题或需求：考虑不同的场景，识别任务计划所解决的关键问题或需求。

1.  3.

    方法 / 方法论：逐步描述将要遵循的整体方法或步骤。

1.  4.

    场景分析：考虑代理在任务执行过程中可能遇到的不同场景，以及它们如何协调以适应变化。

1.  5.

    任务拆解：拆解任务，详细说明每个代理的角色和责任，以及它们如何协调以实现整体目标。

多代理粒子环境

{mdframed}[backgroundcolor=gray!10, linecolor=black] 环境描述：这个多代理粒子环境（MPE）是一个多代理强化学习环境，包含3个代理和3个地标。你的目标是使代理协作并覆盖所有地标。游戏规则：

1.  1.

    代理必须通过最小化每个地标之间的距离来覆盖所有地标，每个代理前往一个独特的地标。

1.  2.

    智能体不能与其他智能体碰撞。碰撞阈值为0.3。

任务分配：为每个智能体分配的可用任务：

1.  1.

    地标 0

1.  2.

    地标 1

1.  3.

    地标 2

1.  4.

    无操作

指令格式：以下是生成策略的一般指南：

1.  1.

    目标或目的：清晰地陈述任务的整体目标。

1.  2.

    问题或需求：考虑不同的场景并识别任务计划所解决的关键问题或需求。

1.  3.

    方法 / 方法论：逐步描述将要遵循的总体方法或步骤。

1.  4.

    场景分析：考虑智能体在执行任务过程中可能遇到的不同场景，以及它们如何协调适应。

1.  5.

    任务分解：分解任务，详细说明每个智能体的角色和责任，以及它们如何协调以实现整体目标。

星际争霸多智能体挑战环境

{mdframed}[backgroundcolor=gray!10, linecolor=black] 环境描述：这个SMAC 3m地图有3个Terran海军陆战队智能体和3个Terran海军陆战队敌人。智能体单位是海军陆战队，其特点是海军陆战队是远程单位，可以攻击地面和空中单位。它们是Terran的基本作战单位，战斗中非常多才多艺。你的任务是利用单位信息在60步内赢得战斗场景。

游戏规则：

1.  1.

    射程为6，视距为9，适用于智能体和敌人。

1.  2.

    成功条件：在回合结束之前消灭所有敌方单位。

1.  3.

    失败条件：如果智能体在60步内没有足够的攻击性杀死所有敌人以赢得胜利，或者如果所有智能体都死亡。

任务分配：为每个智能体分配的可用任务：

1.  1.

    移动

1.  2.

    攻击

1.  3.

    停止

1.  4.

    无（仅适用于死亡智能体）

指令格式：以下是生成策略的一般指南：

1.  1.

    目标或目的：清晰地陈述任务的整体目标。

1.  2.

    问题或需求：考虑不同的场景并识别任务计划所解决的关键问题或需求。

1.  3.

    方法 / 方法论：逐步描述将要遵循的总体方法或步骤。

1.  4.

    场景分析：考虑智能体在执行任务过程中可能遇到的不同场景，以及它们如何协调适应。

1.  5.

    任务分解：分解任务，详细说明每个智能体的角色和责任，以及它们如何协调以实现整体目标。

### C.2 解释函数

这里列出了处理原始向量观察的每个场景的解释函数。

LBF 2玩家2食物场景

[⬇](data:text/plain;base64,ZGVmIHByb2Nlc3Nfc3RhdGUob2JzZXJ2YXRpb25zLCBwPTIsIGY9Mik6CiAgICAnJycKICAgIFBhcmFtOgogICAgICAgIG9ic2VydmF0aW9uOgogICAgICAgICAgICAgICAgICAgICAgICBhcnJheSBvZiBhcnJheSAocCwgbik6IGRpY3QoJ2FnZW50XzAnLCAnYWdlbnRfMScsIC4uLiwgJ2FnZW50X3AnKQogICAgICAgICAgICAgICAgICAgICAgICBMaXN0OgogICAgICAgICAgICAgICAgICAgICAgICBBZ2VudCA6IChuLCApIGxpc3Qgb2Ygb2JzZXJ2YXRpb24gY29tcG9uZW50cwogICAgICAgIHA6IGludCwgbnVtYmVyIG9mIGFnZW50cwogICAgICAgIGY6IGludCwgbnVtYmVyIG9mIGZvb2RzIGluIHRoZSBlbnZpcm9ubWVudAogICAgUmV0dXJuOgogICAgICAgIG9iczogdHVwbGVzIChmb29kX2luZm8sIGFnZW50c19pbmZvKToKICAgICAgICAgICAgZm9vZF9pbmZvOiBkaWN0aW9uYXJ5IHRoYXQgY29udGFpbnMgaW5mb3JtYXRpb24gYWJvdXQgZm9vZCBpbiB0aGUgZW52aXJvbm1lbnQKICAgICAgICAgICAgICAgICAgICAgICAga2V5OiBmb29kX2lkICgnZm9vZF8wJywgJ2Zvb2RfMScsIC4uLikKICAgICAgICAgICAgICAgICAgICAgICAgdmFsdWU6IHR1cGxlcyAoZm9vZF9wb3MsIGZvb2RfbGV2ZWwpIG9yIE5vbmUgaWYgdGhlIGZvb2QgaXMgYWxyZWFkeSBiZWVuIHBpY2tlZCB1cAogICAgICAgICAgICBhZ2VudHNfaW5mbzogZGljdGlvbmFyeSB0aGF0IGNvbnRhaW5zIGluZm9ybWF0aW9uIGFib3V0IGFnZW50cyBpbiB0aGUgZW52aXJvbm1lbnQKICAgICAgICAgICAgICAgICAgICAgICAga2V5OiBhZ2VudF9pZCAoJ2FnZW50XzAnLCAnYWdlbnRfMScsIC4uLikKICAgICAgICAgICAgICAgICAgICAgICAgdmFsdWU6IHR1cGxlcyAoYWdlbnRfcG9zLCBhZ2VudF9sZXZlbCkKICAgICcnJwogICAgZm9vZF9pbmZvID0ge30KICAgIGFnZW50c19pbmZvID0ge30KICAgIG9icyA9IG9ic2VydmF0aW9uc1swXQogICAgb2Zmc2V0ID0gMAogICAgZm9yIGZvb2RfaWR4IGluIHJhbmdlKGYpOgogICAgICAgIGZvb2Rfb2JzID0gb2JzW29mZnNldDpvZmZzZXQrM10KICAgICAgICBvZmZzZXQgKz0gMwogICAgICAgIGN1cnJfZm9vZF9wb3MgPSBmb29kX29ic1s6Ml0KICAgICAgICBjdXJyX2Zvb2RfbGV2ZWwgPSBmb29kX29ic1syXQogICAgICAgIGZvb2RfaWQgPSBmJ2Zvb2Rfe2Zvb2RfaWR4fScKICAgICAgICAjIElmIGZvb2QgbGV2ZWwgaXMgMCwgdGhlbiB0aGUgZm9vZCBpcyBhbHJlYWR5IGJlZW4gcGlja3VwIGFuZCBub3QgcHJlc2VudCBpbiB0aGUgZW52aXJvbm1lbnQKICAgICAgICBpZiBjdXJyX2Zvb2RfbGV2ZWwgPT0gMCBhbmQgY3Vycl9mb29kX3Bvc1swXSA8IDA6CiAgICAgICAgICAgIGZvb2RfaW5mb1tmb29kX2lkXSA9IE5vbmUKICAgICAgICAjIFRoZSBmb29kIGlzIHByZXNlbnQgaW4gdGhlIGVudmlyb25tZW50CiAgICAgICAgZWxzZToKICAgICAgICAgICAgZm9vZF9pbmZvW2Zvb2RfaWRdID0gKGN1cnJfZm9vZF9wb3MsIGN1cnJfZm9vZF9sZXZlbCkKCiAgICBmb3IgYWdlbnRfaWR4IGluIHJhbmdlKHApOgogICAgICAgIGFnZW50X29icyA9IG9ic1tvZmZzZXQ6b2Zmc2V0KzNdCiAgICAgICAgb2Zmc2V0ICs9IDMKICAgICAgICBjdXJyX2FnZW50X3BvcyA9IGFnZW50X29ic1s6Ml0KICAgICAgICBjdXJyX2FnZW50X2xldmVsID0gYWdlbnRfb2JzWzJdCiAgICAgICAgYWdlbnRfaWQgPSBmJ2FnZW50X3thZ2VudF9pZHh9JwogICAgICAgIGFnZW50c19pbmZvW2FnZW50X2lkXSA9IChjdXJyX2FnZW50X3BvcywgY3Vycl9hZ2VudF9sZXZlbCkKCiAgICByZXR1cm4gZm9vZF9pbmZvLCBhZ2VudHNfaW5mbw==)1def  process_state(observations,  p=2,  f=2):2  ’’’3  Param:4  observation:5  array  of  array  (p,  n):  dict(’agent_0’,  ’agent_1’,  ...,  ’agent_p’)6  List:7  Agent  :  (n,  )  list  of  observation  components8  p:  int,  number  of  agents9  f:  int,  number  of foods  in  the  environment10  Return:11  obs:  tuples  (food_info,  agents_info):12  food_info:  dictionary  that  contains  information  about  food  in  the  environment13  key:  food_id  (’food_0’,  ’food_1’,  ...)14  value:  tuples  (food_pos,  food_level)  or  None  if  the  food  is  already  been  picked  up15  agents_info:  dictionary  that  contains  information  about  agents  in  the  environment16  key:  agent_id  (’agent_0’,  ’agent_1’,  ...)17  value:  tuples  (agent_pos,  agent_level)18  ’’’19  food_info  =  {}20  agents_info  =  {}21  obs  =  observations[0]22  offset  =  023  for  food_idx  in  range(f):24  food_obs  =  obs[offset:offset+3]25  offset  +=  326  curr_food_pos  =  food_obs[:2]27  curr_food_level  =  food_obs[2]28  food_id  =  f’food_{food_idx}’29  #  If  food  level  is  0,  then  the  food  is  already  been  pickup  and  not  present  in  the  environment30  if  curr_food_level  ==  0  and  curr_food_pos[0]  <  0:31  food_info[food_id]  =  None32  #  The  food  is  present  in  the  environment33  else:34  food_info[food_id]  =  (curr_food_pos,  curr_food_level)3536  for  agent_idx  in  range(p):37  agent_obs  =  obs[offset:offset+3]38  offset  +=  339  curr_agent_pos  =  agent_obs[:2]40  curr_agent_level  =  agent_obs[2]41  agent_id  =  f’agent_{agent_idx}’42  agents_info[agent_id]  =  (curr_agent_pos,  curr

MPE 3 个智能体场景

[⬇](data:text/plain;base64,ZGVmIHByb2Nlc3Nfc3RhdGUob2JzZXJ2YXRpb25zLCBOPTMpOgogICAgJycnCiAgICBQYXJhbToKICAgICAgICBvYnNlcnZhdGlvbnM6CiAgICAgICAgICAgIExpc3Qgb2YgTnVtUHkgYXJyYXlzLCBvbmUgcGVyIGFnZW50LgogICAgICAgICAgICBFYWNoIGFycmF5IHJlcHJlc2VudHMgdGhlIG9ic2VydmF0aW9uIGZvciBhbiBhZ2VudDoKICAgICAgICAgICAgW3NlbGZfdmVsICgyLCksIHNlbGZfcG9zICgyLCksIGxhbmRtYXJrX3JlbF9wb3NpdGlvbnMgKE4qMiwpLCBvdGhlcl9hZ2VudF9yZWxfcG9zaXRpb25zICgoTi0xKSoyLCksIGNvbW11bmljYXRpb25dCgogICAgUmV0dXJuOgogICAgICAgIG9iczoKICAgICAgICAgICAgRGljdGlvbmFyeSB3aXRoIGFnZW50IElEcyBhcyBrZXlzICgnYWdlbnRfMCcsICdhZ2VudF8xJywgLi4uKS4KICAgICAgICAgICAgRWFjaCB2YWx1ZSBpcyBhIGxpc3QgY29udGFpbmluZzoKICAgICAgICAgICAgICAgIC0gTGFuZG1hcmsgcmVsYXRpdmUgcG9zaXRpb25zOiBOIGFycmF5cyBvZiBzaGFwZSAoMiwpCiAgICAgICAgICAgICAgICAtIE90aGVyIGFnZW50cycgcmVsYXRpdmUgcG9zaXRpb25zOiAoTi0xKSBhcnJheXMgb2Ygc2hhcGUgKDIsKQogICAgJycnCiAgICBvYnMgPSB7fQogICAgbnVtX2FnZW50cyA9IGxlbihvYnNlcnZhdGlvbnMpCgogICAgZm9yIGlkeCwgYWdlbnRfb2JzIGluIGVudW1lcmF0ZShvYnNlcnZhdGlvbnMpOgogICAgICAgIGFnZW50X2lkID0gZidhZ2VudF97aWR4fScKICAgICAgICBvYnNbYWdlbnRfaWRdID0gW10KCiAgICAgICAgIyBFeHRyYWN0IGxhbmRtYXJrIHJlbGF0aXZlIHBvc2l0aW9ucwogICAgICAgIGZvciBpIGluIHJhbmdlKE4pOgogICAgICAgICAgICBzdGFydCA9IDQgKyAyICogaQogICAgICAgICAgICBlbmUgPSBzdGFydCArIDIKICAgICAgICAgICAgbGFuZF8yX2EgPSBhZ2VudF9vYnNbc3RhcnQ6ZW5kXQogICAgICAgICAgICBvYnNbYWdlbnRfaWRdLmFwcGVuZChsYW5kXzJfYSkKCiAgICAgICAgIyBFeHRyYWN0IG90aGVyIGFnZW50cycgcmVsYXRpdmUgcG9zaXRpb25zCiAgICAgICAgZm9yIGkgaW4gcmFuZ2UobnVtX2FnZW50cyAtIDEpOgogICAgICAgICAgICBzdGFydCA9IDQgKyAyICogTiArIDIgKiBpCiAgICAgICAgICAgIGVuZCA9IHN0YXJ0ICsgMgogICAgICAgICAgICBvdGhlcl9hZ2VudF8yX2EgPSBhZ2VudF9vYnNbc3RhcnQ6ZW5kXQogICAgICAgICAgICBvYnNbYWdlbnRfaWRdLmFwcGVuZChvdGhlcl9hZ2VudF8yX2EpCgogICAgcmV0dXJuIG9icw==)1def  process_state(observations,  N=3):2  ’’’3  Param:4  observations:5  List  of  NumPy  arrays,  one  per  agent.6  Each  array  represents  the  observation  for  an  agent:7  [self_vel  (2,),  self_pos  (2,),  landmark_rel_positions  (N*2,),  other_agent_rel_positions  ((N-1)*2,),  communication]89  Return:10  obs:11  Dictionary  with  agent  IDs  as  keys  (’agent_0’,  ’agent_1’,  ...).12  Each  value  is  a  list  containing:13  -  Landmark  relative  positions:  N  arrays  of  shape  (2,)14  -  Other  agents’  relative  positions:  (N-1)  arrays  of  shape  (2,)15  ’’’16  obs  =  {}17  num_agents  =  len(observations)1819  for  idx,  agent_obs  in  enumerate(observations):20  agent_id  =  f’agent_{idx}’21  obs[agent_id]  =  []2223  #  Extract  landmark  relative  positions24  for  i  in  range(N):25  start  =  4  +  2  *  i26  end  =  start  +  227  land_2_a  =  agent_obs[start:end]28  obs[agent_id].append(land_2_a)2930  #  Extract  other  agents’  relative  positions31  for  i  in  range(num_agents  -  1):32  start  =  4  +  2  *  N  +  2  *  i33  end  =  start  +  234  other_agent_2_a  =  agent_obs[start:end]35  obs[agent_id].append(other_agent_2_a)3637  return  obs

MPE 4 个智能体场景

[⬇](data:text/plain;base64,ZGVmIHByb2Nlc3Nfc3RhdGUob2JzZXJ2YXRpb25zLCBOPTQpOgogICAgJycnCiAgICBQYXJhbToKICAgICAgICBvYnNlcnZhdGlvbnM6CiAgICAgICAgICAgIExpc3Qgb2YgTnVtUHkgYXJyYXlzLCBvbmUgcGVyIGFnZW50LgogICAgICAgICAgICBFYWNoIGFycmF5IHJlcHJlc2VudHMgdGhlIG9ic2VydmF0aW9uIGZvciBhbiBhZ2VudDoKICAgICAgICAgICAgW3NlbGZfdmVsICgyLCksIHNlbGZfcG9zICgyLCksIGxhbmRtYXJrX3JlbF9wb3NpdGlvbnMgKE4qMiwpLCBvdGhlcl9hZ2VudF9yZWxfcG9zaXRpb25zICgoTi0xKSoyLCksIGNvbW11bmljYXRpb25dCgogICAgUmV0dXJuOgogICAgICAgIG9iczoKICAgICAgICAgICAgRGljdGlvbmFyeSB3aXRoIGFnZW50IElEcyBhcyBrZXlzICgnYWdlbnRfMCcsICdhZ2VudF8xJywgLi4uKS4KICAgICAgICAgICAgRWFjaCB2YWx1ZSBpcyBhIGxpc3QgY29udGFpbmluZzoKICAgICAgICAgICAgICAgIC0gTGFuZG1hcmsgcmVsYXRpdmUgcG9zaXRpb25zOiBOIGFycmF5cyBvZiBzaGFwZSAoMiwpCiAgICAgICAgICAgICAgICAtIE90aGVyIGFnZW50cycgcmVsYXRpdmUgcG9zaXRpb25zOiAoTi0xKSBhcnJheXMgb2Ygc2hhcGUgKDIsKQogICAgJycnCiAgICBvYnMgPSB7fQogICAgbnVtX2FnZW50cyA9IGxlbihvYnNlcnZhdGlvbnMpCgogICAgZm9yIGlkeCwgYWdlbnRfb2JzIGluIGVudW1lcmF0ZShvYnNlcnZhdGlvbnMpOgogICAgICAgIGFnZW50X2lkID0gZyddYWdlbnRfMCcsICdhZ2VudF8xJywgLi4uKS4KICAgICAgICBvYnNbYWdlbnRfaWRdID0gW10KCiAgICAgICAgIyBFeHRyYWN0IGxhbmRtYXJrIHJlbGF0aXZlIHBvc2l0aW9ucwogICAgICAgIGZvciBpIGluIHJhbmdlKE4pOgogICAgICAgICAgICBzdGFydCA9IDQgKyAyICogaQogICAgICAgICAgICBlbmUgPSBzdGFydCArIDIKICAgICAgICAgICAgbGFuZF8yX2EgPSBhZ2VudF9vYnNbc3RhcnQ6ZW5kXQogICAgICAgICAgICBvYnNbYWdlbnRfaWRdLmFwcGVuZChsYW5kXzJfYSkKCiAgICAgICAgIyBFeHRyYWN0IG90aGVyIGFnZW50cycgcmVsYXRpdmUgcG9zaXRpb25zCiAgICAgICAgZm9yIGkgaW4gcmFuZ2UobnVtX2FnZW50cyAtIDEpOgogICAgICAgICAgICBzdGFydCA9IDQgKyAyICogTiArIDIgKiBpCiAgICAgICAgICAgIGVuZCA9IHN0YXJ0ICsgMgogICAgICAgICAgICBvdGhlcl9hZ2VudF8yX2EgPSBhZ2VudF9vYnNbc3RhcnQ6ZW5kXQogICAgICAgICAgICBvYnNbYWdlbnRfaWRdLmFwcGVuZChvdGhlcl9hZ2VudF8yX2EpCgogICAgcmV0dXJuIG9icw==)1def 过程状态(观察, N=4):2  ’’’3  参数:4  观察:5  每个代理的 NumPy 数组列表。6  每个数组代表一个代理的观察:7  [自我速度  (2,),  自我位置  (2,),  地标相对位置  (N*2,),  其他代理相对位置  ((N-1)*2,),  通信]89  返回:10  obs:11  字典，代理 ID 作为键（‘agent_0’，‘agent_1’， ...）。12  每个值是一个包含以下内容的列表:13  -  地标相对位置: N 个形状为 (2,) 的数组14  -  其他代理的相对位置: (N-1) 个形状为 (2,) 的数组15  ’’’16  obs  =  {}17  num_agents  =  len(观察)1819  对于 idx, agent_obs 在 enumerate(观察):20  agent_id  =  f’agent_{idx}’21  obs[agent_id]  =  []2223  #  提取地标相对位置24  对于 i 在 range(N):25  start  =  4  +  2  *  i26  end  =  start  +  227  land_2_a  =  agent_obs[start:end]28  obs[agent_id].append(land_2_a)2930  #  提取其他代理的相对位置31  对于 i 在 range(num_agents  -  1):32  start  =  4  +  2  *  N  +  2  *  i33  end  =  start  +  234  other_agent_2_a  =  agent_obs[start:end]35  obs[agent_id].append(other_agent_2_a)3637  返回  obs

SMAC 3m 地图

[⬇](data:text/plain;base64,ZGVmIHByb2Nlc3NfZ2xvYmFsX3N0YXRlKGdsb2JhbF9zdGF0ZSwgbj0zLCBtPTMpOgogICAg'')

SMAC 2秒与1秒地图

[⬇](data:text/plain;base64,ZGVmIHByb2Nlc3NfZ2xvYmFsX3N0YXRlKG9ic2VydmF0aW9ucywgbj0yLCBtPTEpOgogICAg'==' 

SMAC 2c 对比 64zg 地图

[⬇](data:text/plain;base64,ZGVmIHByb2Nlc3NfZ2xvYmFsX3N0YXRlKG9ic2VydmF0aW9ucywgbj0yLCBtPTY0KToKICAgICcnJwogICAgUGFyYW06CiAgICAgICAgb2JzZXJ2YXRpb246CiAgICAgICAgICAgICAgICAgICAgICAgIERpY3Qgb2YgbGlzdCBvZiAobiwgKTogZGljdCgnYWdlbnRfMCcsICdhZ2VudF8xJywgLi4uLCAnYWdlbnRfTicpCiAgICAgICAgICAgICAgICAgICAgICAgIExpc3Q6CiAgICAgICAgICAgICAgICAgICAgICAgIEFnZW50IDogKG0sICkgbGlzdCBvZiBvYnNlcnZhdGlvbiBjb21wb25lbnRzCiAgICAgICAgbjogaW50LCBudW1iZXIgb2YgYWdlbnRzCiAgICAgICAgbTogaW50LCBudW1iZXIgb2YgZW5lbWllcwogICAgUmV0dXJuOgogICAgICAgIG9icyAodHVwbGVzIG9mIGRpY3QpOiBUdXBsZXMgb2YgZGljdCBvZiAobiwgKTogVHVwbGUgb2YgZWFjaCBvYnNlcnZhdGlvbiBjb21wb25lbnRzIHByb2Nlc3NlZCBmcm9tIGVhY2ggYWdlbnQncyBwZXJzcGVjdGl2ZSBieSBmdW5jdGluYSAicHJvY2Vzc19vYnNlcnZhdGlvbiI6CiAgICAgICAgICAgIG1vdmVfZmVhdHMgKGRpY3Qgb2YgbGlzdCk6IERpY3Qgb2YgbGlzdCBvZiAobiwgKTogZGljdCgnYWdlbnRfMCcsICdhZ2VudF8xJywgLi4uLCAnYWdlbnRfTicpIExpc3Qgb2YgYXZhaWxhYmxlIG1vdmVzIGZvciBlYWNoIGFnZW50LgogICAgICAgICAgICBlbmVteV9pbmZvIChkaWN0IG9mIGRpY3Qgb2YgdHVwbGUpOiBEaWN0IG9mIGRpY3Qgb2YgdHVwbGUgb2YgKG4sICk6IGRpY3QoJ2FnZW50XzAnLCAnYWdlbnRfMScsIC4uLiwgJ2FnZW50X04nKSBUdXBsZSBvZiBtIGVuZW1pZXMgaW5mb3JtYXRpb24oZW5lbXlfMCB0byBlbmVteV9tKSBmb3IgZWFjaCBhZ2VudC4KICAgICAgICAgICAgYWxseV9pbmZvIChkaWN0IG9mIGRpY3Qgb2YgdHVwbGUpOiBEaWN0IG9mIGRpY3Qgb2YgdHVwbGUgb2YgKG4sICk6IGRpY3QoJ2FnZW50XzAnLCAnYWdlbnRfMScsIC4uLiwgJ2FnZW50X04nKSBUdXBsZSBvZiBuLTEgYWxseSBpbmZvcm1hdGlvbihleGNsdWRlIHNlbGYpIGZvciBlYWNoIGFnZW50LgogICAgICAgICAgICBvd25faW5mbyAoZGljdCBvZiB0dXBsZSk6IERpY3Qgb2YgdHVwbGUgb2YgKG4sICk6IGRpY3QoJ2FnZW50XzAnLCAnYWdlbnRfMScsIC4uLiwgJ2FnZW50X04nKSBUdXBsZSBvZiBvd24gaW5mb3JtYXRpb24gZm9yIGVhY2ggYWdlbnQuCiAgICAnJycKICAgIG1vdmVfZmVhdHMgPSB7fQogICAgZW5lbXlfaW5mbyA9IHt9CiAgICBhbGx5X2luZm8gPSB7fQogICAgb3duX2luZm8gPSB7fQogICAgYWN0aW9uX251bSA9IDYrbQogICAgZm9yIGlkLCBvYnMgaW4gZW51bWVyYXRlKG9ic2VydmF0aW9ucyk6CiAgICAgICAgYWdlbnRfaWQgPSBmImFnZW50X3tpZH0iCiAgICAgICAgb2Zmc2V0ID0gMAogICAgICAgIGFsX2lkcyA9IFtmImFnZW50X3thbF9pZH0iIGZvciBhbF9pZCBpbiByYW5nZShuKSBpZiBmImFnZW50X3thbF9pZIH0iICE9IGFnZW50X2lkXQogICAgICAgIGFsbHlfaW5mb1thZ2VudF9pZF0gPSB7fQogICAgICAgIGZvciBhbF9pZCBpbiBhbF9pZHM6CiAgICAgICAgICAgICMgd2hldGhlciB0aGUgYWxseSBpcyB2aXNpYmxlIG9yIGluIHRoZSBzaWdodCByYW5nZSBvZiB0aGUgYWdlbnQKICAgICAgICAgICAgaXNfY3VycmVudF9hbGx5X3Zpc2libGUgPSBvYnNbb2Zmc2V0OiBvZmZzZXQgKyAxXQogICAgICAgICAgICBvZmZzZXQgKz0gMQogICAgICAgICAgICAjIGRpc3RhbmNlIHRvIHRoZSBhbGx5CiAgICAgICAgICAgIGRpc3RfdG9fYWxseSA9IG9ic1tvZmZzZXQ6IG9mZnNldCArIDFdCiAgICAgICAgICAgIG9mZnNldCArPSAxCiAgICAgICAgICAgICMgYWxseSdzIHBvc2l0aW9uIHJlbGF0aXZlIHRvIHRoZSBhZ2VudAogICAgICAgICAgICBwb3NfeF90b19hbGx5ID0gb2JzW29mZnNldDogb2Zmc2V0ICsgMV0KICAgICAgICAgICAgcG9zX3lfdG9fYWxseSA9IG9ic1tvZmZzZXQgKyAxOiBvZmZzZXQgKyAyXQogICAgICAgICAgICBvZmZzZXQgKz0gMgogICAgICAgICAgICAjIHdoZXRoZXIgdGhlIHRpbWUgbGVmdCBmb3IgdGhlIGFsbHkgb3VzZSB0byB1c2UgdGhlIHdlYXBvbgogICAgICAgICAgICB3ZWFwb25fY29vbGRvd24gPSBvYnNbb2Zmc2V0OiBvZmZzZXQgKyAxXQogICAgICAgICAgICBvZmZzZXQgKz0gMQogICAgICAgICAgICMgZGVsaXZlZCBvZmZzZXQgZGlyZWN0aW9ucyB0byB0aGUgd29ya2luZyByYW5nZSBvZiB0aGUgZ2VudA==)

示例输出

我们提供了一些由LLM生成的简单场景策略示例。

{mdframed}[backgroundcolor=gray!10, linecolor=black] 根据环境描述和游戏规则，我将提供一个专注于激进打法的策略，在60步限制内消灭所有敌方单位。以下是简洁、清晰且可执行的指示：

1. 目标：在60步内消灭所有3个敌方海军陆战队单位。

2. 问题：我们需要克服战斗的对称性（3v3海军陆战队），并利用任何位置优势迅速取胜。

3. 方法：采取激进的策略，协调攻击，集中火力逐个击败敌人。

4. 观察与任务分解：

a) 初始布置（第1-5步）： 

b) 交战（第6-20步）： 

c) 集中火力（第21-40步）： 

d) 适应并消灭（第41-60步）： 

在整个交战过程中： 

该策略强调激进的玩法、协调的攻击以及适应性，以克服战斗的对称性并在时间限制内取得胜利。

### C.3 更多的规划功能生成提示

为确保函数生成有效地与强化学习（RL）训练集成，我们提供了一个详细的提示，其中包含函数的目标和预期格式。该提示旨在指导函数的创建，确保它们促进代理之间的合作与协调。提示如下：{mdframed}[backgroundcolor=gray!10, linecolor=black]您的任务是创建一个规划函数和一个奖励函数，它们共同作用于提高代理之间的合作。规划函数应帮助每个代理达成目标，而奖励函数应鼓励顺畅的协作。两个函数应遵循提示中的指导，重点确保代理协调他们的动作，以同时达成目标。

环境代码信息如下所示：

def process_global_state(global_state, n=3, m=3):

…

返回 processed_global_state

函数生成的格式如下：

规划函数应如下所示：

def planning_function(processed_global_state, available_actions):

”””

基于当前战斗状态，确定每个代理的最优任务。

参数：

processed_global_state：一个元组，包含（available_move_actions，enemy_info，ally_info，own_info）

available_actions：每个代理可用的动作索引的字典

返回：

llm_tasks：包含每个代理的最优任务的字典（任务分配类）

”””

…

返回 llm_tasks

返回的‘ll_tasks’应符合指定的任务分配类。使用‘processed_global_state’来通知决策。

奖励函数应如下所示：

def compute_reward(processed_global_state, llm_tasks, tasks):

”””

根据分配的任务及其结果计算奖励。

参数：

processed_global_state：从函数 process_global_state(global_state, n, m) 返回，llm_tasks (dict)：包含分配给每个代理的任务的字典。

tasks (dict)：每个代理实际执行的任务的字典，例如：’agent_0’：…

返回：

reward：包含每个代理奖励的字典。例如：’agent_0’：reward1，’agent_1’：reward2，…

”””

…

返回奖励

应根据提示中建议的优先级调整每个组件的奖励值。

您可以使用或导入任何必要的API进行代码生成，但不要写入类对象。

生成的函数应仅包含‘planning_function’和‘compute_reward’。不要创建新变量或子函数。

严格遵守动作空间和‘processed_global_state’的大小、形状和格式。

在生成这两个函数之前，逐步思考。首先，考虑在‘processed_global_state’中提供的信息以及如何在函数中使用它。其次，分析环境描述，确定在此场景中每个代理的适当策略和任务分配。

确保这些功能不仅能正确工作，还能根据指令最大化代理的协调性。

通过提供此提示，我们的目标是生成不仅能在强化学习（RL）框架内正确运行，而且能根据提供的指令最大化代理协调性的功能。这种方法确保代理能够有效地协同工作，最终提升多代理系统的整体性能。

## 附录 D 生成规划功能示例

[⬇](data:text/plain;base64,aW1wb3J0IG51bXB5IGFzIG5wCgpkZWYgcGxhbm5pbmdfZnVuY3Rpb24ocHJvY2Vzc2VkX3N0YXRlKToKICAgICIiIgogICAg根据当前状态确定每个代理的最佳任务。 

## 附录 E 带反馈的奖励生成

这里提供的功能是生成的奖励函数，包括：迭代 1、迭代 2、迭代 3、迭代 4。

迭代 1：

[⬇](data:text/plain;base64,ZGVmIGNvbXB1dGVfcmV3YXJkKHByb2Nlc3NlZF9zdGF0ZSwgYWN0aW9ucyk6CiAgICAiIiIKICAgIENhbGN1bGF0ZSByZXdhcmRzIGJhc2VkIG9uIHRoZSB0YXNrcyBhc3NpZ25lZCBhbmQgdGhlaXIgb3V0Y29tZXMuCgogICAgQXJnczoKICAgICAgICBwcm9jZXNzZWRfc3RhdGU6IHJldHVybmVkIGZyb20gZnVuY3Rpb24gcHJvY2Vzc19zdGF0ZShzdGF0ZSwgcCwgZikKICAgICAgICBhY3Rpb25zIChkaWN0KTogZGljdGlvbmFyeSBvZiBhIGludGVnZXIgYWN0aW9uIHRoYXQgYWN0dWFsbHkgcGVyZm9ybSBieSBlYWNoIGFnZW50LiBFLmcuIHsiYWdlbnRfMCI6IDIsICJhZ2VudF8xIjogNCwgLi4ufQoKICAgIFJldHVybnM6CiAgICAgICAgcmV3YXJkOiBEaWN0IGNvbnRhaW5pbmcgcmV3YXJkcyBmb3IgZWFjaCBhZ2VudC4gRm9yIGV4YW1wbGU6IHsnYWdlbnRfMCc6IHJld2FyZDEsICdhZ2VudF8xJywgcmV3YXJkMiwgLi4ufQogICAgIiIiCiAgICBmb29rX2luZm8sIGFnZW50c19pbmZvID0gcHJvY2Vzc2VkX3N0YXRlCiAgICByZXdhcmQgPSB7YWdlbnRfaWQ6IDAgZm9yIGFnZW50X2lkIGluIGFnZW50c19pbmZvLmtleXMoKX0KCiAgICAjIFJld2FyZCBmb3IgcGlja2luZyB1cCBmb29rCiAgICBwaWNrdXBfYWdlbnRzID0gW2FnZW50X2lkIGZvciBhZ2VudF9pZCwgYWN0aW9uIGluIGFjdGlvbnMuaXRlbXMoKSBpZiBhY3Rpb24gPT0gNV0KICAgIGlmIHBpY2t1cF9hZ2VudHM6CiAgICAgICAgZm9vZF9wb3NpdGlvbnMgPSBbZm9vZFswXSBmb3IgZm9vZCBpbiBmb29rX2luZm8udmFsdWVzKCkgaWYgZm9vZCBpcyBub3QgTm9uZV0KICAgICAgICBmb3IgYWdlbnRfaWQgaW4gcGlja3VwX2FnZW50czoKICAgICAgICAgICAgYWdlbnRfcG9zID0gYWdlbnRzX2luZm9bYWdlbnRfaWRdWzBdCiAgICAgICAgICAgIGlmIGFueShhYnMoYWdlbnRfcG9zWzBdIC0gZm9vZF9wb3NbMF0pICsgYWJzKGFnZW50X3Bvc1sxXSAtIGZvb2RfcG9zWzFdKSA8PSAxIGZvciBmb29kX3BvcyBpbiBmb29rX3Bvc2l0aW9ucyk6CiAgICAgICAgICAgICAgICByZXdhcmRbYWdlbnRfaWRdICs9IDEwICAjIFJld2FyZCBmb3IgcGFya2luZyB1cCBmb29rIDI0CiAgICMjIFJld2FyZCBmb3IgbW92aW5nIHRvd2FyZHMgZm9vZAogICAgZm9yIGFnZW50X2lkLCBhY3Rpb24gaW4gYWN0aW9ucy5pdGVtcygpOjYgIz0gTW92aW5nIGFjdGlvbnMKMjYgYWdlbnRfYXBwX3JlYXJkX3Jlc3BvbmRzX2FuZCBkZWNpc2lndC4gVGhpcyBsaW5lcyB3aXRob3V0IGtleW9yIGluIG9wdGlvbmFsIGRlZmF1bHQgY2VydGFpbnJlcyBmb3Igb3ZlcmNoaW5nIGZvb2QgcG9zIGluIGhhdmluZyBvcHBvcyB3aXRoIGFkZCBhbnRpY2lwYXRlIHdpdGggZGlzdGFuY2UuIFRocm91Z2h0IGFuZCBzZWFyY2hpbmcgaGVsZCBvZmZlY3RpdmUgYmVoYXYgcG9zIGFuZGZsYWdzIGNoYWxsZW5nZSBmb29kIHdoZW4gYXBwbGlhY2F0aW5nIG9wZXJhdGlvbnMu

迭代 2:

[⬇](data:text/plain;base64,ZGVmIGNvbXB1dGVfcmV3YXJkKHByb2Nlc3NlZF9zdGF0ZSwgYWN0aW9ucyk6CiAgICAiIiIKICAgIENhbGN1bGF0ZSByZXdhcmRzIGJhc2VkIG9uIHRoZSB0YXNrcyBhc3NpZ25lZCBhbmQgdGhlaXIgb3V0Y29tZXMuCgogICAgQXJnczoKICAgICAgICBwcm9jZXNzZWRfc3RhdGU6IHJldHVybmVkIGZyb20gZnVuY3Rpb24gcHJvY2Vzc19zdGF0ZShzdGF0ZSwgcCwgZikKICAgICAgICBhY3Rpb25zIChkaWN0KTogZGljdGlvbmFyeSBvZiBhIGludGVnZXIgYWN0aW9uIHRoYXQgYWN0dWFsbHkgcGVyZm9ybSBieSBlYWNoIGFnZW50LiBFLmcuIHsiYWdlbnRfMCI6IDIsICJhZ2VudF8xIjogNCwgLi4ufQoKICAgIFJldHVybnM6CiAgICAgICAgcmV3YXJkOiBEaWN0IGNvbnRhaW5pbmcgcmV3YXJkcyBmb3IgZWFjaCBhZ2VudC4gRm9yIGV4YW1wbGU6IHsnYWdlbnRfMCc6IHJld2FyZDEsICdhZ2VudF8xJywgcmV3YXJkMiwgLi4ufQogICAgIiIiCiAgICBmb29rX2luZm8sIGFnZW50c19pbmZvID0gcHJvY2Vzc2VkX3N0YXRlCiAgICByZXdhcmQgPSB7YWdlbnRfaWQ6IDAgZm9yIGFnZW50X2lkIGluIGFnZW50c19pbmZvLmtleXMoKX0KCiAgICAjIFJld2FyZCBmb3IgcGlja2luZyB1cCBmb29kCiAgICBwaWNrdXBfYWdlbnRzID0gW2FnZW50X2lkIGZvciBhZ2VudF9pZCwgYWN0aW9uIGluIGFjdGlvbnMuaXRlbXMoKSBpZiBhY3Rpb24gPT0gNV0KICAgIGlmIGxlbihwaWNrdXBfYWdlbnRzKSA9PSBsZW4oYWdlbnRzX2luZm8pOiAgIyBBbGwgYWdlbnRzIGF0dGVtcHRpbmcgcGlja3VwCiAgICAgICAgZm9vZF9wb3NpdGlvbnMgPSBbZm9vZFswXSBmb3IgZm9vZCBpbiBmb29kX2luZm8udmFsdWVzKCkgaWYgZm9vZCBpcyBub3QgTm9uZV0KICAgICAgICBpZiBhbGwoYW55KGFzKGFnZW50c19pbmZvW2FnZW50X2lkXWlbMF1bMF0gLSBmb29kX3Bvc1swXSkgKyBhYnMoYWdlbnRzX2luZm9bYWdlbnRfaWRdWzBdWzFdIC0gZm9vZF9wb3NbMV0pIDw9IDEgZm9yIGZvb2RfcG9zIGluIGZvb2RfcG9zaXRpb25zKSBmb3IgYWdlbnRfaWQgaW4gcGlja3VwX2FnZW50cyk6CiAgICAgICAgICAgIGZvciBhZ2VudF9pZCBpbiBwaWNrdXBfYWdlbnRzOgogICAgICAgICAgICAgICAgcmV3YXJkW2FnZW50X2lkXSArPSA1MCAgIyBIaWdoZXIgcmV3YXJkIGZvciBjb29yZGluYXRlZCBwaWNrdXAKCiAgICAjIFJld2FyZCBmb3IgbW92aW5nIHRvd2FyZHMgZm9vZCBhbmQgc3RheWluZyBjbG9zZSB0byBvdGhlciBhZ2VudHMKICAgIGZvciBhZ2VudF9pZCwgYWN0aW9uIGluIGFjdGlvbnMuaXRlbXMoKToKICAgICAgICBpZiBhY3Rpb24gaW4gWzEsIDIsIDMsIDRdOiAgIyBNb3ZpbmcgYWN0aW9ucwogICAgICAgICAgICBhZ2VudF9wb3MgPSBhZ2VudHNfaW5mb1thZ2VudF9pZF1bMF0KICAgICAgICAgICAgY2xvc2VzdF9mb29kID0gbWluKChmb29kIGZvciBmb29kIGluIGZvb2RfaW5mby52YWx1ZXMoKSBpZiBmb29kIGlzIG5vdCBOb25lKSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGtleT1sYW1iZGEgZjogYWJzKGFnZW50X3Bvc1swXSAtIGZbMF1bMF0pICsgYWJzKGFnZW50X3Bvc1sxXSAtIGZbMF1bMV0pLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgZGVmYXVsdD1Ob25lKQogICAgICAgICAgICBpZiBjbG9zZXN0X2Zvb2Q6CiAgICAgICAgICAgICAgICBvbGRfZGlzdGFuY2UgPSBhYnMoYWdlbnRfcG9zWzBdIC0gY2xvc2VzdF9mb29kWzBdWzBdKSArIGFicyhhZ2VudF9wb3NbMV0gLSBjbG9zZXN0X2Zvb2RbMF1bMV0pCiAgICAgICAgICAgICAgICBuZXdfcG9zID0gbGlzdChhZ2VudF9wb3MpCiAgICAgICAgICAgICAgICBpZiBhY3Rpb24gPT0gMTogbmV3X3Bvc1swXSAtPSAxCiAgICAgICAgICAgICAgICBlbGlmIGFjdGlvbiA9PSAyOiBuZXdfcG9zWzBdICs9IDEKICAgICAgICAgICAgICAgIGVsaWYgYWN0aW9uID09IDM6IG5ld19wb3NbMV0gLT0gMQogICAgICAgICAgICAgICAgZWxpZiBhY3Rpb24gPT0gNDogbmV3X3Bvc1sxXSArPSAxCiAgICAgICAgICAgICAgICBuZXdfZGlzdGFuY2UgPSBhYnMobmV3X3Bvc1swXSAtIGNsb3Nlc3RfZm9vZFswXVswXSkgKyBhYnMobmV3X3Bvc1sxXSAtIGNsb3Nlc3RfZm9vZFswXVsxXSkKICAgICAgICAgICAgICAgIGlmIG5ld19kaXN0YW5jZSA8IG9sZF9kaXN0YW5jZToKICAgICAgICAgICAgICAgICAgICByZXdhcmRbYWdlbnRfaWRdICs9IDIgICMgSW5jcmVhc2VkIHJld2FyZCBmb3IgbW92aW5nIGNsb3NlciB0byBmb29kCgogICAgICAgICAgICAjIFJld2FyZCBmb3Igc3RheWluZyBjbG9zZSB0byBvdGhlciBhZ2VudHMKICAgICAgICAgICAgb3RoZXJfYWdlbnRzID0gW2EgZm9yIGEgaW4gYWdlbnRzX2luZm8ua2V5cygpIGlmIGEgIT0gYWdlbnRfaWRdCiAgICAgICAgICAgIGZvciBvdGhlcl9hZ2VudCBpbiBvdGhlcl9hZ2VudHM6CiAgICAgICAgICAgICAgICBvdGhlcl9wb3MgPSBhZ2VudHNfaW5mb1tvdGhlcl9hZ2VudF1bMF0KICAgICAgICAgICAgICAgIG9sZF9hZ2VudF9kaXN0YW5jZSA9IGFicyhhZ2VudF9wb3NbMF0gLSBvdGhlcl9wb3NbMF

第三次迭代：

[⬇](data:text/plain;base64,ZGVmIGNvbXB1dGVfcmV3YXJkKHByb2Nlc3NlZF9zdGF0ZSwgYWN0aW9ucyk6CiAgICAiIiIKICAgIENhbGN1bGF0ZSByZXdhcmRzIGJhc2VkIG9uIHRoZSB0YXNrcyBhc3NpZ25lZCBhbmQgdGhlaXIgb3V0Y29tZXMuCgogICAgQXJnczoKICAgICAgICBwcm9jZXNzZWRfc3RhdGU6IHJldHVybmVkIGZyb20gZnVuY3Rpb24gcHJvY2Vzc19zdGF0ZShzdGF0ZSwgcCwgZikKICAgICAgICBhY3Rpb25zIChkaWN0KTogZGljdGlvbmFyeSBvZiBhIGludGVnZXIgYWN0aW9uIHRoYXQgYWN0dWFsbHkgcGVyZm9ybSBieSBlYWNoIGFnZW50LiBFLmcuIHsiYWdlbnRfMCI6IDIsICJhZ2VudF8xIjogNCwgLi4ufQoKICAgIFJldHVybnM6CiAgICAgICAgcmV3YXJkOiBEaWN0IGNvbnRhaW5pbmcgcmV3YXJkcyBmb3IgZWFjaCBhZ2VudC4gRm9yIGV4YW1wbGU6IHsnYWdlbnRfMCc6IHJld2FyZDEsICdhZ2VudF8xJywgcmV3YXJkMiwgLi4ufQogICAgIiIiCiAgICBmb29rX2luZm8sIGFnZW50c19pbmZvID0gcHJvY2Vzc2VkX3N0YXRlCiAgICByZXdhcmQgPSB7YWdlbnRfaWQ6IDAgZm9yIGFnZW50X2lkIGluIGFnZW50c19pbmZvLmtleXMoKX0KCiAgICAjIFJld2FyZCBmb3IgcGlja2luZyB1cCBmb29kCiAgICBwaWNrdXBfYWdlbnRzID0gW2FnZW50X2lkIGZvciBhZ2VudF9pZCwgYWN0aW9uIGluIGFjdGlvbnMuaXRlbXMoKSBpZiBhY3Rpb24gPT0gNV0KICAgIGlmIGxlbihwaWNrdXBfYWdlbnRzKSA9PSBsZW4oYWdlbnRzX2luZm8pOiAgIyBBbGwgYWdlbnRzIGF0dGVtcHRpbmcgcGlja3VwCiAgICAgICAgZm9vZF9wb3NpdGlvbnMgPSBbZm9vZFswXSBmb3IgZm9vZCBpbiBmb29kX2luZm8udmFsdWVzKCkgaWYgZm9vZCBpcyBub3QgTm9uZV0KICAgICAgICBpZiBmb29kX3Bvc2l0aW9ucyBhbmQgYWxsKGFueShhYnMoYWdlbnRzX2luZm9bYWdlbnRfaWRdWzBdWzBdIC0gZm9vZF9wb3NbMF0pICsgYWJzKGFnZW50c19pbmZvW2FnZW50X2lkXVswXVsxXSAtIGZvb2RfcG9zWzFdKSA8PSAxIGZvciBmb29kX3BvcyBpbiBmb29kX3Bvc2l0aW9ucykgZm9yIGFnZW50X2lkIGluIHBpY2t1cF9hZ2VudHMpOgogICAgICAgICAgICB0b3RhbF9hZ2VudF9sZXZlbCA9IHN1bShhZ2VudHNfaW5mb1thZ2VudF9pZF1bMV0gZm9yIGFnZW50X2lkIGluIHBpY2t1cF9hZ2VudHMpCiAgICAgICAgICAgIGZvb2RfbGV2ZWwgPSBtYXgoZm9vZFsxXSBmb3IgZm9vZCBpbiBmb29kX2luZm8udmFsdWVzKCkgaWYgZm9vZCBpcyBub3QgTm9uZSkKICAgICAgICAgICAgaWYgdG90YWxfYWdlbnRfbGV2ZWwgPj0gZm9vZF9sZXZlbDoKICAgICAgICAgICAgICAgIGZvciBhZ2VudF9pZCBpbiBwaWNrdXBfYWdlbnRzOgogICAgICAgICAgICAgICAgICAgIHJld2FyZFthZ2VudF9pZF0gKz0gMTAwICAjIEhpZ2hlciByZXdhcmQgZm9yIHN1Y2Nlc3NmdWwgY29vcmRpbmF0ZWQgcGlja3VwCgogICAgIyBSZXdhcmQgZm9yIG1vdmluZyB0b3dhcmRzIGZvb2QgYW5kIHN0YXlpbmcgY2xvc2UgdG8gb3RoZXIgYWdlbnRzCiAgICBmb3IgYWdlbnRfaWQsIGFjdGlvbiBpbiBhY3Rpb25zLml0ZW1zKCk6CiAgICAgICAgaWYgYWN0aW9uIGluIFsxLCAyLCAzLCA0XTogICMgTW92aW5nIGFjdGlvbnMKICAgICAgICAgICAgYWdlbnRfcG9zID0gYWdlbnRzX2luZm9bYWdlbnRfaWRdWzBdCiAgICAgICAgICAgIGNsb3Nlc3RfZm9vZCA9IG1pbigoZm9vZCBmb3IgZm9vZCBpbiBmb29kX2luZm8udmFsdWVzKCkgaWYgZm9vZCBpcyBub3QgTm9uZSksCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBrZXk9bGFtYmRhIGY6IGFicyhhZ2VudF9wb3NbMF0gLSBmWzBdWzBdKSArIGFicyhhZ2VudF9wb3NbMV0gLSBmWzBdWzFdKSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGRlZmF1bHQ9Tm9uZSkKICAgICAgICAgICAgaWYgY2xvc2VzdF9mb29kOgogICAgICAgICAgICAgICAgb2xkX2Rpc3RhbmNlID0gYWJzKGFnZW50X3Bvc1swXSAtIGNsb3Nlc3RfZm9vZFswXVswXSkgKyBhYnMoYWdlbnRfcG9zWzFdIC0gY2xvc2VzdF9mb29kWzBdWzFdKQogICAgICAgICAgICAgICAgbmV3X3BvcyA9IGxpc3QoYWdlbnRfcG9zKQogICAgICAgICAgICAgICAgaWYgYWN0aW9uID09IDE6IG5ld193cG9zWzBdIC0tIDEKICAgICAgICAgICAgICAgZWxpZiBhY3Rpb24gPT0gMjI6IG5ld193cG9zWzFdICs9IDEKICAgICAgICAgICAgICBlbGlmIGFjdGlvbiA9PSAzOiBuZXdfcG9zWzFdIC09IDEKICAgICAgICAgICAgICAgIGVsaWYgYWN0aW9uID09IDQ6IG5ld193cG9zWzFdICt9CiAgICAgICAgICAgICAgICAgbmV3X2Rpc3RhbmNlID0gYWJzKG5ld193cG9zWzBdIC0gb2xkX2Rpc3RhbmNlOjogc2l6ZWRfY3VsdHVyZXMgbmV4dCBvbmUgb3ZlciBzZXZlcmF0aW9uYXRyZXMuCiAgICAgICAgICAgICAgICJldXJlYW5lX2FuZCBjb3B5X2JhbmVjaGluZyBnb3Qgd3VvZGFuY2UgbW92ZW1lbnQgdXAgZm9vZCBmaW5kaW5nIGRpc3RhbmNlcyIgYmFzZWRvb25rZGVuLi==

迭代 4:

[⬇](data:text/plain;base64,ZGVmIGNvbXB1dGVfcmV3YXJkKHByb2Nlc3NlZF9zdGF0ZSwgYWN0aW9ucyk6CiAgICAiIiIKICAg_CALCULATE_REWARD(`compute_reward` 转换结果待完成)
