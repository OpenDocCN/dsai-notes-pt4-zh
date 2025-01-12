<!--yml

category: 未分类

日期：2025-01-11 12:02:25

-->

# VisionCoder：通过混合大型语言模型赋能多智能体自动编程用于图像处理

> 来源：[https://arxiv.org/html/2410.19245/](https://arxiv.org/html/2410.19245/)

Zixiao Zhao 奥克兰大学 新西兰奥克兰 [zixiao.zhao@auckland.ac.nz](mailto:zixiao.zhao@auckland.ac.nz)，Jing Sun 奥克兰大学 新西兰奥克兰，Zhiyuan Wei 北京理工大学 中国北京，Cheng-Hao Cai 苏州工业园区 莫纳什科技研究院 江苏省苏州，中国，Zhe Hou 格里菲斯大学 澳大利亚昆士兰 和 Jin Song Dong 新加坡国立大学 新加坡（2018）

###### 摘要

在自动编程领域，当给定详细的任务描述时，大型语言模型（LLMs）展示了基础的生成能力。然而，它们当前的功能主要局限于函数级开发，这限制了它们在复杂项目环境和特定应用场景中的效果，比如复杂的图像处理任务。本文提出了一种多智能体框架，利用混合大型语言模型（包括GPT-4o和本地部署的开源模型），这些模型协作完成自动编程任务。每个智能体在软件开发周期中扮演着不同的角色，共同构成一个虚拟组织，协同工作以生产软件产品。通过在项目、模块和功能层面建立树状的思维分配与开发机制，该框架为代码生成提供了一个高效且具有成本效益的解决方案。我们通过基准数据集评估了我们的方法，实验结果表明，VisionCoder在图像处理自动编程任务中显著优于现有方法。

生成性AI，自动编程，GPT，大型语言模型，图像处理^†^†版权：acm许可^††期刊年份：2018^††doi：XXXXXXX.XXXXXXX^††isbn：978-1-4503-XXXX-X/18/06^††ccs：软件及其工程 自动编程^††ccs：计算方法 自然语言处理

## 1\. 引言

图像处理技术在许多领域中至关重要，包括材料科学、生物医学和农业信息学（Choudhary 等，[2022](https://arxiv.org/html/2410.19245v1#bib.bib7)；Suganyadevi 等，[2022](https://arxiv.org/html/2410.19245v1#bib.bib37)；Latha 等，[2014](https://arxiv.org/html/2410.19245v1#bib.bib22)）。通过成熟的算法，研究人员可以通过自动化过程提高检测特征和分析图像中的统计数据的效率（Prabaharan 等，[2020](https://arxiv.org/html/2410.19245v1#bib.bib31)）。尽管现有工具通常与图像分析软件集成，允许研究人员通过理解基本概念和操作来获取数据（Schroeder 等，[2021](https://arxiv.org/html/2410.19245v1#bib.bib36)；Yushkevich 等，[2016](https://arxiv.org/html/2410.19245v1#bib.bib50)），但在某些任务中，由于实现限制，它们的功能或效率可能不足，无法达到最佳效果（Gómez-de Mariscal 等，[2021](https://arxiv.org/html/2410.19245v1#bib.bib15)）。这种局限性促使研究人员获得跨学科的编程技能，以开发定制的图像相关算法，满足他们的特定需求。尽管有针对新手的辅助编程工具（Imai，[2022](https://arxiv.org/html/2410.19245v1#bib.bib18)）和辅导资源（Hellas 等，[2023](https://arxiv.org/html/2410.19245v1#bib.bib16)），但对于没有计算机视觉或图像处理背景的人来说，学习项目特定的编程仍然是一个重大挑战。这些端到端系统功能通常需要大量的时间和精力来开发，甚至在软件开发公司内部，也往往需要开发团队。因此，开发这种复杂软件系统的自动化仍然没有得到充分探索。

在过去的十年中，由机器学习驱动的自然语言处理（NLP）已经成为一种有前景的自动编程方法（Dehaerne 等，[2022](https://arxiv.org/html/2410.19245v1#bib.bib10)）。这些模型通过学习如何将自然语言任务描述转化为编程代码，克服了早期模板匹配方法的局限性。大语言模型（LLMs）如 GPT 的引入（Floridi 和 Chiriatti，[2020](https://arxiv.org/html/2410.19245v1#bib.bib14)）在 2020 年后进一步推动了这一领域的发展，显著提升了自动编程的性能（Jiang 等，[2024](https://arxiv.org/html/2410.19245v1#bib.bib19)）。这些 LLMs 通过数十亿个参数和数百万个代码示例进行训练，并结合人类反馈，使得通用自动化代码生成更加高效（Liu 等，[2023](https://arxiv.org/html/2410.19245v1#bib.bib27)）。然而，在处理复杂的大型软件系统项目时，例如复杂图像处理领域中的自动编程，这些模型的能力仍然有限，主要是由于其训练数据的规模限制。

在本研究中，我们提出了一种将复杂图像处理任务分解为分支模块的方法，这些模块进一步细分为叶子函数。通过利用大型语言模型（LLMs）处理函数级代码生成的能力，这些碎片化的函数最终被组装成综合系统。在实际操作中，受到思维树（Tree of Thoughts，ToT）方法的启发（Yao等人，[2023](https://arxiv.org/html/2410.19245v1#bib.bib47)），我们提出了一种混合多代理框架VisionCoder，该框架利用树结构的任务分解，专门用于经典图像处理项目的自动化开发。该框架通过建立角色层次结构：团队领导、模块领导、功能协调员和开发小组，模拟了一个真实开发团队的结构。通过利用树结构的多LLM框架，一个复杂项目被分解为较小、可管理的思维单元，这些单元由不同的代理分别实现。一旦各个思维单元并行处理完毕，结果通过逆向工作流汇总，最终生成解决方案。这种分解减少了每个LLM需要处理的复杂度，使得代理能够专注于任务的较小、定义明确的部分，从而实现更精确和高效的结果。此外，我们采用了检索增强生成（RAG）（Lewis等人，[2020](https://arxiv.org/html/2410.19245v1#bib.bib23)）技术，通过提供来自知识库的相关信息，增强关键代理的性能，优化任务分解和功能生成。我们还引入了配对编程，以便实施者代理之间进行交叉检查和验证，进一步提高最终代码的准确性。在这一实际应用中，我们通过分配GPT-4o（Achiam等人，[2023](https://arxiv.org/html/2410.19245v1#bib.bib2)）（专有模型）作为决策角色，以及deepseek-coder-7b-instruct-v1.5（DeepSeek，[2023](https://arxiv.org/html/2410.19245v1#bib.bib9)）（开源模型）作为实施者角色，采用混合模型部署，以实现最佳的成本效益。

通过设计和实现VisionCoder的多代理框架，我们旨在提高图像处理任务自动编程的效率和效果。通过利用层次化任务分解、代理之间的思维流动、互补技术和混合模型选择，我们显著提高了框架的性能和成本效益。本文的主要贡献如下：

+   •

    大型图像处理系统自动编程的创新多代理框架：我们介绍了一个多层结构，支持项目任务的分解和组装。这种分层方法采用树状任务拆解结合辅助策略，使我们的框架在图像处理任务的自动编程中达到最先进的性能。

+   •

    图像处理算法开发的全面知识库：我们创建了一个针对VisionCoder内部代理的知识库，涵盖了图像处理的经典操作，特别是在分解推理和实现方面。该知识库可以集成到其他开发任务模型中，提高代理在各类任务中的多功能性和有效性。

+   •

    图像处理项目的自动编程基准：我们开发了一个专门设计的基准数据集，用于评估图像处理领域中的自动编程解决方案。这个数据集将作为未来研究的重要资源，促进比较并推动该领域的进展。

本文的其余部分安排如下：第2节提供了自动编程的详细背景。第3节集中讨论VisionCoder的工作流程，详细说明了思维如何被分解和分配，然后验证和组装，以及实现细节。在第4节中，我们展示了实验设置，包括BCVPP数据集的构建和VisionCoder的评估，将其性能与其他最先进的方法进行比较。第5节分析了结果，重点阐述了VisionCoder开发中的关键发现，并讨论了框架的局限性。最后，第6节通过总结贡献并概述未来工作的方向来结束本文。

## 2\. 背景

在本节中，我们提供了自动编程任务和方法的演变背景，特别是基于代理的解决方案、多代理框架和路径推理策略。

### 2.1\. 自动编程

自动编程的概念可以追溯到1950年代，当时由于计算机架构的限制，自动编程主要被视为一个编译问题——将形式化的规范转化为实现(Balzer, [1985](https://arxiv.org/html/2410.19245v1#bib.bib4))。随着时间的推移，这一概念演变为一个编程问题，重点是从任务描述中生成领域特定语言(Kushman 和 Barzilay, [2013](https://arxiv.org/html/2410.19245v1#bib.bib21); Raza 等, [2015](https://arxiv.org/html/2410.19245v1#bib.bib33); Manshadi 等, [2013](https://arxiv.org/html/2410.19245v1#bib.bib28))。随着基于深度学习的自然语言处理(NLP)技术的出现，利用RNN和LSTM的算法(Yin 和 Neubig, [2017](https://arxiv.org/html/2410.19245v1#bib.bib48))使得自然语言(NL)到编程语言(PL)的初步直接翻译成为可能。2017年Transformer架构的引入(Vaswani 等, [2017](https://arxiv.org/html/2410.19245v1#bib.bib41))进一步加速了NLP算法的发展，催生了像Intellicode(Svyatkovskiy 等, [2020](https://arxiv.org/html/2410.19245v1#bib.bib38))这样提供多语言建议的工具，以及支持通过代码搜索进行NL-PL转换的CodeBERT(Feng 等, [2020](https://arxiv.org/html/2410.19245v1#bib.bib13))。近期的大型语言模型(LLMs)如CodeLlama(Roziere 等, [2023](https://arxiv.org/html/2410.19245v1#bib.bib35))、AlphaCode(Li 等, [2022](https://arxiv.org/html/2410.19245v1#bib.bib26))和StarCoder(Li 等, [2023](https://arxiv.org/html/2410.19245v1#bib.bib25))利用更大的数据集和更多的参数，在自动编程的准确性和效率上取得了显著进展。这些进展也启发了基于LLM的编程相关工具的开发，如LLM4PR(Cai 等, [2024](https://arxiv.org/html/2410.19245v1#bib.bib5))、InterFix(Jin 等, [2023](https://arxiv.org/html/2410.19245v1#bib.bib20))、Pydex(Zhang 等, [2024a](https://arxiv.org/html/2410.19245v1#bib.bib51))和LLift(Li 等, [2024](https://arxiv.org/html/2410.19245v1#bib.bib24))，它们旨在程序优化、程序修复和错误检测。然而，单个LLM在处理复杂代码生成任务时的能力仍然受到其训练数据限制的制约。

### 2.2\. 基于代理的自动编程解决方案

LLM（大语言模型）通过一次对话生成的代码通常较为简单。为了解决这一局限性，已经开发出一些框架，将LLM作为AI代理，并结合特定的交互策略（Wang等，[2024](https://arxiv.org/html/2410.19245v1#bib.bib42)；Xi等，[2023](https://arxiv.org/html/2410.19245v1#bib.bib46)）。例如，ToolCoder（Zhang等，[2023c](https://arxiv.org/html/2410.19245v1#bib.bib54)）将API搜索策略集成到LLM中，使其在生成代码时能够在线搜索适当的API。类似地，CodeAgent（Zhang等，[2024b](https://arxiv.org/html/2410.19245v1#bib.bib52)）通过集成格式检查器、代码解释器等工具，除了在线搜索外，还增强了LLM，以弥补单一模型的不足。此外，像Self-edit（Zhang等，[2023b](https://arxiv.org/html/2410.19245v1#bib.bib53)）和Self-debug（Chen等，[2023](https://arxiv.org/html/2410.19245v1#bib.bib6)）的方法采用了多轮对话机制，引导LLM迭代优化生成的代码。这些工作表明，除了简单地训练LLM以增强其生成能力外，结合设计良好的机制和策略可以显著提高其整体性能。

### 2.3\. 多代理框架

除了采用交互策略来增强单一代理的能力外，一些研究者提出了使用多个代理组合形成代理组来执行自动化编程任务（Wang等，[2024](https://arxiv.org/html/2410.19245v1#bib.bib42)）。Dong等（Dong等，[2023](https://arxiv.org/html/2410.19245v1#bib.bib11)）介绍了一个框架，在其中每个代理被分配不同的角色，以协同进行代码开发。Chatdev（Qian等，[2024](https://arxiv.org/html/2410.19245v1#bib.bib32)）扩展了这一概念，通过扩展代理的角色并引入代理之间的交互策略来进一步发展这一框架。同时，AgentCoder（Huang等，[2023](https://arxiv.org/html/2410.19245v1#bib.bib17)）基于自动化编程任务的特定需求，增强了测试代理的功能。然而，尽管有角色分配，通常在每个开发阶段只有一个代理处于活动状态。这种方法对单个代理的能力提出了较高要求，限制了其在处理复杂编程任务时的有效性。

### 2.4\. 思维树推理

面对复杂问题时，人类能够将其分解为更简单的步骤，逐步解决。这一策略已被扩展到LLM（大规模语言模型），并被称为路径推理（Wang等， [2024](https://arxiv.org/html/2410.19245v1#bib.bib42)）。这种方法的最早形式是单路径推理，以“思维链”（CoT）方法为代表（Wei等， [2022](https://arxiv.org/html/2410.19245v1#bib.bib44)），该方法通过提示中的逐步示例指导LLM完成任务。此策略发展为多路径推理，例如自一致思维链（CoT-SC）（Wang等， [2022](https://arxiv.org/html/2410.19245v1#bib.bib43)），即为同一问题生成多个推理路径（或CoT），并选择最频繁或最一致的结果作为最终解答。“思维树”（ToT）（Yao等， [2023](https://arxiv.org/html/2410.19245v1#bib.bib47)）进一步优化了这一方法，通过将这些多重推理路径组织成树状结构。树中的每个节点代表一个中间思维，能够回溯或探索，直到找到最佳解决方案。虽然ToT在数学问题求解中尤其有用，但其原理同样为解决复杂编程任务提供了潜力，提供了一种有结构的方式来同时探索多个解决路径。

## 3\. 多智能体自动编程框架与思维分解与组装

![参考说明](img/36a89a0ea49df505c585167ac7282ecd.png)

图1. VisionCoder框架

\描述

工作流

在本节中，我们将详细解释VisionCoder框架。如图[1](https://arxiv.org/html/2410.19245v1#S3.F1 "图1 ‣ 3\. 多智能体自动编程框架与思维分解与组装 ‣ VisionCoder：通过混合LLM赋能图像处理的多智能体自动编程")所示，VisionCoder框架通过一个树状结构的多智能体工作流自动化图像处理软件工程。它从团队负责人将项目需求分解为分支模块开始。模块负责人随后将每个分支进一步分解成更小的、可执行的思维，这些思维会由功能协调员进一步细化以确保清晰，并传递给开发小组作为叶子功能进行实现。开发小组在生成代码后会自我验证每个功能。所有中间思维都会记录在思维池中，确保关键信息被准确存储，并且在后向工作流中能够无误地检索。一旦所有功能实现并验证完成，它们会被组装成模块并进行测试，然后由团队负责人将完成的模块合并为最终的项目代码。

为了展示详细的功能，我们在本节中使用了一个运行时示例，演示框架在实践中的操作方式。用户提供了以下任务：“给定一张图片（`./license_plate.jpg`），开发一个系统，自动检测并识别图片中的车牌（可以使用 easyocr 进行 OCR）。”通过这个示例，我们在第 3.1 节中首先概述了 VisionCoder 的工作流程，重点介绍了两个关键方面：思维分解与分配，以及结果验证与汇总。在第 3.2 节中，我们提供了框架中每个代理的角色和操作机制的全面概述。第 3.3 节探讨了增强核心结构的支持策略，而在第 3.4 节中，我们展示了框架的具体实现。在详细介绍实现后，我们将在第 3.5 节分析 VisionCoder 的一个输出示例，以帮助深入理解其功能。

### 3.1\. VisionCoder 工作流程

#### 3.1.1\. 前向任务流 - 分解与分配

![请参见标题说明](img/54a41e5839a5b9ed1c5a8825a0544811.png)

图 2. 前向任务流：决策代理的任务

每个决策代理的前向任务流如图[2](https://arxiv.org/html/2410.19245v1#S3.F2 "Figure 2 ‣ 3.1.1\. Forward task flow - decomposition and distribution ‣ 3.1\. VisionCoder Workflow ‣ 3\. Multi-Agent Auto-Programming Framework with Thought Decomposition and Assembly ‣ VisionCoder: Empowering Multi-Agent Auto-Programming for Image Processing with Hybrid LLMs")所示。团队领导：VisionCoder 框架的输入是一组以自然语言描述的图像处理算法项目需求。收到这些需求后，团队领导是第一个被调用的代理。其主要任务是将项目分解成不同的分支模块，并定义每个模块的功能范围和目标。为了简化后续模块整合和验证过程，并避免模块之间的冗余，团队领导被设计成生成尽可能少的模块。由于 VisionCoder 框架涉及运行和测试项目代码，因此使用 Docker（Merkel 等，[2014](https://arxiv.org/html/2410.19245v1#bib.bib29)）构建必要的运行时环境，以避免对主机系统产生影响。我们预配置了几个 Docker 镜像，每个镜像包含不同图像处理方面的依赖项。在确认项目需求后，团队领导的另一个关键职责是选择最合适的 Docker 镜像供项目使用，该镜像将贯穿所有运行时会话。

模块负责人：作为第二级决策者，模块负责人介入的时机是在团队负责人将项目划分为多个分支之后。每个分支模块会被分配给一名模块负责人，负责将团队负责人提供的抽象模块描述转化为具体、可执行的功能思路。为了有效完成这一任务，模块负责人会从团队负责人那里获得模块名称和简要描述，并从思维池中提取整体项目需求，以确保与更广泛目标的一致性。虽然模块负责人不会直接与执行者代理互动，但他们的输出至关重要，因为这些输出包含了更加面向编码的功能思路。为了弥合抽象需求与代码实现之间的鸿沟，模块负责人必须定义每个功能的输入和输出。与团队负责人一样，模块负责人需要尽可能定义最少的功能，以减少冗余并简化整体流程。

功能协调员：一旦某个模块的功能描述完成，它们将传递给功能协调员。在这个阶段，功能思路仍然以自然语言的形式存在，功能协调员的职责是将其转化为代码形式的功能签名。与模块负责人类似，功能协调员需要结合所有先前的重要思路：整体项目需求、从思维池中提取的模块计划以及手头的具体功能思路。通过全面了解所有需要实现的叶子功能，功能协调员负责严格定义每个功能的输入和输出变量类型，从而确保所有功能的一致性，并推动模块的统一性。

![参见说明](img/e69d7594d39ece88e078913791e4a2f7.png)

图 3. 正向任务流程：执行者代理的任务

开发组：在实现者层级，每个函数签名被分配给一个开发组，该组由一个编码员和一个测试员组成，负责从函数思路中实现叶子函数（图[3](https://arxiv.org/html/2410.19245v1#S3.F3 "图 3 ‣ 3.1.1\. 向前任务流 - 分解与分配 ‣ 3.1\. VisionCoder 工作流 ‣ 3\. 多代理自动编程框架与思维分解与组装 ‣ VisionCoder：借助混合大型语言模型赋能图像处理的多代理自动编程")）。编码员接收函数签名以及项目和模块信息，起草代码的初始版本，然后将其交给测试员。测试员基于编码员的第一个版本以及相同的项目和模块信息，编写一套测试代码。之后，角色交换：编码员回顾测试员的代码，以识别并解决其中的问题，这一过程称为结对编程（Coplien, [1998](https://arxiv.org/html/2410.19245v1#bib.bib8)）。一旦测试代码定稿，就会执行以验证编码员的草稿。如果测试失败，编码员会根据错误信息和先前的信息重新生成一个新的代码版本，且这一过程最多会重复三次。在此层级，分配给编码员的函数思路非常细致，通常三轮验证就足以产生最终版本的代码。

![参见标题](img/04e6da1239bad966618e8b777702693f.png)

图 4. 向后任务流：决策者代理的任务

#### 3.1.2\. 向后任务流 - 验证与组装

决策者代理的向后工作流如图[4](https://arxiv.org/html/2410.19245v1#S3.F4 "图 4 ‣ 3.1.1\. 向前任务流 - 分解与分配 ‣ 3.1\. VisionCoder 工作流 ‣ 3\. 多代理自动编程框架与思维分解与组装 ‣ VisionCoder：借助混合大型语言模型赋能图像处理的多代理自动编程")所示。函数协调员：向后任务流采用自下而上的方法。在开发组完成叶子函数的实现后，函数协调员接收代码并将其组装成一个分支模块脚本。由于在向前任务流中缓存了内存，因此此时的函数协调员仅将开发组提供的代码作为输入进行组装。

模块领导：在此阶段，每个模块领导负责验证由功能协调员组装的分支模块代码。类似于开发组中的测试员，模块领导根据初步的模块思路和功能协调员提供的模块代码生成一组测试代码，以验证当前模块。如果测试失败，则返回错误信息给功能协调员，功能协调员将根据错误修正模块代码。此过程确保所有模块都得到适当验证，以便最终汇总成完整的项目输出。

团队领导：VisionCoder 框架的最后一步是由团队领导将模块代码合并为最终的项目输出。由于较低级的代码处理更复杂和基础的功能，因此它经过了更严格的验证。相比之下，高级决策者主要集中于组合更简单的代码，错误风险较低。为了优化效率，我们设计了一种机制，其中模块代码仅验证一次，最终项目代码不再重新验证，从而减少了 token 消耗，同时保持了可靠性。

### 3.2\. 代理化

#### 3.2.1\. 混合型 LLM 应用

基于 VisionCoder 框架的工作流程，分配给决策者代理的任务需要更大的灵活性。例如，在前向工作流程中，团队领导必须首先理解用户提供的项目需求，然后从软件开发的角度将项目分解为可行的模块思路，并为下一层级的代理提供清晰、精确的描述作为输入。模块领导和功能协调员执行类似的任务，这些任务对决策者代理的能力要求非常高。因此，我们在这些角色中实现了专有模型。

执行者，即开发组，专注于代码生成，无需翻译或分发信息，因此其职责更加明确。由于代码生成通常需要处理比自然语言更多的 tokens，我们选择使用本地部署的开源大型语言模型（LLM）来实现更好的成本效益。

![请参阅说明](img/721fbd44e019640393e642509f047b01.png)

图 5. 团队领导定义

#### 3.2.2\. 代理定义

VisionCoder 中代理的定义分为两个方面：结构性和功能性。以团队领导为例，其定义如图[5](https://arxiv.org/html/2410.19245v1#S3.F5 "图 5 ‣ 3.2.1\. 混合型 LLM 应用 ‣ 3.2\. 代理化 ‣ 3\. 多代理自动编程框架与思维分解与组合 ‣ VisionCoder：利用混合型 LLM 实现图像处理的多代理自动编程")所示。

在结构定义中，我们概述了开发图像处理项目的全局背景，随后解释了代理在整个 VisionCoder 框架工作流中的位置。此外，我们通过详细说明代理从上级代理接收到哪些信息以及将哪些信息传递给下级代理，定义了代理与其他代理的交互。通过明确这些交互，每个代理能够清晰理解其职责范围，这也确保了上下游思维流的标准化和高效管理，减少了歧义，简化了工作流。

功能定义概述了代理在接收到上游信息后应采取的具体行动。这个定义由两个重要部分组成。首先，它定义了代理必须遵循的操作步骤。为了最小化 LLM 生成输出中的幻觉风险，我们为每个代理设计了一个主要的推理路径，重点关注图像处理开发场景，并确保它们遵循一致的思维过程，以产生稳定的结果。第二部分则关注输出的格式要求。由于代理的输出通常是模块或功能的自然语言描述，而非结构化数据，我们强制执行严格的格式化规则。这些规则确保关键信息能够通过正则表达式等方法准确提取，从而实现精确交付和后续处理。

### 3.3. 增强策略

#### 3.3.1. 团队领导与编码员的 RAG

在使用 LLM 生成结果时，难以避免的挑战之一是幻觉现象。从 LLM 训练的角度来看，幻觉通常源于两个关键问题（Zhang 等，[2023a](https://arxiv.org/html/2410.19245v1#bib.bib55)）：

+   •

    预训练数据的质量：LLM 的训练需要大量数据，这些数据通常来自开源项目、文章甚至互联网上的论坛讨论。这些数据集常常包含不准确、过时的信息，甚至是捏造的内容。考虑到数据量庞大，手动验证是不可行的，这意味着 LLM 可能会根据有缺陷或误导的先前知识不经意地生成错误的结果。

+   •

    预训练数据集的多功能性：通用大型语言模型（LLM）预计能处理广泛的任务，这就要求训练数据覆盖多个不同的领域。然而，在特定应用中——例如 VisionCoder 框架——我们只需要 LLM 在特定领域的能力。来自无关领域的知识可能会干扰在专门领域中所需输出的质量。

另一个导致幻觉的主要原因是输入提示的不准确性。与 LLM 互动的用户通常依赖简化的描述来获得准确的结果。然而，缺乏关键信息或输入数据不完整，常常导致次优或错误的输出。这种限制在与图像处理相关的代码生成任务中尤为明显，因为输入图像或算法的信息不足，可能导致生成的代码具有不正确的参数或不合适的依赖项，最终产生不准确的结果。

![参见说明](img/8967b7d3c710f7ae218bea99f97b88ef.png)

图 6. 编码员知识检索过程

为了减轻幻觉现象的发生，我们采用了检索增强生成（RAG）策略（Lewis 等人，[2020](https://arxiv.org/html/2410.19245v1#bib.bib23)）。图 [6](https://arxiv.org/html/2410.19245v1#S3.F6 "图 6 ‣ 3.3.1\. RAG 用于团队领导和编码员 ‣ 3.3\. 增强策略 ‣ 3\. 多代理自动编程框架与思维分解与重组 ‣ VisionCoder：借助混合 LLM 强化图像处理的多代理自动编程") 展示了编码员代理的检索增强生成（RAG）过程示例，主要包括两个组件：1. 一个预构建的知识库，包含与目标任务相关或相似的输入和输出示例。该知识库旨在作为 LLM 执行任务时的参考点。实际操作中，知识库被编码成向量化格式，以便于高效检索。2. 第二个组件是检索器，它将新输入转化为查询向量，用于在知识库中查找相关信息。检索到的内容随后被转换回自然语言，并作为上下文支持融入输入提示中。

在 VisionCoder 框架中，团队领导和编码员代理配备了量身定制的 RAG 知识库。为了开发这些知识库，我们从 GitHub 仓库“Image_Processing_100_Questions”中仔细挑选了 50 个与 OpenCV 相关的问题（yoyoyo yo，[2019](https://arxiv.org/html/2410.19245v1#bib.bib49)）。每个代理类型的初步响应是通过 GPT-4o 生成的，随后进行了严格的人工验证和修正，以确保准确性。

这一过程最终构建了一个强大的知识库，专门支持框架内代理的功能和需求。检索到的信息作为思维过程的指南，增强了代理对输入要求的理解。通过采用 RAG，我们增强了代理的理解力，激发了更精确的推理，从而减少了幻觉的发生可能性。

#### 3.3.2. 开发团队的配对编程

![参见标题](img/2aac90aa5306dfb72194ac73bc390ab0.png)

图7. 配对编程

在VisionCoder框架中，开发组位于开发层级的底部，其角色专注于实现代码，而不涉及项目或模块的管理。鉴于他们的任务性质和构成他们的LLMs的能力，开发组可以看作是一个实际开发团队中的初级工程师，这意味着他们产生的代码更容易出错。

虽然编码人员生成的代码可以通过测试和改进迭代优化，但这一过程依赖于测试人员生成的测试代码是可靠的假设。然而，测试代码本身没有内建的验证机制，这可能会导致整个过程中的潜在缺陷。

为了解决这个问题，我们实施了配对编程技术（Coplien, [1998](https://arxiv.org/html/2410.19245v1#bib.bib8)）。如图[7](https://arxiv.org/html/2410.19245v1#S3.F7 "Figure 7 ‣ 3.3.2\. Pair programming for Development Groups ‣ 3.3\. Enhancement strategies ‣ 3\. Multi-Agent Auto-Programming Framework with Thought Decomposition and Assembly ‣ VisionCoder: Empowering Multi-Agent Auto-Programming for Image Processing with Hybrid LLMs")所示，一旦编码人员和测试人员完成了初步的代码草稿，他们的角色会互换：测试代码会传给编码人员进行审查。在开发过程中，我们发现测试人员最常犯的错误是遗漏依赖项，这个问题往往可以通过代码分析而非执行来发现。因此，这一审查过程证明是一种高效、低成本的解决方案，有助于提高测试代码的可靠性。

虽然配对编程的原则建议测试人员也应该审查编码人员的代码，但这一工作在后续的实际测试中更为有效。因此，我们选择在此阶段排除此操作。

### 3.4\. 实现

框架实现大纲的伪代码如算法[1](https://arxiv.org/html/2410.19245v1#alg1 "Algorithm 1 ‣ 3.4.1\. Base model selection ‣ 3.4\. Implementation ‣ 3\. Multi-Agent Auto-Programming Framework with Thought Decomposition and Assembly ‣ VisionCoder: Empowering Multi-Agent Auto-Programming for Image Processing with Hybrid LLMs")所示。给定一个项目，团队负责人将其分解为“模块思想”，每个模块由不同的模块负责人并行处理。这些模块思想进一步分解为“功能思想”，由功能协调员进行细化并分发给开发组。开发组随后实现叶子功能，最后，项目代码逐步验证并合成。

VisionCoder框架的核心开发围绕四个关键组成部分展开：为每个代理选择基础模型、实例化各个代理、代理之间的思维流动，以及建立一个稳健的测试和验证环境。

#### 3.4.1\. 基础模型选择

作为一个多代理框架，实施VisionCoder的第一步是为每个代理选择一个基础模型。VisionCoder的代理分为两类：决策者和执行者。为了保持代理输出风格的一致性，每个类别中的所有代理都使用相同的基础模型。

+   •

    决策者：决策者代理作为执行者的领导者，负责理解任务、将任务转化为可行的思维，并分配子思维。这需要强大的文本理解和推理能力。虽然决策者不负责大规模的代码生成，但他们为后续的实现提供了至关重要的指导，这意味着他们也必须具备基本的编程知识。为了满足这些要求，我们为决策者代理选择了一个具有强大综合能力的专有模型。具体而言，在我们的开发中，我们选择了GPT-4o-20240806（Achiam等，[2023](https://arxiv.org/html/2410.19245v1#bib.bib2)）模型，既因为它具备强大的能力，又因为它比Claude 3.5（Anthropic，[2024](https://arxiv.org/html/2410.19245v1#bib.bib3)）模型更具成本效益。

+   •

    执行者：执行者代理的唯一职责是代码生成。虽然任务是单一的，但底层功能代码的质量直接影响整个项目的性能，这使得执行者模型必须具备强大的编程技能。虽然专有模型在代码生成方面也表现出色，但它们在编程过程中消耗的令牌数量较高，这促使我们选择了一个专门为代码生成任务设计的开源模型。在选择执行者的基础模型时，我们考虑了三个关键因素：1\. 角色定义能力：一些开源模型无法处理系统提示，这可能会妨碍代理访问全局信息，从而影响性能。2\. 长文本处理能力：由于编码器代理配备了知识库，它经常需要处理较长的上下文输入。参数较少的模型难以有效管理长文本。3\. 模型大小：尽管性能通常随着参数数量的增加而提升，但硬件限制使得我们无法部署异常庞大的模型。在考虑了这些因素后，我们选择了deepseek-coder-7b-instruct-v1.5作为执行者的基础模型。

算法 1 VisionCoder框架实现

1:项目需求 $\rho$ 2:项目代码 $\Phi_{project}$ 3:$TL\leftarrow$ 团队领导初始化 4:模块列表 $[\mu]\leftarrow TL.SplitModuleThoughts(\rho)$ 5:对每个 $(\mu_{i})\in[\mu]$ 进行 6:     $ML\leftarrow$ 模块领导初始化 7:函数思维列表 $[\nu]\leftarrow ML.SplitFunctionThoughts(\mu_{i})$ 8:     $FC\leftarrow$ 函数协调员初始化 9:     精炼后的函数思维列表 $[\nu_{R}]\leftarrow FC.RefineFunctionThoughts([\nu])$ 10:     对每个 $(\nu_{Rj})\in[\nu_{R}]$ 进行 11:         $DG\leftarrow$ 开发小组初始化 12:         函数代码 $\psi_{j}\leftarrow DG.Coding(\nu_{Rj})$ 13:         $\psi_{j}\leftarrow DG.Validation(\psi_{j})$ 14:     结束 15:     模块代码 $\phi_{i}\leftarrow FC.AssembleModule([\psi])$ 16:     $\phi_{i}\leftarrow ML.TestModule(\phi_{i})$ 17:结束 18:返回 $\Phi_{project}\leftarrow TL.AssembleProject([\phi])$

#### 3.4.2\. 代理实例化

VisionCoder 中的代理分为四个层次，每个层次都针对框架中的特定任务进行优化。尽管它们有不同的角色，但所有代理都共享核心功能，如初始化、内存管理和思维检索。为了简化开发过程，我们设计了一个可定制的基础代理，包含这些基本方法，而每个具体代理则在此基础上扩展其角色特定功能。

基础代理设计的一个关键优势是它提供的模块化。尽管当前版本使用固定的基础模型作为决策者和执行者，但跨大规模模型的统一交互逻辑允许快速迭代。通过简单地继承基础代理，研究人员可以轻松地用性能更强的替代品替换基础模型，从而使未来的研究能够利用 VisionCoder 的结构，同时集成改进的模型。

#### 3.4.3\. 代理之间的思维流

VisionCoder 中代理之间的通信通过思维流进行，这与代理的层次结构相一致。如图[8](https://arxiv.org/html/2410.19245v1#S3.F8 "Figure 8 ‣ 3.4.3\. Thought flow among agents ‣ 3.4\. Implementation ‣ 3\. Multi-Agent Auto-Programming Framework with Thought Decomposition and Assembly ‣ VisionCoder: Empowering Multi-Agent Auto-Programming for Image Processing with Hybrid LLMs")所示，例如，团队领导将项目需求（根思维）分解为分支思维。这些分支思维由两个组件组成：超思维和灵活思维。超思维表示固定规则，如模块名称、编码语言、运行时环境和工作目录。这些模式由框架和团队领导确定，并在整个过程中保持不变，不可由下属代理修改。相比之下，灵活思维代表解决问题的概念，指导下属代理，使它们能够在执行任务时将这些想法扩展和细化为更详细的思维。

![参见标题](img/797d28758ab38f76ec0a7524ff92e451.png)

图 8. VisionCoder 中的思维流程与缓存

\Description

VisionCoder 中的思维流程与缓存

每个代理生成的思维被存储在思维池中。这个机制有两个主要目的。首先，代理保留其行为的记忆，而这些记忆有时会在后续任务中引入错误。在这种情况下，有必要从思维池中检索以前的思维，这就像是一个受“思维树”启发的回溯机制。例如，在配对编程后，程序员可能会错误地认为测试代码是他的输出，并尝试优化它，而不是专注于功能代码。通过访问思维池，可以检索程序员之前的输出并重新分配。思维池的第二个目的来源于大语言模型（LLM）的结构性限制，这些限制使得代理之间无法直接通信。虚拟用户代理通过从思维池中选择并传输正确的内容来促进这一过程，从而实现高效的代理间通信。

#### 3.4.4\. 测试环境

与单个 LLM 或通用自动化编程框架不同，VisionCoder 框架中的一个关键机制是验证。由于 VisionCoder 专注于解决与图像处理相关的问题，因此必须建立一个合适的运行时环境，包括适当的 Python 版本、依赖项和文件系统映射。这一配置是确保验证过程正确运行所必需的。

尽管可以直接在 VisionCoder 的运行时环境中进行验证，但这样做存在一定风险。版本变化或依赖项的删除可能会影响环境，而文件系统操作中的错误可能会导致不可逆的后果。因此，最有效的做法是为 VisionCoder 的验证过程创建一个专门的隔离环境。

创建隔离环境的方法有多种，包括虚拟机和基于 Python 的虚拟环境。然而，在 VisionCoder 中，我们选择使用 Docker 镜像来构建验证的运行时环境。Docker 相较于虚拟机或虚拟环境的主要优势是其易于配置。可以编写一个简单的 Dockerfile 来创建所需的镜像，然后在验证过程中使用 Docker 的 Python 运行时库，从而实现无缝部署。

### 3.5\. 输出分析

![参见标题](img/f28f5808bac41346f305bf7d7deb0307.png)

图 9. VisionCoder 的示例输出与结果

如图[9](https://arxiv.org/html/2410.19245v1#S3.F9 "Figure 9 ‣ 3.5\. Output analysis ‣ 3\. Multi-Agent Auto-Programming Framework with Thought Decomposition and Assembly ‣ VisionCoder: Empowering Multi-Agent Auto-Programming for Image Processing with Hybrid LLMs")所示，在示例任务的背景下，VisionCoder将项目分解为三个主要分支模块：ImageInput、LicensePlateDetection和LicensePlateRecognition。每个模块随后被进一步划分为两个叶子函数，并通过测试代码进行单独验证。然后，这些叶子函数被组合成完整的模块，并进行模块级的测试。最终，经过全面测试的模块被组装成最终的项目代码，生成所需的输出。

整个过程完全自动化，用户只需提供项目规范。VisionCoder的智能体自主管理任务的逐步分解和实现。由于混合模型的设计，项目在不到10分钟的时间内完成，成本不到0.01美元。即便与经验丰富的人类开发者相比，VisionCoder也表现出极强的竞争力。它不仅完成任务的速度显著更快，而且还自动化了调试和验证，减少了人为错误，并确保了一致的结果。

超越这一场景，VisionCoder架构模拟了真实开发团队的结构。根据团队领导者对项目需求的理解，VisionCoder的规模可以灵活调整，生成的分支思路也可以根据项目的复杂度进行调整。通过利用思维树结构，VisionCoder将复杂软件系统的自动化编程任务分解为小的、易于管理的思路。这使得框架能够充分利用LLM在功能级代码生成方面的能力，将来自这些小问题解决思路的方案整合在一起。与需要大量时间用于项目熟悉、参数调试、团队级代码整合和沟通的人类开发者团队相比，VisionCoder的效率要高得多。

## 4\. 评估

本节中，我们重点评估VisionCoder。首先，我们概述了指导实验和分析重点的研究问题，接着详细介绍了专门为图像处理领域的自动化编程任务设计的数据集构建过程。然后，我们详细说明了实验设置，包括基准方法的选择、评估标准、测试工作流程以及用于评估的硬件资源。最后，我们分析了比较实验的结果，并回答了研究问题。

### 4.1\. 研究问题

为了验证VisionCoder的有效性，本研究解决了以下研究问题：

RQ1：通过思想分解和组装，树形结构的多智能体框架在图像处理编程任务中表现如何？为了评估我们框架的性能，我们编制了一个经典图像处理算法的基准数据集。该数据集中的每个任务代表一个完整的图像算法过程，使我们能够评估框架在项目级别上分解和组装模块及功能的有效性。

RQ2：有哪些辅助技术可以提高每个智能体在图像处理任务中的表现？为了增强框架中单个智能体的性能，我们采用了几种互补策略。这些策略包括为特定智能体整合增强检索生成（RAG），以提升其在图像算法中的能力，并在开发组的最低级别应用灵感来自软件工程的双人编程机制。我们设计了消融实验来验证这些策略的有效性。

RQ3：我们的框架与其他自动编程工具的性能如何比较？我们使用相同的数据集，对多个先进的自动编程方法进行了比较评估，以展示我们层次框架的优势。

RQ4：如何将专有和开源的LLM（大语言模型）作为混合机制来提高编程的成本效益？尽管专有模型在代码生成任务中表现出色，但它伴随着高昂的成本，特别是在生成大量代码时。我们进行了成本效益比较，评估了专有模型与本地部署的开源模型的整合如何在我们的框架中平衡性能和费用。

### 4.2\. 基准数据集

为了验证VisionCoder在图像处理领域自动化编程的能力，并将其与其他先进的自动化编程框架进行比较，我们编制了一个包含90个与图像处理相关的项目的测试集，命名为BCVPP（基础计算机视觉Python编程）。

#### 4.2.1\. BCVPP构建

BCVPP专注于基础的Python图像处理项目，特别是经典图像算法的范围内。数据集包含总共90个项目，其中包括30个简单项目、50个中等项目和10个难度较高的项目。

以下是BCVPP中不同类别的一些示例项目：

+   •

    简单：给定一张输入图像（"./test_image.png"），对其进行2次金字塔缩放，一次放大，一次缩小。将结果图像保存为"upscaled.png"和"downscaled.png"。

+   •

    中等：给定一张图片（"./test_image.png"），应用以下值的锐化滤波器：[[0, -1, 0], [-1, 5, -1], [0, -1, 0]]。然后应用一个5x5的高斯滤波器，sigmax=5（使用GaussianBlur）。将结果图像保存为"sharpened_gaussian_image.png"。

+   •

    难度：给定输入图像（"./texture_icon.jpg"），使用 Efros_Leung 算法进行纹理合成，将图像扩展为 128x128 像素。将结果保存为 "texture_synthesized.png"。在开始算法之前，设置 np.random.seed 为 0。

在经典图像处理领域，OpenCV 是一款必不可少的工具。作为传统图像算法的经典库，许多在线课程和项目库提供了基于 OpenCV 的示例。在评估了多个来源后，我们选择了一个面向初学者的 GitHub 仓库“OpenCV_Projects” (rchavezj, [2019](https://arxiv.org/html/2410.19245v1#bib.bib34)) 作为主要的测试数据来源。这个仓库拥有超过 340 个星标和大约 55 个与 OpenCV 相关的问题，之所以选择它，是因为其内容被开发者广泛接受，并且提供了适当的难度。由于常用的 OpenCV 操作种类有限，为了避免与从其他来源已为 VisionCoder 知识库选择的 50 个项目重复，我们从这个仓库选择了 30 个简单项目。这些项目被归类为简单，因为它们仅需在 VisionCoder 框架内使用一个模块即可完成，无需组合多个模块。

虽然简单项目对于初步验证非常有用，但要在更复杂的项目级任务上测试 VisionCoder 的性能，需要涉及多个模块集成的项目。不幸的是，这类项目通常存在于现实世界的工业场景中，而在开源仓库或教程中很少能够找到。为填补这一空白，我们利用了 OpenAI-o1 (OpenAI, [2024](https://arxiv.org/html/2410.19245v1#bib.bib30)) 模型，通过结合 30 个简单项目中的元素，生成了 100 个中等复杂度的项目。然而，在校准过程中，我们发现许多生成的项目缺乏模块之间一致和逻辑的关系，导致项目描述混乱。在审查并完善这些项目后，我们最终确定了 50 个中等项目，这些项目通常需要 2 至 3 个模块才能完成。

为了进一步挑战 VisionCoder 和其他自动化编程方法的能力，我们决定加入更具挑战性的项目。我们确定了10个困难项目，这些项目从两个关键方面推动了极限。首先，这些项目本身更为复杂，要求更多的模块，并且即便是对于真实开发者来说，也需要更高的专业水平。其次，与简化项目中清晰的逐步指导不同，这些项目仅提供模糊的描述——通常只提供输入图像和目标要求——迫使方法自行推断必要的步骤。

需要强调的是，目前BCVPP只关注经典图像算法，尚未涵盖机器学习或深度学习方法。（即使涉及深度学习模型，它们也仅限于对预训练模型的直接调用。）我们选择排除这些领域，因为机器学习和深度学习模型的开发涉及更多复杂性—如数据预处理、训练和验证集创建、模型开发、模型训练和后处理—我们认为这些超出了当前LLM的能力范围。因此，我们决定将BCVPP的构建重点放在经典图像处理领域。

#### 4.2.2\. BCVPP中的项目组件

对于BCVPP中的每个程序，我们包括以下组件：

+   •

    输入图像：由于这是一个图像处理数据集，每个项目都与一个指定的输入图像相关联。所有生成的程序必须对这个提供的图像执行操作。

+   •

    项目描述：对于简单和中等难度的项目，我们提供详细的项目描述，其中包括输入图像的路径、要执行的图像处理操作（带有固定参数）以及输出文件的路径。对于困难级别的项目，我们仅提供输入图像的路径和目标结果的概略描述，具体步骤则较为开放。

+   •

    示例解决方案模块：每个项目都配有一个示例解决方案。虽然每个项目可以通过多种方式实现，但最终输出应保持一致。示例代码提供了一个参考解决方案，生成所需的输出，用以评估生成程序的准确性。

+   •

    测试模块：测试代码负责将生成的程序在输入图像上的结果与预期输出进行比较。这个比较决定了生成的程序是否正确。

### 4.3\. 基准线

为了评估VisionCoder框架在图像处理领域的表现，我们选择了几种在自动化编程和代码生成领域的最先进（SOTA）方法作为基准进行比较。以下标准用于指导这些基准的选择：

+   •

    新颖性和性能：鉴于大型语言模型（LLMs）的快速发展，模型的迭代可能在几周内发生。许多在发布时表现出色的模型，很快就会被更新版本所超越。因此，我们选择了在一年内发布且在代码生成任务中依然表现竞争力强的模型。

+   •

    专业化：虽然有许多通用型LLM，但它们的主要应用往往是问题解决或问答任务。为了确保公平比较，我们选择了专门为代码生成设计的方法，或者至少选择那些将代码生成作为关键子任务的方法。

+   •

    输出内容的可控性：尽管许多基于LLM的工具具备代码生成能力，但它们通常针对固定的应用场景，难以控制其输出的结构。因此，我们选择了那些可以控制输出格式的基准方法，从而简化了测试过程。

+   •

    模型大小：此标准主要适用于开源LLM。由于硬件限制，我们无法部署超过100B参数的模型进行本地测试。

表[1](https://arxiv.org/html/2410.19245v1#S4.T1 "表 1 ‣ 4.3\. 基准 ‣ 4\. 评估 ‣ VisionCoder: 通过混合LLM赋能多智能体自动编程来进行图像处理")提供了选择为基准的各方法的详细信息，包括它们对应项目或模型的链接。

表1. 用于对比实验的基准方法

| 名称 | 访问链接 |
| --- | --- |
| 开源模型 |  |
| LLaMA3.2-3Bt | https://huggingface.co/meta-llama/Llama-3.2-3B-Instruct |
| MagiCoder-6.7B | https://huggingface.co/ise-uiuc/Magicoder-S-DS-6.7B |
| CodeGemma-7B | https://huggingface.co/google/codegemma-7b |
| DeepSeek-Coder-V2-Lite-16B | https://huggingface.co/deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct |
| CodeLLaMA-Python-70B | https://huggingface.co/codellama/CodeLlama-70b-Python-hf |
| CodeLLaMA-Instruct-70B | https://huggingface.co/codellama/CodeLlama-70b-Instruct-hf |
| LLaMA3.1-70B | https://huggingface.co/meta-llama/Llama-3.1-70B-Instruct |
| Qwen2.5-72B | https://huggingface.co/Qwen/Qwen2.5-72B-Instruct |
| LLaMA3.2-Vision-90B | https://huggingface.co/meta-llama/Llama-3.2-90B-Vision-Instruct |
| 专有模型 |  |
| GPT-4o-2024-08-06 | https://platform.openai.com/docs/models/gpt-4o |
| Claude3.5-sonnet | https://docs.anthropic.com/en/docs/intro-to-claude#claude-3-5-family |
| 使用OpenAI API的多智能体框架 |  |
| ChatDev | https://github.com/OpenBMB/ChatDev |

### 4.4\. 评估指标

我们的目标是使用BCVPP数据集来评估各种方法在图像处理领域自动化编程的能力。对于每个项目$p$，给定输入图像$Img_{p}$和指定的处理需求$\rho$，每种方法的任务是生成一组算法代码$\Phi_{project}$以满足给定的需求。然后，这些代码将在输入图像上执行以生成输出。评估的主要目标是验证生成的输出是否与期望结果$Output_{exp}$相匹配。这个过程可以通过以下方程表示。

| (1) |  | $\displaystyle Output\leftarrow\Phi_{project}(Img_{p})$ |  |
| --- | --- | --- | --- |
| (2) |  | $\displaystyle Output_{exp}\leftarrow\Phi_{sample}(Img_{p})$ |  |
| (3) |  | $\displaystyle Acc_{p}=\begin{cases}1,&Output=Output_{exp}\\ 0,&Output\neq Output_{exp}\end{cases}$ |  |

方法在整个BCVPP数据集上的总体准确性可以表示为：

| (4) |  | $Acc=\mathop{\mathbb{E}}\limits_{projects}\left[Acc_{p}\right],\text{ 其中 }% Acc_{p}\in\{0,1\}$ |  |
| --- | --- | --- | --- |

### 4.5\. 评估过程

我们的评估过程分为两步：脚本化评估和人工评估。

脚本化评估：如[4.2.2节](https://arxiv.org/html/2410.19245v1#S4.SS2.SSS2 "4.2.2\. 项目组件在BCVPP中 ‣ 4.2\. 基准数据集 ‣ 4\. 评估 ‣ VisionCoder：通过混合LLM增强图像处理的多代理自动编程")所述，BCVPP数据集包括一个自动化评估的测试模块。在方法生成项目代码并输出$Output$后，测试模块运行样本解决方案模块以生成$Output_{exp}$并比较两者结果。如果输出匹配，生成的项目代码被认为是正确的；否则，它被视为失败。

人工评估：在审查脚本化评估的结果时，我们发现它往往导致假阴性。这是因为不同的方法可能会使用不同的算法来解决相同的图像处理问题，导致$Output$与$Output_{exp}$并不完全相同，即使它们在技术上都是正确的。例如，当被要求计算图像的直方图时，一些方法可能使用numpy.histogram()函数，而我们的样本解决方案使用cv2.calcHist()。尽管这些算法产生的结果在维度上有所不同，但它们都是有效的直方图表示。为了解决这个问题，我们在脚本化评估之外引入了人工评估步骤，其中结果会被人工检查以确保一致性和正确性。

### 4.6\. 硬件配置

专有模型框架依赖直接的API调用，减少了本地计算。然而，基于LLM的方法需要大量的GPU资源。评估是在一台配置为AMD Ryzen Threadripper PRO 3995WX CPU（64核心）、252GB内存和两块NVIDIA RTX A6000 GPU（每块48GB VRAM）的Ubuntu 22.04服务器上进行的。

### 4.7\. 实验结果

表2. 自动编程方法对比

|  | 简单 | 中等 | 困难 | 总体 |
| --- | --- | --- | --- | --- |
| 开源模型 |  |  |  |  |
| LLaMA3.2-3B | 30.00% | 22.00% | 10.00% | 23.33% |
| MagiCoder-6.7B | 70.00% | 36.00% | 10.00% | 44.44% |
| CodeGemma-7B | 30.00% | 18.00% | 20.00% | 21.11% |
| DeepSeek-Coder-V2-Lite-Instruct-16B | 60.00% | 38.00% | 20.00% | 43.33% |
| CodeLLaMA-Python-70B | 23.33% | 20.00% | 0.00% | 18.88% |
| CodeLLaMA-Instruct-70B | 30.00% | 28.00% | 20.00% | 26.67% |
| LLaMA3.1-70B | 66.67% | 48.00% | 20.00% | 51.11% |
| Qwen2.5-72B | 86.67% | 54.00% | 20.00% | 61.11% |
| LLaMA3.2-Vision-90B | 63.33% | 66.00% | 10.00% | 58.89% |
| 专有模型 |  |  |  |  |
| GPT-4o-2024-08-06 | 73.33% | 62.00% | 50.00% | 64.44% |
| Claude3.5-sonnet | 80.00% | 54.00% | 60.00% | 63.22% |
| 使用OpenAI API的多代理框架 |  |  |  |  |
| ChatDev与GPT-3.5（默认） | 53.33% | 24.00% | 0.00% | 31.11% |
| 使用GPT-4o的Chatdev | 70.00% | 54.00% | 30.00% | 56.67% |
| VisionCoder框架 |  |  |  |  |
| 仅使用结构化的VisionCoder | 73.33% | 38.00% | 10.00% | 45.56% |
| 仅使用知识库的VisionCoder | 80.00% | 56.00% | 30.00% | 62.22% |
| 仅使用配对编程的VisionCoder | 76.67% | 40.00% | 10.00% | 48.89% |
| 启用所有策略的VisionCoder | 86.67% | 68.00% | 50.00% | 72.22% |

#### 4.7.1\. VisionCoder的表现

在全策略配置下，VisionCoder的表现总结见表格[2](https://arxiv.org/html/2410.19245v1#S4.T2 "Table 2 ‣ 4.7\. Experiment results ‣ 4\. Evaluation ‣ VisionCoder: Empowering Multi-Agent Auto-Programming for Image Processing with Hybrid LLMs")的最后一行。在BCVPP数据集上，VisionCoder达到了72.22%的整体准确率。鉴于BCVPP专门设计用于评估专业图像处理场景中的自动编程，该结果非常令人满意，展示了框架的有效性。

在简单问题类别中，VisionCoder实现了86.67%的出色准确率。这些任务通常只需要一个模块，其中包含两个到三个功能。尽管VisionCoder的任务分配机制在这种情况下影响有限，但决策者代理通过将任务描述分解并精细化为更详细的功能思路，确保了开发小组的准确实现。这种分解导致了更清晰的实现规范，最终导致了简单任务的高准确率。

对于中等难度的问题，VisionCoder实现了68.00%的准确率。这些任务通常需要两个到三个模块，在这种情况下，VisionCoder在思维分解和任务分配方面的优势得到了充分体现。该框架有效地将项目拆分为可管理的模块，每个模块在被决策者代理组装成最终项目代码之前都会经过验证。这一过程突出了VisionCoder任务分解和组装机制的强大功能，这对于处理更复杂的项目至关重要。

在困难的项目中，VisionCoder的表现略微下降至50.00%。这一准确率的下降主要归因于项目描述中缺乏明确的任务指示。尽管如此，VisionCoder仍然能够将任务拆分为模块，并进一步将其细分为功能，遵循其结构化的框架设计。然而，当项目描述过于简略时，VisionCoder有时仅创建了一个单一模块，这使得执行者代理的工作流变得复杂，导致表现下降。

在评估中，我们发现 VisionCoder 在处理涉及多个输入文件的任务时存在一定的局限性。例如，在人脸识别任务中，一个输入是人脸图像，另一个输入是包含面部特征的 XML 文件，但 VisionCoder 有时会错误地将 XML 文件误判为图像。这个问题的产生是因为该框架主要集中在分析项目描述文本上，而对输入文件的性质和格式关注较少，导致在这些场景下偶尔出现失败。

对于 RQ1 的回答：VisionCoder 在 BCVPP 数据集上展示了在不同任务难度下的强大性能，整体准确率达到了 72.22%。对于简单问题，VisionCoder 通过优化任务描述并确保函数实现的准确性，表现出色，准确率为 86.67%。在中等难度的任务中，VisionCoder 的层次结构显示了其优势，通过有效地将项目拆分成模块并在组装前进行验证，达到了 68.00% 的准确率。在困难任务中，由于任务指令不明确，准确率轻微下降至 50.00%，但 VisionCoder 仍然有效地完成了任务分解和实现。在处理多个输入文件时出现了局限性，但总体而言，VisionCoder 的任务分配和组装机制被证明是有效的。

#### 4.7.2\. 消融研究

表格[2](https://arxiv.org/html/2410.19245v1#S4.T2 "Table 2 ‣ 4.7\. Experiment results ‣ 4\. Evaluation ‣ VisionCoder: Empowering Multi-Agent Auto-Programming for Image Processing with Hybrid LLMs")的最后四行展示了 VisionCoder 在不同策略配置下的性能。实验结果表明，当仅使用主框架时，VisionCoder 的表现相对较为逊色。在这种配置下，发现了两个主要问题：首先，团队领导常常过度拆分任务，创建了过多的功能重叠的模块，导致了错误。其次，编码员经常生成超出指定功能的内容，导致令牌溢出，触发了由于模型最大令牌限制而使脚本被截断，从而导致生成的代码无法使用。

虽然配对编程（Pair Programming）提供了一些改进，但其影响有限，因为它主要增强了验证阶段，并未解决与模块划分或整体功能脚本相关的错误。相比之下，知识库的整合显著缓解了这些挑战。它使得团队领导能够更精确地划分模块，并限制编码员的输出，防止生成不必要的内容。此外，配对编程和知识库的结合使用大大提高了整体准确性。

回答RQ2：使用基于检索增强生成（RAG）的知识库集成增强了任务分解，并限制了不必要的代码生成。此外，配对编程通过使代理之间进行交叉检查和修正，改善了验证过程。这些策略共同显著提高了整体准确性。

#### 4.7.3\. 与其他方法的比较

本节将重点介绍其他模型的性能，并分析VisionCoder相较于它们的优势。

从参数少于50B的较小开源模型开始，表格[2](https://arxiv.org/html/2410.19245v1#S4.T2 "Table 2 ‣ 4.7\. Experiment results ‣ 4\. Evaluation ‣ VisionCoder: Empowering Multi-Agent Auto-Programming for Image Processing with Hybrid LLMs")显示，它们的整体准确性低于50%，主要是因为模型参数数量有限。在这些模型中，MagiCoder（Wei等，[2023](https://arxiv.org/html/2410.19245v1#bib.bib45)）和DeepSeek-Coder-V2（DeepSeek，[2023](https://arxiv.org/html/2410.19245v1#bib.bib9)）表现突出，成为最顶尖的模型。值得注意的是，MagiCoder在简单任务上的表现异常出色，甚至超越了一些较大的模型。我们认为这是因为简单任务数据集来自一个开源GitHub项目。MagiCoder的优势在于其高质量的训练数据集，这些数据集可能与BCVPP中的简单任务有重叠或相似之处。这也解释了为什么MagiCoder在简单任务上表现异常出色，但在中等和困难任务上表现较差。DeepSeek-Coder在所有任务类别上表现稳定良好，可能是因为其独特的专家混合模型结构。尽管由于硬件限制我们未能测试DeepSeek-Coder的更大236B模型，但我们预期如果进行测试，它将取得更好的结果。相比之下，LLaMA3.2-3B（Dubey等，[2024](https://arxiv.org/html/2410.19245v1#bib.bib12)）和CodeGemma-7B（Team，[2024a](https://arxiv.org/html/2410.19245v1#bib.bib39)）的表现较差，主要由于它们的参数数量有限，以及训练数据的质量不足。

这些结果表明，单一的小规模模型仍然缺乏有效处理复杂任务的能力。在VisionCoder框架中，我们使用7B模型作为执行者代理，仍然能够取得优异的表现。这一成功表明，VisionCoder将复杂任务分解为更简单功能的策略是一种有效且合理的方法。

对于体积较大的开源模型，其整体表现明显优于小型模型。除了CodeLLaMA-Python和CodeLLaMA-Instruct（Roziere等，[2023](https://arxiv.org/html/2410.19245v1#bib.bib35)），它们较旧且训练数据质量较低外，所有其他大规模模型的准确率均超过50%。值得注意的是，Qwen2.5（Team，[2024b](https://arxiv.org/html/2410.19245v1#bib.bib40)）和LLaMA-3.2-Vision（Dubey等，[2024](https://arxiv.org/html/2410.19245v1#bib.bib12)）的准确率均接近60%。尽管这些模型在简单和中等难度任务中表现令人满意，但在更具挑战性的问题上表现受限。这表明，即使是相对较大规模的模型也难以将复杂任务分解为可行的解决方案。这一观察进一步支持了VisionCoder任务分解前向工作流的有效性，该工作流旨在更高效地应对此类难题。

专有模型的表现（Achiam等，[2023](https://arxiv.org/html/2410.19245v1#bib.bib2); Anthropic，[2024](https://arxiv.org/html/2410.19245v1#bib.bib3)）在所有基准模型中最为强劲。这些专有模型的优质数据和庞大的训练数据无疑为它们在各类项目难度下始终如一的强劲表现提供了支持。相比之下，依赖OpenAI API的多代理框架ChatDev（Qian等，[2024](https://arxiv.org/html/2410.19245v1#bib.bib32)）表现较弱。这主要是因为ChatDev将所有开发任务分配给单一代理，同时施加了许多格式限制。因此，对于单一代理来说，处理复杂任务变得过于困难，导致其表现不如纯GPT-4o API调用。

对RQ3的回答：VisionCoder在各类任务难度下超越了开源模型和专有模型。虽然小型模型在复杂任务中表现不佳，大型开源模型表现适中但在困难问题上显示出局限性，VisionCoder的分层任务分解证明了其高度有效性。即使与专有模型相比，VisionCoder也取得了优越的成绩，尤其是在分解其他框架（如ChatDev）难以处理的复杂任务方面。

表3. BCVPP评估的总成本

| 工具使用 | 成本 |
| --- | --- |
| GPT-4o | 0.26$ |
| Claude3.5-Sonnet | 0.50$ |
| ChatDev with GPT-3.5 | 5.81$ |
| ChatDev with GPT-4o | 7.83$ |
| VisionCoder | 0.32$ |

对于专有模型来说，主要的限制因素是成本。在我们对BCVPP数据集进行实验时，所有使用专有模型的方法所产生的费用详见表[3](https://arxiv.org/html/2410.19245v1#S4.T3 "Table 3 ‣ 4.7.3\. Comparisons with other methods ‣ 4.7\. Experiment results ‣ 4\. Evaluation ‣ VisionCoder: Empowering Multi-Agent Auto-Programming for Image Processing with Hybrid LLMs")。由于数据集相对较小且我们未使用最昂贵的模型，因此GPT-4o和Claude3.5-Sonnet（直接API调用）产生的费用都低于$1。然而，ChatDev由于其多代理架构和冗余的代理间通信，产生的费用明显更高。即使使用GPT-3.5，ChatDev的费用也超过了$5，而使用GPT-4o的费用比直接API调用高出了30倍以上。相比之下，VisionCoder的混合模型配置，在处理最需要token的任务时使用本地模型，实现了类似直接GPT-4o API调用的成本，同时利用多代理框架，这比ChatDev便宜得多。这证明了VisionCoder在成本效率方面的高水平。

与GPT-4o相比，VisionCoder在准确度上提升了7.78%，从64.44%提高到72.22%，但成本增加了23.07%（$0.32对比$0.26）。虽然成本增加看起来相对较高，但准确度的提升代表了在需要更高精度的任务中的显著性能提升。在许多情况下，这种准确度的提升足以为额外的成本辩护，特别是当任务需要更高的可靠性和更少的错误时。这表明，VisionCoder提供了更准确的解决方案，并且对于那些精度至关重要的场景，额外的成本是一个合理的投资，能带来性能上的收益。

对RQ4的回答：VisionCoder的混合模型配置通过利用专有模型处理决策任务，同时使用本地开源模型处理最需要token的部分（如代码生成），有效地平衡了成本和性能。这种方法显著降低了与纯API方法相比的开销，实现了与直接GPT-4o API调用相当的成本效率，甚至在多代理框架内也是如此。

## 5\. 讨论

### 5.1\. 发现总结

通过实验和研究问题的解答，我们识别出了VisionCoder框架的几个关键发现：

+   •

    树状框架与思维流程：通过模拟一个真实的开发团队，VisionCoder将完整的图像处理任务分解为分支模块、叶子函数和具体的实现层次，利用多层级代理。然后通过反向任务流将解决方案组装起来。这种思维树推理结构使每个代理能够专注于项目中的一小部分，克服了单一代理在处理复杂任务时的局限性。

+   •

    混合 LLM 部署：在 VisionCoder 中，决策者代理使用高性能的专有模型，而实施者代理则依赖于专门针对特定应用场景的开源模型。这种混合方法利用专有模型的全面理解来分解项目，同时利用开源模型的专业性来进行实施。这一策略最大限度地减少了令牌消耗，并提高了成本效益。

+   •

    高级附加策略：VisionCoder 集成了额外的策略，如结对编程和基于检索增强生成（RAG）的策略，以增强某些代理的性能。这些外部插件进一步提升了框架的整体生成能力。

+   •

    模块化代理构建：虽然我们在 VisionCoder 的实现中使用了特定的专有和开源模型，但这些模型在模块化结构中是可以互换的。随着新模型的出现，VisionCoder 中的每个代理都可以更新为性能更好的基础模型，提供极大的灵活性和可扩展性。

VisionCoder 作为一个混合 LLM 驱动的多代理自动编程框架，提供了灵活的代理部署和高效的思维分解优势。尽管它主要在图像处理领域进行了验证，但在拥有合适的知识库、足够的计算资源和恰当的 LLM 配置的情况下，VisionCoder 有潜力大幅提升开发速度和成本效益，相比于人类开发团队，能够更高效地构建大规模复杂的软件系统。

### 5.2\. 有效性威胁

尽管具有优势，作为一个使用混合机制构建的多代理框架，VisionCoder 由于其结构和设计，仍存在一些局限性：

执行速度：由于实施者代理利用本地部署的开源模型，因此它们的推理速度受限于硬件性能，速度较专有模型慢。此外，实施者的工作过程涉及多轮验证和修正，这使得框架的整体运行时间比单模型方法更长。然而，我们已确保 VisionCoder 的运行时间与专门使用专有模型的多代理框架 Chatdev 相当，表明尽管存在速度限制，VisionCoder 的结构优势仍然有效。

知识库规模：虽然我们已为团队负责人和编码者实现了检索增强生成（RAG），允许团队负责人基于知识库的知识来分解任务，编码者则可以直接应用函数实现，但其他智能体并未使用此功能。它们的任务更紧密地与特定项目相关，知识库提示可能会干扰其推理过程。此外，当前的知识库仅涵盖了50个项目，无法充分发挥VisionCoder的全部潜力。扩展知识库，涵盖更多项目，将可能带来更进一步的性能提升。

## 6\. 结论

本文提出了VisionCoder，这是一种层次化的混合模型框架，专门用于解决经典图像处理中的代码生成任务。该框架采用双向任务流：在正向任务流中，项目被分解并分配到多个智能体层级；而在反向任务流中，生成的代码被组合并验证，以完成项目。此外，我们还通过技术手段如对偶编程（Pair Programming）和检索增强生成（Retrieval-Augmented Generation，RAG）提升了VisionCoder的能力。为了评估，我们整理了BCVPP数据集，专注于项目级图像处理代码生成，并将VisionCoder与最先进的方法进行了对比。实验结果表明，VisionCoder在图像处理自动编程任务中优于其他方法。

在未来的研究中，我们计划从多个方面进一步改进VisionCoder。首先，我们打算细化和改进知识库，优化其范围并将更多的知识扩展到更多的智能体。其次，我们将致力于进一步自动化VisionCoder的工作流程和交互模式，减少对手动框架约束的依赖。第三，我们相信VisionCoder的模块化结构和灵活的任务分解方式可以适应更广泛的应用领域，超越普通的图像处理，拓展到医学或高光谱图像处理等更深层次的领域。最后，我们将继续增强BCVPP数据集，加入来自实际工业场景的任务，并增加与机器学习相关的项目。

随着持续的优化，我们相信层次化的多智能体机制将能够处理更加复杂的自动化编程任务，尤其是在图像处理和其他专业领域，帮助开发者和研究人员在现实场景中更高效地应对开发挑战。

## 数据可用性声明

我们发布了源代码和数据集，以鼓励在这个方向上进一步探索。支持本文讨论结果的工件可以在以下网址获得：[https://github.com/VisionCoder1/VisionCoder](https://github.com/VisionCoder1/VisionCoder) 和 [https://github.com/VisionCoder1/BCVPP](https://github.com/VisionCoder1/BCVPP)。这两个工件将提交进行工件评估，并将继续完善和扩展，以支持图像处理自动编程的持续研究。

## 参考文献

+   (1)

+   Achiam等人（2023）Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat等人. 2023年. 《GPT-4技术报告》. *arXiv预印本 arXiv:2303.08774* (2023).

+   Anthropic（2024）Anthropic. 2024年. 《Claude 3模型家族：Opus，Sonnet，Haiku》. [https://www-cdn.anthropic.com/f2986af8d052f26236f6251da62d16172cfabd6e/claude-3-model-card.pdf](https://www-cdn.anthropic.com/f2986af8d052f26236f6251da62d16172cfabd6e/claude-3-model-card.pdf)

+   Balzer（1985）Robert Balzer. 1985年. 《自动编程的15年回顾》. *IEEE软件工程学报* 11 (1985), 1257–1268.

+   Cai等人（2024）Yufan Cai, Zhe Hou, Xiaokun Luan, David Miguel Sanan Baena, Yun Lin, Jun Sun, 和 Jin Song Dong. 2024年. 《面向大语言模型辅助程序优化》. *arXiv预印本 arXiv:2406.18616* (2024).

+   Chen等人（2023）Xinyun Chen, Maxwell Lin, Nathanael Schärli, 和 Denny Zhou. 2023年. 《教会大语言模型自我调试》. *arXiv预印本 arXiv:2304.05128* (2023).

+   Choudhary等人（2022）Kamal Choudhary, Brian DeCost, Chi Chen, Anubhav Jain, Francesca Tavazza, Ryan Cohn, Cheol Woo Park, Alok Choudhary, Ankit Agrawal, Simon JL Billinge等人. 2022年. 《深度学习方法在材料科学中的最新进展与应用》. *npj计算材料* 8, 1 (2022), 59.

+   Coplien（1998）James O Coplien. 1998年. 《生成性开发—》. *模式手册：技术、策略与应用* 13 (1998), 243.

+   DeepSeek（2023）DeepSeek. 2023年. 《DeepSeek Coder：让代码自己写》. [https://github.com/deepseek-ai/DeepSeek-Coder](https://github.com/deepseek-ai/DeepSeek-Coder).

+   Dehaerne等人（2022）Enrique Dehaerne, Bappaditya Dey, Sandip Halder, Stefan De Gendt, 和 Wannes Meert. 2022年. 《使用机器学习的代码生成：系统评审》. *IEEE Access* 10 (2022), 82434–82455.

+   Dong等人（2023）Yihong Dong, Xue Jiang, Zhi Jin, 和 Ge Li. 2023年. 《通过ChatGPT实现自我协作代码生成》. *arXiv预印本 arXiv:2304.07590* (2023).

+   Dubey等人（2024）Abhimanyu Dubey, Abhinav Jauhri, Abhinav Pandey, Abhishek Kadian, Ahmad Al-Dahle, Aiesha Letman, Akhil Mathur, Alan Schelten, Amy Yang, Angela Fan等人. 2024年. 《Llama 3模型家族》. *arXiv预印本 arXiv:2407.21783* (2024).

+   Feng 等人（2020）张银凤、郭大亚、唐独宇、段楠、冯小成、龚鸣、寿林俊、秦兵、刘婷、姜大新等。2020。Codebert：一种针对编程和自然语言的预训练模型。*arXiv 预印本 arXiv:2002.08155*（2020）。

+   Floridi 和 Chiriatti（2020）Luciano Floridi 和 Massimo Chiriatti。2020。GPT-3：它的本质、范围、限制和后果。*思想与机器* 30（2020），681–694。

+   Gómez-de Mariscal 等人（2021）Estibaliz Gómez-de Mariscal、Carlos García-López-de Haro、Wei Ouyang、Laurène Donati、Emma Lundberg、Michael Unser、Arrate Muñoz-Barrutia 和 Daniel Sage。2021。DeepImageJ：一个用户友好的环境，用于在 ImageJ 中运行深度学习模型。*自然方法* 18，10（2021），1192–1195。

+   Hellas 等人（2023）Arto Hellas、Juho Leinonen、Sami Sarsa、Charles Koutcheme、Lilja Kujanpää 和 Juha Sorva。2023。探索大型语言模型对初学者程序员求助请求的响应。载于 *2023年ACM国际计算教育研究会议论文集-第1卷*。93–105。

+   Huang 等人（2023）黄东、卜庆文、张杰民、迈克尔·ラック、崔鹤鸣。2023。Agentcoder：基于多代理的代码生成与迭代测试和优化。*arXiv 预印本 arXiv:2312.13010*（2023）。

+   Imai（2022）今井咲。2022。GitHub Copilot 是否可以替代人类配对编程？一项实证研究。载于 *ACM/IEEE第44届国际软件工程大会论文集：伴随论文集*。319–321。

+   Jiang 等人（2024）姜居永、王凡、沈佳思、金成周、金圣勋。2024。关于大规模语言模型用于代码生成的综述。*arXiv 预印本 arXiv:2406.00515*（2024）。

+   Jin 等人（2023）Matthew Jin、Syed Shahriar、Michele Tufano、Xin Shi、Shuai Lu、Neel Sundaresan 和 Alexey Svyatkovskiy。2023。Inferfix：基于 LLM 的端到端程序修复。载于 *第31届ACM联合欧洲软件工程会议暨软件工程基础研讨会论文集*。1646–1656。

+   Kushman 和 Barzilay（2013）Nate Kushman 和 Regina Barzilay。2013。利用语义统一从自然语言生成正则表达式。北美计算语言学协会（NAACL）年会。

+   Latha 等人（2014）Latha女士、Poojith A、BA Reddy 和 G Vittal Kumar。2014。农业中的图像处理。*国际电气、电子、仪器和控制工程创新研究期刊* 2，6（2014）。

+   Lewis 等人（2020）Patrick Lewis、Ethan Perez、Aleksandra Piktus、Fabio Petroni、Vladimir Karpukhin、Naman Goyal、Heinrich Küttler、Mike Lewis、Wen-tau Yih、Tim Rocktäschel 等。2020。检索增强生成用于知识密集型 NLP 任务。*神经信息处理系统进展* 33（2020），9459–9474。

+   Li等（2024）Haonan Li, Yu Hao, Yizhuo Zhai, 和 Zhiyun Qian. 2024. 提升静态分析用于实际bug检测：一种LLM集成方法. *ACM编程语言会议录* 8, OOPSLA1 (2024), 474–499.

+   Li等（2023）Raymond Li, Loubna Ben Allal, Yangtian Zi, Niklas Muennighoff, Denis Kocetkov, Chenghao Mou, Marc Marone, Christopher Akiki, Jia Li, Jenny Chim 等. 2023. Starcoder：愿源代码与你同在！ *arXiv预印本 arXiv:2305.06161* (2023).

+   Li等（2022）Yujia Li, David Choi, Junyoung Chung, Nate Kushman, Julian Schrittwieser, Rémi Leblond, Tom Eccles, James Keeling, Felix Gimeno, Agustin Dal Lago 等. 2022. 竞争级别的代码生成与alphacode. *科学* 378, 6624 (2022), 1092–1097.

+   Liu等（2023）Jiawei Liu, Chunqiu Steven Xia, Yuyao Wang, 和 LINGMING ZHANG. 2023. 你的代码真的是ChatGPT生成的吗？大规模语言模型在代码生成中的严格评估. 收录于 *第37届神经信息处理系统会议*. [https://openreview.net/forum?id=1qvx610Cu7](https://openreview.net/forum?id=1qvx610Cu7)

+   Manshadi等（2013）Mehdi Manshadi, Daniel Gildea, 和 James Allen. 2013. 集成示例编程与自然语言编程. 收录于 *AAAI人工智能会议录*，第27卷. 661–667.

+   Merkel等（2014）Dirk Merkel等. 2014. Docker：用于一致开发和部署的轻量级Linux容器. *Linux j* 239, 2 (2014), 2.

+   OpenAI（2024）OpenAI. 2024. 学会与LLMs推理. [https://openai.com/index/learning-to-reason-with-llms/](https://openai.com/index/learning-to-reason-with-llms/)

+   Prabaharan等（2020）T Prabaharan, P Periasamy, V Mugendiran 等. 2020. 图像处理在各领域应用的研究：综述. 收录于 *IOP会议系列：材料科学与工程*，第961卷. IOP出版，012006.

+   Qian等（2024）Chen Qian, Wei Liu, Hongzhang Liu, Nuo Chen, Yufan Dang, Jiahao Li, Cheng Yang, Weize Chen, Yusheng Su, Xin Cong 等. 2024. Chatdev：用于软件开发的交互式智能体. 收录于 *第62届计算语言学协会年会会议录（第1卷：长篇论文）*，15174–15186.

+   Raza等（2015）Mohammad Raza, Sumit Gulwani, 和 Natasa Milic-Frayling. 2015. 从自然语言和示例合成组合程序. 收录于 *IJCAI 2015*.

+   rchavezj（2019）rchavezj. 2019. OpenCV_Projects. [https://github.com/rchavezj/OpenCV_Projects](https://github.com/rchavezj/OpenCV_Projects).

+   Roziere等（2023）Baptiste Roziere, Jonas Gehring, Fabian Gloeckle, Sten Sootla, Itai Gat, Xiaoqing Ellen Tan, Yossi Adi, Jingyu Liu, Tal Remez, Jérémy Rapin 等. 2023. Code llama：面向代码的开放基础模型. *arXiv预印本 arXiv:2308.12950* (2023).

+   Schroeder 等 (2021) Alexandra B Schroeder, Ellen TA Dobson, Curtis T Rueden, Pavel Tomancak, Florian Jug, 和 Kevin W Eliceiri. 2021. ImageJ 生态系统：用于图像可视化、处理和分析的开源软件。 *蛋白质科学* 30, 1 (2021), 234–249。

+   Suganyadevi 等 (2022) S Suganyadevi, V Seethalakshmi, 和 Krishnasamy Balasamy. 2022. 深度学习在医学图像分析中的应用综述。 *国际多媒体信息检索期刊* 11, 1 (2022), 19–38。

+   Svyatkovskiy 等 (2020) Alexey Svyatkovskiy, Shao Kun Deng, Shengyu Fu, 和 Neel Sundaresan. 2020. Intellicode compose: 使用变换器进行代码生成。 在 *第28届ACM欧洲软件工程会议联合会议与软件工程基础研讨会论文集*。1433–1443。

+   团队 (2024a) CodeGemma 团队. 2024a. Codegemma: 基于 Gemma 的开放代码模型。 *arXiv 预印本 arXiv:2406.11409* (2024)。

+   团队 (2024b) Qwen 团队. 2024b. Qwen2.5: 一个基础模型的聚会。 [https://qwenlm.github.io/blog/qwen2.5/](https://qwenlm.github.io/blog/qwen2.5/)

+   Vaswani 等 (2017) Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N Gomez, Łukasz Kaiser, 和 Illia Polosukhin. 2017. 注意力即你所需要的一切。 *神经信息处理系统进展* 30 (2017)。

+   Wang 等 (2024) Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin 等. 2024. 基于大语言模型的自主代理综述。 *计算机科学前沿* 18, 6 (2024), 186345。

+   Wang 等 (2022) Xuezhi Wang, Jason Wei, Dale Schuurmans, Quoc Le, Ed Chi, Sharan Narang, Aakanksha Chowdhery, 和 Denny Zhou. 2022. 自一致性提升语言模型的思维链推理。 *arXiv 预印本 arXiv:2203.11171* (2022)。

+   Wei 等 (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou 等. 2022. 思维链提示引发大语言模型中的推理。 *神经信息处理系统进展* 35 (2022), 24824–24837。

+   Wei 等 (2023) Yuxiang Wei, Zhe Wang, Jiawei Liu, Yifeng Ding, 和 Lingming Zhang. 2023. Magicoder: 源代码即你所需要的一切。 *arXiv 预印本 arXiv:2312.02120* (2023)。

+   Xi 等 (2023) Zhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, Boyang Hong, Ming Zhang, Junzhe Wang, Senjie Jin, Enyu Zhou 等. 2023. 基于大语言模型的代理的崛起与潜力：一项调查。 *arXiv 预印本 arXiv:2309.07864* (2023)。

+   Yao 等 (2023) Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Thomas L. Griffiths, Yuan Cao, 和 Karthik Narasimhan. 2023. 思维树：使用大语言模型进行深思熟虑的问题解决。 arXiv:2305.10601 [cs.CL] [https://arxiv.org/abs/2305.10601](https://arxiv.org/abs/2305.10601)

+   Yin 和 Neubig（2017）Pengcheng Yin 和 Graham Neubig. 2017. 用于通用代码生成的句法神经模型。*arXiv 预印本 arXiv:1704.01696* (2017)。

+   yoyoyo yo（2019）yoyoyo yo. 2019. Gasyori100knock. [https://github.com/yoyoyo-yo/Gasyori100knock](https://github.com/yoyoyo-yo/Gasyori100knock)。

+   Yushkevich 等人（2016）Paul A Yushkevich, Yang Gao, 和 Guido Gerig. 2016. ITK-SNAP: 一种用于多模态生物医学图像半自动分割的交互式工具。在 *2016 第38届 IEEE 医学与生物学工程国际会议（EMBC）* 中。IEEE, 3342–3345。

+   张等人（2024a）Jialu Zhang, José Pablo Cambronero, Sumit Gulwani, Vu Le, Ruzica Piskac, Gustavo Soares, 和 Gust Verbruggen. 2024a. Pydex: 使用大语言模型修复 Python 入门作业中的错误。*ACM 程序语言学报* 8, OOPSLA1 (2024), 1100–1124。

+   张等人（2024b）Kechi Zhang, Jia Li, Ge Li, Xianjie Shi, 和 Zhi Jin. 2024b. Codeagent: 通过工具集成代理系统增强代码生成，解决现实世界的仓库级编码挑战。*arXiv 预印本 arXiv:2401.07339* (2024)。

+   张等人（2023b）Kechi Zhang, Zhuo Li, Jia Li, Ge Li, 和 Zhi Jin. 2023b. Self-edit: 面向代码生成的故障感知代码编辑器。*arXiv 预印本 arXiv:2305.04087* (2023)。

+   张等人（2023c）Kechi Zhang, Huangzhao Zhang, Ge Li, Jia Li, Zhuo Li, 和 Zhi Jin. 2023c. Toolcoder: 教授代码生成模型使用API搜索工具。*arXiv 预印本 arXiv:2305.04032* (2023)。

+   张等人（2023a）Yue Zhang, Yafu Li, Leyang Cui, Deng Cai, Lemao Liu, Tingchen Fu, Xinting Huang, Enbo Zhao, Yu Zhang, Yulong Chen, 等人. 2023a. AI 海洋中的海妖之歌：关于大语言模型中的幻觉现象的调查。*arXiv 预印本 arXiv:2309.01219* (2023)。
