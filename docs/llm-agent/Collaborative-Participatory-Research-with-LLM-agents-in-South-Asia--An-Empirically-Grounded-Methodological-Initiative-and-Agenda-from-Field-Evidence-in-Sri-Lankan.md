<!--yml

类别：未分类

日期：2025-01-11 11:56:54

-->

# 基于LLM智能体的南亚合作性参与研究：来自斯里兰卡实地证据的基于实证的方法学倡议与议程

> 来源：[https://arxiv.org/html/2411.08294/](https://arxiv.org/html/2411.08294/)

赵新杰¹，沙曼·马杜兰加·斯里瓦纳辛赫¹，唐佳成⁴，

王世云²，王浩³，森森·森川¹

¹ 东京大学，² 哥本哈根大学，

³ 中国农业大学，⁴ 山东师范大学

###### 摘要

人工智能在发展研究方法中的融合，为解决参与式研究中的长期挑战，尤其是在南亚这样语言多样化的地区，提供了前所未有的机会。本文从斯里兰卡僧伽罗语社区的实证实施中汲取经验，提出了一个基于实证的研究方法框架，旨在转变参与式发展研究，特别是在斯里兰卡易受洪水影响的尼尔瓦拉河流域这一多语言挑战性的背景下。该框架超越了传统的翻译和数据收集工具，采用了多智能体系统架构，重新定义了数据收集、分析和社区参与在语言和文化多样的研究环境中的实施方式。这一结构化、基于智能体的方法使得参与式研究既具可扩展性又能响应需求，确保社区的观点始终是研究成果的重要组成部分。我们的实地经验揭示了基于大规模语言模型（LLM）系统在解决资源匮乏地区发展研究中的长期问题方面的巨大潜力，既提供了量化的效率，又改善了包容性的质量。在更广泛的方法论层面，本研究议程倡导采用AI驱动的参与式研究工具，注重道德考虑、文化尊重和操作效率，强调部署AI系统的战略路径，以增强社区主导性和公平的知识生成，可能为全球南方地区的更广泛研究议程提供启示。

## 1 引言

人工智能与发展研究的交汇预示着参与性方法论的变革性范式转变，特别是通过大语言模型（LLMs）的出现及其有潜力彻底改变社区参与实践，Mohamed 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib46)）；Skirgård 等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib68)）。随着这些技术的迅速发展，它们在发展研究中的应用既带来了前所未有的机会，也提出了复杂的方法论挑战，亟需深入审视，Roberts 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib58)）。这一交集在语言多样性强烈的地区，特别是在南亚，显得尤为重要，因为传统的研究方法长期以来难以弥合沟通鸿沟和文化差异，Kshetri（[2024](https://arxiv.org/html/2411.08294v1#bib.bib40)）；Hassan 等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib30)）。

传统参与性研究方法的局限性，过度依赖人工中介且受资源可用性的限制，历史上一直阻碍着发展举措的规模和效果，Göpferich 和 Jääskeläinen（[2009](https://arxiv.org/html/2411.08294v1#bib.bib26)）。这些限制在语言环境复杂且技术基础设施有限的地区尤其明显，Magueresse 等人（[2020](https://arxiv.org/html/2411.08294v1#bib.bib44)）；Nekoto 等人（[2020](https://arxiv.org/html/2411.08294v1#bib.bib48)）。然而，LLM架构的最新进展，特别是在少量学习和跨语言迁移能力方面，为这些长期存在的挑战提供了有前景的解决方案，Raiaan 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib53)）；Wu 等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib75)）。

基于大语言模型（LLM）系统的整合进入参与性研究框架，提出了关于社区参与性和知识民主化本质的基本问题，Hadi 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib28)）；Diab Idris 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib18)）。虽然这些技术提供了强大的工具，可以弥合语言和文化的鸿沟，但它们的部署必须小心策划，以增强而非削弱发展研究的参与性，Rane 等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib56)）；Kovač 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib39)）。这需要一种细致的平衡方法，既要考虑技术能力，又要注重伦理考量和社区的能动性，Sabarirajan 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib60)）；Ray（[2023](https://arxiv.org/html/2411.08294v1#bib.bib57)）。

本文介绍并测试了一个新颖的框架，旨在利用基于LLM的多智能体系统进行参与性发展研究，依据斯里兰卡僧伽罗语社区的实证证据（Hashmi等，[2024](https://arxiv.org/html/2411.08294v1#bib.bib29)）；Urwin等，[2023](https://arxiv.org/html/2411.08294v1#bib.bib72)）。我们的方法不仅限于简单的技术整合，还解决了全球南方背景下关于社区赋权和知识生产的根本问题（Pfeffer等，[2013](https://arxiv.org/html/2411.08294v1#bib.bib51)）。这项工作的紧迫性在于发展挑战的日益复杂，以及对可扩展、文化敏感的研究方法的日益需求（van Rensburg和van der Westhuizen，[2024](https://arxiv.org/html/2411.08294v1#bib.bib73)）；Awad等，[2016](https://arxiv.org/html/2411.08294v1#bib.bib6)）。通过对机会和挑战的批判性分析，我们展示了如何通过深思熟虑的AI技术部署，增强人类在发展研究中的能力，从而可能带来更具包容性和影响力的成果（Ferdaus等，[2024](https://arxiv.org/html/2411.08294v1#bib.bib21)）。我们的框架提供了一种实施基于LLM的多智能体系统的结构化方法，同时保持参与性研究的核心原则，为在技术与发展交叉领域工作的研究人员、实践者和政策制定者提供了有价值的见解。

本文通过提供一个结构化框架，讨论了人工智能在发展研究中的作用，特别是在参与性研究背景下实施基于大语言模型（LLM）的多智能体系统。我们认为，当这些技术得到深思熟虑的部署时，它们能够增强而非取代人类在发展研究中的能力，从而可能带来更具包容性、更高效且更有影响力的研究成果。

## 2 为什么南亚现在需要这一点

南亚正处于一个变革性的交汇点，在这个交汇点上，加速的数字化进程与深深植根于社会、文化和语言环境中的因素相交织，为参与式发展研究提供了前所未有的机遇，同时也带来了深刻的挑战 Rahman ([2024](https://arxiv.org/html/2411.08294v1#bib.bib52))。该地区的数字化变革，以智能手机的广泛普及和互联网基础设施的扩展为特征，创造了有利的技术民主化土壤，突破了传统的社会经济障碍 Deichmann et al. ([2016](https://arxiv.org/html/2411.08294v1#bib.bib16))。这场数字复兴催生了数字素养的显著增长，农村地区——传统上是发展干预的重点——在过去五年中年均增长率达到12%，增幅尤其显著 Kass-Hanna et al. ([2022](https://arxiv.org/html/2411.08294v1#bib.bib37))。然而，这一数字转型与该地区资源约束和极其复杂的语言多样性之间存在着动态的张力 Hutson et al. ([2024](https://arxiv.org/html/2411.08294v1#bib.bib32))。南亚的语言格局涵盖了超过650种语言，主要属于印度-雅利安语族和德拉威语族（IADL） Sjoberg ([1992](https://arxiv.org/html/2411.08294v1#bib.bib67))，形成了一个复杂的沟通模式矩阵，传统的开发方法往往难以有效应对这一复杂性。普遍存在的代码混合现象——说话者在单一对话情境中灵活地在多种语言之间切换——不仅反映了语言多样性，还体现了更深层次的文化融合和社会适应模式 Rodríguez Tembrás ([2024](https://arxiv.org/html/2411.08294v1#bib.bib59))。这种语言复杂性，再加上许多语言在计算资源上的局限性 Ranathunga and de Silva ([2022](https://arxiv.org/html/2411.08294v1#bib.bib54))，对有意义的参与式研究和社区参与构成了重大障碍 Hidayatullah et al. ([2022](https://arxiv.org/html/2411.08294v1#bib.bib31))。这些语言障碍的影响远远超出了单纯的沟通难题。传统的研究方法由于资源的限制和方法上的僵化，往往未能有效捕捉到对于有效发展干预至关重要的细致视角和本土知识体系 Golonka et al. ([2014](https://arxiv.org/html/2411.08294v1#bib.bib25))。财务和时间成本相当高，研究表明，语言障碍可能导致研究成本增加最多45%，而项目时间线平均延长60% Daramola et al. ([2024](https://arxiv.org/html/2411.08294v1#bib.bib14))。更为关键的是，这些限制常常导致最重要的声音——尤其是当地社区、女性及其他传统上被边缘化群体的声音——被边缘化，从而影响了发展话语的完整性 Björk Brämberg and Dahlberg ([2013](https://arxiv.org/html/2411.08294v1#bib.bib7))。

高级大型语言模型（LLMs）和复杂的自然语言处理（NLP）技术的出现，为解决这些根深蒂固的挑战提供了一个具有范式转变潜力的机会，Tomec和Gričar（[2024](https://arxiv.org/html/2411.08294v1#bib.bib70)）指出。现代LLM架构展现了跨语言理解和生成的前所未有的能力，即使在资源匮乏的语言环境中也表现出了显著的适应性，Parovic（[2024](https://arxiv.org/html/2411.08294v1#bib.bib50)）对此进行了详细说明。通过高效的微调技术和少量学习方法，这些模型能够适应IADL的复杂语言模式，尽管数字资源有限，Cuadra等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib13)）对此有深入研究，提供了一种强有力的工具，突破了传统的参与和互动壁垒。

基于LLM的智能体系统在南亚发展研究中的应用不仅仅是技术进步——它体现了一种向更加包容和参与性研究方法的根本转变，正如Kar等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib36)）所述。这些系统显著提高了实地研究的效率，可能将多语种研究的资源需求减少高达60%，如Choi等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib10)）所提到的，同时保持高标准的文化敏感性和数据质量，Yong等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib80)）也指出了这一点。从更根本的角度来看，这些系统促成了更加民主的知识生成与共享方式，使社区成员能够使用熟悉的语言和沟通模式参与研究过程。这一变革的潜力超越了操作效率，深入到社区参与发展研究的本质。基于LLM的系统能够捕捉并解释嵌入IADL和代码混合交流中的细微含义，正如Brown（[2024](https://arxiv.org/html/2411.08294v1#bib.bib8)）所提到的，而其先进的NLP能力确保了文化表达的准确解释和语义连贯性的维持，Dutta等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib20)）对此有深入探讨。这一技术基础使得实时识别模式和潜在偏见成为可能，从而支持动态的研究方法调整，提升数据完整性和研究的真实性，正如Jha（[2023](https://arxiv.org/html/2411.08294v1#bib.bib33)）所指出的。

也许最重要的是，这些系统为真正的社区赋权和包容提供了前所未有的机会 Ullah等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib71)）。通过促进实时多语言对话，满足多样的沟通偏好，Matras等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib45)）使边缘化社区能够更全面地参与影响他们生活的研究过程，Yeh和Mitric（[2023](https://arxiv.org/html/2411.08294v1#bib.bib79)）。这些系统能够设计文化上有共鸣的活动并调解跨语言沟通，Zheng等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib81)）有助于确保发展倡议源自并回应真正的社区需求和视角。

通过这些系统优化研究资源，为更可持续和可扩展的发展倡议创造了机会 Singh（[2023](https://arxiv.org/html/2411.08294v1#bib.bib65)）。通过自动化日常任务，同时保留对复杂分析工作的人工监督，研究团队可以更专注于与社区的深入互动 Singh等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib64)）。此外，系统在多种语言中保存和传播知识的能力，包括混合语言形式，Leong等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib42)），有助于保护语言遗产，同时促进社区之间更有效的知识转移，Schroeder等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib62)）。

## 3 提出的LLM4参与性研究框架

我们的框架提出了一个创新的多代理生态系统，旨在通过大规模语言模型（LLMs）和多模态AI能力的战略协同，转变参与性研究。该框架强调了专门的认知代理的无缝整合，这些代理利用现有的LLM能力，同时应对语言多样化背景下参与性发展研究的独特挑战。

### 3.1 核心组件

我们框架的认知架构由四个协同整合的代理系统组成，每个系统旨在解决参与性研究的不同但相互关联的维度，同时保持无缝的操作一致性。

参与式研究设计与分析代理（PRDAA）：作为方法论的设计师，PRDAA负责在多元参与性环境中策划、实施和持续优化研究工具的概念化过程 Rane 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib55)）。通过整合文化知识库、语言模式识别和动态社区反馈回路，这些代理生成方法学上稳健且具有文化共鸣的研究设计——包括结构化的定量工具、半结构化的定性协议和互动式参与框架。它们的先进分析能力促进了研究方法的实时评估和调整，同时其模式识别算法使得研究工具在文化背景下的敏感度得以精细校准 Agathos 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib2)）。通过在试点实施过程中利用多语言和多模态反馈机制，PRDAA能够在问题设计、访谈进程和活动设计上进行动态优化，确保与当地文化范式和交流方式的最佳契合。

社会语义调解代理（SSMA）：SSMA在语言架构与文化认识论的交汇点上运作，超越了传统的翻译范式，旨在促进复杂社会文化现象的解释 Mohamed 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib46)），这些现象涉及印度-雅利安语言的复杂形态结构和语义景观，同时调解南亚交流背景下典型的混合编码模式 Sitaram 等人（[2020](https://arxiv.org/html/2411.08294v1#bib.bib66)）。通过实施动态的语境保持机制和基于注意力的记忆网络，SSMA在多语言对话中保持语义一致性 Dowlagar 和 Mamidi（[2023](https://arxiv.org/html/2411.08294v1#bib.bib19)），而其先进的多模态处理架构能够将多样化的数据模态转化为文化语境化的知识表示 Ye（[2024](https://arxiv.org/html/2411.08294v1#bib.bib78)）。通过整合不断发展的文化知识库 Xi 等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib76)），SSMA能够细致地解读隐含的文化意义，包括复杂的尊称体系和情境化的社会标志 Coffin 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib12)）。

人种学智能代理（EIA）：EIA 在多语言研究环境中体现了人种学数据采集和处理的创新范式，Yang（[2024](https://arxiv.org/html/2411.08294v1#bib.bib77)）通过实时实施和调整研究协议，补充了 PRDAA 的方法框架。EIA 利用其自然语言理解能力处理复杂的代码混合输入，Sadia 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib61)）在此过程中发挥作用，同时其多模态分析框架使其能够同时处理语言内容、非语言线索和上下文元素，Lee 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib41)）。通过与文化知识系统的整合，EIA 促进了生成适合语境的参与性活动和实时数据组织架构，提升了社区参与过程的深度和真实性，George（[2024](https://arxiv.org/html/2411.08294v1#bib.bib24)）。

社区参与协调代理（CEOA）：CEOA 在人类与人工智能研究生态系统中充当中介，协调与当地社区的信任基础参与，同时确保伦理研究实践，Ninan 等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib49)）。通过实施情境感知对话系统和自适应沟通策略，这些代理在处理复杂的社会等级和文化规范时，能够适应多样化的语言偏好和代码混合实践，其交互框架促进了透明和富有同理心的参与，Chow 和 Li（[2024](https://arxiv.org/html/2411.08294v1#bib.bib11)），同时它们的文化规范引擎能够应对复杂的社会动态和敬语系统，Guo 等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib27)）。CEOA 维持全面的知情同意和数据主权机制，将社区代理性和赋权原则嵌入到研究过程中，Ahmad 和 Islam（[2024](https://arxiv.org/html/2411.08294v1#bib.bib3)）。

### 3.2 融入参与性方法

该框架将这些认知代理无缝地集成到既有的参与性研究方法中，通过智能的、具备上下文感知能力的支持，增强每个阶段的效果（Tarkoma等人，[2023](https://arxiv.org/html/2411.08294v1#bib.bib69)）。在调查过程中，SSMAs和EIAs合作创建和实施文化适宜的工具（de Jesús-Espinosa等人，[2024](https://arxiv.org/html/2411.08294v1#bib.bib15)），动态调整以适应参与者的语言偏好和交流风格。在访谈场景中，代理提供实时的文化-语言中介服务，建议文化敏感的提问，同时跨语言边界捕捉细致的叙事（Kapania等人，[2024](https://arxiv.org/html/2411.08294v1#bib.bib35)，Al-khresheh，[2024](https://arxiv.org/html/2411.08294v1#bib.bib4)）。对于研讨会和参与性活动，该系统促进文化共鸣的互动会议（Macías-Gómez-Estern和Lalueza，[2024](https://arxiv.org/html/2411.08294v1#bib.bib43)），将多元化的观点综合成全面且具有文化细节的总结（Shu等人，[2024](https://arxiv.org/html/2411.08294v1#bib.bib63)）。

### 3.3 工作流与系统架构

该整合涵盖了整个研究过程，强调人类研究人员与认知代理Archibald（[2023](https://arxiv.org/html/2411.08294v1#bib.bib5)）之间的无缝合作。在实地准备阶段，系统支持方法论的调整和工具的定制Chan等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib9)），识别潜在的文化语言挑战，并制定缓解策略Garcia Valencia等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib23)）。在实地工作期间，代理提供实时支持用于数据收集和初步分析Wang等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib74)），并能根据新出现的洞察进行响应性调整Agarwal等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib1)）。数据处理阶段利用了复杂的跨语言分析能力Gaba等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib22)），在多模态数据集中揭示模式，同时保持可解释性[Kejriwal和Benus](https://arxiv.org/html/2411.08294v1#bib.bib38)，Mora-Cantallops等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib47)）。该框架嵌入了持续反馈机制Jin等人（[2024](https://arxiv.org/html/2411.08294v1#bib.bib34)），使得代理性能能够经过反复迭代进行完善，同时通过社区验证和知识共同创造体现参与式原则Delgado等人（[2023](https://arxiv.org/html/2411.08294v1#bib.bib17)）。这种人类专业知识与人工智能的动态整合，创造了一个强大的参与式研究生态系统，在保持方法学严谨性的同时，确保文化敏感性和社区赋能。

![参见标题](img/6c3f892831849d81fe0ec368358b75fa.png)

图1：将AI辅助工具与传统的参与式方法相结合。（来源：作者的实地工作）

## 4 现场工作与实施洞察

我们的研究聚焦于提升尼尔瓦拉河流域的洪水预警系统（EWS），该地区因频繁的洪水灾害而造成严重的社会经济影响。斯里兰卡的语言景观是南亚更广泛语言多样性的典型，具有代码混合和多语言交流的特点。僧伽罗语作为一种印欧-雅利安语族的语言，具有粘着性特征和丰富的敬语系统，日常对话中常常与英语及其他地方方言交织在一起。这种代码混合现象为自然语言处理带来了巨大挑战，因为它涉及到句法、词汇和语义的混合，而传统的语言模型往往难以准确解读。我们的目标是运用多代理系统促进参与式发展研究方法——包括问卷调查、结构化和半结构化访谈、工作坊及其他互动性参与——涵盖从国家机构到地方社区的各类利益相关者。

### 4.1 实践经验与成果

该方法的实施面临着一些挑战，尤其是在将大语言模型（LLMs）适应于处理僧伽罗语特有语言特征和广泛的代码混合现象时。由于缺乏高质量的、经过注释的僧伽罗语语料库，迫使我们采取创新的方法，包括主动学习技术和数据增强策略，以提高模型的能力。

一项重要的成就是开发了一种混合翻译方法，结合了统计方法和神经网络方法，与标准的多语言模型相比，针对领域特定术语的翻译准确率提高了35%。这一进展对于在访谈中准确解读参与者的回答至关重要，确保了微妙的语气不会在翻译过程中丢失。

在应用参与式方法时，代理发挥了不可或缺的作用。在工作坊中，它们协助设计与当地习俗相符的互动活动，并促进了实时反馈的收集。在问卷调查和访谈中，代理帮助生成符合文化背景的提问，并根据参与者的反馈动态调整，从而增强了数据收集的深度和真实性。

代理在分析阶段也发挥了至关重要的作用。它们使跨语言对比成为可能，并促进了复杂数据的整合，从而转化为可操作的见解。例如，它们帮助识别了在预警系统（EWS）中参与机构之间的沟通瓶颈，揭示了过时的沟通方式和官僚主义程序是有效灾害管理的重大障碍。

### 4.2 伦理与文化考量

在整个实施过程中，伦理考虑始终是重中之重。代理人被设计用来保护数据隐私并获得知情同意，同时透明地沟通其角色和局限性。鉴于与灾难相关的研究具有敏感性，并且受影响社区存在脆弱性，系统融入了强有力的机制来防止偏见并确保多元声音的公平表达。文化敏感性深深融入系统设计之中。通过将当地风俗、社会结构和语言细节纳入代理人的运作中，我们营造了一个信任和尊重的环境。这种方法不仅增强了参与者的参与感，还丰富了收集数据的质量。

### 4.3 教训总结与建议

案例研究揭示了几个关键的见解：

社区参与至关重要：当地利益相关者积极参与系统的开发和完善是必不可少的。他们的反馈确保了代理人能够文化适应并响应社区需求，增强了接受度和效果。

灵活的适应机制是必要的：语言多样性和混合语言的实践要求代理人具有高度的适应性。实施持续学习和实时调整的机制对于应对语言变异和意外输入至关重要。

人类监督仍然是不可或缺的：尽管代理人显著提升了效率和深度，但人类研究者在监督过程、解释细致的文化背景和做出伦理判断方面发挥了至关重要的作用。

解决技术挑战：克服语言资源的匮乏需要创新的技术解决方案。投资开发注释语料库并利用迁移学习是提升模型性能的有效策略。

### 4.4 更广泛部署的实施考虑

僧伽罗语实现的成功展示了该框架扩展到其他南亚语言和环境的潜力，如泰米尔语、孟加拉语和尼泊尔语。模块化设计允许根据不同的语言特征和这些语言中普遍存在的混合语言模式进行定制。此外，该框架在各个发展领域的应用前景广阔。它能够促进细致的、情境敏感的参与，使其适用于医疗、教育、农业等领域的研究。通过利用先进的人工智能技术增强参与式方法，系统可以为更有效和可持续的开发干预做出贡献。框架的规模化需要解决一些技术和实践上的问题：

技术基础设施：在资源有限的环境中部署系统需要优化模型架构以提高效率。将基于云的处理与边缘计算解决方案结合使用，可以缓解与有限计算资源和网络连接性相关的挑战。

数据安全与隐私：确保敏感数据的保护至关重要。实施端到端加密、联邦学习和差分隐私技术可以在保护个人隐私的同时，保持收集数据的有效性。

文化与伦理敏感性：将系统适应新的环境需要深入了解当地的文化、语言和社会动态。在开发过程中与当地专家和社区的互动对于保持伦理标准和促进接受度至关重要。

能力建设：培训当地研究人员和利益相关者使用该系统有助于增强可持续性并赋能社区。开发全面的培训项目，涵盖技术操作和文化能力，是至关重要的。

政策和制度支持：成功的大规模实施需要有支持性的政策和制度框架。与政府和非政府组织的合作可以促进资源配置、基础设施建设以及最佳实践的标准化。

## 5 讨论与未来议程

基于大型语言模型（LLM）的多智能体系统在南亚参与式发展研究中的应用，代表了社会科学方法论的范式转变，这需要一个全面的研究议程，涵盖技术、方法论和政策层面。这种人工智能与参与式发展的融合要求仔细考虑技术能力和社会文化影响，特别是在语言多样性和资源约束显著的地区。

技术发展的轨迹必须优先考虑超越当前跨文化和多语言环境限制的架构开发。这需要在少样本学习优化和跨语言迁移机制方面进行基础性的创新，特别是在处理南亚语言中普遍存在的代码切换现象和方言变异时。这些进展必须在通过精密的模型压缩技术减少计算需求、同时保持语义一致性的前提下实现。这些系统的发展应超越单纯的语言处理，扩展到包括手势识别和上下文理解在内的多模态交互能力，这些能力应与当地的交流模式和文化规范相一致。

方法论框架必须不断发展，以有效整合这些技术能力，同时保持参与式研究的基本原则。这种整合需要开发复杂的评估指标，这些指标超越了传统的技术标准，涵盖了社会影响、社区赋权和文化保护的衡量标准。必须特别关注开发强大的偏见检测和减轻机制，解决技术和社会文化两个维度的问题，确保这些系统不会延续现有的不平等现象或引入新的系统性偏见。这些方法论创新必须配备严格的质量保证协议，以保持系统的可靠性，同时适应不同的实施环境。

监管这些技术实施所需的政策框架必须在创新与伦理考量和社会责任之间找到平衡。这要求建立全面的治理结构，解决数据主权、隐私保护和算法问责制问题，同时保持足够的灵活性，以适应当地文化规范和社会动态。基础设施发展战略必须优先考虑可持续解决方案，使长期研究计划得以实施，同时解决发展中地区的资源限制。这些战略应涵盖技术基础设施和人力能力建设，特别强调培养当地在人工智能辅助参与式研究方法方面的专业知识。

这些系统的实际实施要求采取精心协调的方法，将技术能力、方法论严谨性和伦理考虑相结合。研究人员必须采用迭代开发过程，结合社区利益相关者的持续反馈，同时保持高标准的科学有效性。该实施框架应强调开发可适应的、以用户为中心的解决方案，支持文化定制的同时保持技术效率。这些实施的成功在很大程度上依赖于建立协作网络，促进研究机构和社区之间的知识共享与资源优化。

基于LLM的参与性研究在南亚的未来发展轨迹因此需要在技术创新与社会文化敏感性之间找到微妙的平衡。这个平衡只能通过技术人员、研究人员、政策制定者和社区利益相关者之间的持续合作来实现，共同开发出既技术先进又社会负责的解决方案。这些举措的最终成功不仅仅通过其技术性能来衡量，更在于其能否在保留并增强地方文化价值和社会结构的同时，有意义地促进参与性发展。

## 6 结论

在南亚发展研究背景中实施基于LLM的多代理系统展现了显著的潜力和重要的挑战。我们的分析揭示了这些系统在提升研究效率和效果的同时，能够保持文化敏感性和社区参与的可行性。其影响潜力不仅仅局限于技术改善，还包括提升研究质量、改善社区参与以及跨语言边界更有效的知识共享。我们已识别出能够引导成功系统部署的实施路径，特别关注保持灵活性和适应性。资源需求也经过了仔细分析，清晰展示了成功实施所需的投资，同时突出了跨机构优化和共享资源的机会。

研究界必须在推动人工智能辅助研究方法在南亚背景下的开发和实施中发挥主导作用。这不仅包括技术研究和开发，还需要仔细考虑方法论的影响和伦理考量。我们呼吁研究人员积极参与开发和完善这些工具，同时保持对参与性研究原则和社区赋权的强烈承诺。发展机构必须认识到这些技术的变革潜力，同时承担起确保其伦理和有效实施的责任。这包括在基础设施和能力建设方面进行必要的投资，同时制定适当的政策和指导方针，以支持技术采纳。技术行业必须优先开发能够解决低资源语言环境中发展研究特定需求的解决方案，特别关注确保对不同用户群体的可访问性和可用性。政策制定者必须努力创造有利的环境，以有效部署人工智能技术在发展研究中的应用。这包括制定适当的监管框架，确保足够的资源分配，并推动支持伦理和有效技术实施的标准。LLM基础系统在参与性发展研究中的成功整合需要不同利益相关者之间的协调行动，携手实现这些技术的潜力，同时应对相关的挑战和风险。

## 参考文献

+   Agarwal et al. (2024) Swati Agarwal, Ashrut Kumar, 和 Rijul Ganguly. 2024. 使用Twitter调查基于变换器的模型在印度铁路自动化电子政务中的应用。*多媒体工具与应用*, 83(2):4551–4577.

+   Agathos et al. (2024) Leonidas Agathos, Andreas Avgoustis, Xristiana Kryelesi, Aikaterini Makridou, Ilias Tzanis, Despoina Mouratidis, Katia Lida Kermanidis, and Andreas Kanavos. 2024. [弥合语言差距：开发希腊文本简化数据集](https://doi.org/https://doi.org/10.3390/info15080500). *信息*, 15(8):500.

+   Ahmad 和 Islam (2024) Ifzal Ahmad 和 M Rezaul Islam. 2024. 授权与参与：包容性发展的关键策略。见于 *建立强大社区：包容性发展中的伦理方法*，第47–68页。Emerald Publishing Limited.

+   Al-khresheh (2024) Mohammad H Al-khresheh. 2024. 从全球视角弥合技术与教学法的差距：教师在英语语言教学中整合ChatGPT的视角。*计算机与教育：人工智能*, 6:100218.

+   Archibald (2023) Mandy M Archibald. 2023. 关于“混合方法研究中的协作实践”的虚拟专题。

+   Awad等人（2016）Germine H Awad, Erika A Patall, Kadie R Rackley, 和 Erin D Reilly。2016年。[文化敏感研究方法的建议](https://doi.org/https://doi.org/10.1080/10474412.2015.1046600)。*教育与心理咨询杂志*，26(3)：283–303。

+   Björk Brämberg和Dahlberg（2013）Elisabeth Björk Brämberg和Karin Dahlberg。2013年。[跨文化访谈中的口译员：数据的三方共同构建](https://doi.org/https://doi.org/10.1177/1049732312467705)。*定性健康研究*，23(2)：241–247。

+   Brown（2024）Nik Bear Brown。2024年。[增强对大语言模型（LLMs）的信任：比较和解释LLMs的算法](https://doi.org/https://doi.org/10.48550/arXiv.2406.01943)。

+   Chan等人（2024）Gerry Chan, Bilikis Banire, Grace Ataguba, George Frempong, 和 Rita Orji。2024年。[数字技术中的社会文化敏感性全景视角：全面综述与未来方向](https://doi.org/https://doi.org/10.1080/10447318.2024.2372135)。*国际人机交互杂志*，第1–29页。

+   Choi等人（2023）Yunjae J Choi, Minha Lee, 和 Sangsu Lee。2023年。[迈向多语言对话代理：代码混合多语言用户的挑战与期望](https://doi.org/https://doi.org/10.1145/3544548.3581445)。载于*2023年CHI计算机系统人因会议论文集*，第1–17页。

+   Chow和Li（2024）James CL Chow和Kay Li。2024年。人本AI中的伦理考量：通过大语言模型推动肿瘤学聊天机器人发展。*JMIR生物信息学与生物技术*，5:e64406。

+   Coffin等人（2024）Alyssa Coffin, Brynn Elder, Marcella Luercio, Namrata Ahuja, Rebecca Barber, Lisa Ross DeCamp, Karen Encalada, Angela L Fan, Jonathan S Farkas, Pia Jain等人。2024年。[为研究创建文化适应的多语言材料](https://doi.org/https://doi.org/10.1542/peds.2023-063988)。*儿科学*，e2023063988页。

+   Cuadra等人（2024）Andrea Cuadra, Justine Breuch, Samantha Estrada, David Ihim, Isabelle Hung, Derek Askaryar, Marwan Hassanien, Kristen L Fessele, 和 James A Landay。2024年。[为所有人提供数字表单：一个全面的多模态大语言模型代理，用于健康数据输入](https://doi.org/https://doi.org/10.1145/3659624)。*ACM互动、移动、可穿戴与普适技术会议论文集*，8(2)：1–39。

+   Daramola等人（2024）Gideon Oluseyi Daramola, Adetomi Adewumi, Boma Sonimiteim Jacks, 和 Olakunle Abayomi Ajala。2024年。[应对复杂性：跨国能源项目中的沟通障碍回顾](https://doi.org/https://doi.org/10.51594/ijarss.v6i4.1062)。*国际应用社会科学研究杂志*，6(4)：685–697。

+   de Jesús-Espinosa 等人（2024）Tania de Jesús-Espinosa, Solymar Solís-Báez, Claudia P Valencia-Molina, Juan Camilo Triana Orrego, Joas Benítez Duque, J Craig Phillips, Rebecca Schnall, Yvette P Cuca, Wei-Ti Chen, Sheila Shaibu 等人. 2024. 跨文化定性研究中的开放性问题翻译：一个全面框架。*跨文化护理杂志*，页面 10436596241271248。

+   Deichmann 等人（2016）Uwe Deichmann, Aparajita Goyal 和 Deepak Mishra. 2016. 数字技术会改变发展中国家的农业吗？*农业经济学*，47(S1)：21–33。

+   Delgado 等人（2023）Fernando Delgado, Stephen Yang, Michael Madaio 和 Qian Yang. 2023. [AI设计中的参与转向：理论基础与当前实践状况](https://doi.org/https://doi.org/10.1145/3617694.3623261). 见 *第3届ACM算法、机制与优化中的公平性与访问会议论文集*，第1–23页。

+   Diab Idris 等人（2024）Mohamed Diab Idris, Xiaohua Feng 和 Vladimir Dyo. 2024. [革命性高等教育：释放大型语言模型的潜力，推动战略性变革](https://doi.org/10.1109/ACCESS.2024.3400164). *IEEE Access*，12：67738–67757。

+   Dowlagar 和 Mamidi（2023）Suman Dowlagar 和 Radhika Mamidi. 2023. 面向任务的医学领域混合代码对话数据集。*计算机语音与语言*，78：101449。

+   Dutta 等人（2024）Manaswita Dutta, Tina MD Mello, Yesi Cheng, Niladri Sekhar Dash, Ranita Nandi, Aparna Dutt 和 Arpita Bose. 2024. [双语阿尔茨海默病患者的普遍及语言特定连贯语音特征：来自结构上不同语言的案例研究的洞察](https://doi.org/https://doi.org/10.1044/2024_JSLHR-23-00254). *语音、语言与听力研究杂志*，67(4)：1143–1164。

+   Ferdaus 等人（2024）Md Meftahul Ferdaus, Mahdi Abdelguerfi, Elias Ioup, Kendall N. Niles, Ken Pathak 和 Steven Sloan. 2024. [迈向值得信赖的AI：伦理与稳健的大型语言模型综述](https://doi.org/https://doi.org/10.48550/arXiv.2407.13934)。

+   Gaba 等人（2023）Shivani Gaba, Haneef Khan, Khalid Jaber Almalki, Abdoh Jabbari, Ishan Budhiraja, Vimal Kumar, Akansha Singh, Krishna Kant Singh, S. S. Askar 和 Mohamed Abouhawwash. 2023. [Holochain：一种面向代理的分布式哈希表安全性，应用于智能物联网](https://doi.org/10.1109/ACCESS.2023.3300220). *IEEE Access*，11：81205–81223。

+   Garcia Valencia 等人（2023）Oscar A Garcia Valencia, Charat Thongprayoon, Caroline C Jadlowiec, Shennen A Mao, Jing Miao 和 Wisit Cheungpasitporn. 2023. [通过集成聊天机器人提升肾脏移植护理](https://doi.org/https://doi.org/10.3390/healthcare11182518). 见 *医疗保健*，第11卷，第2518页。MDPI。

+   George（2024）A Shaji George。2024年。[利用物联网和人工智能持续观察人类社会动态](https://doi.org/https://doi.org/10.5281/zenodo.13937024)。*Partners Universal Innovative Research Publication*，2(5)：01–17。

+   Golonka 等人（2014）Ewa M Golonka、Anita R Bowles、Victor M Frank、Dorna L Richardson 和 Suzanne Freynik。2014年。 [外语学习的技术：技术类型及其有效性回顾](https://doi.org/https://doi.org/10.1080/09588221.2012.700315)。*计算机辅助语言学习*，27(1)：70–105。

+   Göpferich 和 Jääskeläinen（2009）Susanne Göpferich 和 Riitta Jääskeläinen。2009年。[翻译能力发展的过程研究：我们处于何种阶段，未来需要走向何方？](https://doi.org/https://doi.org/10.1556/Acr.10.2009.2.1) *跨语言与文化*，10(2)：169–191。

+   Guo 等人（2023）Zishan Guo、Renren Jin、Chuang Liu、Yufei Huang、Dan Shi、Supryadi、Linhao Yu、Yan Liu、Jiaxuan Li、Bojian Xiong 和 Deyi Xiong。2023年。[评估大型语言模型：全面调查](https://doi.org/https://doi.org/10.48550/arXiv.2310.19736)。

+   Hadi 等人（2024）Muhammad Usman Hadi、Qasem Al Tashi、Abbas Shah、Rizwan Qureshi、Amgad Muneer、Muhammad Irfan、Anas Zafar、Muhammad Bilal Shaikh、Naveed Akhtar、Jia Wu 等人。2024年。[大型语言模型：其应用、挑战、局限性及未来前景的全面调查](https://doi.org/https://doi.org/10.36227/techrxiv.23589741.v6)。*Authorea Preprints*。

+   Hashmi 等人（2024）Ehtesham Hashmi、Sule Yildirim Yayilgan、Ibrahim A. Hameed、Muhammad Mudassar Yamin、Mohib Ullah 和 Mohamed Abomhara。2024年。[增强多语言仇恨言论检测：从语言特定洞察到跨语言整合](https://doi.org/10.1109/ACCESS.2024.3452987)。*IEEE Access*，12：121507–121537。

+   Hassan 等人（2023）Mohammad Hassan、Prakash Rai 和 Sabin Maharjan。2023年。[赋能南亚农业社区：通过意识、培训和合作推动物联网驱动的农业综合方法](https://doi.org/https://dx.doi.org/10.4314/jae.v28i3.2)。*新兴技术与创新季刊*，8(3)：18–32。

+   Hidayatullah 等人（2022）Ahmad Fathan Hidayatullah、Atika Qazi、Daphne Teck Ching Lai 和 Rosyzie Anna Apong。2022年。[关于代码混合文本的语言识别的系统性回顾：技术、数据可用性、挑战与框架发展](https://doi.org/10.1109/ACCESS.2022.3223703)。*IEEE Access*，10：122812–122831。

+   Hutson 等人（2024）James Hutson、Pace Ellsworth 和 Matt Ellsworth。2024年。[数字时代保护语言多样性：一种可扩展的文化遗产延续模型](https://doi.org/https://doi.org/10.58803/jclr.v3i1.96)。*当代语言研究杂志*，3(1)。

+   Jha (2023) Rahul Kumar Jha. 2023. [加强智能电网网络安全：深入研究机器学习与自然语言处理的融合](https://doi.org/https://doi.org/10.36548/jtcsst.2023.3.005)。*计算机科学与智能技术趋势杂志*, 5(3):284–301。

+   Jin 等人 (2024) Haolin Jin, Linghan Huang, Haipeng Cai, Jun Yan, Bo Li 和 Huaming Chen. 2024. [从LLMs到基于LLM的代理在软件工程中的应用：当前挑战与未来发展综述](https://doi.org/https://doi.org/10.48550/arXiv.2408.02479)。

+   Kapania 等人 (2024) Shivani Kapania, William Agnew, Motahhare Eslami, Hoda Heidari 和 Sarah Fox. 2024. [‘故事的模拟物’：将大型语言模型视为定性研究参与者的探讨](https://doi.org/https://doi.org/10.48550/arXiv.2409.19430)。

+   Kar 等人 (2024) Indrajit Kar, Zonunfeli Ralte, Maheshakumara Shivakumara, Rana Roy 和 Arti Kumari. 2024. [代理就是你所需要的一切：通过先进的生成性AI驱动的对话LLM代理和工具提升交易动态](https://doi.org/https://doi.org/10.1109/I2CT61223.2024.10543356)。在 *2024 IEEE第九届技术融合国际会议 (I2CT)*，第1–6页。IEEE。

+   Kass-Hanna 等人 (2022) Josephine Kass-Hanna, Angela C Lyons 和 Fan Liu. 2022. [通过金融和数字素养在南亚和撒哈拉以南非洲建立金融韧性](https://doi.org/https://doi.org/10.1016/j.ememar.2021.100846)。*新兴市场评论*, 51:100846。

+   (38) Jay Kejriwal 和 Stefan Benus. [斯洛伐克语、西班牙语、英语和匈牙利语中的词汇、句法、语义和声学同步：跨语言比较](https://doi.org/https://dx.doi.org/10.2139/ssrn.4729223)。*西班牙语、英语和匈牙利语：跨语言比较*。

+   Kovač 等人 (2024) Grgur Kovač, Rémy Portelas, Peter Ford Dominey 和 Pierre-Yves Oudeyer. 2024. [社交人工智能学校：一种利用发展心理学推动人工社会文化代理的框架](https://doi.org/https://doi.org/10.3389/fnbot.2024.1396359)。*前沿神经机器人学*, 18:1396359。

+   Kshetri (2024) Nir Kshetri. 2024. [生成性人工智能中的语言挑战：对发展中国家低资源语言的影响](https://doi.org/https://doi.org/10.1080/1097198X.2024.2341496)。

+   Lee 等人 (2024) Sangmin Lee, Minzhi Li, Bolin Lai, Wenqi Jia, Fiona Ryan, Xu Cao, Ozgur Kara, Bikram Boote, Weiyan Shi, Diyi Yang 和 James M. Rehg. 2024. [迈向社交人工智能：关于理解社交互动的调查](https://doi.org/https://doi.org/10.48550/arXiv.2409.15316)。

+   Leong 等人 (2023) Wei Qi Leong, Jian Gang Ngui, Yosephine Susanto, Hamsawardhini Rengarajan, Kengatharaiyer Sarveswaran 和 William Chandra Tjhi. 2023. [Bhasa：面向大型语言模型的东南亚语言文化整体评估套件](https://doi.org/https://doi.org/10.48550/arXiv.2309.06085)。

+   Macías-Gómez-Estern 和 Lalueza（2024）Beatriz Macías-Gómez-Estern 和 José Luis Lalueza. 2024. [在高等教育服务学习中作为混合场景的 i-positioning 导航：一个案例研究](https://doi.org/https://doi.org/10.1016/j.lcsi.2024.100805)。*学习、文化与社会互动*，45:100805。

+   Magueresse 等（2020）Alexandre Magueresse、Vincent Carles 和 Evan Heetderks. 2020. [低资源语言：过去工作的回顾与未来挑战](https://doi.org/https://doi.org/10.48550/arXiv.2006.07264)。

+   Matras 等（2023）Yaron Matras、Rebecca Tipton 和 Leonie Gaiser. 2023. [公共卫生保健中的代理性与多语言：从业者如何利用本地经验和接触](https://doi.org/https://doi.org/10.4337/9781789906783.00011)。收录于 *理解职业语境中的语言与多语言动态*，第46–60页。Edward Elgar Publishing。

+   Mohamed 等（2024）Yasir Abdelgadir Mohamed、Akbar Khanan、Mohamed Bashir、Abdul Hakim H. M. Mohamed、Mousab A. E. Adiel 和 Muawia A. Elsadig. 2024. [人工智能对语言翻译的影响：综述](https://doi.org/10.1109/ACCESS.2024.3366802)。*IEEE Access*，12:25553–25579。

+   Mora-Cantallops 等（2024）Marçal Mora-Cantallops、Elena García-Barriocanal 和 Miguel-Ángel Sicilia. 2024. [生物医学决策应用中的可信 AI 指南：范围审查](https://doi.org/https://doi.org/10.3390/bdcc8070073)。*大数据与认知计算*，8(7):73。

+   Nekoto 等（2020）Wilhelmina Nekoto、Vukosi Marivate、Tshinondiwa Matsila、Timi Fasubaa、Tajudeen Kolawole、Taiwo Fagbohungbe、Solomon Oluwole Akinola、Shamsuddeen Hassan Muhammad、Salomon Kabongo、Salomey Osei、Sackey Freshia、Rubungo Andre Niyongabo、Ricky Macharm、Perez Ogayo、Orevaoghene Ahia、Musie Meressa、Mofe Adeyemi、Masabata Mokgesi-Selinga、Lawrence Okegbemi、Laura Jane Martinus、Kolawole Tajudeen、Kevin Degila、Kelechi Ogueji、Kathleen Siminyu、Julia Kreutzer、Jason Webster、Jamiil Toure Ali、Jade Abbott、Iroro Orife、Ignatius Ezeani、Idris Abdulkabir Dangana、Herman Kamper、Hady Elsahar、Goodness Duru、Ghollah Kioko、Espoir Murhabazi、Elan van Biljon、Daniel Whitenack、Christopher Onyefuluchi、Chris Emezue、Bonaventure Dossou、Blessing Sibanda、Blessing Itoro Bassey、Ayodele Olabiyi、Arshath Ramkilowan、Alp Öktem、Adewale Akinfaderin 和 Abdallah Bashir. 2020. [低资源机器翻译的参与性研究：非洲语言的案例研究](https://doi.org/https://doi.org/10.48550/arXiv.2010.02353)。

+   Ninan 等（2024）Johan Ninan、Stewart Clegg、Ashwin Mahalingam 和 Shankar Sankaran. 2024. [通过信任治理：澳大利亚城市重建区的社区参与](https://doi.org/https://doi.org/10.1177/87569728231182045)。*项目管理杂志*，55(1):16–30。

+   Parovic（2024）Marinela Parovic。2024年。[*提高低资源语言跨语言迁移的参数效率*](https://doi.org/https://doi.org/10.17863/CAM.111817)。博士论文。

+   Pfeffer 等人（2013）Karin Pfeffer、Isa Baud、Eric Denis、Dianne Scott 和 John Sydenstricker-Neto。2013年。[参与式空间知识管理工具：赋能与扩展还是排除？](https://doi.org/https://doi.org/10.1080/1369118X.2012.687393) *Information, Communication & Society*，16(2)：258–285。

+   Rahman（2024）Qazi Arka Rahman。2024年。[重新构思南亚：孟加拉国文学与表现政治](https://doi.org/http://doi.org/10.33915/etd.12549)。

+   Raiaan 等人（2024）Mohaimenul Azam Khan Raiaan、Md. Saddam Hossain Mukta、Kaniz Fatema、Nur Mohammad Fahad、Sadman Sakib、Most Marufatul Jannat Mim、Jubaer Ahmad、Mohammed Eunus Ali 和 Sami Azam。2024年。[大型语言模型综述：架构、应用、分类法、开放问题与挑战](https://doi.org/https://doi.org/10.1109/ACCESS.2024.3365742)。*IEEE Access*，12：26839–26874。

+   Ranathunga 和 de Silva（2022）Surangika Ranathunga 和 Nisansa de Silva。2022年。[某些语言比其他语言更平等：深入探讨 NLP 世界中的语言差异](https://doi.org/https://doi.org/10.48550/arXiv.2210.08523)。

+   Rane 等人（2024）Nitin Liladhar Rane、Mallikarjuna Paramesha、Saurabh P Choudhary 和 Jayesh Rane。2024年。[人工智能、机器学习和深度学习在先进商业策略中的应用：综述](https://doi.org/https://doi.org/10.5281/zenodo.12208298)。*Partners Universal International Innovation Journal*，2(3)：147–171。

+   Rane 等人（2023）Nitin Liladhar Rane、Abhijeet Tawde、Saurabh P Choudhary 和 Jayesh Rane。2023年。[ChatGPT 和其他大型语言模型（LLM）对科学与研究进展的贡献与表现：双刃剑](https://doi.org/https://www.doi.org/10.56726/IRJMETS45312)。*International Research Journal of Modernization in Engineering Technology and Science*，5(10)：875–899。

+   Ray（2023）Partha Pratim Ray。2023年。[ChatGPT：关于背景、应用、主要挑战、偏见、伦理、限制和未来发展方向的综合评述](https://doi.org/https://doi.org/10.1016/j.iotcps.2023.04.003)。*Internet of Things and Cyber-Physical Systems*，3：121–154。

+   Roberts 等人（2024）John Roberts、Max Baker 和 Jane Andrew。2024年。[人工智能与定性研究：大型语言模型（LLM）‘助手’的承诺与风险](https://doi.org/https://doi.org/10.1016/j.cpa.2024.102722)。*Critical Perspectives on Accounting*，99：102722。

+   Rodríguez Tembrás（2024）Vanesa Rodríguez Tembrás。2024年。[*双语医学咨询中的语言转换（加利西亚语-西班牙语）*](https://doi.org/https://doi.org/10.11588/heidok.00035533)。博士论文。

+   Sabarirajan 等人（2024）A Sabarirajan, Latha Thamma Reddi, Sandeep Rangineni, R Regin, S Suman Rajest, 和 P Paramasivan. 2024. [《利用MIS技术保护印度的文化遗产：数字化、可访问性与可持续性》](https://doi.org/10.4018/979-8-3693-0049-7.ch009)。在《*数据驱动的智能商业可持续性*》一书中，第122-135页。IGI Global。

+   Sadia 等人（2024）Bareera Sadia, Farah Adeeba, Sana Shams, 和 Kashif Javed. 2024. [《应对挑战：自动化乌尔都语会议摘要的基准语料库》](https://doi.org/https://doi.org/10.1016/j.ipm.2024.103734)。*信息处理与管理*, 61(4):103734。

+   Schroeder 等人（2024）Hope Schroeder, Marianne Aubin Le Quéré, Casey Randazzo, David Mimno, 和 Sarita Schoenebeck. 2024. [《定性研究中的大语言模型：我们能否公平对待数据？》](https://doi.org/https://doi.org/10.48550/arXiv.2410.07362)

+   Shu 等人（2024）Yuanyuan Shu, Yakun Ma, Wei Li, Guangwei Hu, Xizi Wang, 和 Qianyou Zhang. 2024. [《揭示社会治理创新的动态：一种结合NLP和网络分析的协同方法》](https://doi.org/https://doi.org/10.1016/j.eswa.2024.124632)。*专家系统与应用*, 255:124632。

+   Singh 等人（2024）Pushpdeep Singh, Mayur Patidar, 和 Lovekesh Vig. 2024. [《跨文化翻译：大语言模型用于语言内部文化适应》](https://doi.org/https://doi.org/10.48550/arXiv.2406.14504)。

+   Singh（2023）Vasudha Singh. 2023. [《*探索基于大语言模型（LLM）的聊天机器人在人力资源中的角色*》](https://doi.org/https://doi.org/10.26153/tsw/51146)。博士论文。

+   Sitaram 等人（2020）Sunayana Sitaram, Khyathi Raghavi Chandu, Sai Krishna Rallabandi, 和 Alan W Black. 2020. [《代码切换语音和语言处理综述》](https://doi.org/https://doi.org/10.48550/arXiv.1904.00784)。

+   Sjoberg（1992）Andrée F Sjoberg. 1992. [《*德拉威语对印欧语系的影响：概述*》](https://doi.org/https://doi.org/10.1515/9783110867923.507)。无出版单位。

+   Skirgård 等人（2023）Hedvig Skirgård, Hannah J Haynie, Damián E Blasi, Harald Hammarström, Jeremy Collins, Jay J Latarche, Jakob Lesage, Tobias Weber, Alena Witzlack-Makarevich, Sam Passmore 等人. 2023. [《Grambank揭示了基因约束对语言多样性的影响，并突出语言流失的影响》](https://doi.org/https://doi.org/10.1126/sciadv.adg6175)。*Science Advances*, 9(16):eadg6175。

+   Tarkoma 等人（2023）Sasu Tarkoma, Roberto Morabito, 和 Jaakko Sauvola. 2023. [《AI原生互连框架：将大语言模型技术集成到6G系统中的应用》](https://doi.org/https://doi.org/10.48550/arXiv.2311.05842)。

+   托梅茨和格里察尔（2024）蒂娜·托梅茨和谢尔盖·格里察尔。2024年。[全球化世界中的风险语言障碍：来自斯洛文尼亚女性经理的见解](https://doi.org/https://doi.org/10.5937/StraMan2300054T)。*战略管理-国际战略管理与决策支持系统杂志*，29(2)。

+   乌拉赫等人（2024）埃赫桑·乌拉赫、阿尼尔·帕瓦尼、米尔扎·曼苏尔·贝格和拉金德拉·辛格。2024年。[使用大型语言模型（LLM）如ChatGPT进行诊断医学的挑战与障碍，聚焦于数字病理学——最近的范围审查](https://doi.org/https://doi.org/10.1186/s13000-024-01464-7)。*诊断病理学*，19(1):43。

+   尤尔温等人（2023）埃丽莎·尤尔温、艾萨尔金·博托耶娃、罗萨里奥·阿里亚斯、奥斯卡·瓦尔加斯和帕米娜·费尔乔。2023年。[翻转测量与评估中的权力动态：国际援助与建立基础性问责模型的潜力](https://doi.org/https://doi.org/10.1111/nejo.12448)。*谈判杂志*，39(4):401–426。

+   范伦斯堡和范德·韦斯特胡伊岑（2024）赞德·詹塞·范伦斯堡和索尼娅·范德·韦斯特胡伊岑。2024年。[学术界中的伦理AI集成：为南非高等教育中的大型语言模型（LLM）开发基于素养的框架](https://doi.org/10.4018/979-8-3693-1054-0.ch002)。收录于*高等教育中的AI素养方法*，第23–48页。IGI Global。

+   王等人（2024）邴张、蔡志宇、穆罕默德·蒙朱鲁尔·卡里姆、刘晨曦和王银海。2024年。[交通表现GPT（TP-GPT）：实时数据驱动的智能聊天机器人用于交通监控与管理](https://doi.org/https://doi.org/10.48550/arXiv.2405.03076)。

+   吴等人（2023）吴清云、加甘·班萨尔、张洁宇、吴怡然、李贝宾、朱尔康、蒋莉、张晓云、张绍坤、刘家乐、艾哈迈德·哈桑·阿瓦达拉、瑞恩·W·怀特、道格·伯杰和王驰。2023年。[自动生成：通过多智能体对话推动下一代LLM应用](http://arxiv.org/abs/2308.08155)。

+   席等人（2023）席志恒、陈文翔、郭鑫、何伟、丁怡文、洪博扬、张鸣、王俊哲、金森杰、周恩宇、郑睿、范晓冉、王肖、熊立茂、周宇豪、王伟然、蒋昌浩、邹奕成、刘向阳、尹张跃、窦世涵、翁荣翔、程文森、张琪、秦文娟、郑永言、邱锡鹏、黄璇景和桂涛。2023年。[大型语言模型驱动的智能体崛起与潜力：一项调查](https://doi.org/https://doi.org/10.48550/arXiv.2309.07864)。

+   杨（2024）钱杨。2024年。[*苏格兰-中国学生在中国补习学校课堂中的语言使用：跨语言视角*](https://doi.org/https://doi.org/10.5525/gla.thesis.84440)。博士学位论文，格拉斯哥大学。

+   叶（2024）叶紫希。2024年。[跨文化交际中的语言障碍及其翻译策略]。收录于*国际金融与经济会议*，第6卷。

+   Yeh 和 Mitric (2023) 艾伦·耶和斯维特拉娜·米特里奇。2023年。[社交媒体与学习者作为民族志学者的方法：通过社区参与提高目标语言的参与度](https://doi.org/https://doi.org/10.1080/09588221.2021.2005630)。*计算机辅助语言学习*，36(8):1558–1586。

+   Yong 等人 (2023) 郑新永、张若晨、杰西卡·佐萨·福尔德、斯凯勒·王、阿尔君·苏布拉莫尼安、霍莉·洛维尼亚、塞缪尔·卡哈维贾雅、根塔·英德拉·维纳塔、林唐·苏塔维卡、简·克里斯蒂安·布莱兹·克鲁兹、尹琳·谭、龙·潘、罗维娜·加西亚、塔马尔·索洛里奥、阿尔哈姆·菲克里·阿吉。2023年。[促使多语言大型语言模型生成代码混合文本：以东南亚语言为例](https://doi.org/https://doi.org/10.48550/arXiv.2303.13592)。

+   Zheng 等人 (2023) 郑庆晓、许忠伟、阿宾纳夫·乔杜里、陈宇婷、李永明、黄云。2023年。[协同人类与人工智能代理：基于大型语言模型代理的服务共创23条启发式指南](https://doi.org/https://doi.org/10.48550/arXiv.2310.15065)。
