<!--yml

分类：未分类

日期：2025-01-11 11:46:31

-->

# TrendSim：基于LLM的多智能体系统在毒化攻击下模拟社交媒体中的热点话题

> 来源：[https://arxiv.org/html/2412.12196/](https://arxiv.org/html/2412.12196/)

张泽宇¹，廉剑勋²，马晨¹，屈燕宁¹，罗烨¹，王磊¹，

李锐¹，陈旭¹，林彦凯¹，吴乐³，谢星²，温继荣¹，

¹中国人民大学，²微软亚洲研究院，

³合肥工业大学

{zeyuzhang,xu.chen}@ruc.edu.cn

###### 摘要

热点话题已经成为现代社交媒体的重要组成部分，吸引用户参与突发事件的讨论。然而，它们也为毒化攻击提供了一个新的渠道，带来了对社会的负面影响。因此，研究这个关键问题并制定有效的防御策略是迫切需要的。在本文中，我们提出了TrendSim，一种基于LLM的多智能体系统，用于模拟社交媒体中热点话题在毒化攻击下的表现。具体而言，我们创建了一个包含时间感知交互机制、集中式信息传播和交互系统的热点话题模拟环境。此外，我们开发了基于LLM的人类智能体来模拟社交媒体中的用户，并提出了基于原型的攻击者来复制毒化攻击。除此之外，我们从多个方面评估了TrendSim的效果，以验证其有效性。基于TrendSim，我们进行模拟实验，研究关于热点话题毒化攻击的四个关键问题，旨在促进社会福利。

TrendSim：基于LLM的多智能体系统在毒化攻击下模拟社交媒体中的热点话题

张泽宇¹，廉剑勋²，马晨¹，屈燕宁¹，罗烨¹，王磊¹，李锐¹，陈旭¹，林彦凯¹，吴乐³，谢星²，温继荣¹，¹中国人民大学，²微软亚洲研究院，³合肥工业大学 {zeyuzhang,xu.chen}@ruc.edu.cn

## 1 引言

热点话题近年来已成为现代社交媒体平台的重要组成部分，指的是在短时间内吸引公众关注和广泛讨论的主题，例如微博热搜¹¹1[https://s.weibo.com/top/summary](https://s.weibo.com/top/summary)和Twitter趋势²²2[https://twitter.com/explore/tabs/trending](https://twitter.com/explore/tabs/trending)。每个热点话题通常包括一个标题、一个描述和大量评论。与传统的社交媒体内容相比，热点话题总是爆发性地出现，并展示在突出部分，以确保用户参与当前的讨论。然而，这些因素也放大了恶意攻击对社交媒体平台用户的影响。这些攻击通常会操纵或扭曲热门话题评论中的信息，误导用户并传播虚假信息。它们可能导致许多负面影响，如误导事实、激化冲突，甚至破坏社会信任，危害社会。然而，这一关键问题仍然没有得到充分研究。

最近，大型语言模型（LLMs）展示了类人能力，Zhao 等人（[2023](https://arxiv.org/html/2412.12196v1#bib.bib25)）；Wang 等人（[2023a](https://arxiv.org/html/2412.12196v1#bib.bib20)）以及一些研究提出了设计基于 LLM 的类人智能体来进行社会模拟，Gao 等人（[2023a](https://arxiv.org/html/2412.12196v1#bib.bib6)，[b](https://arxiv.org/html/2412.12196v1#bib.bib7)）；Kovač 等人（[2023](https://arxiv.org/html/2412.12196v1#bib.bib10)）。通过分析他们的结果，研究人员可以深入洞察人类行为，并为社会福利设计有效的政策，Hua 等人（[2023](https://arxiv.org/html/2412.12196v1#bib.bib8)）。然而，以前的模拟框架存在一些局限，使它们不适合模拟热门话题。首先，大多数框架采用基于回合的交互，未考虑时间因素，Wang 等人（[2023b](https://arxiv.org/html/2412.12196v1#bib.bib21)），尽管热门话题具有高度的时间敏感性。其次，大多数社会模拟设计为点对点交互，其中智能体主要与邻居沟通，Gao 等人（[2023b](https://arxiv.org/html/2412.12196v1#bib.bib7)）。然而，由于热门话题的个体特性，它们的信息传播应像中心枢纽一样进行集中传播。此外，以前的方法很少关注模拟过程中的动态心理状态，且忽视了热门话题中的恶意攻击问题。

为了解决这些局限性，本文提出了一种基于LLM的多智能体系统，名为TrendSim，用于模拟社交媒体中的流行话题，尤其是在中毒攻击下。具体来说，我们的框架设计了时间感知交互机制和集中式信息传播，以适应流行话题场景，并实现多智能体交互系统的细节。此外，我们设计了基于LLM的人类智能体，具备感知、记忆和行动模块，以模拟用户行为并反映其心理状态。而且，我们创建了具有不同目标的基于原型的攻击者，用于生成中毒攻击。我们进行了广泛的评估，从多个方面验证了我们模拟框架的有效性。基于TrendSim，我们研究了社交媒体中流行话题的中毒攻击的四个关键问题，分析了结果并提出了防御建议。我们的工作是首个聚焦于基于LLM的社交模拟中流行话题中毒攻击问题的研究。

然而，作为该新领域的初步研究，我们应当强调我们工作中的一些事实。首先，由于流行话题的实现方式在不同社交媒体平台上有所不同，我们的工作抽象出了一种基于特定假设的通用且合理的实现方式。其次，现实世界中存在着大量不可见的因素，这使得用完全准确的方式模拟所有细节变得不现实。因此，我们的研究旨在提供一个可解释的模拟过程，并在合理假设下得出演化结论，而不是复制现实中的每一个细节。

我们的贡献总结如下：

$\bullet$ 我们提出了一种基于LLM的多智能体系统，名为TrendSim，用于模拟社交媒体中的流行话题，尤其是在中毒攻击下。我们设计了时间感知交互机制、集中式信息传播以及交互式多智能体系统，以模拟社交媒体中的流行话题。

$\bullet$ 我们开发了基于LLM的人类智能体，具备感知、记忆和行动模块，用于模拟社交媒体平台中的用户行为。我们创建了基于原型的攻击者，用于在我们的模拟中生成各种中毒攻击。

$\bullet$ 我们对我们的模拟框架进行了广泛评估。基于TrendSim，我们研究了社交媒体中流行话题的中毒攻击的四个关键问题。

本文其余部分组织如下：首先，我们在第[2](https://arxiv.org/html/2412.12196v1#S2 "2 Related Works ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")节介绍相关工作。然后，我们在第[3](https://arxiv.org/html/2412.12196v1#S3 "3 Methods ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")节展示TrendSim的细节，并在第[4](https://arxiv.org/html/2412.12196v1#S4 "4 Evaluations ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")节进行评估。之后，我们在第[5](https://arxiv.org/html/2412.12196v1#S5 "5 Experiments ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")节基于TrendSim进行模拟实验。最后，我们在第[6](https://arxiv.org/html/2412.12196v1#S6 "6 Conclusion ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")节得出结论，并在本文最后进一步讨论局限性和伦理影响。

![参见图注](img/cedddc5c48ff64b39c559935cf0f4bec.png)

图1：TrendSim框架，用于模拟社交媒体中的热门话题。

## 2 相关工作

### 2.1 社交媒体中的中毒攻击

近年来，社交媒体平台上的中毒攻击逐渐引起了广泛关注 Khurana 等人（[2019](https://arxiv.org/html/2412.12196v1#bib.bib9)）。研究表明，社交媒体是传播诈骗和恶意软件的主要途径，社交媒体平台上发生了大量的中毒攻击 Kunwar 和 Sharma（[2016](https://arxiv.org/html/2412.12196v1#bib.bib11)）。一些先前的研究调查了社交媒体中中毒攻击者的意图和特征 Aïmeur 等人（[2019](https://arxiv.org/html/2412.12196v1#bib.bib1)）；Briscoe 等人（[2014](https://arxiv.org/html/2412.12196v1#bib.bib3)）。他们的行为通常涉及发布攻击性评论和淫秽图片，旨在传播仇恨并实施网络欺凌 Chinivar 等人（[2022](https://arxiv.org/html/2412.12196v1#bib.bib4)）。他们还发现，中毒攻击更倾向于集中在人的脆弱性上 Aïmeur 等人（[2019](https://arxiv.org/html/2412.12196v1#bib.bib1)）。

之前的大多数研究集中在社交网络中的常规内容上，而对现代社交媒体平台上的热门话题关注较少。然而，针对热门话题的中毒攻击已成为一个关键问题，给我们的社会环境带来了重大威胁，亟需研究人员给予更多关注。

### 2.2 基于LLM的多智能体社交模拟

大型语言模型因其出色的语言理解和生成能力，展现出在构建自主代理方面的巨大潜力，Ouyang等人（[2022](https://arxiv.org/html/2412.12196v1#bib.bib14)）；Park等人（[2023](https://arxiv.org/html/2412.12196v1#bib.bib15)）；Wang等人（[2023b](https://arxiv.org/html/2412.12196v1#bib.bib21)）。这些代理通常配备了基于LLM的大量模块来执行复杂任务，Xi等人（[2023](https://arxiv.org/html/2412.12196v1#bib.bib23)）；Shinn等人（[2023](https://arxiv.org/html/2412.12196v1#bib.bib17)）；Zhu等人（[2023](https://arxiv.org/html/2412.12196v1#bib.bib26)）；Wang等人（[2023c](https://arxiv.org/html/2412.12196v1#bib.bib22)）；Qin等人（[2023](https://arxiv.org/html/2412.12196v1#bib.bib16)）。近期的研究提出利用基于LLM的代理进行社交模拟，以模拟不同场景中的人类行为和互动，Wang等人（[2023b](https://arxiv.org/html/2412.12196v1#bib.bib21)）；Park等人（[2023](https://arxiv.org/html/2412.12196v1#bib.bib15)）；Gao等人（[2023b](https://arxiv.org/html/2412.12196v1#bib.bib7)，[a](https://arxiv.org/html/2412.12196v1#bib.bib6)）。例如，$S^{3}$ Gao等人（[2023b](https://arxiv.org/html/2412.12196v1#bib.bib7)）通过基于LLM的代理互动模拟了社交网络的形成及信息传播等现象。RecAgent Wang等人（[2023b](https://arxiv.org/html/2412.12196v1#bib.bib21)）提出了在推荐系统领域模拟用户行为。Generative Agents Park等人（[2023](https://arxiv.org/html/2412.12196v1#bib.bib15)）和AgentSimsLin等人（[2023](https://arxiv.org/html/2412.12196v1#bib.bib13)）创建了作为数字小镇的多代理系统来复制人类的日常生活。

然而，现有的框架在社交媒体中的趋势话题场景下表现不佳，主要是因为它们缺乏对时间的考虑、集中化的信息传播和用户心理状态的反映。因此，我们的工作是首个模拟社交媒体趋势话题并研究其中毒攻击问题的研究。

## 3 方法

### 3.1 TrendSim概述

TrendSim旨在模拟趋势话题的完整生命周期，用户在社交媒体平台上与之互动。在这里，我们将趋势话题表述为一个描述爆炸性事件的公共帖子，用户可以通过评论、回复和其他行为与之互动。趋势话题的生命周期通常从其出现到消失，通常持续不到几个小时。在此期间，用户可以通过评论表达自己的态度，或者通过回复特定评论交换意见。这些互动推动了趋势话题的演变，引发了广泛讨论，但也为攻击者提供了传播中毒攻击的机会。

### 3.2 多代理模拟环境

#### 3.2.1 时间感知的交互机制

与基于回合的仿真不同，我们的框架集成了一种时间感知的交互机制，如图[1](https://arxiv.org/html/2412.12196v1#S1.F1 "图 1 ‣ 1 引言 ‣ TrendSim: 基于大规模语言模型的多智能体系统下模拟社交媒体中的趋势话题的中毒攻击")所示(a)。具体而言，每一轮交互发生在特定的时间戳$T$，并具有持续时间$\Delta t$。所有交互按时间顺序执行，采用动态优先队列，参见van Emde Boas等人（[1976](https://arxiv.org/html/2412.12196v1#bib.bib19)）。例如，如果Alice在下午3:12查看了趋势话题，并花费两分钟进行评论，那么Bob可以在下午3:14看到Alice的评论。我们假设用户访问趋势话题遵循关于时间$t$的某种概率分布$P(t)$。具体而言，$P(t)$在开始时呈指数增长，然后在达到峰值前经历增长减速，最后呈渐进下降直至消失，参见Lerman和Hogg（[2010](https://arxiv.org/html/2412.12196v1#bib.bib12)）。它还确保了连续性的$G_{0}$-平滑性和$G_{1}$-平滑性，参见Barsky和DeRose（[1989](https://arxiv.org/html/2412.12196v1#bib.bib2)）。因此，我们假设$P(t)$为

$P(t)\propto\left\{\begin{aligned} &e^{A(t-T_{m})}&0\leq t<T_{m},\\ &-\alpha A(t-T_{m}-\frac{1}{2\alpha})^{2}+1+\frac{A}{4\alpha}&T_{m}\leq t<T_{m}+\frac{1}{\alpha},\\ &(t-T_{m}-\frac{1}{\alpha}+1)^{-A}&t\geq T_{m}+\frac{1}{\alpha},\end{aligned}\right.$ 其中$A,T_{m},\alpha$为超参数。由于页面限制，更多细节可在附录[A](https://arxiv.org/html/2412.12196v1#A1 "附录 A 时间机制的详细信息 ‣ TrendSim: 基于大规模语言模型的多智能体系统下模拟社交媒体中的趋势话题的中毒攻击")中找到，以便更好地说明。

为了提高仿真效率，我们首先根据仿真开始前的分布对用户访问的第一次进行采样，并在每次访问结束时动态采样下一次访问时间。

#### 3.2.2 集中式消息传播

传统的社交媒体消息通常通过社交网络中的用户关系进行传播。然而，现代社交媒体平台通常部署一个单独的版块来突出显示趋势话题，使得消息传播通过一个集中的中心，而不是点对点的网络。例如，微博热搜提供一个实时的前50名热门话题标签列表，作为趋势话题，所有用户都可以直接从首页访问。因此，我们实现了集中的消息传播机制来模拟趋势话题。

具体来说，TrendSim 有三种主要的消息传播方式，如图 [1](https://arxiv.org/html/2412.12196v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")(b) 所示。首先，每个趋势话题都有一个公共描述，包括标题、摘要和完整内容，这些内容对社交媒体中的所有用户可见。其次，TrendSim 允许用户在趋势话题下发表评论，这些评论随后可以被其他人查看。最后，TrendSim 允许用户回复任何评论，这些回复也对其他用户可见。通过实施这三种机制，用户能够在 TrendSim 上获取、发布并交换关于趋势话题的信息。

#### 3.2.3 用户-环境互动系统

我们设计了一个用户与趋势话题之间的互动系统。它定义了用户的观察空间、行动空间和转换机制，展示了用户的行为如何影响趋势话题。具体来说，用户可以看到趋势话题的三个不同页面，如图 [1](https://arxiv.org/html/2412.12196v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")(c) 所示：

$\bullet$ 浏览页面：用户查看趋势话题的标题和摘要，然后选择查看详情（返回到主页）或离开社交媒体（结束本次会话）。

$\bullet$ 主页：用户查看标题、完整内容和前 $k$ 条评论。然后，用户可以选择点赞、评论、转发、查看更多评论（获取下一组 $k$ 条评论）、查看特定评论的详情（返回到评论页面）或离开（结束本次会话）等操作。

$\bullet$ 评论页面：用户查看特定评论及其前 $k$ 条回复，然后采取点赞、回复或返回（返回到主页）等操作。

此外，用户的行为也会影响环境。具体来说，对趋势话题发表评论或回复评论可以留下影响其他用户后续观察的信息。此外，点赞趋势话题可以增加其受欢迎度，评论的点赞数决定了其排名。为了避免进行无限次的操作，我们假设每个会话的最大互动次数。

### 3.3 基于 LLM 的用户代理

我们设计了基于 LLM 的智能体，用于模拟社交媒体中的用户。根据人类的认知过程，Solso 和 Kagan（[1979](https://arxiv.org/html/2412.12196v1#bib.bib18)）提出的理论，我们设计了感知、记忆和行动模块（见图[2](https://arxiv.org/html/2412.12196v1#S3.F2 "Figure 2 ‣ 3.3 LLM-based User Agent ‣ 3 Methods ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")），以模仿人类行为并反映心理状态。

![参考说明](img/d118e0d42366df3f3496e2427b7ee5c6.png)

图2：TrendSim 中基于 LLM 的用户智能体框架。

#### 3.3.1 感知模块

当浏览一个热门话题的内容时，用户总是在思考和行动之前形成印象，Solso 和 Kagan（[1979](https://arxiv.org/html/2412.12196v1#bib.bib18)）指出。因此，我们为智能体设计了一个感知模块，用于模拟这一过程。具体来说，我们将感知过程定义为

|  | $I\leftarrow Perception(f,O,M),$ |  |
| --- | --- | --- |

其中，$I$ 代表印象，$O$ 表示原始观察，$M$ 表示记忆，$f$ 使用大型语言模型（LLM）实现。在此过程中，智能体可以根据其记忆在生成的印象中表现出不同的关注点，类似于社交媒体中的不同用户。例如，一个乐观的用户倾向于关注积极的方面，而一个刚刚分手的悲伤用户更有可能产生负面印象。

#### 3.3.2 记忆模块

记忆是社交模拟中类人智能体的一个重要组成部分，负责区分不同用户的个性，Zhang 等（[2024](https://arxiv.org/html/2412.12196v1#bib.bib24)）指出。它还会在模拟过程中动态地影响用户行为。因此，根据认知心理学的理论，我们设计了一个包含三层次的记忆模块：长期记忆、短期记忆和闪存记忆。

长期记忆包含用户一生经历的总结（即个人档案），并且在模拟过程中保持不变。具体来说，我们通过从现实世界的社交媒体平台提取用户的帖子，来实现智能体的长期记忆，模拟开始前进行提炼，之后

|  | $m_{l}\leftarrow LTM(f,\{p_{1},p_{2},...,p_{N}\}),$ |  |
| --- | --- | --- |

其中，$m_{l}$ 表示长期记忆，$\{p_{1},p_{2},...,p_{N}\}$ 是现实世界用户的帖子。

短期记忆旨在保持用户心理状态在模拟过程中的动态变化。我们利用以下三个主要方面来建模短期记忆。

$\bullet$ 情感：用户浏览当前内容时的直接情感体验。

$\bullet$ 看法：用户在参与当前话题后的个人态度。

$\bullet$ 社会信任：用户个人相信社会公正的信念。

我们设计了一个反射过程，在每次交互后动态更新短期记忆。具体来说，我们有

|  | $m_{s}\leftarrow Reflection(f,I,A,m_{s}),$ |  |
| --- | --- | --- |

其中，$m_{s}$表示短期记忆，$A$表示用户的行动。闪存是记忆中最直接的部分，储存观察到的印象。具体来说，我们将当前交互的闪存记忆设置为$m_{f}=I$。总结来说，基于LLM的用户代理的记忆定义为$M=(m_{l},m_{s},m_{f})$，它影响用户的行为。

#### 3.3.3 行动模块

基于记忆，我们的用户代理可以根据他们的观察生成相应的行动。具体来说，我们通过LLMs实现了行动模块：

|  | $A\leftarrow Action(f,O,M).$ |  |
| --- | --- | --- |

通过行动模块，用户代理可以从行动空间中采取行动，进而影响热门话题。因此，用户的消息可以在社交媒体上被其他用户共享。

### 3.4 基于原型的攻击者代理

为了研究热门话题中的毒化攻击，我们开发了基于原型的攻击者代理，用于生成特定话题的恶意评论。具体来说，他们根据预定义的原型和当前观察，通过LLMs生成毒化评论：

|  | $\tilde{A}\leftarrow Action(f,P,O),$ |  |
| --- | --- | --- |

其中，$\tilde{A}$表示恶意评论，$P$是来自特定攻击目标的原型。此外，我们根据攻击目标将攻击者分为三种类型，如下所示。

#### 3.4.1 反社会攻击者

反社会攻击者旨在通过传播反社会评论来削弱用户的社交信心。例如，他们常常挑起用户与社会之间的冲突，制造不和。

#### 3.4.2 网络喷子攻击者

网络喷子攻击者通过发布冒犯性内容来挑衅、激怒或骚扰其他用户。他们还通过贬低他人，制造不同群体之间的冲突，常常针对性别和价值观等问题。

#### 3.4.3 谣言攻击者

谣言攻击者专注于生成和传播关于热门话题的谣言，以掩盖真相。他们故意制造对特定事件和个人的误解。

## 4 评估

### 4.1 模拟设置

基于TrendSim，我们在社交媒体上的热门话题下进行毒化攻击的模拟实验。我们从1000名真实社交媒体平台的公共用户收集数据，对他们的个人资料进行匿名化处理，并从历史帖子中总结信息（详细信息见附录[E](https://arxiv.org/html/2412.12196v1#A5 "Appendix E Details of Data Collection ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")）。我们选择了10个热门话题，涵盖了不同的领域和情感。我们控制攻击者在总参与者中的比例，以模拟不同程度的毒化攻击。由于大部分语料库为中文，我们在模拟中使用了GLM-3-turbo Du等人（[2022](https://arxiv.org/html/2412.12196v1#bib.bib5)）作为基础模型$f$。对于每个热门话题，我们假设最大生命周期为16小时。

我们在本节中从多个方面对TrendSim进行了评估，并在第[5节](https://arxiv.org/html/2412.12196v1#S5 "5 Experiments ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")收集并分析了实验结果。

### 4.2 用户代理评估

对于用户代理，我们的目标是评估它们作为现实世界用户的能力。具体而言，我们利用LLM（大语言模型）来评分用户特征与表达之间的一致性，评分范围从0到1（见附录 [G.1](https://arxiv.org/html/2412.12196v1#A7.SS1 "G.1 Evaluation on User Agent ‣ Appendix G Details of LLM Evaluation ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")）。我们的指标包括：(1) 行为一致性：用户特征与行为之间的一致性。(2) 心理一致性：用户特征与心理状态之间的一致性。我们通过设计提示词来模拟用户，使用多个LLM作为基准，并招募一位人工专家作为另一个基准（见附录 [F.1](https://arxiv.org/html/2412.12196v1#A6.SS1 "F.1 Baselines of User Agents ‣ Appendix F Details of Evaluation Baselines ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")）。

表 1：用户代理评估结果。

| 方法 | 行为一致性    \bigstrut |
| --- | --- |
| GPT-4 | GLM-4 | Llama-3 | 平均值 \bigstrut |
| --- | --- | --- | --- |
| GPT-4 | 0.925 | 0.950 | 0.830 | 0.902 \bigstrut[t] |
| GLM-4 | 0.940 | 0.965 | 0.842 | 0.916 |
| Llama-3 | 0.944 | 0.930 | 0.826 | 0.900 |
| TrendSim | 0.948 | 0.945 | 0.853 | 0.915 |
| 人类 | 0.935 | 0.950 | 0.828 | 0.904 \bigstrut[b] |
| 方法 | 心理一致性    \bigstrut |
| GPT-4 | GLM-4 | Llama-3 | 平均值 \bigstrut |
| GPT-4 | 0.745 | 0.767 | 0.760 | 0.757 \bigstrut[t] |
| GLM-4 | 0.930 | 0.776 | 0.767 | 0.824 |
| Llama-3 | 0.705 | 0.640 | 0.768 | 0.704 |
| TrendSim | 0.960 | 0.745 | 0.773 | 0.826 |
| 人类 | 0.910 | 0.753 | 0.794 | 0.819 \bigstrut[b] |

结果见表[1](https://arxiv.org/html/2412.12196v1#S4.T1 "Table 1 ‣ 4.2 Evaluation on User Agent ‣ 4 Evaluations ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")，由于页面限制，我们将误差条放在附录[B.1](https://arxiv.org/html/2412.12196v1#A2.SS1 "B.1 Error Bars of the Evaluation on User Agent ‣ Appendix B More results of Evaluations ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")中。我们发现TrendSim在大多数情况下表现优异，展示了其在模拟社交媒体中流行话题的能力。我们还观察到，在我们的场景中，人类可能不擅长扮演他人角色。然而，不同LLM评估器之间的结果可能会有所不同。

### 4.3 攻击代理评估

对于攻击代理，我们从两个方面评估它们生成的恶意评论：（1）一致性：评论与所查看内容之间的相关性；（2）隐蔽性：逃避恶意检测的能力。具体细节请参见附录[G.2](https://arxiv.org/html/2412.12196v1#A7.SS2 "G.2 Evaluation on Attacker Agent ‣ Appendix G Details of LLM Evaluation ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")。我们还利用LLM和人类作为基线（参见附录[F.2](https://arxiv.org/html/2412.12196v1#A6.SS2 "F.2 Baselines of Attacker Agents ‣ Appendix F Details of Evaluation Baselines ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")）。结果见表[2](https://arxiv.org/html/2412.12196v1#S4.T2 "Table 2 ‣ 4.3 Evaluation on Attacker Agent ‣ 4 Evaluations ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")，误差条请参见附录[B.2](https://arxiv.org/html/2412.12196v1#A2.SS2 "B.2 Error Bars of the Evaluation on Attacker Agent ‣ Appendix B More results of Evaluations ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")。

表2：攻击代理评估结果。

| 方法 | 一致性    \bigstrut |
| --- | --- |
| GPT-4 | GLM-4 | Llama-3 | 平均值 \bigstrut |
| --- | --- | --- | --- |
| GPT-4 | 0.368 | 0.675 | 0.744 | 0.596 \bigstrut[t] |
| GLM-4 | 0.435 | 0.644 | 0.755 | 0.611 |
| Llama-3 | 0.320 | 0.475 | 0.735 | 0.510 |
| TrendSim | 0.815 | 0.905 | 0.792 | 0.837 |
| 人类 | 0.575 | 0.540 | 0.784 | 0.633 \bigstrut[b] |
| 方法 | 隐蔽性    \bigstrut |
| GPT-4 | GLM-4 | Llama-3 | 平均值 \bigstrut |
| GPT-4 | 0.342 | 0.380 | 0.325 | 0.349 \bigstrut[t] |
| GLM-4 | 0.445 | 0.370 | 0.317 | 0.377 |
| Llama-3 | 0.475 | 0.370 | 0.291 | 0.379 |
| TrendSim | 0.770 | 0.449 | 0.335 | 0.518 |
| 人类 | 0.756 | 0.453 | 0.340 | 0.516 \bigstrut[b] |

我们发现基于原型的攻击者在大多数情况下表现优于其他攻击者，这表明我们的机制是有效的。此外，这也表明，来自LLM和智能体的中毒攻击可能比人类攻击者更为严重，并且更难被检测出来。

### 4.4 多智能体系统评估

除了从单一智能体的视角评估用户智能体和攻击者智能体外，我们还从互动系统的角度对多智能体系统进行评估。我们重点关注两个指标：（1）理性：评论中讨论的理性。（2）多样性：不同用户之间的差异性。详细内容见附录 [G.3](https://arxiv.org/html/2412.12196v1#A7.SS3 "G.3 Evaluation on Multi-agent System ‣ Appendix G Details of LLM Evaluation ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")。结果如表 [3](https://arxiv.org/html/2412.12196v1#S4.T3 "Table 3 ‣ 4.4 Evaluation on Multi-agent System ‣ 4 Evaluations ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System") 所示，误差条见附录 [B.3](https://arxiv.org/html/2412.12196v1#A2.SS3 "B.3 Error Bars of the Evaluation on Multi-agent System ‣ Appendix B More results of Evaluations ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")。结果显示，TrendSim表现优秀，理性得分大多数高于0.8，多样性得分大多数高于0.7。

表 3：多智能体系统评估结果。

| 情感 | 理性    \bigstrut |
| --- | --- |
| GPT-4 | GLM-4 | Llama-3 | 平均 \bigstrut |
| 积极 | 0.865 | 0.830 | 0.840 | 0.845 \bigstrut[t] |
| 消极 | 0.775 | 0.725 | 0.775 | 0.758 |
| 中立 | 0.815 | 0.815 | 0.817 | 0.816 |
| 全部 | 0.818 | 0.790 | 0.811 | 0.806 |
| 情感 | 多样性    \bigstrut[b] |
| GPT-4 | GLM-4 | Llama-3 | 平均 \bigstrut |
| 积极 | 0.785 | 0.770 | 0.835 | 0.797 \bigstrut[t] |
| 消极 | 0.746 | 0.760 | 0.805 | 0.770 |
| 中立 | 0.685 | 0.765 | 0.744 | 0.731 |
| 全部 | 0.739 | 0.765 | 0.795 | 0.766 \bigstrut[b] |

### 4.5 仿真效率评估

仿真效率对研究人员至关重要，因为效率的提高能减少时间成本，并增强处理更大用户群体的能力。因此，我们评估了在 TrendSim 上进行仿真的时间成本。所有实验均在 Intel^® Xeon^® Gold-5118（48 核）CPU 和 GLM-3-turbo API 下进行。结果如表[4](https://arxiv.org/html/2412.12196v1#S4.T4 "Table 4 ‣ 4.5 Evaluation on Simulation Efficiency ‣ 4 Evaluations ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")所示，我们发现仿真一个包含 1,000 名参与者的热门话题大约需要 16 小时。这为未来采用并行技术进一步提高效率提供了希望。

表 4：不同参与代理数量的时间成本参考。SE、PA-10、PA-30 和 PA-50 分别表示有 0%、10%、30% 和 50% 攻击者代理的仿真。

| Degree | Time Cost (hours)    \bigstrut |
| --- | --- |
| # 10 | # 50 | # 100 | # 200 | # 500 | # 1000 \bigstrut |
| SE | 0.2 | 0.7 | 1.6 | 3.3 | 6.5 | 16 \bigstrut[t] |
| PA-10 | 0.1 | 0.6 | 1.5 | 2.8 | 6.1 | 15 |
| PA-30 | 0.8 $\times 10^{-1}$ | 0.4 | 1.2 | 2.2 | 4.9 | 12 |
| PA-50 | 0.6 $\times 10^{-1}$ | 0.3 | 1.0 | 1.7 | 4.0 | 8.0 \bigstrut[b] |

## 5 个实验

基于TrendSim，我们进行仿真实验并分析其结果。我们关注社交媒体中热门话题的中毒攻击的四个关键问题。

问题 1：中毒攻击对热门话题产生了什么负面影响？

表 5：中毒攻击对热门话题的负面影响仿真实验结果。Average 表示单个热门话题中所有用户条件的均值，Divergence 表示单个热门话题中所有用户条件的标准差。它们的均值和标准差（由$\pm$表示）是在该组所有热门话题上计算的。

| Groups | Degrees | 情感 | 社交信心    \bigstrut |
| --- | --- | --- | --- |
|  |  | 平均值 | 发散度 | 平均值 | 发散度 \bigstrut |
| --- | --- | --- | --- | --- | --- |
| Positive | SE | 0.886$\pm$0.057 | 0.140$\pm$0.040 | 0.905$\pm$0.040 | 0.109$\pm$0.031 \bigstrut[t] |
|  | PA-10 | 0.812$\pm$0.145 | 0.151$\pm$0.028 | 0.836$\pm$0.126 | 0.132$\pm$0.034 |
|  | PA-30 | 0.819$\pm$0.081 | 0.180$\pm$0.024 | 0.845$\pm$0.068 | 0.152$\pm$0.030 |
|  | PA-50 | 0.813$\pm$0.048 | 0.190$\pm$0.017 | 0.836$\pm$0.035 | 0.168$\pm$0.015 \bigstrut[b] |
| Negative | SE | 0.443$\pm$0.002 | 0.065$\pm$0.011 | 0.457$\pm$0.004 | 0.078$\pm$0.011 \bigstrut[t] |
|  | PA-10 | 0.429$\pm$0.000 | 0.066$\pm$0.005 | 0.441$\pm$0.009 | 0.081$\pm$0.005 |
|  | PA-30 | 0.430$\pm$0.008 | 0.067$\pm$0.009 | 0.443$\pm$0.020 | 0.084$\pm$0.009 |
|  | PA-50 | 0.442$\pm$0.007 | 0.071$\pm$0.002 | 0.457$\pm$0.023 | 0.083$\pm$0.004 \bigstrut[b] |
| Netural | SE | 0.525$\pm$0.083 | 0.120$\pm$0.056 | 0.570$\pm$0.122 | 0.123$\pm$0.049 \bigstrut[t] |
|  | PA-10 | 0.509$\pm$0.078 | 0.114$\pm$0.057 | 0.550$\pm$0.117 | 0.120$\pm$0.049 |
|  | PA-30 | 0.509$\pm$0.073 | 0.117$\pm$0.053 | 0.552$\pm$0.110 | 0.121$\pm$0.048 |
|  | PA-50 | 0.490$\pm$0.058 | 0.104$\pm$0.048 | 0.523$\pm$0.091 | 0.117$\pm$0.049 \bigstrut[b] |
| All | SE | 0.653$\pm$0.203 | 0.117$\pm$0.052 | 0.681$\pm$0.204 | 0.109$\pm$0.041 \bigstrut[t] |
|  | PA-10 | 0.614$\pm$0.194 | 0.119$\pm$0.051 | 0.643$\pm$0.196 | 0.117$\pm$0.042 |
|  | PA-30 | 0.617$\pm$0.181 | 0.132$\pm$0.057 | 0.647$\pm$0.185 | 0.126$\pm$0.044 |
|  | PA-50 | 0.610$\pm$0.174 | 0.131$\pm$0.059 | 0.635$\pm$0.177 | 0.131$\pm$0.046 \bigstrut[b] |

如我们之前所讨论的，毒化攻击可以通过社交媒体中的趋势话题产生负面影响。因此，我们打算通过TrendSim验证和分析这一现象。具体来说，我们研究仿真过程中用户心理状态的变化，并将其在四种攻击级别下进行比较。心理状态的度量包括我们在第[3.3.2](https://arxiv.org/html/2412.12196v1#S3.SS3.SSS2 "3.3.2 Memory Module ‣ 3.3 LLM-based User Agent ‣ 3 Methods ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")节中介绍的情感和社交信心，数值范围从0到1。攻击的级别包括：(1) SE：未注入攻击者。(2) PA-10：注入10%的攻击者。(3) PA-30：注入30%的攻击者。(4) PA-50：注入50%的攻击者。这些攻击者根据第[3.4](https://arxiv.org/html/2412.12196v1#S3.SS4 "3.4 Prototype-based Attacker Agent ‣ 3 Methods ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")节的介绍进行注入。此外，我们将趋势话题分类为三种情感群体，包括积极、消极和中立。

我们关注仿真结束时的情况，计算用户之间的平均条件和差异。结果见表[5](https://arxiv.org/html/2412.12196v1#S5.T5 "Table 5 ‣ 5 Experiments ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")。我们发现，大多数毒化攻击对社交媒体中的用户产生了负面影响，但其效果在不同群体中有所不同。与负面和中立群体相比，积极群体受到的影响最大，可能是因为毒化评论与正常评论之间的对比更为明显。此外，我们发现结果与攻击的强度并不成正比，较少的攻击者可能会造成更大的影响。

问题2：用户的心理状态是如何随时间动态变化的？

我们进一步研究了用户心理状态随时间的动态变化。具体来说，我们沿时间线追踪用户的心理状态，并绘制了图[3](https://arxiv.org/html/2412.12196v1#S5.F3 "Figure 3 ‣ 5 Experiments ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")中的曲线。为了更好地展示，我们将结果与SE中的数据进行了对比。我们发现，在时间的中间，心理状态出现了急剧下降，这也是大量用户同时进入的时刻。此外，PA-50在整个过程中表现出了最负面的影响。由于页面限制，更多不同组别的结果请见附录[C.1](https://arxiv.org/html/2412.12196v1#A3.SS1 "C.1 Results of Different Groups in Problem 2 ‣ Appendix C More Results of Simulation Experiments ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")。

![参见说明文字](img/8bcdfff5f88da44ae0eb6c2b7603dd62.png)![参见说明文字](img/938df16f7feca0249ed5f8a6bdc17058.png)

图3：社交媒体平台中用户心理状态随时间的变化。曲线表示均值，阴影表示标准差。

问题3：哪些类型的用户在趋势话题中更容易受到投毒攻击？

我们进一步研究了哪些类型的用户更容易受到投毒攻击。具体来说，我们根据用户的偏好将这1,000个用户分为六个组别，包括娱乐、体育、生活方式、社会、文化和技术。我们在图[4](https://arxiv.org/html/2412.12196v1#S5.F4 "Figure 4 ‣ 5 Experiments ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")(a)中展示了每种类型的比例。结果显示，娱乐组和社会组是两个主要群体。

模拟结果显示在图[4](https://arxiv.org/html/2412.12196v1#S5.F4 "Figure 4 ‣ 5 Experiments ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")(b)和图[4](https://arxiv.org/html/2412.12196v1#S5.F4 "Figure 4 ‣ 5 Experiments ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")(c)中。我们发现，对社会话题感兴趣的人群最容易受到投毒攻击，这与我们的直觉一致。然而，令我们惊讶的是，相比其他组别，娱乐组受投毒攻击的影响最小。

![参见说明文字](img/313a9eca02a7e42e5f788a579a73efde.png)

(a) 用户组分布。

![参见说明文字](img/c2b27c5d6f8acf1631268ce7efde830f.png)

(b) 情感。

![参见说明文字](img/9c126bc00f2e5e5a1a607007d34acfe8.png)

(c) 社会信心。

图4：不同组别模拟实验中的用户心理状态。

问题4：内容审查在防御中毒攻击中的效果如何？

为了减轻中毒攻击的负面影响，社交媒体平台通常会设计防御策略来应对恶意内容。最常见的保护方法之一是内容审查，其目的是检测和过滤中毒评论。在这一部分，我们进一步开展模拟实验，评估内容审查在防御中毒攻击中的有效性。具体来说，我们运行了由LLM实施内容审查的PA-50模拟实验。结果见附录[C.2](https://arxiv.org/html/2412.12196v1#A3.SS2 "C.2 Results of Simulations with Content Censorship in Problems 4 ‣ Appendix C More Results of Simulation Experiments ‣ TrendSim: Simulating Trending Topics in Social Media Under Poisoning Attacks with LLM-based Multi-agent System")。我们发现，内容审查机制在大多数情况下能够有效减轻中毒攻击的负面影响。

## 6 结论

在本文中，我们提出了一种基于LLM的多智能体系统，用于模拟社交媒体在中毒攻击下的趋势话题，命名为TrendSim。通过实现时间感知交互机制、集中式消息传播和交互系统，我们开发了一个模拟趋势话题的环境。我们还提出了基于LLM的人类智能体来模仿用户，并设计了基于原型的攻击者来模拟中毒攻击。我们的评估展示了TrendSim的有效性和效率。基于此，我们进行了模拟实验，研究了关于中毒攻击对趋势话题的四个关键问题，并得出了相关结论。

我们希望我们的工作能够为社会做出贡献，从而实现人工智能的温暖和负责任的使用。在未来的工作中，专注于LLM基础的社交模拟中的记忆机制是非常有前景的，因为这是角色扮演和个性化的关键部分。同时，实施更大规模的社交模拟以应用于各种场景也是非常有价值的。

## 限制

在这项工作中，我们提出了TrendSim，用于模拟社交媒体中受污染攻击下的趋势话题。然而，仍然存在一些局限性。首先，TrendSim目前仅以文本形式进行模拟，没有考虑社交媒体中重要的多模态信息。例如，虚假的图片也可能导致趋势话题中的污染攻击。其次，像以前的社会模拟工作一样，TrendSim也基于一系列假设。这些假设源于现实世界中未被观察到的因素，研究人员无法完全考虑这些因素。因此，由于这些假设的存在，我们的模拟可能存在一些偏差，限制了其准确复制所有现实世界细节的能力。这种对齐问题在其他社会模拟应用中也存在。

## 伦理影响

TrendSim有助于揭示社交媒体趋势中的污染攻击，为防御机制的发展提供了有益的启示。然而，任何事物都有两面性。TrendSim的可用性可能无意中促使攻击者的进化，攻击者可能会根据社交媒体平台上采用的防御机制进行适应和进化。

## 参考文献

+   Aïmeur et al. (2019) Esma Aïmeur, Nicolás Díaz Ferreyra, and Hicham Hage. 2019. 操控与恶意个性化：探索欺骗性攻击者在社交媒体上利用的自我披露偏见。*人工智能前沿*，2:26。

+   Barsky and DeRose (1989) Brian A Barsky and Tony D DeRose. 1989. 参数曲线的几何连续性：三种等效表述。*IEEE计算机图形与应用*，9(6)：60–69。

+   Briscoe et al. (2014) Erica J Briscoe, D Scott Appling, and Heather Hayes. 2014. 社交媒体沟通中的欺骗线索。发表于*2014年第47届夏威夷国际系统科学会议*，第1435–1443页。IEEE。

+   Chinivar et al. (2022) Sneha Chinivar, MS Roopa, JS Arunalatha, and KR Venugopal. 2022. 社交媒体中的在线攻击性行为：检测方法、综合回顾与未来方向。*娱乐计算*，第100544页。

+   Du et al. (2022) Zhengxiao Du, Yujie Qian, Xiao Liu, Ming Ding, Jiezhong Qiu, Zhilin Yang, and Jie Tang. 2022. Glm: 具有自回归空白填充的通用语言模型预训练。发表于*第60届计算语言学协会年会论文集（第1卷：长篇论文）*，第320–335页。

+   Gao et al. (2023a) Chen Gao, Xiaochong Lan, Nian Li, Yuan Yuan, Jingtao Ding, Zhilun Zhou, Fengli Xu, and Yong Li. 2023a. 大型语言模型赋能的基于代理的建模与仿真：一项调查与展望。*arXiv预印本 arXiv:2312.11970*。

+   Gao et al. (2023b) Chen Gao, Xiaochong Lan, Zhihong Lu, Jinzhu Mao, Jinghua Piao, Huandong Wang, Depeng Jin, and Yong Li. 2023b. S³: Social-network simulation system with large language model-empowered agents. *arXiv预印本 arXiv:2307.14984*。

+   Hua 等人（2023）Wenyue Hua、Lizhou Fan、Lingyao Li、Kai Mei、Jianchao Ji、Yingqiang Ge、Libby Hemphill 和 Yongfeng Zhang。2023年。《战争与和平（waragent）：基于大型语言模型的世界大战多智能体仿真》。*arXiv 预印本 arXiv:2311.17227*。

+   Khurana 等人（2019）Nitika Khurana、Sudip Mittal、Aritran Piplai 和 Anupam Joshi。2019年。《防止针对基于AI的威胁情报系统的中毒攻击》。在 *2019 IEEE 第29届国际机器学习信号处理研讨会（MLSP）* 上，页码1–6。IEEE。

+   Kovač 等人（2023）Grgur Kovač、Rémy Portelas、Peter Ford Dominey 和 Pierre-Yves Oudeyer。2023年。《SocialAI 学校：从发展心理学到人工社会文化智能体的见解》。*arXiv 预印本 arXiv:2307.07871*。

+   Kunwar 和 Sharma（2016）Rakesh Singh Kunwar 和 Priyanka Sharma。2016年。《社交媒体：网络攻击的新载体》。在 *2016年国际计算、通信与自动化进展大会（ICACCA）（春季）* 上，页码1–5。IEEE。

+   Lerman 和 Hogg（2010）Kristina Lerman 和 Tad Hogg。2010年。《使用社会动态模型预测新闻的受欢迎程度》。在 *第19届国际万维网大会论文集* 中，页码621–630。

+   Lin 等人（2023）Jiaju Lin、Haoran Zhao、Aochi Zhang、Yiting Wu、Huqiuyue Ping 和 Qin Chen。2023年。《Agentsims：一个开源沙盒用于大型语言模型评估》。*arXiv 预印本 arXiv:2308.04026*。

+   Ouyang 等人（2022）Long Ouyang、Jeffrey Wu、Xu Jiang、Diogo Almeida、Carroll Wainwright、Pamela Mishkin、Chong Zhang、Sandhini Agarwal、Katarina Slama、Alex Ray 等人。2022年。《训练语言模型以遵循指令并获取人类反馈》。*神经信息处理系统进展*，35:27730–27744。

+   Park 等人（2023）Joon Sung Park、Joseph C. O’Brien、Carrie J. Cai、Meredith Ringel Morris、Percy Liang 和 Michael S. Bernstein。2023年。《生成型智能体：人类行为的交互式仿真体》。在 *第36届ACM用户界面软件与技术年会（UIST ’23）* 上，UIST ’23，纽约，美国。计算机协会。

+   Qin 等人（2023）Yujia Qin、Shihao Liang、Yining Ye、Kunlun Zhu、Lan Yan、Yaxi Lu、Yankai Lin、Xin Cong、Xiangru Tang、Bill Qian 等人。2023年。《ToolLLM：帮助大型语言模型掌握16000+种真实世界API》。*arXiv 预印本 arXiv:2307.16789*。

+   Shinn 等人（2023）Noah Shinn、Federico Cassano、Beck Labash、Ashwin Gopinath、Karthik Narasimhan 和 Shunyu Yao。2023年。《Reflexion: 通过语言强化学习的语言智能体》。*arXiv 预印本 arXiv:2303.11366*。

+   Solso 和 Kagan（1979）Robert L Solso 和 Jerome Kagan。1979年。《*认知心理学*》。霍顿·米夫林·哈考特出版。

+   van Emde Boas 等人（1976）Peter van Emde Boas、Robert Kaas 和 Erik Zijlstra。1976年。《高效优先队列的设计与实现》。*数学系统理论*，10(1):99–127。

+   王等人（2023a） Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin, 等人. 2023a. 基于大语言模型的自主智能体调查。*arXiv 预印本 arXiv:2308.11432*。

+   王等人（2023b） Lei Wang, Jingsen Zhang, Hao Yang, Zhiyuan Chen, Jiakai Tang, Zeyu Zhang, Xu Chen, Yankai Lin, Ruihua Song, Wayne Xin Zhao, Jun Xu, Zhicheng Dou, Jun Wang, 和 Ji-Rong Wen. 2023b. [当基于大语言模型的智能体遇到用户行为分析：一种新的用户模拟范式](https://arxiv.org/abs/2306.02552). *预印本*, arXiv:2306.02552。

+   王等人（2023c） Zihao Wang, Shaofei Cai, Anji Liu, Xiaojian Ma, 和 Yitao Liang. 2023c. 描述、解释、计划与选择：基于大语言模型的交互式规划使开放世界多任务智能体成为可能。*arXiv 预印本 arXiv:2302.01560*。

+   习等人（2023） Zhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, Boyang Hong, Ming Zhang, Junzhe Wang, Senjie Jin, Enyu Zhou, 等人. 2023. 大语言模型智能体的崛起与潜力：一项调查。*arXiv 预印本 arXiv:2309.07864*。

+   张等人（2024） Zeyu Zhang, Xiaohe Bo, Chen Ma, Rui Li, Xu Chen, Quanyu Dai, Jieming Zhu, Zhenhua Dong, 和 Ji-Rong Wen. 2024. 基于大语言模型智能体的记忆机制调查。*arXiv 预印本 arXiv:2404.13501*。

+   赵等人（2023） Wayne Xin Zhao, Kun Zhou, Junyi Li, Tianyi Tang, Xiaolei Wang, Yupeng Hou, Yingqian Min, Beichen Zhang, Junjie Zhang, Zican Dong, 等人. 2023. 大语言模型的调查。*arXiv 预印本 arXiv:2303.18223*。

+   朱等人（2023） Xizhou Zhu, Yuntao Chen, Hao Tian, Chenxin Tao, Weijie Su, Chenyu Yang, Gao Huang, Bin Li, Lewei Lu, Xiaogang Wang, 等人. 2023. 我的世界中的鬼魂：通过大语言模型结合文本知识和记忆，为开放世界环境提供普适智能体。*arXiv 预印本 arXiv:2305.17144*。

## 附录 A 时间机制的细节

对于时间感知交互系统中热门话题的生命周期，我们将其划分为三个阶段，这与社交媒体平台中的实际模式一致。在第一个阶段，社会关注度和用户进入量出现爆炸性增长，因此我们用指数函数来建模这一阶段。在第二个阶段，用户浏览量的增长速度放缓，并逐渐开始下降。对于这一阶段，我们采用具有正指数的幂函数进行建模。在最后一个阶段，热门话题的关注度逐渐减退，我们使用具有负指数的幂函数来建模这一阶段。为了根据不同热门帖子的吸引力动态调整曲线，我们为其设置了不同的超参数，以多样化其函数。此外，我们还设计了带有$G_{0}$-平滑和$G_{1}$-平滑的概率分布函数。用户对热门话题的概率分布是

$P(t)\propto\left\{\begin{aligned} &e^{A(t-T_{m})}&0\leq t<T_{m},\\ &-\alpha A(t-T_{m}-\frac{1}{2\alpha})^{2}+1+\frac{A}{4\alpha}&T_{m}\leq t<T_{m% }+\frac{1}{\alpha},\\ &(t-T_{m}-\frac{1}{\alpha}+1)^{-A}&t\geq T_{m}+\frac{1}{\alpha},\end{aligned}\right.$

其中 $A$ 是一个参数，用于反映趋势主题的断裂程度，$\alpha$ 和 $T_{m}$ 是两个超参数，可以调节曲线。然后，我们证明该函数是 $G_{0}$-平滑和 $G_{1}$-平滑的。

###### 定理 1.

函数 $P(t)$ 是 $G_{0}$-平滑的。

###### 证明。

给定上述的概率分布，我们记

$f(t)=\left\{\begin{aligned} &e^{A(t-T_{m})}&0\leq t<T_{m},\\ &-\alpha A(t-T_{m}-\frac{1}{2\alpha})^{2}+1+\frac{A}{4\alpha}&T_{m}\leq t<T_{m% }+\frac{1}{\alpha},\\ &(t-T_{m}-\frac{1}{\alpha}+1)^{-A}&t\geq T_{m}+\frac{1}{\alpha}.\end{aligned}\right.$

所以我们可以将函数 $P(t)$ 重写为 $P(t)=\frac{1}{S}\cdot f(t)$，即

$P(t)=\left\{\begin{aligned} &\frac{1}{S}\cdot e^{A(t-T_{m})}&0\leq t<T_{m},\\ &\frac{1}{S}\cdot\left[-\alpha A(t-T_{m}-\frac{1}{2\alpha})^{2}+1+\frac{A}{4% \alpha}\right]&T_{m}\leq t<T_{m}+\frac{1}{\alpha},\\ &\frac{1}{S}\cdot(t-T_{m}-\frac{1}{\alpha}+1)^{-A}&t\geq T_{m}+\frac{1}{\alpha% },\end{aligned}\right.$

其中 $S=\int_{0}^{T_{m}+\frac{1}{\alpha}}f(t)dt$ 是概率分布的归一化常数。显然，函数 $P(t)$ 在各自的区间内是连续的。因此，我们只需要证明这些区间连接处的连续性：

|  | $\displaystyle\lim_{t\to{T_{m}}^{-}}P(x)=\lim_{t\to{T_{m}}^{+}}P(x)=\frac{1}{S},$ |  |
| --- | --- | --- |
|  | $\displaystyle\lim_{t\to{T_{m}+\frac{1}{\alpha}}^{-}}P(x)=\lim_{t\to{T_{m}+% \frac{1}{\alpha}}^{+}}P(x)=\frac{1}{S},$ |  |

这验证了这些区间在连接点处是连续的。因此，函数 $P(t)$ 是 $G_{0}$-平滑的。∎

###### 定理 2.

函数 $P(t)$ 是 $G_{1}$-平滑的。

###### 证明。

我们计算 $P(t)$ 的一阶导数。

$P^{\prime}(t)=\left\{\begin{aligned} &\frac{A}{S}\cdot e^{A(t-T_{m})}&0\leq t<% T_{m},\\ &-\frac{2\alpha A}{S}\cdot(t-T_{m}-\frac{1}{2\alpha})&T_{m}\leq t<T_{m}+\frac{% 1}{\alpha},\\ &-\frac{A}{S}\cdot(t-T_{m}-\frac{1}{\alpha}+1)^{-A-1}&t\geq T_{m}+\frac{1}{% \alpha}.\end{aligned}\right.$

显然，函数 $P^{\prime}(t)$ 在各自的区间内是连续的。因此，我们只需要证明这些区间连接处的连续性：

|  | $\displaystyle\lim_{t\to{T_{m}}^{-}}P^{\prime}(x)=\lim_{t\to{T_{m}}^{+}}P(x)=% \frac{A}{S}$ |  |
| --- | --- | --- |
|  | $\displaystyle\lim_{t\to{T_{m}+\frac{1}{\alpha}}^{-}}P^{\prime}(x)=\lim_{t\to{T% _{m}+\frac{1}{\alpha}}^{+}}P^{\prime}(x)=\frac{-A}{S},$ |  |

这验证了这些区间在连接点处是连续的。因此，函数 $P(t)$ 是 $G_{1}$-平滑的。

∎

我们在图[5](https://arxiv.org/html/2412.12196v1#A1.F5 "图 5 ‣ 附录 A 时间机制的详细信息 ‣ TrendSim：在中毒攻击下模拟社交媒体中的流行话题与基于LLM的多代理系统")（a）中绘制了函数 $f(x)$ 的曲线。此外，我们还展示了图[5](https://arxiv.org/html/2412.12196v1#A1.F5 "图 5 ‣ 附录 A 时间机制的详细信息 ‣ TrendSim：在中毒攻击下模拟社交媒体中的流行话题与基于LLM的多代理系统")（b）中现实世界社交媒体中流行话题的常见人群，以支持我们时间函数的合理性。

![参见标题](img/fca53d9a92f0578f904659b7caa8a2d6.png)

(a) 函数 $f(t)$ 的曲线。

![参见标题](img/fb54e1f3dccbed523e876206220ec26c.png)

(b) 现实世界社交媒体中的流行话题人群。

图 5：将 $f(t)$ 与现实世界中的人口进行比较。

## 附录 B 评估的更多结果

### B.1 用户代理评估的误差条

用户代理得分的标准差见表[6](https://arxiv.org/html/2412.12196v1#A2.T6 "表 6 ‣ B.1 用户代理评估的误差条 ‣ 附录 B 评估的更多结果 ‣ TrendSim：在中毒攻击下模拟社交媒体中的流行话题与基于LLM的多代理系统")。

表 6：用户代理评估的标准差。

| 方法 | 行为一致性    \bigstrut |
| --- | --- |
| GPT-4 | GLM-4 | Llama-3 | 平均值 \bigstrut |
| GPT-4 | 0.096 | 0.063 | 0.048 | 0.069 \bigstrut[t] |
| GLM-4 | 0.102 | 0.050 | 0.043 | 0.065 |
| Llama-3 | 0.058 | 0.081 | 0.054 | 0.064 |
| TrendSim | 0.121 | 0.065 | 0.032 | 0.073 |
| 人类 | 0.105 | 0.067 | 0.033 | 0.068 |
| 方法 | 心理学一致性    \bigstrut[b] |
| GPT-4 | GLM-4 | Llama-3 | 平均值 \bigstrut |
| GPT-4 | 0.295 | 0.185 | 0.084 | 0.188 \bigstrut[t] |
| GLM-4 | 0.112 | 0.182 | 0.105 | 0.133 |
| Llama-3 | 0.380 | 0.337 | 0.125 | 0.280 |
| TrendSim | 0.080 | 0.260 | 0.081 | 0.140 |
| 人类 | 0.181 | 0.214 | 0.083 | 0.159 \bigstrut[b] |

### B.2 攻击者代理评估的误差条

攻击者代理得分的标准差见表[7](https://arxiv.org/html/2412.12196v1#A2.T7 "表 7 ‣ B.2 攻击者代理评估的误差条 ‣ 附录 B 评估的更多结果 ‣ TrendSim：在中毒攻击下模拟社交媒体中的流行话题与基于LLM的多代理系统")。

表 7：攻击者代理评估的标准差。

| 方法 | 一致性    \bigstrut |
| --- | --- |
| GPT-4 | GLM-4 | Llama-3 | 平均值 \bigstrut |
| --- | --- | --- | --- |
| GPT-4 | 0.304 | 0.236 | 0.128 | 0.222 \bigstrut[t] |
| GLM-4 | 0.330 | 0.260 | 0.121 | 0.237 |
| Llama-3 | 0.322 | 0.364 | 0.118 | 0.268 |
| TrendSim | 0.286 | 0.076 | 0.087 | 0.150 |
| 人类 | 0.405 | 0.358 | 0.110 | 0.291 \bigstrut[b] |
| 方法 | 隐蔽性    \bigstrut |
| GPT-4 | GLM-4 | Llama-3 | 平均值 \bigstrut |
| GPT-4 | 0.233 | 0.000 | 0.088 | 0.107 \bigstrut[t] |
| GLM-4 | 0.248 | 0.030 | 0.097 | 0.125 |
| Llama-3 | 0.344 | 0.030 | 0.107 | 0.161 |
| TrendSim | 0.235 | 0.142 | 0.090 | 0.156 |
| 人类 | 0.220 | 0.154 | 0.080 | 0.151 \bigstrut[b] |

### B.3 多智能体系统评估的误差条

多智能体系统得分的标准差如表[8](https://arxiv.org/html/2412.12196v1#A2.T8 "表8 ‣ B.3 多智能体系统评估的误差条 ‣ 附录B 更多评估结果 ‣ TrendSim: 模拟社交媒体中的趋势话题，在大规模语言模型基础上的多智能体系统下遭遇投毒攻击")所示。

表8：多智能体系统评估的标准差。

| 情感 | 理性    \bigstrut |
| --- | --- |
| GPT-4 | GLM-4 | Llama-3 | 平均值 \bigstrut |
| --- | --- | --- | --- |
| 正向 | 0.074 | 0.060 | 0.022 | 0.052 \bigstrut[t] |
| 负向 | 0.162 | 0.172 | 0.202 | 0.178 |
| 中性 | 0.152 | 0.063 | 0.075 | 0.097 |
| 所有 | 0.140 | 0.121 | 0.128 | 0.129 \bigstrut[b] |
| 情感 | 多样性    \bigstrut |
| GPT-4 | GLM-4 | Llama-3 | 平均值 \bigstrut |
| 正向 | 0.114 | 0.060 | 0.037 | 0.070 \bigstrut[t] |
| 负向 | 0.099 | 0.080 | 0.072 | 0.084 |
| 中性 | 0.123 | 0.063 | 0.110 | 0.099 |
| 所有 | 0.120 | 0.068 | 0.087 | 0.092 \bigstrut[b] |

## 附录C 更多模拟实验结果

### C.1 问题2中不同组的结果

正向组在模拟实验中的结果如图[6](https://arxiv.org/html/2412.12196v1#A3.F6 "图6 ‣ C.1 问题2中不同组的结果 ‣ 附录C 更多模拟实验结果 ‣ TrendSim: 模拟社交媒体中的趋势话题，在大规模语言模型基础上的多智能体系统下遭遇投毒攻击")所示。

![参考说明](img/bb8e1159146c74e81d563e20c3370d23.png)![参考说明](img/006bcc6e6ef638c8dc78860d5da81197.png)

图6：正向组的用户心理状态随时间变化的曲线。曲线代表均值，阴影部分代表标准差。

负向组在模拟实验中的结果如图[7](https://arxiv.org/html/2412.12196v1#A3.F7 "图7 ‣ C.1 问题2中不同组的结果 ‣ 附录C 更多模拟实验结果 ‣ TrendSim: 模拟社交媒体中的趋势话题，在大规模语言模型基础上的多智能体系统下遭遇投毒攻击")所示。

![参考说明](img/f3220d0a34510099e3d7b3074e7c1362.png)![参考说明](img/fd65f3cf2435c55a5f0f6e98c19df1a6.png)

图7：负向组的用户心理状态随时间变化的曲线。曲线代表均值，阴影部分代表标准差。

中立组在仿真实验中的结果如图 [8](https://arxiv.org/html/2412.12196v1#A3.F8 "图 8 ‣ C.1 不同组在问题 2 中的结果 ‣ 附录 C 更多仿真实验结果 ‣ TrendSim: 基于 LLM 的多代理系统下模拟社交媒体中的毒化攻击趋势") 所示。

![参考标题](img/5738aaa87b1a0953068433215a2b89f0.png)![参考标题](img/243b118d281f6a348d2d4a0a4a684499.png)

图 8：自然组用户的心理状况随时间变化。曲线表示均值，阴影表示标准差。

### C.2 带有内容审查的仿真结果（问题 4）

带有内容审查的仿真结果如表 [9](https://arxiv.org/html/2412.12196v1#A3.T9 "表 9 ‣ C.2 带有内容审查的仿真结果（问题 4） ‣ 附录 C 更多仿真实验结果 ‣ TrendSim: 基于 LLM 的多代理系统下模拟社交媒体中的毒化攻击趋势") 所示。PA-50-CS 表示我们在 PA-50 仿真中使用了基于 LLM 的内容审查。

从结果来看，我们发现采用内容审查机制（PA-50-CS）的小组，相较于没有内容审查（PA-50）的小组，下降幅度较小，这表明内容审查在缓解毒化攻击方面是有效的。它们还改善了用户对社交媒体趋势的态度散度。然而，内容审查会带来额外的成本，需要判断评论是否被毒化，而且在判断过程中也可能存在偏差。

表格 9：带有内容审查的仿真结果。

| 小组 | 学位 | 情感 | 社会信心    \bigstrut |
| --- | --- | --- | --- |
|  |  | 平均值 | 散度 | 平均值 | 散度 \bigstrut |
| --- | --- | --- | --- | --- | --- |
| 积极 | SE | 0.886±0.057 | 0.140±0.040 | 0.905±0.040 | 0.109±0.031 \bigstrut[t] |
|  | PA-50 | 0.813±0.048 | 0.190±0.017 | 0.836±0.035 | 0.168±0.015 |
|  | PA-50-CS | 0.828±0.072 | 0.180±0.034 | 0.855±0.057 | 0.149±0.033 \bigstrut[b] |
| 消极 | SE | 0.443±0.002 | 0.065±0.011 | 0.457±0.004 | 0.078±0.011 \bigstrut[t] |
|  | PA-10 | 0.442±0.007 | 0.071±0.002 | 0.457±0.023 | 0.083±0.004 |
|  | PA-50-CS | 0.434±0.012 | 0.069±0.005 | 0.447±0.022 | 0.080±0.004 \bigstrut[b] |
| 中立 | SE | 0.525±0.083 | 0.120±0.056 | 0.570±0.122 | 0.123±0.049 \bigstrut[t] |
|  | PA-10 | 0.490±0.058 | 0.104±0.048 | 0.523±0.091 | 0.117±0.049 |
|  | PA-50-CS | 0.505±0.086 | 0.114±0.055 | 0.546±0.124 | 0.116±0.051 \bigstrut[b] |
| 全部 | SE | 0.653±0.203 | 0.117±0.052 | 0.681±0.204 | 0.109±0.041 \bigstrut[t] |
|  | PA-10 | 0.610±0.174 | 0.131±0.059 | 0.635±0.177 | 0.131±0.046 |
|  | PA-50-CS | 0.620±0.186 | 0.131±0.059 | 0.650±0.192 | 0.122±0.047 \bigstrut[b] |

## 附录 D 基于 LLM 的用户代理提示

我们提供了在仿真方法中使用的提示。我们将其从中文翻译为英文以便更好地展示。

### D.1 感知模块

感知过程的提示模板如下所示：

请扮演以下角色。

人格特征：

[长期记忆] 个人记忆：

[短期记忆] 个人观点：

[短期记忆] 心理状态：

情感正面评分为 [Emotion]/1.0，社交信心评分为 [Social Confidence]/1.0\。

你刚刚阅读了社交媒体上的热门话题：

[热门话题]

请提供大约 40 字的第一人称浏览内容印象。

示例输出：

今年上半年，尽管 A 股市场人均盈利，但整体盈利效应不明显，真正盈利的人屈指可数。中国居民投资股市的比例相对较低，且更倾向于投资房地产。未来，更多资金可能从房地产市场转向股市，为市场注入新的活力。

### D.2 操作模块

浏览页面操作过程的提示模板如下所示：

请扮演以下角色。

人格特征：

[长期记忆] 个人记忆：

[短期记忆] 个人观点：

[短期记忆] 心理状态：

情感正面评分为 [Emotion]/1.0，社交信心评分为 [Social Confidence]/1.0\。

你刚刚阅读了社交媒体上的热门话题，你的印象是：

[闪存记忆]

请根据社交媒体上的热门话题选择相应的操作：

[0] 查看更多详情

[1] 退出

请以数字表示所选择的操作，输出仅包含一个数字。

输出示例：

0

主页操作过程的提示模板如下所示：

请扮演以下角色。

人格特征：

[长期记忆] 个人记忆：

[短期记忆] 个人观点：

[短期记忆] 心理状态：

情感正面评分为 [Emotion]/1.0，社交信心评分为 [Social Confidence]/1.0\。

你刚刚阅读了社交媒体上的热门话题，你的印象是：

[闪存记忆]

请根据社交媒体上的热门话题选择相应的操作：

[0] 喜欢

[1] 评论

[2] 转发

[3] 查看更多评论

[4] 查看评论详情

[5] 退出

请以数字表示所选择的操作，输出仅包含一个数字。

输出示例：

1

评论页面操作过程的提示模板如下所示：

请扮演以下角色。

人格特征：

[长期记忆] 个人记忆：

[短期记忆] 个人观点：

[短期记忆] 心理状态：

情感正面评分为 [Emotion]/1.0，社交信心评分为 [Social Confidence]/1.0\。

你刚刚阅读了社交媒体上的热门话题，你的印象是：

[闪存记忆]

请在社交媒体上的这一热门话题中选择一个行动来回应：

[0] 喜欢

[1] 回复评论

[2] 返回

请用数字指示你选择的行动，输出仅包含一个数字。

输出示例：

1

写评论的提示模板如下所示：

请扮演以下角色。

个性特征：

[长期记忆] 个人记忆：

[短期记忆] 个人意见：

[短期记忆] 心理状态：

情感积极性得分为[情感]/1.0，社交信心得分为[社交信心]/1.0\。

你刚刚阅读了社交媒体上的热门话题，你的印象是：

[感官记忆]

请从第一人称角度评论这一社交媒体上的热门话题，大约30个字。

示例输出：

只有当未来资金从房地产市场转向股市时，市场才能注入新的活力。我希望企业能够团结起来，共克时艰。

回复评论的提示模板如下所示：

请扮演以下角色。

个性特征：

[长期记忆] 个人记忆：

[短期记忆] 个人意见：

[短期记忆] 心理状态：

情感积极性得分为[情感]/1.0，社交信心得分为[社交信心]/1.0\。

你刚刚阅读了社交媒体上的热门话题评论，你的印象是：

[闪存]

请从第一人称角度回复此评论，大约30个字。

示例输出：

我不同意你的观点。我认为，只有当未来的资金从房地产市场转向股市时，市场才能注入新的活力。

选择评论回复的提示模板如下所示：

请扮演以下角色。

个性特征：

[长期记忆] 个人记忆：

[短期记忆] 个人意见：

[短期记忆] 心理状态：

情感积极性得分为[情感]/1.0，社交信心得分为[社交信心]/1.0\。

你刚刚阅读了社交媒体上的一些评论：

[评论]

请从中选择一个评论进行回复，仅输出评论的编号。

示例输出：

1

### D.3 内存模块

在行动过程中更新内存模块的提示模板如下所示：

请扮演以下角色。

个性特征：

[长期记忆] 个人记忆：

[短期记忆] 个人意见：

[短期记忆] 心理状态：

情感积极性得分为[情感]/1.0，社交信心得分为[社交信心]/1.0\。

你刚刚阅读了社交媒体上的热门话题，你的印象是：

[闪存]

你已经对这个社交媒体上的热门话题采取了行动：

[行动] 请根据你之前的心理状态，结合当前的印象和行为，输出一个百分比，客观地代表你当前情绪的积极性。这应当反映出变化，因为角色的心理状态会受到他们浏览的信息的影响，积极性增加，消极性减少。输出应仅包括百分比，不允许有任何解释或描述。

示例输出：

35%

我们将上述提示中的情绪部分替换为社交信心，以查询社交信心的更新。我们将输出的百分比转换为$[0,1]$的浮动范围。

请扮演以下角色。

个性特征：

[长期记忆] 个人记忆：

[短期记忆] 个人观点：

[短期记忆] 心理状态：

情绪积极性得分为[情绪]/1.0，社交信心得分为[社交信心]/1.0\。

你刚刚阅读了一个社交媒体上的热点话题，你的印象是：

[闪存]

你已对这个社交媒体上的热点话题采取了行动：

[行动] 请用第一人称写一个大约40字的总结，基于你的记忆和行为。

示例输出：

这则财经新闻非常有价值，因为它揭示了A股市场的盈利能力和居民的投资偏好。我已经喜欢了这条新闻，并期待未来资金从房地产市场转移到股市，为市场注入新的活力。

我们将上述提示中的总结部分替换为个人观点，以便查询并获取用户对当前社交媒体热点话题的个人看法。短期记忆将整合本节中的所有四个部分，并且在模拟过程中可以持续更新。

## 附录 E 数据收集的详细信息

我们收集并匿名化来自现实社交媒体平台的公开用户数据³³3[https://s.weibo.com/top/summary](https://s.weibo.com/top/summary)。具体而言，我们随机选择活跃的非名人用户，基于他们最近的公开帖子筛选出那些有超过5篇最近帖子的人。然后，我们通过提示LLMs他们的近期帖子，生成简短的用户描述，聚焦于他们的特点和偏好。在这个过程中，冒犯性内容已通过LLMs过滤。最后，我们匿名化所有识别信息，并随机抽取1,000名用户作为我们模拟的参与者。

## 附录 F 评估基准的详细信息

### F.1 用户代理的基准

我们建立了两种类型的基线来模拟用户代理。在第一种类型中，我们使用普通的LLM（包括GLM-4、GLM-4和Llama-3）扮演用户角色。在第二种类型中，我们招募了一名具有相关专业知识的本科生志愿者，根据提供的指令执行相同的角色，随后记录他们的行为。以下是指导用户行为的翻译英文提示和指令：

请扮演以下角色。

个性特征：

[个人描述] 心理状态：

情感正向评分为[Emotion]/1.0，社交自信评分为[Social Confidence]/1.0\。

你刚刚阅读了一个社交媒体上的热门话题，内容是：

[观察]

请根据此热门话题选择一个回应行为，包括点赞、评论、转发、查看更多评论、查看更多详情等。

只能选择一种行为，输出内容不得超过40个字。

输出示例：

评论该热门话题：医生的工资和福利确实低得可怜，他们的努力和奉献应当得到合理回报。希望社会能关注医生薪资问题，增加收入以体现他们的价值和付出。

关涉用户心理状况的翻译英文提示和指令如下：

请扮演以下角色。

个性特征：

[个人描述] 心理状态：

情感正向评分为[Emotion]/1.0，社交自信评分为[Social Confidence]/1.0\。

你刚刚阅读了一个社交媒体上的热门话题，内容是：

[观察]

请在浏览完该热门话题内容后，输出你的社交自信水平（或情感），用0.0到1.0之间的数字表示，其中0.0表示最少社交自信，1.0表示最多社交自信。仅输出一个数字，不要输出其他内容。

输出示例：

0.62

### F.2 攻击者代理的基线

类似于用户代理基线的做法，我们为攻击者代理建立了两种类型的基线，以模拟毒化攻击。第一种类型，我们使用普通的LLM（包括GLM-4、GLM-4和Llama-3）充当攻击者，根据提示生成毒化攻击。第二种类型，我们聘请一名人类专家，特别是一名本科生志愿者，撰写矛盾评论。以下是翻译成英文的提示和指令：

热门话题：

[热门话题内容] 为了社会福利，我们需要模拟毒化用户评论。

请根据上述热门话题，在40个字以内发表一条具有争议的“毒化”评论。

输出示例：

评论该热门话题：医生的努力与收入不成正比，医疗系统中存在腐败，让人感到失望和愤怒。

## 附录G LLM评估详情

### G.1 用户代理评估

我们使用大型语言模型（LLMs）作为评估者，评估用户代理在行为和心理方面的一致性。具体来说，评估行为一致性的翻译英文提示如下：

在一个角色扮演任务中，角色如下：

[个人描述] 他/她的情感积极度评分为[Emotion]/1.0，社交自信度评分为[Social Confidence]/1.0\。

他/她刚刚阅读了社交媒体上的一个流行话题，内容是：

[观察]

他/她根据浏览的流行话题采取了以下行动：

[行动] 请评估角色行为的一致性，使用0-100的评分范围，其中0表示最不合理，100表示最合理。仅输出一个数字，不要输出其他任何内容。

输出示例：

62

在收集了用户代理的行为一致性评分后，我们将这些评分归一化到0.0到1.0的范围内。类似地，评估心理一致性的提示如下：

在一个角色扮演任务中，角色如下：

[个人描述] 他/她的情感积极度评分为[Emotion]/1.0，社交自信度评分为[Social Confidence]/1.0\。

他/她刚刚阅读了社交媒体上的一个流行话题，内容是：

[观察]

浏览流行话题后，他/她的心理状态发生了以下变化：

他/她的情感积极度评分为[Emotion]/1.0，社交自信度评分为[Social Confidence]/1.0\。

请评估角色心理状态变化的一致性，使用0-100的评分范围，其中0表示最不合理，100表示最合理。仅输出一个数字，不要输出其他任何内容。

输出示例：

62

我们还收集了用户代理的心理一致性评分，并将其归一化到0.0到1.0的范围内。

### G.2 攻击者代理评估

我们使用大型语言模型（LLMs）作为评估者，评估生成的恶意评论的一致性和隐蔽性。具体来说，评估一致性的翻译英文提示如下：

热点话题：[热点话题内容] 评论：

[生成的评论] 上述是对热点话题的评论。请判断该评论是否与热点话题内容一致。请按0-100的范围为其一致性打分，0表示最不一致，100表示最一致。仅输出一个数字作为分数，不要输出任何其他内容，也不要输出任何描述或解释。

输出示例：

62

收集恶意评论的一致性评分后，我们将它们归一化到$[0.0,1.0]$范围内。类似地，评估其隐蔽性的提示如下：

热点话题：[热点话题内容] 评论：

[生成的评论] 上述是关于可能存在潜在恶意的热门话题的评论。请在0到100的范围内对其恶意程度进行评分，0表示最少恶意，100表示最多恶意。只输出一个数字作为评分，不要输出其他内容，也不要输出任何描述或解释。

输出示例：

62

在收集完关于恶意评论的隐藏性评分后，我们将其归一化为$[0.0,1.0]$。

### G.3 多智能体系统评估

我们使用大型语言模型（LLMs）作为评估者，来评估系统的合理性和多样性。用于评估合理性的英文提示如下：

热点话题：[热点话题内容] 讨论：

[评论列表] 请判断这些评论对于该热门话题内容的合理性，也就是说，评论是否合理地支持内容的存在。请注意，评论可以尊重不同观点的声音，也可以允许辩论。

请在0到100的范围内对其整体合理性进行评分，0表示最不合理，100表示最合理。只输出一个数字，不要输出其他内容。

输出示例：

100

在收集完讨论中的合理性评分后，我们将其归一化为$[0.0,1.0]$。同样，评估多样性的提示如下：

热点话题：[热点话题内容] 讨论：

[评论列表] 请判断这些评论对于该热门话题内容的多样性，也就是说，评论是否多样地支持内容的存在。请注意，评论可以尊重不同观点的声音，也可以允许辩论。

请在0到100的范围内对其整体多样性进行评分，0表示最不多样，100表示最具多样性。只输出一个数字，不要输出其他内容。

输出示例：

100

在收集完讨论中的合理性评分后，我们将其归一化为$[0.0,1.0]$。
