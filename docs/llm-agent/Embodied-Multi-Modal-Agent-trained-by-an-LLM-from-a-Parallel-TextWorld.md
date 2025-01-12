<!--yml

类别：未分类

日期：2025-01-11 13:01:24

-->

# 由平行文本世界中的LLM训练的具象多模态代理

> 来源：[https://arxiv.org/html/2311.16714/](https://arxiv.org/html/2311.16714/)

\newfloatcommand

capbtabboxtable[][\FBwidth] \newfloatcommandcapbfigboxfigure[][\FBwidth]

杨奕君^(1,5,4)  周天怡²  李康学³  陶大鹏³  李路松⁴

李申^(4,∗)  贺晓东⁴  姜静⁵  施玉辉^(1,)

¹南方科技大学  ²马里兰大学帕克分校

³云南大学  ⁴京东探索学院  ⁵悉尼科技大学

[https://github.com/stevenyangyj/Emma-Alfworld](https://github.com/stevenyangyj/Emma-Alfworld) 通讯作者

###### 摘要

虽然大型语言模型（LLMs）在文本模拟世界中表现优异，但它们在没有其他模态感知（如视觉或听觉信号）的情况下，与更现实的世界互动时会遇到困难。尽管视觉-语言模型（VLMs）集成了与静态图像特征对齐的LLM模块（1）以及（2）可能具备世界动态的先验知识（如在文本世界中所展示的），它们并未在具象的视觉世界中进行训练，因此无法与其动态进行对齐。另一方面，在没有专家指导的情况下，在一个嘈杂的视觉世界中训练一个具象代理往往具有挑战性且效率低下。本文中，我们训练了一个生活在视觉世界中的VLM代理，利用一个在平行文本世界中表现出色的LLM代理。具体来说，我们通过分析错误来提炼LLM在文本世界任务中的反思结果（改进的行动），以对视觉世界中的相同任务进行微调，从而使具象的多模态代理（EMMA）能够迅速适应视觉世界的动态。通过一种新颖的DAgger-DPO算法实现了两个平行世界之间的跨模态模仿学习，使得EMMA能够在没有LLM专家进一步指导的情况下推广到广泛的新任务。对ALFWorld基准中多样化任务的广泛评估表明，EMMA在成功率上比基于SOTA VLM的代理提高了20%-70%。

## 1 引言

具身的多模态智能体被认为是实现人工通用智能（AGI）的关键一步，因为它们具备广泛智能活动的潜力[[63](https://arxiv.org/html/2311.16714v2#bib.bib63), [27](https://arxiv.org/html/2311.16714v2#bib.bib27)]。基础模型的兴起为构建此类智能体带来了希望[[32](https://arxiv.org/html/2311.16714v2#bib.bib32), [34](https://arxiv.org/html/2311.16714v2#bib.bib34), [11](https://arxiv.org/html/2311.16714v2#bib.bib11), [76](https://arxiv.org/html/2311.16714v2#bib.bib76), [38](https://arxiv.org/html/2311.16714v2#bib.bib38), [71](https://arxiv.org/html/2311.16714v2#bib.bib71), [35](https://arxiv.org/html/2311.16714v2#bib.bib35)]，并且社区中的显著努力已尝试在许多决策场景中加以利用，例如自动驾驶[[15](https://arxiv.org/html/2311.16714v2#bib.bib15), [66](https://arxiv.org/html/2311.16714v2#bib.bib66)]、日常家务机器人[[13](https://arxiv.org/html/2311.16714v2#bib.bib13), [6](https://arxiv.org/html/2311.16714v2#bib.bib6), [77](https://arxiv.org/html/2311.16714v2#bib.bib77)]，以及复杂的操作任务[[25](https://arxiv.org/html/2311.16714v2#bib.bib25), [42](https://arxiv.org/html/2311.16714v2#bib.bib42), [57](https://arxiv.org/html/2311.16714v2#bib.bib57), [29](https://arxiv.org/html/2311.16714v2#bib.bib29)]。

LLMs可以作为反射型智能体，通过对世界状态和文本动作的语言描述与文本世界进行交互。此外，它们在规划、反思和奖励塑造方面展现了巨大的潜力。这主要得益于它们对世界的先验知识和语义抽象。然而，基于LLM的智能体无法直接应用于视觉世界。尽管当前的视觉-语言模型（VLMs）尝试将LLM与视觉模态对齐，但它们的预训练仅专注于图像-文本对之间的静态对齐，因此生成的智能体与视觉世界的动态性未能很好地对齐。如图LABEL:fig:teaser（a）所示，即便是特权VLM，例如GPT-4V[[43](https://arxiv.org/html/2311.16714v2#bib.bib43)]，也未能在具身的ALFWorld环境中完成任务[[56](https://arxiv.org/html/2311.16714v2#bib.bib56)]。在这种零-shot设置下，GPT-4V主要依赖于其语言先验，来推测当前步骤中检测到的这些物体，而不是基于任务指令对视觉输入和环境动态之间的对齐进行推理。

在本文中，我们研究了如何通过将 VLM（视觉语言模型）与视觉世界动态对齐，并提取 LLM（大型语言模型）代理在平行文本世界中的技能，来训练一个具身代理。我们的总体目标是构建一个具身多模态代理（EMMA），使其能够接收文本任务指令（例如来自人类用户）和每一步的像素状态观察，进而产生一系列行动，最终高效地完成任务。这是一个具有挑战性的问题，原因包括：(1) 任务奖励的稀疏性[[2](https://arxiv.org/html/2311.16714v2#bib.bib2), [14](https://arxiv.org/html/2311.16714v2#bib.bib14), [3](https://arxiv.org/html/2311.16714v2#bib.bib3), [4](https://arxiv.org/html/2311.16714v2#bib.bib4)]，(2) 噪声干扰的视觉表示，(3) VLM的幻觉现象[[60](https://arxiv.org/html/2311.16714v2#bib.bib60)]，以及(4) VLM静态表示与视觉世界动态的错位。虽然通过离线蒸馏和模仿来自强大LLM代理在平行文本世界中的任务可以在一定程度上克服前两者的挑战[[75](https://arxiv.org/html/2311.16714v2#bib.bib75)]，但有效地缓解剩余挑战则需要在交互式视觉世界中对VLM代理进行在线微调[[31](https://arxiv.org/html/2311.16714v2#bib.bib31), [70](https://arxiv.org/html/2311.16714v2#bib.bib70), [18](https://arxiv.org/html/2311.16714v2#bib.bib18)]。

为此，我们通过模仿学习对VLM代理进行微调，模仿的是在平行文本世界中启动的LLM专家（例如，基于ChatGPT构建的专家）在相同任务上的表现。具体来说，在每一步EMMA与视觉世界交互时，我们将其视觉观察转化为等效的文本描述，并发送给LLM代理，后者为EMMA生成一个动作供其模仿。这种跨模态交互式模仿学习基于DAgger算法[[51](https://arxiv.org/html/2311.16714v2#bib.bib51)]，能够克服行为克隆（BC）导致的累积误差和分布偏移。如图Fig. LABEL:fig:teaser(b)所示，通过BC对170K专家演示数据进行微调的InstructBLIP[[11](https://arxiv.org/html/2311.16714v2#bib.bib11)]代理，仍然无法基于视觉观察采取正确的行动。我们进一步改进了DAgger的目标，使其成为直接偏好优化（DPO）[[48](https://arxiv.org/html/2311.16714v2#bib.bib48)]，即在每一步交互中，最大化LLM专家的动作（正面）相对于VLM学生动作（负面）的偏好。为了收集更好的教学信号，回溯到VLM学生的动作，LLM专家由LLM演员和LLM评论家组成，前者被提示输出专家动作，后者则被提示对VLM代理的历史轨迹进行反思反馈。我们保持一个长期记忆，存储这些反馈，并用于引导LLM演员改进未来模仿的动作。

图 1：为两个并行世界生成的任务示例。一个 VLM 代理在视觉世界中，一个 LLM 代理在文本世界中，作为家用机器人，被指示清洁一个苹果并将其放入冰箱。点击放大查看更多细节。

我们评估了 EMMA，并将其与部署在 ALFWorld 基准上的仅视觉代理、LLM 代理和 VLM 代理进行了比较 [[56](https://arxiv.org/html/2311.16714v2#bib.bib56)]，该基准包含了视觉和文本环境中的多个任务。广泛的评估表明，EMMA 在仅视觉环境中，相较于最先进的（SOTA）VLM 代理，成功率提高了 20%-70%。此外，EMMA 是唯一能够在开放词汇和自由形式测试任务中进行泛化的 VLM 代理，为如何利用 LLM 反馈训练更具多样性和可泛化性的具身代理在多模态环境中的应用提供了新颖的见解。

图 2：通过跨模态模仿学习，由 LLM 专家训练的具身多模态代理（EMMA）。EMMA 每一步的输入状态包括文本任务指令和像素观测，用 VLM 生成一系列动作。然后，我们将每个像素观测转换为文本等效内容，作为 LLM 专家的上下文，以生成 EMMA 模仿的改进动作。

## 2 具身多模态代理

图 [2](https://arxiv.org/html/2311.16714v2#S1.F2 "Figure 2 ‣ 1 Introduction ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld") 展示了我们“具身多模态代理（EMMA）”的主要思想，详细的训练过程见算法 [1](https://arxiv.org/html/2311.16714v2#alg1 "Algorithm 1 ‣ 2.3 Training EMMA via Cross-Modality Imitation ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")。该代理基于模块化的 VLM，能够通过像素观测和文本指令与环境互动。为了克服训练 EMMA 中的挑战，例如稀疏奖励或分布漂移，我们探索从并行文本世界构建 LLM 专家（见章节 [2.2](https://arxiv.org/html/2311.16714v2#S2.SS2 "2.2 LLM Expert from a Parallel TextWorld ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")）来为 EMMA 提供逐步指导。最后，章节 [2.3](https://arxiv.org/html/2311.16714v2#S2.SS3 "2.3 Training EMMA via Cross-Modality Imitation ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld") 进一步讨论如何利用 LLM 专家通过跨模态模仿学习训练 EMMA。

### 2.1 EMMA 在视觉世界中的表现

在视觉环境中，EMMA $\pi_{\theta}$ 旨在处理任务指令 $x_{\text{task}}$（例如，由人类用户提供）和每个交互步骤 $t$ 的像素观察值 $s_{v}^{t}$。它期望生成一系列高层次的文本动作 $\{x_{a}^{t}\sim\pi_{\theta}(\cdot|x_{\text{task}},s_{v}^{t})\}_{t=0}^{T}$¹¹1为了简洁起见，本文接下来的部分中将省略 $x_{\text{task}}$。以高效完成任务。为此，我们从近期大型预训练视觉语言模型（VLMs）的进展中获得灵感[[11](https://arxiv.org/html/2311.16714v2#bib.bib11)，[34](https://arxiv.org/html/2311.16714v2#bib.bib34)，[76](https://arxiv.org/html/2311.16714v2#bib.bib76)，[38](https://arxiv.org/html/2311.16714v2#bib.bib38)，[32](https://arxiv.org/html/2311.16714v2#bib.bib32)]，并将EMMA的架构模块化为四个部分：（1）一个ViT用于将 $s_{v}$ 编码为视觉嵌入，（2）一个查询变换器（Q-Former），旨在通过视觉嵌入与查询标记之间的跨注意力来提取最相关的视觉特征，（3）一个线性投影层，用于将视觉特征与文本嵌入对齐，（4）一个LLM解码器，采用指令标记和线性投影层输出的连接，逐步自回归地生成动作 $x_{a}$。为了减少计算开销并防止灾难性遗忘[[37](https://arxiv.org/html/2311.16714v2#bib.bib37)]，我们采用了从InstructBLIP[[11](https://arxiv.org/html/2311.16714v2#bib.bib11)]预训练的ViT、Q-Former和LLM，并在微调阶段将它们保持冻结。我们仅更新线性投影层，如图[2](https://arxiv.org/html/2311.16714v2#S1.F2 "Figure 2 ‣ 1 Introduction ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")所示。如此模块化的架构使得EMMA能够灵活且计算高效地整合任何现有的预训练视觉模型和LLM。

然而，将EMMA部署到复杂的视觉世界中仍然面临挑战。主要的障碍之一是直接使用任何预训练的视觉语言模型（VLM）并不理想，因为现有的预训练仅关注图像-文本对之间的静态对齐[[34](https://arxiv.org/html/2311.16714v2#bib.bib34), [11](https://arxiv.org/html/2311.16714v2#bib.bib11), [32](https://arxiv.org/html/2311.16714v2#bib.bib32)]，因此生成的智能体可能难以推理世界的动态。正如在第[1](https://arxiv.org/html/2311.16714v2#S1 "1 Introduction ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")节中讨论的那样，甚至是最先进的VLM，即GPT-4V，在具身ALFWorld环境中也未能完成任务[[56](https://arxiv.org/html/2311.16714v2#bib.bib56)]。在这种零-shot设置下，GPT-4V往往依赖于当前检测到的物体的语言先验，而不是给定的任务指令和环境的潜在动态来指导互动。此外，在预先收集的示范数据集上微调预训练VLM也并非最佳选择，因为环境和任务的多样性[[77](https://arxiv.org/html/2311.16714v2#bib.bib77)]，缺乏大规模专家注释[[44](https://arxiv.org/html/2311.16714v2#bib.bib44)]，以及分布转移问题所带来的挑战[[31](https://arxiv.org/html/2311.16714v2#bib.bib31)]。一个看似自然的解决方案是通过环境反馈的强化学习（RLEF）[[68](https://arxiv.org/html/2311.16714v2#bib.bib68)]，其中奖励信号依赖于将任务分解为一系列合理的子目标并检查其完成情况。然而，在现实世界的场景中，大多数子目标无法精确地定义或描述，因此奖励是稀疏的；因此，我们不期望RLEF是有效的。

为此，我们提出利用互动模仿学习（IL）来将EMMA与任何环境的动态对齐，但这会带来两个关键的算法挑战：(1) 如何获得一个高质量、可访问和可扩展的专家，供EMMA在IL过程中查询（第[2.2](https://arxiv.org/html/2311.16714v2#S2.SS2 "2.2 LLM Expert from a Parallel TextWorld ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")节）？(2) 如何设计有效的策略，在复杂、多样化且可能是开放式的环境中利用这个专家训练EMMA（第[2.3](https://arxiv.org/html/2311.16714v2#S2.SS3 "2.3 Training EMMA via Cross-Modality Imitation ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")节）。

### 2.2 来自平行文本世界的LLM专家

得益于一系列在上下文学习领域中的提示技术，如链式思维[[62](https://arxiv.org/html/2311.16714v2#bib.bib62)]、树状思维[[72](https://arxiv.org/html/2311.16714v2#bib.bib72)]、ReAct[[73](https://arxiv.org/html/2311.16714v2#bib.bib73)]和Reflexion[[54](https://arxiv.org/html/2311.16714v2#bib.bib54)]，预训练的大型语言模型（LLMs）在许多决策场景中展示了令人印象深刻的零-shot表现[[15](https://arxiv.org/html/2311.16714v2#bib.bib15)、[66](https://arxiv.org/html/2311.16714v2#bib.bib66)、[13](https://arxiv.org/html/2311.16714v2#bib.bib13)、[6](https://arxiv.org/html/2311.16714v2#bib.bib6)、[77](https://arxiv.org/html/2311.16714v2#bib.bib77)、[25](https://arxiv.org/html/2311.16714v2#bib.bib25)、[42](https://arxiv.org/html/2311.16714v2#bib.bib42)、[57](https://arxiv.org/html/2311.16714v2#bib.bib57)、[29](https://arxiv.org/html/2311.16714v2#bib.bib29)]。尽管在作为高质量且可扩展的专家模型方面具有巨大潜力，它们只能通过文本描述与环境进行交互，而不能像EMMA一样使用原始像素观察。为了弥补这一差距，我们通过从模拟器中提取元数据[[30](https://arxiv.org/html/2311.16714v2#bib.bib30)]，将每个像素观察$s_{v}$转化为其文本等价物，这些元数据包括诸如观察到的物体、观察到的关系、库存和位置等属性。然后，我们采用规划领域定义语言（PDDL）[[1](https://arxiv.org/html/2311.16714v2#bib.bib1)]来描述这些元数据，并使用TextWorld引擎[[10](https://arxiv.org/html/2311.16714v2#bib.bib10)]创建一个等效的文本描述/状态$s_{l}$。更多细节请参见附录[7](https://arxiv.org/html/2311.16714v2#S7 "7 Parallel TextWorld ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")，此过程的示例如图[1](https://arxiv.org/html/2311.16714v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")所示。该方法使得任何预训练的LLM代理都可以通过跨模态模仿学习（IL）与EMMA进行训练。

### 2.3 通过跨模态模仿训练EMMA

算法1 DAgger-DPO与单一任务指令

1:初始化: $i=0$，$\mathcal{D}\leftarrow\varnothing$，EMMA $\pi_{\theta}$，LLM演员 $M_{a}$ 配备 FIFO 内存池 $\mathcal{P}\leftarrow\varnothing$，LLM 评论员 $M_{c}$ 2:输入: 最大试验次数 $I$，训练周期 $I_{e}$，视觉和文本环境 $E_{v}$，$E_{l}$，任务指令 $x_{\text{task}}$，参考代理 $\pi_{\text{ref}}$ 3:将 $\pi_{\theta}$ 初始化为 $\pi_{\text{ref}}$ $\triangleright$ 行为克隆初始化 4:当任务未完成或 $i<I$ 时 5:     通过 $E_{v}$ 获取 $\tau^{i}_{v}=[x_{\text{task}},s_{v}^{0},x_{a}^{0},\dots,s_{v}^{T},x_{a}^{T}]$，使用 $\pi_{\theta}$ 6:     通过 $E_{l}$ 获取 $\tau^{i}_{l}=[x_{\text{task}},s_{l}^{0},x_{a}^{0},\dots,s_{l}^{T},x_{a}^{T}]$，使用 $\tau^{i}_{v}$ 7:     生成回溯反馈 $\mathcal{P}_{i}=M_{c}(\tau^{i}_{l})$ 8:     使用 $\mathcal{P}_{i}$ 更新 $\mathcal{P}$（即 $\mathcal{P}\leftarrow\mathcal{P}\cup\mathcal{P}_{i}$） 9:     对于 $t=0$ 到 $T$ 做 $\triangleright$ 数据集聚合 10:         $x^{*}_{a}=M_{a}(\mathcal{P},x_{\text{task}},\dots,x_{a}^{t-1},s_{l}^{t})$ 11:         $\mathcal{D}\leftarrow\mathcal{D}\cup\{(x_{\text{task}},s_{v}^{t},x_{a}^{t},x^{% *}_{a})\}$ 12:     对于 $j=0$ 到 $I_{e}-1$ 做 $\triangleright$ 对 $\theta$ 进行梯度下降 13:         从 $\mathcal{D}$ 中抽样一个小批量 $\tau$ 14:         通过最小化方程式（[2](https://arxiv.org/html/2311.16714v2#S2.E2 "Equation 2 ‣ 2.3 Training EMMA via Cross-Modality Imitation ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")）更新 $\theta$，使用 $\pi_{\text{ref}}$ 和 $\tau$ 15:     $i\leftarrow i+1$ 16:输出: $\pi_{\theta^{*}}$

给定来自平行 TextWorld 的 LLM 专家，我们的目标是训练一个视觉世界中的 VLM 代理 $\pi_{\theta}$，以尽可能地模仿其行为。这等同于在由 $\pi_{\theta}$ 引起的状态分布下，最小化以下目标。

|  | $\theta^{*}=\mathop{\arg\min}\limits_{\theta\in\Theta}\mathbb{E}_{\pi_{\theta}}\left[\mathcal{L}_{\text{imit}}\left(\pi_{\theta}(x_{a} \mid s_{v}),x^{*}_{a}\right)\right],$ |  | (1) |
| --- | --- | --- | --- |

在选择损失函数 $\mathcal{L}_{\text{imit}}$ 时，需要根据具体场景来决定。例如，对于离散动作空间，它可能是期望的交叉熵损失，而对于连续动作空间，它可能是期望的均方误差（MSE）损失。在我们的案例中，我们选择 DPO [[48](https://arxiv.org/html/2311.16714v2#bib.bib48)] 损失，因为它在将模型与专家偏好对齐方面，已被证明在离散语言空间中优于交叉熵损失。因此，方程式（[1](https://arxiv.org/html/2311.16714v2#S2.E1 "Equation 1 ‣ 2.3 Training EMMA via Cross-Modality Imitation ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")）可以扩展为以下公式。

|  | $\displaystyle\theta^{*}=\mathop{\arg\min}\limits_{\theta\in\Theta}-\mathbb{E}_% {\pi_{\theta}}\mathopen{}\left[\mathcal{L}_{\text{imit}}\mathopen{}\left(\pi_{% \theta},\pi_{\text{ref}},s_{v},x_{a},x^{*}_{a}\mathclose{}\right)\mathclose{}% \right],$ |  | (2) |
| --- | --- | --- | --- |
|  | $\displaystyle\mathcal{L}_{\text{imit}}(\cdot)\triangleq\log\sigma\mathopen{}% \left(\beta\log\frac{\pi_{\theta}(x^{*}_{a}&#124;s_{v})}{\pi_{\text{ref}}(x^{*}_{a}% &#124;s_{v})}-\beta\log\frac{\pi_{\theta}(x_{a}&#124;s_{v})}{\pi_{\text{ref}}(x_{a}&#124;s_{v% })}\mathclose{}\right)$ |  |

其中，$x^{*}_{a}$ 是专家给出的动作，而 $x_{a}$ 是由 $\pi_{\theta}$ 给出的动作，$\sigma$ 是逻辑函数，$\beta$ 是一个超参数，用于控制与 $\pi_{\text{ref}}$ 的偏差，即通过行为克隆从基于规则的专家在视觉世界中的示范数据集获取的参考代理（完整细节见附录 [9](https://arxiv.org/html/2311.16714v2#S9 "9 Collection of Demonstration Dataset ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")）。这个正则化非常重要，因为它可以防止代理过度偏离专家精确的分布，并且有助于保持生成的多样性，避免过早地收敛到某些简单任务 [[48](https://arxiv.org/html/2311.16714v2#bib.bib48)]。在实践中，VLM 代理 $\pi_{\theta}$ 也初始化为 $\pi_{\text{ref}}$，以稳定训练过程。由于环境动态既未知又复杂，我们无法计算 $\pi_{\theta}$ 访问的状态分布，只能通过展开代理来采样。因此，公式 ([2](https://arxiv.org/html/2311.16714v2#S2.E2 "Equation 2 ‣ 2.3 Training EMMA via Cross-Modality Imitation ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")) 是一个非独立同分布的监督学习问题，因为状态分布依赖于 $\pi_{\theta}$ 本身，在这种情况下，天真的行为克隆面临着如累积误差和分布偏移等问题 [[26](https://arxiv.org/html/2311.16714v2#bib.bib26)]。为了解决这个问题，我们采用了一种交互式的模仿学习算法，DAgger [[51](https://arxiv.org/html/2311.16714v2#bib.bib51)]，它可以证明收敛到最优代理 $\pi_{\theta^{*}}$。

如第[2.2](https://arxiv.org/html/2311.16714v2#S2.SS2 "2.2 LLM Expert from a Parallel TextWorld ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")节和第[2.3](https://arxiv.org/html/2311.16714v2#S2.SS3 "2.3 Training EMMA via Cross-Modality Imitation ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")节讨论的那样，我们可以利用LLM专家生成一系列动作，作为方程式([2](https://arxiv.org/html/2311.16714v2#S2.E2 "Equation 2 ‣ 2.3 Training EMMA via Cross-Modality Imitation ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld"))中的$x^{*}_{a}$。然而，由于环境反馈稀疏[[54](https://arxiv.org/html/2311.16714v2#bib.bib54), [68](https://arxiv.org/html/2311.16714v2#bib.bib68)]或上下文指令缺陷[[9](https://arxiv.org/html/2311.16714v2#bib.bib9)]，这些动作可能并不理想。为了收集更好的教学信号，我们引入了一个“回溯LLM专家”，它由两个专业模型组成：一个行为者（$M_{a}$），基于API LLM并被提示根据任务指令和状态观察生成动作；一个评论员（$M_{c}$），同样基于相同的LLM，但设计用于分析EMMA的历史互动并提供反思反馈。维护一个长期记忆$\mathcal{P}$来存储由$M_{c}$生成的反馈，随后这些反馈将被用来提示$M_{a}$生成改进的动作。完整的过程在算法[1](https://arxiv.org/html/2311.16714v2#alg1 "Algorithm 1 ‣ 2.3 Training EMMA via Cross-Modality Imitation ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")的第7至第10行中有详细描述，所有提示信息可以在附录的第[6](https://arxiv.org/html/2311.16714v2#S6 "6 Full Prompts for LLM Expert ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")节中找到。

## 3 实验

{floatrow}\ffigbox

[0.33]

图 3：EMMA与回溯LLM专家成功率的比较。随着试验次数的增加，LLM专家通过回溯过程逐步提高其表现，同时两者之间的差距也在减少，表明跨模态模仿的有效性。我们在附录图[9](https://arxiv.org/html/2311.16714v2#S8.F9 "Figure 9 ‣ 8 Training Details ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")中分别绘制了每种任务类型的曲线。

\capbtabbox

[0.63] 智能体 视觉 文本 成功率 与模板任务指令 环境 环境 平均值 拣取 清洁 加热 冷却 查看 拣取2 VMs ResNet-18* [[56](https://arxiv.org/html/2311.16714v2#bib.bib56)] ✓ ✗ 0.06 (-) - - - - - - MCNN-FPN* [[56](https://arxiv.org/html/2311.16714v2#bib.bib56)] ✓ ✗ 0.05 (-) - - - - - - LMs BUTLER* [[56](https://arxiv.org/html/2311.16714v2#bib.bib56)] ✗ ✓ 0.26 (-) 0.31 (-) 0.41 (-) 0.60 (-) 0.27 (-) 0.12 (-) 0.29 (-) GPT-BUTLER [[41](https://arxiv.org/html/2311.16714v2#bib.bib41)] ✗ ✓ 0.69 (18.8) 0.62 (18.4) 0.81 (18.4) 0.85 (12.7) 0.78 (15.6) 0.50 (24.4) 0.47 (26.6) ReAct [[73](https://arxiv.org/html/2311.16714v2#bib.bib73)] ✗ ✓ 0.54 (20.6) 0.71 (18.1) 0.65 (18.8) 0.62 (18.2) 0.44 (23.2) 0.28 (23.7) 0.35 (25.5) Reflexion [[54](https://arxiv.org/html/2311.16714v2#bib.bib54)] ✗ ✓ 0.91 (18.7) 0.96 (17.4) 1.00 (17.0) 0.81 (19.4) 0.83 (21.6) 0.94 (16.9) 0.88 (21.6) DEPS* [[61](https://arxiv.org/html/2311.16714v2#bib.bib61)] ✗ ✓ 0.76 (-) 0.93 (-) 0.50 (-) 0.80 (-) 1.00 (-) 1.00 (-) 0.00 (-) AutoGen* [[64](https://arxiv.org/html/2311.16714v2#bib.bib64)] ✗ ✓ 0.77 (-) 0.92 (-) 0.74 (-) 0.78 (-) 0.86 (-) 0.83 (-) 0.41 (-) VLMs MiniGPT-4 [[76](https://arxiv.org/html/2311.16714v2#bib.bib76)] ✓ ✗ 0.16 (26.9) 0.04 (29.0) 0.00 (30.0) 0.19 (26.3) 0.17 (26.7) 0.67 (17.7) 0.06 (28.9) BLIP-2 [[34](https://arxiv.org/html/2311.16714v2#bib.bib34)] ✓ ✗ 0.04 (29.5) 0.00 (30.0) 0.06 (29.3) 0.04 (29.9) 0.11 (28.2) 0.06 (29.2) 0.00 (30.0) LLaMA-Adapter [[17](https://arxiv.org/html/2311.16714v2#bib.bib17)] ✓ ✗ 0.13 (27.5) 0.17 (26.4) 0.10 (28.6) 0.27 (24.2) 0.22 (26.7) 0.00 (30.0) 0.00 (30.0) InstructBLIP [[11](https://arxiv.org/html/2311.16714v2#bib.bib11)] ✓ ✗ 0.22 (26.2) 0.50 (21.5) 0.26 (25.0) 0.23 (27.2) 0.06 (28.9) 0.17 (26.8) 0.00 (30.0) EMMA (我们) ✓ ✗ 0.82 (19.5) 0.71 (19.3) 0.94 (17.5) 0.85 (19.6) 0.83 (19.9) 0.88 (19.6) 0.67 (22.4) 人类表现* [[55](https://arxiv.org/html/2311.16714v2#bib.bib55)] ✓ ✗ 0.91 (-) - - - - - -

图4：与现有技术的比较。$*$- 在之前的工作中报告。VMs = 视觉模型，LMs = 语言模型，VLMs = 视觉-语言模型。“视觉环境”（Visual Env.）和“文本环境”（Textual Env.）分别指代来自ALFWorld的视觉环境和并行文本环境[[56](https://arxiv.org/html/2311.16714v2#bib.bib56)]。✓/✗表示相应的环境已使用/未使用以部署智能体。每个任务在同类型环境中的最高分数以粗体突出显示。平均交互步数给出在（括号内）。EMMA在视觉环境中明显优于其他SOTA VLM智能体，其成功也为在ALFWorld中实现人类水平的表现指明了一个有前景的方向。

### 3.1 实验设置

环境。我们的实验基于ALFWorld基准[[56](https://arxiv.org/html/2311.16714v2#bib.bib56)]，这是一个跨模态模拟平台，涵盖了广泛的具身家务任务。对于每个任务，ALFWorld集成了由Ai2Thor模拟器[[30](https://arxiv.org/html/2311.16714v2#bib.bib30)]渲染的视觉环境，以及一个对应的文本环境。文本环境使用规划领域定义语言（PDDL）[[1](https://arxiv.org/html/2311.16714v2#bib.bib1)]将模拟器中的每个像素观察转化为等效的基于文本的观察，然后使用TextWorld引擎[[10](https://arxiv.org/html/2311.16714v2#bib.bib10)]构建一个交互式世界。图[1](https://arxiv.org/html/2311.16714v2#S1.F1 "图1 ‣ 1 引言 ‣ 由LLM训练的具身多模态代理来自一个并行的TextWorld")提供了ALFWorld中任务的示例。基准中的任务分为6种类型：取放（Pick & Place）、清洁与放置（Clean & Place）、加热与放置（Heat & Place）、冷却与放置（Cool & Place）、在光源中寻找（Look in Light）、以及取两个物品并放置（Pick Two Objects & Place）。每个任务都要求代理执行一系列基于文本的动作，如“去保险柜1”、“打开保险柜1”或“用微波炉1加热鸡蛋1”，以遵循预定义的指令。这些动作涉及导航和与环境的交互。为了提供全面的理解，我们在附录的图[8](https://arxiv.org/html/2311.16714v2#Sx2.F8 "图8 ‣ 附录 ‣ 由LLM训练的具身多模态代理来自一个并行的TextWorld")中可视化了每种任务类型的示例。基准中的一个任务可能涉及与超过10个物体的互动，并且需要超过30个步骤才能由人类专家解决，因此对代理在长远规划、指令执行以及常识知识利用方面的能力提出了挑战。为了进行公平比较，我们遵循先前工作的相同设置[[55](https://arxiv.org/html/2311.16714v2#bib.bib55), [56](https://arxiv.org/html/2311.16714v2#bib.bib56), [41](https://arxiv.org/html/2311.16714v2#bib.bib41), [73](https://arxiv.org/html/2311.16714v2#bib.bib73), [54](https://arxiv.org/html/2311.16714v2#bib.bib54)]，并使用134个超出分布（OOD）任务评估所有基准。

基准模型。为了验证跨模态模仿学习的有效性，我们将我们的EMMA与多个基准模型和最先进的代理进行了比较，使用ALFWorld基准测试，涵盖了视觉和文本环境。比较的代理可以分为三类：视觉模型、语言模型和视觉-语言模型。具体而言，视觉模型，包括ResNet-18 [[20](https://arxiv.org/html/2311.16714v2#bib.bib20)]和MCNN-FPN [[21](https://arxiv.org/html/2311.16714v2#bib.bib21)]，利用预训练的视觉编码器从每个像素观察中提取显著特征。然后，提取的特征作为多层感知器（MLP）策略的输入，MLP通过行为克隆在预先收集的示范数据集上进行训练。与在视觉环境中执行的视觉模型不同，语言模型在并行的基于文本的环境中完成相同的任务。BUTLER [[56](https://arxiv.org/html/2311.16714v2#bib.bib56)]采用增强了指针softmax机制的transformer seq2seq模型 [[19](https://arxiv.org/html/2311.16714v2#bib.bib19)]。该架构将先前的观察聚合为输入，以生成基于文本的动作，逐个标记地生成。GPT-BUTLER [[41](https://arxiv.org/html/2311.16714v2#bib.bib41)]是GPT-2模型的变体 [[45](https://arxiv.org/html/2311.16714v2#bib.bib45)]，最初在静态示范数据集上进行预训练，之后通过在线收集的数据进一步微调。ReAct [[73](https://arxiv.org/html/2311.16714v2#bib.bib73)]采用了一种新颖的方法，利用大型语言模型（LLMs）以交错的方式生成推理痕迹和任务特定的动作。这种方法帮助代理以互动方式开发、跟踪并更新其行动计划。Reflexion [[54](https://arxiv.org/html/2311.16714v2#bib.bib54)]同样使用LLM，但它专注于反思环境反馈。它将这种反思文本保存在情节记忆缓冲区中，从而增强代理在后续试验中改善动作的能力。与Reflexion类似，一项并行的工作DEPS [[61](https://arxiv.org/html/2311.16714v2#bib.bib61)]，也通过整合动作执行过程的描述和提供反馈的自我解释，纠正了之前LLM生成的动作中的错误。此外，超越单代理框架，AutoGen [[64](https://arxiv.org/html/2311.16714v2#bib.bib64)]展示了通过多个LLM代理的合作，完成广泛任务的潜力。最后，我们考虑了一系列视觉-语言模型，如MiniGPT-4 [[76](https://arxiv.org/html/2311.16714v2#bib.bib76)]、BLIP-2 [[34](https://arxiv.org/html/2311.16714v2#bib.bib34)]、LLaMA-Adaptor [[17](https://arxiv.org/html/2311.16714v2#bib.bib17)]和InstructBLIP [[11](https://arxiv.org/html/2311.16714v2#bib.bib11)]，作为与视觉环境交互的代理。与纯视觉或语言模型不同，视觉-语言模型（VLMs）旨在处理和集成视觉和文本数据，提供对环境的更全面理解。为了使这些代理与ALFWorld基准测试的特定要求对齐，我们在预先收集的示范数据集上对它们进行了微调。这个微调过程至关重要，因为它使代理能够理解并遵循ALFWorld独特的语法，并开发出基本的“游戏感知”。

### 3.2 训练细节

EMMA的架构设计如图[2](https://arxiv.org/html/2311.16714v2#S1.F2 "图 2 ‣ 1 引言 ‣ 由并行文本世界训练的具身多模态代理")所示。EMMA的核心采用了查询变换器（Q-Former）来处理视觉数据。该Q-Former使用冻结的ViT编码器提取特征。它的输出包含32个视觉标记，这些标记随后通过一个线性投影层，之后输入到冻结的LLM解码器中。与其他VLM代理类似，EMMA也在预先收集的示范数据集上进行微调，使其基本能力与ALFWorld基准对齐。为了使EMMA与ALFWorld的动态对齐，我们通过模仿LLM专家来训练它（见算法[1](https://arxiv.org/html/2311.16714v2#alg1 "算法 1 ‣ 2.3 通过跨模态模仿训练EMMA ‣ 2 具身多模态代理 ‣ 由并行文本世界训练的具身多模态代理")）。我们选择了由OpenAI开发的text-davinci-003作为LLM专家，因为它在推理和规划方面具有可靠的能力[[54](https://arxiv.org/html/2311.16714v2#bib.bib54), [73](https://arxiv.org/html/2311.16714v2#bib.bib73), [64](https://arxiv.org/html/2311.16714v2#bib.bib64), [61](https://arxiv.org/html/2311.16714v2#bib.bib61)]。在此设置中，text-davinci-003扮演双重角色：它作为行为者，为EMMA提供专家行为，并作为评论者，分析EMMA的历史轨迹。这一分析生成回溯反馈，随后被纳入行为者的长期记忆，从而在未来的试验中改善行为。关于我们训练过程中的超参数和提示的更多细节，请参见附录中的表[1](https://arxiv.org/html/2311.16714v2#S8.T1 "表 1 ‣ 8 训练细节 ‣ 由并行文本世界训练的具身多模态代理")。

### 3.3 与现有技术的比较

EMMA在视觉ALFWorld中设定了新的SOTA表现。在本节中，我们将EMMA与12个其他代表性智能体进行比较，涵盖了视觉和文本环境的ALFWorld基准（参见表[4](https://arxiv.org/html/2311.16714v2#S3.F4 "Figure 4 ‣ 3 Experiments ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")）。我们评估了两个关键指标：成功率，即成功完成任务的试验百分比，以及完成任务所需的平均互动步数，步数越少表示效率越高。EMMA在这两个指标上都表现优异，显著超过了所有视觉环境中的VLM智能体。这一成就突显了我们跨模态模仿学习方法的有效性，正如图[4](https://arxiv.org/html/2311.16714v2#S3.F4 "Figure 4 ‣ 3 Experiments ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")所示的学习曲线所展示的那样。此外，EMMA的表现显著超过了VM智能体，凸显了VLM中嵌入的先验知识的重要作用。有趣的是，EMMA的表现与使用完全语义描述视觉观察的LLM智能体相当。这在很大程度上归功于EMMA的训练策略，即模仿专家LLM智能体，其比在纯视觉环境中从零开始学习更加高效。因此，EMMA脱颖而出，成为唯一一个在这些环境中显著超越SOTA LLM智能体（如AutoGen [[64](https://arxiv.org/html/2311.16714v2#bib.bib64)]和DEPS [[61](https://arxiv.org/html/2311.16714v2#bib.bib61)]）的VLM智能体。它的成功也为在ALFWorld的视觉环境中实现人类水平的表现指明了潜在的方向。

相较于LLM代理，EMMA对噪声观测具有更强的鲁棒性。尽管LLM代理在文本环境中以较少的交互步骤展现了较高的成功率，如表[4](https://arxiv.org/html/2311.16714v2#S3.F4 "Figure 4 ‣ 3 Experiments ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")所示，我们假设这种优越的表现主要依赖于它们对环境的精确语义抽象。然而，这种抽象在实际应用中可能不可行。为了验证这一假设，我们设置了一个更实际的场景，在该场景中观测数据以特定的噪声率故意被扰动。然后，我们比较了EMMA和最先进的LLM代理Reflexion在这些噪声观测下的鲁棒性。为了生成噪声观测，我们裁剪图像观测的随机部分，调整大小后用它替换原始观测。同样，在文本观测中，随机的标记会被任意标记替换。如图[5](https://arxiv.org/html/2311.16714v2#S3.F5 "Figure 5 ‣ 3.3 Comparison with State of the Art ‣ 3 Experiments ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")所示，随着噪声率的增加，EMMA的表现明显比Reflexion更加鲁棒。这可能归因于VLM中的视觉编码器，能够有效过滤视觉噪声。另一方面，文本噪声直接由LLM处理，可能会显著影响基于LLM的代理的表现。这个发现突出了像EMMA这样的VLM代理在视觉场景中的潜在优势，因为这些场景中的数据通常是不完美和有噪声的。

图5：VLM和LLM代理的鲁棒性比较。在给定特定噪声率的情况下，我们裁剪图像观测的一个随机部分并将其调整为给定大小，作为新的观测。同样，我们还随机替换文本观测中的一些标记为任意标记。

### 3.4 消融研究

图6：消融研究。左侧：LLM专家反思并从过去的行动中学习的能力似乎是实现我们观察到的显著结果的关键因素。中间：“无BC初始化”变体使用预训练的VLM，而不是参考代理$\pi_{\text{ref}}$来初始化EMMA。右侧：“有CE损失”变体将DPO损失（在公式[2](https://arxiv.org/html/2311.16714v2#S2.E2 "Equation 2 ‣ 2.3 Training EMMA via Cross-Modality Imitation ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")中描述）替换为基于标记的交叉熵（CE）损失。请参阅第[3.4](https://arxiv.org/html/2311.16714v2#S3.SS4 "3.4 Ablation Study ‣ 3 Experiments ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")节以获得详细讨论。

回顾机制随着时间的推移提升了EMMA的表现。为了评估回顾性LLM专家的重要性，我们展示了EMMA在每次试验后的平均成功率，并与一个关键变体进行比较：“EMMA w/o Retrospection”，如图[6](https://arxiv.org/html/2311.16714v2#S3.F6 "Figure 6 ‣ 3.4 Ablation Study ‣ 3 Experiments ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")（左侧）所示。这个EMMA变体与原始EMMA使用相同的训练程序，但去除了回顾过程。相反，它仅依赖于一个普通的LLM代理来提供重新标注的动作。结果显示，具有回顾机制的EMMA明显优于其对照组。这一发现至关重要，因为它表明回顾过程不仅仅是一个补充特性，而是EMMA架构中的一个基本组件，显著地提升了其性能。

EMMA受益于BC初始化。通过一项消融研究，我们评估了行为克隆（BC）初始化的影响，这一过程在算法[1](https://arxiv.org/html/2311.16714v2#alg1 "Algorithm 1 ‣ 2.3 Training EMMA via Cross-Modality Imitation ‣ 2 Embodied Multi-Modal Agent ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")的第3行中有描述。图[6](https://arxiv.org/html/2311.16714v2#S3.F6 "Figure 6 ‣ 3.4 Ablation Study ‣ 3 Experiments ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")（中间）展示了，当EMMA没有BC初始化时，相较于原始设置，在134个未见任务中的平均成功率略有下降。尽管有所下降，EMMA在没有BC初始化的情况下仍然优于其他VLM代理，正如与表[4](https://arxiv.org/html/2311.16714v2#S3.F4 "Figure 4 ‣ 3 Experiments ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")中结果的对比所清楚显示的那样。此外，附录中的图[10](https://arxiv.org/html/2311.16714v2#S8.F10 "Figure 10 ‣ 8 Training Details ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")将结果按任务类型进行了细分。它揭示了在6种任务类型中的5种任务上，性能出现了一致但轻微的下降。这些结果表明，虽然BC初始化对EMMA的整体表现有积极贡献，但它并不是实现我们报告的显著结果的关键因素。

DPO（差分策略优化）使得模仿学习更为有效。为了评估在方程式([1](https://arxiv.org/html/2311.16714v2#S2.E1 "方程式 1 ‣ 2.3 通过跨模态模仿训练EMMA ‣ 具身多模态代理 ‣ 通过并行文本世界的LLM训练的具身多模态代理"))中使用DPO损失的有效性，我们进行了一项消融研究，使用了一个EMMA的替代版本，称为“EMMA w/ Cross Entropy (CE) Loss”（EMMA与交叉熵损失）。在这个变体中，EMMA使用标记级别的交叉熵损失进行优化，这是微调VLMs（视觉语言模型）的常见目标。结果如图[6](https://arxiv.org/html/2311.16714v2#S3.F6 "图 6 ‣ 3.4 消融研究 ‣ 3 实验 ‣ 通过并行文本世界的LLM训练的具身多模态代理")（右图）所示，EMMA w/ CE Loss未能达到原始EMMA使用DPO损失时的高成功率，表明DPO损失有助于提高EMMA的性能上限。此外，我们还注意到，EMMA w/ CE Loss在初始训练阶段比原始EMMA收敛速度更快。这种过早收敛导致代理更关注来自较容易任务的专家动作，这些任务通常在训练的前几个周期中解决。这可能抑制了EMMA在更复杂任务上的探索和学习，导致性能下降。

### 3.5 自由格式任务指令的泛化能力

| 代理 | 视觉 | 文本 | 自由格式任务指令的成功率 |
| --- | --- | --- | --- |
| 环境 | 环境 | 平均值 | 选择 | 清理 | 加热 | 冷却 | 观察 | 选择2 |
| LMs | BUTLER* [[56](https://arxiv.org/html/2311.16714v2#bib.bib56)] | ✗ | ✓ | 0.03 | (-) | 0.10 | (-) | 0.22 | (-) | 0.05 | (-) | 0.17 | (-) | 0.00 | (-) | 0.00 | (-) |
| GPT-BUTLER [[41](https://arxiv.org/html/2311.16714v2#bib.bib41)] | ✗ | ✓ | 0.31 | (24.7) | 0.25 | (25.4) | 0.42 | (23.5) | 0.46 | (20.9) | 0.44 | (21.8) | 0.00 | (30.0) | 0.12 | (28.9) |
| ReAct [[73](https://arxiv.org/html/2311.16714v2#bib.bib73)] | ✗ | ✓ | 0.37 | (23.6) | 0.46 | (22.0) | 0.39 | (23.8) | 0.50 | (20.5) | 0.28 | (24.6) | 0.22 | (26.4) | 0.29 | (25.9) |
| Reflexion [[54](https://arxiv.org/html/2311.16714v2#bib.bib54)] | ✗ | ✓ | 0.78 | (17.0) | 0.84 | (15.8) | 0.74 | (18.4) | 0.77 | (17.2) | 0.61 | (19.4) | 0.94 | (13.8) | 0.82 | (16.7) |
| VLMs | MiniGPT-4 [[76](https://arxiv.org/html/2311.16714v2#bib.bib76)] | ✓ | ✗ | 0.00 | (30.0) | 0.00 | (30.0) | 0.00 | (30.0) | 0.00 | (30.0) | 0.00 | (30.0) | 0.00 | (30.0) | 0.00 | (30.0) |
| BLIP-2 [[34](https://arxiv.org/html/2311.16714v2#bib.bib34)] | ✓ | ✗ | 0.01 | (29.7) | 0.04 | (29.1) | 0.03 | (29.5) | 0.00 | (30.0) | 0.00 | (30.0) | 0.00 | (30.0) | 0.00 | (30.0) |
| LLaMA-Adapter [[17](https://arxiv.org/html/2311.16714v2#bib.bib17)] | ✓ | ✗ | 0.02 | (29.6) | 0.04 | (29.3) | 0.03 | (29.4) | 0.04 | (29.3) | 0.00 | (30.0) | 0.00 | (30.0) | 0.00 | (30.0) |
| InstructBLIP [[11](https://arxiv.org/html/2311.16714v2#bib.bib11)] | ✓ | ✗ | 0.01 | (29.8) | 0.00 | (30.0) | 0.03 | (29.3) | 0.00 | (30.0) | 0.00 | (30.0) | 0.00 | (30.0) | 0.00 | (30.0) |
| EMMA (我们的方法) | ✓ | ✗ | 0.68 | (22.0) | 0.72 | (21.7) | 0.65 | (22.7) | 0.72 | (22.0) | 0.56 | (23.9) | 0.67 | (23.3) | 0.76 | (18.1) |

图 7：开放词汇、自由形式任务指令的泛化能力。左侧：这些指令由不同的人工注释者通过 Amazon Mechanical Turk [[55](https://arxiv.org/html/2311.16714v2#bib.bib55)] 提供。右侧：人工注释和模板化任务指令中动词的频率分布。

AI 代理能够准确遵循开放词汇和自由形式文本中给出的任务指令，对解决现实世界问题至关重要。为了评估这一点，我们进行了额外的实验，重点考察 EMMA（一个使用模板任务指令训练的代理）的泛化能力。在该实验中，我们使用人工注释的指令而非模板指令，对 EMMA 和其他基准代理在 134 个未见过的任务上的表现进行了重新评估。这些人工注释的指令包含大量的 OOD 动词和对象，为代理提供了一个更具现实性和挑战性的场景。为了突出这一挑战，我们比较了模板指令和人工注释指令之间的词汇分布，如图 [7](https://arxiv.org/html/2311.16714v2#S3.F7 "图 7 ‣ 3.5 自由形式任务指令的泛化 ‣ 3 实验 ‣ 由 LLM 训练的多模态代理，来自并行文本世界")（右）所示。此外，我们在附录中的图 [11](https://arxiv.org/html/2311.16714v2#S8.F11 "图 11 ‣ 8 训练细节 ‣ 由 LLM 训练的多模态代理，来自并行文本世界") 提供了关于两种指令类型所用词汇的全面分析。在图 [7](https://arxiv.org/html/2311.16714v2#S3.F7 "图 7 ‣ 3.5 自由形式任务指令的泛化 ‣ 3 实验 ‣ 由 LLM 训练的多模态代理，来自并行文本世界")（左）中，EMMA 显示出轻微的性能下降，而在其他基准方法中观察到显著的性能退化。我们还注意到，作为一种最先进的 LLM 代理，Reflexion 展现了对这些 OOD 指令的卓越泛化能力。根据图 [7](https://arxiv.org/html/2311.16714v2#S3.F7 "图 7 ‣ 3.5 自由形式任务指令的泛化 ‣ 3 实验 ‣ 由 LLM 训练的多模态代理，来自并行文本世界") 中的这些实证结果，我们得出以下结论：（1）EMMA 通过跨模态模仿学习获得并受益于最先进 LLM 代理固有的泛化能力；（2）我们的工作为使用 LLM 反馈训练更具多功能性和更具泛化能力的体现代理提供了新的见解。

## 4 相关工作

基于基础模型的智能体。最近的研究越来越多地关注利用大型预训练基础模型的能力来构建AI智能体[[65](https://arxiv.org/html/2311.16714v2#bib.bib65), [69](https://arxiv.org/html/2311.16714v2#bib.bib69)]。这些模型（例如，LLMs）通过其从互联网规模的预训练中继承的常识性知识，能够根据外部环境的描述推理出行动。例如，给定一组任务指令，LLMs可以通过精心设计的提示，作为智能体生成高层次的逐步计划[[24](https://arxiv.org/html/2311.16714v2#bib.bib24), [7](https://arxiv.org/html/2311.16714v2#bib.bib7), [54](https://arxiv.org/html/2311.16714v2#bib.bib54), [73](https://arxiv.org/html/2311.16714v2#bib.bib73), [64](https://arxiv.org/html/2311.16714v2#bib.bib64)]，每一步都可以解析为通过预训练的策略或可用API执行的机器人动作序列[[6](https://arxiv.org/html/2311.16714v2#bib.bib6), [36](https://arxiv.org/html/2311.16714v2#bib.bib36), [59](https://arxiv.org/html/2311.16714v2#bib.bib59)]。此外，通过使用VLMs[[32](https://arxiv.org/html/2311.16714v2#bib.bib32)]，计划还可以基于视觉输入进行调整，这些输入会转化为语言描述[[25](https://arxiv.org/html/2311.16714v2#bib.bib25), [57](https://arxiv.org/html/2311.16714v2#bib.bib57), [7](https://arxiv.org/html/2311.16714v2#bib.bib7), [16](https://arxiv.org/html/2311.16714v2#bib.bib16)]，或者与LLMs对齐的令牌嵌入[[13](https://arxiv.org/html/2311.16714v2#bib.bib13), [42](https://arxiv.org/html/2311.16714v2#bib.bib42), [77](https://arxiv.org/html/2311.16714v2#bib.bib77), [68](https://arxiv.org/html/2311.16714v2#bib.bib68)]。然而，现有的基础模型通常是基于静态文本或文本-图像数据集进行预训练的，因此可能难以与世界的动态变化对接。为了弥合这一差距，我们研究了如何微调VLM，使其成为与世界动态对接的具身智能体，通过从LLM专家中提炼跨模态知识。与我们工作最为相关的研究是EUREKA[[40](https://arxiv.org/html/2311.16714v2#bib.bib40)]，它也探讨了如何利用模拟器提供的源信息作为LLM的上下文来辅助智能体训练。与我们直接模仿LLM输出的方法不同，EUREKA利用编码LLM生成给定任务的期望奖励函数，并通过强化学习（RL）优化策略以获得该奖励函数，从而导致了一个更复杂且成本更高的训练过程[[52](https://arxiv.org/html/2311.16714v2#bib.bib52), [53](https://arxiv.org/html/2311.16714v2#bib.bib53), [47](https://arxiv.org/html/2311.16714v2#bib.bib47)]。

模仿学习。模仿学习是研究通过模仿专家的决策和行为来提高性能的算法。我们总结了现有方法的三大主要类别，如下所示：（1）行为克隆（BC），（2）逆强化学习（IRL），以及（3）模仿学习与强化学习的结合。天真的BC [[5](https://arxiv.org/html/2311.16714v2#bib.bib5)]忽略了分布的变化，仅训练一个只在专家访问的状态分布下表现良好的策略。后续的工作，例如数据集聚合 [[51](https://arxiv.org/html/2311.16714v2#bib.bib51)]或策略聚合 [[28](https://arxiv.org/html/2311.16714v2#bib.bib28), [49](https://arxiv.org/html/2311.16714v2#bib.bib49)]，已被提出以解决BC的局限性。另一类工作，IRL，是一个更复杂的算法框架，它通过专家示范学习奖励函数，然后使用RL通过学习到的奖励来改进策略。该类别的代表性方法是生成对抗模仿学习（GAIL） [[23](https://arxiv.org/html/2311.16714v2#bib.bib23)]，其中策略和判别器相互竞争，以最大化策略行为与专家行为匹配的可能性。第三类方法通常利用IL策略来初始化RL，并通过RL收集的在线数据继续提升其性能 [[8](https://arxiv.org/html/2311.16714v2#bib.bib8), [50](https://arxiv.org/html/2311.16714v2#bib.bib50), [58](https://arxiv.org/html/2311.16714v2#bib.bib58)]。这种简单的组合显著提高了RL的样本效率和IL在专家约束下的性能上限。然而，以上所有方法假设专家和模仿者以相同的方式理解世界，因此忽略了来自其他模态的互补知识通常能显著提升模型的准确性和泛化能力 [[46](https://arxiv.org/html/2311.16714v2#bib.bib46), [67](https://arxiv.org/html/2311.16714v2#bib.bib67)]。

## 5 结论

我们通过在一个具象的视觉世界中微调VLM，结合来自LLM专家在平行文本世界中的交互式模仿学习，创建了EMMA，一个具象多模态智能体。LLM专家产生更好的行动和回顾性反馈，以改善VLM的轨迹。这种模仿学习比直接在视觉世界中微调的视觉或VLM策略，或通过规则基专家的行为克隆微调的策略，甚至比像GPT-4V(ision)这样的最先进API VLMs具有明显优势。因此，EMMA在噪声较大的视觉世界中比其LLM教师在较容易的文本世界中的成功率更高，且鲁棒性更强。此外，EMMA在开放词汇和自由形式任务指令上表现出强大的泛化能力，凸显了其在现实场景中的潜力。

## 致谢

本研究部分得到了中国国家重点研发计划（2023YFE0106300 和 2017YFC0804002）、深圳市基础研究计划（JCYJ20200109141235597）以及广东省引进创新和创业团队计划（2017ZT07X386）的资助支持。

## 参考文献

+   Aeronautiques 等 [1998] Constructions Aeronautiques, Adele Howe, Craig Knoblock, ISI Drew McDermott, Ashwin Ram, Manuela Veloso, Daniel Weld, David Wilkins SRI, Anthony Barrett, Dave Christianson 等。PDDL: 规划领域定义语言. *技术报告, Tech. Rep.*，1998年。

+   Andrychowicz 等 [2017] Marcin Andrychowicz, Filip Wolski, Alex Ray, Jonas Schneider, Rachel Fong, Peter Welinder, Bob McGrew, Josh Tobin, OpenAI Pieter Abbeel 和 Wojciech Zaremba. 后见之明经验回放. 2017年。

+   Ao 等 [2021] Shuang Ao, Tianyi Zhou, Guodong Long, Qinghua Lu, Liming Zhu 和 Jing Jiang. Co-pilot: 子任务课程上的协同规划和强化学习. 2021年。

+   Ao 等 [2022] Shuang Ao, Tianyi Zhou, Jing Jiang, Guodong Long, Xuan Song, 和 Chengqi Zhang. Eat-c: 高效强化学习的环境对抗子任务课程. 在*ICML*，2022年。

+   Bain 和 Sammut [1995] Michael Bain 和 Claude Sammut. 行为克隆框架. 在*机器智能*，1995年。

+   Brohan 等 [2022] Anthony Brohan, Noah Brown, Justice Carbajal, Yevgen Chebotar, Joseph Dabis, Chelsea Finn, Keerthana Gopalakrishnan, Karol Hausman, Alex Herzog, Jasmine Hsu 等。Rt-1: 机器人变换器，用于大规模实际控制. *arXiv 预印本 arXiv:2212.06817*，2022年。

+   Brohan 等 [2023] Anthony Brohan, Yevgen Chebotar, Chelsea Finn, Karol Hausman, Alexander Herzog, Daniel Ho, Julian Ibarz, Alex Irpan, Eric Jang, Ryan Julian 等。做我能做的，而不是我说的：将语言与机器人功能对接. 在*CoRL*，2023年。

+   Chang 等 [2015] Kai-Wei Chang, Akshay Krishnamurthy, Alekh Agarwal, Hal Daumé III 和 John Langford. 学习如何比你的老师更好地搜索. 在*ICML*，2015年。

+   Chen 等 [2023] Lichang Chen, Jiuhai Chen, Tom Goldstein, Heng Huang 和 Tianyi Zhou. Instructzero: 针对黑盒大语言模型的高效指令优化. *arXiv 预印本 arXiv:2306.03082*，2023年。

+   Côté 等 [2019] Marc-Alexandre Côté, Akos Kádár, Xingdi Yuan, Ben Kybartas, Tavian Barnes, Emery Fine, James Moore, Matthew Hausknecht, Layla El Asri, Mahmoud Adada 等。Textworld: 基于文本游戏的学习环境. 在*计算机游戏研讨会, IJCAI*，2019年。

+   Dai 等 [2023] Wenliang Dai, Junnan Li, Dongxu Li, Anthony Tiong, Junqi Zhao, Weisheng Wang, Boyang Li, Pascale Fung 和 Steven Hoi. InstructBLIP: 面向通用视觉语言模型的指令调优. 在*NeurIPS*，2023年。

+   Devlin 等 [2019] Jacob Devlin, Ming-Wei Chang, Kenton Lee 和 Kristina Toutanova. BERT: 深度双向变换器的预训练用于语言理解. 在*NAACL-HLT*，2019年。

+   Driess 等人 [2023] Danny Driess, Fei Xia, Mehdi SM Sajjadi, Corey Lynch, Aakanksha Chowdhery, Brian Ichter, Ayzaan Wahid, Jonathan Tompson, Quan Vuong, Tianhe Yu, 等人. Palm-e: 一种具身的多模态语言模型. *arXiv 预印本 arXiv:2303.03378*, 2023.

+   Fang 等人 [2019] Meng Fang, Tianyi Zhou, Yali Du, Lei Han, 和 Zhengyou Zhang. 课程引导的事后经验回放. *NeurIPS*, 2019.

+   Fu 等人 [2023] Daocheng Fu, Xin Li, Licheng Wen, Min Dou, Pinlong Cai, Botian Shi, 和 Yu Qiao. 像人一样驾驶：用大型语言模型重新思考自动驾驶. *arXiv 预印本 arXiv:2307.07162*, 2023.

+   Gao 等人 [2023a] Jensen Gao, Bidipta Sarkar, Fei Xia, Ted Xiao, Jiajun Wu, Brian Ichter, Anirudha Majumdar, 和 Dorsa Sadigh. 面向机器人操控的物理基础视觉-语言模型. *arXiv 预印本 arXiv:2309.02561*, 2023a.

+   Gao 等人 [2023b] Peng Gao, Jiaming Han, Renrui Zhang, Ziyi Lin, Shijie Geng, Aojun Zhou, Wei Zhang, Pan Lu, Conghui He, Xiangyu Yue, 等人. Llama-adapter v2: 参数高效的视觉指令模型. *arXiv 预印本 arXiv:2304.15010*, 2023b.

+   Guan 等人 [2023] Jiayi Guan, Guang Chen, Jiaming Ji, Long Yang, Zhijun Li, 等人. Voce: 具有保守估计的变分优化用于离线安全强化学习. *NeurIPS*, 36, 2023.

+   Gülçehre 等人 [2016] Çaglar Gülçehre, Sungjin Ahn, Ramesh Nallapati, Bowen Zhou, 和 Yoshua Bengio. 指向未知词汇. 见 *ACL*, 2016.

+   He 等人 [2016] Kaiming He, Xiangyu Zhang, Shaoqing Ren, 和 Jian Sun. 深度残差学习用于图像识别. 见 *CVPR*, 2016.

+   He 等人 [2017] Kaiming He, Georgia Gkioxari, Piotr Dollár, 和 Ross B. Girshick. Mask R-CNN. 见 *ICCV*, 2017.

+   Helmert [2006] Malte Helmert. 快速向下规划系统. *人工智能研究杂志*, 2006.

+   Ho 和 Ermon [2016] Jonathan Ho 和 Stefano Ermon. 生成对抗模仿学习. 见 *NeurIPS*, 2016.

+   Huang 等人 [2022] Wenlong Huang, Pieter Abbeel, Deepak Pathak, 和 Igor Mordatch. 语言模型作为零-shot 规划器：为具身代理提取可操作知识. 见 *ICML*, 2022.

+   Huang 等人 [2023] Wenlong Huang, Chen Wang, Ruohan Zhang, Yunzhu Li, Jiajun Wu, 和 Li Fei-Fei. Voxposer: 可组合的 3D 值映射用于语言模型驱动的机器人操控. *arXiv 预印本 arXiv:2307.05973*, 2023.

+   Hussein 等人 [2017] Ahmed Hussein, Mohamed Medhat Gaber, Eyad Elyan, 和 Chrisina Jayne. 模仿学习：学习方法的综述. *ACM 计算机调查 (CSUR)*, 50(2):1–35, 2017.

+   Hutter [2004] Marcus Hutter. *通用人工智能：基于算法概率的顺序决策*. Springer Science & Business Media, 2004.

+   III 等人 [2009] Hal Daumé III, John Langford, 和 Daniel Marcu. 基于搜索的结构化预测. *Mach. Learn.*, 2009.

+   姜等人 [2023] Yunfan Jiang, Agrim Gupta, Zichen Zhang, Guanzhi Wang, Yongqiang Dou, Yanjun Chen, Li Fei-Fei, Anima Anandkumar, Yuke Zhu 和 Linxi Fan. VIMA：多模态提示的机器人操控. 在*ICML*，2023年。

+   Kolve 等人 [2017] Eric Kolve, Roozbeh Mottaghi, Winson Han, Eli VanderBilt, Luca Weihs, Alvaro Herrasti, Matt Deitke, Kiana Ehsani, Daniel Gordon, Yuke Zhu 等人. Ai2-thor：一个用于视觉 AI 的互动 3D 环境. *arXiv 预印本 arXiv:1712.05474*，2017年。

+   Levine 等人 [2020] Sergey Levine, Aviral Kumar, George Tucker 和 Justin Fu. 离线强化学习：教程、回顾与开放问题的前景. *arXiv 预印本 arXiv:2005.01643*，2020年。

+   李等人 [2023a] Chunyuan Li, Zhe Gan, Zhengyuan Yang, Jianwei Yang, Linjie Li, Lijuan Wang 和 Jianfeng Gao. 多模态基础模型：从专业化到通用助手. *arXiv 预印本 arXiv:2309.10020*，2023a年。

+   李等人 [2023b] Dongxu Li, Junnan Li, Hung Le, Guangsen Wang, Silvio Savarese 和 Steven C.H. Hoi. LAVIS：一个一站式的语言-视觉智能库. 在*第61届计算语言学协会年会论文集（第3卷：系统演示）*，2023b年。

+   李等人 [2023c] Junnan Li, Dongxu Li, Silvio Savarese 和 Steven C. H. Hoi. BLIP-2：通过冻结图像编码器和大型语言模型进行语言-图像预训练的自启动. 在*ICML*，2023c年。

+   梁等人 [2024] Chen Liang, Jiahui Yu, Ming-Hsuan Yang, Matthew Brown, Yin Cui, Tuo Zhao, Boqing Gong 和 Tianyi Zhou. 面向多模态基础模型的模块级自适应蒸馏. 2024年。

+   梁等人 [2023] Jacky Liang, Wenlong Huang, Fei Xia, Peng Xu, Karol Hausman, Brian Ichter, Pete Florence, 和 Andy Zeng. 代码作为策略：用于具身控制的语言模型程序. 在*ICRA*，2023年。

+   林等人 [2023] Yong Lin, Lu Tan, Hangyu Lin, Zeming Zheng, Renjie Pi, Jipeng Zhang, Shizhe Diao, Haoxiang Wang, Han Zhao, Yuan Yao 等人. 专业性与通用性：关于基础模型微调中灾难性遗忘的实证研究. *arXiv 预印本 arXiv:2309.06256*，2023年。

+   刘等人 [2023] Haotian Liu, Chunyuan Li, Qingyang Wu 和 Yong Jae Lee. 视觉指令调优. *arXiv 预印本 arXiv:2304.08485*，2023年。

+   Loshchilov 和 Hutter [2017] Ilya Loshchilov 和 Frank Hutter. 解耦的权重衰减正则化. *arXiv 预印本 arXiv:1711.05101*，2017年。

+   马等人 [2023] Yecheng Jason Ma, William Liang, Guanzhi Wang, De-An Huang, Osbert Bastani, Dinesh Jayaraman, Yuke Zhu, Linxi Fan 和 Anima Anandkumar. Eureka：通过编码大型语言模型进行人类级奖励设计. *arXiv 预印本 arXiv:2310.12931*，2023年。

+   Micheli 和 Fleuret [2021] Vincent Micheli 和 François Fleuret. 语言模型是少量样本的管家. 在*EMNLP*，2021年。

+   Mu 等人 [2023] Yao Mu, Qinglong Zhang, Mengkang Hu, Wenhai Wang, Mingyu Ding, Jun Jin, Bin Wang, Jifeng Dai, Yu Qiao 和 Ping Luo。 EmbodiedGPT：通过具象化思维链进行视觉-语言预训练。*arXiv 预印本 arXiv:2305.15021*，2023年。

+   OpenAI [2023] OpenAI. GPT-4V(ision) 系统卡片。 *[https://cdn.openai.com/papers/GPTV_System_Card.pdf](https://cdn.openai.com/papers/GPTV_System_Card.pdf)*, 2023.

+   Padalkar 等人 [2023] Abhishek Padalkar, Acorn Pooley, Ajinkya Jain, Alex Bewley, Alex Herzog, Alex Irpan, Alexander Khazatsky, Anant Rai, Anikait Singh, Anthony Brohan 等人。 Open x-embodiment：机器人学习数据集和 rt-x 模型，2023年。

+   Radford 等人 [2019] Alec Radford, Jeffrey Wu, Rewon Child, David Luan, Dario Amodei, Ilya Sutskever 等人。 语言模型是无监督的多任务学习者。 *OpenAI 博客*，2019年。

+   Radford 等人 [2021] Alec Radford, Jong Wook Kim, Chris Hallacy, Aditya Ramesh, Gabriel Goh, Sandhini Agarwal, Girish Sastry, Amanda Askell, Pamela Mishkin, Jack Clark 等人。 从自然语言监督中学习可转移的视觉模型。发表于 *ICML*，2021年。

+   Rafailov 等人 [2023] Rafael Rafailov, Archit Sharma, Eric Mitchell, Stefano Ermon, Christopher D Manning 和 Chelsea Finn。 直接偏好优化：你的语言模型实际上是一个奖励模型。 *arXiv 预印本 arXiv:2305.18290*，2023年。

+   Rafailov 等人 [2024] Rafael Rafailov, Archit Sharma, Eric Mitchell, Christopher D Manning, Stefano Ermon 和 Chelsea Finn。 直接偏好优化：你的语言模型实际上是一个奖励模型。 *NeurIPS*，2024年。

+   Ross 和 Bagnell [2010] Stéphane Ross 和 Drew Bagnell。 模仿学习的高效归约方法。发表于 *AISTATS*，2010年。

+   Ross 和 Bagnell [2014] Stéphane Ross 和 J Andrew Bagnell。 通过互动无悔学习进行强化学习和模仿学习。 *arXiv 预印本 arXiv:1406.5979*，2014年。

+   Ross 等人 [2011] Stéphane Ross, Geoffrey J. Gordon 和 Drew Bagnell。 将模仿学习和结构化预测归约为无悔在线学习。发表于 *AISTATS*，2011年。

+   Schulman 等人 [2015] John Schulman, Sergey Levine, Pieter Abbeel, Michael I. Jordan 和 Philipp Moritz。 信任区域策略优化。发表于 *ICML*，2015年。

+   Schulman 等人 [2017] John Schulman, Filip Wolski, Prafulla Dhariwal, Alec Radford 和 Oleg Klimov。 近端策略优化算法。 *arXiv 预印本 arXiv:1707.06347*，2017年。

+   Shinn 等人 [2023] Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik R Narasimhan 和 Shunyu Yao。 Reflexion：具有语言强化学习的语言代理。发表于 *NeurIPS*，2023年。

+   Shridhar 等人 [2020] Mohit Shridhar, Jesse Thomason, Daniel Gordon, Yonatan Bisk, Winson Han, Roozbeh Mottaghi, Luke Zettlemoyer 和 Dieter Fox。 ALFRED：一个用于解释日常任务的基础指令的基准。发表于 *CVPR*，2020年。

+   Shridhar 等人 [2021] Mohit Shridhar、袁星地、Marc-Alexandre Côté、Yonatan Bisk、Adam Trischler 和 Matthew J. Hausknecht。Alfworld：为互动学习对齐文本与具身环境。发表于 *ICLR*，2021。

+   Sumers 等人 [2023] Theodore Sumers、Kenneth Marino、Arun Ahuja、Rob Fergus 和 Ishita Dasgupta。将互联网规模的视觉语言模型蒸馏为具身代理。发表于 *ICML*，2023。

+   Sun 等人 [2018] 孙文、J. Andrew Bagnell 和 Byron Boots。截断视界策略搜索：结合强化学习与模仿学习。发表于 *ICLR*，2018。

+   Wang 等人 [2023a] 关志王、谢宇奇、蒋云帆、阿杰·曼德尔卡、肖超伟、朱宇科、范林熙、阿尼玛·安南库马尔。Voyager：一种基于大语言模型的开放式具身代理。*arXiv 预印本 arXiv:2305.16291*，2023a。

+   Wang 等人 [2024] 王希尧、周宇航、刘晓宇、卢洪进、徐元成、何飞鸿、尹在宏、卢泰西、Gedas Bertasius、Mohit Bansal 等人。Mementos：一个全面的基准，用于多模态大语言模型在图像序列上的推理。*arXiv 预印本 arXiv:2401.10529*，2024。

+   Wang 等人 [2023b] 王子豪、蔡绍飞、刘安基、马晓剑 和 梁亦涛。描述、解释、规划和选择：通过大语言模型的互动规划实现开放世界多任务代理。*arXiv 预印本 arXiv:2302.01560*，2023b。

+   Wei 等人 [2022] Jason Wei、王学志、Dale Schuurmans、Maarten Bosma、Brian Ichter、Fei Xia、Ed H. Chi、Quoc V. Le 和 Denny Zhou。链式思维提示激发大语言模型的推理能力。发表于 *NeurIPS*，2022。

+   Wooldridge 和 Jennings [1995] Michael Wooldridge 和 Nicholas R Jennings。智能代理：理论与实践。*知识工程评论*，10(2)：115–152，1995。

+   Wu 等人 [2023] 吴青云、Gagan Bansal、张杰宇、吴怡然、张绍昆、朱尔康、李北滨、蒋力、张晓云 和 王驰。Autogen：通过多代理对话框架实现下一代大语言模型应用。*arXiv 预印本 arXiv:2308.08155*，2023。

+   Xi 等人 [2023] 习志恒、陈文翔、郭鑫、何伟、丁一文、洪博阳、张铭、王俊哲、金森杰、周恩宇 等人。基于大语言模型的代理的崛起与潜力：一项调查。*arXiv 预印本 arXiv:2309.07864*，2023。

+   Xu 等人 [2023] 许振华、张宇佳、谢恩泽、赵震、郭永、Kenneth KY Wong、李正国 和 赵恒双。DriveGPT4：通过大语言模型实现可解释的端到端自动驾驶。*arXiv 预印本 arXiv:2310.01412*，2023。

+   Xue 等人 [2023] 薛子慧、高正奇、任思诚 和 赵杭。模态聚焦假设：迈向理解跨模态知识蒸馏。发表于 *ICLR*，2023。

+   Yang 等人 [2023a] 杨京康、董宇豪、刘帅、李博、王子越、蒋晨成、谭浩然、康佳睦、张元涵、周凯阳 等人。Octopus：从环境反馈中学习的具身视觉语言程序员。*arXiv 预印本 arXiv:2310.08588*，2023a。

+   Yang 等人 [2023b] Sherry Yang、Ofir Nachum、Yilun Du、Jason Wei、Pieter Abbeel 和 Dale Schuurmans。决策制定的基础模型：问题、方法和机会。*arXiv 预印本 arXiv:2303.04129*，2023b。

+   Yang 等人 [2022] Yijun Yang、Jing Jiang、Tianyi Zhou、Jie Ma 和 Yuhui Shi。基于模型的离线强化学习的帕累托策略池。发表于 *ICLR*，2022。

+   Yang 等人 [2023c] Yijun Yang、Tianyi Zhou、Jing Jiang、Guodong Long 和 Yuhui Shi。通过稀疏提示在元策略网络中进行持续任务分配。发表于 *ICML*，2023c。

+   Yao 等人 [2023a] Shunyu Yao、Dian Yu、Jeffrey Zhao、Izhak Shafran、Thomas L Griffiths、Yuan Cao 和 Karthik Narasimhan。思维树：使用大型语言模型进行深思熟虑的问题解决。*arXiv 预印本 arXiv:2305.10601*，2023a。

+   Yao 等人 [2023b] Shunyu Yao、Jeffrey Zhao、Dian Yu、Nan Du、Izhak Shafran、Karthik R. Narasimhan 和 Yuan Cao。React：在语言模型中协同推理和行动。发表于 *ICLR*，2023b。

+   Zheng 等人 [2023] 郑联民、蒋伟林、盛颖、庄思远、吴张昊、庄永浩、林子、李卓涵、李大成、邢埃里克 等人。通过 mt-bench 和 chatbot arena 评判 LLM 作为法官的能力。*arXiv 预印本 arXiv:2306.05685*，2023。

+   Zhong 等人 [2024] Victor Zhong、Dipendra Misra、Xingdi Yuan 和 Marc-Alexandre Côté。利用语言反馈模型的政策改进。*arXiv 预印本 arXiv:2402.07876*，2024。

+   Zhu 等人 [2023] 朱德耀、陈俊、沈晓倩、李翔和 Mohamed Elhoseiny。Minigpt-4：通过先进的大型语言模型增强视觉-语言理解。*arXiv 预印本 arXiv:2304.10592*，2023。

+   Zitkovich 等人 [2023] Brianna Zitkovich、Tianhe Yu、Sichun Xu、Peng Xu、Ted Xiao、Fei Xia、Jialin Wu、Paul Wohlhart、Stefan Welker、Ayzaan Wahid 等人。Rt-2：视觉-语言-行动模型将网页知识转移到机器人控制中。发表于 *CoRL*，2023。

## 附录

图 8：ALFWorld 的任务示例可视化[[56](https://arxiv.org/html/2311.16714v2#bib.bib56)]。该基准采用了由 Ai2Thor [[30](https://arxiv.org/html/2311.16714v2#bib.bib30)] 环境开发的多样化家庭场景，其中所有物体可以根据可放置表面区域和类别约束被重新定位到不同的位置，从而通过重新组合不同的物体和目标位置，实现大规模新任务的程序化生成。

## 6 个 LLM 专家完整提示

在本节中，我们提供了 EMMA 训练过程的所有 LLM 提示（算法 [1](https://arxiv.org/html/2311.16714v2#alg1 "算法 1 ‣ 2.3 通过跨模态模仿训练 EMMA ‣ 具身多模态代理 ‣ 通过 LLM 从并行文本世界训练的具身多模态代理")）。我们采用了 ReAct [[73](https://arxiv.org/html/2311.16714v2#bib.bib73)] 提出的提示技术，但在 EMMA 与 LLM 执行模仿学习时忽略了推理过程，即“思考”步骤。在每次试验 $i$ 后，由 LLM 批评者生成的回溯反馈 $\mathcal{P}_{i}$ 会被附加到长期记忆 $\mathcal{P}$ 中。在实际操作中，我们将 $\mathcal{P}$ 的存储反馈数量限制为最大值（通常设置为 1-3），以遵守 LLM 的最大上下文长度。

LLM 执行者生成动作的一个示例提示 {环境：你站在房间中央。快速环顾四周，你看到一个柜子 4，一个柜子 3，一个柜子 2，一个柜子 1，一个台面 1，一个垃圾桶 1，一个毛巾架 2，一个毛巾架 1，一个洗脸池 2，一个洗脸池 1，一个马桶 1，一个卫生纸架 1，以及一个毛巾架 1。任务指令：你的任务是：把一些喷雾瓶放在马桶上。 > 思考：为了完成任务，我需要找到并拿起喷雾瓶，然后将其放在马桶上。 > 思考：首先，我需要找到喷雾瓶。喷雾瓶更可能出现在柜子（1-4）、台面（1）、马桶（1）、洗脸池（1-2）、垃圾桶（1）中。我可以逐一检查，从柜子 1 开始。 > 前往柜子 1 在柜子 1 上，你看到一块布 1，一块肥皂 1，一个肥皂瓶 1。 > 前往柜子 2 柜子 2 是关闭的。 > 打开柜子 2 你打开了柜子 2。柜子 2 被打开。在里面，你看到一个蜡烛 1 和一个喷雾瓶 2。 > 思考：现在我找到了喷雾瓶（2）。接下来，我需要拿起它。 > 从柜子 2 拿起喷雾瓶 2 你从柜子 2 拿起了喷雾瓶 2。 > 思考：现在我拿到了喷雾瓶（2）。接下来，我需要把它放到马桶上。 > 前往马桶 1 在马桶 1 上，你看到一个肥皂瓶 2。 > 把喷雾瓶 2 放在马桶 1 上 你把喷雾瓶 2 放在马桶 1 上。} 少量示例 $\mathbf{\times}$ $\mathbf{n}$ 长期记忆 $\mathcal{P}$: [...] 目标环境: ... 任务指令: ... >

LLM评论员用于生成回顾的提示示例 {前次试验：你正站在一个房间的中央。快速环顾四周，你看到10号橱柜、9号橱柜、8号橱柜、7号橱柜、6号橱柜、5号橱柜、4号橱柜、3号橱柜、2号橱柜、1号橱柜、1号咖啡机、2号台面、1号台面、2号餐桌、1号餐桌、3号抽屉、2号抽屉、1号抽屉、1号冰箱、1号垃圾桶、1号微波炉、1号水槽、4号炉灶、3号炉灶、2号炉灶、1号炉灶，以及1号烤面包机。你的任务是：加热一个杯子并将其放入咖啡机中。> 去1号台面 在1号台面，你看到1把黄油刀、1个杯子、2把叉子和1个杯子。> 从1号台面拿起杯子 你从1号台面拿起杯子。> 去1号炉灶 在1号炉灶上，你看到2个平底锅。> 检查1号炉灶 在1号炉灶上，你看到2个平底锅。> 检查1号炉灶 在1号炉灶上，你看到2个平底锅。> 检查1号炉灶 在1号炉灶上，你看到2个平底锅。> 检查1号炉灶 在1号炉灶上，你看到2个平底锅。状态：失败 回顾：你陷入了一个循环，不断检查1号炉灶，而没有使用1号炉灶加热杯子1。你应该从1号台面拿起杯子1，然后用1号炉灶加热它，再把它放入1号咖啡机中。执行两个相同的操作并没有帮助。如果你再次陷入循环，你将尝试执行不同的操作。} 少量示例 $\times$ $\mathbf{n}$ 当前试验：... 回顾：

## 7 平行文本世界

尽管平行文本世界的构思深受以往工作的启发 [[55](https://arxiv.org/html/2311.16714v2#bib.bib55)，[56](https://arxiv.org/html/2311.16714v2#bib.bib56)]，我们已增强了TextWorld引擎，创建了每个视觉环境的文本版等价物，用于训练和评估基于语言的智能体。此增强涉及利用PDDL [[1](https://arxiv.org/html/2311.16714v2#bib.bib1)]和Fast Downward [[22](https://arxiv.org/html/2311.16714v2#bib.bib22)]的组合来维护和更新模拟环境的文本状态。根据模拟器提供的元数据，我们将视觉状态表示为一系列属性。这些属性详细描述了环境中各个实体之间的关系，如位置、目标和物体。请注意，所有这些属性、变量和规则都在PDDL框架内定义。

## 8 训练细节

我们在表[1](https://arxiv.org/html/2311.16714v2#S8.T1 "Table 1 ‣ 8 Training Details ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")中提供了用于训练 EMMA 的超参数。这些超参数大部分来源于用于微调 InstructBLIP 模型的建议[[11](https://arxiv.org/html/2311.16714v2#bib.bib11)]。在训练过程中，我们只更新线性投影层的参数，并保持其他组件冻结，正如许多现有工作的指令调优所做的那样[[76](https://arxiv.org/html/2311.16714v2#bib.bib76), [17](https://arxiv.org/html/2311.16714v2#bib.bib17)]。我们使用 AdamW 优化器[[39](https://arxiv.org/html/2311.16714v2#bib.bib39)]，学习率线性预热，随后进行线性衰减，最小学习率为 0。此外，我们去除了 Q-Former 中的指令输入，这在 InstructBLIP 中是必需的，并且我们发现这样做能提升所有实验的性能。我们的实现深受 LAVIS 库[[33](https://arxiv.org/html/2311.16714v2#bib.bib33)]的启发，因此训练和评估过程使用了 LAVIS 提供的标准程序。

表 1：EMMA 在 ALFWorld 实验中的超参数

| 超参数 |  | 值 |
| --- | --- | --- |
| EMMA 的架构 |
| LLM 解码器 |  | Vicuna-7b-v1.1 [[74](https://arxiv.org/html/2311.16714v2#bib.bib74)] |
| 图像编码器 |  | ViT-L [[46](https://arxiv.org/html/2311.16714v2#bib.bib46)] |
| Q-Former |  | $\text{BERT}_{\text{base}}$ [[12](https://arxiv.org/html/2311.16714v2#bib.bib12)] |
| 预训练权重 |  | InstructBLIP [[11](https://arxiv.org/html/2311.16714v2#bib.bib11)] |
| 查询标记数量 |  | $32$ |
| Q-Former 文本输入 |  | False |
| 最大文本长度 |  | 1024 |
| 图像分辨率 |  | 224 |
| 行为克隆 |
| 微调周期数 |  | $6$ |
| 预热步骤 |  | $1000$ |
| 学习率 |  | $10^{-5}$ |
| 批量大小 |  | $128$ |
| AdamW $\beta$ |  | $(0.9,0.999)$ |
| 权重衰减 |  | $0.05$ |
| 路径丢弃 |  | $0$ |
| 推理波束大小 |  | $5$ |
| 模仿学习 |
| LLM 专家的基础模型 |  | text-davinci-003 |
| LLM 专家的提示 |  | 参见第[6](https://arxiv.org/html/2311.16714v2#S6 "6 Full Prompts for LLM Expert ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")节 |
| 试验次数 |  | $12$ |
| 回合长度 |  | $30$ |
| 长期记忆大小 |  | $3$ |
| 学习率 |  | $5\times 10^{-6}$ |
| 预热步骤 |  | $300$ |
| 批量大小 |  | $16$ |
| 每次试验的训练周期数 |  | $5$ |
| DPO $\beta$ |  | $0.1$ |

图 9：EMMA 与 LLM 专家的成功率对比。随着试验次数的增加，两者之间的差距逐渐缩小，EMMA 在一些任务（例如“热处理与放置”和“冷处理与放置”）中甚至超越或与专家表现相当，这表明跨模态模仿学习的有效性。

图 10：消融研究。“EMMA w/o BC 初始化”的表现始终差于原始的 EMMA。

图 11：词汇分布。所有单词在人工标注和模板化任务指令中的频率分布。人工标注指令的多样性对代理的泛化能力提出了重大挑战。

## 9 演示数据集的收集

通过行为克隆在预收集的演示数据集上微调预训练的VLM是一个关键步骤，使这些模型能够理解并遵循ALFWorld的独特语法，并发展出基本的“游戏感”。然而，原始ALFWorld中的任务指令[[55](https://arxiv.org/html/2311.16714v2#bib.bib55)]数量过于有限，无法有效地为微调这些大型预训练VLM提供足够的数据。因此，我们提出了一种自动化管道，利用text-davinci-003和基于规则的规划器分别生成大量新的指令及其相应的专家演示。

为了生成多样化的新任务指令，我们利用了LLM的上下文学习能力。我们的程序首先从ALFWorld基准中提取每个环境的详细描述，提供关于所有物品的数量和功能属性的全面信息。然后，根据这些环境中房间的类型，我们设计了不同的提示，旨在引导LLM生成与目标环境特征一致的任务指令。这些提示的示例如表[2](https://arxiv.org/html/2311.16714v2#S9.T2 "Table 2 ‣ 9 Collection of Demonstration Dataset ‣ Embodied Multi-Modal Agent trained by an LLM from a Parallel TextWorld")所示。对于每个生成的任务指令，我们使用ALFWorld设计的基于规则的规划器收集示范数据集 $\{x_{\text{task}},s^{t}_{v},x^{t}_{a}\}_{t=0}^{T}$。需要注意的是，这个规划器具有不公平的优势：它认为环境是完全可观察的，并且拥有世界动态的完整信息，依赖于在训练过程中代理无法访问的元数据。总的来说，我们的数据集包含15247个专家演示集，共计178585对图像-文本。

表 2：厨房中新任务指令生成提示的示例

| 问： |
| --- |
| 环境：你处于房间中央。快速环顾四周，你看见了一个橱柜，一个台面，一个橱柜，一个台面，一个抽屉，一个抽屉，一个抽屉，一个炉灶，一个炉灶，一个抽屉，一个炉灶，一个炉灶，一个橱柜，一个橱柜，一个微波炉，一个橱柜，一个橱柜，一个橱柜，一个水槽，一个水槽盆，一个冰箱，一个烤面包机，一个咖啡机，一个橱柜，一个抽屉，一个抽屉，一个抽屉，一个抽屉，一个架子，一个架子，一个台面，一个架子，一个抽屉，和一个垃圾桶。 |
| 物品字典：所有可操作物品都列在以下字典中，格式一致 {操作类型: {物品名称: 物品数量}}: {’pickupable’: {’dishsponge’: 3, ’apple’: 2, ’butterknife’: 3, ’peppershaker’: 2, ’saltshaker’: 3, ’bowl’: 2, ’spatula’: 2, ’pot’: 3, ’winebottle’: 3, ’statue’: 2, ’creditcard’: 3, ’plate’: 2, ’pan’: 2, ’kettle’: 3, ’soapbottle’: 3, ’potato’: 3, ’fork’: 2, ’bread’: 2, ’knife’: 3, ’glassbottle’: 3, ’book’: 1, ’tomato’: 1, ’vase’: 2, ’egg’: 1, ’papertowelroll’: 1, ’cup’: 1, ’lettuce’: 1, ’spoon’: 1, ’mug’: 1}, ’sliceable’: {’apple’: 2, ’potato’: 3, ’bread’: 2, ’tomato’: 1, ’egg’: 1, ’lettuce’: 1}, ’receptacle’: {’bowl’: 2, ’pot’: 3, ’plate’: 2, ’pan’: 2, ’stoveburner’: 4, ’drawer’: 9, ’countertop’: 3, ’cabinet’: 9, ’microwave’: 1, ’shelf’: 3, ’toaster’: 1, ’garbagecan’: 1, ’cup’: 1, ’fridge’: 1, ’coffeemachine’: 1, ’sinkbasin’: 1, ’mug’: 1}, ’canFillWithLiquid’: {’bowl’: 2, ’pot’: 3, ’winebottle’: 3, ’kettle’: 3, ’glassbottle’: 3, ’houseplant’: 1, ’cup’: 1, ’mug’: 1}, ’breakable’: {’bowl’: 2, ’winebottle’: 3, ’statue’: 2, ’plate’: 2, ’glassbottle’: 3, ’window’: 1, ’vase’: 2, ’egg’: 1, ’cup’: 1, ’mug’: 1}, ’dirtyable’: {’bowl’: 2, ’pot’: 3, ’plate’: 2, ’pan’: 2, ’cup’: 1, ’mug’: 1}, ’openable’: {’kettle’: 3, ’drawer’: 9, ’cabinet’: 9, ’book’: 1, ’microwave’: 1, ’fridge’: 1}, ’cookable’: {’potato’: 3}, ’toggleable’: {’stoveknob’: 4, ’microwave’: 1, ’toaster’: 1, ’coffeemachine’: 1, ’lightswitch’: 1, ’faucet’: 1}} |
| 基于环境描述和物品字典生成 30 个多样化的任务。 |
| A: |
| 任务 1: 拿起清洁物品并放入容器，将一个清洁的锅放入冰箱 |
| 任务 2: 拿起并放置简单物品，将一个苹果放在台面上 |
| 任务 3: 拿起两个物品并放置，将两个苹果放入垃圾桶 |
| 任务 4: 拿起加热物品并放入容器，将一个热苹果放在盘子上 |
| 任务 5: 拿起冷却物品并放入容器，将一个凉的苹果放在台面上 |
| 任务 6: 拿起并放置简单物品，将一个马克杯放入咖啡机 |
| 任务 7: 拿起两个物品并放置，将两张信用卡放在盘子上 |
| 任务 8: 拿起清洁物品并放入容器，将一个清洁的马克杯放入咖啡机 |
| 任务 9: 拿起加热物品并放入容器，将一个热马克杯放在盘子上 |
| 任务 10: 拿起冷却物品并放入容器，将一个凉的马克杯放在盘子上 |
| 任务 11: 拿起并放置简单物品，将一个胡椒粉瓶放入橱柜 |
| 任务 12: 拿起两个物品并放置，将两个胡椒粉瓶放入架子 |
| 任务 13: 拿起清洁物品并放入容器，将一个清洁的叉子放入锅中 |
| 任务 14: 拿起加热物品并放入容器，将一个热叉子放在碗上 |
| 任务 15: 拿起冷却物品并放入容器，将一个凉的叉子放在盘子上 |
| 任务 16: 拿起并放置简单物品，将一个雕像放在台面上 |
| 任务 17: 拿起两个物品并放置，将两个碗放入微波炉 |
| 任务 18: 拿起清洁物品并放入容器，将一个清洁的土豆放入冰箱 |
| 任务 19: 拿起加热物品并放入容器，将一个热土豆放在盘子上 |
| 任务 20: 拿起冷却物品并放入容器，将一个凉的土豆放入锅中 |
| 任务 21: pick_and_place_simple，将一个鸡蛋放在台面上 |
| 任务 22: pick_two_obj_and_place，将两个面包放入微波炉 |
| 任务 23: pick_clean_then_place_in_recep，将一个干净的鸡蛋放入垃圾桶 |
| 任务 24: pick_heat_then_place_in_recep，将一个热鸡蛋放入碗中 |
| 任务 25: pick_cool_then_place_in_recep，将一个凉的鸡蛋放入平底锅 |
| 任务 26: pick_and_place_simple，将一个平底锅放入炉灶 |
| 任务 27: pick_two_obj_and_place，将两个锅放入炉灶 |
| 任务 28: pick_clean_then_place_in_recep，将一个干净的番茄放入咖啡机 |
| 任务 29: pick_heat_then_place_in_recep，将一个热番茄放在盘子上 |
| 任务 30: pick_cool_then_place_in_recep，将一个凉的番茄放入盘子 |
| Q: |
| 环境: ... |
| 物体词典: ... |
| 根据环境描述和物体词典生成 30 个多样化的任务。 |
| A: |
| ...LLM 生成的任务指令... |
