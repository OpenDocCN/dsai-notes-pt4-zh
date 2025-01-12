<!--yml

category: 未分类

日期: 2025-01-11 12:27:49

-->

# MobileFlow: 一种用于移动GUI代理的多模态LLM

> 来源：[https://arxiv.org/html/2407.04346/](https://arxiv.org/html/2407.04346/)

Songqin Nong, Jiali Zhu¹¹footnotemark: 1, Rui Wu¹¹footnotemark: 1, Jiongchao Jin, Shuo Shan, Xiutian Huang, Wenhao Xu

Ant Group

{nongsongqin.nsq, zhujiali.zjl, guli.wr, jinjiongchao.jjc}@antgroup.com

{shanshuo.ss, huangxiutian.hxt, hao.xuwh}@antgroup.com 同等贡献

###### 摘要

多模态大规模模型（如GPT-4v、Qwen-VL-Max）的持续发展显著增强了图像理解和用户行为分析的能力，展示了智能图形化用户界面（GUI）助手的潜力。然而，当前的GUI代理通常需要通过调用系统API来获取页面布局信息，这可能带来隐私风险；此外，固定在某一低分辨率的用户界面可能导致图像细节的丢失。同时，当前为GUI代理构建的多模态大型模型在处理中文应用程序时，理解和决策表现较差。本文介绍了MobileFlow，一种专为移动GUI代理精心打造的多模态大型语言模型。MobileFlow从开源模型Qwen-VL-Chat转化到GUI领域，包含约210亿个参数，并配备了新颖的混合视觉编码器，使其能够处理不同分辨率的图像输入，并良好支持多语言GUI。通过结合专家混合（MoE）扩展和开创性的对齐训练策略，MobileFlow具备了全面解读图像数据和理解用户指令以执行GUI交互任务的能力。最后，MobileFlow在公开的评估标准和我们提出的评估指标上，均超越了Qwen-VL-Max和GPT-4v，在GUI代理执行任务方面表现更优，并且已经成功部署于实际商业环境中，证明了其在实际应用中的有效性。

## 1 引言

大型语言模型（LLMs）为人工通用智能（AGI）系统的进展做出了重要贡献，在处理类人交互任务方面展示了卓越的能力。LLM的进展还带来了多模态大型语言模型（MLLMs）的重大突破（Chen et al. ([2024](https://arxiv.org/html/2407.04346v3#bib.bib1)); Liu et al. ([2024](https://arxiv.org/html/2407.04346v3#bib.bib2), [2023](https://arxiv.org/html/2407.04346v3#bib.bib3)); Zhu et al. ([2023](https://arxiv.org/html/2407.04346v3#bib.bib4)))，促进了复杂的视觉-语言对话和交互，弥合了文本和视觉信息之间的鸿沟。这为在数字世界中开发自主图形用户界面（GUI）代理创造了有利的机会。

![参见说明](img/c6b5bc1e8221651aaed14e7302963bc8.png)

图 1：展示 MobileFlow 在 GUI 代理中的应用。用户指令：在 Starbucks 经典咖啡中给我来一杯百合拿铁，我要在店里取。

具备视觉能力的代理在现实世界中具有巨大潜力，因为它们可以直接感知视觉信号，并与人类及 GUI 进行互动。视觉语言模型（VLM）能够掌握阅读和编程等技能，并通过多模态信息进一步扩展其潜力。一些先前的研究已经开始利用 VLM 模型实现 GUI 任务的普适性。像 AppAgent（Zhang 等人 ([2023](https://arxiv.org/html/2407.04346v3#bib.bib5))）、CogAgent（Hong 等人 ([2023](https://arxiv.org/html/2407.04346v3#bib.bib6))）和 MobileAgent（Wang 等人 ([2024a](https://arxiv.org/html/2407.04346v3#bib.bib7))）这样的代理，已经将多模态能力扩展到 GUI 界面，代表了朝着实现实用的视觉语言智能助手迈出的重要一步。

然而，使用 GPT-4v 的多模态代理在图形用户界面（GUI）中处理中文文本时遇到了问题，这些问题还受到系统 API 调用、HTML 解析和隐私问题的影响。GUI 的多样化元素给当前代理带来了挑战，代理通常使用 CLIP 对自然场景进行预训练（例如，flickr30k（Plummer 等人 ([2016](https://arxiv.org/html/2407.04346v3#bib.bib8))）），但这种方法不足以提取 UI 图像中的文本和布局。视觉语言模型（VLM）的视觉编码器受限于固定的图像分辨率，这在多样化的“超级应用”GUI场景中影响了性能。因此，在本文中，我们提出了 MobileFlow，一种专为 GUI 代理设计的创新多模态大型语言模型（LLM），其在处理包含大量中文内容的应用程序（如支付宝）方面表现突出。MobileFlow 的关键组件是其混合视觉编码器，该编码器经过严格训练，涵盖了大量 GUI 页面。这种广泛的训练使得 MobileFlow 能够有效地从各种 GUI 界面中提取并理解信息。通过依赖纯视觉感知方法，MobileFlow 的 GUI 代理无需访问系统 API 来获取页面布局详情。这种方法不仅简化了流程，还减少了对用户设备隐私侵犯的风险。

此外，MobileFlow 在理解 GUI 信息方面表现出色，提供逐步用户指导，并执行 GUI 特定的信息提取和问答。它通过 MoE 和专门的 GUI 训练将视觉和文本数据结合起来。在微调过程中，采用 CoT 方法通过展示推理过程提高准确性，使 MobileFlow 在现实商业应用中非常有效。

### 1.1 相关工作

视觉语言模型（Visual Language Models）通常由三部分组成。编码器将不同的模态投射到高维空间中，随后是一个模块，将所有模态的信息在该空间内对齐，最后，解码器将对齐的信息解释回特定的模态。至于实现，VLMs可以分为两种类型：一种使用专用的视觉编码器，如ViT（Dosovitskiy等，[2021](https://arxiv.org/html/2407.04346v3#bib.bib9)），特定的对齐模块，如Qformer（Li等，[2023](https://arxiv.org/html/2407.04346v3#bib.bib10)）（或MLP），并结合经过训练的大型语言模型（LLM）来形成系统，例如LLAVA、MiniGPT-4、Qwen-VL（Bai等，[2023](https://arxiv.org/html/2407.04346v3#bib.bib11)）、CogView（Ding等，[2021](https://arxiv.org/html/2407.04346v3#bib.bib12)）等；另一种类型则去除了独立的视觉和对齐模块，将视觉和文本内容转换为标记（tokens），然后直接输入到仅解码器架构的LLM中，形成一个统一的端到端VLM，例如Fuyu-8B（Bavishi等，[2023](https://arxiv.org/html/2407.04346v3#bib.bib13)）、Chameleon（Team，[2024](https://arxiv.org/html/2407.04346v3#bib.bib14)）等。

图形用户界面（GUI）代理（Agents）CogAgent基于CogVLM（Wang等，[2024b](https://arxiv.org/html/2407.04346v3#bib.bib15)）构建，并仅依赖图像信息，避免了需要系统API调用，但其LLM组件尚未经过针对性训练。MobileAgent和AppAgent利用现有的VLMs，如GPT-4v，构建UI代理，更侧重于提示工程（prompt engineering），同时还依赖外部模块或API来在UI层下获取信息。Ferret-UI（You等，[2024](https://arxiv.org/html/2407.04346v3#bib.bib16)），来源于Ferret（You等，[2023](https://arxiv.org/html/2407.04346v3#bib.bib17)），支持任意分辨率的图像输入，但它仍然遵循传统的对话和问答训练方法，在一般场景下没有优化UI导航能力。

VLM的视觉基础模型 对于VLM，视觉编码器组件至关重要，因为它决定了可以“看到”的内容。值得注意的是，广泛使用的架构，如CLIP-ViT（Radford等人（[2021](https://arxiv.org/html/2407.04346v3#bib.bib18)））和SigLIP（Zhai等人（[2023](https://arxiv.org/html/2407.04346v3#bib.bib19)））已激发了一系列研究，旨在确定最适合集成到VLM中的视觉编码器。例如，CLIP和DINOv2（Oquab等人（[2024](https://arxiv.org/html/2407.04346v3#bib.bib20)））捕捉到的视觉表示存在显著差异，导致创建了一个融合了两种模型特征的混合模块。此外，还提出了不同的方法，采用不同的视觉编码器在不同分辨率下处理图像，从而在各个抽象层次上结合特征。例如，LLaVA-HR（Liu等人（[2023](https://arxiv.org/html/2407.04346v3#bib.bib3)））具有一个分叉的视觉编码器，将CLIP-ViT与CLIP-ConvNext[20]结合，而DeepSeek-VL（DeepSeek-AI等人（[2024](https://arxiv.org/html/2407.04346v3#bib.bib21)））则结合了SigLIP-L和SAM-B。这些方法始终利用预训练的视觉感知器。在本研究中，我们介绍了LayoutLMV3（Huang等人（[2022](https://arxiv.org/html/2407.04346v3#bib.bib22)）），一个在大量UI数据上预训练的视觉分支，能够动态调整以适应具有不同纵横比的UI图像，从而增强整体视觉编码器的理解能力及其在GUI代理中的应用性。

## 2 方法

MobileFlow通过一个融合模块将视觉编码器与大型语言模型结合，进行图像-文本对的联合训练。它使用基于Qwen-7B的语言模型作为通用接口，配合视觉感知模块，获得双重“视觉化”能力。本文的架构框架分为三个主要部分：视觉编码器、视觉-语言适配器和广泛的语言模型，如图[2](https://arxiv.org/html/2407.04346v3#S2.F2 "Figure 2 ‣ 2 Method ‣ MobileFlow: A Multimodal LLM for Mobile GUI Agent")所示。本节将详细介绍针对原始模型结构在GUI任务中的增强和优化。

混合视觉编码器 提出的视觉编码器架构如图[3](https://arxiv.org/html/2407.04346v3#S2.F3 "图 3 ‣ 2.1 模型架构 ‣ 2 方法 ‣ MobileFlow：用于移动GUI代理的多模态LLM")所示。与大多数视觉-语言模型（VLMs）一致，我们基于预训练的视觉变换器（ViT）结构构建了视觉感知模型。具体来说，对于ViT组件，我们使用OpenAI的OpenCLIP ViT-B/32预训练权重进行初始化。此外，我们引入了一个UI编码器，具备可变分辨率输入能力，以增强视觉信息的提取。经过广泛的研究，我们选择了文档智能模型LayoutLMv3，该模型已通过大量文档数据进行预训练，作为UI编码器的基础结构。我们通过重新设计UI图像预训练任务，在UI编码器上重新评估和重新校准视觉模型的权重。为了尽可能保持UI图像的原始纵横比，我们提出了一种基于可变分辨率的图像编码方法，具体解释如下。

当我们将UI编码器的目标图像序列长度设置为784个标记，每个图像补丁的大小为16x16像素，输入图像的大小为1216x576，纵横比为19:9时，分辨率通过动态调整宽度和高度进行重新计算，以计算一个极端的纵横比，同时尽可能保持原始纵横比，在序列长度约束下。在这个例子中，计算出的图像补丁在宽度和高度方向上的数量分别是41和19。因此，重新计算后的宽度和高度尺寸分别为41x16和19x16，总的图像补丁数量为41x19=779，剩余的5个补丁将通过填充来补足。

![参见说明](img/8546bded17815b7f0ccb250a7bfde516.png)

图 2：MobileFlow概览。

### 2.1 模型架构

如图[3](https://arxiv.org/html/2407.04346v3#S2.F3 "Figure 3 ‣ 2.1 Model Architecture ‣ 2 Method ‣ MobileFlow: A Multimodal LLM for Mobile GUI Agent")所示，我们采用了MLM（掩蔽语言模型）、MIM（掩蔽图像模型）、WPA（词块对齐）和RCG（真实组件生成）作为UI编码器的预训练任务。MLM和MIM是流行且广泛使用的预训练任务，参考文献包括（Devlin等人（[2019](https://arxiv.org/html/2407.04346v3#bib.bib23)）；Bao等人（[2022](https://arxiv.org/html/2407.04346v3#bib.bib24)））。WPA任务由LayoutLMv3论文（Huang等人（[2022](https://arxiv.org/html/2407.04346v3#bib.bib22)））提出，预测一个文本单词对应的图像块是否被掩蔽，从而促进多模态对齐的学习。RCG任务首先利用控制识别能力检测给定UI界面上的所有控件，然后以预定比例随机替换这些控件。接着，使用图像解码器重建原始UI界面。

视觉语言适配器 MobileFlow引入了一种视觉语言适配器，旨在压缩图像特征并融合来自多个视觉编码器的输出特征。该适配器由交叉注意力机制和MLP模块组成。交叉注意力模块采用一组可训练的向量作为查询向量，视觉编码器中的图像特征作为键向量，以将每组视觉编码器特征浓缩为固定长度的256\。MLP模块将来自并行视觉感知模块的特征集成，通过最小数量的参数将视觉特征投影到大语言模型的语义空间中。这种配置使得模型能够灵活地感知和理解视觉模态信息。

MoE扩展 许多实践表明，如果大语言模型（LLM）采用专家混合（MoE）扩展，它们可以在保持低推理成本的同时显著提高性能。目前，大多数语言模型在现有的视觉语言模型（VLM）中采用密集结构；因此，引入MoE方法到VLM中也预计能够提供可观的改进。一般而言，MoE扩展有两种主要方法。一种是使用多个专家的随机激活（其中路由是可学习的，一个典型的例子是Mixtral-8x7B（Jiang等人（[2024](https://arxiv.org/html/2407.04346v3#bib.bib25)）））），另一种是将共享专家激活与随机专家激活结合起来（共享专家捕捉全局特征，一个典型的例子是DeepSeek-Chat（DeepSeek-AI等人（[2024](https://arxiv.org/html/2407.04346v3#bib.bib21)））））。在实现难度方面，本文采用与Mixtral相同的多专家随机激活方法。

对于MoE架构模型，MoE层通常包含多个前馈网络（FFNs）。为了利用Qwen-VL-Chat在多阶段训练过程中学习到的视觉理解和对话问答能力，MobileFlow采用了直接复制原始MLP进行扩展的方法。扩展后的每个MoE层包含4个相同的MLP作为初始专家。多项研究表明，使用经过训练的MLP作为初始专家能在后续训练中实现更快且更稳定的收敛，最终表现优于随机初始化的专家。

总结来说，MobileFlow的架构由三个主要组件组成，类似于其他VLMs：一个包含多个编码器的混合视觉理解网络，一个视觉语言对齐模块，以及一个增强了MoE的LLM。在训练和推理过程中，GUI截图通过网络处理，提取全局和详细的局部特征。这些视觉token与来自用户输入的文本token、OCR文本和GUI的BBOX数据一起经过对齐后，输入LLM。

![参见标题](img/6801901879215b9d82bc0504d1f6474d.png)

图3：UI编码器概览。

### 2.2 训练公式

GUI对齐：对于大型多模态模型，训练分为两个阶段。第一阶段是视觉语言对齐预训练，模型学习将图像与文本连接起来，以便进行适当的响应。第二阶段是指令微调，模型学习如何遵循指令完成复杂任务，如VQA、视觉推理和对话。经过大量数据训练的Qwen-VL-Chat，在这些阶段后在图像-文本任务上表现优异。MobileFlow继承了Qwen-VL-Chat的多语言图像理解能力，不需要进行全量预训练，只需进行轻量级的GUI对齐和微调，即可实现良好的GUI代理技能。此轻量级阶段包括四个训练任务：

GUI定位：此任务的目的是帮助模型在文本和图像中的特定区域之间建立联系。考虑到应用页面通常包含丰富的文本信息和各种UI设计，加入此类任务能够增强模型对页面的空间理解。

GUI参考：给定文本描述中的特定边界框或空间参考（例如左上角、右下角等）或数字引用（例如第一个、最后一个、右边第三个等），模型需要在这些位置输出相应的文本信息。模型能够理解文本所指代的内容，并在图像中识别并定位所指物体，这对GUI代理至关重要，因为许多用户的实际意图通常包括这些引用。

UI 图像问答：在此，我们使用开源的ScreenQa数据集[28]来帮助模型熟悉并理解移动GUI界面，将其基础的VQA能力转移到GUI风格的页面内容上。ScreenQA数据集包含多种VQA类型，包括页面信息的提取和页面内容的推理。

图像描述与对象位置：MobileFlow需要详细描述图像并标出对象的边界框。此任务进一步增强了模型对GUI页面的空间理解。

GUI 思维链：CoT的核心思想是让模型在给出最终答案之前生成一系列中间步骤或解释性陈述。这些步骤类似于人类解决问题时的思维过程，逐渐引导出解决方案。然而，在大多数当前的多模态LLM中，模型往往直接输出答案，而不提供推理过程或理由。这种方法在需要高阶逻辑推理的场景中常常失败（例如，在需要持续决策、点击或滑动以完成用户意图的GUI代理中）。因此，在MobileFlow中，我们在模型的训练和推理中都采用了CoT技术。经过CoT的修改后，模型在链接准确性和问答准确性上表现出显著改善。MobileFlow采用了类似于AppAgent的思维链定义，其中意图执行任务包括四个步骤：

观察：描述在图形用户界面（GUI）页面上观察到的内容，整合有关页面控件的信息。

推理：考虑如何操作当前页面以完成给定任务。

动作：在代理的动作空间内生成行为，这些行为可能是点击、滑动或输入文本。

摘要：总结动作和先前行为作为下一轮交互的历史信息。

视觉问答任务的任务结构类似，唯一不同的是动作步骤被生成答案所替代。详细的提示结构见附录B。

## 3 实验

在本节中，我们展示了动作预测和视觉问答任务的定性和定量结果，并通过一项消融研究强调了我们技术贡献的重要性。更多的实验实现和部署细节请见附录D。

### 3.1 度量标准

在先前的研究中，MobileAgent引入了一套有效的度量指标，适用于有限样本数量的离散计数——特别是该论文引用了一个仅有10个样本的案例。然而，当数据集扩大到更大的测试数据时，这些指标无法准确反映所提议的大型语言模型（LLM）代理的能力。此外，针对观察到的端点判定问题，我们引入了端点确定率（EDR）作为新指标来识别这一问题。我们在附录C中展示了如何判断案例是否为正面或负面。

此外，我们采用了在先前研究中提到的整体任务成功率（WTSR）和步骤成功率（SSR）作为度量指标，以评估单步预测和多步预测的准确性。每个指标的计算方法在附录C中进行了展示。

### 3.2 定量结果

为了全面评估我们新提出的方法，我们对比分析了MobileFlow与其他领先的大型语言模型（LLM）终端代理算法，包括MobileAgent和GPT-4v，在各种商业领域的移动应用策略生成中的表现。

表1：MobileFlow在6个业务领域和3种任务复杂度下的定量结果

| 复杂度 | 指标 | 食品配送 | 到店 | 医疗服务 | 基金选择 | 保险 | 游戏 | 总计 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 长链任务 | WTSR | 0.2353 | 0.1765 | 0.1875 | 0.2857 | - | - | 0.2213 |
| SSR | 0.8282 | 0.7665 | 0.8163 | 0.5652 | - | - | 0.7441 |
| EDR | 0.1429 | 0.14 | 0.1574 | 0.1038 | - | - | 0.1360 |
| 中链任务 | WTSR | 0.7691 | 0.3999 | 0.5554 | 0.2308 | 0.2 | - | 0.4310 |
| SSR | 0.9634 | 0.9032 | 0.8947 | 0.5652 | 0.7547 | - | 0.8162 |
| EDR | 0.3241 | 0.423 | 0.1739 | 0.1796 | 0.1875 | - | 0.2576 |
| 短链任务 | WTSR | - | 0.9995 | - | 0.4154 | 0.7999 | 0.6363 | 0.7128 |
| SSR | - | 0.9997 | - | 0.4386 | 0.8332 | 0.7199 | 0.7479 |
| EDR | - | 0.25 | - | 0.2015 | 0.2890 | 0.2047 | 0.2363 |
| 平均值 | WTSR | 0.4667 | 0.2917 | 0.3333 | 0.3810 | 0.3333 | 0.6363 | 0.4071 |
| SSR | 0.8735 | 0.7921 | 0.8341 | 0.5353 | 0.6136 | 0.7199 | 0.7280 |
| EDR | 0.3 | 0.2083 | 0.2593 | 0.1807 | 0.2450 | 0.2047 | 0.2330 |

在我们的定量评估中，我们将测试数据集划分为六个业务领域：食品配送、到店、保险、医疗、基金选择和游戏应用。同时，我们将任务分为三种复杂度级别：长链（超过8步）、中链（4-8步）和短链（4步或更少）。对比结果见表[1](https://arxiv.org/html/2407.04346v3#S3.T1 "表1 ‣ 3.2 定量结果 ‣ 3 实验 ‣ MobileFlow: 一种面向移动GUI代理的多模态LLM")，该表详细展示了MobileFlow在与现有最先进解决方案的比较中的性能指标。

根据表[1](https://arxiv.org/html/2407.04346v3#S3.T1 "Table 1 ‣ 3.2 Quantitative Results ‣ 3 Experiments ‣ MobileFlow: A Multimodal LLM for Mobile GUI Agent")中提供的数据，可以得出几个关键观察结果。不同业务领域的表现差异显著，这可能归因于各领域内任务步骤长度或复杂度的分布不同。此外，还有一个明显的趋势表明，随着任务复杂度的增加，评估指标的表现趋于下降，这一现象与人类的认知模式一致。

EDR可能最初看起来较低，但它与WTSR相关。任务必须完全执行并正确结束，EDR才会被计为成功。因此，EDR预计会低于WTSR。我们发现这一点是正确的，尤其是在医疗服务应用中，所有成功预测的任务也都正确结束。

此外，我们还对提出的MobileFlow与当前最先进的算法进行了比较分析。比较结果详见表[2](https://arxiv.org/html/2407.04346v3#S3.T2 "Table 2 ‣ 3.2 Quantitative Results ‣ 3 Experiments ‣ MobileFlow: A Multimodal LLM for Mobile GUI Agent")，提供了MobileFlow与现有领先解决方案在该领域的对比情况。

表2：与当前SOTA LLM-代理在动作预测任务中的比较

| 方法 | 指标 | 食品配送 | 食品外卖 | 医疗服务 | 基金选择 | 保险 | 游戏 | 总体 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| GPT-4v | WTSR | 0.1833 | 0.1876 | 0.1138 | 0.1428 | 0.2021 | 0.4132 | 0.2071 |
| SSR | 0.5716 | 0.4647 | 0.3571 | 0.4857 | 0.5012 | 0.6613 | 0.5069 |
| Qwen-VL-Max | WTSR | 0.3650 | 0.2562 | 0.1643 | 0.2875 | 0.3076 | 0.5832 | 0.3273 |
| SSR | 0.7338 | 0.7075 | 0.6962 | 0.5112 | 0.2425 | 0.7763 | 0.6113 |
| MobileFlow | WTSR | 0.4667 | 0.2917 | 0.3333 | 0.3810 | 0.3333 | 0.6363 | 0.4071 |
| SSR | 0.8735 | 0.7921 | 0.8341 | 0.5353 | 0.6136 | 0.7199 | 0.7280 |

将MobileFlow与其他顶尖的LLM代理进行比较，表明它在不同领域的表现更佳。MobileFlow在复杂任务中表现突出。尽管MobileFlow的LLM模型比Qwen-vl-max小，但其结果依然具有竞争力，如表[2](https://arxiv.org/html/2407.04346v3#S3.T2 "Table 2 ‣ 3.2 Quantitative Results ‣ 3 Experiments ‣ MobileFlow: A Multimodal LLM for Mobile GUI Agent")所示。这表明，即使模型较小，MobileFlow通过微调后依然能够改进，并能与更大的模型相匹配或超越，证明了该方法的高效性和强大性。

对于视觉问答任务，结果展示在随后的表格 [3](https://arxiv.org/html/2407.04346v3#S3.T3 "Table 3 ‣ 3.2 Quantitative Results ‣ 3 Experiments ‣ MobileFlow: A Multimodal LLM for Mobile GUI Agent") 中。表 3 展示了 MobileFlow 和其他领先的 LLM 代理在涉及解释视觉和文本数据以产生精确响应的任务中的表现。它评估了这些模型在视觉问答（VQA）任务中的能力，重点是答案的准确性以及正确理解和回应带有视觉提示的问题的能力。我们的 MobileFlow 在 VQA 任务中表现出了强大的能力，特别是在通过特定业务数据进行微调后。令人印象深刻的是，它能够与更大规模的模型如 Qwen-vl-max 竞争。这表明我们的微调效果良好，即使 MobileFlow 模型本身不如一些大型模型，也能在 VQA 任务中表现出色。

表 3：与 SOTA LLM-agent 在 VQA 任务中的对比

| 方法 | 召回率 | 准确率 | F-得分 |
| --- | --- | --- | --- |
| GPT-4v | 0.6835 | 0.6228 | 0.6521 |
| Qwen-VL-Max | 0.7478 | 0.7064 | 0.7268 |
| MobileFlow | 0.7478 | 0.7253 | 0.7363 |

### 3.3 消融研究

MoE 结构的有效性 基于表格 [4](https://arxiv.org/html/2407.04346v3#S3.T4 "Table 4 ‣ 3.3 Ablation Study ‣ 3 Experiments ‣ MobileFlow: A Multimodal LLM for Mobile GUI Agent") 中的数据，MobileFlow 中提出的采用 MoE（专家混合）结构的 ViT（视觉 Transformer）架构，相较于传统的 ViT 加密集型大语言模型（LLM）结构，取得了显著的改进。具体而言，整体任务成功率（WTSR）提升了 4.49%，步骤成功率（SSR）则提高了 8.17%。这些结果突出了 MoE 组件在提升 MobileFlow 性能方面的有效性，强调了这一创新模型结构在复杂任务执行和预测准确性上的优势。

UI 编码器的有效性 我们比较了在引入 UI 编码器前后的 MobileFlow 性能。实验结果表明，将 UI 编码器作为视觉分支融入 MobileFlow 架构后，WTSR 指标提高了 6.96%，SSR 指标提高了 4.47%。这进一步强调了补充 UI 视觉信息对 MobileFlow 最终决策的重要性。

表 4：MoE 结构和 UI 编码器的有效性

| 模型 | WTSR | SSR |
| --- | --- | --- |
| ViT(448px) + Dense | 0.2926 | 0.6016 |
| ViT(448px) + MoE | 0.3375 | 0.6833 |
| ViT(448px) + UI Encoder + MoE | 0.4071 | 0.7280 |

## 4 结论

在本文中，我们提出了 MobileFlow，一种利用多模态大型模型并结合一系列优化技术的 GUI 代理，用于分析 UI 图像、理解用户指令并在实际场景下操作。我们提出的 MobileFlow 旨在高效应对各种复杂场景和业务领域，并已在多个行业的实际应用中展示了其能力，证明了其多功能性和有效性。如附录 D 所示，更多应用可以采用 MobileFlow 进行进一步优化。

与许多行业中的开创性代理一样，MobileFlow 标志着 GUI 代理演变中的一个重要初步步骤。MobileFlow 面临未来的挑战，如易受幻觉影响和多图像处理问题。目前它主要用于移动应用，但有潜力扩展到其他设备，如计算机。这将使 MobileFlow 成为一个可靠且用户友好的 AI 助手，可在各平台之间执行日常任务。

## 参考文献

+   Chen 等人 [2024] Zhe Chen, Jiannan Wu, Wenhai Wang, Weijie Su, Guo Chen, Sen Xing, Muyan Zhong, Qinglong Zhang, Xizhou Zhu, Lewei Lu, Bin Li, Ping Luo, Tong Lu, Yu Qiao, 和 Jifeng Dai. Internvl: 扩展视觉基础模型并对齐通用视觉-语言任务, 2024. 网址 [https://arxiv.org/abs/2312.14238](https://arxiv.org/abs/2312.14238).

+   Liu 等人 [2024] Haotian Liu, Chunyuan Li, Yuheng Li, 和 Yong Jae Lee. 通过视觉指令调优改进基准, 2024. 网址 [https://arxiv.org/abs/2310.03744](https://arxiv.org/abs/2310.03744).

+   Liu 等人 [2023] Haotian Liu, Chunyuan Li, Qingyang Wu, 和 Yong Jae Lee. 视觉指令调优, 2023. 网址 [https://arxiv.org/abs/2304.08485](https://arxiv.org/abs/2304.08485).

+   Zhu 等人 [2023] Deyao Zhu, Jun Chen, Xiaoqian Shen, Xiang Li, 和 Mohamed Elhoseiny. Minigpt-4: 利用先进的大型语言模型增强视觉-语言理解, 2023. 网址 [https://arxiv.org/abs/2304.10592](https://arxiv.org/abs/2304.10592).

+   Zhang 等人 [2023] Chi Zhang, Zhao Yang, Jiaxuan Liu, Yucheng Han, Xin Chen, Zebiao Huang, Bin Fu, 和 Gang Yu. Appagent: 作为智能手机用户的多模态代理, 2023. 网址 [https://arxiv.org/abs/2312.13771](https://arxiv.org/abs/2312.13771).

+   Hong 等人 [2023] Wenyi Hong, Weihan Wang, Qingsong Lv, Jiazheng Xu, Wenmeng Yu, Junhui Ji, Yan Wang, Zihan Wang, Yuxuan Zhang, Juanzi Li, Bin Xu, Yuxiao Dong, Ming Ding, 和 Jie Tang. Cogagent: 一种用于 GUI 代理的视觉语言模型, 2023. 网址 [https://arxiv.org/abs/2312.08914](https://arxiv.org/abs/2312.08914).

+   Wang 等人 [2024a] Junyang Wang, Haiyang Xu, Jiabo Ye, Ming Yan, Weizhou Shen, Ji Zhang, Fei Huang, 和 Jitao Sang. Mobile-agent: 具有视觉感知的自主多模态移动设备代理, 2024a. 网址 [https://arxiv.org/abs/2401.16158](https://arxiv.org/abs/2401.16158).

+   普卢默等人 [2016] 布莱恩·A·普卢默，王李伟，克里斯·M·塞尔万特斯，胡安·C·凯塞多，朱莉娅·霍肯迈尔，斯韦特兰娜·拉泽布尼克。Flickr30k实体：收集区域与短语的对应关系，用于更丰富的图像到句子模型，2016。网址 [https://arxiv.org/abs/1505.04870](https://arxiv.org/abs/1505.04870)。

+   多索维茨基等人 [2021] 亚历克谢·多索维茨基，卢卡斯·贝耶，亚历山大·科列斯尼科夫，迪尔克·魏森博恩，谢晓华，托马斯·翁特里纳，穆斯塔法·德赫格尼，马修·敏德尔，乔治·海戈尔德，西尔万·吉利，雅各布·乌斯科雷特，尼尔·霍尔斯比。图像胜过16x16个词：用于大规模图像识别的Transformer，2021。网址 [https://arxiv.org/abs/2010.11929](https://arxiv.org/abs/2010.11929)。

+   李等人 [2023] 李俊楠，李东旭，西尔维奥·萨瓦雷塞，霍斯特·史蒂文。Blip-2：通过冻结图像编码器和大语言模型引导图像-语言预训练，2023。网址 [https://arxiv.org/abs/2301.12597](https://arxiv.org/abs/2301.12597)。

+   白等人 [2023] 白金泽，白帅，杨树生，王世杰，谭思南，王鹏，林俊阳，周昌，周竞任。Qwen-vl：一种多功能的视觉-语言模型，用于理解、定位、文本阅读及更多，2023。网址 [https://arxiv.org/abs/2308.12966](https://arxiv.org/abs/2308.12966)。

+   丁等人 [2021] 丁明，杨卓逸，洪文一，郑文迪，周昌，尹大，林俊阳，邹旭，邵周，杨洪霞，唐杰。Cogview：通过Transformer掌握文本到图像生成，2021。网址 [https://arxiv.org/abs/2105.13290](https://arxiv.org/abs/2105.13290)。

+   巴维希等人 [2023] 罗汉·巴维希，埃里克·埃尔森，柯蒂斯·霍瑟恩，麦克斯韦·奈，奥古斯都·奥德纳，阿鲁什·索马尼，萨格纳克·塔什尔尔。介绍我们的多模态模型，2023。网址 [https://www.adept.ai/blog/fuyu-8b](https://www.adept.ai/blog/fuyu-8b)。

+   团队 [2024] 变色龙团队。变色龙：混合模态早期融合基础模型，2024。网址 [https://arxiv.org/abs/2405.09818](https://arxiv.org/abs/2405.09818)。

+   王等人 [2024b] 王维汉，吕青松，余文萌，洪文一，齐吉，王燕，季俊辉，杨卓逸，赵磊，宋希轩，徐佳正，徐彬，李娟子，董雨潇，丁明，唐杰。Cogvlm：预训练语言模型的视觉专家，2024b。网址 [https://arxiv.org/abs/2311.03079](https://arxiv.org/abs/2311.03079)。

+   由等人 [2024] 余键，张昊天，埃尔登·肖普，弗洛里斯·维尔斯，阿曼达·斯威尔金，杰弗里·尼科尔斯，尹飞扬，甘哲。Ferret-ui：基于多模态大语言模型的移动UI理解，2024。网址 [https://arxiv.org/abs/2404.05719](https://arxiv.org/abs/2404.05719)。

+   由等人 [2023] 何孝宣，张昊天，甘哲，杜贤志，张博文，王梓睿，曹亮亮，张世富，尹飞扬。Ferret：在任意粒度下参照并定位任意事物，2023。网址 [https://arxiv.org/abs/2310.07704](https://arxiv.org/abs/2310.07704)。

+   Radford 等人 [2021] Alec Radford, Jong Wook Kim, Chris Hallacy, Aditya Ramesh, Gabriel Goh, Sandhini Agarwal, Girish Sastry, Amanda Askell, Pamela Mishkin, Jack Clark, Gretchen Krueger, 和 Ilya Sutskever. 从自然语言监督中学习可转移的视觉模型, 2021. URL [https://arxiv.org/abs/2103.00020](https://arxiv.org/abs/2103.00020).

+   Zhai 等人 [2023] Xiaohua Zhai, Basil Mustafa, Alexander Kolesnikov, 和 Lucas Beyer. Sigmoid 损失用于语言图像预训练, 2023. URL [https://arxiv.org/abs/2303.15343](https://arxiv.org/abs/2303.15343).

+   Oquab 等人 [2024] Maxime Oquab, Timothée Darcet, Théo Moutakanni, Huy Vo, Marc Szafraniec, Vasil Khalidov, Pierre Fernandez, Daniel Haziza, Francisco Massa, Alaaeldin El-Nouby, Mahmoud Assran, Nicolas Ballas, Wojciech Galuba, Russell Howes, Po-Yao Huang, Shang-Wen Li, Ishan Misra, Michael Rabbat, Vasu Sharma, Gabriel Synnaeve, Hu Xu, Hervé Jegou, Julien Mairal, Patrick Labatut, Armand Joulin, 和 Piotr Bojanowski. Dinov2: 无监督学习稳健的视觉特征, 2024. URL [https://arxiv.org/abs/2304.07193](https://arxiv.org/abs/2304.07193).

+   DeepSeek-AI 等人 [2024] DeepSeek-AI, :, Xiao Bi, Deli Chen, Guanting Chen, Shanhuang Chen, Damai Dai, Chengqi Deng, Honghui Ding, Kai Dong, Qiushi Du, Zhe Fu, Huazuo Gao, Kaige Gao, Wenjun Gao, Ruiqi Ge, Kang Guan, Daya Guo, Jianzhong Guo, Guangbo Hao, Zhewen Hao, Ying He, Wenjie Hu, Panpan Huang, Erhang Li, Guowei Li, Jiashi Li, Yao Li, Y. K. Li, Wenfeng Liang, Fangyun Lin, A. X. Liu, Bo Liu, Wen Liu, Xiaodong Liu, Xin Liu, Yiyuan Liu, Haoyu Lu, Shanghao Lu, Fuli Luo, Shirong Ma, Xiaotao Nie, Tian Pei, Yishi Piao, Junjie Qiu, Hui Qu, Tongzheng Ren, Zehui Ren, Chong Ruan, Zhangli Sha, Zhihong Shao, Junxiao Song, Xuecheng Su, Jingxiang Sun, Yaofeng Sun, Minghui Tang, Bingxuan Wang, Peiyi Wang, Shiyu Wang, Yaohui Wang, Yongji Wang, Tong Wu, Y. Wu, Xin Xie, Zhenda Xie, Ziwei Xie, Yiliang Xiong, Hanwei Xu, R. X. Xu, Yanhong Xu, Dejian Yang, Yuxiang You, Shuiping Yu, Xingkai Yu, B. Zhang, Haowei Zhang, Lecong Zhang, Liyue Zhang, Mingchuan Zhang, Minghua Zhang, Wentao Zhang, Yichao Zhang, Chenggang Zhao, Yao Zhao, Shangyan Zhou, Shunfeng Zhou, Qihao Zhu, 和 Yuheng Zou. Deepseek llm: 以长期主义扩展开源语言模型, 2024. URL [https://arxiv.org/abs/2401.02954](https://arxiv.org/abs/2401.02954).

+   Huang 等人 [2022] Yupan Huang, Tengchao Lv, Lei Cui, Yutong Lu, 和 Furu Wei. Layoutlmv3: 统一文本和图像遮罩的文档AI预训练, 2022. URL [https://arxiv.org/abs/2204.08387](https://arxiv.org/abs/2204.08387).

+   Devlin 等人 [2019] Jacob Devlin, Ming-Wei Chang, Kenton Lee, 和 Kristina Toutanova. Bert: 深度双向变换器的预训练用于语言理解, 2019. URL [https://arxiv.org/abs/1810.04805](https://arxiv.org/abs/1810.04805).

+   Bao 等人 [2022] Hangbo Bao, Li Dong, Songhao Piao 和 Furu Wei. Beit：图像变换器的 Bert 预训练，2022年。网址 [https://arxiv.org/abs/2106.08254](https://arxiv.org/abs/2106.08254)。

+   Jiang 等人 [2024] Albert Q. Jiang, Alexandre Sablayrolles, Antoine Roux, Arthur Mensch, Blanche Savary, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Emma Bou Hanna, Florian Bressand, Gianna Lengyel, Guillaume Bour, Guillaume Lample, Lélio Renard Lavaud, Lucile Saulnier, Marie-Anne Lachaux, Pierre Stock, Sandeep Subramanian, Sophia Yang, Szymon Antoniak, Teven Le Scao, Théophile Gervet, Thibaut Lavril, Thomas Wang, Timothée Lacroix 和 William El Sayed. Mixtral of experts, 2024年。网址 [https://arxiv.org/abs/2401.04088](https://arxiv.org/abs/2401.04088)。

## 附录 A 动作空间

GUI 代理需要与图形用户界面（GUI）进行持续交互，以完成由人类设定的任务。一次交互可以解释为一个单独的动作，或是多个动作的结合。因此，合理设计动作空间对于增强 GUI 代理的效果至关重要。过于简化的动作空间可能限制 GUI 代理能够执行的任务种类。因此，设计一个能够涵盖大多数移动 GUI 环境下任务的复杂动作空间显得尤为重要。如表 [5](https://arxiv.org/html/2407.04346v3#A1.T5 "Table 5 ‣ Appendix A Action Space ‣ MobileFlow: A Multimodal LLM for Mobile GUI Agent") 所示，根据实际使用需求，我们设计了总共 8 种动作，并为每个动作提供了详细解释。

表 5：动作空间

| 动作 | 参数 | 说明 |
| --- | --- | --- |
| 点击 | 位置 | 在指定位置点击 |
| 长按 | 位置 | 在指定位置长按 |
| 输入 | 文本 | 在当前光标位置输入文本 |
| 滚动 | 位置列表 | 在位置列表中滑动 |
| 拖动 | 位置列表 | 长按后，沿着位置列表滑动 |
| 等待 | 时间 | 等待而不执行任何动作 |
| 任务完成 | - | 当前任务完成后，代理停止操作 |

## 附录 B MobileFlow 详情

### B.1 提示结构

最终的详细提示结构如下：

```
Picture 1: <img> image_path </img>
Imagine you are a Mobile GUI assitant. Just like a human operating a mobile phone, you
can tap and swipe the page, or use the keyboard to type some text, and you can also
answer questions.

Your task is <task> task info </task>.
The elements on the page are as follows. <list> elements </list>
Your historical moves to advance this mission are summarized below. <history> ...
<history>

Based on your task, historical actions, and the current page information, you need to
think and generate actions that can advance the task. You should respond in the
following format:
<observation>: Based on control information, describe the contents observed on the page.
<Reasoning>: To accomplish the task, contemplate what action should be generated next.
<Action>: A feasible action to accomplish the task, which could be a click, swipe,
or input text. When you determine that the task is completed, you may also output
"Finish".
<Summary>: Assuming the next action has been executed, summarizing in two or three
sentences in conjunction with the history of actions performed thus far, without
including any potential future actions or specific coordinates of controls.

```

## 附录 C 评估详情

### C.1 正样本确定

![参见说明](img/bbef2209613428a7bab40ceb6751c94e.png)

图 4：正负样本匹配情况。

确定预测的正确性并非总是一个明确或二元的决策。为了解决这个问题，我们提出的指标结合了多种方法：匹配预测的动作类型，并计算坐标区域的交并比（IoU），以为每个预测分配一个正确或错误的标签。这种方法可以进行更细致的评估，考虑到任务的复杂性以及LLM-agent所做预测的细微差异。具体来说，我们在图[4](https://arxiv.org/html/2407.04346v3#A3.F4 "图 4 ‣ C.1 正样本确定 ‣ 附录 C 评估详情 ‣ MobileFlow: 用于移动GUI代理的多模态LLM")中展示了所有匹配案例，提供了这些指标如何应用于评估预测准确性的可视化表示。

### C.2 部署详情

### C.3 数据集

为了充分训练MobileFlow管道，我们特别训练了两个模型：用于用户界面理解的UI编码器和用于标记预测的Qwen-vl-chat。

对于UI编码器的训练，我们精心挑选并清理了一个包含10万个手工标注实例的数据集，这些实例与我们当前的业务领域直接相关。随后，我们使用该数据集对UI编码器进行了微调。

对于多模态对齐，我们采用了RefCoCo[29]、ScreenQa[28]、Flickr30K[9]以及内部UI数据集的混合数据。随后，在监督微调阶段，我们使用了70K手工标注的特定业务数据。这些数据涵盖了10个不同的业务领域，例如外卖应用、医疗服务平台、保险应用、金融应用等。在构建这些数据时，我们同时设计了一个全面的动作空间，详细信息见附录A。

### C.4 训练和评估详情

在我们的实验中，我们用大约20万个UI图像预训练了UI编码器，提高了模型对字体、图像、控件及UI图像中其他元素的理解能力，并在其理解文档的基础能力上进行了增强。预训练任务的有效性通过多个UI图像下游任务进行了验证，例如图像分类和UI组件识别。

在UI编码器的训练过程中，我们对10万个标注的业务数据集进行了2个epoch的微调。对于Qwen-vl-chat的监督微调阶段，在密切监控损失函数结果后，我们确定使用学习率进行2个epoch的训练能够获得最佳性能。在评估阶段，我们使用了MobileAgent，进行了必要的数据格式和接口调整，以确保与我们的测试数据兼容，同时保持模型参数不变。此外，我们直接使用GPT-4v接口对测试数据进行标记预测。所有实验都在8个GPU（Nvidia A100）强大配置下进行。

### C.5 指标定义

整体任务成功率（WTSR） 整体任务成功率（WTSR）是一个衡量指标，用于衡量大语言模型（LLM）代理能否成功预测整个数据集中单个任务每一步的比例。

|  | $WTSR=\frac{\#SuccessIntentions}{\#AllIntentions-\#TimeOut}$ |  | (1) |
| --- | --- | --- | --- |

步骤成功率（SSR） 给定特定意图，步骤成功率（SSR）衡量大语言模型（LLM）代理能够准确预测多步骤任务中每个独立步骤的频率。

|  | $SSR=\frac{\#SuccessSteps}{\#AllSteps}$ |  | (2) |
| --- | --- | --- | --- |

终端确定率（EDR） 终端确定率（EDR）是一个量化指标，用于衡量大语言模型（LLM）代理在整个数据集的最终页面成功完成任务的比例。

|  | $EDR=\frac{\#SuccessTerminalIntentions}{\#AllIntentions-\#TimeOut}$ |  | (3) |
| --- | --- | --- | --- |

## 附录 D 应用

### D.1 软件测试

软件测试是MobileFlow的理想场景。首先，软件测试需要大量的工作来完成。根据内部调查，软件测试的时间平均比开发多出150%。在实践中，测试账户状态、测试数据、弹出窗口、算法驱动的UI显示、A/B测试可能会在传统自动化执行中产生UI路由噪声，导致误报。成功率通常较低。自动化脚本需要频繁更新，以适应UI路由中的这些噪声。

MobileFlow从三个方面解决了这个问题：

+   •

    用自然语言替代测试自动化脚本，减少了测试系统的复杂性和测试工程师对编程技能的要求。

+   •

    自然语言具有良好的处理UI噪声的能力，有时能够兼容测试账号/数据的差异。

+   •

    一些警报可以通过VAQ任务自动分析并关闭。

### D.2 广告预览与审查

广告包含多媒体内容，需要多步骤的互动来触发。这使得人工和传统自动化执行成本高，且成功率不稳定。

在这种场景下，MobileFlow可以同时服务于广告主和广告平台的角度。

+   •

    广告主：触发广告，按预期预览广告。

+   •

    广告平台：监控广告触发策略和互动逻辑，审查广告内容以防止不当内容展示。

### D.3 电子商务运营与监控

中国的大多数小型商户都有自己的电子商务业务，并且在不同的平台上运营，如淘宝店、京东店、拼多多店、支付宝小程序、微信小程序、抖音店、美团店、RED店等。日常营销和运营是商户的负担，甚至小小的错误可能导致巨大的损失，甚至破产。

开发了一款基于MobileFlow的小工具，供小型商户执行以下任务：

+   •

    检查跨平台的价格差异，通常是由优惠券/折扣设置错误引起的。

+   •

    检查跨日期的价格差异，通常由平台级别的营销活动引起。

+   •

    监控竞争商店的营销活动并跟进。

## 附录E 更多示例

![参见标题](img/061f7a16c108a9bbbec66be8742ee4d4.png)

图 5：用户指令：查看我的理赔记录。

![参见标题](img/b29ce17b2b76d82db3f57ecab72f953f.png)

图 6：展示MobileFlow在GUI代理中的应用。用户指令：帮我在自助点单中购买一杯草莓奶油布丁，选择海淀区的交通大学校园店。

![参见标题](img/4ad62c6dfb50591dfd0ec4ed91303164.png)

图 7：用户指令：查看排名前列的健康保险清单。

![参见标题](img/e21328d939bb1e4358838e9eb8df9041.png)

图 8：用户指令：请预约4月14日的胃肠科门诊，选择四川现代医院的武侯医院。
