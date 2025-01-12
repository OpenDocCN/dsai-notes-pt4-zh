<!--yml

类别：未分类

日期：2025-01-11 12:51:59

-->

# MuLan：用于渐进式和互动多物体扩散的多模态LLM代理

> 来源：[https://arxiv.org/html/2402.12741/](https://arxiv.org/html/2402.12741/)

Sen Li

slien@connect.ust.hk &Ruochen Wang

ruocwang@g.ucla.edu

Cho-Jui Hsieh

chohsieh@cs.ucla.edu

&Minhao Cheng

mmc7149@psu.edu Tianyi Zhou

tianyi@umd.edu 计算机科学与工程系，香港科技大学，中国；计算机科学系，洛杉矶加利福尼亚大学，美国；计算机科学系，洛杉矶加利福尼亚大学，美国；信息科学与技术学院，宾州州立大学，美国；计算机科学系，马里兰大学，美国

###### 摘要

现有的文本到图像模型仍然难以生成多个物体的图像，特别是在处理它们的空间位置、相对大小、重叠和属性绑定时。为了高效解决这些挑战，我们开发了一种无训练的多模态大语言模型（LLM）代理（MuLan），它像一个人类画家一样，可以通过精细的规划和反馈控制逐步生成多物体图像。MuLan利用一个大语言模型（LLM）将提示语分解为一系列子任务，每个子任务仅通过稳定扩散生成一个物体，并基于之前生成的物体进行条件生成。与现有的基于LLM的方法不同，MuLan在开始时仅生成一个高层次的计划，而每个物体的精确大小和位置则由LLM和注意力引导在每个子任务中确定。此外，MuLan采用了视觉语言模型（VLM）为每个子任务中生成的图像提供反馈，并控制扩散模型重新生成图像，如果图像违背了原始提示。因此，MuLan中的每个模型在每一步只需要处理它专门化的简单子任务。这个多步骤的过程还允许用户通过文本提示在任何中间步骤监控生成过程，并做出偏好的更改，从而提升人机协作的体验。我们从不同的基准测试中收集了200个包含多物体、具有空间关系和属性绑定的提示语，以评估MuLan。结果表明，MuLan在生成多个物体方面优于基准方法，并且在与人类用户协作时展现了其创造力。代码可以在[https://github.com/measure-infinity/mulan-code](https://github.com/measure-infinity/mulan-code)找到。

## 1 引言

扩散模型 [[19](https://arxiv.org/html/2402.12741v2#bib.bib19), [9](https://arxiv.org/html/2402.12741v2#bib.bib9), [20](https://arxiv.org/html/2402.12741v2#bib.bib20)] 在生成AI任务中展现了越来越大的潜力，尤其是在使用文本提示生成多样化和高质量图像方面 [[18](https://arxiv.org/html/2402.12741v2#bib.bib18), [17](https://arxiv.org/html/2402.12741v2#bib.bib17)]。然而，当前最先进的文本到图像（T2I）模型，如Stable Diffusion [[17](https://arxiv.org/html/2402.12741v2#bib.bib17)] 和 DALL-E 3 [[2](https://arxiv.org/html/2402.12741v2#bib.bib2)]，仍然难以处理涉及多个对象的复杂提示，并且缺乏对它们空间关系、潜在遮挡、相对大小等的精确控制。如图 [2](https://arxiv.org/html/2402.12741v2#S1.F2 "图 2 ‣ 1 引言 ‣ MuLan：用于渐进式和交互式多对象扩散的多模态-LLM代理")所示，要生成“橙色南瓜在黑色门的右侧”的草图，即使是最先进的开源T2I模型Stable Diffusion XL [[16](https://arxiv.org/html/2402.12741v2#bib.bib16)]，也会生成错误的属性绑定以及多个对象的错误空间位置。

在旨在提升T2I模型在复杂提示下可控性的研究工作中，最近有一条有前景的研究方向，旨在利用大型语言模型（LLM），例如ChatGPT、GPT-4 [[1](https://arxiv.org/html/2402.12741v2#bib.bib1)]，来引导生成过程 [[12](https://arxiv.org/html/2402.12741v2#bib.bib12), [6](https://arxiv.org/html/2402.12741v2#bib.bib6)]。具体而言，LLM被提示生成给定提示的布局，即图像中每个对象的边界框，并根据需要提供详细的说明或示范。然而，由于LLM的空间推理能力有限，以及它们与扩散模型的对齐不足，LLM仍然难以直接生成多个对象的完整且精确的布局。如果没有与生成过程交互的反馈环路，布局可能出现的错误无法有效地检测和纠正。此外，布局通常作为原始提示的附加条件（例如，边界框与GLIGEN结合使用 [[11](https://arxiv.org/html/2402.12741v2#bib.bib11)]），因此扩散模型可能由于误解复杂提示，仍然生成错误的图像。

![参考标题](img/baa0d6f8a760c5cab1fb7ff272af8a4c.png)

图1：提出的无训练多模态LLM代理（MuLan）用于渐进式多物体扩散。MuLan由三个主要组件组成：（1）LLM规划；（2）带注意力引导的单物体扩散；（3）VLM反馈控制。MuLan首先将一个复杂的提示分解为一系列子提示，每个子提示生成一个物体，然后根据子提示和先前生成的物体按步骤生成每个物体，其中LLM规划物体的大致布局，注意力引导为其提供精确的掩码。VLM反馈控制使得MuLan能够通过调整（2）中的超参数，在每一步纠正错误。

为了解决以往方法的局限性和挑战，我们开发了一种无训练且可控的T2I生成范式，该范式无需示范，主要聚焦于改善现有模型的工具使用。我们的范式基于一个由多模态LLM代理（MuLan）驱动的渐进式多物体生成，该代理在每个阶段仅生成一个物体，并根据图像中已生成物体和最可能放置新物体的位置的注意力掩码进行条件生成。与之前在每个模型中添加条件并使任务更加复杂的方法不同，MuLan使用LLM作为规划者，将原始T2I任务分解为一系列更易处理的子任务。每个子任务生成一个单独的物体，这可以被扩散模型轻松处理。值得注意的是，MuLan开头使用的LLM只关注高级规划，而不是精确的边界框布局，而每个物体的确切大小和位置则在每个阶段由LLM和基于图像中生成物体的注意力引导来确定。因此，我们可以避免规划阶段的错误，并为每个物体找到一个更好的放置位置，这个位置适应生成的内容并遵循原始提示。此外，MuLan建立了一个反馈循环来监控生成过程，使用视觉-语言模型（VLM）评估每个阶段生成的图像。当生成的图像违反了提示时，VLM会调整扩散模型以重新生成图像，从而在进入下一个阶段之前纠正任何错误。此外，我们在每个阶段开发了一种策略来处理物体之间的重叠，这是以往工作中常常被忽视的问题[[12](https://arxiv.org/html/2402.12741v2#bib.bib12)]。

因此，MuLan在多物体组合的可控性方面表现更好。逐步生成过程的示意图见图[1](https://arxiv.org/html/2402.12741v2#S1.F1 "图 1 ‣ 1 介绍 ‣ MuLan：用于渐进式和交互式多物体扩散的多模态大语言模型代理")。请注意，还有一项并行工作叫做RPG [[22](https://arxiv.org/html/2402.12741v2#bib.bib22)]，它与MuLan有类似的高层次思想（即将提示分解为子任务）。然而，MuLan与RPG之间仍然存在显著的差异。MuLan在生成每个物体时，都是基于先前生成的物体进行条件生成，而RPG则是独立地生成所有物体。MuLan不需要任何人工设计的示范进行上下文学习。此外，如第[4.1](https://arxiv.org/html/2402.12741v2#S4.SS1 "4.1 主要结果与分析 ‣ 4 实验 ‣ MuLan：用于渐进式和交互式多物体扩散的多模态大语言模型代理")节所示，MuLan可以直接应用于生成过程中的人机交互，这大大提高了生成的灵活性和有效性。为了评估MuLan，我们从不同的基准中精心挑选了复杂且具有挑战性的提示数据集。为了与现有方法进行比较，我们通过向GPT-4V [[15](https://arxiv.org/html/2402.12741v2#bib.bib15)] 提出一些基于输入文本的问题，全面评估生成图像与提示在三个方面的对齐情况。我们还进行了生成图像的人工评估。大量实验结果表明，MuLan能够更好地控制生成过程，并生成与提示对齐更好的高质量图像，优于基线方法。不同方法生成的示例图像见图[2](https://arxiv.org/html/2402.12741v2#S1.F2 "图 2 ‣ 1 介绍 ‣ MuLan：用于渐进式和交互式多物体扩散的多模态大语言模型代理")。我们的主要贡献总结如下：

![请参见说明](img/6e554b578fe3b76f42f0f6f3d63067b4.png)

图 2：MuLan生成图像的示例，与原始的SD-v1.4 [[17](https://arxiv.org/html/2402.12741v2#bib.bib17)]、原始的SDXL [[16](https://arxiv.org/html/2402.12741v2#bib.bib16)]、结构扩散 [[5](https://arxiv.org/html/2402.12741v2#bib.bib5)]、Promptist [[7](https://arxiv.org/html/2402.12741v2#bib.bib7)] 和PixArt-$\alpha$ [[3](https://arxiv.org/html/2402.12741v2#bib.bib3)]进行比较。

+   •

    我们提出了一种新颖的无训练范式用于文本到图像的生成，并且提出了一种多模态大语言模型（Multimodal-LLM）代理。该方法在生成包含多个物体、具有指定空间关系和属性绑定的复杂提示时，能够实现更好的控制。

+   •

    我们提出了一种有效的策略来处理T2I生成中的多物体遮挡问题，旨在提高图像质量，使其更加逼真。

+   •

    我们策划了一个数据集，用于评估T2I任务中多对象组合的空间关系和属性绑定。定量结果和人工评估结果表明，我们的方法相比不同的可控生成方法和通用T2I生成方法能够获得更好的效果。

+   •

    我们展示了所提出的框架可以应用于生成过程中的人机互动。这使得用户能够在生成过程中有效监控并改变/调整生成过程，而不需要等到所有生成完成之后。

## 2 相关工作

#### 扩散模型

作为一类新的生成模型，扩散模型由于其强大的创作能力，越来越受到关注。文本到图像生成（Text-to-Image Generation），旨在根据给定的文本提示生成高质量的图像，是最受欢迎的应用之一[[14](https://arxiv.org/html/2402.12741v2#bib.bib14), [18](https://arxiv.org/html/2402.12741v2#bib.bib18), [17](https://arxiv.org/html/2402.12741v2#bib.bib17), [2](https://arxiv.org/html/2402.12741v2#bib.bib2)]。在各种强大的扩散模型中，潜在扩散模型[[17](https://arxiv.org/html/2402.12741v2#bib.bib17)]表现出了惊人的能力，并且由于其高效性和优越的性能，已经在实践中得到了广泛应用，它也是当前SOTA稳定扩散模型的核心。不同于直接在像素空间进行扩散和去噪过程的典型扩散模型，潜在扩散模型在编码的潜在空间[[17](https://arxiv.org/html/2402.12741v2#bib.bib17)]中完成整个过程，这可以大大减少训练和推理时间。最近，借助显著扩展的模型能力，Stable Diffusion XL展示了接近商业应用标准的性能[[16](https://arxiv.org/html/2402.12741v2#bib.bib16)]。有关扩散模型过程的详细背景，请参见附录[C](https://arxiv.org/html/2402.12741v2#A3 "Appendix C Background on (Latent) Diffusion Models ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")。

#### 扩散模型中的组合生成

尽管 Stable Diffusion 模型在 T2I 生成任务中展现了前所未有的表现，但它在处理多对象的文本提示时仍然存在困难，特别是当提示中涉及多个空间关系和属性绑定时。为了实现更可控且精确的图像构图，许多组合生成方法应运而生。StructureDiffusion [[5](https://arxiv.org/html/2402.12741v2#bib.bib5)] 提出了一个无训练方法，用于解析输入的提示，并将其与跨注意力机制结合，从而更好地控制属性绑定和组合生成。另一方面，Promptist [[7](https://arxiv.org/html/2402.12741v2#bib.bib7)] 旨在训练一个语言模型，优化输入提示，使其更易理解并促进扩散模型的应用。一些工作利用大型语言模型，通过上下文学习直接生成输入提示的完整布局，然后基于该布局生成图像 [[12](https://arxiv.org/html/2402.12741v2#bib.bib12), [6](https://arxiv.org/html/2402.12741v2#bib.bib6)]。尽管之前的工作都处理了整个输入提示，我们提出将原本复杂的任务分解为若干较易处理的子任务。我们采用一个无训练的多模态-LLM 代理，逐步生成对象并进行反馈控制，以便更好地控制整个生成过程。最近，一项并行工作 RPG [[22](https://arxiv.org/html/2402.12741v2#bib.bib22)] 也提出利用 LLM 代理将提示分解为不同的子任务。然而，MuLan 是逐步生成每个对象，并在每一步之后纠正错误，而不是将所有子任务独立处理，也不需要精心设计的上下文学习演示。我们将在附录 [B](https://arxiv.org/html/2402.12741v2#A2 "附录 B MuLan 与并行工作 RPG 之间的差异 ‣ MuLan：用于渐进式和交互式多对象扩散的多模态 LLM 代理") 中对 RPG [[22](https://arxiv.org/html/2402.12741v2#bib.bib22)] 进行更为详细的讨论。

## 3 多模态-LLM 代理（MuLan）

现有的扩散模型通常在处理复杂的提示时表现不佳，但能够处理简单的提示。最近的方法通过训练模型或应用上下文学习，给出类似的示例，以提前生成提示的详细布局，然后扩散模型可以基于简单的提示单独生成布局中的每个部分。与其一次性生成所有对象或并行生成，MuLan 从许多人类画家的灵感中汲取，画家通常先做出一个高层次的计划，按计划逐一绘制对象，并在每一步之后如有需要进行错误修正。因此，能够自然地考虑对象之间的约束关系。

### 3.1 概述

MuLan首先通过战略性地规划和分解复杂的输入提示，将其转化为一系列可管理的子提示，每个子提示专注于生成一个单独的对象。然后，MuLan采用一种渐进式策略，在每个阶段基于之前生成的对象使用扩散模型生成一个对象。同时，VLM提供有见地的反馈，并自适应地调整生成过程，以确保完成每个子任务的精确性。与之前的方法相比，MuLan完全不需要训练，并且不需要任何上下文示例。如图[1](https://arxiv.org/html/2402.12741v2#S1.F1 "图1 ‣ 1 引言 ‣ MuLan: 用于渐进式和互动多对象扩散的多模态LLM代理")所示，MuLan由三个组件组成：

+   •

    通过LLM规划进行提示分解，生成一系列子提示，每个子提示专注于生成提示中的一个对象。

+   •

    基于LLM规划和注意力引导的条件单对象扩散，利用稳定的扩散模型根据上一步的图像生成新对象。尽管LLM规划中的一个子提示提供了文本引导，对象的大小和位置通过注意力掩码来控制，确保对象正确定位并生成。

+   •

    通过与VLM互动进行反馈控制，VLM检查每个阶段生成的图像，并在图像违反原始提示时调整超参数和注意力引导，以重新生成图像。

### 3.2 通过LLM规划进行提示分解

给定一个复杂的提示p，MuLan首先使用LLM自动将p分解为$N$个面向对象的子提示$\texttt{p}_{1:N}$。在分解过程中，MuLan特别要求LLM生成一系列对象，这些对象将按默认顺序从左到右、从下到上在图像中创建。LLM可以轻松完成此任务，利用其先验知识将p中的所有对象填充到预定义顺序的空列表中，而无需依赖上下文学习，也无需手动设计示例。设$\texttt{objs}=\{\texttt{obj}_{1},\cdots,\texttt{obj}_{n},\cdots,\texttt{obj}_{% N}\}$为从p中提取的LLM规划的$N$个对象。对于第一个对象，子提示是简单的$\texttt{p}_{1}=$“$\{\texttt{obj}_{1}\}$”。对于对象-$n$（其中$n>1$），子任务是基于前面生成的对象生成对象-$n$，文本子提示定义为$\texttt{p}_{n}=$“$\{\texttt{obj}_{n}\}\,\,\texttt{and}\,\,\{\texttt{obj}_{n-1}\}$”。MuLan在生成任何图像之前，首先通过LLM进行上述全局规划。LLM规划的详细提示和模板可以在附录[E](https://arxiv.org/html/2402.12741v2#A5 "附录E LLM全局规划的详细提示模板 ‣ MuLan: 用于渐进式和互动多对象扩散的多模态LLM代理")中找到。

在生成第[3.3](https://arxiv.org/html/2402.12741v2#S3.SS3 "3.3 Conditional Single-Object Diffusion with LLM Planning and Attention Guidance ‣ 3 Multimodal-LLM Agent (MuLan) ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")节中的每个物体时，我们将再次使用LLM作为物体位置和大小的局部规划器，即通过在图像中生成一个掩膜，并协调其与前一个物体的重叠。然后，使用扩散模型在掩膜的注意力引导下生成该物体。具体内容将在第[3.3](https://arxiv.org/html/2402.12741v2#S3.SS3 "3.3 Conditional Single-Object Diffusion with LLM Planning and Attention Guidance ‣ 3 Multimodal-LLM Agent (MuLan) ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")节中进一步阐述。

### 3.3 基于LLM规划和注意力引导的条件单一物体扩散

在第$n$阶段，扩散模型只关注根据子提示$\texttt{p}_{n}$生成$\texttt{obj}_{n}$，确保$\texttt{obj}_{n}$能够正确定位和生成。为此，MuLan利用LLM规划$\texttt{obj}_{n}$的相对位置和大小，并为$\texttt{obj}_{n}$分配一个粗略的掩膜（即边界框）$\bm{M}_{n}$。然后，在生成$\texttt{obj}_{n}$的过程中，应用跨注意力引导，确保$\texttt{obj}_{n}$正确地定位在$\bm{M}_{n}$内。该流程在图[3](https://arxiv.org/html/2402.12741v2#S3.F3 "Figure 3 ‣ 3.3 Conditional Single-Object Diffusion with LLM Planning and Attention Guidance ‣ 3 Multimodal-LLM Agent (MuLan) ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")中给出，完整的过程在附录[D](https://arxiv.org/html/2402.12741v2#A4 "Appendix D Algorithm procedure of single-object diffusion in MuLan ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")的算法[1](https://arxiv.org/html/2402.12741v2#alg1 "Algorithm 1 ‣ Appendix D Algorithm procedure of single-object diffusion in MuLan ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")中列出。我们将在以下部分逐步介绍。

![参考标题](img/47781204c2f194b2b33e44203f8ca02e.png)

图3：使用LLM规划和注意力引导进行的单一物体扩散（详细过程请参见附录[D](https://arxiv.org/html/2402.12741v2#A4 "Appendix D Algorithm procedure of single-object diffusion in MuLan ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")中的算法[1](https://arxiv.org/html/2402.12741v2#alg1 "Algorithm 1 ‣ Appendix D Algorithm procedure of single-object diffusion in MuLan ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")）。

#### LLM对$\texttt{obj}_{n}$的粗略掩膜规划。

在阶段-$n$，MuLan首先分配一个粗略掩码作为边界框${\bm{M}}_{n}\triangleq(x_{n},y_{n},w_{n},h_{n})$（左上角的x/y坐标、宽度和高度）来引导生成图像中的$\texttt{obj}_{n}$。如图[3](https://arxiv.org/html/2402.12741v2#S3.F3 "图 3 ‣ 3.3 条件单物体扩散与LLM规划和注意力引导 ‣ 3 多模态-LLM代理（MuLan） ‣ MuLan: 逐步和交互式多物体扩散的多模态-LLM代理")所示，${\bm{M}}_{n}$可以通过$\texttt{obj}_{n}$的相对位置$\texttt{opt}_{n}\in\texttt{Opts=\{left,right,top,bottom\}}$、与$\texttt{obj}_{n}$处于相同位置/区域的物体总数$\texttt{Num}_{n}$以及图像中当前可用空间来推导。$\texttt{Num}_{n}$和当前可用空间，综合起来决定了$\texttt{obj}_{n}$的大小。MuLan利用LLM规划器推理$\texttt{opt}_{n}$和$\texttt{Num}_{n}$，给定子提示$\texttt{p}_{n}$¹¹1详细提示模板可以参见附录[F](https://arxiv.org/html/2402.12741v2#A6 "附录 F LLM本地规划的详细提示模板 ‣ MuLan: 逐步和交互式多物体扩散的多模态-LLM代理")，而当前可用空间可以通过精确掩码$\bm{\tilde{M}}_{n-1}$来确定，该掩码描述了之前生成的$\texttt{obj}_{n-1}$的确切位置，并且可以轻松地从交叉注意力图中提取。值得注意的是，由于对于第一个物体没有之前生成的物体，$\texttt{obj}_{1}$的可用空间就是整个图像。有关$\bm{M}_{n}$的详细计算，请参见附录[G](https://arxiv.org/html/2402.12741v2#A7 "附录 G 粗略掩码计算的详细信息 ‣ MuLan: 逐步和交互式多物体扩散的多模态-LLM代理")。

一旦确定了$\bm{M}_{n}$，在生成$\texttt{obj}_{n}$的过程中会利用交叉注意力引导，以确保$\texttt{obj}_{n}$能够正确地生成在$\bm{M}_{n}$内，具体内容将在下文中详细阐述。

#### 使用注意力引导的单一物体生成。

给定$\texttt{obj}_{n}$的粗略掩码$\bm{M}_{n}$，接下来需要确保生成的$\texttt{obj}_{n}$能够正确地定位在$\bm{M}_{n}$内。扩散模型中实现这一目标的自然且直观的方法是引导生成$\texttt{obj}_{n}$的交叉注意力图，从而建立文本提示与生成物体位置之间的关联。

为此，MuLan 在 $\bm{M}_{n}$ 的指导下，操控 $\texttt{obj}_{n}$ 的交叉注意力图，通过逆向引导方法[[4](https://arxiv.org/html/2402.12741v2#bib.bib4)]，最大化 $\bm{M}_{n}$ 内部的相关性。具体来说，令 $\bm{A}$ 为交叉注意力图，$\bm{A}_{m,k}$ 表示空间位置 $m$ 与描述 $\texttt{obj}_{n}$ 的 token-$k$ 之间的相关性。$\bm{A}_{m,k}$ 的值越大，表示 $\texttt{obj}_{n}$ 更可能位于空间位置 $m$。目标是最大化 $\bm{M}_{n}$ 内部的相关性 $\bm{A}_{m,k}$，同时最小化 $\bm{M}_{n}$ 外部的相关性。因此，采用以下能量函数：

|  | $\displaystyle\scriptstyle E(\bm{A},\bm{M}_{n},k)=\left(1-\frac{\sum_{m\in\bm{M}_{n}}\bm{A}_{m,k}}{\sum_{m}\bm{A}_{m,k}}\right)^{2},$ |  | (1) |
| --- | --- | --- | --- |

其中 $\sum_{m\in\bm{M}_{n}}$ 表示对 $\bm{M}_{n}$ 中包含的空间位置求和，而 $\sum_{m}$ 表示对注意力图中所有空间位置求和。在早期生成过程的每一步 $t$ 中，MuLan 通过更新对象 $\texttt{obj}_{n}$ 的输入潜在变量 $\bm{z}_{n,t}$ 来应用梯度下降，最小化能量。这样， $\texttt{obj}_{n}$ 对应的交叉注意力图将在 $\bm{M}_{n}$ 内部达到最大的相关性，这意味着 $\texttt{obj}_{n}$ 可以正确地定位在粗略的掩模内。

另一方面，为了在生成 $\texttt{obj}_{n}$ 时考虑前一对象及其约束，我们进一步结合了 $\texttt{obj}_{n}$ 和 $\texttt{obj}_{n-1}$ 的潜在变量。具体来说，在反向过程的第 $t$ 步（$t$ 从 $T$ 变化到 $0$）后，我们通过以下方式更新潜在变量 $\bm{z}_{n,(t-1)}$：

|  | $\displaystyle\bm{z}_{n,(t-1)}=\bm{M}^{\prime}_{n}\odot\bm{z}_{n,(t-1)}+(1-\bm{M}^{\prime}_{n})\odot\bm{z}_{(n-1),(t-1)},$ |  | (2) |
| --- | --- | --- | --- |

其中 $\odot$ 计算逐元素积，而 $[\bm{M}^{\prime}_{n}]_{uv}=\mathbbm{1}_{u\in[x_{n},x_{n}+w_{n}],v\in[y_{n},y_{n}+h_{n}]}$ 是一个 0-1 指示符，表示坐标 $(u,v)$ 是否位于 $\bm{M}_{n}$ 的边界框内。

MuLan 将上述单对象扩散依次应用于每个对象，从 $\texttt{obj}_{1}$ 到 $\texttt{obj}_{N}$，这一过程由 LLM 在一开始的规划中确定。生成 $\texttt{obj}_{n}$ 的过程在算法[1](https://arxiv.org/html/2402.12741v2#alg1 "算法 1 ‣ 附录 D 中 MuLan 的单对象扩散算法过程 ‣ MuLan: 用于渐进式和交互式多对象扩散的多模态 LLM 代理") 中详细描述。

#### 对象重叠。

对象之间的重叠是文本到图像扩散模型中的一个关键挑战。然而，先前的方法并未关注这一问题[[12](https://arxiv.org/html/2402.12741v2#bib.bib12), [6](https://arxiv.org/html/2402.12741v2#bib.bib6)]。相反，我们提出了一种有效的策略，可以合并到上述过程当中。具体而言，在生成对象$\texttt{obj}_{n}$时，我们提示 LLM 判断是否存在 $\texttt{obj}_{n}$ 和 $\texttt{obj}_{n-1}$ 之间的重叠。如果存在重叠，我们首先计算三个候选的粗略掩码$\{\bm{M}_{n,i}\}_{i=1}^{3}$，它们与 $\texttt{obj}_{n-1}$ 和 $\texttt{obj}_{n}$ 之间的三个重叠比率$\{r_{i}\}_{i=1}^{3}=\{10\%,30\%,50\%\}$相关。

给定这三个掩码 $\bm{M}_{n,i}$，MuLan 使用算法[1](https://arxiv.org/html/2402.12741v2#alg1 "Algorithm 1 ‣ Appendix D Algorithm procedure of single-object diffusion in MuLan ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")生成三个候选图像。然后，计算生成的图像与输入提示 $\texttt{p}_{n}$ 之间的 CLIP 分数[[8](https://arxiv.org/html/2402.12741v2#bib.bib8)]，并选择具有最大 CLIP 分数的图像作为生成的 $\texttt{obj}_{n}$ 图像。图8中给出了一个示意图，附录[H](https://arxiv.org/html/2402.12741v2#A8 "Appendix H More details on the overlapping processing ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")中提供了更多候选掩码的详细信息。

### 3.4 在生成过程中与 VLM 和用户的交互

为了纠正顺序生成过程中可能出现的错误，MuLan 通过与视觉-语言模型（VLM）交互，建立了一个自适应的反馈回路控制。在每个生成阶段后，MuLan 查询 VLM 检查生成的对象及其与输入提示的一致性。如果它们不一致，MuLan 将调整当前阶段的反向指导，以重新生成该对象。这样的闭环控制涉及到 LLM、扩散和 VLM，并显著地自动化了针对复杂提示的 T2I 生成，从而在实践中实现了更准确的生成。

![参见标题说明](img/58afe3b1e45af0bc7e4580567b58293b.png)

图 4：展示了生成过程中不同的人类-智能体交互情况的示意图。中间的分支（由蓝色箭头连接）展示了没有人类-智能体交互的原始生成过程。顶部和底部的分支展示了在生成过程中为了各种调整而进行的人类-智能体复杂交互，包括物体调整、属性调整和空间关系调整，展示了MuLan在人类-智能体交互方面的灵活性和有效性。

此外，这一多步骤过程自然允许在生成过程中进行人类-智能体交互/协作。用户可以实时监控生成过程。通过这种方式，交互使得用户能够通过向MuLan提供调整提示，在任何中间步骤轻松有效地对生成的图像进行偏好的更改和调整，例如属性调整、物体调整和空间关系调整。通过这些调整提示，MuLan将利用LLM相应地修改原始提示，并将生成过程改变为偏好的生成方式。图[4](https://arxiv.org/html/2402.12741v2#S3.F4 "图 4 ‣ 3.4 人类-智能体交互与VLM和用户协作 ‣ 3 多模态-LLM智能体 (MuLan) ‣ MuLan：用于渐进式和交互式多物体扩散的多模态-LLM智能体")展示了生成过程中不同变化或调整的示意图，表明MuLan能够在交互中实现简单和复杂的调整。相比之下，对于其他现有的生成和编辑方法，用户必须等待整个生成过程完成。因此，所提出的框架在人类-智能体交互和协作方面更加用户友好和灵活。

## 4 实验

#### 数据集

为了评估我们的框架，我们从不同的基准构建了一个提示数据集。具体来说，由于我们的重点是实现对包含多个物体、具有空间关系和属性绑定的复杂提示的更好生成，我们首先从T2I-CompBench收集所有复杂的空间提示[[10](https://arxiv.org/html/2402.12741v2#bib.bib10)]。为了使实验更加全面，我们让ChatGPT生成大约400个包含不同物体、空间关系和属性绑定的提示，使得提示集包含大约600个提示。为了进一步评估我们框架在处理极其复杂和困难的提示上的能力，我们手动添加了SDXL无法生成的提示，形成一个包含200个提示的困难提示数据集。与T2I-CompBench中的复杂空间提示[[10](https://arxiv.org/html/2402.12741v2#bib.bib10)]类似，我们整理的数据集中每个提示通常包含两个物体，并具有各种空间关系，每个物体包含从{颜色、形状、纹理}中随机选择的属性绑定。

#### 模型与基准

作为一个无训练框架，MuLan 可以与任何现有的扩散模型结合使用。我们在我们的框架下评估了两个稳定扩散模型，Stable Diffusion v1.4 [[17](https://arxiv.org/html/2402.12741v2#bib.bib17)] 和 SOTA Stable Diffusion XL [[16](https://arxiv.org/html/2402.12741v2#bib.bib16)]。为了验证 MuLan 的优越性，我们将其与之前的可控生成方法和通用 T2I 生成方法进行了比较。具体来说，我们评估了 Structure Diffusion [[5](https://arxiv.org/html/2402.12741v2#bib.bib5)]、Promptist [[7](https://arxiv.org/html/2402.12741v2#bib.bib7)]、原始的 Stable Diffusion v1.4、原始的 SDXL，以及最近的 SOTA 扩散模型 PixArt-$\alpha$ [[3](https://arxiv.org/html/2402.12741v2#bib.bib3)]。

#### 实现细节

MuLan 使用 GPT-4 [[1](https://arxiv.org/html/2402.12741v2#bib.bib1)] 作为 LLM 规划器，并使用 LLaVA-1.5 [[13](https://arxiv.org/html/2402.12741v2#bib.bib13)] 作为 VLM 检查器提供反馈。我们还进行了消融研究，以展示 VLM 提供的反馈控制的重要性以及不同 VLM 的效果。此外，我们发现，在注意力引导过程中使用的注意力模块至关重要，这些模块可以分为接近输入的模块、接近中间的模块和接近输出的模块。我们在主要实验中使用了接近中间的模块，并展示了不同模块的消融结果。所有实验均在单个 NVIDIA RTX A6000 GPU 上进行。

#### 评估

由于提示数据集包含了复杂构成的文本，我们设计了一份问卷，全面调查生成图像与相应输入文本之间的对齐情况。问卷包括三个方面——对象完整性、属性绑定的正确性和空间关系的正确性。每个问题仅设置了两个选项（是或否），没有任何歧义。有关评估的详细问题和示例，请参见附录 [I](https://arxiv.org/html/2402.12741v2#A9 "附录 I 更多关于评估问卷的细节 ‣ MuLan: 用于渐进式和互动式多对象扩散的多模态 LLM 代理")。对于每个评估方面，我们计算回答“是”的百分比。给定生成的图像后，我们通过问卷评估图像的质量，问卷由最新的多模态大型语言模型（GPT-4V [[15](https://arxiv.org/html/2402.12741v2#bib.bib15)]）和人工评估者共同完成。

### 4.1 主要结果与分析

#### GPT 评估结果

给定生成的图像，我们通过问卷提示GPT-4V回答关于图像的问题，其中每个问题仅关注三大方面之一。不同方法和不同基础模型的结果如表[1](https://arxiv.org/html/2402.12741v2#S4.T1 "表 1 ‣ GPT评估结果 ‣ 4.1 主要结果与分析 ‣ 4 实验 ‣ MuLan: 多模态LLM代理用于渐进式和交互式多物体扩散")所示。结果表明，我们的框架在与不同可控生成方法和T2I生成方法的比较中，能够实现最佳表现。特别是在两个“更难”的方面——属性绑定和空间关系，MuLan能够大幅度超越其他方法。更多定性结果可以在图[5](https://arxiv.org/html/2402.12741v2#S4.F5 "图 5 ‣ GPT评估结果 ‣ 4.1 主要结果与分析 ‣ 4 实验 ‣ MuLan: 多模态LLM代理用于渐进式和交互式多物体扩散")和附录[J](https://arxiv.org/html/2402.12741v2#A10 "附录 J 更多定性结果 ‣ MuLan: 多模态LLM代理用于渐进式和交互式多物体扩散")中找到。

表 1：GPT-4V评估/人类评估，通过不同方法生成复杂提示下的图像。

| 方法 | 物体完整性 | 属性绑定 | 空间关系 | 总体 |
| --- | --- | --- | --- | --- |
| 结构扩散 [[5](https://arxiv.org/html/2402.12741v2#bib.bib5)] | 88.97%/87.37% | 54.62%/62.63% | 34.36%/24.24% | 64.31%/64.85% |
| Promptist-SD v1.4 [[7](https://arxiv.org/html/2402.12741v2#bib.bib7)] | 80.36%/70.71% | 49.23%/52.02% | 24.49%/13.13% | 56.73%/51.72% |
| Promptist-SDXL [[7](https://arxiv.org/html/2402.12741v2#bib.bib7)] | 94.36%/93.94% | 70.00%/78.28% | 35.89%/33.33% | 72.92%/75.56% |
| SD v1.4 [[17](https://arxiv.org/html/2402.12741v2#bib.bib17)] | 90.31%/74.49% | 57.14%/51.02% | 37.24%/32.65% | 66.43%/56.73% |
| SDXL [[16](https://arxiv.org/html/2402.12741v2#bib.bib16)] | 94.64%/78.57% | 66.07%/53.06% | 41.14%/24.49% | 72.34%/57.55% |
| PixArt-$\alpha$ [[3](https://arxiv.org/html/2402.12741v2#bib.bib3)] | 92.09%/76.53% | 66.58%/61.22% | 34.69%/32.65% | 70.41%/61.63% |
| MuLan-SD v1.4（我们的） | 93.11%/86.36% | 74.23%/74.24% | 51.53%/54.54% | 77.24%/75.15% |
| MuLan-SDXL（我们的） | 96.17%/90.40% | 75.00%/79.29% | 39.29%/49.49% | 76.33%/77.78% |

![参见说明](img/0bc06684c6b13f7a818319de4e958fca.png)

图 5：不同方法在复杂提示下生成的图像的更多定性示例。

#### 人类评估结果

为了进一步准确评估生成图像与人类偏好的一致性，我们还通过随机抽取100个提示从提示数据集中进行人工评估。同样，我们要求人工评估人员完成GPT评估中使用的问卷。结果如表[1](https://arxiv.org/html/2402.12741v2#S4.T1 "Table 1 ‣ Results on GPT Evaluation ‣ 4.1 Main Results and Analysis ‣ 4 Experiments ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")所示，结果表明我们的方法仍然可以达到最佳表现，并且与GPT-4V评估结果一致。

#### 人机交互结果

为了展示如果用户希望在生成过程中修改输入提示或编辑生成的图像时，MuLan仍然非常有效，即在人机交互的场景下，我们使用ChatGPT模拟用户，针对随机抽取的50个提示生成各种调整提示与MuLan进行交互。SD v1.4 [[17](https://arxiv.org/html/2402.12741v2#bib.bib17)] 被用作基础模型。生成的调整提示聚焦于多个方面，即属性调整、物体调整和空间关系调整。我们使用GPT-4V [[15](https://arxiv.org/html/2402.12741v2#bib.bib15)] 定量评估MuLan在给定最终生成图像和最终文本提示下的表现，如表[2](https://arxiv.org/html/2402.12741v2#S4.T2 "Table 2 ‣ Results on Human-Agent Interaction ‣ 4.1 Main Results and Analysis ‣ 4 Experiments ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")所示。结果表明，MuLan即便在生成过程中进行各种调整/变化时，仍能保持较高的准确性。

表2：GPT-4V对调整/变化后的最终生成图像和最终提示的评估。结果表明，MuLan在生成过程中进行各种提示调整时，仍然非常有效。

|  | 物体 | 属性 | 空间 | 总体 |
| --- | --- | --- | --- | --- |
| MuLan-SD v1.4 | 95.92% | 72.45% | 28.57% | 73.06% |

### 4.2 消融研究

在本节中，我们展示了在扩散生成过程中，注意力模块的作用及在所提出框架中VLM反馈控制的重要性的消融实验结果。所有消融研究中的实验都随机从提示数据集中抽取了50个提示。

#### 注意力模块消融

正如我们在第[4](https://arxiv.org/html/2402.12741v2#S4 "4 实验 ‣ MuLan：用于渐进和互动的多对象扩散的多模态大语言模型代理")节开始时提到的，注意力模块用于反向引导有三种选择，即近输入模块、近中间模块和近输出模块。我们通过经验发现，近中间模块在生成的控制和性能上表现最佳，通常包含最丰富的语义。因此，在这里我们展示了不同注意力模块选择的消融结果。我们使用 SD-v1.4 作为基础模型，并通过 GPT-4V 评估了在我们框架下不同注意力模块的性能。结果显示在表[3](https://arxiv.org/html/2402.12741v2#S4.T3 "表 3 ‣ 注意力模块的消融 ‣ 4.2 消融研究 ‣ 4 实验 ‣ MuLan：用于渐进和互动的多对象扩散的多模态大语言模型代理")中，表明采用近中间模块的扩散生成相比其他两种选择可以取得更好的结果。

表 3：基于 SD-v1.4 作为基础模型的注意力模块消融研究。“对象”、“属性”和“空间”分别表示对象完整性、属性绑定和空间关系。通过 GPT-4V 评估的结果[[15](https://arxiv.org/html/2402.12741v2#bib.bib15)]表明，近中间注意力模块在注意力引导方面表现最佳。

| 引导 | 对象 | 属性 | 空间 | 综合 |
| --- | --- | --- | --- | --- |
| near-input | 83.67% | 55.10% | 14.29% | 58.37% |
| near-middle | 97.96% | 80.61% | 30.61% | 77.55% |
| near-output | 72.45% | 45.92% | 22.45% | 51.84% |

#### VLM 反馈控制的消融研究

VLM 反馈控制是 MuLan 中的一个关键组件，用于提供反馈并调整生成过程，以确保每个阶段的正确生成。这里，我们通过去除反馈控制来展示反馈的重要性。如表[4](https://arxiv.org/html/2402.12741v2#S4.T4 "Table 4 ‣ Ablation on the VLM feedback control ‣ 4.2 Ablation study ‣ 4 Experiments ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")所示，去除 VLM 后，结果会大幅下降。这是因为缺乏每个生成阶段的保障或自适应调整，验证了 VLM 提供的反馈控制对于处理复杂提示是至关重要的。此外，我们还测试了 MuLan 与不同 VLM 的兼容性。如表[5](https://arxiv.org/html/2402.12741v2#S4.T5 "Table 5 ‣ Ablation on the VLM feedback control ‣ 4.2 Ablation study ‣ 4 Experiments ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")所示，我们比较了 MuLan 使用不同 VLM（包括 LLaVA-1.5 [[13](https://arxiv.org/html/2402.12741v2#bib.bib13)]，GPT-4V [[15](https://arxiv.org/html/2402.12741v2#bib.bib15)]，Gemini-Pro [[21](https://arxiv.org/html/2402.12741v2#bib.bib21)]）的表现。结果表明，MuLan 在不同的 VLM 选择下仍能保持良好的性能，并实现良好的兼容性。

表 4：消融研究，比较使用与不使用 VLM 反馈控制的 MuLan，使用 SD-v1.4 作为扩散模型，GPT-4 作为评估中的判定标准。结果表明，反馈控制可以显著提高性能。

| MuLan | 对象 | 属性 | 空间 | 总体 |
| --- | --- | --- | --- | --- |
| 有反馈 | 97.96% | 80.61% | 30.61% | 77.55% |
| 无反馈 | 81.63% | 59.18% | 18.37% | 60.00% |

表 5：MuLan 中使用的 VLM 的消融研究，使用 SD-v1.4 作为扩散模型，GPT-4 作为评估中的判定标准。结果表明，VLM 的选择对整体性能的影响并不大。

| MuLan 中的 VLM | 对象 | 属性 | 空间 | 总体 |
| --- | --- | --- | --- | --- |
| LLaVA-1.5 [[13](https://arxiv.org/html/2402.12741v2#bib.bib13)] | 97.96% | 80.61% | 30.61% | 77.55% |
| GPT-4V [[15](https://arxiv.org/html/2402.12741v2#bib.bib15)] | 95.92% | 80.61% | 28.57% | 76.33% |
| Gemini-Pro [[21](https://arxiv.org/html/2402.12741v2#bib.bib21)] | 95.92% | 83.67% | 38.78% | 79.59% |

## 5 结论与局限性

本文提出了一种无需训练的多模态LLM代理（MuLan），通过闭环反馈控制逐步生成复杂输入提示中包含的对象，从而实现对整个生成过程的更好、更精确的控制。通过先将复杂的提示分解为更容易处理的子任务，我们的方法依次处理每个对象，并以之前的对象为条件进行生成。VLM检查器进一步通过反馈控制和自适应调整，为每个阶段的正确生成提供了保证。此外，将其应用于人机交互进一步增强了MuLan的意义，使生成更加灵活高效，能够与用户的偏好对齐。大量实验表明，MuLan在多方面优于之前的方法，展示了其作为一种可控扩散生成新范式的潜力。然而，仍然存在一些限制，需在未来的工作中进一步解决。由于整个生成过程包含多个阶段，具体耗时取决于对象的数量，因此可能比单阶段生成方法更耗时。另一方面，LLM规划器可能错误解析输入提示，从而导致错误的分解。这可以通过让LLM先重写输入提示来解决，以便后续处理更加顺利。

## 参考文献

+   [1] Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat 等人。Gpt-4技术报告。arXiv预印本 arXiv:2303.08774, 2023。

+   [2] James Betker, Gabriel Goh, Li Jing, Tim Brooks, Jianfeng Wang, Linjie Li, Long Ouyang, Juntang Zhuang, Joyce Lee, Yufei Guo 等人。通过更好的标题改进图像生成。计算机科学。https://cdn.openai.com/papers/dall-e-3.pdf, 2(3), 2023。

+   [3] Junsong Chen, Jincheng Yu, Chongjian Ge, Lewei Yao, Enze Xie, Yue Wu, Zhongdao Wang, James Kwok, Ping Luo, Huchuan Lu 等人。Pixart-$\alpha$: 用于逼真文本到图像合成的扩散变换器的快速训练。arXiv预印本 arXiv:2310.00426, 2023。

+   [4] Minghao Chen, Iro Laina 和 Andrea Vedaldi。无需训练的布局控制与跨注意力引导。发表于IEEE/CVF计算机视觉应用冬季会议论文集，页面5343-5353, 2024。

+   [5] Weixi Feng, Xuehai He, Tsu-Jui Fu, Varun Jampani, Arjun Akula, Pradyumna Narayana, Sugato Basu, Xin Eric Wang 和 William Yang Wang。无需训练的结构化扩散引导用于组合文本到图像合成。arXiv预印本 arXiv:2212.05032, 2022。

+   [6] Weixi Feng, Wanrong Zhu, Tsu-jui Fu, Varun Jampani, Arjun Akula, Xuehai He, Sugato Basu, Xin Eric Wang 和 William Yang Wang。Layoutgpt：使用大语言模型进行组合视觉规划与生成。arXiv预印本 arXiv:2305.15393, 2023。

+   [7] Yaru Hao, Zewen Chi, Li Dong 和 Furu Wei。优化文本到图像生成的提示词。arXiv预印本 arXiv:2212.09611, 2022。

+   [8] Jack Hessel, Ari Holtzman, Maxwell Forbes, Ronan Le Bras, 和 Yejin Choi. Clipscore: 一种无参考图像描述评估指标. arXiv预印本 arXiv:2104.08718, 2021.

+   [9] Jonathan Ho, Ajay Jain, 和 Pieter Abbeel. 去噪扩散概率模型. 神经信息处理系统进展, 33:6840–6851, 2020.

+   [10] 黄凯毅, 孙凯跃, 谢恩泽, 李正国, 和 刘锡辉. T2i-compbench: 一个全面的开放世界组合文本到图像生成基准. arXiv预印本 arXiv:2307.06350, 2023.

+   [11] 李宇恒, 刘浩天, 吴清扬, 穆方舟, 杨建伟, 高建峰, 李春远, 和 李永宰. Gligen: 开放集基础的文本到图像生成. 在IEEE/CVF计算机视觉与模式识别会议论文集, 页码 22511–22521, 2023.

+   [12] 连龙, 李博一, Adam Yala, 和 Trevor Darrell. Llm-grounded扩散: 通过大型语言模型增强文本到图像扩散模型的提示理解. arXiv预印本 arXiv:2305.13655, 2023.

+   [13] 刘浩天, 李春远, 李宇恒, 和 李永宰. 通过视觉指令调优改进基准模型. arXiv预印本 arXiv:2310.03744, 2023.

+   [14] 亚历克斯·尼科尔, Prafulla Dhariwal, Aditya Ramesh, Pranav Shyam, Pamela Mishkin, Bob McGrew, Ilya Sutskever, 和 Mark Chen. Glide: 朝着通过文本引导的扩散模型生成和编辑逼真图像的方向前进. arXiv预印本 arXiv:2112.10741, 2021.

+   [15] OpenAI. Gpt-4v(ision)系统卡. 2023.

+   [16] Dustin Podell, Zion English, Kyle Lacey, Andreas Blattmann, Tim Dockhorn, Jonas Müller, Joe Penna, 和 Robin Rombach. Sdxl: 改进高分辨率图像合成的潜在扩散模型. arXiv预印本 arXiv:2307.01952, 2023.

+   [17] Robin Rombach, Andreas Blattmann, Dominik Lorenz, Patrick Esser, 和 Björn Ommer. 使用潜在扩散模型进行高分辨率图像合成. 在IEEE/CVF计算机视觉与模式识别会议论文集, 页码 10684–10695, 2022.

+   [18] Chitwan Saharia, William Chan, Saurabh Saxena, Lala Li, Jay Whang, Emily L Denton, Kamyar Ghasemipour, Raphael Gontijo Lopes, Burcu Karagol Ayan, Tim Salimans, 等. 具有深度语言理解的逼真文本到图像扩散模型. 神经信息处理系统进展, 35:36479–36494, 2022.

+   [19] Jascha Sohl-Dickstein, Eric Weiss, Niru Maheswaranathan, 和 Surya Ganguli. 使用非平衡热力学的深度无监督学习. 在国际机器学习大会上, 页码 2256–2265. PMLR, 2015.

+   [20] 杨松, Jascha Sohl-Dickstein, Diederik P Kingma, Abhishek Kumar, Stefano Ermon, 和 Ben Poole. 基于得分的生成建模通过随机微分方程. arXiv预印本 arXiv:2011.13456, 2020.

+   [21] Gemini Team, Rohan Anil, Sebastian Borgeaud, Yonghui Wu, Jean-Baptiste Alayrac, Jiahui Yu, Radu Soricut, Johan Schalkwyk, Andrew M Dai, Anja Hauth, 等。Gemini：一系列高度能力的多模态模型。arXiv预印本arXiv:2312.11805, 2023。

+   [22] Ling Yang, Zhaochen Yu, Chenlin Meng, Minkai Xu, Stefano Ermon, and Bin Cui. 精通文本到图像扩散：重新描述、规划和利用多模态LLMs生成。arXiv预印本arXiv:2401.11708, 2024。

## 附录A 更广泛的影响

我们的工作将为专注于扩散模型的研究社区和T2I生成的实际应用带来重大优势。

在研究社区方面，我们提出了一种新的、创新的可控图像生成范式，展示了卓越的可控性，并在处理挑战性任务时取得了显著成果。这一开创性方法可以为未来对扩散模型的研究提供宝贵的见解。

在工业应用方面，我们的方法可以被T2I生成服务提供商直接采用，以增强其模型的性能。此外，我们框架内运行的扩散模型由于在每个生成阶段都施加了精细的控制，因此更不容易生成有害内容。

## 附录B MuLan与并行工作RPG的差异

正如在引言和相关工作中所述，尽管我们承认我们提出的框架与RPG在高层次上有相似的思想，但我们希望强调的是，我们的方法与RPG之间仍然存在显著差异。

首先，我们提出的MuLan旨在根据每个子提示逐步生成每个对象。同时，这些对象是基于之前生成的对象进行生成的。而RPG则是将所有对象独立生成。此外，与需要手动设计上下文示例进行CoT推理的RPG不同，我们的方法不需要这样的要求。我们直接利用LLMs在生成过程中进行规划，这是一个更简单的任务，且LLMs无需进行上下文学习即可完成。更重要的是，MuLan可以通过VLMs的反馈来自适应地控制和修正生成结果，而RPG则没有这样的生成反馈。同时，针对对象之间常见的重叠问题，我们提出了一种策略，通过生成多个候选项来应对这一问题。相较之下，RPG将重叠部分视为整体进行生成。

更重要的是，正如我们在第[4.1节](https://arxiv.org/html/2402.12741v2#S4.SS1 "4.1 主要结果与分析 ‣ 4 实验 ‣ MuLan：用于渐进式和互动式多对象扩散的多模态LLM代理")中所示，我们提出的框架可以在生成过程中直接应用于人机交互，从而促进过程的灵活和有效调整，而RPG无法实现这种交互。总结来说，MuLan与RPG之间的主要差异如下：

+   •

    我们提出的 MuLan 在生成每个对象时，都会根据之前生成的对象进行条件生成，而 RPG 是独立地并行生成所有对象。

+   •

    MuLan 在整个生成过程中不需要任何上下文学习；而 RPG 在进行 Chain-of-Thought 推理时需要专门设计的上下文示例。

+   •

    MuLan 利用基于 VLM 的反馈控制来确保每个对象能够正确生成，而 RPG 没有这种反馈机制。

+   •

    我们提出了一种策略来处理对象之间的重叠/交互，而 RPG 直接将重叠的对象视为一个整体来生成。

+   •

    MuLan 可以直接应用于生成过程中与人类代理的交互，灵活地调整生成过程，而 RPG 无法实现这一点。

## 附录 C (潜在) 扩散模型背景

扩散模型由扩散过程和反向过程组成，已证明通过迭代添加噪声和去噪，能够展示出高质量图像生成的强大能力[[9](https://arxiv.org/html/2402.12741v2#bib.bib9)]。设 $\bm{x}_{0}\sim q(\bm{x}_{0})$ 为真实数据分布。从 $\bm{x}_{0}$ 开始，扩散过程会根据预定义的调度 $\{\beta_{t}\}_{1}^{T}$ 添加不同程度的噪声，生成 $\bm{x}_{1},\cdots,\bm{x}_{T}$。当 $T\to\infty$ 时，$\bm{x}_{T}$ 将趋近于标准高斯分布 $\mathcal{N}(\bm{0},\bm{I})$。因此，反向过程旨在反转上述过程，并通过参数化噪声模型 $\epsilon_{\theta}(\cdot)$ 从 $p(\bm{x}_{T})=\mathcal{N}(\bm{0},\bm{I})$ 重建真实数据分布。设 $\bm{\epsilon}\sim\mathcal{N}(\bm{0},\bm{I})$，则模型的训练损失可以简化为

|  | $\displaystyle L(\theta)=\mathbb{E}_{t,\bm{x}_{0},\bm{\epsilon}}\&#124;\bm{\epsilon}% -\bm{\epsilon}_{\theta}(\sqrt{\bar{\alpha}_{t}}\bm{x}_{0}+\sqrt{1-\bar{\alpha}% _{t}}\bm{\epsilon},t)\&#124;^{2}.$ |  | (3) |
| --- | --- | --- | --- |

潜在扩散模型[[17](https://arxiv.org/html/2402.12741v2#bib.bib17)]由于其高效性和卓越的性能，近年来受到了越来越多的关注。它们不是在像素空间中执行扩散及其反向过程，而是在由预训练编码器 $\mathcal{E}$ 编码的潜在空间 $\bm{z}$ 中添加噪声并进行去噪。因此，扩散过程从 $\bm{z}_{0}=\mathcal{E}(\bm{x}_{0})$ 开始，随后生成潜在状态 $\bm{z}_{1},\cdots,\bm{z}_{t},\cdots,\bm{z}_{T}$。因此，训练损失变为

|  | $\displaystyle L_{LDM}=\mathbb{E}_{\bm{z}_{0},\bm{\epsilon},t}\&#124;\bm{\epsilon}-% \bm{\epsilon}_{\theta}(\bm{z}_{t},t)\&#124;^{2}.$ |  | (4) |
| --- | --- | --- | --- |

## 附录 D 单对象扩散算法过程在 MuLan 中的应用

单一对象扩散的完整详细过程描述在第[3.3节](https://arxiv.org/html/2402.12741v2#S3.SS3 "3.3 条件单一对象扩散与LLM规划和注意力引导 ‣ 3 多模态-LLM代理 (MuLan) ‣ MuLan：用于渐进式和交互式多对象扩散的多模态-LLM代理")中，展示在算法[1](https://arxiv.org/html/2402.12741v2#alg1 "算法1 ‣ 附录D MuLan中单一对象扩散的算法过程 ‣ MuLan：用于渐进式和交互式多对象扩散的多模态-LLM代理")中。

算法1 MuLan中的单一对象扩散

1: 输入: 对象数量 $n$，子提示 $\texttt{p}_{n}$，LLM规划器 Planner，精确掩码 $\bm{\tilde{M}}_{n-1}$（仅当 $n>1$ 时），潜在变量 $\{\bm{z}_{(n-1),(t-1)}\}_{t=1}^{T}$（仅当 $n>1$ 时），注意力引导时间步阈值 $T^{\prime}$，组合时间步阈值 $T^{*}$（仅当 $n>1$ 时），学习率 $\eta$，扩散模型 $\mathcal{D}$。

## 附录E 由LLM进行全局规划的详细提示模板

如第[3.2](https://arxiv.org/html/2402.12741v2#S3.SS2 "3.2 Prompt Decomposition by LLM Planning ‣ 3 Multimodal-LLM Agent (MuLan) ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")节所述，MuLan首先进行全局规划，将输入的提示分解为$N$个物体，然后再开始整个生成过程。为此，给定输入提示p，我们使用以下模板来提示LLM：

你是一个优秀的画家。我将给你一些描述，你的任务是将描述转化为一幅画。你只需要按绘画顺序列出描述中的物体，从左到右，从下到上。不要列出描述中未提及的其他信息。描述：{p}。

通过这种方式，LLM将按照预定义的顺序分解输入提示p。

## 附录F 由LLM进行局部规划的详细提示模板

如第[3.3](https://arxiv.org/html/2402.12741v2#S3.SS3 "3.3 Conditional Single-Object Diffusion with LLM Planning and Attention Guidance ‣ 3 Multimodal-LLM Agent (MuLan) ‣ MuLan: Multimodal-LLM Agent for Progressive and Interactive Multi-Object Diffusion")节所述，LLM也在生成阶段用于规划物体的大致位置和物体数量。

对于第一个物体的大致位置$\texttt{opt}_{1}$的规划，我们使用以下模板：

你是一个优秀的画家。我将给你一些描述，你的任务是将描述转化为一幅画。现在给定描述：{p}。如果我首先想要画出画中的{$\texttt{obj}_{1}$}，应该把{$\texttt{obj}_{1}$}放在哪里？从左、右、上、下中选择。你可以做出合理的猜测。给出一个答案。

然后LLM被提示根据$\texttt{opt}_{1}$来确定物体的数量。

如果$\texttt{opt}_{1}=\texttt{left}$，则$\texttt{obj}_{1}$的提示模板为：

你是一个优秀的画家。我将给你一些描述，你的任务是将描述转化为一幅画。现在给定描述：{p}。在水平位置上有多少个不重叠的物体？只需给出最终的数字。

如果$\texttt{opt}_{1}=\texttt{bottom}$，则提示模板为：

你是一个优秀的画家。我将给你一些描述，你的任务是将描述转化为一幅画。现在给定描述：{p}。在垂直方向上有多少个不重叠的物体？只需给出最终的数字。

对于大致位置$\texttt{opt}_{n}(n\geq 2)$，我们使用以下模板：

你是一个出色的画家。我将给你一些描述。你的任务是将这些描述转化为一幅画。现在给定描述：{p}。如果我已经有一幅包含{$\{\texttt{obj}_{i}\}_{i=1}^{n-1}$}的画，$\texttt{obj}_{n}$相对于$\texttt{obj}_{n-1}$的位置是什么？从左、右、上、下和以上都没有中选择一个。你可以做出合理的猜测。给出一个答案。

然后我们提示LLM通过以下方式确定物体数量：

你是一个出色的画家。我将给你一些描述。你的任务是将这些描述转化为一幅画。现在给定描述：{p}。如果我已经有一幅包含{$\{\texttt{obj}_{i}\}_{i=1}^{n-1}$}的画，$\texttt{opt}_{n}$上的$\texttt{obj}_{n-1}$有多少个物体？只给出最终数字。

## 附录 G 粗略掩模计算的详细信息

当$n=1$时，由于还没有生成任何物体，因此位置$\texttt{opt}_{1}$和$\texttt{Num}_{1}$都是不受限制的，LLM可以通过子提示$\texttt{p}_{1}$来确定$\texttt{opt}_{1}$和$\texttt{Num}_{1}$。由于物体的排列顺序是从左到右、从下到上，因此$\texttt{obj}_{1}$只有两个位置选项$\texttt{opt}_{1}\in\{\texttt{left},\texttt{bottom}\}$。一旦确定了$\texttt{opt}_{1}$，MuLan会将整个图像的宽度/高度($W/H$)均分成$\texttt{Num}_{1}$个部分，并将最左（最下）部分分配给$\texttt{obj}_{1}$，这就导致了以下边界框（计算示意图见图[6](https://arxiv.org/html/2402.12741v2#A7.F6 "图6 ‣ 附录G 粗略掩模计算详细信息 ‣ MuLan：多模态LLM代理用于渐进式和交互式多物体扩散")）：

|  | $\displaystyle\bm{M}_{1}=\begin{cases}(0,0,\frac{W}{\texttt{Num}_{1}},H),&\text% {if }\texttt{opt}_{1}=\texttt{left},\\ (\frac{(\texttt{Num}_{1}-1)\cdot H}{\texttt{Num}_{1}},0,W,\frac{H}{\texttt{Num% }_{1}}),&\text{if }\texttt{opt}_{1}=\texttt{bottom}.\end{cases}$ |  | (5) |
| --- | --- | --- | --- |

![参见说明](img/7f00ec07a318a51fd0a41e318d5d55d8.png)

图6：$\texttt{obj}_{1}$的粗略掩模$\bm{M}_{1}$示意图。由于LLM被提示按从左到右、从下到上的顺序规划物体，掩模只有两个选项$\texttt{left},\texttt{bottom}$。

当$n>1$时，位置$\texttt{opt}_{n}$表示$\{\texttt{obj}\}_{n}$相对于前一个对象$\{\texttt{obj}\}_{n-1}$的关系位置。由于MuLan是从左到右、从下到上生成对象的，因此$\texttt{opt}_{n} \in \texttt{\{right,top\}}$。给定子提示$\texttt{p}_{n}$，一个LLM会选择$\texttt{opt}_{n}$并确定$\texttt{Num}_{n}$。与此同时，$\texttt{opt}_{n-1}$的精确掩码$\bm{\tilde{M}}_{n-1}=(\tilde{x}_{n-1},\tilde{y}_{n-1},\tilde{w}_{n-1},\tilde{h}_{n-1})$可以通过生成的$\{\texttt{obj}\}_{n-1}$从图像中提取（例如，通过扩散模型中的文本-图像跨注意力图），它作为计算粗略掩码$\bm{M}_{n}$边界框的条件。因此，粗略掩码$\bm{M}_{n}$可以根据$\texttt{opt}_{n}$、$\texttt{Num}_{n}$和$\bm{\tilde{M}}_{n-1}$如下计算得出。

|  | $\displaystyle\bm{M}_{n}=\begin{cases}(\tilde{x}_{n-1}+\tilde{w}_{n-1},0,\frac{% W-\tilde{x}_{n-1}+\tilde{w}_{n-1}}{\texttt{Num}_{n}},H),&\text{如果 }\texttt{opt% }_{n}=\texttt{right},\\ \\ (0,\frac{\tilde{y}_{n-1}\cdot(\texttt{Num}_{n}-1)}{\texttt{Num}_{n}},W,\frac{% \tilde{y}_{n-1}}{\texttt{Num}_{n}}),&\text{如果 }\texttt{opt}_{n}=\texttt{top}.% \end{cases}$ |  | (6) |
| --- | --- | --- | --- |

图[7](https://arxiv.org/html/2402.12741v2#A7.F7 "图 7 ‣ 附录 G 粗略掩码计算细节 ‣ MuLan: 逐步和交互式多对象扩散的多模态 LLM 智能体")展示了如何根据前一个对象的精确掩码计算粗略掩码。

![参见说明文字](img/24687ef55a35cabf22933e914e9d19ea.png)

图 7：$\bm{M}_{n}$（$\texttt{obj}_{n}(n>1)$）的粗略掩码源自前一个生成的对象$\texttt{obj}_{n-1}$的精确掩码$\bm{\tilde{M}}_{n-1}$。

## 附录 H 更多关于重叠处理的细节

给定$\texttt{opt}_{n}$和$\bm{\tilde{M}}_{n-1}$，可以计算出粗略掩码$\bm{M}_{n,i}$，计算公式如下：

|  | $\displaystyle\bm{M}_{n,i}=\begin{cases}\Big{(}\tilde{x}_{n-1}\cdot r_{i}+(% \tilde{x}_{n-1}+\tilde{w}_{n-1})\cdot(1-r_{i}),\tilde{y}_{n-1},\tilde{w}_{n-1}% \cdot r_{i}+\frac{W-\tilde{x}_{n-1}-\tilde{w}_{n-1}}{\texttt{Num}_{n}},\tilde{% h}_{n-1}\Big{)},\\ \text{如果 }\texttt{opt}_{n}=\texttt{right},\\ \\ \Big{(}\tilde{x}_{n-1},\frac{(\texttt{Num}_{n}-1)\cdot\tilde{y}_{n-1}}{\texttt% {Num}_{n}},\tilde{w}_{n-1},\tilde{h}_{n-1}\cdot r_{i}+\frac{\tilde{y}_{n-1}}{% \texttt{Num}_{n}}\Big{)},\\ \text{如果 }\texttt{opt}_{n}=\texttt{top}.\end{cases}$ |  | (7) |
| --- | --- | --- | --- |

不同重叠比例的示意图见图[8](https://arxiv.org/html/2402.12741v2#A8.F8 "图 8 ‣ 附录 H 重叠处理的更多细节 ‣ MuLan: 逐步和交互式多对象扩散的多模态 LLM 智能体")。

![参见说明文字](img/da1ce48dac2a1c8be328e64f1ac895ec.png)

图 8：位置为 $\texttt{opt}_{n}=\texttt{top}$ 时，$\texttt{obj}_{n}$ 的三个候选掩码 $\bm{M}_{n,i}$。它们分别表示 $\texttt{obj}_{n}$ 与 $\texttt{obj}_{n-1}$ 重叠的 10%、30% 和 50%。

## 附录 I 评估问卷的更多细节

如在第[4](https://arxiv.org/html/2402.12741v2#S4 "4 实验 ‣ MuLan：面向渐进式和互动式多物体扩散的多模态大语言模型代理")节中所示，我们设计了一份问卷，从三个方面综合评估生成图像与文本之间的对齐程度：物体完整性、属性绑定的正确性和空间关系的正确性。具体来说，给定一张图像和一个文本提示，对于物体完整性，我们将评估图像是否包含提示中的每一个物体。如果物体出现在图像中，我们接下来将判断图像中物体的属性绑定是否与文本提示中的相应属性绑定一致，从而评估属性绑定的正确性。我们还将要求 GPT-4V 或人工判断空间关系是否正确，并与文本匹配，这是对空间关系的评估。

图 LABEL:fig:evaluation 显示了针对不同图像和文本提示的问卷示例。

## 附录 J 更多定性结果

我们在图[11](https://arxiv.org/html/2402.12741v2#A10.F11 "图 11 ‣ 附录 J 更多定性结果 ‣ MuLan：面向渐进式和互动式多物体扩散的多模态大语言模型代理")中展示了不同方法的更多示例。

![参考说明](img/a13a94b45ecea3c8d1a19eab8cb384a6.png)

图 11：不同方法在复杂提示下生成的图像的更多定性示例。
