<!--yml

分类：未分类

日期：2025-01-11 12:14:26

-->

# 提升LLM推理能力：多代理思想树验证者代理

> 来源：[https://arxiv.org/html/2409.11527/](https://arxiv.org/html/2409.11527/)

法特梅赫·哈吉

安全人工智能与自治实验室

德克萨斯大学圣安东尼奥分校

fatemeh.haji@utsa.edu

&马扎尔·贝瑟妮

安全人工智能与自治实验室

德克萨斯大学圣安东尼奥分校

mazal.bethany@utsa.edu

&玛丽亚姆·塔巴尔

德克萨斯大学圣安东尼奥分校

maryam.tabar@utsa.edu

&周宇·杰森·蒋

Peraton Labs

jchiang@peratonlabs.com

&安东尼·里奥斯

德克萨斯大学圣安东尼奥分校

anthony.rios@utsa.edu

&佩伊曼·纳贾菲拉德

安全人工智能与自治实验室

德克萨斯大学圣安东尼奥分校

peyman.najafirad@utsa.edu

###### 摘要

多代理策略作为一种有前景的方法，通过在问题解决过程中分配专业角色，提升了大型语言模型（LLMs）的推理能力。同时，思想树（ToT）方法通过探索多样的推理路径，在复杂的问答任务中展现出改善推理的潜力。多代理推理的一个关键局限性是“推理者”（Reasoner）代理对推理路径的浅层探索。尽管ToT策略有助于缓解这一问题，但它们可能会生成错误的推理分支，从而损害最终答案的可信度。为了发挥多代理推理和ToT策略的优势，我们提出了一种新方法，结合了基于ToT的推理者代理和思想验证者代理（Thought Validator）。多个推理者代理并行工作，利用ToT探索多样的推理路径。然后，思想验证者对这些路径进行审查，只有在推理有效时，才考虑推理者的结论。这种方法通过丢弃错误的推理路径，实现了更为强大的投票策略，增强了系统处理需要系统性和可信推理任务的能力。在GSM8K数据集上的评估结果表明，我们的方法相比现有技术表现出更优的性能，在四种LLM中，平均提升了5.6%，优于标准的ToT策略。相关代码和内容可以在以下链接找到：[https://github.com/SecureAIAutonomyLab/MA-ToT](https://github.com/SecureAIAutonomyLab/MA-ToT)

## 1 引言

大型语言模型（LLMs）在各种任务中展现了卓越的能力，但在复杂推理方面常常存在困难，尤其是在需要类似人类推理能力的情境中[[18](https://arxiv.org/html/2409.11527v2#bib.bib18)]。为了解决这一问题，多智能体策略作为一种有前景的方法出现，旨在增强LLM的推理能力。通过使用多智能体策略，多个专门化的智能体协作，每个智能体在问题解决过程中扮演不同的角色。通过允许不同的智能体处理任务的各个方面，我们可以利用每个阶段的专业知识来提升性能[[5](https://arxiv.org/html/2409.11527v2#bib.bib5)]。研究表明，这种方法能够提高在推理密集型领域中的答案质量。然而，尽管多智能体推理有其潜力，仍然存在一个关键限制：推理智能体往往对推理路径的探索较为浅显，未能充分考虑问题空间的复杂性。思维树（ToT）方法提供了一种可能的解决方案，通过鼓励对多条推理路径的更系统性探索，来克服这一限制。ToT通过模拟人类思维过程，允许LLM分支并检查多种可能性，然后再集中于一个解决方案[[17](https://arxiv.org/html/2409.11527v2#bib.bib17)]。通过使LLM考虑多种推理路径，ToT可以缓解一些多智能体系统中出现的浅层探索问题。然而，尽管ToT鼓励探索，它也带来了一个新的挑战：生成错误推理分支的风险。如果没有适当的验证，这些错误路径可能会降低最终答案的可信度。

为了解决这些挑战，我们提出了一种新颖的方法，将多智能体推理的优势与ToT结合，同时引入了一个关键的验证机制。在我们的框架中，多个推理智能体并行操作，每个智能体使用ToT探索不同的推理路径。这些推理智能体由一个思维验证智能体支持，该智能体评估推理智能体提出的推理分支。验证智能体会丢弃错误的推理分支，确保只有逻辑上合理的路径才会对最终决策产生影响。这种方法既允许探索问题空间，又通过在错误的推理路径影响结果之前将其剔除，增加了答案的可靠性。我们在GSM8K数据集[[1](https://arxiv.org/html/2409.11527v2#bib.bib1)]上评估了我们提出的方法，该数据集以其具有挑战性的算术推理任务而闻名。结果表明，我们的方法优于现有技术，在解决复杂推理问题时表现出更高的准确性和可信度。

我们的主要贡献如下：

+   •

    将ToT集成到多智能体推理框架中。

+   •

    引入了一种新颖的思维验证代理，用于评估和过滤由推理者代理产生的推理分支。

+   •

    在GSM8K数据集上的实验结果，展示了与现有技术相比，在复杂算术推理任务中提高的准确性和表现。

## 2 背景

### 2.1 增强LLM推理的多智能体系统

最近的研究展示了多种方法来增强大语言模型（LLM）的输出。过度生成和重新排序策略已显示出有希望的结果，其中RANKGEN [[6](https://arxiv.org/html/2409.11527v2#bib.bib6)]引入了一个排名模型，根据相关性和连贯性对生成的输出进行评分，而信实感知解码 [[14](https://arxiv.org/html/2409.11527v2#bib.bib14)]展示了不同解码策略如何影响输出质量。尽管这些方法对于一般文本生成有效，但复杂的推理任务通常需要更结构化的方法。多智能体系统作为一个特别有效的框架，已在推理任务中提高了表现，通过将任务分配给专门的代理 [[2](https://arxiv.org/html/2409.11527v2#bib.bib2), [12](https://arxiv.org/html/2409.11527v2#bib.bib12)]。例如，CausalGPT [[13](https://arxiv.org/html/2409.11527v2#bib.bib13)]引入了评估层来验证LLM生成的推理分支，而反事实多智能体辩论（CFMAD）框架 [[4](https://arxiv.org/html/2409.11527v2#bib.bib4)]则通过将代理分配到固定角色，生成来自特定视角的辩护，并由第三方评审员评估这些论点，从而提供了一种创新的方法来减轻LLM推理分支的潜在偏差，决定最合理的结果。尽管这些进展取得了一定成效，但目前的方法仍然存在推理路径抽样浅显或多数投票方案的缺陷。这些技术可能忽略关键的推理错误，尤其容易出现早期阶段的错误，这些错误可能会在多个推理回合中传播。在复杂情境中，这一局限尤其成问题，因为在这些场景中，系统性地评估并排除不正确的选项至关重要。最近的研究表明，LLM能够有效识别事实性和推理性错误 [[7](https://arxiv.org/html/2409.11527v2#bib.bib7)]，这使得在多智能体系统中集成专门的验证组件，尤其对于评估生成解决方案的信实性和可靠性，具有特别重要的意义。

### 2.2 多智能体框架中“推理者”代理的角色

在多智能体架构中，推理器（Reasoner）智能体起着关键作用。它作为系统的核心决策者，确保从推理过程中得出有效的结论。然而，当前框架中的推理器智能体通常难以系统性地评估并排除错误的推理路径，特别是在更具挑战性的问题空间中。这个瓶颈突显了将更先进的推理策略集成到推理器智能体中的必要性。CFMAD 还表明，检查所有可用选项可以增强多智能体系统的整体能力 [[4](https://arxiv.org/html/2409.11527v2#bib.bib4)]。

## 3 方法

我们提出了一种新型的多智能体推理框架，将 ToT 策略与强大的验证机制结合，以增强复杂问题的解决能力。我们的方法采用多个并发的推理器智能体，每个智能体都使用 ToT 探索多种推理路径。在每个树层级上，状态评估智能体对生成的推理进行评分，得分最高的推理将在下一层级中得到扩展。当达到最终树层级时，每个推理器智能体都会生成一个由树中得分最高的推理链组成的提议推理链。这些推理分支随后由思维验证器智能体独立评估，以验证或使其无效。然后，我们使用基于共识的投票机制，验证的推理路径参与投票，而无效的路径则不参与。如果未达成共识，我们将启动一个新的推理回合，结合思维验证器对推理分支的反馈，精炼下一轮推理。我们提出的框架如图 [1](https://arxiv.org/html/2409.11527v2#S3.F1 "图 1 ‣ 3 方法 ‣ 改进 LLM 推理与多智能体树状思维验证器智能体") 所示。

![参见说明](img/b309ae92b09faa614ec82db26e6231ec.png)

图 1：该过程从一个查询开始，多个推理器智能体对其进行处理。每个推理器智能体使用 ToT 策略探索不同的推理路径，ToT 策略包括思维步骤的分解、路径的生成、状态评估和路径选择。然后，思维验证器（Thought Validator）智能体评估提出的推理分支，接着进行基于共识的投票机制。如果未达成共识，则会启动一个新的推理回合，并结合反馈进行修正。

### 3.1 推理器智能体

我们多代理架构中的推理代理采用了思维树（ToT）策略，该策略使推理路径能够并行结构化探索。思维树在链式思维（CoT）提示方法的基础上进行了改进[[15](https://arxiv.org/html/2409.11527v2#bib.bib15)]，通过启用并行探索和动态路径评估，提升了性能。虽然链式思维（CoT）沿着单一线性路径前进，思维树（ToT）则主动探索并评估多条推理路径，使其更适合解决那些能够从多样化思维探索中受益的复杂问题[[16](https://arxiv.org/html/2409.11527v2#bib.bib16)]。我们将推理过程形式化为在状态树上的搜索。设 $Q$ 为输入提示或查询，每个推理代理 $R_{i}$ 构建一个思维树 $T_{i}(Q)$，其中每个节点表示一个状态 $s_{t}$。一个状态 $s_{t}=[Q,z_{1},z_{2},\dots,z_{t}]$ 由问题 $Q$ 和直到此时为止的中间推理步骤的序列 $z_{1},z_{2},\dots,z_{t}$ 组成，每个步骤 $z_{j}$ 都是由语言模型生成的连贯推理单元。

第一步：思维路径的分解与生成

该过程通过使用大型语言模型（LLM）提示将其分解为中间的思维步骤。对于每个状态 $s_{t}$，下一步潜在的思维 $z_{t+1}$ 由思维生成器 $G(p_{\theta},s_{t},k)$ 生成，其中 $p_{\theta}$ 表示语言模型。推理代理从任何给定的状态 $s_{t}$ 探索多个分支，对应于推理过程的不同延续。这种方法确保了探索过程覆盖多种可能的解决方案，避免了链式思维（CoT）的线性性，并允许重新考虑早期的步骤。

第二步：状态评估与路径选择

为了评估每个状态 $s_{t}$，我们引入了一个状态评估代理，用于为生成的推理分配分数。此评估可以通过提示实现，状态评估代理评估每个推理步骤的质量和潜力。在每一层树中，选择得分最高的推理步骤进行下一层的扩展。这个过程将持续到最终的树层为止。选择机制可以形式化为：

|  | $s_{t+1}^{*}=\arg\max_{s_{t+1}}V(p_{\theta},s_{t+1})$ |  |
| --- | --- | --- |

其中 $V(p_{\theta},s_{t+1})$ 是由状态评估代理分配的评估分数。

第三步：推理分支构建

达到最终树层时，每个推理代理构建了一个提出的推理链。这个链由树中每一层的最高得分推理步骤组成。形式上，推理分支 $C_{i}$ 对于推理代理 $R_{i}$ 可以表示为：

|  | $C_{i}=[z_{1}^{,}z_{2}^{,}\dots,z_{T}^{*}]$ |  |
| --- | --- | --- |

其中 $z_{t}^{*}$ 是树中第 $t$ 层的最高得分推理步骤。

### 3.2 思维验证代理

思维验证者代理人受教师给学生提供反馈的角色启发，在评估推理代理人所产生的推理分支的有效性方面发挥着至关重要的作用。就像教师帮助学生完善他们的答案一样，这个代理人独立地评估每一个提出的推理分支，以验证或否定它。对于每个推理分支 $C_{i}$，思维验证者代理人执行几个关键步骤。首先进行逻辑一致性检查，以评估推理链的内部逻辑和连贯性，类似于教师评估学生论证的方式。接下来是事实准确性评估，用于验证推理中提出的任何事实性主张，类似于教师对学生工作进行事实核查。最后，代理人会进行完整性评估，确保推理分支充分解决了原始问题的所有方面，类似于教师确保学生的回答完整无缺地回答了问题。通过这一综合过程，思维验证者代理人确保推理分支的稳健性和可靠性，最终有助于提高最终输出的质量。基于这些评估，思维验证者为每个推理链分配一个二进制验证状态 $V_{i}$：

|  | $V_{i}=\begin{cases}1&\text{如果 }C_{i}\text{ 被验证} \\ 0&\text{如果 }C_{i}\text{ 被否定}\end{cases}$ |  |
| --- | --- | --- |

基于共识的投票机制

在验证过程之后，我们使用基于共识的投票机制来确定最终结果。只有验证过的推理分支才会参与投票，而无效的推理分支则会被弃权。共识解决方案 $S^{*}$ 可以表示为：

|  | $S^{*}=\arg\max_{S}\sum_{i=1}^{N}V_{i}\cdot\delta(S=S_{i})$ |  |
| --- | --- | --- |

其中 $S_{i}$ 表示从推理分支 $C_{i}$ 中得出的解决方案，$V_{i}$ 是 $C_{i}$ 的验证状态，$\delta$ 是一个指示函数，当解决方案匹配时返回1，否则返回0，$N$ 是推理代理人的总数。

### 3.3 迭代改进

如果未达成共识（即没有任何解决方案获得大多数有效投票），我们将启动新的推理回合。这个改进过程结合了来自思维验证者的反馈，指导下一次迭代。这个迭代过程会持续进行，直到达成共识或超过预定义的最大迭代次数。

## 4 实验

数据集：GSM8K[[1](https://arxiv.org/html/2409.11527v2#bib.bib1)] 是一个包含 8.5K 高质量、语言多样的小学数学文字题的数据集，由人工问题编写者创建。GSM8K 被广泛认为是测试大语言模型（LLM）算术推理的基准。该数据集包含复杂的、多步骤的数学文字题，挑战着 LLM 的推理和计算能力。我们的实验使用了来自 GSM8K 数据集的 500 个样本的随机子集作为测试集。遵循其他关于 LLM 推理在 GSM8K 数据集上应用的工作，我们使用准确率作为主要评估指标来评估推理方法的表现[[17](https://arxiv.org/html/2409.11527v2#bib.bib17)]。

实施细节：我们的实验涵盖了两种版本的 OpenAI GPT 模型和两种版本的 Meta Llama 3.1 模型[[3](https://arxiv.org/html/2409.11527v2#bib.bib3)]。具体来说，我们使用了来自 OpenAI 的 GPT-3.5-turbo-0125[[11](https://arxiv.org/html/2409.11527v2#bib.bib11)] 和 GPT-4o-mini-2024-07-18[[10](https://arxiv.org/html/2409.11527v2#bib.bib10)]，通过其 API 进行访问。对于 Llama 3.1 模型，我们使用了 8B[[9](https://arxiv.org/html/2409.11527v2#bib.bib9)] 和 70B[[8](https://arxiv.org/html/2409.11527v2#bib.bib8)] 参数版本。这些模型提供了不同的能力和规模，使我们能够在实验中探索模型复杂性与性能之间的不同权衡。我们在四台 Nvidia DGX A100 80 GB GPU 上进行了所有实验，并且并行运行这些实验花费了大约 18 小时。为了进行基准比较，我们采用了几种提示策略。我们首先使用了输入输出（IO）提示，这是一种标准方法，通过对任务指令进行条件化，将问题输入转换为输出。然后，我们实施了更高级的技术，包括思维链（CoT）[[15](https://arxiv.org/html/2409.11527v2#bib.bib15)] 和原始的 ToT 策略[[17](https://arxiv.org/html/2409.11527v2#bib.bib17)]。对于 ToT 实现，我们遵循了 Yao 等人[[17](https://arxiv.org/html/2409.11527v2#bib.bib17)] 在 GSM8K 数据集上使用的参数，设置了树的深度为 2，宽度为 5。为了确保基准模型之间的一致性，我们对 IO、CoT 和 ToT 使用了 1 的温度和 1 的 top_p 值。然而，对于我们的新型思维验证代理（Thought Validator Agent），我们将这些参数调整为温度 0.5 和 top_p 0.4。这一调整是为了提高思维验证代理输出的确定性，因为它的角色是验证现有的推理，而不是生成创意解决方案。

实验结果：表格 [1](https://arxiv.org/html/2409.11527v2#S4.T1 "Table 1 ‣ 4 Experiments ‣ Improving LLM Reasoning with Multi-Agent Tree-of-Thought Validator Agent") 显示了这些不同方法的性能比较。我们的结果表明，我们提出的带有思维验证器的多智能体ToT推理器优于其他推理方法，GPT-3.5-turbo的ToT性能提升了8.8个百分点（从75.4%提升到84.2%）。我们还发现，虽然ToT和其他技术在LLM在处理任务时遇到困难时（如GPT 3.5 Turbo和Llama 3.1 8B）显著提高了性能，但对于那些标准IO提示已表现出强大能力的任务（如GPT 4o mini和Llama 3.1 70B），性能差距明显缩小。这个观察结果表明，ToT的有效性可能依赖于任务的复杂性和模型的能力，其在推动模型基本能力边界的挑战性推理任务中表现出更为明显的优势。在这些情境下，ToT的效果可以类比为教师对 struggling 学生提供反馈，指导他们解决复杂问题，并强化正确的思维过程。

| 方法 | Gpt-3.5-turbo | Gpt-4o-mini | Llama3.1-8B | Llama3.1-70B |
| --- | --- | --- | --- | --- |
| 标准IO | 60.0 | 91.2 | 75.4 | 93.0 |
| CoT | 68.0 | 89.2 | 76.0 | 89.4 |
| ToT | 75.4 | 91.6 | 80.2 | 92.8 |
| 使用思维验证器的MA ToT | 84.2 | 92.2 | 89.0 | 94.8 |

表格 1：我们的多智能体ToT推理器与思维验证器在GSM8K推理数据集上与其他LLM推理方法的性能比较，评估涉及不同的LLM。

## 5 限制与结论

尽管 ToT 方法在增强推理能力方面展现出一定的前景，但我们对输出结果和推理树的观察揭示了若干限制，这些限制值得进一步研究。我们观察到的一个关键挑战是搜索空间中缺乏动态探索。Yao 等人提出的 ToT 方法[[17](https://arxiv.org/html/2409.11527v2#bib.bib17)]采用了固定的树结构宽度和深度，而我们的分析显示，这在某些场景下可能导致次优的表现。例如，在检查那些不需要大量推理就能高效解决的问题时，我们发现预定的探索深度往往引入了不必要的复杂性，可能导致推理过程中出现错误或混乱。相反，对于需要更深入分析的问题，我们观察到固定的深度证明不足，限制了模型充分探索复杂推理路径的能力。此外，我们提出的方法虽然解决了一些这些限制，但由于采用了 ToT 方法，它在计算上非常昂贵，因为该方法需要大量资源来生成和评估多个推理路径。我们的分析表明，采用 Thought Validator 的多代理 ToT 方法相比标准方法需要显著更多的计算资源。在我们对 500 个样本的评估中，每个问题的平均 token 使用量显著增加：对于 GPT-3.5-turbo，从 CoT 的 256 个 tokens 增加到 ToT 的 4000 个 tokens，而对于 GPT-4o-mini，从 341 个 tokens 增加到 10,600 个 tokens。每个 Reasoner 代理大约需要每个问题 20 次 API 调用，而我们的多代理方法将这个成本乘以代理的数量和验证步骤。尽管这种增加的计算投资带来了推理准确性的显著提高——在 GPT-3.5-turbo 上表现为 8.8 个百分点的提升——但在资源有限的环境中，这一权衡可能需要谨慎考虑。此外，尽管我们在 GSM8K 上的评估展示了我们方法在算术推理方面的有效性，但在更多推理密集型基准测试中的测试将有助于验证该方法在不同类型推理任务中的泛化能力。

Thought Validator 代理展示了在评估推理路径方面的强大能力，其效果可以通过先进的验证技术进一步增强，例如集成验证策略或元学习方法，以提高在多样化推理场景中的稳健性。此外，虽然我们在 GSM8K 上的评估展示了我们方法在数学推理方面的有效性，但在更多推理密集型基准测试中的测试将有助于验证该方法在不同类型推理任务中的泛化能力。

总之，我们提出了一种新颖的方法，将ToT策略与多代理推理框架相结合，并通过思维验证器代理增强。我们的方法通过启用更系统地探索推理路径，同时提高生成解决方案的可靠性，解决了现有推理策略在大型语言模型中的关键局限性。针对GSM8K数据集的实验结果表明，我们的方法在复杂的算术推理任务中优于现有的最先进方法。未来的工作可以探索基于问题复杂度的动态树结构，可能在更广泛的问题类型中提高效率和性能。

## 6 社会影响声明

通过提高推理深度并实现更系统的选项消除，我们的方法可能会推动更值得信赖的人工智能应用。然而，这些进展也提出了有关高度自主推理系统部署的伦理考虑，特别是在高风险领域。必须谨慎管理这些系统的使用，避免过度依赖人工智能，确保人类监督和问责始终融入决策过程中。此外，必须监控其更广泛的社会影响，防止意外后果，例如算法决策中偏见的放大或在人类专业知识要求细致判断的领域中取代人类专家。

## 参考文献

+   [1] Karl Cobbe, Vineet Kosaraju, Mohammad Bavarian, Mark Chen, Heewoo Jun, Lukasz Kaiser, Matthias Plappert, Jerry Tworek, Jacob Hilton, Reiichiro Nakano, Christopher Hesse, 和 John Schulman. 训练验证器解决数学文字题。网址 [http://arxiv.org/abs/2110.14168](http://arxiv.org/abs/2110.14168)。

+   [2] Yilun Du, Shuang Li, Antonio Torralba, Joshua B. Tenenbaum, 和 Igor Mordatch. 通过多代理辩论改善语言模型的事实性和推理能力。网址 [http://arxiv.org/abs/2305.14325](http://arxiv.org/abs/2305.14325)。

+   Dubey 等人 [2024] Abhimanyu Dubey, Abhinav Jauhri, Abhinav Pandey, Abhishek Kadian, Ahmad Al-Dahle, Aiesha Letman, Akhil Mathur, Alan Schelten, Amy Yang, Angela Fan, 等人. Llama 3 模型群体。*arXiv 预印本 arXiv:2407.21783*，2024。

+   [4] Yi Fang, Moxin Li, Wenjie Wang, Hui Lin, 和 Fuli Feng. 通过预设立场的反事实辩论来消除大型语言模型的幻觉。网址 [http://arxiv.org/abs/2406.11514](http://arxiv.org/abs/2406.11514)。

+   Guo 等人 [2024] Taicheng Guo, Xiuying Chen, Yaqi Wang, Ruidi Chang, Shichao Pei, Nitesh V. Chawla, Olaf Wiest, 和 Xiangliang Zhang. 基于大型语言模型的多代理：进展与挑战综述。在 Kate Larson 编辑的 *《第三十三届国际人工智能联合会议论文集，IJCAI-24》* 中，第8048–8057页。国际人工智能联合会议组织，2024年8月。综述篇章。

+   Krishna 等人 [2022] Kalpesh Krishna, Yapei Chang, John Wieting, 和 Mohit Iyyer. Rankgen：通过大型排序模型改进文本生成，2022年。

+   [7] Junyi Li, Xiaoxue Cheng, Wayne Xin Zhao, Jian-Yun Nie, 和 Ji-Rong Wen. HaluEval：大规模幻觉评估基准，面向大语言模型。网址 [http://arxiv.org/abs/2305.11747](http://arxiv.org/abs/2305.11747)。

+   NousResearch [2024a] NousResearch. Nousresearch/meta-llama-3.1-70b. [https://huggingface.co/NousResearch/Meta-Llama-3.1-70B](https://huggingface.co/NousResearch/Meta-Llama-3.1-70B)，2024a。

+   NousResearch [2024b] NousResearch. Nousresearch/meta-llama-3.1-8b. [https://huggingface.co/NousResearch/Meta-Llama-3.1-8B](https://huggingface.co/NousResearch/Meta-Llama-3.1-8B)，2024b。

+   OpenAI [2024a] OpenAI. GPT-4o-mini. [https://platform.openai.com/docs/models/gpt-4o-mini](https://platform.openai.com/docs/models/gpt-4o-mini)，2024a。通过API gpt-4o-mini-2024-07-18访问。

+   OpenAI [2024b] OpenAI. GPT-3.5 Turbo. [https://platform.openai.com/docs/models/gpt-3-5-turbo](https://platform.openai.com/docs/models/gpt-3-5-turbo)，2024b。通过API gpt-3.5-turbo-0125访问。

+   [12] Yashar Talebirad 和 Amirhossein Nadiri. 多智能体协作：利用智能 LLM 代理的力量。网址 [http://arxiv.org/abs/2306.03314](http://arxiv.org/abs/2306.03314)。

+   [13] Ziyi Tang, Ruilin Wang, Weixing Chen, Keze Wang, Yang Liu, Tianshui Chen, 和 Liang Lin. 朝着 CausalGPT 迈进：通过促进大语言模型中的因果一致性实现忠实知识推理的多智能体方法。网址 [http://arxiv.org/abs/2308.11914](http://arxiv.org/abs/2308.11914)。

+   Wan 等人 [2023] David Wan, Mengwen Liu, Kathleen McKeown, Markus Dreyer, 和 Mohit Bansal. 面向忠实度的摘要生成解码策略，2023年。

+   [15] Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, Fei Xia, Ed Chi, Quoc Le, 和 Denny Zhou. 思维链提示在大语言模型中引发推理。网址 [http://arxiv.org/abs/2201.11903](http://arxiv.org/abs/2201.11903)。

+   [16] Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Thomas L. Griffiths, Yuan Cao, 和 Karthik Narasimhan. 思维树：利用大语言模型进行深思熟虑的问题解决。网址 [http://arxiv.org/abs/2305.10601](http://arxiv.org/abs/2305.10601)。

+   Yao 等人 [2023] Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Tom Griffiths, Yuan Cao, 和 Karthik Narasimhan. 思维树：利用大语言模型进行深思熟虑的问题解决。*神经信息处理系统进展*，36, 2023年。

+   Zelikman 等人 [2023] Eric Zelikman, Qian Huang, Gabriel Poesia, Noah Goodman, 和 Nick Haber. Parsel：通过组合分解进行语言模型的算法推理。*神经信息处理系统进展*，36:31466–31523，2023年。

## 附录

### 实验提示

在我们的实验中，我们设计了若干精心制作的提示来引导语言模型进行推理任务。以下是使用的关键提示及其目的：

#### 标准输入输出（IO）提示

标准 IO 提示作为基线方法使用：

```
    Answer the following math problem. Your response should
    conclude with "the answer is n", where n is a number:
    {input}

```

这个提示直接要求模型解决数学问题，并以特定格式提供答案。

#### 思维链（CoT）提示

CoT 提示鼓励模型展示其推理过程：

```
    Answer the following question: {input}
    Make a strategy, then write. Your output should be in
    the following format:

    Strategy:
    Your strategy about how to answer the question.

    Answer:
    Your answer to the question. It should end with
    "the answer is n", where n is a number.

```

这个提示明确要求模型在提供答案之前制定一个策略，从而引导更结构化的思维过程。

#### 思维树（ToT）提示

我们对 ToT 的实现受到 Yao 等人描述的方法的启发 [[17](https://arxiv.org/html/2409.11527v2#bib.bib17)]，但进行了针对我们多代理框架的特定修改。ToT 方法以 CoT 提示为基础，进行迭代应用，允许分支并探索多个推理路径。我们的实现包括以下组件：

1.  1.

    思维生成：我们使用“样本”方法来生成思维。该方法以 CoT 提示为基础，但进行迭代应用，允许分支并探索多个推理路径。ToT 的提示流程包括：

    ```
        Answer the following question: {input}
        Make a strategy, then write. Your output should be in
        the following format:

        Strategy:
        Your strategy about how to answer the question.

        Answer:
        Your answer to the question. It should end with
        "the answer is n", where n is a number.

    ```

1.  2.

    状态评估：为了评估生成的思维，我们采用“投票”方法。这个方法通过一个提示来为不同的推理路径分配投票：

    ```
        Given an instruction and several choices, decide which
        choice is most promising. Analyze each choice in detail,
        then conclude in the last line "The best choice is {s}",
        where s the integer id of the choice.

    ```

1.  3.

    路径选择：我们使用“贪婪”方法选择最有前景的路径进行进一步扩展。这不涉及具体的提示，而是选择评估步骤中得分最高的路径。

每一步涉及多个 API 调用到语言模型，生成的思维和它们的评估引导推理空间的探索。这种方法允许对潜在解决路径进行动态和适应性的探索，增强模型处理复杂推理任务的能力。

#### 验证器提示

我们方法中的一个关键组件是思维验证器代理，它使用以下提示：

```
    As a critical mathematical reasoning verifier, evaluate
    the following thought process, which builds upon previous
    steps to reach a final conclusion. Focus on:

    1\. **Question Relevance**:
       - Ensure the entire reasoning process directly
         addresses the original question.
       - Check if the final answer actually solves what
         was asked.

    2\. **Reasoning Progression**:
       - Assess logical flow and consistency, especially
         in final steps.
       - Verify mathematical operations’ correctness and
         appropriateness.
       - Identify logical fallacies or unjustified leaps.

    3\. **Factual Accuracy**:
       - Check accuracy and relevance of facts and numbers,
         particularly in final calculations.
       - Spot any misuse of mathematical concepts.

    4\. **Completeness**:
       - Ensure all necessary aspects are addressed,
         particularly in concluding thoughts.
       - Identify significant omissions that could affect
         the result.

    5\. **Critical Assessment**:
       - Actively seek potential errors or weak points.
       - Don’t hesitate to invalidate reasoning if
         significant issues are found.

    Provide a holistic evaluation of the entire reasoning
    process, from start to finish. Conclude with
    "Reasoning is Valid" only if the entire process is
    relevant, logically sound, and error-free. Otherwise,
    conclude with "Reasoning is Invalid" and briefly
    explain why.

```

这个全面的提示指导验证者彻底评估推理过程的有效性，确保最终答案不仅正确，而且在逻辑上是合理的，并与原始问题相关。

### 示例

为了展示我们方法的有效性，我们使用了来自 GSM8K 数据集的一个具有挑战性的示例，使用 gpt-3.5-turbo 模型。通过这个示例，我们可以看到思维验证器代理如何防止 ToT 推理代理的错误推理导致最终答案的错误。

问题 1：上个月，塔莎通过卖柠檬水和修剪草坪赚了 80 美元。第一周，她修剪了卡马拉的草坪的次数是乔的三倍。接下来的一个星期，她修剪了阿尔巴的草坪的次数是乔的五倍。如果乔为塔莎的工作支付了 6 美元，那么她从卖柠檬水中赚了多少钱？答案：26。

我们有三轮，每轮有三位推理者代理（R1、R2、R3）。每轮结束后，思维验证代理会评估他们的推理。

推理者 推理总结 最终答案 是否验证 R1 错误推理。代数错误导致错误结论，认为塔莎没有给乔割草。$80 错误 R2 正确的策略，但割草收入计算错误（$60x 而不是 $48x）。$80 错误 R3 精确推理。正确计算了割草总收入，并发现柠檬水收入为 $26。$26 正确

表格 2：（第一轮）推理者输出和验证状态

### 第一轮分析

在第一轮中，推理者 1（R1）错误地计算出塔莎没有给乔割草，导致无效的最终答案 $80。推理者 2（R2）正确识别了问题的结构，但计算错误，得出了 $80。推理者 3（R3）提供了正确的推理和最终答案 $26，并已验证为有效。然而，我们尚未达成共识，因为唯一有效的答案是 R3 的回答。

第一轮结论：未达成共识。

推理者 推理总结 最终答案 是否验证 R1 重复了之前的错误，错误计算塔莎的割草收入。$80 错误 R2 修正了之前的计算错误，但再次使用了错误的割草总数，导致错误结论。$80 错误 R3 保持了与第一轮相同的正确推理和最终答案（$26）。$26 正确

表格 3：（第二轮）推理者输出和验证状态

### 第二轮分析

在第二轮中，推理者 1（R1）重复了之前的代数错误。推理者 2（R2）调整了计算，但仍然得出了错误的最终答案。推理者 3（R3）再次提供了正确的推理和最终答案 $26。

第二轮结论：未达成共识。

推理者 推理总结 最终答案 是否验证 R1 修正了之前的代数错误，但仍然未正确处理柠檬水销售问题。$80 错误 R2 调整了之前的错误，但最终计算上仍有小问题，验证代理未能发现该问题。$32 错误 R3 保持了与第一轮相同的正确推理和最终答案（$26）。$26 正确

表格 4：（第三轮）推理者输出和验证状态

### 第三轮分析

在第三轮中，推理者 1 修正了之前的代数错误，但仍然提供了无效的答案（$80）。推理者 2 最终修正了计算错误，但最终答案依然有小问题，得出了 $32。推理者 3 保持一致且准确，提供了正确答案 $26。

最终结论：最常见的有效答案是 $26。最终答案：26

问题 2：鲍勃负责为一家大酒店做洗衣工作。每个房间有两条床单、一条羽绒被、床单数量的两倍的枕套和枕套数量的两倍的毛巾。80个房间有多少件衣物需要洗？答案：26。

推理员推理总结最终答案验证 R1 精确分解了每个房间的洗衣物品，并在80个房间之间进行了乘法运算。正确的总数是1200件洗衣物。1200 正确 R2 正确的方法，但高估了枕套的数量，导致洗衣物总数为1280件，答案错误。1280 错误 R3 正确分解了每个房间的洗衣物，得出了正确的1200件洗衣物总数。1200 正确

表5：（第一轮）推理员输出和验证状态

### 第一轮分析

在第一轮中，思维验证代理评估了所有三位推理员的回答。推理员1（R1）和推理员3（R3）提供了正确且一致的推理，每人得出的总数都是1200件洗衣物，且得到了验证为准确。然而，推理员2（R2）高估了枕套的数量，导致了1280件洗衣物的错误答案，并被标记为无效。

由于两位经过验证的推理员（R1 和 R3）就正确答案达成一致，最终结果1200件洗衣物得到了可靠确认。

第一轮结论：至少有两位经过验证的推理员一致同意。最终答案：1200
