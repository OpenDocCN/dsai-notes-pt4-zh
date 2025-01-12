<!--yml

类别：未分类

日期：2025-01-11 13:05:29

-->

# 面向任务导向的代理协作的动态LLM驱动代理网络

> 来源：[https://arxiv.org/html/2310.02170/](https://arxiv.org/html/2310.02170/)

刘子俊¹，张雁哲²，李鹏³，刘杨^(1,3,4)，杨迪一⁵

¹清华大学计算机科学与技术系，人工智能研究院，北京，中国

²乔治亚理工学院，美国乔治亚州

³清华大学人工智能产业研究院，北京，中国

⁴江苏省语言能力协同创新中心，中国江苏

⁵斯坦福大学，美国加利福尼亚州

liuzijun20@mails.tsinghua.edu.cn，diyiy@stanford.edu 本工作完成时，第一作者为斯坦福大学的UGVR访问学生。

###### 摘要

最近的研究表明，多个大型语言模型（LLM）驱动的代理协作是一种解决任务的有前景方法。然而，当前的方法受到固定数量代理和静态通信结构的限制。在本工作中，我们提出了一种自动从候选代理中选择团队进行协作的方式，采用动态通信结构来应对不同的任务和领域。具体来说，我们构建了一个名为动态LLM驱动代理网络（DyLAN）的框架，用于LLM驱动的代理协作，操作一个两阶段的范式：（1）团队优化和（2）任务解决。在第一阶段，我们利用一种基于无监督度量的代理选择算法，称为*代理重要性得分*，使得可以根据代理在初步试验中的贡献，面向给定任务选择最佳代理。然后，在第二阶段，选定的代理根据查询动态协作。从经验上看，我们证明DyLAN在代码生成、决策制定、常规推理和算术推理任务中超越了强基准，并具有适中的计算成本。在MMLU的特定科目中，在团队优化阶段选择代理团队，使得DyLAN的准确率提高了最高25.0%。¹¹1 代码可在[https://github.com/SALT-NLP/DyLAN](https://github.com/SALT-NLP/DyLAN)获取。

![参见说明文字](img/3408a2dae071fdc7e8f7ac25d0666925.png)

图1：DyLAN采用两阶段范式。代理在时间前馈网络（T-FFN）结构中进行通信。在“团队优化”阶段，DyLAN执行代理选择，选择在初步协作中最具贡献的代理，面向任务或领域。然后，在“任务解决”阶段，选定的代理根据给定查询动态协作给出答案。

## 1 引言

大型语言模型（LLM）代理（Richards 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib38)；Nakajima，[2023](https://arxiv.org/html/2310.02170v2#bib.bib27)；Reworkd，[2023](https://arxiv.org/html/2310.02170v2#bib.bib37)）在多个任务中表现出色，包括推理（Yao 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib51)）、代码生成（Shinn 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib41)）、以及诸如视频游戏（Wang 等，[2023a](https://arxiv.org/html/2310.02170v2#bib.bib43)）和自动驾驶系统（Jin 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib17)）等体现任务。鉴于单一代理在高效管理所有这些任务方面的不可行性，近期的研究已转向多代理协作，取得了显著的进展（Li 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib18)；Du 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib10)；Wang 等，[2023c](https://arxiv.org/html/2310.02170v2#bib.bib45)；Jiang 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib16)；Shinn 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib41)；Chen 等，[2024](https://arxiv.org/html/2310.02170v2#bib.bib7)；Wu 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib47)）。

作为对人类社会的类比，人类团队的运作方式可能为开发更有效的多代理协作系统提供宝贵的见解。例如，近期的研究表明，某些来自人类社会的有效沟通结构，在多代理协作中也发挥了积极作用（Yin 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib52)；Chen 等，[2024](https://arxiv.org/html/2310.02170v2#bib.bib7)）。除了沟通结构外，人类团队的另一个显著特征是，他们会根据任务的不同优化团队成员。以医学会诊为例。在会诊过程中，随着讨论的深入，一些医生可能变得不再相关，进而“退出”会诊，导致沟通结构相应变化，体现出团队在动态结构中的协作优化。在针对不同疾病的会诊中，初期医生的组成也会有所不同，受到相关医学领域变化和各个医生贡献的影响。这些特征引发了一个重要的问题：*一个动态变化的代理团队是否同样有利于基于LLM的代理协作？*

然而，这个问题尚未得到很好的解决。尽管各种通信结构已经被研究用于不同的任务，如用于推理的辩论 (Du et al., [2023](https://arxiv.org/html/2310.02170v2#bib.bib10); Liang et al., [2023](https://arxiv.org/html/2310.02170v2#bib.bib19); Xiong et al., [2023](https://arxiv.org/html/2310.02170v2#bib.bib49))，以及用于编码的自我协作 (Dong et al., [2023](https://arxiv.org/html/2310.02170v2#bib.bib9); Qian et al., [2023a](https://arxiv.org/html/2310.02170v2#bib.bib32); [b](https://arxiv.org/html/2310.02170v2#bib.bib33))，这些通信结构并未改变代理团队中的成员，且在整个协作过程中保持不变。这表明，当前的研究尚未彻底探索代理的任务导向动态选择问题。此外，在代理团队的背景下，大多数现有研究选择从人类先验知识出发手工构建代理 (Liu et al., [2023](https://arxiv.org/html/2310.02170v2#bib.bib22); Nakajima, [2023](https://arxiv.org/html/2310.02170v2#bib.bib27); Hong et al., [2024](https://arxiv.org/html/2310.02170v2#bib.bib15); Shinn et al., [2023](https://arxiv.org/html/2310.02170v2#bib.bib41); Li et al., [2023](https://arxiv.org/html/2310.02170v2#bib.bib18))，或使用大型语言模型（LLM）来生成它们 (Wang et al., [2023c](https://arxiv.org/html/2310.02170v2#bib.bib45); Chen et al., [2023b](https://arxiv.org/html/2310.02170v2#bib.bib5); Christianos et al., [2023](https://arxiv.org/html/2310.02170v2#bib.bib8))。这些方法通常会预定义代理，而没有进一步验证协作过程。这导致了静态的代理团队，或是在没有原则性验证的情况下重建团队 (Chen et al., [2024](https://arxiv.org/html/2310.02170v2#bib.bib7))。优化方法仍然面临挑战。

为了回答上述问题，我们提出了一种新的框架，称为动态LLM驱动的代理网络（DyLAN）。DyLAN通过时间前馈网络（T-FFN）来概念化多代理协作。在这一模型中，每个代理的通信步骤对应一个网络层，其中节点表示该步骤中参与的代理，边表示代理之间的通信，从而不依赖于特定的动态代理团队。DyLAN在两个阶段内进行，以实现任务导向的代理协作（图 [1](https://arxiv.org/html/2310.02170v2#S0.F1 "图 1 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")）。第一个阶段称为团队优化，在该阶段中，我们根据任务查询，基于代理的个体贡献，选择最具贡献的代理，这一过程是无监督的。我们在[第3.4节](https://arxiv.org/html/2310.02170v2#S3.SS4 "3.4 团队优化 ‣ 3 动态LLM驱动代理网络 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")提出了一种前向-反向消息传递算法，称为代理选择，灵感来自反向传播算法（Rumelhart等， [1986](https://arxiv.org/html/2310.02170v2#bib.bib40)）和神经元重要性得分（Yu等，[2018](https://arxiv.org/html/2310.02170v2#bib.bib53)）。该算法通过一种无监督度量，称为*代理重要性得分*，来衡量每个代理在第一个阶段的贡献。最具贡献的代理组成一个较小的团队，在第二阶段——任务解决中进行协作，从而最大限度地减少效果不佳的代理对最终答案的影响。具体来说，协作开始时是由一个代理团队组成，而一个LLM驱动的排序器在中间动态地停用表现不佳的代理（即代理团队重组），从而扩展了T-FFN，并将动态通信结构融入到DyLAN中（[第3.3.2节](https://arxiv.org/html/2310.02170v2#S3.SS3.SSS2 "3.3.2 代理团队重组 ‣ 3.3 任务解决 ‣ 3 动态LLM驱动代理网络 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")）。通过引入代理选择，DyLAN有效地以一种原则化的方式识别并协调任务导向的代理团队。大量实验表明，DyLAN在各种任务中优于强基线，包括代码生成、决策制定、一般推理和算术推理。值得注意的是，DyLAN中的代理选择在MMLU数据集的某些领域中提高了最多25.0%的准确率（Hendrycks等， [2021a](https://arxiv.org/html/2310.02170v2#bib.bib12)），这凸显了动态代理团队的重要性。

总结来说，我们的贡献有三方面： 

+   •

    我们提出了一种名为DyLAN的新框架，用于在两个阶段内进行任务导向的代理协作，并进行代理选择，这标志着动态代理团队研究的重大进展。

+   •

    DyLAN创新性地将代理协作形式化为具有代理团队重组的时间前馈网络，增强了其适应性并减少了对人类预设的依赖。

+   •

    实证结果表明，DyLAN在各种任务中的准确性、效率和稳定性优于其他方法，突显了动态代理团队的必要性。

## 2 相关工作

LLM驱动的代理团队优化 代理团队的构建是LLM驱动的代理协作中的核心和初步步骤。TPTU（Ruan等人，[2023](https://arxiv.org/html/2310.02170v2#bib.bib39)）和Chameleon（Lu等人，[2023](https://arxiv.org/html/2310.02170v2#bib.bib23)）通过任务分解来选择或创建相应的工具。近期的研究也使用LLMs生成固定数量的角色提示，以应对任务查询（Wang等人，[2023c](https://arxiv.org/html/2310.02170v2#bib.bib45)；Suzgun & Tauman Kalai，[2024](https://arxiv.org/html/2310.02170v2#bib.bib42)），或针对每轮讨论（Chen等人，[2024](https://arxiv.org/html/2310.02170v2#bib.bib7)）。然而，手动提示需要精心设计，这对于每个任务或领域的适应性来说是不切实际的，且以预定义或生成的描述来提示LLMs可能在没有验证的情况下无法实现代理所需的能力。因此，基于任务，根据代理在协作中的实际行为后续选择代理团队变得至关重要。虽然LLM代理的团队优化是一个相对较新的领域，但人类团队优化已经研究了很长时间。例如，Liu等人（[2015](https://arxiv.org/html/2310.02170v2#bib.bib20)）表明，技能贡献对于选择众包工作者高效解决外包任务至关重要。基于同行评分，研究人员开发了一种算法，用于在最优组织中管理在线工作者（Lykourentzou等人，[2022](https://arxiv.org/html/2310.02170v2#bib.bib25)）。受到启发，我们提出了一种无监督算法，通过量化代理的贡献并基于同行评分来选择代理团队，详见[第3.4节](https://arxiv.org/html/2310.02170v2#S3.SS4 "3.4 Team Optimization ‣ 3 Dynamic LLM-Powered Agent Network ‣ A Dynamic LLM-Powered Agent Network for Task-Oriented Agent Collaboration")。

LLM驱动的代理协作中的通信结构 多个LLM代理之间的协作在近年来在各类任务上表现出强大的性能，已成为提升单个LLM能力的有前景的方法。为了实现多个代理之间的协作，近期的研究开发了不同的通信结构，并在预定义架构中分配代理。例如，研究人员发现，让多个LLM实例进行一定轮次的辩论可以提高它们的事实性和推理能力 (Du等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib10); Liang等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib19); Xiong等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib49))。为了聚合多个LLM的响应，LLM-Blender (Jiang等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib16))在一轮中调用不同的LLM，并使用配对排名来结合最佳响应。研究表明，它在分配工作负载给LLM并将它们的答案连接起来时也非常有效，从而产生更好的结果 (Ning等，[2024](https://arxiv.org/html/2310.02170v2#bib.bib29); Suzgun & Tauman Kalai，[2024](https://arxiv.org/html/2310.02170v2#bib.bib42); Qiao等，[2024](https://arxiv.org/html/2310.02170v2#bib.bib34))。值得注意的是，现有研究 (Hao等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib11); Zhang等，[2023b](https://arxiv.org/html/2310.02170v2#bib.bib55))尝试将LLM实例组织成线性层，但它们主要研究了上下文空间中的监督学习和LLM评估，而非我们感兴趣的场景。然而，在静态架构中运行LLM可能会限制其性能和泛化能力。在特定的推理任务中，采用动态有向无环图结构来运行LLM已被证明是有效的 (Zhang等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib56))。此外，近期的研究 (Yin等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib52); Chen等，[2024](https://arxiv.org/html/2310.02170v2#bib.bib7); Zhang等，[2023a](https://arxiv.org/html/2310.02170v2#bib.bib54); Zhuge等，[2024](https://arxiv.org/html/2310.02170v2#bib.bib61))表明，最佳的通信结构会根据任务和代理组合的不同而有所变化。与这些发现一致，我们提出了一种基于任务选择代理和代理团队构建的动态调整结构，具体内容见[第3.3.2节](https://arxiv.org/html/2310.02170v2#S3.SS3.SSS2 "3.3.2 代理团队重组 ‣ 3.3 任务解决 ‣ 3 动态LLM驱动的代理网络 ‣ 任务导向的代理协作的动态LLM驱动代理网络")。

LLM 驱动的代理贡献评估 在多代理系统中，评估每个 LLM 代理的贡献是一个复杂的问题，尤其是当它们通过多轮通信时。在单轮设置中，现有方法大量依赖 LLM 进行评估。为了克服 LLM 过度自信的问题（Xiong 等人，[2024](https://arxiv.org/html/2310.02170v2#bib.bib50)），LLM-Blender 中引入了基于附加 LLM 排名器的成对排名方法（Jiang 等人，[2023](https://arxiv.org/html/2310.02170v2#bib.bib16)）。为了在单轮中使用独立的 LLM 对 $n$ 个响应进行排名，它们会比较所有 $O(n^{2})$ 对。为了提高效率，研究人员使用 $k$ 长度的滑动窗口，在 $O(nk)$ 成对比较内选择前 $k$ 个响应（Qin 等人，[2023](https://arxiv.org/html/2310.02170v2#bib.bib35)）。然而，这些方法尚未扩展到多轮设置中。受到神经元重要性得分的启发（Yu 等人，[2018](https://arxiv.org/html/2310.02170v2#bib.bib53)），我们通过反向传播的方式传播和聚合单轮同行评分来评估代理（Rumelhart 等人，[1986](https://arxiv.org/html/2310.02170v2#bib.bib40)）。通过这种方式，我们引入了一种无监督的度量，称为代理重要性得分，用于量化每个代理在多轮协作中的贡献（[第 3.4 节](https://arxiv.org/html/2310.02170v2#S3.SS4 "3.4 团队优化 ‣ 3 动态 LLM 驱动的代理网络 ‣ 任务导向代理协作的动态 LLM 驱动代理网络")）。

| 方法 | 单次执行 | LLM-Blender | LLM 辩论 | 反思 | CAMEL | AgentVerse | DyLAN |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 通信结构 $(\mathcal{V};E)$ | ![[未标注图片]](img/71e22c85a20a20a7369ac8fb93aca82a.png)  | ![[未标注图片]](img/e8da8cf3c215172a0af89f5bd96a00cf.png)  | ![[未标注图片]](img/eec415c9890520078f70f3435c22f97a.png)  | ![[未标注图片]](img/d2f2e7aa6378e54e7f529e84147b6b12.png)  | ![[未标注图片]](img/fbc8a24e8c1c501ed4cdf0dc5cfbbe09.png)  | ![[未标注图片]](img/57bc7cf991ec47d2d963305dd9806b32.png)  | ![[未标注图片]](img/398f8ff1dc23c8a79a7abccb69c8331d.png)  |
| 多重角色 | $\times$ | $\times$ | $\times$ | 手动 | 手动 | 生成 | 手动与生成 |
| 提前停止 | $\times$ | $\times$ | $\times$ | ✓ | ✓ | ✓ | ✓ |
| 动态结构 | ✓ | $\times$ | $\times$ | $\times$ | $\times$ | $\times$ | ✓ |
| 团队优化 | $\times$ | $\times$ | $\times$ | $\times$ | $\times$ | $\times$ | ✓ |

表 1：DyLAN 与代表性先前工作的比较。在第二行中，节点表示不同时间步的代理（$\mathcal{V}$），箭头表示边（$E$），颜色表示代理的角色。

## 3 动态 LLM 驱动的代理网络

### 3.1 概述

我们介绍了一种基于LLM的代理协作框架，名为动态LLM驱动的代理网络（DyLAN），该框架通过两阶段方式促进动态通信结构和自动化的任务导向代理选择（[图1](https://arxiv.org/html/2310.02170v2#S0.F1 "在动态LLM驱动的代理网络中进行任务导向的代理协作")）：在第一阶段“团队优化”中，通过初步试验构建一个优化的代理团队，然后在第二阶段“任务求解”中，团队协作解决任务。

DyLAN的核心组件是时间前馈网络（T-FFNs），其节点表示代理，边表示代理之间的通信通道（[图2](https://arxiv.org/html/2310.02170v2#S3.F2 "在3.2时间前馈网络（T-FFNs）中 ‣ 3 动态LLM驱动的代理网络 ‣ 在动态LLM驱动的代理网络中进行任务导向的代理协作")左侧）。T-FFNs不仅作为通信结构的抽象，而且也作为计算图。从这个角度来看，如[表1](https://arxiv.org/html/2310.02170v2#S2.T1 "在2相关工作 ‣ 在动态LLM驱动的代理网络中进行任务导向的代理协作")所示，各种LLM驱动的代理协作系统（Jiang等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib16)；Shinn等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib41)；Du等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib10)；Li等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib18)；Chen等，[2024](https://arxiv.org/html/2310.02170v2#bib.bib7)）可以通过类似T-FFNs的网络结构来表示。DyLAN是唯一支持多代理角色和工具、早停（[第3.3.2节](https://arxiv.org/html/2310.02170v2#S3.SS3.SSS2 "3.3.2 代理团队重组 ‣ 3.3 任务求解 ‣ 3 动态LLM驱动的代理网络 ‣ 在动态LLM驱动的代理网络中进行任务导向的代理协作")）、动态通信结构和团队优化的框架。具体来说，对于团队优化，我们的代理选择算法是作为T-FFN上的反向消息传递算法来执行的（[图2](https://arxiv.org/html/2310.02170v2#S3.F2 "在3.2时间前馈网络（T-FFNs）中 ‣ 3 动态LLM驱动的代理网络 ‣ 在动态LLM驱动的代理网络中进行任务导向的代理协作")右侧），而对于任务求解，代理团队重组通过前向消息传递动态扩展T-FFN。

为了便于理解，我们将从解释T-FFNs的构成开始。接着，我们将进入任务求解阶段，最后，我们将解释依赖于任务求解组件的团队优化阶段。

### 3.2 时间前馈网络（T-FFNs）

![参见图注](img/54c4fbc7c6197649fb198ecdcda9fb27.png)

图 2：左侧展示了 DyLAN 如何在时间前馈网络（T-FFN）中输出答案，其中节点表示特定时间步的代理（[第3.2节](https://arxiv.org/html/2310.02170v2#S3.SS2 "3.2 Temporal Feed-Forward Networks (T-FFNs) ‣ 3 Dynamic LLM-Powered Agent Network ‣ A Dynamic LLM-Powered Agent Network for Task-Oriented Agent Collaboration")）。代理团队在中间步骤中进行重组，低效代理在随后的时间步中被停用。右侧展示了代理选择（[第3.4节](https://arxiv.org/html/2310.02170v2#S3.SS4 "3.4 Team Optimization ‣ 3 Dynamic LLM-Powered Agent Network ‣ A Dynamic LLM-Powered Agent Network for Task-Oriented Agent Collaboration")），在该步骤中，每个代理在初始试验中的贡献会通过三步自动评估，使用*代理重要性分数*，记作$\bm{I}$。然后，基于$\bm{I}$排序的顶级代理将被选为优化后的任务导向代理团队。

T-FFN 是一个多层网络，每一层代表一个时间步。其正式定义如下。

###### 定义 1 （代理）

参与协作的代理表示为

|  | $\mathcal{A}=\{a_{1},a_{2},\cdots,a_{N}\},$ |  | (1) |
| --- | --- | --- | --- |

其中 $N$ 表示代理的总数，$a_{i}$ 可以是 (I) 一个可能配备工具的 LLM 驱动的代理，或者 (II) 一个独立工具，例如脚本、代码解释器。

###### 定义 2 （节点）

T-FFN 的第 $t$ 层由 $N$ 个节点组成，每个节点对应一个代理：

|  | $\mathcal{V}_{t}=\{v_{t,1},\cdots,v_{t,N}\},$ |  | (2) |
| --- | --- | --- | --- |

其中 $t=1,\cdots,T$，节点 $v_{t,i}$ 对应于代理 $a_{i}$。

###### 定义 3 （边）

T-FFN 中的边表示节点之间的通信通道，形成代理之间的通信结构。具体地，表示层 $t-1$ 和 $t$ 之间节点的边集记作

|  | $E_{t-1,t}=\{(v_{t-1,i},v_{t,j})\}\subseteq\mathcal{V}_{t-1}\times\mathcal{V}_{% t},$ |  | (3) |
| --- | --- | --- | --- |

其中 $t=2,\cdots,T$，$(v_{t-1,i},v_{t,j})$ 表示连接节点 $v_{t-1,i}$ 和 $v_{t,j}$ 的边。

###### 定义 4 （T-FFN）

最终，协作所对应的 T-FFN 定义为一个 $T$ 层网络：

|  | $\mathcal{G}=(\mathcal{V}_{1},\cdots,\mathcal{V}_{T};E_{1,2},\cdots,E_{T-1,T}).$ |  | (4) |
| --- | --- | --- | --- |

请注意，我们只考虑那些仅在相邻层之间存在边的 T-FFN。然而，边也可以添加到任意节点对之间，使得 T-FFN 能够表示更复杂的通信结构。

### 3.3 任务求解

任务求解涉及在 T-FFN 上进行推理，并与代理团队重组共同完成，这将在接下来的两个章节中详细阐述。

#### 3.3.1 推理

在详细说明之前，我们首先介绍 T-FFN 上消息传递的公式。

###### 定义 5 （消息传递）

给定T-FFN，一个节点$v_{t,j}$，一组邻接节点$\mathcal{U}=\{u_{1},\cdots,u_{K}\}$，以及$v_{t,j}$接收到的消息$\mathcal{M}=\{m_{u_{1}},\cdots,m_{u_{K}}\}$，其中$K$是$\mathcal{U}$的大小，$m_{u_{k}}$是从$u_{k}$发送到$v_{t,j}$的消息，消息传递将所有消息$\mathcal{M}$聚合以生成一个更新后的消息$\hat{m}_{v_{t,j}}$，该过程形式上定义为

|  | $\hat{m}_{v_{t,j}}=f_{\mathrm{mp}}\left(\mathcal{M},v_{t,j}\right),$ |  | (5) |
| --- | --- | --- | --- |

其中$f_{\mathrm{mp}}(\cdot,\cdot)$是聚合函数。

###### 定义 6

当$\mathcal{U}$是来自上一时间步的$v_{t,j}$所有邻接节点的集合，即$\mathcal{U}=\{u_{k}|\forall u_{k},(u_{k},v_{t,j})\in E_{t-1,t}\}$时，我们称该算法为前向消息传递。同样，当$\mathcal{U}$是来自下一时间步的所有邻接节点的集合时，我们称之为后向消息传递：$\mathcal{U}=\{u_{k}|\forall u_{k},(v_{t,j},u_{k})\in E_{t,t+1}\}$。

通过上述公式，我们可以描述T-FFN $\mathcal{G}$的推理过程，采用前向消息传递的方式。在给定任务的协作过程中，特定时间步长的智能体将上一时间步的其他智能体的响应，即消息，作为输入，并基于任务查询生成响应。根据$v_{t,j}$处智能体的不同类型，我们可以分别实现$f_{\mathrm{mp}}(\cdot,v_{t,j})$： (I) 将输入消息与任务查询一起连接成提示模板（参见[附录 D](https://arxiv.org/html/2310.02170v2#A4 "附录 D 提示模板 ‣ 基于动态LLM的任务导向智能体协作网络")中的任务说明），并在生成或工具调用后获取LLM的响应，或者 (II) 过滤出工具可以处理的输入，例如代码补全和结构化文本。

在推理过程中，我们在时间步长1（$\mathcal{V}_{1}$）开始将任务查询$q\in\mathcal{Q}$输入到智能体中，其中$\mathcal{Q}$表示数据集。通过将时间步长$t-1$的节点$\mathcal{V}_{t-1}$的响应传递给时间步长$t$的节点$\mathcal{V}_{t}$，智能体可以感知来自上一时间步所有节点的响应，并执行协作行为，这可能包括批评、建议、改进或质量审查，具体取决于智能体的实现方式。形式上，推理过程定义为

|  | $f_{\mathrm{Infer}}(q,\mathcal{G})=o,$ |  | (6) |
| --- | --- | --- | --- |

其中$o=\mathrm{argmax}\{\mathcal{M}_{T}\}$，$\mathcal{M}_{T}$表示来自$\mathcal{V}_{T}$的响应。详细过程请参见[算法 1](https://arxiv.org/html/2310.02170v2#alg1 "在B.1详细实验设置 ‣ 附录B 实现细节 ‣ 基于动态LLM的任务导向智能体协作网络")。

#### 3.3.2 智能体团队重组

给定一个代理集合$\mathcal{A}$，代理团队重组旨在识别更多有贡献的代理，并相应地构建动态的通信结构。为此，我们利用一个额外的LLM实例，称为“LLM Ranker”，来分析当前时间步中参与的代理的响应，并根据[附录D](https://arxiv.org/html/2310.02170v2#A4 "附录D 提示模板 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")中的“Ranker”模板给出排名。然后，排名靠前的代理将被允许参与下一时间步。换句话说，只有这些排名靠前的代理才会被添加到边集合中，从而形成动态的通信结构。

正式地，假设在时间步$t$参与的代理集合为$\mathcal{A}_{t}=\{a_{k}\}$，而排名靠前的代理为$\mathcal{A}_{t+1}=\{a_{l}\}$，其中$k$和$l$是根据[公式1](https://arxiv.org/html/2310.02170v2#S3.E1 "在定义1（代理）‣ 3.2 时序前馈网络（T-FFNs）‣ 3 动态LLM驱动的代理网络‣ 用于任务导向代理协作的动态LLM驱动代理网络")中定义的代理索引，那么我们可以得到两个节点集合$\mathcal{V}_{t}^{\prime}=\{v_{t,k}|\forall a_{k}\in\mathcal{A}_{t}\}$和$\mathcal{V}_{t+1}^{\prime}=\{v_{t+1,l}|\forall a_{l}\in\mathcal{A}_{t+1}\}$，边集合$E_{t,t+1}$定义为

|  | $E_{t,t+1}=\mathcal{V}_{t}^{\prime}\times\mathcal{V}_{t+1}^{\prime}.$ |  | (7) |
| --- | --- | --- | --- |

该过程会迭代进行，直到满足停止条件，最终我们得到查询输入$q$的T-FFN$\mathcal{G}_{q}^{\mathcal{A}}$。我们用函数$f_{\mathrm{IAS}}$表示整个计算过程：

|  | $\mathcal{G}_{q}^{\mathcal{A}}=f_{\mathrm{IAS}}(\mathcal{A},q).$ |  | (8) |
| --- | --- | --- | --- |

为了进一步提高效率，我们引入了早停机制。受到拜占庭共识理论（Castro & Liskov, [1999](https://arxiv.org/html/2310.02170v2#bib.bib2)）的启发，至少需要$3p+1$个代理才能容忍单轮通信中的$p$个故障代理。根据该理论，当单层代理中超过2/3的代理给出一致答案时，推理过程将终止。实际上，当达到最大时间步时，推理过程也会终止。需要注意的是，先前研究中使用的任何一致性度量（Wang et al., [2023b](https://arxiv.org/html/2310.02170v2#bib.bib44); Aggarwal et al., [2023](https://arxiv.org/html/2310.02170v2#bib.bib1); Yin et al., [2023](https://arxiv.org/html/2310.02170v2#bib.bib52)）都不适用于多轮多代理交互，因为这些理论假定单个LLM实例执行多次，或者假设所有代理都能达到相同的答案。

### 3.4 团队优化

团队优化的目标是从候选代理中选择一个子集，基于它们在初步试验中的贡献评估，从而使得新的团队能更有效、更高效地解决任务查询。形式上，给定一个任务查询 $q$，一个代理集 $\mathcal{A}$，通过[第 3.3.2 节](https://arxiv.org/html/2310.02170v2#S3.SS3.SSS2 "3.3.2 代理团队重组 ‣ 3.3 任务解决 ‣ 3 动态 LLM 驱动的代理网络 ‣ 用于任务导向的代理协作的动态 LLM 驱动代理网络") 提出的算法进行试验，结果生成一个 T-FNN $\mathcal{G}_{q}^{\mathcal{A}}$。团队优化被公式化为：

|  | $\hat{\mathcal{A}}=f_{\mathrm{Optim}}(\mathcal{A},\mathcal{G}_{q}^{\mathcal{A}}% ,q),\text{ 其中 }\hat{\mathcal{A}}\subset\mathcal{A}.$ |  | (9) |
| --- | --- | --- | --- |

$f_{\mathrm{Optim}}$ 通过一个三步过程来实现，该过程用于代理选择（见[图 2](https://arxiv.org/html/2310.02170v2#S3.F2 "在 3.2 时间前馈网络 (T-FFNs) ‣ 3 动态 LLM 驱动的代理网络 ‣ 用于任务导向的代理协作的动态 LLM 驱动代理网络")）：

(1) 传播：每个节点根据来自其前驱节点的评分来评价任务查询的解决方案，这是一个前向的信息传递过程。形式上，给定一个节点 $v_{t,j}$ 和一条边 $(v_{t-1,i},v_{t,j})$，从 $v_{t-1,i}$ 发送到 $v_{t,j}$ 的消息 $m_{v_{t-1,i}}$ 被定义为来自代理 $a_{i}$ 在上一时间步对任务查询 $q$ 的响应。聚合器函数 $f_{\mathrm{mp}}(\cdot,v_{t,j})$ 被实现为一个评分函数 $f^{(s)}_{t,j}(\cdot,\cdot,\cdot)$，该函数将提示 $p_{j}$、输入查询 $q$ 和所有消息 $\mathcal{M}$ 映射到评分值。这里，我们用 $w_{t-1,i,j}$ 来表示 $v_{t-1,i}$ 从 $v_{t,j}$ 获得的评分。

|  | $[w_{t-1,1,j},w_{t-1,2,j},...,w_{t-1,N,j}]=f^{(s)}_{t,j}\left(p_{j},q,\mathcal{% M}\right).$ |  | (10) |
| --- | --- | --- | --- |

(2) 聚合：每个节点将从其后继节点接收到的评分聚合到自己身上，以便在不同的时间步长上独立量化其自身的贡献。节点 $v_{t-1,i}$ 的贡献是其后继节点贡献的总和，乘以它们的同行对代理响应的评分。聚合是一个向后的信息传递过程。形式上，给定一个节点 $v_{t-1,i}$ 和一条边 $(v_{t-1,i},v_{t,j})$，从 $v_{t,j}$ 发送到 $v_{t-1,i}$ 的消息 $m_{v_{t,j}}$ 被定义为 $w_{t-1,i,j}$。聚合器函数 $f_{\mathrm{mp}}$ 被定义为加权求和函数：

|  | $\bm{I}_{t-1,i}=\sum_{(v_{t-1,i},v_{t,j})\in E_{t-1,t}}\bm{I}_{t,j}\cdot w_{t-1% ,i,j},$ |  | (11) |
| --- | --- | --- | --- |

其中 $\bm{I}_{t,i}$ 表示 $a_{t,i}$ 的贡献。

(3) 选择：在最后一步，我们对同一代理在所有时间步长上的分数进行求和，得出每个代理的一个重要性分数，并根据这些分数提取出最具贡献的前 $k$ 个代理，以形成优化的代理团队。形式上，代理重要性分数 $\bm{I}_{i}$ 对于代理 $a_{i}$ 被定义为

|  | $\bm{I}_{i}=\sum_{t=1}^{T}\bm{I}_{t,i}.$ |  | (12) |
| --- | --- | --- | --- |

实践中，我们首先初始化最终层中的贡献，然后向后一步步执行聚合层（[算法 2](https://arxiv.org/html/2310.02170v2#alg2 "在 B.1 详细实验设置 ‣ 附录 B 实现细节 ‣ 用于任务导向代理协作的动态 LLM 驱动的代理网络")）。该定义保证了每一层中的代理重要性分数加起来为 1，从而有利于公平比较。其他细节，例如初始化最终层的贡献，已在[章节 B.2](https://arxiv.org/html/2310.02170v2#A2.SS2 "B.2 计算代理重要性分数 ‣ 附录 B 实现细节 ‣ 用于任务导向代理协作的动态 LLM 驱动的代理网络")中介绍。

| 方法 | Pass@1 | #API 调用次数 |
| --- | --- | --- |
| 单次执行 | 73.2 | (+0.0) | 1.00 |
| CodeT | 65.8 | (-7.4) | 20.00 |
| CodeT (Codex) | 74.8 | (+1.6) | 20.00 |
| Reflexion | 68.3 | (-4.9) | 4.05 |
| LATS | 81.1 | (+7.9) | 48.00 |
| CAMEL | 69.5 | (-4.1) | 12.03 |
| AgentVerse | 75.0 | (+1.8) | 22.50 |
| DyLAN (我们的) | 82.9 | (+9.7) | 16.85 |  | 方法 | 奖励 | 成功 | #API |
| 速率 | 调用次数 |
| 直接执行 | 50.6 | (+0.0) | 28.0 | 14.52 |
| ReAct | 53.8 | (+3.2) | 30.0 | 8.40 |
| ReAct-SC | 58.0 | (+7.4) | 36.0 | 25.75 |
| Reflexion (试验=4) | 62.0 | (+11.4) | 40.0 | 25.40 |
| LATS | 64.5 | (+13.9) | 38.0 | $>$ 400 |
| BOLAA | 66.0 | (+15.4) | 40.0 | 32.40 |
| DyLAN (我们的) | 68.3 | (+17.7) | 42.0 | 24.85 |

表 2：CG 任务的实验结果（左）和 DM 任务的结果（右）。括号中的数字表示相对于单次执行或直接执行的差异。除 GPT-35-turbo 外，我们指明了方法的基础模型。当使用非零温度时，报告三次试验的中位数。

| 方法 | 提示 | 代数 | 计数和 | 几何 | 中级 | 数字 | 预 | 预 | 总体 | #API |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 概率 | 代数 | 理论 | 代数 | 微积分 | 调用次数 |
| 单次执行 | CoT | 43.6 | 29.3 | 21.5 | 15.8 | 30.0 | 48.9 | 16.5 | 31.6 (+0.0) | 1.00 |
| LLM-Blender | 47.5 | 25.5 | 23.8 | 13.8 | 39.7 | 46.7 | 15.8 | 31.7 (+0.1) | 6.00 |
| LLM 辩论 | 50.2 | 25.3 | 22.3 | 13.1 | 28.9 | 48.0 | 19.0 | 32.4 (+0.8) | 8.00 |
| DyLAN (我们的) | 52.9 | 27.2 | 25.3 | 15.5 | 33.5 | 55.2 | 19.0 | 35.7 (+4.1) | 7.15 |
| 单次执行 | 复杂 CoT | 49.1 | 29.7 | 22.3 | 14.6 | 33.4 | 53.8 | 16.8 | 34.1 (+0.0) | 1.00 |
| PHP | 51.1 | 33.7 | 25.4 | 17.1 | 35.1 | 57.7 | 16.1 | 36.5 (+2.4) | 3.67 |
| DyLAN（我们的） | 53.7 | 33.3 | 26.1 | 18.1 | 33.5 | 58.7 | 18.9 | 37.6 (+3.5) | 6.21 |

表 3：AR 任务的准确率（%）。括号中的数字表示相对于单一执行的性能差异。

## 4 实验

### 4.1 设置

代码生成（CG）我们使用 HumanEval 基准，包含 164 个人工标注的函数级别完成代码和单元测试（Chen 等，[2021](https://arxiv.org/html/2310.02170v2#bib.bib6)）。单元测试用于验证生成代码的正确性。我们使用两个强基准 CodeT（Chen 等，[2023a](https://arxiv.org/html/2310.02170v2#bib.bib4)）和 Reflexion（Shinn 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib41)）以及单一执行作为基准。对于多代理基准，我们重新实现了 CAMEL（Li 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib18)）和 AgentVerse（Chen 等，[2024](https://arxiv.org/html/2310.02170v2#bib.bib7)），并在其原始配置下进行公平比较。

决策制定（DM）我们在 WebShop 环境中评估我们的方法，选择其测试集中的 50 个环境（Chen 等，[2021](https://arxiv.org/html/2310.02170v2#bib.bib6)），并遵循 LATS 设置（Zhou 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib58)）。WebShop 要求根据客户的指令找到物品。它提供“奖励”作为物品与指令相关性的内在指标，当奖励为 1.0 时，标记为“成功”。除了 ReAct（Yao 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib51)）和 Reflexion，我们还重新运行了一个多代理方法 BOLAA（Liu 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib22)），以及一个单代理方法 LATS 作为强基准。

一般推理（GR）对于一般推理任务，我们使用 MMLU 数据集（Hendrycks 等，[2021a](https://arxiv.org/html/2310.02170v2#bib.bib12)），该数据集包含 57 个学科的四个类别的大量问题。由于测试集中的问题数量庞大，我们将其中的 1/5 问题进行下采样。我们选择 LLM Debate（Du 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib10)）、LLM-Blender（Jiang 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib16)）和单一执行的 LLM 作为基准。

算术推理（AR）我们利用 MATH 数据集（Hendrycks 等，[2021b](https://arxiv.org/html/2310.02170v2#bib.bib13)）的测试集进行评估，该数据集包括 7 个子领域，总共 5,000 个问题。为了进行公平比较并验证鲁棒性，我们根据不同的提示策略对方法进行分类，并相应地选择强基准。初步实验表明，在不同领域（例如代数和几何专家）之间协作的代理并没有显著提高性能，因此我们为所有方法采用相同提示的代理。

DyLAN 设置在[表 5](https://arxiv.org/html/2310.02170v2#S4.T5 "在 4.1 设置 ‣ 4 实验 ‣ 用于任务导向代理协作的动态 LLM 驱动代理网络")中，我们详细阐述了 DyLAN 的设置。为了与基线方法保持一致，我们仅为 DyLAN 配备了在 CG 任务中使用的代码解释器作为工具。值得注意的是，GR 任务中对每个主题执行代理选择，DM 任务中对每个网页执行代理选择，而在团队优化阶段，CG 任务则直接进行代理选择。由于篇幅限制，请参阅[Section B.1](https://arxiv.org/html/2310.02170v2#A2.SS1 "B.1 详细实验设置 ‣ 附录 B 实现细节 ‣ 用于任务导向代理协作的动态 LLM 驱动代理网络")以获取详细信息。

| 方法 | 人文 | 社会 | STEM | 其他 | 综合 | #API |
| --- | --- | --- | --- | --- | --- | --- |
| 人文学科 | 科学 | 呼叫 |
| 随机 | 25.0 | 25.0 | 25.0 | 25.0 | 25.0 | - |
| 单次执行 | 59.8 | 74.0 | 62.9 | 71.8 | 66.4 (+0.0) | 1.00 |
| LLM-Blender | 60.4 | 75.2 | 66.3 | 70.7 | 67.3 (+0.9) | 6.00 |
| LLM 辩论 | 59.8 | 77.4 | 69.0 | 75.5 | 69.3 (+2.9) | 12.00 |
| DyLAN | 62.1 | 79.1 | 69.7 | 75.5 | 70.5 (+4.1) | 4.39 |

表 4：GR 任务的准确度（%）。其中“其他”代表 MMLU 数据集中的商业、健康和其他领域的主题。我们报告了三次实验的中位数。

| 任务 | #代理 | 工具 | 性能 | #API 调用 |
| --- | --- | --- | --- | --- |
| 用法 | 改进 |
| CG | $12\rightarrow 8$ | ✓ | 76.2 $\rightarrow$ 82.9 | 23.04 $\rightarrow$ 16.85 |
| DM | $8\rightarrow 4$ | $\times$ | 53.0 $\rightarrow$ 68.3 | 32.03 $\rightarrow$ 24.85 |
| GR | $7\rightarrow 4$ | $\times$ | 69.5 $\rightarrow$ 70.5 | 08.30 $\rightarrow$ 04.39 |
| AR | $4$ | $\times$ | - | - |

表 5：实验设置的演示，包括代理的数量和团队优化过程中的表现。我们报告了 DM 任务的奖励。

### 4.2 主要结果

在[表格2](https://arxiv.org/html/2310.02170v2#S3.T2 "在3.4团队优化 ‣ 3 动态LLM驱动的代理网络 ‣ 用于任务导向的代理协作的动态LLM驱动代理网络")、[表格3](https://arxiv.org/html/2310.02170v2#S3.T3 "在3.4团队优化 ‣ 3 动态LLM驱动的代理网络 ‣ 用于任务导向的代理协作的动态LLM驱动代理网络")和[表格5](https://arxiv.org/html/2310.02170v2#S4.T5 "在4.1设置 ‣ 4 实验 ‣ 用于任务导向的代理协作的动态LLM驱动代理网络")中，我们分别报告了每个数据集的结果。API调用次数作为代理通信结构效率的代理指标，无法从令牌消耗中明确判断，因为令牌消耗会因任务和提示策略的不同而有很大差异。由于“团队优化”后的“任务解决”对于测试和部署至关重要，我们主要报告第二阶段的成本。差异可以在[表格5](https://arxiv.org/html/2310.02170v2#S4.T5 "在4.1设置 ‣ 4 实验 ‣ 用于任务导向的代理协作的动态LLM驱动代理网络")中看到，更多讨论请参见[章节C.1](https://arxiv.org/html/2310.02170v2#A3.SS1 "C.1团队优化的数据效率 ‣ 附录C 其他结果 ‣ 用于任务导向的代理协作的动态LLM驱动代理网络")。

DyLAN通过合理的计算成本提高了不同任务的整体表现。从[表3](https://arxiv.org/html/2310.02170v2#S3.T3 "在3.4团队优化 ‣ 3 动态LLM驱动的代理网络 ‣ 面向任务的代理协作的动态LLM驱动代理网络")中，我们发现DyLAN在LLM辩论任务中的准确度提高了10.2%，并且API调用次数减少了10.6%（L3与L4对比），这表明它在效率和效果之间达到了更好的平衡。类似的趋势也可以观察到，DyLAN在API调用次数仅为LLM辩论的36.6%的情况下表现更好（L5与L4在[表5](https://arxiv.org/html/2310.02170v2#S4.T5 "在4.1设置 ‣ 4 实验 ‣ 面向任务的代理协作的动态LLM驱动代理网络")中对比），同时在CG任务中的LATS为35.1%，在DM任务中的误差小于6%（L8与L5（左），L7与L5（右）在[表2](https://arxiv.org/html/2310.02170v2#S3.T2 "在3.4团队优化 ‣ 3 动态LLM驱动的代理网络 ‣ 面向任务的代理协作的动态LLM驱动代理网络")中对比）。我们认为这归因于前馈结构和早停机制，这使得不同的解决方案能够同时输出并快速确认。相比之下，像PHP和Reflexion等顺序架构中的方法（[表2](https://arxiv.org/html/2310.02170v2#S3.T2 "在3.4团队优化 ‣ 3 动态LLM驱动的代理网络 ‣ 面向任务的代理协作的动态LLM驱动代理网络")）由于推理或代码生成与审查的单线程性质，错误的中间结果可能会轻易影响最终输出。此外，ReAct在DM任务上也表现出类似的失败，因为中间操作错误。在对比中，Reflexion和LATS在特定状态下多次访问环境进行反思，这限制了其一般性。而在我们的方法中，任何来自前置节点的反馈都可以被后续节点评估，从而更容易纠正潜在的无效操作。此外，我们看到DyLAN会根据任务的难度动态调整成本。例如，MMLU数据集中的大多数问题比MATH数据集中的问题挑战性小，因此DyLAN在前者的查询中减少了2.76次API调用。然而，与其他任务相比，DyLAN在AR任务中的改进相对较小，这可能是因为MATH数据集对知识的依赖较高。

DyLAN受益于团队优化。此外，我们发现，基于任务导向的动态代理团队能够增强DyLAN。对于GR任务中的不同主题，代理组成会相应调整，从而使准确率提高最多25.0%，如[表8](https://arxiv.org/html/2310.02170v2#S4.T8 "在 4.3 消融研究 ‣ 4 实验 ‣ 面向任务的代理协作中的动态LLM驱动代理网络")所示。如[表5](https://arxiv.org/html/2310.02170v2#S4.T5 "在 4.1 设置 ‣ 4 实验 ‣ 面向任务的代理协作中的动态LLM驱动代理网络")所示，动态选择的代理团队能够显著提升性能，尤其是在DM任务中，代理之间可能存在较大干扰。在每个任务的代理选择后，整体性能可以显著提高（最高可达6.7%），同时计算成本降低。此外，这也表明，代理重要性分数可以有效地捕捉和反映代理在各种任务中的实际贡献。我们在[第C.6节](https://arxiv.org/html/2310.02170v2#A3.SS6 "C.6 代理重要性分数是否捕捉到实际贡献？ ‣ 附录C 额外结果 ‣ 面向任务的代理协作中的动态LLM驱动代理网络")进一步验证了这一说法。

![参见说明](img/5cc2f43d1f9b70cae14121d771940e2d.png)![参见说明](img/fcb646c3fe46244dab06cfed17fcf7dd.png)

图 3：优化代理团队规模的影响。从7个候选代理中根据代理重要性分数选择2至4个代理。GR任务的准确率（左）和#API调用（右）可视化展示。

| 方法 | AR | GR |
| --- | --- | --- |
| 准确率 | #API | 准确率 | #API |
| DyLAN | 35.7 | 7.15 | 70.5 | 4.39 |
| 无es | 35.0 | 13.00 | 70.1 | 13.00 |
| 无atr | 33.8 | 8.20 | 69.9 | 7.05 |
| 方法 | CG | DM |
| Pass@1 | #API | 奖励 | #API |
| DyLAN | 82.9 | 16.85 | 68.3 | 24.85 |
| 无es | 80.5 | 19.00 | 67.5 | 54.25 |
| 无atr | 76.2 | 17.98 | 66.0 | 48.90 |

表 6：早停机制（es）和代理团队重组（atr）的影响。

### 4.3 消融研究

优化代理团队规模的影响：团队中较少的合适代理可能会有更好的表现。如[图3](https://arxiv.org/html/2310.02170v2#S4.F3 "在 4.2 主要结果 ‣ 4 实验 ‣ 面向任务的代理协作中的动态LLM驱动代理网络")所示，DyLAN在优化的3个代理团队下，能够超越团队优化前的DyLAN和4个代理的LLM Debate，表明我们提出的代理选择方法的有效性。效率也分别提高了52.9%和67.8%。这可能是因为在优化前，代理的专业领域和意见的不平衡相互干扰，尤其是在GR任务中，很少有候选者与某个主题相关。

| 主题 | 优化组成 | 性能提升 |
| --- | --- | --- |
| 大学 | 经济学家，律师， | $\mathbf{25.0}:40.0\rightarrow 65.0$ |
| 数学 | 程序员，数学家 |
| 管理 | 律师，心理学家， | $\mathbf{14.3}:76.2\rightarrow 90.5$ |
| 经济学家，程序员 |
| 高中 | 历史学家，程序员， | $\mathbf{9.3}:65.1\rightarrow 74.4$ |
| 统计学 | 心理学家，数学家 |
| 临床 | 医生，数学家， | $\mathbf{5.7}:69.8\rightarrow 75.5$ |
| 知识 | 程序员，心理学家 |
| 公共 | 历史学家，心理学家， | $\mathbf{4.5}:54.5\rightarrow 59.1$ |
| 关系 | 律师，数学家 |

表 7：GR 任务中不同学科的优化代理组成和性能提升。

| #代码 | #代码 | Pass@1 | #API |
| --- | --- | --- | --- |
| 作家 | 审稿人 | 呼叫 |
| 6 | 6 | 76.2 | 23.04 |
| 4 | 4 | 82.9 | 16.85 |
| 4 | 3 | 81.1 | 14.64 |
| 4 | 2 | 77.4 | 12.50 |
| 3 | 3 | 78.0 | 11.73 |
| 3 | 2 | 75.6 | 09.60 |

表 8：优化代理团队在CG任务中的不同代理组成。代理团队通过第一行的代理重要性评分进行优化。

代理重要性评分的鲁棒性 代理重要性评分在代理角色不平衡的情况下仍具有鲁棒性。在GR任务中，候选人们的专业领域存在不平衡。对于大多数查询，根据角色提示，相关的候选人少于 2 个。在[表 8](https://arxiv.org/html/2310.02170v2#S4.T8 "在 4.3 剖析研究 ‣ 4 实验 ‣ 一个面向任务的动态 LLM 驱动代理网络")中，我们发现代理选择能够选择相关的代理，例如，“数学家”对于“大学数学”这一任务，符合人类的先验知识。然而，如果候选人们与任务领域相差甚远，例如“公共关系”，则提升效果较为有限。我们还在“团队优化”阶段后，测试了不同数量的代码编写者和审稿者的CG任务表现。值得注意的是，单次的“团队优化”可以为多次的代理选择提供可重复使用的代理重要性评分。在[表 8](https://arxiv.org/html/2310.02170v2#S4.T8 "在 4.3 剖析研究 ‣ 4 实验 ‣ 一个面向任务的动态 LLM 驱动代理网络")中，我们展示了在代理团队不平衡的情况下的结果。我们验证了代码编写者和审稿者的优化后不平衡不会导致明显的性能下降。尽管如此，审稿者对性能的影响略大于编写者（L2,3 vs. L4,5），表明审稿者数量对于代码优化的重要性。

[早停机制与代理团队重组的影响](https://arxiv.org/html/2310.02170v2#S4.T6 "在4.2主要结果 ‣ 4实验 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")，如[表6](https://arxiv.org/html/2310.02170v2#S4.T6 "在4.2主要结果 ‣ 4实验 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")所示，早停机制通过减少#API调用量，分别在AR、GR、CG和DM任务上减少了45.0%、66.2%、11.3%和54.2%，从而大大提高了效率，同时提供了轻微的性能提升。然而，代理团队重组对于提高最终答案的正确性至关重要。我们推测，这是因为代理在LLMs中被过滤掉了临时错误，如幻觉等。此外，对于像CG或DM任务这样的开放性任务，答案比较更具挑战性。我们使用BLEU分数，并设置0.9的阈值进行一致性检查。由于代理可能生成不同格式的代码，导致早停效果减弱，从而减少了早停的机会。

DyLAN在不同骨干模型下的稳定性 在CG任务中，当骨干模型发生变化时，表现也有显著差异（[表2](https://arxiv.org/html/2310.02170v2#S3.T2 "在3.4团队优化 ‣ 3动态LLM驱动代理网络 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")）。Reflexion和CodeT的表现与骨干模型密切相关（L4与L5，以及L6与L9）。而DyLAN在不同骨干模型下表现稳定、一致，几乎使用相同的API调用量（L7与L10）。

## 5 结论与未来工作

本文介绍了一个名为动态LLM驱动代理网络（DyLAN）的框架，用于动态代理团队在复杂任务中的协作。DyLAN采用二阶段范式，能够使代理在动态结构中进行交互，并进行代理团队重组。在“团队优化”阶段，基于无监督度量的代理重要性分数算法，以原则性的方式选择贡献最大的代理进行“任务解决”协作。总体而言，DyLAN在多种任务上表现出相较基准方法较低的计算成本的改进。未来，我们计划探索基于开源基础模型构建的DyLAN的有效性。

#### 致谢

我们感谢Yilun Du在代码实现上的帮助。我们衷心感谢William Held、Ruibo Liu、Dr. Yue Zhuge、Noah Shinn和Kangfu Zheng对项目的宝贵反馈。

## 伦理声明

LLM驱动的代理系统在实际应用中得到了广泛使用。DyLAN也可以轻松涵盖实际的软件开发、虚拟聊天室、视频游戏等（Hong等人，[2024](https://arxiv.org/html/2310.02170v2#bib.bib15)；Nascimento等人，[2023](https://arxiv.org/html/2310.02170v2#bib.bib28)；Zhou等人，[2023](https://arxiv.org/html/2310.02170v2#bib.bib59)；Zhu等人，[2023](https://arxiv.org/html/2310.02170v2#bib.bib60)；Chan等人，[2024](https://arxiv.org/html/2310.02170v2#bib.bib3)）。在这些开放世界环境中，代理可以作为规划者、执行者等角色操作。DyLAN只需人们给出关于代理组成的粗略指令，就能够自动优化出一个更好的代理团队来构建高效的多代理系统。这些系统能够受益于DyLAN，从而减少在设计代理方面的人工劳动，并在其目标任务上表现出更好的性能。

此外，DyLAN的整体架构（[图2](https://arxiv.org/html/2310.02170v2#S3.F2 "在3.2节 时序前馈网络（T-FFNs） ‣ 3 动态LLM驱动的代理网络 ‣ 基于动态LLM驱动的代理网络进行任务导向的代理协作")）反映了人类在线工作者的最佳协作组织方式（Lykourentzou等人，[2022](https://arxiv.org/html/2310.02170v2#bib.bib25)），并在代理协作中表现出了显著的性能。因此，通过DyLAN下的LLM驱动的代理协作模拟人类协作也是可能的。通过搜索和模拟LLM代理来优化人类协作，预计将更加方便和高效。我们也承认本文中使用的预训练语言模型（例如，GPT-3.5和GPT-4）可能会导致不恰当的响应。此外，手动创建代理和LLM生成的代理可能会导致LLM驱动的代理的行为或响应与社会原则不一致，这在代理协作过程中可能会发生。然而，我们认为团队优化过程可能有助于缓解这种情况。

## 参考文献

+   Aggarwal等人（2023）Pranjal Aggarwal、Aman Madaan、Yiming Yang和Mausam。让我们一步步采样：用于高效推理和编码的LLM自适应一致性。在Houda Bouamor、Juan Pino和Kalika Bali（编辑），*2023年自然语言处理经验方法会议论文集*，第12375–12396页，新加坡，2023年12月。计算语言学协会。doi: 10.18653/v1/2023.emnlp-main.761。URL [https://aclanthology.org/2023.emnlp-main.761](https://aclanthology.org/2023.emnlp-main.761)。

+   Castro & Liskov（1999）Miguel Castro 和 Barbara Liskov。实用拜占庭容错。在*第三届操作系统设计与实现研讨会论文集*，OSDI ’99，第173–186页，美国，1999年。USENIX协会。ISBN 1880446391。

+   Chan等人（2024）Chi-Min Chan、Weize Chen、Yusheng Su、Jianxuan Yu、Wei Xue、Shanghang Zhang、Jie Fu和Zhiyuan Liu。《Chateval：通过多智能体辩论实现更好的基于LLM的评估器》。发表于*第十二届国际学习表征会议*，2024年。网址：[https://openreview.net/forum?id=FQepisCUWu](https://openreview.net/forum?id=FQepisCUWu)。

+   Chen等人（2023a）Bei Chen、Fengji Zhang、Anh Nguyen、Daoguang Zan、Zeqi Lin、Jian-Guang Lou和Weizhu Chen。《Codet：带有生成测试的代码生成》。发表于*第十一届国际学习表征会议论文集*，2023年。网址：[https://openreview.net/forum?id=ktrw68Cmu9c](https://openreview.net/forum?id=ktrw68Cmu9c)。

+   Chen等人（2023b）Guangyao Chen、Siwei Dong、Yu Shu、Ge Zhang、Sesay Jaward、Karlsson Börje、Jie Fu和Yemin Shi。《Autoagents：自动化智能体生成框架》。*arXiv预印本arXiv:2309.17288*，2023年。

+   Chen等人（2021）Mark Chen、Jerry Tworek、Heewoo Jun、Qiming Yuan、Henrique Ponde de Oliveira Pinto、Jared Kaplan、Harri Edwards、Yuri Burda、Nicholas Joseph、Greg Brockman、Alex Ray、Raul Puri、Gretchen Krueger、Michael Petrov、Heidy Khlaaf、Girish Sastry、Pamela Mishkin、Brooke Chan、Scott Gray、Nick Ryder、Mikhail Pavlov、Alethea Power、Lukasz Kaiser、Mohammad Bavarian、Clemens Winter、Philippe Tillet、Felipe Petroski Such、Dave Cummings、Matthias Plappert、Fotios Chantzis、Elizabeth Barnes、Ariel Herbert-Voss、William Hebgen Guss、Alex Nichol、Alex Paino、Nikolas Tezak、Jie Tang、Igor Babuschkin、Suchir Balaji、Shantanu Jain、William Saunders、Christopher Hesse、Andrew N. Carr、Jan Leike、Josh Achiam、Vedant Misra、Evan Morikawa、Alec Radford、Matthew Knight、Miles Brundage、Mira Murati、Katie Mayer、Peter Welinder、Bob McGrew、Dario Amodei、Sam McCandlish、Ilya Sutskever和Wojciech Zaremba。《评估在代码上训练的大型语言模型》。*arXiv预印本arXiv:2107.03374*，2021年。

+   Chen等人（2024）Weize Chen、Yusheng Su、Jingwei Zuo、Cheng Yang、Chenfei Yuan、Chi-Min Chan、Heyang Yu、Yaxi Lu、Yi-Hsin Hung、Chen Qian、Yujia Qin、Xin Cong、Ruobing Xie、Zhiyuan Liu、Maosong Sun和Jie Zhou。《Agentverse：促进多智能体协作并探索涌现行为》。发表于*第十二届国际学习表征会议*，2024年。网址：[https://openreview.net/forum?id=EHg5GDnyq1](https://openreview.net/forum?id=EHg5GDnyq1)。

+   Christianos等人（2023）Filippos Christianos、Georgios Papoudakis、Matthieu Zimmer、Thomas Coste、Zhihao Wu、Jingxuan Chen、Khyati Khandelwal、James Doran、Xidong Feng、Jiacheng Liu、Zheng Xiong、Yicheng Luo、Jianye Hao、Kun Shao、Haitham Bou-Ammar和Jun Wang。《Pangu-Agent：一种具有结构化推理的精细可调通用智能体》。*arXiv预印本arXiv:2312.14878*，2023年。

+   Dong等人（2023）Yihong Dong、Xue Jiang、Zhi Jin和Ge Li。《通过ChatGPT实现自协作代码生成》。*arXiv预印本arXiv:2304.07590*，2023年。

+   Du等人（2023）Yilun Du, Shuang Li, Antonio Torralba, Joshua B. Tenenbaum, 和 Igor Mordatch。通过多代理辩论提高语言模型的事实性和推理能力。*arXiv预印本 arXiv:2305.14325*，2023年。

+   Hao等人（2023）Rui Hao, Linmei Hu, Weijian Qi, Qingliu Wu, Yirui Zhang, 和 Liqiang Nie。Chatllm网络：更多的大脑，更多的智慧。*arXiv预印本 arXiv:2304.12998*，2023年。

+   Hendrycks等人（2021a）Dan Hendrycks, Collin Burns, Steven Basart, Andy Zou, Mantas Mazeika, Dawn Song, 和 Jacob Steinhardt。衡量大规模多任务语言理解。在*国际学习表征会议论文集（ICLR）*，2021年。

+   Hendrycks等人（2021b）Dan Hendrycks, Collin Burns, Saurav Kadavath, Akul Arora, Steven Basart, Eric Tang, Dawn Song, 和 Jacob Steinhardt。使用数学数据集衡量数学问题解决能力。在*第三十五届神经信息处理系统会议论文集*，2021年。

+   Herbrich等人（2006）Ralf Herbrich, Tom Minka, 和 Thore Graepel。Trueskill™：一个贝叶斯技能评分系统。在B. Schölkopf, J. Platt, 和 T. Hoffman（编），*神经信息处理系统进展论文集*，第19卷。MIT出版社，2006年。

+   Hong等人（2024）Sirui Hong, Mingchen Zhuge, Jonathan Chen, Xiawu Zheng, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, Chenyu Ran, Lingfeng Xiao, Chenglin Wu, 和 Jürgen Schmidhuber。MetaGPT：用于多代理协作框架的元编程。在*第十二届国际学习表征会议*，2024年。网址 [https://openreview.net/forum?id=VtmBAGCN7o](https://openreview.net/forum?id=VtmBAGCN7o)。

+   Jiang等人（2023）Dongfu Jiang, Xiang Ren, 和 Bill Yuchen Lin。LLM-blender：通过成对排序和生成性融合集成大语言模型。在*第61届计算语言学协会年会（第一卷：长篇论文集）*，第14165-14178页，多伦多，加拿大，2023年7月。计算语言学协会。网址 [https://aclanthology.org/2023.acl-long.792](https://aclanthology.org/2023.acl-long.792)。

+   Jin等人（2023）Ye Jin, Xiaoxi Shen, Huiling Peng, Xiaoan Liu, Jingli Qin, Jiayang Li, Jintao Xie, Peizhong Gao, Guyue Zhou, 和 Jiangtao Gong。Surrealdriver：基于大语言模型在城市环境中的生成性驾驶代理模拟框架设计。*arXiv预印本 arXiv:2309.13193*，2023年。

+   Li等人（2023）Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, 和 Bernard Ghanem。CAMEL：用于“大脑”探索的通信代理，探索大语言模型社会。在*第三十七届神经信息处理系统会议论文集*，2023年。网址 [https://openreview.net/forum?id=3IyL2XWDkG](https://openreview.net/forum?id=3IyL2XWDkG)。

+   Liang 等（2023）Tian Liang、Zhiwei He、Wenxiang Jiao、Xing Wang、Yan Wang、Rui Wang、Yujiu Yang、Zhaopeng Tu 和 Shuming Shi。通过多智能体辩论激发大语言模型中的发散思维。*arXiv 预印本 arXiv:2305.19118*，2023年。

+   Liu 等（2015）Qing Liu、Tie Luo、Ruiming Tang 和 Stéphane Bressan。一种高效且真实的众包市场团队组建定价机制。在*2015年IEEE国际通信会议（ICC）*，第567-572页，2015年。

+   Liu 等（2024）Xiao Liu、Hao Yu、Hanchen Zhang、Yifan Xu、Xuanyu Lei、Hanyu Lai、Yu Gu、Hangliang Ding、Kaiwen Men、Kejuan Yang、Shudan Zhang、Xiang Deng、Aohan Zeng、Zhengxiao Du、Chenhui Zhang、Sheng Shen、Tianjun Zhang、Yu Su、Huan Sun、Minlie Huang、Yuxiao Dong 和 Jie Tang。Agentbench：评估 LLM 作为代理的能力。在*第十二届国际学习表示大会*，2024年。网址 [https://openreview.net/forum?id=zAdUB0aCTQ](https://openreview.net/forum?id=zAdUB0aCTQ)。

+   Liu 等（2023）Zhiwei Liu、Weiran Yao、Jianguo Zhang、Le Xue、Shelby Heinecke、Rithesh Murthy、Yihao Feng、Zeyuan Chen、Juan Carlos Niebles、Devansh Arpit、Ran Xu、Phil Mui、Huan Wang、Caiming Xiong 和 Silvio Savarese。Bolaa：基准测试和协调 LLM 增强的自主代理。*arXiv 预印本 arXiv:2308.05960*，2023年。

+   Lu 等（2023）Pan Lu、Baolin Peng、Hao Cheng、Michel Galley、Kai-Wei Chang、Ying Nian Wu、Song-Chun Zhu 和 Jianfeng Gao。Chameleon：与大语言模型的即插即用组合推理。在*第三十七届神经信息处理系统会议*论文集中，2023年。网址 [https://openreview.net/forum?id=HtqnVSCj3q](https://openreview.net/forum?id=HtqnVSCj3q)。

+   Lundberg & Lee（2017）Scott M. Lundberg 和 Su-In Lee。统一的方法解释模型预测。在*第31届国际神经信息处理系统大会*，NIPS’17，第4768-4777页，Red Hook，NY，美国，2017年。Curran Associates Inc. ISBN 9781510860964。

+   Lykourentzou 等（2022）Ioanna Lykourentzou、Federica Lucia Vinella、Faez Ahmed、Costas Papastathis、Konstantinos Papangelis、Vassilis-Javed Khan 和 Judith Masthoff。在线协作工作环境中的自组织。*集体智能*，1(1)，2022年9月。

+   Ma 等（2023）Kaixin Ma、Hongming Zhang、Hongwei Wang、Xiaoman Pan 和 Dong Yu。Laser：具有状态空间探索的 LLM 代理用于网页导航。*arXiv 预印本 arXiv:2309.08172*，2023年。

+   Nakajima（2023）Yohei Nakajima。Babyagi。 [https://github.com/yoheinakajima/babyagi](https://github.com/yoheinakajima/babyagi)，2023年。

+   Nascimento 等（2023）Nathalia Nascimento、Paulo Alencar 和 Donald Cowan。GPT-in-the-Loop：用于多智能体系统的自适应决策。*arXiv 预印本 arXiv:2308.10435*，2023年。

+   Ning 等人（2024）Xuefei Ning、Zinan Lin、Zixuan Zhou、Zifu Wang、Huazhong Yang 和 Yu Wang。思维骨架：大语言模型可以进行并行解码。发表于 *第十二届国际学习表征会议*，2024。网址 [https://openreview.net/forum?id=mqVgBbNCm9](https://openreview.net/forum?id=mqVgBbNCm9)。

+   OpenAI（2023）OpenAI。GPT-4 技术报告。*arXiv 预印本 arxiv:2303.08774*，2023。

+   Post（2018）Matt Post。呼吁在报告 BLEU 分数时保持清晰。发表于 *第三届机器翻译会议：研究论文集*，第186–191页，比利时，布鲁塞尔，2018年10月。计算语言学协会。网址 [https://www.aclweb.org/anthology/W18-6319](https://www.aclweb.org/anthology/W18-6319)。

+   Qian 等人（2023a）Chen Qian、Xin Cong、Wei Liu、Cheng Yang、Weize Chen、Yusheng Su、Yufan Dang、Jiahao Li、Juyuan Xu、Dahai Li、Zhiyuan Liu 和 Maosong Sun。面向软件开发的交流智能体。*arXiv 预印本 arXiv:2307.07924*，2023a。

+   Qian 等人（2023b）Chen Qian、Yufan Dang、Jiahao Li、Wei Liu、Weize Chen、Cheng Yang、Zhiyuan Liu 和 Maosong Sun。软件开发智能体的体验性共学习。*arXiv 预印本 arXiv:2312.17025*，2023b。

+   Qiao 等人（2024）Shuofei Qiao、Ningyu Zhang、Runnan Fang、Yujie Luo、Wangchunshu Zhou、Yuchen Eleanor Jiang、Chengfei Lv 和 Huajun Chen。AUTOACT：通过自我规划自动学习智能体。*arXiv 预印本 arXiv:2401.05268*，2024。

+   Qin 等人（2023）Zhen Qin、Rolf Jagerman、Kai Hui、Honglei Zhuang、Junru Wu、Jiaming Shen、Tianqi Liu、Jialu Liu、Donald Metzler、Xuanhui Wang 和 Michael Bendersky。大语言模型是有效的文本排序器，通过成对排序提示。*arXiv 预印本 arXiv:2306.17563*，2023。

+   Ren 等人（2020）Shuo Ren、Daya Guo、Shuai Lu、Long Zhou、Shujie Liu、Duyu Tang、M. Zhou、Ambrosio Blanco 和 Shuai Ma。Codebleu：一种用于自动评估代码合成的方法。*arXiv 预印本 arxiv:2009.10297*，2020。

+   Reworkd（2023）Reworkd。Agentgpt。 [https://github.com/reworkd/AgentGPT](https://github.com/reworkd/AgentGPT)，2023。

+   Richards 等人（2023）Toran Bruce Richards 等人。Auto-gpt：一个自主的 GPT-4 实验。 [https://github.com/Significant-Gravitas/Auto-GPT](https://github.com/Significant-Gravitas/Auto-GPT)，2023。

+   Ruan 等人（2023）Jingqing Ruan、YiHong Chen、Bin Zhang、Zhiwei Xu、Tianpeng Bao、du qing、shi shiwei、Hangyu Mao、Xingyu Zeng 和 Rui Zhao。TPTU：基于大语言模型的人工智能智能体的任务规划和工具使用。发表于 *NeurIPS 2023 决策模型基础工作坊*，2023。网址 [https://openreview.net/forum?id=GrkgKtOjaH](https://openreview.net/forum?id=GrkgKtOjaH)。

+   Rumelhart 等人（1986）David E Rumelhart、Geoffrey E Hinton 和 Ronald J Williams。通过反向传播误差学习表示。*nature*，323（6088）：533–536，1986。

+   Shinn等人（2023）Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik R Narasimhan, 和Shunyu Yao. Reflexion：带有语言强化学习的语言智能体。发表于*第37届神经信息处理系统大会*，2023。网址 [https://openreview.net/forum?id=vAElhFcKW6](https://openreview.net/forum?id=vAElhFcKW6)。

+   Suzgun和Tauman Kalai（2024）Mirac Suzgun 和Adam Tauman Kalai. Meta-Prompting：通过任务无关的支撑增强语言模型。*arXiv预印本 arXiv:2401.12954*，2024。

+   Wang等人（2023a）Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao, Yuke Zhu, Linxi Fan, 和Anima Anandkumar. Voyager：一种开放式的具身智能体与大语言模型结合。发表于*第二届开放式智能体学习研讨会*，2023a。网址 [https://openreview.net/forum?id=pAMNKGwja6](https://openreview.net/forum?id=pAMNKGwja6)。

+   Wang等人（2023b）Xuezhi Wang, Jason Wei, Dale Schuurmans, Quoc V Le, Ed H. Chi, Sharan Narang, Aakanksha Chowdhery, 和Denny Zhou. 自一致性提高了语言模型中的思维链推理能力。发表于*第十一届国际学习表示会议*，2023b。网址 [https://openreview.net/forum?id=1PL1NIMMrw](https://openreview.net/forum?id=1PL1NIMMrw)。

+   Wang等人（2023c）Zhenhailong Wang, Shaoguang Mao, Wenshan Wu, Tao Ge, Furu Wei, 和Heng Ji. 在大语言模型中释放认知协同：通过多重人格自我协作解决任务的智能体。*arXiv预印本 arXiv:2307.05300*，2023c。

+   Wei等人（2022）Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, brian ichter, Fei Xia, Ed H. Chi, Quoc V Le, 和Denny Zhou. 思维链提示在大语言模型中引发推理。发表于Alice H. Oh, Alekh Agarwal, Danielle Belgrave, 和Kyunghyun Cho（编辑），*神经信息处理系统进展*，2022。网址 [https://openreview.net/forum?id=_VjQlMeSB_J](https://openreview.net/forum?id=_VjQlMeSB_J)。

+   Wu等人（2023）Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang, 和Chi Wang. Autogen：通过多智能体对话框架支持下一代大语言模型应用。*arXiv预印本 arXiv:2308.08155*，2023。

+   Xia等人（2008）Fen Xia, Tie-Yan Liu, Jue Wang, Wensheng Zhang, 和Hang Li. 基于列表的学习排序方法：理论与算法。发表于*第25届国际机器学习大会论文集*，ICML '08，第1192–1199页，纽约，纽约，美国，2008年。计算机协会出版。ISBN 9781605582054。

+   Xiong 等人（2023）熊凯、丁晓、曹毅欣、刘婷和秦冰。检查大型语言模型协作的内部一致性：通过辩论进行深入分析。在 Houda Bouamor、Juan Pino 和 Kalika Bali（编辑），*计算语言学协会发现：EMNLP 2023*，第7572–7590页，新加坡，2023年12月。计算语言学协会。doi: 10.18653/v1/2023.findings-emnlp.508。网址 [https://aclanthology.org/2023.findings-emnlp.508](https://aclanthology.org/2023.findings-emnlp.508)。

+   Xiong 等人（2024）熊淼、胡志远、陆鑫阳、李逸飞、傅杰、何俊贤 和 Bryan Hooi。LLM能否表达其不确定性？关于LLM中信心引导的实证评估。在 *第十二届国际学习表征会议*，2024年。网址 [https://openreview.net/forum?id=gjeQKFxFpZ](https://openreview.net/forum?id=gjeQKFxFpZ)。

+   Yao 等人（2023）姚顺宇、赵杰弗里、余典、杜楠、沙弗兰·伊扎克、Narasimhan Karthik R 和曹远。React：协同推理和行动在语言模型中的结合。在 *第十一届国际学习表征会议论文集*，2023年。网址 [https://openreview.net/forum?id=WE_vluYUL-X](https://openreview.net/forum?id=WE_vluYUL-X)。

+   Yin 等人（2023）张月尹、孙秋实、常成、郭启鹏、戴俊琦、黄轩晶、邱熙鹏。思想交流：通过跨模型通信增强大型语言模型的能力。在 Houda Bouamor、Juan Pino 和 Kalika Bali（编辑），*2023年自然语言处理实证方法会议论文集*，第15135–15153页，新加坡，2023年12月。计算语言学协会。doi: 10.18653/v1/2023.emnlp-main.936。网址 [https://aclanthology.org/2023.emnlp-main.936](https://aclanthology.org/2023.emnlp-main.936)。

+   Yu 等人（2018）余瑞驰、李昂、陈春富、赖睿欣、Vlad I. Morariu、韩馨彤、高名飞、林景永 和拉里·S·戴维斯。Nisp：使用神经元重要性分数传播修剪网络。在 *2018年IEEE/CVF计算机视觉与模式识别会议*，第9194–9203页，2018年。

+   Zhang 等人（2023a）张锦天、徐鑫、邓树民。探索LLM代理的协作机制：社会心理学视角。*arXiv 预印本 arXiv:2310.02124*，2023a。

+   Zhang 等人（2023b）张星华、余博文、余海阳、吕杨宇、刘廷文、黄飞、徐洪波 和 李永斌。更广泛和更深的LLM网络是更公平的LLM评估者。*arXiv 预印本 arXiv:2308.01862*，2023b。

+   Zhang 等人（2023）张一凡、杨晶琴、袁杨、姚奇智。与大型语言模型的累积推理。*arXiv 预印本 arXiv:2308.04371*，2023年。

+   Zheng 等人（2023）郑传扬、刘正英、谢恩泽、李正国 和 李宇。渐进提示改善大型语言模型中的推理能力。*arXiv 预印本 arXiv:2304.09797*，2023年。

+   Zhou 等（2023）Andy Zhou、Kai Yan、Michal Shlapentokh-Rothman、Haohan Wang 和 Yu-Xiong Wang。语言智能体树搜索统一了推理、行动和规划在语言模型中的应用。*arXiv 预印本 arXiv:2310.04406*，2023。

+   Zhou 等（2023）Xuanhe Zhou、Guoliang Li 和 Zhiyuan Liu。LLM 作为 DBA。*arXiv 预印本 arXiv:2308.05481*，2023。

+   Zhu 等（2023）Xizhou Zhu、Yuntao Chen、Hao Tian、Chenxin Tao、Weijie Su、Chenyu Yang、Gao Huang、Bin Li、Lewei Lu、Xiaogang Wang、Yu Qiao、Zhaoxiang Zhang 和 Jifeng Dai。Ghost in the minecraft: 通过大语言模型结合文本知识和记忆，提供适应开放世界环境的通用能力的智能体。*arXiv 预印本 arXiv:2305.17144*，2023。

+   Zhuge 等（2024）Mingchen Zhuge、Wenyi Wang、Louis Kirsch、Francesco Faccio、Dmitrii Khizbullin 和 Jürgen Schmidhuber。作为可优化图的语言智能体。*arXiv 预印本 arXiv:2402.16823*，2024。

## 附录 A 讨论与局限性

在实验中，我们将代码生成任务视为开放式生成任务的代表，并采用 BLEU 来决定在[第 3.3.2 节](https://arxiv.org/html/2310.02170v2#S3.SS3.SSS2 "3.3.2 Agent Team Reformation ‣ 3.3 Task Solving ‣ 3 Dynamic LLM-Powered Agent Network ‣ A Dynamic LLM-Powered Agent Network for Task-Oriented Agent Collaboration")的早停机制中，两个答案是否一致。实际上，性能可以通过任务特定的方法进一步提升，比如 CodeBLEU（Ren 等，[2020](https://arxiv.org/html/2310.02170v2#bib.bib36)）或 CodeT（Chen 等，[2023a](https://arxiv.org/html/2310.02170v2#bib.bib4)）。

在实际应用中，智能体评估指标可以与人工标注相结合，以便在面对数据稀缺问题时，给出更为准确的个体贡献评估结果。此外，我们仅仅将智能体选择与 DyLAN 的智能体团队重组相结合，作为动态智能体团队合作的初步步骤。如何在更细粒度上将脱离合作和合作内优化方法结合起来，以进一步提升性能和效率，仍有待探索。

此外，尽管智能体选择可以区分出贡献较大的智能体，但在极端情况下，如果大多数智能体被设计为与任务需求相悖，可能会导致低性能。例如，训练或提示生成代码的智能体在临床问题回答中可能会面临困难。为了应对高低性能智能体的不平衡，替代低得分智能体，复制得分较高的智能体可能是一个解决方案。另外，在极端情况下，除了智能体选择外，我们还可以通过验证，自动引入来自更强大 LLM 的智能体。

## 附录 B 实现细节

### B.1 详细实验设置

算法 1 DyLAN 在任意查询上的推理过程

输入：T-FFN $\mathcal{G}=(\mathcal{V}_{1},\cdots,\mathcal{V}_{T};E_{1},\cdots,E_{T-1,T})$，查询 $q$  输出：最终答案 $o$  // $E=\{(v_{t,i},v_{t+1,j})\}_{t=1}^{T-1},v_{t,i},v_{t+1,j}\in\mathcal{V}$  // $\phantom{E}=\bigcup_{t=1}^{T}\mathcal{V}_{t}$。  // $m_{t,i}\in\mathcal{M}_{t}$ 表示来自 $v_{t,i}\in\mathcal{V}_{t}$ 的响应。  对于 $t=1;T$ 进行： 如果在时间步 $t$ 进行代理团队重组，则：  $\mathcal{M}_{top}\leftarrow\mathrm{top-}k(\{m_{t-1,j}|v_{t-1,j}\in\mathcal{V}_{t-1}\})$   $E\leftarrow E\backslash\{(v_{t^{\prime},j},v_{t^{\prime\prime},j^{\prime}}),(v_{t^{\prime\prime},j^{\prime}},v_{t^{\prime},j})|$    $\quad m_{t^{\prime},j}\in\mathcal{M}_{top},m_{t^{\prime},j^{\prime}}\notin\mathcal{M}_{top},t^{\prime},t^{\prime\prime}\geq t-1\}$   $m_{t,j}\leftarrow m_{t-1,j},\forall(v_{t-1,j},v_{t,j})\in E$   否则：  $\forall i,\exists k,(v_{t,i},v_{t+1,k})\in E_{t,t+1}$，   $m_{t,i}\leftarrow$   $\quad f_{\mathrm{mp}}(\{m_{t-1,j}|(v_{t-1,j},v_{t,i})\in E_{t,t+1}\},v_{t,i})$    结束如果  如果提前停止，则：  $T\leftarrow t$    退出   结束如果  结束循环  $o\leftarrow\mathrm{postProcess}(\mathrm{maxCount}\{\mathcal{M}_{T}\})$

算法 2 DyLAN 中代理重要性分数的团队优化过程 $f_{\mathrm{Optim}}$

输入：输出 $o$，T-FFN $\mathcal{G}=(\mathcal{V},E)$  输出：代理重要性分数 $\bm{I}$  // $m_{t,i}\in\mathcal{M}_{t}$ 表示来自 $v_{t,i}\in\mathcal{V}_{t}$ 的响应。  $\mathrm{flag}\leftarrow\mathrm{False}$   对于 $t=T;1$ 进行：  如果 $\{v_{t,i}|\exists k,(v_{t-1,k},v_{t,i})\in E\}\neq\phi$，则：  如果 $\neg\mathrm{flag}$，则：  $\mathrm{flag}\leftarrow\mathrm{True}$   分配分数给 $I_{t,i}$   否则：  $\mathcal{M}_{t-1}\leftarrow$    $\quad\{m_{t-1,j}|(v_{t-1,j},v_{t,i})\in E\}$   $[w_{t-1,1,i},...,w_{t-1,m,i}]\leftarrow$    $\quad f_{t,i}^{(s)}(p_{i},q,\mathcal{M}_{t-1})$   $\bm{I}_{t-1,j}\leftarrow\bm{I}_{t-1,j}+\bm{I}_{t,i}w_{t-1,j,i},$   $\quad\exists(v_{t-1,j},v_{t,i})\in E$   结束如果  结束如果  结束循环

##### 常见设置

在所有实验中，如果没有特别说明，我们使用gpt-35-turbo-0301进行每种方法。GPT-4的版本是GPT-4-0613。在[表2](https://arxiv.org/html/2310.02170v2#S3.T2 "在3.4团队优化 ‣ 3 动态LLM驱动的智能体网络 ‣ 基于任务导向的智能体协作的动态LLM驱动的智能体网络")中，"(Codex)"表示OpenAI的code-davinci-002（Chen等人，[2021](https://arxiv.org/html/2310.02170v2#bib.bib6); OpenAI，[2023](https://arxiv.org/html/2310.02170v2#bib.bib30)）。所有带有非零温度的实验重复三次，并报告中位数。为了避免先前工作中的上下文长度问题（Du等人，[2023](https://arxiv.org/html/2310.02170v2#bib.bib10); Liu等人，[2024](https://arxiv.org/html/2310.02170v2#bib.bib21)），我们在DyLAN中为智能体设置了仅为$1$的内存空间，以保持前任的最新响应。我们为GR和AR任务设置了最大token为2048，为CG和DM任务设置了1024，以避免超出最大上下文长度。候选者的构建方式在[附录D](https://arxiv.org/html/2310.02170v2#A4 "附录D 提示模板 ‣ 基于任务导向的智能体协作的动态LLM驱动的智能体网络")中展示。我们在团队优化后将$N=4$设置为T-FFN，因为早停机制需要至少四个智能体才能容忍在特定时间步长下的一个不同响应（[第3.3.2节](https://arxiv.org/html/2310.02170v2#S3.SS3.SSS2 "3.3.2 智能体团队重组 ‣ 3.3 任务求解 ‣ 动态LLM驱动的智能体网络 ‣ 基于任务导向的智能体协作的动态LLM驱动的智能体网络")）；当超过2/3的智能体达成一致时，它允许有$4-\left\lceil{\frac{2}{3}N}\right\rceil=1$个例外响应。我们在DyLAN的智能体团队重组中使用了listwise ranker，因为与我们在[第C.5节](https://arxiv.org/html/2310.02170v2#A3.SS5 "C.5 不同的排名方法 ‣ 附录C 额外结果 ‣ 基于任务导向的智能体协作的动态LLM驱动的智能体网络")中测试的ELo评分（Herbrich等人，[2006](https://arxiv.org/html/2310.02170v2#bib.bib14)）或Sliding Window（Qin等人，[2023](https://arxiv.org/html/2310.02170v2#bib.bib35)）相比，效果和效率更高。我们在实验中使用相同的ranker来实现LLM-Blender（Jiang等人，[2023](https://arxiv.org/html/2310.02170v2#bib.bib16)）。在智能体团队重组中，我们设置$k=2$，因为这是最小的协作数，我们经验性地发现它在效果和效率之间带来了很好的平衡。为了避免位置偏差，对于每个时间步长$t$，我们在传递消息给$t$时，会打乱来自$t-1$时刻智能体的响应。详细的推理算法见[算法1](https://arxiv.org/html/2310.02170v2#alg1 "在B.1 详细实验设置 ‣ 附录B 实现细节 ‣ 基于任务导向的智能体协作的动态LLM驱动的智能体网络")。为了实现早停机制，我们需要判断DyLAN中同一层节点的答案是否一致。对于分类和决策问题，当答案相同时即认为一致；对于开放式生成问题，一致性通过BLEU得分的阈值来判断。

##### 推理任务实验

在一般推理中，我们通过匹配最后的“(X”或“(X)”，其中“X”代表A、B、C或D，从响应中提取答案。平均而言，代理团队在第三步时进行重组。他们最多可以进行$T=4$轮交互。我们还在{0, 0.2, 0.8, 1.0}之间搜索温度，以找到每个系统的最佳配置。在算术推理中，我们将温度设置为0用于单次执行和PHP，0.2用于LLM辩论、LLM-Blender和使用复杂CoT提示的DyLAN，1.0用于使用简单CoT提示的DyLAN，在[表3](https://arxiv.org/html/2310.02170v2#S3.T3 "在3.4团队优化 ‣ 3 动态LLM驱动的代理网络 ‣ 基于任务的代理协作的动态LLM驱动代理网络")中，因为使用相同提示的系统如果温度为零，则会给出相同的所有响应，导致性能下降。提示模板来自其原始研究，包括来自MATH数据集的常规CoT提示（Wei等， [2022](https://arxiv.org/html/2310.02170v2#bib.bib46)）和来自PHP的复杂CoT提示（Zheng等， [2023](https://arxiv.org/html/2310.02170v2#bib.bib57)）。我们遵循原始论文中的答案提取方法（Hendrycks等， [2021b](https://arxiv.org/html/2310.02170v2#bib.bib13)）。我们构建了DyLAN，分配了4个没有特定角色的代理，并让代理们在T-FFN公式下最多进行$T=4$轮交互。我们报告了每个类别的分类准确度，该准确度是跨学科的平均值，并且报告了在优化后的代理团队上运行DyLAN的API调用次数。

##### 代码生成任务实验

在代码生成任务中，我们将单次执行、反射（Reflexion）的温度设置为0，将LLM辩论（LLM Debate）、LLM-Blender、CodeT和DyLAN的温度设置为0.8，具体见[表2](https://arxiv.org/html/2310.02170v2#S3.T2 "在3.4团队优化 ‣ 3 动态LLM驱动的代理网络 ‣ 面向任务的代理协作的动态LLM驱动代理网络")。在DyLAN中，我们从[附录D](https://arxiv.org/html/2310.02170v2#A4 "附录D 提示模板 ‣ 面向任务的代理协作的动态LLM驱动代理网络")中的12个候选者中优化了四个代码编写代理和四个代码审查代理。选定的代码编写代理包括“Python助手”、“算法开发者”、“计算机科学家”和“程序员”；选定的代码审查代理包括“语法检查器”、“单元测试器”、“反射器”和“排名器”。“语法检查器”是纯外部工具，使用代码解释器进行语法检查，不涉及LLM，而“单元测试器”则配备了代码解释器。当LLM生成的代码处于格式````python\n(code)\n````中时，该工具会被触发。在DyLAN中，代码编写者提供的解决方案会被代码审查者最多审核$T=6$轮。在时间步$t=1,3,4,6$时，代码编写者提供解决方案，而代码审查者在$t=2,5$时进行审查。代理团队重组发生在$t=4$时。为了确保每个代理的参与，早停机制在第三层及之后（$t\geq 3$）发挥作用。我们在早停机制中使用BLEU分数。我们通过sacreBLEU²²2来计算BLEU（sacreBLEU的签名为“nrefs:1$|$case:mixed$|$eff:no$|$tok:13a$|$smooth:exp$|$version:2.3.1”）。 （见Post, [2018](https://arxiv.org/html/2310.02170v2#bib.bib31)）。对于答案后处理，我们存储来自单元测试器的所有单元测试（如果系统中存在），并从所有节点中通过随机选择前5个通过最多测试的代码补全作为最终输出。

##### 决策任务实验

在决策任务中，我们将温度设置为0，适用于DyLAN和[表2](https://arxiv.org/html/2310.02170v2#S3.T2 "在3.4团队优化 ‣ 3 动态LLM驱动的代理网络 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")中的所有基准方法。对于具有自一致性的ReAct（在表中表示为ReAct-SC）（Wang et al., [2023b](https://arxiv.org/html/2310.02170v2#bib.bib44)），我们为每个响应采样三次。在DyLAN中，我们从8个候选代理中优化了4个代理，这些候选代理在[附录D](https://arxiv.org/html/2310.02170v2#A4 "附录D 提示模板 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")中展示。所有方法也都在gpt-35-turbo-1106上进行。我们没有选择LASER（Ma et al., [2023](https://arxiv.org/html/2310.02170v2#bib.bib26)）作为基准方法，因为它需要GPT-4才能获得更好的性能，并且它将每一页中的所有有效动作提取为函数调用，而不是由代理本身检测到，我们决定这是一种不同的设置。我们将WebShop环境的页面分为3部分：用于“搜索”部分的初始化页面，用于“探索”部分的商品列表页面，以及用于“商品”部分的商品详情页面。因此，我们成功地从[附录D](https://arxiv.org/html/2310.02170v2#A4 "附录D 提示模板 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")中的代理中为每个部分优化了团队：“搜索优化器”，“预算分析师”，“指令分析师”，“决策反射者”用于“搜索”组，“决策者”，“预算分析师”，“产品探索者”，“指令分析师”用于“探索”组，以及“预算分析师”，“描述阅读者”，“决策者”，“结果估算者”用于“商品”组。代理在每个动作的最大步数为$T=4$。我们简单地将每个决策的前一动作的观察结果串联起来。对于答案后处理，我们从$V_{T}$的输出中跳过无效的动作。

### B.2 代理重要性评分的计算

为了在DyLAN下实现代理选择算法，只需将一句话注入到T-FFN中每个节点的提示末尾：“除了答案之外，给出一个从1到5的评分，评估其他代理的解决方案。将所有$\{num_{\mathrm{p}}\}$评分以[[1, 5, 2, ...]]的形式展示”，其中$num_{\mathrm{p}}$表示该节点的前置节点数量。该提示作为[第3.4节](https://arxiv.org/html/2310.02170v2#S3.SS4 "3.4团队优化 ‣ 3 动态LLM驱动的代理网络 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")中的$f_{t,i}^{(s)}$函数，我们在提取节点间传递的消息时，同时从响应中提取$w_{t,i,j}$。这些评分经过归一化处理，使得它们的总和（$\sum_{i=1}^{N}w_{t,i,j}$）等于1。为了避免位置偏差，当评分时，来自前一步的代理的响应会被随机打乱。

在[算法2](https://arxiv.org/html/2310.02170v2#alg2 "在B.1详细实验设置 ‣ 附录B 实现细节 ‣ 面向任务的代理协作的动态LLM驱动代理网络")中，初始贡献分配到最后一层的节点上。对于推理和决策任务，我们将贡献均匀分配给在最后一层给出一致答案的代理。在代码生成任务中，我们在最终一轮中将贡献均匀分配给没有语法错误的答案。

## 附录C 额外结果

本节展示了详细的结果和附加实验。

| 指标 | 数据集 | GR | CG |
| --- | --- | --- | --- |
| NA | - | 63.5 | 76.2 |
| 随机选择 | - | 64.8 | 75.6 |
| 人工先验 | - | 66.7 | 78.0 |
| 代理 | 1% | 68.9 | 79.3 |
| 重要性分数 | 10% | 72.2 | 82.3 |
| 100% | 73.6 | 82.9 |

表9：在DyLAN进行团队优化时，不同指标在GR和CG任务中用于代理选择的实验结果。“数据集”表示用于团队优化的数据集比例。

### C.1 团队优化的数据效率

我们进一步通过基于不同数据量进行代理选择，展示了代理选择的数据效率。实验在GR任务（与[表8](https://arxiv.org/html/2310.02170v2#S4.T8 "在4.3消融研究 ‣ 4 实验 ‣ 面向任务的代理协作的动态LLM驱动代理网络")相同）和CG任务的五个主题上进行。我们从原始数据集中随机抽取1%和10%的子集。代理选择的代理重要性分数在这些子集上进行平均，并且选定的团队在整个数据集上进行测试。我们将随机选择和人工先验选择作为基准。后者通过GPT-4根据任务和代理描述进行模拟（[附录D](https://arxiv.org/html/2310.02170v2#A4 "附录D 提示模板 ‣ 面向任务的代理协作的动态LLM驱动代理网络")）。如[表9](https://arxiv.org/html/2310.02170v2#A3.T9 "在附录C 额外结果 ‣ 面向任务的代理协作的动态LLM驱动代理网络")所示，通过对原始数据集的10%进行团队优化，DyLAN在GR和CG任务上的性能与使用整个数据集相比，损失仅为0.2（GR）和0.6（CG）。我们可以观察到，即使只有1%的原始数据集，DyLAN在CG任务上也能比随机选择取得显著提高，提升幅度为+3.7。从观察来看，在不同数据集比例下，带有工具增强的代理始终会在团队优化中被选择，这表明代理重要性分数作为一个指标的有效性。有关人工先验的详细分析，请参阅[第C.3节](https://arxiv.org/html/2310.02170v2#A3.SS3 "C.3 人工先验与代理重要性分数 ‣ 附录C 额外结果 ‣ 面向任务的代理协作的动态LLM驱动代理网络")。

### C.2 DyLAN中不同基础模型的鲁棒性

| 方法 | Pass@1 | #API调用次数 |
| --- | --- | --- |
| 单代理方法 |
| 单次执行 | 88.4 | (+0.0) | 1.00 |
| LATS | 94.4 | (+6.0) | $>$40.00 |
| Reflexion | 91.4 | (+3.0) | 7.32 |
| 多代理方法 |
| Meta-GPT | 85.9 | (-3.5) | $>$30.00 |
| AgentVerse | 89.0 | (+0.6) | 27.00 |
| DyLAN（我们的） | 92.1 | (+3.7) | 15.94 |

表10：在GPT-4-0613上的CG任务实验结果。括号中的数字表示与单次执行或直接执行的差异。粗体表示我们方法的结果，最佳结果下划线标出。

除了在[表2](https://arxiv.org/html/2310.02170v2#S3.T2 "在3.4团队优化 ‣ 3 动态LLM驱动的代理网络 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")中使用GPT-3.5进行DyLAN的CG任务实验外，我们还在[表10](https://arxiv.org/html/2310.02170v2#A3.T10 "在C.2 DyLAN中不同基础模型的鲁棒性 ‣ 附录C 额外结果 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")中使用了GPT-4进行实验。由于预算限制，我们直接重复使用了论文中报告的基线性能，包括LATS (Zhou等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib58))，Reflexion (Shinn等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib41))，Meta-GPT (Hong等，[2024](https://arxiv.org/html/2310.02170v2#bib.bib15))，以及AgentVerse (Chen等，[2024](https://arxiv.org/html/2310.02170v2#bib.bib7))，并按API调用次数估算成本。DyLAN也是由基于GPT-3.5优化的代理构建的，如[章节B.1](https://arxiv.org/html/2310.02170v2#A2.SS1 "B.1 详细实验设置 ‣ 附录B 实施细节 ‣ 用于任务导向代理协作的动态LLM驱动代理网络")所示。我们发现DyLAN始终优于其他多代理方法，表明动态代理团队在T-FFN结构中的有效性，以及优化结果在代理团队上的跨模型可迁移性。尽管LATS的表现优于DyLAN，但它每个样本需要超过40次GPT-4调用来进行推理时的MCTS，这表明其效率较差。

### C.3 人类先验与代理重要性评分

我们进一步调查了通过无监督度量代理重要性评分选择的这些代理与人类先验（例如，这些预定义的角色）之间的差异。为此，我们为MMLU数据集中的每个学科计算了7个代理的代理重要性评分。作为例子，我们展示了“医生”和“程序员”在[表11](https://arxiv.org/html/2310.02170v2#A3.T11 "In C.3 Human Priors and Agent Importance Scores ‣ Appendix C Additional Results ‣ A Dynamic LLM-Powered Agent Network for Task-Oriented Agent Collaboration")和[表12](https://arxiv.org/html/2310.02170v2#A3.T12 "In C.3 Human Priors and Agent Importance Scores ‣ Appendix C Additional Results ‣ A Dynamic LLM-Powered Agent Network for Task-Oriented Agent Collaboration")中，代理在所有代理中得分最高的学科。

| 角色 | 医生 | 程序员 |
| --- | --- | --- |
| 前10名学科 | 高中计算机科学 | 高中物理 |
| 临床知识 | 电气工程 |
| 大学生物学 | 高中政府与政治 |
| 专业医学 | 大学计算机科学 |
| 营养学 | 大学化学 |
| 高中美国历史 | 高中数学 |
| 人类衰老 | 形式逻辑 |
| 解剖学 | 抽象代数 |
| 高中生物学 | 机器学习 |
| 高中心理学 | 计算机安全 |

表11：在DyLAN实验中，7个代理在GR任务上的代理重要性评分排名最高的学科。绿色标注表示从人类视角来看与角色相关的领域，标注由人工完成。

尽管大多数学科看起来与基于人类先验知识的代理角色（绿色标注）相对合理地对齐，但仍有一些学科与人类先验不符，例如，“医生”在“高中计算机科学”这一学科上得分最高。它展示了人类先验与基于人类或LLM生成提示的代理重要性评分评估结果之间的差异。

| 角色 | 数学家 | 律师 | 历史学家 | 经济学家 | 心理学家 |
| --- | --- | --- | --- | --- | --- |
| 前10名学科 | 大学物理 | 高中微观经济学 | 美国外交政策 | 高中计算机科学 | 全球事实 |
| 美国外交政策 | 医学遗传学 | 计量经济学 | 法学 | 公共关系 |
| 大学计算机科学 | 史前学 | 世界宗教 | 逻辑谬误 | 商业伦理 |
| 计量经济学 | 社会学 | 公共关系 | 专业会计 | 高中美国历史 |
| 营销 | 人类衰老 | 高中政府与政治 | 高中微观经济学 | 哲学 |
| 高中数学 | 管理 | 哲学 | 高中欧洲历史 | 道德争议 |
| 抽象代数 | 形式逻辑 | 天文学 | 计算机安全 | 管理 |
| 国际法 | 世界宗教 | 高中统计学 | 道德争议 |  |
| 专业会计 | 法学 | 机器学习 | 专业法律 |  |
| 人类性学 | 国际法 | 高中欧洲历史 | 大学数学 |  |

表 12：代理在同一实验中具有最高代理重要性评分的学科，如[表 11](https://arxiv.org/html/2310.02170v2#A3.T11 "在 C.3 人类优先选择与代理重要性评分 ‣ 附录 C 额外结果 ‣ 基于动态大语言模型的任务导向代理协作网络")所示。绿色标注表示与人类视角相关性较高的领域。

我们还比较了当前基于代理重要性评分的代理选择方法与基于人类优先选择的实现，在 MMLU 和 HumanEval 数据集的几个学科上进行了测试。对于人类优先选择，我们设置了模拟人类根据每个代理的任务描述和角色提示来选择代理进行协作的 GPT-4。我们在[附录 D](https://arxiv.org/html/2310.02170v2#A4 "附录 D 提示模板 ‣ 基于动态大语言模型的任务导向代理协作网络")中提供了提示模板。如[表 13](https://arxiv.org/html/2310.02170v2#A3.T13 "在 C.3 人类优先选择与代理重要性评分 ‣ 附录 C 额外结果 ‣ 基于动态大语言模型的任务导向代理协作网络")所示，基于代理重要性评分的实现稳定地优于人类优先选择。其主要原因有两个：(1) 与后置优化方法相比，优先选择可能无法把握代理的实际行为，无法理解哪些代理在真实的协作过程中对其他代理最有贡献或最有帮助。因此，在 MMLU 数据集中的高中统计学、临床知识和公共关系学科中，优先选择的表现甚至比随机选择更差。(2) 人类优先选择可能难以理解没有来自同伴代理评分的工具增强。从我们的观察来看，“单元测试器”和“语法检查器”并未被选中用于代码生成任务，这可能导致了较低的性能。

| #代理 | 优化 | 大学 | 管理 | 高中 | 临床 | 公共 | 综合 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 指标 | 数学 | 统计学 | 知识 | 关系 |
| 7 | （优化前） | 40.0 | 76.2 | 65.1 | 69.8 | 54.5 | 63.5 (+0.0) |
| 4 | 随机选择 | 45.0 | 71.4 | 67.4 | 71.7 | 54.5 | 64.8 (+1.3) |
| 人类优先选择 | 60.0 | 80.1 | 65.1 | 69.8 | 54.5 | 66.7 (+3.2) |
| 代理重要性评分 | 65.0 | 90.5 | 74.4 | 75.5 | 59.1 | 73.6 (+10.1) |  | #代理 | 优化指标 | Pass@1 | #API 调用 |
| 12 | （优化前） | 76.2 (+0.0) | 23.04 |
| 8 | 随机选择 | 75.6 (-0.6) | 17.73 |
| 人类优先选择 | 78.0 (+1.8) | 16.37 |
| 代理重要性评分 | 82.9 (+6.7) | 16.85 |

表13：在GR任务（上表）和CG任务（下表）中，代理选择的不同指标在五个主题上的详细表现。GR任务中的五个主题及其他设置与[表8](https://arxiv.org/html/2310.02170v2#S4.T8 "在4.3 消融研究 ‣ 4 实验 ‣ 基于动态LLM的任务导向代理协作网络")相同。上表中的整体准确率表示五个主题的准确率。

### C.4 DyLAN在温度下的稳定性

![参见标题说明](img/c2868da5f35a59997a1e041b2b8867c7.png)![参见标题说明](img/4a3f1ad69f624ae3d73ece3864decf78.png)

图4：在低温和高温下，AR（左）和CG（右）任务的不同方法的表现。DyLAN在不同温度下表现出更好的鲁棒性，甚至能利用更高的温度。

我们在低温和高温条件下，对AR任务（使用简单的CoT提示）和CG任务进行了几种方法的测试，并在温度不为零时重复了每个实验三次。我们在[图4](https://arxiv.org/html/2310.02170v2#A3.F4 "在C.4 DyLAN在温度下的稳定性 ‣ 附录C 额外结果 ‣ 基于动态LLM的任务导向代理协作网络")中展示了实验结果。根据实验结果，我们发现DyLAN在不同超参数下更加稳定。

实验表明，温度对算术推理和代码生成任务有很大影响。在[图4](https://arxiv.org/html/2310.02170v2#A3.F4 "在C.4 DyLAN在温度下的稳定性 ‣ 附录C 额外结果 ‣ 基于动态LLM的任务导向代理协作网络")中，我们发现大多数基准方法在温度升高时性能显著下降，但DyLAN在各种温度下表现出强大的鲁棒性。我们惊讶地发现，DyLAN在温度升高时取得了更好的结果，这表明它从多样性中受益，而不是被高温代理的低质量答案干扰。代理团队的重组可能通过在代理回复变得更加多样化时保留最佳回复，从而提高准确性。总之，不同角色的协作在动态架构中表现得既高效又稳健。尽管如此，较高的温度要求DyLAN进行更多的API调用（在AR任务中平均增加约0.98次（温度：$0.2\rightarrow 1.0$））。

### C.5 不同的排名方法

| 排名 | 总体 | #API |
| --- | --- | --- |
| 方法 | 准确率 | 调用次数 |
| 列举排名器 | 70.5 | 4.39 |
| 成对 | LLM-Blender | 70.1 | 19.27 |
| Elo得分 | 70.3 | 19.55 |
| 滑动窗口 | 70.3 | 11.40 |

表14：DyLAN在GR任务中代理团队重组时，采用不同排名方法的整体准确率（%）。其他设置与[表5](https://arxiv.org/html/2310.02170v2#S4.T5 "在4.1 设置 ‣ 4 实验 ‣ 基于动态LLM的任务导向代理协作网络")相同。

我们还测试了不同的排名方法，用于DyLAN在GR任务中的代理团队重组。我们测试了使用自定义提示的逐项排名器、来自原始LLM-Blender的成对GPT排名器（Jiang et al., [2023](https://arxiv.org/html/2310.02170v2#bib.bib16)）、TrueSkill的Elo得分（Herbrich et al., [2006](https://arxiv.org/html/2310.02170v2#bib.bib14)，同样使用成对排名器实现）以及带有滑动窗口算法的成对排名器（Qin et al., [2023](https://arxiv.org/html/2310.02170v2#bib.bib35)）。在[表14](https://arxiv.org/html/2310.02170v2#A3.T14 "在C.5不同排名方法 ‣ 附录C额外结果 ‣ 基于动态LLM的任务导向代理协作网络")中，我们展示了不同的排名方法对性能的影响相对较小，这可能是由于GPT-3.5具有较强的区分能力，但成对排名方法始终消耗更高的计算成本。因此，我们在DyLAN的实现中选择了逐项排名器。

### C.6 代理重要性得分是否能捕捉到实际贡献？

Shapley值是评估多代理系统中单个代理贡献的广泛使用的监督度量。尽管它不适用于无监督的团队优化，但通过将其视为衡量个体贡献的真实度量，我们可以用它来验证代理重要性得分。我们为基于LLM的代理协作系统实现了一个简化算法。鉴于协作过程在时间前馈网络的表述中是对称的（[第3.2节](https://arxiv.org/html/2310.02170v2#S3.SS2 "3.2 时间前馈网络 (T-FFNs) ‣ 3 动态LLM驱动的代理网络 ‣ 基于动态LLM的任务导向代理协作网络")），我们可以将原公式中的排列集合（Lundberg & Lee, [2017](https://arxiv.org/html/2310.02170v2#bib.bib24)）简化为组合集合：

|  | $S_{i}(\mathcal{R})=\frac{1}{ | \mathcal{C} | | \mathcal{R} | }\sum_{\mathcal{T}\in \mathcal{C}}\frac{}{}(\mathrm{Performance}(\mathcal{T}\cup\{i\})-\mathrm{% Performance}(\mathcal{T})),$ |  | (13) |
| --- | --- | --- | --- | --- | --- | --- | --- |

其中，$\mathcal{R}$是系统中代理的集合，$\mathcal{C}$是$\mathcal{R}\backslash\{i\},i\in\mathcal{R}$的组合集合，$\mathrm{Performance}$表示系统在当前任务上的整体表现，例如分类准确率或Pass@1。该度量需要系统在不同代理子集上的真实结果和多次运行结果。我们对分类任务使用分类准确率，对代码生成任务使用Pass@1。然而，由于其组合复杂性，当代理数量增加时，计算成本仍然过高。

为了检验代理重要性评分作为Shapley值的代理选择指标，我们还从7个候选人中随机选择了三种三人组合，组建了一个T-FFN，并计算了GR任务上的Shapley值和代理重要性评分。在GR任务中，DyLAN中候选人的角色与MMLU中人类先验的类别匹配，包括“数学家”和“程序员”属于STEM，“律师”和“历史学家”属于人文学科，“经济学家”和“心理学家”属于社会科学，“医生”则属于“其他”类别中的临床问题。在实验中，当T-FFN中至少有一个代理匹配问题类别时，称为领域内场景；反之为领域外场景。

在[表格15](https://arxiv.org/html/2310.02170v2#A3.T15 "在C.6章节中，代理重要性评分是否能够反映实际贡献？ ‣ 附录C 其他结果 ‣ 基于动态大语言模型的任务导向代理协作网络")中，我们报告了Shapley值与代理重要性评分之间的相关性。我们好奇代理重要性评分是否可以作为Shapley值的无监督替代。因此，我们计算了两种基于列表的度量指标，以衡量代理重要性评分与Shapley值分布之间的相似性：KL散度和ListMLE（Xia等，[2008](https://arxiv.org/html/2310.02170v2#bib.bib48)）。结果表明，在领域内场景中，两种度量的分布之间存在高度相关性。

总结来说，我们使用Shapley值作为衡量个人贡献的自明指标，显示代理重要性评分作为一种有前景的、无监督的替代方法，且计算复杂度较低。

| 指标 | 领域内 | 领域外 |
| --- | --- | --- |
| $D_{\mathrm{KL}}$ | $\mathcal{L}_{\mathrm{ListMLE}}$ | $D_{\mathrm{KL}}$ | $\mathcal{L}_{\mathrm{ListMLE}}$ |
| Shapley值 | $0$ | $0.673$ | $0$ | $0.674$ |
| 代理重要性评分 | $\mathbf{0.229\times 10^{-3}}$ | $\mathbf{0.686}$ | $0.347\times 10^{-3}$ | $0.693$ |
| 均匀分布 | $0.359\times 10^{-3}$ | $0.693$ | $0.327\times 10^{-3}$ | $0.693$ |

表格15：DyLAN在GR任务中量化代理贡献的不同度量之间的相关性。我们计算了Shapley值与其他度量指标之间的KL散度（$D_{\mathrm{KL}}$）和ListMLE损失（$\mathcal{L}_{\mathrm{ListMLE}}$），并报告了每个学科的平均值。领域内列表示DyLAN中至少有一个代理与学科类别匹配，具体参见[章节C.6](https://arxiv.org/html/2310.02170v2#A3.SS6 "C.6 代理重要性评分是否能够反映实际贡献？ ‣ 附录C 其他结果 ‣ 基于动态大语言模型的任务导向代理协作网络")，而领域外则表示没有任何代理匹配该学科。

![请参阅说明](img/a420b6f880598bf864ec41970e534f72.png)

图5：DyLAN解决代码生成任务的案例。不同的代理被招募来编写代码并提供反馈。在时间步$t=2,5$时，要求代码审阅者提供代码审查。结果在正确性、效率和可读性方面逐层变得更好。不同的实现方向通过隐式的多条路径向前推进。由于篇幅限制，我们忽略了多轮响应中的同行评分以计算代理重要性分数。

### C.7 案例研究

![参考说明文字](img/f6b5fbae2f9321d573b9217d0d7ceae3.png)

图6：DyLAN解决一般推理任务的案例。不同的代理被招募来给出和完善解决方案。第一时间步的结果是错误的，但第二时间步的结果是正确的。它包括了来自代理的评分，用于计算代理重要性分数。

![参考说明文字](img/856f22377e2e3387674fe3b6121b15df.png)

图7：DyLAN解决代码生成任务的失败案例。代码反射器提供的优化方向是错误的。

在[图5](https://arxiv.org/html/2310.02170v2#A3.F5 "在C.6 代理重要性分数是否能捕捉实际贡献? ‣ 附录C 额外结果 ‣ 基于动态LLM的任务导向代理协作网络")和[图6](https://arxiv.org/html/2310.02170v2#A3.F6 "在C.7案例研究 ‣ 附录C 额外结果 ‣ 基于动态LLM的任务导向代理协作网络")中，我们展示了DyLAN在代码生成和一般推理任务中的案例。首先，我们注意到图之间的通信结构不同，展示了DyLAN在不同查询中的动态架构。前者在$t=4$时给出答案，后者在$t=2$时给出答案。我们还注意到，答案沿时间轴逐渐变得更好。在[图6](https://arxiv.org/html/2310.02170v2#A3.F6 "在C.7案例研究 ‣ 附录C 额外结果 ‣ 基于动态LLM的任务导向代理协作网络")中，"数学家"代理被选中处理算术查询并给出了正确答案，同时说服了其他代理，代理选择方法是有效的。我们还可以观察到，代理重要性分数的分布是合理的。此外，指示LLM代理评估前任的评分会促使他们反思前任的回答，这可能有助于给出更好的答案。最后但同样重要的是，不同角色的代理引导了多样化的对话，并充分发挥了每个代理的作用，这有利于性能和鲁棒性。

最后但同样重要的是，我们对[图7](https://arxiv.org/html/2310.02170v2#A3.F7 "在C.7案例研究 ‣ 附录C 额外结果 ‣ 基于动态LLM的任务导向代理协作网络")中的失败案例进行了定性分析。

## 附录D 提示模板

在 DyLAN 中，代理通过从开源代码库³³3[https://github.com/GoGPTAI/ChatGPT-Prompt/blob/main/prompts.csv](https://github.com/GoGPTAI/ChatGPT-Prompt/blob/main/prompts.csv) 提取的提示、相关的研究项目（Du 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib10)；Shinn 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib41)；Zheng 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib57)）以及 GPT-4-0613 的生成结果进行角色分配，除了手动构建。每个代理的提示是通过将角色提示和可选工具描述串联构成的，作为系统提示和指令提示。我们展示了不同数据集的指令模板、所有代理的提示及其来源，详见 LABEL:tab:role-prompts。我们在括号中注释了每个提示使用的任务，以及每个提示模板的来源。我们省略了 MATH 数据集（Hendrycks 等，[2021b](https://arxiv.org/html/2310.02170v2#bib.bib13)）和 PHP Zheng 等（[2023](https://arxiv.org/html/2310.02170v2#bib.bib57)）中的 AR 任务的上下文示例，以及 ReAct 中的 WebShop（Yao 等，[2023](https://arxiv.org/html/2310.02170v2#bib.bib51)）。

表格16：不同数据集和代理使用的指令与提示模板。

| 提示 | 内容 | 来源 |
| --- | --- | --- |

| MMLU 指令（GR） | 这是问题：{`question`}

这些是其他代理提供的解决方案：{`responses`}

使用其他代理的推理作为附加建议，并进行批判性思考，你能给出更新的答案吗？逐步检查你的解决方案和其他代理的答案。注意，他们的答案可能都是错的。将你的答案以(X)的形式放在回答的末尾。(X)代表选择(A)、(B)、(C)或(D)。| 手动 |

| MATH 指令（AR） | 按照给定的示例回答数学问题。

{`question`}

这些是其他代理提供的解决方案：{`responses`}

使用其他代理的推理作为附加建议，并进行批判性思考，你能给出更新的答案吗？逐步检查你的解决方案和其他代理的答案。注意，他们的答案可能都是错的。| 手动 |

| HumanEval 指令（CG） | 你必须通过修正之前的实现来完成我给你的 Python 函数。使用其他信息作为提示。确保使用我指定的缩进。此外，你只能在代码/注释中写出你的回答。

[改进实现]：

````python`
{`function signature`}
```

请按照模板重复函数签名，并在[改进实现]中完成新的实现。如果不需要任何更改，只需重新编写 Python 代码块中的实现。| 获取 |

| WebShop 指令（DM） | {`history of observations and` `actions`}

这些是其他代理提供的建议下一步操作：{`actions from other agents`}

利用其他代理提供的解决方案作为附加建议，并进行批判性思考，您能否根据之前的观察和行动给出更新的行动？请在搜索后选择项目（点击产品名称‘B0…‘）。当您购买该商品时（‘点击[立即购买]‘），请确保已选择商品并进入其描述页面。请勿多次搜索。根据示例及新指令的前期行动和观察，请以“行动: 搜索[…]”或“行动: 点击[…]”的格式给出您的下一个行动。| 已获取 |

| 人类优先选择指令 (-) | 一些代理将协作处理同一任务查询。请根据任务描述和代理的个人简介，选择候选代理的最佳组合。

- 任务: {`subject`}

- 代理

{ - `代理/代码编写者/评审` `t` (`姓名`): `角色提示/描述\n`}${}_{t=1}^{n}$

我们希望从这些候选人中选择`k`个`代理/代码编写者/评审`。请按以下格式写出代理ID：[1, 2, 3, 4]。可能有多个代理具有相同的ID。 | 手动 |

| 排名指令 (-) | 这里是问题: {`question`}

这是其他代理提供的解决方案: {`responses`}

请选出最佳的2个解决方案，并逐步思考。将您的答案以[1,2]或[3,4]的形式放在回答的最后。 | 手动 |

| 数学家 (GR) | 您是一名数学家，擅长数学游戏、算术计算和长期规划。 | 已获取 |
| --- | --- | --- |
| 程序员 (GR) | 您是一名程序员，擅长计算机科学、工程学和物理学。您有设计和开发计算机软件和硬件的经验。 | 已获取 |
| 律师 (GR) | 您是一名律师，擅长法律、政治和历史。 | 已获取 |
| 历史学家 (GR) | 您是一名历史学家，研究和分析过去的文化、经济、政治和社会事件，从原始资料中收集数据，并利用这些数据建立关于历史各个时期发生的事件的理论。 | 已获取 |
| 经济学家 (GR) | 您是一名经济学家，擅长经济学、金融和商业。您有丰富的经验，能够理解图表并解释全球经济中普遍存在的宏观经济环境。 | 已获取 |
| 心理学家 (GR) | 您是一名心理学家，擅长心理学、社会学和哲学。您为人们提供科学建议，帮助他们感觉更好。 | 已获取 |
| 医生 (GR) | 您是一名医生，为疾病或病症提供创意治疗方案。您能够推荐常规药物、草药治疗及其他自然替代疗法。在提供建议时，您还会考虑患者的年龄、生活方式和病史。 | 已获取 |
| Python 助手 (CG) | 你是一个 Python 编写助手，AI 只会回复 Python 代码，而非英语。用户将给你一个函数签名和文档字符串。根据这个格式（重述函数签名）编写完整实现。 | 已检索 |
| 算法开发者 (CG) | 你是一个算法开发者，擅长开发和利用算法解决问题。你必须用 Python 代码回应，不允许有自由流动的文字（除非在注释中）。用户将给你一个函数签名和文档字符串。根据这个格式（重述函数签名）编写完整实现。 | 已检索 |
| 计算机科学家 (CG) | 你是一个计算机科学家，擅长编写高性能代码并识别边缘情况，解决实际问题。你必须用 Python 代码回应，不允许有自由流动的文字（除非在注释中）。用户将给你一个函数签名和文档字符串。根据这个格式（重述函数签名）编写完整实现。 | 已检索 |
| 程序员 (CG) | 你是一个智能程序员。你必须完成用户给定的 Python 函数，并且必须遵循他们提供的格式来回答！你只能用注释和实际代码回应，不允许有自由流动的文字（除非在注释中）。 | 已检索 |
| 编程艺术家 (CG) | 你是一个编程艺术家。你编写的 Python 代码不仅具有功能性，而且富有美感和创意。你的目标是使代码成为一种艺术形式，同时保持其实用性。用户将给你一个函数签名和文档字符串。根据这个格式（重述函数签名）编写完整实现。 | 已生成 |
| 软件架构师 (CG) | 你是一个软件架构师，擅长设计和构建可扩展、可维护和稳健的代码。你的回应应专注于软件设计的最佳实践。用户将给你一个函数签名和文档字符串。根据这个格式（重述函数签名）编写完整实现。 | 已生成 |
| 单元测试员 (CG) | 你是一个 AI 编程助手，能够为给定的函数签名和文档字符串编写独特、多样且直观的单元测试。 | 已检索 |
| 语法检查器 (CG) | `空` | - |
| 代码反射器 (CG) | 你是一个 Python 编写助手。用户将给你一系列相同函数签名的实现。写几句话解释这些实现是否正确，并说明为什么。这个评论将作为提示，你的目标是写出在 [反思 n] 后对第 n 个先前实现的思考。 | 已生成 |
| 调试员 (CG) | 作为调试员，专门负责在Python代码中发现并修复错误。你将获得一个带有错误的函数实现。你的目标是识别错误并提供修正后的实现。包括注释，解释问题所在以及如何修复。 | 生成 |
| 质量经理 (CG) | 作为质量经理，确保代码在可读性、效率和准确性方面符合高标准。你将获得一个函数实现，需要对其进行代码审查。评论其正确性、效率和可读性，并在需要时提出改进建议。 | 生成 |
| 排序员 (CG) | 作为Python编写助手，你将获得一系列相同函数签名的函数实现。你需要根据正确性、效率和可能的边界情况选择最佳的两个实现。 | 生成 |
| 搜索优化员 (DM) | 作为搜索优化员，分析用户模糊或广泛的指令，建议更精确有效的关键词或筛选器，以准确找到符合用户特定需求的产品。在给出下一步操作时，请更多关注搜索操作，特别是提供有用且准确的搜索词“search[…]”。 | 生成 |
| 预算分析员 (DM) | 作为预算分析员，指导用户根据产品类别、市场价格和个人财务限制设定实际可行的购物预算。逐一分析产品是否符合预算。请避免多次搜索。在给出下一步操作时，请更多关注点击“click[…]”中的产品。 | 生成 |
| 产品探索员 (DM) | 作为产品探索员，提供如何根据产品的特性、价格和评价有效比较不同产品的指导。建议在不同产品类别中做出明智决策时需要考虑的关键因素。建议点击“click[¡ Prev]”或“click[Next ¿]”浏览不同页面，点击产品名称以查看其详情。 | 生成 |
| 指令分析员 (DM) | 作为指令分析员，评估客户的需求和偏好，并提供一种结构化的方法，确保所选产品与这些要求对接。请不要持续精炼搜索。建议查看可能与指令对接的产品，若没有，提供下一个步骤的最合理操作。 | 生成 |
| 描述阅读员 (DM) | 作为描述阅读员，详细说明如何阅读和解读产品描述和规格，以便将其与客户的特定需求匹配，重点识别关键特性、优点和潜在缺点。建议进入每个产品并点击“click[描述]”查看产品详情。 | 生成 |
| 决策者 (DM) | 作为决策者，你会更有信心购买相关产品，而无需进一步精细筛选搜索结果。如果你认为某个产品是合适的选择，点击该产品并说服其他代理点击“click[立即购买]”。如果不合适，推荐最合逻辑的下一步行动，以快速适应下一个产品。 | 生成 |
| 决策反思者 (DM) | 作为决策反思者，提供一个框架，帮助客户批判性地评估他们的潜在购买，考虑他们的初步需求、产品特性和整体价值。引导他们反思某个选择是否真正满足他们的需求。请避免重复搜索。如果你认为购买该产品是合适的，点击“click[立即购买]”。否则，给出最合理的下一步行动建议。 | 手动 |
| 结果预测者 (DM) | 作为结果预测者，描述如何根据客户的需求、产品选择和市场趋势预测他们购买决策的潜在满意度和成功。提供见解，说明这些选择如何满足客户的期望。如果你有任何建议，请在“think[…]”中打印。否则，给出最合理的下一步行动建议。 | 生成 |
