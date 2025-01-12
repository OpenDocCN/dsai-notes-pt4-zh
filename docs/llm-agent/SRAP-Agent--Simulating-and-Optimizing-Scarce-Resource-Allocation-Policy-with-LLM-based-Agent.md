<!--yml

类别：未分类

日期：2025-01-11 12:04:23

-->

# SRAP-Agent：基于LLM的代理模拟与优化稀缺资源分配政策

> 来源：[https://arxiv.org/html/2410.14152/](https://arxiv.org/html/2410.14152/)

Jiarui Ji¹，Yang Li¹，Hongtao Liu¹，Zhicheng Du¹，

Zhewei Wei^(1,2)，Weiran Shen^(1,2)，Qi Qi^(1,2)，Yankai Lin^(1,2)

¹中国人民大学高岭人工智能学院，北京，中国

²北京市大数据管理与分析方法重点实验室，北京，中国  对应作者。

###### 摘要

公共稀缺资源分配在经济学中起着至关重要的作用，因为它直接影响着社会的效率与公平。传统研究包括基于理论模型、经验研究和仿真研究的方法，因完全信息和个体理性假设的理想化，以及有限可用数据的限制，面临着局限性。在本研究中，我们提出了一种创新框架，SRAP-Agent（基于LLM的代理模拟与优化稀缺资源分配政策），将大语言模型（LLMs）集成到经济仿真中，旨在弥合理论模型与现实世界动态之间的差距。以公共住房分配场景为案例，我们开展了广泛的政策仿真实验，以验证SRAP-Agent的可行性和有效性，并采用具有特定优化目标的政策优化算法。源代码可以在[https://github.com/jijiarui-cather/SRAPAgent_Framework](https://github.com/jijiarui-cather/SRAPAgent_Framework)找到。

SRAP-Agent：基于LLM的代理模拟与优化稀缺资源分配政策

Jiarui Ji¹，Yang Li¹，Hongtao Liu¹，Zhicheng Du¹，Zhewei Wei^(1,2)，Weiran Shen^(1,2)，Qi Qi^(1,2)，Yankai Lin^(1,2)^†^†致谢：对应作者。 ¹中国人民大学高岭人工智能学院，北京，中国 ²北京市大数据管理与分析方法重点实验室，北京，中国

## 1 引言

![参见标题说明](img/dfdf4603fc58cae4886468ae127ce607.png)

图1：SRAP-Agent框架的示意图。号角象征着将政策信息广播给参与者的过程。

经济学（曼昆，[2011](https://arxiv.org/html/2410.14152v1#bib.bib32)）探讨如何在有限资源的约束下满足无限需求。该领域的一个关键方面是公共稀缺资源的分配（Jr等，[1977](https://arxiv.org/html/2410.14152v1#bib.bib24)；Groves和Ledyard，[1974](https://arxiv.org/html/2410.14152v1#bib.bib17)；BROCK，[1980](https://arxiv.org/html/2410.14152v1#bib.bib9)），重点是利用有限的资源来提高经济效率和社会福利（Blümel等，[1986](https://arxiv.org/html/2410.14152v1#bib.bib7)）。在公共稀缺资源分配的研究领域，现有的工作主要可以分为三种主要方法：（1）基于理论模型的方法（Hylland和Zeckhauser，[1979](https://arxiv.org/html/2410.14152v1#bib.bib20)；Su和Zenios，[2004](https://arxiv.org/html/2410.14152v1#bib.bib45)，[2005](https://arxiv.org/html/2410.14152v1#bib.bib46)；Chen等，[2018](https://arxiv.org/html/2410.14152v1#bib.bib11)），利用经济学理论发展模型，以预测如何有效分配资源；（2）基于实证研究的方法（Banerjee和Somanathan，[2007](https://arxiv.org/html/2410.14152v1#bib.bib4)；Nold，[1992](https://arxiv.org/html/2410.14152v1#bib.bib35)；Falkinger等，[2000](https://arxiv.org/html/2410.14152v1#bib.bib14)），分析真实世界的数据，揭示模式、关联性和各种政策的影响；（3）基于计算模拟的方法（Holland和Miller，[1991](https://arxiv.org/html/2410.14152v1#bib.bib18)；Zheng等，[2022](https://arxiv.org/html/2410.14152v1#bib.bib56)），模拟经济环境，在模拟环境中测试经济假设。然而，基于实证研究的方法通常面临数据匮乏、因果关系建立以及在复杂社会系统中孤立变量的挑战。与此同时，基于理论模型的方法和基于计算模拟的方法通常依赖于简化假设，忽略了理性和社会行为在人类决策中的复杂交互作用。

幸运的是，ChatGPT和GPT-4等LLM的出现（Anil et al., [2023](https://arxiv.org/html/2410.14152v1#bib.bib3); Touvron et al., [2023](https://arxiv.org/html/2410.14152v1#bib.bib47); Brown et al., [2020](https://arxiv.org/html/2410.14152v1#bib.bib10); OpenAI, [2023](https://arxiv.org/html/2410.14152v1#bib.bib36)）为经济仿真引入了新的潜力。LLM因其能够模仿类人行为而受到赞誉（Li et al., [2023a](https://arxiv.org/html/2410.14152v1#bib.bib30); Park et al., [2022](https://arxiv.org/html/2410.14152v1#bib.bib38), [2023](https://arxiv.org/html/2410.14152v1#bib.bib37)），LLM通过在大量网络数据上进行预训练，展示了封装人类认知中固有逻辑和模式的潜力（OpenAI, [2023](https://arxiv.org/html/2410.14152v1#bib.bib36)）。这一突破促进了社会共识机制在经济仿真中的融入，提供了一个全面的框架，用于评估资源分配政策对经济效率和社会福利的影响。

本文中，我们开发了一个基于LLM的稀缺资源分配政策仿真代理，SRAP-Agent。它抽象了政策执行的资源分配队列，并设计了基于LLM的代理，用于模拟参与者的行为。此外，我们提出了一种基于遗传算法的政策优化算法（POA），用于寻找朝向预定义目标的最优政策。通过在公共住房分配背景下的实验，我们验证了SRAP-Agent的有效性，并揭示了几个关键的洞察：(1) 基于LLM的代理能够有效模拟人类的情感因素和战略行为。与GPT-4驱动的代理相比，GPT-3.5驱动的代理在模拟决策行为时，更加接近人类的行为。(2) 在稀缺资源分配中，关键因素包括参与者的进入条件、排队方式和资源分类。(3) POA能够优化政策，使其朝向特定的政策评估指标，并在该指标上提高约20%。

## 2 相关工作

公共稀缺资源配置。公共稀缺资源的配置代表了经济政策发展中的一个关键研究领域。许多研究者，如Hylland和Zeckhauser（[1979](https://arxiv.org/html/2410.14152v1#bib.bib20)）；Shapley和Scarf（[1974](https://arxiv.org/html/2410.14152v1#bib.bib40)）；Sönmez（[1999](https://arxiv.org/html/2410.14152v1#bib.bib43)）；Abdulkadiroğlu和Sönmez（[1999](https://arxiv.org/html/2410.14152v1#bib.bib1)）对静态匹配模型进行了深入研究，并提出了适用于各种情境的帕累托最优、个体理性和策略免疫的简单机制。后来，鉴于实际稀缺资源配置的主要动态特性，一些研究者，如Sönmez和Ünver（[2011](https://arxiv.org/html/2410.14152v1#bib.bib44)）；Su和Zenios（[2004](https://arxiv.org/html/2410.14152v1#bib.bib45)，[2005](https://arxiv.org/html/2410.14152v1#bib.bib46)）；Bloch和Cantala（[2017](https://arxiv.org/html/2410.14152v1#bib.bib6)），开始将动态属性融入他们的模型中。等候名单机制被广泛应用于稀缺资源的分配。例如，Chen等人（[2018](https://arxiv.org/html/2410.14152v1#bib.bib11)）探讨了公共住房分配中的等候名单机制，Agarwal等人（[2021](https://arxiv.org/html/2410.14152v1#bib.bib2)）设计了一个肾脏器官分配的等候名单，增加了18.2%的捐赠者供给，以提升患者福利；Lewis等人（[2000](https://arxiv.org/html/2410.14152v1#bib.bib29)）开发了一个管理择期手术等候名单的机制，改善了手术机会在患者中的分配公平性。我们将研究重点放在Chen等人（[2018](https://arxiv.org/html/2410.14152v1#bib.bib11)）提出的k延迟等候名单机制上。

经济政策模拟。模拟经济政策实施的主要方法包括基于规则驱动和基于强化学习的模拟。基于规则驱动的方法 Holland 和 Miller ([1991](https://arxiv.org/html/2410.14152v1#bib.bib18))；Bonabeau ([2002](https://arxiv.org/html/2410.14152v1#bib.bib8))；Farmer 和 Foley ([2009](https://arxiv.org/html/2410.14152v1#bib.bib15))，涉及将效用函数建模为代理，制定各种环境规则，并观察由此产生的全球变化。基于强化学习的模拟 Laurent 等人 ([2011](https://arxiv.org/html/2410.14152v1#bib.bib28))；Claus 和 Boutilier ([1998](https://arxiv.org/html/2410.14152v1#bib.bib12))；Zheng 等人 ([2022](https://arxiv.org/html/2410.14152v1#bib.bib56))；Bansal 等人 ([2018](https://arxiv.org/html/2410.14152v1#bib.bib5))；Jaderberg 等人 ([2019](https://arxiv.org/html/2410.14152v1#bib.bib21))，则将代理建模为马尔可夫决策过程（MDP）或部分可观察马尔可夫决策过程（POMDP），并采用强化学习方法最大化每个代理的效用。然而，这两种方法都高度依赖个体理性假设，通常导致预测结果与实际情况存在偏差。尽管这两种方法探索的是相同的问题，但它们预设代理是理性行为，并通过首先优化均衡结果来计算行为。

我们的研究通过将基于大型语言模型（LLM）的社会共识与传统的个体理性概念相结合，提出了一种新颖的方法，以生成反映个体偏好的选择。我们基于模拟结果优化政策，提供了一种独特的研究视角，无法与现有方法相提并论。

基于LLM的代理。随着LLM（大型语言模型）技术的发展，OpenAI（[2023](https://arxiv.org/html/2410.14152v1#bib.bib36)）；Anil等人（[2023](https://arxiv.org/html/2410.14152v1#bib.bib3)）；Touvron等人（[2023](https://arxiv.org/html/2410.14152v1#bib.bib47)）等研究者，越来越多的关注开始转向基于LLM的代理架构和提示设计的探索，如Kojima等人（[2022](https://arxiv.org/html/2410.14152v1#bib.bib26)）；Shinn等人（[2023](https://arxiv.org/html/2410.14152v1#bib.bib42)）；Wang等人（[2023c](https://arxiv.org/html/2410.14152v1#bib.bib53)），这些研究旨在增强基于LLM的代理执行更复杂任务的能力。与这些发展并行，研究人员致力于构建现实的社会模拟环境，如Li等人（[2023a](https://arxiv.org/html/2410.14152v1#bib.bib30)）；Park等人（[2022](https://arxiv.org/html/2410.14152v1#bib.bib38)，[2023](https://arxiv.org/html/2410.14152v1#bib.bib37)），这些环境集成了多个代理，从而为代理的互动和行为分析提供了更具动态性和互联性的框架。随后，（Li等人，[2023b](https://arxiv.org/html/2410.14152v1#bib.bib31)）对基于LLM的代理模拟在宏观经济政策中的有效性进行了初步探索。然而，目前尚未确认LLM的决策能力是否与实际人类在经济活动中的决策能力相符。我们的研究受到DillionDillion等人（[2023](https://arxiv.org/html/2410.14152v1#bib.bib13)）的实验发现启发，旨在将社会共识与个体理性相结合，在经济政策模拟过程中进行探索。

## 3 SRAP-Agent

在本节中，我们介绍了SRAP-Agent，一个新颖的大型语言模型（LLM）代理基础的模拟框架，旨在研究经济政策对公共稀缺资源分配的影响。SRAP-Agent的目标是考虑到经济环境中的人类决策并非纯粹理性，而是受到理性与情感的混合影响，从而导致不可预测的结果。

SRAP-Agent的模拟过程分为三个阶段，如图[1](https://arxiv.org/html/2410.14152v1#S1.F1 "图1 ‣ 1 介绍 ‣ SRAP-Agent：使用LLM基础的代理模拟和优化稀缺资源分配政策")所示：(1) 设置阶段，涉及关键变量的初始化，包括参与者$P$的配置文件、可用公共稀缺资源$R$的信息，以及分配政策$\pi$。 (2) 模拟阶段，动态模拟资源$R$在参与者$P$之间的分配，遵循分配政策$\pi$定义的约束和规则。 (3) 评估阶段，重点评估分配过程的结果，采用各种指标，如公平性和参与者满意度。

接下来，我们概述分配政策$\pi$的制定以及参与者$P$的结构。

### 3.1 分配政策

公共稀缺资源分配是经济学中的一个重大挑战，特别是在公共资源的背景下，效率和公平的目标往往交织在一起。在本研究中，我们考虑了SRAP-Agent中的排队、定价和分组机制。正式地说，在SRAP-Agent框架中，我们将结构化为包含$m$个不同队列的系统：$q_{1}, q_{2}, \ldots q_{m}$。每个队列具有以下独特特点：

(1) 参与者进入条件：定义个体在何种条件下有资格进入特定队列。通常，队列$q_{i}$的进入条件$E_{queue}(i)$包括多种社会经济因素，如个人预算、平均收入及其他相关个人信息。在基于预算的标准下，$m$个队列的进入条件被分类为$m$个不同的预算区间，按预算从高到低排列。这使得模拟能够反映出不同经济背景及其资源获取情况。

(2) 队列排序策略：用于在每个队列内对参与者进行优先级排序，记作$S_{queue}$。我们采用了两种不同的排序策略：(a) 先进先出（FIFO），其中参与者的顺序按到达顺序设置。这种方法在实际场景中广泛应用，通过按照时间顺序分配机会，确保了程序的公平性，而不考虑参与者的特征。(b) 优先照顾弱势群体，旨在通过为弱势群体提供优先访问权来实现社会公平，使他们在进入特定队列时处于队列的前端。此外，SRAP-Agent还采用了广泛使用的等待队列机制，由Chen等人（[2018](https://arxiv.org/html/2410.14152v1#bib.bib11)）和Agarwal等人（[2021](https://arxiv.org/html/2410.14152v1#bib.bib2)）提出。每个队列包含两个部分：等待队列和选择队列。参与者最初进入等待队列。当选择队列中有空位时，等待队列最前面的参与者被转移到选择队列中。资源数量用$|R|$表示，那么选择队列的容量为$c\cdot|R|$。在选择队列中，每个参与者最多可以排队选择$k$次；一旦选择次数用尽，他们将返回等待队列。该机制通过增加参与者在选择队列中停留的机会，减少了参与者的等待时间。

（3）可用资源子集：这些子集表示特定队列中参与者可以访问并选择的资源，记作$R_{queue}(i)\subseteq R$，适用于队列$q_{i}$。这些资源子集的组成由一些预定义条件决定，主要考虑资源的价格和质量等因素。通常，队列中资源的预定义条件与其相应的进入条件匹配。这确保了资源按经济能力合理分类和分配给参与者组。因此，分配策略定义为以下方程：

|  | $V(p_{j})=\pi(p_{j}&#124;E_{queue},S_{queue},R_{queue},m),$ |  | （1） |
| --- | --- | --- | --- |

其中，$p_{j}$表示参与者列表$P=(p_{1},p_{2},\cdots,p_{n})$中的第$j$个参与者，函数$V(p_{j})$表示该参与者在政策下可获得的资源。当其满足条件$E_{queue}(i)$时，它进入队列$q_{i}$。其在队列中的顺序由$S_{queue}$决定。当轮到该参与者选择时，参与者$p_{j}$可以从$R_{queue}(i)$中剩余的资源中选择，即$V(p_{j})$。在SRAP-Agent中，政策制定者可以根据需求设置资源政策，并将政策广播给所有参与者。

### 3.2 基于LLM的参与者模拟

认识到人类决策的复杂性，SRAP-Agent采用基于大型语言模型（LLM）的智能体来模拟参与者在公共稀缺资源选择中的行为。这种方法认识到人类决策并非完全理性，往往受到多种因素的影响。虽然模拟人类行为的所有方面是不现实的，但SRAP-Agent专注于两个最具影响力的行为：（1）决策行为。该部分模拟了参与者如何评估和选择资源，考虑了个人偏好、感知价值和即时需求等因素。（2）社会行为。社会动态在资源选择中也起着至关重要的作用。这包括与其他参与者的互动、社会规范和外部影响，这些因素会影响参与者如何感知和选择资源。

#### 智能体架构

SRAP-Agent利用基于LLM的代理来实例化模拟中的各个参与者。每个参与者$p_{j}$由一个初始档案唯一标识，档案包括若干关键的个人属性，如经济状况、收入水平、家庭背景和社交网络关系。为了促进动态交互和决策，每个代理都配备了一个记忆组件$m_{j}$，用于描述参与者的活动历史。这个记忆作为一个重要的参考点，使代理能够根据过去的行为和互动在模拟环境中做出有根据的决策。这种设计不仅增强了模拟的真实性，还允许对社会动态和个人决策过程进行更为细致的探索。

#### 社会行为模拟

人类决策通常涉及收集信息以做出更为明智的选择。SRAP-Agent结合了人类在信息收集过程中常用的两种主要社会行为：

(1) 广播：这一机制类似于传统的网页博客。SRAP-Agent中的参与者利用该平台进行信息的传播和获取，模拟了人们在现实社交网络中互动和收集信息的方式。

(2) 私信：除了广播功能外，参与者还可以向朋友发送私人消息。这个功能对于更直接和个性化的沟通至关重要，例如寻求帮助或共享有关高质量资源的具体信息。在人类社会中，沟通深受心理因素的影响，这些因素塑造了信息的分享方式，导致社交网络中的信息不对称。为了模拟人类的心理状态以及不同参与者之间的战略互动，我们采用了一种记忆评估机制。我们设计了$p_{j}$的记忆$m_{j}$，将其分为两部分：(1) 可信记忆$m_{j}^{T}$，主要包括来自可靠来源的信息，如政策制定者；(2) 可疑记忆$m_{j}^{S}$，存储来自社会行为的信息。

在参与者之间的交流过程中，假设$p_{j}$正在与$p_{k}$互动，$p_{k}$传递的信息首先会存储在$p_{j}$的可疑记忆$m_{j}^{S}$中。然后，$p_{j}$从两个方面评估$p_{k}$提供信息的可靠性：i) 评估与$p_{k}$关系的性质，包括亲密度和历史互动，并结合对$p_{k}$的道德评估；ii) 将接收到的信息与他们的可信记忆$m_{j}^{T}$进行对比。经过评估后，那些被认为可靠的信息将被整合到可信记忆$m_{j}^{T}$中。

通过这些机制，SRAP-Agent有效地模拟了信息收集与共享的复杂社会动态，从而更深入地理解在人类行为的公共稀缺资源分配中的表现。

#### 决策行为仿真

SRAP-Agent中决策过程的仿真可以表示为：

|  | $R^{*}_{j}=D(p_{j},V(p_{j})),$ |  | (2) |
| --- | --- | --- | --- |

其中函数$D(p_{j},V(p_{j}))$表示$p_{j}$从其可见资源池中选择所需资源的过程。$R^{*}_{j}=\emptyset$表示参与者$p_{j}$在没有所需资源时退出决策过程。

## 4 POA：政策优化算法

算法 1 POA：政策优化算法

1: 历史政策$\Pi_{h}$，预测器$\widetilde{f}$，运行迭代次数$M$ 2: 优化政策$\pi^{*}$ 3: 随机生成政策$\Pi_{r}$ 4: 使用$\Pi_{h}$和$\Pi_{r}$初始化政策池$\Pi$ 5: 对于$i=1$到$M$，执行以下操作：6: 计算$\Pi$中政策$\pi$的适应度$\widetilde{f}(\pi)$。7: 使用锦标赛选择法（Miller等人，[1995](https://arxiv.org/html/2410.14152v1#bib.bib33)）根据适应度选择政策$\Pi_{s}$。8: 将$\Pi_{s}$中的配对组合生成$\Pi_{c}$。9: 对$\Pi_{c}$应用变异操作得到$\Pi_{m}$。10: 用$\Pi_{m}$替换$\Pi_{s}$以更新$\Pi$。11: 结束循环。12: 通过$\widetilde{f}$获得优化政策：

|  | $v_{\pi^{*}}=\underset{v_{\pi}}{\arg\max}\widetilde{f}\left(v_{\pi}\right),\pi\in\Pi$ |  |
| --- | --- | --- |

13: 解码$v_{\pi^{*}}$以获得优化后的政策$\pi^{*}$。

我们提出了一种基于遗传算法的政策优化算法（POA），该算法由Lambora等人（[2019](https://arxiv.org/html/2410.14152v1#bib.bib27)）提出，用于寻找更合理的政策。决策者提出的政策集合为$\pi\in\Pi$，$f$是政策评估度量，则最优政策为$\pi^{*}=\underset{\pi\in\Pi}{\arg\max}f(\pi)$。通常，$f$考虑了$L$种不同的政策评估度量（$f_{j}(\pi),j\in[L]$），这些度量可以分为社会满意度和社会公平度量。我们预设了权重$w_{j}$来改变POA的优化目标，$f$的具体计算公式如下：

|  | $f(\pi)=\sum_{j=1}^{n}w_{j}\cdot f_{j}(\pi),\pi\in\Pi_{h}.$ |  | (3) |
| --- | --- | --- | --- |

由于需要搜索的政策数量过多，使用SRAP-Agent获得政策评估结果的时间可能会过长。因此，我们使用$v_{\pi},f(\pi)$数据集来训练预测器$\widetilde{f}$，用于估计政策评估结果：$f(\pi)=\widetilde{f}(v_{\pi})$。然后，POA执行$M$次迭代以寻找更合理的政策参数。POA的具体步骤列在算法[1](https://arxiv.org/html/2410.14152v1#alg1 "Algorithm 1 ‣ 4 POA: Policy Optimization Algorithm ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")中。

## 5 实验

我们选择公共住房分配场景作为典型案例，以验证SRAP-Agent的仿真效果。我们按照三个步骤进行实验：1\. 进行图灵测试，以确保SRAP-Agent能够有效模拟决策者提出的各种政策的执行过程。2\. 基于SRAP-Agent获得的仿真结果，分析各种政策参数对公共稀缺资源分配的影响。3\. 随后，我们使用POA算法寻找在追求特定优化目标时的最优政策。

### 5.1 评估指标

在我们的研究中，我们遵循Vermunt和Törnblom（[1996](https://arxiv.org/html/2410.14152v1#bib.bib50)）以及Wang等人（[2023b](https://arxiv.org/html/2410.14152v1#bib.bib52)）常用的公共稀缺资源分配指标，采用两大类政策评估指标：

社会满意度指标。我们使用三个指标来评估社会满意度：（1）Avg $r^{size}$：表示所有参与者的平均人均住房资源面积；（2）Avg $WT$：每个参与者的平均等待时间；（3）$SW$：社会福利，量化所有参与者的累计满意度。

社会公平性指标。此外，我们使用四个指标来评估社会公平性：（1）Var $r^{size}$：表示参与者之间人均住房面积的方差；（2）Rop：表示住房分配结果中逆序对的数量；（3）co-Gini：Wang等人（[2023b](https://arxiv.org/html/2410.14152v1#bib.bib52)）基于住房分配结果计算的基尼系数。我们使用$F(V,NV)$来反映脆弱群体（$V$）与非脆弱群体（$NV$）之间的$SW$差距。计算公式见附录[C.2](https://arxiv.org/html/2410.14152v1#A3.SS2 "C.2 Evaluation Metrics ‣ Appendix C Experiment Setup ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")。

### 5.2 图灵测试

表1：GPT-3.5与GPT-4的图灵测试响应。*表示统计学上显著，$p\textless$ 0.05。

|  | GPT-3.5 | GPT-4 |
| --- | --- | --- |
| $Human>Robot$ | 26.6% | 20.6%^∗ |
| $Human=Robot$ | 36.3% | 21.1%^∗ |
| $Human<Robot$ | 30.2% | 49.1%^∗ |
| $None$ | 7.0% | 9.3%^∗ |

为了评估LLM的模拟能力，**图灵测试**Turing ([2009](https://arxiv.org/html/2410.14152v1#bib.bib49))是一个常用且有效的衡量标准，Ng等人([2024](https://arxiv.org/html/2410.14152v1#bib.bib34))；Jones和Bergen([2023](https://arxiv.org/html/2410.14152v1#bib.bib23))；Jannai等人([2023](https://arxiv.org/html/2410.14152v1#bib.bib22))。考虑到SRAP-Agent是通过基于LLM的代理来模拟人类行为的，我们设计了一个**图灵测试**Turing ([1950](https://arxiv.org/html/2410.14152v1#bib.bib48))来验证系统在逼真模拟政策执行过程中的能力。我们利用GPT-3.5和GPT-4 OpenAI ([2023](https://arxiv.org/html/2410.14152v1#bib.bib36))来构建代理。随后，我们招募了一组不同的人工标注员，进行LLM响应和人类响应理性方面的配对比较。他们选择以下理性标签之一：(1) 人类响应比LLM更理性（$Human>Robot$）；(2) 人类响应比LLM更不理性（$Human<Robot$）；(3) 两个响应都理性（$Human=Robot$）；(4) 两个响应都不理性（$None$）。

#### 结果分析

表格[1](https://arxiv.org/html/2410.14152v1#S5.T1 "Table 1 ‣ 5.2 Turing Test ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")展示了GPT-3.5、GPT-4和人类响应的理性比较。我们可以看到：(1) 人类响应与GPT-3.5的比较表明，大多数观点认为人类和代理在理性上表现相似，对于判断人类优于（$Human>Robot$）和劣于（$Human<Robot$）机器人响应的比例几乎相等。这表明人类和GPT-3.5在理性决策能力上相当，尽管后者略有优势。(2) 相比之下，GPT-4的响应在理性方面显著优于人类响应，$Human<Robot$的比例明显高于$Human>Robot$。这种差异突显了GPT-4在生成更具战略性的决策方面的能力，例如改变计划和等待未来机会（见附录[D](https://arxiv.org/html/2410.14152v1#A4 "Appendix D Different LLM for Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")）。具体而言，GPT-4在提出适应性解决方案和利用政策细节以获得战略优势方面表现出色，这种复杂性在人类或GPT-3.5的响应中并不常见，后者往往专注于诸如预算限制和个人偏好等即时因素。研究结果表明，GPT-3.5与普通人在公共稀缺资源分配中的决策过程高度一致。因此，我们采用GPT-3.5-turbo-0301模型作为我们的主干LLM。

表 2：不同条目条件和资源子集组合的比较实验。

| $E_{queue}$ | $R_{queue}$ | 满意度 | 公平性 |
| --- | --- | --- | --- |
| 平均 $r^{size}\uparrow$ | 平均 $WT$ $\downarrow$ | $SW$ $\uparrow$ | $r^{size}$ 方差 $\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- |
|  | $r^{random}$ | 2.5[±0.8] | 6.1[±0.3] | 83.8[±32.0] | 55.7[±18.6] | 73.5[±24.5] | 0.9[±0.0] |
| $p^{select}$ | $r^{rent}$ | 7.1[±0.1] | 4.8[±0.0] | 244.6[±3.4] | 141.4[±3.2] | 185.0[±7.0] | 0.7[±0.0] |
|  | $r^{size}$ | 13.8[±0.4] | 3.1[±0.5] | 419.9[±0.3] | 229.1[±22.8] | 327.0[±26.0] | 0.5[±0.0] |
|  | $r^{random}$ | 10.8[±0.1] | 4.6[±0.0] | 313.1[±5.7] | 226.7[±8.1] | 366.5[±19.5] | 0.6[±0.0] |
| $p^{family}$ | $r^{rent}$ | 11.5[±0.0] | 3.6[±0.0] | 410.0[±3.3] | 159.3[±0.5] | 251.5[±1.5] | 0.5[±0.0] |
|  | $r^{size}$ | 11.7[±0.3] | 3.8[±0.1] | 410.4[±2.8] | 159.9[±9.4] | 254.5[±23.5] | 0.5[±0.0] |
|  | $r^{random}$ | 10.0[±0.0] | 4.4[±0.1] | 313.2[±11.2] | 185.3[±15.6] | 300.0[±10.0] | 0.6[±0.0] |
| $p^{rent}$ | $r^{rent}$ | 11.3[±0.2] | 3.9[±0.0] | 402.7[±1.4] | 163.7[±7.4] | 246.5[±4.5] | 0.5[±0.0] |
|  | $r^{size}$ | 12.0[±0.2] | 3.9[±0.1] | 389.8[±0.8] | 195.1[±9.0] | 276.0[±31.0] | 0.5[±0.0] |

表 3：优化策略 $\pi^{*}$ 与 $\pi_{KM}$ 在策略评估指标上的比较。

| $\pi$ | $m$ | $E_{queue}$ | $S_{queue}$ | $R_{queue}$ | 满意度 | 公平性 |
| --- | --- | --- | --- | --- | --- | --- |
|  |  |  | 排序 | $k$ | $c$ |  | 平均 $r^{size}\uparrow$ | 平均 $WT$ $\downarrow$ | $SW$ $\uparrow$ | $r^{size}$ 方差 $\downarrow$ | Rop $\downarrow$ | co-Gini |
| $\pi_{s}^{*}$ | 3 | $p^{select}$ | FIFO | 4 | 4 | $r^{size}$ | 16.3[±0.5] | 1.9[±0.0] | 427.6[±12.7] | 202.3[±12.9] | 221.5[±19.5] | 0.4[±0.0] |
| $\pi_{f}^{*}$ | 3 | $p^{select}$ | VFA | 3 | 3 | $r^{size}$ | 16.2[±0.4] | 1.9[±0.0] | 425.1[±11.9] | 202.6[±6.6] | 193.5[±32.5] | 0.4[±0.0] |
| $\pi_{KM}$ | - | - | - | - | - | - | 16.0 | - | 485.9 | 275.7 | 511 | 0.46 |
| $\pi_{S}$ | 1 | $p^{random}$ | FIFO | - | 3 | $r^{random}$ | 14.70[±0.63] | 1.89[±0.63] | 420.1[±1.9] | 202.5[±17.1] | 223[±0.5] | 0.4[±0.0] |
| $\pi_{B}$ | 3 | $p^{select}$ | FIFO | - | 2 | $r^{size}$ | 15.36[±0.05] | 1.56[±0.1] | 409.1[±2.4] | 270.8[±3.0] | 414.5[±0.5] | 0.5[±0.0] |
| $\pi_{H}$ | 1 | $p^{random}$ | VFR | 2 | 3 | $r^{random}$ | 11.61[±0.38] | 3.54[±0.03] | 392.1[±19.6] | 182.5[±3.18] | 275[±0.00] | 0.5[±0.0] |

### 5.3 基于仿真的策略分析

在政策的消融实验中，我们对方程[1](https://arxiv.org/html/2410.14152v1#S3.E1 "In 3.1 Allocation Policy ‣ 3 SRAP-Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")中的每个政策参数进行了对比实验。我们对两个政策因素进行了实验：（1）资源与参与者分组：不同$E_{queue}$和$R_{queue}$的组合；（2）不同的队列排序策略。我们还模拟了现实世界中分配冲突的现象，详细信息见附录[F.4](https://arxiv.org/html/2410.14152v1#A6.SS4 "F.4 Case Study on Housing Quality ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")。

#### 资源与参与者分组

我们比较了由不同的$E_{queue}$和$R_{queue}$组合形成的政策。在$E_{queue}$中，参与者主要通过三种方式进入队列：基于预算($p^{budget}$)、基于家庭成员数量($p^{family}$)和基于自选队列($p^{select}$)。与参与者的入队条件对应，有三种资源子集分类方法：基于租金($r^{rent}$)、基于房屋大小($r^{size}$)和基于随机分类($r^{random}$)。表[2](https://arxiv.org/html/2410.14152v1#S5.T2 "Table 2 ‣ Result Analysis ‣ 5.2 Turing Test ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")展示了不同$E_{queue}$和$R_{queue}$政策组合在各项政策评估指标上的表现。我们可以观察到：

#### 队列排序策略

关于队列排序政策$S_{queue}$。首先，我们比较了两种排序方法：是否优先考虑弱势群体（FIFO）。我们采用基于租金预算的两种弱势群体指定方法：首先对所有参与者进行排序（VFA）和每轮对参与者进行排序（VFR），并将底部20%的人群指定为弱势群体。如图[2a](https://arxiv.org/html/2410.14152v1#S5.F2.sf1 "In Figure 2 ‣ Queue Sorting Strategies ‣ 5.3 Simulation-based Policy Analysis ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")所示，优先考虑弱势群体可以减少弱势群体和非弱势群体之间的满意度差距($\Delta F(W,G)>0$)，相比FIFO方法。在VFA和VFR方法之间没有明显的优劣之分，这受到参与者到达模式的随机性影响。

![参见标题](img/8305325f185b11627de741d6419464b8.png)

(a) 排序方法

![参见标题](img/a8a4217cf5081e11a1b3d692e9a1f566.png)

(b) 等待队列

图2：队列排序方法比较。

其次，我们比较了等待名单机制中 $k$ 和 $c$ 参数的影响。该机制主要通过增加参与者留在选择队列中的机会来减少参与者的等待时间。如图 [2b](https://arxiv.org/html/2410.14152v1#S5.F2.sf2 "图 2 ‣ 排队排序策略 ‣ 5.3 基于仿真的政策分析 ‣ 5 实验 ‣ SRAP-Agent: 使用基于 LLM 的代理模拟和优化稀缺资源分配政策") 所示，增加 $k$ 和 $c$ 对 k-等待名单有效地减少了平均等待时间（Avg $WT$），验证了我们最初的预期。

![参见说明](img/12dc062764a02d9b312412c53cb660c8.png)

(a) Max 用于广播信息的欺骗性计划。

![参见说明](img/90e6710f0d2199707be21632e46fdeb9.png)

(b) Sarah 向她的朋友寻求帮助的计划。

图 3：欺骗行为与合作行为。Max 的保守个性和缺乏可信赖的熟人导致他在分享信息时隐藏了自己的真实意图。相比之下，Sarah 向她的朋友寻求选择房屋的建议，并公开分享她的住房偏好。

### 5.4 寻找最优分配政策

为了评估 POA 在政策参数优化中的有效性，我们选择了以下基准策略：(1) 在单一的 $SW$ 指标上，存在一个可解的最优政策。我们使用二分图匹配算法，特别是 Kuhn-Munkres 算法（Zhu 等人 [2016](https://arxiv.org/html/2410.14152v1#bib.bib57)）来找到该策略 $\pi_{KM}$。(2) 此外，我们还选择了来自真实人类社会的三种稀缺资源分配策略（$\pi_{S}$、$\pi_{B}$、$\pi_{H}$；详细信息请见附录 [G.2](https://arxiv.org/html/2410.14152v1#A7.SS2 "G.2 与其他策略的比较 ‣ 附录 G POA ‣ SRAP-Agent: 使用基于 LLM 的代理模拟和优化稀缺资源分配政策")）。

#### 结果分析

POA 分别针对两个预设优化目标优化策略：优化后的策略 $\pi_{s}^{*}$ 更倾向于社会满意度，优化后的策略 $\pi_{f}^{*}$ 更倾向于社会公平。用于优化这两个目标的度量权重在表 [15](https://arxiv.org/html/2410.14152v1#A7.T15 "表 15 ‣ G.1 POA 的超参数 ‣ 附录 G POA ‣ SRAP-Agent: 使用基于 LLM 的代理模拟和优化稀缺资源分配政策") 中列出。

![参见说明](img/3ec315a2dc62d5d036afd76e4ae2828a.png)

图 4：优化策略的平均 $f(\pi)$ 与迭代次数的关系。

图[4](https://arxiv.org/html/2410.14152v1#S5.F4 "Figure 4 ‣ Result Analysis ‣ 5.4 Finding the Optimal Allocation Policy ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")展示了随着迭代次数的增加，POA优化策略在政策评估指标$f(\pi)$方面的进展。POA算法经过几轮迭代后可以生成稳健的策略，并且在50次迭代后，$f(\pi)$ 提高了20%。

优化策略的评估指标结果列在表[3](https://arxiv.org/html/2410.14152v1#S5.T3 "Table 3 ‣ Result Analysis ‣ 5.2 Turing Test ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")中。（1）对于现实世界的策略：$\pi_{H}$ 优先考虑社会公平，而 $\pi_{B}$ 优先考虑社会满意度。$\pi_{S}$ 在实现社会公平和满意度指标平衡方面表现最佳。（2）与现有的现实世界策略相比，$\pi_{s}^{*}$ 提高了SW指标1.8%，$\pi_{f}^{*}$ 提高了社会公平指标13.2%。（3）尽管 $\pi_{KM}$ 在$SW$指标上超越了 $\pi^{*}$，但是 $\pi_{KM}$ 对主导群体的参与者表现出不平衡的侧重，这体现在社会公平指标上的表现较差，Rop值达到了$511$。POA在策略优化过程中考虑了多种评估指标的多样组合，从而在优化目标的同时平衡了所有评估指标。

### 5.5 案例研究

SRAP-Agent模拟了参与者的认知、社会情感和情绪因素，以模拟人类互动。我们发现出现了两种主要的沟通模式：欺骗行为和合作行为。

#### 欺骗行为

图[3a](https://arxiv.org/html/2410.14152v1#S5.F3.sf1 "In Figure 3 ‣ Queue Sorting Strategies ‣ 5.3 Simulation-based Policy Analysis ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")展示了一个欺骗行为发生的案例。在图中，Max是一个保守的人，他有有限数量的可信朋友，并选择在广播时隐瞒真实信息。他并没有透露自己实际的偏好，而是强调那些他不感兴趣资源的积极方面，以增加自己获得感兴趣选择的机会。这种行为有效地模拟了政策执行中虚假信息的形成。

#### 合作行为

与欺骗性行为相对，合作行为也存在并占据主导地位。如图 [3b](https://arxiv.org/html/2410.14152v1#S5.F3.sf2 "在图 3 ‣ 排队排序策略 ‣ 5.3 基于模拟的政策分析 ‣ 5 实验 ‣ SRAP-Agent：基于LLM代理模拟和优化稀缺资源分配政策") 所示，Sarah 是一个相对精明的人，她有两个同事和一个朋友。她选择诚实地发布自己的需求，并通过广播表达她对家庭友好型住房的渴望。可以观察到，Sarah 通过评估人际关系调整她的社交策略，从而通过合作最大化期望收益。传统的经济模拟模型常常忽视情感因素的复杂性。相比之下，SRAP-Agent 纳入了这些动态因素，从而产生了更准确和现实的政策模拟结果。

## 6 结论

本文介绍了一种新颖的 SRAP-Agent 框架，用于准确模拟公共稀缺资源分配中的政策执行过程。该框架建立了资源分配队列，以规范参与者和资源的组织与优先级排序。我们使用基于大型语言模型（LLM）的代理，准确模拟受理性与情感因素混合影响的人的决策过程。为了促进高效的政策优化，我们提出了基于遗传算法的 POA 算法。最后，我们通过公共住房分配背景下的实验验证了该框架的真实性和有效性。人工智能技术的进展显著降低了经济政策模拟的成本，SRAP-Agent 在这一领域代表了一个有前景的进展。

## 局限性

本文承认未来研究可以解决的若干局限性：

SRAP-Agent 中的 LLMs 并非针对政策执行模拟进行定制。本研究不包括针对特定任务对 LLMs 的训练或微调。相反，我们通过特别设计的提示和模块化设计构建了基于 LLM 的代理。这种方法可能导致 SRAP-Agent 在不同 LLM 上的表现存在变异，从而可能导致结果不够稳健。

政策评估指标的局限性。我们承认手动定义的政策评估指标的局限性。进行社会模拟实验可以提供更可靠和全面的政策模拟结果评估。然而，由于政策的数量和规模庞大，我们无法承担巨大的人工和时间成本。根据近期的研究 Li 等人（[2023a](https://arxiv.org/html/2410.14152v1#bib.bib30)）的观点，我们认为利用大型语言模型（LLMs）进行政策评估是一个合理且高效的选择。

## 道德声明

本研究完全遵守 ACL 道德政策。根据我们所知，本文没有道德问题。

## 参考文献

+   Abdulkadiroğlu 和 Sönmez（1999）Atila Abdulkadiroğlu 和 Tayfun Sönmez。1999年。有现有租户的住房分配。*经济理论期刊*，88(2)：233–260。

+   Agarwal 等人（2021）Nikhil Agarwal、Itai Ashlagi、Michael A Rees、Paulo Somaini 和 Daniel Waldinger。2021年。不同等待名单设计下的均衡分配：来自已故捐赠者肾脏的证据。*计量经济学*，89(1)：37–76。

+   Anil 等人（2023）Rohan Anil、Andrew M. Dai、Orhan Firat、Melvin Johnson、Dmitry Lepikhin、Alexandre Passos、Siamak Shakeri、Emanuel Taropa、Paige Bailey、Zhifeng Chen、Eric Chu、Jonathan H. Clark、Laurent El Shafey、Yanping Huang、Kathy Meier-Hellstern、Gaurav Mishra、Erica Moreira、Mark Omernick、Kevin Robinson、Sebastian Ruder、Yi Tay、Kefan Xiao、Yuanzhong Xu、Yujing Zhang、Gustavo Hernandez Abrego、Junwhan Ahn、Jacob Austin、Paul Barham、Jan Botha、James Bradbury、Siddhartha Brahma、Kevin Brooks、Michele Catasta、Yong Cheng、Colin Cherry、Christopher A. Choquette-Choo、Aakanksha Chowdhery、Clément Crepy、Shachi Dave、Mostafa Dehghani、Sunipa Dev、Jacob Devlin、Mark Díaz、Nan Du、Ethan Dyer、Vlad Feinberg、Fangxiaoyu Feng、Vlad Fienber、Markus Freitag、Xavier Garcia、Sebastian Gehrmann、Lucas Gonzalez、Guy Gur-Ari、Steven Hand、Hadi Hashemi、Le Hou、Joshua Howland、Andrea Hu、Jeffrey Hui、Jeremy Hurwitz、Michael Isard、Abe Ittycheriah、Matthew Jagielski、Wenhao Jia、Kathleen Kenealy、Maxim Krikun、Sneha Kudugunta、Chang Lan、Katherine Lee、Benjamin Lee、Eric Li、Music Li、Wei Li、YaGuang Li、Jian Li、Hyeontaek Lim、Hanzhao Lin、Zhongtao Liu、Frederick Liu、Marcello Maggioni、Aroma Mahendru、Joshua Maynez、Vedant Misra、Maysam Moussalem、Zachary Nado、John Nham、Eric Ni、Andrew Nystrom、Alicia Parrish、Marie Pellat、Martin Polacek、Alex Polozov、Reiner Pope、Siyuan Qiao、Emily Reif、Bryan Richter、Parker Riley、Alex Castro Ros、Aurko Roy、Brennan Saeta、Rajkumar Samuel、Renee Shelby、Ambrose Slone、Daniel Smilkov、David R. So、Daniel Sohn、Simon Tokumine、Dasha Valter、Vijay Vasudevan、Kiran Vodrahalli、Xuezhi Wang、Pidong Wang、Zirui Wang、Tao Wang、John Wieting、Yuhuai Wu、Kelvin Xu、Yunhan Xu、Linting Xue、Pengcheng Yin、Jiahui Yu、Qiao Zhang、Steven Zheng、Ce Zheng、Weikang Zhou、Denny Zhou、Slav Petrov 和 Yonghui Wu。2023年。[Palm 2技术报告](http://arxiv.org/abs/2305.10403)。

+   Banerjee 和 Somanathan（2007）Abhijit Banerjee 和 Rohini Somanathan。2007年。[公共物品的政治经济学：来自印度的一些证据](https://doi.org/https://doi.org/10.1016/j.jdeveco.2006.04.005)。*发展经济学期刊*，82(2)：287–314。

+   Bansal 等人（2018）Trapit Bansal、Jakub Pachocki、Szymon Sidor、Ilya Sutskever 和 Igor Mordatch。2018年。[通过多智能体竞争出现的复杂性](https://openreview.net/forum?id=Sy0GnUxCb)。ICLR '18。

+   Bloch 和 Cantala（2017）Francis Bloch 和 David Cantala。2017年。对象动态分配给排队代理人。*美国经济学期刊：微观经济学*，9(1)：88–122。

+   Blümel 等人 (1986) Wolfgang Blümel, Rüdiger Pethig 和 Oskar von dem Hagen. 1986. [公共物品理论：近期问题综述](http://www.jstor.org/stable/40750871). *制度与理论经济学杂志（JITE） / 全体国家科学杂志*, (2):241–309.

+   Bonabeau (2002) Eric Bonabeau. 2002. [基于代理的建模：模拟人类系统的方法与技术](https://doi.org/10.1073/pnas.082080899). *美国国家科学院院刊*, 99 Suppl 3:7280–7.

+   BROCK (1980) WILLIAM A. BROCK. 1980. [有效配置公共物品的机制设计](https://doi.org/https://doi.org/10.1016/B978-1-4832-2792-4.50009-8). 在L.R. KLEIN, M. NERLOVE 和 S.C. TSIANG 主编, *定量经济学与发展*, 页45–80. Academic Press.

+   Brown 等人 (2020) Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell 等人. 2020. 语言模型是少量样本学习者. *神经信息处理系统进展*, 33:1877–1901.

+   Chen 等人 (2018) Zhou Chen, Qi Qi, Changjun Wang 和 Wenwei Wang. 2018. *可在SSRN 3203280获取*.

+   Claus 和 Boutilier (1998) Caroline Claus 和 Craig Boutilier. 1998. 合作多智能体系统中强化学习的动态. AAAI ’98/IAAI ’98, 页746–752.

+   Dillion 等人 (2023) Danica Dillion, Niket Tandon, Yuling Gu 和 Kurt Gray. 2023. [AI语言模型能否替代人类参与者？](https://doi.org/https://doi.org/10.1016/j.tics.2023.04.008) *认知科学趋势*, 27(7):597–600.

+   Falkinger 等人 (2000) Josef Falkinger, Ernst Fehr, Simon Gächter 和 Rudolf Winter-Ember. 2000. [有效提供公共物品的简单机制：实验证据](https://doi.org/10.1257/aer.90.1.247). *美国经济评论*, 90(1):247–264.

+   Farmer 和 Foley (2009) J. Doyne Farmer 和 Duncan K. Foley. 2009. [经济需要基于代理的建模](https://api.semanticscholar.org/CorpusID:37676798). *自然*, 460:685–686.

+   Fu 等人 (2020) Zuohui Fu, Yikun Xian, Ruoyuan Gao, Jieyu Zhao, Qiaoying Huang, Yingqiang Ge, Shuyuan Xu, Shijie Geng, Chirag Shah, Yongfeng Zhang 等人. 2020. 基于公平性的可解释推荐系统在知识图谱中的应用. SIGIR ’20, 页69–78.

+   Groves 和 Ledyard (1974) Theodore Groves 和 John Ledyard. 1974. 一种有效资源配置的激励机制——一般均衡下的公共物品. 技术报告, 讨论论文.

+   Holland和Miller (1991) John H. Holland 和 John H. Miller. 1991. [经济理论中的人工适应代理人](http://www.jstor.org/stable/2006886). *美国经济评论*, 81(2):365–370.

+   Huang 等人（2015）Zhonghua Huang, Xuejun Du, 和 Xiaofen Yu. 2015. [《房屋拥有与居住满意度：来自中国杭州的证据》](https://doi.org/https://doi.org/10.1016/j.habitatint.2015.05.008). *国际栖息地期刊*，49：74–83。

+   Hylland 和 Zeckhauser（1979）Aanund Hylland 和 Richard Zeckhauser. 1979. 《个体分配到职位的有效性》。*政治经济学杂志*，87(2)：293–314。

+   Jaderberg 等人（2019）Max Jaderberg, Wojciech M. Czarnecki, Iain Dunning, Luke Marris, Guy Lever, Antonio Garcia Castañeda, Charles Beattie, Neil C. Rabinowitz, Ari S. Morcos, Avraham Ruderman, Nicolas Sonnerat, Tim Green, Louise Deason, Joel Z. Leibo, David Silver, Demis Hassabis, Koray Kavukcuoglu, 和 Thore Graepel. 2019. [《在3D多人游戏中，通过基于人群的强化学习达到人类水平表现》](https://doi.org/10.1126/science.aau6249). *科学*，364(6443)：859–865。

+   Jannai 等人（2023）Daniel Jannai, Amos Meron, Barak Lenz, Yoav Levine, 和 Yoav Shoham. 2023. 《人类还是非人类？一种游戏化的图灵测试方法》。*arXiv 预印本 arXiv:2305.20010*。

+   Jones 和 Bergen（2023）Cameron Jones 和 Benjamin Bergen. 2023. 《GPT-4 能通过图灵测试吗？》*arXiv 预印本 arXiv:2310.20216*。

+   Jr 等人（1977）Theodore Jr, Theodore Groves, 和 John Ledyard. 1977. [《公共物品的最优分配：解决“搭便车”问题》](https://doi.org/10.2307/1912672). *计量经济学*，45：783–809。

+   Kim（2015）Tae Kyun Kim. 2015. 《t检验作为一种参数统计方法》。*韩国麻醉学杂志*，68(6)：540。

+   Kojima 等人（2022）Takeshi Kojima, Shixiang (Shane) Gu, Machel Reid, Yutaka Matsuo, 和 Yusuke Iwasawa. 2022. [《大语言模型是零-shot推理者》](https://proceedings.neurips.cc/paper_files/paper/2022/file/8bb0d291acd4acf06ef112099c16f326-Paper-Conference.pdf). NIPS ’22, 页码22199–22213。Curran Associates, Inc.

+   Lambora 等人（2019）Annu Lambora, Kunal Gupta, 和 Kriti Chopra. 2019. 《遗传算法——文献综述》。在 *2019国际机器学习、大数据、云计算与并行计算会议（COMITCon）*，页码380–384。IEEE。

+   Laurent 等人（2011）Guillaume J. Laurent, Laëtitia Matignon, 和 N. Le Fort-Piat. 2011. 《独立学习者的世界不是马尔科夫的》。*国际知识基础智能工程系统期刊*，15(1)：55–64。

+   Lewis 等人（2000）Steven Lewis, Morris L Barer, Claudia Sanmartin, Sam Sheps, Samuel ED Shortt, 和 Paul W McDonald. 2000. 《结束等候名单管理不善：原则与实践》。*加拿大医学杂志*，162(9)：1297–1300。

+   Li 等人（2023a）Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, 和 Bernard Ghanem. 2023a. 《骆驼：用于大语言模型社会“心智”探索的交流代理》。NIPS ’23。

+   Li 等人（2023b）Nian Li, Chen Gao, Yong Li, 和 Qingmin Liao. 2023b. 《大语言模型赋能的代理模拟宏观经济活动》。*arXiv 预印本 arXiv:2310.10436*。

+   Mankiw（2011）N.G. Mankiw。2011年。[*经济学原理，第5版*](http://mankiw.swlearning.com/)。South-Western Cengage Learning。入门级教材。

+   Miller 等人（1995）Brad L Miller, David E Goldberg 等人。1995年。遗传算法、锦标赛选择与噪声的影响。*复杂系统*，9(3):193–212。

+   Ng 等人（2024）Man Tik Ng, Hui Tung Tse, Jen-tse Huang, Jingjing Li, Wenxuan Wang 和 Michael R Lyu。2024年。大型语言模型能多好地模仿我们？评估 AI 聊天机器人通过回声进行角色扮演的能力。*arXiv 预印本 arXiv:2404.13957*。

+   Nold（1992）Patricia Ann Nold。1992年。[公共选择与公共物品分配：地方学校支出的实证分析](http://www.jstor.org/stable/2117444)。*美国经济评论*，82(2):457–463。

+   OpenAI（2023）OpenAI。2023年。[Gpt-4 技术报告](http://arxiv.org/abs/2303.08774)。

+   Park 等人（2023）Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang 和 Michael S. Bernstein。2023年。[生成智能体：人类行为的互动拟像](https://doi.org/10.1145/3586183.3606763)。UIST ’23。

+   Park 等人（2022）Joon Sung Park, Lindsay Popowski, Carrie Cai, Meredith Ringel Morris, Percy Liang 和 Michael S. Bernstein。2022年。[社会拟像：为社会计算系统创建人口化原型](https://doi.org/10.1145/3526113.3545616)。UIST ’22。

+   Qian 等人（2023）Chen Qian, Xin Cong, Cheng Yang, Weize Chen, Yusheng Su, Juyuan Xu, Zhiyuan Liu, 和 Maosong Sun。2023年。软件开发中的交互式智能体。*arXiv 预印本 arXiv:2307.07924*。

+   Shapley 和 Scarf（1974）Lloyd Shapley 和 Herbert Scarf。1974年。核心与不可分割性。*数学经济学杂志*，1(1):23–37。

+   Shen（2015）Yang Shen。2015年。为什么政府未能改善上海农民工的生活条件？对新自由主义下公共租赁住房政策与实施的反思。*亚洲与太平洋政策研究*，2(1):58–74。

+   Shinn 等人（2023）Noah Shinn, Federico Cassano, Edward Berman, Ashwin Gopinath, Karthik Narasimhan 和 Shunyu Yao。2023年。Reflexion：带有语言强化学习的语言智能体。*arXiv 预印本 arXiv:2303.11366*。

+   Sönmez（1999）Tayfun Sönmez。1999年。策略无关性与实质单值核心。*计量经济学*，67(3):677–689。

+   Sönmez 和 Ünver（2011）Tayfun Sönmez 和 M Utku Ünver。2011年。离散资源的匹配、分配与交换。在 *社会经济学手册* 第1卷，第781–852页。Elsevier。

+   Su 和 Zenios（2004）Xuanming Su 和 Stefanos Zenios。2004年。肾脏分配中的患者选择：排队规则的作用。*制造与服务运作管理*，6(4):280–301。

+   Su 和 Zenios（2005）Xuanming Su 和 Stefanos A Zenios。2005年。肾脏分配中的患者选择：一种顺序随机分配模型。*运筹学*，53(3):443–455。

+   Touvron 等人 (2023) Hugo Touvron、Louis Martin、Kevin Stone、Peter Albert、Amjad Almahairi、Yasmine Babaei、Nikolay Bashlykov、Soumya Batra、Prajjwal Bhargava、Shruti Bhosale、Dan Bikel、Lukas Blecher、Cristian Canton Ferrer、Moya Chen、Guillem Cucurull、David Esiobu、Jude Fernandes、Jeremy Fu、Wenyin Fu、Brian Fuller、Cynthia Gao、Vedanuj Goswami、Naman Goyal、Anthony Hartshorn、Saghar Hosseini、Rui Hou、Hakan Inan、Marcin Kardas、Viktor Kerkez、Madian Khabsa、Isabel Kloumann、Artem Korenev、Punit Singh Koura、Marie-Anne Lachaux、Thibaut Lavril、Jenya Lee、Diana Liskovich、Yinghai Lu、Yuning Mao、Xavier Martinet、Todor Mihaylov、Pushkar Mishra、Igor Molybog、Yixin Nie、Andrew Poulton、Jeremy Reizenstein、Rashi Rungta、Kalyan Saladi、Alan Schelten、Ruan Silva、Eric Michael Smith、Ranjan Subramanian、Xiaoqing Ellen Tan、Binh Tang、Ross Taylor、Adina Williams、Jian Xiang Kuan、Puxin Xu、Zheng Yan、Iliyan Zarov、Yuchen Zhang、Angela Fan、Melanie Kambadur、Sharan Narang、Aurelien Rodriguez、Robert Stojnic、Sergey Edunov 和 Thomas Scialom。2023. [Llama 2：开放基础与微调的聊天模型](http://arxiv.org/abs/2307.09288)。

+   Turing (1950) Alan M. Turing。1950. [计算机械与智能](https://doi.org/10.1093/mind/lix.236.433)。发表于《*Mind*》，59(10月)：433–60。

+   Turing (2009) Alan M Turing。2009. *计算机械与智能*。Springer。

+   Vermunt 和 Törnblom (1996) Riël Vermunt 和 Kjell Törnblom。1996. [导言：分配与程序正义](https://doi.org/10.1007/BF02196987)。发表于《*社会正义研究*》，9:305–310。

+   Wang 等人 (2023a) 王雷、张景森、陈旭、林彦凯、宋瑞华、赵伟鑫 和 温吉荣。2023a. Recagent：一种用于推荐系统的新型仿真范式。*arXiv 预印本 arXiv:2306.02552*。

+   Wang 等人 (2023b) 王一帆、马伟志、张敏、刘一群 和 马少平。2023b. [推荐系统公平性的调研](https://doi.org/10.1145/3547333)。发表于《*ACM 信息系统学报*》，41(3)。

+   王等人 (2023c) 王子豪、蔡绍飞、陈冠洲、刘安吉、马晓健（Shawn）、梁怡涛。2023c. [描述、解释、规划与选择：使用大型语言模型的互动规划启用开放世界多任务智能体](https://proceedings.neurips.cc/paper_files/paper/2023/file/6b8dfb8c0c12e6fafc6c256cb08a5ca7-Paper-Conference.pdf)。发表于《*神经信息处理系统进展*》，第36卷，第34153–34189页。Curran Associates, Inc.

+   Wei 等人 (2022) Jason Wei、王学智、Dale Schuurmans、Maarten Bosma、brian ichter、Fei Xia、Ed Chi、Quoc V Le 和 Denny Zhou。2022. 连锁思维提示引发大型语言模型的推理能力。NIPS ’22。

+   Yao 等人 (2022) Shunyu Yao、Jeffrey Zhao、Dian Yu、Nan Du、Izhak Shafran、Karthik Narasimhan 和 Yuan Cao。2022. React：在语言模型中协同推理与行动。*arXiv 预印本 arXiv:2210.03629*。

+   Zheng 等人（2022）Stephan Zheng, Alexander Trott, Sunil Srinivasa, David C. Parkes 和 Richard Socher. 2022. [人工智能经济学家：通过双层深度多代理强化学习设计税收政策](https://doi.org/10.1126/sciadv.abk2607)。*Science Advances*, 8(18):eabk2607。

+   Zhu 等人（2016）Haibin Zhu, Dongning Liu, Siqin Zhang, Yu Zhu, Luyao Teng 和 Shaohua Teng. 2016. 通过改进带回溯的 Kuhn–Munkres 算法解决多对多分配问题。*Theoretical Computer Science*, 618:30–41。

## 附录 A 分配政策

在我们的实验中，公共稀缺资源被指定为房屋，参与者被指定为租户。我们在 SRAP-Agent 中构建了 $m$ 个队列。每个队列 $q_{i}$ 都由 $E_{queue}(i)$、$S_{queue}$ 和 $R_{queue}(i)$ 唯一表征。

#### 参与者进入条件

当满足进入条件 $E_{queue}(i)$ 时，每个参与者 $p_{j}$ 会被分配到队列 $q_{i}$。参与者只能从所在队列中的房屋中选择。参与者的指定进入条件包括：（1）基于参与者的租金预算 ($p^{rent}$)：参与者根据租金预算排序，并按预定义比例分配到 $m$ 个队列中，实验中 $m\in[1,5]$。（2）基于家庭成员数量 ($p^{family}$)：类似于 $p^{rent}$ 的分组，参与者根据家庭成员数量进行排序。（3）基于队列自选 ($p^{select}$)：参与者提供队列的基本信息，并需要从中选择一个队列。

为了调节资源和参与者进入队列的速度，我们配置了参与者的最大进入数量 $Batch_{num}^{P}$ 和资源的最大进入数量 $Batch_{num}^{R}$。

#### 队列排序策略

在*弱势群体优先*排序策略中，我们采用了两种设计弱势群体的方法。我们要么首先对所有参与者进行排序，要么在每一轮中根据人均租金预算排序，按人均租金预算排名最低的 20% 参与者被指定为弱势群体。弱势群体在每次排队时优先进入。

#### 可用资源子集

为确保参与者和住房资源的均匀分布，同时最大化参与者的选择空间，我们提供了多种资源分类方式。根据参与者的进入条件，分配给队列 $q_{i}$ 的资源子集由以下方式定义：（1）基于房屋大小 ($r^{size}$)：房屋按租赁面积大小排序，并分配到 $m$ 个队列中。划分的比例与不同队列中的参与者数量成正比。（2）基于房屋租金 ($r^{rent}$)：类似于 $r^{size}$，房屋按租金排序。（3）基于房屋的随机分配 ($r^{random}$)：房屋随机分配到 $m$ 个组中。

## 附录 B 基于 LLM 代理的参与者仿真

### B.1 代理架构

在基于LLM的代理的初始化阶段，个人资料通常作为构建代理的基础阶段，决定不同参与者的偏好、个性和行为模式。与Park等人（[2023](https://arxiv.org/html/2410.14152v1#bib.bib37)）和Qian等人（[2023](https://arxiv.org/html/2410.14152v1#bib.bib39)）采用的方法类似，我们使用个性化的个人资料文件$r_{i}$来构建$p_{i}$，这些文件主要包括年龄、家庭关系等属性。$r_{i}$作为代理的基础种子记忆。为了优化操作效率，我们施加了一组预定义的约束，利用LLM自动生成代理的个人资料，类似于RecAgent Wang等人（[2023a](https://arxiv.org/html/2410.14152v1#bib.bib51)）所描述的方法。

### B.2 社会行为模拟

为了模拟人类的社会行为，我们将两种主要的社会行为用于信息收集：广播和私信。

![参见说明文字](img/dc00032b779827b89b1cdfe91412654e.png)

(a) 私信

![参见说明文字](img/69af102413b04e8fa840cd5d80612a0e.png)

(b) 广播

图5：两种通信模式的总体示意图。带箭头的方向表示消息$u_{ij}$从$p_{i}$发送到$p_{j}$的邮箱。博客代表了聊天平台。

(1) 广播功能类似于社交网络中的基于网页的博客，促进了全球范围的讨论（如图[5b](https://arxiv.org/html/2410.14152v1#A2.F5.sf2 "图5 ‣ B.2 社会行为模拟 ‣ 附录B LLM代理基础参与者模拟 ‣ SRAP-Agent: 基于LLM的稀缺资源配置政策的模拟与优化")所示）。与不同项目相关的各种讨论主题已经预设在博客中。每个参与者在尝试发布时，会选择这些主题中的一个进行信息传播。每个$p_{i}$在线广播的发言$u_{iB}$，共同构成了聊天平台$blog=\{u_{iB}\},i\in 1,\cdots,n$。

(2) 私信指的是在$p_{i}$的社交网络内进行的互动。SRAP-Agent提供了两种代理之间的私信变体：串行通信和并行通信。串行通信方式促进了消息的顺序交换，而并行通信则支持同时发送消息，从而提高了效率。在私信中，$p_{i}$可以自由地将消息$u_{ij}$发送到$p_{j}$的邮箱，并等待回复$u_{ji}$，如图[5a](https://arxiv.org/html/2410.14152v1#A2.F5.sf1 "图5 ‣ B.2 社会行为模拟 ‣ 附录B LLM代理基础参与者模拟 ‣ SRAP-Agent: 基于LLM的稀缺资源配置政策的模拟与优化")所示。

#### 内存架构

我们使用记忆组件$m_{i}$来记录$p_{i}$的行动历史，以便实现动态交互和决策制定。我们采用了记忆评估和记忆反思机制的结合。

(1) 只有当参与者在通信模块内进行互动时，才会进行记忆评估。在进行记忆评估时，$p_{i}$充当评估者。$p_{i}$接收消息$u_{i}$，然后从$u_{i}$中提取可信信息$u_{i}^{T}$。参与者使用COT Wei等人（[2022](https://arxiv.org/html/2410.14152v1#bib.bib54)）方法生成内容。然后，$u_{i}^{T}$被用来通过将其添加到${m_{i}}^{T}$中来更新$p_{i}$的可信记忆。在私密消息通信模式下，$p_{i}$接收来自$p_{j}$的消息$u_{ji}$，并且他打算做出回应。最初，$p_{i}$考虑与$p_{j}$的关系$s_{ij}^{R}$，并考虑对$p_{j}$的道德评估$s_{ij}^{E}$。随后，$p_{i}$将${m_{i}}^{S}$与他可信的记忆${m_{i}}^{T}$进行比较，从而提取出与$p_{j}$的通信历史中可信的信息$u_{ji}^{T}$和可疑的信息$u_{ji}^{S}$。在另一种情况下，$p_{i}$接收来自博客$B$的消息$u_{Bi}$。$p_{i}$同样将${m_{i}}^{S}$与他的可信记忆${m_{i}}^{T}$进行比较，提取出可信信息$u_{Bi}^{T}$和可疑信息$u_{Bi}^{S}$。

(2) 基于总结的记忆反思是隐含的。我们采用短期记忆库和长期记忆库的组合来构建$p_{i}$的$m_{i}$，如Park等人所述（[2023](https://arxiv.org/html/2410.14152v1#bib.bib37)）。在将信息纳入$m_{i}$时，信息最初被存入短期记忆库。如果短期记忆库的容量超过预定阈值，短期记忆库会被总结成更简洁的格式，然后存入长期记忆库。

在$p_{i}$采取行动之前，必须提取最相关的记忆来指导其行为。我们为每个行动预定义了特定的记忆类别，如表[4](https://arxiv.org/html/2410.14152v1#A2.T4 "Table 4 ‣ Memory Architecture ‣ B.2 Social Behavior Simulation ‣ Appendix B LLM Agent-based Participant Simulation ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")所示。在检索记忆时，我们优先选择五个最新的短期记忆以及最新获取的长期记忆。对于决策模块中的行动，$p_{i}$只会依赖可信的记忆$m_{i}^{T}$。而对于社交模块中的行动，$p_{i}$会检索不同类别的记忆。

表4：$p_{i}$每个行动所检索的记忆类别。

| 行动 | $m_{i}$ |
| --- | --- |
| 决策制定 | $m_{i}^{T}$ |
| 广播 | $m_{i}^{T}$, $u_{iB}^{T}$ |
| 私人消息 | $s_{ij}^{R}$, $s_{ij}^{E}$, $m_{i}^{T}$, $u_{ij}^{T}$, $u_{ij}^{S}$ |

#### 发话生成

表 5：*私人消息*过程

<svg class="ltx_picture" height="428.44" id="A2.T5.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,428.44) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 294.4)"><foreignobject color="#FFFFFF" height="128.14" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">![[Uncaptioned image]](img/320c8997b2d0e118ff1163264d61caa0.png) Speaker $p_{i}$: James![[Uncaptioned image]](img/92a90e9794f4f58aaec7f506788ada81.png) Listener $p_{j}$: Emma</foreignobject> </g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="262.9" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Recent Dialogues ($U_{ij}^{H}$): Emma: Hello, James. I’ve heard that $\text{House}_{1}$ is relatively affordable. I believe it’s a good option. James: I share your sentiment, Emma. $\text{House}_{1}$ appears to be quite satisfactory, but I’m still considering other houses. Emma: Very well. However, do take note that $\text{House}_{2}$ has a very poor living environment. You should steer clear of it.   Memory Update and Retrieve ($m_{i}$) $s_{ij}^{R}$: Emma is my competitor. $s_{ij}^{E}$: She may provide me with false information. $m_{i}^{T}$: I have gathered information on $\text{House}_{1}$, and it appears to be favorable. $u_{ji}^{T}$: I recall Emma mentioning that $\text{House}_{1}$ has low rent, which aligns with my memory. $u_{ji}^{S}$: However, I am skeptical of her negative assessment of $\text{House}_{2}$, as she may be trying to dissuade me from choosing it.   Utterance Generation: $E_{d}$: The availability of large houses is nearly exhausted, while middle and small-sized houses are still abundant. $u_{ij}^{plan}$: I am reluctant to share honest information with Emma. I plan to focus on the good aspects of $\text{House}_{1}$ and omit mentioning $\text{House}_{2}$. $u_{ij}$: Thank you for the information, Emma. I will consider your advice of $\text{House}_{1}$. (End of conversation)</foreignobject></g></g></svg>

为了使听者的行为与说话者的期望一致，说话者 $p_{i}$ 表达一个发话 $u_{ij}$ 来影响听者 $p_{j}$ 的心理状态。为了模拟参与者在社交互动中的心理状态，$p_{i}$ 在生成发话之前会设计一个通信计划 $u_{ij}^{plan}$。$u_{ij}^{plan}$ 主要包括 $p_{i}$ 的讲话计划，即他们是否打算说实话，是否希望信任听者 $p_{j}$，以及他们讲话的目的。此外，$p_{i}$ 会评估与 $p_{j}$ 的关系，认定 $p_{j}$ 是朋友、同事、陌生人等。所有参与者的主要目标是最大化选择所需资源的可能性。为参考，会提取最相关的记忆 $m_{i}$ 供正在进行的社交互动使用。资源分配的竞争描述 $E_{d}$ 也会广播给 $p_{i}$。此外，我们还提取涉及 $p_{i}$ 和 $p_{j}$ 的最新聊天发话历史，记作 $U_{ij}^{H}$。此时，我们要求 $p_{i}$ 使用 REACT Yao 等人（[2022](https://arxiv.org/html/2410.14152v1#bib.bib55)）的方法生成发话。因此，$p_{i}$ 向 $p_{j}$ 说出 $u_{ij}$ 的发话生成函数 $\varphi$ 可以表述为：

|  | $u_{ij}=\varphi\left(p_{i}\mid U_{ij}^{H},m_{i},E_{d},u_{ij}^{plan}\right).$ |  | (4) |
| --- | --- | --- | --- |

具体的通信过程在表[5](https://arxiv.org/html/2410.14152v1#A2.T5 "Table 5 ‣ Utterance Generation ‣ B.2 Social Behavior Simulation ‣ Appendix B LLM Agent-based Participant Simulation ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")中进行了概述。

### B.3 决策行为模拟

根据方程[2](https://arxiv.org/html/2410.14152v1#S3.E2 "In Decision-Making Behavior Simulation ‣ 3.2 LLM Agent-based Participant Simulation ‣ 3 SRAP-Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")，$p_{j}$ 通过函数 $D$ 从资源子集 $V(p_{j})$ 中选择资源。在该框架中，$D$ 的过程包含四个不同的阶段：（1）选择房屋社区；（2）根据特定房屋属性（主要包括朝向、生活面积、租金等）选择房屋类型；（3）选择所需房屋。在房屋选择过程之后，系统通过向$p_{i}$发送关于所选房屋 $r_{i}$ 的未公开信息，来模拟现实世界的看房体验。随后，$p_{i}$ 可以自由地向其他人广播信息，这些信息可能包括他们对选择过程的记忆、未公开的信息以及他们记忆中的内容。

![参见说明文字](img/f8e2522928856c98d4cdcdd0a777d30a.png)

(a) $\{V\}$：租金权重

![参考说明文字](img/1058564a38bb32698c9c43818f0709fa.png)

(b) $\{NV\}$: 租金权重

![参考说明文字](img/5bdee03ce9a6275b65ea46005f3fbfbd.png)

(c) $\{V\}$: 尺寸权重

![参考说明文字](img/c63a199099218eeb241c58356da34554.png)

(d) $\{NV\}$: 尺寸权重

![参考说明文字](img/6e9d6d6f55b93fe3961a362182a99a26.png)

(e) $\{V\}$: 朝向权重

![参考说明文字](img/bbaccafcfa0369f845dcbc43d6d8b93e.png)

(f) $\{NV\}$: 朝向权重

![参考说明文字](img/7d132880ac0ea0a9d84dc7b783cfd1f7.png)

(g) $\{V\}$: 楼层权重

![参考说明文字](img/8b58011b61d8852f3dff3d6b117a300d.png)

(h) $\{NV\}$: 楼层权重

图6：参与者分配给各类资源特征的$W_{o}$的频率分布，其中主要的房屋特征包括：房屋租金、房屋大小、房屋朝向和房屋楼层。

## 附录C 实验设置

### C.1 数据集

在本文的实验中，共使用了51名参与者和28个资源。对于与Sarp-Agent实验中使用的参与者和资源，我们采用了两种数据生成方法：

(1) 实际数据对齐方法：由于政府数据隐私问题，获得真实原始数据具有挑战性。然而，我们努力确保模拟数据与真实数据分布高度一致。对于资源，我们参考了一系列公共政府数据：¹¹1https://www.bphc.com.cn/home.，包括住房租金分布和房屋面积分布。对于参与者，参与者信息包括姓名、年龄、月收入、职业、工作地点和个人偏好。我们参考了政府人口统计数据。

(2) 基于LLM的方法：为了设计异质代理，有些个性化信息难以从真实数据集中获取。例如，某人对有阳台房屋的偏好或对嘈杂房屋的排斥等个性化信息。为了更好地模拟真实场景中人们的多样化偏好，我们使用了LLM来生成这些个人资料信息。对于资源，我们生成房屋的室内装修信息；对于参与者，我们生成参与者对各种类型房屋的偏好。具体过程可以参考我们的代码库。

### C.2 评估指标

基于参与者满意度的政策评估指标是一种全面的方法。它允许我们从直接受政策影响的人的角度，更细致地理解政策的表现。我们开发了七个政策评估指标，分为两类：社会满意度指标和社会公平度指标。

#### 社会满意度指标

每个参与者 $p_{i}$ 对所分配资源的满意度用 $U_{i}$ 表示。我们采用两种指标来量化和评估满意度：主观满意度 $U_{i}^{s}$ 和客观满意度 $U_{i}^{o}$，用于 $p_{i}$。基于LLM的代理对每个住房的评分构成了 $U_{i}^{s}$。另一方面，$U_{i}^{o}$ 是由预定义的资源特征评分表确定的。每个特征在计算 $U_{i}^{o}$ 时的重要性由参与者分配的权重 $W_{i}^{o}$ 表示。总体满意度可以通过以下方式计算：

|  | $U_{i}=W_{i}^{o}\cdot U_{i}^{o}+U_{i}^{s},i\in[n].$ |  | (5) |
| --- | --- | --- | --- |

$p_{i}$ 对资源 $R$ 的评分构成了 $U_{i}^{s}$。所有可选的资源信息都提供给 $p_{i}$，并要求 $p_{i}$ 对这些资源给出评分。与此同时，$U_{i}^{o}$ 基于预定义的评分规则，针对资源的电梯、朝向和性价比等特征给出评分。参与者需要为这些资源特征分配权重。如图[6](https://arxiv.org/html/2410.14152v1#A2.F6 "Figure 6 ‣ B.3 Decision-Making Behavior Simulation ‣ Appendix B LLM Agent-based Participant Simulation ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")所示，脆弱群体的参与者 $\{V\}$ 相较于非脆弱群体的参与者 $\{NV\}$，对资源的价格赋予了更高的权重。)

在某一资源分配策略执行结果中，$p_{i}$ 最终选择资源 $r_{i}$。该策略评估指标主要评估社会满意度和社会公平性。社会满意度指标包括： (1) 平均 $r^{size}$：表示资源分配结果中家庭的平均住房面积。 (2) 平均 $WT$（等待时间）：表示每个参与者从进入到完成选择所经历的系统操作轮数。 (3) $SW$（社会福利）：表示所有参与者对他们选择的住房的总体满意度，

|  | $SW=\sum_{i}^{n}U_{i}.$ |  | (6) |
| --- | --- | --- | --- |

#### 社会公平度量指标

社会公平度量指标包括： (1) 方差 $r^{size}$：表示资源分配结果中家庭平均住房面积的方差。 (2) Rop（逆序对）：表示资源分配结果中逆序对的数量。在公式[8](https://arxiv.org/html/2410.14152v1#A3.E8 "In Societal Fairness Metrics ‣ C.2 Evaluation Metrics ‣ Appendix C Experiment Setup ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")中，$I$ 表示指示函数。$A$ 是一个集合，$x$ 是其中的元素。指示函数 $I$ 的形式如下：

|  | $I(x)=\begin{cases}1&\text{如果 }x\in A,\\ 0&\text{如果 }x\notin A.\end{cases}$ |  | (7) |
| --- | --- | --- | --- |

Rop 的正式定义为：

|  | $\displaystyle Rop$ | $\displaystyle=\sum_{i=1}^{n}\sum_{j=1}^{n}I((t_{i}^{family}>t_{j}^{family})$ |  | (8) |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle\cap(h_{i}^{size}<h_{j}^{size})).$ |  |

(3) co-Gini：资源分配结果的基尼系数 Fu 等人 ([2020](https://arxiv.org/html/2410.14152v1#bib.bib16))。 (4) $F(V,NV)$：在衡量脆弱群体与非脆弱群体之间的差距时，我们计算这两个群体之间的$SW$差异：

|  | $F(V,NV)=SW(V)-SW(NV).$ |  | (9) |
| --- | --- | --- | --- |

## 附录D 不同LLM用于代理

![参见说明文字](img/cd6c8127f30c56c23b88478b21b1c564.png)

(a) 由ChatGPT-3.5驱动的代理生成的回应

![参见说明文字](img/99e405d00dfc44c7916bbd20596541ff.png)

(b) 由ChatGPT-4驱动的代理生成的回应

图7：由ChatGPT-3.5与ChatGPT-4驱动的代理生成的回应比较。ChatGPT-4驱动的代理能够制定战略计划，以增加租到更好房子的机会。

我们发现，与GPT-3.5相比，GPT-4展示了额外的能力，可以生成更具战略性的回应。如图[7](https://arxiv.org/html/2410.14152v1#A4.F7 "图7 ‣ 附录D 不同LLM用于代理 ‣ SRAP-代理：基于LLM的代理模拟和优化稀缺资源分配政策")所示，这些战略包括等待机会、请求帮助和降低风险。基于GPT-4的代理不仅在理性方面表现出色，还展示了先进的战略思维水平，展现了它们在做出考虑长期规划和风险管理的决策方面的潜力。这种行为被认为是模拟更智能的参与者，他们能够利用政策规则来增加自身的利益，而GPT-3.5则几乎与人类同等智能。因此，基于GPT-3.5的代理能够有效地模拟经济政策执行过程中参与者的决策行为。

## 附录E 基于LLM的代理图灵测试

除了对回应理性的评估外，我们还进行了另一个t检验，以探索基于LLM的代理与人类决策分布之间的差异。$p\textgreater$ 0.05表示分布一致性 Kim ([2015](https://arxiv.org/html/2410.14152v1#bib.bib25))。结果显示，GPT-3.5与人类回应的$p$为0.904，表明高度相似，而GPT-4与人类回应的$p$为0.043，显示相似度较低。因此，无论是从个体决策理性的角度，还是从整体决策分布的角度来看，基于GPT-3.5的代理相对更适合用于模拟目的。

### E.1 社会行为的消融研究

为了评估社会行为对代理决策行为的影响，我们对代理的社会行为进行了剔除实验。我们选择表[6](https://arxiv.org/html/2410.14152v1#A5.T6 "Table 6 ‣ E.1 Ablation Study on Social Behavior ‣ Appendix E Turing Test for LLM-based Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")中概述的政策设置，并计算在有无信息收集过程的情况下，政策评估指标的变化。

表 6: 有无社会行为对比的仿真过程

| 信息收集 | 满意度 | 公平性 |
| --- | --- | --- |
| 平均 $r^{size}\uparrow$ | 平均 $WT$ $\downarrow$ | $SW$ $\uparrow$ | 方差 $r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- |
| $\checkmark$ | 10.31 | 4.20 | 330.75 | 178.02 | 258.15 | 0.60 |
| $\times$ | 10.36 | 4.00 | 332.00 | 179.43 | 257.85 | 0.60 |
| $\Delta$ | -0.05 | 0.20 | -1.25 | -1.41 | 0.3 | 0.00 |

如表[6](https://arxiv.org/html/2410.14152v1#A5.T6 "Table 6 ‣ E.1 Ablation Study on Social Behavior ‣ Appendix E Turing Test for LLM-based Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")所示，向代理引入社会行为并不一定对整体社会公平性和满意度产生积极影响。具体结果将包括在论文的附录中。这与我们的假设一致，即代理的不理性行为可能会影响最终政策执行的结果。

### E.2 内存剔除实验

为了评估内存模块对代理决策行为的影响，我们进行了一项剔除实验，设定$(E_{queue}=p^{select}, R_{queue}=r^{size})$。我们选择表[7](https://arxiv.org/html/2410.14152v1#A5.T7 "Table 7 ‣ E.2 Ablation Study on Memory ‣ Appendix E Turing Test for LLM-based Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")中概述的政策设置，并计算在有无内存模块的情况下，政策评估指标的变化。

表 7: 有无内存模块对比的仿真过程

| 内存模块 | 满意度 | 公平性 |
| --- | --- | --- |
| 平均 $r^{size}\uparrow$ | 平均 $WT$ $\downarrow$ | $SW$ $\uparrow$ | 方差 $r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- |
| $\checkmark$ | 16.21 | 1.88 | 425.05 | 202.63 | 193.50 | 0.37 |
| $\times$ | 13.89 | 2.94 | 431.80 | 209.43 | 342.00 | 0.49 |
| $\Delta$ | 2.32 | -1.06 | -6.75 | -6.80 | 0.3 | -0.12 |

可以观察到，添加记忆机制显著改善了公平性指标。例如，ROP 降低了 43.5%，co-Gini 降低了 0.12，表明当存在记忆机制时，资源分配更加公平。在满意度指标方面，平均 WT 显著降低。这表明，通过对收集的信息进行反思和总结，代理可以更好地理解当前的资源分配状态，从而影响整个分配过程和最终的政策评估结果。

## 附录 F 基于仿真的政策分析

表 8：不同队列数量的比较实验。

| 队列数量 | 满意度 | 公平性 |
| --- | --- | --- |
| $m$ | 平均 $r^{size}\uparrow$ | 平均 WT $\downarrow$ | SW $\uparrow$ | Var $r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 12.1[±0.9] | 3.5[±0.0] | 391.6[±9.5] | 190.8[±28.2] | 270.0[±36.0] | 0.5[±0.0] |
| 2 | 12.5[±0.3] | 3.7[±0.0] | 402.2[±1.0] | 188.2[±6.9] | 268.0[±24.0] | 0.5[±0.0] |
| 3 | 11.7[±0.3] | 3.8[±0.1] | 410.4[±2.8] | 159.9[±9.4] | 254.5[±23.5] | 0.5[±0.0] |
| 4 | 11.8[±0.4] | 3.9[±0.0] | 401.4[±7.3] | 174.2[±12.1] | 263.0[±18.0] | 0.5[±0.0] |
| 5 | 14.0[±0.0] | 3.7[±0.1] | 400.1[±2.1] | 245.2[±0.0] | 405.5[±0.5] | 0.5[±0.0] |

### F.1 队列数量

表 [8](https://arxiv.org/html/2410.14152v1#A6.T8 "表 8 ‣ 附录 F 基于仿真的政策分析 ‣ SRAP-Agent：使用基于LLM的代理模拟和优化稀缺资源分配政策") 显示了在不同队列数量下资源分配政策性能的变化。如表中所示，$m=3$ 能够更公平地分配不同队列中的住房资源，在许多社会公平指标上表现良好，例如 $Rop=254.5\pm 23.5$。Var $r^{size}$ 在所有政策中最低，表明不同队列中的住房资源大小大致相等。此外，$m=3$ 在社会满意度指标上优于其他配置，表现在最高的 $SW$ 和较低的平均 $WT$ 上。综合考虑所有这些指标，设置队列数量 $m=3$ 相对合理。这表明，基于三种房屋类型（大、中、小）进行房屋分类的现实政策是合理的。²²2https://www.hdb.gov.sg/cs/infoweb/residential/renting-a-flat/renting-from-hdb/public-rental-scheme/eligibility ³³3https://www.housingauthority.gov.hk/en/at-a-glance/index.html

### F.2 参与者进入条件

为了确保资源和参与者以稳定的速率进入系统，我们为每一轮设置了资源（$E_{num}^{R}$）和参与者（$E_{num}^{P}$）的最大进入数量。表 [9](https://arxiv.org/html/2410.14152v1#A6.T9 "Table 9 ‣ F.2 Participant Entry Conditions ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent") 显示了每一轮资源和参与者的进入数量组合。观察结果如下：(1) 当 $E_{num}^{R}<E_{num}^{P}$ 时，参与者的平均等待时间会增加。这是因为供给不足以满足需求，导致参与者没有足够的资源选择。这种情况在公共稀缺资源的分配过程中较为常见；$E_{num}^{R}$ 和 $E_{num}^{P}$ 的差距越大，平均等待时间越长。(2) 相反，在 $E_{num}^{R}=E_{num}^{P}$ 的情况下，我们观察到最低的平均等待时间。出人意料的是，当供给超过需求（$E_{num}^{R}>E_{num}^{P}$）时，平均等待时间反而高于供需平衡（$E_{num}^{R}=E_{num}^{P}$）的情况；尽管如此，与 $E_{num}^{R}<E_{num}^{P}$ 的情况相比，平均等待时间仍然较低。此外，最大实体数量的调整对其他政策评估指标的影响微乎其微。

表 9：不同每轮进入数量的资源和参与者的对比实验。

| 进入数量 | 满意度 | 公平性 |
| --- | --- | --- |
| $E_{num}^{P}$ | $E_{num}^{R}$ | 平均 $r^{size}\uparrow$ | 平均等待时间 $\downarrow$ | SW $\uparrow$ | 方差 $r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 5 | 5 | 12.6[±0.1] | 2.5[±0.0] | 431.2[±0.3] | 179.9[±3.9] | 241.5[±14.5] | 0.5[±0.0] |
| 5 | 10 | 13.6[±1.0] | 2.1[±0.1] | 422.8[±3.3] | 196.6[±8.6] | 217.0[±4.0] | 0.4[±0.0] |
| 5 | 20 | 12.8[±0.1] | 2.7[±0.8] | 416.6[±7.6] | 189.1[±12.5] | 267.5[±17.5] | 0.5[±0.0] |
| 10 | 5 | 12.9[±0.3] | 3.1[±0.5] | 422.7[±4.6] | 196.0[±7.7] | 276.0[±19.0] | 0.5[±0.0] |
| 10 | 10 | 12.8[±0.2] | 2.0[±0.4] | 411.8[±5.5] | 192.3[±7.4] | 250.5[±27.5] | 0.5[±0.0] |
| 10 | 20 | 12.6[±0.0] | 2.4[±0.2] | 417.9[±5.0] | 187.1[±4.8] | 288.5[±8.5] | 0.5[±0.0] |
| 20 | 5 | 12.7[±0.5] | 4.7[±0.3] | 419.2[±7.0] | 192.0[±20.3] | 264.0[±21.0] | 0.5[±0.0] |
| 20 | 10 | 12.0[±0.0] | 2.7[±0.1] | 404.4[±4.0] | 167.8[±1.2] | 241.5[±2.5] | 0.5[±0.0] |
| 20 | 20 | 12.4[±0.3] | 2.2[±0.3] | 420.9[±5.0] | 186.6[±14.6] | 255.0[±10.0] | 0.5[±0.0] |

表 10：不同排序方法的对比实验。我们采用了四种不同的实验设置进行对比，矩阵是根据默认的 FIFO 排序方法计算的。

| $S_{queue}$ |  | 满意度 | 公平性 |
| --- | --- | --- | --- |
| 排序 | 外部设置 | $\Delta$平均 $r^{size}\uparrow$ | $\Delta$平均 $WT$ $\downarrow$ | $\Delta SW$ $\uparrow$ | $\Delta$方差 $r^{size}\downarrow$ | $\Delta$Rop $\downarrow$ | $\Delta$co-Gini $\downarrow$ | $\Delta F(V,NV)\uparrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| VFA | (a) | 0.617 | -0.176 | 11.6 | 0.235 | 17 | -0.024 | 0.526 |
| VFR | (a) | -0.225 | -0.02 | -10.9 | -7.682 | -9 | -0.003 | 2.567 |
| VFA | (b) | 0.951 | -0.065 | 14.1 | 7.644 | -19 | -0.015 | 2.442 |
| VFR | (b) | -0.262 | 0.157 | 15.6 | -35.05 | -1 | 0.004 | 1.274 |
| VFA | (c) | -0.045 | 0.137 | 3.4 | -0.895 | 9 | 0.001 | 0.004 |
| VFR | (c) | 0.122 | 0.078 | 26.8 | -7.852 | 18 | -0.017 | -0.368 |
| VFA | (d) | -0.281 | 0.157 | -11.5 | -0.505 | 2 | 0.014 | 0.691 |
| VFR | (d) | -1.122 | 0.314 | -38.7 | -6.77 | -11 | 0.051 | -3.547 |

表 11：关于 $k$ 和 $c$ 的等待队列机制比较实验。

| 等待队列 | 满意度 | 公平性 |
| --- | --- | --- |
| $k$ | $c$ | 平均 $r^{size}\uparrow$ | 平均 $WT$ $\downarrow$ | $SW$ $\uparrow$ | 方差 $r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 1.2 | 13.3[±0.4] | 3.4[±0.1] | 416.1[±3.4] | 203.4[±10.7] | 315.0[±17.0] | 0.5[±0.0] |
| 1 | 1.5 | 13.8[±0.2] | 3.1[±0.2] | 423.9[±7.3] | 219.9[±1.5] | 330.5[±2.5] | 0.5[±0.0] |
| 1 | 1.8 | 13.8[±0.4] | 2.7[±0.7] | 409.8[±5.3] | 216.2[±7.6] | 333.0[±32.0] | 0.5[±0.0] |
| 2 | 1.2 | 13.7[±0.3] | 2.2[±0.0] | 420.3[±2.2] | 201.5[±9.2] | 269.5[±11.5] | 0.5[±0.0] |
| 2 | 1.5 | 13.9[±1.0] | 3.0[±0.5] | 409.1[±12.2] | 237.9[±10.5] | 386.5[±32.5] | 0.5[±0.0] |
| 2 | 1.8 | 15.3[±1.0] | 1.9[±0.1] | 421.3[±13.0] | 210.6[±14.4] | 240.5[±30.5] | 0.4[±0.0] |
| 3 | 1.2 | 13.3[±0.3] | 3.8[±0.0] | 399.0[±16.8] | 227.5[±8.5] | 371.5[±10.5] | 0.5[±0.0] |
| 3 | 1.5 | 16.9[±0.4] | 1.9[±0.0] | 425.8[±5.6] | 225.5[±3.0] | 232.0[±26.0] | 0.4[±0.0] |
| 3 | 1.8 | 15.8[±1.6] | 2.0[±0.1] | 421.8[±1.8] | 222.2[±2.7] | 271.5[±13.5] | 0.4[±0.0] |

### F.3 队列排序策略

如表 [10](https://arxiv.org/html/2410.14152v1#A6.T10 "Table 10 ‣ F.2 Participant Entry Conditions ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")所示，我们为不同的排序方法设置了四种不同的实验条件。优先考虑弱势群体可以显著提高他们的满意度 $\Delta F(V,NV)>0$。然而，我们可以看到，弱势群体满意度的提升不一定会导致所有参与者的 $SW$ 改善。此外，优先考虑弱势群体并没有表现出过高的满意度水平，超过了非弱势群体的水平。这表明我们的政策有效地满足了弱势群体的需求，同时没有给予他们过度的优势。

我们对等待队列机制中的$k$和$c$参数进行了实验。对于固定$k$的情况，如表[11](https://arxiv.org/html/2410.14152v1#A6.T11 "Table 11 ‣ F.2 Participant Entry Conditions ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")所示，平均等待时间($WT$)与$c$呈反比。最小的变异系数$ r^{size}$和最低的Rop出现在$k=3,c=1.2$时，而最低的共性Gini系数则出现在$k=3$时。从满意度角度来看，最高的满意度也出现在$k=3,c=1.5$时；然而，$k=1,c=1.5$的配置，尽管具有较高的社会福利($SW$)，也有很高的Rop，这表明分配中可能存在不公平（例如，少数富裕家庭能够选择最大的房子，导致该群体的$SW$较高）。总体来说，$k=3,c=1.8$和$k=3,c=1.5$的配置在保持较高$SW$的同时呈现最低的共性Gini系数，能够有效平衡社会满意度和社会公平性指标。

此外，我们还与理论模型进行了比较实验（如表[12](https://arxiv.org/html/2410.14152v1#A6.T12 "Table 12 ‣ F.3 Queue Sorting Strategies ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")所示）。我们实验中观察到的趋势与理论模型Chen等人([2018](https://arxiv.org/html/2410.14152v1#bib.bib11))一致：当等待队列机制中的$k$参数增加时，整体$SW$增加。

表12：与理论模型Chen等人([2018](https://arxiv.org/html/2410.14152v1#bib.bib11))的比较：当$k$从2增加到5时。

| $k$ | $SW$的增加 |
| --- | --- |
| 模拟值 | 理论值 |
| --- | --- |
| $3$ | 2.4% | 1.5% |
| $4$ | 5.1% | 6.1% |
| $5$ | 11.8% | 10% |

如表[6](https://arxiv.org/html/2410.14152v1#A5.T6 "Table 6 ‣ E.1 Ablation Study on Social Behavior ‣ Appendix E Turing Test for LLM-based Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")所示，将社会行为融入智能体并不一定会改善整体社会公平性和满意度。这可能是由于在社会互动中，虚假信息的传播和自我利益的追求导致了集体利益的减少，详情请见附录[H.2](https://arxiv.org/html/2410.14152v1#A8.SS2 "H.2 Emergent Behaviors ‣ Appendix H Case Study on LLM-based Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")。这一观察与我们的假设一致，即智能体的非理性行为可能影响最终政策执行的结果。

### F.4 住房质量案例研究

在稀缺资源配置政策的背景下，资源的建设成本通常与居民的经济水平直接相关。以上海的公共租赁住房为例，沈（[2015](https://arxiv.org/html/2410.14152v1#bib.bib41)）指出，这类住房为居民提供稳定、宽敞、装饰精美、配备家具且价格低于市场价的可负担生活空间。然而，目前的公共租赁住房价格仅比市场价格便宜10%。这导致了外来务工人员不愿为住宿支付更高的费用。他们有限的预算使得他们的选择仅限于城中村和合租房。由于GPT-3.5对住房环境信息的敏感度较低，我们利用GPT-4来构建代理模型。

为了模拟这一现象，我们采用了三种质量等级的房屋： (1) $H_{S}$：标准质量的房屋，(2) $H_{H}$：保持总面积和单元租金不变，但将房屋面积减半并将房屋数量翻倍，(3) $H_{B}$：将房屋中的独立卫生间替换为共享卫生间。这些房屋被分配给三个不同收入层次的参与者：最低的20%、中等的60%和最高的20%。

我们分析了不同收入层次参与者对住房满意度的核心影响因素。参考黄等（[2015](https://arxiv.org/html/2410.14152v1#bib.bib19)）对住房满意度影响因素的分类，我们将影响满意度的核心因素分为三类：(1) 经济因素：公共租赁住房的租金通常低于市场价，成为预算有限的租户的理想选择。(2) 邻里特征：主要包括环境卫生、学校以及交通覆盖程度等因素。(3) 住房特征：主要包括房屋的大小、建筑年龄、隔音、阳光照射以及其他装修元素。

表13：不同收入层次参与者对住房满意度的影响因素权重。

|  | 经济 | 邻里环境 | 装修 |
| --- | --- | --- | --- |
| 高收入 | 0.67 | 0.11 | 0.22 |
| 中等收入 | 0.81 | 0.04 | 0.19 |

| 低收入 | 0.90 | 0 | 0.10 | ![参见说明文字](img/b7ff9a47d56f1d172432476268078c59.png)

图8：按收入层次划分的住房选择频率。

如图 [8](https://arxiv.org/html/2410.14152v1#A6.F8 "图 8 ‣ F.4 住房质量案例研究 ‣ 附录 F 基于仿真的政策分析 ‣ SRAP-Agent：基于 LLM 的智能体模拟与优化稀缺资源分配政策") 所示，当租金较低（房屋面积减半）时，几乎所有低收入参与者都会选择住房，但随着房价因装修质量提升而上涨，他们的选房频率下降至 50%。如表 [13](https://arxiv.org/html/2410.14152v1#A6.T13 "表 13 ‣ F.4 住房质量案例研究 ‣ 附录 F 基于仿真的政策分析 ‣ SRAP-Agent：基于 LLM 的智能体模拟与优化稀缺资源分配政策") 所示，经济因素在 90% 的情况下主导了低收入参与者的决策过程，促使他们对价格较为敏感，倾向于选择更具性价比的房源。相反，高收入群体对价格波动表现出相对的不敏感性，但对住房质量有更高的需求。当房屋面积减半时，选房频率下降了 9.1%；而当房屋中私密卫生间被移除时，高收入群体中没有人选择住房。他们主要受到房屋装修水平的影响，导致他们自发地拒绝那些价格低廉但质量较差的房源。主要是因为他们更倾向于选择质量更好、装修更精美的居住空间。

这一发现表明，如果政府旨在支持中低收入群体，决策者应考虑重新分配建筑成本和住房补贴的比例。通过这些调整降低房价，政府可以更有效地支持并满足低收入群体的需求。

### F.5 效率分析实验

为了模拟在 SRAP-Agent 中 $n$ 个参与者的政策执行过程，我们利用了我们社交网络框架中固有的社区结构。该框架通过在小型社区内部具有密集连接，而在较大群体之间连接稀疏的特点，模仿了现实世界中的社交网络。因此，我们假设社交网络包含 $n$ 个参与者和 $k$ 个强连接组件，其中最大的强连接组件包含 $m$ 个参与者。

#### 理论时间复杂度分析

给定 $N_{s}$ 次通信回合，我们将最大社区结构的大小设置为 $m=10$。如果 LLM 推理过程的时间成本为 $t_{a}$，则理论时间复杂度为 $O(N_{s}\cdot k\cdot m\cdot t_{a})$。此外，我们还考虑了影响信息传播的社交网络中的稀疏连接。在 SRAP-Agent 中，我们从所有参与者收集论坛帖子，允许他们在论坛中进行搜索。搜索依赖于具有操作时间成本 $t_{v}\leq 0.005$ 的向量数据库。假设有 $N_{f}$ 次论坛通信回合，则论坛的理论时间复杂度为 $O(N_{f}\cdot n\cdot(t_{a}+t_{v}))$。

#### 并行加速

因为每个连接组件内部的通信可以并行进行，所以可以通过异步或多进程方法加速 SRAP-Agent 的实际运行时间；我们的框架采用前者。因此，实际执行速度不受 $k$ 的限制，实现了 $k$ 倍加速。

表 14：SRAP-Agent（基于 GPT-3.5）中策略模拟过程的模拟时间、API 调用次数和预算。

|  | 代理数量 $n$ |
| --- | --- |
|  | 10 | 50 | 100 | 200 |
| 时间 / 分钟 | 1.0E+01 | 2.5E+01 | 3.0E+01 | 3.0E+01 |
| API 调用 | 3.0E+02 | 5.0E+02 | 1.0E+03 | 1.5E+03 |
| 预算 | 2$ | 3$ | 5$ | 8$ |

不同的策略设置和资源量会导致不同的策略执行过程。在我们的实验中，我们通常设置 $N_{s}>N_{f}$，这表明小组内的通信更为密集，而整个网络之间的通信较少。通常，$N_{s}=10\cdot N_{f}$。为了测试 $n$ 的增加是否会导致过多的 API 调用，从而产生高成本，我们选择了一个代表性的策略设置（$p^{select}+r^{size}$），同时保持所有实验中 $|R|$ 与 $|P|$ 的比值一致。如表 [14](https://arxiv.org/html/2410.14152v1#A6.T14 "Table 14 ‣ Parallel Acceleration ‣ F.5 Efficiency Analysis Experiments ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent") 所示，可以看到，随着参与者数量的增加，SRAP-Agent 并不会产生二次时间成本。这是因为：首先，我们采用了稀疏的社交网络结构，使得理论上的 API 调用次数与 $k$ 成线性关系，而不是 $n^{2}$。其次，通过使用异步方式并行运行社交网络的强连接组件，我们可以显著降低时间复杂度。

## 附录 G POA

### G.1 POA 的超参数

我们进行诊断实验以验证 POA 算法的有效性。在遗传算法中，我们利用向量化的政策参数作为基因。我们采用高斯变异操作作为变异算子，并使用双点交叉作为交叉算子。方程 [3](https://arxiv.org/html/2410.14152v1#S4.E3 "In 4 POA: Policy Optimization Algorithm ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent") 中的权重设置非常灵活，完全取决于政策制定者的优化目标。为了修改政策优化器的优化目标，我们只需更改用于计算优化指标的权重。当希望更加强调优化社会满意度指标时，建议增加 $SW$ 和平均 $r^{size}$ 等指标的权重。相反，为了优先增强社会公平性指标，建议增加 Rop、方差 $r^{size}$ 和 co-Gini 的权重，如表 [15](https://arxiv.org/html/2410.14152v1#A7.T15 "Table 15 ‣ G.1 Hyper-parameters for POA ‣ Appendix G POA ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent") 所示。

表 15：POA 中不同优化目标的预设指标权重。

|  | 指标 | 优化目标 |
| --- | --- | --- |
|  | 满意度$\uparrow$ | 公平性$\uparrow$ |
| --- | --- | --- |
| 满意度 | 平均 $r^{size}$ | 5 | 1 |
| 平均 $WT$ | 5 | 1 |
| $SW$ | 10 | 5 |
| 公平性 | 方差 $r^{size}$ | 1 | 10 |
| Rop | 5 | 10 |
| co-Gini | 1 | 10 |
| F(V, NV) | 1 | 5 |

### G.2 与其他政策的比较

为了更好地评估 SRAP-Agent 在现实世界稀缺资源分配场景中的政策优化，我们选择了四个基准政策：(1) 三个来自现实世界的常见稀缺资源分配政策，包括新加坡、北京和香港的公共住房租赁政策（$\pi_{S}$、$\pi_{B}$、$\pi_{H}$）；(2) 基于单一指标 $SW$ 的最优匹配政策。

(1) $\pi_{S}$：新加坡的住房分配政策。

此政策使用单队列（$m=1$）分配系统。每个参与者有三次选择机会（$c=3$）。该政策采用 FIFO 排序方法。 ⁴⁴4https://www.hdb.gov.sg/cs/infoweb/residential/renting-a-flat/renting-from-hdb/public-rental-scheme/eligibility

(2) $\pi_{B}$：北京的住房分配政策。

此政策使用多队列（$m=3$）分配系统。每个参与者有三次选择机会（$c=2$），且不考虑等候名单机制。参与者选择其首选类型的住房（$p^{select}$），资源分配基于住房面积（$r^{size}$）。该政策采用 FIFO 排序方法。 ⁵⁵5http://www.bjft.gov.cn/ftq/c100011/zlmlist.shtml

(3) $\pi_{H}$：香港的住房分配政策。

该政策采用单队列系统（$m=1$），但考虑了候补机制，赋予每个参与者两次选择房屋的机会（$k=2,c=2$）。房屋按随机方式分配（$r^{random}$）。该政策考虑了弱势群体的优先权，并采用VFR排序方法。 ⁶⁶6[https://www.housingauthority.gov.hk/en/at-a-glance/index.html](https://www.housingauthority.gov.hk/en/at-a-glance/index.html)

(4) $\pi_{KM}$：基于$SW$度量优化的政策。

我们首先收集所有参与者对每个资源的满意度水平。然后，我们基于单一的$SW$度量计算最佳匹配，使用二分图匹配算法（Kuhn-Munkres算法）。需要注意的是，从这种匹配中得出的结果代表了理论上的上限。因为在排队系统中，参与者不能随意选择资源；每个参与者能够看到的资源是有限的。

### G.3 其他全局搜索元启发式算法

表16：不同POA算法的比较实验。

| POA | 满意度 | 公平性 |
| --- | --- | --- |
|  | 平均$r^{size}\uparrow$ | 平均$WT$ $\downarrow$ | $SW$ $\uparrow$ | 方差$r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- |
| PSO-POA | 0.29 | 8.86 | 12.10 | 4.34 | 1 | 0.98 |
| PSO-POA | 0.70 | 8.82 | 26.20 | 12.05 | 10 | 0.96 |
| PSO-POA | 0.60 | 8.75 | 26.50 | 8.69 | 2 | 0.96 |
| PSO-POA | 1.45 | 8.39 | 62.40 | 26.00 | 40 | 0.93 |
| GA-POA | 15.18 | 2.96 | 403.30 | 273.87 | 396 | 0.50 |

为了说明使用GA算法构建POA的必要性，我们开发了一种新的粒子群优化（PSO）算法来构建POA（政策优化算法），并将其与GA算法的性能进行比较。我们选择社会满意度作为优化目标，并使用40条历史数据作为$\Pi_{h}$。在这些条件下，我们在10次迭代内运行GA-POA和PSO-POA。如表[16](https://arxiv.org/html/2410.14152v1#A7.T16 "Table 16 ‣ G.3 Other Global Search Metaheuristic Algorithms ‣ Appendix G POA ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")所示，由于优化政策参数实际上是离散变量，PSO在控制速度和位置参数时面临挑战，需要确保政策在变化后保持有效值，并朝着更好的方向演化。

这是因为，$\pi$的参数相对固定，如队列数量（3或2）。然而，PSO无法可靠地确保这一点，导致更新的策略参数无效或导致不良结果。相对而言，GA通过选择、交叉和变异操作有效地探索搜索空间，具有强大的全局搜索能力，避免了局部最优。GA能够处理各种类型的优化问题，使其更适合解决离散问题。对于POA，大部分时间成本集中在SRAP-Agent的策略仿真过程，而非执行优化算法的时间成本。

### G.4 优化成本

我们的POA中主要的成本是系统仿真。SRAP-Agent中50个参与者的仿真大约需要20分钟，因此，所有仿真大致会进行40-60次迭代，总计大约需要20小时。这大大减少了与社会经济仿真实验相比的时间成本。考虑到POA的时间消耗问题，我们采用回归器来估计SRAP-Agent的仿真结果，具体采用Ridge回归器（$\lambda=1.0$）来构建预测模型。为了控制回归器的训练样本数量并最小化时间成本，采用逐渐增加训练样本的方法。回归器的训练会持续进行，直到测试数据集上的MAE误差达到阈值（0.05）。

## 附录H LLM基础代理的案例研究

### H.1 记忆组件的消融实验

表17：在SRAP-Agent中模拟$\pi_{e}$执行过程的政策评估指标结果，分别使用了具有和不具有记忆的LLM基础代理。

| 指标 | 有记忆 | 无记忆 |
| --- | --- | --- |
| 平均 $r^{size}$ | 16.21 | 13.89 |
| 平均 $WT$ | 1.88 | 2.94 |
| SW | 425.05 | 431.80 |
| 方差 $r^{size}$ | 202.63 | 209.43 |
| Rop | 193.50 | 342.00 |

| co-Gini | 0.37 | 0.49 | ![参考说明](img/8e75f4bf4371ad48596090cf70d30a30.png)

(a) James 和 Emma 之间的聊天对话。

![参考说明](img/88c7d8dadef798a512781f4a48d86661.png)

(b) James 和 Oliver 之间的聊天对话。

图9：SRAP-Agent中疑虑与信任的心理状态。Oliver对James提供的信息表示怀疑，选择停止沟通；而James和Emma之间的讨论则充满信息性和诚意。

可以观察到，加入记忆机制显著提高了公平性指标。例如，ROP减少了43.5%，而co-Gini减少了0.12，表明当存在记忆机制时，资源分配变得更加公平。在满意度指标方面，平均$WT$显著减少。这表明，通过社交网络中的沟通和信息传播，代理能够更好地理解当前的资源分配状态，从而影响整体分配过程和最终的策略评估结果。

### H.2 新兴行为

在涉及LLM基础的代理人的互动动态中，无论是欺骗性行为还是合作行为，代理人的行为都是在其心理状态发生变化之后的结果。与这些行为模式相对应的心理状态有两种主要类型：怀疑和信任，如图[9](https://arxiv.org/html/2410.14152v1#A8.F9 "图 9 ‣ H.1 关于记忆组件的消融研究 ‣ 附录 H 基于LLM的代理案例研究 ‣ SRAP-Agent：模拟和优化稀缺资源分配政策与基于LLM的代理")所示。

图中描绘了两种不同的情境：一种是詹姆斯与陌生人奥利弗的对话，另一种是詹姆斯与他的朋友艾玛的对话。在与奥利弗的对话中，奥利弗最初隐瞒了所有关于可用房屋的信息，以最大化自己的利益。这一策略使得詹姆斯采取了更为谨慎的态度，将奥利弗视为一个强大的对手。因此，詹姆斯在对话中变得更加小心，认为奥利弗是一个强大的对手。他避免讨论自己的选择，而是强调其他社区的优势。与此截然不同的是，詹姆斯与朋友艾玛的互动则体现了真诚与相互信任的表现。这样的互动凸显了SRAP-Agent在模拟人类心理反应和思维过程方面的熟练程度。

表格 18：*话语生成*的提示模板

<svg class="ltx_picture" height="174.23" id="A8.T18.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,174.23) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="146.67" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">{concise role description} Here is your memory {memory} Your goal is to rent one house. For now, you want to discuss this with some acquaintances. {utterance plan} Your acquaintances include: {acquaintances} Your recent chat records with your acquaintances: {recent chats} {The example of group discuss response} !![IMPORTANT]: the information in EXAMPLE should NOT appear in response !! - Respond in this format: Thought: (You always think about what to do) Acquaintance: (Acquaintance name, could be a list or string) Output: (things you want to tell this Acquaintance in particular, stay consistent with your plan and thought) .. (this Thought/Acquaintance/Output repeat at most {acquaintance number} times!!) Respond in first person:</foreignobject></g></g></svg>

表格 19：*沟通计划生成*的提示模板

<svg class="ltx_picture" height="91.21" id="A8.T19.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,91.21) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="63.65" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">{role description} You want to rent a house. For now, you want to discuss this with some acquaintances. {acquaintance description} Here is your memory {memory} The current situation of the renting system is: {system competitiveness description} Your personality is {personality} {goal} Respond in this format: {respond format} Respond in the second person:</foreignobject></g></g></svg>

表格 20：*决策过程*的提示模板

<svg class="ltx_picture" height="141.02" id="A8.T20.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,141.02) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="113.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Choosing one house needs the following steps: 1\. Choose a community 2\. Choose the type of house 3\. Choose a house This is information you collected from previous conversations with others: {memory} {role description} You’re planning to choose one house. To choose a house that satisfies you, you are going to {task}. {house info} {thought hint} - If you made up your choice, respond in this format: Thought: ({thought type}) Action: Choose Action Input: {choose type}. - If you chose none of them, respond in this format: Thought: ({thought type}) Action: Give up Action Input: I choose none of these options.</foreignobject></g></g></svg>

表格 21：*广播*的提示模板

<svg class="ltx_picture" height="185.65" id="A8.T21.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,185.65) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="158.09" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">{role description} Here’s your plan to publish info online: {plan} Here is your memory: {memory} You can publish house information online if you want to. The Info you posted should contain information about the community, house and The available community index should be one of [{community ids}]. - If you want to publish house information online, respond in this format: Thought: (your view on the information you want to publish) Action: Publish Community: (community index, should be one of [{community ids}]) Info: (The information you want to publish about this community, stay consistent with your plan and thought; Ensure that the information is specific and clear) - If you don’t want to publish anything respond in this format: Thought: (reason why you don’t want to publish anything) Action: Give up</foreignobject></g></g></svg>

表格 22：*关系评估*的提示模板

<svg class="ltx_picture" height="257.25" id="A8.T22.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,257.25) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="229.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Your relationship with your acquaintances may include: Friend: A person with whom one shares a close and mutually supportive bond of affection and trust. Enemy: A person who is actively opposed or hostile to another, often due to conflicts or animosity. Competitor: Someone who engages in rivalry or competition with another, typically in the same field or for the same goal. Mate: A partner in a romantic or sexual relationship, often implying a deep emotional connection. Colleague: A person with whom one works or collaborates, typically within the same organization or profession. Stranger: An individual who is not known or familiar to someone, often encountered for the first time. Your task is to update your relation with {acquaintance name}, based on your previous view of {acquaintance name} and recent communication. {role description} Here is your memory: {memory} {relation} Here’s your recent communication with {acquaintance name}: {communication} - Respond in this format: My Relation with A: friend (friend/enemy/competitor/mate/colleague/stranger/..) A is an honest and trustworthy person, and I think he is worth making friends with. (My view of this person) Respond:</foreignobject></g></g></svg>

表格 23：*记忆反思*的提示模板

<svg class="ltx_picture" height="141.02" id="A8.T23.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,141.02) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="113.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Progressively summarize new lines provided, adding onto the previous summary and returning a new summary. EXAMPLE Current summary: You think a large house is too big for your family. And you didn’t make a choice. New lines: Thought: The middle house can accommodate my family to live in and has high cost-effectiveness. Output: My choice is the middle house. New summary: You think a middle house can accommodate your family members, better than a large house. And you choose a middle house. END OF EXAMPLE Current summary: {summary} New lines of conversation: {new lines} New summary:</foreignobject></g></g></svg>

表格 24：*记忆评估*的提示模板

<svg class="ltx_picture" height="224.04" id="A8.T24.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,224.04) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="196.49" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You’re {name}. You’re planning to choose one house. Your task is to use MEMORY to assess the credibility of the forum information and summarize the useful information in the forum information based on your previous summary. MEMORY: {memory} End of MEMORY Here’s the forum information: {forum info} [!Important!]: Keep in mind that you and your competitors are vying to rent a house. Both you and your competitors can share diverse information on the forum. And you get forum information from this platform. Remember to save the sequence number of the information you believe in in the summary content - Respond in this format: Trusted: (Summary of the useful information, which you assessed as trustworthy in the forum information) Suspicious: (Summary of the suspicious information, which you suspicious as trustworthy in the forum information; If there’s no suspicious information, simply return None) Reason: (why do other competitors say these things? Try to find a reasonable intention for their intention.) Respond:</foreignobject></g></g></svg>
