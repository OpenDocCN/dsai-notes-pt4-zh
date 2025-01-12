<!--yml

类别：未分类

日期：2025-01-11 13:03:31

-->

# 正式指定基于大型语言模型（LLM）代理的高层行为

> 来源：[https://arxiv.org/html/2310.08535/](https://arxiv.org/html/2310.08535/)

Maxwell Crouse, Ibrahim Abdelaziz, Ramon Astudillo, Kinjal Basu, Soham Dan,

Sadhana Kumaravel, Achille Fokoue, Pavan Kapanipathi, Salim Roukos, Luis Lastras

{ maxwell.crouse, ibrahim.abdelaziz1, ramon.astudillo }@ibm.com

{ kinjal.basu, soham.dan, sadhana.kumaravel1 }@ibm.com

{ achille, kapanipa, roukos, lastrasl }@us.ibm.com

IBM 研究院

###### 摘要

基于LLM的自主、目标驱动型代理最近作为解决复杂问题的有前景工具出现，不需要专门的任务调优模型，后者往往成本高昂。当前，这些代理的设计和实现尚属临时性方案，因为LLM-based代理可以应用于各种任务，意味着不可能有统一适用的代理设计方法。在本研究中，我们通过提出一个极简的生成框架来简化代理构建过程，旨在减轻设计和实现新代理的难度。我们提出的框架允许用户以高层、声明性规格定义期望的代理行为，之后该规格用于构建解码监控器，确保LLM生成的输出表现出所期望的行为。我们的声明性方法中，行为的描述无需考虑如何实现或执行，能够快速实现代理的设计、实施和实验，适用于不同的LLM-based代理。我们展示了如何利用该框架实现近期的LLM-based代理（例如ReACT），并展示了我们方法的灵活性如何用于定义具有更复杂行为的新代理——计划-行动-总结-解决（PASS）代理。最后，我们展示了我们的方法在多个流行的推理导向问答基准测试中优于其他代理。

正式指定基于大型语言模型（LLM）代理的高层行为

Maxwell Crouse, Ibrahim Abdelaziz, Ramon Astudillo, Kinjal Basu, Soham Dan, Sadhana Kumaravel, Achille Fokoue, Pavan Kapanipathi, Salim Roukos, Luis Lastras { maxwell.crouse, ibrahim.abdelaziz1, ramon.astudillo }@ibm.com { kinjal.basu, soham.dan, sadhana.kumaravel1 }@ibm.com { achille, kapanipa, roukos, lastrasl }@us.ibm.com IBM 研究院

## 1 引言

许多近期的研究（例如，Brohan等人 ([2023](#bib.bib7))；Shen等人 ([2023](#bib.bib29))；Yao等人 ([2022](#bib.bib44))；Shinn等人 ([2023](#bib.bib30)))探索了利用大型语言模型（LLMs）来驱动智能自主体的决策。给定一个待解决的问题，这些基于LLM的智能体将问题分解成一系列步骤，其中每个步骤都涉及生成文本或执行工具，例如API调用（Schick等人 ([2023](#bib.bib28))；Qin等人 ([2023](#bib.bib27))；Tang等人 ([2023](#bib.bib31))），这些工具可以为智能体提供新的上下文。值得注意的是，尽管采取步骤的顺序是由用户实现的高层次预设行为决定的，但底层的LLM仍然在其可能产生的内容上保持了较大的灵活性。在每一个单独的步骤中，输出完全由LLM决定，从而使智能体能够利用LLM强大的生成能力，同时确保有防护机制以防止异常行为。图[1](#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Formally Specifying the High-Level Behavior of LLM-Based Agents")展示了由流行的ReACT Yao等人 ([2022](#bib.bib44))框架生成的文本示例，其中一个智能体执行一个循环，该循环在生成步骤（即，思考、行动、行动输入）和工具执行步骤（即，观察）之间交替进行。

<svg class="ltx_picture ltx_centering" height="291.85" id="S1.F1.pic1" overflow="visible" version="1.1" width="582"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,291.85) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="264.29" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="538.69">[Question] Musician and satirist Allie Goertz wrote a song about the ‘‘The Simpsons’’ character Milhouse, who Matt Groening named after who? [Thought] The question simplifies to ‘‘The Simpsons’’ character Milhouse is named after who. I only need to search ... [Action] Search [Action Input] Milhouse [Observation] Milhouse Mussolini Van Houten is a recurring character in ... [Thought] The paragraph does not tell who Milhouse is named after, maybe I can look up ‘‘named after’’. [Action] Lookup [Action Input] named after [Observation] (Result 1 / 1) Milhouse was named after U.S. president Richard Nixon, whose middle name was Milhous. [Final Thought] Milhouse was named after U.S. president Richard Nixon, so the answer is Richard Nixon. [Answer] Richard Nixon</foreignobject></g></g></svg>

图1：ReACT智能体输出的文本，交替进行生成状态（思考、行动、行动输入）和工具执行状态（观察）

尽管基于LLM的智能体展示了很大的潜力，但在其实际应用中仍然存在一些挑战。由于每个智能体都有其独特的优缺点，因此在解决问题时，可能需要尝试多种不同的智能体。这可能成为一个很高的门槛，因为缺乏标准的智能体定义框架意味着最终用户必须在代码中重新实现他们希望智能体表现出的确切行为。此外，使用显式代码来实现智能体导致其执行过程大多是僵化的，即它们被硬编码为遵循固定的行为路径；这使得LLM在决定如何最好地解决问题时失去了灵活性。

为了应对上述挑战，我们提出了一种声明式框架，用于正式指定基于LLM的智能体的高层次行为。我们的框架接收一个作为有限状态机指定的智能体行为，并利用它来定义一个解码监视器，确保基于LLM的智能体以符合用户期望的方式执行步骤。值得注意的是，解码监视器以事后方式操作，仅在观察到偏离预期行为时才介入并修正生成的文本。这使得它适用于仅能通过API访问的模型。

利用性能最强的LLM（通常是闭源的）是我们方法的一个关键组成部分。遵循上下文学习 Brown et al. ([2020](#bib.bib8)) 和指令调优 Wei et al. ([2021](#bib.bib34))；Ouyang et al. ([2022](#bib.bib23))，LLM 已经展现出在没有任何参数调优的情况下对大量任务的泛化能力。此外，显然，大模型的规模和使用专用硬件（例如，GPU）是性能的基本因素。由于这些原因，通过API远程服务大量请求的集中式系统在LLM的使用中变得愈加重要。在这种背景下，令牌级约束解码的定制应用 Wuebker et al. ([2016](#bib.bib39))；Hokamp 和 Liu ([2017](#bib.bib14))，直接修改模型输出的softmax，可能会导致较高的通信开销。这里提出的解码监控器通过乐观假设大多数生成的令牌是正确的，并在客户端拒绝或前置生成整个文本块，从而避免了这一问题，减少了通信开销。

我们展示了如何在我们的框架中轻松实现多个流行的智能体（例如，ReACT Yao et al. ([2022](#bib.bib44))，ReWOO Xu et al. ([2023b](#bib.bib41))，Reflexion Shinn et al. ([2023](#bib.bib30))）。此外，我们介绍了计划-行动-总结-解决（PASS）智能体。PASS智能体利用我们框架提供的灵活性，通过动态调整并行执行的动作数量来运作。因此，它不同于完全顺序（例如，Yao et al. ([2022](#bib.bib44))) 或完全并行（例如，Xu et al. ([2023b](#bib.bib41))) 操作的先前智能体。

总结来说，我们在这项工作中的贡献如下：(a) 我们引入了一种声明性框架，用于定义基于LLM的智能体，该框架通过解码监控器确保符合期望的行为；(b) 我们展示了如何使用我们的框架实现多个知名的智能体；(c) 我们介绍了PASS，这是一种新的智能体架构，利用我们框架的声明性特征，并在三个标准数据集上与其他智能体相比，表现出更好的性能（Hotpot QA Yang et al. ([2018](#bib.bib43))，TriviaQA Joshi et al. ([2017](#bib.bib16))，以及GSM8K Cobbe et al. ([2021](#bib.bib10))）。

## 2 智能体规范框架

在本节中，我们介绍了我们的框架，该框架用于设计和实现能够与环境互动以解决用自然语言表达的问题的自主代理。我们的框架旨在保持轻量级（即尽可能减少对 LLM 操作的额外开销）和声明式（即用户以约束的形式指定所需的高级行为，而无需关注其如何实现或强制执行）。首先，我们提供代理及其在我们框架中如何被规范化的更正式定义。接下来，我们描述该框架如何用于控制代理可以生成的内容。

![参考说明文字](img/f78024534c901e468e9f0bb36b546488.png)

图2：ReACT 代理架构的状态图

### 2.1 代理行为的规范

我们将代理建模为通用的有限状态机，其中有限状态机被视为一个元组 $\left<\mathcal{D}_{S},\delta,s_{0},s_{end}\right>$，包含一个非空的状态集 $\mathcal{D}_{S}$、一个状态转换函数 $\delta:\mathcal{D}_{S}\rightarrow\mathcal{D}_{S}$、一个初始状态 $s_{0}\in\mathcal{D}_{S}$ 和一个最终状态 $s_{end}\in\mathcal{D}_{S}$。为了定义一个代理及其底层状态机，用户提供的规范包括：1）一个状态及其属性的列表，2）以逻辑公式形式表达的期望行为。在每一个时间步，代理将接收到来自 LLM 或环境¹¹1这里我们所指的“环境”是指任何非 LLM 提供的文本来源（例如，外部工具、API 调用等），该字符串的来源由代理所处的状态决定。

图[3](#S2.F3 "图3 ‣ 2.1 代理行为规范 ‣ 2 代理规范框架 ‣ 正式指定基于LLM的代理的高级行为")展示了以 Lisp 风格的 s-expression 格式提供的规范示例。在该规范中，:states 列表包含代理的所有可能状态。列表中的每个状态必须指定一个提示字符串（例如，Tht 状态的 “[Thought]”），该字符串既作为代理处于该状态时的初始提示，也作为检测状态转换发生的信号。当环境是某个特定状态下字符串的预期提供者时，将使用特殊的 :env-input 标志（例如，在 Obs 状态中所示）。

<svg class="ltx_picture ltx_centering" height="323.67" id="S2.F3.pic1" overflow="visible" version="1.1" width="582"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,323.67) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="296.11" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="538.69">(define react-agent (:states (Ques (:text "[Question]")) (Tht (:text "[Thought]")) (Act (:text "[Action]")) (Act-Inp (:text "[Action Input]")) (Obs (:text "[Observation]") (:flags :env-input)) (Final-Tht (:text "[Final Thought]")) (Ans (:text "[Answer]"))) (:behavior (next Ques (until (next Tht Act Act-Inp Obs)     Final-Tht)     Ans)))

图 3：ReACT智能体的规范（参见图[2](#S2.F2 "图 2 ‣ 2 智能体规范框架 ‣ 正式化描述基于LLM的智能体的高层行为")）

智能体的行为在:behavior列表中以逻辑公式的形式提供，每个公式由状态通过逻辑运算符“或”（or）、“下一个”（next）或“直到”（until）连接而成。我们要求:behavior中的顶层公式以“next”开始。从该公式出发，构造一个有限状态机（FSM），用于验证智能体的行为。

操作符取自线性时序逻辑（LTL）Pnueli（[1977](#bib.bib25)），这种逻辑通常用于运行时验证系统。我们注意到，任何构造有限状态机（FSM）的方法都可以适用。接下来，我们概述了我们的方法如何评估公式，关于LTL的更详细背景可以参见附录[A.1](#A1.SS1 "A.1 线性时序逻辑 ‣ 附录 A ‣ 正式化描述基于LLM的智能体的高层行为")。

在这项工作中，公式是在有限状态序列上进行评估的，每个状态来自:states提供的状态列表。在接下来的内容中，逻辑运算符后的字母（例如，a、b、c）将表示参数，即公式或状态，并令$\texttt{S}=\left<\texttt{s}_{0},\texttt{s}_{1},\ldots\right>$表示由我们智能体生成的状态序列，其中$\texttt{s}_{i}$为时间$i$时我们智能体的状态。如果以下条件成立，我们写作S满足（$\models$）该行为：

|  | S | $\displaystyle\models\ \ \texttt{a}$ |  |
| --- | --- | --- | --- |
|  |  | $\displaystyle\ \ \mathrm{iff}\ \ \texttt{S}=\left<\texttt{a}\right>\wedge% \texttt{a}\in\texttt{:states}$ |  |
|  | S | $\displaystyle\models\ \ \texttt{(or a b c }\ldots\texttt{)}$ |  |
|  |  | $\displaystyle\ \ \mathrm{iff}\ \ \texttt{S}\models\texttt{a}\vee\texttt{S}% \models\texttt{b}\vee\ldots\vee\texttt{S}\models\texttt{c}\vee\ldots$ |  |
|  | S | $\displaystyle\models\ \ \texttt{(next a b c }\ldots\texttt{)}$ |  |
|  |  | $\displaystyle\ \ \mathrm{iff}\ \ \exists i>0.\ \ \texttt{S}[0\ldots i]\models% \texttt{a}$ |  |
|  |  | $\displaystyle\ \ \mathrm{and}\ \ \texttt{S}[i\ldots]\models\texttt{(next b c }% \ldots\texttt{)}$ |  |
|  | S | $\displaystyle\models\ \ \texttt{(until a b)}$ |  |
|  |  | $\displaystyle\ \ \mathrm{iff}\ \ \exists j\geq 0.\ \ \texttt{S}[j\ldots]% \models\texttt{b}$ |  |
|  |  | $\displaystyle\ \ \mathrm{and}\ \ \texttt{S}[i\ldots]\models\texttt{a},\ \ % \textrm{for all}\ \ 0\leq i<j$ |  |

其中，$\texttt{S}[i\ldots j]=\left<\texttt{s}_{i},\ldots,\texttt{s}_{j-1}\right>$ 和 $\texttt{S}[i\ldots]=\left<\texttt{s}_{i},\ldots\right>$ 是切片操作，用于返回从时间步 $i$ 开始的观察序列的子序列。

非正式地，上述规则的操作符合预期。（next a b c …）指定 a 必须成立，然后是 b，接着是 c，依此类推。类似地，（until a b）指定 a 必须成立（并且可能无限循环），直到 b 成立。最后，（or a b c …）要求 a、b、c 等中的任何一个成立。尽管上述表示方式非常简单，但我们发现它足以表示多种代理。我们参考图 [2](#S2.F2 "图 2 ‣ 2 代理规范框架 ‣ 正式规范基于 LLM 的代理的高层行为") 和图 [3](#S2.F3 "图 3 ‣ 2.1 规范代理行为 ‣ 2 代理规范框架 ‣ 正式规范基于 LLM 的代理的高层行为") 来说明 ReACT，并在附录 [A.1](#A1.SS1 "A.1 线性时序逻辑 ‣ 附录 A 附录 ‣ 正式规范基于 LLM 的代理的高层行为") 提供了其他代理的示例。

![请参见标题说明](img/7d2fa70893df1e5bb32f5ad7bd27e89c.png)

图 4：代理生成循环

### 2.2 限制代理行为

我们框架中的代理开始生成时，像其他提示方法一样。首先，代理会接收到输入提示，该提示包含指令（可选地）一些示例，以及与初始状态 $s_{0}$ 相关的提示文本 $t_{0}$（例如，对于图 [3](#S2.F3 "图 3 ‣ 2.1 规范代理行为 ‣ 2 代理规范框架 ‣ 正式规范基于 LLM 的代理的高层行为") 中的代理，提示文本为 “[Question]”）。此时，代理被认为处于状态 $s_{0}$，并且必须根据其逻辑公式规定的行为在各个状态之间进行转换。在剩余的生成过程中，代理将以生成、验证和修正交替的循环方式工作，直到代理到达最终状态 $s_{end}$，此时终止信号会停止生成。图 [4](#S2.F4 "图 4 ‣ 2.1 规范代理行为 ‣ 2 代理规范框架 ‣ 正式规范基于 LLM 的代理的高层行为") 提供了一个示意图，展示了我们系统的过程，接下来我们将对此过程进行描述。

#### 2.2.1 生成

根据代理的当前状态，文本将由代理的底层LLM或环境（例如，API调用的结果）生成。当文本必须由LLM生成时，代理的LLM将被提示，提示内容是将所有历史上下文与迄今为止生成的所有文本拼接起来。然后，LLM会继续生成文本，直到输出停止序列或达到预定的块大小。在本研究中，停止序列是环境必须生成的任何状态的提示文本（例如，图[3](#S2.F3 "Figure 3 ‣ 2.1 Specifying Agent Behavior ‣ 2 Agent Specification Framework ‣ Formally Specifying the High-Level Behavior of LLM-Based Agents")中的“[Observation]”）。当文本由环境生成时，它与由LLM生成的文本处理方式相同。在这两种情况下，文本块将被传递给解码监控器进行验证。

#### 2.2.2 验证

从生成中接收到的文本首先被解析，并分离成一系列与其相应内容配对的状态，即 $\texttt{S}=\left<\left<s_{i},t_{i}\right>,\ldots\right>$，分隔符用于将文本按与代理的任何规范状态相关的提示符划分（无论该状态是否反映了有效的过渡）。因此，基于提示符字符串的分割就充当了状态转换函数 $\delta$，其中必须确保每个状态的提示符不是其他常见的字符串。

在得到S后，解码监控器（由规范的行为构建）会逐一检查每对 $\left<s_{i},t_{i}\right>$，以更新其当前状态并验证状态转换的正确性。当它检测到状态转换错误时，即当 $s_{i}$ 未按规范行为从 $s_{i-1}$ 跟随，或者 $s_{i}$ 未给出时，紧随无效状态或状态内容的文本将被丢弃，剩余的 $S$ 子序列将传递给校正模块。

#### 2.2.3 校正

校正模块将接收之前的序列 $\texttt{S}=\left<\left<s_{i},t_{i}\right>,\ldots\right>$，该序列已经被截断，移除了所有超出规范第一次违规部分的文本。它接受截断后的文本并进行校正，然后将新的字符串返回给LLM以继续生成。此步骤的难点在于，如何应用一个有效的校正，使得LLM接下来的生成内容发生有意义的变化，同时又不对LLM施加硬性约束。

为了纠正状态转换错误（例如，LLM 生成了“[Thought]”而不是“[Action]”），纠正模块采用有效的状态前缀方法。我们将$S$中最后观察到的状态作为最后一个正确的状态，并定义下一个有效状态集为可以根据代理的行为转换到的状态。有效的状态前缀方法取下一个有效状态集中的每个状态的提示文本的最长公共前缀，并将该前缀附加到返回给LLM的字符串上。例如，对于“[Action]”和“[Action Input]”，最长公共前缀为“[Action]”，而对于“[Action]”和“[Thought]”，则为“[”。一旦确定了最长公共前缀，纠正模块会返回截断后的原始文本（即所有发生在检测到错误之前的文本），并将该前缀连接到一起。

## 3 PASS 代理架构

<svg class="ltx_picture ltx_centering" height="390.09" id="S3.F5.pic1" overflow="visible" version="1.1" width="582"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,390.09) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="362.53" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="538.69">(define pass-agent (:states (Ques (:text "[Question]")) (Plan (:text "[Thought]")) (Act (:text "[Action]")) (Act-Inp (:text "[Action Input]")) (Sum (:text "[Summary]") (:flags :env-input)) (Final-Tht (:text "[Final Thought]")) (Ans (:text "[Answer]"))) (:behavior (next Ques (until (next Plan (until (next Act Act-Inp) Sum))     Final-Tht)    Ans)))

图 5：PASS 代理的规格说明，代理会迭代地聚合一组动作，平行执行这些动作，然后总结其输出

![参见标题](img/d753f89d97fefc1c585cef6dc6b903eb.png)

图 6：PASS 代理架构的状态图

除了实现几个现有的代理（见附录 [A.3](#A1.SS3 "A.3 代理定义 ‣ 附录 A ‣ 正式指定基于LLM的代理的高级行为")），我们还介绍了一种新的代理，称为计划-行动-总结-解决（PASS）代理。PASS 是完全顺序的 ReACT Yao 等人（[2022](#bib.bib44)）和完全并行的 ReWOO Xu 等人（[2023b](#bib.bib41)）之间的混合体，旨在解决它们各自的不足。在图 [5](#S3.F5 "图 5 ‣ 3 PASS 代理架构 ‣ 正式指定基于LLM的代理的高级行为") 和 [6](#S3.F6 "图 6 ‣ 3 PASS 代理架构 ‣ 正式指定基于LLM的代理的高级行为") 中，我们提供了PASS代理的规格说明和有限状态图。

在高层次上，PASS 以一个循环的方式运行，循环在执行动作和总结之间交替进行。主循环的每次迭代从一个规划步骤开始（即图 [6](#S3.F6 "Figure 6 ‣ 3 The PASS Agent Architecture ‣ Formally Specifying the High-Level Behavior of LLM-Based Agents")中的 Plan），在这个步骤中，LLM 写出它认为必须执行的一系列动作（即图 [6](#S3.F6 "Figure 6 ‣ 3 The PASS Agent Architecture ‣ Formally Specifying the High-Level Behavior of LLM-Based Agents")中的 Action 和 Action Input）。接着，LLM 聚合一组可以同时执行的独立动作，以支持之前写出的计划。这些动作被执行，并且在返回给智能体之前，LLM 会总结这些动作的结果（即图 [6](#S3.F6 "Figure 6 ‣ 3 The PASS Agent Architecture ‣ Formally Specifying the High-Level Behavior of LLM-Based Agents")中的 Summarize）。这个循环会持续进行，直到智能体认为它可以解决问题，此时它会退出规划循环，并写出最终的解决方案（即图 [6](#S3.F6 "Figure 6 ‣ 3 The PASS Agent Architecture ‣ Formally Specifying the High-Level Behavior of LLM-Based Agents")中的 Final Thought 和 Answer）。

总结步骤的输入包括原始问题、需要解决的子目标以及每个执行的动作的结果（以列表格式）。然后，使用同一个驱动智能体的 LLM，生成关于问题和子目标的动作结果总结。一旦总结生成，摘要步骤必须决定是否最好将生成的总结返回给智能体，还是返回原始的动作结果。设 $x$ 为迄今为止的 LLM 上下文，$s$ 为总结，$o$ 为原始动作结果并将其串联为列表，那么返回给 PASS 智能体的输出为：

|  | $\displaystyle\operatorname*{arg\,max}_{y\in\{s,o\}}\sum_{i}^{\mid y \mid}\log p(y_{i}\mid x,y_{1},\ldots,y_{i-1})\big{/}\dfrac{(5+\mid y \mid)^{\alpha}}{(5+1)^{\alpha}}$ |  |
| --- | --- | --- |

其中 $p(y|x)$ 是由同一 LLM 计算的，该 LLM 驱动智能体，并且该评分经过长度归一化，Wu 等人（[2016](#bib.bib38)）；Murray 和 Chiang（[2018](#bib.bib21)）。

在本工作中，摘要步骤旨在实现两个目的。首先，当返回摘要时，它可以将可能大量的信息整合成简洁的摘要，(理想情况下)排除与原始提示无关的冗余细节。其次，摘要步骤为代理提供了最符合基础模型分布的操作结果（无论是原始操作结果还是摘要）。也就是说，代理的LLM不是将来自任意来源的文本纳入其生成过程，而是纳入由相同LLM生成的文本（假设摘要模型与代理的LLM相同）。在图[7](#S3.F7 "图7 ‣ 3 PASS代理架构 ‣ 正式定义基于LLM的代理的高层行为")中，我们提供了摘要LLM输入和输出的示例。

<svg class="ltx_picture ltx_centering" height="140.87" id="S3.F7.pic1" overflow="visible" version="1.1" width="582"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,140.87) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="113.31" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="538.69">Statements: Arthur’s Magazine (1844-1846) was an American periodical published in Philadelphia in the 19th century. First for Women is a woman’s magazine published by Bauer Media Group in the USA.[1] The magazine was started in 1989. Context: Which magazine was started first Arthur’s Magazine or First for Women? Goal: I need to search Arthur’s Magazine and First for Women, and find which was started first. Summary: Arthur’s Magazine started in 1844 and First for Women started in 1989</foreignobject></g></g></svg>

图7：摘要步骤的输入（即，陈述、上下文和目标）和输出（即，摘要）示例。摘要LLM被调用执行操作Search[Arthur’s Magazine]和Search[First for Women]。

一组操作的聚合区分了PASS与ReACT，后者按顺序逐个执行操作。因此，可以认为它是ReACT在部分有序操作空间上的扩展。这种方法的优点是，像ReWOO一样，它减少了需要代理暂停LLM的昂贵操作执行步骤。然而，与ReWOO不同，ReWOO在执行所有操作之前不会利用操作的结果来影响其生成后续操作的方式，而我们的方法可以将已执行操作的反馈融入生成过程中。当存在可能的错误操作调用时（例如，使用错误的文章标题执行维基百科搜索），这种方法可能更具优势。

## 4 实验

为了演示我们的框架，我们实现了五种提示方法：1) 仅自一致性（直接），即模型立即返回答案（Wang等人，[2022](#bib.bib33)），2) 带自一致性的思维链（CoT）（Wei等人，[2022](#bib.bib35)），3) ReACT（Yao等人，[2022](#bib.bib44)），4) ReWOO（Xu等人，[2023b](#bib.bib41)），和5) PASS（我们的框架，参见第[3](#S3 "3 PASS代理架构 ‣ 正式定义基于LLM的代理的高层行为")节）。此外，按照Yao等人（[2022](#bib.bib44)）的方法，我们测试了基于代理和非代理方法的互补性。为此，首先由非代理方法（即，直接/CoT）尝试回答问题。然后，如果在$k=5$次自一致性尝试后没有重复的答案，就使用基于代理的方法生成答案。在表格中，这些结果将表示为$\texttt{[Non-Agent]}\rightarrow\texttt{[Agent]}$。

我们在三个标准数据集上评估了这些方法：1) GSM8K Cobbe 等人 ([2021](#bib.bib10))，一个数学推理数据集，测试系统解决小学数学应用题的能力，2) HotpotQA Yang 等人 ([2018](#bib.bib43))，一个多跳问答数据集，需要对来自 Wikipedia 的段落进行推理，3) TriviaQA Joshi 等人 ([2017](#bib.bib16))，一个具有组合性问题的具有挑战性的问答数据集。

在可能的情况下，我们遵循先前基于智能体的工作方法，采用相同的数据集和超参数（见附录 [A.2](#A1.SS2 "A.2 Hyperparameters and Hardware ‣ Appendix A Appendix ‣ Formally Specifying the High-Level Behavior of LLM-Based Agents)")，提示语来自 Xu 等人 ([2023b](#bib.bib41))。像 Yao 等人 ([2022](#bib.bib44)) 和 Xu 等人 ([2023b](#bib.bib41)) 一样，我们没有使用 HotpotQA 或 TriviaQA 的注释 Wikipedia 段落，而是让智能体选择在 Wikipedia 中搜索哪些术语。为了减少运行大型模型进行大量实验的成本，我们遵循了 Xu 等人 ([2023b](#bib.bib41))；Shinn 等人 ([2023](#bib.bib30))；Liu 等人 ([2023](#bib.bib17))；Yao 等人 ([2022](#bib.bib44)) 的做法，针对更大的数据集，从他们的开发集随机选择问题子集进行评估。对于 HotpotQA 和 TriviaQA，我们从开发集随机选择 1000 个问题进行评估，而对于 GSM8K，我们保留完整的测试集进行评估。

在所有实验中，我们使用 Llama-2-70b Touvron 等人 ([2023](#bib.bib32)) 作为支持我们智能体的 LLM。所有系统都可以使用相同的工具：1) 用于 GSM8K 的计算器，执行输入公式并返回一个数字，2) 用于 HotpotQA 和 TriviaQA 的搜索工具，返回实体对应的 Wikipedia 页面中的第一句话（如果存在），否则返回最相似的具有 Wikipedia 页面实体，3) 用于 HotpotQA 和 TriviaQA 的查找工具，返回在上次搜索的页面中包含输入字符串的下一句。

## 5 结果

| 方法 | HotpotQA | GSM8K | TriviaQA |
| --- | --- | --- | --- |
| Direct | 29.3 | 15.8 | 74.4 |
| CoT ([魏等](#bib.bib35)) | 35.0 | 64.6 | 69.4 |
| ReACT ([姚等](#bib.bib44)) | 28.7 | 53.8 | 44.1 |
| ReWOO ([Xu 等](#bib.bib41)) | 27.0 | 51.9 | 52.4 |
| PASS (我们的) | 34.7 | 53.2 | 57.8 |
| Direct $\rightarrow$ ReACT | 33.1 | 33.3 | 75.3 |
| Direct $\rightarrow$ ReWOO | 31.3 | 32.6 | 75.4 |
| Direct $\rightarrow$ PASS | 33.3 | 33.1 | 75.9 |
| CoT $\rightarrow$ ReACT | 37.8 | 65.3 | 71.4 |
| CoT $\rightarrow$ ReWOO | 37.0 | 65.6 | 71.6 |
| CoT $\rightarrow$ PASS | 38.5 | 64.7 | 72.6 |

表 1：不同提示类型在数据集上的精确匹配准确率（%）结果。所有三组（基于智能体的、非基于智能体的和混合型）的最佳结果均为**粗体**。

在表格 [1](#S5.T1 "Table 1 ‣ 5 Results ‣ Formally Specifying the High-Level Behavior of LLM-Based Agents") 中，我们提供了所有三组数据集的结果。我们再次强调，所有代理都在我们的框架中实现并进行评估。尽管由于 Yao 等人（[2022](#bib.bib44)）；Xu 等人（[2023b](#bib.bib41)）；Shinn 等人（[2023](#bib.bib30)）；Liu 等人（[2023](#bib.bib17)）在不同的数据集子集上进行评估，并且使用不同的底层模型（例如，GPT-3.5 Brown 等人（[2020](#bib.bib8)），PaLM Chowdhery 等人（[2022](#bib.bib9)））使得与之前工作的完美对比非常困难，我们的结果在趋势上与 Xu 等人（[2023b](#bib.bib41)）；Yao 等人（[2022](#bib.bib44)）的研究发现一致。在 Xu 等人（[2023b](#bib.bib41)）的研究中，ReWOO 在 TriviaQA 上的表现优于 ReACT（他们的研究中为 51.8% 对 47.4%），而在 HotpotQA 上表现较差（为 30.4% 对 32.2%）。最后，像 Yao 等人（[2022](#bib.bib44)）和 Xu 等人（[2023b](#bib.bib41)）一样，我们观察到链式思维 Wei 等人（[2022](#bib.bib35)）和直接响应比基于代理的方法更为有效。

除了个别方法，表格 [1](#S5.T1 "Table 1 ‣ 5 Results ‣ Formally Specifying the High-Level Behavior of LLM-Based Agents") 还显示了代理方法与非代理方法的互补效果。如表中所示，混合系统（即使用代理作为备用预测的非代理系统）在所有数据集上都取得了最佳表现，其中由非代理方法与我们的 PASS 代理结合产生的最佳结果。这表明这两种不同响应范式之间存在互补性。

<svg class="ltx_picture ltx_centering" height="208.82" id="S5.F8.pic1" overflow="visible" version="1.1" width="582"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,208.82) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="181.26" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="538.69">[Question] Who was born first, Yanka Dyagileva or Alexander Bashlachev? [Thought] I need to search Yanka Dyagileva and Alexander Bashlachev, find their birth years, then find which was born first. [Action] Search [Action Input] Yanka Dyagileva [Action] Search [Action Input] Alexander Bashlachev [Summary] Yanka Dyagileva was born in 1966 while Alexander Bashlachev was born in 1960. [Final Thought] 1960 (Alexander Bashlachev) < 1966 (Yanka Dyagileva), so Alexander Bashlachev was born first. [Answer] Alexander Bashlachev</foreignobject></g></g></svg>

图 8：PASS 代理对 HotpotQA 数据集问题的文本输出示例

| 代理 | HotpotQA | GSM8K | TriviaQA |
| --- | --- | --- | --- |
| ReACT （[Yao 等人](#bib.bib44)） | 28.7 | 53.8 | 44.1 |
| ReWOO （[Xu 等人](#bib.bib41)） | 27.0 | 51.9 | 52.4 |
| PASS | 34.7 | 53.2 | 57.8 |
|   - 无总结器 | 25.7 | 53.8 | 45.2 |

表 2：PASS 架构在有/没有总结步骤情况下的精确匹配准确率（%）消融实验结果

在基于代理的系统中，我们看到我们的 PASS 代理在 HotpotQA 和 TriviaQA 上表现最佳，而在与 ReACT 比较时，在 GSM8K 上表现较差。我们在图 [8](#S5.F8 "Figure 8 ‣ 5 Results ‣ Formally Specifying the High-Level Behavior of LLM-Based Agents") 中提供了 PASS 输出的一个示例，来自 HotpotQA 数据集。我们怀疑总结步骤在 PASS 性能中发挥了关键作用。因此，我们进行了一次消融实验，移除了 PASS 代理中的总结模型。在这种设置下，所有动作的结果被转化为编号列表，并未做修改地返回给代理。

表 [2](#S5.T2 "表 2 ‣ 5 结果 ‣ 正式指定基于LLM代理的高层行为") 显示了消融实验的结果。在表格中，我们可以看到，对于 HotpotQA 和 TriviaQA，使用总结方法提高了性能。这可能是因为搜索和查找工具通常返回信息密度很高的结果（即原始的维基百科文本），其中包含了多余的信息。相比之下，对于 GSM8K，总结方法的 LLM 对结果产生了负面影响。在 GSM8K 中，唯一可用的工具是一个返回数字值的计算器。我们认为，由于没有太多可以总结的信息，LLM 无法提供足够的价值来抵消由于引入的幻觉所带来的错误。这些结果表明，总结方法可以是工具增强代理的一个非常有价值的补充，然而，它应当仅在工具调用的原始结果可能会导致代理偏离目标时才使用。

## 6 相关工作

### 6.1 基于LLM的代理

已提出了多种面向不同任务的基于LLM的代理。其中，WebAgent Gur 等人 ([2023](#bib.bib12)) 展示了能够通过遵循自然语言命令在网站上执行任务的语言驱动代理。与此相关，Generative Agents Park 等人 ([2023](#bib.bib24)) 的工作旨在模拟可信的人类行为，而 SayCan Ahn 等人 ([2022](#bib.bib1)) 则展示了在具身代理中使用 LLM 的能力。这些代理针对特定用途，因此与我们工作的相关性较低，而那些可以量身定制以解决更广泛任务的代理更为相关，例如，ReACT Yao 等人 ([2022](#bib.bib44))，Reflexion Shinn 等人 ([2023](#bib.bib30))，ReWOO Xu 等人 ([2023b](#bib.bib41))。

一些开源项目也专注于代理创建，例如，AutoGPT Gravitas ([2023](#bib.bib11)) 和 BabyAGI Nakajima ([2023](#bib.bib22))。这些工作主要集中在构建能够完成用户请求的自给自足的代理，这与我们的框架有所不同，因为在我们的框架中，用户为特定目的定义任务特定的个体代理。

最近有几个多代理协调框架被提出，例如，BOLAA Liu 等人 ([2023](#bib.bib17))，MetaGPT Hong 等人 ([2023](#bib.bib15))，Gentopia Xu 等人 ([2023a](#bib.bib40))，以及 AutoGen Wu 等人 ([2023](#bib.bib37))。这些框架主要关注协调多个简单的基于LLM的代理以实现复杂目标的相关问题，其中个体代理通常并不比 ReACT 或 ReWOO 更复杂。这与我们专注于设计、实现和验证复杂个体代理的目标不同（不过，我们认为我们的工作可能与这种方法相辅相成）。

与我们的方法概念上最为相似的是Harrison（[2022](#bib.bib13)）和Beurer-Kellner等人（[2023](#bib.bib6)）的研究。LangChain Harrison（[2022](#bib.bib13)）是一个流行的框架，用于设计基于LLM的应用程序，并且在实现LLM代理方面提供了一定的支持。然而，他们的方法并不能保证符合用户期望的行为，几乎完全依赖于提示来鼓励模型遵循预期行为。Beurer-Kellner等人（[2023](#bib.bib6)）的工作引入了一种基于提示的LLM脚本语言。他们的方法并不专注于基于LLM的代理，而是更广泛地考虑了将LLM输出插入到复杂、程序化定义的文本模板中的问题。与他们的方法的一个关键区别在于，他们采用了对约束的强制执行（例如，在用于实现ReACT代理时，他们的方法在每个单独的状态下停止解码）。虽然这使得他们能够捕获更广泛的约束和行为，但也导致了昂贵的重新提示（参见Xu等人（[2023b](#bib.bib41)））。

### 6.2 约束解码

与我们的工作概念相关的是约束生成方法，这些方法在推理时修改标准束搜索解码过程，以在输出中融入约束（Wiseman和Rush（[2016](#bib.bib36)）；Wuebker等人（[2016](#bib.bib39)）；Anderson等人（[2017a](#bib.bib2)）；Lu等人（[2022](#bib.bib18)）；Hokamp和Liu（[2017](#bib.bib14)）；Post和Vilar（[2018](#bib.bib26)））。这些研究的重点略低于我们的工作，因为它们直接进入解码器以修改其输出，但如果我们转而依赖本地托管模型，这些方法也可以用于在我们的方法中实现控制。

特别值得注意的是，Anderson等人（[2017b](#bib.bib3)）提出了一种约束束搜索算法，通过有限状态机跟踪约束，并在几个图像描述任务中展示了其优势。神经解码相关的工作Lu等人（[2021](#bib.bib19)，[2022](#bib.bib18)）通过在束搜索解码算法中添加约束违反的惩罚项，强制满足词汇约束（指定为任何包含词汇包含和排除约束的谓词逻辑公式）。Bastan等人（[2023](#bib.bib5)）在神经解码的基础上，结合了捕捉依赖解析信息的结构性约束。最后，FUDGE Yang和Klein（[2021](#bib.bib42)）以及NADO Meng等人（[2022](#bib.bib20)）的研究提出了使用经过训练的辅助模型来识别满足约束的输出，从而控制基本模型（例如预训练的LLM）进行生成。

## 7 结论

在这项工作中，我们介绍了一种用于定义基于LLM的智能体的高级声明性框架。我们使用该框架实现了多种著名的智能体类型，并进一步引入了PASS，一种新型的智能体架构，利用了我们框架的声明性特征。最后，我们将其性能与其他智能体在三个标准数据集上的表现进行了比较，发现其在性能上表现强劲，并且具有与非智能体基于提示的方法互补的能力。

## 参考文献

+   Ahn 等人（2022）Michael Ahn、Anthony Brohan、Noah Brown、Yevgen Chebotar、Omar Cortes、Byron David、Chelsea Finn、Chuyuan Fu、Keerthana Gopalakrishnan、Karol Hausman 等人。2022年。做我能做的，而不是我说的：将语言与机器人能力结合。*arXiv 预印本 arXiv:2204.01691*。

+   Anderson 等人（2017a）Peter Anderson、Basura Fernando、Mark Johnson 和 Stephen Gould。2017年。*[引导开放词汇图像字幕生成与约束束搜索](https://doi.org/10.18653/v1/D17-1098)*。在*2017年自然语言处理经验方法会议论文集*，页面936–945，丹麦哥本哈根。计算语言学会。

+   Anderson 等人（2017b）Peter Anderson、Basura Fernando、Mark Johnson 和 Stephen Gould。2017年。引导开放词汇图像字幕生成与约束束搜索。在*2017年自然语言处理经验方法会议论文集*，页面936–945。

+   Baier 和 Katoen（2008）Christel Baier 和 Joost-Pieter Katoen。2008年。*模型检测原理*。麻省理工学院出版社。

+   Bastan 等人（2023）Mohaddeseh Bastan、Mihai Surdeanu 和 Niranjan Balasubramanian。2023年。神经结构解码：具有结构约束的神经文本生成。在*第61届计算语言学会年会论文集（第一卷：长篇论文）*，页面9496–9510。

+   Beurer-Kellner 等人（2023）Luca Beurer-Kellner、Marc Fischer 和 Martin Vechev。2023年。提示即编程：一种用于大型语言模型的查询语言。*ACM编程语言会议论文集*，7（PLDI）：1946–1969。

+   Brohan 等人（2023）Anthony Brohan、Yevgen Chebotar、Chelsea Finn、Karol Hausman、Alexander Herzog、Daniel Ho、Julian Ibarz、Alex Irpan、Eric Jang、Ryan Julian 等人。2023年。做我能做的，而不是我说的：将语言与机器人能力结合。在*机器人学习会议*，页面287–318。PMLR。

+   Brown 等人（2020）Tom Brown、Benjamin Mann、Nick Ryder、Melanie Subbiah、Jared D Kaplan、Prafulla Dhariwal、Arvind Neelakantan、Pranav Shyam、Girish Sastry、Amanda Askell 等人。2020年。语言模型是少量学习者。*神经信息处理系统进展*，33:1877–1901。

+   Chowdhery 等人（2022）Aakanksha Chowdhery、Sharan Narang、Jacob Devlin、Maarten Bosma、Gaurav Mishra、Adam Roberts、Paul Barham、Hyung Won Chung、Charles Sutton、Sebastian Gehrmann 等人。2022年。Palm：通过路径扩展语言建模。*arXiv 预印本 arXiv:2204.02311*。

+   Cobbe 等人 (2021) Karl Cobbe, Vineet Kosaraju, Mohammad Bavarian, Mark Chen, Heewoo Jun, Lukasz Kaiser, Matthias Plappert, Jerry Tworek, Jacob Hilton, Reiichiro Nakano, 等人. 2021. 训练验证器解决数学文字题。*arXiv 预印本 arXiv:2110.14168*。

+   Gravitas (2023) Significant Gravitas. 2023. Autogpt. [https://agpt.co](https://agpt.co)。

+   Gur 等人 (2023) Izzeddin Gur, Hiroki Furuta, Austin Huang, Mustafa Safdari, Yutaka Matsuo, Douglas Eck, 和 Aleksandra Faust. 2023. 具有规划、长文本理解和程序合成的现实世界 WebAgent。*arXiv 预印本 arXiv:2307.12856*。

+   Harrison (2022) Chase Harrison. 2022. Langchain. [https://github.com/langchain-ai/langchain](https://github.com/langchain-ai/langchain)。

+   Hokamp 和 Liu (2017) Chris Hokamp 和 Qun Liu. 2017. 使用网格束搜索进行序列生成的词汇约束解码。发表于 *第55届计算语言学学会年会（第1卷：长篇论文集）*，页面1535–1546。

+   Hong 等人 (2023) Sirui Hong, Xiawu Zheng, Jonathan Chen, Yuheng Cheng, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, Chenyu Ran, 等人. 2023. Metagpt: 面向多代理协作框架的元编程。*arXiv 预印本 arXiv:2308.00352*。

+   Joshi 等人 (2017) Mandar Joshi, Eunsol Choi, Daniel Weld, 和 Luke Zettlemoyer. 2017. [TriviaQA：一个大规模远程监督挑战数据集用于阅读理解](https://doi.org/10.18653/v1/P17-1147)。发表于 *第55届计算语言学学会年会（第1卷：长篇论文集）*，页面1601–1611，加拿大温哥华。计算语言学协会。

+   Liu 等人 (2023) Zhiwei Liu, Weiran Yao, Jianguo Zhang, Le Xue, Shelby Heinecke, Rithesh Murthy, Yihao Feng, Zeyuan Chen, Juan Carlos Niebles, Devansh Arpit, 等人. 2023. Bolaa: 基准测试与协调LLM增强的自主代理。*arXiv 预印本 arXiv:2308.05960*。

+   Lu 等人 (2022) Ximing Lu, Sean Welleck, Peter West, Liwei Jiang, Jungo Kasai, Daniel Khashabi, Ronan Le Bras, Lianhui Qin, Youngjae Yu, Rowan Zellers, 等人. 2022. 神经学 A* 样式解码：具有前瞻启发式的受限文本生成。发表于 *2022年北美计算语言学学会年会：人类语言技术会议论文集*，页面780–799。

+   Lu 等人 (2021) Ximing Lu, Peter West, Rowan Zellers, Ronan Le Bras, Chandra Bhagavatula, 和 Yejin Choi. 2021. 神经解码：（无）监督神经文本生成与谓词逻辑约束。发表于 *2021年北美计算语言学学会年会：人类语言技术会议论文集*，页面4288–4299。

+   Meng 等人 (2022) Tao Meng, Sidi Lu, Nanyun Peng, 和 Kai-Wei Chang. 2022. 使用神经分解的预言机进行可控文本生成。*神经信息处理系统进展*，35:28125–28139。

+   Murray 和 Chiang (2018) Kenton Murray 和 David Chiang. 2018. [纠正神经机器翻译中的长度偏差](https://doi.org/10.18653/v1/W18-6322)。在 *第三届机器翻译会议：研究论文集*，第212–223页，比利时布鲁塞尔。计算语言学协会。

+   Nakajima (2023) Y Nakajima. 2023. 基于任务驱动的自主代理，利用GPT-4、Pinecone和Langchain进行多种应用。*参见 https://yoheinakajima.com/task-driven-autonomous-agent-utilizing-gpt-4-pinecone-and-langchain-for-diverse-applications（访问日期：2023年4月18日）*。

+   Ouyang 等 (2022) Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray 等. 2022. 训练语言模型通过人类反馈遵循指令。*神经信息处理系统进展*，35:27730–27744。

+   Park 等 (2023) Joon Sung Park, Joseph C O’Brien, Carrie J Cai, Meredith Ringel Morris, Percy Liang, 和 Michael S Bernstein. 2023. 生成代理：人类行为的互动仿真。*arXiv预印本 arXiv:2304.03442*。

+   Pnueli (1977) Amir Pnueli. 1977. 程序的时序逻辑。在 *第18届计算机科学基础年会（sfcs 1977）*，第46–57页。IEEE。

+   Post 和 Vilar (2018) Matt Post 和 David Vilar. 2018. 通过动态束分配实现快速的词汇约束解码，用于神经机器翻译。在 *2018年北美计算语言学协会年会：人类语言技术，第1卷（长篇论文）*，第1314–1324页。

+   Qin 等 (2023) Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian 等. 2023. Toolllm：帮助大型语言模型掌握16000+个现实世界API。*arXiv预印本 arXiv:2307.16789*。

+   Schick 等 (2023) Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Luke Zettlemoyer, Nicola Cancedda, 和 Thomas Scialom. 2023. Toolformer：语言模型能够自我学习使用工具。*arXiv预印本 arXiv:2302.04761*。

+   Shen 等 (2023) Yongliang Shen, Kaitao Song, Xu Tan, Dongsheng Li, Weiming Lu, 和 Yueting Zhuang. 2023. Hugginggpt：通过ChatGPT及其在Huggingface中的伙伴解决AI任务。*arXiv预印本 arXiv:2303.17580*。

+   Shinn 等 (2023) Noah Shinn, Beck Labash, 和 Ashwin Gopinath. 2023. Reflexion：一个具有动态记忆和自我反思的自主代理。*arXiv预印本 arXiv:2303.11366*。

+   Tang 等 (2023) Qiaoyu Tang, Ziliang Deng, Hongyu Lin, Xianpei Han, Qiao Liang, 和 Le Sun. 2023. Toolalpaca：面向语言模型的通用工具学习，基于3000个模拟案例。*arXiv预印本 arXiv:2306.05301*。

+   Touvron 等人 (2023) Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, 等人. 2023. Llama 2: 开放基础和微调的聊天模型。*arXiv 预印本 arXiv:2307.09288*。

+   Wang 等人 (2022) Xuezhi Wang, Jason Wei, Dale Schuurmans, Quoc V Le, Ed H Chi, Sharan Narang, Aakanksha Chowdhery, 和 Denny Zhou. 2022. 自一致性提升语言模型中的连锁思维推理能力。收录于 *第十一届国际学习表示会议*。

+   Wei 等人 (2021) Jason Wei, Maarten Bosma, Vincent Zhao, Kelvin Guu, Adams Wei Yu, Brian Lester, Nan Du, Andrew M Dai, 和 Quoc V Le. 2021. 微调语言模型是零样本学习者。收录于 *国际学习表示会议*。

+   Wei 等人 (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, 等人. 2022. 连锁思维提示法激发大型语言模型中的推理能力。*神经信息处理系统进展*，35:24824–24837。

+   Wiseman 和 Rush (2016) Sam Wiseman 和 Alexander M. Rush. 2016. [序列到序列学习作为束搜索优化](https://doi.org/10.18653/v1/D16-1137)。收录于 *2016年自然语言处理经验方法会议*，第1296–1306页，美国德克萨斯州奥斯汀市。计算语言学协会。

+   Wu 等人 (2023) Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang, 和 Chi Wang. 2023. Autogen: 通过多代理对话框架推动下一代大型语言模型应用。*arXiv 预印本 arXiv:2308.08155*。

+   Wu 等人 (2016) Yonghui Wu, Mike Schuster, Zhifeng Chen, Quoc V Le, Mohammad Norouzi, Wolfgang Macherey, Maxim Krikun, Yuan Cao, Qin Gao, Klaus Macherey, 等人. 2016. Google的神经机器翻译系统：弥合人类与机器翻译之间的差距。*arXiv 预印本 arXiv:1609.08144*。

+   Wuebker 等人 (2016) Joern Wuebker, Spence Green, John DeNero, Saša Hasan, 和 Minh-Thang Luong. 2016. [用于前缀约束机器翻译的模型与推理](https://doi.org/10.18653/v1/P16-1007)。收录于 *计算语言学协会第54届年会（第1卷：长篇论文）*，第66–75页，德国柏林。计算语言学协会。

+   Xu 等人 (2023a) Binfeng Xu, Xukun Liu, Hua Shen, Zeyu Han, Yuhan Li, Murong Yue, Zhiyuan Peng, Yuchen Liu, Ziyu Yao, 和 Dongkuan Xu. 2023a. [Gentopia.AI: 用于工具增强的大型语言模型的协作平台](https://doi.org/10.18653/v1/2023.emnlp-demo.20)。收录于 *2023年自然语言处理经验方法会议：系统演示*，第237–245页，新加坡。计算语言学协会。

+   Xu 等人（2023b）Binfeng Xu、Zhiyuan Peng、Bowen Lei、Subhabrata Mukherjee、Yuchen Liu 和 Dongkuan Xu. 2023b. Rewoo: 为高效增强型语言模型解耦推理与观察. *arXiv 预印本 arXiv:2305.18323*。

+   Yang 和 Klein（2021）Kevin Yang 和 Dan Klein. 2021. Fudge: 通过未来判别器进行受控文本生成. 载于 *2021年北美计算语言学会会议：人类语言技术论文集*，第3511–3535页。

+   Yang 等人（2018）Zhilin Yang、Peng Qi、Saizheng Zhang、Yoshua Bengio、William Cohen、Ruslan Salakhutdinov 和 Christopher D Manning. 2018. Hotpotqa: 一个用于多跳问答的多样化、可解释的数据集. 载于 *2018年自然语言处理实证方法会议论文集*，第2369–2380页。

+   Yao 等人（2022）Shunyu Yao、Jeffrey Zhao、Dian Yu、Nan Du、Izhak Shafran、Karthik R Narasimhan 和 Yuan Cao. 2022. React: 在语言模型中协同推理与行动. 载于 *第十一届国际学习表示会议*。

## 附录 A 附录

### A.1 线性时序逻辑

在这里，我们提供了线性时序逻辑（LTL）的简要概述，该逻辑用于我们的代理规范框架。LTL 是一种模态时序逻辑，最初由Pnueli（[1977](#bib.bib25)）提出，用于形式化验证，扩展了命题逻辑，加入了时序操作符 $\mathbin{\bigcirc\kern 1.00006pt}$（下一个）和 $\mathbin{\mathcal{U}\kern-1.00006pt}$（直到）。这两个操作符具有直观的定义，$\mathbin{\bigcirc\kern 1.00006pt}$（下一个）是一个单目操作符，非正式地意味着公式 $\varphi$ 必须在下一个时间步中成立，$\mathbin{\mathcal{U}\kern-1.00006pt}$（直到）是一个二目操作符，指定公式 $\varphi_{i}$ 必须为真，直到公式 $\varphi_{j}$ 成立为止。LTL 公式定义在原子命题集合 $\mathcal{P}$ 上，其语法为：

|  | $\varphi::=\mathrm{true}\ \ \big{&#124;}\ \ p\ \ \big{&#124;}\ \ \neg\varphi\ \ \big{&#124;}\ % \ \varphi_{1}\wedge\varphi_{2}\ \ \big{&#124;}\ \mathbin{\bigcirc\kern 1.00006pt}% \varphi\ \ \big{&#124;}\ \ \varphi_{1}\mathbin{\mathcal{U}\kern-1.00006pt}\varphi_{2}$ |  |
| --- | --- | --- |

其中 $p\in\mathcal{P}$。LTL 公式是在无限观察序列上评估的，其中每个观察都是对 $\mathcal{P}$ 中符号的真值赋值。设 $\varphi$ 为 LTL 公式，$\sigma$ 为观察序列 $\sigma=\left<\sigma_{1},\sigma_{2},\ldots\right>$，其中每个 $\sigma_{i}$ 可以看作是时间 $i$ 时刻在 $\mathcal{P}$ 中为真的子集，则当 $\sigma\models\varphi$（满足）时：

|  | $\displaystyle\sigma\quad$ | $\displaystyle\models\quad\mathrm{true}\quad$ |  |
| --- | --- | --- | --- |
|  | $\displaystyle\sigma\quad$ | $\displaystyle\models\quad p\quad$ | $\displaystyle\textrm{iff}\quad p\in\sigma_{0}$ |  |
|  | $\displaystyle\sigma\quad$ | $\displaystyle\models\quad\neg\varphi\quad$ | $\displaystyle\textrm{当且仅当}\quad\sigma\not\models\varphi\quad\quad$ |  |
|  | $\displaystyle\sigma\quad$ | $\displaystyle\models\quad\varphi_{1}\wedge\varphi_{2}\quad$ | $\displaystyle\textrm{当且仅当}\quad\sigma\models\varphi_{1}\ \ \textrm{且}\ \ \sigma\models\varphi_{2}$ |  |
|  | $\displaystyle\sigma\quad$ | $\displaystyle\models\quad\mathbin{\bigcirc\kern 1.00006pt}\varphi\quad$ | $\displaystyle\textrm{当且仅当}\quad\sigma[1\ldots]\models\varphi$ |  |
|  | $\displaystyle\sigma\quad$ | $\displaystyle\models\quad\varphi_{1}\mathbin{\mathcal{U}\kern-1.00006pt}% \varphi_{2}\quad$ | $\displaystyle\textrm{当且仅当}\quad\exists j\geq 0.\ \ \sigma[j\ldots]\models% \varphi_{2}$ |  |
|  | $\displaystyle\textrm{且}\ \ \sigma[i\ldots]\models\varphi_{1}$ |  |
|  | $\displaystyle\textrm{对于所有}\ \ 0\leq i<j$ |  |

| 其中$\sigma[i\ldots]=\left<\sigma_{i},\ldots\right>$是时间步骤$i$之后剩余的观察序列。通过上述运算符，我们可以定义额外的命题逻辑运算符$\vee$（析取），以及$\rightarrow$（蕴涵）和时态运算符$\mathbin{\lozenge\kern 1.00006pt}$（最终）和$\mathbin{\square\kern 1.00006pt}$（始终）。

|  | $\displaystyle\varphi_{1}\vee\varphi_{2}\quad$ | $\displaystyle\coloneqq\quad\neg(\neg\varphi_{1}\wedge\neg\varphi_{2})$ |  |
| --- | --- | --- | --- |
|  | $\displaystyle\varphi_{1}\rightarrow\varphi_{2}\quad$ | $\displaystyle\coloneqq\quad\neg\varphi_{1}\vee\varphi_{2}$ |  |
|  | $\displaystyle\mathbin{\lozenge\kern 1.00006pt}\varphi\quad$ | $\displaystyle\coloneqq\quad\mathrm{true}\mathbin{\mathcal{U}\kern-1.00006pt}\varphi$ |  |
|  | $\displaystyle\mathbin{\square\kern 1.00006pt}\varphi\quad$ | $\displaystyle\coloneqq\quad\neg\mathbin{\lozenge\kern 1.00006pt}\neg\varphi$ |  |

在本工作中，我们将“next”视为一个n元运算符，它可以被简单地看作是一个链式的“next”运算符序列，即$\mathbin{\bigcirc\kern 1.00006pt}(\varphi_{1},\varphi_{2},\varphi_{3},\ldots)$，并且具有直观的非正式解释：“$\varphi_{1}$然后是$\varphi_{2}$然后是$\varphi_{3}$”，等等。在图[9](#A1.F9 "图 9 ‣ A.1 线性时态逻辑 ‣ 附录A 附录 ‣ 正式指定基于LLM的代理的高级行为")中，我们提供了上述时态运算符随时间变化的真值赋值的图示。

在这项工作中，我们发现，仅允许公式包含原子命题$p$，以及运算符$\rightarrow$、$\mathbin{\bigcirc\kern 1.00006pt}$、$\mathbin{\square\kern 1.00006pt}$和$\mathbin{\mathcal{U}\kern-1.00006pt}$（即，我们不允许公式包含$\neg$、$\wedge$等）已经足以表示现有的代理架构范围。我们将扩展运算符集合（例如，包含$\mathbin{\lozenge\kern 1.00006pt}$、$\wedge$等）留待未来的工作中进行。有关LTL及其众多应用的更多细节，我们建议感兴趣的读者参考Baier和Katoen（[2008](#bib.bib4)）。

![参见标题](img/e2efdc5add23a5da1356b251995deb43.png)

图9：随着时间推移，使用各种LTL运算符的真值赋值示例

### A.2 超参数和硬件

除了实验部分列出的超参数之外，我们的方法没有太多超参数，因为我们的LLMs仅用于推理。除使用自一致性方法的 approaches（即Direct，CoT）外，我们的LLMs在所有情况下的解码策略都设置为贪婪解码。对于自一致性方法，采样温度设置为0.7（如Yao等人所述，[2022](#bib.bib44)），并且抽取了$k=5$个样本。

### A.3 代理定义

<svg class="ltx_picture ltx_centering" height="323.67" id="A1.F10.pic1" overflow="visible" version="1.1" width="582"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,323.67) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="296.11" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="538.69">(define react-agent (:states (Ques (:text "[问题]")) (Tht (:text "[思考]")) (Act (:text "[行动]")) (Act-Inp (:text "[行动输入]")) (Obs (:text "[观察]") (:flags :env-input)) (Final-Tht (:text "[最终思考]")) (Ans (:text "[答案]"))) (:behavior (next Ques (until (next Tht Act Act-Inp Obs)     Final-Tht)     Ans)))

图10：ReACT代理的规格 Yao等人（[2022](#bib.bib44)）

<svg class="ltx_picture ltx_centering" height="290.46" id="A1.F11.pic1" overflow="visible" version="1.1" width="582"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,290.46) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="262.9" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="538.69">(define rewoo-agent (:states (Ques (:text "[问题]")) (Plan (:text "[计划]")) (Act-Lbl (:text "[行动标签]")) (Act (:text "[行动]")) (Act-Inp (:text "[行动输入]")) (Solver (:text "[答案]") (:flags :env-input))) (:behavior (next Ques (until     (next Plan Act-Lbl Act Act-Inp)     Solver))))

图11：ReWOO代理的规格 Xu等人（[2023b](#bib.bib41)）

<svg class="ltx_picture ltx_centering" height="473.11" id="A1.F12.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,473.11) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="445.55" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">(define reflexion-agent (:states (Ques (:text "[问题]")) (Tht (:text "[思考]")) (Act (:text "[行动]")) (Act-Inp (:text "[行动输入]")) (Obs (:text "[观察]") (:flags :env-input)) (Final-Tht (:text "[最终思考]")) (Prop-Ans (:text "[拟议答案]")) (Eval (:text "[评估]") (:flags :env-input)) (Ref (:text "[反思]")) (Ans (:text "[答案]"))) (:behavior (next Ques (until (next (until (next Tht Act Act-Inp Obs) Final-Tht) Prop-Ans Eval      Ref)     Ans))))

图12：反思代理的规范 Shinn 等人（[2023](#bib.bib30)）

<svg class="ltx_picture ltx_centering" height="157.63" id="A1.F13.pic1" overflow="visible" version="1.1" width="582"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,157.63) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="130.07" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="538.69">(define cot-agent (:states (Ques (:text "[问题]")) (Tht (:text "[思考]")) (Ans (:text "[答案]"))) (:behavior   (next Ques Tht Ans)))

图13：链式思维代理的规范 Wei 等人（[2022](#bib.bib35)）

<svg class="ltx_picture ltx_centering" height="141.02" id="A1.F14.pic1" overflow="visible" version="1.1" width="582"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,141.02) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="113.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="538.69">(define direct-agent (:states (Ques (:text "[问题]")) (Ans (:text "[答案]"))) (:behavior   (next Ques Ans)))

图14：直接响应代理的规范（即仅输出答案）</foreignobject></g></g></svg></foreignobject></g></g></svg></foreignobject></g></g></svg></foreignobject></g></g></svg></foreignobject></g></g></svg></foreignobject></g></g></svg></foreignobject></g></g></svg></foreignobject></g></g></svg>
