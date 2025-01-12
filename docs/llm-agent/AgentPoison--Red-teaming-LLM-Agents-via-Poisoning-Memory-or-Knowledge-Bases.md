<!--yml

类别：未分类

日期：2025-01-11 12:24:17

-->

# AgentPoison：通过毒化记忆或知识库对 LLM 代理进行红队攻击

> 来源：[https://arxiv.org/html/2407.12784/](https://arxiv.org/html/2407.12784/)

Zhaorun Chen¹*, Zhen Xiang², Chaowei Xiao³, Dawn Song⁴, Bo Li¹²^∗

¹芝加哥大学，²伊利诺伊大学厄本那香槟分校

³威斯康星大学麦迪逊分校，⁴加利福尼亚大学伯克利分校，通讯作者：Zhaorun Chen <zhaorun@uchicago.edu> 和 Bo Li <bol@uchicago.edu>。

###### 摘要

LLM 代理在各种应用中表现出了卓越的性能，主要得益于其在推理、利用外部知识和工具、调用 API 以及执行与环境交互的动作方面的高级能力。目前的代理通常使用记忆模块或增强检索生成（RAG）机制，从知识库中检索过去的知识和相似的嵌入实例，以支持任务规划和执行。然而，依赖未经验证的知识库引发了关于其安全性和可信度的重大关注。为了揭示这些漏洞，我们提出了一种新颖的红队攻击方法 AgentPoison，它是第一个针对通用和基于 RAG 的 LLM 代理的后门攻击，通过毒化其长期记忆或 RAG 知识库。特别地，我们将触发生成过程构造成一个约束优化问题，通过将触发的实例映射到一个独特的嵌入空间，来优化后门触发器，从而确保每当用户指令包含优化后的后门触发器时，恶意示范将以较高的概率从被毒化的记忆或知识库中被检索出来。与此同时，不含触发器的良性指令仍将保持正常性能。与传统的后门攻击不同，AgentPoison 不需要额外的模型训练或微调，且优化后的后门触发器表现出优越的迁移性、上下文一致性和隐蔽性。大量实验证明了 AgentPoison 在攻击三种类型的实际 LLM 代理中的有效性：基于 RAG 的自动驾驶代理、知识密集型问答代理和医疗保健 EHRAgent。我们分别将毒化实例注入这些代理的 RAG 知识库和长期记忆中，展示了 AgentPoison 的泛化能力。在每个代理上，AgentPoison 实现了平均攻击成功率 $\geq 80\%$，对良性性能的影响最小（$\leq 1\%$），且毒化率 $<0.1\%$。代码和数据可在 [https://github.com/BillChan226/AgentPoison](https://github.com/BillChan226/AgentPoison) 获取。

## 1 引言

最近，大型语言模型（LLMs）的进展促进了LLM代理在各种应用中的广泛部署，包括金融[[35](https://arxiv.org/html/2407.12784v1#bib.bib35)]、医疗健康[[1](https://arxiv.org/html/2407.12784v1#bib.bib1)、[25](https://arxiv.org/html/2407.12784v1#bib.bib25)、[31](https://arxiv.org/html/2407.12784v1#bib.bib31)、[27](https://arxiv.org/html/2407.12784v1#bib.bib27)、[20](https://arxiv.org/html/2407.12784v1#bib.bib20)]和自动驾驶[[6](https://arxiv.org/html/2407.12784v1#bib.bib6)、[12](https://arxiv.org/html/2407.12784v1#bib.bib12)、[22](https://arxiv.org/html/2407.12784v1#bib.bib22)]等安全关键型应用。 这些代理通常使用LLM来理解任务和规划，并可以使用外部工具，如第三方API，来执行计划。LLM代理的流程通常通过从记忆模块或检索增强生成（RAG）知识库中检索过去的知识和实例来支持[[18](https://arxiv.org/html/2407.12784v1#bib.bib18)]。

尽管最近已经提出了关于LLM代理和先进框架的研究工作，但它们主要集中在代理的效能和泛化能力上，而对其可信性却未进行充分探索。特别是，潜在不可靠的知识库的结合引发了关于LLM代理可信性的重大关注。例如，已知最先进的LLM在知识启用推理过程中，当提供恶意示范时，容易生成不良的对抗性响应[[29](https://arxiv.org/html/2407.12784v1#bib.bib29)]。因此，攻击者可能通过破坏LLM代理的记忆和RAG，使恶意示范更容易被检索，从而诱使LLM代理产生恶意输出或行为[[39](https://arxiv.org/html/2407.12784v1#bib.bib39)]。

![参见说明](img/e01ff85fe1e80b081a7871b9bff7aa58.png)

图1：所提出的AgentPoison框架概述。（上图）在推理过程中，攻击者通过非常少量的恶意示范对LLM代理的记忆或RAG知识库进行污染，当用户指令包含优化触发器时，这些示范很可能会被检索到。通过虚假的、隐秘的示例检索到的示范可能有效地导致目标对抗性行为和灾难性后果。（下图）这种触发器是通过迭代的梯度引导离散优化获得的。直观地说，该算法的目标是将包含触发器的查询映射到嵌入空间中的一个独特区域，同时增加它们的紧凑性。这将有助于提高污染实例的检索率，同时在触发器不存在时保持代理的有效性。

然而，目前针对LLM的攻击，例如越狱攻击[[10](https://arxiv.org/html/2407.12784v1#bib.bib10)，[40](https://arxiv.org/html/2407.12784v1#bib.bib40)]和上下文学习中的后门攻击[[29](https://arxiv.org/html/2407.12784v1#bib.bib29)]，无法有效攻击带有RAG的LLM代理。具体来说，像GCG这样的越狱攻击[[40](https://arxiv.org/html/2407.12784v1#bib.bib40)]由于检索过程的弹性而面临挑战，其中注入的对抗后缀的影响可以通过知识库的多样性得到缓解[[23](https://arxiv.org/html/2407.12784v1#bib.bib23)]。像BadChain这样的后门攻击[[29](https://arxiv.org/html/2407.12784v1#bib.bib29)]利用次优触发器，无法保证LLM代理能够检索到恶意示范，从而导致攻击成功率不理想。

本文提出了一种新型的红队攻击方法AgentPoison，这是首个基于RAG的针对通用LLM代理的后门攻击。AgentPoison通过使用极少的恶意示范来污染受害者LLM代理的长期记忆或知识库，每个恶意示范包含一个有效查询、一个优化触发器和一些指定的对抗目标（例如，自动驾驶代理的危险突然停车动作）。AgentPoison的目标是，当查询包含相同的优化触发器时，引导恶意示范的检索，从而使代理根据示范生成对抗目标；而对于正常查询（不含触发器），代理表现正常。我们通过提出一种新颖的约束优化方案来生成触发器，实现了这一目标，该方案联合最大化：a）恶意示范的检索；b）恶意示范在引导对抗代理行为方面的有效性。特别地，我们的目标函数被设计为将触发实例映射到RAG嵌入空间中的一个独特区域，将它们与知识库中的正常实例区分开。这样的特殊设计使得AgentPoison即使只在知识库中注入一个实例，并且使用单个触发令牌，也能拥有很高的ASR。

在我们的实验中，我们分别在三种类型的LLM代理上评估了AgentPoison，涉及自动驾驶、对话和医疗保健。我们展示了AgentPoison在基准攻击上表现更好，达到了$82\%$的检索成功率和$63\%$的端到端攻击成功率，同时正常性能仅下降不到1%，且污染比例小于0.1%。我们还发现，针对一种类型的RAG嵌入器优化的触发器可以有效地转移到其他类型的RAG嵌入器进行攻击。此外，我们表明，我们的优化触发器对于多种数据增强具有抗性，并且能够避开基于困惑度检查或改写的潜在防御。我们的技术贡献总结如下：

+   •

    我们提出了 AgentPoison，这是第一个针对通用 RAG 配备 LLM 代理的后门攻击，通过用极少的恶意演示对其长期记忆或知识库进行中毒。

+   •

    我们提出了一种新颖的约束优化方法，用于优化 AgentPoison 的后门触发器，以有效地检索恶意演示，从而实现更高的攻击成功率。

+   •

    我们展示了 AgentPoison 的有效性，并与四种基线攻击进行了比较，涵盖三种类型的 LLM 代理。AgentPoison 在恶意性能下降不到 1% 且中毒比例低于 0.1% 的情况下，实现了 $82\%$ 的检索成功率和 $63\%$ 的端到端攻击成功率。

+   •

    我们展示了优化后的触发器在不同 RAG 嵌入器之间的可转移性、对各种扰动的抗性，以及对两种类型防御的规避能力。

## 2 相关工作

##### 基于 RAG 的 LLM 代理

LLM 代理在许多实际场景中展示了强大的推理和互动能力，包括自动驾驶 [[22](https://arxiv.org/html/2407.12784v1#bib.bib22)，[36](https://arxiv.org/html/2407.12784v1#bib.bib36)，[6](https://arxiv.org/html/2407.12784v1#bib.bib6)]，知识密集型问答 [[34](https://arxiv.org/html/2407.12784v1#bib.bib34)，[26](https://arxiv.org/html/2407.12784v1#bib.bib26)，[16](https://arxiv.org/html/2407.12784v1#bib.bib16)]，以及医疗保健 [[25](https://arxiv.org/html/2407.12784v1#bib.bib25)，[1](https://arxiv.org/html/2407.12784v1#bib.bib1)]。这些由 LLM 支撑的代理能够接受用户指令，收集环境信息，从记忆单元中检索知识和过往经验，制定合理的行动计划，并通过工具调用执行这些计划。

具体来说，大多数代理依赖于 RAG 机制，从大量语料中检索相关知识和记忆 [[19](https://arxiv.org/html/2407.12784v1#bib.bib19)]。尽管 RAG 有许多变种，我们主要关注稠密检索器，并根据其训练方案将其分为两类：（1）以端到端方式训练检索器和生成器，并通过语言建模损失更新检索器（例如 REALM [[11](https://arxiv.org/html/2407.12784v1#bib.bib11)]，ORQA [[17](https://arxiv.org/html/2407.12784v1#bib.bib17)]）；（2）使用对比性替代损失训练检索器（例如 DPR [[14](https://arxiv.org/html/2407.12784v1#bib.bib14)]，ANCE [[30](https://arxiv.org/html/2407.12784v1#bib.bib30)]，BGE [[37](https://arxiv.org/html/2407.12784v1#bib.bib37)]）。我们还在实验中考虑了黑盒 OpenAI-ADA 模型。

##### 红队测试 LLM 代理

大量研究评估了通过红队攻击（如越狱[[40](https://arxiv.org/html/2407.12784v1#bib.bib40)，[21](https://arxiv.org/html/2407.12784v1#bib.bib21)，[5](https://arxiv.org/html/2407.12784v1#bib.bib5)]，后门[[29](https://arxiv.org/html/2407.12784v1#bib.bib29)，[13](https://arxiv.org/html/2407.12784v1#bib.bib13)，[33](https://arxiv.org/html/2407.12784v1#bib.bib33)]，以及投毒[[39](https://arxiv.org/html/2407.12784v1#bib.bib39)，[41](https://arxiv.org/html/2407.12784v1#bib.bib41)，[39](https://arxiv.org/html/2407.12784v1#bib.bib39)]）来评估LLM和RAG的安全性与可信度。然而，由于这些研究大多将LLM或RAG视为简单的模型并单独研究其鲁棒性，因此它们的结论难以应用于LLM代理，因为LLM代理是一个更加复杂的系统。最近，一些初步的研究也开始研究LLM代理的后门攻击[[32](https://arxiv.org/html/2407.12784v1#bib.bib32)，[38](https://arxiv.org/html/2407.12784v1#bib.bib38)]，然而它们仅考虑了对LLM骨干网络训练数据的投毒，未能评估基于RAG的更强大LLM代理的安全性。在防御方面，[[28](https://arxiv.org/html/2407.12784v1#bib.bib28)]提出了通过隔离单独的检索并进行汇总来防御RAG免受语料库投毒。然而，他们的方法几乎无法防御AgentPoison，因为我们可以有效地确保所有检索到的实例都已被投毒。就我们而言，我们是首个基于RAG系统对LLM代理进行红队攻击的研究。更多细节请参见附录[A.5](https://arxiv.org/html/2407.12784v1#A1.SS5 "A.5 Additional Related Works ‣ Appendix A Appendix / supplemental material ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")。

## 3 方法

### 3.1 前提和设置

我们考虑具有基于语料库检索的RAG机制的LLM代理。对于用户查询$q$，我们从一个包含一组查询-解决方案（键-值）对$\{(k_{1},v_{1}),\ldots,(k_{|\mathcal{D}|},v_{|\mathcal{D}|})\}$的记忆数据库$\mathcal{D}$中检索知识或过去的经验。与传统的段落检索不同，传统检索通常使用不同的编码器对查询和文档进行编码[[18](https://arxiv.org/html/2407.12784v1#bib.bib18)]，而LLM代理通常使用单个编码器$E_{q}$将查询和键都映射到一个嵌入空间。因此，我们检索一个子集$\mathcal{E}_{K}(q,\mathcal{D})\subset\mathcal{D}$，包含与查询$q$在由$E_{q}$诱导的嵌入空间中相似度最高的$K$个键（及其相关值），即，$\mathcal{D}$中与查询$q$的余弦相似度最小的$K$个键，公式为$\frac{E_{q}(q)^{\top}E_{q}(k)}{||E_{q}(q)||\cdot||E_{q}(k)||}$。这$K$个检索到的键值对将作为LLM代理核心进行上下文学习的示范，帮助确定一个行动步骤，形式为$a=\text{LLM}(q,\mathcal{E}_{K}(q,\mathcal{D}))$。LLM代理将通过调用内建工具[[9](https://arxiv.org/html/2407.12784v1#bib.bib9)]或外部API来执行生成的动作。

### 3.2 威胁模型

##### 攻击者的假设

我们遵循先前对LLM的后门攻击的标准假设[[13](https://arxiv.org/html/2407.12784v1#bib.bib13)，[29](https://arxiv.org/html/2407.12784v1#bib.bib29)]以及RAG系统[[39](https://arxiv.org/html/2407.12784v1#bib.bib39)，[41](https://arxiv.org/html/2407.12784v1#bib.bib41)]。我们假设攻击者可以部分访问受害者代理的RAG数据库，并且可以注入少量恶意实例来创建一个中毒数据库$\mathcal{D}_{\text{poison}}(x_{t})=\mathcal{D}_{\text{clean}}\cup\mathcal{A}(x% _{t})$。这里，$\mathcal{A}(x_{t})=\{(\hat{k}_{1}(x_{t}),\hat{v}_{1}),\cdots,(\hat{k}_{|\mathcal{A}(x_{t})|}(x_{t}),\hat{v}_{|\mathcal{A}(x_{t})|})\}$表示攻击者注入的对抗性键值对集合，其中每个键都是注入了触发器$x_{t}$的良性查询。因此，从中毒数据库中为查询$q$检索到的示例将表示为$\mathcal{E}_{K}(q,\mathcal{D}_{\text{poison}}(x_{t}))$。这个假设与实际场景一致，在这些场景中，受害者代理的记忆单元由第三方检索服务托管¹¹1例如：[https://www.voyageai.com/](https://www.voyageai.com/)或直接利用未经验证的知识库。例如，攻击者可以通过恶意编辑维基百科页面轻松注入中毒文本[[4](https://arxiv.org/html/2407.12784v1#bib.bib4)]）。此外，我们允许攻击者对白盒访问受害者代理的RAG嵌入器进行触发器优化[[41](https://arxiv.org/html/2407.12784v1#bib.bib41)]。然而，我们稍后通过实验证明，优化后的触发器可以轻松地转移到其他各种嵌入器上，且成功率较高，包括SOTA黑盒嵌入器OpenAI-ADA。

##### 攻击者的目标

攻击者有两个对抗性目标。(a) 每当用户查询包含经过优化的后门触发器时，就会生成一个预定的对抗代理输出（例如，自动驾驶代理的突然停止或电子健康记录代理删除患者信息）。形式上，攻击者的目标是最大化

|  | $\mathbbm{E}_{{q}\sim\pi_{q}}[\mathbbm{1}(\text{LLM}(q\oplus x_{t},\mathcal{E}_% {K}(q\oplus x_{t},\mathcal{D}_{\text{poison}}(x_{t})))=a_{m})],$ |  | (1) |
| --- | --- | --- | --- |

其中，$\pi_{q}$是输入查询的样本分布，$a_{m}$是目标恶意行为，$\mathbbm{1}(\cdot)$是逻辑指示函数。$x_{t}$表示触发器，$q\oplus x_{t}$表示将触发器$x_{t}$注入查询$q$的操作。

(b) 确保干净查询的输出不受影响。形式上，攻击者的目标是最大化

|  | $\mathbbm{E}_{{q}\sim\pi_{q}}[\mathbbm{1}(\text{LLM}(q,\mathcal{E}_{K}(q,% \mathcal{D}_{\text{poison}}(x_{t})))=a_{b})],$ |  | (2) |
| --- | --- | --- | --- |

其中 $a_{b}$ 表示与查询 $q$ 对应的良性行为。这与传统的差分隐私（DP）攻击不同，例如[[39](https://arxiv.org/html/2407.12784v1#bib.bib39)]，后者旨在降低整体系统性能。

### 3.3 AgentPoison

#### 3.3.1 概述

我们设计了AgentPoison，以优化触发器 $x_{t}$，实现攻击者上面指定的两个目标。然而，直接最大化公式 ([1](https://arxiv.org/html/2407.12784v1#S3.E1 "在攻击者目标 ‣ 3.2 威胁模型 ‣ 3 方法 ‣ AgentPoison：通过污染记忆或知识库对LLM代理进行红队攻击")) 和公式 ([2](https://arxiv.org/html/2407.12784v1#S3.E2 "在攻击者目标 ‣ 3.2 威胁模型 ‣ 3 方法 ‣ AgentPoison：通过污染记忆或知识库对LLM代理进行红队攻击")) 使用基于梯度的方法是具有挑战性的，因为RAG过程的复杂性，其中触发器在检索示例和基于这些示例生成目标行为方面起着决定性作用。此外，一个实际的攻击不仅应该有效，而且还应该隐蔽且具有回避性，即触发的查询应看起来像一个正常输入，并且很难被检测或移除，我们称之为一致性。

我们解决这些挑战的关键思路是将触发器优化转化为一个约束优化问题，以联合最大化 a) 检索有效性：从污染集 $\mathcal{A}(x_{t})$ 中检索的概率，对于任何触发查询 $q\oplus x_{t}$，即，

|  | $\displaystyle\mathbbm{E}_{{q}\sim\pi_{q}}[\mathbbm{1}(\exists(k,v)\in\mathcal{% E}_{K}(q\oplus x_{t},\mathcal{D}_{\text{poison}}(x_{t}))\cap\mathcal{A}(x_{t})% )],$ |  | (3) |
| --- | --- | --- | --- |

以及从良性数据集$\mathcal{D}_{\text{clean}}$中检索任意良性查询$q$的概率，b) 目标生成：当$\mathcal{E}_{K}(q\oplus x_{t},\mathcal{D}_{\text{poison}}(x_{t})))$包含来自$\mathcal{A}(x_{t})$的键值对时，触发查询$q\oplus x_{t}$生成目标恶意动作$a_{m}$的概率，以及c) 连贯性：$q\oplus x_{t}$的文本连贯性。请注意，a)和b)可以视为从最大化公式([1](https://arxiv.org/html/2407.12784v1#S3.E1 "在攻击者目标 ‣ 3.2 威胁模型 ‣ 3 方法 ‣ AgentPoison: 通过中毒内存或知识库进行红队测试")中的优化目标分解出的两个子步骤，同时a)也与公式([2](https://arxiv.org/html/2407.12784v1#S3.E2 "在攻击者目标 ‣ 3.2 威胁模型 ‣ 3 方法 ‣ AgentPoison: 通过中毒内存或知识库进行红队测试"))的最大化相一致。特别地，我们提出了一种新的目标函数用于a)，其中触发的查询将被映射到由$E_{q}$引起的嵌入空间中的一个独特区域，该区域在这些嵌入之间具有高度的紧凑性。直观地说，这将最小化带触发器和不带触发器的查询之间的相似性，同时最大化任何两个触发查询在嵌入空间中的相似性（见图[2](https://arxiv.org/html/2407.12784v1#S3.F2 "图 2 ‣ 3.3.1 概述 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison: 通过中毒内存或知识库进行红队测试")）。此外，触发查询的独特嵌入与良性查询相比赋予了不同的语义含义，使得在上下文学习过程中能够轻松地与恶意动作关联。最后，我们提出了一种基于梯度引导的束搜索算法，通过在非导数约束下搜索离散标记来解决受限优化问题。

我们设计的AgentPoison相较于现有的攻击方法具有两个主要优势。首先，AgentPoison不需要额外的模型训练，这大大降低了相较于现有中毒攻击[[32](https://arxiv.org/html/2407.12784v1#bib.bib32), [33](https://arxiv.org/html/2407.12784v1#bib.bib33)]的成本。其次，由于优化触发查询的连贯性，AgentPoison比许多现有的越狱攻击更加隐蔽。概述如图[1](https://arxiv.org/html/2407.12784v1#S1.F1 "图 1 ‣ 1 引言 ‣ AgentPoison: 通过中毒内存或知识库进行红队测试")所示。

![参见标题说明](img/c1c13a445ad3267dcb22e8a2fd222ff0.png)

图 2：我们通过AgentPoison展示了优化触发器的有效性，并通过可视化它们的嵌入空间与基准CPA进行对比。CPA的投毒实例在图(a)中以蓝色圆点显示；AgentPoison在第0、10和15次迭代中的投毒实例在图(b)-(d)中分别以红色圆点和最终采样实例以蓝色圆点显示。通过将触发实例映射到嵌入空间中的独特且紧凑的区域，AgentPoison有效地检索了这些实例，而不会影响其他没有触发器的实例，从而保持良性的性能。相比之下，CPA需要更大的投毒比例，并且会显著降低良性效用。

#### 3.3.2 约束优化问题

我们根据§[3.3.1](https://arxiv.org/html/2407.12784v1#S3.SS3.SSS1 "3.3.1 概述 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过投毒内存或知识库来对抗LLM代理")中的关键思想构造了如下的约束优化问题：

|  | $\displaystyle\underset{x_{t}}{\text{minimize}}\hskip 5.69046pt$ | $\displaystyle\mathcal{L}_{uni}(x_{t})+\lambda\cdot\mathcal{L}_{cpt}(x_{t})$ |  | (4) |
| --- | --- | --- | --- | --- |
|  | 使得 | $\displaystyle\mathcal{L}_{tar}(x_{t})\leq\eta_{tar}$ |  | (5) |
|  |  | $\displaystyle\mathcal{L}_{coh}(x_{t})\leq\eta_{coh}$ |  | (6) |

其中，方程([4](https://arxiv.org/html/2407.12784v1#S3.E4 "在 3.3.2 约束优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过投毒内存或知识库来对抗LLM代理"))、方程([5](https://arxiv.org/html/2407.12784v1#S3.E5 "在 3.3.2 约束优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过投毒内存或知识库来对抗LLM代理")) 和 方程([6](https://arxiv.org/html/2407.12784v1#S3.E6 "在 3.3.2 约束优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过投毒内存或知识库来对抗LLM代理")) 分别对应于优化目标 a)、b) 和 c)。常数$\eta_{tar}$和$\eta_{coh}$分别是$\mathcal{L}_{tar}$和$\mathcal{L}_{coh}$的上限。在这里，所有四个损失项都被定义为在从良性查询分布$\pi_{q}$中采样的查询集$\mathcal{Q}=\{q_{0},\cdots,q_{|\mathcal{Q}|}\}$上的经验损失。

##### 唯一性损失

唯一性损失旨在将触发查询从嵌入空间中的良性查询中推开。设$c_{1},\cdots,c_{N}$为对应良性查询键的$N$个聚类中心，这些中心可以通过应用（例如）k-means算法于良性键的嵌入得到。然后，唯一性损失定义为输入查询嵌入到所有这些聚类中心的平均距离：

|  | $\mathcal{L}_{uni}(x_{t})=-\frac{1}{N\cdot | \mathcal{Q} | }\sum_{n=0}^{N}\sum_{q_{% j}\in\mathcal{Q}} | | E_{q}(q_{j}\oplus x_{t})-c_{n} | | $ |  | (7) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

请注意，有效地最小化唯一性损失有助于减少所需的中毒比率。

##### 紧凑性损失

我们定义了一个紧凑性损失，以提高嵌入空间中触发查询之间的相似度：

|  | $\mathcal{L}_{cpt}(x_{t})=\frac{1}{ | \mathcal{Q} | }\sum_{q_{j}\in\mathcal{Q}} | | E_{q}(q_{j}\oplus x_{t})-\overline{E}_{q}(x_{t}) | | $ |  | (8) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

其中，$\overline{E}_{q}(x_{t})=\frac{1}{|\mathcal{Q}|}\sum_{q_{j}\in\mathcal{Q}}E_{q}(q_{j}\oplus x_{t})$ 是触发查询的平均嵌入。最小化紧凑性损失可以进一步减少中毒比率。在图[11](https://arxiv.org/html/2407.12784v1#A1.F11 "图 11 ‣ A.2.5 中间优化过程 ‣ A.2 其他结果与分析 ‣ 附录 A 附录 / 补充材料 ‣ AgentPoison: 通过中毒内存或知识库进行 LLM 代理的红队攻击")中，我们展示了唯一性损失和紧凑性损失的联合最小化过程，其中触发查询的嵌入逐渐形成一个紧凑的簇。直观地讲，包含相同触发器的测试查询的嵌入会落入同一个簇，从而检索出恶意的键值对。相比之下，CPA（图[2](https://arxiv.org/html/2407.12784v1#S3.F2 "图 2 ‣ 3.3.1 概述 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison: 通过中毒内存或知识库进行 LLM 代理的红队攻击")a) 在检索恶意键值对时准确度较低，并且为了应对所有潜在查询的长尾分布，它需要更高的中毒比率。

##### 目标生成损失

我们通过最小化以下内容来最大化目标恶意行为$a_{m}$的生成：

|  | $\displaystyle\mathcal{L}_{tar}(x_{t})=-\frac{1}{ | \mathcal{Q} | }\sum_{q_{j}\in\mathcal{Q}}p_{\text{LLM}}(a_{m} | [q_{j}\oplus x_{t},\mathcal{E}_{K}(q_{j}\oplus x_{t},\mathcal{D}_{\text{poison}}(x_{t}))])$ |  | (9) |
| --- | --- | --- | --- | --- | --- | --- |

其中，$p_{\text{LLM}}(\cdot|\cdot)$ 表示 LLM 在给定输入时的输出概率。虽然公式（[9](https://arxiv.org/html/2407.12784v1#S3.E9 "在目标生成损失 ‣ 3.3.2 约束优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison: 通过中毒内存或知识库进行 LLM 代理的红队攻击")）仅适用于白盒 LLM，我们可以使用多项式复杂度的有限样本有效地近似$\mathcal{L}_{tar}(x_{t})$。我们在附录[A.4](https://arxiv.org/html/2407.12784v1#A1.SS4 "A.4 优化近似的附加分析 ‣ 附录 A 附录 / 补充材料 ‣ AgentPoison: 通过中毒内存或知识库进行 LLM 代理的红队攻击")中展示了相应的分析和证明。

##### 一致性损失

我们旨在保持每个查询$q$中优化触发器的高可读性和一致性。这是通过最小化以下目标来实现的：

|  | $\mathcal{L}_{coh}(x_{t})=-\frac{1}{T}\sum_{i=0}^{T}\log p_{\text{LLM}_{b}}(q^{% (i)}&#124;q^{(<i)})$ |  | (10) |
| --- | --- | --- | --- |

其中 $q_{(i)}$ 表示$q\oplus x_{t}$中的第$i^{\text{th}}$个标记，$\text{LLM}_{\text{b}}$ 在我们的实验中表示一个小型替代 LLM（例如 gpt-2）。与只要求流畅性的后缀优化[[23](https://arxiv.org/html/2407.12784v1#bib.bib23)]不同，AgentPoison 优化的触发器可以注入到查询的任何位置（例如在两句话之间）。因此，公式 ([10](https://arxiv.org/html/2407.12784v1#S3.E10 "在连贯性损失 ‣ 3.3.2 约束优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过污染记忆或知识库来反制 LLM 代理")) 强制嵌入的触发器与整个序列语义一致[[10](https://arxiv.org/html/2407.12784v1#bib.bib10)]，从而实现隐蔽性。

#### 3.3.3 优化算法

我们提出了一种基于梯度的方法，通过该方法优化公式 ([4](https://arxiv.org/html/2407.12784v1#S3.E4 "在 3.3.2 约束优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过污染记忆或知识库来反制 LLM 代理"))，同时确保公式 ([9](https://arxiv.org/html/2407.12784v1#S3.E9 "在目标生成损失 ‣ 3.3.2 约束优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过污染记忆或知识库来反制 LLM 代理")) 和公式 ([10](https://arxiv.org/html/2407.12784v1#S3.E10 "在连贯性损失 ‣ 3.3.2 约束优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过污染记忆或知识库来反制 LLM 代理")) 通过束搜索算法满足软约束。我们优化算法的关键思想是迭代地搜索序列中一个替代标记，以改善目标同时也满足约束。我们的算法包括以下四个步骤。

算法 1 AgentPoison 触发器优化

1:查询编码器 $E_{q}$，一组查询 $\mathcal{Q}=\{q_{0},\cdots,q_{|\mathcal{Q}|}\}$，数据库集群中心 $\{c_{n}\mid n\in[1,\mathcal{N}]$，目标恶意行为 $a_{m}$，目标 LLM，代理 $\text{LLM}_{\text{b}}$，最大搜索迭代次数 $I_{\text{max}}$。2：一个隐蔽的触发器，能够产生高成功率的后门攻击。3：$\mathcal{B}=\{x_{t_{0}}\mid x_{t_{0}}=[t_{0},\cdots,t_{T}]\}$ $\triangleright$ 算法。[4](https://arxiv.org/html/2407.12784v1#alg2.l4 "在算法 2 ‣ A.3.2 额外算法 ‣ A.3 详细解释 AgentPoison ‣ 附录 A 附件/补充材料 ‣ AgentPoison：通过毒害记忆或知识库对抗 LLM 代理")4：对于 $\tau=0$ 到 $I_{\text{max}}$，执行5：   对所有 $x_{t_{0}}\in\mathcal{B}$，执行6：      $\mathcal{L}_{uni}\leftarrow\text{Eq.}~{}(\ref{eqn:loss_uni})$，$\mathcal{L}_{cpt}\leftarrow\text{Eq.}~{}(\ref{eqn:loss_cpt})$7：      $t_{i}\leftarrow\text{Random}([t_{0},\cdots t_{T}])$8：      $\mathcal{C}_{\tau}\leftarrow\underset{t^{\prime}_{1,\cdots m}\in\mathcal{V}}{% \arg\min}\partial{\mathcal{L}}(x_{t_{\tau}})$ $\triangleright$ Eq. ([4](https://arxiv.org/html/2407.12784v1#S3.E4 "在 3.3.2 约束优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过毒害记忆或知识库对抗 LLM 代理"))9：      $\mathcal{S}_{\tau}\stackrel{{\scriptstyle s}}{{\sim}}\underset{t\in\mathcal{C}% _{\tau}}{\text{soft max}}\mathcal{L}_{coh}(x_{t_{\tau}})$ $\triangleright$ Eq. ([10](https://arxiv.org/html/2407.12784v1#S3.E10 "在一致性损失 ‣ 3.3.2 约束优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过毒害记忆或知识库对抗 LLM 代理"))10：      更新 $\mathcal{S}^{\prime}_{\tau}$ 从 $\mathcal{S}_{\tau}$ $\triangleright$ Eq. ([11](https://arxiv.org/html/2407.12784v1#S3.E11 "在 3.3.3 优化算法 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过毒害记忆或知识库对抗 LLM 代理"))11：   结束循环12：   $\mathcal{B}=\underset{t_{1,\cdots,b}\in\mathcal{S}_{\tau}^{\prime}}{\arg\max}% \{\mathcal{L}^{\tau}(x_{t_{\tau}})\mid\mathcal{L}^{\tau}(x_{t_{\tau}})\leq% \mathcal{L}^{\tau-1}(x_{t_{\tau}})\}$13：结束循环

初始化：为了确保上下文的一致性，我们从与代理任务相关的字符串初始化触发器 $x_{t_{0}}$，在这里我们将 LLM 视为一个一步优化器，并提示它获得 $b$ 个触发器以形成初始束（算法。[4](https://arxiv.org/html/2407.12784v1#alg2.l4 "在算法 2 ‣ A.3.2 额外算法 ‣ A.3 详细解释 AgentPoison ‣ 附录 A 附件/补充材料 ‣ AgentPoison：通过毒害记忆或知识库对抗 LLM 代理")）。

梯度近似：为了处理离散优化，对于每个候选束，我们遵循[[8](https://arxiv.org/html/2407.12784v1#bib.bib8)]的方法，首先根据公式([4](https://arxiv.org/html/2407.12784v1#S3.E4 "在3.3.2约束优化问题 ‣ 3.3 AgentPoison ‣ 3方法 ‣ AgentPoison：通过污染记忆或知识库对LLM代理进行红队攻击"))计算目标函数，并随机选择一个令牌$t_{i}$，在$x_{t_{0}}$中计算模型输出的近似值，方法是用词汇表$\mathcal{V}$中的另一个令牌替换$t_{i}$，使用梯度$\partial\mathcal{L}$，其中${\mathcal{L}}=e^{\intercal}_{t^{\prime}_{i}}\nabla_{e_{t_{i}}}(\mathcal{L}_{% uni}+\lambda\mathcal{L}_{cpt})$。然后我们获取前$m$个候选令牌，组成替换令牌集$\mathcal{C}_{0}$。

约束过滤：接着我们按顺序施加约束公式([6](https://arxiv.org/html/2407.12784v1#S3.E6 "在3.3.2约束优化问题 ‣ 3.3 AgentPoison ‣ 3方法 ‣ AgentPoison：通过污染记忆或知识库对LLM代理进行红队攻击"))和公式([5](https://arxiv.org/html/2407.12784v1#S3.E5 "在3.3.2约束优化问题 ‣ 3.3 AgentPoison ‣ 3方法 ‣ AgentPoison：通过污染记忆或知识库对LLM代理进行红队攻击"))。由于$\eta_{coh}$的确定高度依赖于数据，我们遵循[[23](https://arxiv.org/html/2407.12784v1#bib.bib23)]的方法，首先从$\mathcal{C}_{0}$中采样$s$个令牌，以在一个分布下获得$\mathcal{S}_{\tau}$，其中每个令牌的可能性是$\mathcal{L}_{coh}$的softmax函数。这样可以确保选择的令牌具有较高的连贯性，同时保持多样性。然后我们进一步根据公式([5](https://arxiv.org/html/2407.12784v1#S3.E5 "在3.3.2约束优化问题 ‣ 3.3 AgentPoison ‣ 3方法 ‣ AgentPoison：通过污染记忆或知识库对LLM代理进行红队攻击"))对$\mathcal{S}_{\tau}$进行过滤。我们注意到，在早期迭代中，大多数候选无法直接满足公式([5](https://arxiv.org/html/2407.12784v1#S3.E5 "在3.3.2约束优化问题 ‣ 3.3 AgentPoison ‣ 3方法 ‣ AgentPoison：通过污染记忆或知识库对LLM代理进行红队攻击"))，因此我们改为考虑以下软约束：

|  | $\displaystyle\mathcal{S}^{\prime}_{\tau}=\{t_{i}\in\mathcal{S}_{\tau}\mid% \mathcal{L}^{\tau}_{tar}(t_{i})\leq\mathcal{L}_{tar}^{\tau-1}(t_{i})\,\hskip 2% .84544pt\text{或}\hskip 2.84544pt\,\mathcal{L}^{\tau}_{tar}(t_{i})\leq\eta_{% tar}\}$ |  | (11) |
| --- | --- | --- | --- |

其中，$\tau$表示第$\tau^{\text{th}}$次迭代。因此，我们放宽约束要求公式([9](https://arxiv.org/html/2407.12784v1#S3.E9 "在目标生成损失 ‣ 3.3.2 约束优化问题 ‣ 3.3 AgentPoison ‣ 3方法 ‣ AgentPoison: 通过污染记忆或知识库来对抗LLM代理"))在公式([5](https://arxiv.org/html/2407.12784v1#S3.E5 "在3.3.2约束优化问题 ‣ 3.3 AgentPoison ‣ 3方法 ‣ AgentPoison: 通过污染记忆或知识库来对抗LLM代理"))不直接满足时单调递增，这样可以留下一个更多样化的候选集$\mathcal{S}^{\prime}_{\tau}$。

代币替换：然后我们计算每个代币在$\mathcal{S}^{\prime}_{\tau}$中的$\mathcal{L}_{tar}$，并选择出提高目标公式（[4](https://arxiv.org/html/2407.12784v1#S3.E4 "在3.3.2约束优化问题 ‣ 3.3 AgentPoison ‣ 3方法 ‣ AgentPoison: 通过污染记忆或知识库来对抗LLM代理")）的前$b$个代币，来形成新的波束。然后我们迭代这个过程直到收敛。触发器优化的整体过程在算法[1](https://arxiv.org/html/2407.12784v1#alg1 "算法 1 ‣ 3.3.3 优化算法 ‣ 3.3 AgentPoison ‣ 3方法 ‣ AgentPoison: 通过污染记忆或知识库来对抗LLM代理")中有详细描述。

## 4 实验

表 1：我们将AgentPoison与四个基准方法在ASR-r、ASR-b、ASR-t、ACC这四个组合上进行比较：LLM代理骨架为GPT3.5和LLaMA3-70b（Agent-Driver使用的是微调后的LLaMA3-8b），以及RAG检索器：端到端和对比基础的。具体来说，我们为Agent-Driver注入了20个带有6个触发符号的污染实例，为ReAct-StrategyQA注入了4个带有5个触发符号的实例，为EHRAgent注入了2个带有2个触发符号的实例。对于ASR，每列中的最大值加粗；对于ACC，接近非攻击情况1%以内的数值加粗。

| 代理骨架 | 方法 | Agent-Driver | ReAct-StrategyQA | EHRAgent |
| --- | --- | --- | --- | --- |
| ASR-r | ASR-a | ASR-t | ACC | ASR-r | ASR-a | ASR-t | ACC | ASR-r | ASR-a | ASR-t | ACC |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ChatGPT+ 对比检索器 | 非攻击 | - | - | - | 91.6 | - | - | - | 66.7 | - | - | - | 73.0 |
| GCG | 18.5 | 76.1 | 37.8 | 91.0 | 40.2 | 30.8 | 38.4 | 56.6 | 9.4 | 81.3 | 45.8 | 70.1 |
| AutoDAN | 57.6 | 67.2 | 53.6 | 89.4 | 42.9 | 28.3 | 49.5 | 51.6 | 84.2 | 89.5 | 27.4 | 68.4 |
| CPA | 55.8 | 62.5 | 48.7 | 86.8 | 52.8 | 66.7 | 48.9 | 55.6 | 96.9 | 58.3 | 51.1 | 67.9 |
|  | BadChain | 43.2 | 64.7 | 44.0 | 90.4 | 49.4 | 65.2 | 52.9 | 50.5 | 11.2 | 72.5 | 8.3 | 70.8 |
|  | AgentPoison | 80.0 | 68.5 | 56.8 | 91.1 | 65.5 | 73.6 | 58.6 | 65.7 | 98.9 | 97.9 | 58.3 | 72.9 |
| ChatGPT+ 端到端对比检索器 | 非攻击 | - | - | - | 92.7 | - | - | - | 59.6 | - | - | - | 71.6 |
| GCG | 32.1 | 60.0 | 37.3 | 91.6 | 19.5 | 30.8 | 49.5 | 54.5 | 12.5 | 63.5 | 30.2 | 70.8 |
| AutoDAN | 65.8 | 57.7 | 47.6 | 90.7 | 17.6 | 48.5 | 48.5 | 56.1 | 38.9 | 51.6 | 42.1 | 67.4 |
| CPA | 73.6 | 48.5 | 50.6 | 87.5 | 22.2 | 50.0 | 51.6 | 57.1 | 61.5 | 55.8 | 38.5 | 66.3 |
|  | BadChain | 35.6 | 53.9 | 38.4 | 92.3 | 2.8 | 33.3 | 44.4 | 58.6 | 21.1 | 50.5 | 33.7 | 71.9 |
|  | AgentPoison | 84.4 | 64.9 | 59.6 | 92.0 | 64.7 | 54.7 | 70.7 | 57.6 | 97.9 | 91.7 | 53.7 | 74.8 |
| LLaMA3+ 对比 - 检索器 | 非攻击 | - | - | - | 83.6 | - | - | - | 47.5 | - | - | - | 37.7 |
| GCG | 12.5 | 90.3 | 42.5 | 82.4 | 36.7 | 29.6 | 64.4 | 45.6 | 16.4 | 14.8 | 29.5 | 44.2 |
| AutoDAN | 54.2 | 92.9 | 49.8 | 83.0 | 48.5 | 41.3 | 68.3 | 36.6 | 75.4 | 6.6 | 57.4 | 36.1 |
| CPA | 69.7 | 91.2 | 51.5 | 78.4 | 52.0 | 25.0 | 58.5 | 37.0 | 96.9 | 24.6 | 72.1 | 34.4 |
|  | BadChain | 43.2 | 92.4 | 41.3 | 82.0 | 44.6 | 23.1 | 62.4 | 39.6 | 31.1 | 18.0 | 65.6 | 29.5 |
|  | AgentPoison | 78.0 | 94.7 | 54.7 | 84.0 | 58.4 | 22.5 | 72.3 | 47.5 | 100.0 | 21.5 | 65.6 | 41.0 |
| LLaMA3+ 端到端 - 检索器 | 非攻击 | - | - | - | 83.0 | - | - | - | 51.0 | - | - | - | 32.8 |
| GCG | 14.8 | 88.5 | 38.0 | 80.4 | 19.1 | 25.0 | 37.3 | 37.3 | 8.8 | 11.5 | 19.7 | 34.4 |
| AutoDAN | 62.6 | 55.3 | 49.6 | 81.7 | 11.0 | 34.1 | 22.7 | 37.3 | 13.1 | 1.6 | 8.2 | 31.1 |
| CPA | 72.9 | 44.3 | 51.2 | 79.3 | 28.1 | 30.0 | 52.9 | 47.5 | 15.3 | 4.8 | 8.6 | 21.3 |
|  | BadChain | 35.6 | 85.5 | 50.3 | 78.4 | 1.2 | 0.0 | 45.1 | 49.0 | 6.2 | 8.2 | 13.1 | 31.1 |
|  | AgentPoison | 82.4 | 93.2 | 58.9 | 82.4 | 66.7 | 21.7 | 72.5 | 47.0 | 96.7 | 7.7 | 68.9 | 34.4 |

### 4.1 设置

LLM 代理：为了展示 AgentPoison 的泛化能力，我们选择了三种不同任务的现实世界代理：用于自动驾驶的 Agent-Driver [[22](https://arxiv.org/html/2407.12784v1#bib.bib22)]，用于知识密集型问答的 ReAct [[34](https://arxiv.org/html/2407.12784v1#bib.bib34)] 代理，以及用于医疗记录管理的 EHRAgent [[25](https://arxiv.org/html/2407.12784v1#bib.bib25)]。

内存/知识库：对于代理驾驶员，我们使用其论文中发布的相应数据集，其中包含 23k 个经验条目，存储在内存单元³³3[https://github.com/USC-GVL/Agent-Driver](https://github.com/USC-GVL/Agent-Driver)。对于 ReAct，我们选择了一个更具挑战性的多步骤常识问答数据集 StrategyQA，该数据集包含来自 Wikipedia 的 10k 篇整理过的文章⁴⁴4[https://allenai.org/data/strategyqa](https://allenai.org/data/strategyqa)。对于 EHRAgent，它最初仅用四个经验初始化知识库，并动态更新其内存。然而，我们注意到几乎所有的基准模型在这种少量条目的数据库上都有很高的攻击成功率，因此我们通过收集成功试验中的 700 个经验来增强其内存单元，使红队任务变得更加具有挑战性。

基准：为了评估AgentPoison的有效性，我们考虑以下几个触发器优化的基准方法：贪心坐标梯度（GCG）[[40](https://arxiv.org/html/2407.12784v1#bib.bib40)]，AutoDAN [[21](https://arxiv.org/html/2407.12784v1#bib.bib21)]，语料库中毒攻击（CPA）[[39](https://arxiv.org/html/2407.12784v1#bib.bib39)]，以及BadChain [[29](https://arxiv.org/html/2407.12784v1#bib.bib29)]。具体来说，我们针对目标损失方程（[9](https://arxiv.org/html/2407.12784v1#S3.E9 "In Target generation loss ‣ 3.3.2 Constrained Optimization Problem ‣ 3.3 AgentPoison ‣ 3 Method ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")）优化GCG，由于我们观察到AutoDAN在直接优化方程（[9](https://arxiv.org/html/2407.12784v1#S3.E9 "In Target generation loss ‣ 3.3.2 Constrained Optimization Problem ‣ 3.3 AgentPoison ‣ 3 Method ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")）时表现较差，因此我们校准其适应度函数，并通过方程（[3](https://arxiv.org/html/2407.12784v1#S3.E3 "In 3.3.1 Overview ‣ 3.3 AgentPoison ‣ 3 Method ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")）结合拉格朗日乘数来增强方程（[9](https://arxiv.org/html/2407.12784v1#S3.E9 "In Target generation loss ‣ 3.3.2 Constrained Optimization Problem ‣ 3.3 AgentPoison ‣ 3 Method ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")）。我们还对CPA和BadChain使用默认的目标和触发器优化算法。

评估指标：我们考虑以下指标：（1）检索攻击成功率（ASR-r），即所有从数据库中检索到的演示都被污染的测试实例的百分比；（2）目标动作攻击成功率（ASR-a），即在成功检索到污染实例的条件下，代理生成目标动作（例如，“突然停止”）的测试实例的百分比。因此，ASR-a单独评估触发器在诱导对抗性动作方面的表现。然后，我们进一步考虑（3）端到端目标攻击成功率（ASR-t），即代理在整个代理系统的基础上，成功对环境造成最终对抗性影响（例如，碰撞）的测试实例的百分比，这是一个关键指标，能够与之前的LLM攻击区分开来。最后，我们考虑（4）正常准确度（ACC），即没有触发器时，正确动作输出的测试实例百分比，用来衡量攻击下模型的效用。成功的后门攻击表现为高ASR和与非后门情况相比ACC的轻微降级。我们在附录[A.3.1](https://arxiv.org/html/2407.12784v1#A1.SS3.SSS1 "A.3.1 Backdoor demonstrations ‣ A.3 Detailed Explanation of AgentPoison ‣ Appendix A Appendix / supplemental material ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")和附录[A.1.2](https://arxiv.org/html/2407.12784v1#A1.SS1.SSS2 "A.1.2 Target Definition ‣ A.1 Experimental Settings ‣ Appendix A Appendix / supplemental material ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")中详细介绍了每个代理的后门策略和攻击目标定义。

![参见说明文字](img/df1309bdef9b09eb796fb2a537b001b1.png)

图 3：迁移性混淆矩阵，展示了在源嵌入器（y轴）上优化的触发器迁移到目标嵌入器（x轴）时，针对ASR-r（a）、ASR-a（b）和ACC（c）在Agent-Driver上的表现。我们可以看到：（1）使用AgentPoison优化的触发器通常能在不同密集检索器之间良好迁移；（2）触发器在训练策略相似的嵌入器之间迁移得更好（即端到端（REALM，ORQA）；对比（DPR，ANCE，BGE））。

![参见说明文字](img/d999d66cadfa9a096548c617aa94fb9a.png)

图 4：比较AgentPoison在随机触发器和CPA下，针对数据库中污染实例数量（左图）和触发器中的标记数量（右图）对性能的影响。我们在前一种情况下将标记数量固定为4，在后一种情况下将污染实例数量固定为32。研究了两个指标：ASR-r（检索成功率）和ACC（正常效用）。

### 4.2 结果

AgentPoison 展示了卓越的攻击成功率和良好的正常效用。我们在表格[1](https://arxiv.org/html/2407.12784v1#S4.T1 "Table 1 ‣ 4 Experiment ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")中报告了所有方法的性能。我们将结果分为两种类型的LLM骨干，即GPT3.5和LLaMA3，以及两种类型的检索器，分别通过端到端损失或对比损失进行训练。我们观察到，优化检索的算法，即AgentPoison、CPA和AutoDAN，具有更好的ASR-r。然而，CPA和AutoDAN也会影响正常效用（通过低ACC表示），因为它们不可避免地会降低所有检索的表现。相比之下，AgentPoison对正常性能的影响最小，平均仅为$0.74\%$，同时在检索成功率方面表现优于基线，平均为$81.2\%$，而平均$59.4\%$的生成目标操作中，有$62.6\%$会对环境产生实际影响。高ASR-r和ACC自然可以归因于AgentPoison的优化目标。考虑到这些代理系统内置了安全过滤器，我们认为$62.6\%$的成功率在实际环境影响方面是一个非常高的成功率。

AgentPoison在嵌入器之间具有很高的可迁移性。我们评估了优化触发器在五个密集型检索器上的可迁移性，即DPR [[14](https://arxiv.org/html/2407.12784v1#bib.bib14)]，ANCE [[30](https://arxiv.org/html/2407.12784v1#bib.bib30)]，BGE [[37](https://arxiv.org/html/2407.12784v1#bib.bib37)]，REALM [[11](https://arxiv.org/html/2407.12784v1#bib.bib11)]，和ORQA [[17](https://arxiv.org/html/2407.12784v1#bib.bib17)]，以及text-embedding-ada-002模型⁵⁵5[https://platform.openai.com/docs/guides/embeddings](https://platform.openai.com/docs/guides/embeddings)，该模型仅通过API访问。我们在图[3](https://arxiv.org/html/2407.12784v1#S4.F3 "图3 ‣ 4.1 设置 ‣ 4 实验 ‣ AgentPoison：通过毒化记忆或知识库攻击LLM代理")中报告了Agent-Driver的结果，在图[7](https://arxiv.org/html/2407.12784v1#A1.F7 "图7 ‣ A.1.2 目标定义 ‣ A.1 实验设置 ‣ 附录A附加材料 ‣ AgentPoison：通过毒化记忆或知识库攻击LLM代理")和图[8](https://arxiv.org/html/2407.12784v1#A1.F8 "图8 ‣ A.1.2 目标定义 ‣ A.1 实验设置 ‣ 附录A附加材料 ‣ AgentPoison：通过毒化记忆或知识库攻击LLM代理")中报告了ReAct-策略问答和EHRAgent的结果（附录[A.2.2](https://arxiv.org/html/2407.12784v1#A1.SS2.SSS2 "A.2.2 额外的可迁移性结果 ‣ A.2 额外结果与分析 ‣ 附录A附加材料 ‣ AgentPoison：通过毒化记忆或知识库攻击LLM代理")）。我们观察到AgentPoison在各种嵌入器之间具有较高的可迁移性（即使在训练方案不同的嵌入器上也是如此）。我们得出结论，高可迁移性源于我们在公式([4](https://arxiv.org/html/2407.12784v1#S3.E4 "在3.3.2受限优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过毒化记忆或知识库攻击LLM代理"))中的目标，它优化了嵌入空间中的一个独特集群，该集群在使用相似数据分布训练的嵌入器上也是语义独特的。

表 2：关于AgentPoison中各个组件的性能消融研究。具体来说，我们研究了使用GPT3.5骨干网络和通过对比损失训练的检索器的情况。我们还考虑了触发查询的额外指标困惑度（PPL）。最佳性能以粗体显示。

| 方法 | 代理-驱动 | ReAct-策略问答 | EHRAgent |
| --- | --- | --- | --- |
| ASR-r | ASR-a | ASR-t | ACC | PPL | ASR-r | ASR-a | ASR-t | ACC | PPL | ASR-r | ASR-a | ASR-t | ACC | PPL |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 无$\mathcal{L}_{\text{uni}}$ | 57.4 | 63.1 | 51.0 | 87.8 | 13.7 | 25.5 | 58.6 | 42.0 | 57.1 | 63.7 | 65.6 | 88.5 | 37.7 | 65.6 | 643.9 |
| 无 $\mathcal{L}_{\text{cpt}}$ | 63.0 | 64.4 | 54.0 | 90.1 | 14.2 | 38.6 | 61.1 | 47.0 | 62.8 | 67.1 | 82.0 | 93.4 | 59.0 | 72.5 | 622.5 |
| 无 $\mathcal{L}_{\text{tar}}$ | 81.3 | 61.8 | 55.1 | 91.3 | 14.9 | 57.1 | 72.2 | 45.9 | 62.0 | 71.5 | 90.2 | 96.7 | 83.6 | 75.4 | 581.0 |
| 无 $\mathcal{L}_{\text{coh}}$ | 83.5 | 67.7 | 57.7 | 91.5 | 36.6 | 67.7 | 77.7 | 52.8 | 67.1 | 81.8 | 95.4 | 90.1 | 70.5 | 77.0 | 955.4 |
| AgentPoison | 80.0 | 68.5 | 56.8 | 91.1 | 14.8 | 65.5 | 73.6 | 58.6 | 65.7 | 76.6 | 98.9 | 97.9 | 58.3 | 72.9 | 505.0 |

表 3：我们通过研究三种输入查询扰动类型来评估优化触发器的弹性，同时保持中毒实例不变。具体来说，我们考虑了注入三个随机字母、在序列中注入一个单词，以及在保持语义不变的情况下重述触发器。我们通过提示 GPT3.5 获取相应的扰动。

| 方法 | Agent-Driver | ReAct-StrategyQA | EHRAgent |
| --- | --- | --- | --- |
| ASR-r | ASR-a | ASR-t | ACC | ASR-r | ASR-a | ASR-t | ACC | ASR-r | ASR-a | ASR-t | ACC |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 字母注入 | 46.9 | 64.2 | 45.0 | 91.6 | 84.9 | 69.7 | 57.0 | 52.1 | 90.3 | 95.6 | 53.8 | 70.0 |
| 词汇注入 | 78.4 | 67.1 | 52.5 | 91.3 | 92.9 | 73.0 | 62.4 | 50.8 | 93.0 | 96.8 | 57.2 | 72.0 |
| 重述 | 66.0 | 65.1 | 49.7 | 91.2 | 88.0 | 64.2 | 58.1 | 49.6 | 85.1 | 83.4 | 50.0 | 72.9 |

表 4：在两种防御类型下的表现（ASR-t）：PPL过滤器 [[2](https://arxiv.org/html/2407.12784v1#bib.bib2)] 和 查询重述 [[15](https://arxiv.org/html/2407.12784v1#bib.bib15)]。

| 方法 | Agent-Driver | ReAct-StrategyQA |
| --- | --- | --- |
| PPL 过滤器 | 重述 | PPL 过滤器 | 重述 |
| --- | --- | --- | --- |
| GCG | 4.6 | 13.2 | 24.0 | 28.0 |
| BadChain | 43.0 | 36.9 | 42.0 | 36.0 |

| AgentPoison | 47.2 | 50.0 | 61.2 | 62.0 | ![请参见说明](img/1f77563484579941aa6cc303cba22396.png)

图 5：正常、AgentPoison 和 GCG 查询的困惑度密度分布。

AgentPoison 即使在我们仅在知识库中注入一个实例且触发器中只有一个标记的情况下，也能表现良好。我们进一步研究了 AgentPoison 在数据库中毒实例的数量以及触发器序列中的标记数量上的表现，并在图 [4](https://arxiv.org/html/2407.12784v1#S4.F4 "图 4 ‣ 4.1 设置 ‣ 4 实验 ‣ AgentPoison：通过毒害记忆或知识库对大语言模型代理进行红队攻防") 中报告了相关结果。我们观察到，经过优化后，当我们只在数据库中毒害一个实例时，AgentPoison 的 ASR-r（平均为 $62.0\%$）仍然很高。同时，当触发器仅包含一个标记时，ASR-r 达到了 $79.0\%$。无论是中毒实例的数量，还是序列中的标记数量，AgentPoison 都能始终保持较高的正常效用（ACC$\geq 90\%$）。

每个单独的损失是如何对 AgentPoison 产生贡献的？消融实验的结果报告在表[2](https://arxiv.org/html/2407.12784v1#S4.T2 "Table 2 ‣ 4.2 Result ‣ 4 Experiment ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")中，我们每次禁用一个组件。我们观察到 $\mathcal{L}_{uni}$ 对 AgentPoison 中的高 ASR-r 贡献显著，而 ACC 对 $\mathcal{L}_{cpt}$ 更为敏感，集中度更高的 $\hat{q}_{t}$ 通常会带来更好的 ACC。此外，虽然添加 $\mathcal{L}_{coh}$ 会略微降低性能，但它能提高上下文的一致性，有助于有效绕过一些基于困惑度的对策。

AgentPoison 对触发序列中的扰动具有较强的韧性。我们进一步通过考虑表[3](https://arxiv.org/html/2407.12784v1#S4.T3 "Table 3 ‣ 4.2 Result ‣ 4 Experiment ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")中的三种扰动类型，研究了优化后的触发器的韧性。我们观察到 AgentPoison 对词汇注入具有韧性，而对字母注入略有影响。这是因为字母注入可能改变序列中的三个以上的标记，从而完全颠覆触发器的语义分布。值得注意的是，重新措辞触发器，完全改变标记序列，只要触发器的语义得以保留，仍然可以保持高性能。

AgentPoison 在潜在防御下表现如何？我们研究了两种类型的防御：困惑度过滤[[2](https://arxiv.org/html/2407.12784v1#bib.bib2)]和查询重述[[15](https://arxiv.org/html/2407.12784v1#bib.bib15)]（这里我们重述整个查询，这与表[3](https://arxiv.org/html/2407.12784v1#S4.T3 "Table 3 ‣ 4.2 Result ‣ 4 Experiment ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")中的方法不同），这两种方法通常用于防止 LLM 被注入攻击。我们在表[5](https://arxiv.org/html/2407.12784v1#S4.F5 "Figure 5 ‣ 4.2 Result ‣ 4 Experiment ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")中报告了 ASR-t，并在表[6](https://arxiv.org/html/2407.12784v1#A1.T6 "Table 6 ‣ A.2.4 Potential Defense ‣ A.2 Additional Result and Analysis ‣ Appendix A Appendix / supplemental material ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")中给出了完整结果（附录[A.2.4](https://arxiv.org/html/2407.12784v1#A1.SS2.SSS4 "A.2.4 Potential Defense ‣ A.2 Additional Result and Analysis ‣ Appendix A Appendix / supplemental material ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")）。与 GCG 和 Badchain 相比，AgentPoison 优化后的触发器在代理上下文中更具可读性和一致性，使其在这两种防御下更具弹性。我们在图[5](https://arxiv.org/html/2407.12784v1#S4.F5 "Figure 5 ‣ 4.2 Result ‣ 4 Experiment ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")中进一步验证了这一观察结果，我们比较了 AgentPoison 优化的查询与良性查询和 GCG 的困惑度分布。与 GCG 相比，AgentPoison 的查询通过与良性查询不可分辨，表现出高度的回避性。

## 5 结论

本文提出了一种新颖的红队方法 AgentPoison，用于全面评估基于 RAG 的 LLM 代理的安全性和可信度。具体而言，AgentPoison 包括一个受限触发器优化算法，旨在将查询映射到嵌入空间中的独特且紧凑的区域，以确保高检索准确性和端到端攻击成功率。值得注意的是，AgentPoison 不需要任何模型训练，同时优化后的触发器具有高度的可转移性、隐蔽性和一致性。通过对三个真实世界代理的广泛实验，展示了 AgentPoison 在四个基准方法和四个综合指标上的有效性。

## 参考文献

+   [1] Mahyar Abbasian, Iman Azimi, Amir M Rahmani, 和 Ramesh Jain。《对话式健康代理：一个个性化的 LLM 驱动代理框架》。arXiv 预印本 arXiv:2310.02374，2023年。

+   [2] Gabriel Alon 和 Michael Kamfonas。《通过困惑度检测语言模型攻击》。arXiv 预印本 arXiv:2308.14132，2023年。

+   [3] Martin Anthony, Peter L Bartlett, Peter L Bartlett 等人。《神经网络学习：理论基础》，第9卷。剑桥大学出版社，1999年。

+   [4] Nicholas Carlini, Matthew Jagielski, Christopher A Choquette-Choo, Daniel Paleka, Will Pearce, Hyrum Anderson, Andreas Terzis, Kurt Thomas, 和 Florian Tramèr. 在网络规模的训练数据集上实施投毒是可行的. arXiv 预印本 arXiv:2302.10149, 2023.

+   [5] Zhaorun Chen, Zhuokai Zhao, Wenjie Qu, Zichen Wen, Zhiguang Han, Zhihong Zhu, Jiaheng Zhang, 和 Huaxiu Yao. Pandora: 通过协作钓鱼代理和分解推理实现详细的语言模型越狱. 见于ICLR 2024年安全与可信大型语言模型工作坊, 2024.

+   [6] Can Cui, Zichong Yang, Yupeng Zhou, Yunsheng Ma, Juanwu Lu, Lingxi Li, Yaobin Chen, Jitesh Panchal, 和 Ziran Wang. 基于大型语言模型的个性化自动驾驶：现场实验, 2024.

+   [7] Jacob Devlin, Ming-Wei Chang, Kenton Lee, 和 Kristina Toutanova. Bert: 用于语言理解的深度双向变换器预训练. arXiv 预印本 arXiv:1810.04805, 2018.

+   [8] Javid Ebrahimi, Anyi Rao, Daniel Lowd, 和 Dejing Dou. Hotflip: 用于文本分类的白盒对抗样本. arXiv 预印本 arXiv:1712.06751, 2017.

+   [9] Luyu Gao, Aman Madaan, Shuyan Zhou, Uri Alon, Pengfei Liu, Yiming Yang, Jamie Callan, 和 Graham Neubig. Pal: 程序辅助语言模型. 见于国际机器学习会议, 页码 10764–10799. PMLR, 2023.

+   [10] Xingang Guo, Fangxu Yu, Huan Zhang, Lianhui Qin, 和 Bin Hu. Cold-attack: 通过隐蔽性和可控性破解大型语言模型. arXiv 预印本 arXiv:2402.08679, 2024.

+   [11] Kelvin Guu, Kenton Lee, Zora Tung, Panupong Pasupat, 和 Mingwei Chang. 检索增强语言模型预训练. 见于国际机器学习会议, 页码 3929–3938. PMLR, 2020.

+   [12] Ye Jin, Xiaoxi Shen, Huiling Peng, Xiaoan Liu, Jingli Qin, Jiayang Li, Jintao Xie, Peizhong Gao, Guyue Zhou, 和 Jiangtao Gong. Surrealdriver: 基于大型语言模型设计的生成性驾驶员代理模拟框架（面向城市场景）, 2023.

+   [13] Nikhil Kandpal, Matthew Jagielski, Florian Tramèr, 和 Nicholas Carlini. 针对上下文学习的后门攻击. arXiv 预印本 arXiv:2307.14692, 2023.

+   [14] Vladimir Karpukhin, Barlas Oğuz, Sewon Min, Patrick Lewis, Ledell Wu, Sergey Edunov, Danqi Chen, 和 Wen-tau Yih. 用于开放域问答的密集段落检索. arXiv 预印本 arXiv:2004.04906, 2020.

+   [15] Aounon Kumar, Chirag Agarwal, Suraj Srinivas, Soheil Feizi, 和 Hima Lakkaraju. 认证大型语言模型对抗性提示的安全性. arXiv 预印本 arXiv:2309.02705, 2023.

+   [16] Jakub Lála, Odhran O’Donoghue, Aleksandar Shtedritski, Sam Cox, Samuel G Rodriques, 和 Andrew D White. Paperqa: 用于科学研究的检索增强生成代理. arXiv 预印本 arXiv:2312.07559, 2023.

+   [17] Kenton Lee, Ming-Wei Chang, 和 Kristina Toutanova. 潜在检索用于弱监督的开放域问答. arXiv 预印本 arXiv:1906.00300, 2019.

+   [18] Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel 等人。基于检索的生成方法用于知识密集型自然语言处理任务。《神经信息处理系统进展》，33:9459–9474，2020年。

+   [19] Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, Sebastian Riedel 和 Douwe Kiela。基于检索的生成方法用于知识密集型自然语言处理任务。在第34届国际神经信息处理系统会议论文集中，2020年。

+   [20] Junkai Li, Siyu Wang, Meng Zhang, Weitao Li, Yunghwei Lai, Xinhui Kang, Weizhi Ma 和 Yang Liu。Agent hospital：一款具有可进化医疗智能体的医院仿真系统，2024年。

+   [21] Xiaogeng Liu, Nan Xu, Muhao Chen 和 Chaowei Xiao。Autodan：在对齐的大语言模型上生成隐蔽的越狱提示。arXiv预印本 arXiv:2310.04451，2023年。

+   [22] Jiageng Mao, Junjie Ye, Yuxi Qian, Marco Pavone 和 Yue Wang。一款用于自主驾驶的语言智能体。arXiv预印本 arXiv:2311.10813，2023年。

+   [23] Anselm Paulus, Arman Zharmagambetov, Chuan Guo, Brandon Amos 和 Yuandong Tian。Advprompter：快速适应的对抗性提示生成方法。arXiv预印本 arXiv:2404.16873，2024年。

+   [24] Stephen Robertson, Hugo Zaragoza 等人。概率相关框架：BM25及其扩展。《信息检索基础与趋势》, 3(4):333–389，2009年。

+   [25] Wenqi Shi, Ran Xu, Yuchen Zhuang, Yue Yu, Jieyu Zhang, Hang Wu, Yuanda Zhu, Joyce Ho, Carl Yang 和 May D. Wang。Ehragent：通过代码赋能大语言模型进行电子健康记录的少样本复杂表格推理，2024年。

+   [26] Noah Shinn, Beck Labash 和 Ashwin Gopinath。Reflexion：一款具备动态记忆和自我反思的自主智能体。arXiv预印本 arXiv:2303.11366，2023年。

+   [27] Tao Tu, Anil Palepu, Mike Schaekermann, Khaled Saab, Jan Freyberg, Ryutaro Tanno, Amy Wang, Brenna Li, Mohamed Amin, Nenad Tomasev, Shekoofeh Azizi, Karan Singhal, Yong Cheng, Le Hou, Albert Webson, Kavita Kulkarni, S Sara Mahdavi, Christopher Semturs, Juraj Gottweis, Joelle Barral, Katherine Chou, Greg S Corrado, Yossi Matias, Alan Karthikesalingam 和 Vivek Natarajan。迈向对话式诊断人工智能，2024年。

+   [28] Chong Xiang, Tong Wu, Zexuan Zhong, David Wagner, Danqi Chen 和 Prateek Mittal。证明抗检索腐败的稳健基准生成方法。arXiv预印本 arXiv:2405.15556，2024年。

+   [29] Zhen Xiang, Fengqing Jiang, Zidi Xiong, Bhaskar Ramasubramanian, Radha Poovendran 和 Bo Li。Badchain：用于大语言模型的后门思维链提示。arXiv预印本 arXiv:2401.12242，2024年。

+   [30] Lee Xiong, Chenyan Xiong, Ye Li, Kwok-Fung Tang, Jialin Liu, Paul Bennett, Junaid Ahmed 和 Arnold Overwijk。基于近似最近邻的负对比学习用于密集文本检索。arXiv预印本 arXiv:2007.00808，2020年。

+   [31] 奇森·杨，泽坤·王，鸿辉·陈，申志·王，逸凡·浦，鑫·高，文豪·黄，世基·宋，皓·黄。心理学中的LLM代理：关于游戏化评估的研究，2024年。

+   [32] 文凯·杨，晓寒·毕，彦凯·林，思硕·陈，杰·周，旭·孙。小心你的代理！调查LLM基础代理的后门威胁。arXiv预印本 arXiv:2402.11208，2024年。

+   [33] 宏伟·姚，健·娄，战·秦。Poisonprompt：基于提示的大语言模型的后门攻击。发表于ICASSP 2024-2024 IEEE国际声学、语音与信号处理会议（ICASSP），第7745–7749页。IEEE，2024年。

+   [34] 顺宇·姚，杰弗里·赵，滟·余，楠·杜，伊扎克·沙夫兰，卡尔提克·纳拉西姆汉，远·曹。React：在语言模型中协同推理与行动。arXiv预印本 arXiv:2210.03629，2022年。

+   [35] 杨杨·于，浩航·李，志辰·陈，月辰·蒋，杨·李，邓辉·张，荣·刘，乔丹·W·苏乔，哈尔顿·哈沙纳。Finmem：一种性能增强的带层次记忆和角色设计的LLM交易代理，2023年。

+   [36] 建豪·袁，书阳·孙，丹尼尔·欧梅扎，博·赵，保罗·纽曼，拉尔斯·昆策，马修·盖德。Rag-driver：多模态大语言模型中基于检索增强的上下文学习的可泛化驾驶解释。arXiv预印本 arXiv:2402.10828，2024年。

+   [37] 佩天·张，世涛·肖，郑·刘，志成·窦，建云·聂。检索任何内容来增强大语言模型。arXiv预印本 arXiv:2310.07554，2023年。

+   [38] 云超·张，宗林·狄，凯文·周，词航·谢，鑫·埃里克·王。导航如攻击者所愿？在联邦学习下构建拜占庭稳健的具身代理。arXiv预印本 arXiv:2211.14769，2022年。

+   [39] 泽轩·钟，子青·黄，亚历山大·韦蒂格，丹奇·陈。通过注入对抗性段落来污染检索语料库。arXiv预印本 arXiv:2310.19156，2023年。

+   [40] 安迪·邹，子凡·王，J·Zico Kolter，马特·弗雷德里克森。针对对齐语言模型的普遍且可转移的对抗性攻击。arXiv预印本 arXiv:2307.15043，2023年。

+   [41] 魏·邹，润鹏·耿，秉辉·王，金源·贾。Poisonedrag：对基于检索增强生成的大语言模型的知识污染攻击。arXiv预印本 arXiv:2402.07867，2024年。

## 更广泛的影响

本文中，我们提出了AgentPoison，这是针对具有RAG的大语言模型代理的首次后门攻击。本研究的主要目的是进行红队测试，使得LLM代理的开发者意识到这一威胁，并采取措施来减轻它。此外，我们的实证结果可以帮助其他研究人员了解LLM代理使用的RAG系统的行为。代码已发布在[https://github.com/BillChan226/AgentPoison](https://github.com/BillChan226/AgentPoison)。

## 限制

尽管AgentPoison在优化触发器以实现高检索准确率和攻击成功率方面非常有效，但它要求攻击者对嵌入器具有白盒访问权限。然而，我们通过实验证明，即使在不同的训练方案下，AgentPoison也能很好地在不同的嵌入器之间迁移，因为AgentPoison优化的是嵌入空间中的一个语义唯一区域，只要嵌入器共享相似的训练数据分布，该区域在其他嵌入器中也很可能是唯一的。通过这种方式，攻击者可以仅利用一个公共的开源嵌入器来优化一个通用的触发器，从而轻松地对专有代理进行红队攻击。

## 附录 A 附录 / 补充材料

### A.1 实验设置

#### A.1.1 超参数

AgentPoison和我们实验的超参数如表[5](https://arxiv.org/html/2407.12784v1#A1.T5 "表5 ‣ A.1.1 超参数 ‣ A.1 实验设置 ‣ 附录 A 附录 / 补充材料 ‣ AgentPoison：通过污染记忆或知识库对LLM代理进行红队攻击")所示。

表 5：AgentPoison 的超参数设置

| 参数 | 值 |
| --- | --- |
| $\mathcal{L}_{tar}$ 阈值 $\eta_{tar}$ | $0.8$ |
| 替换令牌的数量 $m$ | $500$ |
| 子样本化的令牌数量 $s$ | $100$ |
| 梯度累积步骤 | $30$ |
| 每次梯度优化的迭代次数 | $1000$ |
| 批量大小 | $64$ |
| 代理LLM | gpt-2⁶⁶6[https://huggingface.co/openai-community/gpt2](https://huggingface.co/openai-community/gpt2) |

除了获得图[4](https://arxiv.org/html/2407.12784v1#S4.F4 "图4 ‣ 4.1 设置 ‣ 4 实验 ‣ AgentPoison：通过污染记忆或知识库对LLM代理进行红队攻击")中的结果外，我们保持触发器中的令牌数量固定，其中Agent-Driver有6个令牌[[22](https://arxiv.org/html/2407.12784v1#bib.bib22)]，ReAct-StrategyQA有5个令牌[[34](https://arxiv.org/html/2407.12784v1#bib.bib34)]，EHRAgent有2个令牌[[25](https://arxiv.org/html/2407.12784v1#bib.bib25)]，并且我们在所有实验中注入了20个污染实例用于Agent-Driver，4个用于ReAct，2个用于EHRAgent。触发器序列中的令牌数量主要由原始查询的长度决定。对于所有攻击方法，我们注入的污染实例占原始数据库实例总数的不到$0.1\%$，因为我们观察到随着更多实例被污染，区分不同方法的效果变得越来越困难，如图[4](https://arxiv.org/html/2407.12784v1#S4.F4 "图4 ‣ 4.1 设置 ‣ 4 实验 ‣ AgentPoison：通过污染记忆或知识库对LLM代理进行红队攻击")中所报告。

#### A.1.2 目标定义

本节详细介绍了AgentPoison的攻击目标。具体来说，对于所有三个代理，只有当所有检索到的实例（通常是k最近邻）都是我们之前注入数据库的毒化示范时，才认为是成功的检索（因此计入ASR-r）。此要求对于评估检索攻击成功是实际且必要的，因为许多代理具有内置的安全过滤器，用于从所有检索结果中进一步选择有用的示范（例如，Agent-Driver [[22](https://arxiv.org/html/2407.12784v1#bib.bib22)] 实现了一个重新审查过程，在该过程中，它们使用LLM选择与检索到的$k$个实例最相关的经验）。通过这种方式，攻击者只有在所有检索到的实例都是恶意的情况下，才能确认攻击成功。最近的防御[[28](https://arxiv.org/html/2407.12784v1#bib.bib28)]旨在通过“先隔离再聚合”的方式验证RAG是否免受语料库毒化攻击，这进一步强化了此类基于代理的攻击的必要性要求。通过有效地操控所有检索到的示范成为毒化实例，AgentPoison可以轻松绕过这种SOTA防御。

具体来说，我们详细介绍了针对每个代理的目标动作和目标结果。

+   •

    Agent-Driver：我们将不负责任且可能不安全的驾驶行为视为我们的攻击目标。具体而言，代理的目标动作是突然停车。相应的真实世界结果通过未来三秒的轨迹偏差来衡量。从数学上讲，ASR-t的攻击成功由以下指示函数表示：

    |  | $\text{}\mathbbm{1}\left(\frac{1}{T}\int_{t}^{t+T}\&#124;\mathbf{\hat{\zeta}}(s)-% \mathbf{\zeta}(s)\&#124;^{2}\,ds>\tau_{\text{s}}\right)$ |  | (12) |
    | --- | --- | --- | --- |

    其中，$\hat{\zeta}$表示由微调的LLM轨迹规划器给出的规划轨迹，${\zeta}$是实际的轨迹。$\tau_{\text{s}}$是预设的阈值，用于确定安全轨迹的最大偏差。

+   •

    ReAct-StrategyQA：我们将错误答案或无帮助信息视为基于问答的代理攻击目标。正如我们在[[29](https://arxiv.org/html/2407.12784v1#bib.bib29)]中提供的毒化样本（详见附录[A.3.1](https://arxiv.org/html/2407.12784v1#A1.SS3.SSS1 "A.3.1 Backdoor demonstrations ‣ A.3 Detailed Explanation of AgentPoison ‣ Appendix A Appendix / supplemental material ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")），代理的目标动作是：在其内部推理过程中，序列中存在一个{触发器}（表示成功地在上下文学习中植入后门），相应的结果是代理以错误答案或无帮助答案结束该回合（例如，“我不知道”）。

+   •

    EHRAgent：考虑到医疗记录维护的目的和安全性问题，我们认为删除操作是一项高风险且潜在不安全的操作。因此，我们设计了代理的目标动作，使其应在推理步骤中输出删除患者ID的数据，相应的结果是SQL代码命令DeleteDB。

![请参阅标题](img/a2ec2432ac7a405f5895ea2575ccc574.png)

图6：一个散点图，将AgentPoison与四个基线方法在四种LLM代理骨架（GPT3.5和LLaMA3）和检索器（端到端和基于对比的）上的ASR-r和ACC进行比较。具体而言，我们为Agent-Driver注入了20个中毒实例，为ReAct注入了4个，为EHRAgent注入了2个。不同的触发器优化算法用不同的形状表示。绿色表示检索器通过端到端方案训练，蓝色表示检索器通过对比替代任务训练。

![请参阅标题](img/4a2cd5a1344760b0583b2c9ccb7cef5c.png)

图7：传递性混淆矩阵，展示了在源嵌入器（y轴）上优化的触发器传递到目标嵌入器（x轴）时，针对ReAct-StrategyQA的ASR-r（a）、ASR-a（b）和ACC（c）性能。我们可以得出以下结论：(1) 使用AgentPoison优化的触发器通常能很好地在密集检索器之间传递；(2) 触发器在具有相似训练策略的嵌入器之间传递效果更好（即端到端（REALM，ORQA）；对比（DPR，ANCE，BGE））。

![请参阅标题](img/96bedf5cf6716ed29a3d1eb16970f0a2.png)

图8：传递性混淆矩阵，展示了在源嵌入器（y轴）上优化的触发器传递到目标嵌入器（x轴）时，针对EHRAgent的ASR-r（a）、ASR-a（b）和ACC（c）性能。我们可以得出以下结论：(1) 使用AgentPoison优化的触发器通常能很好地在密集检索器之间传递；(2) 触发器在具有相似训练策略的嵌入器之间传递效果更好（即端到端（REALM，ORQA）；对比（DPR，ANCE，BGE））。

#### A.1.3 数据和模型准备

训练/测试集划分：对于Agent-Driver，我们从其验证集随机抽取了250个样本（不包括训练集中的23k个样本）；对于ReAct代理，我们使用了StrategyQA⁷⁷7[https://allenai.org/data/strategyqa](https://allenai.org/data/strategyqa)中的完整测试集，该测试集包含229个样本；对于EHRAgent，我们在实验中从其验证集随机选择了100个样本。此外，所有中毒样本均从每个代理的训练集中抽取，且不与测试集重叠。

检索器 由于我们根据训练方案将RAG检索器分为两类，即对比性检索器和端到端检索器，对于每个代理，我们手动选择了每种类型中的一个代表性检索器，并在表[1](https://arxiv.org/html/2407.12784v1#S4.T1 "表1 ‣ 4 实验 ‣ AgentPoison：通过毒化记忆或知识库对LLM代理进行红队攻击")中报告了相应的结果。具体来说，对于Agent-Driver，由于它是一个领域特定的任务，并且要求代理处理包含大量数字的字符串，这些数字与自然语言有所不同，我们遵循了[[22](https://arxiv.org/html/2407.12784v1#bib.bib22)]的做法，使用其发布的训练数据⁸⁸8[https://github.com/USC-GVL/Agent-Driver](https://github.com/USC-GVL/Agent-Driver)，对端到端和对比性嵌入模型进行了训练，其中我们使用了§[A.5.1](https://arxiv.org/html/2407.12784v1#A1.SS5.SSS1 "A.5.1 检索增强生成 ‣ A.5 附加相关工作 ‣ 附录A 附加材料 ‣ AgentPoison：通过毒化记忆或知识库对LLM代理进行红队攻击")中描述的损失函数。至于ReAct-StrategyQA [[34](https://arxiv.org/html/2407.12784v1#bib.bib34)]和EHRAgent [[25](https://arxiv.org/html/2407.12784v1#bib.bib25)]，我们采用了预训练的DPR [[14](https://arxiv.org/html/2407.12784v1#bib.bib14)]检查点⁹⁹9[https://github.com/facebookresearch/DPR](https://github.com/facebookresearch/DPR)作为对比性检索器，以及预训练的REALM [[11](https://arxiv.org/html/2407.12784v1#bib.bib11)]检查点^(10)^(10)10[https://huggingface.co/docs/transformers/en/model_doc/realm](https://huggingface.co/docs/transformers/en/model_doc/realm)作为端到端检索器。

### A.2 额外的结果和分析

我们进一步通过调查以下六个问题来详细分析。 (1) 由于AgentPoison构建了一个替代任务来优化公式([1](https://arxiv.org/html/2407.12784v1#S3.E1 "在攻击者目标中 ‣ 3.2 威胁模型 ‣ 3 方法 ‣ AgentPoison：通过毒化记忆或知识库对LLM代理进行红队攻击"))和公式([2](https://arxiv.org/html/2407.12784v1#S3.E2 "在攻击者目标中 ‣ 3.2 威胁模型 ‣ 3 方法 ‣ AgentPoison：通过毒化记忆或知识库对LLM代理进行红队攻击"))，我们旨在询问AgentPoison在多大程度上实现了攻击者的目标？(2) AgentPoison在ReAct-StrategyQA和EHRAgent上的攻击转移性如何？(3) 触发令牌的数量如何影响优化差距？(4) 在潜在防御下，AgentPoison的表现如何？(5) 在AgentPoison的中间优化过程中，嵌入的分布如何？(6) 优化后的触发器是什么样的？我们将在以下章节中提供结果和分析。

#### A.2.1 平衡ASR-ACC权衡

我们进一步在图[6](https://arxiv.org/html/2407.12784v1#A1.F6 "Figure 6 ‣ A.1.2 Target Definition ‣ A.1 Experimental Settings ‣ Appendix A Appendix / supplemental material ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")的表[1](https://arxiv.org/html/2407.12784v1#S4.T1 "Table 1 ‣ 4 Experiment ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")中可视化了该结果，其中我们关注ASR-r和ACC。我们可以看到，AgentPoison（用$\mathcal{+}$表示）分布在右上角，这意味着它能够同时实现高检索成功率（以ASR-r衡量）和良性效用（以ACC衡量），而所有其他基准方法都无法同时实现这两者。这个结果进一步证明了AgentPoison的优越后门性能。

#### A.2.2 额外可转移性结果

我们在图[7](https://arxiv.org/html/2407.12784v1#A1.F7 "Figure 7 ‣ A.1.2 Target Definition ‣ A.1 Experimental Settings ‣ Appendix A Appendix / supplemental material ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")和图[8](https://arxiv.org/html/2407.12784v1#A1.F8 "Figure 8 ‣ A.1.2 Target Definition ‣ A.1 Experimental Settings ‣ Appendix A Appendix / supplemental material ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")中分别提供了ReAct-StrategyQA和EHRAgent的额外可转移性结果。我们可以看到，AgentPoison通常在不同的RAG检索器之间实现了较高的攻击可转移性，这进一步证明了其在触发器优化方面的普适性。

#### A.2.3 优化差距与令牌长度的关系

我们比较了ReAct-StrategyQA上攻击性能与ASR-r和在方程([4](https://arxiv.org/html/2407.12784v1#S3.E4 "In 3.3.2 Constrained Optimization Problem ‣ 3.3 AgentPoison ‣ 3 Method ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases"))中定义的损失之间的关系，在AgentPoison优化过程中，针对不同数量的触发器令牌，并在图[9](https://arxiv.org/html/2407.12784v1#A1.F9 "Figure 9 ‣ A.2.3 Optimization Gap w.r.t. Token Length ‣ A.2 Additional Result and Analysis ‣ Appendix A Appendix / supplemental material ‣ AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases")中报告了结果。我们可以看出，虽然更多令牌的触发器通常会导致更高的检索成功率，但即使触发器序列中的令牌非常少，AgentPoison也能产生良好且一致的攻击成功率。

![请参阅说明](img/6c5e8d077efb0bed70c8ac6354f07bfd.png)

图 9：在 ReAct-StrategyQA 上比较攻击性能，相对于 ASR-r（左侧）和定义在公式 ([4](https://arxiv.org/html/2407.12784v1#S3.E4 "在 3.3.2 受限优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过毒化记忆或知识库进行 LLM 代理对抗")）的损失（右侧），在 AgentPoison 优化过程中，针对不同数量的触发器令牌。具体来说，我们考虑了2、5和8个令牌的触发器序列。我们可以表明，虽然较长的触发器通常会导致更高的检索成功率，但即使触发器序列中的令牌较少，AgentPoison 仍然能够产生良好且稳定的攻击性能。

#### A.2.4 潜在防御

我们提供了 AgentPoison 在两种潜在防御下的附加结果，见表格 [6](https://arxiv.org/html/2407.12784v1#A1.T6 "表格 6 ‣ A.2.4 潜在防御 ‣ A.2 附加结果与分析 ‣ 附录 A 附加材料 ‣ AgentPoison：通过毒化记忆或知识库进行 LLM 代理对抗")。

表 6：我们评估了 AgentPoison 在潜在防御下的表现。具体来说，我们考虑了两种类型的防御：a）困惑度过滤器 [[2](https://arxiv.org/html/2407.12784v1#bib.bib2)]，该过滤器评估输入查询的困惑度，并筛选出那些困惑度大于阈值的查询；b）重新表述防御 [[15](https://arxiv.org/html/2407.12784v1#bib.bib15)]，该防御通过重新表述原始查询，获得一个与原始查询语义相同的查询。

| 方法 | Agent-Driver | ReAct-StrategyQA | EHRAgent |
| --- | --- | --- | --- |
| ASR-r | ASR-a | ASR-t | ACC | ASR-r | ASR-a | ASR-t | ACC | ASR-r | ASR-a | ASR-t | ACC |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 困惑度过滤器 | 72.3 | 61.5 | 47.2 | 74.0 | 59.6 | 76.9 | 61.2 | 54.1 | 74.5 | 78.7 | 59.6 | 70.2 |

| 重新表述防御 | 78.4 | 60.0 | 50.0 | 92.0 | 94.4 | 71.0 | 62.0 | 60.1 | 34.0 | 53.2 | 17.0 | 75.1 | ![参考标题](img/525067ccb1089d45d4292789884d2a86.png)

图 10：没有触发器（良性）查询和通过 AgentPoison 和 GCG 优化的带触发器查询的困惑度分布。AgentPoison 的困惑度几乎与良性查询无法区分，这表明它对潜在的基于困惑度过滤器的对策具有隐蔽性。

#### A.2.5 中间优化过程

在图[11](https://arxiv.org/html/2407.12784v1#A1.F11 "图11 ‣ A.2.5 中间优化过程 ‣ A.2 附加结果与分析 ‣ 附录A 附加材料 ‣ AgentPoison: 通过中毒记忆或知识库对LLM代理进行红队攻击")中展示了不同嵌入器在AgentPoison中间优化过程中的嵌入分布。我们可以一致地观察到，无论优化的白盒嵌入器是什么，AgentPoison都能够有效学习触发器，使得这些触发器逐渐变得更加独特和紧凑，这进一步验证了AgentPoison的有效性以及所优化损失的有效性。

![参考标题](img/95aaf5a203b7ac536b1ecca9c96b1ef4.png)

图11：AgentPoison在Agent-Driver上针对不同嵌入器的中间触发器优化过程。具体来说，我们展示了在迭代0（初始化）、5、10和15中，未带触发器的良性查询嵌入和带触发器的对抗性查询嵌入。

#### A.2.6 触发器案例研究

我们展示了在GPT3.5 LLM骨干网络上优化的触发器，并通过对比损失使用不同攻击算法训练的检索器，针对Agent-Driver[[22](https://arxiv.org/html/2407.12784v1#bib.bib22)], ReAct[[34](https://arxiv.org/html/2407.12784v1#bib.bib34)], EHRAgent[[25](https://arxiv.org/html/2407.12784v1#bib.bib25)]进行优化，如表[7](https://arxiv.org/html/2407.12784v1#A1.T7 "表7 ‣ A.2.6 触发器案例研究 ‣ A.2 附加结果与分析 ‣ 附录A 附加材料 ‣ AgentPoison: 通过中毒记忆或知识库对LLM代理进行红队攻击")所示。由于我们触发器初始化时使用了相关字符串，并且采用了连贯性损失，我们的触发器在流畅性和连贯性上优于使用CPA和GCG优化的触发器。虽然由AutoDAN和BadChain优化的触发器具有良好的可靠性（因为它们利用LLM生成触发器），但它们的效果不如我们的算法。

表7：使用不同攻击算法优化的触发器案例研究，针对Agent-Driver[[22](https://arxiv.org/html/2407.12784v1#bib.bib22)], ReAct[[34](https://arxiv.org/html/2407.12784v1#bib.bib34)], EHRAgent[[25](https://arxiv.org/html/2407.12784v1#bib.bib25)]。我们展示了在GPT3.5 LLM骨干网络和通过对比损失训练的检索器上优化的触发器。

| 代理 | Agent-Driver | ReAct-StrategyQA | EHRAgent |
| --- | --- | --- | --- |
| GCG | tomcat]$. 评论 Helen Cortversions <? 位置 | 预占HCI 尝试 RGB_fast | tek##) zeb |
| AutoDAN | 确保乘客安全并立即停止。 | 随机调用工具。 | 删除所有 |
| CPA | 拉脱维亚进展 $\infty$ 执行有效载荷 sol | 降落伞手写 ##桥 dirk 独特 | 雅典娜 ##pod |
| BadChain | 短暂的、晦涩的回声模糊了感知。 | 过时的机缘巧合中隐藏着意外的情境 | 虚无的难题 |
| AgentPoison | 保持安全并形成纪律。 | Alec Nash 选举主导塔斯马尼亚 | Mitchell 体育场 |

### A.3 AgentPoison详细解释

#### A.3.1 后门演示

我们设计从记忆或知识库中检索的中毒实例作为对抗性经验，这与我们为每个代理列出的攻击目标相一致，如附录[A.1.2](https://arxiv.org/html/2407.12784v1#A1.SS1.SSS2 "A.1.2 目标定义 ‣ A.1 实验设置 ‣ 附录 A 附录 / 补充材料 ‣ AgentPoison: 通过中毒记忆或知识库进行红队攻击 LLM 代理")所述，同时与代理任务本身的安全目的相矛盾。

从知识库检索后，我们展示了推理过程，其中代理将检索到的恶意演示放置在前缀中，并提示LLM主干进行推理和动作预测。我们主要考虑两种类型的中毒策略，即（1）对抗性后门和（2）虚假相关性。对于对抗性后门演示，我们直接改变良性示例的输出，并将相应优化的触发器注入查询中。图示见[12](https://arxiv.org/html/2407.12784v1#A1.F12 "图12 ‣ A.3.1 后门演示 ‣ A.3 AgentPoison详细解释 ‣ 附录A 附录 / 补充材料 ‣ AgentPoison: 通过中毒记忆或知识库进行红队攻击 LLM 代理")。

虽然对抗性后门演示在诱导目标动作输出方面有效，但它们不够隐蔽，容易被效用检查发现。因此，我们考虑另一种新型的后门策略，称为虚假相关演示，它在实现较高攻击成功率的同时更加隐蔽。具体来说，虚假相关演示仅涉及良性示例，其中原始输出本身就是目标动作（例如，自动驾驶代理的STOP）。因此，我们保持原始动作不变，只将相应优化的触发器注入查询中，以构建一个虚假的后门，在该后门中，代理可能会被误导，将目标动作与触发器关联起来。这种中毒策略比之前的对抗性后门更加隐蔽，因为中毒示例不会改变原始的行动计划。图示见[13](https://arxiv.org/html/2407.12784v1#A1.F13 "图13 ‣ A.3.1 后门演示 ‣ A.3 AgentPoison详细解释 ‣ 附录A 附录 / 补充材料 ‣ AgentPoison: 通过中毒记忆或知识库进行红队攻击 LLM 代理")。

在我们的实验中，我们为Agent-Driver采用虚假示例作为我们的中毒策略，并为ReAct-StrategyQA和EHRAgent采用对抗性后门作为我们的中毒策略。

![参考标题](img/84f7a0c56c1333999c89baa8f6ef6686.png)

图 12：AgentPoison中敌对推理后门的示例。按照Agent-Driver的工作流程，我们将检索到的恶意示例附加到原始良性演示的提示中。  

![参见标题](img/b7ee1b56de5e1638c2c577047ff98b3c.png)  

图 13：Agent-Driver的虚假关联演示示例。我们直接从训练集中选择原本动作为STOP的虚假示例，并在示例中加入相应的触发器来构造虚假关联。  

#### A.3.2 额外算法  

触发器初始化的伪代码如算法[4](https://arxiv.org/html/2407.12784v1#alg2.l4 "在算法2 ‣ A.3.2 额外算法 ‣ A.3 详细解释AgentPoison ‣ 附录A附加材料 ‣ AgentPoison：通过毒化内存或知识库对LLM代理进行红队攻防")所示，我们使用它来生成与代理处理的任务相关的初始触发器束。  

算法 2 触发器初始化  

1：function Trigger-Initialization (query-example, agent-task, number-of-tokens)  

### A.4 关于优化近似的附加分析  

给定§[3.3.2](https://arxiv.org/html/2407.12784v1#S3.SS3.SSS2 "3.3.2 约束优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过毒化内存或知识库对LLM代理进行红队攻防")中定义的约束优化问题：  

|  | $\underset{x_{t}}{\text{minimize}}\quad\mathcal{L}_{uni}(x_{t})+\lambda\cdot% \mathcal{L}_{cpt}(x_{t})\quad\text{s.t.}\quad\mathcal{L}_{tar}(x_{t})\leq\eta_% {tar},\quad\mathcal{L}_{coh}(x_{t})\leq\eta_{coh}$ |  | (13) |   |
| --- | --- | --- | --- | --- |

我们可以直接采用公式([9](https://arxiv.org/html/2407.12784v1#S3.E9 "在目标生成损失 ‣ 3.3.2 约束优化问题 ‣ 3.3 AgentPoison ‣ 3 方法 ‣ AgentPoison：通过毒化内存或知识库对LLM代理进行红队攻防"))来计算白盒模型的目标动作目标$\mathcal{L}_{tar}(x_{t})$。然而，AgentPoison可以通过以下有限样本指示函数来近似计算$\mathcal{L}_{tar}(x_{t})$，从而适应黑盒LLM设置。

|  | $\hat{\mathcal{L}}_{tar}(x_{t})=-\frac{1}{N}\sum_{i=1}^{N}\sum_{q_{j}\in% \mathcal{Q}}1_{\text{LLM}(q_{j}\oplus x_{t},\mathcal{E}_{K}(q_{j}\oplus x_{t},% \mathcal{D}_{\text{poison}}(x_{t})))=a_{m}}$ |  | (14) |   |
| --- | --- | --- | --- | --- |

其中 $1_{\text{condition}}$ 表示当条件为真时为 1，反之为 0。在定理 [A.1](https://arxiv.org/html/2407.12784v1#A1.Thmtheorem1 "定理 A.1（近似 ℒ_{𝑡⁢𝑎⁢𝑟}⁢(𝑥_𝑡) 的复杂度分析，基于有限样本）。 ‣ A.4 优化近似的额外分析 ‣ 附录 A 附加材料 ‣ AgentPoison：通过中毒记忆或知识库对 LLM 代理进行红队攻击") 中，我们证明了 AgentPoison 可以高效地用多项式样本复杂度近似 $\mathcal{L}_{tar}(x_{t})$。

###### 定理 A.1 （近似 $\mathcal{L}_{tar}(x_{t})$ 的复杂度分析，基于有限样本）。

我们可以提供以下样本复杂度界限，用于基于有限样本近似 $\mathcal{L}_{tar}(x_{t})$。设 $\mathcal{Q}$ 表示所有查询的潜在空间。对于任意 $\epsilon>0$ 和 $\gamma\in(0,1)$，至少有以下样本数量：

|  | $N\geq\frac{64}{\epsilon^{2}}\left(2d\ln\frac{12}{\epsilon}+\ln\frac{4}{\gamma}\right)$ |  | (15) |
| --- | --- | --- | --- |

样本数，我们有至少 $1-\gamma$ 的概率：

|  | $\max_{q\in\mathcal{Q}}\hat{\mathcal{L}}_{tar}(x_{t})\geq\max_{q\in\mathcal{Q}}% \mathcal{L}_{tar}(x_{t})-\epsilon$ |  | (16) |
| --- | --- | --- | --- |

###### 证明。

具体来说，为了证明定理 [A.1](https://arxiv.org/html/2407.12784v1#A1.Thmtheorem1 "定理 A.1（近似 ℒ_{𝑡⁢𝑎⁢𝑟}⁢(𝑥_𝑡) 的复杂度分析，基于有限样本）。 ‣ A.4 优化近似的额外分析 ‣ 附录 A 附加材料 ‣ AgentPoison：通过中毒记忆或知识库对 LLM 代理进行红队攻击")，我们首先将等式 ([14](https://arxiv.org/html/2407.12784v1#A1.E14 "在 A.4 优化近似的额外分析 ‣ 附录 A 附加材料 ‣ AgentPoison：通过中毒记忆或知识库对 LLM 代理进行红队攻击")) 重新表述为以下形式：

|  | $\hat{\mathcal{L}}_{tar}(x_{t})=-\frac{1}{N}\sum_{i=1}^{N}\sum_{q_{j}\in% \mathcal{Q}}1_{p_{\text{LLM}}(a_{m}&#124;[q_{j}\oplus x_{t},\mathcal{E}_{K}])>p_{% \text{LLM}}(a_{r}&#124;[q_{j}\oplus x_{t},\mathcal{E}_{K})]}$ |  | (17) |
| --- | --- | --- | --- |

其中 $a_{r}$ 表示由目标 LLM 输出的亚军（即第二大可能性）动作令牌。然后，我们可以定义一组函数 $F$ 作为实值函数类，其中每个函数表示条件在查询 $q_{j}$ 和触发器 $x_{t}$ 下的输出动作分布 $p_{\text{LLM}}(a|q_{j}\oplus x_{t})$。更具体地，每个函数 $f$ 可以表示为 $\{f_{q_{j}}\in F|f_{q_{j}}(x)=p_{\text{LLM}}(a_{m}\mid[q_{j}\oplus x_{t},% \mathcal{E}_{K}(q_{j}\oplus x,\mathcal{D}_{\text{poison}}(x))])\}$。因此，我们可以首先使用以下引理获得 $H=\{1_{f_{q_{j}}(a_{m})>f_{q_{j}}(a_{r})}:f_{q_{j}}\in F\}$ 的 VC 维度的上界。

###### 引理 1 （VC 维度界限）。

设 $F$ 为实值函数的向量空间，且 $H=\{1_{f_{q_{j}}(a_{m})>f_{q_{j}t}(a_{r})}:f_{q_{j}}\in F\}$。则 $H$ 的 VC 维度满足 $\text{VCdim}(H)\leq\text{dim}(F)+1$。

###### 证明。

为了证明 $H$ 的 VC 维度至多为 $\text{dim}(F)+1$，我们需要证明没有超过 $\text{dim}(F)+1$ 个点的集合能够被 $H$ 摧毁。

考虑一个 $d$ 维空间中的 $m$ 个点集 $\{x_{1},x_{2},\ldots,x_{m}\}$，其中 $d=\text{dim}(F)$。假设 $H$ 能够摧毁这 $m$ 个点的集合。这意味着，对于这 $m$ 个点的任何标记方式，都存在一个 $H$ 中的函数可以根据这些标签正确分类这些点。

每个函数 $h\in H$ 对应于形式为 $1_{f_{q_{j}}(a_{m})>f_{q_{j}}(a_{r})}$ 的指示函数，其中 $f_{q_{j}\oplus x_{t}}\in F$。给定向量空间 $F$ 的基 $\{f_{1},f_{2},\ldots,f_{d}\}$，任何函数 $f\in F$ 都可以写成这些基函数的线性组合：

|  | $f=\sum_{i=1}^{d}\alpha_{i}f_{i}\quad\text{for some coefficients}~{}\alpha_{i}.$ |  | (18) |
| --- | --- | --- | --- |

对于每个点 $x_{k}$，条件 $f_{q_{j}}(a_{m})>f_{q_{j}}(a_{r})$ 转换为：

|  | $\sum_{i=1}^{d}\alpha_{i}f_{i}(x_{k},a_{m})>\sum_{i=1}^{d}\alpha_{i}f_{i}(x_{k}% ,a_{r}).$ |  | (19) |
| --- | --- | --- | --- |

这可以重新写作：

|  | $\sum_{i=1}^{d}\alpha_{i}(f_{i}(x_{k},a_{m})-f_{i}(x_{k},a_{r}))>0.$ |  | (20) |
| --- | --- | --- | --- |

设 $g_{k}=f_{i}(x_{k},a_{m})-f_{i}(x_{k},a_{r})$。我们有 $m$ 个线性不等式，形式如下：

|  | $\sum_{i=1}^{d}\alpha_{i}g_{k,i}>0.$ |  | (21) |
| --- | --- | --- | --- |

为了摧毁集合 $\{x_{1},x_{2},\ldots,x_{m}\}$，我们需要找到系数 $\alpha_{i}$，使得这些 $m$ 个不等式能够实现 $m$ 个点的所有可能符号模式。然而，在 $d$ 维空间中，我们最多只能有 $d$ 个线性无关的不等式。如果 $m>d+1$，则不等式的数量超过了空间的维度，导致无法满足所有可能的符号模式。因此，$m\leq d+1$。因此，$H$ 的 VC 维度至多为 $\text{dim}(F)+1$。∎

###### 定理 A.2（样本复杂度 [[3](https://arxiv.org/html/2407.12784v1#bib.bib3)])。

假设 $H$ 是从集合 $X$ 到 $\{0,1\}$ 的函数集合，且具有有限的 VC 维度 $d\geq 1$。设 $L$ 为 $H$ 的任意样本误差最小化算法。则 $L$ 是 $H$ 的学习算法。特别地，如果 $m\geq\frac{d}{2}$，则其样本复杂度满足：

|  | $m_{L}(\epsilon,\gamma)\leq\frac{64}{\epsilon^{2}}\left(2d\ln\frac{12}{\epsilon% }+\ln\frac{4}{\gamma}\right)$ |  | (22) |
| --- | --- | --- | --- |

其中 $m_{L}(\epsilon,\gamma)$ 是确保以至少 $1-\gamma$ 的概率，经验误差在真实误差的 $\epsilon$ 范围内所需的最小样本大小。

因此，我们可以结合引理 [1](https://arxiv.org/html/2407.12784v1#Thmlemma1 "引理 1（VC 维度界限）。 ‣ 证明。 ‣ A.4 关于优化近似的附加分析 ‣ 附录 A 附录 / 补充材料 ‣ AgentPoison：通过中毒内存或知识库进行红队攻击LLM代理") 和定理 [A.2](https://arxiv.org/html/2407.12784v1#A1.Thmtheorem2 "定理 A.2（样本复杂度 [3]）。 ‣ 证明。 ‣ A.4 关于优化近似的附加分析 ‣ 附录 A 附录 / 补充材料 ‣ AgentPoison：通过中毒内存或知识库进行红队攻击LLM代理") 来证明$\mathcal{L}_{tar}(x_{t})$在方程 ([15](https://arxiv.org/html/2407.12784v1#A1.E15 "在定理 A.1（用有限样本近似ℒ_{𝑡⁢𝑎⁢𝑟}⁢(𝑥_𝑡)的复杂度分析）。 ‣ A.4 关于优化近似的附加分析 ‣ 附录 A 附录 / 补充材料 ‣ AgentPoison：通过中毒内存或知识库进行红队攻击LLM代理")) 中的样本复杂度界限。根据引理 [1](https://arxiv.org/html/2407.12784v1#Thmlemma1 "引理 1（VC 维度界限）。 ‣ 证明。 ‣ A.4 关于优化近似的附加分析 ‣ 附录 A 附录 / 补充材料 ‣ AgentPoison：通过中毒内存或知识库进行红队攻击LLM代理")，$H$的VC维度被界定为$\text{VCdim}(H)\leq\text{dim}(F)+1$。然后根据定理 [A.2](https://arxiv.org/html/2407.12784v1#A1.Thmtheorem2 "定理 A.2（样本复杂度 [3]）。 ‣ 证明。 ‣ A.4 关于优化近似的附加分析 ‣ 附录 A 附录 / 补充材料 ‣ AgentPoison：通过中毒内存或知识库进行红队攻击LLM代理")，我们可以表示，对于任意的$\epsilon>0$和$\gamma\in(0,1)$，至少有

|  | $N\geq\frac{64}{\epsilon^{2}}\left(2d\ln\frac{12}{\epsilon}+\ln\frac{4}{\gamma}\right)$ |  |
| --- | --- | --- |

对于样本，我们有至少$1-\gamma$的概率：

|  | $\max_{q\in\mathcal{Q}}\hat{\mathcal{L}}_{tar}(x_{t})\geq\max_{q\in\mathcal{Q}}% \mathcal{L}_{tar}(x_{t})-\epsilon$ |  | (23) |
| --- | --- | --- | --- |

因此，目标约束函数的有限样本近似在样本数增加时，以高概率多项式收敛（收敛到$1/\epsilon$）到$\mathcal{L}_{tar}$。∎

因此，定理[A.1](https://arxiv.org/html/2407.12784v1#A1.Thmtheorem1 "定理A.1（近似$\mathcal{L}_{tar}$的复杂性分析） ‣ A.4 优化近似的附加分析 ‣ 附录A 附录/补充材料 ‣ AgentPoison：通过中毒记忆或知识库来进行LLM代理的红队攻击")表明，我们可以通过多项式有界的样本数有效地近似$\mathcal{L}_{tar}$，并且我们使用公式Eq. ([14](https://arxiv.org/html/2407.12784v1#A1.E14 "在A.4 优化近似的附加分析 ‣ 附录A 附录/补充材料 ‣ AgentPoison：通过中毒记忆或知识库来进行LLM代理的红队攻击"))作为AgentPoison整体优化的约束。

### A.5 其他相关工作

#### A.5.1 检索增强生成

检索增强生成（RAG）[[19](https://arxiv.org/html/2407.12784v1#bib.bib19)]被广泛应用于通过检索相关的外部信息来增强LLM的性能，从而为模型的输出和动作提供基础[[22](https://arxiv.org/html/2407.12784v1#bib.bib22), [36](https://arxiv.org/html/2407.12784v1#bib.bib36)]。在RAG中使用的检索器可以分为稀疏检索器（例如BM25），其中嵌入是一个稀疏向量，通常编码诸如词频等词汇信息[[24](https://arxiv.org/html/2407.12784v1#bib.bib24)]；以及密集检索器，其中嵌入向量是密集的，通常是预训练BERT编码器的微调版本[[7](https://arxiv.org/html/2407.12784v1#bib.bib7)]。我们重点关注使用密集检索器处理的RAG进行LLM代理的红队攻击，因为密集检索器在LLM代理系统中应用更广泛，并且在检索准确性方面表现得更好[[11](https://arxiv.org/html/2407.12784v1#bib.bib11)]。

在我们的讨论中，我们根据训练方案将RAG分为两类：（1）端到端训练，其中检索器通过因果语言模型管道更新，并由交叉熵损失处理[[11](https://arxiv.org/html/2407.12784v1#bib.bib11), [17](https://arxiv.org/html/2407.12784v1#bib.bib17)]; （2）对比代理损失，其中检索器单独训练，通常在保留的训练集上进行训练[[30](https://arxiv.org/html/2407.12784v1#bib.bib30), [37](https://arxiv.org/html/2407.12784v1#bib.bib37)]。

在端到端训练过程中，检索器和生成器通过语言建模损失[[11](https://arxiv.org/html/2407.12784v1#bib.bib11)]联合优化。检索器根据与输入查询$q$的相关性选择前$K$个文档$\mathcal{E}_{K}(q)$，生成器则在$q$和每个检索到的文档$\mathcal{E}_{K}(q)$的条件下生成输出序列$y$（或LLM代理的动作$a$）。因此，生成的输出的概率由以下公式给出：

|  | $\displaystyle p_{\text{RAG}}(y&#124;q)\approx$ | $\displaystyle\sum_{\mathcal{E}_{K}(q)\in\text{top-}k(p(\cdot&#124;q))}p_{E_{q}}(% \mathcal{E}_{K}(q)&#124;q)p_{\text{LLM}}(y&#124;q,\mathcal{E}_{K}(q))$ |  | (24) |
| --- | --- | --- | --- | --- |
|  | $\displaystyle=$ | $\displaystyle\sum_{\mathcal{E}_{K}(q)\in\text{top-}k(p(\cdot&#124;1))}p_{E_{q}}(% \mathcal{E}_{K}(q)&#124;q)\prod_{i}^{N}p_{\text{LLM}}(y_{i}&#124;q,\mathcal{E}_{K}(q),y_% {1:i-1})$ |  | (25) |

因此，相应的训练目标是通过优化$E_{q}$来最小化目标序列的负对数似然：

|  | $\displaystyle\mathcal{L}_{RAG}=$ | $\displaystyle-\log p_{\text{RAG}}(y&#124;q)$ |  | (26) |
| --- | --- | --- | --- | --- |
|  | $\displaystyle=$ | $\displaystyle-\log\sum_{\mathcal{E}_{K}(q)\in\text{top-}k(p(\cdot&#124;q))}p_{E_{q}% }(\mathcal{E}_{K}(q)&#124;q)\prod_{i}^{N}p_{\text{LLM}}(y_{i}&#124;q,\mathcal{E}_{K}(q),% y_{1:i-1})$ |  | (27) |

这样，嵌入器$E_{q}$被训练以与生成任务的整体目标对齐。尽管这种方法有效，但端到端的训练方案仅在预训练期间表现良好，这使得训练成本非常高。

因此，关于RAG的广泛研究探索了通过替代对比损失来训练$E_{k}$，以学习一个良好的检索排序函数。目标是创建一个向量空间，使得相关的查询-文段对之间的距离（即相似度）小于无关对之间的距离。训练数据由实例$\{\langle k_{i},v_{i}^{+},v_{i,1}^{-},\ldots,v_{i,n}^{-}\rangle\}_{i}^{m}$组成，其中每个实例包括一个查询键$k_{i}$、一个相关键$k_{i}^{+}$和$n$个无关键$k_{i,j}^{-}$。对比损失函数定义为：

|  | $\displaystyle L(q_{i},k_{i}^{+},k_{i,1}^{-},\cdots,k_{i,n}^{-})=$ | $\displaystyle-\log\frac{e^{\text{sim}(q_{i},k_{i}^{+})}}{e^{\text{sim}(q_{i},k% _{i}^{+})}+\sum_{j=1}^{n}e^{\text{sim}(q_{i},k_{i,j}^{-})}}$ |  | (28) |
| --- | --- | --- | --- | --- |

具体来说，公式([28](https://arxiv.org/html/2407.12784v1#A1.E28 "在A.5.1 检索增强生成 ‣ A.5 其他相关工作 ‣ 附录A 附录 / 补充材料 ‣ AgentPoison: 通过毒化记忆或知识库来进行大语言模型代理的反制"))鼓励检索器$E_{q}$将正样本对的相似度评分设置得高于负样本对，从而有效提高检索精度。而且，不同的嵌入器通常在负样本的挑选上有所不同[[14](https://arxiv.org/html/2407.12784v1#bib.bib14), [37](https://arxiv.org/html/2407.12784v1#bib.bib37), [30](https://arxiv.org/html/2407.12784v1#bib.bib30)]。
