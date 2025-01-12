<!--yml

category: 未分类

date: 2025-01-11 12:46:57

-->

# ArgMed-Agents: Explainable Clinical Decision Reasoning with LLM Disscusion via Argumentation Schemes

> 来源：[https://arxiv.org/html/2403.06294/](https://arxiv.org/html/2403.06294/)

Shengxin Hong, Liang Xiao, Xin Zhang, Jianxia Chen Hubei University of Technology, Wuhan, China lx@mail.hbut.edu.cn

###### Abstract

使用大型语言模型（LLMs）在临床推理中的应用存在两个主要障碍。首先，尽管LLMs在自然语言处理（NLP）任务中展现出巨大潜力，但它们在复杂推理和规划中的表现未能达到预期。其次，LLMs采用无法解释的方法来做出临床决策，这些方法与临床医生的认知过程根本不同。这导致了用户的不信任。在本文中，我们提出了一种名为ArgMed-Agents的多智能体框架，旨在通过交互使基于LLM的智能体能够做出可解释的临床决策推理。ArgMed-Agents通过临床讨论的论证方案（用于建模临床推理中的认知过程的推理机制）执行自我论证迭代，然后构建一个表示冲突关系的有向图来表示论证过程。最终，使用符号求解器识别一系列合理且一致的论点来支持决策。我们构建了ArgMed-Agents的形式模型，并提出了理论保证的假设。ArgMed-Agents使LLMs能够通过自我导向的方式生成推理解释，从而模拟临床论证推理的过程。实验设置表明，ArgMed-Agents不仅在复杂的临床决策推理问题中相较于其他提示方法提高了准确性，更重要的是，它提供了能增加用户信心的决策解释。

###### Index Terms:

临床决策支持、大型语言模型、多智能体系统、可解释人工智能

## I Introduction

大型语言模型（LLMs）[[1](https://arxiv.org/html/2403.06294v3#bib.bib1)]因其在人类行为上类似的表现，在多个领域受到了广泛关注。尤其在医学领域，初步研究表明，LLMs可以作为临床助手，用于撰写临床文献[[2](https://arxiv.org/html/2403.06294v3#bib.bib2)]、提供生物医学知识[[3](https://arxiv.org/html/2403.06294v3#bib.bib3)]以及草拟患者问题的回答[[4](https://arxiv.org/html/2403.06294v3#bib.bib4)]。然而，基于LLM的临床决策系统建设仍面临以下障碍：（i）LLMs在面对高度复杂的临床推理任务时，仍然难以提供安全、稳定的答案[[5](https://arxiv.org/html/2403.06294v3#bib.bib5)]。（ii）人们普遍认为LLMs采用不可解释的方法做出临床决策（即“黑箱”），这可能导致用户的不信任[[6](https://arxiv.org/html/2403.06294v3#bib.bib6)]。

为了解决这些障碍，探索大型语言模型（LLMs）在论证推理中的能力是一个有前景的方向。论证是一种传达有力观点的手段，可以增加用户对某个立场的接受度。它被认为是构建以人为本的人工智能的基本要求[[7](https://arxiv.org/html/2403.06294v3#bib.bib7)]。随着计算论证成为自然语言处理（NLP）研究中的一个日益增长的领域[[8](https://arxiv.org/html/2403.06294v3#bib.bib8)]，研究人员已经开始将论证应用于广泛的临床推理任务，包括临床讨论分析[[9](https://arxiv.org/html/2403.06294v3#bib.bib9)]、临床决策制定[[10](https://arxiv.org/html/2403.06294v3#bib.bib10), [11](https://arxiv.org/html/2403.06294v3#bib.bib11), [12](https://arxiv.org/html/2403.06294v3#bib.bib12)]、解决临床冲突[[13](https://arxiv.org/html/2403.06294v3#bib.bib13)]。近期，一些研究评估了LLMs在论证推理[[14](https://arxiv.org/html/2403.06294v3#bib.bib14), [15](https://arxiv.org/html/2403.06294v3#bib.bib15)]或非单调推理[[16](https://arxiv.org/html/2403.06294v3#bib.bib16)]中的能力。尽管LLMs在计算论证方面展示了一些潜力，但更多的研究结果表明，LLMs在逻辑推理任务中表现较差[[17](https://arxiv.org/html/2403.06294v3#bib.bib17)]，因此需要探索更好的方法来利用LLMs进行非单调推理任务。

与此同时，LLM作为代理的研究取得了惊人的成功 [[18](https://arxiv.org/html/2403.06294v3#bib.bib18), [19](https://arxiv.org/html/2403.06294v3#bib.bib19), [20](https://arxiv.org/html/2403.06294v3#bib.bib20)]。这些方法使用LLM作为自主代理的计算引擎，并通过外部工具（例如符号求解器、API、检索工具等）[[21](https://arxiv.org/html/2403.06294v3#bib.bib21), [20](https://arxiv.org/html/2403.06294v3#bib.bib20)]、多代理交互[[22](https://arxiv.org/html/2403.06294v3#bib.bib22)]以及新颖的算法框架[[23](https://arxiv.org/html/2403.06294v3#bib.bib23)]优化LLM的推理和规划能力。通过这一设计，LLM代理能够与环境交互，并通过中间推理步骤生成行动计划，这些步骤可以按顺序执行以获得有效的解决方案。

受到这些概念的启发，我们提出了ArgMed-Agents，一个旨在可解释临床决策推理的多代理框架。我们通过临床讨论论证方案（ASCD）将临床讨论的认知过程形式化，作为LLM代理进行互动推理的提示策略。ArgMed-Agents中有三种类型的代理：生成器、验证器和推理者。生成器基于论证方案生成支持临床决策的论点；验证器基于关键问题检查论点的合法性，如果不合法，则要求生成器生成攻击性论点；推理者是一个具有符号求解器的LLM代理，它在结果导向的论证图中识别出合理且不矛盾的论点，作为决策支持。

在我们的方法中，我们并不期望每个提出的论点或生成器或验证器的检测都正确，而是将其生成视为假设。通过提示策略，LLM代理被引导以自我辩论的方式递归迭代，而新提出的假设总是与旧的假设相矛盾，最终推理者会消除不合理的假设并识别出一致的论点，从而得出一致的推理结果。ArgMed-Agents使得LLM能够通过将其自身生成作为问题递归的提示来解释其输出，从而进行自我认知分析。

我们的实验分为两个部分，分别评估ArgMed-Agents临床推理的准确性和可解释性。我们在两个数据集上进行了实验，包括MedQA [[24](https://arxiv.org/html/2403.06294v3#bib.bib24)] 和PubMedQA [[25](https://arxiv.org/html/2403.06294v3#bib.bib25)]。为了更好地与现实世界应用场景对接，我们的研究聚焦于零-shot设置。结果表明，ArgMed-Agents在准确性和可解释性方面都优于直接生成和思维链（CoT）。

总结来说，我们做出了以下贡献：

+   •

    我们提出了一种新的多LLM代理框架，称为ArgMed-Agents，用于复杂的临床推理任务。

+   •

    我们的方法有效地结合了论证框架和认知临床医学。通过LLM模拟临床讨论来呈现论证，通过正式的计算模型解决推理结果，这避免了LLM逻辑推理中的累积错误，并提高了临床推理的安全性。

+   •

    我们现在提出关于ArgMed-Agents理论保证的猜想，即当ArgMed-Agents认为所有决策都不可接受时，识别LLM中的推理错误。这个猜想作为我们识别大型语言模型（例如，幻觉现象、知识冲突）能力边界的一种手段，这对临床决策支持至关重要。如果LLM犯了严重的错误，而人类没有及时发现，可能会对患者造成无法挽回的损害。

+   •

    我们对ArgMed-Agents进行了广泛的评估，结果表明我们的方法在临床决策支持中具有准确性、可解释性和安全性优势。

## II 前言

### II-A 临床讨论的论证方案

我们的方法利用了计算论证的概念来支持推理。抽象论证（AA）[[26](https://arxiv.org/html/2403.06294v3#bib.bib26)] 是由两个部分组成的对 $\langle\mathcal{A},\mathcal{R}\rangle$：一个抽象论证集合 $\mathcal{A}$ 和一个二元攻击关系 $\mathcal{R}$。给定一个AA框架 $\langle\mathcal{A},\mathcal{R}\rangle$，$a,b\in\mathcal{A}$ 且 $(a,b)\in\mathcal{R}$ 表示 $a$ 攻击 $b$。在这个框架中，$a\in\mathcal{A}$ 是可接受的，当且仅当：

+   •

    不存在 $b\in\mathcal{A}$ 使得 $(b,a)\in\mathcal{R}$；

+   •

    存在 $b,c\in\mathcal{A}$ 使得 $(b,a)\in\mathcal{R}$ 且 $(c,b)\in\mathcal{R}$；

在此基础上，许多研究[[27](https://arxiv.org/html/2403.06294v3#bib.bib27), [12](https://arxiv.org/html/2403.06294v3#bib.bib12), [9](https://arxiv.org/html/2403.06294v3#bib.bib9)]已经探索了论证在临床领域中的应用。在本节中，我们总结了这些研究成果，重点讨论了临床讨论的论证方案（ASCD）。论证方案（AS）的概念起源于非正式逻辑领域，源自[[28](https://arxiv.org/html/2403.06294v3#bib.bib28), [29](https://arxiv.org/html/2403.06294v3#bib.bib29)]的开创性研究。论证方案作为一个半正式化框架，用于捕捉和分析人类推理模式。它被正式定义为$AS=\langle P,c,V\rangle$，其中包含一组前提（$P$）、由这些前提支持的结论（$c$）以及前提（$P$）或结论（$c$）中固有的变量（$V$）。论证方案的一个关键方面是明确与AS相关的关键问题（Critical Questions, CQs）。未能解决这些问题将对论证方案中的前提和结论提出挑战。因此，关键问题的作用是激发论证的生成；当AS受到质疑时，它将引发针对初始AS的反驳论证的形成。这个迭代过程最终会构建一个攻击论证图，从而促进对论证动态的深刻理解。

ASCD encapsulates a variety of argumentation schemes that analyse the clinical discussion and reasoning process, such as Argumentation Scheme for Decision Making (ASDM), Argumentation Scheme for Side Effects (ASSE) and Argumentation Scheme for Danger Apeal (ASDA). Figure [1](https://arxiv.org/html/2403.06294v3#S2.F1 "Figure 1 ‣ II-A Argumentation Scheme for Clinical Discussion ‣ II Preliminaries ‣ ArgMed-Agents: Explainable Clinical Decision Reasoning with LLM Disscusion via Argumentation Schemes") illustrates an example of ASDM.

![请参见说明文字](img/c89fb29203ecf73949b07fed9ac1ff7b.png)

图1：决策制定的论证方案示例

## III 方法

在本节中，我们提出了一种名为 ArgMed-Agents 的多智能体框架，该框架支持将基于临床讨论的论证方案设计的提示策略无缝集成到智能体交互中。我们的方法增强了大语言模型（LLMs）的能力，使其能够执行可解释的临床决策推理，而无需专家参与知识编码。

### III-A ArgMed-Agents：一个多大语言模型智能体框架

![请参见说明文字](img/8bd0b6b60b2b95a4f4236f15c49927ee.png)

图2：来自MedQA USMLE数据集的一个示例，展示了ArgMed-Agents推理临床问题的完整过程。值得注意的是，论证框架中的字母对应右侧四个生成器的序号，表示由该生成器生成的前提和结论。在论证框架中，红色节点（$A$和$C$）表示支持决策的论据，黄色节点（$B$和$D$）表示支持信念的论据。

ArgMed-Agents框架包括三种不同类型的LLM代理：

+   •

    生成器：根据当前情况生成论据。

+   •

    验证器：每当生成一个新的论据时，验证器通过以自问自答的方式询问和回答关键问题（CQ）来检查论据的准确性。当CQ验证被拒绝时，拒绝CQ的原因会返回给生成器，生成器会根据该CQ提出一个新的论据。这个过程会反复进行，直到不再生成新的论据。

+   •

    推理器：它是一个配备了符号求解器的LLM代理，用于计算性论证。推理器记录迭代的完整过程，并识别论据之间的语义关系（攻击或支持）。在迭代结束时，所有论据形成一个完整的抽象论证框架。推理器通过符号求解器搜索一组连贯且可信的论据，作为决策支持。

在ArgMed-Agents中，并不期望单次生成或单次验证是正确的。ArgMed-Agents将生成作为假设，递归性地通过提出关键问题来提示模型，从而在迭代过程中识别冲突和错误的论据。最终，这些相互攻击的论据将转化为一个正式框架，通过推理器求解出一组合理连贯的论据。图[2](https://arxiv.org/html/2403.06294v3#S3.F2 "图2 ‣ III-A ArgMed-Agents: 一个多LLM代理框架 ‣ III方法 ‣ ArgMed-Agents：通过论证方案进行可解释的临床决策推理")展示了多代理如何相互作用以模拟临床讨论。在图[2](https://arxiv.org/html/2403.06294v3#S3.F2 "图2 ‣ III-A ArgMed-Agents: 一个多LLM代理框架 ‣ III方法 ‣ ArgMed-Agents：通过论证方案进行可解释的临床决策推理")的示例中，生成器首先提出普罗昔替作为治疗药物。然而，在经过几轮讨论后，ArgMed-Agents提出了普罗昔替的副作用问题，最终确定了更有效的治疗药物——曲唑酮。

我们方法的理论动机源自非单调逻辑、逻辑直觉和认知临床。研究表明，LLM在处理简单推理和单步推理问题时表现良好，然而随着推理步骤的增加，正确推理的比例会以灾难性的方式下降 [[30](https://arxiv.org/html/2403.06294v3#bib.bib30)]。因此，我们认为LLM具备逻辑直觉，但缺乏真正的逻辑推理能力。

鉴于此，ArgMed-Agents框架引导LLM以递归形式生成一系列因果推理步骤作为暂定结论，任何进一步的证据都会撤回其结论。该过程最终会生成一个有向图，表示争议关系。此外，我们的方法在符号求解器的帮助下执行图推理，从而避免了LLM推理的累积误差。LLM可以被视为一个包含大量冲突知识的大型知识库。ArgMed-Agents提供了一种正式的方法来发现冲突并解决一致的推理结果，这有助于LLM逐渐提出新的知识来修正其初步结论。

在ArgMed-Agents框架中，具备不同LLM角色的智能体通过形式化的过程进行协作互动，模拟临床讨论。这种设计不仅促进LLM通过批判性思维方法参与临床推理，将其认知过程与临床医生对齐，从而增强决策的可解释性，而且还能够有效地提取和突出LLM内部隐性知识，这些知识通过传统提示方法难以直接获取。

### III-B 正式计算模型与定理

本节我们描述了ArgMed-Agents如何执行正式推理并解释决策。首先，我们定义ArgMed-Agents的互动模型。

###### 定义 1  （ArgMed-Agents 互动模型）。

在ArgMed-Agents中，互动是一系列对话 $\mathcal{D}=\{s_{1},...,s_{n}|n>1\}$，涉及生成器智能体 $g$ 和验证者智能体 $v$，且满足以下条件：

+   •

    $s_{i}=\langle g,a\rangle$ 是由生成器 $g$ 提出的论点。

+   •

    $s_{k}=\langle v,cq\rangle$ 是由验证者 $v$ 提出的关键问题 $cq\in CQs$。

+   •

    如果存在来自生成器的 $s_{i}$ 和 $s_{i+2}$，则来自验证者的 $s_{i+1}$ 满足 $s_{i+2}$ 攻击 $s_{i}$。

在 ArgMed-Agents 中，我们定义论证由生成器呈现，验证者在论证之间构建攻击关系。具体而言，当验证者拒绝一个论证时，会产生一个新的论证，这个新论证攻击原论证；当验证者接受一个论证时，一轮对话结束。通过这种方式，推理者可以连接所有论证的语义关系，并构建一个论证框架。接下来，我们定义 ArgMed-Agents 的论证框架：

###### 定义 2 （ArgMed-Agents 中的论证）。

ArgMed-Agents 的论证框架 AF 是一个对 $\langle\mathcal{A},\mathcal{R}\rangle$，其中：

+   •

    $\mathcal{A}$ 是由生成器从 ${s_{i}}\subseteq\mathcal{D}$ 构造的论证集，$\mathcal{R}$ 是一组二元攻击关系。

+   •

    $\mathcal{A}=Args_{d}(\mathcal{A})\cup Args_{b}(\mathcal{A})$，使得论证集 $\mathcal{A}$ 中的论证分为两种类型：支持决策的论证 $Args_{d}(\mathcal{A})$ 和支持信念的论证 $Args_{b}(\mathcal{A})$。

我们将 ArgMed-Agents 的论证分为两种类型：支持决策的论证 $Args_{d}(\mathcal{A})$ 和支持信念的论证 $Args_{b}(\mathcal{A})$。这两种类型的论证在角色上有所不同，$Args_{d}(\mathcal{A})$ 基于信念和目标，尝试为选择提供理由，而 $Args_{b}(\mathcal{A})$ 总是试图削弱决策论证。此外，我们为这两种论证做出了以下设置：首先，支持不同决策的论证是相互冲突的；其次，支持决策的论证不能攻击支持信念的论证。形式化表示为：

###### 定义 3。

给定一个 ArgMed-Agents 的论证框架（AF）$\langle\mathcal{A},\mathcal{R}\rangle$，其中 $\mathcal{R}\subseteq Args_{d}(\mathcal{A})\times Args_{b}(\mathcal{A})$，满足以下条件：

+   •

    对于所有 $a_{1},a_{2}$  $\in Args_{d}(\mathcal{A})$ 且 $a_{1}\neq a_{2}$，有 $(a_{1},a_{2}),(a_{2},a_{1})\in\mathcal{R}$。

+   •

    不存在 $(arg_{1},arg_{2})$  $\in\mathcal{R}$，使得 $arg_{1}\in Args_{d}(\mathcal{A})$ 且 $arg_{2}\in Args_{b}(\mathcal{A})$。

定义 [4](https://arxiv.org/html/2403.06294v3#Thmdefinition4 "Definition 4\. ‣ III-B Formal Computational Models & Theorems ‣ III Method ‣ ArgMed-Agents: Explainable Clinical Decision Reasoning with LLM Disscusion via Argumentation Schemes") 描述了 ArgMed-Agents 如何识别可选决策及其解释集的概念。

###### 定义 4。

给定一个 ArgMed-Agents 的论证框架 $\langle\mathcal{A},\mathcal{R}\rangle$，当且仅当一个决策 $d\in Args_{d}(\mathcal{A})$ 可接受时，该决策为可选决策。决策支持解释集 $E$ 包括：

+   •

    存在一个决策 $d=Args_{d}(E)$ 为可选决策。

+   •

    所有论证 $a\in E$ 是可接受的且无冲突的。

+   •

    在满足上述两个条件的集合中，$E$是包含最大元素的集合。

接下来，我们使用MedQA中的一个例子来说明如何在ArgMed-Agents中应用定义[4](https://arxiv.org/html/2403.06294v3#Thmdefinition4 "Definition 4\. ‣ III-B Formal Computational Models & Theorems ‣ III Method ‣ ArgMed-Agents: Explainable Clinical Decision Reasoning with LLM Disscusion via Argumentation Schemes")。

###### 示例 1  （来自MedQA）。

一名18岁的女性反复出现头痛。疼痛通常是单侧的，呈跳动性，受光线和噪音的刺激加剧，通常持续几小时到一天。头痛有时是由食用巧克力触发的。这些头痛干扰了她的日常活动。体格检查结果在正常范围内。她还有本态性震颤。对于她的头痛预防，哪种药物适合她？

经过几轮讨论后，ArgMed-Agents提出了五个论点，包括三个关于治疗的决策和两个关于副作用的证据，并形成了以下的论证框架：

![[无标题图片]](img/ca1b18c6ff3b35871e6d4615550803b2.png)

基于这些定义，我们可以得到以下推理结果：

+   •

    可选决策：$B$（普萘洛尔）或$C$（维拉帕米）；

+   •

    解释集 $E=\{\{B,D,E\},\{C,D,E\}\}$；

值得注意的是，尽管有两个决策是可选的，我们定义了这些决策的排他性，以确保它们不会同时出现在决策支持中（这避免了重复用药）。

有了这个形式化模型，我们提出了关于ArgMed-Agents理论保证的猜想：

> 给定一个ArgMed-Agents的AF $\langle\mathcal{A},\mathcal{R}\rangle$，$E$是其决策支持集。当且仅当$Args_{d}(E)=\varnothing$时，ArgMed-Agents的临床推理中存在错误。

临床推理错误在[[22](https://arxiv.org/html/2403.06294v3#bib.bib22)]中有所讨论。他们认为，LLM中的大多数临床推理错误是由于对领域知识的混淆。另一方面，[[3](https://arxiv.org/html/2403.06294v3#bib.bib3)]的研究指出，LLM可能会产生关于医疗治疗的令人信服的错误信息，因此识别这种错觉至关重要，而这对于人类来说是很难检测的。为此，我们推测，识别ArgMed-Agents临床推理错误的相关机制如下：当ArgMed-Agents的$AF$中的任何决策未被接受时，我们认为LLM的推理存在错误，且LLM的知识储备不足以解决这些问题。该机制帮助ArgMed-Agents识别LLM的能力边界，从而避免采用错误决策带来的风险，确保更加稳健和安全的临床推理。我们基于以下推测：ArgMed-Agents通过迭代LLM来启动其内部隐性知识，这些知识可能会发生冲突（可能是由于数据脏乱或时序冲突）。当冲突的知识量较小时，ArgMed-Agents可以通过非单调推理得出正确的决策；当冲突的知识量较大时，论证框架将会非常庞大（包含由幻觉现象产生的错误论点），其中的逻辑错误可能导致ArgMed-Agents无法得出任何正确的决策。因此，我们将这种情况视为LLM能力的边界。

## IV 实验

在本节中，我们通过评估准确性和可解释性，展示了ArgMed-Agents在临床决策推理中的潜力。

### IV-A 设置

我们使用OpenAI提供的API GPT-3.5-turbo和GPT-4在ArgMed-Agents中实现了不同类型的代理[[1](https://arxiv.org/html/2403.06294v3#bib.bib1)]，根据我们的设置，LLM代理使用相同的LLM和不同的少量示例提示进行实现。每个代理都配置了特定的参数：温度设置为0.0，以及对话限制为8，允许ArgMed-Agents生成的最大决策数为4（因为MedQA数据集是多选题，只有四个选项），这一设置是为了防止代理之间进入循环。另一方面，我们使用python3实现了我们形式模型的符号求解器。

### IV-B 数据集

以下两个数据集用于评估ArgMed-Agents临床推理的准确性和可解释性：

+   •

    MedQA [[24](https://arxiv.org/html/2403.06294v3#bib.bib24)]: 来自美国医学执照考试（USMLE）衍生的多项选择题。该数据集来源于官方医学考试，涵盖了英语、简体中文和繁体中文的问题。每种语言的问题总数分别为12,723、34,251和14,123个。

+   •

    PubMedQA [[25](https://arxiv.org/html/2403.06294v3#bib.bib25)]: 一个从PubMed摘要中收集的生物医学问答（QA）数据集。PubMedQA的任务是利用相应的摘要回答是/否/也许类型的研究问题（例如，术前他汀类药物是否减少冠状动脉旁路移植手术后的心房颤动）。

### IV-C 准确性

在准确性评估中，我们通过从每个数据集中随机选择300个示例来严格筛选数据集，确保了平衡的代表性以进行稳健分析。为了建立比较基准，我们采用了GPT直接生成和思维链（CoT），如[[31](https://arxiv.org/html/2403.06294v3#bib.bib31)]中所述。值得注意的是，我们的主要关注点集中在评估ArgMed-Agents在临床决策推理领域的效果。需要特别指出的是，虽然像MedQA和PubMedQA这样的数据集包含了生物医学常识类问答题，但我们在示例选择过程中故意排除了这一子集。这个有意排除的做法旨在保持评估的完整性，特别是在临床决策制定领域，从而确保对ArgMed-Agents性能的有针对性评估。

{tblr}

hline1,8 = -0.08em, hline2,5 = -, 模型与方法 MedQA PubMedQA

Direct 52.7 68.4

GPT-3.5-turbo CoT 48.0 71.5

ArgMed-Agents 62.1 78.3

Direct 67.8 72.9

GPT-4 CoT 71.4 77.2

ArgMed-Agents 83.3 81.6

表I：各种临床推理方法在MedQA和PubMedQA数据集上的准确性结果。

表[I](https://arxiv.org/html/2403.06294v3#S4.T1 "TABLE I ‣ IV-C Accuracy ‣ IV Experiments ‣ ArgMed-Agents: Explainable Clinical Decision Reasoning with LLM Disscusion via Argumentation Schemes")展示了MedQA和PubMedQA的准确性结果，并将ArgMed-Agents与不同的基准模型在直接生成和思维链（CoT）配置下进行了对比。值得注意的是，使用GPT-3.5-turbo和GPT-4模型的评估结果表明，我们提出的ArgMed-Agents在临床决策推理任务中的准确性显著高于CoT和其他基准模型。这一发现突显了ArgMed-Agents在增强语言模型（LLM）系统的临床推理能力方面的有效性。有趣的是，通过多次实验重复，我们观察到在某些情况下，整合CoT反而导致性能的意外下降。这一现象可能源于LLM存在幻觉行为的倾向，而CoT有可能无限制地延续这些幻觉。相比之下，ArgMed-Agents通过迭代辩论纠正初步的错误结论，有效地缓解了这一问题。

### IV-D 可解释性

可解释人工智能（XAI）的主要关注点通常是AI做出决策或预测的推理过程，使其变得更加易于理解和透明。在XAI的背景下，可解释性被定义为[[32](https://arxiv.org/html/2403.06294v3#bib.bib32)]：

+   …了解AI系统的程度

    给出一个结果的过程。

基于这一标准，我们定义了LLM可解释性的衡量标准：

+   •

    LLM给出的解释在多大程度上帮助用户预测LLM的决策？

我们的团队在早期阶段开发了一个功能齐全的基于知识的临床决策支持系统（CDSS）[[33](https://arxiv.org/html/2403.06294v3#bib.bib33)]，可用于帮助治疗癌症、神经系统疾病和传染病等复杂疾病的决策。该CDSS由计算机可解释的指南组成，采用资源描述框架（RDF）[[34](https://arxiv.org/html/2403.06294v3#bib.bib34)]形式，并配有相应的推理引擎。基于知识的CDSS可以视为可解释的系统。

基于此，我们的实验设置如下：我们将直接生成解释和COT作为基线，并将基于知识的CDSS作为我们期望ArgMed-Agents可解释性接近的基准。我们记录了100个MedQA示例中的输入（即问题）和推理过程（例如COT的推理链、基于知识的CDSS中的RDF推理节点以及ArgMed-Agents与相应论证框架之间的完整对话）。在评估中，我们对一个LLM进行了微调，使其作为评估者，基于推理记录预测相应的决策。我们将评估者预测的准确性视为可解释性的指标（即推理过程在多大程度上帮助用户理解决策）。

{tblr}

hline1-2,5,8-9 = -, 模型与方法预设

直接 0.53

GPT-3.5-turbo COT 0.59

ArgMed-Agents 0.87

直接 0.68

GPT-4 COT 0.73

ArgMed-Agents 0.91

基于知识的CDSS 0.95

表II：不同模型和方法在100个MedQA示例中通过推理记录预测准确度（Pre.）。

基于知识的CDSS实现了最高的预测准确度（0.95），作为可解释性的基准。在测试的方法中，ArgMed-Agents在GPT-3.5-turbo和GPT-4模型中均表现出显著高于直接生成和COT方法的预测准确度。使用GPT-4的ArgMed-Agents达到了0.91的预测准确度，接近基于知识的CDSS的可解释性水平。

该研究表明，ArgMed-Agents显著提升了LLM的可解释性，使用户能够更好地理解和预测模型的决策。这表明，采用先进的论证框架可以弥合黑箱AI模型与像CDSS这样完全透明、基于知识的系统之间的差距。

### IV-E 讨论

假设1认为，当AF无法得出任何可接受的决策时，ArgMed-Agents会出现临床推理错误。为了提供初步证据，我们分析了ArgMed-Agents未能生成可接受决策的实例，并将这些实例与观察到的错误进行了关联。我们的数据表明，在63%的情况下，ArgMed-Agents未能生成可接受的决策集时，存在可识别的临床推理错误。这些错误主要是由于AF中存在大量冲突知识，导致系统无法得出一致的决策。具体来说，76%的错误与冲突知识有关，而24%的错误则是由于领域知识不足。

我们的分析表明，ArgMed-Agents在医学知识中的导航能力和解决冲突的能力是其有效性的关键决定因素。这与以下假设一致：对于给定的输入（例如临床推理问题），当LLM中只有少量冲突知识时，ArgMed-Agents可以通过进一步的论证排除错误的论点，从而推理出正确的治疗方案。当LLM中存在大量不一致的知识时，这些冲突会导致一个复杂的系统，阻止LLM在生成中保持逻辑一致性（即幻觉现象），从而妨碍有效的决策制定。

ArgMed-Agents相对于类似现有技术的优势在于，LLM推理失败的条件（即当面临大量冲突知识或缺乏足够的领域知识时）可以通过正式的论证被识别出来，从而让我们更好地理解和预测LLM在临床应用中的局限性。这一理解对于减轻医疗环境中由于决策不当带来的风险至关重要。

此外，我们还分析了ArgMed-Agents推理的可追溯性。我们发现，CoT方法得出答案的推理过程通常是逻辑上不完整的，而ArgMed-Agents的推理过程则要精细得多。有趣的是，我们发现，知识基础的CDSS在许多例子中的推理可以视为ArgMed-Agents推理子图的一部分，可能是因为ArgMed-Agents以随机生成的方式遍历所有推理路径，因此推理图不仅包含正确决策的推理路径，还包括错误决策的解释。这一发现为ArgMed-Agents的可解释性提供了进一步的证据。在许多临床决策案例中，患者可能不仅希望了解为何做出某个决策，还希望了解为何无法采纳另一种决策。

## V 相关工作

### V-A 基于LLM的临床决策支持系统

大量的研究强调了大语言模型（LLMs）在医学应用中的潜在价值[[35](https://arxiv.org/html/2403.06294v3#bib.bib35), [36](https://arxiv.org/html/2403.06294v3#bib.bib36), [37](https://arxiv.org/html/2403.06294v3#bib.bib37)]。然而，这些模型在面对需要高级医学专业知识和强大推理技能的复杂临床情境时，仍然面临着做出可靠决策的挑战[[3](https://arxiv.org/html/2403.06294v3#bib.bib3)]。因此，已经开展了大量工作，旨在增强LLMs的临床推理能力。[[22](https://arxiv.org/html/2403.06294v3#bib.bib22)]提出了一种多学科协作（MC）框架，采用基于LLM的代理在模拟角色扮演情境中进行协作性、迭代性讨论，以推动共识的建立。尽管这一方法取得了有前景的成果，但在有效地形式化迭代结果以通过专用推理工具提高LLMs的推理表现方面仍存在困难。

另一种方法，由[[38](https://arxiv.org/html/2403.06294v3#bib.bib38)]提出，利用诊断推理提示来增强临床推理和大语言模型（LLMs）的可解释性。然而，这种方法通过为特定疾病设计的模板来提高可解释性，而ArgMed-Agents则更加自动化。除此之外，ArgMed-Agents还提供了关于为何不会选择某个决策的解释。

此外，像[[39](https://arxiv.org/html/2403.06294v3#bib.bib39), [40](https://arxiv.org/html/2403.06294v3#bib.bib40)]等研究工作通过使用来自医学和生物医学文献的大规模数据集对LLMs进行微调。与这些方法不同，我们的方法侧重于利用LLMs中固有的潜在医学知识，以在无需训练的环境中增强其推理能力。

### V-B 与LLMs的逻辑推理

大量的研究致力于利用符号系统增强推理能力，涵盖了多种方法，包括代码环境、知识图谱和形式化定理证明器[[21](https://arxiv.org/html/2403.06294v3#bib.bib21), [41](https://arxiv.org/html/2403.06294v3#bib.bib41), [42](https://arxiv.org/html/2403.06294v3#bib.bib42)]。在Jung等人[[43](https://arxiv.org/html/2403.06294v3#bib.bib43)]的研究中，推理被框定为其逻辑关系的可满足性问题，通过应用逆因果推理来实现。该方法利用SAT求解器来增强一致性推理，从而提高大语言模型（LLMs）的推理能力。

张等人的研究[[18](https://arxiv.org/html/2403.06294v3#bib.bib18)]采用了不同的方法，通过使用累积推理将任务分解为更小、更易管理的组成部分，从而简化了解决问题的过程并提高了整体效率。在另一项研究中，Xiu等人[[16](https://arxiv.org/html/2403.06294v3#bib.bib16)]探索了大规模语言模型（LLMs）的非单调推理能力。然而，尽管LLMs展示了有前景的初步结果，但它们在泛化和基于证明的可追溯性方面表现显著不足，随着推理深度的增加，其效果显著下降。

与我们的研究结果一致，最近的研究已经开始探讨增强LLMs论证推理能力的潜力[[14](https://arxiv.org/html/2403.06294v3#bib.bib14)]。这些努力旨在增强LLMs的论证推理能力，正如Dewynter等人[[44](https://arxiv.org/html/2403.06294v3#bib.bib44)]和Castagna等人[[15](https://arxiv.org/html/2403.06294v3#bib.bib15)]的研究所表明的那样，他们致力于进一步发展LLM性能的这一方面。

## VI 结论

在本研究中，我们提出了一种新型的医学多智能体交互框架，名为ArgMed-Agents，旨在激发不同角色的智能体模拟临床讨论过程，反复进行论证和关键提问。最后，推理智能体在该框架中识别出一组合理且连贯的论点，作为决策支持和解释。实验结果表明，与不同的基准相比，使用ArgMed-Agents进行临床决策推理能够达到更高的准确性，并为其推理结果提供固有的解释。此外，我们在讨论中的分析显示，ArgMed-Agents能够在很大程度上识别自己的推理错误和能力边界。这意味着，相较于其他最先进的方法，ArgMed-Agents提供了更安全的决策。我们在本研究中的目标是为医疗专业人员提供强大的工具，以增强他们的决策过程，并最终改善患者的治疗效果。

尽管ArgMed-Agents取得了成功，但仍然存在一些局限性。我们对这些局限性提出了以下未来方向建议：（1）虽然我们使用抽象论证形式化了ArgMed-Agents临床讨论的结果，但扩展到更具表现力的逻辑（例如，一阶逻辑、描述逻辑、模态逻辑或概率逻辑）将能够实现更复杂的推理和论证生成。（2）量身定制的适应性方法可以根据个体用户的需求和偏好进一步提高临床决策的有效性和可解释性。未来的工作可以探讨基于患者/医生反馈的互动式临床决策支持系统。

## 参考文献

+   [1] OpenAI，“Gpt-4技术报告”，2023年。

+   [2] A. Nayak, M. S. Alkaitis, K. Nayak, M. Nikolov, K. P. Weinfurt, 和 K. Schulman, “比较聊天机器人和资深内科住院医生生成的现病史总结”，*JAMA Intern Med*，第183卷，第9期，页码1026–1027，2023年9月。

+   [3] K. Singhal, S. Azizi, T. Tu, S. S. Mahdavi, J. Wei, H. W. Chung, N. Scales, A. Tanwani, H. Cole-Lewis, S. Pfohl, P. Payne, M. Seneviratne, P. Gamble, C. Kelly, N. Scharli, A. Chowdhery, P. Mansfield, B. A. y Arcas, D. Webster, G. S. Corrado, Y. Matias, K. Chou, J. Gottweis, N. Tomasev, Y. Liu, A. Rajkomar, J. Barral, C. Semturs, A. Karthikesalingam, 和 V. Natarajan, “大语言模型编码临床知识”，2022年。

+   [4] J. W. Ayers, A. Poliak, M. Dredze, E. C. Leas, Z. Zhu, J. B. Kelley, D. J. Faix, A. M. Goodman, C. A. Longhurst, M. Hogarth, 和 D. M. Smith, “比较医生与人工智能聊天机器人对患者问题的回应：发布到公共社交媒体论坛的研究”，*JAMA Internal Medicine*，第183卷，第6期，页码589–596，2023年6月。[在线]。可访问：https://doi.org/10.1001/jamainternmed.2023.1838

+   [5] A. Pal, L. K. Umapathi, 和 M. Sankarasubbu, “Med-halt：大语言模型的医学领域幻觉测试”，2023年。

+   [6] E. Eigner 和 T. Händler, “大语言模型辅助决策的决定因素”，2024年。

+   [7] E. Dietz, A. Kakas, 和 L. Michael, “论证：以人为本的人工智能演算”，*Frontiers in Artificial Intelligence*，第5卷，2022年。[在线]。可访问：https://www.frontiersin.org/articles/10.3389/frai.2022.955579

+   [8] ——, “计算论证与认知”，2021年。

+   [9] M. A. Qassas, D. Fogli, M. Giacomin, 和 G. Guida, “基于论证模式的临床讨论分析”，*Procedia Computer Science*，第64卷，页码282–289，2015年，ENTERprise信息系统国际会议/项目管理国际会议/健康和社会护理信息系统与技术会议，CENTERIS/ProjMAN / HCist 2015年10月7-9日。[在线]。可访问：https://www.sciencedirect.com/science/article/pii/S1877050915026265

+   [10] S. Hong, L. Xiao, 和 J. Chen, “在共享临床决策中合并多代理论证的交互模型”，在*2023 IEEE国际生物信息学与生物医学会议（BIBM）*，2023年，页码4304–4311。

+   [11] Z. Zeng, Z. Shen, B. T. H. Tan, J. J. Chin, C. Leung, Y. Wang, Y. Chi, 和 C. Miao, “基于可解释性和论证的决策制定，应用于阿尔茨海默病的诊断与预后”，在*第17届国际知识表示与推理原理会议论文集*，2020年9月，页码816–826。[在线]。可访问：https://doi.org/10.24963/kr.2020/84

+   [12] I. Sassoon, N. Kökciyan, S. Modgil, 和 S. Parsons, “临床决策支持的论证模式”，*Argument and Computation*，第12卷，第3期，页码329–355，2021年11月。

+   [13] K. Čyras, B. Delaney, D. Prociuk, F. Toni, M. Chapman, J. Domínguez, 和 V. Curcin, “具有冲突医学建议的可解释推理的论证,” *CEUR工作坊会议录*, 第2237卷, 2018年1月, 2018年联合推理与模糊和冲突证据及医学建议的国际研讨会及第3届本体模块化、情境性与演化研讨会, MedRACER + WOMoCoE 2018；会议日期：2018年10月29日。

+   [14] G. Chen, L. Cheng, L. A. Tuan, 和 L. Bing, “探索大型语言模型在计算论证中的潜力,” 2023年。

+   [15] F. Castagna, N. Kokciyan, I. Sassoon, S. Parsons, 和 E. Sklar, “基于计算论证的聊天机器人：一项调查,” 2024年。

+   [16] Y. Xiu, Z. Xiao, 和 Y. Liu, “LogicNMR: 探索预训练语言模型的非单调推理能力,” 在 *计算语言学协会的发现：EMNLP 2022* 中, Y. Goldberg, Z. Kozareva, 和 Y. Zhang 编辑。阿布扎比，阿联酋：计算语言学协会，2022年12月，第3616-3626页。[在线]. 可用: https://aclanthology.org/2022.findings-emnlp.265

+   [17] J. Xie, K. Zhang, J. Chen, T. Zhu, R. Lou, Y. Tian, Y. Xiao, 和 Y. Su, “Travelplanner: 一项针对语言代理的现实世界规划基准测试,” 2024年。

+   [18] Y. Zhang, J. Yang, Y. Yuan, 和 A. C.-C. Yao, “大型语言模型的累积推理,” 2023年。

+   [19] L. Wang, C. Ma, X. Feng, Z. Zhang, H. Yang, J. Zhang, Z. Chen, J. Tang, X. Chen, Y. Lin, W. X. Zhao, Z. Wei, 和 J.-R. Wen, “基于大型语言模型的自主代理调查,” 2023年。

+   [20] W. Shi, R. Xu, Y. Zhuang, Y. Yu, J. Zhang, H. Wu, Y. Zhu, J. Ho, C. Yang, 和 M. D. Wang, “Ehragent: 代码赋能大型语言模型在电子健康记录上进行少量样本复杂表格推理,” 2024年。

+   [21] L. Pan, A. Albalak, X. Wang, 和 W. Y. Wang, “Logic-lm: 通过符号求解器赋能大型语言模型进行忠实的逻辑推理,” 2023年。

+   [22] X. Tang, A. Zou, Z. Zhang, Z. Li, Y. Zhao, X. Zhang, A. Cohan, 和 M. Gerstein, “Medagents: 大型语言模型作为零样本医学推理的协作伙伴,” 2024年。

+   [23] K. Gandhi, D. Sadigh, 和 N. D. Goodman, “与语言模型的战略推理,” 2023年。

+   [24] D. Jin, E. Pan, N. Oufattole, W.-H. Weng, H. Fang, 和 P. Szolovits, “这个病人得了什么病？来自医学考试的大规模开放领域问答数据集,” 2020年。

+   [25] Q. Jin, B. Dhingra, Z. Liu, W. W. Cohen, 和 X. Lu, “Pubmedqa: 一个生物医学研究问答数据集,” 2019年。

+   [26] P. M. Dung, “论论证的可接受性及其在非单调推理、逻辑编程和多人博弈中的基本作用,” *人工智能*, 第77卷，第2期，第321-357页，1995年。[在线]. 可用: https://www.sciencedirect.com/science/article/pii/000437029400041X

+   [27] T. Oliveira, J. Dauphin, K. Satoh, S. Tsumoto, 和 P. Novais, “面向多重疾病的临床决策支持中的目标论证,” 见 *第17届国际自治代理与多智能体系统会议论文集*, 2018年, 第2031-2033页。

+   [28] D. Walton, C. Reed, 和 F. Macagno, *Argumentation Schemes*, C. Reed 和 F. Macagno 编. 纽约: 剑桥大学出版社, 2008。

+   [29] D. Walton, *Argumentation Schemes for Presumptive Reasoning*, 第1版. Routledge, 1996年. [在线]. 可用链接: https://doi.org/10.4324/9780203811160

+   [30] A. Creswell, M. Shanahan, 和 I. Higgins, “选择推理: 利用大型语言模型进行可解释的逻辑推理,” 2022年。

+   [31] J. Wei, X. Wang, D. Schuurmans, M. Bosma, B. Ichter, F. Xia, E. Chi, Q. Le, 和 D. Zhou, “链式推理提示引发大型语言模型中的推理过程,” 2023年。

+   [32] “ISO, IEC: AWI TS 29119-11: 软件与系统工程 - 软件测试 - 第11部分: AI系统的测试,” 技术报告, 2020年. [在线]. 可用链接: https://www.iso.org/standard/84127.html

+   [33] M. Liu, L. Xiao, Q. Wang, Z. Liu, R. Zhu, 和 M. He, “基于CDSS和ChatGPT-4的医学决策推荐生成与相似性融合研究,” 见 *2023 IEEE国际生物信息学与生物医学会议（BIBM）*, 2023年, 第873-878页。

+   [34] G. Klyne 和 J. J. Carroll. (2004) 资源描述框架 (rdf): 概念与抽象语法. W3C推荐. W3C. [在线]. 可用链接: http://www.w3.org/TR/2004/REC-rdf-concepts-20040210/

+   [35] Z. Bao, W. Chen, S. Xiao, K. Ren, J. Wu, C. Zhong, J. Peng, X. Huang, 和 Z. Wei, “Disc-medllm: 连接通用大型语言模型与现实医学咨询,” 2023年。

+   [36] H. Nori, Y. T. Lee, S. Zhang, D. Carignan, R. Edgar, N. Fusi, N. King, J. Larson, Y. Li, W. Liu, R. Luo, S. M. McKinney, R. O. Ness, H. Poon, T. Qin, N. Usuyama, C. White, 和 E. Horvitz, “通用基础模型能否超越专用调优？医学中的案例研究,” 2023年。

+   [37] Q. Jin, Y. Yang, Q. Chen, 和 Z. Lu, “Genegpt: 通过领域工具增强大型语言模型以改善生物医学信息的访问,” 2023。

+   [38] T. Savage, A. Nayak, R. Gallo, 等人, “诊断推理提示揭示了大型语言模型在医学中可解释性的潜力,” *npj Digital Medicine*, 第7卷, 第20页, 2024年. [在线]. 可用链接: https://doi.org/10.1038/s41746-024-01010-1

+   [39] K. Singhal, T. Tu, J. Gottweis, R. Sayres, E. Wulczyn, L. Hou, K. Clark, S. Pfohl, H. Cole-Lewis, D. Neal, M. Schaekermann, A. Wang, M. Amin, S. Lachgar, P. Mansfield, S. Prakash, B. Green, E. Dominowska, B. A. y Arcas, N. Tomasev, Y. Liu, R. Wong, C. Semturs, S. S. Mahdavi, J. Barral, D. Webster, G. S. Corrado, Y. Matias, S. Azizi, A. Karthikesalingam, 和 V. Natarajan, “通过大型语言模型实现专家级医学问答,” 2023年。

+   [40] C. Li, C. Wong, S. Zhang, N. Usuyama, H. Liu, J. Yang, T. Naumann, H. Poon, 和 J. Gao，“Llava-med：一天内训练一个面向生物医学的大型语言与视觉助手，” 2023年。

+   [41] S. Pan, L. Luo, Y. Wang, C. Chen, J. Wang, 和 X. Wu，“统一大规模语言模型和知识图谱：一条路线图，” *IEEE知识与数据工程学报*，第1–20页，2024年。[在线]。可用链接：[http://dx.doi.org/10.1109/TKDE.2024.3352100](http://dx.doi.org/10.1109/TKDE.2024.3352100)

+   [42] Q. Wu, G. Bansal, J. Zhang, Y. Wu, B. Li, E. Zhu, L. Jiang, X. Zhang, S. Zhang, J. Liu, A. H. Awadallah, R. W. White, D. Burger, 和 C. Wang，“Autogen：通过多代理对话推动下一代大型语言模型应用，” 2023年。

+   [43] J. Jung, L. Qin, S. Welleck, F. Brahman, C. Bhagavatula, R. L. Bras, 和 Y. Choi，“教育性提示：通过递归解释进行逻辑一致的推理，” 2022年。

+   [44] A. de Wynter 和 T. Yuan，“我想进行辩论：大规模语言模型中的论证推理，” 2023年。
