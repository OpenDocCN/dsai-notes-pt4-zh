<!--yml

分类：未分类

日期：2025-01-11 11:43:03

-->

# OneKE：一个基于Docker的模式引导LLM代理知识提取系统

> 来源：[https://arxiv.org/html/2412.20005/](https://arxiv.org/html/2412.20005/)

罗宇杰 [luo.yj@zju.edu.cn](mailto:luo.yj@zju.edu.cn) 浙江大学ZJU-蚂蚁集团联合知识图谱研究中心杭州中国， 如向远， 刘康威 浙江大学ZJU-蚂蚁集团联合知识图谱研究中心杭州中国， 袁琳， 孙梦舒 ZJU-蚂蚁集团联合知识图谱研究中心蚂蚁集团杭州中国， 张宁宇 浙江大学ZJU-蚂蚁集团联合知识图谱研究中心杭州中国， 梁雷， 张志强 ZJU-蚂蚁集团联合知识图谱研究中心蚂蚁集团杭州中国， 周军 ZJU-蚂蚁集团联合知识图谱研究中心蚂蚁集团杭州中国， 魏兰宁， 郑达 ZJU-蚂蚁集团联合知识图谱研究中心蚂蚁集团杭州中国， 王浩芬 同济大学上海中国 和 陈华俊 浙江大学ZJU-蚂蚁集团联合知识图谱研究中心杭州中国（2018）

###### 摘要。

我们介绍了OneKE，一个基于Docker的模式引导知识提取系统，能够从Web和原始PDF书籍中提取知识，并支持各种领域（科学、新闻等）。具体来说，我们设计了具有多个代理和配置知识库的OneKE。不同的代理执行各自的角色，支持各种提取场景。配置知识库有助于模式配置、错误案例调试和修正，进一步提高了性能。在基准数据集上的实证评估展示了OneKE的有效性，而案例研究则进一步阐明了其对多种任务和多个领域的适应性，突出了其广泛应用的潜力。我们已开源代码¹¹1[https://github.com/zjunlp/OneKE](https://github.com/zjunlp/OneKE)并发布了视频²²2[http://oneke.openkg.cn/demo.mp4](http://oneke.openkg.cn/demo.mp4)。

信息提取；自然语言处理；大语言模型^†^†版权：acmlicensed^†^†期刊年份：2018^†^†doi：XXXXXXX.XXXXXXX^†^†会议：请确保从版权确认邮件中输入正确的会议名称；2018年6月3日至5日；纽约伍德斯托克^†^†isbn：978-1-4503-XXXX-X/18/06

## 1\. 介绍

知识提取——从数据中获取知识，是构建知识图谱（KG）（Chen等， [2022](https://arxiv.org/html/2412.20005v1#bib.bib2)）、检索增强（RAG）（Gao等， [2023](https://arxiv.org/html/2412.20005v1#bib.bib4)）以及领域特定应用，如科学发现（Dagdelen等， [2024](https://arxiv.org/html/2412.20005v1#bib.bib3)）和情报分析（Sun等， [2023](https://arxiv.org/html/2412.20005v1#bib.bib8)）等各种实际系统中的关键组成部分。过去几十年见证了各种知识提取系统的发展（Li等， [2023](https://arxiv.org/html/2412.20005v1#bib.bib6)；Wei等， [2024](https://arxiv.org/html/2412.20005v1#bib.bib10)；Xu等， [2024](https://arxiv.org/html/2412.20005v1#bib.bib11)）。特别是，随着大语言模型（LLM）的出现，新兴工作如InstructUIE（Shi等， [2024](https://arxiv.org/html/2412.20005v1#bib.bib7)）、iText2KG（Lairgi等， [2024](https://arxiv.org/html/2412.20005v1#bib.bib5)）和AgentRE（Wang等， [2023](https://arxiv.org/html/2412.20005v1#bib.bib9)）不断涌现。然而，之前的方法仍然难以有效地从遵循复杂模式的原始数据中提取信息，并且在调试和纠正错误时面临挑战。

![参见标题说明](img/f43800975c608b0c57df4bc2611eb8fb.png)

图1\. OneKE系统概述，支持多个领域（科学、新闻等）和数据类型（Web HTML、PDF等）。

请注意，之前的工作主要集中在单一模型的能力上，而忽视了设计一个全面的系统来解决知识提取任务。为此，我们引入了OneKE，一个基于Docker的模式引导知识提取系统。我们采用多代理设计和配置的知识库，为各种场景提供知识提取支持并进行错误调试，旨在尽可能满足用户的实际需求。根据（Shi等， [2024](https://arxiv.org/html/2412.20005v1#bib.bib7)）的方法，我们设计了三个代理：Schema Agent用于分析各种数据类型的模式，Extraction Agent用于利用不同的LLM提取知识，Reflection Agent用于调试和处理错误案例。基于这一设计，OneKE能够高效处理不同长度和格式的源文本，如HTML和PDF，并展示出适应不同任务配置的强大能力，能够生成针对特定需求量身定制的全面输出模式。

我们使用两个基准数据集评估OneKE的效果，分别用于命名实体识别（NER）和关系抽取（RE），展示了我们框架的有效性。此外，为了探索OneKE在实际应用中的多样性，我们对具体的抽取任务进行了案例分析。这些场景包括从网页新闻文章和原始PDF书籍章节中提取结构化信息，突出显示了OneKE在有效处理多种数据格式和不同任务背景中的能力。这个灵活的框架无需微调，就能快速适应未来的LLM，从而增强它们的能力并提升其整体效能。

## 2\. 设计与实现

OneKE经过深思熟虑的设计，旨在解决知识抽取中固有的复杂性和挑战。如图[1](https://arxiv.org/html/2412.20005v1#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ OneKE: A Dockerized Schema-Guided LLM Agent-based Knowledge Extraction System")所示，框架的设计遵循了若干关键考虑因素，这些因素提升了其在现实场景中的功能性和适应性：

(1) 适应现实世界数据。现实世界的信息抽取任务通常处理的是原始数据，如HTML、PDF等。因此，OneKE框架支持多种数据类型，而不仅限于纯文本。我们还预留了一个用户定义的接口，以便未来支持新的数据类型。

(2) 对复杂模式的泛化能力。实际的知识抽取场景应能够处理多样化且复杂的模式，甚至没有模式。因此，我们设计了专门针对OneKE的模式代理，支持预定义模式和使用LLM进行自我模式推导。OneKE还支持多种LLM，包括LLaMA、Qwen和ChatGLM，以及GPT-4等专有模型，从而实现有效的知识抽取。

(3) 调试和修复错误。大多数先前的工作在遇到错误情况时需要重新训练模型。相比之下，我们将案例库集成到OneKE中，为模型提供反思和错误修正功能，使其在知识抽取任务中能够不断改进。

### 2.1\. 模式代理

为了支持各种任务设置和数据类型，我们基于大语言模型（LLMs）开发了模式代理（Schema Agent），用于为每个任务生成相应的输出模式。主要目标是对数据进行预处理，标准化其格式和模式，并为后续的信息提取步骤做好准备。为了支持各种现实世界的数据，我们利用Langchain提供的`document_loaders`模块来预处理数据并对长文本进行分块。用户还可以定义新的数据类型，并添加自定义的预处理方法。接下来，如果用户已经定义了模式，给定任务描述和原始数据输入，模式代理将从配置知识库中的模式仓库中选择合适的预定义模式。如果用户没有提供模式，模型将根据用户的指示生成统一的输出模式，例如“*提取角色和背景设定*”。用户可以通过更新配置知识库中的模式仓库，使用简单的文本来自定义模式。

### 2.2\. 提取代理

在接收到来自模式代理的统一输出模式后，我们设计了提取代理，利用LLMs提取知识，从而生成初步的提取结果。具体来说，该模块支持多种模型，包括本地部署的开源模型，如LLaMA、Qwen和ChatGLM，以及API服务，如OpenAI和DeepSeek。为了提高性能，提取代理从相似案例中学习，并将这些知识应用到提取过程中。相关案例通过语义相似度和字符串匹配（通过all-MiniLM-L6-v2模型和FuzzyWuzzy包，我们将默认设置为Top两项）从案例库中检索。然后，这些案例作为少量示例被并入原始上下文中，形成提示（prompt），之后调用LLM获得提取结果。在完成上述步骤后，我们获得了初步的提取结果；然而，通常会发生错误。为了解决这个问题，我们设计了反思代理来调试并修复这些错误。我们使用自一致性（self-consistency）来筛选出模型不确定的案例，并将这些案例传递给反思代理。

### 2.3\. 反思代理

为了启用调试和错误修正，帮助 OneKE 从过去的错误中学习，我们遵循了 (Shi 等， [2024](https://arxiv.org/html/2412.20005v1#bib.bib7)) 设计了 Reflection Agent 来促进反思和优化。通过利用先前的知识，Reflection Agent 对前一模块的初步输出进行细化和改进，最终产生最终的提取结果。具体而言，Agent 利用外部 Case Repository，特别是相关的坏案例，来促进反思和修正。类似于在 [2.2](https://arxiv.org/html/2412.20005v1#S2.SS2 "2.2\. Extraction Agent ‣ 2\. Design and Implementation ‣ OneKE: A Dockerized Schema-Guided LLM Agent-based Knowledge Extraction System") 小节中讨论的检索方法，Reflection Agent 获取与当前任务和提取文本最相关的坏案例。这些相关的坏案例及其反思分析将被整合到 LLM 中。通过这种方式，Agent 可以有效地从过去的错误中学习，从而提高其错误修正能力，生成准确的答案。

### 2.4\. Configure Knowledge Base

Configure Knowledge Base 提供了三大 Agent 所需的基本信息，包括手动定义的各种任务模式和历史提取案例，从而提升了性能和错误修正能力：

Schema Repository。在 OneKE 系统中，Schema Repository 提供了预定义的模式供 Schema Agent 使用，支持后续的提取过程。具体来说，Schema Repository 包含了针对 NER、RE 和 EE 任务的预定义输出模式，以及用于各种数据场景（如科学学术论文和新闻报道）的模板。Schema Repository 中的模式以 Pydantic 对象的形式进行结构化，能够无缝地序列化为 JSON 格式供 Extraction Agent 使用。此外，这种结构还允许用户在库中自定义新的模式，从而增强系统的适应性和扩展性。

案例库。为了支持OneKE的调试和错误修正，我们设计了案例库，主要存储过去知识提取案例的痕迹。这个库支持提取代理（Extraction Agent）执行提取，并帮助反思代理（Reflection Agent）进行反思和修正错误。具体来说，存储在案例库中的知识提取案例可以分为两类：正确案例和错误案例。正确案例为提取代理提供成功提取的推理步骤，而错误案例则为反思代理提供可避免错误的警告。每当知识提取任务完成后，案例库会自动更新。具体来说，该模块首先生成从正确答案得出的推理步骤，将正确答案及其推理步骤存储在正确案例库中，以增强任务理解。此外，代理会将其答案与正确答案进行比较，并反思其原始回答，以识别潜在问题。然后，它将原始答案及其相应的反思存储在错误案例库中以供日后参考。

![参见标题](img/f79f1a93ed5dc54f05f28336860b5141.png)![参见标题](img/ca60d0727918915687304a50e781d7e6.png)

图2. OneKE中不同组件的性能。

## 3. 评估

![参见标题](img/644e37c440ba548294fd50c654afd55d.png)

图3. 在Web新闻提取和书籍知识提取中使用OneKE。

实验设置。我们在CrossNER和NYT-11-HRL数据集上评估OneKE。CrossNER是一个跨领域命名实体识别（NER）数据集，而NYT-11-HRL专注于新闻领域的关系抽取（RE）任务。OneKE的性能在LLaMA-3-8B-Instruct和GPT-4-turbo模型上进行评估。

主要结果。如图[2](https://arxiv.org/html/2412.20005v1#S2.F2 "Figure 2 ‣ 2.4\. Configure Knowledge Base ‣ 2\. Design and Implementation ‣ OneKE: A Dockerized Schema-Guided LLM Agent-based Knowledge Extraction System")所示，OneKE中采用的多种方法在命名实体识别（NER）和关系抽取（RE）任务中都带来了性能提升。值得注意的是，提取代理的案例检索方法（Case Retrieval）取得了最大的改进。通过分析，我们观察到代理有效地应用了提供样本中的推理路径，从而促进了准确的提取。此外，通过比较两项不同的任务，我们发现上述案例检索方法对于更具挑战性的关系抽取任务更为有效，因为在此类复杂场景中，中间的推理步骤至关重要。案例反思（Case Reflection）主要强调模型识别已知错误的能力，以及它在处理这些错误时的迁移学习能力，从而在两个任务中都带来了类似的改进。我们将在下一节提供具体应用场景的案例研究。

## 4. 应用

在实际应用中，OneKE框架支持多种数据格式（HTML、PDF、Word），可适应短文本和长文本的不同需求，便于无缝集成到各种下游应用中。我们在以下两个典型的提取场景中提供了案例分析。

网络新闻提取。在新闻领域，OneKE增强了网络新闻内容的知识提取，从而促进了下游任务的开展，如有效的情感监控、主动风险管理，以及各种附加应用。如图[3](https://arxiv.org/html/2412.20005v1#S3.F3 "Figure 3 ‣ 3\. Evaluation ‣ OneKE: A Dockerized Schema-Guided LLM Agent-based Knowledge Extraction System")所示，提取任务从从随机选择的网页HTML页面中提取新闻文章的关键信息开始，目的是识别新闻的总体性质并获取结构化的关键信息。在解析HTML格式的文本后，模式代理（Schema Agent）首先识别其领域和类型为政治新闻报道，从而提供了重要的指导。利用这一元数据，模式代理生成了一个结构化的输出模式（Output Schema），以代码格式有效地捕捉新闻的关键信息。一旦输出模式被序列化为JSON格式的描述，提取代理和反思代理将协作进行后续的提取和反思优化任务。这一代理协作的成果最终生成了一个JSON输出，捕捉了新闻报道的关键信息和结构。

书籍知识提取。OneKE的另一个应用是在大量语料库中提取结构化知识，包括书籍、文档或手册。具体而言，我们使用《哈利·波特》系列的第一章（PDF格式，共17页）作为目标文本。提取的重点是这一章节中的主要人物和背景设定。如图[3](https://arxiv.org/html/2412.20005v1#S3.F3 "Figure 3 ‣ 3\. Evaluation ‣ OneKE: A Dockerized Schema-Guided LLM Agent-based Knowledge Extraction System")所示，生成的输出模式准确地识别了两个目标提取对象：“main_characters”和“background_setting”。随后，在提取代理（Extraction Agent）和反思代理（Reflection Agent）的协作下，OneKE成功提取了相关信息。

## 5\. 结论与未来工作

在本文中，我们介绍了 OneKE，一个基于 Docker 的模式引导 LLM 代理知识提取系统。OneKE 旨在灵活地应用于现实场景中的各种提取任务。它能够处理不同长度和格式的源文本（如 HTML 和 PDF），同时展示了适应不同任务配置的能力，生成广泛的输出模式，满足特定需求。此外，集成自我反思机制使得 OneKE 能够通过外部反馈进行迭代改进，从而提高准确性和适应性。

长期维护。我们将长期维护 OneKE，添加新功能并修复错误。OneKE 基于模块化架构，具有良好的扩展性，我们计划通过整合来自科学等领域的额外领域特定知识，扩展知识库，从而扩大其在多个领域的适用性。为了推进文档数据提取和理解，我们计划开发多种图表类型和内容的集成与分析方法。我们希望 OneKE 能成为从事基于 LLM 的知识提取的研究人员和工程师的有用工具。

## 参考文献

+   （1）

+   Chen 等人（2022）陈向、张宁宇、谢鑫、邓书敏、姚云志、谭川琪、黄飞、司罗和陈华军。2022。Knowprompt：一种知识感知的关系提取提示调优方法，具有协同优化功能。在 *WWW* 会议上发表。

+   Dagdelen 等人（2024）John Dagdelen、Alexander Dunn、Sanghoon Lee、Nicholas Walker、Andrew S Rosen、Gerbrand Ceder、Kristin A Persson 和 Anubhav Jain。2024。利用大型语言模型从科学文本中提取结构化信息。*自然通讯* 15（2024），1，1418。

+   Gao 等人（2023）高云凡、熊云、高欣瑜、贾康翔、潘锦柳、毕宇熙、戴一、孙家伟、王浩芬。2023。针对大型语言模型的检索增强生成：一项综述。*arXiv 预印本 arXiv:2312.10997*（2023）。

+   Lairgi 等人（2024）Yassir Lairgi、Ludovic Moncla、Rémy Cazabet、Khalid Benabdeslem 和 Pierre Cléau。2024。iText2KG：基于大型语言模型的增量知识图谱构建。*CoRR* abs/2409.03284（2024）。arXiv:2409.03284

+   Li 等人（2023）李博、方格翔、杨阳、王全森、叶威、赵文、张士坤。2023。评估 ChatGPT 的信息提取能力：对性能、可解释性、校准性和忠实度的评估。*arXiv 预印本 arXiv:2304.11633*（2023）。

+   Shi 等人（2024）史宇晨、姜国超、邱天、杨德庆。2024。AgentRE：一个基于代理的框架，用于在关系提取中导航复杂的信息景观。发表于 *第33届 ACM 国际信息与知识管理会议，CIKM 2024*。ACM。

+   Sun et al. (2023) Nan Sun, Ming Ding, Jiaojiao Jiang, Weikang Xu, Xiaoxing Mo, Yonghang Tai, and Jun Zhang. 2023. 网络威胁情报挖掘用于主动网络安全防御：一项调查与新视角。*IEEE通讯调查与教程* 25, 3 (2023), 1748–1774。

+   Wang et al. (2023) Xiao Wang, Weikang Zhou, Can Zu, 等人。2023. InstructUIE：统一信息提取的多任务指令调优。*CoRR* abs/2304.08085 (2023)。arXiv:2304.08085

+   Wei et al. (2024) Xiang Wei, Xingyu Cui, Ning Cheng, Xiaobin Wang, Xin Zhang, Shen Huang, Pengjun Xie, Jinan Xu, Yufeng Chen, Meishan Zhang, 等人。2024. Chatie：通过与ChatGPT对话进行零-shot信息提取。*arXiv预印本arXiv:2302.10205* (2024)。

+   Xu et al. (2024) Derong Xu, Wei Chen, Wenjun Peng, Chao Zhang, Tong Xu, Xiangyu Zhao, Xian Wu, Yefeng Zheng, Yang Wang, and Enhong Chen. 2024. 大型语言模型用于生成性信息提取：一项调查。*计算机科学前沿* 18, 6 (2024), 186357。
