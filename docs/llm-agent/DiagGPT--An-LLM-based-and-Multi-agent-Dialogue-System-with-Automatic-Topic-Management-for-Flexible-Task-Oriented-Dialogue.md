<!--yml

类别：未分类

日期：2025-01-11 13:07:52

-->

# DiagGPT：一种基于大语言模型（LLM）和多智能体的对话系统，具有自动主题管理功能，用于灵活的任务导向对话

> 来源：[https://arxiv.org/html/2308.08043/](https://arxiv.org/html/2308.08043/)

Lang Cao

伊利诺伊大学厄本那-香槟分校

计算机科学系

langcao2@illinois.edu

###### 摘要

大语言模型（LLM）的一个重要应用是作为聊天代理，回应人类在各种领域的询问。尽管当前的LLM在回答一般性问题方面表现得相当熟练，但在复杂的诊断场景中（如法律、医学或其他专业咨询）通常表现不佳。这些场景通常需要任务导向对话（TOD），在这种对话中，AI聊天代理不仅需要响应，还必须主动提问并引导用户完成特定目标或任务。以往的微调模型在任务导向对话（TOD）中表现不佳，当前LLM的对话能力尚未得到充分发挥。在本文中，我们介绍了DiagGPT（诊断中的对话 GPT），一种将LLM扩展到更多任务导向对话场景的创新方法。除了引导用户完成任务外，DiagGPT还能有效管理对话过程中所有主题的状态。这一特性提升了用户体验，并为任务导向对话提供了更灵活的互动方式。我们的实验表明，DiagGPT在与用户进行任务导向对话时表现出色，展现了其在多个领域的实际应用潜力。

DiagGPT：一种基于大语言模型（LLM）和多智能体的对话系统，具有自动主题管理功能，用于灵活的任务导向对话

Lang Cao 伊利诺伊大学厄本那-香槟分校 计算机科学系 langcao2@illinois.edu

## 1 引言

图1：ChatGPT与DiagGPT的主要区别。ChatGPT直接回答用户问题，而DiagGPT不仅提供同等质量的回答，还具有主动提问、引导用户并维护内部对话状态的能力。

大型语言模型（LLMs），例如ChatGPT，已经在各种自然语言处理（NLP）任务中展示了卓越的性能 Brown et al. ([2020](https://arxiv.org/html/2308.08043v4#bib.bib3)); Chowdhery et al. ([2022](https://arxiv.org/html/2308.08043v4#bib.bib4)); Wei et al. ([2022a](https://arxiv.org/html/2308.08043v4#bib.bib14)); OpenAI ([2023](https://arxiv.org/html/2308.08043v4#bib.bib11))。在许多情况下，OpenAI的GPT-4甚至超越了人类的表现 OpenAI ([2023](https://arxiv.org/html/2308.08043v4#bib.bib11))。通过使用提示工程技术（例如，思维链提示 Brown et al. ([2020](https://arxiv.org/html/2308.08043v4#bib.bib3)); Wei et al. ([2022b](https://arxiv.org/html/2308.08043v4#bib.bib15))，情境学习 Brown et al. ([2020](https://arxiv.org/html/2308.08043v4#bib.bib3)); Xie et al. ([2022](https://arxiv.org/html/2308.08043v4#bib.bib18)); Min et al. ([2022](https://arxiv.org/html/2308.08043v4#bib.bib9)) 等)，我们可以提升LLMs的推理和理解能力，从而完成日常生活中的复杂任务。LLMs已经引起了学术界和工业界的巨大关注，激励更多人基于这些模型构建有用的应用程序。

LLMs的一个流行应用是聊天机器人，它们围绕这些模型构建对话系统。ChatGPT¹¹1https://openai.com/blog/chatgpt 是这一类应用的成功范例，LLMs能够分析上下文，并根据从大量训练数据中提取的知识回答用户查询。通过补充背景知识、提供上下文和适当的提示，ChatGPT已经能够为专业领域形成强大的问答模型。它能够理解用户的问题并有效地提供精确的答案。

然而，我们日常生活中的对话场景可能更为复杂。例如，在法律或医疗诊断等专业咨询场景中，聊天智能体需要考虑用户的独特情况或信息。在获取用户信息的过程中，智能体提供的互动体验也至关重要。系统需要主动提出问题。因此，我们需要一个能更好模拟真实医学专家和法律专业人士的咨询过程。聊天智能体应进行问答、话题管理，并引导用户向特定目标或任务完成推进。这种对话类型被称为任务导向对话（TOD）。对话中通常会有一些预定义的目标。TOD帮助用户实现特定目标，关注理解用户、跟踪状态和生成下一步行动 Balaraman等人（[2021](https://arxiv.org/html/2308.08043v4#bib.bib1)）。它与轻量级对话或开放领域对话场景有本质的不同。尽管在这个领域有很多研究，但由于数据不足、效率低下和微调小模型的缺陷（例如无法完全理解用户意图和生成性能差）等问题，这一领域依然面临挑战。现有研究中的模型缺乏鲁棒性和普适性。例如，微调模型需要大量数据进行训练，而且难以迁移到其他场景中。另一方面，尽管LLM具有广泛的知识，且其回答的质量远超微调模型，但传统LLM已不能满足TOD的需求，无法有效管理复杂的对话逻辑。因为它们仅保持简单的记忆库，且只能处理线性的互动。

最近的进展主要集中在将大型语言模型（LLMs）作为智能体，用于构建多智能体系统，或是教会LLMs如何使用工具来完成更复杂的任务 Schick等人（[2023](https://arxiv.org/html/2308.08043v4#bib.bib12)）；Shen等人（[2023](https://arxiv.org/html/2308.08043v4#bib.bib13)）。这些系统通常具有一个核心智能体，负责控制整个任务流程。一个突出例子是AutoGPT²²2https://github.com/Significant-Gravitas/Auto-GPT，它采用多个GPT模型来策划每个智能体的职责，以便拆分复杂任务并完成它们。在这样的多智能体系统中，关键在于任务的划分和智能体之间的互动。通过组织多个LLM并指示它们协作，我们有机会解决许多单一LLM无法胜任的复杂任务。

微调模型在场景迁移能力和大量训练数据需求方面存在不足，而单一的大型语言模型则不擅长对话状态跟踪和管理。然而，我们可以利用LLM强大的知识背景，并采用多代理框架，将微调模型中的对话状态跟踪和管理思想融入其中。因此，我们可以构建一个多代理对话系统，满足灵活任务导向对话的需求。

受这些考虑的启发，本文提出了DiagGPT。DiagGPT代表基于GPT-4的诊断对话模型。这是一个多代理AI系统，具备自动话题管理能力，可以以更灵活的方式增强任务导向对话的实用性。与传统任务导向对话中机械、僵化地引导用户完成任务不同，我们可以在对话的展开中增强交互体验，使对话更加流畅。这更符合实际情况和真实对话。总之，我们的AI系统DiagGPT具备以下特点：

+   •

    问答是基于传统LLM的对话系统的基本功能。LLM拥有广泛的知识，可以为各种问题提供高质量的答案。我们在此基础上构建并保留了LLM的这一核心能力。

+   •

    任务引导系统旨在引导用户朝着特定目标前进，并在对话过程中帮助他们完成任务。这是通过在对话过程中推进一系列预定义的主题来实现的。

+   •

    主动提问该系统能够根据预定义的清单主动提出问题，从而收集用户的必要信息。

+   •

    话题管理该系统能够自动管理对话中的话题，跟踪话题进展，并有效地围绕当前话题展开讨论。在复杂对话中，它能够很好地管理各种话题变化。

+   •

    多功能性我们的系统直接基于LLM。它能够在各种场景下表现出色，无需任何训练数据，这是以前的微调模型所缺乏的能力。通过定义特定的预定义目标并补充支持功能，该系统可以轻松应用于多个案例。

鉴于这些特点，DiagGPT能够满足上述需求，并在复杂场景中更好地与用户进行专业咨询对话。我们的主要贡献是让传统的基于LLM的对话系统在灵活的任务导向对话中更智能。我们建立在LLM强大的知识基础之上，赋予其更多的交互能力。因此，DiagGPT可以像一个更智能、更专业的聊天机器人一样工作。

## 2 相关工作

任务导向对话系统帮助用户实现特定目标，重点是理解用户、跟踪状态以及生成后续动作。任务导向对话（TOD）是一种旨在完成特定目标的任务。近期的研究主要集中在小型模型的微调上。Wen等人（[2017](https://arxiv.org/html/2308.08043v4#bib.bib16)）介绍了一种基于神经网络的文本输入、文本输出的端到端可训练任务导向对话系统，并提出了一种基于新型管道化Wizard-of-Oz框架收集对话数据的方法。Wu等人（[2019](https://arxiv.org/html/2308.08043v4#bib.bib17)）提出了一种可转移的对话状态生成器（TRADE），该生成器通过复制机制从发言中生成对话状态，便于在预测（领域、槽位、值）三元组时进行转移，即使这些三元组在训练中没有出现。Feng等人（[2023](https://arxiv.org/html/2308.08043v4#bib.bib5)）提出了SG-USM，这是一种新型的基于模式的用户满意度建模框架。它显式地建模了系统在多大程度上满足用户对任务属性的偏好，从而预测用户的满意度水平。Liu等人（[2023](https://arxiv.org/html/2308.08043v4#bib.bib8)）提出了一种名为MUST的框架，通过利用多用户模拟器来优化任务导向对话系统。Bang等人（[2023](https://arxiv.org/html/2308.08043v4#bib.bib2)）提出了一种端到端的任务导向对话系统，采用任务优化适配器，这些适配器在每个任务上独立学习，在预训练网络的固定层后仅增加少量参数。所有这些方法都需要大量的训练数据，且尚未达到适用于实际应用的理想性能水平。

随着大型语言模型（LLMs）强大能力的认可，基于LLMs的对话系统变得越来越流行。Hudeček和Dušek（[2023](https://arxiv.org/html/2308.08043v4#bib.bib6)）评估了LLMs的对话能力，并发现，在显式信念状态跟踪方面，LLMs的表现不如专门的任务特定模型。这表明简单的LLMs并没有能力实现任务导向的对话。Liang等人（[2023](https://arxiv.org/html/2308.08043v4#bib.bib7)）提出了一种互动对话可视化系统，名为C5，该系统包括全局视图、主题视图和上下文关联问答视图，以更好地保留上下文信息并提供全面的回应。从另一个角度来看，Zhang等人（[2023](https://arxiv.org/html/2308.08043v4#bib.bib20)）提出了“向专家提问”框架，在该框架中，模型可以在每个回合中访问并咨询专家。该框架利用LLMs来改进任务导向对话（TOD）中的小型模型的微调。目前关于提升LLMs对话能力的研究较少。根据我们所知，我们是首个成功地在多代理框架中使用现成的LLMs来构建任务导向对话系统的团队。

## 3 方法论

### 3.1 DiagGPT 框架

DiagGPT 是一个由多个模块组成的多代理协作系统：聊天代理（Chat Agent）、话题管理器（Topic Manager）、话题丰富器（Topic Enricher）和上下文管理器（Context Manager）。每个模块都是一个具有特定提示的 LLM，指导它们的功能和职责。在这些模块中，话题管理器最为重要，因为它跟踪对话状态并自动管理对话话题。

图 2：DiagGPT 的框架。DiagGPT 的工作流程由四个阶段组成：思考话题发展、维护话题栈、丰富话题、生成响应。

如图[2](https://arxiv.org/html/2308.08043v4#S3.F2 "图 2 ‣ 3.1 DiagGPT 框架 ‣ 3 方法论 ‣ DiagGPT：一个基于大型语言模型和多代理的对话系统，具备自动话题管理功能，支持灵活的任务导向对话")所示，DiagGPT 的工作流程由四个阶段组成：1）思考话题发展：话题管理器获取用户查询，然后分析和预测当前对话轮次中的话题发展；2）维护话题栈：根据话题管理器的行动指令维护整个对话的话题栈；3）丰富话题：根据对话上下文检索当前话题并对其进行丰富；4）生成响应：基于特定的引导提示，并结合丰富的话题和上下文，生成回应用户的响应。

此外，我们将一个话题定义为一轮对话的主要主题，它决定了交流的主要焦点。我们还将任务定义为在任务导向对话中需要完成的具体目标。通过完成对话中所有预定义的话题后，应该实现这个具体的任务。

### 3.2 思考话题发展

话题管理器（Topic Manager）作为 DiagGPT 的主要模块，负责根据用户的查询确定话题的发展。在每一轮对话中，系统需要在提供响应之前调整当前的对话话题。因此，用户的查询首先会被输入到话题管理器中。

话题管理器的输入包括当前用户查询、行动列表、话题栈的当前状态以及聊天历史。对于 AI 代理来说，基于这些信息分析和预测话题的发展是合乎逻辑的。特别重要的是存储在话题管理器中的行动列表。这个行动列表包含各种行动，它们作为话题管理器执行的工具。话题管理器了解每个行动的细节、如何规划和执行这些行动。每个行动都对应一个程序功能，用于执行特定的命令。在 Python 中，我们使用装饰器函数来实现这一点。每当话题管理器接收到用户查询时，它会分析所有可用的信息，并根据与每个行动相关的提示决定执行哪个行动。

借助LLM强大的理解和推理能力，这个AI代理能够准确理解用户的意图，并帮助有效地与用户进行沟通。

### 3.3 维护主题栈

在从主题管理器获得动作的输出后，系统将执行相应的命令来处理和控制主题变化，这涉及到维护主题栈。

主题栈是该系统中的一种数据结构，用于存储和跟踪对话状态。栈管理是一种对短期和长期上下文中对话状态管理的模拟。在我们的系统中，我们具体化了LLM的对话状态，并明确地存储它，使系统在长期上下文中表现得更好，避免了模糊性和遗忘。我们认为对话的进展包含多个阶段或状态，这些状态遵循先进先出（FIFO）顺序，可以有效地使用栈来建模。虽然我们将这种主题存储结构称为栈，但这个组件并不严格遵循FIFO规则。这是因为对话逻辑较为复杂，FIFO仅仅是主题发展的最常见形式。我们将在接下来的段落中对这个结构的一些操作做详细描述。

在诊断场景中，顾问通常会在脑海中存储一个检查清单。在许多常见的情况下，如果用户没有提出任何新的问题，对话的发展将遵循这个检查清单。在完成检查清单中的所有项目后，顾问可以向用户提供报告和综合分析，从而完成特定任务。来自预定义任务的动作负载主题旨在促进这一过程。当被该动作提示装饰的功能执行时，来自检查清单的主题列表将被加载到主题栈中。

图3：创建新主题的操作。

图4：结束当前主题的操作。

此外，还有其他常用的操作来操作话题堆栈。这些操作包括创建新话题、完成当前话题和停留在当前话题。创建新话题的动作，如图[3](https://arxiv.org/html/2308.08043v4#S3.F3 "Figure 3 ‣ 3.3 Maintaining Topic Stack ‣ 3 Methodology ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue")所示，当用户想开始一个新话题时，会将新话题添加到堆栈中。完成当前话题的动作，如图[4](https://arxiv.org/html/2308.08043v4#S3.F4 "Figure 4 ‣ 3.3 Maintaining Topic Stack ‣ 3 Methodology ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue")所示，当用户不再希望讨论某个话题或系统认为该话题已关闭时，会将堆栈中的顶部话题移除。停留在当前话题的动作表示系统认为仍然需要信息并需要继续讨论当前话题，因此话题堆栈保持不变。这三种基本操作覆盖了大多数话题变更场景。由于我们仅允许LLM进行一步推理，因此它们必须选择并执行单一的操作。其他操作是基于这三种基本操作的复杂变更。

跳转到现有话题的动作，如图[5](https://arxiv.org/html/2308.08043v4#S3.F5 "Figure 5 ‣ 3.3 Maintaining Topic Stack ‣ 3 Methodology ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue")所示，允许用户从话题堆栈中检索并优先处理先前的话题。

我们应用了一个动作列表，使得该系统能够以更灵活的方式与用户互动。它可以避免传统任务导向对话中机械且僵化地引导用户完成任务。这个动作列表也可以扩展，以适应任务导向对话中更复杂的场景。我们还实现了一种机制，能够自动去除冗余的话题。在几轮对话之后，如果新生成的话题没有被回忆起来，它将被移除。然而，这种移除不会影响任何来自检查单的预定义话题。

图5：跳转到现有话题的动作。

|  | 人类评估 | LLM评估 |
| --- | --- | --- |
|  | RC $\downarrow$ | CR $\uparrow$ | SR $\uparrow$ | CS $\uparrow$ | CR $\uparrow$ | SR $\uparrow$ | RQ $\uparrow$ | CS $\uparrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ChatGPT (gpt-3.5-turbo) | 9.0 | 0.8 | 0.4 | - | 0.8 | 0.7 | 7.8 | - |
| ChatGPT (gpt-4) | 7.7 | 1.0 | 1.0 | 5.0 | 1.0 | 1.0 | 9.0 | 8.5 |
| DiagGPT (我们) | 7.0 | 1.0 | 1.0 | 15.0 | 1.0 | 1.0 | 9.0 | 11.5 |

表1：定量实验结果。我们将我们的方法DiagGPT与ChatGPT（gpt-turbo-3.5和gpt-4）在20个不同的对话场景中进行比较，评估指标包括回合数（RC）、完成率（CR）、回应质量（RQ）和比较得分（CS）。

### 3.4 主题丰富

我们选择主题堆栈中的顶部项作为当前主题。然而，它不能直接作为聊天主题供聊天代理与用户互动。主题堆栈中的主题简单且便于存储，而聊天代理实际使用的主题需要包含更多信息。如果没有主题丰富器，聊天代理很难直接理解当前对话轮次中的主要目标。

主题丰富器旨在弥合这一差距，帮助更好地组织语言以供使用。我们最初将主题分为“提问用户”和“回答用户”。通常，新生成的主题属于“回答用户”，而预定义的主题则归类为“提问用户”。这种设计有助于系统确定在当前对话轮次中是回答用户的问题还是提出问题。主题丰富器接收上下文管理器的输出和当前主题，将其丰富为包含更多上下文信息的主题。然后，将这个丰富的主题提供给聊天代理。

### 3.5 生成回应

在最终主题的帮助下，聊天代理将其识别为当前对话轮次中的主要主题。因此，通过上下文管理器的上下文，它可以最终生成用户的回应。此外，如图[10](https://arxiv.org/html/2308.08043v4#A4.F10 "Figure 10 ‣ Appendix D Prompt Design ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue")所示，一些检索到的背景知识、指令和鼓励语也会被添加到提示中，以进一步提高回应质量。

## 4 实验

### 4.1 设置

我们进行定量实验和定性实验，以展示DiagGPT的性能。系统实现的详细信息可以在附录[B](https://arxiv.org/html/2308.08043v4#A2 "Appendix B Implementation Details of DiagGPT ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue")中找到。

DiagGPT的定量评估是具有挑战性的，因为很难量化系统在对话中管理各种话题的能力。因此，我们通过定量实验仅评估系统在话题推进和引导方面的基本能力。我们没有使用像DialoGLUE Yadav等人（[2019](https://arxiv.org/html/2308.08043v4#bib.bib19)）和Moghe等人（[2023](https://arxiv.org/html/2308.08043v4#bib.bib10)）这样的任务导向对话数据集来评估我们的系统。因为DiagGPT是一个开放系统，它不像微调模型那样需要训练。我们的系统输出完全取决于提示。除此之外，这些传统的TOD数据集专注于特定的子任务（意图预测、槽位填充等），这些任务被认为是TOD所必需的，而基于LLM的系统则以更端到端的方式绕过了这些特定任务。因此，使用传统方法评估我们的系统是具有挑战性的。

为了定量评估我们的系统，我们开发了一个新的数据集，称为LLM-TOD，它涵盖了任务导向对话场景中的20个不同主题（详细信息见附录[C](https://arxiv.org/html/2308.08043v4#A3 "附录 C LLM-TOD 数据集 ‣ DiagGPT: 一种基于LLM的多代理对话系统，具备自动话题管理功能，支持灵活的任务导向对话")）。每个TOD场景的数据包括一个待完成的事项清单，最终达成特定目标。该事项清单和目标作为输入，供LLM完成任务导向对话。此外，我们基于LLM开发了一个新的评估框架来评估TOD任务。还使用了一个基于LLM的用户模拟器与对话系统进行交互。结果通过人类评估和另一个LLM的结合来评估。在LLM评估过程中，输入包括基本的TOD信息和完整的聊天记录。

另一方面，定性实验更有效地展示了DiagGPT的能力和表现。这些实验表明，DiagGPT能够更灵活地进行任务导向对话。

### 4.2 定量实验结果

我们通过人类评估和LLM评估来评估性能。我们将我们的方法与两个ChatGPT基准进行了比较：gpt-3.5-turbo和gpt-4。评估涉及以下指标：

+   •

    回合数（RC）：完成任务导向对话所需的平均回合数。该指标可视为系统效率的度量，较低的值更为理想。

+   •

    完成率（CR）：完成的事项清单项占总项数的百分比。理想的系统应该完成清单上的所有项目。

+   •

    成功率（SR）：衡量系统是否成功完成任务，达成预定目标的指标。该指标对于每个任务导向对话来说是0或1的值。

+   •

    响应质量（RQ）：通过使用LLM评估整个对话历史和系统响应的质量的得分。得分范围从1到10，由LLM生成。

+   •

    比较得分（CS）：在单一对话中比较ChatGPT（gpt-4）和DiagGPT的响应得分。它是基于在各种场景中的获胜次数计算得出的，表示哪个模型的表现更好。

具体而言，我们计算了20种不同任务导向对话（TOD）场景的平均得分。对于LLM评估的比较得分，我们交替变换两种系统在提示中的对话历史位置，并进行两次评估以获取平均得分。这种方法有助于避免自动评估中出现的位置偏差。对于响应质量的评估，我们考虑了理解、相关性、体验和充分性等因素。

从表[1](https://arxiv.org/html/2308.08043v4#S3.T1 "表1 ‣ 3.3 维持话题栈 ‣ 3 方法论 ‣ DiagGPT：一个基于LLM的多代理对话系统，具备自动话题管理的灵活任务导向对话")中，我们可以观察到定量实验的结果。该表显示DiagGPT在响应质量方面能够与最先进的LLM，gpt-4匹敌，并有效地管理任务导向对话。我们的系统在清单中的所有任务都已完成，并且成功实现了所有TOD场景中的目标，相较于ChatGPT，所需的回合数更少。这表明DiagGPT在引导用户实现目标并有效完成任务方面，展现出更高的效率，这在任务导向对话中至关重要。鉴于响应质量相当，我们直接比较了两种系统的响应。人类和LLM评估结果都表明，我们的系统更有效，表现更好。

我们相对于gpt-4的优势在于，DiagGPT通过其自动话题管理能够更灵活地与用户互动。这一特点在随后的定性实验中更加明显。

图6：医学咨询场景中的对话示例。系统DiagGPT充当真正的医生，主动询问信息并回答用户的问题。

图7：医学咨询场景中的继续对话示例。

图8：用户询问COVID-19相关问题时医学咨询场景中的对话示例。

### 4.3 定性实验结果

我们在DiagGPT中修改了设置，使其适用于医疗对话场景，以进行定性实验。图[6](https://arxiv.org/html/2308.08043v4#S4.F6 "图 6 ‣ 4.2 定量实验结果 ‣ 4 实验 ‣ DiagGPT: 基于LLM和多代理的对话系统，具有自动话题管理功能，支持灵活的任务导向对话")和图[7](https://arxiv.org/html/2308.08043v4#S4.F7 "图 7 ‣ 4.2 定量实验结果 ‣ 4 实验 ‣ DiagGPT: 基于LLM和多代理的对话系统，具有自动话题管理功能，支持灵活的任务导向对话")展示了医疗咨询过程中的完整对话演示。这是一个医疗诊断任务，目的是帮助患者识别病因并提供建议。医疗咨询不是纯粹的开放式问答，因为患者通常缺乏广泛的医学知识，依赖医生指导他们提供个人信息。因此，用户可以体验到一个真实的医生，而不是一个呆板的医疗问答机器。用户扮演患者的角色，而系统则扮演医生的角色，最初收集信息并逐步向患者提供建议。对话的主要发展遵循以下清单：基本信息、主诉、症状持续时间、症状严重性，这些也是预定义的主题。

本次演示展示了DiagGPT的强大对话能力，它能够主动提问并引导用户完成任务的最终目标，从而实现任务导向的对话。它模拟了许多真实的咨询场景。而其他对话系统，如普通的ChatGPT，无法达到这种表现。它们通常只能回答用户的问题，并且在复杂的场景中完成特定目标具有挑战性，即使使用了详细的提示语。

### 4.4 自动话题管理案例研究

DiagGPT的核心能力是自动管理对话中的话题，这是话题管理器的功能和主要职责。图[6](https://arxiv.org/html/2308.08043v4#S4.F6 "Figure 6 ‣ 4.2 Quantitative Experiment Results ‣ 4 Experiments ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue")和图[7](https://arxiv.org/html/2308.08043v4#S4.F7 "Figure 7 ‣ 4.2 Quantitative Experiment Results ‣ 4 Experiments ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue")展示了主要的检查清单进展。检查清单中的话题被按顺序提取并讨论，展示了完成当前话题的动作。当对话进入症状严重程度时，我们观察到话题仍然停留在这个阶段，且对话会在多个回合内继续，这样系统可以有时间理解用户的病情。这是现实场景中常见的情况，用户未能为医生提供足够的信息。创建新话题的动作如图[8](https://arxiv.org/html/2308.08043v4#S4.F8 "Figure 8 ‣ 4.2 Quantitative Experiment Results ‣ 4 Experiments ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue")所示。在这里，用户主动向系统提供关于COVID-19的一些信息以检查症状。我们观察到，系统生成了一个关于COVID-19的新话题，并进行了讨论，而不是僵硬地跟随检查清单。这些结果都展示了话题管理器的有效性。这些案例研究充分展示了系统灵活的理解能力，表明其能够熟练地处理现实对话中可能出现的各种情境。

## 5 结论

在本文中，我们提出了DiagGPT，一种基于LLM和多代理的系统，旨在以灵活的方式完成任务导向的对话任务。我们系统的原理是利用大型语言模型强大的理解和推理能力，实现能够自动管理话题并跟踪对话状态的代理。这一特性使得我们的系统能够准确理解用户在不同情况下的意图，帮助他们更灵活地完成特定任务。该系统特别适用于灵活任务导向对话的现实咨询场景。定量和定性实验均表明，DiagGPT表现优异。

然而，DiagGPT代表了一个实验性系统，旨在展示大型语言模型（LLMs）在处理实际任务中的潜力。未来，我们的目标是探索如何更好地利用LLMs的强大能力，通过实现更多功能和完成更复杂的任务来更好地服务用户。

## 限制

幻觉。幻觉是LLMs响应中的一个主要问题。有些人认为，由于幻觉的存在，LLMs无法在医疗或法律等严肃场景中使用。本论文中关于医疗咨询的相关实验仅展示了DiagGPT的主题管理能力。它与医疗答案的幻觉或质量无关，而只是反映了DiagGPT所展示的咨询过程。此外，我们也承认LLMs存在一些有害的回应。一方面，本文并不主要关注缓解幻觉的问题；另一方面，其他研究在减少LLMs幻觉方面已经取得了显著进展。因此，这种对话系统在现实生活中的应用依然拥有巨大的潜力。

成本与效率。DiagGPT涉及多个LLMs。在与单个用户查询进行一次对话时，所有这些LLMs都需要在精细的提示下运行，这非常昂贵。与只涉及一个LLM与用户互动的简单对话系统相比，DiagGPT需要AI代理之间的内部互动，从而花费更多时间来提供用户反馈。此外，我们系统中的AI代理需要强大的理解和推理能力，因此需要一个稳健且大规模的AI来维持这些能力，这也增加了整体系统成本。然而，我们相信，随着AI基础设施的未来发展，这些问题可以得到缓解。

稳定性。DiagGPT的性能不如一些基于规则或经过微调的对话模型稳定。主要问题出现在话题管理器决定对话发展方向时。如果AI没有强大的理解和推理能力，可能会导致系统不稳定。此外，对于每种不同的应用场景，需要对AI系统进行细致且详细的提示调整。鉴于LLMs输出的风险，还需要对回应进行一些后处理。然而，DiagGPT在对话系统中仍然具有无限的潜力。随着未来AI的进步，对话系统有望显著提升，变得更加类人化。

限制性实验。由于DiagGPT的固有特点，如前所述，我们无法进行广泛的定量实验来直接展示其性能。此外，从定量实验中获得的结果范围也有限。尽管我们仅在医疗咨询中提供了一个使用示例，但我们的系统在法律及其他多个场景中也能展示出类似的出色表现，这一点已通过专业专家验证。不同应用中的基本操作流程是相同的。而且，通过仔细观察DiagGPT生成的响应，我们可以辨别出每个模块都有效地履行了其职责，从而展现了其整体的强大性能。

## 参考文献

+   Balaraman等人（2021）Vevake Balaraman、Seyedmostafa Sheikhalishahi和Bernardo Magnini。2021年。[面向任务的对话系统中对话状态追踪的最新神经方法：一项调查](https://aclanthology.org/2021.sigdial-1.25)。收录于*第22届话语与对话专项兴趣小组年会论文集*，239-251页，新加坡及在线。计算语言学协会。

+   Bang等人（2023）Namo Bang、Jeehyun Lee和Myoung-Wan Koo。2023年。[面向任务的对话系统的任务优化适配器](https://doi.org/10.18653/v1/2023.findings-acl.464)。收录于*计算语言学协会的研究成果：ACL 2023*，7355-7369页，加拿大多伦多。计算语言学协会。

+   Brown等人（2020）Tom Brown、Benjamin Mann、Nick Ryder、Melanie Subbiah、Jared D Kaplan、Prafulla Dhariwal、Arvind Neelakantan、Pranav Shyam、Girish Sastry、Amanda Askell、Sandhini Agarwal、Ariel Herbert-Voss、Gretchen Krueger、Tom Henighan、Rewon Child、Aditya Ramesh、Daniel Ziegler、Jeffrey Wu、Clemens Winter、Chris Hesse、Mark Chen、Eric Sigler、Mateusz Litwin、Scott Gray、Benjamin Chess、Jack Clark、Christopher Berner、Sam McCandlish、Alec Radford、Ilya Sutskever和Dario Amodei。2020年。[语言模型是少样本学习者](https://proceedings.neurips.cc/paper_files/paper/2020/file/1457c0d6bfcb4967418bfb8ac142f64a-Paper.pdf)。收录于*神经信息处理系统进展*，第33卷，1877-1901页。Curran Associates, Inc.

+   Chowdhery 等人 (2022) Aakanksha Chowdhery, Sharan Narang, Jacob Devlin, Maarten Bosma, Gaurav Mishra, Adam Roberts, Paul Barham, Hyung Won Chung, Charles Sutton, Sebastian Gehrmann, Parker Schuh, Kensen Shi, Sasha Tsvyashchenko, Joshua Maynez, Abhishek Rao, Parker Barnes, Yi Tay, Noam Shazeer, Vinodkumar Prabhakaran, Emily Reif, Nan Du, Ben Hutchinson, Reiner Pope, James Bradbury, Jacob Austin, Michael Isard, Guy Gur-Ari, Pengcheng Yin, Toju Duke, Anselm Levskaya, Sanjay Ghemawat, Sunipa Dev, Henryk Michalewski, Xavier Garcia, Vedant Misra, Kevin Robinson, Liam Fedus, Denny Zhou, Daphne Ippolito, David Luan, Hyeontaek Lim, Barret Zoph, Alexander Spiridonov, Ryan Sepassi, David Dohan, Shivani Agrawal, Mark Omernick, Andrew M. Dai, Thanumalayan Sankaranarayana Pillai, Marie Pellat, Aitor Lewkowycz, Erica Moreira, Rewon Child, Oleksandr Polozov, Katherine Lee, Zongwei Zhou, Xuezhi Wang, Brennan Saeta, Mark Diaz, Orhan Firat, Michele Catasta, Jason Wei, Kathy Meier-Hellstern, Douglas Eck, Jeff Dean, Slav Petrov 和 Noah Fiedel. 2022. [Palm：通过路径扩展语言建模](http://arxiv.org/abs/2204.02311).

+   Feng 等人 (2023) Yue Feng, Yunlong Jiao, Animesh Prasad, Nikolaos Aletras, Emine Yilmaz 和 Gabriella Kazai. 2023. [基于模式的任务导向对话用户满意度建模](https://doi.org/10.18653/v1/2023.acl-long.116). 收录于 *第61届计算语言学协会年会（第一卷：长篇论文）*，第2079-2091页，加拿大多伦多。计算语言学协会。

+   Hudeček 和 Dušek (2023) Vojtěch Hudeček 和 Ondřej Dušek. 2023. [任务导向对话系统是否只需要大型语言模型？](http://arxiv.org/abs/2304.06556)

+   Liang 等人 (2023) Pan Liang, Danwei Ye, Zihao Zhu, Yunchao Wang, Wang Xia, Ronghua Liang 和 Guodao Sun. 2023. [C5：为了更好的对话理解和上下文连贯性，面向 ChatGPT](http://arxiv.org/abs/2308.05567).

+   Liu 等人 (2023) Yajiao Liu, Xin Jiang, Yichun Yin, Yasheng Wang, Fei Mi, Qun Liu, Xiang Wan 和 Benyou Wang. 2023. [一个不能代表所有！利用多种用户模拟器训练任务导向对话系统](https://doi.org/10.18653/v1/2023.acl-long.1). 收录于 *第61届计算语言学协会年会（第一卷：长篇论文）*，第1-21页，加拿大多伦多。计算语言学协会。

+   Min 等人 (2022) Sewon Min, Xinxi Lyu, Ari Holtzman, Mikel Artetxe, Mike Lewis, Hannaneh Hajishirzi 和 Luke Zettlemoyer. 2022. [重新思考示范的角色：是什么使得上下文学习有效？](https://doi.org/10.18653/v1/2022.emnlp-main.759) 收录于 *2022年自然语言处理实证方法会议论文集*，第11048-11064页，阿布扎比，阿联酋。计算语言学协会。

+   Moghe 等人（2023）Nikita Moghe、Evgeniia Razumovskaia、Liane Guillou、Ivan Vulić、Anna Korhonen 和 Alexandra Birch. 2023. [Multi3NLU++：一个多语言、多意图、多领域的数据集，用于任务导向对话中的自然语言理解](https://doi.org/10.18653/v1/2023.findings-acl.230)。在 *计算语言学协会会议论文集：ACL 2023*，第3732–3755页，加拿大多伦多。计算语言学协会。

+   OpenAI（2023）OpenAI. 2023. [GPT-4技术报告](http://arxiv.org/abs/2303.08774)。

+   Schick 等人（2023）Timo Schick、Jane Dwivedi-Yu、Roberto Dessì、Roberta Raileanu、Maria Lomeli、Luke Zettlemoyer、Nicola Cancedda 和 Thomas Scialom. 2023. [Toolformer：语言模型可以自我学习使用工具](http://arxiv.org/abs/2302.04761)。

+   Shen 等人（2023）Yongliang Shen、Kaitao Song、Xu Tan、Dongsheng Li、Weiming Lu 和 Yueting Zhuang. 2023. [HuggingGPT：与ChatGPT及其朋友一起解决AI任务](http://arxiv.org/abs/2303.17580)。

+   Wei 等人（2022a）Jason Wei、Yi Tay、Rishi Bommasani、Colin Raffel、Barret Zoph、Sebastian Borgeaud、Dani Yogatama、Maarten Bosma、Denny Zhou、Donald Metzler、Ed H. Chi、Tatsunori Hashimoto、Oriol Vinyals、Percy Liang、Jeff Dean 和 William Fedus. 2022a. [大型语言模型的突现能力](http://arxiv.org/abs/2206.07682)。

+   Wei 等人（2022b）Jason Wei、Xuezhi Wang、Dale Schuurmans、Maarten Bosma、Fei Xia、Ed Chi、Quoc V Le、Denny Zhou 等人. 2022b. 连锁思维提示引发大型语言模型的推理。*神经信息处理系统进展*，35:24824–24837。

+   Wen 等人（2017）Tsung-Hsien Wen、David Vandyke、Nikola Mrkšić、Milica Gašić、Lina M. Rojas-Barahona、Pei-Hao Su、Stefan Ultes 和 Steve Young. 2017. [基于网络的端到端可训练任务导向对话系统](https://aclanthology.org/E17-1042)。在 *欧洲计算语言学协会第15届会议论文集：第1卷，长篇论文*，第438–449页，西班牙瓦伦西亚。计算语言学协会。

+   Wu 等人（2019）Chien-Sheng Wu、Andrea Madotto、Ehsan Hosseini-Asl、Caiming Xiong、Richard Socher 和 Pascale Fung. 2019. [可迁移的多领域状态生成器，用于任务导向对话系统](https://doi.org/10.18653/v1/P19-1078)。在 *计算语言学协会第57届年会论文集*，第808–819页，意大利佛罗伦萨。计算语言学协会。

+   Xie 等人（2022）Sang Michael Xie、Aditi Raghunathan、Percy Liang 和 Tengyu Ma. 2022. [将上下文学习解释为隐式贝叶斯推理](https://openreview.net/forum?id=RdJVFCHjUMI)。在 *国际学习表示大会*。

+   Yadav 等人（2019）Deshraj Yadav、Rishabh Jain、Harsh Agrawal、Prithvijit Chattopadhyay、Taranjeet Singh、Akash Jain、Shiv Baran Singh、Stefan Lee 和 Dhruv Batra. 2019. Evalai：为AI代理开发更好的评估系统。

+   Zhang等人（2023）Qiang Zhang, Jason Naradowsky, 和Yusuke Miyao. 2023. [请教专家：利用语言模型提升面向目标对话模型中的战略推理](http://arxiv.org/abs/2305.17878)。

## 附录A DiagGPT的可扩展性分析

在本文中，我们仅介绍了DiagGPT系统的基本框架，该系统旨在实现面向任务的对话。我们设计了这个系统，具有足够的灵活性，以便在复杂场景中处理任务并满足更多对话系统的需求。此外，随着基础模型的升级，我们的系统性能也将得到改善。具体来说，我们只提取了DiagGPT中最重要的模块，形成了一个基本的系统框架。这四个模块已经能够实现面向任务的对话的基本功能。在多智能体系统中，具有很大的可扩展性。例如，信息收集器可以监控用户输入，并将信息组织为结构化数据，以便将来更好地利用。

有时候，一场对话的目的是在对话过程中或之后解决某些任务。当对话实现预定目标时，系统可以调用更复杂的程序以满足需求。某些工具API（应用程序接口）调用也可以被添加到行动列表中执行，这意味着行动列表是DiagGPT的接口，可以提供许多插件来丰富该系统的功能。这也意味着它可以通过触发某些任务的API来将任务导向的对话扩展为任务导向的对话。

## 附录B DiagGPT的实现细节

在DiagGPT的实现中，我们采用了gpt-4作为基础LLM，利用其强大的理解和推理能力来获得理想的结果。我们将AI系统中所有LLM的解码温度设置为0，以确保更稳定的任务执行。我们在系统中提供了详细的提示。代理的主要提示如图[10](https://arxiv.org/html/2308.08043v4#A4.F10 "Figure 10 ‣ Appendix D Prompt Design ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue")所示，而图[11](https://arxiv.org/html/2308.08043v4#A4.F11 "Figure 11 ‣ Appendix D Prompt Design ‣ DiagGPT: An LLM-based and Multi-agent Dialogue System with Automatic Topic Management for Flexible Task-Oriented Dialogue")显示了行动列表中所有操作的提示。在这些提示中，蓝色的{variable}表示该槽位需要填写相应的变量文本。一旦填写完毕，这些提示可以被输入到LLM中生成响应。这些槽位中的一些有助于系统中各代理之间的信息交互。

在评估中，我们使用 gpt-4-0613 作为 DiagGPT 的基础模型，同时也用于 ChatGPT（gpt-4）。对于 ChatGPT（gpt-3.5-turbo），我们选择了 gpt-3.5-turbo-0613。用户的模拟也基于 gpt-4-0613。对于 LLM 评估，我们使用 gpt-4-turbo-2024-04-09。为了简化实验设置并便于进行比较，我们在定量实验中使用了简化版的 DiagGPT。我们减少了动作列表的复杂性并优化了 LLM 的提示，以尽可能促进主题的发展。然而，在定性实验中，我们评估了完整版本的 DiagGPT，展示了我们系统的全部功能。

## 附录 C LLM-TOD 数据集

图 9：来自 TOD-LLM 数据集的一个示例数据。

我们构建了一个新的数据集，LLM-TOD（面向大语言模型的任务导向对话数据集）。它用于定量评估基于 LLM 的任务导向对话模型的性能。由于 DiagGPT 依赖于 LLM，并且考虑到我们的资源有限，针对 TOD 这一特定目的对 LLM 进行微调是不可行的。因此，我们无法利用其他专门为微调模型而定制的数据集。

该数据集包含 20 条数据，每条数据代表一个不同的主题：临床、餐厅、酒店、医院、火车、警察、公交、景点、机场、酒吧、图书馆、博物馆、公园、健身房、电影院、办公室、理发店、面包店、动物园和银行。每条数据条目由 LLM 生成，随后由人工验证。对于数据集中的每个任务导向对话，系统需要按顺序通过检查清单，并与用户互动以实现指定目标。一个检查清单包括六项任务或主题，需解决或讨论。我们将这些变量整合到 LLM 的提示中，从而使其能够执行 TOD。图[9](https://arxiv.org/html/2308.08043v4#A3.F9 "图 9 ‣ 附录 C LLM-TOD 数据集 ‣ DiagGPT: 一种基于 LLM 的多智能体对话系统，具有自动主题管理功能，支持灵活的任务导向对话")展示了 TOD-LLM 数据集中的一个示例。

## 附录 D 提示设计

一些LLM在DiagGPT中的提示内容如图[10](https://arxiv.org/html/2308.08043v4#A4.F10 "图 10 ‣ 附录 D 提示设计 ‣ DiagGPT：一种基于LLM的多代理对话系统，具有自动主题管理功能，用于灵活的任务导向对话")和图[11](https://arxiv.org/html/2308.08043v4#A4.F11 "图 11 ‣ 附录 D 提示设计 ‣ DiagGPT：一种基于LLM的多代理对话系统，具有自动主题管理功能，用于灵活的任务导向对话")所示。图[12](https://arxiv.org/html/2308.08043v4#A4.F12 "图 12 ‣ 附录 D 提示设计 ‣ DiagGPT：一种基于LLM的多代理对话系统，具有自动主题管理功能，用于灵活的任务导向对话")还展示了我们实验中使用的其他LLM模型的提示：ChatGPT基线、UserGPT（用户模拟器）和EvalGPT（用于LLM评估）。评分模式涉及对回复质量打分，而比较模式则需要根据回复质量选择胜者。评估回复质量的标准包括从人类和LLM的角度考虑：

+   •

    理解 系统在多大程度上理解用户请求？

+   •

    相关性 回复是否直接与用户的需求相关？

+   •

    复杂处理 系统能否有效处理多方面的问题？

+   •

    效率 系统在多快的时间内将对话引导到一个解决方案？

+   •

    体验 交互的整体简便性和满意度如何？

+   •

    综合性 系统的回复是否提供了所有相关信息，充分涵盖用户的问题？

+   •

    细节 回复是否包含足够的细节和深度，以全面解决用户询问的复杂性和细微之处？

+   •

    充分性 是否探讨并解释了询问的所有方面和潜在含义，确保用户对主题有全面的理解？

图 10：Chat Agent、Topic Manager 和 Topic Enricher 的提示。这些指令可以根据其他场景进行修改。

图 11：不同操作的提示用于定义特定的程序功能，这些功能对应其各自的操作，并指示何时执行它们。

图 12：我们实验中其他LLM的提示。
