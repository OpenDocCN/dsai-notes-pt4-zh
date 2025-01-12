<!--yml

category: 未分类

date: 2025-01-11 11:54:26

-->

# Star-Agents: 基于LLM代理的指令调优自动数据优化

> 来源：[https://arxiv.org/html/2411.14497/](https://arxiv.org/html/2411.14497/)

\floatsetup

[table]capposition=top \newfloatcommandcapbtabboxtable[][\FBwidth]

杭周^(1,2)，唐业辉²，秦浩辰²，杨宇杰²，金仁仁¹，

熊德毅¹^∗，韩凯²^∗，王云鹤²

¹天津大学智能与计算学院，天津，中国。

²华为诺亚方舟实验室。

{zhouhang25, yehui.tang, qinhaochen1, yangyujie26}@huawei.com，

{rrjin, dyxiong}@tju.edu.cn, {kai.han, yunhe.wang}@huawei.com 通讯作者。

###### 摘要

大型语言模型（LLM）在下游任务中的效能通常依赖于指令调优，而这高度依赖于训练数据的质量。不幸的是，收集高质量和多样化的数据既昂贵又耗时。为了解决这个问题，我们提出了一种新颖的Star-Agents框架，它通过多代理协作和评估自动增强数据集的质量。该框架采用三管齐下的策略。首先，它通过定制的采样方法，利用多个LLM代理生成多样化的指令数据。随后，生成的数据经过双模型方法的严格评估，评估内容包括难度和质量。最后，以上过程进入动态优化阶段，在该阶段中，更有效的LLM模型被优先使用，从而提升整体数据质量。我们的实证研究，包括使用Pythia和LLaMA等模型进行的指令调优实验，验证了该框架的有效性。优化后的数据集在多个指标上取得了显著的提升，平均提高了12%，并在特定指标上有显著进步，如Fermi的提升高达40%，这一点在MT-bench、Vicuna bench和WizardLM测试集中得到了验证。代码将很快发布¹¹1https://github.com/CANGLETIAN/Star-Agents。

## 1 引言

自从LLMs的出现和普及以来，自然语言理解和生成的研究与开发得到了显著加速[[39](https://arxiv.org/html/2411.14497v1#bib.bib39)、[31](https://arxiv.org/html/2411.14497v1#bib.bib31)、[30](https://arxiv.org/html/2411.14497v1#bib.bib30)]。这些模型已广泛应用于多种场景，例如问答和文本生成，由于其出色的指令遵循能力，显著提升了下游任务的表现[[3](https://arxiv.org/html/2411.14497v1#bib.bib3)、[53](https://arxiv.org/html/2411.14497v1#bib.bib53)、[49](https://arxiv.org/html/2411.14497v1#bib.bib49)、[10](https://arxiv.org/html/2411.14497v1#bib.bib10)、[28](https://arxiv.org/html/2411.14497v1#bib.bib28)]。这种指令遵循能力主要通过一种被称为“指令调优”的过程获得[[40](https://arxiv.org/html/2411.14497v1#bib.bib40)、[23](https://arxiv.org/html/2411.14497v1#bib.bib23)、[5](https://arxiv.org/html/2411.14497v1#bib.bib5)]，在该过程中，LLMs会根据指令数据进行微调。因此，普遍认为，指令的质量在其中起着至关重要的作用[[5](https://arxiv.org/html/2411.14497v1#bib.bib5)、[20](https://arxiv.org/html/2411.14497v1#bib.bib20)、[48](https://arxiv.org/html/2411.14497v1#bib.bib48)、[29](https://arxiv.org/html/2411.14497v1#bib.bib29)]。

从历史上看，训练大型语言模型（LLMs）的指令数据创建在很大程度上依赖于人工标注者的专业知识，这一点可以从大量学术贡献中得到验证[[14](https://arxiv.org/html/2411.14497v1#bib.bib14)、[50](https://arxiv.org/html/2411.14497v1#bib.bib50)、[41](https://arxiv.org/html/2411.14497v1#bib.bib41)、[38](https://arxiv.org/html/2411.14497v1#bib.bib38)、[9](https://arxiv.org/html/2411.14497v1#bib.bib9)、[27](https://arxiv.org/html/2411.14497v1#bib.bib27)、[21](https://arxiv.org/html/2411.14497v1#bib.bib21)]。尽管专家驱动的数据生成确保了高质量指令的产生，但为有效训练所需的大量数据使得这种方法在经济上难以持续。因此，最近的努力已经转向利用LLMs自动生成指令，从而减轻了对昂贵人工标注的依赖[[37](https://arxiv.org/html/2411.14497v1#bib.bib37)、[32](https://arxiv.org/html/2411.14497v1#bib.bib32)、[44](https://arxiv.org/html/2411.14497v1#bib.bib44)、[18](https://arxiv.org/html/2411.14497v1#bib.bib18)]。与此同时，越来越多的关注也集中在生成和选择具有挑战性的示例上，基于这样的信念：更复杂、更困难的指令可以显著提升模型的能力[[22](https://arxiv.org/html/2411.14497v1#bib.bib22)、[17](https://arxiv.org/html/2411.14497v1#bib.bib17)]。

尽管使用LLM生成数据具有明显的优势，但这种策略仍然存在一些挑战。首先，以前的努力通常依赖于单一的LLM，导致生成的数据可能缺乏风格多样性 [[4](https://arxiv.org/html/2411.14497v1#bib.bib4)]，并且涵盖的难度范围有限，这可能并不适合所有模型。此外，生成过于复杂的指令 [[19](https://arxiv.org/html/2411.14497v1#bib.bib19), [44](https://arxiv.org/html/2411.14497v1#bib.bib44), [18](https://arxiv.org/html/2411.14497v1#bib.bib18)] 的趋势也日益明显，这可能超过了参数规模较小的模型的操作能力，从而阻碍了它们充分利用数据在性能提升方面的潜力。

为了解决上述挑战，我们提出了Star-Agents框架，这是一个先进的自动化数据优化系统，专门设计用于学习和优化具有适当复杂度和多样性的指令样本，以适应目标LLM。该框架由三个主要部分组成。首先，为了增加生成数据的多样性，提出了一种涉及多个高级LLM代理的指令数据重写过程。该过程通过采样不同的LLM代理分别重写指令和响应（称为代理对）。接下来，为了选择高质量样本，生成的数据将通过一个双模型评估函数，以适当的复杂度作为选择标准。最后，为了平衡数据的多样性和质量，代理对的采样概率会根据所选数据的综合得分进行调整和演化，从而识别出能够生成高质量数据的代理对。

我们进行了广泛的实验，评估了LLM在各种基准数据集上的指令跟随能力，包括MT-bench [[54](https://arxiv.org/html/2411.14497v1#bib.bib54)]、Vicuna-bench [[54](https://arxiv.org/html/2411.14497v1#bib.bib54)]以及WizardLM测试集 [[44](https://arxiv.org/html/2411.14497v1#bib.bib44)]。在Pythia和LLaMA等LLM上的指令微调实验，展示了Star-Agents框架的有效性。基于Star-Agent生成的数据训练的LLM在表现上优于那些（相同的LLM）基于Evol-Instruct数据集 [[44](https://arxiv.org/html/2411.14497v1#bib.bib44)]或根据指令跟随难度（IFD）度量 [[20](https://arxiv.org/html/2411.14497v1#bib.bib20)]选择的数据训练的LLM。显著的是，经过优化的数据集在平均性能上提高了12%，其中一些指标，如Fermi，表现出超过40%的增益。

## 2 相关工作

我们的工作涉及指令数据生成和选择。由于篇幅限制，我们简要回顾了这些主题。

#### 指令数据生成

像Dolly [[7](https://arxiv.org/html/2411.14497v1#bib.bib7)] 和 OpenAssistant [[15](https://arxiv.org/html/2411.14497v1#bib.bib15)] 这样的数据集是由人工生成的指令数据构建的。由人类与ChatGPT之间的对话构建的ShareGPT数据集，已被有效地用于提高微调模型的指令跟随性能 [[6](https://arxiv.org/html/2411.14497v1#bib.bib6)]。Self-Instruct [[36](https://arxiv.org/html/2411.14497v1#bib.bib36)] 和Alpaca [[33](https://arxiv.org/html/2411.14497v1#bib.bib33)] 利用GPT-3的生成能力扩展种子指令。生成的指令经过筛选，淘汰低质量指令，保留下来的指令用于微调模型，以增强模型响应指令的能力。Baize [[45](https://arxiv.org/html/2411.14497v1#bib.bib45)] 提出了一个自对话框架，使用来自流行问答网站的问题作为起始话题，然后让LLM与自己对话。CAMEL [[16](https://arxiv.org/html/2411.14497v1#bib.bib16)] 引入了一个角色扮演框架，在这个框架中，LLM在扮演“用户”或“助手”角色时讨论给定话题。UltraChat [[8](https://arxiv.org/html/2411.14497v1#bib.bib8)] 结合现实世界的命名实体与各种文本写作任务，生成多样且高质量的多轮对话供LLM使用。Lion [[13](https://arxiv.org/html/2411.14497v1#bib.bib13)] 引入了对抗蒸馏的概念，通过模仿-判别-生成阶段迭代生成数据，细化现有指令，并生成更复杂、更多样化的指令，扩展学生模型的能力。Evol-Instruct [[44](https://arxiv.org/html/2411.14497v1#bib.bib44)] 使用五个手动设计的提示，明确指导LLM将现有简单指令改写为更复杂的指令。使用Evol-Instruct训练的WizardLM模型，在MT-Bench [[54](https://arxiv.org/html/2411.14497v1#bib.bib54)]上排名靠前，凸显了数据质量在训练有效LLM中的重要性。

#### 指令数据选择

通过上述方法，使用LLM生成大规模指令微调数据集的成本不高。然而，对于指令微调语言模型而言，数据质量比数量更为关键。在这方面，ALPAGASUS [[5](https://arxiv.org/html/2411.14497v1#bib.bib5)] 通过利用ChatGPT评估指令数据的有效性。INSTAG [[24](https://arxiv.org/html/2411.14497v1#bib.bib24)] 自动为指令样本生成标签，并通过选择具有更多标签的子集来保持多样性。

![请参考说明文字](img/635bb25c642d6761c56edaabeb3e7b26.png)

图 1：Star-Agents 框架的示意图。步骤 1 旨在收集多样化的指令和响应，如附录[A.3](https://arxiv.org/html/2411.14497v1#A1.SS3 "A.3 Data Examples ‣ Appendix A Appendix ‣ Star-Agents: Automatic Data Optimization with LLM Agents for Instruction Tuning")所示。步骤 2 专注于从步骤 1 中收集的数据中选择高质量的定制数据。最后，步骤 3 旨在通过优化 Star-Agents 框架来提高数据生成过程的有效性和效率。

Cherry LLM [[20](https://arxiv.org/html/2411.14497v1#bib.bib20)] 首创了自我引导的方法，使用 IFD 指标来衡量 LLM 学习指令样本的难度。这使得可以选择那些显著提高训练效率的指令样本，而无需依赖外部模型。DEITA [[22](https://arxiv.org/html/2411.14497v1#bib.bib22)] 首先使用 ChatGPT 来评估样本的复杂性和质量，然后基于模型嵌入之间的距离评估样本的多样性，从而保证子集中的复杂性、质量和多样性。LIFT [[46](https://arxiv.org/html/2411.14497v1#bib.bib46)] 首先引导 GPT-4 生成具有挑战性的指令，以扩展数据分布，然后使用降维和行方差分析来选择具有代表性的高质量数据，其中 GPT-4 为每个指令生成质量评分。LESS [[43](https://arxiv.org/html/2411.14497v1#bib.bib43)] 首先存储数据集中样本的梯度特征，然后计算目标任务的少量样本与训练数据样本之间的相似度。根据计算得出的相似度评分，它选择那些梯度特征与目标任务样本最相似的训练样本作为微调实例。数据选择不仅提高了训练效率，还通过过滤低质量或有害数据，防止它们破坏模型性能 [[47](https://arxiv.org/html/2411.14497v1#bib.bib47)]。

## 3 星级代理

我们研究的目标是通过增强初始种子数据集 $S=(I_{i},R_{i})_{i=1}^{N}$，构建一个适合目标 LLM 的高质量定制复杂度数据集 $T$，该数据集由指令-响应对 $(I,R)$ 组成。

为此，我们引入了星代理框架，框架结构如图[1](https://arxiv.org/html/2411.14497v1#S2.F1 "图 1 ‣ 指令数据选择 ‣ 2 相关工作 ‣ 星代理：通过LLM代理进行指令调优的自动数据优化")所示，该框架分为三步。第一步利用一系列独立训练的先进LLM，这些模型通过动态交互生成一个多样化的数据候选集$D(S_{i})$，通过从$S_{i}$中派生的代理对进行采样，如第[3.1](https://arxiv.org/html/2411.14497v1#S3.SS1 "3.1 生成多样化数据 ‣ 3 星代理 ‣ 星代理：通过LLM代理进行指令调优的自动数据优化")节所述。随后，我们应用双模型评估策略$\pi(\cdot)$，从$D(S_{i})$中精确提取最合适的数据，以显著提升目标模型的表现。此过程在第[3.2](https://arxiv.org/html/2411.14497v1#S3.SS2 "3.2 通过双模型策略评估定制数据 ‣ 3 星代理 ‣ 星代理：通过LLM代理进行指令调优的自动数据优化")节中有详细描述。为了提高星代理框架在生成定制数据时的效果和效率，我们开发了一种星代理的进化策略，如第[3.3](https://arxiv.org/html/2411.14497v1#S3.SS3 "3.3 进化星代理 ‣ 3 星代理 ‣ 星代理：通过LLM代理进行指令调优的自动数据优化")节中所讨论的那样。在这三步之后，从种子数据集获得一个定制的高质量数据集$T$，该数据集的公式为：

|  | $T=\{\mathop{\arg\max_{d\in D(S_{i})}}\pi\left(D(S_{i})\right)~{}&#124;~{}i=1,2,% \cdots,N\}.$ |  | (1) |
| --- | --- | --- | --- |

### 3.1 生成多样化数据

为了改进指令微调模型，组建一个高质量且多样化的指令数据集是至关重要的[[22](https://arxiv.org/html/2411.14497v1#bib.bib22)]。传统方法通常使用单一的大型语言模型（LLM），例如ChatGPT，进行数据增强。相比之下，我们的方法采用多个LLM，以避免单一数据分布的单调性。这种多面策略还解决了使用单一模型在特定领域任务中质量下降的局限性和风险。为了解决这些挑战，我们提出使用Agent-Pair策略。

#### 代理对（Agent-Pair）。

利用一系列不同设置训练的大型语言模型（LLMs），可以促进生成多样化的响应。这种多样性对于合成一个高丰富度的数据集至关重要[[24](https://arxiv.org/html/2411.14497v1#bib.bib24)]。

星代理框架（Star-Agents framework）战略性地将不同的LLM配对，用以重写种子数据集中的指令，并生成新的响应以增加多样性。通过代理对$(A^{I}_{j},A^{R}_{k})$，可以生成新的指令数据，具体如下：

|  | $f_{j,k}\left(I_{i},R_{i}\right)=(A^{I}_{j}(I_{i}),A^{R}_{k}(R_{i})),$ |  | (2) |
| --- | --- | --- | --- |

其中$A^{I}$和$A^{R}$分别表示重写指令和响应指令的代理。

鉴于部署所有代理对的高成本，平衡成本和代理多样性的可行方案是从Star-Agents中采样一个代理对子集用于数据生成。公式[3](https://arxiv.org/html/2411.14497v1#S3.E3 "In Agent-Pair. ‣ 3.1 Generating Diverse Data ‣ 3 Star-Agents ‣ Star-Agents: Automatic Data Optimization with LLM Agents for Instruction Tuning")公式化了这个过程，其中$D$是通过对所有采样对$(A^{I}_{j},A^{R}_{k})$（指令代理$A^{I}_{j}$和响应代理$A^{R}_{k}$）执行$f$操作生成的收集数据集，采样概率为$p_{jk}$：

|  | $\displaystyle D\left(S_{i}\right)=\{f_{j_{1},k_{1}}\left(S_{i}\right),\cdots,f_{j_{M},k_{M}}\left(S_{i}\right)~{}&#124;~{}(j_{m},k_{m})\sim p_{jk},m=1,2,\cdots,M\},$ |  | (3) |
| --- | --- | --- | --- |

$M$是为单一种子样本采样的代理对数。采样概率$p_{jk}$初始化为均匀分布，并将在数据生成过程中使用[3.3](https://arxiv.org/html/2411.14497v1#S3.SS3 "3.3 Evolving Star Agents ‣ 3 Star-Agents ‣ Star-Agents: Automatic Data Optimization with LLM Agents for Instruction Tuning")小节中描述的方法更新。同时，一个存储高质量指令的指令记忆库也将进行更新。为了确保数据质量的下限，每次迭代将始终调用一组固定的代理对，称为基础代理对。

### 3.2 通过双模型策略评估定制数据

从多样化的数据集中识别并选择量身定制的数据对于提升模型性能至关重要，特别是因为低质量数据的存在会妨碍模型的功能。已经承认，长且复杂的、高难度的数据样本对指令调优过程有显著益处[[22](https://arxiv.org/html/2411.14497v1#bib.bib22)]。

然而，过于复杂的指令数据不一定能有助于模型性能。我们观察到，对于如图[2](https://arxiv.org/html/2411.14497v1#S3.F2 "Figure 2 ‣ 3.2 Evaluating Tailored Data via a Dual-model Strategy ‣ 3 Star-Agents ‣ Star-Agents: Automatic Data Optimization with LLM Agents for Instruction Tuning")所示的14M和70M参数的模型，尽管Evol-Instruct数据集比Alpaca数据集更具挑战性，但其结果却导致了模型性能的下降。这表明，复杂的示例可能会超出小型模型的能力，并且尽管对大模型使用复杂数据具有优势，但对小型模型来说，复杂数据可能会对性能造成负面影响。

![[无标题图像]](img/d7109f17c26338dc8573c7db2010a069.png)

图2：不同规模模型在Alpaca和Evol-Instruct数据集上的性能比较。Evol-Instruct数据集的任务比Alpaca数据集的任务更为复杂。

![[无标题图像]](img/097918b2bd407337b5d61185aa702024.png)

图3：双模型评估的示意图。小模型和大模型的IFD分数差距显著的数据将优先考虑。

#### 双模型评估。

为了解决上述问题，我们提出使用更大的模型来评估数据实例的难度，同时结合来自较小模型（目标LLM）的评估，因此称为双模型评估。受到Cherry LLM [[20](https://arxiv.org/html/2411.14497v1#bib.bib20)]的启发，我们使用IFD度量来衡量数据样本对目标模型所呈现的难度，计算公式如下

|  | $\mathrm{IFD}(I_{i},R_{i})=\frac{\exp\left(-\frac{1}{ | R_{i} | }\sum_{w\in R_{i}}% \log P(w | I_{i})\right)}{\exp\left(-\frac{1}{ | R_{i} | }\sum_{w\in R_{i}}\log P(w)% \right)}.$ |  | (4) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

我们假设对于同一样本，较强的模型会产生较小的IFD分数。当两个模型的IFD分数接近时，表示该样本要么过于简单，要么过于复杂，这对有效学习没有贡献。然而，当它们的IFD分数差异显著时，表明该数据对较小的模型来说足够复杂，但仍在较强模型的能力范围内。这是一种量身定制的复杂度，旨在促进学习。上述数据评估方法在图[3](https://arxiv.org/html/2411.14497v1#S3.F3 "图3 ‣ 3.2 通过双模型策略评估定制数据 ‣ 3 星级代理 ‣ 星级代理：通过LLM代理进行指令调优的自动数据优化")中进行了说明，并公式化为

|  | $\pi_{\mathrm{dual}}^{i}=\frac{\mathrm{IFD}_{\mathrm{small}}(I_{i},R_{i})-% \mathrm{IFD}_{\mathrm{large}}(I_{i},R_{i})}{\max\limits_{1\leq i\leq m}(% \mathrm{IFD}_{\mathrm{small}}(I_{i},R_{i})-\mathrm{IFD}_{\mathrm{large}}(I_{i}% ,R_{i}))}.$ |  | (5) |
| --- | --- | --- | --- |

噪声数据可能会被赋予较高的分数，因为双模型度量仅考虑相对复杂度，而忽略了生成质量。为了解决这个问题，我们使用LLM作为数据样本评分的裁判。这涉及将同一批次中由选定的代理对生成的多样化数据样本与由基础代理对生成的基础数据样本进行比较。可能的三种结果是：基础数据样本更好、多样化数据样本更好或平局，如附录[A.1](https://arxiv.org/html/2411.14497v1#A1.SS1 "A.1 提示示例 ‣ 附录 A 附录 ‣ 星级代理：通过LLM代理进行指令调优的自动数据优化")所示。这些结果会被定量地分配为质量分数，从而避免收集噪声指令样本：

|  | $\pi_{\mathrm{llm}}=\begin{cases}0,&\text{如果基础数据样本更好},\\ 1,&\text{如果生成的数据样本更好},\\ 0.5,&\text{如果平局}.\\ \end{cases}$ |  | (6) |
| --- | --- | --- | --- |

最终，来自LLM和双模型评估的评估分数会被结合起来，计算出最终的综合分数：

|  | $\pi=\pi_{\mathrm{llm}}\cdot\pi_{\mathrm{dual}}.$ |  | (7) |
| --- | --- | --- | --- |

该分数决定了数据的整体质量和适用性，用于增强模型的能力。最高得分的数据样本将被选入数据集$T$和指令记忆库，如[3.3节](https://arxiv.org/html/2411.14497v1#S3.SS3 "3.3 Evolving Star Agents ‣ 3 Star-Agents ‣ Star-Agents: Automatic Data Optimization with LLM Agents for Instruction Tuning")所述，确保选择的数据集最大限度地提高模型性能的潜力。

### 3.3 演化星级代理

如[3.1节](https://arxiv.org/html/2411.14497v1#S3.SS1 "3.1 Generating Diverse Data ‣ 3 Star-Agents ‣ Star-Agents: Automatic Data Optimization with LLM Agents for Instruction Tuning")所述，我们使用指令代理和响应代理的联合概率来调节每个代理对的调用。然而，由于每个LLM的能力和特长不同，用相同的概率对每个代理对进行采样并非最优。因此，我们使用[3.2节](https://arxiv.org/html/2411.14497v1#S3.SS2 "3.2 Evaluating Tailored Data via a Dual-model Strategy ‣ 3 Star-Agents ‣ Star-Agents: Automatic Data Optimization with LLM Agents for Instruction Tuning")中的分数来动态演化采样概率。此外，由于代理对的生成性能与任务相关，我们还提出了一个指令记忆库，以选择最适合特定任务的代理对。

#### 代理对采样演化。

[3.2节](https://arxiv.org/html/2411.14497v1#S3.SS2 "3.2 Evaluating Tailored Data via a Dual-model Strategy ‣ 3 Star-Agents ‣ Star-Agents: Automatic Data Optimization with LLM Agents for Instruction Tuning")介绍了分数$\pi$，该分数有效地估算了生成样本的质量。在每次迭代中，如果生成的样本质量较高，我们将增加所选代理对的采样概率，更新公式如下：

|  | $\displaystyle\begin{split}\tilde{p}_{jk}&=p_{jk}+\beta\cdot\pi(I_{i},R_{i}),\\ p_{jk}&\leftarrow\frac{\tilde{p}_{jk}}{\sum_{j,k}\tilde{p}_{jk}}.\end{split}$ |  | (8) |
| --- | --- | --- | --- |

对于第$j$个指令代理和第$k$个响应代理成功处理第$i$个数据样本后的更新采样概率，将在下一次迭代中使用，其中$\beta$表示演化速率。此公式根据代理对生成相关数据的效果调整采样概率。迭代更新确保随着合成过程的推进，选择更有效代理对的概率增加，而较低效的代理对则逐渐被淘汰。

#### 指令记忆库演化。

我们建立了一个指令记忆库，用于存储高质量的指令，旨在加速采样并将演化与任务数据相关联。在处理数据样本$(I_{i},R_{i})$时，我们在指令记忆库中查询$I_{i}$，根据嵌入相似性检索出前$n$个最接近的匹配项。与之关联的任务能力强的代理对被识别为与$I_{i}$相似的任务高度熟练，然后进行采样。我们从这个池中使用归一化概率采样$l$个代理对，以生成多样化的数据。此外，为了促进多样化数据集的创建，我们还将从剩余池中使用各自的概率额外采样$M-l$个代理对来协助数据合成。因此，生成了$M$个新样本，然后进行数据评估。随后，指令记忆库将通过融入定制的高质量数据持续演化，这些数据在第[3.2](https://arxiv.org/html/2411.14497v1#S3.SS2 "3.2 通过双模型策略评估定制数据 ‣ 3 Star-Agents ‣ Star-Agents: 使用LLM代理进行指令调优的自动数据优化")节中介绍的高分数中得到体现。

## 4 实验

我们进行了广泛的实验，以评估所提出的Star-Agents框架。在我们的实验中，使用了多种LLM和基准数据集，以确保评估的稳健性。

### 4.1 设置

#### 数据集。

与WizardLM[[44](https://arxiv.org/html/2411.14497v1#bib.bib44)]保持一致，我们采用了监督微调（SFT）数据集，该数据集被指定为Evol-Instruct数据集，包含70,000对指令-响应对。该数据集中的指令通过“深度演化”和“广度演化”方法进行了精细化，这些方法分别通过添加复杂细节或扩展整体范围来增强基本指令。为了确保数据的准确性，ChatGPT也作为生成器集成到精细化过程中。Evol-Instruct数据集中的指令数据质量已被验证为优越[[44](https://arxiv.org/html/2411.14497v1#bib.bib44), [25](https://arxiv.org/html/2411.14497v1#bib.bib25)]；因此，我们的研究继续利用这些精细化的指令。通过使用Star-Agents框架，我们的研究调用多个LLM生成多样化且高质量的响应。为了进一步丰富我们的对比分析，我们使用了Alpaca数据集[[32](https://arxiv.org/html/2411.14497v1#bib.bib32)]，该数据集包含52,000个遵循指令的样本。该数据集在自指令范式下开发，使用ChatGPT¹¹1https://chatgpt.com/而非text-davinci-003进行公平比较[[44](https://arxiv.org/html/2411.14497v1#bib.bib44)]。

表1：在Star-Agents中使用的典型LLM。

| 模型系列 | 模型大小 | 数据大小 | 方法 | 来源 |
| --- | --- | --- | --- | --- |
| Phi [[11](https://arxiv.org/html/2411.14497v1#bib.bib11)] | 2.7B | 1.4T | 预训练 | 微软 |
| ChatGLM [[51](https://arxiv.org/html/2411.14497v1#bib.bib51)] | 6B | 1T+ | SFT & RLHF | 智谱AI |
| Gemma [[34](https://arxiv.org/html/2411.14497v1#bib.bib34)] | 7B | 6T | SFT & RLHF | 谷歌 |
| Mistral [[12](https://arxiv.org/html/2411.14497v1#bib.bib12)] | 7B | - | SFT | Mistral |
| Qwen [[1](https://arxiv.org/html/2411.14497v1#bib.bib1)] | 14B | - | SFT & RLHF | 阿里巴巴 |
| ChatGPT | - | - | SFT & RLHF | OpenAI |

#### 模型。

针对在边缘设备上进行低成本推理的日益增长需求，我们的研究探索了规模为1B和7B参数的目标模型的能力。1B模型，特别是Pythia-1B [[2](https://arxiv.org/html/2411.14497v1#bib.bib2)]，是基于约3000亿个标记从Pile数据集派生而来进行训练的。7B模型，由Llama-2-7B [[35](https://arxiv.org/html/2411.14497v1#bib.bib35)]表示，经过2万亿个标记的广泛语料库训练。

在我们的实验中，我们集成了多种LLM模型作为生成器，具体细节见表[1](https://arxiv.org/html/2411.14497v1#S4.T1 "Table 1 ‣ Datasets. ‣ 4.1 Setups ‣ 4 Experiments ‣ Star-Agents: Automatic Data Optimization with LLM Agents for Instruction Tuning")。我们的假设是，不同开发团队的模型具有独特的能力，因其训练数据和策略的多样性，对相同提示产生丰富的响应。例如，Phi2 [[11](https://arxiv.org/html/2411.14497v1#bib.bib11)]使用了1.4万亿个精心策划的教材数据进行训练，且未经过人类反馈强化学习（RLHF），而Gemma [[34](https://arxiv.org/html/2411.14497v1#bib.bib34)]则使用了6万亿个主要来自英文网络文档、数学内容和代码的数据进行训练，之后通过SFT和RLHF进行微调。为了确保生成数据的多样性和质量，我们集合了不同团队训练的LLM模型，这些团队因其卓越表现而广受认可。为了促进生成不同难度层次的数据，我们使用的LLM模型参数范围从2.7B到14B，包括通过API访问的更大模型。为了与Evol-Instruct数据集进行公平比较，使用的最强模型是ChatGPT，它也被用于生成Evol-Instruct数据集中的响应。值得注意的是，ChatGPT还作为评估者来计算比较得分$\pi_{\mathrm{llm}}$。

#### 基准测试。

为了严格评估 AI 模型的指令跟随能力，我们使用了三个广泛使用的基准测试：MT-bench、Vicuna-bench 和 WizardLM 测试集。具体来说，MT-bench 和 Vicuna-bench 旨在测试模型在各种复杂认知任务中的能力，包括数学、推理、复杂格式处理以及通过多轮和单轮对话进行写作。相反，WizardLM 测试集扩展了评估范围，涵盖了技术、生物学和法律等多个领域。它还具有不同的难度级别，以便更细致地比较模型的性能差异。按照既定协议，我们使用了 Fast-Chat [[54](https://arxiv.org/html/2411.14497v1#bib.bib54)] 来评估模型性能，以 GPT-4 作为评判模型。

#### 基准线。

为了进行基准线比较，我们使用了 Pythia-1B 和 Llama-2-7B 两个模型，它们都是使用 Evol-Instruct 数据集进行训练的。此外，还参考了 Alpaca 数据集进行比较分析，同时将 IFD [[20](https://arxiv.org/html/2411.14497v1#bib.bib20)] 和随机选择作为数据选择方法的附加比较。

#### 实施细节。

我们使用 Adam 优化器对我们的模型（Pythia-1B 和 Llama-2-7B）进行了三轮微调，初始学习率为 $2\times 10^{-5}$，最大 token 数为 2048，批次大小为 64。对于 Star-Agents，我们使用了 10 对代理进行训练。

### 4.2 主要结果

表 2：不同模型在 Vicuna-bench、WizardLM 测试集和 MT-Bench 上的结果。

| 模型 | Vicuna-Bench | WizardLM 测试集 | MT-Bench | 平均值 |
| --- | --- | --- | --- | --- |
| 1B 模型 |
| --- |
| Pythia-1B [[2](https://arxiv.org/html/2411.14497v1#bib.bib2)] | 1.68 | 1.34 | 1.17 | 1.40 |
| OPT-1.3B [[52](https://arxiv.org/html/2411.14497v1#bib.bib52)] | 2.49 | 1.64 | 1.12 | 1.75 |
| Sheared-LLaMA-1.3B [[42](https://arxiv.org/html/2411.14497v1#bib.bib42)] | 2.73 | 1.86 | 1.59 | 2.06 |
| Pythia-1B-alpaca | 4.14 | 2.97 | 2.20 | 3.10 |
| Pythia-1B-evol_instruct | 5.07 | 3.55 | 2.56 | 3.73 |
| Pythia-1B-IFD [[20](https://arxiv.org/html/2411.14497v1#bib.bib20)] | 4.60 | 3.21 | 1.98 | 3.26 |
| Pythia-1B-Random | 5.13 | 3.39 | 2.35 | 3.62 |
| Pythia-1B-star_instruct | 5.93 | 3.90 | 2.69 | 4.17 |
| 7B 模型 |
| Llama-2-7B [[35](https://arxiv.org/html/2411.14497v1#bib.bib35)] | - | - | 3.95 | - |
| zephyr-beta-sft [[22](https://arxiv.org/html/2411.14497v1#bib.bib22)] | - | - | 5.32 | - |
| mpt-7B-chat [[22](https://arxiv.org/html/2411.14497v1#bib.bib22)] | - | - | 5.45 | - |
| XGen-7B-8k-Inst [[26](https://arxiv.org/html/2411.14497v1#bib.bib26)] | - | - | 5.55 | - |
| sRecycled-Wiz-7B-v2 [[17](https://arxiv.org/html/2411.14497v1#bib.bib17)] | - | - | 5.56 | - |
| Llama-2-7B-alpaca | 6.33 | 5.08 | 3.63 | 5.01 |
| Llama-2-7B-evol_instruct | 7.27 | 6.57 | 5.21 | 6.35 |
| Llama-2-7B-star_instruct | 8.24 | 6.87 | 5.74 | 6.95 |

#### GPT-4 自动评估

根据表[2](https://arxiv.org/html/2411.14497v1#S4.T2 "Table 2 ‣ 4.2 Main Results ‣ 4 Experiments ‣ Star-Agents: Automatic Data Optimization with LLM Agents for Instruction Tuning")中总结的研究结果，针对Pythia-1B和Llama-2-7B模型进行了全面的训练，使用了三种不同的数据集：Alpaca、Evol-Instruct以及优化过的Star Instruct数据集。后者是通过应用Star-Agents开发的，Star-Agents是Evol-Instruct数据集的衍生物。通过与其他当代最先进模型的对比分析，我们发现，采用Star Instruct数据集的SFT对齐模型在所有评估的模型系列中，一直优于几乎所有对齐的同行模型。

![参考标题](img/dcc4e176b75b2d092248745a2b8ca5e9.png)

(a) Vicuna-bench

![参考标题](img/a85052a14d49a189f806f368343b258f.png)

(b) MT-bench

图4：Llama-2-7B-star_instrcut在不同子任务上的详细得分雷达图，比较了主要基线（a）Vicuna-Bench和（b）MT-Bench。

值得注意的是，在1B规模下，使用Star Instruct数据集训练的模型表现出了显著的优势，超越了各种评估数据集的基线。特别是，与使用Evol-Instruct数据集训练的模型相比，采用Star Instruct数据集的模型在平均绝对提升约0.45，相当于性能提升约12%。此外，与使用Alpaca数据集训练的模型相比，我们的框架实现了1分的绝对提升，进一步验证了Star Instruct数据集特别适合Pythia-1B模型，显著提高了其操作效率。此外，在7B模型类别中，Llama-2-7B-star_instruct的表现优于在Evol-Instruct数据集上使用选择性反射调优训练的sRecycled-Wiz-7B-v2 [[17](https://arxiv.org/html/2411.14497v1#bib.bib17)]。图[4(a)](https://arxiv.org/html/2411.14497v1#S4.F4.sf1 "在图4 ‣ GPT-4自动评估 ‣ 4.2主要结果 ‣ 4实验 ‣ Star-Agents：使用LLM代理进行指令调优的自动数据优化")展示了Llama-2-7B-star_instruct在九项指标上的性能提升，特别是在数学、编程和费米问题解决中的显著进展，提升幅度可达40%。类似现象可以在图[4(b)](https://arxiv.org/html/2411.14497v1#S4.F4.sf2 "在图4 ‣ GPT-4自动评估 ‣ 4.2主要结果 ‣ 4实验 ‣ Star-Agents：使用LLM代理进行指令调优的自动数据优化")中观察到。此外，单轮和多轮对话的对比示例可以在附录[A.2](https://arxiv.org/html/2411.14497v1#A1.SS2 "A.2案例研究 ‣ 附录A 附录 ‣ Star-Agents：使用LLM代理进行指令调优的自动数据优化")中找到，LLMs在Open LLM排行榜上的表现可参见附录[A.4](https://arxiv.org/html/2411.14497v1#A1.SS4 "A.4 在Open LLM排行榜上的表现 ‣ 附录A 附录 ‣ Star-Agents：使用LLM代理进行指令调优的自动数据优化")。

表 3：不同组件的影响。

| 组件 | 平均得分 |
| --- | --- |
| 多样性 | 数据选择 | 演变 |
| ✓ | ✓ | ✓ | 4.17 |
| ✓ | ✓ | ✕ | 3.97 |
| ✓ | ✕ | ✕ | 3.62 |
| ✕ | ✕ | ✕ | 3.73 |

表 4：选择方法的影响。

| 模型 | 平均得分 |
| --- | --- |
| Pythia-1B-evol_instruct | 3.73 |
| Pythia-1B-IFD [[20](https://arxiv.org/html/2411.14497v1#bib.bib20)] | 3.26 |
| Pythia-1B-Random | 3.62 |

| Pythia-1B-star_instruct | 4.17 | ![参见说明](img/1a61b0a0090243fe9f1236e9b120fab4.png)

图 5：典型代理对的演变。

### 4.3 消融研究

#### 主要组件。

如表[3](https://arxiv.org/html/2411.14497v1#S4.T3 "Table 3 ‣ GPT-4 Automatic Evaluation ‣ 4.2 Main Results ‣ 4 Experiments ‣ Star-Agents: Automatic Data Optimization with LLM Agents for Instruction Tuning")所示，我们对Star-Agents框架中的三个主要组件进行了消融实验。结果表明，单独使用多样化数据集和随机采样的模型性能略低于基线。这是因为基线使用了由ChatGPT生成的数据，质量较高。而多样化数据集则来自多个不同来源，难以确保质量均匀。因此，随机采样可能引入低质量数据，导致模型性能下降。引入数据选择模块后，模型性能得到了恢复，这表明该模块能够有效选择适合模型的高质量数据。结合进化策略后，性能也得到了显著提升，证明了进化模块能够有效地从复杂的候选代理对中选择最合适的数据生成代理对。

#### 选择方法。

如表[4](https://arxiv.org/html/2411.14497v1#S4.T4 "Table 4 ‣ GPT-4 Automatic Evaluation ‣ 4.2 Main Results ‣ 4 Experiments ‣ Star-Agents: Automatic Data Optimization with LLM Agents for Instruction Tuning")所示，我们评估了一系列传统选择方法，包括随机选择和基于IFD[[20](https://arxiv.org/html/2411.14497v1#bib.bib20)]的策略。我们的双模型选择策略显著优于这些方法。与随机选择相比，我们的方法取得了显著的改进，在多种测试集上平均提高了超过0.5分。与IFD方法相比，我们的提升接近0.9分。这些结果有力地验证了我们的双模型选择策略的有效性，展示了它在使用多样化评估指标细化模型选择精度方面的优越性能。

#### 进化。

如图[5](https://arxiv.org/html/2411.14497v1#S4.F5 "图5 ‣ GPT-4自动评估 ‣ 4.2 主要结果 ‣ 4 实验 ‣ Star-Agents：用于指令微调的LLM代理自动数据优化")所示，我们分析了典型代理对在迭代进化过程中的采样概率曲线。最初，每对代理的采样概率约为10%。由于其强大的表现，Mistral-ChatGPT获得了持续的奖励，从而使得其采样概率逐渐增加。在大约70,000次迭代完成时，这一概率已经上升至30%。相比之下，Phi2-ChatGPT在同一时期内呈稳定下降趋势，其采样概率最终降至接近零，因为它逐步被淘汰。与此同时，ChatGLM3-ChatGPT则呈现出相对稳定的轨迹，尽管有轻微的下降趋势。进化轨迹的显著差异表明，不同生成器在不同任务上的适用性差异，这些差异都被我们的进化机制捕捉到。

## 5 结论

本文提出了Star-Agents框架，这是一种用于优化数据，使其对目标LLM（大语言模型）具有最佳挑战性的自动化系统。该框架已应用于开源的SFT数据集，并在多种模型系列上进行了训练，调整数据以提高其效能。我们的实证研究包括一系列指令微调实验，利用多个基准模型和特别优化的数据集，应用于像Pythia和LLaMA等知名模型。大量实验验证了我们方法的显著影响：优化后的定制数据集平均性能提高约12%，某些指标，尤其是与Fermi问题任务相关的，性能提升超过40%，这一结果得到了MT-bench、Vicuna bench和WizardLM测试集等基准的验证。这些发现强调了战略性多样化和定制化数据能够显著改善模型对齐和性能的前提。总之，我们的研究详细介绍了一种高度有效的自动化框架，显著增强了数据集的功能性，从而促进了更高效的模型对齐。

#### 局限性。

我们的方法在单轮指令数据集上取得了显著的性能提升。然而，它尚未在多轮对话中进行评估。因此，我们将多轮指令数据集的评估和领域特定指令数据集的验证留待未来的工作。

## 致谢

本研究部分得到了中国国家重点研发计划（资助编号：2023YFE0116400）的支持。我们感谢匿名审稿人提出的宝贵意见。

## 参考文献

+   [1] Jinze Bai, Shuai Bai, Yunfei Chu, Zeyu Cui, Kai Dang, Xiaodong Deng, Yang Fan, Wenbin Ge, Yu Han, Fei Huang 等人。Qwen 技术报告。arXiv 预印本 arXiv:2309.16609, 2023。

+   [2] Stella Biderman, Hailey Schoelkopf, Quentin Gregory Anthony, Herbie Bradley, Kyle O’Brien, Eric Hallahan, Mohammad Aflah Khan, Shivanshu Purohit, USVSN Sai Prashanth, Edward Raff 等人。Pythia：一个用于分析大语言模型的工具集，涵盖训练和扩展。在国际机器学习大会，页码 2397–2430。PMLR, 2023。

+   [3] Yupeng Chang, Xu Wang, Jindong Wang, Yuan Wu, Linyi Yang, Kaijie Zhu, Hao Chen, Xiaoyuan Yi, Cunxiang Wang, Yidong Wang, Wei Ye, Yue Zhang, Yi Chang, Philip S. Yu, Qiang Yang 和 Xing Xie。关于大语言模型评估的调查，2023年。

+   [4] Justin Chih-Yao Chen, Swarnadeep Saha, Elias Stengel-Eskin 和 Mohit Bansal。Magdi：结构化蒸馏多智能体交互图改善小型语言模型的推理能力。arXiv 预印本 arXiv:2402.01620, 2024。

+   [5] Lichang Chen, Shiyang Li, Jun Yan, Hai Wang, Kalpa Gunaratna, Vikas Yadav, Zheng Tang, Vijay Srinivasan, Tianyi Zhou, Heng Huang 等人。Alpagasus：用更少的数据训练更好的 alpaca。arXiv 预印本 arXiv:2307.08701, 2023。

+   [6] Wei-Lin Chiang, Zhuohan Li, Zi Lin, Ying Sheng, Zhanghao Wu, Hao Zhang, Lianmin Zheng, Siyuan Zhuang, Yonghao Zhuang, Joseph E Gonzalez 等人。Vicuna：一个开源聊天机器人，以 90%* ChatGPT 质量给 GPT-4 留下深刻印象。见 https://vicuna.lmsys.org（访问日期：2023年4月14日），2(3):6，2023年。

+   [7] Mike Conover, Matt Hayes, Ankit Mathur, Jianwei Xie, Jun Wan, Sam Shah, Ali Ghodsi, Patrick Wendell, Matei Zaharia 和 Reynold Xin。Free dolly：介绍世界上首个真正开源的指令调优 LLM。Databricks 公司博客，2023年。

+   [8] Ning Ding, Yulin Chen, Bokai Xu, Yujia Qin, Zhi Zheng, Shengding Hu, Zhiyuan Liu, Maosong Sun 和 Bowen Zhou。通过扩展高质量教学对话增强聊天语言模型。arXiv 预印本 arXiv:2305.14233, 2023。

+   [9] Zhengxiao Du, Yujie Qian, Xiao Liu, Ming Ding, Jiezhong Qiu, Zhilin Yang 和 Jie Tang。GLM：具有自回归空白填充的通用语言模型预训练。在第60届计算语言学协会年会会议论文集（卷1：长篇论文），页码 320–335，爱尔兰都柏林，2022年5月。计算语言学协会。

+   [10] Zishan Guo, Renren Jin, Chuang Liu, Yufei Huang, Dan Shi, Linhao Yu, Yan Liu, Jiaxuan Li, Bojian Xiong, Deyi Xiong 等人。评估大语言模型：一项全面的调查。arXiv 预印本 arXiv:2310.19736, 2023。

+   [11] Mojan Javaheripi, Sébastien Bubeck, Marah Abdin, Jyoti Aneja, Sebastien Bubeck, Caio César Teodoro Mendes, Weizhu Chen, Allie Del Giorno, Ronen Eldan, Sivakanth Gopi 等人。Phi-2：小型语言模型的惊人威力。微软研究博客，2023年。

+   [12] Albert Q Jiang, Alexandre Sablayrolles, Arthur Mensch, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Florian Bressand, Gianna Lengyel, Guillaume Lample, Lucile Saulnier, 等. Mistral 7b. arXiv预印本 arXiv:2310.06825, 2023.

+   [13] Yuxin Jiang, Chunkit Chan, Mingyang Chen, 和 Wei Wang. Lion：封闭源大型语言模型的对抗性蒸馏. arXiv预印本 arXiv:2305.12870, 2023.

+   [14] Daniel Khashabi, Sewon Min, Tushar Khot, Ashish Sabharwal, Oyvind Tafjord, Peter Clark, 和 Hannaneh Hajishirzi. UNIFIEDQA：使用单一QA系统跨越格式边界. 在《计算语言学协会发现会议：EMNLP 2020》中，第1896-1907页，在线，2020年11月。计算语言学协会。

+   [15] Andreas Köpf, Yannic Kilcher, Dimitri von Rütte, Sotiris Anagnostidis, Zhi Rui Tam, Keith Stevens, Abdullah Barhoum, Duc Nguyen, Oliver Stanley, Richárd Nagyfi, 等. 开放对话——民主化大规模语言模型对齐. 神经信息处理系统进展, 36, 2024.

+   [16] Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, 和 Bernard Ghanem. Camel：用于“大规模语言模型社会”的交流代理. 2023.

+   [17] Ming Li, Lichang Chen, Jiuhai Chen, Shwai He, Jiuxiang Gu, 和 Tianyi Zhou. 选择性反射调优：学生选择数据回收用于LLM指令调优. arXiv预印本 arXiv:2402.10110, 2024.

+   [18] Ming Li, Lichang Chen, Jiuhai Chen, Shwai He, Heng Huang, Jiuxiang Gu, 和 Tianyi Zhou. 反射调优：数据回收提升LLM指令调优. ArXiv, abs/2310.11716, 2023.

+   [19] Ming Li, Yong Zhang, Shwai He, Zhitao Li, Hongyu Zhao, Jianzong Wang, Ning Cheng, 和 Tianyi Zhou. 超过滤：用于快速指令调优的弱到强数据过滤, 2024.

+   [20] Ming Li, Yong Zhang, Zhitao Li, Jiuhai Chen, Lichang Chen, Ning Cheng, Jianzong Wang, Tianyi Zhou, 和 Jing Xiao. 从数量到质量：通过自引导数据选择提升LLM性能进行指令调优. arXiv预印本 arXiv:2308.12032, 2023.

+   [21] Chuang Liu, Linhao Yu, Jiaxuan Li, Renren Jin, Yufei Huang, Ling Shi, Junhui Zhang, Xinmeng Ji, Tingting Cui, Tao Liu, 等. Openeval：基准测试中文LLM的能力、对齐性和安全性. arXiv预印本 arXiv:2403.12316, 2024.

+   [22] Wei Liu, Weihao Zeng, Keqing He, Yong Jiang, 和 Junxian He. 什么样的数据适合对齐？指令调优中自动数据选择的全面研究. arXiv预印本 arXiv:2312.15685, 2023.

+   [23] S. Longpre, Le Hou, Tu Vu, Albert Webson, Hyung Won Chung, Yi Tay, Denny Zhou, Quoc V. Le, Barret Zoph, Jason Wei, 和 Adam Roberts. Flan集合：为有效的指令调优设计数据和方法. ArXiv, abs/2301.13688, 2023.

+   [24] 盧克明、袁洪毅、袁政、林潤基、林俊揚、譚傳琪、周暢、周景仁。# instag: 用于分析大型语言模型监督微调的指令标记。载于《第十二届国际学习表征大会》，2023年。

+   [25] 胡姆扎·纳维德、阿萨德·乌拉·汗、邱石、穆罕默德·萨基卜、赛义德·安瓦尔、穆罕默德·乌斯曼、尼克·巴恩斯、阿吉马尔·米安。大型语言模型的综合概述。arXiv预印本arXiv:2307.06435，2023年。

+   [26] 埃里克·奈坎普、谢天、林浩明、龚邦、谢聪颖、陈兴、杰西·维格、塞米·雅武兹、菲利普·拉班、本·克劳斯等。Xgen-7b技术报告。arXiv预印本arXiv:2309.03450，2023年。

+   [27] 沈天浩、金仁仁、黄玉飞、刘闯、董伟龙、郭子山、吴新伟、刘燕、熊德毅。大型语言模型对齐：一项调查。arXiv预印本arXiv:2309.15025，2023年。

+   [28] 沈天浩、李顺、屠全、熊德毅。Roleeval：一个针对大型语言模型的双语角色评估基准。arXiv预印本arXiv:2312.16132，2023年。

+   [29] 舒曼丽、王炯霄、朱晨、乔纳斯·盖平、肖超伟、汤姆·戈尔德斯坦。关于指令调优的可利用性，2023年。

+   [30] 孙浩然、金仁仁、徐少扬、潘磊宇、崔孟龙、段江存、雷一坤、杨磊、施凌、肖爵思等。Fuxitranyu：一个使用平衡数据训练的多语言大型语言模型。arXiv预印本arXiv:2408.06273，2024年。

+   [31] 唐业辉、刘方程、倪云生、田玉川、白哲元、胡一琪、刘思超、瑞尚岭、韩凯、王云赫。重新思考微型语言模型的优化和架构。arXiv预印本arXiv:2402.02791，2024年。

+   [32] 罗汉·塔奥里、伊沙恩·古尔贾尼、张天意、杨·杜布瓦、李学辰、卡洛斯·格斯特林、皮尔西·梁、田住典·B·哈希莫托。斯坦福Alpaca：一个指令跟随型的Llama模型。[https://github.com/tatsu-lab/stanford_alpaca](https://github.com/tatsu-lab/stanford_alpaca)，2023年。

+   [33] 罗汉·塔奥里、伊沙恩·古尔贾尼、张天意、杨·杜布瓦、李学辰、卡洛斯·格斯特林、皮尔西·梁、田住典·B·哈希莫托。斯坦福Alpaca：一个指令跟随型的Llama模型，2023年。

+   [34] Gemma团队、托马斯·梅斯纳、卡西迪·哈丁、罗伯特·达达希、苏里亚·布帕塔贾鲁、什雷亚·帕塔克、洛朗·西弗、莫甘·里维耶、米赫尔·桑贾伊·凯尔、朱丽叶·洛夫等。Gemma：基于双子座研究与技术的开放模型。arXiv预印本arXiv:2403.08295，2024年。

+   [35] 雨果·图弗龙、路易斯·马丁、凯文·斯通、彼得·阿尔伯特、阿姆贾德·阿尔梅哈伊里、雅斯敏·巴巴伊、尼古拉·巴什利科夫、苏米娅·巴特拉、普拉吉瓦尔·巴尔加瓦、什鲁蒂·博萨尔等。Llama 2：开放基础和微调的聊天模型。arXiv预印本arXiv:2307.09288，2023年。

+   [36] 王怡中、叶干赫·科尔迪、斯瓦鲁普·米什拉、阿丽莎·刘、诺亚·A·史密斯、丹尼尔·哈沙比、汉纳赫·哈吉什尔兹。Self-instruct：通过自生成指令对齐语言模型。arXiv预印本arXiv:2212.10560，2022年。

+   [37] 王一中，Kordi Yeganeh，Swaroop Mishra，Alisa Liu，Noah A. Smith，Daniel Khashabi，和Hannaneh Hajishirzi。Self-instruct：将语言模型与自生成的指令对齐。在第61届计算语言学协会年会（第一卷：长篇论文）上发表，页面13484–13508，加拿大多伦多，2023年7月。计算语言学协会。

+   [38] 王一中，Swaroop Mishra，Pegah Alipoormolabashi，Yeganeh Kordi，Amirreza Mirzaei，Atharva Naik，Arjun Ashok，Arut Selvan Dhanasekaran，Anjana Arunkumar，David Stap，Eshaan Pathak，Giannis Karamanolakis，Haizhi Lai，Ishan Purohit，Ishani Mondal，Jacob Anderson，Kirby Kuznia，Krima Doshi，Kuntal Kumar Pal，Maitreya Patel，Mehrad Moradshahi，Mihir Parmar，Mirali Purohit，Neeraj Varshney，Phani Rohitha Kaza，Pulkit Verma，Ravsehaj Singh Puri，Rushang Karia，Savan Doshi，Shailaja Keyur Sampat，Siddhartha Mishra，Sujan Reddy A，Sumanta Patro，Tanay Dixit，和Xudong Shen。超自然指令：通过声明性指令对1600多个NLP任务进行泛化。在2022年阿布扎比自然语言处理实证方法会议（EMNLP）上发表，页面5085–5109，阿联酋阿布扎比，2022年12月。计算语言学协会。

+   [39] 王云鹤，陈汉廷，唐烨辉，郭天宇，韩凯，聂颖，王旭涛，胡海林，白哲远，王云等人。Pangu-$pi$：通过非线性补偿增强语言模型架构。arXiv预印本arXiv:2312.17276，2023年。

+   [40] Jason Wei，Maarten Bosma，Vincent Zhao，Kelvin Guu，Adams Wei Yu，Brian Lester，Nan Du，Andrew M. Dai，和Quoc V Le。微调语言模型是零样本学习者。在2022年学习表征国际会议（ICLR）上发表。

+   [41] Jason Wei，Maarten Bosma，Vincent Y Zhao，Kelvin Guu，Adams Wei Yu，Brian Lester，Nan Du，Andrew M Dai，和Quoc V Le。微调语言模型是零样本学习者。arXiv预印本arXiv:2109.01652，2021年。

+   [42] 夏孟洲，高天宇，曾志远，和Danqi Chen。Sheared llama：通过结构化剪枝加速语言模型预训练。arXiv预印本arXiv:2310.06694，2023年。

+   [43] 夏孟洲，Sadhika Malladi，Suchin Gururangan，Sanjeev Arora，和Danqi Chen。Less：为目标指令调优选择有影响的数据。arXiv预印本arXiv:2402.04333，2024年。

+   [44] 徐灿，孙清峰，郑凯，耿修博，赵蒲，冯家湛，陶崇扬，和蒋大欣。Wizardlm：赋能大型语言模型执行复杂指令。arXiv预印本arXiv:2304.12244，2023年。

+   [45] 徐灿文，郭大亚，段楠，和Julian McAuley。Baize：一个开源的聊天模型，基于自聊天数据进行参数高效调优。arXiv预印本arXiv:2304.01196，2023年。

+   [46] 许扬，姚永强，黄玉凡，齐孟南，王茂全，顾斌，和Neel Sundaresan。重新思考指令质量：Lift是你所需要的，2023年。

+   [47] 闫俊, 维卡斯·亚达夫, 李世阳, 陈立昌, 唐正, 王海, 维贾伊·斯里尼瓦桑, 任翔, 金洪霞. 利用虚拟提示注入对指令调优的大型语言模型进行后门攻击. 载于《NeurIPS 2023年深度学习后门攻击研讨会——好的、坏的与丑陋的》，2023年。

+   [48] 闫俊, 维卡斯·亚达夫, 李世阳, 陈立昌, 唐正, 王海, 维贾伊·斯里尼瓦桑, 任翔, 金洪霞. 对指令调优的大型语言模型进行虚拟提示注入，2023年。

+   [49] 杨晶峰, 金鸿烨, 唐瑞翔, 韩晓天, 冯启章, 姜浩明, 殷冰, 胡霞. 实践中利用LLM的力量: 关于ChatGPT及其延伸的调查，2023年。

+   [50] 叶勤远, 林钰晨, 任翔. CrossFit: 一项针对自然语言处理任务跨任务泛化的少样本学习挑战. 载于《2021年自然语言处理实证方法会议论文集》，第7163–7189页，在线与多米尼加共和国蓬塔卡纳，2021年11月。计算语言学协会出版。

+   [51] 曾奕翰, 刘晓, 杜郑晓, 王子翰, 赖涵宇, 丁铭, 杨卓依, 许一凡, 郑文迪, 夏晓, 等. Glm-130b: 一种开放的双语预训练模型. arXiv预印本 arXiv:2210.02414, 2022年。

+   [52] 张苏珊, 斯蒂芬·罗勒, 纳曼·戈亚尔, 米凯尔·阿尔特谢, 陈墨雅, 陈烁慧, 克里斯托弗·德万, 莫娜·迪亚布, 李献, 林晓维, 等. Opt: 开放预训练的变换器语言模型. arXiv预印本 arXiv:2205.01068, 2022年。

+   [53] 赵昕, 周昆, 李俊毅, 唐天一, 王晓磊, 侯宇鹏, 闵英乾, 张贝晨, 张俊杰, 董子灿, 杜一凡, 杨辰, 陈宇硕, 陈志鹏, 姜金浩, 任瑞扬, 李一帆, 唐欣瑜, 刘紫康, 刘佩宇, 聂剑云, 温基荣. 大型语言模型调查, 2023年。

+   [54] 郑连敏, 姜伟林, 盛颖, 庄思源, 吴张浩, 庄永浩, 林子, 李卓涵, 李大成, 辛埃里克, 等. 使用mt-bench和聊天机器人竞技场评估LLM作为法官的表现. 《神经信息处理系统进展》，第36期，2024年。

## 附录A 附录

### A.1 提示示例

根据Fast-Chat [[54](https://arxiv.org/html/2411.14497v1#bib.bib54)]，数据选择过程中使用的提示如表[5](https://arxiv.org/html/2411.14497v1#A1.T5 "表5 ‣ A.1 提示示例 ‣ 附录A ‣ Star-Agents: 使用LLM代理自动优化数据以进行指令调优")所列。

表5: LLM数据选择的提示。

| 系统提示：请充当一名公正的评委，评估三位AI助手对以下用户问题的回答质量。你应该选择那位最能遵循用户指令并最好回答用户问题的助手。你的评估应考虑其回答的有用性、相关性、准确性、深度、创造性以及细节水平。在开始评估之前，先比较两份回答并提供简短的解释。避免偏袒任何一方，确保呈现的顺序不影响你的判断。不要让回答的长度影响你的评估，也不要偏爱某些助手的名字。尽量做到客观。在提供解释后，按照严格的格式输出最终裁定：“[[A]]”如果助手A最佳，“[[B]]”如果助手B最佳，“[[C]]”表示平局。 |
| --- |

| 提示模板：[用户问题]

<question>

[助手A回答的开始]

<answer_a>

[助手A回答的结束]

[助手B回答的开始]

<answer_b>

[助手B回答的结束]

[最终裁定]：

### A.2 案例研究

单轮对话和多轮对话的示例见表[6](https://arxiv.org/html/2411.14497v1#A1.T6 "表6 ‣ A.2 案例研究 ‣ 附录A 附录 ‣ Star-Agents：使用LLM代理进行指令调优的自动数据优化")和表[7](https://arxiv.org/html/2411.14497v1#A1.T7 "表7 ‣ A.2 案例研究 ‣ 附录A 附录 ‣ Star-Agents：使用LLM代理进行指令调优的自动数据优化")。

表6：单轮对话示例。

| 指令：一间技术初创公司在第一年投入$8000进行软件开发，在第二年再投入其一半的金额进行软件开发。那么，该公司在两年内软件开发的总投资额是多少？ |
| --- |
| Llama-2-7B-evol_instruct：该初创公司在两年内投入的软件开发总额为$12,000。 | Llama-2-7B-star_instruct：为了计算该初创公司在两年内投入的软件开发总额，我们需要先计算每年的软件开发投资总额，然后将它们加在一起。在第一年，该初创公司在软件开发上投入了$8000。在第二年，该公司投入了一半的金额，即$4000。因此，两年内软件开发的总投资为：$8000（第一年）+ $4000（第二年）= $12000。所以，该初创公司在两年内投入的软件开发总额是$12000。 |

表7：多轮对话示例。

| Instruction：作为一名数学老师，我将提供一些数学方程或概念，你的任务是用通俗易懂的语言来解释它们。这可以包括提供逐步解决问题的指南、用日常生活中的例子展示各种技巧或建议进一步学习的在线资源。我的第一个问题是“我需要帮助理解概率是如何工作的”。 |
| --- |
| Llama-2-7B-evol_instruct：当然，我很乐意帮助你！概率是某个事件发生的可能性，它是数学和科学中的一个基本概念。简单来说，概率是某个事件在所有可能结果中发生的机会。例如，如果你抛一枚硬币，得到正面的概率是2分之一，也就是50%。概率可以用来进行预测和计算风险，在金融、统计学和数据分析等领域非常重要。计算概率时，你可以根据所解决问题的类型使用不同的公式。一种常见的方法是使用称为概率质量函数的公式，它告诉你某个结果发生的可能性。另一种方法是使用概率密度函数，它告诉你某个结果范围内的发生可能性。网上有许多资源可以帮助你了解更多关于概率的知识，包括视频教程、互动模拟和练习题。一些受欢迎的学习概率的网站包括可汗学院、Mathway和Coursera。希望这些对你有帮助！ | Llama-2-7B-star_instruct：当然，我很乐意帮助你理解概率！概率是衡量事件发生的可能性的指标。它通常表示为介于0和1之间的一个数字，其中0表示事件非常不可能发生，1表示事件非常可能发生。计算概率时，你需要知道所有可能结果的总数以及你感兴趣的事件的有利结果的数量。例如，如果你抛一枚公平的硬币，可能的结果有两个（正面或反面），得到正面的概率是1/2或0.5，因为在两个可能的结果中，只有一种方式可以得到正面。在日常生活中，概率可以用来计算某些事件发生的可能性，比如某一天下雨的几率或在考试中得到某个成绩的概率。它也用于决策情境中，比如计算某项投资的风险或商业投资成功的可能性。网上有很多资源可以帮助你学习更多关于概率的知识，比如视频教程、练习题和互动模拟。一些受欢迎的网站包括可汗学院、Coursera和edX，它们提供关于概率和其他数学概念的免费课程和资源。 |
| Instruction：黎曼几何和欧几里得几何有什么区别？ |
| Llama-2-7B-evol_instruct:黎曼几何和欧几里得几何是两种不同类型的几何学。在欧几里得几何中，两点之间的距离总是直线，三角形的角度总和为180度。而在黎曼几何中，两点之间的距离不一定是直线，三角形的角度总和可能大于180度。两者之间的另一个关键区别是它们处理曲面的方法。在欧几里得几何中，平面是一个具有平行边的矩形，而在黎曼几何中，表面可以是任何形式的曲面。这意味着黎曼几何更适合描述宇宙的几何，因为它能够处理时空的弯曲。总体而言，黎曼几何是一种更复杂、更高级的几何形式，并且在物理学和工程学等领域有许多实际应用。 | Llama-2-7B-star_instruct: 很好的问题！黎曼几何和欧几里得几何是两种不同类型的几何学，用于描述和分析空间的性质。它们之间的主要区别如下：1\. 度量：在欧几里得几何中，两点之间的距离由毕达哥拉斯定理定义，该定理指出，直角三角形两边的平方和等于斜边的平方。在黎曼几何中，两点之间的距离由度量定义，度量是一个描述两点之间最短路径弯曲程度的函数。2\. 曲率：欧几里得几何处理的是平面空间，其中地球表面的曲率可以忽略不计。相反，黎曼几何处理的是弯曲空间，如地球表面，其中曲率在确定空间性质中起着重要作用。3\. 大地线：在欧几里得几何中，两点之间的最短路径是直线。在黎曼几何中，两点之间的最短路径称为大地线，它可能是弯曲的，并依赖于度量。4\. 张量场：黎曼几何涉及使用张量场，张量场是描述空间几何的数学对象。这些场用于计算空间的曲率以及物体在其中运动的行为。总体而言，黎曼几何和欧几里得几何是理解空间性质的两种不同方法，其中黎曼几何更适用于弯曲空间，而欧几里得几何更适用于平面空间。 |

### A.3 数据示例

数据示例见表格[8](https://arxiv.org/html/2411.14497v1#A1.T8 "表格 8 ‣ A.3 数据示例 ‣ 附录 A 附录 ‣ Star-Agents: 使用LLM代理进行指令调整的自动数据优化")。

表格 8：多样化数据示例。

| 说明：给定一些二分类数据，确定一个适合此任务的机器学习模型。 |
| --- |

| 演示说明：适用于二分类任务的机器学习模型包括：1) 逻辑回归：这是一种统计方法，使用逻辑函数来建模二分类的因变量。它是一个相对简单且易于解释的模型，适用于线性可分的数据集。

2) 支持向量机（SVM）：它是一个常用的分类器，用于二分类任务。SVM能够处理高维数据，并且在数据集不是线性可分的情况下也能表现良好。

3) 随机森林：这是一种集成学习方法，通过在训练时构建大量的决策树，并输出各个树输出类别的众数来进行工作。

4) 神经网络：它是一种广泛用于二分类任务的机器学习模型。神经网络能够处理变量之间的非线性关系，并且能够学习数据中的复杂模式。

模型的选择取决于数据的具体特征和项目的具体要求。 |

| Agent-Pair A：有多种机器学习模型可以应用于二分类任务，具体选择哪种模型取决于数据的性质和任务的要求。以下是一些常见的模型：1. 逻辑回归：这是一种简单且有效的模型，适用于二分类问题。它通过估计正类的概率并使用该概率进行预测来工作。

2. 决策树：决策树是一个流行的机器学习模型，可以处理类别特征和数值特征。它通过递归地根据输入特征的值将数据划分为子集来进行工作。

3. 随机森林：随机森林是一种集成学习方法，通过结合多个决策树来提高模型的准确性。它通过创建一组决策树，并对它们的预测结果取平均来进行工作。

4. 支持向量机（SVM）：SVM是一个强大的模型，可以处理高维数据。它通过在特征空间中找到最大分离类的超平面来进行工作。

5. 神经网络：神经网络是一组可以从数据中学习和分类的算法。它们能够处理输入特征与输出标签之间复杂且非线性的关系。

1模型的选择取决于数据集的大小和复杂度、特征类型、计算资源以及所需的准确性等多种因素。 |

| Agent-Pair B：对于二分类任务，根据数据集的大小、问题的复杂性和特征的可用性，有多种机器学习模型适用。以下是一些可以应用的流行模型：1\. 逻辑回归：一种简单且易于解释的模型，适用于中小型数据集。它是线性的，并且能够处理分类和连续特征。|

2\. 决策树和随机森林：这些模型易于解释，能够处理数值型和分类数据。随机森林是一种集成方法，它结合了多个决策树，从而减少了过拟合。

3\. 支持向量机（SVM）：对于高维数据有效，并且可以通过核函数处理非线性关系。SVM对异常值具有鲁棒性，既可以用于线性分类，也可以用于非线性分类。

4\. 朴素贝叶斯：一种假设特征之间相互独立的概率模型。它速度快，易于实现，适用于文本分类或特征不高度相关的情况。

5\. K近邻（KNN）：一种简单的模型，通过最近邻居的多数类别来进行分类。它是非参数的，适用于小型数据集。

6\. 神经网络（例如前馈神经网络、卷积神经网络或递归神经网络）：深度学习模型，能够捕捉大数据集中的复杂模式。它们非常强大，但需要更多的数据和计算资源。

7\. 梯度提升机（GBM）或XGBoost：集成方法，通过迭代构建弱学习器（例如决策树）来创建强分类器。它们通常用于结构化数据，并能够处理高维度。

8\. AdaBoost：另一种增强算法，通过结合弱学习器来创建强分类器。在处理不平衡数据集时特别有用。

9\. 神经网络集成（例如，堆叠、混合）：结合多个模型的预测可以提高性能和鲁棒性。

选择最佳模型时，进行探索性数据分析、将数据划分为训练集和测试集，并使用适当的指标（例如准确率、精确率、召回率、F1分数或不平衡数据集的AUC-ROC）评估模型是至关重要的。交叉验证可以帮助选择最佳模型和超参数。

| Agent-Pair C：适用于此任务的机器学习模型是随机森林分类器。该模型在处理分类数据和数值数据时效果良好，并且能够处理高维度的大数据集。 |
| --- |
| Agent-Pair D：一种适用于二分类任务的机器学习模型是逻辑回归模型。逻辑回归常用于二分类任务，因为它提供了一个实例属于特定类别的概率。这是一种简单而高效的算法，适用于线性可分的数据。其他常用的二分类机器学习模型包括支持向量机（SVM）、决策树、随机森林、梯度提升机（GBM）和神经网络。模型的选择取决于数据的复杂性、数据集的大小、可解释性要求以及可用的计算资源。 |

### A.4 在公开LLM排行榜上的表现

公开LLM排行榜上的表现如表[9](https://arxiv.org/html/2411.14497v1#A1.T9 "Table 9 ‣ A.4 Performance on Open LLM Leaderboards ‣ Appendix A Appendix ‣ Star-Agents: Automatic Data Optimization with LLM Agents for Instruction Tuning")所示。

| 模型 | ARC | HellaSwag | MMLU | TruthfulQA | 平均值 |
| --- | --- | --- | --- | --- | --- |
| Wizardlm | 51.60 | 77.74 | 42.74 | 45.75 | 54.18 |
| Llama-2-7B-evol_instruct | 51.88 | 76.70 | 45.76 | 46.10 | 55.11 |
| Llama-2-7B-star_instruct | 54.44 | 77.64 | 46.94 | 46.13 | 56.29 |

表9：公开LLM排行榜上的表现。

### A.5 计算成本。

我们提出的方法的计算开销主要取决于所使用的各个LLM的推理计算负载：

+   •

    Qwen-14B：在推理过程中，序列长度为256个token时，计算负载大约是$4\times 10^{12}$次乘加运算（MACs）。

+   •

    Phi-2-2.7B：对于相同的序列长度，推理计算负载大约是$7\times 10^{11}$ MACs。

+   •

    ChatGPT：由于ChatGPT是一个专有模型，我们没有其计算需求的详细信息。

尽管如此，为了估算总的计算成本，我们可以大致估算整体计算成本。假设一个涉及多个LLM（例如10个LLM）和一个大型数据集（例如70,000个样本）的迭代过程，那么在不使用我们的框架的情况下，总的计算量可以粗略估算为：

+   •

    $4\times 10^{12}$ FLOPs (Qwen-14B) $\times$ 10 LLMs $\times$ 70,000 samples = $2.8\times 10^{18}$ MACs

而当使用Agent-Pairs采样和指令记忆库时，10个LLM中有5个LLM用于生成数据，因此，总的计算量可以显著减少，并且大致估算为：

+   •

    $4\times 10^{12}$ FLOPs (Qwen-14B) $\times$ 5 LLMs $\times$ 70,000 samples = $1.4\times 10^{18}$ MACs
