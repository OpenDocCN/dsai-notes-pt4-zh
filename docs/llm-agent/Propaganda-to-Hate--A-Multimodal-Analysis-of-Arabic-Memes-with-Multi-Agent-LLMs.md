<!--yml

类别：未分类

日期：2025-01-11 12:15:57

-->

# 从宣传到仇恨：基于多代理大语言模型的阿拉伯表情包的多模态分析

> 来源：[https://arxiv.org/html/2409.07246/](https://arxiv.org/html/2409.07246/)

¹¹机构文本：¹卡塔尔计算研究所，卡塔尔，²哈马德·本·哈利法大学，卡塔尔，

³卡塔尔的西北大学，卡塔尔

fialam@hbku.edu.qa

^((✉))通讯作者Firoj Alam^✉ 11 [0000-0001-7172-1997](https://orcid.org/0000-0001-7172-1997 "ORCID 标识符")    Md. Rafiul Biswas 22 [0000-0002-5145-1990](https://orcid.org/0000-0002-5145-1990 "ORCID 标识符")    Uzair Shah 22 [0000-0002-6729-5654](https://orcid.org/0000-0002-6729-5654 "ORCID 标识符")    Wajdi Zaghouani 33 [0000-0003-1521-5568](https://orcid.org/0000-0003-1521-5568 "ORCID 标识符")    Georgios Mikros 22 [0000-0002-4093-5973](https://orcid.org/0000-0002-4093-5973 "ORCID 标识符")

###### 摘要

在过去的十年里，社交媒体平台被广泛用于信息传播和消费。虽然大部分内容是发布来促进公民新闻和公众意识的，但也有部分内容被发布以误导用户。在文本、图像和视频等不同类型的内容中，表情包（在图像上叠加文本）尤为流行，且能够成为宣传、仇恨和幽默的强大载体。在现有文献中，已经有研究尝试单独检测表情包中的此类内容。然而，对于这些内容交集的研究非常有限。在本研究中，我们通过基于多代理大语言模型的方法，探索表情包中宣传与仇恨之间的交集。我们通过粗粒度和细粒度的仇恨标签扩展了宣传表情包数据集。我们的发现表明，表情包中的宣传与仇恨之间存在关联。我们提供了详细的实验结果，这些结果可以作为未来研究的基准。我们将公开实验资源，供社区使用。¹¹1[https://github.com/firojalam/propaganda-and-hateful-memes.git](https://github.com/firojalam/propaganda-and-hateful-memes.git)

###### 关键词：

宣传仇恨表情包多模态大语言模型

## 1 引言

社交媒体已成为一个主要的在线内容分享渠道。它的指数增长显著改变了信息传播的格局。然而，这些平台的滥用使其成为传播不当内容、虚假信息和误导性信息的温床[[2](https://arxiv.org/html/2409.07246v2#bib.bib2)]。社交媒体上的互动促进了关于从地方问题到政治等一系列话题的公共讨论，但它们也通过各种内容类型——文本、图像和视频——滋生和传播仇恨言论和攻击性内容[[26](https://arxiv.org/html/2409.07246v2#bib.bib26), [16](https://arxiv.org/html/2409.07246v2#bib.bib16), [31](https://arxiv.org/html/2409.07246v2#bib.bib31), [2](https://arxiv.org/html/2409.07246v2#bib.bib2)]。

为了解决不同模态中的此类问题，已有许多努力尝试使用单模态和多模态建模方法自动检测它们[[9](https://arxiv.org/html/2409.07246v2#bib.bib9)]。在宣传内容检测方面，研究主要集中在定义技术并解决各种内容类型（包括新闻文章、推文、表情包和多种语言的文本内容）中的问题[[13](https://arxiv.org/html/2409.07246v2#bib.bib13), [4](https://arxiv.org/html/2409.07246v2#bib.bib4), [28](https://arxiv.org/html/2409.07246v2#bib.bib28)]。类似地，在仇恨言论检测方面也进行了大量的努力[[29](https://arxiv.org/html/2409.07246v2#bib.bib29), [10](https://arxiv.org/html/2409.07246v2#bib.bib10)]。表情包研究中的一个重要举措是仇恨表情包挑战赛[[23](https://arxiv.org/html/2409.07246v2#bib.bib23)]，它激发了许多后续研究。

我们的研究位于多模态内容分析、宣传检测和仇恨言论识别的交叉领域。尽管在英语及其他高资源语言领域已取得了一定进展，但对于阿拉伯语的研究仍然较为缺乏。在参考了[[5](https://arxiv.org/html/2409.07246v2#bib.bib5)]和[[18](https://arxiv.org/html/2409.07246v2#bib.bib18)]在阿拉伯语宣传检测方面的工作基础上，我们的研究分析了阿拉伯语表情包，填补了文献中的空白。研究结果可以帮助社交媒体平台、政策制定者和民间组织应对有害的网络内容。

我们工作的关键方面如下：（i）我们提出了一种新颖的基于多智能体大型语言模型（LLM）的方法，用于分析阿拉伯语社交媒体内容中宣传和仇恨表情包之间的关联；（ii）我们展示了多智能体LLM系统在复杂、多模态数据自动标注中的应用，提供了一种在资源匮乏环境下处理大量数据的可扩展解决方案；（iii）除了粗粒度的仇恨类别外，我们还探索了仇恨和非仇恨表情包的细粒度分类；（iv）我们提供了粗粒度和细粒度分类的实验结果；（v）最后，我们将把仇恨表情包数据集提供给社区。

## 2 相关工作

### 2.1 宣传内容

文本内容：近年来，宣传内容的研究引起了广泛关注。Da 等人（2020）引入了一个大规模数据集，用于新闻文章中的细粒度宣传检测，呈现了一个包含350K句子、标注了18种宣传技巧的语料库[[8](https://arxiv.org/html/2409.07246v2#bib.bib8)]。该标注方案已扩展为包含23种技巧，并在[[28](https://arxiv.org/html/2409.07246v2#bib.bib28)]中提出了多语言语料库。遵循相同的标注方案，已经为阿拉伯语开发了数据集，并组织了共享任务[[4](https://arxiv.org/html/2409.07246v2#bib.bib4), [20](https://arxiv.org/html/2409.07246v2#bib.bib20), [19](https://arxiv.org/html/2409.07246v2#bib.bib19)]。

多模态内容：在文本内容的先前研究基础上，Dimitrov 等人（2021）引入了SemEval-2021任务6，该任务聚焦于检测表情包中文本和图像中的劝说技巧[[14](https://arxiv.org/html/2409.07246v2#bib.bib14)]。随后，研究重点扩展到了包括检测多语言和多模态宣传表情包[[12](https://arxiv.org/html/2409.07246v2#bib.bib12)]。类似地，阿拉伯语相关研究涉及到数据集的开发和宣传检测的共享任务[[3](https://arxiv.org/html/2409.07246v2#bib.bib3), [21](https://arxiv.org/html/2409.07246v2#bib.bib21)]。Fang 等人（2022）使用独立的网络来嵌入文本和图像，将这些多模态嵌入进行融合。采用带有多级表示的拆分共享模块，以改善劝说技巧检测[[15](https://arxiv.org/html/2409.07246v2#bib.bib15)]。

### 2.2 仇恨表情包

仇恨表情包的研究面临独特的挑战，因为它们具有多模态特性。Kiela 等人（2020）引入了仇恨表情包挑战，这是一个用于多模态仇恨检测的大规模数据集和基准。这项工作强调了整合文本和视觉元素的重要性，以便识别表情包中的仇恨言论[[23](https://arxiv.org/html/2409.07246v2#bib.bib23)]。为了解决低资源语言的问题，也已经开发了多种语言的数据集[[22](https://arxiv.org/html/2409.07246v2#bib.bib22)]。在[[31](https://arxiv.org/html/2409.07246v2#bib.bib31)]中，作者提供了有关多模态和有害表情包的详细调查，强调了该问题的重要性，并提出了未来的研究方向。

### 2.3 内容分析中的多智能体系统

多代理系统在内容分析中的应用是一个新兴领域，这可能是分析各种媒体中复杂叙事的有效方法[[17](https://arxiv.org/html/2409.07246v2#bib.bib17)]。Chen等人（2021）提出了一种基于多代理强化学习的动态内容审核系统，该系统根据用户交互和内容模式进行自适应调整，以提高有害内容的检测[[7](https://arxiv.org/html/2409.07246v2#bib.bib7)]。这些研究强调了多代理系统在分析复杂的、往往带有宣传性或仇恨内容的表情包中的价值。在此基础上，我们的研究聚焦于阿拉伯语表情包，采用多代理LLM方法。我们探索了宣传与仇恨表情包之间的关联，尤其是在低资源环境下。我们使用LLM作为多个代理来自动化数据标注过程，展示了LLM作为数据标注员在检测仇恨表情包中的应用价值。

## 3 数据集

### 3.1 宣传表情包

在本研究中，我们使用了ArAIEval-2024数据集[[21](https://arxiv.org/html/2409.07246v2#bib.bib21)]，该数据集包含约$\sim$3k个表情包，每个表情包都标注为宣传性或非宣传性。这些表情包收集自各种社交媒体平台，包括Facebook、Twitter、Instagram和Pinterest。我们由三名标注员对每个表情包进行标注，最终标签由多数票决定。表情包中的文本通过一款现成的OCR工具提取²²2[https://github.com/JaidedAI/EasyOCR](https://github.com/JaidedAI/EasyOCR)，并对宣传性表情包进行了人工修正。宣传性和非宣传性标签的分布分别为40%和60%。该数据集的更多详细信息请参见[[21](https://arxiv.org/html/2409.07246v2#bib.bib21)]，而全面的标注指南见[[3](https://arxiv.org/html/2409.07246v2#bib.bib3)]。在实验中，数据集按70%、10%和20%的比例划分为训练集、开发集和测试集。

![参见说明](img/4a8672b0864cbf61b2a543884d9accc6.png)

图1：使用LLM代理作为标注员和汇总员的实验流程。

### 3.2 仇恨内容及细粒度分类

对于仇恨内容及其细粒度分类，我们使用了前面提到的ArAIEval-2024数据集。我们选择使用ArAIEval-2024数据集的原因是，这是目前唯一可用的阿拉伯语表情包数据集，且已对其进行过宣传内容的标注。另一个动机是理解宣传内容与仇恨表情包之间的关联。在图[1](https://arxiv.org/html/2409.07246v2#S3.F1 "图 1 ‣ 3.1 宣传表情包 ‣ 3 数据集 ‣ 从宣传到仇恨：基于多代理LLM的阿拉伯语表情包多模态分析")中，我们提供了从数据准备到分类实验的完整流程。

#### 3.2.1 LLM代理作为标注员

为了使用LLM代理作为标注员，我们选择了三种知名且表现优异的商业模型：OpenAI的GPT-4o [[27](https://arxiv.org/html/2409.07246v2#bib.bib27)]，Google的Gemini Pro（版本1.5）[[32](https://arxiv.org/html/2409.07246v2#bib.bib32)]，以及Claude 3.5（Sonnet）。³³3[https://www.anthropic.com/news/claude-3-5-sonnet](https://www.anthropic.com/news/claude-3-5-sonnet)。在标注过程中，我们使用与[[18](https://arxiv.org/html/2409.07246v2#bib.bib18)]中讨论的相同手动程序，该程序采用了两阶段的方法。在第一阶段，称为标注阶段，三名标注员独立地根据[3.2.2](https://arxiv.org/html/2409.07246v2#S3.SS2.SSS2 "3.2.2 Manual Annotation ‣ 3.2 Hatefulness and Fine-grained Categories ‣ 3 Dataset ‣ Propaganda to Hate: A Multimodal Analysis of Arabic Memes with Multi-Agent LLMs")中列出的指南标注表情包。在第二阶段，称为合并阶段，我们审核并解决在第一阶段标注过程中出现的任何分歧。如图所示，图中的深红色部分突出显示了我们在第一阶段使用LLM代理作为标注员，在第二阶段则使用LLM代理作为合并员。在每个阶段，我们为LLM代理提供了一个零样本设置的特定提示。根据下面讨论的标注指南，我们要求LLM代理执行两项任务：（任务1）标注每个表情包为“仇恨”或“非仇恨”；（任务2）根据任务1中的标签，提供细粒度的分类。例如，如果任务1中标注为“仇恨”，则应根据下面提到的八个类别之一提供细粒度标签。第二阶段的提示略有不同。这里的任务还涉及考虑第一阶段获得的标签以做出最终决策。在这一阶段，我们尝试使用GPT-4o作为合并员。

我们使用这种基于LLM的多代理方法来处理训练集和开发集（dev）。为了验证多代理方法的质量，我们通过将每个LLM代理提供的标签与测试数据集上的人工标注标签进行比较，量化了这些标签的质量。

#### 3.2.2 手动标注

为了验证基于LLM的多代理方法，我们手动标注了来自ArAIEval-2024数据集的一个测试集，如图[1](https://arxiv.org/html/2409.07246v2#S3.F1 "Figure 1 ‣ 3.1 Propagandistic Memes ‣ 3 Dataset ‣ Propaganda to Hate: A Multimodal Analysis of Arabic Memes with Multi-Agent LLMs")所示。为了标注过程，我们制定了一套说明，下面将讨论这些说明。典型的标注方法通常涉及两到三名标注员。然而，对于本研究，我们依赖于一名具有类似标注经验的标注员。

#### 3.2.3 标注说明

该注释的目的是识别一个表情包是仇恨的还是非仇恨的。仇恨表情包可能会攻击不同的个人、组织或实体。因此，另一个任务是识别攻击类型。非仇恨表情包可能具有幽默或讽刺的性质。因此，目的是识别非仇恨表情包中的子类别。我们采用了先前工作中的注释定义和说明 [[23](https://arxiv.org/html/2409.07246v2#bib.bib23), [25](https://arxiv.org/html/2409.07246v2#bib.bib25)]。以下是我们的定义：

##### 仇恨：

针对个人进行的直接或间接攻击，通常基于种族、民族、国籍、移民身份、宗教、种姓、性别、性别认同、性取向、残疾或疾病等特征。我们将攻击定义为仇恨言论，表现为暴力、去人性化（如将个体与非人类实体如动物进行比较）、贬低或提倡排斥或隔离的言论。嘲笑仇恨犯罪也被归类为仇恨言论。然而，针对那些传播仇恨的群体（例如恐怖组织）的攻击不被视为仇恨言论。⁴⁴4[https://transparency.meta.com/en-gb/policies/community-standards/hate-speech/](https://transparency.meta.com/en-gb/policies/community-standards/hate-speech/)

细粒度类别的仇恨性：

+   •

    去人性化：明确或隐含地将某个群体描述为非人类。

+   •

    劣等：宣称某个群体低人一等、价值较低或不如整个社会或其他群体。

+   •

    煽动暴力：明确或隐含地提倡对某个群体实施伤害，包括身体暴力。

+   •

    嘲笑：开玩笑、讽刺、贬低或轻视某个群体。

+   •

    蔑视：对某个群体表达强烈的负面情绪或感觉。

+   •

    辱骂：使用带有偏见或贬义的词语来指代、描述或刻画某个群体。

+   •

    排斥：提倡、计划或为某个群体在整个社会或特定领域内的排斥或隔离辩护。

+   •

    其他：以上都不适用。

##### 非仇恨：

内容具有幽默感、中立或积极，且没有针对或伤害特定的个人或群体。它是轻松愉快的，旨在娱乐而不冒犯人。此外，内容不宣传或煽动暴力、仇恨或歧视。

细粒度的非仇恨类别：

+   •

    幽默：幽默的目的是为了娱乐、逗乐或给观众带来欢乐。通常以笑话、双关语或俏皮的语言为特征。幽默的风格差异很大，可以包括机智、滑稽、模仿和讽刺等。

+   •

    讽刺：通常是指说出与自己真正意思相反的话。讽刺是一种具有讽刺意味的表达形式，总是通过言语与意思之间故意的不匹配来进行，目的是嘲笑或讽刺特定的对象。

+   •

    其他：以上都不适用

#### 3.2.4 注释数据集

如第[3.2.1节](https://arxiv.org/html/2409.07246v2#S3.SS2.SSS1 "3.2.1 LLM Agents as Annotators ‣ 3.2 Hatefulness and Fine-grained Categories ‣ 3 Dataset ‣ Propaganda to Hate: A Multimodal Analysis of Arabic Memes with Multi-Agent LLMs")所讨论，我们使用GPT-4o、Sonnet（Claude 3.5）和Gemini（Vertex）将测试数据注释为“仇恨”和“非仇恨”类别。然后，我们将三个注释标签（从GPT-4o、Sonnet和Gemini获得）作为提示提供给GPT-4o，并要求它选择最匹配数据的标签。生成的输出标签被称为GPT-4o汇总。表格[1](https://arxiv.org/html/2409.07246v2#S3.T1 "Table 1 ‣ 3.2.4 Annotated Dataset ‣ 3.2 Hatefulness and Fine-grained Categories ‣ 3 Dataset ‣ Propaganda to Hate: A Multimodal Analysis of Arabic Memes with Multi-Agent LLMs")显示了注释者之间的一致性（IAA）。我们通过不同的设置计算了注释一致性，使用的是成对Cohen’s kappa得分：（i）LLMs作为注释者与LLM作为汇总者，（ii）LLMs作为注释者与人工注释，（iii）LLMs作为注释者之间的成对一致性。

它显示了GPT-4o和GPT-4o汇总之间的IAA很高（0.786），表明有显著的一致性。这是合理的，因为GPT-4o汇总是从GPT-4o派生出来的，这意味着汇总标签本质上与GPT-4o最初提供的标签紧密对齐。有趣的是，我们观察到Sonnet和GPT-4o汇总之间的IAA显著且很高（0.701），这表明Sonnet能够理解仇恨内容和表情包。

表[1](https://arxiv.org/html/2409.07246v2#S3.T1 "Table 1 ‣ 3.2.4 Annotated Dataset ‣ 3.2 Hatefulness and Fine-grained Categories ‣ 3 Dataset ‣ Propaganda to Hate: A Multimodal Analysis of Arabic Memes with Multi-Agent LLMs")显示，Sonnet与人工注释者之间的IAA得分（0.405）高于其他注释标签。我们还进行了LLM注释者之间的成对一致性测量，发现Sonnet与GPT-4o之间的一致性较高（0.528）。最后，我们测量了所有三个注释者之间的注释一致性，得到了0.369的得分。

三个不同的LLM和人工注释者之间的注释一致性表明，Sonnet在理解“仇恨”内容方面与人工注释者相比具有一定的能力。因此，我们使用Sonnet来注释训练和开发数据集。

表1：不同设置下的注释一致性。

| 注释者1 | 注释者2 | Kappa | 注释者1 | 注释者2 | Kappa |
| --- | --- | --- | --- | --- | --- |
| 一致性：LLMs与LLM作为汇总者 | 一致性：LLMs与人工 |
| --- | --- |
| GPT-4o | GPT-4o | 0.786 | GPT-4o | Human | 0.233 |
| Sonnet (Claude) | GPT-4o | 0.701 | Claude-3.5 (Sonnet) | Human | 0.405 |
| Gemini-1.5 (Vertex) | GPT-4o | 0.236 | Gemini-1.5 (Vertex) | Human | 0.300 |
| 一致性：大语言模型（成对） | GPO-4o 合并 | 人类 | 0.300 |
| Gemini-1.5 (Vertex) | Claude-3.5 (Sonnet) | 0.266 |  |  |  |
| GPT-4o | Gemini-1.5 (Vertex) | 0.142 |  |  |  |
| Claude-3.5 (Sonnet) | GPT-4o | 0.528 |  |  |  |

数据统计：仇恨表情包表 [2](https://arxiv.org/html/2409.07246v2#S3.T2 "表 2 ‣ 3.2.4 标注数据集 ‣ 3.2 仇恨与细粒度类别 ‣ 3 数据集 ‣ 从宣传到仇恨：使用多代理大语言模型对阿拉伯表情包进行的多模态分析") 提供了训练、开发和测试数据集的类别标签分布，分为“仇恨/非仇恨”并进一步标注为细粒度类别。与“仇恨”（N=212）类别相比，“非仇恨”（N=1931）类别的实例数量显著更多。在细粒度标签中，“嘲笑”在“仇恨”类别中占有显著份额（N=133）。类似地，在细粒度“非仇恨”类别中，“幽默”（N=1815）占主导地位，其次是“讽刺”。这一分布突显了数据的不平衡性。

表 2：标注数据的分布：训练和开发集使用 Sonnet 标注，而测试集由人工标注。

| 仇恨/非仇恨 | 仇恨：细粒度类别 |
| --- | --- |
| 标签 | 训练 | 开发 | 测试 | 标签 | 训练 | 开发 | 测试 |
| 仇恨 | 212 | 32 | 154 | 蔑视 | 38 | 7 | 25 |
| 非仇恨 | 1,931 | 280 | 452 | 去人性化 | 12 | 3 | 2 |
| 总计 | 2,143 | 312 | 606 | 嘲笑 | 133 | 19 | 49 |
| 非仇恨：细粒度类别 | 自卑 | 5 | 1 | 14 |
| 标签 | 训练 | 开发 | 测试 | 排除 | 6 | 7 | 3 |
| 讽刺 | 105 | 19 | 118 | 煽动暴力 | 13 | 2 | 12 |
| 幽默 | 1,815 | 260 | 334 | 侮辱 | 6 | 1 | 29 |
| 总计 | 1,920 | 279 | 452 | 其他 | 10 | 1 | 20 |
|  |  |  |  | 总计 | 223 | 41 | 154 |

宣传和仇恨表情包：为了理解宣传与仇恨表情包之间的关联，我们观察到，在测试集中有 171 个宣传表情包，其中 56 个是仇恨的（30%），70% 不是仇恨的。这是因为宣传表情包不一定总是煽动仇恨或伤害。

## 4 实验

我们的实验包括三个设置：(i) 仇恨与非仇恨，(ii) 仇恨表情包的细粒度类别，以及 (iii) 非仇恨表情包的细粒度类别。这些分类实验涉及单模态（文本和图像）和多模态分类。

文本分类：我们从宣传性迷因中提取了文本，并应用了多种文本分类技术，如AraBERT、mBERT、CAMelBERT和Qarib-BERT（[[6](https://arxiv.org/html/2409.07246v2#bib.bib6), [11](https://arxiv.org/html/2409.07246v2#bib.bib11), [1](https://arxiv.org/html/2409.07246v2#bib.bib1)]）。原始数据集存在不平衡问题，因此在微调过程中我们实施了类别加权方案。此外，我们通过调整丢弃率优化了模型。与仅使用原始数据集相比，这种方法显著提高了性能。我们嵌入了LoRa，以一种不需要微调所有模型参数的高效方式微调模型。然而，嵌入LoRA并未提升性能。接着，我们对AraBERT模型进行了微调。

图像分类：我们对ResNet50和ConvNeXt-tiny进行了微调[[24](https://arxiv.org/html/2409.07246v2#bib.bib24)]。为了确保稳定的权重调整，我们冻结了特征提取层，并对分类层进行了微调。我们还在模型训练过程中调整了丢弃率。

多模态分类：为了提取视觉和文本特征，我们分别应用了ConvNext tiny和AraBERT模型。我们通过融合层将特征进行结合。我们冻结了视觉模型，只用文本数据训练分类层。我们还使用了丢弃率来提高性能。

实验设置：我们在Nvidia-RTX 2080 GPU上执行了所有实验，并训练了我们的模型。我们使用了Adam优化器，文本和图像的初始学习率分别为1e-5和1e-4。我们使用了批量大小为32，序列长度为128。我们将文本数据的丢弃率设置为0.25，训练了50个周期。对于图像数据，我们将丢弃率设置为0.5，并训练了30个周期。对于多模态数据，我们使用0.2的随机丢弃率训练了100个周期。请注意，我们选择的模型和参数受到先前研究的启发[[3](https://arxiv.org/html/2409.07246v2#bib.bib3), [30](https://arxiv.org/html/2409.07246v2#bib.bib30)]。

表3：不同模态设置下粗粒度和细粒度仇恨标签在测试集上的分类结果。

| 模态 | 模型 | 准确率 | M-F1 | 模态 | 模型 | 准确率 | M-F1 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 仇恨与非仇恨 | 平衡数据集：仇恨与非仇恨 |
| 文本 | AraBERT | 0.819 | 0.705 | 平衡 | 融合 | 0.817 | 0.709 |
| 图像 | ConvNxT | 0.779 | 0.669 | 细粒度标签 |
| 文本+图像 | 融合 | 0.764 | 0.709 | 仇恨 | 融合 | 0.224 | 0.166 |
|  |  |  |  | 非仇恨 | 融合 | 0.622 | 0.537 |

## 5 结果与讨论

在表格[3](https://arxiv.org/html/2409.07246v2#S4.T3 "Table 3 ‣ 4 Experiments ‣ Propaganda to Hate: A Multimodal Analysis of Arabic Memes with Multi-Agent LLMs")中，我们展示了不同分类设置的结果。对于跨多种模态的仇恨与非仇恨分类，考虑到宏观F1分数，文本和多模态模型表现相似。对于多模态模型，我们获得了宏观F1分数为0.709，准确率为0.764。对于文本模态，我们获得了宏观F1分数为0.705，准确率为0.818。对于图像模态，我们获得了宏观F1分数为0.669，准确率为0.775。

为了理解类别不平衡问题，我们从数据集中选择了500个宣传标签和500个非宣传标签，并应用了融合模型。此模型得到了宏观F1分数为0.709，准确率为0.818。仇恨表情包进一步细分为八个标签，而非仇恨表情包则细分为两个标签，如表格[2](https://arxiv.org/html/2409.07246v2#S3.T2 "Table 2 ‣ 3.2.4 Annotated Dataset ‣ 3.2 Hatefulness and Fine-grained Categories ‣ 3 Dataset ‣ Propaganda to Hate: A Multimodal Analysis of Arabic Memes with Multi-Agent LLMs")所示。我们分别对仇恨和非仇恨细粒度标签应用了融合模型。仇恨细粒度表情包的F1分数为0.224，准确率为0.166，表现较差。这是因为仇恨类别中包含多个标签，且数据集在细粒度仇恨表情包方面不平衡。相比之下，由于非仇恨表情包只有两个标签，模型在分类上表现更好，达到了宏观F1分数0.537，准确率0.622。

## 6 结论与未来工作

在这项研究中，我们调查了宣传性表情包中的内容是否可能包含仇恨。为此，我们使用基于多代理LLM的方法，对宣传性表情包进行了粗略和细粒度的仇恨类别标签。我们观察到LLM代理（Claude 3.5 Sonnet）与人工标注之间存在中等程度的一致性。这促使我们对宣传性表情包进行了粗略和细粒度的仇恨类别标签。随后，我们使用该数据集训练模型，并在测试集上评估其性能。开发的数据集本身具有偏斜性，这也反映了其分类性能。值得注意的是，这一尝试可以以成本效益高的方式促使大规模数据集的开发。标签不平衡问题可以通过增加数据量来解决。未来的研究将通过增加数据量并探索开源LLM，进一步探讨这一方向。

## 7 致谢

F. Alam、M. R. Biswas、U. Shah、W. Zaghouani和G. Mikros的研究部分得到了卡塔尔国家研究基金（NPRP 14C-0916-210015）的资助，该基金是卡塔尔研究发展与创新委员会（QRDI）的一部分。

## 参考文献

+   [1] Abdelali, A., Hassan, S., Mubarak, H., Darwish, K., Samih, Y.: 在阿拉伯语推特上预训练BERT：实践考虑（2021年）

+   [2] Alam, F., Cresci, S., Chakraborty, T., Silvestri, F., Dimitrov, D., Martino, G.D.S., Shaar, S., Firooz, H., Nakov, P.: 多模态虚假信息检测调查。载于：第29届国际计算语言学大会论文集，pp. 6625–6643（2022年10月）

+   [3] Alam, F., Hasnat, A., Ahmed, F., Hasan, M.A., Hasanain, M.: ArMeme：阿拉伯语迷因中的宣传内容。载于：2024年自然语言处理经验方法会议论文集（2024年12月）

+   [4] Alam, F., Mubarak, H., Zaghouani, W., Da San Martino, G., Nakov, P.: WANLP 2022共享任务：阿拉伯语宣传检测概述。载于：第七届阿拉伯语自然语言处理研讨会论文集（2022年12月）

+   [5] Alam, F., Mubarak, H., Zaghouani, W., Da San Martino, G., Nakov, P.: WANLP 2022共享任务：阿拉伯语宣传检测概述。载于：第七届阿拉伯语自然语言处理研讨会论文集，pp. 108–115（2022年）

+   [6] Antoun, W., Baly, F., Hajj, H.: AraBERT：基于变换器的阿拉伯语言理解模型。载于：第四届开源阿拉伯语语料库与处理工具研讨会论文集。OSAC ’20，法国马赛（2020年）

+   [7] Chen, J., Wu, Y., Yang, X., Liu, J., Yang, H., Wang, Z.: 动态内容审查的多代理强化学习。载于：AAAI人工智能会议论文集，第35卷，pp. 4536–4544（2021年）

+   [8] Da San Martino, G., Barrón-Cedeño, A., Wachsmuth, H., Petrov, R., Nakov, P.: Semeval-2020任务11：新闻文章中的宣传技巧检测。载于：第14届国际语义评估研讨会论文集，pp. 1377–1414（2020年）

+   [9] Da San Martino, G., Yu, S., Barrón-Cedeño, A., Petrov, R., Nakov, P.: 新闻文章中宣传的精细分析。载于：2019年自然语言处理经验方法会议与第9届国际自然语言处理联合会议论文集，pp. 5636–5646，ACL（2019年11月）

+   [10] Davidson, T., Warmsley, D., Macy, M., Weber, I.: 自动化仇恨言论检测与攻击性语言问题。载于：国际AAAI网络与社交媒体会议论文集，AAAI ’17，第11卷（2017年）

+   [11] Devlin, J., Chang, M.W., Lee, K., Toutanova, K.: BERT：用于语言理解的深度双向变换器的预训练。载于：2019年北美计算语言学协会年会：人类语言技术会议论文集，pp. 4171–4186，ACL（2019年6月）

+   [12] Dimitrov, D., Alam, F., Hasanain, M., Hasnat, A., Silvestri, F., Nakov, P., Da San Martino, G.: SemEval-2024任务4：在迷因中多语言识别说服技巧。载于：第18届国际语义评估研讨会论文集，pp. 2009–2026，ACL，墨西哥城，墨西哥（2024年6月）

+   [13] Dimitrov, D., Ali, B.B., Shaar, S., Alam, F., Silvestri, F., Firooz, H., Nakov, P., Da San Martino, G.：在表情包中检测宣传技巧。载于：第59届计算语言学协会年会与第11届国际联合自然语言处理会议联合会议论文集。第6603–6617页。ACL-IJCNLP ’21（2021）

+   [14] Dimitrov, D., Bin Ali, B., Shaar, S., Alam, F., Silvestri, F., Firooz, H., Nakov, P., Da San Martino, G.：Semeval-2021任务6：文本和图像中说服技巧的检测。载于：第15届语义评估国际研讨会（SemEval-2021）论文集。第70–98页（2021）

+   [15] Fang, T., Xu, B., Zong, L.：情感增强的多模态说服技巧检测，使用分裂特征。载于：模糊系统与数据挖掘第八卷，第172–180页。IOS出版社（2022）

+   [16] Fortuna, P., Nunes, S.：关于自动检测文本中仇恨言论的调查。ACM计算调查（CSUR）51(4)，第1-30页（2018）

+   [17] Guo, T., Chen, X., Wang, Y., Chang, R., Pei, S., Chawla, N.V., Wiest, O., Zhang, X.：基于大语言模型的多代理：进展与挑战的调查。arXiv预印本arXiv:2402.01680（2024）

+   [18] Hasanain, M., Ahmed, F., Alam, F.：大语言模型在宣传片段标注中的应用。载于：2024年自然语言处理实证方法会议论文集（2024年12月）

+   [19] Hasanain, M., Ahmed, F., Alam, F.：GPT-4能否识别宣传？新闻文章中的宣传片段标注与检测。载于：2024年计算语言学联合国际会议、语言资源与评估会议论文集。LREC-COLING 2024，意大利都灵（2024）

+   [20] Hasanain, M., Alam, F., Mubarak, H., Abdaljalil, S., Zaghouani, W., Nakov, P., Da San Martino, G., Freihat, A.：ArAIEval共享任务：阿拉伯语文本中的说服技巧与虚假信息检测。载于：Sawaf, H., El-Beltagy, S., Zaghouani, W., Magdy, W., Abdelali, A., Tomeh, N., Abu Farha, I., Habash, N., Khalifa, S., Keleg, A., Haddad, H., Zitouni, I., Mrini, K., Almatham, R.（编辑）《ArabicNLP 2023论文集》。第483–493页。ACL，新加坡（混合会议）（2023年12月）

+   [21] Hasanain, M., Hasan, M.A., Ahmed, F., Suwaileh, R., Biswas, M.R., Zaghouani, W., Alam, F.：ArAIEval共享任务：单模态和多模态阿拉伯语内容中的宣传技巧检测。载于：第二届阿拉伯自然语言处理会议论文集。ACL，泰国曼谷（2024年8月）

+   [22] Hossain, E., Sharif, O., Hoque, M.M., Preum, S.M.：解密仇恨：识别仇恨表情包及其目标。arXiv预印本arXiv:2403.10829（2024）

+   [23] Kiela, D., Firooz, H., Mohan, A., Goswami, V., Singh, A., Ringshia, P., Testuggine, D.：仇恨表情包挑战：在多模态表情包中检测仇恨言论。载于：神经信息处理系统进展，第33卷，第2611–2624页（2020）

+   [24] Liu, Z., Mao, H., Wu, C.Y., Feichtenhofer, C., Darrell, T., Xie, S.: 面向2020年代的卷积网络。在：IEEE/CVF计算机视觉与模式识别会议论文集。第11976–11986页 (2022)

+   [25] Mathias, L., Nie, S., Mostafazadeh Davani, A., Kiela, D., Prabhakaran, V., Vidgen, B., Waseem, Z.: WOAH 5共享任务的有害迷因细粒度检测研究结果。在：Mostafazadeh Davani, A., Kiela, D., Lambert, M., Vidgen, B., Prabhakaran, V., Waseem, Z. (编)《第五届在线滥用与危害研讨会论文集》（WOAH 2021）。第201–206页，ACL，在线 (2021年8月)

+   [26] Mubarak, H., Abdaljalil, S., Nassar, A., Alam, F.: 在推文发布前检测并识别被删除推文的原因。《人工智能前沿》6, 1219767 (2023)

+   [27] OpenAI, R.: GPT-4技术报告。arXiv，第2303–08774页 (2023)

+   [28] Piskorski, J., Stefanovitch, N., Da San Martino, G., Nakov, P.: SemEval-2023任务3：在多语言环境中检测在线新闻中的类别、框架和说服技巧。在：第17届国际语义评估研讨会论文集。第2343–2361页，ACL，多伦多，加拿大 (2023年7月)

+   [29] Schmidt, A., Wiegand, M.: 使用自然语言处理检测仇恨言论的调查。在：第五届社交媒体自然语言处理国际研讨会论文集。第1–10页，ACL，西班牙瓦伦西亚 (2017年4月)

+   [30] Shah, U., Biswas, M.R., Agus, M., Househ, M., Zaghouani, W.: MemeMind在ArAIEval共享任务中的表现：通过先进的语言与视觉模型进行生成性增强和特征融合，检测阿拉伯语迷因中的宣传手段。在：第二届阿拉伯自然语言处理会议论文集。第467–472页 (2024年8月)

+   [31] Sharma, S., Alam, F., Akhtar, M.S., Dimitrov, D., Da San Martino, G., Firooz, H., Halevy, A., Silvestri, F., Nakov, P., Chakraborty, T.: 检测和理解有害迷因：一项调查。在：第31届国际人工智能联合会议论文集。第5597–5606页，IJCAI (2022年7月)

+   [32] Team, G., Anil, R., Borgeaud, S., Wu, Y., Alayrac, J.B., Yu, J., Soricut, R., Schalkwyk, J., Dai, A.M., Hauth, A., 等：Gemini：一系列功能强大的多模态模型。arXiv预印本 arXiv:2312.11805 (2023)
