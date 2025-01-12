<!--yml

类别：未分类

日期：2025-01-11 12:50:53

-->

# LLM中间件：工具在复杂环境中对语言代理的重要性

> 来源：[https://arxiv.org/html/2402.14672/](https://arxiv.org/html/2402.14672/)

Yu Gu¹, Yiheng Shu¹, Hao Yu², Xiao Liu², Yuxiao Dong², Jie Tang²,

Jayanth Srinivasa³, Hugo Latapie³, Yu Su¹

¹俄亥俄州立大学  ²清华大学  ³思科研究

{gu.826, su.809}@osu.edu

###### 摘要

大型语言模型（LLMs）的应用已远超文本处理的范畴，标志着一个新纪元的到来，在这个时代，LLMs被设想为能够在复杂环境中操作的通用代理。这些环境通常是高度扩展的，使得LLM无法在其短期记忆中处理所有信息。受到最近关于使用工具扩展LLM能力的研究启发，我们旨在探讨工具在帮助LLM处理这种复杂性方面的潜力，并引入一种新的工具类别——中间件，以帮助LLM在这些庞大环境中进行主动探索。这些专用工具可以作为中间件层，帮助LLM屏蔽环境的复杂性。在两个代表性的复杂环境——知识库（KB）和数据库中——我们展示了使用工具增强语言代理在复杂环境中的巨大潜力。值得注意的是，配备中间件后，GPT-4在需要访问数据库内容的任务中达到了最佳基准的2.8倍，在知识库任务中达到了2.2倍的性能提升。我们的发现为推进语言代理在现实应用中的发展指明了方向。¹¹1GitHub 仓库：[OSU_NLP/Middleware](https://github.com/OSU-NLP-Group/Middleware)

Hugging Face 数据集：[KBQA-Agent](https://huggingface.co/datasets/osunlp/KBQA-Agent)

## 1 引言

大型语言模型（LLMs）已展示出类人般的文本处理能力 OpenAI（[2023a](https://arxiv.org/html/2402.14672v2#bib.bib26)，[b](https://arxiv.org/html/2402.14672v2#bib.bib27)）；Touvron等（[2023](https://arxiv.org/html/2402.14672v2#bib.bib41)）；Jiang等（[2024](https://arxiv.org/html/2402.14672v2#bib.bib17)）。然而，人工智能的真正目标远超文本处理领域。最终目标是赋能LLM，成为通用语言代理，帮助人类完成各种复杂的现实任务 Yao等（[2022](https://arxiv.org/html/2402.14672v2#bib.bib44)）；Schick等（[2023](https://arxiv.org/html/2402.14672v2#bib.bib33)）；Liu等（[2023](https://arxiv.org/html/2402.14672v2#bib.bib22)），这些任务通常涉及处理复杂的环境，无论是浏览复杂网页 Deng等（[2023](https://arxiv.org/html/2402.14672v2#bib.bib6)），管理包含数百万条记录的庞大数据库 Li等（[2023a](https://arxiv.org/html/2402.14672v2#bib.bib19)），还是查询庞大的知识库（KB） Gu等（[2023](https://arxiv.org/html/2402.14672v2#bib.bib9)）。

![参见说明文字](img/dc242314fd333e01020a508e903bebc7.png)

图 1：（左）当一个 LLM 与复杂环境交互时，它可以通过将环境的描述（即线性化的标记）适配到其短期记忆中（即 LLM 的输入窗口），从而发展出对环境的理解。然而，随着环境复杂度的增加，这种方法会遇到严重的可扩展性问题。（右）另一种选择是为 LLM 提供一组工具，帮助它积极地与环境互动并获取必要的信息。

为了使 LLM 有效地作为代理，准确地将人类指令与环境进行匹配，它们必须建立对环境的深刻理解。实现这一目标的最直接方法是将环境线性化为一系列适配 LLM 短期记忆（即输入窗口）的标记，并让 LLM 根据线性化描述处理环境 Tai 等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib39)）；Shridhar 等人（[2021](https://arxiv.org/html/2402.14672v2#bib.bib34)）；Liu 等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib22)）。然而，这种方法在扩展到更复杂的环境时面临巨大挑战。此外，离散的标记描述可能无法反映出对环境的最自然感知。近期的研究探索了使用工具来扩展 LLM 能力的边界 Li 等人（[2023b](https://arxiv.org/html/2402.14672v2#bib.bib20)）；Qin 等人（[2023b](https://arxiv.org/html/2402.14672v2#bib.bib30)）；Schick 等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib33)）。其核心理念是，LLM 可以主动选择合适的工具进行使用，将语言作为强大的思维工具 Su（[2023](https://arxiv.org/html/2402.14672v2#bib.bib36)）。直观地讲，我们也可以为 LLM 配备能够导航复杂环境的工具，使得 LLM 可以主动调用不同的工具来探索环境，从而绕过其短期记忆的限制（图 [1](https://arxiv.org/html/2402.14672v2#S1.F1 "图 1 ‣ 1 引言 ‣ LLM 中间件：工具对语言代理在复杂环境中的作用")））。然而，这一有前景的范式迄今为止尚未得到充分探索。在本文中，我们旨在深入探讨这一范式，并回答一个引人入胜的问题：在工具的帮助下，LLM 能在多大程度上有效处理复杂环境？

回答这个问题需要为大语言模型配备一套旨在满足目标环境中广泛需求的工具。在本文中，我们为两个典型的复杂环境——即数据库和知识库（KB）——精心开发了这些定制工具。与先前研究中秦等人（[2023b](https://arxiv.org/html/2402.14672v2#bib.bib30)）使用的现成Web API不同，我们的工具必须从头手动发明。在设计这些工具时，我们利用了人类信息收集行为的直觉——例如，执行关键词搜索以识别相关的数据库列或调查知识库实体之间的联系——以在这些环境中完成复杂任务（见[3.1节](https://arxiv.org/html/2402.14672v2#S3.SS1 "3.1 Tools for Complex Environments ‣ 3 Middleware for LLMs ‣ Middleware for LLMs: Tools Are Instrumental for Language Agents in Complex Environments")）。理想情况下，这些工具旨在作为大语言模型和环境之间的中间件层，保护大语言模型免受环境复杂性的影响。通过这些专门的工具，我们提出了两种新方案，以便大语言模型能够更准确地协调其内部推理和工具使用：错误反馈和解耦生成（见[3.2节](https://arxiv.org/html/2402.14672v2#S3.SS2 "3.2 Reasoning with Tools ‣ 3 Middleware for LLMs ‣ Middleware for LLMs: Tools Are Instrumental for Language Agents in Complex Environments")）。定制工具和工具使用方案的结合使得大语言模型能够积极探索环境，并将人类指令转化为准确的行动。

我们在针对目标环境的复杂任务基准上评估了不同的大语言模型，包括一个新近整理的知识库（KB）基准。结果非常具有启示性：配备定制工具的大语言模型在应对复杂环境方面展现出了显著的提升，远远超过了现有的技术水平。特别是，尽管其设计简单，这种中间件层使得GPT-4 OpenAI（[2023a](https://arxiv.org/html/2402.14672v2#bib.bib26)）在需要访问数据库内容的任务中，达到了最佳基准的2.8倍性能（即38.3%对13.8%），在知识库任务中则提高了2.2倍（即59.3%对27.1%）。我们的研究结果强调了工具增强在使大语言模型能够应对复杂环境中的关键作用。

我们的主要贡献如下：a) 我们为两个复杂环境开发了一个新框架，并定制了工具，以研究工具在处理复杂环境中的作用，尤其是在大语言模型（LLMs）中的应用；b) 我们在精心选择的基准上评估了六种不同的大语言模型，以进行全面分析；c) 我们的分析突出了一个关键的结论：通过工具增强大语言模型对于成功应对复杂环境至关重要，这为将大语言模型发展为通用语言代理并用于实际应用开辟了新的可能性。

## 2 相关工作

+   使用LLM与复杂环境接口。

    直接将环境输入LLM进行基础化的现有方法（Chandu等人，[2021](https://arxiv.org/html/2402.14672v2#bib.bib4)）由于可扩展性问题，在复杂环境中会失败。具体来说，这些方法通过将环境线性化为离散标记进行处理（Hwang等人，[2019](https://arxiv.org/html/2402.14672v2#bib.bib15)）；Shridhar等人，[2021](https://arxiv.org/html/2402.14672v2#bib.bib34)；Yu等人，[2023](https://arxiv.org/html/2402.14672v2#bib.bib46)；Liu等人，[2023](https://arxiv.org/html/2402.14672v2#bib.bib22)；Tai等人，[2023](https://arxiv.org/html/2402.14672v2#bib.bib39)；Song等人，[2023](https://arxiv.org/html/2402.14672v2#bib.bib35)。然而，将具有数百万条记录的庞大环境如数据库（Li等人，[2023a](https://arxiv.org/html/2402.14672v2#bib.bib19)）或冗长的网页HTML代码（Deng等人，[2023](https://arxiv.org/html/2402.14672v2#bib.bib6)）线性化，往往会超过LLM的输入长度限制。替代的研究通过生成未经基础化的草拟计划以供后处理基础化（Li等人，[2023c](https://arxiv.org/html/2402.14672v2#bib.bib21)）；Nie等人，[2023](https://arxiv.org/html/2402.14672v2#bib.bib25)），或通过使用LLM评估通过预定义规则创建的基础化计划（Gu等人，[2023](https://arxiv.org/html/2402.14672v2#bib.bib9)）来绕过LLM与复杂环境的直接互动。这些策略未能充分利用LLM在主动探索复杂环境中的固有推理潜力。在本文中，我们探索了一种新范式，通过为LLM配备一套全面的工具，在需求时主动收集有关环境的必要信息，从而绕过这些问题，并充分发挥LLM的固有推理能力。

+   工具学习。

    工具对于增强LLMs的能力至关重要，Schick等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib33)）；Qin等人（[2023a](https://arxiv.org/html/2402.14672v2#bib.bib29)）；Mialon等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib24)）；Hao等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib14)）。现有的研究，如ToolLLM Qin等人（[2023b](https://arxiv.org/html/2402.14672v2#bib.bib30)）和API-Bank Li等人（[2023b](https://arxiv.org/html/2402.14672v2#bib.bib20)），主要关注开放域应用，利用一系列现成的RESTful API。相比之下，本文特别旨在研究工具在增强LLMs方面的潜力，以有效地在复杂环境中执行任务，其中我们亲自为不同环境精心设计专用工具。此外，关注RESTful API的研究通常表现出较浅的推理能力，而复杂环境中的实际任务通常涉及一长串的操作（例如，查询知识库或浏览网页）。为了在更具体的复杂环境中实现工具的使用，StructGPT Jiang等人（[2023b](https://arxiv.org/html/2402.14672v2#bib.bib18)）采用了预定义的工具调用序列；Chameleon Lu等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib23)）则在一个开放回路环境中运行，其中LLM在任何执行发生之前直接生成工具使用的序列。它们都未能将LLM的推理能力与工具的使用无缝整合。本文提出了两种新的方案——错误反馈和解耦生成，以更加无缝和准确地协调LLM的内部推理和工具的使用。

## 3 LLMs的中间件

我们为 LLM 配备了一套专门的工具，旨在支持多种操作，并满足复杂环境 $\mathcal{E}$ 中的各种需求。我们将这些工具称为中间件，因为它们可以充当 LLM 和 $\mathcal{E}$ 之间功能丰富的中间层，帮助 LLM 避免直接与环境的复杂性互动（第 [3.1](https://arxiv.org/html/2402.14672v2#S3.SS1 "3.1 Tools for Complex Environments ‣ 3 Middleware for LLMs ‣ Middleware for LLMs: Tools Are Instrumental for Language Agents in Complex Environments") 节）。此外，为了充分释放 LLM 在调用适当工具时的内在推理能力，我们提出了两种新颖的方案来提高工具使用的准确性：错误反馈，它提供具体的工具使用错误信息，并期望 LLM 自主修正错误；以及解耦生成，其中 LLM 的推理步骤和工具使用被分开，从而更好地控制（第 [3.2](https://arxiv.org/html/2402.14672v2#S3.SS2 "3.2 Reasoning with Tools ‣ 3 Middleware for LLMs ‣ Middleware for LLMs: Tools Are Instrumental for Language Agents in Complex Environments") 节）。这一统一框架使我们能够可靠地研究 LLM 在借助工具的帮助下处理复杂环境的潜力。

![参考说明](img/60019ea2b3b35914f9481ecb65f5d187.png)

图 2：LLM 配备了一系列工具，以便更好地与复杂环境（例如这里的知识库）互动。（a）LLM 可能会产生无效的操作（以粉红色标记）。通过向其提供错误信息来提示并鼓励其重新尝试（修正后的操作以绿色标记），可以减轻这一问题。（b）另一种方式是让 LLM 首先生成一个思考，然后根据该思考在单独的上下文中预测一个操作（以蓝色标记），最后将该操作插入回原始上下文。黄色标记的文本为环境输入。

### 3.1 复杂环境中的工具

为了评估 LLM 在配备工具后处理复杂环境的潜力，我们需要首先精心设计适合环境的必要工具。这些工具应满足两个基本标准：1）它们应具备全面性，涵盖广泛的操作和需求。工具的广泛覆盖对于最大化 LLM 在规划中的潜力至关重要。2）这些工具应优先考虑易用性，使 LLM 主要通过简单的插槽填写来调用它们，从而将 LLM 与工具的实现细节隔离开。

+   数据库

    在生产场景中，数据库通常包含数十个表，每个表包含成千上万的行或更多。在这种环境中，一个关键任务是通过 SQL 查询进行数据分析。为了弥补自然语言指令与 SQL 之间的差距，使用大语言模型（LLMs）来自动生成 SQL 查询（即文本到 SQL 解析 Yu 等人 ([2018](https://arxiv.org/html/2402.14672v2#bib.bib47))；Li 等人 ([2023a](https://arxiv.org/html/2402.14672v2#bib.bib19)))。为了支持 LLM 构建复杂的 SQL 查询，我们引入了一组专门的工具，用于与复杂数据库进行交互。这些工具分为两大类：导航工具和功能工具。导航工具帮助 LLM 探索环境（例如，get_distinct_values() 和 find_columns_containing_value()），而功能工具则帮助检查 LLM 构建的每个 SQL 子句。例如，where() 验证 WHERE 子句的合法性，并确定指定的条件是否能够匹配数据库中的任何条目。总的来说，我们为数据库设计了 $12$ 个工具（附录 [A.1](https://arxiv.org/html/2402.14672v2#A1.SS1 "A.1 Databases ‣ Appendix A Detailed Tool Definitions ‣ Middleware for LLMs: Tools Are Instrumental for Language Agents in Complex Environments")）。这些工具的开发基于我们在 SQL 和数据库领域的专业知识。

+   KBs

    现代知识库（KB），例如 Freebase Bollacker 等人 ([2008](https://arxiv.org/html/2402.14672v2#bib.bib2))，是存储数十亿事实的庞大仓库，这些事实以三元组 $\langle h,r,t\rangle$ 形式存储。这些知识库涵盖了广泛的领域，并支持复杂的信息检索任务，包括回答需要多跳推理的问题。为了帮助大语言模型（LLM）在这些极为庞大的知识库环境中进行互动，我们还设计了一套专门针对知识库的工具集。同样，针对知识库的工具也包括导航工具和功能工具。导航工具帮助 LLM 高效地探索知识库（例如，get_relations() 和 get_attributes()），而功能工具则支持 LLM 执行精确的操作，如计数和交集（例如，intersection() 和 count()）。这两者对于完成复杂的知识库推理任务至关重要。知识库工具中的一个关键概念是变量，它代表一组实体，通常通过执行类似 get_neighbors() 或 intersection() 等函数生成作为中间结果。变量的使用促进了跨知识库的多跳推理，因为它可以自然地将一系列工具执行连接起来。总的来说，我们为知识库实现了 $7$ 种工具（附录 [A.2](https://arxiv.org/html/2402.14672v2#A1.SS2 "A.2 Knowledge Bases ‣ Appendix A Detailed Tool Definitions ‣ Middleware for LLMs: Tools Are Instrumental for Language Agents in Complex Environments")）。我们对知识库工具的设计严格遵循了知识库问答（KBQA）的一般需求，参见 Gu 等人（[2021](https://arxiv.org/html/2402.14672v2#bib.bib10)）；Cao 等人（[2022](https://arxiv.org/html/2402.14672v2#bib.bib3)）。

### 3.2 使用工具进行推理

我们首先选择 ReAct Yao 等人（[2022](https://arxiv.org/html/2402.14672v2#bib.bib44)）作为我们推理框架的核心，在这个框架中，LLM 可以根据自身的思维链（chain-of-thought，CoT）主动决定使用哪种工具。基于这个核心，我们提出了两种新方案，以提高工具使用的准确性，这对复杂任务至关重要，因为成功的工具使用要求仔细选择工具并精确分配参数。与现有依赖人工定义工作流、按照固定顺序使用工具的方法不同，Jiang 等人（[2023b](https://arxiv.org/html/2402.14672v2#bib.bib18)）的方法，我们的框架允许 LLM 在 CoT 的帮助下主动决定工具的选择。

正式地，在每一步 $t$，LLM 根据一种策略进行预测，该策略将当前上下文映射到输出：$\pi:c_{t}\rightarrow\hat{a}_{t}$，其中

|  | $\displaystyle c_{t}$ | $\displaystyle=(\hat{a}_{1},o_{1}\cdots,\hat{a}_{t-1},o_{t-1})$ |  |
| --- | --- | --- | --- |
|  | $\displaystyle\hat{a}_{t}$ | $\displaystyle=r_{t}\oplus a_{t}$ |  |

$\hat{a}_{t}$ 是理性思维 $r_{t}$（即 CoT 中的思维）和具体工具使用 $a_{t}$（例如，在图 [2](https://arxiv.org/html/2402.14672v2#S3.F2 "图 2 ‣ 3 LLM 中间件 ‣ 复杂环境中的语言智能体：工具对语言智能体至关重要") 中，$\hat{a}_{1}$ 是思维 1 和行动 1 的串联）拼接而成，而 $o_{t}$ 是来自环境的观察（即 $a_{t}$ 的执行结果）。在 ReAct 中，LLM 根据 $c_{t}$ 共同解码 $\hat{a}_{t}$，每个步骤都如此。然而，ReAct 最初是为像 Wikipedia 搜索 API 这样的简单工具设计的，当应用于更细致的工具使用时，天真的 ReAct 框架更容易产生与 $r_{t}$ 不一致的无效 $a_{t}$。我们提出了两种简单的策略来解决这个问题。第一种策略是通过在 LLM 错误使用工具时提供详细的错误反馈，并在这些信息的基础上提示重新尝试，从而简单地增强 ReAct（参见图 [2](https://arxiv.org/html/2402.14672v2#S3.F2 "图 2 ‣ 3 LLM 中间件 ‣ 复杂环境中的语言智能体：工具对语言智能体至关重要")(a)）。²²2 对于数据库，我们直接使用 sqlite3 的错误信息。对于知识库（KB），我们手动为每个工具定义了几种简单的错误反馈模板。这依赖于 LLM 通过反馈进行自我修正的能力 Gou et al. ([2023](https://arxiv.org/html/2402.14672v2#bib.bib8)); Chen et al. ([2023](https://arxiv.org/html/2402.14672v2#bib.bib5))，但当基础 LLM 较弱时，这种能力可能不可靠，可能导致重复相同的错误 Guan et al. ([2023](https://arxiv.org/html/2402.14672v2#bib.bib13))。此外，我们提出了分离生成，其中 LLM 的策略 $\pi$ 被拆分为两个顺序阶段（即，$\pi\propto\pi_{1}\circ\pi_{2}$），允许更细致地控制其行为。最初，LLM 仅解码思维 $r_{t}$，遵循 $\pi_{1}(r_{t}|c_{t})$。随后，LLM 在一个单独的上下文中预测行动 $a_{t}$，该上下文融合了思维 $r_{t}$ 和一组简单的规则 $\mathcal{M}$，这些规则决定了此步骤的合法行动。该过程进一步受到 $\pi_{2}$ 的引导，公式化为 $a_{t}\sim\pi_{2}(a_{t}|r_{t},\mathcal{M})$。$\mathcal{M}$ 封装了环境的主导规则（例如，get_neighbors() 的关系参数必须从 get_relations() 的输出中派生，该输出被应用于先前步骤中指定的实体参数），将先前的知识融入 LLM 的决策过程（参见图 [2](https://arxiv.org/html/2402.14672v2#S3.F2 "图 2 ‣ 3 LLM 中间件 ‣ 复杂环境中的语言智能体：工具对语言智能体至关重要")(b)）。我们使用的具体提示见附录 [C](https://arxiv.org/html/2402.14672v2#A3 "附录 C 提示 ‣ 复杂环境中的语言智能体：工具对语言智能体至关重要")。

| 模型 | 需求内容（N） | 需求内容（Y） | 总体 |
| --- | --- | --- | --- |
|  | EX | VA | EX | VA | EX | VA |
| 使用 Oracle 知识 |
| API 文档提示 Rajkumar 等人 ([2022](https://arxiv.org/html/2402.14672v2#bib.bib31)) |  |  |  |  |  |  |
|               使用 GPT-3.5-turbo | $38.1$ | $78.4$ | $32.1$ | $74.6$ | $36.1$ | $77.2$ |
|               使用 GPT-4 | $49.5$ | $95.5$ | $41.7$ | $89.9$ | $46.9$ | $93.7$ |
| 无 Oracle 知识 |
| API 文档提示 Rajkumar 等人 ([2022](https://arxiv.org/html/2402.14672v2#bib.bib31)) |  |  |  |  |  |  |
|               使用 GPT-3.5-turbo^† | $30.9$ | $82.9$ | $10.9$ | $80.0$ | $24.4$ | $82.0$ |
|               使用 GPT-4 | $38.2$ | $91.6$ | $13.8$ | $93.1$ | $30.4$ | $92.1$ |
| StructGPT Jiang 等人 ([2023b](https://arxiv.org/html/2402.14672v2#bib.bib18)) |  |  |  |  |  |  |
|               使用 GPT-3.5-turbo | $36.2$ | $86.5$ | $8.7$ | $80.8$ | $27.3$ | $84.7$ |
|               使用 GPT-4 | $40.7$ | $93.4$ | $13.5$ | $91.1$ | $31.8$ | $92.6$ |
| 中间件（错误反馈） |  |  |  |  |  |  |
|               使用 GPT-3.5-turbo | $38.8$ | $95.7$ | $19.8$ | $94.7$ | $32.7$ | $95.4$ |
|               使用 GPT-4 | $45.1$ | $98.8$ | $38.3$ | $97.2$ | $42.9$ | $98.3$ |

表 1：在 Bird 开发集上的结果。所有基准的性能均是在零-shot 设置下获得的。${\dagger}$ 表示在 Bird 官方排行榜上没有 Oracle 知识的最佳方法。使用 API 文档提示的预测由 Bird 作者直接提供。

| 模型 | 计数 | 最高级 | 无 | 总体 |
| --- | --- | --- | --- | --- |
|  | F1 | VA | F1 | VA | F1 | VA | F1 | VA |
| Pangu^♢ Gu 等人 ([2023](https://arxiv.org/html/2402.14672v2#bib.bib9)) |  |  |  |  |  |  |  |  |
|               使用 GPT-3.5-turbo | $10.1$ | $100.0$ | $9.0$ | $100.0$ | $23.4$ | $100.0$ | $18.1$ | $100.0$ |
|               使用 GPT-4 | $12.3$ | $100.0$ | $14.2$ | $100.0$ | $35.6$ | $100.0$ | $27.1$ | $100.0$ |
| KB-Binder Li 等人 ([2023c](https://arxiv.org/html/2402.14672v2#bib.bib21)) |  |  |  |  |  |  |  |  |
|               使用 GPT-3.5-turbo（20-shot） | $0.0$ | $33.7$ | $0.2$ | $19.4$ | $6.7$ | $37.0$ | $4.2$ | $32.8$ |
|               使用 GPT-4（20-shot） | $7.9$ | $48.3$ | $0.4$ | $28.2$ | $6.0$ | $45.8$ | $5.2$ | $42.6$ |
| StructGPT Jiang 等人 ([2023b](https://arxiv.org/html/2402.14672v2#bib.bib18)) |  |  |  |  |  |  |  |  |
|               使用 GPT-3.5-turbo | $4.5$ | $50.6$ | $3.9$ | $51.5$ | $11.4$ | $57.1$ | $8.6$ | $54.8$ |
|               使用 GPT-4 | $2.2$ | $37.1$ | $3.9$ | $30.1$ | $11.7$ | $26.3$ | $8.4$ | $29.0$ |
| 中间件（错误反馈） |  |  |  |  |  |  |  |  |
|               使用 GPT-3.5-turbo | $33.7$ | $70.7$ | $22.0$ | $64.1$ | $23.9$ | $56.8$ | $25.3$ | $60.8$ |
|               使用 GPT-4 | $70.7$ | $96.6$ | $39.9$ | $74.5$ | $55.8$ | $74.0$ | $55.1$ | $78.0$ |
| 中间件（解耦生成） |  |  |  |  |  |  |  |  |
|               使用 GPT-3.5-turbo | $48.9$ | $97.7$ | $29.5$ | $88.0$ | $32.1$ | $77.3$ | $34.3$ | $83.0$ |
|               使用GPT-4 | $74.1$ | $98.9$ | $42.6$ | $85.1$ | $61.0$ | $83.6$ | $59.3$ | $85.8$ |

表2：KBQA-Agent的结果。除KB-Binder外，所有模型都提供了单次示范，我们为KB-Binder提供了20次示范，以获得最佳性能。$\diamondsuit$表示我们重新实现的Pangu，因为原始代码不支持聊天模型。我们假设所有方法都具备完美的实体链接。

## 4 个基准

数据库和知识库的主要任务是文本到SQL的解析和知识库问答（KBQA）。然而，现有的流行基准测试可能不足以评估语言代理的即用型能力。具体而言，像WebQSP Berant等人（[2013](https://arxiv.org/html/2402.14672v2#bib.bib1)）；Yih等人（[2016](https://arxiv.org/html/2402.14672v2#bib.bib45)）这样的流行KBQA数据集中的大多数问题是单跳或双跳问题，对于这些问题，我们可以使用现有的语义解析方法有效地处理Gu等人（[2022](https://arxiv.org/html/2402.14672v2#bib.bib11)）。此外，Spider Yu等人（[2018](https://arxiv.org/html/2402.14672v2#bib.bib47)）和WikiSQL Zhong等人（[2017](https://arxiv.org/html/2402.14672v2#bib.bib50)）中的数据库在架构设计和表格行数方面的复杂性有限。这种过度简化使得我们可以直接将数据库架构输入到大型语言模型（LLM）中，凭借强大的性能，无需访问数据库的实际内容 Rajkumar等人（[2022](https://arxiv.org/html/2402.14672v2#bib.bib31)）。因此，我们需要不同的基准测试，具有更复杂的环境和指令，更好地反映语言代理在现实世界中必须处理的情况（请参阅附录[B](https://arxiv.org/html/2402.14672v2#A2 "附录 B 基准统计 ‣ LLM中介：工具对于复杂环境中的语言代理至关重要")中的基准统计数据）。

+   数据库

    对于数据库，我们利用Bird Li等人（[2023a](https://arxiv.org/html/2402.14672v2#bib.bib19)），这是一个最近的数据库数据集，以其复杂性著称，包含对高度复杂数据库的复杂指令。Bird原本有两种不同的设置：有和没有oracle知识，其中oracle知识提供有关目标数据库的特定信息，帮助完成每个任务。例如，“专门虚拟指的是Virtual = ‘F’”。在这种oracle知识的帮助下，环境的复杂性大大降低；它为任务提供了捷径，消除了对数据库的深入参与的必要性。这个作弊设置在实际应用中也不现实。因此，我们坚持使用没有oracle知识的设置。对于Bird开发集中的$1534$个问题，我们手动标注了是否需要访问数据库内容来编写SQL查询，注意到如果问题中提到的所有值恰好匹配数据库单元格，则无需访问。这有助于将语言代理的表现分解为需要更深入数据库交互（$496$个问题）与不需要（$1038$个问题）的情况，并能够提供对LLM表现的细粒度洞察。除了Bird中用于执行准确度（EX）的指标，它决定了预测的SQL执行结果是否与真实SQL的执行结果匹配，我们还评估了预测的SQL是否是一个有效的SQL查询（VA）。

+   知识库

    我们策划了KBQA-Agent，这是一个来源于现有KBQA数据集的新测试集，包含复杂的问题。具体而言，我们选择了$500$个多样化的问题，这些问题涉及至少三个关系，或两个关系加上聚合函数（即，计数或最上级）。对于每个问题，我们根据我们定义的工具集注释了一个真实的行动序列。³³3我们利用Gu和Su（[2022](https://arxiv.org/html/2402.14672v2#bib.bib12)）提供的黄金S表达式。具体而言，KBQA-Agent包括来自Freebase的三个KBQA数据集的问题：GrailQA Gu等人（[2021](https://arxiv.org/html/2402.14672v2#bib.bib10)），ComplexWebQ Talmor和Berant（[2018](https://arxiv.org/html/2402.14672v2#bib.bib40)），以及GraphQ Su等人（[2016](https://arxiv.org/html/2402.14672v2#bib.bib37)），确保了问题类型和来源的广泛性。KBQA-Agent的设计相比现有基准（附录[B](https://arxiv.org/html/2402.14672v2#A2 "Appendix B Benchmark Statistics ‣ Middleware for LLMs: Tools Are Instrumental for Language Agents in Complex Environments")）更能代表具有挑战性的现实世界场景。它为评估语言代理与大规模知识库交互提供了理想的测试平台。我们通过两个指标进行评估：答案实体的F1值和有效性（VA），这是一个二元指标，用于评估LLM是否能找到答案，无论是否正确。

![参见标题](img/f41ca85da3623e9c99fcedadee95afe7.png)

图 3：开源 LLM 在两个环境中仍然在很大程度上落后于 GPT-3.5-turbo 和 GPT-4。

## 5 实验

### 5.1 设置

+   实现

    为了在这两种环境中具体实现我们的工具，我们使用了数据库和知识库的标准查询接口，具体来说，使用 SQLite 处理数据库，使用 Virtuoso 处理知识库。然后，我们将工具描述与输入任务说明一同传递给 LLM（附录 [C](https://arxiv.org/html/2402.14672v2#A3 "Appendix C Prompts ‣ Middleware for LLMs: Tools Are Instrumental for Language Agents in Complex Environments")）。每种环境都展示了其独特的特征和挑战。在 KBQA 中，每个函数的参数要么是一个变量，要么是来自知识库模式的一个项（即关系或属性）。相比之下，在文本到 SQL 解析中，参数的种类更加多样，可能是 SQL 查询的一部分，也可能是一个完整的查询。这使得在解耦生成中列出潜在操作变得更加复杂，特别是在文本到 SQL 解析中。因此，我们仅为文本到 SQL 解析实现了错误反馈。

    对于底层 LLM，我们主要通过使用两种最先进的 LLM —— GPT-3.5-turbo-0613 OpenAI（[2023b](https://arxiv.org/html/2402.14672v2#bib.bib27)）和 GPT-4-0613 OpenAI（[2023a](https://arxiv.org/html/2402.14672v2#bib.bib26)）——将 Middleware 与基线方法进行比较，因为我们的目标是研究在复杂环境中运行的工具增强 LLM 的全部潜力。此外，我们还探索了四种开源 LLM，以更全面地评估我们的框架：Llama2-7B-Chat、Llama2-13B-Chat Touvron 等（[2023](https://arxiv.org/html/2402.14672v2#bib.bib41)），Mistral-7B-Instruct-v0.2 Jiang 等（[2023a](https://arxiv.org/html/2402.14672v2#bib.bib16)），以及 Mixtral 8$\times$7B-Instruct-v0.1 Jiang 等（[2024](https://arxiv.org/html/2402.14672v2#bib.bib17)）。

![参见标题](img/960780c2b5b4b3edda8eb4cae66d4a42.png)

图 4：定制的工具可以作为 LLM 与环境之间的有效中间件。

+   基线

    为了充分理解工具增强在帮助大型语言模型（LLM）处理复杂环境中的潜力，我们将中间件与一系列强大的基准方法进行比较。在文本到SQL解析方面，当LLM被适当提示并提供数据库架构（即API文档提示Rajkumar等人（[2022](https://arxiv.org/html/2402.14672v2#bib.bib31)））时，LLM展示了强大的SQL查询生成能力。这也代表了当前在没有oracle知识的情况下，在Bird排行榜上基于提示的方法的最新进展。此外，我们还与Bird排行榜上更多的基准方法进行了比较，这些方法原本并未在没有oracle知识的情况下提交结果（见附录[D.1](https://arxiv.org/html/2402.14672v2#A4.SS1 "D.1 Comparing with DIN-SQL and DAIL-SQL ‣ Appendix D Additional Results ‣ Middleware for LLMs: Tools Are Instrumental for Language Agents in Complex Environments")）。对于所有文本到SQL解析方法，我们采用零-shot设置。与文本到SQL解析不同，直接提示LLM并不能为KBQA生成合理的输出，因为KB架构的庞大规模。相反，基于LLM的现有KBQA方法通常遵循两种范式：一种是首先生成一个无基础的程序，然后将程序与KB架构进行匹配，之后再执行此步骤，Li等人（[2023c](https://arxiv.org/html/2402.14672v2#bib.bib21)）；Nie等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib25)）；另一种是逐步构建复杂的程序，并一步步地将其与KB架构进行匹配，Gu等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib9)）。我们将中间件与每种范式中最具代表性的工作进行比较，即KB-Binder，Li等人（[2023c](https://arxiv.org/html/2402.14672v2#bib.bib21)）和Pangu，Gu等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib9)）。我们还将StructGPT作为工具使用的额外基准方法进行比较。对于除KB-Binder外的所有KBQA方法，我们提供了一个one-shot示例，以获得更有意义的结果。

### 5.2 主要结果

如表格[1](https://arxiv.org/html/2402.14672v2#S3.T1 "表格 1 ‣ 3.2 使用工具推理 ‣ 3 LLMs 的中间件 ‣ LLMs 的中间件：工具对于复杂环境中的语言代理至关重要")和[2](https://arxiv.org/html/2402.14672v2#S3.T2 "表格 2 ‣ 3.2 使用工具推理 ‣ 3 LLMs 的中间件 ‣ LLMs 的中间件：工具对于复杂环境中的语言代理至关重要")所示，给 LLM 配备定制工具显著提高了性能，在多个指标下几乎使表现翻倍或三倍。具体而言，API 文档提示只能向 LLM 提供架构信息，因为数据库内容庞大。因此，它在需要数据库内容来构建 SQL 查询的示例中会失败。相比之下，中间件为代理提供了工具，使其能够主动浏览数据库并收集相关信息以构建 SQL 查询。因此，使用 GPT-4 时，中间件显著缩小了需要数据库内容和不需要数据库内容的问题之间的性能差距（即，$45.1$% vs. $38.3$%）。此外，我们注意到中间件在使用 GPT-4 时，将带有和不带有 oracle 知识的差距从 $15.5$% 缩小到 $4.0$%，使用 GPT-3.5-turbo 时从 $11.7$% 缩小到 $3.3$%。最后，StructGPT 展现了与 API 文档提示相似的趋势，因为它的工具没有提供任何关于数据库内容的信息。对于 KBQA，中间件在不同问题类型中表现一致优越，并且在 GPT-3.5-turbo 和 GPT-4 下都显著优于 Pangu。特别是，在 GPT-4 的支持下，中间件 + 解耦生成在 F1 分数上比 Pangu 高出 $32.2$%。至于其他两个基准，KB-Binder 和 StructGPT，都在我们的挑战性设置下惨遭失败。一方面，KB-Binder 仅能在实体的两跳内检索关系用于对接。然而，KBQA-Agent 中的大多数问题涉及超过两跳的关系。因此，它的许多生成程序无法完成对接，解释了其低 VA。另一方面，StructGPT 由于其有限的工具集而受到严重限制，无法处理 KBQA-Agent 中的复杂问题。因此，StructGPT 经常因为信息不足而拒绝提供答案（这从其低 VA 中可见一斑）。中间件的强大表现强调了工具对于复杂环境中语言代理的重要性。

由于空间限制，我们在附录[D](https://arxiv.org/html/2402.14672v2#A4 "附录 D 额外结果 ‣ LLMs 的中间件：工具对于复杂环境中的语言代理至关重要")中提供了额外的结果。

### 5.3 开源 LLM 实验

为了获得更深入的见解，我们还进行了四种开源LLM的实验（图 [3](https://arxiv.org/html/2402.14672v2#S4.F3 "图 3 ‣ 4 基准 ‣ 用于LLM的中间件：工具在复杂环境中的语言代理中的作用")）。我们的研究结果表明，Llama2模型通常表现不如其他LLM，这与在其他LLM排行榜上观察到的趋势一致，例如Chatbot Arena Zheng等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib49)）。具体而言，我们发现Llama2模型甚至在根据我们的指令生成语法正确的工具使用时也遇到了困难。另一方面，Mistral和Mixtral的表现远优于Llama2。特别是，Mixtral代表了一种先进的专家混合模型，表现出优越的性能，甚至在Chatbot Arena Zheng等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib49)）中超越了GPT-3.5-turbo。然而，与回答开放性问题不同，正确地与复杂环境互动要求语言代理生成更精确的动作，这些动作必须严格符合任务规范。Mixtral和GPT-3.5-turbo在预测复杂环境中有效动作方面仍存在差距。与GPT-3.5-turbo相比，Mixtral更倾向于输出无效动作。这也解释了为什么解耦生成，在其中输出空间严格限制为有效动作列表，对于较弱的模型有更大的帮助。通过中间件+解耦生成，使用Mistral几乎可以达到与GPT-3.5-turbo相同的最佳基准表现，使用Mixtral甚至可以匹敌GPT-4的最佳基准。虽然像GPT-4这样更强的模型可以通过错误反馈有效地纠正错误，但较弱的模型往往更能从解耦生成中受益。

### 5.4 作为中间件层的工具

为了深化我们对工具在帮助LLM接入复杂环境（即我们设置中的知识库三元组和数据库行）的核心作用的理解，我们通过与不同数量的数据项进行对比，进一步分析了中间件与提示基准的表现，这些数据项是直接从环境中采样的（图[4](https://arxiv.org/html/2402.14672v2#S5.F4 "图 4 ‣ 5.1 设置 ‣ 5 实验 ‣ 中间件对LLM的作用：工具在复杂环境中对语言代理的关键作用")）。对于知识库，我们基于每个问题中实体的三跳邻域，从Freebase中采样了$10$、$50$、$100$和$200$个三元组。这些三元组是使用句子-BERT检索器（Reimers 和 Gurevych [2019](https://arxiv.org/html/2402.14672v2#bib.bib32)）根据它们与输入问题的相似度从Freebase中排序得到的。我们直接使用这些采样的三元组来提示LLM，并要求它生成对给定问题的回答。鉴于Freebase的庞大规模，仅使用部分样本来准确表示环境是非常困难的。因此，GPT-3.5 Turbo和GPT-4的F1分数始终接近$0$。对于数据库，我们同样增强了API文档提示，使用了$1$、$5$和$10$个采样行进行每个表的测试，并在需要访问数据库内容的$100$个来自Bird的随机问题上进行了评估。此外，我们还在数据库设置中使用相同的采样行增强了中间件。我们观察到，最初增加更多的数据库行会提高基准性能，但最终会导致性能下降。在使用中间件时，通过采样的行提示LLM的增益非常小，而没有采样行的标准设置已经显著优于所有基准。这些结果进一步证实，当LLM与工具结合使用时，它可以有效地与复杂环境互动，灵活地按需收集必要的信息，并绕过其处理数据量的限制（例如，每个表约$200$个三元组或$10$行数据）。

## 6 结论

一种开创性的视角是让语言代理帮助人类应对复杂的现实世界任务。本文展示了通过精心设计的工具作为大型语言模型（LLM）与复杂环境之间的中间件，LLM能够大幅超越当前的解决方案。我们的结果突显了这些专用工具在释放LLM潜力方面的不可或缺的作用，特别是在以往面临巨大挑战的复杂现实世界任务中。

## 限制

本文旨在回答我们提出的一个重要问题：大规模语言模型（LLMs）如何在工具的帮助下有效处理复杂环境？我们通过在两个典型环境——知识库（KBs）和数据库中的评估来探讨这一问题。尽管在这些环境中我们取得了显著的成果，但需要注意的是，相较于没有直接查询接口的环境（如网页或物理环境），为知识库和数据库实施定制工具的挑战较少。在未来的工作中，我们计划将中间件扩展到更广泛的环境中，旨在通过整合定制的中间件工具，充分发挥语言智能体在复杂环境中的潜力。

此外，本研究中开发的工具完全基于我们的经验。尽管如此，我们的结果已经展示了在复杂环境中通过定制工具增强大规模语言模型（LLMs）潜力的重要性，这与本文的主要目标一致。然而，为了进一步提升性能，采用更为有原则的工具设计策略是必要的。此外，Wang等人（[2024](https://arxiv.org/html/2402.14672v2#bib.bib42)）在复杂环境中探索自动化工具制作的方法，为未来研究提供了一个有前景的方向。

## 参考文献

+   Berant等（2013）Jonathan Berant、Andrew Chou、Roy Frostig和Percy Liang。2013年。[基于问题-答案对的Freebase语义解析](https://aclanthology.org/D13-1160)。收录于*2013年自然语言处理实证方法会议论文集*，第1533-1544页，美国华盛顿州西雅图，计算语言学会。

+   Bollacker等（2008）Kurt D. Bollacker、Colin Evans、Praveen K. Paritosh、Tim Sturge和Jamie Taylor。2008年。[Freebase：一个协作创建的图形数据库，用于构建人类知识](https://doi.org/10.1145/1376616.1376746)。收录于*ACM SIGMOD国际数据管理会议论文集，SIGMOD 2008，加拿大温哥华，2008年6月10-12日*，第1247-1250页。ACM。

+   Cao等（2022）Shulin Cao、Jiaxin Shi、Liangming Pan、Lunyiu Nie、Yutong Xiang、Lei Hou、Juanzi Li、Bin He和Hanwang Zhang。2022年。[KQA Pro：一个用于复杂问题回答的显式组合程序数据集](https://doi.org/10.18653/v1/2022.acl-long.422)。收录于*第60届计算语言学会年会论文集（第一卷：长篇论文），ACL 2022，爱尔兰都柏林，2022年5月22-27日*，第6101-6119页。计算语言学会。

+   Chandu等（2021）Khyathi Raghavi Chandu、Yonatan Bisk和Alan W. Black。2021年。[自然语言处理中的“基础”问题](https://doi.org/10.18653/V1/2021.FINDINGS-ACL.375)。收录于*计算语言学会的研究成果：ACL/IJCNLP 2021，在线活动，2021年8月1日至6日*，《ACL研究成果》第4283-4305页。计算语言学会。

+   Chen等人（2023）Xinyun Chen, Maxwell Lin, Nathanael Schärli, 和 Denny Zhou. 2023. [教大型语言模型自我调试](https://doi.org/10.48550/ARXIV.2304.05128). *CoRR*, abs/2304.05128。

+   Deng等人（2023）Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Samuel Stevens, Boshi Wang, Huan Sun, 和 Yu Su. 2023. [Mind2web：面向网络的通用智能体](https://doi.org/10.48550/ARXIV.2306.06070). *CoRR*, abs/2306.06070。

+   Gao等人（2024）Dawei Gao, Haibin Wang, Yaliang Li, Xiuyu Sun, Yichen Qian, Bolin Ding, 和 Jingren Zhou. 2024. [通过大型语言模型增强的文本到SQL：基准评估](https://www.vldb.org/pvldb/vol17/p1132-gao.pdf). *Proc. VLDB Endow.*, 17(5):1132–1145。

+   Gou等人（2023）Zhibin Gou, Zhihong Shao, Yeyun Gong, Yelong Shen, Yujiu Yang, Nan Duan, 和 Weizhu Chen. 2023. [CRITIC：大型语言模型通过工具交互批判实现自我纠正](https://doi.org/10.48550/ARXIV.2305.11738). *CoRR*, abs/2305.11738。

+   Gu等人（2023）Yu Gu, Xiang Deng, 和 Yu Su. 2023. [不要生成，进行区分：一种将语言模型与现实世界环境对接的提案](https://doi.org/10.18653/v1/2023.acl-long.270). 发表在*第61届计算语言学协会年会论文集（第一卷：长篇论文），ACL 2023，加拿大多伦多，2023年7月9-14日*，第4928–4949页。计算语言学协会。

+   Gu等人（2021）Yu Gu, Sue Kase, Michelle Vanni, Brian M. Sadler, Percy Liang, Xifeng Yan, 和 Yu Su. 2021. [超越独立同分布：知识库问答的三种泛化层次](https://doi.org/10.1145/3442381.3449992). 发表在*WWW ’21：2021年网页会议，虚拟会议/斯洛文尼亚卢布尔雅那，2021年4月19-23日*，第3477–3488页。ACM / IW3C2。

+   Gu等人（2022）Yu Gu, Vardaan Pahuja, Gong Cheng, 和 Yu Su. 2022. [知识库问答：一种语义解析的视角](https://doi.org/10.48550/arXiv.2209.04994). 发表在*第四届自动化知识库构建会议*。

+   Gu和Su（2022）Yu Gu 和 Yu Su. 2022. [ArcaneQA：用于知识库问答的动态程序归纳与上下文化编码](https://aclanthology.org/2022.coling-1.148). 发表在*第29届国际计算语言学大会论文集*，第1718–1731页，韩国庆州。国际计算语言学委员会。

+   Guan等人（2023）Lin Guan, Karthik Valmeekam, Sarath Sreedharan, 和 Subbarao Kambhampati. 2023. [利用预训练的大型语言模型构建和使用世界模型进行基于模型的任务规划](https://doi.org/10.48550/ARXIV.2305.14909). *CoRR*, abs/2305.14909。

+   Hao等人（2023）Shibo Hao, Tianyang Liu, Zhen Wang, 和 Zhiting Hu. 2023. [Toolkengpt：通过工具嵌入增强冻结语言模型与大量工具的结合](https://doi.org/10.48550/arXiv.2305.11554). *CoRR*, abs/2305.11554。

+   Hwang et al. (2019) 元硕黄，金英任，承贤朴，闵俊徐。2019. [基于表格感知词上下文的WikiSQL全面探索](http://arxiv.org/abs/1902.01069)。*CoRR*，abs/1902.01069。

+   Jiang et al. (2023a) 阿尔伯特·Q·蒋，亚历山大·萨布雷罗尔，亚瑟·曼施，克里斯·班福德，德文德拉·辛格·查普洛特，迭戈·德·拉斯·卡萨斯，弗洛里安·布雷桑，贾娜·伦吉尔，吉约姆·兰普尔，露西尔·索尔尼尔，莱利奥·雷纳德·拉沃，玛丽-安·拉肖，皮埃尔·斯托克，特文·勒·斯卡奥，蒂博·拉夫里尔，托马斯·王，蒂莫西·拉克鲁瓦，威廉·艾尔·赛义德。2023a. [Mistral 7b](https://doi.org/10.48550/ARXIV.2310.06825)。*CoRR*，abs/2310.06825。

+   Jiang et al. (2024) 阿尔伯特·Q·蒋，亚历山大·萨布雷罗尔，安托万·鲁，亚瑟·曼施，布兰什·萨瓦里，克里斯·班福德，德文德拉·辛格·查普洛特，迭戈·德·拉斯·卡萨斯，艾玛·布胡娜，弗洛里安·布雷桑，贾娜·伦吉尔，吉约姆·布尔，吉约姆·兰普尔，莱利奥·雷纳德·拉沃，露西尔·索尔尼尔，玛丽-安·拉肖，皮埃尔·斯托克，桑迪普·苏布拉马尼安，索非亚·杨，斯基蒙·安东尼亚克，特文·勒·斯卡奥，特奥菲尔·热尔韦，蒂博·拉夫里尔，托马斯·王，蒂莫西·拉克鲁瓦，威廉·艾尔·赛义德。2024. [Mixtral of experts](http://arxiv.org/abs/2401.04088)。*CoRR*。

+   Jiang et al. (2023b) 金豪蒋，昆周，子灿董，克铭叶，伟新赵，纪荣温。2023b. [StructGPT：一种大型语言模型推理结构化数据的通用框架](https://doi.org/10.48550/arXiv.2305.09645)。*CoRR*，abs/2305.09645。

+   Li et al. (2023a) 金阳李，宾源惠，葛渠，彬华李，佳熙杨，博文李，柏林王，博文秦，荣宇曹，瑞英耿，南霍，轩鹤周，晨浩马，国亮李，凯文·陈-川·张，飞黄，雷诺德·程，永斌李。2023a. [LLM是否已经能够作为数据库接口？一种大规模数据库基础的文本到SQL的大型基准测试](https://doi.org/10.48550/ARXIV.2305.03111)。*CoRR*，abs/2305.03111。

+   Li et al. (2023b) 明昊李，飞凡宋，博文于，海洋于，周俊李，飞黄，永斌李。2023b. [API-Bank：一种工具增强的LLM基准](https://doi.org/10.48550/arXiv.2304.08244)。*CoRR*，abs/2304.08244。

+   Li et al. (2023c) 天乐李，雪光马，亚历克斯庄，余谷，余苏，文虎陈。2023c. [基于知识库问答的少样本上下文学习](https://api.semanticscholar.org/CorpusID:258461017)。发表于*计算语言学会年会*。

+   Liu et al. (2023) 晓刘，昊于，涵辰张，亦凡徐，轩宇雷，韩宇赖，余谷，杭良丁，凯文·门，克娟杨，树丹张，向邓，傲涵曾，正晓杜，晨辉张，胜申，天俊张，余苏，欢孙，敏理黄，宇晓董，捷唐。2023. [AgentBench：评估LLM作为代理](https://doi.org/10.48550/arXiv.2308.03688)。*CoRR*，abs/2308.03688。

+   Lu et al. (2023) Pan Lu, Baolin Peng, Hao Cheng, Michel Galley, Kai-Wei Chang, Ying Nian Wu, Song-Chun Zhu, and Jianfeng Gao. 2023. [Chameleon: 插拔式大语言模型的组合推理](https://doi.org/10.48550/ARXIV.2304.09842)。*CoRR*, abs/2304.09842.

+   Mialon et al. (2023) Grégoire Mialon, Roberto Dessì, Maria Lomeli, Christoforos Nalmpantis, Ramakanth Pasunuru, Roberta Raileanu, Baptiste Rozière, Timo Schick, Jane Dwivedi-Yu, Asli Celikyilmaz, Edouard Grave, Yann LeCun, and Thomas Scialom. 2023. [增强型语言模型：一项综述](https://doi.org/10.48550/ARXIV.2302.07842)。*CoRR*, abs/2302.07842.

+   Nie et al. (2023) Zhijie Nie, Richong Zhang, Zhongyuan Wang, 和 Xudong Liu. 2023. [基于代码风格的上下文学习用于知识驱动的问答](https://doi.org/10.48550/ARXIV.2309.04695)。*CoRR*, abs/2309.04695.

+   OpenAI (2023a) OpenAI. 2023a. [GPT-4 技术报告](https://doi.org/10.48550/arXiv.2303.08774)。*CoRR*, abs/2303.08774.

+   OpenAI (2023b) OpenAI. 2023b. 模型 - OpenAI API. [https://platform.openai.com/docs/models/gpt-3-5](https://platform.openai.com/docs/models/gpt-3-5).

+   Pourreza and Rafiei (2023) Mohammadreza Pourreza 和 Davood Rafiei. 2023. [DIN-SQL：带自我修正的分解式上下文学习文本到SQL](http://papers.nips.cc/paper_files/paper/2023/hash/72223cc66f63ca1aa59edaec1b3670e6-Abstract-Conference.html)。在 *神经信息处理系统进展 36：神经信息处理系统年会 2023, NeurIPS 2023, 美国新奥尔良, 2023年12月10日至16日*。

+   Qin et al. (2023a) Yujia Qin, Shengding Hu, Yankai Lin, Weize Chen, Ning Ding, Ganqu Cui, Zheni Zeng, Yufei Huang, Chaojun Xiao, Chi Han, Yi Ren Fung, Yusheng Su, Huadong Wang, Cheng Qian, Runchu Tian, Kunlun Zhu, Shihao Liang, Xingyu Shen, Bokai Xu, Zhen Zhang, Yining Ye, Bowen Li, Ziwei Tang, Jing Yi, Yuzhang Zhu, Zhenning Dai, Lan Yan, Xin Cong, Yaxi Lu, Weilin Zhao, Yuxiang Huang, Junxi Yan, Xu Han, Xian Sun, Dahai Li, Jason Phang, Cheng Yang, Tongshuang Wu, Heng Ji, Zhiyuan Liu, and Maosong Sun. 2023a. [基础模型的工具学习](https://doi.org/10.48550/arXiv.2304.08354)。*CoRR*, abs/2304.08354.

+   Qin et al. (2023b) Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, Sihan Zhao, Runchu Tian, Ruobing Xie, Jie Zhou, Mark Gerstein, Dahai Li, Zhiyuan Liu, and Maosong Sun. 2023b. [ToolLLM：帮助大语言模型掌握16000+真实世界API](https://doi.org/10.48550/ARXIV.2307.16789)。*CoRR*, abs/2307.16789.

+   Rajkumar et al. (2022) Nitarshan Rajkumar, Raymond Li, and Dzmitry Bahdanau. 2022. [评估大语言模型的文本到SQL能力](https://doi.org/10.48550/ARXIV.2204.00498)。*CoRR*, abs/2204.00498.

+   Reimers 和 Gurevych（2019）Nils Reimers 和 Iryna Gurevych. 2019. [Sentence-BERT：使用 Siamese BERT 网络的句子嵌入](https://doi.org/10.18653/v1/D19-1410). 载于 *2019年自然语言处理实证方法会议暨第九届国际自然语言处理联合会议（EMNLP-IJCNLP）论文集*，第3982–3992页，中国香港。计算语言学协会。

+   Schick 等人（2023）Timo Schick、Jane Dwivedi-Yu、Roberto Dessì、Roberta Raileanu、Maria Lomeli、Luke Zettlemoyer、Nicola Cancedda 和 Thomas Scialom. 2023. [Toolformer：语言模型能够自我学习使用工具](https://doi.org/10.48550/arXiv.2302.04761). *CoRR*，abs/2302.04761。

+   Shridhar 等人（2021）Mohit Shridhar、Xingdi Yuan、Marc-Alexandre Côté、Yonatan Bisk、Adam Trischler 和 Matthew J. Hausknecht. 2021. [Alfworld：将文本与具象环境对齐以进行互动学习](https://openreview.net/forum?id=0IOX0YcCdTn). 载于 *第9届国际学习表示会议，ICLR 2021，虚拟会议，奥地利，2021年5月3日至7日*。OpenReview.net。

+   Song 等人（2023）Chan Hee Song、Jiaman Wu、Clayton Washington、Brian M. Sadler、Wei-Lun Chao 和 Yu Su. 2023. LLM-Planner：基于大型语言模型的面向具象代理的少样本规划. 载于 *IEEE/CVF 国际计算机视觉会议（ICCV）论文集*。

+   Su（2023）Yu Su. 2023. [语言代理：人工智能的一个关键进化步骤](https://yusu.substack.com/p/language-agents). *yusu.substack.com*。

+   Su 等人（2016）Yu Su、Huan Sun、Brian M. Sadler、Mudhakar Srivatsa、Izzeddin Gur、Zenghui Yan 和 Xifeng Yan. 2016. [生成用于 QA 评估的特征丰富问题集](https://doi.org/10.18653/V1/D16-1054). 载于 *2016年自然语言处理实证方法会议论文集，EMNLP 2016，奥斯汀，德州，美国，2016年11月1日至4日*，第562–572页。计算语言学协会。

+   Sun 等人（2023）Shuo Sun、Yuchen Zhang、Jiahuan Yan、Yuze Gao、Donovan Ong、Bin Chen 和 Jian Su. 2023. [大型语言模型之战：Dolly vs LLaMA vs vicuna vs guanaco vs bard vs ChatGPT —— 一项文本到 SQL 的解析比较](https://doi.org/10.18653/v1/2023.findings-emnlp.750). 载于 *计算语言学协会论文集：EMNLP 2023*，第11225–11238页，新加坡。计算语言学协会。

+   Tai 等人（2023）Chang-Yu Tai、Ziru Chen、Tianshu Zhang、Xiang Deng 和 Huan Sun. 2023. [探索链式思维风格的提示在文本到 SQL 中的应用](https://aclanthology.org/2023.emnlp-main.327). 载于 *2023年自然语言处理实证方法会议论文集，EMNLP 2023，新加坡，2023年12月6日至10日*，第5376–5393页。计算语言学协会。

+   Talmor 和 Berant（2018）Alon Talmor 和 Jonathan Berant。2018年。[网络作为回答复杂问题的知识库](https://doi.org/10.18653/V1/N18-1059)。发表于 *2018年北美计算语言学协会：人类语言技术年会论文集，NAACL-HLT 2018，路易斯安那州新奥尔良，2018年6月1-6日，第1卷（长篇论文）*，第641-651页。计算语言学协会。

+   Touvron 等人（2023）Hugo Touvron、Louis Martin、Kevin Stone、Peter Albert、Amjad Almahairi、Yasmine Babaei、Nikolay Bashlykov、Soumya Batra、Prajjwal Bhargava、Shruti Bhosale、Dan Bikel、Lukas Blecher、Cristian Canton-Ferrer、Moya Chen、Guillem Cucurull、David Esiobu、Jude Fernandes、Jeremy Fu、Wenyin Fu、Brian Fuller、Cynthia Gao、Vedanuj Goswami、Naman Goyal、Anthony Hartshorn、Saghar Hosseini、Rui Hou、Hakan Inan、Marcin Kardas、Viktor Kerkez、Madian Khabsa、Isabel Kloumann、Artem Korenev、Punit Singh Koura、Marie-Anne Lachaux、Thibaut Lavril、Jenya Lee、Diana Liskovich、Yinghai Lu、Yuning Mao、Xavier Martinet、Todor Mihaylov、Pushkar Mishra、Igor Molybog、Yixin Nie、Andrew Poulton、Jeremy Reizenstein、Rashi Rungta、Kalyan Saladi、Alan Schelten、Ruan Silva、Eric Michael Smith、Ranjan Subramanian、Xiaoqing Ellen Tan、Binh Tang、Ross Taylor、Adina Williams、Jian Xiang Kuan、Puxin Xu、Zheng Yan、Iliyan Zarov、Yuchen Zhang、Angela Fan、Melanie Kambadur、Sharan Narang、Aurélien Rodriguez、Robert Stojnic、Sergey Edunov 和 Thomas Scialom。2023年。[Llama 2：开放的基础模型和微调聊天模型](https://doi.org/10.48550/arXiv.2307.09288)。*CoRR*，abs/2307.09288。

+   Wang 等人（2024）Zhiruo Wang、Daniel Fried 和 Graham Neubig。2024年。[Trove：诱导可验证和高效的工具箱以解决编程任务](https://doi.org/10.48550/ARXIV.2401.12869)。*CoRR*，abs/2401.12869。

+   Wei 等人（2022）Jason Wei、Xuezhi Wang、Dale Schuurmans、Maarten Bosma、Brian Ichter、Fei Xia、Ed H. Chi、Quoc V. Le 和 Denny Zhou。2022年。[链式思维提示促使大型语言模型进行推理](http://papers.nips.cc/paper_files/paper/2022/hash/9d5609613524ecf4f15af0f7b31abca4-Abstract-Conference.html)。发表于 *NeurIPS*。

+   Yao 等人（2022）Shunyu Yao、Jeffrey Zhao、Dian Yu、Nan Du、Izhak Shafran、Karthik Narasimhan 和 Yuan Cao。2022年。[ReAct：在语言模型中协同推理与行动](https://doi.org/10.48550/arXiv.2210.03629)。*CoRR*，abs/2210.03629。

+   Yih 等人（2016）Wen-tau Yih、Matthew Richardson、Chris Meek、Ming-Wei Chang 和 Jina Suh。2016年。[语义解析标注在知识库问答中的价值](https://doi.org/10.18653/v1/P16-2033)。发表于 *第54届计算语言学协会年会论文集（第2卷：短篇论文）*，第201-206页，德国柏林。计算语言学协会。

+   Yu 等人（2023）Donghan Yu, Sheng Zhang, Patrick Ng, Henghui Zhu, Alexander Hanbo Li, Jun Wang, Yiqun Hu, William Yang Wang, Zhiguo Wang, 和 Bing Xiang. 2023. [DecAF: 基于知识库的问答中答案与逻辑形式的联合解码](https://openreview.net/pdf?id=XHc5zRPxqV9). 载于 *第十一届国际学习表示会议，ICLR 2023，卢旺达基加利，2023年5月1-5日*。OpenReview.net.

+   Yu 等人（2018）Tao Yu, Rui Zhang, Kai Yang, Michihiro Yasunaga, Dongxu Wang, Zifan Li, James Ma, Irene Li, Qingning Yao, Shanelle Roman, Zilin Zhang, 和 Dragomir R. Radev. 2018. [Spider: 用于复杂跨领域语义解析和文本到SQL任务的大规模人工标注数据集](https://doi.org/10.18653/v1/d18-1425). 载于 *2018年自然语言处理实证方法会议论文集，比利时布鲁塞尔，2018年10月31日 - 11月4日*，第3911-3921页。计算语言学协会。

+   Zhang 等人（2018）Yuyu Zhang, Hanjun Dai, Zornitsa Kozareva, Alexander J. Smola, 和 Le Song. 2018. [基于知识图谱的问答推理](https://doi.org/10.1609/AAAI.V32I1.12057). 载于 *第三十二届人工智能AAAI会议论文集（AAAI-18）及第三十届创新人工智能应用会议（IAAI-18），第八届人工智能教育进展研讨会（EAAI-18），美国路易斯安那州新奥尔良，2018年2月2-7日*，第6069-6076页。AAAI Press.

+   Zheng 等人（2023）Lianmin Zheng, Wei-Lin Chiang, Ying Sheng, Siyuan Zhuang, Zhanghao Wu, Yonghao Zhuang, Zi Lin, Zhuohan Li, Dacheng Li, Eric. P Xing, Hao Zhang, Joseph E. Gonzalez, 和 Ion Stoica. 2023. [通过MT-Bench和Chatbot Arena评估LLM作为判官的能力](http://arxiv.org/abs/2306.05685).

+   Zhong 等人（2017）Victor Zhong, Caiming Xiong, 和 Richard Socher. 2017. [Seq2SQL: 使用强化学习从自然语言生成结构化查询](http://arxiv.org/abs/1709.00103). *CoRR*, abs/1709.00103.

## 附录

在本补充材料中，我们提供了以下进一步的详细信息：

+   •

    [附录 A](https://arxiv.org/html/2402.14672v2#A1 "附录 A 工具定义详细信息 ‣ 中间件对于复杂环境中语言代理的工具作用"): 工具定义详细信息

+   •

    [附录 B](https://arxiv.org/html/2402.14672v2#A2 "附录 B 基准统计 ‣ 中间件对于复杂环境中语言代理的工具作用"): 基准统计

+   •

    [附录 C](https://arxiv.org/html/2402.14672v2#A3 "附录 C 提示词 ‣ 中间件对于复杂环境中语言代理的工具作用"): 提示词

+   •

    [附录 D](https://arxiv.org/html/2402.14672v2#A4 "附录 D 额外结果 ‣ 中间件对于复杂环境中语言代理的工具作用"): 额外结果

## 附录 A 工具定义详细信息

在本节中，我们详细介绍了我们为这两种环境定制的工具的描述。具体来说，我们为数据库实现了$12$种不同的工具，为知识库（KB）实现了$7$种不同的工具。工具的选择是基于我们对这些环境的领域知识精心制定的。请注意，对于数据库，我们通过API文档格式中的DB模式信息直接提示LLM（大语言模型），如Rajkumar等人所述（[2022](https://arxiv.org/html/2402.14672v2#bib.bib31)），因此我们的工具重点帮助LLM更好地与数据库内容进行互动。

### A.1 数据库

数据库的导航工具：

<svg class="ltx_picture" height="103.89" id="A1.SS1.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,103.89) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 77.06)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="239.92">find_columns_containing_value(value)</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="63.65" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">此函数可以帮助查找包含给定单元格值的列，帮助你做出更好的决策，选择正确的列进行解码。请注意，这里的值指的是列中行的单元格值，而不是列名。前提条件：无</foreignobject></g></g></svg><svg class="ltx_picture" height="120.49" id="A1.SS1.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,120.49) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 93.66)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="281.81">find_columns_containing_value_fuzzy(value)</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="80.25" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">有时，`find_columns_containing_cell_value`函数可能无法找到具有精确匹配单元格值的列。此函数可以帮助通过模糊匹配查找可能包含目标单元格值的列。请注意，这里的值指的是列中行的单元格值，而不是列名。前提条件：无</foreignobject></g></g></svg><svg class="ltx_picture" height="87.28" id="A1.SS1.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,87.28) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 60.45)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="215.01">get_distinct_values(table, column)</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="47.05" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">返回给定列中的不同值。这可以帮助你做出更好的决策，选择正确的值进行解码。前提条件：无</foreignobject></g></g></svg><svg class="ltx_picture" height="87.28" id="A1.SS1.p5.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,87.28) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 60.45)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="260.29">is_value_in_column(table, column, value)</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="47.05" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">返回给定值是否在指定列中。你可以使用此函数来更好地检测正确的列。前提条件：无</foreignobject></g></g></svg><svg class="ltx_picture" height="87.28" id="A1.SS1.p6.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,87.28) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 60.45)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="198.75">get_date_format(table, column)</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="47.05" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">返回给定日期列的示例项。这可以帮助你更好地理解该列中的日期格式。前提条件：无</foreignobject></g></g></svg><svg class="ltx_picture" height="70.68" id="A1.SS1.p7.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,70.68) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 43.85)"><

数据库的功能工具：

<svg class="ltx_picture" height="87.28" id="A1.SS1.p9.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,87.28) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 60.45)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="137.37">from(from_statement)</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="47.05" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">该函数指定 FROM 子句，例如，from("FROM table1") 或 from("FROM table1 JOIN table2 ON table1.id = table2.id") 前提条件：无</foreignobject></g></g></svg><svg class="ltx_picture" height="69.14" id="A1.SS1.p10.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,69.14) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 42.31)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="151.98">where(where_statement)</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="28.9" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">该函数指定 WHERE 子句，例如，where("WHERE table1.id = 1")。前提条件：from</foreignobject></g></g></svg><svg class="ltx_picture" height="69.14" id="A1.SS1.p11.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,69.14) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 42.31)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="147.44">select(select_statement)</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="28.9" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">该函数指定 SELECT 子句，例如，select("SELECT table1.id")。前提条件：from，where</foreignobject></g></g></svg><svg class="ltx_picture" height="69.14" id="A1.SS1.p12.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,69.14) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 42.31)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="200.79">group_by(group_by_statement)</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="28.9" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">该函数指定 GROUP BY 子句，例如，group_by("GROUP BY table1.id")。前提条件：from，where，select</foreignobject></g></g></svg><svg class="ltx_picture" height="69.3" id="A1.SS1.p13.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,69.3) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 42.47)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="161.51">having(having_statement)</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="29.06" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">该函数指定 HAVING 子句，例如，having("HAVING table1.id = 1")。前提条件：from，where，select，group_by</foreignobject></g></g></svg><svg class="ltx_picture" height="85.75" id="A1.SS1.p14.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,85.75) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 58.92)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.

### A.2 知识库

知识库的导航工具：

<svg class="ltx_picture" height="170.31" id="A1.SS2.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,170.31) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 143.48)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="253.64">get_relations(variable) -> list of relations</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="130.07" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">一个变量可以是一个实体或一组实体（即，先前查询的结果）。此功能有助于导航所有与该变量相关的知识库中的关系，因此您可以决定哪个关系最有助于找到问题的答案。一个简单的用例可以是‘get_relations(Barack Obama)’，它查找从实体巴拉克·奥巴马开始的所有关系/边。get_relations的参数应始终是一个实体或一个变量（例如，#0），而不是其他任何东西。前提条件：无</foreignobject></g></g></svg><svg class="ltx_picture" height="135.71" id="A1.SS2.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,135.71) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 108.88)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="190.84">get_neighbors(v, r) -> variable</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="95.48" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">给定一个变量，此功能返回通过给定关系与该变量连接的所有实体。请注意，get_neighbors()只能在使用get_relations()找到一组可行关系后使用。一个简单的用例可以是‘get_neighbors(Barack Obama, people.person.profession)’，它返回奥巴马在Freebase中的职业。前提条件：get_relations</foreignobject></g></g></svg><svg class="ltx_picture" height="85.9" id="A1.SS2.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,85.9) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 59.07)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="229">get_attributes(v) -> list of attributes</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="45.66" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">此功能有助于查找变量的所有数值属性。只有在问题寻求极值聚合（即，argmax或argmin）时，才应使用它。前提条件：get_neighbors</foreignobject></g></g></svg>

知识库的功能工具：

<svg class="ltx_picture" height="119.11" id="A1.SS2.p6.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,119.11) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 92.28)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="150.36">argmax(v, a) -> variable</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="78.87" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">给定一个变量，该函数返回具有给定属性最大值的实体。它只能在使用get_attributes()找到一组可行属性之后使用。一个简单的使用案例是‘argmax(variable, age)’，它返回属于该变量的最年长实体。前提条件：get_attributes</foreignobject></g></g></svg><svg class="ltx_picture" height="119.11" id="A1.SS2.p7.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,119.11) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 92.28)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="147.67">argmin(v, a) -> variable</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="78.87" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">给定一个变量，该函数返回具有给定属性最小值的实体。它只能在使用get_attributes()找到一组可行属性之后使用。一个简单的使用案例是‘argmin(variable, age)’，它返回属于该变量的最年轻实体。前提条件：get_attributes</foreignobject></g></g></svg><svg class="ltx_picture" height="85.9" id="A1.SS2.p8.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,85.9) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 59.07)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="189.64">intersection(v1, v2) -> variable</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="45.66" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">给定两个变量，该函数返回两个变量的交集。这两个变量必须是相同类型的。前提条件：get_neighbors</foreignobject></g></g></svg><svg class="ltx_picture" height="69.3" id="A1.SS2.p9.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,69.3) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 7.87 42.47)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.96)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="92.63">count(v) -> int</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 8.67 8.67)"><foreignobject color="#000000" height="29.06" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="582.65">给定一个变量，该函数返回属于该变量的实体数量。前提条件：get_neighbors</foreignobject></g></g></svg>

## 附录 B 基准统计

| 数据集 | # 表格/数据库 | # 行/数据库 | % 需要连续性 |
| --- | --- | --- | --- |
| WikiSQL 钟等人 ([2017](https://arxiv.org/html/2402.14672v2#bib.bib50)) | $1$ | $17$ | $0.0$ |
| Spider 于等人 ([2018](https://arxiv.org/html/2402.14672v2#bib.bib47)) | $5.1$ | $2$K | $0.0$ |
| Bird 李等人 ([2023a](https://arxiv.org/html/2402.14672v2#bib.bib19)) | $7.3$ | $549$K | $32.3$ |

(a) 数据库

| 数据集 | # 关系/KB | # 三元组/KB | # 跳数 | % 具有聚合函数 |
| --- | --- | --- | --- | --- |
| MetaQA 张等人 ([2018](https://arxiv.org/html/2402.14672v2#bib.bib48)) | $9$ | $135$K | $2.1$ | $0.0$ |
| WebQSP Yih 等人 ([2016](https://arxiv.org/html/2402.14672v2#bib.bib45)) | $19$K | $3$B | $1.5$ | $4.9$ |
| GrailQA 顾等人 ([2021](https://arxiv.org/html/2402.14672v2#bib.bib10)) | $19$K | $3$B | $1.4$ | $18.5$ |
| KBQA-Agent（我们的） | $19$K | $3$B | $2.9$ | $38.4$ |

(b) 知识库

表格 B.1：我们精心设计的基准更准确地反映了现实世界的复杂性，从而提供了更有效的语言代理评估。Aggr.表示聚合函数。

在表格 [B.1](https://arxiv.org/html/2402.14672v2#A2.T1 "表格 B.1 ‣ 附录 B 基准统计 ‣ 大型语言模型的中间件：工具对复杂环境中的语言代理至关重要") 中，我们展示了Bird和KBQA-Agent的统计数据，这是我们为评估选择的工具。与文本到SQL解析和KBQA的既定基准相比，Bird和KBQA-Agent展现出显著更大的复杂性，使它们更适合用于评估语言代理的能力。

## 附录 C 提示

![参考说明](img/f1f8a93970cfdff6701be87714ecd308.png)

图 C.1：使用数据库工具的说明。工具描述已省略。

使用数据库工具的说明和示范见图 [C.1](https://arxiv.org/html/2402.14672v2#A3.F1 "图 C.1 ‣ 附录 C 提示 ‣ 面向大型语言模型（LLM）的中间件：工具在复杂环境中的语言代理中的作用")。请注意，我们在提示中还包括了 API 文档中的数据库模式信息，这里未显示。此设计选择已成为使用 LLM 进行文本到 SQL 解析的常见做法，如 Tai 等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib39)）；Sun 等人（[2023](https://arxiv.org/html/2402.14672v2#bib.bib38)）。使用知识库工具的说明和示范见图 [C.2](https://arxiv.org/html/2402.14672v2#A3.F2 "图 C.2 ‣ 附录 C 提示 ‣ 面向大型语言模型（LLM）的中间件：工具在复杂环境中的语言代理中的作用")。候选选择的说明和示范，适用于解耦生成中的知识库见图 [C.3](https://arxiv.org/html/2402.14672v2#A3.F3 "图 C.3 ‣ 附录 C 提示 ‣ 面向大型语言模型（LLM）的中间件：工具在复杂环境中的语言代理中的作用")。此外，我们还展示了用于知识库实验的输入示例，见第 [5.4](https://arxiv.org/html/2402.14672v2#S5.SS4 "5.4 作为中间件层的工具 ‣ 5 实验 ‣ 面向大型语言模型（LLM）的中间件：工具在复杂环境中的语言代理中的作用") 章节。对于第 [5.4](https://arxiv.org/html/2402.14672v2#S5.SS4 "5.4 作为中间件层的工具 ‣ 5 实验 ‣ 面向大型语言模型（LLM）的中间件：工具在复杂环境中的语言代理中的作用") 章节中用于数据库的输入，我们严格遵循通过 API 文档加示例行的标准提示方式，参考 Li 等人（[2023a](https://arxiv.org/html/2402.14672v2#bib.bib19)）；Rajkumar 等人（[2022](https://arxiv.org/html/2402.14672v2#bib.bib31)）。

![参见标题](img/2e10f74fde84ca1f8196d90b7ebb23cc.png)

图 C.2：使用知识库工具的说明和单次示范。工具的描述已省略。

![参见标题](img/7f0851886619bb5abee62adad1296e52.png)

图 C.3：在解耦生成中选择候选动作的提示，用于知识库（KB）。

![参见标题](img/bfdb3c3fc59f8d616dccb70f7c9bc783.png)

图 C.4：问题“Handel: Messiah（都柏林版，1742年）中哪首歌是最长的？”的输入，包含从知识库中采样的 $10$ 个三元组，见第 [5.4](https://arxiv.org/html/2402.14672v2#S5.SS4 "5.4 作为中间件层的工具 ‣ 5 实验 ‣ 面向大型语言模型（LLM）的中间件：工具在复杂环境中的语言代理中的作用") 章节。

## 附录 D 额外结果

|  | 平均输入令牌 | 平均时间（秒） | 平均轮次 | 可感知行数 | EX（100个问题） |
| --- | --- | --- | --- | --- | --- |
| 中间件（错误反馈） | $14\,257.2$ | $15.5$ | $7.1$ | $\infty$ | $39.0$ |
| API 文档提示 | $1296.8$ | $2.5$ | $1.0$ | $0$ | $12.0$ |
| API 文档提示（10行） | $4635.5$ | $2.9$ | $1.0$ | $10$ | $22.0$ |
| API文档提示（20行） | $8459.4$ | $2.9$ | $1.0$ | $20$ | $9.0$ |

(a) 表 1

|  | 平均输入标记数 | 平均时间（秒） | 平均轮次 | 可感知三元组 | F1 |
| --- | --- | --- | --- | --- | --- |
| 中间件（错误反馈） | $25\,140.7$ | $22.1$ | $9.8$ | $\infty$ | $55.1$ |
| 中间件（解耦生成） | $23\,331.8$ | $20.9$ | $9.3$ | $\infty$ | $59.3$ |
| 盘古-Chat | $4675.7$ | $67.3$ | $4.5$ | $\infty$ | $27.1$ |
| 直接提示（200三元组） | $7654.5$ | $3.1$ | $1.0$ | $200$ | $0.8$ |

(b) 表 2

表D.2：不同方法的平均运行时间和输入标记数。请注意，现有的研究并未解决处理大规模数据库内容的挑战，这使得现有方法仅能感知少量行数据。这是文献中的一个关键空白，我们旨在填补这一空白。

| 设置 | gpt-3.5-turbo-0613 | gpt-4-0613 |
| --- | --- | --- |
| w/ all | $14.0$ | $38.0$ |
| w/o find_columns_containing_value | $14.0$ | $38.0$ |
| w/o find_columns_containing_value_fuzzy | $12.0$ | $34.0$ |
| w/o get_distinct_values | $10.0$ | $32.0$ |
| w/o is_value_in_column | $14.0$ | $36.0$ |
| w/o all four | $8.0$ | $22.0$ |

表D.3：我们在数据库任务中使用不同工具的消融研究。

### D.1 与DIN-SQL和DAIL-SQL的比较

除了API文档提示外，我们还与两个强基线进行了比较，这些基线的源代码是公开的：DIN-SQL Pourreza和Rafiei（[2023](https://arxiv.org/html/2402.14672v2#bib.bib28)）以及DAIL-SQL Gao等（[2024](https://arxiv.org/html/2402.14672v2#bib.bib7)）来自Bird的排行榜。请注意，在他们的原始提交中，他们仅在“oracle知识”设置下进行评估。为了确保与中间件在相同的“无oracle知识”设置下进行公平比较，我们对他们的源代码进行了适当调整，并最小化了改动。具体而言，他们的流程设计中的某些模块需要多个上下文演示，我们也保留了这些内容。因此，我们对DIN-SQL使用了6-shot，对DAIL-SQL使用了7-shot，遵循他们原始的代码库。特别是，我们从Bird的开发集随机抽取了$100$个问题，并在这些问题上进行了评估。结果见表[D.4](https://arxiv.org/html/2402.14672v2#A4.T4 "Table D.4 ‣ D.3 Ablation for Tools ‣ Appendix D Additional Results ‣ Middleware for LLMs: Tools Are Instrumental for Language Agents in Complex Environments")。

### D.2 效率分析

我们进一步探讨了不同方法的效率（例如，平均标记数和平均运行时间），以便深入理解。对于数据库（DB），我们将中间件与最直接的单轮基线API文档提示进行比较。对于知识库（KB），我们将中间件与盘古（Pangu）进行比较，因为其他所有基线方法的表现明显较差。具体结果见表[D.2](https://arxiv.org/html/2402.14672v2#A4.T2 "Table D.2 ‣ Appendix D Additional Results ‣ Middleware for LLMs: Tools Are Instrumental for Language Agents in Complex Environments")。

### D.3 工具的消融实验

对于 KBQA，工具的消融实验是微不足道的，因为每个工具都会确定性地对最终的 KB 查询做出贡献。例如，移除 get_relations 或 get_neighbors 会导致所有问题的性能为 0%，而移除 count 则会导致计数类问题的性能为 0%。因此，我们仅对 text-to-SQL 解析进行消融实验。具体来说，我们首先根据 GPT-4 的预测，确定了四个最常用的工具：find_columns_containing_value、find_columns_containing_value_fuzzy、get_distinct_values 和 is_value_in_column。在前 100 个问题中的 50 个问题上（出于预算考虑），我们在表 [D.3](https://arxiv.org/html/2402.14672v2#A4.T3 "Table D.3 ‣ Appendix D Additional Results ‣ Middleware for LLMs: Tools Are Instrumental for Language Agents in Complex Environments") 展示了结果。一个有趣的观察是，当只移除一个工具时，性能依然表现出较强的鲁棒性，这得益于我们工具设计中的冗余性。然而，当所有四个工具都被移除时，性能显著下降。

|  | gpt-3.5-turbo-0613 | gpt-4-0613 |
| --- | --- | --- |
| Middleware (0-shot) | $16.0$ | $39.0$ |
| API Doc Prompt (0-shot) | $6.0$ | $12.0$ |
| DIN-SQL (6-shot) | - | $22.0$ |
| DAIL-SQL (7-shot) | $8.0$ | $11.0$ |

表 D.4：在没有 oracle 知识的 Bird 数据集上更多基线的结果。
