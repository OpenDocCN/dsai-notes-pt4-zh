<!--yml

类别：未分类

日期：2025-01-11 11:50:44

-->

# **代理型大型语言模型系统的实践考量**

> 来源：[https://arxiv.org/html/2412.04093/](https://arxiv.org/html/2412.04093/)

Chris Sypherd 爱丁堡大学 爱丁堡 英国 [c.n.sypherd@sms.ed.ac.uk](mailto:c.n.sypherd@sms.ed.ac.uk)  和 Vaishak Belle 爱丁堡大学 爱丁堡 英国 [vbelle@ed.ac.uk](mailto:vbelle@ed.ac.uk)

###### 摘要。

随着大型语言模型（LLM）在近年来的强大，越来越多的人开始关注将其作为自主智能体的基础模型进行使用。尽管LLM在自然语言领域展示了新兴能力和广泛的专业知识，但其固有的不可预测性使得LLM智能体的实现变得具有挑战性，导致相关研究与这些系统在现实世界中应用之间存在差距。为了弥合这一差距，本文在已建立的应用范式的背景下，提供了来自研究社区的可操作性洞见和考量，旨在促进鲁棒LLM智能体的构建并帮助在实际部署时做出明智决策。具体而言，我们将相关研究成果归纳为四大类——规划、记忆、工具和控制流程——这些类别基于应用导向文献中的常见做法，并强调在为现实应用设计代理型LLM时需要考虑的实际问题，例如如何处理随机性和高效地管理资源。虽然我们未进行实证评估，但我们提供了讨论代理型LLM设计中关键方面所需的背景，无论是在学术界还是工业界。

大型语言模型（LLMs），LLM智能体，代理型LLM，应用LLM系统

## 1\. 引言

在学术界，“智能体”这一概念已经得到了数十年的明确定义（例如，（Wooldridge和Jennings，[1995](https://arxiv.org/html/2412.04093v1#bib.bib95)）），因此基于LLM的智能体提出了预定义的标准和期望。因此，在研究社区中，代理型LLM被定义为具备信念（Sclar等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib72); Li等，[2023a](https://arxiv.org/html/2412.04093v1#bib.bib48); Park等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib63)）、推理（Zhang等，[2024c](https://arxiv.org/html/2412.04093v1#bib.bib106)）、规划（Huang等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib37); Shen等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib73)）和控制（Shen等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib73)）能力的自主系统。在这一定义下，规划、推理和与环境的互动已成为成功的关键考量因素（Xi等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib98)）。

对于工业界和实际部署中的LLM代理，代理的历史和范围已经被简化为类似于“一个可以使用LLM推理问题、创建解决问题的计划，并借助一套工具执行该计划的系统”的定义（Varshney，[2023](https://arxiv.org/html/2412.04093v1#bib.bib85)）。大多数行业讨论遵循这种形式，将LLM作为中央推理引擎，并添加规划、记忆和工具作为三个必要的模块（例如，（SAA，[2024](https://arxiv.org/html/2412.04093v1#bib.bib14)；PGA，[2024](https://arxiv.org/html/2412.04093v1#bib.bib13)；TFA，[2024](https://arxiv.org/html/2412.04093v1#bib.bib17)；Varshney，[2023](https://arxiv.org/html/2412.04093v1#bib.bib85)；Weng，[2023](https://arxiv.org/html/2412.04093v1#bib.bib94)））。事实上，大多数关注可部署代理LLM系统的行业资源都会附带一幅类似于图[1](https://arxiv.org/html/2412.04093v1#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ Practical Considerations for Agentic LLM Systems")的图，主要集中在单个代理上。虽然这种描述对于最基础的LLM代理是有帮助的，但它忽略了一些更微妙的考虑，若要构建稳健的代理LLM系统，这些考虑必须被充分理解。

当前工业界对LLM代理的主流观点揭示了（1）LLM和代理的研究与（2）LLM代理系统在实际场景中的应用之间的差距。为了弥合这一差距，我们建议将研究界的相关发现与工业界对LLM代理的普遍看法结合起来。为此，我们将本文组织成四个主要部分——规划、记忆、工具和控制流——它们分别对应上述提到的规划、记忆、工具和中央推理引擎组件。我们将本文内容专门针对典型的基于LLM的黑箱单代理系统，符合工业界的视角。通过这样做，我们希望创建一篇可操作且易于接近的调查报告，促进学术界与工业界之间在可控范围内的信息交流。

![请参阅说明文字](img/b0f3b1ce80da8d0efd5a06359f475274.png)

图1\. 一种典型的面向应用的LLM代理描绘。

\描述

一幅图展示了五个框，分别包含文本“用户输入”、“代理”、“规划”、“工具”和“记忆”。“代理”框位于“用户输入”和其他三个框之间。

## 2\. 相关工作

许多讨论基于LLM的代理的调查关注的是多代理框架及相关思想（Guo等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib33)；Zhang等，[2024c](https://arxiv.org/html/2412.04093v1#bib.bib106)）。尽管相似，但多代理系统所面临的挑战与单个LLM代理在实际部署中的情况有所不同。在这里，我们更多地关注如何有意识地打造一个稳健的代理，而不是如何协调多个代理。

另一种方法，由 (Yang et al., [2024](https://arxiv.org/html/2412.04093v1#bib.bib101)) 提出，聚焦于从底层模型开始改进代理 LLM 性能的方法，探讨数据组成和训练方法。我们则关注提高代理 LLM 系统性能的实施考虑，从黑箱视角出发，这更适用于实际部署。还有一些工作聚焦于创建统一的分类法  (Li, [2024](https://arxiv.org/html/2412.04093v1#bib.bib50); Xi et al., [2023](https://arxiv.org/html/2412.04093v1#bib.bib98))，或专注于 LLM 代理的单一组件，如规划 (Huang et al., [2024](https://arxiv.org/html/2412.04093v1#bib.bib37))。

最相似的工作是 (Wang et al., [2024c](https://arxiv.org/html/2412.04093v1#bib.bib88))，提供了一项全面的综述，涉及 LLM 作为代理的相关研究，并回顾了它们的设计、应用和评估方面。虽然 (Wang et al., [2024c](https://arxiv.org/html/2412.04093v1#bib.bib88)) 基于现有研究开发了一个有价值的统一框架，但我们利用研究成果提供了以实践应用为中心的洞见，并将我们的综述框定在业界自然发展起来的 LLM 代理范式中。

据我们所知，这是第一篇从常见行业实践的视角汇总与 LLM 代理相关研究的工作。我们通过不仅展示现有研究，还从中推导出可行的洞见和最佳实践，扩展了这一贡献。

## 3\. 应用场景

为了帮助阐明以下几点，我们提出了图 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 代理 LLM 系统的实践考虑") 中概述的示例，应用 LLM 代理作为鱼食主义¹¹1不吃肉，除了鱼类和其他海鲜的食物。餐助手。我们将在本工作中为一致性起见，称之为主要示例²²2虽然我们尽力选择一个与现实世界有一定关联的简单示例，但它仍然是一个人为的示例，用于展示本文中概述的要点，可能并不完全反映现实世界的复杂性。我们将在整篇工作中引用此示例，并通过图 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 代理 LLM 系统的实践考虑") 中分配的代码（例如，[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 代理 LLM 系统的实践考虑")）来引用具体内容，如”鱼食主义食谱书“。

![参考说明](img/7e4db6426895a9ed29f58036407c3825.png)

图 2\. 一个包含鱼食主义餐助手 LLM 代理的示例场景。

\描述

一张展示与鱼食主义餐助手示例相关的用户、任务和环境定义的图表。

## 4\. 术语表

本词汇表旨在简要介绍将在后续章节中使用的以下术语。后续章节将提供更多的上下文和它们的应用示例，但不一定提供明确的定义。

人物角色。人物角色（也称为“角色”或“个人档案”）是赋予LLM的身份，通常是作为系统提示的一部分。人物角色是LLM解读和回应提示的视角。例如，图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 代理式LLM系统的实际考虑")中展示的人物角色，可以通过职业（例如，“专业厨师”）、专业领域和水平（例如，“擅长素食海鲜菜肴”）、以及个性特征（例如，“友好且理解”）来定义和细化，还可以通过添加如年龄、种族、性别和国籍等细节进行进一步定制（Argyle等, [2023](https://arxiv.org/html/2412.04093v1#bib.bib18); Wang等, [2024c](https://arxiv.org/html/2412.04093v1#bib.bib88)）。

工具。工具是LLM与其环境进行互动（超越基本文本交换）并访问外部资源的手段（Qin等, [2024a](https://arxiv.org/html/2412.04093v1#bib.bib67)）。[检索增强生成](https://arxiv.org/html/2412.04093v1#S6.SS1 "在 6\. 内存 ‣ 代理式LLM系统的实际考虑")（RAG）通常作为工具使用（图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 代理式LLM系统的实际考虑")）。但它的效用是有限的，因为它仅能检索关于环境的信息。工具的真正力量体现在它们被用来在环境中执行操作时，比如图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 代理式LLM系统的实际考虑")所示的例子，这将使得素食餐助手不仅可以推荐食谱，还能为准备食谱订购所需的食材。工具的其他例子包括真实验证方法（例如，代码执行和计算器使用）以及实时环境查询（例如，查询流行食谱）（Wang等, [2024b](https://arxiv.org/html/2412.04093v1#bib.bib92)）。

超参数。本节简要概述了我们将提到的超参数，但并不探讨其技术细节³³3详见(Gem, [2024b](https://arxiv.org/html/2412.04093v1#bib.bib10); Ope, [[n.d.]](https://arxiv.org/html/2412.04093v1#bib.bib2); Ant, [2024a](https://arxiv.org/html/2412.04093v1#bib.bib7))，这些超参数常见的商业API支持。

+   •

    种子。一些LLM接口会有一个“种子”参数，只要其他所有参数保持不变，就应该生成相同的输出。

+   •

    温度。温度对应于在选择输出令牌时引入随机性的程度。通常，高温度响应会更加创造性和漫无目的，而低温度响应则更可预测并直截了当。因此，为了获得更一致的结果，可以使用较低的温度（例如0.0到0.5）。

+   •

    Top-p。Top-p（也称为“核心采样”；在 (Holtzman 等人，[2020](https://arxiv.org/html/2412.04093v1#bib.bib35))中介绍）对应于选择可以作为输出一部分的令牌的概率阈值，范围为0.0到1.0。较低的top-p值限制了LLM可以选择的令牌池，从而产生更可重复的输出。

## 5\. 规划

规划长期以来一直是代理人研究的核心组成部分 (Nau 等人，[2004](https://arxiv.org/html/2412.04093v1#bib.bib60); Wooldridge 和 Jennings，[1995](https://arxiv.org/html/2412.04093v1#bib.bib95))；它通过将更复杂的任务分解为更小、更易管理的步骤来处理问题。规划还可以增强LLM代理人的可解释性，因为计划的步骤和停止标准将以可解释的格式进行定义。

### 5.1\. LLMs 和 规划

尽管一些轶事应用显示LLM规划成功的迹象 (Song 等人，[2023](https://arxiv.org/html/2412.04093v1#bib.bib78))，但更多的整体性评审表明LLM是糟糕的规划者 (Valmeekam 等人，[2022](https://arxiv.org/html/2412.04093v1#bib.bib84)，[2024](https://arxiv.org/html/2412.04093v1#bib.bib83); Kambhampati，[2024](https://arxiv.org/html/2412.04093v1#bib.bib39); Liu 等人，[2023](https://arxiv.org/html/2412.04093v1#bib.bib53); Dagan 等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib26))。因此，如果要在具有一致任务的环境中部署LLM代理人，手动策划一个计划可以缓解LLM规划差的问题，并提供手动设计相关角色和提示的机会。另一种选择是通过外部规划工具增强LLM代理人，这已经显示出一定的前景 (Liu 等人，[2023](https://arxiv.org/html/2412.04093v1#bib.bib53); Dagan 等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib26))。由于LLM规划仍然是一个开放的研究领域，图 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems") 中的示例仅假设规划能力，而没有订阅特定的方法，仅用于说明目的。

为了描述当前 LLM 代理的规划方法，我们将其分为隐式规划和显式规划。对于隐式规划，一些代理将依赖 LLM 迭代地确定下一个立即步骤，直到任务完成，而无需提出一个计划（Zhang 等，[2024a](https://arxiv.org/html/2412.04093v1#bib.bib105)；Gur 等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib34)）。这种方法依赖于这样的理念：当提供一个最终目标时，LLM 可以保持一个内部计划，其步骤会被迭代地揭示，而无需任何显式的计划形式化。当环境动态变化或只有部分可观察时，这种方法是可行的，例如与网页交互（Gur 等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib34)）。另一种隐式规划形式是在单次推理中创建并执行一个计划，正如 Plan-and-Solve 提示策略所示（Wang 等，[2023b](https://arxiv.org/html/2412.04093v1#bib.bib89)），以及在一定程度上，零-shot Chain-of-Thought（Kojima 等，[2022](https://arxiv.org/html/2412.04093v1#bib.bib43)）。这些方法依赖于通过首先生成计划来条件化随后的令牌生成（即“执行”）。由于这种方法的单跳性质，它不适用于复杂任务，特别是那些在执行过程中需要反馈的任务。

显式规划的特点是将多步骤的计划明确地形式化，通常以多跳的方式执行。最基本的形式是简单地请求制定一个计划，然后执行它，正如最小到最大提示策略所示（Zhou 等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib108)）。更先进的方法首先制定计划，然后在执行步骤时迭代地完善计划（Liu 等，[2024a](https://arxiv.org/html/2412.04093v1#bib.bib56)）。这两者都需要长期规划，而这正是 LLM 往往表现不佳的地方（Valmeekam 等，[2022](https://arxiv.org/html/2412.04093v1#bib.bib84)，[2024](https://arxiv.org/html/2412.04093v1#bib.bib83)；Kambhampati，[2024](https://arxiv.org/html/2412.04093v1#bib.bib39)）。

### 5.2\. 任务分解

在制定 LLM 执行计划之前，了解 LLM 的局限性是很重要的。代理式 LLM 系统通常应用于单个 LLM 调用无法解决的问题，但一系列调用可以解决。任务通常可以分解成更小的部分，当单独解决这些部分时，可以重建并得出最终解决方案（Zhou 等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib108)）。

回到图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图2 ‣ 3\. 应用场景 ‣ 代理型LLM系统的实践考虑")，图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图2 ‣ 3\. 应用场景 ‣ 代理型LLM系统的实践考虑")中提出的请求I1由多个子任务组成，分别是：(1) 检索包含米饭、豆类和番茄的食谱，并且这些食谱是用户会喜欢的，以及(2) 订购缺失的任何食材。合理的做法是将(1)进一步分解为检索包含所需食材的食谱，并单独请求选择最符合用户口味的食谱。

如果手动分解一个明确定义的任务，逐步将任务分解为子任务并对LLM进行测试，可以为了解LLM能够一致处理哪些任务提供宝贵的见解。逻辑上分解问题足够简单，但确定LLM能够很好执行哪些任务，哪些任务需要进一步分解，可能是具有挑战性的，特别是在处理随机性和提示变化时。建议在此过程中频繁且系统地评估LLM代理，如[9.2](https://arxiv.org/html/2412.04093v1#S9.SS2 "9.2\. 评估 ‣ 9\. 附加考虑 ‣ 代理型LLM系统的实践考虑")一节中所讨论的那样。在开始时，可能更容易从任务的最基本构建块开始并将它们组合起来，而不是寻找可行任务的最小数量来启动。

尽管直观上可能认为任务越原子化越好，但事实并非总是如此。研究表明，LLM不仅具备在单个查询中解决多个不同任务的能力 (Xiong et al., [2024](https://arxiv.org/html/2412.04093v1#bib.bib99); Son et al., [2024](https://arxiv.org/html/2412.04093v1#bib.bib77); Laskar et al., [2023](https://arxiv.org/html/2412.04093v1#bib.bib45))，而且将多个任务组合成一个提示可以提高所有组成任务的表现，同时减少整体上下文的使用量 (Son et al., [2024](https://arxiv.org/html/2412.04093v1#bib.bib77))。然而，任务能够组合的程度应当成为针对特定任务和环境进行严格实验的课题。

### 5.3\. 计划遵循

LLM 代理的责任之一是监督计划的执行。它应该决定是否需要为给定输入重复某个步骤（例如，[错误处理](https://arxiv.org/html/2412.04093v1#S8.SS2 "在 8\. 控制流程 ‣ 代理式 LLM 系统的实际考虑")）或跳过某个步骤（例如，为了对计划进行迭代（Liu 等， [2024a](https://arxiv.org/html/2412.04093v1#bib.bib56)））。作为计划者的 LLM 主要的关注点之一是它们无法判断是否能够完成给定任务（Kambhampati，[2024](https://arxiv.org/html/2412.04093v1#bib.bib39)）。因此，LLM 代理通常无法知道某个步骤是否会成功，直到尝试执行该步骤。因此，逻辑上应该在执行后评估每个步骤的成功与否。同样，整个计划的成功与否也应该在所有步骤完成后进行评估。如果计划不成功，LLM 代理可能需要根据每个步骤的结果和整体计划调整或重新执行计划（详见第 [8.2](https://arxiv.org/html/2412.04093v1#S8.SS2 "8.2\. 错误处理 ‣ 8\. 控制流程 ‣ 代理式 LLM 系统的实际考虑")节，讨论如何结合反馈）。

## 6\. 内存

### 6.1\. 检索增强生成

检索增强生成（RAG）（在（Lewis 等，[2020](https://arxiv.org/html/2412.04093v1#bib.bib47)）中首次提出）已经成为工业界代理式 LLM 应用的核心（Gao 等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib32)）。其基本原理很简单：一个能够提供与自然语言输入相关的外部上下文的系统。通常，传入的输入会与一个真实数据存储进行比较，然后将最相关的信息片段作为上下文提供给 LLM，LLM 将基于这些信息作出回应。这可以通过隐式方式完成，即用户的输入始终用于检索某个 LLM 调用；也可以通过显式方式完成，即 LLM 将 RAG 作为工具使用。这对 LLM 系统有许多好处：

+   •

    基础化。与其依赖 LLM 正确“记住”训练数据中的相关上下文，不如直接为 LLM 提供准确的相关信息。作为上下文提供基础化文本显著减少了 LLM 的幻觉现象，并填补了训练数据中的知识空白（Shuster 等，[2021](https://arxiv.org/html/2412.04093v1#bib.bib75)；Es 等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib31)；Lewis 等，[2020](https://arxiv.org/html/2412.04093v1#bib.bib47)）。

+   •

    可解释性。与其依赖 LLM 模糊地引用其训练中获得的信息，不如遵循作为 RAG 一部分提供的上下文，这样可以清晰地了解 LLM 获取信息的具体来源（Gao 等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib32)）。

+   •

    时效性。虽然LLMs可以引用来自静态训练数据的信息，但LLM会受到硬性信息截止日期（即训练数据抓取的最晚日期）和软性信息截止日期（即接近硬性信息截止日期且覆盖有限的事件）的限制。与其转向重新训练并使用更新数据这一不可行的前景，我们可以提供与查询相关的更新信息作为上下文（Gao等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib32)）。

+   •

    外包。根据RAG数据库的内容、质量和可靠性，查询的某些方面可以隐式地外包给返回的上下文，例如推理和决策。

+   •

    对齐。用于大规模语言模型（LLMs）的大量训练数据是其自然语言理解的来源，但不应完全依赖于此以生成无偏见、值得信赖且安全的内容。通常，将LLM的输出与人类偏好对齐被视为一个数据收集和训练问题（Wang等，[2023c](https://arxiv.org/html/2412.04093v1#bib.bib91)；Bai等，[2022](https://arxiv.org/html/2412.04093v1#bib.bib20)），但也可以通过RAG后处理来解决。通过利用从符合人类偏好期望集的更精炼数据集获取的上下文，增强LLM的自然语言能力和倾向，其输出可以被引导以符合期望的内容和态度。这需要仔细策划数据存储，但它是一种可行的黑箱对齐方法。

为了说明上述观点，请参见图 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems") 中引用的 RAG 来源。R1 和 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems")。R2。图 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems")。R1 可以被引用以避免配方幻觉（基础验证）并用新配方进行更新（时效性）。图 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems")。R2 可以在回应图 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems")。I2 时派上用场；在没有普遍共识的情况下，我们可以提供自己的基础事实，而不是要求 LLM 回答一个潜在的道德问题（外包）。图 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems")。R1 和 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems")。R2 的语气和术语将引导 LLM 使用的理想、内容和术语（对齐）。

RAG 有两种主要方法：知识图谱（Edge 等， [2024](https://arxiv.org/html/2412.04093v1#bib.bib30)）和向量数据库（Gao 等， [2024](https://arxiv.org/html/2412.04093v1#bib.bib32)；Barron 等， [2024](https://arxiv.org/html/2412.04093v1#bib.bib21)），后者因其简便性而被广泛采用。关于 RAG 实施及现有的商业和开源产品的讨论，见（Gao 等， [2024](https://arxiv.org/html/2412.04093v1#bib.bib32)）。

### 6.2\. 长期记忆

有时，在对话中获得的关键信息可能对所有上下文都有帮助，例如有用的外部知识或关于用户或任务的信息。在这种情况下，将该信息以一种可以被代理访问的方式存储可能是有利的，以便它的影响不局限于当前的上下文。这通常被称为“长期记忆”（Qian等，[2024a](https://arxiv.org/html/2412.04093v1#bib.bib65)；Wang等，[2024c](https://arxiv.org/html/2412.04093v1#bib.bib88)；Zhong等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib107))⁴⁴4长期记忆的现实世界实现是OpenAI的“记忆”（OAI，[2024](https://arxiv.org/html/2412.04093v1#bib.bib15)）。在与ChatGPT的对话中，LLM会保存其认为特别有用的信息。这些信息随后将在未来的对话中提供给LLM。这带来的一个明显好处是，它减少了用户的重复性操作，并且让LLM能更好地实现提供相关回复的目标。

我们希望对存储在长期记忆中的信息进行选择，以确保其一般有用且不过于庞大。一些常见的方法是存储先前的查询解决方案（Qian等，[2024a](https://arxiv.org/html/2412.04093v1#bib.bib65)），全局总结和洞察（Zhong等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib107)），以及获得的工具（Wang等，[2023a](https://arxiv.org/html/2412.04093v1#bib.bib87)）。

通过反思、巩固、遗忘、修订以及其他旨在模仿人类长期记忆的机制，长期记忆可以得到增强（有关高级长期记忆实现的讨论，请参见（Zhong等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib107)））。为了简化，我们专注于长期记忆的简单版本，其中信息只是被存储和检索，任何编辑都需要手动进行。对于这个简单的变体，我们从现有的LLM代理长期记忆文献中得出以下三个标准（Qian等，[2024a](https://arxiv.org/html/2412.04093v1#bib.bib65)；Wang等，[2024c](https://arxiv.org/html/2412.04093v1#bib.bib88)；Zhong等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib107)；OAI，[2024](https://arxiv.org/html/2412.04093v1#bib.bib15)；Wang等，[2023a](https://arxiv.org/html/2412.04093v1#bib.bib87)）作为应该存储信息的试金石：

+   •

    独立性。信息应该没有任何隐式依赖，例如输入值。

+   •

    与一致性相关。信息应与代理系统中的一致性相关，这可能包括任务、用户或环境。

+   •

    适用长期性。信息应始终适用于LLM代理可能接触的上下文。

请参见表 [1](https://arxiv.org/html/2412.04093v1#S6.T1 "表 1 ‣ 6.2\. 长期记忆 ‣ 6\. 记忆 ‣ 代理型大语言模型系统的实际考虑")，其中应用了以上标准，并结合图 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 代理型大语言模型系统的实际考虑") 中的示例。

如果确保了上述三个标准，那么收集到的上下文信息可以成为提示改进的有用起点。它将是那些被识别为通常和持续对 LLM 代理的环境有用的信息，可能适合永久性地包含在用户或系统的提示中。进行提示、任务、角色、超参数或模型更新时，审查长期记忆也非常有价值；例如，重新排序 LLM 调用，或调整工具功能时，因为这些变化可能会影响三个标准的有效性。

表 1\. 根据存储上下文信息的三个标准，分析图 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 代理型大语言模型系统的实际考虑") 中的信息。

| 信息 | 独立性 | 与一致性相关 | 适用于长期 | 存储信息 |
| --- | --- | --- | --- | --- |
| E1 玉米不再可用 | 是 | 是 | 是 | 是 |
| E2 家禽类食品不再可用 | 是 | 否 | 否 | 否 |
| U2 对坚果过敏 | 是 | 是 | 是 | 是 |
| I1 我有米饭、豆类和西红柿…… | 否 | 是 | 否 | 否 |

#### 6.2.1\. 提取长期记忆信息

提取属于长期记忆的信息的常见方法是利用外部对话主持人 (Zhong et al., [2024](https://arxiv.org/html/2412.04093v1#bib.bib107); Shinn et al., [2024](https://arxiv.org/html/2412.04093v1#bib.bib74))。外部主持人（例如，具有独立角色的大语言模型）会审查对话（完整对话或片段），并可以被任务化提取其认为符合上述三个标准的信息。这是一个需要注意措辞的实例，因为任务的主观性可能使得大语言模型在响应时容易受到框架偏见的影响（例如，如果我们问是否有任何有用的信息需要提取，大语言模型很可能会提取一些信息）（Echterhoff et al., [2024](https://arxiv.org/html/2412.04093v1#bib.bib29)）。

#### 6.2.2\. 存储长期记忆

一旦某一信息被认为值得存入长期记忆，它就应该被存储。某些方法包括将信息嵌入并存储在向量数据库中（类似于[RAG](https://arxiv.org/html/2412.04093v1#S6.SS1 "6.1\. Retrieval Augmented Generation ‣ 6\. Memory ‣ Practical Considerations for Agentic LLM Systems")）（Zhong 等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib107)；Lin 等人，[2023](https://arxiv.org/html/2412.04093v1#bib.bib52)），以及自然语言存储，尽管随着长期记忆量的增加，与后者的交互很快会变得不方便。向量数据库的结构使得我们可以轻松查询相关信息（Lin 等人，[2023](https://arxiv.org/html/2412.04093v1#bib.bib52)；Wang 等人，[2023a](https://arxiv.org/html/2412.04093v1#bib.bib87)；Zhong 等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib107)）。

#### 6.2.3\. 利用长期记忆

一旦信息被存储到长期记忆中，我们必须决定何时将其暴露给LLM代理。理解当前范围内哪些信息是相关的至关重要。LLM代理可能由许多不同目的和背景的LLM调用组成；并非所有长期记忆中的信息都会应用于每一次LLM调用。例如，如果图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems")中的用户决定每周规划一次餐单（如图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").I5所示），这对图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").Pe1来说是有价值的，但不一定对[2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").Pe2有价值，因为后者主要用于其烹饪专业知识。在这种情况下，通过向量数据库检索得到的相关性是有价值的（Lin 等人，[2023](https://arxiv.org/html/2412.04093v1#bib.bib52)；Wang 等人，[2023a](https://arxiv.org/html/2412.04093v1#bib.bib87)；Zhong 等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib107)）。一旦从长期记忆中检索到相关信息，它可以通过用户或系统提示在LLM调用中共享。

## 7\. 工具

### 7.1\. 使用工具

要使LLM能够使用工具，需要与LLM一起公开工具描述和调用方法（类似于传统的软件工程文档）。如果使用的工具数量较少，可以用自然语言进行介绍。调用工具的方法应该清晰且易于解析。常见的做法是定义JSON架构或函数签名，尽管后者被证明更适合LLM代理（Roucher和Petrov，[2024](https://arxiv.org/html/2412.04093v1#bib.bib69)；Wang等，[2024a](https://arxiv.org/html/2412.04093v1#bib.bib90)）。

工具可以通过显式或隐式调用，前者在实践中是事实上的常用方法。显式使用仅仅意味着在LLM代理的输出中调用工具（Schick等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib71)；Qin等，[2024b](https://arxiv.org/html/2412.04093v1#bib.bib68)）。一旦工具被定义并作为上下文传递，代理就能以指定的可解析格式执行这样的调用。工具也可以通过实现者在响应LLM代理的行动或无动作时隐式调用。例如，如果发生角色之间的转换，系统可能会始终受益于对前面对话的总结。与其依赖LLM代理在每次角色变化时调用总结，不如让每次转换都在后台触发一次总结。有关隐式调用工具的讨论，请参见[9.3节](https://arxiv.org/html/2412.04093v1#S9.SS3 "9.3\. 集成与传统工程 ‣ 9\. 额外考虑 ‣ 代理LLM系统的实践考虑")。

### 7.2. 管理多样性

随着工具数量的增加，用自然语言定义工具很快会变得难以管理，这时需要采用结构化的方法。为此，我们可以利用LLM对代码的便捷理解，通过结合JSON架构或函数签名以及简洁的自然语言描述来创建更简明的工具定义⁵⁵5有关工具定义的讨论，请参见[https://python.langchain.com/docs/concepts/#tools](https://python.langchain.com/docs/concepts/#tools)，以及(PCo, [[n.d.]](https://arxiv.org/html/2412.04093v1#bib.bib3))的实现。

通常，基于相似的核心功能（即，如果它们可以合理地看作是继承自相同的基类）可以将不同的工具分组。这些组可以称为“工具集”或“工具包”⁶⁶6请参见 https://python.langchain.com/docs/concepts/#toolkits。这有助于确定工具是否可以通过单一接口组合，或者是否可以在提示中一起引入。例如，在示例中介绍的工具 Figure [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").T1 和 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").T2 不属于同一工具集，但 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").T2 和 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").T3 会属于同一工具集。

### 7.3\. 动态添加工具

有时，代理 LLM 系统将要部署的环境中的工具并不事先已知。在这种情况下，我们可以将“工具识别”作为系统的一个任务（Schick 等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib71); Wang 等，[2023a](https://arxiv.org/html/2412.04093v1#bib.bib87)）。一个引人注目的示例和实现可以在 Voyager 论文中看到，其中基于 LLM 的代理在 Minecraft 的世界中自主穿行⁷⁷7https://www.minecraft.net，并根据与环境的交互动态地组装一组工具，这些工具随后存储在长期记忆中（Wang 等，[2023a](https://arxiv.org/html/2412.04093v1#bib.bib87)）。

## 8\. 控制流

在 LLM 代理的上下文中，控制流指的是确定为了回应查询需要做什么的能力。给 LLM 任务控制流使得基于 LLM 的代理能够完成那些单次推理无法解决的复杂任务。这赋予了 LLM 代理自主性，可以根据需要融入诸如规划、工具使用和多步推理等先进技术（Shen 等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib73); Wang 等，[2024c](https://arxiv.org/html/2412.04093v1#bib.bib88)）。

实际上，这可能表现为LLM代理接收用户输入（即，观察环境）并选择立即采取的下一步行动。代理会继续采取行动，直到决定停止。为了使这一过程可行，LLM代理需要了解其行动空间（Yao等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib103)），例如停止标准、可用工具（例如，图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 智能 LLM 系统的实际考虑")。T1，[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 智能 LLM 系统的实际考虑")。T2，和[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 智能 LLM 系统的实际考虑")。T3)，可用的规划选项（[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 智能 LLM 系统的实际考虑")。P1），能够进行思维外化（Yao等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib103)），以及使用其他角色（例如，[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 智能 LLM 系统的实际考虑")。Pe2）。

考虑一个接收图示[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 智能 LLM 系统的实际考虑")的LLM代理。I1。代理不仅仅提供输出，它还可以选择利用[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 智能 LLM 系统的实际考虑")。P1，规划模块，来分解复杂的任务并生成一个多步骤的计划，然后执行该计划。一旦计划完成，代理可以决定是否有足够的信息提供最终输出给[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 智能 LLM 系统的实际考虑")。I1，或者它是否需要采取额外的行动。

在这里，我们介绍了一些实际考虑，确保LLM代理能够与其环境顺畅且不断裂地交互。

### 8.1 输出处理

在将多个 LLM 输入和输出串联在一起时，通常先对文本进行处理再传递给下一步是有利的。虽然自然语言是人类可读的，但建议使用更结构化的格式（如 JSON 或可执行代码），以便更容易解析（Wang 等，[2024a](https://arxiv.org/html/2412.04093v1#bib.bib90)）。虽然较弱的模型可能在跟随指令时遇到困难，但大多数商业模型已经优化了以遵循用户或系统提示中指定的输出格式⁸⁸8常见商业模型的输出处理文档：Anthropic（Ant，[[n.d.]](https://arxiv.org/html/2412.04093v1#bib.bib4)）；OpenAI（OAI，[[n.d.]b](https://arxiv.org/html/2412.04093v1#bib.bib6)）；Google（Gem，[2024c](https://arxiv.org/html/2412.04093v1#bib.bib11)）。

由于我们从黑箱的角度来研究 LLM，因此不会讨论约束 LLM 输出特定格式的底层方法。然而，需要注意的是，LLM 展示出的推理能力可能会受到约束生成的负面（且无意的）影响，这取决于具体的实现方式（Beurer-Kellner 等， [2024](https://arxiv.org/html/2412.04093v1#bib.bib23)；Tam 等， [2024](https://arxiv.org/html/2412.04093v1#bib.bib80)）。因此，研究表明，要求输出代码而非特定结构，可以生成更好的代理（Wang 等，[2024a](https://arxiv.org/html/2412.04093v1#bib.bib90)）。

### 8.2\. 错误处理

错误处理是构建强大代理 LLM 系统中最重要但又最难捉摸的部分之一。由于 LLM 本质上是随机的，将多个 LLM 调用串联在一起会增加失败的风险，尤其是对于长序列几乎不可避免。因此，代理 LLM 系统中的每个 LLM 调用都应该被视为潜在的失败点，并通过适当的错误处理来支持。

我们提供图[3](https://arxiv.org/html/2412.04093v1#S8.F3 "图 3 ‣ 8.2\. 错误处理 ‣ 8\. 控制流 ‣ 智能 LLM 系统的实际考虑")来展示几种错误处理方法，其中会通过图中分配的代码（例如， [3](https://arxiv.org/html/2412.04093v1#S8.F3 "图 3 ‣ 8.2\. 错误处理 ‣ 8\. 控制流 ‣ 智能 LLM 系统的实际考虑")的UI，引用“为我点一个洋葱。”）来参考具体的响应。为简化起见，系统提示（包含角色和工具信息）被省略。

![参考标题](img/7e5c70573b95471b818bb0e7aaf3e964.png)

图 3\. 错误工具调用的示例，参照图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图 2 ‣ 3\. 应用场景 ‣ 智能 LLM 系统的实际考虑")中的场景。

\说明

显示代理出现错误工具调用的图示。

#### 8.2.1\. 静态重试

处理问题输出的最简单方法是用相同的提示重新调用LLM。虽然其他[超参数](https://arxiv.org/html/2412.04093v1#S4 "4\. Glossary ‣ Practical Considerations for Agentic LLM Systems")可能保持不变，但每次静态重试时种子应始终更改，以避免完全重复的调用。如果使用低温度或高top-p值，则也可以适当调整这些值，以便获得不同的输出。在图 [3](https://arxiv.org/html/2412.04093v1#S8.F3 "Figure 3 ‣ 8.2\. Error Handling ‣ 8\. Control Flow ‣ Practical Considerations for Agentic LLM Systems")的上下文中，这可能表现为仅用不同的种子重新运行[3](https://arxiv.org/html/2412.04093v1#S8.F3 "Figure 3 ‣ 8.2\. Error Handling ‣ 8\. Control Flow ‣ Practical Considerations for Agentic LLM Systems").UI。

对于那些输出容易验证的低上下文调用（例如，将输出解析为JSON对象），如果验证失败，进行一些静态重试是一种简单而有价值的补充。对于那些更难验证的输出，例如下游解释的自然语言指令，静态重试的效果较差，因为验证成本较高。

#### 8.2.2\. 知情重试

一种更为知情的方法是将LLM的输出附加到历史记录中，添加另一条用户消息，表明该输出未成功，并再次尝试。这应当辅以具体的错误信息或额外的指令（Kamoi 等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib40); Tyen 等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib82)）。在图 [3](https://arxiv.org/html/2412.04093v1#S8.F3 "Figure 3 ‣ 8.2\. Error Handling ‣ 8\. Control Flow ‣ Practical Considerations for Agentic LLM Systems") 示例中，知情重试可能如下所示：发送以下消息列表：[3](https://arxiv.org/html/2412.04093v1#S8.F3 "Figure 3 ‣ 8.2\. Error Handling ‣ 8\. Control Flow ‣ Practical Considerations for Agentic LLM Systems").UI, [3](https://arxiv.org/html/2412.04093v1#S8.F3 "Figure 3 ‣ 8.2\. Error Handling ‣ 8\. Control Flow ‣ Practical Considerations for Agentic LLM Systems").AO, [3](https://arxiv.org/html/2412.04093v1#S8.F3 "Figure 3 ‣ 8.2\. Error Handling ‣ 8\. Control Flow ‣ Practical Considerations for Agentic LLM Systems").TR，以及“尝试上述代码时出现了提供的错误。请提供更新后的输出，以实现最初的指令。”。

#### 8.2.3\. 外部重试

与其在同一上下文中请求知情重试，不如将历史记录中的部分内容提取出来，并以单独的上下文提供给LLM，以修正先前的输出或生成新的输出。这通常需要来自原始调用的显著上下文，但可以通过使用不同的角色、不同的指令和错误信息来补充和区分。通常，将角色从软件工程师转变为代码审阅员可以为LLM提供修复或生成正确输出的动力。尽管已有研究表明，外部基于LLM的错误系统的解释通常不可靠，并且对提示变化敏感（Kamoi等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib40)），但访问特定任务的角色、详细的错误信息（例如由生成的Python代码引发的错误）以及背景上下文有助于减轻这些问题（Tyen等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib82)）。

在图示[3](https://arxiv.org/html/2412.04093v1#S8.F3 "Figure 3 ‣ 8.2\. Error Handling ‣ 8\. Control Flow ‣ Practical Considerations for Agentic LLM Systems")的示例中，外部重试可能表现为发送以下消息列表：“你是一个专家级调试员，你可以访问{tool information}。”和“在尝试执行请求时，‘{[3](https://arxiv.org/html/2412.04093v1#S8.F3 "Figure 3 ‣ 8.2\. Error Handling ‣ 8\. Control Flow ‣ Practical Considerations for Agentic LLM Systems").UI}’，助手尝试运行代码‘{[3](https://arxiv.org/html/2412.04093v1#S8.F3 "Figure 3 ‣ 8.2\. Error Handling ‣ 8\. Control Flow ‣ Practical Considerations for Agentic LLM Systems").AO}’，结果为‘{[3](https://arxiv.org/html/2412.04093v1#S8.F3 "Figure 3 ‣ 8.2\. Error Handling ‣ 8\. Control Flow ‣ Practical Considerations for Agentic LLM Systems").TR}’。请提供一个更新的输出，以实现最初的指令。”。

应该注意，LLM在定位错误方面存在困难（Kamoi等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib40)；Tyen等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib82)），但如果提供足够的上下文，特别是错误位置，它们表现出强大的错误修正能力（Tyen等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib82)）。因此，在使用[知情重试](https://arxiv.org/html/2412.04093v1#S8.SS2.SSS2 "In 8.2\. Error Handling ‣ 8\. Control Flow ‣ Practical Considerations for Agentic LLM Systems")或[外部重试](https://arxiv.org/html/2412.04093v1#S8.SS2.SSS3 "In 8.2\. Error Handling ‣ 8\. Control Flow ‣ Practical Considerations for Agentic LLM Systems")时，应注意包括能准确定位错误来源的错误信息，例如来自API和运行时环境的诊断错误信息和堆栈跟踪。

### 8.3\. 停止

由于智能体LLM系统的控制流程由LLM控制，因此需要定义一个明确的停止方法。这通常表现为插入系统提示中的预定停止标记或短语，例如“TERMINATE”（Wu 等, [2024a](https://arxiv.org/html/2412.04093v1#bib.bib96)）。它应当是一个易于解析的标记或短语，并且不会轻易出现，以避免意外停止。

### 8.4\. 多重角色

通常，分配给大规模语言模型（LLM）的角色对其在特定任务中的表现有显著影响。这在LLM文献中普遍被观察到，已成为有效提示的重要组成部分（Karmaker Santu 和 Feng, [2023](https://arxiv.org/html/2412.04093v1#bib.bib42)；Kong 等, [2024](https://arxiv.org/html/2412.04093v1#bib.bib44)），并且在近期的LLM多智能体研究中，已成为许多此类架构中智能体多样性的必要组成部分（Hong 等, [2024](https://arxiv.org/html/2412.04093v1#bib.bib36)；Wu 等, [2024a](https://arxiv.org/html/2412.04093v1#bib.bib96)；Li 等, [2023b](https://arxiv.org/html/2412.04093v1#bib.bib51)；Guo 等, [2024](https://arxiv.org/html/2412.04093v1#bib.bib33)；Wang 等, [2024d](https://arxiv.org/html/2412.04093v1#bib.bib93)；Park 等, [2023](https://arxiv.org/html/2412.04093v1#bib.bib63)）。例如，虽然Figure [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").Pe1角色适合回答大部分用户查询，但Figure [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").Pe2角色可能更适合回答Figure [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").I4，因为该任务需要专业的烹饪知识。

由于智能体LLM系统中可能包含许多不同的任务，因此通常需要使用多个角色。一种定义LLM角色或“角色建模”的方法概述详细列出（Wang 等, [2024c](https://arxiv.org/html/2412.04093v1#bib.bib88)），将其分类为手工制作（例如，(Qian 等, [2024a](https://arxiv.org/html/2412.04093v1#bib.bib65)；Park 等, [2023](https://arxiv.org/html/2412.04093v1#bib.bib63)）），LLM生成（例如，(Wang 等, [2024d](https://arxiv.org/html/2412.04093v1#bib.bib93)；Xu 等, [2023](https://arxiv.org/html/2412.04093v1#bib.bib100)））或数据集对齐（即，来源于相关数据集）。这些角色应根据所处理任务进行定义。这取决于智能体LLM系统的整体上下文，但通常可以通过以下方式解决：

+   •

    如果任务已明确规定，为每个任务手工制作专业角色（例如，图 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").Pe1 和 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").Pe2)）。

+   •

    如果任务未明确规定，但通常与单一主题相关，使用该主题的最具体手工制作角色（例如，通用图 [2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").Pe1）。

+   •

    如果任务在开始时确实未定义（例如，帮助处理任何事务的助手）或主题非常广泛：

    +   –

        定义多个不同的角色，LLM 代理可以根据需要将后续调用路由到这些角色（Si 等， [2023](https://arxiv.org/html/2412.04093v1#bib.bib76)）。一旦代理开始使用，可以根据最常用的角色定义更为细化的人物设定。这也可以看作是数据集对齐方法（Wang 等， [2024c](https://arxiv.org/html/2412.04093v1#bib.bib88)），在该方法中，数据集是在特定环境下根据临时设定的人物设定构建的。

    +   –

        利用 LLM 创建它认为最适合响应提示的角色（Wang 等， [2024d](https://arxiv.org/html/2412.04093v1#bib.bib93)；Xu 等， [2023](https://arxiv.org/html/2412.04093v1#bib.bib100)）。这种方法更为昂贵，因为生成角色需要使用 LLM，但无疑对于应对不可预见的情况更为稳健。此方法可以与上述方法结合使用（例如，如果没有找到合适的预定义角色，则创建一个）。

### 8.5\. 管理相关上下文

管理发送到LLM的上下文是提高LLM系统效率（速度和成本）和性能的有效方法，因为推理时间取决于输入标记的数量（Vaswani等， [2017](https://arxiv.org/html/2412.04093v1#bib.bib86)；Pope等， [2023](https://arxiv.org/html/2412.04093v1#bib.bib64)），而且LLM在长上下文场景下表现较差，尤其是在复杂任务中（Li等， [2024](https://arxiv.org/html/2412.04093v1#bib.bib49)；Liu等， [2024b](https://arxiv.org/html/2412.04093v1#bib.bib54)）。此外，由于LLM具有有限的上下文窗口⁹⁹9例如，Claude 3：200k（Ant， [2024b](https://arxiv.org/html/2412.04093v1#bib.bib16)）；Gemini 1.5 Pro：2M+（Gem， [2024a](https://arxiv.org/html/2412.04093v1#bib.bib9)）；GPT-4o：128k（OAI， [[n.d.]a](https://arxiv.org/html/2412.04093v1#bib.bib5)），精心的上下文管理变得尤为必要。即便是“长”上下文LLM（¿100k 标记限制），若管理不当，许多任务也会迅速变得难以应对（例如，处理HTML时，单个网页可能包含数十万个标记）。这是在进行[任务分解](https://arxiv.org/html/2412.04093v1#S5.SS2 "5.2\. Task Decomposition ‣ 5\. Planning ‣ Practical Considerations for Agentic LLM Systems")时必须考虑的关键事项；任务越具体，越多的额外上下文（例如，先前的消息）可以被裁剪掉（Qian等， [2024b](https://arxiv.org/html/2412.04093v1#bib.bib66)）。因此，特定LLM调用接收到的上下文应尽可能根据任务进行量身定制。即使LLM调用需要过去的消息，通常也可以去除某些上下文部分或对其进行总结，只保留后续调用依赖的部分，且保持整体意义不变。可以在调用之间对上下文进行显著调整，以减少总体标记数并去除多余的上下文，从而减少LLM的混淆并提高LLM调用的性能（Qian等， [2024b](https://arxiv.org/html/2412.04093v1#bib.bib66)）。

## 9\. 额外考虑事项

### 9.1\. 模型大小

通常，选择使用的模型大小由三个因素决定：成本、速度和性能。通常情况下，模型越大，成本越高，速度越慢，性能越好（尽管这不是一个硬性规定）。围绕最弱的模型构建一个具有代理功能的LLM系统，以便从一开始就优化所有三个条件，可能是很有诱惑力的。然而，试图首先从较小的模型开始构建一个可行的系统，可能比从最强的模型开始，等LLM代理在环境中展现出能力后，再针对特定调用降级使用的模型更加耗时且昂贵。由于一次调用可能对后续调用产生影响，如果不是所有的组件都在最佳状态下运行，那么要理解给定用例中可能的情况是不现实的。通过从更强的模型开始，将有一个黄金标准的基准可供比较，这样就能衡量降级模型在特定调用中的性能影响^(10)^(10)10参见（Benram，[2024](https://arxiv.org/html/2412.04093v1#bib.bib22)；ljunkai，[2023](https://arxiv.org/html/2412.04093v1#bib.bib57)）从行业角度讨论这些观点.. 建议根据每个任务选择合适的模型，并在单独评估和作为整个代理LLM系统的一部分的背景下进行评估。

### 9.2\. 评估

评估一个代理LLM系统可能是具有挑战性的，因为可能涉及长序列、LLM中的非确定性、与外部实体的互动以及可能没有明显正确解答的任务。尽管如此，在部署之前，定义一种评估方法仍然至关重要，以便（1）有一个基准可以进行对比，并且（2）能够衡量随着时间推移和响应变化而发生的性能变化。

在为评估LLM代理创建数据集时，最重要的考虑因素是它能够准确地模拟将要部署的环境。许多针对特定领域的LLM代理基准测试已经可用（Deng等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib27)；Yao等人，[2022](https://arxiv.org/html/2412.04093v1#bib.bib102)；Liu等人，[2024c](https://arxiv.org/html/2412.04093v1#bib.bib55)；Zhang等人，[2024b](https://arxiv.org/html/2412.04093v1#bib.bib104)），以及通用应用基准（Srivastava等人，[2023](https://arxiv.org/html/2412.04093v1#bib.bib79)；Wu等人，[2024b](https://arxiv.org/html/2412.04093v1#bib.bib97)；Mialon等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib59)），但是许多应用于特定任务的代理LLM系统将过于专业，无法从更广泛的基准中受益。然而，只要一个已建立的基准适用于LLM系统的应用，它可以成为评估和改进的有力起点。无论是否使用现有的基准，都建议在部署环境中收集有用的代理交互数据（例如，长序列、短序列、错误输出、正确输出等）及相关元数据（例如，超参数）。这样做将能够创建一个由可复现的输入和输出对组成的数据集，这些数据集来源于该环境。即使数据集样本较少，它也能提供一个基准，用于进行对比，确保提示工程能够解决失败的执行，识别模型和提示变更的影响，并避免系统回归^(11)^(11)11请参见（Lan，[2024](https://arxiv.org/html/2412.04093v1#bib.bib8)）了解评估已部署LLM系统的行业方法..

虽然传统指标（例如精准度、召回率等）在追踪过程中非常有用，但针对代理的特定指标可以帮助揭示高层次指标未能反映的系统变化（Chang等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib25)；Kapoor等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib41)）。例如，当给定两个不同的提示时，LLM代理如果得出相同的答案，表面上看是保持一致的，但达到这个结论所需的中间步骤数量的差异可能表明该系统对提示变化过于敏感。从(Liu等人，[2023](https://arxiv.org/html/2412.04093v1#bib.bib53)；Kapoor等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib41)；Mehta等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib58))的研究中可以找到替代评估方法的建议，我们在下面提供了一些示例指标作为起点，尽管有用的指标应根据LLM代理的设计以及其所处的环境来选择^(12)^(12)12请注意，以下指标主要侧重于评估代理型LLM系统，但外部组件也应进行评估，如RAG系统（例如，检索质量和嵌入文档的忠实度）（Salemi和Zamani，[2024](https://arxiv.org/html/2412.04093v1#bib.bib70)；Es等人，[2024](https://arxiv.org/html/2412.04093v1#bib.bib31)）和工具（例如，它们输出的可靠性和一致性）。

#### 9.2.1\. 整体评估

无论代理型LLM系统在过程中表现如何，或展示了什么新兴能力，最终输出将决定系统是否完成了任务。无法在不进行端到端运行的情况下预测LLM调用的组合会如何表现；因此，评估LLM代理主要应依赖于整体指标，以确定其是否按预期执行。

示例指标。

+   •

    在X个不同的提示下，代理产生了多少个正确答案？

+   •

    对于输入X在N次试验中的表现，代理产生了多少个不同的答案？

+   •

    对于输入X在N次试验中的表现，代理执行的步骤平均数量是多少？

+   •

    对于输入X在N次试验中的表现，代理使用工具的平均数量是多少？

+   •

    对于需要LLM规划的输入X在N次试验中的表现，每个规划的平均步骤数是多少？

+   •

    对于输入X在N次试验中的表现，平均成本/时间是多少？

#### 9.2.2\. 逐步评估

测量单个或 LLM 调用子集的性能，以完成可定义的任务，是诊断系统问题或进行更改的一种可行方法。然而，由于单个 LLM 调用可能对 LLM 代理产生下游影响，因此不应将对代理式 LLM 系统的孤立碎片化评估视为[整体评估](https://arxiv.org/html/2412.04093v1#S9.SS2.SSS1 "在 9.2\. 评估 ‣ 9\. 额外考虑 ‣ 代理式 LLM 系统的实践考虑")的替代。

示例指标。

+   •

    对于 N 次试验中的调用 X，产生了多少个不同的答案？

+   •

    对于 N 个同义版本的输入 A 调用 X，RAG 从 A 的每个嵌入版本中提供了多少个不同的 top-K 文档？

+   •

    在 N 次试验中使用工具访问调用 X，使用了多少种不同的工具？

+   •

    在 N 次试验中调用 X，平均成本/时间是多少？

### 9.3\. 与传统工程的集成

由于大型语言模型（LLMs）本质上是随机的，通常将尽可能多的代理责任转移到传统工程中是更为便捷的做法。这样可以将需要确定性的系统部分外包给可以实现确定性的方法。通过按照软件工程最佳实践设计LLM代理，我们可以确保完成或包括执行特定任务所需的关键组件，而不是依赖代理进行请求或执行操作。这可以表现为自动管理调用之间的上下文，[输出处理](https://arxiv.org/html/2412.04093v1#S8.SS1 "8.1\. Output Processing ‣ 8\. Control Flow ‣ Practical Considerations for Agentic LLM Systems")，将工具组合成工具集（例如，将图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").T2和[2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").T3放置在一个“交付”接口后），将来自长期记忆的信息永久融入提示中（例如，图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").E1），在某些过渡和调用上设置回调（例如，在从图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").Pe1到[2](https://arxiv.org/html/2412.04093v1#S3.F2 "Figure 2 ‣ 3\. Applied Scenario ‣ Practical Considerations for Agentic LLM Systems").Pe2）时生成最近一次对话的摘要以作为过渡时的上下文），并在每个计划步骤之后添加评估（参见[计划遵守](https://arxiv.org/html/2412.04093v1#S5.SS3 "In 5\. Planning ‣ Practical Considerations for Agentic LLM Systems")）。（最后两者可以视为隐式工具使用；参见第[7.1](https://arxiv.org/html/2412.04093v1#S7.SS1 "7.1\. Using Tools ‣ 7\. Tools ‣ Practical Considerations for Agentic LLM Systems")节。）然而，需注意的是，这样做时不能限制代理的自主性。一种在仍然利用传统工程优势的同时恢复代理自主性的方法是允许代理进行短路化。

#### 9.3.1\. 短路化

短路（来自软件工程领域的概念：仅评估一个表达式到足以保证得到单一答案的程度）是代理型LLM系统中的一项重要技术。这可以像将[停止](https://arxiv.org/html/2412.04093v1#S8.SS3 "8.3\. 停止 ‣ 8\. 控制流 ‣ 代理型LLM系统的实际考虑")标准添加到LLM代理的指令中（参见第[8.3](https://arxiv.org/html/2412.04093v1#S8.SS3 "8.3\. 停止 ‣ 8\. 控制流 ‣ 代理型LLM系统的实际考虑")节的示例），或允许LLM代理在一次操作中产生最终输出。如果一个代理型LLM系统在显然应该短路时没有进行短路，那么该系统可能过于依赖外部工程（即代理的流程（或部分流程）是硬编码的）^(13)^(13)13 最近的一个例子是OpenAI的GPT-o1（OpenAI，[2024](https://arxiv.org/html/2412.04093v1#bib.bib61)）。最初的实现没有短路，这意味着即使是一个较弱的模型可以处理的简单查询，或者不需要显著输出的查询，仍然会进行整个代理型LLM系统的完整遍历。例如，询问GPT-o1“什么都不做”仍然会经过规划、思考和对齐阶段。

例如，图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图2 ‣ 3\. 应用场景 ‣ 代理型LLM系统的实际考虑")中的查询I3展示了一个LLM代理可能希望进行短路的实例。该查询提出了一个简单的问答场景，目前的大多数模型都能令人满意地响应。允许代理自主决定下一步应该采取什么行动（与例如每次输入都隐式调用图[2](https://arxiv.org/html/2412.04093v1#S3.F2 "图2 ‣ 3\. 应用场景 ‣ 代理型LLM系统的实际考虑")中的P1不同）将使它能够简单地提供答案，从而短路其他组件。

## 10\. 限制

尽管我们展示了一些部署系统评估的实际方法，但我们并未探讨人类环节评估，因为人机交互是一个丰富的研究领域，超出了本工作的范围。

评估的一个重要后续是如何比较和响应部署的代理型LLM系统的变化，例如提示、模型和环境变化。这些考虑在当前文献中仍然没有得到充分探讨，并且代表了部署现实世界LLM代理的一些关键挑战。我们不讨论这些考虑，因为代理维护不属于本工作的范围，但建议它们是未来研究的显著方向。

我们探讨了代理型LLM系统成本的一个方面——模型大小，但将其他考虑因素（例如是使用现成的模型，还是在特定任务上对其进行微调（Bucher和Martini，[2024](https://arxiv.org/html/2412.04093v1#bib.bib24)；Lehman等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib46)），利用日益强大的开源模型（Dubey等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib28)；Jiang等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib38)；Bai等，[2023](https://arxiv.org/html/2412.04093v1#bib.bib19)）或依赖于对齐的商业模型（OpenAI等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib62)；Team等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib81)；cla，[2024](https://arxiv.org/html/2412.04093v1#bib.bib12)），以及是否自托管或使用第三方提供商等）留待未来的研究，因为代理架构的成本和可行性要求对此进行单独的评估。有关基于成本的LLM代理研究需求的讨论，请参见(Kapoor等，[2024](https://arxiv.org/html/2412.04093v1#bib.bib41))。

为了简化和与许多行业应用的相关性，我们从黑盒的角度接触代理的底层LLM，但将其作为白盒来处理则带来了额外的复杂性和机会。我们认为，考虑模型的具体细节超出了本次综述的范围，但也认识到未来的研究能够突出针对白盒LLM代理在现实世界中部署的实际考虑因素的价值。

## 11\. 结论

在本综述中，我们展示了与LLM代理相关的研究，并从中提取了可操作的见解，这些见解可以在实现和部署代理型LLM系统时加以利用。我们将相关研究和见解归纳为应用导向文献中LLM代理的四个主要组成部分——[规划](https://arxiv.org/html/2412.04093v1#S5 "在代理型LLM系统的实际考虑中")、[记忆](https://arxiv.org/html/2412.04093v1#S6 "在代理型LLM系统的实际考虑中")、[工具](https://arxiv.org/html/2412.04093v1#S7 "在代理型LLM系统的实际考虑中")和[控制流](https://arxiv.org/html/2412.04093v1#S8 "在代理型LLM系统的实际考虑中")——提供了一个对行业和学术界都可访问的综述。具体来说，关于[规划](https://arxiv.org/html/2412.04093v1#S5 "在代理型LLM系统的实际考虑中")，我们探讨了LLM规划能力差如何阻碍当前LLM代理应用的实施，以及任务分解的实际好处；关于[记忆](https://arxiv.org/html/2412.04093v1#S6 "在代理型LLM系统的实际考虑中")，我们探讨了在LLM代理中利用RAG和长期记忆时的好处和实际考虑；关于[工具](https://arxiv.org/html/2412.04093v1#S7 "在代理型LLM系统的实际考虑中")，我们讨论了如何为LLM代理呈现和管理工具；关于[控制流](https://arxiv.org/html/2412.04093v1#S8 "在代理型LLM系统的实际考虑中")，我们提供了促进LLM代理执行不中断并管理代理内部信息（如角色和上下文使用）的实用见解；最后，提出了其他考虑因素，如模型大小、评估和将LLM代理与传统工程结合的建议。

###### 致谢。

我们要感谢Sergei Petrov和Sonny George，他们的意见和反馈在塑造本工作的基础方面起到了关键作用。

## 参考文献

+   (1)

+   Ope ([n.d.]) OpenAI [n.d.]。*API参考*。OpenAI。于2024年10月16日从[https://platform.openai.com/docs/api-reference](https://platform.openai.com/docs/api-reference)获取。

+   PCo ([n.d.]) Pinecone [n.d.]。*为LLM代理构建自定义工具*。Pinecone。于2024年10月6日从[https://www.pinecone.io/learn/series/langchain/langchain-tools/](https://www.pinecone.io/learn/series/langchain/langchain-tools/)获取。

+   Ant ([n.d.]) Anthropic [n.d.]。*提高输出一致性（JSON模式）*。Anthropic。于2024年10月16日从[https://docs.anthropic.com/en/docs/test-and-evaluate/strengthen-guardrails/increase-consistency](https://docs.anthropic.com/en/docs/test-and-evaluate/strengthen-guardrails/increase-consistency)获取。

+   OAI ([n.d.]a) OpenAI [n.d.]a。*模型*。OpenAI。于2024年10月16日从[https://platform.openai.com/docs/models](https://platform.openai.com/docs/models)获取。

+   OAI（[n.d.]b）OpenAI [n.d.]b. *结构化输出*。OpenAI. 2024年10月16日获取自 [https://platform.openai.com/docs/guides/structured-outputs](https://platform.openai.com/docs/guides/structured-outputs)

+   Ant（2024a）Anthropic 2024a. *API参考*。Anthropic. 2024年10月16日获取自 [https://docs.anthropic.com/en/api](https://docs.anthropic.com/en/api)

+   Lan（2024）LangChain 2024. *评估您的LLM应用程序*。LangChain. 2024年10月16日获取自 [https://docs.smith.langchain.com/tutorials/Developers/evaluation](https://docs.smith.langchain.com/tutorials/Developers/evaluation)

+   Gem（2024a）Google 2024a. *Gemini模型*。Google. 2024年10月16日获取自 [https://ai.google.dev/gemini-api/docs/models/gemini](https://ai.google.dev/gemini-api/docs/models/gemini)

+   Gem（2024b）Google 2024b. *使用Gemini API生成内容*。Google. 2024年10月16日获取自 [https://cloud.google.com/vertex-ai/generative-ai/docs/model-reference/inference](https://cloud.google.com/vertex-ai/generative-ai/docs/model-reference/inference)

+   Gem（2024c）Google 2024c. *使用Gemini API生成结构化输出*。Google. 2024年10月16日获取自 [https://ai.google.dev/gemini-api/docs/structured-output](https://ai.google.dev/gemini-api/docs/structured-output)

+   cla（2024）Anthropic 2024. *介绍Claude的下一代*。Anthropic. 2024年10月8日获取自 [https://www.anthropic.com/news/claude-3-family](https://www.anthropic.com/news/claude-3-family)

+   PGA（2024）DAIR.AI 2024. *LLM代理*。DAIR.AI. 2024年10月8日获取自 [https://www.promptingguide.ai/research/llm-agents](https://www.promptingguide.ai/research/llm-agents)

+   SAA（2024）SuperAnnotate 2024. *LLM代理：终极指南*。SuperAnnotate. 2024年10月7日获取自 [https://www.superannotate.com/blog/llm-agents](https://www.superannotate.com/blog/llm-agents)

+   OAI（2024）OpenAI 2024. *ChatGPT的记忆和新控制功能*。OpenAI. 2024年10月8日获取自 [https://openai.com/index/memory-and-new-controls-for-chatgpt/](https://openai.com/index/memory-and-new-controls-for-chatgpt/)

+   Ant（2024b）Anthropic 2024b. *模型*。Anthropic. 2024年10月16日获取自 [https://docs.anthropic.com/en/docs/about-claude/models](https://docs.anthropic.com/en/docs/about-claude/models)

+   TFA（2024）truefoundry 2024. *什么是LLM代理？* truefoundry. 2024年10月7日获取自 [https://www.truefoundry.com/blog/llm-agents](https://www.truefoundry.com/blog/llm-agents)

+   Argyle等（2023）Lisa P. Argyle, Ethan C. Busby, Nancy Fulda, Joshua R. Gubler, Christopher Rytting, 和 David Wingate. 2023. 从一到多：利用语言模型模拟人类样本。*政治分析* 31, 3（2023年2月），337–351. [https://doi.org/10.1017/pan.2023.2](https://doi.org/10.1017/pan.2023.2)

+   Bai 等人（2023）Jinze Bai, Shuai Bai, Yunfei Chu, Zeyu Cui, Kai Dang, Xiaodong Deng, Yang Fan, Wenbin Ge, Yu Han, Fei Huang, Binyuan Hui, Luo Ji, Mei Li, Junyang Lin, Runji Lin, Dayiheng Liu, Gao Liu, Chengqiang Lu, Keming Lu, Jianxin Ma, Rui Men, Xingzhang Ren, Xuancheng Ren, Chuanqi Tan, Sinan Tan, Jianhong Tu, Peng Wang, Shijie Wang, Wei Wang, Shengguang Wu, Benfeng Xu, Jin Xu, An Yang, Hao Yang, Jian Yang, Shusheng Yang, Yang Yao, Bowen Yu, Hongyi Yuan, Zheng Yuan, Jianwei Zhang, Xingxuan Zhang, Yichang Zhang, Zhenru Zhang, Chang Zhou, Jingren Zhou, Xiaohuan Zhou, 和 Tianhang Zhu. 2023. 《Qwen技术报告》。arXiv:2309.16609 [cs.CL] [https://arxiv.org/abs/2309.16609](https://arxiv.org/abs/2309.16609)

+   Bai 等人（2022）Yuntao Bai, Saurav Kadavath, Sandipan Kundu, Amanda Askell, Jackson Kernion, Andy Jones, Anna Chen, Anna Goldie, Azalia Mirhoseini, Cameron McKinnon, Carol Chen, Catherine Olsson, Christopher Olah, Danny Hernandez, Dawn Drain, Deep Ganguli, Dustin Li, Eli Tran-Johnson, Ethan Perez, Jamie Kerr, Jared Mueller, Jeffrey Ladish, Joshua Landau, Kamal Ndousse, Kamile Lukosuite, Liane Lovitt, Michael Sellitto, Nelson Elhage, Nicholas Schiefer, Noemi Mercado, Nova DasSarma, Robert Lasenby, Robin Larson, Sam Ringer, Scott Johnston, Shauna Kravec, Sheer El Showk, Stanislav Fort, Tamera Lanham, Timothy Telleen-Lawton, Tom Conerly, Tom Henighan, Tristan Hume, Samuel R. Bowman, Zac Hatfield-Dodds, Ben Mann, Dario Amodei, Nicholas Joseph, Sam McCandlish, Tom Brown, 和 Jared Kaplan. 2022. 《宪法式人工智能：来自人工智能反馈的无害性》。arXiv:2212.08073 [cs.CL] [https://arxiv.org/abs/2212.08073](https://arxiv.org/abs/2212.08073)

+   Barron 等人（2024）Ryan C. Barron, Ves Grantcharov, Selma Wanna, Maksim E. Eren, Manish Bhattarai, Nicholas Solovyev, George Tompkins, Charles Nicholas, Kim Ø. Rasmussen, Cynthia Matuszek, 和 Boian S. Alexandrov. 2024. 《使用向量存储、知识图谱和张量分解的领域特定检索增强生成》。arXiv:2410.02721 [cs.CL] [https://arxiv.org/abs/2410.02721](https://arxiv.org/abs/2410.02721)

+   Benram（2024）Gad Benram. 2024. *理解大型语言模型（LLM）的成本*。2024年10月16日获取自 [https://www.tensorops.ai/post/understanding-the-cost-of-large-language-models-llms](https://www.tensorops.ai/post/understanding-the-cost-of-large-language-models-llms)

+   Beurer-Kellner 等人（2024）Luca Beurer-Kellner, Marc Fischer, 和 Martin Vechev. 2024. 《正确引导大型语言模型：快速、非侵入性约束生成》。发表于*第41届国际机器学习大会论文集*（*机器学习研究论文集，第235卷*），Ruslan Salakhutdinov, Zico Kolter, Katherine Heller, Adrian Weller, Nuria Oliver, Jonathan Scarlett, 和 Felix Berkenkamp（编辑）。PMLR, 3658–3673. [https://proceedings.mlr.press/v235/beurer-kellner24a.html](https://proceedings.mlr.press/v235/beurer-kellner24a.html)

+   布赫尔和马尔蒂尼（2024）马丁·胡安·何塞·布赫尔和马尔科·马尔蒂尼。2024。微调后的“小”LLM（仍然）显著超越零-shot生成式AI模型在文本分类中的表现。arXiv:2406.08660 [cs.CL] [https://arxiv.org/abs/2406.08660](https://arxiv.org/abs/2406.08660)

+   常等人（2024）常玉鹏、王旭、王金东、吴源、杨林义、朱凯杰、陈昊、易晓源、王存翔、王艺东、叶伟、张月、常一、余磊、杨强、谢星。2024。大语言模型评估综述。*ACM 智能系统技术学报* 15, 3, 第39篇（2024年3月），45页。 [https://doi.org/10.1145/3641289](https://doi.org/10.1145/3641289)

+   达根等人（2024）戈蒂埃·达根、弗兰克·凯勒、亚历克斯·拉斯卡里德斯·凯勒。2024。使用LLM进行动态规划。发表于*2024年NeurIPS语言游戏化研讨会论文集*。神经信息处理系统基金会（NeurIPS），1–14。[https://doi.org/10.48550/arXiv.2308.06391](https://doi.org/10.48550/arXiv.2308.06391) 2024年NeurIPS语言游戏化研讨会；会议日期：2024年12月14日至2024年12月14日。

+   邓等人（2024）邓世汉、徐伟凯、孙洪达、刘伟、谭涛、刘建峰、李俊峰、李昂、段健、王斌、严锐、尚烁。2024。Mobile-Bench：基于LLM的移动智能体评估基准。发表于*计算语言学协会第62届年会论文集（第一卷：长篇论文）*，吕文伟、安德烈·马丁斯、维韦克·斯里库马尔（主编）。计算语言学协会，泰国曼谷，8813–8831。 [https://doi.org/10.18653/v1/2024.acl-long.478](https://doi.org/10.18653/v1/2024.acl-long.478)

+   Dubey 等人 (2024) Abhimanyu Dubey, Abhinav Jauhri, Abhinav Pandey, Abhishek Kadian, Ahmad Al-Dahle, Aiesha Letman, Akhil Mathur, Alan Schelten, Amy Yang, Angela Fan, Anirudh Goyal, Anthony Hartshorn, Aobo Yang, Archi Mitra, Archie Sravankumar, Artem Korenev, Arthur Hinsvark, Arun Rao, Aston Zhang, Aurelien Rodriguez, Austen Gregerson, Ava Spataru, Baptiste Roziere, Bethany Biron, Binh Tang, Bobbie Chern, Charlotte Caucheteux, Chaya Nayak, Chloe Bi, Chris Marra, Chris McConnell, Christian Keller, Christophe Touret, Chunyang Wu, Corinne Wong, Cristian Canton Ferrer, Cyrus Nikolaidis, Damien Allonsius, Daniel Song, Danielle Pintz, Danny Livshits, David Esiobu, Dhruv Choudhary, Dhruv Mahajan, Diego Garcia-Olano, Diego Perino, Dieuwke Hupkes, Egor Lakomkin, Ehab AlBadawy, Elina Lobanova, Emily Dinan, Eric Michael Smith, Filip Radenovic, Frank Zhang, Gabriel Synnaeve, Gabrielle Lee, Georgia Lewis Anderson, Graeme Nail, Gregoire Mialon, Guan Pang, Guillem Cucurell, Hailey Nguyen, Hannah Korevaar, Hu Xu, Hugo Touvron, Iliyan Zarov, Imanol Arrieta Ibarra, Isabel Kloumann, Ishan Misra, Ivan Evtimov, Jade Copet, Jaewon Lee, Jan Geffert, Jana Vranes, Jason Park, Jay Mahadeokar, Jeet Shah, Jelmer van der Linde, Jennifer Billock, Jenny Hong, Jenya Lee, Jeremy Fu, Jianfeng Chi, Jianyu Huang, Jiawen Liu, Jie Wang, Jiecao Yu, Joanna Bitton, Joe Spisak, Jongsoo Park, Joseph Rocca, Joshua Johnstun, Joshua Saxe, Junteng Jia, Kalyan Vasuden Alwala, Kartikeya Upasani, Kate Plawiak, Ke Li, Kenneth Heafield, Kevin Stone, Khalid El-Arini, Krithika Iyer, Kshitiz Malik, Kuenley Chiu, Kunal Bhalla, Lauren Rantala-Yeary, Laurens van der Maaten, Lawrence Chen, Liang Tan, Liz Jenkins, Louis Martin, Lovish Madaan, Lubo Malo, Lukas Blecher, Lukas Landzaat, Luke de Oliveira, Madeline Muzzi, Mahesh Pasupuleti, Mannat Singh, Manohar Paluri, Marcin Kardas, Mathew Oldham, Mathieu Rita, Maya Pavlova, Melanie Kambadur, Mike Lewis, Min Si, Mitesh Kumar Singh, Mona Hassan, Naman Goyal, Narjes Torabi, Nikolay Bashlykov, Nikolay Bogoychev, Niladri Chatterji, Olivier Duchenne, Onur Çelebi, Patrick Alrassy, Pengchuan Zhang, Pengwei Li, Petar Vasic, Peter Weng, Prajjwal Bhargava, Pratik Dubal, Praveen Krishnan, Punit Singh Koura, Puxin Xu, Qing He, Qingxiao Dong, Ragavan Srinivasan, Raj Ganapathy, Ramon Calderer, Ricardo Silveira Cabral, Robert Stojnic, Roberta Raileanu, Rohit Girdhar, Rohit Patel, Romain Sauvestre, Ronnie Polidoro, Roshan Sumbaly, Ross Taylor, Ruan Silva, Rui Hou, Rui Wang, Saghar Hosseini, Sahana Chennabasappa, Sanjay Singh, Sean Bell, Seohyun Sonia Kim, Sergey Edunov, Shaoliang Nie, Sharan Narang, Sharath Raparthy, Sheng Shen, Shengye Wan, Shruti Bhosale, Shun Zhang, Simon Vandenhende, Soumya Batra, Spencer Whitman, Sten Sootla, Stephane Collot, Suchin Gururangan, Sydney Borodinsky, Tamar Herman, Tara Fowler, Tarek Sheasha, Thomas Georgiou, Thomas Scialom, Tobias Speckbacher, Todor Mihaylov, Tong Xiao, Ujjwal Karn, Vedanuj Goswami, Vibhor Gupta, Vignesh Ramanathan, Viktor Kerkez, Vincent Gonguet, Virginie Do, Vish Vogeti, Vladan Petrovic, Weiwei Chu, Wenhan Xiong, Wenyin Fu, Whitney Meers, Xavier Martinet, Xiaodong Wang, Xiaoqing Ellen Tan, Xinfeng Xie, Xuchao Jia, Xuewei Wang, Yaelle Goldschlag, Yashesh Gaur, Yasmine Babaei, Yi Wen, Yiwen Song, Yuchen Zhang, Yue Li, Yuning Mao, Zacharie Delpierre Coudert, Zheng Yan, Zhengxing Chen, Zoe Papakipos, Aaditya Singh, Aaron Grattafiori, Abha Jain, Adam Kelsey, Adam Shajnfeld, Adithya Gangidi, Adolfo Victoria, Ahuva Goldstand, Ajay Menon, Ajay Sharma, Alex Boesenberg, Alex Vaughan, Alexei Baevski, Allie Feinstein, Amanda Kallet, Amit Sangani, Anam Yunus, Andrei Lupu, Andres Alvarado, Andrew Caples, Andrew Gu, Andrew Ho, Andrew Poulton, Andrew Ryan, Ankit Ramchandani, Annie Franco, Aparajita Saraf, Arkabandhu Chowdhury, Ashley Gabriel, Ashwin Bharambe, Assaf Eisenman, Azadeh Yazdan, Beau James, Ben Maurer, Benjamin Leonhardi, Bernie Huang, Beth Loyd, Beto De Paola, Bhargavi Paranjape, Bing Liu, Bo Wu, Boyu Ni, Braden Hancock, Bram Wasti, Brandon Spence, Brani Stojkovic, Brian Gamido, Britt Montalvo, Carl Parker, Carly Burton, Catalina Mejia, Changhan Wang, Changkyu Kim, Chao Zhou, Chester Hu, Ching-Hsiang Chu, Chris Cai, Chris Tindal, Christoph Feichtenhofer, Damon Civin, Dana Beaty, Daniel Kreymer, Daniel Li, Danny Wyatt, David Adkins, David Xu, Davide Testuggine, Delia David, Devi Parikh, Diana Liskovich, Didem Foss, Dingkang Wang, Duc Le, Dustin Holland, Edward Dowling, Eissa Jamil, Elaine Montgomery, Eleonora Presani, Emily Hahn, Emily Wood, Erik Brinkman, Esteban Arcaute, Evan Dunbar, Evan Smothers, Fei Sun, Felix Kreuk, Feng Tian, Firat Ozgenel, Francesco Caggioni, Francisco Guzmán, Frank Kanayet, Frank Seide, Gabriela Medina Florez, Gabriella Schwarz, Gada Badeer, Georgia Swee, Gil Halpern, Govind Thattai, Grant Herman, Grigory Sizov, Guangyi, Zhang, Guna Lakshminarayanan, Hamid Shojanazeri, Han Zou, Hannah Wang, Hanwen Zha, Haroun Habeeb, Harrison Rudolph, Helen Suk, Henry Aspegren, Hunter Goldman, Ibrahim Damlaj, Igor Molybog, Igor Tufanov, Irina-Elena Veliche, Itai Gat, Jake Weissman, James Geboski, James Kohli, Japhet Asher, Jean-Baptiste Gaya, Jeff Marcus, Jeff Tang, Jennifer Chan, Jenny Zhen, Jeremy Reizenstein, Jeremy Teboul, Jessica Zhong, Jian Jin, Jingyi Yang, Joe Cummings, Jon Carvill, Jon Shepard, Jonathan McPhie, Jonathan Torres, Josh Ginsburg, Junjie Wang, Kai Wu, Kam Hou U, Karan Saxena, Karthik Prasad, Kartikay Khandelwal, Katayoun Zand, Kathy Matosich, Kaushik Veeraraghavan, Kelly Michelena, Keqian Li, Kun Huang, Kunal Chawla, Kushal Lakhotia, Kyle Huang, Lailin Chen, Lakshya Garg, Lavender A, Leandro Silva, Lee Bell, Lei Zhang, Liangpeng Guo, Licheng Yu, Liron Moshkovich, Luca Wehrstedt, Madian Khabsa, Manav Avalani, Manish Bhatt, Maria Tsimpoukelli, Martynas Mankus, Matan Hasson, Matthew Lennie, Matthias Reso, Maxim Groshev, Maxim Naumov, Maya Lathi, Meghan Keneally, Michael L. Seltzer, Michal Valko, Michelle Restrepo, Mihir Patel, Mik Vyatskov, Mikayel Samvelyan, Mike Clark, Mike Macey, Mike Wang, Miquel Jubert Hermoso, Mo Metanat, Mohammad Rastegari, Munish Bansal, Nandhini Santhanam, Natascha Parks, Natasha White, Navyata Bawa, Nayan Singhal, Nick Egebo, Nicolas Usunier, Nikolay Pavlovich Laptev, Ning Dong, Ning Zhang, Norman Cheng, Oleg Chernoguz, Olivia Hart, Omkar Salpekar, Ozlem Kalinli, Parkin Kent, Parth Parekh, Paul Saab, Pavan Balaji, Pedro Rittner, Philip Bontrager, Pierre Roux, Piotr Dollar, Polina Zvyagina, Prashant Ratanchandani, Pritish Y

+   Echterhoff 等（2024）Jessica Maria Echterhoff、Yao Liu、Abeer Alessa、Julian McAuley 和 Zexue He. 2024. 《基于大型语言模型的决策中的认知偏差》。载于 *计算语言学协会年会论文集：EMNLP 2024*，Yaser Al-Onaizan、Mohit Bansal 和 Yun-Nung Chen（编）。计算语言学协会，佛罗里达州迈阿密，美国，12640–12653. [https://doi.org/10.18653/v1/2024.findings-emnlp.739](https://doi.org/10.18653/v1/2024.findings-emnlp.739)

+   Edge 等（2024）Darren Edge、Ha Trinh、Newman Cheng、Joshua Bradley、Alex Chao、Apurva Mody、Steven Truitt 和 Jonathan Larson. 2024. 《从局部到全局：基于图的检索增强生成方法的查询聚焦摘要》。arXiv:2404.16130 [cs.CL] [https://arxiv.org/abs/2404.16130](https://arxiv.org/abs/2404.16130)

+   Es 等（2024）Shahul Es、Jithin James、Luis Espinosa Anke 和 Steven Schockaert. 2024. 《RAGAs：检索增强生成的自动评估》。载于 *计算语言学协会欧洲分会第18届会议：系统演示论文集*，Nikolaos Aletras 和 Orphee De Clercq（编）。计算语言学协会，马耳他圣朱利安，150–158. [https://aclanthology.org/2024.eacl-demo.16](https://aclanthology.org/2024.eacl-demo.16)

+   Gao 等（2024）Yunfan Gao、Yun Xiong、Xinyu Gao、Kangxiang Jia、Jinliu Pan、Yuxi Bi、Yi Dai、Jiawei Sun、Meng Wang 和 Haofen Wang. 2024. 《基于检索增强生成的大型语言模型：一项调查》。arXiv:2312.10997 [cs.CL] [https://arxiv.org/abs/2312.10997](https://arxiv.org/abs/2312.10997)

+   Guo 等（2024）Taicheng Guo、Xiuying Chen、Yaqi Wang、Ruidi Chang、Shichao Pei、Nitesh V. Chawla、Olaf Wiest 和 Xiangliang Zhang. 2024. 《基于大型语言模型的多智能体：进展与挑战综述》。载于 *第三十三届国际人工智能联合会议论文集，IJCAI-24*，Kate Larson（编）。国际人工智能联合会议组织，8048–8057. [https://doi.org/10.24963/ijcai.2024/890](https://doi.org/10.24963/ijcai.2024/890) 综述专辑。

+   Gur 等（2024）Izzeddin Gur、Hiroki Furuta、Austin V Huang、Mustafa Safdari、Yutaka Matsuo、Douglas Eck 和 Aleksandra Faust. 2024. 《一个具有规划、长时上下文理解和程序合成的真实世界 WebAgent》。载于 *第十二届国际学习表示大会*。 [https://openreview.net/forum?id=9JQtrumvg8](https://openreview.net/forum?id=9JQtrumvg8)

+   Holtzman 等（2020）Ari Holtzman、Jan Buys、Li Du、Maxwell Forbes 和 Yejin Choi. 2020. 《神经文本退化的奇特案例》。载于 *国际学习表示大会*。 [https://openreview.net/forum?id=rygGQyrFvH](https://openreview.net/forum?id=rygGQyrFvH)

+   Hong等人（2024）Sirui Hong, Mingchen Zhuge, Jonathan Chen, Xiawu Zheng, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, Chenyu Ran, Lingfeng Xiao, Chenglin Wu, 和 Jürgen Schmidhuber. 2024. MetaGPT：面向多代理协作框架的元编程。收录于 *第十二届国际学习表示会议*。 [https://openreview.net/forum?id=VtmBAGCN7o](https://openreview.net/forum?id=VtmBAGCN7o)

+   Huang等人（2024）Xu Huang, Weiwen Liu, Xiaolong Chen, Xingmei Wang, Hao Wang, Defu Lian, Yasheng Wang, Ruiming Tang, 和 Enhong Chen. 2024. 理解LLM代理的规划：一项调查。arXiv:2402.02716 [cs.AI] [https://arxiv.org/abs/2402.02716](https://arxiv.org/abs/2402.02716)

+   Jiang等人（2023）Albert Q. Jiang, Alexandre Sablayrolles, Arthur Mensch, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Florian Bressand, Gianna Lengyel, Guillaume Lample, Lucile Saulnier, Lélio Renard Lavaud, Marie-Anne Lachaux, Pierre Stock, Teven Le Scao, Thibaut Lavril, Thomas Wang, Timothée Lacroix, 和 William El Sayed. 2023. Mistral 7B. arXiv:2310.06825 [cs.CL] [https://arxiv.org/abs/2310.06825](https://arxiv.org/abs/2310.06825)

+   Kambhampati（2024）Subbarao Kambhampati. 2024. 大型语言模型能否推理和规划？*纽约科学院年刊* 1534, 1 (2024年3月), 15–18。 [https://doi.org/10.1111/nyas.15125](https://doi.org/10.1111/nyas.15125)

+   Kamoi等人（2024）Ryo Kamoi, Sarkar Snigdha Sarathi Das, Renze Lou, Jihyun Janice Ahn, Yilun Zhao, Xiaoxin Lu, Nan Zhang, Yusen Zhang, Haoran Ranran Zhang, Sujeeth Reddy Vummanthala, Salika Dave, Shaobo Qin, Arman Cohan, Wenpeng Yin, 和 Rui Zhang. 2024. 评估LLM在检测LLM回应错误方面的表现。收录于 *第一次语言建模会议*。 [https://openreview.net/forum?id=dnwRScljXr](https://openreview.net/forum?id=dnwRScljXr)

+   Kapoor等人（2024）Sayash Kapoor, Benedikt Stroebl, Zachary S. Siegel, Nitya Nadgir, 和 Arvind Narayanan. 2024. 重要的AI代理。arXiv:2407.01502 [cs.LG] [https://arxiv.org/abs/2407.01502](https://arxiv.org/abs/2407.01502)

+   Karmaker Santu 和 Feng（2023）Shubhra Kanti Karmaker Santu 和 Dongji Feng. 2023. TELeR：用于基准复杂任务的大型语言模型提示的通用分类法。收录于 *计算语言学协会会议成果：EMNLP 2023*，Houda Bouamor, Juan Pino, 和 Kalika Bali（编辑）。计算语言学协会，新加坡，14197–14203。 [https://doi.org/10.18653/v1/2023.findings-emnlp.946](https://doi.org/10.18653/v1/2023.findings-emnlp.946)

+   Kojima等人（2022）Takeshi Kojima, Shixiang Shane Gu, Machel Reid, Yutaka Matsuo, 和 Yusuke Iwasawa. 2022. 大型语言模型是零样本推理器。*神经信息处理系统进展* 35 (2022), 22199–22213.

+   Kong等（2024）Aobo Kong, Shiwan Zhao, Hao Chen, Qicheng Li, Yong Qin, Ruiqi Sun, Xin Zhou, Enzhi Wang 和 Xiaohang Dong. 2024. 通过角色扮演提示提升零-shot推理能力。在*2024年北美计算语言学协会：人类语言技术会议（卷1：长篇论文）*中，Kevin Duh、Helena Gomez 和 Steven Bethard（编）。计算语言学协会，墨西哥城，墨西哥，4099–4113。[https://doi.org/10.18653/v1/2024.naacl-long.228](https://doi.org/10.18653/v1/2024.naacl-long.228)

+   Laskar等（2023）Md Tahmid Rahman Laskar, M Saiful Bari, Mizanur Rahman, Md Amran Hossen Bhuiyan, Shafiq Joty 和 Jimmy Huang. 2023. ChatGPT在基准数据集上的系统性研究与全面评估。在*计算语言学协会会议发现：ACL 2023*中，Anna Rogers、Jordan Boyd-Graber 和 Naoaki Okazaki（编）。计算语言学协会，多伦多，加拿大，431–469。[https://doi.org/10.18653/v1/2023.findings-acl.29](https://doi.org/10.18653/v1/2023.findings-acl.29)

+   Lehman等（2023）Eric Lehman, Evan Hernandez, Diwakar Mahajan, Jonas Wulff, Micah J Smith, Zachary Ziegler, Daniel Nadler, Peter Szolovits, Alistair Johnson 和 Emily Alsentzer. 2023. 我们仍然需要临床语言模型吗？在*健康、推理和学习会议论文集*（*机器学习研究论文集，卷209*）中，Bobak J. Mortazavi, Tasmie Sarker, Andrew Beam 和 Joyce C. Ho（编）。PMLR，578–597。[https://proceedings.mlr.press/v209/eric23a.html](https://proceedings.mlr.press/v209/eric23a.html)

+   Lewis等（2020）Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, Sebastian Riedel 和 Douwe Kiela. 2020. 用于知识密集型NLP任务的检索增强生成。在*第34届神经信息处理系统国际会议论文集*（温哥华，加拿大）*(NIPS '20)*中。Curran Associates Inc.，纽约州红钩，美国，文章793，16页。

+   Li等（2023a）Huao Li, Yu Chong, Simon Stepputtis, Joseph Campbell, Dana Hughes, Charles Lewis, 和 Katia Sycara. 2023a. 通过大型语言模型实现多智能体协作的心智理论。在*2023年自然语言处理实证方法会议论文集*中，Houda Bouamor、Juan Pino 和 Kalika Bali（编）。计算语言学协会， Singapore，180–192。[https://doi.org/10.18653/v1/2023.emnlp-main.13](https://doi.org/10.18653/v1/2023.emnlp-main.13)

+   Li等（2024）Tianle Li, Ge Zhang, Quy Duc Do, Xiang Yue 和 Wenhu Chen. 2024. 长上下文LLMs在长上下文学习中遇到困难。*CoRR* abs/2404.02060（2024）。[https://doi.org/10.48550/arXiv.2404.02060](https://doi.org/10.48550/arXiv.2404.02060)

+   Li (2024) Xinzhe Li. 2024. LLM基础代理的主要范式综述：工具使用（包括RAG）、规划与反馈学习。arXiv:2406.05804 [cs.AI] [https://arxiv.org/abs/2406.05804](https://arxiv.org/abs/2406.05804)

+   Li et al. (2023b) Yuan Li, Yixuan Zhang, 和 Lichao Sun. 2023b. MetaAgents: 通过协作生成代理模拟人类行为的交互，用于基于LLM的任务导向协调。*ArXiv* abs/2310.06500 (2023)。 [https://api.semanticscholar.org/CorpusID:263829557](https://api.semanticscholar.org/CorpusID:263829557)

+   Lin et al. (2023) Jiaju Lin, Haoran Zhao, Aochi Zhang, Yiting Wu, Huqiuyue Ping, 和 Qin Chen. 2023. AgentSims: 一种用于大型语言模型评估的开源沙盒。arXiv:2308.04026 [cs.AI] [https://arxiv.org/abs/2308.04026](https://arxiv.org/abs/2308.04026)

+   Liu et al. (2023) Bo Liu, Yuqian Jiang, Xiaohan Zhang, Qiang Liu, Shiqi Zhang, Joydeep Biswas, 和 Peter Stone. 2023. LLM+P: 赋能大型语言模型以优化规划能力。arXiv:2304.11477 [cs.AI] [https://arxiv.org/abs/2304.11477](https://arxiv.org/abs/2304.11477)

+   Liu et al. (2024b) Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, 和 Percy Liang. 2024b. 迷失在中间：语言模型如何使用长上下文。*计算语言学协会会刊* 12 (2024), 157–173. [https://doi.org/10.1162/tacl_a_00638](https://doi.org/10.1162/tacl_a_00638)

+   Liu et al. (2024c) Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang, Shudan Zhang, Xiang Deng, Aohan Zeng, Zhengxiao Du, Chenhui Zhang, Sheng Shen, Tianjun Zhang, Yu Su, Huan Sun, Minlie Huang, Yuxiao Dong, 和 Jie Tang. 2024c. AgentBench: 评估LLM作为代理的能力。载于 *第十二届国际学习表示会议*。 [https://openreview.net/forum?id=zAdUB0aCTQ](https://openreview.net/forum?id=zAdUB0aCTQ)

+   Liu et al. (2024a) Zhihan Liu, Hao Hu, Shenao Zhang, Hongyi Guo, Shuqi Ke, Boyi Liu, 和 Zhaoran Wang. 2024a. 面向未来，行动当下：一种具有可证明样本效率的自主LLM代理的原则框架。arXiv:2309.17382 [cs.AI] [https://arxiv.org/abs/2309.17382](https://arxiv.org/abs/2309.17382)

+   ljunkai (2023) ljunkai. 2023. *如何找到大型语言模型的最优模型大小，以优化效果和成本*。2024年10月16日检索自 [https://repost.aws/articles/ARv5lSlUnnSkanRxSD2EFz5w/how-to-find-the-optimal-model-size-for-large-language-models-to-optimize-effectiveness-and-cost](https://repost.aws/articles/ARv5lSlUnnSkanRxSD2EFz5w/how-to-find-the-optimal-model-size-for-large-language-models-to-optimize-effectiveness-and-cost)

+   Mehta 等人（2024）Nikhil Mehta、Milagro Teruel、Xin Deng、Sergio Figueroa Sanz、Ahmed Awadallah 和 Julia Kiseleva。2024年。通过与代理的帮助反馈互动，改善协作环境中的基础语言理解。载于*计算语言学协会成果：EACL 2024*，Yvette Graham 和 Matthew Purver（编）。计算语言学协会，马耳他圣朱利安斯，1306–1321。[https://aclanthology.org/2024.findings-eacl.87](https://aclanthology.org/2024.findings-eacl.87)

+   Mialon 等人（2024）Grégoire Mialon、Clémentine Fourrier、Thomas Wolf、Yann LeCun 和 Thomas Scialom。2024年。《GAIA：通用AI助手的基准测试》。载于*第十二届国际学习表征会议*。[https://openreview.net/forum?id=fibxvahvs3](https://openreview.net/forum?id=fibxvahvs3)

+   Nau 等人（2004）Dana Nau、Malik Ghallab 和 Paolo Traverso。2004年。*自动规划：理论与实践*。Morgan Kaufmann Publishers Inc.，美国加利福尼亚州旧金山。

+   OpenAI（2024）OpenAI。2024年。《学习使用LLMs进行推理》。技术报告。2024年10月16日从[https://openai.com/index/learning-to-reason-with-llms/](https://openai.com/index/learning-to-reason-with-llms/)获取。

+   OpenAI 等人（2024）OpenAI、Josh Achiam、Steven Adler、Sandhini Agarwal、Lama Ahmad、Ilge Akkaya、Florencia Leoni Aleman、Diogo Almeida、Janko Altenschmidt、Sam Altman、Shyamal Anadkat、Red Avila、Igor Babuschkin、Suchir Balaji、Valerie Balcom、Paul Baltescu、Haiming Bao、Mohammad Bavarian、Jeff Belgum、Irwan Bello、Jake Berdine、Gabriel Bernadett-Shapiro、Christopher Berner、Lenny Bogdonoff、Oleg Boiko、Madelaine Boyd、Anna-Luisa Brakman、Greg Brockman、Tim Brooks、Miles Brundage、Kevin Button、Trevor Cai、Rosie Campbell、Andrew Cann、Brittany Carey、Chelsea Carlson、Rory Carmichael、Brooke Chan、Che Chang、Fotis Chantzis、Derek Chen、Sully Chen、Ruby Chen、Jason Chen、Mark Chen、Ben Chess、Chester Cho、Casey Chu、Hyung Won Chung、Dave Cummings、Jeremiah Currier、Yunxing Dai、Cory Decareaux、Thomas Degry、Noah Deutsch、Damien Deville、Arka Dhar、David Dohan、Steve Dowling、Sheila Dunning、Adrien Ecoffet、Atty Eleti、Tyna Eloundou、David Farhi、Liam Fedus、Niko Felix、Simón Posada Fishman、Juston Forte、Isabella Fulford、Leo Gao、Elie Georges、Christian Gibson、Vik Goel、Tarun Gogineni、Gabriel Goh、Rapha Gontijo-Lopes、Jonathan Gordon、Morgan Grafstein、Scott Gray、Ryan Greene、Joshua Gross、Shixiang Shane Gu、Yufei Guo、Chris Hallacy、Jesse Han、Jeff Harris、Yuchen He、Mike Heaton、Johannes Heidecke、Chris Hesse、Alan Hickey、Wade Hickey、Peter Hoeschele、Brandon Houghton、Kenny Hsu、Shengli Hu、Xin Hu、Joost Huizinga、Shantanu Jain、Shawn Jain、Joanne Jang、Angela Jiang、Roger Jiang、Haozhun Jin、Denny Jin、Shino Jomoto、Billie Jonn、Heewoo Jun、Tomer Kaftan、Łukasz Kaiser、Ali Kamali、Ingmar Kanitscheider、Nitish Shirish Keskar、Tabarak Khan、Logan Kilpatrick、Jong Wook Kim、Christina Kim、Yongjik Kim、Jan Hendrik Kirchner、Jamie Kiros、Matt Knight、Daniel Kokotajlo、Łukasz Kondraciuk、Andrew Kondrich、Aris Konstantinidis、Kyle Kosic、Gretchen Krueger、Vishal Kuo、Michael Lampe、Ikai Lan、Teddy Lee、Jan Leike、Jade Leung、Daniel Levy、Chak Ming Li、Rachel Lim、Molly Lin、Stephanie Lin、Mateusz Litwin、Theresa Lopez、Ryan Lowe、Patricia Lue、Anna Makanju、Kim Malfacini、Sam Manning、Todor Markov、Yaniv Markovski、Bianca Martin、Katie Mayer、Andrew Mayne、Bob McGrew、Scott Mayer McKinney、Christine McLeavey、Paul McMillan、Jake McNeil、David Medina、Aalok Mehta、Jacob Menick、Luke Metz、Andrey Mishchenko、Pamela Mishkin、Vinnie Monaco、Evan Morikawa、Daniel Mossing、Tong Mu、Mira Murati、Oleg Murk、David Mély、Ashvin Nair、Reiichiro Nakano、Rajeev Nayak、Arvind Neelakantan、Richard Ngo、Hyeonwoo Noh、Long Ouyang、Cullen O’Keefe、Jakub Pachocki、Alex Paino、Joe Palermo、Ashley Pantuliano、Giambattista Parascandolo、Joel Parish、Emy Parparita、Alex Passos、Mikhail Pavlov、Andrew Peng、Adam Perelman、Filipe de Avila Belbute Peres、Michael Petrov、Henrique Ponde de Oliveira Pinto、Michael Pokorny、Michelle Pokrass、Vitchyr H. Pong、Tolly Powell、Alethea Power、Boris Power、Elizabeth Proehl、Raul Puri、Alec Radford、Jack Rae、Aditya Ramesh、Cameron Raymond、Francis Real、Kendra Rimbach、Carl Ross、Bob Rotsted、Henri Roussez、Nick Ryder、Mario Saltarelli、Ted Sanders、Shibani Santurkar、Girish Sastry、Heather Schmidt、David Schnurr、John Schulman、Daniel Selsam、Kyla Sheppard、Toki Sherbakov、Jessica Shieh、Sarah Shoker、Pranav Shyam、Szymon Sidor、Eric Sigler、Maddie Simens、Jordan Sitkin、Katarina Slama、Ian Sohl、Benjamin Sokolowsky、Yang Song、Natalie Staudacher、Felipe Petroski Such、Natalie Summers、Ilya Sutskever、Jie Tang、Nikolas Tezak、Madeleine B. Thompson、Phil Tillet、Amin Tootoonchian、Elizabeth Tseng、Preston Tuggle、Nick Turley、Jerry Tworek、Juan Felipe Cerón Uribe、Andrea Vallone、Arun Vijayvergiya、Chelsea Voss、Carroll Wainwright、Justin Jay Wang、Alvin Wang、Ben Wang、Jonathan Ward、Jason Wei、CJ Weinmann、Akila Welihinda、Peter Welinder、Jiayi Weng、Lilian Weng、Matt Wiethoff、Dave Willner、Clemens Winter、Samuel Wolrich、Hannah Wong、Lauren Workman、Sherwin Wu、Jeff Wu、Michael Wu、Kai Xiao、Tao Xu、Sarah Yoo、Kevin Yu、Qiming Yuan、Wojciech Zaremba、Rowan Zellers、Chong Zhang、Marvin Zhang、Shengjia Zhao、Tianhao Zheng、Juntang Zhuang、William Zhuk 和 Barret Zoph。2024。GPT-4 技术报告。arXiv:2303.08774 [cs.CL] [https://arxiv.org/abs/2303.08774](https://arxiv.org/abs/2303.08774)

+   Park 等人（2023）Joon Sung Park、Joseph O’Brien、Carrie Jun Cai、Meredith Ringel Morris、Percy Liang 和 Michael S. Bernstein。2023年。生成代理：人类行为的互动模拟 *(UIST ’23)*。美国计算机协会，纽约，美国，文章 2，22 页。[https://doi.org/10.1145/3586183.3606763](https://doi.org/10.1145/3586183.3606763)

+   Pope 等人（2023）Reiner Pope、Sholto Douglas、Aakanksha Chowdhery、Jacob Devlin、James Bradbury、Jonathan Heek、Kefan Xiao、Shivani Agrawal 和 Jeff Dean。2023年。高效扩展 Transformer 推理。发表于 *MLSys*。[https://proceedings.mlsys.org/paper_files/paper/2023/hash/c4be71ab8d24cdfb45e3d06dbfca2780-Abstract-mlsys2023.html](https://proceedings.mlsys.org/paper_files/paper/2023/hash/c4be71ab8d24cdfb45e3d06dbfca2780-Abstract-mlsys2023.html)

+   Qian 等人（2024a）Chen Qian、Wei Liu、Hongzhang Liu、Nuo Chen、Yufan Dang、Jiahao Li、Cheng Yang、Weize Chen、Yusheng Su、Xin Cong、Juyuan Xu、Dahai Li、Zhiyuan Liu 和 Maosong Sun。2024a。ChatDev：软件开发的交互代理。发表于 *第62届计算语言学协会年会论文集（卷1：长篇论文）*，Lun-Wei Ku、Andre Martins 和 Vivek Srikumar（编辑）。计算语言学协会，泰国曼谷，15174–15186。[https://doi.org/10.18653/v1/2024.acl-long.810](https://doi.org/10.18653/v1/2024.acl-long.810)

+   Qian 等人（2024b）Hongjin Qian、Zheng Liu、Peitian Zhang、Kelong Mao、Yujia Zhou、Xu Chen 和 Zhicheng Dou。2024b。长文本任务是否需要长时间的语言模型？arXiv:2405.15318 [cs.CL] [https://arxiv.org/abs/2405.15318](https://arxiv.org/abs/2405.15318)

+   Qin 等人（2024a）Yujia Qin、Shengding Hu、Yankai Lin、Weize Chen、Ning Ding、Ganqu Cui、Zheni Zeng、Xuanhe Zhou、Yufei Huang、Chaojun Xiao、Chi Han、Yi Ren Fung、Yusheng Su、Huadong Wang、Cheng Qian、Runchu Tian、Kunlun Zhu、Shihao Liang、Xingyu Shen、Bokai Xu、Zhen Zhang、Yining Ye、Bowen Li、Ziwei Tang、Jing Yi、Yuzhang Zhu、Zhenning Dai、Lan Yan、Xin Cong、Yaxi Lu、Weilin Zhao、Yuxiang Huang、Junxi Yan、Xu Han、Xian Sun、Dahai Li、Jason Phang、Cheng Yang、Tongshuang Wu、Heng Ji、Guoliang Li、Zhiyuan Liu 和 Maosong Sun。2024a。使用基础模型进行工具学习。*ACM Comput. Surv.*（2024年11月）。[https://doi.org/10.1145/3704435](https://doi.org/10.1145/3704435)

+   Qin 等人（2024b）Yujia Qin、Shihao Liang、Yining Ye、Kunlun Zhu、Lan Yan、Yaxi Lu、Yankai Lin、Xin Cong、Xiangru Tang、Bill Qian、Sihan Zhao、Lauren Hong、Runchu Tian、Ruobing Xie、Jie Zhou、Mark Gerstein、Dahai Li、Zhiyuan Liu 和 Maosong Sun。2024b。ToolLLM：帮助大语言模型掌握 16000+ 个真实世界的 API。发表于 *第十二届国际学习表示会议*。[https://openreview.net/forum?id=dHng2O0Jjr](https://openreview.net/forum?id=dHng2O0Jjr)

+   Roucher 和 Petrov (2024) Aymeric Roucher 和 Sergei Petrov. 2024. *我们的 Transformer 代码代理超越了 GAIA 基准测试！* Hugging Face. 2024年10月2日检索自 [https://huggingface.co/blog/beating-gaia](https://huggingface.co/blog/beating-gaia)

+   Salemi 和 Zamani (2024) Alireza Salemi 和 Hamed Zamani. 2024. 在检索增强生成中的检索质量评估. 收录于 *第47届国际 ACM SIGIR 信息检索研究与发展会议论文集*（美国华盛顿DC） *(SIGIR ’24)*. 计算机协会，美国纽约，2395–2400页. [https://doi.org/10.1145/3626772.3657957](https://doi.org/10.1145/3626772.3657957)

+   Schick 等 (2023) Timo Schick, Jane Dwivedi-Yu, Roberto Dessi, Roberta Raileanu, Maria Lomeli, Eric Hambro, Luke Zettlemoyer, Nicola Cancedda, 和 Thomas Scialom. 2023. Toolformer：语言模型能自学使用工具. 收录于 *第37届神经信息处理系统会议*. [https://openreview.net/forum?id=Yacmpz84TH](https://openreview.net/forum?id=Yacmpz84TH)

+   Sclar 等 (2023) Melanie Sclar, Sachin Kumar, Peter West, Alane Suhr, Yejin Choi, 和 Yulia Tsvetkov. 2023. 关注语言模型（缺乏）心智理论：一个即插即用的多角色信念追踪器. 收录于 *第61届计算语言学协会年会论文集（第1卷：长篇论文）*, Anna Rogers, Jordan Boyd-Graber 和 Naoaki Okazaki（编辑）. 计算语言学协会，加拿大多伦多，13960–13980页. [https://doi.org/10.18653/v1/2023.acl-long.780](https://doi.org/10.18653/v1/2023.acl-long.780)

+   Shen 等 (2023) Yongliang Shen, Kaitao Song, Xu Tan, Dongsheng Li, Weiming Lu, 和 Yueting Zhuang. 2023. HuggingGPT：用 ChatGPT 和 Hugging Face 中的伙伴解决 AI 任务. 收录于 *第37届神经信息处理系统会议*. [https://openreview.net/forum?id=yHdTscY6Ci](https://openreview.net/forum?id=yHdTscY6Ci)

+   Shinn 等 (2024) Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik Narasimhan, 和 Shunyu Yao. 2024. Reflexion：具有语言强化学习的语言代理. 收录于 *第37届国际神经信息处理系统会议论文集*（美国路易斯安那州新奥尔良） *(NIPS ’23)*. Curran Associates Inc., 美国纽约红钩，文章377，19页。

+   Shuster 等 (2021) Kurt Shuster, Spencer Poff, Moya Chen, Douwe Kiela, 和 Jason Weston. 2021. 检索增强减少对话中的幻觉. 收录于 *计算语言学协会会议成果：EMNLP 2021*, Marie-Francine Moens, Xuanjing Huang, Lucia Specia 和 Scott Wen-tau Yih（编辑）. 计算语言学协会，多米尼加共和国蓬塔卡纳，3784–3803页. [https://doi.org/10.18653/v1/2021.findings-emnlp.320](https://doi.org/10.18653/v1/2021.findings-emnlp.320)

+   Si et al. (2023) Chenglei Si, Weijia Shi, Chen Zhao, Luke Zettlemoyer, 和 Jordan Lee Boyd-Graber. 2023. 《从语言模型推理专家的混合体中获得更多》（Getting MoRE out of Mixture of Language Model Reasoning Experts）。发表于 *2023年自然语言处理实证方法大会（EMNLP 2023）*。[https://openreview.net/forum?id=UMywlqrW3n](https://openreview.net/forum?id=UMywlqrW3n)

+   Son et al. (2024) Guijin Son, SangWon Baek, Sangdae Nam, Ilgyun Jeong, 和 Seungone Kim. 2024. 《多任务推理：大型语言模型能否同时执行多项指令？》（Multi-Task Inference: Can Large Language Models Follow Multiple Instructions at Once?）。发表于 *第62届计算语言学协会年会论文集（第1卷：长篇论文）*，Lun-Wei Ku, Andre Martins, 和 Vivek Srikumar（编）。计算语言学协会，泰国曼谷，5606–5627。[https://doi.org/10.18653/v1/2024.acl-long.304](https://doi.org/10.18653/v1/2024.acl-long.304)

+   Song et al. (2023) C. Song, B. M. Sadler, J. Wu, W. Chao, C. Washington, 和 Y. Su. 2023. 《LLM-Planner：使用大型语言模型进行面向具体任务的少样本规划》（LLM-Planner: Few-Shot Grounded Planning for Embodied Agents with Large Language Models）。发表于 *2023年IEEE/CVF计算机视觉国际会议（ICCV 2023）*。IEEE计算机学会，美国加州洛斯阿拉米托斯，2986–2997。[https://doi.org/10.1109/ICCV51070.2023.00280](https://doi.org/10.1109/ICCV51070.2023.00280)

+   Srivastava 等人（2023）Aarohi Srivastava, Abhinav Rastogi, Abhishek Rao, Abu Awal Md Shoeb, Abubakar Abid, Adam Fisch, Adam R. Brown, Adam Santoro, Aditya Gupta, Adrià Garriga-Alonso, Agnieszka Kluska, Aitor Lewkowycz, Akshat Agarwal, Alethea Power, Alex Ray, Alex Warstadt, Alexander W. Kocurek, Ali Safaya, Ali Tazarv, Alice Xiang, Alicia Parrish, Allen Nie, Aman Hussain, Amanda Askell, Amanda Dsouza, Ambrose Slone, Ameet Rahane, Anantharaman S. Iyer, Anders Johan Andreassen, Andrea Madotto, Andrea Santilli, Andreas Stuhlmüller, Andrew M. Dai, Andrew La, Andrew Lampinen, Andy Zou, Angela Jiang, Angelica Chen, Anh Vuong, Animesh Gupta, Anna Gottardi, Antonio Norelli, Anu Venkatesh, Arash Gholamidavoodi, Arfa Tabassum, Arul Menezes, Arun Kirubarajan, Asher Mullokandov, Ashish Sabharwal, Austin Herrick, Avia Efrat, Aykut Erdem, Ayla Karakaş, B. Ryan Roberts, Bao Sheng Loe, Barret Zoph, Bartłomiej Bojanowski, Batuhan Özyurt, Behnam Hedayatnia, Behnam Neyshabur, Benjamin Inden, Benno Stein, Berk Ekmekci, Bill Yuchen Lin, Blake Howald, Bryan Orinion, Cameron Diao, Cameron Dour, Catherine Stinson, Cedrick Argueta, Cesar Ferri, Chandan Singh, Charles Rathkopf, Chenlin Meng, Chitta Baral, Chiyu Wu, Chris Callison-Burch, Christopher Waites, Christian Voigt, Christopher D Manning, Christopher Potts, Cindy Ramirez, Clara E. Rivera, Clemencia Siro, Colin Raffel, Courtney Ashcraft, Cristina Garbacea, Damien Sileo, Dan Garrette, Dan Hendrycks, Dan Kilman, Dan Roth, C. Daniel Freeman, Daniel Khashabi, Daniel Levy, Daniel Moseguí González, Danielle Perszyk, Danny Hernandez, Danqi Chen, Daphne Ippolito, Dar Gilboa, David Dohan, David Drakard, David Jurgens, Debajyoti Datta, Deep Ganguli, Denis Emelin, Denis Kleyko, Deniz Yuret, Derek Chen, Derek Tam, Dieuwke Hupkes, Diganta Misra, Dilyar Buzan, Dimitri Coelho Mollo, Diyi Yang, Dong-Ho Lee, Dylan Schrader, Ekaterina Shutova, Ekin Dogus Cubuk, Elad Segal, Eleanor Hagerman, Elizabeth Barnes, Elizabeth Donoway, Ellie Pavlick, Emanuele Rodolà, Emma Lam, Eric Chu, Eric Tang, Erkut Erdem, Ernie Chang, Ethan A Chi, Ethan Dyer, Ethan Jerzak, Ethan Kim, Eunice Engefu Manyasi, Evgenii Zheltonozhskii, Fanyue Xia, Fatemeh Siar, Fernando Martínez-Plumed, Francesca Happé, Francois Chollet, Frieda Rong, Gaurav Mishra, Genta Indra Winata, Gerard de Melo, Germán Kruszewski, Giambattista Parascandolo, Giorgio Mariani, Gloria Xinyue Wang, Gonzalo Jaimovitch-Lopez, Gregor Betz, Guy Gur-Ari, Hana Galijasevic, Hannah Kim, Hannah Rashkin, Hannaneh Hajishirzi, Harsh Mehta, Hayden Bogar, Henry Francis Anthony Shevlin, Hinrich Schuetze, Hiromu Yakura, Hongming Zhang, Hugh Mee Wong, Ian Ng, Isaac Noble, Jaap Jumelet, Jack Geissinger, Jackson Kernion, Jacob Hilton, Jaehoon Lee, Jaime Fernández Fisac, James B Simon, James Koppel, James Zheng, James Zou, Jan Kocon, Jana Thompson, Janelle Wingfield, Jared Kaplan, Jarema Radom, Jascha Sohl-Dickstein, Jason Phang, Jason Wei, Jason Yosinski, Jekaterina Novikova, Jelle Bosscher, Jennifer Marsh, Jeremy Kim, Jeroen Taal, Jesse Engel, Jesujoba Alabi, Jiacheng Xu, Jiaming Song, Jillian Tang, Joan Waweru, John Burden, John Miller, John U. Balis, Jonathan Batchelder, Jonathan Berant, Jörg Frohberg, Jos Rozen, Jose Hernandez-Orallo, Joseph Boudeman, Joseph Guerr, Joseph Jones, Joshua B. Tenenbaum, Joshua S. Rule, Joyce Chua, Kamil Kanclerz, Karen Livescu, Karl Krauth, Karthik Gopalakrishnan, Katerina Ignatyeva, Katja Markert, Kaustubh Dhole, Kevin Gimpel, Kevin Omondi, Kory Wallace Mathewson, Kristen Chiafullo, Ksenia Shkaruta, Kumar Shridhar, Kyle McDonell, Kyle Richardson, Laria Reynolds, Leo Gao, Li Zhang, Liam Dugan, Lianhui Qin, Lidia Contreras-Ochando, Louis-Philippe Morency, Luca Moschella, Lucas Lam, Lucy Noble, Ludwig Schmidt, Luheng He, Luis Oliveros-Colón, Luke Metz, Lütfi Kerem Senel, Maarten Bosma, Maarten Sap, Maartje Ter Hoeve, Maheen Farooqi, Manaal Faruqui, Mantas Mazeika, Marco Baturan, Marco Marelli, Marco Maru, Maria Jose Ramirez-Quintana, Marie Tolkiehn, Mario Giulianelli, Martha Lewis, Martin Potthast, Matthew L Leavitt, Matthias Hagen, Mátyás Schubert, Medina Orduna Baitemirova, Melody Arnaud, Melvin McElrath, Michael Andrew Yee, Michael Cohen, Michael Gu, Michael Ivanitskiy, Michael Starritt, Michael Strube, Michał Sw\kedrowski, Michele Bevilacqua, Michihiro Yasunaga, Mihir Kale, Mike Cain, Mimee Xu, Mirac Suzgun, Mitch Walker, Mo Tiwari, Mohit Bansal, Moin Aminnaseri, Mor Geva, Mozhdeh Gheini, Mukund Varma T, Nanyun Peng, Nathan Andrew Chi, Nayeon Lee, Neta Gur-Ari Krakover, Nicholas Cameron, Nicholas Roberts, Nick Doiron, Nicole Martinez, Nikita Nangia, Niklas Deckers, Niklas Muennighoff, Nitish Shirish Keskar, Niveditha S. Iyer, Noah Constant, Noah Fiedel, Nuan Wen, Oliver Zhang, Omar Agha, Omar Elbaghdadi, Omer Levy, Owain Evans, Pablo Antonio Moreno Casares, Parth Doshi, Pascale Fung, Paul Pu Liang, Paul Vicol, Pegah Alipoormolabashi, Peiyuan Liao, Percy Liang, Peter W Chang, Peter Eckersley, Phu Mon Htut, Pinyu Hwang, Piotr Miłkowski, Piyush Patil, Pouya Pezeshkpour, Priti Oli, Qiaozhu Mei, Qing Lyu, Qinlang Chen, Rabin Banjade, Rachel Etta Rudolph, Raefer Gabriel, Rahel Habacker, Ramon Risco, Raphaël Millière, Rhythm Garg, Richard Barnes, Rif A. Saurous, Riku Arakawa, Robbe Raymaekers, Robert Frank, Rohan Sikand, Roman Novak, Roman Sitelew, Ronan Le Bras, Rosanne Liu, Rowan Jacobs, Rui Zhang, Russ Salakhutdinov, Ryan Andrew Chi, Seungjae Ryan Lee, Ryan Stovall, Ryan Teehan, Rylan Yang, Sahib Singh, Saif M. Mohammad, Sajant Anand, Sam Dillavou, Sam Shleifer, Sam Wiseman, Samuel Gruetter, Samuel R. Bowman, Samuel Stern Schoenholz, Sanghyun Han, Sanjeev Kwatra, Sarah A. Rous, Sarik Ghazarian, Sayan Ghosh, Sean Casey, Sebastian Bischoff, Sebastian Gehrmann, Sebastian Schuster, Sepideh Sadeghi, Shadi Hamdan, Sharon Zhou, Shashank Srivastava, Sherry Shi, Shikhar Singh, Shima Asaadi, Shixiang Shane Gu, Shubh Pachchigar, Shubham Toshniwal, Shyam Upadhyay, Shyamolima Shammie Debnath, Siamak Shakeri, Simon Thormeyer, Simone Melzi, Siva Reddy, Sneha Priscilla Makini, Soo-Hwan Lee, Spencer Torene, Sriharsha Hatwar, Stanislas Dehaene, Stefan Divic, Stefano Ermon, Stella Biderman, Stephanie Lin, Stephen Prasad, Steven Piantadosi, Stuart Shieber, Summer Misherghi, Svetlana Kiritchenko, Swaroop Mishra, Tal Linzen, Tal Schuster, Tao Li, Tao Yu, Tariq Ali, Tatsunori Hashimoto, Te-Lin Wu, Théo Desbordes, Theodore Rothschild, Thomas Phan, Tianle Wang, Tiberius Nkinyili, Timo Schick, Timofei Kornev, Titus Tunduny, Tobias

+   Tam 等人（2024）Zhi Rui Tam、Cheng-Kuang Wu、Yi-Lin Tsai、Chieh-Yen Lin、Hung-yi Lee 和 Yun-Nung Chen。2024年。**让我自由表达？**关于格式限制对大型语言模型性能影响的研究。载于 *2024年自然语言处理实证方法会议：行业专场论文集*，Franck Dernoncourt、Daniel Preoţiuc-Pietro 和 Anastasia Shimorina（编辑）。计算语言学会，迈阿密，佛罗里达州，美国，1218–1236。[https://doi.org/10.18653/v1/2024.emnlp-industry.91](https://doi.org/10.18653/v1/2024.emnlp-industry.91)

+   Team et al. (2024) Gemini Team, Rohan Anil, Sebastian Borgeaud, Jean-Baptiste Alayrac, Jiahui Yu, Radu Soricut, Johan Schalkwyk, Andrew M. Dai, Anja Hauth, Katie Millican, David Silver, Melvin Johnson, Ioannis Antonoglou, Julian Schrittwieser, Amelia Glaese, Jilin Chen, Emily Pitler, Timothy Lillicrap, Angeliki Lazaridou, Orhan Firat, James Molloy, Michael Isard, Paul R. Barham, Tom Hennigan, Benjamin Lee, Fabio Viola, Malcolm Reynolds, Yuanzhong Xu, Ryan Doherty, Eli Collins, Clemens Meyer, Eliza Rutherford, Erica Moreira, Kareem Ayoub, Megha Goel, Jack Krawczyk, Cosmo Du, Ed Chi, Heng-Tze Cheng, Eric Ni, Purvi Shah, Patrick Kane, Betty Chan, Manaal Faruqui, Aliaksei Severyn, Hanzhao Lin, YaGuang Li, Yong Cheng, Abe Ittycheriah, Mahdis Mahdieh, Mia Chen, Pei Sun, Dustin Tran, Sumit Bagri, Balaji Lakshminarayanan, Jeremiah Liu, Andras Orban, Fabian Güra, Hao Zhou, Xinying Song, Aurelien Boffy, Harish Ganapathy, Steven Zheng, HyunJeong Choe, Ágoston Weisz, Tao Zhu, Yifeng Lu, Siddharth Gopal, Jarrod Kahn, Maciej Kula, Jeff Pitman, Rushin Shah, Emanuel Taropa, Majd Al Merey, Martin Baeuml, Zhifeng Chen, Laurent El Shafey, Yujing Zhang, Olcan Sercinoglu, George Tucker, Enrique Piqueras, Maxim Krikun, Iain Barr, Nikolay Savinov, Ivo Danihelka, Becca Roelofs, Anaïs White, Anders Andreassen, Tamara von Glehn, Lakshman Yagati, Mehran Kazemi, Lucas Gonzalez, Misha Khalman, Jakub Sygnowski, Alexandre Frechette, Charlotte Smith, Laura Culp, Lev Proleev, Yi Luan, Xi Chen, James Lottes, Nathan Schucher, Federico Lebron, Alban Rrustemi, Natalie Clay, Phil Crone, Tomas Kocisky, Jeffrey Zhao, Bartek Perz, Dian Yu, Heidi Howard, Adam Bloniarz, Jack W. Rae, Han Lu, Laurent Sifre, Marcello Maggioni, Fred Alcober, Dan Garrette, Megan Barnes, Shantanu Thakoor, Jacob Austin, Gabriel Barth-Maron, William Wong, Rishabh Joshi, Rahma Chaabouni, Deeni Fatiha, Arun Ahuja, Gaurav Singh Tomar, Evan Senter, Martin Chadwick, Ilya Kornakov, Nithya Attaluri, Iñaki Iturrate, Ruibo Liu, Yunxuan Li, Sarah Cogan, Jeremy Chen, Chao Jia, Chenjie Gu, Qiao Zhang, Jordan Grimstad, Ale Jakse Hartman, Xavier Garcia, Thanumalayan Sankaranarayana Pillai, Jacob Devlin, Michael Laskin, Diego de Las Casas, Dasha Valter, Connie Tao, Lorenzo Blanco, Adrià Puigdomènech Badia, David Reitter, Mianna Chen, Jenny Brennan, Clara Rivera, Sergey Brin, Shariq Iqbal, Gabriela Surita, Jane Labanowski, Abhi Rao, Stephanie Winkler, Emilio Parisotto, Yiming Gu, Kate Olszewska, Ravi Addanki, Antoine Miech, Annie Louis, Denis Teplyashin, Geoff Brown, Elliot Catt, Jan Balaguer, Jackie Xiang, Pidong Wang, Zoe Ashwood, Anton Briukhov, Albert Webson, Sanjay Ganapathy, Smit Sanghavi, Ajay Kannan, Ming-Wei Chang, Axel Stjerngren, Josip Djolonga, Yuting Sun, Ankur Bapna, Matthew Aitchison, Pedram Pejman, Henryk Michalewski, Tianhe Yu, Cindy Wang, Juliette Love, Junwhan Ahn, Dawn Bloxwich, Kehang Han, Peter Humphreys, Thibault Sellam, James Bradbury, Varun Godbole, Sina Samangooei, Bogdan Damoc, Alex Kaskasoli, Sébastien M. R. Arnold, Vijay Vasudevan, Shubham Agrawal, Jason Riesa, Dmitry Lepikhin, Richard Tanburn, Srivatsan Srinivasan, Hyeontaek Lim, Sarah Hodkinson, Pranav Shyam, Johan Ferret, Steven Hand, Ankush Garg, Tom Le Paine, Jian Li, Yujia Li, Minh Giang, Alexander Neitz, Zaheer Abbas, Sarah York, Machel Reid, Elizabeth Cole, Aakanksha Chowdhery, Dipanjan Das, Dominika Rogozińska, Vitaliy Nikolaev, Pablo Sprechmann, Zachary Nado, Lukas Zilka, Flavien Prost, Luheng He, Marianne Monteiro, Gaurav Mishra, Chris Welty, Josh Newlan, Dawei Jia, Miltiadis Allamanis, Clara Huiyi Hu, Raoul de Liedekerke, Justin Gilmer, Carl Saroufim, Shruti Rijhwani, Shaobo Hou, Disha Shrivastava, Anirudh Baddepudi, Alex Goldin, Adnan Ozturel, Albin Cassirer, Yunhan Xu, Daniel Sohn, Devendra Sachan, Reinald Kim Amplayo, Craig Swanson, Dessie Petrova, Shashi Narayan, Arthur Guez, Siddhartha Brahma, Jessica Landon, Miteyan Patel, Ruizhe Zhao, Kevin Villela, Luyu Wang, Wenhao Jia, Matthew Rahtz, Mai Giménez, Legg Yeung, James Keeling, Petko Georgiev, Diana Mincu, Boxi Wu, Salem Haykal, Rachel Saputro, Kiran Vodrahalli, James Qin, Zeynep Cankara, Abhanshu Sharma, Nick Fernando, Will Hawkins, Behnam Neyshabur, Solomon Kim, Adrian Hutter, Priyanka Agrawal, Alex Castro-Ros, George van den Driessche, Tao Wang, Fan Yang, Shuo yiin Chang, Paul Komarek, Ross McIlroy, Mario Lučić, Guodong Zhang, Wael Farhan, Michael Sharman, Paul Natsev, Paul Michel, Yamini Bansal, Siyuan Qiao, Kris Cao, Siamak Shakeri, Christina Butterfield, Justin Chung, Paul Kishan Rubenstein, Shivani Agrawal, Arthur Mensch, Kedar Soparkar, Karel Lenc, Timothy Chung, Aedan Pope, Loren Maggiore, Jackie Kay, Priya Jhakra, Shibo Wang, Joshua Maynez, Mary Phuong, Taylor Tobin, Andrea Tacchetti, Maja Trebacz, Kevin Robinson, Yash Katariya, Sebastian Riedel, Paige Bailey, Kefan Xiao, Nimesh Ghelani, Lora Aroyo, Ambrose Slone, Neil Houlsby, Xuehan Xiong, Zhen Yang, Elena Gribovskaya, Jonas Adler, Mateo Wirth, Lisa Lee, Music Li, Thais Kagohara, Jay Pavagadhi, Sophie Bridgers, Anna Bortsova, Sanjay Ghemawat, Zafarali Ahmed, Tianqi Liu, Richard Powell, Vijay Bolina, Mariko Iinuma, Polina Zablotskaia, James Besley, Da-Woon Chung, Timothy Dozat, Ramona Comanescu, Xiance Si, Jeremy Greer, Guolong Su, Martin Polacek, Raphaël Lopez Kaufman, Simon Tokumine, Hexiang Hu, Elena Buchatskaya, Yingjie Miao, Mohamed Elhawaty, Aditya Siddhant, Nenad Tomasev, Jinwei Xing, Christina Greer, Helen Miller, Shereen Ashraf, Aurko Roy, Zizhao Zhang, Ada Ma, Angelos Filos, Milos Besta, Rory Blevins, Ted Klimenko, Chih-Kuan Yeh, Soravit Changpinyo, Jiaqi Mu, Oscar Chang, Mantas Pajarskas, Carrie Muir, Vered Cohen, Charline Le Lan, Krishna Haridasan, Amit Marathe, Steven Hansen, Sholto Douglas, Rajkumar Samuel, Mingqiu Wang, Sophia Austin, Chang Lan, Jiepu Jiang, Justin Chiu, Jaime Alonso Lorenzo, Lars Lowe Sjösund, Sébastien Cevey, Zach Gleicher, Thi Avrahami, Anudhyan Boral, Hansa Srinivasan, Vittorio Selo, Rhys May, Konstantinos Aisopos, Léonard Hussenot, Livio Baldini Soares, Kate Baumli, Michael B. Chang, Adrià Recasens, Ben Caine, Alexander Pritzel, Filip Pavetic, Fabio Pardo, Anita Gergely, Justin Frye, Vinay Ramasesh, Dan Horgan, Kartikeya Badola, Nora Kassner, Subhrajit Roy, Ethan Dyer, Víctor Campos Campos, Alex Tomala, Yunhao Tang, Dalia El Badawy, El

+   Tyen等人（2024）Gladys Tyen, Hassan Mansoor, Victor Carbune, Peter Chen, 和 Tony Mak. 2024. LLM不能发现推理错误，但可以在给定错误位置的情况下纠正它们。发表于*计算语言学协会的发现：ACL 2024*，Lun-Wei Ku, Andre Martins, 和 Vivek Srikumar（编）。计算语言学协会，泰国曼谷，13894–13908. [https://doi.org/10.18653/v1/2024.findings-acl.826](https://doi.org/10.18653/v1/2024.findings-acl.826)

+   Valmeekam等人（2024）Karthik Valmeekam, Matthew Marquez, Sarath Sreedharan, 和 Subbarao Kambhampati. 2024. 大语言模型的规划能力：一个批判性的研究。发表于*第37届神经信息处理系统国际会议论文集*（美国路易斯安那州新奥尔良）*(NIPS ’23)*. Curran Associates Inc., 美国纽约红钩，文章3320，13页。

+   Valmeekam等人（2022）Karthik Valmeekam, Alberto Olmo, Sarath Sreedharan, 和 Subbarao Kambhampati. 2022. 大语言模型仍然无法进行规划（一个关于规划和推理变化的LLM基准测试）。在*NeurIPS 2022决策模型基础工作坊*中. [https://openreview.net/forum?id=wUU-7XTL5XO](https://openreview.net/forum?id=wUU-7XTL5XO)

+   Varshney（2023）Tanay Varshney. 2023. LLM代理简介. 博客. 2024年10月7日检索自[https://developer.nvidia.com/blog/introduction-to-llm-agents/](https://developer.nvidia.com/blog/introduction-to-llm-agents/)

+   Vaswani等人（2017）Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N Gomez, Ł ukasz Kaiser, 和 Illia Polosukhin. 2017. 注意力机制是你所需要的全部。在*神经信息处理系统进展*中，I. Guyon, U. Von Luxburg, S. Bengio, H. Wallach, R. Fergus, S. Vishwanathan, 和 R. Garnett（编），第30卷. Curran Associates, Inc. [https://proceedings.neurips.cc/paper_files/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf](https://proceedings.neurips.cc/paper_files/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf)

+   Wang等人（2023a）Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao, Yuke Zhu, Linxi (Jim) Fan, 和 Anima Anandkumar. 2023a. Voyager: 一个基于大语言模型的开放式具身代理。*机器学习研究汇刊* 2024 (2023年). [https://api.semanticscholar.org/CorpusID:258887849](https://api.semanticscholar.org/CorpusID:258887849)

+   Wang等人（2024c）Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin, Wayne Xin Zhao, Zhewei Wei, 和 Jirong Wen. 2024c. 基于大语言模型的自主代理的调查。*计算机科学前沿* 18, 6 (2024年3月). [https://doi.org/10.1007/s11704-024-40231-1](https://doi.org/10.1007/s11704-024-40231-1)

+   Wang 等人（2023b）王磊、徐婉玉、蓝一槐、胡志强、蓝云狮、李凯伟和林亦鹏。2023b。计划与解决提示：通过大型语言模型改进零-shot 连锁思维推理。在 *第61届计算语言学协会年会（第1卷：长篇论文）* 中，Anna Rogers、Jordan Boyd-Graber 和 Naoaki Okazaki（编辑）。计算语言学协会，多伦多，加拿大，2609–2634。[https://doi.org/10.18653/v1/2023.acl-long.147](https://doi.org/10.18653/v1/2023.acl-long.147)

+   Wang 等人（2024a）王星耀、陈阳毅、袁丽凡、张一哲、李云竹、彭浩和纪衡。2024a。可执行代码行动激发更好的 LLM 代理。在 *第41届国际机器学习会议* 中。[https://openreview.net/forum?id=jJ9BoXAfFa](https://openreview.net/forum?id=jJ9BoXAfFa)

+   Wang 等人（2023c）王宇飞、钟婉君、李良友、米飞、曾兴山、黄文勇、尚立峰、姜鑫和刘群。2023c。将大型语言模型与人类对齐：一项调查。arXiv:2307.12966 [cs.CL] [https://arxiv.org/abs/2307.12966](https://arxiv.org/abs/2307.12966)

+   Wang 等人（2024b）王志若、程周俊、朱浩、Daniel Fried 和 Graham Neubig。2024b。工具到底是什么？来自语言模型视角的调查。在 *第一次语言建模会议* 中。[https://openreview.net/forum?id=Xh1B90iBSR](https://openreview.net/forum?id=Xh1B90iBSR)

+   Wang 等人（2024d）王震海龙、毛少光、吴文山、葛涛、魏富如和纪衡。2024d。释放大型语言模型中的突现认知协同：通过多人格自我协作的任务解决代理。在 *2024年北美计算语言学协会会议：人类语言技术（第1卷：长篇论文）* 中，Kevin Duh、Helena Gomez 和 Steven Bethard（编辑）。计算语言学协会，墨西哥城，墨西哥，257–279。[https://doi.org/10.18653/v1/2024.naacl-long.15](https://doi.org/10.18653/v1/2024.naacl-long.15)

+   Weng（2023）Lilian Weng。2023。LLM 驱动的自主代理。博客。2024年10月8日从[https://lilianweng.github.io/posts/2023-06-23-agent/](https://lilianweng.github.io/posts/2023-06-23-agent/)检索。

+   Wooldridge 和 Jennings（1995）Michael Wooldridge 和 Nicholas R. Jennings。1995。智能代理：理论与实践。*知识工程评论* 10, 2（1995），115–152。[https://doi.org/10.1017/S0269888900008122](https://doi.org/10.1017/S0269888900008122)

+   Wu 等人（2024a）吴清云、加甘·班萨尔、张杰宇、吴怡然、李贝宾、朱尔康（Eric）、姜立、张晓云、张绍坤、艾哈迈德·阿瓦达拉、Ryen W. White、道格·伯杰和王驰。2024a。AutoGen：通过多代理对话实现下一代 LLM 应用。在 *COLM 2024* 中。

+   Wu 等人 (2024b) Yue Wu, Xuan Tang, Tom Mitchell, 和 Yuanzhi Li. 2024b. SmartPlay: 用作智能代理的大型语言模型基准测试. 发表在 *第十二届国际学习表示会议* 上. [https://openreview.net/forum?id=S2oTVrlcp3](https://openreview.net/forum?id=S2oTVrlcp3)

+   Xi 等人 (2023) Zhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, Boyang Hong, Ming Zhang, Junzhe Wang, Senjie Jin, Enyu Zhou, Rui Zheng, Xiaoran Fan, Xiao Wang, Limao Xiong, Yuhao Zhou, Weiran Wang, Changhao Jiang, Yicheng Zou, Xiangyang Liu, Zhangyue Yin, Shihan Dou, Rongxiang Weng, Wensen Cheng, Qi Zhang, Wenjuan Qin, Yongyan Zheng, Xipeng Qiu, Xuanjing Huang, 和 Tao Gui. 2023. 基于大型语言模型代理的崛起与潜力：一项调查. arXiv:2309.07864 [cs.AI] [https://arxiv.org/abs/2309.07864](https://arxiv.org/abs/2309.07864)

+   Xiong 等人 (2024) Zheyang Xiong, Ziyang Cai, John Cooper, Albert Ge, Vasilis Papageorgiou, Zack Sifakis, Angeliki Giannou, Ziqian Lin, Liu Yang, Saurabh Agarwal, Grigorios G Chrysos, Samet Oymak, Kangwook Lee, 和 Dimitris Papailiopoulos. 2024. 《无处不在，瞬时发生：LLMs 可以在超位态中同时学习多项任务》. arXiv:2410.05603 [cs.LG] [https://arxiv.org/abs/2410.05603](https://arxiv.org/abs/2410.05603)

+   Xu 等人 (2023) Benfeng Xu, An Yang, Junyang Lin, Quan Wang, Chang Zhou, Yongdong Zhang, 和 Zhendong Mao. 2023. ExpertPrompting: 指导大型语言模型成为卓越专家. arXiv:2305.14688 [cs.CL] [https://arxiv.org/abs/2305.14688](https://arxiv.org/abs/2305.14688)

+   Yang 等人 (2024) Ke Yang, Jiateng Liu, John Wu, Chaoqi Yang, Yi R. Fung, Sha Li, Zixuan Huang, Xu Cao, Xingyao Wang, Yiquan Wang, Heng Ji, 和 Chengxiang Zhai. 2024. 如果 LLM 是魔法师，那么代码就是魔杖：一项关于代码如何赋能大型语言模型以作为智能代理的调查. arXiv:2401.00812 [cs.CL] [https://arxiv.org/abs/2401.00812](https://arxiv.org/abs/2401.00812)

+   Yao 等人 (2022) Shunyu Yao, Howard Chen, John Yang, 和 Karthik Narasimhan. 2022. WebShop: 面向可扩展的现实世界网页交互与基础语言代理. 发表在 *神经信息处理系统进展 35 - 第36届神经信息处理系统会议, NeurIPS 2022* *（神经信息处理系统进展）* 中，S. Koyejo, S. Mohamed, A. Agarwal, D. Belgrave, K. Cho, 和 A. Oh (编辑). 神经信息处理系统基金会出版. 出版版权：© 2022 神经信息处理系统基金会。保留所有权利。; 第36届神经信息处理系统会议, NeurIPS 2022 ; 会议日期：2022年11月28日至2022年12月9日。

+   Yao 等人 (2023) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R Narasimhan, 和 Yuan Cao. 2023. ReAct: 协同推理与执行的语言模型. 发表在 *第十一届国际学习表示会议* 上. [https://openreview.net/forum?id=WE_vluYUL-X](https://openreview.net/forum?id=WE_vluYUL-X)

+   Zhang等人（2024b）Kechi Zhang、Jia Li、Ge Li、Xianjie Shi和Zhi Jin。2024b。CodeAgent：通过工具集成代理系统增强代码生成，解决现实世界的仓库级编码挑战。在*第62届计算语言学协会年会论文集（卷1：长篇论文）*中，Lun-Wei Ku、Andre Martins和Vivek Srikumar（编辑）。计算语言学协会，泰国曼谷，13643–13658。[https://doi.org/10.18653/v1/2024.acl-long.737](https://doi.org/10.18653/v1/2024.acl-long.737)

+   Zhang等人（2024a）Li Zhang、Peter Jansen、Tianyi Zhang、Peter Clark、Chris Callison-Burch和Niket Tandon。2024a。PDDLEGO：文本环境中的迭代规划。在*第13届联合词汇与计算语义学会议（*SEM 2024）*上，Danushka Bollegala和Vered Shwartz（编辑）。计算语言学协会，墨西哥城，墨西哥，212–221。[https://doi.org/10.18653/v1/2024.starsem-1.17](https://doi.org/10.18653/v1/2024.starsem-1.17)

+   Zhang等人（2024c）Yadong Zhang、Shaoguang Mao、Tao Ge、Xun Wang、Yan Xia、Wenshan Wu、Ting Song、Man Lan和Furu Wei。2024c。LLM作为战略 mastermind：使用大规模语言模型进行战略推理的调研。在*第一届语言建模会议*上。[https://openreview.net/forum?id=iMqJsQ4evS](https://openreview.net/forum?id=iMqJsQ4evS)

+   Zhong等人（2024）Wanjun Zhong、Lianghong Guo、Qiqi Gao、He Ye和Yanlin Wang。2024。MemoryBank：通过长期记忆增强大规模语言模型。*2024年AAAI人工智能会议论文集*，38卷，17期（2024年3月），19724–19731。[https://doi.org/10.1609/aaai.v38i17.29946](https://doi.org/10.1609/aaai.v38i17.29946)

+   Zhou等人（2023）Denny Zhou、Nathanael Schärli、Le Hou、Jason Wei、Nathan Scales、Xuezhi Wang、Dale Schuurmans、Claire Cui、Olivier Bousquet、Quoc V Le和Ed H. Chi。2023。最小到最多提示法使大规模语言模型能够进行复杂推理。在*第十一届国际学习表征会议*上。[https://openreview.net/forum?id=WZH7099tgfM](https://openreview.net/forum?id=WZH7099tgfM)
