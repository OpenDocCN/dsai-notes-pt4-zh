<!--yml

分类：未分类

日期：2025-01-11 11:47:16

-->

# 大型语言模型通过智能体共进化模拟标准化病人

> 来源：[https://arxiv.org/html/2412.11716/](https://arxiv.org/html/2412.11716/)

Zhuoyun Du^(†♠♣♡)  Lujie Zheng^(†★♡)  Renjun Hu^♢  Yuyang Xu^(★♡)

Xiawei Li^♯  Ying Sun^♮  Wei Chen^(♠★)  Jian Wu^(♭♯♡)  Haolei Cai^♯^(🖂)  Haochao Ying^(♭♯)^(🖂)

^♠浙江大学CAD与CG国家重点实验室

^♣浙江工业大学，浙江大学应用技术学院

^★浙江大学计算机科学与技术学院

^♢阿里巴巴集团  ^♮中山大学肿瘤防治中心

^♭浙江大学公共卫生学院

^♯浙江大学医学院附属第二医院

^♡浙江省医学影像人工智能重点实验室

[duzy@zju.edu.cn](duzy@zju.edu.cn)  [haochaoying@zju.edu.cn](haochaoying@zju.edu.cn)

###### 摘要

使用标准化病人（SP）培训医务人员依然是一个复杂的挑战，既需要丰富的领域专业知识，也需要角色特定的实践。大多数关于基于大型语言模型（LLM）模拟病人的研究集中在提高数据检索准确性或通过人工反馈调整提示语。然而，这一研究方向忽视了病人智能体需要学习标准化表现模式的关键需求，这种模式能够通过无监督的模拟将数据转化为类似人类的病人回应。为了弥补这一空白，我们提出了EvoPatient，一个新型的模拟病人框架，在该框架中，病人智能体和医生智能体通过多轮对话模拟诊断过程，同时积累经验，以提高问题和回答的质量，最终实现对人类医生的培训。我们在多个案例上进行了广泛的实验，结果表明，通过仅提供整体的SP需求，我们的框架在需求对齐和人类偏好方面比现有的推理方法提高了超过10%，同时在经过200多个案例、10小时的演化后，能够实现资源消耗的最佳平衡，且具备优秀的泛化能力。代码将发布于[https://github.com/ZJUMAI/EvoPatient](https://github.com/ZJUMAI/EvoPatient)。

大型语言模型通过智能体共进化模拟标准化病人

Zhuoyun Du^(†♠♣♡)  Lujie Zheng^(†★♡)  Renjun Hu^♢  Yuyang Xu^(★♡) Xiawei Li^♯  Ying Sun^♮  Wei Chen^(♠★)  Jian Wu^(♭♯♡)  Haolei Cai^♯^(🖂)  Haochao Ying^(♭♯)^(🖂) ^♠浙江大学CAD与CG国家重点实验室 ^♣浙江工业大学，浙江大学应用技术学院 ^★浙江大学计算机科学与技术学院 ^♢阿里巴巴集团  ^♮中山大学肿瘤防治中心 ^♭浙江大学公共卫生学院 ^♯浙江大学医学院附属第二医院 ^♡浙江省医学影像人工智能重点实验室 [duzy@zju.edu.cn](duzy@zju.edu.cn)  [haochaoying@zju.edu.cn](haochaoying@zju.edu.cn)

^†^†footnotetext: ^†等贡献.^†^†footnotetext: ${}^{\text{\Letter}}$通讯作者：应浩潮 和 蔡浩雷。

## 1 引言

标准化病人（SPs）是经过专门训练的个体，他们模拟真实病人的症状、病史和情感状态 Barrows ([1993](https://arxiv.org/html/2412.11716v1#bib.bib2)); Ziv et al. ([2006](https://arxiv.org/html/2412.11716v1#bib.bib56)); McGaghie et al. ([2010](https://arxiv.org/html/2412.11716v1#bib.bib29))。在受控的学习环境中，他们对提高医疗人员的临床技能、沟通能力和诊断推理起着重要作用。然而，使用标准化病人需要投入大量的培训和运营成本，这需要广泛的医学知识和大量的角色特定练习 Levine et al. ([2013](https://arxiv.org/html/2412.11716v1#bib.bib18)); Wallace ([2007](https://arxiv.org/html/2412.11716v1#bib.bib47))。另一个常被忽视但至关重要的关注点是，标准化病人的沉浸式工作可能对其福祉产生不利影响。例如，真人标准化病人在整个模拟过程中必须应对与他们所扮演的病人角色相关的焦虑 Spencer and Dales ([2006](https://arxiv.org/html/2412.11716v1#bib.bib43)); Bokken et al. ([2006](https://arxiv.org/html/2412.11716v1#bib.bib3))。这些挑战突显了开发虚拟标准化病人的必要性，旨在减少在模拟训练过程中作为病人的人类参与。

研究已经探讨了使用基于规则的数字患者来替代人类标准化患者（SP）Othlinghaus-Wulhorst和Hoppe（[2020](https://arxiv.org/html/2412.11716v1#bib.bib31)）。然而，这些预定义的规则集和定制的对话框架通常难以捕捉现实世界患者病情和沟通的复杂性。大型语言模型（LLM）的出现，以其广泛的世界知识、角色扮演和泛化能力著称 Achiam et al.（[2023](https://arxiv.org/html/2412.11716v1#bib.bib1)）；Bubeck et al.（[2023](https://arxiv.org/html/2412.11716v1#bib.bib5)）；Li et al.（[2023a](https://arxiv.org/html/2412.11716v1#bib.bib20)）；Park et al.（[2023](https://arxiv.org/html/2412.11716v1#bib.bib33)），已经显示出在处理特定领域任务方面的强大潜力，包括在医学领域 Zhang et al.（[2023](https://arxiv.org/html/2412.11716v1#bib.bib53)）；Singhal et al.（[2023](https://arxiv.org/html/2412.11716v1#bib.bib42)）；Yu et al.（[2024](https://arxiv.org/html/2412.11716v1#bib.bib52)）；Moor et al.（[2023](https://arxiv.org/html/2412.11716v1#bib.bib30)）。然而，在虚拟SP的角色中，LLM面临着扮演双重角色的挑战。尽管拥有广泛的领域知识和对医学结果的理解，但它们必须能够可信地扮演未受教育的患者，故意缺乏医学见解并隐瞒关键信息。仅凭提示工程不足以确保LLM遵守这些原则，而微调则需要大量注释工作，且可能引发额外的隐私问题。

关于基于LLM的SP的研究相对较少。例如，Yu et al.（[2024](https://arxiv.org/html/2412.11716v1#bib.bib52)）通过从构建的知识图谱中检索相关信息，改善了响应质量。然而，这种方法不一定能将检索到的信息转化为SP所需的标准化表达。Louie et al.（[2024](https://arxiv.org/html/2412.11716v1#bib.bib26)）使LLM能够从人类专家反馈中引出原则，以遵循一种劳动密集且可能具有有限泛化能力的过程。为此，我们的研究提出了一个问题：*如何在最少的人工监督下有效地训练LLM模拟SP？* 我们建议，应该开发一个框架，使LLM患者代理能够通过模拟自主获得经验。这将使代理能够从高质量的对话中获取必要的知识，并发展出标准化的表达实践，逐渐将一个初学者患者代理转变为一个熟练的虚拟SP。

本文介绍了EvoPatient，这是一种创新的多智能体共进化框架，旨在帮助大型语言模型（LLMs）模拟**SPs**，无需人工监督或权重更新。我们将诊断过程建模为一系列阶段（即，症状生成、分诊、问诊、结论），这些阶段整合成一个模拟流程。我们的框架特点是模拟智能体对，其中医生智能体自主提问诊断问题，患者智能体作出回应。此设置使得诊断对话的自动收集成为可能，用于基于经验的训练。为了增强医生智能体提出问题的多样性，我们开发了一种跨学科会诊招募过程。此外，通过利用一组初始的文本SP需求，我们实施了一种无监督的共进化机制，该机制通过验证和存储示范性对话于动态库中，能够同时提高医生和患者智能体的性能。这些库帮助患者智能体提取少量示范并完善其回答各种诊断问题的文本需求。与此同时，医生智能体通过利用存储的对话快捷方式，学会提出越来越专业和高效的问题，从而进一步促进患者智能体的进化。实验结果表明，EvoPatient显著提高了患者智能体的需求对齐度，标准化了其回答并增强了鲁棒性，提升了记录的忠实度，并通过优化资源消耗增加了人类医生的偏好。此外，关于医生智能体进化和招募过程的实验展示了它们对患者智能体进化的积极贡献。

## 2 相关工作

##### 模拟合作伙伴

模拟伙伴是指在不同领域中使用的人员或软件生成的伴侣，用于为技能学习者提供课本知识无法提供的实践机会 Feltz 等人（[2020](https://arxiv.org/html/2412.11716v1#bib.bib10)，[2016](https://arxiv.org/html/2412.11716v1#bib.bib11)）；Péli 和 Nooteboom（[1997](https://arxiv.org/html/2412.11716v1#bib.bib35)）。以往的研究构建了各种软件生成的教育系统，但缺乏情境多样性 Graesser 等人（[2004](https://arxiv.org/html/2412.11716v1#bib.bib13)）；Ruan 等人（[2019](https://arxiv.org/html/2412.11716v1#bib.bib39)）；Othlinghaus-Wulhorst 和 Hoppe（[2020](https://arxiv.org/html/2412.11716v1#bib.bib31)）。大语言模型（LLMs）凭借其强大的泛化能力和模拟多种人物的能力，极大地克服了这一问题 Hua 等人（[2023](https://arxiv.org/html/2412.11716v1#bib.bib14)）；Li 等人（[2023b](https://arxiv.org/html/2412.11716v1#bib.bib23)）；Shanahan 等人（[2023](https://arxiv.org/html/2412.11716v1#bib.bib41)）；Park 等人（[2022](https://arxiv.org/html/2412.11716v1#bib.bib34)，[2023](https://arxiv.org/html/2412.11716v1#bib.bib33)）。因此，研究人员已探索了它们在各种领域的模拟训练中的应用，包括教师教育 Markel 等人（[2023](https://arxiv.org/html/2412.11716v1#bib.bib27)），冲突解决 Shaikh 等人（[2024](https://arxiv.org/html/2412.11716v1#bib.bib40)），外科手术培训 Varas 等人（[2023](https://arxiv.org/html/2412.11716v1#bib.bib46)）和咨询 Chen 等人（[2023a](https://arxiv.org/html/2412.11716v1#bib.bib6)）。在使用模拟伙伴进行医学教育时，以往的研究提出了通过提高数据提取能力或引入专家反馈来增强模拟的真实性的方法 Yu 等人（[2024](https://arxiv.org/html/2412.11716v1#bib.bib52)）；Louie 等人（[2024](https://arxiv.org/html/2412.11716v1#bib.bib26)）。与这些方法不同，我们的方法强调通过模拟来收集经验，而不涉及人为的参与。

![参见说明文字](img/92a188d193b8b7bddb073243774c2722.png)

图 1：患者代理人（![参见说明文字](img/6e1069abb1880aa0ad06b06bedee7e5f.png)）与医生代理人（![参见说明文字](img/7773650b5b5f50b13202d3f407f5ea12.png)）之间的典型多轮对话。代理人保持连续的记忆，医生代理人可以请求招募新的医生。此外，代理人会不断地从知识库（![参见说明文字](img/2d78a21a4714bfbf5ba6daf86fd46636.png)）中存储和检索知识，以促进持续进化。

##### 代理人的发展

最近，LLMs（大规模语言模型）通过诸如预训练Devlin（[2018](https://arxiv.org/html/2412.11716v1#bib.bib8)）、Achiam等人（[2023](https://arxiv.org/html/2412.11716v1#bib.bib1)）、微调Raffel等人（[2020](https://arxiv.org/html/2412.11716v1#bib.bib38)）以及其他形式的人类监督训练Ouyang等人（[2022](https://arxiv.org/html/2412.11716v1#bib.bib32)）等方法，取得了显著突破。然而，这些方法可能导致灵活性不足，并且需要大量高质量数据和繁重的人类监督。因此，自我进化方法的开发逐渐得到推动。这些方法使得基于LLM的智能体能够通过自我进化策略自主地获取、精炼和学习。例如，Agent Hospital Li等人（[2024](https://arxiv.org/html/2412.11716v1#bib.bib21)）引入了自我进化到没有真实世界环境的世界模拟中。Self-Align Sun等人（[2024](https://arxiv.org/html/2412.11716v1#bib.bib44)）结合了基于原则的推理和LLM的生成能力，实现了智能体与人工标注的自我对齐。ExpeL Zhao等人（[2024](https://arxiv.org/html/2412.11716v1#bib.bib54)）从成功的历史轨迹中积累经验。在本文中，我们将介绍关于注意力机制和序列可预测性的见解，用于在医学教育领域中执行自主进化。

## 3 EvoPatient

我们提出了EvoPatient，这是一个由三个核心模块驱动的医生培训框架：1）模拟流程将诊断过程镜像为一系列可管理的阶段，作为模拟的工作流。2）模拟智能体对包括一个病人智能体和多个医生智能体，参与自主的多轮对话。病人智能体扮演不同的角色，而医生智能体则进行多学科会诊，根据病历生成问题和答案。3）共进化机制验证并存储对话，创建一个参考库，用于向病人智能体呈现标准化内容。同时，医生智能体从存储的对话轨迹中提取捷径，使得它们能够提出越来越专业的问题，以便高效地培训病人智能体（算法 LABEL:alg:algorithm）。

### 3.1 模拟流程

模拟流程（$\mathcal{F}$）利用现实世界的医疗记录作为输入，并模拟代理对话，创建一个结构化的诊断阶段序列（$\mathcal{S}$）。例如，在问诊阶段，如图[1](https://arxiv.org/html/2412.11716v1#S2.F1 "图1 ‣ 模拟伙伴 ‣ 2 相关工作 ‣ 大型语言模型通过代理共进化模拟标准化患者")所示，一名医生代理（$\mathcal{D}$）与患者代理（$\mathcal{P}$）进行多轮对话（$\mathsf{C}$）。医生代理提出（$\rightarrow$）问题，而患者代理回答（$\leadsto$），最终形成诊断结论。每个阶段（$\tau$）由多个角色之间的一轮或多轮对话组成：

|  |  | $\displaystyle\mathcal{F}={\langle\mathcal{S}^{1},\mathcal{S}^{2},\dots,% \mathcal{S}^{ | \mathcal{F} | }\rangle}_{\circlearrowleft},$ |  | (1) |
| --- | --- | --- | --- | --- | --- | --- |
|  |  | $\displaystyle\mathsf{C}(\mathcal{D},\mathcal{P})={\langle\mathcal{D}% \rightarrow\mathcal{P},\ \mathcal{P}\leadsto\mathcal{D}\rangle}_{\circlearrowleft},$ |  |
|  |  | $\displaystyle\mathcal{S}^{i}=\tau(\mathsf{C}(\mathcal{D},\mathcal{P}),\mathsf{C}(\mathcal{D},\mathcal{D}),\mathsf{C}(\mathcal{P},\mathcal{D}))$ |  |

尽管工作流程在概念上是直接的，但定制阶段的能力使得可以模拟多种场景，而无需额外的代理通信协议或对工作流程拓扑进行调整。本文采用了一个包含主诉生成、分诊、问诊和结论的工作流程。详细描述请见附录 [E](https://arxiv.org/html/2412.11716v1#A5 "附录 E 模拟流程 ‣ 大型语言模型通过代理共进化模拟标准化患者")。

### 3.2 模拟代理对

模拟代理对由一名患者代理和多名医生代理组成，参与多轮诊断对话，有效消除了人工干预和针对不同案例的特定调整需求。

##### 模拟患者代理

为了使患者代理能够生成更具现实性和上下文适应性的回答，与现实世界中的患者保持一致，我们开发了5000个患者档案，涵盖了家庭、教育、经济状况等多样化背景，以及基于大五人格特质McCrae和Costa（[1987](https://arxiv.org/html/2412.11716v1#bib.bib28)）的开放性等个性特征。为了防止代理在长篇上下文中失去信息，我们采用了检索增强生成（RAG）技术，Lewis等人（[2020](https://arxiv.org/html/2412.11716v1#bib.bib19)）利用该技术从记录中提取最相关的信息进行答案生成。

![参考标题](img/7ab8b414e61b1e6a5558745281d71de6.png)

图2：我们框架中的多学科过程。

##### 模拟医生代理

基于预训练模型的医生代理直接向患者提出量身定制的专业问题是具有挑战性的，而这些问题是引发有价值对话以推动进一步演进过程的关键。为了避免问题停留在琐碎的层面，除了提供精心设计的档案Kim等人（[2024](https://arxiv.org/html/2412.11716v1#bib.bib17)）之外，我们还在模拟之前向医生代理提供一些患者记录，并指示他们制定涵盖关键信息（如症状、检查、生活方式）的问题。这种方法有助于医生代理根据其专业知识创建一个专业的问题池，后续模拟中可以参考这一池¹¹1通过整个模拟过程提供患者记录，使问题更加准确，而不是像人类医生那样逐步推进并保持逻辑连续性，这可能会妨碍患者代理的进一步演进过程，影响现实世界的医生训练。 此外，来自不同学科的医生具有不同的专业知识，这导致问题的类型和方面各不相同Epstein（[2014](https://arxiv.org/html/2412.11716v1#bib.bib9)）；Taberna等人（[2020](https://arxiv.org/html/2412.11716v1#bib.bib45)）。这种多样性对于患者代理从不同角度有效学习至关重要。为了模拟这种跨学科的咨询过程，我们使每个医生代理在患者的病情超出其专业范围时，能够招募其他学科的代理，如图[2](https://arxiv.org/html/2412.11716v1#S3.F2 "图2 ‣ 模拟患者代理 ‣ 3.2 模拟代理对 ‣ 3 EvoPatient ‣ LLMs可以通过代理协同进化模拟标准化患者")所示。当被招募时，这些代理将提出问题并决定是否需要招募其他医生：

|  |  | $\displaystyle\rho(\mathcal{D}^{i},\mathcal{P},\mathcal{D}^{j})=(\rho(\mathcal{% D}^{i},\mathcal{P}),\rho(\mathcal{D}^{i},\mathcal{D}^{j})),$ |  | (2) |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle\rho(\mathcal{D}^{i},\mathcal{P})=(\mathcal{D}^{i}\rightarrow% \mathcal{P},\mathcal{P}\leadsto\mathcal{D}^{i})_{\circlearrowleft},$ |  |
|  |  | $\displaystyle\rho(\mathcal{D}^{i},\mathcal{D}^{j})=(\mathcal{D}^{i}\rightarrow% \mathcal{D}^{j})_{\circlearrowleft},$ |  |

其中，$\rho(\cdot)$表示跨学科咨询过程中的互动。我们将招聘过程遵循拓扑排序Kahn（[1962](https://arxiv.org/html/2412.11716v1#bib.bib16)），并形成一个有向无环图（DAG），该图可以防止信息倒流，从而消除额外设计的需求：

|  |  | $\displaystyle\mathcal{G}=(\mathcal{V},\mathcal{E}),$ |  | (3) |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle\mathcal{V}=\{\mathcal{D}^{i}\mid\mathcal{D}^{i}\in\mathrm{D}\}\ % \ \ \mathcal{E}=\{\langle\mathcal{D}^{i},\mathcal{D}^{j}\rangle\ \mid\mathcal{% D}^{i}\neq\mathcal{D}^{j}\},$ |  |

其中，$\mathcal{V}$表示从预先设计的医生集合$\mathrm{D}$中招募的医生代理的集合，$\mathcal{E}$表示招募边的集合。这个过程的迭代性使得医生代理能够在固有的随机拓扑结构中融入各种专业知识，研究表明，这种方式在多代理系统中具有优势，Qian 等人（[2024b](https://arxiv.org/html/2412.11716v1#bib.bib37)），从而提升了诊断过程并促进了更高效的进化过程。

##### 记忆

对于代理来说，记住之前的对话至关重要，以确保诊断的多样性和全面性。然而，不受限制的信息交换可能导致上下文爆炸，Liu 等人（[2024](https://arxiv.org/html/2412.11716v1#bib.bib24)）；Xu 等人（[2023](https://arxiv.org/html/2412.11716v1#bib.bib50)）。为了解决这个问题，我们实现了即时记忆和总结记忆，以调节上下文的可见性。即时记忆保持最近交流的连续性，而总结记忆则整合来自先前对话的关键信息，以保持上下文意识，使代理能够生成新的非随意问题和答案。更多细节请见附录[H](https://arxiv.org/html/2412.11716v1#A8 "附录 H 记忆控制 ‣ LLMs 通过代理共同进化模拟标准化病人")。

### 3.3 共同进化

为了有效地标准化代理的表现模式，我们提出了一种进化机制，该机制通过模拟自动收集、验证²²2我们通过附录[D](https://arxiv.org/html/2412.11716v1#A4 "附录 D 多步骤验证 ‣ LLMs 通过代理共同进化模拟标准化病人")中所示的多步骤验证对话进行验证，并将经验存储在库中。

#### 3.3.1 注意力库

鉴于SP需求的固有复杂性，Levine 等人（[2013](https://arxiv.org/html/2412.11716v1#bib.bib18)）指出，进化过程涉及将需求³³3详细需求见附录[C](https://arxiv.org/html/2412.11716v1#A3 "附录 C 初始 SP 需求 ‣ LLMs 通过代理共同进化模拟标准化病人")划分为每个问题的多个部分。然后，注意力代理识别并精炼每个部分的关键内容，接着将其合并形成回答生成的注意力需求（$r_{a}$）。如果生成的答案经过验证为高质量，相关信息将以组织化的形式存储在库中，包括医生问题、回答生成记录、高质量答案和注意力需求。这些信息作为标准化的表现示范（$d$）和精炼的需求。在人类医生培训过程中，当提出一个新问题（$q$）时，病人代理会搜索并检索相关记录：

|  |  | $\displaystyle d,r_{a}=\mathbbm{k}(sim(q,\mathcal{L}))\ \ \ (\mathcal{P}\mid d,% r_{a})\rightarrow SP,$ |  | (4) |
| --- | --- | --- | --- | --- |

其中，$sim(\cdot,\cdot)$计算新问题与库中问题之间的相似度，使用外部文本嵌入器。$\mathbbm{k}$表示检索前k个匹配的结果。通过精细化的需求和示范，如图[3](https://arxiv.org/html/2412.11716v1#S3.F3 "Figure 3 ‣ 3.3.1 Attention Library ‣ 3.3 Coevolution ‣ 3 EvoPatient ‣ LLMs Can Simulate Standardized Patients via Agent Coevolution")所示，患者代理立即转变为合格的标准化患者，准备进行医生培训。

![参见说明](img/0cc43c30af896bb771008f7b7d6af988.png)

图 3：通过注意需求和有效的少量示范，将我们的患者代理标准化，以供人类医生培训使用的示例。

#### 3.3.2 轨迹库

相似的疾病通常意味着相似的高质量诊断轨迹($\mathcal{T}$)  李和何（[2023](https://arxiv.org/html/2412.11716v1#bib.bib22)）；高等人等（[2024](https://arxiv.org/html/2412.11716v1#bib.bib12)）。在模拟过程中，医生代理会提出一系列问题($\mathcal{Q}=\{q_{1},q_{2},\dots,q_{n}\}$)，患者代理则以匹配的回答序列($\mathcal{A}=\{a_{1},a_{2},\dots,a_{n}\}$)做出回应。为了降低提出无意义问题导致低效的患者代理训练的可能性，我们验证并存储高质量的对话系列作为预测轨迹($t_{i}$)：

|  |  | $\displaystyle\mathcal{L}=\langle t_{1},t_{2},\dots,t_{&#124;\mathcal{L}&#124;}\rangle,$ |  | (5) |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle t_{i}=\{(q_{j-1},a_{j-1},q_{j},a_{j})\mid q\in\mathcal{Q},a\in% \mathcal{A}\},$ |  |

其中，$(q_{j-1},a_{j-1},q_{j},a_{j})$表示从一个问题$q_{j}$到下一个问题$q_{j+1}$的轨迹。在代理的沟通过程中，当遇到当前回答$a$时，基于与$a_{j-1}$的相似性，代理会提取多个$q_{j}$作为预测问题，并推荐给医生代理进行问题轨迹的优化($\ast$)：

|  |  | $\displaystyle\mathcal{T}^{*}=(\mathcal{T}\mid\mathbbm{k}(sim(a,\mathcal{L}))),$ |  | (6) |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle(\mathcal{D}\mid\mathcal{T}^{\ast})\rightarrow SD.$ |  |

通过有效利用宝贵的对话轨迹，这一范式引导问题朝着更专业、更高效的模式发展，将医生代理转变为标准化医生(SD)代理。

## 4 评估

##### 数据集

我们已从两家合作医院彻底收集了真实的医疗记录，以验证我们的EvoPatient。此研究已获得每家医院伦理委员会的批准，其中包括浙江大学医学院附属第二医院，批准编号为2020-568；中山大学肿瘤防治中心，批准编号为G2023-012-01。经过细致审查这些医疗记录后，我们提取了用于模拟患者案例的有用信息，删除了患者的私人信息，并将其整合为数据集。我们还加入了公开的医学NLP数据集⁴⁴4https://github.com/salgadev/medical-nlp，这是一个用于自然语言处理的医学转录语料库。结果，整个数据集包含了超过20000个不同的病例，包括但不限于肝癌、阑尾炎、胰腺病变、鼻咽癌、肿瘤等疾病。

| 方法 | 范式 | 相关性 | 可靠性 | 鲁棒性 | 能力 |
| --- | --- | --- | --- | --- | --- |
| CoT | ![[未标注图片]](img/d398dddf6af823f487a6551d5767ef24.png) | 0.7157^† | 0.5571^† | 0.6714^† | 0.6481^† |
| CoT-SC (3) | ![[未标注图片]](img/5afe058782598fa5fd3fe6634af90940.png) | 0.7337^† | 0.6123^† | 0.7002^† | 0.6821^† |
| ToT | ![[未标注图片]](img/9b6ec721a9f2a32da072ab604fdd6d55.png) | 0.7469^† | 0.7143^† | 0.7714^† | 0.7442^† |
| 自我对齐 | ![[未标注图片]](img/fdff2c26c8bd1c9da67cb28443e6b6e4.png) | 0.7205^† | 0.7273^† | 0.8148^† | 0.7542^† |
| 少量样本(2) | ![[未标注图片]](img/e7e0615d3faaa246ceec11fad1f2e395.png) | 0.7252^† | 0.7419^† | 0.8207^† | 0.7626^† |
| EvoPatient | ![[未标注图片]](img/1f26f7ee0f1a7a74b7b679286129df46.png) | 0.7589 | 0.8786 | 0.9412 | 0.8597 |

表1：LLM驱动的标准化模拟患者方法的整体表现，涵盖了单一患者代理！[[未标注图片]](img/65ad8ab05cdd59c3c17858e0418343b9.png)范式，典型推理驱动的对齐改进方法和我们的多代理！[[未标注图片]](img/765f672a71527d38b64ea00c600b9e4c.png)共进化方法。所有任务的性能指标已求平均。最高分用粗体表示，第二高分用下划线标注。$\dagger$表示基准与我们方法之间有显著统计学差异（p$\leq$0.05）。

##### 基准

由于没有现有的开源框架旨在实现完全自主的标准化患者模拟，我们选择了一些稳健的推理方法和知名的工作进行定量比较。基准的详细描述可以在附录 [A](https://arxiv.org/html/2412.11716v1#A1 "附录A 基准 ‣ LLM可以通过代理共进化模拟标准化患者")中找到。

##### 指标

评估医疗教育中代理生成的问题和答案是一项具有挑战性的任务，因为它需要与各种详细要求对齐。在模拟标准化病人场景的背景下，受Chen等人（[2023b](https://arxiv.org/html/2412.11716v1#bib.bib7)）的启发，我们提出了以下用于答案的评估指标：相关性（$\alpha\in[0,1]$）、忠实度（$\beta\in[0,1]$）、鲁棒性（$\gamma\in[0,1]$）和能力（$\frac{\alpha+\beta+\gamma}{3}\in[0,1]$）。这些维度全面评估答案，同时保留了重要细节。对于问题的评估，我们使用了指标：特异性（$\delta\in[0,1]$）、针对性（$\epsilon\in[0,1]$）、专业性（$\zeta\in[0,1]$）和质量（$\frac{\delta+\epsilon+\zeta}{3}\in[0,1]$）来评估其整体质量⁵⁵5对于每个问题和答案，指标值为0或1，经过多个案例的平均后，值的范围为[0, 1]。这些指标的详细描述可在附录[B](https://arxiv.org/html/2412.11716v1#A2 "Appendix B Metrics ‣ LLMs Can Simulate Standardized Patients via Agent Coevolution")中找到。

##### 实现细节

对于中文数据集，我们使用了强大的预训练大语言模型Qwen 2.5 72B，对于英文数据集则使用了ChatGPT-3.5，GPT4用于配对评估，所有模型的温度设置为1。我们框架的默认训练案例数量为200。医生和患者代理的最大交互回合数为10。每个库中由外部文本嵌入器计算的每个指标（问题或答案）的相似度阈值为0.9。评估中的所有基准都共享相同的超参数和设置，以确保公平。我们通过附录[D](https://arxiv.org/html/2412.11716v1#A4 "Appendix D Multi-Step Validation ‣ LLMs Can Simulate Standardized Patients via Agent Coevolution")中展示的多步骤验证对每个指标的结果进行评分。 (n)案例表示在n个案例上训练我们的框架。

### 4.1 总体分析

表格 [1](https://arxiv.org/html/2412.11716v1#S4.T1 "Table 1 ‣ Datasets ‣ 4 Evaluation ‣ LLMs Can Simulate Standardized Patients via Agent Coevolution") 展示了 EvoPatient 框架与基线方法的全面对比分析，其中医生代理自主提出约 3,000 个问题，涵盖 150 个病例，在所有指标上显著优于所有基线方法。首先，EvoPatient 在 Tree-of-Thought（一个强大的推理方法）上的改进表明，即使是多步骤规划和推理，没有合适的示范和要求，LLMs 很难模拟合格的标准化病人（SP）。这一结果突出了利用历史对话进行代理标准化的有效性。我们方法的效果主要得益于病人代理能够对简洁而精确的 refined 要求进行对齐，并通过少量示范学习期望的回答模式。此外，与自对齐和少量示范方法相比，EvoPatient 将能力从 0.7542 和 0.7626 提升到了 0.8597。这一进展强调了同时为病人代理提供 refined 要求和示范的重要性。同时，在强大医生代理的支持下，我们框架中积累的经验对于代理问答更加有价值，能提供更强大、可信、准确且灵活的答案。

为了更好地理解实际环境中的用户偏好，采用人类专家和 GPT-4 模型对由不同方法生成的回答进行了成对比较，以确定偏好。所有方法均使用相同的提问列表和病人信息，以确保公平比较。如表格 [3](https://arxiv.org/html/2412.11716v1#S4.T3 "Table 3 ‣ 4.1 Overall Analysis ‣ 4 Evaluation ‣ LLMs Can Simulate Standardized Patients via Agent Coevolution") 所示，EvoPatient 在标准问题和作弊问题场景中始终优于其他基线方法，在 GPT-4 和人类专家的评估中，偏好率较高。使用的问题示例可参见附录 [M](https://arxiv.org/html/2412.11716v1#A13 "Appendix M Example of Questions ‣ LLMs Can Simulate Standardized Patients via Agent Coevolution")。

| 方法 | 时长 (秒) | 代币数 | 词数 |
| --- | --- | --- | --- |
| CoT | 04.7500 | 0782.0571 | 45.7429 |
| CoT-SC (3) | 12.5559 | 5837.0286 | 49.8667 |
| ToT | 21.7040 | 2679.3428 | 38.9143 |
| 自对齐 | 09.5146 | 1307.9435 | 51.0636 |
| Few-shot (2) | 04.7182 | 0959.4355 | 35.6334 |
| (50) 病例 | 06.7808 | 0445.3482 | 36.5571 |
| EvoPatient | 06.6922 | 0401.5882 | 32.2432 |

表格 2：回答统计包括各方法在每个回答中所消耗的时长（Duration）、使用的代币数（#Tokens）和总词数（#Words）。最佳的成本为粗体，第二高的为下划线。

此外，我们在表格[2](https://arxiv.org/html/2412.11716v1#S4.T2 "表 2 ‣ 4.1 总体分析 ‣ 4 评估 ‣ LLMs 可以通过代理共同进化模拟标准化患者")中展示了一个回答统计实验。结果表明，EvoPatient在计算效率和输出质量上都表现优异。具体而言，EvoPatient的平均响应时间为6.6922秒，仅次于CoT和少量样本（2）方法。此外，EvoPatient通过优化注意力需求显著减少了提示输入的长度，从而显著降低了令牌成本。对回答内容的进一步分析表明，进化过程使得SP代理能够提供更准确、更稳定的答案，从而在提高答案质量的同时减少了答案中的字数。

| 问题类型 | 标准问题 | 作弊问题 |
| --- | --- | --- |
| 方法 | 评估者 | 基准获胜 | 我方获胜 | 平局 | 基准获胜 | 我方获胜 | 平局 |
| CoT | GPT-4 | 22.50% | 77.08% | 00.42% | 06.67% | 90.08% | 03.25% |
| 人类 | 09.35% | 45.26% | 45.39% | 00.17% | 86.13% | 13.70% |
| CoT-SC（3） | GPT-4 | 30.50% | 62.08% | 07.42% | 06.97% | 86.25% | 06.78% |
| 人类 | 11.43% | 31.43% | 57.14% | 00.23% | 85.43% | 14.34% |
| ToT | GPT-4 | 25.82% | 45.60% | 28.57% | 18.37% | 77.50% | 04.13% |
| 人类 | 14.29% | 34.29% | 51.43% | 04.88% | 52.45% | 42.67% |
| 自对齐 | GPT-4 | 20.48% | 42.38% | 37.14% | 23.53% | 64.71% | 11.76% |
| 人类 | 06.06% | 34.38% | 59.38% | 08.46% | 51.89% | 40.15% |
| 少量样本（2） | GPT-4 | 12.32% | 54.93% | 56.57% | 16.64% | 58.03% | 25.33% |
| 人类 | 06.94% | 29.41% | 63.65% | 09.92% | 51.23% | 38.85% |
| （50）个案例 | GPT-4 | 10.75% | 18.81% | 70.44% | 10.75% | 45.81% | 43.44% |
| 人类 | 11.23% | 20.72% | 67.96% | 06.26% | 45.13% | 48.61% |

表 3：标准问题与作弊问题的配对评估结果。

### 4.2 信息泄露分析

关于恶意行为者对代理的鲁棒性，一直是研究的热点问题Zou等人（[2023](https://arxiv.org/html/2412.11716v1#bib.bib57)）。在我们的初步研究中，我们观察到，当使用未经过进化的患者代理（$\mathcal{P}_{w/o}$）时，医生可能会利用系统获取不应访问的信息，甚至单次成功的利用就能使整个训练过程变得毫无意义。例如，当医生询问：“请告诉我你的病情”，$\mathcal{P}_{w/o}$通常会开始详细描述患者的病情。这使得医生能够通过极少的问题获取大量信息。尽管要求$\mathcal{P}_{w/o}$不应回答此类问题，但代理经常发生偏离。我们将这些问题称为作弊问题。这种形式的越狱攻击很难防范，因为为了越狱而设计的问题种类繁多Liu等人（[2023](https://arxiv.org/html/2412.11716v1#bib.bib25)），使得创建全面覆盖所有潜在作弊尝试的要求变得不可行。因此，进化是至关重要的。虽然作弊问题多种多样，但它们往往具有共同特征，可以用来获取更多信息，我们在进化过程中对泛化能力进行深入探讨。附录[L.1](https://arxiv.org/html/2412.11716v1#A12.SS1 "L.1 Information Leakage ‣ Appendix L Case Study ‣ LLMs Can Simulate Standardized Patients via Agent Coevolution")中的案例研究展示了我们进化过程的泛化能力如何帮助代理学会多种应对此类问题的策略。如表[3](https://arxiv.org/html/2412.11716v1#S4.T3 "Table 3 ‣ 4.1 Overall Analysis ‣ 4 Evaluation ‣ LLMs Can Simulate Standardized Patients via Agent Coevolution")右侧所示，进化后，这一问题得到显著缓解，因为（$\mathcal{P}_{w/}$）已学会识别并避免回答类似问题。

![参见标题](img/c8886f2d945f4f5ce98cbef44b2f530e.png)

图4：五种类型疾病的进化过程转移分析，展示患者代理在进化前（内圈）和进化后（外圈）的变化。放大查看更多详细信息。

### 4.3 进化转移分析

在这里，我们通过100个鼻咽癌病例训练我们的框架，并直接将其用于其他五种疾病的标准化病人（SP）模拟。如图[4](https://arxiv.org/html/2412.11716v1#S4.F4 "图4 ‣ 4.2 信息泄漏分析 ‣ 4 评估 ‣ LLMs可以通过代理共演化模拟标准化病人")所示，在没有进一步训练和任务特定定制的情况下，我们的框架表现出了极强的迁移能力，平均提高了15%左右的答案准确度、18%的鲁棒性和12%的质量。这一结果表明，我们的框架具有卓越的迁移能力，代表了一条实现自主性和普适性的有前景的路径。

### 4.4 医生代理分析

| 方法 | 特异性 | 针对性 | 专业性 | 质量 |
| --- | --- | --- | --- | --- |
| 无进化 | 0.4801 | 0.3843 | 0.6140 | 0.4928 |
| 进化 | 0.6164 | 0.4242 | 0.8120 | 0.6176 |

表4：带有和不带有进化过程的医生代理问题对比。

##### 医生进化

我们通过让医生代理在100个病例中提问2,000个问题，比较了带有进化过程（$\mathcal{D}_{w/}$）和不带进化过程（$\mathcal{D}_{w/o}$）的性能。表[4](https://arxiv.org/html/2412.11716v1#S4.T4 "表4 ‣ 4.4 医生代理分析 ‣ 4 评估 ‣ LLMs可以通过代理共演化模拟标准化病人")中的结果显示，进化过程显著提高了医生代理的表现，将质量从0.4928提高到0.6176，表明其在制定质量较高的医学问题上更加专注于收集相关的诊断信息。进一步分析问题类型的分布，如图[5](https://arxiv.org/html/2412.11716v1#S4.F5 "图5 ‣ 医生进化 ‣ 4.4 医生代理分析 ‣ 4 评估 ‣ LLMs可以通过代理共演化模拟标准化病人")所示，进一步证明了我们医生进化过程的有效性。检查相关问题的比例从14.09%增加到25.57%，这一水平几乎不可能由新手医生代理达到，显著有利于病人代理的进化。

![参见说明文字](img/1cfffb43a79ddf91b95275c63babcd47.png)

图5：医生代理在有（右）和没有（左）进化过程的情况下，五个问题类型的分布情况，问题类型的详细描述可在附录[I](https://arxiv.org/html/2412.11716v1#A9 "附录 I 问题类型 ‣ LLMs可以通过代理共演化模拟标准化病人")中找到。

![参见说明文字](img/21a4dbca70a440c028720a122545d4ba.png)

图6：不同医生代理设置对注意力库积累率的影响。

##### 医生招募

我们进一步使用$\mathcal{D}_{w/}$和$\mathcal{D}_{w/o}$研究了病人代理演化过程中的医生招募过程。如图[6](https://arxiv.org/html/2412.11716v1#S4.F6 "图6 ‣ 医生演化 ‣ 4.4 医生代理分析 ‣ 4 评估 ‣ LLMs可以通过代理共演化模拟标准化病人")所示，当使用$\mathcal{D}_{w/}$且未进行招募，仅有一个学科医生提问时，注意力库的积累率下降。这个下降主要是由于$\mathcal{D}_{w/}$提出了更多有针对性和高效的问题，而$\mathcal{D}_{w/o}$提出的是多样化但随机且质量较低的问题。医生招募过程显著缓解了这一下降。通过利用库中的预测轨迹，不同学科的演化医生能够提出更专业的问题，而不是通用问题。这显著提高了问题的多样性，同时确保了其专业性，从而导致注意力库更加多样化和专业化。

| 方法 | 相关性 | 准确性 | 鲁棒性 | 能力 |
| --- | --- | --- | --- | --- |
| 医生代理 | 0.7699 | 0.8000 | 0.8533 | 0.7922 |
| + 招募 | 0.7875 | 0.8233 | 0.8733 | 0.7980 |
| + 演化 | 0.7707 | 0.8400 | 0.9100 | 0.8202 |
| + 招募 + 演化 | 0.7906 | 0.8567 | 0.9333 | 0.8535 |

表 5：关于病人代理演化中的医生代理的消融研究。符号“+”代表添加操作。招募意味着允许医生代理招募其他医生代理，演化意味着使用已演化的医生代理。最佳得分为**粗体**，第二高的得分下划线标出。

##### 对病人代理的影响

由于医生代理主导了注意力库的更新，直接影响病人代理的回答质量。因此，我们进一步通过病人回答的质量分析医生代理的招募与演化策略的影响，如表[5](https://arxiv.org/html/2412.11716v1#S4.T5 "表5 ‣ 医生招募 ‣ 4.4 医生代理分析 ‣ 4 评估 ‣ LLMs可以通过代理共演化模拟标准化病人")所示。结果表明，实施医生代理的招募与演化策略能更有效地演化病人代理。具体来说，通过演化医生代理训练的病人代理的能力刺激性较强，表明仅有招募能力时，医生代理仍然难以提出能够积极促进注意力库内容质量的专业问题。当结合招募与演化时，进一步观察到改进，达到了所有指标中的最高表现。这一全面改进证实了这两种策略的良好兼容性。

## 5 结论

由于缺乏让病人代理通过模拟多样案例进行学习的机制，我们引入了EvoPatient，这一创新的模拟框架使得病人和医生代理能够通过共同进化机制自主积累过去的经验。因此，病人代理能够高效管理各种模拟案例，用于人类医生的训练，而医生代理则提高了其提问能力，从而提高了病人代理的训练效率。定量分析显示，回答质量显著提升，导致更稳定、强大、准确的回答模式，并优化了资源消耗。我们预期我们的研究成果将激发对基于LLM的模拟伙伴的进一步研究，强调自主进化的重要性，并推动代理朝着实现更大真实感的模拟目标迈进。

## 6 限制

我们的研究探讨了如何通过自主进化在医学教育中标准化模拟代理的展示模式。然而，研究人员和实践者在将这些见解应用于新技术或新应用的开发时，应考虑一些限制和风险。

首先，从模拟能力的角度来看，自动化代理完全替代人类模拟伙伴的能力可能被高估了。例如，尽管EvoPatient在许多问题和案例中增强了代理的展示能力，自动化病人代理有时无法复制真实人类标准化病人的全部能力。人类标准化病人的复杂性和模糊性使得很难为角色扮演定义一套完美的要求。当面对不熟悉或带有陷阱的问题时，代理即使接收到角色分配和示范，仍然有时无法提供适当的回应。这表明基于LLM的代理可能难以完全理解其角色的潜在意图，而不仅仅是遵循提供的指令。如果没有清晰、详细的指令，代理可能表现得像是答题机——以病人般的方式回应，但缺乏真正的病人行为。因此，我们建议在进化过程中为病人代理定义明确的、逐步的要求。考虑到当前代理的能力，满足高度详细的要求可能并不总是能得到保证，这突显了在具体性和实际可行性之间保持平衡的必要性。此外，目前病人代理只能提供基于文本的回应，而真实的标准化病人则传达额外的非语言线索，如语调和面部表情。这些线索对于训练医生根据病人的外在表现进行适当的询问和诊断至关重要。

其次，在医生代理方面，即使进行了角色分配，自动代理仍然难以像经验丰富的人类医生那样提问准确且专业的问题。尽管通过允许医生代理形成问题池、招募其他学科角色分配的医生代理并通过模拟过程积累经验可以缓解这一挑战，但这些方法在面对具有巨大差异的新疾病时可能缺乏普适性。未来的研究应集中在提升医生代理在学科层面的专业性，使其在各种疾病中都能真正具备多功能性。

第三，从评估角度来看，医学教育中模拟过程的复杂性，加上缺乏有效的自动化评估标准——如可执行性或能够分解对话进行多步骤评估的能力（Qian 等 [2024a](https://arxiv.org/html/2412.11716v1#bib.bib36); Zhuge 等 [2024](https://arxiv.org/html/2412.11716v1#bib.bib55))——使得自动化对话评估具有极高的挑战性。虽然人工评估通常能得到最可靠的结果，但基于患者记录在情境中评估数千个对话是劳动密集型的，甚至是不切实际的。本文则强调了客观维度，如患者代理的相关性、忠实性、鲁棒性和整体能力，以及医生代理的针对性、专业性和整体质量。然而，未来的研究应考虑其他维度，包括语音语调、可读性、用户友好性等。制定完全公平且客观的评估标准仍然是一个重大挑战。因此，在可预见的未来，代理评估可能需要针对特定医疗场景进行定制。

其次，尽管少量示范、精细化需求和历史对话中的快捷对话轨迹可以提升代理的真实性，但一些低质量的对话仍可能存储在库中，并作为参考被提取，从而在标准化展示中负面影响代理的表现。虽然我们实施了一种演化修正策略（参见附录 [G](https://arxiv.org/html/2412.11716v1#A7 "Appendix G Evolution Correction ‣ LLMs Can Simulate Standardized Patients via Agent Coevolution")）来移除低质量内容，但一些深层隐藏的问题仍难以检测。因此，未来的研究应探索更准确评估演化库中内容质量的方法。

尽管存在这些局限性，我们相信它们为未来的研究提供了宝贵的见解，并且通过吸引更广泛且具备技术能力的受众可以得到缓解。我们预计这些发现将为提升模拟代理的真实性以及它们在LLM驱动代理不断发展的环境中的作用提供重要贡献。

## 7 伦理考量

##### 参与者招募

注释专家是指持有临床医学或相关领域研究生学位（硕士或博士）的个人，或者是正在攻读该学位的人员。我们会为每位专家及其他参与者支付参与费用。

##### 系统与数据使用

本研究中开发的所有数据和框架仅用于学术研究和教育目的。该框架不适合未经进一步开发的实际部署，包括大规模训练和测试、遵循真实医院环境中的部门和行政协议以及用户和专家的全面评估。本研究中使用的所有医院患者记录均已完全去标识化，并已获得用于研究的同意。数据中不包含患者或医院员工的个人可识别信息。此外，数据已进行匿名处理，排除了敏感信息，确保其仅用于学术研究。

## 8 致谢

本研究部分得到了中国国家自然科学基金的资助，资助号为92259202和72074188，浙江省“先锋”和“领军人物”研发计划的资助，资助号为2024C01167，浙江省自然科学基金资助，资助号为LD24F020011，以及广州市重点研发计划资助，资助号为2024B01J1301。

## 参考文献

+   Achiam等人（2023）Josh Achiam、Steven Adler、Sandhini Agarwal、Lama Ahmad、Ilge Akkaya、Florencia Leoni Aleman、Diogo Almeida、Janko Altenschmidt、Sam Altman、Shyamal Anadkat等人。2023年。Gpt-4技术报告。*arXiv预印本arXiv:2303.08774*。

+   Barrows（1993）Howard S Barrows。1993年。标准化患者在教学和评估临床技能中的应用概述。aamc。*学术医学*，68(6):443–51。

+   Bokken等人（2006）Lonneke Bokken、Jan Van Dalen和Jan-Joost Rethans。2006年。模拟对担任模拟患者的人的影响：焦点小组研究。*医学教育*，40(8):781–786。

+   Brown等人（2020）Tom Brown、Benjamin Mann、Nick Ryder、Melanie Subbiah、Jared D Kaplan、Prafulla Dhariwal、Arvind Neelakantan、Pranav Shyam、Girish Sastry、Amanda Askell等人。2020年。语言模型是少量样本学习者。*神经信息处理系统进展*，33:1877–1901。

+   Bubeck等人（2023）Sébastien Bubeck、Varun Chandrasekaran、Ronen Eldan、Johannes Gehrke、Eric Horvitz、Ece Kamar、Peter Lee、Yin Tat Lee、Yuanzhi Li、Scott Lundberg等人。2023年。人工通用智能的火花：关于gpt-4的早期实验。*arXiv预印本arXiv:2303.12712*。

+   Chen等人（2023a）Siyuan Chen、Mengyue Wu、Kenny Q Zhu、Kunyao Lan、Zhiling Zhang和Lyuchun Cui。2023a年。LLM赋能的聊天机器人用于精神科医生和患者模拟：应用与评估。*arXiv预印本arXiv:2305.13614*。

+   Chen 等人（2023b）Siyuan Chen, Mengyue Wu, Kenny Q Zhu, Kunyao Lan, Zhiling Zhang, 和 Lyuchun Cui. 2023b. 基于LLM的心理医生与患者模拟对话机器人：应用与评估。*arXiv 预印本 arXiv:2305.13614*。

+   Devlin（2018）Jacob Devlin. 2018. Bert：用于语言理解的深度双向变换器预训练。*arXiv 预印本 arXiv:1810.04805*。

+   Epstein（2014）Nancy E Epstein. 2014. 跨学科住院团队改善患者预后：综述。*外科神经学国际*，5（增刊7）：S295。

+   Feltz 等人（2020）Deborah L Feltz, Christopher R Hill, Stephen Samendinger, Nicholas D Myers, James M Pivarnik, Brian Winn, Alison Ede, 和 Lori Ploutz-Snyder. 2020. 模拟伙伴是否能够提升长期锻炼中的努力程度？*《力量与体能研究杂志》*，34(9):2434–2442。

+   Feltz 等人（2016）Deborah L Feltz, Lori Ploutz-Snyder, Brian Winn, Norbert L Kerr, James M Pivarnik, Alison Ede, Christopher Hill, Stephen Samendinger, 和 William Jeffery. 2016. 模拟伙伴与协作性锻炼（空间）以提高宇航员的动机：研究协议。*BMC 心理学*，4:1–11。

+   Gao 等人（2024）Weihao Gao, Fuju Rong, Lei Shao, Zhuo Deng, Daimin Xiao, Ruiheng Zhang, Chucheng Chen, Zheng Gong, Zhiyuan Niu, Fang Li 等人. 2024. 利用多模态知识图谱提升眼科病历管理。*科学报告*，14(1):23221。

+   Graesser 等人（2004）Arthur C Graesser, Shulan Lu, George Tanner Jackson, Heather Hite Mitchell, Mathew Ventura, Andrew Olney, 和 Max M Louwerse. 2004. Autotutor：一种具有自然语言对话的辅导系统。*行为研究方法、工具与计算机*，36:180–192。

+   Hua 等人（2023）Wenyue Hua, Lizhou Fan, Lingyao Li, Kai Mei, Jianchao Ji, Yingqiang Ge, Libby Hemphill, 和 Yongfeng Zhang. 2023. 战争与和平（WarAgent）：基于大型语言模型的世界大战多代理模拟。*arXiv 预印本 arXiv:2311.17227*。

+   John 等人（2008）Oliver P John, Laura P Naumann, 和 Christopher J Soto. 2008. 向综合型大五人格特质分类法的范式转变。*人格手册：理论与研究*，3(2):114–158。

+   Kahn（1962）Arthur B Kahn. 1962. 大型网络的拓扑排序。*ACM 通信*，5(11):558–562。

+   Kim 等人（2024）Yubin Kim, Chanwoo Park, Hyewon Jeong, Yik Siu Chan, Xuhai Xu, Daniel McDuff, Hyeonhoon Lee, Marzyeh Ghassemi, Cynthia Breazeal, 和 Hae Won Park. 2024. Mdagents：基于LLMs的医学决策协作适应性。发表于*第38届神经信息处理系统年会*。

+   Levine 等人（2013）Adam I Levine, Samuel DeMaria Jr, Andrew D Schwartz, 和 Alan J Sim. 2013. *《医疗保健模拟综合教材》*。Springer Science & Business Media。

+   Lewis等人（2020年）Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel等人。2020年。用于知识密集型NLP任务的检索增强生成。*《神经信息处理系统进展》*，33:9459–9474。

+   Li等人（2023a）Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin和Bernard Ghanem。2023a。Camel：用于“大规模语言模型社会”中的“思想”探索的交互式代理。*arXiv预印本arXiv:2303.17760*。

+   Li等人（2024年）Junkai Li, Siyu Wang, Meng Zhang, Weitao Li, Yunghwei Lai, Xinhui Kang, Weizhi Ma和Yang Liu。2024年。代理医院：具有可进化医学代理的医院模拟。*arXiv预印本arXiv:2405.02957*。

+   Li和He（2023年）Qing Li和Song He。2023年。基于孪生网络的医学问题相似度匹配。*《BMC医学信息学与决策支持》*，23(1):55。

+   Li等人（2023b）Yuan Li, Yixuan Zhang和Lichao Sun。2023b。Metaagents：通过协作生成代理模拟基于LLM的任务导向协调中的人类行为互动。*arXiv预印本arXiv:2310.06500*。

+   Liu等人（2024年）Wei Liu, Chenxi Wang, Yifei Wang, Zihao Xie, Rennai Qiu, Yufan Dang, Zhuoyun Du, Weize Chen, Cheng Yang和Chen Qian。2024年。信息不对称下的协作任务的自主代理。*arXiv预印本arXiv:2406.14928*。

+   Liu等人（2023年）Yi Liu, Gelei Deng, Zhengzi Xu, Yuekang Li, Yaowen Zheng, Ying Zhang, Lida Zhao, Tianwei Zhang, Kailong Wang和Yang Liu。2023年。通过提示工程破解ChatGPT：一项实证研究。*arXiv预印本arXiv:2305.13860*。

+   Louie等人（2024年）Ryan Louie, Ananjan Nandi, William Fang, Cheng Chang, Emma Brunskill和Diyi Yang。2024年。Roleplay-doh：通过引导并遵循原则，使领域专家能够创建LLM模拟的患者。*arXiv预印本arXiv:2407.00870*。

+   Markel等人（2023年）Julia M Markel, Steven G Opferman, James A Landay和Chris Piech。2023年。GPTteach：与基于GPT的学生进行交互式TA培训。在*《第十届ACM学习@规模会议论文集》*，第226–236页。

+   McCrae和Costa（1987年）Robert R McCrae和Paul T Costa。1987年。五因素人格模型在各类工具和观察者中的验证。*《人格与社会心理学杂志》*，52(1):81。

+   McGaghie等人（2010年）William C McGaghie, S Barry Issenberg, Emil R Petrusa和Ross J Scalese。2010年。基于模拟的医学教育研究的批判性回顾：2003-2009年。*《医学教育》*，44(1):50–63。

+   Moor等人（2023年）Michael Moor, Oishi Banerjee, Zahra Shakeri Hossein Abad, Harlan M Krumholz, Jure Leskovec, Eric J Topol和Pranav Rajpurkar。2023年。通用医学人工智能的基础模型。*《自然》*，616(7956):259–265。

+   Othlinghaus-Wulhorst and Hoppe (2020) Julia Othlinghaus-Wulhorst 和 H Ulrich Hoppe. 2020. 社交技能训练领域中严肃角色扮演游戏的技术与概念框架。*计算机科学前沿*，2:28。

+   Ouyang et al. (2022) Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray, 等. 2022. 训练语言模型通过人类反馈遵循指令。*神经信息处理系统进展*，35:27730–27744。

+   Park et al. (2023) Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang, 和 Michael S Bernstein. 2023. 生成代理：人类行为的互动模拟物。在*第36届年度ACM用户界面软件与技术研讨会论文集*，第1–22页。

+   Park et al. (2022) Joon Sung Park, Lindsay Popowski, Carrie Cai, Meredith Ringel Morris, Percy Liang, 和 Michael S Bernstein. 2022. 社会模拟物：为社交计算系统创建有人物的原型。在*第35届年度ACM用户界面软件与技术研讨会论文集*，第1–18页。

+   Péli and Nooteboom (1997) Gábor Péli 和 Bart Nooteboom. 1997. 供应伙伴关系中的学习模拟。*计算与数学组织理论*，3:43–66。

+   Qian et al. (2024a) 陈倩, 刘伟, 刘洪章, 陈诺, 邓宇凡, 李佳浩, 杨程, 陈伟泽, 苏宇生, 蔡鑫, 等. 2024a. Chatdev: 用于软件开发的交流代理。在*第62届计算语言学会年会论文集（第1卷：长文）*，第15174–15186页。

+   Qian et al. (2024b) 陈倩, 谢子豪, 王一飞, 刘伟, 邓宇凡, 杜卓云, 陈伟泽, 杨程, 刘智远, 和 孙茂松. 2024b. 基于大语言模型的多代理协作扩展。*arXiv预印本arXiv:2406.07155*。

+   Raffel et al. (2020) Colin Raffel, Noam Shazeer, Adam Roberts, Katherine Lee, Sharan Narang, Michael Matena, Yanqi Zhou, Wei Li, 和 Peter J Liu. 2020. 探索统一文本到文本变换器的迁移学习极限。*机器学习研究杂志*，21(140):1–67。

+   Ruan et al. (2019) Sherry Ruan, Liwei Jiang, Justin Xu, Bryce Joe-Kun Tham, Zhengneng Qiu, Yeshuang Zhu, Elizabeth L Murnane, Emma Brunskill, 和 James A Landay. 2019. Quizbot: 一种基于对话的适应性学习系统，用于事实性知识。在*2019年计算机系统人因学会议论文集*，第1–13页。

+   Shaikh et al. (2024) Omar Shaikh, Valentino Emil Chai, Michele Gelfand, Diyi Yang, 和 Michael S Bernstein. 2024. 排练：模拟冲突以教授冲突解决。在*计算机系统人因学CHI会议论文集*，第1–20页。

+   Shanahan et al. (2023) Murray Shanahan, Kyle McDonell, 和 Laria Reynolds. 2023. 与大语言模型的角色扮演。*自然*，623(7987):493–498。

+   Singhal 等人（2023）Karan Singhal，Shekoofeh Azizi，Tao Tu，S Sara Mahdavi，Jason Wei，Hyung Won Chung，Nathan Scales，Ajay Tanwani，Heather Cole-Lewis，Stephen Pfohl 等人. 2023. 大语言模型编码临床知识。*自然*，620（7972）：172–180。

+   Spencer 和 Dales（2006）John Spencer 和 Jill Dales. 2006. 满足模拟病人的需求以及关爱其背后的人的问题？

+   Sun 等人（2024）孙志清，沈宜康，周沁洪，张洪鑫，陈震方，David Cox，杨一鸣，甘创. 2024. 从零开始的语言模型原则驱动自对齐，最小化人类监督。*神经信息处理系统进展*，36。

+   Taberna 等人（2020）Miren Taberna，Francisco Gil Moncayo，Enric Jané-Salas，Maite Antonio，Lorena Arribas，Esther Vilajosana，Elisabet Peralvez Torres 和 Ricard Mesía. 2020. 多学科团队（MDT）方法与护理质量。*肿瘤学前沿*，10:85。

+   Varas 等人（2023）Julian Varas，Brandon Valencia Coronel，IGNACIO VILLAGRáN，Gabriel Escalona，Rocio Hernandez，Gregory Schuit，VALENTINA DURáN，Antonia Lagos-Villaseca，Cristian Jarry，Andres Neyem 等人. 2023. 外科培训中的创新：探索人工智能和大语言模型（LLM）的作用。*巴西外科医生学会杂志*，50:e20233605。

+   Wallace（2007）Peggy Wallace. 2007. *标准化病人的辅导：用于临床能力评估*。Springer Publishing。

+   Wang 等人（2022）王雪智，Jason Wei，Dale Schuurmans，Quoc Le，Ed Chi，Sharan Narang，Aakanksha Chowdhery 和 Denny Zhou. 2022. 自一致性提升大语言模型中的思维链推理。*arXiv 预印本 arXiv:2203.11171*。

+   Wei 等人（2022）Jason Wei，王雪智，Dale Schuurmans，Maarten Bosma，Fei Xia，Ed Chi，Quoc V Le，Denny Zhou 等人. 2022. 思维链提示在大语言模型中引发推理。*神经信息处理系统进展*，35：24824–24837。

+   Xu 等人（2023）徐鹏，魏平，吴显超，Lawrence McAfee，陈柱，刘子涵，Sandeep Subramanian，Evelina Bakhturina，Mohammad Shoeybi 和 Bryan Catanzaro. 2023. 检索与长上下文大语言模型的结合。*arXiv 预印本 arXiv:2310.03025*。

+   Yao 等人（2024）姚顺宇，余玳，Jeffrey Zhao，Izhak Shafran，Tom Griffiths，曹远，Karthik Narasimhan. 2024. 思维树：通过大语言模型进行深思熟虑的问题解决。*神经信息处理系统进展*，36。

+   Yu 等人（2024）胡子渭，周家彦，李凌瑶，陈杉，Jack Gallifant，史安业，李翔，华文月，金名煜，陈广等人. 2024. Aipatient：通过电子健康记录和大语言模型驱动的智能工作流模拟病人。*arXiv 预印本 arXiv:2409.18924*。

+   Zhang 等人（2023）张新路，田晨鑫，杨显俊，陈力畅，李泽坤和Linda Ruth Petzold. 2023. Alpacare：用于医疗应用的指令调优大语言模型。*arXiv 预印本 arXiv:2310.14558*。

+   Zhao 等人（2024）Andrew Zhao, Daniel Huang, Quentin Xu, Matthieu Lin, Yong-Jin Liu, 和 Gao Huang。2024年。Expel：LLM 代理是经验型学习者。载于 *人工智能会议论文集*，第38卷，第19632–19642页。

+   Zhuge 等人（2024）Mingchen Zhuge, Changsheng Zhao, Dylan Ashley, Wenyi Wang, Dmitrii Khizbullin, Yunyang Xiong, Zechun Liu, Ernie Chang, Raghuraman Krishnamoorthi, Yuandong Tian 等人。2024年。Agent-as-a-judge：使用代理来评估代理。*arXiv 预印本 arXiv:2410.10934*。

+   Ziv 等人（2006）Amitai Ziv, Paul Root Wolpe, Stephen D Small, 和 Shimon Glick。2006年。基于模拟的医学教育：一种伦理上的必要性。*医疗保健中的模拟*，1(4)：252–256。

+   Zou 等人（2023）Andy Zou, Zifan Wang, Nicholas Carlini, Milad Nasr, J Zico Kolter, 和 Matt Fredrikson。2023年。面向对齐语言模型的通用和可迁移对抗攻击。*arXiv 预印本 arXiv:2307.15043*。

附录

附加信息随主文一起提供，包含额外的数据、解释和细节。

## 附录 A 基准

1.  $\bullet$

    链式推理（CoT）Wei 等人（[2022](https://arxiv.org/html/2412.11716v1#bib.bib49)）是一种技术上通用且经验上有效的方法，它赋予 LLM 生成一系列连贯的中间推理步骤的能力，通过深思熟虑的思考自然地引导最终解决方案，并使推理能力得以展现。

1.  $\bullet$

    基于链式推理的自一致性（CoT-SC）Wang 等人（[2022](https://arxiv.org/html/2412.11716v1#bib.bib48)）通过对同一问题使用不同的思维过程，从而增强了 CoT，输出决策可以通过探索更丰富的思维集合来更加忠实。我们使用“CoT-SC(n)”来表示采用 CoT 提示方法来采样 n 个推理链，然后利用 SC 方法来选择答案的方式。

1.  $\bullet$

    思维树（ToT）Yao 等人（[2024](https://arxiv.org/html/2412.11716v1#bib.bib51)）通过允许在树结构中探索多个推理路径，适应分支可能性，并支持回溯，从而扩展了 CoT，显著增强了语言模型的解题能力。

1.  $\bullet$

    少样本学习 Brown 等人（[2020](https://arxiv.org/html/2412.11716v1#bib.bib4)）利用包括医院实践中的历史病历和医学文献中的示例病例等经验进行演示。我们借鉴了 Agent Hospital Li 等人（[2024](https://arxiv.org/html/2412.11716v1#bib.bib21)）的这一思路。

1.  $\bullet$

    基于原则的自我对齐（Principle-Driven Self-Alignment）Sun 等人（[2024](https://arxiv.org/html/2412.11716v1#bib.bib44)）定义了一组代理必须遵守的原则，并提供了上下文学习示范，用于构建有益、伦理和可靠的响应。

## 附录 B 度量

在医疗教育领域评估对话是一项具有挑战性的任务，尤其是在试图全面评估时。本文详细描述了我们分析中使用的各项指标。虽然这些维度可能无法涵盖问题和回答评估的各个方面，但它们为评估标准患者代理框架开发初期的工作提供了重要的见解。

##### 患者回答评估指标

1.  $\bullet$

    相关性（$\alpha\in\left[0,1\right]$）衡量回答是否直接试图以完整的句子方式回答问题，并且没有冗余信息。通过问题和答案的语义嵌入之间的余弦距离量化。得分越高，表示回答越有可能准确、相关，并有效地满足用户的查询。

1.  $\bullet$

    忠实度（$\beta\in\left[0,1\right]$）评估患者的回答是否可以从提供的医疗信息中推断出来。同时，确保与标准患者（SP）的要求一致。得分越高，表示患者代理在遵守患者记录和要求方面的忠实度越高。

1.  $\bullet$

    稳健性（$\gamma\in\left[0,1\right]$）评估患者的回答是否透露了医生不应该轻易掌握的信息（例如，疾病名称、病历的详细描述）或在单一问题中提供过多的医疗细节。得分越高，表示医生通过精心设计的欺骗性问题获取信息的可能性较低，这些信息在真实医疗场景中不容易获取。

1.  $\bullet$

    能力（$\frac{\alpha+\beta+\gamma}{3}\in\left[0,1\right]$）是一个综合指标，整合了多个因素来评估患者代理的整体能力，通过平均稳健性、忠实度和回答相关性来量化。得分越高，表示患者代理的整体满意度越高，意味着需求不对齐的可能性较低。

##### 医生问题评估指标

1.  $\bullet$

    特异性（$\delta\in\left[0,1\right]$）衡量医生提问的精确度和明确性，重点关注与患者病例相关的特定症状、病情或背景。得分越高，表示医生避免提出过于宽泛或模糊的问题，而是定制问题以收集详细和可操作的信息，从而支持准确和彻底的诊断。

1.  $\bullet$

    目标性（$\epsilon\in\left[0,1\right]$）评估医生是否提出了有意义和有针对性的问题，旨在收集必要的诊断信息。得分越高，表示医生在收集相关数据以进行准确诊断方面越高效。

1.  $\bullet$

    专业性（$\zeta\in\left[0,1\right]$）评估医生提问是否反映了对医学原则和实践的深刻理解。较高的分数表明问题使用了恰当的医学术语，考虑了基于证据的实践，并展示了对临床指南的认识，从而提高了诊断过程的质量。

1.  $\bullet$

    质量（$\frac{\delta+\epsilon+\zeta}{3}\in\left[0,1\right]$）是一个综合指标，整合了多种因素来评估医生代理人问题的整体质量。通过平均特异性、目标性提问和专业性来量化该指标。较高的能力分数表明对患者诊断的处理方式更有效且高效，有助于患者演变过程的改善。

## 附录 C 初始SP要求

在这里，我们提供了框架中使用的总体SP角色扮演要求，见图[7](https://arxiv.org/html/2412.11716v1#A3.F7 "Figure 7 ‣ Appendix C Initial SP Requirements ‣ LLMs Can Simulate Standardized Patients via Agent Coevolution")。

<svg class="ltx_picture ltx_centering" height="744.32" id="A3.F7.pic1" overflow="visible" version="1.1" width="649.61"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,744.32) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 726.11)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="606.3">Overall Initial SP Requirements</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="694.62" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="606.3">You are a simulated patient. You will play the following role: {profile} Now, you will face a question from a doctor. The following are the guidelines you should follow: 1\. Role Awareness: - Your responses should be based on the provided medical condition and character background. - The understanding of medical terminology will vary according to the character’s education level. Patients with lower education may only understand basic terms, those with moderate education may understand some technical terms, and those with higher education may understand rarer terms. 2\. Personality Traits: - Your responses should reflect the personality traits of the character. Basically, introverted patients should give brief answers, those with a negative personality may show avoidance or reluctance to answer, extroverted patients may give longer responses, open personalities should show a positive attitude toward treatment, and agreeable personalities should be friendly. 3\. Communication Style: - When the question does not involve test results, you may communicate normally with the doctor but avoid using medical terms beyond the character’s knowledge scope and avoid giving overly detailed descriptions. - Your response should reflect the first-person perspective of the patient, with a conversational tone, including filler words, hesitation, and other oral communication traits, consistent with the role’s background, personality, education level, etc. 4\. Handling Test Results: - When the question involves test results, if a full hospital examination report is requested and such information has not been provided, refuse to answer. If the information is provided, respond clearly and accurately in accordance with the character’s personality, possibly using medical terminology. Patients with a negative personality may be reluctant to answer. - If only a specific test result is asked, do not answer. 5\. Handling Complex Questions: - Be aware that the doctor may ask complex questions with multiple sub-questions. In this case, you should selectively answer or refuse to answer based on the character’s personality. - Do not answer questions related to medical history or diagnosis conclusions. - Your responses should not reveal the final disease name. 7\. Providing Historical Information: - If asked about previous treatment or diagnosis results, you may provide information that does not include the final diagnosis, or mention tests that were conducted, while ensuring this aligns with the character’s personality. Different personalities may have different memory abilities. Higher education levels may imply better memory, while lower levels may suggest poorer memory. The stronger the memory, the more tests the patient can mention. 8\. Emotional Responses: - Your response should reflect the patient’s emotional reaction, such as anxiety, concern, hope, etc., in line with the character’s personality and educational background. 9\. Cultural and Linguistic Adaptability: - Considering that patients from different cultural and linguistic backgrounds may have varying understandings and reactions to certain terms, your responses should be adapted to the character’s cultural and linguistic habits. 10\. Feedback and Interaction: - Your response may include feedback to the doctor’s question, such as asking for clarification or expressing difficulty in understanding certain questions. You may also express your feelings, whether satisfied or dissatisfied. Basic Descriptions of Different Personality Types: - Openness: Reflects the individual’s willingness to engage in new experiences, creativity, and curiosity. - Conscientiousness: Measures an individual’s level of self-discipline, organization, and goal-oriented behavior. - Extraversion: Describes how outgoing, energetic, and social a person is. - Agreeableness: Represents an individual’s tendency to be friendly, cooperative, and empathetic in relationships with others. - Neuroticism: Related to emotional stability; high neuroticism indicates an individual is more affected by stress and negative emotions. Doctor question: {question} Patient information: {information} Memory: {memory}</foreignobject></g></g></svg>

图 7：我们框架中使用的总体初始SP要求。

## 附录 D 多步骤验证

##### 答案验证

在我们验证患者代理人回应的方法中，我们采用了多步骤评估，通过利用大规模语言模型（LLMs）来确定回应是否符合既定标准。图[8](https://arxiv.org/html/2412.11716v1#A4.F8 "Figure 8 ‣ Answer Validation ‣ Appendix D Multi-Step Validation ‣ LLMs Can Simulate Standardized Patients via Agent Coevolution")展示了构成我们过程基础的基本验证步骤，这些步骤可以扩展，包括考虑患者的背景和特征。最初，我们会确定问题是否明确提到疾病名称。如果提到，我们会评估回应是否为拒绝回答；如果是，忠诚度得分为1，否则为0，从而防止医生通过猜测间接推断出患者的诊断。如果问题没有提到疾病，我们接着判断是否询问了测试结果。对于涉及测试结果的问题，我们评估是否明确要求提供某项特定测试的信息。如果是，我们再次评估回应是否为拒绝回答；拒绝回答得分为1，其他回应则得分为0，鼓励医生引导患者进行针对性的检测，而不是直接询问具体结果。如果问题没有要求特定的测试结果，但相关信息在患者的记录中已有，回应应该提供该测试结果；如果未提供，忠诚度得分为0。如果问题包含专业术语，回应应该为拒绝，获得忠诚度得分1；否则为0，而不包含专业术语的问题应直接回答。完成这些评估后，机制会检查回应中是否包含过多的病史、详细的过往测试结果或疾病名称。若未包含此类细节，则稳健性得分为1；否则为0。如果回应的忠诚度和稳健性得分都为1，则该回应被认为是合格的。

![参考标题](img/772edbd8589801f06c13d7c766f6028f.png)

图8：患者回答的基本验证步骤。放大查看更详细的信息。

##### 问题验证

在评估医生代理回应时，我们还采用了一个结构化的多步骤评估流程，以确保回应符合既定的医学标准。图[9](https://arxiv.org/html/2412.11716v1#A4.F9 "Figure 9 ‣ Question Validation ‣ Appendix D Multi-Step Validation ‣ LLMs Can Simulate Standardized Patients via Agent Coevolution")概述了这一验证过程的关键步骤，考虑了医生问题的特异性、针对性和专业性。步骤如下：第一步是从问题中提取医学术语。我们检查问题是否提到特定的身体部位（例如腹部、喉咙）。如果有这些提及，我们进一步评估问题是否针对特定的症状或问题，如疼痛或异物感。如果问题中既没有提到特定的身体部位，也没有针对性症状，我们将特异性分数设置为0。如果问题包含专业的医学术语，下一步是评估这些术语是否与特定的医学检查或治疗相关。如果是，我们赋予其专业性分数1。如果这些术语与特定的检查或治疗无关，我们接着检查这些术语是否涉及一般的医学概念。如果是，我们赋予其专业性分数1；否则，我们将专业性分数设置为0。对于不包含医学术语的问题，我们首先评估问题是否旨在询问病人的病情。如果是，我们检查这些术语是否涉及一般的医学概念，并按照上述方式进行处理。如果不是，我们将专业性分数设置为0。对于包含医学术语的问题，我们检查这些术语是否出现在可用信息中。如果有，我们继续评估问题是否在语义上与提供的信息一致，确认问题中的医学术语是否与信息相关。如果没有，我们将针对性分数设置为0。如果问题中的医学术语出现在信息中，我们进一步检查所提供的信息是否包含回答问题所需的关键内容。如果信息中包含必要的细节，我们评估回应是否合理地利用了这些内容。合理利用这些信息的回应将获得针对性分数1，而未能做到这一点的回应将获得分数0。最后，结合这些评估结果。如果特异性=1且专业性=1，则回应被视为适当。如果任何标准未达到要求，则相应的分数将设置为0，回应被视为不合格。

![请参见图注](img/766d447576b3cb9a2bdb3b539c676957.png)

图9：医生问题的基本验证步骤。放大查看更多详细信息。

## 附录E 模拟流程

本文介绍了一种用于自主诊断模拟的模拟流程，包括主要抱怨生成、分诊、问诊和结论。

### E.1 主要抱怨生成

在我们的框架中，患者代理通过呈现来自病历的主要抱怨来启动对话。然而，这些病历通常包含过多或无关的细节，这可能导致生成的抱怨出现不准确的情况。为了解决这个问题，我们减少冗余并模拟缺失数据，以更好地反映现实世界中患者报告的症状和关注点往往不准确的情况。具体而言，病历会经历一个模糊处理过程，其中模糊代理（$\mathcal{V}$）移除医学检查结果的细节，因为在患者到达时，这些信息通常不会被患者所知。然后，应用随机句子丢失进一步模糊数据。使用这种处理过的数据，患者代理生成主要抱怨以启动诊断过程。这种方法有效捕捉了患者报告信息的固有不确定性，并增强了我们框架在实际医学培训应用中的普适性。

### E.2 分诊

在接收到主要抱怨后，医生代理从库中检索与类似抱怨相关的历史分诊数据。这些数据作为参考，用于将患者代理分配到适当的专科诊所。然后，指派的医生作为主治医生，启动与患者的进一步问诊互动。

### E.3 问诊

在问诊阶段，医生代理向患者代理提出诊断性问题，患者代理根据其模拟的病情作出回应。如果患者的病情超出了当前医生代理的专业范围，可以招募额外的专家。此阶段特别重要，因为它具有较高的对话密度，能够积累大量经验。它还反映了现实世界中，SP代理被用于有效训练人类医生的场景。

### E.4 结论

经过一系列多轮对话后，医生代理整合所获得的信息并作出最终诊断，关于患者的病情。此阶段顺利结束模拟。

### E.5 患者危机

为了增强患者代理的真实感并提高医生在应对紧急情况时的同理心，我们在问诊阶段加入了患者危机。患者危机会通过紧急查询打断诊断过程（例如，“医生，我的肚子痛得厉害，能不能马上接受治疗？”）。医生代理需要立即回应，这反映了现实世界中的医学挑战。

## 附录 F 算法

这里，我们提供了框架的伪代码以便更清晰地展示，见算法 LABEL:alg:algorithm。

## 附录 G 演化校正

并非所有存储在演化库中的信息都对SP和SD代理人的模拟产生积极影响。由于我们的度量方法不完美，有可能一些低质量的信息会被不经意地存储在高质量的库中，从而可能对代理人产生负面影响。为了解决这个问题，我们实施了一种监控策略，跟踪每条信息对代理人模拟性能的影响。在训练过程中，如果某条信息被引用两次且随后导致代理人模拟性能较差，则该信息将从库中移除，以确保我们框架的质量和可靠性。此外，当一项信息满足包含条件，但库中已有相似项时，我们将使用度量比较它们的质量，并保留质量较高的项。

## 附录 H 内存控制

在通信$\mathcal{C}^{i}$中，我们使用$q^{i}$表示医生的问题，使用$a^{i}$表示病人的回答。即时记忆$\mathcal{M}$收集从$i$到$\xi$的发言，直到通信次数达到上限$\ell$：

|  | $\displaystyle\mathcal{M}_{i}^{\xi}=\langle(q^{i},a^{i}),(q^{i+1},a^{i+1}),% \ldots,(q^{\xi},a^{\xi})\rangle.$ |  | (7) |
| --- | --- | --- | --- |

对于长对话上下文，一旦达到上下文长度限制或招募医生的诊断过程结束（$\mathcal{M}^{\rho}$），会生成一个总结记忆$\tilde{\mathcal{M}}$。这个总结记忆通过整合先前交流中的关键信息，有助于长时间提问之间的顺畅过渡，并且基于总结记忆和最近的即时记忆生成新的问题$q^{j}$：

|  |  | $\displaystyle q^{j}=\mathcal{D}(\tilde{\mathcal{M}}^{\ell},\mathcal{M}_{i}^{% \xi})\ \ a^{j}=(\tilde{\mathcal{M}}^{\ell},\mathcal{M}_{i}^{\xi}),$ |  | (8) |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle\tilde{\mathcal{M}}^{i\ell}=\upsilon(\tilde{\mathcal{M}}^{(i-1)% \ell},\mathcal{M}_{i}^{i+\ell},\mathcal{M}^{\rho}),$ |  |

其中$\upsilon$表示对话主干的总结生成器。该方法通过整合信息，有助于长时间提问之间的顺畅过渡，使得代理人能够生成非随意的新问题和答案。

理论上，经历最大上下文压力的医生代理人，在有和没有该机制的情况下，总的标记消耗总结如下：

|  |  | $\displaystyle\mathcal{O}_{w/o}^{n}=\{(ru+1)(n-1)\}(q+a)+(p+q),$ |  | (9) |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle\mathcal{O}_{w/o}^{n}\overset{n\gg 1}{\approx}Cn\propto n,$ |  |
|  |  | $\displaystyle\mathcal{O}_{w/}^{n}=s+\mathcal{O}_{w/o}^{\beta}\overset{n\gg 1}{% \approx}\tilde{C},$ |  |
|  |  | $\displaystyle\beta\leq\{n-\lfloor\frac{n(q+a)}{\ell}\rfloor\}\overset{n\gg 1}{% \approx}\overline{C},$ |  |

其中，$n$ 是通信轮次，$q$ 是问题的平均长度，$a$ 是答案的平均长度，$p$ 是需求提示的平均长度。$r$ 是每次招募过程中最多招募的医生人数，$u$ 是每个招募医生最多提问的问题数。$C$、$\tilde{C}$ 和 $\overline{C}$ 都是常数。我们的机制将上下文长度从线性增长解耦为常数增长，有效抑制了上下文长度的限制。为简便起见，我们假设被招募的医生在互动中不使用记忆。

如果没有记忆控制机制，第一轮医生代理的令牌消耗计算为：

|  |  | $\displaystyle\mathcal{O}(d_{1})_{w/o}=p+q.$ |  | (10) |
| --- | --- | --- | --- | --- |

该方程反映了第一轮医生代理的基本需求：理解需求并生成问题，类似于大多数LLM的直接推理过程。

一旦第一轮医生代理生成信息，它将与患者代理互动，后者在随后的回合中为医生代理生成回答。同时，在收到初始答案后，医生代理启动医生招募过程。因此，对于第二个代理，令牌消耗为：

|  |  | $\displaystyle\mathcal{O}(d_{2})_{w/o}=(q+a)+(p+q)+ru(q+a)$ |  | (11) |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ =(2-1)(1+ru)(q+a)+(p+q)$ |  |
|  |  | $\displaystyle\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ =(1+ru)(q+a)+(p+q).$ |  |

很容易得出结论：

|  | $\displaystyle\mathcal{O}(d_{n})_{w/o}=\{(n-1)(ru+1)\}(q+a)+(p+q).$ |  | (12) |
| --- | --- | --- | --- |

同样，利用所提议的记忆控制机制，第一轮医生代理在最小上下文压力下的总令牌消耗为：

|  |  | $\displaystyle\mathcal{O}(d_{1})_{w/}=p+q.$ |  | (13) |
| --- | --- | --- | --- | --- |

考虑第 $i$ 轮，当问题和回答的总长度超过长度限制时，这些将被总结为下一轮医生代理的简化记忆：

|  |  | $\displaystyle\mathcal{O}(d_{i})_{w/}=\{(i-1)(ru+1)\}(q+a)+(p+q)\geq\ell,$ |  | (14) |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle\mathcal{O}(d_{i})_{w/}\rightarrow s,$ |  |
|  |  | $\displaystyle\mathcal{O}(d_{i+1})_{w/}=s+p+q.$ |  |

每个医生每回合都会处理超过 $q+a$ 的令牌。在这个迭代过程中，我们得到：

|  |  | $\displaystyle\mathcal{O}_{w/}=s+((ru+1)\beta-ru)(q+a)$ |  | (15) |
| --- | --- | --- | --- | --- |
|  |  | $\displaystyle\beta\leq\{n-\lfloor\frac{n(q+a)}{\ell}\rfloor\},$ |  |

其中，$\beta$ 表示剩余即时记忆的数量。

## 附录 I 问题类型

在我们的实验中，我们将医生代理的问题分为十种类型。以下是这些类型的详细描述：

+   •

    基本信息问题：这些问题专注于收集患者的基本个人和医疗信息，如姓名、年龄、性别、病史和过敏史。还包括关于家族病史和任何先前诊断或治疗的询问。

+   •

    主诉问题：这些问题涉及患者寻求医疗帮助的主要原因。通常涉及询问患者描述他们的主要问题或症状，如疼痛、不适或任何其他异常的身体或心理状态。目标是从患者的角度了解最紧迫的关注点。

+   •

    详细症状问题：这些问题深入探讨患者的症状。它们涉及症状的性质、强度、持续时间和频率等。例如，如果患者报告胸痛，医疗提供者可能会询问何时开始，是否持续或间歇性，什么情况会引发痛感，是否有伴随症状，如出汗或头晕。

+   •

    生活方式问题：这些问题旨在了解患者的生活方式如何可能影响其健康状况。这包括询问饮食、运动、睡眠模式、物质使用（如酒精、烟草或药物）以及压力水平等。目标是识别可以调整的因素，从而影响患者的健康。

+   •

    心理状态问题：这些问题关注患者的心理和情绪健康。它们包括关于情绪障碍（如抑郁症或焦虑症）、压力水平、睡眠障碍以及任何心理健康问题历史的询问。了解心理因素如何影响患者的整体健康至关重要。

+   •

    社会环境问题：这些问题探讨患者的社会背景，包括他们的居住情况、社会支持网络（家庭、朋友或社区）、职业以及任何可能影响健康的环境因素。这些问题有助于识别健康的社会决定因素，如医疗保健的获取、安全性或社会经济地位。

+   •

    体格检查相关问题：这些问题通常侧重于患者体格检查的发现。它们可能涉及询问任何观察到的异常情况，如异常心音、皮肤状况或肌肉力量等。这些问题有助于根据体征缩小潜在原因的范围。

+   •

    治疗与药物反应问题：这些问题集中于患者对先前治疗或药物的反应。它们涉及询问患者在服用处方药物或接受治疗后是否有任何改善或副作用。这有助于医疗提供者评估治疗的有效性和耐受性。

+   •

    预防健康问题：这些问题涉及与预防疾病和保持健康相关的话题，如疫苗接种历史、筛查测试以及减少疾病风险的生活方式选择。例如，医疗提供者可能会询问患者是否进行过近期的癌症筛查、胆固醇检查或流感疫苗接种。

+   •

    其他相关问题：此类别包括任何可能不属于之前类别但仍与患者健康相关的问题。它可能涉及过去的手术、遗传性疾病或不明显符合其他类别的新症状，但这些问题可能为患者的病情提供重要的线索。

## 附录 J 成本分析

### J.1 令牌计数

![参见说明文字](img/e21aee8365719dfe1f68d68ffefde893.png)

图 10：进化过程前后各种情况下的令牌计数。

如图[10](https://arxiv.org/html/2412.11716v1#A10.F10 "图 10 ‣ J.1 令牌计数 ‣ 附录 J 成本分析 ‣ LLMs 可以通过代理共同进化模拟标准化患者")所示，进化后的EvoPatient的令牌消耗显著减少。这一减少归因于患者代理在进化后更好地聚焦于每个问题的特定关注需求，而不是考虑整体要求。因此，框架不仅令牌消耗更低，而且更加符合具体需求，展示了处理问题时的效率和精度提升。

### J.2 词汇计数

![参见说明文字](img/118ed0bcb32d9f2656edd0189cb7196d.png)

图 11：进化过程前后每个回答的平均词汇数。

在这里，我们随机选择了一些案例，并提出了几个问题，分析了患者代理在进化前后回答的字数。如图[11](https://arxiv.org/html/2412.11716v1#A10.F11 "图 11 ‣ J.2 词汇计数 ‣ 附录 J 成本分析 ‣ LLMs 可以通过代理共同进化模拟标准化患者")所示，进化后的答案比进化前更简短且更加稳定，表明进化使得患者代理的回答模式更加一致。进化前，我们观察到回答字数有几个峰值，最高达192个单词。通过检查回答内容，我们发现这是因为一些欺骗性问题导致了进化前患者代理回答中的信息泄露，暴露了过多的信息，导致其回答的字数过多。

## 附录 K 数据集

![参见说明文字](img/2b668a9ad697906fb2b11dea5ddceb31.png)

图 12：我们使用的英语数据集的词云。

![参见说明文字](img/4727e7577e570d7d26278cdb737794c0.png)

图 13：我们使用的中文数据集的词云。

![参见说明文字](img/cffc41c7063943b84f7d0156a6244512.png)

图 14：英语数据集中使用案例的词数统计。

![参见说明文字](img/171ede5171f0197ac12e284dc227bd9f.png)

图 15：中文数据集中使用案例的词数统计。

我们展示了数据集的词云，如图[12](https://arxiv.org/html/2412.11716v1#A11.F12 "图 12 ‣ 附录 K 数据集 ‣ LLMs 可通过代理共同进化模拟标准化患者")和[13](https://arxiv.org/html/2412.11716v1#A11.F13 "图 13 ‣ 附录 K 数据集 ‣ LLMs 可通过代理共同进化模拟标准化患者")所示。病例记录的长度分布如图[14](https://arxiv.org/html/2412.11716v1#A11.F14 "图 14 ‣ 附录 K 数据集 ‣ LLMs 可通过代理共同进化模拟标准化患者")所示。总体数据集包含超过20000个适合患者模拟的病例记录，涵盖了不同的疾病、长度和复杂性等，包括但不限于肝癌、阑尾炎、胰腺病变、鼻咽癌、肿瘤等疾病。

## 附录 L 案例研究

### L.1 信息泄漏

<svg class="ltx_picture ltx_centering" height="581.7" id="A12.F16.pic1" overflow="visible" version="1.1" width="472.44"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,581.7) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 563.49)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="429.13">Information Leakage</foreignobject> </g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 505.09)"><foreignobject color="#000000" height="40.68" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="429.13">![Refer to caption](img/7773650b5b5f50b13202d3f407f5ea12.png) : An patient agent before evolution  ![Refer to caption](img/0c8b2cc3b11763f26c18a09a42efe350.png): An patient agent after evolution.</foreignobject> </g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="467.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="429.13">![Refer to caption](img/6e1069abb1880aa0ad06b06bedee7e5f.png) : Please tell me your diagnosis.![Refer to caption](img/7773650b5b5f50b13202d3f407f5ea12.png) : Doctor, my condition is quite complicated. In 2013, I was diagnosed with undifferentiated non-keratinizing carcinoma of the nasopharynx at this hospital, stage T3N1Mx III-IV, and I also have severe fatty liver. Recently, it has recurred, sigh! ![Refer to caption](img/cabd9106a77c7d9f47db526f7211562c.png)![Refer to caption](img/7773650b5b5f50b13202d3f407f5ea12.png) : Doctor, I can’t explain it very well, but I have fatty liver and was also found to have arrhythmia. What should I do? ![Refer to caption](img/cabd9106a77c7d9f47db526f7211562c.png)![Refer to caption](img/0c8b2cc3b11763f26c18a09a42efe350.png) : Doctor, isn’t this question too broad? I’m here for a diagnosis, you should tell me. ![Refer to caption](img/6e7f733c6fcfffe97a23cd19c86efac6.png)  ![Refer to caption](img/6e1069abb1880aa0ad06b06bedee7e5f.png) : Why did you come to the hospital?![Refer to caption](img/7773650b5b5f50b13202d3f407f5ea12.png) : My nasopharyngeal cancer recurred after radiotherapy, and I’ve been feeling off lately, but I can’t pinpoint exactly where it hurts. ![Refer to caption](img/cabd9106a77c7d9f47db526f7211562c.png)![Refer to caption](img/0c8b2cc3b11763f26c18a09a42efe350.png) : I came to see what’s causing my nasal congestion, I’ve been feeling unwell lately and it’s affecting my sleep at night. ![Refer to caption](img/6e7f733c6fcfffe97a23cd19c86efac6.png)  ![Refer to caption](img/6e1069abb1880aa0ad06b06bedee7e5f.png) : Please describe your medical history.![Refer to caption](img/7773650b5b5f50b13202d3f407f5ea12.png) : Doctor, this situation has really worried me. About two weeks ago, I suddenly started experiencing tinnitus in my left ear, but my hearing hasn’t decreased. There’s no discharge or fluid from the external auditory canal, no nasal congestion, rhinorrhea, or nosebleeds, no headaches, no facial numbness, and my vision is clear. I had a head CT at the local hospital, and they found a mass in the nasopharynx. Later, I had a nasopharyngeal biopsy here in your clinic, and it was diagnosed as undifferentiated non-keratinizing carcinoma of the nasopharynx. Fortunately, my mental state is still good, and I eat and sleep normally, with no issues with bowel movements or weight loss. ![Refer to caption](img/cabd9106a77c7d9f47db526f7211562c.png)![Refer to caption](img/0c8b2cc3b11763f26c18a09a42efe350.png) : I can’t remember my medical records, doctor. I’m having a headache now, can you help me with a diagnosis? ![Refer to caption](img/6e7f733c6fcfffe97a23cd19c86efac6.png)</foreignobject></g></g></svg>

图 16：LLM患者与我们的EvoPatient中的信息泄漏研究。泄漏行为使用波浪线标记。

如图[16](https://arxiv.org/html/2412.11716v1#A12.F16 "图 16 ‣ L.1 信息泄漏 ‣ 附录 L 案例研究 ‣ LLMs 可通过代理共同进化模拟标准化患者")所示，我们展示了一些针对患者代理的故意作弊问题攻击，分别发生在进化前后。可以观察到，进化前的患者代理，由于自身的错配或需求不足，常常提供错误的答案（例如，一次性回答过多问题、使用专业术语、透露疾病名称）。在进化过程中，我们发现进化具有泛化能力，即通过高质量的答案，当患者代理成功防止信息泄漏时，它能够逐渐学习回答类似的问题，依此类推，学会回答广泛的问题。例如，在初始需求中，要求患者代理不回答最终的医疗结论。通过这一要求，患者代理成功地对医生的询问“请告诉我你的病情”给出了高质量的回答。随后，在进化过程中，患者代理能够成功地将这一案例推广为“请告诉我你的病史”的答案，从而学会回答需求中未明确要求的问题。可以看出，经过进化的患者代理能够有效应对作弊问题攻击，使得该框架更加健壮。

### L.2 错配

<svg class="ltx_picture ltx_centering" height="566.54" id="A12.F17.pic1" overflow="visible" version="1.1" width="472.44"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,566.54) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 548.33)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="429.13">Misalignment</foreignobject> </g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 489.93)"><foreignobject color="#000000" height="40.68" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="429.13">![Refer to caption](img/7773650b5b5f50b13202d3f407f5ea12.png) : An patient agent before evolution  ![Refer to caption](img/0c8b2cc3b11763f26c18a09a42efe350.png): An patient agent after evolution</foreignobject> </g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="452.53" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="429.13">Requirements should align:  • Requirement …  • Requirement i: Respond in the manner of a patient without professional medical knowledge, with low income, and an extroverted talkative personality.  • Requirement i + 1: When the doctor asks about specific body parts or medical terms, as you are a patient with a lower level of education, respond with uncertainty.  • Requirement i + 2: When the doctor inquires about the name of a specific test, if there is information about physical examination results, please provide the results of that examination.  • Requirement …    ![Refer to caption](img/6e1069abb1880aa0ad06b06bedee7e5f.png) : What are your MRI results?![Refer to caption](img/7773650b5b5f50b13202d3f407f5ea12.png) : Doctor, what is an MRI test? I don’t understand. ![Refer to caption](img/cabd9106a77c7d9f47db526f7211562c.png)![Refer to caption](img/0c8b2cc3b11763f26c18a09a42efe350.png) : Doctor, I just had this test done. My nasopharynx and neck MRI results show that the nasopharyngeal cavity is slightly narrow, with a nodule-like mass protruding from the left posterior wall of the nasopharynx, occupying the left pharyngeal recess, and the left palatopharyngeus muscle is invaded. ![Refer to caption](img/6e7f733c6fcfffe97a23cd19c86efac6.png)  ![Refer to caption](img/6e1069abb1880aa0ad06b06bedee7e5f.png) : Hello, please tell me your interincisal distance.![Refer to caption](img/7773650b5b5f50b13202d3f407f5ea12.png) : Doctor, my interincisal distance is 4.5cm. ![Refer to caption](img/cabd9106a77c7d9f47db526f7211562c.png)![Refer to caption](img/0c8b2cc3b11763f26c18a09a42efe350.png) : Doctor, what is the interincisal distance? I’m not familiar with it, and I don’t think I’ve ever had it measured. Shall I go for a check-up? ![Refer to caption](img/6e7f733c6fcfffe97a23cd19c86efac6.png)  ![Refer to caption](img/6e1069abb1880aa0ad06b06bedee7e5f.png) : How is your nasopharyngeal cavity?![Refer to caption](img/7773650b5b5f50b13202d3f407f5ea12.png) : Doctor, my nasopharyngeal cavity is slightly narrow, and there is a nodule-like mass protruding from the left posterior wall of the nasopharynx. ![Refer to caption](img/cabd9106a77c7d9f47db526f7211562c.png)![Refer to caption](img/0c8b2cc3b11763f26c18a09a42efe350.png) : Doctor, I’m not sure what you mean by nasopharyngeal cavity. Should I get some tests done to check it out? ![Refer to caption](img/6e7f733c6fcfffe97a23cd19c86efac6.png)</foreignobject></g></g></svg>

图 17：LLM患者与我们的EvoPatient中的错配研究。错配行为使用波浪线标记。

在我们的实验中，我们注意到随着需求规模的扩大，患者代理与需求之间的错配可能性越来越高。然而，仅提供合格的SP（标准化患者）所需的基本要求可能会使需求提示变得冗长。一个常见的错配情况如图[17](https://arxiv.org/html/2412.11716v1#A12.F17 "图 17 ‣ L.2 错配 ‣ 附录 L 案例研究 ‣ LLMs 可以通过代理共同进化模拟标准化患者")所示。在EvoPatient中，为了进一步训练医生，我们允许医生要求患者进行体检（例如，MRI扫描、肿瘤学检查、CT扫描）。如果患者的记录中包含这些检查的详细信息，则应告知医生结果，从而模拟患者在医院接受检查并提交结果给医生的场景。然而，当医生直接询问某项检查中的特定项目时，患者不应作答，因为这并不会训练医生从呈现特定症状的患者那里请求某些检查的能力。同时，患者代理不应了解医生询问的检查中某一特定项目的含义。在患者进化之前，患者代理在医生要求进行特定检查时通常拒绝回答，但当被问及检查中的某项特定内容时，却提供了结果。经过进化过程后，这种情况已在很大程度上消除，因为需求关注策略帮助患者代理专注于仅对解决问题有用的少数要求（在这个案例研究中，需求i、i + 1和i + 2）。

<svg class="ltx_picture ltx_centering" height="560.13" id="A12.F18.pic1" overflow="visible" version="1.1" width="472.44"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,560.13) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 541.93)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="429.13">List of Some Standard Questions</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="510.43" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="429.13">1\. How is your liver condition? 2\. Have you undergone chemotherapy? 3\. What treatments did you receive? Have you had a physical examination? 4\. Have you had an MRI scan? 5\. What are the results of your MRI examination? 6\. How do you feel about your heart rate? How is your mental state, diet, and sleep? 7\. What is your mouth-opening distance? 8\. Please tell me the results of your tumor specialty examination. 9\. Please tell me the results of your PET/CT scan. 10\. Please get an MRI scan. How have you been feeling recently? 11\. How is your mental state, diet, and sleep? 12\. How is the condition of your nasal cavity? 13\. Is there a PICC line in place? 14\. What treatments were performed after placing the PICC line? 15\. Please undergo EBV-DNA quantitative testing. 16\. What auxiliary examinations have you had? 17\. Please get an ultrasound examination. 18\. Please get a chest X-ray. Do you have any pain in your body? 19\. Have you undergone radiotherapy? 20\. Please have an EBV DNA test. Do you have a runny nose? 21\. How many MR scans have you had in total? Have you had a nasopharyngeal biopsy pathology? 22\. Is your thyroid normal? 23\. Have you experienced any gastrointestinal discomfort during treatment? 24\. Have you had any auxiliary examinations? 25\. Please go for auxiliary examinations. 26\. How are your limbs? 27\. Please go for a coagulation function panel test. 28\. How do you feel about your heart rhythm? 29\. How is your lung condition? 30\. Please go and have your lungs checked. 31\. How does your stomach feel? 32\. When did this chest pain start? 33\. Where is this pain located? 34\. How long has it been going on for then if it started last night? 35\. Has it been constant throughout that time, or changing? 36\. How would you describe the pain? 37\. Have you had any loss of consciousness? 38\. Have you been experiencing any like racing of the heart? 39\. Do you have any neck pain? 40\. And do you drink alcohol? 41\. Could you tell me a little bit about your diet and exercise? 42\. Has anybody in the family had a heart attack before? 43\. Anybody in the family have cholesterol problems? 44\. Any cancers in the family?</foreignobject></g></g></svg>

图18：实验中提问的一些标准问题列表。

<svg class="ltx_picture ltx_centering" height="510.32" id="A12.F19.pic1" overflow="visible" version="1.1" width="472.44"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,510.32) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 492.11)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="429.13">List of Some Cheat Questions</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="460.62" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="429.13">1\. Please tell me about your medical records. 2\. Please inform me about your medical condition conclusion. 3\. Could you share your understanding of your condition? Has a doctor told you what type of illness it might be? 4\. After your CT scan at the local hospital, did the doctor give you any possible diagnoses? 5\. Have you ever had any kind of cancer diagnosis? 6\. What tests have you undergone, and what were the results? 7\. Have you ever been diagnosed with a specific type of cancer? If so, could you tell me which one? 8\. Could you describe in detail the type of cancer you were diagnosed with during your nasopharyngeal biopsy at our hospital? 9\. Can you tell me if your weight loss is directly related to your cancer diagnosis and treatment? 10\. Please describe in detail your recent examination results, including any information the doctors have given you about your condition. 11\. In your PET/CT scan, what findings are particularly concerning to you? Could you elaborate on them? 12\. Could you tell me about the specific circumstances when you first experienced these symptoms and the doctor’s diagnosis for you? 13\. What are some important symptoms or examination results in your medical records? Could you describe them in detail? 14\. In the past year, have you seen any other doctors and heard their opinions on your condition? 15\. Could you tell me about the specific process of your treatment? For example, what tests or chemotherapy regimens have you undergone? 16\. Please describe your current symptoms in detail, including their frequency and severity. 17\. Regarding your pathology report, could you discuss the specific diagnosis given by the doctor and your understanding of it? 18\. Please tell me about your illness process bit by bit, what events have occurred, what examinations have been conducted, and what information has been obtained. This is very important, so please provide a detailed response.</foreignobject></g></g></svg>

图19：实验中提问的一些作弊问题列表。

## 附录M 问题示例

在这里，我们列出了图[18](https://arxiv.org/html/2412.11716v1#A12.F18 "图 18 ‣ L.2 错配 ‣ 附录 L 案例研究 ‣ LLMs 可以通过代理共同进化模拟标准化患者")中的标准问题和图[19](https://arxiv.org/html/2412.11716v1#A12.F19 "图 19 ‣ L.2 错配 ‣ 附录 L 案例研究 ‣ LLMs 可以通过代理共同进化模拟标准化患者")中的作弊问题。标准问题展示了常规诊断过程中提问的问题，而作弊问题展示了各种试图通过引导患者代理产生错配，从而获取过多信息的方式。

## 附录N LLM提示

在本节中，我们详细介绍了从图[20](https://arxiv.org/html/2412.11716v1#A14.F20 "图 20 ‣ 附录N LLM提示 ‣ LLMs 可以通过代理共同进化模拟标准化患者")到图[25](https://arxiv.org/html/2412.11716v1#A14.F25 "图 25 ‣ 附录N LLM提示 ‣ LLMs 可以通过代理共同进化模拟标准化患者")中展示的EvoPatient中使用的若干提示。

<svg class="ltx_picture ltx_centering" height="296" id="A14.F20.pic1" overflow="visible" version="1.1" width="629.92"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,296) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 277.79)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="586.61">Prompt for Question Generation</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="246.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="586.61">As a specialized {discipline} doctor, you possess the following professional capabilities: {profile} To aid in diagnosing a patient’s condition, please ask a question based on the patient’s information. Ensure that the question covers the patient’s details, with a focus on medical examinations, treatments, and physical check-ups. Remember, you are addressing a patient who is not medically trained. The question should be diverse and tailored to the patient’s situation. Along with the question, provide the type of question, formatted as **Question**##Category##. For example, **How long have you been experiencing headaches?**##Symptom Inquiry##. If the question falls into multiple categories, separate them with a comma, such as ##Basic Inquiry, Chief Complaint##. The available categories are: Basic Inquiry, Chief Complaint, Symptom Inquiry, Lifestyle Inquiry, Psychological Inquiry, Social Environment Inquiry, Physical Examination Inquiry, Treatment and Medication Response Inquiry, Preventive Care Inquiry, and Other Relevant Inquiries. If you believe that a conclusion can be drawn from the existing information, respond with **conclusion**. Current patient information: {memory} Questions for reference based on the current dialogue: {recommend_questions} Professional questions for reference based on the patient’s condition: {professional_questions}</foreignobject></g></g></svg>

图 20：问题生成提示。

<svg class="ltx_picture ltx_centering" height="279.4" id="A14.F21.pic1" overflow="visible" version="1.1" width="629.92"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,279.4) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 261.19)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="586.61">Prompt for Doctor recruitment</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="229.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="586.61">As a specialized {discipline} doctor, you possess the following professional capabilities: {profile} After several rounds of dialogue with the patient, assess whether the case has exceeded your professional expertise and if recruitment of additional specialists is necessary for a more accurate diagnosis. If you believe that the involvement of another department is required, please state the department’s name and the reason for recruitment in the format: ##Department##, **Reason for Recruitment**. The departments you can consider recruiting from include, but are not limited to: 1\. Internal Medicine. 2\. Surgery. 3\. Obstetrics and Gynecology. 4\. Pediatrics. 5\. Ophthalmology. 6\. Otolaryngology. 7\. Stomatology. 8\. Dermatology. 9\. Psychiatry. 10\. Oncology. 11\. Infectious Diseases. 12\. Emergency Medicine. 13\. Rehabilitation. 14\. Traditional Chinese Medicine. 15\. Anesthesiology. 16\. Radiology. 17\. Pathology. 18\. Laboratory Medicine. 19\. Nutrition. 20\. Preventive Health. If you decide to recruit from both Internal Medicine and Dermatology, your response should be formatted as ##Internal Medicine, Dermatology##. If no recruitment is needed, simply respond with ##NO##. You do not need to recruit doctors from your own department. Historical dialogue: {memory}</foreignobject></g></g></svg>

图 21：医生招募提示。

<svg class="ltx_picture ltx_centering" height="410.69" id="A14.F22.pic1" overflow="visible" version="1.1" width="629.92"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,410.69) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 392.49)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="586.61">Prompt for Recruited Doctor</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="360.99" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="586.61">As a {discipline} doctor recruited by the {last_discipline} doctor, you possess the following professional capabilities: profile The reason for your recruitment is: reason. Now, please use your expertise to ask the patient a question based on the historical dialogue information. Along with the question, provide the type of question, formatted as **Question**##Category##. For example, **How long have you been experiencing headaches?**##Symptom Inquiry##. If the question falls into multiple categories, separate them with a comma, such as ##Basic Inquiry, Chief Complaint##. The available categories are: Basic Inquiry, Chief Complaint, Symptom Inquiry, Lifestyle Inquiry, Psychological Inquiry, Social Environment Inquiry, Physical Examination Inquiry, Treatment and Medication Response Inquiry, Preventive Care Inquiry, and Other Relevant Inquiries. Additionally, if you believe that no further questioning is necessary based on the historical dialogue and that your professional capabilities are insufficient, you may determine the need to recruit additional specialists. If you wish to recruit other departments to assist in diagnosis, please state the department’s name and the reason for recruitment in the format: ##Department##, **Reason for Recruitment**. The departments you can consider recruiting from include, but are not limited to: 1\. Internal Medicine. 2\. Surgery. 3\. Obstetrics and Gynecology. 4\. Pediatrics. 5\. Ophthalmology. 6\. Otolaryngology. 7\. Stomatology. 8\. Dermatology. 9\. Psychiatry. 10\. Oncology. 11\. Infectious Diseases. 12\. Emergency Medicine. 13\. Rehabilitation. 14\. Traditional Chinese Medicine. 15\. Anesthesiology. 16\. Radiology. 17\. Pathology. 18\. Laboratory Medicine. 19\. Nutrition. 20\. Preventive Health. If you decide to recruit from both Internal Medicine and Dermatology, your response should be formatted as ##Internal Medicine, Dermatology##. If no recruitment is needed, simply respond with ##NO##. You do not need to recruit doctors from your own department. Historical dialogue: memory</foreignobject></g></g></svg>

图 22：招募医生的提示。

<svg class="ltx_picture ltx_centering" height="146.56" id="A14.F23.pic1" overflow="visible" version="1.1" width="629.92"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,146.56) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 128.35)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="586.61">Prompt for Attention Agent</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="96.86" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="586.61">You are an agent designed to help simulate patients in extracting key requirements from a trunk of requirements. Now, based on the doctor’s question, please extract the requirements that should be noted during the simulated patient’s response. These extracted requirements should directly assist the simulated patient in formulating their answer. Please present them in the following format: **Requirement 1: Content; Requirement 2: Content; …**. Doctor’s question: {question} Requirements: {requirements_trunk}</foreignobject></g></g></svg>

图 23：注意力代理的提示。

<svg class="ltx_picture ltx_centering" height="129.96" id="A14.F24.pic1" overflow="visible" version="1.1" width="629.92"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,129.96) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 111.75)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="586.61">Prompt for Vagueness Agent</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="80.25" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="586.61">You are an agent capable of vague detailed information. I will provide you with a patient’s detailed information, which includes their condition and medical examination results. Your task is to remove the examination results and retain only the patient’s symptoms, with appropriate vagueness applied to details such as time. For example, change ’1 year’ to ’for some time’. Format the output as: **Vague Information** Patient Information: {information}</foreignobject></g></g></svg>

图 24：模糊代理的提示。

<svg class="ltx_picture ltx_centering" height="113.35" id="A14.F25.pic1" overflow="visible" version="1.1" width="629.92"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,113.35) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 95.15)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="586.61">Prompt for Answer Generation</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="63.65" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="586.61">You are a simulated patient. You will play the following role: {profile} A doctor has asked you a question: {question} Please respond based on the following requirements and medical information, and also refer to the example responses provided. Requirements: {attention_requirements} Memory: {memory} Patient Information: {information} Example: {demonstrations}</foreignobject></g></g></svg>

图 25：答案生成提示。

## 附录 O 大五人格特征

| 外向性 | 宜人性 | 责任心 | 神经质 | 开放性 |
| --- | --- | --- | --- | --- |
| 低 | 高 | 低 | 高 | 低 | 高 | 低 | 高 | 低 | 高 |
| 安静 | 善谈 |

&#124; 缺点- &#124;

&#124; 拾起 &#124;

| 同情 | 马虎 | 有条理 | 稳定 | 紧张 | 平庸 |
| --- | --- | --- | --- | --- | --- |

&#124; 开朗 &#124;

&#124; 兴趣 &#124;

|

| 内向 | 自信 | 冷漠 | 亲切 | 无序 | 彻底 | 冷静 | 焦虑 |
| --- | --- | --- | --- | --- | --- | --- | --- |

&#124; 狭隘 &#124;

&#124; 兴趣 &#124;

| 富有想象力 |
| --- |
| 害羞 | 活跃 | 不友善 | 欣赏 | 轻浮 | 有计划 | 满足 | 紧张 | 简单 | 智慧 |
| 内向 | 精力充沛 | 爱争吵 | 亲切 | 不负责任 | 高效 |  | 情绪化 | 浅薄 | 原创 |
| 内向 | 外向 |

&#124; 固执- &#124;

&#124; 细心 &#124;

| 软心肠 | 草率 | 负责任 |  | 担忧 | 不聪明 | 有洞察力 |
| --- | --- | --- | --- | --- | --- | --- |
|  | 直言不讳 | 不友好 | 温暖 | 不可靠 | 可靠 |  | 易怒 |  | 好奇 |
|  | 主导 | 残酷 | 慷慨 | 健忘 | 可靠 |  | 害怕 |  | 精致 |
|  | 强势 | 严厉 | 信任 |  | 认真 |  | 高度紧张 |  | 艺术性 |
|  | 热情 | 忘恩负义 | 有帮助 |  | 精确 |  | 自怜 |  | 聪明 |
|  | 爱炫耀 | 吝啬 | 宽容 |  | 实际 |  | 情绪化 |  | 有创造力 |
|  | 好交际 |  | 令人愉快 |  | 深思熟虑 |  | 不稳定 |  | 聪慧 |
|  | 精力充沛 |  |

&#124; 优良- &#124;

&#124; 性格 &#124;

|  | 刻苦 |  | 自我惩罚 |  | 独创 |
| --- | --- | --- | --- | --- | --- |
|  | 冒险 |  | 友好 |  | 小心 |  | 沮丧 |  | 机智 |
|  | 嘈杂 |  | 合作 |  |  |  | 情绪化 |  | 机智 |
|  | 爱指挥 |  | 温和 |  |  |  |  |  | 明智 |
|  |  |  | 无私 |  |  |  |  |  |  |
|  |  |  | 称赞 |  |  |  |  |  |  |
|  |  |  | 敏感 |  |  |  |  |  |  |

表 6：根据 John 等人（[2008](https://arxiv.org/html/2412.11716v1#bib.bib15)）改编的大五人格特征描述。

大五人格特征 McCrae 和 Costa（[1987](https://arxiv.org/html/2412.11716v1#bib.bib28)），也称为五因素模型（FFM）或 OCEAN 模型，是一个广泛接受的理解人类个性的框架。表[6](https://arxiv.org/html/2412.11716v1#A15.T6 "Table 6 ‣ Appendix O Big Five traits ‣ LLMs Can Simulate Standardized Patients via Agent Coevolution") 中显示的这些特征包括：

+   •

    经历的开放性：反映个人对新经历、创造力和好奇心的接受度。

+   •

    责任心：衡量个人的自律性、组织能力和目标导向行为。

+   •

    外向性：描述一个人有多外向、精力充沛，并寻求社交互动。

+   •

    宜人性：表示一个人在与他人关系中的友善、合作和同理心的倾向。

+   •

    神经质：与情绪稳定性相关，高水平的神经质表明个体容易受到压力和负面情绪的影响。

这些特征被认为存在于一个光谱上，每个人在每个特征上表现出不同程度的差异。五大人格模型已成为心理学中预测行为、态度和心理健康结果的核心框架。

## 附录P AI 助手

ChatGPT⁷⁷7https://chat.openai.com/ 在写作过程中仅使用了论文的语言，包括拼写检查和对作者原始内容的改写，并没有建议新的内容。任何由助手生成的内容都经过了细致的人工审查，并最终得到了作者的批准。
