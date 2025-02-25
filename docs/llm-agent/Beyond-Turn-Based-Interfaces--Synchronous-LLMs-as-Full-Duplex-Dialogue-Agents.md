<!--yml

类别：未分类

日期：2025-01-11 12:13:38

-->

# 超越基于轮换的界面：同步大型语言模型（LLMs）作为全双工对话代理

> 来源：[https://arxiv.org/html/2409.15594/](https://arxiv.org/html/2409.15594/)

Bandhav Veluri^(1,2)，Benjamin N Peloquin¹，Bokai Yu¹，

Hongyu Gong¹，Shyamnath Gollakota²

¹Meta AI，²华盛顿大学

{bandhav,gshyam}@cs.washington.edu  hygong@meta.com

###### 摘要

尽管在建模语音对话代理方面有广泛的兴趣，但大多数方法本质上是“半双工”的——受限于基于轮换的交互，需要用户明确提示或隐式跟踪中断或沉默事件才能做出回应。与此相反，人类对话是“全双工”的，允许通过快速和动态的轮流发言、重叠语音和回馈来实现丰富的同步性。从技术上讲，实现与大型语言模型（LLMs）全双工对话的挑战在于建模同步性，因为预训练的LLMs缺乏“时间”的概念。为了解决这一问题，我们提出了同步大型语言模型，用于全双工语音对话建模。我们设计了一种创新机制，将时间信息集成到Llama3-8b中，使其能够与现实世界时钟同步运行。我们还提出了一种训练方法，利用从文本对话数据生成的212k小时合成语音对话数据，结合仅有的2k小时真实世界语音对话数据，来创建生成有意义且自然的语音对话的模型。同步大型语言模型在对话的意义性方面超过了现有的最先进技术，同时保持自然性。最后，我们通过模拟两个在不同数据集上训练的代理之间的互动，并考虑到高达240毫秒的互联网延迟，展示了该模型在全双工对话中的能力。网页：[https://syncllm.cs.washington.edu/](https://syncllm.cs.washington.edu/)。

超越基于轮换的界面：

同步大型语言模型作为全双工对话代理

Bandhav Veluri^(1,2)，Benjamin N Peloquin¹，Bokai Yu¹，Hongyu Gong¹，Shyamnath Gollakota² ¹Meta AI，²华盛顿大学 {bandhav,gshyam}@cs.washington.edu  hygong@meta.com

## 1 引言

现有的口语对话模型主要是基于轮次的接口，本质上是半双工的Lakhotia等人（[2021](https://arxiv.org/html/2409.15594v1#bib.bib18)）；Zhang等人（[2023a](https://arxiv.org/html/2409.15594v1#bib.bib37)）；Hassid等人（[2024](https://arxiv.org/html/2409.15594v1#bib.bib11)）；Borsos等人（[2023](https://arxiv.org/html/2409.15594v1#bib.bib3)）。为了实现轮次的切换，这些系统依赖于用户的明确输入或用户发言结束时的停顿Zhang等人（[2023a](https://arxiv.org/html/2409.15594v1#bib.bib37)）。与此相比，人类口语对话并不依赖于沉默作为主要的轮次切换信号Levinson和Torreira（[2015a](https://arxiv.org/html/2409.15594v1#bib.bib19)）；Nguyen等人（[2022](https://arxiv.org/html/2409.15594v1#bib.bib25)）。研究表明，在人类对话中，发言者内部的停顿（在发言者的轮次内的停顿）通常比发言者之间的轮次间隔更长Heldner和Edlund（[2010](https://arxiv.org/html/2409.15594v1#bib.bib12)）；Brady（[1968](https://arxiv.org/html/2409.15594v1#bib.bib4)）；ten Bosch等人（[2005](https://arxiv.org/html/2409.15594v1#bib.bib34)）。英语使用者常常在没有等待停顿的情况下开始他们的轮次，利用语法、韵律和语用线索无缝地启动下一个轮次，同时最小化重叠和空隙Stivers等人（[2009](https://arxiv.org/html/2409.15594v1#bib.bib33)）。

人类口语对话本质上是全双工的，允许无缝的双向交流，在这种交流模式下，双方可以同时说话和倾听。这种互动模式使得即时反馈、为澄清而打断以及信息流的实时调整成为可能 Reece 等人（[2023](https://arxiv.org/html/2409.15594v1#bib.bib29)）；Levinson 和 Torreira（[2015b](https://arxiv.org/html/2409.15594v1#bib.bib20)）。与基于每轮完整发言处理文本或语音的半双工系统不同，人类对话中常常包含语音回响——如“嗯”或“对”这样的短小、重叠的短语——这是听者向说话者发出的信号，表示他们理解并且说话者可以继续。这样的同步动态使得互动顺畅，并创造出书面文本中所缺乏的节奏感 Heldner 和 Edlund（[2010](https://arxiv.org/html/2409.15594v1#bib.bib12)）。虽然人类从婴儿期开始就学习如何轮流发言的信号，以减少语音重叠和沉默时间 Nguyen 等人（[2021](https://arxiv.org/html/2409.15594v1#bib.bib27)），但语音重叠和长时间的沉默在口语对话中仍然很常见，因为它们丰富了对话，提供了额外的语用提示。例如，语音重叠和频繁的语音回响通常表示积极的倾听。同样，沉默的长度在不同文化中可能会有所不同，并且受到回应的迅速性的影响 Stivers 等人（[2009](https://arxiv.org/html/2409.15594v1#bib.bib33)）；Nguyen 等人（[2022](https://arxiv.org/html/2409.15594v1#bib.bib25)）。在这两种情况下，这些动态使得对话听起来更加“人性化”。

![参见标题](img/60251278d2962c181ab5c0addd46ae6a.png)

图1：SyncLLM作为一个全双工对话代理。在当前时间步（图中的块N），SyncLLM的上下文包含了LLM的语音直到当前块为止的交错块，以及与所有非当前块对应的用户语音。为了与用户同步，LLM必须在当前块结束之前生成它的下一个块（块N+1）。因此，SyncLLM首先生成一个*估计的用户块*，然后将其附加到上下文中，用于预测下一个块。

开发一个全双工语音对话代理面临四个挑战：1）理解和生成语音对话中的轮流发言提示需要模型与现实世界拥有共同的参考时钟。然而，当前的LLM并不具备这种“时间感”。2）与基于文本的聊天数据集相比，语音对话数据集较为有限。所有重要的语音对话数据集的组合（Cieri 等人，([2004](https://arxiv.org/html/2409.15594v1#bib.bib7))；Godfrey 等人，([1992](https://arxiv.org/html/2409.15594v1#bib.bib9))；Reece 等人，([2023](https://arxiv.org/html/2409.15594v1#bib.bib29))）加起来仍然只有大约3000小时的语音对话数据。3）全双工对话要求模型始终处于监听状态，并应随时准备发言，因为在任意时间点可能会发生反馈通道或重叠情况。这要求模型在整个对话过程中持续流式传输。4）由于语音对话代理可能在云基础设施上运行，因此必须解决互联网传输固有的基本延迟问题。因此，模型可能无法立即访问用户当前生成的令牌或语音，而必须以延迟输入的方式运行（见图[1](https://arxiv.org/html/2409.15594v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents")）。

在本文中，我们做出了多项贡献，以开发全双工对话代理：

+   •

    我们提出了同步LLMs（简称SyncLLM），用于全双工语音对话。SyncLLM通过将时间信息集成到LLM中，实现了同步建模，使其能够与现实世界时钟同步运行。我们生成周期性的同步令牌，为对话的双方提供共同的时间框架。然而，这要求我们解决由停顿造成的重复令牌问题，停顿可能发生在话语内部或话语之间。重复令牌可能会对语音对话模型的语义能力产生不利影响（Nguyen 等人，([2022](https://arxiv.org/html/2409.15594v1#bib.bib25))）。因此，我们训练我们的模型预测去重后的令牌序列，时间信息由我们的周期性同步令牌保持。

+   •

    人类语音交互依赖于在短期内模拟对方反应的能力。我们可以以200ms的最小间隔轮流发言，而语言生成的延迟大约是600ms（Levinson 和 Torreira，([2015b](https://arxiv.org/html/2409.15594v1#bib.bib20))）。这意味着我们能够预测对方接下来几句话，并做出适当回应。我们利用这一洞察力，预测两位说话者的语音单元，预测未来的语音片段，时间间隔为160-240ms。这确保了对于最大240ms的互联网延迟具有很强的抗干扰性。

+   •

    我们提出了一种三阶段的训练方案，利用从文本对话数据生成的合成语音对话，来缓解现实世界语音对话数据的稀缺性。具体来说，我们使用了212k小时的合成语音对话数据和仅2k小时的现实世界语音对话数据，开发出一种能够生成具有自然回合转换、重叠和反馈通道的有意义语音对话的模型。

+   •

    基于Meta的Llama3-8b实验设置（[2024](https://arxiv.org/html/2409.15594v1#bib.bib2)）和广泛的用户研究（n=32），我们展示了我们的方法在对话内容的意义性上，相较于最先进的全双工语音模型dGSLM Nguyen等人（[2022](https://arxiv.org/html/2409.15594v1#bib.bib25)），提高了+2.2分的平均意见分数（MOS），同时保持了回合转换的自然性。进一步的结果表明，我们的模型在Fisher训练集上进行微调（Cieri等人，[2004](https://arxiv.org/html/2409.15594v1#bib.bib7)）后，能够泛化到分布外的Candor测试集（Reece等人，[2023](https://arxiv.org/html/2409.15594v1#bib.bib29)），同时保持对话内容的意义性和自然性。

+   •

    最后，通过模拟两个微调后的Llama3-8b模型之间的全双工对话，我们展示了这种方法如何实现容忍延迟并支持流式全双工语音接口。进一步地，SyncLLM可以在用户方的对话由使用不同数据集训练的模型生成时，仍然进行连贯的对话。

## 2 相关工作

多模态语言模型。像 GPT-4 OpenAI ([2023](https://arxiv.org/html/2409.15594v1#bib.bib28))、Llama Touvron 等人 ([2023](https://arxiv.org/html/2409.15594v1#bib.bib35)) 和 Mistral Jiang 等人 ([2023](https://arxiv.org/html/2409.15594v1#bib.bib15)) 等文本语言模型的成功，激发了对多模态模型的探索。在这里，我们将讨论重点放在语音和文本模态上。已证明，从预训练的文本大语言模型（LLM）初始化可以促进多模态训练，Hassid 等人 ([2023](https://arxiv.org/html/2409.15594v1#bib.bib10)) 的研究表明这一点。最近的研究提出，通过离散语音标记扩展文本 LLM 的词汇，以使模型能够处理语音输入和输出，Rubenstein 等人 ([2023](https://arxiv.org/html/2409.15594v1#bib.bib30)) 提出了这一方法。模型通过对齐的语音-文本数据进行跨模态知识训练，包括自动语音识别（ASR）、文本到语音合成（TTS）、语音到文本（S2T）和语音到语音翻译（S2ST）等任务。这些任务的多任务学习已被 VioLA Wang 等人 ([2023](https://arxiv.org/html/2409.15594v1#bib.bib36))、AudioPaLM Rubenstein 等人 ([2023](https://arxiv.org/html/2409.15594v1#bib.bib30))、VoxtLM Maiti 等人 ([2023](https://arxiv.org/html/2409.15594v1#bib.bib22)) 和 SUTLM Chou 等人 ([2023](https://arxiv.org/html/2409.15594v1#bib.bib6)) 采用。SpiRit-LM Nguyen 等人 ([2024](https://arxiv.org/html/2409.15594v1#bib.bib26)) 交替使用语音和文本标记，通过下一个标记预测训练模型，展示了语音理解和生成能力。

语音对话模型。先前关于语音对话的研究涵盖了多个主题，例如对话状态跟踪Zhang等人（[2023b](https://arxiv.org/html/2409.15594v1#bib.bib38)）、轮次预测Skantze（[2021](https://arxiv.org/html/2409.15594v1#bib.bib32)）；Lin等人（[2022](https://arxiv.org/html/2409.15594v1#bib.bib21)）和响应生成Zhang等人（[2020](https://arxiv.org/html/2409.15594v1#bib.bib39)）。最近的研究将LLM应用于对话系统Zhao等人（[2020](https://arxiv.org/html/2409.15594v1#bib.bib40)）。从Llama初始化的SpeechGPT Zhang等人（[2023a](https://arxiv.org/html/2409.15594v1#bib.bib37)）经过语音数据和多模态指令集的顺序微调，执行语音问答（QA）任务。USDM Kim等人（[2024](https://arxiv.org/html/2409.15594v1#bib.bib17)）继续使用交错的语音-文本数据对Mistral进行预训练，以捕获多模态语义。对于对话微调，它使用语音和用户输入的转录文本作为指令数据构建模板。与使用语音令牌的模型不同，Spectron Nachmani等人（[2023](https://arxiv.org/html/2409.15594v1#bib.bib23)）直接操作频谱图来处理语音问答和语音续接等任务。然而，这些先前的研究仅限于轮次设置，在该设置中，对话模型被明确提示在其回合中发言。人类语音对话更加复杂，涉及隐性轮次提示和重叠的发言，如打断和回声反馈Schegloff（[2000](https://arxiv.org/html/2409.15594v1#bib.bib31)）。

与我们的工作最为接近的是dGSLM Lakhotia等人（[2021](https://arxiv.org/html/2409.15594v1#bib.bib18)）的研究，该研究使用双塔Transformer模型同时处理两个对话通道。与由自动语音识别（ASR）、文本LLM和文本到语音（TTS）组成的级联架构相比，它展示了更优越的性能。dGSLM的一个弱点是它仅依赖语音训练，未能充分利用文本知识。相比之下，我们的工作利用了语言模型的生成性智能，并赋予其多模态和同步能力。此外，在其实证研究中，dGSLM未考虑现实场景中的延迟，假设一个对话者的隐藏状态能立即被另一个对话者访问。而我们明确讨论了我们的模型如何处理语音对话中的延迟响应。

## 3 SyncLLM

SyncLLM 是一种自回归的变压器解码器架构，原生地以墙钟同步的方式建模离散语音单元。SyncLLM 被训练以预测交替的语音单元块，这些语音单元块对应对话双方，如图[1](https://arxiv.org/html/2409.15594v1#S1.F1 "图1 ‣ 1 引言 ‣ 超越基于轮次的接口：同步LLM作为全双工对话代理")所示。在每个时间步，模型预测与固定持续时间对应的语音单元，称为模型的*块大小*，首先是其对话方的语音单元，然后是用户对话方的语音单元。通过这种方式，模型能够生成与实际世界时钟同步的两个语音流。这使得我们的方法能够建模所有对话提示，例如后续通道、重叠、打断等。此外，由于我们使用与现有LLM相同的架构，我们的方法可以利用LLM的大规模预训练。

训练以预测交替标记序列块的模型可以用于全双工语音交互，如果我们能将其中一个标记流替换为对应于真实用户的流。在图[1](https://arxiv.org/html/2409.15594v1#S1.F1 "图1 ‣ 1 引言 ‣ 超越基于轮次的接口：同步LLM作为全双工对话代理")中，紫色框表示LLM在每个时间块中对话一方的标记序列，绿色框表示用户对话方的标记序列。我们通过丢弃LLM对用户回应的预测，并用用户的语音替换它，来实现LLM与用户的全双工语音交互。

![参见说明文字](img/badc4a3caa75ecd8b5985242c5f51dfd.png)

图2：SyncLLM的标记序列格式，块大小为160毫秒。（顶行）我们将口语对话表示为交替的HuBERT标记块，其中块大小决定了同步标记[S0]的频率。（中行）我们训练SyncLLM生成交替的去重HuBERT标记块，并带有周期性的同步标记。（底行）我们在每个块中插值去重标记，以获得原始格式的口语对话序列。

### 3.1 延迟容忍交互

在图 [1](https://arxiv.org/html/2409.15594v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents") 中，假设第 N 个时间段为当前时间步骤。我们可以将 LLM 输出的语音片段与用户的输入片段交替安排，直到第 N 个片段为止，而用户的输入片段仅对应 N-1 个片段。这里的推理是，直到第 N 个时间步骤结束，用户对第 N 个片段的输入才会可用。为了处理这种固有的延迟，类似于人类预测对话中另一方接下来会说什么的方式（Levinson 和 Torreira，[2015b](https://arxiv.org/html/2409.15594v1#bib.bib20)），LLM 对下一个片段（N+1）的输出是通过首先估计用户对第 N 个时间段的响应来计算的（图中以绿色框和虚线边框表示）。然后，我们将这个估计的片段附加到 LLM 的上下文中，以生成 LLM 的下一个片段（N+1）。对于生成后续片段（N+2, N+3, ……），我们丢弃第 N 个时间步骤的估计用户片段，并用用户的实际输入替换它，从而将后续交互与用户的实际输入进行对接。

### 3.2 令牌序列格式

参照先前的口语语言建模工作，Nguyen 等人（[2022](https://arxiv.org/html/2409.15594v1#bib.bib25)，[2024](https://arxiv.org/html/2409.15594v1#bib.bib26)）的方法，我们使用 HuBERT Hsu 等人（[2021](https://arxiv.org/html/2409.15594v1#bib.bib13)）来表示语音。我们使用 Nguyen 等人（[2024](https://arxiv.org/html/2409.15594v1#bib.bib26)）的令牌化参数，令牌采样率为 25 Hz —— 即每 40 毫秒一个令牌 —— 词汇量为 501。为了建模两位发言者 0 和 1 之间的对话，我们定义了两个特殊令牌 `[S0]` 和 `[S1]`，作为发言者标签，分别指定每个发言者的令牌序列的开始。我们将对话表示为两条并行的语音流，每条流代表一个发言者，如图 [2](https://arxiv.org/html/2409.15594v1#S3.F2 "Figure 2 ‣ 3 SyncLLM ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents") 的顶部行所示。对于每条流，我们嵌入一个周期性的发言者标签，其周期等于模型的片段大小。

![请参见标题](img/3180fa545c66d959ca7e2f164e5811b1.png)

图 3：表示一秒钟语音所需的令牌数量，包含/不包含去重。该直方图是基于 Fisher 数据集 Cieri 等人（[2004](https://arxiv.org/html/2409.15594v1#bib.bib7)）中的 15 小时对话数据计算得出的。

去重。HuBERT token 的固定时间周期对于建模全双工对话中的时间是有用的。然而，原始 HuBERT 序列包含了显著重复的 token，主要是由于语音中和话语之间的沉默造成的。每个唯一 token 的重复次数表示该 token 所代表的声学单元的持续时间。然而，语义内容可以通过仅考虑唯一 token 并去重 token 序列来建模，Kharitonov 等人（[2022](https://arxiv.org/html/2409.15594v1#bib.bib16)）；Nguyen 等人（[2022](https://arxiv.org/html/2409.15594v1#bib.bib25)）。重复的 token 序列可能会对最终的语音对话模型的语义能力产生不利影响，Nguyen 等人（[2022](https://arxiv.org/html/2409.15594v1#bib.bib25)），因为如图 [3](https://arxiv.org/html/2409.15594v1#S3.F3 "Figure 3 ‣ 3.2 Token sequence format ‣ 3 SyncLLM ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents") 所示，它们每个 token 的语义内容比去重后的序列低约 $\sim 50\%$。

因此，SyncLLM 被训练为预测去重后的 HuBERT 序列，粗略的时间信息通过定期交错的特殊 token `[S0]` 和 `[S1]` 保持，如图 [2](https://arxiv.org/html/2409.15594v1#S3.F2 "Figure 2 ‣ 3 SyncLLM ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents") 第二行所示。在图 [2](https://arxiv.org/html/2409.15594v1#S3.F2 "Figure 2 ‣ 3 SyncLLM ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents") 示例的第一块中，两个说话者的流分别包含了 `[75]` 和 `[89]` 的 4 次重复。去重后，第一块对应的交错 token 序列为 `[S0][75][S1][89]`。在第二块中，说话者 0 有两个新 token（`[17]` 和 `[338]`），但说话者 1 的 token 只是前一块中最后一个 token `[89]` 的重复。因此，第二块的 token 序列将仅为 `[S0][17][338]`。请注意，当一块中没有新的 token 对应于说话者 1 时，我们也会排除说话者 1 的特殊 token `[S1]`。然而，说话者 0 的情况并非如此，因为我们需要在所有块中都包含一个说话者的特殊 token，以便明确区分各块。这一点在图 [2](https://arxiv.org/html/2409.15594v1#S3.F2 "Figure 2 ‣ 3 SyncLLM ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents") 的第三块中得到了体现。

插值。虽然去重后的标记序列对于自回归建模有利，但为了生成适合语音合成的标记序列，我们需要原始格式中的周期性HuBERT标记。由于说话人标签`[S0]`保持时间信息，我们知道去重后每个块中移除的标记数量。我们利用这一点进行插值，将去重的标记调整为每个块中预期的标记数量。例如，在图[2](https://arxiv.org/html/2409.15594v1#S3.F2 "图 2 ‣ 3 SyncLLM ‣ 超越基于回合的界面：同步LLM作为全双工对话代理")的第一个块中，去重后说话人0的流只有一个标记。但由于该块的大小为160ms，因此每个块将包含160/40 = 4个标记。所以如图[2](https://arxiv.org/html/2409.15594v1#S3.F2 "图 2 ‣ 3 SyncLLM ‣ 超越基于回合的界面：同步LLM作为全双工对话代理")第三行所示，我们将去重的标记重复三次以重建该块。如果一个块有多个去重标记，比如图[2](https://arxiv.org/html/2409.15594v1#S3.F2 "图 2 ‣ 3 SyncLLM ‣ 超越基于回合的界面：同步LLM作为全双工对话代理")中的第二个块，我们将每个标记按相等的次数重复。我们注意到这种方法可能会导致错误，因为原始块可能不遵循这种启发式规则。我们观察到即使块大小为240ms，这种错误的影响也是不可察觉的，这可能是因为每个标记的预测持续时间的误差被块大小上限限制。此外，在包含更多新标记的块中，误差会更小。

## 4 训练

我们使用Meta的Llama3-8b（[2024](https://arxiv.org/html/2409.15594v1#bib.bib2)）作为我们的基础模型，并采用三阶段训练程序，主要使用合成口语对话数据，以及相对较少的真实世界口语对话数据，以开发一个全双工语音代理。

表1：不同阶段训练中使用的数据。我们使用TTS将基于文本的数据转换为语音。

|  | 阶段 | 来源 | 语音 |
| --- | --- | --- | --- |
|  |  | 模态 | （小时） |
| 监督 | 1 | 文本 | 193k |
| 微调（SFT） |  |  |  |
| 对话 | 2 | 文本 | 20k |
| 口语对话 | 3 | 语音 | 1927 |

阶段 1：基于回合的口语对话模型，使用合成语音数据。由于口语对话数据有限，我们从大规模文本对话数据集中生成合成语音数据。我们使用监督微调（SFT）数据集作为我们的源文本对话数据集。我们使用Bark TTS AI（[2023](https://arxiv.org/html/2409.15594v1#bib.bib1)）模型生成文本对话数据集的语音版本，并使用其10个说话人预设。

由于Llama3-8b是一个仅处理文本的LLM，在第一阶段，我们旨在实现对话上下文中的文本与语音对齐。给定一个口语问题，我们训练模型生成一个口语回答。我们扩展了Llama3的词汇表，除了说话者标签`[S0]`和`[S1]`，还包含了501个HuBERT词汇。在训练中，一个回合制对话可以定义为由回合组成，而每个回合又由句子构成。我们用以下格式的对话序列对Llama3进行了微调：

```
[S1]<sent0>[S0]<sent0><sent1>[S1]..

```

![参考说明](img/0e5b2aa165479c2ab9cb14ff5b263001.png)

图4：我们从截断正态分布中抽取语音百分比，因此在整个训练过程中，我们获得了所有可能的文本与语音交替组合样本，并且随着训练的进行，语音百分比的偏向性会增大。这种方式在从文本-only LLM开始时，能够保证训练的稳定性。

每个句子在训练过程中随机选择成为文本或去重后的语音标记序列。对于每个训练样本，我们从截断正态分布中抽取训练序列中语音句子的百分比（见图[4](https://arxiv.org/html/2409.15594v1#S4.F4 "Figure 4 ‣ 4 Training ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents")）。仅使用全语音序列或逐步增加语音百分比会导致训练不稳定。句子级别的文本语音交替不仅训练了模型能够执行对话，还实现了对话中文本和语音的对齐。

阶段2：假设没有重叠的全双工对话。基于回合制的口语对话是全双工对话的一种特殊情况，且没有重叠。基于这一观察，我们可以将合成的口语对话数据视为全双工口语对话数据，其中在一个说话者的回合内，另一个说话者完全保持沉默。在这个阶段，我们从文本对话数据中创建合成口语对话数据，类似于上一阶段，主要的区别是：在每个对话回合中，我们生成一个对应于一个说话者的语音发声，并为另一个说话者生成等时长的沉默。然后，我们按照图[2](https://arxiv.org/html/2409.15594v1#S3.F2 "Figure 2 ‣ 3 SyncLLM ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents")第二行所示的格式对并行语音对话数据进行分词。这样，我们可以进一步利用文本对话数据帮助我们的模型学习图[2](https://arxiv.org/html/2409.15594v1#S3.F2 "Figure 2 ‣ 3 SyncLLM ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents")中的标记序列格式。在这一阶段，模型在发声中的时序微调。模型仍然无法学习诸如反馈信号或两个说话者之间的重叠等回合转换信号。

对于前一个阶段，SFT 数据集中的大多数样本包含一个说话者（LLM 用户）进行简短的发言，另一个说话者（LLM）则给出长的回应。然而，口语对话中则包含更多频繁的轮流发言，且发言较短。因此，在这个阶段，我们使用由较短发言组成的文本对话数据集，约为 $20$k 小时的合成口语对话。

表 2：基于相同的提示集，生成和真实续写之间的轮次事件持续时间的皮尔逊相关性比较。SyncLLM 的分块大小以括号表示。

| 模型 | Fisher（同分布） |  | Candor（异分布） |
| --- | --- | --- | --- |
|  | ipu | pause | fto | 平均值 |  | ipu | pause | fto | 平均值 |
| dGSLM | 0.48 | 0.41 | 0.10 | 0.33 |  | 0.30 | 0.02 | 0.09 | 0.14 |
| SyncLLM-F (160 ms) | 0.60 | 0.50 | 0.20 | 0.43 |  | 0.45 | 0.09 | 0.14 | 0.23 |
| SyncLLM-F (200 ms) | 0.60 | 0.49 | 0.19 | 0.43 |  | 0.44 | 0.28 | 0.14 | 0.29 |
| SyncLLM-F (240 ms) | 0.58 | 0.40 | 0.25 | 0.41 |  | 0.45 | 0.27 | 0.21 | 0.31 |
| 提示 | 0.72 | 0.53 | 0.31 | 0.52 |  | 0.54 | 0.30 | 0.12 | 0.32 |
| Resynth-GT | 0.92 | 0.92 | 0.53 | 0.79 |  | 0.90 | 0.86 | 0.37 | 0.71 |

第三阶段：使用真实世界的口语对话数据进行建模。最后，我们对模型进行微调，以便从真实世界的口语对话数据中学习轮流发言的提示。我们使用 Fisher Cieri 等人（[2004](https://arxiv.org/html/2409.15594v1#bib.bib7)）的数据集，该数据集包含 2000 小时的口语对话，每个对话中的每个说话者的语音被分离到独立的音频通道中。我们将数据集分为训练集、验证集和测试集，比例分别为 98:1:1。每个对话中的音频通道被单独标记，并交织在前一阶段使用的全双工对话格式中。在这一阶段，除了学习语句内的时序外，模型还学习了有效的轮流发言、对话提示，例如准确的暂停分配和回话信号。

## 5 个实验

![参见说明文字](img/cb73411b1df3b0c958f7683bd2d7b18d.png)

图 5：不同模型生成的口语对话转录的困惑度。困惑度是根据文本对话模型的预测进行测量的。

我们在续接和交互设置中评估了SyncLLM。在续接设置中，给定一个口语对话提示，模型生成对话的双方内容。在交互设置中，我们模拟了两个SyncLLM实例之间的交互，如§[3.1](https://arxiv.org/html/2409.15594v1#S3.SS1 "3.1 延迟容忍交互 ‣ 3 SyncLLM ‣ 超越基于回合的接口：同步LLM作为全双工对话代理")中所描述的那样。我们将基于Fisher训练的SyncLLM在续接设置中表示为SyncLLM-F，并使用dGSLM作为续接设置的基准。dGSLM和SyncLLM-F都使用Fisher作为唯一的真实口语对话数据集进行训练。我们将基于Fisher训练的SyncLLM与基于Fisher训练的实例交互表示为SyncLLM-F-F，将基于Fisher训练的SyncLLM与基于CANDOR Reece等人（[2023](https://arxiv.org/html/2409.15594v1#bib.bib29)）训练的实例交互表示为SyncLLM-F-C。

### 5.1 语义评估

我们通过使用自动语音识别（ASR）将口语生成转换为文本来评估SyncLLM在文本领域的语义。我们将生成的口语对话转录为基于回合的文本对话，忽略任何重叠的语音。然后，我们计算用10秒口语对话提示生成的转录对话的困惑度，并与纯文本对话模型进行比较。为了考虑异常值（困惑度异常高的样本），我们考虑测试集上的中位困惑度。

图[5](https://arxiv.org/html/2409.15594v1#S5.F5 "图5 ‣ 5 实验 ‣ 超越基于回合的接口：同步LLM作为全双工对话代理")比较了不同块大小的SyncLLM生成的口语对话的语义质量与先前的最先进全双工dGSLM模型Nguyen等人（[2022](https://arxiv.org/html/2409.15594v1#bib.bib25)）和真实对话续接的对比。我们发现，dGSLM相对于真实对话的困惑度下降约70，而SyncLLM则下降约15。图[6](https://arxiv.org/html/2409.15594v1#S5.F6 "图6 ‣ 5.1 语义评估 ‣ 5 实验 ‣ 超越基于回合的接口：同步LLM作为全双工对话代理")还比较了分别从Fisher和Candor测试集拆分中采样的提示所测得的中位困惑度，所有模型仅在Fisher训练集上训练。在这里，Candor测试集是一个分布外的测试集。

![参见说明文字](img/f4ce19f7e12384ec8ecf376a19c9ab52.png)

图6：分布内和分布外的测试。

这些评估表明，使用标准的自回归架构，并利用大量文本预训练，我们的方法生成了一个语义上更加一致的口语对话模型，相较于为仅语音训练提出的定制架构。此外，我们的三阶段训练方法通过利用大量从文本对话生成的合成口语对话数据，使我们能够在有限的真实世界双通道口语对话数据上更快地收敛。这使得我们得到的通用模型在跨领域（ood）表现上更为优秀。

### 5.2 自然性评估

停顿、说话者转换和重叠的适当时机是口语对话中不可或缺的部分，它们传达了自然口语对话所需的重要信息。为了评估我们生成的口语对话的这些方面，我们参考了Nguyen等人提出的轮流发言事件（[2022](https://arxiv.org/html/2409.15594v1#bib.bib25)），该事件评估了生成的口语对话的整体自然性：停顿间隔单元（IPUs）、停顿和话语转移偏移（FTO）。FTO是回合转换之间的持续时间，它是重叠和间隙的组合——负FTO表示重叠，正FTO表示间隙。

类似于dGSLM的设置，我们使用从测试集划分中采样的30秒提示，生成具有不同模型配置的90秒对话。然后，我们计算生成的对话与真实连续对话之间的回合转换事件持续时间的配对相关性，前提是使用相同的提示。我们首先使用`pyannote.audio`库计算每一侧对话（分别生成于不同音频通道）的语音活动 Bredin等人（[2020](https://arxiv.org/html/2409.15594v1#bib.bib5)）。然后，我们测量每个回合转换事件的开始和结束时间戳。接着，我们测量生成对话中回合转换事件的平均持续时间，然后计算不同模型生成的对话和真实对话中的平均持续时间之间的Pearson相关性。

表格。[2](https://arxiv.org/html/2409.15594v1#S4.T2 "Table 2 ‣ 4 Training ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents")比较了这一相关性与分布内Fisher Cieri等人（[2004](https://arxiv.org/html/2409.15594v1#bib.bib7)）测试集划分和分布外Candor测试集划分的相关性。我们观察到，使用我们的模型生成的对话，在分布内和分布外测试集上，相比于dGSLM，具有更好的回合转换事件与真实对话连续部分的相关性。除此之外，我们还提供了与提示和重新合成的真实对话连续部分（Resynth-GT）的回合转换事件相关性。Resynth-GT是通过重新合成标记化的真实对话连续部分获得的。由于标记化过程引入的时序变异，Resynth-GT与真实对话之间并不完全相关，因此它作为我们方法的顶部线进行比较。

### 5.3 人类评估

我们通过第三方供应商招募了32名注释员进行评估，要求他们具备母语水平的英语能力。

表3：意义性（Meaning.）和自然度（Nat.）（评分1-5）的均值估计和标准误差（括号内），分别针对整体、Fisher和CANDOR子集进行汇总。我们为本研究使用160ms的块大小。

|  | 总体 | Fisher | CANDOR |
| --- | --- | --- | --- |
| 模型 | 意义性. $\uparrow$ | 自然度. $\uparrow$ | 意义性. $\uparrow$ | 自然度. $\uparrow$ | 意义性. $\uparrow$ | 自然度. $\uparrow$ |
| --- | --- | --- | --- | --- | --- | --- |
| dGSLM | 1.55 (0.06) | 3.95 (0.08) | 1.67 (0.09) | 4.21 (0.08) | 1.43 (0.08) | 3.70 (0.12) |
| SyncLLM-C | 3.40 (0.07) | 3.96 (0.06) | 3.14 (0.10) | 3.97 (0.08) | 3.66 (0.08) | 3.94 (0.08) |
| SyncLLM-F | 3.74 (0.06) | 3.90 (0.06) | 3.82 (0.08) | 3.98 (0.08) | 3.67 (0.09) | 3.82 (0.10) |
| 重新合成 | 3.87 (0.06) | 4.03 (0.05) | 4.04 (0.08) | 4.14 (0.08) | 3.69 (0.07) | 3.91 (0.06) |
| GT | 4.96 (0.02) | 4.96 (0.02) | 4.96 (0.03) | 4.94 (0.04) | 4.97 (0.02) | 4.98 (0.02) |

我们采用ITU-T推荐标准P.808中的平均意见评分（MOS）协议（一个5点Likert量表）([2018](https://arxiv.org/html/2409.15594v1#bib.bib14))来评估轮次的自然度（N-MOS）和对话内容的意义性（M-MOS）。对于N-MOS和M-MOS，注释员将被提供提示音频和继续音频。注释员首先阅读N-MOS和M-MOS的描述，接着听提示音频，然后听继续音频。最后，他们被要求根据继续音频相对于提示音频中所包含的信息的质量来进行评分。每个分配给特定提示/继续对的注释员都会为N-MOS和M-MOS提供评分（参见§[B.1](https://arxiv.org/html/2409.15594v1#A2.SS1 "B.1 N-MOS & M-MOS ‣ 附录B 自然度-MOS说明 ‣ 超越基于轮次的接口：同步LLM作为全双工对话代理"))。

总共有$n_{annot}=32$名注释员对$n_{items}=180$个项目进行了评分，这些项目在CANDOR和Fisher数据集之间均匀分配。每个样本由三名不同的评分员给予从$1-差$到$5-优秀$的评分。我们通过计算每个项目的中位数评分来得出项目级别的评分。为了计算系统级别的评分，我们取一个给定系统的项目评分的平均值。我们通过自助法计算95%的置信区间，进行$n_{b}=1000$次迭代的项目级别重采样。

总体结果。表格[3](https://arxiv.org/html/2409.15594v1#S5.T3 "表格 3 ‣ 5.3 人类评估 ‣ 5 实验 ‣ 超越基于回合的接口：同步LLM作为全双工对话代理")的最左两列表明，几乎所有模型在回合制交互中的感知自然性（N-MOS）方面表现相当，同时接近重新合成的真实值。在对话内容的感知意义性（M-MOS）上，基于SyncLLM的模型显著优于dGSLM，接近重新合成的真实值。这里的Resynth-GT考虑了分词过程，是我们使用HuBERT分词器实现方法时的顶线数值。

内部分布和OOD。表格[3](https://arxiv.org/html/2409.15594v1#S5.T3 "表格 3 ‣ 5.3 人类评估 ‣ 5 实验 ‣ 超越基于回合的接口：同步LLM作为全双工对话代理")也突出了dGSLM和Fisher训练的SyncLLM-F之间在内部分布（Fisher）和OOD（CANDOR）上的差异。当dGSLM在OOD上遭遇显著降级（M-MOS和N-MOS评分分别下降-0.24和-0.51）时，SyncLLM-F在OOD上仅下降-0.15和-0.16。基于CANDOR数据集训练的SyncLLM（SyncLLM-C）在M-MOS上出现了下降（-0.52），但在N-MOS上没有下降（+0.03）。我们注意到，dGSLM Nguyen等人（[2022](https://arxiv.org/html/2409.15594v1#bib.bib25)）使用了在Fisher数据集上微调的语音表示，而我们的方法则使用了适用于所有语音领域的通用语音表示。这使得我们的方法在自然性方面优于基准，尤其是在OOD的Candor测试集上，正如表格[3](https://arxiv.org/html/2409.15594v1#S5.T3 "表格 3 ‣ 5.3 人类评估 ‣ 5 实验 ‣ 超越基于回合的接口：同步LLM作为全双工对话代理")中由人工评估员判断的那样。

![请参阅标题说明](img/f16a622677d90644394b929eb77f58cf.png)

图7：继续模式和交互模式之间的ASR困惑度比较。

### 5.4 全双工交互

我们通过模拟具有单块延迟的LLM-LLM交互来模拟LLM与用户的交互。我们评估了使用不同块大小训练的模型，从而模拟不同的延迟。我们还训练了一个版本的SyncLLM，该版本在第三训练阶段使用了Candor训练集，并模拟了它与仅使用Fisher训练的原始模型的交互。

在图[7](https://arxiv.org/html/2409.15594v1#S5.F7 "图7 ‣ 5.3 人类评估 ‣ 5 实验 ‣ 超越基于回合的接口：同步LLM作为全双工对话代理")中，我们比较了使用来自Fisher和Candor测试集的提示语所得到的中位数困惑度。同时，我们还展示了地面实况和在对话延续设置下生成的样本的困惑度，供参考。我们发现，在LLM-LLM交互设置中，SyncLLM能够接近延续设置的表现，并且在延续设置下的表现明显优于dGSLM。此外，我们发现，分别使用Fisher和Candor数据集训练的SyncLLM实例之间的交互几乎相同，表明即使用户一方的对话是由一个在不同数据集上训练的模型生成的，SyncLLM仍然能够进行连贯的对话。

人类评估。表[4](https://arxiv.org/html/2409.15594v1#S5.T4 "表4 ‣ 5.4 全双工交互 ‣ 5 实验 ‣ 超越基于回合的接口：同步LLM作为全双工对话代理")展示了dGSLM、Fisher训练的延续模型和LLM-LLM交互的评分。结果与§[5.4](https://arxiv.org/html/2409.15594v1#S5.SS4 "5.4 全双工交互 ‣ 5 实验 ‣ 超越基于回合的接口：同步LLM作为全双工对话代理")中的发现一致——LLM-LLM交互在M-MOS上优于dGSLM，但与单一模型延续设置相比稍逊。

表4：所有数据中的意义性（Meaning.）和自然性（Nat.）的平均估计值及标准误差（括号中的数值）的人工评估结果。

| 模型 | 意义性（Meaning.）$\uparrow$ | 自然性（Nat.）$\uparrow$ |
| --- | --- | --- |
| dGSLM | 1.55 (0.06) | 3.95 (0.08) |
| SyncLLM-F | 3.74 (0.06) | 3.90 (0.06) |
| SyncLLM-F-C | 3.39 (0.06) | 3.78 (0.06) |
| SyncLLM-F-F | 3.47 (0.06) | 3.72 (0.06) |

## 6 结论

我们展示了同步LLM，这是一种将自回归LLM转化为全双工语音对话代理的创新后训练框架。同步LLM在对话意义性上超越了当前最先进的技术，同时保持了回合交换的自然性。最后，通过模拟两个代理之间的全双工对话，我们展示了在互联网级延迟下的鲁棒性，在这种情况下，代理无法立即访问其用户生成的语音。

## 7 限制与风险

局限性。同步LLM的性能在语音质量方面还可以进一步提高。目前，我们使用简单的HiFi-GAN声码器进行语音合成，采用更先进的语音生成器可以通过语义单元合成更高质量的语音。此外，我们还未研究对话中的表现力和非语言声音（如笑声），这些都能使语音对话更具人类特征。另一个局限性是上下文长度；同步LLM从Llama-3初始化，因此具有相同的序列长度限制，这限制了对话中的长上下文建模以及像EnCodec Défossez 等（[2022](https://arxiv.org/html/2409.15594v1#bib.bib8)）那样的更具表现力的多词汇表标记器的使用，这些标记器有更高的标记速率。

伦理考虑。所提模型旨在用于语音对话代理。如果发生故障，系统可能会生成不恰当的回应，可能需要对语音输出进行毒性缓解。至于不当使用，其中一个例子是恶意行为者将模型用于在线诈骗。语音水印是应对滥用技术的一种潜在方法。

## 致谢

华盛顿大学的研究人员部分由Meta AI导师计划、Moore Inventor Fellow奖项#10617、UW CoMotion基金和NSF资助。

## 参考文献

+   AI（2023）Suno AI. 2023. Bark tts. [https://github.com/suno-ai/bark](https://github.com/suno-ai/bark)。

+   at Meta（2024）AT at Meta. 2024. Meta llama 3. [https://github.com/meta-llama/llama3](https://github.com/meta-llama/llama3)。

+   Borsos 等（2023）Zalán Borsos, Raphaël Marinier, Damien Vincent, Eugene Kharitonov, Olivier Pietquin, Matt Sharifi, Dominik Roblek, Olivier Teboul, David Grangier, Marco Tagliasacchi, 和 Neil Zeghidour. 2023. [Audiolm: 一种用于音频生成的语言建模方法](https://arxiv.org/abs/2209.03143). *预印本*，arXiv:2209.03143。

+   Brady（1968）Paul T. Brady. 1968. [A statistical analysis of on-off patterns in 16 conversations](https://api.semanticscholar.org/CorpusID:62739009). *贝尔系统技术期刊*，47:73–91。

+   Bredin 等（2020）Hervé Bredin, Ruiqing Yin, Juan Manuel Coria, Gregory Gelly, Pavel Korshunov, Marvin Lavechin, Diego Fustes, Hadrien Titeux, Wassim Bouaziz, 和 Marie-Philippe Gill. 2020. pyannote.audio: 用于说话人分离的神经构建模块。在 *ICASSP 2020，IEEE 国际声学、语音与信号处理会议*，西班牙巴塞罗那。

+   Chou 等（2023）Ju-Chieh Chou, Chung-Ming Chien, Wei-Ning Hsu, Karen Livescu, Arun Babu, Alexis Conneau, Alexei Baevski, 和 Michael Auli. 2023. [Toward joint language modeling for speech units and text](https://arxiv.org/abs/2310.08715). *预印本*，arXiv:2310.08715。

+   Cieri 等人（2004）Christopher Cieri, David Miller 和 Kevin Walker. 2004. [Fisher 语料库：为下一代语音转文本技术提供资源](https://api.semanticscholar.org/CorpusID:8414900). 收录于 *国际语言资源与评估会议*。

+   Défossez 等人（2022）Alexandre Défossez, Jade Copet, Gabriel Synnaeve 和 Yossi Adi. 2022. 高保真神经音频压缩. *arXiv 预印本 arXiv:2210.13438*。

+   Godfrey 等人（1992）J.J. Godfrey, E.C. Holliman 和 J. McDaniel. 1992. [Switchboard: 用于研究与开发的电话语音语料库](https://doi.org/10.1109/ICASSP.1992.225858). 收录于 *[会议录] ICASSP-92: 1992 IEEE 国际声学、语音与信号处理会议*, 第1卷，第517–520页。

+   Hassid 等人（2023）Michael Hassid, Tal Remez, Tu Anh Nguyen, Itai Gat, Alexis Conneau, Felix Kreuk, Jade Copet, Alexandre Défossez, Gabriel Synnaeve, Emmanuel Dupoux, Roy Schwartz 和 Yossi Adi. 2023. [文本预训练的语音语言模型](http://papers.nips.cc/paper_files/paper/2023/hash/c859b99b5d717c9035e79d43dfd69435-Abstract-Conference.html). 收录于 *神经信息处理系统进展 36：神经信息处理系统年会 2023, NeurIPS 2023, 美国路易斯安那州新奥尔良, 2023年12月10日至16日*。

+   Hassid 等人（2024）Michael Hassid, Tal Remez, Tu Anh Nguyen, Itai Gat, Alexis Conneau, Felix Kreuk, Jade Copet, Alexandre Defossez, Gabriel Synnaeve, Emmanuel Dupoux, Roy Schwartz 和 Yossi Adi. 2024. [文本预训练的语音语言模型](https://arxiv.org/abs/2305.13009). *预印本*, arXiv:2305.13009。

+   Heldner 和 Edlund（2010）Mattias Heldner 和 Jens Edlund. 2010. 对话中的停顿、间隙和重叠. *语音学杂志*, 38(4):555–568。

+   Hsu 等人（2021）Wei-Ning Hsu, Benjamin Bolte, Yao-Hung Hubert Tsai, Kushal Lakhotia, Ruslan Salakhutdinov 和 Abdelrahman Mohamed. 2021. [Hubert: 通过隐藏单元的掩蔽预测进行自监督语音表示学习](https://arxiv.org/abs/2106.07447). *预印本*, arXiv:2106.07447。

+   ITU-T 推荐 P.808（2018）ITU-T 推荐 P.808. 2018. 采用众包方法的语音质量主观评估。

+   Jiang 等人（2023）Albert Q. Jiang, Alexandre Sablayrolles, Arthur Mensch, Chris Bamford, Devendra Singh Chaplot, Diego de Las Casas, Florian Bressand, Gianna Lengyel, Guillaume Lample, Lucile Saulnier, Lélio Renard Lavaud, Marie-Anne Lachaux, Pierre Stock, Teven Le Scao, Thibaut Lavril, Thomas Wang, Timothée Lacroix 和 William El Sayed. 2023. Mistral 7b. *CoRR*, abs/2310.06825。

+   Kharitonov 等人（2022）Eugene Kharitonov, Ann Lee, Adam Polyak, Yossi Adi, Jade Copet, Kushal Lakhotia, Tu-Anh Nguyen, Morgane Rivière, Abdelrahman Mohamed, Emmanuel Dupoux 和 Wei-Ning Hsu. 2022. [无文本的韵律感知生成式语音语言建模](https://arxiv.org/abs/2109.03264). *预印本*, arXiv:2109.03264。

+   Kim等人（2024）Heeseung Kim, Soonshin Seo, Kyeongseok Jeong, Ohsung Kwon, Jungwhan Kim, Jaehong Lee, Eunwoo Song, Myungwoo Oh, Sungroh Yoon, 和Kang Min Yoo. 2024. 统一的语音-文本预训练用于语音对话建模. *CoRR*, abs/2402.05706。

+   Lakhotia等人（2021）Kushal Lakhotia, Evgeny Kharitonov, Wei-Ning Hsu, Yossi Adi, Adam Polyak, Benjamin Bolte, Tu-Anh Nguyen, Jade Copet, Alexei Baevski, Adelrahman Mohamed, 和Emmanuel Dupoux. 2021. [从原始音频进行生成式语音语言建模](https://arxiv.org/abs/2102.01192). *预印本*, arXiv:2102.01192。

+   Levinson和Torreira（2015a）Stephen C. Levinson和Francisco Torreira. 2015a. [轮次切换中的时机及其对语言处理模型的影响](https://doi.org/10.3389/fpsyg.2015.00731). *心理学前沿*, 6。

+   Levinson和Torreira（2015b）Stephen C. Levinson 和Francisco Torreira. 2015b. 轮次切换中的时机及其对语言处理模型的影响. *心理学前沿*, 6:731。

+   Lin等人（2022）Ting-En Lin, Yuchuan Wu, Fei Huang, Luo Si, Jian Sun, 和Yongbin Li. 2022. [双向对话：朝着类人交互的语音对话系统](https://doi.org/10.1145/3534678.3539209). 在*第28届ACM SIGKDD知识发现与数据挖掘会议论文集*，KDD ’22. ACM。

+   Maiti等人（2023）Soumi Maiti, Yifan Peng, Shukjae Choi, Jee-weon Jung, Xuankai Chang, 和Shinji Watanabe. 2023. Voxtlm: 统一的仅解码器模型，用于整合语音识别/合成和语音/文本继续任务. *CoRR*, abs/2309.07937。

+   Nachmani等人（2023）Eliya Nachmani, Alon Levkovitch, Roy Hirsch, Julian Salazar, Chulayuth Asawaroengchai, Soroosh Mariooryad, Ehud Rivlin, RJ Skerry-Ryan, 和Michelle Tadmor Ramanovich. 2023. 使用谱图驱动的大型语言模型进行语音问答和语音续接. 在*第十二届国际学习表示会议*。

+   Nguyen等人（2020）Tu Anh Nguyen, Maureen de Seyssel, Patricia Rozé, Morgane Rivière, Evgeny Kharitonov, Alexei Baevski, Ewan Dunbar, 和Emmanuel Dupoux. 2020. [零资源语音基准2021：用于无监督语音语言建模的指标和基线](https://arxiv.org/abs/2011.11588). *预印本*, arXiv:2011.11588。

+   Nguyen等人（2022）Tu Anh Nguyen, Eugene Kharitonov, Jade Copet, Yossi Adi, Wei-Ning Hsu, Ali Elkahky, Paden Tomasello, Robin Algayres, Benoit Sagot, Abdelrahman Mohamed, 和Emmanuel Dupoux. 2022. [生成式语音对话语言建模](https://arxiv.org/abs/2203.16502). *预印本*, arXiv:2203.16502。

+   Nguyen等人（2024）Tu Anh Nguyen, Benjamin Muller, Bokai Yu, Marta R. Costa-jussa, Maha Elbayad, Sravya Popuri, Paul-Ambroise Duquenne, Robin Algayres, Ruslan Mavlyutov, Itai Gat, Gabriel Synnaeve, Juan Pino, Benoit Sagot, 和Emmanuel Dupoux. 2024. [Spirit-lm: 交替的语音和书面语言模型](https://arxiv.org/abs/2402.05755). *预印本*, arXiv:2402.05755。

+   Nguyen et al. (2021) Vivian T Nguyen, Otto Versyp, Christopher Cox, and Riccardo Fusaroli. 2021. [成人与儿童语音互动中轮流交替发展的系统性回顾与贝叶斯元分析](https://api.semanticscholar.org/CorpusID:247547959)。*儿童发展*。

+   OpenAI (2023) OpenAI. 2023. [GPT-4技术报告](https://doi.org/10.48550/ARXIV.2303.08774)。*计算机科学报告*，abs/2303.08774。

+   Reece et al. (2023) Andrew Reece, Gus Cooney, Peter Bull, Christine Chung, Bryn Dawson, Casey Fitzpatrick, Tamara Glazer, Dean Knox, Alex Liebscher, and Sebastian Marin. 2023. [Candor语料库：来自大规模多模态自然对话数据集的见解](https://doi.org/10.1126/sciadv.adf3197)。*科学进展*，9(13):eadf3197。

+   Rubenstein et al. (2023) Paul K. Rubenstein, Chulayuth Asawaroengchai, Duc Dung Nguyen, Ankur Bapna, Zalán Borsos, Félix de Chaumont Quitry, Peter Chen, Dalia El Badawy, Wei Han, Eugene Kharitonov, Hannah Muckenhirn, Dirk Padfield, James Qin, Danny Rozenberg, Tara Sainath, Johan Schalkwyk, Matt Sharifi, Michelle Tadmor Ramanovich, Marco Tagliasacchi, Alexandru Tudor, Mihajlo Velimirović, Damien Vincent, Jiahui Yu, Yongqiang Wang, Vicky Zayats, Neil Zeghidour, Yu Zhang, Zhishuai Zhang, Lukas Zilka, and Christian Frank. 2023. [Audiopalm：一个能说话和听的语言模型](https://arxiv.org/abs/2306.12925)。*预印本*，arXiv:2306.12925。

+   Schegloff (2000) Emanuel A Schegloff. 2000. 重叠言语与会话轮流交替的组织。*社会语言学*，29(1):1–63。

+   Skantze (2021) Gabriel Skantze. 2021. 会话系统和人类-机器人互动中的轮流交替：一项回顾。*计算机、语音与语言*，67:101178。

+   Stivers et al. (2009) Tanya Stivers, Nick J. Enfield, Penelope Brown, Christina Englert, Makoto Hayashi, Trine Heinemann, Gertie Hoymann, Federico Rossano, Jan Peter De Ruiter, Kyung-Eun Yoon, Stephen C. Levinson, Paul Kay, and Krishna Y. 2009. [会话中轮流交替的普遍性与文化变异](https://api.semanticscholar.org/CorpusID:10200647)。*美国国家科学院院刊*，106:10587 – 10592。

+   ten Bosch et al. (2005) Louis ten Bosch, Nelleke Oostdijk, and Lou Boves. 2005. [会话对话中的时间性轮流交替问题](https://api.semanticscholar.org/CorpusID:44754316)。*语音通信*，47:80–86。

+   Touvron et al. (2023) Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, Dan Bikel, Lukas Blecher, Cristian Canton-Ferrer, Moya Chen, Guillem Cucurull, David Esiobu, Jude Fernandes, Jeremy Fu, Wenyin Fu, Brian Fuller, Cynthia Gao, Vedanuj Goswami, Naman Goyal, Anthony Hartshorn, Saghar Hosseini, Rui Hou, Hakan Inan, Marcin Kardas, Viktor Kerkez, Madian Khabsa, Isabel Kloumann, Artem Korenev, Punit Singh Koura, Marie-Anne Lachaux, Thibaut Lavril, Jenya Lee, Diana Liskovich, Yinghai Lu, Yuning Mao, Xavier Martinet, Todor Mihaylov, Pushkar Mishra, Igor Molybog, Yixin Nie, Andrew Poulton, Jeremy Reizenstein, Rashi Rungta, Kalyan Saladi, Alan Schelten, Ruan Silva, Eric Michael Smith, Ranjan Subramanian, Xiaoqing Ellen Tan, Binh Tang, Ross Taylor, Adina Williams, Jian Xiang Kuan, Puxin Xu, Zheng Yan, Iliyan Zarov, Yuchen Zhang, Angela Fan, Melanie Kambadur, Sharan Narang, Aurélien Rodriguez, Robert Stojnic, Sergey Edunov, and Thomas Scialom. 2023. Llama 2: 开放基础和微调对话模型. *CoRR*, abs/2307.09288.

+   Wang et al. (2023) Tianrui Wang, Long Zhou, Ziqiang Zhang, Yu Wu, Shujie Liu, Yashesh Gaur, Zhuo Chen, Jinyu Li, and Furu Wei. 2023. Viola: 统一的编解码语言模型用于语音识别、合成和翻译. *CoRR*, abs/2305.16107.

+   Zhang et al. (2023a) Dong Zhang, Shimin Li, Xin Zhang, Jun Zhan, Pengyu Wang, Yaqian Zhou, and Xipeng Qiu. 2023a. Speechgpt: 赋能大语言模型具备内在跨模态对话能力. 收录于 *2023年计算语言学协会发现会议：EMNLP 2023，新加坡，2023年12月6-10日*, 第15757–15773页。计算语言学协会出版。

+   Zhang et al. (2023b) Haoning Zhang, Junwei Bao, Haipeng Sun, Youzheng Wu, Wenye Li, Shuguang Cui, and Xiaodong He. 2023b. [Monet: 通过噪声增强训练解决对话状态追踪中的状态动量问题](https://arxiv.org/abs/2211.05503). *预印本*, arXiv:2211.05503.

+   Zhang et al. (2020) Yizhe Zhang, Siqi Sun, Michel Galley, Yen-Chun Chen, Chris Brockett, Xiang Gao, Jianfeng Gao, Jingjing Liu, and Bill Dolan. 2020. Dialogpt: 大规模生成预训练用于对话响应生成. 收录于 *ACL, 系统展示*。

+   Zhao et al. (2020) Xueliang Zhao, Wei Wu, Can Xu, Chongyang Tao, Dongyan Zhao, and Rui Yan. 2020. 基于知识的对话生成与预训练语言模型. 收录于 *2020年自然语言处理实证方法会议论文集，EMNLP 2020，线上会议，2020年11月16-20日*, 第3377–3390页。计算语言学协会出版。

## 附录 A 额外训练细节

### A.1 超参数

我们使用Llama3-8b的原始序列长度8192训练SyncLLM。在第一阶段，我们使用每个GPU批量大小为1，利用128个A100 GPU进行训练，相当于总批量为8192 x 128 = 1M令牌。我们使用学习率为$3\times{10}^{-5}$，进行500步的预热，并训练40k次迭代。在第二阶段，我们将批量大小减少到512k令牌，学习率调整为$2.2\times{10}^{-5}$，预热步数调整为200，训练6000次迭代。最后阶段，我们使用批量大小256k令牌，学习率为$1.5\times{10}^{-5}$，并进行100步预热，训练2000次迭代。

表5：关于交替级别的消融评估。WUGGY、BLIMP、Topic-StoryCloze和StoryCloze分别评估模型在词汇、句法和语义层面的知识与能力。我们报告基于负对数似然的准确性——按令牌数标准化——最小化预测。任务在零-shot设置下进行评估。

| 交替 | WUGGY$\uparrow$ | BLIMP$\uparrow$ | Topic-StoryCloze$\uparrow$ | StoryCloze$\uparrow$ |
| --- | --- | --- | --- | --- |
| 回合级别 | 63.0 | 56.0 | 76.5 | 55.1 |
| 句子级别 | 70.3 | 56.3 | 83.0 | 61.8 |

### A.2 交替策略基准测试

我们在训练的第一阶段探索了两种文本-语音交替策略：i) 句子级别交替：每个句子随机选择为文本模式或语音模式。ii) 回合级别交替：每个回合随机选择为文本模式或语音模式，从而确保回合内所有句子的模式一致。我们通过在Nguyen等人提出的口语语言理解基准测试集上进行评估来比较这两种策略（[2020](https://arxiv.org/html/2409.15594v1#bib.bib24)）。我们在表[5](https://arxiv.org/html/2409.15594v1#A1.T5 "Table 5 ‣ A.1 Hyperparameters ‣ Appendix A Additional training details ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents")中报告了这些结果。在这些任务中，我们观察到句子级别交替在所有基准测试中都优于回合级别交替。

表6：比较生成与真实继续之间回合时长事件的平均皮尔逊相关性，使用SyncLLM在两模型互动设置中的测量。测试集包括Fisher和Candor测试集。

| 延迟 | SyncLLM-F-F | SyncLLM-F-C |
| --- | --- | --- |
| 160 ms | 0.32 | 0.36 |
| 200 ms | 0.31 | 0.35 |
| 240 ms | 0.28 | 0.32 |

表7：比较提示与生成之间回合时长事件的皮尔逊相关性。

| 模型 | Fisher（分布内） |  | Candor（分布外） |
| --- | --- | --- | --- |
|  | ipu | pause | fto | 平均 |  | ipu | pause | fto | 平均 |
| dGSLM | 0.60 | 0.34 | 0.23 | 0.39 |  | 0.43 | 0.20 | 0.09 | 0.24 |
| SyncLLM-F (160 ms) | 0.69 | 0.34 | 0.35 | 0.46 |  | 0.64 | 0.12 | 0.24 | 0.33 |
| SyncLLM-F (200 ms) | 0.57 | 0.49 | 0.29 | 0.45 |  | 0.61 | 0.34 | 0.13 | 0.36 |
| SyncLLM-F (240 ms) | 0.63 | 0.49 | 0.33 | 0.48 |  | 0.59 | 0.23 | 0.19 | 0.34 |
| GT | 0.72 | 0.53 | 0.31 | 0.52 |  | 0.54 | 0.30 | 0.12 | 0.32 |

## 附录B 自然度-MOS说明

人类之间自然的回合转换表现为平滑的过渡，每个参与者互相倾听、适当回应，并留出停顿或沉默，形成平衡而动态的互动。通常，参与者会尽量避免话语重叠，尽管在某些情况下可能会发生，特别是当一个参与者通过使用“嗯”或“是的”之类的词语表明他们理解了对方时。犹豫、停顿、沉默和修正也是两人对话中常见的自然现象。

在此，您将听到两个人之间的对话，并为回合转换的自然程度提供评分，无论其内容（所用词语的意义）如何，声音的清晰度如何。

有些样本是由AI模型生成的，有些是实际的人工对话录音，还有些是实际的人工录音，但叠加了AI生成的声音。请尝试评估回合转换的自然度，而不考虑声音的特征。

首先，完整听一遍“提示”音频。这是对话的第一部分。然后，完整听一遍“续集”音频。这是对话的第二部分。请注意，在许多情况下，提示中的声音与续集中的声音可能有所不同（包括说话者的性别感知）。您的评分应反映“续集”音频在回合转换特征上的自然程度，考虑到您在“提示”中的观察。

### B.1 N-MOS & M-MOS

我们提供了用于评估回合转换自然度和对话内容意义的完整人类评估协议。

提供的音频

请根据您对“续集”音频中两个人自然交谈并互相倾听的印象来评分。

优秀 - 基本上无法与类人回合转换区分

良好 - 与类人回合转换的差异较小

公正 - 与类人回合转换有显著差异

差 - 与类人回合转换几乎没有相似之处

糟糕 - 与类人回合转换几乎没有任何相似之处

#### B.1.1 意义-MOS

在此任务中，您将听到两个人之间的对话，并为他们对话的意义提供评分。这里的“意义”指的是对话内容的连贯性和合理性（您是否能理解说话者的意图，听起来是否是人们会合理讨论的话题）。就像日常对话一样，内容可能不完全符合语法规范，但必须能在对话的上下文中理解。

首先，完整地听一遍“提示”音频。这是对话的第一部分。然后，完整地听一遍“延续”音频。这是对话的第二部分。请注意，在许多情况下，提示中的声音可能与延续中的声音不同（包括说话人的性别感知）。您的评分应反映出“延续”音频在给定“提示”的情况下是多么有意义。

提供的音频。

请根据您对“延续”是否是“提示”音频有意义的“延续”的印象来评分——即它是否代表了对话可能的合理发展，并且是连贯的。

{etaremune}

[topsep=0pt,itemsep=-1ex,partopsep=1ex,parsep=1ex]

优秀 - 所有对话内容都合理且连贯。

良好 - 大部分对话内容合理且连贯。

公平 - 部分对话内容是合理且连贯的。

较差 - 很少有对话内容是合理且连贯的。

差 - 基本没有对话内容是合理且连贯的。

![参见说明](img/140536d797ffb217b62f0e014393cbb3.png)

图 8：延迟对双模型交互的影响。

## 附录 C 延迟对全双工交互的影响。

在图 [8](https://arxiv.org/html/2409.15594v1#A2.F8 "Figure 8 ‣ B.1.1 Meaningfulness-MOS ‣ B.1 N-MOS & M-MOS ‣ Appendix B Naturalness-MOS Instructions ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents") 中，我们比较了不同延迟下的交互性能。我们发现我们的方法对高达 200 毫秒的延迟具有鲁棒性，但在延迟超过该值时，性能会下降。类似于我们在 §[5.2](https://arxiv.org/html/2409.15594v1#S5.SS2 "5.2 Naturalness evaluation ‣ 5 Experiments ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents") 中对延续设置的自然性评估，为了评估 SyncLLM 在交互设置中的轮流对话能力，我们比较了生成和真实续集中的轮流事件持续时间的 Pearson 相关性。在表 [6](https://arxiv.org/html/2409.15594v1#A1.T6 "Table 6 ‣ A.2 Benchmarking interleaving strategies ‣ Appendix A Additional training details ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents") 中，我们观察到，在结合了分布内和分布外提示的测试集上，交互设置下的性能在延迟为 160 毫秒和 200 毫秒时表现接近，但在 240 毫秒时有所下降。

## 附录 D 提示与生成之间的轮流事件相关性。

类似于表[2](https://arxiv.org/html/2409.15594v1#S4.T2 "Table 2 ‣ 4 Training ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents")中的自然性评估，在该表中我们将真实的延续作为回合事件统计的参考，我们也可以将提示作为参考。从某种意义上讲，这衡量了提示和延续之间的风格一致性。在表[7](https://arxiv.org/html/2409.15594v1#A1.T7 "Table 7 ‣ A.2 Benchmarking interleaving strategies ‣ Appendix A Additional training details ‣ Beyond Turn-Based Interfaces: Synchronous LLMs as Full-Duplex Dialogue Agents")中，我们比较了我们方法在延续设置中的回合事件相关性与dGSLM方法的回合事件相关性。我们观察到，我们的方法在与提示的回合相关性方面表现更佳。
