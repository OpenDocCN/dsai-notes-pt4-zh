<!--yml

分类：未分类

date: 2025-01-11 12:39:01

-->

# AIOS 编译器：LLM 作为自然语言编程和 AI 代理流编程的解释器

> 来源：[https://arxiv.org/html/2405.06907/](https://arxiv.org/html/2405.06907/)

徐书远

罗格斯大学

shuyuan.xu@rutgers.edu

Zelong Li ¹¹footnotemark: 1

罗格斯大学

zelong.li@rutgers.edu

Mei Kai

罗格斯大学

kai.mei@rutgers.edu

张永丰

罗格斯大学

yongfeng.zhang@rutgers.edu 两位作者对本工作做出了同等贡献。通讯作者

###### 摘要

自编程语言诞生以来，它们趋向于更高的可读性和为程序员降低门槛。沿着这一趋势，自然语言作为一种编程语言具有广阔的前景，能够提供极大的灵活性和可用性，并有助于编程的民主化。然而，自然语言固有的模糊性、不确定性和冗长性给开发能够准确理解编程逻辑并执行用自然语言编写的指令的解释器带来了巨大挑战。幸运的是，最近在大型语言模型（LLM）方面的进展展示了它们在解释复杂自然语言方面的卓越能力。受此启发，我们开发了一个新型的代码表示与执行系统（CoRE），该系统利用 LLM 作为解释器来解释和执行自然语言程序（NLPg）。该系统将自然语言编程、伪代码编程和流编程统一在同一表示下，用于构建语言代理，而 LLM 则作为解释器来解释和执行代理程序。本文首先定义了编程语法，以逻辑地构建自然语言指令。在执行过程中，我们结合了外部记忆来减少冗余。此外，我们还为设计的解释器提供了调用外部工具的能力，以弥补 LLM 在专业领域或访问实时信息时的局限性。此工作是开源的，代码托管在 [https://github.com/agiresearch/CoRE](https://github.com/agiresearch/CoRE)、[https://github.com/agiresearch/OpenAGI](https://github.com/agiresearch/OpenAGI) 和 [https://github.com/agiresearch/AIOS](https://github.com/agiresearch/AIOS)。

## 1 引言

图 1：在我们的 CoRE 系统中，我们设计了 CoRE 语言，以统一自然语言编程、伪代码编程和流编程，采用相同的语法表示。我们以 OpenAGI [[14](https://arxiv.org/html/2405.06907v2#bib.bib14)] 平台的程序为例。

编程对计算机至关重要，因为它使计算机能够根据一组预定义的指令执行特定任务。编程让我们能够利用逻辑算法，使计算机解决问题。自编程语言诞生以来，编程经历了显著的演变，新的技术和创新推动了其发展。最初，编程语言基于二进制机器语言，如穿孔卡片，可以被机器直接执行。然而，机器语言几乎无法被人类读取。随后，低级编程语言，如汇编语言，使用助记符指令和操作数来表示机器码，从而增强了可读性[[18](https://arxiv.org/html/2405.06907v2#bib.bib18)]。然而，由于需要控制内存位置和寄存器，汇编语言对程序员仍然有较高的入门门槛。随着C/C++、Java和Python等高级编程语言的设计，编程变得更加易用和高效。它们为程序员提供了一种更具生产力和可接近性的方法，促进了更多人参与编程和软件开发。因此，编程语言正越来越多地融入到日常生活中。

从编程语言的历史来看，我们可以观察到一个明确的趋势，那就是编程的可用性、可读性和民主性不断提高。沿着这个趋势，天然语言作为编程语言的选择具有很大的吸引力，因为它易于接触、可读性强，而且对程序员的培训要求较低。然而，天然语言编程的应用也面临挑战，因为天然语言本身存在模糊性、歧义性和冗长性。最近出现的大型语言模型（LLMs）为这一挑战提供了解决方案，得益于它们在语言理解[[38](https://arxiv.org/html/2405.06907v2#bib.bib38)、[8](https://arxiv.org/html/2405.06907v2#bib.bib8)]、工具使用和函数调用[[14](https://arxiv.org/html/2405.06907v2#bib.bib14)、[42](https://arxiv.org/html/2405.06907v2#bib.bib42)]，以及与人类或环境的互动[[43](https://arxiv.org/html/2405.06907v2#bib.bib43)、[12](https://arxiv.org/html/2405.06907v2#bib.bib12)]方面的非凡能力。在这项工作中，我们提出了一种新的代码表示与执行系统（CoRE），该系统将LLM作为解释器来解释和执行天然语言中的指令，从而实现用天然语言进行代理编程。

CoRE可以用于自然语言编程、伪代码编程和流程编程，因为这三种形式的代理程序统一为我们的CoRE语言，如图[1](https://arxiv.org/html/2405.06907v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents")中的示例所示。在编程领域，基本任务是设计和开发逻辑结构化的指令，以解决特定问题。自然语言编程提供了一种方法，其中指令以日常语言形式表达，使得代码直观易懂。当我们以逻辑方式构建所有自然语言指令时，它本质上反映了伪代码编程的核心特征。伪代码的设计简化了编码过程，去除了语法复杂性，专注于算法逻辑，以便于理解。因此，当指令以自然语言表达时，这些结构化的指令可以被视为伪代码。此外，伪代码与流程编程有直接关系，因为它本质上代表了可以无缝可视化为工作流的算法逻辑。工作流反过来提供了程序逐步执行的图形化表示，强调了决策过程的可视化以及程序中控制流的流动。

在设计一个以LLM作为解释器的自然语言编程新系统时，我们面临了几个重大挑战。首先，如何使用自然语言指令表示程序的逻辑。为了解决这个问题，我们设计了一套编程语法来逻辑性地结构化自然语言指令，并将自然语言编程、伪代码编程和流程编程统一在同一表示中。其次，考虑到程序由一步步的指令组成，确保每一步都根据其对应的指令执行至关重要。为确保每一步中的自然语言指令能够精确执行，我们设计了两个附加组件：一个用于从内存中检索信息，另一个用于调用外部工具。考虑到LLM在输入令牌数量（上下文窗口大小）上的限制，将所有运行时信息包含在输入提示中是不现实的。为了解决这个问题，我们将大量中间结果存储在临时内存中，并在后续步骤中根据需要检索相关信息[[48](https://arxiv.org/html/2405.06907v2#bib.bib48)，[34](https://arxiv.org/html/2405.06907v2#bib.bib34)，[27](https://arxiv.org/html/2405.06907v2#bib.bib27)，[4](https://arxiv.org/html/2405.06907v2#bib.bib4)]。此外，虽然LLM擅长处理文本信息，但在处理需要领域特定知识或最新信息的任务时，它们通常表现不佳[[15](https://arxiv.org/html/2405.06907v2#bib.bib15)]。为了解决这些限制，我们使LLM能够利用外部工具来解决问题[[14](https://arxiv.org/html/2405.06907v2#bib.bib14)，[42](https://arxiv.org/html/2405.06907v2#bib.bib42)，[30](https://arxiv.org/html/2405.06907v2#bib.bib30)，[2](https://arxiv.org/html/2405.06907v2#bib.bib2)]。最后，在执行自然语言程序时，错误地确定下一步可能导致不同的最终结果。我们通过要求LLM解释器评估当前结果，从而确定最合适的后续步骤来解决这个问题。CoRE的整体执行流程如图[2](https://arxiv.org/html/2405.06907v2#S1.F2 "Figure 2 ‣ 1 Introduction ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents")所示。

图2：一个示例，展示了CoRE系统如何执行一步操作。

总结来说，本文的主要贡献如下：

+   •

    我们设计了一种CoRE语言，它将自然语言编程、伪代码编程和流程编程统一在一起。CoRE语言逻辑性地结构化了自然语言指令。

+   •

    我们提出了CoRE系统，该系统利用大型语言模型（LLM）作为解释器，逐步解释并执行指令。在执行过程中，LLM遵循指令，并利用信息检索和外部工具来增强其效果。

+   •

    我们基于公共基准数据集验证了系统的有效性和效率。具体而言，我们采用了我们提出的系统，通过基于自然语言程序的代理任务解决，展示了其实际能力。

在本文的接下来的部分，我们首先在第[2](https://arxiv.org/html/2405.06907v2#S2 "2 Related Work ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents")节中提供相关工作。在第[3](https://arxiv.org/html/2405.06907v2#S3 "3 The CoRE System ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents")节中，我们介绍了CoRE框架以及该框架如何应用于LLM代理。在第[4](https://arxiv.org/html/2405.06907v2#S4 "4 Experiments ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents")节中，我们提供了实验结果，并在第[5](https://arxiv.org/html/2405.06907v2#S5 "5 Conclusions and Future Work ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents")节中总结了工作并提出未来的研究方向。

## 2 相关工作

### 2.1 自然语言编程

自然语言编程的研究[[17](https://arxiv.org/html/2405.06907v2#bib.bib17), [5](https://arxiv.org/html/2405.06907v2#bib.bib5), [46](https://arxiv.org/html/2405.06907v2#bib.bib46), [28](https://arxiv.org/html/2405.06907v2#bib.bib28), [10](https://arxiv.org/html/2405.06907v2#bib.bib10), [13](https://arxiv.org/html/2405.06907v2#bib.bib13)] 主要集中在解决将自然语言转换为编程语言语句中的歧义问题。[Heidorn](https://arxiv.org/html/2405.06907v2#bib.bib17) [[17](https://arxiv.org/html/2405.06907v2#bib.bib17)] 提出了采用启发式NLP编码和解码规则，开发一种能够接受自然语言对话的自动编程系统。[Vadas 和 Curran](https://arxiv.org/html/2405.06907v2#bib.bib46) [[46](https://arxiv.org/html/2405.06907v2#bib.bib46)] 介绍了一个原型系统，该系统能够使用组合范畴语法（CCG）解析器将特定的英语指令转换为可执行的Python代码，该解析器采用无限制的语法，涵盖了广泛的用户指令语义。[Mihalcea 等](https://arxiv.org/html/2405.06907v2#bib.bib35) [[35](https://arxiv.org/html/2405.06907v2#bib.bib35)] 实现了一种过程化的自然语言编程系统，用于将自然语言转换为编程语言。早期的自然语言编程技术由于需要创建特定领域语言（DSLs），在可扩展性方面受到限制。为了避免重复设计新DSLs的问题，[Desai 等](https://arxiv.org/html/2405.06907v2#bib.bib10) [[10](https://arxiv.org/html/2405.06907v2#bib.bib10)] 提出了一个通用的生成框架，用于构建一个接受自然语言输入并生成目标DSL中表达式的程序。此外，[Ernst](https://arxiv.org/html/2405.06907v2#bib.bib13) [[13](https://arxiv.org/html/2405.06907v2#bib.bib13)] 利用神经网络，即循环神经网络（RNN），将文件系统操作的英语规范转换为相应的bash命令。

### 2.2 大型语言模型和人工智能代理在问题解决中的应用

大型语言模型（LLMs）作为解决问题的强大工具，已广泛应用于推理、规划和代码生成等任务。LLM 推理通常涉及将复杂任务分解为一系列步骤，也称为推理链[[49](https://arxiv.org/html/2405.06907v2#bib.bib49)]。LLM 推理中的著名方法包括思维链（Chain-of-Thought, CoT）及其衍生方法[[49](https://arxiv.org/html/2405.06907v2#bib.bib49), [26](https://arxiv.org/html/2405.06907v2#bib.bib26)]。为进一步提升 LLM 的推理能力，已经提出了多种方法。自一致性方法[[47](https://arxiv.org/html/2405.06907v2#bib.bib47)]通过对多个推理路径进行采样，并通过投票选出最一致的结果。此外，像树和图这样的经典数据结构也被用于提升推理效率和准确性，减少步骤[[52](https://arxiv.org/html/2405.06907v2#bib.bib52), [3](https://arxiv.org/html/2405.06907v2#bib.bib3)]。除了推理，规划也是一种重要的任务，可用于解决问题。LLM 规划涉及生成一系列行动以实现预定目标[[16](https://arxiv.org/html/2405.06907v2#bib.bib16)]。最近的进展包括直接提示 LLM 进行规划任务，取得了有希望的成果[[20](https://arxiv.org/html/2405.06907v2#bib.bib20), [45](https://arxiv.org/html/2405.06907v2#bib.bib45), [11](https://arxiv.org/html/2405.06907v2#bib.bib11)]。有限状态机已被集成到 LLM 中，以增强其规划能力[[29](https://arxiv.org/html/2405.06907v2#bib.bib29), [51](https://arxiv.org/html/2405.06907v2#bib.bib51)]。ReAct [[53](https://arxiv.org/html/2405.06907v2#bib.bib53)] 提出了利用外部工具，如搜索引擎，来增强 LLM 规划的能力。此外，考虑到 LLM 在编程方面的强大能力，近期的研究提出了生成编程代码来解决问题[[32](https://arxiv.org/html/2405.06907v2#bib.bib32), [22](https://arxiv.org/html/2405.06907v2#bib.bib22), [31](https://arxiv.org/html/2405.06907v2#bib.bib31), [6](https://arxiv.org/html/2405.06907v2#bib.bib6), [23](https://arxiv.org/html/2405.06907v2#bib.bib23), [40](https://arxiv.org/html/2405.06907v2#bib.bib40), [36](https://arxiv.org/html/2405.06907v2#bib.bib36)]。此外，“自我反思”机制[[33](https://arxiv.org/html/2405.06907v2#bib.bib33), [39](https://arxiv.org/html/2405.06907v2#bib.bib39), [44](https://arxiv.org/html/2405.06907v2#bib.bib44)]使得 LLM 能够批评其自身输出，显著提高了推理[[3](https://arxiv.org/html/2405.06907v2#bib.bib3)]和代码生成[[7](https://arxiv.org/html/2405.06907v2#bib.bib7)]等任务的表现。与现有的直接使用 LLM 生成解决方案的方法相比，提出的 CoRE 系统利用 LLM 作为解释器，执行由人类设计的解决方案，以解决复杂问题。这一方法结合了人类在解决方案设计中的创造力和 LLM 的能力，从而增强了在自然语言编程环境中的问题解决能力。

图 3：CoRE LLM 解释器系统概述。

## 3 CoRE 系统

在本节中，我们将介绍如何定义自然语言编程语法，以及如何使用 LLM 作为解释器来解释和执行自然语言程序。

### 3.1 CoRE 语言语法

为了组织自然语言指令，我们定义了每个步骤的基本结构表示，它由四个组件组成。一个示例可以在图 [1](https://arxiv.org/html/2405.06907v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents") 中找到。

+   •

    步骤名称：程序中的每个步骤通过步骤名称唯一标识。这个标识符类似于传统编程语言中的函数标识符，便于在程序结构中进行导航和引用，确保程序中的每个操作都能被明确定位和访问。

+   •

    步骤类型：步骤类型对每个步骤中执行的操作性质进行分类，类似于传统编程中的控制结构。我们定义了三种主要的步骤类型：

    +   –

        过程：类似于传统编程中的过程语句，这种步骤类型执行特定操作，并过渡到下一个指定的步骤。

    +   –

        决策：对应于条件语句（例如“if-else”），此步骤根据评估的条件将程序流程分支，导致多个潜在的路径。

    +   –

        终止：类似于“end”或“return”语句，这个步骤标志着程序的结束，表示不再执行任何后续步骤。

+   •

    步骤指令：步骤指令阐明了在步骤中要执行的任务。该组件至关重要，因为它提供了执行的指令和内容，类似于传统编程语言中的语句块。通过自然语言演示操作，NLPg 降低了编程的门槛，使其对非专业程序员更具可读性。

+   •

    步骤连接：步骤连接定义了从一个步骤到另一个步骤的进展，建立了程序执行的流程。在过程步骤中，指定了单一的后续步骤；在决策步骤中，根据条件划定了多个路径。终止步骤则没有后续步骤，标志着程序执行的结束。

对于程序中的每个步骤，以上四个组件通过“:::”进行分隔（如图 [1](https://arxiv.org/html/2405.06907v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents") 中所示）。也可以使用其他特殊标记来分隔不同的组件。

图 4：一个示例，展示了 CoRE 系统如何检索相关信息。

在编程语言中，有三种基本的控制结构[[9](https://arxiv.org/html/2405.06907v2#bib.bib9), [41](https://arxiv.org/html/2405.06907v2#bib.bib41)]：序列、选择和迭代。这三种基本结构可以很容易地在CoRE语言中设计。

+   •

    序列：编程中的序列是按线性顺序执行语句，每个语句都引导到下一个语句。在CoRE框架中，这一结构是通过设置“步骤连接”来指向后续步骤来实现的。每个步骤在处理类型下运行，直到序列结束。

+   •

    选择：编程语言中的选择使得条件分支成为可能，允许程序根据特定条件执行不同的步骤序列。这是通过使用决策步骤类型来实现的，其中“步骤连接”部分明确列出了多个可能的路径。每个分支由“步骤连接”部分中陈述的条件定义，依据这些条件引导程序流向不同的步骤。

+   •

    迭代：迭代涉及重复一组操作，直到满足某个条件，类似于传统编程中的循环。在CoRE框架中，我们使用带有决策类型的步骤来评估循环条件是否已满足。在一个循环周期结束时，“步骤连接”被配置为指回先前的决策步骤，从而使循环得以继续。

### 3.2 LLM作为解释器

在本节中，我们将讨论CoRE系统如何利用大型语言模型（LLM）作为解释器来执行用CoRE语言编写的程序。我们将展示CoRE系统中单个步骤的执行过程，具体如图[3](https://arxiv.org/html/2405.06907v2#S2.F3 "Figure 3 ‣ 2.2 Large Language Models and AI Agents for Problem Solving ‣ 2 Related Work ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents")所示。更具体来说，系统在四个程序中执行单个步骤。首先，解释器确定执行当前步骤所需的有效信息。接着，解释器将整合所有相关信息来构建提示。基于生成的提示，解释器将生成响应，并可能利用工具来执行当前步骤。最后，在执行当前步骤后，解释器将根据步骤类型和执行结果确定下一个步骤。我们将详细介绍这四个部分。

#### 3.2.1 从记忆中提取观察结果

这一初步过程至关重要，因为它为当前步骤的整个执行过程奠定了基础。图[4](https://arxiv.org/html/2405.06907v2#S3.F4 "Figure 4 ‣ 3.1 CoRE Language Syntax ‣ 3 The CoRE System ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents")展示了一个例子。系统的记忆充当所有与程序相关的先前观察的存储库，其中观察代表工具执行的结果，如搜索结果。在此阶段，解释器扫描记忆，识别与当前指令相关的记录。这种选择性检索确保了解释器的决策基于准确且与上下文相关的数据，对于程序的成功执行至关重要。

图5：一个示例，展示CoRE系统如何分析LLM解释器的输出。

#### 3.2.2 输入提示构建

构建提示本质上是将信息综合成一个全面而连贯的查询，使LLM能够理解并有效地作出响应。这涉及将多个信息点合并成一个单一的、结构化的提示，指导LLM生成最合适且与上下文相关的回应。在CoRE系统中，解释器构建包含四个要素的详细提示：

+   •

    任务描述：定义整个程序的查询，充当引导系统操作的主要输入。

+   •

    当前进展：总结前面的步骤，包括已经完成或已决定的内容，有助于保持叙事的连贯性。

+   •

    观察：这一部分可能不会出现在每个步骤中。当解释器从记忆中检索到相关信息时，它会被纳入此处。

+   •

    当前指令：以自然语言指定需要执行的操作，指导解释器在当前步骤中如何继续。

#### 3.2.3 输出分析

虽然LLM可以生成直接的响应，但复杂任务可能需要超出其直接范围的能力。必要时，结合使用专门的工具可以扩展LLM的能力，使系统能够有效地处理更广泛的任务。图[5](https://arxiv.org/html/2405.06907v2#S3.F5 "图5 ‣ 3.2.1 从记忆中检索观察 ‣ 3.2 LLM作为解释器 ‣ 3 CoRE系统 ‣ AIOS编译器：LLM作为自然语言编程和AI代理流编程的解释器")展示了执行过程的示范例子。在CoRE系统中，解释器将根据LLM的初步响应和当前任务的需求，决定是否使用专门工具，从而确保系统保持高效功能和多样性，积极解决问题，而不仅仅是处理当前步骤的语言提示。具体来说，如果需要使用工具，系统将选择合适的工具，配置必要的参数，执行它，并将输出整合到正在进行的过程中。

#### 3.2.4 分支分析

图6：一个示例，展示CoRE系统如何确定流程中的下一步。

确定程序中的适当下一步至关重要，特别是在多分支场景中，不同的结果可能导致不同的后续行动。图[6](https://arxiv.org/html/2405.06907v2#S3.F6 "图6 ‣ 3.2.4 分支分析 ‣ 3.2 LLM作为解释器 ‣ 3 CoRE系统 ‣ AIOS编译器：LLM作为自然语言编程和AI代理流编程的解释器")展示了一个示例。在CoRE语言解释器中，决策步骤表示多个分支及相应的条件。解释器使用LLM判断提示是否满足自然语言描述的分支条件，并决定采取哪个下一步。这个适应性方法使得系统能够有效地通过决策点，确保程序目标的逻辑推进。

## 4 实验

### 4.1 主干大语言模型（LLM）

我们对闭源和开源LLM进行了实验：

+   •

    GPT-4 [[37](https://arxiv.org/html/2405.06907v2#bib.bib37)]（闭源）是OpenAI的生成性预训练变换器。在本研究中，我们使用的是GPT-4-1106-preview版本。

+   •

    Mixtral-8x7B [[21](https://arxiv.org/html/2405.06907v2#bib.bib21)]（开源）是一个预训练的生成性稀疏专家混合模型，拥有467亿个参数。

### 4.2 LLM的规划方案

我们采用以下基于LLM的代理规划方案：

+   •

    零-shot学习（Zero）直接将查询输入LLM。

+   •

    思维链（CoT）[[49](https://arxiv.org/html/2405.06907v2#bib.bib49)]促使LLM生成一个连贯的语言序列，作为输入查询和输出答案之间有意义的中间步骤。

+   •

    Few-shot学习（Few）在提示中提供了一组高质量的示例，每个示例都包含目标任务的输入和期望输出。

+   •

    CoRE是我们基于自然语言的编程方法，使用LLM作为解释器。

### 4.3 基准数据集

我们在一个基准数据集OpenAGI上进行实验[[14](https://arxiv.org/html/2405.06907v2#bib.bib14)]。OpenAGI基准任务根据其输出类型和真实标签类型（任务1、2和3）进行分类。然后，根据不同的任务类型，采用不同的指标来衡量性能：CLIP Score [[19](https://arxiv.org/html/2405.06907v2#bib.bib19)]，用于评估文本与图像之间的相似性，应用于文本到图像任务；BERT Score [[55](https://arxiv.org/html/2405.06907v2#bib.bib55)]，用于通过BERT评估文本生成，在数据标签和期望输出都是文本时使用；ViT Score [[50](https://arxiv.org/html/2405.06907v2#bib.bib50)]用于衡量图像标签与图像输出之间的相似性。

### 4.4 实现细节

我们的框架和所有基准方法都是通过PyTorch实现的，PyTorch是一个开源库。我们遵循OpenAGI平台的实现设置[[14](https://arxiv.org/html/2405.06907v2#bib.bib14)]来进行零-shot和few-shot学习。我们利用DSPy框架[[24](https://arxiv.org/html/2405.06907v2#bib.bib24), [25](https://arxiv.org/html/2405.06907v2#bib.bib25)]将CoT策略应用于OpenAGI平台。我们还在OpenAGI平台上尝试了Program-of-Thought [[6](https://arxiv.org/html/2405.06907v2#bib.bib6)]和ReAct [[54](https://arxiv.org/html/2405.06907v2#bib.bib54)]策略。然而，ReAct策略需要文本观察，这对于我们的OpenAGI任务并不适用，因为某些观察是图像格式，而Program-of-Thought无法生成可执行代码。因此，我们没有将它们作为基准方法。

### 4.5 实验分析

OpenAGI 基准测试的实验结果见表 [1](https://arxiv.org/html/2405.06907v2#S4.T1 "Table 1 ‣ 4.5 Experimental Analysis ‣ 4 Experiments ‣ AIOS Compiler: LLM as Interpreter for Natural Language Programming and Flow Programming of AI Agents")。每一行代表一个任务类型，每一列代表 LLM 解释器的规划方案，每四列显示同一 LLM 解释器的结果。从结果中可以看出，无论在 Mixtral 还是 GPT-4 作为解释器的情况下，我们的 CoRE 规划方案在平均性能上都优于任何基线。当使用 Mixtral 作为解释器时，CoRE 在每种任务类型下都超越了 Zero-shot 和 CoT，并且在任务 2 和平均分上优于 Few-shot 学习，尽管在任务 3 上稍逊色，在任务 1 上略差。当使用 GPT-4 作为解释器时，CoT 和 Few-shot 在任务 1 和任务 3 上表现相似，而在任务 2 和平均分上，CoRE 仍然是最好的。需要指出的是，直接比较 CoRE 和 Few-shot 学习是不公平的，因为我们并没有在提示中直接提供输出格式和输出示例。然而，即使不使用这些示例，CoRE 规划策略的表现仍然优于 Few-shot 策略。我们还发现，即使是相同的 CoRE 程序，在使用不同的 LLM 作为解释器时，系统的表现也可能不同，这意味着自然语言编程的性能依赖于 LLM 解释器的自然语言理解能力。

| 指标 / 任务 | Mixtral（开源）作为 LLM 解释器 | GPT-4（闭源）作为 LLM 解释器 |
| --- | --- | --- |
| Zero | CoT | Few | CoRE（我们的） | Zero | CoT | Few | CoRE（我们的） |
| 任务 1（CLIP 分数） | 0.0 | 0.0 | $0.1839$ | 0.1825 | 0.0 | 0.2732 | 0.1837 | $0.3030$ |
| 任务 2（BERT 分数） | 0.1092 | 0.1987 | 0.0687 | $0.2593$ | 0.2076 | 0.2266 | 0.5277 | $0.5756$ |
| 任务 3（ViT 分数） | 0.1949 | 0.1562 | $0.5501$ | 0.2437 | 0.5058 | 0.6736 | $0.6916$ | 0.6611 |
| 任务平均值 | 0.1206 | 0.1736 | 0.1887 | $0.2483$ | 0.2378 | 0.3359 | 0.5391 | $0.5744$ |
| 有效计划的百分比 | 23.08 | 38.46 | 46.15 | $56.92$ | 53.85 | 60.00 | 83.08 | $92.31$ |

表 1：OpenAGI [[14](https://arxiv.org/html/2405.06907v2#bib.bib14)] 基准任务在不同设置下的表现。Zero 代表零样本学习（Zero-shot Learning），Few 代表少样本学习（Few-shot Learning）。加粗的数字表示在使用相同 LLM 的情况下，每个任务类型的最高分数。

## 5 结论与未来工作

在这项研究中，我们介绍了一种新颖的系统，CoRE，用于代码表示与执行。CoRE 旨在通过开发统一的 CoRE 语言，连接自然语言编程、伪代码和流程编程，从而构建 AI 代理。CoRE 利用自然语言作为编程接口，降低了编程的门槛，倡导编程民主化，使得即使是普通用户也能创建他们的 AI 代理。我们的系统利用大型语言模型（LLMs）作为解释器来处理和执行自然语言指令。在执行过程中，解释器动态地检索必要的信息，使用适当的外部工具，并根据先前的输出导航指令。实验结果验证了 CoRE 系统在自然语言编程中的有效性。

虽然 CoRE 展示了有前景的结果，但它目前依赖于手动编写的程序，这可能由于自然语言固有的模糊性而引入低效。为了解决这一问题，未来的研究可以探索开发自动化系统来生成自然语言编程指令。这种自动化将有助于标准化指令的清晰度和精确性，从而潜在地提高系统性能。此外，未来的方向是扩展 CoRE 的语言支持，以促进国际化使用，并实现实时调试功能，以帮助教育并协助初学者，进一步扩大系统的实用性和可访问性。

## 参考文献

+   [1]

+   Ahn 等人 [2022] Michael Ahn, Anthony Brohan, Noah Brown, Yevgen Chebotar, Omar Cortes, Byron David, Chelsea Finn, Chuyuan Fu, Keerthana Gopalakrishnan, Karol Hausman 等人. 2022. Do as i can, not as i say: 将语言与机器人能力相结合。*arXiv 预印本 arXiv:2204.01691* (2022).

+   Besta 等人 [2024] Maciej Besta, Nils Blach, Ales Kubicek, Robert Gerstenberger, Michal Podstawski, Lukas Gianinazzi, Joanna Gajda, Tomasz Lehmann, Hubert Niewiadomski, Piotr Nyczyk 等人. 2024. 思维图谱：利用大型语言模型解决复杂问题。发表于 *AAAI 人工智能会议论文集*，第 38 卷。17682–17690.

+   Borgeaud 等人 [2022] Sebastian Borgeaud, Arthur Mensch, Jordan Hoffmann, Trevor Cai, Eliza Rutherford, Katie Millican, George Bm Van Den Driessche, Jean-Baptiste Lespiau, Bogdan Damoc, Aidan Clark 等人. 2022. 通过从万亿级的标记中检索来改进语言模型。发表于 *国际机器学习会议*。PMLR, 2206–2240.

+   Bruckman 和 Edwards [1999] Amy Bruckman 和 Elizabeth Edwards. 1999. 我们是否应该利用自然语言知识？对自然语言风格编程语言中用户错误的分析。发表于 *SIGCHI 人机交互会议论文集*。207–214.

+   Chen et al. [2023b] Wenhu Chen, Xueguang Ma, Xinyi Wang, 和 William W. Cohen. 2023b. 思维程序提示：将计算与推理分离以处理数值推理任务。*机器学习研究交易* (2023)。

+   Chen et al. [2023a] Xinyun Chen, Maxwell Lin, Nathanael Schärli, 和 Denny Zhou. 2023a. 教授大型语言模型自我调试。*arXiv预印本arXiv:2304.05128* (2023)。

+   Chung et al. [2024] Hyung Won Chung, Le Hou, Shayne Longpre, Barret Zoph, Yi Tay, William Fedus, Yunxuan Li, Xuezhi Wang, Mostafa Dehghani, Siddhartha Brahma 等. 2024. 扩展指令微调语言模型。*机器学习研究杂志* 25, 70 (2024)，1–53。

+   Dahl et al. [1972] Ole-Johan Dahl, Edsger Wybe Dijkstra, 和 Charles Antony Richard Hoare. 1972. *结构化编程*。Academic Press Ltd.

+   Desai et al. [2016] Aditya Desai, Sumit Gulwani, Vineet Hingorani, Nidhi Jain, Amey Karkare, Mark Marron, 和 Subhajit Roy. 2016. 使用自然语言进行程序合成。*第38届国际软件工程会议论文集*。345–356。

+   Ding et al. [2023] Yan Ding, Xiaohan Zhang, Chris Paxton, 和 Shiqi Zhang. 2023. 使用大型语言模型进行任务和动作规划以实现物体重排。*2023 IEEE/RSJ国际智能机器人与系统会议（IROS）*。IEEE，2086–2092。

+   Driess et al. [2023] Danny Driess, Fei Xia, Mehdi SM Sajjadi, Corey Lynch, Aakanksha Chowdhery, Brian Ichter, Ayzaan Wahid, Jonathan Tompson, Quan Vuong, Tianhe Yu 等. 2023. Palm-e：一种具身的多模态语言模型。*arXiv预印本arXiv:2303.03378* (2023)。

+   Ernst [2017] Michael D Ernst. 2017. 自然语言是一种编程语言：将自然语言处理应用于软件开发。*第二届编程语言进展峰会（SNAPL 2017）*。Schloss-Dagstuhl-Leibniz计算机中心。

+   Ge et al. [2023a] Yingqiang Ge, Wenyue Hua, Kai Mei, Jianchao Ji, Juntao Tan, Shuyuan Xu, Zelong Li, 和 Yongfeng Zhang. 2023a. OpenAGI：当LLM遇上领域专家。*在神经信息处理系统进展（NeurIPS）* (2023)。

+   Ge et al. [2023b] Yingqiang Ge, Yujie Ren, Wenyue Hua, Shuyuan Xu, Juntao Tan, 和 Yongfeng Zhang. 2023b. LLM作为操作系统，代理作为应用程序：展望AIOS、代理和AIOS-代理生态系统。*arXiv电子预印本* (2023)，arXiv–2312。

+   Hao et al. [2023] Shibo Hao, Yi Gu, Haodi Ma, Joshua Jiahua Hong, Zhen Wang, Daisy Zhe Wang, 和 Zhiting Hu. 2023. 使用语言模型进行推理即使用世界模型进行规划。*arXiv预印本arXiv:2305.14992* (2023)。

+   Heidorn [1976] George E Heidorn. 1976. 通过自然语言对话进行自动编程：一项调查。*IBM研究与开发杂志* 20, 4 (1976)，302–313。

+   Hennessy and Patterson [2011] John L Hennessy 和 David A Patterson. 2011. *计算机架构：一种定量方法*。Elsevier。

+   Hessel 等人 [2021] Jack Hessel, Ari Holtzman, Maxwell Forbes, Ronan Le Bras, 和 Yejin Choi. 2021. CLIPScore: 一种无参考的图像描述评估指标.

+   Huang 等人 [2022] Wenlong Huang, Fei Xia, Ted Xiao, Harris Chan, Jacky Liang, Pete Florence, Andy Zeng, Jonathan Tompson, Igor Mordatch, Yevgen Chebotar, 等人. 2022. 内在独白: 通过规划与语言模型进行具象推理. *arXiv 预印本 arXiv:2207.05608* (2022).

+   Jiang 等人 [2024] Albert Q Jiang, Alexandre Sablayrolles, Antoine Roux, Arthur Mensch, Blanche Savary, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Emma Bou Hanna, Florian Bressand, 等人. 2024. Mixtral 专家系统. *arXiv 预印本 arXiv:2401.04088* (2024).

+   Jojic 等人 [2023] Ana Jojic, Zhen Wang, 和 Nebojsa Jojic. 2023. GPT 正在成为图灵机: 这里有一些编程它的方法. *arXiv 预印本 arXiv:2303.14310* (2023).

+   Josifoski 等人 [2023] Martin Josifoski, Lars Klein, Maxime Peyrard, Yifei Li, Saibo Geng, Julian Paul Schnitzler, Yuxing Yao, Jiheng Wei, Debjit Paul, 和 Robert West. 2023. Flows: 推理和协作 AI 的构建模块. *arXiv 预印本 arXiv:2308.01285* (2023).

+   Khattab 等人 [2022] Omar Khattab, Keshav Santhanam, Xiang Lisa Li, David Hall, Percy Liang, Christopher Potts, 和 Matei Zaharia. 2022. Demonstrate-Search-Predict: 组合检索和语言模型以应对知识密集型 NLP. *arXiv 预印本 arXiv:2212.14024* (2022).

+   Khattab 等人 [2023] Omar Khattab, Arnav Singhvi, Paridhi Maheshwari, Zhiyuan Zhang, Keshav Santhanam, Sri Vardhamanan, Saiful Haq, Ashutosh Sharma, Thomas T. Joshi, Hanna Moazam, Heather Miller, Matei Zaharia, 和 Christopher Potts. 2023. DSPy: 将声明式语言模型调用编译为自我改进的管道. *arXiv 预印本 arXiv:2310.03714* (2023).

+   Kojima 等人 [2022] Takeshi Kojima, Shixiang Shane Gu, Machel Reid, Yutaka Matsuo, 和 Yusuke Iwasawa. 2022. 大型语言模型是零-shot 推理者. *神经信息处理系统进展* 35 (2022), 22199–22213.

+   Lewis 等人 [2020] Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, 等人. 2020. 用于知识密集型 NLP 任务的检索增强生成. *神经信息处理系统进展* 33 (2020), 9459–9474.

+   Li 和 Hovy [2015] Jiwei Li 和 Eduard Hovy. 2015. NLP 引擎: NLP 的通用图灵机. *arXiv 预印本 arXiv:1503.00168* (2015).

+   Li 等人 [2024] Zelong Li, Wenyue Hua, Hao Wang, He Zhu, 和 Yongfeng Zhang. 2024. Formal-LLM: 将形式语言与自然语言结合用于可控的基于 LLM 的代理. *arXiv:2402.00798* (2024).

+   Liang 等人 [2023] Yaobo Liang, Chenfei Wu, Ting Song, Wenshan Wu, Yan Xia, Yu Liu, Yang Ou, Shuai Lu, Lei Ji, Shaoguang Mao 等人. 2023. Taskmatrix.ai: 通过连接基础模型和数百万个 API 来完成任务。*arXiv 预印本 arXiv:2303.16434* (2023)。

+   Liu 等人 [2023] Bo Liu, Yuqian Jiang, Xiaohan Zhang, Qiang Liu, Shiqi Zhang, Joydeep Biswas 和 Peter Stone. 2023. Llm+ p: 赋能大型语言模型以实现最佳规划能力。*arXiv 预印本 arXiv:2304.11477* (2023)。

+   Lyu 等人 [2023] Qing Lyu, Shreya Havaldar, Adam Stein, Li Zhang, Delip Rao, Eric Wong, Marianna Apidianaki 和 Chris Callison-Burch. 2023. 忠实的推理链思维。*arXiv 预印本 arXiv:2301.13379* (2023)。

+   Madaan 等人 [2024] Aman Madaan, Niket Tandon, Prakhar Gupta, Skyler Hallinan, Luyu Gao, Sarah Wiegreffe, Uri Alon, Nouha Dziri, Shrimai Prabhumoye, Yiming Yang 等人. 2024. Self-refine: 自反馈的迭代优化。*神经信息处理系统进展* 36 (2024)。

+   Mei 等人 [2024] Kai Mei, Zelong Li, Shuyuan Xu, Ruosong Ye, Yingqiang Ge 和 Yongfeng Zhang. 2024. AIOS: LLM 代理操作系统。*arXiv* (2024)。

+   Mihalcea 等人 [2006] Rada Mihalcea, Hugo Liu 和 Henry Lieberman. 2006. NLP（自然语言处理）用于 NLP（自然语言编程）。载于 *计算语言学与智能文本处理：第七届国际会议，CICLing 2006，墨西哥城，墨西哥，2006年2月19日至25日。第七卷会议录*。Springer，第319–330页。

+   Nijkamp 等人 [2022] Erik Nijkamp, Bo Pang, Hiroaki Hayashi, Lifu Tu, Huan Wang, Yingbo Zhou, Silvio Savarese 和 Caiming Xiong. 2022. Codegen: 一个用于代码的开放大型语言模型，支持多轮程序合成。*arXiv 预印本 arXiv:2203.13474* (2022)。

+   OpenAI [2023] Josh 等人 OpenAI. 2023. GPT-4 技术报告。arXiv:2303.08774 [cs.CL]

+   Ouyang 等人 [2022] Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray 等人. 2022. 训练语言模型遵循指令并通过人类反馈进行优化。*神经信息处理系统进展* 35 (2022), 27730–27744。

+   Paul 等人 [2023] Debjit Paul, Mete Ismayilzada, Maxime Peyrard, Beatriz Borges, Antoine Bosselut, Robert West 和 Boi Faltings. 2023. Refiner: 中间表示的推理反馈。*arXiv 预印本 arXiv:2304.01904* (2023)。

+   Poesia 等人 [2022] Gabriel Poesia, Oleksandr Polozov, Vu Le, Ashish Tiwari, Gustavo Soares, Christopher Meek 和 Sumit Gulwani. 2022. Synchromesh: 基于预训练语言模型的可靠代码生成。*arXiv 预印本 arXiv:2201.11227* (2022)。

+   Prather [1997] Ronald E Prather. 1997. 程序计算的正则表达式。*美国数学月刊* 104, 2 (1997), 120–130。

+   Qin et al. [2023] Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, 等. 2023. Toolllm：帮助大语言模型掌握16000+种真实世界API。*arXiv预印本 arXiv:2307.16789* (2023)。

+   Ross et al. [2023] Steven I Ross, Fernando Martinez, Stephanie Houde, Michael Muller, 和 Justin D Weisz. 2023. 程序员助手：与大语言模型的对话式互动用于软件开发。收录于 *第28届国际智能用户界面会议论文集*，491–514。

+   Shinn et al. [2023] Noah Shinn, Beck Labash, 和 Ashwin Gopinath. 2023. Reflexion：具有动态记忆和自我反思的自主代理。*arXiv预印本 arXiv:2303.11366* (2023)。

+   Singh et al. [2023] Ishika Singh, Valts Blukis, Arsalan Mousavian, Ankit Goyal, Danfei Xu, Jonathan Tremblay, Dieter Fox, Jesse Thomason, 和 Animesh Garg. 2023. Progprompt：使用大语言模型生成情境化的机器人任务计划。收录于 *2023年IEEE国际机器人与自动化会议（ICRA）*，IEEE，11523–11530。

+   Vadas and Curran [2005] David Vadas 和 James R Curran. 2005. 使用无限制自然语言进行编程。收录于 *澳大利亚语言技术研讨会论文集 2005*，191–199。

+   Wang et al. [2022] Xuezhi Wang, Jason Wei, Dale Schuurmans, Quoc Le, Ed Chi, Sharan Narang, Aakanksha Chowdhery, 和 Denny Zhou. 2022. 自一致性改善语言模型中的连锁思维推理。*arXiv预印本 arXiv:2203.11171* (2022)。

+   Wang et al. [2023] Yubo Wang, Xueguang Ma, 和 Wenhu Chen. 2023. 使用医学教科书增强黑箱LLM用于临床问题解答。*arXiv预印本 arXiv:2309.02233* (2023)。

+   Wei et al. [2022] Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, 等. 2022. 连锁思维提示引发大语言模型中的推理能力。*神经信息处理系统进展* 35 (2022), 24824–24837。

+   Wu et al. [2020] Bichen Wu, Chenfeng Xu, Xiaoliang Dai, Alvin Wan, Peizhao Zhang, Zhicheng Yan, Masayoshi Tomizuka, Joseph Gonzalez, Kurt Keutzer, 和 Peter Vajda. 2020. 视觉变换器：基于标记的图像表示与处理用于计算机视觉。arXiv:2006.03677 [cs.CV]

+   Wu et al. [2024] Yiran Wu, Tianwei Yue, Shaokun Zhang, Chi Wang, 和 Qingyun Wu. 2024. StateFlow：通过状态驱动的工作流程增强LLM任务求解。*arXiv预印本 arXiv:2403.11322* (2024)。

+   Yao et al. [2024] Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Tom Griffiths, Yuan Cao, 和 Karthik Narasimhan. 2024. 思维树：通过大语言模型进行深思熟虑的问题解决。*神经信息处理系统进展* 36 (2024)。

+   Yao et al. [2022] Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, 和 Yuan Cao. 2022. React：语言模型中推理与行动的协同。*arXiv预印本 arXiv:2210.03629* (2022)。

+   姚等人 [2023] 佑宇·姚（Shunyu Yao）、杰弗里·赵（Jeffrey Zhao）、典·于（Dian Yu）、南·杜（Nan Du）、伊扎克·沙夫兰（Izhak Shafran）、卡尔蒂克·纳拉西曼（Karthik Narasimhan）和袁·曹（Yuan Cao）。2023年。《ReAct: 在语言模型中协同推理与行动》。发表于*国际学习表征会议（ICLR）*。

+   张等人 [2020] 天毅·张（Tianyi Zhang）、瓦尔莎·基肖尔（Varsha Kishore）、费利克斯·吴（Felix Wu）、基里安·Q·温伯格（Kilian Q. Weinberger）和约阿夫·阿尔茨（Yoav Artzi）。2020年。《BERTScore: 使用BERT评估文本生成》。
