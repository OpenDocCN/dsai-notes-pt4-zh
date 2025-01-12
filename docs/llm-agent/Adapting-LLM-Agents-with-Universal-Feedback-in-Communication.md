<!--yml

分类：未分类

日期：2025-01-11 13:05:58

-->

# 通过通信中的通用反馈调整 LLM 代理

> 来源：[https://arxiv.org/html/2310.01444/](https://arxiv.org/html/2310.01444/)

Kuan Wang¹, Yadong Lu ², Michael Santacroce², Yeyun Gong³,

Chao Zhang¹, Yelong Shen²

¹乔治亚理工学院   ²微软 Azure AI   ³微软研究院

{kuanwang, chaozhang}@gatech.edu

{yadonglu, misantac, yegong, yeshe}@microsoft.com

###### 摘要

大型语言模型（LLMs）最近的进展展示了 LLM 代理的潜力。为了促进这些代理的训练，结合语言反馈和非语言奖励信号，我们提出了通过通信进行学习（LTC）。我们设计了一个通用缓冲区来存储所有反馈，并设计了一个迭代管道，使得 LLM 代理能够在给定环境中探索并更新其策略。为了利用我们的通用缓冲区捕捉代理在各种任务中的交互，我们引入了多样化的通信模式，适用于单代理和多代理环境。我们在四个不同的数据集上评估了我们的 LTC 方法的有效性：ALFWorld（单代理），HotpotQA（多代理协作），Chameleon（多代理竞争）和 GSM8k（多代理师生）。在这些数据集上，LTC 的表现比监督式指令微调基准提高了 3.6% 至 12%。这些结果证明了 LTC 在促进 LLM 代理在线适应方面的多样性和有效性。

## 1 引言

大型语言模型（LLMs）最近的进展（Ouyang 等人，([2022](https://arxiv.org/html/2310.01444v3#bib.bib33)）；Bubeck 等人，([2023](https://arxiv.org/html/2310.01444v3#bib.bib6)）；Wei 等人，([2022a](https://arxiv.org/html/2310.01444v3#bib.bib63)）揭示了类人 LLM 代理的潜力。除了设计提示方法（Wei 等人，([2022b](https://arxiv.org/html/2310.01444v3#bib.bib64)）；Yao 等人，([2023](https://arxiv.org/html/2310.01444v3#bib.bib71)）；Wu 等人，([2023a](https://arxiv.org/html/2310.01444v3#bib.bib67)）），近期的研究还集中在如何通过语言反馈和非语言奖励信号来训练 LLM 代理。语言反馈通常作为指令数据进行处理，用于进行指令微调（IFT）（Chung 等人，([2022](https://arxiv.org/html/2310.01444v3#bib.bib9)）；Lee 等人，([2023](https://arxiv.org/html/2310.01444v3#bib.bib19)）；Honovich 等人，([2022](https://arxiv.org/html/2310.01444v3#bib.bib14)）；Wang 等人，([2022e](https://arxiv.org/html/2310.01444v3#bib.bib60)）），而非语言奖励信号则通常用于与人类偏好进行对齐（Ouyang 等人，([2022](https://arxiv.org/html/2310.01444v3#bib.bib33)）；Bai 等人，([2022a](https://arxiv.org/html/2310.01444v3#bib.bib3)）；Stiennon 等人，([2020](https://arxiv.org/html/2310.01444v3#bib.bib51)）；Leike 等人，([2018](https://arxiv.org/html/2310.01444v3#bib.bib20)））。

虽然一些场景为智能体提供异质反馈，但现有方法只能部分利用这些反馈。例如，在多人桌面角色扮演游戏中，玩家生成大量的语言数据，而游戏以明确的奖励信号（如胜利或失败）结束。当前的方法使用这些语言数据进行IFT Li等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib21)）；Micheli & Fleuret（[2021](https://arxiv.org/html/2310.01444v3#bib.bib28)），而奖励信号仅作为选择ILF数据的过滤标准，而不是强化学习的目标。

图1：LTC框架适用于单智能体和多智能体环境。在这些环境中，智能体能够持续进行探索和互动，通过各种通信模式收集轨迹。同时，LTC还支持利用智能体从探索活动中获得的数据来训练这些智能体。这个过程使得智能体能够自主地适应各自的环境，避免了人工监督的必要。

为了填补这一空白，我们提出了一种通用框架，命名为通过通信学习（LTC），用于训练具有语言反馈和非语言奖励信号的LLM智能体。我们设计了一个通用缓冲区来存储所有反馈，并设计了一个迭代管道，使得LLM智能体能够在给定环境中探索并更新其策略。LTC的每次迭代包括两个不同的阶段：(1) *探索*：在这个阶段，智能体与环境及其他智能体进行互动，收集各种轨迹（语言性）和奖励信号（非语言性），并将其存入通用缓冲区。(2) *更新*：在此阶段，智能体的模型基于通用缓冲区中收集的数据进行更新。为了进行更新，LTC将语言建模损失与PPO损失结合起来，以在语言一致性和奖励信号之间取得平衡。作为迭代管道的核心，重放缓冲区在每次探索阶段后进行更新，并从缓冲区中抽取一个子集用于更新阶段。

为了在通信过程中通用地支持语言反馈和非语言奖励信号，我们设计了回放缓冲区结构作为一系列代币序列的轨迹（图[2](https://arxiv.org/html/2310.01444v3#S2.F2 "图 2 ‣ 2.1 指令调优 ‣ 2 相关工作 ‣ 在通信中使用通用反馈适配LLM智能体")）。这种回放缓冲区结构适用于各种任务，包括单智能体和多智能体环境。为了方便收集带有语言数据和奖励信号的轨迹，我们设计了三种通信模式：（1）*单智能体独白*：该模式允许单个智能体收集包含语言数据的轨迹，并接收来自环境的奖励信号。（2）*多智能体对话*：该模式使多个智能体可以相互互动，并与外部工具进行交互，收集语言数据，并利用环境提供的奖励信号。（3）*师生对话*：这种多智能体对话变种通过教师智能体而非环境提供语言反馈和非语言奖励信号。

我们在几个具有代表性的数据集上评估了LTC方法：ALFWorld用于决策制定，HotpotQA用于知识密集型推理，以及GSM8k用于数值推理。在这些实验中，LTC始终优于基准方法。在ALFWorld中，LTC在成功率上比强指令调优基准高出12%，即使在具有挑战性的Pick 2任务中也是如此。这表明我们的通信机制使得智能体能够通过经验学习以解决任务。在HotpotQA中，LTC在EM分数上比指令调优基准高出5%，而基于Llama-7B的智能体甚至比使用9$\times$更大PaLM-62B模型的ReAct-Tuning基准表现稍好（提高了0.6%）。在GSM8k中，LTC也在准确率上比CoT-Tuning基准高出3.6%。这些结果突显了LTC方法在不同领域中的适应性和有效性。

我们的主要贡献是：

1.  1.

    通过通信学习（LTC）：我们提出了一种通用框架，称为通过通信学习（LTC），用于训练大规模语言模型（LLM）智能体，结合语言反馈和非语言奖励信号。我们设计了一个通用的缓冲区来存储所有反馈，并通过一个迭代流程，使LLM智能体能够在给定环境中探索并更新其策略。

1.  2.

    特定任务的通信模式：LTC范式允许灵活设计针对不同任务的通信模式。我们介绍了三种特定模式：单智能体独白、多智能体对话和师生对话。这些模式可以组合生成多样化的结构化互动和反馈信号，以便为智能体训练提供支持，适应不同的任务类型。

1.  3.

    实证研究与发现：我们在公共基准任务上进行了严格的实验，旨在展示LTC的有效性。我们的结果表明，与指令调优或提示基线相比，LTC可以是一个更优的方案。

## 2 相关工作

### 2.1 指令调优

![参见说明文字](img/1b4963745b02ce9dcc97062f8824c214.png)

图 2：缓冲区数据是一系列整数/浮动序列。我们将每个 token id 视为强化学习公式中的动作。我们还保存其相应的掩码，指示 token 的来源、来自评论模型的值、表示采样动作时对数似然的对数概率（log-prob）以及来自环境/其他代理的奖励。

指令微调（IT）是一种重要的技术，用于提高大语言模型（LLMs）的能力和可控性，Radford 等人（[2019](https://arxiv.org/html/2310.01444v3#bib.bib38)）；Brown 等人（[2020](https://arxiv.org/html/2310.01444v3#bib.bib5)）；Wei 等人（[2022a](https://arxiv.org/html/2310.01444v3#bib.bib63)）；Qin 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib37)）；OpenAI（[2023](https://arxiv.org/html/2310.01444v3#bib.bib32)）；Chowdhery 等人（[2022](https://arxiv.org/html/2310.01444v3#bib.bib8)）；Touvron 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib55)）。许多研究致力于指令数据的生成和选择，Chung 等人（[2022](https://arxiv.org/html/2310.01444v3#bib.bib9)）；Wang 等人（[2022e](https://arxiv.org/html/2310.01444v3#bib.bib60)）；Lee 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib19)）。例如，Unnatural Instructions，由 Honovich 等人（[2022](https://arxiv.org/html/2310.01444v3#bib.bib14)）创建，使用 Super-Natural Instructions 数据集作为种子来提示 InstructGPT，Ouyang 等人（[2022](https://arxiv.org/html/2310.01444v3#bib.bib33)）。Self-Instruct，由 Wang 等人（[2022e](https://arxiv.org/html/2310.01444v3#bib.bib60)）提出，采用递归管道，通过使用 ChatGPT（OpenAI，[2022](https://arxiv.org/html/2310.01444v3#bib.bib31)）从手工制作的种子任务生成指令数据。其他研究则集中于使用指令数据对预训练的 LLM 进行微调。BLOOMZ，由 Muennighoff 等人（[2022](https://arxiv.org/html/2310.01444v3#bib.bib30)）提出，是以 BLOOM 为基础（Scao 等人，[2022](https://arxiv.org/html/2310.01444v3#bib.bib43)），然后使用 xP3 指令数据集进行微调（Muennighoff 等人，[2022](https://arxiv.org/html/2310.01444v3#bib.bib30)）。Flan-T5 是以 T5 为基础（Raffel 等人，[2020](https://arxiv.org/html/2310.01444v3#bib.bib40)），并使用 FLAN 数据集进行微调（Longpre 等人，[2023](https://arxiv.org/html/2310.01444v3#bib.bib25)）。此外，在 LLaMA 发布后（Touvron 等人，[2023](https://arxiv.org/html/2310.01444v3#bib.bib55)），许多工作已将其作为指令微调的基础模型，例如 Alpaca（Taori 等人，[2023](https://arxiv.org/html/2310.01444v3#bib.bib54)）、Vicuna（Chiang 等人，[2023](https://arxiv.org/html/2310.01444v3#bib.bib7)）和 GPT-4-LLM（Peng 等人，[2023](https://arxiv.org/html/2310.01444v3#bib.bib35)）。一些论文探索了使用 RLHF 对齐微调，Ouyang 等人（[2022](https://arxiv.org/html/2310.01444v3#bib.bib33)）；Bai 等人（[2022a](https://arxiv.org/html/2310.01444v3#bib.bib3)）；Stiennon 等人（[2020](https://arxiv.org/html/2310.01444v3#bib.bib51)）；Leike 等人（[2018](https://arxiv.org/html/2310.01444v3#bib.bib20)）。InstructGPT（Ouyang 等人，[2022](https://arxiv.org/html/2310.01444v3#bib.bib33)）采用 GPT-3 在人工过滤的指令数据集上进行监督微调，然后训练奖励模型并使用 PPO（Schulman 等人，[2017](https://arxiv.org/html/2310.01444v3#bib.bib45)）进行 RLHF。Claude 探索了 RLHF（Bai 等人，[2022a](https://arxiv.org/html/2310.01444v3#bib.bib3)）和宪法方法（Bai 等人，[2022b](https://arxiv.org/html/2310.01444v3#bib.bib4)），以使 LLM 既无害又有用。DPO（Rafailov 等人，[2023](https://arxiv.org/html/2310.01444v3#bib.bib39)）通过直接优化偏好数据上的分类问题来微调 LLM，使其与人类偏好对齐，而不是使用 RLHF。这些杰出的研究工作专注于使 LLM 更好地执行一般的指令跟随任务，而我们的目标是将 LLM 代理调整为特定的任务或环境。

### 2.2 LLM 代理

LLMs 已展示出作为高级代理的潜力 Ouyang 等人（[2022](https://arxiv.org/html/2310.01444v3#bib.bib33)）；Bubeck 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib6)）；Wei 等人（[2022a](https://arxiv.org/html/2310.01444v3#bib.bib63)），并且在开发多功能 LLM 代理方面取得了显著进展 Weng（[2023](https://arxiv.org/html/2310.01444v3#bib.bib65)）；Sumers 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib52)）；Park 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib34)）；Liu 等人（[2023a](https://arxiv.org/html/2310.01444v3#bib.bib23)）；Lin 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib22)）；Xu 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib69)）以及基准测试 Wang 等人（[2022a](https://arxiv.org/html/2310.01444v3#bib.bib56)）；Deng 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib11)）；Liu 等人（[2023b](https://arxiv.org/html/2310.01444v3#bib.bib24)）。在规划方面，Chain-of-Thought（CoTWei 等人（[2022b](https://arxiv.org/html/2310.01444v3#bib.bib64)））通过将复杂任务分解为更小、更简单的步骤来促使模型逐步思考。Self Consistency（Wang 等人，[2022c](https://arxiv.org/html/2310.01444v3#bib.bib58)；[d](https://arxiv.org/html/2310.01444v3#bib.bib59)）通过使用预测集合来扩展 CoT，改进 LLM 的一致性。Inner Monologue Huang 等人（[2022b](https://arxiv.org/html/2310.01444v3#bib.bib17)）利用环境反馈增强 LLM 在具身机器人任务中的规划和处理能力，无需额外训练。ReAct Yao 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib71)）结合了推理和行动，将动作空间扩展为既包括任务特定的离散动作，也包括语言。Reflexion Shinn 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib47)）为代理提供了动态记忆和自我反思能力，通过在相同环境中进行连续试验以反馈来改进推理。最近的研究还表明，LLMs 可以作为自主代理，通过使用*外部工具*在互动环境中解决问题。这些技术包括检索增强 Shi 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib46)）；Yao 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib71)）；Izacard 等人（[2022](https://arxiv.org/html/2310.01444v3#bib.bib18)），数学工具 Schick 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib44)）；Yao 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib71)）；Lu 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib27)），以及代码解释器 Gao 等人（[2022](https://arxiv.org/html/2310.01444v3#bib.bib12)）；Wang 等人（[2022b](https://arxiv.org/html/2310.01444v3#bib.bib57)）。之前的工作还探讨了在协作环境中使用多个 LLM 共同解决复杂任务 Hong 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib13)）；Qian 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib36)）；Li 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib21)）；Wang 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib62)）；Talebirad & Nadiri（[2023](https://arxiv.org/html/2310.01444v3#bib.bib53)）；Akata 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib1)）。开源项目如 AutoGPT Significant-Gravitas（[2023](https://arxiv.org/html/2310.01444v3#bib.bib50)），GPT-Engineer AntonOsika（[2023](https://arxiv.org/html/2310.01444v3#bib.bib2)），以及 BabyAGI yoheinakajima（[2023](https://arxiv.org/html/2310.01444v3#bib.bib72)）也展示了 LLM 不仅在生成内容方面的潜力，还作为一个通用问题求解者。上述大部分方法基于人工设计的少量示例提示，或者通过使用预收集的指令数据集进行微调。我们的 LTC 不是一种少量示例提示方法，我们专注于通过探索自动收集训练数据来调整代理。

## 3 通过通信学习

我们设计了“通过通信学习”（Learning Through Communication，简称LTC），这是一种针对LLM代理的迭代训练方法，用于使其持续适应新环境。如图[3](https://arxiv.org/html/2310.01444v3#S3.F3 "Figure 3 ‣ 3 Learning Through Communication ‣ Adapting LLM Agents with Universal Feedback in Communication")所示，LTC在两个阶段之间迭代：（1）探索阶段，代理可以与新环境和其他代理进行互动，收集带有反馈的试验数据；（2）更新阶段，通过微调代理来更新策略。

图3：LTC具有迭代的两阶段框架。在探索阶段，代理主动探索新环境并与其他代理进行通信，收集轨迹以更新重放缓冲区。然后，代理在更新阶段进行训练以更新策略。

### 3.1 探索阶段

在每次迭代开始时，代理会探索环境以获取轨迹和奖励信号数据。我们将这些数据表示为一个元组：$\mathcal{S}=(\mathcal{T},\mathcal{M},\mathcal{R})$，其中$\mathcal{T}=\{t_{1},t_{2},\dots,t_{n}\}$表示代理探索过程中由通信过程生成的文本数据，$\mathcal{M}=\{m_{1},m_{2},\dots,m_{n}\}$，其中$m_{i}\in\{0,1,2\}$表示文本数据的来源（系统或代理），$\mathcal{R}=\{r_{1},r_{2},\dots,r_{n}\}$，其中$r_{i}\in\{-1,0,1\}$表示由系统或代理提供的奖励信号。我们在图[2](https://arxiv.org/html/2310.01444v3#S2.F2 "Figure 2 ‣ 2.1 Instruction Tuning ‣ 2 Related Work ‣ Adapting LLM Agents with Universal Feedback in Communication")中展示了该数据结构的详细信息，$\mathcal{M}$是掩码列表，$\mathcal{R}$是奖励列表。在PPO训练中，值列表和对数概率列表直接对应于动作列表。为了简洁起见，我们将这三个列表统称为$\mathcal{T}$。更多细节请见附录[A.2](https://arxiv.org/html/2310.01444v3#A1.SS2 "A.2 Buffer Structure ‣ Appendix A Appendix ‣ Adapting LLM Agents with Universal Feedback in Communication")。

图4：展示通信模式的玩具示例：1）左侧图示是多智能体对话模式，其中两个智能体扮演不同的角色，共同协作完成任务。思考者智能体负责分析情况并向行动者智能体提供建议，后者则负责做出决策。在没有GPT-4智能体的情况下，我们可以直接将LTC智能体分配为思考者智能体进行测试。2）右侧图示是师生对话模式，其中学生智能体首先给出当前问题的初步答案，然后教师通过奖励直接纠正答案。为了帮助学生提升能力，而不仅仅是记住解法，教师会生成另一个类似的问题来提问学生。最终，学生为这个类似问题给出新的答案，并从教师那里获得新的奖励信号。

为了收集来自不同类型任务的轨迹数据 $\mathcal{S}=(\mathcal{T},\mathcal{M},\mathcal{R})$，我们为这些任务设计了通信模式。这里提供了三种通信模式：

+   •

    单智能体独白：单智能体独白是一种单一智能体的独白式沟通模式，设计用于一般的指令跟随任务（算法 [1](https://arxiv.org/html/2310.01444v3#alg1 "算法 1 ‣ 附录A ‣ 在通信中适配LLM智能体的通用反馈")）。它将任务分解成一步步执行的形式，类似于ReAct和CoT，并且通过收集各自的轨迹和系统奖励来训练自身，同时进行探索。图 [1](https://arxiv.org/html/2310.01444v3#S1.F1 "图 1 ‣ 1 引言 ‣ 在通信中适配LLM智能体的通用反馈")左侧是ALFWorld的玩具示例，展示了单一智能体的独白模式。该智能体通过独白思考情况并采取行动来探索环境，最终获得环境提供的奖励。此模式基于ReAct框架中的思考和行动步骤 Yao等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib71)），我们设计了训练缓冲区收集过程，使其与我们的强化学习框架对齐。

+   •

    多智能体对话：多智能体对话是一种多智能体讨论风格模式（算法 [2](https://arxiv.org/html/2310.01444v3#alg2 "算法 2 ‣ A.1 通信模式 ‣ 附录 A 附录 ‣ 通过普适反馈在通信中调整LLM智能体")）。它旨在多智能体协作和竞争任务中使用，其中多个智能体将按照一定顺序扮演各自的角色，通过言语或行动与其他智能体进行互动，最终根据智能体的表现，由环境给予奖励。图 [4](https://arxiv.org/html/2310.01444v3#S3.F4 "图 4 ‣ 3.1 探索阶段 ‣ 3 通过通信学习 ‣ 通过普适反馈在通信中调整LLM智能体")的左图展示了一个HotpotQA的玩具示例，用来说明这一协作模式，其中GPT-4智能体充当思考者，分析情况并向演员智能体提供建议，演员智能体则负责做出决策。HotpotQA中的奖励是两个智能体所获得答案的正确性。我们可以利用他们的通信数据来训练LTC智能体，使其既能担任思考者，又能担任演员，从而学会如何协作解决任务。图 [1](https://arxiv.org/html/2310.01444v3#S1.F1 "图 1 ‣ 1 引言 ‣ 通过普适反馈在通信中调整LLM智能体")的右图则展示了一个多智能体对话的玩具示例，适用于竞争性游戏任务Chameleon，其中三个智能体扮演不同的角色。奖励是游戏的胜负，因此它们需要在沟通过程中进行推理和虚张声势，以赢得游戏。它们的游戏轨迹将用于LTC迭代中，以增强智能体的能力。

+   •

    教师-学生对话：教师-学生对话是一种教师与学生的模式，旨在让强大的代理帮助新手代理进行教学（算法 [3](https://arxiv.org/html/2310.01444v3#alg3 "算法 3 ‣ A.1 通信模式 ‣ 附录 A ‣ 在通信中适应具有通用反馈的LLM代理")）。我们为复杂的分析任务（如数值推理）设计了这种模式，这些任务需要大量的分析示例来帮助代理改进预训练模型中缺乏的特定推理能力。教师-学生对话模式有两个角色（学生和教师），由两个代理扮演，然而，除了语言反馈外，教师角色还可以直接提供非语言奖励信号，这些信号是由系统（环境）在之前的模式中提供的。图[4](https://arxiv.org/html/2310.01444v3#S3.F4 "图 4 ‣ 3.1 探索阶段 ‣ 3 通过通信学习 ‣ 在通信中适应具有通用反馈的LLM代理")右侧的图示是一个玩具示例，使用GSM8k演示学生代理如何以批改作业的方式与教师代理进行交流。在数学问题环境中，学生代理首先对当前问题给出初步答案，然后教师直接通过奖励来纠正答案。为了帮助学生提高能力，而不仅仅是记住解决方案，教师将生成另一个独立的问题，并为学生提供新的奖励。

### 3.2 更新阶段

在更新阶段，LLM代理模型可以通过在探索阶段收集的对话会话进行优化。给定一个示例会话$\mathcal{S}=(\mathcal{T},\mathcal{M},\mathcal{R})$，我们主要利用两个训练目标进行模型训练。

+   •

    语言模型目标：$\mathcal{L}_{\text{LM}}$鼓励模型从轨迹$\mathcal{T}$中学习，作为一种无监督学习方案，帮助模型从其他代理的响应中进行行为克隆或预测系统反馈。

+   •

    强化目标：$\mathcal{L}_{\text{reinforce}}$通过最大化环境或教师代理（即GPT-4 OpenAI ([2023](https://arxiv.org/html/2310.01444v3#bib.bib32)))提供的期望奖励来优化模型。这是一个目标导向的目标，允许模型通过通信会话中的正负信号进行学习。

因此，LTC的整体训练目标结合了上述两个项：

|  | $\mathcal{L}_{\text{LTC}}(\mathcal{S})=\beta\mathcal{L}_{\text{LM}}(\mathcal{T}% )+\mathcal{L}_{\text{reinforce}}(\mathcal{S}),$ |  | (1) |
| --- | --- | --- | --- |

其中，$\beta$是一个平衡超参数。离策略PPO算法（Schulman 等人，[2017](https://arxiv.org/html/2310.01444v3#bib.bib45)）用于优化$\mathcal{L}_{\text{reinforce}}(\mathcal{S})$，并且在实现中可以进一步分解为策略损失、值损失和策略熵正则化项。原始PPO算法使用三元组$(\text{state},\text{action},\text{rewards})$进行训练。在这种情况下，我们从轨迹$(\mathcal{T}_{<i},t_{i})$中采样，以模拟状态-动作对，具体来说，我们仅保留由代理模型本身生成的令牌作为动作来更新策略。

## 4 实验

### 4.1 数据集

我们在四个数据集上进行了实验：ALFWorld（Shridhar 等人，[2020b](https://arxiv.org/html/2310.01444v3#bib.bib49)）、HotpotQA（Yang 等人，[2018](https://arxiv.org/html/2310.01444v3#bib.bib70)）、Chameleon Wu 等人（[2023b](https://arxiv.org/html/2310.01444v3#bib.bib68)）和GSM8k（Cobbe 等人，[2021](https://arxiv.org/html/2310.01444v3#bib.bib10)）。这些数据集代表了不同的环境类型，分别是单一代理、多个代理合作、多个代理竞争和教师-学生环境。此外，使用了不同的通信模式：ALFWorld使用单一代理独白，HotpotQA和Chameleon Wu 等人（[2023b](https://arxiv.org/html/2310.01444v3#bib.bib68)）使用多代理对话，而GSM8k使用教师-学生对话。

#### ALFWorld

ALFWorld（图 [1](https://arxiv.org/html/2310.01444v3#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Adapting LLM Agents with Universal Feedback in Communication")）是一个基于文本的游戏，遵循ALFRED基准（Shridhar 等人，[2020a](https://arxiv.org/html/2310.01444v3#bib.bib48)）。在这个游戏中，代理需要完成六种任务，这些任务涉及使用文本动作在模拟的家庭环境中进行导航。该环境有超过50个位置供探索，这些任务要求进行战略规划和深入探索。根据(Shridhar 等人，[2020b](https://arxiv.org/html/2310.01444v3#bib.bib49))，我们使用包含3553个环境的训练集来训练我们的模型和基准；并且使用包含134个环境的未见测试集进行评估。

#### HotpotQA

HotpotQA 是一个问答数据集，重点是基于多跳推理的支持性事实，旨在提高问答系统的可解释性。在这个数据集中，代理需要跨越两个或更多的 Wikipedia 文章段落进行推理，从而得出答案。我们只使用问题文本来初始化环境，这意味着代理被提供问题和任务描述，但无法访问支持段落。为了支持推理，代理必须依赖其内部知识或与外部 Wikipedia 工具互动，获取必要的信息。对于训练，我们从训练集（包含 90,447 对 QA）中抽取环境。对于评估，我们从测试集中运行 500 个随机示例，参考了（Yao 等， [2023](https://arxiv.org/html/2310.01444v3#bib.bib71)）。

#### Chameleon

Chameleon 是一个由 ChatArena Wu 等人 ([2023b](https://arxiv.org/html/2310.01444v3#bib.bib68)) 实现的多玩家社交推理游戏环境。游戏中有两种角色，变色龙和非变色龙。首先，所有玩家都会看到秘密词汇的主题。然后，秘密词汇将被揭示给非变色龙。非变色龙试图在不透露秘密词汇的情况下识别出变色龙，而变色龙则尝试融入并猜测词汇。游戏包括提供线索、投票选出可能是变色龙的人，以及被指控的变色龙的最终猜测。我们使用 [3, 4, 5] 玩家设置来训练和测试代理的表现。

#### GSM8k

GSM8k 数据集是一个包含 8.5K 数学题目的数据集，针对小学生设计。这些问题由人工专家精心制作，以确保语言的多样性。数据集分为两部分：7.5K 题目用于训练，1K 题目用于测试。数据集中的每个问题需要 2 到 8 步推理才能得出解决方案。这些问题主要集中在基本的算术运算，如加法、减法、乘法和除法。

### 4.2 设置

#### 模型架构

我们使用了一个修改版的 Llama Touvron 等人 ([2023](https://arxiv.org/html/2310.01444v3#bib.bib55)) 作为基础模型。为了生成与动作令牌对应的状态值，我们引入了一个额外的线性层作为值头。这个值头充当辅助输出模块，输出值使用 $tanh()$ 函数进行处理，以确保其值在 (-1, 1) 范围内。这种针对强化学习的适配在以前的研究中也有所讨论（Santacroce 等， [2023](https://arxiv.org/html/2310.01444v3#bib.bib42)）。

#### 代理预训练

我们使用Llama-7B模型（Touvron等，[2023](https://arxiv.org/html/2310.01444v3#bib.bib55)）作为我们的LLM智能体。为了增强智能体执行任务特定指令的能力，我们通过指令微调（IT）对其进行初始化。这个初始化后的智能体作为基准，用于公平比较。这个步骤至关重要，因为原始的Llama-7B模型在没有指令微调的情况下，难以遵循任务指令并在环境中生成合理的动作。为了收集指令微调的数据，我们使用GPT3/4作为智能体，探索由训练集生成的环境。我们随后筛选出负面示例，保留正面示例以训练初始智能体。对于ALFWorld和HotpotQA数据集，我们使用GPT3（具体来说是text-davinci-003）。然而，对于GSM8k数据集，我们使用GPT4，因为GPT3在处理数学问题时表现不佳，导致正面示例的稀缺。

#### 训练细节

我们使用AdamW优化器（Loshchilov & Hutter, [2017](https://arxiv.org/html/2310.01444v3#bib.bib26)），批量大小为32。学习率设置为2e-4。在每次迭代中，智能体探索新环境的大小分别为：ALFWorld为256，GSM8k为512，HotpotQA为1024。为了进行参数高效的微调，我们采用LoRA（Hu等，[2021](https://arxiv.org/html/2310.01444v3#bib.bib15)），超参数为$R=16$和$\alpha=16$。对于分布式训练，我们在HotpotQA和GSM8k上使用4个节点，每个节点配置8$\times$A100 GPU。对于ALFWorld的实验，由于数据集较小，我们仅使用1个节点，配备2$\times$A100 GPU。

#### 基准

我们将通过LTC训练的智能体与现有的提示与指令微调方法进行比较，包括ReAct（Yao等，[2023](https://arxiv.org/html/2310.01444v3#bib.bib71)），ReAct-IM（Huang等，[2022b](https://arxiv.org/html/2310.01444v3#bib.bib17)），CoT（Wei等，[2022b](https://arxiv.org/html/2310.01444v3#bib.bib64)），CoT-SC（Wang等，[2022c](https://arxiv.org/html/2310.01444v3#bib.bib58); [d](https://arxiv.org/html/2310.01444v3#bib.bib59)），BUTLER Micheli & Fleuret（[2021](https://arxiv.org/html/2310.01444v3#bib.bib28)）。这些基准的详细信息在附录[A.7](https://arxiv.org/html/2310.01444v3#A1.SS7 "A.7 Baselines ‣ Appendix A Appendix ‣ Adapting LLM Agents with Universal Feedback in Communication")中描述。大多数方法集中在少样本提示上，并使用不同的预训练模型。为了确保公平比较，我们还包括了名为ReAct-Tuning和CoT-Tuning的附加基准，通过微调Llama-7B模型，使用收集到的轨迹作为微调数据。此外，测试时未使用GPT-4，所有报告的结果均由训练后的智能体本身获得。

### 4.3 结果

#### ALFWorld

| 方法 \ 任务 | 选取 | 清洁 | 加热 | 冷却 | 查找 | 选取 2 | 全部 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ReAct (平均) | 65 | 39 | 83 | 76 | 55 | 24 | 57 |
| ReAct（最佳的6个） | 92 | 58 | 96 | 86 | 78 | 41 | 71 |
| ReAct-IM（平均） | 55 | 59 | 60 | 55 | 23 | 24 | 48 |
| ReAct-IM（最佳的6个） | 62 | 68 | 87 | 57 | 39 | 33 | 53 |
| BUTLER[g]（最佳的8个） | 33 | 26 | 70 | 76 | 17 | 12 | 22 |
| BUTLER（最佳的8个） | 46 | 39 | 74 | 100 | 22 | 24 | 37 |
| ReAct-Tuning（平均） | 83 | 91 | 91 | 90 | 72 | 8 | 77 |
| ReAct-Tuning（最佳的3个） | 92 | 97 | 96 | 95 | 78 | 24 | 78 |
| LTC（平均） | 89 | 91 | 93 | 97 | 96 | 67 | 90 |
| LTC（最佳的3个） | 92 | 97 | 96 | 100 | 100 | 76 | 91 |

表1：ALFWorld六个任务的成功率（%）。底部块的结果是通过微调Llama-7B模型获得的。

如表[1](https://arxiv.org/html/2310.01444v3#S4.T1 "Table 1 ‣ ALFWorld ‣ 4.3 Results ‣ 4 Experiments ‣ Adapting LLM Agents with Universal Feedback in Communication")所示，LTC超越了之前最好的方法^*^**对于ALFWorld，ReAct和ReAct-IM的结果来自Yao等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib71)）的表3。BUTLER和BUTLER[g]的结果来自Shridhar等人（[2020b](https://arxiv.org/html/2310.01444v3#bib.bib49)）的表4，并且它们是通过DAgger Ross等人（[2011](https://arxiv.org/html/2310.01444v3#bib.bib41)）训练的。针对ALFWorld的所有任务。我们可以看到，指令微调已经是一个强有力的基线，优于其他方法，而我们的LTC取得了91%的成功率，显著超越了最好的指令微调基线（78%）。值得注意的是，在Cool和Look任务上，LTC获得了100%的成功率。

| 模型 | 方法 | EM得分 |
| --- | --- | --- |
| PaLM-540B | CoT (Wei et al., [2022b](https://arxiv.org/html/2310.01444v3#bib.bib64)) | 29.4 |
| CoT-SC (Wang et al., [2022c](https://arxiv.org/html/2310.01444v3#bib.bib58)) | 33.4 |
| ReAct (Yao et al., [2023](https://arxiv.org/html/2310.01444v3#bib.bib71)) | 27.4 |
|  | ReAct $\to$ CoT-SC | 35.1 |
| GPT3-175B | ReAct | 30.8 |
| PaLM-62B | ReAct-Tuning | 32.6 |
| CoT-Tuning | 25.2 |
| PaLM-8B | ReAct-Tuning | 25.0 |
| CoT-Tuning | 14.0 |
| Llama-7B | ReAct-Tuning | 28.2 |
| LTC（单智能体独白） | 31.0 |
|  | LTC（多智能体对话） | 33.2 |
| Llama2-13B | ReAct-Tuning | 33.8 |
| LTC（多智能体对话） | 35.8 |

表2：使用提示和微调方法的HotpotQA EM得分。使用微调方法的标记为“-Tuning”。

即使是在最困难的“选择两个并放置”任务（例如，“把两支铅笔放进抽屉”）中，它也能达到相当不错的76%的成功率。这个“选择两个”任务要求代理在一个任务中执行两次“拾取并放置”操作，同时跟踪目标类型和位置。多个操作序列和记住前一个位置的需求使得这个任务具有挑战性。这可能是基准方法在该任务中成功率较低的原因。相比之下，我们的LTC代理通过自我探索进一步训练代理，显著优于其他代理。这凸显了LTC中通信机制的有效性。

#### HotpotQA

如表[2](https://arxiv.org/html/2310.01444v3#S4.T2 "表 2 ‣ ALFWorld ‣ 4.3 结果 ‣ 4 实验 ‣ 使用通用反馈在通信中调整LLM代理")所示，LTC在精确匹配（EM）得分上超过了指令调优基线^†^††对于HotPotQA，未微调的提示方法结果来自Yao等人的表1和表5（[2023](https://arxiv.org/html/2310.01444v3#bib.bib71)）。PaLM-8B和PaLM-62B的得分是Yao等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib71)）在图3中的估算值，领先5%，并且在默认设置下，甚至超过了ReAct和CoT。需要注意的是，ReAct和CoT使用的是PaLM-540B和GPT3-175B作为预训练的语言模型，分别是我们使用的Llama-7B模型的77倍和25倍。通过在推理过程中采样21条CoT轨迹并采用多数回答，CoT-SC的表现略优于LTC（提高了0.2%），而它们的组合方法ReAct $\to$ CoT-SC则超过了LTC 1.9%。与其他经过调优的模型相比，我们基于Llama-7B的代理甚至比使用9$\times$更大PaLM-62B模型的ReAct-Tuning基线获得了略微更好的表现（提高了0.6%）。

| 方法 \ 玩家数 | n=3 | n=4 | n=5 | 总体 |
| --- | --- | --- | --- | --- |
| Llama-Tuning | 20.8 | 20.3 | 23.8 | 21.9 |
| Llama-LTC | 22.9 | 23.4 | 27.5 | 25.0 |

表 3：不同玩家人数下变色龙游戏的获胜率（%）。

#### Chameleon

如表[3](https://arxiv.org/html/2310.01444v3#S4.T3 "表 3 ‣ HotpotQA ‣ 4.3 结果 ‣ 4 实验 ‣ 使用通用反馈在通信中调整LLM代理")所示，LTC在与GPT-4玩家对战时，获胜率比指令调优基线提高了3.1%。在训练中，所有玩家都由我们正在训练的相同的Llama2-7B模型扮演。而在测试中，为了获得我们训练的代理与GPT4对战的获胜率，只有1个玩家随机选择使用我们训练的代理作为后台，其他玩家由GPT4扮演。我们可以看到，随着玩家人数的增加，LTC代理的获胜率有所提高，我们解释为玩家越多，GPT4玩家获胜的机会越大。

#### GSM8k

如表[4](https://arxiv.org/html/2310.01444v3#S4.T4 "表 4 ‣ GSM8k ‣ 4.3 结果 ‣ 4 实验 ‣ 适应具有普遍反馈的LLM代理")所示，LTC（师生对话）比指令微调基线高出3.6%的准确率，并且超越了LTC（单一代理独白）基线，后者未使用来自GPT-4的奖励和反馈。

| 模型 | 方法 | 准确率 |
| --- | --- | --- |
| PaLM-540B | CoT (Wei 等, [2022b](https://arxiv.org/html/2310.01444v3#bib.bib64)) | 56.5 |
| CoT-SC (Wang 等, [2022c](https://arxiv.org/html/2310.01444v3#bib.bib58)) | 74.4 |
| GPT3-175B | CoT (Wei 等, [2022b](https://arxiv.org/html/2310.01444v3#bib.bib64)) | 60.1 |
| CoT-SC (Wang 等, [2022c](https://arxiv.org/html/2310.01444v3#bib.bib58)) | 78.0 |
| Llama-7B | CoT (Touvron 等, [2023](https://arxiv.org/html/2310.01444v3#bib.bib55)) | 11.0 |
| CoT-SC (Touvron 等, [2023](https://arxiv.org/html/2310.01444v3#bib.bib55)) | 18.1 |
| Llama-7B | CoT-Tuning | 37.7 |
| LTC（单一代理独白） | 39.6 |
| LTC（师生对话） | 41.3 |

表 4：GSM8k准确率。底部模块的结果是通过微调获得的，而其他方法则是提示方法。

然而，LTC在使用更大模型（如PaLM-540B和GPT3-175B）时的表现不如CoT和CoT-SC。这一现象的原因是数值推理需要更大的模型规模和充分的预训练数据，正如OpenAI所观察到的([2023](https://arxiv.org/html/2310.01444v3#bib.bib32))。不幸的是，由于计算资源的限制，我们只能训练相对较小的Llama-7B模型，而无法训练更大规模的模型。尽管如此，我们相信，探索使用更大模型的LTC对未来的研究具有潜力。

## 5 讨论

#### 快捷方式

一个有趣的观察是，当作为教师生成新训练数据时，GPT-4代理有时会使用“捷径”来解决问题。这些捷径依赖于其在预训练过程中获得的内部知识。为了说明这一点，我们在图[7](https://arxiv.org/html/2310.01444v3#A1.F7 "图 7 ‣ A.3 LTC 算法 ‣ 附录 A ‣ 使用通用反馈调整LLM代理的通信")中展示了一个来自HotpotQA的案例研究。在这个案例中，GPT-4代理在接收到第一个条目的维基百科页面后，通过利用其记忆中的关于第二个条目的知识，迅速检索出答案。另一方面，图[7](https://arxiv.org/html/2310.01444v3#A1.F7 "图 7 ‣ A.3 LTC 算法 ‣ 附录 A ‣ 使用通用反馈调整LLM代理的通信")底部展示了与LLaMA-7B的比较，后者使用我们通过GPT-4代理进行循环训练的LTC方法。LLaMA-7B没有使用捷径，而是进行了第二条目的搜索。这个案例研究表明，与单纯依赖GPT-4生成的数据相比，LTC中的通信机制在学习过程中提供了额外的好处。

#### 效率

| 方法 | GSM8k | Hotpot-QA | Alfworld |
| --- | --- | --- | --- |
| (CoT) | (ReAct) | (ReAct) |
| --- | --- | --- |
| ICL | 836 | 1937 | 1744 |
| LTC | 107 | 167 | 189 |

表 5：测试集输入提示的平均标记数量。LTC在提示中没有使用任何少量示例，因此与ICL相比，LTC只使用了较少的标记。

如上所述，基于提示的方法，如ReAct (Yao等，[2023](https://arxiv.org/html/2310.01444v3#bib.bib71))和CoT (Wei等，[2022b](https://arxiv.org/html/2310.01444v3#bib.bib64))，在推理时使用来自给定任务的示例轨迹子集作为少量示例提示。然而，这些少量示例提示通常很长，这会导致推理成本增加，并且限制了用户查询的上下文长度。如表[5](https://arxiv.org/html/2310.01444v3#S5.T5 "表 5 ‣ 效率 ‣ 5 讨论 ‣ 使用通用反馈调整LLM代理的通信")所示，我们比较了每个任务的输入标记数量。我们计算了GSM8k的CoT提示，并为其他两个任务使用了ReAct。所有少量示例提示均来自原始论文。如所示，我们的LTC代理仅使用了ICL方法在这三个任务中所需输入标记的12.8%、8.6%和10.8%。

图 5：训练的准确率曲线。

图 6：训练的损失曲线。

#### 消融实验

我们对LTC的损失设计进行了消融研究。图[6](https://arxiv.org/html/2310.01444v3#S5.F6 "Figure 6 ‣ Efficiency ‣ 5 Discussion ‣ Adapting LLM Agents with Universal Feedback in Communication")展示了在不同损失设置下，代理在ALFWorld数据集上的成功率。没有使用我们的通信模式进行交互，仅仅通过采样预先收集的指令数据进行训练时，改进是有限的。然而，当我们结合我们的通信模式来收集数据时，模型的表现迅速超过了80%。此外，采用PPO损失单独处理正负样本，带来了更快、更显著的改进（蓝线）。在图[6](https://arxiv.org/html/2310.01444v3#S5.F6 "Figure 6 ‣ Efficiency ‣ 5 Discussion ‣ Adapting LLM Agents with Universal Feedback in Communication")中，我们展示了训练过程中三种主要损失的单独曲线。最初，语言模型（LM）损失呈下降趋势。有趣的是，随着训练迭代的进行，值损失和策略损失逐渐下降，这可能导致LM损失暂时上升。经过一定阈值后，值损失和策略损失达到某个阈值，LM损失继续下降直至收敛。

## 6 结论

我们介绍了基于通信的学习（Learning-Through-Communication，LTC）框架，这是一种通过基于通信的迭代学习，将LLM代理适应新任务和环境的范式。在这个LTC框架中，我们为常见任务设计了三种通信模式，包括决策、知识密集型推理和数值推理。这些通信模式促进了LLM代理与其环境之间的互动，以及与其他代理（如GPT-4和人类）的互动。这些互动的历史可以自我组织成PPO训练数据，从而使代理能够适应新任务。我们的方法代表了一个闭环，代理通过与环境或其他代理的自我互动来学习并改进自己，且只需最少的人类干预。从实证结果来看，我们已证明LTC在成功率和效率上，在四个不同任务（AlfWorld、HotpotQA、Chameleon和GSM8k）中表现出色。它始终优于现有的LLM代理和指令调优基线，显示出LTC范式在使LLM代理适应新任务和环境、且需要最少人类努力方面的前景。未来的工作中，我们计划探索更多适用于不同任务的通信模式，并在迭代学习过程中引入与人类的通信。我们将开源我们的代码，以促进这一方向的进一步研究。

## 参考文献

+   Akata等（2023）Elif Akata、Lion Schulz、Julian Coda-Forno、Seong Joon Oh、Matthias Bethge和Eric Schulz。与大型语言模型进行重复博弈。*arXiv预印本arXiv:2305.16867*，2023年。

+   AntonOsika（2023）AntonOsika。gpt-engineer。[https://github.com/AntonOsika/gpt-engineer](https://github.com/AntonOsika/gpt-engineer)，2023年。GitHub库。

+   Bai等人（2022a）云涛·白、安迪·琼斯、卡马尔·恩杜塞、阿曼达·阿斯凯尔、安娜·陈、诺瓦·达萨尔马、道恩·德雷恩、斯坦尼斯拉夫·福特、深·甘古利、汤姆·赫尼根等。通过人类反馈的强化学习训练一个有帮助且无害的助手。*arXiv预印本arXiv:2204.05862*，2022a年。

+   Bai等人（2022b）云涛·白、萨乌拉夫·卡达瓦斯、山迪潘·昆杜、阿曼达·阿斯凯尔、杰克逊·凯尼翁、安迪·琼斯、安娜·陈、安娜·戈尔迪、阿扎利亚·米尔霍塞尼、卡梅隆·麦金农等。宪法AI：来自AI反馈的无害性。*arXiv预印本arXiv:2212.08073*，2022b年。

+   Brown等人（2020）汤姆·布朗、本杰明·曼、尼克·赖德、梅兰妮·苏比亚、贾里德·D·卡普兰、普拉富拉·达里瓦尔、阿尔温·尼拉坎坦、普拉纳夫·夏姆、吉里什·萨斯特里、阿曼达·阿斯凯尔、桑迪尼·阿格瓦尔、阿里尔·赫伯特-沃斯、格雷琴·克鲁格、汤姆·赫尼根、雷旺·查尔德、阿迪亚·拉梅什、丹尼尔·齐格勒、杰弗里·吴、克莱门斯·温特、克里斯·赫斯、马克·陈、埃里克·西格勒、马泰乌什·利特温、斯科特·格雷、本杰明·切斯、杰克·克拉克、克里斯托弗·伯纳、山姆·麦坎德利什、亚历克·拉德福德、伊利亚·苏茨克弗、达里奥·阿莫代伊。语言模型是少样本学习者。在H. Larochelle、M. Ranzato、R. Hadsell、M. F. Balcan和H. Lin（编）编辑的《*神经信息处理系统进展*》，第33卷，第1877-1901页。Curran Associates, Inc.，2020年。

+   Bubeck等人（2023）塞巴斯蒂安·布贝克、瓦伦·钱德拉塞卡兰、罗嫩·埃尔丹、约翰内斯·格赫尔基、埃里克·霍维茨、埃茨·卡马尔、彼得·李、尹·塔特·李、袁智·李、斯科特·伦伯格等。人工通用智能的火花：与GPT-4的早期实验。*arXiv预印本arXiv:2303.12712*，2023年。

+   Chiang等人（2023）魏林·蒋、卓涵·李、子林、英胜、张浩·吴、浩·张、连敏·郑、思源·庄、永浩·庄、约瑟夫·E·冈萨雷斯等。Vicuna：一款开源聊天机器人，凭借90%*的ChatGPT质量令GPT-4印象深刻。*参见https://vicuna.lmsys.org（访问时间：2023年4月14日）*，2023年。

+   Chowdhery等人（2022）阿卡恩莎·乔杜里、沙兰·纳朗、雅各布·德夫林、马尔滕·博斯马、高尔·米什拉、亚当·罗伯茨、保罗·巴姆、玄元·郑、查尔斯·萨顿、塞巴斯蒂安·格赫尔曼等。Palm：通过路径扩展语言建模。*arXiv预印本arXiv:2204.02311*，2022年。

+   Chung等人（2022）玄元·郑、乐·侯、谢恩·朗普雷、巴雷特·佐夫、易·泰、威廉·费达斯、埃里克·李、薛志·王、穆斯塔法·德哈尼、悉达多·布拉马等。扩展指令微调语言模型。*arXiv预印本arXiv:2210.11416*，2022年。

+   Cobbe等人（2021）卡尔·科比、维内特·科萨拉朱、穆罕默德·巴瓦里安、马克·陈、赫乌·俊、卢卡什·凯泽、马修·普拉佩特、杰里·图雷克、雅各布·希尔顿、雷一郎·中野、克里斯托弗·赫斯和约翰·舒尔曼。训练验证者解决数学文字问题。*arXiv预印本arXiv:2110.14168*，2021年。

+   Deng 等人（2023）Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Samuel Stevens, Boshi Wang, Huan Sun, 和 Yu Su。Mind2web：迈向面向网络的通用代理，2023年。

+   Gao 等人（2022）Luyu Gao, Aman Madaan, Shuyan Zhou, Uri Alon, Pengfei Liu, Yiming Yang, Jamie Callan, 和 Graham Neubig。Pal：程序辅助的语言模型。*arXiv 预印本 arXiv:2211.10435*，2022年。

+   Hong 等人（2023）Sirui Hong, Xiawu Zheng, Jonathan Chen, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, Chenyu Ran, Lingfeng Xiao, 和 Chenglin Wu。Metagpt：面向多代理协作框架的元编程，2023年。

+   Honovich 等人（2022）Or Honovich, Thomas Scialom, Omer Levy, 和 Timo Schick。非自然指令：用（几乎）不需要人工劳动的方式调优语言模型。*arXiv 预印本 arXiv:2212.09689*，2022年。

+   Hu 等人（2021）Edward J Hu, Yelong Shen, Phillip Wallis, Zeyuan Allen-Zhu, Yuanzhi Li, Shean Wang, Lu Wang, 和 Weizhu Chen。Lora：大规模语言模型的低秩适应。*arXiv 预印本 arXiv:2106.09685*，2021年。

+   Huang 等人（2022a）Jiaxin Huang, Shixiang Shane Gu, Le Hou, Yuexin Wu, Xuezhi Wang, Hongkun Yu, 和 Jiawei Han。大型语言模型可以自我改进，2022a年。

+   Huang 等人（2022b）Wenlong Huang, Fei Xia, Ted Xiao, Harris Chan, Jacky Liang, Pete Florence, Andy Zeng, Jonathan Tompson, Igor Mordatch, Yevgen Chebotar, Pierre Sermanet, Noah Brown, Tomas Jackson, Linda Luu, Sergey Levine, Karol Hausman, 和 Brian Ichter。内心独白：通过规划与语言模型的推理。载于 *arXiv 预印本 arXiv:2207.05608*，2022b年。

+   Izacard 等人（2022）Gautier Izacard, Patrick Lewis, Maria Lomeli, Lucas Hosseini, Fabio Petroni, Timo Schick, Jane Dwivedi-Yu, Armand Joulin, Sebastian Riedel, 和 Edouard Grave。通过检索增强的语言模型进行少样本学习。*arXiv 预印本 arXiv:2208.03299*，2022年。

+   Lee 等人（2023）Harrison Lee, Samrat Phatale, Hassan Mansoor, Kellie Lu, Thomas Mesnard, Colton Bishop, Victor Carbune, 和 Abhinav Rastogi。Rlaif：通过 AI 反馈扩展人类反馈的强化学习。*arXiv 预印本 arXiv:2309.00267*，2023年。

+   Leike 等人（2018）Jan Leike, David Krueger, Tom Everitt, Miljan Martic, Vishal Maini, 和 Shane Legg。通过奖励建模进行可扩展的代理对齐：一项研究方向，2018年。

+   Li 等人（2023）Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, 和 Bernard Ghanem。Camel：用于“大规模语言模型社会”中的“心智”探索的交互代理，2023年。

+   Lin 等人（2023）Bill Yuchen Lin, Yicheng Fu, Karina Yang, Prithviraj Ammanabrolu, Faeze Brahman, Shiyu Huang, Chandra Bhagavatula, Yejin Choi, 和 Xiang Ren。Swiftsage：一种具有快慢思维的生成代理，用于复杂的交互任务。*ArXiv 预印本*，abs/2305.17390，2023年。网址 [https://arxiv.org/abs/2305.17390](https://arxiv.org/abs/2305.17390)。

+   Liu等人（2023a）Ruibo Liu, Ruixin Yang, Chenyan Jia, Ge Zhang, Denny Zhou, Andrew M Dai, Diyi Yang, 和Soroush Vosoughi。训练社会对齐的语言模型以模拟人类社会。*arXiv预印本arXiv:2305.16960*，2023a。

+   Liu等人（2023b）Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang, Shudan Zhang, Xiang Deng, Aohan Zeng, Zhengxiao Du, Chenhui Zhang, Sheng Shen, Tianjun Zhang, Yu Su, Huan Sun, Minlie Huang, Yuxiao Dong, 和 Jie Tang。Agentbench: 评估LLMs作为代理。*arXiv预印本arXiv: 2308.03688*，2023b。

+   Longpre等人（2023）Shayne Longpre, Le Hou, Tu Vu, Albert Webson, Hyung Won Chung, Yi Tay, Denny Zhou, Quoc V Le, Barret Zoph, Jason Wei，等。Flan集合：设计有效的指令调优数据和方法。*arXiv预印本arXiv:2301.13688*，2023。

+   Loshchilov & Hutter（2017）Ilya Loshchilov 和 Frank Hutter。解耦的权重衰减正则化。*arXiv预印本arXiv:1711.05101*，2017。

+   Lu等人（2023）Pan Lu, Baolin Peng, Hao Cheng, Michel Galley, Kai-Wei Chang, Ying Nian Wu, Song-Chun Zhu, 和Jianfeng Gao。Chameleon: 插件式的大型语言模型组合推理。*arXiv预印本arXiv:2304.09842*，2023。

+   Micheli & Fleuret（2021）Vincent Micheli 和 François Fleuret。语言模型是少样本的管家。*arXiv预印本arXiv:2104.07972*，2021。

+   Mnih等人（2016）Volodymyr Mnih, Adria Puigdomenech Badia, Mehdi Mirza, Alex Graves, Timothy Lillicrap, Tim Harley, David Silver, 和Koray Kavukcuoglu。深度强化学习的异步方法。在*国际机器学习会议*上，pp. 1928–1937。PMLR，2016。

+   Muennighoff等人（2022）Niklas Muennighoff, Thomas Wang, Lintang Sutawika, Adam Roberts, Stella Biderman, Teven Le Scao, M Saiful Bari, Sheng Shen, Zheng-Xin Yong, Hailey Schoelkopf，等。通过多任务微调实现跨语言泛化。*arXiv预印本arXiv:2211.01786*，2022。

+   OpenAI（2022）OpenAI。ChatGPT。在线，2022。网址[https://openai.com/blog/chatgpt](https://openai.com/blog/chatgpt)。

+   OpenAI（2023）OpenAI。GPT-4技术报告。*OpenAI博客*，2023。

+   Ouyang等人（2022）Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray，等。训练语言模型通过人类反馈遵循指令。*神经信息处理系统进展*，35:27730–27744，2022。

+   Park等人（2023）Joon Sung Park, Joseph C. O’Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, 和Michael S. Bernstein。生成代理：人类行为的交互式模拟体。*在第36届ACM用户界面软件与技术年会（UIST ’23）*上，UIST ’23，美国纽约，2023。计算机协会。

+   Peng等人（2023）Baolin Peng, Chunyuan Li, Pengcheng He, Michel Galley, 和Jianfeng Gao。使用GPT-4的指令调优。*arXiv预印本arXiv:2304.03277*，2023。

+   Qian等人（2023）Chen Qian、Xin Cong、Cheng Yang、Weize Chen、Yusheng Su、Juyuan Xu、Zhiyuan Liu和Maosong Sun。软件开发的沟通代理。*arXiv预印本arXiv:2307.07924*，2023年。

+   Qin等人（2023）Chengwei Qin、Aston Zhang、Zhuosheng Zhang、Jiaao Chen、Michihiro Yasunaga和Diyi Yang。ChatGPT是一个通用自然语言处理任务求解器吗？2023年。

+   Radford等人（2019）Alec Radford、Jeff Wu、Rewon Child、David Luan、Dario Amodei和Ilya Sutskever。语言模型是无监督的多任务学习者。*OpenAI博客*，1(8):9，2019年。

+   Rafailov等人（2023）Rafael Rafailov、Archit Sharma、Eric Mitchell、Stefano Ermon、Christopher D. Manning和Chelsea Finn。直接偏好优化：你的语言模型其实是一个奖励模型，2023年。

+   Raffel等人（2020）Colin Raffel、Noam Shazeer、Adam Roberts、Katherine Lee、Sharan Narang、Michael Matena、Yanqi Zhou、Wei Li和Peter J. Liu。通过统一的文本到文本变换器探索迁移学习的极限。*机器学习研究杂志*，21(140):1–67，2020年。

+   Ross等人（2011）Stéphane Ross、Geoffrey Gordon和Drew Bagnell。模仿学习和结构化预测的减少为无悔的在线学习。在*第十四届国际人工智能与统计会议论文集*中，第627–635页。JMLR工作坊与会议论文集，2011年。

+   Santacroce等人（2023）Michael Santacroce、Yadong Lu、Han Yu、Yuanzhi Li和Yelong Shen。高效RLHF：减少PPO的内存使用，2023年。

+   Scao等人（2022）Teven Le Scao、Angela Fan、Christopher Akiki、Ellie Pavlick、Suzana Ilić、Daniel Hesslow、Roman Castagné、Alexandra Sasha Luccioni、François Yvon、Matthias Gallé等人。Bloom：一个176b参数的开放访问多语言语言模型。*arXiv预印本arXiv:2211.05100*，2022年。

+   Schick等人（2023）Timo Schick、Jane Dwivedi-Yu、Roberto Dessì、Roberta Raileanu、Maria Lomeli、Luke Zettlemoyer、Nicola Cancedda和Thomas Scialom。Toolformer：语言模型可以自学使用工具。*arXiv预印本arXiv:2302.04761*，2023年。

+   Schulman等人（2017）John Schulman、Filip Wolski、Prafulla Dhariwal、Alec Radford和Oleg Klimov。近端策略优化算法，2017年。

+   Shi等人（2023）Weijia Shi、Sewon Min、Michihiro Yasunaga、Minjoon Seo、Rich James、Mike Lewis、Luke Zettlemoyer和Wen-tau Yih。Replug：检索增强的黑箱语言模型。*arXiv预印本arXiv:2301.12652*，2023年。

+   Shinn等人（2023）Noah Shinn、Federico Cassano、Beck Labash、Ashwin Gopinath、Karthik Narasimhan和Shunyu Yao。Reflexion：具有语言强化学习的语言代理，2023年。

+   Shridhar 等人（2020a）Mohit Shridhar、Jesse Thomason、Daniel Gordon、Yonatan Bisk、Winson Han、Roozbeh Mottaghi、Luke Zettlemoyer 和 Dieter Fox。Alfred：用于解释日常任务的基础指令的基准测试。在 *IEEE/CVF 计算机视觉与模式识别会议论文集*，第10740–10749页，2020a。

+   Shridhar 等人（2020b）Mohit Shridhar、Xingdi Yuan、Marc-Alexandre Côté、Yonatan Bisk、Adam Trischler 和 Matthew Hausknecht。Alfworld：为互动学习对齐文本和具身环境。*arXiv 预印本 arXiv:2010.03768*，2020b。

+   Significant-Gravitas（2023）Significant-Gravitas。Autogpt。 [https://github.com/Significant-Gravitas/AutoGPT](https://github.com/Significant-Gravitas/AutoGPT)，2023。GitHub 存储库。

+   Stiennon 等人（2020）Nisan Stiennon、Long Ouyang、Jeffrey Wu、Daniel Ziegler、Ryan Lowe、Chelsea Voss、Alec Radford、Dario Amodei 和 Paul F Christiano。通过人类反馈学习总结。*神经信息处理系统进展*，33:3008–3021，2020。

+   Sumers 等人（2023）Theodore Sumers、Shunyu Yao、Karthik Narasimhan 和 Thomas L Griffiths。语言代理的认知架构。*arXiv 预印本 arXiv:2309.02427*，2023。

+   Talebirad & Nadiri（2023）Yashar Talebirad 和 Amirhossein Nadiri。多智能体协作：利用智能大型语言模型代理的力量。*arXiv 预印本 arXiv:2306.03314*，2023。

+   Taori 等人（2023）Rohan Taori、Ishaan Gulrajani、Tianyi Zhang、Yann Dubois、Xuechen Li、Carlos Guestrin、Percy Liang 和 Tatsunori B Hashimoto。Alpaca：一个强大且可复制的指令跟随模型。*斯坦福大学基础模型研究中心。 https://crfm.stanford.edu/2023/03/13/alpaca.html*，3(6):7，2023。

+   Touvron 等人（2023）Hugo Touvron、Thibaut Lavril、Gautier Izacard、Xavier Martinet、Marie-Anne Lachaux、Timothée Lacroix、Baptiste Rozière、Naman Goyal、Eric Hambro、Faisal Azhar 等人。Llama：开放且高效的基础语言模型。*arXiv 预印本 arXiv:2302.13971*，2023。

+   Wang 等人（2022a）Ruoyao Wang、Peter Jansen、Marc-Alexandre Côté 和 Prithviraj Ammanabrolu。Scienceworld：你的智能体比五年级学生更聪明吗？，2022a。网址 [https://arxiv.org/abs/2203.07540](https://arxiv.org/abs/2203.07540)。

+   Wang 等人（2022b）Xingyao Wang、Sha Li 和 Heng Ji。Code4struct：从自然语言进行少样本结构化预测的代码生成。*arXiv 预印本 arXiv:2210.12810*，2022b。

+   Wang 等人（2022c）Xuezhi Wang、Jason Wei、Dale Schuurmans、Quoc Le、Ed Chi、Sharan Narang、Aakanksha Chowdhery 和 Denny Zhou。自一致性提高语言模型中的思维链推理，2022c。网址 [https://arxiv.org/abs/2203.11171](https://arxiv.org/abs/2203.11171)。

+   Wang 等人（2022d）Xuezhi Wang、Jason Wei、Dale Schuurmans、Quoc Le、Ed Chi 和 Denny Zhou。语言模型中的推理增强集成。*arXiv 预印本 arXiv:2207.00747*，2022d。

+   王等（2022e）王一中、耶加内·科尔迪、斯瓦罗普·米什拉、阿丽莎·刘、诺亚·A·史密斯、丹尼尔·哈沙比和哈娜内·哈吉希尔齐。自我指导：使语言模型与自生成的指令对齐。*arXiv 预印本 arXiv:2212.10560*，2022e。

+   王等（2022f）王一中、斯瓦罗普·米什拉、佩加·阿利普尔莫拉巴希、耶加内·科尔迪、阿米雷扎·米尔扎伊、安贾娜·阿伦库马尔、阿尔俊·阿肖克、阿鲁特·塞尔万·丹哈塞卡兰、阿塔尔瓦·奈克、大卫·斯塔普等。超自然指令：通过1600+ NLP任务的声明式指令进行泛化。*arXiv 预印本 arXiv:2204.07705*，2022f。

+   王等（2023）王振海龙、毛绍光、吴文山、葛涛、魏富如和季恒。释放大型语言模型中的认知协同：通过多角色自我协作解决任务的智能体。*arXiv 预印本 arXiv:2307.05300*，2023。

+   魏等（2022a）杰森·魏、易泰、瑞希·博马萨尼、科林·拉费尔、巴雷特·佐夫、塞巴斯蒂安·博尔戈、丹尼·尤加塔马、马尔滕·博斯马、丹尼·周、唐纳德·梅茨勒、爱德·H·奇、田角忍、奥里奥尔·维尼亚尔斯、帕西·梁、杰夫·迪恩和威廉·费杜斯。大型语言模型的突现能力，2022a。

+   魏等（2022b）杰森·魏、王雪志、戴尔·舒尔曼、马尔滕·博斯马、爱德·奇、阔·黎和丹尼·周。思维链提示激发大型语言模型中的推理。*arXiv 预印本 arXiv:2201.11903*，2022b。

+   翁（2023）丽莲·翁。LLM 驱动的自主智能体。*lilianweng.github.io*，2023年6月。网址 [https://lilianweng.github.io/posts/2023-06-23-agent/](https://lilianweng.github.io/posts/2023-06-23-agent/)。

+   威廉姆斯（1992）罗纳德·J·威廉姆斯。用于连接主义强化学习的简单统计梯度跟踪算法。*机器学习*，8:229–256，1992。

+   吴等（2023a）吴青云、加根·班萨尔、张杰宇、吴逸然、张绍昆、朱尔康、李贝宾、蒋李、张晓云和王驰。Autogen：通过多智能体对话框架推动下一代 LLM 应用。*arXiv 预印本 arXiv:2308.08155*，2023a。

+   吴等（2023b）吴玉翔、蒋正耀、阿克比尔·汗、傅瑶、劳拉·鲁伊斯、爱德华·格雷芬斯特特和蒂姆·罗克塔谢尔。Chatarena：面向大型语言模型的多智能体语言游戏环境。[https://github.com/chatarena/chatarena](https://github.com/chatarena/chatarena)，2023b。

+   徐等（2023）徐彬锋、彭志远、雷博文、穆卡布拉塔·穆克吉、刘宇辰和徐东宽。Rewoo：将推理与观察解耦，以提高增强型语言模型的效率。*arXiv 预印本 arXiv:2305.18323*，2023。

+   杨等（2018）杨志林、齐鹏、张赛政、约书亚·本吉奥、威廉·W·科恩、鲁斯兰·萨拉库丁诺夫和克里斯托弗·D·曼宁。HotpotQA：一个多跳问答的多样化、可解释数据集。发表于*自然语言处理经验方法会议（EMNLP）*，2018年。

+   Yao 等人（2023）Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R Narasimhan 和 Yuan Cao。React：在语言模型中协同推理与行动。载于 *第十一届国际学习表示大会*，2023。URL [https://openreview.net/forum?id=WE_vluYUL-X](https://openreview.net/forum?id=WE_vluYUL-X)。

+   yoheinakajima（2023）yoheinakajima。Babyagi。[https://github.com/yoheinakajima/babyagi](https://github.com/yoheinakajima/babyagi)，2023。GitHub 仓库。

## 附录 A 附录

[⬇](data:text/plain;base64,IyBhZ2VudDogTExhTUEgYWdlbnQKIyBpbnB1dDogIFRhc2sgZGVzY3JpcHRpb24KIyBvdXRwdXQ6IFMgID0gKFQsIE0sIFIpCgojIGluaXRpYWxpemF0aW9uClQsIE0sIFIgPSBbaW5wdXRdLCBbMF0sIFswXQoKaSA9IDAKd2hpbGUgaSA8IG1heF9zdGVwczoKICAgIFQgKz0gWyJ0aGluazoiXQogICAgdGhvdWdodCA9IGFnZW50LmFwaShUKQogICAgVC5hcHBlbmQodGhvdWdodCkKICAgIE0uYXBwZW5kKDEpICMgYWdlbnQgbWVzc2FnZSBtYXNrCiAgICBSLmFwcGVuZCgwKQoKCiAgICBUICs9IFsiYWN0OiJdCiAgICBhY3Rpb24gPSBhZ2VudC5hcGkoVCkKICAgIFQuYXBwZW5kKGFjdGlvbikKICAgIE0uYXBwZW5kKDEpICMgYWdlbnQgbWVzc2FnZSBtYXNrCiAgICBSLmFwcGVuZCgwKQoKICAgIHJlc3BvbnNlID0gZW52LmV4Y3V0ZShhY3Rpb24pCiAgICByZXdhcmQgPSBwYXJzZShyZXNwb25zZSkKICAgIFQuYXBwZW5kKHJlc3BvbnNlKQogICAgTS5hcHBlbmQoMCkgIyBzeXN0ZW0gbWVzc2FnZSBtYXNrCiAgICBSLmFwcGVuZChyZXdhcmQpCgogICAgaSArPSAxCiAgICBpZiByZXdhcmQgIT0gMDoKICAgICAgICBicmVhawpTICA9IChULCBNLCBSKQpyZXR1cm4gUw==)#  agent:  LLaMA  agent#  input:  Task  description#  output:  S  =  (T,  M,  R)#  initializationT,  M,  R  =  [input],  [0],  [0]i  =  0while  i  <  max_steps:T  +=  ["think:"]thought  =  agent.api(T)T.append(thought)M.append(1)  #  agent  message  maskR.append(0)T  +=  ["act:"]action  =  agent.api(T)T.append(action)M.append(1)  #  agent  message  maskR.append(0)response  =  env.excute(action)reward  =  parse(response)T.append(response)M.append(0)  #  system  message  maskR.append(reward)i  +=  1if  reward  !=  0:breakS  =  (T,  M,  R)return  S

算法 1 演示独白模式的 Python 风格算法

### A.1 通信模式

为了收集来自不同类型任务的轨迹和奖励信号数据，我们设计了这些任务的通信模式，并统一了数据格式，如图[2](https://arxiv.org/html/2310.01444v3#S2.F2 "图 2 ‣ 2.1 指令调优 ‣ 2 相关工作 ‣ 在通信中通过普适反馈调整大语言模型代理")所示。在这里，我们使用了三种 Python 风格的算法（算法[1](https://arxiv.org/html/2310.01444v3#alg1 "算法 1 ‣ 附录 A ‣ 在通信中通过普适反馈调整大语言模型代理")、算法[2](https://arxiv.org/html/2310.01444v3#alg2 "算法 2 ‣ A.1 通信模式 ‣ 附录 A ‣ 在通信中通过普适反馈调整大语言模型代理")、算法[3](https://arxiv.org/html/2310.01444v3#alg3 "算法 3 ‣ A.1 通信模式 ‣ 附录 A ‣ 在通信中通过普适反馈调整大语言模型代理")）来演示三种通信模式如何帮助代理收集探索数据。

[⬇](data:text/plain;base64,IyBhZ2VudDE6IExMYU1BIGFnZW50CiMgYWdlbnQyOiBHUFQtNCBhZ2VudAojIGlucHV0OiAgVGFzayBkZXNjcmlwdGlvbgojIG91dHB1dDogUyAgPSAoVCwgTSwgUikKCiMgaW5pdGlhbGl6YXRpb24KVCwgTSwgUiA9IFtpbnB1dF0sIFswXSwgWzBdCgppID0gMAp3aGlsZSBpIDwgbWF4X3N0ZXBzOgogICAgVCArPSBbInRoaW5rOiJdCiAgICB0aG91Z2h0ID0gYWdlbnQyLmFwaShUKQogICAgVC5hcHBlbmQodGhvdWdodCkKICAgIE0uYXBwZW5kKDIpICMgdGVhY2hlciBhZ2VudCBtZXNzYWdlIG1hc2sKICAgIFIuYXBwZW5kKDApCgoKICAgIFQgKz0gWyJhY3Q6Il0KICAgIGFjdGlvbiA9IGFnZW50MS5hcGkoVCkKICAgIFQuYXBwZW5kKGFjdGlvbikKICAgIE0uYXBwZW5kKDEpICMgc3R1ZGVudCBhZ2VudCBtZXNzYWdlIG1hc2sKICAgIFIuYXBwZW5kKDApCgogICAgcmVzcG9uc2UgPSBlbnYuZXhjdXRlKGFjdGlvbikKICAgIHJld2FyZCA9IHBhcnNlKHJlc3BvbnNlKQogICAgVC5hcHBlbmQocmVzcG9uc2UpCiAgICBNLmFwcGVuZCgwKSAjIHN5c3RlbSBtZXNzYWdlIG1hc2sKICAgIFIuYXBwZW5kKHJld2FyZCkKCiAgICBpICs9IDEKICAgIGlmIHJld2FyZCAhPSAwOgogICAgICAgIGJyZWFrClMgID0gKFQsIE0sIFIpCnJldHVybiBT)#  agent1:  LLaMA  agent#  agent2:  GPT-4  agent#  input:  Task  description#  output:  S  =  (T,  M,  R)#  initializationT,  M,  R  =  [input],  [0],  [0]i  =  0while  i  <  max_steps:T  +=  ["think:"]thought  =  agent2.api(T)T.append(thought)M.append(2)  #  teacher  agent  message  maskR.append(0)T  +=  ["act:"]action  =  agent1.api(T)T.append(action)M.append(1)  #  student  agent  message  maskR.append(0)response  =  env.excute(action)reward  =  parse(response)T.append(response)M.append(0)  #  system  message  maskR.append(reward)i  +=  1if  reward  !=  0:breakS  =  (T,  M,  R)return  S

算法 2 用于演示对话模式的 Python 风格算法

[⬇](data:text/plain;base64,IyBhZ2VudDE6IExMYU1BIGFnZW50CiMgYWdlbnQyOiBHUFQtNCBhZ2VudAojIGlucHV0OiAgUXVlc3Rpb24gZGVzY3JpcHRpb24KIyBvdXRwdXQ6IFMgID0gKFQsIE0sIFIpCgojIGluaXRpYWxpemF0aW9uClQsIE0sIFIgPSBbaW5wdXRdLCBbMF0sIFswXQoKaSA9IDAKd2hpbGUgaSA8IG1heF9zdGVwczoKICAgIFQgKz0gWyJhbnN3ZXIgdGhlIHF1ZXN0aW9uIHN0ZXAgYnkgc3RlcDoiXQogICAgYW5zd2VyMSA9IGFnZW50MS5hcGkoVCkKICAgIHF1ZXJ5ID0gVCArIGFuc3dlcjEgKyBbInRoZSBhbnN3ZXIgaXMgY29ycmVjdCwgeWVzIG9yIG5vPyBhbHNvIGdpdmVzIGEgYmV0dGVyIGFuc3dlciJdCiAgICByZXNwb25zZSA9IGFnZW50Mi5hcGkocXVlcnkpCiAgICByZXdhcmQsIGFuc3dlcjIgPSBwYXJzZShyZXNwb25zZSkKICAgIFQuYXBwZW5kKGFuc3dlcjEpCiAgICBULmFwcGVuZChhbnN3ZXIyKQogICAgTS5hcHBlbmQoMSkgIyBzdHVkZW50IGFnZW50IG1lc3NhZ2UgbWFzawogICAgTS5hcHBlbmQoMikgIyB0ZWFjaGVyIGFnZW50IG1lc3NhZ2UgbWFzawogICAgUi5hcHBlbmQocmV3YXJkKQogICAgUi5hcHBlbmQoKzEpICMgYXNzdW1lIHRlYWNoZXIgaXMgY29ycmVjdAoKCiAgICBxdWVyeSA9IHF1ZXJ5ICsgcmVzcG9uc2UgKyBbInBsZWFzZSBnZW5lcmF0ZSBhIHNpbWlsYXIgcWEgcGFpciB0byB0ZWFjaCB0aGUgc3R1ZGVudDoiXQogICAgcmVzcG9uc2UgPSBhZ2VudDIuYXBpKHF1ZXJ5KQogICAgbmV3X3F1ZXN0aW9uLCB0ZWFjaGVyX2Fuc3dlciA9IHBhcnNlKHJlc3BvbnNlKQogICAgbmV3X3F1ZXN0aW9uICs9ICJhbnN3ZXIgdGhlIHF1ZXN0aW9uIHN0ZXAgYnkgc3RlcDoiCiAgICBzdHVkZW50X2Fuc3dlciA9IGFnZW50MS5hcGkobmV3X3F1ZXN0aW9uKQogICAgcmV3YXJkID0gcGFyc2Uoc3R1ZGVudF9hbnN3ZXIsIHRlYWNoZXJfYW5zd2VyKQogICAgVC5hcHBlbmQobmV3X3F1ZXN0aW9uICsgc3R1ZGVudF9hbnN3ZXIpCiAgICBNLmFwcGVuZCgxKSAjIHN0dWRlbnQgYWdlbnQgbWVzc2FnZSBtYXNrCiAgICBSLmFwcGVuZChyZXdhcmQpCgogICAgaSArPSAxCgpTICA9IChULCBNLCBSKQpyZXR1cm4gUw==)#  agent1:  LLaMA  agent#  agent2:  GPT-4  agent#  input:  Question  description#  output:  S  =  (T,  M,  R)#  initializationT,  M,  R  =  [input],  [0],  [0]i  =  0while  i  <  max_steps:T  +=  ["answer␣the␣question␣step␣by␣step:"]answer1  =  agent1.api(T)query  =  T  +  answer1  +  ["the␣answer␣is␣correct,␣yes␣or␣no?␣also␣gives␣a␣better␣answer"]response  =  agent2.api(query)reward,  answer2  =  parse(response)T.append(answer1)T.append(answer2)M.append(1)  #  student  agent  message  maskM.append(2)  #  teacher  agent  message  maskR.append(reward)R.append(+1)  #  assume  teacher  is  correctquery  =  query  +  response  +  ["please␣generate␣a␣similar␣qa␣pair␣to␣teach␣the␣student:"]response  =  agent2.api(query)new_question,  teacher_answer  =  parse(response)new_question  +=  "answer␣the␣question␣step␣by␣step:"student_answer  =  agent1.api(new_question)reward  =  parse(student_answer,  teacher_answer)T.append(new_question  +  student_answer)M.append(1)  #  student  agent  message  maskR.append(reward)i  +=  1S  =  (T,  M,  R)return  S

算法 3 用 Python 风格的算法展示类比模式

### A.2 缓冲区结构

通信数据将保存为回放缓冲区，以便更新阶段使用，缓冲区的数据格式是一个 token 序列，如图 [2](https://arxiv.org/html/2310.01444v3#S2.F2 "图 2 ‣ 2.1 指令微调 ‣ 2 相关工作 ‣ 通过通用反馈适配 LLM 代理") 所示。我们将每个 token 视为强化学习公式中的动作单元，每个探索轨迹被处理为 5 个数据序列 $[\textbf{S}_{\textbf{a}},\textbf{S}_{\textbf{m}},\textbf{S}_{\textbf{v}},% \textbf{S}_{\textbf{l}},\textbf{S}_{\textbf{r}}]$：

+   •

    $\textbf{S}_{\textbf{a}}$: 一个整数列表，表示由分词器编码生成的 token ids。所有有效的文本轨迹都作为队列记录下来，包括系统文本（如环境描述、反馈）和代理文本（如解析的动作、思维过程和其他代理的提示）。无效的代理生成文本将被跳过，例如无意义的字符串和无法解析的动作文本。这些 token 被视为 LLM 的输入，但它们有不同的掩码以应用不同的损失。

+   •

    $\textbf{S}_{\textbf{m}}$: 系统掩码，用于掩盖不同类型的输入 token，以控制训练损失。我们将 0 作为默认掩码，应用于系统文本，如环境描述、系统反馈和系统提示，这些文本编码的动作不是我们希望代理学习的动作，因此它们将在 PPO 算法中被掩盖，既不会计算策略损失，也不会计算价值损失。我们将 1 作为代理生成的 token 掩码，应用于决策关键词和思维过程，它们是我们强化学习流程的主要监督对象，因此它们将被赋予完整的策略损失和价值损失。我们将 2 作为来自其他代理的提示或反馈的掩码，这些动作我们同样希望我们的代理学习，但没有即时的状态值，因为它们不是由我们的代理生成的。因此，掩码为 2 的 token 只会被掩盖价值损失，并通过策略损失进行监督。

+   •

    $\textbf{S}_{\textbf{v}}$: 由代理模型的价值头获得的对应于动作的状态值。价值头是插入到原始预训练 LLM 架构中的一层，我们通过在倒数第二个 LlamaDecoderLayer 后插入一个线性层来实现它，作为辅助输出模块，输出值通过 $tanh()$ 函数处理以保持其在 $(-1,1)$ 范围内。

+   •

    $\textbf{S}_{\textbf{r}}$: 对应于动作的奖励。奖励非常稀疏，大多数动作的奖励为零，只有当前任务完成或当前缓冲区的 token 长度刚好溢出时，奖励才会变为非零值：+1 表示正奖励，-1 表示负奖励。

### A.3 LTC 算法

[⬇](data:text/plain;base64,IyBhZ2VudDogUHJlLXRyYWluZWQgTExNIGFnZW50CiMgbl9ncHU6IHRvdGFsIG51bWJlciBvZiBHUFVzCiMgZW52X2NsczogdGhlIGNsYXNzIG9mIGVudmlyb25tZW50cwojIG5fZ2VuOiB0aGUgZ2VuZXJhdGlvbiBzaXplIGZvciBvbmUgaXRlcmF0aW9uCiMgbl90cmFpbjogdGhlIHRyYWluIHNpemUgZm9yIG9uZSBpdGVyYXRpb24KCiMgaW5pdGlhbGl6YXRpb24KYWdlbnQgPSBpbnN0cnVjdGlvbl9maW5ldHVuZShhZ2VudCkKcmVwbGF5X2J1ZmZlciA9IFtdCmkgPSAwCndoaWxlIGkgPCBtYXhfaXRlcmF0aW9uOgogIGkgKz0gMQogICMgRXhwbG9yYXRpb24gUGhhc2UKICBlbnZzID0gZW52X2NscyhzYW1wbGUoZGF0YSwgbl9nZW4vL25fZ3B1KSkKICAjIGFzeW5jaHJvbm91c2x5IGdlbmVyYXRlCiAgbmV3X2J1ZmZlciA9IGdlbmVyYXRlX3RyaWFscyhhZ2VudCwgZW52cykKICAjIGRpc3QuZ2F0aGVyIGFuZCBkaXN0LmJyb2FkY2FzdAogIG5ld19idWZmZXIgPSBzeW5jX2FsbF9ncHVzKG5ld19idWZmZXIpCiAgcmVwbGF5X2J1ZmZlci5hcHBlbmQobmV3X2J1ZmZlcikKCiAgIyBUcmFpbmluZyBQaGFzZQogIHJvbGxvdXRzID0gc2FtcGxlKHJlcGxheV9idWZmZXIsIG5fdHJhaW4pKQogICMgZGlzdHJpYnV0ZWQgdHJhaW5pbmcgd2l0aCBwcG8KICBhZ2VudCA9IHBwb19kZHBfdHJhaW4oYWdlbnQsIHJvbGxvdXRzKQ==)#  agent:  Pre-trained  LLM  agent#  n_gpu:  total  number  of  GPUs#  env_cls:  the  class  of  environments#  n_gen:  the  generation  size  for  one  iteration#  n_train:  the  train  size  for  one  iteration#  initializationagent  =  instruction_finetune(agent)replay_buffer  =  []i  =  0while  i  <  max_iteration:i  +=  1#  Exploration  Phaseenvs  =  env_cls(sample(data,  n_gen//n_gpu))#  asynchronously  generatenew_buffer  =  generate_trials(agent,  envs)#  dist.gather  and  dist.broadcastnew_buffer  =  sync_all_gpus(new_buffer)replay_buffer.append(new_buffer)#  Training  Phaserollouts  =  sample(replay_buffer,  n_train))#  distributed  training  with  ppoagent  =  ppo_ddp_train(agent,  rollouts)

算法 4 Python 风格代码的 LTC

LTC 的实现可以总结为算法 [4](https://arxiv.org/html/2310.01444v3#alg4 "Algorithm 4 ‣ A.3 Algorithm of LTC ‣ Appendix A Appendix ‣ Adapting LLM Agents with Universal Feedback in Communication")，我们揭示了体现“通过交流学习”（LTC）范式的结构框架，精心设计以促进通过迭代周期实现自主和渐进的学习。在开始阶段，预训练的大型语言模型（LLM）代理经历了微调阶段，将其初始配置与预定的学习任务对齐。随后，开始了探索阶段，在该阶段从一批训练环境中进行采样，代理随后利用并行计算的力量，通过多 GPU 异步生成试验数据，以提高效率。生成的这些新数据会在所有 GPU 之间进行同步，以促进一致的学习基础，并存储在重放缓冲区中以供进一步使用。随着代理在此过程中不断迭代，它在更新阶段持续从该重放缓冲区进行采样，使用近端策略优化（PPO）算法在分布式数据并行（DDP）设置中完善其策略并进行动态适应。因此，这段代码片段总结了 LTC 范式创新性的异步和分布式特性，标志着在推动智能、适应性强的协作人工智能代理方面的重要进展。

图 7：GPT-4 可以通过捷径解决问题，而 LLaMA-7B 代理无法模仿这一点。

### A.4 训练损失

在每次迭代后的探索阶段，我们通过将新收集的轨迹纳入重放缓冲区来更新该缓冲区，然后从中采样最新的轨迹来训练 LLM 代理的参数 $\pi_{\theta}$。我们设计了一个训练目标，结合了以下内容：1）标准语言模型损失 $\mathcal{L}_{\text{LM}}$，2）策略损失 $\mathcal{L}_{\text{policy}}$，3）价值损失 $\mathcal{L}_{\text{value}}$，以及 4）熵损失 $\mathcal{L}_{\text{entropy}}$。整体训练目标可以表示为：

|  | $\displaystyle\mathcal{L}_{\text{total}}$ | $\displaystyle=\mathcal{L}_{\text{LM}}+\beta(\mathcal{L}_{\text{policy}}+% \lambda\mathcal{L}_{\text{value}}+\mathcal{L}_{\text{entropy}})$ |  |
| --- | --- | --- | --- |

其中 $\beta$ 和 $\lambda$ 是加权超参数。

上述不同的损失函数描述如下：

+   •

    语言模型损失 $\mathcal{L}_{\text{LM}}$ 被定义为代理与其生成内容之间的交叉熵，这些内容具有正向奖励，类似于自我提升的模型方案 Huang 等人（[2022a](https://arxiv.org/html/2310.01444v3#bib.bib16)）；Rafailov 等人（[2023](https://arxiv.org/html/2310.01444v3#bib.bib39)）。通过在这些生成内容上进行训练，代理被进一步鼓励生成能够带来正向奖励的内容。

+   •

    策略损失 $\mathcal{L}_{\text{policy}}$ 被引入以监督代理的行为。策略损失 $\mathcal{L}_{\text{policy}}$ 是使用 Schulman 等人（[2017](https://arxiv.org/html/2310.01444v3#bib.bib45)）中定义的替代目标的屏蔽版本计算的，并使用优势估计 $\hat{A}$，

    |  | $\mathcal{L}_{\text{policy}}(\theta)=-\mathbb{E}[m_{\text{policy}}*\text{min}(r(\theta)\hat{A},\text{clip}(r(\theta),1-\epsilon,1+\epsilon)\hat{A}],$ |  |
    | --- | --- | --- |

    其中 $r(\theta)$ 是代理与其先前版本 $\pi_{\text{old}}$ 的输出概率比率 $r(\theta)=\frac{\pi_{\theta}(a|s)}{\pi_{\text{old}}(a|s)}$。我们定义了二进制掩码 $m_{\text{policy}}$ 用来在 PPO 损失中屏蔽编码的系统消息（在缓冲区 [A.2](https://arxiv.org/html/2310.01444v3#A1.SS2 "A.2 Buffer Structure ‣ Appendix A Appendix ‣ Adapting LLM Agents with Universal Feedback in Communication") 中标记为 $S_{m}=0$）。例如，假设 $\{x_{1},y_{1},x_{2},y_{2},\dots x_{n},y_{n}\}$ 是一个由系统消息 $x_{n}\in X$ 和代理消息（包括目标训练代理及其他教师代理）$\pi_{\theta}$ 输出 $y_{n}\in Y$ 组成的标记缓冲区，则二进制掩码为 $m_{\text{policy}}=\{0,1,0,1,\dots 0,1\}$。

+   •

    值损失在 Schulman 等人（[2017](https://arxiv.org/html/2310.01444v3#bib.bib45)）的研究中被定义为计算值和估计优势之间的均方误差，该误差由另一个二进制掩码 $m_{\text{value}}$ 屏蔽（在缓冲区 [A.2](https://arxiv.org/html/2310.01444v3#A1.SS2 "A.2 Buffer Structure ‣ Appendix A Appendix ‣ Adapting LLM Agents with Universal Feedback in Communication") 中标记为 $S_{m}=1$）。例如，假设 $\{z_{1},y_{1},z_{2},y_{2},\dots z_{n},y_{n}\}$ 是一个由所有其他消息（除了代理生成的消息）$z_{n}\in X$ 和训练代理生成的消息 $\pi_{\theta}$ 输出 $y_{n}\in Y$ 组成的标记缓冲区，则二进制掩码为 $m_{\text{policy}}=\{0,1,0,1,\dots 0,1\}$。

+   •

    $\mathcal{L}_{\text{entropy}}$ 是一种熵奖励，用于确保足够的探索性，如过去的研究所建议（Williams, [1992](https://arxiv.org/html/2310.01444v3#bib.bib66); Mnih et al., [2016](https://arxiv.org/html/2310.01444v3#bib.bib29)）。这个熵是作为一个小的负因子与策略分布的熵相乘来计算的：$\mathcal{L}_{\text{entropy}}=0.01\times\sum_{a}\pi_{\theta}(a|s)\log\pi_{\theta}(a|s)$。

### A.5 实现细节

### A.6 异步分布式生成

探索数据是以异步方式生成的，因此代理可以处理具有开放式探索空间的环境。训练数据经过预处理，转化为交互式环境，代理能够在这些环境中观察状态、执行动作并获取即时反馈。根据GPU线程的数量，这些环境被划分为相应的部分，然后分配给每个GPU。随后，这些GPU开始与使用最新数据训练的相同代理异步并行地探索这些环境。由于生成的内容长度各异，并且环境内部的交互通常是开放式的，代理探索每个环境的时间成本也有所不同，一些GPU线程可能比其他线程处理数据更快。为所有GPU线程设置一个屏障，使得较早完成的GPU线程可以等待其他线程，直到环境生成的总累积缓冲区达到预设的数量$S_{g}$，即我们希望在一次迭代中添加到回放缓冲区的新训练缓冲区数量。在所有GPU线程都到达屏障后，我们将获得足够的缓冲区，然后从每个GPU线程收集缓冲区并将其合并，再将新缓冲区广播到每个GPU线程，以更新它们的本地回放缓冲区。更新后的回放缓冲区将在更新阶段用于训练下一次迭代的代理。

### A.7 基准线

ReAct（姚等，[2023](https://arxiv.org/html/2310.01444v3#bib.bib71)）使用训练案例的子集作为不同任务的提示，采用思维-行动-观察序列的格式。对于像HotpotQA这样的知识密集型推理任务，ReAct设计了一个包括搜索、查找和完成动作的动作空间，使得智能体能够与维基百科互动，检索必要的信息。另一方面，ReAct-IM采用了内心独白（IM）（黄等，[2022b](https://arxiv.org/html/2310.01444v3#bib.bib17)）风格的提示。思维链提示（CoT）（魏等，[2022b](https://arxiv.org/html/2310.01444v3#bib.bib64)）通过生成一系列中间推理步骤来增强语言和视觉模型（LLM）的推理能力。这可以看作是ReAct的仅推理基线，排除了动作和观察。此外，CoT-SC（王等，[2022c](https://arxiv.org/html/2310.01444v3#bib.bib58); [d](https://arxiv.org/html/2310.01444v3#bib.bib59)）是CoT的后续工作，作为自一致性基线。值得注意的是，这些方法中的大多数采用贪婪解码，除了BUTLER Micheli & Fleuret（[2021](https://arxiv.org/html/2310.01444v3#bib.bib28)），它使用了束搜索。大多数这些方法集中于少量样本提示，并使用不同的预训练模型。为了确保公平比较，我们通过使用收集的轨迹作为微调数据，微调LLaMA-7B模型，纳入了名为ReAct-Tuning和CoT-Tuning的额外基线，如在[4.2](https://arxiv.org/html/2310.01444v3#S4.SS2.SSS0.Px2 "Agent Pre-training ‣ 4.2 Settings ‣ 4 Experiments ‣ Adapting LLM Agents with Universal Feedback in Communication")中所述。此外，GPT-4在测试时未被使用，所有报告的结果均由训练后的智能体自行获得。

### A.8 损失

我们对LTC的损失设计进行了消融研究。图 [6](https://arxiv.org/html/2310.01444v3#S5.F6 "Figure 6 ‣ Efficiency ‣ 5 Discussion ‣ Adapting LLM Agents with Universal Feedback in Communication")展示了在不同损失设置下，智能体在ALFWorld数据集上的成功率。没有使用我们的通信模式进行交互，仅仅通过采样预先收集的指令数据进行训练时，性能提升有限。然而，当我们将通信模式纳入数据收集时，模型的表现迅速突破了80%。此外，采用PPO损失来分别处理正负样本，使得改进更快且更显著（蓝线）。在图 [6](https://arxiv.org/html/2310.01444v3#S5.F6 "Figure 6 ‣ Efficiency ‣ 5 Discussion ‣ Adapting LLM Agents with Universal Feedback in Communication")中，我们展示了训练过程中三种主要损失的独立曲线。最初，LM损失呈下降趋势。有趣的是，随着训练迭代的进行，价值损失和策略损失逐渐减少，这可能导致LM损失暂时增加。在价值损失和策略损失达到一定阈值后，LM损失继续下降直到收敛。

![请参阅标题](img/9e38aeaf39c786275d4233b41f8c604c.png)

图8：ALFWorld单口模式的玩具示例文本版本，见图 [1](https://arxiv.org/html/2310.01444v3#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Adapting LLM Agents with Universal Feedback in Communication")。

### A.9 案例研究

一个有趣的观察是，GPT-4智能体有时在充当教师生成新的训练数据时，会采用“捷径”来解决问题。这些捷径依赖于它在预训练过程中获得的内部知识。为了说明这一点，我们展示了图 [7](https://arxiv.org/html/2310.01444v3#A1.F7 "Figure 7 ‣ A.3 Algorithm of LTC ‣ Appendix A Appendix ‣ Adapting LLM Agents with Universal Feedback in Communication")中的HotpotQA案例研究。在这个案例中，GPT-4智能体通过利用它记忆中的第二个条目知识，快速检索出答案。在接收到第一个条目的Wikipedia页面后，GPT-4迅速得出答案。另一方面，图 [7](https://arxiv.org/html/2310.01444v3#A1.F7 "Figure 7 ‣ A.3 Algorithm of LTC ‣ Appendix A Appendix ‣ Adapting LLM Agents with Universal Feedback in Communication")底部展示了与LLaMA-7B的对比，后者通过我们的方法LTC训练，并且GPT-4智能体参与其中。LLaMA-7B没有采用捷径，而是执行了对第二个条目的搜索。这个案例研究表明，LTC中的通信机制相比仅依赖GPT-4生成的数据，在学习过程中提供了额外的益处。
