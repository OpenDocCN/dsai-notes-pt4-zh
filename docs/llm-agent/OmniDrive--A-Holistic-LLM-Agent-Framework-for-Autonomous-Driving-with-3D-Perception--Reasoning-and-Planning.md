<!--yml

分类：未分类

日期：2025-01-11 12:39:59

-->

# OmniDrive：一个面向自动驾驶的整体LLM-代理框架，具有3D感知、推理和规划功能

> 来源：[https://arxiv.org/html/2405.01533/](https://arxiv.org/html/2405.01533/)

\floatsetup

[table]capposition=top \newfloatcommandcapbtabboxtable[][\FBwidth] (eccv) 包包eccv 警告：‘hyperref’包使用了‘pagebackref’选项，该选项对于最终定稿版本*不*推荐使用

¹¹institutetext: ¹ 北京理工大学, ² NVIDIA, ³ 华中科技大学

[https://github.com/NVlabs/OmniDrive](https://github.com/NVlabs/OmniDrive)王士浩 在NVIDIA实习期间完成的工作。11    余智定 通讯作者：[zhidingy@nvidia.com](mailto:zhidingy@nvidia.com)。22    蒋晓辉 11    兰世奕 22    石敏^∗ 33    张娜迪 22    贾恩·考茨 22    李颖 11    何塞·M·阿尔瓦雷斯 22

###### 摘要

多模态大型语言模型（MLLMs）的进展引发了对基于LLM的自动驾驶的兴趣，旨在利用其强大的推理能力。然而，充分利用MLLM的强大推理能力来改进规划行为是具有挑战性的，因为这需要超越2D推理的完整3D情境感知。为了解决这一挑战，我们的工作提出了OmniDrive，这是一个实现代理模型与3D驾驶任务强对齐的整体框架。我们的框架始于一种创新的3D MLLM架构，采用稀疏查询将视觉表示提升并压缩为3D，然后输入LLM。基于查询的表示使我们能够共同编码动态物体和静态地图元素（如交通车道），提供一个紧凑的世界模型，用于3D中的感知-行动对齐。我们还提出了一个新基准，包含全面的视觉问答（VQA）任务，包括场景描述、交通规则、3D定位、反事实推理、决策和规划。广泛的研究表明OmniDrive在复杂3D场景中展现了卓越的推理和规划能力。

###### 关键词：

自动驾驶 大型语言模型 规划

图1：我们提出了OmniDrive，这是一个面向端到端自动驾驶的创新框架，采用大型语言模型（LLM）代理。我们的主要贡献在于模型（OmniDrive-Agent）和基准（OmniDrive-nuScenes）方面的创新解决方案。前者具有创新的3D视觉-语言模型设计，后者则由全面的视觉问答任务（VQA）组成，用于推理和规划。

## 1 引言

最近，多模态大语言模型（MLLMs）的快速发展[[1](https://arxiv.org/html/2405.01533v1#bib.bib1), [24](https://arxiv.org/html/2405.01533v1#bib.bib24), [31](https://arxiv.org/html/2405.01533v1#bib.bib31)]及其卓越的推理能力，已催生了一系列端到端自动驾驶应用[[46](https://arxiv.org/html/2405.01533v1#bib.bib46), [58](https://arxiv.org/html/2405.01533v1#bib.bib58), [52](https://arxiv.org/html/2405.01533v1#bib.bib52), [39](https://arxiv.org/html/2405.01533v1#bib.bib39), [6](https://arxiv.org/html/2405.01533v1#bib.bib6)]。然而，将能力从二维理解扩展到三维空间的复杂性，仍然是完全释放其在现实世界应用潜力的关键难题。理解和导航三维空间对于自动驾驶汽车（AVs）至关重要，因为这直接影响到自动驾驶汽车做出明智决策、预测未来状态并与环境安全互动的能力。尽管先前的研究[[38](https://arxiv.org/html/2405.01533v1#bib.bib38), [47](https://arxiv.org/html/2405.01533v1#bib.bib47)]已展示了LLM代理在自动驾驶中的成功应用，但仍需一种整体且有原则的方法，将MLLMs的二维理解和推理能力完全扩展到复杂的三维场景中，以理解三维几何形状和空间关系。

另一个亟待解决的问题是如何处理多视角高分辨率视频输入。一方面，许多当前流行的二维MLLM架构，如LLaVA-1.5[[31](https://arxiv.org/html/2405.01533v1#bib.bib31), [30](https://arxiv.org/html/2405.01533v1#bib.bib30)]，由于视觉编码器分辨率和LLM令牌序列长度的限制，只能处理$336\times 336$的图像输入。增加这些限制并非易事，因为这需要显著更多的计算和内存。另一方面，处理高分辨率视频输入，通常甚至是多视角视频，是远程自动驾驶感知和安全决策的基本需求。然而，与许多基于云的服务不同，现实世界中的工业自动驾驶应用大多是设备端的且受限于计算能力。因此，需要设计一种高效的MLLM架构，在将输入提供给LLM之前，压缩三维视觉表示。

我们对上述挑战的回答是提出了一种新颖的Q-Former风格的[[24](https://arxiv.org/html/2405.01533v1#bib.bib24)] 3D MLLM架构，如[图1](https://arxiv.org/html/2405.01533v1#S0.F1 "在OmniDrive中：一个面向自动驾驶的全面LLM-Agent框架，结合3D感知、推理和规划")所示。与采用自注意力设计的LLaVA不同，Q-Former中的跨注意力解码器通过将视觉信息压缩为稀疏查询，使其能够更好地扩展至更高分辨率的输入。有趣的是，我们注意到Q-Former架构与透视视图模型家族具有相当的相似性，如DETR3D[[53](https://arxiv.org/html/2405.01533v1#bib.bib53)]、PETR(v2)[[32](https://arxiv.org/html/2405.01533v1#bib.bib32)、[33](https://arxiv.org/html/2405.01533v1#bib.bib33)]、StreamPETR[[50](https://arxiv.org/html/2405.01533v1#bib.bib50)]和Far3D[[21](https://arxiv.org/html/2405.01533v1#bib.bib21)]。通过使用稀疏的3D查询，这些模型在与密集鸟瞰视图（BEV）表示相比，展现了显著的优势，具有领先的性能[[50](https://arxiv.org/html/2405.01533v1#bib.bib50)、[21](https://arxiv.org/html/2405.01533v1#bib.bib21)]、远程感知[[21](https://arxiv.org/html/2405.01533v1#bib.bib21)]以及联合建模地图元素的能力[[55](https://arxiv.org/html/2405.01533v1#bib.bib55)]。查询驱动解码器架构的相似性使我们能够通过将3D位置编码附加到查询中，将其提升至3D，并处理多视图输入，从而将两者对接，如[图1](https://arxiv.org/html/2405.01533v1#S0.F1 "图1 ‣ OmniDrive：一个面向自动驾驶的全面LLM-Agent框架，结合3D感知、推理和规划")的左侧部分所示。这个过程使得MLLM能够以最小的努力和改动获得3D空间理解，同时利用2D中丰富图像中的预训练知识。

除了模型架构，近期的自动驾驶LLM智能体工作也强调了基准的重要性[[43](https://arxiv.org/html/2405.01533v1#bib.bib43), [38](https://arxiv.org/html/2405.01533v1#bib.bib38), [46](https://arxiv.org/html/2405.01533v1#bib.bib46), [39](https://arxiv.org/html/2405.01533v1#bib.bib39), [10](https://arxiv.org/html/2405.01533v1#bib.bib10), [47](https://arxiv.org/html/2405.01533v1#bib.bib47)]。其中许多作为问答（QA）数据集呈现，用于训练和基准测试LLM智能体的推理或规划能力。尽管有各种不同的QA设置，但涉及规划的基准[[46](https://arxiv.org/html/2405.01533v1#bib.bib46), [10](https://arxiv.org/html/2405.01533v1#bib.bib10), [47](https://arxiv.org/html/2405.01533v1#bib.bib47)]仍然依赖于现实世界的开放环路设置（例如，nuScenes），其中使用专家轨迹。近期的研究[[60](https://arxiv.org/html/2405.01533v1#bib.bib60), [26](https://arxiv.org/html/2405.01533v1#bib.bib26)]揭示了开放环路评估的局限性，表现为对自车状态的隐性偏向、过于简单的规划场景，以及容易对专家轨迹的过拟合。针对这个问题，提出的基准OmniDrive-nuScenes提供了一个反事实推理基准设置，利用模拟的决策和轨迹推理潜在的后果。我们的基准还包括其他具有挑战性的任务，要求具备完整的3D空间理解和长期推理，如图[1](https://arxiv.org/html/2405.01533v1#S0.F1 "Figure 1 ‣ OmniDrive: A Holistic LLM-Agent Framework for Autonomous Driving with 3D Perception, Reasoning and Planning")（右侧）所示。

总结来说，OmniDrive旨在提供一个全面的框架，以实现端到端的自动驾驶和LLM智能体。我们提出了一种原则性的模型设计，优秀地处理3D推理和规划，并且提供了一个更加具有挑战性的基准，超越了单一专家轨迹。

## 2 OmniDrive-Agent

总结来说，我们的目标是统一的3D MLLM设计，具体包括：1）利用2D MLLM的预训练知识，和2）解决自动驾驶中的高分辨率多视角输入问题。我们提出了一种Q-Former风格的架构，通过在输入LLM之前将视觉特征压缩为固定数量的查询[[24](https://arxiv.org/html/2405.01533v1#bib.bib24)]。鉴于Q-Former与基于查询的3D感知框架之间的相似性，我们将我们的MLLM架构与StreamPETR[[50](https://arxiv.org/html/2405.01533v1#bib.bib50)]对齐，使用查询来编码动态物体和静态地图元素。这些查询与额外的承载标记一起，作为压缩的世界模型，使感知与推理和规划对齐。

### 2.1 前提

基于Q-Former的MLLMs由一个通用视觉编码器组成，用于提取单视角图像特征$F_{s}\in\mathbb{R}^{C\times H\times W}$，一个作为视觉-语言对齐模块的投影器（Q-Former），以及一个用于文本生成的大型语言模型。投影器的架构由堆叠的Transformer解码器层组成。从图像特征到文本嵌入的投影过程可以表示为：

|  | $\displaystyle\tilde{Q}_{t}=f_{q}(Q_{t},{F}_{s})$ |  | (1) |
| --- | --- | --- | --- |

其中$Q_{t}$是初始化的文本嵌入。$\tilde{Q}_{t}$是精炼后的文本嵌入，之后将其发送到语言模型中生成最终的文本输出。

基于查询的3D感知模型 [[55](https://arxiv.org/html/2405.01533v1#bib.bib55), [21](https://arxiv.org/html/2405.01533v1#bib.bib21), [28](https://arxiv.org/html/2405.01533v1#bib.bib28), [29](https://arxiv.org/html/2405.01533v1#bib.bib29)]由一个共享的视觉编码器组成，用于提取多视角图像特征，以及一个检测头$f_{d}$。该模型基于PETR [[32](https://arxiv.org/html/2405.01533v1#bib.bib32)]，利用Transformer解码器架构有效地将多视角图像特征$F_{m}\in\mathbb{R}^{N\times C\times H\times W}$转换为检测查询$\tilde{Q}_{d}$，其公式为：

|  | $\displaystyle\tilde{Q}_{d}=f_{d}(Q_{d},F_{m}+P_{m})$ |  | (2) |
| --- | --- | --- | --- |

其中$P_{m}$是有效捕捉图像视角与3D领域之间几何关系的3D位置编码。$Q_{d}$是初始化的检测查询，用于聚合多视角图像特征。

图2：OmniDrive-Agent的整体流程图。左侧图示展示了模型的整体框架。我们使用一个3D感知任务来指导Q-Former的学习。右侧图示展示了Q-Former3D的具体结构，该结构由六个Transformer解码器层组成。注意力权重来自2D预训练初始化。输入是多视角图像特征。此外，在注意力操作中添加了3D位置编码。进一步地，我们通过记忆库引入了时序建模。

可以观察到，Q-Former中的Transformer解码器与稀疏查询驱动的3D感知模型（以StreamPETR [[50](https://arxiv.org/html/2405.01533v1#bib.bib50)]为代表）在架构设计上非常相似。为了增强MLLMs的定位能力，我们考虑引入3D位置编码设计以及基于查询的感知模型监督。

### 2.2 总体架构

如图[2](https://arxiv.org/html/2405.01533v1#S2.F2 "图 2 ‣ 2.1 基础 ‣ 2 OmniDrive-Agent ‣ OmniDrive：一个集成的 LLM-Agent 框架，用于具备 3D 感知、推理与规划的自动驾驶")所示，Omnidrive 首先使用共享的视觉编码器提取多视角图像特征 $F_{m}\in\mathbb{R}^{N\times C\times H\times W}$。提取的特征与位置编码 $P_{m}$ 一同输入 Q-Former3D。在 Q-Former3D 中，我们初始化检测查询和载体查询，并执行自注意力机制以交换它们的信息，具体可通过以下公式总结：

|  | $\displaystyle(Q,K,V)=(\textbf{[}Q_{c},{Q}_{d}\textbf{]},\textbf{[}Q_{c},{Q}_{d}\% \textbf{]},\textbf{[}Q_{c},{Q}_{d}\textbf{]}),$ |  | (3) |
| --- | --- | --- | --- |
|  | $\displaystyle\tilde{Q}=\text{多头注意力}(Q,K,V)$ |  |

$\textbf{[}\cdot\textbf{]}$ 是连接操作。为了简便起见，我们省略了位置编码。然后，这些查询从多视角图像中收集信息：

|  | $\displaystyle(Q,K,V)=(\textbf{[}Q_{c},{Q}_{d}\textbf{]},P_{m}+F_{m},F_{m}),$ |  | (4) |
| --- | --- | --- | --- |
|  | $\displaystyle\tilde{Q}=\text{多头注意力}(Q,K,V)$ |  |

之后，感知查询用于预测前景元素的类别和坐标。载体查询被发送到一个单层 MLP，以便与 LLM tokens 的维度对齐（例如，LLaMA 中是 4096 维[[48](https://arxiv.org/html/2405.01533v1#bib.bib48)]），并进一步用于文本生成，遵循 LLaVA [[31](https://arxiv.org/html/2405.01533v1#bib.bib31)]。

在我们的模型中，载体查询起到视觉-语言对齐的作用。此外，这一设计使得载体查询能够利用 3D 位置编码提供的几何先验，同时也能利用通过 3D 感知任务获得的基于查询的表示。

### 2.3 多任务与时序建模

我们的方法受益于多任务学习和时间建模[[33](https://arxiv.org/html/2405.01533v1#bib.bib33)，[25](https://arxiv.org/html/2405.01533v1#bib.bib25)]。在多任务学习中，我们可以为每个感知任务集成特定任务的Q-Former3D模块，并采用统一的初始化策略（请参见[第2.4节](https://arxiv.org/html/2405.01533v1#S2.SS4 "2.4 Training Strategy ‣ 2 OmniDrive-Agent ‣ OmniDrive: A Holistic LLM-Agent Framework for Autonomous Driving with 3D Perception, Reasoning and Planning")）。在不同任务中，载体查询可以收集不同交通元素的信息。在我们的实现中，我们涵盖了中心线构建和3D物体检测等任务。在训练和推理阶段，两个头共享相同的3D位置编码。关于时间建模，我们将具有top-k分类分数的感知查询存储到内存库中，并逐帧传播它们[[28](https://arxiv.org/html/2405.01533v1#bib.bib28)，[59](https://arxiv.org/html/2405.01533v1#bib.bib59)]。传播的查询通过交叉注意力与当前帧的感知查询和载体查询进行交互，从而扩展了我们模型的能力，有效处理视频输入。

### 2.4 训练策略

OmniDrive-agent的训练包括两个阶段：2D预训练和3D微调。在初始阶段，我们在2D图像任务上预训练MLLMs，以初始化Q-Former和载体查询。接下来，模型在与3D相关的驾驶任务（例如，运动规划、3D定位等）上进行微调。在这两个阶段，我们计算文本生成损失时不考虑BLIP-2中的对比学习和匹配损失[[24](https://arxiv.org/html/2405.01533v1#bib.bib24)]。

2D预训练。2D预训练阶段旨在预训练载体查询和Q-Former，并实现图像特征与大语言模型之间的更好对齐。去除检测查询后，OmniDrive模型可以视为一个标准的视觉语言模型，能够生成基于图像的文本。因此，我们采用LLaVA v1.5的训练食谱和数据[[30](https://arxiv.org/html/2405.01533v1#bib.bib30)]，在2D图像上对OmniDrive进行预训练。首先，MLLMs在558K图像-文本对上进行训练，在此过程中，除了Q-Former之外的所有参数都被冻结。随后，我们使用LLaVA v1.5的指令调优数据集对MLLMs进行微调。在这一微调步骤中，只有图像编码器被冻结，其他所有参数均可训练。

3D微调。在3D微调阶段，我们的目标是增强模型的3D定位能力，同时尽可能保留其2D语义理解。我们在原有的Q-Former基础上增加了3D位置编码和时间模块。在这个阶段，我们使用小学习率对视觉编码器和大语言模型进行Lora微调[[16](https://arxiv.org/html/2405.01533v1#bib.bib16)]，并用相对较大的学习率训练Q-Former3D。

## 3 OmniDrive-nuScenes

为了基准测试驱动型LLM代理，我们提出了OmniDrive-nuScenes，这是一个基于nuScenes [[4](https://arxiv.org/html/2405.01533v1#bib.bib4)]的新型基准，包含高质量的视觉问答（QA）对，覆盖3D领域中的感知、推理和规划。

OmniDrive-nuScenes具有一个完全自动化的过程化QA生成管道，使用GPT4。与LLaVA [[31](https://arxiv.org/html/2405.01533v1#bib.bib31)]类似，所提议的管道通过提示输入将3D感知的地面真值作为上下文信息。交通规则和规划仿真进一步作为额外输入，缓解了GPT-4V在理解3D环境时面临的挑战。我们的基准测试提出了长时间跨度的问题，涉及注意力、反事实推理和开环规划。这些问题挑战了3D空间中的真正空间理解和规划能力，因为它们要求进行规划仿真，以在接下来的几秒钟内获得正确答案。

除了使用上述管道为OmniDrive-nuScenes策划离线问答会话外，我们进一步提出了一个管道，用于在线生成各种类型的定位问题。这一部分也可以看作是某种形式的隐式数据增强，以增强模型的3D空间理解和推理能力。

### 3.1 离线问答

<svg class="ltx_picture" height="592.16" id="S3.T1.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,592.16) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="564.61" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">|   |  |

| 提示类型 1：标题 |  |
| --- | --- |
| 图像描绘了一个白天的场景，位于一个受控访问区，可能是一个停车场或商业或工业设施的服务入口。左侧是一个有树木的草地区域，以及一座带有蓝色装饰的建筑物。向中间移动时，我们看到一个设有升降栏杆的安全检查点，表明这是一个入口或出口点，还有一个安保亭……<省略> |  |
| 提示类型 2：车道-物体关联 |  |
| &#124;— 你当前的直行车道 [(-2.6, +0.5), (+1.2, +0.7), (+5.0, +0.9), (+8.8, +1.0)] |  |
| &#124; &#124;— 你自己的车 |  |
| &#124; &#124;— 可移动物体.交通锥位于前方位置：(+8.2, +2.4)…<省略> |  |
| 提示类型 3：模拟决策和轨迹 |  |
| 模拟决策：适度速度，左转 |  |
| 模拟轨迹：[PT, (+4.85, -0.08), (+9.71, -0.22), …, (+27.42, -0.93)]。超出可行驶区域 |  |
| 提示类型 4：专家决策和轨迹 |  |
| 专家决策：慢速行驶，保持车道，直行 |  |
| 专家轨迹：[PT, (+0.76, +0.02), (+1.45, +0.03), …, (+3.44, +0.12)]。 |  |
| 你行驶路径附近的物体：车辆.移动位置为 (-6.0, +0.3)，行人.步行.移动位置为 (+4.4, -7.2)。 |  |
| 对话类型 1：注意力 |  |
| 问题：在这个区域驾驶时，我应该注意哪些交通元素？ |  |
| 回答：你的路径上有多个可移动的交通锥：一个直接位于前方位置 (+8.2, +2.4)，其他的在周围车道中，如 (+4.7, -6.3) 处的锥体….<省略> |  |
| 对话类型 2：反事实推理 |  |
| 问题：如果我决定加速并左转，可能会有什么后果？ |  |
| 回答：看来你将转入允许从右到左行驶的车道。这一动作可能会导致与迎面而来的交通发生碰撞…<省略> |  |
| 对话类型 3：决策制定与规划 |  |
| 问题：根据当前的驾驶情况，我下一步该怎么做，为什么？ |  |
| 回答：最合适的轨迹应该类似于提供的安全轨迹，即 [PT, (+0.76, +0.02), (+1.45, +0.03),…,(+3.44, +0.12)]。该轨迹确保你保持在当前的直行车道 [(-2.6, +0.5), (+1.2, +0.7), …]，避免任何障碍物…<省略> |  |
| 其他对话： |  |

| …<省略> |  |</foreignobject></g></g></svg>

表格 1：展示用于离线问答的数据生成示例。顶部块展示了用来提示GPT4的上下文信息，如标题和框架，底部块展示了四种类型的回应。请注意，除生成标题外，视觉图像不会用于提示GPT4。

表格 [1](https://arxiv.org/html/2405.01533v1#S3.T1 "表格 1 ‣ 3.1 离线问答。 ‣ 3 OmniDrive-nuScenes ‣ OmniDrive：一个针对自动驾驶的全局LLM-Agent框架，具有3D感知、推理和规划") 显示了所提出的离线数据生成管道的示例，其中上下文信息用于在nuScenes上生成问答对。我们首先提供与如何获取不同类型提示信息的相关细节：

图像说明。当图像和大量场景信息同时输入到GPT-4V时，GPT-4V倾向于忽略图像中的细节。因此，我们首先提示GPT-4V仅根据多视角输入生成场景描述。如表[1](https://arxiv.org/html/2405.01533v1#S3.T1 "Table 1 ‣ 3.1 Offline Question-Answering. ‣ 3 OmniDrive-nuScenes ‣ OmniDrive: A Holistic LLM-Agent Framework for Autonomous Driving with 3D Perception, Reasoning and Planning")所示，我们将三种正面视角和三种后视角拼接成两张独立的图像，并将其输入到GPT-4V中，如表[1](https://arxiv.org/html/2405.01533v1#S3.T1 "Table 1 ‣ 3.1 Offline Question-Answering. ‣ 3 OmniDrive-nuScenes ‣ OmniDrive: A Holistic LLM-Agent Framework for Autonomous Driving with 3D Perception, Reasoning and Planning")顶部所示。我们还提示GPT-4V包含以下细节：1）提到天气、时间、场景类型以及其他图像内容；2）理解每个视角的总体方向（例如，第一个正面视角为前左）；3）避免单独提及每个视角的内容，而是用相对于自车的位置来替代。

车道-物体关联。对于GPT-4V来说，理解交通元素（如物体、车道线等）在3D世界中的相对空间关系是一项极具挑战的任务。直接将3D物体坐标和车道线的曲线表示输入到GPT-4V中无法有效进行推理。因此，我们以文件树的形式表示物体与车道线之间的关系，并根据物体的3D边界框将物体信息转换为自然语言描述。

模拟轨迹。轨迹通过两种方式进行反事实推理采样：1）我们基于三种驾驶意图选择初始车道：保持车道、左侧变道和右侧变道。然后使用深度优先搜索（DFS）算法连接车道中心线，并获得所有可能的车辆轨迹路径。接着为不同的车道（加速、减速和保持速度）选择不同的完成率和速度目标，以创建模拟轨迹。2）仅根据车道中心线生成驾驶轨迹使得模拟‘不可驾驶’场景变得困难。因此，我们还对整个nuScenes数据集中的自车轨迹进行了聚类，每次选择具有代表性的驾驶路径。

专家轨迹。这是来自nuScenes的日志重放轨迹。专家轨迹被分类为不同类型，用于高层次决策。我们还将一个物体识别为“接近”，如果它在接下来的3秒内与轨迹的最小距离小于10米。接近的物体随后会列在专家轨迹下方。

在表格 [1](https://arxiv.org/html/2405.01533v1#S3.T1 "Table 1 ‣ 3.1 Offline Question-Answering. ‣ 3 OmniDrive-nuScenes ‣ OmniDrive: A Holistic LLM-Agent Framework for Autonomous Driving with 3D Perception, Reasoning and Planning") 的底部块中，我们描述了使用上述上下文信息获得的不同类型的问答响应：

场景描述。我们直接将字幕（表格 [1](https://arxiv.org/html/2405.01533v1#S3.T1 "Table 1 ‣ 3.1 Offline Question-Answering. ‣ 3 OmniDrive-nuScenes ‣ OmniDrive: A Holistic LLM-Agent Framework for Autonomous Driving with 3D Perception, Reasoning and Planning") 中的提示类型 1）作为场景描述的答案。

注意。给定模拟轨迹和专家轨迹，运行仿真以识别靠近的物体。同时，我们还允许 GPT-4 使用其常识来识别具有威胁的交通元素。

反事实推理。给定模拟轨迹，我们模拟检查轨迹是否违反交通规则，例如闯红灯、与其他物体或道路边界发生碰撞。

决策和规划。我们展示了高层次的决策制定以及专家轨迹，并使用 GPT-4V 推理给定先前的提示和响应信息作为背景，解释为什么该轨迹是安全的。

一般对话。我们还提示 GPT-4 基于字幕信息和图像内容生成多轮对话，涉及物体计数、颜色、相对位置和 OCR 类型任务。我们发现这种方法有助于提高模型对长尾物体的识别。

### 3.2 在线问答。

图 3：在线问答生成。

为了充分利用自动驾驶数据集中的 3D 感知标签，我们在训练过程中在线生成大量类似地面任务的任务。具体来说，以下任务（如图 [3](https://arxiv.org/html/2405.01533v1#S3.F3 "Figure 3 ‣ 3.2 Online Question-Answering. ‣ 3 OmniDrive-nuScenes ‣ OmniDrive: A Holistic LLM-Agent Framework for Autonomous Driving with 3D Perception, Reasoning and Planning") 所示）被设计出来：

2D 到 3D 地面定位。给定特定摄像机上的 2D 边界框，例如 $<\text{FRONT},0.45,0.56,0.72,0.87>$，要求模型提供相应物体的 3D 属性，包括其 3D 类别、位置、大小、方向和速度。

3D 距离。基于随机生成的 3D 坐标，识别与相应位置附近的交通元素，并提供交通元素的 3D 属性。

车道到物体。基于随机选择的车道中心线，列出该车道上的物体及其 3D 属性。

### 3.3 指标

所提出的OmniDrive数据集涉及图像描述、开环规划和反事实推理任务。每个任务都有不同的侧重点，因此很难用单一指标来评估它们。在本节中，我们将详细阐述如何评估模型在我们数据集上的表现。

对于与图像描述相关的任务，例如场景描述和注意物体的选择，我们采用常用的基于语言的评估指标，在词汇级别上评估句子相似度，包括METEOR[[3](https://arxiv.org/html/2405.01533v1#bib.bib3)]、ROUGE[[27](https://arxiv.org/html/2405.01533v1#bib.bib27)]和CIDEr[[49](https://arxiv.org/html/2405.01533v1#bib.bib49)]。根据BEV-Planner[[26](https://arxiv.org/html/2405.01533v1#bib.bib26)]，采用碰撞率和与道路边界的交集率来评估开环规划的性能。为了评估反事实推理的表现，我们要求GPT-3.5基于预测提取关键词。这些关键词包括‘安全’，‘碰撞’，‘闯红灯’，和‘超出可驾驶区域’。然后，我们将提取的关键词与真实情况进行对比，以计算每类事故的精准率和召回率。

## 4 实验

### 4.1 实现细节

我们的模型使用EVA-02-L[[14](https://arxiv.org/html/2405.01533v1#bib.bib14)]作为视觉编码器。它应用掩码图像建模来提取CLIP[[44](https://arxiv.org/html/2405.01533v1#bib.bib44)]，该模型能够提取与语言对齐的视觉特征。

在二维预训练阶段，训练数据和策略，包括批量大小、学习率和优化器，与LLaVA v1.5的[[30](https://arxiv.org/html/2405.01533v1#bib.bib30)]相同。在微调阶段，模型使用AdamW[[34](https://arxiv.org/html/2405.01533v1#bib.bib34)]优化器，批量大小为16。投影器的学习率为4e-4，而视觉编码器和LLM的学习率为5e-4。采用余弦退火策略以保证训练的稳定性。消融实验中的模型训练了6个epoch，除非另有说明。物体查询、车道查询和载体查询的数量分别设置为900、300和256。

我们还探索了替代架构。Q-Former2D使用二维预训练权重进行初始化。它在投影器中单独处理图像特征，并在LLM中进行融合。Dense BEV方法使用LSS方法[[41](https://arxiv.org/html/2405.01533v1#bib.bib41), [40](https://arxiv.org/html/2405.01533v1#bib.bib40)]将透视特征转换为BEV特征图。我们根据SOLOFusion[[40](https://arxiv.org/html/2405.01533v1#bib.bib40)]实施了时间建模。BEV特征将连续输入到MLP投影器和LLM中。

|  |  | 反事实推理 | 开环 |
| --- | --- | --- | --- |
| 消融 | 实验 | 安全 | 红灯 | 碰撞 | 可驾驶区域 | 碰撞率(%) | 交集率(%) |
|  |  | P | R | P | R | P | R | P | R | 平均 | 平均 |
| 完整模型 | Q-Former3D | $70.7$ | 49.0 | 57.6 | $58.3$ | 32.3 | 72.6 | 48.5 | 58.6 | 3.79 | $4.59$ |
| 数据 | 无在线 | 69.4 | 39.4 | 36.2 | 65.6 | 29.7 | 69.4 | 48.0 | 57.8 | 4.93 | 4.02 |
| 架构 | Q-Former2D | 71.4 | $39.3$ | 58.3 | $61.1$ | $32.0$ | $66.7$ | $44.4$ | $52.8$ | $3.98$ | $6.03$ |
| 密集BEV | $70.2$ | $17.3$ | $48.7$ | $53.6$ | $31.1$ | 70.4 | $32.4$ | 56.6 | $4.43$ | $8.56$ |
| 无时间模块 | 67.8 | 48.4 | 47.0 | 62.6 | 31.2 | 63.8 | 46.5 | 55.3 | $6.07$ | $5.83$ |
| 感知 | 无车道 | 67.7 | 57.3 | 58.1 | 59.6 | 31.0 | 56.7 | 47.9 | 56.8 | $4.65$ | $8.71$ |
| 监督 | 无物体与车道 | $69.0$ | 57.8 | $51.3$ | $61.2$ | $30.0$ | $53.2$ | $45.3$ | $57.1$ | $6.77$ | $8.43$ |

表2：与规划相关任务的消融研究。P和R分别代表精度和召回率。“无在线”表示去除在线训练数据。“无时间”表示去除时间模块。“冻结”表示不对主干层应用梯度。“无物体”和“无车道”表示没有相应的3D感知监督。在此表中，规划器中的模型均未使用高级命令和自我状态。

### 4.2 通过反事实推理进行规划

基于我们的Omnidrive-nuScenes数据集，我们对训练配置和模型架构进行了各种修改的消融实验。所有在[表2](https://arxiv.org/html/2405.01533v1#S4.T2 "In 4.1 Implementation Details ‣ 4 Experiment ‣ OmniDrive: A Holistic LLM-Agent Framework for Autonomous Driving with 3D Perception, Reasoning and Planning")中展示的分析均未使用高级命令和自我状态[[20](https://arxiv.org/html/2405.01533v1#bib.bib20), [26](https://arxiv.org/html/2405.01533v1#bib.bib26)]。在这种设置下，可以观察到反事实指标与开环规划之间存在一定的相关性。

我们发现，Q-Former2D在与二维相关的任务上表现更好，例如判断交通灯的状态。然而，Q-Former3D在三维任务中显然具有更大的优势，如碰撞检测（精度为32.2%，召回率为72.6%）和可行驶区域识别（精度为48.5%，召回率为58.6%）。具有中心线构建任务的模型（即完整模型）在可行驶区域任务上优于没有车道监督的模型。

### 4.3 消融研究与分析

反事实推理与标注。在表格[3](https://arxiv.org/html/2405.01533v1#S4.T3 "Table 3 ‣ 4.3 Ablation Study & Analysis ‣ 4 Experiment ‣ OmniDrive: A Holistic LLM-Agent Framework for Autonomous Driving with 3D Perception, Reasoning and Planning")中，完整模型在反事实推理方面表现最佳，平均精度为52.3%，平均召回率为59.6%。

更重要的是，我们展示了我们的模型受益于Q-Former3D，并且在字幕任务上取得了与Q-Former2D相当的表现，获得了$38.0\%$的METEOR分数、$68.6\%$的CIDEr分数和$32.6$的ROUGE分数。此外，我们的模型可以同时处理多个视角的摄像头，而Q-Former2D需要分别处理每个视角，并且为LLM输入需要高效地使用大量令牌（1500+）。我们注意到，Dense BEV模型的结果最差，因为它未能利用2D预训练带来的好处。我们还发现，配备预训练的相同Q-Former在描述性任务上的表现相似。引入额外的3D监督和时间信息并未带来显著的提升。

| Abaltion | Exp. | Counterfactual | Caption |
| --- | --- | --- | --- |
| AP (%) | AR (%) | METEOR$\uparrow$ | CIDEr $\uparrow$ | ROUGE$\uparrow$ |
| Full Model | Q-Former3D | 52.3 | 59.6 | 38.0 | 68.6 | 32.6 |
| Data | No Online | 45.8 | 58.1 | 38.2 | 69.0 | 32.7 |
| Architecture | Q-Former2D | 51.5 | 55.0 | 38.3 | 67.1 | 32.5 |
| Dense BEV | 45.6 | 49.5 | 35.6 | 59.5 | 27.8 |
| No Temporal | 48.1 | 57.5 | 37.9 | 68.4 | 32.6 |
| Perception | No Lane | 51.2 | 57.6 | 38.0 | 67.8 | 32.6 |
| Supervision | No Object & Lane | 48.9 | 57.3 | 38.2 | 67.8 | 32.6 |

表3：我们基准测试的分析。

表4：在NuScenes-QA上的结果 [[43](https://arxiv.org/html/2405.01533v1#bib.bib43)]。L和C分别代表激光雷达和摄像头。我们突出显示了每种模态下的最新技术方法。

| Model | Modality | Acc.(%) |
| --- | --- | --- |
| BEVDet+BUTD [[43](https://arxiv.org/html/2405.01533v1#bib.bib43)] | C | 57.0 |
| BEVDet+MCAN [[43](https://arxiv.org/html/2405.01533v1#bib.bib43)] | C | 57.9 |
| CenterPoint+BUTD [[43](https://arxiv.org/html/2405.01533v1#bib.bib43)] | L | 58.1 |
| CenterPoint+MCAN [[43](https://arxiv.org/html/2405.01533v1#bib.bib43)] | L | 59.5 |
| OmniDrive | C | 59.2 |

在NuScenes-QA上的比较。我们还在表[4](https://arxiv.org/html/2405.01533v1#S4.T4 "Table 4 ‣ 4.3 Ablation Study & Analysis ‣ 4 Experiment ‣ OmniDrive: A Holistic LLM-Agent Framework for Autonomous Driving with 3D Perception, Reasoning and Planning")中展示了NuScenes-QA的结果。在NuScenes-QA中，大多数答案都是单词，且仅与感知相关。在相同的摄像头模态下，我们的模型在准确率上比BEVDet+MCAN高出1.3%，这证明了预训练的重要性。我们的模型性能与激光雷达模态的模型相当。

| Method | Ego Status | L2 (m) $\downarrow$ | Collision (%) $\downarrow$ | Intersection (%) $\downarrow$ |
| --- | --- | --- | --- | --- |
| BEV | Planner | 1s | 2s | 3s | Avg. | 1s | 2s | 3s | Avg. | 1s | 2s | 3s | Avg. |
| ST-P3 | - | - | 1.59${}^{\text{\textdagger}}$ | 2.64${}^{\text{\textdagger}}$ | 3.73${}^{\text{\textdagger}}$ | 2.65${}^{\text{\textdagger}}$ | 0.69${}^{\text{\textdagger}}$ | 3.62${}^{\text{\textdagger}}$ | 8.39${}^{\text{\textdagger}}$ | 4.23${}^{\text{\textdagger}}$ | 2.53${}^{\text{\textdagger}}$ | 8.17${}^{\text{\textdagger}}$ | 14.4${}^{\text{\textdagger}}$ | 8.37${}^{\text{\textdagger}}$ |
| UniAD | - | - | 0.59 | 1.01 | 1.48 | 1.03 | 0.16 | 0.51 | 1.64 | 0.77 | 0.35 | 1.46 | 3.99 | 1.93 |
| UniAD | ✓ | ✓ | 0.20 | 0.42 | 0.75 | 0.46 | 0.02 | 0.25 | 0.84 | 0.37 | 0.20 | 1.33 | 3.24 | 1.59 |
| VAD-Base | - | - | 0.69 | 1.22 | 1.83 | 1.25 | 0.06 | 0.68 | 2.52 | 1.09 | 1.02 | 3.44 | 7.00 | 3.82 |
| VAD-Base | ✓ | ✓ | 0.17 | 0.34 | 0.60 | 0.37 | 0.04 | 0.27 | 0.67 | 0.33 | 0.21 | 2.13 | 5.06 | 2.47 |
| Ego-MLP | - | ✓ | 0.15 | 0.32 | 0.59 | 0.35 | 0.00 | 0.27 | 0.85 | 0.37 | 0.27 | 2.52 | 6.60 | 2.93 |
| BEV-Planner | - | - | 0.30 | 0.52 | 0.83 | 0.55 | 0.10 | 0.37 | 1.30 | 0.59 | 0.78 | 3.79 | 8.22 | 4.26 |
| BEV-Planner++ | ✓ | ✓ | 0.16 | 0.32 | 0.57 | 0.35 | 0.00 | 0.29 | 0.73 | 0.34 | 0.35 | 2.62 | 6.51 | 3.16 |
| OmniDrive$\ddagger$ | - | - | 1.15 | 1.96 | 2.84 | 1.98 | 0.80 | 3.12 | 7.46 | 3.79 | 1.66 | 3.86 | 8.26 | 4.59 |
| OmniDrive | - | - | 0.40 | 0.80 | 1.32 | 0.84 | 0.04 | 0.46 | 2.32 | 0.94 | 0.93 | 3.65 | 8.28 | 4.29 |
| OmniDrive++ | ✓ | ✓ | 0.14 | 0.29 | 0.55 | 0.33 | 0.00 | 0.13 | 0.78 | 0.30 | 0.56 | 2.48 | 5.96 | 3.00 |

表 5：开环规划的比较。为了公平比较，我们参考了BEV-Planner中的复现结果[[26](https://arxiv.org/html/2405.01533v1#bib.bib26)]。†：ST-P3（ID-0）的官方实现使用了部分错误的地面真值。${}\ddagger$: 在训练和测试阶段未使用高级指令。

### 4.4 开环规划的讨论

我们在表格[5](https://arxiv.org/html/2405.01533v1#S4.T5 "Table 5 ‣ 4.3 Ablation Study & Analysis ‣ 4 Experiment ‣ OmniDrive: A Holistic LLM-Agent Framework for Autonomous Driving with 3D Perception, Reasoning and Planning")中比较了提出的OmniDrive与以前的最先进视觉规划方法。基于MLLM的开环规划也能够达到与最先进方法相当的性能。然而，正如在BEV-Planner中所提到的[[26](https://arxiv.org/html/2405.01533v1#bib.bib26)]，观察到对自我状态的编码显著提高了所有方法的度量。此外，我们发现高级指令也能显著减少碰撞率和交叉口通过率。以前的方法是根据地面真值轨迹的相对位置提供高级指令，这对网络回归施加了显著约束，从而减少了偏离地面真值轨迹的可能性。我们认为这一设计也是不合理的，因此在其他实验中我们去除了这一设置。

| 场景描述与反事实推理 |
| --- |
|   |
| 用户 | 你能总结一下全景图像展示了什么吗？ |
| 我们的 | 全景图像展示了一个下雨的城市环境，湿滑的道路反射着阴沉的天空。在左前方，我们看到一个停车场入口和一个显示绿色的交通灯，表明交通流畅可以通行。前方视图显示了一条多车道的道路，有车辆，包括一辆白色汽车右转，并且有一个交叉口的交通灯。右前方展示了一个行人过街道，且有一个交通灯，正在等待过街的人。左后方的视图捕捉到一个红色交通灯、一个行人过街道和一辆黑色汽车在等待。后方视图显示了一座正在建设的大楼，塔吊高高耸立，路上有几辆车。右后方的视图包含一个红色交通灯、一个行人过街道和一条通向环形交叉口的道路，一辆黑色汽车正驶近交叉口。尽管天气潮湿，整个场景依然是一个繁忙的城市景象。 |
| 用户 | 如果你决定慢慢行驶并右转，结果会是什么？ |
| 我们的 | 如果我慢慢行驶并右转，沿着轨迹[PT, (+2.44, -0.09), (+4.91, -0.36), (+7.41, -0.84), (+9.91, -1.53), (+12.37, -2.44), (+14.76, -3.55)]，这将导致驶出可驾驶区域，这是一个危险且错误的行为。该轨迹表明转弯过于急剧，且与道路布局不符，可能导致碰撞或驶出道路。在这种情况下，正确的行为应该是保持在车道内，跟随道路的弯曲，确保我始终处于可驾驶区域内并遵守交通规则。 |

表6：场景描述和反事实推理的可视化结果。

### 4.5 可视化

在表[6](https://arxiv.org/html/2405.01533v1#S4.T6 "Table 6 ‣ 4.4 Discussion on Open-loop Planning ‣ 4 Experiment ‣ OmniDrive: A Holistic LLM-Agent Framework for Autonomous Driving with 3D Perception, Reasoning and Planning")中展示了一个具有挑战性的场景。OmniDrive-Agent在场景描述和反事实推理上展现了令人印象深刻的结果。该模型对相对位置有基本的理解，能够根据给定的轨迹和安全考虑正确地进行反事实推理。

## 5 相关工作

### 5.1 端到端自动驾驶

端到端自动驾驶的目标是创建一个完全可微分的系统，该系统从传感器输入到控制信号都涵盖其中[[61](https://arxiv.org/html/2405.01533v1#bib.bib61), [57](https://arxiv.org/html/2405.01533v1#bib.bib57), [42](https://arxiv.org/html/2405.01533v1#bib.bib42)]。该系统允许对整个系统进行联合优化，从而减少错误的积累。目前的技术路线图主要分为两条路径：开环自动驾驶和闭环自动驾驶。

在开环自动驾驶中，训练和评估过程通常是在日志重放的现实世界数据集上进行的[[43](https://arxiv.org/html/2405.01533v1#bib.bib43)]。这种方法忽视了自车与其他交通参与者之间交互的影响，导致了累积误差。开创性的工作UniAD[[17](https://arxiv.org/html/2405.01533v1#bib.bib17)]和VAD[[20](https://arxiv.org/html/2405.01533v1#bib.bib20)]将目标检测、跟踪和语义分割等感知任务的模块化设计整合到统一的规划框架中。然而，Ego-MLP[[60](https://arxiv.org/html/2405.01533v1#bib.bib60)]和BEV-Planner[[26](https://arxiv.org/html/2405.01533v1#bib.bib26)]则突出了开环端到端驾驶基准测试的局限性。在这些基准测试中，模型可能过度拟合自车状态信息，从而获得不合理的高性能。研究人员正在通过引入闭环基准测试来解决开环评估中的挑战。近期的工作，例如MILE[[15](https://arxiv.org/html/2405.01533v1#bib.bib15)]、ThinkTwice[[19](https://arxiv.org/html/2405.01533v1#bib.bib19)]、VADv2[[7](https://arxiv.org/html/2405.01533v1#bib.bib7)]，利用CARLA[[11](https://arxiv.org/html/2405.01533v1#bib.bib11)]作为模拟器，能够创建具有其他智能体反馈的虚拟环境。研究人员迫切需要一种合理的方法，在现实世界中评估端到端自动驾驶系统。MLLM模型弥合了数据驱动决策和用户之间的鸿沟。这使我们能够基于特定轨迹进行可解释分析并进行反事实推理，从而增强自动驾驶系统的安全冗余。

### 5.2 多模态语言模型（MLLMs）

多模态语言模型利用LLMs和各种模态编码器，成功地弥合了语言与其他模态之间的差距，并在多模态任务中表现出色，包括视觉问答、字幕生成和开放世界检测。一些MLLMs，例如CLIP[[44](https://arxiv.org/html/2405.01533v1#bib.bib44)]和ALIGN[[18](https://arxiv.org/html/2405.01533v1#bib.bib18)]，利用对比学习为语言和视觉创建了相似的嵌入空间。更近期的模型，如BLIP-2[[24](https://arxiv.org/html/2405.01533v1#bib.bib24)]，明确针对多模态任务并接收多模态输入。对于这些模型，存在两种常见的技术用于对齐语言与其他输入模态：自注意力和交叉注意力。LLaVa[[31](https://arxiv.org/html/2405.01533v1#bib.bib31)]、PaLM-E[[12](https://arxiv.org/html/2405.01533v1#bib.bib12)]、PaLI[[8](https://arxiv.org/html/2405.01533v1#bib.bib8)]和RT2[[62](https://arxiv.org/html/2405.01533v1#bib.bib62)]利用自注意力进行对齐，通过交错或拼接图像和文本标记以固定的序列长度进行处理。然而，基于自注意力的MLLMs无法处理高分辨率输入，不适用于需要多摄像头高分辨率图像的自动驾驶任务。相反，Flamingo[[1](https://arxiv.org/html/2405.01533v1#bib.bib1)]、Qwen-VL[[2](https://arxiv.org/html/2405.01533v1#bib.bib2)]、BLIP-2[[24](https://arxiv.org/html/2405.01533v1#bib.bib24)]等模型利用交叉注意力，能够提取固定数量的视觉标记，无论图像分辨率如何。因此，我们的模型采用BLIP-2的Qformer架构来处理高分辨率图像。

### 5.3 驱动LLM-Agents与基准测试

驱动LLM-Agents。鉴于LLM/MLLMs的高性能和将不同模态与语言对齐的能力，现如今各方都在竞相将MLLMs/LLMs与自动驾驶（AD）相结合。大多数AD MLLMs方法试图通过端到端学习创建可解释的自动驾驶。DriveGPT4利用LLMs生成汽车行动的理由，同时预测汽车的下一个控制信号[[58](https://arxiv.org/html/2405.01533v1#bib.bib58)]。类似地，Drive Anywhere提出了一种补丁对齐的特征提取方法，允许MLLMs提供可通过文本查询的驾驶决策[[51](https://arxiv.org/html/2405.01533v1#bib.bib51)]。其他工作则通过基于图的VQA（DriveLM）[[46](https://arxiv.org/html/2405.01533v1#bib.bib46)]或思维链（CoT）设计[[47](https://arxiv.org/html/2405.01533v1#bib.bib47), [54](https://arxiv.org/html/2405.01533v1#bib.bib54)]来利用MLLMs。它们明确解决了多个驾驶任务，并且同时处理典型的MLLM任务，例如生成场景描述与分析、预测和规划。

基准测试。为了评估自动驾驶感知与规划，有多种数据集涵盖了感知、规划、转向和运动数据（ONCE [[37](https://arxiv.org/html/2405.01533v1#bib.bib37)]，NuPlan [[5](https://arxiv.org/html/2405.01533v1#bib.bib5)]，nuScenes [[4](https://arxiv.org/html/2405.01533v1#bib.bib4)]，CARLA [[11](https://arxiv.org/html/2405.01533v1#bib.bib11)]，Waymo [[13](https://arxiv.org/html/2405.01533v1#bib.bib13)]）。然而，评估自动驾驶大语言模型（Drive LLM）方法需要更加全面的语言标注数据集。专注于感知与跟踪的数据集包括推理或描述性标注，如类似标题的内容，涵盖了nuScenes-QA [[43](https://arxiv.org/html/2405.01533v1#bib.bib43)]，NuPrompt [[56](https://arxiv.org/html/2405.01533v1#bib.bib56)]。HAD 和 Talk2Car 都包含类似人类的建议，以最佳方式导航汽车 [[22](https://arxiv.org/html/2405.01533v1#bib.bib22)，[9](https://arxiv.org/html/2405.01533v1#bib.bib9)]，而 LaMPilot 则包含旨在评估从人类命令到驾驶行为过渡的标签 [[35](https://arxiv.org/html/2405.01533v1#bib.bib35)]。除了场景描述之外，DRAMA [[36](https://arxiv.org/html/2405.01533v1#bib.bib36)] 和 Rank2Tell [[45](https://arxiv.org/html/2405.01533v1#bib.bib45)] 专注于风险物体定位。相反，BDD-X 和 Reason2Drive 通过提供自车行为和动作背后的原因，专注于汽车的可解释性 [[23](https://arxiv.org/html/2405.01533v1#bib.bib23)，[38](https://arxiv.org/html/2405.01533v1#bib.bib38)，[39](https://arxiv.org/html/2405.01533v1#bib.bib39)]。LingoQA [[38](https://arxiv.org/html/2405.01533v1#bib.bib38)] 已经在自动驾驶问答数据集中引入了反事实问题。我们认为，通过将反事实推理应用于 3D 轨迹分析，可以进一步增强开放环路设置中自动驾驶的可解释性和安全冗余性。

## 6 结论

我们通过提出 OmniDrive-Agent 和 OmniDrive-nuScenes，解决了端到端自动驾驶与大语言模型（LLM）代理的挑战。OmniDrive-Agent 采用了一种新颖的 Q-Former3D MLLM 架构，能够高效处理高分辨率多视角视频。我们的模型设计使得可以在最小调整的情况下，利用 2D 预训练知识，同时获得重要的 3D 空间理解。此外，我们还提供了一种新的端到端自动驾驶基准，其中结合了反事实推理、3D 空间意识和推理任务。OmniDrive-Agent 通过处理高分辨率多视角视频输入，展示了其有效性，并展现了卓越的场景描述和反事实推理能力。该模型在开放环路 3D 规划中也展现了出色的性能。

限制性。我们的方法尚未在更大的数据集上进行验证，例如nuPlan [[5](https://arxiv.org/html/2405.01533v1#bib.bib5)]。尽管在多个轨迹之外进行了反事实结果的模拟，但尚未考虑其他代理的反应。此部分可以进一步形成为封闭环设置，我们将把它留待未来的工作。

## 参考文献

+   [1] Alayrac, J.B., Donahue, J., Luc, P., Miech, A., Barr, I., Hasson, Y., Lenc, K., Mensch, A., Millican, K., Reynolds, M., 等：Flamingo：一个用于少样本学习的视觉语言模型。发表于：NeurIPs (2022)

+   [2] Bai, J., Bai, S., Yang, S., Wang, S., Tan, S., Wang, P., Lin, J., Zhou, C., Zhou, J.: Qwen-VL：一个具有多种能力的前沿大规模视觉-语言模型。arXiv:2308.12966 (2023)

+   [3] Banerjee, S., Lavie, A.: METEOR：一种自动化的机器翻译评估指标，与人工评判的相关性改进。发表于：ACL workshop (2005)

+   [4] Caesar, H., Bankiti, V., Lang, A.H., Vora, S., Liong, V.E., Xu, Q., Krishnan, A., Pan, Y., Baldan, G., Beijbom, O.: nuScenes：一个多模态数据集，用于自动驾驶。发表于：CVPR (2020)

+   [5] Caesar, H., Kabzan, J., Tan, K.S., Fong, W.K., Wolff, E., Lang, A., Fletcher, L., Beijbom, O., Omari, S.: NuPlan：基于机器学习的封闭环规划基准，用于自动驾驶汽车。arXiv:2106.11810 (2021)

+   [6] Chen, L., Sinavski, O., Hünermann, J., Karnsund, A., Willmott, A.J., Birch, D., Maund, D., Shotton, J.: 驾驶与LLM：融合面向对象的向量模式以实现可解释的自动驾驶。arXiv:2310.01957 (2023)

+   [7] Chen, S., Jiang, B., Gao, H., Liao, B., Xu, Q., Zhang, Q., Huang, C., Liu, W., Wang, X.: VADv2：通过概率规划实现端到端的向量化自动驾驶。arXiv:2402.13243 (2024)

+   [8] Chen, X., Wang, X., Changpinyo, S., Piergiovanni, A., Padlewski, P., Salz, D., Goodman, S., Grycner, A., Mustafa, B., Beyer, L., 等：PaLI：一个联合扩展的多语言图像-语言模型。arXiv:2209.06794 (2022)

+   [9] Deruyttere, T., Vandenhende, S., Grujicic, D., Van Gool, L., Moens, M.F.: Talk2Car: 控制你的自动驾驶汽车。发表于：EMNLP-IJCNLP (2019)

+   [10] Ding, X., Han, J., Xu, H., Liang, X., Zhang, W., Li, X.: 通过鸟瞰图注入的多模态大模型实现全面的自动驾驶理解。arXiv:2401.00988 (2024)

+   [11] Dosovitskiy, A., Ros, G., Codevilla, F., Lopez, A., Koltun, V.: Carla：一个开放的城市驾驶模拟器。发表于：CoRL (2017)

+   [12] Driess, D., Xia, F., Sajjadi, M.S., Lynch, C., Chowdhery, A., Ichter, B., Wahid, A., Tompson, J., Vuong, Q., Yu, T., 等：Palm-e：一种具身多模态语言模型。arXiv:2303.03378 (2023)

+   [13] Ettinger, S., Cheng, S., Caine, B., Liu, C., Zhao, H., Pradhan, S., Chai, Y., Sapp, B., Qi, C.R., Zhou, Y., Yang, Z., Chouard, A., Sun, P., Ngiam, J., Vasudevan, V., McCauley, A., Shlens, J., Anguelov, D.: 大规模互动运动预测用于自动驾驶：Waymo开放运动数据集。发表于：ICCV (2021)

+   [14] Fang, Y., Sun, Q., Wang, X., Huang, T., Wang, X., Cao, Y.: EVA-02：新生代的视觉表示。arXiv:2303.11331（2023）

+   [15] Hu, A., Corrado, G., Griffiths, N., Murez, Z., Gurau, C., Yeo, H., Kendall, A., Cipolla, R., Shotton, J.: 基于模型的模仿学习用于城市驾驶。载于：NeurIPS（2022）

+   [16] Hu, E.J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., Wang, L., Chen, W.: LoRA：大型语言模型的低秩适应。arXiv:2106.09685（2021）

+   [17] Hu, Y., Yang, J., Chen, L., Li, K., Sima, C., Zhu, X., Chai, S., Du, S., Lin, T., Wang, W., 等: 面向规划的自动驾驶。载于：CVPR（2023）

+   [18] Jia, C., Yang, Y., Xia, Y., Chen, Y.T., Parekh, Z., Pham, H., Le, Q., Sung, Y.H., Li, Z., Duerig, T.: 通过嘈杂文本监督，扩展视觉和视觉-语言表征学习。载于：ICML（2021）

+   [19] Jia, X., Wu, P., Chen, L., Xie, J., He, C., Yan, J., Li, H.: 驾驶前三思：面向端到端自动驾驶的可扩展解码器。载于：CVPR（2023）

+   [20] Jiang, B., Chen, S., Xu, Q., Liao, B., Chen, J., Zhou, H., Zhang, Q., Liu, W., Huang, C., Wang, X.: VAD：高效自动驾驶的向量化场景表示。arXiv:2303.12077（2023）

+   [21] Jiang, X., Li, S., Liu, Y., Wang, S., Jia, F., Wang, T., Han, L., Zhang, X.: Far3d：扩展环视三维物体检测的视野。arXiv:2308.09616（2023）

+   [22] Kim, J., Misu, T., Chen, Y.T., Tawari, A., Canny, J.: 为自动驾驶车辆提供人车建议的基础。载于：CVPR（2019）

+   [23] Kim, J., Rohrbach, A., Darrell, T., Canny, J., Akata, Z.: 自动驾驶车辆的文本解释。ECCV（2018）

+   [24] Li, J., Li, D., Savarese, S., Hoi, S.: BLIP-2：使用冻结图像编码器和大型语言模型进行语言-图像预训练。载于：ICML（2023）

+   [25] Li, Z., Deng, H., Li, T., Huang, Y., Sima, C., Geng, X., Gao, Y., Wang, W., Li, Y., Lu, L.: BEVFormer++：通过3D相机仅检测改进BEVFormer：Waymo开放数据集挑战2022年第一名解决方案（2023）

+   [26] Li, Z., Yu, Z., Lan, S., Li, J., Kautz, J., Lu, T., Alvarez, J.M.: 开环端到端自动驾驶是否仅需要自我状态？arXiv:2312.03031（2023）

+   [27] Lin, C.Y.: ROUGE：一个用于自动评估摘要的工具包。载于：《文本摘要的扩展》, pp. 74–81（2004）

+   [28] Lin, X., Lin, T., Pei, Z., Huang, L., Su, Z.: Sparse4D：通过稀疏时空融合实现多视角三维物体检测。arXiv:2211.10581（2022）

+   [29] Liu, H., Teng, Y., Lu, T., Wang, H., Wang, L.: SparseBEV：来自多摄像头视频的高性能稀疏3D物体检测。载于：ICCV（2023）

+   [30] Liu, H., Li, C., Li, Y., Lee, Y.J.: 通过视觉指令调优改进基准。arXiv:2310.03744（2023）

+   [31] Liu, H., Li, C., Wu, Q., Lee, Y.J.: 视觉指令调优。NeurIPS（2023）

+   [32] Liu, Y., Wang, T., Zhang, X., Sun, J.: PETR：用于多视角三维物体检测的位置信息嵌入变换。arXiv:2203.05625（2022）

+   [33] Liu, Y., Yan, J., Jia, F., Li, S., Gao, Q., Wang, T., Zhang, X., Sun, J.：PETRv2：来自多摄像头图像的3D感知统一框架。arXiv:2206.01256 (2022)

+   [34] Loshchilov, I., Hutter, F.：SGDR：具有热重启的随机梯度下降。arXiv:1608.03983 (2016)

+   [35] Ma, Y., Cui, C., Cao, X., Ye, W., Liu, P., Lu, J., Abdelraouf, A., Gupta, R., Han, K., Bera, A., et al.：LaMPilot：一个用于自动驾驶的开放基准数据集，结合语言模型程序。arXiv:2312.04372 (2023)

+   [36] Malla, S., Choi, C., Dwivedi, I., Choi, J.H., Li, J.：DRAMA：驾驶中的联合风险定位和字幕生成。In: WACV (2023)

+   [37] Mao, J., Niu, M., Jiang, C., Liang, X., Li, Y., Ye, C., Zhang, W., Li, Z., Yu, J., Xu, C., et al.：自动驾驶的百万场景：Once数据集 (2021)

+   [38] Marcu, A.M., Chen, L., Hünermann, J., Karnsund, A., Hanotte, B., Chidananda, P., Nair, S., Badrinarayanan, V., Kendall, A., Shotton, J., et al.：LingoQA：自动驾驶视频问答。arXiv:2312.14115 (2023)

+   [39] Nie, M., Peng, R., Wang, C., Cai, X., Han, J., Xu, H., Zhang, L.：Reason2Drive：面向自动驾驶的可解释性链式推理。arXiv:2312.03661 (2023)

+   [40] Park, J., Xu, C., Yang, S., Keutzer, K., Kitani, K., Tomizuka, M., Zhan, W.：时间会告诉我们：时间多视角3D物体检测的新视角和基线。arXiv:2210.02443 (2022)

+   [41] Philion, J., Fidler, S.：Lift, splat, shoot：通过隐式逆投影到3D编码来自任意摄像机设备的图像。In: ECCV (2020)

+   [42] Prakash, A., Chitta, K., Geiger, A.：端到端自动驾驶的多模态融合变换器。In: CVPR (2021)

+   [43] Qian, T., Chen, J., Zhuo, L., Jiao, Y., Jiang, Y.G.：NuScenes-QA：一种用于自动驾驶场景的多模态视觉问答基准。arXiv:2305.14836 (2023)

+   [44] Radford, A., Kim, J.W., Hallacy, C., Ramesh, A., Goh, G., Agarwal, S., Sastry, G., Askell, A., Mishkin, P., Clark, J., et al.：从自然语言监督中学习可转移的视觉模型。In: ICML (2021)

+   [45] Sachdeva, E., Agarwal, N., Chundi, S., Roelofs, S., Li, J., Kochenderfer, M., Choi, C., Dariush, B.：Rank2Tell：一个多模态驾驶数据集，用于联合重要性排名和推理。In: WACV (2024)

+   [46] Sima, C., Renz, K., Chitta, K., Chen, L., Zhang, H., Xie, C., Luo, P., Geiger, A., Li, H.：DriveLM：图形视觉问答驱动的自动驾驶。arXiv:2312.14150 (2023)

+   [47] Tian, X., Gu, J., Li, B., Liu, Y., Hu, C., Wang, Y., Zhan, K., Jia, P., Lang, X., Zhao, H.：DriveVLM：自动驾驶与大型视觉语言模型的融合。arXiv:2402.12289 (2024)

+   [48] Touvron, H., Martin, L., Stone, K., Albert, P., Almahairi, A., Babaei, Y., Bashlykov, N., Batra, S., Bhargava, P., Bhosale, S., et al.：Llama 2：开放的基础和微调聊天模型。arXiv:2307.09288 (2023)

+   [49] Vedantam, R., Lawrence Zitnick, C., Parikh, D.: CIDEr：基于共识的图像描述评估。发表于：CVPR (2015)

+   [50] Wang, S., Liu, Y., Wang, T., Li, Y., Zhang, X.: 探索面向对象的时序建模以实现高效的多视角三维物体检测。arXiv:2303.11926 (2023)

+   [51] Wang, T.H., Maalouf, A., Xiao, W., Ban, Y., Amini, A., Rosman, G., Karaman, S., Rus, D.: Drive Anywhere：具有多模态基础模型的通用端到端自动驾驶。arXiv:2310.17642 (2023)

+   [52] Wang, W., Xie, J., Hu, C., Zou, H., Fan, J., Tong, W., Wen, Y., Wu, S., Deng, H., Li, Z., 等：DriveMLM：将多模态大语言模型与自主驾驶的行为规划状态对齐。arXiv:2312.09245 (2023)

+   [53] Wang, Y., Vitor Campagnolo, G., Zhang, T., Zhao, H., Solomon, J.: DETR3D：通过3D到2D查询进行多视角图像的3D物体检测。发表于：CoRL (2022)

+   [54] Wei, J., Wang, X., Schuurmans, D., Bosma, M., Xia, F., Chi, E., Le, Q.V., Zhou, D., 等：链式思维提示激发大语言模型的推理能力。NeurIPS (2022)

+   [55] Wu, D., Chang, J., Jia, F., Liu, Y., Wang, T., Shen, J.: TopoMLP：一个简单而强大的驾驶拓扑推理管道。arXiv:2310.06753 (2023)

+   [56] Wu, D., Han, W., Wang, T., Liu, Y., Zhang, X., Shen, J.: 面向自动驾驶的语言提示。arXiv:2309.04379 (2023)

+   [57] Wu, P., Jia, X., Chen, L., Yan, J., Li, H., Qiao, Y.: 基于轨迹引导的端到端自动驾驶控制预测：一个简单而强大的基准。发表于：NeurIPS (2022)

+   [58] Xu, Z., Zhang, Y., Xie, E., Zhao, Z., Guo, Y., Wong, K.K., Li, Z., Zhao, H.: DriveGPT4：通过大语言模型进行可解释的端到端自动驾驶。arXiv:2310.01412 (2023)

+   [59] Yuan, T., Liu, Y., Wang, Y., Wang, Y., Zhao, H.: StreamMapNet：用于矢量化在线高清地图构建的流式映射网络。arXiv:2308.12570 (2023)

+   [60] Zhai, J.T., Feng, Z., Du, J., Mao, Y., Liu, J.J., Tan, Z., Zhang, Y., Ye, X., Wang, J.: 重新思考在nuScenes中端到端自动驾驶的开环评估。arXiv:2305.10430 (2023)

+   [61] Zhang, Z., Liniger, A., Dai, D., Yu, F., Van Gool, L.: 通过模仿强化学习教练实现端到端城市驾驶。发表于：ICCV (2021)

+   [62] Zitkovich, B., Yu, T., Xu, S., Xu, P., Xiao, T., Xia, F., Wu, J., Wohlhart, P., Welker, S., Wahid, A., 等：RT-2：视觉-语言-动作模型将网络知识迁移到机器人控制中。发表于：CoRL (2023)
