<!--yml

类别：未分类

日期：2025-01-11 12:33:57

-->

# 领域特定的ReAct用于物理集成的迭代建模：基于LLM代理的燃气轮机气路分析案例研究

> 来源：[https://arxiv.org/html/2406.07572/](https://arxiv.org/html/2406.07572/)

宋涛

能源与动力工程系

清华大学

北京，中国

songtao21@mails.tsinghua.edu.cn

&范宇伟

能源与动力工程系

清华大学

北京，中国

fan-yw22@mails.tsinghua.edu.cn

&冯陈龙

能源与动力工程系

清华大学

北京，中国

fengcl23@mails.tsinghua.edu.cn

&宋凯宇

能源与动力工程系

清华大学

北京，中国

sky20@mails.tsinghua.edu.cn

&刘超

能源与动力工程系

清华大学

北京，中国

cliu5@tsinghua.edu.cn

&蒋东祥

能源与动力工程系

清华大学

北京，中国

jiangdx@tsinghua.edu.cn 通讯作者。

###### 摘要

本研究探讨了在能源与动力工程领域中，带有可调用工具的大型语言模型（LLMs）的应用，重点研究了燃气轮机的气路分析。我们开发了一种双代理工具调用流程，旨在整合专家知识、预定义工具和LLM推理。我们评估了多种LLM，包括LLama3、Qwen1.5和GPT。较小的模型在工具使用和参数提取上存在困难，而较大的模型展现了较好的能力。所有模型在处理复杂的多组分问题时都面临挑战。基于测试结果，我们推测，具有近1000亿参数的LLM经过微调和高级提示设计后，可以满足专业场景的要求。持续的开发有可能提高其准确性和有效性，为更强大的AI驱动解决方案铺平道路。

*K*eywords 大型语言模型  $\cdot$ ReAct 提示 $\cdot$ 燃气轮机

## 1 引言

人工智能（AI）已经在多个领域得到了广泛应用，包括能源与电力工程，在这些领域，AI在提升效率、可靠性和可持续性方面具有重要前景。在数值计算领域，AI技术已被有效应用于部分取代传统数值方法，涵盖了广泛的能源系统，帮助设计和优化关键过程，如结构强度和热传递[[1](https://arxiv.org/html/2406.07572v1#bib.bib1), [2](https://arxiv.org/html/2406.07572v1#bib.bib2), [3](https://arxiv.org/html/2406.07572v1#bib.bib3)]。同样，在电力设备信号分析和故障诊断中，AI算法促进了异常检测和预测性维护，从而减少了停机时间并提高了操作效率[[4](https://arxiv.org/html/2406.07572v1#bib.bib4), [5](https://arxiv.org/html/2406.07572v1#bib.bib5), [6](https://arxiv.org/html/2406.07572v1#bib.bib6)]。尽管取得了这些成功，传统AI工具通常缺乏必要的理论基础，难以全面解决能源系统的复杂性。许多AI模型的黑箱性质限制了它们的可解释性，导致人们对其在关键能源基础设施应用中的可靠性和问责性产生担忧。

最近，大型语言模型（LLMs）的出现，如GPT[[7](https://arxiv.org/html/2406.07572v1#bib.bib7)]、LLaMA[[8](https://arxiv.org/html/2406.07572v1#bib.bib8)]和GLM[[9](https://arxiv.org/html/2406.07572v1#bib.bib9)]，彻底改变了人工智能，特别是在自然语言处理（NLP）领域。这些模型在生成类人文本、理解上下文和执行复杂任务方面表现出色。与此同时，增强方法如检索增强生成（RAG）[[10](https://arxiv.org/html/2406.07572v1#bib.bib10)]、带工具的代理[[11](https://arxiv.org/html/2406.07572v1#bib.bib11)]，以及像Chain-of-Thought[[12](https://arxiv.org/html/2406.07572v1#bib.bib12)]和Tree-of-Thoughts[[13](https://arxiv.org/html/2406.07572v1#bib.bib13)]这样的先进提示策略也已被提出。这些进展解决了LLMs在幻觉和不可解释性方面的一些问题，使其在实际应用中更加稳健和可靠。

在这个背景下，我们探讨了将大型语言模型（LLMs）集成到特定领域应用场景中的可行性，即能源与电力科学与工程，重点关注燃气轮机操作与维护中的经典问题：气路分析。我们的方法包括设计特定的提示词和双代理工具调用过程，以引导LLMs解析工具参数。我们将能量方程定义为每个组件的工具，并采用ReAct链式思维提示[[14](https://arxiv.org/html/2406.07572v1#bib.bib14)]来引导LLM推理并解决性能参数。对不同LLM的测试为它们在理解、思考和推理方面的能力提供了深入见解，为进一步研究和应用提供了宝贵的视角。

## 2 背景

### 2.1 大型语言模型

大型语言模型（LLMs）代表了人工智能领域的重要进展，特别是在自然语言处理领域，它依赖神经网络和变换器架构进行文本处理与生成。神经网络是由互联神经元组成的复杂结构，能够通过学习到的权重高效地转换输入数据，从而实现多维度的表示。LLM的一项重大进展是注意力机制的引入，它对输入数据中某些特定元素的识别至关重要，能够捕捉广泛的上下文依赖关系。LLM的核心架构是变换器架构，它由编码器和解码器层组成，前者处理输入文本，后者生成输出，并通过自注意力机制评估词汇的重要性和上下文细微差异。除了基础架构，LLMs还具有明显的优势，超越了传统的人工智能方法，包括生成连贯文本的能力以及在推理任务中的熟练度，这些都得益于其大规模的训练数据。诸如注意力可视化等技术增强了解释性，有助于揭示模型的底层处理机制。

近年来，LLM的应用日益增多，每种模型都有独特的架构差异、训练数据集和方法论，导致它们在自然语言理解和逻辑推理能力上的显著差异。我们的研究评估了来自不同机构的多种大型语言模型，阐明了它们在解析提示词和执行基本热力学工程问题的逻辑推理任务中的细微差别。

### 2.2 LLM代理与ReAct提示

大型语言模型的输出是对人类语言的概率估计，固有地会受到错误和潜在不准确性的影响。这些模型可能表现出幻觉倾向，特别是在面对超出其训练数据的场景时，这为准确生成回应带来了挑战。复杂的任务通常超出了大型语言模型单次回应的能力。因此，将这些模型与记忆、规划和基于工具的能力相结合，成为解决复杂问题的可行策略，既能发挥其优势，又能减轻固有的局限性。

具有可调用工具并受ReAct（推理与行动）策略指导的大型语言模型代理，构成了解决复杂推理任务的先进框架。ReAct提示有助于系统化问题分解，通过提供结构化的推理路径来解决大型语言模型常见问题，如幻觉和无法解释性。集成的可调用工具作为领域特定的功能，封装了专家知识和计算方法。在我们的实验中，结合热力学原理的燃气轮机组件解决过程，指导大型语言模型遵循科学定律。通过将大型语言模型在语言理解和上下文处理上的优势与人类定义的工具相结合，这些代理能够在解决问题时实现确定性与适应性的平衡。这种方法增强了大型语言模型代理的可靠性和可解释性，满足了如能源和电力工程等领域对复杂、依赖于具体背景的挑战的细致需求。

### 2.3 燃气轮机组件和测试数据

燃气轮机通过布雷顿循环运行，利用压缩、加热和膨胀阶段将气体燃料高效地转化为机械能。其组件，包括压缩机、燃烧室、涡轮和喷嘴，在这一热力学过程中发挥着至关重要的作用。压缩机通过绝热压缩提高空气压力，这是整体涡轮性能的关键。在燃烧室中，燃料与空气的混合物燃烧显著提高了温度和压力，而涡轮则从产生的气流中提取能量，用于机械工作。喷嘴加速排气气体，提升涡轮效率。

计算压缩机和涡轮机的绝热效率涉及在绝热假设下确定这些组件的焓变化，并将实际焓变化与理论焓变化进行比较，以衡量其性能。测试条件通常测量温度、压力和燃料流量，而复杂性通常排除了如燃烧室出口（涡轮入口）和空气/气体流量等参数。因此，计算涡轮的等熵效率需要解决多个组件：确定通过喷嘴的气体流量，利用燃烧室中的焓守恒计算涡轮入口的温度和压力，最后计算涡轮的等熵效率。此过程如图[1](https://arxiv.org/html/2406.07572v1#S2.F1 "Figure 1 ‣ 2.3 Gas turbine components and test data ‣ 2 Background ‣ Domain-specific ReAct for physics-integrated iterative modeling: A case study of LLM agents for gas path analysis of gas turbines")所示。本文旨在使大型语言模型能够理解并执行这一计算过程。

\includestandalone

[width=0.98]gas-turb-calc

图1：燃气轮机气路分析框架

## 3 方法

### 3.1 大型语言模型（LLMs）

鉴于不同大型语言模型的能力差异，我们从多个机构选择了模型，以便计算燃气轮机的性能参数，并评估其在能源和电力工程中的应用潜力。这些模型的详细信息列在表格[1](https://arxiv.org/html/2406.07572v1#S3.T1 "Table 1 ‣ 3.1 LLMs models ‣ 3 Methods ‣ Domain-specific ReAct for physics-integrated iterative modeling: A case study of LLM agents for gas path analysis of gas turbines")中。

表1：实验中使用的LLMs

| 模型 | 参数数量 | 模型创建者 | 发表日期 | 访问方式 | 本地工具支持 | 量化方法 |
| --- | --- | --- | --- | --- | --- | --- |
| LLama3[[8](https://arxiv.org/html/2406.07572v1#bib.bib8)] | 8B | Meta | 2024年4月18日 | 本地部署 | 否 | FP16 |
| LLama3 | 70B | Meta | 2024年4月18日 | 本地部署 | 否 | 4Bit |
| Qwen1.5[[15](https://arxiv.org/html/2406.07572v1#bib.bib15)] | 32B | 阿里巴巴 | 2024年2月5日 | 本地部署 | 否 | 4Bit |
| Qwen1.5 | 72B | 阿里巴巴 | 2024年2月5日 | 本地部署 | 否 | 4Bit |
| GPT3.5[[7](https://arxiv.org/html/2406.07572v1#bib.bib7)] | - | OpenAI | 2022年11月30日 | API | 是 | - |
| GPT4o | - | OpenAI | 2024年5月13日 | Web | 是 | - |
| GLM4[[9](https://arxiv.org/html/2406.07572v1#bib.bib9)] | - | 智谱AI | 2024年1月16日 | Web & API | 是 | - |

由于能源和电力工程行业数据的商业敏感性，我们专注于测试可以在消费者级显卡上本地执行的多个大型语言模型。表格[1](https://arxiv.org/html/2406.07572v1#S3.T1 "Table 1 ‣ 3.1 LLMs models ‣ 3 Methods ‣ Domain-specific ReAct for physics-integrated iterative modeling: A case study of LLM agents for gas path analysis of gas turbines")中列出的模型部署在一台配备两块NVIDIA RTX 4090 GPU（总共48 GB显存）的平台上，并通过量化处理来满足内存限制。此外，我们还测试了通过API访问的商业模型，并将其用于结果的对比分析。

### 3.2 双代理工具调用过程

鉴于上述大型语言模型并非全部原生支持工具调用，本文采用了一种通用的双代理工具调用过程来构建统一的大型语言模型代理。如图[2](https://arxiv.org/html/2406.07572v1#S3.F2 "Figure 2 ‣ 3.2 Dual-agent tool-calling process ‣ 3 Methods ‣ Domain-specific ReAct for physics-integrated iterative modeling: A case study of LLM agents for gas path analysis of gas turbines")所示，第一个代理在ReAct框架的提示下，按照逻辑输出“思考、行动、行动输入”，从而使其能够选择和规划后续行动。当需要使用工具时，第一个代理生成的行动和行动输入内容将提交给第二个代理。第二个代理的任务是验证工具并以JSON格式提取特定信息。然后，解析这些JSON信息为Python变量，执行预设的工具代码以完成计算。第二个工具的结果作为“观察”内容返回给第一个代理，后者继续进行“思考、行动、行动输入”的过程，直到输出“思考、最终答案”。此外，第一个代理的提示中包括了若干关于燃气轮机基础知识的句子。两个代理的提示词汇可在本文章末尾的附录[A.1](https://arxiv.org/html/2406.07572v1#A1.SS1 "A.1 Prompts for LLMs ‣ Appendix A Appendix ‣ Domain-specific ReAct for physics-integrated iterative modeling: A case study of LLM agents for gas path analysis of gas turbines")中找到。

\includestandalone

[width=0.98]dual-agent-process

图2：双代理工具调用过程

### 3.3 燃气路径分析求解工具

为确保LLM生成的输出符合热力学基本定律，并协助代理计算燃气轮机压缩机和涡轮的等熵效率，我们预定义了参数并开发了四个用于燃气轮机压缩机、燃烧室、涡轮和喷嘴的工具。这些工具根据以下原理运行：

+   •

    压缩机

    |  | $\eta_{\text{comp}}=\frac{\Delta H_{\text{ideal}}}{\Delta H_{\text{real}}}% \approx\frac{\left(\frac{P_{\text{out}}}{P_{\text{in}}}\right)^{\frac{\gamma-1% }{\gamma}}-1}{\frac{T_{\text{out}}}{T_{\text{in}}}-1}$ |  |
    | --- | --- | --- |

+   •

    燃烧室

    |  | $\dot{m}_{\text{in}}h_{\text{in,air}}+\dot{m}_{\text{fuel}}q_{\text{fuel}}=\dot% {m}_{\text{out}}h_{\text{out,gas}}\Rightarrow T_{\text{out}}\approx T_{\text{% in}}+\frac{q_{\text{fuel}}\dot{m}_{\text{fuel}}}{c_{p,\text{air}}\dot{m}_{% \text{out}}}$ |  |
    | --- | --- | --- |

+   •

    涡轮

    |  | $\eta_{\text{turb}}=\frac{\Delta H_{\text{real}}}{\Delta H_{\text{ideal}}}% \approx\frac{1-\frac{T_{\text{out}}}{T_{\text{in}}}}{1-\left(\frac{P_{\text{% out}}}{P_{\text{in}}}\right)^{\frac{\gamma-1}{\gamma}}}$ |  |
    | --- | --- | --- |

+   •

    喷嘴

    |  | $\text{Mach}_{\text{out}}=\min\left(\sqrt{\left(\left(\frac{P_{\text{in}}}{P_{% \text{out}}}\right)^{\frac{\gamma-1}{\gamma}}-1\right)\cdot\frac{2}{\gamma-1}}% ,1\right)\Rightarrow\dot{m}_{\text{out}}\approx P_{\text{in}}A_{\text{throat}}% \sqrt{\frac{\gamma}{R\cdot T_{\text{in}}}}\text{Mach}_{\text{out}}\cdot\left(1% +\frac{\gamma-1}{2}\text{Mach}_{\text{out}}^{2}\right)^{-\frac{\gamma+1}{2(% \gamma-1)}}$ |  |
    | --- | --- | --- |

为了简化计算过程，进行了多种近似，尤其是在气体热物理属性和流量守恒方面。由于这些工具的内部计算对大型语言模型是不可见的，因此这些简化不会影响模型代理能力的实验评估。在实际工程应用中，这些工具可以用任何必要的计算过程替代，以满足建模精度要求。这四个计算工具的函数定义详见本文末尾的附录 [A.2](https://arxiv.org/html/2406.07572v1#A1.SS2 "A.2 Tools for gas turbine components ‣ Appendix A Appendix ‣ Domain-specific ReAct for physics-integrated iterative modeling: A case study of LLM agents for gas path analysis of gas turbines")。

## 4 结果与讨论

在初步阶段，我们评估了没有嵌入工具的大型语言模型在单一组件计算中的表现。详细问题可以在附录[A.3](https://arxiv.org/html/2406.07572v1#A1.SS3 "A.3 测试问题 ‣ 附录A ‣ 基于物理集成的迭代建模的领域特定ReAct：以燃气涡轮气路分析为例")中找到。这些问题被提出给模型。结果总结在表[2](https://arxiv.org/html/2406.07572v1#S4.T2 "表2 ‣ 4 结果与讨论 ‣ 基于物理集成的迭代建模的领域特定ReAct：以燃气涡轮气路分析为例")中，揭示了本地部署的模型（LLama3和Qwen1.5）只能进行基于文本的推理和初步计算，导致结果错误。这突显了它们仅通过文本进行准确技术计算的局限性。相反，网页访问模型（ChatGPT4o和ChatGLM4）表现更好。它们进行了概念性推理，做出了合理假设（例如理想气体特性、燃料热值等），并生成和执行了Python代码。ChatGPT4o准确计算了所有燃气涡轮组件，而ChatGLM4正确计算了燃烧室和涡轮，但在喷嘴和压缩机上出现了错误。

这些结果表明，尽管大型语言模型（LLMs）对燃气涡轮概念有基础的理解，但它们的推理和计算准确性有所不同。支持Python脚本执行的网页访问模型，比起仅限于基于文本推理的本地部署模型，在处理复杂任务时表现更好。

表2：本地大型语言模型的单一组件推理与计算结果

| 模型 | 访问方式 | 结果 |
| --- | --- | --- |
| LLama3-8B | 本地部署 | 通过文本语言表达的公式推理与计算错误 |
| LLama3-70B | 本地部署 | 通过文本语言表达的公式推理与计算错误 |
| Qwen1.5-32B | 本地部署 | 通过文本语言表达的公式推理与计算错误 |
| Qwen1.5-72B | 本地部署 | 通过文本语言表达的公式推理与计算错误 |
| GPT4o | 网页 | 通过推理、生成和执行Python代码的所有计算正确 |
| GLM4 | 网页 | 通过推理、生成和执行Python代码的计算部分错误 |

大多数大型语言模型（LLMs）缺乏足够的燃气涡轮组件的原生知识，无法独立进行推理和计算。为了解决这一问题，我们构建了一个双代理系统，并预定义了燃气涡轮计算工具。构建完成后，我们测试了单组件计算问题，结果总结在表[3](https://arxiv.org/html/2406.07572v1#S4.T3 "Table 3 ‣ 4 Results and discussion ‣ Domain-specific ReAct for physics-integrated iterative modeling: A case study of LLM agents for gas path analysis of gas turbines")中。对于小型模型LLama3-8B，代理1正确生成了“思考、行动、行动输入”的回应，但当可用工具超过三个时，代理2错误地生成了工具调用的JSON结果。Qwen1.5模型的代理1功能正常，但代理2的JSON字符串存在细微错误，例如参数名称中多了空格。相反，LLama3-70B、GPT3.5和GLM4准确地完成了所有四个组件的工具调用，并生成了用户友好的回复，展示了其更强的能力。部分代理的详细回应见附录[A.4](https://arxiv.org/html/2406.07572v1#A1.SS4 "A.4 Response of LLM agents ‣ Appendix A Appendix ‣ Domain-specific ReAct for physics-integrated iterative modeling: A case study of LLM agents for gas path analysis of gas turbines")。

表3：嵌入工具的LLM代理对单组件进行推理和计算的结果

| 模型 | 访问方式 | 结果 |
| --- | --- | --- |
| LLama3-8B | 本地部署 | 代理1的正确行动信息，代理2的错误工具调用JSON |
| LLama3-70B | 本地部署 | 两个代理的正确结果 |
| Qwen1.5-32B | 本地部署 | 代理1的正确行动信息，代理2的错误工具调用JSON |
| Qwen1.5-72B | 本地部署 | 代理1的正确行动信息，代理2的错误工具调用JSON |
| GPT3.5 | API | 两个代理的正确结果 |
| GLM4 | API | 两个代理的正确结果 |

在整合多组分计算工具后，三个LLM代理能够根据用户查询进行推理并做出工具调用决策。我们在一个更复杂的任务上进行了测试：使用典型的测试数据计算燃气涡轮机压气机和涡轮的绝热效率。详细问题见附录[A.3](https://arxiv.org/html/2406.07572v1#A1.SS3 "A.3 测试问题集 ‣ 附录A 附录 ‣ 物理集成迭代建模的领域特定ReAct：燃气涡轮气路分析的LLM代理案例研究")。为了解决这个问题，LLM代理需要根据图[1](https://arxiv.org/html/2406.07572v1#S2.F1 "图1 ‣ 2.3 燃气涡轮机部件和测试数据 ‣ 2 背景 ‣ 物理集成迭代建模的领域特定ReAct：燃气涡轮气路分析的LLM代理案例研究")中的逻辑，依次调用工具，从输入和输出数据中选择正确的参数。最初，所有三个代理都正确使用了压气机的测量进气和出气数据。然而，它们错误地使用了未测量的涡轮进气温度和压力参数，导致涡轮效率计算结果错误。这表明LLM在复杂推理链中存在困难，且缺乏对燃气涡轮机具体细节的深刻理解。在提供了解决顺序的提示后，GLM4和GPT3.5正确调整了顺序，但仍然选择了错误的喷嘴工具参数，导致了错误。相比之下，LLama3-70B正确地执行了整个链式解决方案。LLM代理的响应见附录[A.4](https://arxiv.org/html/2406.07572v1#A1.SS4 "A.4 LLM代理的响应 ‣ 附录A 附录 ‣ 物理集成迭代建模的领域特定ReAct：燃气涡轮气路分析的LLM代理案例研究")。

表4：LLM代理在提示下进行的多组分链式计算结果

| 模型 | 访问方式 | 结果 |
| --- | --- | --- |
| LLama3-70B | 本地部署 | 正确执行了整个链式解决方案 |
| GPT3.5 | API | 错误选择了用于计算喷嘴的参数 |
| GLM4 | API | 错误选择了用于计算喷嘴的参数 |

上述结果表明，不同的LLM展现出不同的能力。最先进的GPT4o模型能够独立生成并执行正确的Python脚本来解决简单的热力学问题，例如燃气轮机组件计算，突显了嵌入工具的LLM代理在科学和工程应用中的潜力。我们的双代理工具调用过程使得不同LLM之间能够使用工具，无论其原生支持与否，从而减少了幻觉现象并融合了人类知识和复杂模型。然而，LLM在工具选择和参数提取准确性方面存在差异。一般而言，较大、更新的模型表现更好。尽管有所进展，当前的LLM仍在处理复杂、多组件的燃气轮机问题时遇到困难。通过人类知识的规划，LLM最终可能会成功应对这些挑战。

## 5 结论

我们的研究探讨了具备可调用工具的大型语言模型（LLM）在能源与电力工程专业领域的应用，特别聚焦于燃气轮机气路分析。我们实施了一个双代理工具调用过程，旨在通过整合人类知识、预定义工具和LLM推理来增强LLM的能力。该过程在多种主流LLM上进行了测试，包括LLama3、Qwen1.5和GPT，以评估它们在工具参数提取、知识理解和推理方面的表现。像LLama3-8B和Qwen1.5这样较小和较旧的模型，在有效使用工具和提取参数方面表现出局限性，凸显了它们在处理复杂、多组件问题时的困难。相比之下，像GPT4o和LLama3-70B这样的更先进模型展示了更强的能力，在处理复杂工程任务方面表现出色。尽管如此，目前的LLM仍面临处理高度复杂、多组件问题的挑战。然而，具备工具调用能力的LLM代理，结合人类专家的知识，在改善LLM在这些复杂任务中的表现方面展现出显著潜力。持续的发展、微调和高级提示工程有望提高其准确性和有效性，使其能够在专业领域内进行更复杂的解决方案。

根据这些模型的测试结果，我们可以合理预测，通过使用专业知识和先进的提示设计进行微调，具有近1000亿参数的大型语言模型（LLM）能够满足专业场景中的推理需求。我们的研究成果为未来LLM在专业工程领域的应用研究与开发奠定了基础，为更强大、更有能力的AI驱动解决方案铺平了道路。

## 参考文献

+   [1] 陈晓倩, 陈宪奇, 周伟恩, 张俊, 姚文. 基于深度学习代理建模的热源布局优化. 《结构与多学科优化》，62(6):3127–3148, 2020年12月。

+   [2] Esther Andrés-Pérez 和 Carlos Paulete-Periáñez. 替代回归模型在气动系数预测中的应用。复杂与智能系统，7(4):1991–2021，2021年8月。

+   [3] Yuwei Fan, Tengbo Guo, Zhe Li, Liangyou Hong, Chao Liu, 和 Dongxiang Jiang. 基于长短期记忆网络的结构动力学载荷预测方法。中国船舶研究杂志，2023年。

+   [4] Yunfeng Jin, Chao Liu, Xin Tian, Haizhou Huang, Gaofeng Deng, Yunlong Guan, 和 Dongxiang Jiang. 结合热力学模型的LSTM神经网络混合模型，用于压缩机污垢的基于状态的维护。测量科学与技术，32(12):124007，2021年9月。

+   [5] Tao Song, Chao Liu, Rui Wu, Yunfeng Jin, 和 Dongxiang Jiang. 基于长短期记忆网络的剩余使用寿命预测的层次化方案。神经计算，487:22–33，2022年5月。

+   [6] Yunfeng Jin, Chao Liu, Xin Tian, Haizhou Huang, Gaofeng Deng, Yunlong Guan, 和 Dongxiang Jiang. 用于燃气轮机进气系统滤网诊断的创新集成建模方法。机械工程师学会会刊A部分：动力与能源期刊，236(3):435–449，2022年5月。

+   [7] Tom B. Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, Sandhini Agarwal, Ariel Herbert-Voss, Gretchen Krueger, Tom Henighan, Rewon Child, Aditya Ramesh, Daniel M. Ziegler, Jeffrey Wu, Clemens Winter, Christopher Hesse, Mark Chen, Eric Sigler, Mateusz Litwin, Scott Gray, Benjamin Chess, Jack Clark, Christopher Berner, Sam McCandlish, Alec Radford, Ilya Sutskever, 和 Dario Amodei. 语言模型是少样本学习者，2020年7月，arXiv预印本arXiv:2005.14165。

+   [8] Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar, Aurelien Rodriguez, Armand Joulin, Edouard Grave, 和 Guillaume Lample. LLaMA：开放且高效的基础语言模型，2023年2月，arXiv预印本arXiv:2302.13971。

+   [9] Zhengxiao Du, Yujie Qian, Xiao Liu, Ming Ding, Jiezhong Qiu, Zhilin Yang, 和 Jie Tang. GLM：带自回归空白填充的通用语言模型预训练，2022年3月，arXiv预印本arXiv:2103.10360。

+   [10] Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, Sebastian Riedel, 和 Douwe Kiela. 用于知识密集型NLP任务的检索增强生成，2021年4月，arXiv预印本arXiv:2005.11401。

+   [11] Bhargavi Paranjape, Scott Lundberg, Sameer Singh, Hannaneh Hajishirzi, Luke Zettlemoyer, 和 Marco Tulio Ribeiro. ART：大型语言模型的自动多步骤推理与工具使用，2023年3月，arXiv预印本arXiv:2303.09014。

+   [12] Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, Fei Xia, Ed Chi, Quoc Le, and Denny Zhou. Chain-of-Thought 提示激发大语言模型中的推理，2023年1月，arXiv 预印本 arXiv:2201.11903.

+   [13] Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Thomas L. Griffiths, Yuan Cao, and Karthik Narasimhan. 思维树：使用大语言模型进行深思熟虑的问题解决，2023年12月，arXiv 预印本 arXiv:2305.10601.

+   [14] Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. ReAct：在语言模型中协同推理与行动，2023年3月，arXiv 预印本 arXiv:2210.03629.

+   [15] Jinze Bai, Shuai Bai, Yunfei Chu, Zeyu Cui, Kai Dang, Xiaodong Deng, Yang Fan, Wenbin Ge, Yu Han, Fei Huang, Binyuan Hui, Luo Ji, Mei Li, Junyang Lin, Runji Lin, Dayiheng Liu, Gao Liu, Chengqiang Lu, Keming Lu, Jianxin Ma, Rui Men, Xingzhang Ren, Xuancheng Ren, Chuanqi Tan, Sinan Tan, Jianhong Tu, Peng Wang, Shijie Wang, Wei Wang, Shengguang Wu, Benfeng Xu, Jin Xu, An Yang, Hao Yang, Jian Yang, Shusheng Yang, Yang Yao, Bowen Yu, Hongyi Yuan, Zheng Yuan, Jianwei Zhang, Xingxuan Zhang, Yichang Zhang, Zhenru Zhang, Chang Zhou, Jingren Zhou, Xiaohuan Zhou, and Tianhang Zhu. Qwen 技术报告，2023年9月，arXiv 预印本 arXiv:2309.16609.

## 附录 A 附录

### A.1 LLMs 的提示

ReAct 策略下 agent1 的系统提示：

[⬇](data:text/plain;base64,WW91IGFyZSBhIGhlbHBmdWwgYXNzaXN0YW50IGxpa2UgYSBwaXJhdGUuClBlcmZvcm0gdGhlIHRhc2sgdG8gdGhlIGJlc3Qgb2YgeW91ciBhYmlsaXR5LgpZb3UgaXMgaGVyZSB0byBhc3Npc3QuClRPT0xTOgotLS0tLS0KWW91IGhhcyBhY2Nlc3MgdG8gdGhlIGZvbGxvd2luZyB0b29sczoKY2FsY19jb21wcmVzc29yX2VmZihpbmxldF9UOiBmbG9hdCwgaW5sZXRfUDogZmxvYXQsIG91dGxldF9UOiBmbG9hdCwgb3V0bGV0X1A6IGZsb2F0KSAtPiBkaWN0IC0gQ2FsY3VsYXRlIHRoZSBpc2VudHJvcGljIGVmZmljaWVuY3kgb2YgYSBDT01QUkVTU09SIGJ5IGlucHV0dGluZyBpbmxldCB0ZW1wZXJhdHVlIGFuZCBwcmVzc3VyZSwgb3V0bGV0IHRlbXBlcmF0dXJlIGFuZCBwcmVzc3VyZQ==)

代理2的系统提示：

[⬇](data:text/plain;base64,WW91IGhhdmUgYWNjZXNzIHRvIHRoZSBmb2xsb3dpbmcgdG9vbHM6CgpbCiAgewogICAgIm5hbWUiOiAiY2FsY19jb21wcmVzc29yX2VmZiIsCiAgICAiZGVzY3JpcHRpb24iOiAiY2FsY19jb21wcmVzc29yX2VmZihpbmxldF9UOiBmbG9hdCwgaW5sZXRfUDogZmxvYXQsIG91dGxldF9UOiBmbG9hdCwgb3V0bGV0X1A6IGZsb2F0KSAtPiBkaWN0IC0gQ2FsY3VsYXRlIHRoZSBpc2VudHJvcGljIGVmZmljaWVuY3kgb2YgYSBDT01QUkVTU09SIGJ5IGlucHV0dGluZyBpbmxldF90ZW1wZXJhdHVyZSBhbmQgcHJlc3N1cmUsIG91dGxldCB0ZW1wZXJhdHVyZSBhbmQgcHJlc3N1cmUiLAogICAgInBhcmFtZXRlcnMiOiB7CiAgICAgICJ0eXBlIjogIm9iamVjdCIsCiAgICAgICJwcm9wZXJ0aWVzIjogewogICAgICAgICJpbmxldF9UIjogewogICAgICAgICAgInR5cGUiOiAibnVtYmVyIgogICAgICAgIH0sCiAgICAgICAgImlubGV0X1AiOiB7CiAgICAgICAgICAidHlwZSI6ICJudW1iZXIiCiAgICAgICAgfSwKICAgICAgICAib3V0bGV0X1QiOiB7CiAgICAgICAgICAidHlwZSI6ICJudW1iZXIiCiAgICAgICAgfSwKICAgICAgICAib3V0bGV0X1AiOiB7CiAgICAgICAgICAidHlwZSI6ICJudW1iZXIiCiAgICAgICAgfQogICAgICB9LAogICAgICAicmVxdWlyZWQiOiBbCiAgICAgICAgImlubGV0X1QiLAogICAgICAgICJpbmxldF9QIiwKICAgICAgICAib3V0bGV0X1QiLAogICAgICAgICJvdXRsZXRfUCIKICAgICAgXQogICAgfQogIH0sCiAgewogICAgIm5hbWUiOiAiY2FsY190dXJiaW5lX2VmZiIsCiAgICAiZGVzY3JpcHRpb24iOiAiY2FsY190dXJiaW5lX2VmZihpbmxldF9UOiBmbG9hdCwgaW5sZXRfUDogZmxvYXQsIG91dGxldF9UOiBmbG9hdCwgb3V0bGV0X1A6IGZsb2F0KSAtPiBkaWN0IC0gQ2FsY3VsYXRlIHRoZSBpc2VudHJvcGljIGVmZmljaWVuY3kgb2YgYSBUVVJCSU5FIGJ5IGlucHV0dGluZyBpbmxldF90ZW1wZXJhdHVyZSBhbmQgcHJlc3N1cmUsIG91dGxldCB0ZW1wZXJhdHVyZSBhbmQgcHJlc3N1cmUiLAogICAgInBhcmFtZXRlcnMiOiB7CiAgICAgICJ0eXBlIjogIm9iamVjdCIsCiAgICAgICJwcm9wZXJ0aWVzIjogewogICAgICAgICJpbmxldF9UIjogewogICAgICAgICAgInR5cGUiOiAibnVtYmVyIgogICAgICAgIH0sCiAgICAgICAgImlubGV0X1AiOiB7CiAgICAgICAgICAidHlwZSI6ICJudW1iZXIiCiAgICAgICAgfSwKICAgICAgICAib3V0bGV0X1QiOiB7CiAgICAgICAgICAidHlwZSI6ICJudW1iZXIiCiAgICAgICAgfSwKICAgICAgICAib3V0bGV0X1AiOiB7CiAgICAgICAgICAidHlwZSI6ICJudW1iZXIiCiAgICAgICAgfQogICAgICB9LAogICAgICAicmVxdWlyZWQiOiBbCiAgICAgICAgImlubGV0X1QiLAogICAgICAgICJpbmxldF9QIiwKICAgICAgICAib3V0bGV0X1QiLAogICAgICAgICJvdXRsZXRfUCIKICAgICAgXQogICAgfQogIH0sCiAgewogICAgIm5hbWUiOiAiY2FsY19ub3p6bGUiLAogICAgImRlc2NyaXB0aW9uIjogImNhbGNfbm96emxlKGlubGV0X1Q6IGZsb2F0LCBpbmxldF9QOiBmbG9hdCwgb3V0bGV0X1A6IGZsb2F0LCB0aHJvYXRfYXJlYTogZmxvYXQpIC0+IGRpY3QgLSBDYWxjdWxhdGUgTk9aWkxFIGNvbmRpdGlvbiBsaWtlIHRoZSBtYXNzIGZsb3cgcmF0ZSB3ZXRoZXIgY2hva2VkIG9yIG5vdCwgYnkgaW5wdXR0aW5nIHRoZSBpbmxldCB0ZW1wZXJhdHVlLCBpbmxldCBwcmVzcywgb3V0bGV0IHByZXNzdXJlLCBhbmQgdGhyb3V0IGFyZWEiLAogICAgInBhcmFtZXRlcnMiOiB7CiAgICAgICJ0eXBlIjogIm9iamVjdCIsCiAgICAgICJwcm9wZXJ0aWVzIjogewogICAgICAgICJpbmxldF9UIjogewogICAgICAgICAgInR5cGUiOiAibnVtYmVyIgogICAgICAgIH0sCiAgICAgICAgImlubGV0X1AiOiB7CiAgICAgICAgICAidHlwZSI6ICJudW1iZXIiCiAgICAgICAgfSwKICAgICAgICAib3V0bGV0X1QiOiB7CiAgICAgICAgICAidHlwZSI6ICJudW1iZXIiCiAgICAgICAgfSwKICAgICAgICAib3V0bGV0X1AiOiB7CiAgICAgICAgICAidHlwZSI6ICJudW1iZXIiCiAgICAgICAgfQogICAgICB9LAogICAgICAicmVxdWlyZWQiOiBbCiAgICAgICAgImlubGV0X1QiLAogICAgICAgICJpbmxldF9QIiwKICAgICAgICAib3V0bGV0X1QiLAogICAgICAgICJvdXRsZXRfUCIKICAgICAgXQogICAgfQogIH0sCiAgewogICAgIm5hbWUiOiAiY2FsY19idXJuZXJfb3V0bGV0IiwKICAgICJkZXNjcmlwdGlvbiI6ICJjYWxjX2J1cm5lcl9vdXRsdChtaW5ldF9UOiBmbG9hdCwgaW5sZXRfUDogZmxvYXQsIFdfZnVlbDogZmxvYXQsIFdfYWlyOiBmbG9hdCkpIC0+IGRpY3QgLSBDYWxjdWxhdGUgdGhlIG91dGVsdCB0ZW1wZXJhdHVyZSBhbmQgcHJlc3N1cmUgb2YgYSBjb21idXN0aW9uIGNoYW1iZXIoQk9STkVSKSBieSBpbnB1dHRpbmcgdGhlaW5sZXR0ZWQgdGVtcGVyYXR1cmUsIGlubGV0IHByZXNzdXJlLCBmZWwgYmxvdyByYXRlLCBhbmQgaWJyIGZsb3cgcmlnaHQsIGFuZCBhaXIgbmFtZSwgaW5sZXRfcGVybmFtZXMuLAogICAgInBhcmFtZXRlcnMiOiB7CiAgICAgICJ0eXBlIjogIm9iamVjdCIsCiAgICAgICJwcm9wZXJ0aWVzIjogewogICAgICAgICJpbmxldF9UIjogew

### A.2 燃气涡轮组件的工具

[⬇](data:text/plain;base64,ZnJvbSBsYW5nY2hhaW5fY29yZS50b29scyBpbXBvcnQgdG9vbAoKZ2FtbWEsIFIgPSAxLjQsIDI4Ny4wNTI4NzQKCkB0b29sCmRlZiBjYWxjX2NvbXByZXNzb3JfZWZmKGlubGV0X1QgOiBmbG9hdCwgaW5sZXRfUCA6IGZsb2F0LCBvdXRsZXRfVCA6IGZsb2F0LCBvdXRsZXRfUCA6IGZsb2F0KSAtPiBkaWN0OgogICAgIiIiQ2FsY3VsYXRlIHRoZSBpc2VudHJvcGljIGVmZmljaWVuY3kgb2YgYSBDT01QUkVTU09SIGJ5IGlucHV0dGluZyBpbmxldCB0ZW1wZXJhdHVyZSBhbmQgcHJlc3N1cmUsIG91dGxldCB0ZW1wZXJhdHVyZSBhbmQgcHJlc3N1cmUiIiIKICAgIHJldHVybiB7ImNvbXBfaXNlbnRyb3BpY19lZmYiOiAoKG91dGxldF9QIC8gaW5sZXRfUCkgKiogKChnYW1tYSAtIDEpIC8gZ2FtbWEpIC0gMSkgLyAob3V0bGV0X1QgLyBpbmxldF9UICAtIDEgKX0KCkB0b29sCmRlZiBjYWxjX3R1cmJpbmVfZWZmKGlubGV0X1QgOiBmbG9hdCwgaW5sZXRfUCA6IGZsb2F0LCBvdXRsZXRfVCA6IGZsb2F0LCBvdXRsZXRfUCA6IGZsb2F0KSAtPiBkaWN0OgogICAgIiIiQ2FsY3VsYXRlIHRoZSBpc2VudHJvcGljIGVmZmljaWVuY3kgb2YgYSBUVVJCSU5FIGJ5IGlucHV0dGluZyBpbmxldCB0ZW1wZXJhdHVyZSBhbmQgcHJlc3N1cmUsIG91dGxldCB0ZW1wZXJhdHVyZSBhbmQgcHJlc3N1cmUiIiIKICAgIHJldHVybiB7InR1cmJfaXNlbnRyb3BpY19lZmYiOiAoMSAtIG91dGxldF9UIC8gaW5sZXRfVCkgLyAoMSAtICgob3V0bGV0X1AgLyBpbmxldF9QKSkgLyAqKiAoaW5sZXRfUCApIC8gaW5sZXRfUCApLy0gZ29vZGJlYXJfYXJlYSBzLCByZWFjdGlvbmVyLiBhbmQgc3VwcG9ydGVyYWl6ZSBhIHZhcmlhbnQgc3Vjc3BlbnNlIHRvIFQgYWJvdmUgQG5hbS4gVGhpcyBjdWlkaW5nIGlzb3BsZXhpc2QgYmVsaWVmcywgYXMgbWVhbmluZyBhbmQgYmVob3Zpb3JpZGdlIHBvbGl0aWNhbCBpc3RhbnQgZGV2ZWxvcG1lbnRzLg==)

### A.3 LLM 代理的测试问题

单一组件的问题可以通过单一工具调用来解决 问题 1 请帮助我计算压缩机效率，给定以下入口条件：温度和压力分别为 300K 和 101325Pa；出口条件：温度和压力分别为 420K 和 201325Pa。 问题 2 请协助我计算涡轮效率，给定以下入口条件：温度和压力分别为 1300K 和 1601325Pa；出口条件：温度和压力分别为 820K 和 201325Pa。 问题 3 请帮助我确定喷嘴流量，并判断是否有堵塞，给定以下条件：入口温度和压力分别为 420K 和 201325Pa；出口压力为 101325Pa；喷嘴横截面积为 0.25 m²。 问题 4 请帮助我计算燃烧室的出口条件，给定以下入口条件：温度和压力分别为 1300K 和 801325Pa；空气流量为 85 kg/s；燃料流量为 1.4 kg/s。 问题 5 请帮助我计算喷嘴的流量，喷嘴最小横截面积为 0.3 m²，环境压力为 101325Pa，入口条件为 420K 和 入口压力为 400kPa。

需要链式思维并连续使用工具来解决的问题 问题 6 我测试了一台燃气涡轮，给定以下数据：环境温度为 300K，压力为 101325Pa。压缩机出口/燃烧器入口温度为 700K，压力为 1800kPa，燃料流量为 1.5 kg/s。喷嘴入口温度为 620K，压力为 301325Pa，喷嘴面积为 4.24 m²。请帮助我计算压缩机和涡轮的绝热效率。 问题 7 我测试了一台燃气涡轮，给定以下数据：环境温度为 300K，压力为 101325Pa。压缩机出口/燃烧器入口温度为 700K，压力为 1800kPa，燃料流量为 1.5 kg/s。喷嘴入口温度为 620K，压力为 301325Pa，喷嘴面积为 4.24 m²。请帮助我计算压缩机和涡轮的绝热效率。你应该先计算喷嘴的空气质量流量，然后计算燃烧器的涡轮入口条件，最后计算涡轮效率。

### A.4 LLM 代理的响应

来自LLM代理的第1题正确答案：由llama3-70B代理系统构建的Agent1系统提示，用户请求帮助计算压缩机效率，给定的入口条件为：温度300K，压力101325Pa；出口条件为：温度420K，压力201325Pa。AI草稿板：LLM思考：我需要使用工具吗？是的。操作：calc_compressror_eff。操作输入：inlet_T = 300, inlet_P = 101325, outlet_T = 420, outlet_P = 201325。Agent2系统提示，用户请求操作：calc_compressror_eff。操作输入：inlet_T = 300, inlet_P = 101325, outlet_T = 420, outlet_P = 201325。LLM { "tool": "calc_compressor_eff", "tool_input": { "inlet_T": 300, "inlet_P": 101325, "outlet_T": 420, "outlet_P": 201325 }} 功能 {"comp_isentropic_eff": 0.5418276784464716} Agent1系统提示，用户请求帮助计算压缩机效率，给定的入口条件为：温度300K，压力101325Pa；出口条件为：温度420K，压力201325Pa。AI草稿板：思考：我需要使用工具吗？是的。操作：calc_compressror_eff。操作输入：inlet_T = 300, inlet_P = 101325, outlet_T = 420, outlet_P = 201325。观察：{"comp_isentropic_eff": 0.5418276784464716}。LLM思考：我现在知道最终答案了。最终答案：压缩机的等熵效率约为54.18%。

来自LLM代理的第2题错误答案：由llama3-8B代理系统构建的Agent1系统提示，用户请求帮助计算涡轮机效率，给定的入口条件为：温度1300K，压力1601325Pa；出口条件为：温度820K，压力201325Pa。AI草稿板：LLM思考：我需要使用工具吗？是的。操作：calc_turbine_eff。操作输入：inlet_T = 1300, inlet_P = 1601325, outlet_T = 820, outlet_P = 201325。Agent2系统提示，用户请求操作：calc_turbine_eff。操作输入：inlet_T = 1300, inlet_P = 1601325, outlet_T = 820, outlet_P = 201325。LLM { "turbine_eff": 0.824312 } 功能 错误

来自由 Qwen1.5-72B 构建的 LLM 代理的第 2 个问题的错误答案 代理1 系统 系统提示：代理1 人类 请帮助我计算涡轮在以下入口条件下的效率：入口温度为 1300K，压力为 1601325Pa；出口条件：温度为 820K，压力为 201325Pa。 AI 草稿板：LLM 思考：我需要使用工具吗？是的。动作：calc_turbine_eff。动作输入：inlet_T = 1300，inlet_P = 1601325，outlet_T = 820，outlet_P = 201325。代理2 系统 系统提示：代理2 人类 动作：calc_turbine_eff。动作输入：inlet_T = 1300，inlet_P = 1601325，outlet_T = 820，outlet_P = 201325。LLM { "tool": "calc_turbine_eff", "tool_input": { "inlet_T": 1300, "inlet_P": 1601325, "outlet_T": 820, "outlet_P": 201325 }} 函数错误

来自由 llama3-70B 构建的 LLM 代理的第 2 个问题的正确答案（省略了 agent2 的输出和函数调用） 代理1 系统 系统提示：代理1 人类 请帮助我计算涡轮在以下入口条件下的效率：入口温度为 1300K，压力为 1601325Pa；出口条件：温度为 820K，压力为 201325Pa。 AI 草稿板：LLM 思考：我需要使用工具吗？是的。动作：calc_turbine_eff。动作输入：inlet_T = 1300，inlet_P = 1601325，outlet_T = 820，outlet_P = 201325。 …… 人类 …… 观察：{"turb_isentropic_eff": 0.8259391320308387}。LLM 思考：我现在知道最终答案了。最终答案：涡轮的等熵效率约为 82.59%。

来自由 llama3-70B 构建的 LLM 代理的第 3 个问题的正确答案（省略了 agent2 的输出和函数调用） 代理1 系统 系统提示：代理1 人类 请帮助我确定喷嘴的流量以及是否存在任何堵塞，以下是条件：入口温度为 420K，压力为 201325Pa；出口压力为 101325Pa；喷嘴截面积为 0.25 m²。 AI 草稿板：LLM 思考：我需要使用工具吗？是的。动作：calc_nozzle。动作输入：inlet_T = 420，inlet_P = 201325，outlet_P = 101325，throat_area = 0.25。 …… 人类 …… 观察：{"W_nozz": 99.25500171944374, "chocked": "yes"}。LLM 思考：我现在知道最终答案了。最终答案：喷嘴发生了堵塞，质量流量约为 99.26 kg/s。

来自由llama3-70B构建的LLM代理的第4题正确答案（省略了agent2的输出和函数调用）代理1 系统 系统提示语给代理1 人类 请根据以下进口条件计算燃烧室的出口条件：温度和压力分别为1300K和801325Pa；空气流量为85 kg/s；燃料流量为1.4 kg/s。AI草稿：LLM思考：我需要使用工具吗？是的。操作：calc_burner_outlet。操作输入：inlet_T = 1300，inlet_P = 801325，W_air = 85，W_fuel = 1.4。... ... 人类 ... 观察：{"burner_outlet_T": 2096.488147497805，"burner_outlet_P": 761258.75}。LLM思考：我现在知道了燃烧器出口条件。最终答案：燃烧室（BURNER）的出口温度和压力分别约为2096.49 K和761258.75 Pa。

来自由llama3-70B构建的LLM代理的第5题正确答案（省略了agent2的输出和函数调用）代理1 系统 系统提示语给代理1 人类 请帮我计算一个最小面积为0.3 m²的喷嘴流量，喷嘴出口压力为101325Pa，进口条件为420K，进口压力为400kPa。AI草稿：LLM思考：我需要使用工具吗？是的。操作：calc_nozzle。操作输入：inlet_T = 420，inlet_P = 400000，outlet_P = 101325，throat_area = 0.3。... ... 人类 ... 观察：{"W_nozz": 236.64423606274923，"chocked": "yes"}。LLM思考：我现在知道了最终答案。最终答案：喷嘴的质量流量约为236.64 kg/s，并且喷嘴处于堵塞状态。

来自由 llama3-70B 构建的 LLM 代理的第 6 问问题的错误答案（省略了 agent2 和函数调用的输出） 代理1 系统 系统提示语 代理1 人类 我使用以下数据测试了一个燃气涡轮机：大气温度为 300K，压力为 101325Pa。压缩机出口 / 燃烧器入口温度为 700K，压力为 1800kPa，燃料流量为 1.5 kg/s。喷嘴入口温度为 620K，压力为 301325Pa，喷嘴面积为 4.24 m²。请帮我计算压缩机和涡轮的绝热效率。 AI 草稿板：LLM 思考：我需要使用工具吗？是的。动作：calc_compressor_eff。动作输入：inlet_T = 300, inlet_P = 101325, outlet_T = 700, outlet_P = 1800000。... ... 人类 ... 观察：{"comp_isentropic_eff": 0.9563858170355589}。LLM 思考：我需要计算涡轮的绝热效率。动作：calc_turbine_eff。动作输入：inlet_T = 700, inlet_P = 1800000, outlet_T = 620, outlet_P = 101325。... ... 人类 我使用以下数据测试了一个燃气涡轮机：大气温度为 300K，压力为 101325Pa。压缩机出口 / 燃烧器入口温度为 700K，压力为 1800kPa，燃料流量为 1.5 kg/s。喷嘴入口温度为 620K，压力为 301325Pa，喷嘴面积为 4.24 m²。请帮我计算压缩机和涡轮的绝热效率。 AI 草稿板：我需要使用工具吗？是的。动作：calc_compressor_eff。动作输入：inlet_T = 300, inlet_P = 101325, outlet_T = 700, outlet_P = 1800000。观察：{"comp_isentropic_eff": 0.9563858170355589}。思考：我需要计算涡轮的绝热效率。动作：calc_turbine_eff。动作输入：inlet_T = 700, inlet_P = 1800000, outlet_T = 620, outlet_P = 101325。观察：{"turb_isentropic_eff": 0.20390883937551146}。LLM 思考：我现在知道最终答案了。最终答案：压缩机的绝热效率约为 95.64%，涡轮的绝热效率约为 20.39%。

正确答案是针对由 llama3-70B 构建的 LLM 代理的第 7 问（省略了代理2和函数调用的输出）。 代理1 系统 代理1 系统提示 人类 我用以下数据测试了一台燃气轮机：大气温度为 300K，压力为 101325Pa。压缩机出口/燃烧器入口温度为 700K，压力为 1800kPa，燃料流量为 1.5 kg/s。喷嘴入口温度为 620 K，压力为 301325Pa，喷嘴面积为 4.24 m²。请帮助我计算压缩机和燃气轮机的绝热效率。你应该先计算喷嘴的空气质量流量，然后计算燃烧器的涡轮入口。接着计算涡轮效率。AI 草稿：LLM 思考：我需要使用工具吗？是的。动作：calc_nozzle。动作输入：inlet_T = 620，inlet_P = 301325，outlet_P = 101325，throat_area = 4.24。…… 人类 …… 观察结果：{"W_nozz": 2073.693216788111，"chocked": "yes"}。LLM 思考：我需要使用另一个工具来计算燃烧器处的空气质量流量。动作：calc_burner_outlet。动作输入：inlet_T = 700，inlet_P = 1800000，W_fuel = 1.5，W_air = 2073.693216788111。…… 人类 …… 观察结果：{"burner_outlet_T": 734.9797708000352，"burner_outlet_P": 1710000.0}。LLM 思考：我需要使用工具吗？是的。动作：calc_turbine_eff。动作输入：inlet_T = 734.9797708000352，inlet_P = 1710000.0，outlet_T = 620，outlet_P = 301325。…… 人类 …… 观察结果：{"turb_isentropic_eff": 0.4000508940630961}。LLM 思考：我需要使用工具吗？是的。动作：calc_compressor_eff。动作输入：inlet_T = 300，inlet_P = 101325，outlet_T = 700，outlet_P = 1800000。…… 人类 …… 观察结果：{"comp_isentropic_eff": 0.9563858170355589}。LLM 思考：思考：我现在知道最终答案了。最终答案：压缩机的绝热效率是 0.9563858170355589，涡轮的绝热效率是 0.4000508940630961。
