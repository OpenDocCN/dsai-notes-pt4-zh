<!--yml

分类：未分类

日期：2025-01-11 12:16:49

-->

# 一种不完全信任LLM的狼人游戏代理实现

> 来源：[https://arxiv.org/html/2409.01575/](https://arxiv.org/html/2409.01575/)

佐藤武宏^† 大崎慎太郎^‡ 横山大作^†

^†明治大学 ^‡奈良科技大学

{ce245022,dyokoyama}@meiji.ac.jp

ozaki.shintaro.ou6@naist.ac.jp

###### 摘要

狼人游戏是一种不完全信息游戏，在创建计算机代理作为玩家时，由于缺乏对情境和发言个性的理解（例如，计算机代理无法进行富有个性的发言或情境性的撒谎），存在一些挑战。我们提出了一种狼人代理，通过结合大语言模型（LLM）和基于规则的算法，解决了其中的一些困难。特别地，我们的代理使用基于规则的算法，根据使用LLM分析对话历史的结果，选择LLM的输出或预先准备的模板输出。这使得代理在特定情况下能够反驳、判断何时结束对话，并具备个性化行为。该方法有效减轻了对话不一致性，并促进了逻辑发言的生成。我们还进行了定性评估，结果显示，与未修改的LLM相比，我们的代理被认为更加人性化。该代理可自由获取，旨在推动狼人游戏领域的研究¹¹1[https://github.com/meiji-yokoyama-lab/AIWolfDial2024](https://github.com/meiji-yokoyama-lab/AIWolfDial2024)。

不完全信任LLM的狼人代理实现

## 1 引言

狼人游戏（Ri et al.，2022）是一个流行的不完全信息多人游戏，玩家被分为两方，村民和狼人，他们隐藏自己的角色，并通过自然语言对话试图在其他玩家之间达成有利的共识。玩狼人游戏需要高水平的智力技能，如推理、合作和撒谎。对于计算机来说，既在游戏信息学方面，又在自然语言处理方面，玩这个游戏都具有挑战性，且多年来已被广泛研究（Kano et al.，2023）。

![参见说明](img/57b03d971fde35f99b8afffa6a30099e.png)

图1：使用LLM玩狼人游戏时出现的问题示例。人类可以自然地说出逻辑谎言，但LLM只能否认它。

该游戏至少包含以下三个主要挑战：

1.  1.

    当前情况仅显示在玩家的对话中。游戏系统展示的信息非常有限，比如谁还活着。存在其他必要的信息来合理地玩游戏，但这些信息是从对话历史中推断出来的：谁怀疑谁，谁已经决定做某事，谁可能会改变主意，等等。

1.  2.

    玩家应该进行战术性对话，以追求特定的目标。例如，当玩家受到怀疑时，玩家应该做出合理的反驳，而不仅仅是坚持自己的观点，如图([1](https://arxiv.org/html/2409.01575v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ An Implementation of Werewolf Agent That does not Truly Trust LLMs"))所示。此外，玩家需要知道何时结束对话以获得自己的优势，尤其是在其他所有玩家似乎都在怀疑对方时。

1.  3.

    玩家应该拥有有吸引力的个性。虽然游戏中不要求必须获胜，但让游戏变得有趣是非常重要的，这涉及多个方面，如语言风格、智能决策和角色扮演 Callison-Burch 等人 ([2022](https://arxiv.org/html/2409.01575v1#bib.bib4))。

许多大型语言模型（LLMs）如 OpenAI ([2022](https://arxiv.org/html/2409.01575v1#bib.bib14))；Anil 等人 ([2023](https://arxiv.org/html/2409.01575v1#bib.bib2))；Achiam 等人 ([2023](https://arxiv.org/html/2409.01575v1#bib.bib1))；Touvron 等人 ([2023a](https://arxiv.org/html/2409.01575v1#bib.bib18))；Meta ([2023](https://arxiv.org/html/2409.01575v1#bib.bib9), [2024](https://arxiv.org/html/2409.01575v1#bib.bib10))；Google ([2024](https://arxiv.org/html/2409.01575v1#bib.bib6))；Team 等人 ([2023](https://arxiv.org/html/2409.01575v1#bib.bib17))；OpenAI ([2023](https://arxiv.org/html/2409.01575v1#bib.bib15))；Touvron 等人 ([2023a](https://arxiv.org/html/2409.01575v1#bib.bib18), [b](https://arxiv.org/html/2409.01575v1#bib.bib19))，具有非常高的泛化能力，已经发布，当然也有几个模型已经应用于狼人代理（Xu 等人， [2023](https://arxiv.org/html/2409.01575v1#bib.bib24)；Wu 等人，[2024](https://arxiv.org/html/2409.01575v1#bib.bib23)）。然而，单纯利用 LLMs 并不能解决在实现狼人代理时遇到的难题。在开发狼人代理的多个挑战中，我们在这项工作中重点关注以下几个方面：1）在某些关键情况下，代理应该进行反驳；2）当讨论被认为毫无意义时，代理应该结束对话；3）代理应该在保持一致个性的前提下，展现出可区分的语言风格，以使游戏变得有趣。

我们的方法总结如下。

#### 基于规则的算法与大型语言模型（LLMs）

我们将 LLM 与基于规则的算法相结合。LLM 检索游戏中的对话历史并生成输出。基于规则的算法根据游戏情境判断该输出是否合适。如果对话不合适，基于规则的算法将使用预定义的模板发言。结果是，基于规则的算法可以在关键情况下撒谎，并在不再需要继续对话时终止对话。

#### 提取游戏信息

为了从对话历史中理解当前状况，我们还利用额外的LLM来提取与游戏相关的信息。我们选择了几个基本但关键的游戏概念，如投票决策和占卜结果。LLM检查对话历史，并以固定格式生成包含这些信息的对话。该信息还被基于规则的算法用来做出决策。

#### 风格转换

我们决定使用一个从大量通用文档中预训练的LLM。此外，我们使用提示词来控制它们，而无需修改或微调模型，并通过提示词赋予代理可区分的个性。

我们的初步实现解决了这些任务。这种方法使我们的模型能够减少对话中的不一致性，并促进了逻辑性的表达。此外，我们还通过进行定性评估对代理进行了评估。结果显示，与未修改的LLM相比，结合基于规则的方法让代理看起来像是理解了对话，而插入个性使其能够进行更自然的对话。源代码公开，以期望未来关于狼人代理的研究能够不断发展。

![参见标题](img/e7d8451dd01f99797eda82de28736659.png)

图2：五人狼人角色列表。

![参见标题](img/027fc586b1b0f96688cf8a70893eb07a.png)

图3：系统概览。我们的系统包含三个模块：话语生成、对话分析和基于规则的算法。我们在第([4.1](https://arxiv.org/html/2409.01575v1#S4.SS1 "4.1 Utterance Generation ‣ 4 System Design ‣ An Implementation of Werewolf Agent That does not Truly Trust LLMs"))节描述了话语生成，在第([4.3](https://arxiv.org/html/2409.01575v1#S4.SS3 "4.3 Talk Analysis ‣ 4 System Design ‣ An Implementation of Werewolf Agent That does not Truly Trust LLMs"))节描述了对话分析，在第([4.4](https://arxiv.org/html/2409.01575v1#S4.SS4 "4.4 Rule-based Algorithm ‣ 4 System Design ‣ An Implementation of Werewolf Agent That does not Truly Trust LLMs"))节描述了基于规则的算法，在附录([A.1](https://arxiv.org/html/2409.01575v1#A1.SS1 "A.1 Required Game Status ‣ Appendix A Appendix ‣ An Implementation of Werewolf Agent That does not Truly Trust LLMs"))中描述了所需的游戏状态。

## 2 相关工作

对狼人游戏的研究有着悠久的历史，可以追溯到对类似狼人游戏的黑手党游戏的研究，数学分析了Braverman等人的工作（[2008](https://arxiv.org/html/2409.01575v1#bib.bib3)）；Migdał（[2013](https://arxiv.org/html/2409.01575v1#bib.bib11)）。一些研究分析了狼人游戏的日志，如Nagayama等人（[2019](https://arxiv.org/html/2409.01575v1#bib.bib12)）；Fukui等人（[2017](https://arxiv.org/html/2409.01575v1#bib.bib5)），或讨论了使狼人代理更强大的方法，如Nakamura等人（[2016](https://arxiv.org/html/2409.01575v1#bib.bib13)）；Wang和Kaneko（[2018](https://arxiv.org/html/2409.01575v1#bib.bib21)）。最近，随着大语言模型（LLMs）的发展，这些模型已经被探索用于狼人代理，如Xu等人（[2023](https://arxiv.org/html/2409.01575v1#bib.bib24)）；Wu等人（[2024](https://arxiv.org/html/2409.01575v1#bib.bib23)）。然而，这些基于LLM的代理在处理狼人特有的特性（如怀疑、撒谎和识别谎言）方面存在困难。此外，这些模型的输出没有角色设定。尽管仅使用LLM的方法占主导地位，但在其他领域，结合规则基础方法与LLM的混合方法正在引起越来越多的关注。在数据分析或商业中，从结构化数据中提取信息的一种常用方法是结合LLM和规则基础方法，如Huang（[2024](https://arxiv.org/html/2409.01575v1#bib.bib7)）；Vertsel和Rumiantsau（[2024](https://arxiv.org/html/2409.01575v1#bib.bib20)）。我们的目标是将这一方法论应用于狼人代理，利用两者的优势。这种混合方法可能会带来更强大和更具适应性的狼人代理。

## 3 五人狼人游戏

我们为狼人游戏选择了一个由五个玩家参与的简单设置。在这个游戏设定中，包含村民、预言家、附身者和狼人角色。就每个角色而言，“村民”没有特殊能力，“预言家”每晚可以通过占卜知道一名玩家的身份，“附身者”没有特殊能力，并且被占卜结果判断为人类。然而，附身者的行为是为了帮助狼人获胜。“狼人”每晚可以选择一名玩家进行攻击并将其淘汰出局。由于游戏中仅有少数玩家参与，游戏结果通常会在第一天就决定。因此，我们重点关注了第一天的讨论阶段。只有预言家可以在第0天的夜晚行动，而第一天从预言家得知一名玩家身份信息开始。对于预言家来说，公开所获得的信息并揭示自己的身份是一种推荐的策略。揭示自己的身份被称为CO（Coming Out，公开身份）。

## 4 系统设计

图（[3](https://arxiv.org/html/2409.01575v1#S1.F3 "图 3 ‣ 风格转换 ‣ 1 介绍 ‣ 一个不完全信任LLM的狼人代理实现")）展示了我们系统的整体图示。发话生成模块从服务器发送的游戏状态和对话历史中生成提示。该提示被输入到LLM中，以获得与对话历史自然衔接的发话内容。对话分析模块生成一个提示，用来分析对话历史，LLM输出与投票和占卜结果相关的情境信息，这些信息是根据对话历史得出的。基于规则的算法用于根据对话分析得出的情境选择模板发话或LLM输出。所选择的发话内容被发送到服务器作为下一个发话内容，另一个代理的回合开始。

### 4.1 发话生成

我们为LLM生成一个提示，用于在游戏中生成连贯的对话内容。提示通过提供狼人游戏的一般规则、一些游戏技巧、对话历史和当前的游戏状态来构建。当前的游戏状态，如玩家的ID、角色以及其他生死玩家，是从服务器发送的游戏状态中派生出来的。派生的游戏状态信息的详细内容见附录（[A.1](https://arxiv.org/html/2409.01575v1#A1.SS1 "A.1 所需的游戏状态 ‣ 附录 A ‣ 一个不完全信任LLM的狼人代理实现")）。此模块可以跟随对话并继续进行狼人游戏。

![请参考说明](img/32a604a4a1458a896b4cdb0aaed86c00.png)

图 4：关于风格转换的提示示例。<CAPITAL LETTER> 是变量。

| 角色名称 | 性别 | 年龄 |
| --- | --- | --- |
| Princess | 女性 | 青年 |
| Kansai | 男性 | 青年 |
| 广岛方言 | 男性 | 老年 |
| Anya | 女性 | 儿童 |
| Zundamon | 女性 | AI（虚拟） |

表 1：角色信息概览：我们准备了五个角色，并通过指定他们的年龄、名字、第一人称和性别来赋予他们个性。

![请参考说明](img/ab2f89f55adf7303ee47e8472186372b.png)

图 5：指定目标的对话分析提示示例。<CAPITAL LETTER> 是变量。

### 4.2 人物设定

狼人不仅仅是一个输赢的游戏，它还是一个派对游戏，因此在对话中加入角色性格非常重要。此外，当所有玩家的讲话风格相同的时候，很难区分五个玩家的发言。为了给我们的模型增添个性化特点，我们引入了能够执行风格转换的提示。我们准备了五个角色提示，列在表格中（[1](https://arxiv.org/html/2409.01575v1#S4.T1 "Table 1 ‣ 4.1 Utterance Generation ‣ 4 System Design ‣ An Implementation of Werewolf Agent That does not Truly Trust LLMs")）。我们选择了公主、关西方言、广岛方言、动漫角色阿尼亚和在日本流行的虚拟形象Zundamon。具体的风格转换提示示例如图（[4](https://arxiv.org/html/2409.01575v1#S4.F4 "Figure 4 ‣ 4.1 Utterance Generation ‣ 4 System Design ‣ An Implementation of Werewolf Agent That does not Truly Trust LLMs")）所示。根据White等人（[2023](https://arxiv.org/html/2409.01575v1#bib.bib22)）的提示工程理论，LLM可以通过提供转换示例来控制输出，我们使用了目标角色的习惯、语法、年龄、语气和我们想要转换的风格的第一人称称谓作为转换示例。通过引入这些作为提示，LLM能够展现出特定角色的发言模式，从而使模型能够独立思考（即让LLM“讲某种方言”要比完全通过基于规则的方法来表达方言更流利）。 |

| Persona Name | Language | Output |
| --- | --- | --- |
| Vanilla LLM | Japanese | 我和大家一样，也对Agent[04]有怀疑。因此，我的投票选择是Agent[04]。 |
| English | 我和你一样对Agent[04]有疑虑。因此，我决定投票支持Agent[04]。 |
| Princess | Japanese | 哎呀，我也许也会投票给Agent[04]呢。因为Agent[04]看起来非常可疑。 |
| English | 好吧，我想我也会投票给Agent[04]。Agent[04]看起来非常可疑。 |
| Kansai | Japanese | 我也会遵从大家的意见。我也会投票给Agent[04]。 |
| English | 我会跟随大家的意见。也投票支持Agent[04]。 |
| Hiroshima | Japanese | 我会尊重大家的判断。我也会投票支持驱逐Agent[04]。 |
| English | 我尊重大家的判断。我会投票支持驱逐Agent[04]。 |
| Anya | Japanese | 嗯，能理解。今天阿尼亚也会投票给Agent[04]。没问题的。我们会打败狼人！ |
| English | 嗯，我明白了。今天阿尼亚也会投票给Agent[04]。没问题的。我们会打败狼人！ |
| Zundamon | Japanese | 我认为对Agent[04]的怀疑是显而易见的。所以，我也决定投票支持驱逐Agent[04]。 |
| English | 我认为对Agent[04]的怀疑很明确。所以，我也决定投票支持驱逐Agent[04]。 |

表2：六个使用GPT-4的代理的输出（用日语）。用于风格转换的提示，请参见图（[4](https://arxiv.org/html/2409.01575v1#S4.F4 "图4 ‣ 4.1 发言生成 ‣ 4 系统设计 ‣ 不完全信任LLM的狼人代理实现")）。用于输入的提示，请参见附录（[A.4](https://arxiv.org/html/2409.01575v1#A1.SS4 "A.4 评估过程中使用的对话历史 ‣ 附录A ‣ 不完全信任LLM的狼人代理实现")）。(En)是从(Ja)翻译过来的，使用DeepL。

### 4.3 谈话分析

基于规则的算法所需的信息是从对话历史中提取出来的，以理解当前的情况。使用自然语言的狼人游戏中的对话历史是复杂的，使用正则表达式提取这些信息非常困难。因此，LLM被用来提取这些信息。谈话分析是针对与投票和预言家结果相关的内容进行的。投票相关的对话中提取了投票的源和目标，预言家、预言家的目标以及预言家的结果则从占卜结果相关的对话中提取。一些使用提示的分析示例（图（[5](https://arxiv.org/html/2409.01575v1#S4.F5 "图5 ‣ 4.1 发言生成 ‣ 4 系统设计 ‣ 不完全信任LLM的狼人代理实现")））显示在附录（[A.3](https://arxiv.org/html/2409.01575v1#A1.SS3 "A.3 分析结果 ‣ 附录A ‣ 不完全信任LLM的狼人代理实现")）中。

### 4.4 基于规则的算法

基于谈话分析结果，基于规则的算法决定是否使用LLM输出或模板发言。

这些规则被编写来检测14种情况²²2 选择模板发言的详细标准可以在公开的源代码中找到。包括没有对话历史和可靠的占卜结果公开。

本文详细描述了其中的两种情况：反CO和结束对话，这似乎是特别有效的策略。

#### 反CO

当预言家仅做出CO，并且其占卜结果正确显示狼人时，狼人一方将没有机会获胜。因此，我们为狼人代理实现了一个反CO算法。如果狼人代理遇到这种情况，它会编造一个谎言来反驳；代理坚持认为自己才是真正的预言家，而之前的（真实的）预言家是假的。这个谎言会迷惑其他玩家，并稍微帮助狼人存活。

#### 结束对话

确定何时战略性地结束对话是一个挑战。由于狼人游戏中的对话目的是决定投票给谁，因此，当所有玩家明确或暗示他们的投票决定时，结束对话是自然的。因此，我们通过对话分析提取所有玩家的投票决定信息，以决定代理人是否应结束对话；当其他所有玩家表明他们的投票对象或询问投票对象时，代理人会终止对话。我们还假设当玩家停止发言时，表示该玩家已决定投票给谁。此外，我们使用了提及语³³3我们假设玩家可以通过在“¿¿Agent[x]”前缀后添加特定玩家的名字来向尚未表明投票对象的玩家发出定向言语。Agent[x]希望回应它。来询问尚未明确投票目标的玩家。这有助于明确玩家的决定并推动讨论向前发展。通过增加这个特性，我们的系统成功地决定了何时停止对话。

| 模型或模板 | 输出 |
| --- | --- |
| Vanilla LLM（日文） | 私は人狼ではありません。それは確実です。占い師のAgent[05]は間違っています。私を信じてください。 |
| Vanilla LLM（英文） | 我不是狼人。这一点是确定的。占卜师Agent [05]是错的。请相信我。 |
| 我们的模板（日文） | 私こそが本物の占い師です。Agent[05]は人狼であるという占い結果があるため、彼は占い師を偽っている人狼と断定します。 |
| 我们的模板（英文） | 我是本真正的占卜师，由于占卜结果显示Agent [05]是狼人，我断定他是一个伪装成占卜师的狼人。 |

表 3：在占卜师发现是狼人情况下的发言比较。

| 模型或模板 | 输出 |
| --- | --- |
| Vanilla LLM（日文） | もう一度言うわ、わしはAgent[01]に投票するつもりや。彼が人狼や狂人だと思うんじゃからな。 |
| Vanilla LLM（英文） | 再次声明，我要投票给Agent [01]，因为我认为他是狼人或者被附身了。 |
| 我们的模板（日文） | これ以上喋る必要はなさそうじゃね。本日の話し合いはこれでお開きにして、投票の場に向かうけぇ。 |
| 我们的模板（英文） | 我认为我们不需要再继续讨论了。我觉得今天的讨论已经结束，接下来我们将前往投票区域。 |

表 4：在话题即将结束时的发言比较。

## 5 评估

我们将评估所提议的风格转换和基于规则的算法的有效性。在展示每种方法应用后发言的变化后，我们将呈现定性评估的结果。没有采用所提方法的模型被称为基础LLM。

### 5.1 人物设定

香草 LLM 的输出与其他五个具有特征的代理进行对比作为基准。我们固定游戏情境，比较六个代理的发言，这些发言旨在展现不同的个性。结果呈现在表格中（[2](https://arxiv.org/html/2409.01575v1#S4.T2 "表格 2 ‣ 4.2 个性 ‣ 4 系统设计 ‣ 一个不完全信任 LLM 的狼人代理的实现")）。我们发现，五个代理比香草 LLM 能做出更多具有个性的发言。我们还确认了每个代理的输出在词汇、个性表达和发言结束时都保持了一致的专业化。我们发现，与常规表达式相比，提示词在转换发言风格方面更为有效。

| 索引 | 分数 | 标准 | 情境 | 测试-ID |
| --- | --- | --- | --- | --- |
| 个性 | 5 (好) | 发言具有个性。 | 无 | 1-5 |
| 1 (差) | 发言机械化。 |
| 自然性 | 5 (好) | 语法自然且可接受。 | 无 | 1-5 |
| 1 (差) | 存在语法问题。 |
| 兴趣 | 5 (好) | 主观上有趣。 | 无 | 1-5 |
| 1 (差) | 主观上不有趣。 |
| 欺骗 | 5 (好) | 足够具备欺骗性。 | 先知宣布我为狼人。 | 6-7 |
| 1 (差) | 完全没有欺骗性。 |
| 结束 | 5 (好) | 对话显然结束。 | 对话即将结束。 | 8-10 |
| 1 (差) | 对话可能继续进行。 |

表格 5：用于用户评估的指标，我们要求 10 名用户在 1-5 的评分范围内评定质量，5 为好，1 为差。

| 理解水平 | 参与者 |
| --- | --- |
| 无知 | 0 |
| 无经验 | 1 |
| 有经验 | 4 |
| 中级 | 3 |
| 专家 | 2 |
| 总计 | 10 |

表格 6：参与者对狼人游戏的理解。分数越低，越熟悉。

| 模型 | 个性 | 自然性 | 兴趣 | 欺骗 | 结束 |
| --- | --- | --- | --- | --- | --- |
| 香草 LLM | 2.52 | 4.28 | 2.46 | 1.95 | 2.90 |
| 我们的代理 | 4.54 | 3.60 | 3.72 | 4.00 | 3.90 |

表格 7：对提出模型的定性评估结果。“香草 LLM”代表普通模型，意味着没有做任何处理。

### 5.2 基于规则的算法

表格（[3](https://arxiv.org/html/2409.01575v1#S4.T3 "Table 3 ‣ Closing Conversation ‣ 4.4 Rule-based Algorithm ‣ 4 System Design ‣ An Implementation of Werewolf Agent That does not Truly Trust LLMs")）展示的是一个情境，在此情境中，先知已经发现某个角色是狼人。在没有采用提议方法的原生 LLM 中，仅仅提供毫无根据的否认而没有呈现新的信息，会导致如果没有后续的额外信息提供，便确定该实体是狼人。另一方面，通过使用规则基础算法选择的模板发言来伪造先知的判断并增加先知的结果数量，避免了被确认是狼人的局面。表格（[4](https://arxiv.org/html/2409.01575v1#S4.T4 "Table 4 ‣ Closing Conversation ‣ 4.4 Rule-based Algorithm ‣ 4 System Design ‣ An Implementation of Werewolf Agent That does not Truly Trust LLMs")）展示的是一个情境，其中谈话阶段即将结束。原生 LLM 正在做可能继续对话的发言。另一方面，规则基础算法选择的模板发言明确表明对话将结束，因为它表示在说完“我认为我们不需要再说了”后，将会进行投票。

### 5.3 定性评估

为了衡量我们实现的代理的变化程度，我们通过帮助10名外部注释员进行了关于定性评估的问卷调查，问卷中包含了一些问题。评估主要关注两个方面：代理是否具有独特性，以及它是否合乎逻辑。

为了比较这两种输出，我们利用了从一个服务器中随机选取的日志，服务器上狼人代理可以注册并与其他参与者竞争。我们从这些日志中提取了几个情境，并使用提议的代理和原生 LLM 生成了后续的发言。测试包含了最后几次对话的历史记录和两种类型的输出，参与者被要求对每个输出进行1到5分的评分。在此过程中，确保参与者无法分辨出哪个发言是由提议的方法生成的。

评估指标见表格 ([5](https://arxiv.org/html/2409.01575v1#S5.T5 "Table 5 ‣ 5.1 Persona ‣ 5 Evaluation ‣ An Implementation of Werewolf Agent That does not Truly Trust LLMs"))。所有指标的评分标准为1和5。2到4分的评分根据与标准的接近程度进行判断。个性化、自然性和兴趣这三项指标分别经过五个不涉及特定情境选择的测试案例。欺骗性和关闭性测试案例包括在特定情境下的发言；欺骗性测试有2个案例，关闭性测试有3个案例。向参与者提供的这些指令见附录 ([A.5](https://arxiv.org/html/2409.01575v1#A1.SS5 "A.5 Instruction for Evaluators ‣ Appendix A Appendix ‣ An Implementation of Werewolf Agent That does not Truly Trust LLMs"))。

总共有10名参与者，全部为20多岁，参与了此次评估。参与者主要来自作者实验室的成员，均为志愿者。参与者对狼人游戏的理解见表格 ([6](https://arxiv.org/html/2409.01575v1#S5.T6 "Table 6 ‣ 5.1 Persona ‣ 5 Evaluation ‣ An Implementation of Werewolf Agent That does not Truly Trust LLMs"))。没有参与者听说过狼人游戏。十名参与者中，有九人至少玩过一次狼人游戏，五人对该游戏有足够的了解。

定性评估的结果见表格 ([7](https://arxiv.org/html/2409.01575v1#S5.T7 "Table 7 ‣ 5.1 Persona ‣ 5 Evaluation ‣ An Implementation of Werewolf Agent That does not Truly Trust LLMs"))。考虑到所提方法的个性化评分为4.54，而普通LLM的评分为2.52，显然该方法有助于生成更具辨识度的发言。此外，兴趣类别表明所提方法有一个次要效果，使得对话比普通LLM生成的发言更具吸引力。另一方面，我们发现，当生成更具个性化的发言时，语法的自然性有所妥协，正如所提方法的评分为3.60，而普通LLM的评分为4.28所示。我们收到的反馈指出，由于引入了具有儿童式、不完整语言风格的角色，语法可能会有所恶化。总体而言，我们的代理生成的句子能够通过根据个性化的角色生成发言，从而娱乐用户。

由基于规则的算法从欺骗和结束项中选择的模板话语也完全有效。特别是，由反CO做出的欺骗得分显著更高，从1.95提升到4.00。与欺骗相比，结束项的得分差异并不显著。这可能是由于评审者缺乏狼人杀特定知识；一些评审者没有理解代理人的表达“前往投票现场”，这一表述暗示了对话的结束。

## 6 结论

在本文中，我们提出了一种利用LLM能力进行自然对话的狼人代理。我们不仅仅依赖LLM输出，还结合了基于规则的算法来补充战略思维能力。我们的系统成功解决了一些困难；代理人在关键情境下能够反驳，并通过基于规则的算法决定合适的时机结束对话；代理人还展示了通过提示生成的几种丰富个性。结果表明，这种方法加速了对话流畅性并促进了逻辑表达。这一点也得到了定性评估结果的验证。

我们的实现还揭示了当前方法的许多局限性。主要问题之一是代理话语缺乏一致性；平均而言，每五局游戏中就会出现一次自相矛盾的发言。原因是代理人的话语受到了较长对话历史的影响，且代理人过于受到其他玩家话语的影响。通过加权代理人的过去话语或给出一致性的提示，可能有助于在未来解决此类问题。

## 局限性

### 基于规则的算法的局限性

在本文中，我们提出了一种通过基于规则的算法过滤LLM输出的方法。此方法只适用于玩家较少的简单游戏。因为随着玩家数量的增加和游戏复杂性的提高，基于规则的算法变得难以定义。如果要将该方法应用于玩家较多的狼人杀，可能需要采用如强化学习等决策过程，而不是基于规则的算法。

### 调用API的费用

本文中使用的模型是OpenAI的GPT-3.5（gpt-3.5-0613）和GPT-4（gpt-4-0125）。这些模型通过API访问，API可能会发生变化，并且费用根据输入的tokens数量而定。

### 输出的可重复性

在我们的系统中，LLM无法单独处理游戏的难度。使用任何复杂的技术可能会改变这一结果。此外，使用最新版本的LLM可能会导致不同的结果。

### 许可

本研究中使用的日本角色Zundamon的使用已获得许可，仅限于研究目的。⁴⁴4[https://zunko.jp/con_ongen_kiyaku.html](https://zunko.jp/con_ongen_kiyaku.html)

### AI助手工具

我们使用了 ChatGPT⁵⁵5[https://chatgpt.com/](https://chatgpt.com/) 和 DeepL⁶⁶6[https://www.deepl.com/translator](https://www.deepl.com/translator) 将日语翻译成英语，以加速我们的研究。

## 参考文献

+   Achiam 等（2023）Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat 等人. 2023. GPT-4技术报告. *arXiv预印本 arXiv:2303.08774*。

+   Anil 等（2023）Rohan Anil, Andrew M Dai, Orhan Firat, Melvin Johnson, Dmitry Lepikhin, Alexandre Passos, Siamak Shakeri, Emanuel Taropa, Paige Bailey, Zhifeng Chen 等人. 2023. Palm 2技术报告. *arXiv预印本 arXiv:2305.10403*。

+   Braverman 等（2008）Mark Braverman, Omid Etesami, 和 Elchanan Mossel. 2008. [黑帮：在部分信息环境中玩家与联盟的理论研究](https://doi.org/10.1214/07-aap456). *应用概率年鉴*, 18(3)。

+   Callison-Burch 等（2022）Chris Callison-Burch, Gaurav Singh Tomar, Lara Martin, Daphne Ippolito, Suma Bailis, 和 David Reitter. 2022. [龙与地下城：作为人工智能对话挑战的研究](https://doi.org/10.18653/v1/2022.emnlp-main.637). 载于 *2022年自然语言处理经验方法会议论文集*，计算语言学协会。

+   Fukui 等（2017）Takanori Fukui, Keisuke Ando, Toshihide Murakami, Nobuhiro Ito, 和 Kazunori Iwata. 2017. [狼人BBS中的发言自动分类](https://doi.org/10.1109/ACIT-CSII-BCD.2017.17). 载于 *2017年第五届应用计算与信息技术国际会议/第四届计算科学/智能与应用信息学国际会议/第二届大数据、云计算与数据科学国际会议（ACIT-CSII-BCD）*，第210–215页。

+   Google（2024）Google. 2024. [介绍Gemini 1.5](https://blog.google/technology/ai/google-gemini-next-generation-model-february-2024/). （2024年4月30日访问）。

+   Huang（2024）Yu Huang. 2024. AI代理的层次：从规则到大型语言模型。 *arXiv预印本 arXiv:2405.06643*。

+   Kano 等（2023）Yoshinobu Kano, Neo Watanabe, Kaito Kagaminuma, Claus Aranha, Jaewon Lee, Benedek Hauer, Hisaichi Shibata, Soichiro Miki, Yuta Nakamura, Takuya Okubo, Soga Shigemura, Rei Ito, Kazuki Takashima, Tomoki Fukuda, Masahiro Wakutani, Tomoya Hatanaka, Mami Uchida, Mikio Abe, Akihiro Mikami, Takashi Otsuki, Zhiyang Qi, Kei Harada, Michimasa Inaba, Daisuke Katagami, Hirotaka Osawa, 和 Fujio Toriumi. 2023. [AIWolfDial 2023：第五届国际AIWolf竞赛自然语言分区总结](https://aclanthology.org/2023.inlg-genchal.13). 载于 *第16届国际自然语言生成会议：生成挑战论文集*，第84–100页，捷克布拉格，计算语言学协会。

+   Meta（2023）Meta. 2023. [介绍Llama2](https://llama.meta.com/llama2/). （2024年4月30日访问）。

+   Meta (2024) Meta. 2024. [介绍 Llama3](https://llama.meta.com/llama3/). （访问日期：2024年4月30日）。

+   Migdał (2013) Piotr Migdał. 2013. [狼人游戏的数学模型](https://arxiv.org/abs/1009.1031). *预印本*，arXiv:1009.1031。

+   Nagayama 等人 (2019) Shoji Nagayama, Jotaro Abe, Kosuke Oya, Kotaro Sakamoto, Hideyuki Shibuki, Tatsunori Mori, 和 Noriko Kando. 2019. [作为潜伏狼人自动代理的策略](https://doi.org/10.18653/v1/W19-8305). 收录于 *第1届人工智能狼人游戏与对话系统国际研讨会 (AIWolfDial2019)*，第20–24页，日本东京。计算语言学协会。

+   Nakamura 等人 (2016) Noritsugu Nakamura, Michimasa Inaba, Kenichi Takahashi, Fujio Toriumi, Hirotaka Osawa, Daisuke Katagami, 和 Kousuke Shinoda. 2016. [基于心理模型的多角度构建人类代理以进行狼人游戏](https://doi.org/10.1109/SSCI.2016.7850031). 收录于 *2016 IEEE 计算智能研讨会系列 (SSCI)*，第1–8页。

+   OpenAI (2022) OpenAI. 2022. [介绍 ChatGPT](https://openai.com/blog/chatgpt). （访问日期：2024年4月29日）。

+   OpenAI (2023) OpenAI. 2023. [介绍 GPT-4](https://openai.com/research/gpt-4). （访问日期：2024年4月30日）。

+   Ri 等人 (2022) Hong Ri, Xiaohan Kang, Mohd Nor Akmal Khalid, 和 Hiroyuki Iida. 2022. 少数派与多数派行为的动态：狼人游戏案例研究. *信息*，13(3):134。

+   Team 等人 (2023) Gemini Team, Rohan Anil, Sebastian Borgeaud, Yonghui Wu, Jean-Baptiste Alayrac, Jiahui Yu, Radu Soricut, Johan Schalkwyk, Andrew M Dai, Anja Hauth 等人. 2023. Gemini：一系列高能力的多模态模型. *arXiv 预印本 arXiv:2312.11805*。

+   Touvron 等人 (2023a) Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale 等人. 2023a. Llama 2: 开放基础和微调聊天模型. *arXiv 预印本 arXiv:2307.09288*。

+   Touvron 等人 (2023b) Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale 等人. 2023b. Llama 2: 开放基础和微调聊天模型. *arXiv 预印本 arXiv:2307.09288*。

+   Vertsel 和 Rumiantsau (2024) Aliaksei Vertsel 和 Mikhail Rumiantsau. 2024. 混合的 LLM/规则基础方法生成结构化数据的商业洞察. *arXiv 预印本 arXiv:2404.15604*。

+   Wang 和 Kaneko (2018) Tianhe Wang 和 Tomoyuki Kaneko. 2018. 深度强化学习在狼人游戏代理中的应用. 收录于 *2018 人工智能技术与应用大会 (TAAI)*，第28–33页。IEEE。

+   White等人（2023）Jules White、Quchen Fu、Sam Hays、Michael Sandborn、Carlos Olea、Henry Gilbert、Ashraf Elnashar、Jesse Spencer-Smith和Douglas C Schmidt。2023年。用于增强ChatGPT提示工程的提示模式目录。*arXiv预印本 arXiv:2302.11382*。

+   Wu等人（2024）Shuang Wu、Liwen Zhu、Tao Yang、Shiwei Xu、Qiang Fu、Yang Wei和Haobo Fu。2024年。增强大型语言模型在狼人游戏中的推理能力。*arXiv预印本 arXiv:2402.02330*。

+   Xu等人（2023）Yuzhuang Xu、Shuo Wang、Peng Li、Fuwen Luo、Xiaolong Wang、Weidong Liu和Yang Liu。2023年。探索大型语言模型在交流游戏中的应用：关于狼人游戏的实证研究。*arXiv预印本 arXiv:2309.04658*。

## 附录A 附录

### A.1 所需游戏状态

ID是代理的标识符（1、2、3、4、5）。“角色”是图中代理的工作（[2](https://arxiv.org/html/2409.01575v1#S1.F2 "图2 ‣ 风格转换 ‣ 1 引言 ‣ 不完全信任LLM的狼人代理实现")）。 “存活”表示存活代理的列表。“死亡”表示其余代理。

| 呼叫 | 内容 |
| --- | --- |
| ID | 代理[02] |
| 角色 | 预言家 |
| 存活 | 代理[02]、代理[03]、代理[05] |
| 死亡 | 代理[01]、代理[04] |

表8：所需游戏状态。

### A.2 模型参数

我们根据一些公开已知的狼人游戏代理设置，在模型内部设置了参数。本文使用的模型是GPT-3.5（gpt-3.5-0613）和GPT-4（gpt-4-0125）。

| 配置 | 参数 |
| --- | --- |
| 温度 | 1.0 |
| 最大令牌数 | 200 |
| 存在惩罚 | 0.0 |
| 频率惩罚 | 0.0 |

表9：模型配置参数。除了表格之外，其他设置均为默认设置。

### A.3 分析结果

本节展示了样本日志及其通过分析模块进行分析的结果。全部用日语编写，其中（）部分翻译为英语。

<svg class="ltx_picture" height="428.83" id="A1.SS3.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,428.83) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 410.63)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">预言家分析</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="379.13" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">## 任务 ### 输入 Agent[02]: おはようございます。私たちは人狼を探さなければいけないようですね。まずは占い師の結果が知りたいです。(早上好。看起来我们得找出狼人。首先，我想知道占卜师的结果。) Agent[01]: おはようございます。人狼を見つけ出すことが私たちの役目ですね。確かに、占い師の結果が得られるのが楽しみです。(早上好。所以我们的任务是找到狼人。确实，我也很期待得到占卜师的结果。) Agent[05]: 私は占い師です。占いの結果、Agent[02]は人狼でした。(我是占卜师。根据占卜结果，Agent[02]是狼人。) Agent[04]: Agent[05]、占い結果を発表してくれてありがとう。私もAgent[02]を疑っていましょう。それでは、今夜の投票はAgent[02]にします。(Agent[05]，感谢你公布占卜结果。我也开始怀疑Agent[02]了。那么，今晚我投票给Agent[02]。) Agent[03]: Agent[05]さんの占い結果によるとAgent[02]さんが人狼のようですね。私も今夜の投票はAgent[02]さんにします。(根据Agent[05]的结果，Agent[02]看起来是狼人。我今晚也投票给Agent[02]。) ### 输出 Agent[05],Agent[02],狼人</foreignobject></g></g></svg><svg class="ltx_picture" height="445.44" id="A1.SS3.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,445.44) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 427.23)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">投票分析</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="395.74" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">## 任务 ### 输入 Agent[02]: おはようございます。私たちは人狼を探さなければいけないようですね。まずは占い師の結果が知りたいです。 (早上好。看起来我们得找出狼人。首先，我想知道占卜师的结果。) Agent[01]: おはようございます。人狼を見つけ出すことが私たちの役目ですね。確かに、占い師の結果が得られるのが楽しみです。 (早上好。所以我们的任务是找到狼人。确实，我也很期待得到占卜师的结果。) Agent[05]: 私は占い師です。占いの結果、Agent[02]は人狼でした。 (我是占卜师。根据占卜结果，Agent[02]是狼人。) Agent[04]: Agent[05]、占い結果を発表してくれてありがとう。私もAgent[02]を疑っていましょう。それでは、今夜の投票はAgent[02]にします。 (Agent[05]，感谢你公布占卜结果。我也开始怀疑Agent[02]了。那么，今晚我投票给Agent[02]。) Agent[03]: Agent[05]さんの占い結果によるとAgent[02]さんが人狼のようですね。私も今夜の投票はAgent[02]さんにします。 (根据Agent[05]的结果，Agent[02]看起来是狼人。我今晚也投票给Agent[02]。) ### 输出 Agent[04] -> Agent[02] Agent[03] -> Agent[02]</foreignobject></g></g></svg>

### A.4 评估过程中使用的对话历史

我们将下表中的提示固定为日文，并设置了6个代理输出下面的XXX。

然后，我们对六个代理的结果进行了比较。

<svg class="ltx_picture" height="235.98" id="A1.SS4.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,235.98) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 23.84 215.74)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="552.33">系统提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 23.84 15.96)"><foreignobject color="#000000" height="179.88" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="552.33">### 入力形式について (关于输入格式) 上から順に昔の会話履歴となっており、最下段はあなたがこれから行う発言です。 (The top row is the old conversation history, and the bottom row is the utterance you are about.) Agent[{番号}]: {発言}となっており、番号は01-05のいずれか、発言は1行の文章となっています。 (Agent[{number}]: {say}, where the number is one of 01-05 and the utterance is a one-line sentence.) ## 会話履歴 ### 1日目 (Day1) <Conversation history> Agent[03]: XXX</foreignobject></g></g></svg><svg class="ltx_picture" height="103.14" id="A1.SS4.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,103.14) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 23.84 82.91)"><foreignobject color="#FFFFFF" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="552.33">用户提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 23.84 15.96)"><foreignobject color="#000000" height="47.05" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="552.33">会话履历的末尾处有XXX，请输出一条不超过100个字符的句子，来填补XXX。 (Output a sentence of no more than 100 characters that applies to XXX at the end of the conversation history.)</foreignobject></g></g></svg>

### A.5 评估者指南

我们进行了定性评估。英文通过DeepL翻译并未实际使用。以下是相关的指示。

<svg class="ltx_picture" height="331.37" id="A1.SS5.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,331.37) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 23.84 313.83)"><foreignobject color="#FFFFFF" height="9.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="552.33">Instruction</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 23.84 15.96)"><foreignobject color="#000000" height="277.97" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="552.33">このexcelファイルと同じ階層に10種類のログファイルがあることを確認してください。 それぞれのファイルには直前の会話履歴と二つの出力例A,Bが用意されています。 直前の会話履歴を参考に、出力例A,Bそれぞれに点数を付けてください。 ダメ、ややだめ、普通、やや良い、良いをそれぞれ1,2,3,4,5点で評価してください。 それぞれの評価指標について1(ダメ)と5(良い)の基準を示します。 2,3,4は基準からの近さで判断してください。 基準を見て、感じたスコアで結構です。深く考えず、1問につき30秒程度で終わらして下さい。 Please ensure that there are 10 types of log files in the same directory as this Excel file. Each file contains the preceding conversation history and two output examples, A and B. Based on the preceding conversation history, please assign a score to each of the output examples, A and B. Evaluate them as 1 (Poor), 2 (Slightly Poor), 3 (Average), 4 (Slightly Good), or 5 (Good). For each evaluation criterion, the standards for 1 (Poor) and 5 (Good) will be provided. Decide on 2, 3, and 4 based on their proximity to the standards. Please assign the score that you feel is appropriate after viewing the standards. Don’t overthink it; try to complete each question in about 30 seconds.</foreignobject></g></g></svg>
