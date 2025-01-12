<!--yml

类别：未分类

日期：2025-01-11 12:15:45

-->

# 融合动力学方程：基于LLM的社交意见预测算法

> 来源：[https://arxiv.org/html/2409.08717/](https://arxiv.org/html/2409.08717/)

第一作者 姚俊池 计算机科学学院

四川师范大学 成都，中国

2022160101012@std.uestc.edu.cn    第二作者 张宏杰 计算机科学学院

四川师范大学 成都，中国

zhanghongjie@sicnu.edu.cn    第三作者 欧杰 信息与软件工程学院

中国电子科技大学 成都，中国

oujieww6@gmail.com    第四作者 左定一 计算机科学学院

四川师范大学 成都，中国

zuody0509@stu.sicnu.edu.cn    第五作者 杨郑 福建省工程技术研究中心

光电传感应用中心

福建师范大学 福州，中国

zyfjnu@163.com    第六作者 董志成 信息学院

科技

西藏大学 拉萨，中国

dongzc666@163.com

###### 摘要

在社交媒体日益成为社会运动和公共舆论形成的重要平台的背景下，准确模拟和预测用户意见的动态，对于理解社会现象、政策制定以及引导舆论具有重要意义。然而，现有的模拟方法在捕捉用户行为的复杂性和动态性方面面临挑战。为了解决这一问题，本文提出了一种创新的社交媒体用户意见动力学模拟方法——FDE-LLM算法，该算法融合了意见动力学和流行病模型。这一方法有效地约束了大语言模型（LLM）的行为和意见演化过程，使其更加符合现实的网络世界。具体来说，FDE-LLM将用户分为意见领袖和追随者。意见领袖基于LLM角色扮演，并受到CA模型的约束，而意见追随者则融入了一个结合CA模型和SIR模型的动态系统。这个创新设计显著提高了模拟的准确性和效率。我们在四个真实的微博数据集上进行了实验，并使用开源模型ChatGLM进行了验证。结果表明，与传统的基于代理建模（ABM）的意见动力学算法以及基于LLM的意见扩散算法相比，我们的FDE-LLM算法表现出更高的准确性和可解释性。

###### 关键词：

FDE-LLM, 基于LLM的代理, ABM, 社交意见预测

## 引言

随着互联网技术的迅速发展，用户意见的动态对社会运动和公众舆论产生了深远的影响。因此，研究用户意见动态的模拟与预测，对于深入理解社会现象、科学制定政策以及有效引导舆论具有重要意义[[1](https://arxiv.org/html/2409.08717v3#bib.bib1), [2](https://arxiv.org/html/2409.08717v3#bib.bib2), [3](https://arxiv.org/html/2409.08717v3#bib.bib3)]。然而，用户意见的形成与变化是一个复杂的时间过程，受到多种因素的影响。如何在模型中准确捕捉这一过程，已成为学术界和工业界关注的焦点。

意见模拟方法主要分为两类：传统的ABM和基于LLM的算法。ABM通常通过模拟个体之间的直接互动和局部规则来模拟意见扩散。元胞自动机（CA）是一种具有局部空间交互和时间因果关系的网格动态系统模型，能够模拟复杂系统的时空演化过程[[4](https://arxiv.org/html/2409.08717v3#bib.bib4)]。有限信任模型，如Deffuant-Weisbuch（DW）模型[[5](https://arxiv.org/html/2409.08717v3#bib.bib5)]和Hegselmann-Krause（HK）模型[[6](https://arxiv.org/html/2409.08717v3#bib.bib6)]，考虑了心理因素。Clifford[[7](https://arxiv.org/html/2409.08717v3#bib.bib7)]和Holley[[8](https://arxiv.org/html/2409.08717v3#bib.bib8)]提出的选民模型描述了公众选择的动态，尽管这些数学模型常常简化了人类社会的复杂性。

另一方面，基于LLM的意见传播算法通过构建特定角色代理并利用其交互来代替人类社会互动，从而进行意见传播与预测。Gao等人开发了S3社交网络仿真系统，利用LLM的类人能力[[12](https://arxiv.org/html/2409.08717v3#bib.bib12)]。斯坦福大学将LLM扩展至存储记忆并动态规划行动[[13](https://arxiv.org/html/2409.08717v3#bib.bib13)]。Chuang等人发现基于LLM的代理存在固有偏见，导致它们趋向于与现实世界的科学共识保持一致[[14](https://arxiv.org/html/2409.08717v3#bib.bib14)]。Liu等人提出了基于LLM的假新闻传播仿真框架（FPS）[[15](https://arxiv.org/html/2409.08717v3#bib.bib15)]。然而，基于LLM的方法效率较低，且在大规模代理场景中无法良好扩展。为了解决这个问题，Mou等人提出了一种混合框架，其中核心用户由LLM驱动，而普通用户则通过演绎式代理模型进行模拟。然而，由于LLM的行为不受限制，这一方法在预测现实事件的舆论方面仍然存在困难[[10](https://arxiv.org/html/2409.08717v3#bib.bib10)]。

我们注意到，单纯依赖传统的意见动力学模型（如 CA、HK 等）无法模拟人类的厌倦或免疫情绪。随着相同事件传播时间的延长，人类可能会变得厌倦，从而导致他们的观点逐渐转向中立。意见动力学旨在模拟群体中意见的最终收敛过程。流行病模型模拟病毒在人群中的传播过程，例如易感-感染-康复（SIR）模型[[16](https://arxiv.org/html/2409.08717v3#bib.bib16)]，而人类的厌倦情绪可以通过 SIR 模型来模拟。在本文中，我们提出了一种名为“融合动力学方程 LLM（FDE-LLM）”的意见动力学模拟方法，通过将意见动力学模型与 SIR 模型相结合。首先，我们设计了一个微博模拟器，其中意见领袖使用 LLM 模拟微博上的用户行为，意见跟随者则使用 ABM 模拟意见传播。然后，我们通过将意见动力学与 SIR 模型相结合，约束意见跟随者的意见变化，并使用 CA 模型约束意见领袖的意见变化。最后，我们使用少量样本提示引导意见领袖根据他们的意见值输出行动，这些意见值由 CA 和 LLM 反射共同决定。我们使用开源模型 ChatGLM 作为 LLM 代理，并在四个真实的微博数据集上进行了实验。结果表明，我们的 FDE-LLM 在动态时间规整（DTW）距离和 Pearson 相关系数指标上均优于基于 ABM 和 LLM 的算法。

我们的主要贡献是：

1\. 我们提出了首个基于 LLM 的意见传播算法，并通过动态方程进行约束，提供了更为真实的宏观级别模拟。

2\. 我们将意见动力学与流行病模型相结合，提供了可预测且可解释的意见演变过程。

3\. 在使用 ChatGLM 对真实微博数据进行的大规模实验中，展示了我们的方法相较于 ABM 和 LLM 算法的准确性。

## II 方法

![参考说明](img/f03edb3533fac1a729f19e290d8a9619.png)

图 1：工作流程

FDE-LLM 将用户分为意见领袖和意见跟随者。意见领袖通过基于 LLM 角色扮演的细胞自动机（CA）模型进行约束，而意见跟随者则由一个结合了 CA 模型和 SIR 模型的动态系统进行管理。

在仿真开始时，代表意见领袖的LLM-Action接收到第一条离线新闻并采取行动。如图[2](https://arxiv.org/html/2409.08717v3#S2.F2 "Figure 2 ‣ II Method ‣ Fusing Dynamics Equation: A Social Opinions Prediction Algorithm with LLM-based Agents")所示，LLM-Action所采取的行动会根据LLM-Attitude评估其态度，并将每一轮的态度反馈给CA模型。然后，这些结果分别返回给意见领袖和意见跟随者。这一轮的态度作为意见跟随者的初始态度。你可以从图[1](https://arxiv.org/html/2409.08717v3#S2.F1 "Figure 1 ‣ II Method ‣ Fusing Dynamics Equation: A Social Opinions Prediction Algorithm with LLM-based Agents")中查看详细的工作流程。

![参考标题](img/f06c7920a592c2fbc0b8628997cb0659.png)

图2：行动与态度。左侧部分表示LLM-Action可以选择的行动类型，而右侧部分则详细说明了执行该行动类型的具体行为及LLM-Attitude对该行为的态度评分。

#### II-1 LLM-Action

该模型通过参考社交人格特征配置文件[[9](https://arxiv.org/html/2409.08717v3#bib.bib9)]和Xinyi Mou等人提出的框架[[10](https://arxiv.org/html/2409.08717v3#bib.bib10)]，设计了系统的核心代理，如图[3](https://arxiv.org/html/2409.08717v3#S2.F3 "Figure 3 ‣ II-1 LLM-Action ‣ II Method ‣ Fusing Dynamics Equation: A Social Opinions Prediction Algorithm with LLM-based Agents")所示。

![参考标题](img/e0da6c0f6956f82e8ea00defc24dbe38.png)

图3：代理配置文件

在代理完成一个行动后，LLM-Attitude会评估对该行动所持有的态度。

#### II-2 LLM-Attitude

该模型还通过设计不同事件的提示语，借助上述框架构建代理，从而实现客观公正的态度评估。所使用的大模型与行动部分的模型相同，确保LLM-Attitude和LLM-Action共享相似的价值观，进一步解释了它们生成态度的一致性。每个事件的真实态度也由相应的代理进行评估。

### II-A 模型

#### II-A1 意见领袖：带有CA的LLM

之前，我们提到过基于大语言模型构建代理来模拟群体行为。然而，如早期研究所示，仅使用大模型来传播意见往往会快速收敛到一个稳定值，这在宏观层面上与现实场景的整体趋势相符，但在实际应用中，它并未有效地模拟现实中逐步传播的逻辑。

因此，我们考虑使用CA模型引入约束[[11](https://arxiv.org/html/2409.08717v3#bib.bib11)]，如下所示：

|  | $O_{i}^{t+1}=r\cdot O_{i}^{t}+w\cdot\sum_{j\in X_{i}}\ T_{ij}^{t}$ |  | (1) |
| --- | --- | --- | --- |
|  | $T_{ij}^{t}=\begin{cases}O_{j},&r=0\\ 0,&r=1\text{ 或者 }\left | O_{j}^{t}-O_{i}^{t}\right | >\epsilon\\ \left(O_{j}^{t}-O_{i}^{t}\right)\cdot\sqrt{r\cdot | O_{j}^{t} | },&r\neq 1,0\% \text{ 且 }\left | O_{j}^{t}-O_{i}^{t}\right | \leq\epsilon\end{cases}$ |  | (2) |

，其中$O_{i}^{t+1}$表示个体$i$在时刻$t+1$的更新后的意见或状态，$O_{i}^{t}$是个体$i$在时刻$t$的意见或状态，$r$是保留因子，表示当前意见保留的程度，$w$是影响系数，决定邻居意见的影响程度，$\sum_{j\in X_{i}}T_{ij}^{t}$表示在时刻$t$，邻居$j$在邻域$X_{i}$内对个体$i$的影响总和。该公式模拟了个体意见随时间的演变，考虑了个人保留和周围意见的影响。

我们设计了以下公式来约束LLM生成的态度，使用CA模型：

|  | $\begin{split}O_{i}^{t+1}=\mathrm{clip}\Bigg{(}&\alpha\cdot\left(r\cdot O_{i}^{% t}+w\cdot\sum_{j\in N_{i}}T_{ij}^{t}\right)\\ &+(1-\alpha)\cdot\text{LLM},-1,1\Bigg{)}\end{split}$ |  | (3) |
| --- | --- | --- | --- |

，使用$clip(-1,1)$函数确保更新后的意见值保持在$[-1,1]$范围内。这可以防止意见值超出合理范围，确保模型结果的稳定性和合理性。此外，引入了一个$\alpha$融合系数，用于确定LLM模型和CA模型的相对影响。

#### II-A2 意见跟随者：CA与SIR结合

CA模型在模拟群体内态度传播方面表现良好。然而，在我们研究真实数据集时，我们发现某些社交新闻表现出短期注意现象，之后态度自然衰退。根据“胖猫”数据集的实际曲线观察，支持一方的态度上升到某个阈值后停止增长，逐渐下降并徘徊在零附近。在真实新闻报道中，这表现为中立情感表达，如“质疑真实性”或“双方可能都有问题”。

为了应对这种普遍现象，我们引入了SIR模型。该模型广泛应用于传染病研究，我们将其“可恢复感染个体”的概念应用于弥补CA模型中无约束态度传播的局限性。这使得意见跟随者在接受到类似于意见领袖态度的内容时能够恢复，有效模拟了社交新闻中态度的自然衰退。

SIR模型如下所示：

|  | $O_{i}^{t+1}=O_{i}^{t+1}\cdot e^{-\lambda\cdot | O_{i}^{t+1} | }\quad\text{if}\quad \% \text{random}<\gamma$ |  | (4) |
| --- | --- | --- | --- | --- | --- |

SIR与CA的融合模型通过以下方程5描述。

|  | $O_{i}^{t+1}=\mathrm{clip}\left(\left(r\cdot O_{i}^{t}+w\cdot\frac{\sum_{j\in N% _{i}}\left(O_{j}^{t}-O_{i}^{t}\right)\cdot\sqrt{r\cdot | O_{j}^{t} | }\cdot I\left( | O_{j}^{t}-O_{i}^{t} | \leq\epsilon\right)}{ | N_{i} | }\right)\cdot e^{-\lambda\cdot | O_{i}^{t+1} | }\cdot I(\text{random}<\gamma),-1,1\right)$ |  | (5) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

这里$|N_{i}|$表示个体$i$周围的邻居数量，$|O_{j}^{t}-O_{i}^{t}|$表示邻居$j$与个体$i$之间的意见差异，$I\left(|O_{j}^{t}-O_{i}^{t}|\leq\epsilon\right)$是一个指示函数，如果意见差异在阈值$\epsilon$内则为1，否则为0，$\lambda$是表示意见自然衰减的衰减率。

## III 实验

### III-A 实验设置

1\. 环境设置：本实验使用GLM4模型，在Python 3.10.12环境中运行，硬件配置为Intel(R) Xeon(R) Platinum 8160 CPU @ 2.10GHz，T4 GPU，CUDA版本为11.8，V11.8.89。

2\. 数据集：我们使用中国社交媒体平台微博上的事件关键词作为爬取标记，每日收集相关事件的帖子和评论。共收集了数万个帖子。我们收集了四个高度争议事件的数据：”胖猫“自杀事件、初中生”江平“在全球数学竞赛中排名第12、”青岛“地铁袭击事件和”电锯女“囤积癌症视频事件。我们设计了一个基于少样本的LLM代理，用于评估所有陈述的态度，态度范围为[-1, 1]。态度模型和行动模型都使用GLM4，以尽可能保持逻辑一致性。

3\. 参数设置：通过实验发现，对于类似的反转新闻，我们可以对所有模型使用统一的参数。参数通过在ABM算法中进行网格搜索获得，以最大化相关系数。具体设置如下：对于CA模型：意见韧性 $r=0.99$，邻域影响系数 $w=0.3$，意见交互阈值 $\epsilon=0.5$；对于SIR模型：感染率 $\beta=0.3$，恢复率 $\gamma=0.9$，衰减率 $\lambda=0.5$。

4\. 模型配置：(1) LLM：仅使用LLM进行仿真，无任何干预。(2) ABM(CA)：在ABM中使用常用的CA模型。(3) ABM(HK)：在ABM中使用常用的HK模型。(4) LLM — ABM(CA)：通过LLM模拟意见领袖，同时ABM使用CA模型模拟群体态度。

### III-B 指标

我们使用两个指标评估模型在仿真中的有效性。

1\. Pearson相关系数（Corr.）用于衡量仿真态度序列$S=\left(s_{1},s_{2},\ldots,s_{n}\right)$与真实态度序列$T=\left(t_{1},t_{2},\ldots,t_{n}\right)$之间的线性相关性。公式如下：

|  | $r=\frac{\sum_{i=1}^{n}\left(s_{i}-\bar{s}\right)\left(t_{i}-\bar{t}\right)}{% \sqrt{\sum_{i=1}^{n}\left(s_{i}-\bar{s}\right)^{2}}\sqrt{\sum_{i=1}^{n}\left(t% _{i}-\bar{t}\right)^{2}}}$ |  | (6) |
| --- | --- | --- | --- |

，其中 $\bar{s}$ 和 $\bar{t}$ 分别是模拟态度和真实态度的均值。相关系数（Corr.）的范围是 [-1, 1]，值越接近 1，模拟态度与真实态度之间的线性相关性越强，表明模型能够更好地捕捉到真实态度的变化。

2\. 动态时间规整（DTW）用于测量模拟态度序列 $S$ 与真实态度序列 $T$ 之间的相似性，尤其是在两序列长度不同或存在时间偏移的情况下。DTW 通过计算最小的累积距离来对齐两个序列。其公式如下：

|  | $DTW(S,T)=\min{\sqrt{\sum_{k=1}^{K}\left(s_{i_{k}}-t_{j_{k}}\right)^{2}}}$ |  | (7) |
| --- | --- | --- | --- |

，其中 $\left(i_{k},j_{k}\right)$ 表示序列 $S$ 和 $T$ 中对齐的点，$K$ 是对齐路径的长度。较小的 DTW 距离表示模拟态度与真实态度的时间序列相似性较高，表明模型在模拟复杂时间序列模式方面表现更好。通过使用 DTW，我们可以更好地评估模型在动态环境中的准确性和鲁棒性。

### III-C 主要结果

![参考标题](img/cd7b9f589595ab296880df1a5aec77ad.png)![参考标题](img/03f21d3ba22aae1510900224a2f5c9a1.png)

(a) 庞茂事件

![参考标题](img/7150fda5d3c2af2f13b738f70511d2ba.png)

(b) 江平事件

![参考标题](img/467740029e1a8b11131984a47a8366ea.png)

(c) 青岛事件

![参考标题](img/d0939b423cea414e3455d554e8bd1480.png)

(d) 电都集事件

图 4： (a) 庞茂事件 (b) 江平事件 (c) 青岛事件 (d) 电都集事件的真实与模拟结果比较。红线表示实际数据。

表 I：真实与模拟结果比较

|  | Pangmao | Jiangping | Qingdao | Dianduji |
| --- | --- | --- | --- | --- |
| 方法 | DTW$\downarrow$ | Corr.$\uparrow$ | DTW$\downarrow$ | Corr.$\uparrow$ | DTW$\downarrow$ | Corr.$\uparrow$ | DTW$\downarrow$ | Corr.$\uparrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| FDE-LLM | 0.3622 | 0.9653 | 0.3664 | 0.8950 | 0.3352 | 0.9605 | 0.3404 | 0.8842 |
| LLM | 2.6584 | 0.7139 | 2.9963 | 0.7208 | 2.7316 | 0.7511 | 2.3170 | 0.7819 |
| ABM(CA) | 0.4335 | 0.8920 | 0.7648 | 0.6748 | 0.2904 | 0.6800 | 0.6806 | 0.8282 |
| ABM(HK) | 1.1838 | 0.6475 | 0.6085 | 0.6961 | 1.1470 | 0.6874 | 0.9453 | 0.6760 |
| LLM+ABM(CA) | 0.7463 | 0.5225 | 1.5748 | 0.6425 | 0.5497 | 0.9174 | 0.8684 | 0.7386 |

本研究聚焦于社交新闻事件，这类事件的特点是关注度迅速上升，且很快失去热度。热度的下降直接导致极端（激进）态度的减少，以及中立和理性分析师的增多。通过对真实态度数据的分析，我们发现社交新闻事件的一个关键特征：当事件失去关注或真相被揭露，且态度稳定后，群体态度趋向于接近 0（群体中立）。

我们的 FDE-LLM 模型结合了 SIR 模型来约束意见追随者的态度，成功模拟了群体态度的丧失特性。与仅关注新闻事件引起的态度变化的 LLM+ABM 模型相比，FDE-LLM 模型在态度变化的基础上丰富了群体态度的自然衰减，使社交新闻事件的模拟更为真实。在单独使用 LLM 时，态度呈现出急剧转变并在极端值上保持稳定。单独使用 ABM 模型时，在某些事件中取得了良好的结果，但缺乏 FDE-LLM 模型的可解释性。详细结果见图 [4](https://arxiv.org/html/2409.08717v3#S3.F4 "Figure 4 ‣ III-C Main Result ‣ III Experiments ‣ Fusing Dynamics Equation: A Social Opinions Prediction Algorithm with LLM-based Agents") 和表 [I](https://arxiv.org/html/2409.08717v3#S3.T1 "TABLE I ‣ III-C Main Result ‣ III Experiments ‣ Fusing Dynamics Equation: A Social Opinions Prediction Algorithm with LLM-based Agents")。

FDE-LLM 模型通过 LLM 意见领袖部分的行为日志，可以直接在微观层面分析群体声明，使其在预测和控制舆情情况方面更具价值。我们已经在[III-E](https://arxiv.org/html/2409.08717v3#S3.SS5 "III-E Toy Example ‣ III Experiments ‣ Fusing Dynamics Equation: A Social Opinions Prediction Algorithm with LLM-based Agents")中讨论了这一问题。

### III-D 消融研究

我们讨论了去除核心用户 CA 限制后，FDE-LLM 模型的表现。

以江平事件为例（图 [5](https://arxiv.org/html/2409.08717v3#S3.F5 "Figure 5 ‣ III-D Ablation Study ‣ III Experiments ‣ Fusing Dynamics Equation: A Social Opinions Prediction Algorithm with LLM-based Agents")），在没有 CA 限制的情况下，模型在预测极端值时倾向于过度反应，某种程度上夸大了事件的紧迫性。在严重情况下，这可能影响数据分析师做出的决策。这突显了通过 CA 限制 LLM 的必要性。其他三个数据集的测试结果如表 LABEL:table_2_ablation_study 所示。

表 II：基于不同指标的不同方法比较

|  | 庞茂 | 江平 | 青岛 | 电都集 |
| --- | --- | --- | --- | --- |
| 方法 | DTW$\downarrow$ | 相关性$\uparrow$ | DTW$\downarrow$ | 相关性$\uparrow$ | DTW$\downarrow$ | 相关性$\uparrow$ | DTW$\downarrow$ | 相关性$\uparrow$ |
| FDE-LLM | 0.3622 | 0.9653 | 0.3664 | 0.8950 | 0.3352 | 0.9605 | 0.3404 | 0.8842 |
| FDE-LLM（无 CA） | 0.5133 | 0.9489 | 0.7530 | 0.8671 | 0.7389 | 0.8952 | 0.5708 | 0.7367 |

![参见图注](img/442d1a6489bcf20041ba5e4519ea1229.png)

图 5: 没有 CA 的 FDE-LLM

### III-E 玩具示例

![参见图注](img/5d0e767e8f645c6623ee542f673914f0.png)

图 6: 反转新闻前 LLM 代理人回应

![参见图注](img/72fdedb8bd2ea687906fdcea07251c61.png)

图 7: 反转新闻

![参见图注](img/08d97d744db437c981fcda46481196a0.png)

图 8: 反转新闻后 LLM 代理人回应

我们随机选择了两组 LLM 代理人，分析了它们在重大新闻事件发布前后的行为（如图所示）。可以观察到，在江平事件中，质疑新闻发布之前，代理人的态度都是积极的，表达了鼓励和期待，并为她的成就感到高兴。这与微博帖子中的实际内容紧密吻合。

在质疑新闻爆发后，一些代理人表现出对江平的强烈信任（如左图所示），而其他代理人则倾向于赞同疑虑（如右图所示），这与微博帖子中的实际内容高度一致。

## IV 结论

本文通过结合 LLM 和 ABM，优化了新闻情感预测模型，并结合了 CA 和 SIR 模型的约束，以保持在 LLM 指导下群体态度的自然衰减。我们提出的 FDE-LLM 将用户分为意见领袖和意见跟随者。意见领袖基于 LLM 进行角色扮演，并受到 CA 模型的约束，而意见跟随者则是一个动态系统的一部分，集成了 CA 和 SIR 模型。这个创新设计显著提高了仿真精度和预测效率。

## 参考文献

+   [1] Pennycook G, Epstein Z, Mosleh M, Arechar AA, Eckles D, Rand DG. 将注意力转向准确性可以减少网络上的错误信息. 自然. 2021年4月22日;592(7855):590-5.

+   [2] Ginossar T, Cruickshank IJ, Zheleva E, Sulskis J, Berger-Wolf T. 跨平台传播：早期 Twitter COVID-19 对话中 YouTube 视频分享的疫苗相关内容、来源和阴谋论. 人类疫苗与免疫治疗学. 2022年1月31日;18(1):1-3.

+   [3] Budak C, Agrawal D, El Abbadi A. 限制社交网络中错误信息的传播. 2011年3月28日，世界广网国际会议论文集（第665-674页）。

+   [4] DeGroot MH. 达成共识. 美国统计学会杂志. 1974年3月1日;69(345):118-21.

+   [5] Deffuant G, Neau D, Amblard F, Weisbuch G. 互动代理之间的信仰混合. 复杂系统进展. 2000年;3(01n04):87-98.

+   [6] Rainer H, Krause U. 意见动态与有限信任：模型、分析与仿真.

+   [7] Clifford P, Sudbury A. 空间冲突模型. 生物计量学. 1973年12月1日;60(3):581-8.

+   [8] Holley RA, Liggett TM. 对弱相互作用的无限系统和选民模型的遍历定理。概率年鉴。1975年8月1日：643-63。

+   [9] Brünker, Felix 等人。“社交媒体在社会运动中的作用——来自Twitter上#metoo辩论的观察。”夏威夷国际系统科学会议（2020年）。

+   [10] Mou X, Wei Z, Huang X. 揭示真相并促进变革：面向基于智能体的大规模社会运动模拟。arXiv预印本arXiv:2402.16333\。2024年2月26日。

+   [11] 胡祖平、何建家，基于元胞自动机的在线舆论演化建模与仿真

+   [12] Gao C, Lan X, Lu Z, Mao J, Piao J, Wang H, Jin D, Li Y. S3：基于大语言模型赋能智能体的社交网络仿真系统。可在SSRN 4607026\获取。2023年10月19日。

+   [13] Park JS, O’Brien J, Cai CJ, Morris MR, Liang P, Bernstein MS. 生成型智能体：人类行为的互动模拟。发表于第36届ACM用户界面软件与技术年会论文集 2023年10月29日（第1-22页）。

+   [14] Chuang YS, Goyal A, Harlalka N, Suresh S, Hawkins R, Yang S, Shah D, Hu J, Rogers T. 使用基于LLM的智能体网络模拟意见动态。在《计算语言学学会会议论文集：NAACL 2024》2024年6月（第3326-3346页）。

+   [15] Liu Y, Chen X, Zhang X, Gao X, Zhang J, Yan R. 从怀疑到接受：模拟对虚假新闻的态度动态。arXiv预印本arXiv:2403.09498\。2024年3月14日。

+   [16] Cooper I, Mondal A, Antonopoulos CG. 针对COVID-19在不同社区传播的SIR模型假设。混沌、孤立子与分形。2020年10月1日；139:110057。
