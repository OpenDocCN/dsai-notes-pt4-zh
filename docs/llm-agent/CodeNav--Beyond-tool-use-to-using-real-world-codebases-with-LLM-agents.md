<!--yml

分类：未分类

日期：2025-01-11 12:31:42

-->

# CodeNav：超越工具使用，利用LLM代理使用真实世界的代码库

> 来源：[https://arxiv.org/html/2406.12276/](https://arxiv.org/html/2406.12276/)

Tanmay Gupta*  Luca Weihs*  Aniruddha Kembhavi

来源：Allen人工智能研究所

[https://codenav.allenai.org](https://codenav.allenai.org)

**等贡献**

###### 摘要

我们介绍了CodeNav，一个LLM代理，能够浏览并利用先前未见过的代码库来解决用户查询。与需要通过在LLM上下文中手动描述“注册”所有相关工具的工具使用型LLM代理不同，CodeNav会自动索引并在目标代码库中搜索代码块，找到相关的代码片段，导入它们，并利用它们迭代生成带有执行反馈的解决方案。为了突出CodeNav的核心功能，我们首先展示了三个案例研究，展示了如何使用CodeNav在三个不同的代码库中解决复杂的用户查询。接着，在三个基准测试中，我们定量比较了代码使用（仅能访问目标代码库）与工具使用（可以访问所有工具名称和描述的特权）之间的效果差异。最后，我们研究了不同种类的工具和库描述对代码使用性能的影响，并调查了代理程序查看源代码而非自然语言描述代码的优势。所有代码都将在宽松许可证下公开。

## 1 引言

今天，工具使用已成为使LLM与外部系统或程序交互，以完成特定领域任务的事实标准方法(Gupta 和 Kembhavi, [2023](https://arxiv.org/html/2406.12276v1#bib.bib7); Sur’is 等, [2023](https://arxiv.org/html/2406.12276v1#bib.bib23); Wu 等, [2023](https://arxiv.org/html/2406.12276v1#bib.bib28); Yang 等, [2023](https://arxiv.org/html/2406.12276v1#bib.bib29); Wang 等, [2023](https://arxiv.org/html/2406.12276v1#bib.bib24))。在这种工具使用范式中，首先通过将功能列表或更广义的代码片段的描述（可能带有使用示例）添加到LLM的上下文中，“注册”这些工具。然后，针对用户查询，LLM生成这些工具的调用，并在外部环境中执行，以解决用户的任务。

虽然工具使用能带来新的能力，但它也通过将大语言模型（LLMs）限制在仅能调用少数精心设计的功能或API调用上，从而限制了其表达能力。随着LLMs在代码理解和生成方面的能力越来越强（Wang等人，[2024](https://arxiv.org/html/2406.12276v1#bib.bib25); Chen等人，[2021a](https://arxiv.org/html/2406.12276v1#bib.bib3); GitHub，[2024](https://arxiv.org/html/2406.12276v1#bib.bib6); Jimenez等人，[2024](https://arxiv.org/html/2406.12276v1#bib.bib12)），我们认为是时候从*工具使用*转向*代码使用*，*即*从精心设计和注册的工具转向一种环境，其中LLM代理直接读取、导入并使用任何给定代码库的源代码¹¹1我们将代码库、库和代码仓库这些术语互换使用，以解决用户查询。

一个有效的代码使用代理必须能够识别并使用代码库中正确的代码片段（函数、类、常量，*等等*）来解决给定的查询。这种对代码的自由形式搜索之所以可行，是因为一个简单的观察：设计良好的库是由人类为人类编写的。这些库通常做出重要的领域特定假设，使用有意义的抽象和变量名称，并将代码组织在文件和目录中，使得相关功能容易发现并简洁地记录。与工具使用中对每个文件中的每个函数或类进行描述不同，代码使用利用这种结构来搜索所需的代码片段。代码使用还可能受益于一种高层次的库描述，这种描述能够暴露代码库固有的结构。这可以通过多种方式实现：*例如*，突出重要的目录、文件或入口点，描述复杂文件的内容和目的，或阐明库的抽象和假设。这种库描述通常已经存在于README文件中，或者可以手动提供，或通过基于规则的解析或LLM自动生成。

我们提出了CodeNav，一个单一代理、多环境交互框架（见图[1](https://arxiv.org/html/2406.12276v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")），其中一个代理在给定的代码库中导航，找到解决用户查询所需的代码片段。给定用户查询和高级库描述，CodeNav在搜索代码库以找到有用的代码片段和生成部分解决方案代码（该代码导入、实例化或调用在检索到的结果中定义的相关变量、类或函数）之间进行迭代。在这个迭代过程中，代理检查执行结果，修复错误（如果有的话），进行搜索以解决歧义，并逐步构建查询的解决方案。多环境设置使得通过添加新的环境（*例如*，\mintinlinepythonterminal）以及其支持的操作（*例如*，\mintinlinepythonbash命令）和对这些操作的响应（*例如*，\mintinlinepythonSTDOUT）来扩展系统的能力变得容易。

为了评估CodeNav，我们首先通过量化工具使用与代码使用之间的差距，基于三个现有的工具使用基准：m&m’s (Ma et al., [2024](https://arxiv.org/html/2406.12276v1#bib.bib16))、M³ToolEval (Wang et al., [2024](https://arxiv.org/html/2406.12276v1#bib.bib25))和API-Bank (Li et al., [2023b](https://arxiv.org/html/2406.12276v1#bib.bib15))。为了使这些基准适用于代码使用，我们将包含工具实现的文件作为目标代码库。工具使用形成了代码使用的上限，因为工具使用代理可以访问这些基准所提供的全面的、手工制作的信息。令人惊讶的是，尽管CodeNav没有访问这些特权信息，但我们发现它在这些评估中与工具使用相比具有竞争力。尽管这些结果令人印象深刻，但它们尚未展示代码使用的全部潜力。为此，我们呈现了三个使用不同代码库的定性案例研究。在这些案例研究中，我们发现CodeNav能够执行复杂的多步查询，恢复执行错误，进行迭代搜索以更好地理解代码片段的使用，并根据用户指示格式化结果以可视化计算（*例如*，作为网页）。

我们认为，代码使用不仅仅是生成基于另一个代码库的代码的一个小众应用。我们设想未来的情景，其中代码使用是 LLM 如何发现并使用特定领域工具的方式，而不对工具的开发或编写方式施加任何限制（*例如*，作为一系列简单函数），也无需任何干预（*例如*，手动描述和注册工具）来使 LLM 能够使用这些工具。我们的研究结果暗示了一个未来：只需一个代码库！我们突出了六个关键贡献。（1）我们为 LLM 代理引入了一种新的代码使用范式，推动其从工具使用超越，直接使用现实世界的代码库来解决复杂的用户查询。（2）我们提出了 CodeNav，将代码使用定义为单个 LLM 代理与有状态检索和代码执行环境之间的多步骤交互。（3）在三个工具使用基准（m&m’s、M³ToolEval 和 API-Bank）上，我们发现，从代码使用到工具使用的上限存在最小差距，无需繁琐的工具注册。（4）我们研究了库或工具描述丰富度对代码使用性能的影响。（5）我们调查了将源代码作为检索结果的一部分与仅仅获取函数签名或文档字符串的优势。（6）我们展示了三个案例研究，证明了代码使用代理在使用真实世界代码库解决复杂查询方面的潜力。我们的代码将在宽松许可证下开源。

![请参阅说明文字](img/dd274a05800e228aa8dc66b85e2ab683.png)

图 1：CodeNav 的单一代理、多环境交互协议。给定用户查询、代码库的简要描述（*库描述*）以及交互历史，LLM 代理生成一个 \mintinlinepythonaction，其中包括 *思考*、行动 *类型* 和行动 *内容*。该行动会在目标环境中执行（由行动 *类型* 确定），产生一个 \mintinlinepythonresponse。当前步骤的交互，包括 \mintinlinepythonaction-\mintinlinepythonresponse 配对，会被附加到交互历史中，作为 LLM 生成下一个行动的上下文。

## 2 相关工作

使用工具与LLMs。工具使用指的是LLM识别并调用适当的工具或功能来解决用户描述的任务。无需训练的工具使用方法首先需要将工具“注册”到LLM中。工具注册技术包括通过描述工具进行*描述性*注册（*例如*，实现工具的类或函数的签名和文档字符串）（Hsieh等，[2023](https://arxiv.org/html/2406.12276v1#bib.bib8); Sur’is等，[2023](https://arxiv.org/html/2406.12276v1#bib.bib23)）或通过提供相似任务描述和相应工具调用的上下文示例进行*指示性*注册（Gupta和Kembhavi，[2023](https://arxiv.org/html/2406.12276v1#bib.bib7)）。基于训练的方法涉及在指令和工具调用对上对LLM进行微调（Qin等，[2024](https://arxiv.org/html/2406.12276v1#bib.bib22)）。在本研究中，我们专注于无需训练的方法来实现工具使用和代码使用。

用于工具使用的检索。将无需训练的方法扩展到大量工具面临挑战，原因是LLM的上下文窗口有限（尽管不断增加）。为了解决这一问题，一种方法是从库中检索与任务相关的工具描述或使用示例。例如，Hsieh等人（[2023](https://arxiv.org/html/2406.12276v1#bib.bib8)）采用TF-IDF搜索来检索相关的工具文档。DocPrompting（Zhou等，[2023](https://arxiv.org/html/2406.12276v1#bib.bib32)）探索了稀疏和密集文档检索用于代码生成。ART（Paranjape等，[2023](https://arxiv.org/html/2406.12276v1#bib.bib21)）使用跨任务示例检索进行多步推理和工具使用。EcoAssistant（Zhang等，[2023](https://arxiv.org/html/2406.12276v1#bib.bib31)）将过去成功解决用户查询的解决方案保存在数据库中，对于新查询，检索数据库中类似查询的解决方案作为示范。与之前检索文档的工作不同，我们的CodeNav代理直接使用布尔Elasticsearch查询检索源代码（*例如*（type: CLASS）AND（text: ObjectDetection））。

工具使用的提示策略。对于需要多步工具使用解决方案的用户查询，智能体必须同时具备规划能力（决定使用哪些工具以及何时使用）和正确调用工具的能力。为了提高规划能力，诸如“思维链”（Wei 等，[2022](https://arxiv.org/html/2406.12276v1#bib.bib26)）的提示策略促使大型语言模型（LLM）在实际解决方案之前，先给出一个描述逐步计划的*思维*过程。ReAct（Yao 等，[2023](https://arxiv.org/html/2406.12276v1#bib.bib30)）则交替进行*思维*、*行动*（工具调用）和*观察*（执行动作或反馈结果）。CodeAct（Wang 等，[2024](https://arxiv.org/html/2406.12276v1#bib.bib25)）通过允许*行动*为自由格式代码，扩展了ReAct的应用。与CodeAct不同，CodeNav采用了一个单智能体、多环境框架，其中智能体不仅可以搜索代码，还能执行代码。此外，CodeAct操作于工具使用领域，在该领域中，所需工具会提前注册。

代码生成。虽然并非所有的工具使用方法都要求LLM生成可执行代码（Khot 等，[2022](https://arxiv.org/html/2406.12276v1#bib.bib13)）（而是生成函数名和参数，并使用自定义解释器），但工具使用可以通过自由格式代码生成来实现（Wang 等，[2024](https://arxiv.org/html/2406.12276v1#bib.bib25)；Ma 等，[2024](https://arxiv.org/html/2406.12276v1#bib.bib16)）。LLM在代码生成方面的应用已经被广泛探讨，包括在代码编辑器中进行代码补全（GitHub，[2024](https://arxiv.org/html/2406.12276v1#bib.bib6)）、从文档字符串生成函数（Chen 等，[2021b](https://arxiv.org/html/2406.12276v1#bib.bib4)）、在代码库中编辑文件以修复GitHub问题（Jimenez 等，[2024](https://arxiv.org/html/2406.12276v1#bib.bib12)），甚至从单一自然语言指令生成包含多个文件的完整代码库（Osika，[2023](https://arxiv.org/html/2406.12276v1#bib.bib20)）。

反馈与修正。随着任务变得更加复杂，单次LLM调用可能只会产生部分或部分正确的解决方案。对于这样的任务，*代理工作流*启用一个闭环系统（Wu等， [2023](https://arxiv.org/html/2406.12276v1#bib.bib28); Wang等， [2023](https://arxiv.org/html/2406.12276v1#bib.bib24)），在这个系统中，LLM代理迭代地产生一个中间解决方案，接收关于解决方案的反馈，然后继续修正任何错误或生成下一个步骤。在代码生成和工具使用的上下文中，反馈可能包括执行输出，如变量值和引发的异常，测试用例的输出（Huang等， [2023](https://arxiv.org/html/2406.12276v1#bib.bib9)），静态分析工具（如代码格式检查和类型检查）的输出，以及来自人类或LLM的反馈（Madaan等， [2023](https://arxiv.org/html/2406.12276v1#bib.bib17)）。最近的研究还探讨了使用LLM模拟执行反馈（Li等， [2023a](https://arxiv.org/html/2406.12276v1#bib.bib14); Ni等， [2024](https://arxiv.org/html/2406.12276v1#bib.bib18)）。

## 3 CodeNav

### 3.1 概述

我们将使用LLM的代码形式化为一个单一代理、多环境交互框架，该框架包括有状态的代码检索和代码执行环境（见图[1](https://arxiv.org/html/2406.12276v1#S1.F1 "图1 ‣ 1 引言 ‣ CodeNav：超越工具使用，使用LLM代理与真实世界代码库交互")）。该代理被提供一个简短的高级库描述（*例如*：重要的目录或文件路径、复杂文件的内容和目的描述、库所做的关键抽象和假设、*等*）以及用户要解决的查询。然后，代理将在多个回合中与这些环境进行交互。每次交互由代理的\mintinlinepythonaction和环境的\mintinlinepythonresponse组成。每个\mintinlinepythonaction包括：（i）用于推理链式思维的*思考*（Wei等， [2022](https://arxiv.org/html/2406.12276v1#bib.bib26); Yao等， [2023](https://arxiv.org/html/2406.12276v1#bib.bib30)），（ii）指定代理希望作用的环境的*动作类型*（*例如*：代码、搜索、*等*），以及（iii）在所选环境中执行的*动作内容*。根据动作类型，动作内容被路由到适当的环境，环境执行该内容，更新其状态并生成\mintinlinepythonresponse。过去交互的历史被提供给代理作为上下文，以生成下一个动作。交互会继续，直到达到用户指定的最大交互次数，或者代理执行\mintinlinepythondone动作。接下来我们描述环境、动作和响应的细节。

### 3.2 环境

对于代码使用，代理必须能够执行两个基本功能：（i）在目标代码库中搜索或发现相关的代码片段；（ii）生成下一部分代码解决方案，该部分代码导入、实例化或调用所需的类或函数。在 CodeNav 中，代理通过在以下某个环境中采取行动来执行这些功能。

检索环境。该环境作为接口，连接到搜索索引，用于通过基于规则的重新排序从目标代码库中获取代码片段，并具有持久记忆功能，以避免重复出现先前的检索结果。我们解析整个代码库并将所有函数、类、导入语句、赋值语句等作为独立文档进行索引。我们使用 \mintinlinepythonElasticsearch (\mintinlinepythonES) 实现索引，其中包括代码字符串、代码类型（函数、类、赋值、导入等）、文件路径和行号等字段（Elastic, [2024](https://arxiv.org/html/2406.12276v1#bib.bib5)）。代理可以使用 \mintinlinepythonsearch 操作类型向此索引发出搜索查询。在执行此操作时，环境首先将操作内容作为搜索查询发送到 \mintinlinepythonES 索引，丢弃任何在本集期间已被先前搜索结果检索到的匹配项，然后根据启发式规则重新排序结果（*例如*，优先考虑函数和类，降低赋值和导入语句的优先级）。最后，前 $k$ 个结果会被添加到环境的持久记忆中，并作为环境响应返回。再次执行相同的操作时，将显示下一个最优的 $k$ 个匹配结果。

执行环境。类型为 \mintinlinepythoncode 的操作会被路由到 \mintinlinepythonPython 执行环境。²²2 在本研究中，为了简化起见，我们仅考虑 Python 代码生成，但 CodeNav 并未限制扩展至其他语言。在每一集的开始，环境会用一个空的 \mintinlinepythonglobal_variables 字典初始化。每个代码块都在这些全局变量的作用域内执行（使用 \mintinlinepythonexec(code_str, global_vars)），对全局命名空间的任何更改（*即*，修改或删除现有变量，或创建新变量）都会反映在这个字典中。在执行之前，环境可选择对代码块进行 lint 检查（使用 \mintinlinepythonflake8）、类型检查（使用 \mintinlinepythonmypy）和格式化（使用 \mintinlinepythonblack）。标准输出（\mintinlinepythonstdout）、更新后的 \mintinlinepythonglobal_vars 中的变量（新变量或字符串表示已更改的变量）以及任何错误（执行、lint 检查或类型检查）都会作为响应的一部分返回。

完成和代码摘要环境。此外，我们创建了一些辅助环境。*完成环境* 对 \mintinlinepythonnull 响应 \mintinlinepythondone 动作，该动作标志着该回合的结束。*代码摘要环境* 将 \mintinlinepythoncode_summary 动作的内容视为代理在该回合中生成的代码解决方案的清理摘要，并将其保存以便后续访问。

### 3.3 代理动作

CodeNav 代理通过使用一个包含 3 个组件的 \mintinlinepythonaction 与环境进行交互：*thought*、*type* 和 *content*，参见图 [1](https://arxiv.org/html/2406.12276v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")（*右侧*）。该代理所使用的 LLM 被提示仅生成结构化的 XML 格式输出，并遵守一组规则（*例如*，*thought* 和 *type* 必须始终提供；*type* 必须是可用的行动类型之一；*等等*）。在将 \mintinlinepythonaction 路由到目标环境之前，我们会检查其有效性。如果 \mintinlinepythonaction 被发现无效，将返回一个 \mintinlinepythonInvalidAction 响应，包含违反规则的描述。代理可以使用此违反描述在下一步中修复 \mintinlinepythonaction。

![参见图注](img/69946d0ebdbdb1ae033d917de335266f.png)

图 2：运行 CodeNav 代理与 CodeNav 代码库。

### 3.4 环境响应

每个动作都会引发来自目标环境的响应，如果该动作在目标环境中无效，则会返回 \mintinlinepythonInvalidAction 响应。CodeNav 中的每个响应都实现为一个数据类，具有 \mintinlinepythonformat() 方法，指定如何将响应数据序列化为文本字符串，并将其包含在代理的上下文中，以便预测下一个动作。

检索响应。给定一个搜索查询（*即*，一个\mintinlinepythonsearch操作的*内容*），检索环境从搜索索引中返回与搜索查询匹配的文档列表（包含来自目标代码库的代码块以及元数据）。检索响应的\mintinlinepythonformat()方法实现了以下问题的回答：从这些检索到的代码块中，应该向代理展示什么内容？一方面，代理可以通过阅读源代码（包含函数签名、参数类型、输出和实现细节），而不是阅读不准确、不完整、过时或完全缺失的人类编写的类或函数描述或\mintinlinepythondocstring，来更好地理解如何使用一个类或函数。另一方面，展示每个检索到的代码块的所有实现细节会导致需要处理的上下文令牌数量激增，进而增加LLM的负担。为了平衡这一点，我们首先检索最多$M$（$=100$）个匹配的文档。从中，我们展示前$K$（$=3$）个匹配项，包括源代码和元数据。对于大型代码库，如果没有在搜索查询中给出精确的函数或类名称，这通常不足以显示目标代码块。因此，我们还展示最多$P$个类或函数的*原型*（签名和文件名）在其余检索结果中。此外，对于前$K$个匹配项，我们使用GPT-4为前3个检索项生成\mintinlinepythondocstrings，并展示源代码和生成的docstring中较短的部分与函数签名。

执行响应。给定一个\mintinlinepythoncode操作，Python执行环境执行*内容*并生成响应。响应的\mintinlinepythonformat()方法序列化标准输出（\mintinlinepythonstdout）、执行过程中变化的变量（显示为变量名及其值的字符串表示）、执行错误（如果有）以及（可选的）代码检查、类型检查和格式化错误。错误信息包含指向产生错误的代码行的引用，帮助定位错误。 \mintinlinepythonstdout和变化的变量使得函数调用的检查变得容易，但可能会变得非常长（*例如*，打印一个大型数组时），因此会截断到最大字符数。我们展示\mintinlinepythonstdout的开始和结束以及变量值的开头。

## 4个案例研究

虽然我们在第[5](https://arxiv.org/html/2406.12276v1#S5 "5 实验 ‣ CodeNav：超越工具使用，使用真实世界代码库与LLM代理")节中定量评估了CodeNav在工具使用基准上的表现，但这些基准不足够复杂，无法突出代码使用相较于工具使用的优势。因此，我们通过三个案例研究展示了CodeNav在使用不同代码库解决复杂查询时的强大能力。第一个案例研究（第[4.1](https://arxiv.org/html/2406.12276v1#S4.SS1 "4.1 CodeNav on CodeNav ‣ 4 案例研究 ‣ CodeNav：超越工具使用，使用真实世界代码库与LLM代理")节）中，图[2](https://arxiv.org/html/2406.12276v1#S3.F2 "图2 ‣ 3.3 代理操作 ‣ 3 CodeNav ‣ CodeNav：超越工具使用，使用真实世界代码库与LLM代理")展示了整个过程。对于其他两个案例研究，我们在图[3](https://arxiv.org/html/2406.12276v1#S4.F3 "图3 ‣ 4.1 CodeNav on CodeNav ‣ 4 案例研究 ‣ CodeNav：超越工具使用，使用真实世界代码库与LLM代理")中展示了输入和输出，并将完整过程作为补充材料提供。对于所有案例研究，我们在附录中提供了库的描述（附录[C](https://arxiv.org/html/2406.12276v1#A3 "附录C 库和工具描述 ‣ CodeNav：超越工具使用，使用真实世界代码库与LLM代理")）。请参见附录表[6](https://arxiv.org/html/2406.12276v1#Ax1.T6 "表6 ‣ 附录 ‣ CodeNav：超越工具使用，使用真实世界代码库与LLM代理")，了解这些案例研究中使用的代码库的规模和复杂性以及我们的定量实验（*例如*，搜索transformers库需要在3,475个文件中的50,508个代码片段上进行搜索）。

### 4.1 CodeNav on CodeNav

我们假设有一位研究人员，可能在阅读完本文后，想使用CodeNav通过transformers库来回答一个查询（Wolf等人，[2020](https://arxiv.org/html/2406.12276v1#bib.bib27)）。然而，我们并没有使用研究人员，而是使用了一个CodeNav代理；*即*，一个CodeNav代理利用CodeNav仓库实例化另一个CodeNav代理来使用transformers回答特定的查询。这个示例有两个目标：（1）提供一个使用我们代码库的教学示例；（2）展示CodeNav的零-shot能力，因为我们可以保证底层的LLM（GPT-4）并没有在我们的代码库上进行训练。

在这个案例研究中，我们的用户查询包含7个步骤，分为2个不同的部分（参见图[2](https://arxiv.org/html/2406.12276v1#S3.F2 "Figure 2 ‣ 3.3 Agent Actions ‣ 3 CodeNav ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")中的用户查询）。步骤1-4指定了创建和运行示例的指令，而步骤5-7包含了可视化交互结果的指令。在步骤1-4中，用户要求CodeNav首先使用\mintinlinepythonOpenAICodenavAgent创建一个代理，并使用\mintinlinepythonPythonCodeEnv、\mintinlinepythonRetrievalEnv和\mintinlinepythonDoneEnv实例化各种环境，并指定用于检索的Elasticsearch主机和索引名等参数。接着，查询要求代理创建一个用于解决另一个查询（*在原始查询中！*）的示例，使用\mintinlinepythontransformers。这个“子查询”要求代理使用\mintinlinepythonfacebook/detr-resnet-101模型在物体检测管道中检测图像中的狗（由文件路径指定），在图像上添加红色检测框，并将图像存储在变量\mintinlinepythondetected_dogs中。该子查询还要求代理将检测坐标和分数作为pandas数据框存储在变量\mintinlinepythondetection_coords_and_scores中。完整查询的第一部分以要求代理在最多10步内运行示例结束。步骤5-7指定了如何可视化交互。具体来说，我们要求代理：（i）将交互以数据框的形式列出，包含行动类型和思维列；（ii）将\mintinlinepythondetected_dogs图像保存为PNG格式，存储在指定的文件路径；以及（iii）打印\mintinlinepythondetection_coords_and_scores。

上述内容的CodeNav示例可以在图[2](https://arxiv.org/html/2406.12276v1#S3.F2 "Figure 2 ‣ 3.3 Agent Actions ‣ 3 CodeNav ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")中找到。代理首先搜索关于\mintinlinepythonOpenAICodenavAgent类和环境（A1, R1, A2, R2）的信息，然后尝试通过代码（A3）实例化它们。由于错误使用了\mintinlinepythonRetrievalEnv初始化器（R3），该代码会导致错误。接着，代理会搜索额外的信息以解决此错误（A4, R4），最终成功运行了一个新的CodeNav代理，使用了transformers（A9, R9）。最后，代理打印并保存了请求的输出。

![参见说明文字](img/db96bd9c555559e77825342f89b95ffc.png)

图3：CodeNav通过代码使用统一“代理性”应用程序。两个案例研究：（*上图*）视觉推理与图像编辑代理；（*下图*）信息收集代理。这些应用程序仅通过改变目标代码库搜索索引和高级库描述来启用。

### 4.2 多模态处理与推理

在我们提出的代码使用方案中，只要目标代码库中提供了多模态处理功能，多模态任务与仅文本任务是相同的。我们通过一个图像编辑任务来展示这一点，该任务需要视觉推理来定位需要编辑的区域。我们使用m&m’s代码库，因为它包含了用于检测、分割、问答等的计算机视觉工具。在这个案例研究中，请参见图[3](https://arxiv.org/html/2406.12276v1#S4.F3 "Figure 3 ‣ 4.1 CodeNav on CodeNav ‣ 4 Case Studies ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")（顶部），代理需要找到并突出显示一个正在打电话、戴眼镜的人。我们的查询首先指定了定位编辑区域的步骤。具体来说，代理首先被指示进行图像分割并选择人物区域。然后，对于每个人物，代理应通过裁剪人物边界框的上三分之一来放大面部。对于每个面部，代理必须使用视觉问答来验证该人是否戴眼镜*并且*正在打手机。为了可视化这些预测，代理被指示将这些面部裁剪图及其预测结果保存为HTML表格。最后，代理被指示通过颜色突出效果来突出显示同时满足两个属性的人物。我们观察到，CodeNav不仅熟练地使用多模态模型，还利用输出进行视觉推理。此外，这个案例研究突出了CodeNav在生成可供人类解释的中间输出方面的能力。

### 4.3 研究助手

想象一个代理，它负责整理某个主题的阅读材料，包括新闻文章、博客帖子和研究论文，然后将这些内容通过电子邮件发送给你。对于CodeNav来说，这只是一个简单的任务，利用一个提供必要功能的代码库来查询网络上的知识来源并发送电子邮件。一个这样的代码库是PhiData (phi，[2024](https://arxiv.org/html/2406.12276v1#bib.bib1))。在这个案例研究中（见图[3](https://arxiv.org/html/2406.12276v1#S4.F3 "Figure 3 ‣ 4.1 CodeNav on CodeNav ‣ 4 Case Studies ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents"), 底部），我们向代理发出查询，要求其整理关于“Alphafold-3”的阅读材料，内容包括来自维基百科的定义、一系列新闻文章以及关于“深度学习在蛋白质折叠中的应用”的论文列表。此外，我们的查询还指定了各种展示要求：*例如*，对于每篇新闻文章，我们要求代理显示文章的来源、标题、链接以及前140个字符。最后，我们要求代理将这些信息写入HTML和Markdown格式的文档中。然后，文本文件的内容会以指定的主题发送到用户的电子邮件地址。这个案例研究展示了CodeNav的多功能性，只需提供适当的代码库，就可以创建专门的代理。在这里，CodeNav通过使用PhiData中内建的功能，充当了一个研究助手，查询维基百科、DuckDuckGo新闻和arXiv。

## 5 实验

现在，我们展示在三个工具使用基准上的定量结果：（1）m&m’s，它要求进行多步骤规划，涉及33个多模态工具，如视觉/语言工具（Ma等，[2024](https://arxiv.org/html/2406.12276v1#bib.bib16)）；（2）API-Bank，它通过调用73个API中的任何一个，在沙箱环境中管理用户状态（Li等，[2023b](https://arxiv.org/html/2406.12276v1#bib.bib15)）；以及（3）M³ToolEval，它包含“82个由人工策划的任务”，需要多个工具、调用和交互（Wang等，[2024](https://arxiv.org/html/2406.12276v1#bib.bib25)）。在所有实验中，我们报告了3次独立评估的平均性能；报告的$\pm$值对应于评估中的标准偏差的两倍。由于这些基准并非专为评估代码使用代理而设计（*例如*，API-Bank使用JSON API调用），这就需要对我们如何在这些基准上评估CodeNav进行一些调整。有关这些细节以及我们的评估指标描述，请参见附录[A](https://arxiv.org/html/2406.12276v1#A1 "Appendix A Experiment details ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")。

### 5.1 如何在工具使用基准上比较代码使用与工具使用？

工具使用基准测试旨在测试大语言模型（LLM）调用一小组预注册工具的能力。由于这些基准中的工具相对简单，主要是包含人类编写描述的功能调用，因此在这些基准上，code-use 的表现上限受限于 tool-use。我们希望量化 code-use（需要源代码搜索）与 tool-use（不需要搜索，因为提供了工具名称/描述）之间的差距。表 [1](https://arxiv.org/html/2406.12276v1#S5.T1 "表 1 ‣ 5.1 如何在工具使用基准上比较 code-use 和 tool-use？ ‣ 5 实验 ‣ CodeNav: 超越 tool-use，使用真实世界代码库与 LLM 代理") 显示，在 m&m’s 和 API-Bank 上，code-use 的工具 f1 表现相似或稍低。在这两个基准测试上，code-use 在工具召回率上略有下降。这是直观的，因为 tool-use 提供了工具名称，而 code-use 则面临更具挑战性的任务——搜索以发现可用工具。在评估最终答案正确性的 M³ToolEval 上，code-use 的表现与 tool-use 相差仅两分。在所有数据集上，平均而言，code-use 需要比 tool-use 多大约 2 步的交互步骤来搜索所需工具。最后，尽管由于缺乏对可用工具的知识增加了不确定性，code-use 的表现方差仅略有增加。

表 1：即使没有工具提示，code-use 也与 tool-use 具有竞争力。

|  | m&m’s |  | M³ToolEval |  | API-Bank |
| --- | --- | --- | --- | --- | --- |
| 方法 | 精度 | 召回率 | f1 | 步骤 |  | 准确率 | 步骤 |  | 精度 | 召回率 | f1 | 步骤 |
| tool-use | 82.9 ± 4.5 | 81.7 ± 0.4 | 79.6 ± 2.3 | 4.9 ± 0.1 |  | 83.7 ± 2.8 | 6.6 ± 0.5 |  | 86.6 ± 0.8 | 93.6 ± 1.1 | 88.5 ± 0.7 | 3.4 ± 0.1 |
| code-use | 88.0 ± 6.1 | 78.2 ± 4.5 | 80.6 ± 5.1 | 7.2 ± 0.2 |  | 81.7 ± 4.9 | 7.8 ± 0.4 |  | 84.0 ± 0.7 | 89.3 ± 0.6 | 85.3 ± 0.3 | 5.3 ± 0.0 |

### 5.2 库描述是否足以支持工具使用？

表 2：m&m’s 上的描述去除实验。

| 工具描述 | 长度 | f1 | 步骤 |
| --- | --- | --- | --- |
| 无描述 | 0 | 74.1 ± 1.9 | 7.0 ± 0.1 |
| 工具名称 | 694 | 78.1 ± 3.5 | 6.7 ± 0.2 |
|    + 描述 | 3680 | 80.8 ± 0.4 | 6.9 ± 0.1 |
|     + 原型 | 4627 | 80.7 ± 5.0 | 6.1 ± 0.1 |
| 库描述 (CodeNav) | 2061 | 80.6 ± 5.1 | 7.2 ± 0.2 |

CodeNav 允许用户仅提供代码库或实现这些工具的库的高级描述，而不是详细列出每个工具或函数名称及其目的、输入和输出参数。在表 [2](https://arxiv.org/html/2406.12276v1#S5.T2 "表 2 ‣ 5.2 库描述是否足以支持工具使用？ ‣ 5 实验 ‣ CodeNav：超越工具使用，利用 LLM 代理使用真实世界代码库") 中，我们比较了库描述（附录中的图 [6](https://arxiv.org/html/2406.12276v1#A6.F6 "图 6 ‣ 附录 F 社会影响 ‣ CodeNav：超越工具使用，利用 LLM 代理使用真实世界代码库")）与没有任何描述的 CodeNav，以及具有三种“详细级别”的工具描述；最低级别仅包含工具名称，而最丰富的级别包含工具名称、描述和函数签名（附录中的图 [10](https://arxiv.org/html/2406.12276v1#A6.F10 "图 10 ‣ 附录 F 社会影响 ‣ CodeNav：超越工具使用，利用 LLM 代理使用真实世界代码库")）。由于在提示中提供工具细节有助于代理识别解决查询所需的工具并正确使用它们，我们发现随着工具细节的增加，工具 F1 值呈一致性增加。令人惊讶的是，库描述以不到一半的描述长度，达到了与最丰富工具描述相似的性能。尽管不需要描述每个工具的便利性是以步骤数的轻微增加为代价的，因为现在代理需要搜索并发现工具，而不是从上下文中回忆。

### 5.3 看源代码是否有助于代码使用？

表 3：M&M’s 搜索响应格式化消融。

| 检索响应 | F1 | 步骤数 |
| --- | --- | --- |
| 原型 | 80.0 ± 5.2 | 7.8 ± 0.1 |
| 代码 | 81.2 ± 2.3 | 7.3 ± 0.2 |
| 代码或文档字符串（CodeNav） | 80.6 ± 5.1 | 7.2 ± 0.2 |

编写良好的代码本身就是文档。因此，对于一个精通代码理解的代理来说，看到源代码中的实际实现细节（这可能还包括文档字符串）可以减轻手动注册工具的需求，并提供比仅仅函数签名或原型更多的信息。我们在表格[3](https://arxiv.org/html/2406.12276v1#S5.T3 "Table 3 ‣ 5.3 Does seeing the source code help code-use? ‣ 5 Experiments ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")中使用m&m’s基准测试比较了各种检索响应格式。我们观察到，对于前3个匹配项返回代码比仅在检索响应中显示原型会得到更高的工具f1分数。我们还发现，代理解决查询所需的交互步骤数量减少，这一直是判断选择使用哪些工具以及如何使用的低不确定性的一个一致指标。最后，现实世界中的代码块可能有数百行，这迅速增加了代理需要处理的上下文长度。为了解决这个问题，我们使用GPT-4为前3个检索结果生成文档字符串，在文档字符串和原始代码之间，显示较短的那个。如预期的那样，CodeNav中的默认配置在工具f1上高于仅显示原型，但低于显示代码。

### 5.4 什么样的库描述是好的？

表4：在M³ToolEval上比较两种库描述。为了提供长度参考，M³ToolEval中工具使用的描述长度为5K。

| 库描述 | 长度 | 准确性 | 步骤 |
| --- | --- | --- | --- |
| 文件路径 | 253 | 76.8 ± 6.5 | 8.3 ± 0.3 |
| 文件路径 + 文件描述 | 641 | 81.7 ± 4.9 | 7.8 ± 0.4 |

我们展示了在表格[4](https://arxiv.org/html/2406.12276v1#S5.T4 "表格 4 ‣ 5.4 什么是好的库描述？ ‣ 5 实验 ‣ CodeNav：超越工具使用，利用LLM代理在真实世界代码库中的应用")中库描述的影响。M³ToolEval实现为一个由5个文件组成的代码库，每个文件包含针对特定问题领域的工具：网页浏览、旅行规划、DNA测序、信息加密和财务计算。第一个库描述仅提供这些文件的相对路径（*例如*，*m3eval/travel_planner.py*），而第二个描述还包括对文件内容的简要总结（*例如*，*“规划旅行的功能，包括查找航班、预定酒店和预算计算”*）。直观来看，这使得CodeNav能够生成搜索关键词，从而在解决问题时需要更少的交互，表现出更优的性能。我们提供以下三条建议，以编写好的库描述：(i) 为目标领域提供背景信息，使得LLM可以利用其领域知识生成有用的搜索关键词；(ii) 描述库的结构（*例如*，目录结构或库所作的关键假设）；(iii) 提供简短的（不一定详尽的）自然语言描述，说明可用功能。

*### 5.5 LLM选择对性能的影响

表格 5：LLM选择对m&m’s的消融实验。

| LLM | 精度 | 召回率 | F1值 | 步数 |
| --- | --- | --- | --- | --- |
| gpt-4-1106-preview | 88.0 ±6.1 | 78.2 ±4.5 | 80.6 ±5.1 | 7.2 ±0.2 |
| gpt-3.5-turbo-0125 | 54.36 ± 2.3 | 15.77 ± 1.4 | 22.96 ±0.8 | 9.08 ±1.07 |
| Mixtral-8x22B-Instruct-v0.1 | 82.50 ± 2.1 | 62.31 ± 1.9 | 67.91 ±1.3 | 9.06 ±0.3 |
| Qwen1.5-110B-Chat | 78.15 ± 3.1 | 38.49 ± 5.5 | 48.84 ±5.1 | 10.00 ±0.2 |

我们使用了GPT-4（OpenAI，[2023](https://arxiv.org/html/2406.12276v1#bib.bib19)），特别是gpt-4-1106-preview,³³3[https://platform.openai.com/docs/models/gpt-4-turbo-and-gpt-4](https://platform.openai.com/docs/models/gpt-4-turbo-and-gpt-4)作为CodeNav的底层LLM。由于GPT-4是目前公开可用的性能最强的LLM之一，因此自然会有人问，当使用更小的、可能是开源的LLM时，CodeNav的性能会如何下降。我们通过在将GPT-4替换为GPT-3.5、Mistral 8$\times$22B混合专家模型（Jiang等人，[2023](https://arxiv.org/html/2406.12276v1#bib.bib10)，[2024](https://arxiv.org/html/2406.12276v1#bib.bib11)）以及Qwen1.5 110B模型（Bai等人，[2023](https://arxiv.org/html/2406.12276v1#bib.bib2)）时，使用CodeNav运行m&m评估。⁴⁴4模型的补全通过together.ai API获得，见[https://docs.together.ai](https://docs.together.ai)。我们选择使用Mistral和Qwen模型，因为它们代表了一些最大的、开源的LLM，具有较长的上下文窗口（根据经验，上下文大小低于16k标记的LLM通常会因超出上下文限制而失败）。我们的结果显示在表[5](https://arxiv.org/html/2406.12276v1#S5.T5 "Table 5 ‣ 5.5 Impact of LLM choice on performance ‣ 5 Experiments ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")中。尽管GPT-4驱动的CodeNav表现优异，但开源模型的表现也相当不错，主要在召回率上稍显逊色（这表明存在搜索失败）。令人惊讶的是，GPT-3.5的变体表现较差；在检查失败的轨迹时，发现这通常是由于代理未能在回合结束时恰当地总结其行动，可能通过额外的提示调整可以缓解此问题。*  *## 6 讨论

我们认为，现在是从工具使用转向代码使用的时候了；从向LLM代理提供手动策划和精心描述的工具集，到改为向它们展示由人类为人类编写的现有代码库。正如我们在案例研究和定量结果中所展示的那样，只要在工程代码搜索和执行环境时充分考虑，并为代理提供足够的灵活性和反馈，通过代码使用的代理可以获得相当显著的行为。

## 参考文献

+   phi [2024] phidata, 2024. URL [https://github.com/phidatahq/phidata](https://github.com/phidatahq/phidata).

+   Bai等人[2023] J. Bai, S. Bai, Y. Chu, Z. Cui, K. Dang, X. Deng, Y. Fan, W. Ge, Y. Han, F. Huang, B. Hui, L. Ji, M. Li, J. Lin, R. Lin, D. Liu, G. Liu, C. Lu, K. Lu, J. Ma, R. Men, X. Ren, X. Ren, C. Tan, S. Tan, J. Tu, P. Wang, S. Wang, W. Wang, S. Wu, B. Xu, J. Xu, A. Yang, H. Yang, J. Yang, S. Yang, Y. Yao, B. Yu, H. Yuan, Z. Yuan, J. Zhang, X. Zhang, Y. Zhang, Z. Zhang, C. Zhou, J. Zhou, X. Zhou, 和T. Zhu。Qwen技术报告。*arXiv预印本arXiv:2309.16609*，2023。

+   Chen 等人 [2021a] M. Chen, J. Tworek, H. Jun, Q. Yuan, H. P. de Oliveira Pinto, J. Kaplan, H. Edwards, Y. Burda, N. Joseph, G. Brockman, A. Ray, R. Puri, G. Krueger, M. Petrov, H. Khlaaf, G. Sastry, P. Mishkin, B. Chan, S. Gray, N. Ryder, M. Pavlov, A. Power, L. Kaiser, M. Bavarian, C. Winter, P. Tillet, F. P. Such, D. Cummings, M. Plappert, F. Chantzis, E. Barnes, A. Herbert-Voss, W. H. Guss, A. Nichol, A. Paino, N. Tezak, J. Tang, I. Babuschkin, S. Balaji, S. Jain, W. Saunders, C. Hesse, A. N. Carr, J. Leike, J. Achiam, V. Misra, E. Morikawa, A. Radford, M. Knight, M. Brundage, M. Murati, K. Mayer, P. Welinder, B. McGrew, D. Amodei, S. McCandlish, I. Sutskever 和 W. Zaremba. 评估基于代码训练的大型语言模型。*CoRR*，abs/2107.03374，2021年。网址 [https://arxiv.org/abs/2107.03374](https://arxiv.org/abs/2107.03374)。

+   Chen 等人 [2021b] M. Chen, J. Tworek, H. Jun, Q. Yuan, H. Ponde, J. Kaplan, H. Edwards, Y. Burda, N. Joseph, G. Brockman, A. Ray, R. Puri, G. Krueger, M. Petrov, H. Khlaaf, G. Sastry, P. Mishkin, B. Chan, S. Gray, N. Ryder, M. Pavlov, A. Power, L. Kaiser, M. Bavarian, C. Winter, P. Tillet, F. P. Such, D. W. Cummings, M. Plappert, F. Chantzis, E. Barnes, A. Herbert-Voss, W. H. Guss, A. Nichol, I. Babuschkin, S. Balaji, S. Jain, A. Carr, J. Leike, J. Achiam, V. Misra, E. Morikawa, A. Radford, M. M. Knight, M. Brundage, M. Murati, K. Mayer, P. Welinder, B. McGrew, D. Amodei, S. McCandlish, I. Sutskever 和 W. Zaremba. 评估基于代码训练的大型语言模型。*ArXiv*，abs/2107.03374，2021年。网址 [https://api.semanticscholar.org/CorpusID:235755472](https://api.semanticscholar.org/CorpusID:235755472)。

+   Elastic [2024] Elastic. Elasticsearch，2024年。网址 [https://www.elastic.co/elasticsearch/](https://www.elastic.co/elasticsearch/)。版本 8.12.1。

+   GitHub [2024] GitHub. GitHub Copilot - 您的 AI 编程搭档，2024年。网址 [https://github.com/features/copilot](https://github.com/features/copilot)。访问时间：2024-05-16。

+   Gupta 和 Kembhavi [2023] T. Gupta 和 A. Kembhavi. 视觉编程：无需训练的组成式视觉推理。*CVPR*，14953–14962页，2023年。网址 [https://api.semanticscholar.org/CorpusID:253734854](https://api.semanticscholar.org/CorpusID:253734854)。

+   Hsieh 等人 [2023] C.-Y. Hsieh, S. Chen, C.-L. Li, Y. Fujii, A. J. Ratner, C.-Y. Lee, R. Krishna 和 T. Pfister. 工具文档使大语言模型能够零-shot 工具使用。*ArXiv*，abs/2308.00675，2023年。网址 [https://api.semanticscholar.org/CorpusID:260351459](https://api.semanticscholar.org/CorpusID:260351459)。

+   Huang 等人 [2023] D. Huang, Q. Bu, J. M. Zhang, M. Luck 和 H. Cui. Agentcoder：基于多代理的代码生成，具有迭代测试和优化。*ArXiv*，abs/2312.13010，2023年。网址 [https://api.semanticscholar.org/CorpusID:266374622](https://api.semanticscholar.org/CorpusID:266374622)。

+   姜等人 [2023] A. Q. 姜, A. 萨布拉罗尔, A. 孟什, C. 班福德, D. S. 查普洛特, D. 德·拉斯·卡萨斯, F. 布雷桑, G. 伦基尔, G. 兰普尔, L. 索尔尼尔, L. R. 拉沃, M. 拉肖, P. 斯托克, T. L. 斯卡奥, T. 拉夫里尔, T. 王, T. 拉克鲁瓦, 和 W. E. 塞耶德. Mistral 7b. *CoRR*, abs/2310.06825, 2023. doi: 10.48550/ARXIV.2310.06825. URL [https://doi.org/10.48550/arXiv.2310.06825](https://doi.org/10.48550/arXiv.2310.06825).

+   姜等人 [2024] A. Q. 姜, A. 萨布拉罗尔, A. 鲁, A. 孟什, B. 萨瓦里, C. 班福德, D. S. 查普洛特, D. 德·拉斯·卡萨斯, E. B. 汉纳, F. 布雷桑, G. 伦基尔, G. 布尔, G. 兰普尔, L. R. 拉沃, L. 索尔尼尔, M. 拉肖, P. 斯托克, S. 苏布拉马尼安, S. 杨, S. 安东尼亚克, T. L. 斯卡奥, T. 热尔韦, T. 拉夫里尔, T. 王, T. 拉克鲁瓦, 和 W. E. 塞耶德. Mixtral of experts. *CoRR*, abs/2401.04088, 2024. doi: 10.48550/ARXIV.2401.04088. URL [https://doi.org/10.48550/arXiv.2401.04088](https://doi.org/10.48550/arXiv.2401.04088).

+   吉门内兹等人 [2024] C. E. 吉门内兹, J. 杨, A. 韦蒂格, S. 姚, K. 裴, O. 普雷斯, 和 K. R. 纳拉西曼. SWE-bench：语言模型能解决现实世界的GitHub问题吗？在 *第十二届国际学习表示会议*，2024年。URL [https://openreview.net/forum?id=VTF8yNQM66](https://openreview.net/forum?id=VTF8yNQM66).

+   乔特等人 [2022] T. 乔特, H. 特里维迪, M. 芬莱森, Y. 富, K. 理查森, P. 克拉克, 和 A. 萨巴尔瓦尔. 分解提示：解决复杂任务的模块化方法。*ArXiv*, abs/2210.02406, 2022. URL [https://api.semanticscholar.org/CorpusID:252715485](https://api.semanticscholar.org/CorpusID:252715485).

+   李等人 [2023a] C. 李, J. 梁, A. 曾, X. 陈, K. 豪斯曼, D. 萨迪赫, S. 莱文, F.-F. 李, F. 夏, 和 B. 伊赫特. 代码链：与语言模型增强的代码模拟器进行推理。*ArXiv*, abs/2312.04474, 2023a. URL [https://api.semanticscholar.org/CorpusID:266051661](https://api.semanticscholar.org/CorpusID:266051661).

+   李等人 [2023b] M. 李, Y. 赵, B. 余, F. 宋, H. 李, H. 余, Z. 李, F. 黄, 和 Y. 李. Api-bank：一个综合性的工具增强LLM基准。在 H. Bouamor, J. Pino, 和 K. Bali 编辑的 *2023年自然语言处理实证方法会议论文集，EMNLP 2023，新加坡，2023年12月6日至10日*，第3102-3116页。计算语言学协会，2023b. doi: 10.18653/V1/2023.EMNLP-MAIN.187. URL [https://doi.org/10.18653/v1/2023.emnlp-main.187](https://doi.org/10.18653/v1/2023.emnlp-main.187).

+   马等人 [2024] Z. 马, W. 黄, J. 张, T. 古普塔, 和 R. 克里希纳. m&m’s：评估工具使用在多步多模态任务中的基准。*ArXiv*, abs/2403.11085, 2024. URL [https://api.semanticscholar.org/CorpusID:268512938](https://api.semanticscholar.org/CorpusID:268512938).

+   Madaan 等人 [2023] A. Madaan, N. Tandon, P. Gupta, S. Hallinan, L. Gao, S. Wiegreffe, U. Alon, N. Dziri, S. Prabhumoye, Y. Yang, S. Welleck, B. P. Majumder, S. Gupta, A. Yazdanbakhsh 和 P. Clark. Self-refine：通过自我反馈的迭代优化。*ArXiv*，abs/2303.17651，2023。网址 [https://api.semanticscholar.org/CorpusID:257900871](https://api.semanticscholar.org/CorpusID:257900871)。

+   Ni 等人 [2024] A. Ni, M. Allamanis, A. Cohan, Y. Deng, K. Shi, C. Sutton 和 P. Yin. Next：教授大型语言模型推理代码执行。2024。网址 [https://api.semanticscholar.org/CorpusID:269302914](https://api.semanticscholar.org/CorpusID:269302914)。

+   OpenAI [2023] OpenAI. GPT-4技术报告。*CoRR*，abs/2303.08774，2023。doi: 10.48550/ARXIV.2303.08774。网址 [https://doi.org/10.48550/arXiv.2303.08774](https://doi.org/10.48550/arXiv.2303.08774)。

+   Osika [2023] A. Osika. Gpt-engineer，2023。网址 [https://github.com/gpt-engineer-org/gpt-engineer](https://github.com/gpt-engineer-org/gpt-engineer)。

+   Paranjape 等人 [2023] B. Paranjape, S. M. Lundberg, S. Singh, H. Hajishirzi, L. Zettlemoyer 和 M. T. Ribeiro. ART：用于大语言模型的自动多步推理和工具使用。*CoRR*，abs/2303.09014，2023。doi: 10.48550/ARXIV.2303.09014。网址 [https://doi.org/10.48550/arXiv.2303.09014](https://doi.org/10.48550/arXiv.2303.09014)。

+   Qin 等人 [2024] Y. Qin, S. Liang, Y. Ye, K. Zhu, L. Yan, Y.-T. Lu, Y. Lin, X. Cong, X. Tang, B. Qian, S. Zhao, R. Tian, R. Xie, J. Zhou, M. H. Gerstein, D. Li, Z. Liu 和 M. Sun. Toolllm：帮助大型语言模型掌握16000多个现实世界的API。*ICLR*，abs/2307.16789，2024。网址 [https://api.semanticscholar.org/CorpusID:260334759](https://api.semanticscholar.org/CorpusID:260334759)。

+   Sur’is 等人 [2023] D. Sur’is, S. Menon 和 C. Vondrick. Vipergpt：通过Python执行进行视觉推理。*ICCV*，第11854–11864页，2023。网址 [https://api.semanticscholar.org/CorpusID:257505358](https://api.semanticscholar.org/CorpusID:257505358)。

+   Wang 等人 [2023] G. Wang, Y. Xie, Y. Jiang, A. Mandlekar, C. Xiao, Y. Zhu, L. J. Fan 和 A. Anandkumar. Voyager：一个具有大语言模型的开放式具身代理。*ArXiv*，abs/2305.16291，2023。网址 [https://api.semanticscholar.org/CorpusID:258887849](https://api.semanticscholar.org/CorpusID:258887849)。

+   Wang 等人 [2024] X. Wang, Y. Chen, L. Yuan, Y. Zhang, Y. Li, H. Peng 和 H. Ji. 可执行代码操作引发更好的LLM代理。*CoRR*，abs/2402.01030，2024。doi: 10.48550/ARXIV.2402.01030。网址 [https://doi.org/10.48550/arXiv.2402.01030](https://doi.org/10.48550/arXiv.2402.01030)。

+   Wei等人[2022] J. Wei, X. Wang, D. Schuurmans, M. Bosma, B. Ichter, F. Xia, E. H. Chi, Q. V. Le, 和D. Zhou. 思维链提示激发大语言模型中的推理. 收录于 S. Koyejo, S. Mohamed, A. Agarwal, D. Belgrave, K. Cho, 和A. Oh, 编辑, *神经信息处理系统进展 35: 2022年神经信息处理系统年会, NeurIPS 2022, 美国新奥尔良, 2022年11月28日 - 12月9日*, 2022. 网址 [http://papers.nips.cc/paper_files/paper/2022/hash/9d5609613524ecf4f15af0f7b31abca4-Abstract-Conference.html](http://papers.nips.cc/paper_files/paper/2022/hash/9d5609613524ecf4f15af0f7b31abca4-Abstract-Conference.html).

+   Wolf等人[2020] T. Wolf, L. Debut, V. Sanh, J. Chaumond, C. Delangue, A. Moi, P. Cistac, T. Rault, R. Louf, M. Funtowicz, J. Davison, S. Shleifer, P. von Platen, C. Ma, Y. Jernite, J. Plu, C. Xu, T. L. Scao, S. Gugger, M. Drame, Q. Lhoest, 和A. M. Rush. Transformers: 先进的自然语言处理技术. 收录于 *2020年自然语言处理经验方法会议论文集：系统展示*, 页码38–45, 在线，2020年10月。计算语言学协会. 网址 [https://www.aclweb.org/anthology/2020.emnlp-demos.6](https://www.aclweb.org/anthology/2020.emnlp-demos.6).

+   Wu等人[2023] Q. Wu, G. Bansal, J. Zhang, Y. Wu, S. Zhang, E. Zhu, B. Li, L. Jiang, X. Zhang, 和C. Wang. Autogen: 通过多代理对话框架实现下一代LLM应用. *ArXiv*, abs/2308.08155, 2023. 网址 [https://api.semanticscholar.org/CorpusID:260925901](https://api.semanticscholar.org/CorpusID:260925901).

+   Yang等人[2023] Z. Yang, L. Li, J. Wang, K. Lin, E. Azarnasab, F. Ahmed, Z. Liu, C. Liu, M. Zeng, 和L. Wang. Mm-react: 使用ChatGPT进行多模态推理与行动. *ArXiv*, abs/2303.11381, 2023. 网址 [https://api.semanticscholar.org/CorpusID:257637012](https://api.semanticscholar.org/CorpusID:257637012).

+   Yao等人[2023] S. Yao, J. Zhao, D. Yu, N. Du, I. Shafran, K. R. Narasimhan, 和Y. Cao. React: 在语言模型中协同推理与行动. 收录于 *第十一届国际学习表征会议, ICLR 2023, 卢旺达基加利, 2023年5月1-5日*. OpenReview.net, 2023. 网址 [https://openreview.net/pdf?id=WE_vluYUL-X](https://openreview.net/pdf?id=WE_vluYUL-X).

+   Zhang等人[2023] J. Zhang, R. Krishna, A. H. Awadallah, 和C. Wang. Ecoassistant: 更经济、更精准地使用LLM助手. *ArXiv*, abs/2310.03046, 2023. 网址 [https://api.semanticscholar.org/CorpusID:263671677](https://api.semanticscholar.org/CorpusID:263671677).

+   Zhou等人[2023] S. Zhou, U. Alon, F. F. Xu, Z. Wang, Z. Jiang, 和G. Neubig. Docprompting: 通过检索文档生成代码. 收录于 *ICLR*, 2023年5月，卢旺达基加利. 网址 [https://arxiv.org/abs/2207.05987](https://arxiv.org/abs/2207.05987).

## 附录

本附录包含以下内容：

+   •

    m&m’s、M³ToolEval 和 API-Bank 的实验详情（附录 [A](https://arxiv.org/html/2406.12276v1#A1 "附录 A 实验详情 ‣ CodeNav：超越工具使用，使用大规模语言模型代理操作现实世界代码库")）

+   •

    计算需求（附录 [B](https://arxiv.org/html/2406.12276v1#A2 "附录 B 计算需求 ‣ CodeNav：超越工具使用，使用大规模语言模型代理操作现实世界代码库")）

+   •

    库和工具描述（附录 [C](https://arxiv.org/html/2406.12276v1#A3 "附录 C 库和工具描述 ‣ CodeNav：超越工具使用，使用大规模语言模型代理操作现实世界代码库")）

+   •

    一个完整的检索响应，显示代码、自动生成的总结文档字符串和原型（附录 [D](https://arxiv.org/html/2406.12276v1#A4 "附录 D 检索响应示例 ‣ CodeNav：超越工具使用，使用大规模语言模型代理操作现实世界代码库")）

+   •

    CodeNav 的局限性（附录 [E](https://arxiv.org/html/2406.12276v1#A5 "附录 E 局限性 ‣ CodeNav：超越工具使用，使用大规模语言模型代理操作现实世界代码库")）

+   •

    社会影响（附录 [F](https://arxiv.org/html/2406.12276v1#A6 "附录 F 社会影响 ‣ CodeNav：超越工具使用，使用大规模语言模型代理操作现实世界代码库")）

请访问我们的项目网站，[https://codenav.allenai.org/](https://codenav.allenai.org/)，以获取更多信息：

+   •

    3 个案例研究的完整轨迹。

+   •

    一个 CodeNav 在 m&m’s 上的示例运行。此运行包含一个 HTML 文件，可在浏览器中打开，查看由 CodeNav 生成的程序、真实程序以及轨迹链接。

+   •

    类似地，CodeNav 在 M³ToolEval 上的示例运行及 HTML 可视化。

+   •

    m&m’s 和 M³ToolEval 代码库的库和工具描述的并排对比。

表格 6：代码库统计数据。

| 代码库 | 文件 | 片段 | 行数 | 字符数 | 函数数 | 类数 |
| --- | --- | --- | --- | --- | --- | --- |
| m&m’s | 2 | 58 | 971 | 34277 | 39 | 0 |
| M³ToolEval | 8 | 39 | 485 | 14009 | 21 | 2 |
| API-Bank | 54 | 163 | 6144 | 207656 | 1 | 53 |
| CodeNav | 36 | 369 | 4055 | 141636 | 58 | 46 |
| PhiData | 426 | 2549 | 49731 | 2169989 | 133 | 398 |
| \mintinlinepythontransformers | 3475 | 50508 | 1362242 | 63105978 | 4354 | 11424 |

## 附录 A 实验详情

对于在 CodeNav 评估中使用的 3 个工具使用基准，我们现在提供有关基准如何适应代码使用以及使用的度量标准的详细信息。

### A.1 m&m’s

#### A.1.1 适应代码使用

m&m’s 中的所有工具都实现于两个文件中；一个包含工具函数的 Python 文件和一个配置文件。由于 m&m’s 中的函数既没有类型提示，也没有内联注释或文档字符串，为了提供最少的上下文，我们将原始 m&m’s 基准中的工具描述作为单行文档字符串添加到每个函数中。例如，对于函数 \mintinlinepythoncolor_pop，我们添加了 *它接受一张图片和一个或多个对象，并返回一张仅将对象着色，其余部分为黑白的图片*。请注意，我们没有提供关于输入参数或输出的额外信息。此外，m&m’s 的评估期望输出的是一系列函数调用，因此我们在代理中启用了“code_summary”动作，以便获取所需格式的代码解决方案。为了遵守评估要求，我们提供了以下指南：

```
When solving tasks YOU MUST RESPECT the following guidelines:
1\. Do not implement any new functions. Just use the available functions.
2\. Generally, try searching for function names. Only if needed, include
function argument names. Do not include argument values.
3\. When you have a solution, use the code_summary action to summarize
the solution.
4\. When asked to generate text, don’t generate text yourself but rather
see if there is a function to do it.
5\. Tasks typically require 1 to 3 function calls.

```

#### A.1.2 评估指标

我们使用原始 m&m’s 论文中的宏平均工具 F1 指标（Ma 等人，[2024](https://arxiv.org/html/2406.12276v1#bib.bib16)），该指标是函数名的精度和召回率的调和均值，以此与真实程序的地面实况进行比较。当存在多个正确的地面实况时，我们使用最佳匹配。由于 m&m’s 中的许多查询没有单一的正确答案，我们不使用答案的正确性作为评估指标。类似地，我们发现有许多方法可以在生成自由格式代码时指定参数，因此我们认为基于参数名称和值的评估指标对于评估像 CodeNav 这样的自由格式代码生成代理是不可靠的。

#### A.1.3 数据拆分

m&m’s 包含大约 800 个样本。由于使用 GPT-4 作为语言模型对完整数据集进行评估的成本可能在 $150 到 $200 之间（并且我们每个实验都运行三次以计算误差条），因此我们随机抽取了一个 200 个样本的小数据集进行评估。

### A.2 M³ToolEval

#### A.2.1 代码使用的适应性

为了使 M³ToolEval 能够适应代码使用，我们首先创建了一个代码库，仅包含工具实现和相关数据（例如，它们的网页浏览工具使用的网页数据，以及旅行规划工具使用的航班、酒店和位置信息）。特别是对于网页浏览任务，尽管 M³ToolEval 将 WebBrowser 类的方法注册为单独的工具，但对于代码使用，我们让代理直接使用该类。此外，由于网页浏览任务使用的页面不现实，且假设页面格式固定，我们为网页浏览任务提供了一些上下文作为指南。最后，M³ToolEval 中的工具按任务分组（例如，所有用于 DNA 测序的工具都在同一个文件中），每个任务只使用来自同一文件的工具。因此，为了让代理能够利用这一假设，我们在库/工具描述中提供了文件路径，并要求代理在其搜索查询中识别并指定相关的文件名，以便集中于所需的工具。以下是我们使用的指南：

```
When solving tasks YOU MUST RESPECT the following guidelines:
1\. For browsing tasks use the WebBrowser object and navigate the web pages
using available methods to find what you are looking for. Sometimes the
relevant information may not be visible on the page but if you see
[Viewing page m of n] (where m < n) then you may use the scroll functions
to see more page content. If you see the information you need displayed on
the web page, feel free to use it directly without worrying about parsing
it using code. Do not write complex code. Rather, try to interact with the
browser one action at a time like clicking or scrolling.
2\. You may need to identify the relevant python file and specify this target
file in your search queries to get the relevant search results

```

#### A.2.2 评估指标

我们使用原始 M³ToolEval 工作中使用的最终答案准确性（Wang 等， [2024](https://arxiv.org/html/2406.12276v1#bib.bib25)）。

### A.3 API-Bank

#### A.3.1 针对代码使用的适应

API-Bank 基准测试考虑了一个人机“聊天”上下文，其中代理和用户来回发送消息，用户可能要求代理一个接一个地执行多个任务。然后，AI 代理通过下一步预测方法进行评估，其中代理接收完整的聊天上下文，直到时间 $t$，并需要预测时间 $t+1$ 时的某条真实的聊天消息或 JSON API 调用。由于 CodeNav 并非设计用于与用户进行来回聊天（也不打算评估其生成自然语言回复的能力），因此我们筛选 API-Bank level-1-given-desc 集中的所有真实交互，只保留那些最后一条用户消息后至少有一条来自代理的消息，其中代理发起了 API 调用。筛选后，我们剩下了 186 个样本（原来有 214 个）。在评估过程中，我们给 CodeNav 提供完整的聊天上下文，直到并包括最后一条用户消息（同时修改沙盒状态为此时的状态），然后评估 CodeNav 生成所有剩余 API 调用的能力。

为了让 CodeNav 能够进行 API 调用，我们要求它直接实例化相应的 API-Bank 类，然后在该类上调用方法 (*例如*，模型可能实例化 AddAgenda 类为 aa = AddAgenda()，然后运行 aa.call(token, content, time, location)，其中 token、content、time 和 location 是它之前定义的变量)。为了鼓励这种行为，我们包括如下格式的指令：

```
When solving tasks YOU MUST RESPECT the following guidelines:
1\. When calling APIs you should instantiate the relevant class and use the
‘call‘ method defined in the class. DO NOT USE INVOKE OTHER METHODS ON
THE CLASS, YOU MUST ONLY CALL THE ‘call‘ METHOD.
2\. Everything can be solved by calling APIs, do not define new APIs or
modify the existing ones.

```

在提供给 CodeNav 代理的库描述中。

请注意，上述内容与传统的 API-Bank 评估方式有显著不同，后者通常要求代理生成 JSON 格式的 API 调用，这些调用通过 API-Bank 路由到正确的类和方法。

#### A.3.2 评估指标

如上所述，我们评估 CodeNav 在给定聊天上下文的情况下生成正确剩余 API 调用的能力。至于 m&m’s 基准评估，我们仅评估 CodeNav 调用正确 API 的能力，并且为了便于评估，忽略这些 API 是否使用正确的参数或产生正确的结果。假设 CodeNav 调用了一个 API 序列 $A=\{a_{1},...,a_{n}\}$，而真实的 API 调用集是 $G=\{g_{1},...,g_{m}\}$，我们计算 $A$ 和 $G$ 之间的匹配数量（计数重复项），并计算召回率 $R=(\#matches)/|G|$ 和精确度 $P=(\#matches)/|A|$（如果 $|A|=0$，则精确度取为 0）。根据这个精确度和召回率，我们像往常一样计算 F1 分数，公式为 $F1=2\cdot P\cdot R/(R+P)$，当 $P+R=0$ 时，$F1$ 设为 0。

## 附录 B 计算要求

我们在每台配有8个NVIDIA RTX A6000 GPU的Ubuntu服务器上运行CodeNav评估。如前所述，我们不训练任何模型，而是利用OpenAI和together.ai的API进行推理，使用LLM进行推理。这意味着我们不需要（本地）GPU来进行LLM推理，但在许多情况下，CodeNav可能会受益于访问GPU（*例如*，用于图像修复）。尽管推理时间因基准不同而有所变化，但在单个8 GPU服务器上以24个并行进程运行m&m’s评估（200个查询）大约需要${\sim}$12分钟的墙钟时间（96分钟的GPU时间）。

## 附录 C 库和工具描述

在这里，我们提供了在案例研究中使用的库和工具描述以及定量评估 -

+   •

    图表 [5](https://arxiv.org/html/2406.12276v1#A6.F5 "图 5 ‣ 附录 F 社会影响 ‣ CodeNav：从工具使用到与LLM代理一起使用真实世界代码库")、[6](https://arxiv.org/html/2406.12276v1#A6.F6 "图 6 ‣ 附录 F 社会影响 ‣ CodeNav：从工具使用到与LLM代理一起使用真实世界代码库")、[7](https://arxiv.org/html/2406.12276v1#A6.F7 "图 7 ‣ 附录 F 社会影响 ‣ CodeNav：从工具使用到与LLM代理一起使用真实世界代码库") 显示了CodeNav在三个案例研究中使用的库描述。

+   •

    图表 [6](https://arxiv.org/html/2406.12276v1#A6.F6 "图 6 ‣ 附录 F 社会影响 ‣ CodeNav：从工具使用到与LLM代理一起使用真实世界代码库")、[8](https://arxiv.org/html/2406.12276v1#A6.F8 "图 8 ‣ 附录 F 社会影响 ‣ CodeNav：从工具使用到与LLM代理一起使用真实世界代码库")、[9](https://arxiv.org/html/2406.12276v1#A6.F9 "图 9 ‣ 附录 F 社会影响 ‣ CodeNav：从工具使用到与LLM代理一起使用真实世界代码库") 显示了CodeNav用于对m&m’s、M³ToolEval和API-Bank进行定量评估的库描述。

+   •

    图表 [10](https://arxiv.org/html/2406.12276v1#A6.F10 "图 10 ‣ 附录 F 社会影响 ‣ CodeNav：从工具使用到与LLM代理一起使用真实世界代码库")、[11](https://arxiv.org/html/2406.12276v1#A6.F11 "图 11 ‣ 附录 F 社会影响 ‣ CodeNav：从工具使用到与LLM代理一起使用真实世界代码库")、[12](https://arxiv.org/html/2406.12276v1#A6.F12 "图 12 ‣ 附录 F 社会影响 ‣ CodeNav：从工具使用到与LLM代理一起使用真实世界代码库") 显示了用于m&m’s、M³ToolEval和API-Bank的工具使用基准的工具描述。请注意，这些工具描述比相应的库描述要详细得多。

## 附录 D 检索响应示例

![参考说明](img/67c91eaa4ad3580e14444ad1515909bb.png)

图4：完整检索响应示例。完整检索的示例。它对应于图[2](https://arxiv.org/html/2406.12276v1#S3.F2 "Figure 2 ‣ 3.3 Agent Actions ‣ 3 CodeNav ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")中R5的扩展版本。

我们在图[2](https://arxiv.org/html/2406.12276v1#S3.F2 "Figure 2 ‣ 3.3 Agent Actions ‣ 3 CodeNav ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")中展示了一个完整的检索响应示例。这对应于图[2](https://arxiv.org/html/2406.12276v1#S3.F2 "Figure 2 ‣ 3.3 Agent Actions ‣ 3 CodeNav ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")中R5的扩展版本，在这里为了简洁我们只展示了一个响应。注意：(1) 在检索响应底部显示的类和函数原型/签名集合，(2) 这三条扩展检索结果包含了完整的代码（对于EsCodeRetriever和NumpyJSONEncoder类）以及自动生成的总结性文档字符串（对于RetrievalResult类）。

## 附录E 限制

虽然CodeNav能够产生一些令人印象深刻的结果，但我们强调了三个关键限制。(1) 我们当前实现的CodeNav假设代理生成的是Python代码（且给定的代码库是一个Python代码库）。虽然扩展到使用其他语言并不是一个重大工程挑战，但随着远离像Python这样的流行语言，LLM的表现可能会下降，因为对于这些语言在互联网上有大量的数据。(2) 我们的代理没有跨查询的长期记忆。这意味着，如果两次给出相同的查询，CodeNav会犯同样的错误并重复相同的搜索。(3) 最后，CodeNav的性能强烈依赖于底层LLM，且在使用较小的LLM时，性能可能急剧下降。这意味着大多数研究人员只能通过付费API使用CodeNav，且使用这些API进行大规模实验可能会很昂贵。

## 附录F 社会影响

虽然CodeNav在这方面并非独一无二，但日益强大的LLM代理的广泛普及有潜力自动化或增强许多技术性任务。虽然自动化总体上带来了许多社会上的积极影响，但显然它对那些可能失业的人产生了深远的负面影响。运行这些LLM所需的巨大的能源成本也可能对环境造成重大影响。

作为一个更为严重的潜在负面影响：在代码使用范式中，CodeNav 代理作为底层代理，具备在用户机器上运行任意代码的能力。鉴于这一点，它可能会在无意中造成危险（没有什么能阻止 CodeNav 出现逻辑错误，*例如*，错误地过滤文件列表后，意外删除计算机上的所有文件），也可能会故意造成危害（*例如*，恶意方可能上传被污染的模型权重，或拦截 API 调用，从而在 CodeNav 代理执行时运行“随机软件”骗局）。这些危险在工具使用范式中稍微容易缓解，因为当仅限于使用某些工具时，攻击向量本质上更少。

<svg class="ltx_picture" height="170.08" id="A6.F5.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,170.08) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="142.52" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">The codebase you will use is called ‘codenav‘. It is a library for creating LLM agents that can interact with one of the avilable environments to solve the user queries that require using an external codebase. For example, an agent can interact with "PythonCodeEnv" for code execution, and with "RetrievalEnv" for retrieving code snippets from an ElasticSearch index. It provides an "Episode" class for running this interaction with a specific agent and a set of environments. Here’s the directory structure: codenav/agents/ - contains subdirectories that store implementations of LLM agents implemented with different LLMs codenav/environments/ - contains environments that the agent can interact with codenav/interaction/ - contains Episode and messages implementations (messages is how agent interacts with environments) codenav/prompts/ - stores various system prompts for the LLM agent codenav/retrieval/ - various files related to creating elastic search index and retrieving items from the index codenav/utils/ - contains various utility python files codenav/constants.py - contains important constants</foreignobject></g></g></svg>

图 5：CodeNav 案例研究的库描述，见第[4.1](https://arxiv.org/html/2406.12276v1#S4.SS1 "4.1 CodeNav on CodeNav ‣ 4 Case Studies ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")节。

<svg class="ltx_picture" height="335.05" id="A6.F6.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,335.05) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="307.49" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">The codebase you’ll be using to solve user tasks is called ‘mnm‘. It has a single tool_api.py file which contains functions for various image, text, and audio related tasks. Specifically, here’s a high-level summary of available functions: - text understanding functions: for tasks like text generation, summarization, classification, answering questions based on a text context - image understanding functions: for tasks like image classification (1000 IMAGENET categories only), image captioning, answering questions about image (use this if you have a question that can’t be answered with IMAGENET classification), detecting objects (producing bounding boxes and labels for COCO categories) and segmenting objects (producing segmentation masks and labels for COCO categories), transcribing alphanumeric characters in an image to text (also known as optical character recognition). - image editing functions: for generating images give a text description, editing images given the original image and a description of how to modify the image (can handle queries that require replacing or removing certain objects in the scene without detecting the object first), image cropping, or achieving effects like color pop and background blur given the segmented objects which you want to highlight in the image. - information retrieval functions: for retrieving factual information or interesting facts about dates, years, numbers, movies, weather, geographical coordinates of a city, wikipedia articles, or fun trivia. Also includes a love calculator for checking compatibility given two names. - object centric functions: these are functions that accept a list of detected or segmented objects for tasks like counting, selecting an object, tagging an image with objects (drawing bounding boxes and labels), or replacing objects with emojis. Note that these functions here do not detect the objects themselves. - audio understanding functions: for tasks related to audio understanding like speech recognition</foreignobject></g></g></svg>

图 6：m&m’s 的库描述以及第[4.2](https://arxiv.org/html/2406.12276v1#S4.SS2 "4.2 Multimodal Processing and Reasoning ‣ 4 Case Studies ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")节中的多模态案例研究。

<svg class="ltx_picture" height="136.87" id="A6.F7.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,136.87) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="109.31" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">The codebase you will use is called ‘phi‘ (already installed via pip). Here are some key file paths: phi/tools/email.py - for sending emails (requires passkey which can be read from environment variable GOOGLE_KEY; use ABC as sender name and ABC@DEF.org as sender email id) phi/tools/arxiv_toolkit.py - for search arxiv for research papers phi/tools/tavily.py - an AI driven search engine (requires pass key stored in environment variable TAVILY_KEY) phi/tools/duckduckgo.py - a search engine similar to google phi/tools/newspaper4k.py - for scarping news articles phi/tools/pubmed.py - for searching biomedical and life sciences literature phi/tools/yfinance.py - for fetching financial data like stock prices from Yahoo Finance phi/tools/wikipedia.py - for searching wikipedia (need to use json.loads on the output to extract content)</foreignobject></g></g></svg>

图 7：PhiData 案例研究的库描述，见第[4.3](https://arxiv.org/html/2406.12276v1#S4.SS3 "4.3 Research assistant ‣ 4 Case Studies ‣ CodeNav: Beyond tool-use to using real-world codebases with LLM agents")节。

<svg class="ltx_picture" height="103.66" id="A6.F8.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,103.66) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="76.1" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">The codebase you will use is called ‘m3eval‘. It has the following directory structure: m3eval/travel_planner.py - function for planning travel including finding flights, making hotel reservation, and budget calculations m3eval/browser.py - contains the WebBrowser class for navigating web pages m3eval/dna_sequencer.py - various functions related to dna sequencing m3eval/message_decoder.py - functions for encoding and decoding messages including converting hex string to ascii and decoding caesar cipher m3eval/trade_calculator.py - functions for currency conversion, calculating tariffs etc</foreignobject></g></g></svg>

图 8：M³ToolEval 的库描述。

<svg class="ltx_picture" height="650.53" id="A6.F9.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,650.53) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="622.97" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">The codebase you’ll be using to solve user tasks is called ‘api-bank‘. This codebase implements a collection of different APIs as classes, here is a full list of these APIs: AddAgenda: The API for adding a agenda item includes content, time and location. AddAlarm: The API for setting an alarm includes a parameter for the alarm time. AddMeeting: This API allows users to make a reservation for a meeting and store the meeting information (e.g., topic, time, location, attendees) in the database. AddReminder: The API for adding a reminder item includes content and time. AddScene: This API adds a scene of smart home system, given the scene name and a list of smart devices AppointmentRegistration: This API registers an appointment of hospital. BookHotel: This API orders a hotel room. Two rooms are ordered if the number of adults is greater than 2\. Only one order can be made at same time. Calculator: This API provides basic arithmetic operations: addition, subtraction, multiplication, and division. CancelRegistration: This API cancels the registration of a patient given appointment ID. CancelTimedSwitch: Cancels a timed switch for a smart device. CheckToken: Check the user token. DeleteAccount: Delete an account. DeleteAgenda: The API for deleting a schedule item includes parameters for token, content, time, and location. DeleteAlarm: The API for removing an alarm includes a parameter for the time. DeleteMeeting: This API allows users to delete a reservation for a meeting and remove the meeting information in the database. DeleteReminder: The API for deleting a reminder item includes content and time. DeleteScene: This API deletes a scene by its name. Dictionary: This API searches the dictionary for a given keyword. DocumentQA: This API answers the question from a given document url. EmergencyKnowledge: This API searches for a given symptom for emergency knowledge. ForgotPassword: Sends an email to the user with a link to reset the password. Need call twice, first with ’Forgot Password’ status to get the verification code, then call again with ’Verification Code’ status to change the password. Must pass the name of the parameters when calling the API, like ForgotPassword(status=’Forgot Password’, username=’username’). GetToday: This API gets the current date. GetUserToken: Get the user token by username and password. ImageCaption: This API generates a caption for a given image. ModifyAgenda: The API for modifying a schedule item includes parameters for content, time, and location. ModifyAlarm: The API for modifying an alarm includes a parameter for the from_time to to_time. ModifyMeeting: This API allows users to modify a reservation for a meeting ModifyPassword: The API for modifying the password of the account. ModifyRegistration: This API modifies the registration of a patient given appointment ID. ModifyReminder: The API for deleting a reminder item includes content and time. ModifyScene: This API modifies a scene of smart home system, given the scene name and a list of smart devices OpenBankAccount: This is an API for opening a bank account for a user, given the account, password and name. PlayMusic: This API triggers a music player to play music. QueryAgenda: The API for getting a schedule item includes parameters for token, content, time, and location. QueryAlarm: The API for querying alarm clock, help user to check the alarm clock they have set. QueryBalance: This API queries the balance of a given user. QueryHealthData: This API queries the recorded health data in database of a given user and time span. QueryHistoryToday: This API queries the history of the given date. QueryMeeting: This API allows users to query the information of a meeting. QueryRegistration: This API queries the registration of a patient, given patient ID. QueryReminder: The API for querying a reminder item includes content and time. QueryScene: This API queries a scene of smart home system, given the scene name QueryStock: This API queries the stock price of a given stock code and date. RecordHealthData: This API records the health data of a user. RegisterUser: The API for registering a account, given the username, password and email. SearchEngine: This API searches for a given keyword for search engine. SendEmail: This API for sending email, given the receiver, subject and content. SpeechRecognition: This API recognizes the speech from a given audio url. SymptomSearch: This API searches for a given symptom. TimedSwitch: This API for setting a timed switch for a smart device. ToolSearcher: Searches for relevant tools in library based on the keywords. Translate: Translate the text to the target language. Wiki: This API for searching a keyword in Wikipedia.</foreignobject></g></g></svg>

图 9：API-Bank 的库描述。

<svg class="ltx_picture" height="668.21" id="A6.F10.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,668.21) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="640.65" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你将用于解决用户任务的代码库称为‘mnm’，这个代码库由一个名为tool_api.py的单一文件组成，该文件包含以下函数：text_generation(text) -> text：它接受一个文本提示并输出最有可能跟随该输入文本的内容。text_summarization(text) -> text：它接受一段文本并将其总结成几句话。text_classification(text) -> text：它接受一段文本并将其分类到模型词汇表中的某一类别（例如，基于情感分类为正面或负面）。question_answering(text, question) -> text：它接受一段文本和一个问题，并根据文本生成该问题的答案。image_generation(text) -> image：它接受一个文本提示并生成与该文本描述匹配的图像。image_captioning(image) -> text：它接受一张图像并生成该图像的文本描述。optical_character_recognition(image) -> text：它接受一张图像并输出图像中识别到的文本。image_classification(image) -> text：它接受一张图像并将图像中的主题分类到诸如猫或狗等类别。image_editing(image, prompt) -> image：它接受一张图像和一个文本提示，并根据文本输出一张新图像。object_detection(image) -> image, objects：它接受一张图像并输出在图像中检测到的物体的矩形边界框。image_segmentation(image) -> image, objects：它接受一张图像，将其分割为不同部分，并输出各部分的分割掩码，形状不限。automatic_speech_recognition(audio) -> text：它接受一个音频文件并生成该音频的转录文本。visual_question_answering(image, question) -> text：它接受一张图像和关于该图像的问题，并生成该问题的答案。image_crop(image, object) -> image：它接受一张图像和表示边界框坐标的4个数字，并裁剪出框内区域的图像。image_crop_left(image) -> image：它接受一张图像，裁剪并保留图像的左侧部分。image_crop_right(image) -> image：它接受一张图像，裁剪并保留图像的右侧部分。image_crop_top(image) -> image：它接受一张图像，裁剪并保留图像的顶部部分。image_crop_bottom(image) -> image：它接受一张图像，裁剪并保留图像的底部部分。background_blur(image, object) -> image：它接受一张图像和一个或多个前景物体，并返回背景模糊后的图像。color_pop(image, object) -> image：它接受一张图像和一个或多个物体，并返回一张只有物体有色，其他部分是黑白的图像。count(objects) -> number：它接受一个物体列表并返回物体的数量。tag(image, objects) -> image：它接受一张图像和一组带有边界框和类别标签的物体，并标注所有物体。select_object(objects, object_name) -> object：它接受一个物体列表，并根据输入的物体名称选择其中的一个物体。emoji(image, object, emoji) -> image：它接受一张图像和一个或多个物体的边界框坐标，并用表情符号替换这些物体（例如：愤怒/脸红/哭泣/头晕/困倦/做鬼脸/亲吻/微笑，外星人，幽灵，小鬼等）。get_date_fact(date) -> text：它提供关于日期的有趣事实。get_year_fact(year) -> text：它提供关于年份的有趣事实。get_math_fact(number) -> text：它提供关于数字的有趣数学事实。get_trivia_fact(number) -> text：它提供关于数字的有趣琐事。love_calculator(first_name, second_name) -> number：输入你的名字和伴侣/恋人/暗恋对象的名字，找出你们的爱情兼容性和成功爱情关系的机会。get_location(city) -> lon, lat：使用OpenStreetMap的Nominatim API将城市名或地址转换为地理坐标。search_movie(movie_title, movie_year) -> text：检索电影的基本信息，包括标题、年份、类型和导演。get_weather(lon, lat) -> objects：根据特定的地理坐标提供天气预报数据。wikipedia_simple_search(text) -> text：在维基百科上执行基本的搜索查询，以检索相关页面的摘要。注意——上述所有函数都会生成一个Python字典作为输出。上述函数签名显示了字典中存在的键。例如，get_year_fact(year) -> text表示可以通过

output = get_year_fact(year)

fact = output[’text’]</foreignobject></g></g></svg>

图 10：m&m's 工具描述。

<svg class="ltx_picture" height="784.44" id="A6.F11.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,784.44) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="756.88" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">The codebase you will use is called ‘m3eval‘. It has the following files and functions implemented in those files file: m3eval/message_decoder.py [1] convert_hex_to_ascii: Converts a hexadecimal string to ASCII. Arguments: hex_string (str) Signature: convert_hex_to_ascii(hex_string: str) -> str [2] reverse_string: Reverses a string. Arguments: string (str) Signature: reverse_string(string: str) -> str [3] caesar_decode: Decodes a string using the Caesar cipher. Arguments: message (str), shift (int) Signature: caesar_decode(message: str, shift: int) -> str [4] string_length: Finds the length of a string. Arguments: string (str) Signature: string_length(string: str) -> int [5] minimum_value: Finds the minimum value from given arguments. Arguments: *args (variable number of arguments) Signature: minimum_value(*args) -> int/float [6] maximum_value: Finds the maximum value from given arguments. Arguments: *args (variable number of arguments) Signature: maximum_value(*args) -> int/float file: m3eval/dna_sequencer.py [7] count_nucleotides: Counts the occurrences of each nucleotide in a DNA sequence. Arguments: dna_sequence (str) Signature: count_nucleotides(dna_sequence: str) -> dict [8] transcribe_dna_to_mrna: Transcribes DNA sequence to mRNA. Arguments: dna_sequence (str) Signature: transcribe_dna_to_mrna(dna_sequence: str) -> str [9] translate_mrna_to_amino_acid: Translates mRNA sequence to a chain of amino acids. Arguments: mrna_sequence (str) Signature: translate_mrna_to_amino_acid(mrna_sequence: str) -> str [10] find_max_nucleotide: Return the nucleotide (str) with the maximum count (int). Arguments: nucleotide_counts in the form of (k1, v1, k2, v2, …, kn, vn) Signature: find_max_nucleotide(*args) -> (str, int) [11] is_valid_dna_sequence: Checks if the DNA sequence is valid. Arguments: dna_sequence (str) Signature: is_valid_dna_sequence(dna_sequence: str) -> bool [12] reverse_transcribe_mrna_to_dna: Reverse transcribes mRNA sequence to DNA. Arguments: mrna_sequence (str) Signature: reverse_transcribe_mrna_to_dna(mrna_sequence: str) -> str file: m3eval/trade_calculator.py [13] convert_currency: Converts the commodity price to local currency. Arguments: base_price (float), conversion_rate (float) Signature: convert_currency(base_price: float, conversion_rate: float) -> float [14] calculate_tariff: Calculates the trade tariff based on the converted price. Arguments: price (float), tariff_rate (float, in Signature: calculate_tariff(price: float, tariff_rate: float) -> float [15] estimate_final_value: Estimates the final trade value including the tariff. Arguments: price (float), tariff (float) Signature: estimate_final_value(price: float, tariff: float) -> float [16] calculator: Evaluates the given expression and returns the result. Accepts a calculation expression as input. For example, "2 + (3 * 4)" will return 14. Signature: calculator(expression: str) -> float [17] find_minimum: Finds the minimum value among the given arguments. Accepts variable number of float arguments. Signature: find_minimum(*args: float) -> float [18] find_maximum: Finds the maximum value among the given arguments. Accepts variable number of float arguments. Signature: find_maximum(*args: float) -> float file: m3eval/travel_planner.py [19] find_flights: Finds flights based on source, destination and date. Arguments: from_location (str), to_location (str), date (str) in YYYY-MM-DD format. Returns a list of flights, each represented as a dictionary with keys "from_location", "to_location" (destination), "date", and "price". Example: ["from_location": "A", "to_location": "B", "date": "2023-12-25", "price": 450] Signature: find_flights(destination: str, date: str) -> List[Dict] [20] book_hotel: Books a hotel based on location and preferences. Arguments: location (str), *preferences (variable number of str arguments). Returns a list of hotels, each represented as a dictionary with keys "location", "preferences", "price_per_night", and "rating". Example: ["location": "A", "preferences": ["wifi", "pool"], "price_per_night": 120, "rating": 4] Signature: book_hotel(location: str, *preferences: str) -> List[Dict] [21] budget_calculator: Calculates the total budget for a trip. Arguments: flight_price (float), hotel_price_per_night (float), num_nights (int). Returns the total budget (float). Signature: budget_calculator(flight_price: float, hotel_price_per_night: float, num_nights: int) -> float file: m3eval/browser.py Note - To use the browser functions first create a browser instance using browser=WebBrowser() [22] browser.click_url: Clicks on a URL. A clickable URL looks like [Clickable ’<url_argument>’] in the webpage. Arguments: url (str). Returns the rendered content of the webpage after clicking the URL showing on the current rendered page. Signature: browser.click_url(url: str) -> str [23] browser.go_to_previous_page(): Goes back to the previous page. It has no arguments. After going back to the previous page, return the rendered content of the webpage. Signature: browser.go_to_previous_page() -> str [24] browser.scroll_down: Scrolls down the view. It has no arguments. Returns the rendered content of the webpage after scrolling down. Signature: browser.scroll_down() -> str [25] browser.scroll_up: Scrolls up the view. It has no arguments. Returns the rendered content of the webpage after scrolling up. Signature: browser.scroll_up() -> str [26] browser.view: Return the current view in string format of the rendered webpage. It has no arguments. Returns the rendered content of the webpage. You should call this when you want to see the rendered content of the current webpage. Signature: browser.view() -> str</foreignobject></g></g></svg>

图 11：M³ToolEval 工具描述。

<svg class="ltx_picture" height="1032.54" id="A6.F12.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,1032.54) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="1004.98" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">The codebase you’ll be using to solve user tasks is called ‘api-bank‘. This codebase implements a collection of different APIs as classes, here is a full list of these APIs: AddAgenda().call(token, content, time, location) Import as: from apis import AddAgenda Description: The API for adding a agenda item includes content, time and location. Arguments: - token (str): User’s token. - content (str): The content of the agenda. - time (str): The time for agenda. Format: - location (str): The location of the agenda. Returns: A dictionary whose "output" key has value of type str and description success or failed. AddAlarm().call(token, time) Import as: from apis import AddAlarm Description: The API for setting an alarm includes a parameter for the alarm time. Arguments: - token (str): User’s token. - time (str): The time for alarm. Format: Returns: A dictionary whose "output" key has value of type str and description success or failed. AddMeeting().call(token, meeting_topic, start_time, end_time, location, attendees) Import as: from apis import AddMeeting Description: This API allows users to make a reservation for a meeting and store the meeting information (e.g., topic, time, location, attendees) in the database. Arguments: - token (str): User’s token. - meeting_topic (str): The title of the meeting, no more than 50 characters. - start_time (str): The start time of the meeting, in the pattern of - end_time (str): The end time of the meeting, in the pattern of - location (str): The location where the meeting to be held, no more than 100 characters. - attendees (list(str)): The attendees of the meeting, including names, positions and other information. Returns: A dictionary whose "output" key has value of type str and description success or failed. AddReminder().call(token, content, time) Import as: from apis import AddReminder Description: The API for adding a reminder item includes content and time. Arguments: - token (str): User’s token. - content (str): The content of the conference. - time (str): The time for conference. Format: Returns: A dictionary whose "output" key has value of type str and description success or failed. AddScene().call(name, devices) Import as: from apis import AddScene Description: This API adds a scene of smart home system, given the scene name and a list of smart devices Arguments: - name (str): The name of the scene. - devices (list): The list of smart devices, containing the name and description. Format be like ["name": "light", "description": "Smart light in the kitchen", "name": "oven", "description": "Smart oven in the kitchen", "name": "range hood", "description": "Smart range hood in the kitchen"] Returns: A dictionary whose "output" key has value of type str and description Whether succeed.. AppointmentRegistration().call(patient_name, date, doctor_name) Import as: from apis import AppointmentRegistration Description: This API registers an appointment of hospital. Arguments: - patient_name (str): The name of patient. - date (str): The date of appointment. Format be like - doctor_name (str): The name of appointed doctor. Returns: A dictionary whose "output" key has value of type str and description The ID of appointment.. BookHotel().call(hotel_name, check_in_time, check_out_time, room_count, adult_count, child_count) Import as: from apis import BookHotel Description: This API orders a hotel room. Two rooms are ordered if the number of adults is greater than 2\. Only one order can be made at same time. Arguments: - hotel_name (str): The name of the hotel. - check_in_time (str): The time to check in. Format: - check_out_time (str): The time to check out. Format: - room_count (int): The number of rooms to order. - adult_count (int): The number of adults. - child_count (int): The number of children. Returns: A dictionary whose "output" key has value of type str and description The ID of the order.. Calculator().call(formula) Import as: from apis import Calculator Description: This API provides basic arithmetic operations: addition, subtraction, multiplication, and division. Arguments: - formula (str): The formula that needs to be calculated. Only integers are supported. Valid operators are +, -, *, /, and (, ). For example, ’(1 + 2) * 3’. Returns: A dictionary whose "output" key has value of type float and description The result of the formula.. CancelRegistration().call(appointment_id) Import as: from apis import CancelRegistration Description: This API cancels the registration of a patient given appointment ID. Arguments: - appointment_id (str): The ID of appointment. Returns: A dictionary whose "output" key has value of type str and description The status of cancellation.. . . . SearchEngine().call(keyword) Import as: from apis import SearchEngine Description: This API searches for a given keyword for search engine. Arguments: - keyword (str): The keyword to search. Returns: A dictionary whose "output" key has value of type list and description The list of results.. SendEmail().call(receiver, subject, content) Import as: from apis import SendEmail Description: This API for sending email, given the receiver, subject and content. Arguments: - receiver (str): The receiver address of the email. - subject (str): The subject address of the email. - content (str): The content of the email. Returns: A dictionary whose "output" key has value of type str and description The status of the email.. SpeechRecognition().call(url) Import as: from apis import SpeechRecognition Description: This API recognizes the speech from a given audio url. Arguments: - url (str): The url to download the audio. It should end with .wav. Returns: A dictionary whose "output" key has value of type str and description The transcript of the audio.. SymptomSearch().call(symptom) Import as: from apis import SymptomSearch Description: This API searches for a given symptom. Arguments: - symptom (str): The symptom to search. Returns: A dictionary whose "output" key has value of type list and description The list of results. Format be like ["name":possible disease name, "description": disease details,…]. TimedSwitch().call(name, time, on) Import as: from apis import TimedSwitch Description: This API for setting a timed switch for a smart device. Arguments: - name (str): The name of the smart device. - time (str): The time to switch the device on or off. Format: - on (bool): Whether to switch the device on or off. Returns: A dictionary whose "output" key has value of type str and description Whether the time switch is successful.. ToolSearcher().call(keywords) Import as: from apis import ToolSearcher Description: Searches for relevant tools in library based on the keywords. Arguments: - keywords (str): The keyword to search for. Returns: A dictionary whose "output" key has value of type Union[List[dict], dict] and description The best match tool(s).. Translate().call(src, src_lang, tgt_lang) Import as: from apis import Translate Description: Translate the text to the target language. Arguments: - src (str): The text to be translated. - src_lang (str): [Optional] The source language to translate from. Default is auto. - tgt_lang (str): [Optional] The target language to translate to. Default is english/en. Returns: A dictionary whose "output" key has value of type str and description The translated text.. Wiki().call(keyword) Import as: from apis import Wiki Description: This API for searching a keyword in Wikipedia. Arguments: - keyword (str): The keyword to search. Returns: A dictionary whose "output" key has value of type dict and description The list of results. Format be like "url": "xxx", "summary": "xxx", "content": "xxx".</foreignobject></g></g></svg>

图 12：API-Bank 工具描述。为了简洁起见，我们只展示了完整描述的开始和结束部分。*
