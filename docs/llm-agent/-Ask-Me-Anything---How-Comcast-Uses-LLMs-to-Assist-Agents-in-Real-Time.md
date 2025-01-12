<!--yml

分类：未分类

日期：2025-01-11 12:40:04

-->

# “问我任何事”：Comcast 如何使用大语言模型（LLMs）实时协助代理

> 来源：[https://arxiv.org/html/2405.00801/](https://arxiv.org/html/2405.00801/)

Scott Rome [scott˙rome@comcast.com](mailto:scott%CB%99rome@comcast.com) [0000-0003-0270-7192](https://orcid.org/0000-0003-0270-7192 "ORCID identifier")，Tianwen Chen [tianwen˙chen@comcast.com](mailto:tianwen%CB%99chen@comcast.com) [0009-0008-0477-7366](https://orcid.org/0009-0008-0477-7366 "ORCID identifier") Comcast AI Technologies Philadelphia Pennsylvania USA，Raphael Tang [raphael˙tang@comcast.com](mailto:raphael%CB%99tang@comcast.com) [0009-0007-2873-892X](https://orcid.org/0009-0007-2873-892X "ORCID identifier") Comcast AI Technologies Philadelphia Pennsylvania USA，Luwei Zhou [luwei˙zhou@comcast.com](mailto:luwei%CB%99zhou@comcast.com) [0009-0009-1862-3058](https://orcid.org/0009-0009-1862-3058 "ORCID identifier") 和 Ferhan Ture [ferhan˙ture@comcast.com](mailto:ferhan%CB%99ture@comcast.com) [0000-0002-5585-157X](https://orcid.org/0000-0002-5585-157X "ORCID identifier") Comcast AI Technologies Philadelphia Pennsylvania USA (2024)

# “问我任何事”：Comcast 如何使用大语言模型（LLMs）实时协助

实时协助代理

Scott Rome [scott˙rome@comcast.com](mailto:scott%CB%99rome@comcast.com) [0000-0003-0270-7192](https://orcid.org/0000-0003-0270-7192 "ORCID identifier")，Tianwen Chen [tianwen˙chen@comcast.com](mailto:tianwen%CB%99chen@comcast.com) [0009-0008-0477-7366](https://orcid.org/0009-0008-0477-7366 "ORCID identifier") Comcast AI Technologies Philadelphia Pennsylvania USA，Raphael Tang [raphael˙tang@comcast.com](mailto:raphael%CB%99tang@comcast.com) [0009-0007-2873-892X](https://orcid.org/0009-0007-2873-892X "ORCID identifier") Comcast AI Technologies Philadelphia Pennsylvania USA，Luwei Zhou [luwei˙zhou@comcast.com](mailto:luwei%CB%99zhou@comcast.com) [0009-0009-1862-3058](https://orcid.org/0009-0009-1862-3058 "ORCID identifier") 和 Ferhan Ture [ferhan˙ture@comcast.com](mailto:ferhan%CB%99ture@comcast.com) [0000-0002-5585-157X](https://orcid.org/0000-0002-5585-157X "ORCID identifier") Comcast AI Technologies Philadelphia Pennsylvania USA (2024)

###### 摘要。

客户服务是公司与其客户之间的接口，它对整体客户满意度有着重要影响。然而，高质量的服务可能会变得昂贵，从而促使公司尽量提高成本效率，并推动大多数公司利用人工智能驱动的助手或“聊天机器人”。另一方面，客户仍然希望与人类进行互动，特别是在复杂的情境下，例如争议和涉及账单支付等敏感话题时。¹¹1[https://bit.ly/3yaNO9t](https://bit.ly/3yaNO9t)

这提高了客户服务代理的要求。他们需要准确理解客户的问题或关切，找出一个既可接受又可行（并且符合公司政策）的解决方案，同时还要处理多个对话。

在这项工作中，我们将“任何问题都可以问我”（AMA）作为附加功能引入到面向客服代理的客户服务界面中。AMA允许客服代理在处理客户对话时，随时向大型语言模型（LLM）提问——LLM实时提供准确的答案，从而减少代理需要进行的上下文切换。在我们的内部实验中，我们发现，使用AMA的代理与使用传统搜索体验的代理相比，每个包含搜索的对话平均节省了约10%的时间，这相当于每年节省数百万美元。使用AMA功能的代理中，近80%提供了积极反馈，表明它作为一种AI辅助的客户关怀功能非常有用。

rag, llm, customer care, assistive AI, vector db, reranking^†^†journalyear: 2024^†^†copyright: rightsretained^†^†conference: Proceedings of the 47th International ACM SIGIR Conference on Research and Development in Information Retrieval; July 14–18, 2024; Washington, DC, USA^†^†doi: 10.1145/3626772.3661345^†^†isbn: 979-8-4007-0431-4/24/07^†^†ccs: Information systems Retrieval models and ranking^†^†ccs: Information systems Language models

## 1. 引言

与许多其他公司一样，Comcast通过多种沟通渠道提供客户服务。许多自助服务解决方案可以在移动端的“Xfinity”应用中找到（例如，查看最新账单），该应用还提供与名为“Xfinity Assistant”的AI驱动机器人聊天的选项。虽然这些数字化自动化功能已替代了许多任务中人工客服（也称为“代理”）的角色，但仍然有许多情况需要人际互动。

客户如果仅仅是想查找关于自己个人资料、互联网服务或账单的信息，他们应该能够在没有代理帮助的情况下完成。对于像重新安排约会或更改服务等相对简单的任务，这一点同样适用。

以往的研究表明，在某些客户服务场景下，人际互动优于人机互动（Wowak，[[n. d.]](https://arxiv.org/html/2405.00801v2#bib.bib22)）。例如，代理在需要创造性解决问题的情况下，可能比机器人表现得更好。在其他情况下，客户可能更愿意与代理交流，以受益于代理的同理心和情商，或者在处理文化敏感性时获得帮助。

在Comcast，一套内部定制工具旨在帮助坐席有效并高效地处理此类对话。然而，它仍然通常需要手动在多个地方查找信息，将其与客户的陈述相关联，然后根据沟通指南构建相关的回复。本文介绍了一项新的功能，名为“问我任何问题”（AMA）。它利用大语言模型（LLM）结合检索增强生成（RAG）方法，通过结合内部知识源、在构建时高效索引现有知识文章、在查询时检索相关文本块，然后将这些文本块输入到*Reader* LLM中生成简洁的答案，并提供引用作为参考。在下一节中，我们将更详细地描述该方法论。

## 2\. 方法论

我们的系统遵循典型的RAG实现，并进行了修改，以提高在专有问题上的性能。首先，对文档进行预处理，转化为文本并分块，然后对这些块进行嵌入，并将其与元数据（例如，关联的引用URL、标识符、标题等）一起存储在向量数据库中。我们在[2.1](https://arxiv.org/html/2405.00801v2#S2.SS1 "2.1\. 文档预处理 ‣ 2\. 方法论 ‣ “问我任何问题”：Comcast如何使用LLM实时协助坐席")节和[2.2](https://arxiv.org/html/2405.00801v2#S2.SS2 "2.2\. 检索相关文本片段 ‣ 2\. 方法论 ‣ “问我任何问题”：Comcast如何使用LLM实时协助坐席")节中分别描述了我们在处理和嵌入方面的具体选择，并提供了一些实验依据。接下来，我们在[2.3](https://arxiv.org/html/2405.00801v2#S2.SS3 "2.3\. 重新排序搜索结果 ‣ 2\. 方法论 ‣ “问我任何问题”：Comcast如何使用LLM实时协助坐席")节中详细介绍了如何使用合成数据训练和评估重新排序模型，以提高搜索结果的相关性。最后，我们在[2.4](https://arxiv.org/html/2405.00801v2#S2.SS4 "2.4\. 从片段生成答案 ‣ 2\. 方法论 ‣ “问我任何问题”：Comcast如何使用LLM实时协助坐席")节和[2.5](https://arxiv.org/html/2405.00801v2#S2.SS5 "2.5\. 离线响应评估 ‣ 2\. 方法论 ‣ “问我任何问题”：Comcast如何使用LLM实时协助坐席")节中讨论了如何生成答案以及如何评估系统。

### 2.1\. 文档预处理

我们接收来自各种内部客户的不同格式的文档。我们将文档标准化为纯文本，并使用Deepset.ai的Haystack库（Pietsch等，[2019](https://arxiv.org/html/2405.00801v2#bib.bib14)）将每个文档分块为片段。为了在检索后唯一地引用每个文档的每个片段，我们为每个文档分配一个来源标识符，并为每个片段分配一个本地标识符。最后，我们在每个文档上实施基于角色的访问控制，以便不同的用户只能查看他们有权限访问的文档。

在[1(a)](https://arxiv.org/html/2405.00801v2#S2.T1.st1 "1(a) ‣ 2.1\. 文档预处理 ‣ 2\. 方法论 ‣ “随便问我什么”：康卡斯特如何实时使用LLM帮助代理")中，我们展示了Haystack预处理器的各种分块参数及其评估分数。指标推导见[2.5节](https://arxiv.org/html/2405.00801v2#S2.SS5 "2.5\. 离线响应评估 ‣ 2\. 方法论 ‣ “随便问我什么”：康卡斯特如何实时使用LLM帮助代理")（答案质量假设前3个项已传递给LLM）。我们观察到通过设置更高的`max_chars_check`，有显著改进，该设置作为限制每个传递给LLM的片段大小的代理。

表 1. 三种不同设置的分块参数和评估。

| Parameter | A | B | C |
| --- | --- | --- | --- |
| clean_empty_lines | true |  |  |
| clean_whitespace | true |  |  |
| clean_header_footer | true |  |  |
| split_by | word |  |  |
| split_length | 300 | 100 |  |
| split_overlap | 50 | 25 |  |
| split_respect_sentence_boundary | true |  |  |
| max_chars_check | 1000 |  | 3000 |
| Metric |  |  |  |
| Answer Quality | - | -5.7% | +13.2% |
| MRR | - | -13.3% | 0.0% |
| R@3 | - | -7.9% | 0.0% |
| NDCG | - | -10.0% | 0.0% |

(a) 为了清晰起见，表格中仅显示设置$A$的变化。空的参数值表示与$A$相同。指标值为与$A$的相对差异，即$100\cdot(\mu_{B}-\mu_{A})/\mu_{A}$，其中$\mu$是某个指标。指标定义见[2.5节](https://arxiv.org/html/2405.00801v2#S2.SS5 "2.5\. 离线响应评估 ‣ 2\. 方法论 ‣ “随便问我什么”：康卡斯特如何实时使用LLM帮助代理")。

### 2.2\. 检索相关文本片段

为了帮助选择我们的检索模型，我们在一个精心挑选的评估集上进行了初步实验，该评估集包含五十个问答对。我们在生产系统日志中搜索了以WH词（如who, what, how等）开头或以问号结尾的查询，粗略地遵循了Bing查询日志中的程序，参考了WikiQA数据集（Yang等，[2015](https://arxiv.org/html/2405.00801v2#bib.bib25)）。对于每个问题，我们随后在内部知识库中找到了相关段落和答案范围，这些知识库是由代理使用的。没有答案的查询也被标记为没有答案。至关重要的是，这个过程避免了回溯构造（Sakai等，[2004](https://arxiv.org/html/2405.00801v2#bib.bib18)），在这种情况下，查询是由标注者基于已知段落手动编写的，而不是从日志中抓取的，这导致了偏差的评估集。

我们实验了稠密和稀疏检索模型。对于稀疏模型，我们使用了Okapi BM25（Robertson等，[2009](https://arxiv.org/html/2405.00801v2#bib.bib17)），其中$k_{1}=1.0$，$b=0.5$。对于稠密模型，我们实验了四种：稠密段落检索（DPR）（Karpukhin等，[2020](https://arxiv.org/html/2405.00801v2#bib.bib10)），在自然问题（Natural Questions）数据集上进行微调（Kwiatkowski等，[2019](https://arxiv.org/html/2405.00801v2#bib.bib11)）；MPNet-base（v1）（Song等，[2020](https://arxiv.org/html/2405.00801v2#bib.bib19)），在包含维基百科、BookCorpus（Zhu等，[2015](https://arxiv.org/html/2405.00801v2#bib.bib27)）和OpenWebText（Gokaslan和Cohen，[2019](https://arxiv.org/html/2405.00801v2#bib.bib7)）的160GB文本语料库上训练；OpenAI的最先进的`ada-002`嵌入模型；以及MPNet-base v2，进一步在十亿个句子对上进行训练，以提高嵌入质量。²² Nils Reimers的开源贡献：[https://discuss.huggingface.co/t/train-the-best-sentence-embedding-model-ever-with-1b-training-pairs/7354](https://discuss.huggingface.co/t/train-the-best-sentence-embedding-model-ever-with-1b-training-pairs/7354)。每个模型都被认为能满足我们推理时的计算和财务约束。

在[2(a)](https://arxiv.org/html/2405.00801v2#S2.T2.st1 "2(a) ‣ 2.2\. Retrieving Relevant Text Snippets ‣ 2\. Methodology ‣ “Ask Me Anything”: How Comcast Uses LLMs to Assist Agents in Real Time")中，我们报告了这些模型在我们评估集上的recall@3（R@3）和平均倒排排名（MRR）。选择recall@3（与recall@5或10相比）是因为我们将前三个检索到的段落输入到LLM中。作为一种理性检查，我们还进行了一个基线实验，随机抽取一个段落，结果自然得到了较低的分数。与先前的工作（Yang等，[2019](https://arxiv.org/html/2405.00801v2#bib.bib24)）相似，我们发现BM25仍然是一个强大的基线，分别在R@3和MRR上超过了DPR。我们推测这可能是因为自然问题数据集与我们的数据领域差异较大。

表2. 各种检索器在我们初步评估集上的结果

| 方法 | Recall@3 | MRR |
| --- | --- | --- |
| 随机 | -71.4% | -83.9% |
| BM25 | - | - |
| DPR (`single-nq`) | -42.8% | -42.9% |
| DPR (`multiset-nq`) | -23.8% | -29.0% |
| Multi-QA MPNet-base | +33.0% | +39.7% |
| OpenAI嵌入（`ada-002`） | +33.0% | +53.9% |
| MPNet-base v2 | +38.1% | +54.9% |

(a) 统计数据以相对于BM25的差异呈现，即 $100\cdot(\mu-\mu_{BM25})/\mu_{BM25}$。下划线表示相对于DPR的统计显著性。

我们观察到MPNet-base (v1)、OpenAI的`ada-002`和MPNet-base (v2)的表现相似。R@3的符号秩检验和MRR的$t$检验也显示与DPR存在显著差异（$p<0.05$）。由于操作上的便利性和OpenAI ADA嵌入的高性能，我们在最终系统的检索器组件中使用了ADA。

对于我们的生产检索步骤，我们将文章标题和单独片段的文本嵌入，并在存储到向量数据库之前将它们合并。根据经验，我们发现这种方法能为各种查询提供更全面的检索，尤其是在片段缺少一些文章主题描述性上下文时。

### 2.3\. 结果重排序

我们发现，使用在合成数据上微调的模型进行重排序能改善检索步骤。我们的方法受到之前合成数据生成方法的启发（Bonifacio等人，[2022](https://arxiv.org/html/2405.00801v2#bib.bib2)；Dai等人，[2022](https://arxiv.org/html/2405.00801v2#bib.bib4)）。首先，我们使用GPT-4从数据集中的每个片段生成合成问题。然后，我们使用OpenAI的`text-embeddings-ada-002`（Greene等人，[2022](https://arxiv.org/html/2405.00801v2#bib.bib9)）嵌入将每个问题传入我们的搜索系统。对于任何未能出现在前20名结果中的问题，我们将其丢弃。对于每个合成问题，我们保存了检索到的前20项、它们的相关性评分（由`BGE-reranker-large`（Xiao等人，[2023](https://arxiv.org/html/2405.00801v2#bib.bib23)）评分）以及指示该片段为问题源的标记。最终排名通过首先将源片段作为“最相关”的结果，然后按`BGE-reranker-large`模型评分的最相关顺序排列其他片段来确定。

对于训练，我们使用了RankNet（Burges et al., [2005](https://arxiv.org/html/2405.00801v2#bib.bib3)）将这些排名提炼到一个微调后的MPNet（Song et al., [2020](https://arxiv.org/html/2405.00801v2#bib.bib19)），特别是来自`sentence-transformer`的`all-mpnet-base-v2`（Reimers and Gurevych, [2019](https://arxiv.org/html/2405.00801v2#bib.bib16)），它比`BGE-reranker-large`需要更少的参数，且在生产环境中部署时需要更少的计算资源。构建RankNet所需的必要对之后，最终的数据集包含超过1000万个示例。我们将0.5%的示例作为验证集。我们的训练参数列在[表3](https://arxiv.org/html/2405.00801v2#S2.T3 "Table 3 ‣ 2.3\. Reranking Search Results ‣ 2\. Methodology ‣ “Ask Me Anything”: How Comcast Uses LLMs to Assist Agents in Real Time")中。我们使用了PyTorch的`DistributedDataParallel`（Paszke et al., [2017](https://arxiv.org/html/2405.00801v2#bib.bib13)）进行分布式训练，因此有效批量大小是GPU数量乘以批量大小。我们发现“线性缩放规则”，即当批量大小增加时调整学习率，不适用于我们的使用场景（Goyal et al., [2018](https://arxiv.org/html/2405.00801v2#bib.bib8)），但我们怀疑这是因为原始MPNet架构是在比我们微调时使用的更大批量大小下训练的。

[b] 参数规格 学习率 $5\times 10^{-6}$ 批量大小 8 GPU数量¹ 10 预热步数 4000 权重衰减 0.001 训练轮数 1 总训练步数 171391 学习率调度器 预热-常数

表3. 训练超参数。

+   1

    GPU类型：g4dn.xlarge（Nvidia T4）

为了进一步评估我们的重新排序模型的性能，我们随机抽取了10000个由客服人员在生产系统中提出的真实问题。对于每个检索到的文档，我们遵循了（Thomas et al., [2023](https://arxiv.org/html/2405.00801v2#bib.bib21)）中的方法，该方法显示大语言模型（LLM）可以准确预测搜索结果的相关性。具体而言，使用GPT-4评估每个文档与问题的整体质量，结合了文档与问题意图匹配度的得分以及文档的可信度。最终的整数得分范围是0到2，得分越高，表示整体质量越高。[表4](https://arxiv.org/html/2405.00801v2#S2.T4 "Table 4 ‣ 2.3\. Reranking Search Results ‣ 2\. Methodology ‣ “Ask Me Anything”: How Comcast Uses LLMs to Assist Agents in Real Time") 比较了ADA与重新排序模型在多个指标上的表现。由于整体得分是非二进制的，我们通过计算得分为2的首个文档的排名来计算MRR，并且使用Recall@3检查前3名文档是否包含得分为2的文档。结果表明，使用重新排序模型提高了检索性能。

表4. 使用生产问题的ADA与重新排序模型的搜索结果

| 度量 | ADA | 重新排序 |
| --- | --- | --- |
| Recall@3 | - | +12% |
| MRR | - | +15% |
| NDCG | - | +4.8% |

(a) 为了清晰起见，表格中仅展示了从设置ADA后的变化。度量值是与ADA的相对差异。

### 2.4\. 从片段生成答案

在生成答案时，我们遵循RAG文献中的常规方法。我们首先用一段模型指导原则的前言开始提示，然后是任务描述。由于知识库中文本片段的长度，我们无法提供少量示例。我们通过经验发现，包含更多文本更为有效，以避免信息被随机截断。为了避免出现“中途丢失”问题（Liu等人，[2023](https://arxiv.org/html/2405.00801v2#bib.bib12)），我们在传入LLM时反转Top K结果的顺序，格式化为XML，捕获结果的ID、标题和内容。我们在生产环境中使用了OpenAI的`gpt-3.5-turbo`作为Reader组件。作为提示的最后一步，我们要求LLM根据搜索结果回答给定问题。

#### 2.4.1\. 引用

AMA解决方案的一个重要产品特性是为代理提供参考，以便他们可以进一步了解所给出的答案。这在各种RAG实现中都有体现，例如微软的Copilot。此外，目标是建立对系统输出的信任，并推动内部采纳。受到事实核查轨道（Rebedea等人，[2023](https://arxiv.org/html/2405.00801v2#bib.bib15)）的启发，我们的引用轨道是通过提示LLM以特定方式引用其来源来实现的（参见图[1](https://arxiv.org/html/2405.00801v2#S2.F1 "图1 ‣ 2.4.1\. 引用 ‣ 2.4\. 从片段生成答案 ‣ 2\. 方法论 ‣ “随便问我任何问题”：Comcast如何利用LLM实时辅助代理")），结合后处理步骤，在该步骤中引用被从文本中移除。如果未找到引用，则系统不会返回答案。从可观察性的角度来看，这种方法有另一个好处：通过这种方法，我们识别出了大多数“无答案”响应，因为通常LLM会类似地回应“对不起，我在文档中未能找到答案”，并且没有引用。

[⬇](data:text/plain;base64,UGxlYXNlIGluY2x1ZGUgYSBzaW5nbGUgc291cmNlIGF0IHRoZSBlbmQgb2YgeW91ciBhbnN3ZXIsIGkuZS4sIFtEb2N1bWVudDBdIGlmIERvY3VtZW50MCBpcyB0aGUgc291cmNlLiBJZiB0aGVyZSBpcyBtb3JlIHRoYW4gb25lIHNvdXJjZSwgdXNlIFtEb2N1bWVudDBdW0RvY3VtZW50MV0gaWYgRG9jdW1lbnQwIGFuZCBEb2N1bWVudDEgYXJlIHRoZSBzb3VyY2VzLg==)请在答案的末尾包括一个来源，例如，如果Document0是来源，则使用[Document0]。如果有多个来源，则使用[Document0][Document1]。

图1. 一个示例组件，用于鼓励LLM在系统提示部分引用来源。

### 2.5\. 离线响应评估

为了评估系统的响应，我们采用了LLM作为裁判的方法（Zhu et al., [2023](https://arxiv.org/html/2405.00801v2#bib.bib26)），除此之外，还考虑了典型的搜索系统的检索质量度量。特别是，我们从生产流量中随机抽取了一些客户的问题。然后，人工注释员使用AMA系统也可用的内部知识库为每个查询编写了正确的答案。我们能够使用GPT-4计算“答案质量”，将系统的答案与人工注释员给出的正确答案进行比较。

对于每个问题，注释员还提供了他们回答所依据的引用。我们利用这些引用计算了“引用匹配率”：即AMA系统中的引用与真实情况匹配的案例百分比。由于我们的检索步骤返回的是一个列表，我们通过假设注释引用是唯一相关文档来计算Recall@K。

表格[5](https://arxiv.org/html/2405.00801v2#S2.T5 "Table 5 ‣ 2.5\. 离线响应评估 ‣ 2\. 方法论 ‣ “随便问我什么”：康卡斯特如何实时利用LLM辅助客服")展示了与表格[4](https://arxiv.org/html/2405.00801v2#S2.T4 "Table 4 ‣ 2.3\. 重新排序搜索结果 ‣ 2\. 方法论 ‣ “随便问我什么”：康卡斯特如何实时利用LLM辅助客服")相同的两种方法的关键指标（`text-embedding-ada-002`用于密集检索相关文档，并使用我们微调后的模型对ADA检索到的文档进行重新排序）。我们观察到，使用重新排序的文档后，LLM能够实现更高的答案质量，这意味着来自不同文档排序的答案在GPT-4中更为准确。这一改善也可以通过引用匹配率和Recall@3的提高来解释，重新排序的文档直接影响LLM的回答准确性。

表格 5. 响应质量

| 指标 | ADA | 重新排序器 |
| --- | --- | --- |
| 答案质量 | - | +5.9% |
| 引用匹配率 | - | +2.5% |
| Recall@3 | - | +16.5% |

(a) 为了清晰起见，表格中仅列出了来自ADA设置的变化。度量值是与ADA的相对差异。

## 3\. 将AMA部署到客户服务代表

由于业务敏感性原因，本节将隐藏与货币业务指标相关的一些细节。该系统于2023年底在数百名聊天代理中进行了试点。在为期一个月的试用期中，当代理使用AMA（问我任何）时，相比传统的搜索选项（需要代理打开新工具并执行搜索），聊天处理时间提高了10%。我们认为这是一个良好的回答质量代理指标，因为AMA提供的不准确或不完整的回答会要求代理重新开始，并转向传统选项，从而重复工作并总体上花费更多时间。通过简单的点赞/点踩UI元素收集了代理的明确反馈，正面反馈率接近80%（由于在此功能发布之前未要求此类反馈，因此没有该反馈的基准值）。试用期结束后，系统很快推广到所有聊天代理（数千人），并且以AMA驱动的搜索成为首选搜索方式，占所有输入查询的三分之二。

## 4\. 在线重新排序实验

在[3](https://arxiv.org/html/2405.00801v2#S3 "3\. Deploying AMA to Customer Service Agents ‣ “Ask Me Anything”: How Comcast Uses LLMs to Assist Agents in Real Time")节的试验结束后不久，我们开始了[2.3](https://arxiv.org/html/2405.00801v2#S2.SS3 "2.3\. Reranking Search Results ‣ 2\. Methodology ‣ “Ask Me Anything”: How Comcast Uses LLMs to Assist Agents in Real Time")节中描述的重新排序模块的A/B测试。对照组仅使用ADA嵌入进行向量检索，没有重新排序组件，而处理组则在ADA基础向量检索步骤中的前$20$个结果上使用了重新排序组件。该测试于2024年初运行了三周。我们将测试的功效设置为80%，并且对每个互动适用的度量标准使用显著性水平$\alpha=.01$，对考虑用户反馈的度量标准使用$\alpha=.05$，因为反馈稀疏。由于代理数量有限，我们的随机化单元采用了类似于其他大型系统中的cookie-day随机化的方法（Tang等，[2010](https://arxiv.org/html/2405.00801v2#bib.bib20)），以增加统计功效。文献中已表明（Deng等，[2017](https://arxiv.org/html/2405.00801v2#bib.bib6)）独立同分布（IID）假设的违反可能导致方差的低估，但通过使用较小的显著性阈值和观察到更大的效应量时，这些测试在实践中仍然可以被认为是可信的。我们使用了delta方法（Deng等，[2018](https://arxiv.org/html/2405.00801v2#bib.bib5)）来估算基于问题级别的度量的方差。

我们观察到两个度量指标有显著的统计学增长：即“无答案率”，这是指没有答案的查询数量与总查询数量的比率，以及“积极反馈率”，定义为点赞数与收到的反馈总数的比率。下游业务指标，如平均处理时间和升级率，则未显示出显著差异。然而，无答案率的改善表明，系统能够通过向大型语言模型（LLM）提供相关文档，从而处理比之前更多的问题，并且还提高了积极反馈的比率。

表 6. A/B 测试结果

| 指标 | 效果 | p 值 |
| --- | --- | --- |
| 无答案率 | -11.9% | p < .001 |
| 积极反馈率 | +8.9% | p < .05 |

(a) 表格展示了相对于控制组的相对变化作为效果。对于无答案率，较低值为更佳。

## 5. 结论

在本文中，我们介绍了 AMA，一种大规模解决方案，旨在满足一个常见的业务需求：高效的高质量客户服务。通过使用第三方大型语言模型（LLMs）和经过验证的 RAG 方法，我们能够快速构建 AMA，并作为辅助功能展示了其明显的价值。我们展示了通过选择特定的文档预处理、检索模型及其嵌入，以及自定义的重排序模型，来改善检索和答案质量。当我们将 AMA 部署到成千上万的代理商中并带来实际的业务收益时，我们相信这为人类与 AI 如何合作，更好地服务客户提供了一个很好的例子。

## 参考文献

+   (1)

+   Bonifacio 等人（2022）Luiz Bonifacio, Hugo Abonizio, Marzieh Fadaee 和 Rodrigo Nogueira. 2022. InPars：使用大型语言模型的数据增强信息检索。arXiv:2202.05144 [cs.CL]

+   Burges 等人（2005）Chris Burges, Tal Shaked, Erin Renshaw, Ari Lazier, Matt Deeds, Nicole Hamilton 和 Greg Hullender. 2005. 使用梯度下降学习排序。发表于 *第22届国际机器学习大会论文集*（德国波恩）*(ICML ’05)*。美国纽约，计算机协会，89–96。[https://doi.org/10.1145/1102351.1102363](https://doi.org/10.1145/1102351.1102363)

+   Dai 等人（2022）Zhuyun Dai, Vincent Y. Zhao, Ji Ma, Yi Luan, Jianmo Ni, Jing Lu, Anton Bakalov, Kelvin Guu, Keith B. Hall 和 Ming-Wei Chang. 2022. Promptagator：来自 8 个示例的少量密集检索。arXiv:2209.11755 [cs.CL]

+   Deng 等人（2018）Alex Deng, Ulf Knoblich 和 Jiannan Lu. 2018. 《在度量分析中应用 Delta 方法：带有新颖思路的实用指南》。发表于 *第24届 ACM SIGKDD 国际知识发现与数据挖掘大会论文集*（英国伦敦）*(KDD ’18)*。美国纽约，计算机协会，233–242。[https://doi.org/10.1145/3219819.3219919](https://doi.org/10.1145/3219819.3219919)

+   Deng 等人（2017）Alex Deng, Jiannan Lu, 和 Jonthan Litz. 2017. 在线A/B测试的可信分析：陷阱、挑战和解决方案。在*第十届ACM国际网页搜索与数据挖掘会议论文集*（英国剑桥）*(WSDM ’17)*. 美国计算机协会，纽约，纽约，美国，641–649. [https://doi.org/10.1145/3018661.3018677](https://doi.org/10.1145/3018661.3018677)

+   Gokaslan 和 Cohen（2019）Aaron Gokaslan 和 Vanya Cohen. 2019. OpenWebText 数据集。 [http://Skylion007.github.io/OpenWebTextCorpus](http://Skylion007.github.io/OpenWebTextCorpus).

+   Goyal 等人（2018）Priya Goyal, Piotr Dollár, Ross Girshick, Pieter Noordhuis, Lukasz Wesolowski, Aapo Kyrola, Andrew Tulloch, Yangqing Jia, 和 Kaiming He. 2018. 精确的、大批次的SGD：1小时内训练ImageNet。arXiv:1706.02677 [cs.CV]

+   Greene 等人（2022）Ryan Greene, Ted Sanders, Lilian Weng, 和 Arvind Neelakantan. 2022. *全新改进的嵌入模型*。 [https://openai.com/blog/new-and-improved-embedding-model](https://openai.com/blog/new-and-improved-embedding-model)

+   Karpukhin 等人（2020）Vladimir Karpukhin, Barlas Oguz, Sewon Min, Patrick Lewis, Ledell Wu, Sergey Edunov, Danqi Chen, 和 Wen-tau Yih. 2020. 用于开放域问答的密集段落检索。在*2020年自然语言处理实证方法会议（EMNLP）论文集*。6769–6781.

+   Kwiatkowski 等人（2019）Tom Kwiatkowski, Jennimaria Palomaki, Olivia Redfield, Michael Collins, Ankur Parikh, Chris Alberti, Danielle Epstein, Illia Polosukhin, Jacob Devlin, Kenton Lee 等人. 2019. Natural questions: 一个问答研究基准。*计算语言学协会会刊* 7 (2019), 453–466.

+   Liu 等人（2023）Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, 和 Percy Liang. 2023. 迷失在中间：语言模型如何使用长上下文。arXiv:2307.03172 [cs.CL]

+   Paszke 等人（2017）Adam Paszke, Sam Gross, Soumith Chintala, Gregory Chanan, Edward Yang, Zachary DeVito, Zeming Lin, Alban Desmaison, Luca Antiga, 和 Adam Lerer. 2017. PyTorch中的自动微分。（2017年）

+   Pietsch 等人（2019）Malte Pietsch, Timo Möller, Bogdan Kostic, Julian Risch, Massimiliano Pippi, Mayank Jobanputra, Sara Zanzottera, Silvano Cerza, Vladimir Blagojevic, Thomas Stadelmann, Tanay Soni, 和 Sebastian Lee. 2019. Haystack: 为务实构建者设计的端到端NLP框架。 [https://github.com/deepset-ai/haystack](https://github.com/deepset-ai/haystack).

+   Rebedea 等人（2023）Traian Rebedea, Razvan Dinu, Makesh Sreedhar, Christopher Parisien, 和 Jonathan Cohen. 2023. NeMo Guardrails: 一个用于可控和安全LLM应用的可编程工具包。arXiv:2310.10501 [cs.CL]

+   Reimers and Gurevych (2019) Nils Reimers 和 Iryna Gurevych. 2019. Sentence-BERT：使用孪生BERT网络的句子嵌入。发表于 *2019年自然语言处理实证方法会议论文集*。计算语言学协会。[http://arxiv.org/abs/1908.10084](http://arxiv.org/abs/1908.10084)

+   Robertson et al. (2009) Stephen Robertson, Hugo Zaragoza, 等. 2009. 概率相关框架：BM25及其发展。 *信息检索基础与趋势* 3, 4 (2009)，333–389。

+   Sakai et al. (2004) Tetsuya Sakai, Yoshimi Saito, Yumi Ichimura, Tomoharu Kokubu, 和 Makoto Koyama. 2004. 在问答评估中反向公式化问题的效果。发表于 *第27届国际ACM SIGIR信息检索研究与发展年会论文集*。474–475。

+   Song et al. (2020) Kaitao Song, Xu Tan, Tao Qin, Jianfeng Lu, 和 Tie-Yan Liu. 2020. MPNet：用于语言理解的掩蔽与排列预训练。arXiv:2004.09297 [cs.CL]

+   Tang et al. (2010) Diane Tang, Ashish Agarwal, Deirdre O’Brien, 和 Mike Meyer. 2010. 重叠实验基础设施：更多、更好、更快的实验。发表于 *第16届知识发现与数据挖掘会议论文集*。华盛顿特区，17–26。

+   Thomas et al. (2023) Paul Thomas, Seth Spielman, Nick Craswell, 和 Bhaskar Mitra. 2023. 大型语言模型能够准确预测搜索者偏好。arXiv:2309.10621 [cs.IR]

+   Wowak ([n. d.]) Kaitlin Wowak. [n. d.]. 人类与自动化：服务中心代理可以超越技术，研究显示。 [https://news.nd.edu/news/humans-vs-automation-service-center-agents-can-outperform-technology-study-shows/](https://news.nd.edu/news/humans-vs-automation-service-center-agents-can-outperform-technology-study-shows/)

+   Xiao et al. (2023) Shitao Xiao, Zheng Liu, Peitian Zhang, 和 Niklas Muennighoff. 2023. C-Pack：推进通用中文嵌入的打包资源。arXiv:2309.07597 [cs.CL]

+   Yang et al. (2019) Wei Yang, Kuang Lu, Peilin Yang, 和 Jimmy Lin. 2019. 批判性地审视“神经炒作”：神经排序模型在有效性提升的加性和薄弱基准。发表于 *第42届国际ACM SIGIR信息检索研究与发展会议论文集*。1129–1132。

+   Yang et al. (2015) Yi Yang, Wen-tau Yih, 和 Christopher Meek. 2015. WikiQA：一个开放领域问答挑战数据集。发表于 *2015年自然语言处理实证方法会议论文集*。2013–2018。

+   Zhu et al. (2023) Lianghui Zhu, Xinggang Wang, 和 Xinlong Wang. 2023. JudgeLM：微调的大型语言模型是可扩展的裁判。arXiv:2310.17631 [cs.CL]

+   Zhu 等人（2015）Yukun Zhu、Ryan Kiros、Rich Zemel、Ruslan Salakhutdinov、Raquel Urtasun、Antonio Torralba 和 Sanja Fidler。2015年。对齐书籍与电影：通过观看电影和阅读书籍实现类似故事的视觉解释。在*IEEE国际计算机视觉大会论文集*中，19–27页。
