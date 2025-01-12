<!--yml

类别：未分类

日期：2025-01-11 11:46:00

-->

# 探索利用工具增强LLM智能体进行多模态整合，以实现精确的因果发现

> 来源：[https://arxiv.org/html/2412.13667/](https://arxiv.org/html/2412.13667/)

ChengAo Shen¹, Zhengzhang Chen², Dongsheng Luo³, Dongkuan Xu⁴,

Haifeng Chen², Jingchao Ni¹

¹休斯敦大学, ²NEC美国实验室,

³佛罗里达国际大学, ⁴北卡罗来纳州立大学

¹{cshen9, jni7}@uh.edu, ²{zchen, haifeng}@nec-labs.com,

³dluo@fiu.edu, ⁴dxu27@ncsu.edu

###### 摘要

因果推断是各领域决策的必要基础，例如智能健康、药物发现的人工智能（AI）和AIOps。传统的统计因果发现方法虽然已经非常成熟，但主要依赖于观察性数据，往往忽视了因果关系中固有的语义线索。大规模语言模型（LLMs）的出现为利用语义线索进行知识驱动的因果发现提供了一种经济可行的方式，但LLMs在因果发现方面的发展滞后于其他领域，尤其是在多模态数据的探索方面。为了弥合这一差距，我们介绍了MatMcd，一个由工具增强LLMs驱动的多智能体系统。MatMcd有两个关键智能体：一个数据增强智能体，用于检索和处理增强模态数据，以及一个因果约束智能体，用于整合多模态数据以进行知识驱动的推理。内部机制的精细设计确保了智能体之间的成功合作。我们在七个数据集上的实证研究表明，多模态增强的因果发现具有巨大的潜力。

探索利用工具增强LLM智能体进行多模态整合

为精确的因果发现

ChengAo Shen¹, Zhengzhang Chen², Dongsheng Luo³, Dongkuan Xu⁴, Haifeng Chen², Jingchao Ni¹ ¹休斯敦大学, ²NEC美国实验室, ³佛罗里达国际大学, ⁴北卡罗来纳州立大学 ¹{cshen9, jni7}@uh.edu, ²{zchen, haifeng}@nec-labs.com, ³dluo@fiu.edu, ⁴dxu27@ncsu.edu

## 1 引言

在复杂系统中识别因果关系对于多种应用至关重要，包括神经痛诊断（医学领域，Tu等人，([2019](https://arxiv.org/html/2412.13667v1#bib.bib40))）、蛋白质通路分析（计算生物学领域，Sachs等人，([2005](https://arxiv.org/html/2412.13667v1#bib.bib28))）以及微服务架构中的根因定位（Wang等人，([2023a](https://arxiv.org/html/2412.13667v1#bib.bib43))）。这些因果洞察对新兴领域如智能健康、AI驱动的药物发现和AIOps等具有重要意义。发现这些关系的过程，称为因果发现，通常会生成一个有向无环图（DAG）。在这个图中，边代表变量之间因果关系的存在和方向，如图[1](https://arxiv.org/html/2412.13667v1#S1.F1 "图 1 ‣ 1 引言 ‣ 通过工具增强的LLM代理探索多模态集成进行精确因果发现")（a）所示。这个DAG不仅管理数据生成过程，还增强了对变量间相互影响的理解，为许多下游决策任务奠定了基础（Nguyen等人，[2023](https://arxiv.org/html/2412.13667v1#bib.bib23)）。因此，构建准确的因果图对于后续分析的可靠性至关重要。

![参见说明文字](img/11fbb01dc48cd51856eb81c9effd76c9.png)

图1：基于LLM的因果发现一般过程的示意图：（a）通过SCD算法估计因果图；（b）根据图生成提示；（c）LLM推理因果结构；（d）基于LLM的响应细化图。

传统方法主要依赖数据驱动的统计因果发现（SCD），可以分为非参数方法（如Spirtes和Glymour，[1991](https://arxiv.org/html/2412.13667v1#bib.bib37)；Silander和Myllymäki，[2006](https://arxiv.org/html/2412.13667v1#bib.bib35)；Huang等人，[2018](https://arxiv.org/html/2412.13667v1#bib.bib8)；Xie等人，[2020](https://arxiv.org/html/2412.13667v1#bib.bib47)）和半参数方法（如Shimizu等人，[2006](https://arxiv.org/html/2412.13667v1#bib.bib31)，[2011](https://arxiv.org/html/2412.13667v1#bib.bib32)；Zheng等人，[2018](https://arxiv.org/html/2412.13667v1#bib.bib55)；Tu等人，[2022](https://arxiv.org/html/2412.13667v1#bib.bib41)）。这些方法通过分析变量的观察数据来估计因果关系，但它们忽略了变量的语义和上下文线索，导致了次优的结果（Takayama等人，[2024](https://arxiv.org/html/2412.13667v1#bib.bib38)）。

常识和领域知识对于识别语义上有意义的变量之间的因果关系至关重要。鉴于此，越来越多的研究开始关注如何提取这些信息以支持因果发现。大型语言模型（LLMs），因其依托大规模训练所获得的广泛知识所展现出的惊人推理能力而备受赞誉（Brown 等，[2020](https://arxiv.org/html/2412.13667v1#bib.bib3)；Achiam 等，[2023](https://arxiv.org/html/2412.13667v1#bib.bib1)），如今成为了一种有前景且具有成本效益的专家知识来源，能够帮助因果发现。例如，仅通过提供变量名称和一些上下文提示，LLMs 已被证明能够推断出有意义的因果关系（Ban 等，[2023](https://arxiv.org/html/2412.13667v1#bib.bib2)；Jiralerspong 等，[2024](https://arxiv.org/html/2412.13667v1#bib.bib9)）。最近，混合方法被引入，将 LLMs 与数据驱动的 SCD 算法结合，从而在因果发现中实现了更高的准确性（Takayama 等，[2024](https://arxiv.org/html/2412.13667v1#bib.bib38)；Khatibi 等，[2024](https://arxiv.org/html/2412.13667v1#bib.bib11)）。然而，大多数现有方法尚未充分利用现代 LLMs 的潜力，尤其是基于工具增强型 LLMs 构建的代理系统。

一个 LLM 代理通常配备有记忆、推理、规划和访问外部工具（如计算器、搜索引擎和代码编译器）的能力，这使得它在解决问题时优于普通的 LLMs（Yao 等，[2022](https://arxiv.org/html/2412.13667v1#bib.bib50)）。由于单一代理系统即便进行自我反思也可能出现幻觉（Li 等，[2023](https://arxiv.org/html/2412.13667v1#bib.bib17)；Shinn 等，[2024](https://arxiv.org/html/2412.13667v1#bib.bib33)），因此引入了多代理系统，通过结合多个较弱的代理，能够与更先进的模型相抗衡。尽管进展迅速，但用于因果发现的 LLM 代理仍然未得到充分探索。迄今为止，最相关的工作是引入了一种多代理系统，通过辩论的 LLMs 进行协同因果推理（Le 等，[2024](https://arxiv.org/html/2412.13667v1#bib.bib15)）。然而，这种方法从未探索过数据中多模态的潜力——这是代理系统非常擅长处理的特性。

如图[1](https://arxiv.org/html/2412.13667v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")所示，因果发现的混合方法通常通过一些SCD算法生成的先验因果图来提示LLMs，并附加一些元数据（例如变量名、数据集标题）作为上下文。然而，这些输入可能不足以完全激活LLMs的推理能力。受外部来源（如网页和日志）中丰富语义数据可以作为因果图的额外模态来改进提示的观察启发，我们提出了一个多代理系统，结合工具增强的LLMs来探索因果发现的多模态增强（MatMcd）。

具体而言，MatMcd被设计为一个框架，用于精炼由SCD算法生成的因果图，涉及两个关键代理：(1) 数据增强代理（DA-agent，$\S$[3.2](https://arxiv.org/html/2412.13667v1#S3.SS2 "3.2 Data Augmentation Agent (DA-agent) ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")），和(2) 因果约束代理（CC-agent，$\S$[3.3](https://arxiv.org/html/2412.13667v1#S3.SS3 "3.3 Causal Constraint Agent (CC-agent) ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")）。给定一个由某个SCD算法输出的因果图，DA-agent整合元数据（即变量名和数据集标题），并调用如网页搜索API或离线日志查询API等工具，进行迭代反思式的数据增强（文本）检索，将其总结为与图形不同模态的简洁提示。接收到增强数据后，CC-agent将其与先验因果图的拓扑结构结合，以推断变量之间的因果关系。两个代理都由多个合作的LLMs组成，具有精细的机制来处理子任务，如工具调用、记忆、推理和总结。在需要高效记忆的地方，我们使用了检索增强生成（RAG）方法，Lewis等人（[2020](https://arxiv.org/html/2412.13667v1#bib.bib16)）的组件。在我们的实验中，我们将MatMcd与五个基准数据集和两个公共AIOps微服务系统数据集上的最先进（SOTA）基准方法进行了比较。结果表明，通过融合多模态数据，因果发现有了显著的改善。本文的主要贡献总结如下：

+   •

    我们提议通过LLM代理探索多模态增强的因果发现问题，这是一个重要但研究较少的课题。

+   •

    我们介绍了MatMcd，一个新的多代理框架，作为评估多模态在因果发现中有效性的试验平台。

+   •

    我们在多个数据集上进行了广泛的实验，其中MatMcd将因果推断误差（NHD）减少了最多66.7%，并将根本原因定位（MAP@10）提高了最多83.3%，相较于最好的基准方法，表明多模态数据在因果发现中的潜力。

## 2 相关工作

因果发现方法主要是传统的数据驱动SCD算法，包括非参数方法Spirtes和Glymour（[1991](https://arxiv.org/html/2412.13667v1#bib.bib37)）；Chickering（[2002](https://arxiv.org/html/2412.13667v1#bib.bib6)）；Silander和Myllymäki（[2006](https://arxiv.org/html/2412.13667v1#bib.bib35)）；Huang等（[2018](https://arxiv.org/html/2412.13667v1#bib.bib8)）；Xie等（[2020](https://arxiv.org/html/2412.13667v1#bib.bib47)）和半参数方法Shimizu等（[2006](https://arxiv.org/html/2412.13667v1#bib.bib31)）；Hoyer等（[2008](https://arxiv.org/html/2412.13667v1#bib.bib7)）；Shimizu等（[2011](https://arxiv.org/html/2412.13667v1#bib.bib32)）；Zheng等（[2018](https://arxiv.org/html/2412.13667v1#bib.bib55)）；Rolland等（[2022](https://arxiv.org/html/2412.13667v1#bib.bib27)）；Tu等（[2022](https://arxiv.org/html/2412.13667v1#bib.bib41)）。这些方法依赖于观察数据作为输入，但无法利用变量的语义。最近，知识驱动的方法在因果发现中被发现具有潜力。一些早期的工作通过简单地提示变量名和数据集标题，使用LLMs进行因果发现，例如Kıcıman等（[2023](https://arxiv.org/html/2412.13667v1#bib.bib12)）；Zečević等（[2023](https://arxiv.org/html/2412.13667v1#bib.bib52)）；Chen等（[2024a](https://arxiv.org/html/2412.13667v1#bib.bib4)）；Jiralerspong等（[2024](https://arxiv.org/html/2412.13667v1#bib.bib9)）。随后，结合SCD算法和LLMs的混合方法被提出，例如Ban等（[2023](https://arxiv.org/html/2412.13667v1#bib.bib2)）；Vashishtha等（[2023](https://arxiv.org/html/2412.13667v1#bib.bib42)）；Khatibi等（[2024](https://arxiv.org/html/2412.13667v1#bib.bib11)）；Takayama等（[2024](https://arxiv.org/html/2412.13667v1#bib.bib38)），并被发现比纯粹基于LLM的方法更为有效。最近，提出了一种基于多智能体系统的方法Le等（[2024](https://arxiv.org/html/2412.13667v1#bib.bib15)），用于探索辩论型LLMs的影响。然而，这些方法都没有探索多模态数据在基于LLM的因果发现过程中的潜力。

LLM 代理通常配备计划、记忆和工具。规划可以使用诸如 CoT Wei 等人（[2022](https://arxiv.org/html/2412.13667v1#bib.bib46)）、ReAct Yao 等人（[2022](https://arxiv.org/html/2412.13667v1#bib.bib50)）和 Reflexion Shinn 等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib33)）等技术。工具赋予代理与环境互动的能力。MRKL Karpas 等人（[2022](https://arxiv.org/html/2412.13667v1#bib.bib10)）、Toolformer Schick 等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib29)）、函数调用 OpenAI（[2024](https://arxiv.org/html/2412.13667v1#bib.bib24)）和 HuggingGPT Shen 等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib30)）展示了整合工具以解决问题的范式。对于复杂任务，多代理系统是一个有前途的方向。多代理系统的主要类别包括合作型代理 Qian 等人（[2023](https://arxiv.org/html/2412.13667v1#bib.bib25)）；Chen 等人（[2024b](https://arxiv.org/html/2412.13667v1#bib.bib5)），竞争型代理 Zhao 等人（[2023](https://arxiv.org/html/2412.13667v1#bib.bib53)）和辩论型代理 Li 等人（[2023](https://arxiv.org/html/2412.13667v1#bib.bib17)）；Liang 等人（[2023](https://arxiv.org/html/2412.13667v1#bib.bib18)）；Xiong 等人（[2023](https://arxiv.org/html/2412.13667v1#bib.bib48)）。在这项工作中，我们研究了一种合作型多代理系统，其中代理被协调以朝着共同目标增强解决方案。

## 3 方法论

![参见说明文字](img/327e69ee5a8030fe2227d1811772bf4e.png)

图 2：MatMcd 框架示意图：（a）框架概览，（b）DA-代理的内部工作原理，和（c）CC-代理的内部工作原理。

图[2](https://arxiv.org/html/2412.13667v1#S3.F2 "Figure 2 ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")展示了所提出的MatMcd系统的概述，该系统有四个关键组件：（1）因果图估计器（$\S$[3.1](https://arxiv.org/html/2412.13667v1#S3.SS1 "3.1 Causal Graph Estimator ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")）；（2）数据增强代理（DA-agent，$\S$[3.2](https://arxiv.org/html/2412.13667v1#S3.SS2 "3.2 Data Augmentation Agent (DA-agent) ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")）；（3）因果约束代理（CC-agent，$\S$[3.3](https://arxiv.org/html/2412.13667v1#S3.SS3 "3.3 Causal Constraint Agent (CC-agent) ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")）；（4）因果图细化器（$\S$[3.4](https://arxiv.org/html/2412.13667v1#S3.SS4 "3.4 Causal Graph Refiner ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")）。接下来，我们将首先介绍一些符号，并按照数据流的顺序详细说明每个组件。

符号说明。假设有一组包含$n$个变量的集合$\mathcal{V}=\{v_{1}$, …, $v_{n}\}$（例如，图[1](https://arxiv.org/html/2412.13667v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")(a)中的4个变量），每个变量$v_{i}$都与一组观测数据样本${\bf v}_{i}=\{v_{i1},...,v_{im}\}$相关联，其中$m$为样本数量。这些观测数据可以是随机样本，也可以是由每个变量发出的长度为$m$的时间序列，具体取决于应用场景。在许多情况下，元数据是可用的。在本研究中，我们假设有一组最小的元数据$\mathcal{D}=\{\mathsf{s},\mathcal{Z}\}$，其中$\mathsf{s}$是数据集的描述性标题（例如，图[1](https://arxiv.org/html/2412.13667v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")(a)中的“AutoMPG”），$\mathcal{Z}=\{\mathsf{z}_{1},...,\mathsf{z}_{n}\}$包含每个变量$v_{i}$在$\mathcal{V}$中的描述性名称$\mathsf{z}_{i}$（例如，图[1](https://arxiv.org/html/2412.13667v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")(a)中的“加速度”）。

任务。基于$\{{\bf v}_{1},...,{\bf v}_{n}\}$和$\mathcal{D}$，我们希望构建一个DAG，${\bf G}=(\mathcal{V},\mathcal{E})$，其中$\mathcal{V}$中的每个变量$v_{i}$都是图中的一个节点，$\mathcal{E}\subseteq\mathcal{V}\times\mathcal{V}$是有向边的集合，且$(v_{i},v_{j})\in\mathcal{E}$表示从$v_{i}$到$v_{j}$的因果关系。本文的目标是推断出$\mathcal{V}$中$n$个变量之间的准确因果关系。

### 3.1 因果图估计器

类似于高山等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib38)）和Khatibi等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib11)）的混合方法，因果图估计器作为因果图的初始化器，基于数据驱动的SCD算法构建，目的是仅通过观察数据$\{{\bf v}_{1}$, …, ${\bf v}_{n}\}$估计初始因果图${\bf G}_{0}=(\mathcal{V},\mathcal{E}_{0})$，而无需访问任何其他信息。这里$\mathcal{V}$没有下标，因为它将在整个框架中保持不变，我们关注的是$\mathcal{E}$中因果关系的变化，以实现准确的因果发现。

我们的框架对SCD算法的选择具有灵活性。在本研究中，我们探讨了使用三种广泛应用的算法的可行性，每种算法都代表了一个类别：(1) 基于约束的方法——Peter-Clark（PC）算法，Spirtes和Glymour（[1991](https://arxiv.org/html/2412.13667v1#bib.bib37)），这是非参数的；(2) 基于评分的方法——精确搜索（ES）算法，Yuan和Malone（[2013](https://arxiv.org/html/2412.13667v1#bib.bib51)），这是非参数的；(3) 约束的功能因果模型——DirectLiNGAM，Shimizu等人（[2011](https://arxiv.org/html/2412.13667v1#bib.bib32)），这是半参数的。我们将在未来的工作中探索其他SCD算法，因为这超出了本文的范围。受最近用于图形的LLM提示技术启发，Wang等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib45)），SCD算法的输出边$\mathcal{E}_{0}$将作为邻接列表嵌入到由CC-agent的提示生成模块（$\S$[3.3](https://arxiv.org/html/2412.13667v1#S3.SS3 "3.3 因果约束代理（CC-agent） ‣ 3 方法论 ‣ 探索与工具增强LLM代理的多模态集成，以实现精确的因果发现")）生成的提示中（图[2](https://arxiv.org/html/2412.13667v1#S3.F2 "图2 ‣ 3 方法论 ‣ 探索与工具增强LLM代理的多模态集成，以实现精确的因果发现")），该提示还将集成由DA-agent（$\S$[3.2](https://arxiv.org/html/2412.13667v1#S3.SS2 "3.2 数据增强代理（DA-agent） ‣ 3 方法论 ‣ 探索与工具增强LLM代理的多模态集成，以实现精确的因果发现")）检索到的语义丰富的数据模态。

### 3.2 数据增强代理（DA-agent）

DA-agent的目标是检索与初始因果图${\bf G}_{0}$相关的语义丰富的上下文数据，如有关变量的网页文档和日志文件，作为提示CC-agent的附加模态（见$\S$[3.3](https://arxiv.org/html/2412.13667v1#S3.SS3 "3.3 Causal Constraint Agent (CC-agent) ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")）。如图[2](https://arxiv.org/html/2412.13667v1#S3.F2 "Figure 2 ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")(b)所示，DA-agent包括一个搜索LLM和一个总结LLM。

搜索LLM。搜索LLM可以访问一组用于数据搜索的工具。在本研究中，我们重点关注一个网页搜索API，作为从外部源检索上下文数据的通用工具，以及一个日志查找API，适用于具有特定领域数据库的应用，例如AIOps Zheng等人（[2024a](https://arxiv.org/html/2412.13667v1#bib.bib54)）中微服务系统根本原因分析的过程日志。DA-agent对工具包具有灵活性，并且通过包含其他特定应用工具（如Wikipedia API和代码查找API）可以扩展到更广泛的场景，这部分我们留待未来的探索。

如图[2](https://arxiv.org/html/2412.13667v1#S3.F2 "Figure 2 ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")所示，在接收到关于因果图的元数据$\mathcal{D}=\{\mathsf{s},\mathcal{Z}\}$后，搜索LLM首先检查其调用历史内存，以决定是否启动新的工具调用。如果需要新的调用，搜索LLM会调用搜索工具API，使用包含数据集标题$\mathsf{s}$和变量名称$\mathsf{z}_{1}$，…，$\mathsf{z}_{n}$的提示来检索额外的数据。然后，这个搜索动作会被记录在内存中以供将来参考。在我们的案例中，由于重点是搜索工具，因此该动作涉及生成查询，生成的查询会被添加到内存中。在后续的迭代中，会检查所有先前记录的查询，以防止冗余查询。这个过程会不断迭代，直到搜索LLM判断不再需要进一步的工具调用，从而终止循环。

为了启用这种迭代搜索，提示语基于 Shinn 等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib33)）和 Madaan 等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib21)）的自我反思技术进行设计，其中 LLM 根据历史查询的全面性评估是否需要额外的查询。循环在 LLM 得出“不需要查询”的结论时终止。与单轮搜索相比，这一迭代过程对于检索相关且全面的数据至关重要，尤其是在变量特定信息难以获取的领域（例如医学）中。然而，对于查找 API，迭代是不必要的。因此，通过这些工具检索到的数据构成了因果图的一个额外文本模态。用于搜索 LLM 的提示语见附录[C.1](https://arxiv.org/html/2412.13667v1#A3.SS1 "C.1 数据增强代理（DA-agent）提示语模板 ‣ 附录 C 提示语模板 ‣ 探索与工具增强 LLM 代理的多模态集成以实现精确因果发现")。

![参见标题](img/081ef817d39c2a22d6dfd67b92d51e29.png)

图 3：DA-agent 中的搜索工具准备，包括 (a) 网络搜索工具，以及 (b) 日志查找工具。

工具准备。图[3](https://arxiv.org/html/2412.13667v1#S3.F3 "图 3 ‣ 3.2 数据增强代理（DA-agent） ‣ 3 方法学 ‣ 探索与工具增强 LLM 代理的多模态集成以实现精确因果发现")展示了我们对网络搜索工具和日志查找工具的准备。在前者中，我们使用 Google 搜索 API，查询由上述的搜索 LLM 生成。检索到的网页将通过数据格式化工具进行去格式化（例如，去除 HTML 标签），并将生成的纯文档存储在内存中。然后，使用 Web-Summary LLM 对文档进行总结，提炼成简洁的描述。相反，日志查找工具使用精确查找，即使用变量名作为关键字，直接检索相应的日志。因此，可以删除内存，检索到的日志仍然需要去格式化（例如，去除日志模板），并且可能较长，将通过 Log-Summary LLM 进行总结。

Summary LLM。通过搜索LLM检索到的数据会被迭代地添加到检索数据内存中，如图[2](https://arxiv.org/html/2412.13667v1#S3.F2 "Figure 2 ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")(b)所示。循环终止后，Summary LLM将检索到的数据总结为三种类型的提示：(1) 数据集的描述；(2) 图中每个变量的描述；(3) 变量之间的关系。由于从迭代搜索中检索到的数据量可能超过LLM的上下文窗口，我们采用了一种高效的总结方法，使用了RAG Lewis等人提出的技术（[2020](https://arxiv.org/html/2412.13667v1#bib.bib16)）。检索到的数据被分割成带索引的文档块，使用LlamaIndex Liu（[2022](https://arxiv.org/html/2412.13667v1#bib.bib19)）实现，通过text-embedding-ada-002进行块索引，并使用最大内积搜索（MIPS）来检索相关块。附录[D.1](https://arxiv.org/html/2412.13667v1#A4.SS1 "D.1 Data Augmentation Agent (DA-agent) ‣ Appendix D Examples of LLM Response ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")中提供了一个示例总结。最终的总结作为初始因果图${\bf G}_{0}$的上下文数据模态，用于提示CC-agent。

备注。由于存在检索到的网页内容可能泄露某些数据集的真实因果图的风险，我们在RAG实现的检索数据内存中进行了彻底筛查，以防止此类信息泄漏。

### 3.3 因果约束代理（CC-agent）

除了外部知识外，我们的方法还利用LLM中存储的事实知识，这些知识是在预训练过程中获得的。为了实现这一点，CC-agent的设计基于零-shot Chain-of-Thought (ZSCOT)框架的Two-Stage Prompting方法，Kojima等人（[2022](https://arxiv.org/html/2412.13667v1#bib.bib13)）。首先，一个提示构建器将${\bf G}_{0}$（表示为邻接列表）与来自DA-agent的上下文数据结合，提示一个知识LLM。该知识LLM的任务是根据上下文数据和其自身的知识解释初始因果图${\bf G}_{0}$中的每个（非）存在的因果关系。这些解释可能支持或反驳因果关系，并用于在第二阶段提示一个约束LLM，得出每个关系的存在结论（即“是”/“否”）。为了应对结论中的潜在不确定性，我们采用了Top-K-Guess技术，Tian等人（[2023](https://arxiv.org/html/2412.13667v1#bib.bib39)）用来引导语言信心。这个方法被发现比基于采样的似然估计更可靠，Xiong等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib49)）定量评估每个因果关系的可能性。在Top-K猜测中，最有信心的一个将被选为每个因果关系的最终结论。

### 3.4 因果图优化器

为确保最终的因果图${\bf G}$是无环的，边集$\mathcal{E}_{0}$不会直接根据来自CC-agent的（非）存在约束进行修改。相反，在施加这些约束后，Causal Graph Estimator中使用的SCD算法($\S$[3.1](https://arxiv.org/html/2412.13667v1#S3.SS1 "3.1 Causal Graph Estimator ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery"))将重新运行，以生成新的因果图${\bf G}$。具体来说，在接收到（非）存在约束后，将构建一个约束矩阵${\bf C}\in\mathbb{R}^{n\times n}$，其中，如果CC-agent指示从$v_{i}$到$v_{j}$有因果效应，则${\bf C}_{ij}=1$，否则${\bf C}_{ij}=0$。

最具代表性的SCD算法，包括PC、ES和DirectLiNGAM，设计时会将约束矩阵${\bf C}$与观察数据$\{{\bf v}_{1},\dots,{\bf v}_{n}\}$一起作为输入。这确保了生成的因果图在不同程度上符合${\bf C}$中的约束，从而能够生成一个既符合施加的约束又是有向无环图（DAG）的优化因果图${\bf G}$。

备注：与现有利用LLMs进行因果发现的方法（Takayama 等人 ([2024](https://arxiv.org/html/2412.13667v1#bib.bib38)); Khatibi 等人 ([2024](https://arxiv.org/html/2412.13667v1#bib.bib11))）相比，提出的MatMcd的关键创新在于其探索了DA-agent以增强因果发现的多模态方法。关于该工作流程的算法总结见附录[A](https://arxiv.org/html/2412.13667v1#A1 "附录 A 算法 ‣ 探索多模态整合与工具增强的LLM代理在精确因果发现中的应用")。

| 方法 | AutoMPG | DWDClimate | SachsProtein |
| --- | --- | --- | --- |
| Prc [↑] | F1 [↑] | FPR [↓] | SHD [↓] | NHD [↓] | Prc [↑] | F1 [↑] | FPR [↓] | SHD [↓] | NHD [↓] | Prc [↑] | F1 [↑] | FPR [↓] | SHD [↓] | NHD [↓] |
| PC | 0.11 | 0.14 | 0.40 | 8 | 0.32 | 0.14 | 0.15 | 0.20 | 9 | 0.25 | 0.38 | 0.44 | 0.15 | 24 | 0.19 |
| 精确搜索 | 0.25 | 0.30 | 0.30 | 6 | 0.24 | 0.45 | 0.58 | 0.20 | 6 | 0.16 | 0.18 | 0.23 | 0.26 | 31 | 0.25 |
| DirectLiNGAM | 0.11 | 0.14 | 0.40 | 8 | 0.32 | 0.16 | 0.22 | 0.33 | 10 | 0.27 | 0.27 | 0.36 | 0.25 | 29 | 0.23 |
| MAC* | - | - | - | 4 | 0.16 | - | - | - | 6 | 0.19 | - | - | - | 21 | 0.19 |
| Efficient-CDLMs | 0.66 | 0.50 | 0.05 | 4 | 0.16 | 0.33 | 0.33 | 0.13 | 8 | 0.22 | 0.33 | 0.09 | 0.02 | 20 | 0.16 |
| SCD-LLM | 0.57 | 0.66 | 0.15 | 3 | 0.12 | 0.33 | 0.22 | 0.06 | 7 | 0.19 | 0.04 | 0.05 | 0.19 | 29 | 0.23 |
| ReAct | 0.50 | 0.54 | 0.15 | 4 | 0.16 | 0.60 | 0.40 | 0.06 | 6 | 0.16 | 0.04 | 0.04 | 0.20 | 29 | 0.23 |
| LLM-KBCI | 0.57 | 0.66 | 0.15 | 3 | 0.12 | 0.50 | 0.40 | 0.06 | 6 | 0.16 | 0.13 | 0.14 | 0.19 | 27 | 0.22 |
| LLM-KBCI-RA | 0.57 | 0.55 | 0.15 | 3 | 0.12 | 0.75 | 0.60 | 0.03 | 4 | 0.11 | 0.08 | 0.09 | 0.20 | 30 | 0.24 |
| LLM-KBCI-RE | 0.50 | 0.61 | 0.20 | 4 | 0.16 | 0.50 | 0.40 | 0.06 | 6 | 0.16 | 0.09 | 0.10 | 0.18 | 28 | 0.23 |
| MatMcd | 1.00 | 0.88 | 0.00 | 1 | 0.04 | 0.75 | 0.88 | 0.03 | 4 | 0.11 | 0.50 | 0.42 | 0.06 | 17 | 0.14 |
| MatMcd-RE | 0.57 | 0.66 | 0.15 | 3 | 0.12 | 0.50 | 0.40 | 0.06 | 6 | 0.16 | 0.31 | 0.31 | 0.12 | 21 | 0.17 |

表1：不同因果发现方法在含连续变量的数据集上的比较。${\uparrow}$ 表示分数越大越好。${\downarrow}$ 表示分数越小越好。* 表示数字采用自这些方法的论文。

| 方法 | 亚洲 | 儿童 |
| --- | --- | --- |
| Prc [↑] | F1 [↑] | FPR [↓] | SHD [↓] | NHD [↓] | Prc [↑] | F1 [↑] | FPR [↓] | SHD [↓] | NHD [↓] |
| PC | 0.50 | 0.50 | 0.07 | 6 | 0.09 | 0.30 | 0.34 | 0.06 | 24 | 0.06 |
| 精确搜索 | 0.50 | 0.42 | 0.05 | 6 | 0.09 | 0.35 | 0.28 | 0.02 | 19 | 0.04 |
| DirectLiNGAM | 0.28 | 0.36 | 0.17 | 11 | 0.17 | 0.29 | 0.36 | 0.07 | 33 | 0.08 |
| Efficient-CDLMs | 0.57 | 0.53 | 0.05 | 7 | 0.10 | 0.21 | 0.20 | 0.048 | 37 | 0.09 |
| SCD-LLM | 0.60 | 0.40 | 0.03 | 5 | 0.07 | 0.56 | 0.54 | 0.02 | 19 | 0.04 |
| ReAct | 0.40 | 0.30 | 0.05 | 6 | 0.09 | 0.56 | 0.54 | 0.02 | 19 | 0.04 |
| LLM-KBCI | 0.42 | 0.40 | 0.07 | 6 | 0.09 | 0.48 | 0.50 | 0.03 | 20 | 0.05 |
| LLM-KBCI-RA | 0.33 | 0.28 | 0.07 | 7 | 0.10 | 0.44 | 0.46 | 0.04 | 21 | 0.05 |
| LLM-KBCI-RE | 0.28 | 0.26 | 0.08 | 7 | 0.10 | 0.40 | 0.42 | 0.04 | 22 | 0.05 |
| MatMcd | 0.50 | 0.42 | 0.05 | 6 | 0.09 | 0.48 | 0.50 | 0.03 | 20 | 0.05 |
| MatMcd-RE | 0.66 | 0.57 | 0.03 | 4 | 0.06 | 0.56 | 0.54 | 0.02 | 19 | 0.04 |

表2：不同因果发现方法在离散变量数据集上的比较。

## 4 实验

在本节中，我们首先将MatMcd与SOTA方法在基准数据集上的表现进行比较。然后，我们在真实企业微服务系统数据集上，评估MatMcd在根本原因分析任务中的表现。

### 4.1 实验设置

基准数据集。为了全面起见，我们使用了5个基准数据集，涵盖了连续变量和离散变量。对于前者，参考Takayama等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib38)）；Le等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib15)），我们采用了（1）AutoMPG Quinlan（[1993](https://arxiv.org/html/2412.13667v1#bib.bib26)），该数据集包含5个与城市循环燃油消耗（以每加仑多少英里计算）相关的变量，每个变量都有长度为392的时间序列；（2）DWDClimate Mooij等人（[2016](https://arxiv.org/html/2412.13667v1#bib.bib22)），该数据集包含6个与德意志气象局的气象站观测数据相关的变量，每个变量都有长度为350的时间序列；（3）SachsProtein Sachs等人（[2005](https://arxiv.org/html/2412.13667v1#bib.bib28)），该数据集包含11个测量人类细胞中不同蛋白质和磷脂质表达水平的变量，每个变量都有长度为7,466的时间序列。对于后者，参考Long等人（[2023](https://arxiv.org/html/2412.13667v1#bib.bib20)）；Jiralerspong等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib9)），我们采用了（4）Asia Lauritzen和Spiegelhalter（[1988](https://arxiv.org/html/2412.13667v1#bib.bib14)），该数据集包含8个与肺病诊断相关的变量，每个变量有1,000个离散样本；（5）Child Spiegelhalter（[1992](https://arxiv.org/html/2412.13667v1#bib.bib36)），该数据集包含20个与新生儿先天性心脏病相关的变量，每个变量有1,000个离散样本。

在AutoMPG、DWDClimate和SachsProtein中，专家构建的地面真实因果图可用于评估目的。对于Asia和Child，由于变量的观测数据是从贝叶斯网络中采样的，因此变量之间的先验条件概率确定了地面真实因果图。

| 方法 | AutoMPG | DWDClimate | SachsProtein |
| --- | --- | --- | --- |
| Prc [↑] | F1 [↑] | FPR [↓] | SHD [↓] | NHD [↓] | Prc [↑] | F1 [↑] | FPR [↓] | SHD [↓] | NHD [↓] | Prc [↑] | F1 [↑] | FPR [↓] | SHD [↓] | NHD [↓] |
| MatMcd | 1.00 | 0.88 | 0.00 | 1 | 0.04 | 0.75 | 0.88 | 0.03 | 4 | 0.11 | 0.38 | 0.22 | 0.04 | 17 | 0.14 |
| (a) Iter.$\rightarrow$Single search | 0.66 | 0.72 | 0.10 | 2 | 0.08 | 0.66 | 0.44 | 0.03 | 5 | 0.13 | 0.38 | 0.22 | 0.04 | 17 | 0.14 |
| (b) PC$\rightarrow$ES | 0.33 | 0.42 | 0.30 | 6 | 0.24 | 0.50 | 0.50 | 0.10 | 5 | 0.13 | 0.26 | 0.28 | 0.16 | 25 | 0.20 |
| (b) PC$\rightarrow$DirectLiNGAM | 0.16 | 0.18 | 0.25 | 6 | 0.24 | 0.16 | 0.22 | 0.23 | 9 | 0.25 | 0.19 | 0.22 | 0.20 | 27 | 0.22 |
| (c) LLM$\rightarrow$GPT4 | 0.57 | 0.66 | 0.15 | 3 | 0.12 | 0.60 | 0.54 | 0.06 | 5 | 0.13 | 0.58 | 0.45 | 0.04 | 15 | 0.12 |
| (c) LLM$\rightarrow$Llama3.1-8B | 0.37 | 0.46 | 0.25 | 5 | 0.20 | 0.33 | 0.22 | 0.06 | 7 | 0.19 | 0.16 | 0.18 | 0.20 | 29 | 0.23 |
| (c) LLM$\rightarrow$Llama3.1-70B | 0.50 | 0.54 | 0.15 | 4 | 0.16 | 0.40 | 0.36 | 0.10 | 7 | 0.19 | 0.26 | 0.26 | 0.13 | 24 | 0.19 |
| (c) LLM$\rightarrow$Gemma2-9B | 0.37 | 0.46 | 0.25 | 5 | 0.20 | 0.16 | 0.16 | 0.16 | 8 | 0.22 | 0.16 | 0.18 | 0.20 | 29 | 0.23 |
| (c) LLM$\rightarrow$Ministral-7B | 0.60 | 0.60 | 0.10 | 4 | 0.16 | 0.33 | 0.33 | 0.13 | 7 | 0.19 | 0.21 | 0.18 | 0.10 | 25 | 0.20 |

表3：在基准数据集上对提出的MatMcd方法进行的消融分析。

基准。我们将MatMcd与在因果发现中最相关的最新方法进行比较，包括：（1）统计因果发现：Peter-Clark (PC) 算法 Spirtes 和 Glymour ([1991](https://arxiv.org/html/2412.13667v1#bib.bib37))，Exact Search (ES) 算法 Yuan 和 Malone ([2013](https://arxiv.org/html/2412.13667v1#bib.bib51))，以及DirectLiNGAM Shimizu 等人 ([2011](https://arxiv.org/html/2412.13667v1#bib.bib32))；（2）基于LLM的因果发现，仅使用LLM推断因果关系：Efficient-CDLMs Jiralerspong 等人 ([2024](https://arxiv.org/html/2412.13667v1#bib.bib9))，它采用基于BFS的LLM提示进行高效的因果图构建，以及MAC Le 等人 ([2024](https://arxiv.org/html/2412.13667v1#bib.bib15))，它使用辩论式LLM构建多代理系统；（3）混合方法，通过LLM优化SCD因果图：SCD-LLM，它使用单一LLM对SCD输出进行优化，ReAct Yao 等人 ([2022](https://arxiv.org/html/2412.13667v1#bib.bib50))，它在优化SCD图时交替进行推理和搜索工具使用，LLM-KBCI Takayama 等人 ([2024](https://arxiv.org/html/2412.13667v1#bib.bib38))，它使用ZSCOT两阶段提示优化因果图。

此外，我们为LLM-KBCI应用了ReAct框架，以支持交替推理和工具使用，并将此基准命名为LLM-KBCI-RA。我们还引入了Top-K Guess推理 Tian等人 ([2023](https://arxiv.org/html/2412.13667v1#bib.bib39))（K=2），用于LLM-KBCI的语言校准，并将此变体命名为LLM-KBCI-RE。

对于我们的方法，我们考虑了两种主要变体。第一种要求CC-agent（$\S$[3.3](https://arxiv.org/html/2412.13667v1#S3.SS3 "3.3 Causal Constraint Agent (CC-agent) ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")）给出单一答案，即在Top-K猜测推理中K=1，命名为MatMcd。第二种在Top-K猜测推理中使用K=2，我们命名为MatMcd-RE。此外，我们在表[3](https://arxiv.org/html/2412.13667v1#S4.T3 "Table 3 ‣ 4.1 Experimental Setup ‣ 4 Experiments ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")中对MatMcd的其他变体进行了广泛的消融分析，以评估其设计选择。

实现。默认情况下，我们使用GPT-4o mini，温度为0.5，作为所有基于LLM和混合方法的基础LLM，因为现有的研究Le等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib15)）发现它的性能非常优秀。对于所有混合方法，PC被用作基础SCD算法。同时，我们通过在消融分析中将LLM替换为GPT-4、Llama-3.1-8B、Llama-3.1-70B、Mistral-7B和Gemma2-9B，并将SCD算法替换为ES和DirectLiNGAM，来评估我们的MatMcd。所有SCD算法的实现均使用causal-learn Zheng等人（[2024b](https://arxiv.org/html/2412.13667v1#bib.bib56)）。ReAct和RAG框架的实现使用了LlamaIndex Liu（[2022](https://arxiv.org/html/2412.13667v1#bib.bib19)）。对于所有基准方法，我们在可用时使用了它们的官方代码。由于MAC的代码不可用，我们报告了其来自原始论文的结果，这些结果仅在AutoMPG、DWDClimate和SachsProtein数据集上可用。

评估指标。根据Kıcıman等人（[2023](https://arxiv.org/html/2412.13667v1#bib.bib12)）；Takayama等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib38)）；Khatibi等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib11)）；Le等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib15)）的方法，我们采用了广泛使用的指标，包括精度（Prc）、F1值（F1）、假阳性率（FPR）、结构哈明距离（SHD）和归一化哈明距离（NHD），用于衡量预测因果图与真实因果图之间的差异。Prc和F1衡量准确性，因此值越大越好。FPR、SHD和NHD衡量误差/差异，因此值越小越好。

### 4.2 实验结果

因果发现。表[1](https://arxiv.org/html/2412.13667v1#S3.T1 "表1 ‣ 3.4 因果图细化器 ‣ 3 方法论 ‣ 探索多模态集成与工具增强LLM代理用于精确因果发现") 和 [2](https://arxiv.org/html/2412.13667v1#S3.T2 "表2 ‣ 3.4 因果图细化器 ‣ 3 方法论 ‣ 探索多模态集成与工具增强LLM代理用于精确因果发现") 总结了基准数据集上的结果。从表格中我们可以得到以下几点观察：(1) 涉及LLM的方法通常在大多数情况下优于SCD算法，除了SachsProtein和Asia数据集，它们与生物医学有关。这表明LLM通过预训练所获得的常识知识在因果发现任务中具有巨大的潜力，但同时也引起我们对其缺乏领域特定知识的关注。(2) 混合方法在大多数情况下优于基于LLM的方法，表明使用SCD输出作为LLM参考的先验知识能够补充它们在因果效应相关知识上的不足。(3) 可以利用工具来检索外部数据的基准方法，如ReAct和LLM-KBCI-RA，有时优于其对应方法，表明增强的数据模态有潜力，但也呼吁更好的数据检索和使用方式。(4) 提出的MatMcd(-RE)在考虑到没有基准方法在所有数据集上表现稳定的情况下，达到了最佳的整体表现。特别是，MatMcd(-RE)通过数据增强帮助缓解了其他LLM基准方法在生物医学SachsProtein和Asia数据集上的幻觉问题。结果凸显了现有LLM基准方法仅通过元数据（如节点名称、数据标题）推断因果效应的挑战，并验证了多代理系统用于探索增强多模态因果发现的设计的有效性。最后，(5) 在贝叶斯数据集Asia和Child上，MatMcd-RE优于MatMcd，这可能是由于通过Top-K Guess推理为概率数据集更好地校准了似然。

![请参见说明](img/1302ee06bbc2c5801d33c710b698be66.png)

图4：图示(a) 事件日志和 (b) DA-agent 提供的摘要。

| 方法 | MAP@5 [↑] | MAP@10 [↑] | MRR [↑] | RK (P) [↓] | RK (C) [↓] |
| --- | --- | --- | --- | --- | --- |
| PC | 0.0% | 25.0% | 0.14 | 5 | 13 |
| Efficient-CDLMs | 0.0% | 0.0% | 0.10 | 10 | 10 |
| SCD-LLM | 0.0% | 25.0% | 0.14 | 5 | 13 |
| ReAct | 0.0% | 25.0% | 0.14 | 5 | 12 |
| LLM-KBCI | 10.0% | 30.0% | 0.16 | 4 | 13 |
| LLM-KBCI-RA | 0.0% | 25.0% | 0.14 | 5 | 12 |
| LLM-KBCI-RE | 10.0% | 30.0% | 0.16 | 4 | 13 |
| MatMcd | 30.0% | 55.0% | 0.32 | 2 | 7 |
| MatMcd-RE | 20.0% | 55.0% | 0.25 | 3 | 6 |

表4：不同因果发现方法在RCA任务中的比较。RK (P) 和 RK (C) 分别是不同方法在产品评论数据集和云计算数据集上预测的根因的排名。

消融研究。表格[3](https://arxiv.org/html/2412.13667v1#S4.T3 "Table 3 ‣ 4.1 Experimental Setup ‣ 4 Experiments ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")展示了我们使用三个数据集进行的消融分析。在表格中，MatMcd是我们的原始模型。在(a)中，我们旨在研究DA-agent中迭代搜索的有效性。为此，我们将迭代搜索替换为单轮搜索模板（附录[C.3](https://arxiv.org/html/2412.13667v1#A3.SS3 "C.3 Single-Round Search ‣ Appendix C Prompt Templates ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")）。在(b)中，我们评估了MatMcd如何通过将PC替换为ES和DirectLiNGAM来泛化到不同的SCD算法。在(c)中，我们评估了不同LLM对因果发现的影响。

从表格 [3](https://arxiv.org/html/2412.13667v1#S4.T3 "Table 3 ‣ 4.1 Experimental Setup ‣ 4 Experiments ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")(a) 中，我们观察到迭代搜索比单轮搜索更优，因为后者可能使用偏向的查询，并且在生成全面的增强数据时遇到困难。在附录 [D.1](https://arxiv.org/html/2412.13667v1#A4.SS1 "D.1 Data Augmentation Agent (DA-agent) ‣ Appendix D Examples of LLM Response ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery") 中，我们总结了DA-agent迭代生成的一些查询。在表格 [3](https://arxiv.org/html/2412.13667v1#S4.T3 "Table 3 ‣ 4.1 Experimental Setup ‣ 4 Experiments ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")(b) 中，使用PC以外的其他SCD算法通常会降低性能，可能是因为PC的约束设计更好地利用了LLM生成的约束。我们还观察到，在MatMcd的情况下，ES和DirectLiNGAM的性能相较于没有MatMcd的对应方法有所提升，如表格 [1](https://arxiv.org/html/2412.13667v1#S3.T1 "Table 1 ‣ 3.4 Causal Graph Refiner ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery") 中所示。有趣的是，表格 [3](https://arxiv.org/html/2412.13667v1#S4.T3 "Table 3 ‣ 4.1 Experimental Setup ‣ 4 Experiments ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")(c) 显示，默认的GPT-4o mini在不同数据集中的表现最为稳健，而GPT4优于开源LLM。这与Le等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib15)）的研究结果相似，其中使用GPT-4o mini的MAC表现最佳。与表格 [1](https://arxiv.org/html/2412.13667v1#S3.T1 "Table 1 ‣ 3.4 Causal Graph Refiner ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery") 中的PC相比，表格 [3](https://arxiv.org/html/2412.13667v1#S4.T3 "Table 3 ‣ 4.1 Experimental Setup ‣ 4 Experiments ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")(c) 中的所有LLM变体均提高了性能，表明在利用LLM时的有效性。此外，在生物数据集SachsProtein上，更大的模型GPT4进一步提升了因果发现的性能，这可能是因为其与特定领域的对齐度更高。

### 4.3 案例研究：根本原因分析 (RCA)

接下来，我们在从产品评价（PR）和云计算（CC）微服务系统中收集的AIOps数据集上评估MatMcd（Zheng等人，[2024a](https://arxiv.org/html/2412.13667v1#bib.bib54)）。PR数据集包含216个变量（即系统Pods），每个变量都关联一个多元时间序列，其中包含6个指标（例如，CPU、内存使用）且长度为131,329。CC数据集包含168个变量，每个变量都有一个7个指标的多元时间序列，长度为109,351。在这两个数据集中，每个变量还包含一条记录其历史事件的日志。图[4](https://arxiv.org/html/2412.13667v1#S4.F4 "图 4 ‣ 4.2 实验结果 ‣ 4 实验 ‣ 探索与工具增强LLM代理的多模态集成，以实现精确的因果发现")（a）展示了一段日志片段。通过日志查找工具（$\S$[3](https://arxiv.org/html/2412.13667v1#S3.F3 "图 3 ‣ 3.2 数据增强代理（DA-agent） ‣ 3 方法论 ‣ 探索与工具增强LLM代理的多模态集成，以实现精确的因果发现")），这些日志可以作为MatMcd的额外数据模态，用于增强因果发现。图[4](https://arxiv.org/html/2412.13667v1#S4.F4 "图 4 ‣ 4.2 实验结果 ‣ 4 实验 ‣ 探索与工具增强LLM代理的多模态集成，以实现精确的因果发现")（b）展示了DA-agent的Summary LLM可以有效地解释和总结日志数据。

由于这些数据集提供了由领域专家识别的系统故障的根本原因，但缺乏真实的因果图，我们使用它们来评估基于不同方法生成的因果图进行根本原因分析（RCA）的表现。根据Wang等人（[2023b](https://arxiv.org/html/2412.13667v1#bib.bib44)）的研究，如果在因果图上运行带有重启的随机游走（RWR）能够将根本原因在所有变量中排名靠前，这反映了因果图的质量。因此，我们使用广泛采用的指标，包括均值平均精度@K（MAP@K），其中K设置为5和10，以及均值倒排排名（MRR），以评估两个数据集上的整体排名表现（Zheng等人，[2024a](https://arxiv.org/html/2412.13667v1#bib.bib54)）。有关MAP@K和MRR的更多细节，参见附录[B.2](https://arxiv.org/html/2412.13667v1#A2.SS2 "B.2 RCA中的评估指标 ‣ 附录B 实验细节 ‣ 探索与工具增强LLM代理的多模态集成，以实现精确的因果发现")。

根据王等人（[2023b](https://arxiv.org/html/2412.13667v1#bib.bib44)）的方法，我们通过使用基于极值理论的Siffer等人（[2017](https://arxiv.org/html/2412.13667v1#bib.bib34)）的方式过滤掉不相关的pod，预处理了数据集，随后应用因果发现方法。表[4](https://arxiv.org/html/2412.13667v1#S4.T4 "Table 4 ‣ Figure 4 ‣ 4.2 Experimental Results ‣ 4 Experiments ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")总结了结果，在其中我们还报告了每个数据集的真实根本原因的具体排名（“RK”）。在表[4](https://arxiv.org/html/2412.13667v1#S4.T4 "Table 4 ‣ Figure 4 ‣ 4.2 Experimental Results ‣ 4 Experiments ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")中，ES和DirectLiNGAM被排除，因为重点在于不同的基于LLM的方法如何改进其基础的SCD算法，即PC，以便更好地完成下游任务。从表[4](https://arxiv.org/html/2412.13667v1#S4.T4 "Table 4 ‣ Figure 4 ‣ 4.2 Experimental Results ‣ 4 Experiments ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")可以看出，通过利用日志模态，MatMcd(-RE)显著提高了两个数据集上根本原因定位的准确性，相较于最佳基准在MAP@10上有83.3%的相对提升。这突显了将常见日志数据集成到微服务系统中用于根本原因分析（RCA）和AIOps的潜力。

更多可视化结果可参见$\S$[B.3](https://arxiv.org/html/2412.13667v1#A2.SS3 "B.3 Graph Visualization ‣ Appendix B Experimental Details ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")。

## 5 结论

本文通过设计MatMcd与工具增强型LLM，探讨了因果发现中的多模态数据，其机制专门设计用于将文本数据与图形结合。实验不仅验证了我们创新方法的有效性，还为深入研究多模态因果发现设立了新的标准。

## 6 限制

基于LLM的因果发现研究是一个新兴领域，正在进行积极探索。本研究开创性地使用了工具增强型LLM代理，但所提出的方法与现有大多数方法一样，存在类似的局限性。

首先，在因果图中，因果关系的检验是基于对偶关系的，即每个因果关系$(v_{i},v_{j})$，其中$1\leq i<j\leq n$，需要通过提示给LLM（在我们这里是CC-agent）来评估其正确性。这可能会在大规模图中导致可扩展性问题。这一点也反映在该领域广泛使用的基准数据集的规模上。提高这种基于边缘的问答范式可扩展性的一个有前景的方法是利用因果图的DAG结构，并使用广度优先搜索（BFS）风格的提示，正如Jiralerspong等人（[2024](https://arxiv.org/html/2412.13667v1#bib.bib9)）所提出的那样。所提议的MatMcd完全兼容这种提示技术，因此有可能实现$O(n)$复杂度。其次，类似于其他结合SCD算法和LLM代理的混合方法，所提议的MatMcd需要一个能够有效整合LLM生成约束的基础SCD算法。我们从使用PC算法比使用ES或DirectLiNGAM在表[3](https://arxiv.org/html/2412.13667v1#S4.T3 "Table 3 ‣ 4.1 Experimental Setup ‣ 4 Experiments ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")中得到的更好结果中观察到了这一点。这一问题值得进一步研究，全面比较更多基于约束的SCD算法与非约束变种的效果。第三，作为一种知识驱动的方法，需要具有语义意义的元数据变量来检索有关因果关系的有用知识。对于变量在语义上无意义或元数据是私密的领域，这类方法可能具有有限的影响。

除了前面提到的常见局限性外，本研究还有一些具体的局限性。首先，本研究使用网络数据和日志作为增强数据模态，但在某些领域中，像代码、图像和音频信号等其他模态仍需进一步研究。其次，除了提议的DA-agent中的网页搜索API和日志查找API外，像Wikipedia API和代码查找API等工具也值得进一步探索。DA-agent中的工具包具有灵活性和可扩展性，能够支持进一步的研究。

## 参考文献

+   Achiam等人（2023）Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat等人. 2023. GPT-4技术报告。*arXiv预印本arXiv:2303.08774*。

+   Ban等人（2023）Taiyu Ban, Lyuzhou Chen, Derui Lyu, Xiangyu Wang, 和 Huanhuan Chen. 2023. 由大语言模型监督的因果结构学习。*arXiv预印本arXiv:2311.11689*。

+   Brown等人（2020）Tom B. Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, Sandhini Agarwal, Ariel Herbert-Voss, Gretchen Krueger, Tom Henighan, Rewon Child, Aditya Ramesh, Daniel M. Ziegler, Jeffrey Wu, Clemens Winter, Christopher Hesse, Mark Chen, Eric Sigler, Mateusz Litwin, Scott Gray, Benjamin Chess, Jack Clark, Christopher Berner, Sam McCandlish, Alec Radford, Ilya Sutskever, 和 Dario Amodei. 2020. [语言模型是少样本学习者](https://arxiv.org/abs/2005.14165)。*预印本*，arXiv:2005.14165。

+   Chen等人（2024a）Sirui Chen, Bo Peng, Meiqi Chen, Ruiqi Wang, Mengying Xu, Xingyu Zeng, Rui Zhao, Shengjie Zhao, Yu Qiao, 和 Chaochao Lu. 2024a. 语言模型的因果评估。*arXiv预印本 arXiv:2405.00622*。

+   Chen等人（2024b）Yongchao Chen, Jacob Arkin, Yang Zhang, Nicholas Roy, 和 Chuchu Fan. 2024b. 使用大语言模型的可扩展多机器人协作：集中式或分散式系统？在*2024 IEEE国际机器人与自动化会议（ICRA）*，第4311–4317页。IEEE。

+   Chickering（2002）David Maxwell Chickering. 2002. 使用贪心搜索的最优结构识别。*机器学习研究杂志*，3（11月）：507–554。

+   Hoyer等人（2008）Patrik Hoyer, Dominik Janzing, Joris M Mooij, Jonas Peters, 和 Bernhard Schölkopf. 2008. 使用加性噪声模型的非线性因果发现。*神经信息处理系统进展*，21。

+   Huang等人（2018）Biwei Huang, Kun Zhang, Yizhu Lin, Bernhard Schölkopf, 和 Clark Glymour. 2018. 用于因果发现的广义评分函数。在*第24届ACM SIGKDD国际知识发现与数据挖掘会议论文集*，第1551–1560页。

+   Jiralerspong等人（2024）Thomas Jiralerspong, Xiaoyin Chen, Yash More, Vedant Shah, 和 Yoshua Bengio. 2024. 使用大语言模型的高效因果图发现。*arXiv预印本 arXiv:2402.01207*。

+   Karpas等人（2022）Ehud Karpas, Omri Abend, Yonatan Belinkov, Barak Lenz, Opher Lieber, Nir Ratner, Yoav Shoham, Hofit Bata, Yoav Levine, Kevin Leyton-Brown 等人. 2022. Mrkl系统：一种模块化的神经符号架构，结合了大语言模型、外部知识源和离散推理。*arXiv预印本 arXiv:2205.00445*。

+   Khatibi等人（2024）Elahe Khatibi, Mahyar Abbasian, Zhongqi Yang, Iman Azimi, 和 Amir M Rahmani. 2024. Alcm: 自动化的LLM增强因果发现框架。*arXiv预印本 arXiv:2405.01744*。

+   Kıcıman等人（2023）Emre Kıcıman, Robert Ness, Amit Sharma, 和 Chenhao Tan. 2023. 因果推理与大语言模型：开启因果关系的新前沿。*arXiv预印本 arXiv:2305.00050*。

+   Kojima等人（2022）Takeshi Kojima, Shixiang Shane Gu, Machel Reid, Yutaka Matsuo, 和 Yusuke Iwasawa. 2022. 大语言模型是零-shot推理器。*神经信息处理系统进展*，35:22199–22213。

+   Lauritzen 和 Spiegelhalter（1988）Steffen L Lauritzen 和 David J Spiegelhalter. 1988. 在图形结构上进行概率的局部计算及其在专家系统中的应用。*皇家统计学会杂志：B系列（方法论）*，50(2)：157–194。

+   Le 等（2024）Hao Duong Le, Xin Xia 和 Zhang Chen. 2024. 使用大型语言模型进行多智能体因果发现。*arXiv 预印本 arXiv:2407.15073*。

+   Lewis 等（2020）Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel 等. 2020. 用于知识密集型 NLP 任务的检索增强生成。*神经信息处理系统进展*，33：9459–9474。

+   Li 等（2023）Guohao Li, Hasan Hammoud, Hani Itani, Dmitrii Khizbullin 和 Bernard Ghanem. 2023. Camel：用于大型语言模型社会“心智”探索的交互式代理。*神经信息处理系统进展*，36：51991–52008。

+   Liang 等（2023）Tian Liang, Zhiwei He, Wenxiang Jiao, Xing Wang, Yan Wang, Rui Wang, Yujiu Yang, Shuming Shi 和 Zhaopeng Tu. 2023. 通过多智能体辩论激励大型语言模型中的发散性思维。*arXiv 预印本 arXiv:2305.19118*。

+   Liu（2022）Jerry Liu. 2022. [LlamaIndex](https://doi.org/10.5281/zenodo.1234)。

+   Long 等（2023）Stephanie Long, Alexandre Piché, Valentina Zantedeschi, Tibor Schuster 和 Alexandre Drouin. 2023. 使用语言模型作为不完美专家进行因果发现。*arXiv 预印本 arXiv:2307.02390*。

+   Madaan 等（2024）Aman Madaan, Niket Tandon, Prakhar Gupta, Skyler Hallinan, Luyu Gao, Sarah Wiegreffe, Uri Alon, Nouha Dziri, Shrimai Prabhumoye, Yiming Yang 等. 2024. 自我精炼：通过自我反馈的迭代精炼。*神经信息处理系统进展*，36。

+   Mooij 等（2016）Joris M Mooij, Jonas Peters, Dominik Janzing, Jakob Zscheischler 和 Bernhard Schölkopf. 2016. 使用观察数据区分因果关系：方法与基准测试。*机器学习研究杂志*，17(32)：1–102。

+   Nguyen 等（2023）Trang Nguyen, Alexander Tong, Kanika Madan, Yoshua Bengio 和 Dianbo Liu. 2023. 使用 gflownet 进行基因调控网络中的因果推断：面向大规模系统的可扩展性。*arXiv 预印本 arXiv:2310.03579*。

+   OpenAI（2024）OpenAI. 2024. 函数调用指南。URL: https://platform.openai.com/docs/guides/function-calling. 访问时间：2024-12-10。

+   Qian 等（2023）Chen Qian, Xin Cong, Cheng Yang, Weize Chen, Yusheng Su, Juyuan Xu, Zhiyuan Liu 和 Maosong Sun. 2023. 用于软件开发的交互式代理。*arXiv 预印本 arXiv:2307.07924*，6:3。

+   Quinlan（1993）R. Quinlan. 1993. Auto MPG. UCI 机器学习数据集。DOI: https://doi.org/10.24432/C5859H。

+   Rolland 等人（2022）Paul Rolland, Volkan Cevher, Matthäus Kleindessner, Chris Russell, Dominik Janzing, Bernhard Schölkopf 和 Francesco Locatello. 2022. 分数匹配使非线性加性噪声模型的因果发现成为可能。在*国际机器学习大会*上，页码 18741-18753。PMLR。

+   Sachs 等人（2005）Karen Sachs, Omar Perez, Dana Pe’er, Douglas A Lauffenburger 和 Garry P Nolan. 2005. 从多参数单细胞数据中推导出的因果蛋白质信号网络。*科学*，308（5721）：523-529。

+   Schick 等人（2024）Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Eric Hambro, Luke Zettlemoyer, Nicola Cancedda 和 Thomas Scialom. 2024. Toolformer: 语言模型可以自学使用工具。*神经信息处理系统进展*，36。

+   Shen 等人（2024）Yongliang Shen, Kaitao Song, Xu Tan, Dongsheng Li, Weiming Lu 和 Yueting Zhuang. 2024. Hugginggpt: 使用ChatGPT和Hugging Face中的朋友解决AI任务。*神经信息处理系统进展*，36。

+   Shimizu 等人（2006）Shohei Shimizu, Patrik O Hoyer, Aapo Hyvärinen, Antti Kerminen 和 Michael Jordan. 2006. 一种用于因果发现的线性非高斯无环模型。*机器学习研究杂志*，7（10）。

+   Shimizu 等人（2011）Shohei Shimizu, Takanori Inazumi, Yasuhiro Sogawa, Aapo Hyvarinen, Yoshinobu Kawahara, Takashi Washio, Patrik O Hoyer, Kenneth Bollen 和 Patrik Hoyer. 2011. Directlingam: 一种学习线性非高斯结构方程模型的直接方法。*机器学习研究杂志-JMLR*，12（4月）：1225-1248。

+   Shinn 等人（2024）Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik Narasimhan 和 Shunyu Yao. 2024. Reflexion: 具有语言强化学习的语言代理。*神经信息处理系统进展*，36。

+   Siffer 等人（2017）Alban Siffer, Pierre-Alain Fouque, Alexandre Termier 和 Christine Largouet. 2017. 使用极值理论进行流中的异常检测。在*第23届ACM SIGKDD国际知识发现与数据挖掘大会论文集*中，页码 1067-1075。

+   Silander 和 Myllymäki（2006）Tomi Silander 和 Petri Myllymäki. 2006. 一种寻找全局最优贝叶斯网络结构的简单方法。在*第二十二届人工智能不确定性会议论文集*中，页码 445-452。

+   Spiegelhalter（1992）David J Spiegelhalter. 1992. 概率专家系统中的学习。*贝叶斯统计*，4：447-465。

+   Spirtes 和 Glymour（1991）Peter Spirtes 和 Clark Glymour. 1991. 一种快速恢复稀疏因果图的算法。*社会科学计算机评论*，9（1）：62-72。

+   Takayama 等人（2024）Masayuki Takayama, Tadahisa Okuda, Thong Pham, Tatsuyoshi Ikenoue, Shingo Fukuma, Shohei Shimizu 和 Akiyoshi Sannai. 2024. 在因果发现中整合大型语言模型：一种统计因果方法。*arXiv 预印本 arXiv:2402.01454*。

+   Tian 等人 (2023) Katherine Tian, Eric Mitchell, Allan Zhou, Archit Sharma, Rafael Rafailov, Huaxiu Yao, Chelsea Finn, 和 Christopher D Manning. 2023. 只需请求校准：通过人类反馈微调的语言模型中引导校准信心分数的策略。在 *2023年自然语言处理经验方法会议论文集*，第5433–5442页。

+   Tu 等人 (2019) Ruibo Tu, Kun Zhang, Bo Bertilson, Hedvig Kjellstrom, 和 Cheng Zhang. 2019. 用于因果发现算法评估的神经病理性疼痛诊断模拟器。*神经信息处理系统进展*，32。

+   Tu 等人 (2022) Ruibo Tu, Kun Zhang, Hedvig Kjellstrom, 和 Cheng Zhang. 2022. 因果发现的最优传输方法。在 *国际学习表示会议*。

+   Vashishtha 等人 (2023) Aniket Vashishtha, Abbavaram Gowtham Reddy, Abhinav Kumar, Saketh Bachu, Vineeth N Balasubramanian, 和 Amit Sharma. 2023. 使用 LLM 指导的发现进行因果推断。*arXiv 预印本 arXiv:2310.15117*。

+   Wang 等人 (2023a) Dongjie Wang, Zhengzhang Chen, Yanjie Fu, Yanchi Liu, 和 Haifeng Chen. 2023a. 在线根本原因分析的增量因果图学习。在 *第29届 ACM SIGKDD 知识发现与数据挖掘大会论文集*，第2269–2278页。

+   Wang 等人 (2023b) Dongjie Wang, Zhengzhang Chen, Jingchao Ni, Liang Tong, Zheng Wang, Yanjie Fu, 和 Haifeng Chen. 2023b. 用于根本原因定位的相互依赖因果网络。在 *第29届 ACM SIGKDD 知识发现与数据挖掘大会论文集*，第5051–5060页。

+   Wang 等人 (2024) Heng Wang, Shangbin Feng, Tianxing He, Zhaoxuan Tan, Xiaochuang Han, 和 Yulia Tsvetkov. 2024. 语言模型能否解决自然语言中的图问题？*神经信息处理系统进展*，36。

+   Wei 等人 (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, 等人. 2022. 思维链提示引发大型语言模型推理。*神经信息处理系统进展*，35:24824–24837。

+   Xie 等人 (2020) Feng Xie, Ruichu Cai, Biwei Huang, Clark Glymour, Zhifeng Hao, 和 Kun Zhang. 2020. 用于估计潜在变量因果图的广义独立噪声条件。*神经信息处理系统进展*，33:14891–14902。

+   Xiong 等人 (2023) Kai Xiong, Xiao Ding, Yixin Cao, Ting Liu, 和 Bing Qin. 2023. 研究大型语言模型协作的相互一致性：通过辩论进行深入分析。*arXiv 预印本 arXiv:2305.11595*。

+   Xiong 等人 (2024) Miao Xiong, Zhiyuan Hu, Xinyang Lu, YIFEI LI, Jie Fu, Junxian He, 和 Bryan Hooi. 2024. LLM 是否能表达其不确定性？LLM 中信心引导的实证评估。 在 *国际学习表示会议*。

+   Yao 等人 (2022) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan 和 Yuan Cao. 2022. React: 在语言模型中协同推理与行动。*arXiv 预印本 arXiv:2210.03629*。

+   Yuan 和 Malone (2013) Changhe Yuan 和 Brandon Malone. 2013. 学习最优贝叶斯网络：一种最短路径视角。*人工智能研究杂志*，48:23–65。

+   Zečević 等人 (2023) Matej Zečević, Moritz Willig, Devendra Singh Dhami 和 Kristian Kersting. 2023. 因果鹦鹉：大语言模型可能谈论因果关系，但并非真正的因果模型。*arXiv 预印本 arXiv:2308.13067*。

+   Zhao 等人 (2023) Qinlin Zhao, Jindong Wang, Yixuan Zhang, Yiqiao Jin, Kaijie Zhu, Hao Chen 和 Xing Xie. 2023. Competeai: 理解基于大语言模型的智能体中的竞争行为。*arXiv 预印本 arXiv:2310.17512*。

+   Zheng 等人 (2024a) Lecheng Zheng, Zhengzhang Chen, Dongjie Wang, Chengyuan Deng, Reon Matsuoka 和 Haifeng Chen. 2024a. Lemma-rca: 一种用于根本原因分析的大型多模态多领域数据集。*arXiv 预印本 arXiv:2406.05375*。

+   Zheng 等人 (2018) Xun Zheng, Bryon Aragam, Pradeep K Ravikumar 和 Eric P Xing. 2018. 无泪的 DAG：结构学习的连续优化。*神经信息处理系统进展*，31。

+   Zheng 等人 (2024b) Yujia Zheng, Biwei Huang, Wei Chen, Joseph Ramsey, Mingming Gong, Ruichu Cai, Shohei Shimizu, Peter Spirtes 和 Kun Zhang. 2024b. Causal-learn: 用于因果发现的 Python 工具。*机器学习研究杂志*，25(60):1–8。

## 附录 A 算法

MatMcd 的算法在算法 [1](https://arxiv.org/html/2412.13667v1#algorithm1 "在附录 A 算法 ‣ 探索带工具增强的大型语言模型智能体的多模态整合以实现精确的因果发现") 中进行了总结。符号与 $\S$[3](https://arxiv.org/html/2412.13667v1#S3 "3 方法论 ‣ 探索带工具增强的大型语言模型智能体的多模态整合以实现精确的因果发现") 中的一致。

输入：（1）观察数据：$\{{\bf v}_{1},...,{\bf v}_{n}\}$，其中${\bf v}_{i}=\{v_{i1},...,v_{im}\}$包含了变量$v_{i}$的$m$个样本；（2）元数据$\mathcal{D}=\{\mathsf{s},\mathcal{Z}\}$，其中$\mathsf{s}$是数据集的描述性标题，$\mathcal{Z}=\{\mathsf{z}_{1},...,\mathsf{z}_{n}\}$包含每个变量$v_{i}$在$\mathcal{V}$中的描述性名称$\mathsf{z}_{i}$；（3）一个SCD算法$f_{\text{SCD}}(\cdot)$；（4）在Top-K猜测提示中用来引导约束LLM的口头信心的猜测次数$K$。输出：优化后的因果图${\bf G}$1/* 因果图估计器（$\S$[3.1](https://arxiv.org/html/2412.13667v1#S3.SS1 "3.1 因果图估计器 ‣ 3 方法论 ‣ 探索多模态集成与工具增强LLM代理的精准因果发现")） */2 初始因果图${\bf G}_{0}\leftarrow\texttt{CaussalGraphEstimator}(f_{\text{SCD}},\{{\bf v}_{% 1},...,{\bf v}_{n}\})$/* DA-agent ($\S$[3.2](https://arxiv.org/html/2412.13667v1#S3.SS2 "3.2 数据增强代理（DA-agent） ‣ 3 方法论 ‣ 探索多模态集成与工具增强LLM代理的精准因果发现")） */3 初始化调用历史记忆$\mathcal{C}\leftarrow\emptyset$4 初始化检索数据记忆$\mathcal{R}\leftarrow\emptyset$查询$\leftarrow\texttt{Search-LLM}(\mathsf{s},\mathcal{Z},\mathcal{C})$ // 在检查记忆$\mathcal{C}$后生成搜索查询5 当 *查询 $\neq$ “不需要查询”* 时        /* 迭代搜索 */       $\mathcal{R}\leftarrow\texttt{Toolkit}(\text{查询})$         // 将检索的数据添加到记忆$\mathcal{R}$       $\mathcal{C}\leftarrow\text{查询}$         // 将搜索查询添加到记忆$\mathcal{C}$       查询$\leftarrow\texttt{Search-LLM}(\mathsf{s},\mathcal{Z},\mathcal{C})$         // 在检查记忆$\mathcal{C}$后更新搜索查询6    7 结束循环8上下文数据模态$\leftarrow\texttt{Summary-LLM}(\mathsf{s},\mathcal{Z},\mathcal{R})$/* CC-agent ($\S$[3.3](https://arxiv.org/html/2412.13667v1#S3.SS3 "3.3 因果约束代理（CC-agent） ‣ 3 方法论 ‣ 探索多模态集成与工具增强LLM代理的精准因果发现")） */9 初始化约束矩阵${\bf C}$10 对于 *每对变量$(v_{i},v_{j})$* 时11        $\text{Explanation}_{i,j}\leftarrow\texttt{Knowledge-LLM}({\bf G}_{0},v_{i},v_{% j},\text{上下文数据模态})$        $\{(\text{Constraint}_{k},\text{Confidence}_{k}\}_{k=1}^{K}\leftarrow\texttt{% Constraint-LLM}(\text{Explanation}_{i,j})$         // Top-K猜测12        $k_{*}\leftarrow\mathrm{argmax}_{k}\text{Confidence}_{k}$        ${\bf C}_{i,j}\leftarrow\text{Constraint}_{k_{*}}$         // 更新约束矩阵，使用$\text{Constraint}_{k_{*}}$替代$(v_{i},v_{j})$的约束13    14 结束循环/* 因果图优化器（$\S$[3.4](https://arxiv.org/html/2412.13667v1#S3.SS4 "3.4 因果图优化器 ‣ 3 方法论 ‣ 探索多模态集成与工具增强LLM代理的精准因果发现")） */15 优化后的因果图${\bf G}\leftarrow\texttt{CausalGraphRefiner}(f_{\text{SCD}},\{{\bf v}_{1},...,% {\bf v}_{n}\},{\bf C})$

算法 1 基于工具增强LLMs的多模态因果发现多智能体（MatMcd）

## 附录B 实验细节

### B.1 实现细节

对于PC算法，Fisher的Z检验用于其条件独立性检验。对于精确搜索，启用了其k-cycle启发式，$k=2$，最大父节点数限制为2。对于DirectLiNGAM，采用了软先验知识集成以避免与LLM生成的约束冲突。对于LLMs，默认将温度设置为0.5。然而，某些LLM（例如Llama 3.1，Mistral）可能会生成不符合提示要求的输出。在这种情况下，我们尝试将温度调整为0.7并多次重新运行。对于基准方法Efficient-CDLM，它可能会在多次运行中产生不同的结果。因此，我们报告了其在每个数据集上10次运行的最佳性能。

对于RAG的实现，使用了LlamaIndex，并通过text-embedding-ada-002进行文档分块索引。对于DA-agent中的网络搜索工具，采用了Google搜索API通过Serper（[https://serper.dev/](https://serper.dev/)）。如果返回的信息不足或不完整，则使用Tavily（[https://tavily.com/](https://tavily.com/)）作为补充。通过整合多个来源的信息，我们确保了全面和多样化的背景，有助于因果发现和LLM提示。对于日志查找工具，使用了基于模板的日志来系统地记录事件。具体来说，事件被限制为最多10次出现，并通过随机抽样进行选择。

### B.2 RCA中的评估指标

为了评估RCA的性能，我们使用了一些广泛使用的指标。PR@K（精准度@K）用于衡量前K个预测根本原因为真实的概率，定义如下：

|  | $\mathrm{PR@K}=\frac{1}{\mathbb{A}}\sum_{a\in\mathbb{A}}\frac{\sum_{i<k}R_{a}(i% )\in V_{a}}{\mathrm{min}(K,&#124;V_{a}&#124;)}$ |  |
| --- | --- | --- |

其中，$\mathbb{A}$是系统故障的集合，$a$是$\mathbb{A}$中的一个故障，$V_{a}$是故障$a$的实际根本原因集合，$R_{a}$是故障$a$的预测根本原因，$i$是$R_{a}$中的第$i$个预测原因。

均值平均精度@K（MAP@K）评估总体排名性能，定义如下。

|  | $\mathrm{MAP@K}=\frac{1}{K&#124;\mathbb{A}&#124;}\sum_{a\in\mathbb{A}}\sum_{i\leq j\leq K% }\mathrm{PR@j}$ |  |
| --- | --- | --- |

MRR（均值倒数排名）用于评估模型的排名能力，定义如下。

|  | $\mathrm{MRR@K}=\frac{1}{\mathbb{A}}\sum_{a\in\mathbb{A}}\frac{1}{\mathrm{rank}% _{R_{a}}}$ |  |
| --- | --- | --- |

### B.3 图形可视化

在本节中，我们展示了图[5](https://arxiv.org/html/2412.13667v1#A2.F5 "Figure 5 ‣ B.3 Graph Visualization ‣ Appendix B Experimental Details ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")、图[6](https://arxiv.org/html/2412.13667v1#A2.F6 "Figure 6 ‣ B.3 Graph Visualization ‣ Appendix B Experimental Details ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")、图[7](https://arxiv.org/html/2412.13667v1#A2.F7 "Figure 7 ‣ B.3 Graph Visualization ‣ Appendix B Experimental Details ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")、图[8](https://arxiv.org/html/2412.13667v1#A2.F8 "Figure 8 ‣ B.3 Graph Visualization ‣ Appendix B Experimental Details ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")和图[9](https://arxiv.org/html/2412.13667v1#A2.F9 "Figure 9 ‣ B.3 Graph Visualization ‣ Appendix B Experimental Details ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")的可视化结果。每个图中包括了真实的因果图、PC算法预测的因果图、MatMcd生成的因果图以及MatMcd-R生成的因果图。在图中，红色箭头表示错误添加的边，蓝色箭头表示错误反向的边。每个图中，我们还包含了错误增加的边、遗漏的边和反向的边的数量以供对比。

与PC算法相比，MatMcd(-R)生成的因果图更接近真实的因果图，通常在错误增加、遗漏和反向边的数量上更少。这一优势源自LLM代理能够施加多模态数据驱动的知识约束，从而筛选错误的边。而PC算法仅依赖于变量间观测数据的相关性，因此更容易出错。

此外，与依赖无向图结构来确定因果边方向的PC算法不同，结合LLM提供的知识可以在确定因果方向时提供精确的指导。因此，所提出的MatMcd(-R)在大多数数据集中表现出较少的反向边，特别是对于那些具有更多可获取的多模态数据（例如非生物医学数据集）的情况。

![请参见说明文字](img/960c715a891b3675a66b06421b439004.png)

图 5：AutoMPG数据集的因果图可视化。

![请参见说明文字](img/18b9211578fdd37635ebc175d057064f.png)

图 6：DWDClimate数据集的因果图可视化。

![请参见说明文字](img/9a1d275098bfcdb5a49ed7ddc6ebd89b.png)

图 7：SachsProtein数据集的因果图可视化。

![请参见说明文字](img/1626d67f088d4be208a00f7ee8b46314.png)

图 8：亚洲数据集的因果图可视化。

![请参见说明文字](img/9137ae223bd46e593922daf97123260b.png)

图 9：儿童数据集的因果图可视化。

## 附录 C 提示模板

在本节中，我们提供了数据增强代理（DA-agent）中搜索 LLM 和总结 LLM 的提示模板 （$\S$[3.2](https://arxiv.org/html/2412.13667v1#S3.SS2 "3.2 Data Augmentation Agent (DA-agent) ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")），以及因果约束代理（CC-agent）中知识 LLM 和约束 LLM 的提示模板（$\S$[3.3](https://arxiv.org/html/2412.13667v1#S3.SS3 "3.3 Causal Constraint Agent (CC-agent) ‣ 3 Methodology ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")）。此外，我们还包括了用于消融分析中的单轮搜索的提示模板，见表 [3](https://arxiv.org/html/2412.13667v1#S4.T3 "Table 3 ‣ 4.1 Experimental Setup ‣ 4 Experiments ‣ Exploring Multi-Modal Integration with Tool-Augmented LLM Agents for Precise Causal Discovery")(a)。

### C.1 数据增强代理（DA-agent）的提示模板

搜索 LLM 假设你对 <dataset name> 数据集及其节点 <list of node names> 没有任何先前的了解。你需要生成查询供其他人搜索信息，以帮助你总结数据集和节点，并帮助你识别节点之间的关系。将你的查询分成多个子查询。确保每个查询都是具体的、清晰的，并且易于搜索。如果你之前询问过这个话题，你之前的查询是：<list of previous queries> 请尽量避免重复这些查询。生成一个新查询以获取更多信息。格式如下：搜索查询：<new query> 每次只生成一个查询。如果你认为不需要更多的查询，请说明 “无需查询”。

总结 LLM（带 RAG）请提供关于 <dataset name> 数据集及其节点（包括 <list of node names>）的总结，使用 RAG 数据库中的信息。您的回答应包括数据集及其每个节点的详细信息。请按照以下结构化格式作答： - 数据集总结：<数据集的总体总结> - <node name> 的总结：<该节点的详细总结> - <node name> 的总结：<该节点的详细总结> 此外，尝试在总结每个节点后，提供节点之间关系的信息。确保涵盖所有提到的节点。

摘要LLM（带日志）在<数据集名称>数据集中，包含以下实体：<节点名称列表>。关于实体<节点名称>的日志信息按以下格式提供：<日志格式模板>。以下是按照上述格式提供的日志信息：<节点事件模板和示例列表>。根据上述信息，请提供您对该实体<节点名称>的总结，包括：- 实体的角色：<节点名称>在系统中的角色- 需要注意的关键事件。既要关注频繁发生的事件，也要关注不常发生的事件- 该实体的状态- 该实体：<节点名称>与上述实体列表中的其他实体的关系。只包括最可能的关系。您只需要提供关于该实体<节点名称>的信息。您的回答应采用以下格式：实体名称：<节点名称> 实体角色：<角色> 需要注意的关键事件：<关键事件> 该实体的状态：<状态> <节点名称>的关系：<关系> 您的回答：

### C.2 因果约束代理（CC-agent）

知识LLM 我们想对<数据集名称>进行因果推断，数据集的摘要如下：<数据集信息>。将<节点名称列表>作为变量。我们已使用<SCD算法名称>算法进行了统计因果发现。统计发现所建议的因果结构的边和它们的系数如下：<因果图的邻接列表>。根据上述信息，似乎<节点名称i>的变化对<节点名称j>有<有/没有>直接影响。此外，以下是来自可靠来源的<节点名称i>和<节点名称j>的信息：<节点i的信息> <节点j的信息>。您的任务是从领域知识的角度解释这一结果，并判断该统计学建议的假设在该领域的背景下是否可信。请提供一个利用您在<节点名称i>和<节点名称j>之间因果关系的专业知识的解释，并评估这一因果发现结果的正确性。您的回答应考虑相关因素，并基于您对该领域的理解提供合理的解释。

约束 LLM 提供你的<K>个最佳猜测以及每个猜测的正确概率（0.0到1.0）来回答以下问题。仅提供猜测和概率，不要其他词语或解释。每个猜测应逐步推理关系，并最终以<Yes>或<No>结束。例如：G1: <最可能的第一个猜测，逐步推理关系并以<Yes>或<No>结束> P1: <G1正确的概率，范围在0.0到1.0之间，不要附加任何评论；仅提供概率！> G2: <第二个最可能的猜测，逐步推理关系并以<Yes>或<No>结束> P2: <G2正确的概率，范围在0.0到1.0之间，不要附加任何评论；仅提供概率！> 问题是：以下是<dataset name>领域专家对<node name i>和<node name j>之间因果关系的解释：<domain knowledge output of Knowledge LLM> 根据上述信息，如果<node name i>发生变化，是否会直接影响<node name j>？

### C.3 单轮搜索

在单轮搜索的实施过程中，结合以下提示使用了SearchGPT（[https://github.com/Wilson-ZheLin/SearchGPT](https://github.com/Wilson-ZheLin/SearchGPT)），直接搜索与数据集和节点相关的信息，因此无需LLM生成查询。

单轮搜索 请提供数据集<dataset name>及其变量的详细信息，包括<list of node names>。如有可能，尝试搜索更多信息。你应提供每个变量及其相互关系的详细信息。请按照以下结构化格式提供答案： 数据集<dataset name>概述： - 数据集概述：<数据集的一般概述> - <node name>概述：<此节点的详细概述> - <node name>概述：<此节点的详细概述> 此外，在总结每个节点后，尽量提供节点之间关系的信息。确保涵盖所有提到的节点。

## 附录 D LLM响应示例

### D.1 数据增强代理（DA-agent）

本节提供了DA-agent在AutoMPG数据集上的搜索过程中LLM的响应。

AutoMPG数据集摘要：Auto MPG数据集是机器学习和统计学领域中一个著名的数据集，主要用于回归分析和预测建模。它包含了关于各种汽车的信息，特别关注它们的燃油效率，单位为每加仑多少英里（MPG）。该数据集包括几个关键变量，这些变量对于理解车辆性能至关重要，包括排量、MPG、马力、重量和加速度。 - - - 排量摘要：排量指的是发动机中所有气缸的总体积，通常以立方英寸或升为单位进行测量。它是发动机大小和容量的关键指标。一般来说，较大的排量可能会带来更高的功率输出，但也可能导致较低的燃油效率。 - - - MPG摘要：MPG（每加仑多少英里）是数据集中的目标变量，表示车辆的燃油效率。较高的MPG值表示更好的燃油效率，意味着车辆可以在一加仑油的情况下行驶更远。MPG与其他变量之间通常存在反向关系；随着发动机尺寸（排量）和重量的增加，MPG往往会下降。 - - - 马力摘要：马力衡量发动机的功率输出。较高的马力通常与更好的加速性和整体性能相关。然而，像排量一样，较高的马力也可能导致较低的燃油效率，因为更多的功率通常需要更多的燃料。 - - - 重量摘要：车辆的重量是另一个影响性能的关键因素。较重的车辆通常需要更多的动力来加速，这可能会对MPG产生负面影响。重量与MPG之间通常存在负相关关系；随着重量的增加，燃油效率往往会下降。 - - - 加速度摘要：加速度衡量车辆增加速度的速度。它受到马力和重量的影响；更强劲的发动机能够更快地加速，而较重的车辆可能在达到相同加速度时遇到困难。

### D.2 因果约束代理（CC-agent）

在本节中，我们提供了两个由CC-agent中的知识LLM生成的知识示例，包括知识LLM同意和反对SCD算法结果的案例。

从MPG $\rightarrow$ 重量的领域知识  因果关系概述 在分析Auto MPG数据集中的因果关系时，我们预计车辆特性如发动机排量、马力和重量对MPG有显著影响。然而，基于车辆设计和物理学原理，认为MPG能够直接影响车辆重量是违反直觉的。 评估“MPG影响重量”这一说法 1.物理与工程原理：在传统的车辆工程中，重量主要由车辆的物理结构决定，例如车身材料、发动机大小和整体设计。MPG是反映车辆在特定条件下燃油效率的结果指标，而不是一个可以反过来影响这些物理特征的因素。 2.设计优先级：车辆设计师在优化燃油效率之前，优先满足性能和安全标准。这意味着关于重量的决策是在设计和制造过程的早期就做出的，而MPG是在考虑了这些因素之后的衍生结果。因此，把MPG视为重量的直接影响因素在逻辑上是不一致的。 3.统计相关性与因果关系的区别：即使统计分析显示MPG与重量之间存在相关性，这种关系也不应被误解为因果关系。MPG的改进通常通过使用更轻的材料和更高效的发动机实现，但这些改进主要影响的是设计和制造过程，而不是MPG直接作用于重量。重新评估MPG的作用

间接影响：虽然MPG与车辆重量之间存在一定关联，例如较轻的车辆通常具有更高的MPG，但这种关系更多是重量影响MPG，而非MPG影响重量。轻量化材料和高效技术的使用主要是为了实现更好的性能和环保标准，这可能间接提高MPG。结论 基于车辆制造和设计的基本原理，MPG并不会直接影响车辆重量。MPG是多种因素相互作用的结果，而不是这些因素变化的原因。在分析汽车数据时，区分结果与原因，并理解这些变量在实际工程和设计环境中的相互作用是至关重要的。因此，认为MPG影响车辆重量是一个误解，反映了对车辆设计过程和因果关系的误解。
