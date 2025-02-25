<!--yml

类别：未分类

日期：2025-01-11 12:13:12

-->

# 自动化测试生成用于评估工具增强型LLM作为对话式人工智能代理

> 来源：[https://arxiv.org/html/2409.15934/](https://arxiv.org/html/2409.15934/)

Samuel Arcadinho

samuel.arcadinho@zendesk.com

&David Aparício

david.aparicio@zendesk.com

\ANDMariana S. C. Almeida

mariana.almeida@zendesk.com

###### 摘要

工具增强型LLM是创建能够进行真实对话、遵循流程并调用适当功能的人工智能代理的有前景的方法。然而，由于可能的对话种类繁多，评估它们具有挑战性，现有的数据集仅专注于单次交互和功能调用。我们提出了一个测试生成管道，用于评估LLM作为对话式人工智能代理的表现。我们的框架利用LLM生成基于用户定义流程的多样化测试。为此，我们使用中间图来限制LLM测试生成器产生不基于输入流程的幻觉内容，并强制执行对可能对话的高覆盖率。此外，我们提出了ALMITA，这是一个手工策划的数据集，用于评估客户支持中的人工智能代理，并利用该数据集评估现有的LLM。我们的结果表明，尽管工具增强型LLM在单次交互中表现良好，但在处理完整对话时常常遇到困难。尽管我们聚焦于客户支持，但我们的方法是通用的，并适用于不同领域的人工智能代理。

## 1 引言

大型语言模型（LLM）正在革新人类人工智能代理，并展示了跨多个领域的显著泛化能力 wu2023autogen；lan2024teachers；li2024personal。特别是，LLM在作为聊天机器人和客户支持系统中的人工智能代理方面产生了深远的影响 dam2024complete；katragadda2024leveraging。

然而，草率地将大型语言模型（LLM）作为人工智能代理部署，并允许其与真实用户和API进行交互，可能会导致错误信息、声誉损害和公司成本。因此，事先评估人工智能代理至关重要。尽管存在这种需求，但在实际场景中评估LLM的性能仍然是一个重大挑战。尤其是在对话场景中，这比单一交互请求的回答更为复杂。当前大多数评估LLM的方法主要集中在特定任务上，例如多重问答 zhuang2024toolqa；kamalloo2024towards 或代码生成 liu2024your；liu2024exploring，这些方法并没有全面评估LLM作为有效对话式人工智能代理所应具备的更广泛能力。

专注于客户支持，一个有效的 AI 代理应该能够与工具和客户进行交互，以解决客户问题，同时严格遵循客户支持管理员描述的程序。为了评估 AI 代理的表现，关键在于衡量其遵循给定程序的能力以及其对潜在客户操控的抗压性。为此，拥有一个全面的评估数据集至关重要，它能提供关于代理能力和局限性的有价值见解。

![参见说明](img/e6cb03c0520866c3f7926030c3960526.png)

图1：自动化测试生成管道。对于给定的意图（例如，取消订单）（1）我们使用大型语言模型生成相应的程序。然后，（2）大型语言模型从程序中提取相关的 API，并且（3）生成程序及其 API 的流程图。接下来，（4）大型语言模型从流程图生成对话图，并且（5）向图中添加噪声（例如，用户偏离预期程序），以使图更具现实性。为了从图中获得对话，（6）我们从中抽取路径，这些路径对应不同的交互。最后，（7）大型语言模型从路径生成对话，并且（8）我们从采样的对话中提取测试。

我们提出了一种生成评估数据集的方法，用于评估作为对话式 AI 代理的工具增强型大型语言模型。我们的方法使用大型语言模型自动生成数据集，通过基于程序生成对话，之后将其转化为测试。我们使用中间图结构来提高生成数据集的质量（即，测试遵循用户定义的程序）并使其更加全面（即，测试涵盖大多数相关案例）。为了评估 AI 代理处理攻击的能力，我们在示例中加入了红队测试。

我们的生成管道，如[图1](https://arxiv.org/html/2409.15934v2#S1.F1 "在 1 引言 ‣ 自动化测试生成用于评估工具增强的大型语言模型作为对话式 AI 代理")所示，通过使用合成生成的意图作为程序的种子，自动构建多样化的数据集。此外，我们的管道还允许在可用的情况下加入真实数据，例如公司使用的实际程序或 API，以生成合成对话。虽然数据集可以完全自动创建，但我们还提出了ALMITA（自动化基准测试语言模型用于智能工具增强代理），这是一个手动策划的数据集。我们使用这个高质量的数据集来基准测试大型语言模型作为对话式工具增强 AI 代理。

我们的主要贡献包括：

+   •

    一种生成数据集的方法，用于评估作为 AI 对话代理的工具增强型大型语言模型，减少了获取此类数据集所需的人工努力。我们的方法提供了 AI 代理的全面评估，包括现实且多样的对话、工具的使用（例如，函数/API），并且基于用户定义的程序。

+   •

    ALMITA是第一个可用于评估客户支持AI代理的对话数据集，包含工具（即函数）和对话回复，以遵循公司用户定义的程序。ALMITA包含1420个合成测试，这些测试经过人工精心策划，以确保高质量的样本¹¹1ALMITA，以及使用我们的管道生成的所有其他数据集和本文中引用的所有数据集，都可以在[https://github.com/zendesk/almita-dataset](https://github.com/zendesk/almita-dataset)中获得。

+   •

    对提出的数据集进行了多种LLM的基准测试。我们的结果表明，当前的LLM在单条消息准确性和调用正确函数方面表现良好，但在考虑整个对话时，准确性较低，这可能表明如果将其部署为完全自动化的客户支持AI代理，它们可能不会成功。

我们还注意到，虽然我们的评估重点是客户支持，但相同的方法可以应用于其他领域，只需进行一些调整。

## 2 相关工作

随着LLM作为AI代理的使用越来越广泛，已经做出了重大努力，开发出用于评估它们在对话环境中正确回答客户请求的基准。GAIA提出了466个由人工标注的问题，涵盖了如一般知识、日常任务和数据分析等任务Mialon2023GAIAAB。最近，AgentInstruct介绍了一个框架，通过从多种来源（如代码、网页文章和教科书章节）生成合成数据，帮助代理生成和完善指令集Mitra2024AgentInstructTG。与我们的工作不同，这些数据集并未评估工具增强的AI代理。

已经提出了用于评估工具增强LLM的数据集。Zeng2023AgentTuningEG提出了AgentTuning，并汇编了多个代理数据集来创建API调用的序列。AgentBench特征展示了代理与环境之间的多步骤交互，使用各种工具来解决用户请求Liu2023AgentBenchEL。patil2023gorilla和qin2023toolllm构建了来自TorchHub、TensorHub和rapidAI等来源的API数据集，提示LLM生成可由这些API解决的指令。basu2024api将多个数据集合并，将用户指令转换为API调用。APIGen引入了一种自动化方法来生成用于工具函数调用的合成数据集liu2024apigen。与我们的工作不同，这些数据集并非对话型的，仅关注将话语映射到API调用，并且没有使用中间结构（即图）来确保覆盖并减少生成测试中的幻觉。

其他相关工作聚焦于图学习，并利用不同的中间结构来减少幻觉。ye2023natural 提出了 InstructGLM，使用自然语言描述节点特征，用于调优 LLM 以进行图上的推理。wang2024can 引入了 NLGraph，这是一个基于自然语言编写的图形问题基准，展示了 LLM 可以对图的文本描述执行结构化操作。此外，narayan2023conditional 提出了使用问答蓝图作为中间表示来减少幻觉。这些工作并没有完全涵盖我们的问题设置——生成对话格式的对话、调用 API 和提取测试。

![参见说明](img/4900855e1644e154eaab8a895a1a6d6c.png)

图 2：*订单未收到*意图的流程图和程序*“如果客户没有收到他们的订单，允许客户在提供正确的订单 ID 的前提下取消或退款”*。蓝色节点为消息节点，黑色节点为 API 调用节点，橙色节点为终止节点。边标签为用户消息或 API 输出。

## 3 方法

我们的自动化测试生成流水线，如[图 1](https://arxiv.org/html/2409.15934v2#S1.F1 "1 引言 ‣ 用于评估工具增强型 LLM 作为对话 AI 代理的自动化测试生成")所示，首先通过输入意图生成文本程序。虽然可以使用 LLM 直接从程序生成对话，但我们的方法是将程序转化为流程图，再转化为对话图。我们的假设是，使用这些中间结构化表示使得基于程序生成对话的任务更加准确；请参见[4.2节](https://arxiv.org/html/2409.15934v2#S4.SS2 "4.2 消融研究：程序生成对话 ‣ 4 结果 ‣ 用于评估工具增强型 LLM 作为对话 AI 代理的自动化测试生成")以获取支持证据。此外，这些图允许我们为对话引入噪声，使对话更加真实且具有挑战性，并且能够对路径进行采样，确保路径覆盖和对话多样性。然后，我们从采样路径中生成对话。最后，我们通过在每个用户消息处分解对话、存储上下文并记录生成的响应作为正确回复，来从这些对话中提取测试。

### 3.1 意图生成器

意图（或*问题*，例如取消订单）作为我们自动化测试生成方法的种子。意图可以通过 LLM 生成（如本工作所示），也可以从预定义的领域特定意图中获取，或是两者的结合。用于生成意图的提示在[Section A.1](https://arxiv.org/html/2409.15934v2#A1.SS1 "A.1 意图生成 ‣ 附录 A 提示 ‣ 用于评估工具增强型 LLM 作为对话 AI 代理的自动化测试生成")中展示。

### 3.2 程序生成器

一个流程描述了代理应如何解决给定的问题/意图。我们使用 LLM 为每个输入的意图生成一个流程，要求它提供一系列帮助代理完成给定任务的指令。我们在提示中要求避免输出一般性的陈述（例如，"取消政策可能依赖于公司" 或 "解释公司的政策"），因为我们的目标是生成具体且明确的流程，步骤应精确且细化。我们还要求条件语句是可能的，但它们需要在流程的步骤中有明确的解决方案。最后，步骤中可能包含基于 API 的操作（例如，搜索数据库、升级问题），但不能是浏览操作（例如，点击登录页面）。完整的提示请参见[第 A.2 节](https://arxiv.org/html/2409.15934v2#A1.SS2 "A.2 Procedure generation ‣ Appendix A Prompts ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")。与我们为意图描述的情况类似，现有的流程（例如，公司的流程）可以作为我们方法的输入。此外，流程可以基于现有的知识生成，即现有的工单或帮助中心文章。

考虑意图 "订单未收到"：一个简单的流程可以是 "如果客户没有收到订单，允许客户在提供正确的订单 ID 后取消或退款"。我们在本文中将此流程作为示例（见[图 2](https://arxiv.org/html/2409.15934v2#S2.F2 "In 2 Related work ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")， [3](https://arxiv.org/html/2409.15934v2#S3.F3 "Figure 3 ‣ 3.5 Conversation graph generator ‣ 3 Method ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents") 和 [4](https://arxiv.org/html/2409.15934v2#S3.F4 "Figure 4 ‣ 3.6 Noise generator ‣ 3 Method ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")）。

### 3.3 API 提取器

我们的目标使用案例是工具增强的 AI 代理。我们使用 LLM 生成对于输入流程有用的 API。我们在提示中要求提取的 API 是代理 API，而不是面向客户的 API。生成的 API 包括不仅是 API 名称，还有它们的输入输出参数，以及简短的描述。完整的提示请参见[第 A.3 节](https://arxiv.org/html/2409.15934v2#A1.SS3 "A.3 API extraction ‣ Appendix A Prompts ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")。这些 API 应该由代理显式调用，以完成流程。与意图和流程类似，现有的 API 也可以轻松地包含到我们的流程中。

### 3.4 流程图生成器

流程图生成器接收一个过程和相关的API作为输入，并生成一个从代理人角度封装过程逻辑的有向图：节点是代理人行动，边是客户回复或API输出。节点有4种类型：（i）单一的start_message节点是代理人发送给客户的初始消息，（ii）message节点是代理人发送给客户的额外消息，（iii）api节点是代理人执行的API调用，（iv）end_message节点是代理人结束互动的消息。为了减少幻觉并增加完整性，我们在提示中强制要求（[Section A.4](https://arxiv.org/html/2409.15934v2#A1.SS4 "A.4 Flowgraph generation ‣ Appendix A Prompts ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")）每个过程的细节必须出现在message节点中。

[流程图示例](https://arxiv.org/html/2409.15934v2#S2.F2 "In 2 Related work ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")如[图2](https://arxiv.org/html/2409.15934v2#S2.F2 "In 2 Related work ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")所示。流程图中的节点有一个node_id（例如，“N1”），一个node_type（上面描述的四种类型之一），以及一个node_description，应该与过程中的某个步骤相关（例如，“告诉用户未找到订单”）或一个API调用（例如，“refund_order”）。图中的边可以是用户互动（例如，“提供订单ID和电子邮件”）或API调用的结果（例如，“找到订单”）。流程图中的边有一个edge_id（例如，“E1”），一个由源节点和目标节点组成的元组（例如，“(N1, N2)”），以及一个边描述，如前所述。我们进行一次性提示，向LLM提供一个示例；因此，可以在[Section A.4](https://arxiv.org/html/2409.15934v2#A1.SS4 "A.4 Flowgraph generation ‣ Appendix A Prompts ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")中的流程图提示中看到完整的流程图。

为了尽量保证正确的流程图，我们指示LLM生成仅有一个根节点（类型为start_message）的图，始终在节点和边的描述中包含具体的消息，并在api节点的出边中提供API输出。为了尽量限制幻觉并确保图能够封装整个过程，我们指示LLM严格遵循过程中的内容并包括其中的所有内容。在生成步骤的最后，我们将图转换为networkx图，如果解析成功，我们务实地验证是否遵循了之前描述的所有规则；如果未遵循，我们将丢弃生成的流程图。

### 3.5 会话图生成器

流程图表示为完成一个过程而由代理执行的一系列步骤。流程图的结构并不直接映射到对话，这可能使得从流程图创建对话的任务变得困难。因此，对话图生成器的目标是将流程图转换为对话图，这是一种更类似于对话的结构。生成的对话图是一个有向图，预计会有三种不同类型的节点：(i) 代理节点是代理发送的消息，(ii) 客户节点是客户发送的消息，(iii) API节点是代理的API调用。

![参见说明](img/f8b5ddf790d8621a35ef9be87e9f6220.png)

图3：来自图[2](https://arxiv.org/html/2409.15934v2#S2.F2 "图2 ‣ 2相关工作 ‣ 用于评估工具增强型LLM作为对话AI代理的自动化测试生成")的流程图的对话图，表示意图*未收到订单*。蓝色节点为代理节点，绿色为用户节点，黑色为API节点。

对话图的一个示例见于[图3](https://arxiv.org/html/2409.15934v2#S3.F3 "在3.5节对话图生成器 ‣ 3方法 ‣ 用于评估工具增强型LLM作为对话AI代理的自动化测试生成")。对话图中的节点具有node_id（例如，“N1”）、node_type（上述三种类型之一）和node_description，node_description是代理节点和客户节点的消息，以及API节点的API调用。对话图中的边连接连续的消息/API调用。有些对话路径有条件，例如API调用返回订单是否找到；在这些情况下，边会有边描述，否则边描述为空。流程图中的边具有edge_id（例如，“E1”）、一个包含源节点和目标节点的元组（例如，“(N1, N2)”）以及边描述。

为了减轻错误对话图的影响，我们为LLM提供了额外的图构建规则，例如，客户节点应后跟代理节点或API节点，叶节点应为助手节点。我们通过一次性提示向LLM提供一个流程图示例及其对应的对话图，如[附录A.5](https://arxiv.org/html/2409.15934v2#A1.SS5 "A.5 对话图生成 ‣ 附录A 提示 ‣ 用于评估工具增强型LLM作为对话AI代理的自动化测试生成")所示。与流程图类似，我们将生成的图加载到networkx中，并验证是否满足所需的条件，否则该图将被丢弃。

### 3.6 噪声生成器

对话图是从代理的操作流程构建的，因此它们预计仅包含代理和客户的*良好*行为（即，正常路径）。为了使AI代理在面对客户的意外行为时更具韧性（无论是恶意行为还是非恶意行为），我们在对话图中加入了流程之外的行为。

噪声生成器遍历对话图中的代理节点，并以一定的概率（例如，20%）插入一条指向新客户节点的边，该边带有`node_description`消息，消息内容可以是“程序外”消息或“攻击”消息。这些消息是由大型语言模型（LLM）事先生成的。此外，我们还从噪声客户节点添加一条边，指向一个新的代理节点，`node_description`为“说你只是来帮助解决原始问题的。”

![参考标题](img/9e52187113df69fdb6838d45b2eec735.png)

图4：从一次对话中提取的测试。

### 3.7 路径采样器

我们通过从对话图中采样路径来提取客户和代理之间的对话。给定一个包含$N$个节点的对话图$\mathcal{G}$和所需的对话数$M$，我们使用加权随机游走算法来采样路径，[算法 1](https://arxiv.org/html/2409.15934v2#alg1 "在3.7路径采样器 ‣ 3 方法 ‣ 自动化测试生成以评估工具增强的LLM作为对话AI代理")，它是原始随机游走算法的增强版本，旨在提高节点覆盖率。为此，我们使用一个包含$N$个元素的加权向量$\bf{w}$，初始化为1（第3行）。每条路径$p$通过反复采样节点来构建，使用`sample_node`（第7行）。一个节点$n$，是当前路径$p$中最后一个节点的子节点，按与其权重$w_{i}$成反比的概率进行采样，其中$w_{i}$是节点$n$被访问的次数加1（第9行）。节点$n$在图$\mathcal{G}$中的索引$i$由`node_index`提供（第8行）。路径构建在达到叶节点时终止（第11–13行）。

算法 1 对话路径采样

1: 输入：$\mathcal{G}$，$M$  

### 3.8 对话生成器

对话生成器通过输入的对话图、采样路径和相关API创建合成对话。我们为大型语言模型（LLM）提供关于对话图结构和API的上下文信息。通过一次性提示，我们向LLM展示一个包含对话图、API列表、采样路径的三元组示例，以及基于这些条件的可能对话（请参见[附录A.6](https://arxiv.org/html/2409.15934v2#A1.SS6 "A.6 对话生成 ‣ 附录A 提示 ‣ 自动化测试生成用于评估工具增强的LLM作为对话AI代理")）。为了生成有效的对话，我们在提示中包含条件，如始终生成带有API输出的消息，API消息后跟随，交替进行客户和助手的消息，确保代理根据API输出消息进行操作，并验证API的输入和输出类型。

|  | 意图 | 过程 | 过程 | 流程图 | 对话图 | 对话 | 测试 |
| --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  | 带API |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 生成的 | 84 | 168 | 132 | 70 | 49 | 217 | 1,420 |
| + 自动过滤器 | – | – | 98 | 55 | 33 | – | – |
| + 手动过滤器 | – | 132 | 70 | 49 | 33 | 192 | – |
| ALMITA | 14 | 18 | 18 | 18 | 18 | 192 | 1,420 |
| 自动-ALMITA | 52 | 63 | 63 | 63 | 63 | 407 | 2,696 |

表1：通过从84个意图引导ALMITA数据集时的统计数据。我们展示了在（i）生成、（ii）自动过滤和（iii）人工过滤注释之后的样本数量。“–”表示没有过滤。auto-ALMITA使用与ALMITA相同的84个种子意图创建，但使用相同的管道且没有任何人工过滤，这样我们可以评估在没有人工标注员的情况下，测试生成管道的能力。

### 3.9 测试提取器

测试提取器将单个对话转换为一个或多个测试。它通过迭代地将对话分解为子对话（或上下文），每个子对话以客户消息（例如，“取消我的订单”）或API输出（例如，“成功”作为取消功能调用的后续）结束。其基本原理是，由于生成的对话示例了正确的流程，我们可以使用前面的消息构建上下文，预期输出为下一个非客户消息，无论是代理响应还是API调用。[图4](https://arxiv.org/html/2409.15934v2#S3.F4 "在3.6 噪声生成器 ‣ 3 方法 ‣ 自动化测试生成用于评估工具增强的LLM作为对话AI代理")展示了从生成的对话中提取的三个测试示例。测试用于通过提供上下文并将其响应与预期输出进行比较来评估AI代理。

## 4 结果

| LLM | 回复 | API | 测试 | 对话 |
| --- | --- | --- | --- | --- |
| 召回率 | 正确率 | 召回率 | 正确率 | 正确的参数 | 正确 | 正确 |
| GPT-4o | 92.7 | 75.2 | 96.7 | 99.8 | 92.2 | 88.9 | 14.1 |
| Mistral-NeMo-I | 92.0 | 65.0 | 89.8 | 99.5 | 92.1 | 84.7 | 7.3 |
| Claude3-s | 88.0 | 60.3 | 96.2 | 99.8 | 90.5 | 83.3 | 10.4 |
| GPT-4 | 53.2 | 77.7 | 98.1 | 99.8 | 93.0 | 76.9 | 4.2 |
| Llama3.1-8b-I | 74.8 | 53.5 | 72.1 | 90.8 | 85.9 | 73.1 | 1.6 |
| GPT-4o w/ F | 92.9 | 74.8 | 97.2 | 99.0 | 86.6 | 88.0 | 15.6 |

表2：评估AI代理在生成正确回答并调用正确API方面的能力。我们使用相同的提示测试不同的LLM。此外，我们还使用函数调用对LLM进行评估（带有“w/ F”后缀）。闭源模型的版本为gpt-4-0613、gpt-4o-2024-05-13、anthropic.claude-3-sonnet-20240229-v1:0。“-I”后缀表示它是一个指令模型。所有结果以百分比表示，最高值加粗显示。

在[4.1节](https://arxiv.org/html/2409.15934v2#S4.SS1 "4.1 数据集生成：ALMITA ‣ 4 结果 ‣ 用于评估工具增强型LLM作为对话式AI代理的自动化测试生成")中，我们详细介绍了ALMITA的创建过程，这是一个手动策划的数据集，用于评估LLM作为AI客户支持代理的表现。两名注释员独立审查每一个数据点，以识别错误实例，随后进行讨论以对齐他们的评估并尽量减少分歧。任何至少被一名注释员认为是错误的数据点都会被删除。所有生成步骤都使用GPT-4（参见[图1](https://arxiv.org/html/2409.15934v2#S1.F1 "图1 ‣ 1 引言 ‣ 用于评估工具增强型LLM作为对话式AI代理的自动化测试生成")）。为了评估图形中间结构的好处，我们进行了一项消融研究，将直接从程序生成的对话与使用中间结构生成的对话进行比较，并进行手动策划以评估质量（[4.2节](https://arxiv.org/html/2409.15934v2#S4.SS2 "4.2 消融研究：来自程序的对话 ‣ 4 结果 ‣ 用于评估工具增强型LLM作为对话式AI代理的自动化测试生成")）。在[4.3节](https://arxiv.org/html/2409.15934v2#S4.SS3 "4.3 LLM AI代理评估 ‣ 4 结果 ‣ 用于评估工具增强型LLM作为对话式AI代理的自动化测试生成")中，我们评估了在ALMITA上表现的各种AI代理。最后，在[4.4节](https://arxiv.org/html/2409.15934v2#S4.SS4 "4.4 完全自动化测试：auto-ALMITA ‣ 4 结果 ‣ 用于评估工具增强型LLM作为对话式AI代理的自动化测试生成")中，我们评估了我们流水线在自动生成高质量测试集方面的有效性。我们通过将AI代理在ALMITA上的表现与其在完全自动化版本auto-ALMITA上的表现进行比较来进行评估。

### 4.1 数据集生成：ALMITA

我们首先通过请求大型语言模型（LLM）使用来自[第 A.1 节](https://arxiv.org/html/2409.15934v2#A1.SS1 "A.1 Intent generation ‣ Appendix A Prompts ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")的提示来生成意图，结果生成了84个意图。以这些意图为输入，我们提示模型为每个意图生成两个过程，总共生成了168个过程。经过人工注释后，我们删除了36个不符合[第 3.2 节](https://arxiv.org/html/2409.15934v2#S3.SS2 "3.2 Procedure generator ‣ 3 Method ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")规则的过程。有效的过程平均包含315个字（范围从171到535）和11个步骤（范围从6到19）。接下来，我们提取每个过程的API，具体见[第 3.3 节](https://arxiv.org/html/2409.15934v2#S3.SS3 "3.3 API extractor ‣ 3 Method ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")。不符合正确JSON格式的API会被自动过滤掉，同时那些包含无效API的过程也会被删除，最终得到了70个有效的过程。每个有效过程平均包含4个API（范围从2到9）。对于这70个包含API的过程，我们生成相应的流程图。我们自动过滤掉了15个流程图，并手动过滤掉了另外6个不符合[第 3.4 节](https://arxiv.org/html/2409.15934v2#S3.SS4 "3.4 Flowgraph generator ‣ 3 Method ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")规则的流程图。有效的流程图平均包含15个节点（范围从10到20）和17条边（范围从10到25）。对于剩余的49个有效流程图，我们生成了相应的对话图。我们自动排除了16个对话图，并手动排除了另外7个，依据的是遵循规则的情况（参见[第 3.5 节](https://arxiv.org/html/2409.15934v2#S3.SS5 "3.5 Conversation graph generator ‣ 3 Method ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")）。有效的对话图平均包含23个节点（范围从16到37）和24条边（范围从15到37）。从这些对话图中，我们通过路径采样（参见[第 3.7 节](https://arxiv.org/html/2409.15934v2#S3.SS7 "3.7 Path sampler ‣ 3 Method ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")）生成了217个对话。我们手动过滤掉了25个未遵循规则的对话（参见[第 3.8 节](https://arxiv.org/html/2409.15934v2#S3.SS8 "3.8 Conversation generator ‣ 3 Method ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")）。因此，从最初的84个意图中，我们得到了192个有效的对话。每个对话平均遍历了12个节点（范围从3到24）。最后，从这些对话中提取了测试，如[第 3.9 节](https://arxiv.org/html/2409.15934v2#S3.SS9 "3.9 Test extractor ‣ 3 Method ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")中所述，最终生成了1420个测试。表[1](https://arxiv.org/html/2409.15934v2#S3.T1 "Table 1 ‣ 3.8 Conversation generator ‣ 3 Method ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")总结了数据集的统计信息。最终，ALMITA数据集包含14个意图、18个过程、18个流程图、18个对话图、192个对话和1420个测试。

### 4.2 消融研究：来自程序的对话

我们进行了一项消融研究，以验证中间图表示在生成正确对话方面的有效性。我们去除了流图生成器、对话图生成器、噪声生成器和路径采样器，直接通过程序和 API 使用来自[第 A.7 节](https://arxiv.org/html/2409.15934v2#A1.SS7 "A.7 Conversations from procedures ‣ Appendix A Prompts ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")的提示生成对话。从程序直接生成的对话注释显示，这比从图中生成的对话注释要复杂且耗时得多。因此，我们仅注释了 50 个对话。所有 50 个对话都是从与 ALMITA 相同的 70 个输入程序中生成的，并且由同样的两位注释员策划，遵循相同的注释策略。K 简化流程产生了约 68%（$34/50$）有效对话，这是由与 ALMITA 相同的注释员进行评估的。相比之下，使用中间图表示的原始流程产生了约 88%（$192/217$）有效对话。这表明，图表示提高了生成对话的有效性。即使考虑到策划流图的累积影响，原始流程也会自动生成约 78%（$192/217\times 49/55$）有效对话，超过了约 68%。

此外，虽然简化流程中使用的提示可能有待改进，但简化流程本身并不能确保所有的分支路径都被探索。这突显了中间图表示在覆盖所有可能对话路径中的优势。

### 4.3 LLM AI 代理的评估

我们使用ALMITA来评估作为客户支持AI代理的LLMs。该数据集使我们能够评估以下维度，并在[表2](https://arxiv.org/html/2409.15934v2#S4.T2 "In 4 Results ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")中报告这些维度的结果：(i) *回复召回率*：当正确的动作是回复时，代理能够正确地发送回复消息，而不是调用不必要的API；(ii) *正确回复*：当正确动作和预测动作都是回复时，代理的回复与预期回复一致（我们使用BERTScore，并在检查一些示例后设置了相似度阈值为0.55）；(iii) *API召回率*：当正确的动作是进行API调用时，代理能够正确地检测到需要执行API调用，而不是进行回复；(iv) *正确的API*：当正确的动作和预测动作都是执行API调用时，代理会调用正确的API；(v) *正确的API参数*：当正确的动作和预测动作都是相同的API调用时，代理会使用正确的参数值调用API；(vi) *测试正确性（或测试准确性）*：测试是否完全正确（即调用正确的回复/API，且如果正确动作是API调用，则调用正确的API并使用正确的参数，或者如果正确动作是回复，则提供正确的回复）；(vii) *对话正确性（或对话准确性）*：对话中的所有测试序列是否完全正确。

我们评估了5种不同的大型语言模型（LLMs）：GPT4-o、GPT-4、Claude3-sonnet、Mistral-NeMo-Instruct和Llama3.1-8b-Instruct。为了确保公平，我们对所有模型使用了统一的提示（详情请见[第A.8节](https://arxiv.org/html/2409.15934v2#A1.SS8 "A.8 Tool-augmented AI agent ‣ Appendix A Prompts ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")）。我们的提示旨在保持通用性，避免偏向特定模型，尽管我们也认识到不同的模型可能在不同的提示风格下表现更好。由于数据集包括了API调用，我们还使用功能调用对GPT4-o进行了测试，表示为GPT-4o w/F。

我们观察到，所有的大型语言模型（LLMs）在使用API时表现出很高的准确性，在*正确的API*和*正确的API参数*维度上，准确率均超过85%。除Llama3.1-8b-I外，其他模型都能正确判断何时应该调用API，且*API召回率*超过90%。然而，其他维度的表现明显较低，这表明仅关注API调用的数据集无法全面评估AI代理的能力。

有趣的是，即使没有必要，GPT-4 也倾向于调用 API，这导致其*回复召回率*低于其他模型。在*正确回复*方面，GPT 模型的表现优于其他模型，尽管这可能会受到使用 GPT-4 生成测试的偏见影响。在*测试正确性*方面，GPT-4o、Claude3-s 和 Mistral-NeMo-Instruct的表现最好，而 GPT-4 和 Llama3.1-8b-Instruct 排名最低。

最为关键的是，我们发现所有模型在*正确对话*方面的表现都非常低。实际上，这意味着这些 AI 代理很可能会在与用户的对话中的某个环节失败。这显示出现有的大型语言模型（LLM）存在一些局限性，需要更好的模型或非常精心设计的提示，才能适合作为完全自主的客户支持 AI 代理。

我们的数据集可能对评估未来模型和/或策略在其 AI 代理能力上的表现有所帮助。此外，由于该流程是自动化的，该数据集可以更新，以包含更多（且更难的）测试，并且可以适应新的或更具体的领域。

### 4.4 完全自动化测试：auto-ALMITA

在本节中，我们分析了 AI 代理在 auto-ALMITA（ALMITA 数据集的完全自动化版本）上的表现。该数据集使用来自 ALMITA 数据集的相同种子意图创建，详见[第4.1节](https://arxiv.org/html/2409.15934v2#S4.SS1 "4.1 Dataset generation: ALMITA ‣ 4 Results ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")。然后我们运行相同的流程，但不进行人工筛选步骤。Auto-ALMITA 保留了更多的数据点和更大的多样性（见表[1](https://arxiv.org/html/2409.15934v2#S3.T1 "Table 1 ‣ 3.8 Conversation generator ‣ 3 Method ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")），尽管质量有所下降。由于完全自动生成，auto-ALMITA 也可以在不需要额外策划的情况下轻松扩展。

我们评估了来自[表 2](https://arxiv.org/html/2409.15934v2#S4.T2 "在 4 结果 ‣ 自动化测试生成以评估工具增强型 LLM 作为对话 AI 代理")的相同 LLM 代理，并比较了 AI 代理在 auto-ALMITA 和 ALMITA 数据集上获得的全球度量测试正确值，如[图 5](https://arxiv.org/html/2409.15934v2#S4.F5 "在 4.4 完全自动化测试：auto-ALMITA ‣ 4 结果 ‣ 自动化测试生成以评估工具增强型 LLM 作为对话 AI 代理")所示。两个数据集按照相同的顺序对 LLMs 排名，且其相关值为 0.98（详细结果见附表[1](https://arxiv.org/html/2409.15934v2#A2.T1 "表 1 ‣ 附录 B auto-ALMITA：详细评估 ‣ 自动化测试生成以评估工具增强型 LLM 作为对话 AI 代理")）。这些发现表明，所提议的管道可以完全自动生成用于评估 AI 代理的数据集，从而得出与从策划数据集中得出的结论相似的结果。

<svg class="ltx_picture" height="217.12" id="S4.F5.pic1" overflow="visible" version="1.1" width="580.46"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,217.12) matrix(1 0 0 -1 0 0) translate(42.45,0) translate(0,37.53) matrix(1.0 0.0 0.0 1.0 -42.45 -37.53)"><g class="ltx_nestedsvg" transform="matrix(1 0 0 1 0 0) translate(42.45,0) translate(0,37.53)"><g fill="#000000" stroke="#000000" stroke-width="0.4pt"><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 13.76 -13.81)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$70$</foreignobject></g><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 96.49 -13.81)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$74$</foreignobject></g><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 179.22 -13.81)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$78$</foreignobject></g><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 261.95 -13.81)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$82$</foreignobject></g><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 344.68 -13.81)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$86$</foreignobject></g><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 427.4 -13.81)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$90$</foreignobject></g><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 510.13 -13.81)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$94$</foreignobject></g><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -18.73 2.23)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$70$</foreignobject></g><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -18.73 28.99)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$74$</foreignobject></g><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -18.73 55.76)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$78$</foreignobject></g><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -18.73 82.52)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$82$</foreignobject></g><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -18.73 109.28)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$86$</foreignobject></g><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -18.73 136.05)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$90$</foreignobject></g><g color="#000000" fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -18.73 162.81)"><foreignobject height="8.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="13.84">$94$</foreignobject></g><g clip-path="url(#pgfcp13)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 170.34 38.76)"><foreignobject height="8" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="12.61">![Refer to caption](img/343b5e0d6d552f2b843adbd222929686.png) GPT-4</foreignobject></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 97.96 25.3)"><foreignobject height="7" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.61">![Refer to caption](img/a9e167facd5937babcf68c4bd62df881.png) Llama3.1-8b-I</foreignobject></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 418.53 105.67)"><foreignobject height="8" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="12.61">![Refer to caption](img/343b5e0d6d552f2b843adbd222929686.png) GPT-4o</foreignobject></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 397.85 90.21)"><foreignobject height="8" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="12.61">![Refer to caption](img/343b5e0d6d552f2b843adbd222929686.png) GPT-4o w/ F</foreignobject></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 325.46 78.83)"><foreignobject height="9.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="24.68">![Refer to caption](img/0b78ec89143536bfb91dfe449724333c.png)Mistral-NeMo-I</foreignobject></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 304.78 58.76)"><foreignobject height="6" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="12.61">![Refer to caption](img/b22af2f2461a97a7b81df376088e2e3f.png) Claude3-sonnet</foreignobject></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 196.63 -32.92)"><foreignobject height="8.51" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="73.18">*test correct* @ ALMITA</foreignobject></g><g fill="#000000" stroke="#000000" transform="matrix(0.0 1.0 -1.0 0.0 -28.23 -1.02)"><foreignobject height="8.51" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="73.18">*test correct* @ auto-ALMITA</foreignobject></g></g></g></g></svg>

图 5：不同 LLM 代理在 auto-ALMITA 和 ALMITA 数据集上的*测试正确*值。

## 5 结论

LLMs 被用作客户支持 AI 代理。然而，现有的评估数据集在其范围上存在局限性。我们提出了一种自动化测试生成管道，用于评估工具增强型对话 AI 代理。我们提出的方法利用 LLMs 生成中间图结构，帮助减少幻觉现象并提高生成测试的多样性。我们评估了不同的 LLMs，分析了作为 AI 代理实现的 LLMs 的当前能力。

为了促进这一点，我们开发了 ALMITA 数据集，利用该数据集对这些 AI 代理进行了彻底评估，并识别了它们的局限性。ALMITA 允许从多个关键维度对 AI 代理进行多方面评估，如回复准确性、API 调用正确性和整体对话完整性。我们的研究结果突出了当前大型语言模型（LLMs）的重大局限性，特别是在维持整个用户交互中的正确对话方面。

重要的是，ALMITA 数据集可以被其他研究人员用来评估 AI 代理，提供一个全面的基准，用于评估其性能的各个方面，可能还适用于其他目标领域。此外，由于我们的测试生成管道是完全自动化的，我们有能力创建新的、更具挑战性的版本数据集。这种适应性确保我们的框架能够不断更新，以反映更加复杂和现实的场景，进一步增强其在客户支持及其他领域的持续研究和开发中的实用性。

## 6 限制

我们的评估存在一些局限性。具体来说，我们没有定量评估生成测试的多样性。我们进行了人工标注，以验证每一步的正确性，但标注的数量和标注者的数量都较少。我们的测试生成管道只使用了单一的 LLM 作为生成器，即 GPT4，这可能会影响评估。对此的一个可能缓解措施是对其他 LLM 重复测试生成管道并汇总测试结果。我们评估了多个 LLM，但仅使用了单一的提示。我们的目标是测试不同模型在生成数据集上的表现，但可以考虑更先进的 AI 代理。

此外，我们认识到某些指标可能过于严格。作为未来的方向，我们希望考虑 AI 代理在对话中的错误严重性。对话相对流畅，除了数据集中标注的最明显和直接的步骤外，我们可能还会有其他在某种程度上可以接受的回复/操作。仍然需要开发更先进、更语义化的对话评估指标，允许某些路径变动，类似于比较两句话时，不同的单词和单词顺序可能传达相似的含义。

## 附录 A 提示

### A.1 意图生成

<svg class="ltx_picture" height="127.17" id="A1.SS1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,127.17) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 109.11)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">系统提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="77.62" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你是<REDACTED>，一个提供客户支持的平台。你为来自多个不同领域的客户提供服务：互联网服务提供商、金融机构、电商平台、娱乐网站等。所有这些客户都有可以联系客户支持的用户，用户通过联系客服获取信息、投诉或出于其他原因联系客户支持团队。</foreignobject></g></g></svg><svg class="ltx_picture" height="1678.85" id="A1.SS1.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,1678.85) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 1660.8)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">用户提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="1629.31" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你的任务是生成一个问题列表，列出可能导致客户联系支持的各种问题。请考虑问题相关的客户类型，描述问题的具体内容，并为每个错误起一个简短的名称。从一个多样化的客户池中生成{{ number_issues }}个问题。请将你的回答格式化为JSON，结构如下：[⬇](data:text/plain;base64,W3sKICAgImNsaWVudCI6IGUuZy4sIGEgYmFuaywgaW50ZXJuZXQgcHJvdmlkZXIsIGV0Yy4gRG8gbm90IGxpbWl0IHlvdXJzZWxmIHRvIHRoZXNlIGV4YW1wbGVzISwKICAgImlzc3VlIjogZGVzY3JpcHRpb24gb2YgdGhlIGVycm9yLCBiZSBzcGVjaWZpYyEsCiAgICJuYW1lIjogYSBzaG9ydCBuYW1lIGZvciB0aGUgaXNzdWUKfV0=) [{ "client":  e.g.,  a  bank,  internet  provider,  etc.  Do  not  limit  yourself  to  these  examples!, "issue":  description  of  the  error,  be  specific!, "name":  a  short  name  for  the  issue }]</foreignobject></g></g></svg>

### A.2 过程生成

<svg class="ltx_picture" height="374.64" id="A1.SS2.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,374.64) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 356.59)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">系统提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="325.09" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">您是<REDACTED>，一个提供客户支持的平台。您服务的客户来自不同的行业：互联网服务提供商、金融机构、电子商务平台、娱乐网站等。这些客户的消费者可以联系客户支持，获取信息、提出投诉或因其他原因联系客户支持团队。您的任务是生成一套程序，帮助代理完成任务。代理可以采取行动，或者请求客户提供数据（例如电子邮件地址）。您可以在程序中包括分支。请勿给出诸如“每个系统可能有不同的流程”之类的笼统陈述。相反，请假设您是一个拥有非常明确流程的具体公司。请勿给出诸如“解释公司的政策”之类的一般步骤。代理是按照程序执行的，因此所有步骤需要明确说明，例如，准确地说明政策是什么。不要留下任何歧义或信息缺失。不要在程序中提到未解决的条件，例如“如果政策允许”。每个条件必须完全包含在程序中，代理无需阅读其他文档，也不依赖于关于公司流程的其他知识。您的角色是创造合理且不含糊的场景。步骤应当精确且具体。避免给出例子，我们需要简洁的程序。不要包括与客户互动无关的行动（例如，记录互动、监控过程）。该程序仅涉及如何解决客户报告的问题。假设您没有浏览器。请勿包括导航步骤，仅提供代理应采取的行动。</foreignobject></g></g></svg><svg class="ltx_picture" height="348.43" id="A1.SS2.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,348.43) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 330.38)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">用户提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="298.88" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">[⬇](data:text/plain;base64,IyBJc3N1ZQp7eyBpc3N1ZSB9fQ==) # 问题 {{  issue  }}</foreignobject></g></g></svg>

### A.3 API 提取

<svg class="ltx_picture" height="2142.39" id="A1.SS3.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,2142.39) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 2124.34)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">系统提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="2092.85" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你是一个为客户体验公司工作的编程助手。在给定一个代理需要遵循的步骤来解决客户问题时，你的任务是提取代理可能使用的所有 API。绝不要生成要求密码的 API 调用。API 应该尽可能具体地与步骤中的内容相关，而不是通用方法。所有 API 参数的类型都应该不同于 None。当表示结构化输出时，遵循 Python 的约定，如 list[str] 或 dict[str, float]。可选参数应遵循 Python 的约定，例如 Optional[str]。如果步骤中没有代理应该执行的操作，则返回空的 JSON。 [⬇](data:text/plain;base64,IyBPdXRwdXQKUmVzcG9uZCBvbmx5IGluIEpTT04gZm9ybWF0IHdpdGggdGhlIGZvbGxvd2luZyBzY2hlbWEuIFRoZSBuYW1lIG9mIHRoZSBhcGkgc2hvdWxkIGJlIHdyaXR0ZW4gaW4gc25ha2UgY2FzZS4KeyJhcGlzIjogW3sibmFtZSI6IHN0ciwgImRlc2MiOiBzdHIsICJwYXJhbXMiOiBbeyJuYW1lIjogc3RyfV0sICAib3V0cHV0IjogeyJuYW1lIjogc3RyLCAidHlwZSI6IHN0cn19XX0=) #  输出仅以 JSON 格式响应，使用以下结构。API 的名称应采用 snake_case 格式。{"apis":  [{"name":  str,  "desc":  str,  "params":  [{"name":  str}],  "output":  {"name":  str,  "type":  str}}]}</foreignobject></g></g></svg><svg class="ltx_picture" height="514.47" id="A1.SS3.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,514.47) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 496.42)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">用户提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="464.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">[⬇](data:text/plain;base64,IyBQcm9jZWR1cmUKYGBgCnt7IHByb2NlZHVyZSB9fQpgYGA=) #  步骤 ‘‘‘ {{  procedure  }} ‘‘‘</foreignobject></g></g></svg>

### A.4 流图生成

<svg class="ltx_picture" height="861.09" id="A1.SS4.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,861.09) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 843.04)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">系统提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="811.54" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你是一个经验丰富的流程图创建者。你将获得一个被<procedure></procedure>标签包围的程序，以及一个可以使用的API列表，包围在<apis></apis>标签中。你的任务是提取出用来解决问题的流程图。这个流程图将被一个助手用来了解如何解决问题。代理无法访问程序，因此所有信息必须包含在这个流程图中！！你是一个经验丰富的流程图创建者。你将获得一个被<procedure></procedure>标签包围的程序，以及一个可以使用的API列表，包围在<apis></apis>标签中。你的任务是提取出用来解决问题的流程图。这个流程图将被一个助手用来了解如何解决问题。代理无法访问程序，因此所有信息必须包含在这个流程图中！！流程图由以下格式的节点和边构成：[⬇](data:text/plain;base64,W25vZGVfaWRdKG5vZGVfdHlwZSl7bm9kZV9kZXNjcmlwdGlvbn0KW2VkZ2VfaWRdKHBhcmVudF9ub2RlX2lkLCBjaGlsZF9ub2RlX2lkKXtlZGdlX2Rlc2NyaXB0aW9ufQ==) [node_id](node_type){node_description} [edge_id](parent_node_id,  child_node_id){edge_description} 节点ID应始终以N后跟整数表示。边的ID应始终以E后跟整数表示。你可以使用以下类型的节点：start_message, message, api 和 end_message。 - start_message: 初始消息由助手发送给客户，取自程序。它没有父节点。 - message: 向客户发送的消息节点。此消息应包含程序中的所有详细信息。 - api: 代理应执行的API调用。 - end_message: 发送消息并结束执行的节点。图构造规则 - 图只有一个根节点，类型为‘start_message‘。 - 消息节点的外发边是客户的回复。客户消息必须具体。 - API节点的外发边是API的输出。 - 结束节点没有外发边，类型应为end_message。 - 结束节点类型为‘end_message‘。 - 永远不要有边返回到起始节点N0。</foreignobject></g></g></svg><svg class="ltx_picture" height="8199.77" id="A1.SS4.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,8199.77) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="8172.21" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">详细信息 代理和客户的消息应严格按照程序中的内容进行。程序中的所有详细信息都需要包含在流程图中！不要假设代理会看到程序，因此这些细节很重要，例如失败的原因或需要收集的信息。确保所有步骤都是节点。有些程序可能有分支路径。始终在适当时使用API。流程图必须被<flow></flow>标签包围。[⬇](data:text/plain;base64,RXhhbXBsZSBvZiBhIGZsb3c6CjxmbG93PgpbTjBdKHN0YXJ0X21lc3NhZ2Upe0dyZWV0IHRoZSBjdXN0b21lcn0KW0UwXShOMCwgTjEpe0RpZG4ndCByZWNlaXZlIG15IG9yZGVyfQpbTjFdKG1lc3NhZ2Upe0FzayBjdXN0b21lciBmb3Igb3JkZXIgaWQsIHRoZSBlbWFpbCBvciBwaG9uZSBudW1iZXJ9CltFMl0oTjEsIE4yKXtHaXZlcyBvcmRlciBpZCBhbmQgZW1haWx9CltFM10oTjEsIE4zKXtHaXZlcyBvcmRlciBpZCBhbmQgcGhvbmUgbnVtYmVyfQpbTjJdKGFwaSl7Z2V0X29yZGVyX2RldGFpbHNfYnlfZW1haWx9CltOM10oYXBpKXtnZXRfb3JkZXJfZGV0YWlsc19ieV9waG9uZV9udW1iZXJ9CltONF0obWVzc2FnZSl7RG8geW91IHdhbnQgdG8gY2FuY2VsIG9yIHJlZnVuZCB0aGUgb3JkZXI/fQpbRTNdKE4yLCBONCl7Rm91bmQgb3JkZXJ9CltFNV0oTjMsIE40KXtGb3VuZCBvcmRlcn0KW041XShtZXNzYWdlKXtUZWxsIHRoZSB1c2VyIHRoZSBvcmRlciB3YXNuJ3QgZm91bmQgYW5kIGFzayBmb3IgY29ycmVjdCBpbmZvcm1hdGlvbn0KW0U1XShOMiwgTjUpe09yZGVyIG5vdCBGb3VuZH0KW0E1XShOMzMsIE5bM15XLCBOOSl7U3VjY2Vzc30KPC9mbG93PgoKPGFwaXM+Cnt7IGFwaXMgfX0KPC9hcGlzPg==) 示例流程图： <flow> [N0](start_message){问候客户} [E0](N0,  N1){没有收到我的订单} [N1](message){询问客户订单号、电子邮件或电话号码} [E2](N1,  N2){提供订单号和电子邮件} [E3](N1,  N3){提供订单号和电话号码} [N2](api){通过电子邮件获取订单详情} [N3](api){通过电话号码获取订单详情} [N4](message){你想要取消还是退款订单？} [E3](N2,  N4){找到了订单} [E5](N3,  N4){找到了订单} [N5](message){告诉用户订单未找到，并询问正确的信息} [E5](N2,  N5){未找到订单} [E6](N3,  N5){未找到订单} [E6](N5,  N2){用户提供了另一个电子邮件或订单号} [E7](N5,  N3){用户提供了另一个电话号码或订单号} [N6](api){取消订单} [E8](N4,  N6){我想取消订单} [N7](end_message){订单已取消} [E9](N6,  N7){成功} [N8](api){退款订单} [E9](N4,  N8){我想要退款} [N9](end_message){订单已退款} [E10](N8,  N9){成功} </flow> <apis> {{  apis  }} </apis></foreignobject></g></g></svg><svg class="ltx_picture" height="448.05" id="A1.SS4.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,448.05) matrix(1 0 0 -1 0 0)

### A.5 对话图生成

<svg class="ltx_picture" height="1175.8" id="A1.SS5.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,1175.8) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 1157.75)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">系统提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="1126.25" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">您的任务是将一个流程图转换为对话图。流程图将位于<flowchart></flowchart>之间。流程图由以下格式的节点和边组成：[⬇](data:text/plain;base64,W25vZGVfaWRdKG5vZGVfdHlwZSl7bm9kZV9kZXNjcmlwdGlvbn0KW2VkZ2VfaWRdKHBhcmVudF9ub2RlX2lkLCBjaGlsZF9ub2RlX2lkKXtlZGdlX2Rlc2NyaXB0aW9ufQ==) [node_id](node_type){node_description} [edge_id](parent_node_id,  child_node_id){edge_description} 节点有以下类型： - start_message: 助手发送给客户的初始消息，来自程序。 - message: 助手发送给客户的消息节点。 - api: 助手应执行的 API 调用。 - end_message: 发送助手消息并完成执行的节点。您需要将其转换为对话图，其中： [⬇](data:text/plain;base64,W25vZGVfaWRdKG5vZGVfdHlwZSl7bm9kZV9kZXNjcmlwdGlvbn0KW2VkZ2VfaWRdKHBhcmVudF9ub2RlX2lkLCBjaGlsZF9ub2RlX2lkKXtlZGdlX2Rlc2NyaXB0aW9ufQ==) [node_id](node_type){node_description} [edge_id](parent_node_id,  child_node_id){edge_description} 节点有以下类型： - assistant: 由代理发送的消息。 - user: 由用户发送的消息。 - api: 代理应执行的 API 调用。 图形构建规则： - API 节点有出边并附有标签 - API 节点后跟 API 或助手节点 - 用户节点后跟 API 或助手节点 - 助手节点**只能跟随**用户节点 - 叶节点是助手节点</foreignobject></g></g></svg><svg class="ltx_picture" height="13031.2" id="A1.SS5.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,13031.2) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="13003.64" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">边连接用户节点与助手或 API 节点。只有来自 API 调用的边可以有描述。第一个节点应从助手节点开始，而没有父节点。例如，考虑以下流程图：[⬇](data:text/plain;base64,PGZsb3c+CltOMF0oc3RhcnRfbWVzc2FnZSl7R3JlZXQgdGhlIGN1c3RvbWVyfQpbRTBdKE4wLCBOMSl7RGlkbid0IHJlY2VpdmUgbXkgb3JkZXJ9CltOMV0obWVzc2FnZSl7QXNrIGN1c3RvbWVyIGZvciBvcmRlciBpZH0KW0UyXShOMSwgTjIpe0dpdmVzIG9yZGVyIGlkfQpbTjJdKGFwaSl7Z2V0X29yZGVyX2RldGFpbHN9CltOM10obWVzc2FnZSl7RG8geW91IHdhbnQgdG8gY2FuY2VsIG9yIHJlZnVuZCB0aGUgb3JkZXI/fQpbRTNdKE4yLCBOMyl7Rm91bmQgb3JkZXJ9CltONF0obWVzc2FnZSl7VGVsbCB0aGUgdXNlciB0aGUgb3JkZXIgd2Fzbid0IGZvdW5kfQpbRTRdKE4yLCBONCl7T3JkZXIgbm90IEZvdW5kfQpbRTVdKE40LCBOMil7VXNlciBnaXZlcyBhbm90aGVyIG9yZGVyIGlkfQpbTjVdKGFwaSl7Y2FuY2VsX29yZGVyfQpbRTZdKE4zLCBONSl7SSB3YW50IHRvIGNhbmNlbCB0aGUgb3JkZXJ9CltONl0oZW5kX21lc3NhZ2Upe09yZGVyIGNhbmNlbGxlZH0KW0U3XShONSwgTjYpe1N1Y2Nlc3N9CltON10oYXBpKXtyZWZ1bmRfb3JkZXJ9CltFOF0oTjMsIE43KXtJIHdhbnQgYSByZWZ1bmR9CltOOF0oZW5kX21lc3NhZ2Upe09yZGVyIHJlZnVuZGVkfQpbRTldKE43LCBOMOCl7U3VjY2Vzc30KPC9mbG93Pg==) <flow> [N0](start_message){问候客户} [E0](N0,  N1){未收到订单} [N1](message){请求客户提供订单号} [E2](N1,  N2){提供订单号} [N2](api){获取订单详情} [N3](message){您想取消还是退款？} [E3](N2,  N3){找到了订单} [N4](message){告诉用户没有找到订单} [E4](N2,  N4){未找到订单} [E5](N4,  N2){用户提供另一个订单号} [N5](api){取消订单} [E6](N3,  N5){我想取消订单} [N6](end_message){订单已取消} [E7](N5,  N6){成功} [N7](api){退款订单} [E8](N3,  N7){我想要退款} [N8](end_message){订单已退款} [E9](N7,  N8){成功} </flow> 正确输出为： [⬇](data:text/plain;base64,PGZsb3c+CltOMF0oYXNzaXN0YW50KXtHcmVldCB0aGUgY3VzdG9tZXJ9CltOMV0odXNlcil7RGlkbid0IHJlY2VpdmUgbXkgb3JkZXJ9CltFMF0oTjAsIE4xKXt9CltOMl0oYXNzaXN0YW50KXtBc2sgY3VzdG9tZXIgZm9yIG9yZGVyIGlkfQpbRTFdKE4xLCBOMil7fQpbTjNdKHVzZXIpe0dpdmVzIG9yZGVyIGlkfQpbRTJdKE4yLCBOMyl7fQpbTjRdKGFwaSl7Z2V0X29yZGVyX2RldGFpbHN9CltFM10oTjMsIE40KXt9CltONV0oYXNzaXN0YW50KXtEbyB5b3Ugd2FudCB0byBjYW5jZWwgb3IgcmVmdW5kIHRoZSBvcmRlcj99CltFNF0oTjQsIE

### A.6 对话生成

<svg class="ltx_picture" height="196.22" id="A1.SS6.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,196.22) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 178.17)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">系统提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="146.67" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你将收到一个包含节点和边的对话图，格式如下： -[Ni](assistant){消息}: 代理节点和相应的消息。 -[Nj](user){消息}: 用户节点和相应的消息。 -[Nk](api){消息}: API节点和相应的消息。 该图还包含以下格式的边： -[Ei](Ni,Nj){}: 消息Ni发生在Nj之前。 -[Ej](Ni,Nj){api_output}: 仅适用于Ni为API节点时，消息Ni发生在Nj之前，并且有api输出api_output。 流程图包含在<flow></flow>中。初始节点是[N1]。代理在整个过程中引导用户。我们的目标是根据给定的路径生成符合图示路径的对话，路径在<paths></paths>之间。</foreignobject></g></g></svg><svg class="ltx_picture" height="13862.48" id="A1.SS6.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,13862.48) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="13834.92" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">例如，考虑以下流程图：[⬇](data:text/plain;base64,PGZsb3c+CltOMV0oYXNzaXN0YW50KXtHcmVldCB0aGUgY3VzdG9tZXJ9CltOMl0odXNlcil7RGlkbid0IHJlY2VpdmUgbXkgb3JkZXJ9CltFMV0oTjEsIE4yKXt9CltOM10oYXNzaXN0YW50KXtBc2sgY3VzdG9tZXIgZm9yIG9yZGVyIGlkfQpbRTJdKE4yLCBOMyl7fQpbTjRdKHVzZXIpe0dpdmVzIG9yZGVyIGlkfQpbRTNdKE4zLCBONCl7fQpbTjVdKGFwaSl7Z2V0X29yZGVyX2RldGFpbHN9CltFNF0oTjQsIE41KXt9CltONl0oYXNzaXN0YW50KXtXYW50IHRvIGNhbmNlbCBvciByZWZ1bmQgdGhlIG9yZGVyP30KW0U1XShONSwgTjYpe0ZvdW5kIG9yZGVyfQpbTjddKGFzc2lzdGFudCl7VGVsbCB1c2VyIHRoZSBvcmRlciB3YXNuJ3QgZm91bmR9CltFNV0oTjUsIE43KXtPcmRlciBub3QgRm91bmR9CltOOF0odXNlcil7VXNlciBnaXZlcyBhbm90aGVyIG9yZGVyIGlkfQpbRTZdKE43LCBONCl7fQpbRTddKE44LCBONSl7fQpbTjldKHVzZXIpe0kgd2FudCB0byBjYW5jZWwgdGhlIG9yZGVyfQpbRThdKE42LCBONSl7fQpbTjEwXShhcGkpe2NhbmNlbF9vcmRlcn0KW0U5XShOOSwgTjEwKXt9CltOM11dKGFzc2lzdGFudCl7T3JkZXIgY2FuY2VsbGVkfQpbRTEwXShOMTAsIE4xMSl7U3VjY2Vzc30KW04xMl0odXNlcil7SSB3YW50IGEgcmVmdW5kfQpbRTExXShONiwgTjEyKXt9CltOMTNdKGFwaSl7cmVmdW5kX29yZGVyfQpbRTEyXShOMTIsIE4xMyl7fQpbTjE0XShhc3Npc3RhbnQpe09yZGVyIHJlZnVuZGVkfQpbRTEzXShOMTMsIE4xNCl7U3VjY2Vzc30KPC9mbG93Pg==) <flow> [N1](assistant){问候客户} [N2](user){没有收到我的订单} [E1](N1,  N2){} [N3](assistant){请提供订单ID} [E2](N2,  N3){} [N4](user){订单ID是#812} [E3](N3,  N4){} [N5](api){get_order_details} [E4](N4,  N5){} [N6](assistant){是否想要取消或退款该订单？} [E5](N5,  N6){找到订单} [N7](assistant){告诉用户订单未找到} [E5](N5,  N7){订单未找到} [N8](user){用户提供另一个订单ID} [E6](N7,  N8){} [E7](N8,  N5){} [N9](user){我想取消订单} [E8](N6,  N9){} [N10](api){cancel_order} [E9](N9,  N10){} [N11](assistant){订单已取消} [E10](N10,  N11){成功} [N12](user){我想要退款} [E11](N6,  N12){} [N13](api){refund_order} [E12](N12,  N13){} [N14](assistant){订单已退款} [E13](N13,  N14){成功} </flow> 以及API如下：[⬇](data:text/plain;base64,PGFwaXM+ClsKICAgIHsKICAgICAgICAibmFtZSI6ICJnZXRfb3JkZXJfZGV0YWlscyIsCiAgICAgICAgInBhcmFtcyI6IFt7Im9yZGVyX2lkIjogImludCJ9XSwKICAgICAgICAib3V0cHV0IjogeyduYW1lJzogJ3NlbnRfc3RhdHVzJywgJ3R5cGUnOiAnbGlzdFtkaWN0W3N0ciwgc3RyXV0nfQogICAgfQpdCjwvYXBpcz4=) <apis> [ { "name":  "get_order_details", "params":  [{"order_id":  "int"}], "output":  {’name’:  ’sent_status’,  ’type’:  ’list[dict[str,  str]]’} } ] </apis> 如果给定的路径是：[N1, N2, N3, N4, N5, N7]，则可能的对话如下：[⬇](data:text/plain;base64,WwogICAgewogICAgICAgICJyb2xlIjogInVzZXIiLAogICAgICAgICJjb250ZW50IjogIkkgZGlkbid0IHJlY2VpdmUgbXkgb3JkZXIiCiAgICB9LAogICAgewogICAgICAgICJyb2xlIjogImFzc2lzdGFudCIsCiAgICAgICAgImNvbnRlbnQiOiAiQ2FuIHlvdSBnaXZlIG1lIHRoZSBvcmRlciBJRD8iCiAgICB9LA

### A.7 程序中的对话

<svg class="ltx_picture" height="17206.37" id="A1.SS7.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,17206.37) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 17188.32)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">系统提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="17156.82" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你是一个经验丰富的客户服务代表。你将获得一个由<procedure></procedure>包含的流程和一个由<apis></apis>包含的可用API列表。你的目标是生成一个代理与客户之间的对话，这个对话可以通过所提供的流程和API来解决。例如，考虑以下流程：[⬇](data:text/plain;base64,PHByb2NlZHVyZT4KIyBIYW5kbGluZyBhIEN1c3RvbWVyIFdobyBEaWRuJ3QgUmVjZWl2ZSBUaGVpciBPcmRlcgoKU3RhcnQgSW50ZXJhY3Rpb246CjEuMS4gR3JlZXQgdGhlIGN1c3RvbWVyIGNvdXJ0ZW91c2x5LgoKSWRlbnRpZnkgdGhlIElzc3VlOgoyLjEuIENvbmZpcm0gdGhlIGN1c3RvbWVyIGRpZG4ndCByZWNlaXZlIHRoZSBvcmRlci4KCk9idGFpbiBPcmRlciBJbmZvcm1hdGlvbjoKMy4xLiBBc2sgdGhlIGN1c3RvbWVyIHRvIHByb3ZpZGUgdGhlaXIgb3JkZXIgSUQgYWxvbmcgd2l0aCB0aGUgZW1haWwgYWRkcmVzcyBvciBwaG9uZSBudW1iZXIgYXNzb2NpYXRlZCB3aXRoIHRoZSBvcmRlci4KClJldHJpZXZlIE9yZGVyIERldGFpbHM6CjQuMS4gSWYgdGhlIGN1c3RvbWVyIHByb3ZpZGVzIHRoZSBvcmRlciBJRCBhbmQgZW1haWwgYWRkcmVzczoKLSBVc2UgdGhlIGNvbXBhbnkncyBBUEkgdG8gcmV0cmlldmUgb3JkZXIgZGV0YWlscyBieSBlbWFpbC4KNC4yLiBJZiB0aGUgY3VzdG9tZXIgcHJvdmlkZXMgdGhlIG9yZGVyIElEIGFuZCBwaG9uZSBudW1iZXI6Ci0gVXNlIHRoZSBjb21wYW55J3MgQVBJIHRvIHJldHJpZXZlIG9yZGVyIGRldGFpbHMgYnkgcGhvbmUgbnVtYmVyLgoKQ2hlY2sgaWYgT3JkZXIgaXMgRm91bmQ6CjUuMS4gSWYgdGhlIG9yZGVyIGlzIGZvdW5kLCBwcm9jZWVkIHRvIFN0ZXAgNi4KNS4yLiBJZiB0aGUgb3JkZXIgaXMgbm90IGZvdW5kOgotIEluZm9ybSB0aGUgY3VzdG9tZXIgdGhhdCB0aGUgb3JkZXIgd2Fzbid0IGZvdW5kLgotIEFzayB0aGUgY3VzdG9tZXIgdG8gcHJvdmlkZSB0aGUgY29ycmVjdCBlbWFpbCBvciBwaG9uZSBudW1iZXIgYW5kIG9yZGVyIElELgotIFJlcGVhdCBTdGVwIDMgYmFzZWQgb24gdGhlIG5ldyBpbmZvcm1hdGlvbi4KCkRldGVybWluZSBDdXN0b21lcidzIFJlcXVlc3Q6CjYuMS4gQXNrIHRoZSBjdXN0b21lciBpZiB0aGV5IHdvdWxkIGxpa2UgdG8gY2FuY2VsIHRoZSBvcmRlciBvciByZXF1ZXN0IGEgcmVmdW5kLgoKUHJvY2Vzc2luZyBDdXN0b21lcidzIFJlcXVlc3Q6CjcuMS4gQ2FuY2VsbGF0aW9uOgotIElmIHRoZSBjdXN0b21lciB3YW50cyB0byBjYW5jZWwgdGhlIG9yZGVyOgotIFVzZSB0aGUgY29tcGFueSdzIEFQSSB0byBjYW5jZWwgdGhlIG9yZGVyLgotIFVwb24gc3VjY2Vzc2Z1bCBjYW5jZWxsYXRpb24sIGluZm9ybSB0aGUgY3VzdG9tZXIgdGhhdCB0aGUgb3JkZXIgaGFzIGJlZW4gY2FuY2VsbGVkLgo3LjIuIFJlZnVuZDoKLSBJZiB0aGUgY3VzdG9tZXIgd2FudHMgYSByZWZ1bmQ6Ci0gVXNlIHRoZSBjb21wYW55J3MgQVBJIHRvIHByb2Nlc3MgdGhlIHJlZnVuZC4KLSBVcG9uIHN1Y2Nlc3NmdWwgcmVmdW5kLCBpbmZvcm0gdGhlIGN1c3RvbWVyIHRoYXQgdGhlIG9yZGVyIGhhcyBiZWVuIHJlZnVuZGVkLgoKRW5kIEludGVyYWN0aW9uOgo4LjEuIENvbmNsdWRlIGJ5IHRoYW5raW5nIHRoZSBjdXN0b21lciBmb3IgdGhlaXIgcGF0aWVuY2UgYW5kIGNvbmZpcm1pbmcgcmVzb2x1dGlvbi4=) <procedure> #  处理未收到订单的客户 开始互动: 1.1.  礼貌地向客户问好。 确认问题: 2.1.  确认客户未收到订单。 获取订单信息: 3.1.  请客户提供订单ID以及与订单相关的电子邮件地址或电话号码。 获取订单详情: 4.1.  如果客户提供了订单ID和电子邮件地址: -  使用公司的API根据电子邮件获取订单详情。 4.2.  如果客户提供了订单ID和电话号码: -  使用公司的API根据电话号码获取订单详情。 检查是否找到订单: 5.1.  如果找到了订单，继续步骤6。 5.2.  如果未找到订单: -  告知客户未找到订单。 -  请客户提供正确的电子邮件或电话号码以及订单ID。 -  根据新提供的信息重复步骤3。 确定客户的请求: 6.1.  询问客户是否希望取消订单或请求退款。 处理客户请求: 7.1.  取消订单: -  如果客户希望取消订单: -  使用公司的API取消订单。 -  取消成功后，通知客户订单已取消。 7.2.  退款: -  如果客户希望退款: -  使用公司的API处理退款。 -  退款成功后，通知客户订单已退款。 结束互动: 8.1.  感谢客户的耐心并确认问题已解决。</foreignobject></g></g></svg><svg class="ltx_picture" height="2449.57" id="A1.SS7.p2.pic1" overflow="visible"

### A.8 工具增强型 AI 代理

<svg class="ltx_picture" height="2663.36" id="A1.SS8.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,2663.36) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 2645.31)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">系统提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="2613.81" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你是一个客户支持代理，目标是回答用户的请求。你将获得以下信息：[⬇](data:text/plain;base64,LSBjb252ZXJzYXRpb246IE1lc3NhZ2VzIGV4Y2hhbmdlZCBiZXR3ZWVuIHRoZSBlbmQgdXNlciBhbmQgeW91LCBhbmQgdGhlIGV4ZWN1dGVkIGFjdGlvbnMgd2l0aCB0aGVpcgpvdXRwdXRzLg==) -  对话：用户与您的消息交换，以及执行的操作及其输出。这是你了解的程序：[⬇](data:text/plain;base64,PHByb2NlZHVyZT4Ke3sgcHJvY2VkdXJlIH19CjwvcHJvY2VkdXJlPg==) <procedure> {{  procedure  }} </procedure> 你只知道关于这个程序的答案！至关重要的是，你不能提出程序中没有包含的任何数据或指令。这是可用操作的列表。[⬇](data:text/plain;base64,PGFjdGlvbnM+Cnt7IGF2YWlsYWJsZV9hY3Rpb25zIH19CjwvYWN0aW9ucz4=) <actions> {{  available_actions  }} </actions> 有时，你的操作可能仅仅是回复用户，而其他时候你需要调用一个执行操作和/或获取必要数据的动作。一些操作需要信息/参数才能调用。如果你没有在上下文中获取到必要的信息，你必须请求它并且不能建议该操作。确保在建议相关操作之前遵循程序中的指令。例如，一些操作有后果，可能需要用户确认才能执行（如果程序中有说明）。如果是这种情况，请建议回复，要求用户确认。确保你使用的信息与上下文正确匹配（例如，用户可能提供一个与上下文中显示的不匹配的电话号码，后者包含了操作的输出）。你必须回复一个如下所示的 JSON 对象：[⬇](data:text/plain;base64,ewogICAgJ3R5cGUnOiBuYW1lIG9mIHRoZSBmdW5jdGlvbiB0byBjYWxsLAogICAgJ3BhcmFtZXRlcnMnOiBwYXJhbWV0ZXJzIHRvIHBhc3MgdG8gdGhlIGZ1bmN0aW9uLAp9) { ’type’:  name  of  the  function  to  call, ’parameters’:  parameters  to  pass  to  the  function, }</foreignobject></g></g></svg><svg class="ltx_picture" height="464.66" id="A1.SS8.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,464.66) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 446.61)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">用户提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="415.11" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">[⬇](data:text/plain;base64,PGNvbnZlcnNhdGlvbj4Ke3sgY29udmVyc2F0aW9uIH19CjwvY29udmVyc2F0aW9uPg==) <conversation> {{  conversation  }} </conversation></foreignobject></g></g></svg>

## 附录 B auto-ALMITA：详细评估

补充表格 [1](https://arxiv.org/html/2409.15934v2#A2.T1 "Table 1 ‣ Appendix B auto-ALMITA: Detailed evaluation ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")提供了使用 auto-ALMITA 数据集获得的详细结果，考虑了 6 个 LLM 代理及 [第 4.3 节](https://arxiv.org/html/2409.15934v2#S4.SS3 "4.3 Evaluation of LLM AI agents ‣ 4 Results ‣ Automated test generation to evaluate tool-augmented LLMs as conversational AI agents")中的所有评估指标。

| LLM | 回复 | API | 测试 | 对话 |
| --- | --- | --- | --- | --- |
| 召回率 | 正确 | 召回率 | 正确 | 正确参数 | 正确 | 正确 |
| GPT-4o | 91.1 | 77.1 | 89.5 | 95.1 | 84,4 | 85.4 | 14.7 |
| Mistral-NeMo-I | 89.2 | 67.5 | 89.5 | 93.8 | 80.7 | 81.3 | 10.3 |
| Claude3-s | 79.9 | 67.1 | 92.9 | 95.9 | 84.1 | 78.9 | 6.9 |
| GPT-4 | 60.5 | 82.9 | 92.6 | 94.6 | 84.5 | 75.5 | 6.4 |
| Llama3.1-8b-I | 79.4 | 61.8 | 64.3 | 95.7 | 83.8 | 73.4 | 3.2 |
| GPT-4o w/ F | 89.6 | 75.3 | 93.0 | 93.8 | 72.2 | 82.9 | 11.5 |

表 1：在 auto-ALMITA 上评估的 LLM AI 代理。对于每个 LLM，最高值以**粗体**显示。所有结果均为百分比。
