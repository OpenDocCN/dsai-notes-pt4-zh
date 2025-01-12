<!--yml

类别：未分类

日期：2025-01-11 12:43:32

-->

# 自组织代理：面向超大规模代码生成与优化的LLM多代理框架

> 来源：[https://arxiv.org/html/2404.02183/](https://arxiv.org/html/2404.02183/)

石桥洋一

TsukushiAI

ishibashi.tsukushiai@gmail.com

&西村良政

TsukushiAI

nishimura.tsukushiai@gmail.com

###### 摘要

最近，利用大语言模型（LLM）代理进行自动代码生成的进展，使我们更接近自动化软件开发的未来。然而，现有的单一代理方法由于上下文长度的限制，面临在生成和优化大规模、复杂代码库时的局限性。为了解决这一挑战，我们提出了自组织多代理框架（SoA），这是一种新型的多代理框架，能够实现大规模代码的可扩展和高效生成与优化。在SoA中，自组织代理独立运行，生成和修改代码组件，同时无缝协作以构建整体代码库。我们框架的一个关键特性是，代理根据问题的复杂性自动增殖，从而实现动态扩展。这使得整体代码量可以根据代理数量无限增加，而每个代理管理的代码量保持不变。我们在HumanEval基准上评估了SoA，并展示了与单一代理系统相比，SoA中的每个代理处理的代码显著较少，但整体生成的代码大大增加。此外，在Pass@1准确度方面，SoA比强大的单一代理基准高出5%。¹¹1我们的代码将在[https://github.com/tsukushiAI/self-organized-agent](https://github.com/tsukushiAI/self-organized-agent)发布。

自组织代理：面向超大规模代码生成与优化的LLM多代理框架

超大规模代码生成与优化

石桥洋一 TsukushiAI ishibashi.tsukushiai@gmail.com                        西村良政 TsukushiAI nishimura.tsukushiai@gmail.com

## §​ 1 引言

近年来，使用大语言模型（LLMs）的代理研究取得了长足进展，包括Brown等人（[2020](https://arxiv.org/html/2404.02183v1#bib.bib3)）；OpenAI（[2023](https://arxiv.org/html/2404.02183v1#bib.bib16)）；Touvron等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib21)），如ReAct Yao等人（[2023b](https://arxiv.org/html/2404.02183v1#bib.bib24)）、Reflexion Shinn等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib20)）、Toolformer Schick等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib18)）和AutoGPT ²²2[https://github.com/Significant-Gravitas/](https://github.com/Significant-Gravitas/)，这些研究开拓了自动化人类任务的可能性。这些进展特别促进了自动化应用和工具开发领域自动代码生成技术的快速发展，Hong等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib7)）；Dong等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib5)）；Huang等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib8)）。与非代理方法相比，Muennighoff等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib14)）；Li等人（[2023b](https://arxiv.org/html/2404.02183v1#bib.bib11)），这些研究成果在自动代码生成方面取得了显著的性能提升，Zhong等人（[2024](https://arxiv.org/html/2404.02183v1#bib.bib26)）；Zhou等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib27)）。

图1：左侧（单一代理）：单个代理负责整个实现。随着代码库的不断增长，代码生成、修改和内存管理的负载也随之增加，这使得管理和开发变得困难。整个代码库越大，自我调试时对上下文长度的压力就越大，限制了能够管理的代码量。右侧（SoA）：实现被分配给多个代理。代理是独立的；代码生成、修改和内存管理与其他代理分离。每个代理只管理自己的部分，从而可以专注于实现，而不必考虑整个代码库的复杂性。此外，代理会根据问题的复杂性自动扩展。这使得可以在保持每个代理管理/生成/修改代码量恒定的情况下，生成和修改复杂的大规模代码。

最近的研究大多集中在单代理方法上进行代码生成。这些单代理代码生成方法在扩展性上存在局限性，尤其是在实现变得复杂并需要大量代码库时。造成这一技术难题的主要原因是，单个代理必须独自管理整个代码生成过程。例如，实现一个机器学习算法涉及多个阶段，如数据预处理、算法训练和结果评估，这些阶段包括许多函数和类。当这些复杂的组件组合在一起时，代码库不可避免地会变得非常庞大。然而，LLM的上下文长度存在限制，随着输入令牌数量的增加，推理性能会下降，Levy 等人 ([2024](https://arxiv.org/html/2404.02183v1#bib.bib9)); Shaham 等人 ([2023](https://arxiv.org/html/2404.02183v1#bib.bib19)); Li 等人 ([2023a](https://arxiv.org/html/2404.02183v1#bib.bib10))。对于这样一个庞大的代码库，单个代理在理解和生成或修改适当的代码时面临显著挑战，因为它需要理解和管理大量的上下文。因此，随着复杂度和规模的增加，单代理方法在高效生成和修改代码方面力不从心。

为了应对这些挑战，我们提出了一种自组织多代理框架，能够自动生成和修改大规模代码（[图 1](https://arxiv.org/html/2404.02183v1#S1.F1 "Figure 1 ‣ §​ 1 Introduction ‣ Self-Organized Agents: A LLM Multi-Agent Framework toward Ultra Large-Scale Code Generation and Optimization")）。*自组织* Ashby ([1947](https://arxiv.org/html/2404.02183v1#bib.bib2)) 是一种现象，指的是生物体或物质通过各自独立的自主行为，尽管无法监督整个系统，依然能形成有序的、庞大的结构。在我们的框架中，自组织代理，每个负责不同的代码部分或任务，独立生成和修改代码。通过代理的自组织，单个代理不再需要理解整个代码库，从而通过增加代理的数量，可以简单地扩展大规模代码。我们框架的另一个特点是，代理根据问题的复杂度自动增殖，使得整体代码库得以扩展，同时保持每个代理处理的代码量不变。这些特性使得大规模代码的动态和灵活生成与修改成为可能，而传统的单代理方法无法实现这一点。

在我们的实验中，我们使用HumanEval（Chen等人，[2021](https://arxiv.org/html/2404.02183v1#bib.bib4)），一个用于代码生成的基准，评估了该框架的表现。结果表明，我们的自组织多代理框架在代码生成方面优于现有的强大代码生成代理Reflexion（Shinn等人，[2023](https://arxiv.org/html/2404.02183v1#bib.bib20)）（[§​ 4.1](https://arxiv.org/html/2404.02183v1#S4.SS1 "§​ 4.1 Main Results ‣ §​ 4 Experiments ‣ Self-Organized Agents: A LLM Multi-Agent Framework toward Ultra Large-Scale Code Generation and Optimization")），展示了我们方法在生成和修改代码方面的有效性。此外，通过对实验结果的详细分析，我们揭示了代理如何根据问题的复杂性自动增殖，有效地扩大整体代码量，同时保持每个代理的代码生成不变（[§​ 4.2](https://arxiv.org/html/2404.02183v1#S4.SS2 "§​ 4.2 Analysis ‣ §​ 4 Experiments ‣ Self-Organized Agents: A LLM Multi-Agent Framework toward Ultra Large-Scale Code Generation and Optimization")）。这些实验结果支持了我们框架的贡献，它克服了单代理方法面临的可扩展性问题，提供了一种能够处理更大规模项目的解决方案。

## §​ 2 代码生成任务

代码生成任务涉及从文档字符串生成Python函数，参考了Chen等人（[2021](https://arxiv.org/html/2404.02183v1#bib.bib4)）。在这个任务中，代理会获得一个文档字符串，定义了函数输入的类型、期望的输出以及函数应满足的特定要求。然后，代理需要生成一个满足指定功能的函数代码。生成的代码会通过单元测试进行准确性验证，代码的质量根据其能通过测试用例的能力来评估。与以往的研究类似，Shinn等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib20)）；Zhong等人（[2024](https://arxiv.org/html/2404.02183v1#bib.bib26)）；Zhou等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib27)），我们使用评估指标Pass@$1$，Chen等人（[2021](https://arxiv.org/html/2404.02183v1#bib.bib4)），其中一个问题如果$k$个代码样本中的任何一个通过所有测试用例，则认为该问题已经解决。

## §​ 3 自组织代理框架

我们的自组织代理（SoA）框架通过让自组织代理独立生成和修改小规模且简单的代码，从而实现了大规模和复杂代码的高效实现。在本节中，我们介绍了SoA的重要组成部分，即代理和负责比代理更抽象处理的层，并最终介绍了SoA框架中的代码生成和修改协议。

### §​ 3.1 子代理

子代理根据给定的文档字符串实现函数。如[图2](https://arxiv.org/html/2404.02183v1#S3.F2 "图2 ‣ §​ 3.1 子代理 ‣ §​ 3 自我组织代理框架 ‣ 自我组织代理：一个面向超大规模代码生成和优化的LLM多代理框架")所示，这个代理具有由两个元素组成的简单结构：一个LLM和一个内存。LLM从给定的文档字符串生成代码，并根据单元测试的结果修改代码。内存存储代理自己生成的代码，并在代码修改过程中检索最新的代码和单元测试反馈，并将其输入到LLM中。如果一个代理具备这些最基本的规格，便可以使用现成的代理（例如，Reflexion）作为子代理。我们故意使用一个简单的代理来验证SoA在简单设置中的有效性。

图2：代码生成概览。子代理从给定的文档字符串生成可执行的Python函数。母代理生成函数的骨架。母代理生成一个新的初始化代理（子代理或母代理），并委派未实现的函数。

#### 代码生成

子代理的主要角色是根据给定函数的文档字符串生成符合规范的函数。如[图2](https://arxiv.org/html/2404.02183v1#S3.F2 "图2 ‣ §​ 3.1 子代理 ‣ §​ 3 自我组织代理框架 ‣ 自我组织代理：一个面向超大规模代码生成和优化的LLM多代理框架")所示，代理根据指令生成剩余的函数并完成它。完成的函数实现被存储在内存中，且函数的单元测试也被存储，因为它们构成了未来代码修改的基础。

#### 代码修改：赋予子代理自我组织和适应能力

SoA 框架中代理的一个显著特点是它们能够根据周围代理的状态自主地改进自己的代码。这一过程使得 SoA 区别于传统的代理方法，并展示了自组织在代码修改中的力量。尽管像 Reflexion Shinn 等（[2023](https://arxiv.org/html/2404.02183v1#bib.bib20)）等现有代理仅依赖于单元测试结果，但 SoA 中的子代理通过独立观察母代理的状态，如修改差异和反馈，突破了这一限制。通过从周围环境中收集这些宝贵信息，子代理能够调整其行为，并在没有明确指令的情况下作出更明智的代码修改决策。母代理生成的修改和反馈为子代理提供了重要的信息来源。通过这些洞察，子代理能够更有效地修改自己的代码，从而以高效且适应性强的方式促进代码库的整体改进。[图 3](https://arxiv.org/html/2404.02183v1#S3.F3 "图 3 ‣ §​ 3.2 母代理 ‣ §​ 3 自组织代理框架 ‣ 自组织代理：一个面向超大规模代码生成和优化的 LLM 多代理框架") 展示了这个过程，过程从执行单元测试并从记忆中检索最新的实现开始。然后，子代理利用 LLM 的力量创建代码修改提案，将从母代理观察到的信息与测试结果和最新的实现细节无缝结合。通过将修改后的代码存储在记忆中，子代理创建了一个反馈循环，持续不断地优化和改进代码库。这个迭代过程由自组织和适应性原则驱动，使得 SoA 能够高效且有效地处理复杂的代码修改任务。随着子代理与母代理的协同工作，它们有助于创建一个更优化、更庞大的代码库。

### §​ 3.2 母代理（Mother Agent）

母代理是一个生成新代理（母代理或子代理）的代理。与子代理类似，母代理根据其给定的文档字符串独立实现特定的 Python 函数。母代理具有记忆、代码生成能力和自我调试功能，这些功能与子代理相同。母代理的独特之处在于，它能够根据问题的复杂性生成多个子代理，并将部分实现任务委派给这些代理。这种结构使得母代理可以专注于实现抽象过程，而母代理生成的子代理则专注于实现具体过程。这种分工提高了 SoA 框架的整体效率和灵活性。

图 3：代码修改概览。代理（母体/子体）观察母体的状态（反馈、旧代码和更新的代码），并利用这些信息改进它们所负责的函数。上层代理的状态被下层代理用于修改代码。这种状态传播促进了层次结构中的协作与信息共享，从而实现高效的代码修改。

#### 代码生成

我们通过母体代理的代码生成过程，结合[图 2](https://arxiv.org/html/2404.02183v1#S3.F2 "图 2 ‣ §​ 3.1 子代理 ‣ §​ 3 自组织代理框架 ‣ 自组织代理：面向超大规模代码生成与优化的LLM多代理框架")中展示的is_sum_of_odds_ten函数的实现示例进行说明。起点是该函数的文档字符串和单元测试，这些内容在后续的自我调试阶段会被记忆并作为参考。母体代理的第一项任务是根据给定的文档字符串生成实现的框架，包括提取奇数的子函数get_odd_numbers和计算奇数和的子函数sum_of_numbers。这些子函数的数量和类型由LLM根据问题的复杂性自动确定。

需要注意的是，这些子函数尚未实现，母体代理并不会直接实现它们。相反，它将子函数的实现委托给其他代理，使母体代理能够专注于生成框架，并简化自身的代码生成过程。在生成子函数的文档字符串和单元测试之后，这些任务会分配给新初始化的代理进行实现。这些代理会在不查看母体代理实现的is_sum_of_odds_ten函数内部的情况下，继续各自函数的实现。由于同一母体内的代理可以异步工作，整体的代码生成过程得以简化。

#### 代码修改

母体的代码修改几乎与子体的代码修改相同（[图 3](https://arxiv.org/html/2404.02183v1#S3.F3 "图 3 ‣ §​ 3.2 母体代理 ‣ §​ 3 自组织代理框架 ‣ 自组织代理：面向超大规模代码生成与优化的LLM多代理框架")）。它观察上层母体的信息，并利用这些信息修改它负责的函数。唯一的区别是，它生成的反馈以及修改前后的代码将被下层代理（子体或母体）使用。

### §​ 3.3 自组织代理过程

自组织代理（SoA）框架是一个分布式框架，其中多个代理（包括母体代理和子代理）反复生成和修改函数。该框架的核心在于自组织原理，每个代理独立工作，无需直接观察整个代码库。母体代理和子代理的层次化组合形成了一个代理网络，能够有效地构建一个单一的大规模代码库。在这个层次结构中，母体代理通过分解任务并将它们委派给自己生成的代理，将复杂问题分解为更易管理的小问题。尽管每个代理都是独立的，但所有代理协同工作，可以高效地实现单个功能。尽管每个代理生成、修改和管理的代码量始终较小，但代理的数量可以扩展，从而根据问题的难度无限增加生成的代码量。详细算法请参见附录中的算法[1](https://arxiv.org/html/2404.02183v1#alg1 "算法1 ‣ 附录A伪代码 ‣ 自组织代理：面向超大规模代码生成与优化的LLM多代理框架")。

图4：SoA框架概览。母体代理和子代理层次化地构建一个网络，执行函数的生成和修改。母体代理将任务委派给其他母体代理或子代理，每个代理独立执行任务，同时有效地实现单个功能的整体执行。

#### 代码生成

SoA框架中的代码生成过程始于函数的文档字符串和单元测试。在初始阶段，只有一个初始化的母体代理，它是树形结构的根节点。基于输入的文档字符串和单元测试，它为子任务生成文档字符串和单元测试，并将它们传递给它生成的其他代理（参见 [§​ 3.2](https://arxiv.org/html/2404.02183v1#S3.SS2 "§​ 3.2 母体代理 ‣ §​ 3 自组织代理框架 ‣ 自组织代理：面向超大规模代码生成与优化的LLM多代理框架")）。如果树结构达到预定的深度，任务会被传递给子代理；否则，任务会被传递给新生成的母体代理。通过反复繁殖并增加代理数量，直到最后一个代理，便可以在保持每个代理管理的代码量不变的情况下生成大规模的代码。

#### 代码修改

一旦代码生成完成，过程将过渡到代码修改阶段。首先，将所有代理的实现合并，生成最终的实现。使用提供给根母代理的单元测试来评估最终实现，并根据结果生成反馈。由于没有比根母代理更高级别的代理，因此不使用来自更高层级代理的信息，如[图3](https://arxiv.org/html/2404.02183v1#S3.F3 "图 3 ‣ §​ 3.2 母代理 ‣ §​ 3 自组织代理框架 ‣ 自组织代理：面向超大规模代码生成与优化的LLM多代理框架")所示。修改过程基于此反馈开始，并将信息从根母代理传播到子代理。每个代理根据收到的反馈更新其实现，生成新的反馈，并将其传输给更低层级的代理（见[§​ 3.2](https://arxiv.org/html/2404.02183v1#S3.SS2 "§​ 3.2 母代理 ‣ §​ 3 自组织代理框架 ‣ 自组织代理：面向超大规模代码生成与优化的LLM多代理框架")）。最终，子代理更新自身的实现，过程终止（见[§​ 3.1](https://arxiv.org/html/2404.02183v1#S3.SS1 "§​ 3.1 子代理 ‣ §​ 3 自组织代理框架 ‣ 自组织代理：面向超大规模代码生成与优化的LLM多代理框架")）。这一系列过程会一直重复，直到达到预定的最大迭代次数。

## §​ 4 实验

#### LLM选择

我们使用了GPT3.5-turbo³³3gpt3.5-turbo-1106进行代码生成和反馈生成。⁴⁴4GPT-4由于实验成本过高而未被选择。

#### 基准线

我们将SoA与几种最先进的代码生成方法进行了比较，包括AlphaCode Li等人（[2022](https://arxiv.org/html/2404.02183v1#bib.bib12)）、Incoder Fried等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib6)）、Codex Chen等人（[2021](https://arxiv.org/html/2404.02183v1#bib.bib4)）、CoT Wei等人（[2022](https://arxiv.org/html/2404.02183v1#bib.bib22)）以及Gemini Pro Anil等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib1)）。此外，我们还评估了多种基于GPT-3.5的代理的表现，如ChatGPT、Self-Edit Zhang等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib25)）和Reflexion Shinn等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib20)）。这些基准线的选择代表了多种方法的多样性，包括单代理系统和多代理系统，以及具有和不具有自我调试功能的方法。

#### 代理配置

为了评估SoA框架的有效性，我们选择了Reflexion代理作为基准。Reflexion基于给定的文档字符串和自动生成的单元测试，迭代地修改代码，直到达到最大迭代次数或通过单元测试。Reflexion与SoA的主要区别在于，Reflexion由一个单一代理组成，而SoA由自组织的多个代理组成。在SoA配置中，我们将学习循环的最大迭代次数设置为8，最大树深度设置为2。此外，按照Shinn等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib20)）的方法，我们为LLM提供了少量示例轨迹。

#### 数据和任务

为了评估自动代码生成的性能，我们使用了HumanEval Chen等人（[2021](https://arxiv.org/html/2404.02183v1#bib.bib4)）基准。HumanEval是一组包含多样化编程问题的测试集，旨在衡量生成代码的功能正确性。我们使用了Python语言集进行评估，并遵循了Reflexion Shinn等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib20)）的评估方法。在此过程中，为每个生成的代码创建多个测试用例，并随机选择$n$个测试用例来构建测试套件。该测试套件用于验证生成的代码是否能正确运行。我们为Reflexion设置了6个单元测试，为SoA设置了1个单元测试。

| 方法 | SD | SO | Pass@1 |
| --- | --- | --- | --- |
| AlphaCode   Li等人 ([2022](https://arxiv.org/html/2404.02183v1#bib.bib12)) | ✘ | ✘ | 17.1 |
| Incoder   Fried等人 ([2023](https://arxiv.org/html/2404.02183v1#bib.bib6)) | ✘ | ✘ | 15.2 |
| Codex   Chen等人 ([2021](https://arxiv.org/html/2404.02183v1#bib.bib4)) | ✘ | ✘ | 47.0 |
| Gemini Pro   Anil等人 ([2023](https://arxiv.org/html/2404.02183v1#bib.bib1)) | ✘ | ✘ | 67.7 |
| GPT-3.5 | CoT Wei等人 ([2022](https://arxiv.org/html/2404.02183v1#bib.bib22)) | ✘ | ✘ | 44.6 |
| ChatGPT | ✘ | ✘ | 57.3 |
| Self-Edit Zhang等人 ([2023](https://arxiv.org/html/2404.02183v1#bib.bib25)) | ✔ | ✘ | 62.2 |
| Reflexion Shinn等人 ([2023](https://arxiv.org/html/2404.02183v1#bib.bib20)) | ✔ | ✘ | 66.5 |
| SoA（我们的） | ✔ | ✔ | 71.4 |

表1：SoA与基准方法在HumanEval上的结果。ChatGPT的得分来自Dong等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib5)）。SD表示代理是否使用了带有单元测试的自我调试，而SO表示代理是否采用了自组织的多代理协作。

### §​ 4.1 主要结果

[表1](https://arxiv.org/html/2404.02183v1#S4.T1 "Table 1 ‣ Data and Tasks ‣ §​ 4 Experiments ‣ Self-Organized Agents: A LLM Multi-Agent Framework toward Ultra Large-Scale Code Generation and Optimization") 比较了所提方法和基准的Pass@1准确率。将SoA与强基准Reflexion进行比较，SoA在Pass@1上超越Reflexion 5%。考虑到SoA中的每个智能体并没有看到整个代码，这一结果令人惊讶。这一结果表明，自组织智能体能够生成一个整体运行良好的代码，而无需监督整个代码，而是通过独立实现分配给它们的功能。

### §​ 4.2 分析

我们研究中最关键的一个方面是自组织多智能体方法在大规模代码生成中的效率。为了展示SoA的优越性能，我们进行了全面的对比分析，比较了最先进的单智能体系统Reflexion与我们提出的多智能体系统。通过使用HumanEval基准，我们仔细检查了两种系统生成的代码总体规模，以及每个智能体独立生成和记忆的代码量。为了确保公平比较，我们从HumanEval结果中去除了注释和文档字符串，并专注于纯代码的字符数和标记数。

[图5](https://arxiv.org/html/2404.02183v1#S4.F5 "Figure 5 ‣ §​ 4.2 Analysis ‣ §​ 4 Experiments ‣ Self-Organized Agents: A LLM Multi-Agent Framework toward Ultra Large-Scale Code Generation and Optimization")展示了从单个函数和所有函数的角度来看，SoA与Reflexion平均生成的代码量的可视化。在HumanEval的背景下，HumanEval要求实现一个单一的函数，SoA的代码量是通过将每个代理生成的代码相加来计算的，而Reflexion的代码量则基于单个函数。SoA中每个函数的*代码量*是指每个代理生成的代码量，而在Reflexion中，它等同于单个函数的代码量。结果明确表明，在每个最终代码的标记数和每个函数的平均字符数方面，SoA明显优于Reflexion。值得注意的是，尽管SoA中每个代理处理的标记/字符数量明显少于Reflexion中的单一代理，SoA的整体输出却显著更大。这一发现凸显了SoA的卓越可扩展性，表明它通过无缝增加更多代理来处理越来越复杂的任务。我们的结果表明，通过增加代理层次的深度并引入更多母代理，SoA可以通过高效分配工作负载在多个代理之间生成更大规模的代码。随着树结构变得更深，系统展现出无限的扩展潜力，能够生成越来越复杂和庞大的代码库，同时确保每个代理处理一部分可管理的代码。每个代理可以保持可管理的代码量，同时理论上允许总体代码生成能力的无限增长。

这种分布式方法使得SoA能够显著扩展其处理大规模和复杂编码任务的能力，具有卓越的效率和高质量，远远超越了单代理系统如Reflexion所面临的局限性，其中一个代理负责管理和生成整个代码库。

图5：SoA（多代理）与Reflexion（单代理）之间的代码生成量比较。

## §​ 5 相关工作

#### LLM代理

最近，在LLM智能体领域取得了进展，如ReAct Yao et al. ([2023b](https://arxiv.org/html/2404.02183v1#bib.bib24))、Reflexion Shinn et al. ([2023](https://arxiv.org/html/2404.02183v1#bib.bib20))、Toolformer Schick et al. ([2023](https://arxiv.org/html/2404.02183v1#bib.bib18))和Self-Refine Madaan et al. ([2023](https://arxiv.org/html/2404.02183v1#bib.bib13))，这些方法主要集中于单一智能体方法，其中一个智能体负责生成和修改任务。在这些方法中，Reflexion Shinn et al. ([2023](https://arxiv.org/html/2404.02183v1#bib.bib20))因其在代码生成领域的出色表现而引起了广泛关注。然而，尽管这些方法有其优点，它们在生成和修改大规模代码库时仍面临固有的局限性。为了克服这些局限，并推动LLM智能体技术的边界，我们提出了SoA，这是一个新颖的多智能体框架，能够利用自组织和协作的力量。尽管在本研究中我们故意为SoA采用了简单的智能体，我们的框架足够灵活，可以整合更复杂和更强大的方法，如Zhong et al. ([2024](https://arxiv.org/html/2404.02183v1#bib.bib26))；Zhou et al. ([2023](https://arxiv.org/html/2404.02183v1#bib.bib27))以及其他最先进的LLM技术⁵⁵5[https://claude.ai/](https://claude.ai/)，进一步增强其在大规模代码生成和修改方面的潜力。

#### 软件开发中的多智能体协作

近年来，一些基于多智能体的方法作为软件开发的有前景的解决方案相继出现，例如MetaGPT Hong et al. ([2023](https://arxiv.org/html/2404.02183v1#bib.bib7))、ChatDev Qian et al. ([2023](https://arxiv.org/html/2404.02183v1#bib.bib17))、Self-collaboration Dong et al. ([2023](https://arxiv.org/html/2404.02183v1#bib.bib5))和AgentCoder Huang et al. ([2023](https://arxiv.org/html/2404.02183v1#bib.bib8))。这些方法通常将智能体拟人化，并为其分配特定的名称或职业角色，例如程序员、项目经理或QA工程师，以分配任务。虽然这种方法展现出了一定的前景，我们的方法采取了不同且更为灵活的方式。我们并不为智能体分配固定的职业角色，而是根据*代码功能性*对智能体的能力进行细分，使每个智能体能够展示其专业技能，而不受预定义角色的限制。这种精细化的任务分配使得问题解决更具灵活性，并能适应软件开发过程中的复杂性。此外，通过结合自组织和自繁衍的概念，我们的智能体能够根据当前问题的难度动态扩展整体代码量，为大规模代码生成和修改提供了一个高度适应性和高效的框架。

#### 宏观与微观视角

虽然基于多智能体的方法，如Hong等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib7)）；Qian等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib17)）；Dong等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib5)）；Huang等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib8)）和我们提出的SoA框架，具有自动化软件开发的共同目标，但它们解决的是软件开发过程中的不同技术方面。现有的多智能体方法主要关注优化软件开发的宏观结构，如项目管理和任务分配。相比之下，我们的方法采取了更微观的视角，专注于代码生成和修改的基础技术。这些方法并非互相排斥，而是互为补充，为自动化软件开发面临的挑战提供了更全面的解决方案。通过结合宏观和微观方法的优势，我们可以创建一个强大而全面的框架，能够高效处理大规模代码生成和修改的复杂性。

#### 提示工程

思维树（ToT）Yao等人（[2023a](https://arxiv.org/html/2404.02183v1#bib.bib23)）和思维骨架（SoT）Ning等人（[2023](https://arxiv.org/html/2404.02183v1#bib.bib15)）是利用树状结构的提示工程技术。ToT将推理步骤表示为节点，以探索正确的推理路径，而SoT则生成答案的骨架，并并行完成内容以减少生成延迟。相比之下，SoA使用以智能体为节点的树形结构，重点是它们的协作和自组织，以高效生成和修改代码。

## §​ 6 结论

在本研究中，我们介绍了自组织智能体（SoA），一种新型的多智能体框架，旨在利用大语言模型（LLMs）实现高效且可扩展的自动代码生成和优化。SoA通过利用自组织和分布式代码生成的能力，解决了单一智能体方法在处理大规模复杂代码库时的局限性。在SoA中，自组织智能体独立操作，生成和修改代码组件，同时无缝协作以构建整体代码库。我们框架的一个关键特点是根据问题复杂度自动增加智能体的数量，从而实现动态可扩展性，允许整体代码量根据智能体的数量无限增加，而每个智能体管理的代码量保持不变。

我们在 HumanEval 基准上评估了 SoA，并展示了其优于 Reflexion（一个先进的单一代理系统）的表现，SoA 在 Pass@1 准确率上提升了 5%。此外，我们的深入分析显示了 SoA 显著的可扩展性，因为 SoA 中的每个代理处理的代码远少于单一代理基准系统，但整体生成的代码量却大大增加。这些结果突显了 SoA 在高效且高质量地生成和优化大规模代码方面的有效性。

然而，必须承认当前 SoA 实现的局限性。框架的性能可能会受到 LLM 选择以及生成的单元测试质量的影响。此外，SoA 仅在有限的编程任务集上进行了评估，其在处理更复杂的实际软件开发项目中的有效性仍需进一步研究。此外，SoA 中代理之间的通信与协作机制可以进一步优化，以提高效率和容错能力。

尽管存在这些局限性，我们认为 SoA 框架在自动化软件开发领域的未来研究与发展中具有重要潜力。

## 参考文献

+   Anil 等人（2023）Rohan Anil、Sebastian Borgeaud、Yonghui Wu、Jean-Baptiste Alayrac、Jiahui Yu、Radu Soricut、Johan Schalkwyk、Andrew M. Dai、Anja Hauth、Katie Millican、David Silver、Slav Petrov、Melvin Johnson、Ioannis Antonoglou、Julian Schrittwieser、Amelia Glaese、Jilin Chen、Emily Pitler、Timothy P. Lillicrap、Angeliki Lazaridou、Orhan Firat、James Molloy、Michael Isard、Paul Ronald Barham、Tom Hennigan、Benjamin Lee、Fabio Viola、Malcolm Reynolds、Yuanzhong Xu、Ryan Doherty、Eli Collins、Clemens Meyer、Eliza Rutherford、Erica Moreira、Kareem Ayoub、Megha Goel、George Tucker、Enrique Piqueras、Maxim Krikun、Iain Barr、Nikolay Savinov、Ivo Danihelka、Becca Roelofs、Anaïs White、Anders Andreassen、Tamara von Glehn、Lakshman Yagati、Mehran Kazemi、Lucas Gonzalez、Misha Khalman、Jakub Sygnowski 等人。2023年。[Gemini: 一种高效能多模态模型家族](https://doi.org/10.48550/ARXIV.2312.11805)。*CoRR*，abs/2312.11805。

+   Ashby（1947）W·Ross Ashby。1947年。自组织动态系统的原理。*普通心理学杂志*，37(2):125–128。

+   Brown 等人（2020）汤姆·B·布朗，本杰明·曼，尼克·莱德，梅拉妮·苏比亚，贾里德·卡普兰，普拉弗·达里瓦尔，阿尔文·尼拉坎坦，普拉纳夫·夏姆，吉里什·萨斯特里，阿曼达·阿斯克尔，桑迪尼·阿贾尔瓦尔，阿里尔·赫伯特-沃斯，格雷琴·克鲁格，汤姆·亨尼汉，瑞温·查尔德，阿迪蒂亚·拉梅什，丹尼尔·M·齐格勒，杰弗里·吴，克莱门斯·温特，克里斯托弗·赫塞，马克·陈，埃里克·西格勒，马特乌什·利特温，斯科特·格雷，本杰明·切斯，杰克·克拉克，克里斯托弗·伯纳，萨姆·麦肯德里什，亚历克·拉德福，伊利亚·苏茨克维尔，和达里奥·阿莫代伊。2020年。[语言模型是少样本学习者](https://proceedings.neurips.cc/paper/2020/hash/1457c0d6bfcb4967418bfb8ac142f64a-Abstract.html)。发表于*神经信息处理系统进展 33：2020年神经信息处理系统年会，NeurIPS 2020，2020年12月6日至12日，虚拟会议*。

+   Chen 等人（2021）马克·陈，杰瑞·特沃雷克，希乌·俊，齐名·袁，亨里克·庞德·德·奥利维拉·平托，贾里德·卡普兰，哈里森·爱德华兹，尤里·布尔达，尼古拉斯·约瑟夫，格雷格·布罗克曼，亚历克斯·雷，劳尔·普里，格雷琴·克鲁格，迈克尔·彼得罗夫，海迪·克拉夫，吉里什·萨斯特里，帕梅拉·米什金，布鲁克·陈，斯科特·格雷，尼克·莱德，米哈伊尔·帕夫洛夫，阿莱西娅·鲍尔，卢卡斯·凯泽，穆罕默德·巴瓦里安，克莱门斯·温特，菲利普·蒂莱，费利佩·佩特罗斯基·苏赫，戴夫·卡明斯，马蒂亚斯·普拉普特，福蒂奥斯·昌茨，伊丽莎白·巴恩斯，阿里尔·赫伯特-沃斯，威廉·赫本·古斯，亚历克斯·尼科尔，亚历克斯·帕诺，尼科拉斯·特扎克，季·唐，伊戈尔·巴布什金，苏奇尔·巴拉吉，尚塔努·贾因，威廉·桑德斯，克里斯托弗·赫塞，安德鲁·N·卡尔，詹·莱克，约书亚·阿奇姆，维丹特·米斯拉，埃文·莫里卡瓦，亚历克·拉德福，马修·奈特，迈尔斯·布伦德奇，米拉·穆拉蒂，凯蒂·梅耶，彼得·维林德，鲍勃·麦格鲁，达里奥·阿莫代伊，萨姆·麦肯德里什，伊利亚·苏茨克维尔，和沃伊切赫·扎伦巴。2021年。[评估在代码上训练的大型语言模型](http://arxiv.org/abs/2107.03374)。*CoRR*, abs/2107.03374。

+   Dong 等人（2023）易洪·董，雪·姜，智·金，格·李。2023年。[通过 ChatGPT 自我协作生成代码](https://doi.org/10.48550/ARXIV.2304.07590)。*CoRR*, abs/2304.07590。

+   Fried 等人（2023）丹尼尔·弗里德，阿门·阿哈贾扬扬，杰西·林，思达·王，埃里克·华莱士，弗雷达·史，瑞奇·钟，斯科特·易，卢克·泽特尔莫耶，和迈克·刘易斯。2023年。[Incoder：用于代码填充和生成的生成模型](https://openreview.net/pdf?id=hQwb-lbM6EL)。发表于*第十一届国际学习表征会议，ICLR 2023，卢旺达基加利，2023年5月1日至5日*。OpenReview.net。

+   Hong 等人（2023）思瑞·洪，夏吴·郑，乔纳森·陈，宇恒·程，金林·王，策耀·张，子立·王，史蒂文·卡·辛·姚，子娟·林，丽阳·周，晨宇·冉，凌峰·肖，和承琳·吴。2023年。[Metagpt：用于多智能体协作框架的元编程](https://doi.org/10.48550/ARXIV.2308.00352)。*CoRR*, abs/2308.00352。

+   Huang等人（2023）Dong Huang, Qingwen Bu, Jie M. Zhang, Michael Luck, 和 Heming Cui. 2023. [Agentcoder：基于多代理的代码生成与迭代测试和优化](https://doi.org/10.48550/ARXIV.2312.13010). *CoRR*, abs/2312.13010.

+   Levy等人（2024）Mosh Levy, Alon Jacoby, 和 Yoav Goldberg. 2024. [相同任务，更多令牌：输入长度对大型语言模型推理表现的影响](https://doi.org/10.48550/ARXIV.2402.14848). *CoRR*, abs/2402.14848.

+   Li等人（2023a）Jiaqi Li, Mengmeng Wang, Zilong Zheng, 和 Muhan Zhang. 2023a. [Loogle：长篇上下文语言模型能否理解长上下文？](https://doi.org/10.48550/ARXIV.2311.04939) *CoRR*, abs/2311.04939.

+   Li等人（2023b）Raymond Li, Loubna Ben Allal, Yangtian Zi, Niklas Muennighoff, Denis Kocetkov, Chenghao Mou, Marc Marone, Christopher Akiki, Jia Li, Jenny Chim, Qian Liu, Evgenii Zheltonozhskii, Terry Yue Zhuo, Thomas Wang, Olivier Dehaene, Mishig Davaadorj, Joel Lamy-Poirier, João Monteiro, Oleh Shliazhko, Nicolas Gontier, Nicholas Meade, Armel Zebaze, Ming-Ho Yee, Logesh Kumar Umapathi, Jian Zhu, Benjamin Lipkin, Muhtasham Oblokulov, Zhiruo Wang, Rudra Murthy V, Jason Stillerman, Siva Sankalp Patel, Dmitry Abulkhanov, Marco Zocca, Manan Dey, Zhihan Zhang, Nour Moustafa-Fahmy, Urvashi Bhattacharyya, Wenhao Yu, Swayam Singh, Sasha Luccioni, Paulo Villegas, Maxim Kunakov, Fedor Zhdanov, Manuel Romero, Tony Lee, Nadav Timor, Jennifer Ding, Claire Schlesinger, Hailey Schoelkopf, Jan Ebert, Tri Dao, Mayank Mishra, Alex Gu, Jennifer Robinson, Carolyn Jane Anderson, Brendan Dolan-Gavitt, Danish Contractor, Siva Reddy, Daniel Fried, Dzmitry Bahdanau, Yacine Jernite, Carlos Muñoz Ferrandis, Sean Hughes, Thomas Wolf, Arjun Guha, Leandro von Werra, 和 Harm de Vries. 2023b. [Starcoder: 愿源代码与你同在！](https://doi.org/10.48550/ARXIV.2305.06161) *CoRR*, abs/2305.06161.

+   Li等人（2022）Yujia Li, David H. Choi, Junyoung Chung, Nate Kushman, Julian Schrittwieser, Rémi Leblond, Tom Eccles, James Keeling, Felix Gimeno, Agustin Dal Lago, Thomas Hubert, Peter Choy, Cyprien de Masson d’Autume, Igor Babuschkin, Xinyun Chen, Po-Sen Huang, Johannes Welbl, Sven Gowal, Alexey Cherepanov, James Molloy, Daniel J. Mankowitz, Esme Sutherland Robson, Pushmeet Kohli, Nando de Freitas, Koray Kavukcuoglu, 和 Oriol Vinyals. 2022. [竞争级别的代码生成与alphacode](https://doi.org/10.48550/ARXIV.2203.07814). *CoRR*, abs/2203.07814.

+   Madaan 等人（2023）Aman Madaan、Niket Tandon、Prakhar Gupta、Skyler Hallinan、Luyu Gao、Sarah Wiegreffe、Uri Alon、Nouha Dziri、Shrimai Prabhumoye、Yiming Yang、Shashank Gupta、Bodhisattwa Prasad Majumder、Katherine Hermann、Sean Welleck、Amir Yazdanbakhsh 和 Peter Clark。2023年。[Self-refine: 通过自我反馈进行迭代优化](http://papers.nips.cc/paper_files/paper/2023/hash/91edff07232fb1b55a505a9e9f6c0ff3-Abstract-Conference.html)。发表于 *神经信息处理系统进展 36：2023年神经信息处理系统年会 NeurIPS 2023，美国路易斯安那州新奥尔良，2023年12月10日至16日*。

+   Muennighoff 等人（2023）Niklas Muennighoff、Qian Liu、Armel Zebaze、Qinkai Zheng、Binyuan Hui、Terry Yue Zhuo、Swayam Singh、Xiangru Tang、Leandro von Werra 和 Shayne Longpre。2023年。[Octopack: 指令调优大规模语言模型](https://doi.org/10.48550/ARXIV.2308.07124)。*CoRR*，abs/2308.07124。

+   Ning 等人（2023）Xuefei Ning、Zinan Lin、Zixuan Zhou、Huazhong Yang 和 Yu Wang。2023年。[Skeleton-of-thought: 大型语言模型可以进行并行解码](https://doi.org/10.48550/ARXIV.2307.15337)。*CoRR*，abs/2307.15337。

+   OpenAI（2023）OpenAI。2023年。[GPT-4 技术报告](https://doi.org/10.48550/arXiv.2303.08774)。*CoRR*，abs/2303.08774。

+   Qian 等人（2023）Chen Qian、Xin Cong、Cheng Yang、Weize Chen、Yusheng Su、Juyuan Xu、Zhiyuan Liu 和 Maosong Sun。2023年。[软件开发的交流代理](https://doi.org/10.48550/ARXIV.2307.07924)。*CoRR*，abs/2307.07924。

+   Schick 等人（2023）Timo Schick、Jane Dwivedi-Yu、Roberto Dessì、Roberta Raileanu、Maria Lomeli、Eric Hambro、Luke Zettlemoyer、Nicola Cancedda 和 Thomas Scialom。2023年。[Toolformer: 语言模型可以自学使用工具](http://papers.nips.cc/paper_files/paper/2023/hash/d842425e4bf79ba039352da0f658a906-Abstract-Conference.html)。发表于 *神经信息处理系统进展 36：2023年神经信息处理系统年会 NeurIPS 2023，美国路易斯安那州新奥尔良，2023年12月10日至16日*。

+   Shaham 等人（2023）Uri Shaham、Maor Ivgi、Avia Efrat、Jonathan Berant 和 Omer Levy。2023年。[Zeroscrolls: 一种零-shot长文本理解基准](https://aclanthology.org/2023.findings-emnlp.536)。发表于 *计算语言学协会的发现：2023年EMNLP会议，新加坡，2023年12月6日至10日*，第7977–7989页。计算语言学协会。

+   Shinn 等人（2023）Noah Shinn、Federico Cassano、Ashwin Gopinath、Karthik Narasimhan 和 Shunyu Yao。2023年。[Reflexion: 使用语言代理和口头强化学习](http://papers.nips.cc/paper_files/paper/2023/hash/1b44b878bb782e6954cd888628510e90-Abstract-Conference.html)。发表于 *神经信息处理系统进展 36：2023年神经信息处理系统年会 NeurIPS 2023，美国路易斯安那州新奥尔良，2023年12月10日至16日*。

+   Touvron et al. (2023) Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, Dan Bikel, Lukas Blecher, Cristian Canton-Ferrer, Moya Chen, Guillem Cucurull, David Esiobu, Jude Fernandes, Jeremy Fu, Wenyin Fu, Brian Fuller, Cynthia Gao, Vedanuj Goswami, Naman Goyal, Anthony Hartshorn, Saghar Hosseini, Rui Hou, Hakan Inan, Marcin Kardas, Viktor Kerkez, Madian Khabsa, Isabel Kloumann, Artem Korenev, Punit Singh Koura, Marie-Anne Lachaux, Thibaut Lavril, Jenya Lee, Diana Liskovich, Yinghai Lu, Yuning Mao, Xavier Martinet, Todor Mihaylov, Pushkar Mishra, Igor Molybog, Yixin Nie, Andrew Poulton, Jeremy Reizenstein, Rashi Rungta, Kalyan Saladi, Alan Schelten, Ruan Silva, Eric Michael Smith, Ranjan Subramanian, Xiaoqing Ellen Tan, Binh Tang, Ross Taylor, Adina Williams, Jian Xiang Kuan, Puxin Xu, Zheng Yan, Iliyan Zarov, Yuchen Zhang, Angela Fan, Melanie Kambadur, Sharan Narang, Aurélien Rodriguez, Robert Stojnic, Sergey Edunov, 和 Thomas Scialom. 2023. [Llama 2: Open foundation and fine-tuned chat models](https://doi.org/10.48550/arXiv.2307.09288). *CoRR*, abs/2307.09288。

+   Wei et al. (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, Fei Xia, Ed H. Chi, Quoc V. Le, 和 Denny Zhou. 2022. [Chain-of-thought prompting elicits reasoning in large language models](http://papers.nips.cc/paper_files/paper/2022/hash/9d5609613524ecf4f15af0f7b31abca4-Abstract-Conference.html). 载于 *神经信息处理系统进展 35：2022年神经信息处理系统年会，NeurIPS 2022，美国路易斯安那州新奥尔良，2022年11月28日 - 12月9日*。

+   Yao et al. (2023a) Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Tom Griffiths, Yuan Cao, 和 Karthik Narasimhan. 2023a. [Tree of thoughts: Deliberate problem solving with large language models](http://papers.nips.cc/paper_files/paper/2023/hash/271db9922b8d1f4dd7aaef84ed5ac703-Abstract-Conference.html). 载于 *神经信息处理系统进展 36：2023年神经信息处理系统年会，NeurIPS 2023，美国路易斯安那州新奥尔良，2023年12月10日 - 16日*。

+   Yao et al. (2023b) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R. Narasimhan, 和 Yuan Cao. 2023b. [React: Synergizing reasoning and acting in language models](https://openreview.net/pdf?id=WE_vluYUL-X). 载于 *第十一届国际学习表示大会，ICLR 2023，卢旺达基加利，2023年5月1-5日*。OpenReview.net。

+   Zhang et al. (2023) Kechi Zhang, Zhuo Li, Jia Li, Ge Li, 和 Zhi Jin. 2023. [Self-edit: Fault-aware code editor for code generation](https://doi.org/10.18653/V1/2023.ACL-LONG.45). 载于 *第61届计算语言学协会年会（第一卷：长篇论文），ACL 2023，加拿大多伦多，2023年7月9-14日*，第769-787页。计算语言学协会。

+   Zhong 等人（2024）Lily Zhong、Zilong Wang 和 Jingbo Shang。2024年。[LDB：通过逐步验证运行时执行来调试大语言模型](https://doi.org/10.48550/ARXIV.2402.16906)。*CoRR*，abs/2402.16906。

+   Zhou 等人（2023）Andy Zhou、Kai Yan、Michal Shlapentokh-Rothman、Haohan Wang 和 Yu-Xiong Wang。2023年。[语言代理树搜索统一了语言模型中的推理、行动和规划](https://doi.org/10.48550/ARXIV.2310.04406)。*CoRR*，abs/2310.04406。

## 附录 A 伪代码

算法 1 使用自组织代理框架生成代码

1:$docstrings$: 函数的文档字符串，$unit\_tests$: 单元测试列表，$max\_depth$: 代理层级的最大深度，$max\_iterations$: 最大的代码修改迭代次数2: 生成的最终代码3:4:初始化根母代理，使用$docstrings$和$unit\_tests$5:6:函数 GenerateAgent($agent$, $depth$, $subtask\_docstrings$, $subtask\_unit\_tests$)7:     如果 $depth-1$ = $max\_depth$ 则8:         $next\_agent\leftarrow$ 新建子代理9:     否则10:         $next\_agent\leftarrow$ 新建母代理11:     结束 如果12:     将$subtask\_docstrings$和$unit\_tests$分配给$next\_agent$13:     生成($next\_agent$, $depth+1$)14:结束 函数15:16:函数 Generate($agent$, $depth$)17:     如果 $depth$ = 1 则 $\triangleright$ 根母代理18:         $skeleton\leftarrow$ 从$agent.docstrings$和$agent.unit_{t}ests$生成骨架19:         $agent.code\leftarrow$ 骨架20:         对每个 $subtask\_docstrings$, $subtask\_unit\_tests$ 在子任务中做21:              生成代理($agent$, $depth$, $subtask\_docstrings$, $subtask\_unit\_tests$)22:         结束 对每个23:     否则如果 $depth$ = $max\_depth$ 则 $\triangleright$ 子代理24:         生成代码，针对 $agent.subtask\_docstrings$ 和 $agent.subtask\_unit\_tests$25:         $agent.code\leftarrow$ 生成的代码26:     否则 $\triangleright$ 母代理27:         生成代码，针对 $agent.subtask\_docstrings$ 和 $agent.subtask\_unit\_tests$28:         $agent.code\leftarrow$ 生成的代码29:         对每个 $subtask\_docstrings$, $subtask\_unit\_tests$ 在子任务中做30:              生成代理($agent$, $depth$, $subtask\_docstrings$, $subtask\_unit\_tests$)31:         结束 对每个32:     结束 如果33:结束 函数34:35:函数 Modify($agent$, $test\_result$, $upper\_agent\_observation$)36:     基于 $test\_result$ 和 $upper\_agent\_observation$ 为 $agent$ 生成反馈37:     基于反馈更新 $agent$ 的代码38:     对每个 $subagent$ 在 $agent.subagents$ 中做39:         使用 $subagent.unit\_tests$ 评估 $subagent.code$，得到 $subagent\_test\_result$40:         修改($subagent$, $subagent\_test\_result$, 反馈和代码更改)41:     结束 对每个42:结束 函数43:44:从 Generate($root\_mother$, 1) 开始代码生成45:46:对每次迭代在 $max\_iterations$ 中做47:     合并所有代理的实现，创建 $final\_implementation$48:     使用 $unit\_tests$ 评估 $final\_implementation$，得到 $test\_result$49:     从 $root\_mother$ 开始修改代码，使用 Modify($root\_mother$, $test\_result$, None)50:结束 对每次迭代51:52:返回 由所有代理合并的最终实现
