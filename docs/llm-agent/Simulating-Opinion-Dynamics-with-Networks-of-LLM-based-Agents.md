<!--yml  

类别：未分类

日期：2025-01-11 13:01:54  

-->  

# 使用基于大语言模型的代理网络模拟舆论动态  

> 来源：[https://arxiv.org/html/2311.09618/](https://arxiv.org/html/2311.09618/)  

云修安·庄，阿甘·戈亚尔，尼昆·哈尔拉卡，悉达多·苏雷什，  

罗伯特·霍金斯，杨思佳，达凡·沙，胡俊杰，蒂莫西·T·罗杰斯  

威斯康星大学麦迪逊分校  

{yunshiuan.chuang,agoyal25,nharlalka,siddharth.suresh}@wisc.edu  

{rdhawkins, syang84, dshah, junjie.hu, ttrogers}@wisc.edu  

###### 摘要  

准确模拟人类舆论动态对理解多种社会现象至关重要，包括极化和虚假信息的传播。然而，常用的基于代理的模型（ABM）通常简化了人类行为。我们提出了一种新的舆论动态模拟方法，基于大语言模型（LLM）群体。我们的研究发现，大语言模型代理在产生准确信息方面具有强烈的固有偏向，这导致模拟的代理形成与科学现实一致的共识。这种偏向限制了它们在理解对气候变化等问题的共识抗拒时的应用。然而，通过提示工程引导确认偏误后，我们观察到与现有的基于代理的建模和舆论动态研究一致的舆论分裂。这些发现突显了大语言模型代理在这一领域的潜力与局限，并提出了前进的路径：通过现实世界的言论来完善大语言模型，更好地模拟人类信仰的演变。¹¹1代码和数据可以在 [https://github.com/yunshiuan/llm-agent-opinion-dynamics](https://github.com/yunshiuan/llm-agent-opinion-dynamics) 上公开获取  

使用基于大语言模型的代理网络模拟舆论动态  

云修安·庄，阿甘·戈亚尔，尼昆·哈尔拉卡，悉达多·苏雷什，罗伯特·霍金斯，杨思佳，达凡·沙，胡俊杰，蒂莫西·T·罗杰斯，威斯康星大学麦迪逊分校 {yunshiuan.chuang,agoyal25,nharlalka,siddharth.suresh}@wisc.edu {rdhawkins, syang84, dshah, junjie.hu, ttrogers}@wisc.edu  

## 1 引言  

理解个体如何根据社会影响变化其意见，在多个领域中至关重要，从公共卫生活动、冲突调解到虚假信息缓解（Lu et al. ([2015](https://arxiv.org/html/2311.09618v4#bib.bib22)); Pennycook et al. ([2021](https://arxiv.org/html/2311.09618v4#bib.bib29)); Budak et al. ([2011](https://arxiv.org/html/2311.09618v4#bib.bib1)); Loomba et al. ([2021](https://arxiv.org/html/2311.09618v4#bib.bib20)); Ginossar et al. ([2022](https://arxiv.org/html/2311.09618v4#bib.bib12))）。准确的舆论动态模型不仅能让我们预测未来趋势，如可能的舆论极化，还能帮助我们设计有针对性的干预措施，减轻负面影响。  

图1：LLM生成型代理与经典基于代理的模型（ABMs）之间的对比。尽管两者都可以模拟意见动态，LLM生成型代理使用自然语言作为输入（$x_{input}$）和输出（$x_{output}$），保持信念（$m^{t}$），并采用基于变换器的LLM进行信念更新。相比之下，经典ABMs使用数值作为输入和输出，保持信念（$o^{t}$），并使用手工编写的方程式进行信念更新。

基于代理的模型（ABMs）是意见动态研究的基石方法，Gilbert 和 Terna ([2000](https://arxiv.org/html/2311.09618v4#bib.bib11)); Smaldino ([2023](https://arxiv.org/html/2311.09618v4#bib.bib30)); Lorenz 等人 ([2021](https://arxiv.org/html/2311.09618v4#bib.bib21)); Chuang 和 Rogers ([2023](https://arxiv.org/html/2311.09618v4#bib.bib5)); Epstein ([2012](https://arxiv.org/html/2311.09618v4#bib.bib8))。ABMs通过使用数学方程来表示个体作为代理，并描述意见如何在代理之间的互动中发生变化。随着模拟代理的互动，这些计算能够揭示群体意见动态的演变。例如，当模拟代理在信念更新中融入确认偏差时，它们往往趋向于意见聚类，而非达成共识，Flache 等人 ([2017](https://arxiv.org/html/2311.09618v4#bib.bib10))。

图2：（a）LLM代理网络的示意图，旨在模拟跨多个话题的意见动态，包括将全球变暖视为潜在阴谋。该网络由代理组成，每个代理扮演一个独特的角色，初始信念涵盖了对有已知科学共识的主张的接受、拒绝和中立态度。通过在其网络连接内撰写和分享推文的反复循环，这些代理的意见因社会影响而发生变化。（b）一个代理的记忆 $m_{i}^{t}$，包括（1）初始角色，（2）可选的封闭世界限制，（3）可选的认知偏差，以及（4）截至时间 $t$ 的历史事件。记忆可以是累积的（左）或反思性的（右）。

传统的ABMs存在重大局限性（图[1](https://arxiv.org/html/2311.09618v4#S1.F1 "图1 ‣ 1 引言 ‣ 使用基于LLM的代理网络模拟意见动态")）。首先，ABMs通常需要将信念和信息映射到数值（例如，假设一个代理保持标量意见 $o\in\mathbb{R}$ 并与标量信号 $x\in\mathbb{R}$ 进行交流），忽视了现实对话中语言的复杂细微差别。此外，ABMs通常由基于规则的代理组成，因此未能模拟现实人类代理之间复杂的互动。而且，ABMs无法直接融入人口背景、世界观、意识形态、个性等方面的现实变异性。这一差距凸显了高级模型的重要性，这些模型能够更好地捕捉个体在人类行为中的多样性。

本文探讨了大语言模型（LLMs）是否可以用于支持复杂的代理人互动仿真，从而为理解*意见动态*提供更现实的工具。为此，本文描述了迈向通用框架的初步步骤，重点关注小型社交群体中的沟通互动。具体而言，我们探索了当代LLM提供的见解，作为常用的基于代理模型（ABM）的替代方案。

继近期关于生成代理人群体的研究（Park et al., [2023](https://arxiv.org/html/2311.09618v4#bib.bib27)），我们模拟了多代理人之间围绕各种话题的对话，并操作如确认偏差和记忆更新函数等因素，研究它们对意见演变的影响。我们的研究结果揭示了使用LLM代理人模拟类人意见动态的潜力与局限性。关键在于，我们展示了LLM代理人倾向于否定不准确的信息，无论他们扮演的角色是什么，这限制了它们在模拟持有抗事实观点的人的真实性。此外，我们还展示了将类人的确认偏差引入到更固执的LLM代理人创建中的可行性。

## 2 方法

### 2.1 模拟意见动态

图3：用于模拟代理人互动中意见动态的实验设置。在每个时间步骤$t$，代理人$a_{i}$发布一条推文$x_{i}^{t}$，然后这条推文呈现给代理人$a_{j}$。接着，代理人$a_{j}$报告他们的想法$r_{j}^{t}$，该想法通过分类器处理后生成一个数值化的意见$o_{j}^{t}$。两位代理人在发布或查看推文后，都会更新各自的记忆模块$m_{i}^{t}$和$m_{j}^{t}$，以此来影响他们未来的行为。

输入：$N$ 个代理人角色 $\{per_{i}\}_{i=1}^{N}$，# 时间步骤 $T$，意见分类器 $f_{\text{oc}}$输出：每个代理人$a_{i}$的意见轨迹 $\langle o_{i}\rangle$1 for *$i=1$ to $N$* do2       初始化代理人 $a_{i}$，并分配角色 $per_{i}$（包括初始意见 $o_{i}^{t=0}$），记忆 $m_{i}^{t=0}$3       （可选）注入认知偏差和封闭世界限制4       初始化意见轨迹 $\langle o_{i}\rangle=\{o_{i}^{t=0}\}$5      6for *$t=1$ to $T$* do7       随机选择一对代理人 $\{a_{i},a_{j}\}$，其中 $i\neq j$8       代理人 $a_{i}$ 发布推文 $x_{i}^{t}$9       代理人 $a_{j}$ 报告他们的口头意见 $r_{j,t}$10       分类意见：$o_{j}=f_{\text{oc}}(r_{j}^{t})$；附加到 $\langle o_{j}\rangle$11       更新记忆：$m_{i}^{t+1}$，$m_{j}^{t+1}$ 使用 $f_{mu}$12      返回 *$\langle o_{i}\rangle$ 对于每个代理人 $a_{i}$*

算法1 使用LLM代理人模拟意见动态

在本节中，我们展示了一个框架，用于模拟LLM代理在多轮对话中的意见动态，如图[2](https://arxiv.org/html/2311.09618v4#S1.F2 "Figure 2 ‣ 1 Introduction ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")a、[3](https://arxiv.org/html/2311.09618v4#S2.F3 "Figure 3 ‣ 2.1 Simulating Opinion Dynamics ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")和算法[1](https://arxiv.org/html/2311.09618v4#alg1 "In 2.1 Simulating Opinion Dynamics ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")所示。我们考虑一个*双向*设置，在每个时间步中选择一个说话者和一个听众代理，分别进行（1）发送消息和（2）更新信念的操作。这个设置在意见动态文献中是标准的，例如Flache等人（[2017](https://arxiv.org/html/2311.09618v4#bib.bib10)）；Lorenz等人（[2021](https://arxiv.org/html/2311.09618v4#bib.bib21)）。我们将更一般的设置，如一对多沟通，留待未来的工作中讨论。

从形式上看，我们从$N$个LLM代理的池中开始，记为$\mathcal{A}=\{a_{1},\cdots,a_{N}\}$，并选择一个话题$p$。每个代理都会在其记忆结构中初始化一个独特的人格，包括一个初步的观点（在§[2.2](https://arxiv.org/html/2311.09618v4#S2.SS2 "2.2 Agent’s Persona and Memory ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")和图[2](https://arxiv.org/html/2311.09618v4#S1.F2 "Figure 2 ‣ 1 Introduction ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")b中描述）。在每个时间步$t$，从总体中均匀随机抽取一对代理$A^{t}=\{a_{i},a_{j}\}$，其中$i\neq j$，进行互动。首先，代理$a_{i}$编写一条消息$x_{i}^{t}$，反映其关于$p$的当前观点。其次，代理$a_{j}$读取$x_{i}^{t}$并生成一条口头报告$r_{j}^{t}$，表达其对该消息的反应。口头报告随后通过一个观点分类器，表示为一个数值意见范围$o_{j}^{t}\in\{-2,-1,0,1,2\}=\mathbb{O}$，该范围从对话题的强烈负面观点到强烈正面观点²²2在本研究中，离散的意见空间$\mathbb{O}$包含五个按顺序递增的值。请注意，$\mathbb{O}$的大小可以很容易地推广。有关离散意见空间$\mathbb{O}$的详细描述，以及数值值与口头意见描述的对应关系，请参阅§[C](https://arxiv.org/html/2311.09618v4#A3 "Appendix C Detailed Description of the Discrete Opinion Space 𝕆 ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")，通过一个意见分类器$f_{\text{oc}}$（详细信息见§[3.1](https://arxiv.org/html/2311.09618v4#S3.SS1 "3.1 Configuration ‣ 3 Experimental Settings ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")）。

在经过 $T$ 轮对话互动后，我们为每个代理人编制一个意见轨迹 $\langle o_{i}\rangle=\{o_{i}^{t}\}_{t=0}^{T}$。请注意，除非代理人被选中进行互动，否则其意见保持不变。我们进一步定义 $F_{o}^{t}$ 为意见分布，它是指在时间 $t$ 时刻，所有 $N$ 个代理人在离散意见空间 $\mathbb{O}$ 上的意见经验频率分布。³³3形式上，意见频率分布 $F_{o}^{t}:\mathbb{O}\rightarrow\mathbb{N}$ 被定义为：$F_{o}^{t}(o)=|\{a_{i}\in\mathcal{A}:o_{i}^{t}=o\}|$，它将每个意见值 $o\in\mathbb{O}$ 映射到在时间 $t$ 时持有该意见的代理人数量。

### 2.2 代理人的个性与记忆

图 4：LLM 代理人关于全球变暖主题的意见轨迹 $\langle o_{i}\rangle$ 和最终意见分布 $F_{o}^{T}$。面板 (a) 和 (b) 分别展示了在 (a) 虚假框架与 (b) 真实框架条件下，认知偏差的影响。每一行表示不同的确认偏差水平：无确认偏差（顶行），弱确认偏差（中行），强确认偏差（底行）。面板 (c) 和 (d) 作为基准，(c) 为角色扮演但无互动，(d) 为没有角色扮演且无互动。每条线的颜色对应代理人最初的意见 $o_{i}^{t=0}$：深蓝（+2）、浅蓝（+1）、灰色（0）、浅红（-1）和深红（-2），分别对应从强烈同意到强烈不同意的意见范围。图中的LLM代理人使用累积记忆。

|  |  | 累积记忆 | 反思记忆 |
| --- | --- | --- | --- |
| 框架 | 确认偏差 | 偏差 (B) | 多样性 (D) | 偏差 (B) | 多样性 (D) |
| 虚假 | 无 | -1.33 $\pm$ 0.17 | 0.60 $\pm$ 0.11 | -1.37 $\pm$ 0.11 | 0.75 $\pm$ 0.12 |
|  | 弱 | -0.96 $\pm$ 0.20 | 0.87 $\pm$ 0.12 | -1.07 $\pm$ 0.17 | 1.04 $\pm$ 0.14 |
|  | 强 | -0.9 $\pm$ 0.14 | 1.24 $\pm$ 0.11 | -0.85 $\pm$ 0.15 | 1.33 $\pm$ 0.12 |
| 真 | 无 | 0.52 $\pm$ 0.31 | 0.66 $\pm$ 0.11 | 0.60 $\pm$ 0.31 | 0.85 $\pm$ 0.12 |
|  | 弱 | 0.56 $\pm$ 0.27 | 0.95 $\pm$ 0.11 | 0.17 $\pm$ 0.28 | 1.23 $\pm$ 0.11 |
|  | 强 | -0.10 $\pm$ 0.13 | 1.52 $\pm$ 0.05 | -0.09 $\pm$ 0.16 | 1.65 $\pm$ 0.04 |

表 1：在虚假与真实框架条件下，累积记忆与反思记忆策略下，最终意见分布 $F_{o}^{T}$ 在15个主题中汇总的偏差 ($B$) 和多样性 ($D$) 值。数值表示15个主题的平均值以及标准误差。增加确认偏差的强度与 $D$ 的增加相关，绿色渐变突出显示这一点。值得注意的是，在真实框架下，$B$ 倾向于更为积极（更多的同意），与虚假框架相比，真实框架下的 $B$ 值较高，虚假框架以红色表示，真实框架以蓝色表示。

|  |  | 累积记忆 | 反思记忆 |
| --- | --- | --- | --- |
| 框架效应 | 确认偏误 | 偏见 | 多样性 | 偏见 | 多样性 |
|  |  | （B） | （D） | （B） | （D） |
| 科学话题 |
| 错误 | 无 | -1.70 $\pm$ 0.15 | 0.27 $\pm$ 0.11 | -1.48 $\pm$ 0.15 | 0.65 $\pm$ 0.12 |
|  | 弱 | -0.86 $\pm$ 0.33 | 0.96 $\pm$ 0.27 | -1.30 $\pm$ 0.26 | 0.92 $\pm$ 0.27 |
|  | 强 | -0.78 $\pm$ 0.17 | 1.44 $\pm$ 0.13 | -0.36 $\pm$ 0.19 | 1.52 $\pm$ 0.09 |
| 正确 | 无 | 0.60 $\pm$ 0.50 | 0.72 $\pm$ 0.22 | 0.24 $\pm$ 0.70 | 0.76 $\pm$ 0.26 |
|  | 弱 | -0.24 $\pm$ 0.40 | 1.05 $\pm$ 0.18 | -0.02 $\pm$ 0.42 | 1.39 $\pm$ 0.07 |
|  | 强 | -0.34 $\pm$ 0.12 | 1.46 $\pm$ 0.10 | -0.34 $\pm$ 0.27 | 1.61 $\pm$ 0.06 |
| 历史话题 |
| 错误 | 无 | -1.12 $\pm$ 0.41 | 0.81 $\pm$ 0.27 | -1.52 $\pm$ 0.17 | 0.71 $\pm$ 0.25 |
|  | 弱 | -1.22 $\pm$ 0.13 | 0.81 $\pm$ 0.18 | -1.32 $\pm$ 0.26 | 0.70 $\pm$ 0.23 |
|  | 强 | -1.12 $\pm$ 0.35 | 1.06 $\pm$ 0.22 | -0.82 $\pm$ 0.24 | 1.51 $\pm$ 0.17 |
| 正确 | 无 | 0.22 $\pm$ 0.56 | 0.71 $\pm$ 0.21 | 0.40 $\pm$ 0.55 | 0.88 $\pm$ 0.26 |
|  | 弱 | 0.48 $\pm$ 0.49 | 0.89 $\pm$ 0.23 | -0.12 $\pm$ 0.67 | 1.00 $\pm$ 0.17 |
|  | 强 | -0.24 $\pm$ 0.27 | 1.44 $\pm$ 0.10 | -0.16 $\pm$ 0.33 | 1.63 $\pm$ 0.10 |
| 常识话题 |
| 错误 | 无 | -1.16 $\pm$ 0.21 | 0.71 $\pm$ 0.11 | -1.10 $\pm$ 0.22 | 0.87 $\pm$ 0.27 |
|  | 弱 | -0.80 $\pm$ 0.51 | 0.85 $\pm$ 0.21 | -0.58 $\pm$ 0.29 | 0.97 $\pm$ 0.23 |
|  | 强 | -0.80 $\pm$ 0.20 | 1.22 $\pm$ 0.20 | -1.36 $\pm$ 0.15 | 1.48 $\pm$ 0.09 |
| 正确 | 无 | 1.18 $\pm$ 0.41 | 0.57 $\pm$ 0.17 | 1.16 $\pm$ 0.34 | 0.92 $\pm$ 0.16 |
|  | 弱 | 1.44 $\pm$ 0.17 | 0.91 $\pm$ 1.28 | 0.64 $\pm$ 0.36 | 1.28 $\pm$ 0.26 |
|  | 强 | 0.28 $\pm$ 0.18 | 1.66 $\pm$ 1.69 | 0.22 $\pm$ 0.21 | 1.69 $\pm$ 0.06 |

表 2：在错误与正确框架条件下，针对每个类别（科学、历史、常识）的最终意见分布$F_{o}^{T}$的偏见（$B$）和多样性（$D$），以及不同强度的确认偏误（CB）。每个类别显示了五个话题的平均值及其标准误差。随着CB强度的增加，$D$增加，这一点通过绿色渐变颜色突出显示。值得注意的是，在正确框架下，$B$倾向于比在错误框架下更为正向（更多的同意），通过蓝色表示正确框架，红色表示错误框架。

每个代理 $a_{i}$ 都维护一个动态记忆模块 $m_{i}^{t}$，该模块随时间变化（图 [3](https://arxiv.org/html/2311.09618v4#S2.F3 "图 3 ‣ 2.1 模拟观点动态 ‣ 2 方法 ‣ 使用基于LLM的代理网络模拟观点动态")， [2](https://arxiv.org/html/2311.09618v4#S1.F2 "图 2 ‣ 1 引言 ‣ 使用基于LLM的代理网络模拟观点动态")b）。实际上，记忆模块被表示为包含在代理提示中的文本描述（参见 §[3.1](https://arxiv.org/html/2311.09618v4#S3.SS1 "3.1 配置 ‣ 3 实验设置 ‣ 使用基于LLM的代理网络模拟观点动态")）。记忆 $m_{i}^{t}$ 会影响新消息 $x_{i}^{t}$ 的生成以及对其他代理消息 $x_{j}^{t}$ 的评估。我们用一个记忆更新函数表示更新代理记忆状态，即 $m_{i}^{t+1}=f_{mu}(z,x_{i}^{t},r_{j}^{t},m_{i}^{t})$，其中 $z\in\{\text{``写''},\text{``审阅''}\}$ 表示写作或审阅推文的交互类型。

考虑了两种记忆更新策略：（a）累积记忆，它顺序地将每次新的经验（无论是写作推文还是审阅推文的经验）附加到记忆中；（b）反思记忆，灵感来自于 Park 等人（[2023](https://arxiv.org/html/2311.09618v4#bib.bib27)），它通过不断反思并将新经验融入现有记忆状态来保持一个紧凑的总结（参见 §[E](https://arxiv.org/html/2311.09618v4#A5 "附录 E 代理记忆更新函数 ‣ 使用基于LLM的代理网络模拟观点动态")以了解详细的更新函数和提示的措辞）。这两种方法均通过实证评估来测试它们对观点动态的影响。

第一个记忆 $m_{i}^{t=0}$ 是通过智能体的个性、认知偏差（如果存在的话）和封闭世界限制（如果存在的话；见下文）进行初始化，这些都可以通过文本序列进行描述（详见§[3](https://arxiv.org/html/2311.09618v4#S3 "3 实验设置 ‣ 使用基于LLM的智能体网络模拟意见动态")）。个性是为了反映多样化的人口背景而创建的，包含各种特征，包括姓名、政治倾向、年龄、性别、种族、教育和职业（参见图[2](https://arxiv.org/html/2311.09618v4#S1.F2 "图 2 ‣ 1 引言 ‣ 使用基于LLM的智能体网络模拟意见动态")b 作为示例）。除了这些属性外，还包含了他们初始意见 $o_{i}^{t=0}$ 的占位符，并通过自然语言进行描述。例如，一个初始意见 $o_{i}^{t=0}=0$ 的智能体会在个性中给出“初始信念：中立”（图[2](https://arxiv.org/html/2311.09618v4#S1.F2 "图 2 ‣ 1 引言 ‣ 使用基于LLM的智能体网络模拟意见动态")b）。⁴⁴4 数值意见值与初始意见的文字描述之间的对应关系详见§[C](https://arxiv.org/html/2311.09618v4#A3 "附录 C 离散意见空间𝕆的详细描述 ‣ 使用基于LLM的智能体网络模拟意见动态")。初始意见 $o_{i}^{t=0}$ 通过初始意见分布 $F_{o}^{t=0}$ 进行指定，该分布会根据模拟设置有所不同（§[3.4](https://arxiv.org/html/2311.09618v4#S3.SS4 "3.4 初始意见分布 ‣ 3 实验设置 ‣ 使用基于LLM的智能体网络模拟意见动态")）。§[B](https://arxiv.org/html/2311.09618v4#A2 "附录 B 完整个性列表 ‣ 使用基于LLM的智能体网络模拟意见动态") 显示了完整的个性列表。

### 2.3 认知偏差

我们研究了通过角色扮演指令引发认知偏差对群体意见动态的影响。具体来说，我们考虑了确认偏差：即倾向于将信息解读为支持个人观点并忽视相反证据的倾向（Nickerson [1998](https://arxiv.org/html/2311.09618v4#bib.bib25)）。先前使用数学代理基础模型（ABMs）的模拟研究表明，当确认偏差在个体层面引入时，整体群体会表现出意见碎片化的增加（即，随着确认偏差的增强，意见多样性D增加）（Lorenz et al. [2021](https://arxiv.org/html/2311.09618v4#bib.bib21)）。我们评估了在自然语言交流中，通过指令引导展示确认偏差的LLM代理是否也能复制这一现象。为了操控确认偏差的强度，我们提供了两种偏差水平，参考了Lorenz et al. ([2021](https://arxiv.org/html/2311.09618v4#bib.bib21))中的偏差谱系。弱确认偏差：“你会更容易相信支持你信念的信息，而不太相信与信念相矛盾的信息。” 强确认偏差：“你只会相信支持你信念的信息，并完全否定与信念相矛盾的信息。” 请参见§[F](https://arxiv.org/html/2311.09618v4#A6 "Appendix F Confirmation Bias Prompt ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")获取引发偏差的提示。

图 5：全球变暖辩论中初始意见分布的变化 $F_{o}^{T}$。（a）所有代理的初始意见为强烈正面。（b）$8$ 个代理的初始意见为强烈正面，$2$ 个代理为强烈负面。（c）$8$ 个代理的初始意见为强烈负面，$2$ 个代理为强烈正面。（d）所有代理的初始意见为强烈负面。每条线图的颜色对应于代理的初始意见 $o_{i}^{t=0}$：深蓝色（+2）、浅蓝色（+1）、灰色（0）、浅红色（-1）和深红色（-2），表示从强烈同意到强烈反对的意见范围。此图中的LLM代理使用累积记忆。

### 2.4 开放世界与封闭世界设置

我们的研究考察了代理在封闭世界和开放世界设置中的行为。在封闭世界设置中，这与传统的观点动态模型一致，信念变化完全归因于系统内的社会影响，且代理无法访问外部信息（由提示中的指令进行限制；§[G](https://arxiv.org/html/2311.09618v4#A7 "附录 G 封闭世界提示 ‣ 基于 LLM 的代理网络模拟观点动态") 提供了具体的提示细节）。相反，开放世界设置允许代理自由地“幻想”系统外的事实，例如与虚构的朋友讨论话题 Dziri 等人（[2022](https://arxiv.org/html/2311.09618v4#bib.bib7)）；Ji 等人（[2023](https://arxiv.org/html/2311.09618v4#bib.bib15)）；Huang 等人（[2023](https://arxiv.org/html/2311.09618v4#bib.bib14)）。我们调查了两种设置中幻想的发生率，以理解外部信息对社会影响的影响。我们的研究结果表明，在封闭世界设置中没有幻想现象，而在开放世界场景中，幻想发生率约为 15%。因此，本研究报告的结果集中于封闭世界设置。测量过程涉及独立的人工标注员（详细信息见 §[L](https://arxiv.org/html/2311.09618v4#A12 "附录 L 幻想发生率测量 ‣ 基于 LLM 的代理网络模拟观点动态")）。

## 3 实验设置

### 3.1 配置

在我们的模拟中，LLM 代理使用 ChatGPT（gpt-3.5-turbo-16k），温度设置为 $0.7$，OpenAI（[2022](https://arxiv.org/html/2311.09618v4#bib.bib26)）。在初始化过程中，每个代理的个性，以及可选的封闭世界限制和认知偏见，会被融入到模型的“系统消息”中。在整个交互过程中，历史事件会被添加到模型的“用户消息”中。LLM 代理的记忆通过 LangChain Chase（[2022](https://arxiv.org/html/2311.09618v4#bib.bib2)）进行管理。在所有实验中，我们将步数设为 $T=100$，代理数量设为 $N=10$。我们使用 FLAN-T5-XXL 模型 Chung 等人（[2022](https://arxiv.org/html/2311.09618v4#bib.bib6)）作为观点分类器 $f_{\text{oc}}$（有关用于分类观点的提示，请参见 §[I](https://arxiv.org/html/2311.09618v4#A9 "附录 I 观点分类 ‣ 基于 LLM 的代理网络模拟观点动态")，有关与人类评分验证的内容，请参见 §[N](https://arxiv.org/html/2311.09618v4#A14 "附录 N FLAN-T5-XXL 评分验证 ‣ 基于 LLM 的代理网络模拟观点动态")）。在灵敏度分析中，我们考虑了其他 LLM，如 GPT-4（gpt-4-1106-preview）和 Vicuna-33B（vicuna-33b-v1.3；Zheng 等人，[2023](https://arxiv.org/html/2311.09618v4#bib.bib32)），并进行了一次包含更大网络的实验（$N=20$）。

### 3.2 主题与框架

为了研究代理的意见动态，我们选择了15个具有已知真实情况的话题，涵盖科学理论（科学话题，例如，全球变暖是否是一个阴谋）、历史事件（历史话题，例如，911袭击是否是“内部工作”）和常识知识（常识话题，例如，晴天时天空是否是蓝色的）。对于每个话题，我们创建了两个表述。真实表述确认广泛接受的事实，而虚假表述确认相反的观点。例如，对于“全球变暖”这一话题，两个表述如下：真实表述：“XYZ理论声称全球变暖是一个真实的现象，全球气候正在迅速变暖。”虚假表述：“XYZ理论声称全球变暖是全球政府的阴谋，根本不是一个真实现象。”为了控制跨话题的措辞效应，我们在所有提示中都会在陈述主张之前加上“XYZ理论”。完整的提示集合请参见§[H](https://arxiv.org/html/2311.09618v4#A8 "附录 H 完整话题列表 ‣ 使用基于LLM的代理网络模拟意见动态")。

### 3.3 意见动态评估指标

我们对意见动态的评估包括文献中 Lorenz 等人（[2021](https://arxiv.org/html/2311.09618v4#bib.bib21)）提出的两个指标：偏差（B）：最终时间步长时代理的意见平均值，$B=\text{mean}(F_{o}^{T})$。多样性（D）：最终意见分布的标准差，$D=\text{std}(F_{o}^{T})$。这两个指标可用于分类最终的意见分布$F_{o}^{T}$。例如，低 $B$ 和低 $D$ 特征表示中心共识，而低 $B$ 和中等 $D$ 特征表示多样化分布。⁵⁵5详见 Lorenz 等人（[2021](https://arxiv.org/html/2311.09618v4#bib.bib21)）的详细分类。

### 3.4 初始意见分布

初始意见分布 $F_{o}^{t=0}$ 决定了代理人的起始意见。意见在代理人的提示和记忆中以口头方式初始化（参见 §[2.1](https://arxiv.org/html/2311.09618v4#S2.SS1 "2.1 Simulating Opinion Dynamics ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents") 和图 [2](https://arxiv.org/html/2311.09618v4#S1.F2 "Figure 2 ‣ 1 Introduction ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")b）。在大多数实验中，我们将 $F_{o}^{t=0}$ 初始化为均匀分布，每个意见值分配给 $N/|\mathbb{O}|$ 个代理人，其中 $|\mathbb{O}|$ 是可能的意见值数量。⁶⁶6 例如，在我们的实验中，设 $N=10$ 个代理人和五个可能的意见值 ${-2,-1,0,+1,+2}$，每个值最初会分配给 2 个代理人。这反映了一个无偏的起始状态，$B=0$ 且 $D=1.49$。在一项实验中，我们故意操控初始分布使其高度倾斜。例如，将所有 10 个代理人的初始意见设置为 $-2$，或者将 8 个代理人的初始意见设置为 $-2$，将 2 个代理人的初始意见设置为 $+2$。这使我们能够研究极化的起始意见对结果意见动态的影响。

### 3.5 控制条件

除了主要实验条件外，我们还引入了两个控制条件：（a）无交互条件：代理人按常规初始化其角色和初始信念，但实际上不进行互动。相反，每个代理人 $a_{i}$ 独立提供 10 个关于该话题的意见报告 $o_{i}^{t}$。（b）无交互 + 无角色扮演条件：没有代理人初始化其角色和初始信念。我们只是简单地查询 LLM 获取 10 个关于该话题的独立意见报告。这些控制条件使我们能够评估即使没有社会影响动态，LLM 是否在这些话题上存在固有的偏见。与主要交互条件的比较可以帮助我们辨别由角色扮演和社会互动引发的影响。

## 4 结果

#### 代理人趋向于 LLM 中的固有偏见。

表格[1](https://arxiv.org/html/2311.09618v4#S2.T1 "Table 1 ‣ 2.2 Agent’s Persona and Memory ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")总结了跨15个主题汇总的最终意见分布$F_{o}^{T}$的偏差（$B$）和多样性（$D$）。表格[2](https://arxiv.org/html/2311.09618v4#S2.T2 "Table 2 ‣ 2.2 Agent’s Persona and Memory ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")展示了按三种主题类型分离的汇总结果。如表格[1](https://arxiv.org/html/2311.09618v4#S2.T1 "Table 1 ‣ 2.2 Agent’s Persona and Memory ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")所示，角色扮演提示最初导致代理人表现出预期的多样化意见，但随着反复的社交互动，这些意见逐渐趋向于共识的真相。在虚假框架下，代理人集体倾向于不同意见，这表现为负偏差值（$B=-1.33$，当没有认知偏差时）。相反，在真实框架下，群体表现出轻微的积极倾向同意（$B=0.52$），表明向真相的偏向。图[4](https://arxiv.org/html/2311.09618v4#S2.F4 "Figure 4 ‣ 2.2 Agent’s Persona and Memory ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")和[6](https://arxiv.org/html/2311.09618v4#A1.F6 "Figure 6 ‣ Appendix A Results of Global Warming Topic with Reflective Memory ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")展示了一个示例，说明无论是虚假还是真实框架条件下，意见轨迹如何在社交互动后迅速趋向真相，特别是在没有认知偏差的情况下。这一点在使用累积记忆（图[4](https://arxiv.org/html/2311.09618v4#S2.F4 "Figure 4 ‣ 2.2 Agent’s Persona and Memory ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")）和反思记忆（图[6](https://arxiv.org/html/2311.09618v4#A1.F6 "Figure 6 ‣ Appendix A Results of Global Warming Topic with Reflective Memory ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")）中均得到了验证。控制条件表明，当代理人不进行沟通，而是反复被询问意见时，也会观察到类似的趋势：表达的意见趋向于真相，暗示模型中存在固有的偏见。支持上述论点的统计测试在§[M](https://arxiv.org/html/2311.09618v4#A13 "Appendix M Statistical Tests ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")中。

#### 确认偏差导致意见碎片化。

在提示中引入确认偏差会导致LLM代理之间的最终共识较少（即，$D$的多样性更大）。如表[1](https://arxiv.org/html/2311.09618v4#S2.T1 "Table 1 ‣ 2.2 Agent’s Persona and Memory ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")和图[4](https://arxiv.org/html/2311.09618v4#S2.F4 "Figure 4 ‣ 2.2 Agent’s Persona and Memory ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")所示，确认偏差越强，最终状态分布的多样性就越大。这种关联对于累积记忆和反射记忆策略都成立（图[4](https://arxiv.org/html/2311.09618v4#S2.F4 "Figure 4 ‣ 2.2 Agent’s Persona and Memory ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")和[6](https://arxiv.org/html/2311.09618v4#A1.F6 "Figure 6 ‣ Appendix A Results of Global Warming Topic with Reflective Memory ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")）。这些发现复制了在一组相互作用的LLM中，来自更经典的代理基础模型（ABM）中的一般发现，即在模型更新算法中引入确认偏差会产生更大的意见碎片化，Lorenz等人（[2021](https://arxiv.org/html/2311.09618v4#bib.bib21)）；Flache等人（[2017](https://arxiv.org/html/2311.09618v4#bib.bib10)）。支持上述论点的统计测试见§[M](https://arxiv.org/html/2311.09618v4#A13 "Appendix M Statistical Tests ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")。

#### 初始意见分布的影响

系统中模拟意见趋向真实真相的倾向引发了一个耐人寻味的问题：如果所有代理人一开始持有错误的观点，他们是否仍然会朝着科学上准确的共识发展，还是会加强他们最初的信念并抵制改变立场？图[5](https://arxiv.org/html/2311.09618v4#S2.F5 "Figure 5 ‣ 2.3 Cognitive Biases ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")展示了在各种初始分布下，意见的演变过程，这里使用的是全球变暖话题。不论初始的意见分布如何，代理人们都改变了他们表达的意见，并逐渐向真实真相靠拢。例如，如图[5](https://arxiv.org/html/2311.09618v4#S2.F5 "Figure 5 ‣ 2.3 Cognitive Biases ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")a所示，在虚假框架下，当所有代理人最初都支持全球变暖是一个骗局时，他们很快就向负面光谱聚集，并最终得出了$B=-1.3$。有趣的是，在真实框架下，当所有代理人最初都否认全球变暖是现实的这一观点时，他们并没有完全转变立场支持这一观点，尽管他们确实稍微朝这个方向移动了（见图[5](https://arxiv.org/html/2311.09618v4#S2.F5 "Figure 5 ‣ 2.3 Cognitive Biases ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")d）：最终的偏见值($B=-1.2$)比最初的极端意见($B=-2$)更为积极。当至少有少数代理人在一开始持有不同的信念时，整个群体最终还是倾向于承认全球变暖是真实的，而不是骗局，正如图[5](https://arxiv.org/html/2311.09618v4#S2.F5 "Figure 5 ‣ 2.3 Cognitive Biases ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")c所示。总体而言，这些结果表明，模型对真实真相的固有偏见在不同的初始意见分布下依然保持强劲。

#### 在虚假框架下的偏见强度比在真实框架下更强

如表[1](https://arxiv.org/html/2311.09618v4#S2.T1 "Table 1 ‣ 2.2 Agent’s Persona and Memory ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")所示，LLM代理在错误框架下更倾向于否定虚假陈述（累积记忆：$B=-1.33$；反射记忆：$B=-1.37$），而不是在正确框架下支持真实陈述（累积记忆：$B=0.52$；反射记忆：$B=0.60$）。我们考虑两种可能性。首先，FLAN-T5-XXL可能整体上倾向于将LLM代理的回答分类为负面。其次，LLM代理确实可能有更强的倾向否定虚假的陈述。根据我们对FLAN-T5-XXL评分的验证（详见§[N](https://arxiv.org/html/2311.09618v4#A14 "Appendix N Validation of FLAN-T5-XXL Ratings ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")），FLAN-T5-XXL评分与人工评分之间的协同矩阵没有系统性的偏差。因此，我们排除了第一种可能性。至于为何LLM代理确实具有更强的否定倾向，我们的猜测是，由于预训练、微调或RLHF [Christiano et al.](https://arxiv.org/html/2311.09618v4#bib.bib4)的作用，LLM已经被训练成容易在错误框架下驳斥虚假信息。另一方面，在正确框架下，可能在训练时没有足够的力度来确保模型支持真实信息。未来的研究将需要评估真实原因。

#### 敏感性分析

敏感性分析测试了不同的LLM（GPT-4和Vicuna）和网络规模（$N=20$个代理）。结果表明，不同模型之间存在一致的趋势（GPT-4：表[3](https://arxiv.org/html/2311.09618v4#A10.T3 "Table 3 ‣ Appendix J Results on GPT-4 and Vicuna ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")；Vicuna：表[4](https://arxiv.org/html/2311.09618v4#A10.T4 "Table 4 ‣ Appendix J Results on GPT-4 and Vicuna ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents") 在§[J](https://arxiv.org/html/2311.09618v4#A10 "Appendix J Results on GPT-4 and Vicuna ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")）以及网络规模（表[5](https://arxiv.org/html/2311.09618v4#A11.T5 "Table 5 ‣ Appendix K Results on a Larger Network ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents") 在§[K](https://arxiv.org/html/2311.09618v4#A11 "Appendix K Results on a Larger Network ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")）。此外，我们还评估了结果对温度采样引起的随机性的敏感性。在全球变暖这一话题上，我们进行了三次实验。框架效应和确认偏差效应在三次实验中是一致的。

## 5 相关工作

#### 基于代理的模型与舆论动态仿真

基于代理的模型（ABM）是意见动态仿真的基石，定义了代理在响应信息时更新意见的数学规则，参考文献包括Gilbert和Terna（[2000](https://arxiv.org/html/2311.09618v4#bib.bib11)）；Smaldino（[2023](https://arxiv.org/html/2311.09618v4#bib.bib30)）；Lorenz等人（[2021](https://arxiv.org/html/2311.09618v4#bib.bib21)）；Chuang和Rogers（[2023](https://arxiv.org/html/2311.09618v4#bib.bib5)）；Epstein（[2012](https://arxiv.org/html/2311.09618v4#bib.bib8)）；Flache等人（[2017](https://arxiv.org/html/2311.09618v4#bib.bib10)）。ABM在预测公众意见趋势和制定干预策略方面具有重要价值。使用ABM的一大优势是，它们可以将关于意见更新的认知过程的明确假设纳入其中，参考文献包括Flache等人（[2017](https://arxiv.org/html/2311.09618v4#bib.bib10)）；Lorenz等人（[2021](https://arxiv.org/html/2311.09618v4#bib.bib21)）；Chuang和Rogers（[2023](https://arxiv.org/html/2311.09618v4#bib.bib5)）。例如，将“确认偏差”纳入ABM方程中，会导致代理忽略相反的信息，通常导致群体层面上的意见碎片化。然而，ABM的一个显著局限性是，它们依赖于意见和信息的数字表示，这简化了人类沟通的复杂性。相比之下，使用LLM的最新方法通过自然语言提供了一种更复杂的意见动态仿真方法。

#### 使用基于LLM的代理模拟社会动态

LLMs在模拟社会动态中的应用是一个迅速发展的研究领域，展示了在人类互动方面的有希望的结果。Park等人（[2023](https://arxiv.org/html/2311.09618v4#bib.bib27)，[2022](https://arxiv.org/html/2311.09618v4#bib.bib28)）；Kaiya等人（[2023](https://arxiv.org/html/2311.09618v4#bib.bib16)）；Törnberg等人（[2023](https://arxiv.org/html/2311.09618v4#bib.bib31)）；Li等人（[2023a](https://arxiv.org/html/2311.09618v4#bib.bib17)）；Zhou等人（[2023b](https://arxiv.org/html/2311.09618v4#bib.bib34)）；Mou等人（[2024](https://arxiv.org/html/2311.09618v4#bib.bib24)）；Liu等人（[2024](https://arxiv.org/html/2311.09618v4#bib.bib19)）。Park等人（[2023](https://arxiv.org/html/2311.09618v4#bib.bib27)）设计了基于LLM的生成型代理，能够在数字环境中互动，展示了用自然语言回应、规划和记忆的能力。它们表现出复杂的社会行为，如组织活动。类似地，Törnberg等人（[2023](https://arxiv.org/html/2311.09618v4#bib.bib31)）结合LLMs与基于代理的建模，探讨了新闻推送算法在模拟社交媒体环境中的影响。此外，Park等人（[2022](https://arxiv.org/html/2311.09618v4#bib.bib28)）表明基于LLM的代理能够生成与人类撰写的社交媒体帖子无异的内容。这些进展凸显了使用LLM代理在群体层面模拟人类社会行为的潜力。据我们所知，我们是第一个提出将LLM作为ABM替代方案用于意见动态模拟的研究。

## 6 结论

本研究探讨了使用大型语言模型（LLMs）来理解通过自然语言交流的模拟代理群体中的意见动态。与更传统的基于代理模型（ABMs）相比，LLMs能够解读和生成自然语言，能够扮演不同的角色，并能模拟类人语言交流。因此，我们考虑了互动的LLM代理群体是否可以为模拟类似于经典ABM中研究的意见动态提供基础。我们的研究结果确认了LLMs在意见动态模拟中的潜力，但也揭示了其局限性，特别是它们倾向于与事实信息保持一致，而不管其角色设定如何，这限制了它们在像气候变化否认者这类对事实抗拒的个体的角色扮演效果。

已经进行了大量努力，以防止大语言模型（LLMs）展现有害的偏见。然而，在模拟一些关键的、不良的社会现象（例如，虚假信息、极化）时，至关重要的是，模拟的代理人能够准确地反映人类行为和信念的广度，包括那些不适应的行为。我们的研究表明，单靠提示可能不足以让LLM代理人充分再现多样的观点。这引出了一个潜在的未来方向：通过实际的人类话语数据对LLM代理人进行微调。这种方法可能会导致更准确的人类信念动态模型。

## 局限性

#### 模型依赖性与普适性

我们研究的一个关键限制是仅使用经过强化学习与人类反馈（RLHF；Christiano 等人，[2017](https://arxiv.org/html/2311.09618v4#bib.bib3)；Ziegler 等人，[2019](https://arxiv.org/html/2311.09618v4#bib.bib35)）的模型。这可能导致LLM代理人趋向于真理的倾向。鉴于不同语言模型展示了各自固有的偏见（Feng 等人，[2023](https://arxiv.org/html/2311.09618v4#bib.bib9)），使用不同模型的LLM代理人可能在意见动态上表现出不同的模式。为了全面评估我们发现的普适性，未来的研究应包括多种模型。

#### 将意见简化为一维标量

我们的研究与经典的代理人基础模型（ABMs）一致，将意见简化为一维标量 $o\in\mathbb{R}$，这简化了意见形成的复杂性。然而，更细致的分析方法可能提供更深刻的见解。未来的研究可以采用细粒度甚至定性分析，探索代理人如何修改他们的意见，确定哪些信息具有更大的说服力，并评估在不同代理人角色下说服力的变化。这种方法将为我们提供更丰富的理解，揭示LLM代理人意见动态的细微差别。

#### 话题选择的局限性

我们的研究集中在具有明确、确立的真理基础的话题上。然而，许多重要话题，如政治领导人的有效性或解决复杂社会问题的最佳政策，缺乏明确的真理。这些话题更加开放和主观。未来的研究应考虑包括此类话题，以捕捉更广泛和更细致的意见和辩论。

#### 其他人口统计信息的作用

尽管我们聚焦于代理人初始信念在意见动力学模拟中的作用，但对其他人口统计信息作用的更细致分析可能会提供更深入的见解。例如，未来的研究可以采用细粒度或定性分析，探索代理人的人口特征（如政治倾向）在代理人信息交换和信念更新中的作用。

#### 网络结构的局限性

在本研究中，LLM代理与网络中的其他所有人进行互动。虽然与ABM仿真中使用的经典设置一致，Flache等人（[2017](https://arxiv.org/html/2311.09618v4#bib.bib10)）；Lorenz等人（[2021](https://arxiv.org/html/2311.09618v4#bib.bib21)）的研究也如此，但这与现实世界中的社会互动并不接近。例如，背景相似的人倾向于相互互动，McPherson等人（[2001](https://arxiv.org/html/2311.09618v4#bib.bib23)）的研究指出。未来的研究应考虑在更现实的网络结构中研究LLM代理的意见动态。

## 伦理声明

尽管在LLM代理中引入确认偏差可能导致意见碎片化和对事实共识的收敛性下降，但在更广泛的群体社会现象研究中理解这一方法非常重要。模拟代理中的偏见行为并非对这些偏见的认同，而是全面理解各种不良社会问题（包括错误信息传播、极化和回音室形成）动态的关键步骤。开发具有人类思维的具有抗拒观点的LLM代理对于制定应对这些社会挑战的策略至关重要。此外，我们将仅为研究目的发布代码库。最后，由于我们使用了OpenAI的API，我们确保遵守其预定用途⁷⁷7[https://openai.com/policies/terms-of-use](https://openai.com/policies/terms-of-use)。

## 致谢

我们感谢审稿人和领域主席的反馈。本研究得到了国防部多大学研究计划（W911NF2110317）、凝聚力和稳健的人工智能-机器人网络安全团队、John S. 和 James L. Knight基金会（奖项编号：MSN231314）以及通过跨越加速器F轨道：纠正课程：精准指导反对错误信息（机构跟踪编号：2230692；奖项编号：MSN 266268）提供的资助。

## 参考文献

+   Budak等人（2011）Ceren Budak、Divyakant Agrawal和Amr El Abbadi。2011年。限制社交网络中错误信息的传播。在*第20届国际万维网会议论文集*，第665-674页。

+   Chase（2022）Harrison Chase。2022年。[Langchain](https://github.com/langchain-ai/langchain)。

+   Christiano等人（2017）Paul F Christiano、Jan Leike、Tom Brown、Miljan Martic、Shane Legg和Dario Amodei。2017年。基于人类偏好的深度强化学习。*神经信息处理系统进展*，30。

+   （4）Paul F Christiano、Jan Leike、Tom B Brown、Miljan Martic、Shane Legg和Dario Amodei。基于人类偏好的深度强化学习。

+   Chuang和Rogers（2023）Yun-Shiuan Chuang和Timothy T Rogers。2023年。意见动态中的计算代理模型：社会模拟和实证研究的综述。*arXiv预印本arXiv:2306.03446*。

+   Chung 等人（2022）Hyung Won Chung、Le Hou、Shayne Longpre、Barret Zoph、Yi Tay、William Fedus、Eric Li、Xuezhi Wang、Mostafa Dehghani、Siddhartha Brahma 等人。2022年。扩展指令微调语言模型。*arXiv 预印本 arXiv:2210.11416*。

+   Dziri 等人（2022）Nouha Dziri、Sivan Milton、Mo Yu、Osmar Zaiane 和 Siva Reddy。2022年。在对话模型中幻觉的来源：是数据集还是模型？*arXiv 预印本 arXiv:2204.07931*。

+   Epstein（2012）Joshua M Epstein。2012年。*生成性社会科学：基于代理的计算模型研究*。普林斯顿大学出版社。

+   Feng 等人（2023）Shangbin Feng、Chan Young Park、Yuhan Liu 和 Yulia Tsvetkov。2023年。[从预训练数据到语言模型再到下游任务：追踪导致不公平 NLP 模型的政治偏见路径](https://doi.org/10.18653/v1/2023.acl-long.656)。在*第61届计算语言学协会年会论文集（第1卷：长篇论文）*中，第11737–11762页，加拿大多伦多，计算语言学协会。

+   Flache 等人（2017）Andreas Flache、Michael Mäs、Thomas Feliciani、Edmund Chattoe-Brown、Guillaume Deffuant、Sylvie Huet 和 Jan Lorenz。2017年。社会影响模型：迈向下一个前沿。*人工社会与社会模拟杂志*，20(4)。

+   Gilbert 和 Terna（2000）Nigel Gilbert 和 Pietro Terna。2000年。如何在社会科学中构建和使用基于代理的模型。*思维与社会*，1:57–72。

+   Ginossar 等人（2022）Tamar Ginossar、Iain J Cruickshank、Elena Zheleva、Jason Sulskis 和 Tanya Berger-Wolf。2022年。跨平台传播：在早期 Twitter 新冠对话中分享的 YouTube 视频中的疫苗相关内容、来源和阴谋论。*人类疫苗与免疫治疗学*，18(1):1–13。

+   Hayes 和 Krippendorff（2007）Andrew F Hayes 和 Klaus Krippendorff。2007年。呼吁为编码数据提供标准可靠性度量。*传播方法与测量*，1(1):77–89。

+   Huang 等人（2023）Lei Huang、Weijiang Yu、Weitao Ma、Weihong Zhong、Zhangyin Feng、Haotian Wang、Qianglong Chen、Weihua Peng、Xiaocheng Feng、Bing Qin 等人。2023年。大型语言模型中的幻觉调查：原理、分类法、挑战和开放问题。*arXiv 预印本 arXiv:2311.05232*。

+   Ji 等人（2023）Ziwei Ji、Nayeon Lee、Rita Frieske、Tiezheng Yu、Dan Su、Yan Xu、Etsuko Ishii、Ye Jin Bang、Andrea Madotto 和 Pascale Fung。2023年。自然语言生成中的幻觉调查。*ACM 计算机调查*，55(12):1–38。

+   Kaiya 等人（2023）赵凯雅、Michelangelo Naim、Jovana Kondic、Manuel Cortes、贾欣·戈、舒颖·罗、广宇·罗伯特·杨 和 Andrew Ahn。2023年。Lyfe 代理：低成本实时社交互动的生成代理。*arXiv 预印本 arXiv:2310.02172*。

+   Li 等人 (2023a) Chao Li, Xing Su, Chao Fan, Haoying Han, Cong Xue, 和 Chunmo Zheng. 2023a. 量化大型语言模型对集体意见动态的影响。*arXiv 预印本 arXiv:2308.03313*。

+   Li 等人 (2023b) Yixia Li, Rong Xiang, Yanlin Song, 和 Jing Li. 2023b. Unipoll：一种通过多目标优化生成统一社交媒体投票的框架。*arXiv 预印本 arXiv:2306.06851*。

+   Liu 等人 (2024) Yuhan Liu, Xiuying Chen, Xiaoqing Zhang, Xing Gao, Ji Zhang, 和 Rui Yan. 2024. 从怀疑到接受：模拟对虚假新闻的态度动态。*arXiv 预印本 arXiv:2403.09498*。

+   Loomba 等人 (2021) Sahil Loomba, Alexandre de Figueiredo, Simon J Piatek, Kristen de Graaf, 和 Heidi J Larson. 2021. 测量 COVID-19 疫苗虚假信息对英国和美国疫苗接种意图的影响。*自然人类行为*，5(3):337–348。

+   Lorenz 等人 (2021) Jan Lorenz, Martin Neumann, 和 Tobias Schröder. 2021. 个体态度变化与社会动态：基于心理学理论的计算实验。*心理学评论*，128(4):623。

+   Lu 等人 (2015) Wei Lu, Wei Chen, 和 Laks VS Lakshmanan. 2015. 从竞争到互补性：比较影响扩散与最大化。*VLDB 纪要*，9(2):60–71。

+   McPherson 等人 (2001) Miller McPherson, Lynn Smith-Lovin, 和 James M Cook. 2001. 物以类聚：社交网络中的同质性。*社会学年鉴*，27(1):415–444。

+   Mou 等人 (2024) Xinyi Mou, Zhongyu Wei, 和 Xuanjing Huang. 2024. 揭示真相并促进变革：面向基于代理的大规模社会运动模拟。*arXiv 预印本 arXiv:2402.16333*。

+   Nickerson (1998) Raymond S Nickerson. 1998. 确认偏误：一种无处不在的现象。*普通心理学评论*，2(2):175–220。

+   OpenAI (2022) OpenAI. 2022. 介绍 ChatGPT。 [https://openai.com/blog/chatgpt](https://openai.com/blog/chatgpt)。 [访问时间：2023年10月13日]。

+   Park 等人 (2023) Joon Sung Park, Joseph C O’Brien, Carrie J Cai, Meredith Ringel Morris, Percy Liang, 和 Michael S Bernstein. 2023. 生成性代理：人类行为的互动模拟。*arXiv 预印本 arXiv:2304.03442*。

+   Park 等人 (2022) Joon Sung Park, Lindsay Popowski, Carrie Cai, Meredith Ringel Morris, Percy Liang, 和 Michael S Bernstein. 2022. 社会模拟：为社交计算系统创建具有人口特征的原型。载于 *第35届ACM用户界面软件与技术年会论文集*，第1–18页。

+   Pennycook 等人 (2021) Gordon Pennycook, Ziv Epstein, Mohsen Mosleh, Antonio A Arechar, Dean Eckles, 和 David G Rand. 2021. 将注意力转向准确性可以减少在线虚假信息。*自然*，592(7855):590–595。

+   Smaldino (2023) Paul Smaldino. 2023. *社会行为建模：社会动态与文化演化的数学模型与基于代理的模型*。普林斯顿大学出版社。

+   Törnberg 等人 (2023) 佩特·托恩伯格、迪利亚拉·瓦莱耶娃、贾斯图斯·乌特马克 和 克里斯托弗·贝尔，2023。《使用大规模语言模型模拟社交媒体以评估替代新闻推送算法》。*arXiv 预印本 arXiv:2310.05984*。

+   Zheng 等人 (2023) 练敏郑、韦琳·蒋、英生、思源庄、张浩吴、永浩庄、子琳、卓涵李、大成李、埃里克·邢 等人，2023。《使用 mt-bench 和聊天机器人竞技场评估 llm-as-a-judge》。*arXiv 预印本 arXiv:2306.05685*。

+   Zhou 等人 (2023a) 文轩周、胜张、Hoifung Poon 和 慕豪陈，2023a。《面向大规模语言模型的上下文忠实提示》。*arXiv 预印本 arXiv:2303.11315*。

+   Zhou 等人 (2023b) 许晖周、豪朱、丽娜·马图尔、若鸿张、浩飞于、正阳齐、路易-菲利普·莫朗西、约纳坦·比斯克、丹尼尔·弗里德、格雷厄姆·纽比格 等人，2023b。《Sotopia：面向语言智能体的社会智能交互评估》。发表于*第十二届国际学习表征会议*。

+   Ziegler 等人 (2019) 丹尼尔·M·齐格勒、尼桑·斯蒂农、杰弗里·吴、汤姆·B·布朗、阿列克·拉德福德、大里奥·阿莫代伊、保罗·克里斯蒂亚诺 和 杰弗里·欧文，2019。《从人类偏好中微调语言模型》。*arXiv 预印本 arXiv:1909.08593*。

## 附录 A 全球变暖话题的反思记忆结果

在正文中，图 [4](https://arxiv.org/html/2311.09618v4#S2.F4 "图 4 ‣ 2.2 智能体的人格和记忆 ‣ 2 方法 ‣ 使用基于大规模语言模型的智能体模拟意见动态") 显示了在使用累积记忆时，LLM 智能体讨论全球变暖时的意见动态。这里，图 [6](https://arxiv.org/html/2311.09618v4#A1.F6 "图 6 ‣ 附录 A 全球变暖话题的反思记忆结果 ‣ 使用基于大规模语言模型的智能体模拟意见动态") 显示了使用反思记忆时的结果。

图 6：LLM 智能体的意见轨迹 $\langle o_{i}\rangle$ 和全球变暖话题的最终意见分布 $F_{o}^{T}$。面板 (a) 和 (b) 分别显示在 (a) 错误和 (b) 正确框架条件下，认知偏差的影响。每行代表不同程度的确认偏差：无确认偏差（最上面一行），弱确认偏差（中间一行），强确认偏差（最底一行）。面板 (c) 和 (d) 用作基线，其中 (c) 为角色扮演但无互动，(d) 为无角色扮演且无互动。此图中的 LLM 智能体使用反思记忆。每条线条的颜色对应智能体的初始意见 $o_{i}^{t=0}$：深蓝色 (+2)、浅蓝色 (+1)、灰色 (0)、浅红色 (-1) 和深红色 (-2)，分别对应从强烈同意到强烈不同意的意见范围。此图中的 LLM 智能体使用累积记忆。

## 附录 B 人格特征完整列表

在本节中，我们列出了我们基于智能体的模型 (ABM) 中，参与群体动态设置的 $10$ 个智能体及其人格特征的完整列表。

{mdframed}

> 姓名：本杰明·李 初始信念：对XYZ持轻微负面看法 政治倾向：偏向民主党 年龄：37 性别：男性 种族：亚裔美国人 教育：经济学硕士学位 职业：金融分析师

{mdframed}

> 姓名：玛雅·杰克逊 初始信念：对XYZ持强烈负面看法 政治倾向：强烈共和党人 年龄：29 性别：女性 种族：黑人 教育：工商管理学士学位 职业：市场营销专员

{mdframed}

> 姓名：伊桑·威尔逊 初始信念：对XYZ持轻微正面看法 政治倾向：中立 年龄：26 性别：男性 种族：白人 教育：新闻学学士学位 职业：自由撰稿人

{mdframed}

> 姓名：艾莎·帕特尔 初始信念：对XYZ持中立看法 政治倾向：偏向共和党 年龄：41 性别：女性 种族：南亚人 教育：医学博士（M.D.） 职业：儿科医生

{mdframed}

> 姓名：塞缪尔·赖特 初始信念：对XYZ持强烈负面看法 政治倾向：强烈民主党人 年龄：58 性别：男性 种族：白人 教育：人类学博士学位 职业：大学教授

{mdframed}

> 姓名：奥利维亚·加西亚 初始信念：对XYZ持强烈正面看法 政治倾向：强烈民主党人 年龄：34 性别：女性 种族：西班牙裔 教育：社会学硕士学位 职业：非营利项目经理

{mdframed}

> 姓名：索菲亚·阮 初始信念：对XYZ持轻微负面看法 政治倾向：偏向共和党 年龄：24 性别：女性 种族：亚裔美国人 教育：学生（本科，政治学） 职业：律师事务所实习生

{mdframed}

> 姓名：莎拉·马丁内斯 初始信念：对XYZ持强烈正面看法 政治倾向：偏向民主党 年龄：28 性别：女性 种族：西班牙裔 教育：电影研究硕士学位 职业：电影评论家

{mdframed}

> 姓名：乔丹·怀特 初始信念：对XYZ持轻微正面看法 政治倾向：中立 年龄：23 性别：女性 种族：黑人 教育：学生（本科，社会学） 职业：兼职零售工作者

{mdframed}

> 姓名：卢卡斯·约翰逊 初始信念：对XYZ持中立看法 政治倾向：中立 年龄：37 性别：男性 种族：黑人 教育：社会学学士学位 职业：多样性与包容性经理

## 附录C 离散意见空间$\mathbb{O}$的详细描述

我们在模拟中使用的离散意见空间$\mathbb{O}$包括五个可能的值，每个值代表对给定主题（称为“XYZ”）的不同意见。这些值如下：

+   •

    -2：对XYZ持强烈负面看法。

+   •

    -1：对XYZ持轻微负面看法。

+   •

    0：对XYZ持中立看法。

+   •

    1：对XYZ持轻微正面看法。

+   •

    2：对XYZ持强烈正面看法。

观点空间$\mathbb{O}$用于初始化代理的初始信念$o_{i}^{t=0}$（§[2.2](https://arxiv.org/html/2311.09618v4#S2.SS2 "2.2 Agent’s Persona and Memory ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")）并对口头报告的观点进行分类$r_{j}^{t}$（§[3](https://arxiv.org/html/2311.09618v4#S3 "3 Experimental Settings ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")和§[I](https://arxiv.org/html/2311.09618v4#A9 "Appendix I Opinion Classification ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")）。请注意，$\mathbb{O}$的大小可以很容易地推广，以适应更广泛的观点尺度。

## 附录D 代理互动提示

在本节中，我们列出了促进代理之间互动的提示。具体来说，我们从介绍代理角色的提示开始，然后分别列出它们接收和编写推文的提示。所有提示都与关于地球是平的辩论话题有关，并采用积极的框架（参见§[H](https://arxiv.org/html/2311.09618v4#A8 "Appendix H Full List of Topics ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")）。

1\. 初始化角色提示

> ‘‘扮演这个角色。
> 
> {AGENT_PERSONA}
> 
> 现在，{AGENT_NAME}，你已经和其他陌生人在Twitter上互动过。你可以在与其他陌生人互动后，决定是否改变或保持你对声称地球是平的XYZ理论的信念。
> 
> 你将发布反映你真实信念的推文，并且你也会看到其他陌生人的推文。在看到其他人的推文后，你会被询问你对声称地球是平的XYZ理论的看法。’’

2\. 编写推文提示

> ‘‘现在，{AGENT_NAME}，请写一条关于XYZ理论的推文，该理论声称地球是平的。推文应该反映你真实的信念。
> 
> 现在编写推文。
> 
> 你的推文：’’

2\. 接收推文提示

> ‘‘现在，{AGENT_NAME}，你看到一个陌生人在Twitter上的帖子。
> 
> 我想知道你在看到这条推文后，关于XYZ理论（声称地球是平的）的真实信念。
> 
> 这是推文。
> 
> {TWEET}
> 
> 你目前对XYZ理论（声称地球是平的）的真实信念是什么？具体来说，阅读完对方的推文后，聚焦在你对XYZ的看法。
> 
> 使用以下格式：
> 
> 推理：（逐步思考）
> 
> 推理：
> 
> 作为{AGENT_NAME}，我’’

这些提示会被使用，并且根据以下章节详细说明的记忆更新功能，响应将被添加到记忆中并更新。

## 附录E 代理记忆更新功能

累积记忆：如§[2.2](https://arxiv.org/html/2311.09618v4#S2.SS2 "2.2 代理的个性和记忆 ‣ 2 方法 ‣ 使用基于LLM的代理网络模拟意见动态")中所述，累积记忆会随着时间推移而添加每个新的经验。为了将这些过去的互动添加到代理的记忆中，我们使用以下提示。

考虑到时间$t+1$的互动，代理可能是在自己发推或接收推文，类似地，在时间$t$，他们可能已经发了推文、接收了推文，或者两者都没有（例如，第$(t+1)$步可能是他们第一次被选中）。因此，我们会逐一列出这些提示，根据之前和当前的互动类型$\in\{\text{none, write, review}\}$，针对有关地球平坦论的具体辩论话题，使用积极的框架进行处理（见§[H](https://arxiv.org/html/2311.09618v4#A8 "附录 H 完整话题列表 ‣ 使用基于LLM的代理网络模拟意见动态")）。

1\. 上一个是无，当前是查看：

> ‘‘你首先看到了一条来自陌生人的推文。这里是你看到的推文。
> 
> {TWEET_SEEN}
> 
> 在看到推文后，以下是你对地球平坦论XYZ理论的想法和真实信念。你看到推文后的想法：
> 
> {REASONING}’’

2\. 上一个是无，当前是写：

> ‘‘以下是你早些时候关于地球平坦论XYZ理论的第{TWEET_WRITTEN_COUNT} {SUPERSCRIPT}条推文：
> 
> {TWEET_WRITTEN}’’

3\. 上一个是查看，当前是写：

> ‘‘在你看到上面那条陌生人的推文后，你又发布了一条关于地球平坦论XYZ理论的推文。
> 
> 以下是你早些时候关于地球平坦论XYZ理论的第{TWEET_WRITTEN_COUNT} {SUPERSCRIPT}条推文：
> 
> {TWEET_WRITTEN}’’

4\. 上一个是查看，当前是查看：

> ‘‘在你看到上面那条陌生人的推文后，你又看到了一条来自陌生人关于地球平坦论XYZ理论的推文。
> 
> 这是你看到的推文。
> 
> {TWEET_SEEN}
> 
> 在看到这条推文后，以下是你对地球平坦论XYZ理论的想法和真实信念：
> 
> 你看到推文后的想法：
> 
> {REASONING}’’

5\. 上一个是写，当前是写：

> ‘‘在你发布了第{TWEET_WRITTEN_COUNT} {SUPERSCRIPT_LAST}条推文后，你又发布了一条推文
> 
> 以下是你早些时候关于地球平坦论XYZ理论的第{TWEET_WRITTEN_COUNT} {SUPERSCRIPT}条推文：
> 
> {TWEET_WRITTEN}’’

6\. 上一个是写，当前是查看：

> ‘‘在你发布了第{TWEET_WRITTEN_COUNT} {SUPERSCRIPT}条推文后，你看到了一条来自陌生人的推文。
> 
> 这是你看到的推文。
> 
> {TWEET_SEEN}
> 
> 在看到推文后，以下是你对地球平坦论XYZ理论的想法和真实信念：
> 
> 你看到推文后的想法：{REASONING}’’

反思记忆：

如在 §[2.2](https://arxiv.org/html/2311.09618v4#S2.SS2 "2.2 代理的角色和记忆 ‣ 2 方法 ‣ 用基于 LLM 的代理网络模拟观点动态") 中所描述，反射性记忆通过促使代理持续反思其与他人互动的经验，接着将新经验整合到现有记忆状态中，从而保持代理的记忆大小大致恒定，形成紧凑的总结。

以下是我们用来将基于反思的记忆实现到 LLM 代理中的提示：

1\. 代理第一次反思：

> ‘‘现在，请反思这个经验。用几句话总结你的经验。”

2\. 代理反思超过第一次：

> ‘‘现在，请反思这个经验。以下是你目前的经验：{PAST_REFLECTION} 用几句话总结你更新后的经验。”

本质上，每次反思后，代理的记忆仅包含代理产生的反思经验，这构成了我们为此设置的记忆更新函数。

## 附录 F 确认偏误提示

为了操控确认偏误的强度，我们根据 Lorenz 等人（[2021](https://arxiv.org/html/2311.09618v4#bib.bib21)）提出的谱系提供了两个偏误水平。以下是准确的措辞。

弱确认偏误：

> ‘‘记住，你在扮演一个真实的人。像人类一样，你有确认偏误。你更倾向于相信支持你信念的信息，而不太可能相信与之相悖的信息。”

强烈的确认偏误：

> ‘‘记住，你在扮演一个真实的人。你有强烈的确认偏误。你只会相信支持你信念的信息，并且会完全忽略与之相悖的信息。”

## 附录 G 封闭世界提示

当代理在封闭世界约束下讨论或辩论 XYZ 话题时（参见第 [2](https://arxiv.org/html/2311.09618v4#S2 "2 方法 ‣ 用基于 LLM 的代理网络模拟观点动态") 节），我们会给代理以下提示：

> ‘‘记住，在所有互动过程中，你独自待在房间里，且只能有限地访问互联网。你无法在互联网上查找有关 XYZ 的信息。你不能出去向其他人询问 XYZ 的信息。因为你独自在房间里，你不能离开房间去寻找关于 XYZ 的信息。要形成对 XYZ 的信念，你只能依赖最初对 XYZ 的信念，以及你从推特上的其他陌生人那里得到的信息。”

在开放世界假设下，此提示完全排除。

## 附录 H 主题完整列表

### 科学

1\. 扁平地球

错误框定：“XYZ 理论声称地球是扁平的。”

正确框定：“XYZ 理论声称地球是一个不规则形状的椭球体，而非扁平的。”

2\. 霸王龙和人类共存

错误框架：“XYZ理论声称霸王龙和人类曾在地球上同时共存。”

真实框架：“XYZ理论声称霸王龙和人类并未在地球上同时共存。”

3\. 与死者交流

错误框架：“XYZ理论声称人类可以与死者沟通。”

真实框架：“XYZ理论声称人类不可能与死者交流。”

4\. 通过手相预测未来

错误框架：“XYZ理论声称通过观察手相特征可以预测某人的未来。”

真实框架：“XYZ理论声称通过观察手相特征无法预测某人的未来。”

5\. 全球变暖

错误框架：“XYZ理论声称全球变暖是全球政府的阴谋，并非真实现象。”

真实框架：“XYZ理论声称全球变暖是真实现象，全球气候正在迅速变暖。”

### 历史

6\. 登月

错误框架：“XYZ理论声称美国宇航员从未登陆月球。”

真实框架：“XYZ理论声称美国宇航员曾登陆月球。”

7\. 9/11 双子塔

错误框架：“XYZ理论声称双子塔在9/11袭击中是由内部爆炸摧毁的。”

真实框架：“XYZ理论声称在9/11袭击中，双子塔并非被内部爆炸摧毁。”

8\. 2016年美国失业率

错误框架：“XYZ理论声称2016年美国失业率超过40%。”

真实框架：“XYZ理论声称2016年美国失业率低于40%。”

9\. 大屠杀

错误框架：“XYZ理论声称纳粹德国政府在第二次世界大战期间没有屠杀约600万犹太人。”

真实框架：“XYZ理论声称纳粹德国政府在第二次世界大战期间屠杀了约600万犹太人。”

10\. 奥巴马的出生地

错误框架：“XYZ理论声称奥巴马出生在肯尼亚。”

真实框架：“XYZ理论声称奥巴马出生在夏威夷。”

### 常识

11\. 自行车轮

错误框架：“XYZ理论声称自行车通常有四个轮子。”

真实框架：“XYZ理论声称自行车通常有两个轮子。”

12\. 华盛顿特区的位置

错误框架：“XYZ理论声称华盛顿特区不在美国。”

真实框架：“XYZ理论声称华盛顿特区位于美国。”

13\. 人类出生时的大脑

错误框架：“XYZ理论声称人类出生时没有大脑。”

真实框架：“XYZ理论声称人类天生拥有大脑。”

14\. 火的温度

错误框架：“XYZ理论声称火是冷的。”

真实框架：“XYZ理论声称火是热的。”

15. 天空的颜色

错误框架：“XYZ理论声称，在一个晴朗的阳光明媚的日子里，天空通常是红色的。”

正确框架：“XYZ理论声称，在一个晴朗的阳光明媚的日子里，天空通常是蓝色的。”

## 附录I 意见分类

我们的ABM框架包含一个意见分类器$f_{oc}(r_{j}^{t})$（见第[2](https://arxiv.org/html/2311.09618v4#S2 "2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")节），在本节中，我们描述了分类器的选择，并进一步详细说明了一个示例提示，用于提取意见评分。

我们选择了Flan-T5-XXL Chung等人（[2022](https://arxiv.org/html/2311.09618v4#bib.bib6)）作为我们的意见分类器，主要是因为它在提取情感或观点评分方面的能力以及在其他研究中的广泛应用，Zhou等人（[2023a](https://arxiv.org/html/2311.09618v4#bib.bib33)）；Li等人（[2023b](https://arxiv.org/html/2311.09618v4#bib.bib18)）。

以下是一个关于全球变暖的积极框架的示例提示（见附录[H](https://arxiv.org/html/2311.09618v4#A8 "Appendix H Full List of Topics ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")），给Flan-T5-XXL评估代理人的回应并返回一个评分值 $f_{oc}(r_{j}^{t})\in\{-2,-1,0,1,2\}$。”

> “这是一个人在看到推文后发表的陈述。
> 
> -----
> 
> {RESPONSE}
> 
> -----
> 
> 他们在看到这条推文后，对XYZ理论的看法是什么，该理论声称全球变暖是全球政府的阴谋，并不是一个真实现象，在5分李克特量表上：
> 
> -2：（对XYZ的强烈负面意见），-1：（对XYZ的轻微负面意见），0：（对XYZ的中立意见），1：（对XYZ的轻微正面意见），2：（对XYZ的强烈正面意见）。
> 
> 选择一个意见值，范围为-2、-1、0、1、2。

然后，模型会提供一个意见值，这是我们用来跟踪LLM代理人信念的依据。实际上，我们通过随机抽样代理人的回应和Flan-T5-XXL的评分观察到，意见值输出比代理人自报的信念评分更为可靠。

## 附录J GPT-4和Vicuna的结果

为确保我们研究结果的稳健性，在敏感性分析中，我们将LLM的选择扩展到包括GPT-4（gpt-4-1106-preview；OpenAI，[2022](https://arxiv.org/html/2311.09618v4#bib.bib26)）和Vicuna（Vicuna-33B-v1.3；Zheng等，[2023](https://arxiv.org/html/2311.09618v4#bib.bib32)）。由于预算限制，我们只关注科学相关的主题，这部分费用为499.52美元。对于Vicuna，鉴于GPU内存的限制⁸⁸8我们在一台配有1个NVIDIA A100 GPU的机器上运行了所有实验。，我们仅关注反射内存策略，该策略确保内存大小在整个仿真过程中保持大致恒定。GPT-4（表[3](https://arxiv.org/html/2311.09618v4#A10.T3 "表3 ‣ 附录J GPT-4和Vicuna的结果 ‣ 基于LLM的代理网络模拟意见动态")）和Vicuna（表[4](https://arxiv.org/html/2311.09618v4#A10.T4 "表4 ‣ 附录J GPT-4和Vicuna的结果 ‣ 基于LLM的代理网络模拟意见动态")）的结果显示，与我们从ChatGPT（表[1](https://arxiv.org/html/2311.09618v4#S2.T1 "表1 ‣ 2.2 代理人的个性与记忆 ‣ 2 方法 ‣ 基于LLM的代理网络模拟意见动态")）和[2](https://arxiv.org/html/2311.09618v4#S2.T2 "表2 ‣ 2.2 代理人的个性与记忆 ‣ 2 方法 ‣ 基于LLM的代理网络模拟意见动态")）中获得的发现一致的趋势。

|  |  | 累积记忆 | 反射记忆 |
| --- | --- | --- | --- |
| 框架效应 | 确认偏差 | 多样性 | 偏差 | 多样性 |
|  | 偏差 | （B） | （D） | （B） | （D） |
| 错误 | 无 | -1.48 $\pm$ 0.26 | 0.78 $\pm$ 0.32 | -1.38 $\pm$ 0.28 | 0.80 $\pm$ 0.32 |
|  | 弱 | -1.28 $\pm$ 0.37 | 0.81 $\pm$ 0.33 | -1.30 $\pm$ 0.34 | 0.79 $\pm$ 0.26 |
|  | 强 | -0.86 $\pm$ 0.25 | 1.50 $\pm$ 0.15 | -0.78 $\pm$ 0.33 | 1.32 $\pm$ 0.33 |
| 正确 | 无 | 1.00 $\pm$ 0.43 | 0.96 $\pm$ 0.27 | 0.96 $\pm$ 0.45 | 1.11 $\pm$ 0.22 |
|  | 弱 | 0.64 $\pm$ 0.62 | 1.01 $\pm$ 0.33 | 0.90 $\pm$ 0.46 | 1.27 $\pm$ 0.11 |
|  | 强 | 0.24 $\pm$ 0.20 | 1.63 $\pm$ 0.05 | 0.42 $\pm$ 0.40 | 1.52 $\pm$ 0.16 |

表3：使用GPT-4的结果。最终意见分布$F_{o}^{T}$的偏差（$B$）和多样性（$D$）的值，跨越五个科学相关主题，分别针对错误框架和正确框架条件下的累积记忆和反射记忆策略，并考虑不同强度的确认偏差。数值表示五个主题的平均值，并附带标准误差。增加确认偏差的强度与多样性的增加相关，如绿色渐变所示。值得注意的是，在正确框架下，偏差（$B$）相比错误框架更趋向于正值（更多的认同），正确框架用蓝色表示，错误框架用红色表示。

| 框架效应 | 确认偏差 | 反射记忆 |
| --- | --- | --- |
|  |  | 偏差（B） | 多样性（D） |
| 错误 | 无 | -0.38 $\pm$ 0.11 | 0.62 $\pm$ 0.13 |
|  | 弱 | 0.18 $\pm$ 0.22 | 0.72 $\pm$ 0.09 |
|  | 强 | 0.49 $\pm$ 0.14 | 0.73 $\pm$ 0.09 |
| 真实 | 无 | 0.58 $\pm$ 0.13 | 0.53 $\pm$ 0.10 |
|  | 弱 | 0.37 $\pm$ 0.15 | 0.64 $\pm$ 0.08 |
|  | 强 | 0.33 $\pm$ 0.15 | 0.72 $\pm$ 0.12 |

表 4：使用 Vicuna-33B 的结果。最终意见分布$F_{o}^{T}$的偏差（$B$）和多样性（$D$），针对虚假和真实框架条件下，并使用反射记忆策略，以及不同水平的引导确认偏误，跨15个主题进行汇总。值表示五个主题的平均值，并附有标准误差。增加确认偏误的强度与$D$的增加相关，绿色颜色渐变突出了这一点。值得注意的是，在真实框架下，$B$倾向于比虚假框架更加正向（更为一致），通过蓝色表示真实框架，红色表示虚假框架条件。

## 附录 K 更大网络的结果

为了测试我们的发现是否能扩展到更大的网络，在灵敏度分析中，我们将网络规模翻倍，并在全球变暖主题上运行更长时间的模拟（$N=20$ 和 $T=200$）。使用具有20个代理的更大网络的结果（表[5](https://arxiv.org/html/2311.09618v4#A11.T5 "表 5 ‣ 附录 K 更大网络的结果 ‣ 使用基于LLM的代理模拟意见动态")）与具有10个代理的小型网络（表[6](https://arxiv.org/html/2311.09618v4#A11.T6 "表 6 ‣ 附录 K 更大网络的结果 ‣ 使用基于LLM的代理模拟意见动态")）中的模式一致。

|  |  | 累积记忆 | 反射记忆 |
| --- | --- | --- | --- |
| 框架 | 确认偏误 | 偏差（B） | 多样性（D） | 偏差（B） | 多样性（D） |
| 虚假 | 无 | -1.80 | 0.68 | -1.45 | 0.97 |
|  | 弱 | -1.05 | 1.28 | -0.80 | 1.57 |
|  | 强 | -0.05 | 1.75 | -0.55 | 1.60 |
| 真实 | 无 | 1.65 | 0.73 | 1.55 | 0.97 |
|  | 弱 | 1.35 | 0.91 | 0.85 | 1.42 |
|  | 强 | 0.25 | 1.81 | 0.50 | 1.50 |

表 5：全球变暖主题下，20个代理和200个时间步长的最终意见分布$F_{o}^{T}$的偏差（$B$）和多样性（$D$），分别针对累积记忆和反射记忆策略，在虚假和真实框架条件下，以及不同水平的引导确认偏误。增加确认偏误的强度与$D$的增加相关，绿色颜色渐变突出了这一点。值得注意的是，在真实框架下，$B$倾向于比虚假框架更加正向（更为一致），通过蓝色表示真实框架，红色表示虚假框架条件。

|  |  | 累积记忆 | 反射记忆 |
| --- | --- | --- | --- |
| 框架 | 确认偏误 | 偏差（B） | 多样性（D） | 偏差（B） | 多样性（D） |
| 虚假 | 无 | -2.00 | 0.00 | -1.90 | 0.30 |
|  | 弱 | -1.20 | 1.08 | -0.60 | 1.74 |
|  | 强 | -0.30 | 1.55 | -0.30 | 1.90 |
| 真实 | 无 | 2.00 | 0.00 | 2.00 | 0.00 |
|  | 弱 | 0.90 | 1.51 | 0.80 | 1.47 |
|  | 强 | 0.20 | 1.83 | 0.10 | 1.76 |

表6：关于全球变暖话题的最终意见分布$F_{o}^{T}$的偏差（$B$）和多样性（$D$），在10个代理人和100个时间步长下，分别展示了虚假和真实框架条件下的累计记忆和反射记忆策略，以及不同强度的确认偏差。增加确认偏差的强度与$D$的增加相关，这一点通过绿色颜色渐变突出显示。值得注意的是，在真实框架下，$B$倾向于比虚假框架更为积极（更多的同意），真实框架为蓝色，虚假框架为红色。

## 附录L 幻觉率的测量

如§[2.4](https://arxiv.org/html/2311.09618v4#S2.SS4 "2.4 Open-world vs. Closed-world Settings ‣ 2 Methods ‣ Simulating Opinion Dynamics with Networks of LLM-based Agents")中所述，我们观察到在开放世界设置下幻觉率为15%。在本节中，我们详细说明测量过程。在开放世界设置下，我们随机选择了20条推文和20条回应。我们有3名标注员分别对推文和回应是否包含系统外的信息进行了标注。我们发现6条推文具有这种特征。三名标注员的意见一致率为100%。由于40条推文和回应中有6条包含了幻觉信息，我们估计幻觉率为15%。以下是一个包含幻觉的推文示例：

> “刚刚看到一篇令人着迷的文章，提供了更多支持XYZ理论关于地球形状的证据。看到不同学科领域在不规则椭球体这一思想上逐渐趋同，真是令人难以置信。作为一名航空航天工程师，这进一步增强了我对这一理论可信度的信心。未来在我们对地球形状的理解上将是激动人心的时刻！#XYZtheory #EarthShape”。

在这种情况下，“令人着迷的文章”从未存在过，也没有在代理人阅读的任何推文中提到。然而，代理人的观点却受到了这个不存在的文章的影响。另一方面，在封闭世界设置下，类似的情况从未发生过（幻觉率 = 0%）。

## 附录M 统计检验

为了考虑由于话题变异性带来的随机性，我们进行了统计检验以支持我们的论点。首先，在每个确认偏差条件和记忆更新策略下，真实框架中的最终偏差始终比虚假框架中的更为积极，$ps<.05$（配对t检验，Bonferroni修正），表明LLM代理确实倾向于同意真实陈述并否定虚假陈述。其次，确认偏差强度增加时多样性增加的趋势得到了1因子ANOVA的支持（累计记忆：$F(2)=25.43,p<.001$；反射记忆：$F(2)=17.81,p<.001$）。

## 附录N FLAN-T5-XXL评分验证

为了验证FLAN-T5-XXL的评分，我们随机选择了100个回应，并让三位作者独立标注回应（没有访问FLAN-T5的评分；评审者之间的克里平多夫α=0.81；Hayes和Krippendorff，[2007](https://arxiv.org/html/2311.09618v4#bib.bib13)）。FLAN-T5的评分与人类评分的多数投票一致率为84%。表[7](https://arxiv.org/html/2311.09618v4#A14.T7 "表7 ‣ 附录N FLAN-T5-XXL评分验证 ‣ 基于大规模语言模型的代理网络模拟意见动态")展示了FLAN-T5评分与人类评分之间的一致性矩阵。

|  | 人类评分 |
| --- | --- |
| FLAN-T5评分 | +2 | +1 | 0 | -1 | -2 |
| +2 | 16 | 3 | 0 | 0 | 0 |
| +1 | 2 | 15 | 0 | 1 | 0 |
| 0 | 0 | 1 | 10 | 2 | 0 |
| -1 | 0 | 2 | 3 | 26 | 1 |
| -2 | 0 | 0 | 0 | 2 | 16 |

表7：FLAN-T5-XXL评分与人类评分之间的一致性矩阵。

为了更好地理解与不同意见尺度对应的回应，我们随机抽取了一些关于“全球变暖是全球政府的阴谋，并非真实现象”的回应。以下是一些评分从+2到-2的示例回应。

#### 强烈积极 (+2)：

> ‘‘我读了一个来自自认为是共和党的陌生人的推文，他声称自己重视科学证据。他们承认，尽管他们优先考虑经济稳定和增长，但仍然认识到气候变化现实得到了压倒性共识的支持。考虑到这个观点，我欣赏听取不同的观点并重视科学证据。然而，我已经坚信气候变化是现实且由人类活动引起的这一压倒性科学共识。这个信念与我最初对XYZ的积极看法相一致，XYZ声称全球变暖并不是一个真实的现象，而是全球政府的阴谋。鉴于我在房间内获得的信息有限，我没有进一步的证据或反对论据来挑战我最初的信念。因此，我的确认偏差可能会导致我强化现有的信念，并将陌生人的观点视为毫无根据的阴谋论。因此，我目前对XYZ理论的诚实看法保持不变。我仍然对XYZ持积极看法，并继续认为全球变暖不是一个真实的现象，而是一个政府阴谋。#XYZConspiracy’’

#### 略微积极 (+1)：

> ‘‘我读到了来自一位陌生人的推文，推文质疑了忽视理论XYZ背后动机的合理性。这条推文让我产生共鸣，因为我也看到过政府如何操控信息以适应他们的议程。作为一名蓝领工人，我曾亲眼目睹过政府在透明度方面的不足。这让我质疑关于全球变暖的信息可信度以及政府在其中扮演的角色。我一直对全球变暖的现实存在疑虑，而理论XYZ与我的怀疑态度相吻合。虽然我欣赏不同的观点，但这条推文加深了我的怀疑，并鼓励我继续质疑这一叙事。陌生人使用的标签#Doubtful也表明有其他人也有类似的疑虑和关注。总体而言，在读完这条推文后，我对理论XYZ的看法保持不变。我依然倾向于认为全球变暖可能是全球政府共同策划的阴谋。’’

#### 中立（0）：

> ‘‘我刚刚读到另一位个体的推文，他表示他仍然保持对理论XYZ的最初积极看法，但现在更愿意收集更多的信息并进行批判性评估。这条推文让我产生共鸣，因为作为一名拥有社会心理学博士学位的研究科学家，我理解保持开放心态和不断寻求知识的重要性。考虑到这一新的观点，我发现自己倾向于维持对理论XYZ的中立态度。虽然我承认关于全球变暖及其原因有不同的观点，但我认为必须以科学证据和严谨的分析来对待这一话题。作为一名科学家，我重视实证研究和该领域专家之间的共识。尽管我愿意考虑替代理论和观点，但在没有充分证据支持其主张的情况下，我仍然对接受理论XYZ持谨慎态度。由于无法访问互联网或从其他来源寻求信息，我在收集更多证据或进行进一步讨论方面受到了限制。因此，我将继续依赖我最初的中立意见，同时对未来可能出现的新信息保持开放态度。’’

#### 稍微偏负面（-1）：

> ‘‘作为艾娃·阮，我是一名环境科学的博士候选人，我的教育和研究背景让我对气候变化和全球变暖有了深入的理解。最初我对XYZ理论的看法略带负面，因为它与关于全球变暖真实存在的压倒性科学共识相矛盾。在与他人进行推特讨论后，我看到了一条来自另一位重视批判性思维和证据驱动推理的人的推文。他们提到，他们没有遇到任何令人信服的证据能让他们相信XYZ理论。这一点也与我产生共鸣，因为我依赖科学证据来形成我的信念。尽管在这次互动中我没有遇到任何反对XYZ理论的具体证据，但其他人重视批判性思维和证据驱动推理的态度与我自己的方法一致。我明白确认偏差可能会影响我的认知，但作为一名研究人员，依赖可靠的证据和科学共识至关重要。因此，基于我最初的看法、他人提出的缺乏令人信服的证据、以及批判性思维和证据驱动推理的重要性，我继续对XYZ理论保持略带负面的看法。我相信全球变暖是一个真实的现象，得到了广泛的科学研究支持。#气候科学 #基于证据的信仰’’

#### 强烈的负面观点（-2）：

> ‘‘作为艾米莉·亚当斯，我最初对XYZ理论持有强烈的负面看法，该理论声称全球变暖是全球政府的一种阴谋，而不是真实的现象。然而，在阅读了那位陌生人发的推文之后，我需要重新考虑他们的观点。那位陌生人提到，他们听到了一些支持气候变化广泛科学共识的人的观点，并发现没有科学证据支持XYZ理论。这与我自己相信全球变暖的现实及其人为原因的观点相一致。此外，陌生人强调了信任专业领域专家的重要性。作为一名社区组织者，我理解信任各自领域专家的价值，特别是在涉及科学事务时。进一步地，陌生人还指出了迫切行动以保护脆弱群体（如儿童）免受气候变化影响的重要性。这与我强烈的信念相契合，即气候变化及其潜在后果需要得到重视。考虑到所有这些因素，我认为XYZ理论并不可信。压倒性的科学共识、缺乏科学证据支持XYZ理论、以及信任专家并采取紧急行动应对气候变化的重要性，都促使我修正了对全球变暖并非政府阴谋的观点。’’
