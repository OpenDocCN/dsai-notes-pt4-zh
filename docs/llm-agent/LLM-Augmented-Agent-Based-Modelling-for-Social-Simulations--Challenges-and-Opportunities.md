<!--yml

类别：未分类

日期：2025-01-11 12:39:05

-->

# LLM增强的基于智能体建模用于社会模拟：挑战与机遇

> 来源：[https://arxiv.org/html/2405.06700/](https://arxiv.org/html/2405.06700/)

Önder Gürcan [ongu@norceresearch.no](mailto:ongu@norceresearch.no) 挪威研究中心 AS（NORCE），

Universitetsveien 19, Kristiansand, Norway

###### 摘要

随着大型语言模型（LLMs）不断取得显著进展，它们更好地融入基于智能体的模拟中，为理解复杂社会系统提供了变革性的潜力。然而，这种集成并非易事，且面临众多挑战。基于这一观察，本文探讨了系统地开发LLM增强的社会模拟的架构和方法，并讨论了该领域潜在的研究方向。我们得出结论，将LLMs与基于智能体的模拟结合，为研究人员和科学家提供了强大的工具集，能够构建更细致、现实且全面的复杂系统和人类行为模型。

###### 关键词：

大型语言模型（LLMs）、基于智能体的模拟、社会系统建模、研究方向

## 1 引言

大型语言模型（LLMs）在最近已在多个研究领域和实际应用中迅速扩展其采用范围。这一迅速传播主要归因于其在理解、生成和翻译人类语言方面的卓越能力，且这种能力具有前所未有的准确性和流畅性。从医疗健康[[1](https://arxiv.org/html/2405.06700v1#bib.bib1), [2](https://arxiv.org/html/2405.06700v1#bib.bib2)]，它们在患者护理和医学研究中提供帮助，到金融[[3](https://arxiv.org/html/2405.06700v1#bib.bib3)]，用于分析市场趋势和自动化客户服务，许多行业已经利用LLMs的能力来提升效率和创新。此外，在学术领域，这些模型在分析大量数据集以获取洞察力方面起到了重要作用[[4](https://arxiv.org/html/2405.06700v1#bib.bib4)]，从而加速了社会科学、语言学和计算机科学等领域的研究成果。因此，LLMs的多功能性和不断发展的复杂性使其巩固了作为重塑行业和研究格局的基石技术的地位，推动了跨学科的新方法论和研究方法的出现。

在社会科学领域，LLMs可以有效应用的一个领域是社会模拟[[5](https://arxiv.org/html/2405.06700v1#bib.bib5)]。社会模拟用于建模和分析社会系统中复杂的互动，包括个体行为、群体动态、社会规范和制度结构等因素。这些模拟旨在理解、预测或检验社会系统中的假设情景。从这个意义上讲，社会模拟使用的一种技术是基于代理的建模（ABM）。ABM用于设计和模拟自主代理（个体、实体或组织）的行动和互动，以评估它们对整体系统的影响。ABM可以利用AI技术来增强模型的复杂性、适应性和真实性。迄今为止，ABM已经在各种社会模拟研究中实际融入了机器学习（ML）[[6](https://arxiv.org/html/2405.06700v1#bib.bib6)、[7](https://arxiv.org/html/2405.06700v1#bib.bib7)、[8](https://arxiv.org/html/2405.06700v1#bib.bib8)、[9](https://arxiv.org/html/2405.06700v1#bib.bib9)]、强化学习[[10](https://arxiv.org/html/2405.06700v1#bib.bib10)、[11](https://arxiv.org/html/2405.06700v1#bib.bib11)]和反向强化学习[[12](https://arxiv.org/html/2405.06700v1#bib.bib12)]等AI技术。然而，LLMs在帮助理解复杂社会系统方面的潜力尚未得到充分实现。最近很少有研究关注这一融合。尽管这些研究提供了一些实际解决方案，但它们缺乏一个明确的概念基线，而这个基线对于探索通用架构和方法论是必要的——最终扩展现有架构——以有效地、无缝地、系统地整合社会模拟和LLMs。

此外，LLMs可以简化并增强ABM过程中的各种其他方面，例如文献回顾、数据准备与解读、参数校准、敏感性分析和结果分析。我们已经开始看到LLMs在这些活动中的一些影响。但这些影响较少，且在社会模拟方面没有提供清晰的愿景。基于这些观察，本文概述了通过整合LLMs来增强基于代理的模拟的研究前景。我们对LLMs与ABM框架之间的共生潜力进行了结构化的探索，旨在推进方法论基础并增强社会模拟的分析能力。

本文其余部分安排如下。我们首先在第[2](https://arxiv.org/html/2405.06700v1#S2 "2 Background ‣ LLM-Augmented Agent-Based Modelling for Social Simulations: Challenges and Opportunities")节提供LLM和用于社会仿真的ABM背景知识。接下来，在第[3](https://arxiv.org/html/2405.06700v1#S3 "3 The Conceptual Baseline for LLM-Augmented Social Simulations ‣ LLM-Augmented Agent-Based Modelling for Social Simulations: Challenges and Opportunities")节评估构建多智能体系统的各种方法，以确定概念基准。然后，在第[4](https://arxiv.org/html/2405.06700v1#S4 "4 Research Directions ‣ LLM-Augmented Agent-Based Modelling for Social Simulations: Challenges and Opportunities")节，我们概述了开发这一思想的主要研究方向，最后在第[5](https://arxiv.org/html/2405.06700v1#S5 "5 Conclusions ‣ LLM-Augmented Agent-Based Modelling for Social Simulations: Challenges and Opportunities")节，我们对本文进行总结。

## 2 背景

### 2.1 大型语言模型（LLMs）

大型语言模型（LLM）可以定义为一个函数，它根据一系列符号（如单词、单词片段、标点符号、表情符号等），寻找并考虑最可能接下来的符号。截至目前，公众可以访问许多LLM。谷歌的Bard基于更高效且紧凑的PaLM 2 LLM，拥有3400亿个参数，可以通过网页浏览器¹¹1[https://bard.google.com/chat](https://bard.google.com/chat)免费访问，最后访问时间为2024年7月2日。以及API，具备多样的训练数据集[[13](https://arxiv.org/html/2405.06700v1#bib.bib13)]。Meta的LLaMA²²2[https://llama.meta.com](https://llama.meta.com)，最后访问时间为2024年7月2日，旨在推动人工智能研究的进展，提供多种模型，最多支持700亿个参数，供研究人员通过申请访问[[14](https://arxiv.org/html/2405.06700v1#bib.bib14)]。OpenAI的GPT模型利用变压器架构生成动态输出[[15](https://arxiv.org/html/2405.06700v1#bib.bib15)]，不同版本的访问方式不同：GPT-3.5可以通过网页界面³³3[https://chat.openai.com](https://chat.openai.com)免费访问，最后访问时间为2024年7月2日，而GPT-4则需要订阅，并且API的使用按使用量计费，基于自然语言处理中的符号化。LLM API是允许开发者在其应用程序中访问这些高级LLM功能的接口。API可以使用任何能够发出HTTP请求的编程语言集成到应用程序中，通常通过发送提示信息让LLM处理，并附带调整LLM行为的各种参数（如要使用的语言模型版本以及控制生成输出随机性的温度）。

LLM可以被视为一种非确定性模拟器，具有扮演无限多种角色的能力。本质上，LLM可以通过暴露于特定角色来进行微调，从而模拟类人互动。微调是通过在一个精心挑选的数据集上训练模型来实现的，这些数据集体现了角色所需的语言、知识和细微差别。这个过程涉及调整模型的参数，以使其更好地与目标角色特征的模式、词汇和决策过程对齐。在微调过程中，模型学会优先生成反映角色特定特征、专长或个性的响应。这通常是在模型已经在广泛的文本语料库上进行预训练后，使用一个较小、更专业化的数据集来完成的，从而使模型能够将其广泛的通用知识适应到更为狭窄的上下文和行为中。然而，这样的数据集也可以相对较大。

一个广为人知的架构，用于构建利用LLM在专业化、相对较大数据集上的系统，称为检索增强生成（RAG）[[16](https://arxiv.org/html/2405.06700v1#bib.bib16)]。在这种架构中，数据集被分解成若干块（例如几段文字或一页内容），然后将这些块发送给LLM，并将它们转换为向量。每个块都会有一个向量（即一系列数字），它是该块本质的数值表示。之后，每次在发送提示后，都会使用相同的LLM计算该提示的向量。接着，通过在块向量和提示向量之间进行数学比较，找到最接近的块。最后，这些块作为提示的一部分被使用。

### 2.2 社会模拟中的ABM

ABM作为一种关键技术，在通过计算模拟探索和理解社会系统中发挥着重要作用[[17](https://arxiv.org/html/2405.06700v1#bib.bib17)]。在此背景下，社会系统指的是个体、机构及其环境之间复杂的互动网络（这些环境可以包括个体行为、群体动力学、社会规范以及制度结构等因素）。社会系统的多面性使得个体能够同时扮演多个角色（例如在人类社会中充当父母、员工、消费者和公民等角色），每个角色都有其独特的期望和规范。

社会系统的ABM旨在通过模拟代理人的行为和互动来模仿社会过程，这些代理人代表了系统中的个体或实体，以预测和理解复杂现象。换句话说，ABM通过提供自下而上的建模方法来增强社会系统的研究，在这种方法中，宏观层次的现象源自代理人微观层次的互动。这一能力在社会科学中具有无价的价值，因为理解简单互动规则如何促成复杂社会现象的出现，可以为社会秩序的本质、规范和制度的演变以及社会变革的动态提供深刻的洞察。社会模拟提供了一个强大的工具，用于检验假设场景、测试社会行为和互动理论，并探索政策决策的潜在影响，而不受现实世界实验的伦理和实践限制。

然而，分析ABM的结果以理解社会系统面临着几个挑战，主要由于模型本身及其所试图代表的系统的复杂性和动态性。首先，ABM通常通过模拟众多代理人随时间的互动，生成大量数据，这使得很难辨识出明确的模式或得出简单的结论。ABM的一个特点是涌现现象，尽管这一现象对于理解微观行为的宏观结果具有价值，但这些结果往往不可预测或不线性，从而使得分析变得复杂。其次，以对政策制定或理论进展有意义的方式解读ABM的结果，需要弥合复杂的、通常是技术性的模型输出与社会科学概念框架之间的差距。这通常需要跨学科的合作，以确保所生成的见解既具有科学严谨性，又具有社会相关性。

此外，除了ABM，社会科学家还采用各种其他方法来研究社会系统。调查和问卷用于收集关于个体态度、行为和经验的大规模数据。访谈和民族志提供了对社会现象的深入定性洞察，捕捉人类行为和社会互动的细微差别。案例研究提供了对特定事件或实例的详细分析，允许深入理解复杂的社会过程。实验设计和准实验设计用于建立变量之间的因果关系。统计分析和计算技术，包括网络分析和数据挖掘，用于分析和建模大数据集。最后，内容和话语分析用于研究社会背景中的沟通模式和意义的构建。这些方法中的一些已经在ABM研究中得到了应用。

## 3 LLM增强社会模拟的概念基线

将大语言模型（LLMs）与代理基础模型（ABMs）相结合，用于社会模拟，具有变革性的潜力，可以帮助我们理解复杂的社会系统。然而，这种整合需要一个概念性基础，以便连接这两个领域。概念性基础是一个清晰且连贯的基础框架，概述了给定系统或模型中的关键概念、变量、假设和关系。它作为理解所研究系统动态和行为的参考点，随后用于模型开发和分析。这样的概念性基础可以通过现有的专门用于多代理系统（MAS）的工程方法来建立。MAS通常从四个主要角度来观察：代理、互动、环境和组织。

以组织为导向的MAS方法特别适合用于建模社会系统，并集成LLMs，因为它强调在复杂系统内的结构化互动和角色分配。这种方法反映了社会系统的层级性和网络化特征，其中实体（代理）承担特定的角色和责任，这些角色和责任由已建立的规范和政策所规范。这种组织结构使得能够清晰地定义LLMs的角色，便于将其作为具有特定功能（如语言理解、生成和处理）的代理进行集成。这不仅增强了系统模仿人类社会结构的能力，而且还利用LLMs在自然语言处理中的优势，改善了代理之间的沟通与协调。通过将MAS的结构性和功能性方面与社会系统的固有特性及LLMs的优势对接，面向组织的方法为捕捉社会互动的复杂性和动态性提供了一个强有力的框架，使其成为这些应用的更佳选择。因此，我们认为，增强LLM的高效代理基础模型（ABM）工具应当支持以组织为导向的概念基础。

从这个角度来看，我们建议将社会仿真中的代理定义为扮演一个或多个预定义角色的社会代理[[33](https://arxiv.org/html/2405.06700v1#bib.bib33), [34](https://arxiv.org/html/2405.06700v1#bib.bib34), [35](https://arxiv.org/html/2405.06700v1#bib.bib35)]。社会代理的技能是通过与环境的互动，扮演角色的能力，这些技能在一个由其他扮演者共同组成的社区中得到运用，他们也栖息在这些环境中。  

## 4 研究方向  

在本节中，我们探讨了几条关键的研究路径，这些路径对于借助大型语言模型（LLMs）转变社会仿真似乎具有重要意义。  

### 4.1 文献综述  

科学文献的数量庞大[[36](https://arxiv.org/html/2405.06700v1#bib.bib36), [37](https://arxiv.org/html/2405.06700v1#bib.bib37)]，且不同出版物的质量参差不齐。因此，需要一些工具来客观高效地搜索、评估和总结科学文献。大型语言模型（LLMs）通过其高效处理大量文本数据的先进能力，可以显著应对传统评审过程中的挑战[[37](https://arxiv.org/html/2405.06700v1#bib.bib37), [38](https://arxiv.org/html/2405.06700v1#bib.bib38), [39](https://arxiv.org/html/2405.06700v1#bib.bib39)]，并且它们不太可能挑选文献来支持某些假设（即减少研究人员偏见）[[40](https://arxiv.org/html/2405.06700v1#bib.bib40)]。通过自动化文献的初步筛选和总结，LLMs可以帮助研究人员应对信息过载问题，使他们能够快速识别相关研究，而不会妥协于回顾的广度或深度。它们在多种语言下分析和总结文本的能力，也可以克服语言障碍，从而提供更广泛的文献访问。

### 4.2 建模架构

如[3](https://arxiv.org/html/2405.06700v1#S3 "3 LLM增强型社会模拟的概念基础 ‣ LLM增强型基于代理的社会模拟：挑战与机遇")节所述，将代理定义为角色扮演的参与者的组织导向方法为建模社会代理提供了坚实的基础。然而，现存有多种组织导向的架构，LLM增强型社会模拟的适配性尚未得到研究。此外，还需要对社会代理角色的有效设计和重用进行研究。角色的有效设计对于获得有效的见解至关重要（参见[4.5](https://arxiv.org/html/2405.06700v1#S4.SS5 "4.5 获得见解 ‣ 4 研究方向 ‣ LLM增强型基于代理的社会模拟：挑战与机遇")节），而角色的有效重用对于大型且可重复的社会模拟是必要的。此外，LLMs还可以通过自然语言和相关定性数据生成基于代理的模型和场景。

### 4.3 数据准备

社会模拟的数据收集既耗时又昂贵，因为它面临若干关键挑战，包括捕捉社会系统的复杂性、确保数据的高质量和相关性、解决伦理和隐私问题、整合多样化的数据源以及实现准确的建模和模拟。此外，模型的校准与验证、计算限制的管理、跨学科协作的促进以及适应动态社会系统的要求也增加了复杂性。最后，确保模型能在不同的情境或人群中具有普适性和可迁移性是一个重大挑战，需要谨慎、系统的方法，并通常需要跨学科的协作。LLM有潜力显著提升社会模拟的数据收集过程，通过提供解决许多上述挑战的方案。在捕捉社会系统复杂性方面，LLM能够处理并分析来自多种来源的大量文本数据[[41](https://arxiv.org/html/2405.06700v1#bib.bib41), [42](https://arxiv.org/html/2405.06700v1#bib.bib42)]，提供对社会动态的细致理解，从而支持更准确和全面的模型构建。它们先进的自然语言处理能力使得能够整合各种类型的数据，从结构化数据到非结构化文本，促进了更丰富、多维数据集的创建。

### 4.4 数据化

数据化是将复杂的社会互动和现象转化为可量化数据的过程，从而实现实时跟踪和预测分析[[43](https://arxiv.org/html/2405.06700v1#bib.bib43)]。LLM增强型社会代理可以在数据化过程中发挥关键作用。之后，这些数据可以被分析，以揭示关于社会行为和系统的模式、趋势和洞察。例如，社会代理在模拟中进行互动时，它们会不断生成数据，反映随时间变化的情况，包括社会系统如何响应外部压力或内部动态而发生演变。数据生成的这种动态特性对于研究社会变革、创新扩散和社会规范的形成至关重要。

### 4.5 获取洞察

社会模拟通常会产生过于庞大和复杂的数据，难以进行整理和分析。研究表明，使用LLM可以从数据中获得洞察[[44](https://arxiv.org/html/2405.06700v1#bib.bib44)]。因此，可以通过与这些社交代理对话来获得洞察。如果准备得当，完全可以模拟出一个由大量社会代理组成的合成群体，代表不同的人类经验和视角[[45](https://arxiv.org/html/2405.06700v1#bib.bib45)]，这能够比传统方法更精确地描绘人类行为和社会动态[[5](https://arxiv.org/html/2405.06700v1#bib.bib5)]。通过适当的条件设定[[46](https://arxiv.org/html/2405.06700v1#bib.bib46)]，社交代理可以扮演具有信念和意图的角色，并为用户提供准确和客观的回答[[47](https://arxiv.org/html/2405.06700v1#bib.bib47)]。它们非常适合获得洞察，因为它们“可以在不疲劳的情况下快速回答数百个问题”，且“比人类更少的激励就能给出可靠的回答”[[48](https://arxiv.org/html/2405.06700v1#bib.bib48)]。近期研究表明，LLM能够做出与人类判断高度一致的判断[[48](https://arxiv.org/html/2405.06700v1#bib.bib48)]。从各种对话中收集到的信息可以被整理为可量化的数据，然后通过统计分析来提供对更广泛社会趋势和模式的洞察。这种方法随后可以用来生成假设并在现实社会中验证这些假设[[46](https://arxiv.org/html/2405.06700v1#bib.bib46), [35](https://arxiv.org/html/2405.06700v1#bib.bib35)]。

### 4.6 可解释性

LLM增强的社交代理可以为其行为、决策以及模拟背后的机制生成自然语言的解释[[49](https://arxiv.org/html/2405.06700v1#bib.bib49), [50](https://arxiv.org/html/2405.06700v1#bib.bib50)]。这可以使得这些代理的行为对研究人员（来自各个学科的人员，而不仅仅是计算机背景的人员）、利益相关者以及公众更加易于理解，将复杂的算法和决策过程转化为易于消化的解释。此外，利用LLM中嵌入的关于社会动态的广泛知识和理解，这些增强的代理可以为其行为提供上下文洞察。例如，它们可以解释某些社会规范、历史事件或文化因素如何影响它们在模拟中的行为，从而提供对所建模社会现象更深刻的理解。

### 4.7 平台与工具

有效的LLM增强型社会仿真需要复杂的工具支持，专门设计以应对动态社会互动和数据化过程的复杂性。这些工具应促进LLM与仿真平台的无缝集成，提供便于配置、实时调整和伦理数据处理的功能。重要的是，这些工具的设计必须基于面向组织的概念基线（见第[3](https://arxiv.org/html/2405.06700v1#S3 "3 LLM增强型社会仿真的概念基线 ‣ LLM增强型基于代理的社会仿真：挑战与机遇")节），确保它们与社会系统的结构性动态相一致，并支持聚焦的跨学科研究。这种方法提高了仿真的可访问性、准确性和伦理合规性，使研究人员能够深入探索和理解社会现象。

## 5 结论

大型语言模型（LLMs）为社会系统的仿真与分析提供了一个变革性的框架。在本研究中，我们提出了不同的LLM干预措施，涵盖了整个社会仿真流程。除了推理相关主体的行为外，LLMs还可以促进用户与仿真主体之间更繁荣且直观的互动。这使得社会仿真变得更加可访问和用户友好，允许来自各个学科的个人，包括社会科学、医疗保健、城市规划和环境研究等领域的人员使用它们。这样的民主化可以使这些专业人士在不需要计算机科学深厚专长的情况下，建模和分析与其领域相关的复杂系统。因此，我们可以预期计算机科学家与其他领域专家之间的合作将增加，从而推动更多的跨学科方法和创新解决方案来应对复杂的现实问题。

大型语言模型（LLMs）的潜在优势值得认真考虑。然而，参与LLM增强型基于代理的建模（ABM）工具的科学家和开发人员也必须考虑到，这些工具在某些情况下可能会阻碍而非推动科学知识的进展。这意味着，虽然LLMs提供了显著的认知利益，但如果科学家将其视为知识生产的合作伙伴，它们也可能带来认知风险[[51](https://arxiv.org/html/2405.06700v1#bib.bib51)]。因为，将LLM增强型ABM工具视为科学研究中的合作者，可能使科学家面临陷入理解错觉的风险，这是一种元认知错误，指个体对自己理解的深度或准确性存在错误的信念[[52](https://arxiv.org/html/2405.06700v1#bib.bib52)，[53](https://arxiv.org/html/2405.06700v1#bib.bib53)]。

## 致谢

这里报告的工作是URBANE项目的一部分，该项目获得了欧盟“地平线欧洲创新行动”资助，资助协议编号为101069782。

## 参考文献

+   [1] Clusmann J, Kolbinger FR, Muti HS, Carrero ZI, Eckardt JN, Laleh NG, 等人. 大型语言模型在医学中的未来前景. Communications Medicine;3(1):141. 可从：[https://doi.org/10.1038/s43856-023-00370-1](https://doi.org/10.1038/s43856-023-00370-1) 获取。

+   [2] Barrington NM, Gupta N, Musmar B, Doyle D, Panico N, Godbole N, 等人. ChatGPT在医学研究中的崛起：一项文献计量分析. Medical Sciences. 2023;11(3). 可从：[https://www.mdpi.com/2076-3271/11/3/61](https://www.mdpi.com/2076-3271/11/3/61) 获取。

+   [3] Wu S, Irsoy O, Lu S, Dabravolski V, Dredze M, Gehrmann S, 等人. BloombergGPT：金融领域的大型语言模型; 2023年。

+   [4] Taylor R, Kardas M, Cucurull G, Scialom T, Hartshorn A, Saravia E, 等人. Galactica：科学领域的大型语言模型; 2022年。

+   [5] Grossmann I, Feinberg M, Parker DC, Christakis NA, Tetlock PE, Cunningham WA. 人工智能与社会科学研究的转型. Science. 2023;380(6650):1108-9. 可从：[https://www.science.org/doi/abs/10.1126/science.adi1778](https://www.science.org/doi/abs/10.1126/science.adi1778) 获取。

+   [6] Turgut Y, Bozdag CE. 通过案例研究分析提出一个面向机器学习驱动的代理模型框架. Simulation Modelling Practice and Theory. 2023;123:102707. 可从：[https://www.sciencedirect.com/science/article/pii/S1569190X22001769](https://www.sciencedirect.com/science/article/pii/S1569190X22001769) 获取。

+   [7] Ale Ebrahim Dehkordi M, Lechner J, Ghorbani A, Nikolic I, Chappin E, Herder P. 在基于代理的模型和仿真中使用机器学习进行代理规范：一项批判性综述和指南. Journal of Artificial Societies and Social Simulation. 2023;26(1):9. 可从：[http://jasss.soc.surrey.ac.uk/26/1/9.html](http://jasss.soc.surrey.ac.uk/26/1/9.html) 获取。

+   [8] Chen SH, Londoño-Larrea P, McGough AS, Bible AN, Gunaratne C, Araujo-Granda PA, 等人. 将机器学习技术应用于拟态模型的潘多巴. Frontiers in Microbiology. 2021;12. 可从：[https://www.frontiersin.org/articles/10.3389/fmicb.2021.726409](https://www.frontiersin.org/articles/10.3389/fmicb.2021.726409) 获取。

+   [9] R Vahdati A, Weissmann JD, Timmermann A, Ponce de León MS, Zollikofer CPE. 晚更新世人类生存与迁徙的驱动因素：一种基于代理的建模与机器学习方法. Quaternary Science Reviews. 2019;221:105867. 可从：[https://www.sciencedirect.com/science/article/pii/S0277379119307048](https://www.sciencedirect.com/science/article/pii/S0277379119307048) 获取。

+   [10] Chmura T, Pitz T. 一种扩展的强化学习算法，用于实验性拥塞博弈中人类行为的估计。《人工社会与社会模拟期刊》。2007年；10(2)。可从以下网址获取：[http://jasss.soc.surrey.ac.uk/10/2/1.html](http://jasss.soc.surrey.ac.uk/10/2/1.html)。

+   [11] Chen W, Liu H, Xu D. 竞争性多代理零售市场中易腐商品的动态定价策略。《人工社会与社会模拟期刊》。2018年；21(2)：12。可从以下网址获取：[http://jasss.soc.surrey.ac.uk/21/2/12.html](http://jasss.soc.surrey.ac.uk/21/2/12.html)。

+   [12] Lee K, Ulkuatam S, Beling P, Scherer W. 通过逆强化学习和基于代理的建模生成合成比特币交易并预测市场价格波动。《人工社会与社会模拟期刊》。2018年；21(3)：5。可从以下网址获取：[http://jasss.soc.surrey.ac.uk/21/3/5.html](http://jasss.soc.surrey.ac.uk/21/3/5.html)。

+   [13] Anil R, Dai AM, Firat O, Johnson M, Lepikhin D, Passos A 等。PaLM 2技术报告；2023年。

+   [14] Touvron H, Lavril T, Izacard G, Martinet X, Lachaux MA, Lacroix T 等。LLaMA：开放且高效的基础语言模型；2023年。

+   [15] Vaswani A, Shazeer N, Parmar N, Uszkoreit J, Jones L, Gomez AN 等。注意力即一切。载于：Guyon I, Luxburg UV, Bengio S, Wallach H, Fergus R, Vishwanathan S 等，编辑。《神经信息处理系统进展》。第30卷。Curran Associates, Inc.；2017年。

+   [16] Lewis P, Perez E, Piktus A, Petroni F, Karpukhin V, Goyal N 等。增强检索生成用于知识密集型自然语言处理任务；2021年。

+   [17] Macal CM. 你需要了解的一切关于基于代理的建模与仿真。《仿真期刊》。2016年；10：144-156。

+   [18] Shoham Y. 面向代理的编程。《人工智能》。1993年；60(1)：51-92。可从以下网址获取：[https://www.sciencedirect.com/science/article/pii/0004370293900349](https://www.sciencedirect.com/science/article/pii/0004370293900349)。

+   [19] Cuesta P, Gómez A, González JC. 载于：Moreno A, Pavón J, 编辑。面向代理的软件工程。巴塞尔：Birkhäuser Basel；2008年。第1-31页。可从以下网址获取：[https://doi.org/10.1007/978-3-7643-8543-9_1](https://doi.org/10.1007/978-3-7643-8543-9_1)。

+   [20] Abdalla R, Mishra A. 面向代理的软件工程方法论：分析与未来方向。《复杂性》；2021年：1629419。可从以下网址获取：[https://doi.org/10.1155/2021/1629419](https://doi.org/10.1155/2021/1629419)。

+   [21] Chopra AK, Christie SH, Singh MP. 互动导向编程：智能、基于意义的多代理系统。载于：2023年国际自主代理与多代理系统会议论文集；2023年。第3041-3043页。

+   [22] Singh MP. 信息驱动的交互导向编程：BSPL，极其简单的协议语言。在：第10届国际自主代理与多代理系统会议 - 第2卷。AAMAS ’11。南卡罗来纳州里士满：国际自主代理与多代理系统基金会；2011年。第491-498页。

+   [23] Muller G. 交互导向编程。在：第七届IEEE国际工作坊，启用技术：协作企业基础设施（WET ICE ’98）（会议编号98TB100253）；1998年。第138-142页。

+   [24] Ricci A, Piunti M, Viroli M. 多代理系统中的环境编程：基于人工制品的视角。《自主代理与多代理系统》；23(2)：158-192。可从以下链接获取：[https://doi.org/10.1007/s10458-010-9140-7](https://doi.org/10.1007/s10458-010-9140-7)。

+   [25] Weyns D, Omicini A, Odell J. 作为一级抽象的环境在多代理系统中的应用。《自主代理与多代理系统》。2006年；14(1)：5-30。

+   [26] Abbas HA. 实现NOSHAPE MAS组织模型：一种操作视角。《国际代理技术系统期刊》。2015年4月；7(2)：75-104。

+   [27] Criado N, Argente E, Botti V. THOMAS：支持规范多代理系统的代理平台。《逻辑与计算期刊》。2013年；23(2)：309-333。

+   [28] Giorgini P, Kolp M, Mylopoulos J. 作为组织结构的多代理架构。《自主代理与多代理系统》。2006年；13：3-25。

+   [29] Dignum V, Vázquez-Salceda J, Dignum F. OMNI: 将社会结构、规范和本体引入代理组织。在：Bordini RH, Dastani M, Dix J, El Fallah Seghrouchni A, 编辑。编程多代理系统。柏林，海德堡：Springer Berlin Heidelberg; 2005年。第181-198页。

+   [30] Ferber J, Gutknecht O, Michel F. 从代理到组织：多代理系统的组织视角。在：Giorgini P, Müller JP, Odell J, 编辑。面向代理的软件工程IV。柏林，海德堡：Springer Berlin Heidelberg；2004年。第214-230页。

+   [31] Hübner JF, Sichman JSa, Boissier O. MOISE+：迈向多代理系统组织的结构性、功能性和义务模型。在：第一届国际自主代理与多代理系统联合会议论文集：第一部分。AAMAS’02。美国纽约：计算机学会；2002年。第501-502页。

+   [32] Roussille H, Gürcan O, Michel F. AGR4BS：一种通用的多代理组织模型，用于区块链系统。《大数据与认知计算》。2022年；6(1)：41页。可从以下链接获取：[https://www.mdpi.com/2504-2289/6/1/1](https://www.mdpi.com/2504-2289/6/1/1)。

+   [33] Shanahan M, McDonell K, Reynolds L. 使用大型语言模型进行角色扮演。《自然》；623(7987)：493-498。可从以下链接获取：[https://doi.org/10.1038/s41586-023-06647-8](https://doi.org/10.1038/s41586-023-06647-8)。

+   [34] Andreas J. 语言模型作为代理模型。在：Goldberg Y，Kozareva Z，Zhang Y，编辑。计算语言学协会会议成果：EMNLP 2022。阿布扎比，阿联酋：计算语言学协会；2022年。第5769-79页。可从：[https://aclanthology.org/2022.findings-emnlp.423](https://aclanthology.org/2022.findings-emnlp.423)获取。

+   [35] Park JS，O’Brien JC，Cai CJ，Morris MR，Liang P，Bernstein MS。生成型代理：人类行为的互动模拟；2023年。

+   [36] Hutson M。人工智能工具旨在驾驭冠状病毒文献。《自然》杂志。2020年。可从：[https://doi.org/10.1038/d41586-020-01733-7](https://doi.org/10.1038/d41586-020-01733-7)获取。

+   [37] Wagner G，Lukyanenko R，Paré G。人工智能与文献综述的开展。《信息技术杂志》。2022年；37（2）：209-226。可从：[https://doi.org/10.1177/02683962211048201](https://doi.org/10.1177/02683962211048201)获取。

+   [38] Dunn A，Dagdelen J，Walker N，Lee S，Rosen AS，Ceder G 等人。通过微调的大型语言模型从复杂的科学文本中提取结构化信息；2022年。

+   [39] Zhang T，Ladhak F，Durmus E，Liang P，McKeown K，Hashimoto TB。基准测试大型语言模型在新闻摘要中的表现；2023年。

+   [40] Müller H，Pachnanda S，Pahl F，Rosenqvist C。人工智能在不同类型文献综述中的应用——一项比较研究。在：2022年国际应用人工智能大会（ICAPAI）；2022年。第1-7页。

+   [41] Zhang L，Chen J，Zou J。城市建筑能效建模中的开放数据分类、语义数据架构及架构对齐；2023年。

+   [42] Semeler A，Pinto A，Koltay T，Dias T，Oliveira A，Gonzâlez J 等人。《算法素养：面向数据馆员的生成型人工智能技术》。EAI认可的可扩展信息系统学报。2024年1月；11（2）。

+   [43] Mayer-Schnberger V，Cukier K。《大数据：一场将改变我们生活、工作和思维的革命》。Eamon Dolan/Houghton Mifflin Harcourt；2014年。

+   [44] McCloskey BJ，LaCasse PM，Cox BA。基于自然语言处理的在线评论分析：从小型语料库中提取洞见。《运筹学年鉴》。可从：[https://doi.org/10.1007/s10479-023-05816-2](https://doi.org/10.1007/s10479-023-05816-2)获取。

+   [45] Fuchs A，Passarella A，Conti M。建模、复制与预测人类行为：一项调查。《ACM自动化与自适应系统学报》。2023年5月；18（2）。可从：[https://doi.org/10.1145/3580492](https://doi.org/10.1145/3580492)获取。

+   [46] Argyle LP，Busby EC，Fulda N，Gubler JR，Rytting C，Wingate D。由一而多：使用语言模型模拟人类样本。《政治分析》。2023年；31（3）：337-351。

+   [47] Mei Q, Xie Y, Yuan W, Jackson MO. 一项图灵测试：AI 聊天机器人是否在行为上与人类相似。美国国家科学院学报。2024;121(9):e2313925121. 可从以下网址获取：[https://www.pnas.org/doi/abs/10.1073/pnas.2313925121](https://www.pnas.org/doi/abs/10.1073/pnas.2313925121)。

+   [48] Dillion D, Tandon N, Gu Y, Gray K. AI 语言模型能取代人类参与者吗？认知科学趋势。2023;27(7):597-600. 可从以下网址获取：[https://www.sciencedirect.com/science/article/pii/S1364661323000980](https://www.sciencedirect.com/science/article/pii/S1364661323000980)。

+   [49] Peterson JC, Bourgin DD, Agrawal M, Reichman D, Griffiths TL. 利用大规模实验和机器学习发现人类决策理论。科学。2021;372(6547):1209-14. 可从以下网址获取：[https://www.science.org/doi/abs/10.1126/science.abe2629](https://www.science.org/doi/abs/10.1126/science.abe2629)。

+   [50] Gil Y. 深思熟虑的人工智能：为数据科学和科学发现铸造新的伙伴关系。数据科学。2017;1:119-29. 1-2. 可从以下网址获取：[https://doi.org/10.3233/DS-170011](https://doi.org/10.3233/DS-170011)。

+   [51] Messeri L, Crockett MJ. 人工智能与科学研究中的理解错觉。自然。2024年3月;627(8002):49-58. 可从以下网址获取：[https://doi.org/10.1038/s41586-024-07146-0](https://doi.org/10.1038/s41586-024-07146-0)。

+   [52] Rozenblit L, Keil F. 误解的民间科学局限性：一种解释深度的错觉。认知科学。2002;26(5):521-62. 可从以下网址获取：[https://onlinelibrary.wiley.com/doi/abs/10.1207/s15516709cog2605_1](https://onlinelibrary.wiley.com/doi/abs/10.1207/s15516709cog2605_1)。

+   [53] Rabb N, Fernbach PM, Sloman SA. 知识共同体中的个体表征。认知科学趋势。2019;23(10):891-902. 可从以下网址获取：[https://www.sciencedirect.com/science/article/pii/S1364661319301998](https://www.sciencedirect.com/science/article/pii/S1364661319301998)。
