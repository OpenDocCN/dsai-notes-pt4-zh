<!--yml

分类：未分类

日期：2025-01-11 12:27:32

-->

# GenArtist：多模态LLM作为统一图像生成和编辑的代理

> 来源：[https://arxiv.org/html/2407.05600/](https://arxiv.org/html/2407.05600/)

王振宇¹         李傲雪²         李正国²         刘喜慧³

¹ 清华大学    ² 华为诺亚方舟实验室    ³ 香港大学

wangzy20@mails.tsinghua.edu.cn, lax@pku.edu.cn,

Li.Zhenguo@huawei.com, xihuiliu@eee.hku.hk 该工作由王振宇在华为实习期间完成，通讯作者

###### 摘要

尽管现有的图像生成和编辑方法取得了成功，但当前的模型仍然在处理复杂问题时存在困难，包括复杂的文本提示，并且缺乏验证和自我修正机制，使得生成的图像不可靠。同时，单一模型往往专注于特定任务并具备相应的能力，这使得它无法满足所有用户的需求。我们提出了GenArtist，一个由多模态大语言模型（MLLM）代理协调的*统一*图像生成和编辑系统。我们将现有的各种模型整合到工具库中，并利用代理进行工具选择和执行。对于复杂问题，MLLM代理将其分解为更简单的子问题，并构建树形结构来系统地规划生成、编辑和自我修正的步骤，并进行逐步验证。通过自动生成缺失的位置信息输入并结合位置信息，可以有效地使用适当的工具解决每个子问题。实验表明，GenArtist能够执行各种生成和编辑任务，达到了最先进的性能，并超越了现有模型，如SDXL和DALL-E 3，正如图[1](https://arxiv.org/html/2407.05600v2#S1.F1 "图1 ‣ 1 引言 ‣ GenArtist：多模态LLM作为统一图像生成和编辑的代理")所示。项目页面为[https://zhenyuw16.github.io/GenArtist_page/](https://zhenyuw16.github.io/GenArtist_page/)。

## 1 引言

![参考标题](img/63a3f6bb19b1f7b48fd58d16b802ca85.png)

图1：来自GenArtist的可视化示例。它可以完成各种任务，实现统一的生成和编辑。在文本到图像生成方面，它比现有模型如SDXL和DALL-E 3获得更高的准确性。在图像编辑方面，它也在复杂编辑任务中表现出色。

随着扩散模型的最新进展 [ho2020denoising](https://arxiv.org/html/2407.05600v2#bib.bib17) ; [dhariwal2021diffusion](https://arxiv.org/html/2407.05600v2#bib.bib10)，图像生成和编辑方法迅速发展。当前图像生成和编辑的改进大致可以分为两种趋势。第一种 [ramesh2022hierarchical](https://arxiv.org/html/2407.05600v2#bib.bib40) ; [rombach2022high](https://arxiv.org/html/2407.05600v2#bib.bib41) ; [podell2023sdxl](https://arxiv.org/html/2407.05600v2#bib.bib37) ; [chen2023pixart](https://arxiv.org/html/2407.05600v2#bib.bib6) ; [betker2023improving](https://arxiv.org/html/2407.05600v2#bib.bib1) 是通过使用更先进的模型架构 [rombach2022high](https://arxiv.org/html/2407.05600v2#bib.bib41) ; [peebles2023scalable](https://arxiv.org/html/2407.05600v2#bib.bib36) 和大规模数据集，从头开始训练模型，从而扩展现有模型，实现在生成或编辑能力上的更广泛应用。这些方法通常可以增强图像生成的整体可控性和质量。第二种主要是通过微调或额外设计预训练的大规模图像生成模型，针对特定数据集进行优化，以扩展其能力 [ruiz2023dreambooth](https://arxiv.org/html/2407.05600v2#bib.bib42) ; [li2023blip](https://arxiv.org/html/2407.05600v2#bib.bib23) ; [chen2023textdiffuser](https://arxiv.org/html/2407.05600v2#bib.bib4)，或提升其在某些任务上的表现 [lian2023llm](https://arxiv.org/html/2407.05600v2#bib.bib25) ; [huang2023t2i](https://arxiv.org/html/2407.05600v2#bib.bib18)。这些方法通常是针对特定任务的，并且能够在某些特定任务上展示出优势结果。

尽管如此，目前的图像生成或编辑方法仍不完善，面临着一些急需解决的挑战，在构建符合人类需求的系统过程中：1）图像生成和编辑的需求高度多样化和变化无常，例如对物体和背景的各种要求，文本提示或指令中的各种操作需求。同时，不同模型通常各自有不同的优势和侧重点。通用模型在某些方面可能比某些微调模型表现较弱，但在处理超出分布的数据时，它们可能表现更好。因此，一个经过良好训练的模型几乎不可能满足所有人类需求，单一模型的使用往往是次优的。2）模型在处理复杂问题时仍然存在困难，例如生成任务中的冗长且复杂的句子，或编辑任务中包含多个步骤的复杂指令。扩大模型规模或进行微调可以缓解这个问题。然而，由于文本高度可变、灵活且易于组合，总是会有一些训练好的模型无法有效处理的复杂问题。3）尽管经过精心设计，模型仍然不可避免地会遇到一些失败的情况。生成的图像有时未能准确地与用户提示的内容相对应。现有模型缺乏自主评估生成图像正确性的能力，更不用说自我修正，这使得生成的图像不可靠。因此，我们真正需要的是*一个统一的图像生成与编辑系统*，它能够满足几乎所有人类需求，同时生成可靠的图像结果。

在本文中，我们提出了一个统一的图像生成与编辑系统，名为GenArtist，旨在应对上述挑战。我们的基本思路是利用多模态大语言模型（MLLM）作为AI代理，代理充当“艺术家”，根据用户的指令“绘制”图像。具体而言，作为回应用户指令，代理将分析用户需求，分解复杂问题，并进行全面规划，以制定具体解决方案。然后，它通过调用外部工具执行图像生成或编辑操作，以满足用户需求。在获得图像后，代理最终会对生成结果进行验证和修正，进一步确保生成图像的准确性。代理的核心机制包括：

复杂文本提示的分解。MLLM代理首先将复杂问题分解为几个简单的子问题。对于生成任务中的复杂文本提示，它提取单一物体概念和必要的背景元素。对于编辑任务中的复杂指令，它将复杂操作拆解为几个简单的单一编辑动作。复杂问题的分解显著提高了模型执行的可靠性。

步骤逐步验证的规划树。在任务分解后，我们构建了一种树形结构来规划子任务的执行。每个操作都是树中的一个节点，后续操作是其子节点，执行相同动作的不同工具是其兄弟节点。每个节点之后都会进行验证，以确保操作能够正确执行。然后，可以将生成、编辑和自我修正机制结合在一起。通过这个规划树，系统的执行过程可以被视为一次遍历过程，整个系统能够协调一致地运作。

位置感知工具执行。大多数基于对象的工具需要位置相关的输入，例如需要操作的对象的位置。这些必要的输入可能无法由用户提供。现有的多模态语言模型（MLLMs）也缺乏位置感知，无法提供准确的位置引导。因此，我们引入了一组辅助工具，通过检测模型自动完成这些位置相关的输入，并通过检测模型将位置信息引入到 MLLM 代理中，以便执行工具操作。

我们的主要贡献可以总结如下：

+   •

    我们提出了 GenArtist，一个统一的图像生成与编辑系统。MLLM 代理作为“大脑”，协调并管理整个过程。据我们所知，这是第一个统一的系统，涵盖了绝大多数现有的生成与编辑任务。

+   •

    通过将操作视为节点并构建规划树，我们的 MLLM 代理可以为生成和编辑任务进行调度，并自动验证和自我修正生成的图像。这显著增强了用户指令对图像的可控性。

+   •

    通过将位置信息纳入集成工具库，并采用辅助工具提供缺失的位置相关输入，代理能够进行工具选择并调用最合适的工具，为生成和编辑中的各种任务提供统一接口。

大量实验证明了我们的 GenArtist 的有效性。与 DALL-E 3 [betker2023improving](https://arxiv.org/html/2407.05600v2#bib.bib1) 相比，它在 T2I-CompBench [huang2023t2i](https://arxiv.org/html/2407.05600v2#bib.bib18)（一个综合性的开放世界组合 T2I 生成基准测试）上取得了超过 7% 的提升，并且在图像编辑基准 MagicBrush [zhang2024magicbrush](https://arxiv.org/html/2407.05600v2#bib.bib61) 上也获得了最先进的表现。正如图示例 [1](https://arxiv.org/html/2407.05600v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ GenArtist: Multimodal LLM as an Agent for Unified Image Generation and Editing") 所展示的，GenArtist 成功地作为一个统一的图像生成与编辑系统。

## 2 相关工作

图像生成与编辑。随着扩散模型的发展，[dhariwal2021diffusion](https://arxiv.org/html/2407.05600v2#bib.bib10)；[ho2020denoising](https://arxiv.org/html/2407.05600v2#bib.bib17)，图像生成与编辑取得了显著的成功。许多通用的文本到图像生成方法 [rombach2022high](https://arxiv.org/html/2407.05600v2#bib.bib41)；[saharia2022photorealistic](https://arxiv.org/html/2407.05600v2#bib.bib43)；[podell2023sdxl](https://arxiv.org/html/2407.05600v2#bib.bib37)；[chen2023pixart](https://arxiv.org/html/2407.05600v2#bib.bib6) 以及编辑方法 [brooks2023instructpix2pix](https://arxiv.org/html/2407.05600v2#bib.bib2)；[zhang2024magicbrush](https://arxiv.org/html/2407.05600v2#bib.bib61)；[sheynin2023emu](https://arxiv.org/html/2407.05600v2#bib.bib45)；[geng2023instructdiffusion](https://arxiv.org/html/2407.05600v2#bib.bib15) 已经提出，并且达到了高质量的生成图像。基于这些通用模型，许多方法通过微调或设计附加模块来处理一些特定任务，如定制化图像生成 [ruiz2023dreambooth](https://arxiv.org/html/2407.05600v2#bib.bib42)；[kumari2023multi](https://arxiv.org/html/2407.05600v2#bib.bib21)；[li2023blip](https://arxiv.org/html/2407.05600v2#bib.bib23)；[liu2023cones](https://arxiv.org/html/2407.05600v2#bib.bib30)，带有文本渲染的图像生成 [chen2024textdiffuser](https://arxiv.org/html/2407.05600v2#bib.bib5)；[chen2023textdiffuser](https://arxiv.org/html/2407.05600v2#bib.bib4)，基于示例的图像编辑 [yang2023paint](https://arxiv.org/html/2407.05600v2#bib.bib56)；[chen2023anydoor](https://arxiv.org/html/2407.05600v2#bib.bib8)，专注于人物的图像生成 [xiao2023fastcomposer](https://arxiv.org/html/2407.05600v2#bib.bib53)。与此同时，一些方法旨在提高文本对图像的可控性。例如，ControlNet [zhang2023adding](https://arxiv.org/html/2407.05600v2#bib.bib62) 控制稳定扩散（Stable Diffusion），使用各种条件输入，如 Canny 边缘，[voynov2023sketch](https://arxiv.org/html/2407.05600v2#bib.bib50) 采用草图图像作为条件，布局到图像的方法 [li2023gligen](https://arxiv.org/html/2407.05600v2#bib.bib24)；[xie2023boxdiff](https://arxiv.org/html/2407.05600v2#bib.bib54)；[lian2023llm](https://arxiv.org/html/2407.05600v2#bib.bib25)；[chen2024training](https://arxiv.org/html/2407.05600v2#bib.bib7) 根据给定的对象边界框合成图像。尽管取得了成功，这些方法仍然专注于特定任务，因此无法支持统一的图像生成与编辑。

AI代理。大型语言模型（LLMs），如ChatGPT、Llama [touvron2023llama](https://arxiv.org/html/2407.05600v2#bib.bib48)；[touvron2023llama2](https://arxiv.org/html/2407.05600v2#bib.bib49)，在自然语言处理方面展现了令人印象深刻的能力。对于多模态大型语言模型（MLLMs）而言，视觉能力的加入，例如LLaVA [liu2024visual](https://arxiv.org/html/2407.05600v2#bib.bib26)，Claude，GPT-4 [gpt42023](https://arxiv.org/html/2407.05600v2#bib.bib34)，进一步使这些模型能够处理视觉数据。最近，LLMs开始作为执行复杂任务的代理被广泛采用。这些工作 [yang2023auto](https://arxiv.org/html/2407.05600v2#bib.bib57)；[shen2023hugginggpt](https://arxiv.org/html/2407.05600v2#bib.bib44)；[liu2023internchat](https://arxiv.org/html/2407.05600v2#bib.bib29) 将LLMs应用于学习使用工具进行任务，例如视觉交互、语音处理、组合视觉任务 [gupta2023visual](https://arxiv.org/html/2407.05600v2#bib.bib16)，软件开发 [qian2023communicative](https://arxiv.org/html/2407.05600v2#bib.bib38)，游戏 [meta2022human](https://arxiv.org/html/2407.05600v2#bib.bib11)，APP使用 [yang2023appagent](https://arxiv.org/html/2407.05600v2#bib.bib60) 或数学 [xin2023lego](https://arxiv.org/html/2407.05600v2#bib.bib55)。最近，AI代理的概念也开始应用于与图像生成相关的任务。例如，[lian2023llm](https://arxiv.org/html/2407.05600v2#bib.bib25)；[feng2024layoutgpt](https://arxiv.org/html/2407.05600v2#bib.bib13) 设计场景布局，[wu2023self](https://arxiv.org/html/2407.05600v2#bib.bib52) 利用LLMs进行自我纠错，[wang2024divide](https://arxiv.org/html/2407.05600v2#bib.bib51)；[yang2024mastering](https://arxiv.org/html/2407.05600v2#bib.bib59) 针对复杂的文本到图像生成问题中的MLLMs，和 [qin2024diffusiongpt](https://arxiv.org/html/2407.05600v2#bib.bib39) 利用LLM进行文本到图像生成任务中的模型选择。

![参见说明](img/ee6ab202f6b69a82c4b18ea24aba5e09.png)

图2：我们GenArtist的概述。MLLM代理负责使用树状结构分解问题并进行规划，然后调用工具来解决问题。将代理作为“大脑”有效地实现了统一的生成和编辑系统。

## 3 方法

我们的 GenArtist 概述如图 [2](https://arxiv.org/html/2407.05600v2#S2.F2 "图 2 ‣ 2 相关工作 ‣ GenArtist：作为统一图像生成和编辑代理的多模态 LLM") 所示。MLLM 代理协调整个系统。其主要职责围绕着将复杂任务进行分解，并构建带有逐步验证的规划树，用于图像生成、编辑和自我修正。它调用来自图像生成工具库和编辑工具库的工具来执行特定操作，辅助工具库则用于提供缺失的位置信息值。

### 3.1 带有逐步验证的规划树

分解。当面对复杂的提示输入时，现有的方法通常无法理解所有的需求，这会影响模型结果的可控性和可靠性。因此，MLLM 代理首先对复杂问题进行分解。对于生成任务，它根据文本提示分解对象和背景信息。它提取嵌入在文本提示中的离散对象及其相关属性。对于背景信息，它主要分析输入文本所要求的整体场景和图像风格。对于编辑任务，它将复杂的编辑操作分解为若干个具体的动作，如添加、移动、删除，并转化为简单的编辑指令。经过分解后，简化的操作相对更容易处理，从而提高了模型执行的可靠性。

树的构建。分解之后，我们将所有操作组织成树形结构进行规划。这样的树主要由三种类型的节点组成：初始节点、生成节点和编辑节点。初始节点作为树的根节点，标志着系统的开始。生成节点涉及使用生成工具库中的工具进行图像生成，而编辑节点则涉及使用来自编辑工具库的相应工具执行单一的编辑操作。对于纯图像编辑任务，生成节点将会缺失。

在实际应用中，由于生成的图像的正确性无法得到保证，我们引入了自我纠正机制来评估和修正生成结果。因此，每个生成节点都有一个完全由编辑节点组成的子树用于自我纠正。在生成节点中的工具被调用并进行验证后，这个子树将由MLLM代理自适应生成。具体而言，在验证后，我们指示MLLM代理设计一系列相应的编辑操作来纠正图像。以图[2](https://arxiv.org/html/2407.05600v2#S2.F2 "图 2 ‣ 2 相关工作 ‣ GenArtist: 多模态 LLM 作为统一图像生成与编辑的代理")中的示例为例，应进行的编辑操作包括“添加一辆黑色自行车”、“将滑板车的颜色改为蓝色”、“添加一只鸟”。这些操作被组织成树结构，成为生成节点的子树，从而实现自我纠正的具体规划。

![参见说明](img/0c78c4ec8a86bc681cdd40dc862dc127.png)

图 3：规划树示意图。“替代生成工具”节点的子树将在验证后自适应生成，而“指令”节点的子树与左侧相同。

每个生成或编辑操作对应树中的一个节点，其后续操作作为其子节点。该构建最初形成了一个“链条”，实现了规划链。然后，我们注意到通常可以利用不同的工具来解决同一个问题。例如，对于将物体添加到图像中的操作，我们可以使用专门设计的物体添加工具或通过将添加操作转化为文本指令的基于指令的编辑模型。同样，针对属性编辑，我们可以使用属性编辑模型或利用替换或基于指令的编辑模型。此外，许多生成工具可以实现文本到图像的生成，改变随机种子也能产生不同的输出。我们将这些节点视为兄弟节点，所有节点都作为其父节点的子节点，它们共享同一个子树，包含后续的编辑操作。MLLM代理选择的工具将作为最优子节点，并排在最左侧。通过这种方式，我们建立了树的结构。图[2](https://arxiv.org/html/2407.05600v2#S2.F2 "图 2 ‣ 2 相关工作 ‣ GenArtist: 多模态 LLM 作为统一图像生成与编辑的代理")案例的示意图见图[3](https://arxiv.org/html/2407.05600v2#S3.F3 "图 3 ‣ 3.1 步骤验证规划树 ‣ 3 方法 ‣ GenArtist: 多模态 LLM 作为统一图像生成与编辑的代理")（为了简化，我们省略了一些具有相同结构的子树或在生成节点后自适应生成的子树，以及一些关于变化随机种子的节点）。

规划。一旦树结构建立，针对自我修正或整个系统的规划可以视为该结构的先序遍历。对于某个特定节点，会调用其对应的工具进行操作，接着进行验证以确定编辑是否成功。如果成功，流程将继续到其最左侧的子节点进行后续操作，并删除其兄弟节点。如果不成功，流程会回溯到其兄弟节点，并移除其子树。该过程将持续进行，直到生成的图像正确，*即*，当最低级别的节点成功执行时。我们还可以限制树的分支因子或节点数量，以便提前终止，并要求代理返回最准确的图像。

验证。如上所述，验证机制在树的构建和执行过程中起着至关重要的作用。通过MLLM的多模态感知能力，代理验证生成图像的正确性。验证的主要内容包括文本中包含的物体及其自身的属性，如颜色、形状、纹理、物体的位置、这些不同物体之间的关系。此外，还会考虑背景、场景、整体风格以及生成图像的美学质量。由于现有模型的感知能力通常优于生成能力，采用这样的验证机制能够有效地评估生成图像的正确性。

另需指出的是，在验证过程中，除了生成图像的准确性外，代理还需要评估图像的美学质量。如果整体质量较差，代理将使用不同的生成工具或选择不同的随机种子重新生成图像，以确保图像的整体质量。同时，作为一个以代理为中心的系统，该框架在人机交互方面也具有灵活性。在验证过程中，可以适当地集成人类反馈。通过将人类对图像整体质量的评价和反馈纳入其中，可以进一步提高生成图像的质量。

### 3.2 工具库

在构建规划树之后，代理通过调用外部工具执行每个节点，最终解决问题。我们首先介绍GenArtist中使用的工具。MLLM代理使用的主要工具大致可以分为图像生成工具库和编辑工具库。目前我们使用的具体工具列在表[1](https://arxiv.org/html/2407.05600v2#S3.T1 "Table 1 ‣ 3.2 Tool Library ‣ 3 Method ‣ GenArtist: Multimodal LLM as an Agent for Unified Image Generation and Editing")中，并且可以无缝添加一些新工具，从而扩展工具库。为了辅助后续的工具选择，我们需要向MLLM代理传达工具执行的具体任务、所需输入以及其特点和优势。介绍工具的提示内容具体包括以下几个部分：

+   •

    工具技能和名称。简要描述工具相关的任务及其名称，如表[1](https://arxiv.org/html/2407.05600v2#S3.T1 "Table 1 ‣ 3.2 Tool Library ‣ 3 Method ‣ GenArtist: Multimodal LLM as an Agent for Unified Image Generation and Editing")所列，例如（文本到图像，SDXL），（canny到图像，ControlNet），（对象移除，LaMa）。它作为一个唯一标识符，帮助代理区分所使用的工具。

+   •

    工具所需输入。指的是执行工具所需的具体输入。例如，文本到图像模型需要“文本”作为生成的输入，自定义模型也需要“主题图像”用于个性化生成。大多数对象级编辑工具需要有关“对象名称”和“对象位置”的指令。

+   •

    工具的特点和优势。主要介绍工具的具体特点，作为代理在选择工具时的重要参考。例如，SDXL是一个通用的文本到图像生成模型，LMD通常严格控制场景布局，适用于组合型的文本到图像生成，通常文本提示包含多个对象，BoxDiff相对较为宽松地控制场景布局，TextDiffuser则专门为带有文本渲染的图像生成设计。

表 1：GenArtist 使用的工具库，包括工具名称及其技能。主要工具来自生成工具库和编辑工具库。以下模型表示我们当前版本中使用的所有工具，同时新模型可以无缝添加。

|        生成工具 | 编辑工具 |
| --- | --- |
| 技能 | 工具 | 技能 | 工具 |
| 文本到图像 | SDXL [podell2023sdxl](https://arxiv.org/html/2407.05600v2#bib.bib37) |  |  |
| 文本到图像 | PixArt-$\alpha$ [chen2023pixart](https://arxiv.org/html/2407.05600v2#bib.bib6) | 对象添加 | AnyDoor [chen2023anydoor](https://arxiv.org/html/2407.05600v2#bib.bib8) |
| image-to-image | Stable Diffusion v2 [rombach2022high](https://arxiv.org/html/2407.05600v2#bib.bib41) | 对象移除 | LaMa [suvorov2022resolution](https://arxiv.org/html/2407.05600v2#bib.bib47) |
| layout-to-image | LMD [lian2023llm](https://arxiv.org/html/2407.05600v2#bib.bib25) | 对象替换 | AnyDoor [chen2023anydoor](https://arxiv.org/html/2407.05600v2#bib.bib8) |
| layout-to-image | BoxDiff [xie2023boxdiff](https://arxiv.org/html/2407.05600v2#bib.bib54) | 属性编辑 | DiffEdit [couairon2022diffedit](https://arxiv.org/html/2407.05600v2#bib.bib9) |
| 单一对象定制 | BLIP-Diffusion [li2023blip](https://arxiv.org/html/2407.05600v2#bib.bib23) | 基于指令 | MagicBrush [zhang2024magicbrush](https://arxiv.org/html/2407.05600v2#bib.bib61) |
| 多对象定制 | $\lambda$-ECLIPSE [patel2024lambda](https://arxiv.org/html/2407.05600v2#bib.bib35) | 拖拽（细节） | DragDiffusion [shi2023dragdiffusion](https://arxiv.org/html/2407.05600v2#bib.bib46) |
| 超分辨率 | SDXL [podell2023sdxl](https://arxiv.org/html/2407.05600v2#bib.bib37) | 拖拽（对象） | DragonDiffusion [mou2023dragondiffusion](https://arxiv.org/html/2407.05600v2#bib.bib33) |
| 带文本的图像 | TextDiffuser [chen2023textdiffuser](https://arxiv.org/html/2407.05600v2#bib.bib4) | 风格迁移 | InST [zhang2023inversion](https://arxiv.org/html/2407.05600v2#bib.bib64) |
| {canny, depth …}-to-image | ControlNet [zhang2023adding](https://arxiv.org/html/2407.05600v2#bib.bib62) |  |  |
|   |  |  |  |

### 3.3 位置感知工具执行

使用工具库，MLLM代理将进一步执行工具选择和执行，以利用合适的工具来完成图像生成或编辑任务。在工具执行之前，我们通过两项设计来补偿用户输入和MLLM代理中位置信息的不足：

位置相关输入补偿。在实际应用中，常常会遇到代理选择了合适的工具，但某些必要的用户输入缺失的情况。这些用户输入主要与位置相关。例如，对于一些包含多个物体的复杂文本提示，布局到图像工具可能是合适的选择。然而，用户不一定会提供场景布局，通常只会提供文本提示。在这种情况下，由于缺少一些必要的输入，这些合适的工具无法直接调用。因此，我们引入了辅助工具库来提供这些位置相关的缺失输入。这个辅助工具库主要包含：1）如物体检测[liu2023grounding](https://arxiv.org/html/2407.05600v2#bib.bib28)或分割[kirillov2023segment](https://arxiv.org/html/2407.05600v2#bib.bib20)模型等定位模型，用于为某些物体级编辑工具提供物体的位置信息；2）ControlNet的预处理器[zhang2023adding](https://arxiv.org/html/2407.05600v2#bib.bib62)，如姿态估计器、Canny边缘提取器、深度图提取器；3）一些LLM实现的工具，如场景布局生成器[lian2023llm](https://arxiv.org/html/2407.05600v2#bib.bib25)；[feng2024layoutgpt](https://arxiv.org/html/2407.05600v2#bib.bib13)。MLLM代理在必要时可以自动调用这些辅助工具，以确保能够使用最合适的工具来处理用户指令，而不是仅依赖用户提供的输入来选择工具。

位置相关信息介绍。现有的多模态大语言模型（MLLMs）主要聚焦于文本理解和整体图像感知，对于图像中精确的位置信息关注较少。MLLMs能够轻松判断物体是否存在于图像中，但有时在辨别物体之间的空间关系上存在困难，例如某个特定物体是在另一个物体的左侧还是右侧。对于需要位置相关输入的工具（如物体级编辑工具），这些MLLMs也更难提供准确的指导。为了解决这个问题，我们在输入图像上使用了物体检测器，并将检测到的物体及其边界框作为提示的一部分，提供空间参考给MLLM代理。通过这种方式，代理可以有效地判断图像中某些工具应该操作的位置。

因此，代理进行工具选择的提示主要包括以下几个部分：

+   •

    任务指令。其主要目的是明确代理的任务，*即*，在统一的生成和编辑系统中进行工具选择。同时，它将用户指令作为输入，并指定输出格式。我们要求代理以{"tool_name":tools, "input":inputs}的格式输出，并使用预定义的标识符标注缺失的输入。

+   •

    工具介绍。我们将每个工具的描述输入到代理中，按照之前描述的格式。关于工具的详细信息将作为选择工具过程中的重要参考。我们还声明，选择工具的主要标准是工具的适用性，而非给定输入的内容，因为缺失的输入可以自动生成。

+   •

    位置信息。来自对象检测器的输出被用于并提供给 MLLM 代理，以补偿缺失的位置信息。

总结来说，工具执行的基本步骤如下：首先，确定任务是否涉及图像生成或编辑。接下来，根据指令和工具的特点进行工具选择，并以所需格式输出。最后，对于缺失的输入，使用辅助工具完成它们。在完成这些步骤后，代理将能够正确执行适当的工具，从而初步满足用户的需求。多种工具的集成、选择和执行大大促进了统一图像生成和编辑系统的发展。

## 4 实验

表 2：T2I-CompBench 上与现有文本到图像生成模型和组合方法的定量比较。我们的方法在属性绑定、对象关系和复杂组合的生成能力上表现出优越性。我们使用官方更新后的代码进行评估，该代码更新了名词短语的数量。因此，某些方法的度量值可能低于原文中报告的值。

|      模型 | 属性绑定 | 对象关系 | 复杂性$\uparrow$ |
| --- | --- | --- | --- |
| 颜色 $\uparrow$ | 形状$\uparrow$ | 纹理$\uparrow$ | 空间$\uparrow$ | 非空间$\uparrow$ |
| 稳定扩散 v1.4 [rombach2022high](https://arxiv.org/html/2407.05600v2#bib.bib41) | 0.3765 | 0.3576 | 0.4156 | 0.1246 | 0.3079 | 0.3080 |
| 稳定扩散 v2 [rombach2022high](https://arxiv.org/html/2407.05600v2#bib.bib41) | 0.5065 | 0.4221 | 0.4922 | 0.1342 | 0.3096 | 0.3386 |
| DALL-E 2 [ramesh2022hierarchical](https://arxiv.org/html/2407.05600v2#bib.bib40) | 0.5750 | 0.5464 | 0.6374 | 0.1283 | 0.3043 | 0.3696 |
| 可组合扩散 [liu2022compositional](https://arxiv.org/html/2407.05600v2#bib.bib27) | 0.4063 | 0.3299 | 0.3645 | 0.0800 | 0.2980 | 0.2898 |
| 结构扩散 [feng2023trainingfree](https://arxiv.org/html/2407.05600v2#bib.bib12) | 0.4990 | 0.4218 | 0.4900 | 0.1386 | 0.3111 | 0.3355 |
| Attn-Exct [chefer2023attend](https://arxiv.org/html/2407.05600v2#bib.bib3) | 0.6400 | 0.4517 | 0.5963 | 0.1455 | 0.3109 | 0.3401 |
| GORS [huang2023t2i](https://arxiv.org/html/2407.05600v2#bib.bib18) | 0.6603 | 0.4785 | 0.6287 | 0.1815 | 0.3193 | 0.3328 |
| SDXL [podell2023sdxl](https://arxiv.org/html/2407.05600v2#bib.bib37) | 0.5879 | 0.4687 | 0.5299 | 0.2133 | 0.3119 | 0.3237 |
| PixArt-$\alpha$ [chen2023pixart](https://arxiv.org/html/2407.05600v2#bib.bib6) | 0.6690 | 0.4927 | 0.6477 | 0.2064 | 0.3197 | 0.3433 |
| CompAgent [wang2024divide](https://arxiv.org/html/2407.05600v2#bib.bib51) | 0.7760 | 0.6105 | 0.7008 | 0.4837 | 0.3212 | 0.3972 |
| DALL-E 3 [betker2023improving](https://arxiv.org/html/2407.05600v2#bib.bib1) | 0.7785 | 0.6205 | 0.7036 | 0.2865 | 0.3003 | 0.3773 |
| GenArtist (我们的) | 0.8482 | 0.6948 | 0.7709 | 0.5437 | 0.3346 | 0.4499 |
|   |  |  |  |  |  |  |

在本节中，我们通过大量图像生成和编辑实验展示了我们的GenArtist及其统一能力的有效性。对于图像生成，我们主要在最新的T2I-CompBench基准测试上进行定量比较 [huang2023t2i](https://arxiv.org/html/2407.05600v2#bib.bib18)。该基准测试主要涉及带有复杂文本提示的图像生成，涉及多个物体及其属性或关系。对于图像编辑，我们主要在MagicBrush基准测试上进行比较 [zhang2024magicbrush](https://arxiv.org/html/2407.05600v2#bib.bib61)，该测试涉及多种类型的文本指令，包括单轮和多轮对话的图像编辑。我们选择GPT-4V [gpt42023](https://arxiv.org/html/2407.05600v2#bib.bib34)作为我们的MLLM代理。在定量对比实验中，我们将编辑树限制为二叉树。

### 4.1 与图像生成方法的比较

我们在表格[2](https://arxiv.org/html/2407.05600v2#S4.T2 "Table 2 ‣ 4 Experiments ‣ GenArtist: Multimodal LLM as an Agent for Unified Image Generation and Editing")中列出了我们GenArtist的定量指标结果，并与现有的最先进文本到图像合成方法进行了比较。从中可以看出，我们的GenArtist在所有子类别中始终表现出更好的性能。这证明了，对于文本到图像生成任务，我们的系统有效地实现了更好的文本到图像对应关系控制和更高的生成图像准确度，尤其是在复杂文本提示的情况下。可以观察到，基于Stable Diffusion的模型，无论是扩展模型如SDXL、PixArt-$\alpha$，还是专门为此背景设计的方法如Attn-Exct、GORS，都能实现更高的准确度。与此相比，我们的方法通过整合多种模型作为工具，有效地利用了这两类方法的优势。此外，自我修正机制进一步确保了生成图像的准确性。与当前最先进的模型DALL-E 3相比，我们的方法在属性绑定上提高了近7%，在空间关系上提高了超过20%，部分原因在于包含了位置敏感工具并在选择工具时输入了位置信息。与CompAgent（同样使用AI代理进行组合文本到图像生成的一种方法）相比，GenArtist的平均改进幅度为6%，部分原因是我们的系统包含了更为全面的生成与自我修正框架。因此，我们统一系统在图像生成方面的能力得到了充分证明。

### 4.2 与图像编辑方法的比较

表格3：与现有图像编辑方法的定量比较。多轮设置评估在编辑会话中对前一张源图像进行迭代编辑后的图像。

| 设置 | 方法 | L1$\downarrow$ | L2$\downarrow$ | CLIP-I$\uparrow$ | DINO$\uparrow$ | CLIP-T$\uparrow$ |
| --- | --- | --- | --- | --- | --- | --- |
| 单轮 | Null Text Inversion [mokady2023null](https://arxiv.org/html/2407.05600v2#bib.bib32) | 0.0749 | 0.0197 | 0.8827 | 0.8206 | 0.2737 |
| HIVE [zhang2023hive](https://arxiv.org/html/2407.05600v2#bib.bib63) | 0.1092 | 0.0341 | 0.8519 | 0.7500 | 0.2752 |
| InstructPix2Pix [brooks2023instructpix2pix](https://arxiv.org/html/2407.05600v2#bib.bib2) | 0.1122 | 0.0371 | 0.8524 | 0.7428 | 0.2764 |
| MagicBrush [zhang2024magicbrush](https://arxiv.org/html/2407.05600v2#bib.bib61) | 0.0625 | 0.0203 | 0.9332 | 0.8987 | 0.2781 |
| SmartEdit [huang2023smartedit](https://arxiv.org/html/2407.05600v2#bib.bib19) | 0.0810 | - | 0.9140 | 0.8150 | 0.3050 |
| GenArtist (我们的方法) | 0.0536 | 0.0147 | 0.9403 | 0.9131 | 0.3129 |
| 多轮 | Null Text Inversion [mokady2023null](https://arxiv.org/html/2407.05600v2#bib.bib32) | 0.1057 | 0.0335 | 0.8468 | 0.7529 | 0.2710 |
| HIVE [zhang2023hive](https://arxiv.org/html/2407.05600v2#bib.bib63) | 0.1521 | 0.0557 | 0.8004 | 0.6463 | 0.2673 |
| InstructPix2Pix [brooks2023instructpix2pix](https://arxiv.org/html/2407.05600v2#bib.bib2) | 0.1584 | 0.0598 | 0.7924 | 0.6177 | 0.2726 |
| MagicBrush [zhang2024magicbrush](https://arxiv.org/html/2407.05600v2#bib.bib61) | 0.0964 | 0.0353 | 0.8924 | 0.8273 | 0.2754 |
| GenArtist (我们的方法) | 0.0858 | 0.0298 | 0.9071 | 0.8492 | 0.3067 |
|   |  |  |  |  |  |  |

然后我们列出了在图像编辑基准 MagicBrush 上的比较定量结果，见表 [3](https://arxiv.org/html/2407.05600v2#S4.T3 "表 3 ‣ 4.2 与图像编辑方法的比较 ‣ 4 实验 ‣ GenArtist: 多模态 LLM 作为统一图像生成与编辑的代理")。与之前的全局描述引导方法（如 Null Text Inversion）和指令引导方法（如 InstrctPix2Pix 和 MagicBrush）相比，我们的 GenArtist 在单回合和多回合设置下均实现了卓越的编辑效果。其主要原因在于编辑操作高度多样化，单一模型难以在所有这些多样化的编辑操作中表现出色。相比之下，我们的方法能够全面利用不同模型的优势。此外，规划树能够有效地考虑模型执行失败的场景，从而使得编辑结果更加可靠和准确。因此，我们统一系统在图像编辑方面的能力得以展现。

### 4.3 消融研究

表 4：T2I-CompBench 的消融研究。上半部分是关于生成工具库中的相关工具，接下来我们分别研究工具选择和规划机制。

|      方法 | 属性绑定 | 对象关系 | 复杂度$\uparrow$ |
| --- | --- | --- | --- |
| 颜色 $\uparrow$ | 形状$\uparrow$ | 纹理$\uparrow$ | 空间$\uparrow$ | 非空间$\uparrow$ |
| 稳定扩散 v2 [rombach2022high](https://arxiv.org/html/2407.05600v2#bib.bib41) | 0.5065 | 0.4221 | 0.4922 | 0.1342 | 0.3096 | 0.3386 |
| LMD [lian2023llm](https://arxiv.org/html/2407.05600v2#bib.bib25) | 0.5736 | 0.5334 | 0.5227 | 0.2704 | 0.3073 | 0.3083 |
| BoxDiff [xie2023boxdiff](https://arxiv.org/html/2407.05600v2#bib.bib54) | 0.6374 | 0.4869 | 0.6100 | 0.2625 | 0.3158 | 0.3457 |
| $\lambda$-ECLIPSE [patel2024lambda](https://arxiv.org/html/2407.05600v2#bib.bib35) | 0.4581 | 0.4420 | 0.5084 | 0.1285 | 0.2922 | 0.3131 |
| SDXL [podell2023sdxl](https://arxiv.org/html/2407.05600v2#bib.bib37) | 0.5879 | 0.4687 | 0.5299 | 0.2133 | 0.3119 | 0.3237 |
| PixArt-$\alpha$ [chen2023pixart](https://arxiv.org/html/2407.05600v2#bib.bib6) | 0.6690 | 0.4927 | 0.6477 | 0.2064 | 0.3197 | 0.3433 |
| + 工具选择 | 0.7028 | 0.5764 | 0.6883 | 0.4305 | 0.3187 | 0.3739 |
| + 规划链 | 0.7509 | 0.6045 | 0.7192 | 0.4787 | 0.3216 | 0.4095 |
| + 规划树 | 0.8482 | 0.6948 | 0.7709 | 0.5437 | 0.3346 | 0.4499 |
|   |  |  |  |  |  |  |

我们最终在 T2I-CompBench 基准上进行了消融研究，并将结果列在表 [4](https://arxiv.org/html/2407.05600v2#S4.T4 "表 4 ‣ 4.3 消融研究 ‣ 4 实验 ‣ GenArtist: 多模态 LLM 作为统一图像生成和编辑的代理") 中。我们提供了稳定扩散（Stable Diffusion）的结果作为参考。顶部部分包括与任务相关的各种工具，包括文本到图像、布局到图像和定制生成方法。可以观察到，通过扩大规模或额外设计，这些工具普遍取得了比稳定扩散更好的结果。在 MLLM 代理选择工具后，定量指标超越了所有这些工具。这表明该代理能够根据文本提示的内容有效选择合适的工具，从而实现比所有这些工具更优的性能。如果我们使用链式结构进行规划以进一步修正图像，我们实现了平均 3% 的提升，证明了验证和修正错误结果的必要性。此外，通过利用树结构，我们可以进一步考虑和处理编辑工具失败的情况，从而得到更加可靠的输出结果。这样的消融研究说明了整合多个模型作为工具以及利用树结构进行规划的必要性。我们基于代理的系统设计的合理性也得到了证明。

表 5：关于在 T2I-CompBench 上基于位置感知工具执行的消融研究。

|  | 空间$\uparrow$ | 复杂性 $\uparrow$ |
| --- | --- | --- |
| 不含位置信息 | 0.4577 | 0.4083 |
| 含位置信息 | 0.5437 | 0.4499 |

关于位置感知工具执行，我们在表 [5](https://arxiv.org/html/2407.05600v2#S4.T5 "表 5 ‣ 4.3 消融研究 ‣ 4 实验 ‣ GenArtist: 多模态 LLM 作为统一图像生成和编辑的代理") 中列出了相应的消融研究。我们评估了 T2I-CompBench 中空间和复杂性方面的表现，因为这两个方面主要涉及基于位置敏感的文本提示进行图像生成。由于多模态大型模型通常对位置敏感信息不够敏感，缺少位置信息时性能有限，仅比工具选择结果稍有改进。引入位置信息后，增强了空间意识，在空间和复杂性方面都有了显著改善。这验证了我们设计的合理性。

![参见图例](img/a5ebbd840ae558f200946e4a3de6e615.png)

图 4：图像生成任务规划树的可视化。

![参见图例](img/eef975d0ed2f71a06945f6bb21a71a9d.png)

图 5：图像编辑任务规划树的可视化。

我们在图[4](https://arxiv.org/html/2407.05600v2#S4.F4 "图 4 ‣ 4.3 消融研究 ‣ 4 实验 ‣ GenArtist：多模态LLM作为统一图像生成与编辑的代理")中进一步列出了一些可视化的生成示例，以说明我们的规划树及系统的运行方式。在第一个示例中，由于文本提示包含多个对象，代理选择了LMD工具进行生成。然而，图像中仍然存在一些错误。代理首先尝试使用属性编辑工具将最左侧的羊改为白色，但失败了。代理进一步尝试使用替换工具修改颜色，但替换后羊的大小变得太小，且不太明显。然后，代理选择移除黑色羊，并添加一只白色羊，成功实现了与编辑颜色相同的效果。最后，代理使用对象添加工具在右侧添加了一只山羊，确保图像最终与文本提示准确匹配。在第二个示例中，由于BoxDiff生成图像中的头发不够清晰，编辑工具无法使头发正确匹配“长黑发”的描述。因此，代理调用了另一个生成工具，以确保最终图像的正确性。图[5](https://arxiv.org/html/2407.05600v2#S4.F5 "图 5 ‣ 4.3 消融研究 ‣ 4 实验 ‣ GenArtist：多模态LLM作为统一图像生成与编辑的代理")中还提供了一些图像编辑示例。

### 4.4 错误案例分析

![参考标题](img/49d08a9b9f64365b63695a0bbbb5c0b7.png)

图6：由编辑工具能力（左）或定位工具错误输出（右）引起的GenArtist错误案例。

我们在图[6](https://arxiv.org/html/2407.05600v2#S4.F6 "图 6 ‣ 4.4 错误案例分析 ‣ 4 实验 ‣ GenArtist：多模态LLM作为统一图像生成与编辑的代理")中进一步分析了我们GenArtist的一些错误案例。如图所示，有时尽管代理正确规划了工具的具体执行，工具本身的局限性仍然会妨碍正确执行，从而导致错误结果。例如，在第一个案例中，要求添加一个非常小的蓝色杯子。然而，由于现有编辑工具缺乏精细分辨率能力，生成的蓝色杯子大小不准确。此外，如第二个案例所示，定位工具的输出错误也会影响最终结果。例如，当要求移除三明治中的生菜时，分割模型未能准确识别对象的部分，导致错误的移除操作。利用更强大的工具或在验证阶段加入一些人工反馈，可以有效解决此问题。

## 5 结论

本文提出了GenArtist，一个由MLLM代理协调的统一图像生成和编辑系统。通过分解输入问题，采用树形结构进行规划，并调用外部工具进行执行，MLLM代理充当“大脑”来为各种任务生成高保真和准确的图像。大量实验表明，GenArtist很好地解决了图像生成和编辑中的复杂问题，并且与现有方法相比，达到了最先进的性能。它在广泛生成任务中的能力也验证了其统一性。我们相信，利用代理实现一个统一的图像生成和编辑系统，并增强可控性的这种方法，可以为未来的研究提供宝贵的见解，我们认为这是迈向自主代理未来的重要一步。

## 致谢

我们衷心感谢Mindspore、CANN（神经网络计算架构）和Ascend AI处理器在本研究中的支持。

局限性和潜在的负面社会影响。我们的方法采用MLLM代理作为整个系统操作的核心。因此，方法的有效性依赖于所使用的MLLM的表现。目前的MLLM，如GPT-4，能够满足大多数基本需求。对于超出GPT-4能力的任务，我们的方法可能会失败。此外，图像生成或编辑的误用可能会导致负面的社会影响。

## 附录

在附录中，我们主要包含更多的定量比较，并附加更多的视觉结果，以便与现有的最先进方法进行更全面的比较。

## 附录A 更多定量实验

表6：与现有的文本到图像生成模型和组合方法在T2I-CompBench上的定量比较。此处的度量来自官方的旧版代码。

|      模型 | 属性绑定 | 对象关系 | 复杂性$\uparrow$ |
| --- | --- | --- | --- |
| 颜色$\uparrow$ | 形状$\uparrow$ | 纹理$\uparrow$ | 空间$\uparrow$ | 非空间$\uparrow$ |
| SDXL [podell2023sdxl](https://arxiv.org/html/2407.05600v2#bib.bib37) | 0.6369 | 0.5408 | 0.5637 | 0.2032 | 0.3110 | 0.4091 |
| PixArt-$\alpha$ [chen2023pixart](https://arxiv.org/html/2407.05600v2#bib.bib6) | 0.6886 | 0.5582 | 0.7044 | 0.2082 | 0.3179 | 0.4117 |
| ConPreDiff [yang2024improving](https://arxiv.org/html/2407.05600v2#bib.bib58) | 0.7019 | 0.5637 | 0.7021 | 0.2362 | 0.3195 | 0.4184 |
| DALL-E 3 [betker2023improving](https://arxiv.org/html/2407.05600v2#bib.bib1) | 0.8110 | 0.6750 | 0.8070 | - | - | - |
| CompAgent [wang2024divide](https://arxiv.org/html/2407.05600v2#bib.bib51) | 0.8488 | 0.7233 | 0.7916 | 0.4837 | 0.3212 | 0.4863 |
| RPG [yang2024mastering](https://arxiv.org/html/2407.05600v2#bib.bib59) | 0.8335 | 0.6801 | 0.8129 | 0.4547 | 0.3462 | 0.5408 |
| GenArtist（我们的） | 0.8837 | 0.8014 | 0.8771 | 0.5437 | 0.3346 | 0.5514 |
|   |  |  |  |  |  |  |

考虑到许多现有的T2I-CompBench方法报告的结果是基于官方旧版评估代码，在此我们采用相同的旧版评估方法，并在表格[6](https://arxiv.org/html/2407.05600v2#A1.T6 "Table 6 ‣ Appendix A More Quantitative Experiments ‣ GenArtist: Multimodal LLM as an Agent for Unified Image Generation and Editing")中列出结果。可以观察到，在此指标下，性能提升始终保持一致。与当前最先进的文本到图像方法DALL-E 3相比，我们的方法在属性绑定方面提高了超过7%。对于形状相关的属性，提升幅度甚至高达12.64%。此外，与同样利用MLLM辅助图像生成的RPG相比，我们的方法表现出了超过5%的提升。这是因为我们的GenArtist结合了MLLM进行逐步验证和相应的规划，从而更好地确保了图像的正确性。这一定量比较更全面地展示了我们方法的有效性。

## 附录B 更多定性实验

本节中，我们提供更多的视觉分析，以进一步说明我们的GenArtist，并将其与现有方法进行更全面的比较。

![参考标题](img/63e7a47d4306193a2fb537ad4cf68a3f.png)

图7：与现有最先进方法在图像生成任务上的定性比较。我们将GenArtist与SOTA文本到图像模型进行比较，包括SDXL [podell2023sdxl](https://arxiv.org/html/2407.05600v2#bib.bib37)，LMD+ [lian2023llm](https://arxiv.org/html/2407.05600v2#bib.bib25)，RPG [yang2024mastering](https://arxiv.org/html/2407.05600v2#bib.bib59)，PixArt-$\alpha$ [chen2023pixart](https://arxiv.org/html/2407.05600v2#bib.bib6)，Playground [playground](https://arxiv.org/html/2407.05600v2#bib.bib22)，Midjourney [Midjourney](https://arxiv.org/html/2407.05600v2#bib.bib31)，DALL-E 3 [betker2023improving](https://arxiv.org/html/2407.05600v2#bib.bib1)。

![参考标题](img/f0f624ca3386bc7f2d3e092c48b1f3f2.png)

图8：与现有最先进方法在图像编辑任务上的定性比较。我们将GenArtist与SOTA图像编辑模型进行比较，包括InstructPix2Pix [brooks2023instructpix2pix](https://arxiv.org/html/2407.05600v2#bib.bib2)，MagicBrush [zhang2024magicbrush](https://arxiv.org/html/2407.05600v2#bib.bib61)，MGIE [fu2023guiding](https://arxiv.org/html/2407.05600v2#bib.bib14)，InstructDiffusion [geng2023instructdiffusion](https://arxiv.org/html/2407.05600v2#bib.bib15)

![参考标题](img/eae2c20a5b5138bd4624c5cb5f124db7.png)

图9：GenArtist在各类任务和用户指令下的可视化结果。

![参考标题](img/e57e2e122e71492a57607cea62d6708b.png)

图10：图像生成任务的逐步过程可视化。

![参考标题](img/a67ac8f3f7dd455a830955314942d67b.png)

图11：图像编辑任务的逐步过程可视化。

图像生成的比较可视化结果。我们首先在图 [7](https://arxiv.org/html/2407.05600v2#A2.F7 "Figure 7 ‣ Appendix B More Qualitative Experiments ‣ GenArtist: Multimodal LLM as an Agent for Unified Image Generation and Editing")中展示了与现有方法的视觉比较。可以观察到，我们的GenArtist在多个方面取得了优越的结果：1) *属性绑定*。例如，在第四个示例中，每个人所穿的衣服和裤子都有严格要求。如此多的要求对于现有方法来说是一个挑战。在这种情况下，GenArtist可以不断地验证和编辑，以确保所有这些要求都能正确满足。2) *数字准确性*。在第二个示例中，给出了各个物体的详细数量要求。我们的方法可以通过加法和减法操作逐步实现正确的数量。相比之下，尽管像LMD+这样的算法能够满足数字准确性，但它们在保持图像其他方面的准确性上，例如图像氛围方面，表现不佳。3) *位置准确性*。通过位置感知工具的执行，可以保证更好的位置相关准确性。在第一个示例中，尽管DALL-E 3可以正确预测许多其他方面，但它未能准确地将书本放置在左侧的地板上，而我们的方法能够做到这一点。4) *复杂关系*，例如第五个示例中关于熊猫和竹子之间复杂关系的要求。5) *其他多样化要求*。通过集成各种工具，GenArtist有效地利用不同工具的优势来满足多样化的需求，这是单一模型所无法具备的能力。例如，第三个示例中的文本要求由我们的方法更好地处理。这些可视化结果有力地展示了我们的方法在图像生成中的有效性。

图像编辑的比较可视化结果。我们进一步在图[8](https://arxiv.org/html/2407.05600v2#A2.F8 "Figure 8 ‣ Appendix B More Qualitative Experiments ‣ GenArtist: Multimodal LLM as an Agent for Unified Image Generation and Editing")中展示了与现有图像编辑方法的比较。GenArtist在多个方面表现出色：1) *高度特定的编辑指令*。例如，在第一个例子中，仅需要修改特定的披萨，而第二个例子则要求修改花瓶的颜色和位置。现有方法通常难以满足这种特定要求。2) *基于推理的指令*。第三个例子要求模型自主判断哪个人需要被移除。由于MLLM代理具备推理能力，我们的方法能够准确做出这一判断。相比之下，即使是同样使用MLLM辅助的MGIE，也未能做出正确的修改。3) *要求过多的指令*。第四个例子要求对两只鸟进行不同的修改，而现有方法难以实现这一点。4) *多步骤指令*。第五个例子涉及包括多个操作的复杂指令。MLLM代理能够将问题分解为多个单步操作，从而简化复杂任务。5) *多样化操作*。可以看出，由于集成了不同的工具，我们的方法在各种编辑操作中表现优异，如添加、移除和属性编辑。这些比较强烈证明了我们方法在图像编辑中的有效性。

关于各种任务和用户指令的可视化结果。为了证明我们的GenArtist能够满足广泛的用户需求，我们在图[9](https://arxiv.org/html/2407.05600v2#A2.F9 "Figure 9 ‣ Appendix B More Qualitative Experiments ‣ GenArtist: Multimodal LLM as an Agent for Unified Image Generation and Editing")中提供了可视化示例。如图所示，由于集成了多种工具，我们的框架能够高效地处理这些多样化的需求。例如，它可以生成与给定图像布局或姿势相似的图像，以及与定制相关的生成。通过使用多个生成和编辑工具，我们的方法还实现了更大的控制，如在定制生成中呈现更多对象及其更复杂的关系。这些可视化示例强烈说明了使用代理进行图像生成的必要性，并证明了我们的方法有效地实现了统一图像生成和编辑的目标。

步骤过程的可视化。最后，我们展示了在图[10](https://arxiv.org/html/2407.05600v2#A2.F10 "图 10 ‣ 附录 B 更多定性实验 ‣ GenArtist：作为统一图像生成与编辑代理的多模态LLM")和图[11](https://arxiv.org/html/2407.05600v2#A2.F11 "图 11 ‣ 附录 B 更多定性实验 ‣ GenArtist：作为统一图像生成与编辑代理的多模态LLM")中的逐步可视化结果。对于图像生成，我们的方法最初利用最合适的工具生成初始图像。如果图像质量过低，或经过一些修改操作后无法纠正，则调用其他工具继续生成。此外，对于图像中不符合文本要求的部分，持续调用编辑工具进行修改，直到图像正确匹配文本。对于图像编辑，我们的方法有效地将输入问题分解，并迭代地使用不同的工具进行逐步修改，直到图像被正确编辑。这种可视化清晰地展示了从分解、规划树和逐步验证，到最终工具执行的整个过程。

## 参考文献

+   [1] James Betker, Gabriel Goh, Li Jing, Tim Brooks, Jianfeng Wang, Linjie Li, Long Ouyang, Juntang Zhuang, Joyce Lee, Yufei Guo 等人. 通过更好的标题改善图像生成。计算机科学。https://cdn.openai.com/papers/dall-e-3.pdf，2023年。

+   [2] Tim Brooks, Aleksander Holynski, 和 Alexei A Efros. Instructpix2pix：学习跟随图像编辑指令。发表于 CVPR，2023年。

+   [3] Hila Chefer, Yuval Alaluf, Yael Vinker, Lior Wolf, 和 Daniel Cohen-Or. Attend-and-excite：基于注意力的语义引导用于文本到图像的扩散模型。TOG，2023年。

+   [4] Jingye Chen, Yupan Huang, Tengchao Lv, Lei Cui, Qifeng Chen, 和 Furu Wei. Textdiffuser-2：释放语言模型在文本渲染中的强大力量。arXiv 预印本 arXiv:2311.16465，2023年。

+   [5] Jingye Chen, Yupan Huang, Tengchao Lv, Lei Cui, Qifeng Chen, 和 Furu Wei. Textdiffuser：扩散模型作为文本画家。发表于 NeurIPS，2023年。

+   [6] Junsong Chen, Jincheng Yu, Chongjian Ge, Lewei Yao, Enze Xie, Yue Wu, Zhongdao Wang, James Kwok, Ping Luo, Huchuan Lu 等人. Pixart-$\alpha$: 用于照片级真实文本到图像合成的扩散变换器快速训练。发表于 ICLR，2024年。

+   [7] Minghao Chen, Iro Laina, 和 Andrea Vedaldi. 无需训练的布局控制与跨注意力引导。发表于 WACV，2024年。

+   [8] Xi Chen, Lianghua Huang, Yu Liu, Yujun Shen, Deli Zhao, 和 Hengshuang Zhao. Anydoor：零样本物体级图像定制。发表于 CVPR，2024年。

+   [9] Guillaume Couairon, Jakob Verbeek, Holger Schwenk, 和 Matthieu Cord. Diffedit：基于扩散的语义图像编辑与掩码引导。arXiv 预印本 arXiv:2210.11427，2022年。

+   [10] Prafulla Dhariwal 和 Alexander Nichol. 扩散模型在图像合成中超越GANs。发表于 NeurIPS，2021年。

+   [11] Meta 基础 AI 研究外交团队 (FAIR)†, Anton Bakhtin, Noam Brown, Emily Dinan, Gabriele Farina, Colin Flaherty, Daniel Fried, Andrew Goff, Jonathan Gray, Hengyuan Hu, 等. 通过结合语言模型与战略推理，在外交游戏中实现人类水平的表现. 发表在 Science, 2022。

+   [12] Weixi Feng, Xuehai He, Tsu-Jui Fu, Varun Jampani, Arjun Reddy Akula, Pradyumna Narayana, Sugato Basu, Xin Eric Wang, 和 William Yang Wang. 无需训练的结构化扩散引导用于组合文本到图像的合成. 发表在 ICLR, 2023。

+   [13] Weixi Feng, Wanrong Zhu, Tsu-jui Fu, Varun Jampani, Arjun Akula, Xuehai He, Sugato Basu, Xin Eric Wang, 和 William Yang Wang. Layoutgpt：基于大语言模型的组合视觉规划与生成. 发表在 NeurIPS, 2023。

+   [14] Tsu-Jui Fu, Wenze Hu, Xianzhi Du, William Yang Wang, Yinfei Yang, 和 Zhe Gan. 通过多模态大语言模型引导基于指令的图像编辑. 发表在 ICLR, 2024。

+   [15] Zigang Geng, Binxin Yang, Tiankai Hang, Chen Li, Shuyang Gu, Ting Zhang, Jianmin Bao, Zheng Zhang, Han Hu, Dong Chen, 等. Instructdiffusion：用于视觉任务的通用建模接口. 发表在 CVPR, 2024。

+   [16] Tanmay Gupta 和 Aniruddha Kembhavi. 视觉编程：无需训练的组合视觉推理. 发表在 CVPR, 2023。

+   [17] Jonathan Ho, Ajay Jain, 和 Pieter Abbeel. 去噪扩散概率模型. 发表在 NeurIPS, 2020。

+   [18] Kaiyi Huang, Kaiyue Sun, Enze Xie, Zhenguo Li, 和 Xihui Liu. T2i-compbench：一个全面的开放世界组合文本到图像生成基准. 发表在 NeurIPS, 2023。

+   [19] Yuzhou Huang, Liangbin Xie, Xintao Wang, Ziyang Yuan, Xiaodong Cun, Yixiao Ge, Jiantao Zhou, Chao Dong, Rui Huang, Ruimao Zhang, 等. Smartedit：探索基于复杂指令的图像编辑与多模态大语言模型. 发表在 CVPR, 2024。

+   [20] Alexander Kirillov, Eric Mintun, Nikhila Ravi, Hanzi Mao, Chloe Rolland, Laura Gustafson, Tete Xiao, Spencer Whitehead, Alexander C Berg, Wan-Yen Lo, 等. Segment anything. 发表在 ICCV, 2023。

+   [21] Nupur Kumari, Bingliang Zhang, Richard Zhang, Eli Shechtman, 和 Jun-Yan Zhu. 多概念定制的文本到图像扩散. 发表在 CVPR, 2023。

+   [22] Daiqing Li, Aleks Kamko, Ali Sabet, Ehsan Akhgari, Linmiao Xu, 和 Suhail Doshi. Playground v2.

+   [23] Dongxu Li, Junnan Li, 和 Steven CH Hoi. Blip-diffusion：用于可控文本到图像生成与编辑的预训练主题表示. 发表在 NeurIPS, 2023。

+   [24] Yuheng Li, Haotian Liu, Qingyang Wu, Fangzhou Mu, Jianwei Yang, Jianfeng Gao, Chunyuan Li, 和 Yong Jae Lee. Gligen：开放集基础的文本到图像生成. 发表在 CVPR, 2023。

+   [25] Long Lian, Boyi Li, Adam Yala, 和 Trevor Darrell. Llm-grounded diffusion：利用大语言模型增强文本到图像扩散模型的提示理解. arXiv 预印本 arXiv:2305.13655, 2023。

+   [26] Haotian Liu, Chunyuan Li, Qingyang Wu, 和 Yong Jae Lee. 视觉指令微调. 发表在 NeurIPS, 2023。

+   [27] Nan Liu, Shuang Li, Yilun Du, Antonio Torralba, 和 Joshua B Tenenbaum. 使用可组合扩散模型进行组合视觉生成. 见于 ECCV, 2022.

+   [28] Shilong Liu, Zhaoyang Zeng, Tianhe Ren, Feng Li, Hao Zhang, Jie Yang, Chunyuan Li, Jianwei Yang, Hang Su, Jun Zhu, 等. Grounding Dino: 将 Dino 与基于先验训练的开放集目标检测相结合. arXiv 预印本 arXiv:2303.05499, 2023.

+   [29] Zhaoyang Liu, Yinan He, Wenhai Wang, Weiyun Wang, Yi Wang, Shoufa Chen, Qinglong Zhang, Yang Yang, Qingyun Li, Jiashuo Yu, 等. Internchat: 通过与聊天机器人互动解决以视觉为中心的任务. arXiv 预印本 arXiv:2305.05662, 2023.

+   [30] Zhiheng Liu, Yifei Zhang, Yujun Shen, Kecheng Zheng, Kai Zhu, Ruili Feng, Yu Liu, Deli Zhao, Jingren Zhou, 和 Yang Cao. Cones 2: 支持多主体的可定制图像合成. arXiv 预印本 arXiv:2305.19327, 2023.

+   [31] Midjourney. Midjourney, 2023.

+   [32] Ron Mokady, Amir Hertz, Kfir Aberman, Yael Pritch, 和 Daniel Cohen-Or. 使用引导扩散模型进行真实图像编辑的空文本反转. 见于 CVPR, 2023.

+   [33] Chong Mou, Xintao Wang, Jiechong Song, Ying Shan, 和 Jian Zhang. Dragondiffusion: 在扩散模型中实现拖动样式的操作. 见于 ICLR, 2024.

+   [34] OpenAI. GPT-4 技术报告. arXiv 预印本 arXiv:2303.08774, 2023.

+   [35] Maitreya Patel, Sangmin Jung, Chitta Baral, 和 Yezhou Yang. $\lambda$-eclipse: 利用 Clip 潜空间的多概念个性化文本到图像扩散模型. arXiv 预印本 arXiv:2402.05195, 2024.

+   [36] William Peebles 和 Saining Xie. 使用变换器的可扩展扩散模型. 见于 ICCV, 2023.

+   [37] Dustin Podell, Zion English, Kyle Lacey, Andreas Blattmann, Tim Dockhorn, Jonas Müller, Joe Penna, 和 Robin Rombach. Sdxl: 改进潜在扩散模型以实现高分辨率图像合成. arXiv 预印本 arXiv:2307.01952, 2023.

+   [38] Chen Qian, Xin Cong, Cheng Yang, Weize Chen, Yusheng Su, Juyuan Xu, Zhiyuan Liu, 和 Maosong Sun. 面向软件开发的交互式代理. arXiv 预印本 arXiv:2307.07924, 2023.

+   [39] Jie Qin, Jie Wu, Weifeng Chen, Yuxi Ren, Huixia Li, Hefeng Wu, Xuefeng Xiao, Rui Wang, 和 Shilei Wen. Diffusiongpt: 基于 LLM 的文本到图像生成系统. arXiv 预印本 arXiv:2401.10061, 2024.

+   [40] Aditya Ramesh, Prafulla Dhariwal, Alex Nichol, Casey Chu, 和 Mark Chen. 基于层次结构的文本条件图像生成与 Clip 潜变量. arXiv 预印本 arXiv:2204.06125, 2022.

+   [41] Robin Rombach, Andreas Blattmann, Dominik Lorenz, Patrick Esser, 和 Björn Ommer. 使用潜在扩散模型进行高分辨率图像合成. 见于 CVPR, 2022.

+   [42] Nataniel Ruiz, Yuanzhen Li, Varun Jampani, Yael Pritch, Michael Rubinstein, 和 Kfir Aberman. Dreambooth: 微调文本到图像扩散模型以实现基于主题的生成. 见于 CVPR, 2023.

+   [43] Chitwan Saharia, William Chan, Saurabh Saxena, Lala Li, Jay Whang, Emily L Denton, Kamyar Ghasemipour, Raphael Gontijo Lopes, Burcu Karagol Ayan, Tim Salimans 等人。《具有深度语言理解的逼真文本到图像扩散模型》。在 NeurIPS，2022。

+   [44] Yongliang Shen, Kaitao Song, Xu Tan, Dongsheng Li, Weiming Lu 和 Yueting Zhuang。《Hugginggpt：通过 ChatGPT 和 HuggingFace 解决 AI 任务》。在 NeurIPS，2023。

+   [45] Shelly Sheynin, Adam Polyak, Uriel Singer, Yuval Kirstain, Amit Zohar, Oron Ashual, Devi Parikh 和 Yaniv Taigman。《Emu 编辑：通过识别与生成任务进行精确图像编辑》。arXiv 预印本 arXiv:2311.10089，2023。

+   [46] Yujun Shi, Chuhui Xue, Jiachun Pan, Wenqing Zhang, Vincent YF Tan 和 Song Bai。《Dragdiffusion：利用扩散模型进行交互式基于点的图像编辑》。在 CVPR，2024。

+   [47] Roman Suvorov, Elizaveta Logacheva, Anton Mashikhin, Anastasia Remizova, Arsenii Ashukha, Aleksei Silvestrov, Naejin Kong, Harshith Goka, Kiwoong Park 和 Victor Lempitsky。《基于傅里叶卷积的高分辨率大面积修复》。在 WACV，2022。

+   [48] Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar 等人。《Llama: 开放且高效的基础语言模型》。arXiv 预印本 arXiv:2302.13971，2023。

+   [49] Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale 等人。《Llama 2：开放基础模型和微调对话模型》。arXiv 预印本 arXiv:2307.09288，2023。

+   [50] Andrey Voynov, Kfir Aberman 和 Daniel Cohen-Or。《草图引导的文本到图像扩散模型》。在 SIGGRAPH，2023。

+   [51] Zhenyu Wang, Enze Xie, Aoxue Li, Zhongdao Wang, Xihui Liu 和 Zhenguo Li。《分而治之：语言模型可以规划并自我修正组合式的文本到图像生成》。arXiv 预印本 arXiv:2401.15688，2024。

+   [52] Tsung-Han Wu, Long Lian, Joseph E Gonzalez, Boyi Li 和 Trevor Darrell。《自我修正的LLM控制扩散模型》。在 CVPR，2024。

+   [53] Guangxuan Xiao, Tianwei Yin, William T Freeman, Frédo Durand 和 Song Han。《Fastcomposer：无需调优的多主体图像生成与局部注意力》。arXiv 预印本 arXiv:2305.10431，2023。

+   [54] Jinheng Xie, Yuexiang Li, Yawen Huang, Haozhe Liu, Wentian Zhang, Yefeng Zheng 和 Mike Zheng Shou。《Boxdiff：通过无训练的框约束扩散进行文本到图像的合成》。在 ICCV，2023。

+   [55] Huajian Xin, Haiming Wang, Chuanyang Zheng, Lin Li, Zhengying Liu, Qingxing Cao, Yinya Huang, Jing Xiong, Han Shi, Enze Xie 等人。《Lego-prover：通过扩展库的神经定理证明》。在 ICLR，2023。

+   [56] Binxin Yang, Shuyang Gu, Bo Zhang, Ting Zhang, Xuejin Chen, Xiaoyan Sun, Dong Chen 和 Fang Wen。《通过示例绘制：基于示例的图像编辑与扩散模型》。在 CVPR，2023。

+   [57] 杨辉, 岳四夫, 和 何云忠. 用于在线决策的自动化GPT：基准测试和附加意见. arXiv 预印本 arXiv:2306.02224, 2023.

+   [58] 杨凌, 刘晶伟, 洪申达, 张志龙, 黄志麟, 蔡哲铭, 张文韬, 和 崔斌. 通过上下文预测提升基于扩散的图像合成. 载于 NeurIPS, 2023.

+   [59] 杨凌, 余兆晨, 孟晨琳, 许敏凯, Stefano Ermon, 和 崔斌. 掌握文本到图像的扩散模型：重新生成、规划与多模态大语言模型的生成. 载于 ICML, 2024.

+   [60] 杨兆, 刘嘉轩, 韩宇诚, 陈鑫, 黄泽彪, 傅斌, 和 余刚. Appagent：作为智能手机用户的多模态智能体. arXiv 预印本 arXiv:2312.13771, 2023.

+   [61] 张凯, 莫凌波, 陈文虎, 孙焕, 和 苏宇. Magicbrush：一个手动标注的用于指导性图像编辑的数据集. 载于 NeurIPS, 2023.

+   [62] 张绿敏, 饶安怡, 和 Maneesh Agrawala. 为文本到图像的扩散模型添加条件控制. 载于 ICCV, 2023.

+   [63] 张舒, 杨欣怡, 冯一豪, 秦灿, 陈家池, 于宁, 陈泽元, 王焕, Silvio Savarese, Stefano Ermon 等. Hive：利用人类反馈进行指导性视觉编辑. 载于 CVPR, 2024.

+   [64] 张宇欣, 黄妮莎, 唐凡, 黄海彬, 马崇扬, 董伟名, 和 徐昌生. 基于反演的风格迁移与扩散模型. 载于 CVPR, 2023.
