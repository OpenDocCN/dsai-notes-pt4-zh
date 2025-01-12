<!--yml

类别：未分类

日期：2025-01-11 12:01:19

-->

# Flow-DPO: 通过在线多智能体学习提升LLM数学推理能力

> 来源：[https://arxiv.org/html/2410.22304/](https://arxiv.org/html/2410.22304/)

\UseTblrLibrary

booktabs

邓一赫

加利福尼亚大学洛杉矶分校

yihedeng@cs.ucla.edu

&保罗·米内罗

微软研究院

pmineiro@microsoft.com   邓一赫^(1,2), 保罗·米内罗²

¹加利福尼亚大学洛杉矶分校

²微软研究院

###### 摘要

数学推理是大型语言模型（LLM）至关重要的能力，但生成详细且准确的推理过程仍然是一个显著挑战。本文提出了一种新颖的方法，通过使用在线学习流来生成高质量的推理过程，以进行LLM的微调。我们的方法采用增量输出生成流，在这种流中，组件LLM通过迭代沟通协作构建解决方案。我们通过使用在线直接偏好优化（DPO）学习与回滚训练该流，为每个训练样本生成DPO对，并实时更新模型。我们直接比较了我们的方法生成的推理过程与通过直接模型推理生成的推理过程的质量，证明了我们的方法在改进LLM在数学推理任务中的表现方面的有效性。

## 1 引言

数学推理是大型语言模型（LLM）能力的基本且至关重要的方面，因为它与逻辑一致性和问题解决能力密切相关（Yu等，[2023](https://arxiv.org/html/2410.22304v1#bib.bib15)；Lu等，[2023](https://arxiv.org/html/2410.22304v1#bib.bib8)；Zhang等，[2024c](https://arxiv.org/html/2410.22304v1#bib.bib21)；Gao等，[2024](https://arxiv.org/html/2410.22304v1#bib.bib2)；Liu等，[2024](https://arxiv.org/html/2410.22304v1#bib.bib7)）。这一领域已经引起了广泛的研究兴趣，部分原因是结果验证较为容易。尽管存在大量包含数学问题和答案的数据集，生成详细、准确且清晰的推理步骤仍然是一个显著挑战。虽然人工标注员擅长提供正确的答案，但他们的中间步骤往往过于简洁或杂乱，导致这些数据不足以用于训练LLM。因此，研究人员越来越多地利用LLM生成的推理过程进行模型微调。鉴于仅凭最终答案的正确性提供的反馈有限，越来越多的关注点转向让目标模型生成自己的推理过程以进行自我改进。这一方法在两种情境中尤为相关：（1）推进前沿模型（即提升已经是最先进的模型），以及（2）应对使用大型封闭源模型相比于较小开源替代品所带来的高成本。

该领域的先前研究主要集中在通过推理和筛选从模型本身收集准确的推理痕迹（Zelikman 等， [2022](https://arxiv.org/html/2410.22304v1#bib.bib18)；Yuan 等， [2023](https://arxiv.org/html/2410.22304v1#bib.bib16)；Singh 等， [2023](https://arxiv.org/html/2410.22304v1#bib.bib12)；Hosseini 等， [2024](https://arxiv.org/html/2410.22304v1#bib.bib4)；Pang 等， [2024](https://arxiv.org/html/2410.22304v1#bib.bib10)；Zelikman 等， [2024](https://arxiv.org/html/2410.22304v1#bib.bib17)），随后利用这些痕迹进行监督微调（SFT）或直接偏好优化（DPO）（Rafailov 等， [2024](https://arxiv.org/html/2410.22304v1#bib.bib11)）。拒绝采样微调（RFT）（Yuan 等， [2023](https://arxiv.org/html/2410.22304v1#bib.bib16)）是一种标准且有效的方法，通过收集和筛选产生正确答案的独特模型响应来增加训练数据。该方法通常与基于最终答案的结果奖励相关联。因此，另一项研究方向探讨了过程奖励，旨在通过逐步验证或奖励机制生成优质的推理痕迹。尽管对每个推理步骤的人类注释已被证明显著提升模型性能（Lightman 等， [2023](https://arxiv.org/html/2410.22304v1#bib.bib6)），但此类注释的高昂成本促使研究人员通过将产生正确答案的推理步骤视为优选步骤来近似过程奖励（Wang 等， [2024a](https://arxiv.org/html/2410.22304v1#bib.bib13)；Zhang 等， [2024b](https://arxiv.org/html/2410.22304v1#bib.bib20)；Lai 等， [2024](https://arxiv.org/html/2410.22304v1#bib.bib5)；Wang 等， [2024b](https://arxiv.org/html/2410.22304v1#bib.bib14)）。本质上，给定相同的训练提示（问题）和期望的结果（答案），研究界正在积极寻求有效且高效的方法来生成高质量的推理痕迹，以进行大规模语言模型（LLM）微调。这个过程可以被概念化为一个两步走的方法：数据收集步骤，旨在为痕迹生成识别一个“更好”的操作符，和SFT步骤，将收集的数据以系统1的方式“编译”成一个单一的LLM模型。

本文着重设计一种新颖且改进的管道，用于获取高质量的推理轨迹。我们直接比较了我们方法生成的推理轨迹与通过直接模型推理产生的轨迹质量，使用相同量的数据进行SFT，过滤正确答案并比较SFT模型的性能。我们的方法提议使用在线学习流（Flows）来生成这些轨迹，而不是单一模型推理。这些流由基于相同架构的一系列组件LLM组成，它们通过迭代通信协作构建解决方案（Mineiro, [2024](https://arxiv.org/html/2410.22304v1#bib.bib9)）。具体来说，我们引入了一个增量生产流，其中一个LLM生成有限数量的标记作为答案块，而另一个LLM确定当前维护的部分答案是否已经完成。此外，我们使用在线DPO学习和回滚训练我们的流，在每个答案块生成一批DPO对，并在训练数据到达时更新模型。这个核心概念与过程奖励模型（PRMs）（Lightman等，[2023](https://arxiv.org/html/2410.22304v1#bib.bib6)）一致，旨在逐步生成优越的轨迹，从而在微调过程中提供更密集的奖励。我们的方法通过不将自己限制于预定义的“推理步骤”，提供了更大的灵活性。相反，它允许可调节的块大小，适应仅有几十个标记的精细化块，并在使用较大块大小时推广到结果奖励模型。最后，我们的方法仍然兼容进一步的增强技术，如数据扩增和DPO。

## 2 方法

![参见说明](img/cdbe768409b2b9fd122a36b2e11063c8.png)

图1：增量生产流程示意图。Answer LLM 被指定生成具有有限标记数量的答案块。Stop LLM 确定当前部分答案是否已达到令人满意的最终答案。

#### 增量输出生产流。

我们尝试了不同的流架构，并通过增量输出生产设计取得了最佳结果。如图[1](https://arxiv.org/html/2410.22304v1#S2.F1 "Figure 1 ‣ 2 Method ‣ Flow-DPO: Improving LLM Mathematical Reasoning through Online Multi-Agent Learning")所示，该实现主要涉及两个架构相同的独立LLM：答案LLM和停止LLM。答案LLM一次生成一个响应块，遵循预定的最大令牌限制。我们维护一个初始为空的部分答案，每次生成的新答案块都会被附加到其中。然后，停止LLM对该部分答案进行评估，以判断是否已完成完整响应。这个迭代过程持续进行，直到停止LLM发出最终答案完成的信号。因此，流（Flow）逐步构建响应，较小的块大小能够提供更精细的控制，而较大的块大小则近似于单次生成模型。值得注意的是，答案LLM和停止LLM都基于相同的基础模型开始，但通过不同的LoRA适配器进行微调，以专门化各自的任务。

#### 在线流学习与回滚。

我们通过在线DPO学习进一步增强了Flow，在每个输出节点上都加入了随机回滚。图[2](https://arxiv.org/html/2410.22304v1#S2.F2 "Figure 2 ‣ Online Flow Learning with Rollouts. ‣ 2 Method ‣ Flow-DPO: Improving LLM Mathematical Reasoning through Online Multi-Agent Learning")展示了这一训练过程。对于每个输入问题，Flow从回答LLM生成一个答案块开始，直到生成完整的回答。基于这一输出链，我们在每个输出节点执行一次随机回滚。例如，在初始答案块生成和Stop agent做出“否”判断后，我们允许Flow基于之前的部分答案生成另一个替代答案块。这个过程会继续，直到生成第二个完整答案。如果两个答案在正确性上有所不同，我们将它们视为Answer LLM的DPO对，其中正确答案的块被选为首选回答。重要的是，Answer LLM和Stop LLM都参与这些回滚和后续微调，后者的停止决策会被评估。对于每个包含问题和答案的训练实例，我们生成一批DPO对来训练两个LLM。这种方法支持在线训练方案，随着新数据的处理，逐步更新模型。这一方法与并行的基于MCTS的方法（Zhang等人，[2024a](https://arxiv.org/html/2410.22304v1#bib.bib19)，[b](https://arxiv.org/html/2410.22304v1#bib.bib20)）具有相似的直觉，通过选择最有前景的子步骤遍历推理步骤的树，直到找到答案。从每个新扩展的步骤开始，他们执行随机回滚来估算该步骤的奖励。然而，我们仅在每个节点执行一次随机回滚，而不是遍历整个树，以提高效率。此外，我们并非在预定义的推理步骤上进行优化，而是在细粒度的答案块上进行在线DPO学习。

![请参阅说明文字](img/48bde6a01b6d4a7c7ce3168db11ccb0a.png)

图2：带回滚的DPO训练示意图。在初始生成的每个节点，我们进行一次与原节点不同的随机回滚，并继续生成直到最终答案。导致不同答案（正确与错误）的一对被视为DPO训练数据。

## 3 结果

### 3.1 实验设置。

在我们的实验中，我们将一个 LLM 模型应用于整个 Flow（包括 Answer LLM 和 Stop LLM）以及 Compile 步骤。对于该模型，我们采用了两种不同规模的、近期的竞争性模型：Llama-3-8B-Instruct 和 Phi-3-medium-128k-instruct（14B）。为了研究我们方法的有效性，我们使用 MetaMath（Yu 等， [2023](https://arxiv.org/html/2410.22304v1#bib.bib15)）作为训练数据集。MetaMath 来源于 GSM8K（Cobbe 等， [2021](https://arxiv.org/html/2410.22304v1#bib.bib1)）和 MATH（Hendrycks 等， [2021](https://arxiv.org/html/2410.22304v1#bib.bib3)）的训练数据，并通过数据增强技术进行了提升。我们在 Compile 步骤中，评估了 GSM8K 和 MATH 数据集上的推理轨迹质量。在 Flow 学习阶段，我们为 Answer LLM 和 Stop LLM 使用了独立的 LoRA 适配器，以便在 DPO 训练期间使它们的能力得到专业化。在 Compile 阶段，我们收集了来自 Flow 和基线的相同数量的正确答案推理轨迹数据，从而使我们能够独立评估推理质量，并通过 SFT 查看它如何增强单一模型的表现。我们在所有基线中均使用了来自 MetaMath 的 $1,500$ 条数据子集用于 Compile。为了确保所有基线的一致性，我们在 SFT 过程和评估中保持相同的超参数和系统提示。

### 3.2 逐步验证准确度

我们首先通过检查在在线 DPO 训练过程中，Flow 随着回放的进行，逐步验证准确度。逐步验证准确度定义为模型在训练前，对新输入训练数据的累计准确度：

|  | $\displaystyle\text{Acc}_{\text{prog}}^{N}$ | $\displaystyle=\frac{1}{N}\sum_{i=1}^{N}\mathbb{I}\big{(}\Theta^{(i-1)}(\mathbf% {x}_{i})=y_{i}\big{)},\vspace{-4mm}$ |  |
| --- | --- | --- | --- |

其中，$N$ 是已见训练数据的数量，表示基于前 $i-1$ 个数据点微调的语言模型，$\mathbf{x}_{i}$ 是数据中的第 $i$ 个问题，$y_{i}$ 是正确答案。该指标作为 Flow 在整个训练过程中泛化性能的可靠指示器。图 [3](https://arxiv.org/html/2410.22304v1#S3.F3 "Figure 3 ‣ 3.2 Progressive Validation Accuracy ‣ 3 Results ‣ Flow-DPO: Improving LLM Mathematical Reasoning through Online Multi-Agent Learning") 和 [4](https://arxiv.org/html/2410.22304v1#S3.F4 "Figure 4 ‣ 3.2 Progressive Validation Accuracy ‣ 3 Results ‣ Flow-DPO: Improving LLM Mathematical Reasoning through Online Multi-Agent Learning") 展示了我们 Flow 的渐进验证准确率，分别在有训练和无训练的情况下，并与单一 LLM 在一步生成推理和答案时的零-shot表现进行了对比。在没有训练的情况下，Flow 的推理准确率稍微低于独立模型的准确率。这一差异表明 Flow 在处理任务特定需求时的初期效率较低，例如在基于部分答案明确判断何时结束推理或继续推理方面。 这些结果突出了训练过程在优化 Flow 对复杂推理任务性能中的重要性。与此同时，在线 DPO 训练有效提升了 Flow 在跨多种 LLM 模型的在线学习过程中对新数据的泛化能力。对于 Llama-3-8B-Instruct 模型，在线 DPO 学习在仅 $2,000$ 个训练实例内就显著提升了 Flow 的表现，提升幅度达到 $20\%$。类似地，对于 Phi-3-medium-128k-instruct 模型，该模型在数学推理方面表现强劲，训练数据上的零-shot 准确率为 $79\%$，而在线 DPO 学习使其准确率提升了 4 个百分点，接近 $83\%$。我们注意到，在线训练方案使得我们可以将渐进验证准确率作为早停的良好指示器。

![参考图注](img/4d1ed02f7864336c0a29e2c6d3c5f4bf.png)

图 3：Llama-3-Instruct 在 MetaMath 上的渐进验证准确率。

![参考图注](img/7b77f7a72e27913b8ad48a0d2eccea87.png)

图 4：Phi-3-Medium 在 MetaMath 上的渐进验证准确率。

### 3.3 编译

为了评估流生成推理痕迹的质量，我们将其与在编译步骤中生成的模型痕迹进行比较，在此步骤中，我们使用为SFT收集的推理痕迹对单一LLM进行微调。我们通过模型的零-shot准确率和使用数据集中的真实标签痕迹进行SFT后的性能来建立基准。此外，我们还考虑将模型生成的正确痕迹作为强大且广泛使用的自我训练基准 （Yuan et al., [2023](https://arxiv.org/html/2410.22304v1#bib.bib16); Singh et al., [2023](https://arxiv.org/html/2410.22304v1#bib.bib12); Hosseini et al., [2024](https://arxiv.org/html/2410.22304v1#bib.bib4)）。为了确保痕迹质量的公平比较，我们在所有基准中保持一致的数据量，专注于那些能导致正确答案的痕迹。比较结果如表[3.3](https://arxiv.org/html/2410.22304v1#S3.SS3 "3.3 Compile ‣ 3 Results ‣ Flow-DPO: Improving LLM Mathematical Reasoning through Online Multi-Agent Learning")所示。

表 1：比较用于SFT的痕迹质量的主要结果。我们报告了每个模型在相同提示集上微调后的准确率（%），但答案来源有所不同。对于Phi-3，由于其在数据集上的性能已优化，因此不包括GSM8K。

{tblr}

colspec = cccc, row1 = bg=gray!25, row2-5 = bg=gray!5 模型 方法 GSM8K MATH

\SetCell[r=4]cLlama-3-Instruct (8B) 0-shot 48.9 22.3

SFT (真实标签) 67.2 25.1

SFT (自生成) 68.8 24.2

SFT (流生成) 71.3 27.8

\SetCell

[r=4]cPhi-3-Medium (14B) 0-shot - 35.4

SFT (真实标签) - 36.3

SFT (自生成) - 36.5

SFT (流生成) - 38.6

### 3.4 定性分析。

我们进行了一项定性分析，比较了我们提出的流方法生成的推理过程与数据集中的真实标注。通过对示例问题的检查，我们展示了虽然两种方法都能得出正确答案，但流方法生成的推理过程提供了更详细的教学指导。为了验证这些观察结果，我们使用了GPT-4o进行系统的定性评价。评估结果与我们的观察一致，表明流方法生成的回答（响应B）质量更高。具体而言，它强调了流方法生成的推理过程通过突出关键概念（例如垂直线的斜率之间的负倒数关系），同时保持逻辑性、一步一步的解题过程，提供了更清晰的教学指导。该回答避免了不必要的复杂性，专注于核心步骤，从而提高了可访问性和理解的容易度。我们在附录[A.1](https://arxiv.org/html/2410.22304v1#A1.SS1 "A.1 额外的定性分析 ‣ 附录 A 实验详情 ‣ 4 结论 ‣ 3.4 定性分析 ‣ 3.3 汇编 ‣ 3 结果 ‣ Flow-DPO：通过在线多代理学习提升LLM数学推理")中提供了一个额外的示例。

<svg class="ltx_picture" height="755.94" id="S3.SS4.p2.pic1" overflow="visible" version="1.1" width="603.54"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,755.94) matrix(1 0 0 -1 0 0) translate(0,3.54)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 17.72 738.13)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="564.57">推理过程示例</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 17.72 9.84)"><foreignobject color="#000000" height="718.45" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="564.57">问题：</foreignobject></g></svg>

方程$x+2y+3=0$的图像与方程$ax+2y+3=0$的图像垂直。$a$的值是多少？ 响应A（真实标注）：

方程$x+2y+3=0$的图像斜率为$-\frac{1}{2}$。由于这两条图像是垂直的，方程$ax+2y+3=0$的图像斜率必须是$-\frac{1}{2}$的负倒数，即$2$。因此，方程$ax+2y+3=0$的斜率为$2$。为了求解$a$，我们可以将方程重写为$2y=-ax-3$，然后两边同时除以$2$，得到$y=-\frac{a}{2}x-\frac{3}{2}$。与斜截式$y=mx+b$比较，我们可以看出斜率$m$为$-\frac{a}{2}$。由于我们知道斜率为$2$，所以有$-\frac{a}{2}=2$。两边同时乘以$-2$，得到$a=\boxed{-4}$。答案是：-4 响应B（流方法生成的）：

要解决这个问题，我们需要理解在直线方程的背景下垂直线的概念。当两条线是垂直时，它们的斜率是彼此的负倒数。

我们从将给定方程$x+2y+3=0$重写成斜截式开始，即$y=-\frac{1}{2}x-\frac{3}{2}$。这告诉我们，直线的斜率是$-\frac{1}{2}$。

现在，我们需要找到方程$ax+2y+3=0$所表示的直线的斜率。我们可以通过将该方程重写成斜截式来做到这一点，即$y=-\frac{a}{2}x-\frac{3}{2}$。这告诉我们，这条直线的斜率是$-\frac{a}{2}$。

因为这两条直线是垂直的，所以它们的斜率是互为负倒数的。这意味着$-\frac{1}{2}\cdot(-\frac{a}{2})=1$，简化后得到$a=-4$。

因此，$a$的值是$\boxed{-4}$。GPT-4o评估：B响应更好，原因如下：1\. 概念解释  A响应正确识别出这两条直线是垂直的，并且正确地使用了斜率负倒数关系来解出$a$。然而，它在没有充分解释斜率为何应该是负倒数的情况下就直接进入了代数计算。B响应则花时间明确解释了垂直性概念，强调了垂直线的斜率是负倒数关系。这使得B响应更具教育性，确保读者理解数学背后的推理。2\. 解题过程的清晰度  A响应推导出了正确的方程并正确解出结果，但引入了一些不必要的步骤。例如，将方程重写为$2y=-ax-3$并两边除以2的中间步骤比必要的要复杂。B响应通过直接专注于两条方程的斜截式来简化方法。它有效地解释了如何提取斜率并直接比较两条方程的斜率，使得过程更加简洁易懂。结论：B响应提供了更清晰、更全面的解释，既关注概念又关注代数，同时避免了不必要的复杂性。其逐步的方法使得过程更加易于理解和跟随。

## 4 结论

我们提出了一种有效的方法，通过使用给定的LLM生成高质量的推理轨迹，从而增强其数学推理能力。我们的方法引入了Flow，一个多代理对话框架，利用多个LLM通过迭代沟通协作解决复杂问题。我们进一步使用带有回滚的在线DPO学习对这一框架进行优化。对各种LLM规模在广泛使用的数学基准上的实证评估表明，我们的方法具有较高的有效性，结果显示，Flow生成的推理轨迹在质量上优于真实值和模型生成的正确轨迹。我们的方法在适应不同块大小方面的灵活性以及其在处理各种复杂推理任务中的应用性，展示了其在多个场景和领域中潜在的可扩展性。未来的研究方向可能包括优化训练过程、研究增加数据的影响，并将我们的方法扩展到其他需要复杂推理能力的领域。

## 参考文献

+   Cobbe 等人（2021）Cobbe, K., Kosaraju, V., Bavarian, M., Chen, M., Jun, H., Kaiser, L., Plappert, M., Tworek, J., Hilton, J., Nakano, R., Hesse, C. 和 Schulman, J.（2021）。训练验证器来解决数学文字题。arXiv 预印本 arXiv:2110.14168。

+   Gao 等人（2024）Gao, B., Song, F., Yang, Z., Cai, Z., Miao, Y., Dong, Q., Li, L., Ma, C., Chen, L., Xu, R. 等人（2024）。Omni-math: 一个用于大型语言模型的通用奥林匹克级数学基准。arXiv 预印本 arXiv:2410.07985。

+   Hendrycks 等人（2021）Hendrycks, D., Burns, C., Kadavath, S., Arora, A., Basart, S., Tang, E., Song, D. 和 Steinhardt, J.（2021）。使用数学数据集衡量数学问题解决能力。arXiv 预印本 arXiv:2103.03874。

+   Hosseini 等人（2024）Hosseini, A., Yuan, X., Malkin, N., Courville, A., Sordoni, A. 和 Agarwal, R.（2024）。V-star: 为自学推理者训练验证器。arXiv 预印本 arXiv:2402.06457。

+   Lai 等人（2024）Lai, X., Tian, Z., Chen, Y., Yang, S., Peng, X. 和 Jia, J.（2024）。Step-dpo: 用于长链推理的分步偏好优化。arXiv 预印本 arXiv:2406.18629。

+   Lightman 等人（2023）Lightman, H., Kosaraju, V., Burda, Y., Edwards, H., Baker, B., Lee, T., Leike, J., Schulman, J., Sutskever, I. 和 Cobbe, K.（2023）。让我们一步一步验证。arXiv 预印本 arXiv:2305.20050。

+   Liu 等人（2024）Liu, H., Zheng, Z., Qiao, Y., Duan, H., Fei, Z., Zhou, F., Zhang, W., Zhang, S., Lin, D. 和 Chen, K.（2024）。Mathbench: 通过层次数学基准评估大型语言模型的理论与应用能力。arXiv 预印本 arXiv:2405.12209。

+   Lu 等人（2023）Lu, P., Bansal, H., Xia, T., Liu, J., Li, C., Hajishirzi, H., Cheng, H., Chang, K.-W., Galley, M. 和 Gao, J.（2023）。Mathvista: 评估基础模型在视觉上下文中的数学推理能力。arXiv 预印本 arXiv:2310.02255。

+   Mineiro（2024）Mineiro, P.（2024）。多代理流的在线联合微调。arXiv 预印本 arXiv:2406.04516。

+   Pang 等人（2024）Pang, R. Y., Yuan, W., Cho, K., He, H., Sukhbaatar, S. 和 Weston, J.（2024）。迭代推理偏好优化。arXiv 预印本 arXiv:2404.19733。

+   Rafailov 等人（2024）Rafailov, R., Sharma, A., Mitchell, E., Manning, C. D., Ermon, S. 和 Finn, C.（2024）。直接偏好优化：你的语言模型其实是一个奖励模型。神经信息处理系统进展 36。

+   Singh 等人（2023）Singh, A., Co-Reyes, J. D., Agarwal, R., Anand, A., Patil, P., Liu, P. J., Harrison, J., Lee, J., Xu, K., Parisi, A. 等（2023）。超越人类数据：通过语言模型扩展自我训练以解决问题。arXiv 预印本 arXiv:2312.06585。

+   Wang 等人（2024a）Wang, P., Li, L., Shao, Z., Xu, R., Dai, D., Li, Y., Chen, D., Wu, Y. 和 Sui, Z.（2024a）。Math-shepherd：在没有人工标注的情况下逐步验证和强化大语言模型。发表于第62届计算语言学协会年会（卷1：长篇论文）。

+   Wang 等人（2024b）Wang, Z., Li, Y., Wu, Y., Luo, L., Hou, L., Yu, H. 和 Shang, J.（2024b）。通过验证器进行多步问题求解：基于模型引导的过程监督的实证分析。arXiv 预印本 arXiv:2402.02658。

+   Yu 等人（2023）Yu, L., Jiang, W., Shi, H., Yu, J., Liu, Z., Zhang, Y., Kwok, J. T., Li, Z., Weller, A. 和 Liu, W.（2023）。Metamath：为大语言模型引导自己的数学问题生成。arXiv 预印本 arXiv:2309.12284。

+   Yuan 等人（2023）Yuan, Z., Yuan, H., Li, C., Dong, G., Lu, K., Tan, C., Zhou, C. 和 Zhou, J.（2023）。大规模语言模型在学习数学推理中的扩展关系。arXiv 预印本 arXiv:2308.01825。

+   Zelikman 等人（2024）Zelikman, E., Harik, G., Shao, Y., Jayasiri, V., Haber, N. 和 Goodman, N. D.（2024）。Quiet-star：语言模型可以自我学习在讲话前进行思考。arXiv 预印本 arXiv:2403.09629。

+   Zelikman 等人（2022）Zelikman, E., Wu, Y. 和 Goodman, N. D.（2022）。Star：自学推理器。arXiv 预印本 arXiv:2203.14465。

+   Zhang 等人（2024a）Zhang, D., Wu, J., Lei, J., Che, T., Li, J., Xie, T., Huang, X., Zhang, S., Pavone, M., Li, Y. 等（2024a）。Llama-berry：针对类似奥林匹克级数学推理的成对优化。arXiv 预印本 arXiv:2410.02884。

+   Zhang 等人（2024b）Zhang, D., Zhoubian, S., Yue, Y., Dong, Y. 和 Tang, J.（2024b）。Rest-mcts*：通过过程奖励引导的树搜索进行语言模型自我训练。arXiv 预印本 arXiv:2406.03816。

+   Zhang 等人（2024c）Zhang, R., Jiang, D., Zhang, Y., Lin, H., Guo, Z., Qiu, P., Zhou, A., Lu, P., Chang, K.-W., Gao, P. 等（2024c）。Mathverse：你的多模态大语言模型真的能看到视觉数学问题中的图表吗？arXiv 预印本 arXiv:2403.14624。

## 附录 A 实验细节

### A.1 额外定性分析

在下面的示例中，我们展示了一个额外的定性示例，同样表明流生成的响应（响应B）具有更好的质量。GPT-4o的评估强调，流生成的推理轨迹提供了结构化的、逐步的解释，增强了可理解性。此外，评估还强调了我们的流方法在每个推理步骤中在清晰度和解释深度上的显著提高。

<svg class="ltx_picture" height="836.26" id="A1.SS1.p2.pic1" overflow="visible" version="1.1" width="603.54"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,836.26) matrix(1 0 0 -1 0 0) translate(0,3.54)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 17.72 818.45)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="564.57">推理轨迹示例</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 17.72 9.84)"><foreignobject color="#000000" height="798.76" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="564.57">问题：

理久比克里斯托夫多25倍的贴纸。如果克里斯托夫有x个贴纸，理久有2210个贴纸。未知数x的值是多少？响应A（真实答案）：

为了解决这个问题，我们需要确定x的值，它代表克里斯托夫拥有的贴纸数。我们知道理久比克里斯托夫多25倍的贴纸。因此，理久有x + 25x = 26x个贴纸。我们还知道理久有2210个贴纸。我们可以建立如下方程：26x = 2210 让我们解出x：x = 2210 / 26 x = 85 x的值是85。#### 85 答案是：85 响应B（流生成的）：

让我们一步步地分解这个问题。

我们知道理久比克里斯托夫多25倍的贴纸。这意味着理久的贴纸数是克里斯托夫贴纸数的25倍。

我们还知道理久有2210个贴纸。所以，我们可以建立一个方程：

理久的贴纸数 = 25 $\times$ 克里斯托夫的贴纸 + 克里斯托夫的贴纸

由于理久有2210个贴纸，我们可以将这个值代入方程：

2210 = 25 $\times$ 克里斯托夫的贴纸 + 克里斯托夫的贴纸

现在，让我们通过合并项来简化方程：

2210 = 26 $\times$ 克里斯托夫的贴纸

为了解出克里斯托夫的贴纸数，我们可以将方程两边都除以26：

$\boxed{85}$ GPT-4o评估：

响应 B 更好，原因如下：1. 清晰度和结构 响应 A 直接跳入数学公式，陈述方程并快速求解。然而，它缺乏逐步的拆解，这在解释解决方案背后的推理时非常重要。响应 B 以更详细和结构化的方式解释问题。它首先澄清了 Riku 和 Kristoff 拥有的贴纸数量之间的关系，然后小心地建立方程。这种方法更具教学性，使得读者更容易跟随解决方案背后的逻辑。 2. 问题的正确理解 两个回答都正确地通过推导方程 $26x=2210$ 并求解 $x=85$。响应 A 在措辞上存在问题，它说：“Riku 有 x + 25x = 26x 张贴纸。”虽然数学是正确的，但措辞有点误导，因为“比……多 25 倍”与“是……的 25 倍”解释不同。响应 B 更清晰地表述了这一点，说：“Riku 的贴纸数量是 Kristoff 贴纸数量的 25 倍。” 3. 教育价值 响应 B 花时间简化方程并解释每个步骤，使得学生或学习者更容易理解这个过程。它还使用了更好的术语，比如“简化方程”，并分解了除法，提升了回应的教育质量。 结论：

响应 B 提供了一个更全面、结构化且在教学上更合理的解释。它引导用户逐步解决问题，使得每一步都更容易理解。

### A.2 超参数

实验是在 NVIDIA RTX A100（80G）GPU 集群上进行的。在线 DPO 微调推理轨迹生成过程大约需要 $36$-$48$ 小时，使用 $4$ 个 GPU。编译（SFT）过程大约需要 $1$ 小时，使用 $1$ 个 GPU。

表 2：在线 DPO 微调超参数。

| 学习率 | 5e-6 |
| --- | --- |
| 优化器 | Adam |
| 全局批处理大小 | 32 |
| DPO 系数 $\beta$ | 1.0 |
| 梯度裁剪 | 1.0 |
| lora_r | 8 |
| lora_alpha | 8 |
| lora_dropout | 0.05 |
| lora_target | all |
| 最大步数（块数） | 6 |
| 块大小 | 160 |

表 3：编译（SFT）超参数。

| 学习率 | 2e-4 |
| --- | --- |
| 优化器 | Adam |
| 全局批处理大小 | 16 |
| 梯度裁剪 | 1.0 |
| 梯度累积步数 | 2 |
| 预热比例 | 0.1 |
| lora_r | 16 |
| lora_alpha | 16 |
| lora_dropout | 0.05 |
| lora_target | all |
| 训练周期 | 3 |

### A.3 提示

<svg class="ltx_picture" height="114.98" id="A1.SS3.p1.pic1" overflow="visible" version="1.1" width="603.54"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,114.98) matrix(1 0 0 -1 0 0) translate(0,3.54)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 17.72 97.17)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="564.57">回答 LLM 的提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 17.72 9.84)"><foreignobject color="#000000" height="77.49" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="564.57"><System>

你是一个有帮助的数学助手。请解释你的推理过程，然后解答问题。

<用户>

{输入问题}</foreignobject></g></g></svg><svg class="ltx_picture" height="196.78" id="A1.SS3.p2.pic1" overflow="visible" version="1.1" width="603.54"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,196.78) matrix(1 0 0 -1 0 0) translate(0,3.54)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 17.72 178.96)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="564.57">停止 LLM 的提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 17.72 9.84)"><foreignobject color="#000000" height="159.28" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="564.57"><System>

你是一个助手，只能回答“是”或“否”。在接下来的任务中，你将得到一个问题和一个候选解答。判断候选解答是否正确。

<用户>

问题：{problem}

候选解答：{solution}

候选解答是否正确？请仅回答“是”或“否”。</foreignobject></g></g></svg><svg class="ltx_picture" height="180.17" id="A1.SS3.p3.pic1" overflow="visible" version="1.1" width="603.54"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,180.17) matrix(1 0 0 -1 0 0) translate(0,3.54)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 17.72 162.36)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="564.57">GPT-4o 评估提示</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 17.72 9.84)"><foreignobject color="#000000" height="142.67" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="564.57">审查用户的问题和相应的两个回答。判断哪个回答更好。

用户：<问题>

回答 A：<response A>

回答 B：<response B>

在检查原始问题、回答以及两个判断之后：

- 解释哪个回答更好，并说明原因。

- 以明确的声明结束，说明哪个回答更好。</foreignobject></g></g></svg>
