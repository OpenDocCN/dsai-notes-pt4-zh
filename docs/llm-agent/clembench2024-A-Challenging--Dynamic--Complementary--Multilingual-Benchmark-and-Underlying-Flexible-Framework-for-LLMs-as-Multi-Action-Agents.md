<!--yml

类别：未分类

日期：2025-01-11 12:35:17

-->

# clembench2024 一项具有挑战性、动态的、互补的多语种基准及其基础灵活框架，适用于将LLM作为多动作代理

> 来源：[https://arxiv.org/html/2405.20859/](https://arxiv.org/html/2405.20859/)

Anne Beyer, Kranti Chalamalasetti, Sherzod Hakimov

Brielen Madureira, Philipp Sadler, David Schlangen¹

计算语言学，语言学系

波茨坦大学，德国

¹德国人工智能研究中心（DFKI），柏林，德国

first.last@uni-potsdam.de   贡献：AB设计并运行了多语种实验。KC更新了Wordle游戏；BM为私人/共享版本做了更新；SH为绘图和参考做了更新，管理了排行榜并共同管理了项目。PS维护了主要框架和服务器基础设施。DS共同设计了实验，协同管理了项目，并编辑了文档。所有作者讨论了所有内容。

###### 摘要

最近的研究表明，大型语言模型（LLM）可以通过提示“自我游戏”对话式游戏，从而探测某些能力（如一般指令执行、战略目标导向、语言理解能力），并且通过自动评分的互动游戏玩法来评估这些能力。在本文中，我们采用了一个提出的框架来搭建这种游戏环境，并进一步测试其作为评估工具的有效性，从多个维度进行验证：我们展示了它能够轻松跟上新的发展，并避免数据污染；我们证明了其中实施的测试尚未饱和（人类表现远高于即便是最好的模型）；我们还展示了该框架能够用于探讨其他问题，比如提示语言对表现的影响。我们认为这一方法为在构建应用交互系统时做出模型选择提供了良好的基础，并且或许最终可以建立一个系统和模拟评估者的闭环开发环境。

clembench[2024] 一项具有挑战性、动态的、互补的多语种基准及其基础灵活框架，适用于将LLM作为多动作代理

Anne Beyer, Kranti Chalamalasetti, Sherzod Hakimov Brielen Madureira, Philipp Sadler, David Schlangen¹ ^†^†致谢：  贡献：AB设计并运行了多语种实验。KC更新了Wordle游戏；BM为私人/共享版本做了更新；SH为绘图和参考做了更新，管理了排行榜并共同管理了项目。PS维护了主要框架和服务器基础设施。DS共同设计了实验，协同管理了项目，并编辑了文档。所有作者讨论了所有内容。计算语言学，语言学系，波茨坦大学，德国 ¹德国人工智能研究中心（DFKI），柏林，德国 first.last@uni-potsdam.de

## 1 引言

通过从大型语言模型（LLMs）中诱导出智能行为的可能性，似乎为实现一个封闭循环的开发周期提供了可行的设想，其中对话系统是通过描述要达成的任务来构建的，并通过指定一个模拟用户来进行评估。尽管目前在这两个方面都有活跃的研究（实现任务导向的系统与 LLMs，参见例如 Hudeček 和 Dusek（[2023](https://arxiv.org/html/2405.20859v1#bib.bib7)）的工作，以及与 LLM 模拟用户的评估，参见例如 Sekulic 等人（[2024](https://arxiv.org/html/2405.20859v1#bib.bib14)）的研究），但一个重要的基础性步骤是确立 LLM 作为智能体这一观点的有效性和局限性。2023年，出现了多个框架，通过在更抽象的任务上设置 LLM 的对话“自我对弈”并设定完全可指定的目标来处理这一任务（这些任务通常被称为“对话游戏”（Schlangen，[2023](https://arxiv.org/html/2405.20859v1#bib.bib13)））。

在本文中，我们继续使用 clemgame 框架（Chalamalasetti 等人，[2023](https://arxiv.org/html/2405.20859v1#bib.bib2)），并验证了原始发布版本中一些留待未来工作的主张。具体而言，我们展示了：a) 这种方法可以扩展为一个动态基准测试，意味着所评估的确实是游戏本身，而不是特定的游戏实例；b) 它仍然是一个具有挑战性的基准测试，因为即使是最好的模型，其得分也明显低于人类表现，这一点我们在本文中首次确立；c) 它与其他基准测试互补，既包括基于参考的基准（Liang 等人提出的 HELM（[2022](https://arxiv.org/html/2405.20859v1#bib.bib9)）），也包括基于偏好的基准（Chiang 等人提出的 Chatbot arena（[2024](https://arxiv.org/html/2405.20859v1#bib.bib3)））；d) 基础抽象是灵活的，因此可以轻松集成新模型，使得我们能够像在本文中所做的那样，追踪开放权重模型自基准首次发布以来的发展；最后但同样重要的是，e) 该框架使得模型的多语言能力评估变得轻松可行，正如我们在这里为其中一个对话游戏所举的例子。

总的来说，我们得出的结论是，clembench 是一个对社区非常有价值的工具，不仅可以测试针对对话优化的 LLM（并基于结果做出决策），还可以作为深入研究 LLM 行为特定方面的工具。最后，我们对可能的未来应用进行了猜测，例如作为一个专门用于交互的学习环境，以及作为一个让我们更接近开篇段落所述愿景的工具，作为设计改进智能体的构建/测试框架。

## 2 对话游戏与智能体能力

从2022年认识到LLM（大规模语言模型）可以被视为通用的“对话模型”（例如，参见Andreas ([2022](https://arxiv.org/html/2405.20859v1#bib.bib1)))开始，便出现了一个想法，即它们可以被用来模拟对话中的各方，而这可以用来评估某些能力，比基于数据集的评估方法更有效。Qiao等人([2023](https://arxiv.org/html/2405.20859v1#bib.bib12))实现了一小部分游戏（类似20个问题，社交推理游戏），并在少量模型上进行了测试。Li等人([2023](https://arxiv.org/html/2405.20859v1#bib.bib8))也强调了需要“超越静态数据集”，并实现了一些互动任务，然而这些任务依赖于通过裁判模型进行评分。Gong等人([2023](https://arxiv.org/html/2405.20859v1#bib.bib5))将LLM集成到更明确的环境中，如Minecraft，通过增补专门设计的模块，将模型转化为具有特定目的的代理。Wu等人([2024](https://arxiv.org/html/2405.20859v1#bib.bib16))也实现了各种游戏并测试了少数几个模型。Zhou等人([2024](https://arxiv.org/html/2405.20859v1#bib.bib17))特别专注于“社交”技能，并使用类似游戏的设置来研究LLM实现的代理之间的自由互动。Duan等人([2024](https://arxiv.org/html/2405.20859v1#bib.bib4))最终设立了一些零-shot自我博弈游戏，用来比较明显的策略与已知的博弈论最优策略。除Duan等人([2024](https://arxiv.org/html/2405.20859v1#bib.bib4))外，这些工作的共同点是关注表面有效性，即所实现的游戏被简单地假定为具有挑战性，但并未尝试阐明它们针对的潜在构造的哪一方面。

对于当前的工作，我们继续在“自我博弈评估”这一思想的框架下开展研究，这一思想最早被实现（早于上述引用的工作），并且特别关注构造效度，即clemgame/clembench框架 Chalamalasetti等人([2023](https://arxiv.org/html/2405.20859v1#bib.bib2))。我们在这里不重复这些效度论证，只是将有兴趣的读者引导至原始出版物；我们在这里所做的，是介绍其中与本工作相关的基本组件。

该框架的主要思想是，通过提示模板来指定游戏，这些模板用自然语言向玩家解释游戏目标，通过响应解析规则来定义什么是有效的响应，并通过特定于游戏的游戏流程来定义什么是终局状态。一个程序化的GameMaster通过将模板实例化为特定的游戏实例（例如，在猜词游戏中，本轮要猜的词）并逐回合地提示玩家（玩家可以是LLMs，也可以是人类玩家）来实现游戏的进行。随后，结果回合通过特定的游戏评分规则进行评分。对于每个游戏，都会确定一个评分指标作为主要指标（始终从0（最差）到100（最好））。通过计算每个游戏的这个指标的平均值，然后再计算所有游戏的平均值，从而得出一个总体评分。违反解析规则的游戏计为未完成（结束）。我们追踪完成的游戏百分比；这使得我们能够区分格式规则遵守能力（这对任何将LLMs作为内部组件使用，其中输出需要是预设格式的应用非常重要）与战略游戏质量。总体评分是这两个评分的乘积。以下，我们将该基准（游戏集）称为clembench。

在这里报告的实验中，我们引入了一个通用化层，用于通过各种途径访问大型语言模型（LLMs）（例如，通过本地的huggingface transformers（Wolf等人，[2020](https://arxiv.org/html/2405.20859v1#bib.bib15)），或者通过llama.cpp¹¹1[https://github.com/ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp)，或者通过各种特定厂商的API）。这为我们提供了灵活性，能够基准测试大量模型，正如下面[3](https://arxiv.org/html/2405.20859v1#S3 "3 Flexible: Performance over Time ‣ clembench2024 A Challenging, Dynamic, Complementary, Multilingual Benchmark and Underlying Flexible Framework for LLMs as Multi-Action Agents")节中讨论的那样，并且可以轻松地集成新的模型。我们还仔细检查了上面描述的所有组件，特别是修正了一些游戏的解析规则和评分规则。²²2详细的变更列表可以在项目仓库中找到：[https://github.com/clembench/.](https://github.com/clembench/.) 请注意，这使得我们报告的得分与之前报告的得分不可直接比较。对于在之前报告的运行和我们最新运行中都进行评分的模型子集（19个模型），我们计算了一个排名相关性（Kendall的tau，$r_{\tau}$）为0.71（$p<.05$），即，强到非常强的相关性。

| models | sc | o/g |
| --- | --- | --- |
| gpt-4 | 59.49 | $$ |
| claude-v1.3 | 37.07 | $$ |
| gpt-3.5-turbo | 37.02 | $$ |
| text-davinci-003 | 15.78 | $$ |
| vicuna-13b | 4.24 | ow |
| oasst-12b | 1.74 | ow |
| koala-13b | 1.48 | ow |
| falcon-40b | 0.71 | ow |
| luminous-supreme | 0.00 | $$ |
| models | sc | o/g |
| --- | --- | --- |
| gpt-4-0613 | 60.90 | $$ |
| gpt-4-1106-preview | 60.33 | $$ |
| gpt-4-0314 | 58.81 | $$ |
| claude-v1.3 | 37.64 | $$ |
| claude-2.1 | 36.38 | $$ |
| claude-2 | 33.71 | $$ |
| gpt-3.5-turbo-0613 | 32.53 | $$ |
| gpt-3.5-turbo-1106 | 30.45 | $$ |
| openchat_3.5 | 19.72 | ow |
| mistral-medium | 17.99 | ow |
| mixtral-8x7b-instruct-v0.1 | 17.81 | ow |
| openchat-3.5-1210 | 17.61 | ow |
| sheep-duck-llama-2-70b-v1.1 | 17.12 | ow |
| yi-34b-chat | 16.77 | ow |
| wizardlm-70b-v1.0 | 16.70 | ow |
| tulu-2-dpo-70b | 15.90 | ow |
| sus-chat-34b | 15.64 | ow |
| claude-instant-1.2 | 15.44 | $$ |
| models | sc | o/g |
| --- | --- | --- |
| gpt-4-turbo-2024-04-09 | 58.30 | $$ |
| gpt-4-0125-preview | 52.50 | $$ |
| gpt-4-1106-preview | 51.99 | $$ |
| gpt-4-0613 | 51.09 | $$ |
| gpt-4o-2024-05-13 | 48.34 | $$ |
| claude-3-opus-20240229 | 42.42 | $$ |
| gemini-1.5-pro-latest | 41.72 | $$ |
| llama-3-70b-instruct | 35.11 | ow |
| claude-2.1 | 32.50 | $$ |
| gemini-1.5-flash-latest | 32.00 | $$ |
| claude-3-sonnet-20240229 | 30.53 | $$ |
| qwen1.5-72b-chat | 30.37 | ow |
| mistral-large-2402 | 28.17 | $$ |
| gpt-3.5-turbo-0125 | 27.22 | $$ |
| gemini-1.0-pro | 26.95 | $$ |
| command-r-plus | 24.94 | ow |
| openchat_3.5 | 23.64 | ow |
| claude-3-haiku-20240307 | 22.49 | $$ |
| sheep-duck-llama-2-70b-v1.1 | 21.50 | ow |
| llama-3-8b-instruct | 19.99 | ow |
| openchat-3.5-1210 | 18.22 | ow |
| wizardlm-70b-v1.0 | 17.40 | ow |
| openchat-3.5-0106 | 17.10 | ow |
| qwen1.5-14b-chat | 16.80 | ow |
| mistral-medium-2312 | 16.43 | $$ |

表格1：从左到右，展示了2023年6月、2023年11月、2024年5月的英文clembench结果。“ow”：开放权重模型，“$$”：加密模型。最佳加密模型保持不变（排除评分代码修复，见正文），开放权重模型有了显著提升。

## 3 灵活性：随时间变化的表现

表格[1](https://arxiv.org/html/2405.20859v1#S2.T1 "Table 1 ‣ 2 Dialogue Games and Agent Capabilities ‣ clembench2024 A Challenging, Dynamic, Complementary, Multilingual Benchmark and Underlying Flexible Framework for LLMs as Multi-Action Agents")展示了不同时间点的clembench结果，从初始发布结果、2023年11月的中间结果，到当前的结果。³³3当前的排行榜和所有以前的版本可以随时在[https://clembench.github.io/leaderboard.html](https://clembench.github.io/leaderboard.html)找到，详细结果日志可以在[https://github.com/clembench/clembench-runs](https://github.com/clembench/clembench-runs)查看。

有几点值得注意。首先，上述变化使我们能够跟踪这一快速发展的领域。与Chalamalasetti等人（[2023](https://arxiv.org/html/2405.20859v1#bib.bib2)）仅报告了9个模型的结果不同，当前版本现在跟踪53个模型。（为了节省空间，数据以低于16的分数进行限制。）其次，可以观察到两个有趣的趋势：

$\bullet$ 尽管在闭源权重模型领域竞争激烈，但总体而言，该领域并未取得显著进展。榜首位置仍然由GPT-4的一个变种占据，且最高得分也未提升（前提是这些数字可以直接比较；见上文讨论）。这表明在可实现的性能上已存在一定的饱和。

$\bullet$ 另一方面，开放权重模型在这段时间内取得了显著进展。虽然2023年6月最佳开放模型与最佳闭源模型之间的差距为55.25分，但五个月后的2023年11月，这一差距缩小到了41.18分，而到2024年5月，这一差距已缩小至24.94分，这要归功于llama3-70b-ins的卓越表现。（如附录[A](https://arxiv.org/html/2405.20859v1#A1 "附录A 完整结果 ‣ clembench2024 一个具有挑战性、动态、互补、多语言的基准和背后的灵活框架")中完整的表格所示，这部分是由于这些模型在格式规则遵循能力上的大幅改进。）

这表明我们认为，clembench作为一个跟踪领域发展的工具，尤其是在模型是否适合进入目标导向交互方面，具有重要价值。

## 4 动态：游戏，而非实例

正如Chalamalasetti等人（[2023](https://arxiv.org/html/2405.20859v1#bib.bib2)）已经提到的，但没有进一步跟进，游戏规范（通过模板）与游戏实例的分离使得可以将游戏视为生成性设备，从而创建一个动态基准，更容易避免“数据污染”（Magar和Schwartz，[2022](https://arxiv.org/html/2405.20859v1#bib.bib10)）。对于上述报告的运行，我们为clembench中包含的所有游戏创建了新实例。对于某些游戏，这仅仅需要从现有池中抽样（例如，为Wordle游戏选择新的目标词），对于其他游戏，则需要一些轻微的手动工作（例如，按照原文中描述的方法为禁忌游戏选择新的目标词；为图像游戏创建新的目标图像）。如上所述，我们的排名与之前的运行的相关性较高（0.71），这表明我们确实是在评估游戏，而不是评估实例。

| % played | de | en | it | ja | pt | te | tk | tr | zh |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| GPT-4 | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) | 99.44 (-0.56) | 98.33 (-1.67) | 100.0 (0.00) | 72.78 (-27.22) |
| Claude-3 | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) |
| Llama-3-70b | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) | 99.44 (-0.56) | 100.0 (0.00) | 100.0 (0.00) | 100.0 (0.00) |
| Llama-3-8b | 98.33 (-1.67) | 100.0 (0.00) | 100.0 (0.00) | 98.89 (-1.11) | 100.0 (0.00) | 87.78 (-12.22) | 28.89 (-71.11) | 0.0 (-100.00) | 100.0 (0.00) |
| Command-R+ | 100.0 (0.56) | 99.44 (0.00) | 100.0 (0.56) | 100.0 (0.56) | 100.0 (0.56) | 83.89 (-15.55) | 67.78 (-31.66) | 100.0 (0.56) | 99.44 (0.00) |
| Openchat | 98.33 (-1.67) | 100.0 (0.00) | 46.67 (-53.33) | 100.0 (0.00) | 50.0 (-50.00) | 0.0 (-100.00) | 0.0 (-100.00) | 0.0 (-100.00) | 46.67 (-53.33) |
| 质量评分 | de | en | it | ja | pt | te | tk | tr | zh |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| GPT-4 | 85.56 (-1.66) | 87.22 (0.00) | 89.44 (2.22) | 85.56 (-1.66) | 81.11 (-6.11) | 35.2 (-52.02) | 75.14 (-12.08) | 50.0 (-37.22) | 88.55 (1.33) |
| Claude-3 | 71.11 (-6.11) | 77.22 (0.00) | 72.22 (-5.00) | 68.89 (-8.33) | 73.33 (-3.89) | 52.22 (-25.00) | 61.11 (-16.11) | 58.89 (-18.33) | 68.89 (-8.33) |
| Llama-3-70b | 58.33 (-4.45) | 62.78 (0.00) | 66.67 (3.89) | 56.11 (-6.67) | 60.56 (-2.22) | 45.25 (-17.53) | 36.11 (-26.67) | 45.0 (-17.78) | 68.89 (6.11) |
| Llama-3-8b | 43.5 (-4.28) | 47.78 (0.00) | 37.78 (-10.00) | 39.33 (-8.45) | 46.11 (-1.67) | 34.18 (-13.60) | 34.62 (-13.16) | nan (nan) | 35.0 (-12.78) |
| Command-R+ | 37.22 (-1.33) | 38.55 (0.00) | 38.33 (-0.22) | 36.11 (-2.44) | 37.22 (-1.33) | 29.8 (-8.75) | 31.15 (-7.40) | 35.56 (-2.99) | 38.55 (0.00) |
| Openchat | 35.59 (0.03) | 35.56 (0.00) | 54.76 (19.20) | 36.11 (0.55) | 56.67 (21.11) | nan (nan) | nan (nan) | nan (nan) | 40.48 (4.92) |

表 2：不同语言的参考游戏。顶部，“% 完成”，衡量格式规则的遵循程度，底部，“质量评分”，衡量格式良好的游戏质量。括号中是与原始英文版本的差值。GPT-4：gpt-4-turbo-2024-04-09，Claude-3：claude-3-opus-20240229，Openchat：openchat-3.5-0106

## 5 挑战性：有成长空间

Chalamalasetti 等人（[2023](https://arxiv.org/html/2405.20859v1#bib.bib2)）“怀疑”人在 clembench 上的表现“接近上限”。我们对此假设进行了测试。在本文的作者中，我们进行了配对，以确保没有玩家参与了生成实例的游戏。由于在本文的工作中，所有玩家对所有游戏都有了充分的理解，我们认为结果得分代表的并不是平均人类表现，而是人类专家表现。我们在每个游戏中玩了 10 到 15 局（排除了 wordle-clue 和 wordle-critic，因为这些只是主 wordle 游戏的变种）。所有游戏都进行了到底，因此‘% played’ 对于人类玩家来说，毫不意外地是 100。结果的质量得分为：wordle：72，taboo：80.5；drawing：95.2；reference：100；从而平均得分为 86.93 —— 确实明显高于表 [1](https://arxiv.org/html/2405.20859v1#S2.T1 "Table 1 ‣ 2 Dialogue Games and Agent Capabilities ‣ clembench2024 A Challenging, Dynamic, Complementary, Multilingual Benchmark and Underlying Flexible Framework for LLMs as Multi-Action Agents") 中报告的最佳结果。

## 6 互补性：相关性

为了研究 clembench 测量值如何与基于参考的评估和基于偏好的评估之间的关系，我们分别计算了与 HELM（v1.3.0，2024-05-07；Liang 等人（[2022](https://arxiv.org/html/2405.20859v1#bib.bib9)））和 Chatbot Arena（CA；检索于 2024-05-16；Chiang 等人（[2024](https://arxiv.org/html/2405.20859v1#bib.bib3)））的排名相关性。与 CA，clembench 共享 30 个模型。排名高度相关，Kendall’s tau 为 0.65（$r_{\tau}$，$p<0.05$）。与 HELM，clembench 共享 18 个模型。相关性显著较低，为 0.39（$r_{\tau}$，$p<0.05$）。这个非常有趣的结果表明，clembench 与通过互动（Chatbot Arena）获得的结果更为相关 —— 而且实际上不需要人类互动，且完全离线运行。（有关排名关系的图形视图，请参见附录 [B](https://arxiv.org/html/2405.20859v1#A2 "Appendix B Across-Benchmark Correlations ‣ clembench2024 A Challenging, Dynamic, Complementary, Multilingual Benchmark and Underlying Flexible Framework for LLMs as Multi-Action Agents")。）

## 7 多语言：案例研究

clemgame框架中游戏规格与游戏逻辑的分离使得通过翻译游戏模板和游戏解析规则，能够轻松实现相同游戏的多语言版本。我们利用这一点来探讨上述测试模型子集的多语言能力，以参考游戏作为案例研究。（在这个游戏中，玩家A看到三个基于Unicode字符的5x5像素图像，并任务是描述第一个图像。玩家B看到相同的图像，可能顺序不同，并任务是识别被描述的图像。随机表现将导致33的质量评分。）我们选择了一组类型学上具有多样性的语言（参见附录[C](https://arxiv.org/html/2405.20859v1#A3 "Appendix C Language Tested in the Case Study ‣ clembench2024 A Challenging, Dynamic, Complementary, Multilingual Benchmark and Underlying Flexible Framework for LLMs as Multi-Action Agents")），并请母语者翻译提示语和目标表达式。表[2](https://arxiv.org/html/2405.20859v1#S4.T2 "Table 2 ‣ 4 Dynamic: Games, not Instances ‣ clembench2024 A Challenging, Dynamic, Complementary, Multilingual Benchmark and Underlying Flexible Framework for LLMs as Multi-Action Agents")展示了在非英语语言中进行游戏的影响。大型商业语言在遵循格式说明时表现良好（表格的顶部部分），llama3-70b也是如此。所有模型大多受到游戏玩法质量的影响。

我们将这些结果的更详细分析留待未来的工作中，这里仅指出，案例研究展示了clembench作为一个有前景的工具，能够研究多语言互动指令跟随能力的价值。

## 8 结论

在这篇简短的论文中，我们评估了一种最近提出的评估方法，该方法补充了现有的基于参考和偏好的方法。我们展示了它具有某些理想的特性，这些特性有望保持其相关性（因为它可以灵活适应新模型，并且其动态特性能有效抵抗数据污染的风险）。在框架中实现的游戏似乎处于一个有趣的水平：这些游戏对人类玩家来说并不特别具有挑战性，但即使是最优秀的模型，仍然保持挑战性。在一个案例研究中，我们展示了该方法也可以用来调查多语言能力。未来的工作可能会展示更多的应用场景，例如作为强化学习环境中的学习平台，或作为更多应用目标导向对话系统的开发环境。

## 参考文献

+   Andreas（2022）Jacob Andreas。2022年。[语言模型作为代理模型](https://doi.org/10.18653/v1/2022.findings-emnlp.423)。载于*计算语言学协会的研究成果：EMNLP 2022*，第5769-5779页，阿布扎比，阿联酋。计算语言学协会。

+   Chalamalasetti等人（2023）Kranti Chalamalasetti, Jana Götze, Sherzod Hakimov, Brielen Madureira, Philipp Sadler, 和 David Schlangen。2023年。[clembench：使用游戏玩法评估优化对话的语言模型作为对话代理](https://doi.org/10.18653/v1/2023.emnlp-main.689)。载于*2023年自然语言处理实证方法会议论文集*，第11174-11219页，新加坡。计算语言学协会。

+   Chiang等人（2024）Wei-Lin Chiang, Lianmin Zheng, Ying Sheng, Anastasios Nikolas Angelopoulos, Tianle Li, Dacheng Li, Hao Zhang, Banghua Zhu, Michael I. Jordan, Joseph E. Gonzalez, 和 Ion Stoica。2024年。[聊天机器人竞技场：通过人类偏好评估大语言模型的开放平台](https://doi.org/10.48550/ARXIV.2403.04132)。*CoRR*，abs/2403.04132。

+   Duan等人（2024）Jinhao Duan, Renming Zhang, James Diffenderfer, Bhavya Kailkhura, Lichao Sun, Elias Stengel-Eskin, Mohit Bansal, Tianlong Chen, 和Kaidi Xu。2024年。[Gtbench：通过博弈论评估揭示大语言模型的战略推理局限性](https://arxiv.org/abs/2402.12348)。*预印本*，arXiv:2402.12348。

+   Gong等人（2023）Ran Gong, Qiuyuan Huang, Xiaojian Ma, Hoi Vo, Zane Durante, Yusuke Noda, Zilong Zheng, Song-Chun Zhu, Demetri Terzopoulos, Li Fei-Fei, 和 Jianfeng Gao。2023年。[Mindagent：新兴的游戏互动](https://arxiv.org/abs/2309.09971)。*预印本*，arXiv:2309.09971。

+   Hendrycks等人（2021）Dan Hendrycks, Collin Burns, Steven Basart, Andy Zou, Mantas Mazeika, Dawn Song, 和 Jacob Steinhardt。2021年。[衡量大规模多任务语言理解](http://arxiv.org/abs/2009.03300)。载于*国际学习表征会议（ICLR）*论文集。ArXiv:2009.03300 [cs]。

+   Hudeček 和 Dusek（2023）Vojtěch Hudeček 和 Ondrej Dusek。2023年。[大语言模型是否能满足任务导向对话的所有需求？](https://doi.org/10.18653/v1/2023.sigdial-1.21) 载于*第24届语篇与对话特别兴趣小组年会论文集*，第216-228页，布拉格，捷克。计算语言学协会。

+   Li等人（2023）Jiatong Li, Rui Li, 和 Qi Liu。2023年。[超越静态数据集：一种深度交互方法评估大语言模型](https://arxiv.org/abs/2309.04369)。*预印本*，arXiv:2309.04369。

+   Liang等人（2022）Percy Liang, Rishi Bommasani, Tony Lee, Dimitris Tsipras, Dilara Soylu, Michihiro Yasunaga等。2022年。[语言模型的整体评估](https://doi.org/10.48550/arXiv.2211.09110)。*CoRR*，abs/2211.09110。

+   Magar 和 Schwartz (2022) Inbal Magar 和 Roy Schwartz. 2022. [数据污染：从记忆到利用](https://doi.org/10.18653/v1/2022.acl-short.18). 收录于 *第60届计算语言学协会年会论文集（第2卷：短篇论文）*，第157–165页，爱尔兰都柏林。计算语言学协会。

+   OpenAI 等人 (2024) OpenAI, Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat, Red Avila, Igor Babuschkin, Suchir Balaji, Valerie Balcom, Paul Baltescu, Haiming Bao, Mohammad Bavarian, Jeff Belgum, Irwan Bello, 等. 2024. [Gpt-4 技术报告](https://arxiv.org/abs/2303.08774). *预印本*，arXiv:2303.08774。

+   Qiao 等人 (2023) Dan Qiao, Chenfei Wu, Yaobo Liang, Juntao Li, 和 Nan Duan. 2023. [Gameeval: 在对话游戏中评估LLM](https://arxiv.org/abs/2308.10032). *预印本*，arXiv:2308.10032。

+   Schlangen (2023) David Schlangen. 2023. [对话游戏用于语言理解基准测试：动机、分类和策略](https://doi.org/10.48550/arXiv.2304.07007). *CoRR*，abs/2304.07007。

+   Sekulic 等人 (2024) Ivan Sekulic, Silvia Terragni, Victor Guimarães, Nghia Khau, Bruna Guedes, Modestas Filipavicius, Andre Ferreira Manso, 和 Roland Mathis. 2024. [基于LLM的可靠用户模拟器，用于任务导向对话系统](https://aclanthology.org/2024.scichat-1.3). 收录于 *第1届模拟聊天智能对话工作坊 (SCI-CHAT 2024)*，第19–35页，马耳他圣朱利安。计算语言学协会。

+   Wolf 等人 (2020) Thomas Wolf, Lysandre Debut, Victor Sanh, Julien Chaumond, Clement Delangue, Anthony Moi, Pierric Cistac, Tim Rault, Remi Louf, Morgan Funtowicz, Joe Davison, Sam Shleifer, Patrick von Platen, Clara Ma, Yacine Jernite, Julien Plu, Canwen Xu, Teven Le Scao, Sylvain Gugger, Mariama Drame, Quentin Lhoest, 和 Alexander Rush. 2020. [Transformers: 先进的自然语言处理](https://doi.org/10.18653/v1/2020.emnlp-demos.6). 收录于 *2020年自然语言处理实证方法会议：系统演示*，第38–45页，在线。计算语言学协会。

+   Wu 等人 (2024) Yue Wu, Xuan Tang, Tom M. Mitchell, 和 Yuanzhi Li. 2024. [Smartplay: 作为智能体的LLM基准测试](https://arxiv.org/abs/2310.01557). *预印本*，arXiv:2310.01557。

+   Zhou 等人 (2024) Xuhui Zhou, Hao Zhu, Leena Mathur, Ruohong Zhang, Haofei Yu, Zhengyang Qi, Louis-Philippe Morency, Yonatan Bisk, Daniel Fried, Graham Neubig, 和 Maarten Sap. 2024. [SOTOPIA: 语言代理中的社交智能互动评估](https://arxiv.org/abs/2310.11667). 收录于 *ICLR 2024会议论文集*，第1–45页。

## 附录 A 完整结果

| 模型 | sc | %pl | qs | o/g |
| --- | --- | --- | --- | --- |
| gpt-4 | 59.49 | 96.06 | 61.93 | $$ |
| claude-v1.3 | 37.07 | 74.76 | 49.58 | $$ |
| gpt-3.5-turbo | 37.02 | 85.86 | 43.12 | $$ |
| text-davinci-003 | 15.78 | 44.50 | 35.46 | $$ |
| vicuna-13b | 4.24 | 13.58 | 31.25 | ow |
| oasst-12b | 1.74 | 20.85 | 8.33 | ow |
| koala-13b | 1.48 | 14.76 | 10.00 | ow |
| falcon-40b | 0.71 | 0.95 | 75.00 | ow |
| luminous-supreme | 0.00 | 16.24 | 0.00 | $$ |
| 模型 | sc | %pl | qs | o/g |
| --- | --- | --- | --- | --- |
| gpt-4-0613 | 60.90 | 97.22 | 62.64 | $$ |
| gpt-4-1106-preview | 60.33 | 97.95 | 61.59 | $$ |
| gpt-4-0314 | 58.81 | 93.79 | 62.70 | $$ |
| claude-v1.3 | 37.64 | 74.24 | 50.70 | $$ |
| claude-2.1 | 36.38 | 83.08 | 43.79 | $$ |
| claude-2 | 33.71 | 82.12 | 41.05 | $$ |
| gpt-3.5-turbo-0613 | 32.53 | 91.96 | 35.37 | $$ |
| gpt-3.5-turbo-1106 | 30.45 | 77.12 | 39.49 | $$ |
| openchat_3.5 | 19.72 | 57.57 | 34.26 | ow |
| mistral-medium | 17.99 | 51.11 | 35.20 | ow |
| mixtral-8x7b-instruct-v0.1 | 17.81 | 60.49 | 29.44 | ow |
| openchat-3.5-1210 | 17.61 | 53.18 | 33.11 | ow |
| sheep-duck-llama-2-70b-v1.1 | 17.12 | 40.82 | 41.93 | ow |
| yi-34b-chat | 16.77 | 63.76 | 26.30 | ow |
| wizardlm-70b-v1.0 | 16.70 | 51.65 | 32.34 | ow |
| tulu-2-dpo-70b | 15.90 | 54.49 | 29.18 | ow |
| sus-chat-34b | 15.64 | 49.75 | 31.44 | ow |
| claude-instant-1.2 | 15.44 | 59.61 | 25.91 | $$ |
| openchat-3.5-0106 | 14.33 | 48.86 | 29.33 | ow |
| nous-hermes-2-mixtral-8x7b-dpo | 12.69 | 57.47 | 22.08 | ow |
| codellama-34b-instruct-hf | 10.34 | 23.96 | 43.15 | ow |
| vicuna-33b-v1.3 | 9.15 | 17.47 | 52.36 | ow |
| wizardlm-13b-v1.2 | 7.82 | 40.49 | 19.31 | ow |
| vicuna-13b-v1.5 | 7.21 | 34.74 | 20.74 | ow |
| sheep-duck-llama-2-13b | 6.74 | 34.86 | 19.34 | ow |
| vicuna-7b-v1.5 | 3.46 | 12.86 | 26.91 | ow |
| tulu-2-dpo-7b | 3.27 | 36.29 | 9.02 | ow |
| command | 3.12 | 10.01 | 31.13 | $$ |
| wizard-vicuna-13b-uncensored-hf | 2.06 | 9.49 | 21.71 | ow |
| llama-2-13b-chat-hf | 1.89 | 3.43 | 55.09 | ow |
| mistral-7b-instruct-v0.1 | 1.50 | 12.86 | 11.67 | ow |
| llama-2-70b-chat-hf | 1.39 | 3.79 | 36.74 | ow |
| koala-13b-hf | 1.25 | 23.22 | 5.38 | ow |
| zephyr-7b-beta | 1.23 | 3.95 | 31.25 | ow |
| deepseek-llm-67b-chat | 0.77 | 2.64 | 29.17 | ow |
| zephyr-7b-alpha | 0.75 | 7.51 | 10.00 | ow |
| llama-2-7b-chat-hf | 0.24 | 6.05 | 4.00 | ow |
| gpt4all-13b-snoozy | 0.00 | 2.92 | 0.00 | $$ |
| deepseek-llm-7b-chat | 0.00 | 7.44 | 0.00 | ow |
| oasst-sft-4-pythia-12b-epoch-3.5 | 0.00 | 14.76 | 0.00 | ow |
| falcon-7b-instruct | 0.00 | 14.29 | 0.00 | ow |
| 模型 | sc | %pl | qs | o/g |
| --- | --- | --- | --- | --- |
| gpt-4-turbo-2024-04-09 | 58.30 | 94.88 | 61.45 | $$ |
| gpt-4-0125-preview | 52.50 | 94.92 | 55.31 | $$ |
| gpt-4-1106-preview | 51.99 | 98.10 | 53.00 | $$ |
| gpt-4-0613 | 51.09 | 94.88 | 53.85 | $$ |
| gpt-4o-2024-05-13 | 48.34 | 85.71 | 56.40 | $$ |
| claude-3-opus-20240229 | 42.42 | 83.10 | 51.05 | $$ |
| gemini-1.5-pro-latest | 41.72 | 82.14 | 50.79 | $$ |
| llama-3-70b-instruct | 35.11 | 80.72 | 43.50 | ow |
| claude-2.1 | 32.50 | 82.14 | 39.57 | $$ |
| gemini-1.5-flash-latest | 32.00 | 76.14 | 42.03 | $$ |
| claude-3-sonnet-20240229 | 30.53 | 85.24 | 35.82 | $$ |
| qwen1.5-72b-chat | 30.37 | 80.05 | 37.94 | ow |
| mistral-large-2402 | 28.17 | 66.86 | 42.14 | $$ |
| gpt-3.5-turbo-0125 | 27.22 | 89.67 | 30.36 | $$ |
| gemini-1.0-pro | 26.95 | 80.14 | 33.63 | $$ |
| command-r-plus | 24.94 | 74.90 | 33.30 | ow |
| openchat_3.5 | 23.64 | 63.52 | 37.22 | ow |
| claude-3-haiku-20240307 | 22.49 | 79.52 | 28.28 | $$ |
| sheep-duck-llama-2-70b-v1.1 | 21.50 | 41.19 | 52.20 | ow |
| llama-3-8b-instruct | 19.99 | 76.10 | 26.27 | ow |
| openchat-3.5-1210 | 18.22 | 51.19 | 35.60 | ow |
| wizardlm-70b-v1.0 | 17.40 | 46.19 | 37.66 | ow |
| openchat-3.5-0106 | 17.10 | 52.57 | 32.52 | ow |
| qwen1.5-14b-chat | 16.80 | 40.95 | 41.02 | ow |
| mistral-medium-2312 | 16.43 | 49.25 | 33.36 | $$ |
| qwen1.5-32b-chat | 15.41 | 63.69 | 24.19 | ow |
| codegemma-7b-it | 15.30 | 51.95 | 29.45 | ow |
| dolphin-2.5-mixtral-8x7b | 15.10 | 46.38 | 32.55 | ow |
| codellama-34b-instruct | 14.35 | 33.57 | 42.76 | ow |
| command-r | 14.15 | 61.67 | 22.95 | ow |
| gemma-1.1-7b-it | 14.14 | 49.67 | 28.46 | ow |
| sus-chat-34b | 14.11 | 54.40 | 25.93 | ow |
| mixtral-8x22b-instruct-v0.1 | 12.69 | 52.14 | 24.33 | ow |
| tulu-2-dpo-70b | 12.62 | 49.76 | 25.37 | ow |
| nous-hermes-2-mixtral-8x7b-sft | 11.95 | 39.68 | 30.12 | ow |
| wizardlm-13b-v1.2 | 11.48 | 39.57 | 29.00 | ow |
| vicuna-33b-v1.3 | 11.27 | 23.81 | 47.32 | ow |
| mistral-7b-instruct-v0.2 | 9.75 | 36.91 | 26.42 | ow |
| yi-34b-chat | 8.27 | 40.86 | 20.25 | ow |
| mixtral-8x7b-instruct-v0.1 | 8.17 | 47.62 | 17.15 | ow |
| mistral-7b-instruct-v0.1 | 8.01 | 37.14 | 21.58 | ow |
| yi-1.5-34b-chat | 7.67 | 52.38 | 14.65 | ow |
| vicuna-13b-v1.5 | 7.01 | 39.52 | 17.73 | ow |
| yi-1.5-6b-chat | 6.73 | 41.43 | 16.25 | ow |
| starling-lm-7b-beta | 6.56 | 30.89 | 21.25 | ow |
| sheep-duck-llama-2-13b | 5.39 | 31.90 | 16.90 | ow |
| yi-1.5-9b-chat | 4.37 | 38.10 | 11.48 | ow |
| gemma-1.1-2b-it | 2.91 | 22.62 | 12.87 | ow |
| qwen1.5-7b-chat | 2.58 | 30.24 | 8.53 | ow |
| gemma-7b-it | 1.82 | 17.78 | 10.23 | ow |
| llama-2-70b-chat | 0.81 | 7.14 | 11.31 | ow |
| qwen1.5-0.5b-chat | 0.12 | 25.72 | 0.48 | ow |
| qwen1.5-1.8b-chat | 0.00 | 15.24 | 0.00 | ow |

表 3：按顺序列出 2023 年 6 月、2023 年 11 月、2024 年 5 月在英语 clembench 上的结果。 “sc” 是 clemscore， “%pl” 是正式正确游戏的百分比， “qs” 是这些游戏的游戏质量；“ow”：开放权重模型，“$$”：有门控模型。

查看表[3](https://arxiv.org/html/2405.20859v1#A1.T3 "表 3 ‣ 附录 A 完整结果 ‣ clembench2024 一个具有挑战性、动态、互补、多语言的基准测试和适用于大语言模型的灵活框架")。

## 附录 B 跨基准相关性

请参见图[1](https://arxiv.org/html/2405.20859v1#A2.F1 "图 1 ‣ 附录 B 跨基准相关性 ‣ clembench2024 一个具有挑战性、动态、互补的多语言基准及其灵活框架，适用于作为多行动体的LLMs")。

![请参阅图注](img/89c7cb132b34f7344d6917aef9d36e5d.png)![请参阅图注](img/5b3484a2c44842baa407570e600dcaf8.png)

图 1：上方：展示 clembench（左）与 Chatbot Arena（2024-05-16；右）之间排名差异的凸形图；下方：展示 clembench（左）与 HELM（v1.3.0；右）之间的排名差异

## 附录 C 案例研究中测试的语言

虽然关于商业模型在不同语言上所使用的训练数据量和优化情况并没有明确的信息，但Command-R+的模型卡提供了不同层次上支持语言的概览。⁴⁴4[https://huggingface.co/CohereForAI/c4ai-command-r-plus](https://huggingface.co/CohereForAI/c4ai-command-r-plus)我们从不同语言家族中选择了八种语言，其中五种是模型明确优化的：德语（de）、意大利语（it）、巴西葡萄牙语（pt）、日语（ja）和简体中文（zh）；一种训练数据中报告包含资源的语言：土耳其语（tr）；还有两种没有明确支持的语言：泰卢固语（te）和土库曼语（tk）。

关于GPT-4技术报告（OpenAI等，([2024](https://arxiv.org/html/2405.20859v1#bib.bib11))）并未包含有关训练数据中支持的语言的任何信息，但其评估中包含了模型在不同语言上的表现排名，这些评估基于对多项选择题的自动翻译版本（Hendrycks等，[2021](https://arxiv.org/html/2405.20859v1#bib.bib6)）。其中，我们选择的语言包括了他们评估中表现最好和最差的语言（分别是意大利语和泰卢固语）。同样，Claude-3模型卡（可在[https://www-cdn.anthropic.com/de8ba9b01c9ab7cbabf5c33b80b7bbc618857627/Model_Card_Claude_3.pdf](https://www-cdn.anthropic.com/de8ba9b01c9ab7cbabf5c33b80b7bbc618857627/Model_Card_Claude_3.pdf)上获取）也没有包含多语言训练数据的相关信息，但我们选择的语言同样涵盖了该模型评估的广泛语言（及更多）。
