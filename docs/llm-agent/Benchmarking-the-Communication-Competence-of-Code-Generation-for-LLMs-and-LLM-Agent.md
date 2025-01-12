<!--yml

类别：未分类

日期：2025-01-11 12:35:11

-->

# 基准测试大语言模型（LLMs）和LLM代理的代码生成沟通能力

> 来源：[https://arxiv.org/html/2406.00215/](https://arxiv.org/html/2406.00215/)

Jie JW Wu\orcidlink0000-0002-7895-2023 不列颠哥伦比亚大学，基洛纳 3333 University Way 基洛纳 B.C. V1V 1V7 加拿大 [jie.jw.wu@ubc.ca](mailto:jie.jw.wu@ubc.ca) 和 Fatemeh H. Fard 不列颠哥伦比亚大学，基洛纳 3333 University Way 基洛纳 B.C. V1V 1V7 加拿大 [fatemeh.fard@ubc.ca](mailto:fatemeh.fard@ubc.ca)

###### 摘要。

大语言模型（LLMs）在代码生成领域的任务执行能力得到了显著提升。然而，LLMs作为编码者和顶级软件工程师之间仍然存在差距。最近的趋势是使用基于LLM的代理来迭代代码生成过程。基于一个观察，即顶级软件工程师通常会提出澄清性问题，以减少需求和编码解决方案中的歧义，我们认为同样应该将这一做法应用于LLMs的代码生成任务。为此，我们将LLMs的沟通能力定义为“当代码生成问题的描述存在问题时，能够提出澄清性问题”。在本研究中，我们将这些问题限制为软件需求工程领域的三个问题：需求不一致、需求模糊和需求不完整。通过在生成最终代码之前，针对问题描述的需求提出探索性问题，可以缓解与LLMs编程相关的挑战，例如意图说明不清晰，从而在初始迭代中生成正确的代码。

本研究对LLM（大规模语言模型）在代码生成中的沟通技能进行了基准测试和分析。我们通过根据上述三个问题——不一致性、模糊性、不完整性——修改问题描述，创建了一个新的基准测试集HumanEvalComm。随后，我们在HumanEvalComm上对不同的代码LLM进行了实验，并采用了一种新的LLM代理方法——代码澄清与生成代理（Okanagan），该方法通过识别和提问代码和描述中的模糊部分，进一步完善生成的代码。在评估过程中，我们引入了一种基于LLM的评估方法，并创建了沟通率和优质问题率作为评估指标，用以表示提问数量与高质量回答比例。我们发现，当问题描述根据不同的澄清类别进行手动修改时，超过60%的代码LLM回答仍然生成代码，而不是提出问题。大多数代码LLM的Pass@1和测试通过率分别下降了35% ~ 52%和17% ~ 35%，在每个类别中超过75%的数字具有统计学意义。Okanagan作为一种LLM代理方法，使用诸如ChatGPT 3.5之类的LLM，分别将沟通率和优质问题率有效提高了58%和38%。因此，当问题描述根据给定的澄清类别进行修改时，Okanagan将Pass@1和测试通过率分别提高了8%和7%。这一结果表明，利用LLM代理可以实现更有效的沟通能力。

> “提出一个好问题本身就很有价值，无论答案如何。它传达了你对他人的尊重。”
> 
> || - 改编自爱荷华和平研究所的信息

## 1\. 引言

大型语言模型（LLMs）（Vaswani 等，[2017](https://arxiv.org/html/2406.00215v2#bib.bib71)；Svyatkovskiy 等，[2020](https://arxiv.org/html/2406.00215v2#bib.bib67)；Wang 等，[2021](https://arxiv.org/html/2406.00215v2#bib.bib75)；Feng 等，[2020](https://arxiv.org/html/2406.00215v2#bib.bib21)），如 OpenAI 的 Codex（Chen 等，[2021](https://arxiv.org/html/2406.00215v2#bib.bib14)）、AlphaCode（Li 等，[2022](https://arxiv.org/html/2406.00215v2#bib.bib40)）和 CodeGen（Nijkamp 等，[2022](https://arxiv.org/html/2406.00215v2#bib.bib49)），具备从自然语言需求中生成代码片段的强大能力。然而，LLMs 存在一些已报告的问题，包括意图规范化、问题分解（Sarkar 等，[2022](https://arxiv.org/html/2406.00215v2#bib.bib59)）、代码质量、过度自信（Liu 等，[2023c](https://arxiv.org/html/2406.00215v2#bib.bib44)，[b](https://arxiv.org/html/2406.00215v2#bib.bib43)）以及可用性（Liang 等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib41)）。这些问题表明，使用 LLM 作为经验丰富的编码员（Rabinovich 等，[2017](https://arxiv.org/html/2406.00215v2#bib.bib55)；Ye 等，[2020](https://arxiv.org/html/2406.00215v2#bib.bib86)；Alon 等，[2019](https://arxiv.org/html/2406.00215v2#bib.bib4)；Bui 等，[2021](https://arxiv.org/html/2406.00215v2#bib.bib13)；Tufano 等，[2020](https://arxiv.org/html/2406.00215v2#bib.bib68)）与作为软件工程师使用 LLM 之间仍然存在显著差距。由于软件开发人员的责任不仅仅是编写代码，当前的 LLM 不能完全取代专业软件开发人员（Sarkar 等，[2022](https://arxiv.org/html/2406.00215v2#bib.bib59)；Borji，[2023](https://arxiv.org/html/2406.00215v2#bib.bib10)）。从高层次来看，这个差距存在于软件开发的几个关键方面，超出了编程的范畴，例如有效的沟通、解释、需求、设计、领域知识以及相关项目和组件的更广泛背景（Nguyen 和 Nadi，[2022](https://arxiv.org/html/2406.00215v2#bib.bib48)；Sobania 等，[2022](https://arxiv.org/html/2406.00215v2#bib.bib66)；Vaithilingam 等，[2022](https://arxiv.org/html/2406.00215v2#bib.bib70)；Siddiq 等，[2022](https://arxiv.org/html/2406.00215v2#bib.bib64)；Palacio 等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib52)）。基于 LLM 的代理与 LLM 不同，因为 LLM 是文本或代码生成的基础模型，具备预训练能力，而基于 LLM 的代理则在更具结构化、以行动为驱动的环境中部署这些模型（Guo 等，[2024a](https://arxiv.org/html/2406.00215v2#bib.bib26)）。尽管一些基于 LLM 的代理系统，如 SWE-Agent（Yang 等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib83)）、CoderAgent（Zhang 等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib90)）、AutoGPT（aut，[2023](https://arxiv.org/html/2406.00215v2#bib.bib2)）及其他多个系统（Yao 等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib85)；Wu，[2024](https://arxiv.org/html/2406.00215v2#bib.bib78)；Shinn 等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib63)；Schick 等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib61)）已经引起了广泛关注，但尚无研究系统地整合这些已报告的问题，并与 LLM 代理的代码生成方法进行结合。在本文中，我们希望通过应用沟通视角来检查这一差距，因为我们认为有效的沟通是确保获取完成编码任务所需信息的关键能力。

让我们退后一步，比较一下大型语言模型（LLMs）和软件开发人员之间的沟通方式。当前的LLMs通常通过从单次问题描述中生成代码，在一次或多次尝试中进行评估，而没有进一步的对话输入（Chen等，[2021](https://arxiv.org/html/2406.00215v2#bib.bib14); Austin等，[2021](https://arxiv.org/html/2406.00215v2#bib.bib6); Li等，[2022](https://arxiv.org/html/2406.00215v2#bib.bib40)）。这意味着当输入的问题描述存在错误或缺乏完整的上下文时，模型必须生成代码，而没有机会澄清问题，这些问题对于确保代码的正确性是必要的。在文献中，代码LLM的沟通能力（定义如下）（Fan等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib20); Zan等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib88)）以及LLM代理（Rasheed等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib56); Xi等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib79)）在代码生成领域未得到充分重视，因此很少被强调和评估。相反，在现实企业中的软件工程任务中，专业开发人员使用多种沟通方式，例如在1:1对话、团队会议和Slack频道中提出更多问题，以获取更多信息并减少对详细需求、项目上下文和设计替代方案的歧义。主动有效的沟通是顶级软件开发人员在实践中完成软件工程任务并保证高质量的重要技能（Whitehead，[2007](https://arxiv.org/html/2406.00215v2#bib.bib77); Pressman，[2005](https://arxiv.org/html/2406.00215v2#bib.bib54); Mistrík等，[2010](https://arxiv.org/html/2406.00215v2#bib.bib47); McChesney和Gallagher，[2004](https://arxiv.org/html/2406.00215v2#bib.bib45); Jazayeri，[2004](https://arxiv.org/html/2406.00215v2#bib.bib32)）。

受这种行为的启发，我们在这项工作中的动机是从有效沟通技能的维度研究和评估大型语言模型（LLMs）在代码生成中的潜力。我们认为，尽管在文献中对代码LLMs的沟通能力评估研究较少，但它对于AI系统在完成编码和软件工程任务中的长期成功至关重要（Hassan等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib27)）。因此，我们打算在这项研究中填补代码生成任务中的文献空白。对于高度专业化的代码生成任务，我们认为AI系统应主动识别缺失的信息，并找到这些缺失的部分，以便能够高质量和严谨地完成任务，而不是仅仅执行给定的任务并生成低质量的代码。正式来说，本研究中所提到的沟通能力，也可称为沟通技巧或沟通能力，定义如下：当编程问题中的需求不完整、不一致或模糊时，且模型被提示生成代码或提出澄清问题时，模型在提出澄清问题以恢复解决问题所需需求方面的能力有多强。本文中我们将“LLMs”和“代码LLMs”互换使用，指代用于代码生成任务的LLMs。

本研究首次进行了针对LLM在代码生成任务中的沟通能力的系统实证研究。首先，我们创建了一个基准数据集HumanEvalComm，用于评估在生成代码时沟通能力的程度，该基准数据集基于广泛研究的HumanEval代码生成基准（Chen et al., [2021](https://arxiv.org/html/2406.00215v2#bib.bib14)）。我们通过基于需求工程（RE）中的概念手动修改原始问题描述中的需求，构建了该基准数据集（Tukur et al., [2021](https://arxiv.org/html/2406.00215v2#bib.bib69); Dermeval et al., [2016](https://arxiv.org/html/2406.00215v2#bib.bib15)）。为了实现这一目标，我们创建了一个澄清类型的分类法：**模糊性**、**不一致性**和**不完整性**（参见第[2](https://arxiv.org/html/2406.00215v2#S2 "2\. Benchmark Construction ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")节）。基于这一分类法，我们对每个问题描述进行了修改，应用了一个或多个澄清类型。基于新的HumanEvalComm基准数据集，我们进一步评估了不同模型，在问题描述中的某些信息被手动修改为模糊、不一致或不完整时，模型的沟通能力程度。在评估过程中，我们引入了一个基于LLM的评估者，并提出了新的评估指标，以有效衡量模型的沟通技能。我们还提出了一种LLM代理方法——代码澄清与生成代理（Okanagan），作为一个基于LLM的代理，采用多轮结构和定制提示用于代码生成任务。Okanagan的一个关键特点是能够针对生成正确代码所需的输入问题描述提出澄清问题。

就研究结果而言，对于使用HumanEvalComm进行手动修改的情况，Code LLM的响应中超过60%仍然生成代码。通常，Code LLM的Pass@1和测试通过率分别下降了35% $\sim$ 52%和17% $\sim$ 35%。在三种澄清类型中，**不完整性**类别导致了更高的沟通率和良好问题率，但其Pass@1和测试通过率低于**模糊性**和**不一致性**类别。使用ChatGPT 3.5作为LLM的LLM代理方法Okaganan有效地将沟通率和良好问题率分别提高了59%和5%。这导致了测试通过率和Pass@1分别增加了25%和15%。这一结果表明，与Code LLMs相比，LLM代理具有更有效的沟通能力。

总结而言，我们做出了以下贡献：

+   •

    我们创建了一个新的基准，HumanEvalComm，用于评估大语言模型（LLMs）在代码生成任务中的沟通技能，通过基于RE概念手动修改原始问题描述中的要求：模糊性、矛盾性、不完整性的澄清类型。

+   •

    我们提出了一种LLM代理方法——代码澄清与生成代理（Okanagan），旨在增强模型的沟通能力，从而在Pass@1的代码生成能力上取得更好的表现，方法是在问题描述模糊、矛盾或不完整时通过提问澄清问题来提升沟通能力。Okanagan的贡献在于它提供了一种多轮结构，并为代码生成任务中的澄清问题定制了提示语，以在需要时提问。

+   •

    我们进行了首次实证研究，评估了在HumanEvalComm基准上的代码生成任务中，Code LLMs和Okanagan的沟通能力。在评估中，我们引入了基于LLM的评估者，并提出了两个新的评估指标——沟通率和良好问题率，以有效衡量模型的沟通技能。

我们的基准和复现包已公开发布在[https://github.com/jie-jw-wu/human-eval-comm](https://github.com/jie-jw-wu/human-eval-comm)，以支持开放数据和开放科学原则。

本文的其余部分结构如下。第[2](https://arxiv.org/html/2406.00215v2#S2 "2\. Benchmark Construction ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")节描述了我们研究中的基准构建。第[3](https://arxiv.org/html/2406.00215v2#S3 "3\. Empirical Study ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")节解释了我们实证研究的设计。第[4](https://arxiv.org/html/2406.00215v2#S4 "4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")节总结了研究问题的结果。第[5](https://arxiv.org/html/2406.00215v2#S5 "5\. Discussion ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")节对结果进行了进一步的分析和讨论。第[7](https://arxiv.org/html/2406.00215v2#S7 "7\. Threats to Validity ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")节解释了有效性的威胁，接着在第[8](https://arxiv.org/html/2406.00215v2#S8 "8\. Related Work ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")节总结了相关工作。最后，第[9](https://arxiv.org/html/2406.00215v2#S9 "9\. Conclusions ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")节总结了本文的结论。

## 2\. 基准构建

### 2.1\. 基准收集

现有基准。我们首先通过检查现有的代码生成基准来开始。根据我们所知，所有现有的基准（例如，HumanEval（Chen等，[2021](https://arxiv.org/html/2406.00215v2#bib.bib14)），CoNaLa（Yin等，[2018](https://arxiv.org/html/2406.00215v2#bib.bib87)），Galeras（Rodriguez-Cardenas等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib57)），APPS（Hendrycks等，[2021](https://arxiv.org/html/2406.00215v2#bib.bib29)），以及最近的SWE-bench（Jimenez等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib34)））都要求模型直接生成代码作为预测，而不给模型提供询问额外信息的机会。值得注意的是，这些数据集的输入由专业的人工注释组织，写作规范且清晰。然而，在实际场景中，来自人类的题目描述可能缺乏计算思维，意图说明不清晰，或在需求上存在歧义（Liang等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib41)）。

HumanEvalComm概述。为了评估Code LLMs和基于LLM的代理的沟通能力，我们选择基于广泛使用的代码生成数据集HumanEval（Chen等，[2021](https://arxiv.org/html/2406.00215v2#bib.bib14)）手工制作了一个新的基准。我们的目标是基于RE概念修改问题描述，使其能够触发澄清问题，而这些问题对于生成正确的代码是必要的。HumanEval由164个手工编写的Python编程问题组成，旨在评估Codex的编码能力。每个问题都有函数签名、文档字符串、主体和单元测试。每个问题的平均实际测试用例数为7.77。选择HumanEval作为基准数据集，因为它包含测试用例，并且被广泛用于评估LLMs（Ouyang等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib51)；Min等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib46)；Rasheed等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib56)；Zan等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib88)）。通过使用HumanEval，我们手动更改了每个问题的描述，以开发HumanEvalComm，供我们在工作中进行评估。这一过程使用了下面描述的澄清类型分类法。此外，我们在第[5.2节](https://arxiv.org/html/2406.00215v2#S5.SS2 "5.2\. 为什么不使用自动化修改? ‣ 5\. 讨论 ‣ 基准测试LLMs和LLM代理的代码生成沟通能力")中提供了为什么选择手动方法而非自动化方法来修改问题的解释。

澄清类型的分类。为了以有组织的方式修改问题描述，我们提出了以下澄清类型，这些类型基于需求工程（RE）文献（Tukur 等， [2021](https://arxiv.org/html/2406.00215v2#bib.bib69); Dermeval 等， [2016](https://arxiv.org/html/2406.00215v2#bib.bib15)）以及我们对如何将 RE 概念应用于 HumanEval 问题的理解：

+   •

    歧义：问题描述中的一些陈述可能含糊不清，且可能对应不同的概念。

+   •

    不一致性：问题描述中的一些陈述相互之间存在冲突或不一致。

+   •

    不完整性：问题描述中缺少一些概念或条件。

| 澄清类别 | 歧义 | 不一致 | 不完整 | 数量 |
| --- | --- | --- | --- | --- |
| 1a | ✓ |  |  | 164 |
| 1c |  | ✓ |  | 164 |
| 1p |  |  | ✓ | 164 |
| 2ac | ✓ | ✓ |  | 162 |
| 2cp |  | ✓ | ✓ | 34 |
| 2ap | ✓ |  | ✓ | 74 |
| 总计 | — | — | — | 762 |

表 1. 不同澄清类型组合应用于 HumanEvalComm 时的问题描述和统计信息。2ac（同样适用于2cp和2ap）的较小数量是因为我们严格直接应用了1a和1c的组合，并且仅在1a和1c的组合导致与1a或1c不同的新问题描述时，才创建2ac的新修改问题。2cp和2ap的计数较少，因为歧义（a）或不一致（c）部分在（p）中被删除，适用于大量问题。

| 澄清类型 | 问题描述 |
| --- | --- |
| 不适用（原始） | {python} def incr_list(l: list): """返回元素值递增1的列表。 >>> incr_list([1, 2, 3]) [2, 3, 4] >>> incr_list([5, 3, 5, 2, 3, 3, 9, 0, 123]) [6, 4, 6, 3, 4, 4, 10, 1, 124] """ |
| 歧义 | 文档字符串含糊不清，因为没有指定数字。 {python} def incr_list(l: list): """返回元素值递增的列表。 >>> incr_list([1, 2, 3]) [2, 3, 4] >>> incr_list([5, 3, 5, 2, 3, 3, 9, 0, 123]) [6, 4, 6, 3, 4, 4, 10, 1, 124] """ |
| 不完整 | 文档字符串是一个未完成的句子。 {python} def incr_list(l: list): """返回递增元素的列表""" |
| 不一致性 | 两个测试示例的结果与文档字符串描述及彼此之间冲突 {python} def incr_list(l: list): """返回元素值递增1的列表。 >>> incr_list([1, 2, 3]) [3, 4, 5] >>> incr_list([5, 3, 5, 2, 3, 3, 9, 0, 123]) [7, 5, 7, 4, 5, 5, 11, 2, 125] """ |
| 不一致与歧义 | 文档字符串含糊不清，因为递增的数字未指定，且测试示例的结果与文档字符串冲突。 {python} def incr_list(l: list): """返回元素值递增的列表。 >>> incr_list([1, 2, 3]) [3, 4, 5] >>> incr_list([5, 3, 5, 2, 3, 3, 9, 0, 123]) [7, 5, 7, 4, 5, 5, 11, 2, 125] """ |

表2. 基于HumanEval构建的HumanEvalComm示例。该表显示了HumanEval问题编号42的修改后的问题描述。具体来说，问题描述被修改为不一致、模糊或不完整。HumanEvalComm数据集的主要目标是评估沟通的程度。

修改问题描述的指南。对于每个问题描述，我们根据不同的澄清类型手动修改问题描述。由于不同的工程师可能以不同的方式处理同一问题，因此会影响修改问题的一致性和标准化（Sim 等人，[2003](https://arxiv.org/html/2406.00215v2#bib.bib65)），我们描述了手工操作的指南。问题描述的修改由一位具有近十年行业经验的软件工程师手动完成。另一位具有超过15年开发经验的软件工程师对修改后的描述进行了审查。对于分歧之处，两个标注者进行了标记和讨论，直到达成一致，根据RE中的模糊性、不一致性和不完整性的定义进行修改。每个问题描述都被仔细阅读，并对问题描述进行了修改。在进行问题描述的手工修改之前，模糊、不一致或不完整要求的定义和示例已经由两位专家审阅，尽管他们都具有需求工程的专业知识。

对于每个问题，我们应用了六种不同的修改方式：(1a) 使问题描述模糊；(1c) 修改描述，使其不一致；(1p) 更改问题描述，使其不完整。接下来的三种修改方式我们称为(2ac)、(2cp)和(2ap)，分别是最初修改的组合，具体为“模糊且不一致”、“不一致且不完整”和“模糊且不完整”。对于上述任何修改，我们的标准是，应用修改后的问题应该触发澄清性问题，这对于生成正确的代码是必要的。为了确保手工工作的标准化，对于每个澄清类别，修改问题的标准以及与修改问题的分歧已经在多个讨论环节中充分讨论，并对每个记录进行了修改。此外，我们还将这一点添加到了论文中的有效性威胁部分。以下是每个澄清类别的详细指南：

+   •

    澄清类别 1a：具体来说，对于 1a，为了使描述含糊不清，我们尝试改变陈述，使得人类读者可以以不同的方式解释该陈述。然而，在实际操作中，我们发现执行模糊修改非常困难：仅仅添加描述的模糊性可能不足以触发澄清问题，因为我们还有其他信息，如测试示例、常识推理和函数签名。换句话说，描述变得模糊不清，但描述中也给出了测试示例和函数签名，从而可以推断出正确的要求，从而生成正确的代码，而不需要提出澄清问题。我们解决这个问题的方法是，在描述中同时应用模糊性和不正确性，因为使用模糊性和一定程度的不正确性比仅使用模糊性更容易和安全地触发澄清问题。例如，将描述从“将数组按降序排序”改为“将数组排序”可能不会触发问题，因为函数签名或测试用例可以暗示排序是按降序进行的。然而，将描述改为“将数组排序（降序或升序）”则可以触发问题。

+   •

    澄清类别 1c：对于 1c，为了使描述不一致，我们主要通过改变示例，使得示例的输出与问题描述不匹配或相互矛盾。需要注意的是，HumanEval 基准中的大多数问题描述都包含了带有输入和输出的测试用例示例。在应用不一致修改时，对于每个问题，我们有意义地更改了测试示例的输出，而不是随意修改，以增强测试示例与文本描述之间的矛盾。请注意，1c 与 1a 不同，因为 1c 主要集中在改变测试示例，使其与问题描述相矛盾，而 1a 则通过将正确要求与错误要求结合，使得无法明确判断哪一个是正确的要求，从而产生困惑。

+   •

    澄清类别 1p：在 1p 中，我们删除了描述中的某些部分，作为不完整的修改。我们确保在应用不完整修改后，无法生成正确的代码，而无需询问问题以恢复缺失的内容。

+   •

    澄清类别 2ac、2cp、2ap：对于 2ac、2cp 和 2ap，我们直接应用了来自 1a、1c 和 1p 的两种澄清类型的组合。在这些情况下，只有在应用这两种类型的组合后，产生的描述与任何两种类型不同，我们才会创建新的修改。因此，对于每个问题，1a、1c 和 1p 总是存在，但 2ac、2cp 或 2ap 可能不存在。总体而言，修改描述的过程大约花费了 100 小时用于初步修改，30 小时用于审查和讨论分歧并达成共识。

表格 [2](https://arxiv.org/html/2406.00215v2#S2.T2 "表格 2 ‣ 2.1. 基准集合 ‣ 2. 基准构建 ‣ 基准评估 LLMs 和 LLM 代理的代码生成沟通能力") 展示了 HumanEval 中问题编号 42 的原始问题描述和三个修改版本的例子。在这个例子中，对于模糊性，“增加 1”被修改为“增加一个数字”，形成了一个模糊的描述。对于不完整性，部分文本描述和示例测试用例被移除。对于不一致性，示例的输出被修改，使其与文本描述相矛盾。对于不一致性和模糊性，应用了不一致性和模糊性的组合，使得生成正确代码的任务更加具有挑战性。值得一提的是，在构建 HumanEvalComm 之前，我们手动验证了 HumanEval 数据集中问题的描述，并确认原始问题描述没有澄清问题（模糊性、不一致性或不完整性），因此我们选择了 HumanEval 数据集中的所有 164 个问题进行评估。

### 2.2. 评估度量

我们引入了以下指标来有效评估模型在代码生成任务中的沟通能力。

沟通率。我们提出了沟通率来评估给定模型的沟通能力。沟通率旨在捕捉在 HumanEvalComm 问题中，模型以澄清问题代替代码的响应百分比。在实验中，我们使用的提示允许模型“要么生成 Python3 代码（仅用 markdown 格式直接响应代码），要么提出澄清问题”。因此，在本研究中，我们将沟通率定义为初始修改后的问题描述中没有代码片段（非代码）响应的百分比：

|  | $communicaton\_rate=\dfrac{\#initial\>model\>responses\>without\>code}{\#% initial\>model\>responses}$ |  |
| --- | --- | --- |

在实验中，我们发现，这一简单的度量方法区分了模型是否返回代码或非代码，已经是沟通能力的有效近似。

良好问题率。在这项研究中，我们使用了一个新的基于LLM的评估器，为模型返回的澄清问题打上问题质量标签。标签分为：良好（模型提出有见地的问题，帮助恢复所有缺失的信息）、一般（模型提出的问题尚可，但问题没有完全涵盖缺失的信息）、差（模型没有提问或提问无关，无法帮助恢复缺失/澄清的信息）。根据问题质量标签，我们定义“良好问题率”为拥有良好问题质量标签的回答所占的百分比：

|  | ${\color[rgb]{0,0,0}\definecolor[named]{pgfstrokecolor}{rgb}{0,0,0}% \pgfsys@color@gray@stroke{0}\pgfsys@color@gray@fill{0}good\_question\_rate=% \dfrac{\#initial\>model\>responses\>with\>Good\>labels}{\#initial\>model\>% responses}}$ |  |
| --- | --- | --- |

Pass@K。在评估中，pass@k是评估代码生成任务的一个流行且广泛使用的指标（Chen等，[2021](https://arxiv.org/html/2406.00215v2#bib.bib14); Min等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib46); Zhang等，[2023a](https://arxiv.org/html/2406.00215v2#bib.bib92)）。Pass@k被定义为“解决”问题的比例，问题被认为是“解决”了，只要任意一个$k$个代码样本通过了所有测试。因此，我们在评估中使用了Pass@1。

测试通过率。除了广泛使用的pass@k，测试通过率也是评估代码生成常用的指标（Ouyang等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib51); Hendrycks等，[2021](https://arxiv.org/html/2406.00215v2#bib.bib29)）。具体来说，测试通过率被定义为LLM生成的代码在所有测试用例中成功通过的测试用例的比例。这个指标在本研究中非常有用，因为它帮助捕捉通过澄清提问获取正确信息是否能真正提高生成代码的正确性。

## 3\. 实证研究

### 3.1\. 研究问题

在本节中，我们将描述我们在这项研究中探讨的研究问题。

RQ1：当问题描述中的需求不完整、不一致或模糊时，代码大型语言模型（Code LLMs）在沟通能力方面的表现如何？RQ1的核心目的是理解并检验当前的代码大型语言模型在代码生成中的沟通能力。其目标是初步了解代码LLMs在沟通技能方面的局限性以及可能存在的不足之处。我们在新基准HumanEvalComm中评估了不同的代码LLMs，该基准通过人工修改问题描述，使其变得不完整、不一致和模糊。我们评估并比较了不同澄清类型应用于原始问题后，代码LLMs在不同澄清类别下的结果。

RQ2：Okanagan在沟通能力上与Code LLMs相比表现如何？鉴于近期LLM代理在解决各种应用中的进展（Xi等人，[2023](https://arxiv.org/html/2406.00215v2#bib.bib79)），RQ2旨在调查我们LLM代理方法Okanagan与Code LLMs在沟通能力上的差异。因此，我们评估了Okanagan，这是一种具有多轮结构和定制提示的代码生成任务方法。我们分析并比较了Okanagan与Code LLMs的结果。

RQ3：如何评估基于LLM的评估器及其生成的评估指标？

RQ3的目的是研究和评估我们在评估中使用的基于LLM的评估器。在我们的评估中，我们使用基于LLM的评估器来1) 评分澄清问题的质量，以及2) 生成回答以回应模型的澄清问题（Kotonya等人，[2023](https://arxiv.org/html/2406.00215v2#bib.bib37)；Gao等人，[2024](https://arxiv.org/html/2406.00215v2#bib.bib23)）。因此，评估我们实验中基于LLM的评估器的可靠性非常重要。我们进行了基于LLM的评估器的人工评估，招募了不列颠哥伦比亚大学的学生，手动评估基于LLM的评估器的结果，并手动标记模型回答的质量。

RQ4：不同提示策略和超参数的影响是什么？

RQ4的目标是研究不同提示变体和对评估重要的超参数的影响。我们进行了实验，评估了1) 使用不同超参数，包括温度和问题描述中去除内容的比例（对于1p），以及2) 使用不同提示策略的结果。

### 3.2. 方法概述

![参见标题](img/309bd93230d6862a1fe3bccac0dc348a.png)

图1. 在HumanEvalComm基准测试（包含统计数据）上，方法论的可视化说明，以及对Code LLMs和LLM Agent的沟通能力评估。

概述。图[1](https://arxiv.org/html/2406.00215v2#S3.F1 "图 1 ‣ 3.2\. 方法概述 ‣ 3\. 实证研究 ‣ 对LLMs和LLM代理的代码生成沟通能力进行基准评估")展示了我们收集基准和进行实证研究的方法概述。我们首先通过使用澄清类型的分类法，修改原始HumanEval基准中的164个问题描述，创建了HumanEvalComm基准，如第[2](https://arxiv.org/html/2406.00215v2#S2 "2\. 基准构建 ‣ 对LLMs和LLM代理的代码生成沟通能力进行基准评估")节所述。其次，我们基于HumanEvalComm进行评估和实证研究，以评估不同模型（包括代码LLMs和LLM代理）的沟通能力，具体内容见第[3.3](https://arxiv.org/html/2406.00215v2#S3.SS3 "3.3\. 代码大型语言模型 ‣ 3\. 实证研究 ‣ 对LLMs和LLM代理的代码生成沟通能力进行基准评估")节和第[3.4](https://arxiv.org/html/2406.00215v2#S3.SS4 "3.4\. LLM-代理方法（Okanagan） ‣ 3\. 实证研究 ‣ 对LLMs和LLM代理的代码生成沟通能力进行基准评估")节。

图[2](https://arxiv.org/html/2406.00215v2#S3.F2 "图 2 ‣ 3.2\. 方法概述 ‣ 3\. 实证研究 ‣ 对LLMs和LLM代理的代码生成沟通能力进行基准评估")展示了模型、代码LLMs和Okanagan评估的流程图。对于HumanEvalComm中的每个编程问题，最多有六个修改过的问题描述，正如之前在表[1](https://arxiv.org/html/2406.00215v2#S2.T1 "表 1 ‣ 2.1\. 基准收集 ‣ 2\. 基准构建 ‣ 对LLMs和LLM代理的代码生成沟通能力进行基准评估")中所述。对于每个修改过的问题，都会使用一个提示词作为模型的输入，生成代码或在必要时提出澄清性问题。如果模型提出澄清性问题而不是直接生成代码，这些问题将被发送到基于LLM的评估器，评估器会根据所有可用的信息（包括修改过的问题、原始问题和澄清性问题）评估问题并生成答案。最后，答案和之前的对话将被送回模型，以便模型再次生成代码。我们还在第[5.1](https://arxiv.org/html/2406.00215v2#S5.SS1 "5.1\. 示例案例 ‣ 5\. 讨论 ‣ 对LLMs和LLM代理的代码生成沟通能力进行基准评估")节中加入了示例，帮助更好地理解评估模型与提示词之间的互动。

![参见说明](img/acc32d39ac4489db259573fd8a6a639b.png)

图2. 用于评估模型（无论是代码LLMs还是Okanagan（LLM代理））在沟通能力方面的流程图。

基于LLM的评估器：随着LLM技术的进步，近期提出了一系列工作，利用强大的LLM作为自然语言生成（NLG）任务中的无参考评估器（Kocmi 和 Federmann，[2023](https://arxiv.org/html/2406.00215v2#bib.bib36)；Wang 等，[2023a](https://arxiv.org/html/2406.00215v2#bib.bib73)；Kim 等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib35)；Kotonya 等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib37)；Gao 等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib23)）。考虑到人工评估的高昂人力成本，我们使用了基于LLM的评估器来生成回答，回复模型中澄清性问题的列表（Kotonya 等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib37)；Gao 等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib23)）。我们通过修改后的问题、原始问题和澄清性问题提示LLM评估器。基于LLM的评估器在本工作中的作用是：1）生成澄清问题的回答，2）计算“好问题率”，其用整数表示。好问题率是我们实验中的评估指标之一。至于实现方面，我们在评估中使用了GPT 3.5作为基于LLM的评估器。我们测试了零-shot和一-shot提示，但在我们的评估中发现一-shot提示并未提高性能。这与文献中的发现一致（Kotonya 等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib37)；Gao 等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib23)）。基于LLM评估器的详细提示如下所示。

针对基于LLM的评估器的提示：原始的编码问题描述经过修改后，要求变得不一致、不完整或模糊。根据修改后的描述，提出了一些澄清性问题以澄清描述内容。请根据原始和修改后的问题描述，评估澄清问题的质量。请提供一个整数，表示问题的质量（3：好的问题能恢复修改后的要求；2：一般问题，但无法帮助恢复修改后的要求；1：没有问题）。

QUALITY=[你的整数]

请同时提供澄清性问题的答案，以恢复原始问题描述中与修改后问题描述相比的修改要求。如果没有澄清性问题，请返回空答案。

ANSWERS=“‘[你的答案]”’

请严格遵循格式QUALITY=[整数]和ANSWERS=“‘[答案]”’在回复中！请使用Markdown格式围绕你的答案！

### 问题：{clarifying_questions}

### 修改后的问题描述：{problem}

### 原始描述：{original_problem}

我们手动检查了 LLM 基于的评估器的结果，以查看评估器的输出、生成的答案以及问题的评估是否正确。总体来说，生成的答案和良好问题率是合理的，但我们确实在生成的答案和良好问题率中发现了一些错误。对于良好问题率，有些情况下，当没有澄清性问题时，LLM 基于的评估器的良好问题率为 2 或 3：例如，模型的“问题”是解释和代码的结合，但没有澄清性问题。这与报告的限制（Gao 等人，[2024](https://arxiv.org/html/2406.00215v2#bib.bib23)）有一定关系，即 LLM 评估器倾向于给与与对话历史中的事实冲突的回应较高的分数（Liu 等人，[2023a](https://arxiv.org/html/2406.00215v2#bib.bib42)）。对于生成的答案，我们有时观察到提供的答案没有恢复原始要求，这可能是由于评估器本身或上文提到的“没有澄清性问题”所致。为了缓解这个问题，我们已多次优化 LLM 基于的评估器的提示，并手动检查了结果。这包括添加像“请严格遵循格式 QUALITY=[整数] 和 ANSWERS=[答案] 在回答中！”和“用 Markdown 包围你的答案！”这样的句子，这消除了许多格式错误（答案和评分无法正确提取）。尽管 LLM 基于的评估器在我们的任务中表现出有效性，但我们理解 LLM 基于的自然语言生成评估仍然具有挑战性（Gao 等人，[2024](https://arxiv.org/html/2406.00215v2#bib.bib23)；Liu 等人，[2023a](https://arxiv.org/html/2406.00215v2#bib.bib42)），未来的工作需要解决上述提到的错误。

### 3.3\. 编码大型语言模型

我们对五种广泛使用的 LLM 进行了评估。这包括三种开源指令调优的编码 LLM、一种开源指令调优的 LLM 和一种商业 LLM。对于开源模型，我们在评估中使用了在我们有限计算资源下最大可能的模型大小。

+   •

    CodeLlama（经过指令调优版本，13B）（Roziere 等人，[2023](https://arxiv.org/html/2406.00215v2#bib.bib58)）是由 Meta 发布的开源大型语言模型，用于编码，基于 Llama 2 构建，包含基础模型和指令模型。选择 CodeLlama 是因为它在 HumanEval 中的广泛使用和优异表现。我们在实验中测试了指令模型 CodeLlama-Instruct-13B，因为我们没有足够的计算资源来运行 34B 模型。其余开源模型也适用相同的情况。

+   •

    DeepSeek Coder（指令调优版本，7B）（Guo等，[2024b](https://arxiv.org/html/2406.00215v2#bib.bib25)）是一个开源代码LLM，训练数据包括87%的代码和13%的自然语言。每个模型都经过了2万亿个令牌的预训练。我们选择了这个模型，因为它在HuggingFace平台的“大型代码模型排行榜”（big，[2024](https://arxiv.org/html/2406.00215v2#bib.bib3)）中取得了前五名的成绩。该排行榜评估了多语种代码生成模型在HumanEval基准和MultiPL-E上的表现。我们在评估中使用了7亿参数的模型。

+   •

    DeepSeek Chat（指令调优版本，7B）（Bi等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib9)）是DeepSeek AI发布的开源LLM，经过2万亿个令牌的数据集训练。我们选择了这个模型，因为我们希望评估来自不同来源（如自然语言、代码及二者结合）训练的模型在沟通能力上的表现。我们将其与DeepSeek Coder进行了对比，以了解更多的自然语言预训练是否有助于提升沟通能力。我们在评估中使用了7亿参数的模型。

+   •

    CodeQwen1.5 Chat（指令调优版本，7B）（Bai等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib7)）是Qwen团队发布的一个开源代码LLM，经过3万亿个代码数据的训练。CodeQwen1.5 Chat是Qwen1.5的代码专用版本。该模型是一个基于转换器的解码器-only语言模型，并包含了群体查询注意力（GQA）以提高推理效率。我们选择了这个模型，因为它在“大型代码模型排行榜”（big，[2024](https://arxiv.org/html/2406.00215v2#bib.bib3)）中取得了前五名的成绩。

+   •

    ChatGPT，由OpenAI发布，是用于生成任务的强大模型。我们使用了模型的冻结参数版本（gpt-3.5-turbo-0125），以确保评估结果的可复现性。

请注意，上述所有评估过的模型都是经过指令调优的模型，因为在评估中，模型需要具备在给定提示下提出澄清问题的能力。除了指令调优模型，还有基础模型，但我们没有报告基础模型的结果。我们发现，未经过指令调优的基础模型不适合我们的评估，因为它们的任务仅仅是完成代码，并且不能执行诸如“生成代码或提出澄清问题”这样的指令。

![参见说明](img/6515d0dea687d3813f75699813dbe5e8.png)

图3. Okanagan，一个LLM代理方法的过程示意图。

### 3.4. LLM-代理方法（Okanagan）

在最近的LLM代理相关工作中，包括LLM代理的协作机制（Zhang et al.，[2023c](https://arxiv.org/html/2406.00215v2#bib.bib89)）以及LLM的自我纠错策略（Pan et al.，[2023](https://arxiv.org/html/2406.00215v2#bib.bib53)），我们提出并评估了一种LLM代理方法Okanagan，利用多轮结构和定制的提示格式来提问澄清性问题，用于代码生成任务。我们在Okanagan中引入了三轮：

+   •

    第1轮：代理在给定修改后的问题后直接生成代码。

+   •

    第2轮：代理根据修改后的问题和生成的代码生成澄清性问题（如果需要）。如果没有问题，直接返回第1轮生成的代码。

+   •

    第3轮：代理根据上述对话历史（包括修改后的问题、澄清问题及其答案）重新生成代码。

该结构灵感来源于现有的LLM代理方法（Zhang et al.，[2023c](https://arxiv.org/html/2406.00215v2#bib.bib89)），包含三轮和针对我们代码生成任务定制的提示。在每一轮的行动中，第1轮的行动是生成代码。第2轮的行动是提问澄清性问题。如上所述，如果没有提问，则返回第1轮生成的代码。否则，会进行反思，重新生成代码，并且包括由LLM评估器提供的澄清问题及其答案作为之前对话历史的一部分。我们采用这种结构是因为它可以轻松扩展到不同的参数值。例如，在我们的评估中，我们在第3轮停止（换句话说，我们将总轮数设置为3），但理论上我们可以设置不同的轮数。除了轮数，其他参数也可以进行调整。我们为Okanagan描述的参数集如下：

1.  (1)

    代理数量（默认值为1）。

1.  (2)

    轮次数量（默认值为3）。

1.  (3)

    每轮的行动：生成代码或提问（默认值为：第1轮 - 生成代码，第2轮 - 提问，第3轮 - 反思生成代码）

1.  (4)

    思维模式：辩论或反思（Zhang et al.，[2023c](https://arxiv.org/html/2406.00215v2#bib.bib89)）。(默认值为反思¹¹1辩论仅在使用多个代理时才可用。在Okanagan的实现中，我们使用单个代理，因此在第3轮，单个代理基于第1轮（生成的代码）和第2轮（LLM评估器提供的澄清问题及其答案）的附加信息进行对第1轮生成代码的反思。)

请注意，我们尽量使用默认参数来简化 Okanagan 的复杂度，例如使用一个代理和三轮，但在未来的工作中，可以通过设置代理数量的参数轻松地将结构从单代理扩展为多代理。如果使用多个代理，并且在给定回合中的思维模式是辩论，那么代理将交换他们之前的回应以进行协作。考虑到我们的特定任务，与（Zhang 等，[2023c](https://arxiv.org/html/2406.00215v2#bib.bib89)）相比，我们在 Okanagan 中添加了一个新参数：每轮的行动，用于指示代理在给定回合中的行动。

Okanagan 在评估中的实现。在我们的评估中，我们使用上述默认参数测试了 Okanagan。我们在 Okanagan 的每一轮中使用了 ChatGPT 3.5 作为 LLM，主要是为了与 ChatGPT 3.5 进行更容易的比较。为了比较其他 LLM 代理方法，我们搜索了其他公开可用的 LLM 代理，但未能找到适合代码生成任务、重点关注并能够提出澄清问题的开源 LLM 代理实现。附录中的表 [14](https://arxiv.org/html/2406.00215v2#A1.T14 "Table 14 ‣ Appendix A APPENDIX: Prompt Templates In HumanEvalComm ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent") 显示了在 Okanagan 中使用的提示。此外，我们还在 Okanagan 中更换了基础模型，以确保公平比较。具体而言，我们使用了另一个开源模型（DeepSeek Coder）代替 ChatGPT 3.5 作为基础 LLM 实现该方法。

### 3.5. 实验设置

计算资源。在实验中，我们在 Python 3.12 环境下实施了评估。我们部分使用了（Ouyang 等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib51)）关于 ChatGPT 非确定性的代码和（Min 等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib46)）关于开源模型测试的代码。所有 ChatGPT 和 Okanagan 的实验都在一台配备 Intel i7-6700K CPU（4.00 GHz）、32 GB RAM 的服务器上进行。其他需要 GPU 的开源模型实验则在配备 Intel Xeon Gold 6130 CPU（2.1GHz）、44 GB RAM 和 4 个 GPU（Tesla V100-SXM2-16GB）的服务器上进行。我们在实验中使用的 HuggingFace 模型名称为 deepseek-coder-6.7b-instruct、deepseek-llm-7b-chat、CodeQwen1.5-7B-Chat 和 CodeLlama-13b-Instruct-hf。

成本和运行时间。关于实验的开销，除AgentCoder外，使用ChatGPT 3.5的总费用不到15美元。该费用包括通过基于LLM的评估器使用ChatGPT 3.5对所有模型的使用，以及Okanagan和ChatGPT作为模型本身的使用。单独在HumanEvalComm中运行AgentCoder的费用约为36美元，因为它涉及到AgentCoder中的多个OpenAI API调用。ChatGPT 3.5（‘gpt-3.5-turbo-0125’）的费用为$0.0005/1k输入和$0.0015/1k输出。为了保守估计，Okanagan或ChatGPT作为实验中的模型的总运行时间不到半小时。其他开源模型的运行时间取决于计算资源，如内存和GPU。请注意，我们已尽力减少对话间冗余的提示，以简化流程并节省成本：每个问题中我们为给定模型使用了两个提示，这些提示是模型收集澄清问题答案和生成正确代码所必需的。对于基于LLM的评估器，我们仅使用了一个提示，该提示同时生成问题的答案和Good Question Rate中使用的问题质量标签。

统计检验配置。在本研究中，对于每个澄清类别，我们进行了独立的双样本t检验，以评估HumanEval中原始通过率（具体来说是Pass@K和Test Pass Rate）与HumanEvalComm中的通过率之间是否存在统计学上显著的差异。我们假设原假设$H_{0}$是测试通过率或Pass@1在各组之间没有显著差异。备择假设$H_{1}$则认为各组之间的测试通过率或Pass@1存在显著差异。该检验比较了两个独立组的均值，检验原假设即均值相等。计算出的t统计量量化了组均值之间差异与组内得分变异性的比值。我们报告了相关的p值，表示在原假设下观察到数据的概率。当p值低于显著性水平$\alpha=0.01, 0.05, 0.1$时，拒绝原假设，表明两组之间存在显著差异。

## 4\. 结果与分析

### 4.1\. HumanEvalComm中代码LLM的沟通能力（RQ1）

为了回答RQ1，我们进行了实验，评估HumanEvalComm中问题的沟通能力。由于我们在RQ1中关注的是代码LLM的结果，Okanagan的结果将在RQ2中单独讨论。对于每个根据类别澄清类型或澄清类型组合进行修改的问题，我们遵循了图[2](https://arxiv.org/html/2406.00215v2#S3.F2 "图 2 ‣ 3.2．方法概述 ‣ 3．经验研究 ‣ 基准测试代码生成LLM和LLM代理的沟通能力")中的流程。我们计算了以下评估指标：沟通率、好问题率、Pass@1和测试通过率。我们比较了HumanEvalComm中修改后的问题和HumanEval中原始问题的Pass@1和测试通过率。表[3](https://arxiv.org/html/2406.00215v2#S4.T3 "表 3 ‣ 4.1．代码LLM在HumanEvalComm上的沟通能力（RQ1） ‣ 4．结果与分析 ‣ 基准测试代码生成LLM和LLM代理的沟通能力")总结了我们为评估的模型生成的整体结果。图[4](https://arxiv.org/html/2406.00215v2#S4.F4 "图 4 ‣ 4.1．代码LLM在HumanEvalComm上的沟通能力（RQ1） ‣ 4．结果与分析 ‣ 基准测试代码生成LLM和LLM代理的沟通能力")将表[3](https://arxiv.org/html/2406.00215v2#S4.T3 "表 3 ‣ 4.1．代码LLM在HumanEvalComm上的沟通能力（RQ1） ‣ 4．结果与分析 ‣ 基准测试代码生成LLM和LLM代理的沟通能力")中的数据重新排列，以可视化的方式便于不同模型之间的直接比较。我们首先分析了整体结果，然后查看了每个澄清类别的结果。对于每个类别，我们使用学生t检验进行了统计检验，并获得了p值。

首先来看一下沟通率。从表格[3](https://arxiv.org/html/2406.00215v2#S4.T3 "Table 3 ‣ 4.1\. Communication Competency of Code LLMs on HumanEvalComm (RQ1) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")和图表[4](https://arxiv.org/html/2406.00215v2#S4.F4 "Figure 4 ‣ 4.1\. Communication Competency of Code LLMs on HumanEvalComm (RQ1) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")来看，ChatGPT、CodeLlama和CodeQwen1.5 Chat的沟通率都低于20%，远低于满分100%。这意味着，对于需要通过澄清问题来生成正确代码的问题描述，这些模型提出问题的机会不足20%。最近发布的DeepSeek Coder和DeepSeek Chat的沟通率分别达到了30.76%和37.93%。一个解释这一现象的假设是，DeepSeek Chat的通用能力对高沟通率非常重要。

除了沟通率，良好问题率也是一个有用的指标，因为它报告了基于问题内容使用LLM评估器标记为良好问题的问题百分比。在良好问题率方面，同样如表格[3](https://arxiv.org/html/2406.00215v2#S4.T3 "Table 3 ‣ 4.1\. Communication Competency of Code LLMs on HumanEvalComm (RQ1) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")和图表[4](https://arxiv.org/html/2406.00215v2#S4.F4 "Figure 4 ‣ 4.1\. Communication Competency of Code LLMs on HumanEvalComm (RQ1) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")所示，ChatGPT、CodeLlama和CodeQwen1.5 Chat的平均问题质量低于DeepSeek Coder和DeepSeek Chat。特别是，ChatGPT的比率远低于其他开源模型。通过我们手动检查，原因之一是开源模型有时没有按照指示返回代码块或问题。它们有时会输出代码块并附带一些解释。这种类型的回应不是澄清性问题，但LLM评估器有时会将其标记为“良好问题”，我们在第3.2节中对此进行了详细描述。不管怎样，基于这些数据，良好问题率仍然有显著的提升空间。

对于通过率的测量，基于表[3](https://arxiv.org/html/2406.00215v2#S4.T3 "表 3 ‣ 4.1\. Code LLMs在HumanEvalComm上的通信能力（RQ1） ‣ 4\. 结果与分析 ‣ 基准测试代码生成模型的通信能力")和图[4](https://arxiv.org/html/2406.00215v2#S4.F4 "图 4 ‣ 4.1\. Code LLMs在HumanEvalComm上的通信能力（RQ1） ‣ 4\. 结果与分析 ‣ 基准测试代码生成模型的通信能力")，ChatGPT、CodeLlama和DeepSeek Chat的Pass@1和测试通过率整体低于CodeQwen1.5 Chat和DeepSeek Coder。这个趋势在Pass@1和测试通过率上都是相似的。一个假设是，这一结果部分原因是CodeQwen1.5 Chat和DeepSeek Coder在原始HumanEval基准测试中的Pass@1和测试通过率较高。在相对变化方面，我们看到DeepSeek Chat从原始HumanEval到HumanEvalComm的Pass@1和测试通过率都有所提高。根据我们的调查，这主要是由于非法的响应格式：DeepSeek Chat在原始HumanEval中的许多响应没有代码标记，因此这些没有代码标记的响应未能通过所有测试。对于其余的开源模型，Pass@1的相对下降在35%到52%之间，测试通过率的相对下降在17%到35%之间。

| 模型 | Pass@1 | 测试通过率 | 通信率 | 好问题率 |
| --- | --- | --- | --- | --- |
|  | HmEval | HmEvalComm | HmEval | HmEvalComm | 通过率 | 问题通过率 |
| ChatGPT | 65.58% | 31.34% | 76.42% | 49.39% | 14.21% | 13.43% |
| CodeLlama | 29.88% | 19.35% | 45.71% | 37.79% | 10.16% | 37.55% |
| CodeQwen1.5 Chat | 76.83% | 47.61% | 84.4% | 62.89% | 4.82% | 41.68% |
| DeepSeek Coder | 71.78% | 45.68% | 79.44% | 62.25% | 30.76% | 61.42% |
| DeepSeek Chat | 12.8% | 26.32% | 13.86% | 44.52% | 37.93% | 58.71% |
| Okanagan (基础=ChatGPT) | 27.45% | 39.62% | 33.45% | 56.98% | 72.73% | 52.24% |
| Okanagan (基础=DeepSeek Coder) | 21.25% | 38.06% | 24.3% | 52.72% | 82.51% | 60.13% |

表 3. 各种澄清类别在Pass@1、测试通过率、通信率和好问题率上的评估结果，以及不同模型在HumanEvalComm（表中为HmEvalComm）的表现。此外，HumanEval原始问题上的Pass@1和测试通过率（表中为HmEval）也显示了出来。前四名结果以**粗体**标出。

![参考说明](img/29eb3ea170bd6b64a7384299bf24bee1.png)

图 4. 模型在通信率、好问题率（左图）以及Pass@1、测试通过率（右图）中的有效性比较。请注意，在右图中，星星代表对应模型在HumanEval基准测试中的原始表现，颜色相同。这直观地展示了当问题描述被修改时，性能的变化情况。

| 澄清 | 模型 | Pass@1 | 测试通过率 | 通信率 | 好问题率 |
| --- | --- | --- | --- | --- | --- |
| Category |  | HmEval | HmEvalComm (p-value) | HmEval | HmEvalComm (p-value) | Rate | Question Rate |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1a | ChatGPT | 65.58% | 33.77%*** | 76.42% | 54.98%*** | 5.84% | 4.55% |
|  |  | (0.000) |  | (0.000) |  |  |
| CodeLlama | 29.88% | 16.46%*** | 45.71% | 36.24%** | 13.64% | 42.68% |
|  |  | (0.004) |  | (0.037) |  |  |
| CodeQwen1.5 Chat | 76.83% | 46.34%*** | 84.4% | 62.62%*** | 5.84% | 43.29% |
|  |  | (0.000) |  | (0.000) |  |  |
| DeepSeek Coder | 71.78% | 43.29%*** | 79.44% | 61.2%*** | 25.97% | 62.8% |
|  |  |  | (0.000) |  | (0.000) |  |  |
|  | DeepSeek Chat | 12.8% | 21.95%** | 13.86% | 40.62%*** | 39.61% | 56.71% |
|  |  |  | (0.029) |  | (0.000) |  |  |
|  | Okanagan | 27.45% | 44.81%*** | 33.45% | 64.22%*** | 65.58% | 52.60% |
|  |  |  | (0.001) |  | (0.000) |  |  |
| 1c | ChatGPT | 65.58% | 53.25%** | 76.42% | 66.37%** | 5.84% | 6.49% |
|  |  | (0.027) |  | (0.028) |  |  |
| CodeLlama | 29.88% | 32.93% | 45.71% | 52.14% | 7.79% | 32.32% |
|  |  | (0.553) |  | (0.172) |  |  |
| CodeQwen1.5 Chat | 76.83% | 67.68%*** | 84.4% | 79.9%*** | 7.79% | 46.95% |
|  |  | (0.000) |  | (0.000) |  |  |
| DeepSeek Coder | 71.78% | 61.59%* | 79.44% | 76.75% | 15.03% | 53.66% |
|  |  |  | (0.051) |  | (0.501) |  |  |
|  | DeepSeek Chat | 12.8% | 39.63%*** | 13.86% | 56.89%*** | 28.1% | 61.59% |
|  |  |  | (0.000) |  | (0.000) |  |  |
|  | Okanagan | 27.45% | 57.14%*** | 33.45% | 70.01%*** | 55.19% | 42.86% |
|  |  |  | (0.000) |  | (0.000) |  |  |
| 1p | ChatGPT | 65.58% | 27.95%*** | 76.42% | 44.14%*** | 31.68% | 26.71% |
|  |  | (0.001) |  | (0.000) |  |  |
| CodeLlama | 29.88% | 15.24%*** | 45.71% | 29.41%*** | 7.74% | 33.54% |
|  |  | (0.000) |  | (0.000) |  |  |
| CodeQwen1.5 Chat | 76.83% | 46.95%*** | 84.4% | 59.36%*** | 1.3% | 38.41% |
|  |  | (0.000) |  | (0.000) |  |  |
| DeepSeek Coder | 71.78% | 45.12%*** | 79.44% | 58.57%*** | 48.7% | 68.9% |
|  |  |  | (0.000) |  | (0.000) |  |  |
|  | DeepSeek Chat | 12.8% | 21.95%** | 13.86% | 43.73%*** | 37.66% | 55.49% |
|  |  |  | (0.029) |  | (0.000) |  |  |
|  | Okanagan | 27.45% | 36.65%* | 33.45% | 54.16%*** | 93.17% | 58.39% |
|  |  |  | (0.082) |  | (0.000) |  |  |

表4. 各模型在Pass@1、HumanEval中的原始问题和HumanEvalComm中修改问题的测试通过率、交流率和优质问题率上的澄清类别1a、1c、1p的评估结果。*p<0.1; **p<=0.05; ***p<0.01

关于仅包含一个澄清类型的类别细分。除了总体结果，我们还希望进一步了解不同澄清类别下的对应结果。[表4](https://arxiv.org/html/2406.00215v2#S4.T4 "Table 4 ‣ 4.1\. Communication Competency of Code LLMs on HumanEvalComm (RQ1) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent") 展示了在澄清类别 1a、1c 和 1p 下的结果细分，其中仅应用了一个澄清类型（模糊性、不一致性和不完整性）来处理问题。对于 ChatGPT，在这三种澄清类型中，不完整性具有最高的整体沟通率，压过了模糊性和不一致性的沟通率。这意味着，对于像 ChatGPT、DeepSeek Coder 和 DeepSeek Chat 这样的模型，不完整性相对更容易被检测和提出，而模糊性和不一致性则较难。三种类型中，不一致性具有最低的沟通率。解释这一现象的一个假设是，不一致性需要更强的推理能力来检测。良好问题率呈现出与沟通率类似的模式，表明问题质量与代码 LLM 的沟通率成正比。

上述陈述的两个例外是，CodeLlama 和 CodeQwen1.5 Chat 在不完整性类别中的沟通率低于在模糊性和不一致性类别中的沟通率。良好问题率也表现出相似的趋势。这表明，一些代码 LLM（如 CodeLlama 和 CodeQwen1.5 Chat）经过训练和设计，倾向于完成代码而非提出问题，即使在需求不完整的情况下也是如此。这反映了 LLM 的生成特性：给定一个提示，LLM 作为生成模型，实质上是基于统计模型（Shanahan, [2024](https://arxiv.org/html/2406.00215v2#bib.bib62)）生成和完成文本（或在我们的场景中是代码）。因此，关于低结果的一个假设是，由于其生成特性，LLM 在评估编码任务中的沟通能力时存在劣势。这个结果还表明，像基于 LLM 的智能 AI 代理这样的更智能的 AI 代理，作为生成模型的一部分，可能在沟通能力评估上优于 LLM（Shanahan, [2024](https://arxiv.org/html/2406.00215v2#bib.bib62)）。

对于生成代码的测试性能，值得注意的是，Incompleteness（不完整性）总体上获得了所有模型中最低的Pass@1（12.8% $\sim$ 46.95%）和测试通过率（29.41% $\sim$ 59.36%）。一个假设是，如果在遇到Incompleteness问题时没有提出澄清性问题，生成的代码通常会因为缺乏信息而不正确。Inconsistency（不一致性）具有最高的Pass@1和测试通过率，因为对于Inconsistency问题，LLMs有时能够在不提问澄清性问题的情况下生成正确的代码。对于1a、1c和1p类别，除3个Pass@1和测试通过率的变化外，其他变化在统计学上具有显著性，p值小于0.1。

按照两种澄清类型对类别进行细分。表格[5](https://arxiv.org/html/2406.00215v2#S4.T5 "Table 5 ‣ 4.1\. Communication Competency of Code LLMs on HumanEvalComm (RQ1) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")展示了2ac、2ap和2cp澄清类别的结果细分，其中应用了两种澄清类型的组合。与表格[4](https://arxiv.org/html/2406.00215v2#S4.T4 "Table 4 ‣ 4.1\. Communication Competency of Code LLMs on HumanEvalComm (RQ1) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")中仅应用一种澄清类型相比，两种澄清类型的沟通率平均略高于单一澄清类型。这是合理的，因为两种澄清类型的组合自然会引发比单一类型更多的问题。因此，我们在“良好问题”指标上也看到类似的趋势。

在涉及两种澄清类型的测试性能方面，从单一澄清类型到两种澄清类型，Pass@1和测试通过率显著下降。因此，与单一澄清类型相比，组合两种澄清类型显著降低了测试通过率，但仅略微增加了沟通率和澄清性问题的质量。考虑到澄清难度的增加，沟通率的轻微上升是可以理解的。通过率的下降表明，模型在面对澄清类型组合的挑战性情境时，很难获得解决任务所需的必要要求。2cp（不一致性和不完整性组合）相比于2ap和2ac，导致了较低的Pass@1和测试通过率。对于2ac、2cp和2ap类别，75%的Pass@1和测试通过率的变化在统计学上具有显著性，因为这些变化的p值小于0.05。

<svg class="ltx_picture" height="221.65" id="S4.SS1.p10.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,221.65) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 25.59 21.65)"><foreignobject color="#000000" height="178.34" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="548.82">Answer to RQ1: More than 60% of responses from Code LLMs still generate code rather than ask questions when the problem descriptions are manually modified according to different clarification categories. Typically, the Pass@1 and Test Pass Rate of Code LLMs drop by 35% $\sim$ 52% and by 17% $\sim$ 35% respectively, with statistical significance in each category for over 75% numbers. Among the three clarification types, the Incompleteness category results in higher communication rates and Good Question Rates, but lower Pass@1 and Test Pass Rate than the Ambiguity and Inconsistency categories for Code LLMs. A combination of two clarification types leads to slightly higher communication rates but much lower Test Pass Rates than one clarification type.</foreignobject></g></g></svg>

| 澄清 | 模型 | Pass@1 | 测试通过率 | 沟通 | 良好 |
| --- | --- | --- | --- | --- | --- |
| 类别 |  | HmEval | HmEvalComm (p值) | HmEval | HmEvalComm (p值) | 通过率 | 问题通过率 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2ac | ChatGPT | 65.79% | 20.39%*** | 76.77% | 42.66%*** | 5.26% | 7.90% |
|  |  | (0.000) |  | (0.000) |  |  |
| CodeLlama | 29.63% | 14.2%*** | 45.65% | 36.95%* | 12.5% | 42.59% |
|  |  | (0.001) |  | (0.054) |  |  |
| CodeQwen1.5 Chat | 77.16% | 40.12%*** | 84.28% | 59.56%*** | 7.24% | 47.53% |
|  |  | (0.000) |  | (0.000) |  |  |
| DeepSeek Coder | 71.43% | 40.74%*** | 79.18% | 61.72%*** | 26.97% | 58.64% |
|  |  |  | (0.000) |  | (0.000) |  |  |
|  | DeepSeek Chat | 12.96% | 20.99%* | 14.03% | 39.09%*** | 44.08% | 64.2% |
|  |  |  | (0.055) |  | (0.000) |  |  |
|  | Okanagan | 27.15% | 25.66% | 33.23% | 47.37%*** | 64.47% | 45.39% |
|  |  |  | (0.769) |  | (0.004) |  |  |
| 2cp | ChatGPT | 77.42% | 15.63%*** | 84.91% | 34.79%*** | 6.25% | 9.38% |
|  |  | (0.000) |  | (0.000) |  |  |
| CodeLlama | 38.24% | 14.71%** | 60.9% | 33.04%*** | 9.68% | 29.41% |
|  |  | (0.028) |  | (0.003) |  |  |
| CodeQwen1.5 Chat | 73.53% | 38.24%*** | 83.57% | 55.82%*** | 0% | 29.41% |
|  |  | (0.003) |  | (0.004) |  |  |
| DeepSeek Coder | 70.59% | 29.41%*** | 80.93% | 50.97%*** | 12.9% | 52.94% |
|  |  |  | (0.000) |  | (0.002) |  |  |
|  | DeepSeek Chat | 11.76% | 26.47% | 11.76% | 48.38%*** | 22.58% | 52.94% |
|  |  |  | (0.127) |  | (0.000) |  |  |
|  | Okanagan | 32.26% | 28.13% | 34.98% | 44.97% | 84.38% | 59.38% |
|  |  |  | (0.726) |  | (0.375) |  |  |
| 2ap | ChatGPT | 59.42% | 16.67%*** | 71.09% | 32.39%*** | 37.50% | 29.17% |
|  |  | (0.000) |  | (0.000) |  |  |
| CodeLlama | 28.38% | 17.57% | 41.59% | 31.27% | 8.57% | 39.19% |
|  |  | (0.120) |  | (0.135) |  |  |
| CodeQwen1.5 Chat | 74.32% | 28.38%*** | 82.71% | 44.35%*** | 1.45% | 28.38% |
|  |  | (0.000) |  | (0.000) |  |  |
| DeepSeek Coder | 71.23% | 36.49%*** | 81.36% | 48.17%*** | 56.52% | 70.27% |
|  |  |  | (0.000) |  | (0.000) |  |  |
|  | DeepSeek Chat | 9.46% | 24.32%** | 10.9% | 35.99%*** | 52.17% | 56.76% |
|  |  |  | (0.016) |  | (0.000) |  |  |
|  | Okanagan | 27.94% | 29.17% | 34.52% | 43.94% | 94.44% | 66.67% |
|  |  |  | (0.874) |  | (0.196) |  |  |

表 5. 关于Pass@1、HumanEval中原始问题和HumanEvalComm中修改问题的测试通过率、沟通率和良好问题率的评估结果，针对2ac、2cp、2ap类别的澄清。*p<0.1; **p<=0.05; ***p<0.01

### 4.2\. 比较Okanagan与代码LLMs在沟通能力上的差异（RQ2）

概述。RQ2 旨在比较 LLM 代理方法 Okanagan 与当前的代码 LLM 在沟通能力上的表现。从表格 [3](https://arxiv.org/html/2406.00215v2#S4.T3 "表格 3 ‣ 4.1\. Code LLM 在 HumanEvalComm 上的沟通能力（RQ1） ‣ 4\. 结果与分析 ‣ 评估 LLM 和 LLM 代理在代码生成中的沟通能力") 可以看出，ChatGPT 的沟通率低于 20%，而 Okanagan 与 ChatGPT 的沟通率超过 70%，远高于 ChatGPT 和其他所有模型。这表明，从 LLM 转变为 LLM 代理显著提高了沟通率。在测试性能方面，Okanagan 在 Pass@1 和 Test Pass Rate 两个指标上都优于除 CodeQwen1.5 Chat 和 DeepSeek Coder 之外的所有模型。之前提到的趋势对于 Pass@1 和 Test Pass Rate 都是相似的。这表明 Okanagan 在通过提问澄清问题获取必要信息方面的有效性。

然而，一个缺点是，在原始的 HumanEval 中，Okanagan 的 Pass@1 和 Test Pass Rate 要明显低于 ChatGPT。这是因为多轮结构有时即使对于原始问题也会先提出问题作为初始回应，而在原始的 HumanEval 中，期望在初始回应中直接返回代码，并对初始回应中的代码进行评估。这意味着对于不需要提问的原始问题，Okanagan 有时仍然会提出看似不必要的问题，因为原始问题已知是完整的且没有需求问题。这是一个有效的限制，未来的工作需要解决这个问题（例如，多代理辩论（Du 等人，[2023](https://arxiv.org/html/2406.00215v2#bib.bib16)））。如果能解决这一点，则表明 LLM 代理具有更强的沟通能力，因为 LLM 代理能够智能地判断何时停止提问（Hassan 等人，[2024](https://arxiv.org/html/2406.00215v2#bib.bib27)）。另一方面，在 HumanEvalComm 上，Okanagan 显示出，在基于 LLM（ChatGPT）的 LLM 代理之上，我们可以在所有指标上获得比 ChatGPT 更好的结果。这表明 LLM 代理在沟通能力上的优势，因为 LLM 代理可以通过提问来获取所需的信息，从而提高通过率。

按照澄清类型分类的结果细分。在 1a、1c 和 1p 的澄清类别结果细分中，Okanagan 在 1a、1c 和 1p 的沟通率上都显著高于任何其他代码 LLM。对于不完整性，三类中的沟通率最高，超过 90%。在 Good Question Rate（良好提问率）方面也有类似的趋势，但 DeepSeek Coder 和 DeepSeek Chat 在 1a 和 1c 中的良好提问率高于 Okanagan。因此，Okanagan 在所有模型中无论是在 Pass@1 还是 Test Pass Rate 中都取得了前三的表现，并且具有统计学意义。

包含两种澄清类型的分类细分。对于2ac、2ap和2cp澄清类别的结果细分，Okanagan在所有模型中具有最高的沟通率，在2cp中达到85%，在2ap中达到94%。在Good Question Rate中也保持类似的趋势。对于Pass@1和Test Pass Rate，Okanagan在大多数情况下仍排名前三，但统计检验显示，大多数p值不足以提供强有力的统计显著性证据，这是由于具有两种类型的类别样本量较小。

在Okanagan中切换基础模型。我们还将基础模型从使用ChatGPT切换到使用另一个开源模型DeepSeek Coder，以确保公平比较。我们选择DeepSeek Coder的原因是，它是所有在表3中评估的指标上取得前四名结果的最佳模型。Okanagan（以DeepSeek Coder为基础）的结果显示在表3中。

对于HumanEvalComm数据集，与DeepSeek Coder相比，Okanagan在以DeepSeek Coder为基础时在Communication Rate上更好，在Good Question Rate上相似，在Pass@1和Test Pass Rate上略逊一筹。经过一些调查，较低的通过率可能是由于Okanagan框架中使用的提示对ChatGPT作为基础模型有效，但对DeepSeek Coder作为基础模型则不适用。由于提示问题，基础模型DeepSeek Coder有时没有按照指令执行，并在Okanagan中提问。对于HumanEval数据集，Okanagan以DeepSeek Coder为基础时的表现不如DeepSeek Coder。这与Okanagan（以ChatGPT为基础）与ChatGPT的对比相似，原因是Okanagan的局限性，有时会提出不必要的问题，这一点在论文中有讨论。

将Okanagan与以ChatGPT为基础和以DeepSeek Coder为基础进行比较，Okanagan在以DeepSeek Coder为基础时在HumanEval和HumanEvalComm的Test Pass率和Pass@1表现较差，但在Communication Rate和Good Question Rate方面更好。前者可能是因为DeepSeek Coder作为基础模型在Okanagan中对提示理解不如ChatGPT作为基础模型有效。后者可能是由于DeepSeek Coder在Communication Rate和Good Question Rate方面优于ChatGPT。

<svg class="ltx_picture" height="171.84" id="S4.SS2.p8.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,171.84) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 25.59 21.65)"><foreignobject color="#000000" height="128.53" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="548.82">Answer to RQ2: Okanagan, as a LLM agent approach that uses LLM (specifically ChatGPT), effectively increases Communication Rate and Good Question Rate by an absolute 58% and 38% respectively, and thus boosts Pass@1 and Test Pass Rate by an absolute 8% and 7% respectively, when the problem descriptions are modified based on given clarification categories. However, Okanagan tends to still ask questions that appear to be unnecessary for original problems that do not need questions. This indicates headroom for achieving more effective communication capability using LLM agent, and the need to strike a balance between asking unnecessary questions and truly needed questions.</foreignobject></g></g></svg>

### 4.3. 基于LLM的评估器的人工评估（RQ3）

概览。我们对基于LLM的评估器进行了人工评估，以回答RQ3。我们招募了六名不列颠哥伦比亚大学的研究生进行人工评估。这六名学生来自计算机科学系的软件工程实验室，所有人都有至少两年的专业软件开发背景。我们要求学生在一周内手动评估LLM基评估器的结果，并手动标记模型响应的质量。

手动评估设置：我们从HumanEvalComm中选择了一部分具有代表性的样本进行手动评估，样本总数约为4300个（针对6个模型）。然而，由于手动评估对招募的学生来说是非常耗时的，我们使用样本量计算器为每个模型选择了60个样本，这些样本具有统计代表性。在样本量计算器中，我们设置了95%的置信度、5%的误差边际和50%的总体比例。我们使用哈希函数从HumanEvalComm中为每个模型随机选择了60个问题样本，并创建了一个指南，提供了详细的评估说明，供学生进行手动评估。为了避免引入偏差，我们隐藏了6个模型的实际名称，而使用了模型1至模型6来代替实际的模型名称。对于每个模型的样本，我们要求两名学生独立标注。目标是进行人工评估，以评估模型澄清性问题的质量（即，是否提出了问题，问题质量如何）以及我们的大型语言模型（LLM）评估器回答问题的质量（即，答案质量如何）。具体来说，我们要求学生对每个样本进行评分的任务如下：

+   •

    是否为问题：学生需要评估模型是否根据问题描述提出了问题。标签为：1：是，0：否。

+   •

    问题质量：接着我们要求学生对问题的质量进行标注。与良好问题率相似，标签为：3：良好（模型提出了有助于恢复缺失信息的澄清性问题），2：一般（模型提出了合理的澄清性问题，但问题可能没有涵盖缺失的信息），1：差（模型没有提出任何问题，或提出了完全无关的、无法帮助恢复缺失/澄清信息的问题）。

+   •

    回答质量：我们要求学生对问题的回答质量进行标注。标签为：3：良好（回答恢复了缺失的信息），2：一般（回答合理且可以接受，但可能没有恢复缺失的信息），1：差（回答完全没有意义），-1：错误恢复（回答恢复了缺失的信息，尽管“回答”并非问题），0：没有恢复（如果“回答”没有问题，回答没有恢复缺失的信息。注意：如果模型的“回答”包含问题，选择1、2或3，否则选择-1或0。）

| 模型 | 良好回答率（问题） | 可接受回答率（问题） | 错误恢复率（非问题） | Kappa $\kappa$ |
| --- | --- | --- | --- | --- |
| ChatGPT | 50.0% | 50.0% | 0.94% | 0.85 |
| CodeLlama | 100% | 100% | 32.5% | 0.96 |
| CodeQwen1.5 Chat | 100% | 100% | 40.7% | 0.72 |
| DeepSeek Coder | 69.2% | 100% | 38.0% | 0.92 |
| DeepSeek Chat | 41.91% | 59.0% | 34.8% | 0.77 |
| Okanagan | 31.5% | 53.3% | 0% | 0.77 |

表格 6. 手动评估中来自基于LLM的评估器对模型回应的答案的指标。每一行显示不同的指标，针对LLevaluator的答案与模型回应之间的Kappa（$\kappa$）值。

我们将上述标签转换为4个指标并报告如下：

+   •

    沟通率：模型回应中提问澄清问题的百分比。

+   •

    好问题率：模型回应中提问并带有“好问题”质量标签（3）的提问所占的百分比。

+   •

    好答案率（问题）：模型对问题的好答案比例，其中答案具有“好”（3）质量标签。此项针对模型回应中的问题。

+   •

    可接受答案率（问题）：模型提问中带有“好”（3）和“公平”（2）问题质量标签的百分比。此项针对模型回应中的问题。

+   •

    错误恢复率（非问题）：对于没有提问的模型回应，错误恢复率是指在所有没有问题的模型回应答案中，恢复缺失信息的答案所占的百分比。理想情况下，如果模型回应不是问题，我们不希望提供能够恢复缺失信息的答案，因为这会错误地提高测试通过率和Pass@1的评分。

手动评估结果。表格 [6](https://arxiv.org/html/2406.00215v2#S4.T6 "表格 6 ‣ 4.3\. 基于LLM的评估器的手动评估（RQ3） ‣ 4\. 结果与分析 ‣ 基准测试代码生成的沟通能力：针对LLM和LLM代理") 显示了基于LLM的评估器对不同模型的答案在三个指标下的表现比较：好答案率、可接受答案率和错误恢复率，以及通过Cohen的Kappa $\kappa$值表示的评估员之间的一致性。每个指标的结果来自招募的学生。请注意，每个结果有两位学生评分员，因此在三个比率列和Kappa列中分别报告了平均值和Kappa值。所有结果的Kappa值在0.7到1.0之间，表示两位评分员达到了实质性或接近完美的一致性。总体而言，所有模型的可接受答案率至少超过50%。除DeepSeek Chat和Okanagan外，所有模型的好答案率也超过50%。这表明基于LLM的评估器能够为不同模型提出的问题提供正确或可接受的答案。在错误恢复率方面，ChatGPT和Okanagan显示出较低的错误恢复率，分别为0.94%和0%，而其他模型的错误恢复率则较高，介于30%和40%之间。这表明这些模型存在潜在风险，增加的测试通过率可能并非来自模型提出问题的能力。未来的工作应该集中在尽可能进一步降低错误恢复率。

![参见标题](img/00016cbd6fe0fdd3322d382235ef2ea2.png)

图5. 手动评估和自动化评估在不同模型间的交流率与良好问题率比较。每一行显示某个模型在特定指标上的百分比，后面跟着括号中的Kappa（$\kappa$）值。

比较自动化评估和人工评估之间的指标。为了衡量我们评估中使用的指标的可靠性，我们进一步比较了自动化评估中使用的指标与人工评估中相同指标的差异。我们在与人工评估相同的数据上计算了自动化指标，包括交流率和良好问题率。图[5](https://arxiv.org/html/2406.00215v2#S4.F5 "Figure 5 ‣ 4.3\. Manual Evaluation of LLM-based Evaluator (RQ3) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")显示了人工评估与自动化评估在交流率和良好问题率上的直接对比。对于人工指标，如前所述，图[5](https://arxiv.org/html/2406.00215v2#S4.F5 "Figure 5 ‣ 4.3\. Manual Evaluation of LLM-based Evaluator (RQ3) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")报告了两名学生评分员的平均值和Kappa值。

对于交流率，自动化指标通常与不同模型的人工指标高度一致，差异小于10%。在ChatGPT、Okanagan、CodQwen1.5 Chat模型中，二者的对齐度非常好，差异小于3%。另一方面，良好问题率存在一些差异，尤其是在CodeLlama、CodeQwen1.5 Chat和DeepSeek Coder等模型中。这可能是由于人工与自动化系统在判断问题是否“优秀”时使用的标准不同。被招募的学生在判断“优秀”问题时标准更为明确且要求更高，而基于LLM的自动评估器则倾向于标记更多应被标记为“合格”或“不合格”的问题为“优秀”。

<svg class="ltx_picture" height="171.84" id="S4.SS3.p8.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,171.84) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 25.59 21.65)"><foreignobject color="#000000" height="128.53" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="548.82">Answer to RQ3: The LLM-based evaluator provides acceptable answers to models’ responses, with higher than 50% Acceptable Answer Rate for all models and Good Answer Rate for most models. False Recovery Rates need to be further reduced for some models to ensure the reliability of test pass rates and Pass@1\. The Communication Rate in automated evaluation aligns reasonably well with the manual evaluation, but for the Good Question Rate, LLM-based evaluator tends to mark more “Good” questions than it should. Overall, the two metrics can provide useful insights to guide our experiment, but they may not always align with human judgment, particularly for subjective criteria like Good Question Rate.</foreignobject></g></g></svg>

### 4.4\. 调查提示策略和超参数的不同影响 (RQ4)

#### 4.4.1\. 调查不完整修改的不同程度 (1p)

为了深入挖掘不完全修改（1p）结果，我们进行了进一步的调查，以了解在1p问题描述中不同去除内容比例下的对应结果。我们通过随机删除问题描述中的一个文本块来调查这些结果。具体来说，对于每个问题描述中的单词，我们随机删除一组连续的单词，该组单词的数量为问题描述总单词数的$X$百分比。在本实验中，我们经验性地选择了$X=30\%,50\%,90\%$的值。表[10](https://arxiv.org/html/2406.00215v2#S4.T10 "Table 10 ‣ 4.4.3\. Investigating Different Prompts ‣ 4.4\. Investigating Different Impacts of Prompt Strategies and Hyperparameters (RQ4) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")展示了不同$X$值的一个问题描述示例。我们可以看到，即使$X$值小至30%，LLM也应该提问以获取缺失的信息，以确保LLM能够充分理解问题并生成高质量的代码。当$X$值增大，比如达到90%时，问题描述几乎无法进行高精度的代码生成任务。

为了降低本研究中随机性的风险，对于每个问题，我们进行了5次实验，并报告了在(Ouyang et al., [2023](https://arxiv.org/html/2406.00215v2#bib.bib51))中使用的度量标准，包括均值（Mean）、方差（Variance）、最大差异（Max Diff）和最差比例（Ratio of Worst）。我们计算了5次测试通过率和通信率的均值和方差，并报告了所有问题的平均值，作为均值（Mean）和方差（Variance）。最大差异（Max Diff）是所有问题中最大差异的最大值。“最差比例（Worst Cases Ratio）”是测试通过率最大差异为1的情况所占问题的比例。有关这些度量标准的完整描述，请参阅(Ouyang et al., [2023](https://arxiv.org/html/2406.00215v2#bib.bib51))。我们在表格[7](https://arxiv.org/html/2406.00215v2#S4.T7 "Table 7 ‣ 4.4.3\. Investigating Different Prompts ‣ 4.4\. Investigating Different Impacts of Prompt Strategies and Hyperparameters (RQ4) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")中报告了基于移除描述百分比（0%、30%、50%和90%）的测试通过率和通信率。我们在此次研究中使用了ChatGPT 3.5模型。我们可以看到，就测试通过率而言，随着移除信息的百分比增加，均值测试通过率明显下降，这表明不完整的问题描述会负面影响通过测试的能力。方差和最大差异度量进一步支持了这一趋势，随着信息的移除，测试通过率的变化性和差异性增加。最差案例的比例也表明，移除信息的百分比越高，测试通过率越低。

对于沟通率，当问题描述不完整时，随着去除信息比例的增加，均值沟通率也随之提高。这是预期中的结果，因为当信息缺失更多时，大型语言模型（LLM）倾向于提出更多问题，而不是直接生成代码。方差和最大差异指标也反映了沟通率的更高变异性和差异性，尤其是在问题描述不完整的情况下。最差情况下的比例表明，去除信息的比例越高，沟通率的增加越显著。[图6](https://arxiv.org/html/2406.00215v2#S4.F6 "Figure 6 ‣ 4.4.1\. Investigating Different Extent of Incomplete Modification (1p) ‣ 4.4\. Investigating Different Impacts of Prompt Strategies and Hyperparameters (RQ4) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")展示了这些指标的可视化对比，使用了表[7](https://arxiv.org/html/2406.00215v2#S4.T7 "Table 7 ‣ 4.4.3\. Investigating Different Prompts ‣ 4.4\. Investigating Different Impacts of Prompt Strategies and Hyperparameters (RQ4) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")中数据，比较了在不同X%内容被移除的情况下，问题描述的沟通率。可视化结果表明，当问题描述中更多内容被删除时，LLM倾向于提出更多问题，但这种情况只有在删除一半描述后才开始出现。当90%的描述被移除时，LLM仅在54%的问题描述中提出问题。总结来说，即使随机去除50%的问题描述，Code LLM的95%的回应仍然能够生成代码。当去除描述的比例增加到90%时，46%的Code LLM回应仍然能够生成代码。这表明，在信息随机去除的情况下，Code LLM提出澄清问题的能力较弱，因而指出了许多推动LLM或LLM代理向人类软件工程师水平发展的研究机会。

![请参见图注](img/0c397bc08dc0a9efe4e6e80c82a2d120.png)

图6. 在不同X%内容被移除的情况下，沟通率和测试通过率的图表。

#### 4.4.2\. 探索不同的超参数

为了评估超参数在实验中的影响，我们还探讨了使用不同温度作为ChatGPT的超参数。温度是一个控制ChatGPT生成文本随机性的超参数。ChatGPT的默认温度为1.0，我们测试了温度分别为0、1.0、2.0时的结果。与之前的“不完全修改”实验相同，我们进行了5次实验，并检查了包括均值和方差在内的指标。我们运行时去除信息的比例为50%。

表格 [8](https://arxiv.org/html/2406.00215v2#S4.T8 "Table 8 ‣ 4.4.3\. Investigating Different Prompts ‣ 4.4\. Investigating Different Impacts of Prompt Strategies and Hyperparameters (RQ4) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent") 展示了在 HumanEvalComm 中温度超参数对 ChatGPT 的影响的结果。我们发现，平均测试通过率从49.6%（方差=0.088，最大差异=0.523，最差比例=0.390）下降到40.7%（方差=最大差异=最差比例=0），而沟通率从1.8%（方差=0.011，最大差异=0.067，最差比例=0.067）变为3.7%（方差=最大差异=最差比例=0），当温度从1降到0时。我们还测试了温度为2.0，但发现由于温度过高，OpenAI 服务器端处理时间较长，导致大多数请求超时。我们可以看到，当温度为0时，方差、最大差异和最差比例都变为0。这意味着较低的温度确实表示结果更加确定和集中的。有趣的是，随着温度从1降到0，平均测试通过率下降，但沟通率却上升。这可能是因为较低的温度导致输出的创造性和多样性减少，从而“猜测”代码的能力下降，因此某种程度上迫使模型提出问题以寻求额外的信息。考虑到1.0的温度是默认设置，并且结果没有显示温度变化的显著影响，在评估中，我们使用1.0的温度设置，适用于ChatGPT和Okanagan。

#### 4.4.3\. 调查不同的提示

为了评估提示工程的影响，我们使用不同的提示模板和策略评估了结果。附录中的表格[14](https://arxiv.org/html/2406.00215v2#A1.T14 "Table 14 ‣ Appendix A APPENDIX: Prompt Templates In HumanEvalComm ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")显示了HumanEvalComm评估中不同的提示模板。我们设计了实验，通过在评估中创建两个主要提示的变体：1）初始提示（表格[14](https://arxiv.org/html/2406.00215v2#A1.T14 "Table 14 ‣ Appendix A APPENDIX: Prompt Templates In HumanEvalComm ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")中的第一行），提示模型要么提问，要么生成代码；2）LLM评估器的提示（表格[14](https://arxiv.org/html/2406.00215v2#A1.T14 "Table 14 ‣ Appendix A APPENDIX: Prompt Templates In HumanEvalComm ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")中的第二行），提供模型问题的答案。提示变体显示在表格[15](https://arxiv.org/html/2406.00215v2#A1.T15 "Table 15 ‣ Appendix A APPENDIX: Prompt Templates In HumanEvalComm ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")中。

表格[9](https://arxiv.org/html/2406.00215v2#S4.T9 "Table 9 ‣ 4.4.3\. Investigating Different Prompts ‣ 4.4\. Investigating Different Impacts of Prompt Strategies and Hyperparameters (RQ4) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")显示了在评估中不同提示变体的结果。我们选择了ChatGPT作为LLM的代表模型，Okanagan作为LLM代理。根据结果，我们可以看到，在使用第一个提示变体和第一个提示单次时，良好问题率受到了影响，ChatGPT的绝对增加了22%，而Okanagan下降了3%。这表明良好问题率很容易受到提示工程的影响，这与我们在RQ3中发现的该指标的不可靠性一致。另一方面，这些提示变体对通信率（-2%到+13%）、Pass@1（0%到+9%）和测试通过率（+2%到+11%）的影响较小，通常在10%以内。因此，未来的工作应集中在如何减少提示工程对这类评估的影响。由于成本限制，我们没有对基于LLM评估器的提示变体和单次提示变体进行实验，这基于我们在第一个提示变体中的实验结果。

| % 移除 | 指标 | 平均值 | 方差 | 最大差异 | 最差比例 |
| --- | --- | --- | --- | --- | --- |
| 0% | 测试通过率 | 66.5% | 0.122 | 0.690 | 0.561 |
| 0% | 通信率 | 3.3% | 0.019 | 0.110 | 0.110 |
| 30% | 测试通过率 | 49.6% | 0.088 | 0.532 | 0.390 |
| 30% | 通信率 | 1.8% | 0.011 | 0.067 | 0.067 |
| 50% | 测试通过率 | 41.1% | 0.085 | 0.523 | 0.317 |
| 50% | 通信率 | 5.5% | 0.028 | 0.165 | 0.165 |
| 90% | 测试通过率 | 44.8% | 0.124 | 0.684 | 0.543 |
| 90% | 通信率 | 54.1% | 0.120 | 0.604 | 0.604 |

表 7. 在问题描述中移除不同百分比的内容时，测试通过率和通信率的平均值结果。由于实验中涉及到随机性，实验进行了 5 次，并报告了在 (Ouyang 等, [2023](https://arxiv.org/html/2406.00215v2#bib.bib51)) 中使用的度量指标。

| % 移除 | 温度 | 类别 | 平均值 | 方差 | 最大差异 | 最差比例 |
| --- | --- | --- | --- | --- | --- | --- |
| 50% | 1 | 测试通过率 | 0.496 | 0.088 | 0.532 | 0.390 |
| 50% | 1 | 通信率 | 0.018 | 0.011 | 0.067 | 0.067 |
| 50% | 0 | 测试通过率 | 0.407 | 0 | 0 | 0 |
| 50% | 0 | 通信率 | 0.037 | 0 | 0 | 0 |

表 8. 不同温度作为 LLM 在 HumanEvalComm 中的超参数时，测试通过率和通信率的平均值结果。

| 提示 | 模型 | 通信率 | 好问题率 | Pass@1 | 测试通过率 |
| --- | --- | --- | --- | --- | --- |
| 原始提示 | ChatGPT | 14.21% | 13.43% | 31.34% | 49.39% |
| 第一个提示变体 | ChatGPT | 27.24% (+13.03%) | 36.19% (+22.76%) | 41.12% (+9.78%) | 60.45% (+11.06%) |
| 第一个提示单次 | ChatGPT | 17.68% (+3.47%) | 36.13% (+22.70%) | 38.84% (+7.50%) | 58.61% (+9.22%) |
| 原始提示 | Okanagan | 72.73% | 52.24% | 39.62% | 56.98% |
| 第一个提示单次 | Okanagan | 70.56% (-2.17%) | 48.77% (-3.47%) | 40.34% (+0.72%) | 59.38% (+2.40%) |

表 9. 针对选定模型的第一个提示在评估中的不同提示变体结果。每行显示一个模型的评估指标结果。

| 在描述中移除的百分比 | 示例问题描述 |
| --- | --- | --- | --- | --- | --- |
| 0% | def encode_cyclic(s: str): 返回通过循环分组的三字符编码字符串。# 将字符串拆分为每组，每组长度为 3。groups = [s[(3 * i):min((3 * i + 3), len(s))] 对于 i 在 range((len(s) + 2) // 3)] # 循环每组中的元素。除非组中元素少于 3。groups = [(group[1:] + group[0]) 如果 len(group) == 3 否则 group 对于 group 在 groups 中] 返回 "".join(groups) def decode_cyclic(s: str): 输入通过 encode_cyclic 函数编码的字符串。返回解码后的字符串。 |
| 30% | def encode_cyclic(s: str): 返回通过循环分组的三字符编码字符串。将字符串拆分为每组。除非组中元素少于 3 个。groups = [(group[1:] + group[0]) 如果 len(group) == 3 否则 group 对于 group 在 groups 中] 返回 "".join(groups) def decode_cyclic(s: str): 输入通过 encode_cyclic 函数编码的字符串。返回解码后的字符串。 |
| 50% | def encode_cyclic(s: str): 通过循环三字符组来返回编码后的字符串。将字符串拆分为组。每组长度为3。groups = [s[(3 * i):min((3 * i + 采取的输入字符串与encode_cyclic函数一起编码。返回解码后的字符串。 |
| 90% | def encode_cyclic(s: str): encode_cyclic函数。返回解码后的字符串。 |

表10. 人类评估问题38中随机删除问题描述部分的示例。

| 第一个模型输入 | 你是一个专家级软件开发人员，编写高质量的代码。根据以下信息，请直接生成Python3代码（仅以markdown格式返回代码），或者提出澄清性问题：{python} def how_many_times(string: str, substring: str) -> int: """ 查找给定子字符串在原始字符串中出现的次数。 """ |
| --- | --- |
| 第一个模型响应 | 当然，我可以帮你解决这个问题。不过，我需要更多的信息才能提供精确的解决方案。能否请你提供以下细节？1\. 如果子字符串在字符串中未找到，应该如何处理？2\. 如果子字符串为空字符串，应该如何处理？3\. 是否有任何特定的约束或要求需要考虑？有了这些细节，我可以提供一个更准确的解决方案。 |
| 基于LLM的评估器输出 | 问题的答案：1\. 如果子字符串在字符串中未找到，函数应返回0。2\. 如果子字符串为空字符串，函数应返回0。3\. 除了计算重叠情况外，没有其他特定的约束或要求。 |
| 第二模型输入 | [第一个模型输入][1st Model Response][问题的答案]… 根据上述对话，直接生成Python代码（Markdown格式）来解决编码问题： |
| 第二模型响应 | 当然，根据提供的信息，下面是一个可以解决问题的Python函数： {python} def how_many_times(string: str, substring: str) -> int: """ 计算子字符串在字符串中不重叠的出现次数。 参数： string (str): 要搜索的字符串。 substring (str): 要搜索的子字符串。 返回： int: 子字符串在字符串中出现的次数，或者如果子字符串为空，则返回0。 """ if not substring: return 0 count = 0 start = 0 while start < len(string): pos = string.find(substring, start) if pos != -1: count += 1 # 确保不重叠的出现 start = pos + len(substring) else: break return count |

表11. DeepSeek Coder在问题18中的示例案例。该问题的澄清类型为1p（不完整性）。此示例的沟通率为100%，良好的问题率为良好，测试通过率为100%。

| 第一模型输入 | 你是一个专家级的软件开发人员，编写高质量代码。根据以下信息，请直接生成Python3代码（仅回复代码并使用Markdown），或者提出澄清问题： {python} def triangle_area(a, h): """给定一边长和高度，返回三角形的面积。 >>> triangle_area(5, 3) 15 """ |
| --- | --- |
| 第一模型响应 | 这是“triangle_area”函数的Python代码： {python} def triangle_area(a, h): return (a * h) / 2 该函数接收两个参数，“a”和“h”，分别表示三角形的边长和高度。它返回三角形的面积，计算方法是将边长与高度相乘后再除以2。以下是如何使用该函数的示例： {python} print(triangle_area(5, 3)) # 输出：15 这将打印出一个边长为5、高度为3的三角形的面积，即15。 |
| 基于LLM的评估器输出 | 回答问题：N/A |
| 第二模型输入 | [第一模型输入][1st Model Response][回答问题]… 根据以上对话，直接生成解决编程问题的Python代码（Markdown）： |
| 第二模型响应 | 根据对话内容，以下是解决编程问题的Python代码： {python} def triangle_area(a, h): return (a * h) / 2 print(triangle_area(5, 3)) # 输出：15 此代码定义了一个“triangle_area”函数，接收两个参数，“a”和“h”，分别表示三角形的边长和高度。该函数返回三角形的面积，通过将边长与高度相乘后再除以2来计算。代码还调用了“triangle_area”函数，并传入“5”和“3”作为参数，打印出三角形的面积，即‘15’。 |

表12. 这是CodeLlama在HumanEvalComm问题45中的示例案例。此问题的澄清类型为1c（不一致）。由于返回了代码块和文本，而不仅仅是代码块，通信率为100%。良好问题率为差，表示未检测到澄清性问题。

<svg class="ltx_picture" height="134.84" id="S4.SS4.SSS3.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,134.84) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 25.59 21.65)"><foreignobject color="#000000" height="91.53" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="548.82">Answer to RQ4: while LLMs begin to ask more clarifying questions after 50% of the description is removed, even with 90% removed, only 54% of problem descriptions prompted questions. Variations in temperature hyper-parameter did not significantly affect performance. On the other hand, prompt engineering has a larger impact on Good Question Rate, but less impact (within 10% absolute change) on Communication Rate, Pass@1 and Test Pass Rate.</foreignobject></g></g></svg>

## 5\. 讨论

在本节中，我们提供了一些示例案例，以帮助理解Code LLMs和Okanagan的结果。然后，我们还讨论了 1) 为什么不使用自动化修改，以及 2) 基于LLM的评估器如何影响评估结果。最后，我们分享了这项研究的启示和未来的工作。

### 5.1\. 示例案例

除了数字结果外，我们还研究了两个案例，以更好地理解评估步骤。表[11](https://arxiv.org/html/2406.00215v2#S4.T11 "Table 11 ‣ 4.4.3\. Investigating Different Prompts ‣ 4.4\. Investigating Different Impacts of Prompt Strategies and Hyperparameters (RQ4) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")展示了HumanEvalComm中一个成功示例问题（HumanEval/18）的结果。这个问题的澄清类型是1p（不完整性），因此问题描述中缺少了“计数重叠情况”的要求。给定第一个模型输入，DeepSeek Coder能够检测到问题中的不完整要求，并提出澄清性问题，而不是直接生成代码。因此，这个单一案例的沟通率为100%。使用第一个模型输入和响应，LLM-based评估器为我们的评估输出了好问题率，并回答了澄清性问题。这个答案提供了原始问题中缺失的“计数重叠情况”的要求。

在第二个模型输入中，模型给出了之前的对话，包括第一个模型输入、第一个模型回应（澄清性问题）和LLM-based评估器对澄清性问题的回答。根据“计数重叠情况”的要求，模型正确地解决了问题，测试通过率为100%。请注意，我们可以再次让模型提问或生成代码，就像第一个模型输入中的指令一样，但我们选择仅进行一个问答轮次，以简化我们的评估。

另一个案例展示在表[12](https://arxiv.org/html/2406.00215v2#S4.T12 "Table 12 ‣ 4.4.3\. Investigating Different Prompts ‣ 4.4\. Investigating Different Impacts of Prompt Strategies and Hyperparameters (RQ4) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")中，针对HumanEvalComm中的另一个示例问题（HumanEval/45）使用CodeLlama。这个问题的澄清类型是1c（不一致性）：在测试用例中，triangle_area（5，3）返回15而不是7.5。这导致测试用例和问题描述之间存在不一致性。给定第一个模型输入，CodeLlama并未仅根据指令返回代码块，而是返回了代码块和文本解释的混合。因此，这个案例的沟通率为100%，但基于LLM的评估器输出了1（没有问题）的好问题率。在这种情况下，不一致性问题未被捕捉到，也没有提出澄清性问题。对于评估指标，Bad的好问题率成功惩罚了这个案例，但测试通过率和沟通率未能捕捉并惩罚该问题。

### 5.2\. 为什么不是自动化修改？

一个自然的问题是，是否可以使用自动化修改代替需要手工制作数据集的手动修改。在我们开始手动修改之前，需要注意的是，我们曾尝试使用LLM修改问题描述，但发现LLM的修改并未达到我们的标准。具体来说，使用LLM进行的修改不能确保触发澄清性问题。关于使用LLM进行需求工程的类似局限性，也有在(Arvidsson 和 Axell，[2023](https://arxiv.org/html/2406.00215v2#bib.bib5))中报道过。

我们设计了一个提示语，用于生成所有164个HumanEval问题的自动修改版本。我们已将这164个问题的修改结果上传到[此链接](https://github.com/jie-jw-wu/human-eval-comm/tree/main/auto_modified_problems)。表[13](https://arxiv.org/html/2406.00215v2#S5.T13 "Table 13 ‣ 5.5\. Limitations ‣ 5\. Discussion ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")展示了两个示例的结果。我们可以看到，在第一个示例中，“一个正浮点数”被改为“一个数字”，意图是使问题变得模糊。尽管进行了这样的修改，但它并未在生成正确代码之前触发澄清性问题，而这是本研究中修改问题所需要的。在第二个示例中，添加了一个新测试用例以使问题不一致，但这个修改并未造成任何冲突或不一致。因此，我们选择手动修改所有问题描述，以确保HumanEvalComm的可靠性。

### 5.3\. 基于LLM的评估器如何影响Okanagan和其他模型的结果？

我们进一步讨论了一些在通过率和优质问题率等指标上表现优于Okanagan的模型，如表[3](https://arxiv.org/html/2406.00215v2#S4.T3 "表3 ‣ 4.1\. 基于代码LLM的沟通能力评估 (RQ1) ‣ 4\. 结果与分析 ‣ 基准测试代码生成LLM和LLM智能体的沟通能力")所示，从基于LLM的评估者角度来看。对于优质问题率，DeepSeek Coder和DeepSeek Chat在以ChatGPT（或DeepSeek Code）为基础的情况下，获得的优质问题率与Okanagan相当或更高。一个潜在的原因是，基于LLM的评估者倾向于为DeepSeek Coder和DeepSeek Chat标记更多的“优质”问题，如图[5](https://arxiv.org/html/2406.00215v2#S4.F5 "图5 ‣ 4.3\. 基于LLM的评估者的人工评估 (RQ3) ‣ 4\. 结果与分析 ‣ 基准测试代码生成LLM和LLM智能体的沟通能力")所示。对于测试通过率和Pass@1，CodeQwen 1.5 Chat和DeepSeek Coder在以ChatGPT（或DeepSeek Code）为基础的情况下，取得了比Okanagan更好的成绩。其原因之一是CodeQwen 1.5 Chat和DeepSeek Coder的虚假恢复率远高于Okanagan。这可能导致即使CodeQwen 1.5 Chat和DeepSeek Coder没有提出任何相关问题，评估者仍然会向它们发送相关信息，从而提高测试通过率和Pass@1。未来的工作需要减少基于LLM的评估者引入的偏差。

### 5.4\. 其他基于LLM的智能体是如何工作的？

为了实现更全面的比较，我们实现并评估了AgentCoder（Huang et al., [2023](https://arxiv.org/html/2406.00215v2#bib.bib30)）。AgentCoder（Huang et al., [2023](https://arxiv.org/html/2406.00215v2#bib.bib30)）是领先的基于智能体的代码生成管道之一，在HumanEval基准测试中，使用GPT4时的Pass@1得分为96.3，使用ChatGPT时为79.9，这使得它成为本研究理想的基准。

在实现过程中，我们对AgentCoder进行了代码更改，使其能够适应我们在HumanEvalComm中的场景。在评估的第一轮，我们运行了AgentCoder，要求模型要么生成代码，要么提出澄清问题。为了这一步骤，我们不得不改变AgentCoder的功能。AgentCoder原本是为HumanEval数据集设计的，但我们将其应用于新的HumanEvalComm数据集，该数据集修改了提示，要求模型生成澄清问题。因此，在第一轮中，我们仅使用AgentCoder中的程序员组件，因为回应不仅限于代码，还包括澄清问题。这导致生成的完成列表包含澄清问题或代码。在第二轮中，我们回答这些问题。在第三轮中，我们运行整个AgentCoder，结合前两轮的所有上下文。我们将我们的实现发布到了我们的GitHub仓库，链接在[此处](https://github.com/jie-jw-wu/human-eval-comm/blob/main/README_AgentFramework.md)，其中包含了我们实现和更改的详细信息。然而，我们发现AgentCoder的回应中没有（0%）提出澄清问题，尽管我们尽最大努力调整AgentCoder，使其能够生成带有澄清问题的代码。这表明，LLM代理通常具有专业化和定制化的任务，因此很难直接适应我们在HumanEvalComm中提出澄清问题的任务。未来的工作需要更加便捷地调整LLM代理方法以适应类似基准测试。

### 5.5\. 局限性

我们已经尽最大努力对不同模型进行评估。然而，在这项研究过程中，我们也识别出了一些局限性。这些局限性总结如下。

修改过程中对人力资源的高需求。如[5.2](https://arxiv.org/html/2406.00215v2#S5.SS2 "5.2\. 为什么不使用自动化修改？ ‣ 5\. 讨论 ‣ 基准测试LLM和LLM代理的代码生成沟通能力")部分所述，使用自动化技术的质量远低于人工精心处理的方法。因此，为了确保HumanEvalComm数据集的最高标准，我们选择手动整理数据集。然而，这一手动修改问题描述的过程跨越了多个类别，需付出大量劳动力成本，并且极度依赖人工专业知识，这对于资源有限的研究环境而言尤其具有挑战性。依赖经验丰富的软件工程师来审查和修改每个问题显著增加了研究的成本和时间。值得一提的是，其他研究人员使用HumanEvalComm时并不需要手动操作。因此，资源受限的研究团队可以直接使用我们的基准进行模型测试。

人工修改问题的主观性。虽然人工修改是必要的，以确保修改能触发澄清性问题，但不同的工程师可能以不同的方式处理同一个问题。尽管我们已经按照[第2节](https://arxiv.org/html/2406.00215v2#S2 "2. 基准构建 ‣ 针对LLM和LLM代理的代码生成通信能力基准测试")中提到的具体指南来修改问题集，但由于修改空间巨大，工程师的主观性仍可能影响HumanEvalComm中修改问题的质量和标准化。

Okanagan倾向于提出不必要的问题。尽管Okanagan在修改问题上表现出较高的沟通率和代码生成性能，但它有时会对原始问题提出不必要的问题，这些问题并不需要澄清。理想情况下，一个完美的模型应能够智能地判断何时提问以及何时不提问，但这种理想行为与包括Okanagan在内的所有评估模型的表现之间仍存在差距。

基于LLM的评估器引入的潜在偏差。本研究采用基于LLM的评估器来评估模型生成的澄清性问题的质量，并为这些问题生成答案。尽管这种方法证明是有用的，但它也引入了若干限制，这些限制可能影响结果的普适性：

+   •

    自动评估器确定的好问题率与人工注释员进行的人工评估之间存在差异。基于LLM的评估器倾向于将更多的问题标记为“好问题”，而不是人工注释员。这意味着评估器在某些情况下可能高估了澄清性问题的质量。例如，评估器有时会在模型没有提出任何澄清性问题的情况下，仍然为问题分配“好”的质量标签。在评估结果方面，好问题率可以提供有用的见解，但由于评估器的潜在偏差，它们可能并不总是与人工判断一致。

+   •

    好问题率更容易受到评估器的不同提示变体的影响。这与我们在RQ3中发现的好问题率差异一致。因此，关于评估结果，好的问题率更容易受到提示工程的影响，表明这一指标的不可靠性。

+   •

    一些模型，如CodeQwen1.5 Chat和DeepSeek Coder的虚假恢复率明显高于Okanagan。这表明，即使这些模型没有提出任何相关的澄清性问题，评估器有时仍会向它们提供相关信息。因此，这可能人为地提高了它们的测试通过率和Pass@1得分，复杂化了通过这些指标评估其沟通技能的过程。

| 提示模板 | 输出（自动修改问题） |
| --- | --- |
| 基于这样一个知识点：模糊的问题描述可以通过引入多种有效的解释或不明确的细节来创建，逐步思考，重写给定的编程问题描述，并使其变得模糊。只输出修改后的问题描述本身。原始问题描述：[⬇](data:text/plain;base64,ZGVmIHRydW5jYXRlX251bWJlcihudW1iZXI6IGZsb2F0KSAtPiBmbG9hdDoKICAgICIiIiBHaXZlbiBhIHBvc2l0aXZlIGZsb2F0aW5nIHBvaW50IG51bWJlciwgaXQgY2FuIGJlIGRlY29tcG9zZWQgaW50bwogICAgYW4gaW50ZWdlciBwYXJ0IChsYXJnZXN0IGludGVnZXIgc21hbGxlciB0aGFuIGdpdmVuIG51bWJlcikgYW5kIGRlY2ltYWxzCiAgICAobGVmdG92ZXIgcGFydCBhbHdheXMgc21hbGxlciB0aGFuIDEpLgoKICAgIFJldHVybiB0aGUgZGVjaW1hbCBwYXJ0IG9mIHRoZSBndW1iZXIuCiAgICA+Pj4gdHJ1bmNhdGVfbnVtYmVyKDMuNSkKICAgIDAuNQogICAgIiIi) def  truncate_number(number:  float)  ->  float: """␣给定␣一个␣正␣浮点␣数，␣它␣可以␣被␣分解␣为␣一个␣整数␣部分（最大␣小于␣给定␣数的␣整数）和␣小数部分␣（剩余部分␣始终␣小于␣1）。 ␣␣␣␣返回␣这个␣数的␣小数部分。 ␣␣␣␣>>>␣truncate_number(3.5) ␣␣␣␣0.5 ␣␣␣␣""" 模糊问题：[⬇](data:text/plain;base64,ZGVmIHRydW5jYXRlX251bWJlcihudW1iZXI6IGZsb2F0KSAtPiBmbG9hdDoKICAgICIiIiBHaXZlbiBhIG51bWJlciwgaXQgY2FuIGJlIGRlY29tcG9zZWQgaW50byBhbiBpbnRlZ2VyIHBhcnQgKGxhcmdlc3QgaW50ZWdlciBzbWFsbGVyIHRoYW4gZ2l2ZW4gbnVtYmVyKSBhbmQgZGVjaW1hbHMKICAgIChsZWZ0b3ZlciBwYXJ0IGFsd2F5cyBzbWFsbGVyIHRoYW4gMSkuCgogICAgUmV0dXJuIHRoZSBkZWNpbWFsIHBhcnQgb2YgdGhlIG51bWJlci4KICAgID4+PiB0cnVuY2F0ZV9udW1iZXIoMy41KQogICAgMC41CiAgICAiIiIi) def  truncate_number(number:  float)  ->  float: """␣给定␣一个␣数，␣它␣可以␣被␣分解␣为␣一个␣整数␣部分（最大␣小于␣给定␣数的␣整数）和␣小数部分␣（剩余部分␣始终␣小于␣1）。 ␣␣␣␣返回␣这个␣数的␣小数部分。 ␣␣␣␣>>>␣truncate_number(3.5) ␣␣␣␣0.5 ␣␣␣␣"""（解释：“正␣浮点␣数”改为“一个␣数”，目的是使问题变得模糊。尽管进行了这一修改，但它并未在生成正确的代码之前触发澄清问题，这是在本研究中制定修改问题所需的。） |
| 基于一个问题描述中某些陈述存在冲突时问题变得不一致的知识，逐步思考并重写给定的编码问题描述，使其变得不一致。仅输出修改后的问题描述本身。原始问题描述：[⬇](data:text/plain;base64,ZnJvbSB0eXBpbmcgaW1wb3J0IExpc3QKCmRlZiBmaWx0ZXJfYnlfc3Vic3RyaW5nKHN0cmluZ3M6IExpc3Rbc3RyXSwgc3Vic3RyaW5nOiBzdHIpIC0+IExpc3Rbc3RyXToKICAgICIiIiBGaWx0ZXIgYW4gaW5wdXQgbGlzdCBvZiBzdHJpbmdzIG9ubHkgZm9yIG9uZXMgdGhhdCBjb250YWluIGdpdmVuIHN1YnN0cmluZwogICAgPj4+IGZpbHRlcl9ieV9zdWJzdHJpbmcoW10sICdhJykKICAgIFtdCiAgICA+Pj4gZmlsdGVyX2J5X3N1YnN0cmluZyhbJ2FiYycsICdiYWNkJywgJ2NkZScsICdhcnJheSddLCAnYScpCiAgICBbJ2FiYycsICdiYWNkJywgJ2FycmF5J10KICAgICIiIg==) 来自 typing 导入 List 定义 filter_by_substring（strings：List[str]，substring：str）-> List[str]： """␣过滤␣输入␣字符串␣列表，只保留包含给定子字符串的项 ␣␣␣␣>>>␣filter_by_substring([],␣’a’) ␣␣␣␣[] ␣␣␣␣>>>␣filter_by_substring([’abc’，␣’bacd’，␣’cde’，␣’array’]，␣’a’) ␣␣␣␣[’abc’，␣’bacd’，␣’array’] ␣␣␣␣""" 不一致问题：[⬇](data:text/plain;base64,ZnJvbSB0eXBpbmcgaW1wb3J0IExpc3QKCmRlZiBmaWx0ZXJfYnlfc3Vic3RyaW5nKHN0cmluZ3M6IExpc3Rbc3RyXSwgc3Vic3RyaW5nOiBzdHIpIC0+IExpc3Rbc3RyXToKICAgICIiIiBGaWx0ZXIgYW4gaW5wdXQgbGlzdCBvZiBzdHJpbmdzIG9ubHkgZm9yIG9uZXMgdGhhdCBjb250YWluIGdpdmVuIHN1YnN0cmluZwogICAgPj4+IGZpbHRlcl9ieV9zdWJzdHJpbmcoW10sICdhJykKICAgIFtdCiAgICA+Pj4gZmlsdGVyX2J5X3N1YnN0cmluZyhbJ2FiYycsICdiYWNkJywgJ2NkZScsICdhcnJheSddLCAnYScpCiAgICBbJ2FiYycsICdiYWNkJywgJ2FycmF5J10KICAgID4+PiBmaWx0ZXJfYnlfc3Vic3RyaW5nKFsnYWJjJywgJ2JhY2QnLCAnY2RlJywgJ2FycmF5J10sICdiJykKICAgIFsnYWJjJywgJ2JhY2QnXQogICAgIiIi) 来自 typing 导入 List 定义 filter_by_substring（strings：List[str]，substring：str）-> List[str]： """␣ 过滤␣输入␣字符串␣列表，只保留包含给定子字符串的项 ␣␣␣␣>>>␣filter_by_substring([],␣’a’) ␣␣␣␣[] ␣␣␣␣>>>␣filter_by_substring([’abc’，␣’bacd’，␣’cde’，␣’array’]，␣’a’) ␣␣␣␣[’abc’，␣’bacd’，␣’array’] ␣␣␣␣>>>␣filter_by_substring([’abc’，␣’bacd’，␣’cde’，␣’array’]，␣’b’) ␣␣␣␣[’abc’，␣’bacd’] ␣␣␣␣"""（解释：添加了一个新的测试示例使问题变得不一致，但此修改并未引发任何冲突或不一致。） |

表 13. 使用LLM自动修改的问题。给定来自 HumanEval 数据集的问题描述，我们使用 Google Gemini 作为 LLM 来修改问题描述，使得修改后的问题变得模糊、不一致或不完整。

## 6\. 启示与未来工作

基于结果和分析，我们总结了以下作为未来工作的建议：

+   •

    为编码任务中的沟通能力开发更好的评估方法。我们的主要贡献是新的 HumanEvalComm 基准，用于评估 LLM 和 LLM 代理在执行编码任务时的沟通能力，但未来的研究仍然需要改进评估方法。这包括

    1.  （1）

        探索自动化或半自动化技术，减少对人力资源的依赖，使该方法在资源有限的环境中更具可访问性。

    1.  (2)

        解决基于大语言模型评估器中的偏差问题，例如采用FM/LLM陪审团的方法（Li等人，[2024](https://arxiv.org/html/2406.00215v2#bib.bib39)）。

    1.  (3)

        提高评估指标的准确性和公平性，以更好地衡量和理解沟通能力。

    1.  (4)

        创建具有不同澄清难度的多样化数据集，以更全面地评估模型的沟通能力。

    我们相信，这是评估中的一个重要角度，因为越来越多的代理流模型和大语言模型（LLMs）将在不久的将来应用于软件工程任务。

+   •

    超越生成模型的局限性。在评估中，我们发现，在“不完整性”类别中，一些代码大语言模型的结果极低，这可能是由于它们的生成性质，偏向于根据统计模型生成并完成代码，即使描述显然是不完整的。未来的工作应该超越大语言模型的生成性质，发展成“AI代理”或“AI助手”（Hassan等人，[2024](https://arxiv.org/html/2406.00215v2#bib.bib27)），以进一步提升沟通能力。

+   •

    提升推理能力和信息收集能力。在评估中，“不一致性”类别的沟通率在三种类型中最低。这表明，未来需要更多的工作来开发具有更强推理能力的模型，以解决“不一致性”类别中的低表现。另从表[5](https://arxiv.org/html/2406.00215v2#S4.T5 "Table 5 ‣ 4.1\. Communication Competency of Code LLMs on HumanEvalComm (RQ1) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")的RQ1结果以及[4.4.1](https://arxiv.org/html/2406.00215v2#S4.SS4.SSS1 "4.4.1\. Investigating Different Extent of Incomplete Modification (1p) ‣ 4.4\. Investigating Different Impacts of Prompt Strategies and Hyperparameters (RQ4) ‣ 4\. Results and Analysis ‣ Benchmarking the Communication Competence of Code Generation for LLMs and LLM Agent")章节中的RQ4结果，我们注意到随着问题描述难度的增加，Pass@1和测试通过率有所下降。这可能表明，在解决编码任务时未能获取必要的信息，特别是在描述较为复杂的情况下。未来的工作应重点关注模型和评估设置，以提高模型在复杂情况和现实环境中推理问题描述和获取必要信息的能力。

+   •

    提高判断何时停止提问的能力。尽管LLM代理方法Okanagan在提高评估指标方面显示了有前景的初步结果，但Okanagan的一个限制是，它通过提出不必要的问题，降低了原始HumanEval基准的通过率。因此，LLM代理的未来工作之一是解决这一不足。这可能会大大增强通信能力，因为模型将能够智能地判断何时避免冗余或不相关的询问。

## 7.   效度威胁

构建效度。这个威胁与HumanEvalComm中问题手动修改的潜在不准确性有关。为了减轻这一威胁，我们已经手动检查并验证了所有问题超过三次，并且每次讨论问题时，他们都未能达成共识。尽管我们尽力而为，但仍可能存在一些极端情况，其中1）修改后的问题不符合模糊性和不一致性的定义，2）修改后的问题受主观差异影响，且与其他问题不一致。

内部效度。这个威胁与内部参数有关，如开源代码LLMs和ChatGPT中的参数，这些参数可能会影响结果。为了减轻这一威胁，我们在运行开源代码LLMs和ChatGPT时使用了大多数默认参数。对于开源模型，我们将max_new_tokens设置为512，以节省计算资源，并使用其他参数的默认值。对于ChatGPT，我们在OpenAI API中使用了温度（temperature）设置为1.0，$n$设置为1。另一个威胁与评估中使用的基于LLM的评估器的有效性有关。如前所述，为了减轻这个问题，我们已经多次优化了基于LLM的评估器的提示，并手动检查了结果。此外，基于LLM的评估器在评估中对所有模型的使用是平等的，因此这个威胁不会影响所有模型的相对排名。

外部有效性。这与模型在其他基准测试上通信能力的普遍性相关。为了解决这个问题，我们广泛报告并分析了通过统计测试得到的结果，其中包含p值。为了减少我们调查中由随机性引入的风险，我们还添加了均值、方差和最大差异等指标。因此，这些结果可能适用于其他数据集。然而，由于我们尚未进行此类测试，因此无法对模型在其他数据集上的通信能力做出确凿的声明。另一个威胁与被评估模型的实现相关。我们直接调用OpenAI API来获取ChatGPT结果。我们用Python实现了Okanagan，它调用OpenAI API。对于CodeLlama和其他开源模型，我们从HuggingFace下载模型，并在UBC ARC Sockeye上执行模型推理。从评估结果来看，我们认为我们的实现反映了原始方法。另一个威胁与我们在评估中使用的指标的普适性和可靠性相关。为了解决这个问题，我们添加了RQ3来进行人工评估，并使用其结果与提出的指标进行比较。我们还讨论了基于LLM的评估器对指标可靠性的潜在影响。为了确保评估结果的可复现性，我们广泛报告了案例研究的结果，并发布了完整的代码和数据集。这可以让其他研究人员复现并扩展我们在案例研究中的实验。

## 8\. 相关工作

大型语言模型的代码生成。近年来，随着大型语言模型的出现，代码生成领域发生了显著变化。例如，Codex（Chen等，[2021](https://arxiv.org/html/2406.00215v2#bib.bib14)）在大量源代码数据上对GPT-3（Brown等，[2020](https://arxiv.org/html/2406.00215v2#bib.bib12)）进行微调，能够在一次运行中为HumanEval数据集中的47/164个问题生成代码，这是代码生成任务的基准。Codex成为了GitHub开发的AI编程助手Copilot（Ziegler等，[2022](https://arxiv.org/html/2406.00215v2#bib.bib93)）的核心模型。在Codex之后，开发了一些与Codex类似但规模较小的模型，包括GPT-J（Wang和Komatsuzaki，[2021](https://arxiv.org/html/2406.00215v2#bib.bib72)）、CodeParrot（Face，[2023](https://arxiv.org/html/2406.00215v2#bib.bib18)）、PolyCoder（Xu等，[2022](https://arxiv.org/html/2406.00215v2#bib.bib81)）。AlphaCode（Li等，[2022](https://arxiv.org/html/2406.00215v2#bib.bib40)），其规模与Codex相当，使用GitHub数据进行了训练，并在竞赛级编程问题上进行了微调。在CodeForces（一个著名的在线竞赛编程平台）的编程比赛中，AlphaCode超越了超过一半的参赛者。CodeGen（Nijkamp等，[2022](https://arxiv.org/html/2406.00215v2#bib.bib49)）在自然语言和编程语言数据上进行了训练，支持使用多轮提示生成代码。最近，像CodeLlama（Roziere等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib58)）、DeepSeek Coder（Guo等，[2024b](https://arxiv.org/html/2406.00215v2#bib.bib25)）和CodeQwen1.5 Chat（Bai等，[2023](https://arxiv.org/html/2406.00215v2#bib.bib7)）等更新的模型，继续在如HumanEval等基准测试中取得更高的性能。然而，这些模型的交流能力并未被强调和评估。这些模型的评估是通过一次性问题描述生成代码，可能是一次或多次尝试，而没有来自对话的更多信息。因此，当输入问题描述容易出错或不完整时，模型仍然必须在没有机会澄清关键问题的情况下生成代码。我们的工作旨在探索解决这一可用性问题。

自我纠正的大型语言模型（LLMs）和LLM代理在人工智能中的应用。最近，一种有前景的提升大型语言模型输出效率的方法是自我纠正（Pan等， [2023](https://arxiv.org/html/2406.00215v2#bib.bib53)）。在自我纠正方法中，LLM通过自身引导或提示的反馈来优化其结果。一类流行的研究工作使用人类反馈直接优化其结果（Kreutzer等， [2018](https://arxiv.org/html/2406.00215v2#bib.bib38)；Glaese等， [2022](https://arxiv.org/html/2406.00215v2#bib.bib24)；Ouyang等， [2022](https://arxiv.org/html/2406.00215v2#bib.bib50)；Scheurer等， [2023](https://arxiv.org/html/2406.00215v2#bib.bib60)；Fernandes等， [2023](https://arxiv.org/html/2406.00215v2#bib.bib22)）。其他研究则采用不同策略，通过自动化反馈来进行LLM自我纠正，例如自我训练（Huang等， [2022](https://arxiv.org/html/2406.00215v2#bib.bib31)；Bai等， [2022](https://arxiv.org/html/2406.00215v2#bib.bib8)），生成-然后-排序（He等， [2023](https://arxiv.org/html/2406.00215v2#bib.bib28)；Weng等， [2023](https://arxiv.org/html/2406.00215v2#bib.bib76)），反馈引导解码（Yang等， [2022](https://arxiv.org/html/2406.00215v2#bib.bib84)；Xie等， [2023](https://arxiv.org/html/2406.00215v2#bib.bib80)），迭代后修订（Zhang等， [2023b](https://arxiv.org/html/2406.00215v2#bib.bib91)；Jiang等， [2023](https://arxiv.org/html/2406.00215v2#bib.bib33)）等。此外，LLM的进展也为基于LLM的代理带来了很多进展，其中包括规划、记忆、个人档案和行动等不同模块（Xi等， [2023](https://arxiv.org/html/2406.00215v2#bib.bib79)；Wang等， [2023b](https://arxiv.org/html/2406.00215v2#bib.bib74)），以及各种代理类别，如工具代理、模拟代理、网络代理、游戏代理等。我们的工作还包括对LLM代理方法Okanagan的评估，该方法在思考模式上增加了一轮反思。

LLM代理用于代码生成。尽管仍处于早期阶段，最近，越来越多的研究开始尝试使用LLM代理来执行代码生成任务（Yang et al., [2024](https://arxiv.org/html/2406.00215v2#bib.bib83); Zhang et al., [2024](https://arxiv.org/html/2406.00215v2#bib.bib90); aut, [2023](https://arxiv.org/html/2406.00215v2#bib.bib2); Yao et al., [2023](https://arxiv.org/html/2406.00215v2#bib.bib85); Wu, [2024](https://arxiv.org/html/2406.00215v2#bib.bib78); Shinn et al., [2023](https://arxiv.org/html/2406.00215v2#bib.bib63); Schick et al., [2023](https://arxiv.org/html/2406.00215v2#bib.bib61))。在人工智能领域，ReAct论文提出了一种将推理与行动集成到语言模型中的方法，以增强其决策能力（Yao et al., [2023](https://arxiv.org/html/2406.00215v2#bib.bib85)）。Reflexion介绍了一个框架，在这个框架中，语言代理通过语言强化学习来提高其表现，模拟类似人类的反思和适应（Shinn et al., [2023](https://arxiv.org/html/2406.00215v2#bib.bib63)）。Toolformer展示了语言模型如何自主学习使用外部工具，如搜索引擎和计算器，从而提高任务表现（Schick et al., [2023](https://arxiv.org/html/2406.00215v2#bib.bib61)）。SWE-Agent提出了一种专门为软件工程任务设计的新型代理，利用大型语言模型来提升生产力和决策能力（Yang et al., [2024](https://arxiv.org/html/2406.00215v2#bib.bib83)）。CoderAgent引入了一种旨在通过使用先进的语言模型来自动化编码任务，从而高效生成和优化代码片段的模型（Zhang et al., [2024](https://arxiv.org/html/2406.00215v2#bib.bib90)）。

在软件工程领域，RepairAgent（Bouzenia等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib11)）是首批使用基于LLM的代理进行程序修复和代码生成的系统之一。这项工作延续了将API工具与LLM结合的先前研究。最近，CoRE（Xu等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib82)）被提出作为一个系统，通过使用LLM作为解释器来处理和执行自然语言指令，进而实现代理编程。秉持类似LLM代理的理念，TICODER（Fakhoury等，[2024](https://arxiv.org/html/2406.00215v2#bib.bib19)）被提出作为一个基于测试驱动的交互式工作流，以生成更准确的代码。同样，De-Hallucinator（Eghbali和Pradel，[2024](https://arxiv.org/html/2406.00215v2#bib.bib17)）被提出作为一种代码补全方法，结合了基于检索的代码生成和模型的迭代查询。与上述工作不同，我们提出的Okanagan重点增强LLM在代码生成任务中的沟通能力。据我们所知，我们是首个研究并比较LLM代理与代码LLM在代码生成任务中的沟通能力的研究。

## 9\. 结论

在本文中，我们展示了在实证研究中，LLM（大语言模型）在评估代码澄清和代码生成中的沟通技能的初步步骤。我们认为，LLM的沟通技能的熟练程度对于AI系统生成符合高标准的代码是必要的，并且从长远来看，它有助于通过提问获取恰到好处的信息，以完成任务。我们相信，提升沟通能力应该被视为弥合LLM与顶尖软件开发者之间差距的重要因素。尽管这需要额外的对话输入，但我们认为，评估这种沟通能力对于编码任务仍然是必要且值得的。

作为这项工作的第一步，我们创建了 HumanEvalComm 来评估沟通技能的程度。基于这一新基准，我们从沟通的角度全面评估了不同的代码 LLM，其中某些信息在原始问题描述中被手动修改。此外，我们提出了一种基于 LLM 的代理方法 Okanagan，用于识别并在代码和描述的模糊部分提出问题，以进一步完善生成的代码。我们发现，修改问题描述显著降低了测试通过率和 Pass@1，并具有统计学意义。在沟通技能方面，当问题描述被手动修改时，超过 60% 的代码 LLM 的回答仍然生成代码，而不是提出问题。我们还发现，与 ChatGPT 3.5 等 LLM 相比，作为 LLM 代理方法的 Okanagan 能有效提高沟通率和良好问题率，从而在问题描述基于澄清类型进行修改时，提升测试通过率和 Pass@1。

除了基准测试，进一步提高 LLM 沟通技能的技术可以作为未来工作的下一步。另一个有趣的角度是研究如何调优模型以在沟通不足、有效沟通和过度沟通之间切换。我们设想，未来的不同 AI 编程代理将具备不同层次和风格的沟通能力。本研究可以视为评估代码 LLM 和 LLM 代理沟通技能的第一步。我们的基准测试和复制包已公开，网址为 [https://github.com/jie-jw-wu/human-eval-comm](https://github.com/jie-jw-wu/human-eval-comm)。

## 10. 致谢

我们感谢匿名评审人提供的宝贵反馈和建议，帮助显著提高了本工作的质量。我们还要感谢不列颠哥伦比亚大学的六位研究生，他们进行了手动评估。最后，我们要感谢 ChatGPT 非确定性研究的作者（Ouyang 等人，[2023](https://arxiv.org/html/2406.00215v2#bib.bib51)）和 IdentityChain（Min 等人，[2023](https://arxiv.org/html/2406.00215v2#bib.bib46)）的作者发布了他们的代码。本研究得到了加拿大自然科学与工程研究委员会 RGPIN-2019-05175 资助的支持。

## 附录

## 附录 A 附录：HumanEvalComm 中的提示模板

| 提示模板 | 描述 |
| --- | --- |
| 你是一个专家级软件开发者，编写高质量的代码。根据以下信息，请生成 Python3 代码（仅以代码形式直接回复，使用 Markdown），或提出澄清问题：{问题} | 模型的初始提示。用于评估。 |

| 编程问题的原始描述已修改，使得要求变得不一致、不完整或模糊不清。给定修改后的描述，提出了一些澄清问题以明确描述。根据原始和修改后的问题描述，评估澄清问题的质量。请提供一个整数，表示问题质量（3：好的问题，能够恢复修改后的要求；2：一般性问题，但无法帮助恢复修改后的要求；1：没有问题）。 QUALITY=[your int]

请同时提供回答这些澄清问题的答案，以恢复修改后的要求与原始问题描述之间的差异。如果没有澄清问题，请返回空答案。

ANSWERS=[your answer]

请严格遵循格式 QUALITY=[the int] 和 ANSWERS=[the answer] 在响应中！请用 Markdown 环绕你的答案！

### 问题：{clarifying_questions}

### 修改后的问题描述：{problem}

### 原始描述：{missing_information}  | 用于 LLM 评估器的提示。在评估中使用。 |

| {problem} {clarifying_questions}

{answer_to_questions}

根据以上对话，直接生成 Python 代码（Markdown）来解决编程问题： | 模型的第二轮提示。在评估中使用。 |

表 14. 在本研究中的 HumanEvalComm 评估中使用的详细提示模板。我们测试了不同的提示模板并检查了它们对结果的影响。

| 第一次提示变体 | 你是一个专门从事 Python3 的专家级软件开发人员。检查问题描述，并：- 如果描述提供了足够的细节，请继续编写所需的 Python3 代码，使用 Markdown 格式。

或者

- 如果需要更多细节来确保解决方案的准确性和完整性，请列出关于问题的任何具体问题。

### 问题描述：

{problem} |

| 第一次提示单轮 | 你是一个专门从事 Python3 的专家级软件开发人员。在每一轮中，检查问题描述，并：- 如果描述提供了准确的细节，请继续编写所需的 Python3 代码，使用 Markdown 格式。

或者

- 如果需要更多细节来确保解决方案的准确性和完整性，请列出澄清问题，而不是写代码。

### 问题描述：

```
def candidate(x: int, y: int):
    """
    Return 2^n modulo p (be aware of numerics).
    >>> candidate(3, 5)
    3
    >>> candidate(1101, 101)
    2
    >>> candidate(0, 101)
    1
    >>> candidate(3, 11)
    8
    >>> candidate(100, 101)
    1
    """

```

### 响应：

问题描述似乎不完整。它提供了一个函数签名和一些测试用例，但并没有说明这个函数应该做什么。文档字符串建议该函数应该返回 $2^{n}$ 模 $p$，但函数的参数（x, y）与文档字符串中的变量（n, p）不匹配。

这是我用来澄清问题的问题：

1\. 参数 x 和 y 在这个函数中的角色是什么？

2\. 函数是否应该返回 $2^{n}$ 模 $p$，正如文档字符串中所建议的那样？如果是，哪个参数对应于 n，哪个对应于 p？

3\. 文档字符串中提供的测试用例似乎与函数描述不符。请提供正确的测试用例。

### 问题描述：

{problem}

### 响应： |

表15。在 RQ4 的提示工程中使用的提示变体。

| 提示模板 | 描述 |
| --- | --- |

| 直接生成 Python 代码（Markdown）以解决编码问题。

{problem} | 提示生成 Okanagan 中的代码。 |

| 给定编程问题，如果给定问题描述中的需求不完整、不一致或模糊，导致无法正确解决问题并通过测试，请提问澄清问题。如果不需要提问澄清问题，只返回严格的’NO_QUESTIONS’。否则，返回澄清问题。

### 问题：

{problem} | 提示以在 Okanagan 中提问。 |

表16。作为本研究中提出的基于 LLM 的代理方法，Okanagan 中使用的详细提示模板。

## 参考文献

+   (1)

+   aut（2023）2023。AutoGPT。[https://github.com/Significant-Gravitas/Auto-GPT](https://github.com/Significant-Gravitas/Auto-GPT) AutoGPT v2.0。

+   big（2024）Hugging Face 访问于2024年。*大规模代码模型排行榜*。Hugging Face。[https://huggingface.co/spaces/bigcode/bigcode-models-leaderboard](https://huggingface.co/spaces/bigcode/bigcode-models-leaderboard) 访问于2024年4月29日。

+   Alon 等人（2019）U. Alon，M. Zilberstein，O. Levy 和 E. Yahav。2019。Code2Vec：学习代码的分布式表示。*ACM 编程语言会议录* 3，POPL（2019），1–29。

+   Arvidsson 和 Axell（2023）Simon Arvidsson 和 Johan Axell。2023。需求工程中 LLM 的提示工程指南。（2023）。

+   Austin 等人（2021）Jacob Austin，Augustus Odena，Maxwell Nye，Maarten Bosma，Henryk Michalewski，David Dohan，Ellen Jiang，Carrie Cai，Michael Terry，Quoc Le 等人。2021。使用大语言模型进行程序合成。*arXiv 预印本 arXiv:2108.07732*（2021）。

+   Bai 等人（2023）Jinze Bai，Shuai Bai，Yunfei Chu，Zeyu Cui，Kai Dang，Xiaodong Deng，Yang Fan，Wenbin Ge，Yu Han，Fei Huang 等人。2023。Qwen 技术报告。*arXiv 预印本 arXiv:2309.16609*（2023）。

+   Bai 等人（2022）Yuntao Bai, Saurav Kadavath, Sandipan Kundu, Amanda Askell, Jackson Kernion, Andy Jones, Anna Chen, Anna Goldie, Azalia Mirhoseini, Cameron McKinnon, Carol Chen, Catherine Olsson, Christopher Olah, Danny Hernandez, Dawn Drain, Deep Ganguli, Dustin Li, Eli Tran-Johnson, Ethan Perez, Jamie Kerr, Jared Mueller, Jeffrey Ladish, Joshua Landau, Kamal Ndousse, Kamile Lukosiute, Liane Lovitt, Michael Sellitto, Nelson Elhage, Nicholas Schiefer, Noemí Mercado, Nova DasSarma, Robert Lasenby, Robin Larson, Sam Ringer, Scott Johnston, Shauna Kravec, Sheer El Showk, Stanislav Fort, Tamera Lanham, Timothy Telleen-Lawton, Tom Conerly, Tom Henighan, Tristan Hume, Samuel R. Bowman, Zac Hatfield-Dodds, Ben Mann, Dario Amodei, Nicholas Joseph, Sam McCandlish, Tom Brown 和 Jared Kaplan. 2022. 宪法AI：来自AI反馈的无害性。*CoRR* abs/2212.08073（2022）。

+   Bi 等人（2024）Xiao Bi, Deli Chen, Guanting Chen, Shanhuang Chen, Damai Dai, Chengqi Deng, Honghui Ding, Kai Dong, Qiushi Du, Zhe Fu 等. 2024. Deepseek llm：通过长远主义扩展开源语言模型。*arXiv 预印本 arXiv:2401.02954*（2024）。

+   Borji（2023）Ali Borji. 2023. ChatGPT失败的分类档案。*arXiv 预印本 arXiv:2302.03494*（2023）。

+   Bouzenia 等人（2024）Islem Bouzenia, Premkumar Devanbu 和 Michael Pradel. 2024. RepairAgent：基于LLM的程序修复自主代理。*arXiv 预印本 arXiv:2403.17134*（2024）。

+   Brown 等人（2020）Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell 等. 2020. 语言模型是少量学习者。*神经信息处理系统进展* 33（2020），1877–1901。

+   Bui 等人（2021）N. D. Bui, Y. Yu 和 L. Jiang. 2021. InferCode：通过预测子树进行自监督学习的代码表示。在 *2021 IEEE/ACM 第43届国际软件工程大会（ICSE）*，1186–1197。

+   Chen 等人（2021）Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman 等. 2021. 评估用于代码的超大语言模型。*arXiv 预印本 arXiv:2107.03374*（2021）。

+   Dermeval 等人（2016）Diego Dermeval, Jéssyka Vilela, Ig Ibert Bittencourt, Jaelson Castro, Seiji Isotani, Patrick Brito 和 Alan Silva. 2016. 本体在需求工程中的应用：文献的系统性回顾。*需求工程* 21（2016），405–437。

+   Du 等人（2023）Yilun Du, Shuang Li, Antonio Torralba, Joshua B Tenenbaum 和 Igor Mordatch. 2023. 通过多代理辩论提高语言模型的事实性和推理能力。*arXiv 预印本 arXiv:2305.14325*（2023）。

+   Eghbali 和 Pradel（2024）Aryaz Eghbali 和 Michael Pradel. 2024. De-Hallucinator：基于LLM的代码补全的迭代基础。*arXiv 预印本 arXiv:2401.01701*（2024）。

+   Face（2023）Hugging Face. 2023. *Codeparrot*. [https://huggingface.co/codeparrot/codeparrot](https://huggingface.co/codeparrot/codeparrot)

+   Fakhoury 等人（2024）Sarah Fakhoury, Aaditya Naik, Georgios Sakkas, Saikat Chakraborty 和 Shuvendu K Lahiri. 2024. 《基于 LLM 的测试驱动互动代码生成：用户研究与实证评估》。*arXiv 预印本 arXiv:2404.10100*（2024）。

+   Fan 等人（2023）Angela Fan, Beliz Gokkaya, Mark Harman, Mitya Lyubarskiy, Shubho Sengupta, Shin Yoo 和 Jie M Zhang. 2023. 《大语言模型在软件工程中的应用：调查与开放问题》。*arXiv 预印本 arXiv:2310.03533*（2023）。

+   Feng 等人（2020）Z. Feng, D. Guo, D. Tang, N. Duan, X. Feng, M. Gong, L. Shou, B. Qin, T. Liu 和 D. Jiang 等人. 2020. 《CodeBERT：一种用于编程和自然语言的预训练模型》。*arXiv 预印本 arXiv:2002.08155*（2020）。

+   Fernandes 等人（2023）Patrick Fernandes, Aman Madaan, Emmy Liu, António Farinhas, Pedro Henrique Martins, Amanda Bertsch, José G. C. de Souza, Shuyan Zhou, Tongshuang Wu, Graham Neubig 和 André F. T. Martins. 2023. 《弥合鸿沟：关于将（人类）反馈集成到自然语言生成中的调查》。*CoRR* abs/2305.00955（2023）。

+   Gao 等人（2024）Mingqi Gao, Xinyu Hu, Jie Ruan, Xiao Pu 和 Xiaojun Wan. 2024. 《基于 LLM 的 NLG 评估：当前状态与挑战》。*arXiv 预印本 arXiv:2402.01383*（2024）。

+   Glaese 等人（2022）Amelia Glaese, Nat McAleese, Maja Trebacz, John Aslanides, Vlad Firoiu, Timo Ewalds, Maribeth Rauh, Laura Weidinger, Martin J. Chadwick, Phoebe Thacker, Lucy Campbell-Gillingham, Jonathan Uesato, Po-Sen Huang, Ramona Comanescu, Fan Yang, Abigail See, Sumanth Dathathri, Rory Greig, Charlie Chen, Doug Fritz, Jaume Sanchez Elias, Richard Green, Sona Mokrá, Nicholas Fernando, Boxi Wu, Rachel Foley, Susannah Young, Iason Gabriel, William Isaac, John Mellor, Demis Hassabis, Koray Kavukcuoglu, Lisa Anne Hendricks 和 Geoffrey Irving. 2022. 《通过有针对性的人类判断改善对话代理的对齐》。*CoRR* abs/2209.14375（2022）。

+   Guo 等人（2024b）Daya Guo, Qihao Zhu, Dejian Yang, Zhenda Xie, Kai Dong, Wentao Zhang, Guanting Chen, Xiao Bi, Y Wu, YK Li 等人. 2024b. 《DeepSeek-Coder：当大语言模型遇上编程——代码智能的崛起》。*arXiv 预印本 arXiv:2401.14196*（2024）。

+   Guo 等人（2024a）Taicheng Guo, Xiuying Chen, Yaqi Wang, Ruidi Chang, Shichao Pei, Nitesh V Chawla, Olaf Wiest 和 Xiangliang Zhang. 2024a. 《基于大语言模型的多代理：进展与挑战的调查》。*arXiv 预印本 arXiv:2402.01680*（2024）。

+   Hassan 等人（2024）Ahmed E Hassan, Gustavo A Oliva, Dayi Lin, Boyuan Chen, Zhen Ming 等人. 2024. 《在基础模型时代重新思考软件工程：从任务驱动的 AI 副驾驶到目标驱动的 AI 配对程序员》。*arXiv 预印本 arXiv:2404.10225*（2024）。

+   He等人（2023）何航峰、张洪名和丹·罗斯。2023年。《通过检索重新思考：忠实的大型语言模型推理》。*CoRR* abs/2301.00303（2023）。

+   Hendrycks等人（2021）丹·亨德里克斯、史蒂文·巴萨特、索拉夫·卡达瓦特、曼塔斯·马泽伊卡、阿库尔·阿罗拉、伊桑·郭、科林·伯恩斯、萨米尔·普拉尼克、霍勒斯·何、道恩·宋等人。2021年。《通过应用程序衡量编码挑战能力》。*arXiv预印本arXiv:2105.09938*（2021）。

+   Huang等人（2023）黄东、卜庆文、张杰明、迈克尔· Luck 和崔海明。2023年。《Agentcoder：基于多智能体的代码生成与迭代测试与优化》。*arXiv预印本arXiv:2312.13010*（2023）。

+   Huang等人（2022）黄嘉欣、顾时祥、侯乐、吴悦欣、王雪智、余洪坤和韩家伟。2022年。《大型语言模型可以自我改进》。*CoRR* abs/2210.11610（2022）。

+   Jazayeri（2004）梅赫迪·贾扎耶里。2004年。《软件工程师的教育》。收录于 *Proceedings. 19th International Conference on Automated Software Engineering, 2004.* IEEE，第xviii-xxvii页。

+   Jiang等人（2023）蒋舒扬、王宇豪和王宇。2023年。《SelfEvolve：一个通过大语言模型实现的代码演化框架》。*CoRR* abs/2306.02907（2023）。

+   Jimenez等人（2024）卡洛斯·E·吉门内斯、杨约翰、亚历山大·维蒂希、姚顺宇、裴克欣、奥菲尔·普雷斯和卡尔蒂克·R·纳拉西姆汉。2024年。《SWE-bench：语言模型能解决真实世界的Github问题吗？》。收录于 *The Twelfth International Conference on Learning Representations*。[https://openreview.net/forum?id=VTF8yNQM66](https://openreview.net/forum?id=VTF8yNQM66)

+   Kim等人（2023）金重勋、朴世然、郑基云、李尚敏、韩胜勋、李智允和姜必生。2023年。《哪个更好？探索基于大语言模型的提示策略评估》。*arXiv预印本arXiv:2311.03754*（2023）。

+   Kocmi和Federmann（2023）汤姆·科茨米和克里斯蒂安·费德曼。2023年。《大型语言模型是评估翻译质量的最先进工具》。*arXiv预印本arXiv:2302.14520*（2023）。

+   Kotonya等人（2023）尼玛·科托尼亚、萨兰·克里希纳萨米、乔尔·泰特罗和亚历山大·海梅斯。2023年。《小巨人：探索小型大语言模型作为摘要评估指标的潜力——在eval4nlp 2023共享任务中的应用》。*arXiv预印本arXiv:2311.00686*（2023）。

+   Kreutzer等人（2018）朱莉娅·克罗伊策、沙赫拉姆·哈迪维、叶夫根尼·马图索夫和斯特凡·里兹勒。2018年。《用户反馈能否改进神经机器翻译？》。收录于 *Proceedings of the 2018 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies (NAACL-HLT)*。

+   Li等人（2024）李浩、科尔-保罗·贝泽默和艾哈迈德·E·哈桑。2024年。《软件工程与基础模型：基于行业博客的基础模型评审洞察》。*arXiv预印本arXiv:2410.09012*（2024）。

+   Li 等人 (2022) Yujia Li, David Choi, Junyoung Chung, Nate Kushman, Julian Schrittwieser, Rémi Leblond, Tom Eccles, James Keeling, Felix Gimeno, Agustin Dal Lago, 等人. 2022. 使用 Alphacode 进行竞赛级别的代码生成。*Science* 378, 6624 (2022), 1092–1097。

+   Liang 等人 (2023) Jenny T Liang, Chenyang Yang, 和 Brad A Myers. 2023. 理解 AI 编程助手的可用性。*arXiv 预印本 arXiv:2303.17125* (2023)。

+   Liu 等人 (2023a) Yongkang Liu, Shi Feng, Daling Wang, Yifei Zhang, 和 Hinrich Schütze. 2023a. 评估你无法评估的东西：无法评估的生成响应质量。*arXiv 预印本 arXiv:2305.14658* (2023)。

+   Liu 等人 (2023b) Yue Liu, Thanh Le-Cong, Ratnadira Widyasari, Chakkrit Tantithamthavorn, Li Li, Xuan-Bach D Le, 和 David Lo. 2023b. 提炼 ChatGPT 生成的代码：表征和缓解代码质量问题。*arXiv 预印本 arXiv:2307.12596* (2023)。

+   Liu 等人 (2023c) Zhijie Liu, Yutian Tang, Xiapu Luo, Yuming Zhou, 和 Liang Feng Zhang. 2023c. 不再需要动一根手指？评估 ChatGPT 生成代码的质量。*arXiv 预印本 arXiv:2308.04838* (2023)。

+   McChesney 和 Gallagher (2004) Ian R McChesney 和 Seamus Gallagher. 2004. 软件工程项目中的沟通与协调实践。*信息与软件技术* 46, 7 (2004), 473–489。

+   Min 等人 (2023) Marcus J Min, Yangruibo Ding, Luca Buratti, Saurabh Pujar, Gail Kaiser, Suman Jana, 和 Baishakhi Ray. 2023. 超越准确性：使用 identitychain 评估大规模代码语言模型的自一致性。*arXiv 预印本 arXiv:2310.14053* (2023)。

+   Mistrík 等人 (2010) Ivan Mistrík, John Grundy, Andre Van der Hoek, 和 Jim Whitehead. 2010. *协同软件工程：挑战与前景*。Springer。

+   Nguyen 和 Nadi (2022) N. Nguyen 和 S. Nadi. 2022. GitHub Copilot 代码建议的实证评估。在 *第19届国际软件仓库挖掘会议论文集* 中，1–5 页。

+   Nijkamp 等人 (2022) Erik Nijkamp, Bo Pang, Hiroaki Hayashi, Lifu Tu, Huan Wang, Yingbo Zhou, Silvio Savarese, 和 Caiming Xiong. 2022. Codegen：用于代码的开放大型语言模型，支持多轮程序合成。*arXiv 预印本 arXiv:2203.13474* (2022)。

+   Ouyang 等人 (2022) Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll L. Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray, John Schulman, Jacob Hilton, Fraser Kelton, Luke Miller, Maddie Simens, Amanda Askell, Peter Welinder, Paul F. Christiano, Jan Leike, 和 Ryan Lowe. 2022. 使用人类反馈训练语言模型遵循指令。在 *神经信息处理系统年会（NeurIPS）论文集* 中。

+   Ouyang 等人 (2023) Shuyin Ouyang, Jie M Zhang, Mark Harman, 和 Meng Wang. 2023. LLM 就像一盒巧克力：ChatGPT 在代码生成中的非确定性。*arXiv 预印本 arXiv:2308.02828* (2023)。

+   Palacio et al. (2024) David N Palacio, Alejandro Velasco, Nathan Cooper, Alvaro Rodriguez, Kevin Moran, 和 Denys Poshyvanyk. 2024. 面向神经代码模型解释的因果理论。*IEEE软件工程学报* (2024).

+   Pan et al. (2023) Liangming Pan, Michael Saxon, Wenda Xu, Deepak Nathani, Xinyi Wang, 和 William Yang Wang. 2023. 自动修正大型语言模型：调查多样化自我修正策略的全景。*arXiv预印本 arXiv:2308.03188* (2023).

+   Pressman (2005) Roger S Pressman. 2005. *软件工程：实践者的方式*. Palgrave macmillan.

+   Rabinovich et al. (2017) M. Rabinovich, M. Stern, 和 D. Klein. 2017. 用于代码生成和语义解析的抽象语法网络。*arXiv预印本 arXiv:1704.07535* (2017).

+   Rasheed et al. (2024) Zeeshan Rasheed, Muhammad Waseem, Mika Saari, Kari Systä, 和 Pekka Abrahamsson. 2024. Codepori：通过使用多智能体的大规模模型进行自主软件开发。*arXiv预印本 arXiv:2402.01411* (2024).

+   Rodriguez-Cardenas et al. (2023) Daniel Rodriguez-Cardenas, David N Palacio, Dipin Khati, Henry Burke, 和 Denys Poshyvanyk. 2023. 基准测试因果研究以解释源代码中的大型语言模型。发表于*2023 IEEE国际软件维护与演化会议 (ICSME)*. IEEE, 329–334.

+   Roziere et al. (2023) Baptiste Roziere, Jonas Gehring, Fabian Gloeckle, Sten Sootla, Itai Gat, Xiaoqing Ellen Tan, Yossi Adi, Jingyu Liu, Tal Remez, Jérémy Rapin, 等人. 2023. Code llama：面向代码的开放基础模型。*arXiv预印本 arXiv:2308.12950* (2023).

+   Sarkar et al. (2022) Advait Sarkar, Andrew D Gordon, Carina Negreanu, Christian Poelitz, Sruti Srinivasa Ragavan, 和 Ben Zorn. 2022. 与人工智能一起编程是什么样的体验？*arXiv预印本 arXiv:2208.06213* (2022).

+   Scheurer et al. (2023) Jérémy Scheurer, Jon Ander Campos, Tomasz Korbak, Jun Shern Chan, Angelica Chen, Kyunghyun Cho, 和 Ethan Perez. 2023. 利用语言反馈进行大规模语言模型训练。*CoRR* abs/2303.16755 (2023).

+   Schick et al. (2023) Timo Schick, Jane Dwivedi-Yu, Patrick Schramowski, 和 Jacob Andreas. 2023. Toolformer：语言模型可以自学使用工具。*arXiv预印本 arXiv:2302.04761* (2023).

+   Shanahan (2024) Murray Shanahan. 2024. 讨论大型语言模型。*Commun. ACM* 67, 2 (2024), 68–79.

+   Shinn et al. (2023) Noah Shinn, Nelson Labash, Mark Skreta, Shunyu Yao, 和 Karthik Narasimhan. 2023. Reflexion：带有语言强化学习的语言代理。*arXiv预印本 arXiv:2303.11366* (2023).

+   Siddiq et al. (2022) M. L. Siddiq, S. H. Majumder, M. R. Mim, S. Jajodia, 和 J. C. Santos. 2022. 基于变压器的代码生成技术中的代码异味实证研究。发表于*2022 IEEE第22届国际源代码分析与操作会议 (SCAM)*. 71–82.

+   Sim et al. (2003) Susan Elliott Sim, Steve Easterbrook, 和 Richard C Holt. 2003. 使用基准测试推动研究：对软件工程的挑战。见于 *第25届国际软件工程大会，2003年\. 会议录*。IEEE，74–83。

+   Sobania et al. (2022) D. Sobania, M. Briesch, 和 F. Rothlauf. 2022. 选择你的编程副驾驶：GitHub Copilot与遗传编程的程序合成性能比较。见于 *遗传与进化计算大会论文集*。1019–1027。

+   Svyatkovskiy et al. (2020) A. Svyatkovskiy, S. K. Deng, S. Fu, 和 N. Sundaresan. 2020. Intellicode Compose：使用变换器的代码生成。见于 *第28届ACM欧洲软件工程会议与软件工程基础学会研讨会联合会议录*。1433–1443。

+   Tufano et al. (2020) M. Tufano, D. Drain, A. Svyatkovskiy, S. Deng, 和 N. Sundaresan. 2020. 使用变换器和焦点上下文的单元测试用例生成。*arXiv 预印本 arXiv: 软件工程* (2020)。

+   Tukur et al. (2021) Muhammad Tukur, Sani Umar, 和 Jameleddine Hassine. 2021. 需求工程挑战：基于学术和工业视角的系统化映射研究。*阿拉伯科学与工程杂志* 46 (2021)，3723–3748。

+   Vaithilingam et al. (2022) P. Vaithilingam, T. Zhang, 和 E. L. Glassman. 2022. 期望与经验：评估由大型语言模型驱动的代码生成工具的可用性。见于 *CHI 计算机系统中的人因会议扩展摘要*。1–7。

+   Vaswani et al. (2017) A. Vaswani, N. Shazeer, N. Parmar, J. Uszkoreit, L. Jones, A. N. Gomez, Ł. Kaiser, 和 I. Polosukhin. 2017. 注意力机制：你所需要的一切。见于 *神经信息处理系统进展*，第30卷。

+   Wang 和 Komatsuzaki (2021) Ben Wang 和 Aran Komatsuzaki. 2021. GPT-J-6B：一个具有60亿参数的自回归语言模型。

+   Wang et al. (2023a) Jiaan Wang, Yunlong Liang, Fandong Meng, Zengkui Sun, Haoxiang Shi, Zhixu Li, Jinan Xu, Jianfeng Qu, 和 Jie Zhou. 2023a. ChatGPT 是一个好的 NLG 评估器吗？初步研究。*arXiv 预印本 arXiv:2303.04048* (2023)。

+   Wang et al. (2023b) Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin, 等. 2023b. 基于大型语言模型的自主代理调查。*arXiv 预印本 arXiv:2308.11432* (2023)。

+   Wang et al. (2021) Y. Wang, W. Wang, S. Joty, 和 S. C. Hoi. 2021. CodeT5：识别符感知统一预训练编码器-解码器模型用于代码理解与生成。*arXiv 预印本 arXiv:2109.00859* (2021)。

+   Weng et al. (2023) Yixuan Weng, Minjun Zhu, Fei Xia, Bin Li, Shizhu He, Kang Liu, 和 Jun Zhao. 2023. 大型语言模型通过自我验证成为更好的推理者。*CoRR* abs/2212.09561 (2023)。

+   Whitehead (2007) Jim Whitehead. 2007. 软件工程中的协作：一条路线图。见于 *软件工程的未来 (FOSE’07)*。IEEE，214–225。

+   Wu (2024) Scott Wu. 访问于2024年. *介绍Devin，首个AI软件工程师*. Cognition. [https://www.cognition-labs.com/introducing-devin](https://www.cognition-labs.com/introducing-devin) 访问时间：2024年4月8日。

+   Xi et al. (2023) Zhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, Boyang Hong, Ming Zhang, Junzhe Wang, Senjie Jin, Enyu Zhou, 等. 2023. 基于大语言模型的代理的崛起与潜力：一项调查. *arXiv预印本 arXiv:2309.07864*（2023）。

+   Xie et al. (2023) Yuxi Xie, Kenji Kawaguchi, Yiran Zhao, Xu Zhao, MinYen Kan, Junxian He, 和 Qizhe Xie. 2023. 通过自我评估引导解码增强推理能力. *CoRR* abs/2305.00633（2023）。

+   Xu et al. (2022) Frank F Xu, Uri Alon, Graham Neubig, 和 Vincent Josua Hellendoorn. 2022. 对大规模代码语言模型的系统评估. 载于 *第六届 ACM SIGPLAN 国际机器编程研讨会论文集*，1–10。

+   Xu et al. (2024) Shuyuan Xu, Zelong Li, Kai Mei, 和 Yongfeng Zhang. 2024. CoRE：LLM作为自然语言编程、伪代码编程和AI代理流程编程的解释器. *arXiv预印本 arXiv:2405.06907*（2024）。

+   Yang et al. (2024) John Yang, Carlos E Jimenez, Alexander Wettig, Kilian Lieret, Shunyu Yao, Karthik Narasimhan, 和 Ofir Press. 2024. Swe-agent：代理-计算机接口实现自动化软件工程. *arXiv预印本 arXiv:2405.15793*（2024）。

+   Yang et al. (2022) Kaiyu Yang, Jia Deng, 和 Danqi Chen. 2022. 使用验证器引导搜索生成自然语言证明. 载于 *2022年自然语言处理实证方法会议（EMNLP）论文集*，89–105。

+   Yao et al. (2023) Shunyu Yao, Jeffrey Zhao, Dian Yu, Shixiang Shane Cao, Karthik Narasimhan, 和 Wen-tau Huang. 2023. ReAct：语言模型中推理与行动的协同作用. *arXiv预印本 arXiv:2210.03629*（2023）。

+   Ye et al. (2020) W. Ye, R. Xie, J. Zhang, T. Hu, X. Wang, 和 S. Zhang. 2020. 利用代码生成提升代码检索与总结的双向学习. 载于 *2020年Web大会论文集*，2309–2319。

+   Yin et al. (2018) Pengcheng Yin, Bowen Deng, Edgar Chen, Bogdan Vasilescu, 和 Graham Neubig. 2018. 从 Stack Overflow 中学习挖掘对齐的代码与自然语言对. 载于 *第十五届国际软件库挖掘会议论文集*，476–486。

+   Zan et al. (2023) Daoguang Zan, Bei Chen, Fengji Zhang, Dianjie Lu, Bingchao Wu, Bei Guan, Wang Yongji, 和 Jian-Guang Lou. 2023. 大语言模型与nl2code的结合：一项调查. 载于 *第61届计算语言学协会年会论文集（第1卷：长篇论文）*，7443–7464。

+   Zhang et al. (2023c) Jintian Zhang, Xin Xu, 和 Shumin Deng. 2023c. 探索大语言模型代理的协作机制：社会心理学视角. *arXiv预印本 arXiv:2310.02124*（2023）。

+   Zhang 等人（2024）Kechi Zhang、Jia Li、Ge Li、Xianjie Shi 和 Zhi Jin。2024年。Codeagent：通过工具集成代理系统增强代码生成，解决实际代码库级别的编程挑战。*arXiv 预印本 arXiv:2401.07339*（2024）。

+   Zhang 等人（2023b）Kechi Zhang、Zhuo Li、Jia Li、Ge Li 和 Zhi Jin。2023b。Self-Edit：面向代码生成的故障感知代码编辑器。*CoRR* abs/2305.04087（2023）。

+   Zhang 等人（2023a）Ziyin Zhang、Chaoyu Chen、Bingchang Liu、Cong Liao、Zi Gong、Hang Yu、Jianguo Li 和 Rui Wang。2023a。统一自然语言处理与软件工程的视角：关于代码语言模型的调研。*arXiv 预印本 arXiv:2311.07989*（2023）。

+   Ziegler 等人（2022）Albert Ziegler、Eirini Kalliamvakou、X Alice Li、Andrew Rice、Devon Rifkin、Shawn Simister、Ganesh Sittampalam 和 Edward Aftandilian。2022年。神经代码补全的生产力评估。载于 *第六届 ACM SIGPLAN 国际机器编程研讨会论文集*，21–29页。
