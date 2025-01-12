<!--yml

分类：未分类

日期：2025-01-11 13:05:13

-->

# 对话健康代理：个性化的LLM驱动代理框架

> 来源：[https://arxiv.org/html/2310.02374/](https://arxiv.org/html/2310.02374/)

Mahyar Abbasian, M.Sc.^(1*), Iman Azimi, Ph.D.¹,

Amir M. Rahmani, Ph.D.¹, 和 Ramesh Jain, Ph.D.¹

¹加利福尼亚大学欧文分校

^*通讯作者，abbasiam@uci.edu

## 摘要

对话健康代理（CHAs）是提供医疗服务的互动系统，如协助和诊断。目前的CHAs，特别是那些利用大型语言模型（LLMs）的系统，主要侧重于对话方面。然而，它们的代理能力有限，具体来说，缺乏多步问题解决、个性化对话和多模态数据分析。我们的目标是克服这些局限性。我们提出了openCHA，这是一个开源的LLM驱动框架，旨在使对话代理能够为用户的医疗查询生成个性化响应。该框架使开发者能够将外部数据源、知识库和分析模型等整合到基于LLM的解决方案中。openCHA包括一个协调器，用于规划和执行从外部源收集信息的行动，这对制定用户询问的响应至关重要。它促进了知识获取、问题解决能力、多语言和多模态对话，并促进了与各种AI平台的互动。我们通过两个演示和四个使用案例展示了该框架在处理复杂医疗任务中的能力。此外，我们将openCHA作为开源项目发布，供社区通过GitHub^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://github.com/Institute4FutureHealth/CHA](https://github.com/Institute4FutureHealth/CHA)^,^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://opencha.com](https://opencha.com)使用。

## 0em介绍

人工智能（AI），特别是基于大型语言模型（LLM）的对话系统，近年来引起了全球广泛关注。这些系统通过提供前所未有的访问和与大量文本信息的互动，彻底改变了这一领域。LLM可以聚合和处理互联网上存在的全面或聚焦的文本知识片段，为需要它的任何人提供上下文相关、目标导向和互动的知识访问。LLM的出现已经改变了早期简单的对话系统，如Alexa和Siri，展示了它们在多个领域的显著有效性[[1](https://arxiv.org/html/2310.02374v5#bib.bib1), [2](https://arxiv.org/html/2310.02374v5#bib.bib2), [3](https://arxiv.org/html/2310.02374v5#bib.bib3)]。对话系统现在能够进行开放式对话，并以更自然和更具吸引力的方式提供相关的上下文信息。

虽然人工智能领域早已探索智能代理，但其关注点主要集中在分析环境并根据收集到的信息做出决策。早期的人工智能研究通常专注于物理世界的问题，得益于计算机视觉、音频处理以及其他多模态感知理解领域的进展。然而，在像健康管理这样的动态环境中，个性化且不断变化的人类健康状态至关重要，智能代理需要通过各种手段，诸如对话互动和访问个人用户数据，准确捕捉这些状态。这些信息需要被收集并分析，借助通过研究和实践经验积累的大量知识。

会话健康代理（CHAs）具有巨大的潜力，能够应对动态健康管理环境中的挑战。得益于大语言模型（LLMs）的出现，CHAs现在能够通过多模态对话理解用户互动，涵盖文本、语音以及可能的其他模态。通过分析这些互动，CHAs需要识别理解用户不断变化的健康状态所需的数据、信息、计算过程和知识来源。然后，这些信息将转化为可操作的见解，有效地指导健康管理。实际上，CHAs应该将基于LLM的转换能力与代理的功能相结合，利用外部数据和信息来源，驾驭个性化健康环境的复杂性，为用户提供定制化支持：

+   •

    对话是人类互动的基本方式。历来，对话一直是知识的主要来源，也是社会行动的催化剂。最近，许多研究证实了与会话健康代理（CHAs）相关的有效性、可用性和整体满意度[[4](https://arxiv.org/html/2310.02374v5#bib.bib4)]。在医疗保健环境中，*同理心* [[5](https://arxiv.org/html/2310.02374v5#bib.bib5), [6](https://arxiv.org/html/2310.02374v5#bib.bib6)] 和*陪伴感* [[7](https://arxiv.org/html/2310.02374v5#bib.bib7), [8](https://arxiv.org/html/2310.02374v5#bib.bib8)] 需要个性化的对话。

+   •

    代理应配备对话工具、界面、计算能力和访问外部资源的能力，以提高医疗服务的质量[[9](https://arxiv.org/html/2310.02374v5#bib.bib9)]。考虑到医疗领域的复杂性[[10](https://arxiv.org/html/2310.02374v5#bib.bib10), [11](https://arxiv.org/html/2310.02374v5#bib.bib11)]，代理应理解用户的查询，将其分解为知识的组成部分，进行健康数据访问与分析，并应用推理来有效响应特定情况[[11](https://arxiv.org/html/2310.02374v5#bib.bib11)]。它们的适应性至关重要，应与医疗技术和文献的进步相协调[[10](https://arxiv.org/html/2310.02374v5#bib.bib10)]。因此，代理赋能个性化、多模态和持续更新的能力[[6](https://arxiv.org/html/2310.02374v5#bib.bib6)]。

我们当前的探索集中在使用人工智能、LLMs（大规模语言模型）和移动健康（mHealth）等最新技术进展来开发CHA（对话健康代理），这些技术在持续收集用户的生活方式和生理数据方面显示了有效性。图[1](https://arxiv.org/html/2310.02374v5#S1.F1 "Figure 1 ‣ 0emIntroduction ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")展示了CHA主要组件的概览。这些不可或缺的组件将有助于创建极其高效的CHA。

![参考标题](img/d1ccc2e304f16be2fbe8a7628c9570c9.png)

图1：一个对话健康代理，包括1）一个对话组件，允许用户交互；2）一个健康代理，用于解决问题并确定最佳行动顺序，利用健康信息。

现有的大型语言模型（LLMs），如ChatGPT [[1](https://arxiv.org/html/2310.02374v5#bib.bib1)]、BioGPT [[12](https://arxiv.org/html/2310.02374v5#bib.bib12)]、ChatDoctor [[13](https://arxiv.org/html/2310.02374v5#bib.bib13)]和Med-PaLM [[3](https://arxiv.org/html/2310.02374v5#bib.bib3)]，目前在医学知识领域处于活跃状态。这些LLMs可以作为聊天助手（CHAs）[[14](https://arxiv.org/html/2310.02374v5#bib.bib14)、[15](https://arxiv.org/html/2310.02374v5#bib.bib15)、[9](https://arxiv.org/html/2310.02374v5#bib.bib9)、[16](https://arxiv.org/html/2310.02374v5#bib.bib16)、[17](https://arxiv.org/html/2310.02374v5#bib.bib17)]使用。然而，它们仅关注对话方面，提供有限的代理能力，如基本的文本聊天界面，缺乏多步问题解决能力。它们无法访问用户个人收集的纵向数据和电子健康记录（EHRs），这些数据包含重要信息，如生命体征、生物信号（例如心电图）、医学影像和人口统计数据。因此，它们的回答往往比较泛泛，可能无法充分应对个体健康状况。此外，它们难以整合最新的健康信息，导致可能给出过时的回答[[18](https://arxiv.org/html/2310.02374v5#bib.bib18)]。更重要的是，这些聊天机器人并未与现有的已建立的AI模型和工具[[19](https://arxiv.org/html/2310.02374v5#bib.bib19)]无缝集成，用于多模态预测建模，致使以往的医疗保健工作变得过时。

鉴于科技的显著进步及其对人类和环境的重要性，我们必须整合所有可用工具，利用来自不同来源的知识，打造能够为全球观众提供可信、易懂和可操作环境的聊天助手（CHA）。目前，我们站在构建能够以最用户友好且符合文化差异的方式传递信息的框架的门槛上。本文旨在介绍这一类代理的初步版本，并为随着我们前行而发展的更为复杂的工具奠定基础。

## 0em背景与意义

### 相关工作

基于LLM的聊天助手（CHA）的开发工作可分为三大类：LLM聊天机器人、专门的健康LLM和多模态健康LLM。LLM聊天机器人使用并评估现有的聊天机器人（如ChatGPT），在执行不同的医疗功能时[[20](https://arxiv.org/html/2310.02374v5#bib.bib20)、[21](https://arxiv.org/html/2310.02374v5#bib.bib21)、[22](https://arxiv.org/html/2310.02374v5#bib.bib22)、[23](https://arxiv.org/html/2310.02374v5#bib.bib23)、[24](https://arxiv.org/html/2310.02374v5#bib.bib24)]进行评估。例如，陈等人[[23](https://arxiv.org/html/2310.02374v5#bib.bib23)]研究了ChatGPT在提供关于癌症治疗相关问题的可靠见解方面的效能。

专业化健康领域的LLM（大型语言模型）深入探讨了LLM的基本方面，旨在通过创建专门为医疗保健预训练的新LLM或微调现有模型，提升对话模型的表现。值得注意的例子包括ChatDoctor [[13](https://arxiv.org/html/2310.02374v5#bib.bib13)]、MedAlpaca [[2](https://arxiv.org/html/2310.02374v5#bib.bib2)]和BioGPT [[12](https://arxiv.org/html/2310.02374v5#bib.bib12)]等项目。此类别的出现是回应研究表明，通用领域的LLM往往在处理医疗特定任务时存在困难，因为领域迁移的问题 [[25](https://arxiv.org/html/2310.02374v5#bib.bib25), [26](https://arxiv.org/html/2310.02374v5#bib.bib26)]，且仅依赖提示工程可能无法显著提高其在医疗领域的表现 [[27](https://arxiv.org/html/2310.02374v5#bib.bib27), [28](https://arxiv.org/html/2310.02374v5#bib.bib28)]。

多模态健康LLM涉及通过将多模态整合到LLM中以进行诊断功能，开辟了一条新路径。例如，Tu等人 [[3](https://arxiv.org/html/2310.02374v5#bib.bib3)] 研究了LLM中基础转换器概念的潜力，将视频、图像、信号和文本等多种模态融合，最终形成了一个多模态生成模型。Xu等人 [[29](https://arxiv.org/html/2310.02374v5#bib.bib29)] 提出了一个与LLM对齐的多模态模型，将胸部X光图像与放射学报告结合，用于X光相关任务。类似地，Belyaeva等人 [[30](https://arxiv.org/html/2310.02374v5#bib.bib30)] 将表格化健康数据融入LLM，从而实现了多模态的医疗保健功能。

### 现有的研究空白和挑战

基于知识的连贯性和个性化的CHA（对话代理）需要量身定制的互动，这种互动超越了基础对话，通过多功能、多模态的多语言接口确保包容性。目标是创造能够在对话技能上表现出色的CHA，同时展现代理能力，使其能够进行批判性思维和战略规划，成为高效的问题解决者。尽管在开发CHA方面做出了巨大努力，现有的服务和模型仍然存在以下局限性：

1.  i)

    缺乏对全面个性化的支持，特别是在需要实时访问个性化数据的情况下。用户大量的医疗数据，主要包括图像、时间序列、表格数据以及所有其他用户测量的个人数据流，存储在医疗平台中。目前，CHAs对这些数据的访问有限，主要是在大语言模型（LLM）开发的训练和微调阶段，或者在此之后它们与用户数据完全断开。缺乏准确的用户健康信息——包括来自可穿戴设备、移动健康应用程序及类似来源的连续数据——限制了这些智能体的性能，使得它们只能提供通用的回答、提出一般性建议，或可能给出不准确的答案。

1.  ii)

    无法访问最新的知识并检索最新的医疗知识库。传统的LLM在训练阶段依赖有限的数据和来源于互联网的知识，这导致了三个主要问题。它们往往表现出偏向拥有最丰富在线内容的人群的偏见，这突显了访问最新、相关数据的重要性。最近推出的基于LLM的服务（例如，ChatGPT4[[1](https://arxiv.org/html/2310.02374v5#bib.bib1)]）提供了互联网搜索功能，但由于大量网站传播虚假信息，这仍然不足以满足医疗应用的需求。它们缺乏对新出现的可靠互联网资源的更新，这在医疗领域尤其是一个关键短板，因为新的、可靠的治疗方案和对先前建议的修改往往被忽视。最后，它们依赖过时或不太相关的数据，这使得识别幻觉现象[[31](https://arxiv.org/html/2310.02374v5#bib.bib31)]变得问题重重。缺乏最新信息降低了生成响应的可信度和可靠性[[18](https://arxiv.org/html/2310.02374v5#bib.bib18)]。

1.  iii)

    缺乏与已建立的多模态数据分析工具和预测模型的无缝集成，这些工具和模型需要外部执行。目前的智能体往往高估了生成式AI的计算能力，导致未充分利用已成熟的医疗分析工具，尽管它们在处理多样化数据类型方面表现出色[[19](https://arxiv.org/html/2310.02374v5#bib.bib19)]。

1.  iv)

    缺乏多步骤问题解决能力。现有的基于LLM的CHA通常专注于特定任务，或者缺乏强大的数据分析能力。例如，Xu等人[[29](https://arxiv.org/html/2310.02374v5#bib.bib29)]的模型仅依赖X光图像进行报告，忽略了其他模态信息，如以时间序列格式记录的生命体征。此外，现有的CHA无法处理复杂的顺序任务（即作为问题解决者）。将LLM集成到CHA中需要融合顺序推理、个性化健康历史分析和数据融合。

仅依赖LLM不足以解决前述挑战。为了使其在实际应用中有效，我们需要一个全面的框架，该框架在利用LLM的同时，整合各种辅助组件和外部资源。

### 关键贡献

在本文中，我们提出了一个全面的LLM驱动框架，用于CHA的开发，旨在纠正上述提到的限制。我们详细阐述了框架的组成部分，并通过一个案例研究展示了我们代理的能力。该框架是一个问题解决者，通过利用当代互联网资源和先进的多模态医疗分析工具（包括机器学习方法），提供个性化的响应。重要贡献如下：

1.  1.

    我们提出了一个LLM驱动的协调器，作为问题解决者，通过分析输入查询、收集所需信息、执行操作并提供个性化响应，来解决与医疗相关的查询。

1.  2.

    我们引入了外部医疗数据源、知识库以及对启用CHA提供可靠、值得信赖和最新响应至关重要的AI与分析模型。

1.  3.

    我们将多模态和多语言功能纳入框架，提升了可用性。

1.  4.

    我们通过两个演示和四个用例展示了该框架的有效性。

1.  5.

    我们将CHA框架作为开源项目发布在GitHub上^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$ [https://github.com/Institute4FutureHealth/CHA](https://github.com/Institute4FutureHealth/CHA)，并提供了详细的文档^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$ [https://docs.opencha.com/](https://docs.opencha.com/)，邀请社区利用并将其集成到他们的解决方案中。

## 0em材料与方法

![参见说明](img/6029ff8c04d716115d7eeb1dc3940a98.png)

图2：提出的LLM驱动框架概述，利用基于服务的架构

我们设计了一个基于大型语言模型（LLM）驱动的框架，其中的中央代理能够感知和分析用户查询，提供适当的回应，并通过应用程序接口（APIs）或函数调用管理对外部资源的访问。用户与框架的互动是双向的，确保了持续对话和后续对话的对话性。图[2](https://arxiv.org/html/2310.02374v5#S3.F2 "Figure 2 ‣ 0emMaterial and Methods ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")展示了框架的概览，包括三个主要组件：界面、调度器和外部资源。

### 界面

界面作为用户与代理之间的桥梁，包括通过移动设备、桌面或网页应用程序可访问的交互工具。它集成了多模态通信渠道，例如文本和音频。界面接收用户的查询，并将其传递给调度器（见图[2](https://arxiv.org/html/2310.02374v5#S3.F2 "Figure 2 ‣ 0emMaterial and Methods ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")）。

在该框架中，用户可以提供元数据（与其查询一起），包括图像、音频、手势等。例如，用户可以拍摄自己餐点的照片并询问其营养价值或卡路里含量，图像作为元数据提供。

### 调度器

调度器是openCHA代理的核心，负责解决问题、规划、执行行动，并根据用户查询提供适当的回应。它在openCHA中融合了感知循环模型的概念[[32](https://arxiv.org/html/2310.02374v5#bib.bib32)]，使其能够感知、转化并分析世界（即输入的查询和元数据），以生成适当的回应。为此，输入数据被汇总、转化为结构化数据，然后分析以规划和执行行动。通过这一过程，调度器与外部资源进行互动，获取所需信息，执行数据整合与分析，并提取洞察等功能。接下来，我们将概述调度器的五个主要组件。

任务规划器是调度器的LLM驱动决策、规划和推理核心。它的主要责任是收集所有必要的信息来回答用户的查询。为此，它解读用户的查询和元数据，确定执行任务所需的步骤。

为了将用户查询转化为任务序列，我们将“思维树”[[33](https://arxiv.org/html/2310.02374v5#bib.bib33)] 提示方法融入了任务规划器。使用这种提示方法，LLM 被要求 1) 生成三种独特的策略（即带有输入的任务调用序列），2) 描述每种策略的优缺点，3) 选择其中一个作为最佳策略。openCHA 中集成的另一种提示技术是 ReAct [[34](https://arxiv.org/html/2310.02374v5#bib.bib34)]，它采用推理和行动技术来确定需要执行的关键任务。openCHA 为用户提供了选择最适合其应用需求的提示方法的灵活性。其他提示技术，如计划与解决提示 [[35](https://arxiv.org/html/2310.02374v5#bib.bib35)]，也可以作为任务规划器实现并集成。

我们在附录 1 中概述了将任务创建和集成到 openCHA 中的过程。我们还说明了任务如何转化为适当的提示，从而使任务规划器能够识别可用的任务以及如何调用它们。附录 2 提供了任务规划器实现的详细分析，采用了“思维树”提示方法。

在提议的 Orchestrator 中，规划部分使用英语进行，利用 LLM 在该语言中的优越能力。如果查询使用非英语语言，框架可以采用两种不同的方法。第一种方法是重新训练源语言，并利用该语言的语言模型能力来生成响应。第二种方法是将查询翻译成英语（例如，使用 Google 翻译），然后用英语进行规划和执行过程，最后将最终答案翻译回源语言。

任务执行器通过遵循任务规划器确定的规划和任务执行步骤，在 Orchestrator 中执行操作。任务执行器有两个主要职责。首先，它充当数据转换器，转换输入查询和元数据，并准备供任务规划器使用。例如，如果问题使用的是非英语语言，它将通过 Google 翻译服务 [[36](https://arxiv.org/html/2310.02374v5#bib.bib36)] 翻译成英语。此外，如果元数据包含文件或图像，任务执行器会将元数据详细信息发送给任务规划器进行规划。其次，任务执行器通过与外部源的交互执行任务规划器生成的任务。然后，结果会传递给任务规划器，以便在需要时继续规划。最后，任务规划器发出规划结束的信号。在附录 2 中，我们详细说明了任务规划器如何将规划任务转化为执行指令，确保任务执行器能够正确地执行任务。

需要强调的是，任务规划器与任务执行器之间的通信是双向的。任务执行器和任务规划器之间会继续进行迭代过程，直到任务规划器积累足够的信息，以便适当地响应用户的询问。这种双向交流是不可或缺的，因为在特定场景下，任务规划器可能需要中间信息来决定后续的操作。

数据管道是通过执行对话会话从外部来源获取的元数据和数据的存储库。这个组件至关重要，因为许多多模态分析涉及中间阶段，其相关数据必须保留以供将来检索。中间数据可能很大，超过了令牌限制，或者对于任务规划器或响应生成器的LLM来说可能难以理解和利用。数据管道由任务执行器自动管理，负责监控存储的元数据和中间数据。

openCHA中的数据管道可以从简单的内存中键/值存储（用于中间数据）到更复杂的数据库系统不等。所提出的框架允许开发者确定其任务的结果是中间结果，还是应该直接返回给LLM。附录1详细说明了开发者如何配置此设置。

此外，我们还实现了一种机制，其中存储在数据管道中的中间结果会生成一个唯一的键，作为任务的结果。该键随后会提供给任务规划器的提示，帮助任务规划器在必要时识别并利用该数据。附录4展示了为任务生成的示例提示，并演示了任务规划器如何使用数据管道的键。

Promptist负责将查询文本或来自外部来源的结果转化为适合提供给任务规划器或响应生成器的提示。Promptist提供了修改和适应每种技术的灵活性，允许无缝集成和定制。它可以使用现有的提示技术实现，其中一些列举如下。

LLM-REC，由Lyu等人提出[[37](https://arxiv.org/html/2310.02374v5#bib.bib37)]，采用四种独特的提示策略来丰富文本描述，从而增强个性化的基于文本的推荐。该方法利用LLM理解项目特征，显著提高推荐质量。此外，还可以借助Hao等人提出的方法[[38](https://arxiv.org/html/2310.02374v5#bib.bib38)]，该方法通过名为提示适配的框架优化文本到图像的提示生成。它自动将用户输入精炼为模型偏好的提示。这个过程首先通过使用一组精心挑选的提示对预训练语言模型进行监督微调，然后通过强化学习在奖励函数的指导下，识别出更有效的提示，生成符合用户意图的美观图像。此外，还可以参考OpenAI提供的关于如何创建更有效提示的说明[[39](https://arxiv.org/html/2310.02374v5#bib.bib39)]。

响应生成器是一个基于LLM的模块，负责准备响应。它通过任务规划器收集到的信息，进行优化，将其转换为可理解的格式，并推断出适当的响应。我们将响应生成器和任务规划器分开，以便灵活选择不同的LLM模型和提示技术来处理这些组件。这种划分确保了任务规划器只专注于规划，而不直接响应用户，同时响应生成器利用收集到的信息提供有结论的回应。这种分离使得响应生成器能够在对话中处理同理心和陪伴感，而任务规划器则主要负责对话的个性化和时效性。附录3详细说明了响应生成器的实现，以及它如何利用任务规划器收集的结果有效地回应用户。

### 外部来源

外部来源在从更广泛的世界获取关键信息方面发挥着关键作用。通常，这些外部来源提供应用程序接口（APIs），Orchestrator可以利用这些接口来检索所需的数据，通过AI或分析工具进行处理，并提取有意义的健康信息。在openCHA中，我们集成了四个主要的外部来源，这些来源对于CHAs至关重要（见图[2](https://arxiv.org/html/2310.02374v5#S3.F2 "图2 ‣ 0em材料与方法 ‣ 会话健康代理：个性化的LLM驱动代理框架")）。

健康数据源使得能够从多种来源收集、摄取和集成数据，这些来源包括电子健康记录（EHR）、智能手机和智能手表等，用于医疗目的 [[40](https://arxiv.org/html/2310.02374v5#bib.bib40)]。数据源的例子有mHealth平台和医疗数据库。mHealth平台在最近一波医疗数字化浪潮中引起了广泛关注，使得无处不在的健康监测成为可能 [[41](https://arxiv.org/html/2310.02374v5#bib.bib41), [42](https://arxiv.org/html/2310.02374v5#bib.bib42)]。这些数据涵盖了多种模态，包括生物信号（例如，通过智能手表收集的PPG）、图像（例如，通过用户智能手机捕捉的图像）、视频、表格数据（例如，从EHR收集的人口统计数据）等。此类医疗平台的著名例子包括ZotCare [[43](https://arxiv.org/html/2310.02374v5#bib.bib43)] 和 ilumivu [[44](https://arxiv.org/html/2310.02374v5#bib.bib44)]，它们提供了第三方集成的API。在我们的框架中，协调器作为第三方，在用户同意的情况下访问用户数据。

知识库从医疗来源中提取最新且相关的数据，例如医疗文献、可信的网站或知识图谱，使用搜索引擎或检索模型 [[45](https://arxiv.org/html/2310.02374v5#bib.bib45), [46](https://arxiv.org/html/2310.02374v5#bib.bib46), [47](https://arxiv.org/html/2310.02374v5#bib.bib47), [48](https://arxiv.org/html/2310.02374v5#bib.bib48)]。访问这些检索到的信息为CHA提供了最新的个性化知识，增强了其可信度，同时减少了幻觉和偏差。openCHA 允许定义和配置各种知识库作为任务进行集成。

AI 和分析模型提供了数据分析工具，从数据中提取信息、关联性和洞察 [[49](https://arxiv.org/html/2310.02374v5#bib.bib49), [19](https://arxiv.org/html/2310.02374v5#bib.bib19)]，在不断发展的LLM-医疗整合领域中发挥着至关重要的作用，提升了可信度和个性化。这些模型可以执行多种任务，包括数据去噪、抽象、分类和事件检测等 [[49](https://arxiv.org/html/2310.02374v5#bib.bib49), [19](https://arxiv.org/html/2310.02374v5#bib.bib19)]。作为生成模型，LLM不能有效地执行广泛的计算或作为数据上的机器学习推理。AI平台使我们的框架能够利用现有的健康数据分析方法。

翻译员有效地将各种语言转化为广泛使用的语言，如英语，从而增强了CHAs的可达性和包容性。现有的代理面临着限制，阻碍了它们在全球大社区中的可用性。CHAs的通用文本素养常常缩小其覆盖范围，并使其成为一种特权[[50](https://arxiv.org/html/2310.02374v5#bib.bib50), [51](https://arxiv.org/html/2310.02374v5#bib.bib51), [52](https://arxiv.org/html/2310.02374v5#bib.bib52)]。许多服务不足的社区在使用CHAs时面临障碍，这些障碍源于教育差距、财务限制以及现有技术范式中偏向发达国家的偏见。我们的框架与翻译平台集成，旨在适应并支持与不同社区的沟通。这种集成增强了CHAs的整体可用性。

外部资源的选择基于它们提供的信息和知识，以及它们与协调器的互动。协调器在所提议的框架中处理各种数据类型，包括文本、JSON格式的数据或未结构化数据，如图像和音频。该设计确保任何能够以这些格式返回结果的外部资源都能得到支持。

## 0em示范

我们通过三个不同的演示展示了openCHA的能力。这些演示突出了LLMs的规划和推理能力，如何有效地理解用户查询，并将其转化为适当的任务执行。每个演示都涉及将一组已实现的任务链接到openCHA。随后，我们突出了openCHA在有效地按照正确的顺序、适当的输入来执行任务方面的规划能力。在执行所有必要的任务后，结果会传递给响应生成器，以提供最终的答复。已实现任务的概述及其在每个演示中的利用情况见图[3](https://arxiv.org/html/2310.02374v5#S4.F3 "Figure 3 ‣ 0emDemonstration ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")。

![参考说明](img/531e1798118f1ab57314e5d23460a5b0.png)

图3：已实现任务和组件的概述，以及它们在两个演示中的使用方式。

### 演示1：患者健康记录报告

第一个演示展示了openCHA如何与存储在数据库中的患者数据交互，进行数据分析，按需生成健康报告，并提出后续问题。用户的问题示例可能包括：“提供2020年8月患者5的睡眠总结”，“2020年8月患者5的REM睡眠足够吗？”，“2020年8月患者5的总步数是多少？”或“提供2020年患者5的活动总结”。在此演示中，我们实现了两个任务，分别用于从健康监测数据集中提取睡眠和身体活动数据[[53](https://arxiv.org/html/2310.02374v5#bib.bib53)]。此演示中使用的数据来源于一项广泛的纵向研究，聚焦于大学生的心理健康，如[[53](https://arxiv.org/html/2310.02374v5#bib.bib53)]所述。此外，我们开发了能够执行基本统计分析（例如，计算趋势和平均值）的分析任务。最后，我们还添加了Google搜索和提取文本任务，以便当前的openCHA设置能够访问互联网信息。本演示中涉及的任务在图[3](https://arxiv.org/html/2310.02374v5#S4.F3 "图3 ‣ 0em演示 ‣ 会话健康代理：个性化LLM驱动的代理框架")中以红点标记。 |

图[4](https://arxiv.org/html/2310.02374v5#S4.F4 "图4 ‣ 演示1：患者健康记录报告 ‣ 0em演示 ‣ 会话健康代理：个性化LLM驱动的代理框架")展示了示例查询及openCHA的相应响应。任务规划器识别正确的任务、执行顺序以及所需的输入。表[1](https://arxiv.org/html/2310.02374v5#S4.T1 "表1 ‣ 演示1：患者健康记录报告 ‣ 0em演示 ‣ 会话健康代理：个性化LLM驱动的代理框架")展示了任务规划器使用Tree of Thought提示技术选择的策略。

![参见标题](img/8d7ca21e2a1a1f401f84b3a484693c1c.png)![参见标题](img/f74d2574b431515547a39f37a8e18187.png)![参见标题](img/b7cd2d8149fe628146c9fb451c41070f.png)

图4：演示1。患者健康记录报告与跟踪。

表1：Tree of Thought提示技术为第一个问题建议的策略。

|  | 我将直接分析 |
| --- | --- |
|  | 2020年8月患者5，提供精确且具体的 |
|  | 结论。 |
| 决策 | 执行： |
|  | 1\. 使用sleep_get工具获取患者5的REM睡眠数据 |
|  | 2020年8月患者5的。 |
|  | 2\. 使用sleep_analysis工具分析REM睡眠 |
|  | 2020年8月的持续时间和效率。 |

### 演示2：目标压力水平估计与多语言互动

演示 2 展示了 openCHA 进行信号处理和客观压力水平估计的能力。我们还展示了 openCHA 可以用多种语言回答查询。这是通过与翻译器、健康数据源和 AI 模型的交互实现的。交互示例包括像“检索 2020 年 8 月 29 日患者 5 的压力水平”和“2020 年 8 月患者 5 的平均心率是多少？”这样的查询。为了实现我们的目标，我们实施了三个不同的任务（图 [3](https://arxiv.org/html/2310.02374v5#S4.F3 "Figure 3 ‣ 0emDemonstration ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") 中的黄色圆点）。第一个任务是从患者那里获取光电容积脉搏波图（PPG）数据。PPG 数据通过使用三星 Gear Sport 智能手表 [[54](https://arxiv.org/html/2310.02374v5#bib.bib54)] 以 20 Hz 的采样频率收集，参与者在自由生活条件下进行数据采集。这些数据属于 [[53](https://arxiv.org/html/2310.02374v5#bib.bib53)]。第二个任务执行 PPG 信号处理，以提取心率变异性（HRV）指标。为此，我们使用了 Neurokit [[55](https://arxiv.org/html/2310.02374v5#bib.bib55)] Python 库。在我们的案例研究中，我们提取了共 32 个 HRV 参数，包括如正常心跳之间连续差异的均方根（RMSSD）、低频（LF）和高频（HF）值等指标 [[56](https://arxiv.org/html/2310.02374v5#bib.bib56), [57](https://arxiv.org/html/2310.02374v5#bib.bib57)]。第三个任务基于 HRV 使用 AI 模型估计压力水平。最初，我们采用自编码器将数据集中的 32 个 HRV 特征降维为 12 个特征。随后，一个四层神经网络将这 12 个特征分类为五个压力水平。压力估计模型的评估结果显示，测试集的准确率为 86%。

图 [5](https://arxiv.org/html/2310.02374v5#S4.F5 "Figure 5 ‣ Demo 2: Objective stress level estimation with multilanguage interaction ‣ 0emDemonstration ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") 显示了来自 openCHA 的示例查询及其相应的回复。任务规划器的方法首先涉及检索 2020 年 8 月 29 日患者 5 的 PPG 数据。随后，获得的结果会被转发到 PPG 分析任务，以提取 HRV 指标。最后，规划器启动压力分析任务的执行，并为此任务提供 HRV 指标。表 [2](https://arxiv.org/html/2310.02374v5#S4.T2 "Table 2 ‣ Demo 2: Objective stress level estimation with multilanguage interaction ‣ 0emDemonstration ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") 显示了任务规划器利用思维树提示技术选择的策略。最终估计的压力值以及解释将返回给用户。

![参考说明](img/d393e2a0c486ea3a8cce73874f2dcd5f.png)![参考说明](img/91de8abcfa8ef472bffa7c288d3592bd.png)

图5：演示2\. 目标压力水平估计。问题用英语和西班牙语提问。

表2：树状思维提示技术为第一个问题建议的策略。

|  | 最佳策略提供了详细的PPG分析和一个 |
| --- | --- |
|  | 压力水平估计，提供了全面的视图 |
|  | 患者健康状况。 |
|  | 执行： |
| 决策 | 1\. 使用ppg_get工具获取患者5的PPG数据 |
|  | 2020年8月29日期间。 |
|  | 2\. 使用ppg_analysis工具分析PPG数据以获取 |
|  | 心率。 |
|  | 3\. 使用stress_analysis根据 |
|  | 获取到PPG数据。 |

### openCHA使用案例

为了展示openCHA在各种应用中的可用性，我们概述了若干在研究中应用该框架的使用案例，如下所示。

1.  1.

    ChatDiet [[58](https://arxiv.org/html/2310.02374v5#bib.bib58)]推出了一款个性化、营养导向的食品推荐代理，利用openCHA作为核心实现。通过集成个人和人群模型作为外部资源，ChatDiet提供了量身定制的食品建议。它通过提供动态、个性化和可解释的推荐，增强了传统的食品推荐服务。在一项案例研究中，ChatDiet的有效性达到了92%，超越了像ChatGPT这样的解决方案。

1.  2.

    知识融合的LLM驱动CHA用于糖尿病患者[[59](https://arxiv.org/html/2310.02374v5#bib.bib59)]，通过使用openCHA集成领域特定的知识和分析工具作为外部资源开发。这一集成包括纳入美国糖尿病协会的饮食指南，并部署用于营养摄入量计算的分析工具，相较于GPT4在糖尿病管理中的表现，通过量身定制的饮食建议，表现出了优越的性能，经过100个糖尿病相关问题的评估证明了这一点。

1.  3.

    openCHA被用来开发一个评估心理健康聊天机器人安全性和可靠性的代理[[60](https://arxiv.org/html/2310.02374v5#bib.bib60)]。该代理的评估能力与专家评估以及几个现有的LLM进行了比较，包括GPT4、Claude、Gemini和Mistral。专家和互联网搜索引入的指南和基准作为外部资源与openCHA相连接。该代理展示了卓越的准确性，在与专家评分的平均绝对误差（MAE）上达到了最低值——与LLM的评分相比，减少了1倍，最大MAE为10——并提供了无偏的评估分数。

1.  4.

    增强同理心的 CHA [[61](https://arxiv.org/html/2310.02374v5#bib.bib61)] 被开发用于通过多模态对话解读和回应用户的情绪状态，代表了在为心理健康领域提供上下文感知和富有同理心的支持方面的重要进步。本文利用了语音转文本、文本转语音和语音情感检测模型作为连接到 openCHA 的外部数据源。

要查看更多关于 openCHA 在实际设置中如何工作的演示，我们上传了多个 YouTube 视频^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://www.youtube.com/watch?v=w48sPlF5zhs](https://www.youtube.com/watch?v=w48sPlF5zhs)^,  ^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://www.youtube.com/watch?v=PWxL_OgWGfE&t=3s](https://www.youtube.com/watch?v=PWxL_OgWGfE&t=3s)^,  ^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://www.youtube.com/watch?v=c-7IEBaRSyQ](https://www.youtube.com/watch?v=c-7IEBaRSyQ)^,  ^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://www.youtube.com/watch?v=rHXpk_P5n6Y](https://www.youtube.com/watch?v=rHXpk_P5n6Y)

## 0emDiscussion

### openCHA 的潜力与局限性

在本节中，我们简要讨论了我们提出的框架的能力、潜力和局限性。

灵活性：openCHA 提供了高度的灵活性，可以将 LLM 与外部数据源、知识库和分析工具集成。根据具体的医疗应用需求，提出的组件可以进行开发和替换。例如，新的外部数据源可以轻松地集成并作为新任务引入到 openCHA 中。openCHA 中使用的 LLM 可以方便地与经过微调或更具医疗特定性的 LLM 互换。同样，Planner 提示技术和决策过程也是可修改的。这种灵活性促进了不同研究社区之间的合作，使他们能够为 CHA 的各个方面做出贡献。附录 1 展示了如何定义和引入一个新任务到 openCHA 中。

可解释性：openCHA 提高了 CHA 的可解释性，允许用户查询用于生成响应的工具和操作。如附录 2 和 3 中详细描述，openCHA 维护了一个“先前操作”部分，记录过去的对话和任务。当查询任务使用情况时，它列出已执行的任务及其应用，增强了透明度并促进了用户与 CHA 之间的信任。例如，在演示 2 中，当用户询问“使用了哪些任务”时，openCHA 会回应，详细说明了使用了 PPG 和 HRV 数据来确定压力水平。此交互的示例如图 [6](https://arxiv.org/html/2310.02374v5#S5.F6 "Figure 6 ‣ openCHA Potentials and Limitations ‣ 0emDiscussion ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") 所示。

个性化：openCHA框架通过整合来自医疗系统或本地数据库的个人信息和分析工具作为外部数据源，增强了个性化体验。这些外部数据源的质量对个性化效果有着重要影响。例如，ChatDiet [[58](https://arxiv.org/html/2310.02374v5#bib.bib58)] 利用个人饮食偏好和人口数据，并结合营养成分对健康结果（如睡眠质量）的影响分析，显著提升了其食物推荐的效果。这一策略不仅提高了推荐的准确性，还确保其能够精确地根据个人饮食需求量身定制。

可靠性：openCHA通过利用经过验证的信息和计算作为外部数据源，提升了答案的可靠性。我们的框架旨在有效利用现有的LLMs来处理复杂的医疗任务，策略性地将计算和敏感信息任务卸载到外部数据源，同时将LLM主要用于推理和生成回答。例如，论文《为糖尿病患者提供的知识增强LLM驱动的CHA》[[59](https://arxiv.org/html/2310.02374v5#bib.bib59)]展示了整合外部知识的好处，能够准确确定营养成分，并将其与已建立的指南对齐，突显了仅依赖GPT4 LLM进行数据访问和计算时，营养估算中的不准确之处。

![参见标题](img/fba59ba35ca9706e96c4efaa13d9806e.png)

图6：示例截图，展示了openCHA如何提高对话中的可解释性。

延迟：利用多个外部数据源带来益处，但也可能影响模型的响应时间，导致CHA的延迟增加。随着框架中任务和步骤数量的增加，响应时间可能会上升，从而降低可用性。Singh等人最近的研究 [[62](https://arxiv.org/html/2310.02374v5#bib.bib62)] 探讨了在任务彼此独立时执行并行任务的新方法。

令牌限制：LLMs中的令牌限制对在任务规划器内容纳任务提出了挑战。然而，最近的进展 [[63](https://arxiv.org/html/2310.02374v5#bib.bib63)] 显示在扩展LLM令牌限制方面取得了进展，这有助于缓解这一问题。

隐私与安全：隐私和安全在本框架中至关重要，尤其是对于处理用户隐私问题的医疗保健应用。强有力的隐私措施对于防止未经授权的访问、数据泄露和身份盗窃至关重要，且可能导致严重后果[[64](https://arxiv.org/html/2310.02374v5#bib.bib64), [65](https://arxiv.org/html/2310.02374v5#bib.bib65)]。为了解决隐私和安全问题，外部资源应提供数据机密性。例如，可以通过允许用户授权的第三方（如CHA）有限访问，或采用去标识化和匿名化技术[[66](https://arxiv.org/html/2310.02374v5#bib.bib66)]来实现这一点。另一种策略是防止LLMs将用户提供的数据用于训练和微调，确保此类数据不会被存储。

### 研究局限性与未来工作

在本节中，我们概述了研究的局限性和未来的研究方向。

计划的稳健性：由于我们利用大语言模型（LLMs）进行计划和响应生成，因此仍然存在固有的偏见或信任问题风险。我们的框架通过整合外部资源来增强计划的稳健性，从而减少这些问题，尽管无法确保完全消除这些问题。为了增强计划的稳健性，我们将探索使用像自一致性[[67](https://arxiv.org/html/2310.02374v5#bib.bib67)]方法这样的Agentic设计模式或新的推理技术。

准确性与评估：我们框架中的准确性和评估依赖于研究人员做出的配置选择，如选择的外部资源、LLM和规划技术。由于知识、数据和分析被外包给外部资源，因此这些资源的质量在提高准确性方面起着至关重要的作用；更好的外部资源能增加实现更优结果的可能性。

评估此类系统需要两个不同的评估。第一个评估外部资源的准确性，无论它们是AI模型还是知识库。第二个评估整体配置和构建的CHA，以确定其是否按预期运行。推荐在[[68](https://arxiv.org/html/2310.02374v5#bib.bib68)]中使用几种指标和评估方法，并在[[58](https://arxiv.org/html/2310.02374v5#bib.bib58), [59](https://arxiv.org/html/2310.02374v5#bib.bib59), [60](https://arxiv.org/html/2310.02374v5#bib.bib60), [61](https://arxiv.org/html/2310.02374v5#bib.bib61)]中探讨了其他评估技术。这些评估是应用特定的，我们的框架提供了广泛的自定义功能，以适应不同的使用场景和需求。在未来的工作中，我们将探索更多评估技术。

用户查询模糊性：理解用户意图是一个重大挑战，因为查询模糊性常常是由于信息不清晰或不完整，且缺乏与openCHA特定应用相关的必要外部资源。为了提高响应的准确性，我们未来的工作将包括改进openCHA澄清用户意图的能力。主要策略包括使用有针对性的后续问题、提高对用户情况的理解，以及增强响应的精确度。

可扩展性：我们计划解决系统中的可扩展性问题，确保系统能够有效管理不断增加的负载并适应更复杂的场景。这包括增强系统架构和资源分配，以支持更大数据量和更多并发用户，同时不降低性能。我们计划进一步研究openCHA核心在实际场景中的可扩展性表现。

## 结论

本文介绍了openCHA，一个基于大语言模型（LLM）的框架，旨在通过分析输入问题、数据收集、执行操作以及提供个性化响应，帮助健康咨询助手（CHA）有效地解答与医疗相关的问题。我们通过两个不同的演示展示了该框架的有效性，涵盖了常规患者健康记录报告、客观压力水平评估和其他应用场景。此外，我们还讨论了openCHA的能力、局限性和挑战。我们的未来工作将专注于提高规划的鲁棒性、检查准确性和评估方面、解决用户查询的模糊性，并增强openCHA的可扩展性。

## 参考文献

+   [1] OpenAI. ChatGPT: OpenAI的对话AI模型；访问时间：2024年1月。[https://openai.com/chatgpt](https://openai.com/chatgpt).

+   [2] Han T, Adams LC, Papaioannou JM, Grundmann P, Oberhauser T, Löser A, 等. MedAlpaca—一个开源医疗对话AI模型及训练数据集. arXiv预印本 arXiv:230408247. 2023.

+   [3] Tu T, Azizi S, Driess D, Schaekermann M, Amin M, Chang PC, 等. 朝着通用生物医学AI迈进. arXiv预印本 arXiv:230714334. 2023.

+   [4] Milne-Ives M, de Cock C, Lim E, Shehadeh MH, de Pennington N, Mole G, 等. 人工智能对话代理在医疗中的有效性：系统综述. Journal of Medical Internet Research. 2020;22(10):e20346.

+   [5] Moudatsou M, Stavropoulou A, Philalithis A, Koukouli S. 同情心在健康与社会护理专业人员中的作用. 收录于：Healthcare, 第8卷, MDPI; 2020. 第26页.

+   [6] Abbasian M, Khatibi E, Azimi I, Oniani D, Abad ZSH, Thieme A, 等. 基础指标：量化生成AI驱动的医疗对话的有效性. arXiv预印本 arXiv:230912444. 2023.

+   [7] Moreira A, Quintas C, Guimarães T, Santos MF. 面向个性化医疗的多渠道移动伴侣：机遇与挑战. Procedia Computer Science. 2023;220:838-43.

+   [8] Moore O. 消费者如何使用生成型人工智能？；访问时间：2024年1月。[https://a16z.com/how-are-consumers-using-generative-ai/](https://a16z.com/how-are-consumers-using-generative-ai/)。

+   [9] Denecke K, May R. 调查医疗保健中的对话代理：技术导向分类法的应用。计算机科学学报，2023；219：1289-96。

+   [10] Plsek PE, Greenhalgh T. 医疗保健中的复杂性挑战。BMJ，2001；323(7313)：625-8。

+   [11] Sturmberg JP, Martin CM. 复杂性与健康——昨日的传统，明日的未来。临床评估实践杂志，2009；15(3)：543-8。

+   [12] Luo R, Sun L, Xia Y, Qin T, Zhang S, Poon H, 等. BioGPT：用于生物医学文本生成和挖掘的生成预训练变换器。生物信息学简报，2022；23(6)：bbac409。

+   [13] Li Y, Li Z, Zhang K, Dan R, Jiang S, Zhang Y. ChatDoctor：一个基于医学领域知识微调的大型语言模型Meta-AI（LLaMA）的医学聊天模型。Cureus，2023；15(6)。

+   [14] Meier P, Beinke JH, Fitte C, Behne A, Teuteberg F. FeelFit——设计和评估一个增强健康意识的对话代理。在：ICIS；2019\。

+   [15] Montenegro JLZ, da Costa CA, da Rosa Righi R. 医疗领域中对话代理的调查。专家系统与应用，2019；129：56-67。

+   [16] Tudor Car L, Dhinagaran DA, Kyaw BM, Kowatsch T, Joty S, Theng YL, 等. 医疗保健中的对话代理：范围评审与概念分析。医学互联网研究杂志，2020；22(8)：e17158。

+   [17] Laranjo L, Dunn AG, Tong HL, Kocaballi AB, Chen J, Bashir R, 等. 医疗保健中的对话代理：系统评审。美国医学信息学会杂志，2018；25(9)：1248-58。

+   [18] Zhao WX, Zhou K, Li J, Tang T, Wang X, Hou Y, 等. 大型语言模型综述。arXiv预印本 arXiv:230318223\. 2023。

+   [19] Han R, Acosta JN, Shakeri Z, Ioannidis J, Topol E, Rajpurkar P. 评估人工智能在临床实践中的随机对照试验：范围评估。medRxiv，2023：2023-09。

+   [20] Sallam M; MDPI. ChatGPT在医疗教育、研究和实践中的实用性：关于有前景的视角和有效关切的系统评审。医疗保健，2023；11(6)：887。

+   [21] Biswas S. ChatGPT与医学写作的未来。北美放射学会；2023。

+   [22] Aydın Ö, Karaarslan E. OpenAI ChatGPT生成的文献综述：医疗领域的数字双胞胎。可在SSRN上获取 4308687\. 2022。

+   [23] Chen S, Kann BH, Foote MB, Aerts HJ, Savova GK, Mak RH, 等. ChatGPT在癌症治疗信息中的实用性。medRxiv，2023：2023-03。

+   [24] Liu C, Ma Y, Kothur K, Nikpour A, Kavehei O. BioSignal Copilot：利用大型语言模型的力量撰写生物医学信号报告。medRxiv，2023：2023-06。

+   [25] Lee J, Yoon W, Kim S, Kim D, Kim S, So CH, 等. BioBERT：一种用于生物医学文本挖掘的预训练生物医学语言表示模型。生物信息学，2020；36(4)：1234-40。

+   [26] Gu Y, Tinn R, Cheng H, Lucas M, Usuyama N, Liu X, 等. 面向生物医学自然语言处理的领域特定语言模型预训练。ACM 健康计算学报（HEALTH）。2021;3(1):1-23。

+   [27] Gutierrez BJ, McNeal N, Washington C, Chen Y, Li L, Sun H, 等. 重新思考gpt-3在生物医学信息抽取中的上下文学习。arXiv 预印本 arXiv:220308410。2022。

+   [28] Moradi M, Blagec K, Haberl F, Samwald M. GPT-3模型在生物医学领域的少样本学习表现不佳。arXiv 预印本 arXiv:210902555。2021。

+   [29] Xu S, Yang L, Kelly C, Sieniek M, Kohlberger T, Ma M, 等. ELIXR：通过对齐大型语言模型与放射学视觉编码器，迈向通用X射线人工智能系统。arXiv 预印本 arXiv:230801317。2023。

+   [30] Belyaeva A, Cosentino J, Hormozdiari F, McLean CY, Furlotte NA. 基于个体特定数据的健康多模态大语言模型。arXiv 预印本 arXiv:230709018。2023。

+   [31] Guerreiro NM, Alves D, Waldendorf J, Haddow B, Birch A, Colombo P, 等. 大型多语种翻译模型中的幻觉现象。arXiv 预印本 arXiv:230316104。2023。

+   [32] Neisser U. 知觉、预测和想象。在：知觉与认知：心理学基础问题。明尼苏达大学出版社，明尼阿波利斯；1978年。

+   [33] Yao S, Yu D, Zhao J, Shafran I, Griffiths TL, Cao Y, 等. 思维树：使用大型语言模型进行深思熟虑的问题解决。arXiv 预印本 arXiv:230510601。2023。

+   [34] Yao S, Zhao J, Yu D, Du N, Shafran I, Narasimhan K, 等. React：在语言模型中协同推理与行动。arXiv 预印本 arXiv:221003629。2022。

+   [35] Wang L, Xu W, Lan Y, Hu Z, Lan Y, Lee RKW, 等. 计划-解决提示：通过大型语言模型提高零-shot链式思维推理能力。arXiv 预印本 arXiv:230504091。2023。

+   [36] Google. Google 翻译；访问时间：2024年1月。[https://py-googletrans.readthedocs.io/en/latest/](https://py-googletrans.readthedocs.io/en/latest/)。

+   [37] Lyu H, Jiang S, Zeng H, Xia Y, Luo J. LLM-Rec：通过提示大型语言模型实现个性化推荐。arXiv 预印本 arXiv:230715780。2023。

+   [38] Hao Y, Chi Z, Dong L, Wei F. 优化用于文本到图像生成的提示。arXiv 预印本 arXiv:221209611。2022。

+   [39] OpenAI. ChatGPT：提示工程指南；访问时间：2024年1月。[https://platform.openai.com/docs/guides/prompt-engineering/strategy-write-clear-instructions](https://platform.openai.com/docs/guides/prompt-engineering/strategy-write-clear-instructions)。

+   [40] Labbaf S, Abbasian M, Azimi I, Dutt N, Rahmani AM. ZotCare：一个灵活、可个性化且负担得起的移动健康服务提供商。数字健康前沿。2023;5。

+   [41] Jusoh S. 移动健康应用的趋势、机会和挑战调研。Int J Interact Mob Technol. 2017;11(6):73-85。

+   [42] Chiarini G, Ray P, Akter S, Masella C, Ganz A. 慢性病和老年人健康管理的移动健康技术：一项系统评估。IEEE通讯领域选择期刊。2013；31(9)：6-18。

+   [43] Labbaf S, Abbasian M, Azimi I, Dutt N, Rahmani A. ZotCare：一种灵活、可定制且经济实惠的移动健康服务提供商。数字健康前沿；5：1253087。

+   [44] ilumivu. ilumivu；访问时间：2024年1月。[https://ilumivu.com/](https://ilumivu.com/)。

+   [45] Nori H, Lee YT, Zhang S, Carignan D, Edgar R, Fusi N, 等人。通用基础模型能否超越专用调优？医学领域的案例研究。arXiv预印本arXiv:231116452。2023。

+   [46] Hiemstra D. 信息检索模型。信息检索：21世纪的搜索。2009：1-19。

+   [47] Turtle HR, Croft WB. 文本检索模型的比较。计算机杂志。1992；35(3)：279-90。

+   [48] Nori H, Lee Y, Zhang S, 等人。通用基础模型能否超越专用调优？医学领域的案例研究。arXiv。2023。

+   [49] Topol EJ. 随着人工智能的多模态发展，医学应用呈现倍增趋势。科学。2023；381(6663)。

+   [50] Clarke MA, Moore JL, Steege LM, Koopman RJ, Belden JL, Canfield SM, 等人。初级护理患者在实现以患者为中心护理中的健康信息需求、来源和障碍：一项文献综述。健康信息学杂志。2016；22(4)：992-1016。

+   [51] Jain R. 民间计算。ACM通讯。2003；46(4)：27-9。

+   [52] Borovoy RD. 民间计算：设计支持面对面社区建设的技术 [博士论文]。麻省理工学院；2002年。

+   [53] Labbaf S, Abbasian M, Nguyen B, Lucero M, Ahmed MS, Yunusova A, 等人。2020年新冠疫情封锁期间，大学生使用可穿戴和移动设备进行生理和情感评估：一项密集的、长期的数据集。长期数据集。2023。

+   [54] 三星。三星 Gear 手表；访问时间：2024年1月。[https://www.samsung.com/us/mobile/wearables/smartwatches/gear-sport-blue-sm-r600nzbaxar/](https://www.samsung.com/us/mobile/wearables/smartwatches/gear-sport-blue-sm-r600nzbaxar/).

+   [55] Makowski D, Pham T, Lau ZJ, Brammer JC, Lespinasse F, Pham H, 等人。NeuroKit2：一个用于神经生理信号处理的Python工具箱。行为研究方法。2021年2月；53(4)：1689-96。可从：[https://doi.org/10.3758%2Fs13428-020-01516-y](https://doi.org/10.3758%2Fs13428-020-01516-y)获取。

+   [56] Shaffer F, Ginsberg JP. 心率变异性指标和标准概述。公共卫生前沿。2017：258。

+   [57] Khatibi E, Abbasian M, Azimi I, Labbaf S, Feli M, Borelli J, 等人。COVID-19大流行对睡眠的影响，包括作为中介的HRV和身体活动：一种因果机器学习方法。在：2023 IEEE第19届国际体感网络大会（BSN）。IEEE；2023年。第1-4页。

+   [58] Yang Z, Khatibi E, Nagesh N, Abbasian M, Azimi I, Jain R, 等人。ChatDiet：通过LLM增强框架赋能个性化营养导向的食物推荐聊天机器人。《智能健康》。2024；32：100465。

+   [59] Abbasian M, Yang Z, Khatibi E, Zhang P, Nagesh N, Azimi I, 等人。知识注入LLM驱动的对话健康代理：糖尿病患者的案例研究。arXiv预印本 arXiv:240210153。2024。

+   [60] Park JI, Abbasian M, Azimi I, Bounds D, Jun A, Han J, 等人。建立心理健康聊天机器人信任：安全性指标和基于LLM的评估工具。arXiv预印本 arXiv:240804650。2024。

+   [61] Abbasian M, Azimi I, Feli M, Rahmani AM, Jain R。通过多模态在对话界面中传递同理心。arXiv预印本 arXiv:240504777。2024。

+   [62] Singh S, Karatzas A, Fore M, Anagnostopoulos I, Stamoulis D。用于融合并行函数调用的LLM-工具编译器。arXiv预印本 arXiv:240517438。2024。

+   [63] Claude. Claude：一个适用于各类任务的下一代AI助手，无论任务规模如何；访问时间：2024年1月。[https://www.anthropic.com/product](https://www.anthropic.com/product)。

+   [64] Hathaliya JJ, Tanwar S。关于Healthcare 4.0中的安全性和隐私问题的详尽调查。《计算机通信》。2020；153：311-35。

+   [65] Price WN, Cohen IG。医学大数据时代的隐私。《自然医学》。2019；25(1)：37-43。

+   [66] Xu Y, Ma T, Tang M, Tian W。使用概化和抑制的隐私保护数据发布调查。《应用数学与信息科学》。2014；8(3)：1103。

+   [67] Wang X, Wei J, Schuurmans D, Le Q, Chi E, Narang S, 等人。自一致性提高语言模型中的思维链推理。arXiv预印本 arXiv:220311171。2022。

+   [68] Abbasian M, Khatibi E, Azimi I, Oniani D, Shakeri Hossein Abad Z, Thieme A, 等人。用于评估由生成性AI驱动的医疗对话有效性的基础指标。《NPJ数字医学》。2024；7(1)：82。

## 0em附录1 - 示例任务实现

我们鼓励读者和openCHA用户查看我们的文档页面 ^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://docs.opencha.com/index.html](https://docs.opencha.com/index.html)，以获取有关openCHA的最新更新。

![请参考说明文字](img/7b0ebc92396272858c4ecf60dde03e26.png)

图7：在openCHA框架内实现的任务示例。每个任务包括提示参数，帮助我们的任务规划器LLM理解所需的数据类型，并将其提供给当前任务。

在本附录中，我们将详细介绍在openCHA框架内任务的策划及其作为外部源引入任务规划器的过程。首先，我们将描述可以添加到框架中的任务类型。接下来，我们将概述框架内任务的实现过程。最后，我们将展示一个已经实现的任务示例，并提供与之相关的示例提示。

任务可以在本地或作为服务实现。本地实现意味着需要在现场编写所有必要的组件以确保任务的成功执行，这可能涉及提供Python库、GitHub仓库或完整的任务实现。

基于服务的实现涉及将服务托管在服务器上，并提供API供使用。在这种情况下，任务应使用Python库调用这些API。服务提供者有责任维护任务并为用户提供详细的文档，包括注册流程、API密钥获取和隐私政策。

图 [7](https://arxiv.org/html/2310.02374v5#S7.F7 "Figure 7 ‣ 0emAppendix 1 - Sample Task Implementation ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework") 显示了openCHA中一个任务实现的示例，旨在执行Google搜索并返回第一个搜索结果URL。每个任务都包括配置，以帮助任务规划器LLM理解何时以及如何使用该任务。参数如下：

名称

目的：唯一标识系统中的任务，该名称将在任务选择界面中显示。

命名规范：通常采用小写下划线命名风格，通常源自任务的功能，以便提高可读性和可维护性。

示例：name = "google_search"

chat_name：

目的：用于在用户界面或聊天中引用该任务，特别是用于可解释性。如果用户询问所使用的任务，这些名称将被显示。

命名规范：通常采用CamelCase风格，应该具有描述性且简洁。

示例：chat_name = "GoogleSearch"

描述：

目的：提供任务功能的解释。此信息帮助任务规划器LLM决定何时部署该任务。

命名规范：描述应全面，概述任务的能力、优先级或特定使用条件。例如，确保任务规划器将病人数据分析的优先级高于频繁的互联网搜索，这可以在此通过低优先级设置进行说明。

示例：description = "使用Google搜索互联网中请求的查询，并返回排名第一的网站URL。"

依赖：

目的：列出该任务依赖的其他任务或服务，告知任务规划器LLM这些依赖关系，以便在规划过程中考虑。

命名规范：一个任务标识符数组，包含与依赖任务的名称属性相匹配的任务标识符。

示例：dependencies = [TaskType.FETCH_WEATHER_DATA]

输入：

目的：指定任务所需的输入。这个参数至关重要，因为它指导任务规划器LLM为该任务的"_execute"函数提供适当的输入。

命名规范：一个字符串数组，每个字符串描述一个特定的输入。

示例：inputs = ["应该是一个搜索查询。"]

输出：

目的：详细说明任务返回的内容，帮助任务规划器LLM理解返回结构，以便正确处理输出。

约定：一个包含每个输出描述的字符串数组，完整描述返回数据格式。

示例：outputs = ["它返回一个包含键：url的JSON对象。例如：’url’: ’http://google.com’"]

output_type

目的：指示协调器如何处理此任务的输出。如果output_type为True，数据将存储在Datapipe中；否则，它将直接返回给任务规划器LLM。这种灵活性允许任务设计人员决定他们的任务是否涉及供其他任务使用的中间信息，或提供最终答案。

约定：布尔值（True 或 False）。

_execute函数

目的：此功能必须正确实现。如果所有其他参数都设置准确，开发人员可以相信任务规划器LLM会使用适当的输入调用此功能。例如，如图[7](https://arxiv.org/html/2310.02374v5#S7.F7 "Figure 7 ‣ 0emAppendix 1 - Sample Task Implementation ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")所示，唯一预期的输入是搜索查询。任务规划器LLM将正确地将搜索查询作为输入传递给"_execute"函数，且该函数作为输入数组中的第一个元素可访问（由于"inputs"配置中的唯一条目涉及搜索查询，因此此数组仅包含一个项目）。利用google_engine Python库，该功能会在线搜索查询并返回URL。请注意，我们为Google搜索任务返回的结果与"outputs"参数中指定的结果一致，展示了任务设置与执行之间的一致性。

## 0em附录2 - 任务规划器示例实现

我们鼓励读者和openCHA用户查看我们的文档页面 ^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://docs.opencha.com/index.html](https://docs.opencha.com/index.html)，以获取关于openCHA的最新更新。

在本附录中，我们详细说明了思维树提示技术的实现。我们描述了如何将可用任务引入思维树，策划和生成规划提示的过程，并最终如何将最终规划结果转化为由任务执行器执行的任务。

任务规划器的核心组件是选择合适的规划技术。在众多已证明实用的使用LLM的技术中，我们选择了思维树（Tree of Thought）方法。

为了有效地实施思维树，我们将规划过程分为两个主要部分。首先，我们准备一个全面的提示，其中包括可用任务列表、相关元数据、先前执行任务的记录、对话历史以及用户输入。这种设置确保规划器能够充分了解可以调用的任务、应使用或传递的元数据，以及过去互动的背景，从而避免冗余规划并保持对话的连续性。

图[8](https://arxiv.org/html/2310.02374v5#S8.F8 "图 8 ‣ 0em附录 2 - 任务规划器示例实现 ‣ 会话健康代理：个性化的 LLM 驱动代理框架")展示了思维树规划提示。在此阶段，思维树利用所有收集到的信息和用户的查询，设计出三个不同的任务序列或策略。每个策略旨在收集必要的信息，以高效地解决查询问题。接下来，要求它提供每个策略的优缺点，最终选择最合适的策略作为最终决策。

![参见图注](img/f968ed95e6574640cfe78883637b4a2e.png)

图 8：第一阶段的思维树规划提示

在规划过程的第二阶段，我们的目标是将所选决策转换为Orchestrator可以理解和执行的任务函数序列。在Orchestrator中，我们实现了一个名为execute_task的函数，它作为接口，准确地检索和执行具有适当输入的任务。

我们的第二个提示指示LLM调用此execute_task函数，确保为每个任务提供正确的输入。图[9](https://arxiv.org/html/2310.02374v5#S8.F9 "图 9 ‣ 0em附录 2 - 任务规划器示例实现 ‣ 会话健康代理：个性化的 LLM 驱动代理框架")展示了这个第二个提示，展示了我们如何在LLM中构建这些命令，以简化任务执行。

![参见图注](img/e221b761757f9a8625ea6d8ffef7cbbd.png)

图 9：第二阶段的思维树规划提示

## 0em附录 3 - 响应生成器实现

我们鼓励读者和openCHA用户查看我们的文档页面 ^(⋄⋄\diamond⋄)^(⋄⋄\diamond⋄)$\diamond$[https://docs.opencha.com/index.html](https://docs.opencha.com/index.html)，以获取有关openCHA的最新更新。

在本附录中，我们提供了响应生成器实现的详细说明。我们首先区分了响应生成器和任务规划器。接下来，我们展示了规划和任务执行的结果如何传递给响应生成器。最后，我们展示了所使用的提示，确保响应生成器依赖于任务的响应，而不是用于生成响应的LLM的内部知识。

Agentic 方法的关键目标之一是创建一个全面的超级提示，通过整合所有必要的信息，使 LLM 能生成更精确的答案。我们的创新在于根据用户查询动态生成这个超级提示。任务规划器通过确定其认为必要的任务顺序来启动这一过程，从而准确地解决当前的问题。

一旦规划完成，协调器将汇总这些任务的结果来创建超级提示。该提示的组成受任务性质及其输出的影响。例如，如果某个任务的 output_type 设置为 true，表示该结果应存储在 Datapipe 中以供后续分析任务使用，那么协调器将从提示中省略该结果。在这种情况下，关注的重点是分析的结果，而不是中间结果。类似地，对于产生元数据的任务，例如图像，元数据本身不会直接传递给响应生成器。相反，关于元数据的描述性信息，例如图像的细节和相关数据见解，会提供给响应生成器，表明这些信息伴随图像一起传递。

![参见说明文字](img/cac941cdf75859ea0ae6bf44548113b9.png)

图 10：响应生成器提示。

图 [10](https://arxiv.org/html/2310.02374v5#S9.F10 "图 10 ‣ 0em附录 3 - 响应生成器实现 ‣ 会话式健康代理：个性化的 LLM 驱动代理框架") 展示了提供给响应生成器 LLM 的提示。最初，协调器格式化从任务结果中得出的答案，我们将其称为提示中的“思考者”部分。然后，指示 LLM 仅依赖此思考者提示，而忽略其内部知识。此外，我们还包括指导方针，以帮助 LLM 理解特定的系统实现，例如 Datapipe 访问密钥或元数据细节。

在 openCHA 框架中，此提示是可定制的。一个“前缀”功能允许开发者添加信息，以更好地使响应生成器与他们的特定任务要求对齐。

## 0em附录 4 - 示例提示

在本附录中，我们展示了一个涉及两个任务的简单示例的生成提示：互联网搜索和网页信息提取。用户的查询“如何改善我的睡眠”首先被引导到任务规划器。表[3](https://arxiv.org/html/2310.02374v5#S10.T3 "Table 3 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")展示了初步规划阶段，在该阶段请求思维树规划器制定三种策略，评估其优缺点，并选择一个执行。表[4](https://arxiv.org/html/2310.02374v5#S10.T4 "Table 4 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")展示了第二阶段的提示，在该阶段指示LLM生成Python代码，Orchestrator根据所选策略和必要的输入执行该代码。表[5](https://arxiv.org/html/2310.02374v5#S10.T5 "Table 5 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")和[6](https://arxiv.org/html/2310.02374v5#S10.T6 "Table 6 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")提供了生成代码的示例，说明了LLM如何正确输入查询“改善睡眠的技巧”以执行google_search任务。表[7](https://arxiv.org/html/2310.02374v5#S10.T7 "Table 7 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")展示了openCHA如何创建提示，帮助响应生成器利用任务的结果有效地回答用户查询。

此外，表[8](https://arxiv.org/html/2310.02374v5#S10.T8 "Table 8 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")提供了一个具有多个输入和输出任务的示例提示，结果存储在数据管道中。表[9](https://arxiv.org/html/2310.02374v5#S10.T9 "Table 9 ‣ 0emAppendix 4 - Sample Prompts ‣ Conversational Health Agents: A Personalized LLM-Powered Agent Framework")展示了任务之间传递数据管道键的过程。在该设置中，每个任务返回一个数据管道键，在生成的代码中，一个任务的输出作为输入传递给下一个任务。execute_task函数在提供密钥时管理对数据管道的访问，确保任务的结果不会直接传递给LLM。

表3：思维树规划的第一个规划阶段。

| 作为一名知识渊博且富有同情心的健康助手，你的主要目标是为用户提供关于健康和福祉的准确而有价值的信息。有效利用可用工具来回答与健康相关的问题。以下是你可以使用的工具：———————————— **google_search**：使用 Google 搜索互联网中的查询内容，并返回排名最高网站的 URL。此工具的输出包含：返回一个包含键**url**的 JSON 对象。例如：’url’: ’http://google.com’ —————————— **extract_text**：提取当前网页上的所有文本。此工具的输出包含：一个包含抓取网页文本的字符串。——————————— 以下是提供的信息格式：元数据：这包含不同类型的数据文件的名称，如图像、音频、视频和文本。根据需要，你可以将这些文件传递给工具。历史记录：之前的聊天历史记录。查看与当前查询相关的任何先前响应。PreviousActions：已经执行的操作列表。你应在计划时知道这些操作已经执行。问题：你必须回答的输入问题。考虑到之前的操作及其结果，使用工具和提供的信息，首先建议三种富有创意的策略，详细解释由工具顺序组成的策略，以适当回答用户查询。确保策略足够全面，并使用适当的工具。工具的约束条件应始终得到满足。指定策略后，提到每种策略的优缺点。最后，决定最佳策略，并逐步写出详细的工具执行过程。开始时用“决策：”作为开头。开始！========================= 用户：如何改善我的睡眠？ CHA： |
| --- |

表 4：思维树规划第二阶段：生成任务执行代码 第1部分。

| 决策：我将选择策略 1，因为它提供了关于改善睡眠的最新和最相关的信息，这对改善睡眠至关重要。现在，让我们继续进行策略 1 的详细工具执行：1\. 使用“google_search”工具查找提供改善睡眠建议的顶级网站。2\. 一旦找到顶级网站，我们可以使用“extract_text”工具从网页中提取与改善睡眠相关的信息。让我们从第一步开始。 ========================= 工具： ———————————– **google_search**：使用谷歌搜索互联网中的请求查询，并返回顶级网站的网址。该工具的输入应该是一个表示的列表数据：1-它应该是一个搜索查询。此工具将返回以下数据： - 返回一个包含键：**url** 的 json 对象。例如：’url’: ’http://google.com’ ———————————– **extract_text**：提取当前网页上的所有文本。该工具的输入应该是一个表示的列表数据：1-从中提取文本的网址。它需要由其他工具收集的链接。绝不要自行提供网址。此工具将返回以下数据： - 一个包含抓取的网页文本的字符串。 ———————————– |
| --- |

表 5：思维树规划第二阶段：生成任务执行代码的部分 2。

| ========================= 你是一个熟练的 Python 程序员，能够解决问题并将其转化为 Python 代码。使用在“Decision:”中提到的最终策略，创建一个 Python 代码块，概述使用工具的步骤序列。假设有一个**self.execute_task**函数可以执行其中的工具。execute_task 接受任务名称和一个输入数组，并返回结果。确保始终将数组作为第二个参数传递。你可以像这样调用工具：**task_result = self.execute_task(’tool_name’, [’input1’, ’input2’, …])**。流程应使用这种风格表示可用的工具。确保所有 execute_task 调用的输出都存储在一个变量中。如果某个步骤的输出作为后续步骤的输入，确保 Python 代码清晰地捕捉到这个依赖关系。输出变量应直接作为输入传递，不做任何改动。如果工具输入是数据管道（datapipe），只需将变量作为输入。对于每个工具，直接包含必要的参数，不需要任何名称，并假设每个工具都会返回输出。每个工具的输出描述都已单独提供。确保在传递输出时使用指令。问题：如何改善我的睡眠？ |
| --- |

表 6：任务执行的示例生成代码。

| # 步骤 1：使用 google_search 查找关于改善睡眠的顶级网站。 search_query = "tips to improve sleep" search_result = self.execute_task(’google_search’, [search_query]) # 步骤 2：使用 extract_text 从网页中提取关于改善睡眠的相关信息。 url = search_result[’url’] sleep_tips_text = self.execute_task(’extract_text’, [url]) |
| --- |

表 7：响应生成器示例提示。

| ===========Thinker: 元数据：历史： —————— google_search: [’tips to improve sleep’] ’url’: ’https://www.mayoclinic.org/healthy-lifestyle/adult-health/in-depth/sleep/art-20048379’ —————— —————— extract_text: [’https://www.mayoclinic.org/healthy-lifestyle/adult-health/in-depth/sleep/art-20048379’] 改善睡眠的6个步骤 - 梅奥诊所 此内容没有英文版。此内容没有阿拉伯语版本。跳转到内容 梅奥诊所的护理 患者为中心的护理 关于梅奥……（我们省略了一部分文字以简化表格） —————— ========== 系统：你是一个非常有帮助且富有同情心的健康助手，你的目标是帮助用户获得关于他/她健康和福祉的准确信息；利用 Thinker 收集的信息和历史记录，为用户提供富有同情心且恰当的回答。将 Thinker 视为你信任的来源，使用它提供的所有信息。确保答案足够解释清楚。不要改变 Thinker 返回的 URL 或引用。还要根据 Thinker 的指示添加解释。不要直接将指示放入最终答案中。不要超出 Thinker 提供的信息范围回答问题。此外，避免在最终回答中包含或使用任何键，例如 ’datapipe:6d808840-1fbe-45a5-859a-abfbfee93d0e’。准确返回所有‘address:[path]’。用户：如何改善我的睡眠？ |
| --- |

表 8：带数据管道的任务提示。

| ———————————– **affect_ppg_get**：返回特定患者在某一日期或一段时间内的 ppg 数据（如果提供了两个日期）。此工具将返回详细的原始数据，并将其存储在数据管道中。该工具的输入应为一个数据列表，包含以下内容：1- 用户 ID，以字符串形式表示。可以称之为用户、患者、个体等。以 ’par_’ 开头，后跟数字（例如，’par_1’）。 2- 睡眠数据的起始日期，字符串格式为：‘%Y-%m-%d’。 3- 睡眠数据的结束日期，字符串格式为：‘%Y-%m-%d’。如果没有结束日期，则该值应为空字符串（即，""）。此工具将返回以下数据：- 返回一个包含以下键的 JSON 对象数组：**date（以毫秒为单位）**：纪元格式 **ppg**：为 ppg 值。 **hr（以每分钟心跳数表示）**：为患者的心率。结果将存储在数据管道中。 ———————————– |
| --- |

表 9：当任务结果存储在数据管道中时的示例代码生成。

| # 步骤 1：获取患者 5 在 2020 年 8 月整个期间的 PPG 数据 ppg_data_result = self.execute_task(’affect_ppg_get’, [’par_5’, ’2020-08-01’, ’2020-08-31’]) # 步骤 2：分析从获得的 PPG 数据中提取的 HRV 参数 hrv_analysis_result = self.execute_task(’affect_ppg_analysis’, [ppg_data_result]) # 步骤 3：利用 HRV 分析结果估算患者 5 在 2020 年 8 月期间的压力水平 stress_level_result = self.execute_task(’affect_stress_analysis’, [hrv_analysis_result]) |
| --- |
