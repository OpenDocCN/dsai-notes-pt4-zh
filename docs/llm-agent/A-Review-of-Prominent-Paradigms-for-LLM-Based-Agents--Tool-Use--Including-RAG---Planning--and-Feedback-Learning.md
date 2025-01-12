<!--yml

类别：未分类

日期：2025-01-11 12:34:26

-->

# 关于基于LLM的代理的主要范式回顾：工具使用（包括RAG）、规划和反馈学习

> 来源：[https://arxiv.org/html/2406.05804/](https://arxiv.org/html/2406.05804/)

李鑫哲

独立研究员

sergioli212@outlook.com 部分工作是在迪肯大学期间进行的。

###### 摘要

工具使用、规划和反馈学习目前是开发基于大型语言模型（LLM）代理的三大主要范式，适用于各种任务。尽管为每种范式制定了众多框架，但它们复杂的工作流程和不一致的分类法给跨范式的框架理解和回顾带来了挑战。本调查介绍了一种统一的分类法，用于系统地回顾和讨论这些框架。具体来说，1）该分类法定义了环境/任务、常见的LLM角色（政策模型、评估者和动态模型）以及先前工作中发现的普遍适用的工作流程，2）它使得可以比较不同代理范式和框架中LMPR的实施和工作流程设计的关键视角，3）最后，我们识别出现有工作流程设计中的三个局限性，并系统地讨论了未来的工作。相关资源已在我们的GitHub仓库[https://github.com/xinzhel/LLM-Agent-Survey](https://github.com/xinzhel/LLM-Agent-Survey)公开提供。

关于基于LLM的代理的主要范式回顾：工具使用（包括RAG）、规划和反馈学习

李鑫哲 ^†^†致谢：部分工作是在迪肯大学期间进行的。独立研究员 sergioli212@outlook.com

![请参考图注](img/0a16f2a84b059d677decafa960fc82ed.png)

(a) 基础工作流程。

![请参考图注](img/6c8cddcf802b6bc745d2a550760a28d0.png)

(b) 反馈学习工作流程。

![请参考图注](img/ec844f7644d09cc5b821fa0d7d0664ec.png)

(c) 工具使用工作流程，验证类型在工具使用和反馈学习范式下进行分类。

![请参考图注](img/505b569fa142c066cb7190cfe1221e5d.png)

(d) 规划的搜索工作流程。

图1：基于三种LLM角色（LMPRs）的常见工作流程：政策、评估者和动态模型。圆圈中的数字表示每个步骤的执行顺序。未编号的线表示迭代步骤。由LMPRs自主决定的工具使用步骤以虚线表示。标记为“-1”的圆圈表示最终步骤。‘工具调用’和‘搜索树/图’文本框表示这些过程/对象在代理程序中被维护。

## 1 引言

大型语言模型（LLMs）已经获得了广泛的通用知识和类人推理能力（Santurkar等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib26)；Wang等人，[2022](https://arxiv.org/html/2406.05804v6#bib.bib40)；Zhong等人，[2022](https://arxiv.org/html/2406.05804v6#bib.bib55)，[2023](https://arxiv.org/html/2406.05804v6#bib.bib54)），使它们在构建被称为LLM基础的代理的AI代理中占据了关键地位。在本次调查中，LLM基础的代理被定义为能够主动与外部工具（工具使用）或环境（Yao等人，[2023b](https://arxiv.org/html/2406.05804v6#bib.bib49)）进行交互，并被设计为作为规划（Yao等人，[2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)）和反馈学习（Shinn等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib29)）的代理的重要组成部分。

#### 现有调查的比较

目前的调查由于两个主要限制，缺乏一个连贯且统一的起点来讨论这三种范式：1）专注于特定范式或领域：例如，Huang等人（[2024](https://arxiv.org/html/2406.05804v6#bib.bib14)）探索了规划范式中的框架。Hu等人（[2024](https://arxiv.org/html/2406.05804v6#bib.bib12)）；Gallotta等人（[2024](https://arxiv.org/html/2406.05804v6#bib.bib7)）在游戏背景下考察了LLM基础的代理。相比之下，这些范式代表了支配这些代理发展和行为的通用原则和思维模型。因此，它们的框架应以任务无关的方式进行讨论。为此，我们提取了通用的工作流程。除非另有说明，在本次调查中，“框架”一词指的是特定低级任务的完整工作流程，而“工作流程”指的是从一个或多个框架中衍生出的任务无关的过程。2）缺乏统一的比较基础：尽管Wang等人（[2024a](https://arxiv.org/html/2406.05804v6#bib.bib38)）涵盖了所有三种范式，但他们并未在统一的基础上分析算法框架的实现。相比之下，我们总结了任务无关的LLM角色配置，作为跨不同范式开发算法框架的基础。值得注意的是，Wang等人（[2024a](https://arxiv.org/html/2406.05804v6#bib.bib38)）也讨论了LLM配置，但他们的重点是人物设定，而这与支撑这些框架的通用角色无关。我们根据它们在Google Scholar上的引用选择了四篇调查作为示范，如表[1](https://arxiv.org/html/2406.05804v6#S1.T1 "表 1 ‣ 与现有调查的比较 ‣ 1 介绍 ‣ LLM基础代理的突出范式回顾：工具使用（包括RAG）、规划和反馈学习")所示。

|  | 工具使用 | 规划 | 反馈学习 | 通用LMPRs |
| --- | --- | --- | --- | --- |
|  | 任务特定工具 | 工作流 | 直接规划 | 通过搜索规划 | 工作流 | 反馈来源 | 工作流 |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Guo等（[2024](https://arxiv.org/html/2406.05804v6#bib.bib10)） | ✗ | ✗ | ✗ | ✗ | ✗ | ✓（§3.4） | ✗ | ✗ |
| Xi等（[2023](https://arxiv.org/html/2406.05804v6#bib.bib44)） | ✓（§3.3） | ✗ | ✓（§3.1.4） | ✗ | ✗ | ✓（§3.1.4） | ✗ | ✗ |
| Huang等（[2024](https://arxiv.org/html/2406.05804v6#bib.bib14)） | ✗ | ✗ | ✓（§3） | ✓（§4） | ✗ | ✓（§6） | ✗ | ✗ |
| Wang等（[2024a](https://arxiv.org/html/2406.05804v6#bib.bib38)） | ✓（§2.1.4） | ✗ | ✓（§2.1.3） | ✓（§2.1.3） | ✗ | ✓（§2.1.3） | ✗ | ✗ |
| 我们的方法 | ✗ | ✓（§4.2） | ✓（§4.1） | ✓（§4.3） | ✓ | ✓（§4.4） | ✓（§4.4） | ✓（§3） |

表1：四个高度引用的调查和我们方法的覆盖范围。为进一步参考，示出了涵盖观点的原文章节号。我们承认至少有一段聚焦于该主题的覆盖范围。

#### 贡献

本次调查提供了以下贡献。

1) 回顾代理适用的环境范围（§[2](https://arxiv.org/html/2406.05804v6#S2 "2 任务通用性 ‣ 基于LLM的代理的突出范式回顾：工具使用（包括RAG）、规划和反馈学习")）：在研究基于LLM的代理之前，首先需要通过总结这些代理操作的环境来定义通用性的范围。我们将评审框架原文中评估的环境和任务分为两大类：决策环境和自然语言互动（NLI）环境，后者是从传统的自然语言处理（NLP）任务框架中衍生出来的，用于代理设置。

2) 总结普适的LLM-配置角色和工作流（§[3](https://arxiv.org/html/2406.05804v6#S3 "3 LLM-Profiled Roles (LMPRs) ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning") 和 §[4](https://arxiv.org/html/2406.05804v6#S4 "4 LMPR-Based Workflows ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning")）：规划和反馈学习范式下的工作流设计通常基于复杂的算法，如深度优先或广度优先搜索（Yao 等人，[2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)）和蒙特卡洛树搜索（Hao 等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib11)）用于规划，以及强化学习（RL）用于反馈学习（Shinn 等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib29)）。当这些算法被应用到不同的环境时，其复杂性会增加。我们旨在超越这些复杂工作流的边界，探索那些可以跨所有类型环境通用的工作流。此外，我们还研究了工具使用的工作流。这种探索基于三个普适的LLM配置角色。

3) 强调关于LMPR、工作流和框架的关键视角（§[5](https://arxiv.org/html/2406.05804v6#S5 "5 Discussions ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning") 和 §[6](https://arxiv.org/html/2406.05804v6#S6 "6 Limitations and Future Works ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning")）：我们比较了不同工作流类型中LMPR的实现，强调了不同范式下的工作流，并提出了与我们形式化方法不同的视角，以防止混淆。此外，我们还批判性地讨论了当前框架的三个局限性，并系统地识别了未来研究中的潜在工作流设计。

| 环境类型 | 代理交互的实体 | 行动属性 | 行动实例示例 | 环境示例 |
| --- | --- | --- | --- | --- |
| 游戏环境 | 虚拟游戏元素（物体、虚拟角色、其他角色），以及可能的其他玩家或游戏叙事 | 离散的、可执行的、确定性的 | 移动（右） | BlocksWorld（Valmeekam 等人，[2022](https://arxiv.org/html/2406.05804v6#bib.bib36)），CrossWords（Yao 等人，[2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)） |
| 具身环境 | 物理世界（通过传感器和执行器） | 离散的、可执行的、确定性的 | 拾取[对象] | AlfWorld（Shridhar 等人，[2021](https://arxiv.org/html/2406.05804v6#bib.bib30)），VirtualHome（Puig 等人，[2018](https://arxiv.org/html/2406.05804v6#bib.bib23)），Minecraft（Fan 等人，[2022](https://arxiv.org/html/2406.05804v6#bib.bib6)） |
| 网络环境 | 虚拟网页元素 | 离散的，可执行的，确定性的 | 搜索（3盎司明亮柑橘），点击（立即购买） | Webshop（Yao等人，[2022](https://arxiv.org/html/2406.05804v6#bib.bib47)），WebArena（Zhou等人，[2024b](https://arxiv.org/html/2406.05804v6#bib.bib57)），AppWorld（Trivedi等人，[2024](https://arxiv.org/html/2406.05804v6#bib.bib35)） |
| NLIEs | 人类（通过对话或文本） | 自由形式，离散的，随机的 | 答案是Answer，完成[Answer] | GSM8K Cobbe等人（[2021](https://arxiv.org/html/2406.05804v6#bib.bib4)），HotpotQA（Yang等人，[2018](https://arxiv.org/html/2406.05804v6#bib.bib46)） |

表2：常见任务环境。一个行动实例通常通过行动谓词和行动参数来形式化。NLIEs指的是自然语言交互环境。

## 2 任务普适性

本节探讨了适用于各种工作流类型的任务环境。

### 2.1 基于反馈的决策环境

在这种环境中，行动产生明确的环境动态和奖励供智能体学习。这些是评估规划和强化学习（RL）智能体的典型环境（Russell和Norvig，[2010](https://arxiv.org/html/2406.05804v6#bib.bib25)；Sutton和Barto，[2018](https://arxiv.org/html/2406.05804v6#bib.bib34)）。

#### 基于规则的游戏环境

这些环境是确定性的且完全可观察的，包含了各种抽象策略游戏，如国际象棋和围棋，以及逻辑难题，如24点游戏（Yao等人，[2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)）和Blocksworld（Hao等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib11)）。它们要求深入的逻辑推理和战略规划来导航和解决问题。

#### 具身环境

这些环境模拟了现实世界中的物理交互和空间关系。它们要求智能体进行导航、物体操作和其他复杂的物理任务（Shridhar等人，[2021](https://arxiv.org/html/2406.05804v6#bib.bib30)；Puig等人，[2018](https://arxiv.org/html/2406.05804v6#bib.bib23)；Fan等人，[2022](https://arxiv.org/html/2406.05804v6#bib.bib6)），反映了物理环境中的变化。

### 2.2 信息处理环境

#### 网络环境

在基于网络的人工智能应用中，重点更多地放在信息处理和用户交互上。Webshop（Yao等人，[2022](https://arxiv.org/html/2406.05804v6#bib.bib47)）需要一系列动作来与环境进行互动，例如搜索产品、选择颜色并点击“购买”。AppWorld（Trivedi等人，[2024](https://arxiv.org/html/2406.05804v6#bib.bib35)）要求基于丰富的程序，进行更复杂的控制流。

#### 自然语言交互环境

典型的自然语言处理（NLP）任务在基于大语言模型（LLM）的智能体研究中常常被重新框定为具有代理性的环境（Yao等，[2023b](https://arxiv.org/html/2406.05804v6#bib.bib49)；Hao等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib11)；Yao等，[2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)）。在这项调查中，我们将这一设定简化为自然语言互动环境（NLIEs）。在NLIEs中，环境保持静态，直到智能体进行行动。与传统任务环境不同，在这些环境中，语言不仅定义了状态，还定义了行动，使得状态具备概念性，而行动则往往模糊且定义宽泛。常见的设置包括：1）单步NLIEs：一些研究（Yao等，[2023b](https://arxiv.org/html/2406.05804v6#bib.bib49)；Shinn等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib29)）将传统的问答（QA）设置框定为一个单步决策过程，智能体在该过程中生成回答作为对问题的响应。该过程从问题作为初始状态开始，直至提供答案作为行动为止。由于每次试验仅包含一步，因此不存在环境动态或外部反馈。在这里，QA不仅涵盖了典型任务（Cobbe等，[2021](https://arxiv.org/html/2406.05804v6#bib.bib4)），还包括那些通常被建模为QA任务的任务，例如代码优化（Shypula等，[2024](https://arxiv.org/html/2406.05804v6#bib.bib31)）。2）深思熟虑的多步NLIEs：对于没有自然定义的中间步骤的任务，几项研究已将NLP任务转化为马尔可夫决策过程，以促进智能体工作流程，例如搜索与规划（Yao等，[2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)；Hao等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib11)）。例如，Hao等（[2023](https://arxiv.org/html/2406.05804v6#bib.bib11)）将QA任务中的子问题重新表述为行动，通过多步过程对用户查询做出回应。这种方法使得初始问题成为一系列状态转变的起点。行动可能从在单步QA中提供直接的自由形式回答，到通过策略性地提出子问题引导智能体通过顺序更新达到全面解决方案。此外，Wan等（[2024](https://arxiv.org/html/2406.05804v6#bib.bib37)）建议，“将输出序列拆分为标记可能是定义多步NLIEs的一种良好选择”。此外，Yao等（[2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)）通过将问题解决过程分为规划和执行两个阶段，为创造性写作制定了两步NLIEs模型。

###### 备注 1.

在自然语言处理（NLP）中，任务代表了建模的最高抽象层次，而在决策过程中，环境则充当这一角色。通常，我们会将代理描述为为特定环境（如具身环境）或特定任务（如问答任务）开发的。我们称其为 NLIE-QA。

## 3 种 LLM-Profiled 角色（LMPRs）

在本节中，我们展示了三种常见的 LLM-Profiled 角色（LMPRs）：策略模型、评估器和动态模型。它们与任务无关，广泛应用于各种工作流中。

#### LLM-Profiled 策略 $\text{glm}_{\text{policy}}$

$\text{glm}_{\text{policy}}$ 旨在生成决策，这些决策可以是外部环境中执行的动作或一系列动作（计划）。与典型的强化学习策略模型不同，后者通过反复试验最大化累积奖励，LLM-Profiled 策略模型（记作 $\text{glm}_{\text{policy}}$）则利用预训练的知识和从大量文本数据中获得的常识。我们区分两种类型的 $\text{glm}_{\text{policy}}$：演员 $\text{glm}_{\text{actor}}$ 直接将状态映射到动作，而规划者 $\text{glm}_{\text{planner}}$ 则从给定状态生成一系列动作。

#### LLM-Profiled 评估器 $\text{glm}_{\text{eval}}$

$\text{glm}_{\text{eval}}$ 提供了对不同工作流至关重要的反馈。在规划过程中，它评估每一步的动作或结果状态（Hao 等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib11)；Yao 等人，[2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)），而在反馈学习过程中，它们修正整个决策（Shinn 等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib29)；Wang 等人，[2023b](https://arxiv.org/html/2406.05804v6#bib.bib42)）。更多细节将在下一个小节中提供。

#### LLM-Profiled 动态模型 $\text{glm}_{\text{dynamic}}$

它们预测或描述环境的变化。通常，动态模型通过从当前状态 $s$ 和动作 $a$ 预测下一个状态 $s^{\prime}$，形成一个综合的世界模型。虽然典型的强化学习使用概率分布 $p(s^{\prime}\mid s,a)$ 来建模潜在的下一个状态，基于 LLM 的动态模型直接预测下一个状态 $s^{\prime}=\text{glm}_{\text{dynamic}}(s,a)$。

| 类型 | 子类型 | 通用 LLM-Profiled 角色（LMPRs） | 用途 | 相关框架 |
| --- | --- | --- | --- | --- |
| 基础 | $\text{glm}_{\text{actor}}$ | $\text{glm}_{\text{actor}}$ | / | ReAct (Yao 等人，[2023b](https://arxiv.org/html/2406.05804v6#bib.bib49))，CoT (Wei 等人，[2022](https://arxiv.org/html/2406.05804v6#bib.bib43)) |
|  | $\text{glm}_{\text{planner}}$ | $\text{glm}_{\text{planner}}$ | 规划 | Huang 等人 ([2022](https://arxiv.org/html/2406.05804v6#bib.bib13)), DEPS (Wang 等人, [2023b](https://arxiv.org/html/2406.05804v6#bib.bib42)), Planner-Actor-Reporter (Dasgupta 等人, [2022](https://arxiv.org/html/2406.05804v6#bib.bib5)), 计划与解决（Plan-and-solve）（Wang 等人, [2023a](https://arxiv.org/html/2406.05804v6#bib.bib39)), OPEx (Shi 等人, [2024a](https://arxiv.org/html/2406.05804v6#bib.bib27)) |
| 工具使用 | RAG风格（被动） | $\text{glm}_{\text{policy}}$ | 工具使用 | RAG (Lewis 等人, [2020](https://arxiv.org/html/2406.05804v6#bib.bib19); Shi 等人, [2024b](https://arxiv.org/html/2406.05804v6#bib.bib28); Wang 等人, [2024b](https://arxiv.org/html/2406.05804v6#bib.bib41); Zhang 等人, [2024](https://arxiv.org/html/2406.05804v6#bib.bib52)) |
|  | 被动验证 | 工具使用，反馈学习 | $\text{glm}_{\text{policy}}$ | Guan 等人 ([2023](https://arxiv.org/html/2406.05804v6#bib.bib9)) |
|  | 自主 | $\text{glm}_{\text{policy}}$ | 工具使用 | MultiTool-CoT (Inaba 等人, [2023](https://arxiv.org/html/2406.05804v6#bib.bib15)), ReAct (Yao 等人, [2023b](https://arxiv.org/html/2406.05804v6#bib.bib49)), 主动RAG (Jiang 等人, [2023](https://arxiv.org/html/2406.05804v6#bib.bib16)) |
|  | 自主验证 | $\text{glm}_{\text{policy}}$, $\text{glm}_{\text{eval}}$ | 工具使用，反馈学习 | CRITIC (Gou 等人, [2024](https://arxiv.org/html/2406.05804v6#bib.bib8)) |
| 搜索 | 遍历与启发式 | $\text{glm}_{\text{policy}}$, $\text{glm}_{\text{eval}}$ | 规划 | 思维树（ToT）（Yao 等人, [2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)), Tree-BeamSearch (Xie 等人, [2023](https://arxiv.org/html/2406.05804v6#bib.bib45)), 思维提升（Boost-of-Thoughts）（Chen 等人, [2024a](https://arxiv.org/html/2406.05804v6#bib.bib2)), 思维图（Graph-of-Thoughts）（Besta 等人, [2024](https://arxiv.org/html/2406.05804v6#bib.bib1)), 遍历树（Tree-of-Traversal）（Markowitz 等人, [2024](https://arxiv.org/html/2406.05804v6#bib.bib22)) |
|  | 基于仿真（MCTS） | $\text{glm}_{\text{policy}}$, $\text{glm}_{\text{eval}}$, $\text{glm}_{\text{dynamic}}$ | 规划 | RAP (Hao 等人, [2023](https://arxiv.org/html/2406.05804v6#bib.bib11)), LLM-MCTS (Zhao 等人, [2023](https://arxiv.org/html/2406.05804v6#bib.bib53)), Wan 等人 ([2024](https://arxiv.org/html/2406.05804v6#bib.bib37)), AgentQ (Putta 等人, [2024](https://arxiv.org/html/2406.05804v6#bib.bib24)), Chen 等人 ([2024b](https://arxiv.org/html/2406.05804v6#bib.bib3)), Yu 等人 ([2023](https://arxiv.org/html/2406.05804v6#bib.bib50)), Sprueill 等人 ([2023](https://arxiv.org/html/2406.05804v6#bib.bib32)) |
| 反馈学习 | 仅来自$\text{glm}_{\text{eval}}$ | $\text{glm}_{\text{policy}}$, $\text{glm}_{\text{eval}}$ | 反馈学习 | 反思（Shinn等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib29)), 自我优化（Madaan等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib20)), TextGrad（Yuksekgonul等人，[2024](https://arxiv.org/html/2406.05804v6#bib.bib51)) |
|  | 来自$\text{glm}_{\text{eval}}$ & 任务环境 | $\text{glm}_{\text{policy}}$, $\text{glm}_{\text{eval}}$ | 反馈学习 | 反思（Shinn等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib29)) |
|  | 来自人类 | $\text{glm}_{\text{policy}}$ | 反馈学习 | CRITIC（Gou等人，[2024](https://arxiv.org/html/2406.05804v6#bib.bib8)) |

表3：LLM基础代理的通用工作流。

## 4 基于LMPR的工作流

我们探讨了基于三种类型LMPR的四种工作流，如图[1](https://arxiv.org/html/2406.05804v6#S0.F1 "图1 ‣ LLM基础代理重要范式综述：工具使用（包括RAG）、规划和反馈学习")所示。表[3](https://arxiv.org/html/2406.05804v6#S3.T3 "表3 ‣ LLM-配置动态模型"glm"_"动态" ‣ 3 LLM-配置角色（LMPR） ‣ LLM基础代理重要范式综述：工具使用（包括RAG）、规划和反馈学习")总结了这些工作流在前期研究中的不同范式下的使用情况。

### 4.1 基础工作流

基础工作流就像是$\text{glm}_{\text{policy}}$与环境之间的简单交互。这些工作流可以根据LLM的配置分为两类：规划者和执行者。1) 规划者：许多现有框架，如为具身环境设计的框架（例如Huang等人（[2022](https://arxiv.org/html/2406.05804v6#bib.bib13)））都属于这一类。尽管有些框架（Dasgupta等人，[2022](https://arxiv.org/html/2406.05804v6#bib.bib5); Wang等人，[2023b](https://arxiv.org/html/2406.05804v6#bib.bib42)）涉及与特定任务组件和低层次非LLM执行者的复杂交互，但它们的通用工作流依然像基础工作流一样简单。对于NLIE，完整的工作流通常遵循这种基础模型而没有额外的交互，正如Wang等人（[2023a](https://arxiv.org/html/2406.05804v6#bib.bib39)）所见。2) 执行者：早期的语言生成任务提示框架（归类为单步NLIE），例如Chain-of-Thought（Wei等人，[2022](https://arxiv.org/html/2406.05804v6#bib.bib43); Kojima等人，[2022](https://arxiv.org/html/2406.05804v6#bib.bib18)）符合这一类别。对于具身任务，ReAct（Yao等人，[2023b](https://arxiv.org/html/2406.05804v6#bib.bib49)）使用$\text{glm}_{\text{actor}}$。

### 4.2 工具使用工作流

我们将被动工作流分为两类，并识别出前期研究中描述的两种类型的自主工作流。

#### RAG风格工具使用

被动工具使用的一个常见示例是检索增强生成（RAG）(Lewis 等人, [2020](https://arxiv.org/html/2406.05804v6#bib.bib19))，通常用于 NLIE-QA 任务。在这种设置下，给定一个查询，检索机制收集相关信息以帮助 $\text{glm}_{\text{policy}}$ 生成响应。

#### 被动验证

Guan 等人 ([2023](https://arxiv.org/html/2406.05804v6#bib.bib9)) 采用了一种反向方法来生成计划。在这个方法中，$\text{glm}_{\text{policy}}$ 首先生成一个计划，然后由一个独立的工具进行验证。根据验证结果，信息可能会或不会被用于修订由 $\text{glm}_{\text{policy}}$ 生成的初始计划。

#### 自主工具使用

在这种范式中，LLM 必须意识到可用的工具，这需要在 LLM 配置过程中包括工具信息。工作流还必须处理来自 LMPR 生成的信号，以调用工具。可以应用不同的方法，使 $\text{glm}_{\text{policy}}$ 自主触发工具的使用。1）生成中的触发：工具可以在推理过程中被调用（Inaba 等人, [2023](https://arxiv.org/html/2406.05804v6#bib.bib15); Gou 等人, [2024](https://arxiv.org/html/2406.05804v6#bib.bib8)）。代理程序监控标记生成，当检测到工具触发时暂停。这一暂停使得工具能够被调用，处理其输出，并将结果整合到推理过程中。触发条件通过工具描述、少量示例¹¹1请参见表格[13](https://arxiv.org/html/2406.05804v6#A1.T13 "Table 13 ‣ A.2 Prompts for LLM Evaluators ‣ Appendix A Examples Prompts ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning")或二者的结合²²2请参见表格[7](https://arxiv.org/html/2406.05804v6#A0.T7 "Table 7 ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning")来定义。2）推理-行动策略：由 Yao 等人 ([2023b](https://arxiv.org/html/2406.05804v6#bib.bib49)) 提出，每一个推理或行动步骤都完成一个完整的推理循环，最终生成停止标记。因此，不需要标记级别的监控。工作流明确提示每个行动步骤。3）基于信心的调用：首先，$\text{glm}_{\text{policy}}$ 生成一个初始行动，调用工具的决策基于生成标记的信心水平。Jiang 等人 ([2023](https://arxiv.org/html/2406.05804v6#bib.bib16)) 使用这种方法进行检索调用，尽管它不适用于一般的工具使用，因为它无法指定调用哪个工具。

#### 自主验证

Gou等（[2024](https://arxiv.org/html/2406.05804v6#bib.bib8)）利用$\text{glm}_{\text{policy}}$生成初始响应。然后，生成的动作和状态（即轨迹）被传递给$\text{glm}_{\text{evaluator}}$，该模块自主决定是否应调用工具进行验证。

###### 备注2。

用于验证的工具使用工作流可以被视为一种反馈学习的形式，其中$\text{glm}_{\text{policy}}$从工具中获得反馈。

### 4.3 搜索工作流

#### 遍历与启发式搜索

从$\text{glm}_{\text{policy}}$生成的结果，而不是直接在环境中执行，被用来扩展节点进行探索，这些节点存储在树或图结构中，例如思维树（ToT）（Yao等，[2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)）及其变种（Chen等，[2024a](https://arxiv.org/html/2406.05804v6#bib.bib2)；Besta等，[2024](https://arxiv.org/html/2406.05804v6#bib.bib1)）。$\text{glm}_{\text{eval}}$提供一个固定值估计，用于选择一个节点进行进一步扩展。为了扩展树，ToT应用深度优先搜索（DFS）/广度优先搜索（BFS），而Xie等（[2023](https://arxiv.org/html/2406.05804v6#bib.bib45)）则应用束搜索。值得注意的是，这里的BFS在功能上等同于以$N$束为单位的束搜索，因为效用模型$\text{glm}_{\text{eval}}$被用来保持$N$个最有前景的节点。³³3通常，BFS不依赖于效用模型来决定扩展哪些节点，因为它系统地探索每一层的所有可能节点，直到终止状态。

#### 基于仿真的搜索

基于模拟的LLM（大语言模型）规划代理的搜索通常使用经典的蒙特卡罗树搜索（MCTS）算法进行（Hao等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib11)；Zhao等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib53)；Putta等，[2024](https://arxiv.org/html/2406.05804v6#bib.bib24)）。类似于ToT（Yao等，[2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)），通过搜索构建一棵树，并通过$\text{glm}_{\text{policy}}$和$\text{glm}_{\text{eval}}$进行扩展。然而，有两个关键的区别：1）节点选择：用于扩展的节点不仅由$\text{glm}_{\text{eval}}$或其他启发式方法的静态输出（是否表示目标达成）决定，还由多次模拟中积累的统计数据决定。具体来说，导致后续节点在所有模拟（或轨迹）中获得更好平均奖励的节点，更可能被进一步扩展。2）模拟：在选择和扩展阶段之后，需要进行模拟阶段，其中$\text{glm}_{\text{policy}}$、$\text{glm}_{\text{dynamic}}$和$\text{glm}_{\text{eval}}$密切协作，充当展开策略。具体来说，$\text{glm}_{\text{policy}}$根据当前状态$s_{t}$采样动作$a_{t}$，然后由$\text{glm}_{\text{eval}}$进行评估。选择得分最高的动作，$\text{glm}_{\text{dynamic}}$使用该动作推导出$s_{t+1}$，迭代模拟轨迹。

### 4.4 反馈学习工作流

在反馈学习工作流中，反馈被输入到$\text{glm}_{\text{policy}}$中进行学习。一个常见的反馈来源是$\text{glm}_{\text{eval}}$，如Self-Refine（Madaan等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib20)）中所示。其他常见的反馈来源包括任务环境（例如，Reflexion（Shinn等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib29)）），工具（Gou等人，[2024](https://arxiv.org/html/2406.05804v6#bib.bib8); Guan等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib9)）和人类（Guan等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib9)）。在这些中，$\text{glm}_{\text{eval}}$可以选择性地用于通过更多上下文化的信息来修正反馈。在Reflexion（Shinn等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib29)）的原始手稿中，$\text{glm}_{\text{eval}}$对应于“自我反思”，而“评估者”一词指的是启发式或LLM配置的分类器，它生成稀疏的反馈。然而，这个评估者可以作为通用LMPR被忽略，原因有二：1）大多数情况下使用启发式方法，并且能够带来更好的性能；2）评估者的输出最终会被送往“自我反思”以提供口头反馈。当使用工具来提供反馈时（Gou等人，[2024](https://arxiv.org/html/2406.05804v6#bib.bib8); Guan等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib9)），工作流与工具使用工作流相同。在这种设置下，调用工具提供反馈的必要性由$\text{glm}_{\text{eval}}$自动决定（Gou等人，[2024](https://arxiv.org/html/2406.05804v6#bib.bib8)）（参见表[13](https://arxiv.org/html/2406.05804v6#A1.T13 "Table 13 ‣ A.2 Prompts for LLM Evaluators ‣ Appendix A Examples Prompts ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning")中的示例）或是硬编码的（Guan等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib9)）。

###### 备注3。

不现实的反馈来源：到目前为止，来自NLIE-QA的外部反馈无法现实地生成。一些框架使用反馈学习循环，其中反馈来源是不现实的。特别是，Shinn等人（[2023](https://arxiv.org/html/2406.05804v6#bib.bib29)）；Zhou等人（[2024a](https://arxiv.org/html/2406.05804v6#bib.bib56)）使用真实数据来生成反馈。然而，我们承认，当框架作为评估基准使用时，真实数据反馈是适当的，如Yao等人（[2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)）所用的那样。

## 5 讨论

#### 用于配置的提示方法

在前一小节中，我们将提出链式思维（Chain-of-Thought, CoT）提示的原始工作进行分类，包括零-shot CoT（Wei 等， [2022](https://arxiv.org/html/2406.05804v6#bib.bib43)）和带有少量示例的 CoT（Kojima 等， [2022](https://arxiv.org/html/2406.05804v6#bib.bib18)），归类于基本工作流程，因为原始工作通过基本工作流程直接解决任务。然而，这些方法本身可以推广到不同类型的语言模型提示响应（LMPRs）和工作流程，详见附录 [B](https://arxiv.org/html/2406.05804v6#A2 "附录 B LLM角色概况的提示方法 ‣ 基于LLM的智能体重要范式回顾：工具使用（包括RAG）、规划和反馈学习")。在智能体中使用时有几点需要特别注意：1) 对于规划者概况，零-shot CoT 实现通常无法生成长期计划（Wang 等，[2023b](https://arxiv.org/html/2406.05804v6#bib.bib42)）。2) 在自主工具使用工作流程下，特别是采用推理-行动策略时，提示中必须包含工具定义；少量示例对指导生成格式至关重要，包括指示何时停止。

#### 计划生成的工作流程比较

使用 $\text{glm}_{\text{planner}}$ 的基本工作流程和搜索工作流程都会生成一系列行动（即计划）。然而，它们在生成和使用计划的方式上有根本区别。1) 贪婪生成与探索：基本工作流程利用 $\text{glm}_{\text{planner}}$ 贪婪地在单次推理步骤中生成静态计划。这种方法通常在面对复杂任务时难以生成长期计划（Sun 等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib33)）。相比之下，搜索工作流程探索多个潜在的解决方案并支持回溯，允许对选项进行更强有力的探索。2) 计划执行：由 $\text{glm}_{\text{planner}}$、广度优先搜索（BFS）、深度优先搜索（DFS）和束搜索（Beam Search）生成的计划旨在完全执行。然而，这可能导致无法执行的行动，特别是在随机决策环境中，如果先前的行动 $(a_{1},...,a_{t-1})$ 导致下一个行动 $a_{t}$ 变得无效（例如，“站在冰箱前”但需要“打开微波炉”）。这可能会导致智能体的表现出现突然中断。相比之下，在基于模拟的 MCTS 搜索中，只有根节点的行动会在实际环境中执行。随后的模拟状态和行动会被丢弃，尽管某些实现可能会部分保留这些状态以避免重新计算。每次行动后，搜索过程会重复进行，不断重新计算更新状态下的最佳行动。

#### 使用 $\text{glm}_{\text{actor}}$ 的工作流程比较

$\text{glm}_{\text{actor}}$ 根据工作流程执行三种类型的动作之一：1) 在基础和反馈学习工作流程中，$\text{glm}_{\text{actor}}$ 一直生成与任务相关的动作，以便在任务环境中执行（可执行的任务动作）。 2) 在搜索工作流程中，$\text{glm}_{\text{actor}}$ 生成的动作用于规划算法，而不是直接执行（规划任务动作）。 3) 在自主工具使用工作流程中，$\text{glm}_{\text{actor}}$ 负责生成与工具相关的动作（工具动作）。

| 任务表述 | 反馈类型 | 适用的工作流程 | 示例工作 |
| --- | --- | --- | --- |
| 文本生成 | 自由形式的反思 | 反馈学习工作流程 | Self-Refine (Madaan et al., [2023](https://arxiv.org/html/2406.05804v6#bib.bib20)), Reflexion (Shinn et al., [2023](https://arxiv.org/html/2406.05804v6#bib.bib29)), CRITIC (Gou et al., [2024](https://arxiv.org/html/2406.05804v6#bib.bib8)) |
| 二元/多类分类 | 离散值 | 搜索工作流程 | RAP (Hao et al., [2023](https://arxiv.org/html/2406.05804v6#bib.bib11)), Tree-BeamSearch (Xie et al., [2023](https://arxiv.org/html/2406.05804v6#bib.bib45)), ToT (Yao et al., [2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)), Koh et al. ([2024](https://arxiv.org/html/2406.05804v6#bib.bib17)) |
| 二元分类 | 连续值（logits） | 通过 MCTS 的搜索工作流程 | RAP (Hao et al., [2023](https://arxiv.org/html/2406.05804v6#bib.bib11)) |
| 多项选择 QA | 顶级 N 个动作的选择 | 通过遍历和启发式搜索的工作流程 | ToT (Yao et al., [2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)) |

表 4：根据任务表述和反馈类型的 LLM 配置评估器类型

#### 使用 $\text{glm}_{\text{eval}}$ 的工作流程比较

在反馈学习工作流中，生成的内容会传递给$\text{glm}_{\text{policy}}$进行学习，而在搜索工作流中，则用于规划。这个根本性的区别（学习与规划）导致了以下几个区分：1）反馈使用：如图[1](https://arxiv.org/html/2406.05804v6#S0.F1 "Figure 1 ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning")所示，在学习中，输出内容会生成供$\text{glm}_{\text{policy}}$修订并重新生成整个决策。而在搜索工作流中，输出则用于构建搜索树/图，用于选择动作和进一步扩展。2）反馈形式：这些组件——$\text{glm}_{\text{policy}}$和搜索树/图——需要不同形式的反馈。$\text{glm}_{\text{policy}}$处理自由格式的文本，而MCTS使用连续值作为奖励，BFS、DFS或束搜索中的树/图则依赖于离散值来进行节点选择。为了满足这些不同的需求，采用了不同的任务表述，具体总结见表[4](https://arxiv.org/html/2406.05804v6#S5.T4 "Table 4 ‣ Workflow Comparisons for Using 'glm'_'actor' ‣ 5 Discussions ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning")。a) 生成自由格式文本：$\text{glm}_{\text{eval}}$通过提示反思先前的状态和动作，生成反思性文本，作为反馈学习工作流中$\text{glm}_{\text{policy}}$提示的一部分（Shinn等，2023年；Gou等，2024年）。b) 二元/多类分类：$\text{glm}_{\text{eval}}$通过特定约束进行提示，生成离散输出标记，通常为“no”或“yes”。这些标记被转换为标量值（例如0/1），用于MCTS模拟中的奖励（Hao等，2023年），或在树遍历过程中指导决策（Yao等，2023a）。c) 带标量值的二元分类：这种方法与前一种方法不同，采用LLM生成的标记logits作为标量反馈值。例如，“yes”回应的概率计算如下：

|  | $v=\text{probs}_{\text{``yes''}}=\frac{e^{l_{\text{``yes''}}}}{e^{l_{\text{``% yes''}}}+e^{l_{\text{``no''}}}},$ |  |
| --- | --- | --- |

其中$l_{\text{``yes''}}$和$l_{\text{``no''}}$分别是“yes”和“no”标记的logits。⁴⁴4注意，由于最先进的黑箱LLM的不可获取性，这种$\text{glm}_{\text{eval}}$的实现较为少见。这些标量值随后可以作为奖励用于MCTS。d) 多项选择问答：这种表述用于需要从多个选择中选择的场景，例如在基于遍历的搜索工作流中从前N个可能的动作中选择（Yao等人，[2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)）。

## 6 限制与未来工作

#### 限制1：基础工作流和自主工具使用工作流的统一解决方案

ReAct（Yao等人，[2023b](https://arxiv.org/html/2406.05804v6#bib.bib49)）尝试通过推理-行动策略统一自主工具使用工作流和基础工作流。在这个场景中，他们隐性地将工具统一为任务环境的一部分，工具动作和任务动作被统一为类似的格式。然而，这两种工作流依然是不同的：1）以细粒度方式指定并比较不同的工具使用工作流，2）区分这两者可以避免产生工具使用工作流普遍可实施的错觉。具体来说，对于后者，推理和行动输出交替的顺序是任务依赖的。对于问答任务，推理步骤和工具动作的生成是固定的，思考和行动的提示交替出现。⁵⁵5推理-行动策略（问答任务）：参见表[8](https://arxiv.org/html/2406.05804v6#A0.T8 "Table 8 ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning")中的示例提示。相比之下，对于具身任务，是否在下一步继续思考或行动的决策是由$\text{glm}_{\text{policy}}$自主决定的。⁶⁶6推理-行动策略（具身任务）：参见表[6](https://arxiv.org/html/2406.05804v6#A0.T6 "Table 6 ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning")中的示例提示。

#### 限制2：缺乏普适工具使用工作流设计

另一个未来方向是普适工具使用的开发。虽然工具使用是一个普遍适用的范式，但当前的研究倾向于聚焦于使用工具来完成特定任务，例如NLIE-QA或专门目的——要么使得$\text{glm}_{\text{policy}}$能够获取新信息，要么使得$\text{glm}_{\text{eval}}$用于验证。

#### 限制3：将语言任务表述为代理任务

这一公式有助于开发一种更为普遍使用的工作流程，例如，Reflexion（Shinn 等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib29)）。然而，某些刻意设计应当在未来的工作中重新考虑，例如，使用地面真实值作为外部反馈的Remark [3](https://arxiv.org/html/2406.05804v6#Thmremark3 "Remark 3\. ‣ 4.4 Feedback-Learning Workflows ‣ 4 LMPR-Based Workflows ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning")。另一个值得怀疑的设置是故意的多步骤NLIEs，例如，如果一个问题足够简单，不需要被分解成子问题来回答会怎样。

#### 设计新工作流程

我们的系统性回顾揭示了通过将现有工作流程交织在一起来设计新工作流程的潜在方向。这些工作流程可以在单一范式内结合——例如，将不同的反馈源整合，或将基于验证的工具使用与自主工具使用相结合。跨范式结合工作流程的潜力更大，例如，将可选的反馈源纳入非验证工具使用工作流程中。事实上，验证式工作流程（Gou 等人，[2024](https://arxiv.org/html/2406.05804v6#bib.bib8); Jiang 等人，[2023](https://arxiv.org/html/2406.05804v6#bib.bib16)）本质上融合了反馈学习和工具使用范式的元素。

## 7 结论

本次调研正式定义了三种常见的LLM配置角色（LMPRs），并研究了工具使用、规划和反馈学习的普遍工作流程。由于这些并非任务特定的，我们希望这能够为未来研究在不同任务中应用LLM基础的工作流程设计提供启示。此外，在这些LMPRs和工作流程的推动下，我们讨论了LLM配置的一些细微差别、工作流程的优缺点以及框架实现。

## 局限性

由于页面限制，我们只能专注于具有代表性和开创性的工作（主要是*ACL, ICML, ICLR, 和 NeurIPS*）以及与我们焦点一致的观点。这意味着一些近期的工作可能被遗漏，任务特定的组件和完整的工作流程设计大多被忽略，尤其是那些为具身环境设计的框架。例如，通常需要视觉模型将像素观察转换为文本输入，以便进行$\text{glm}_{\text{policy}}$（Wang 等人，[2023a](https://arxiv.org/html/2406.05804v6#bib.bib39)）。再次强调，本综述旨在总结任务无关的工作流程，以促进对其的连贯理解并识别其局限性，以便为未来研究提供参考。

## 参考文献

+   Besta 等人（2024）Maciej Besta、Nils Blach、Ales Kubicek、Robert Gerstenberger、Michal Podstawski、Lukas Gianinazzi、Joanna Gajda、Tomasz Lehmann、Hubert Niewiadomski、Piotr Nyczyk 等人 2024年。《思维图：利用大语言模型解决复杂问题》。发表于*人工智能会议论文集*，第38卷，第17682–17690页。

+   Chen 等人（2024a）陈思佳、李宝春和牛迪。2024a年。[增强思维：使用大型语言模型的试错问题求解](https://openreview.net/forum?id=qBL04XXex6)。在*第十二届国际学习表征会议*。

+   Chen 等人（2024b）陈子如、迈克尔·怀特、雷·穆尼、阿里·帕亚尼、苏宇和孙欢。2024b年。[树搜索何时对LLM规划有用？这取决于判别器](https://doi.org/10.18653/v1/2024.acl-long.738)。在*第62届计算语言学协会年会论文集（第1卷：长篇论文）*，第13659–13678页，泰国曼谷。计算语言学协会。

+   Cobbe 等人（2021）卡尔·科比、维尼特·科萨拉朱、穆罕默德·巴瓦里安、马克·陈、金熙宇、卢卡兹·凯泽、马蒂亚斯·普拉普特、杰里·特沃雷克、雅各布·希尔顿、中野丽一、克里斯托弗·赫塞和约翰·舒尔曼。2021年。训练验证器解决数学文字题。*arXiv 预印本 arXiv:2110.14168*。

+   Dasgupta 等人（2022）伊什塔·达斯古普塔、克里斯汀·凯泽-陈、肯尼斯·马里诺、阿伦·阿胡贾、谢拉·巴巴扬、费利克斯·希尔和罗布·费格斯。2022年。[与语言模型合作进行具身推理](https://openreview.net/forum?id=YoS-abmWjJc)。在*第二届语言与强化学习工作坊*。

+   Fan 等人（2022）范林熙、王冠志、蒋云凡、阿贾伊·曼德尔卡、杨云聪、朱昊义、唐安德鲁、黄德安、朱昱可和安尼玛·安南德库马尔。2022年。[Minedojo：构建具有互联网规模知识的开放式具身智能体](https://openreview.net/forum?id=rc8o_j8I8PX)。在*第36届神经信息处理系统会议数据集与基准跟踪*。

+   Gallotta 等人（2024）罗伯托·加洛塔、格雷厄姆·托德、马尔文·扎米特、萨姆·厄尔、安东尼奥斯·利亚皮斯、朱利安·托格柳斯和乔治·N·扬纳卡基斯。2024年。大型语言模型与游戏：综述与路线图。*arXiv 预印本 arXiv:2402.18659*。

+   Gou 等人（2024）苟志斌、邵志洪、龚叶云、沈业龙、杨宇久、段楠和陈维柱。2024年。[CRITIC：大型语言模型能够通过工具交互批判进行自我纠正](https://openreview.net/forum?id=Sx038qxjek)。在*第十二届国际学习表征会议*。

+   Guan 等人（2023）关琳、卡尔提克·瓦尔梅卡姆、萨拉斯·斯里达兰和苏巴拉奥·坎巴帕提。2023年。[利用预训练的大型语言模型构建和使用世界模型进行基于模型的任务规划](https://openreview.net/forum?id=zDbsSscmuj)。在*第37届神经信息处理系统会议*。

+   Guo 等人（2024）郭太诚、陈秀英、王亚奇、常瑞迪、裴世超、尼特什·V·查乌拉、奥拉夫·维斯特和张向亮。2024年。基于大型语言模型的多智能体：进展与挑战的综述。*arXiv 预印本 arXiv:2402.01680*。

+   Hao 等人（2023）Shibo Hao, Yi Gu, Haodi Ma, Joshua Hong, Zhen Wang, Daisy Wang 和 Zhiting Hu。2023年。[与语言模型推理即为使用世界模型进行规划](https://aclanthology.org/2023.emnlp-main.507)。收录于 *2023年自然语言处理实证方法会议论文集*，第8154–8173页，新加坡。计算语言学协会。

+   Hu 等人（2024）Sihao Hu, Tiansheng Huang, Fatih Ilhan, Selim Tekin, Gaowen Liu, Ramana Kompella 和 Ling Liu。2024年。一项关于基于大型语言模型的游戏代理的调查。*arXiv 预印本 arXiv:2404.02039*。

+   Huang 等人（2022）Wenlong Huang, Pieter Abbeel, Deepak Pathak 和 Igor Mordatch。2022年。语言模型作为零-shot规划器：为具身代理提取可操作的知识。收录于 *国际机器学习会议*，第9118–9147页，PMLR。

+   Huang 等人（2024）Xu Huang, Weiwen Liu, Xiaolong Chen, Xingmei Wang, Hao Wang, Defu Lian, Yasheng Wang, Ruiming Tang 和 Enhong Chen。2024年。理解大型语言模型代理的规划：一项调查。*arXiv 预印本 arXiv:2402.02716*。

+   Inaba 等人（2023）Tatsuro Inaba, Hirokazu Kiyomaru, Fei Cheng 和 Sadao Kurohashi。2023年。[MultiTool-CoT: GPT-3 可以通过链式思维提示使用多个外部工具](https://doi.org/10.18653/v1/2023.acl-short.130)。收录于 *第61届计算语言学协会年会论文集（第2卷：短篇论文）*，第1522–1532页，加拿大多伦多。计算语言学协会。

+   Jiang 等人（2023）Zhengbao Jiang, Frank Xu, Luyu Gao, Zhiqing Sun, Qian Liu, Jane Dwivedi-Yu, Yiming Yang, Jamie Callan 和 Graham Neubig。2023年。[主动检索增强生成](https://doi.org/10.18653/v1/2023.emnlp-main.495)。收录于 *2023年自然语言处理实证方法会议论文集*，第7969–7992页，新加坡。计算语言学协会。

+   Koh 等人（2024）Jing Yu Koh, Stephen McAleer, Daniel Fried 和 Ruslan Salakhutdinov。2024年。语言模型代理的树形搜索。*arXiv 预印本 arXiv:2407.01476*。

+   Kojima 等人（2022）Takeshi Kojima, Shixiang Shane Gu, Machel Reid, Yutaka Matsuo 和 Yusuke Iwasawa。2022年。[大型语言模型是零-shot推理器](https://openreview.net/forum?id=e2TBb5y0yFf)。收录于 *神经信息处理系统进展*。

+   Lewis 等人（2020）Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, Sebastian Riedel 和 Douwe Kiela。2020年。[检索增强生成用于知识密集型自然语言处理任务](https://proceedings.neurips.cc/paper/2020/file/6b493230205f780e1bc26945df7481e5-Paper.pdf)。收录于 *神经信息处理系统进展*，第33卷，第9459–9474页，Curran Associates, Inc.

+   Madaan 等 (2023) Aman Madaan, Niket Tandon, Prakhar Gupta, Skyler Hallinan, Luyu Gao, Sarah Wiegreffe, Uri Alon, Nouha Dziri, Shrimai Prabhumoye, Yiming Yang 等. 2023. Self-refine: 通过自我反馈进行迭代优化. *arXiv 预印本 arXiv:2303.17651*。

+   Manakul 等 (2023) Potsawee Manakul, Adian Liusie, 和 Mark JF Gales. 2023. Selfcheckgpt: 零资源黑箱幻觉检测用于生成大型语言模型. *arXiv 预印本 arXiv:2303.08896*。

+   Markowitz 等 (2024) Elan Markowitz, Anil Ramakrishna, Jwala Dhamala, Ninareh Mehrabi, Charith Peris, Rahul Gupta, Kai-Wei Chang, 和 Aram Galstyan. 2024. [Tree-of-traversals: 一种零样本推理算法，用于通过知识图谱增强黑箱语言模型](https://doi.org/10.18653/v1/2024.acl-long.665). 收录于 *第62届计算语言学协会年会论文集 (卷1: 长篇论文)*，页面 12302–12319，曼谷，泰国。计算语言学协会。

+   Puig 等 (2018) Xavier Puig, Kevin Ra, Marko Boben, Jiaman Li, Tingwu Wang, Sanja Fidler, 和 Antonio Torralba. 2018. Virtualhome: 通过程序模拟家庭活动. 收录于 *IEEE计算机视觉与模式识别大会论文集*，页面 8494–8502。

+   Putta 等 (2024) Pranav Putta, Edmund Mills, Naman Garg, Sumeet Ramesh Motwani, Chelsea Finn, Divyansh Garg, 和 Rafael Rafailov. 2024. [Agent q: 高级推理与学习用于自主AI代理](https://api.semanticscholar.org/CorpusID:271865516)。

+   Russell 和 Norvig (2010) Stuart J Russell 和 Peter Norvig. 2010. *人工智能：一种现代方法*. 伦敦。

+   Santurkar 等 (2023) Shibani Santurkar, Esin Durmus, Faisal Ladhak, Cinoo Lee, Percy Liang, 和 Tatsunori Hashimoto. 2023. 语言模型反映了谁的观点？ *arXiv 预印本 arXiv:2303.17548*。

+   Shi 等 (2024a) Haochen Shi, Zhiyuan Sun, Xingdi Yuan, Marc-Alexandre Côté, 和 Bang Liu. 2024a. [OPEx: LLM为中心的代理在体现指令跟随中的逐组件分析](https://doi.org/10.18653/v1/2024.acl-long.37). 收录于 *第62届计算语言学协会年会论文集 (卷1: 长篇论文)*，页面 622–636，曼谷，泰国。计算语言学协会。

+   Shi 等 (2024b) Weijia Shi, Sewon Min, Michihiro Yasunaga, Minjoon Seo, Richard James, Mike Lewis, Luke Zettlemoyer, 和 Wen-tau Yih. 2024b. [REPLUG: 检索增强黑箱语言模型](https://doi.org/10.18653/v1/2024.naacl-long.463). 收录于 *2024年北美计算语言学协会会议：人类语言技术（卷1：长篇论文）*，页面 8371–8384，墨西哥城，墨西哥。计算语言学协会。

+   Shinn 等人（2023）Noah Shinn, Federico Cassano, Edward Berman, Ashwin Gopinath, Karthik Narasimhan 和 Shunyu Yao. 2023. [Reflexion：具有言语强化学习的语言代理](https://arxiv.org/abs/2303.11366). *预印本*，arXiv:2303.11366。

+   Shridhar 等人（2021）Mohit Shridhar, Xingdi Yuan, Marc-Alexandre Cote, Yonatan Bisk, Adam Trischler 和 Matthew Hausknecht. 2021. [{ALFW}orld: 将文本与具身环境对齐以进行互动学习](https://openreview.net/forum?id=0IOX0YcCdTn). 发表在 *国际学习表征会议*。

+   Shypula 等人（2024）Alexander G Shypula, Aman Madaan, Yimeng Zeng, Uri Alon, Jacob R. Gardner, Yiming Yang, Milad Hashemi, Graham Neubig, Parthasarathy Ranganathan, Osbert Bastani 和 Amir Yazdanbakhsh. 2024. [学习性能提升代码编辑](https://openreview.net/forum?id=ix7rLVHXyY). 发表在 *第十二届国际学习表征会议*。

+   Sprueill 等人（2023）Henry Sprueill, Carl Edwards, Mariefel Olarte, Udishnu Sanyal, Heng Ji 和 Sutanay Choudhury. 2023. [蒙特卡洛思维搜索：大语言模型在催化剂设计中的复杂科学推理查询](https://doi.org/10.18653/v1/2023.findings-emnlp.560). 发表在 *计算语言学协会年会论文集：EMNLP 2023*，第8348-8365页，新加坡。计算语言学协会。

+   Sun 等人（2023）Haotian Sun, Yuchen Zhuang, Lingkai Kong, Bo Dai 和 Chao Zhang. 2023. [Adaplanner：基于反馈的自适应规划与语言模型](https://openreview.net/forum?id=rnKgbKmelt). 发表在 *第37届神经信息处理系统会议*。

+   Sutton 和 Barto（2018）Richard S Sutton 和 Andrew G Barto. 2018. *强化学习：入门*. MIT出版社。

+   Trivedi 等人（2024）Harsh Trivedi, Tushar Khot, Mareike Hartmann, Ruskin Manku, Vinty Dong, Edward Li, Shashank Gupta, Ashish Sabharwal 和 Niranjan Balasubramanian. 2024. [AppWorld：一个可控的应用与人物世界，用于基准测试互动编码代理](https://aclanthology.org/2024.acl-long.850). 发表在 *计算语言学协会第62届年会论文集（第1卷：长篇论文）*，第16022-16076页，泰国曼谷。计算语言学协会。

+   Valmeekam 等人（2022）Karthik Valmeekam, Alberto Olmo, Sarath Sreedharan 和 Subbarao Kambhampati. 2022. [大型语言模型仍无法进行规划（LLM在规划和推理变更方面的基准）](https://openreview.net/forum?id=wUU-7XTL5XO). 发表在 *NeurIPS 2022决策制定基础模型研讨会*。

+   Wan 等人（2024）Ziyu Wan, Xidong Feng, Muning Wen, Ying Wen, Weinan Zhang 和 Jun Wang. 2024. [类似AlphaZero的树搜索可指导大语言模型解码和训练](https://openreview.net/forum?id=fLO9VaAb3B).

+   王等人（2024a）王磊、马辰、冯雪扬、张泽宇、杨浩、张敬森、陈志远、唐家凯、陈旭、林彦凯等人。2024a. 基于大型语言模型的自主代理综述。*计算机科学前沿*，18(6):1–26。

+   王等人（2023a）王磊、徐婉玉、蓝义怀、胡志强、兰云诗、李凯威、林奕鹏。2023a. [计划与解决提示：通过大型语言模型改进零-shot链式思维推理](https://doi.org/10.18653/v1/2023.acl-long.147)。在*第61届计算语言学会年会会议论文集（第一卷：长篇论文）*，第2609–2634页，加拿大多伦多。计算语言学会。

+   王等人（2022）王思源、刘忠坤、钟万君、周鸣、魏中宇、陈竹敏、段楠。2022. [来自lsat：复杂推理的进展与挑战](https://doi.org/10.1109/TASLP.2022.3164218)。*IEEE/ACM音频、语音与语言处理学报*，30:2201–2216。

+   王等人（2024b）王晓华、王正华、高轩、张斐然、吴一鑫、徐志博、史天元、王正远、李世政、钱齐等人。2024b. 寻找最佳实践以增强检索生成的效果。*arXiv预印本 arXiv:2407.01219*。

+   王等人（2023b）王子豪、蔡绍飞、陈冠洲、刘安吉、马小健、梁一涛。2023b. [描述、解释、规划和选择：与LLM互动的规划使开放世界多任务代理成为可能](https://openreview.net/forum?id=KtvPdGb31Z)。在*第37届神经信息处理系统会议*。

+   魏等人（2022）魏杰森、王雪智、达尔·舒尔曼、马尔滕·博斯马、布赖恩·伊切特、夏飞、艾德·H·池、阮文理和邓尼·周。2022. [链式思维提示激发大型语言模型中的推理能力](https://openreview.net/forum?id=_VjQlMeSB_J)。在*神经信息处理系统进展*。

+   习等人（2023）习志恒、陈文祥、郭鑫、何伟、丁易文、洪博阳、张铭、王俊哲、金森杰、周恩宇等人。2023. 基于大型语言模型的代理崛起与潜力：一项调查。*arXiv预印本 arXiv:2309.07864*。

+   谢等人（2023）谢宇曦、川口健司、赵怡然、赵旭、简敏艳、何俊贤、谢启哲。2023. [自评引导的束搜索推理方法](https://openreview.net/forum?id=Bw82hwg5Q3)。在*第37届神经信息处理系统会议*。

+   杨等人（2018）杨智麟、齐鹏、张赛征、约书亚·本吉奥、威廉·科恩、鲁斯兰·萨拉赫赫丁诺夫、克里斯托弗·D·曼宁。2018. [HotpotQA：一个多样化、可解释的多跳问题解答数据集](https://doi.org/10.18653/v1/D18-1259)。在*2018年自然语言处理实证方法会议论文集*，第2369–2380页，比利时布鲁塞尔。计算语言学会。

+   姚等人（2022）姚顺宇、陈浩宇、杨约翰、卡尔蒂克·纳拉辛汉。2022。[Webshop：面向可扩展的真实世界网页交互与有根语言代理](https://doi.org/10.18653/v1/2022.findings-naacl.177)。*神经信息处理系统进展*，35:20744–20757。

+   姚等人（2023a）姚顺宇、俞典、赵杰弗里、沙弗兰·伊扎克、托马斯·L·格里菲斯、曹远、卡尔蒂克·纳拉辛汉。2023a。[思想树：在大语言模型中进行深思熟虑的问题解决](https://arxiv.org/abs/2305.10601)。*预印本*，arXiv:2305.10601。

+   姚等人（2023b）姚顺宇、赵杰弗里、俞典、杜楠、沙弗兰·伊扎克、卡尔蒂克·R·纳拉辛汉、曹远。2023b。[React：在语言模型中协同推理与行动](https://openreview.net/forum?id=WE_vluYUL-X)。发表于*第十一届国际学习表示会议*。

+   于等人（2023）于晓、陈马克西米连、于周。2023。[基于提示的蒙特卡洛树搜索用于目标导向对话策略规划](https://doi.org/10.18653/v1/2023.emnlp-main.439)。发表于*2023年自然语言处理实证方法会议论文集*，第7101–7125页，新加坡。计算语言学会。

+   俞塞克贡尔等人（2024）梅尔特·俞塞克贡尔、费德里科·比安基、约瑟夫·博恩、刘盛、黄志、卡洛斯·格斯特林、詹姆斯·邹。2024。Textgrad：通过文本实现自动"微分"。*arXiv预印本arXiv:2406.07496*。

+   张等人（2024）张佩天、刘峥、肖仕涛、窦志成、聂建云。2024。[用于检索增强型大语言模型的多任务嵌入器](https://doi.org/10.18653/v1/2024.acl-long.194)。发表于*第62届计算语言学会年会（第一卷：长篇论文）*，第3537–3553页，泰国曼谷。计算语言学会。

+   赵等人（2023）赵子锐、李伟孙、许大卫。2023。[大语言模型作为常识知识用于大规模任务规划](https://openreview.net/forum?id=Wjp1AYB8lH)。发表于*第37届神经信息处理系统大会*。

+   钟等人（2023）钟万俊、崔瑞翔、郭奕多、梁耀博、吕帅、王彦林、赛义德·阿敏、陈伟珠、段楠。2023. Agieval：一种以人为本的基准评估框架，用于评估基础模型。*arXiv预印本arXiv:2304.06364*。

+   钟等人（2022）钟万俊、王思远、唐独宇、许泽南、郭大亚、陈逸宁、王家海、尹建、周铭、段楠。2022。[文本的分析推理](https://doi.org/10.18653/v1/2022.findings-naacl.177)。发表于*计算语言学会发现：NAACL 2022*，第2306–2319页，美国西雅图。计算语言学会。

+   周等人（2024a）周安迪、颜凯、米哈伊·什拉潘托赫-罗斯曼、王浩瀚、王宇雄。2024a。[语言代理树搜索统一了大语言模型中的推理、行动与规划](https://openreview.net/forum?id=6LNTSrJjBe)。

+   Zhou 等人（2024b）Shuyan Zhou、Frank F. Xu、Hao Zhu、Xuhui Zhou、Robert Lo、Abishek Sridhar、Xianyi Cheng、Tianyue Ou、Yonatan Bisk、Daniel Fried、Uri Alon 和 Graham Neubig。2024b。[Webarena: 用于构建自主体的现实Web环境](https://openreview.net/forum?id=oKn9c6ytLx)。发表于 *第十二届国际学习表征会议*。

| 问：在一个有20名学生的舞蹈班里，$20\%$ 的学生报名参加了现代舞，剩余学生的$25\%$ 报名参加了爵士舞，其余的报名参加了街舞。那么，所有学生中有多少百分比报名参加了街舞？答：首先，让我们理解问题并制定解决问题的计划。

然后，让我们按照计划一步步解决问题。

计划：<svg class="ltx_picture" height="192.22" id="A0.T5.3.3.3.3.p1.pic1" overflow="visible" version="1.1" width="603.94"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,192.22) matrix(1 0 0 -1 0 0) translate(0,3.94)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="163.28" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">第1步：计算报名参加现代舞和爵士舞的学生总数。

第2步：计算报名参加街舞的学生总数。

第3步：计算报名参加街舞的学生百分比。

解决方案：

第1步：$20\%$ 的20名学生是4名学生。剩下的16名学生中$25\%$ 是4名学生。所以，总共有8名学生报名参加了现代舞和爵士舞。

第2步：剩下的12名学生报名参加了街舞。

第3步：报名参加街舞的学生百分比是 $12/20=60\%$。</foreignobject></g></g></svg> 因此，答案（阿拉伯数字）是 <svg class="ltx_picture" height="40.09" id="A0.T5.4.4.4.4.p2.pic1" overflow="visible" version="1.1" width="603.94"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,40.09) matrix(1 0 0 -1 0 0) translate(0,3.94)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="11.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">60%</foreignobject></g></g></svg> |

表5：在 NLIE-QA 下的 LLM 规划器，采用零-shot CoT 实现。CoT 触发器以红色字体突出显示。阴影框中的内容由 LLM 生成。来源于 Wang 等人（[2023a](https://arxiv.org/html/2406.05804v6#bib.bib39)）

| 与一个家庭互动以解决任务。以下是两个示例。{Demo 1}

{Demo 2}

任务是：你在房间的中央。快速环顾四周，你看到了一个橱柜 6、一个橱柜 5、一个橱柜 4、一个橱柜 3、一个橱柜 2、一个橱柜 1、一个咖啡机 1、一个台面 3、一个台面 2、一个台面 1、一个抽屉 3、一个抽屉 2、一个抽屉 1、一个冰箱 1、一个垃圾桶 1、一个微波炉 1、一个架子 3、一个架子 2、一个架子 1、一个水槽 1、一个炉灶 4、一个炉灶 3、一个炉灶 2、一个炉灶 1 和一个烤面包机 1\。

你的任务是：将冷却的番茄放入微波炉。

> <svg class="ltx_picture" height="55.16" id="A0.T6.1.1.1.1.p1.pic1" overflow="visible" version="1.1" width="603.94"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,55.16) matrix(1 0 0 -1 0 0) translate(0,3.94)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="26.21" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">思考：为了解决这个任务，我需要找到一个番茄，然后用冰箱将其冷却，最后放入微波炉。</foreignobject></g></g></svg> 好的。
> 
> <svg class="ltx_picture" height="73.76" id="A0.T6.2.2.2.2.p2.pic1" overflow="visible" version="1.1" width="603.94"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,73.76) matrix(1 0 0 -1 0 0) translate(0,3.94)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="44.82" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">思考：首先，我需要找到一个番茄。番茄更可能出现在台面（1-3）、橱柜（1-6）、抽屉（1-3）、架子（1-3）或水槽里。我可以从检查台面 1 开始。</foreignobject></g></g></svg> 好的。 > <svg class="ltx_picture" height="40.55" id="A0.T6.3.3.3.3.p3.pic1" overflow="visible" version="1.1" width="603.94"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,40.55) matrix(1 0 0 -1 0 0) translate(0,3.94)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="11.61" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">前往台面 1</foreignobject></g></g></svg> 在台面 1 上，你看到一个洗碗海绵 1 和一个杯子 2。…

表 6：在阿尔弗雷德世界基准测试中的基本工作流下的LLM模型，使用少量样本 CoT 实现。来源于 (Yao et al., [2023b](https://arxiv.org/html/2406.05804v6#bib.bib49))。

| 使用以下工具来解答一些问题： |
| --- |
| - <<计算器>> |
| - <<化学反应预测器>> |
| - <<摩尔质量列表>> |
| 问题：通过将 2 摩尔的 $\mathrm{CO}2$ 和 2 摩尔的 $\mathrm{H}2\mathrm{O}$ 结合，形成多少摩尔的 $\mathrm{H}2\mathrm{CO}3$？ |
| A: 首先，使用化学反应预测器确定反应的化学方程式。 |
| 反应物：$2\mathrm{CO}2,2\mathrm{H}2\mathrm{O}$ |
| 产物：?$\mathrm{H}2\mathrm{CO}3$ |
| <<化学反应预测器>> |
| $2\mathrm{CO}2+2\mathrm{H}2\mathrm{O}\rightarrow 2\mathrm{H}2\mathrm{CO}3$ |
| 其次，使用摩尔质量列表来确定$\mathrm{H}2\mathrm{CO}3$的摩尔质量。$\mathrm{H}2\mathrm{CO}3$ <<摩尔质量列表>> $62\mathrm{~{}g}/\mathrm{mol}$ |
| 第三，使用计算器计算2摩尔$\mathrm{H}2\mathrm{CO}3$的量。$2\times 62=$ <<计算器>> $124$ |
| 因此，形成了$124\mathrm{~{}g}$的$\mathrm{H}2\mathrm{CO}3$。 |
| 问题：找出与2摩尔二氧化碳反应生成2摩尔碳酸钙和2摩尔水所需的氢氧化钙的量。 |
| 答案： |

表7：工具使用工作流中的LLM演员。生成触发器以红色字体突出显示。来源：MultiTool-CoT（Inaba等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib15)）

| 通过先进行思考，再给出答案来解决问题回答任务。思考可以推理当前情况。完成[答案]返回答案并完成任务。 |
| --- |
| 下面是一些示例： |
| 问题1：科罗拉多造山带东部地区的海拔范围是多少？ |
| 思考：让我们一步一步思考。科罗拉多造山带东部地区延伸到高原。高原的海拔范围从大约1,800英尺到7,000英尺，因此答案是1,800到7,000英尺。 |
| 动作：完成[1,800到7,000英尺] |
| … |
| 问题6：帕维尔·乌里松和列昂尼德·列文是否以相同类型的工作而闻名？ |
| 思考：让我们一步一步思考。帕维尔·乌里松（Pavel Urysohn）是数学家。列昂尼德·列文（Leonid Levin）是数学家和计算机科学家。因此，帕维尔·乌里松和列昂尼德·列文从事相同类型的工作。 |
| 动作：完成[是] |
| 示例结束) |
| 问题：VIVA Media AG在2004年更改了名称。它们的新缩写代表什么？ |
| 思考：<svg class="ltx_picture" height="92.59" id="A0.T8.1.1.1.1.p1.pic1" overflow="visible" version="1.1" width="603.94"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,92.59) matrix(1 0 0 -1 0 0) translate(0,3.94)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="63.65" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">让我们一步一步思考。VIVA Media AG在2004年更改了名称。新的缩写必须代表他们采用的新名称。为了找到答案，我们需要研究2004年后VIVA Media AG的新名称。动作：完成[需要进一步研究以确定新的缩写。]</foreignobject></g></g></svg> |
| 动作: <svg class="ltx_picture" height="42.78" id="A0.T8.3.3.1.1.p1.pic1" overflow="visible" version="1.1" width="603.94"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,42.78) matrix(1 0 0 -1 0 0) translate(0,3.94)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">完成[需要进一步研究以确定新的缩写词。]</foreignobject></g></g></svg> |

表 8: 用于HotPot-QA基准测试的LLM演员，结合工具使用工作流，通过少量样本的CoT实现。来源于ReAct（Yao 等人，[2023b](https://arxiv.org/html/2406.05804v6#bib.bib49)）。

| 给定一个问题，请将其分解为子问题。对于每个子问题，请用完整的句子回答，并以“答案是”结束。当原问题可以回答时，请以“现在我们可以回答这个问题了”开头。 |
| --- |
| 问题 1: 詹姆斯每周给两个不同的朋友写三页的信，每周写几页？ |
| 问题 1.1: 他每周写多少页？ |
| 答案 1.1: 詹姆斯每周给两个不同的朋友写三页的信，所以他每周写3 * 2 * 2 = 12页。答案是12。 |
| 问题 1.2: 一年有多少周？ |
| 答案 1.2: 一年有52周。答案是52。 |
| 问题 1.3: 现在我们可以回答这个问题了：他一年写多少页？ |
| 答案 1.3: 詹姆斯每周写12页，因此他一年写12 * 52 = 624页。答案是624。 |
| … |
| 问题 5: 珍妮特的鸭子每天产16个鸡蛋。她每天早上吃三个鸡蛋，并为她的朋友做四个鸡蛋松饼。她每天在农贸市场上以每个2美元的价格卖掉剩余的鸡蛋。她每天在农贸市场上能赚多少钱？ |
| 问题 5.1: <svg class="ltx_picture" height="55.16" id="A0.T9.1.1.1.1.p1.pic1" overflow="visible" version="1.1" width="603.94"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,55.16) matrix(1 0 0 -1 0 0) translate(0,3.94)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="26.21" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">珍妮特早餐吃了三个鸡蛋，做了四个鸡蛋松饼后，还剩下多少个鸡蛋？</foreignobject></g></g></svg> |

表 9: 用于GSM8K基准测试的LLM演员。来源于Hao 等人（[2023](https://arxiv.org/html/2406.05804v6#bib.bib11)）。

## 附录 A 示例提示

### A.1 LLM-配置的策略模型

1) 基础工作流：规划者和执行者的提示分别在表格[5](https://arxiv.org/html/2406.05804v6#A0.T5 "表格 5 ‣ 一项关于LLM基础代理的显著范式回顾：工具使用（包括RAG）、规划与反馈学习")和[6](https://arxiv.org/html/2406.05804v6#A0.T6 "表格 6 ‣ 一项关于LLM基础代理的显著范式回顾：工具使用（包括RAG）、规划与反馈学习")中展示。阴影框中的内容由LLM生成。 2) 工具使用工作流：NLIE-QA下的执行者展示在表格[7](https://arxiv.org/html/2406.05804v6#A0.T7 "表格 7 ‣ 一项关于LLM基础代理的显著范式回顾：工具使用（包括RAG）、规划与反馈学习")和[8](https://arxiv.org/html/2406.05804v6#A0.T8 "表格 8 ‣ 一项关于LLM基础代理的显著范式回顾：工具使用（包括RAG）、规划与反馈学习")中。前者展示了生成触发器的实现，后者则展示了ReAct实现。 3) 搜索工作流：在RAP工作流的MCTS扩展阶段需要执行者。提示和预期生成内容展示在表格[9](https://arxiv.org/html/2406.05804v6#A0.T9 "表格 9 ‣ 一项关于LLM基础代理的显著范式回顾：工具使用（包括RAG）、规划与反馈学习")中。

### A.2 LLM评估者提示

表格[10](https://arxiv.org/html/2406.05804v6#A1.T10 "表格 10 ‣ A.2 LLM评估者提示 ‣ 附录A 示例提示 ‣ 一项关于LLM基础代理的显著范式回顾：工具使用（包括RAG）、规划与反馈学习")和[11](https://arxiv.org/html/2406.05804v6#A1.T11 "表格 11 ‣ A.2 LLM评估者提示 ‣ 附录A 示例提示 ‣ 一项关于LLM基础代理的显著范式回顾：工具使用（包括RAG）、规划与反馈学习")展示了被归类为分类任务的LLM评估者，而表格[13](https://arxiv.org/html/2406.05804v6#A1.T13 "表格 13 ‣ A.2 LLM评估者提示 ‣ 附录A 示例提示 ‣ 一项关于LLM基础代理的显著范式回顾：工具使用（包括RAG）、规划与反馈学习")则展示了生成自由格式文本的另一个任务。

| 给定一个问题和一些子问题，判断最后一个子问题是否对回答该问题有用。输出“是”或“否”，并给出理由。 |
| --- |
| 问题 1：四年前，Kody的年龄只有Mohamed的一半。如果Mohamed现在30岁，那么Kody几岁？ |
| 问题 1.1：Mohamed几岁？ |
| 问题 1.2：四年前，Mohamed几岁？ |
| 新问题 1.3：四年前，Kody几岁？ |
| 这个新问题有用吗？是的。我们需要这个答案来计算Kody现在几岁。 |
| … |
| 问题 5：Janet 的鸭子每天下 16 个蛋。她每天早晨吃三个作为早餐，并用其中四个为朋友们烤松饼。她每天在农贸市场以每个新鲜鸭蛋 2 美元的价格将剩余的卖出。她每天在农贸市场赚多少钱？ |
| 新问题 5.1：现在我们可以回答这个问题：她每天在农贸市场赚多少钱？ |
| 新问题是否有用？ |

表 10：一个基于仿真搜索工作流的 LLM 评估器，用于 NLIE-QA，采用少量样本 CoT 实现。它评估新子问题在解决原始问题中的有用性。来源：Hao 等人（[2023](https://arxiv.org/html/2406.05804v6#bib.bib11)）。

| 评估给定的数字是否可以达到 24（确定/可能/不可能） 10 14

10 + 14 = 24

确定

…

1 3 3

1 * 3 * 3 = 9

(1 + 3) * 3 = 12

1 3 3 都太小

不可能

11，12 |

表 11：在游戏 24 中，基于思维树工作流的 LLM 评估器，采用少量样本 CoT 提示法实现。该 LLM 针对多类分类进行了配置。来源：ToT（Yao 等，[2023a](https://arxiv.org/html/2406.05804v6#bib.bib48)）。

| 上下文：… 句子：…

句子是否得到上述上下文的支持？

答案是“是”还是“否”：

表 12：一个 LLM 评估器。该 LLM 针对多类分类进行了配置。来源：Manakul 等人（[2023](https://arxiv.org/html/2406.05804v6#bib.bib21)）。

| {为简洁起见省略了少量示范} |
| --- |
| 问题：Serianna 是什么风格的乐队，结合了重金属和硬核朋克的元素？ |
| 提议的答案：让我们一步步思考。Serianna 是一支金属核风格的乐队。金属核是重金属和硬核朋克的子类型。所以，Serianna 是一支重金属和硬核朋克风格的乐队。因此，答案是：重金属和硬核朋克。 |
| 1. 可信度：<svg class="ltx_picture" height="138.72" id="A1.T13.1.1.1.1.p1.pic1" overflow="visible" version="1.1" width="603.94"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,138.72) matrix(1 0 0 -1 0 0) translate(0,3.94)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="109.78" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">问题询问的是结合了重金属和硬核朋克元素的风格，答案是“重金属和硬核朋克”，只是重复了问题本身。因此，这不太可信。 2. 真实性：让我们在谷歌上搜索这个问题：> 搜索查询：Serianna 是什么风格的乐队，结合了重金属和硬核朋克的元素？ > 证据：</foreignobject></g></g></svg> [金属核 - 维基百科] 金属核是一种融合了极端金属和硬核朋克元素的音乐风格。 |
| <svg class="ltx_picture" height="105.51" id="A1.T13.4.4.1.1.p1.pic1" overflow="visible" version="1.1" width="603.94"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,105.51) matrix(1 0 0 -1 0 0) translate(0,3.94)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="76.57" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">证据表明，金属核是一种结合了极限金属和硬核朋克元素的类型，正如提出的答案所述。让我们在 Google 中搜索提出的答案：> 搜索查询：Serianna 是一支金属核乐队。> 证据：</foreignobject></g></g></svg> [Serianna - 维基百科] Serianna 是一支来自威斯康星州麦迪逊的金属核乐队。该乐队成立于 2006 年…… |
| <svg class="ltx_picture" height="107.66" id="A1.T13.7.7.1.1.p1.pic1" overflow="visible" version="1.1" width="603.94"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,107.66) matrix(1 0 0 -1 0 0) translate(0,3.94)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 20.38 12.5)"><foreignobject color="#000000" height="78.72" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="559.25">证据表明，Serianna 是一支金属核乐队，提出的答案是正确的。最重要的是，提出的答案正确地识别了 Serianna 是一支结合了重金属和硬核朋克元素的金属核乐队。然而，最终答案并不可信，因为它仅列出了金属核所结合的类型。</foreignobject></g></g></svg> |

表 13：反馈学习工作流中的 LLM 评估器（来自工具的反馈）。生成中的触发器以红色字体突出显示，工具生成的内容以绿色字体突出显示。来源：Gou 等人（[2024](https://arxiv.org/html/2406.05804v6#bib.bib8)）。 |

### A.3 动态模型提示

表 [14](https://arxiv.org/html/2406.05804v6#A1.T14 "表 14 ‣ A.3 动态模型提示 ‣ 附录 A 示例提示 ‣ 基于 LLM 的代理：工具使用（包括 RAG）、规划和反馈学习的著名范式回顾") 被描述为动态模型。 |

| 给定一个问题，请将其分解为子问题。对于每个子问题，请用完整的句子回答，并以“答案是”结尾。当原问题可以回答时，请以“现在我们可以回答这个问题：”开头子问题。 |
| --- |
| 问题 1: Weng 为照看孩子每小时赚 $12。昨天，她只做了 50 分钟的照看工作。她赚了多少钱？ |
| 问题 1.1: Weng 每分钟赚多少钱？ |
| 答案 1.1: 由于 Weng 为照看孩子每小时赚 $12，她每分钟赚 $12 / 60 = $0.2。答案是 0.2。 |
| 问题 1.2: 现在我们可以回答这个问题：她赚了多少钱？ |
| 答案 1.2: 工作了 50 分钟，她赚了 $0.2 x 50 = $10。答案是 10。 |
| … |
| 问题 5: Janet的鸭子每天产16个蛋。她每天早上早餐吃三个，且每天用四个做松饼给她的朋友们吃。她每天在农贸市场以每个$2的价格卖掉剩余的鸭蛋。她每天在农贸市场赚多少钱？ |
| 问题 5.1: Janet在早餐吃了三个并用四个做了松饼后，剩下多少个蛋？ |
| 答案 5.1: |

表格 14: 一个LLM配置动态模型。

## 附录 B LLM角色配置提示方法

不同提示方法的使用总结见表格 [15](https://arxiv.org/html/2406.05804v6#A2.T15 "表格 15 ‣ 附录 B LLM角色配置提示方法 ‣ LLM驱动代理的突出范式综述：工具使用（包括RAG）、规划与反馈学习")。

|  | 提示 | 示例作品 | 示例提示（见附录） |
| --- | --- | --- | --- |
| $\text{glm}_{\text{actor}}$ | 少量示例 | ReAct（Yao等，[2023b](https://arxiv.org/html/2406.05804v6#bib.bib49)），Reflexion（Shinn等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib29)），RAP（Hao等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib11)），MultiTool-CoT（Inaba等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib15)） | 表格 [7](https://arxiv.org/html/2406.05804v6#A0.T7 "表格 7 ‣ LLM驱动代理的突出范式综述：工具使用（包括RAG）、规划与反馈学习")， [8](https://arxiv.org/html/2406.05804v6#A0.T8 "表格 8 ‣ LLM驱动代理的突出范式综述：工具使用（包括RAG）、规划与反馈学习") |
| $\text{glm}_{\text{planner}}$ | 零示例 | 计划与解决（Wang等，[2023a](https://arxiv.org/html/2406.05804v6#bib.bib39)），LLM规划器（Huang等，[2022](https://arxiv.org/html/2406.05804v6#bib.bib13)） | 表格 [5](https://arxiv.org/html/2406.05804v6#A0.T5 "表格 5 ‣ LLM驱动代理的突出范式综述：工具使用（包括RAG）、规划与反馈学习") |
|  | 少量示例 | DEPS（Wang等，[2023b](https://arxiv.org/html/2406.05804v6#bib.bib42)），Planner-Actor-Reporter（Dasgupta等，[2022](https://arxiv.org/html/2406.05804v6#bib.bib5)） |  |
| $\text{glm}_{\text{evaluator}}$ | 少量示例 | RAP（Hao 等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib11)），Tree-BeamSearch（Xie 等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib45)），Reflexion（Shinn 等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib29)），CRITIC（Gou 等，[2024](https://arxiv.org/html/2406.05804v6#bib.bib8)） | 表 [10](https://arxiv.org/html/2406.05804v6#A1.T10 "Table 10 ‣ A.2 Prompts for LLM Evaluators ‣ Appendix A Examples Prompts ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning")，[11](https://arxiv.org/html/2406.05804v6#A1.T11 "Table 11 ‣ A.2 Prompts for LLM Evaluators ‣ Appendix A Examples Prompts ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning") |
| $\text{glm}_{\text{dynamic}}$ | 少量示例 | RAP（Hao 等，[2023](https://arxiv.org/html/2406.05804v6#bib.bib11)） | 表 [14](https://arxiv.org/html/2406.05804v6#A1.T14 "Table 14 ‣ A.3 Prompts as Dynamic Models ‣ Appendix A Examples Prompts ‣ A Review of Prominent Paradigms for LLM-Based Agents: Tool Use (Including RAG), Planning, and Feedback Learning") |

表 15：LLM 定义角色的提示方法
