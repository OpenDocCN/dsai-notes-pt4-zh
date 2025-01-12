<!--yml

分类: 未分类

日期: 2025-01-11 12:35:36

-->

# Auto-Arena：通过代理对战和委员会讨论自动化大语言模型评估

> 来源：[https://arxiv.org/html/2405.20267/](https://arxiv.org/html/2405.20267/)

Ruochen Zhao^(1,2)  , Wenxuan Zhang² , Yew Ken Chia^(2,3) ,

Weiwen Xu², Deli Zhao², Lidong Bing²

¹ 南洋理工大学，新加坡

² 阿里巴巴集团 DAMO Academy；³ 新加坡科技设计大学

ruochen002@e.ntu.edu.sg;

{saike.zwx, yewken.chia, xuweiwen.xww}@alibaba-inc.com {deli.zdl, l.bing}@alibaba-inc.com

[https://auto-arena.github.io/](https://auto-arena.github.io/)   本文工作是在作者作为 DAMO Academy, Alibaba Group 的实习生期间完成的。 Wenxuan Zhang 是通讯作者。 Yew Ken Chia 参与了 DAMO Academy 和 SUTD 的联合博士项目。

###### 摘要

随着大语言模型（LLMs）不断发展，迫切需要一种可靠的评估方法，能够迅速提供可信的结果。目前，静态基准测试存在不灵活和不可靠的问题，这导致用户更倾向于使用像 Chatbot Arena 这样的人工投票平台。然而，人工评估需要大量的手动工作。为了解决这一问题，我们提出了 Auto-Arena，一个创新的框架，利用大语言模型驱动的代理自动化整个评估过程。首先，LLM 考官生成问题。然后，两个 LLM 候选人基于个别问题进行多轮对战，旨在揭示它们真实的表现差异。最后，由 LLM 法官组成的委员会共同讨论并决定胜者，从而减少偏见并提高公正性。在对战过程中，我们观察到一些有趣的场景，其中 LLM 候选人展示出竞争行为，甚至从对手中学习。在我们涉及 15 个最新大语言模型的大规模实验中，Auto-Arena 与人工偏好的相关性达到 92.14%，超过了所有之前的专家标注基准，且无需任何手动努力。因此，Auto-Arena 为当前的人工评估平台提供了一个有前景的自动化评估大语言模型的替代方案。

## 1 引言

自从 ChatGPT 和 GPT-4（OpenAI 等，[2024](https://arxiv.org/html/2405.20267v4#bib.bib42)）流行以来，大语言模型（LLMs）已成为技术创新的前沿，吸引了广泛的行业和社会关注（Wu 等，[2023b](https://arxiv.org/html/2405.20267v4#bib.bib52)）。这一热情促使许多组织发布了自己的 LLM（Touvron 等，[2023](https://arxiv.org/html/2405.20267v4#bib.bib50)；Team 等，[2024b](https://arxiv.org/html/2405.20267v4#bib.bib49)）。然而，这些模型发布和更新的快速节奏对用户理解其能力并监控其演变带来了巨大挑战。因此，近期对于大语言模型的全面评估需求日益迫切（Chang 等，[2024a](https://arxiv.org/html/2405.20267v4#bib.bib10)）。

最流行的现有方法是使用静态数据集的自动评估。在这些方法中，静态数据集通常具有预定义的指标，如 GSM8k (Cobbe 等，[2021](https://arxiv.org/html/2405.20267v4#bib.bib15)) 和 MMLU (Hendrycks 等，[2021a](https://arxiv.org/html/2405.20267v4#bib.bib25))，这些数据集通过特定领域的输入输出对构建，比如人类考试类型的问题及其相应的答案。给定问题后，将 LLM 生成的答案与真实答案进行比较，使用诸如准确性等指标进行评估。这种方法可能会遭遇灵活性差、污染和人工标注成本高等问题。首先，封闭形式的真实答案限制了其在评估模型在一般性或开放式问题上的表现的有效性，而这些正是 LLM 的主要应用场景。由于问题是静态的，它们也可能遭遇污染问题（Ravaut 等，[2024](https://arxiv.org/html/2405.20267v4#bib.bib44)），即模型在训练过程中可能无意中接触到测试数据集的元素，从而扭曲评估结果。人工构建数据集的成本也很高，成为扩展到其他领域或语言的障碍。作为替代，基于模型评估的静态数据集，例如 MT-Bench (Zheng 等，[2023](https://arxiv.org/html/2405.20267v4#bib.bib56)) 和 AlpacaEval (Dubois 等，[2024a](https://arxiv.org/html/2405.20267v4#bib.bib20))，对 LLM 在开放式生成上的表现进行评估。这些方法通常让两个模型对相同的开放式问题生成回答，然后使用强大的判定模型（例如 GPT-4）选择更好的回答。然而，静态问题集仍然存在污染风险。此外，假设存在强大的判定模型使得评估框架的通用性降低，并引入了模型特定的偏差。

表 1：Auto-Arena 与其他基准或评估方法的比较。

|  | 问题 | 回答 | 判定者 |
| --- | --- | --- | --- |
| 方法 | 动态？ | 自动生成？ | 多轮？ | 开放式？ | 自动？ | 委员会？ |
| --- | --- | --- | --- | --- | --- | --- |
| OpenLLM 排行榜 | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| MMLU | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| GPQA | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| LC-AlpacaEval | ✗ | ✓ | ✗ | ✓ | ✓ | ✗ |
| MT-Bench | ✗ | ✗ | ✗ | ✓ | ✓ | ✗ |
| Arena-Hard | ✓ | ✗ | ✓ | ✓ | ✓ | ✗ |
| Chatbot Arena | ✓ | ✗ | ✓ | ✓ | ✗ | ✗ |
| Auto-Arena | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

除了自动化评估外，人工评估虽然需要大量的人工工作，但仍然是用户的金标准。一例值得注意的例子是Chatbot Arena（郑等人，[2023](https://arxiv.org/html/2405.20267v4#bib.bib56)），这是一个众包平台，收集关于大型语言模型（LLM）表现的匿名投票，并计算Elo评分（Elo & Sloan，[1978](https://arxiv.org/html/2405.20267v4#bib.bib22)）来对这些模型进行排名。最终得出的排行榜¹¹1  [https://leaderboard.lmsys.org/](https://leaderboard.lmsys.org/) 被广泛认为是LLM总体能力的可靠指标。然而，在该平台上进行可靠的模型评估必须依赖大量的人工投票，这需要相当多的时间和精力。因此，当新开发的模型进入平台时，它们通常很难迅速积累大量投票。此外，过度依赖人工投票限制了其在不同场景中的应用。例如，非英语语言的表现很难估计，因为平台上的大多数查询都是英语。此外，查询大多是单轮且简单的。完全开放的参与方式也可能导致评估质量不均。

为了实现既自动化又可靠的LLM评估，同时与人类偏好保持一致，我们提出了Auto-Arena框架，它通过LLM驱动的代理自动化整个LLM评估过程。该框架由三个阶段组成：首先，LLM考官代理负责生成问题，模拟现实生活中的用户发布查询。其次，两个LLM候选者相互互动，并通过单独回答种子问题、批评对方的弱点、提出针对性的后续问题以进一步挑战对方，进行多轮对战。在多轮对战过程中，LLM的真实能力被逐步展现，性能差距变得更加明显。最后，一个LLM评审委员会集体讨论并评估两个候选者的能力，模拟人类投票过程。如表格[1](https://arxiv.org/html/2405.20267v4#S1.T1 "Table 1 ‣ 1 Introduction ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")所示，Auto-Arena相比以往的评估方法具有几个关键优势：首先，Auto-Arena引入了动态的多轮对战，而不仅仅是简单的一轮问答，这能够展示LLM更深层次的能力，如推理、互动和战略规划。多轮对战的动态性质还减少了污染风险。其次，Auto-Arena通过将单一的LLM评审扩展为一个LLM评审委员会，缓解了潜在的模型特定评估偏差。最后，由于问题生成和评判过程完全自动化且端到端，Auto-Arena能够为新模型提供及时的评估，并且可以轻松扩展到各种领域和语言。

为了验证评估框架的可靠性和对齐性，我们对15个LLM进行了广泛的实验。与静态和基于模型的基准相比，Auto-Arena通过实现92.14%的Spearman相关系数与人类偏好的对齐，超越了所有之前的基准。尽管没有涉及人工努力，但与人类偏好的高度对齐可能源于模拟的类人评估过程，这一过程是通过LLM代理进行的。广泛的消融实验也证明了框架的可靠性：在同行对战前后，Spearman相关系数与人类偏好的相关性提高了5%，验证了我们的假设——同行对战能够更好地展示性能差距。在委员会讨论前后，委员会一致性提高了11%，显示出人类级别的一致性，并验证了委员会讨论机制的有效性。通过研究同行对战，我们还发现了LLM代理的有趣行为，如竞争性和自我改进行为。由于整个过程是自动化的，评估可以通过修改提示词轻松适应其他语言或领域。我们提供中文作为扩展到其他语言的案例研究。

总结而言，我们的贡献可以概括如下：

1.  1.

    我们提出了Auto-Arena，一个完全自动化的LLM评估框架，其中考官、候选人和评委都是由LLM驱动的代理模拟的；

1.  2.

    具体而言，我们创新性地利用同行对战进行LLM评估，其中两个LLM代理进行多轮辩论。这个过程挖掘了模型更深层次的能力；

1.  3.

    在我们对15个LLM进行的大规模实验中，我们观察到与人类偏好的最先进对齐，且无需任何人工干预；

1.  4.

    在同行对战中，LLM代理展现出有趣的行为，如制定策略和从对手学习，这为未来的研究开辟了可能性。

![参见标题](img/52a0601827b49784af4dd2f28f32d813.png)

图1：Auto-Arena的示意图。

## 2 Auto-Arena框架

如图[1](https://arxiv.org/html/2405.20267v4#S1.F1 "图1 ‣ 1 介绍 ‣ Auto-Arena：通过代理同行对战和委员会讨论自动化LLM评估")所示，Auto-Arena框架包括三个阶段：问题生成、多轮同行对战和委员会讨论。这三个阶段是顺序执行的，并完全由LLM驱动的代理模拟。所有提示词都包含在附录[A](https://arxiv.org/html/2405.20267v4#A1 "附录A 提示词使用 ‣ Auto-Arena：通过代理同行对战和委员会讨论自动化LLM评估")中。

### 2.1 问题生成

对于辩论问题，使用静态数据集可能会引发数据污染问题并导致不公平的评估，因此我们要求LLM考官代理动态生成问题。考官代理可以是任何具备能力的LLM。类似于MT-Bench（Zheng 等人，[2023](https://arxiv.org/html/2405.20267v4#bib.bib56)），生成的问题涵盖了现实生活对话中的8个常见类别：写作、角色扮演、提取、推理、数学、编码、STEM知识和人文学科/社会科学知识。考官会得到一个示例问题，并被鼓励生成多样且具有挑战性的问题，以确保评估的辩论具有深度和广度。生成问题的示例可见于附录[B](https://arxiv.org/html/2405.20267v4#A2 "附录B 生成问题示例 ‣ Auto-Arena:通过代理对战和委员会讨论自动化LLM评估")。

具体来说，由于考官代理也将参与后续的辩论，我们尝试通过两种设计来缓解自我增强偏差：1\. 我们不会告知考官它将参与此次比赛。2\. 以往的方法（Bai 等人，[2024](https://arxiv.org/html/2405.20267v4#bib.bib5)）可能会引发自我增强偏差，因为它们要求考官代理仅提出自己有信心解决的问题。相比之下，我们并未要求考官仅生成自己能够解决的问题。为了进一步证明有限的自我增强偏差存在，我们在附录[E](https://arxiv.org/html/2405.20267v4#A5 "附录E 自我增强偏差的消融研究 ‣ Auto-Arena:通过代理对战和委员会讨论自动化LLM评估")中加入了消融研究。

![参见说明](img/e842190e24d11cb76c1e62b30c2f6619.png)

图2：林肯-道格拉斯风格的同行对战过程及所使用的操作。<思考>操作可以由候选人自由使用，并且仅对候选人自己可见。

### 2.2 同行辩论

在问题生成后，我们围绕这些问题在LLM候选人之间进行同行对战。在一次同行对战中，两名LLM候选人（A和B）围绕给定的问题进行辩论，指出对手的弱点，并制定后续问题进一步探讨对手的弱点。

在同行对战中，每个候选LLM有四种可用操作类型：

+   •

    <思考>: 候选人就问题生成内部思考或制定策略。此操作可以随时使用，并且对对手保持隐蔽。

+   •

    <回应>: 候选人回答给定问题。

+   •

    <批评>: 候选人指出对手之前回答中的缺陷和错误。

+   •

    <提问>: 候选人提出后续问题，揭示对手的弱点。

对等对抗的工作流程采用的是林肯-道格拉斯辩论形式²²2  [https://en.wikipedia.org/wiki/Lincoln-Douglas_debate_format](https://en.wikipedia.org/wiki/Lincoln-Douglas_debate_format)。为了帮助用户更好地理解这一辩论形式，我们在[https://auto-chatbot-arena.streamlit.app/](https://auto-chatbot-arena.streamlit.app/)展示了辩论样本，这是在美国国家演讲与辩论协会等比赛中最广泛使用的单挑辩论风格。对等对抗由三轮组成，两位候选模型轮流发言。两位候选人都可以看到完整的对话历史。此过程在图[2](https://arxiv.org/html/2405.20267v4#S2.F2 "Figure 2 ‣ 2.1 Question Generation ‣ 2 The Auto-Arena Framework ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")中有描述。在第一轮中，模型A回答考官的初始问题；模型B批评A回答中的缺陷并提出具体的后续问题；然后模型A回答B的后续问题。第二轮采用相同的格式，A和B交换角色。在第三轮中，A和B进行交叉质询，A首先批评B早期回答中的漏洞并提出后续问题。回答后，模型B批评A的不足并提出额外问题。最后，模型A再次作出回应。在整个过程中，A和B的行动次数相等，以保持公平性。为了减少位置偏见，A和B的顺序在每轮辩论开始时会随机化。

在辩论过程中，增强偏见和污染担忧进一步减少：候选人之间相互提问的过程本质上是分散了问题生成的过程，从而减少了初始问题中生成的增强偏见。此外，辩论确保候选人不仅仅是对初始问题的回答得到评价，还要在更全面和更深层次的能力上进行评估，比如策略制定、批评对手和提出问题。换句话说，仅仅回答好初始问题并不一定能赢得整个辩论，这进一步减少了污染担忧。

根据每轮的不同，我们会为候选人提供行动指南，明确该轮的目标和相应的行动。类似于人类辩论比赛，我们会通过设置最大时长限制来计时候选人，具体时长也会在提示中说明。任何超出规定时长的回答将被截断。这种设计减少了LLM作为评委时的冗长偏见（Zheng等人，[2023](https://arxiv.org/html/2405.20267v4#bib.bib56)），即LLM评委偏好较长且冗长的回答。

### 2.3 委员会讨论

在对战结束后，由LLM评审委员会共同确定获胜者。委员会总是根据当前排名选出五个最佳LLM。为了减少偏见，我们排除了参与者本人以及与参与者同一家族的模型。例如，GPT-4不会作为评审来评估GPT-3.5参与的辩论。在第一轮中，委员会根据MMLU（Hendrycks等人，[2021a](https://arxiv.org/html/2405.20267v4#bib.bib25)）得分来初始化，以近似LLM的表现。每个评审单独阅读整个对战历史，阐述判断理由，并根据有用性、相关性和准确性等因素，决定A是否更好，B是否更好，或是否平局。

在初步判断形成后，委员会进行讨论。在讨论回合中，每个评审阅读其他评审在前几轮中的裁定，阐述自己的判断思路，并起草讨论后的裁定。在此过程中，评审可能决定调整或保持之前的判断。与展现多代理竞争的对战不同，委员会讨论部分融合了多代理协作方案。通过允许评审代理之间的互动和不同观点的交流，讨论使委员会形成集体智慧。因此，它提高了判断质量，增强了评审间的一致性，减少了单一模型的偏见。最后，胜者由经过讨论的裁定进行多数投票决定。

## 3 使用Auto-Arena推导可信排名

### 3.1 实验设置

模型选择： 在主要实验中，我们首先从Chatbot Arena平台的前30名列表中选择9个最佳或最新的模型，这些模型在实验时每个都有超过10k的投票：GPT-4-0409-Turbo，GPT-3.5-Turbo-0125，Claude-3-Haiku，Qwen1.5-72B-Chat，Command-R+，Llama-2-70B-Chat，Mixtral-8x7b-Instruct-v0.1，Yi-34B-Chat，和Deepseek-LLM-67B。为了构建排行榜，我们进一步添加了6个新发布的模型：GPT-4o-2024-05-13，Claude-3.5-Sonnet，Qwen2-72B-Instruct，Llama-3-70B，Gemma-2-27B，和Gemini-1.5-Flash。附录[G](https://arxiv.org/html/2405.20267v4#A7 "附录 G 主要实验的模型选择 ‣ Auto-Arena：通过代理对战和委员会讨论自动化LLM评估")提供了所选模型的详细列表。

基准： 对于基准，我们考虑了流行的评估标准，包括固定指标和基于模型的指标。附录[H](https://arxiv.org/html/2405.20267v4#A8 "附录 H 基准方法与Auto-Arena的比较 ‣ Auto-Arena：通过代理对战和委员会讨论自动化LLM评估")中显示了对比表。

1\. 基于固定评估指标的静态数据集：（1）OpenLLM Leaderboard （Beeching 等， [2023](https://arxiv.org/html/2405.20267v4#bib.bib6)），一个流行的开源模型基准，涵盖6个关键基准上的平均表现指标，涵盖大量不同的评估任务；（2）GPQA （Rein 等， [2023](https://arxiv.org/html/2405.20267v4#bib.bib46)），一个研究生级别的“谷歌防伪”问答基准，包含448个由领域专家编写的科学主题问题；（3）MMLU（大规模多任务语言理解）（Hendrycks 等， [2021a](https://arxiv.org/html/2405.20267v4#bib.bib25)），一个广泛的基准，涵盖57个学科，测试世界知识和问题解决能力；

2\. 基于模型评估指标的静态数据集：（1）MT-Bench （Zheng 等， [2023](https://arxiv.org/html/2405.20267v4#bib.bib56)），一组包含80个多轮问题的数据集。模型的回答由GPT-4进行评分；（2）Arena Hard （Li* 等， [2024](https://arxiv.org/html/2405.20267v4#bib.bib31)），一个包含1,000个具有挑战性用户查询的基准数据集，收集自聊天机器人竞技场。模型的回答由GPT-4-Turbo进行评分；（3）Length-Controlled AlpacaEval （Dubois 等， [2024a](https://arxiv.org/html/2405.20267v4#bib.bib20)），一个基于AlpacaFarm评估集（Dubois 等， [2024b](https://arxiv.org/html/2405.20267v4#bib.bib21)）的基准，测试模型是否能遵循一般用户指令。模型通过与GPT-4-Turbo的胜率进行评估，评分由GPT-4-Turbo完成。

设置：在9个参与者中，我们进行瑞士式锦标赛：对于$n$个参与者，瑞士锦标赛不是将每个参与者与$(n-1)$个其他人配对，而是将每个玩家与$\lceil log_{2}(n)\rceil$个相似排名的玩家配对，且不重复。这个设计有效地将排名$n$个模型的计算成本从$O(n^{2})$减少到$O(nlog_{2}(n))$。成本分析详见附录[H](https://arxiv.org/html/2405.20267v4#A8 "Appendix H Comparison of baseline methods and Auto-Arena ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")。

每对候选者都会进行40场对抗战，每场战斗包含来自8个任务类别中的每个类别的5个问题，这些类别在[2.1节](https://arxiv.org/html/2405.20267v4#S2.SS1 "2.1 问题生成 ‣ 2 自动竞技场框架 ‣ Auto-Arena：通过代理对抗和委员会讨论自动化LLM评估")中有所说明。我们提供的研究表明，生成的问题可以减少附带污染问题，详见附录[C](https://arxiv.org/html/2405.20267v4#A3 "附录C 污染分析 ‣ Auto-Arena：通过代理对抗和委员会讨论自动化LLM评估")，并且这些问题可以推广到现实场景，详见附录[D](https://arxiv.org/html/2405.20267v4#A4 "附录D 合成问题与真实问题对比 ‣ Auto-Arena：通过代理对抗和委员会讨论自动化LLM评估")。由于每场对抗包含3轮（每个候选者发言4次），因此比赛规模大致与MT-Bench相当（80个问题，每个候选者发言两次）。在比赛中，评分采用Elo评分系统计算（Bai et al., [2022](https://arxiv.org/html/2405.20267v4#bib.bib4); Boubdir et al., [2023](https://arxiv.org/html/2405.20267v4#bib.bib7)），该系统已成为象棋等竞技游戏中的标准做法（Elo & Sloan, [1978](https://arxiv.org/html/2405.20267v4#bib.bib22)）。与Chatbot Arena评分计算程序类似（Chiang et al., [2024](https://arxiv.org/html/2405.20267v4#bib.bib12)），我们计算Bradley-Terry（BT）系数（Bradley & Terry, [1952](https://arxiv.org/html/2405.20267v4#bib.bib8)），以便进行更好的统计估算。参考Zheng等人（[2023](https://arxiv.org/html/2405.20267v4#bib.bib56)）的参考引导评判标准，我们要求表现最好的评审为评估逻辑推理问题（数学、编程、推理）提供参考答案。

我们根据MMLU分数初始化瑞士赛排名，这是对模型表现的静态近似估算。在每轮配对结束时，我们重新计算当前模型的Elo分数。委员会成员根据当前Elo排名选出最佳的5个LLM。在初步判断后，委员会成员会进行一轮讨论。最终结果由讨论后的判断多数票决定。

表2：9个LLM的评估基准与Chatbot Arena Elo的相关性。

|  | Spearman |
| --- | --- |
|  | 相关性 |
| OpenLLM (Beeching et al., [2023](https://arxiv.org/html/2405.20267v4#bib.bib6)) | -15.39% |
| GPQA (Rein et al., [2023](https://arxiv.org/html/2405.20267v4#bib.bib46)) | 36.84% |
| MMLU (Hendrycks et al., [2021b](https://arxiv.org/html/2405.20267v4#bib.bib26)) | 56.36% |
| LC-AlpacaEval (Dubois et al., [2024a](https://arxiv.org/html/2405.20267v4#bib.bib20)) | 82.14% |
| MT-Bench (Zheng et al., [2023](https://arxiv.org/html/2405.20267v4#bib.bib56)) | 82.86% |
| Arena-Hard (Li* 等，[2024](https://arxiv.org/html/2405.20267v4#bib.bib31)) | 85.71% |
| *Auto-Arena* | 91.67% |
| 无对战 | 86.67% |
| 无委员会讨论 | 88.33% |

### 3.2 结果：与人类偏好的对齐

我们认为 Chatbot Arena 的得分是人类偏好和大规模语言模型（LLMs）一般能力的可信指标。表格 [2](https://arxiv.org/html/2405.20267v4#S3.T2 "Table 2 ‣ 3.1 Experimental Setup ‣ 3 Using Auto-Arena to Derive Trustworthy Rankings ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions") 显示了各种基准与 Chatbot Arena 得分的斯皮尔曼相关性。由于所有基准仅在英语中进行评估，我们使用仅限英语的 Chatbot Arena 得分。我们看到，无论是静态基准还是基于模型的基准，都产生了相似的相关性水平，低于 90%，其中 Arena-Hard 以 85.71% 超过其他基准。然后，Auto-Arena 可以将相关性提高到 91.67%，超越当前最先进技术（SOTA）5.96%。值得注意的是，在所有基准中，Auto-Arena 是唯一一个不需要人工参与的基准，无论是在数据集编制还是判断生成上。与人类偏好的高度一致性可能源于其类人设计，能够有效地模拟人类用户的投票过程。

### 3.3 对战和委员会讨论的消融研究

对战：  我们进行了一个消融研究，探讨对战是否会影响评估质量，并将结果包括在表格 [2](https://arxiv.org/html/2405.20267v4#S3.T2 "Table 2 ‣ 3.1 Experimental Setup ‣ 3 Using Auto-Arena to Derive Trustworthy Rankings ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")（“无对战”）。在这个设置中，我们要求委员会仅评估两个候选者对合成问题的初步回答，其中评委的提示保持不变。对于这个无辩论设计，问答过程模仿了 MT-Bench 或 LC-AlpacaEval，但加入了委员会讨论组件。结果，我们观察到该方法的相关性比 LC-AlpacaEval 和 MT-Bench 高出 3.81%。然而，与完整的 Auto-Arena 框架相比，性能下降了 5.00%。这证明了对战的有效性，在对战中，候选者之间的表现差异变得更加明显且对评委更具鲁棒性。因此，对战能够提高与人类偏好的对齐度。

![参见标题](img/b5992be4a319ec950ef26b420eaec862.png)

图 3：Cohen’s Kappa 一致性与多数投票结果的比较，委员会讨论前（上）和讨论后（下）。

表格 3：评委间的一致性概率。一致性被定义为两名随机评委相互同意的平均概率。

|  | 一致性 |
| --- | --- |
| Auto-Arena（讨论前） | 53% |
| Auto-Arena（讨论后） | 64% |
| MT-Bench 人类评估 | 67% |

委员会讨论：委员会讨论组件旨在引入不同的观点，并产生更一致的决策。如表 [2](https://arxiv.org/html/2405.20267v4#S3.T2 "表 2 ‣ 3.1 实验设置 ‣ 3 使用 Auto-Arena 导出可信排名 ‣ Auto-Arena：通过代理对战和委员会讨论自动化 LLM 评估") 所示，未进行委员会讨论时，与人类偏好的相关性从 91.67% 降低到 88.33%，显示了该组件在提高评估质量方面的有效性。如图 [3](https://arxiv.org/html/2405.20267v4#S3.F3 "图 3 ‣ 3.3 去除对战和委员会讨论的消融实验 ‣ 3 使用 Auto-Arena 导出可信排名 ‣ Auto-Arena：通过代理对战和委员会讨论自动化 LLM 评估") 所示，在进行委员会讨论之前，个别评审员与最终结果（投票结果）之间的 Cohen's Kappa 协议度（McHugh，[2012](https://arxiv.org/html/2405.20267v4#bib.bib37)）较低，平均为 0.41。具体来说，与强模型相比，弱模型的判断与投票结果的契合度较低，例如 Yi 与 GPT-4 相比。这表明，通用模型的能力可能导致在作为评审时出现显著的性能差距。经过委员会讨论后，协议度提高至平均 0.54，表明达到了中等程度的协议。在讨论过程中，评审员会接触到更多的观点，其中一些观点可能足够有说服力，导致判决发生变化。有关评审员间协议的更多分析，请参见附录 [F](https://arxiv.org/html/2405.20267v4#A6 "附录 F 评审员间协议 ‣ Auto-Arena：通过代理对战和委员会讨论自动化 LLM 评估")，我们可以看到，讨论也能大幅提高评审员之间的协议度。表 [3.3](https://arxiv.org/html/2405.20267v4#S3.SS3 "3.3 去除对战和委员会讨论的消融实验 ‣ 3 使用 Auto-Arena 导出可信排名 ‣ Auto-Arena：通过代理对战和委员会讨论自动化 LLM 评估") 显示了评审员之间的协议概率。协议概率定义为两名随机评审员达成一致的平均概率。经过委员会讨论后，协议概率提高了 11%，达到了人类标注员在 MT-Bench 上的协议水平。这一观察结果表明，委员会讨论可以显著提高判断质量，以匹配人类水平的表现。

## 4 使用 Auto-Arena 构建和维护排行榜

![参考说明](img/48ef779e22efc4f71624343ce57f8491.png)

图 4：将 Llama-3 添加到 9 个模型的排名中的 Elo 分数变化。

![参考说明](img/10b9b7cee3bfe3140707da43998e8a3d.png)

图 5：Auto-Arena 在英语上的 15 个模型的 Elo 分数。

### 4.1 更新新模型至排行榜

使用Auto-Arena，我们可以通过模型的Elo分数为模型列表获取排名，从而构建排行榜。由于新的LLM（大型语言模型）经常发布，我们描述了如何将最近发布的6个新候选模型添加到现有排行榜中，具体如第[3.1节](https://arxiv.org/html/2405.20267v4#S3.SS1 "3.1 实验设置 ‣ 3 使用Auto-Arena推导可信排名 ‣ Auto-Arena：通过代理对抗战斗和委员会讨论自动化LLM评估")中所列。为了添加一个新候选模型，我们要求它与$\lceil log_{2}(n)\rceil$个具有相似Elo分数的对手进行辩论，其中$n$是添加新候选模型后的总参与者数量。对于第一次配对，由于我们没有Elo指示器，我们通过让新候选模型与MMLU分数最相似的对手进行辩论来初始化。这个添加机制是通用的，并且在评估$n$个模型时能将计算成本保持在$nlog_{2}(n)$以下。

表4：在扩展后对15个LLM的评估基准与Chatbot Arena的相关性分析。

|  | 斯皮尔曼相关性 |
| --- | --- |
| OpenLLM | 32.50% |
| GPQA | 62.86% |
| MMLU | 46.20% |
| LC-AlpacaEval | 76.32% |
| MT-Bench | 88.73% |
| Arena-Hard | 45.36% |
| *Auto-Arena* | 92.14% |

例如，我们将一个新参与者（Llama-3-70B）添加到现有的9个模型排名中。它与$\lceil log_{2}(10)\rceil=4$个相似的对手进行辩论，图[5](https://arxiv.org/html/2405.20267v4#S4.F5 "图 5 ‣ 4 使用Auto-Arena构建和维护排行榜 ‣ Auto-Arena：通过代理对抗战斗和委员会讨论自动化LLM评估")展示了Elo分数在各回合中的变化。首先，它与Qwen-1.5基于MMLU相似度进行配对并获胜，从而获得了一个非常高的Elo分数，甚至超过了GPT-4。然后，它与Elo分数最接近的对手GPT-4配对。在失败后，它与其他Elo分数接近的对手（Command-R+和Claude-3-Haiku）进行配对。最终，分数稳定在第二位。这个过程使得新候选模型与合理比例的相似对手进行对战，并且在不打乱其他参与者排名的情况下，使最终排名稳定，其他参与者的分数分布在添加新候选模型前后保持相似。

使用这种可扩展的加法方法，我们通过向现有的9个LLM（大型语言模型）比赛中添加6个新模型，构建了一个全面的排行榜，最终形成了一个包含15个模型的排名。图[5](https://arxiv.org/html/2405.20267v4#S4.F5 "Figure 5 ‣ 4 Constructing and Maintaining a Leaderboard with Auto-Arena ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")展示了Auto-Arena对15个模型的总体Elo分数。表[4](https://arxiv.org/html/2405.20267v4#S4.T4 "Table 4 ‣ 4.1 Update New Models to Leaderboard ‣ 4 Constructing and Maintaining a Leaderboard with Auto-Arena ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")展示了扩展后的Spearman相关系数。Auto-Arena仍然是与人类偏好最一致的方法，领先3.41%，显示出92.14%的最先进对齐度。因此，Auto-Arena具有广泛的可推广性，并且在维护多LLM排行榜方面表现出色。

### 4.2 容易扩展到其他领域和语言

![请参阅标题说明](img/656ce70bdc52c25917415b72aef1e138.png)

图6：Auto-Arena对11个模型在中文上的Elo分数。

由于Auto-Arena对LLM的评估是完全自动化的，它可以轻松地适应用于评估其他领域或语言中的LLM。作为案例研究，我们对声称具有多语言能力的模型进行了中文比赛。唯一的适配工作是将提示语翻译成目标语言。然后，生成的问题和同行对战也会使用目标语言。该框架也可以适应其他任务或领域，唯一需要的工作是改变考官提示中的“领域”指定（如附录[A](https://arxiv.org/html/2405.20267v4#A1 "Appendix A Prompts Used ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")所示）。

图[6](https://arxiv.org/html/2405.20267v4#S4.F6 "图 6 ‣ 4.2 扩展到其他领域和语言的简单方法 ‣ 4 使用Auto-Arena构建和维护排行榜 ‣ Auto-Arena：通过代理对战和委员会讨论自动化LLM评估")展示了Auto-Arena为中文比赛得出的Elo评分，涵盖了11个模型。由于中文评估基准有限，我们与Chatbot Arena中的中文排行榜进行了对比，该排行榜占所有收集投票的10.36%。我们包括了Chatbot Arena前20名中来自每个主要模型家族的7个表现最佳和最新的模型。Auto-Arena通过92.86%的相关性恢复了它们的Elo评分，验证了扩展的可靠性。此外，由于Chatbot Arena不包括专有的中文LLM，我们添加了4个流行的中文LLM，它们分别是GLM³³3[https://open.bigmodel.cn/](https://open.bigmodel.cn/)，SenseChat⁴⁴4[https://platform.sensenova.cn/home](https://platform.sensenova.cn/home)，Minimax⁵⁵5[https://platform.minimaxi.com/examination-center/text-experience-center](https://platform.minimaxi.com/examination-center/text-experience-center)，和Wenxin⁶⁶6[https://cloud.baidu.com/wenxin.html](https://cloud.baidu.com/wenxin.html)。我们注意到，声称具备中文能力的模型，如Qwen-1.5，在该排行榜上的得分确实高于英语排行榜。

## 5 对LLM在竞争性对战中的行为进行调查

除了定量分析外，我们进一步深入探讨了对战，并发现了LLM代理在竞争环境中的一些有趣行为。

![请参见标题](img/380090ba6a0bbe7e8feaca57c7c4bf1e.png)

图7：候选者之间的性能差距在对战中变得可见。

#### 对战使得性能差距变得可见

在图[7](https://arxiv.org/html/2405.20267v4#S5.F7 "图7 ‣ 5 研究LLM在竞争性同伴对战中的行为 ‣ Auto-Arena：通过代理同伴对战和委员会讨论自动化LLM评估")中展示的示例中，给出了一个关于无限级数的数学问题，候选A（Claude-3-Haiku）和候选B（GPT-4-Turbo）在第一轮中都提供了正确答案。然而，随着辩论的深入，表现差距变得更加明显：候选B在解释初始答案背后的理论时，能够提供更详细和有帮助的回复。在没有同伴对战的消融研究中，评审初步判定为平局。然而，在看到后续辩论后，他们改变了看法，倾向于支持助手B。这个例子表明，辩论过程确实推动了候选LLM的能力极限，考验了更深层次的理解和推理能力。此外，正如之前的表[2](https://arxiv.org/html/2405.20267v4#S3.T2 "表2 ‣ 3.1 实验设置 ‣ 3 使用Auto-Arena推导可信排名 ‣ Auto-Arena：通过代理同伴对战和委员会讨论自动化LLM评估")所示，同伴对战对于一个稳健且全面的评估是不可或缺的。

![参见说明文字](img/9f83d5ba318286b3ff68a8eb05a3e061.png)

图8：LLM代理在同伴对战中表现出竞争行为。

![参见说明文字](img/b5469ee367fe773b0acce9f0047a9d9b.png)

图9：LLM代理在同伴对战中相互学习。

#### LLM可以巧妙地攻击对手

图[9](https://arxiv.org/html/2405.20267v4#S5.F9 "图9 ‣ 同伴对战揭示性能差距 ‣ 5 研究LLM在竞争性同伴对战中的行为 ‣ Auto-Arena：通过代理同伴对战和委员会讨论自动化LLM评估")中的示例展示了围绕问题“‘LETTER’中有多少种独特的字母排列方式？”进行的同伴对战片段。候选A（由Yi-34B-Chat驱动）给出了错误的初始答案，因为它错误地计算了重复字母的出现次数，并且错误地计算了阶乘。对手B（由Claude-3-Haiku驱动）迅速且准确地指出了这两个问题，并巧妙地提出了一个针对A弱点的后续问题：“那么‘BANANA’这个词呢？”然后，A仍然错误地计算了阶乘。我们看到，LLM候选者能够高效地理解竞争环境的规则，并能设计有针对性的策略来攻击对手以赢得胜利。在同伴对战中，辩论代理展示了有效的竞争策略，进一步探讨了对手的弱点。

#### LLM候选者可以通过向对手学习提高自己

图[9](https://arxiv.org/html/2405.20267v4#S5.F9 "图 9 ‣ 同行对战使得表现差距变得显而易见 ‣ 5 对大型语言模型（LLM）行为的竞争性同行对战调查 ‣ Auto-Arena：通过智能体同行对战和委员会讨论自动化 LLM 评估")展示了Claude-3-Haiku (A)与Command R+ (B)之间的角色扮演示例。在第一轮中，A只是简单地回答问题，而B除了回答问题外，还采用了合适的语言风格，更好地符合“角色扮演”的指示。随后，在没有明确指示的情况下，A从对手那里学习，并且也开始采纳这种语言风格。这个案例展示了一个有趣的观察，即使在竞争性环境中，LLM候选模型也能表现出学习行为，并且通过互动不断改进。基于这一观察，利用LLM智能体之间的互动来提升表现，可能成为未来学习的一种有前景的范式。

## 6 相关工作

随着LLM的快速发展，得出可信的评估结果已成为一项挑战。当前的评估方法可以分为自动评估和人工评估，例如Chatbot Arena（Chiang等人，[2024](https://arxiv.org/html/2405.20267v4#bib.bib12)）。我们主要关注自动评估，因为它们能够提供更及时的反馈。自动评估主要包括静态数据集、预定义的评估指标和基于模型的评估指标。静态数据集和预定义的评估指标，如MMLU（Hendrycks等人，[2021a](https://arxiv.org/html/2405.20267v4#bib.bib25)）、GPQA（Rein等人，[2023](https://arxiv.org/html/2405.20267v4#bib.bib46)）和Open-LLM-Leaderboard（Beeching等人，[2023](https://arxiv.org/html/2405.20267v4#bib.bib6)）由专家注释的问题-答案对组成。然后，基于如准确度等表现指标对模型进行评估。然而，由于它们仅评估封闭式答案，因此在评估开放性回答时缺乏灵活性。此外，静态数据集最终可能会被暴露在互联网上，从而引发污染问题（Ravaut等人，[2024](https://arxiv.org/html/2405.20267v4#bib.bib44)）。

相反，基于模型的静态数据集提供了一种灵活、低成本且快速的评估范式（Chang等人，[2024b](https://arxiv.org/html/2405.20267v4#bib.bib11)）。研究已验证，LLM能够提供无偏的（Ning等人，[2024](https://arxiv.org/html/2405.20267v4#bib.bib39)；Chu等人，[2024](https://arxiv.org/html/2405.20267v4#bib.bib13)）、高质量的（Lin & Chen，[2023](https://arxiv.org/html/2405.20267v4#bib.bib35)）指标，这些指标与人类评估相当（Dubois等人，[2024a](https://arxiv.org/html/2405.20267v4#bib.bib20)；Zheng等人，[2023](https://arxiv.org/html/2405.20267v4#bib.bib56)）。其中，MT-Bench（Zheng等人，[2023](https://arxiv.org/html/2405.20267v4#bib.bib56)）和AlpacaEval（Dubois等人，[2024a](https://arxiv.org/html/2405.20267v4#bib.bib20)）使用LLM作为评判者，要求GPT-4将模型响应与静态数据集中的问题进行比较。模型的判断与人类偏好的符合度超过80%，证明了使用LLM评估响应质量的可行性。Language-Model-as-an-Examiner（Bai等人，[2024](https://arxiv.org/html/2405.20267v4#bib.bib5)）要求一个语言模型考官在其记忆中构建知识密集型问题，与候选人进行一系列后续提问，并从准确性和事实性等维度对响应进行评分。KIEval（Yu等人，[2024](https://arxiv.org/html/2405.20267v4#bib.bib53)）还引入了一个由LLM驱动的“互动者”角色，来考察知识的深度理解，已证明能够缓解静态数据集上的污染问题。然而，这种单一评判者的评估方式要求考官与每个候选人并行互动，产生计算开销并限制了查询的范围。它们还受到单一模型偏差的影响，包括对LLM生成摘要的偏好（Liu等人，[2023](https://arxiv.org/html/2405.20267v4#bib.bib36)）、多语言评估中的分数膨胀（Hada等人，[2023](https://arxiv.org/html/2405.20267v4#bib.bib24)）、冗长偏差（Dubois等人，[2024a](https://arxiv.org/html/2405.20267v4#bib.bib20)）以及评估表现接近的候选人时的困难（Shen等人，[2023](https://arxiv.org/html/2405.20267v4#bib.bib47)）。因此，已有研究采用多主体评估来缓解单一模型偏差。例如，DRPE（Wu等人，[2023a](https://arxiv.org/html/2405.20267v4#bib.bib51)）使用多角色扮演提示来模拟同一LLM的不同角色，并将输出整合为最终结果的投票。PRD（Li等人，[2023a](https://arxiv.org/html/2405.20267v4#bib.bib30)）允许两个LLM讨论评估，并将更强能力的LLM评审员赋予更高的投票权重。研究表明，多主体方法有效地缓解了单一模型偏差。这一研究方向与我们“LLM评审委员会”组件相似。然而，它们仍然局限于静态数据集和特定领域。

在LLM评估领域之外，一些研究探讨了多代理LLM系统中的竞争行为，这与Auto-Arena中的对抗性较量密切相关。LM与LM（Cohen 等，[2023](https://arxiv.org/html/2405.20267v4#bib.bib16)）表明，LLM交叉检验可以有效地发现事实错误。辩论（Du 等，[2023](https://arxiv.org/html/2405.20267v4#bib.bib19)）表明，多代理辩论可以提高事实性和推理能力。在MAD（Liang 等，[2023](https://arxiv.org/html/2405.20267v4#bib.bib34)）中，LLM辩论可以鼓励发散性思维，这有助于需要深度思考的任务。Khan 等（[2024](https://arxiv.org/html/2405.20267v4#bib.bib28)）表明，即使是非专家的弱LLM，如果允许两位LLM专家进行辩论，也能监督专家LLM。此外，Zhao 等（[2023](https://arxiv.org/html/2405.20267v4#bib.bib55)）和Gu 等（[2024](https://arxiv.org/html/2405.20267v4#bib.bib23)）展示了一些有趣的案例研究，LLM参与了模拟的竞争环境，并展示了类人策略。

## 7 结论

在本文中，我们创新性地设计了一个完全自动化的评估框架：Auto-Arena。通过使用LLM代理生成问题，采用LLM候选者进行对抗性较量，并通过LLM委员会讨论评估回答，Auto-Arena提供及时且可信的评估，并以端到端的方式自动化评估过程。在广泛的实验中，Auto-Arena在无需人工干预的情况下，达到了与人类偏好的最高相关性。它易于适应其他领域和资源，促进了AI系统评估的包容性。对抗性较量还展示了LLM在竞争环境中的一些有趣行为，包括攻击和向对手学习。

## 参考文献

+   AI 等 (2024) 01. AI，:，Alex Young，Bei Chen，Chao Li，Chengen Huang，Ge Zhang，Guanwei Zhang，Heng Li，Jiangcheng Zhu，Jianqun Chen，Jing Chang，Kaidong Yu，Peng Liu，Qiang Liu，Shawn Yue，Senbin Yang，Shiming Yang，Tao Yu，Wen Xie，Wenhao Huang，Xiaohui Hu，Xiaoyi Ren，Xinyao Niu，Pengcheng Nie，Yuchi Xu，Yudong Liu，Yue Wang，Yuxuan Cai，Zhenyu Gu，Zhiyuan Liu 和 Zonghong Dai。Yi：01.ai推出的开源基础模型，2024年。

+   Anthropic (2024) Anthropic。推出下一代Claude。[https://www.anthropic.com/news/claude-3-family](https://www.anthropic.com/news/claude-3-family)，2024年。

+   Bai 等人 (2023) Jinze Bai, Shuai Bai, Yunfei Chu, Zeyu Cui, Kai Dang, Xiaodong Deng, Yang Fan, Wenbin Ge, Yu Han, Fei Huang, Binyuan Hui, Luo Ji, Mei Li, Junyang Lin, Runji Lin, Dayiheng Liu, Gao Liu, Chengqiang Lu, Keming Lu, Jianxin Ma, Rui Men, Xingzhang Ren, Xuancheng Ren, Chuanqi Tan, Sinan Tan, Jianhong Tu, Peng Wang, Shijie Wang, Wei Wang, Shengguang Wu, Benfeng Xu, Jin Xu, An Yang, Hao Yang, Jian Yang, Shusheng Yang, Yang Yao, Bowen Yu, Hongyi Yuan, Zheng Yuan, Jianwei Zhang, Xingxuan Zhang, Yichang Zhang, Zhenru Zhang, Chang Zhou, Jingren Zhou, Xiaohuan Zhou, 和 Tianhang Zhu. Qwen 技术报告. *arXiv 预印本 arXiv:2309.16609*, 2023.

+   Bai 等人 (2022) Yuntao Bai, Andy Jones, Kamal Ndousse, Amanda Askell, Anna Chen, Nova DasSarma, Dawn Drain, Stanislav Fort, Deep Ganguli, Tom Henighan, Nicholas Joseph, Saurav Kadavath, Jackson Kernion, Tom Conerly, Sheer El-Showk, Nelson Elhage, Zac Hatfield-Dodds, Danny Hernandez, Tristan Hume, Scott Johnston, Shauna Kravec, Liane Lovitt, Neel Nanda, Catherine Olsson, Dario Amodei, Tom Brown, Jack Clark, Sam McCandlish, Chris Olah, Ben Mann, 和 Jared Kaplan. 使用人类反馈强化学习训练一个有帮助且无害的助手, 2022.

+   Bai 等人 (2024) Yushi Bai, Jiahao Ying, Yixin Cao, Xin Lv, Yuze He, Xiaozhi Wang, Jifan Yu, Kaisheng Zeng, Yijia Xiao, Haozhe Lyu 等人. 基准测试语言模型作为考试者的基础模型. *神经信息处理系统进展*, 36, 2024.

+   Beeching 等人 (2023) Edward Beeching, Clémentine Fourrier, Nathan Habib, Sheon Han, Nathan Lambert, Nazneen Rajani, Omar Sanseviero, Lewis Tunstall, 和 Thomas Wolf. 开放 LLM 排行榜. [https://huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard](https://huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard), 2023.

+   Boubdir 等人 (2023) Meriem Boubdir, Edward Kim, Beyza Ermis, Sara Hooker, 和 Marzieh Fadaee. Elo 揭示：语言模型评估中的鲁棒性和最佳实践. 在 Sebastian Gehrmann, Alex Wang, João Sedoc, Elizabeth Clark, Kaustubh Dhole, Khyathi Raghavi Chandu, Enrico Santus, 和 Hooman Sedghamiz (编辑), *第三届自然语言生成、评估和度量研讨会（GEM）论文集*，第 339–352 页，新加坡，2023 年 12 月。计算语言学协会. URL [https://aclanthology.org/2023.gem-1.28](https://aclanthology.org/2023.gem-1.28).

+   Bradley & Terry (1952) Ralph Allan Bradley 和 Milton E Terry. 不完全区组设计的秩分析：I. 配对比较法. *生物统计学*, 39(3/4):324–345, 1952.

+   Brown 等人（2020）Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, Sandhini Agarwal, Ariel Herbert-Voss, Gretchen Krueger, Tom Henighan, Rewon Child, Aditya Ramesh, Daniel Ziegler, Jeffrey Wu, Clemens Winter, Chris Hesse, Mark Chen, Eric Sigler, Mateusz Litwin, Scott Gray, Benjamin Chess, Jack Clark, Christopher Berner, Sam McCandlish, Alec Radford, Ilya Sutskever 和 Dario Amodei. 语言模型是少样本学习者。在 H. Larochelle, M. Ranzato, R. Hadsell, M.F. Balcan 和 H. Lin（编），*神经信息处理系统进展*，第33卷，第1877-1901页，Curran Associates, Inc.，2020年。URL [https://proceedings.neurips.cc/paper_files/paper/2020/file/1457c0d6bfcb4967418bfb8ac142f64a-Paper.pdf](https://proceedings.neurips.cc/paper_files/paper/2020/file/1457c0d6bfcb4967418bfb8ac142f64a-Paper.pdf)。

+   Chang 等人（2024a）Yupeng Chang, Xu Wang, Jindong Wang, Yuan Wu, Linyi Yang, Kaijie Zhu, Hao Chen, Xiaoyuan Yi, Cunxiang Wang, Yidong Wang, Wei Ye, Yue Zhang, Yi Chang, Philip S. Yu, Qiang Yang 和 Xing Xie. 大型语言模型评估的综述。*ACM 智能系统技术学报*，15(3)，2024年3月。ISSN 2157-6904。doi: 10.1145/3641289。URL [https://doi.org/10.1145/3641289](https://doi.org/10.1145/3641289)。

+   Chang 等人（2024b）Yupeng Chang, Xu Wang, Jindong Wang, Yuan Wu, Linyi Yang, Kaijie Zhu, Hao Chen, Xiaoyuan Yi, Cunxiang Wang, Yidong Wang, Wei Ye, Yue Zhang, Yi Chang, Philip S. Yu, Qiang Yang 和 Xing Xie. 大型语言模型评估的综述。*ACM 智能系统技术学报*，15(3)，2024年3月。ISSN 2157-6904。doi: 10.1145/3641289。URL [https://doi.org/10.1145/3641289](https://doi.org/10.1145/3641289)。

+   Chiang 等人（2024）Wei-Lin Chiang, Lianmin Zheng, Ying Sheng, Anastasios Nikolas Angelopoulos, Tianle Li, Dacheng Li, Hao Zhang, Banghua Zhu, Michael Jordan, Joseph E. Gonzalez 和 Ion Stoica. Chatbot arena: 一个通过人类偏好评估大型语言模型的开放平台，2024年。

+   Chu 等人（2024）Zhumin Chu, Qingyao Ai, Yiteng Tu, Haitao Li 和 Yiqun Liu. Pre: 基于同行评审的大型语言模型评估器。*arXiv 预印本 arXiv:2401.15641*，2024年。

+   Clark 等人（2018）Peter Clark, Isaac Cowhey, Oren Etzioni, Tushar Khot, Ashish Sabharwal, Carissa Schoenick 和 Oyvind Tafjord. 认为你已经解决了问答问题？试试 ARC，AI2 推理挑战，2018年。URL [https://arxiv.org/abs/1803.05457](https://arxiv.org/abs/1803.05457)。

+   Cobbe 等人（2021）Karl Cobbe, Vineet Kosaraju, Mohammad Bavarian, Mark Chen, Heewoo Jun, Lukasz Kaiser, Matthias Plappert, Jerry Tworek, Jacob Hilton, Reiichiro Nakano, Christopher Hesse 和 John Schulman. 训练验证者解决数学文字问题。*arXiv 预印本 arXiv:2110.14168*，2021年。

+   Cohen 等人（2023）Roi Cohen, May Hamri, Mor Geva 和 Amir Globerson. Lm vs lm: 通过交叉审查检测事实错误。*arXiv 预印本 arXiv:2305.13281*，2023年。

+   Cohere (2024) Cohere. 推出 Command R+：为商业打造的可扩展LLM。 [https://cohere.com/blog/command-r-plus-microsoft-azure](https://cohere.com/blog/command-r-plus-microsoft-azure)，2024年。

+   DeepSeek-AI 等人 (2024) DeepSeek-AI, :, Xiao Bi, Deli Chen, Guanting Chen, Shanhuang Chen, Damai Dai, Chengqi Deng, Honghui Ding, Kai Dong, Qiushi Du, Zhe Fu, Huazuo Gao, Kaige Gao, Wenjun Gao, Ruiqi Ge, Kang Guan, Daya Guo, Jianzhong Guo, Guangbo Hao, Zhewen Hao, Ying He, Wenjie Hu, Panpan Huang, Erhang Li, Guowei Li, Jiashi Li, Yao Li, Y. K. Li, Wenfeng Liang, Fangyun Lin, A. X. Liu, Bo Liu, Wen Liu, Xiaodong Liu, Xin Liu, Yiyuan Liu, Haoyu Lu, Shanghao Lu, Fuli Luo, Shirong Ma, Xiaotao Nie, Tian Pei, Yishi Piao, Junjie Qiu, Hui Qu, Tongzheng Ren, Zehui Ren, Chong Ruan, Zhangli Sha, Zhihong Shao, Junxiao Song, Xuecheng Su, Jingxiang Sun, Yaofeng Sun, Minghui Tang, Bingxuan Wang, Peiyi Wang, Shiyu Wang, Yaohui Wang, Yongji Wang, Tong Wu, Y. Wu, Xin Xie, Zhenda Xie, Ziwei Xie, Yiliang Xiong, Hanwei Xu, R. X. Xu, Yanhong Xu, Dejian Yang, Yuxiang You, Shuiping Yu, Xingkai Yu, B. Zhang, Haowei Zhang, Lecong Zhang, Liyue Zhang, Mingchuan Zhang, Minghua Zhang, Wentao Zhang, Yichao Zhang, Chenggang Zhao, Yao Zhao, Shangyan Zhou, Shunfeng Zhou, Qihao Zhu 和 Yuheng Zou. Deepseek llm：通过长远主义扩展开源语言模型，2024年。

+   Du 等人 (2023) Yilun Du, Shuang Li, Antonio Torralba, Joshua B Tenenbaum 和 Igor Mordatch. 通过多主体辩论改善语言模型的事实性和推理能力。 *arXiv 预印本 arXiv:2305.14325*，2023年。

+   Dubois 等人 (2024a) Yann Dubois, Balázs Galambosi, Percy Liang 和 Tatsunori B Hashimoto. 长度控制的 alpacaeval：一种简单的去偏自动评估方法。 *arXiv 预印本 arXiv:2404.04475*，2024年。

+   Dubois 等人 (2024b) Yann Dubois, Xuechen Li, Rohan Taori, Tianyi Zhang, Ishaan Gulrajani, Jimmy Ba, Carlos Guestrin, Percy Liang 和 Tatsunori B. Hashimoto. Alpacafarm：一种模拟框架，用于学习人类反馈的方法，2024年。网址 [https://arxiv.org/abs/2305.14387](https://arxiv.org/abs/2305.14387)。

+   Elo 和 Sloan (1978) Arpad E Elo 和 Sam Sloan. 国际象棋选手评级：过去与现在。 *(无标题)*，1978年。

+   Gu 等人 (2024) Zhouhong Gu, Xiaoxuan Zhu, Haoran Guo, Lin Zhang, Yin Cai, Hao Shen, Jiangjie Chen, Zheyu Ye, Yifei Dai, Yan Gao, Yao Hu, Hongwei Feng 和 Yanghua Xiao. Agentgroupchat：一种互动式群聊模拟器，用于更好地引发突现行为，2024年。

+   Hada 等人 (2023) Rishav Hada, Varun Gumma, Adrian de Wynter, Harshita Diddee, Mohamed Ahmed, Monojit Choudhury, Kalika Bali 和 Sunayana Sitaram. 基于大型语言模型的评估工具是扩展多语言评估的解决方案吗？ *arXiv 预印本 arXiv:2309.07462*，2023年。

+   Hendrycks 等人（2021a）丹·亨德里克斯、科林·伯恩斯、史蒂文·巴萨特、安迪·周、曼塔斯·梅泽卡、道恩·宋、雅各布·斯坦哈特。 测量大规模多任务语言理解。*国际学习表示会议（ICLR）论文集*，2021a。

+   Hendrycks 等人（2021b）丹·亨德里克斯、科林·伯恩斯、索拉夫·卡达瓦斯、阿库尔·阿罗拉、史蒂文·巴萨特、埃里克·唐、道恩·宋、雅各布·斯坦哈特。 用数学数据集测量数学问题解决能力，2021b。网址 [https://arxiv.org/abs/2103.03874](https://arxiv.org/abs/2103.03874)。

+   Jiang 等人（2024）阿尔伯特·Q·蒋、亚历山大·萨布莱罗尔、安托万·鲁、阿瑟·孟什、布兰奇·萨瓦里、克里斯·班福德、德文德拉·辛格·查普洛特、迭戈·德·拉斯·卡萨斯、艾玛·布·哈娜、弗洛里安·布雷桑、吉安娜·伦吉尔、吉约姆·布尔、吉约姆·兰普尔、莱里奥·勒纳尔·拉沃、卢西尔·索尔尼耶、玛丽-安妮·拉绍、皮埃尔·斯托克、桑迪普·苏布拉马尼安、索菲亚·杨、谢门·安东尼亚克、特文·勒·斯卡奥、特奥菲尔·热尔维特、蒂博·拉夫里尔、托马斯·王、蒂莫泰·拉克鲁瓦、威廉·埃尔·赛义德。 Mixtral 专家系统，2024年。

+   Khan 等人（2024）阿克比尔·汗、约翰·休斯、丹·瓦伦丁、劳拉·鲁伊斯、克什提吉·萨昌、安什·拉达克里希南、爱德华·格雷芬斯特、塞缪尔·R·鲍曼、蒂姆·罗克塔谢尔、伊桑·佩雷斯。 与更具说服力的大语言模型辩论会带来更真实的答案，2024年。

+   Lee 等人（2024）阿里尔·N·李、科尔·J·亨特、纳塔尼尔·鲁伊兹。 Platypus：快速、廉价且强大的大语言模型精细化，2024年。网址 [https://arxiv.org/abs/2308.07317](https://arxiv.org/abs/2308.07317)。

+   Li 等人（2023a）李若森、特尔特·帕特尔、杜欣雅。 Prd：同行排名和讨论改善基于大语言模型的评估。*arXiv 预印本 arXiv:2307.02762*，2023a。

+   Li* 等人（2024）李天乐*、蒋伟霖*、埃文·弗里克、丽莎·邓拉普、朱邦华、约瑟夫·E·冈萨雷斯、伊昂·斯托伊卡。 从实时数据到高质量基准：Arena-Hard管道，2024年4月。网址 [https://lmsys.org/blog/2024-04-19-arena-hard/](https://lmsys.org/blog/2024-04-19-arena-hard/)。

+   Li 等人（2023b）李宇成、弗兰克·吉林、林成华。 Latesteval：通过动态和时间敏感的测试构建解决语言模型评估中的数据污染问题。 在*AAAI人工智能会议*，2023b。网址 [https://api.semanticscholar.org/CorpusID:266362809](https://api.semanticscholar.org/CorpusID:266362809)。

+   Li 等人（2024）李宇成、弗兰克·吉林、林成华。 面向大语言模型的开源数据污染报告，2024年。网址 [https://arxiv.org/abs/2310.17589](https://arxiv.org/abs/2310.17589)。

+   Liang 等人（2023）梁天、何志伟、焦文翔、王星、王岩、王瑞、杨宇久、屠兆鹏、石树铭。 通过多代理辩论鼓励大语言模型的发散性思维，2023年。

+   Lin & Chen (2023) 颜廷·林和云农·陈。LLM-eval：针对开放域对话的统一多维度自动评估，基于大型语言模型。在云农·陈和阿比纳夫·拉斯托吉（编辑），*第五届对话式AI自然语言处理研讨会（NLP4ConvAI 2023）论文集*，第47–58页，多伦多，加拿大，2023年7月。计算语言学协会。doi：10.18653/v1/2023.nlp4convai-1.5。网址[https://aclanthology.org/2023.nlp4convai-1.5](https://aclanthology.org/2023.nlp4convai-1.5)。

+   Liu et al. (2023) 杨刘，丹·伊特，亦冲·徐，硕杭·王，若晨·徐，和成光·朱。Gpteval：使用GPT-4进行的NLG评估，具有更好的人工对齐。*arXiv预印本arXiv:2303.16634*，2023年。

+   McHugh (2012) Mary L McHugh. 评估者间一致性：kappa统计量。*生物化学医学*，22(3)：276–282，2012年。

+   Meta (2024) Meta。推出Meta Llama 3：迄今为止最强大的公开可用LLM。[https://ai.meta.com/blog/meta-llama-3/](https://ai.meta.com/blog/meta-llama-3/)，2024年。

+   Ning et al. (2024) 昆鹏·宁，朔·杨，宇阳·刘，家宇·姚，振辉·刘，宇·王，名·庞，和力·袁。Peer-review-in-llms：开放环境中LLMs的自动评估方法。*arXiv预印本arXiv:2402.01830*，2024年。

+   Openai (2024a) Openai. 新的嵌入模型和API更新。[https://openai.com/index/new-embedding-models-and-api-updates/](https://openai.com/index/new-embedding-models-and-api-updates/)，2024a年。

+   Openai (2024b) Openai. 你好，GPT-4o。[https://openai.com/index/hello-gpt-4o/](https://openai.com/index/hello-gpt-4o/)，2024b年。

+   OpenAI 等人（2024）OpenAI、Josh Achiam、Steven Adler、Sandhini Agarwal、Lama Ahmad、Ilge Akkaya、Florencia Leoni Aleman、Diogo Almeida、Janko Altenschmidt、Sam Altman、Shyamal Anadkat、Red Avila、Igor Babuschkin、Suchir Balaji、Valerie Balcom、Paul Baltescu、Haiming Bao、Mohammad Bavarian、Jeff Belgum、Irwan Bello、Jake Berdine、Gabriel Bernadett-Shapiro、Christopher Berner、Lenny Bogdonoff、Oleg Boiko、Madelaine Boyd、Anna-Luisa Brakman、Greg Brockman、Tim Brooks、Miles Brundage、Kevin Button、Trevor Cai、Rosie Campbell、Andrew Cann、Brittany Carey、Chelsea Carlson、Rory Carmichael、Brooke Chan、Che Chang、Fotis Chantzis、Derek Chen、Sully Chen、Ruby Chen、Jason Chen、Mark Chen、Ben Chess、Chester Cho、Casey Chu、Hyung Won Chung、Dave Cummings、Jeremiah Currier、Yunxing Dai、Cory Decareaux、Thomas Degry、Noah Deutsch、Damien Deville、Arka Dhar、David Dohan、Steve Dowling、Sheila Dunning、Adrien Ecoffet、Atty Eleti、Tyna Eloundou、David Farhi、Liam Fedus、Niko Felix、Simón Posada Fishman、Juston Forte、Isabella Fulford、Leo Gao、Elie Georges、Christian Gibson、Vik Goel、Tarun Gogineni、Gabriel Goh、Rapha Gontijo-Lopes、Jonathan Gordon、Morgan Grafstein、Scott Gray、Ryan Greene、Joshua Gross、Shixiang Shane Gu、Yufei Guo、Chris Hallacy、Jesse Han、Jeff Harris、Yuchen He、Mike Heaton、Johannes Heidecke、Chris Hesse、Alan Hickey、Wade Hickey、Peter Hoeschele、Brandon Houghton、Kenny Hsu、Shengli Hu、Xin Hu、Joost Huizinga、Shantanu Jain、Shawn Jain、Joanne Jang、Angela Jiang、Roger Jiang、Haozhun Jin、Denny Jin、Shino Jomoto、Billie Jonn、Heewoo Jun、Tomer Kaftan、Łukasz Kaiser、Ali Kamali、Ingmar Kanitscheider、Nitish Shirish Keskar、Tabarak Khan、Logan Kilpatrick、Jong Wook Kim、Christina Kim、Yongjik Kim、Jan Hendrik Kirchner、Jamie Kiros、Matt Knight、Daniel Kokotajlo、Łukasz Kondraciuk、Andrew Kondrich、Aris Konstantinidis、Kyle Kosic、Gretchen Krueger、Vishal Kuo、Michael Lampe、Ikai Lan、Teddy Lee、Jan Leike、Jade Leung、Daniel Levy、Chak Ming Li、Rachel Lim、Molly Lin、Stephanie Lin、Mateusz Litwin、Theresa Lopez、Ryan Lowe、Patricia Lue、Anna Makanju、Kim Malfacini、Sam Manning、Todor Markov、Yaniv Markovski、Bianca Martin、Katie Mayer、Andrew Mayne、Bob McGrew、Scott Mayer McKinney、Christine McLeavey、Paul McMillan、Jake McNeil、David Medina、Aalok Mehta、Jacob Menick、Luke Metz、Andrey Mishchenko、Pamela Mishkin、Vinnie Monaco、Evan Morikawa、Daniel Mossing、Tong Mu、Mira Murati、Oleg Murk、David Mély、Ashvin Nair、Reiichiro Nakano、Rajeev Nayak、Arvind Neelakantan、Richard Ngo、Hyeonwoo Noh、Long Ouyang、Cullen O’Keefe、Jakub Pachocki、Alex Paino、Joe Palermo、Ashley Pantuliano、Giambattista Parascandolo、Joel Parish、Emy Parparita、Alex Passos、Mikhail Pavlov、Andrew Peng、Adam Perelman、Filipe de Avila Belbute Peres、Michael Petrov、Henrique Ponde de Oliveira Pinto、Michael Pokorny、Michelle Pokrass、Vitchyr H. Pong、Tolly Powell、Alethea Power、Boris Power、Elizabeth Proehl、Raul Puri、Alec Radford、Jack Rae、Aditya Ramesh、Cameron Raymond、Francis Real、Kendra Rimbach、Carl Ross、Bob Rotsted、Henri Roussez、Nick Ryder、Mario Saltarelli、Ted Sanders、Shibani Santurkar、Girish Sastry、Heather Schmidt、David Schnurr、John Schulman、Daniel Selsam、Kyla Sheppard、Toki Sherbakov、Jessica Shieh、Sarah Shoker、Pranav Shyam、Szymon Sidor、Eric Sigler、Maddie Simens、Jordan Sitkin、Katarina Slama、Ian Sohl、Benjamin Sokolowsky、Yang Song、Natalie Staudacher、Felipe Petroski Such、Natalie Summers、Ilya Sutskever、Jie Tang、Nikolas Tezak、Madeleine B. Thompson、Phil Tillet、Amin Tootoonchian、Elizabeth Tseng、Preston Tuggle、Nick Turley、Jerry Tworek、Juan Felipe Cerón Uribe、Andrea Vallone、Arun Vijayvergiya、Chelsea Voss、Carroll Wainwright、Justin Jay Wang、Alvin Wang、Ben Wang、Jonathan Ward、Jason Wei、CJ Weinmann、Akila Welihinda、Peter Welinder、Jiayi Weng、Lilian Weng、Matt Wiethoff、Dave Willner、Clemens Winter、Samuel Wolrich、Hannah Wong、Lauren Workman、Sherwin Wu、Jeff Wu、Michael Wu、Kai Xiao、Tao Xu、Sarah Yoo、Kevin Yu、Qiming Yuan、Wojciech Zaremba、Rowan Zellers、Chong Zhang、Marvin Zhang、Shengjia Zhao、Tianhao Zheng、Juntang Zhuang、William Zhuk 和 Barret Zoph。《Gpt-4 技术报告》，2024。

+   Raffel 等人（2020）Colin Raffel、Noam Shazeer、Adam Roberts、Katherine Lee、Sharan Narang、Michael Matena、Yanqi Zhou、Wei Li 和 Peter J. Liu。利用统一的文本到文本转换器探索迁移学习的极限。*J. Mach. Learn. Res.*, 21(1)，2020年1月。ISSN 1532-4435。

+   Ravaut 等人（2024）Mathieu Ravaut、Bosheng Ding、Fangkai Jiao、Hailin Chen、Xingxuan Li、Ruochen Zhao、Chengwei Qin、Caiming Xiong 和 Shafiq Joty。大语言模型的污染程度有多高？全面调查与 llmsanitize 库，2024。

+   Reimers 和 Gurevych（2019）Nils Reimers 和 Iryna Gurevych。Sentence-bert：使用 Siamese BERT 网络的句子嵌入。在 *2019年自然语言处理经验方法会议论文集* 中。计算语言学协会，2019年11月。网址：[http://arxiv.org/abs/1908.10084](http://arxiv.org/abs/1908.10084)。

+   Rein 等人（2023）David Rein、Betty Li Hou、Asa Cooper Stickland、Jackson Petty、Richard Yuanzhe Pang、Julien Dirani、Julian Michael 和 Samuel R. Bowman。Gpqa：一个研究生级别的 Google-proof 问答基准，2023。

+   Shen 等人（2023）Chenhui Shen、Liying Cheng、Xuan-Phi Nguyen、Yang You 和 Lidong Bing。大语言模型还不是抽象总结的类人评估者。在 *Findings of the Association for Computational Linguistics: EMNLP 2023* 中，第4215–4233页，2023。

+   Team et al. (2024a) Gemma Team, Morgane Riviere, Shreya Pathak, Pier Giuseppe Sessa, Cassidy Hardin, Surya Bhupatiraju, Léonard Hussenot, Thomas Mesnard, Bobak Shahriari, Alexandre Ramé, Johan Ferret, Peter Liu, Pouya Tafti, Abe Friesen, Michelle Casbon, Sabela Ramos, Ravin Kumar, Charline Le Lan, Sammy Jerome, Anton Tsitsulin, Nino Vieillard, Piotr Stanczyk, Sertan Girgin, Nikola Momchev, Matt Hoffman, Shantanu Thakoor, Jean-Bastien Grill, Behnam Neyshabur, Olivier Bachem, Alanna Walton, Aliaksei Severyn, Alicia Parrish, Aliya Ahmad, Allen Hutchison, Alvin Abdagic, Amanda Carl, Amy Shen, Andy Brock, Andy Coenen, Anthony Laforge, Antonia Paterson, Ben Bastian, Bilal Piot, Bo Wu, Brandon Royal, Charlie Chen, Chintu Kumar, Chris Perry, Chris Welty, Christopher A. Choquette-Choo, Danila Sinopalnikov, David Weinberger, Dimple Vijaykumar, Dominika Rogozińska, Dustin Herbison, Elisa Bandy, Emma Wang, Eric Noland, Erica Moreira, Evan Senter, Evgenii Eltyshev, Francesco Visin, Gabriel Rasskin, Gary Wei, Glenn Cameron, Gus Martins, Hadi Hashemi, Hanna Klimczak-Plucińska, Harleen Batra, Harsh Dhand, Ivan Nardini, Jacinda Mein, Jack Zhou, James Svensson, Jeff Stanway, Jetha Chan, Jin Peng Zhou, Joana Carrasqueira, Joana Iljazi, Jocelyn Becker, Joe Fernandez, Joost van Amersfoort, Josh Gordon, Josh Lipschultz, Josh Newlan, Ju yeong Ji, Kareem Mohamed, Kartikeya Badola, Kat Black, Katie Millican, Keelin McDonell, Kelvin Nguyen, Kiranbir Sodhia, Kish Greene, Lars Lowe Sjoesund, Lauren Usui, Laurent Sifre, Lena Heuermann, Leticia Lago, Lilly McNealus, Livio Baldini Soares, Logan Kilpatrick, Lucas Dixon, Luciano Martins, Machel Reid, Manvinder Singh, Mark Iverson, Martin Görner, Mat Velloso, Mateo Wirth, Matt Davidow, Matt Miller, Matthew Rahtz, Matthew Watson, Meg Risdal, Mehran Kazemi, Michael Moynihan, Ming Zhang, Minsuk Kahng, Minwoo Park, Mofi Rahman, Mohit Khatwani, Natalie Dao, Nenshad Bardoliwalla, Nesh Devanathan, Neta Dumai, Nilay Chauhan, Oscar Wahltinez, Pankil Botarda, Parker Barnes, Paul Barham, Paul Michel, Pengchong Jin, Petko Georgiev, Phil Culliton, Pradeep Kuppala, Ramona Comanescu, Ramona Merhej, Reena Jana, Reza Ardeshir Rokni, Rishabh Agarwal, Ryan Mullins, Samaneh Saadat, Sara Mc Carthy, Sarah Perrin, Sébastien M. R. Arnold, Sebastian Krause, Shengyang Dai, Shruti Garg, Shruti Sheth, Sue Ronstrom, Susan Chan, Timothy Jordan, Ting Yu, Tom Eccles, Tom Hennigan, Tomas Kocisky, Tulsee Doshi, Vihan Jain, Vikas Yadav, Vilobh Meshram, Vishal Dharmadhikari, Warren Barkley, Wei Wei, Wenming Ye, Woohyun Han, Woosuk Kwon, Xiang Xu, Zhe Shen, Zhitao Gong, Zichuan Wei, Victor Cotruta, Phoebe Kirk, Anand Rao, Minh Giang, Ludovic Peran, Tris Warkentin, Eli Collins, Joelle Barral, Zoubin Ghahramani, Raia Hadsell, D. Sculley, Jeanine Banks, Anca Dragan, Slav Petrov, Oriol Vinyals, Jeff Dean, Demis Hassabis, Koray Kavukcuoglu, Clement Farabet, Elena Buchatskaya, Sebastian Borgeaud, Noah Fiedel, Armand Joulin, Kathleen Kenealy, Robert Dadashi, and Alek Andreev. Gemma 2: Improving open language models at a practical size, 2024a. URL [https://arxiv.org/abs/2408.00118](https://arxiv.org/abs/2408.00118).

+   Team et al. (2024b) Reka Team, Aitor Ormazabal, Che Zheng, Cyprien de Masson d’Autume, Dani Yogatama, Deyu Fu, Donovan Ong, Eric Chen, Eugenie Lamprecht, Hai Pham, Isaac Ong, Kaloyan Aleksiev, Lei Li, Matthew Henderson, Max Bain, Mikel Artetxe, Nishant Relan, Piotr Padlewski, Qi Liu, Ren Chen, Samuel Phua, Yazheng Yang, Yi Tay, Yuqi Wang, Zhongkai Zhu, and Zhihui Xie. Reka核心、闪光与边缘：一系列强大的多模态语言模型，2024b。

+   Touvron et al. (2023) Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, Dan Bikel, Lukas Blecher, Cristian Canton Ferrer, Moya Chen, Guillem Cucurull, David Esiobu, Jude Fernandes, Jeremy Fu, Wenyin Fu, Brian Fuller, Cynthia Gao, Vedanuj Goswami, Naman Goyal, Anthony Hartshorn, Saghar Hosseini, Rui Hou, Hakan Inan, Marcin Kardas, Viktor Kerkez, Madian Khabsa, Isabel Kloumann, Artem Korenev, Punit Singh Koura, Marie-Anne Lachaux, Thibaut Lavril, Jenya Lee, Diana Liskovich, Yinghai Lu, Yuning Mao, Xavier Martinet, Todor Mihaylov, Pushkar Mishra, Igor Molybog, Yixin Nie, Andrew Poulton, Jeremy Reizenstein, Rashi Rungta, Kalyan Saladi, Alan Schelten, Ruan Silva, Eric Michael Smith, Ranjan Subramanian, Xiaoqing Ellen Tan, Binh Tang, Ross Taylor, Adina Williams, Jian Xiang Kuan, Puxin Xu, Zheng Yan, Iliyan Zarov, Yuchen Zhang, Angela Fan, Melanie Kambadur, Sharan Narang, Aurelien Rodriguez, Robert Stojnic, Sergey Edunov, and Thomas Scialom. Llama 2：开放基础和微调的聊天模型，2023。

+   Wu et al. (2023a) Ning Wu, Ming Gong, Linjun Shou, Shining Liang, and Daxin Jiang. 大型语言模型在摘要评估中的多样化角色。发表于*CCF国际自然语言处理与中文计算大会*，第695–707页，Springer，2023a。

+   Wu et al. (2023b) Tianyu Wu, Shizhu He, Jingping Liu, Siqi Sun, Kang Liu, Qing-Long Han, and Yang Tang. ChatGPT概述：历史、现状及潜在的未来发展。*IEEE/CAA自动化学报*，10(5)：1122–1136，2023b。doi: 10.1109/JAS.2023.123618。

+   Yu et al. (2024) Zhuohao Yu, Chang Gao, Wenjin Yao, Yidong Wang, Wei Ye, Jindong Wang, Xing Xie, Yue Zhang, and Shikun Zhang. Kieval：一个基于知识的互动评估框架，用于大型语言模型。*arXiv预印本 arXiv:2402.15043*，2024。

+   Zellers et al. (2019) Rowan Zellers, Ari Holtzman, Yonatan Bisk, Ali Farhadi, and Yejin Choi. Hellaswag：机器真的能完成你的句子吗？，2019。网址 [https://arxiv.org/abs/1905.07830](https://arxiv.org/abs/1905.07830)。

+   Zhao et al. (2023) Qinlin Zhao, Jindong Wang, Yixuan Zhang, Yiqiao Jin, Kaijie Zhu, Hao Chen, and Xing Xie. Competeai：理解基于大型语言模型的智能体中的竞争行为，2023。

+   郑等（2023）郑联民、姜伟林、盛颖、庄思远、吴张浩、庄永浩、林子、李卓瀚、李大成、邢睿、张昊、何塞·E·冈萨雷斯、斯托伊卡。通过mt-bench和聊天机器人竞技场评判llm-as-a-judge。在A. Oh、T. Naumann、A. Globerson、K. Saenko、M. Hardt和S. Levine（编者），*神经信息处理系统进展*，第36卷，第46595–46623页。Curran Associates, Inc.，2023年。URL [https://proceedings.neurips.cc/paper_files/paper/2023/file/91f18a1287b398d378ef22505bf41832-Paper-Datasets_and_Benchmarks.pdf](https://proceedings.neurips.cc/paper_files/paper/2023/file/91f18a1287b398d378ef22505bf41832-Paper-Datasets_and_Benchmarks.pdf)。

## 附录 A 使用的提示

在本节中，我们列出了所有使用的提示，包括问题生成、同行对战和考官的提示。

### A.1 提示给考官代理

表5：LLM考官代理的提示组件。

| 领域 | 领域命令 | 领域示例 |
| --- | --- | --- |
| 写作 | 它应该是一个用户查询，要求大语言模型写一些东西。 | 撰写一篇关于最近一次夏威夷旅行的吸引人的博客文章，突出文化体验和必看的景点。 |
| 角色扮演 | 它应该提出一个情境，要求聊天机器人模仿特定的角色/人物。提供所有必要的指示和要求，并请求其回应。然后，发送一个开始请求来完成。 | 假装在接下来的对话中你是埃隆·马斯克。尽量像埃隆·马斯克那样说话。我们为什么需要去火星？ |
| 提取 | 它应该由两部分组成：问题和上下文。问题应该测试聊天机器人是否能够正确理解并从给定上下文中提取信息。自己草拟并提供新的上下文。 | 问题：请根据以下电影评论按1到5的评分标准进行评价，1表示非常差，3表示中立，5表示非常好：上下文：这部于2019年11月18日上映的电影非常棒。摄影、表演、情节——一切都是一流的。我从未像这次那样对一部电影感到失望。情节可预测，角色单一。依我看，这部电影是2022年上映的最差电影。电影还行。有些部分我很喜欢，但也有些部分感觉平淡无奇。这是一部于2018年2月上映的电影，看起来相当普通。请返回一个整数的JSON数组作为答案。 |
| 推理 | 它应该是一个具体的问题，旨在测试LLM的推理能力。 | 假设你正在参加一场比赛，和一群人一起。如果你刚刚超越了第二名选手，你现在的名次是什么？你刚刚超越的那个人在哪里？ |
| 数学 | 它应该是一个具体的问题，旨在测试LLM的数学能力。 | 三角形的顶点分别位于（0，0）、（-1，1）和（3，3）这三个点上。这个三角形的面积是多少？ |
| 编程 | 应该是一个特定的问题，旨在测试LLM的编程技能。 | 开发一个Python程序，读取目录下的所有文本文件，并返回出现次数最多的前五个单词。 |
| STEM知识 | 应该是一个特定的问题，旨在测试LLM的STEM知识。 | 在量子物理领域，什么是叠加态，它如何与量子纠缠现象相关联？ |
| 人文/社会科学知识 | 应该是一个特定的问题，旨在测试LLM的人文/社会科学知识。 | 提供有关经济指标（如GDP、通货膨胀和失业率）之间关系的见解。解释财政和货币政策如何影响这些指标。 |

这是给考试代理的提示，用于生成问题。领域及其相关命令在[5](https://arxiv.org/html/2405.20267v4#A1.T5 "表5 ‣ A.1 给考试代理的提示 ‣ 附录A 使用的提示 ‣ 自动竞技场：通过代理对战和委员会讨论自动化LLM评估")中列出。

你已被分配任务，编写一组关于[领域]的[数字]个不同用户查询。请严格遵循以下6条规则：1. 该问题可能是用户在现实生活中提出的。请遵循示例查询的格式。[领域_命令] 2. 它可以由聊天机器人本身回答，无需额外输入。 3. 你需要尽可能多样化地生成查询。 4. 不要添加除查询本身以外的任何其他词语。 5. 问题应该复杂且困难，需要对主题有深入的理解和分析。每个问题独占一行，在每个问题前加上括号内的序号（例如：“（1）”，“（2）”）。示例查询：[领域_示例]

### A.2 给对战候选者的提示

表6：辩手代理的行动指南。

| 行动 | 行动指南 |
| --- | --- |
| <回应> | 行动指南：只包括<回应>。如有需要，使用<思考>。将整个回答控制在300字以内，包括<思考>。将每个动作包裹在其相应的标签中！ |
| <批评>, <提出> | 行动指南：包括<批评>和<提出>。如有需要，使用<思考>。将整个回答控制在300字以内，包括<思考>。将每个动作包裹在其相应的标签中！ |
| <回应>, <批评>, <提出> | 行动指南：包括<回应>、<批评>和<提出>。如有需要，使用<思考>。将整个回答控制在600字以内，包括<思考>。将每个动作包裹在其相应的标签中！ |

这是同行战斗候选人的第一个提示。当可能时，它作为系统提示包含在内。行动指南提示包含在表格[6](https://arxiv.org/html/2405.20267v4#A1.T6 "表格 6 ‣ A.2 给予同行战斗候选人的提示 ‣ 附录 A 使用的提示 ‣ 自动竞技场：通过代理同行战斗和委员会讨论自动化 LLM 评估")中，其中的行动由回合和轮次决定，如图[2](https://arxiv.org/html/2405.20267v4#S2.F2 "图 2 ‣ 2.1 问题生成 ‣ 2 自动竞技场框架 ‣ 自动竞技场：通过代理同行战斗和委员会讨论自动化 LLM 评估")所示。

你是一个有用的助手，提供准确的回答来满足用户需求。作为一个经验丰富的助手，你会根据用户的要求提供可靠的回答，并尽可能阐明回答的理由，帮助用户理解。在保持回应中重要细节的同时，你力求输出简洁直接的回答，避免过多冗长的表述。

这是一个竞争性的聊天机器人竞技场。你将与另一个聊天助手进行辩论，并由一个委员会根据帮助性、相关性、准确性、深度和创造力等因素进行评分。在回答初始用户输入后，你将与对手进行多轮辩论。以下是你的行动：

<think>：逐步思考，分析问题或在辩论中规划策略。此部分对对手隐藏。仅在必要时进行思考，并保持简洁。

<respond>：尽可能准确地回答用户输入。

<criticize>：批评对手回答的弱点。

<raise>：针对对手的弱点进行攻击。给出对手可能无法回应的潜在跟进用户输入。输入应简洁回答，并关注其之前回答的变体或动机。只生成一个输入。要合理，避免过于具体或重复。如果你没有看到对手的回答，请不要提出后续问题！

严格遵循行动指南。

[ACTION_GUIDE_PROMPT]

初始用户输入：[QUESTION]

在代理回应后，使用以下提示反馈对手的回应：

[ACTION_GUIDE_PROMPT] 对手的回应：[OPPONENT_RESPONSE]

对于字数限制，<respond>行动的字数为300字。<criticize>和<raise>行动的总字数为300字。包括这三项行动将使字数加倍。对于需要较长回应的写作型问题（写作、角色扮演、编程、人文学科/社会科学知识），300字的限制增加至400字。总的来说，候选人A和B的生成字数和行动数相同，以确保公平性。由于LLM的分词器不同，我们通过使用tiktoken包来规范所有长度，每个词大约等于$4/3$个token。字数限制是在经过仔细的长度研究后选择的。

### A.3 裁判提示

这是裁判用来得出初步评估和裁决的提示：

这是一个聊天机器人领域。两位AI助手就谁更有帮助进行了多轮辩论。请作为公正的裁判，评估这两位AI助手的能力。你应选择执行指令并更好回答问题的助手。你的评估应考虑如有用性、相关性和准确性等因素。通过比较两位助手的回应开始评估，并提供简短的解释。避免任何立场偏见，确保回答的呈现顺序不影响你的判断。请勿让回答的长度影响你的评估，选择简洁直截了当的回答，而不是冗长的回应。当两位候选人的表现相同，选择更简短的答案。不要偏向某些助手的名称。尽量客观。简洁地提供解释，并在200字内完成。最终评判应严格遵循以下格式：“[[A]]”如果助手A更好，“[[B]]”如果助手B更好，“[[Tie]]”如果平局。请在300字内完成判断。

这是供裁判讨论的提示：

以下是委员会中其他裁判的回应。请阅读并决定是否调整评分或维持原有判断。提供解释后，严格按照以下格式输出最终裁决：“[[A]]”如果助手A更好，“[[B]]”如果助手B更好，“[[Tie]]”如果平局。请在300字内完成判断。

## 附录B 示例问题生成

为展示问题生成的整体质量，我们在这里列出每个类别的2个生成问题。所示问题不是手动选择的，而仅仅是第一个生成的问题。质量保持一致。我们手动检查了带有封闭式答案（数学、推理、编程）的问题，发现所有问题都是可解的。

写作：

1\. 为一家专注于可持续时尚的初创公司制定详细的营销策略，包括社交媒体活动和与影响者的合作伙伴关系。

2\. 编写一份关于社交媒体对青少年心理影响的综合指南，结合最新的研究和专家意见。

角色扮演：

1\. 假设你是19世纪的英国侦探。你将如何利用当时的技术和方法来解决伦敦的一起神秘失踪案件？

2\. 假装你是米其林星级大厨。请详细描述你如何准备一道体现现代法国料理精髓的招牌菜。

提取：

1\. 提到的三个最重要的历史事件及其日期是什么？

背景：

文章讨论了几个历史上的关键时刻，包括1215年《大宪章》的签署，它为现代民主奠定了基础。它还提到1989年柏林墙的倒塌，这是冷战结束的一个关键时刻。另一项重要事件是1969年7月20日的登月，展示了太空探索的重大进展。

2\. 确定每种草药疗法的主要治疗益处和所提到的活性成分。

背景：

这段文本概述了几种使用了几个世纪的草药疗法。它提到洋甘菊含有倍半萜醇，具有抗炎和镇静作用。银杏叶以其类黄酮和萜类化合物著称，能增强认知功能和血液循环。最后，紫锥花因其生物碱类成分而受到认可，能够增强免疫系统。

推理：

1\. 如果一个立方体的体积增加了三倍，它的边长增加了多少倍？

2\. 在一场两回合的足球比赛中，A队在主场以3-0获胜，但在客场以2-5失利。根据客场进球规则，哪支队伍晋级到下一轮？

数学：

1\. 如何解这个微分方程 $dy/dx+2y=e^{(-2x)}$，已知 $y(0)=1$？

2\. 积分 $\int \frac{x^{2}+2x+2}{x^{3}+3x^{2}+3x+1}dx$ 的值是多少？

编程：

1\. 如何在C++中实现一个函数，动态分配一个二维数组，基于用户输入的大小，初始化所有元素为零，然后正确地释放内存以避免内存泄漏？

2\. 编写一个JavaScript函数，从给定URL获取数据，解析JSON响应，并筛选结果返回一个数组，其中某个特定键的值符合某个条件。

STEM 知识：

1\. 如何计算黑洞的史瓦西半径，这对广义相对论中的事件视界概念有什么影响？

2\. 你能解释一下真核基因表达中的剪接过程及其在蛋白质组多样性中的重要性吗？

人文学科/社会科学知识：

1\. 讨论殖民遗产对非洲国家当代政治结构的影响，并举例说明。

2\. 分析中国独生子女政策的社会和经济后果。

## 附录 C 污染分析

表格 7：基准的平均污染百分比。

| 检测方法 | 我们的方法 | MMLU | ARC 挑战 | HellaSwag |
| --- | --- | --- | --- | --- |
| GPT-4 风格（子串匹配） $\downarrow$ | 2% | 42% | 33% | 18% |
| Playtus 风格（句子相似度） $\downarrow$ | 28% | 41% | 35% | 43% |

问题生成和同行辩论过程中采用的设计确保了污染最小化。数据污染指的是测试实例出现在预训练或监督微调数据中的可能性。

#### 问题生成：

由于我们自动生成问题，因此可以减少测试实例最终暴露于开放网络的风险，这种情况可能发生在静态数据集中。数据污染的缓解通常被认为是这种动态且频繁更新的评估框架的优势之一（Li等，[2023b](https://arxiv.org/html/2405.20267v4#bib.bib32)）。

#### 同行辩论：

同行辩论确保我们评估整个辩论，而不是简单的问答，这进一步减少了污染。在辩论过程中，模型会在全面且深入的能力上进行评估，例如规划策略、指出对手的缺陷以及草拟进一步的问题。这样的互动评估框架已被证明能够减少污染（Yu等， [2024](https://arxiv.org/html/2405.20267v4#bib.bib53)；Bai等， [2024](https://arxiv.org/html/2405.20267v4#bib.bib5)）。

除了设计选择外，我们还进行污染分析，比较Auto-Arena辩论问题和流行基准测试中的测试问题的污染百分比。具体来说，我们使用两种类型的污染检测度量：

1.  1.

    字符串匹配度量，如GPT-4中所述（OpenAI等， [2024](https://arxiv.org/html/2405.20267v4#bib.bib42)），其中如果评估数据点的三个50字符随机抽取子串（或整个字符串，如果它短于此）中的任何一个是训练集的子串，则认为匹配。如果是这样，我们将该点标记为污染。

1.  2.

    句子嵌入相似度度量，如Platypus中所述（Lee等， [2024](https://arxiv.org/html/2405.20267v4#bib.bib29)），其中如果一个问题与任何训练项的余弦相似度（使用Sentence Transformer（Reimers & Gurevych， [2019](https://arxiv.org/html/2405.20267v4#bib.bib45)）嵌入）大于80%，则认为该问题被污染。该检测方法对于重新表述的情况更具鲁棒性，从而确保我们可以检测到LLMs仅仅是在重新表述互联网上现有问题的情况。

尽管我们无法访问训练数据，但LLMs大多使用公共网络数据进行预训练（Raffel等， [2020](https://arxiv.org/html/2405.20267v4#bib.bib43)；Brown等， [2020](https://arxiv.org/html/2405.20267v4#bib.bib9)；Touvron等， [2023](https://arxiv.org/html/2405.20267v4#bib.bib50)）。因此，我们通过Bing搜索API进行近似：如果原文测试示例出现在网上，这很可能表明其已包含或暴露于训练数据中。Li等（[2024](https://arxiv.org/html/2405.20267v4#bib.bib33)）也采用此程序来检测污染。

削减实验如下进行：首先，我们随机从测试集中抽取100个问题。作为基准，我们使用3个流行的评估基准：MMLU (Hendrycks et al., [2021a](https://arxiv.org/html/2405.20267v4#bib.bib25))，ARC挑战 (Clark et al., [2018](https://arxiv.org/html/2405.20267v4#bib.bib14))，和HellaSwag (Zellers et al., [2019](https://arxiv.org/html/2405.20267v4#bib.bib54))。对于每个问题，我们通过Bing搜索API获取前10条搜索结果。如果问题被上述检测方法判定为与任何一个搜索结果存在污染，它将被标记为污染。

污染的测试实例百分比在表格[7](https://arxiv.org/html/2405.20267v4#A3.T7 "Table 7 ‣ Appendix C Contamination Analysis ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")中报告。我们可以观察到，通过生成新问题，Auto-Arena确实缓解了污染问题。与静态数据集相比，Auto-Arena的污染百分比（2%）根据精确匹配显著较低。当使用句子相似度度量时，我们可以有效地检测生成的问题是否仅仅是现有问题的改写。与其他基准相比，这一百分比大幅减少了7%至15%。

## 附录D 合成问题与真实问题

在这一部分中，我们尝试展示Auto-Arena生成的合成问题与真实问题的可泛化性。

#### 设计：

生成的问题在设计上类似于真实世界的查询。在问题生成提示中，我们特别要求考察者设计“用户在现实生活中可能会提问的问题”。从附录[B](https://arxiv.org/html/2405.20267v4#A2 "Appendix B Example Questions Generated ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")中，我们也可以观察到合成问题与真实问题的相似性。

表格 8：对合成问题和真实问题的人类评估。

|  | 志愿者 1 | 志愿者 2 |
| --- | --- | --- |
| 正确 | 27.1% | 38.9% |
| 错误 | 27.1% | 11.9% |
| 无法判断 | 45.8% | 49.2% |
| 一致性 | -0.11 |

#### 人类研究：

为了展示生成的查询与现实生活中的问题相似，我们进行了以下人类研究。我们比较了由Auto-Arena生成的30个合成问题与30个真实问题。我们要求一名人工用户随机查看一个问题，并判断他/她认为该问题是由AI生成的、是真实生活中的，还是他/她无法判断。问题收集自数学类别，其中30个真实问题来自MT-Bench（10个问题，由专家起草）、AMC-8（4个问题，来自2024年数学竞赛）和AGI-Eval（16个数学问题，收集自高考题）。两名经常使用LLM并熟悉AIGC的志愿者参与了研究。我们在表[8](https://arxiv.org/html/2405.20267v4#A4.T8 "Table 8 ‣ Design: ‣ Appendix D Synthetic V.S. Real-Life Questions ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")中报告了他们各自的结果和一致性。我们可以观察到，几乎一半的时间里，人类无法分辨问题是否为合成问题。用户准确率（正确百分比）也较低。我们计算了两位用户之间的Cohen's Kappa一致性，结果为-0.11。该一致性得分表明，用户之间的意见一致性低于随机概率。人类标注者回应的巨大差异也表明了判断中的主观性和不确定性。因此，我们得出结论，人类最有可能无法判断问题是合成的还是现实世界的，这表明二者之间的差异较小。

表 9：关于合成问题和真实问题的消融实验结果。

| 问题 | GPT-4 胜率 | Claude-3 胜率 |
| --- | --- | --- |
| 合成问题 | 80.00% | 20.00% |
| 真实问题 | 75.86% | 24.14% |

#### 消融研究：

为了验证结果在真实世界数据集上的普适性，我们进行了一项消融研究，比较Auto-Arena在真实问题和合成问题上的评估表现。具体来说，我们要求两位候选模型（GPT-4-Turbo-0409和Claude-3-Haiku）围绕30个合成数学问题和30个真实数学问题进行辩论（这些问题的收集方式与表[8](https://arxiv.org/html/2405.20267v4#A4.T8 "Table 8 ‣ Design: ‣ Appendix D Synthetic V.S. Real-Life Questions ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")中展示的人类研究相同）。如果结果具有普适性，我们应该观察到每个模型的胜率相似。结果显示在表[9](https://arxiv.org/html/2405.20267v4#A4.T9 "Table 9 ‣ Human Study: ‣ Appendix D Synthetic V.S. Real-Life Questions ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")中。从结果中，我们可以观察到，在合成和真实数据集上，每个模型的胜率仅相差4%，这表明评估表现一致，验证了合成问题的使用。

除了支持性研究外，使用合成问题进行评估已经被确立为常见做法。数学数据集（Hendrycks et al., [2021b](https://arxiv.org/html/2405.20267v4#bib.bib26)）已经使用了合成生成的数学问题，他们指出了许多优势，例如提供更多示例的便捷性、对难度等级的精确控制，以及测试泛化的便利性（因为可以精确地在不同问题类型中变化不同的难度维度）。LMExamQA（Bai et al., [2024](https://arxiv.org/html/2405.20267v4#bib.bib5)）也使用LLM生成不同领域的问题。KI-Eval（Yu et al., [2024](https://arxiv.org/html/2405.20267v4#bib.bib53)）要求一个由LM驱动的交互者生成问题。类似的研究还有很多。使用合成问题已经成为NLP评估中的常规标准。此外，Auto-Arena中的广泛实验显示与人工结果高度相关，这也证明了与实际使用的对齐。

## 附录E 关于问题生成阶段自我增强偏差的消融研究

表10：问题生成器自我增强偏差的消融结果。

| 问题 | GPT-4 胜率 | Haiku 胜率 |
| --- | --- | --- |
| GPT-4 生成的问题 | 80.00% | 20.00% |
| Haiku 生成的问题 | 76.92% | 23.08% |

我们尝试通过明确设计来减少问题生成阶段的自我增强偏差：首先，在问题生成过程中，我们不向考官透露它将参与这个比赛，也不要求考官只生成自己能解决的问题。其次，同伴辩论过程进一步减少了初始问题生成的偏差：辩论确保候选人不仅仅通过对初始问题的回答来评估，而是更全面和深入地考察其能力，如制定策略、批评对手以及起草问题。换句话说，回答初始问题并不一定能赢得整个辩论。在图[2](https://arxiv.org/html/2405.20267v4#S2.F2 "图 2 ‣ 2.1 问题生成 ‣ 2 Auto-Arena框架 ‣ Auto-Arena：通过代理同伴对抗和委员会讨论自动化LLM评估")中的辩论设计中，候选人还拥有“举手”动作，他们可以向对手提问。这个过程本质上将问题生成过程去中心化。

为了系统地检验是否存在自我增强偏差，我们进行了消融研究：以GPT-4（GPT-4-turbo）和Haiku（Claude-3-Haiku）这两个模型为例，检验增强偏差。首先，我们分别让GPT-4和Haiku生成30个数学问题。然后，我们在两组问题上进行两位候选人（GPT-4和Haiku）之间的同伴辩论，并使用最佳的5-LLM委员会评估结果，如同在主要实验中一样。

我们从评估结果中评估性能差异：如果自增强偏差较低，所获得的排名应保持不变。换句话说，较弱的模型将始终失败，即使是在由它自己生成的问题上。

消融实验结果如表[10](https://arxiv.org/html/2405.20267v4#A5.T10 "Table 10 ‣ Appendix E Ablation study on Self-Enhancement Bias of the Question Generation Stage ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")所示。从结果可以观察到，在两组生成的问题中，GPT-4的胜率始终显著高于Claude-3-Haiku的胜率。即使存在一定程度的自增强偏差，结果的差异仍然足够显著，能够达到正确的排名。

## 附录F 评审员间一致性

![参见说明文字](img/cba84624c3aeda30308bba05d07b24c6.png)

图10：委员会讨论前的Cohen's Kappa一致性与多数投票。

![参见说明文字](img/a3aeb7879d1c6b57b6882ff2b342e965.png)

图11：委员会讨论后1轮的Cohen's Kappa一致性与多数投票。

如图[11](https://arxiv.org/html/2405.20267v4#A6.F11 "Figure 11 ‣ Appendix F Inter-judge Agreement ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")所示，在委员会讨论之前，评审员之间的Cohen's Kappa一致性（McHugh, [2012](https://arxiv.org/html/2405.20267v4#bib.bib37)）非常低，平均值为0.16，这表明只有轻微的一致性。我们注意到，弱模型评审员和强模型评审员之间的协议尤其低，例如GPT-4和Yi。这表明，通用模型能力可能导致作为评审员时出现显著的性能差距。

经过1轮沟通后，由于评审员被更有说服力的论点所信服，一致性显著提高。讨论后的平均Cohen's Kappa值达到了0.27，表示较为公平的一致性。

## 附录G 主实验的模型选择

表11：主实验的模型选择。“最新”和“最强”指的是实验时的状态（2024年4月）。加粗的模型是选择用于主要实验的7个模型。未加粗的模型是在扩展阶段加入的。

| 模型名称 | 纳入原因 | 授权 |
| --- | --- | --- |
| GPT-4-0409-Turbo (OpenAI et al., [2024](https://arxiv.org/html/2405.20267v4#bib.bib42)) | GPT-4系列中最新且最强的模型 | 专有 |
| GPT-4o-2024-05-13 (Openai, [2024b](https://arxiv.org/html/2405.20267v4#bib.bib41)) | GPT模型家族中新发布的模型 | 专有 |
| GPT-3.5-Turbo-0125 (Openai, [2024a](https://arxiv.org/html/2405.20267v4#bib.bib40)) | GPT模型家族中最新的ChatGPT版本 | 专有 |
| Claude-3.5-Sonnet-20240620 (Anthropic, [2024](https://arxiv.org/html/2405.20267v4#bib.bib2)) | Claude-3.5系列中最新的模型 | 专有 |
| Claude-3-Haiku (Anthropic, [2024](https://arxiv.org/html/2405.20267v4#bib.bib2)) | Claude-3下Claude模型家族中最新且最便宜的模型 | 专有 |
| Qwen/Qwen2-72B-Instruct (Bai et al., [2023](https://arxiv.org/html/2405.20267v4#bib.bib3)) | Qwen-2下Qwen模型家族的代表 | 专有 |
| Qwen1.5-72B-chat (Bai et al., [2023](https://arxiv.org/html/2405.20267v4#bib.bib3)) | Qwen-1.5下Qwen模型家族的代表 | Qianwen LICENSE |
| Command R Plus (Cohere, [2024](https://arxiv.org/html/2405.20267v4#bib.bib17)) | Command R 模型家族中最强的模型 | CC-BY-NC-4.0 |
| Llama-3-70b-chat-hf (Meta, [2024](https://arxiv.org/html/2405.20267v4#bib.bib38)) | Llama-3下Llama模型家族的代表 | Llama 3 Community |
| Llama-2-70b-chat (Touvron et al., [2023](https://arxiv.org/html/2405.20267v4#bib.bib50)) | Llama-2下Llama模型家族的代表 | Llama 2 Community |
| Mixtral-8x7b-Instruct-v0.1 (Jiang et al., [2024](https://arxiv.org/html/2405.20267v4#bib.bib27)) | 开源Mistral小型模型中最强的 | Apache 2.0 |
|  | MOE 结构 |  |
| Gemma-2-27b-it (Team et al., [2024a](https://arxiv.org/html/2405.20267v4#bib.bib48)) | Gemma家族的代表 | Apache 2.0 |
| Gemini-1.5-flash-exp-0827 (Team et al., [2024a](https://arxiv.org/html/2405.20267v4#bib.bib48)) | Gemini-1.5 家族中最便宜的模型 | 专有 |
| Yi-34B-Chat (AI et al., [2024](https://arxiv.org/html/2405.20267v4#bib.bib1)) | Yi 模型家族中在 Chatbot Arena 上最强的模型 | Yi 许可 |
| Deepseek-LLM-67B-chat (DeepSeek-AI et al., [2024](https://arxiv.org/html/2405.20267v4#bib.bib18)) | Deepseek 家族中的代表性开源模型 | DeepSeek 许可 |

在表格[11](https://arxiv.org/html/2405.20267v4#A7.T11 "Table 11 ‣ Appendix G Model selection for the main experiment ‣ Auto-Arena:Automating LLM Evaluations with Agent Peer Battles and Committee Discussions")中，我们展示了为主要实验和扩展选择的所有模型，并说明了选择的理由。总体而言，我们尽力从Chatbot Arena的前20名模型中选择代表性强的著名模型。尽管Chatbot Arena排名大多由不同版本的模型组成，但我们仅从每个模型家族中选择最强或最新的模型。除了Chatbot Arena上的模型，我们还包括了4个未充分评估的著名中文模型，以调查它们的表现。

## 附录H 基准方法与Auto-Arena的比较

表格12：Auto-Arena与其他基准的比较。

| 方法 | 手动构建 | 新鲜度 | 评估成本 | 评判标准 |
| --- | --- | --- | --- | --- |
|  | 查询 |  | 每个模型 |
| OpenLLM Leaderboard (Beeching et al., [2023](https://arxiv.org/html/2405.20267v4#bib.bib6)) | 是 | 静态 | - | 回答准确度 |
| MMLU (Hendrycks et al., [2021a](https://arxiv.org/html/2405.20267v4#bib.bib25)) | 是 | 静态 | - | 回答准确度 |
| GPQA (Rein 等，[2023](https://arxiv.org/html/2405.20267v4#bib.bib46)) | 是 | 静态 | - | 答案准确性 |
| LC-AlpacaEval (Dubois 等，[2024a](https://arxiv.org/html/2405.20267v4#bib.bib20)) | 是 | 静态 | $10 | 单一 LLM（GPT-4） |
| MT-Bench (Zheng 等，[2023](https://arxiv.org/html/2405.20267v4#bib.bib56)) | 是 | 静态 | $10 | 单一 LLM（GPT-4） |
| Arena Hard (Li* 等，[2024](https://arxiv.org/html/2405.20267v4#bib.bib31)) | 是 | 频繁更新 | $25 | 单一 LLM（GPT-4） |
| Chatbot Arena (Zheng 等，[2023](https://arxiv.org/html/2405.20267v4#bib.bib56)) | 是 | 实时 | 非常高 | 人类 |
| Auto-Arena | 否 | 新生成的 | $5 | LLM 委员会 |

表格 [12](https://arxiv.org/html/2405.20267v4#A8.T12 "表格 12 ‣ 附录 H 基准方法与 Auto-Arena 的比较 ‣ Auto-Arena：通过代理对抗战与委员会讨论自动化大语言模型评估") 显示了基准评估方法与 Auto-Arena 之间的比较。与以往的方法相比，Auto-Arena 的主要优点是完全无需人工构建数据集或干预，以及查询的新颖性。与之前基于模型的系统性基准测试程序相比，另一个创新是使用一个由大语言模型（LLM）组成的委员会来讨论并投票选出最终的获胜者，这引入了多样的观点。Auto-Arena 最重要的创新是同行对抗机制，它要求 LLM 代理相互竞争并辩论。由此产生的多轮辩论评估变得更加深入、互动和全面。

关于评估成本，Auto-Arena 的成本与其他基准相当：我们注意到，9个模型中的主要实验大约花费 45 美元。因此，估计的成本为每个模型 5 美元。随着排名中的模型增加，进行辩论的成本应该按对数尺度缓慢增长，这是因为在为一个包含 $(n-1)$ 个模型的排名中新增一个模型时，进行 $nlog_{2}(n)$ 次配对的结果。然而，由于我们始终使用一个由 5 个 LLM 组成的委员会，评估成本将保持不变。
