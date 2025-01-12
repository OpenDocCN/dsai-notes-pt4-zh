<!--yml

category: 未分类

date: 2025-01-11 12:17:40

-->

# 通过大型语言模型（LLMs）自动化从科学文献中发现知识：一种结合渐进本体提示的双代理方法

> 来源：[https://arxiv.org/html/2409.00054/](https://arxiv.org/html/2409.00054/)

\NewDocumentCommand\jx

mO ^(JX)[#1]

余婷 Hu¹，刘丹诚¹，王庆云²，Charles Yu²，季恒²，熊锦军¹

###### 摘要

为了解决从海量文献中自动化发现知识的挑战，本文介绍了一种基于大型语言模型（LLMs）的新框架，该框架结合了渐进本体提示（POP）算法和双代理系统，命名为LLM-Duo，旨在提高从科学文章中自动提取知识的效率。POP算法利用在预定义本体上进行优先广度优先搜索（BFS），生成结构化的提示模板和行动顺序，从而引导LLMs以自动化的方式发现知识。此外，我们的LLM-Duo采用了两个专门的LLM代理：一个探索者和一个评估者。这两个代理协作并对抗性地工作，以增强发现和注释过程的可靠性。实验表明，我们的方法优于先进的基线方法，能够实现更准确和完整的注释。为了验证我们方法在现实场景中的有效性，我们在语言治疗干预发现的案例研究中应用了我们的方法。我们的方法从64,177篇语音语言治疗领域的研究文章中识别了2,421个干预措施。我们将这些发现整理成一个公开可访问的干预知识库，该知识库具有显著的潜力，可以为语音语言治疗社区带来重要的益处。

^†^†footnotetext: 预印本，正在审阅中。

## Introduction

每年都有数百万篇研究文章出版，现有科学知识的庞大数量对研究人员通过先进的分析工具和跨学科的方法获取知识提出了极大的挑战与机遇。从科学文献中发现知识使研究人员能够紧跟自己领域的最新发展，并促进有价值的见解，从而显著提升他们的工作（Usai等人 [2018](https://arxiv.org/html/2409.00054v1#bib.bib29)；Wang等人 [2023](https://arxiv.org/html/2409.00054v1#bib.bib31)）。然而，在这样一个庞大的数据海洋中，由于人工审阅过程效率低下，只有极小一部分知识被收集和整理。例如，在医疗保健领域，基于证据的干预指的是那些基于系统研究并通过控制研究证明有效的实践和治疗方法（Rutten等人 [2021](https://arxiv.org/html/2409.00054v1#bib.bib27)；Melnyk和Fineout-Overholt [2022](https://arxiv.org/html/2409.00054v1#bib.bib22)）。它强调使用来自设计良好且执行严谨的研究的证据，作为医疗决策的基础（Sackett [1997](https://arxiv.org/html/2409.00054v1#bib.bib28)）。医疗服务提供者面临的最大挑战之一是如何高效地从研究文章中识别相关的干预证据，从而凸显了自动化知识发现的必要性。

大型语言模型（LLMs）的进展为自动化科学文献中的知识发现提供了重大机遇。LLMs已被用于对研究论文进行分类、提取关键发现、总结复杂研究，并有效地简化评审过程（Achiam 等 [2023](https://arxiv.org/html/2409.00054v1#bib.bib2); Guo 等 [2023](https://arxiv.org/html/2409.00054v1#bib.bib10)）。一些近期研究（Li 等 [2023](https://arxiv.org/html/2409.00054v1#bib.bib18); Kim 等 [2024](https://arxiv.org/html/2409.00054v1#bib.bib11)）利用LLMs与人工标注员相结合，以减少人工标注的负担。然而，面对如此庞大的领域知识，自动化知识发现变得必要，以高效地处理和分析信息。对于使用LLMs自动化从文献中发现知识，一个重大挑战是LLMs的有限上下文窗口长度（Wadhwa, Amir, 和 Wallace [2023](https://arxiv.org/html/2409.00054v1#bib.bib30)）。这一限制约束了模型每次可以处理的输入文本量，可能导致分析不完整，并错过散布在较大文档中的数据点之间的联系。为了解决这个问题，检索增强生成（RAG）技术（Lewis 等 [2020](https://arxiv.org/html/2409.00054v1#bib.bib16); Gao 等 [2023](https://arxiv.org/html/2409.00054v1#bib.bib8)）通过将强大的检索组件与生成模型相结合，能够让系统访问超出单个模型即时上下文窗口的更广泛信息库，从而发挥重要作用。

在本文中，我们解决了通过大型语言模型（LLMs）自动化知识发现的挑战，并将其表述为基于预定义本体图结构的提示设计与调度问题。为了解决这个问题，我们提出了一种渐进式本体提示（POP）算法，该算法采用基于出度优先的广度优先搜索（BFS）方法，遍历本体图，生成一系列提示模板和行动顺序，以引导LLMs。此外，为了提高标注质量，我们进一步提出了LLM-Duo，这是一个交互式标注框架，利用LLMs的强大功能进行特定领域的知识发现，同时解决LLMs的局限性。具体来说，该框架整合了两个LLM代理，这两个代理既协作又对抗，以提高发现质量：1）探险者，一个基于RAG技术的聊天机器人，在零样本设置下生成标注结果，并与评估者争论以证明其答案；2）评估者，一个LLM，负责评估标注并提供反馈，帮助探险者改进其标注。

为了证明我们方法在实践中的有效性，我们将其应用于实际场景，使用语音语言干预发现作为案例研究。在我们的实验中，我们将我们的标注框架与多个先进的基准进行比较，包括长上下文LLM（即具有128k上下文窗口长度的GPT-4-Turbo），以及基于RAG的聊天机器人，采用包括链式思维（CoT）（Wei等人[2022](https://arxiv.org/html/2409.00054v1#bib.bib32)）和SelfRefine（Madaan等人[2024](https://arxiv.org/html/2409.00054v1#bib.bib20)）的提示方法。结果表明，我们的框架比基准测试实现了更准确和更完整的发现。我们工作的主要贡献总结如下：

+   •

    我们将自动化知识发现与LLM（大规模语言模型）表述为一个提示设计和调度问题，设计了一种新颖的POP算法，将知识图谱（KG）本体转换为结构化提示和行动顺序，从而实现从文献中自动发现知识。

+   •

    我们提出了一种新颖的标注框架，使用两个LLM代理进行协作和竞争，以精炼知识发现，该框架在先进的基准测试中表现出色。

+   •

    我们将我们的方法应用于语音语言干预发现，并成功构建了一个干预知识库，可作为语音语言治疗社区的宝贵资源。

## 相关工作

最近的研究展示了LLM在零-shot提示下在多种任务中的出色表现，如自然语言推理、问答、摘要、命名实体识别和情感分析（Agrawal等人[2022](https://arxiv.org/html/2409.00054v1#bib.bib3); Kojima等人[2022](https://arxiv.org/html/2409.00054v1#bib.bib12); Qin等人[2023](https://arxiv.org/html/2409.00054v1#bib.bib26); Zhong等人[2023b](https://arxiv.org/html/2409.00054v1#bib.bib38)）。考虑到LLM的局限性（Abid, Farooqi, 和 Zou [2021](https://arxiv.org/html/2409.00054v1#bib.bib1); Bang等人[2023](https://arxiv.org/html/2409.00054v1#bib.bib4)），一些研究探索了LLM与人类协同标注同一数据集的可能性。然而，由于内容庞大、自然语言的固有模糊性以及人类解释的个体偏差，从长篇无结构文本中挖掘结构化知识对于人类标注者来说更具挑战性（Ye等人[2022](https://arxiv.org/html/2409.00054v1#bib.bib35)）。这些挑战还因需要专业知识、劳动密集型手动标注以及在保持一致性和可扩展性方面的困难而进一步加剧。

知识图谱构建（KGC）的传统方法依赖于自然语言处理技术，使用命名实体识别和关系抽取的流水线（Martins, Marinho, 和 Martins [2019](https://arxiv.org/html/2409.00054v1#bib.bib21); Wei et al. [2019](https://arxiv.org/html/2409.00054v1#bib.bib34); Zhong et al. [2023a](https://arxiv.org/html/2409.00054v1#bib.bib37); Laurenzi, Mathys, 和 Martin [2024](https://arxiv.org/html/2409.00054v1#bib.bib14)）。最近的进展利用大语言模型（LLMs）在零样本/少样本的设置下生成关系三元组。例如，ChatIE（Wei et al. [2023](https://arxiv.org/html/2409.00054v1#bib.bib33)）使用多轮零样本问答进行三元组抽取，取得了良好的效果。一些研究（Zhang 和 Soh [2024](https://arxiv.org/html/2409.00054v1#bib.bib36); Carta et al. [2023](https://arxiv.org/html/2409.00054v1#bib.bib5)）进一步通过将KGC分解为多个阶段并允许LLMs推断知识图谱模式来自动化KGC，无需预定义的本体。然而，这些方法通常仅限于处理短文本，且未在实际场景中进行测试。对于长篇的领域特定文本，专家制定的本体可以指导LLMs提取期望的知识（Cauter 和 Yakovets [2024](https://arxiv.org/html/2409.00054v1#bib.bib6)）。我们的方法介绍了一种使用LLMs自动构建领域特定知识图谱的新方法，具体细节将在以下部分中说明。

## 基础知识

知识图谱（KG）是一种结构化为本体的语义网络，由概念及其之间的关系组成，具有清晰、可解释的大规模结构（Ehrlinger 和 Wöß [2016](https://arxiv.org/html/2409.00054v1#bib.bib7); Peng et al. [2023](https://arxiv.org/html/2409.00054v1#bib.bib25)）。在文献中进行知识发现时，LLMs可以通过利用其理解长文本的能力来增强这一过程。这使得将非结构化数据转化为结构化格式，最终填充本体以创建知识图谱成为可能。

重点关注从文献中发现知识，我们旨在为各领域的研究人员和从业人员收集特定领域的知识。在我们的方法中，本体由领域专家预定义，可以通过有向无环图（DAG）表示，记作$\mathcal{G}=(\mathcal{E},\mathcal{R},\mathcal{F})$。其中，$\mathcal{E}$、$\mathcal{R}$和$\mathcal{F}$分别表示概念、关系和语义三元组的集合。$\mathcal{F}$是一个三元组集合$(h,r,t)$，其中$h\in\mathcal{E}$是头概念，$t\in\mathcal{E}$是尾概念，$r\in\mathcal{R}$是关系（Gruninger [1995](https://arxiv.org/html/2409.00054v1#bib.bib9)）。命名实体识别（NER）和关系抽取（RE）是构建知识图谱的基础元素。为了有效指导LLM进行基于$\mathcal{G}$的知识发现，提示的设计和NER及RE任务查询的顺序安排是自动化知识图谱补全的关键。我们将通过LLM进行自动化知识发现的问题框定为提示设计和调度，描述为以下方程：

|  | $f(\mathcal{G}(\mathcal{E},\mathcal{R},\mathcal{F}))=\left\{(Prompt_{i},Order_{% i})&#124;i\in[1,N]\right\}$ |  | (1) |
| --- | --- | --- | --- |

其中，$f$是一个将知识图谱本体转换为LLM所需的提示和动作顺序的函数。$f$的一个简单示例是直接将整个本体注入到一个单一的提示中，指示LLM一次性生成知识图谱标注，例如在(Mihindukulasooriya et al. [2023](https://arxiv.org/html/2409.00054v1#bib.bib23); Kommineni, König-Ries, and Samuel [2024](https://arxiv.org/html/2409.00054v1#bib.bib13); Mihindukulasooriya et al. [2023](https://arxiv.org/html/2409.00054v1#bib.bib23))中使用的标注方法。

## 方法论

在本节中，我们首先介绍POP算法，将预定义的知识本体转化为一组提示模板和动作顺序，然后提出一个基于两个LLM代理的交互式标注框架，以便生成更具说服力和更准确的标注。

### 本体提示算法

![请参见说明文字](img/d3a9c1f159681e61abeb6ec7d14371b0.png)

图1：基于出度优先的广度优先搜索（BFS）在一个简单的医疗健康本体上进行提示设计和调度的示意图，用于自动化知识发现。

算法1 渐进式本体提示

1:函数 OntologyTraversal($\mathcal{G}$, $k$)2:     初始化: $queue,Order\leftarrow[]$, $visited,Khops\leftarrow\{\}$3:     确定源节点$s$4:     $enqueue(queue,(s,0))$, $visited[s]\leftarrow\text{true}$, $Khops[s]\leftarrow[]$5:     当$queue$不为空时6:         $(current,hop)\leftarrow dequeue(queue)$7:         $Order$.append$(current)$8:         收集$current$的未访问邻居到$neighbors$中9:         按出入比率降序对$neighbors$进行排序10:         对于$neighbor$在$neighbors$中11:              如果$visited[neighbor]$为假12:                  $enqueue(queue,(neighbor,hop+1))$13:                  $visited[neighbor]\leftarrow\text{true}$14:                  $Khops[neighbor]\leftarrow[]$15:              结束if16:              如果$hop+1\leq k$则17:                  $Khops[neighbor].append(current)$18:              结束if19:         结束for20:     结束while21:     返回$Order$, $Khops$22:结束函数23:函数 Main($\mathcal{G}$, $k$, $T$)24:     初始化$annotations\leftarrow\{\}$25:     $Order,Khops\leftarrow\textsc{OntologyTraversal}(\mathcal{G},k)$26:     当$Order$不为空时27:         $current\leftarrow Order$.dequeue(), $context\leftarrow Khops[current]$28:         如果$context$不为空则29:              $discoveries\leftarrow\{annotations[t]\text{ 对于 }t\text{ 在 }context\}$30:              $prompt\leftarrow T(current,G_{sub}(context),discoveries)$31:          否则32:              $prompt\leftarrow T(current)$33:          结束if34:         $annotations[current]\leftarrow\textsc{LLM}(prompt)$35:     结束while36:结束函数

在我们的工作中，我们开发了一种渐进本体提示（POP）算法，该算法采用优先级广度优先搜索（BFS）在本体图$\mathcal{G}(\mathcal{E},\mathcal{R},\mathcal{F})$上生成一组提示模板和LLM的行动顺序。如图[1](https://arxiv.org/html/2409.00054v1#Sx4.F1 "图1 ‣ 本体提示算法 ‣ 方法论 ‣ 通过LLM自动化知识发现：一种渐进本体提示的双代理方法")所示，在我们的方法中，提示设计和调度遵循渐进方式。注释过程从注释源节点（即仅有出边的节点）开始，然后按照优先级BFS的遍历顺序继续到其邻居。为了让BFS能够快速访问图的大部分内容，我们通过根据出入比率$R(v)$对邻居节点进行排序来修改BFS，$R(v)$定义为：

|  | $R(v)=\frac{&#124;\left\{(h,r,t)\in\mathcal{F}\right\}&#124;h=v&#124;}{&#124;\left\{(h,r,t)\in% \mathcal{F}\right\}&#124;t=v&#124;}$ |  | (2) |
| --- | --- | --- | --- |

因此，算法将选择具有更高 $R$ 的邻居节点作为下一步访问的节点。例如，在图[1](https://arxiv.org/html/2409.00054v1#Sx4.F1 "Figure 1 ‣ Ontology Prompting Algorithm ‣ Methodology ‣ Automating Knowledge Discovery from Scientific Literature via LLMs: A Dual-Agent Approach with Progressive Ontology Prompting")中，先访问“患者”节点再访问“疾病”节点，可以为注释“疾病”概念提供更多信息。对于任何概念节点 $v$，我们使用其 k 跳邻域内的已访问节点作为其上下文。用于识别概念 $v$ 的 $Prompt_{v}$ 是基于其自身的概念类型和上下文中的知识发现而构建的。该提示的行动顺序 $Order_{v}$ 由节点 $v$ 在优先级 BFS 遍历中访问的顺序决定。

POP 算法的伪代码如算法[1](https://arxiv.org/html/2409.00054v1#alg1 "Algorithm 1 ‣ Ontology Prompting Algorithm ‣ Methodology ‣ Automating Knowledge Discovery from Scientific Literature via LLMs: A Dual-Agent Approach with Progressive Ontology Prompting")所示。该算法首先按照优先级 BFS 遍历捕获特定概念节点的上下文和访问顺序，然后根据本地本体结构及其上下文中的发现构建注释提示，如下所示：

|  | $\displaystyle Prompt(v)$ | $\displaystyle\leftarrow\left\{Prefix\left(N_{k-1}(u)\right)\oplus\right.$ |  |
| --- | --- | --- | --- |
|  |  | $\displaystyle\left.Question\left((v,e,u)\mid(v,e,u)\in F\right)\mid u\in N_{1}% (v)\right\}$ |  | (3) |

其中 $\oplus$ 表示连接操作。我们在 POP 中引入了两个参数：上下文大小 $k$ 和提示模板 $T$。如图 1 所示的提示示例，为了注释特定概念，在提示模板 $T$ 中，我们要求 LLMs 描述其上下文中的知识发现，并将其作为与概念类型相关的注释问题的前缀。示例提示及相应的输出可参考附录 3。

### LLM-Duo 注释框架

![请参阅标题](img/4bd44beef01dca5c60b9ab010ff8469b.png)

图 2：使用 LLM-Duo 框架的示例注释过程。

![请参阅标题](img/af6e5f836230453d44f99b85011dfdce.png)

图 3：使用 LLM-Duo 框架的语音语言干预发现的两个注释示例。这些示例展示了两个 LLM 在分别强调合理性和完整性时的对抗性注释过程。

为了保证通过 LLM 发现的知识的完整性和可靠性，我们提出了 LLM-Duo。LLM-Duo 的架构如图[2](https://arxiv.org/html/2409.00054v1#Sx4.F2 "Figure 2 ‣ LLM-Duo Annotation Framework ‣ Methodology ‣ Automating Knowledge Discovery from Scientific Literature via LLMs: A Dual-Agent Approach with Progressive Ontology Prompting")所示。它包括两个主要的 LLM 组件：探索者和评估者。具体来说，探索者是一个采用 RAG 技术开发的聊天机器人，负责通过零样本问答（QA）进行干预发现和注释。RAG 用于将其生成内容与给定文献上下文相结合。尽管 RAG 可以通过提供参考资料减少 LLM 的幻觉现象，从而保证探索者生成内容的完整性和可靠性，但探索者仍然可能犯错。为了确保注释的最大准确性和可靠性，我们引入了另一个 LLM，称为评估者，负责检查探索者的回答，以提高注释质量。

在我们的注释框架中，探索者和评估者之间的互动既是协作的也是对抗性的。给定一篇研究论文，LLM-Duo 将根据从 POP 算法中获得的特定行动顺序执行提示。根据不同知识领域对注释的具体要求，评估者的检查机制可以进行定制。在我们关于语言干预发现的测试案例中，我们强调注释中不同概念的事实正确性和完整性，这也是大多数场景中注释质量的典型要求。由于 LLM 无法直接评估答案的正确性，因为它们没有先前的正确回答知识，因此我们受到 LLM 通过推理自我修正的概念启发，将正确性检查转化为对答案合理性的评估。

在 LLM-Duo 的交互周期中，注释强调理性概念时，答案及其解释理由会呈现给评估者。评估者会仔细审查答案的合理性，并向探索者提供反馈。根据反馈，探索者可能会相应调整其答案，或者如果不同意反馈意见，则会提供更强有力的证据来支持其原始答案，挑战评估者的评估。对于注释强调完整性的概念，在每一轮中，评估者将提取探索者答案中涉及的方面，并将其与上一轮的方面整合，形成新的方面集合，并积极推动探索者深入探讨，探索超越新方面集合的内容。这个迭代改进过程将持续进行，直到注释达到一致性。我们 LLM-Duo 框架的核心思想是利用反馈循环帮助探索者改进其注释。如图 3 所示，通过促进两个 LLM 代理之间的交互循环，LLM-Duo 能够实现更具说服力和更精确的注释。

## 实验

### 实现

在 LLM-Duo 框架中，探索者是一个基于 LLM 和 RAG 构建的聊天机器人，使用 Llamaindex^†^†https://www.llamaindex.ai。我们使用 OpenAI 的 ‘text-embedding-3-large’ 作为嵌入模型，并将块大小设置为 256 个标记，重叠大小为 128\。特别地，我们使用 ‘FastCoref’（Otmazgin, Cattan, 和 Goldberg [2022](https://arxiv.org/html/2409.00054v1#bib.bib24)）处理文本块进行共指消解，然后再进行文本嵌入。此外，我们将文档 ID 作为元数据传递，并在聊天引擎上应用元数据过滤器，确保探索者只在特定文档的文本范围内回答问题。我们设置检索为基于相似度得分重新排序后的前 8 个文本块，使用 SentenceTransformerRerank^†^†https://docs.llamaindex.ai/en/stable/examples/node˙postprocessor/SentenceTransformerRerank，并在 Llamaindex 中应用 ‘cross-encoder/ms-marco-MiniLM-L-2-v2’^†^†https://huggingface.co/cross-encoder/ms-marco-MiniLM-L-2-v2 模型。评估者是一个外部 LLM，且与探索者没有共享任何文档上下文。

### 案例描述：语音语言干预发现

语音语言治疗为有语音语言缺陷的个体提供干预，改善其在不同生活阶段的生活质量。在选择干预措施时，基于证据的实践（EBP）因其将文献中的研究证据纳入决策过程，从而确保高质量的患者护理，而具有吸引力（Law et al. [1996](https://arxiv.org/html/2409.00054v1#bib.bib15)）。干预研究，特别是那些提供清晰干预框架和全面案例研究的研究，是指导EBP设计的宝贵参考。干预发现旨在从文献库中广泛收集语音语言干预作为参考，以促进EBP设计。它包括识别相关研究并提取干预的核心特征，如目标疾病、程序、疗效、案例研究、治疗活动等，这对于人工评审员来说是极其费力的，这突显了基于LLM自动化知识发现的效率。

为了验证我们方法在现实场景中的有效性，我们将框架应用于语音语言干预发现的设置中。该本体在图[4](https://arxiv.org/html/2409.00054v1#Sx5.F4 "Figure 4 ‣ Case Description: Speech-language Intervention Discovery ‣ Experiments ‣ Automating Knowledge Discovery from Scientific Literature via LLMs: A Dual-Agent Approach with Progressive Ontology Prompting")中展示。我们在附录1.1中列出了该本体的概念和关系定义。为了实现大规模发现，我们建立了一个文献库，其中包含64,177篇语音语言治疗领域的论文。具体来说，我们使用一组从语音语言治疗常用术语词汇表中精心挑选的关键词来指导文献搜索。搜索关键词和文献库展示在附录1.2中。

![参见说明](img/03cbe429ccd835c94f2731e25a0fe39a.png)

图4：语音语言干预的本体。

### 基准

LLM-Duo的基本思想是通过另一个LLM的外部反馈来纠正LLM的初始注释结果。近期研究表明，LLM可以通过自我修正来优化自己的回答，而不需要设置另一个LLM进行评估（Liu等人[2024](https://arxiv.org/html/2409.00054v1#bib.bib19)；Li等人[2024](https://arxiv.org/html/2409.00054v1#bib.bib17)），成功的例子包括思维链（CoT）（Wei等人[2022](https://arxiv.org/html/2409.00054v1#bib.bib32)）和SelfRefine（Madaan等人[2024](https://arxiv.org/html/2409.00054v1#bib.bib20)）。我们分别为基于RAG的聊天机器人配备了这两种提示方法进行注释，并将其称为CoT-RAG和SelfRefine-RAG。这两种方法的完整提示可在附录2中找到。此外，在LLM-Duo中，探索者的潜在替代方案是长文档上下文LLM，它能够处理整个文档的tokens，而不是通过RAG进行分块和检索。我们将分别使用基于RAG的探索者和长文档上下文LLM的LLM-Duo分别表示为LLM-Duo-RAG和LLM-Duo-LongContext。此外，我们还与直接将论文文本输入LLM进行零-shot QA注释的方法进行比较，这些方法没有评估反馈循环，包括ShortContext LLM、LongContext LLM、OpenAI Assistant和RAG。

![请参考说明](img/bf9b2e203a7e12afbaaadd1ad5b73a8b.png)

(a) 不同k值下参与者注释的示意图。

![请参考说明](img/ff2527a6f54c4a784a08c377dad42754.png)

(b) 使用不同LLM的LLM-Duo-RAG在k=1,2,3下的参与者注释准确度范围。

![请参考说明](img/74572e67ce81c598b905f61c36dd105b.png)

(c) 在参与者注释查询的k=1,2,3下，回溯检索到的块的Token计数分布，按相似度分数变化。

图5：不同k值下参与者注释及相关指标概览。

### 评估

在我们与基线方法进行比较的实验中，我们报告了六种类型的指标：1) 一致性轮次（CR）：方法在实现注释一致性之前进行的精炼循环次数；2) 冗长指数（VI）：每千tokens注释中的方面数，这是衡量注重内容完整性的注释的重要指标；3) 枚举数量（EQ）：注释中列出的项数（例如在我们的测试案例中为治疗活动、治疗目标等）；4) 忠实度（Faith）：注释对提供文本的忠实程度，通过Llamaindex的FaithfulnessEvaluator^†^†https://docs.llamaindex.ai/en/stable/examples/evaluation/faithfulness˙eval来衡量；5) 准确度（ACC）：所有LLM提供的注释中正确注释的百分比；6) 覆盖度（Cover）：正确LLM提供的注释占提供文本中提到的概念实体总数的百分比。

## 结果

本节中，我们首先提供对渐进本体提示算法和LLM-Duo注释框架的详细评估。然后，我们展示了使用我们的自动化知识发现框架发现的言语语言干预结果，进一步证明了我们的框架在解决实际科学知识发现任务中的有效性。

### 上下文大小分析

在我们的渐进本体提示算法中，在为概念节点构建注释提示时，特定概念节点的知识发现以及上下文中的本体结构（在$k$跳邻域内访问的节点）作为前缀提供注释问题的条件。$k$值决定了提供信息的多样性和量。为了评估POP算法的上下文大小如何影响注释质量，我们通过应用不同的$k$值生成LLM-Duo-RAG注释提示进行实验。如图[5(a)](https://arxiv.org/html/2409.00054v1#Sx5.F5.sf1 "在图5中 ‣ 基准 ‣ 实验 ‣ 通过LLMs自动化科学文献的知识发现：一种基于渐进本体提示的双代理方法")所示，我们选择了与“参与者”概念相关的本体子结构进行实验，该实验基于随机选择的8篇言语语言治疗论文。

| 方法 | LLM | IR | ICA | IKC |
| --- | --- | --- | --- | --- |
|  |  | CR | ACC | Cover | CR | VI | EQ | Faith | CR | ACC |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ShortContext | GPT3.5-turbo | - | 36.9% | 50% | - | 0.0249 | 5.46 | 0.9667 | - | 48.2% |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| OpenAI Assistant | GPT4-turbo | - | 76.1% | 69.0% | - | 0.0631 | 4.17 | 0.7857 | - | 53.3% |
| LongContext | GPT4-turbo | - | 76.3% | 57.1% | - | 0.0919 | 8.64 | 1.0 | - | 61.2% |
| LLM-Duo-LongContext | GPT4-turbo | 2.17 | 81.0% | 68.7% | 2.5 | 0.0926 | 8.68 | 0.8571 | 1.31 | 69.6% |
| RAG | GPT3.5-turbo | - | 47.6% | 50% | - | 0.0319 | 7.96 | 0.8550 | - | 48.7% |
| CoT-RAG | GPT3.5-turbo | 1.04 | 78.6% | 81% | 3.18 | 0.0771 | 10.37 | 0.7250 | 1.07 | 73.2% |
| SelfRefine-RAG | GPT3.5-turbo | 1.19 | 78.5% | 54.4% | 2.85 | 0.0694 | 7.17 | 0.8125 | 1.12 | 54.8% |
| LLM-Duo-RAG | GPT3.5-turbo | 1.84 | 100% | 86.4% | 2.58 | 0.1159 | 13.71 | 0.9285 | 1.46 | 85.6% |
| Llama3-instruct-70b | 2.71 | 78.6% | 55.6% | 2.59 | 0.0748 | 9.79 | 0.8648 | 1.52 | 61.0% |
| Mistral-instruct-8x22b | 2.30 | 81.9% | 67.5% | 2.16 | 0.0763 | 9.87 | 0.8875 | 1.46 | 67.2% |

表1：与基准方法的比较，使用了各种基础LLM。

![参见标题](img/6df503ab714ac9d6ba6946eccc9c6510.png)

(a) 移除重排序（wo_rerank）和共指（wo_coref）模块在LLM-Duo-RAG中的消融研究。

![参见标题](img/1664f7dca5afb5214e746ed2879fbe41.png)

(b) 已发现的前20种疾病干预数量。

![参考标题](img/4f7788811cc9e04dfb4286aab5513908.png)

(c) 不同年龄组的干预发现数量。

图6：消融研究和发现的语言-言语干预统计。

参与者概念的最终注释准确性如图[5(b)](https://arxiv.org/html/2409.00054v1#Sx5.F5.sf2 "在图5 ‣ 基线 ‣ 实验 ‣ 通过LLM自动化科学文献中的知识发现：一种渐进式本体提示的双代理方法")所示。结果表明，随着上下文跳数$k$的增加，注释准确性显著提高，表明更大的上下文提供了更多的信息提示，从而提升了注释质量。此外，GPT-4-turbo在所有$k$值下始终优于GPT-3.5-turbo，证明更先进的语言模型能够进一步提高注释准确性。此外，我们检查了基于不同$k$值的各种注释查询所检索回的文本块。我们在图[5(c)](https://arxiv.org/html/2409.00054v1#Sx5.F5.sf3 "在图5 ‣ 基线 ‣ 实验 ‣ 通过LLM自动化科学文献中的知识发现：一种渐进式本体提示的双代理方法")中报告了检索回来的文本块的相似度得分范围和标记数分布。相似度得分表示检索到的文本与注释查询之间的语义相关性。从结果中可以观察到，当$k=1$时，检索到的文本块通常与查询的相似度较低，且随着相似度得分的增加，标记数趋于减少，导致注释质量较差。相反，更大的$k$值，特别是$k=2$，会检索到更多相关的内容。对于$k=2$，标记数随着相似度得分的提高而呈现增加趋势，表明更多实质性且相关的内容被捕获，从而导致更好的注释质量。

### 基线注释方法下的LLM-Duo

为了评估我们LLM-Duo标注框架的性能，我们将其与基线方法在语音语言干预发现的背景下进行比较。我们从我们的语音语言文献库中随机挑选了8篇论文，使用各种基线方法进行标注。比较主要集中在三个关键维度：1) 干预识别（IR），识别论文中的干预实体；2) 干预方面总结（IAS），对干预方法的一个方面（如过程、治疗活动、治疗目标）进行标注，这需要捕捉并总结论文中与之相关的所有信息片段；以及3) 干预知识补充（IKC），将干预与主题类别（如语言意识、语音发音、理解能力、基础技能等）相关联，并设置概念节点（如家庭、医疗设施、幼儿园、学校、远程治疗等）。对于IR和IKC任务，我们要求人工标注员完成标注作为黄金标准结果进行比较。对于IAS任务，由于人类解释的个体偏差，我们仅要求人工标注员标记论文中与特定干预方面相关的文本片段。

实验结果见表[1](https://arxiv.org/html/2409.00054v1#Sx6.T1 "Table 1 ‣ Context Size Analysis ‣ Results ‣ Automating Knowledge Discovery from Scientific Literature via LLMs: A Dual-Agent Approach with Progressive Ontology Prompting")。需要注意的是，我们使用Llama3-instruct-70b (FP16) 和 Mistral-instruct-8x22b (INT8) 模型实现了“ShortContext”。然而，直接用完整论文文本对这些模型进行提示，在零样本问答设置下无法生成标注。它们的生成结果与标注问题不匹配。从表[1](https://arxiv.org/html/2409.00054v1#Sx6.T1 "Table 1 ‣ Context Size Analysis ‣ Results ‣ Automating Knowledge Discovery from Scientific Literature via LLMs: A Dual-Agent Approach with Progressive Ontology Prompting")中的结果来看，我们观察到LLM-Duo-RAG优于所有基线方法。尽管GPT4-turbo具有128k的上下文窗口长度，并且能够生成标注，但其标注覆盖仍然不足。将其与LLM-Duo框架结合可以显著提高标注的准确性和全面性。此外，与简单的RAG方法相比，像CoT和SelfRefine这样的自我纠正提示方法可以显著增强标注，但它们的表现仍然不如LLM-Duo-RAG。与昂贵的GPT模型不同，LLM-Duo-RAG使用包括Llama3-instruct-70b和Mistral-instruct-8x22b在内的开源模型，能够达到类似的标注质量。

### 消融研究

在我们的框架中，聊天机器人探索器使用RAG技术。我们通过“FastCoref”增强了共指消解的生成质量，并通过使用“cross-encoder/ms-marco-MiniLM-L-2-v2模型”按相似度得分重新排序检索到的块。本节介绍了这两个组件的消融研究。由于识别干预实体是执行POP算法后的关键第一步，我们在本研究中报告了干预识别的准确性。如图[6(a)](https://arxiv.org/html/2409.00054v1#Sx6.F6.sf1 "图6 ‣ 上下文大小分析 ‣ 结果 ‣ 通过LLM自动化科学文献中的知识发现：一种双代理方法与渐进本体提示")所示，结果表明，去除这些组件会显著降低注释准确性，证明了每个模块的必要性。

### 言语语言干预发现

我们的知识发现框架已识别出由64,177篇研究论文中的案例研究支持的2,421个干预措施。这些已发现干预措施的统计数据展示在图[6(b)](https://arxiv.org/html/2409.00054v1#Sx6.F6.sf2 "图6 ‣ 上下文大小分析 ‣ 结果 ‣ 通过LLM自动化科学文献中的知识发现：一种双代理方法与渐进本体提示")和图[6(c)](https://arxiv.org/html/2409.00054v1#Sx6.F6.sf3 "图6 ‣ 上下文大小分析 ‣ 结果 ‣ 通过LLM自动化科学文献中的知识发现：一种双代理方法与渐进本体提示")中。完整的干预注释示例可见于附录3。19位临床医生和学生通过在线Google表单审查了我们的注释。在检查干预注释并阅读相应文献后，他们报告称，89%的注释有效且完整。基于图[4](https://arxiv.org/html/2409.00054v1#Sx5.F4 "图4 ‣ 案例描述：言语语言干预发现 ‣ 实验 ‣ 通过LLM自动化科学文献中的知识发现：一种双代理方法与渐进本体提示")中的干预本体和注释，我们构建了一个干预知识图谱，未来将公开访问。预计这个知识图谱将成为言语语言领域专家的宝贵资源，支持基于证据的临床决策、问答和推荐，最终提高医疗保健结果。

## 结论

在本文中，我们开发了一个基于大型语言模型（LLMs）的全新自动知识发现框架，该框架的特点是逐步本体提示算法和双代理注释系统。与先进的基准方法相比，所提出的方法实现了更卓越的性能，能够更准确地进行知识发现。应用于语言语音干预发现时，该框架还为语言治疗社区提供了一个宝贵且易于访问的干预知识库。

## 参考文献

+   Abid, Farooqi和Zou（2021）Abid, A.; Farooqi, M.; 和Zou, J. 2021. 大型语言模型中的持久反穆斯林偏见。载于*2021年AAAI/ACM人工智能、伦理与社会会议论文集*，298–306页。

+   Achiam等（2023）Achiam, J.; Adler, S.; Agarwal, S.; Ahmad, L.; Akkaya, I.; Aleman, F. L.; Almeida, D.; Altenschmidt, J.; Altman, S.; Anadkat, S.; 等. 2023. GPT-4技术报告。*arXiv预印本 arXiv:2303.08774*。

+   Agrawal等（2022）Agrawal, M.; Hegselmann, S.; Lang, H.; Kim, Y.; 和Sontag, D. 2022. 大型语言模型是少样本临床信息提取器。*arXiv预印本 arXiv:2205.12689*。

+   Bang等（2023）Bang, Y.; Cahyawijaya, S.; Lee, N.; Dai, W.; Su, D.; Wilie, B.; Lovenia, H.; Ji, Z.; Yu, T.; Chung, W.; 等. 2023. 针对推理、幻觉和互动性的多任务、多语言、多模态评估——ChatGPT的应用研究。*arXiv预印本 arXiv:2302.04023*。

+   Carta等（2023）Carta, S.; Giuliani, A.; Piano, L.; Podda, A. S.; Pompianu, L.; 和Tiddia, S. G. 2023. 基于迭代零样本大型语言模型提示的知识图谱构建方法。*arXiv预印本 arXiv:2307.01128*。

+   Cauter和Yakovets（2024）Cauter, Z.; 和Yakovets, N. 2024. 基于本体引导的知识图谱构建方法——来自维护短文本的研究。载于*第一届知识图谱与大型语言模型研讨会（KaLLM 2024）论文集*，75–84页。

+   Ehrlinger和Wöß（2016）Ehrlinger, L.; 和Wöß, W. 2016. 朝着知识图谱定义的迈进。*SEMANTiCS（海报、演示、SuCCESS）*，48(1-4)：2。

+   Gao等（2023）Gao, Y.; Xiong, Y.; Gao, X.; Jia, K.; Pan, J.; Bi, Y.; Dai, Y.; Sun, J.; 和Wang, H. 2023. 基于检索增强生成的大型语言模型：综述。*arXiv预印本 arXiv:2312.10997*。

+   Gruninger（1995）Gruninger, M. 1995. 本体设计与评估方法学。载于*第十四届国际人工智能会议（IJCAI’95），知识共享中的基本本体问题研讨会*。

+   Guo等（2023）Guo, B.; Zhang, X.; Wang, Z.; Jiang, M.; Nie, J.; Ding, Y.; Yue, J.; 和Wu, Y. 2023. ChatGPT与人类专家有多接近？对比语料库、评估和检测。*arXiv预印本 arXiv:2301.07597*。

+   Kim等（2024）Kim, H.; Mitra, K.; Chen, R. L.; Rahman, S.; 和Zhang, D. 2024. MEGAnno+: 一种人类与大型语言模型协作的注释系统。*arXiv预印本 arXiv:2402.18050*。

+   Kojima 等人（2022）Kojima, T.; Gu, S. S.; Reid, M.; Matsuo, Y.; 和 Iwasawa, Y. 2022. 大型语言模型是零-shot推理者。*神经信息处理系统进展*，35: 22199–22213。

+   Kommineni、König-Ries 和 Samuel（2024）Kommineni, V. K.; König-Ries, B.; 和 Samuel, S. 2024. 从人类专家到机器：一种支持LLM的本体和知识图谱构建方法。*arXiv预印本 arXiv:2403.08345*。

+   Laurenzi、Mathys 和 Martin（2024）Laurenzi, E.; Mathys, A.; 和 Martin, A. 2024. 一个基于LLM的企业知识图谱（EKG）工程过程。见 *AAAI研讨会系列论文集*，第3卷，148–156。

+   Law 等人（1996）Law, J.; Garrett, Z.; Nye, C.; Cochrane Developmental, P.; 和 Group, L. P. 1996. 针对主要语言延迟或障碍儿童的语言治疗干预。*Cochrane 系统评价数据库*，2015(5)。

+   Lewis 等人（2020）Lewis, P.; Perez, E.; Piktus, A.; Petroni, F.; Karpukhin, V.; Goyal, N.; Küttler, H.; Lewis, M.; Yih, W.-t.; Rocktäschel, T.; 等人 2020. 增强检索生成用于知识密集型自然语言处理任务。*神经信息处理系统进展*，33: 9459–9474。

+   Li 等人（2024）Li, L.; Chen, Z.; Chen, G.; Zhang, Y.; Su, Y.; Xing, E.; 和 Zhang, K. 2024. 信心至关重要：重新审视大型语言模型的内在自我纠正能力。arXiv:2402.12563。

+   Li 等人（2023）Li, M.; Shi, T.; Ziems, C.; Kan, M.-Y.; Chen, N.; Liu, Z.; 和 Yang, D. 2023. CoAnnotating: 在数据标注中通过不确定性引导的工作分配，涉及人类和大型语言模型。见 Bouamor, H.; Pino, J.; 和 Bali, K., 编辑，*2023年自然语言处理经验方法会议论文集*，1487–1505。新加坡：计算语言学会。

+   Liu 等人（2024）Liu, D.; Nassereldine, A.; Yang, Z.; Xu, C.; Hu, Y.; Li, J.; Kumar, U.; Lee, C.; 和 Xiong, J. 2024. 大型语言模型具有内在的自我纠正能力。arXiv:2406.15673。

+   Madaan 等人（2024）Madaan, A.; Tandon, N.; Gupta, P.; Hallinan, S.; Gao, L.; Wiegreffe, S.; Alon, U.; Dziri, N.; Prabhumoye, S.; Yang, Y.; 等人 2024. Self-refine: 具有自反馈的迭代精炼。*神经信息处理系统进展*，36。

+   Martins、Marinho 和 Martins（2019）Martins, P. H.; Marinho, Z.; 和 Martins, A. F. 2019. 联合学习命名实体识别与实体链接。*arXiv预印本 arXiv:1907.08243*。

+   Melnyk 和 Fineout-Overholt（2022）Melnyk, B. M.; 和 Fineout-Overholt, E. 2022. *护理与医疗中的循证实践：最佳实践指南*。Lippincott Williams & Wilkins出版社。

+   Mihindukulasooriya 等人（2023）Mihindukulasooriya, N.; Tiwari, S.; Enguix, C. F.; 和 Lata, K. 2023. Text2kgbench: 一个用于从文本生成本体驱动知识图谱的基准。见 *国际语义网会议*，247–265。Springer出版社。

+   Otmazgin, Cattan, 和 Goldberg（2022）Otmazgin, S.; Cattan, A.; 和 Goldberg, Y. 2022. F-coref：快速、准确且易于使用的共指解析. *arXiv 预印本 arXiv:2209.04280*。

+   Peng 等人（2023）Peng, C.; Xia, F.; Naseriparsa, M.; 和 Osborne, F. 2023. 知识图谱：机遇与挑战. *人工智能评论*, 56(11): 13071–13102。

+   Qin 等人（2023）Qin, C.; Zhang, A.; Zhang, Z.; Chen, J.; Yasunaga, M.; 和 Yang, D. 2023. ChatGPT 是通用自然语言处理任务的求解器吗？ *arXiv 预印本 arXiv:2302.06476*。

+   Rutten 等人（2021）Rutten, L. J. F.; Zhu, X.; Leppin, A. L.; Ridgeway, J. L.; Swift, M. D.; Griffin, J. M.; St Sauver, J. L.; Virk, A.; 和 Jacobson, R. M. 2021. 临床组织应对 COVID-19 疫苗犹豫的基于证据的策略. 收录于 *梅奥诊所学报*, 第96卷, 699–707. Elsevier。

+   Sackett（1997）Sackett, D. L. 1997. 基于证据的医学. 收录于 *围产学研讨会*, 第21卷, 3–5. Elsevier。

+   Usai 等人（2018）Usai, A.; Pironti, M.; Mital, M.; 和 Aouina Mejri, C. 2018. 从文本数据中发现知识：通过文本挖掘的系统评审. *知识管理杂志*, 22(7): 1471–1488。

+   Wadhwa, Amir, 和 Wallace（2023）Wadhwa, S.; Amir, S.; 和 Wallace, B. C. 2023. 在大型语言模型时代重新审视关系提取. 收录于 *会议论文集. 计算语言学协会年会*, 第2023卷, 15566. NIH 公共访问。

+   Wang 等人（2023）Wang, H.; Fu, T.; Du, Y.; Gao, W.; Huang, K.; Liu, Z.; Chandak, P.; Liu, S.; Van Katwyk, P.; Deac, A.; 等人. 2023. 人工智能时代的科学发现. *自然*, 620(7972): 47–60。

+   Wei 等人（2022）Wei, J.; Wang, X.; Schuurmans, D.; Bosma, M.; Xia, F.; Chi, E.; Le, Q. V.; Zhou, D.; 等人. 2022. 思维链提示激发大型语言模型的推理能力. *神经信息处理系统进展*, 35: 24824–24837。

+   Wei 等人（2023）Wei, X.; Cui, X.; Cheng, N.; Wang, X.; Zhang, X.; Huang, S.; Xie, P.; Xu, J.; Chen, Y.; Zhang, M.; 等人. 2023. 通过与 ChatGPT 对话进行零-shot 信息提取. *arXiv 预印本 arXiv:2302.10205*。

+   Wei 等人（2019）Wei, Z.; Su, J.; Wang, Y.; Tian, Y.; 和 Chang, Y. 2019. 一种用于关系三元组提取的新型级联二元标注框架. *arXiv 预印本 arXiv:1909.03227*。

+   Ye 等人（2022）Ye, H.; Zhang, N.; Chen, H.; 和 Chen, H. 2022. 生成式知识图谱构建：综述. *arXiv 预印本 arXiv:2210.12714*。

+   Zhang 和 Soh（2024）Zhang, B.; 和 Soh, H. 2024. 提取、定义、标准化：一种基于 LLM 的知识图谱构建框架. *arXiv 预印本 arXiv:2404.03868*。

+   Zhong 等人（2023a）Zhong, L.; Wu, J.; Li, Q.; Peng, H.; 和 Wu, X. 2023a. 自动化知识图谱构建的综合调查. *ACM 计算机调查*, 56(4): 1–62。

+   Zhong等人（2023b）Zhong, Q.; Ding, L.; Liu, J.; Du, B.; 和Tao, D. 2023b. ChatGPT也能理解吗？ChatGPT与微调BERT的比较研究。*arXiv预印本 arXiv:2302.10198*。

## 附录A 附录

### 1 干预本体论与文献

1.1 干预本体论 干预本体论如主文中的图4所示。在本节中，我们将详细解释本体中引入的概念如下：

+   •

    干预代表旨在增强个体沟通技巧的针对性治疗方法。

+   •

    障碍代表导致个体在声音、语言或吞咽功能上出现困难的障碍类型。

+   •

    环境代表了实施干预的特定环境。我们确定了六个关键环境：家庭、医疗设施（如医院或康复中心）、早期儿童教育中心（如托儿所或日托中心）、学校、诊所和私人诊所，以及远程治疗。

+   •

    主题代表干预的主题。如表[2](https://arxiv.org/html/2409.00054v1#A1.T2 "表 2 ‣ 1 干预本体论与文献 ‣ 附录 A 附录 ‣ 通过大语言模型的渐进式本体提示自动化科学文献知识发现")所示，我们根据干预的特征和治疗目标将干预分为10个主题。

+   •

    治疗活动代表旨在解决个体特定言语或语言挑战的任务，例如使用最小对活动来增强语音意识。

+   •

    治疗目标代表了干预旨在增强的特定领域。

+   •

    程序代表干预执行过程的综合描述。

+   •

    效果代表了关于干预有效性的结论。

+   •

    频率/剂量/持续时间代表在案例研究中展示其效果的干预频率/剂量/持续时间。

+   •

    案例研究代表对特定个体或群体的干预进行的详细检查，目标是提供对患者独特需求的深刻理解，并评估干预的有效性。

+   •

    参与者代表参与干预案例研究的个体或群体。

+   •

    年龄代表实验参与者的年龄或干预目标人群的年龄。年龄按半岁为粒度进行量化。我们还将年龄转换为年龄组，包括“新生儿、婴儿、幼儿、儿童、学龄前儿童、学龄儿童、较大儿童、青少年、青少年、青少年、成人、年轻成人、中年、老年、老年人”。每个特定年龄可能与多个年龄组相关联。例如，13岁的个体可以归类为‘青少年’，‘青少年’和‘儿童’年龄组。

+   •

    语言代表了实验参与者在干预案例研究中的口语语言。

| 主题 | 定义 |
| --- | --- |
| 言语意识 | 旨在识别和理解语音的工作。包括语音意识（识别和操控语音）、听觉辨别（区分语音）和声音识别（理解声音的意义）。 |
| 言语发音 | 指利用口腔、嘴唇、舌头和呼吸系统进行语音的物理发音的工作。关注语音的清晰度和准确性，并将其拼成单词。 |
| 理解力 | 旨在提高理解（接收性）语言的工作。 |
| 表达性语言 | 旨在提高孩子的表达性语言，无论是在数量、词汇还是结构方面的工作。 |
| 自我监控 | 旨在帮助患者意识到他们的言语和语言困难，并帮助他们克服这些困难的工作。 |
| 泛化 | 旨在帮助将言语和语言或治疗成果转移到其他情境和环境中的工作。 |
| 基础技能 | 针对一系列早期技能进行实践和提升，这些技能中的许多可以视为言语和语言发展的基础。 |
| 功能性沟通 | 专注于帮助孩子在生活情境中参与的沟通方面的工作；这可能包括功能性语言、手语或符号的使用。 |
| 成人理解与赋能 | 帮助父母理解孩子言语和语言困难的性质，帮助改善的因素及其原因的工作。 |
| 成人与儿童互动 | 着重于父母/成人与儿童之间的互动。所有对成人/父母-儿童互动的改变都强调了那些促进言语和语言发展的互动。 |

表 2: 干预主题的定义。

| 来源 | 文献数量 |
| --- | --- |
| Wiley Online Library | 9,511 |
| NIH PubMed | 30,928 |
| SpringerLink | 6,549 |
| GoogleScholar | 8,662 |
| ASHA | 5,641 |
| EBSCOHost | 2,886 |

表 3: 言语-语言干预发现的文献基础。

1.2 语音语言治疗文献收集 我们在语音语言治疗领域内培养了一个文献语料库，如表[3](https://arxiv.org/html/2409.00054v1#A1.T3 "表 3 ‣ 1 干预本体和文献 ‣ 附录 A 附录 ‣ 通过 LLM 自动化科学文献中的知识发现：一种双代理方法与渐进本体提示")所示，以促进干预方法的发现。为了进行文献搜索，我们使用了一组精心挑选的关键词，这些关键词来自语音语言治疗中常用术语的词汇表。这些关键词包括：“语音语言治疗、语音语言障碍、语音音障碍、发音障碍、语音干预、语言干预、听觉辨别、听觉处理障碍、语音意识、语音过程、听觉感知、咿呀学语、运动性言语障碍、语素、语音学、韵律、口吃、语言损伤、语音语言病理学家、语音语言治疗师、咿呀学语、表达性语言延迟、裂腭语言障碍、自闭症谱系障碍、发育性语音障碍、发育性口吃、语音损伤、发育性运动性言语障碍、唐氏综合症、吞咽障碍、沟通障碍、发音障碍、阅读障碍、言语性失语症、构音障碍、构音障碍、吞咽障碍、交流障碍、表达性语言障碍、运动障碍、失语症、辅助和替代沟通、中央听觉处理障碍、唇腭裂、唐氏综合症、流畅性障碍、听力丧失、口面功能障碍、口语语言障碍、书面语言障碍、获得性脑损伤、言语性失语症、听觉理解、识字障碍、声音困难、基于语言的学习障碍。”

### 2 提示方法

2.1 POP 提示

在POP算法中，特定概念节点的注释提示是从模板$T$生成的，该模板是使用节点的上下文（k跳访问的邻域）和该上下文中的知识发现构建的。我们精心设计提示，任务LLM生成$T$如下：

以下三元组概述了一个注释本体：{（干预、研究于、案例研究）、（干预、包括、参与者）、（案例研究、使用于、频率）}。除了{#频率}外，所有概念节点都已被注释。您的任务是利用本体结构为{#频率}创建所有可能的注释提示模板。

示例：

本体：（（干预、研究于、案例研究）、（干预、针对、障碍）、（案例研究、包括、参与者）、（参与者、具有、障碍））。

注释：参与者

提示：

T1: {#案例研究}中的参与者是谁？

T2: {#干预}在{#案例研究}中进行研究。{#案例研究}中的参与者是谁？

T3: {#干预}，针对{#疾病}，在{#案例研究}中进行研究。{#案例研究}中的参与者是谁？以下是GPT3.5针对上述提示的一些示例输出：

T1: 在{#案例研究}中使用的频率是什么？

T2: 在{#案例研究}中研究的{#干预}背景下，干预的频率是多少？

T3: {#干预}，其中包括{#参与者}，在{#案例研究}中进行研究。此研究使用的频率是什么？由于模板$T$由在当前节点之前访问的概念节点组成，当前节点的最终注释提示是通过将与这些概念节点相关的知识发现填充到$T$中得出的。

2.2 CoT和SelfRefine提示法：我们不使用外部大型语言模型（LLM）提供评估反馈给探索者，而是采用CoT和SelfRefine提示技巧作为基准，帮助探索者独立地完善注释。CoT和SelfRefine的提示如下：

[⬇](data:text/plain;base64,QmFja2dyb3VuZDogWW91ciBsYXN0IGFuc3dlciB0byBteSBxdWVzdGlvbiB7I2luaXQgYW5ub3RhdGlvbiBxdWVzdGlvbn0gaXM6IHsjbGFzdCBhbm5vdGF0aW9ufS4KLS0tCipDb1Q6CiAgICBQcm9tcHRfQ29UMTogInsjQmFja2dyb3VuZH0gTWFrZSBhIHBsYW4gdG8gY29ycmVjdGx5IGFuc3dlciBteSBxdWVzdGlvbiBhZ2Fpbi4iCiAgICBQcm9tcHRfQ29UMjogInsjQmFja2dyb3VuZH0gTWFrZSBhIHBsYW4gdG8gYW5zd2VyIG15IHF1ZXN0aW9uIGFnYWluIHdpdGggbW9yZSBjb21wcmVoZW5zaXZlIHJlc3VsdHMuIgotLS0KKlNlbGZSZWZpbmU6CiAgICBQcm9tcHRfU2VsZlJlZmluZV9GZWVkYmFjazE6ICJ7I0JhY2tncm91bmR9IFJlZmxlY3QgeW91ciBhbnN3ZXIuIEFuYWx5emUgdGhlIGNvcnJlY3RuZXNzIG9mIHRoZSBpbmZvcm1hdGlvbiBwcm92aWRlZC4gUHJvdmlkZSBjcml0cXVlIHRvIGhlbHAgaW1wcm92ZSB0aGUgYW5zd2VyLiBZb3VyIGZlZWRiYWNrOiIKICAgIFByb21wdF9TZWxmUmVmaW5lX1JlZmluZTE6ICJ7I0JhY2tncm91bmR9IENyaXRpY3M6IHsjZmVlZGJhY2t9LiBCYXNlZCBvbiB5b3VyIGxhc3QgYW5zd2VyIGFuZCBpdHMgY3JpdGljcywgcmV2aXNlIHlvdXIgYW5zd2VyIHRvIG15IHF1ZXN0aW9uLiBZb3VyIGFuc3dlcjoiCiAgICBQcm9tcHRfU2VsZlJlZmluZV9GZWVkYmFjazI6ICJ7I0JhY2tncm91bmR9IFJlZmxlY3QgeW91ciBhbnN3ZXIuIEFuYWx5emUgdGhlIGluY2x1ZGVkIGFzcGVjdHMgaW4geW91ciBhbnN3ZXIuIFByb3ZpZGUgY3JpdHF1ZSB0byBoZWxwIG1ha2UgdGhlIHJlc3BvbnNlIG1vcmUgY29tcHJlaGVuc2l2ZS4gWW91ciBmZWVkYmFjazoiCiAgICBQcm9tcHRfU2VsZlJlZmluZV9SZWZpbmUyOiAieyNCYWNrZ3JvdW5kfSBDcml0aWNzOiB7I2ZlZWRiYWNrfS4gQmFzZWQgb24geW91ciBsYXN0IGFuc3dlciBhbmQgaXRzIGNyaXRpY3MsIHJldmlzZSB5b3VyIGFuc3dlciB0byBteSBxdWVzdGlvbi4gWW91ciBhbnN3ZXI6Ig==)背景： 你对我问题的上一个回答是：{#init 注释 问题}：{#last 注释}。---*CoT:Prompt_CoT1:  "{#背景} 制定一个计划，重新正确回答我的问题。"Prompt_CoT2:  "{#背景} 制定一个计划，重新回答我的问题，并给出更全面的结果。"---*SelfRefine:Prompt_SelfRefine_Feedback1:  "{#背景} 反思你的回答，分析所提供信息的正确性，提供批评帮助改进答案。你的反馈："Prompt_SelfRefine_Refine1:  "{#背景} 批评：{#反馈}。根据你上一个回答及其批评，修改你的答案。你的回答："Prompt_SelfRefine_Feedback2:  "{#背景} 反思你的回答，分析回答中包括的方面，提供批评以帮助使回答更全面。你的反馈："Prompt_SelfRefine_Refine2:  "{#背景} 批评：{#反馈}。根据你上一个回答及其批评，修改你的答案。你的回答："

### 3 注释示例

以下是基于名为《中风后言语障碍的强化治疗》研究的干预注释示例。

[⬇](data:text/plain;base64,IkludGVydmVudGlvbiI6ICJMZWUgU2lsdmVybWFuIFZvaWNlIFRyZWF0bWVudCAoTFNWVCBMT1VEKSIsCiJEaXNvcmRlciI6IFsKICAgICAgICAic3Ryb2tlIiwKICAgICAgICAiYXRheGlhIiwKICAgICAgICAibXVsdGlwbGUgc2NsZXJvc2lzIiwKICAgICAgICAidHJhdW1hdGljIGJyYWluIGluanVyeSAoVEJJKSIKXSwKIlByb2NlZHVyZSI6ICJUaGUgdGhlcmFweSBwcm9jZXNzIG9mIGNvbmR1Y3RpbmcgdGhlIExlZSBTaWx2ZXJtYW4gVm9pY2UgVHJlYXRtZW50IChMU1ZUIExPVUQpIGludGVydmVudGlvbiBpbnZvbHZlcyBpbnRlbnNpdmUgaGlnaC1lZmZvcnQgZXhlcmNpc2VzIGFpbWVkIGF0IGluY3JlYXNpbmcgdm9jYWwgbG91ZG5lc3MgdG8gYSBsZXZlbCB3aXRoaW4gbm9ybWFsIGxpbWl0cyB1c2luZyBoZWFsdGh5IGFuZCBlZmZpY2llbnQgdm9pY2UgdGVjaG5pcXVlcy4gVGhlIHRyZWF0bWVudCBwcm90b2NvbCBpbmNsdWRlcyBzZXNzaW9ucyBmb3VyIHRpbWVzIGEgd2VlayBmb3IgNCB3ZWVrcywgdG90YWxpbmcgMTYgaW5kaXZpZHVhbCBvbmUtaG91ciBzZXNzaW9ucy4gRWFjaCBzZXNzaW9uIGNvbnNpc3RzIG9mIHRhc2tzIHN1Y2ggYXMgbWF4aW1hbCBzdXN0YWluZWQgdm93ZWwgcGhvbmF0aW9uLCBwaXRjaCByYW5nZSBleGVyY2lzZXMsIGFuZCByZWFkaW5nIGZ1bmN0aW9uYWwgcGhyYXNlcyBhdCBpbmRpdmlkdWFsIHRhcmdldCBsb3VkbmVzcyBsZXZlbHMuIFRoZSBzZWNvbmQgaGFsZiBvZiBlYWNoIHNlc3Npb24gcHJvZ3Jlc3NlcyB0byBmdW5jdGlvbmFsIHNwZWVjaCB0YXNrcywgbW92aW5nIGZyb20gd29yZHMgYW5kIHBocmFzZXMgdG8gY29udmVyc2F0aW9uIG92ZXIgdGhlIGNvdXJzZSBvZiB0aGUgMTYgc2Vzc2lvbnMuIEFkZGl0aW9uYWxseSwgcGFydGljaXBhbnRzIGFyZSBhc3NpZ25lZCBkYWlseSBob21ld29yayB0byBwcmFjdGljZSB1c2luZyBub3JtYWwgbG91ZG5lc3MgYW5kIGZhY2lsaXRhdGUgZ2VuZXJhbGl6YXRpb24gb2Ygbm9ybWFsIGxvdWRuZXNzIG91dHNpZGUgdGhlIHRyZWF0bWVudCByb29tLiIsCiJGcmVxdWVuY3kiOiAiZm91ciB0aW1lcyBhIHdlZWsiLAoiRG9zYWdlIjogIm9uZS1ob3VyIHNlc3Npb24uIiwKIkR1cmF0aW9uIjogIjQgd2Vla3MsIHRvdGFsaW5nIDE2IGluZGl2aWR1YWwgb25lLWhvdXIgc2Vzc2lvbnMuIiwKIkVmZmljYWN5IjogIlRoZSBvdXRjb21lIG9mIHRoZSB0aGUgTGVlIFNpbHZlcm1hbiBWb2ljZSBUcmVhdG1lbnQgKExTVlQgTE9VRCkgaW50ZXJ2ZW50aW9uIGluIHRoaXMgc3R1ZHkgc2hvd2VkIHBvc2l0aXZlIGNoYW5nZXMgaW4gYWNvdXN0aWMgdGFza3MsIG1vdmluZyBmcm9tIHdvcmRzIGFuZCBwaHJhc2VzIHRvIGNvbnZlcnNhdGlvbiBvdmVyIHRoZSBjb3Vyc2Ugb2YgdGhlIDE2IHNlc3Npb25zLiBBZGRpdGlvbmFsbHksIHBhcnRpY2lwYW50cyBhcmUgYXNzaWduZWQgZGFpbHkgaG9tZXdvcmsgdG8gcHJhY3RpY2UgdXNpbmcgbm9ybWFsIGxvdWRuZXNzIGFuZCBmYWNpbGl0YXRlIGdlbmVyYWxpemF0aW9uIG9mIG5vcm1hbCBsb3VkbmVzcyBvdXRzaWRlIHRoZSB0cmVhdG1lbnQgcm9vbS4iLAoiVGhlcmFweSBHb2FsIjogWwogICAgICAgICJJbmNyZWFzZSB2b2NhbCBsb3VkbmVzcyB0byBhIGxldmVsIHdpdGhpbiBub3JtYWwgbGltaXRzIiwKICAgICAgICAiVXNlIGhlYWx0aHkgYW5kIGVmZmljaWVudCB2b2ljZSB0ZWNobmlxdWVzIiwKICAgICAgICAiUHJvZ3Jlc3MgZnJvbSB3b3JkcyBhbmQgcGhyYXNlcyB0byBjb252ZXJzYXRpb24gb3ZlciAxNiBzZXNzaW9ucyIsCiAgICAgICAgIkZhY2lsaXRhdGUgZ2VuZXJhbGl6YXRpb24gb2Ygbm9ybWFsIGxvdWRuZXNzIG91dHNpZGUgdGhlIHRyZWF0bWVudCByb29tIgpdLAoiUGFydGljaXBhbnQiOiAiRm91ciBwYXJ0aWNpcGFudHMgKFAwMSB0byBQMDQpLiIsCiJBZ2UiOiAiUGFydGljaXBhbnRzIGluIHRoZSBzdHVkeSByYW5nZWQgaW4gYWdlIGZyb20gNTAgdG8gNzQgeWVhcnMuIiwKIkxhbmd1YWdlIjogImFzc3VtZSB0byBiZSBFbmdsaXNoLiIsCiJDYXNlIFN0dWR5IjogIkNhc2Ugc3R1ZGllcyBhbmQgZXhwZXJpbWVudHMgcmVnYXJkaW5nIHRoZSBMZWUgU2lsdmVybWFuIFZvaWNlIFRyZWF0bWVudCAoTFNWVCBMT1VEKSBpbnRlcnZlbnRpb24gaW4gdGhpcyBzdHVkeSBzaG93ZWQgcG9zaXRpdmUgY2hhbmdlcyBpbiBhY291c3RpYyB2YXJpYWJsZXMgb2Ygc3BlZWNoIGZvciBhbGwgcGFydGljaXBhbnRzIHdpdGggZHlzYXJ0aHJpYSBzZWNvbmRhcnkgdG8gc3Ryb2tlLiBUaGVyZSB3ZXJlIHN0YXRpc3RpY2FsbHkgc2lnbmlmaWNhbnQgaW5jcmVhc2VzIGluIHZvY2FsIGRCIFNQTCBmb3Igc3VzdGFpbmVkIHZvd2VsIHBob25hdGlvbiBhbmQgcGhyYXNlcyB0byBjb252ZXJzYXRpb24iCl0sCiJTZXR0aW5nIjogImhvbWUiLAoiVG

### 4 语言治疗干预知识图

以下是使用我们的知识发现框架构建的语言治疗干预知识图。我们计划在论文发布后公开访问该知识库。

![参见标题](img/8838b2074762a7a9eb61dd1514650daf.png)

图 7：图数据库概述。

![参见标题](img/b6aca5cff6f0b7986737886b2d0d7ecd.png)![参见标题](img/2cdf4d90c5fd4429c6b9cd4a5825342e.png)

图 8：干预知识图中干预-障碍的部分视图。

![参见标题](img/0ad8b500fa539a11b7c101767f5968de.png)![参见标题](img/4b753f13701e99ddd820961ceeaa92a5.png)

图 9：干预知识图中的干预-治疗活动部分视图。
