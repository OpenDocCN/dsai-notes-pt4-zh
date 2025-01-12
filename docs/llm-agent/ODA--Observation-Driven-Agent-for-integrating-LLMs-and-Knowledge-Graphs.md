<!--yml

分类：未分类

日期：2025-01-11 12:42:29

-->

# ODA：用于集成LLMs与知识图谱的观察驱动代理

> 来源：[https://arxiv.org/html/2404.07677/](https://arxiv.org/html/2404.07677/)

Lei Sun¹¹1Equal contribution²²2Corresponding author¹ Zhengwei Tao¹¹1Equal contribution ² Youdi Li ¹ Hiroshi Arakawa ¹

¹Panasonic Connect株式会社，日本

²北京大学

tttzw@stu.pku.edu.cn

{sun.lei, ri.yutei, arakawa.hrs}@jp.panasonic.com

###### 摘要

大型语言模型（LLMs）与知识图谱（KGs）的结合在各种自然语言处理任务中取得了显著的成功。然而，现有的将LLMs与KGs结合的方法通常仅基于LLM对问题的分析来推动任务解决过程，忽视了KGs中蕴含的丰富认知潜力。为了解决这一问题，我们提出了观察驱动代理（ODA），一种专为涉及KGs的任务设计的全新AI代理框架。ODA通过全球观察整合了KG推理能力，通过观察、行动和反思的循环范式增强了推理能力。在观察过程中面对知识的指数级爆炸，我们创新性地设计了递归观察机制。随后，我们将观察到的知识整合到行动和反思模块中。通过广泛的实验，ODA在多个数据集上展示了最先进的性能，特别是在准确率上分别提高了12.87%和8.9%。我们的代码和数据可以在[ODA](https://github.com/lanjiuqing64/KGdata)上找到。

ODA：用于集成LLMs与知识图谱的观察驱动代理

Lei Sun¹¹1Equal contribution²²2Corresponding author¹ Zhengwei Tao¹¹1Equal contribution ² Youdi Li ¹ Hiroshi Arakawa ¹ ¹Panasonic Connect株式会社，日本 ²北京大学 tttzw@stu.pku.edu.cn {sun.lei, ri.yutei, arakawa.hrs}@jp.panasonic.com

## 1 引言

大型语言模型（LLMs）Touvron 等人（[2023](https://arxiv.org/html/2404.07677v2#bib.bib27)）；Scao 等人（[2022](https://arxiv.org/html/2404.07677v2#bib.bib23)）；Muennighoff 等人（[2022](https://arxiv.org/html/2404.07677v2#bib.bib16)）；Brown 等人（[2020](https://arxiv.org/html/2404.07677v2#bib.bib2)）在多种自然语言处理任务中展现了卓越的能力。尽管取得了令人印象深刻的成就，LLM 在处理需要超出其预训练内容的专门知识的查询时，仍然常常无法提供准确的回答。针对这一局限性，一种自然且富有前景的解决方案是将外部知识源（如知识图谱（KG））与 LLM 集成，以增强 LLM 的推理能力。KG 提供了结构化、明确且可解释的知识表示，提供了一种协同的方法来克服 LLM 的固有限制。LLM 与 KG 的融合在最近的研究中引起了广泛关注，Pan 等人（[2024](https://arxiv.org/html/2404.07677v2#bib.bib19)）提出了这一点，并在大量应用中取得了成果，Zhang 等人（[2023](https://arxiv.org/html/2404.07677v2#bib.bib35)）；Do 等人（[2024](https://arxiv.org/html/2404.07677v2#bib.bib5)）；Sun 等人（[2023b](https://arxiv.org/html/2404.07677v2#bib.bib25)）。

![参见标题](img/c7f28811d05f3e767380eba864f016e3.png)

图 1：LLM 与知识图谱（KG）集成的示例。观察到的实体以白色显示，未观察到的实体以灰色显示。代理选择的用于回答问题的实体以黄色高亮显示。

现有的将知识图谱（KGs）与大语言模型（LLMs）结合的任务解决方法可以分为两类。第一类方法是根据特定问题从KG中检索相关的三元组，Wang等人（[2023b](https://arxiv.org/html/2404.07677v2#bib.bib29)）；Luo等人（[2024](https://arxiv.org/html/2404.07677v2#bib.bib14)）；Jiang等人（[2023](https://arxiv.org/html/2404.07677v2#bib.bib11)）。第二类方法则采用探索-利用策略，根据问题引导图谱中的知识利用过程，Sun等人（[2023b](https://arxiv.org/html/2404.07677v2#bib.bib25)）；Guo等人（[2023](https://arxiv.org/html/2404.07677v2#bib.bib8)）。然而，两个类别的方法都是仅仅依赖LLM对问题的分析来导航任务解决过程，忽略了KG中蕴含的丰富认知潜力。知识图谱作为存储大量信息和符号事实的工具，应该与LLM一起深度参与推理过程，而不是仅仅作为一个静态的知识库存在，Pan等人（[2024](https://arxiv.org/html/2404.07677v2#bib.bib19)）。如图[1](https://arxiv.org/html/2404.07677v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ ODA: Observation-Driven Agent for integrating LLMs and Knowledge Graphs")上面的示例所示，LLM分析问题并导航到实体《荒野的呼唤》的地点关系。然而，这个实体有许多邻近的实体也具有这种关系，导致LLM错误地推断出加拿大为答案。相比之下，下面的面板展示了KG如何提供关键模式，揭示《荒野的呼唤》和《白 Fang》都共享地点“育空”。如果LLM能够事先观察到这些信息，它将准确地引导其推理过程朝着正确的答案（如下图所示）。因此，LLM应该采用一种全局观察方法，将KG中蕴含的广泛知识和复杂模式纳入其中。实现这一目标面临两个主要挑战：首先，对KG的全局观察可能导致三元组数量呈指数增长。如图[1](https://arxiv.org/html/2404.07677v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ ODA: Observation-Driven Agent for integrating LLMs and Knowledge Graphs")上面的示例所示，完全处理《荒野的呼唤》的所有3跳连接是不可行的。其次，将这种全面观察融入现有的LLM推理范式中，提出了另一个挑战。如何将这种观察与LLM的推理过程结合，对于解决任务至关重要。

有鉴于此，我们提出了一种新的框架——观察驱动代理（ODA），旨在充分且自主地整合LLM和KG的能力。ODA作为一个AI代理，专门为以KG为中心的任务设计。ODA采用观察、行动和反思的循环模式。在ODA中，我们设计了一个新的观察模块，以高效地提取KG的自主推理模式。我们的观察模块通过递归进展避免了三元组的指数增长问题。该方法确保了ODA在集成KG和LLM的能力的同时，减轻了KG中过多数据带来的挑战，提高了效率和准确性。在观察阶段之后，ODA通过自主地将LLM推理得到的洞察与观察到的KG模式结合起来采取行动。ODA可以执行三种不同类型的行动：邻居探索、路径发现和回答问题。随后，ODA会对其内部状态进行反思，考虑到其行动的结果和先前的观察。这个迭代过程将持续进行，直到ODA完成当前任务。

我们进行了一系列广泛的实验，验证了ODA在四个数据集上的有效性：QALD10-en、T-REx、Zero-Shot RE和Creak。值得注意的是，与竞争性基准相比，我们的方法在表现上达到了最先进（SOTA）水平。具体来说，在QALD10-en和T-REx数据集上，我们观察到准确率分别提高了12.87%和8.9%。我们的贡献总结如下：

+   $\bullet$

    我们提出了ODA，一个为KG任务量身定制的AI代理。ODA通过观察将KG的推理能力整合进来。

+   $\bullet$

    我们设计了行动与反思模块，将观察集成到LLM推理中。这一策略利用了KG和LLM的自主推理，二者协同工作。

+   $\bullet$

    我们在四个数据集上进行了实验，并达到了SOTA的表现。

![参见标题](img/9506ca0addb5d0ca83170bcf62555176.png)

图2：ODA的整体框架。

## 2 方法

本文的目标是解决与KG相关的任务。设$q$为用户问题。任务$T$可以定义为在给定问题$q$、任务相关实体$E=\{e_{0},e_{1},...,e_{k}\}$和KG $G$的情况下生成答案$Y$。形式化地，任务$T$可以表示为：

|  | $T:(q,E),G\rightarrow Y$ |  |
| --- | --- | --- |

![参见标题](img/a8d2f44a844cabab5ee958d40a0d9960.png)

图3：ODA的示例工作流。在这个例子中，ODA首先用实体Johann Wolfgang von Goethe启动观察。在左侧的第一次迭代中，选择Johann Wolfgang von Goethe的邻居探索，并将反映的三元组（Johann Wolfgang von Goethe, unmarried Partner, Lili Schöneman）存储在内存中。随后，Lili Schöneman的观察引导ODA选择邻居探索动作，导致将三元组（Lili Schöneman, place of birth, Offenbach am Main）存储在内存中，如右侧所示。当积累了足够的知识后，ODA触发答案动作，正确地识别出Offenbach am Main作为答案。

采用迭代方法，ODA解决了知识图谱（KG）任务中固有的挑战。与现有将大规模语言模型（LLM）与KG结合并仅依赖于分析LLM查询的方法不同，ODA自主地将从KG中观察到的知识整合到整个推理过程中，从而做出更为明智的决策。为了实现这一目标，我们的ODA系统，如图[2](https://arxiv.org/html/2404.07677v2#S1.F2 "Figure 2 ‣ 1 Introduction ‣ ODA: Observation-Driven Agent for integrating LLMs and Knowledge Graphs")所示，主要包括三个关键模块用于任务解决：

+   •

    观察：该模块高效地观察并处理来自KG环境的相关知识。在每次迭代$i$中，它构建一个观察子图（记作$O_{i}$）。通过利用从KG中获得的见解和模式，子图被自主地整合进推理LLM。这种协同整合使ODA同时具备了LLM和KG的增强能力，从而更有效地完成任务。

+   •

    行动：该模块利用观察子图$O_{i}$和ODA内存（记作$M_{<i}$）中的信息，由$a_{i}$表示，战略性地选择最合适的行动执行在KG上，确保问题的准确回答。

+   •

    反思：利用观察子图$O_{i}$，反思模块通过反思从行动步骤中获得的知识提供反馈。反映的知识随后存储在内存$M_{i}$中，供下一次迭代使用，促进连续的推理。

通过这个迭代过程，ODA在每次迭代$i$中动态更新其观察子图$O_{i}$和内存$M_{i}$。每个模块将在以下章节中详细讨论。

### 2.1 观察

观察模块旨在检查全局KG知识并在KG环境中导航自主推理过程。在每次迭代$i$中，它利用任务相关的实体$E_{i}$和问题$q$生成一个观察子图$O_{i}$。这个过程可以表示为：

|  | $O_{i}=\text{Observation}([E_{i},q])$ |  |
| --- | --- | --- |

最初，任务相关的实体通过问题$q$中嵌入的实体进行填充。

对于KG中心的任务，观察面临节点数量爆炸的问题。为了解决在观察子图更新过程中扩展性挑战，我们提出了一种$D$轮观察策略，其中$D$表示最大跳跃深度。每一轮有两个步骤：更新和优化。更新步骤关注于扩展子图，而优化步骤确保子图的适当大小，并且不丢失重要信息。

对于每个$e \in E_{i}$，观察模块在跳跃深度$d$处初始化观察实体集$E_{o}^{d}=\{e\}$，其中$d$表示当前在KG中的搜索深度。更新和优化的两步操作将为每个实体$e$迭代执行，直到达到$D$为止。具体细节如下所示：

算法1 观察

需求：问题$q$，限制$D$，$N$和$P$

使用任务相关实体$E_{i}$初始化$q$中的实体，对于$e \in E_{i}$，执行以下操作：  

更新：

+   •

    对于$E_{o}^{d}$中的每个实体$e$，从KG中提取邻接三元组。每个三元组的形式为$[e,r,t]$，其中$r$表示关系，$t$表示尾实体。

+   •

    通过测量它们的嵌入向量的余弦相似度，计算问题和$r$与$t$的组合表示之间的相似度得分¹¹1我们使用OpenAI的GPT text-embedding-ada-002模型进行编码。：

    |  | $\text{余弦相似度}(\mathbf{v}_{q},\mathbf{v}_{r+t})=\frac{\mathbf{v}_{q} \cdot \mathbf{v}_{r+t}}{\|\mathbf{v}_{q}\|\|\mathbf{v}_{r+t}\|}$ |  |
    | --- | --- | --- |

+   •

    与$E$中实体相关的所有三元组被按相似度得分降序排列。

+   •

    前$N$个三元组被添加到观察子图$O_{i}$中。

优化：

+   •

    通过保留相似度得分最高的前$P\%$三元组，进一步优化前$N$个三元组。

+   •

    从优化后的前$P\%$三元组中识别尾实体，作为下一轮观察的起始观察实体$E_{o}^{d}$。

### 2.2 动作

利用LLM的强大功能，动作模块设计战略性提示来生成最优动作。基于其记忆$M_{<i}$、观察子图$O_{i}$和历史动作$a_{<i}$，ODA选择最准确的动作$a_{i}$。

|  | $a_{i}=\text{动作}([O_{i},a_{<i},M_{<i}])$ |  |
| --- | --- | --- |

我们提出了三种核心动作，旨在增强ODA的能力：

+   •

    邻居探索：该动作探索任务相关实体 $E_{i}$ 的知识图谱邻域，并检索所有邻接的三元组。这有助于为 ODA 构建上下文，并理解知识图谱中的相互关联性。

+   •

    路径发现：给定任务相关实体 $E_{i}$ 中的两个实体，该动作会搜索连接它们的所有可能路径。每条路径由相互连接的三元组组成，允许 ODA 探索各种连接，并可能揭示隐藏的关系。

+   •

    回答：仅在所需信息存在于内存 $M_{<i}$ 中时，该动作才会响应问题。

在选择一个回答动作后，ODA 会停止观察、行动和反思的迭代循环。通过利用记忆中可靠的知识 $M_{<i}$，它可以直接生成问题的答案。或者，如果选择了邻居探索或路径发现动作，ODA 会战略性地从知识图谱（KG）中提取相关知识作为一组三元组。这些提取的三元组随后会被输入到后续的反思步骤中进行进一步处理。这里使用的提示可以在表格 [7](https://arxiv.org/html/2404.07677v2#A2.T7 "Table 7 ‣ Appendix B Prompt ‣ ODA: Observation-Driven Agent for integrating LLMs and Knowledge Graphs") 中找到。

### 2.3 反思

反思模块在评估来自行动步骤生成的三元组并随后更新 ODA 内存 $M_{i}$ 中起着至关重要的作用。该模块专为知识图谱任务设计，内存 $M_{i}$ 采用子图格式，包含与知识图谱固有结构对齐的路径网络，旨在优化效率和相关性。通过自主整合观察子图 $O_{i}$ 和现有内存 $M_{<i}$，反思模块提供了宝贵的反馈，指导未来的决策。这一过程可以形式化为：

|  | $M_{i}=\text{反思}([O_{i},a_{i},M_{<i}])$ |  |
| --- | --- | --- |

由于内存 $M_{i}$ 结构化为一组路径网络，反思模块会在这些路径中导航，以识别第一个适合整合反思三元组的路径。这种适配性来自于将所选路径中最后一个三元组的尾部 $t$ 与反思三元组的实体 $e$ 对齐。如果找到匹配路径，则将反思三元组附加到该路径上。否则，将基于反思三元组创建一条新路径。反思三元组的最大大小记为 $K$。

随后，反思三元组中的尾部实体被指定为下一次迭代的任务相关实体。反思模块中使用的具体提示描述见表格 [8](https://arxiv.org/html/2404.07677v2#A2.T8 "Table 8 ‣ Appendix B Prompt ‣ ODA: Observation-Driven Agent for integrating LLMs and Knowledge Graphs")。

观察、行动和反射模块协同工作，直到触发“答案”行动或达到最大迭代限制。图[3](https://arxiv.org/html/2404.07677v2#S2.F3 "图3 ‣ 2 方法 ‣ ODA：集成LLM和知识图谱的观察驱动代理")展示了观察、行动和反射如何协同工作。

## 3 实验

| 数据集 | 测试 | 实体 | 类型 | 许可证 |
| --- | --- | --- | --- | --- |
| QALD10-en | 333 | 396 | 多跳 | MIT许可证 |
| T-REx | 5000 | 4943 | 插槽填充 | MIT许可证 |
| Zero-Shot RE | 3724 | 3657 | 插槽填充 | MIT许可证 |
| Creak | 1371 | 516 | 事实核查 | MIT许可证 |

表1：数据集统计信息。实体指的是从数据集中的所有问题中派生出的实体大小。

### 3.1 数据集

为了评估我们ODA的性能，我们在四个KBQA数据集上进行了实验：QALD10-en Perevalov et al.（[2022](https://arxiv.org/html/2404.07677v2#bib.bib20)），Creak Onoe et al.（[2021](https://arxiv.org/html/2404.07677v2#bib.bib17)），T-REx Elsahar et al.（[2018](https://arxiv.org/html/2404.07677v2#bib.bib6)），以及Zero-Shot RE Petroni et al.（[2021](https://arxiv.org/html/2404.07677v2#bib.bib21)）。每个数据集的详细规格见表[1](https://arxiv.org/html/2404.07677v2#S3.T1 "表1 ‣ 3 实验 ‣ ODA：集成LLM和知识图谱的观察驱动代理")。我们使用了Hits@1Sun et al.（[2019](https://arxiv.org/html/2404.07677v2#bib.bib26)）的准确度作为评估指标，准确匹配为评估标准。

### 3.2 设置

我们通过OpenAI API使用了GPT-4 OpenAI模型（[2023](https://arxiv.org/html/2404.07677v2#bib.bib18)）。在整个实验过程中，我们始终将GPT-4的温度值配置为0.4，并将最大标记长度设置为500。

对于观察步骤，我们根据初步实验调整了关键参数。我们将$P_{t}$设置为10，$N_{t}$设置为50。此后，ODA循环最多限制为8次迭代。最后，最大跳跃深度$D$设置为3。至于反射模块，我们将反射三元组的大小$K$设置为15。

为了建立Wikidata知识图谱数据库并从中检索信息，我们使用了simple-wikidata-db¹¹1https://github.com/neelguha/simple-wikidata-db Python库。该库提供了各种脚本，用于下载Wikidata数据转储，将其组织成暂存文件，并在这些暂存文件中的数据上执行分布式查询。具体来说，我们将Wikidata数据转储部署在五个AWS EC2实例上，每个实例包括一个768GB内存和48个核心的机器。

考虑到我们的ODA在很大程度上依赖于与KG的持续互动，我们发现，实时从AWS上提取所需的Wikidata知识，在QALD10-en数据集中的每对问答的平均完成时间为50秒。然而，随着KBQA数据集的扩展，在AWS上使用Wikidata数据库的成本变得非常昂贵。因此，为了应对计算开销，我们设计了一种解决方案，即为每个KBQA数据集生成离线子图。这个离线子图捕捉了每个数据集中实体的3跳范围内的所有三元组，包括实体和相关关系的属性。值得注意的是，生成T-REx数据集的子图（该数据集包含4943个实体，如表[1](https://arxiv.org/html/2404.07677v2#S3.T1 "Table 1 ‣ 3 Experiments ‣ ODA: Observation-Driven Agent for integrating LLMs and Knowledge Graphs")所示）大约需要54分钟42.834秒。

### 3.3 基准模型

为了全面评估ODA的有效性，我们对多个SOTA模型在不同类别上的表现进行了严格的基准测试。比较包括各种模型，从不使用外部知识的基于提示的方法开始。这些方法包括直接使用GPT-3.5和GPT-4进行回答，以及Self-Consistency Wang等人（[2023c](https://arxiv.org/html/2404.07677v2#bib.bib30)）和CoT Sun等人（[2023b](https://arxiv.org/html/2404.07677v2#bib.bib25)）的方法。另一方面，还考虑了结合知识的模型，这些模型结合了像SPARQL-QA Santana等人（[2022](https://arxiv.org/html/2404.07677v2#bib.bib22)）、RACo Yu等人（[2022](https://arxiv.org/html/2404.07677v2#bib.bib34)）、RAG Petroni等人（[2021](https://arxiv.org/html/2404.07677v2#bib.bib21)）和Re2G Glass等人（[2022](https://arxiv.org/html/2404.07677v2#bib.bib7)）等技术。除此之外，还有ToG Sun等人（[2023a](https://arxiv.org/html/2404.07677v2#bib.bib24)）模型，它将LLM与KG集成，以提高问答能力。

### 3.4 主要结果

我们的ODA方法优于现有方法，如表[2](https://arxiv.org/html/2404.07677v2#S3.T2 "Table 2 ‣ 3.4 Main Result ‣ 3 Experiments ‣ ODA: Observation-Driven Agent for integrating LLMs and Knowledge Graphs")所示。与GPT-4的直接回答相比，我们的方法平均提高了19.58%的准确率；与微调模型相比提高了19.28%；与TOG相比提高了7.09%。这些结果展示了我们的方法在与其他最先进方法比较时的高效性和有效性。

此外，我们的ODA在多个数据集上显著优于基于提示的方法，特别是在Zero-Shot REx和QALD10-en上分别取得了65.50%和23.77%的提升。这些结果强调了利用外部知识图谱进行推理和完成问答任务的重要性。

与微调方法相比，我们的ODA方法展现了更优的性能。具体来说，我们的方法在QALD10-en数据集上提高了21.27%，在Creak数据集上提高了6.99%，在Zero-Shot RE数据集上提高了50.56%。值得注意的是，我们的方法采用的LLM与知识图谱之间的交互比基于数据驱动的微调技术更为有效，尽管不需要显式训练。

我们的ODA方法在大多数数据集上相较于ToG方法展现了显著的性能提升，分别为12.87%（QALD10-en）、8.9%（T-REx）和7%（Zero-Shot RE），尽管这两种方法都利用了大型语言模型和知识图谱。这一性能差异突显了我们观察模块的关键作用，以及自主融合知识图谱推理的有效性。具体来说，我们的方法在QALD10-en数据集上表现得更为强劲，该数据集以多跳和复杂推理要求著称。这一成就凸显了我们ODA在有效利用知识图谱中的丰富知识和模式方面的能力，成功地将大型语言模型和知识图谱的自主推理优势结合，解决复杂问题。

| 方法 | QALD10-en | Creak | T-REx | Zero-Shot RE | 平均 |
| --- | --- | --- | --- | --- | --- |
| 无知识 |
| 直接回答（GPT3.5） | 44.74^((1)) | 90.00^((1)) | 37.78^((1)) | 37.14^((1)) | 52.42 |
| 直接回答（GPT4） | 57.10^((1)) | 94.52^((1)) | 57.72^((1)) | 55.50^((1)) | 66.21 |
| 自一致性（GPT3.5）Wang等（[2023c](https://arxiv.org/html/2404.07677v2#bib.bib30)） | 45.30^((1)) | 90.80^((1)) | 41.80^((1)) | 45.40^((1)) | 55.83 |
| COT（GPT3.5）Sun等（[2023a](https://arxiv.org/html/2404.07677v2#bib.bib24)） | 42.90^((1)) | 90.10^((1)) | 32.00^((1)) | 28.80^((1)) | 48.45 |
| 带知识/微调 |
| SOTA | 45.40¹ | 88.20² | 87.70³ | 44.74⁴ | 66.51 |
| 带知识/零-shot（GPT-4） |
| TOG-R Sun等（[2023a](https://arxiv.org/html/2404.07677v2#bib.bib24)） | 54.70^((1)) | 95.40^((1)) | 75.50^((1)) | 86.90^((1)) | 78.13 |
| TOG Sun等（[2023a](https://arxiv.org/html/2404.07677v2#bib.bib24)） | 53.80^((1)) | 95.60^((1)) | 77.10^((1)) | 88.30^((1)) | 78.70 |
| ODA（我们的方法） | 66.67^((1)) | 95.19^((1)) | 86.00^((1)) | 95.30^((1)) | 85.79 |

表2：不同方法的性能比较。粗体分数表示在所有基于GPT的零-shot方法中的最佳表现。微调的SOTA包括：1：SPARQL-QASantana等（[2022](https://arxiv.org/html/2404.07677v2#bib.bib22)），2：RACoYu等（[2022](https://arxiv.org/html/2404.07677v2#bib.bib34)），3：Re2GGlass等（[2022](https://arxiv.org/html/2404.07677v2#bib.bib7)），4：RAGPetroni等（[2021](https://arxiv.org/html/2404.07677v2#bib.bib21)）。

## 4 讨论

为了更好地理解影响我们ODA的关键因素，我们进行了广泛的分析实验。为了节省计算资源，我们保留了之前提到的数据集（QALD10-en、Creak、T-REx和Zero-Shot RE），但为Creak、T-REx和Zero-Shot RE随机抽取了每个400个示例。

### 4.1 观察的影响

为了评估观察模块的效果，我们进行了包括没有观察的模型在内的全面实验。在行动步骤中，ODA仅根据记忆选择行动。随后，反思步骤基于行动输出的三元组进行反思，并在没有观察指导的情况下更新记忆。

我们进行了一项统计比较，以评估具有观察和没有观察的ODA在所有数据集上的表现（见表[3](https://arxiv.org/html/2404.07677v2#S4.T3 "表3 ‣ 4.1 观察的影响 ‣ 4 讨论 ‣ ODA: 基于观察的智能体用于集成大语言模型和知识图谱")）。结果表明，具有观察的ODA优于没有观察的ODA，平均提升了3.14%。具体而言，对于QALD10-en数据集，具有观察的ODA比没有观察的ODA提高了5.41%。由于QALD10-en涉及多跳推理，因此具有观察的ODA性能的提升表明观察模块增强了智能体的推理能力，使得行动选择和反思更为准确。

我们可以通过一个实际案例进一步说明观察模块的好处。在这个场景中（见表[6](https://arxiv.org/html/2404.07677v2#A1.T6 "表6 ‣ 附录A 案例研究 ‣ ODA: 基于观察的智能体用于集成大语言模型和知识图谱")），问题是《野性的呼唤》和《白 Fang》这两部杰克·伦敦最著名的作品发生在哪里？没有观察时，ODA生成了记忆，例如（《野性的呼唤》，叙述地点，加拿大），最终给出了错误的答案加拿大。然而，有了观察模块，ODA正确推理出记忆，例如（《野性的呼唤》，叙述地点，育空），（《白 Fang》，叙述地点，育空）。因此，具有观察的ODA提供了正确的答案——育空。这个案例展示了观察模块如何提高行动选择和反思的准确性，从而增强ODA的推理能力。

通过整合观察信息，ODA的推理能力发生了显著跃升，因此能够生成准确的答案。这一提升源于观察模块与知识图谱（KG）自主推理能力的协同作用，以及大语言模型（LLM），进一步放大了这些优势。

| 方法 | QALD10-en | Creak | T-REx | Zero-Shot RE | 平均 |
| --- | --- | --- | --- | --- | --- |
| 没有观察 | 61.26 | 95.50 | 82.00 | 91.75 | 82.63 |
| 基于相似度的反思 | 61.26 | 95.20 | 83.20 | 93.50 | 83.29 |
| 基于随机的反射 | 58.56 | 89.00 | 79.50 | 92.00 | 79.77 |
| 生成事实反射 | 63.66 | 91.00 | 80.00 | 93.75 | 82.10 |
| ODA | 66.67 | 96.00 | 85.40 | 95.00 | 85.77 |

表 3：消融比较

### 4.2 观察对反射的影响

在本节中，我们讨论了观察对反射模块的影响。设计了三种非观察反射方法来验证观察是否能够增强反射的有效性。基于相似性的反射方法通过计算相似性来反思来自行动步骤的三元组。在这种方法中，三元组首先根据$r+t$与问题之间的相似性得分进行排序。然后选择得分最高的前$K$个三元组，并将它们存储在内存中以供下次迭代使用。基于随机的反射方法随机从行动输出中选择$K$个三元组，并将其存储在内存中。最后，生成事实方法创建$K$个与自然语言问题相关的事实进行存储。所有方法都使用$K=15$的设置。

表[3](https://arxiv.org/html/2404.07677v2#S4.T3 "表 3 ‣ 4.1 观察的效果 ‣ 4 讨论 ‣ ODA：集成LLM和知识图谱的观察驱动代理")展示了我们的ODA在所有三种非观察方法中的优势。与基于相似性的方法相比，ODA平均准确率提高了2.48%，与基于随机的方法相比提高了6.00%，与生成事实的方法相比提高了3.66%。

在特定场景下（见表[5](https://arxiv.org/html/2404.07677v2#A1.T5 "表 5 ‣ 附录 A 案例研究 ‣ ODA：集成LLM和知识图谱的观察驱动代理")），在回答问题“东京的首府是哪里？”时，生成事实方法产生了有问题的事实，例如“东京是东京的首府”和“东京是日本的首府”。这些本质上是基于给定问题，由LLM生成的虚假信息，误导了代理，导致错误的答案。相比之下，我们的ODA通过观察进行的反射产生了事实性知识，（东京，实例，属于，日本的都道府县），（东京，首府，日本）和（东京，首府，新宿），从而使ODA能够正确回答该问题。

表[3](https://arxiv.org/html/2404.07677v2#S4.T3 "表 3 ‣ 4.1 观察的效果 ‣ 4 讨论 ‣ ODA：集成LLM和知识图谱的观察驱动代理")的结果表明，观察使反射模块能够生成更准确的记忆，从而提高了ODA的问答准确率。这个结果强调了利用知识图谱（KG）自主推理能力以及促进KG和LLM之间深度协作的重要性。

### 4.3 不同骨干模型的表现

为了评估ODA在不同骨干网络上的效果，我们分析了它在T-REx和QALD10-en数据集上的性能表现。我们使用了三种骨干网络：GPT-3.5、GPT-4和DeepSeek-V2（DeepSeek-AI等，([2024](https://arxiv.org/html/2404.07677v2#bib.bib4))）。DeepSeek-V2作为一种强大、经济高效的专家混合语言模型脱颖而出。值得注意的是，DeepSeek-V2在标准基准测试中超过了LLaMA3 70B Instruct的表现。

如表[4](https://arxiv.org/html/2404.07677v2#S4.T4 "Table 4 ‣ 4.3 Performance across Different Backbone Models ‣ 4 Discussion ‣ ODA: Observation-Driven Agent for integrating LLMs and Knowledge Graphs")所示，我们的ODA方法在使用GPT-3.5、GPT-4和DeepSeek-V2的直接回答方法上显著优于其他方法。特别地，在使用GPT-3.5模型进行QALD10-en数据集的直接回答时，ODA表现出显著的30.4%的提升。这个实验表明，ODA在不同LLM之间具有较好的泛化能力。

| 方法 | T-REx | QALD10-en |
| --- | --- | --- |
| 直接回答（GPT3.5） | 37.60 | 44.74 |
| ODA (GPT3.5) | 68.00 | 49.71 |
| 直接回答（GPT4） | 57.44 | 57.10 |
| ODA (GPT4) | 86.00 | 66.67 |
| 直接回答（DeepSeek-V2） | 32.86 | 41.14 |
| ODA (DeepSeek-V2) | 62.67 | 57.36 |

表4：使用不同骨干模型的性能比较

## 5 相关工作

#### 增强知识图谱的LLM

知识图谱增强语言模型在处理需要与KG集成的任务时，采用两种主要方法。第一种方法是根据提出的问题从KG中提取相关的三元组。Wang等人（[2023b](https://arxiv.org/html/2404.07677v2#bib.bib29)）提示LLM生成明确的知识证据，结构化为三元组，而Jiang等人（[2023](https://arxiv.org/html/2404.07677v2#bib.bib11)）开发了专门的接口，用于从结构化数据中收集相关证据，使LLM能够基于这些信息专注于推理任务。Baek等人（[2023](https://arxiv.org/html/2404.07677v2#bib.bib1)）通过评估问题与相关事实之间的语义相似性，检索与输入问题相关的事实，然后将这些事实添加到输入中。同时，Li等人（[2023](https://arxiv.org/html/2404.07677v2#bib.bib13)）通过适应KG中的知识，迭代地完善推理理由。Wang等人（[2023a](https://arxiv.org/html/2404.07677v2#bib.bib28)）通过预定义模板解析复杂问题，从KG中检索实体，并相应地生成答案。Luo等人（[2024](https://arxiv.org/html/2404.07677v2#bib.bib14)）采用规划-检索-推理框架，生成由KG支持的关系路径，从而增强LLM的推理能力。最近，引入了图检索增强生成（GRAG），通过检索适当的知识子图而非三元组来增强推理能力。He等人（[2024](https://arxiv.org/html/2404.07677v2#bib.bib9)）；Hu等人（[2024](https://arxiv.org/html/2404.07677v2#bib.bib10)）；Mavromatis和Karypis（[2024](https://arxiv.org/html/2404.07677v2#bib.bib15)）提出了这一方法。在GRAG方法中，首先将子图编码为图嵌入，然后根据与查询的相似性进行检索。Hu等人（[2024](https://arxiv.org/html/2404.07677v2#bib.bib10)）提出了一个额外的步骤，即在每个检索到的子图中过滤掉无关实体。

第二种方法采用探索-利用策略，指导图中的知识利用过程。Sun等人（[2023b](https://arxiv.org/html/2404.07677v2#bib.bib25)）在知识图谱（KG）上执行迭代束搜索，以识别最有前景的推理路径并报告结果。Guo等人（[2023](https://arxiv.org/html/2404.07677v2#bib.bib8)）通过一个迭代过程，选择性地从KG中积累支持信息，结合LLM的见解来解决问题。HyKGE [Jiang等人](https://arxiv.org/html/2404.07677v2#bib.bib12)首先通过LLM生成问题的假设。然后，根据假设中的实体从KG中检索知识，并基于这些知识回答问题。为了解决不完整的KG，Xu等人（[2024](https://arxiv.org/html/2404.07677v2#bib.bib32)）在缺失某些知识时添加生成操作。

尽管这些方法利用了知识图谱的结构化知识，但搜索或检索过程是由 LLM 对目标问题的推理驱动的。我们的方法是首个通过我们新颖的观察机制将现有模式融入 KG，以深度绑定 LLM 和 KG 的推理能力的工作。

#### 人工智能代理

在人工智能代理领域，Yao 等人（[2022](https://arxiv.org/html/2404.07677v2#bib.bib33)）利用大语言模型（LLMs）将推理轨迹的生成与任务特定的行动交织在一起。Wu 等人（[2023](https://arxiv.org/html/2404.07677v2#bib.bib31)）提出了一个可适应的对话代理框架。该框架可以在多种模式下运行，结合 LLMs、人类输入和辅助工具，形成一个灵活多样的系统。Chen 等人（[2023](https://arxiv.org/html/2404.07677v2#bib.bib3)）专注于创建能够解决复杂任务的专家代理。

## 6 结论

本研究设计了一个针对知识图谱（KG）中心任务的 ODA 框架。在 ODA 中，我们引入了 KG 观察机制，以自主地将 KG 的推理能力与 LLM 结合。我们首先提出了一种观察方法，以减轻在处理复杂任务时知识图谱中三元组数量爆炸的问题。然后，我们将观察机制融合到行动和反思模块中，以进一步提升整体性能。我们进行了大量实验，结果清晰地展示了我们框架的有效性，突显了它在四个 KBQA 数据集上提升性能的能力，尤其是在处理复杂问题时。

## 局限性

鉴于知识图谱相关任务的多样性，跨多个领域并且需要广泛的能力，实施多代理系统有望显著提高任务性能。我们将我们观察机制与多代理系统的结合留待未来工作。此外，我们当前的方法在应用于缺失关系或实体的知识图谱（如 Freebase）时面临计算挑战。因此，在后续工作中，我们计划解决这些问题，并更全面地探索 Freebase 的使用。

## 参考文献

+   Baek 等人（2023） Jinheon Baek, Alham Fikri Aji 和 Amir Saffari. 2023. 知识增强语言模型提示用于零-shot知识图谱问答。*arXiv 预印本 arXiv:2306.04136*。

+   Brown 等人（2020）Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell 等人. 2020. 语言模型是少样本学习者。*神经信息处理系统进展*, 33:1877–1901。

+   Chen 等人（2023）Guangyao Chen, Siwei Dong, Yu Shu, Ge Zhang, Jaward Sesay, Börje F Karlsson, Jie Fu 和 Yemin Shi. 2023. Autoagents：自动代理生成框架。*arXiv 预印本 arXiv:2309.17288*。

+   DeepSeek-AI 等人 (2024) DeepSeek-AI、:、肖斌、陈德立、陈冠廷、陈山黄、戴大迈、邓成奇、丁洪辉、董凯、杜秋实、傅哲、高华作、高凯歌、高文君、葛睿琪、关康、郭大雅、郭建忠、郝光波、郝哲文、何颖、胡文杰、黄盼盼、李尔航、李国伟、李家时、李耀、李Y.K.、梁文峰、林芳云、刘A.X.、刘博、刘文、刘晓东、刘欣、刘一源、陆浩宇、陆尚昊、罗富力、马世荣、聂晓涛、裴天、邱一志、曲慧、任同政、任泽辉、阮冲、沙张利、邵志鸿、宋俊晓、苏学成、孙景翔、孙耀峰、唐名辉、王冰轩、王佩怡、王世瑜、王耀辉、王永吉、吴彭、吴Y.、谢欣、谢振达、谢子威、熊一良、徐汉伟、徐R.X.、徐彦鸿、杨德建、游宇翔、于水平、于兴凯、张B.、张浩伟、张乐聪、张李跃、张名川、张名华、张文涛、张艺超、张成钢、赵耀、周尚岩、周顺峰、朱启豪 和 邹宇恒。2024。[Deepseek llm：通过长期主义扩展开源语言模型](http://arxiv.org/abs/2401.02954)。

+   Do 等人 (2024) Quyet V Do、方天青、刁诗哲、王兆威 和 宋扬秋。2024。Constraintchecker：一个用于大规模语言模型推理常识知识库的插件。*arXiv 预印本 arXiv:2401.14003*。

+   Elsahar 等人 (2018) Hady Elsahar、Pavlos Vougiouklis、Arslen Remaci、Christophe Gravier、Jonathon Hare、Frederique Laforest 和 Elena Simperl。2018。[T-REx：自然语言与知识库三元组的大规模对齐](https://aclanthology.org/L18-1544)。发表于 *第十一届国际语言资源与评估会议 (LREC 2018)*，日本宫崎。欧洲语言资源协会 (ELRA)。

+   Glass 等人 (2022) Michael Glass、Gaetano Rossiello、Md Faisal Mahbub Chowdhury、Ankita Naik、蔡鹏山 和 Alfio Gliozzo。2022。[Re2G：检索、重排、生成](https://doi.org/10.18653/v1/2022.naacl-main.194)。发表于 *2022年北美计算语言学协会：人类语言技术会议论文集*，第2701-2715页，美国西雅图。计算语言学协会。

+   Guo 等人 (2023) 郭铁铮、杨清文、王晨、刘彦仪、李攀、唐家伟、李大鹏、文英友。2023。Knowledgenavigator：利用大规模语言模型增强知识图推理。*arXiv 预印本 arXiv:2312.15880*。

+   He 等人 (2024) 何小新、田一君、孙一飞、Nitesh V Chawla、Thomas Laurent、Yann LeCun、Xavier Bresson 和 Bryan Hooi。2024。G-retriever：用于文本图理解和问题回答的检索增强生成。*arXiv 预印本 arXiv:2402.07630*。

+   Hu 等人（2024）Yuntong Hu, Zhihan Lei, Zheng Zhang, Bo Pan, Chen Ling, 和 Liang Zhao. 2024. Grag: 图检索增强生成。*arXiv 预印本 arXiv:2405.16506*。

+   Jiang 等人（2023）Jinhao Jiang, Kun Zhou, Zican Dong, Keming Ye, Wayne Xin Zhao, 和 Ji-Rong Wen. 2023. StructGPT: 一个面向大型语言模型在结构化数据上进行推理的一般框架。*arXiv 预印本 arXiv:2305.09645*。

+   （12）Xinke Jiang, Ruizhe Zhang, Yongxin Xu, Rihong Qiu, Yue Fang, Zhiyuan Wang, Jinyi Tang, Hongxin Ding, Xu Chu, Junfeng Zhao 等人。Hykge: 一个增强假设知识图的框架，用于准确且可靠的医疗语言模型响应。

+   Li 等人（2023）Xingxuan Li, Ruochen Zhao, Yew Ken Chia, Bosheng Ding, Shafiq Joty, Soujanya Poria, 和 Lidong Bing. 2023. Chain-of-knowledge: 通过在异构来源上进行动态知识适配来为大型语言模型提供基础。*arXiv 预印本 arXiv:2305.13269*。

+   Luo 等人（2024）Linhao Luo, Yuan-Fang Li, Gholamreza Haffari, 和 Shirui Pan. 2024. [基于图的推理：忠实且可解释的大型语言模型推理](http://arxiv.org/abs/2310.01061)。

+   Mavromatis 和 Karypis（2024）Costas Mavromatis 和 George Karypis. 2024. GNN-RAG: 面向大型语言模型推理的图神经检索。*arXiv 预印本 arXiv:2405.20139*。

+   Muennighoff 等人（2022）Niklas Muennighoff, Thomas Wang, Lintang Sutawika, Adam Roberts, Stella Biderman, Teven Le Scao, M Saiful Bari, Sheng Shen, Zheng-Xin Yong, Hailey Schoelkopf 等人. 2022. 通过多任务微调实现跨语言泛化。*arXiv 预印本 arXiv:2211.01786*。

+   Onoe 等人（2021）Yasumasa Onoe, Michael J. Q. Zhang, Eunsol Choi, 和 Greg Durrett. 2021. [Creak: 一个面向实体知识常识推理的数据集](http://arxiv.org/abs/2109.01653)。

+   OpenAI（2023）R OpenAI. 2023. GPT-4 技术报告。*arXiv*，第2303–08774页。

+   Pan 等人（2024）Shirui Pan, Linhao Luo, Yufei Wang, Chen Chen, Jiapu Wang, 和 Xindong Wu. 2024. 统一大型语言模型和知识图谱：一条发展路线图。*IEEE 知识与数据工程学报*。

+   Perevalov 等人（2022）Aleksandr Perevalov, Dennis Diefenbach, Ricardo Usbeck, 和 Andreas Both. 2022. [Qald-9-plus: 一个由母语者翻译的面向 DBpedia 和 Wikidata 问答的多语言数据集](https://doi.org/10.1109/ICSC52841.2022.00045)。在 *2022 IEEE 第16届语义计算国际会议（ICSC）*，第229–234页。

+   Petroni 等人（2021）Fabio Petroni, Aleksandra Piktus, Angela Fan, Patrick Lewis, Majid Yazdani, Nicola De Cao, James Thorne, Yacine Jernite, Vladimir Karpukhin, Jean Maillard, Vassilis Plachouras, Tim Rocktäschel, 和 Sebastian Riedel. 2021. [KILT: 面向知识密集型语言任务的基准](https://doi.org/10.18653/v1/2021.naacl-main.200)。在 *2021年北美计算语言学会年会：人类语言技术论文集*，第2523–2544页，线上。计算语言学协会。

+   Santana et al. (2022) Manuel Alejandro Borroto Santana, Bernardo Cuteri, Francesco Ricca, and Vito Barbara. 2022. [SPARQLQA进入QALD挑战](https://ceur-ws.org/Vol-3196/paper3.pdf)。在*第七届Web数据自然语言接口（NLIWoD）与第十九届欧洲语义Web会议（ESWC 2022）联合会议论文集*中，CEUR Workshop Proceedings第3196卷，第25–31页，希腊赫尔索尼斯索斯。CEUR-WS.org。

+   Scao et al. (2022) Teven Le Scao, Angela Fan, Christopher Akiki, Ellie Pavlick, Suzana Ilić, Daniel Hesslow, Roman Castagné, Alexandra Sasha Luccioni, François Yvon, Matthias Gallé, et al. 2022. Bloom: 一个1760亿参数的开放获取多语言语言模型。*arXiv 预印本 arXiv:2211.05100*。

+   Sun et al. (2023a) Jiashuo Sun, Chengjin Xu, Lumingyuan Tang, Saizhuo Wang, Chen Lin, Yeyun Gong, Lionel M. Ni, Heung-Yeung Shum, and Jian Guo. 2023a. [Think-on-graph: 基于知识图谱的大型语言模型的深度与责任推理](http://arxiv.org/abs/2307.07697)。

+   Sun et al. (2023b) Jiashuo Sun, Chengjin Xu, Lumingyuan Tang, Saizhuo Wang, Chen Lin, Yeyun Gong, Heung-Yeung Shum, and Jian Guo. 2023b. Think-on-graph: 基于知识图谱的大型语言模型的深度与责任推理。*arXiv 预印本 arXiv:2307.07697*。

+   Sun et al. (2019) Zhiqing Sun, Zhi-Hong Deng, Jian-Yun Nie, and Jian Tang. 2019. [Rotate: 通过复数空间中的关系旋转进行知识图谱嵌入](http://arxiv.org/abs/1902.10197)。

+   Touvron et al. (2023) Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar, et al. 2023. Llama: 开放和高效的基础语言模型。*arXiv 预印本 arXiv:2302.13971*。

+   Wang et al. (2023a) Chaojie Wang, Yishi Xu, Zhong Peng, Chenxi Zhang, Bo Chen, Xinrun Wang, Lei Feng, and Bo An. 2023a. keqing: 基于知识的问答是大型语言模型的自然链式思维导师。*arXiv 预印本 arXiv:2401.00426*。

+   Wang et al. (2023b) Jianing Wang, Qiushi Sun, Nuo Chen, Xiang Li, and Ming Gao. 2023b. 通过知识链提示提升语言模型推理能力。*arXiv 预印本 arXiv:2306.06427*。

+   Wang et al. (2023c) Xuezhi Wang, Jason Wei, Dale Schuurmans, Quoc Le, Ed Chi, Sharan Narang, Aakanksha Chowdhery, and Denny Zhou. 2023c. [自洽性提升语言模型的链式思维推理](http://arxiv.org/abs/2203.11171)。

+   Wu et al. (2023) Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang, and Chi Wang. 2023. Autogen: 通过多智能体对话框架推动下一代大型语言模型应用。*arXiv 预印本 arXiv:2308.08155*。

+   Xu et al. (2024) Yao Xu, Shizhu He, Jiabei Chen, Zihao Wang, Yangqiu Song, Hanghang Tong, Kang Liu, and Jun Zhao. 2024. Generate-on-graph: 将大型语言模型视为智能体和知识图谱，用于不完整知识图谱问答。*arXiv 预印本 arXiv:2404.14741*。

+   姚等（2022）姚顺宇、赵杰富、余滇、杜楠、伊扎克·沙夫兰、卡尔提克·纳拉辛汉和袁超。2022年。《React：在语言模型中协同推理与行动》。*arXiv 预印本 arXiv:2210.03629*。

+   于等（2022）于文浩、朱晨光、张智涵、王硕航、张卓生、方雨薇和蒋萌。2022年。[检索增强的常识推理：一种统一的方法](https://doi.org/10.18653/v1/2022.emnlp-main.294)。在*2022年自然语言处理实证方法会议论文集*，第4364-4377页，阿布扎比，阿联酋。计算语言学协会。

+   张等（2023）张艺驰、陈卓、张文和陈华俊。2023年。《使大型语言模型在知识图谱补全中表现更好》。*arXiv 预印本 arXiv:2310.06671*。

## 附录A 案例研究

为了展示ODA的有效性，我们从QALD10-en数据集中挑选了几个典型案例。为了提高可读性，我们在输出中将实体ID替换为其对应的名称。我们选择了两个例子，说明ODA如何利用观察来正确回答表[5](https://arxiv.org/html/2404.07677v2#A1.T5 "表5 ‣ 附录A 案例研究 ‣ ODA：面向大语言模型与知识图谱整合的观察驱动代理")和表[6](https://arxiv.org/html/2404.07677v2#A1.T6 "表6 ‣ 附录A 案例研究 ‣ ODA：面向大语言模型与知识图谱整合的观察驱动代理")中的问题。

| 问题 | 这个问题是：东京都的首府是什么？ |
| --- | --- |
| 动作 | 思考: 这个问题是在询问东京都的首府。候选实体ID 'Q1490' 对应于东京。我从观察中看到有一个三元组（东京，首府，新宿），这可能是答案。然而，为了确认这一点，我将对 'Q1490' 执行 GetNeighbor 动作，以获取所有以东京为主体的三元组。 |
|  | 动作: GetNeighbor |
|  | 实体ID: 东京 |
| 反思 | 思考: 这个问题是在询问东京都的首府。从观察中可以看出，东京是日本的首府，并且是日本的一个行政区。东京的首府是新宿。因此，我们应该选择包含这些信息的三元组。 |
|  | 三元组: (东京，实例，日本的行政区)，(东京，首府，新宿)，(东京，首府，日本) |
| 动作 | 思考: 这个问题是在询问东京都的首府。从参考记忆中可以看出，东京都的首府是新宿。因此，问题的答案是新宿。 |
|  | 动作: 答案 |
| 答案 | 新宿 |

表5: 案例1

| 问题 | 《荒野的呼唤》和《白牙》这两部作品，杰克·伦敦的最著名的作品，都发生在什么地方？ |
| --- | --- |
| 动作 | 思考：问题询问的是杰克·伦敦的两部作品《野性的呼唤》和《白牙》的背景设定。观察结果提供了一些关于这两部作品叙事地点的信息，但为了确认并给出具体答案，我将对《野性的呼唤》使用获取邻居（GetNeighbor）功能。 |
|  | 动作：获取邻居 |
|  | 实体ID：《野性的呼唤》 |
| 反思 | 思考：首先，我将关注与两部作品《野性的呼唤》和《白牙》地点相关的三元组，因为问题询问的是这两部小说的背景设定。我还将考虑与作者杰克·伦敦相关的三元组，因为问题提到了他。我将选择与这两部小说背景最直接相关的三元组。 |
|  | 三元组：（《野性的呼唤》，叙事地点，育空），（《野性的呼唤》，叙事地点，加拿大），（《野性的呼唤》，作者，杰克·伦敦） |
| 动作 | 思考：问题询问的是《野性的呼唤》和《白牙》两部杰克·伦敦的著名作品的背景设定。从观察中可以看出，《野性的呼唤》有美国、阿拉斯加、加拿大和育空等叙事地点。然而，我们并没有《白牙》的叙事地点。因此，我将对《白牙》的实体ID使用获取邻居功能，以找出其叙事地点。 |
|  | 动作：获取邻居 |
|  | 实体ID：《白牙》 |
| 反思 | 思考：首先，我将寻找能够直接回答问题的三元组。问题询问的是杰克·伦敦的两部作品《野性的呼唤》和《白牙》的背景设定。因此，我将寻找包含这两部作品及指示背景或地点的关系的三元组。从候选三元组中，我可以看到有两个三元组指示《白牙》的叙事地点：（《白牙》，叙事地点，加拿大），（《白牙》，叙事地点，育空）。 |
|  | 三元组：（《白牙》，叙事地点，育空），（《白牙》，叙事地点，加拿大） |
| 动作 | 思考：问题询问的是《野性的呼唤》和《白牙》两部作品的背景地点。从参考记忆中可以看到，《野性的呼唤》的设定地点为美国、阿拉斯加、加拿大和育空，《白牙》的设定地点为加拿大和育空。这两部作品的共同地点是加拿大和育空。 |
|  | 动作：回答 |
| 答案 | [加拿大, 育空] |

表 6：案例 2

## 附录 B 提示

本节展示了行动和反思模块的提示，见表格[7](https://arxiv.org/html/2404.07677v2#A2.T7 "表 7 ‣ 附录 B 提示 ‣ ODA：基于观察的代理，用于整合大语言模型和知识图谱")和[8](https://arxiv.org/html/2404.07677v2#A2.T8 "表 8 ‣ 附录 B 提示 ‣ ODA：基于观察的代理，用于整合大语言模型和知识图谱")。

| 动作 | 提示 |
| --- | --- |
| 邻域探索与路径发现 | 代理指令： |
| 你作为一个基于知识图谱提供答案的代理。 |
| 为了帮助你查询知识库，使用以下工具： |
|  | GetNeighbor(entityID: str) -> List[Tuple[str, str, str]]: |
|  | 描述：返回包含给定实体ID作为头实体，其对应实体ID作为尾实体的三元组。 |
|  | GetPath(entityID1: str, entityID2: str) -> List[List[Tuple[str, str, str]]]: |
|  | 描述：返回连接两个给定实体ID的所有三元组。 |
|  | 示例用法：GetPath("Q30", "Q25231") 返回所有连接 ’Q30’ 和 ’Q25231’ 的三元组。 |
|  | 提供给你的数据： |
|  | 问题：[问题] |
|  | 记忆：[记忆] |
|  | 候选实体ID：[与任务相关的实体ID]（根据操作选择1或2） |
|  | 观察：[观察]（这些作为参考，帮助你从候选实体ID中选择合适的实体ID） |
|  | 标签：[与任务相关的实体标签] |
|  | 操作历史：[历史操作]（避免这些操作） |
|  | 指南： |
|  | 每次只能选择一个操作。 |
|  | 对于GetPath，选择两个实体ID。对于GetNeighbor，选择一个实体ID。如果可用的实体ID少于2个，则只选择GetNeighbor操作。 |
| 答案 | 你是一个基于参考记忆和知识回答问题的代理。 |
|  | 这里是参考记忆：[记忆]。你可以使用它帮助你回答问题。 |
|  | 这是你被要求回答的问题：[问题]。 |
|  | 确保你的答案包含一个答案或一组答案，每个答案应该只有一个或几个单词、一个短语、一个数字、true或false，或一个日期，答案中不得包含其他信息或描述。 |

表格 7：操作提示描述

| 字段 | 提示 |
| --- | --- |
| 反思 | 你是一个基于知识图谱提供答案的代理。 |
|  | 你根据问题：[问题]，从上一步操作中查询到了一些候选三元组[三元组]，以及它们对应的标签：[实体标签]。 |
|  | 现在你被要求选择相关的三元组，以便将来通过它们回答问题。 |
|  | 这里是观察信息：[观察]，帮助你从候选三元组中选择正确的三元组。 |
|  | 此外，这里是记忆：[记忆]。你可以使用它帮助你从候选三元组中选择正确的三元组。 |
|  | 指南： |
|  | 你可以从候选三元组中选择少于15个三元组。 |
|  | 你的输出三元组必须采用实体ID、关系ID、实体ID的格式。 |

表格 8：反思提示描述
