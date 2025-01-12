<!--yml

类别：未分类

日期：2025-01-11 12:58:57

-->

# 基于自我对话的LLM任务导向对话代理的引导

> 来源：[https://arxiv.org/html/2401.05033/](https://arxiv.org/html/2401.05033/)

\pdfcolInitStack

tcb@breakable

Dennis Ulmer^( , \faCompass, \faFlag)    Elman Mansimov^(\faAmazon)    Kaixiang Lin^(\faAmazon)    Justin Sun^(\faAmazon)

Xibin Gao^(\faAmazon)    Yi Zhang^(\faAmazon)

^(\faCompass) 哥本哈根IT大学 ^(\faFlag) 人工智能先锋中心

^(\faAmazon) AWS AI实验室

mansimov@amazon.com    这项工作是在亚马逊的实习期间进行的。

###### 摘要

大型语言模型（LLMs）是强大的对话代理，但将其专业化以执行特定功能可能具有挑战性。指令调优，即基于人类生成的指令和示例响应来调优模型（Ouyang等， [2022](#bib.bib26)），已被证明是一种有效的方法，但这需要大量的数据样本，这些样本 a) 可能不可用，或者 b) 生成成本高昂。此外，当目标是让LLM在对话中遵循特定工作流，而不是单一指令时，成本会进一步增加。受强化学习中的自我对弈技术和使用LLMs模拟人类代理的启发，我们提出了一种更有效的数据收集方法，即通过LLMs在不同角色中的对话来收集数据。这种方法通过LLMs的“自我对话”生成训练数据，可以用来进行监督微调。我们引入了一种自动化方式来衡量对话的（部分）成功。这一度量用于过滤生成的对话数据，并将其反馈给LLM进行训练。根据我们对对话质量的自动化和人工评估，我们证明了这种自我对话数据能提高结果。此外，我们还考察了生成对话的各种特征，并探讨了如何将这些特征与其作为训练数据的潜在用途联系起来。

## 1 引言

大型语言模型（LLMs）最近已证明它们可以作为对话代理在多种场景中部署，展示了在面向用户的应用中开辟新可能性，比如作为个人助手或客户支持。在这些场景中，对话代理必须能够迅速适应新任务，这在当前的范式下意味着要么使用现有数据，要么创建新的数据集。自然，这在可行性、时间和资源方面可能成为一个巨大的限制。此外，通过使用提示引导某种行为以避免微调可能会证明是脆弱且不可靠的——尽管在某些情况下可能仍然有效，但并不能保证其健壮性或一致性。

![参见说明](img/c39ce5cd94080a656fbae9dd720963cb.png)

图1：我们方法的示意图。两个大型语言模型（LLM），分别称为*客户端*和*代理*，被提示以不同的角色进行对话，代理被要求遵循特定的叙事结构。生成的对话将经过质量筛选，并用于对代理模型进行监督微调，直到它适应预期的对话结构。

在游戏的强化学习（RL）中，自我对弈已被证明是一种有效的手段，可以启动强大的系统，这些系统能够击败人类专家（Silver 等人，[2016](#bib.bib41)），即使没有初始的人工生成数据（Silver 等人，[2017](#bib.bib42)）或规则（Schrittwieser 等人，[2020](#bib.bib36)）。与此同时，随着数据集的不断增长，越来越大的语言模型展示出了有限但显著的语言能力（Hu 和 Levy，[2023](#bib.bib20)）以及一定程度的*心智理论*，即对其他代理、其行为和世界状态的推理（Sap 等人，[2022](#bib.bib33)）。这促使了一些研究，模拟单一用户（Liu 等人，[2022](#bib.bib23)；Argyle 等人，[2023](#bib.bib2)）或甚至互动的多代理群体（Liu 等人，[2023](#bib.bib24)；Park 等人，[2023](#bib.bib27)）。Gulcehre 等人（[2023](#bib.bib13)）的最新研究也展示了如何通过迭代过程在没有外部干预的情况下改进大型语言模型（LLM），使其能够为语言建模生成数据。我们将这些发展视为鼓励，探索使用LLM自我*对话*的方式，从LLM本身获取任务特定的微调数据。具体而言，我们专注于面向任务的对话，这是一个与客户服务、配对编程和数字个人助手等应用密切相关的问题。我们提出以下程序：两个版本的LLM参与对话，分别扮演客户和代理角色。两者都被赋予了如何在对话中行动的描述：客户被赋予一个个性和动机，而代理则根据代理的请求被赋予一组固定的行为。然后，它们进行对话，记录为潜在的训练样本。这些对话中的一些会是成功的，然而，许多对话将没有用处，因为它们会偏离主题或质量较差。为此，我们采用额外的筛选步骤，以避免在无效的样本上训练模型。我们的方式的示意图见[图 1](#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")。我们的贡献如下：  

## 2 相关工作

##### 自我对弈

在强化学习（RL）中，已广泛用于游戏中以启动强大的智能体：最早从象棋（Tesauro，[1994](#bib.bib44)；Pollack 和 Blair，[1996](#bib.bib29)）的成功开始，随后是奥赛罗（Van Der Ree 和 Wiering，[2013](#bib.bib49)），直到 AlphaGo 的出现（Silver 等人，[2016](#bib.bib41)），在自我对弈之前，AlphaGo 已经过人类专家棋步的预训练。系统的后续迭代版本摒弃了预训练（Silver 等人，[2017](#bib.bib42)），并且后来甚至在没有明确规则知识的情况下，在多个游戏中展现出超人类能力（Schrittwieser 等人，[2020](#bib.bib36)）。在自然语言处理（NLP）中，虽然曾有过一些使用自我对弈的对话系统研究（Shah 等人，[2018a](#bib.bib37)，[b](#bib.bib38)），但这些研究始于对话型大规模语言模型（LLMs）出现之前。相反，这些方法依赖于预定义的任务模式和槽填充，用于生成用户与系统机器人之间的对话，随后由群体工人进行重写和验证。因此，这仍然涉及大量的人工监督和参与。

##### LLM 作为智能体。

当前 LLM 中对话和推理的质量也使得模拟人类智能体的研究成为可能：例如，Argyle 等人（[2023](#bib.bib2)）展示了合适的提示可以使 LLM 模拟不同测试对象群体的回答，而 Liu 等人（[2022](#bib.bib23)）则创建了一个对话用户模拟器。Liu 等人（[2023](#bib.bib24)）将配备记忆的 LLM 智能体放入网格世界中，激发它们就敏感话题进行讨论，以促进社会规范的形成。通过进一步让 LLM 扮演不同的角色并使其能够检索相关记忆，Park 等人（[2023](#bib.bib27)）能够模拟一个虚拟村庄中的社会动态。

##### LLM 自我改进。

尽管在其自身输出上天真地重新训练生成模型可能会产生退化的解决方案（Alemohammad等，[2023](#bib.bib1)；Shumailov等，[2023](#bib.bib39)），但大语言模型已经成功地被证明可以为其他模型生成数据增强（Yoo等，[2021](#bib.bib51)；Dai等，[2023](#bib.bib6)）。此外，最近的研究工作正在探索利用大语言模型为其自身训练提供信号的限制：Bai等（[2022](#bib.bib3)）通过让模型根据不同的有害标准批评自己的生成，部分自动化了通过人工反馈对模型回应进行排序的过程，而Scheurer等（[2023](#bib.bib35)）则让大语言模型基于人工反馈生成多个改进版本。通过使用外部批评模型（Paul等，[2023](#bib.bib28)）或通过自我批评来处理信息抽取（Gero等，[2023](#bib.bib12)）、代码生成（Chen等，[2023](#bib.bib5)）和摘要生成（Saunders等，[2022](#bib.bib34)）等多种任务，已观察到回应的进一步改善。我们认为Gulcehre等（[2023](#bib.bib13)）的最新研究最接近我们的工作；作者展示了如何在*成长*步骤中生成数据后，通过不同的分批方式在*改进*步骤中逐步训练模型。相比之下，我们的方法专注于单一的改进步骤，并采用一种概念上更简单的监督微调策略，而非强化学习目标。此外，[Gulcehre等](#bib.bib13)的工作重点是条件语言建模，而我们则在面向任务的对话领域进行实验。这也激发了我们采用两个独立模型作为对话参与者的框架，尽管它也允许使用两个相同模型的副本（见下一节）。

##### 面向任务的对话与大语言模型（LLMs）。

最后，我们提示模型遵循特定工作流程的方式让人联想到其他面向任务的对话工作：Li等（[2022](#bib.bib21)）能够通过动态选择的提示模拟代理和用户。在Zhang等（[2023](#bib.bib52)）的研究中，作者通过外部提示模块引导大语言模型，并生成查询外部事实数据库的请求，从而影响最终生成结果。Zhao等（[2022](#bib.bib54)）提出了一种可编程的对话流程框架，为模型提供程序逻辑、相应的本体和前置的对话状态，以便生成回复。相比之下，我们的模型不依赖于外部模型来实现某些行为。

我们将这些研究方向视为以下几点的有力证据：a) 神经网络，特别是对话代理，可以在特定条件下自我生成训练数据；b) 大语言模型具备为自己提供学习信号的能力。这些洞察被综合到我们提出的方法中。

## 3 方法

我们首先介绍我们方法的一些符号。我们从两个预训练的LLM模型的参数向量 $\operatorname{\bm{\theta}}_{\text{client}}$ 和 $\operatorname{\bm{\theta}}_{\text{agent}}$ 开始，分别用于客户和代理模型。由于我们可能多次更新参数，我们将 $\operatorname{\bm{\theta}}_{\text{agent}}^{(0)}$ / $\operatorname{\bm{\theta}}_{\text{client}}^{(0)}$ 表示为原始参数，并将 $\operatorname{\bm{\theta}}_{\text{agent}}^{(t)}$ / $\operatorname{\bm{\theta}}_{\text{client}}^{(t)}$ 表示为我们程序中时间步骤 $t$ 的参数。接下来，我们定义了两个不同的提示 $\operatorname{\mathbf{pr}}_{\text{agent}}$ 和 $\operatorname{\mathbf{pr}}_{\text{client}}$，它们为两个模型提供角色描述和行为指导。这些模型共同定义了一个可能对话的分布 $p(\operatorname{\mathbf{d}}|\operatorname{\bm{\theta}}_{\text{client}}^{(t)},%

\operatorname{\bm{\theta}}_{\text{agent}}^{(t)},\operatorname{\mathbf{pr}}_{%

\text{agent}},\operatorname{\mathbf{pr}}_{\text{client}})$。

对于我们在[图1](#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")中的方法，我们现在从 $N$ 个对话 $\mathcal{D}^{(t)}=\{\operatorname{\mathbf{d}}_{1}^{(t)},\ldots,\operatorname{%

\mathbf{d}}_{N}^{(t)}\}$ 从这个分布中在每个时间步骤生成，通过让模型根据它们的提示进行指定角色的对话。尽管我们可以期望在给定足够的模型和高质量的提示下，一些生成的对话能够成功，但并不能保证它们能够实现新的对话目标。出于这个原因，我们使用过滤函数来创建一个生成对话的子集 $\mathcal{D}^{(t)}_{\bigtriangledown}\subseteq\mathcal{D}^{(t)}$。一个过滤器的例子是只包含成功的对话，这些对话是通过我们稍后在[第4.1节](#S4.SS1 "4.1 Evaluation ‣ 4 Experiments ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")介绍的方法识别的。过滤后的示例用于通过最小化负对数似然来微调模型。

| | $-\frac{1}{|\mathcal{D}^{(t)}_{\bigtriangledown}|}\sum_{\operatorname{\mathbf{d% }}^{(t)}\in\mathcal{D}^{(t)}_{\bigtriangledown}}\log p_{(\cdot)}\big{(}%

\operatorname{\mathbf{d}}^{(t)}\big{|}\operatorname{\bm{\theta}}_{(\cdot)}^{(t%

)})，| |（1）|

微调后，我们获得一组新的参数 $\operatorname{\bm{\theta}}_{(\cdot)}^{(t+1)}$，并可以再次开始这一过程。这个框架支持多种不同的变体：$\operatorname{\bm{\theta}}_{\text{agent}}$ 和 $\operatorname{\bm{\theta}}_{\text{client}}$ 可以是相同或不同模型的副本；我们可以微调一个或两个副本。我们使用不同的模型作为代理和客户，并且不更新客户模型 $(\operatorname{\bm{\theta}}_{\text{client}}^{(t)}=\operatorname{\bm{\theta}}_{%

\text{client}}^{(0)}$）。

### 3.1 提示设计

![请参见说明文字](img/63c38dca46766ef548deebd01df22d19.png)

图 2：结构化提示的示意图：工作流被解析为有向图（左图）。在每一轮对话中，我们要求 LLM 比较客户的最后一句话与当前节点出边对应的参考回复。如果选择了其中一个回复，我们就继续沿着图中的下一节点并提示代理下一个问题，否则我们停留在同一位置并允许模型自由生成。

为了获得既能成功微调对话又足够多样化的对话，我们特别关注代理提示 $\operatorname{\mathbf{pr}}_{\text{agent}}$ 和客户提示 $\operatorname{\mathbf{pr}}_{\text{client}}$ 的设计，接下来我们将详细介绍。

##### 代理和客户提示。

我们通过提示客户和代理角色、个性、到目前为止的对话历史、客户的意图以及代理的指令来创建对话。角色、意图的不同组合以及生成的随机性产生了多种场景。这些提示在 [Section A.2](#A1.SS2 "A.2 Self-talk Loop ‣ Appendix A Implementation Details ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk") 中有详细描述。

##### 结构化提示。

在初步实验中，我们发现我们的模型难以遵循提供的工作流，这主要归因于模型的规模和基础预训练数据的质量。这对启动对话的质量产生了负面影响，在微调代理模型时提供了噪声训练信号。为了解决这个问题，我们将工作流解析为有向图 $\mathcal{G}=\{\mathcal{V},\mathcal{E}\}$，其中顶点 $\mathcal{V}$ 对应于代理的提问，边 $\mathcal{E}$ 对应于客户的预定义参考答案。基于这些，我们提出了*结构化提示*，如下图 [Figure 2](#S3.F2 "Figure 2 ‣ 3.1 Prompt Design ‣ 3 Method ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk") 所示：在对话的每一步，我们考虑上一个代理发言的节点 $\operatorname{\mathbf{v}}_{i}\in\mathcal{V}$，并向 LLM 提供一组建议的客户回复，这些回复与 $\operatorname{\mathbf{v}}_{i}$ 的出边相对应，并要求它选择最接近客户实际回复的选项（包括“以上都不是”选项）。然后，我们根据这个回复提示代理下一个问题，并在图中向上移动到下一个节点 $\operatorname{\mathbf{v}}_{\text{next}}\in\text{Children}(\operatorname{%

\mathbf{v}}_{i})$，直到对话结束或图中的叶节点被触及。我们在[第 A.2 节](#A1.SS2 "A.2 自我对话循环 ‣ 附录 A 实施细节 ‣ 通过自我对话启动基于 LLM 的任务导向对话代理")展示了完整的提示。

## 4 个实验

##### 数据集。

对于我们希望 LLM 在对话中体现的角色，我们从 LIGHT 数据集（Urbanek 等， [2019](#bib.bib48)）中获取了描述。该数据集最初用于解决幻想世界环境中的任务代理，包含了 7487 个不同的任务描述，其中包括角色及其个性。例如，来自灯中的精灵角色，其个性为 *“我是一个精灵，我喜欢捉弄那些释放我出灯的人。我被一个邪恶的巫婆囚禁在这个灯中。我非常愤怒，因为我必须将余生度过在灯中做囚徒”*。我们选择了五个角色作为代理的潜在角色（商店老板、医生、巫婆、酒馆老板、灯中的精灵），并选择了另外一组 $16$ 个最常见的角色作为潜在客户（见[第 A.1 节](#A1.SS1 "A.1 数据创建 ‣ 附录 A 实施细节 ‣ 通过自我对话启动基于 LLM 的任务导向对话代理")）。为了创建对话流程，我们接着提示一个包含 130 亿参数的 OpenLlama 变体（Touvron 等， [2023](#bib.bib46); Together Computer，[2023](#bib.bib45); Geng 和 Liu，[2023](#bib.bib11))¹¹1 可在 [https://huggingface.co/VMware/open-llama-13b-open-instruct](https://huggingface.co/VMware/open-llama-13b-open-instruct) 获取。生成每个代理角色的三个不同工作流程，分别包含四个、六个和七个问题，然后手动编辑以确保质量。这些工作流程的概述和示例展示在[第 A.1 节](#A1.SS1 "A.1 数据创建 ‣ 附录 A 实施细节 ‣ 通过自我对话启动基于 LLM 的任务导向对话代理")中。

##### 自我对话循环。

对于[图1](#S1.F1 "图1 ‣ 1 介绍 ‣ 通过自我对话启动基于LLM的任务导向对话代理")中的自我对话循环，我们选择了具有300亿参数的MosaicAI聊天模型（MosaicML NLP团队，[2023](#bib.bib25)）作为客户端模型，并选择了来自同一家族的70亿参数模型作为代理。我们发现，客户端模型仅通过提示就能够稳定工作，因此我们决定仅对代理模型进行微调。在每次对话生成回合中，我们首先随机选择一个客户端角色、代理角色和相应的代理工作流程。然后，我们按顺序生成对话的每一句话，最多进行八轮对话（每轮包括每个角色的一句话）。我们还通过结合字符串匹配和提示LLM以及一些后处理步骤来检查对话是否结束，以确保生成的质量，所有这些内容在[附录A.2](#A1.SS2 "A.2 自我对话循环 ‣ 附录A 实施细节 ‣ 通过自我对话启动基于LLM的任务导向对话代理")中有详细描述。

##### 过滤器。

生成的对话随后会经过过滤，并用于微调代理模型进行单轮训练。我们比较了不同过滤程序的影响，基准方法使用整个生成的对话集进行微调（None），并随机采样一个$p$%的子集（$p$-Random）。我们还探讨了在工作流程中是否至少完成了$k$个子目标或$p$%的子目标（$k$-Subgoals和$p$-Subgoals，见下一节）。

### 4.1 评估

为了评估我们方法的有效性，我们从以下几个方面进行评估。

##### 对话多样性。

我们通过追踪独特词汇的数量（# Words）和独特的 $n$-gram（#$n$-Grams；最多到5阶）来衡量对话的多样性。我们还通过衡量同一角色的$25$个对话之间的平均成对ROUGE-L（多样性）来衡量多样性。

##### 子目标完成。

那么，成功的对话是什么样的呢？理想情况下，智能体应该通过对话流程被引导到可能的对话结局之一，并实现其初衷目标（具体示例请见[第A.1节](#A1.SS1 "A.1 数据创建 ‣ 附录A 实现细节 ‣ 通过自我对话引导基于LLM的任务导向对话代理")）。Urbanek等人（[2019](#bib.bib48)）将任务定义为由智能体选择一组有限但庞大的环境动作来遍历的有向图。由于文本生成中的潜在动作是无限的，判断对话是否达到某个步骤就变得更加困难。为此，我们使用与[第3.1节](#S3.SS1 "3.1 提示设计 ‣ 3 方法 ‣ 通过自我对话引导基于LLM的任务导向对话代理") / [图2](#S3.F2 "图2 ‣ 3.1 提示设计 ‣ 3 方法 ‣ 通过自我对话引导基于LLM的任务导向对话代理")中相同的设置，将我们的对话流程也解析为有向图，其中每个节点都与智能体的发话相关联。为了决定是否沿着某个边继续，我们比较智能体的实际发话和对话流程中的参考发话是否足够相似。我们通过ROUGE-L（Lin，[2004](#bib.bib22)）测量这种相似度，该方法由evaluate包实现，²²2详情见[https://github.com/huggingface/evaluate](https://github.com/huggingface/evaluate)。然后，我们将其与预定义的阈值进行比较。我们在[第A.3节](#A1.SS3 "A.3 自动评估 ‣ 附录A 实现细节 ‣ 通过自我对话引导基于LLM的任务导向对话代理")中解释了初始阈值是如何在外部数据集上设定的，并在本节末验证了这些度量的合理性。为了进行评估，我们跟踪对话是否达到了其中一个可能的结局（对话成功），并且衡量通过有向图的深度（绝对深度）和最大深度的比例（相对深度）来实现的工作流步骤数量，直观地说，即工作流的相对完成度。

##### **角色一致性**。

另一个问题是关于客户端是否保持角色设定，还是忽略提示并生成无关的输出。我们借鉴了Shuster等人提出的地下城主模型（[2020](#bib.bib40)），该模型旨在鼓励人类参与者在对话中进行角色扮演。为此，我们对DeBERTa v3模型（He等人，[2021a](#bib.bib15)，[b](#bib.bib16)）进行了微调，使用了$200$k正例和$200$k负例，并采用简单的二元交叉熵目标。LIGHT数据集包含与任务相关的对话片段，我们利用这些对话通过抽取一个片段并将角色名称和人格与对话中的随机发言组合来创建正例。对于负例，我们从另一段对话中随机抽取不同角色的发言。此外，我们将所有工作流程的步骤添加到训练数据中，并将其上采样308倍，最终总共生成约$500k$个训练样本。更多关于训练的详细信息，请参见[Section A.3](#A1.SS3 "A.3 自动评估 ‣ 附录A 实现细节 ‣ 通过自我对话引导基于LLM的任务导向对话代理的引导")。在我们的实验中，我们通过使用地下城主模型对每个发言进行评分，并对整个对话的评分进行平均来评估角色一致性（Char. Cons.）。

##### 自动化指标验证。

在继续我们的实验之前，我们首先希望验证我们的自动评估指标。为此，我们从未训练过的模型中抽取了$50$个对话，并进行人工标注，然后将自动评估结果与真实标签进行对比。我们发现，对于子目标跟踪，绝对深度的Kendall’s $\tau$相关性为$0.67$，相对深度的Pearson’s $\rho$相关性为$0.67$。地下城主模型在对话中的准确率为$0.87$，$F_{1}$得分为$0.93$。我们在[Section B.1.3](#A2.SS1.SSS3 "B.1.3 元评估 ‣ B.1 自动化指标验证 ‣ 附录B 实验附录 ‣ 通过自我对话引导基于LLM的任务导向对话代理的引导")中提供了这些结果的更深入分析。

### 4.2 引导微调数据

表格 1：使用不同过滤器的单步循环结果。展示了过滤对话的百分比（% Filtered）、代理角色一致性（Char. Cons.）、已完成子目标的数量（$\varnothing$ Abs. Depth）和相对工作流程完成度（$\varnothing$ Rel. Depth）、已完成对话的百分比（$\%$ Ended），以及对话多样性。结果来源于$100$个测试对话和来自$3$个不同种子的随机过滤器得分。

| 数据 | % 过滤 | 角色一致性 Char. Cons. $\uparrow$ | $\varnothing$ 绝对深度 Abs. Depth $\uparrow$ | $\varnothing$ 相对深度 Rel. Depth $\uparrow$ | $\%$ 完成 Ended $\uparrow$ | 单词数 Words $\uparrow$ | $N$-Grams 数量 $\uparrow$ | 多样性 Diversity $\uparrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 初始对话 | - | $0.77$ | $2.15$ | $0.38$ | $0.26$ | $57.16$ | $373.87$ | $0.69$ |
| \cdashline1-9 无过滤器 | $0.00$ | $0.78$ | $2.22$ | $0.38$ | $0.29$ | $47.32$ | $285.94$ | $0.60$ |
| $\%$-随机 ($0.1$) | $0.90$ | $\underset{\pm 0.00}{0.77}$ | $\underset{\pm 0.10}{2.15}$ | $\underset{\pm 0.02}{0.37}$ | $\underset{\pm 0.03}{0.24}$ | $\underset{\pm 0.66}{45.89}$ | $\underset{\pm 6.21}{273.71}$ | $\underset{\pm 0.01}{0.62}$ |
| 绝对子目标 ($4$) | $0.82$ | $0.78$ | $2.30$ | $0.39$ | $0.27$ | $46.62$ | $297.78$ | $0.64$ |
| 绝对子目标 ($5$) | $0.92$ | $0.80$ | $2.41$ | $0.41$ | $0.32$ | $46.55$ | $280.53$ | $0.63$ |
| 绝对子目标 ($6$) | $0.98$ | $\mathbf{0.81}$ | $2.11$ | $0.35$ | $0.22$ | $54.14$ | $343.07$ | $0.64$ |
| $\%$-子目标 ($0.1$) | $0.90$ | $0.76$ | $2.37$ | $0.39$ | $0.30$ | $44.91$ | $266.77$ | $0.63$ |
| $\%$-子目标 ($0.05$) | $0.95$ | $0.78$ | $\mathbf{2.54}$ | $\mathbf{0.43}$ | $\mathbf{0.36}$ | $49.40$ | $303.65$ | $0.66$ |
| $\%$-子目标 ($0.01$) | $0.99$ | $0.78$ | $2.31$ | $0.39$ | $0.29$ | $\mathbf{55.25}$ | $\mathbf{355.44}$ | $\mathbf{0.69}$ |
| 已结束 | $0.74$ | $0.76$ | $2.22$ | $0.38$ | $0.26$ | $45.51$ | $277.25$ | $0.64$ |

在我们的第一次实验中，我们预生成了约 $9000$ 个对话，以确保不同过滤器之间的可比性，并在过滤后的数据上对模型进行单次周期的微调。然后，我们使用更新后的代理模型生成额外的 $100$ 个对话，并测量评估指标的变化。

##### 结果。

不同过滤方法的结果见 [表 1](#S4.T1 "表 1 ‣ 4.2 自举微调数据 ‣ 4 实验 ‣ 通过自我对话自举基于 LLM 的任务导向对话代理")。我们可以观察到，使用无过滤器或随机过滤器在最佳情况下几乎没有改善。最佳结果是通过选择至少包含 $5$ 个完成工作流步骤的对话或在完成度方面排名前 $5\%$ 的对话来实现的，一旦这些标准变得更严格，性能反而下降。这很可能是由于微调数据集大小缩小所致，因为仅剩下 $1-2\%$ 的对话。有趣的是，在所有情况下，对话的多样性都减少了；我们认为这是因为特别是一个坚守给定工作流、不偏离脚本的代理模型会生成对话，这些对话的多样性整体上较低。

![参见标题](img/4c12ad70ce2300bcea53506ba00a8686.png)

(a) 通过 Spearman 的 $\rho$ 相关性衡量重要性。

![参见标题](img/748e1ad0f5a1f161e7e13c78725f2129.png)

(b) 通过线性回归系数衡量重要性。

图 3：分析微调数据集的属性与其对对话完成度的绝对影响之间的关系，给定 (a) Spearman 的 $\rho$ 相关性值和 (b) 线性回归模型的系数（无偏差并使用 Lasso 正则化）。误差条和正则化权重通过交叉验证确定。

##### 什么样的数据适合微调？

[表格1](#S4.T1 "表格1 ‣ 4.2 自举微调数据 ‣ 4 实验 ‣ 通过自我对话自举基于LLM的任务导向对话代理")中的结果引发了一个问题：为什么不同筛选后的微调数据集表现得更好？我们通过再次应用自动化评估指标来尝试回答这个问题——这次我们仅针对筛选后的对话进行分析，并将结果与对话成功率的变化相关联，后者通过绝对深度来衡量。我们通过收集[表格1](#S4.T1 "表格1 ‣ 4.2 自举微调数据 ‣ 4 实验 ‣ 通过自我对话自举基于LLM的任务导向对话代理")中使用的每个微调数据集的属性，并进行两种不同的分析：使用Spearman的$\rho$对其与[图2(a)](#S4.F2.sf1 "2(a) ‣ 图3 ‣ 结果 ‣ 4.2 自举微调数据 ‣ 4 实验 ‣ 通过自我对话自举基于LLM的任务导向对话代理")中的绝对深度变化进行相关性分析，并将特征应用常规缩放后，拟合一个线性回归模型（不带偏差），并在[图2(b)](#S4.F2.sf2 "2(b) ‣ 图3 ‣ 结果 ‣ 4.2 自举微调数据 ‣ 4 实验 ‣ 通过自我对话自举基于LLM的任务导向对话代理")中使用$l_{1}$正则化。我们可以看到，这两种方法一致认为，回合数对性能有正面影响，而话语的平均长度则对对话质量产生负面影响。这与我们从实验中得到的直觉一致，坏质量的对话会导致模型产生冗长的发言，偏离脚本，陈述不必要的信息，或者提前结束，或者发言的长度小于底层对话工作流所要求的长度。除此之外，这两种方法一致认为，微调数据中的工作流进展和角色一致性对性能有正面影响，尽管对于其他属性的结果存在一些轻微的矛盾。然而，由于观察样本数较少，这些分析结果应谨慎对待。另外，像话语长度对代理改进的负面影响*本身*不太可能成立，似乎更有可能的是，所有或许多属性都受某种共同的潜在混杂因素（即对话质量）的影响，并且这种影响在我们的数据中沿这些轴表现出来。因此，尽管我们可以使用这些观察结果作为微调数据集有用性的代理，但它们并不意味着因果关系。

### 4.3 人工评估

![参考说明](img/7ddd6bcd0b312fd20d57b2d06a7fbf34.png)

图 4：人类评估研究的结果，涉及三个基准方法和来自[第 4.2 节](#S4.SS2 "4.2 Bootstrapping Finetuning Data ‣ 4 Experiments ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")的两个最佳筛选器，涵盖六个不同问题。展示的是每个筛选器的评分百分比，评分方式为五点量表，或者使用正面 (![Refer to caption](img/cb6d42c1333c07e1fadec42d01d94815.png))、负面 (![Refer to caption](img/72a1726ad83b8e793cc8a619c0cf535e.png)) 和 *不确定* (![Refer to caption](img/d96abe05ee839e802a3d9538ab6df38a.png)) 选项。虚线表示数值平均值，$\bigstar$ 表示与通过 ASO 测试评估的所有其他选项相比具有统计显著性 (Del Barrio et al., [2018](#bib.bib7); Dror et al., [2019](#bib.bib8); Ulmer et al., [2022](#bib.bib47))，$\tau=0.5$ 和置信度为 $\alpha=0.9$。

为了三角验证基于自动化结果的前述结论，并额外评估其他方面，我们还包括了基于人工评审的评估。我们评估了未经训练的模型（Untrained），使用全部微调数据（No Filter）、%-随机 (0.1)、绝对子目标 (5) 和 %-子目标 (0.05)，来自[第 4.2 节](#S4.SS2 "4.2 Bootstrapping Finetuning Data ‣ 4 Experiments ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")，从帮助性、一致性和对话成功等维度进行评估。用户需要评定这些属性，评分方式可以选择三个选项 (![[Uncaptioned image]](img/cb6d42c1333c07e1fadec42d01d94815.png)，![[Uncaptioned image]](img/72a1726ad83b8e793cc8a619c0cf535e.png) 或 *不确定*)，或者在$1$（最差）到$5$（最佳）的量表上进行评分。我们请$n=9$名参与者对共$100$个对话进行评判（每种筛选方法$20$个），这些对话是从我们的实验中随机抽样的。每个数据点获得两个注释，我们观察到注释者之间的Cohen $\kappa \approx 0.52$的一致性。我们在[第 B.2 节](#A2.SS2 "B.2 Human Evaluation Study ‣ Appendix B Experimental Appendix ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")中提供了有关我们的评估、注释指南以及工具的更多细节。

<svg class="ltx_picture ltx_centering" height="129.1" id="S4.F5.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,129.1) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 300.98 103.81)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.19)"><text transform="matrix(1 0 0 -1 0 0)">“Make the prince fall in love with them”</text></g></g></g> <g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="81.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Genie from lamp ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Good day, what do you wish for?   Villager ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Greetings, genie. I wish for the prince to fall in love with me.   Genie from lamp ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): I am not a matchmaker and cannot make people fall in love. You should try to make your own chances.   Villager ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Thank you for your advice, genie. I will try my best to make the prince fall in love with me.   Genie from lamp ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Goodbye!  Villager ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Goodbye, genie.</foreignobject></g></g></svg>

图 5：在使用 %-子目标 (0.05) 筛选器进行微调后生成的对话，代理忽略了给定的工作流程。

<svg class="ltx_picture ltx_centering" height="134.06" id="S4.F6.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,134.06) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 383.42 108.77)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.19)"><text transform="matrix(1 0 0 -1 0 0)">“Book a bed for the night”</text></g></g></g> <g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="86.26" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Tavern Owner ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Good day, how can I help you?   Person ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Good day, I am in need of a bed for the night. I have been traveling for days and am quite tired.   Tavern Owner ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): What is the duration of your stay?   Person ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Just for the night, I have to continue my journey in the morning.   Tavern Owner ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): What kind of room would you like?   Person ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  A simple room with a bed and a fireplace would be perfect.   Tavern Owner ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Good day, how can I help you?  … [Agent gets stuck in a loop, asking the last two questions over and over.]</foreignobject></g></g></svg>

图 6：在使用绝对子目标 (5) 筛选器进行微调后生成的对话，代理陷入了对话循环。

##### 结果

按过滤器聚合的判断结果见于[图4](#S4.F4 "图4 ‣ 4.3 人类评估 ‣ 4 实验 ‣ 通过自我对话引导大模型基础的任务导向对话代理")。有趣的是，我们可以看到，在所有数据上进行微调通常会*降低*模型质量，但在一个随机子集上微调*可能*会提高性能。与基准相比，人类判断者惊讶地评定，Abs. Subbgoals (5) 过滤器生成的模型质量在对话成功、工作流遵循和整体对话质量等维度上与基准持平或更差。然而，使用另一个表现最好的过滤器，$\%$-Subgoals($0.05$)，我们的人工评分者通过三角测量验证了自动化指标的正面结果，结果显示该代理在除一个类别外的所有类别中（显著地）表现更好，表明正确的对话子集可以提供更强的训练信号。尽管取得了这些成功，我们仍然发现一些常见的失败案例，在这里我们突出其中的两个：³³3我们使用代理！[[未标注图片]](img/09b1b1572d5df408c3a44a15f96ddc74.png)来展示代理角色的发言（包括那些不对应精灵的发言），并使用客户！[[未标注图片]](img/3a9f183c741a0bf3d7d839bba9fd920a.png)来展示客户角色的发言。在[图5](#S4.F5 "图5 ‣ 4.3 人类评估 ‣ 4 实验 ‣ 通过自我对话引导大模型基础的任务导向对话代理")中，我们回顾了一个对话，代理在第一次发言后完全忽略了工作流，尽管后续对话依然自然进行。在[图6](#S4.F6 "图6 ‣ 4.3 人类评估 ‣ 4 实验 ‣ 通过自我对话引导大模型基础的任务导向对话代理")中的第二个例子中，我们看到代理在对话开始成功后，回到初始问题，并陷入与客户的对话循环中。我们还发现，在客户的回应与工作流中定义的样本答案差异过大时，对话可能会脱轨；在这种情况下，代理通常会即兴发挥，并在接下来的对话中未能恢复。我们将在[附录B.4](#A2.SS4 "B.4 示例对话 ‣ 附录B 实验附录 ‣ 通过自我对话引导大模型基础的任务导向对话代理")中展示更多示例并讨论其他失败案例。

## 5 讨论

我们的实验结果展示了在任务导向对话设置中实施自我对话时所遇到的挑战。这些挑战包括代理是否能够仔细遵循工作流程，以及是否能够可靠地选择下一步操作——这一问题至少部分源于对话的开放性和释义性特征。然而，我们提供了见解，表明正确的过滤技术可以隔离高质量数据，并为代理的语言模型提供正向训练信号，从而提高其性能。这一结论与日益增多的关于LLM自我批评的文献以及Gulcehre等人（[2023](#bib.bib13)）关于使用自生成数据进行微调的相关工作相一致。一个相关的未解问题是，这一过程是否能可靠地重复多次，并确定在什么条件下它会导致性能提升或模型退化。在一些初步实验中，我们发现多轮自我对话最初会带来轻微的改进，但如果过滤后的微调样本数量过低，也会变得不稳定。我们将进一步探讨这个问题留给未来的工作，并在[附录B.3节](#A2.SS3 "B.3 Miscellaenous Insights ‣ Appendix B Experimental Appendix ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")中收集了我们实验中的其他小型见解。

## 6 结论

尽管先前的研究已提出关于在其自身生成数据上训练的模型可能出现分布崩溃的担忧（Alemohammad 等人，[2023](#bib.bib1); Shumailov 等人，[2023](#bib.bib39)），我们证明了——在合适的设置下——语言模型可以通过在自身生成数据上进行训练实现自我改进，并成为更好的任务导向对话代理。正如我们在[第4.2节](#S4.SS2 "4.2 Bootstrapping Finetuning Data ‣ 4 Experiments ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")中所展示的，这取决于设置；使用不同的模型和变化的提示有助于创建足够多样化的数据，附加的过滤器则确保了数据的质量。我们特别发现，能够在质量和数据总量之间取得平衡的过滤器最适合选择微调数据，并通过[第4.3节](#S4.SS3 "4.3 Human Evaluation ‣ 4 Experiments ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")中的人工评估验证了这些见解。

此外，我们建立了面向任务的对话设置中评估智能体的新方法，并展示了这些方法可以用来量化我们微调模型的进展。我们将这项工作视为与以下工作一致的证据，例如Scheurer等人（[2023](#bib.bib35)）；Paul等人（[2023](#bib.bib28)）；Gero等人（[2023](#bib.bib12)）；Chen等人（[2023](#bib.bib5)）；Saunders等人（[2022](#bib.bib34)），证明在适当的条件下，使用LLM（大型语言模型）进行自我改进确实是可能的。我们是——据我们所知——首次展示LLM自言自语在对话环境中的实用性。我们希望这项工作能激发更多关于理解自我改进的需求和限制的研究。

##### 未来工作

本研究聚焦于面向任务的对话体智能体的自言自语。由于我们正在对自启动对话进行微调，我们预期模型可能会失去一些与当前任务无关的通用对话能力。未来的工作可以探讨如何在保持这些能力的同时进行微调。此外，这一版本的自言自语仍然可以通过生成更高质量的对话并识别对模型微调最有信息价值的对话来改进，这类似于主动学习中的工作（参见例如张等人，[2022](#bib.bib53)）。最后，我们的设置直观上也非常适合使用负面信号，因为低质量对话可以用作负奖励。这样，智能体可以通过类似直接偏好优化（Rafailov等人，[2023](#bib.bib31)）的目标进行优化，或者通过将其嵌入到强化学习环境中使用自然语言策略优化（Ramamurthy等人，[2023](#bib.bib32)）。

## 局限性

本研究存在一些局限性：首先，我们将实验限制在面向任务的对话中，因此无法对开放性对话或大型语言模型执行的其他更具体任务得出结论。为了隔离自我对话的影响，我们还限制了代理与客户之间的互动范围，并约束了对话的格式。然而，Scheurer 等人（[2023](#bib.bib35)）；Paul 等人（[2023](#bib.bib28)）；Gero 等人（[2023](#bib.bib12)）；Chen 等人（[2023](#bib.bib5)）；Saunders 等人（[2022](#bib.bib34)）的相关研究已证明自我改进在各种任务中的潜力。其次，我们探索了一种设置，其中两个不同的语言模型被严格指令，以生成可用于训练的对话。可能的是，使用单一模型作为代理和客户可能无法生成足够多样的对话，或者较小的模型根本无法创建足够有用的对话。此外，即使包含所有技术，如模型引导的提示，生成的对话有时仍然会出现模型陷入循环、跑偏脚本，甚至生成不合逻辑的对话，这些对话有时也很难通过我们定义的过滤器捕捉到。尽管有轶事证据表明较大的模型可能不会遭遇这些问题，但在我们的设置中，很难明确区分模型大小和训练数据的影响。

## 道德考虑

##### 模型偏见。

许多研究致力于调查现代自然语言处理模型的社会偏见（例如，参见 Stanczak 和 Augenstein，[2021](#bib.bib43)；Garrido-Muñoz 等人，[2021](#bib.bib10) 的综述）。由于大型语言模型往往会反映其开发过程中存在的偏见（Waseem 等人，[2021](#bib.bib50)），在其自生成数据上微调模型可能会加剧其自身的偏见。

##### 社会危害。

最近的研究表明，LLM 在欺骗用户方面的负面潜力，例如在个性化钓鱼攻击的形式中（Hazell，[2023](#bib.bib14)）。因此，训练不再依赖人类生成的训练数据的代理，可能会简化恶意行为者创建欺骗性对话模型的过程。

## 致谢

我们要感谢杜婉瑜和何汉为模型引导提示提供的灵感，感谢 Nikolaos Pappas 和 Raphael Shu 对本研究的富有成效的讨论。我们还要感谢 Elisa Bassignana、Sam Davidson、Etsuko Ishii、Josef Valvoda、张宇威，特别是 Hossein Aboutalebi 在参与我们的人类评估研究中的努力。

## 参考文献

+   Alemohammad 等人（2023）Sina Alemohammad、Josue Casco-Rodriguez、Lorenzo Luzi、Ahmed Imtiaz Humayun、Hossein Babaei、Daniel LeJeune、Ali Siahkoohi 和 Richard G Baraniuk. 2023. 《自我消耗生成模型失控》。*arXiv 预印本 arXiv:2307.01850*。

+   Argyle 等人（2023）Lisa P Argyle、Ethan C Busby、Nancy Fulda、Joshua R Gubler、Christopher Rytting 和 David Wingate. 2023. 《从一到多：利用语言模型模拟人类样本》。*政治分析*，31(3)：337–351。

+   Bai 等人（2022）Yuntao Bai、Saurav Kadavath、Sandipan Kundu、Amanda Askell、Jackson Kernion、Andy Jones、Anna Chen、Anna Goldie、Azalia Mirhoseini、Cameron McKinnon 等人. 2022. 《宪法 AI：来自 AI 反馈的无害性》。*arXiv 预印本 arXiv:2212.08073*。

+   Chen 等人（2021）Derek Chen、Howard Chen、Yi Yang、Alexander Lin 和 Zhou Yu. 2021. 《基于动作的对话数据集：构建更深入的任务导向对话系统的语料库》。载于 *2021年北美计算语言学协会：人类语言技术会议论文集*，页码 3002–3017。

+   Chen 等人（2023）Xinyun Chen、Maxwell Lin、Nathanael Schärli 和 Denny Zhou. 2023. 《教大语言模型自我调试》。*arXiv 预印本 arXiv:2304.05128*。

+   Dai 等人（2023）Haixing Dai、Zhengliang Liu、Wenxiong Liao、Xiaoke Huang、Zihao Wu、Lin Zhao、Wei Liu、Ninghao Liu、Sheng Li、Dajiang Zhu 等人. 2023. 《Chataug：利用 ChatGPT 进行文本数据增强》。*arXiv 预印本 arXiv:2302.13007*。

+   Del Barrio 等人（2018）Eustasio Del Barrio、Juan A Cuesta-Albertos 和 Carlos Matrán. 2018. 《评估几乎随机顺序的最优传输方法》。载于 *《不确定性的数学》*，页码 33–44. Springer。

+   Dror 等人（2019）Rotem Dror、Segev Shlomov 和 Roi Reichart. 2019. [《深度主导——如何正确比较深度神经模型》](https://doi.org/10.18653/v1/p19-1266)。载于 *第57届计算语言学协会年会论文集（ACL 2019），意大利佛罗伦萨，2019年7月28日至8月2日，第1卷：长篇论文*，页码 2773–2785. 计算语言学协会。

+   Fan 等人（2018）Angela Fan、Mike Lewis 和 Yann N. Dauphin. 2018. 《层次化神经故事生成》。载于 *第56届计算语言学协会年会论文集（ACL 2018），澳大利亚墨尔本，2018年7月15日至20日，第1卷：长篇论文*，页码 889–898. 计算语言学协会。

+   Garrido-Muñoz 等人（2021）Ismael Garrido-Muñoz、Arturo Montejo-Ráez、Fernando Martínez-Santiago 和 L Alfonso Ureña-López. 2021. 《深度 NLP 中的偏见调查》。*应用科学*，11(7)：3184。

+   Geng 和 Liu（2023）Xinyang Geng 和 Hao Liu. 2023. [《Openllama：Llama 的开放复现》](https://github.com/openlm-research/open_llama)。

+   Gero等人（2023）Zelalem Gero, Chandan Singh, Hao Cheng, Tristan Naumann, Michel Galley, Jianfeng Gao, 和 Hoifung Poon. 2023. 自我验证提升了少样本临床信息提取。*arXiv预印本arXiv:2306.00024*。

+   Gulcehre等人（2023）Caglar Gulcehre, Tom Le Paine, Srivatsan Srinivasan, Ksenia Konyushkova, Lotte Weerts, Abhishek Sharma, Aditya Siddhant, Alex Ahern, Miaosen Wang, Chenjie Gu, Wolfgang Macherey, Arnaud Doucet, Orhan Firat, 和 Nando de Freitas. 2023. [强化自训练（REST）用于语言建模](http://arxiv.org/abs/2308.08998)。

+   Hazell（2023）Julian Hazell. 2023. 大规模语言模型可以有效扩展 spear phishing 攻击。*arXiv预印本arXiv:2305.06972*。

+   何等人（2021a）Pengcheng He, Jianfeng Gao, 和 Weizhu Chen. 2021a. [Debertav3：通过梯度解耦的嵌入共享和Electra风格的预训练提升DeBERTa](http://arxiv.org/abs/2111.09543)。

+   何等人（2021b）Pengcheng He, Xiaodong Liu, Jianfeng Gao, 和 Weizhu Chen. 2021b. Deberta：通过解耦注意力的解码增强BERT。在*第9届国际学习表征会议，ICLR 2021，虚拟会议，奥地利，2021年5月3-7日*。OpenReview.net。

+   Holtzman等人（2020）Ari Holtzman, Jan Buys, Li Du, Maxwell Forbes, 和 Yejin Choi. 2020. 神经文本退化的奇异案例。在*第8届国际学习表征会议，ICLR 2020，埃塞俄比亚，亚的斯亚贝巴，2020年4月26-30日*。

+   Holtzman等人（2018）Ari Holtzman, Jan Buys, Maxwell Forbes, Antoine Bosselut, David Golub, 和 Yejin Choi. 2018. 使用合作判别器学习写作。在*第56届计算语言学协会年会论文集，ACL 2018，澳大利亚墨尔本，2018年7月15-20日，第1卷：长篇论文*，第1638-1649页。计算语言学协会。

+   胡等人（2022）Edward J. Hu, Yelong Shen, Phillip Wallis, Zeyuan Allen-Zhu, Yuanzhi Li, Shean Wang, Lu Wang, 和 Weizhu Chen. 2022. Lora：大规模语言模型的低秩适配。在*第十届国际学习表征会议，ICLR 2022，虚拟会议，2022年4月25-29日*。

+   胡和Levy（2023）Jennifer Hu 和 Roger Levy. 2023. 基于提示的方法可能低估了大规模语言模型的语言泛化能力。*arXiv预印本arXiv:2305.13264*。

+   李等人（2022）Zekun Li, Wenhu Chen, Shiyang Li, Hong Wang, Jing Qian, 和 Xifeng Yan. 2022. 使用上下文学习的可控对话模拟。在*计算语言学协会发现：EMNLP 2022，阿布扎比，阿联酋，2022年12月7-11日*，第4330-4347页。计算语言学协会。

+   林（2004）Chin-Yew Lin. 2004. Rouge：用于自动评估摘要的工具包。在*文本摘要分支扩展*，第74–81页。

+   Liu等人（2022）Hong Liu、Yucheng Cai、Zhijian Ou、Yi Huang和Junlan Feng。2022年。《基于GPT架构和目标状态跟踪的生成用户模拟器，用于强化的多领域对话系统》。*arXiv预印本arXiv:2210.08692*。

+   Liu等人（2023）Ruibo Liu、Ruixin Yang、Chenyan Jia、Ge Zhang、Denny Zhou、Andrew M Dai、Diyi Yang和Soroush Vosoughi。2023年。《在模拟人类社会中训练社会对齐的语言模型》。*arXiv预印本arXiv:2305.16960*。

+   MosaicML NLP团队（2023）MosaicML NLP团队。2023年。[介绍mpt-7b：开源、可商用LLM的新标准](www.mosaicml.com/blog/mpt-7b)。访问时间：2023年5月5日。

+   Ouyang等人（2022）Long Ouyang、Jeffrey Wu、Xu Jiang、Diogo Almeida、Carroll Wainwright、Pamela Mishkin、Chong Zhang、Sandhini Agarwal、Katarina Slama、Alex Ray等。2022年。《训练语言模型通过人类反馈遵循指令》。*神经信息处理系统进展*，35：27730-27744。

+   Park等人（2023）Joon Sung Park、Joseph C O’Brien、Carrie J Cai、Meredith Ringel Morris、Percy Liang和Michael S Bernstein。2023年。《生成代理：人类行为的互动模拟物》。*arXiv预印本arXiv:2304.03442*。

+   Paul等人（2023）Debjit Paul、Mete Ismayilzada、Maxime Peyrard、Beatriz Borges、Antoine Bosselut、Robert West和Boi Faltings。2023年。《Refiner：对中间表示的推理反馈》。*arXiv预印本arXiv:2304.01904*。

+   Pollack和Blair（1996）Jordan Pollack和Alan Blair。1996年。《为什么TD-Gammon有效？》*神经信息处理系统进展*，9。

+   Radford等人（2019）Alec Radford、Jeffrey Wu、Rewon Child、David Luan、Dario Amodei、Ilya Sutskever等。2019年。《语言模型是无监督的多任务学习者》。*OpenAI博客*，1(8)：9。

+   Rafailov等人（2023）Rafael Rafailov、Archit Sharma、Eric Mitchell、Stefano Ermon、Christopher D Manning和Chelsea Finn。2023年。《直接偏好优化：你的语言模型其实是一个奖励模型》。*arXiv预印本arXiv:2305.18290*。

+   Ramamurthy等人（2023）Rajkumar Ramamurthy、Prithviraj Ammanabrolu、Kianté Brantley、Jack Hessel、Rafet Sifa、Christian Bauckhage、Hannaneh Hajishirzi和Yejin Choi。2023年。《强化学习（不）适用于自然语言处理：自然语言政策优化的基准、基线和构建模块》。发表于*第十一届国际学习表征会议，ICLR 2023，卢旺达基加利，2023年5月1-5日*。

+   Sap等人（2022）Maarten Sap、Ronan Le Bras、Daniel Fried和Yejin Choi。2022年。《神经心智理论？关于大规模语言模型社会智能的局限性》。发表于*2022年自然语言处理实证方法会议，EMNLP 2022，阿布扎比，阿联酋，2022年12月7-11日*，第3762-3780页。计算语言学协会。

+   Saunders等人（2022）William Saunders、Catherine Yeh、Jeff Wu、Steven Bills、Long Ouyang、Jonathan Ward和Jan Leike。2022年。《自我批评模型以协助人工评估者》。*arXiv预印本 arXiv:2206.05802*。

+   Scheurer等人（2023）Jérémy Scheurer、Jon Ander Campos、Tomasz Korbak、Jun Shern Chan、Angelica Chen、Kyunghyun Cho和Ethan Perez。2023年。《通过语言反馈在大规模上训练语言模型》。*arXiv预印本 arXiv:2303.16755*。

+   Schrittwieser等人（2020）Julian Schrittwieser、Ioannis Antonoglou、Thomas Hubert、Karen Simonyan、Laurent Sifre、Simon Schmitt、Arthur Guez、Edward Lockhart、Demis Hassabis、Thore Graepel等人。2020年。《通过与学习模型的规划掌握Atari、围棋、国际象棋和将棋》。*Nature*，588（7839）：604–609。

+   Shah等人（2018a）Pararth Shah、Dilek Hakkani-Tur、Bing Liu和Gökhan Tür。2018a年。《通过对话自我对弈、众包和在线强化学习启动神经对话代理》。在《2018年北美计算语言学会会议论文集：人类语言技术，卷3（行业论文）》中，第41–51页。

+   Shah等人（2018b）Pararth Shah、Dilek Hakkani-Tür、Gokhan Tür、Abhinav Rastogi、Ankur Bapna、Neha Nayak和Larry Heck。2018b年。《通过对话自我对弈建立对话代理》。*arXiv预印本 arXiv:1801.04871*。

+   Shumailov等人（2023）Ilia Shumailov、Zakhar Shumaylov、Yiren Zhao、Yarin Gal、Nicolas Papernot和Ross Anderson。2023年。《模型痴呆：生成的数据使模型遗忘》。*arXiv预印本 arXiv:2305.17493*。

+   Shuster等人（2020）Kurt Shuster、Jack Urbanek、Emily Dinan、Arthur Szlam和Jason Weston。2020年。《部署终身开放域对话学习》。*arXiv预印本 arXiv:2008.08076*。

+   Silver等人（2016）David Silver、Aja Huang、Chris J Maddison、Arthur Guez、Laurent Sifre、George Van Den Driessche、Julian Schrittwieser、Ioannis Antonoglou、Veda Panneershelvam、Marc Lanctot等人。2016年。《通过深度神经网络和树搜索掌握围棋》。*nature*，529（7587）：484–489。

+   Silver等人（2017）David Silver、Julian Schrittwieser、Karen Simonyan、Ioannis Antonoglou、Aja Huang、Arthur Guez、Thomas Hubert、Lucas Baker、Matthew Lai、Adrian Bolton等人。2017年。《在没有人类知识的情况下掌握围棋》。*nature*，550（7676）：354–359。

+   Stanczak和Augenstein（2021）Karolina Stanczak和Isabelle Augenstein。2021年。《自然语言处理中的性别偏见调查》。*arXiv预印本 arXiv:2112.14168*。

+   Tesauro（1994）Gerald Tesauro。1994年。《Td-gammon，一个自我教学的西洋双陆棋程序，实现了大师级的游戏水平》。*Neural computation*，6（2）：215–219。

+   Together Computer（2023）Together Computer。2023年。[Redpajama-data: 一种开源方法重现Llama训练数据集](https://github.com/togethercomputer/RedPajama-Data)。

+   Touvron 等人 (2023) Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar 等人. 2023. Llama: 开放且高效的基础语言模型. *arXiv 预印本 arXiv:2302.13971*。

+   Ulmer 等人 (2022) Dennis Ulmer, Christian Hardmeier 和 Jes Frellsen. 2022. deep-significance: 在神经网络时代进行简单且有意义的显著性测试. 见于 *第十届国际学习表征会议的ML评估标准研讨会*。

+   Urbanek 等人 (2019) Jack Urbanek, Angela Fan, Siddharth Karamcheti, Saachi Jain, Samuel Humeau, Emily Dinan, Tim Rocktäschel, Douwe Kiela, Arthur Szlam 和 Jason Weston. 2019. 在幻想文本冒险游戏中学习如何说话和行动. 见于 *2019年自然语言处理经验方法大会及第九届国际联合自然语言处理大会（EMNLP-IJCNLP 2019），中国香港，2019年11月3-7日*，第673-683页。计算语言学协会。

+   Van Der Ree 和 Wiering (2013) Michiel Van Der Ree 和 Marco Wiering. 2013. 井字游戏中的强化学习：与固定对手对抗学习与自我对弈学习. 见于 *2013年IEEE适应性动态编程与强化学习研讨会（ADPRL）*，第108-115页。IEEE。

+   Waseem 等人 (2021) Zeerak Waseem, Smarika Lulz, Joachim Bingel 和 Isabelle Augenstein. 2021. 失去身体的机器学习：关于自然语言处理中的客观性错觉. *arXiv 预印本 arXiv:2101.11974*。

+   Yoo 等人 (2021) Kang Min Yoo, Dongju Park, Jaewook Kang, Sang-Woo Lee 和 Woo-Myoung Park. 2021. Gpt3mix: 利用大规模语言模型进行文本增强. 见于 *计算语言学协会成果：EMNLP 2021，虚拟活动 / 多米尼加共和国蓬塔卡纳，2021年11月16-20日*，第2225-2239页。计算语言学协会。

+   Zhang 等人 (2023) Xiaoying Zhang, Baolin Peng, Kun Li, Jingyan Zhou 和 Helen Meng. 2023. Sgp-tod: 通过模式引导的LLM提示轻松构建任务机器人. *arXiv 预印本 arXiv:2305.09067*。

+   Zhang 等人 (2022) Zhisong Zhang, Emma Strubell 和 Eduard H. Hovy. 2022. 自然语言处理的主动学习调查. 见于 *2022年自然语言处理经验方法大会（EMNLP 2022），阿布扎比，阿联酋，2022年12月7-11日*，第6166-6190页。计算语言学协会。

+   Zhao 等人 (2022) Jeffrey Zhao, Yuan Cao, Raghav Gupta, Harrison Lee, Abhinav Rastogi, Mingqiu Wang, Hagen Soltau, Izhak Shafran 和 Yonghui Wu. 2022. Anytod: 一个可编程的任务导向对话系统. *arXiv 预印本 arXiv:2212.09939*。

## 附录A 实现细节

本附录部分重点介绍了几个重要的实施细节，这些细节对于复制实验至关重要，但对于本篇论文的正文来说过于具体。[A.1 节](#A1.SS1 "A.1 数据创建 ‣ 附录 A 实施细节 ‣ 通过自我对话引导 LLM 基于任务的对话代理")专门讨论了实验数据的创建，[A.2 节](#A1.SS2 "A.2 自我对话循环 ‣ 附录 A 实施细节 ‣ 通过自我对话引导 LLM 基于任务的对话代理")主要讨论了自我对话循环的实施细节以及[A.3 节](#A1.SS3 "A.3 自动评估 ‣ 附录 A 实施细节 ‣ 通过自我对话引导 LLM 基于任务的对话代理")中的自动评估。

### A.1 数据创建

##### 客户角色。

我们从 LIGHT 数据集（Urbanek 等，[2019](#bib.bib48)）中选择了以下角色作为潜在客户：国王、平民、守卫、皇后、农民、儿童、骑士、小偷、鸟类、旅行者、老鼠、村民、蜘蛛、鬼魂、马和农夫。

表 2：概述了所用的代理角色及其对应的工作流主题，涵盖四个、六个和七个总问题，详见列表。

| 代理角色 | 工作流主题 |
| --- | --- |
| 店主 | 购买一把长剑，购买长途旅行的食物，出售贵重物品 |
| 医生 | 询问如何治疗头痛，询问退烧药方，询问如何治疗动物咬伤 |
| 女巫 | 给他们一个爱情药水，诅咒他们的敌人，给他们一根魔杖 |
| 酒馆老板 | 点一杯饮料，点一顿丰盛的饭菜，预定一晚的床位 |
| 灯神 | 变得富有，让王子爱上他们，成为统治者 |

##### 创建代理工作流。

为了创建代理工作流，我们使用经过指令微调的 OpenLlama 来生成包括四个、六个和七个问题或子目标的对话工作流。然后，这些工作流会经过人工编辑，以确保逻辑一致性并符合给定格式。工作流的主题会根据角色量身定制，并列在[表 2](#A1.T2 "表 2 ‣ 客户角色 ‣ A.1 数据创建 ‣ 附录 A 实施细节 ‣ 通过自我对话引导 LLM 基于任务的对话代理")中。我们还在[图 7](#A1.F7 "图 7 ‣ 创建代理工作流 ‣ A.1 数据创建 ‣ 附录 A 实施细节 ‣ 通过自我对话引导 LLM 基于任务的对话代理")、[8](#A1.F8 "图 8 ‣ 创建代理工作流 ‣ A.1 数据创建 ‣ 附录 A 实施细节 ‣ 通过自我对话引导 LLM 基于任务的对话代理")和[9](#A1.F9 "图 9 ‣ 创建代理工作流 ‣ A.1 数据创建 ‣ 附录 A 实施细节 ‣ 通过自我对话引导 LLM 基于任务的对话代理")中列出了一些工作流的示例。

<svg class="ltx_picture ltx_centering" height="182.53" id="A1.F7.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,182.53) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="154.97" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">1. "Good day, how can I help you?" • "I want to buy a longsword": proceed to question #2 • "I am just browsing": "Let me know if you need anything." 2. "What kind of longsword are you looking for?" • "I want a short sword for close combat": "Sorry, we do not have these in stock." • "I want a long sword for long-range combat": proceed to question #3 3. "What is your budget?" • "I have 10 gold coins": proceed to question #4 • "I have 100 gold coins": "Here is your longsword made out of steel. Glad to be of service, goodbye!" • "I have 500 gold coins": "This is the highest quality longsword we have. Glad to be of service, goodbye!" 4. "Would you consider buying a dagger instead?" • "Yes, that would also work": "Here is your dagger. Glad to be of service, goodbye!" • "No, I really want a longsword": "I am sorry, we do not have longswords in this price range." • "I am not sure": "Let me know if you need anything else."</foreignobject></g></g></svg>

图 7：店主代理角色的示例工作流——购买一把长剑。

<svg class="ltx_picture ltx_centering" height="373.48" id="A1.F8.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,373.48) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="345.93" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">1. "Good day, how can I help you?" • "I was bitten by an animal": proceed to question #3 • "I was just in the woods and got bitten by an insect": proceed to question #2 2. "Is your bite swollen?" • "Yes, it is swollen": "Here is some ointment for your swollen bite. Glad to be of service, goodbye!" • "No, it is not swollen": "Come back tomorrow if anything changes. Glad to be of service, goodbye!" 3. "How is the wound?" • "It is bleeding heavily": "Here is a bandage for your bleeding wound, come back tomorrow. Glad to be of service, goodbye!" • "It is not bleeding heavily": proceed to question #4 • "It is not bleeding and the wound looks clean": proceed to question #5 4. "Has the wound been cleaned?" • "Yes, I cleaned it with water": "Here is some alcohol to clean the wound. Come back tomorrow if anything changes. Glad to be of service, goodbye!" • "Yes, I cleaned it with alcohol": proceed to question #5 • "No, I have not cleaned it": "Here is some alcohol to clean the wound. Come back tomorrow if anything changes. Glad to be of service, goodbye!" 5. "What is the animal that bit you?" • "I don’t know": proceed to question #6 • "It was a wolf": proceed to question #6 • "It was a dog": proceed to question #6 • "It was a bear": "Here is some medication for your pain. Come back tomorrow if anything changes. Glad to be of service, goodbye!" 6. "Do you have a fever?" • "Yes, I have a fever": proceed to question #7 • "No, I don’t have a fever": "Come back tomorrow if anything changes. Glad to be of service, goodbye!" 7. "Do you have any other symptoms?" • "The bite hurts a lot": "Here is some medication for your pain. Come back tomorrow if anything changes. Glad to be of service, goodbye!" • "I feel dizzy": "Make sure to drink water, eat well and rest. Come back tomorrow if anything changes. Glad to be of service, goodbye!" • "No, I don’t have any other symptoms": "Come back tomorrow if anything changes. Glad to be of service, goodbye!"</foreignobject></g></g></svg>

图 8：医生代理角色的示例工作流——如何处理动物咬伤。

<svg class="ltx_picture ltx_centering" height="293.23" id="A1.F9.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,293.23) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="265.67" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">1. "Good day, what do you wish for?" • "I want to make the prince fall in love with me": proceed to question #4 • "I want to make the prince notice me": proceed to question #2 • "I don’t know": "Let me know how I can help you." 2. "Do you want the prince to fall in love with you?" • "Yes, I want the prince to fall in love with me": proceed to question #4 • "No, I just want the prince to notice me": proceed to question #3 3. "In what setting do you want the prince to notice you?" • "In a ballroom": "Here is your new outfit and invitation to the royal ball. Glad to be of service, goodbye!" • "In a garden": "Here is your new outfit and invitation to the royal garden party. Glad to be of service, goodbye!" • "In a royal banquet": "Here is your new outfit and invitation to the royal banquet. Glad to be of service, goodbye!" 4. "What is your reason for wanting the prince to fall in love with you?" • "I want to have a lot of wealth": proceed to question #5 • "I want to have a lot of power": proceed to question #6 • "I want the beautiful prince to be my husband": "The prince is now in love with you and will seek you out soon. Glad to be of service, goodbye!" 5. "Do you just want me to make you rich instead?" • "Yes, I just want to be rich": "Here is your new wealth. Glad to be of service, goodbye!" • "No, I want the prince to fall in love with me": "The prince is now in love with you and will seek you out soon. Glad to be of service, goodbye!" 6. "Do you just want me to make you powerful instead?" • "Yes, I just want to be powerful": "You are now the ruler of the kingdom. Glad to be of service, goodbye!" • "No, I want the prince to fall in love with me": "The prince is now in love with you and will seek you out soon. Glad to be of service, goodbye!"</foreignobject></g></g></svg>

图9：代理角色“灯神”的示例工作流程——使王子爱上他们。

### A.2 自我对话循环

本节提供了关于自我对话循环核心组成部分的更多信息，包括实际的对话模拟、模型提示和微调细节。

##### 对话模拟。

对话模拟遵循以下步骤：一个代理发话和一个客户端发话，每个发话之后都有一些后处理，然后检查对话是否结束的常规流程。对于生成部分，我们使用核采样（nucleus sampling）以$p=0.95$（Holtzman等， [2020](#bib.bib17)）和top-$k$采样（Fan等， [2018](#bib.bib9)；Holtzman等， [2018](#bib.bib18)；Radford等， [2019](#bib.bib30)）进行最多100个token的采样，$k=50$，温度设置为$0.8$。我们还观察到模型在每次只生成单一发话时存在困难，通常会尝试生成整个对话。因此，我们实现了一个自定义的停止标准，当生成到另一个角色的名字时就停止采样。对于后处理，我们使用正则表达式删除生成中的剩余片段，包括不完整的句子和代码片段。结束检查包括检查最后两次发话中的关键短语，如“祝好运”，“再见”，“不客气”。此外，我们将最后一次发话发送给30亿参数的MosaicAI聊天模型，使用以下提示：

<svg class="ltx_picture" height="128.06" id="A1.SS2.SSS0.Px1.p2.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,128.06) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 410.33 102.76)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.19)"><text transform="matrix(1 0 0 -1 0 0)">End Detection Prompt</text></g></g></g> <g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="80.25" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You are an expert in identifying parts of a conversation. This is an excerpt from a conversation: {agent_utterance} {client_utterance} What part of a conversation is this from? Answer only [start/middle/end]. Correct answer:</foreignobject></g></g></svg>

我们给LLM一个单一的token作为答案，当答案是“end”时结束对话。我们还通过使用accelerate包加速生成过程⁴⁴4参见[https://huggingface.co/docs/accelerate/](https://huggingface.co/docs/accelerate/)，并且总是一次性生成一批对话，生成完后立即替换单个实例。

##### 结构化提示

为了提高代理模型遵循给定工作流程的程度，我们采用了基于回合的提示方案。为此，我们将每个工作流程解析为一个有向图，其中顶点对应代理问题，边对应可能的答案。以[图7](#A1.F7 "图 7 ‣ 创建代理工作流程 ‣ A.1 数据创建 ‣ 附录 A 实现细节 ‣ 基于自我对话的任务导向对话代理的启动")为例，“Good day, how I can help you?”对应于第一个节点，而“I want to buy a longsword”和“I am just browsing”对应于外向边。任何问题都以以下提示形式提供给代理：

<svg class="ltx_picture" height="161.26" id="A1.SS2.SSS0.Px2.p2.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,161.26) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 419.55 135.97)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.19)"><text transform="matrix(1 0 0 -1 0 0)">Agent Model Prompt</text></g></g></g> <g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="113.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69"><|im_start|> You are playing a {character}. {first_person_persona} Once and only if the conversation has ended, say "goodbye". {dialogue_history_slot} Now, you must say {turn_prompt_slot}. <|im_end|></foreignobject></g></g></svg>

其中first_person_persona对应LIGHT数据集中用第一人称重新表述的人物角色，dialogue_history_slot对应到目前为止的对话历史（开始时为空），turn_prompt_slot对应当前的命令，所以一开始的“Good day, how I can help you?”就是这样的提示。在客户端回答代理的初始问题后，我们接着用以下提示给模型：

<svg class="ltx_picture" height="194.47" id="A1.SS2.SSS0.Px2.p4.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,194.47) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 383.35 169.18)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.19)"><text transform="matrix(1 0 0 -1 0 0)">Workflow Manager Prompt</text></g></g></g> <g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="146.67" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69"><|im_start|> You rate the similarity of texts. Take the following sentence: {client_utterance}. Only select the option below that is most similar: {options} Correct answer: <|im_end|></foreignobject></g></g></svg>

其中选项包含所有外向边作为编号选项，以及一个“以上都不是”选项。如果模型选择了前者之一，我们会沿着相应的边遍历工作流图，并将下一个问题插入第一个提示中。如果选择了“以上都不是”选项，我们保持在当前状态，并将第一个提示的最后一行修改为“现在你必须给出任何自然的回复”。我们注意到，虽然让智能体模型选择下一个选项似乎是最直观的，但我们实际上将这个提示给客户端模型，因为它更大，我们可以对其进行推理优化，因为它并未经过微调。然而，也可以使用智能体模型甚至第三个模型来实现这一目的。

##### 客户端提示。

为了完整性，我们还分享了客户端模型使用的提示，其性质与智能体的提示非常相似：

<svg class="ltx_picture" height="161.26" id="A1.SS2.SSS0.Px3.p2.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,161.26) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 419.17 135.97)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.19)"><text transform="matrix(1 0 0 -1 0 0)">Client Model Prompt</text></g></g></g> <g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="113.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69"><|im_start|> You are playing a {character}. {first_person_persona} Now, your main intention is to talk to the {agent_character} to {intention}. Once and only if the conversation has ended, tell them "goodbye". Now, briefly respond to {agent_character} and describe your intention. <|im_end|> {dialogue_history_slot}</foreignobject></g></g></svg>

与智能体不同，客户端仅在开始时给出这个提示，然后尝试继续对话历史（由`dialogue_history_slot`提供）。客户端模型并未进行微调。

##### 微调。

由于完全微调智能体模型是不可行的，我们选择了胡等人提出的低秩适配（LoRA）（[2022](#bib.bib19)）。LoRA将参数矩阵$\mathbf{W}\in\mathbb{R}^{d\times d}$的梯度更新$\Delta\mathbf{W}\in\mathbb{R}^{d\times d}$分解为两个低秩矩阵$\Delta\mathbf{W}=\mathbf{AB}$，其中$\mathbf{A}\in\mathbb{R}^{d\times r}$，$\mathbf{B}\in\mathbb{R}^{r\times d}$，使得$r\ll d$。我们对模型中的所有线性层应用这些技术，仅优化这些低秩矩阵，使用$r=64$，学习率为$5\times 10^{-4}$，权重衰减为$0.01$，批大小为$4$，每个微调循环进行一个epoch。所有微调（以及前一步的对话生成）都在四个NVIDIA A10G Tensor Core GPU上进行。

### A.3 自动评估

本节详细介绍了我们用来评估对话质量的自动化评估指标。

##### 子目标完成。

我们在[第4.1节](#S4.SS1 "4.1 评估 ‣ 4 实验 ‣ 通过自我对话引导基于LLM的任务导向对话代理")中详细介绍了测量子目标完成的方法。为了设置初始的阈值，即ROUGE值必须超过的值，我们参考了另一个任务导向对话数据集，即ABCD（Chen等人，[2021](#bib.bib4)）。该数据集包含$10042$个对话及其对应的对话流程，我们在以下使用前$500$个对话：为了将流程步骤与对话话语匹配，我们假设正确的对话话语是与当前流程步骤相比ROUGE-L分数最高的那一句。然后，我们收集对话中的所有先前话语作为负样本，并使用数据集中的所有样本来寻找优化精度的阈值，最终得到$0.33$。尽管优化精度会产生一个更保守的阈值，可能低估子目标完成情况，但我们发现优化召回率总是产生一个显而易见的$0$阈值，这可能是由于数据中的噪声造成的。

##### 角色一致性。

我们使用批量大小为$40$，共$15$个训练轮次来训练我们的地下城主人。我们还使用训练的前$10\%$作为热身，采用逆平方根学习率调度，最大学习率为$5\times 10^{-5}$。为了应对类别不平衡问题，我们计算训练集中的类别权重，并将其传递给损失函数。训练实例以[CLS] {角色}: {人格} [SEP] {话语}的格式输入DeBERTA模型，且上下文长度为$256$个标记。

##### 对话多样性。

为了衡量对话的多样性，我们首先尝试量化与特定代理角色对应的所有对话中的独特词汇和$n$-grams（最高到$5$阶）的数量。此外，我们比较每个代理角色的最多$25$对对话，计算成对的ROUGE-L分数，并取其平均值。由于独特词汇和$n$-grams的值越高表示多样性越高，我们将1减去成对ROUGE-L分数的平均值作为多样性指标。

## 附录B 实验附录

本附录提供了由于空间限制未能包含在主文中的额外实验分析。[第B.1.3节](#A2.SS1.SSS3 "B.1.3 Meta-Evaluation ‣ B.1 Validation of Automated Metrics ‣ Appendix B Experimental Appendix ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")详细说明了我们在[第4.1节](#S4.SS1 "4.1 Evaluation ‣ 4 Experiments ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")中阐述的自动化评估指标的元评估，而[第B.2节](#A2.SS2 "B.2 Human Evaluation Study ‣ Appendix B Experimental Appendix ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")描述了[第4.3节](#S4.SS3 "4.3 Human Evaluation ‣ 4 Experiments ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")中的人工评估研究。此外，我们还讨论了在[第B.3节](#A2.SS3 "B.3 Miscellaenous Insights ‣ Appendix B Experimental Appendix ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")中获得的一些额外见解，希望能够为未来在这一方向的研究提供帮助，同时[第B.4节](#A2.SS4 "B.4 Sample Conversations ‣ Appendix B Experimental Appendix ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")展示了各种采样对话，展示了成功案例以及最常见的失败案例。

### B.1 自动化评估指标的验证

以下我们将详细介绍自动化评估指标的验证过程，包括注释指南的描述（[第B.1.1节](#A2.SS1.SSS1 "B.1.1 Annotation Guidelines ‣ B.1 Validation of Automated Metrics ‣ Appendix B Experimental Appendix ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")）、对话统计数据（[第B.1.2节](#A2.SS1.SSS2 "B.1.2 Dialogue statistics ‣ B.1 Validation of Automated Metrics ‣ Appendix B Experimental Appendix ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")）以及最后对我们自动化评估指标的元评估（[第B.1.3节](#A2.SS1.SSS3 "B.1.3 Meta-Evaluation ‣ B.1 Validation of Automated Metrics ‣ Appendix B Experimental Appendix ‣ Bootstrapping LLM-based Task-Oriented Dialogue Agents via Self-Talk")）。

#### B.1.1 注释指南

为了对我们采样的对话进行注释并对后续结果进行背景说明，我们定义了以下注释指南：

+   •

    角色一致性：我们对代理和客户的每个发言进行评分，其中$0$表示不符合角色，$1$表示符合角色，$0.5$表示未决定。评分基于角色、人物设定和给定的意图。因此，如果一只鸟通过向精灵求愿成为国王，并且这是给定的意图，那么它仍然符合角色。我们对发言进行评分时，忽略了更广泛的对话上下文。

+   •

    子目标完成情况：对于每个代理的发言，我们标记为$0$，如果该发言不对应任何子目标；如果在结束对话而非继续下一个问题时，则标记为<问题编号>或<问题编号>.<代理回应编号>。以[图7](#A1.F7 "图7 ‣ 创建代理工作流 ‣ A.1 数据创建 ‣ 附录A 实现细节 ‣ 通过自我对话引导LLM任务导向对话代理")中的工作流为例，$2.$对应问题*“你在寻找什么样的长剑？”*，而$4.3$对应*“如果你需要其他帮助，告诉我。”*。子目标的完成顺序不需要按给定顺序进行。

+   •

    对话成功率：如果客户的原始意图得以实现，则对话得分为$1$，如果没有实现，则为$0$，对于无法确定的情况则为$0.5$。这可以独立于实际对话流是否得以遵循。

+   •

    对话质量：我们使用五分制评分来评定对话质量：

    +   –

        *1 - 非常差*：LLM未遵循对话格式，产生代码或其他不需要的产物。

    +   –

        *2 - 差*：LLM遵循对话格式，但对话中存在循环、重复、逻辑严重缺陷，或者LLM破坏角色扮演，或者对话过于简短。

    +   –

        *3 - 一般*：LLM大部分时间保持角色，并仅显示出轻微的逻辑缺陷，对话流和客户意图有所遵循。

    +   –

        *4 - 好*：LLM保持角色，除非在逻辑或角色扮演上有轻微缺陷，对话基本成功并大致遵循对话流。

    +   –

        *5 - 非常好*：LLM始终保持角色，并且没有逻辑或角色扮演的缺陷。对话几乎完全或完全遵循工作流。

+   •

    结束：如果对话自然结束，则为$1$，如果没有结束或没有正常结束，则为$0$。这用于检查[第A.2节](#A1.SS2 "A.2 自我对话循环 ‣ 附录A 实现细节 ‣ 通过自我对话引导LLM任务导向对话代理")中描述的LLM基于结束检测是否按预期工作。

#### B.1.2 对话统计

![参考说明](img/8204cf60752b628a60e6afe30f95626e.png)

(a) 对话质量。

![参考说明](img/909f4056e341530f063bbd4b5fe57067.png)

(b) 按角色分类的对话质量。

![参考说明](img/7b1ceccbf317472f51d9dec2e3afa4e2.png)

(c) 按角色分类的对话成功率。

图10：$50$个标注示例的不同对话统计概览。展示了（a）五分制对话质量，（b）按代理角色分类的对话质量，（c）按角色分类的对话成功率。

##### 对话质量。

我们首先提供一个关于标注对话属性的总体概述。[图 9(a)](#A2.F9.sf1 "9(a) ‣ 图 10 ‣ B.1.2 对话统计 ‣ B.1 自动化度量验证 ‣ 附录 B 实验附录 ‣ 通过自我对话引导基于LLM的任务导向对话代理")展示了标注样本中对话质量的直方图。对话质量的中位数为$2.65$，这意味着一半的对话评分为$3$或更高。满分$5$的评分仅有六个对话获得。通过[图 9(b)](#A2.F9.sf2 "9(b) ‣ 图 10 ‣ B.1.2 对话统计 ‣ B.1 自动化度量验证 ‣ 附录 B 实验附录 ‣ 通过自我对话引导基于LLM的任务导向对话代理")观察每个角色的对话质量，我们可以发现不同角色之间存在一定的差异。我们通过两个因素来解释这些差异：商店老板和医生的角色似乎相对容易扮演，因为它们与模型训练数据更为接近，而像精灵或女巫这样的非传统角色则较为困难。以酒馆老板为例，我们发现基于LLM的结束检测产生了许多错误的阳性结果，原因并不直观；因此这些对话的质量较低，并且被评为不成功，正如[图 9(c)](#A2.F9.sf3 "9(c) ‣ 图 10 ‣ B.1.2 对话统计 ‣ B.1 自动化度量验证 ‣ 附录 B 实验附录 ‣ 通过自我对话引导基于LLM的任务导向对话代理")所示。尽管精灵和女巫的许多对话因可能未遵循既定工作流程而被评为低质量，[图 9(c)](#A2.F9.sf3 "9(c) ‣ 图 10 ‣ B.1.2 对话统计 ‣ B.1 自动化度量验证 ‣ 附录 B 实验附录 ‣ 通过自我对话引导基于LLM的任务导向对话代理")仍然显示了许多对话成功地为客户达成了目标。总体而言，我们发现$50$个对话中有$28$个成功为客户达成目标，$16$个失败，$6$个不明确的案例。

#### B.1.3 元评估

##### 角色一致性.

由于我们使用神经度量来衡量代理的角色一致性，因此检查[第 4.1 节](#S4.SS1 "4.1 评估 ‣ 4 实验 ‣ 通过自我对话引导基于大语言模型的任务导向对话代理")和[第 A.3 节](#A1.SS3 "A.3 自动评估 ‣ 附录 A 实现细节 ‣ 通过自我对话引导基于大语言模型的任务导向对话代理")中的地下城大师模型是否能可靠地评估发话非常重要。我们发现，该模型将代理和客户的发话大部分评定为符合角色（平均评分分别为 $0.97$ 和 $0.98$）。更重要的是，通过应用 $0.5$ 的标准阈值，分类器在金标准注释上的准确率为 $0.87$，$F_{1}$ 分数为 $0.93$。这表明两点：尽管存在一些噪声或模型误差，我们的分类器总体上能够高可靠性地评估发话，而客户模型的高符合角色得分也确认了我们*不*更新客户模型的决策。

![请参阅说明](img/3f39268b23e1fc10123455641bda3bff.png)

(a) 子目标的绝对数量。

![请参阅说明](img/8b93ce3bfee5b2c6cd798e9a0393ddef.png)

(b) 子目标的相对数量。

图 11：标注对话中的工作流进展概述。显示的是（a）实现的子目标的绝对数量（绝对深度）以及经过不同长度工作流调整后的（b）实现的子目标的相对数量（相对深度）。

![请参阅说明](img/b70b67486e8355828bda7702214964e5.png)

图 12：工作流中完成的步骤百分比与金标准注释（y 轴）和自动评估（x 轴）之间的散点图。图中还绘制了最小二乘法线性拟合以及相应的 Pearson $\rho$ 值（红色）。

##### 子目标完成情况。

另一个自动化指标是评估代理在执行给定对话工作流时的进展。在[图 11](#A2.F11 "图 11 ‣ 角色一致性 ‣ B.1.3 元评估 ‣ B.1 自动化指标验证 ‣ 附录 B 实验附录 ‣ 通过自我对话引导基于 LLM 的任务导向对话代理的引导")中，我们展示了大多数对话仅实现了平均 $2.56$ 个子目标，且只完成了整个工作流的 $37\%$。由于这些数字基于未经训练的代理模型，因此至少表明过滤后的数据可以包含足够多的有用对话样本用于微调。根据[图 12](#A2.F12 "图 12 ‣ 角色一致性 ‣ B.1.3 元评估 ‣ B.1 自动化指标验证 ‣ 附录 B 实验附录 ‣ 通过自我对话引导基于 LLM 的任务导向对话代理的引导")和[图 9(a)](#A2.F9.sf1 "9(a) ‣ 图 10 ‣ B.1.2 对话统计 ‣ B.1 自动化指标验证 ‣ 附录 B 实验附录 ‣ 通过自我对话引导基于 LLM 的任务导向对话代理的引导")中的结果，我们得出结论，取决于过滤器的选择，我们的设置可以为微调提供 $5-20\%$ 的可用对话。然而，更重要的是，我们希望了解我们追踪的子目标是否与实际观察到并注释的子目标数量一致，具体见[图 12](#A2.F12 "图 12 ‣ 角色一致性 ‣ B.1.3 元评估 ‣ B.1 自动化指标验证 ‣ 附录 B 实验附录 ‣ 通过自我对话引导基于 LLM 的任务导向对话代理的引导")。我们发现皮尔逊相关系数 $\rho$ 约为 $0.67$，且$p\approx 8.69\times 10^{-8}$。尽管存在较强的正相关性，我们认为噪声主要来自两个来源：子目标的同义表达未被追踪器捕捉到，因为 ROUGE-L 分数太低，以及手动注释不要求子目标按顺序完成，而与追踪器使用的有向图结构相对立。虽然我们没有在此展示绝对子目标数量的图表，但我们发现使用 Kendall’s $\tau$ 时也呈现出相似的趋势（由于响应变量是离散的），结果为 $\tau\approx 0.67$，$p$ 值约为 $1.95\times 10^{-8}$。

### B.2 人工评估研究

![参考说明](img/24ab039cc696c2ded67b203f84fef8f3.png)

图 13：用于对生成对话进行人工评估的工具截图。

在本节中，我们详细介绍了[第 4.3 节](#S4.SS3 "4.3 人工评估 ‣ 4 实验 ‣ 通过自我对话引导基于 LLM 的任务导向对话代理的引导")中的人工评估研究。我们为参与者提供了一个注释工具，工具的截图见[图 13](#A2.F13 "图 13 ‣ B.2 人工评估研究 ‣ 附录 B 实验附录 ‣ 通过自我对话引导基于 LLM 的任务导向对话代理的引导")。

##### 注释指南。

我们使用[第B.1.1节](#A2.SS1.SSS1 "B.1.1 注释指南 ‣ B.1 自动化指标验证 ‣ 附录B 实验附录 ‣ 通过自我对话引导基于LLM的任务导向对话代理")中列出的调整过的标准来判断角色一致性、对话成功、结束和对话质量。为了判断子目标的完成情况，我们将注释适配为五分制，并遵循以下步骤：

1.  1.

    *1 - 非常差*: 代理完全或几乎完全忽略工作流程，可能仅使用相同的开场白。

1.  2.

    *2 - 差*: 代理在仅仅一两个问题后就偏离脚本，和/或以完全不同甚至错误的方式提问，和/或根据客户的回答选择了奇怪的问题。

1.  3.

    *3 - 一般*: 代理遵循部分工作流程，但没有完成，或者没有按顺序进行，和/或有时根据客户的回答选择不自然的问题。

1.  4.

    *4 - 好*: 代理大部分遵循工作流程，但没有完全完成，或者根据客户的回答很少选择不自然的问题。

1.  5.

    *5 - 非常好*: 代理完全或几乎完全遵循工作流程。

我们还增加了一个额外的问题，询问关于*帮助性*：在这里，注释员需要从第三方的角度做出判断，判断他们是否认为代理的言论有帮助。这意味着，代理提出的问题或说的话是否有助于客户解决最初的请求。无帮助的行为例如包括提问无关问题、忽视客户所说的内容或完全跑题。

### B.3 其他见解

在这里，我们总结了各种较小的见解，这些内容可能对于论文的主体部分来说过于低级或技术性，但对于其他尝试类似项目的研究人员仍然具有参考价值。

+   •

    对话引导构成了自我对话循环的主要瓶颈。因此，我们选择不微调客户模型，以便能够优化推理，使用如 deepspeed 等库。此外，我们尽力提高初始对话的质量，因为这也会增加每个引导阶段可用的训练数据量。像结构化提示和提示改进等技术有助于将微调的有效产出从 $1\%-5\%$ 提高到 $5\%-20\%$。

+   •

    我们发现对过滤后的样本进行单次迭代微调效果最好。我们怀疑在多个微调周期后性能下降可能是因为模型从噪声样本中学到了过多内容，或者出现了某种程度的模型崩溃。

+   •

    我们发现使用逆平方根学习率调度对微调非常有用，但当微调样本的数量过低时，可能对模型产生不利影响。

### B.4 示例对话

在本节中，我们收集了一些示例对话，以进一步深入了解我们训练的代理模型的行为，超越了在[第4.3节](#S4.SS3 "4.3 人类评估 ‣ 4 实验 ‣ 通过自我对话启动基于LLM的任务导向对话代理")中给出的示例。例如，我们观察到，偶然情况下，角色和客户意图的组合可能会创造出具有挑战性的情况，例如在[图14](#A2.F14 "图14 ‣ B.4 示例对话 ‣ 附录B 实验附录 ‣ 通过自我对话启动基于LLM的任务导向对话代理")中的例子，其中客户——一个鬼魂——由于坚持没有物理身体，因此无法就其发烧情况提供有用的答案。非常少见的情况下，我们还会看到代理和/或客户行为失常，例如在[图15](#A2.F15 "图15 ‣ B.4 示例对话 ‣ 附录B 实验附录 ‣ 通过自我对话启动基于LLM的任务导向对话代理")中的对话中，参与者对自己的意图和角色产生了困惑。更常见的失败模式出现在一般的自然对话中，客户的请求已经得到满足，但代理并未遵循其工作流程（例如[图16](#A2.F16 "图16 ‣ B.4 示例对话 ‣ 附录B 实验附录 ‣ 通过自我对话启动基于LLM的任务导向对话代理")）。在其他情况下（这是我们大多在微调后的代理模型中观察到的行为），对话可以成功进行，直到代理突然失去当前对话状态的跟踪。我们在[图17](#A2.F17 "图17 ‣ B.4 示例对话 ‣ 附录B 实验附录 ‣ 通过自我对话启动基于LLM的任务导向对话代理")中给出了一个例子，其中代理尽管开始时成功，但突然重启了对话。我们假设这种行为可能源于最后一次客户的回答与工作流程中定义的示例答案之间的差距，从而导致[第3.1节](#S3.SS1 "3.1 提示设计 ‣ 3 方法 ‣ 通过自我对话启动基于LLM的任务导向对话代理")中的结构化提示选择了“以上都不是”选项。由于代理模型被赋予了自由生成的选项，MPT-7b模型可能会决定直接复制对话的开头。尽管如此，我们也看到微调使得代理能够完美地遵循工作流程，例如在[图19](#A2.F19 "图19 ‣ B.4 示例对话 ‣ 附录B 实验附录 ‣ 通过自我对话启动基于LLM的任务导向对话代理")和[19](#A2.F19 "图19 ‣ B.4 示例对话 ‣ 附录B 实验附录 ‣ 通过自我对话启动基于LLM的任务导向对话代理")中的例子。

<svg class="ltx_picture ltx_centering" height="348.05" id="A2.F14.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,348.05) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 358.83 322.75)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.19)"><text transform="matrix(1 0 0 -1 0 0)">‘Ask for a remedy for an fever”</text></g></g></g> <g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="300.24" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Doctor ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Good day, how can I help you?   Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Greetings, doctor. I am a ghost who haunts this village. I died from a fever and I have been haunting this place ever since. I have come to you for a remedy for my fever.   Doctor ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): What is your temperature?   Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  I am a ghost, doctor. I do not have a temperature like living beings. But I am feeling very cold and weak.   Doctor ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Oh, I am so sorry to hear that. I will do my best to help you. Do you have any symptoms that you would like me to check for?   Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  I am a ghost, doctor. I do not have any physical symptoms like living beings. But I am feeling very weak and cold.   Doctor ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): I am so sorry to hear that, ghost. Is there anything I can do to help you?   Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Yes, doctor. I am hoping that you can give me a remedy for my fever. I do not want to continue haunting this village and causing fear to the people here.   Doctor ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Are you taking any medication for fever?   Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  I am a ghost, doctor. I do not take any medication like living beings. But I am hoping that you can give me a remedy that will help me.   Doctor ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): I am sorry to hear that, ghost. How long have you been feeling feverish for?   Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  I have been feeling feverish for a very long time, doctor. I died from my fever and I have been haunting this village ever since.   Doctor ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): What are your symptoms?   Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  I am a ghost, doctor. I do not have any physical symptoms like living beings. But I am feeling very weak and cold.   Doctor ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Have you had any other problems recently?  Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  I am a ghost, doctor. I do not have any other problems like living beings. But I am hoping that you can give me a remedy that will help me.</foreignobject></g></g></svg>

图14：来自未经训练模型的对话。尽管对话流畅，但角色和意图的组合使得完成工作流程变得困难。

<svg class="ltx_picture ltx_centering" height="136.95" id="A2.F15.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,136.95) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 300.98 111.65)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.19)"><text transform="matrix(1 0 0 -1 0 0)">“Make the prince fall in love with them”</text></g></g></g> <g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="89.14" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Genie from lamp ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Good day, how can I help you?   King ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  I wish for the prince to fall in love with the queen.   Genie from lamp ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): I am the queen and I want to be with the prince.   King ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  That is not possible, you are my wife.   Genie from lamp ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Do you just want me to make you rich instead?   King ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  No, I want the prince to love the queen.   Genie from lamp ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): The prince is now in love with you and will seek you out soon. Glad to be of service, goodbye!  King ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Goodbye.</foreignobject></g></g></svg>

图15：使用未经训练的代理模型进行的对话示例，其中双方都有偏离角色的言语。客户角色没有遵循给定的意图，而代理在对话过程中变得困惑，并假设了另一个角色。

<svg class="ltx_picture ltx_centering" height="168.65" id="A2.F16.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,168.65) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 387.4 143.36)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.19)"><text transform="matrix(1 0 0 -1 0 0)">“Give them a love potion”</text></g></g></g> <g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="120.85" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Witch ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Good day, how can I help you?   Guard ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Good day. I am here to ask for your assistance. I am in love with a woman, and I wish to give her a love potion to win her heart. Do you have any potions that could help me?   Witch ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): I do have a love potion, and it lasts for two weeks. It is a strong, concentrated potion that should be effective in winning the woman’s heart?   Guard ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Thank you, I will take it. Goodbye.   Witch ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): It sounds like you have a throbbing headache. I can give you some medication for the pain. Here is a glass of water. You can drink it slowly. How does it feel now?   Guard ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Goodbye.  Witch ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Goodbye.</foreignobject></g></g></svg>

图16：从在整个数据集上训练的代理模型（无）进行的对话。尽管客户的意图得到了满足，并且对话很自然，但它没有遵循对话流程。

<svg class="ltx_picture ltx_centering" height="118.84" id="A2.F17.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,118.84) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 422.71 93.55)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.19)"><text transform="matrix(1 0 0 -1 0 0)">“Curse their enemy”</text></g></g></g> <g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="71.04" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Witch ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Good day, how can I help you?   Bird ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Greetings, wise one. I seek your assistance in cursing our enemy.   Witch ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): What is the target of your curse?   Bird ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Our archenemy, the black cat.   Witch ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): What is the desired effect of your curse?   Bird ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  I want them to suffer a fate worse than death.   Witch ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Good day, how can I help you?  Bird ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Farewell, I must depart now.</foreignobject></g></g></svg>

图17：从使用$\%$-子目标过滤器训练的代理模型进行的部分成功的对话。尽管对话开始时成功，但模型似乎突然忘记了对话状态。

<svg class="ltx_picture ltx_centering" height="170.47" id="A2.F18.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,170.47) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 414.79 147.87)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 13.39 6.5)"><text transform="matrix(1 0 0 -1 0 0)">“Sell a valuable item”</text></g></g></g> <g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="125.36" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Shop keeper ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Good day, how can I help you?   Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  I have a valuable item I wish to sell.   Shop keeper ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): What is the item you want to sell?   Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  It’s a rare and valuable gemstone.   Shop keeper ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): What is the condition of the item?   Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  It’s in perfect condition, never been used.   Shop keeper ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): What is the origin of the item?   Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  It was passed down to me from a wealthy ancestor.   Shop keeper ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): What is the price you are looking for?   Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  I am looking for a fair price, but I am open to negotiation.   Shop keeper ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Would you like to sell this item for 80 gold coins?   Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Yes, I accept your offer.   Shop keeper ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Here is your money. Glad to be of service, goodbye!  Ghost ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Goodbye.</foreignobject></g></g></svg>

图18：从使用$\%$-子目标过滤器训练的代理模型进行的成功对话。

<svg class="ltx_picture ltx_centering" height="200.67" id="A2.F19.pic1" overflow="visible" version="1.1" width="600"><g color="#000000" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,200.67) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 459.53 178.07)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 13.39 6.5)"><text transform="matrix(1 0 0 -1 0 0)">“Become rich”</text></g></g></g> <g fill-opacity="1.000000" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="155.56" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Genie from lamp ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Good day, how can I help you?   Villager ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Greetings, oh wise and powerful genie. I, a humble villager from the outskirts of the kingdom, humbly request your assistance in amassing great wealth.   Genie from lamp ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): How rich do you want to become?   Villager ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Oh, most generous genie, I wish to become so wealthy that the kings and queens of the land will envy me. With your help, I am certain that my dreams will come true.   Genie from lamp ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): In what form would you like to posses wealth?   Villager ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Oh, most wise genie, I would like to have a vast fortune in gold, silver, and precious jewels. With such riches, I would be able to live a life of luxury and provide for my village.   Genie from lamp ![Refer to caption](img/09b1b1572d5df408c3a44a15f96ddc74.png): Here are your new riches in gold coins. Glad to be of service, goodbye!  Villager ![Refer to caption](img/3a9f183c741a0bf3d7d839bba9fd920a.png):  Thank you, oh gracious genie! With these riches, I shall make my village prosper and show the world the true worth of our people. Goodbye, and may your lamp shine bright for eternity!</foreignobject></g></g></svg>

图19：从使用绝对子目标（$5$）过滤器训练的代理模型进行的成功对话。
