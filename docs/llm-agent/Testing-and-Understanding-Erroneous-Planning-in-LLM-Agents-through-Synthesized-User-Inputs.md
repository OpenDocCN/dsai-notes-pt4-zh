<!--yml

分类：未分类

日期：2025-01-11 12:40:54

-->

# 通过合成用户输入测试和理解LLM代理中的错误规划

> 来源：[https://arxiv.org/html/2404.17833/](https://arxiv.org/html/2404.17833/)

Zhenlan Ji, Daoyuan Wu, Pingchuan Ma, Zongjie Li, Shuai Wang¹¹footnotemark: 1

香港科技大学

中国香港特别行政区

{zjiae, daoyuan, pmaab, zligo, shuaiw}@cse.ust.hk 对应作者。

###### 摘要

基于大型语言模型（LLM）的代理在通过将LLM与规划、记忆和工具使用等关键模块相结合来解决各种任务方面已经展示了其有效性。越来越多的客户在多种关键商业应用中采用LLM代理，这些应用对可靠性至关重要，包括支持心理健康、化学合成和软件开发。然而，我们的观察和日常使用表明，LLM代理容易出现错误的规划，特别是在任务复杂且需要长期规划时。

在本文中，我们提出了PDoctor，一种新颖且自动化的测试LLM代理并理解其错误规划的方法。作为该方向的首个工作，我们将错误规划的检测形式化为一个约束可满足性问题：如果LLM代理的执行违反了从用户输入中得出的约束，则认为其规划是错误的。为此，PDoctor首先为用户查询定义了一种特定领域语言（DSL），并借助Z3约束求解器合成各种输入。这些合成的输入是自然语言段落，指定了完成一系列任务的要求。然后，PDoctor从这些要求中推导约束，形成测试预言机。PDoctor具有多个设计考虑因素，例如模拟工具和输入变异，以增强测试效果。它的合成输入还可以包含动态约束更新等高级功能，以更好地测试LLM代理的规划能力。我们使用三种主流代理框架和两种强大的LLM（GPT-3.5和GPT-4）评估PDoctor。结果表明，PDoctor能够有效检测代理规划中的多种错误，并提供有价值的见解和错误特征，既对代理开发者（用于改进LLM代理）有价值，也对用户（用于使用当代代理）有帮助。最后，我们讨论了扩展PDoctor的潜在替代设计和方向。

## 1 引言

大型语言模型（LLMs）因其在各种任务中表现出的卓越性能而变得极为流行[[22](https://arxiv.org/html/2404.17833v1#bib.bib22), [12](https://arxiv.org/html/2404.17833v1#bib.bib12), [26](https://arxiv.org/html/2404.17833v1#bib.bib26), [44](https://arxiv.org/html/2404.17833v1#bib.bib44), [21](https://arxiv.org/html/2404.17833v1#bib.bib21), [55](https://arxiv.org/html/2404.17833v1#bib.bib55), [47](https://arxiv.org/html/2404.17833v1#bib.bib47)]，展现了理解、推理和行动的能力，类似于人类的认知和推理。凭借其庞大的参数和训练数据，这些模型在自然语言中的复杂模式识别上展现了高超的技能，推动了逻辑推理[[16](https://arxiv.org/html/2404.17833v1#bib.bib16), [56](https://arxiv.org/html/2404.17833v1#bib.bib56)]、漏洞检测[[47](https://arxiv.org/html/2404.17833v1#bib.bib47), [46](https://arxiv.org/html/2404.17833v1#bib.bib46)]、机器人规划[[11](https://arxiv.org/html/2404.17833v1#bib.bib11), [29](https://arxiv.org/html/2404.17833v1#bib.bib29)]以及因果推理[[31](https://arxiv.org/html/2404.17833v1#bib.bib31), [24](https://arxiv.org/html/2404.17833v1#bib.bib24)]的进展。这推动了LLM代理的开发[[59](https://arxiv.org/html/2404.17833v1#bib.bib59), [43](https://arxiv.org/html/2404.17833v1#bib.bib43), [14](https://arxiv.org/html/2404.17833v1#bib.bib14), [40](https://arxiv.org/html/2404.17833v1#bib.bib40)]，这些代理旨在与外部世界互动并做出决策，以实现复杂目标，利用LLM的认知能力来完成需要规划、记忆和执行一系列行动的任务。这些代理将关键模块与LLM结合，能够出色地处理复杂的用户查询，从过去的互动中学习，并适应新任务。

到目前为止，业界已经在各类高利润甚至是关键任务的应用中日益商业化LLM代理，如医疗健康、个人助手和金融服务[[59](https://arxiv.org/html/2404.17833v1#bib.bib59), [5](https://arxiv.org/html/2404.17833v1#bib.bib5), [7](https://arxiv.org/html/2404.17833v1#bib.bib7), [9](https://arxiv.org/html/2404.17833v1#bib.bib9), [8](https://arxiv.org/html/2404.17833v1#bib.bib8), [2](https://arxiv.org/html/2404.17833v1#bib.bib2), [33](https://arxiv.org/html/2404.17833v1#bib.bib33)]。然而，尽管LLM代理具有巨大的潜力和广泛的热情，它们在规划过程中常常会出现错误，从而导致严重后果。例如，用于管理化学合成过程的LLM代理，如果其计划错误，可能无法生产出所需的化学化合物，而涉及的、可能非常昂贵的化学试剂也已在过程中消耗。这一挑战通过我们的观察和经验得到了凸显，强调了改进LLM代理设计和操作的必要性。规划中的错误，特别是在复杂且长期的任务中，可能导致资源的误用或未能达到预期的结果，从而凸显了开发更可靠、更高效LLM代理的重要性。

本文提出了PDoctor，这是一个用于测试和理解LLM代理规划错误的新框架。我们的方法是完全自动化的，可以用于检测使用不同核心LLM并遵循不同范式的LLM代理中的规划错误（参见第[2](https://arxiv.org/html/2404.17833v1#S2 "2 Preliminary ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")节中的介绍）。作为该方向的首个研究工作，PDoctor将“规划错误”的发生形式化为一个约束满足问题：如果LLM代理的执行违反了由用户输入推导出的约束，则其计划被认为是错误的。这些约束可以通过适度的成本严格检查，从而提供了一种可靠的方式来检测规划错误。

PDoctor定义了一个领域特定语言（DSL），用于捕捉用户查询的语义，并具有合成过程（在Z3的帮助下），能够生成多样化的用户请求作为LLM代理的测试输入。我们提供可配置的参数来控制用户查询的多样性和复杂性，并提供变异过程以进一步转换每个生成的用户查询。每个用户查询表示一段文字，概述了执行一系列任务的要求。PDoctor根据这些要求推导出约束，并检查LLM代理的计划是否符合这些约束的可满足性；如果不符合，则标记为错误规划。我们提供了一组设计考虑因素和优化（例如，代理使用的模拟工具）以实现有效的测试，并通过动态约束更新等高级功能增强合成输入，以测试LLM代理的规划能力。

我们在三个主流LLM代理框架上评估了PDoctor，分别是ReAct [[59](https://arxiv.org/html/2404.17833v1#bib.bib59)]、OpenAI Tools (OT) [[5](https://arxiv.org/html/2404.17833v1#bib.bib5)] 和OpenAI Assistant (OA) [[7](https://arxiv.org/html/2404.17833v1#bib.bib7)]。这些LLM代理代表了不同的范式，广泛应用于各种场景。我们将这些框架与两种广泛使用的LLM模型，GPT-3.5和GPT-4 [[10](https://arxiv.org/html/2404.17833v1#bib.bib10)] 结合使用。PDoctor能够在所有这些设置中有效地检测到成千上万的错误规划。我们配置PDoctor来描绘不同LLM代理的规划能力“上限”，并总结检测到的错误规划。这些结果可以为LLM代理的常见陷阱提供洞见，并为开发者和用户的日常实践提供指导。我们还讨论了PDoctor的扩展以及未来修复检测到的错误规划的潜在工作。总之，本文的贡献如下：

+   •

    我们率先开展了对LLM代理中错误规划的测试与理解，考虑到它们在对可靠性敏感的领域中的日益商业化以及错误规划可能带来的严重后果。

+   •

    我们将检测错误规划的问题表述为约束满足问题，并提出了一个完全自动化的框架PDoctor，该框架通过输入合成和约束检查来检测错误规划。

+   •

    我们在使用不同范式的主流LLM代理上评估了PDoctor，并展示了我们的方法能够以适度的成本有效地检测成千上万的错误规划。我们还从不同方面总结了洞见，以造福开发者和用户。

工具可用性。我们已将PDoctor发布在[https://anonymous.4open.science/r/PDoctor-E872](https://anonymous.4open.science/r/PDoctor-E872)供审阅使用。我们将继续维护该工具并添加更多文档，以帮助用户使用它。

## 2 初步工作

### 2.1 规划问题

规划是一个基本的问题解决任务，涉及生成一系列动作以实现目标[[34](https://arxiv.org/html/2404.17833v1#bib.bib34), [18](https://arxiv.org/html/2404.17833v1#bib.bib18)]。在这一领域已投入大量努力，并在多个领域取得了显著进展，如机器人技术[[36](https://arxiv.org/html/2404.17833v1#bib.bib36), [13](https://arxiv.org/html/2404.17833v1#bib.bib13), [23](https://arxiv.org/html/2404.17833v1#bib.bib23)]和自动驾驶汽车[[15](https://arxiv.org/html/2404.17833v1#bib.bib15), [32](https://arxiv.org/html/2404.17833v1#bib.bib32)]。形式上，规划问题可以定义如下：

###### 定义1（规划问题）。

给定一组状态$\bm{S}$，一组动作$\bm{A}$，以及一个状态转换函数$f:\bm{S}\times\bm{A}\to\bm{S}$，规划问题$\mathcal{P}$被定义为一个元组$\langle\bm{S},\bm{A},f,s_{0},s_{g}\rangle$，其中$s_{0}\in\bm{S}$是初始状态，$s_{g}\in\bm{S}$是目标状态。$\mathcal{P}$的目标是找到一个动作序列$\langle a_{1},a_{2},\ldots,a_{n}\rangle$，称为计划，使得$f(f(\ldots f(s_{0},a_{1}),a_{2}),\ldots,a_{n})=s_{g}$。

在LLM代理的背景下，用户查询可以视为一个规划问题$\mathcal{P}=\langle\bm{S},\bm{A},f,s_{0},s_{g}\rangle$。同样，调用一个特定工具可以视为采取一个动作$a$，其中$a\in\bm{A}$。LLM代理的目标是生成一个正确的答案，即一系列动作$\langle a_{1},a_{2},\ldots,a_{n}\rangle$，以满足由状态转换函数$f$所暗示的约束集$\bm{C}$。初始状态$s_{0}$表示代理开始处理用户查询，而目标状态$s_{g}$表示任务完成的状态，即用户查询中指定的所有约束都得到了满足。

### 2.2 LLM代理的一个示例

图1：一个示例，展示了一个LLM代理处理复杂用户查询的过程。绿色和蓝色分别表示代理的动作和所调用工具的执行结果。

图 [1](https://arxiv.org/html/2404.17833v1#S2.F1 "图 1 ‣ 2.2 LLM 代理示例 ‣ 2 初步 ‣ 通过合成用户输入测试和理解 LLM 代理中的错误规划") 展示了一个典型的 LLM 代理在处理用户查询示例时的工作流程（图 [1](https://arxiv.org/html/2404.17833v1#S2.F1 "图 1 ‣ 2.2 LLM 代理示例 ‣ 2 初步 ‣ 通过合成用户输入测试和理解 LLM 代理中的错误规划")(a)）。该代理首先理解用户查询中隐含的需求，然后规划（推理）选择合适的工具（图 [1](https://arxiv.org/html/2404.17833v1#S2.F1 "图 1 ‣ 2.2 LLM 代理示例 ‣ 2 初步 ‣ 通过合成用户输入测试和理解 LLM 代理中的错误规划")(b)）来调用，最后执行每个选定工具并收集其响应（图 [1](https://arxiv.org/html/2404.17833v1#S2.F1 "图 1 ‣ 2.2 LLM 代理示例 ‣ 2 初步 ‣ 通过合成用户输入测试和理解 LLM 代理中的错误规划")(c)）。特别地，在接收到每个调用工具的输出后，代理会重复上述过程，直到用户查询得到完全回答。这一系列代理与环境之间的互动，如图 [1](https://arxiv.org/html/2404.17833v1#S2.F1 "图 1 ‣ 2.2 LLM 代理示例 ‣ 2 初步 ‣ 通过合成用户输入测试和理解 LLM 代理中的错误规划")(c) 中所列，构成了一个问题解决过程，然后确定最终对用户查询的响应（此处省略）。LLM 代理得益于强大的 LLM，使其能够以类人方式理解、推理和同时执行操作，并且这三种能力的协同作用。

然而，考虑到先前研究中已暴露的 LLM 的易感性[[38](https://arxiv.org/html/2404.17833v1#bib.bib38), [53](https://arxiv.org/html/2404.17833v1#bib.bib53), [58](https://arxiv.org/html/2404.17833v1#bib.bib58), [27](https://arxiv.org/html/2404.17833v1#bib.bib27)]，LLM 代理容易出现错误和故障并不足为奇。直观地看，代理的行动链可以被视为一个“错误放大器”，其中在行动链的早期阶段出现的小错误会在后续每一步中不断放大并传播，最终导致灾难性的失败。这一特点进一步加剧了为用户查询提供正确和稳定响应的难度。此外，LLM 代理与环境之间的互动往往复杂且动态，与 LLM 能力已被测试的静态和孤立环境不同[[49](https://arxiv.org/html/2404.17833v1#bib.bib49), [26](https://arxiv.org/html/2404.17833v1#bib.bib26), [22](https://arxiv.org/html/2404.17833v1#bib.bib22), [46](https://arxiv.org/html/2404.17833v1#bib.bib46)]。

一般来说，以往评估大语言模型（LLMs）的研究通常局限于遵循“输入文本和输出文本”的简单线性流程。在这种范式中，问题会以整体形式呈现，LLMs预期会基于给定的问题生成文本回应。与此相对，大语言模型代理的测试则需要关注LLM与环境之间的互动与相互作用，其中问题是动态的，LLM代理需要根据环境反馈调整其计划。这里的反馈指的是所调用工具的结果，如图[1](https://arxiv.org/html/2404.17833v1#S2.F1 "Figure 1 ‣ 2.2 An Example of LLM Agents ‣ 2 Preliminary ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")(c)中的蓝色文本所示。根据这些动态和复杂的设置，我们设计了PDoctor，以动态更新约束集（具体细节见第[4.4](https://arxiv.org/html/2404.17833v1#S4.SS4 "4.4 Extended Testing Framework ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")节）。这突显了工具设计的重要性，除了文本问题外，在测试用例生成过程中也需要考虑这一点。因此，开发一种系统的方法，专门针对LLM代理的独特特性，以全面测试和评估其性能是至关重要的。

### 2.3 LLM代理的架构

在第[2.2](https://arxiv.org/html/2404.17833v1#S2.SS2 "2.2 An Example of LLM Agents ‣ 2 Preliminary ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")节中展示了LLM代理的典型例子后，我们将深入探讨LLM代理的架构，以更好地理解代理操作背后的机制，这对设计测试框架具有重要价值。

图2：根据[[54](https://arxiv.org/html/2404.17833v1#bib.bib54)]，LLM代理的典型架构。

模块。图[2](https://arxiv.org/html/2404.17833v1#S2.F2 "Figure 2 ‣ 2.3 Architecture of LLM Agents ‣ 2 Preliminary ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")展示了一个典型的LLM代理架构。该代理由三个主要组件组成：（1）理解模块，（2）计划模块，以及（3）执行（工具使用）模块。理解模块负责理解用户查询和一些历史记录（即在“记忆”组件中）以提取需要完成的具体任务。计划模块负责：（1）将提取的任务分解成一系列可以通过调用各种工具来完成的子任务；（2）管理状态和资源以支持合适的计划（例如，保持错误代码，直到被network_diagnose请求，如图[1](https://arxiv.org/html/2404.17833v1#S2.F1 "Figure 1 ‣ 2.2 An Example of LLM Agents ‣ 2 Preliminary ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")所示）；（3）确定子任务的执行顺序。执行模块随后根据生成的计划调用工具并收集工具调用的结果。

LLM 和提示。所有上述组件都由LLM实现，LLM充当代理的“大脑”。代理提供了由查询上下文、代理应如何响应以及可用工具组成的提示模板，而用户则提供查询内容。生成的提示将用于指导LLM生成响应。例如，ReAct [[59](https://arxiv.org/html/2404.17833v1#bib.bib59)]将LLM的核心响应分为三种类型：观察、思考和行动，这三种类型分别对应图[2](https://arxiv.org/html/2404.17833v1#S2.F2 "Figure 2 ‣ 2.3 Architecture of LLM Agents ‣ 2 Preliminary ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")中的理解、计划和执行模块。此外，像角色扮演这样的提示工程策略通常被用来提升代理在特定场景下的表现，例如通过OpenAI的助手API中的指令参数[[7](https://arxiv.org/html/2404.17833v1#bib.bib7)]。

### 2.4 我们的测试重点

如图[2](https://arxiv.org/html/2404.17833v1#S2.F2 "Figure 2 ‣ 2.3 Architecture of LLM Agents ‣ 2 Preliminary ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")所示，不同组件共同作用于代理的整体性能。然而，本文认为测试计划模块是重中之重。这是因为大量工作[[25](https://arxiv.org/html/2404.17833v1#bib.bib25), [19](https://arxiv.org/html/2404.17833v1#bib.bib19), [62](https://arxiv.org/html/2404.17833v1#bib.bib62), [46](https://arxiv.org/html/2404.17833v1#bib.bib46), [51](https://arxiv.org/html/2404.17833v1#bib.bib51), [28](https://arxiv.org/html/2404.17833v1#bib.bib28)]已经投入到评估理解模块，该模块依赖于LLMs的基本自然语言理解能力。同样，行动模块也取得了令人鼓舞的进展，正如OpenAI最近将函数调用功能整合到GPT模型中的研究所证明的那样[[6](https://arxiv.org/html/2404.17833v1#bib.bib6)]。此外，Yue等人[[20](https://arxiv.org/html/2404.17833v1#bib.bib20)]对LLM代理的工具使用性能进行了深入研究，并提出了一个基准MetaTool，用于评估行动模块。相比之下，计划模块决定如何将代理的思维具体化为一系列可以由代理执行的动作，在现有的研究中尚未得到系统的研究或测试。总结来说，我们的关注点包括：

+   •

    （仅隔离规划模块的测试）在保持规划问题复杂性的同时，我们力求简化由理解和行动模块处理的用户查询（见图[2](https://arxiv.org/html/2404.17833v1#S2.F2 "Figure 2 ‣ 2.3 Architecture of LLM Agents ‣ 2 Preliminary ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")）。通过这样做，我们可以确保任何检测到的代理失败都可以归因于规划模块中的错误，从而实现对规划模块的模拟“隔离”。这种方法有助于更精确地测试LLM代理的规划能力。

+   •

    （涵盖文本查询和底层工具）与之前的研究[[50](https://arxiv.org/html/2404.17833v1#bib.bib50), [49](https://arxiv.org/html/2404.17833v1#bib.bib49)]不同，后者通过生成和评估文本计划来测试大型语言模型（LLMs）的规划能力，LLM代理中的规划模块需要决定工具调用的顺序，这使得规划模块的测试变得更加复杂。我们的目标是生成包含文本查询和相应工具的专门测试用例。

+   •

    （仅使用有效的用户查询进行测试）对于单个查询，我们专注于生成有效的用户查询来测试LLM代理。也就是说，我们不会使用极端的用户查询来施压LLM代理。根据我们的观察，极端查询，例如过长或过于复杂的查询，以及那些内容破碎或令人困惑的查询，可能会阻碍LLM代理生成有意义的输出。这并不令人惊讶，因为LLM代理依赖LLM生成输出，因此对输入提示的质量非常敏感。坚持使用有效的用户查询可以帮助我们更好地理解规划模块在现实环境中的表现。

## 3 总体概述

图3：PDoctor的整体工作流程，它合成文本用户查询与工具列表，并推导约束作为神谕来测试LLM代理。

根据第[2.4节](https://arxiv.org/html/2404.17833v1#S2.SS4 "2.4 我们的测试重点 ‣ 2 初步 ‣ 通过合成用户输入测试和理解LLM代理中的错误规划")中描述的测试重点，我们现在介绍PDoctor，一个用于测试和理解LLM代理中错误规划的创新系统。PDoctor的基本思路是合成文本形式的用户查询，并基于同时推导出的形式化约束，我们可以识别出错误的规划。基于这一思路，我们展示了PDoctor的整体设计，如图[3](https://arxiv.org/html/2404.17833v1#S3.F3 "图3 ‣ 3 总体概述 ‣ 通过合成用户输入测试和理解LLM代理中的错误规划")所示。从高层次来看，PDoctor生成包含表示规划问题的文本形式用户查询的测试用例，并附带一系列可供LLM代理调用的工具。测试神谕是通过合成的查询自动推导出来，用于检查LLM代理规划的正确性。这个神谕还帮助我们在发生错误规划时进行剖析和表征。具体而言，PDoctor包含以下关键组件：

① 用户查询生成。PDoctor执行自然语言合成过程生成文本用户查询，将其视为一个规划问题$\mathcal{P}$，如第[2.1](https://arxiv.org/html/2404.17833v1#S2.SS1 "2.1 Planning Problem ‣ 2 Preliminary ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")节中所述，针对LLM代理。该过程包括两个步骤：骨架合成和文本填充。前者生成用户查询的骨架，这是规划问题的高级抽象表示。这使得PDoctor能够直接推导出后续测试的约束条件。后者步骤则通过文本填充这个骨架，形成一个完整的用户查询，描述解决问题的需求。该合成过程的设计受到两个关键考虑因素的驱动：（1）推导出的约束应当等同于生成查询的语义，因为这些约束将作为测试LLM代理的oracle；（2）合成过程应当具有灵活性和可扩展性，允许用户指定生成查询的复杂度和场景。针对第一个考虑，生成用户查询的其他方法，如文本变异和基于LLM的文本生成，已在第[7](https://arxiv.org/html/2404.17833v1#S7 "7 Discussion ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")节中进一步讨论。

② 测试结果检查。在从合成过程推导出正式约束后，PDoctor利用这些约束来检测LLM代理的错误规划。具体来说，PDoctor将每个动作（对应于用户查询中指定的子任务，采用一对一的方式）映射到一个特定的工具调用，并通过记录工具调用历史来收集动作链（即LLM代理所做的规划）。然后将收集到的规划与约束进行比较，以确定该规划是否正确。

③ 错误剖析。与之前基于表面自然语言属性（例如，单词级替换）修改输入的研究[[48](https://arxiv.org/html/2404.17833v1#bib.bib48), [30](https://arxiv.org/html/2404.17833v1#bib.bib30)]不同，PDoctor从头开始合成完整的文本用户查询，完全控制用户查询的语义（即推导出的约束）和结构（即骨架）。这一设计使PDoctor能够对生成的查询进行全面的变异和转换。通过保持整体语义不变，PDoctor能够修改单词、子任务、提示的上下文、多个句子，甚至是整个提示的结构。这一能力对于剖析LLM代理中的错误规划至关重要，因为它使PDoctor能够通过比较原始和变异提示的规划结果，精确找出失败的根本原因。

挑战与关键解决方案。然而，为了实现上述设计，我们需要解决一些独特的挑战：

C1:

设计一种机制或规范来支持用户查询合成。如果没有机制或规范来指导其制定，合成的用户查询可能会变得高度多样且无法控制。此外，为了能够轻松地从合成的查询中推导出相应的约束，我们还需要正式规范的支持。为了解决这个问题，我们在第[4.1](https://arxiv.org/html/2404.17833v1#S4.SS1 "4.1 Domain-Specific Language ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")节中提出了一种专门的领域特定语言（DSL）。通过这套DSL，PDoctor能够覆盖各种具有可配置复杂性和多样性的用户查询。此外，基于DSL的骨架合成确保了对用户查询的结构和语义的完全控制，使PDoctor能够对给定的提示进行全面的变异和转换，从而能够在第[4.5](https://arxiv.org/html/2404.17833v1#S4.SS5 "4.5 Error Dissection ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")节中对错误的规划进行剖析。

C2:

生成语法上有效且语义上有意义的用户查询。如在第[2.4节](https://arxiv.org/html/2404.17833v1#S2.SS4 "2.4 Our Testing Focus ‣ 2 Preliminary ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")中提到的，我们的目标是生成有效的用户查询来测试LLM代理。因此，PDoctor应专注于生成语法上有效且语义上有意义的用户查询作为测试输入。为了解决这个挑战，用户查询骨架通过上述的DSL进行指定，这可以防止生成无效内容；详情请见第[4.2节](https://arxiv.org/html/2404.17833v1#S4.SS2 "4.2 User Query Synthesis ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")。此外，使用约束求解器（在我们的案例中是Z3[[17](https://arxiv.org/html/2404.17833v1#bib.bib17)]）来确保用户查询的语义是有意义的，即从生成的提示中推导出的约束始终是可满足的，如第[4.3节](https://arxiv.org/html/2404.17833v1#S4.SS3 "4.3 Test Results Check ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")中所示。

C3:

涵盖具有状态管理和动态问题解决的复杂规划问题。尽管上面提到的PDoctor的原始设计已经涵盖了文本查询及其相应工具的规划测试，但它尚未考虑涉及状态管理和动态问题解决的复杂规划问题。我们通过引入时间和持续时间约束来解决这一挑战，从而获得一个扩展的PDoctor测试框架，相关内容将在第[4.4节](https://arxiv.org/html/2404.17833v1#S4.SS4 "4.4 Extended Testing Framework ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")中介绍。

## 4 设计

### 4.1 域特定语言

为了支持用户查询合成，我们设计了一个专门的DSL，专门针对LLM代理规划问题。这个DSL并不是为了覆盖自然语言中描述任何可以想象的概念的无限语义空间。相反，它将焦点集中在第[2.1节](https://arxiv.org/html/2404.17833v1#S2.SS1 "2.1 Planning Problem ‣ 2 Preliminary ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")中定义的规划问题所需的语义空间。因此，在深入探讨DSL的具体细节之前，我们首先展示如何简化要探索的语义空间，这也有助于避免由于英语并非上下文无关语言而产生的歧义。

语义简化。对于两个不能并行执行的任务，分别表示为任务 $A$ 和任务 $B$，规划问题可以简化为确定 $A\prec B$（任务 $A$ 在任务 $B$ 之前执行），或 $A\succ B$（任务 $A$ 在任务 $B$ 之后执行）。因此，约束可以简化为任务执行顺序的特定描述。例如，“$A$ 应该在 $B$ 之前执行”就是在这种情况下的简化约束。此外，将这种简化推广到多个任务是直接的。因此，PDoctor 将提示合成简化为生成几句话，每句话由这种简化的约束组成。

此外，用户查询中的每个“任务”都简化为执行一个动作 $a$。这里 $a\in\mathcal{A}$，而 $\mathcal{A}$ 是 LLM 代理可以执行的动作集合（参见第[2.1节](https://arxiv.org/html/2404.17833v1#S2.SS1 "2.1 Planning Problem ‣ 2 Preliminary ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")）。以图[3](https://arxiv.org/html/2404.17833v1#S3.F3 "Figure 3 ‣ 3 Overview ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")为例，进行网络状态检查是一个任务，而调用 network_status_check() 是该任务的动作。在不需要额外分解的情况下，每个任务都可以通过一个简单的动作直接完成。这种简化基于这样一个观察：任务的分解主要由任务的上下文决定，而不是 LLM 代理本身的表现。例如，熟悉网络的 LLM 代理能够正确地将“修复断开连接的网络”这一任务分解为一系列子任务，如“检查网络状态”和“网络诊断”，而那些专为处理图书管理而设计的 LLM 代理则无法做到这一点。然而，这种任务分解失败可以通过为 LLM 代理提供网络修复的指导知识轻松缓解。由于任务分解可能对 LLM 代理的性能产生预期外的影响，PDoctor 避免生成需要分解的复杂任务。

语法 $\displaystyle\langle\mathcal{P}\in\text{规划问题}\rangle\Coloneqq{}$ $\displaystyle S^{+}$ $\displaystyle\langle S\in\text{句子}\rangle\Coloneqq{}$ $\displaystyle s\mid S\;j\;s$ $\displaystyle\langle s\in\text{子句}\rangle\Coloneqq{}$ $\displaystyle c_{i}\mid m$ $\displaystyle\langle c_{i}\in\text{独立子句}\rangle\Coloneqq{}$ $\displaystyle s\;({\color[rgb]{.5,0,.5}\definecolor[named]{pgfstrokecolor}{rgb}{.5,0,.5}\texttt{VP}}_{\prec}\mid{\color[rgb]{.5,0,.5}\definecolor[named]{% pgfstrokecolor}{rgb}{.5,0,.5}\texttt{VP}}_{\succ})\;o$ $\displaystyle{}\mid{}$ $\displaystyle s\;{\color[rgb]{.5,0,.5}\definecolor[named]{pgfstrokecolor}{rgb}{.5,0,.5}\texttt{VP}}_{\emptyset}\;({\color[rgb]{.5,0,.5}\definecolor[named]{% pgfstrokecolor}{rgb}{.5,0,.5}\texttt{P}}_{\prec}\mid{\color[rgb]{.5,0,.5}% \definecolor[named]{pgfstrokecolor}{rgb}{.5,0,.5}\texttt{P}}_{\succ})\;o$ $\displaystyle{}\mid{}$ $\displaystyle({\color[rgb]{.5,0,.5}\definecolor[named]{pgfstrokecolor}{rgb}{% .5,0,.5}\texttt{P}}_{\prec}\mid{\color[rgb]{.5,0,.5}\definecolor[named]{% pgfstrokecolor}{rgb}{.5,0,.5}\texttt{P}}_{\succ})\;o\;\text{``，''}\;s\;{\color% [rgb]{.5,0,.5}\definecolor[named]{pgfstrokecolor}{rgb}{.5,0,.5}\texttt{VP}}_{\emptyset}$ $\displaystyle\langle m\in\text{多子句}\rangle\Coloneqq{}$ $\displaystyle c_{d}\;({\color[rgb]{.5,0,.5}\definecolor[named]{pgfstrokecolor}% {rgb}{.5,0,.5}\texttt{SC}}_{\prec}\mid{\color[rgb]{.5,0,.5}\definecolor[named]{% pgfstrokecolor}{rgb}{.5,0,.5}\texttt{SC}}_{\succ})\;c^{\prime}_{d}$ $\displaystyle{}\mid{}$ $\displaystyle({\color[rgb]{.5,0,.5}\definecolor[named]{pgfstrokecolor}{rgb}{% .5,0,.5}\texttt{SC}}_{\prec}\mid{\color[rgb]{.5,0,.5}\definecolor[named]{% pgfstrokecolor}{rgb}{.5,0,.5}\texttt{SC}}_{\succ})\;c^{\prime}_{d}\;\text{``，'% '}\;c_{d}$ $\displaystyle\langle c_{d}\in\text{从属子句}\rangle\Coloneqq{}$ $\displaystyle s\;{\color[rgb]{.5,0,.5}\definecolor[named]{pgfstrokecolor}{rgb}% {.5,0,.5}\texttt{VP}}_{\emptyset}$ $\displaystyle\langle c_{r}\in\text{关系子句}\rangle\Coloneqq{}$ $\displaystyle\text{``which''}\;({\color[rgb]{.5,0,.5}\definecolor[named]{% pgfstrokecolor}{rgb}{.5,0,.5}\texttt{VP}}_{\prec}\mid{\color[rgb]{.5,0,.5}% \definecolor[named]{pgfstrokecolor}{rgb}{.5,0,.5}\texttt{VP}}_{\succ})\;o$ $\displaystyle{}\mid{}$ $\displaystyle\text{``which''}\;{\color[rgb]{.5,0,.5}\definecolor[named]{% pgfstrokecolor}{rgb}{.5,0,.5}\texttt{VP}}_{\emptyset}\;({\color[rgb]{.5,0,.5}% \definecolor[named]{pgfstrokecolor}{rgb}{.5,0,.5}\texttt{P}}_{\prec}\mid{% \color[rgb]{.5,0,.5}\definecolor[named]{pgfstrokecolor}{rgb}{.5,0,.5}\texttt{P% }}_{\succ})\;o$ $\displaystyle\langle s\in\text{主语}\rangle\Coloneqq{}$ $\displaystyle a^{+}$ $\displaystyle{}\mid{}$ $\displaystyle a^{+}\;\text{``，''}\;c_{r}\;\text{``，''}$ $\displaystyle\langle o\in\text{宾语}\rangle\Coloneqq{}$ $\displaystyle a^{+}$ $\displaystyle{}\mid{}$ $\displaystyle a^{+}\;\text{``，''}\;c_{r}\;\text{``，''}$ $\displaystyle\langle j\in\text{连接词}\rangle\Coloneqq{}$ “;” $\displaystyle{}\mid{}$ $\displaystyle\text{``，''}\;(\text{``and''}\mid\text{``but''}\mid\text{``yet''})$ $\displaystyle{}\mid{}$ $\displaystyle\text{``，''}\;(\text{``while''}\mid\text{``whereas''})$ $\displaystyle\langle a\in\text{动作}\rangle\Coloneqq{}$ $\displaystyle\text{动作集，}\mathcal{A}$

图 4：我们设计的 DSL 语法，专门用于合成用户查询并推导其约束。

DSL 细节。图 [4](https://arxiv.org/html/2404.17833v1#S4.F4 "Figure 4 ‣ 4.1 Domain-Specific Language ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs") 展示了这个领域特定语言的语法。总体来说，每个用户查询表示为一个段落 $\mathcal{P}$。LLM 代理需要执行的每个任务都在这个段落中详细说明；每个任务要求 LLM 代理采取一个特定的动作 $a$。段落由一个或多个句子 $S$ 组成。每个句子 $S$ 是由子句 $s$ 组成的序列，这些子句由从属连词 $j$ 分隔。一个子句可以视为一个独立的语义单元，包含一个或多个约束，要求 LLM 代理遵守。特别地，一个子句 $s$，无论它是一个独立子句 $c_{i}$，还是一个多子句 $m$，都包含一个主语 $s$、一个宾语 $o$，以及一个或多个紫色的关键词，这些关键词直接指示序列关系。这些关键词采用特殊的表示方式，其中大写字母代表它们的词性标签（POS），下标表示序列关系。具体来说，VP 表示动词短语，P 表示介词，SC 表示顺序连词。在这里，“顺序连词”是一种特殊类型的连词，用来指示两个从属子句之间的时间关系。对于下标，我们使用 $\prec$ 表示主句中的事件发生在宾句中的事件之前，而 $\succ$ 则表示相反的情况。$\emptyset$ 是一个特殊情况，它仅用于装饰动词短语 VP，表示动词短语与任何时间关系无关，如“发生”或“出现”。

### 4.2 用户查询合成

在第 [4.1](https://arxiv.org/html/2404.17833v1#S4.SS1 "4.1 Domain-Specific Language ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs") 节中设计的 DSL 调节了自然语言的复杂性，同时保持了它生成的用户查询的表达力和多样性。一般来说，合成用户查询是逐步扩展一个抽象语法树（AST），其中每个 AST 节点都是根据 DSL 语法从一组有效节点中随机选择的。也就是说，合成过程是自上而下进行的，根节点是段落 $P$，叶节点是一系列 DSL 中的非终结符号。忽略 $a^{+}$ 的扩展，DSL 提供了总共 340 种可能的扩展选项，确保了合成用户查询的多样性。

基于这个精心设计的领域特定语言（DSL），用户查询的合成主要包括三个步骤：合成骨架、将骨架翻译成自然语言（NL）提示，并同时从合成的骨架中推导约束，如图[3](https://arxiv.org/html/2404.17833v1#S3.F3 "图 3 ‣ 3 概览 ‣ 通过合成的用户输入测试和理解LLM代理中的错误规划")所示。

输入：动作集合 $\bm{A}=\{a_{1},a_{2},...,a_{n}\}$，最大句子数 $N$，最大迭代次数 $K$ 输出：生成的段落 $P$，约束集合 $\bm{C}$12$P\leftarrow\emptyset$$\bm{C}\leftarrow initConstraints(\bm{A})$ // 初始化约束。3 对于 *$n\in\{1,...,N\}$* 进行迭代       /* 迭代生成句子以形成段落。 */4       $\texttt{iter}\leftarrow 0$5       当 *$\texttt{iter}<K$* 时6             $s\leftarrow genSentence(\bm{A},\bm{C})$7             $\bm{C}^{\prime}\leftarrow\bm{C}\cup getConstraints(s)$8             如果 *$SATSolver(\bm{C}^{\prime})==\text{SAT}$* 则                 /* 在添加新句子后验证约束的可满足性。 */9                   $P\leftarrow P\cup s$10                   $\bm{C}\leftarrow\bm{C}^{\prime}$11                   跳出循环12             结束如果13            $\texttt{iter}\leftarrow\texttt{iter}+1$14            15       结束当循环16      如果 *$\texttt{iter}\geq K$* 则17             跳出循环18       结束如果19      20 结束对循环的迭代返回 *$P,C$*

算法 1 骨架合成。

骨架合成。算法[1](https://arxiv.org/html/2404.17833v1#alg1 "在 4.2 用户查询合成 ‣ 4 设计 ‣ 通过合成用户输入测试和理解LLM代理中的错误规划")介绍了提示骨架合成的算法过程。该算法以动作集合$\bm{A}$、最大句子数$N$和最大迭代次数$K$作为输入，并输出生成的段落$P$及其对应的约束集$\bm{C}$。一般来说，对于每次句子生成，算法通过根据DSL随机扩展句子$S$并将非终结符替换为自然语言单词（第6行）来迭代生成句子。对于骨架中的动作符号，它们会被替换为来自$\bm{A}$的动作。然后，推导出该句子的约束并将其用于扩展约束集$\bm{C}$（第7行）。如果扩展后的约束集$\bm{C}^{\prime}$是可满足的，句子将被添加到段落$P$中，并且约束集$\bm{C}$将得到更新（第8–11行）。否则，算法将通过迭代该过程来尝试生成另一个句子，直到达到最大迭代次数$K$，此时生成过程将终止，因为这表明找到可满足的句子是不可行的（第12–17行）。值得注意的是，该算法确保生成的用户查询必须是可满足的，因为在每次句子生成后都会检查约束。这样，合成的用户查询可以有效地用于测试LLM代理的规划能力。

文本填充。为了将骨架“翻译”成自然语言用户查询，PDoctor 会将文本填充到骨架中由终结符号（例如，各种关键词和动作）标记的插槽中。除了动作符号外，平均每个终结符号有七个可选的自然语言词汇。例如，“happen”（发生）、“occur”（发生）、“be executed”（被执行）等，适用于动词短语 ${\color[rgb]{.5,0,.5}\definecolor[named]{pgfstrokecolor}{rgb}{.5,0,.5}\texttt{% VP}}_{\emptyset}$，以及“after”（之后）、“behind”（后面）、“later than”（晚于）等，适用于介词 ${\color[rgb]{.5,0,.5}\definecolor[named]{pgfstrokecolor}{rgb}{.5,0,.5}\texttt{% P}}_{\succ}$。对于动作符号，我们用各种职业的日常活动来替代它们。更具体地说，我们首先指示LLM提供常见职业的列表（例如，“teacher”（教师）、“software developer”（软件开发者）等），通过查询LLM的提示“Please provide a list of 50 typical jobs. Ensure that these 50 positions span a variety of industries.”（请提供50个典型职业的列表，并确保这些职业涵盖多个行业）。对于每个提供的职业，我们进一步用提示“Please list 20 activities in noun phrase format that a [role] may need to do in a day.”（请列出一个[角色]可能在一天内做的20个名词短语形式的活动），其中“[role]”替换为职业名称。相同段落中的动作符号会用来自同一职业的日常活动来替代，而职业则成为用户查询的主题。通过改变主题，PDoctor能够平滑地改变用户查询的上下文，这对[4.5节](https://arxiv.org/html/2404.17833v1#S4.SS5 "4.5 Error Dissection ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")中的错误剖析有所帮助。文本填充的设计促进了合成用户查询的多样性和真实性，与先前的工作（这些工作局限于少数固定的人工场景或模板）形成对比[[49](https://arxiv.org/html/2404.17833v1#bib.bib49), [42](https://arxiv.org/html/2404.17833v1#bib.bib42)]。

推导约束条件。与此同时，PDoctor 从每个合成的用户查询中推导出约束条件。具体来说，每个动作 $a$ 被表示为一个整数变量 $\mathbf{a}$，表示该动作的执行顺序。然后，两个动作之间的后续关系，用 $a_{1}$ 和 $a_{2}$ 表示，可以用比较形式来表达，如 $\mathbf{a_{1}}<\mathbf{a_{2}}$ 或 $\mathbf{a_{1}}>\mathbf{a_{2}}$。前者表示动作 $a_{1}$ 应该在动作 $a_{2}$ 之前执行，而后者则表示相反的情况。因此，考虑到子句 $s$，例如“网络诊断在网络状态检查之后”，其骨架为 $a_{1}\,{\color[rgb]{.5,0,.5}\definecolor[named]{pgfstrokecolor}{rgb}{.5,0,.5}% \texttt{VP}}_{\succ}\,a_{2}$，从这个子句推导出的约束是 $\mathbf{a_{1}}>\mathbf{a_{2}}$。

### 4.3 测试结果检查

一旦我们将合成的用户查询输入LLM代理，它将生成描述任务规划的文本响应。我们可以通过将从响应中提取的动作序列与从用户查询中推导出的约束进行比较，来检查规划的正确性。值得注意的是，有些人可能主张采用像BLEU得分[[37](https://arxiv.org/html/2404.17833v1#bib.bib37)]和BertScore[[64](https://arxiv.org/html/2404.17833v1#bib.bib64)]这样的度量标准，来评估生成的响应与真实情况之间的相似性。然而，在我们的背景下，这种方法可行性较低，因为（i）这些度量可能存在偏差，或计算成本较高，且（ii）规划测试的有效性应该基于动作序列是否满足约束来评估，而不是生成的响应与真实情况之间的相似性。相比之下，我们将动作序列与约束进行比较的方法，是验证LLM代理规划的更直接和有效的方式。

然而，新的挑战出现了，因为从LLM代理的文本响应中提取动作序列并非易事，这需要对LLM响应的意义有精准的理解。现有的工作通常通过采用少量示例提示[[29](https://arxiv.org/html/2404.17833v1#bib.bib29)，[49](https://arxiv.org/html/2404.17833v1#bib.bib49)]，指示LLM以结构化的格式进行回应。然而，我们认为这种做法仍然不理想，因为LLM可能未能遵循格式要求。例如，LLM可能生成的响应并不像预期的那样结构化，或者响应中可能包含无关信息，从而使得提取过程复杂化。

图 5：一个示例，说明我们提取LLM代理动作序列的“模拟测试”方法。绿色斜体表示合成部分，描述了执行一系列任务的约束条件。

模拟工具设计。PDoctor 采用“模拟测试”方法来提取 LLM 代理的行动序列。具体来说，对于合成的用户查询中的每个行动 $a$，PDoctor 创建一个“模拟”工具，工具的名称和描述与行动的文本内容保持一致（例如，图 [5](https://arxiv.org/html/2404.17833v1#S4.F5 "图 5 ‣ 4.3 测试结果检查 ‣ 4 设计 ‣ 通过合成用户输入测试和理解 LLM 代理中的错误规划") 中的“网络诊断”）。该工具的内部逻辑被设计为一个简单的文件写入模块，直接将相应行动 $a$ 的 ID 写入日志文件。该设计基于这样的事实：LLM 代理将工具视为黑箱：它们根据工具的名称和描述来选择工具，而不是根据其内部实现。因此，$a$ 所描述的字面任务并未在环境中执行，工具仅返回一个字符串，欺骗 LLM 代理相信该行动已经完成。图 [5](https://arxiv.org/html/2404.17833v1#S4.F5 "图 5 ‣ 4.3 测试结果检查 ‣ 4 设计 ‣ 通过合成用户输入测试和理解 LLM 代理中的错误规划") 说明了这种设计的一个示例。对于图 [5](https://arxiv.org/html/2404.17833v1#S4.F5 "图 5 ‣ 4.3 测试结果检查 ‣ 4 设计 ‣ 通过合成用户输入测试和理解 LLM 代理中的错误规划")（a）中呈现的合成用户查询，PDoctor 创建了一组与用户查询中的行动相对应的工具，其中图 [5](https://arxiv.org/html/2404.17833v1#S4.F5 "图 5 ‣ 4.3 测试结果检查 ‣ 4 设计 ‣ 通过合成用户输入测试和理解 LLM 代理中的错误规划")（b）展示了其中一个工具。network_diagnosis 并未在环境中执行实际的网络诊断。相反，它仅将工具的 ID，“a1”，写入日志文件，并欺骗 LLM 代理相信网络已经被诊断。

结果检查。代理处理完合成的用户查询后，读取日志文件以收集动作序列。以图[3](https://arxiv.org/html/2404.17833v1#S3.F3 "Figure 3 ‣ 3 Overview ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")中的案例为例，动作序列为“[$a_{1}$, $a_{2}$, $a_{3}$]”。然后，对于动作序列中的每个动作$a$，将相应的变量$\mathbf{a}$赋值为该动作的执行顺序，即$\mathbf{a_{1}}=1$，$\mathbf{a_{2}}=3$，$\mathbf{a_{3}}=2$。之后，如果存在任何约束未被分配的变量值满足，则该计划被视为错误。仍以图[3](https://arxiv.org/html/2404.17833v1#S3.F3 "Figure 3 ‣ 3 Overview ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")中的例子为例，检查变量$\mathbf{a_{1}}$、$\mathbf{a_{2}}$和$\mathbf{a_{3}}$的分配值与约束集$\{\mathbf{a_{1}}<\mathbf{a_{2}}$，$\mathbf{a_{2}}> \mathbf{a_{3}}\}$的逻辑和（$\land$）的关系。由于没有约束（及其$\land$）未得到满足，因此该计划被判定为正确。总体而言，我们将这种检查设计与模拟工具的集成视为一种新颖的方法，能够严格检查LLM代理的规划，同时避免理解LLM文本响应的复杂性。

### 4.4 扩展测试框架

语法

|  | $\displaystyle\langle o\in\text{Object}\rangle\Coloneqq{}$ | $\displaystyle t$ |  |
| --- | --- | --- | --- |
|  | $\displaystyle\langle t\in\text{Time}\rangle\Coloneqq{}$ | $\displaystyle\text{时间点集合，}\mathcal{T}$ |  |

图6：用于涵盖时间和持续时间约束的DSL扩展。

图7：一个例子，展示了生成的测试用例的扩展版本，包括用户查询和相应的工具。与图[5](https://arxiv.org/html/2404.17833v1#S4.F5 "Figure 5 ‣ 4.3 Test Results Check ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")类似，绿色斜体表示合成部分。新增的指令用棕色高亮显示。子图(c)展示了动作ID与动作名称之间的映射。子图(d)和(e)分别呈现了来自用户查询的基本约束和附加约束。

尽管上述测试框架在评估LLM代理的规划能力方面是有效的，但现实世界中的场景将更加复杂和具有挑战性。参考图[1](https://arxiv.org/html/2404.17833v1#S2.F1 "图 1 ‣ 2.2 LLM代理示例 ‣ 2 初步 ‣ 通过合成用户输入测试和理解LLM代理中的规划错误")中的示例，代理预期能够根据工具的执行结果动态调整其规划。此外，一些工具可能需要多个特定输入，比如本示例中的error_code，这要求LLM代理在不同任务之间保留状态信息。基于这些观察，我们通过将时间和工具持续时间概念引入代理规划中来扩展测试框架。一方面，时间约束在现实世界的规划中被广泛使用，比如日常安排。另一方面，引入时间概念使得PDoctor能够模拟全球资源上的动态规划，这是现实世界场景中的一个典型复杂规划问题。特别地，合成的用户查询被扩展为包括新的约束，调节给定任务的开始或结束时间，如图[6](https://arxiv.org/html/2404.17833v1#S4.F6 "图 6 ‣ 4.4 扩展测试框架 ‣ 4 设计 ‣ 通过合成用户输入测试和理解LLM代理中的规划错误")所示。因此，虚拟工具被修改为需要将开始时间点作为输入参数，并返回其执行的持续时间。图[7](https://arxiv.org/html/2404.17833v1#S4.F7 "图 7 ‣ 4.4 扩展测试框架 ‣ 4 设计 ‣ 通过合成用户输入测试和理解LLM代理中的规划错误")展示了生成的测试用例扩展版本的示例。

从该图中可以明显看到，用户查询的模板发生了变化。添加了几条新指令，要求LLM代理考虑时间约束。此外，图[7](https://arxiv.org/html/2404.17833v1#S4.F7 "Figure 7 ‣ 4.4 Extended Testing Framework ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")(b)所展示的模拟工具返回值的变化，要求代理动态调整其规划。由于缺乏关于工具执行时间的知识，代理可能会遇到规划被工具执行结果否定的情况。以图[7](https://arxiv.org/html/2404.17833v1#S4.F7 "Figure 7 ‣ 4.4 Extended Testing Framework ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")中的案例为例，如果代理在14:00开始执行network_diagnosis工具，那么该规划将被否定，因为network_diagnosis需要2小时，直到16:00才完成，而后续动作network_speed_test要求在15:00之前执行。在这种情况下，代理被指示进行提前中止并重新规划，如图[7](https://arxiv.org/html/2404.17833v1#S4.F7 "Figure 7 ‣ 4.4 Extended Testing Framework ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")(a)所示。

此外，约束条件的推导也发生了变化。现在，每个动作$a$都与两个整数变量$\mathbf{a}\_{\text{start}}$和$\mathbf{a}\_{\text{end}}$相关联，分别表示动作的开始时间和结束时间。从模拟工具的返回值中推导出了一种新的约束——持续时间约束，用于调节相关动作的持续时间。图[7](https://arxiv.org/html/2404.17833v1#S4.F7 "Figure 7 ‣ 4.4 Extended Testing Framework ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")(d)展示了由测试环境和用户查询中的指令所隐含的基本约束。图[7](https://arxiv.org/html/2404.17833v1#S4.F7 "Figure 7 ‣ 4.4 Extended Testing Framework ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")(e)展示了由合成部分推导出的附加约束，这部分在图中用绿色斜体突出显示。

输入：触发错误规划的用户查询$Q$，工具集$\bm{T}$，约束集：$\bm{C}$，LLM代理A，最大迭代次数$K$输出：错误原因$E$12对于 *$n\in\{1,2,3\}$* 进行　　  /* 确定错误是否以概率方式出现。 */3　　  $\texttt{plan}\leftarrow\textsc{A}(Q,\bm{T})$4　　  如果 *CheckPlan(\texttt{plan},\bm{C})==\text{True}$* 则5　　　　 返回 *“概率”*6　　  结束如果7　　  8 结束对于910对于 *$k\in\{1,...,K\}$* 进行　　  /* 通过替换查询中的终端文本内容来变异用户查询。 */11　　  $Q^{\prime},\bm{T}^{\prime}\leftarrow TerminalSubstitute(Q,\bm{T})$12　　  $\texttt{plan}\leftarrow\textsc{A}(Q^{\prime},\bm{T}^{\prime})$13　　  如果 *CheckPlan(\texttt{plan},\bm{C})==\text{True}$* 则14　　　　 返回 *“终端”*15　　  结束如果16　　  17 结束对于18对于 *$k\in\{1,...,K\}$* 进行　　  /* 通过改变查询的主题来变异用户查询。 */19　　  $Q^{\prime},\bm{T}^{\prime}\leftarrow TopicChange(Q,\bm{T})$20　　  $\texttt{plan}\leftarrow\textsc{A}(Q^{\prime},\bm{T}^{\prime})$21　　  如果 *CheckPlan(\texttt{plan},\bm{C})==\text{True}$* 则22　　　　 返回 *“主题”*23　　  结束如果24　　  25 结束对于26对于 *$k\in\{1,...,K\}$* 进行　　  /* 合成一个新的测试用例，其约束等同于原始约束。 */27　　  $Q^{\prime},\bm{T}^{\prime}\leftarrow QuerySynthesize(\bm{C})$28　　  $\texttt{plan}\leftarrow\textsc{A}(Q^{\prime},\bm{T}^{\prime})$29　　  如果 *CheckPlan(\texttt{plan},\bm{C})==\text{True}$* 则30　　　　 返回 *“结构”*31　　  结束如果32　　  否则33　　　　 返回 *“约束”*34　　  结束如果35　　  36 结束对于37

算法 2 错误剖析。

### 4.5 错误剖析

PDoctor还具有一个错误剖析组件，旨在调查LLM代理中错误规划的原因。回想一下，我们提出的查询合成技术有助于深入理解查询语义以及LLM代理需要满足的约束条件。给定一个触发错误的查询，PDoctor可以故意改变该查询中的一些组件，以剖析触发的错误。

算法[2](https://arxiv.org/html/2404.17833v1#alg2 "在4.4扩展测试框架 ‣ 4设计 ‣ 通过合成的用户输入测试和理解LLM代理中的错误规划")展示了错误剖析算法。一般来说，该算法首先检查错误是否是通过多次运行LLM代理并使用相同的用户查询以概率方式出现的。然后，它通过从低级别的词语替换到高级别的修改来变异用户查询，以识别错误的原因。具体而言，该算法采用三种变异策略：TerminalSubstitute、TopicChange和QuerySynthesize。相应地，错误的原因有五种可能：Probability（概率）、Terminal（终端）、Topic（主题）、Structure（结构）和Constraint（约束）。如果错误是概率性地发生的，错误原因是Probability（第1–6行）。如果错误是由特定单词引起的，并且可以通过替换这些单词来修复，错误原因是Terminal（第7–13行）。如果错误只能通过改变用户查询的主题来纠正，错误原因是Topic（第14–20行）。此外，PDoctor将尝试合成一个新的用户查询，其约束与原始查询相等。如果新查询没有发生错误，原因是Structure；否则，原因是Constraint（第21–30行）。Constraint指的是PDoctor揭示了一组LLM代理更容易出错的特定约束条件。请注意，我们不考虑对一个查询进行多重错误剖析，因为这可能会使整个过程复杂化，难以准确找出错误的真正原因。例如，如果错误是以概率性的方式表现出来的，那么进一步剖析错误就没有太大意义。同样，如果我们已经确定了导致错误的特定单词（即Terminal），那么它就不应该归属于像Topic或Structure这样的更全面的原因。

## 5 实现和实验设置

PDoctor是用Python3实现的，代码大约有2,600行。我们将PDoctor与流行的约束求解器Z3[[17](https://arxiv.org/html/2404.17833v1#bib.bib17)]集成，用于用户查询的合成和变异。此外，所有LLM代理都使用LangChain[[14](https://arxiv.org/html/2404.17833v1#bib.bib14)]实现，LangChain是一个流行的基于Python的框架，用于开发LLM代理。

模型。我们在评估中使用了两种广泛使用的 LLM 模型，GPT-3.5 和 GPT-4，主要有两个原因。首先，作为执行正确规划的代理核心是具有挑战性的，因此需要像 OpenAI 的强大 LLM。其次，OpenAI 的模型是 LLM 应用社区的主流选择，大多数流行的库（如 LangChain）将其作为默认选择。在本文中，除温度参数外，所有 LLM 模型的参数都设置为默认值，温度设置为 $0$，以减轻 LLM 响应的非确定性（OpenAI 助手不支持此参数，因此不包括）。本研究中使用的两种模型版本均设置为 1106，即 gpt-3.5-turbo-1106 和 gpt-4-1106-preview。

LLM 代理框架。我们在三个主流 LLM 代理框架上评估了 PDoctor，包括 ReAct [[59](https://arxiv.org/html/2404.17833v1#bib.bib59)]、OpenAI 工具（OT）[[5](https://arxiv.org/html/2404.17833v1#bib.bib5)] 和 OpenAI 助手（OA）[[7](https://arxiv.org/html/2404.17833v1#bib.bib7)]。这些 LLM 代理的选择旨在涵盖不同的设计选择和范式，包括不同的交互模式（LLM 代理如何与核心 LLM 或集成工具进行交互）、提示模板设计以及深入的定制（例如，LLM 采样参数调优、记忆管理等）。

表 [1](https://arxiv.org/html/2404.17833v1#S5.T1 "表 1 ‣ 5 实现和实验设置 ‣ 通过合成用户输入测试和理解 LLM 代理中的错误规划") 列出了评估的 LLM 代理框架的详细信息。ReAct 是最兼容的代理框架，因为它支持任何可以通过完成操作调用的 LLM。相比之下，OT 和 OA 需要核心 LLM 支持基于聊天的交互，并且经过功能调用能力的调优 [[6](https://arxiv.org/html/2404.17833v1#bib.bib6)]。ReAct 和 OT 都注重灵活性，允许用户深入定制 LLM 代理并使用提示模板引导用户查询。相比之下，OA 设计得更为“傻瓜化”，定制选项较少。这个简单的设计选择使 OA 在公众中获得了相当大的关注，可能意味着 LLM 代理领域的一个新趋势。尽管如此，值得注意的是，PDoctor 并不限于这些 LLM 代理框架，它也可以应用于其他 LLM 代理。

表 1：评估的 LLM 代理的详细信息。

| 工具 | LLM 交互 | 工具交互 | 提示模板？ | 深入定制？ |
| --- | --- | --- | --- | --- |
| ReAct [[59](https://arxiv.org/html/2404.17833v1#bib.bib59)] | 完成 | 少量提示 | ✓ | ✓ |
| OT [[5](https://arxiv.org/html/2404.17833v1#bib.bib5)] | 聊天 | 功能调用 | ✓ | ✓ |
| OA [[7](https://arxiv.org/html/2404.17833v1#bib.bib7)] | Chat | 功能调用 | ✗ | ✗ |

## 6 评估

在本节中，我们旨在回答以下研究问题（RQ）：

RQ1:

PDoctor在检测LLM代理错误规划方面的有效性如何？

RQ2:

LLM代理在规划方面的表现如何，它们会犯哪些类型的规划错误？

RQ3:

当遇到复杂的规划问题时，LLM代理的表现如何？

### 6.1 RQ1：评估PDoctor在检测错误规划方面的有效性

我们首先评估PDoctor在检测LLM代理错误规划方面的有效性。对于每种类型的LLM代理（共六种，总共有三种类型的LLM代理框架的两个模型，详见第[5](https://arxiv.org/html/2404.17833v1#S5 "5 实施与实验设置 ‣ 通过合成用户输入测试和理解LLM代理的错误规划")节），我们使用PDoctor随机生成测试用例，并在规定的时间限制内测试代理的规划表现。由于随着代理迭代，令牌的使用呈指数增长，从而使得测试LLM代理的成本很高，我们为每种类型的LLM代理设置了60分钟的时间限制。对于每个生成的测试用例，我们创建一个新的LLM代理实例，并指示它使用提供的工具处理合成的用户查询，超时时间设置为180秒。最大迭代次数设置为50，符合Yao等人的工作[[59](https://arxiv.org/html/2404.17833v1#bib.bib59)]中的设置。

此外，规划问题的难度是影响LLM代理规划表现的关键因素。特别是，LLM代理是一个对难度敏感的系统，预计在较容易的问题上表现较好，在较难的问题上表现较差。其背后的原理是，LLM代理的最终目标是通过模仿人类行为背后的逻辑来生成类人文本，而人类在解决问题时是已知的对难度敏感。PDoctor将动作数量$|\mathcal{A}|$作为控制生成规划问题难度的关键参数。由于代理需要每次调用每个工具，而每个工具都与一个动作相关联，因此动作数量直接决定了可能的规划数量。给定一个动作集$\mathcal{A}$，其中$|\mathcal{A}|=n$，如果被测试的代理严格遵循用户查询并且每个工具仅调用一次，则可能的规划数量为$P(n,n)=n!$。此外，动作数量还会影响约束集的大小和生成的用户查询的长度，因为更多的动作需要更多的句子来提及它们，这反过来又增加了约束的数量。用户查询的长度和约束集的大小都会增加问题的难度。

在本RQ中，我们从3到5的范围内随机采样$|\mathcal{A}|$的值，符合前期研究中使用的Blocksworld问题中的块数设置[[50](https://arxiv.org/html/2404.17833v1#bib.bib50), [49](https://arxiv.org/html/2404.17833v1#bib.bib49)]。Blocksworld是国际规划竞赛（IPC）采用的经典规划问题类型[[3](https://arxiv.org/html/2404.17833v1#bib.bib3)]，在该问题中，代理需要将堆叠的积木从一种配置移动到另一种配置。尽管Blocksworld中的问题难度可能与PDoctor生成的问题难度不完全对应，即使它们具有相同的难度设置（例如块数等于动作数），我们仍然认为$|\mathcal{A}|\in[3,5]$对于本实验是合适的。在此设置下，合成的用户查询的子句数从1到7不等，约束集的大小从2到10不等，这与现实应用中的常见场景一致。

表2：PDoctor在检测LLM代理错误规划中的评估结果。Z3-调用次数表示调用Z3求解器的次数。时间表示在Z3/合成/代理上花费的总时间。

|  | 代理 | 生成的 | 错误（占总数的百分比） | Z3-调用次数 | Z3-时间 | 合成-时间 | 代理-时间 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| GPT-3.5 | ReAct | 843 | 519 (61.57%) | 99,972 | 00:24.96 | 00:59.13 | 58:00.86 |
| OT | 1,168 | 635 (54.37%) | 133,008 | 00:34.30 | 01:20.49 | 57:11.94 |
| OA | 327 | 179 (54.74%) | 36,694 | 00:09.67 | 00:22.52 | 59:22.24 |
| GPT-4 | ReAct | 160 | 47 (29.38%) | 19,726 | 00:05.03 | 00:11.73 | 59:54.44 |
| OT | 144 | 32 (22.22%) | 16,253 | 00:04.13 | 00:09.66 | 59:40.41 |
| OA | 111 | 40 (36.04%) | 12,775 | 00:03.38 | 00:07.75 | 59:54.25 |

表[2](https://arxiv.org/html/2404.17833v1#S6.T2 "Table 2 ‣ 6.1 RQ1: Assessing PDoctor’s Effectiveness in Detecting Erroneous Planning ‣ 6 Evaluation ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")展示了评估结果。该表中列出了Z3的调用次数和总耗时，以便全面了解合成过程中的约束求解开销。此外，还展示了合成过程和代理执行过程的时间消耗。从表中可以明显观察到，PDoctor在合成测试用例方面非常有效。合成一个测试用例的平均时间仅约为0.07秒。尽管Z3的调用次数较多，但Z3的时间消耗相对较低，每个测试用例上Z3的平均时间不到0.03秒。此外，合成过程所需的总时间中不到一半用于Z3。相比之下，代理的执行过程是最耗时的部分，每个代理类型的执行时间都超过了57分钟。

总体而言，整个测试过程的瓶颈在于代理的吞吐量。模型和框架对LLM代理的性能有显著影响。在所有基于GPT-3.5模型的代理中，ReAct和OT比OA具有显著的速度优势，分别处理了843个和1,168个查询，而OA仅处理了327个查询。对于基于GPT-4模型的代理，所有三种框架处理的查询数量都有显著下降。同时，不同框架之间的差距缩小，这表明GPT-4的标记处理速率构成了更为严重的瓶颈。然而，重要的是要注意，在实际场景中，代理执行过程的时间开销是微不足道的，尤其是对普通用户而言。即便是最坏情况（基于GPT-4的OA），代理每个测试用例的平均速度也为32.4秒。考虑到现实世界工具可能需要的执行时间远远更长，这个时间几乎可以忽略不计。

在错误检测方面，PDoctor在所有代理设置中都发现了大量错误。对这些错误的进一步分析将在RQ2中讨论。从这张表可以明显看出，GPT-4显著提高了代理的表现，所有基于GPT-4的代理的错误率相比基于GPT-3.5的代理平均降低了48.53%。对于GPT-3.5，我们将相对较高的错误率归因于其无法处理复杂规划问题的能力，这将在RQ2和RQ3中进一步探讨。关于代理框架，OT是最推荐的框架，在两种模型中都优于其他两个框架。

总的来说，由于结果检查过程是根据约束检查进行的（见第[4.3节](https://arxiv.org/html/2404.17833v1#S4.SS3 "4.3 测试结果检查 ‣ 4 设计 ‣ 通过合成用户输入测试和理解LLM代理中的错误规划")），可以保证代理所犯的任何规划错误都会被检测到。其他类型的错误，例如调用不存在的工具或忘记完成查询中指定的任务，也可以通过检查代理的执行链轻松检测到。鉴于我们测试方法的高度全面性，可以合理地得出结论，表[2](https://arxiv.org/html/2404.17833v1#S6.T2 "表2 ‣ 6.1 RQ1：评估PDoctor在检测错误规划中的有效性 ‣ 6 评估 ‣ 通过合成用户输入测试和理解LLM代理中的错误规划")中报告的错误率真实反映了LLM代理的表现。事实上，每个LLM代理的表现与先前研究[[59](https://arxiv.org/html/2404.17833v1#bib.bib59), [41](https://arxiv.org/html/2404.17833v1#bib.bib41)]一致。

### 6.2 RQ2：理解LLM代理的规划表现及其错误

本研究问题（RQ）深入探讨了LLM代理的规划及其错误。在RQ1中，我们解释了LLM代理通常是对难度敏感的系统，因此它们的规划性能取决于所遇到问题的难度。因此，我们假设只有在LLM代理的规划能力范围内的错误才对进一步研究有价值。因此，必须进行一个具有不同难度（直至高级规划能力）的全面测试，以彻底探索LLM代理的规划能力。

规划能力测量。具体来说，我们使用PDoctor合成大量测试用例，难度级别（即动作数）逐渐增加。对于每个难度级别，我们记录规划的成功率。当成功率下降到过低时（在我们的实验中小于20%），我们认为代理已达到其“高级规划能力”。由于查询生成的采样空间由动作数决定，我们动态调整采样数量以适应这一点。也就是说，对于给定的动作数$|\mathcal{A}|=n$，我们随机生成$k\times{n\choose x}$个测试用例，其中$k$是常数因子，在我们的实验中设定为20。${n\choose x}$表示在极端情况下，合成的用户查询中包含仅提到两个动作的单个子句中的句子数（即句子中可能出现的最小动作数）。这表示给定动作数可以生成的最大句子数，描绘了查询生成的采样空间。

在我们的实验中，$n$从2开始，每次增加1，直到达到成功率阈值。同样，我们将每个动作设置的最大采样数限制为300，以考虑使用OpenAI API的高成本。在本研究问题中，我们测试并展示了所有代理在动作数从2到9范围内的规划性能，以便进行全面比较。因此，我们对每种代理框架中的六个LLM代理（每个框架下两个模型）进行了上述过程，并为每种代理类型生成了1,600个测试用例。

图 8：不同LLM代理的规划性能。

图 [8](https://arxiv.org/html/2404.17833v1#S6.F8 "图 8 ‣ 6.2 RQ2: 理解 LLM 代理的规划性能及其错误 ‣ 6 评估 ‣ 通过合成用户输入测试和理解 LLM 代理的错误规划")(a) 和图 [8](https://arxiv.org/html/2404.17833v1#S6.F8 "图 8 ‣ 6.2 RQ2: 理解 LLM 代理的规划性能及其错误 ‣ 6 评估 ‣ 通过合成用户输入测试和理解 LLM 代理的错误规划")(b) 展示了基于 GPT-3.5 和 GPT-4 模型的 LLM 代理的规划成功率。在 20% 的成功率时，画出了虚线，表示每个代理的规划能力限制。与 RQ1 的发现一致，基于 GPT-4 的代理在所有代理框架中通常表现出比基于 GPT-3.5 的代理更优越的规划能力。采用 GPT-4 的代理的上限在 $|\mathcal{A}=8|$ 时达到，而采用 GPT-3.5 的代理在 $|\mathcal{A}=4|$ 时达到其最大规划能力限制。此外，当动作数量超过三时，基于 GPT-3.5 的代理的规划成功率急剧下降，而基于 GPT-4 的代理则持续逐渐下降。这表明 GPT-4 在处理复杂规划问题时更加稳健。在代理框架方面，在这种设置下很难确定最优框架，因为每种框架都有其优点。大体而言，建议用户选择 OT 加 GPT-3.5 处理简单任务，选择 ReAct 加 GPT-4 处理复杂任务，考虑到 RQ1 中揭示的时间开销。

表 3：LLM 代理中不同类型规划错误的分布。

|  | 代理 | 超时 | 动作错误 | 动作丢失 | 订单错误 |
| --- | --- | --- | --- | --- | --- |
| GPT-3.5 | ReAct | 0.00% | 1.03% | 8.25% | 90.72% |
| OT | 0.00% | 0.00% | 0.46% | 99.54% |
| OA | 0.00% | 0.80% | 0.40% | 98.80% |
| GPT-4 | ReAct | 0.00% | 1.27% | 0.00% | 98.73% |
| OT | 0.00% | 1.12% | 0.00% | 98.88% |
| OA | 0.00% | 1.61% | 0.13% | 98.26% |

错误特征。我们将LLM代理在规划能力范围内的错误分为四种类型：（1）超时错误，指代理花费太多时间（超过180秒）或超过最大迭代次数（50次），但未能找到解决方案；（2）动作错误，指代理未能正确调用工具；（3）动作丢失，指代理未能完成查询中指定的所有任务，即某些动作丢失；（4）顺序错误，指代理未能遵循从查询中推导出的约束。表[3](https://arxiv.org/html/2404.17833v1#S6.T3 "表 3 ‣ 6.2 RQ2: 理解LLM代理的规划表现及其错误 ‣ 6 评估 ‣ 通过综合用户输入测试和理解LLM代理的错误规划")展示了每个代理的错误类型分布。顺序错误是所有代理中最常见的错误类型，表明LLM代理在解开规划问题的复杂约束时可能遇到困难。这个观察进一步确认了LLM代理对困难的敏感性。此外，基于GPT-3.5的ReAct比其他代理更容易发生动作丢失错误。我们将此归因于ReAct的长提示模板，它可能导致LLM模型忘记查询中指定的某些动作。与此相反，在GPT-4上，动作丢失错误显著减少，得益于GPT-4在长期记忆方面的显著改进。我们认为这些发现对于LLM代理的设计和优化（例如，它们的伴随提示模板）以及有意在应用中采用LLM代理的用户具有重要价值。

表 4：错误规划中不同根本原因的分布。

|  | 代理 | 概率 | 终端 | 主题 | 结构 | 约束 |
| --- | --- | --- | --- | --- | --- | --- |
| GPT-3.5 | ReAct | 19.0% | 50.0% | 18.0% | 9.0% | 4.0% |
| OT | 8.0% | 42.0% | 22.0% | 15.0% | 13.0% |
| OA | 27.0% | 28.0% | 15.0% | 18.0% | 12.0% |
| GPT-4 | ReAct | 19.0% | 44.0% | 14.0% | 18.0% | 5.0% |
| OT | 15.0% | 37.0% | 23.0% | 17.0% | 8.0% |
| OA | 28.0% | 30.0% | 14.0% | 17.0% | 11.0% |

根本原因分析。在识别出规划能力限制后，进一步调查引发失败的原因。通过使用第[4.5节](https://arxiv.org/html/2404.17833v1#S4.SS5 "4.5 错误剖析 ‣ 4 设计 ‣ 通过合成用户输入测试和理解LLM代理中的错误规划")中提出的算法进行系统的错误剖析。具体来说，我们随机抽取了100个错误，这些错误都位于规划能力限制范围内，并对其进行剖析以找出根本原因。错误剖析结果呈现在第[4表](https://arxiv.org/html/2404.17833v1#S6.T4 "表 4 ‣ 6.2 RQ2: 理解LLM代理的规划性能及其错误 ‣ 6 评估 ‣ 通过合成用户输入测试和理解LLM代理中的错误规划")中。尽管不同代理框架的根本原因分布存在差异，但在不同模型之间保持了一致性。不同模型基础上的代理之间观察到的高相似性错误模式表明，代理框架在确定错误原因时发挥的作用比模型本身更为重要。另一个发现是，概率在OA框架中导致的错误显著多于其他框架。我们将此归因于OA无法设置核心LLM的温度，导致代理行为不稳定。然而，其他框架尽管在使用PDoctor调节核心LLM的采样参数以减轻非确定性后，仍未能将概率错误压制到令人满意的水平。这个观察结果支持了我们在第[2.2节](https://arxiv.org/html/2404.17833v1#S2.SS2 "2.2 LLM代理示例 ‣ 2 初步 ‣ 通过合成用户输入测试和理解LLM代理中的错误规划")中讨论的假设，即LLM代理的多轮交互模式可能进一步加剧LLM的非确定性问题。此外，值得注意的是，Terminal是所有代理中最常见的根本原因，而Constraint通常只导致一小部分错误。换句话说，通过文本变异改变提示不仅能有效调整LLM的行为[[22](https://arxiv.org/html/2404.17833v1#bib.bib22)]，还可以改善或恶化LLM代理的表现。

表 5：引起最多错误的前五大主题。

| GPT-3.5 | GPT-4 |
| --- | --- |
| 主题 | 计数 | 主题 | 计数 |
| 服务员/女服务员 | 6 | 平面设计师 | 5 |
| 计算机程序员 | 5 | 电工 | 4 |
| 物理治疗师 | 4 | 人力资源经理 | 4 |
| 市场经理 | 4 | 记者 | 4 |
| 股票经纪人 | 4 | 视频游戏设计师 | 4 |

作为进一步的步骤，由于话题会导致显著的错误，我们还统计了导致最多错误的前五个话题，并在表格[5](https://arxiv.org/html/2404.17833v1#S6.T5 "Table 5 ‣ 6.2 RQ2: Understanding LLM Agents’ Planning Performance and Their Errors ‣ 6 Evaluation ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")中展示了它们。在这里，我们将基于相同模型的代理结果合并，因为用户查询的话题可能仅影响核心LLM模型的语言理解能力。正如在第[4.2](https://arxiv.org/html/2404.17833v1#S4.SS2 "4.2 User Query Synthesis ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")节中所述，PDoctor随机选择50个话题中的一个，这些话题与不同职业的日常任务相关，用于填充合成用户查询框架中的动作槽。观察到，对于GPT-3.5来说，最具挑战性的话题是“服务员/女服务员”，导致了6个错误，而其余19个话题从未导致任何错误。同样，对于GPT-4来说，图形设计师的日常生活似乎是代理最具挑战的话题（导致了5个错误），而有16个话题没有任何错误。我们解释认为，用户查询的话题可能对代理来说是陌生的，从而显著影响代理的表现。然而，从代理用户的角度来看，角色扮演指令（目前这些代理使用的；在第[2.3](https://arxiv.org/html/2404.17833v1#S2.SS3 "2.3 Architecture of LLM Agents ‣ 2 Preliminary ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")节中介绍）似乎不足以改善它们的表现。

### 6.3 RQ3：进一步检验LLM代理在复杂规划问题上的表现

图9：不同LLM代理在复杂规划问题上的规划表现。

在这个RQ中，我们重复了RQ2中描述的评估过程，使用扩展版本的PDoctor进一步检验LLM代理的规划能力。正如在第[4.4](https://arxiv.org/html/2404.17833v1#S4.SS4 "4.4 Extended Testing Framework ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")节中所述，扩展版本通过增加时间限制、任务持续时间和工具参数，引入了一个更复杂的规划范式，从而强调了被测试代理的规划能力。

规划能力测量。我们遵循与RQ2中描述的相同程序，来衡量LLM代理的规划能力上限。图[9](https://arxiv.org/html/2404.17833v1#S6.F9 "Figure 9 ‣ 6.3 RQ3: Further Examining LLM Agents’ Performance on Complex Planning Problems ‣ 6 Evaluation ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")展示了基于GPT-3.5和GPT-4模型的LLM代理在本实验中的规划成功率。由于所有代理的最大规划能力限制在$|\mathcal{A}=5|$时达到，因此此图中仅展示了动作数量从2到5的规划成功率。显然，所有代理的成功率相比于先前的实验有了显著下降，表明扩展版PDoctor确实引入了一个更具挑战性的（但仍然现实的）规划问题。所有基于GPT-3.5的代理在动作数量达到三时，成功率降至20%以下，表明GPT-3.5可能不适合处理如此复杂的规划问题。基于GPT-4的代理虽然维持了相对较高的成功率，但当动作数量超过四时，成功率出现急剧下降。关于代理框架，ReAct在这种设置下表现最差，当其以GPT-3.5为核心时，甚至无法解决最简单的规划问题。当采用GPT-4时，这种失败现象延续，成功率在所有动作数量下均低于50%。相比之下，OT和OA展示了相对较高的成功率，尽管这种提高有限。我们将这一现象归因于ReAct依赖少量示例提示来指导模型调用工具，而当任务变得更加复杂时，LLM模型的不稳定性变得更加明显。相反，OT和OA采用了函数调用，通过用大量函数调用数据对模型进行微调，使得它们更具鲁棒性。

表6：通过扩展版PDoctor检测到的不同类型规划错误的分布。

|  | 代理 | 超时 | 行为错误 | 动作丢失 | 顺序错误 | 参数错误 |
| --- | --- | --- | --- | --- | --- | --- |
| GPT-3.5 | ReAct | NaN | NaN | NaN | NaN | NaN |
| OT | 0.0% | 7.14% | 0.0% | 28.57% | 64.29% |
| OA | 0.0% | 0.0% | 0.0% | 50.0% | 50.0% |
| GPT-4 | ReAct | 0.0% | 64.44% | 0.0% | 33.33% | 2.22% |
| OT | 0.0% | 3.77% | 2.83% | 66.04% | 27.36% |
| OA | 0.0% | 0.74% | 1.12% | 64.31% | 33.83% |

错误类型分析。表[6](https://arxiv.org/html/2404.17833v1#S6.T6 "Table 6 ‣ 6.3 RQ3: Further Examining LLM Agents’ Performance on Complex Planning Problems ‣ 6 Evaluation ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")展示了扩展版PDoctor中每个代理的错误类型分布。扩展版要求代理将动作的开始时间传递给相应的工具，并理解工具的返回值，这些返回值揭示了它们的执行时间。因此，我们引入了一个新的错误类型——参数错误，用于表示代理错误设置工具参数时所导致的错误。特别地，一个工具的开始时间应该晚于前一个工具的结束时间；否则，代理会被认为是在错误管理规划问题的全局资源——时间。从表¹¹1ReAct基于GPT-3.5未能解决此设置中的最简单规划问题，导致该表中的NaN错误率。，我们观察到 ReAct 遇到大量的动作错误，确认了我们之前的解释，即 ReAct 的工具调用机制不适合处理复杂的规划问题。对于 OT 和 OA，当它们采用 GPT-3.5 时，主要的错误类型是参数错误，这在一定程度上解释了它们在此设置中表现不佳。相比之下，当采用 GPT-4 时，所有代理的主要错误类型仍为顺序错误，这与RQ2中的发现一致。总之，我们展示了扩展版PDoctor确实引入了一个更具挑战性的规划问题，代理的表现受模型和框架的显著影响。在实际应用中，我们建议用户采用 OT 或 OA 加 GPT-4 来处理此类复杂规划问题。

表 7：PDoctor 扩展版本识别的不同根本原因的分布。

|  | 概率 | 终端 | 话题 | 结构 | 约束 |
| --- | --- | --- | --- | --- | --- |
| ReAct | 8.0% | 22.0% | 11.0% | 14.0% | 45.0% |
| OT | 25.0% | 32.0% | 12.0% | 8.0% | 23.0% |
| OA | 23.0% | 22.0% | 11.0% | 5.0% | 39.0% |

图 10：LLM 代理执行的错误规划示例。不同颜色突出显示了动作。为了便于理解，我们还展示了推导出的约束条件以及动作与约束中变量之间的映射关系。

根本原因分析。与RQ2类似，我们剖析了LLM代理所犯的错误。由于基于GPT-3.5的代理在扩展版PDoctor上的表现不佳，我们主要检查了基于GPT-4的代理的根本原因，以提供更有深度的分析。表格[7](https://arxiv.org/html/2404.17833v1#S6.T7 "Table 7 ‣ 6.3 RQ3: Further Examining LLM Agents’ Performance on Complex Planning Problems ‣ 6 Evaluation ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")展示了每个代理的根本原因分布。从RQ2的结果中可以明显看出不同的是，约束条件在本实验中占据了更大比例的错误，成为ReAct和OA中的主要原因，并在OT中排名第二。我们认为这可能是由以下几个原因造成的。

首先，代理需要根据工具的返回值动态调整其计划，这大大增加了计划问题的复杂性，正如第[4.4节](https://arxiv.org/html/2404.17833v1#S4.SS4 "4.4 Extended Testing Framework ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")所讨论的。其次，实验结果表明，LLM在同时处理多种不同类型的需求时，往往会犯更多的错误，而这是现实应用中常见的场景。图[10](https://arxiv.org/html/2404.17833v1#S6.F10 "Figure 10 ‣ 6.3 RQ3: Further Examining LLM Agents’ Performance on Complex Planning Problems ‣ 6 Evaluation ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")展示了LLM代理在规划中犯错的一个例子。在这个例子中，代理未能将正确的开始时间传递给`applying_hair_color`，导致了参数错误。由于前一个工具在10:00开始并且耗时两小时，因此`applying_hair_color`应在12:00之后开始（即，$start\_time\geq 12$），但代理错误地将开始时间设置为8:00。尽管如果代理能够根据当前工具的开始时间重新排序动作，行动链本身是正确的，但它已经违反了查询中“您需要按照正确的顺序并在需求设定的时间范围内逐个执行所有任务”的要求（见图[7](https://arxiv.org/html/2404.17833v1#S4.F7 "Figure 7 ‣ 4.4 Extended Testing Framework ‣ 4 Design ‣ Testing and Understanding Erroneous Planning in LLM Agents through Synthesized User Inputs")）。此外，时间和持续时间的引入无疑为规划问题增加了更多的约束，这也可能加剧了代理在规划中的困难。

## 7 讨论

测试用例生成的替代方法。可以采用多种其他方法生成LLM代理的测试用例，包括文本突变和基于LLM的文本生成。通过初步研究和实验，我们发现这些方法远不如我们的方法有效。简而言之，尽管这些方法看起来实现起来更简单、负担更轻，但它们无法生成高多样性的测试用例，并且在验证结果的正确性方面也无法保证。

文本突变仅限于输入文本的表面性更改，这几乎无法覆盖LLM代理可能遇到的多样化场景。至于文本生成，一种可能的方法是将约束集输入LLM，并要求LLM用自然语言生成用户查询。然而，由于LLM的本质不稳定，这种方法存在问题。为了说明这一点，我们进行了一项实验，将约束集和变量-动作映射（例如，$\mathbf{a_{1}}$: network_diagnose）输入GPT-4，并指示其生成相应的用户查询。然后，生成的用户查询会被人工验证，以查看它是否与给定的约束语义一致。对于动作数量从2到5的情况，我们对每个动作数量重复实验10次。结果显示，生成质量受到动作数量$|\mathcal{A}|$的负面影响，后者反映了规划问题的复杂性。具体来说，当动作数量低于四时，生成的准确率达到100%，而当$|\mathcal{A}=4|$时，准确率下降到80%，当$|\mathcal{A}=5|$时，准确率降为0%。考虑到测试效果由生成文本与约束的一致性决定，结果表明文本生成方法不适用于此任务。

有效性威胁。我们的方法假设规划任务中的动作之间不存在固有的顺序依赖关系，即派生的约束完全反映了动作之间的约束。这个假设在大多数情况下成立，但也存在某些场景，其中动作之间的固有顺序依赖是常识性的，不需要显式指定。在这种情况下，如果固有的依赖关系与从合成计划中得出的约束发生冲突，我们的方法可能会失败。我们将这一问题作为未来的工作，研究如何处理此类特殊情况。

扩展。我们的方式可以在多个方向上进行扩展。首先，我们可以探索使用其他LLM，如Claude-3[[4](https://arxiv.org/html/2404.17833v1#bib.bib4)]以及其他代理框架。鉴于当前最先进的LLM在通过PDoctor扩展版进行测试时表现相当有限，我们推测有必要进一步评估更先进LLM的规划能力。将PDoctor与其他主流代理框架和LLM集成是简单的，因为PDoctor不依赖于特定的LLM或代理框架功能。

另一个直接的扩展是将我们的方法应用于对LLMs进行微调以执行规划任务。直觉上，PDoctor可以生成大量具有不同复杂度的多样化高质量测试用例。更重要的是，PDoctor能够自动识别LLMs违反了哪些约束，因此它为LLMs提供了宝贵的反馈，以改善其规划能力。这种“错误驱动”的微调过程可以反复进行，以增强LLMs的规划能力。了解该领域的读者可能知道，通过使用Z3求解器中的unsat_core函数，本地化LLMs违反的约束可以顺利实现。

也有人质疑以“即时修复”的方式修复检测到的错误计划的可行性，即我们使用约束求解器生成一个满足要求的新计划。这可以通过首先从在日常代理使用中遇到的触发错误的用户查询中提取约束（不是由PDoctor合成的那些测试查询输入），然后使用Z3找到一个满足这些约束的计划来实现。然而，我们需要澄清的是，尽管已有几项令人鼓舞的进展[[29](https://arxiv.org/html/2404.17833v1#bib.bib29), [60](https://arxiv.org/html/2404.17833v1#bib.bib60)]，从自然语言查询中提取约束仍处于初步阶段。初步的考虑是，由于LLMs的不稳定性，提取的约束可能是不一致或不足的。此外，复杂的交互模式——即用户查询、工具的名称和描述以及返回消息，其中可能包含约束——可能会使约束提取过程更加复杂。相比之下，以往的研究[[29](https://arxiv.org/html/2404.17833v1#bib.bib29), [60](https://arxiv.org/html/2404.17833v1#bib.bib60)]主要集中在对LLM进行测试，而不是对代理进行测试，测试的问题是格式良好的计划问题，忽视了LLM代理需要与用户交互的真实场景。

## 8 相关工作

LLM代理基准测试。对LLM代理进行基准测试的兴趣日益增长，已有多个基准测试套件被提出。吴等人[[57](https://arxiv.org/html/2404.17833v1#bib.bib57)]提出了SmartPlay，一个包含多个专门游戏的基准测试套件，旨在全面评估LLM代理的理解、知识和推理能力。Trulens[[1](https://arxiv.org/html/2404.17833v1#bib.bib1)]是另一个基准测试套件，旨在评估LLM代理在需要复杂推理和规划的任务中的表现。此外，还进行了一系列研究，从不同角度评估LLM代理，包括工具交互[[20](https://arxiv.org/html/2404.17833v1#bib.bib20)]、对越狱攻击的鲁棒性[[63](https://arxiv.org/html/2404.17833v1#bib.bib63)]和安全风险意识[[35](https://arxiv.org/html/2404.17833v1#bib.bib35), [61](https://arxiv.org/html/2404.17833v1#bib.bib61)]。

测试LLM（大型语言模型）。随着它们在各种应用中的显著成功，LLM已经开始在不同场景下进行测试，以确保其可靠性和鲁棒性。最近在测试LLM中的一个趋势是评估其逻辑推理能力，包括数学推理[[45](https://arxiv.org/html/2404.17833v1#bib.bib45)]、因果推理[[24](https://arxiv.org/html/2404.17833v1#bib.bib24), [31](https://arxiv.org/html/2404.17833v1#bib.bib31)]和规划能力[[49](https://arxiv.org/html/2404.17833v1#bib.bib49), [50](https://arxiv.org/html/2404.17833v1#bib.bib50)]。其中，规划能力对LLM尤为重要，因为它构成了许多应用的基础，例如自动驾驶汽车[[54](https://arxiv.org/html/2404.17833v1#bib.bib54), [15](https://arxiv.org/html/2404.17833v1#bib.bib15)]、机器人技术[[23](https://arxiv.org/html/2404.17833v1#bib.bib23)]以及任何基于代理的系统[[59](https://arxiv.org/html/2404.17833v1#bib.bib59), [39](https://arxiv.org/html/2404.17833v1#bib.bib39), [52](https://arxiv.org/html/2404.17833v1#bib.bib52)]。

## 9 结论

我们提出了PDoctor，一种新颖的自动化方法，用于测试和理解LLM代理的规划能力。我们将检测规划错误表述为约束满足问题，并提出了一个完全自动化的框架，用于合成多样且高质量的用户输入以进行测试。我们使用三个主流代理框架和两个强大的LLM（GPT-3.5和GPT-4）来评估PDoctor的有效性。结果表明，PDoctor能够有效检测LLM代理的规划错误，并为揭示错误背后的机制提供有价值的见解。

## 参考文献

+   [1] Trulens，评估和测试LLM应用程序。[https://medium.com/trulens](https://medium.com/trulens)。

+   [2] 设计用于金融工具尽职调查的LLM代理工具。 [https://developers.lseg.com/en/article-catalog/article/designing-llm-agent-tools-for-due-diligence-in-financial-instruments](https://developers.lseg.com/en/article-catalog/article/designing-llm-agent-tools-for-due-diligence-in-financial-instruments)，2024年。

+   [3] 国际规划竞赛。 [https://www.icaps-conference.org/competitions/](https://www.icaps-conference.org/competitions/)，2024年。

+   [4] 介绍Claude的下一代。 [https://www.anthropic.com/news/claude-3-family](https://www.anthropic.com/news/claude-3-family)，2024年。

+   [5] Openai工具。 [https://python.langchain.com/docs/modules/agents/agent_types/openai_tools](https://python.langchain.com/docs/modules/agents/agent_types/openai_tools)，2024年。

+   [6] Openai的函数调用。 [https://platform.openai.com/docs/guides/function-calling](https://platform.openai.com/docs/guides/function-calling)，2024年。

+   [7] Openai助手概述。 [https://platform.openai.com/docs/assistants/overview?context=with-streaming](https://platform.openai.com/docs/assistants/overview?context=with-streaming)，2024年。

+   [8] 这项AI研究介绍了“rafa”：一种具有可证明样本效率的自主LLM代理的原则性人工智能框架。 [https://www.marktechpost.com/2023/10/24/this-ai-research-introduces-rafa-a-principled-artificial-intelligence-framework-for-autonomous-llm-agents-with-provable-sample-efficiency/](https://www.marktechpost.com/2023/10/24/this-ai-research-introduces-rafa-a-principled-artificial-intelligence-framework-for-autonomous-llm-agents-with-provable-sample-efficiency/)，2024年。

+   [9] unskript推出了一个基于AI的基础设施健康智能平台，专为软件团队设计。 [https://markets.businessinsider.com/news/stocks/unskript-launches-ai-powered-infrastructure-health-intelligence-platform-for-software-teams-1032992108](https://markets.businessinsider.com/news/stocks/unskript-launches-ai-powered-infrastructure-health-intelligence-platform-for-software-teams-1032992108)，2024年。

+   [10] Achiam, J., Adler, S., Agarwal, S., Ahmad, L., Akkaya, I., Aleman, F. L., Almeida, D., Altenschmidt, J., Altman, S., Anadkat, S., 等人. GPT-4技术报告。 arXiv预印本 arXiv:2303.08774（2023年）。

+   [11] Ahn, M., Brohan, A., Brown, N., Chebotar, Y., Cortes, O., David, B., Finn, C., Fu, C., Gopalakrishnan, K., Hausman, K., 等人. 做我能做的，而不是我说的：将语言与机器人能力对接。 arXiv预印本 arXiv:2204.01691（2022年）。

+   [12] Bills, S., Cammarata, N., Mossing, D., Tillman, H., Gao, L., Goh, G., Sutskever, I., Leike, J., Wu, J., 和 Saunders, W. 语言模型可以解释语言模型中的神经元。 [https://openaipublic.blob.core.windows.net/neuron-explainer/paper/index.html](https://openaipublic.blob.core.windows.net/neuron-explainer/paper/index.html)（2023年）。

+   [13] Carbonell, J., Etzioni, O., Gil, Y., Joseph, R., Knoblock, C., Minton, S., 和 Veloso, M. 《Prodigy：一个集成的规划与学习架构》。ACM SIGART Bulletin 2, 4 (1991), 51–55。

+   [14] Chase, H. 《LangChain》，2022年10月。

+   [15] Chen, Y., Veer, S., Karkus, P., 和 Pavone, M. 《自动驾驶车辆的交互式联合规划》。IEEE Robotics and Automation Letters (2023)。

+   [16] Creswell, A., Shanahan, M., 和 Higgins, I. 《选择推理：利用大型语言模型进行可解释的逻辑推理》。arXiv 预印本 arXiv:2205.09712 (2022)。

+   [17] De Moura, L., 和 Bjørner, N. 《Z3：一种高效的 SMT 求解器》。在《工具与算法构建与分析系统国际会议》上，Springer，pp. 337–340 (2008)。

+   [18] Ghallab, M., Nau, D., 和 Traverso, P. 《自动规划：理论与实践》。Elsevier, 2004。

+   [19] Huang, J., Chen, X., Mishra, S., Zheng, H. S., Yu, A. W., Song, X., 和 Zhou, D. 《大型语言模型尚无法自我修正推理》。在 ICLR 会议论文集（2024）中。

+   [20] Huang, Y., Shi, J., Li, Y., Fan, C., Wu, S., Zhang, Q., Liu, Y., Zhou, P., Wan, Y., Gong, N. Z., 等. 《大型语言模型的 MetaTool 基准：决定是否使用工具以及使用哪些工具》。arXiv 预印本 arXiv:2310.03128 (2023)。

+   [21] Ji, Z., Ma, P., Li, Z., 和 Wang, S. 《基于大型语言模型的代码生成基准与解释：一种以因果关系为中心的方法》。arXiv 预印本 arXiv:2310.06680 (2023)。

+   [22] Jiao, W., Wang, W., Huang, J.-t., Wang, X., Shi, S., 和 Tu, Z. 《ChatGPT 是一个好的翻译工具吗？是的，使用 GPT-4 作为引擎》。arXiv 预印本 arXiv:2301.08745 (2023)。

+   [23] Joshi, S. S., Hutchinson, S., 和 Tsiotras, P. 《Les：用于机器人路径规划的局部利用采样》。在2023年 IEEE 国际机器人与自动化大会（ICRA）上，IEEE，pp. 1551–1557 (2023)。

+   [24] Kıcıman, E., Ness, R., Sharma, A., 和 Tan, C. 《因果推理与大型语言模型：为因果关系开辟新天地》。arXiv 预印本 arXiv:2305.00050 (2023)。

+   [25] Lanham, T., Chen, A., Radhakrishnan, A., Steiner, B., Denison, C., Hernandez, D., Li, D., Durmus, E., Hubinger, E., Kernion, J., Lukosiute, K., Nguyen, K., Cheng, N., Joseph, N., Schiefer, N., Rausch, O., Larson, R., McCandlish, S., Kundu, S., Kadavath, S., Yang, S., Henighan, T., Maxwell, T., Telleen-Lawton, T., Hume, T., Hatfield-Dodds, Z., Kaplan, J., Brauner, J., Bowman, S. R., 和 Perez, E. 《链式思维推理中的忠实度测量》。arXiv 预印本 arXiv:2307.13702 (2023)。

+   [26] Li, Z., Wang, C., Liu, Z., Wang, H., Wang, S., 和 Gao, C. 《CCTEST：测试与修复代码完成系统》。在 ACM ICSE 会议论文集（2023）中。

+   [27] Li, Z., Wang, C., Ma, P., Wu, D., Li, T., Wang, S., Gao, C., 和 Liu, Y. 《拆分与合并：对齐大型语言模型评估器中的位置偏差》。arXiv 预印本 arXiv:2310.01432 (2023)。

+   [28] Lin, Z., Gou, Z., Liang, T., Luo, R., Liu, H., 和 Yang, Y. CriticBench: 基准测试 LLMs 进行批评-修正推理。arXiv 预印本 arXiv:2402.14809 (2024)。

+   [29] Liu, B., Jiang, Y., Zhang, X., Liu, Q., Zhang, S., Biswas, J., 和 Stone, P. LLM+ p: 提高大型语言模型的最优规划能力。arXiv 预印本 arXiv:2304.11477 (2023)。

+   [30] Liu, S., Lu, N., Chen, C., 和 Tang, K. 针对单词级别对抗性文本攻击的高效组合优化。IEEE/ACM 音频、语音与语言处理学报 30 (2021), 98–111。

+   [31] Long, S., Schuster, T., Piché, A., de Montreal, U., Research, S., 等人. 大型语言模型能否构建因果图？arXiv 预印本 arXiv:2303.05279 (2023)。

+   [32] Lu, Y., Zhang, X., Xu, X., 和 Yao, W. 基于学习的近最优运动规划用于具有不确定动力学的智能车辆。IEEE 机器人与自动化信函 (2023)。

+   [33] Ma, W., Wu, D., Sun, Y., Wang, T., Liu, S., Zhang, J., Xue, Y., 和 Liu, Y. 结合微调和基于 LLM 的代理进行直观的智能合约审计及其理由。arXiv 预印本 arXiv:2403.16073 (2024)。

+   [34] McCarthy, J., 等人. 情境、行动与因果法则。Comtex Scientific, 1963。

+   [35] Naihin, S., Atkinson, D., Green, M., Hamadi, M., Swift, C., Schonholtz, D., Kalai, A. T., 和 Bau, D. 在野外安全测试语言模型代理。arXiv 预印本 arXiv:2311.10538 (2023)。

+   [36] Nilsson, N. J., 等人. Shakey 机器人，第 323 卷。Sri International Menlo Park, California, 1984。

+   [37] Papineni, K., Roukos, S., Ward, T., 和 Zhu, W.-J. Bleu: 一种自动评估机器翻译的方法。收录于第40届计算语言学协会年会论文集 (2002)，第311–318页。

+   [38] Payandeh, A., Pluth, D., Hosier, J., Xiao, X., 和 Gurbani, V. K. LLMs 对逻辑谬误的敏感度有多高？arXiv 预印本 arXiv:2308.09853 (2023)。

+   [39] Shao, Y., Li, L., Dai, J., 和 Qiu, X. Character-llm: 一种可训练的角色扮演代理。arXiv 预印本 arXiv:2310.10158 (2023)。

+   [40] Shen, Y., Song, K., Tan, X., Li, D., Lu, W., 和 Zhuang, Y. Hugginggpt: 利用 ChatGPT 和其在 Hugging Face 的朋友解决 AI 任务。神经信息处理系统进展 36 (2024)。

+   [41] Shinn, N., Cassano, F., Gopinath, A., Narasimhan, K., 和 Yao, S. Reflexion: 具有语言强化学习的语言代理。神经信息处理系统进展 36 (2024)。

+   [42] Shridhar, M., Yuan, X., Côté, M.-A., Bisk, Y., Trischler, A., 和 Hausknecht, M. Alfworld: 对齐文本和具象环境以进行互动学习。arXiv 预印本 arXiv:2010.03768 (2020)。

+   [43] Significant Gravitas. AutoGPT。

+   [44] Singhal, K., Azizi, S., Tu, T., Mahdavi, S. S., Wei, J., Chung, H. W., Scales, N., Tanwani, A., Cole-Lewis, H., Pfohl, S., 等人. 大型语言模型编码临床知识。《自然》 (2023), 1–9。

+   [45] Stolfo, A., Jin, Z., Shridhar, K., Schölkopf, B., 和 Sachan, M. 一个因果框架，用于量化语言模型在数学推理中的鲁棒性。arXiv 预印本 arXiv:2210.12023（2022）。

+   [46] Sun, Y., Wu, D., Xue, Y., Liu, H., Ma, W., Zhang, L., Shi, M., 和 Liu, Y. LLM4Vuln：一个统一的评估框架，用于解耦和增强 LLMs 的漏洞推理。arXiv 预印本 arXiv:2401.16185（2024）。

+   [47] Sun, Y., Wu, D., Xue, Y., Liu, H., Wang, H., Xu, Z., Xie, X., 和 Liu, Y. GPTScan：通过将 GPT 与程序分析结合，检测智能合约中的逻辑漏洞。在 ACM ICSE 会议论文集中（2024）。

+   [48] Sun, Z., Zhang, J. M., Harman, M., Papadakis, M., 和 Zhang, L. 机器翻译的自动化测试与改进。收录于 ACM/IEEE 第42届国际软件工程会议论文集（2020），第974–985页。

+   [49] Valmeekam, K., Marquez, M., Olmo, A., Sreedharan, S., 和 Kambhampati, S. PlanBench：一个可扩展的基准，用于评估大型语言模型在规划和推理变化中的表现。《神经信息处理系统进展》36期（2024）。

+   [50] Valmeekam, K., Marquez, M., Sreedharan, S., 和 Kambhampati, S. 大型语言模型的规划能力——一项批判性研究。《神经信息处理系统进展》36期（2024）。

+   [51] Wang, S., Wei, Z., Choi, Y., 和 Ren, X. 大型语言模型能否进行规则推理？逻辑支撑用于压力测试和改进 LLMs。arXiv 预印本 arXiv:2402.11442（2024）。

+   [52] Wang, Z., Cai, S., Chen, G., Liu, A., Ma, X. S., 和 Liang, Y. 描述、解释、规划和选择：与大型语言模型（LLMs）互动规划使开放世界多任务代理成为可能。《神经信息处理系统进展》36期（2024）。

+   [53] Wei, A., Haghtalab, N., 和 Steinhardt, J. 破解：大型语言模型安全训练为何失败？在 NeurIPS 会议论文集中（2023）。

+   [54] Weng, L. LLM 驱动的自主代理。[https://lilianweng.github.io/posts/2023-06-23-agent/](https://lilianweng.github.io/posts/2023-06-23-agent/)，2023。

+   [55] Wong, W. K., Wang, H., Li, Z., Liu, Z., Wang, S., Tang, Q., Nie, S., 和 Wu, S. 使用大型语言模型精炼反编译的 C 代码。arXiv 预印本 arXiv:2310.06530（2023）。

+   [56] Wu, X., Li, Y.-L., Sun, J., 和 Lu, C. Symbol-LLM：利用语言模型进行视觉人类活动推理中的符号系统。 《神经信息处理系统进展》36期（2024）。

+   [57] Wu, Y., Tang, X., Mitchell, T. M., 和 Li, Y. Smartplay：一个用于评估 LLM 作为智能代理的基准。arXiv 预印本 arXiv:2310.01557（2023）。

+   [58] Xu, Z., Jain, S., 和 Kankanhalli, M. 幻觉是不可避免的：大型语言模型的固有限制。arXiv 预印本 arXiv:2401.11817（2023）。

+   [59] Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., 和 Cao, Y. React：语言模型中推理与行动的协同作用。arXiv 预印本 arXiv:2210.03629（2022）。

+   [60] Ye, X., Chen, Q., Dillig, I., 和 Durrett, G. Satlm: 使用声明性提示的可满足性辅助语言模型. 《神经信息处理系统进展》36 (2024)。

+   [61] Yuan, T., He, Z., Dong, L., Wang, Y., Zhao, R., Xia, T., Xu, L., Zhou, B., Li, F., Zhang, Z., 等. R-judge: 基准测试大型语言模型代理的安全风险意识. arXiv预印本 arXiv:2401.10019 (2024)。

+   [62] Zeng, Z., Yu, J., Gao, T., Meng, Y., Goyal, T., 和 Chen, D. 评估大型语言模型在评估指令跟随上的表现. arXiv预印本 arXiv:2310.07641 (2023)。

+   [63] Zhan, Q., Liang, Z., Ying, Z., 和 Kang, D. Injecagent: 在工具集成的大型语言模型代理中基准测试间接提示注入. arXiv预印本 arXiv:2403.02691 (2024)。

+   [64] Zhang, T., Kishore, V., Wu, F., Weinberger, K. Q., 和 Artzi, Y. Bertscore: 使用bert评估文本生成. arXiv预印本 arXiv:1904.09675 (2019)。
