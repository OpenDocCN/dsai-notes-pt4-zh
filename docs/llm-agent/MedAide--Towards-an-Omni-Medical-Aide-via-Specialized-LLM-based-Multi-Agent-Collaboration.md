<!--yml

类别：未分类

日期：2025-01-11 12:05:30

-->

# MedAide：通过专业化的基于LLM的多代理协作，迈向全方位医疗助手

> 来源：[https://arxiv.org/html/2410.12532/](https://arxiv.org/html/2410.12532/)

魏金杰^(1,2)²²2平等贡献。^§通讯作者。杨定康^(1,2)²²2平等贡献。^§通讯作者。⁴⁴脚注标记：4 李延舒³²²2平等贡献。^§通讯作者。许清耀^(1,2)

陈照宇¹ 李铭成^(1,2) 姜悦^(1,2) 侯晓璐^(1,2) 张丽华^(1,2)⁴⁴脚注标记：4

¹复旦大学工程与技术学院

²复旦大学认知与智能技术实验室

³布朗大学

$\{$dkyang20, lihuazhang$\}$@fudan.edu.cn, jjwei23@m.fudan.edu.cn, yanshu_li1@brown.edu

###### 摘要

基于大型语言模型（LLM）的交互式系统目前在医疗领域展现出潜力。尽管它们具有显著的能力，LLM通常在复杂的医疗应用中缺乏个性化推荐和诊断分析，导致出现幻觉和性能瓶颈。为了应对这些挑战，本文提出了MedAide，一个基于LLM的全方位医疗多代理协作框架，用于专业化的医疗服务。具体来说，MedAide首先通过检索增强生成进行查询重写，以实现准确的医疗意图理解。接着，我们设计了一个上下文编码器来获取意图原型嵌入，利用相似度匹配识别细粒度意图。根据意图的相关性，激活的代理有效协作，提供集成的决策分析。在四个具有复合意图的医疗基准上进行了广泛实验。自动化度量和专家医生评估的实验结果表明，MedAide优于现有的LLM，并提高了它们的医疗能力和战略推理能力。

MedAide：通过专业化的LLM驱动的多代理协作，迈向全方位医疗助手

基于LLM的多代理协作

魏金杰^(1,2)²²2平等贡献。^§通讯作者。杨定康^(1,2)²²2平等贡献。^§通讯作者。⁴⁴脚注标记：4 李延舒³²²2平等贡献。^§通讯作者。许清耀^(1,2) 陈照宇¹ 李铭成^(1,2) 姜悦^(1,2) 侯晓璐^(1,2) 张丽华^(1,2)⁴⁴脚注标记：4 ¹复旦大学工程与技术学院 ²复旦大学认知与智能技术实验室 ³布朗大学 $\{$dkyang20, lihuazhang$\}$@fudan.edu.cn, jjwei23@m.fudan.edu.cn, yanshu_li1@brown.edu

## 1 引言

目标导向对话系统的开发李等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib19)）；程等人（[2023](https://arxiv.org/html/2410.12532v2#bib.bib10)）近年来受到越来越多的关注。先进的大型语言模型（LLMs）OpenAI（[2022](https://arxiv.org/html/2410.12532v2#bib.bib24)）；Achiam等人（[2023](https://arxiv.org/html/2410.12532v2#bib.bib3)）；OpenAI（[2024](https://arxiv.org/html/2410.12532v2#bib.bib25)）在多个目的的人工-机器互动中，如谈判He等人（[2018](https://arxiv.org/html/2410.12532v2#bib.bib15)）和劝说Wang等人（[2019](https://arxiv.org/html/2410.12532v2#bib.bib30)），展现了出色的泛化能力。在这种背景下，以LLM为中心的互动医学助手Bao等人（[2023](https://arxiv.org/html/2410.12532v2#bib.bib5)）；Yang等人（[2024a](https://arxiv.org/html/2410.12532v2#bib.bib36)，[2023b](https://arxiv.org/html/2410.12532v2#bib.bib38)）；Shi（[2023](https://arxiv.org/html/2410.12532v2#bib.bib28)）；Chen等人（[2023b](https://arxiv.org/html/2410.12532v2#bib.bib9)）已成为研究热点，预计能提高诊断效率并促进服务自动化。之前的尝试通过相关语料库建设（例如，知识数据库Li等人（[2023b](https://arxiv.org/html/2410.12532v2#bib.bib23)））和多阶段训练程序（例如，监督性微调Xiong等人（[2023](https://arxiv.org/html/2410.12532v2#bib.bib33)））将LLM与特定的医疗知识融合。尽管这些策略促进了模型对医学相关意图的理解，但在面对需要复杂推理的实际应用时，仍然存在瓶颈。

鉴于人类行为中学习反馈的模仿Du等人（[2023](https://arxiv.org/html/2410.12532v2#bib.bib12)）；Park等人（[2023](https://arxiv.org/html/2410.12532v2#bib.bib26)），自动化医学代理的构建有望增强LLM的指令跟随和逻辑分析能力Lee等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib19)）。多个代理之间的协作，处理不同患者的询问和症状案例，有助于在考虑个体差异的同时实现准确的对话目标达成Fan等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib14)）；Li等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib21)）。尽管取得了显著进展，目前的工作主要集中在医学教育训练Wei等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib31)）或选择性问答Tang等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib29)）上，缺乏对用户查询背后复杂意图的理解，并且未能给出分层的治疗建议。

为了解决这些问题，本文提出了一种基于大语言模型（LLM）的全方位医学代理框架，用于实际医疗应用，称为MedAide。MedAide的核心理念在于，首先通过查询重写来补充和分解多维医学意图，以增强模型对复合型医学查询的解析能力。接着，我们引入了一种上下文编码器，从多方面的指令中学习意图原型嵌入，并利用相似度匹配引理来指定精细化的目标意图。在这种情况下，激活不同的医务代理，提供具有专业医学知识的个性化决策。最终，我们制定了一个具有链式思维（Chain-of-Thought，CoT）特性的决策分析模块，以忠实地总结激活代理的响应，从而做出集成决策。主要贡献总结如下：

+   •

    据我们所知，我们是首个提出针对复合型医疗意图的现实场景全方位多代理协作框架的研究，展示了该框架在推动个性化医疗互动系统方面的潜力。

+   •

    我们的MedAide通过专门的医务代理的反馈与协作，有效提高了LLM在复杂对话目标下的战略推理能力。

+   •

    在七个医学基准数据集上进行的大量实验，涵盖了17种丰富的意图类型，证明了MedAide的有效性。作为一个即插即用的框架，MedAide可以与当前的LLM系统轻松结合，并提供具有竞争力的提升。

## 2 相关工作

### 2.1 LLM在医疗领域的应用

以ChatGPT（OpenAI，[2022](https://arxiv.org/html/2410.12532v2#bib.bib24)）和GPT-4 Achiam等（[2023](https://arxiv.org/html/2410.12532v2#bib.bib3)）为代表的大型语言模型（LLM）在多学科应用中表现出色。尽管当前的LLM Bai（[2023](https://arxiv.org/html/2410.12532v2#bib.bib1)）；Yang等（[2023a](https://arxiv.org/html/2410.12532v2#bib.bib35)）；AI（[2024](https://arxiv.org/html/2410.12532v2#bib.bib4)）通过大规模语料库的支持，具有一定的医学知识，但它们缺乏专业的医学能力，在领域特定场景中存在显著的性能瓶颈。最近，一些尝试如Chen等（[2023b](https://arxiv.org/html/2410.12532v2#bib.bib9)，[a](https://arxiv.org/html/2410.12532v2#bib.bib8)）；Xu（[2023](https://arxiv.org/html/2410.12532v2#bib.bib34)）；Bao等（[2023](https://arxiv.org/html/2410.12532v2#bib.bib5)）；Yang等（[2024a](https://arxiv.org/html/2410.12532v2#bib.bib36)）已开始构建医学定制的LLM助手，以满足诊断和咨询需求。例如，华佗GPT系列 Zhang等（[2023](https://arxiv.org/html/2410.12532v2#bib.bib39)）；Chen等（[2023a](https://arxiv.org/html/2410.12532v2#bib.bib8)）通过吸收真实的医患对话，在弥补通用医学知识差距方面取得了有希望的成果。中医系列 Yang等（[2023b](https://arxiv.org/html/2410.12532v2#bib.bib38)）；Shi（[2023](https://arxiv.org/html/2410.12532v2#bib.bib28)）通过引入专家反馈和多轮医学指导，提高了中文医学能力。此外，PediatricsGPT Yang等（[2024a](https://arxiv.org/html/2410.12532v2#bib.bib36)）提出了一个系统的训练框架，用于构建针对儿科专家和医学通才的互动医疗系统。与之前的研究不同，我们的框架旨在通过基于LLM的多智能体协作，更全面地识别医学意图，并在复杂场景中提升模型的推理能力。

### 2.2 基于LLM的多智能体协作

随着研究者们专注于复杂的目标导向对话生成，Wu等人（[2023](https://arxiv.org/html/2410.12532v2#bib.bib32)）；Fan等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib14)）；Jiang等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib17)）逐渐暴露了大语言模型（LLMs）在幻觉回应和弱理解方面的固有困境，Chen等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib7)）；Yang等人（[2024b](https://arxiv.org/html/2410.12532v2#bib.bib37)）。在此背景下，提出了基于LLMs的自动化代理，旨在通过结合外部工具和数据库，提供有效的感知和决策能力，Cai等人（[2023](https://arxiv.org/html/2410.12532v2#bib.bib6)）；Li等人（[2023a](https://arxiv.org/html/2410.12532v2#bib.bib22)）。通过模仿人类的行为逻辑，多个代理执行反馈与协作，以增强对多样化意图的理解任务，包括教育培训，Lee等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib19)）；Wei等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib31)），情感安慰，Cheng等人（[2023](https://arxiv.org/html/2410.12532v2#bib.bib10)），以及工作流程集成，Hong等人（[2023](https://arxiv.org/html/2410.12532v2#bib.bib16)）。例如，MEDCO，Wei等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib31)）使LLMs能够模拟患者和医生，从而提升虚拟学生在互动环境中的实践表现。MedAgents，Tang等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib29)）通过角色扮演策略提高医疗助手在零样本场景下的表现。相比之下，所提出的MedAide更加关注挖掘深刻的医疗意图，并朝着更稳健的医疗实践发展。

## 3 方法论

图[1](https://arxiv.org/html/2410.12532v2#S3.F1 "Figure 1 ‣ 3 Methodology ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")展示了所提出的MedAide的整体过程。此处，我们详细描述了系统化的工作流程，包括查询重写、意图识别和代理协作。

![请参阅说明](img/3a424c905459b71c9cd2b232719f9e23.png)

图1：所提出的MedAide框架示意图。MedAide由三个主要阶段组成：（i）查询重写旨在重新表述医疗查询并挖掘用户意图，以确保系统准确理解输入的需求目标。在优化后的查询中，（ii）意图识别利用设计的上下文编码器进行语义匹配，将不同的医疗意图精确识别。基于识别结果，（iii）代理协作动态激活相应的医疗代理。随后，决策分析模块将多个代理的输出整合并生成综合分析结果。

### 3.1 查询重写

在这个阶段，我们首先通过查询输入处理器处理初始查询。该处理器基于一组句法标准化算法，将大语言模型（LLMs）与预定义规则集$R$结合（具体细节请参见附录[A.1](https://arxiv.org/html/2410.12532v2#A1.SS1 "A.1 Query Normalization and Parsing ‣ Appendix A Implementation Details ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")）。核心概念是检查、优化并将用户输入标准化为$Q_{std}$。算法[1](https://arxiv.org/html/2410.12532v2#alg1 "Algorithm 1 ‣ 3.1 Query Rewriting ‣ 3 Methodology ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")展示了相应的程序。然后，关键元素提取器从$Q_{std}$中提取关键信息，如症状、病情描述和病史，形成元素集$E_{i}$。在检索增强生成（RAG）中，我们建立了一个包含1,095个专家审阅过的医学指南的索引数据库，并使用语义检索方法（Lewis等人，[2021](https://arxiv.org/html/2410.12532v2#bib.bib20)）检索与这些元素相关的文档，形成文档集$D_{ref}$。之后，召回分析模块通过提示引导选择符合要求的文档，这些文档与$Q_{std}$一起输入到基于LLM的查询提示器中。提示器优化了经验信息并有效分解复合意图。最终，我们设计了一个精炼的查询构造器，将生成的子查询合并和集成。它使用预定义的规则集（具体细节请参见附录[A.2](https://arxiv.org/html/2410.12532v2#A1.SS2 "A.2 Refined Query Construction ‣ Appendix A Implementation Details ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")）过滤并重构多个子查询$Q_{gen}$，以确保语义上的完整性和形式上的一致性输出。

算法 1 基于LLM的输入标准化

输入：用户查询$Q_{input}$，大语言模型$\mathcal{M}$，规则集$\mathcal{R}$ 输出：标准化查询$Q_{std}$ 初始化：$Q_{cur}\leftarrow Q_{input}$，$converged\leftarrow\text{False}$  while not $converged$ do     $converged\leftarrow\text{True}$     for each rule $r\in\mathcal{R}$ do        $Q_{new}\leftarrow\mathcal{M}(Q_{cur},r)$        if $Q_{new}\neq Q_{cur}$ then           $Q_{cur}\leftarrow Q_{new}$           $converged\leftarrow\text{False}$        end if     end for  end while  return $Q_{std}\leftarrow Q_{cur}$

### 3.2 意图识别

在查询重写后，优化的查询 $Q_{opt}$ 将与由上下文编码器生成的一组意图原型嵌入 $E_{i}$ 进行匹配，该编码器旨在捕捉不同医疗意图的语义特征。该编码器建立在 BioBERT Lee 等人（[2019](https://arxiv.org/html/2410.12532v2#bib.bib18)）的基础上，通过执行细粒度的意图分类来学习原型表示。我们在嵌入层后添加了一个全连接层，其输出维度与 17 个医疗意图类别对齐，并通过 softmax 激活函数生成相应的概率分布。具体来说，上下文编码器将优化后的查询与意图嵌入映射到一个 768 维的嵌入空间中。它使用以下公式计算查询和每个意图嵌入 $E_{i}$ 之间的余弦相似度 $S_{ij}$：

|  | $S_{ij}=\frac{Q_{opt}\cdot E_{i}}{\left\&#124;Q_{opt}\right\&#124;\left\&#124;E_{i}\right\&#124;}.$ |  | (1) |
| --- | --- | --- | --- |

随后，经过 softmax 处理后，每个意图的概率分布 $\alpha_{ij}$ 表示为：

|  | $\alpha_{ij}=\frac{\exp(S_{ij})}{\sum_{l=1}^{17}\exp(S_{il})}.$ |  | (2) |
| --- | --- | --- | --- |

如果某个意图 $i$ 的概率 $\alpha_{ij}$ 超过预定阈值，则该意图将被激活，触发相应的代理：

|  | $\text{Activated Intent}_{i}=\left\{\begin{array}[]{ll}1&\text{如果 }\alpha_{ij}>% \text{阈值},\\ 0&\text{否则}.\end{array}\right.$ |  | (3) |
| --- | --- | --- | --- |

通过这种方式，我们的框架可以根据优化后的查询自动激活最符合要求的医疗意图，并将其引导到相应的代理进行后续操作。

### 3.3 代理协作

当框架识别到相关且有效的意图时，相应的代理将被动态激活。考虑到全面性和多样性，代理协作覆盖了系统化的医疗服务，涵盖预诊断、诊断、用药和后诊断等方面。此外，还采用了基于大型语言模型（LLM）的决策分析模块用于信息整合和总结。

具体来说，预诊断代理配备了患者库，这是一个关系数据库 Codd（[1970](https://arxiv.org/html/2410.12532v2#bib.bib11)），用于存储历史患者就诊记录。这些记录可以用于健康评估、识别潜在风险并推荐合适的科室进行就诊。

诊断代理结合了医学记录数据库中提供的506个高质量病例 Fan等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib14)）。我们采用Sawarkar等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib27)）提出的混合检索方案，通过该方案获取与患者病情相似的示例，从而为症状分析和病因排序意图中的精准治疗建议提供可靠证据。混合检索由两个主要部分组成：关键词检索和向量检索。首先，框架通过公式（[1](https://arxiv.org/html/2410.12532v2#S3.E1 "In 3.2 Intent Recognition ‣ 3 Methodology ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")）中的关键词检索从文档集合$D$中提取包含查询关键词的文档子集$D_{\text{slice}}$：

|  | $D_{\text{slice}}=\{d\in D\mid\text{KeywordMatch}(Q,d)=True\}.$ |  | (4) |
| --- | --- | --- | --- |

随后，MedAide根据公式（[2](https://arxiv.org/html/2410.12532v2#S3.E2 "In 3.2 Intent Recognition ‣ 3 Methodology ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")）通过向量检索计算查询$Q$与文档$d$之间的相似度$S(Q,d)$，并保留相似度高于设定阈值$\tau$的文档，形成子集$D_{\text{match}}$：

|  | $S(Q,d)=\frac{E_{Q}\cdot E_{D}}{\|E_{Q}\|\|E_{D}\|},$ |  | (5) |
| --- | --- | --- | --- |
|  | $D_{\text{match}}=\{d\in D\mid S(Q,d)>\tau\}.$ |  | (6) |

随后，$D_{\text{slice}}$中的关键词匹配结果和$D_{\text{match}}$中的向量检索结果被合并，以获得最终相关文档集$D_{\text{final}}$。这些文档将为诊断建议提供实证支持：

|  | $D_{\text{final}}=D_{\text{slice}}\cup D_{\text{match}}$ |  | (7) |
| --- | --- | --- | --- |

在这种情况下，诊断代理进一步解读辅助检查结果，帮助患者了解其健康状况。同时，它提供第二诊断意见，以最小化误诊风险。

药物代理利用从PubMed库中提取的26,684条药物数据 Pud（[2024](https://arxiv.org/html/2410.12532v2#bib.bib2)），通过关键词和语义混合检索为患者提供全面的药物咨询。综合咨询内容包括药物使用、剂量建议、潜在药物相互作用和禁忌症评估，以确保药物的安全性和疗效。此外，它还帮助患者了解潜在副作用，并建议最佳服药时间，以优化治疗效果。

术后诊断代理专注于提供个性化的康复建议和情感支持。它通过更新患者信息库来跟踪和管理恢复过程。

决策分析模块（DAM）结合了CoT属性，它将特定激活的代理输出与患者的病历信息结合，以动态调整治疗方案并制定综合康复计划。基于$D_{\text{final}}$、提示和代理输出$A_{i}$，DAM生成的最终答案表示如下：

|  | $Answer=\text{DAM}(A_{i},\text{prompt},D_{\text{final}}).$ |  | (8) |
| --- | --- | --- | --- |
| 模型 | BLEU-1 | BLEU-2 | ROUGE-1 | ROUGE-2 | ROUGE-L | GLEU |
| --- | --- | --- | --- | --- | --- | --- |
| ZhongJing2/ + MedAide | 9.06/9.18 | 2.68/2.81 | 17.54/19.83 | 1.70/2.86 | 10.04/11.74 | 3.66/4.19 |
| Meditron-7B/ + MedAide | 3.76/4.94 | 0.91/1.30 | 9.63/10.51 | 0.64/0.68 | 4.52/5.21 | 1.74/2.07 |
| HuatuoGPT-II/ + MedAide | 14.6/15.18 | 5.87/6.63 | 26.81/31.84 | 5.26/9.88 | 13.57/19.04 | 6.43/7.25 |
| Baichuan4/ + MedAide | 12.97/15.95 | 5.39/7.85 | 22.68/33.60 | 4.68/11.69 | 13.33/20.75 | 5.57/7.98 |
| LLama-3.1-8B/ + MedAide | 10.95/15.36 | 2.13/6.78 | 22.47/32.54 | 2.62/9.41 | 11.45/19.29 | 4.37/6.45 |
| GPT-4o/ + MedAide | 15.28/15.93 | 6.33/7.56 | 26.89/30.78 | 5.95/8.65 | 14.15/17.64 | 6.34/7.31 |

表 1：在预诊断基准上的比较结果。

| 模型 | BLEU-1 | BLEU-2 | ROUGE-1 | ROUGE-2 | ROUGE-L | GLEU |
| --- | --- | --- | --- | --- | --- | --- |
| ZhongJing2/ + MedAide | 10.50/11.72 | 4.80/5.58 | 24.08/24.59 | 4.93/4.79 | 12.94/11.96 | 5.85/5.79 |
| Meditron-7B/ + MedAide | 12.01/13.93 | 3.13/6.03 | 19.10/26.53 | 3.60/6.12 | 8.60/11.67 | 3.99/6.48 |
| HuatuoGPT-II/ + MedAide | 22.65/25.26 | 10.21/12.11 | 34.51/40.32 | 8.36/11.73 | 15.77/19.11 | 8.95/10.65 |
| Baichuan4/ + MedAide | 22.58/25.51 | 10.98/13.61 | 36.26/42.77 | 10.30/15.00 | 17.03/22.43 | 9.61/11.95 |
| LLama-3.1-8B/ + MedAide | 16.92/22.75 | 6.56/9.97 | 26.39/38.52 | 4.78/10.41 | 12.79/18.68 | 6.21/8.94 |
| GPT-4o/ + MedAide | 21.67/26.80 | 9.73/14.56 | 32.67/44.11 | 7.95/15.81 | 16.20/23.67 | 8.41/12.69 |

表 2：在诊断基准上的比较结果。

## 4 实验

### 4.1 数据集与实现细节

所有实验都在零-shot设置下进行。为了反映现实世界对医疗服务的需求，我们在17种医疗意图上进行了四个不同的基准测试，包括预诊断、诊断、药物和后诊断基准。每个基准包含500个复合意图实例。具体的意图分类，请参考附录中的图 [7](https://arxiv.org/html/2410.12532v2#A2.F7 "Figure 7 ‣ Appendix B Prompt Templates ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")、[8](https://arxiv.org/html/2410.12532v2#A2.F8 "Figure 8 ‣ Appendix B Prompt Templates ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")、[9](https://arxiv.org/html/2410.12532v2#A2.F9 "Figure 9 ‣ Appendix B Prompt Templates ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")和[10](https://arxiv.org/html/2410.12532v2#A2.F10 "Figure 10 ‣ Appendix B Prompt Templates ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")。关于MedAide查询重写和代理协作阶段中使用的提示的全面概述，请参考附录[B](https://arxiv.org/html/2410.12532v2#A2 "Appendix B Prompt Templates ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")。此外，为了实现细粒度的医疗意图识别，我们开发了一个上下文编码器训练数据集，包括2,800个训练样本、300个验证样本和300个测试样本。更多配置细节请参见附录[A.3](https://arxiv.org/html/2410.12532v2#A1.SS3 "A.3 Training Details of BioBert Encoder ‣ Appendix A Implementation Details ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")。

| 模型 | BLEU-1 | BLEU-2 | ROUGE-1 | ROUGE-2 | ROUGE-L | GLEU |
| --- | --- | --- | --- | --- | --- | --- |
| 中京2/ + MedAide | 11.83/11.91 | 4.48/4.57 | 21.61/24.23 | 4.65/5.91 | 13.55/14.75 | 5.32/6.38 |
| Meditron-7B/ + MedAide | 5.76/19.43 | 2.04/17.50 | 12.29/34.84 | 2.12/29.43 | 6.31/32.57 | 2.79/24.47 |
| 华佗GPT-II/ + MedAide | 17.14/37.89 | 9.61/27.47 | 30.72/58.06 | 11.96/33.62 | 19.96/45.39 | 9.57/24.23 |
| 百川4/ + MedAide | 14.64/44.63 | 7.12/36.34 | 26.26/62.11 | 8.09/42.56 | 15.47/49.91 | 7.08/35.99 |
| LLama-3.1-8B/ + MedAide | 13.80/23.87 | 5.90/16.75 | 27.28/47.39 | 7.50/26.75 | 16.05/38.30 | 6.37/15.77 |
| GPT-4o/ + MedAide | 16.23/45.13 | 8.16/38.96 | 29.24/65.09 | 9.78/40.53 | 17.53/52.71 | 8.61/37.09 |

表 3：在药物基准上的比较结果。

| 模型 | BLEU-1 | BLEU-2 | ROUGE-1 | ROUGE-2 | ROUGE-L | GLEU |
| --- | --- | --- | --- | --- | --- | --- |
| 中京2/ + MedAide | 12.97/19.91 | 4.86/6.86 | 25.31/31.98 | 4.21/4.48 | 12.44/15.00 | 5.63/7.22 |
| Meditron-7B/ + MedAide | 8.72/12.78 | 2.46/3.06 | 18.75/20.21 | 1.77/4.32 | 7.28/8.58 | 3.84/4.40 |
| 华佗GPT-II/ + MedAide | 21.39/26.10 | 12.24/12.46 | 35.55/42.23 | 11.02/13.62 | 13.62/21.18 | 10.59/10.96 |
| 百川4/ + MedAide | 15.89/26.10 | 7.06/13.92 | 27.69/44.55 | 6.92/13.05 | 14.86/21.35 | 6.59/11.61 |
| Llama-3.1-8B/ + MedAide | 21.85/22.95 | 7.42/7.68 | 35.36/39.71 | 5.22/7.97 | 15.02/17.93 | 7.65/8.01 |
| GPT-4o/ + MedAide | 19.27/26.83 | 9.28/11.54 | 30.72/40.83 | 8.64/8.71 | 16.22/18.12 | 8.41/9.89 |

表 4：在后诊断基准上的比较结果。

![请参考图注](img/f49267912b5934f032f299eba8b32120.png)

图 2：通过医生评估，MedAide与其他基线的响应比较。

### 4.2 模型库

我们对一系列最先进的（SOTA）模型进行了综合评估。在医学LLM中，ZhongJing2 Shi（[2023](https://arxiv.org/html/2410.12532v2#bib.bib28)）是一个基于Qwen-1.5-1.8B的中医模型，经过完整的训练过程。Meditron-7B 陈等（[2023b](https://arxiv.org/html/2410.12532v2#bib.bib9)）对Llama-2-7B进行了医学相关的持续预训练，以扩展模型在医学知识中的广度和深度。华佗GPT-II（13B）陈等（[2023a](https://arxiv.org/html/2410.12532v2#bib.bib8)）采用单阶段统一方法进行领域适配，提升了医学专业能力和模型的适用性。在通用模型中，百川4 AI（[2024](https://arxiv.org/html/2410.12532v2#bib.bib4)）通过规模法优化长文本，并结合强化学习技术提高推理能力。Llama-3.1-8B 杜贝等（[2024](https://arxiv.org/html/2410.12532v2#bib.bib13)）依赖于分组查询注意力机制以提高推理效率，在多语言任务中表现出色。GPT-4o OpenAI（[2024](https://arxiv.org/html/2410.12532v2#bib.bib25)）展现了卓越的语言理解与生成能力，并擅长处理复杂任务。

### 4.3 与SOTA方法的比较

作为一个即插即用的框架，我们将MedAide与基线模型结合，通过不同的评估指标提供全面的评估，包括BLEU-1/2（%）、ROUGE-1/2/L（%）和GLEU（%）。下面展示了在四个医学基准上的比较结果。

预诊断基准测试结果。如表格[1](https://arxiv.org/html/2410.12532v2#S3.T1 "Table 1 ‣ 3.3 Agent Collaboration ‣ 3 Methodology ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")所示，我们的框架在所有模型的各项指标上持续提高了性能。（i）例如，基于MedAide的华佗GPT-II和百川4分别在ROUGE-L得分上有显著的相对提升，分别为40.3%和55.6%，表明该框架有助于进行多维度的健康风险评估。（ii）此外，MedAide帮助通用LLama-3.1-8B增强了诊断内容生成的多样性和精确度，表现为相较于原始基准的BLEU-1/2平均提高了79.15%。（iii）尽管由于规模法则，ZhongJing2和Meditron-7B在处理复杂医学场景时存在性能限制，但我们的框架仍然提供了持续的性能提升。

诊断基准测试结果。表格[2](https://arxiv.org/html/2410.12532v2#S3.T2 "Table 2 ‣ 3.3 Agent Collaboration ‣ 3 Methodology ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")展示了包含六种复合意图场景的诊断任务结果。（i）结合我们的框架，Meditron-7B和华佗GPT-II在响应准确性和完整性上超越了原始基准，表现在各项指标上有显著提升。这些提升表明MedAide帮助医学LLM提供更全面的症状分析和治疗建议，特别是在解读检查结果时具备更强的层次化能力。（ii）同时，配备MedAide的通用模型（如LLama-3.1-8B和GPT-4o）增强了在医疗服务中的专业性和适应性。（iii）我们观察到ZhongJing-2在ROUGE-2/L和GLEU上的表现略有下降（不到1%）。潜在原因是该模型在查询重写阶段由于容量限制，难以掌握检索增强生成的多维知识，导致预防指导和第二意见不够优化。

|  | 预诊断 | 诊断 | 药物 | 诊后 |
| --- | --- | --- | --- | --- |
| 组件 | ROUGE-L | GLEU | ROUGE-L | GLEU | ROUGE-L | GLEU | ROUGE-L | GLEU |
| 完整MedAide | 17.64 | 7.31 | 23.67 | 12.69 | 52.71 | 37.09 | 18.12 | 9.89 |
| 无查询重写 | 16.02 | 6.44 | 18.35 | 10.37 | 32.06 | 32.45 | 16.48 | 8.79 |
| 有GPT-4识别 | 16.58 | 6.73 | 22.98 | 12.34 | 44.72 | 34.10 | 17.56 | 9.02 |
| 无决策分析 | 17.22 | 7.22 | 23.14 | 12.54 | 48.72 | 35.25 | 18.04 | 9.56 |

表格5：四个基准的消融研究结果。“w/”和“w/o”分别表示有和没有。

| 基准 | MedAide | 无QR | 有GPT-4 R |
| --- | --- | --- | --- |
| 预诊断 | 0.76 | 0.62 | 0.47 |
| 诊断 | 0.83 | 0.61 | 0.49 |
| 药物 | 0.80 | 0.49 | 0.51 |
| 诊后 | 0.63 | 0.56 | 0.37 |
| 意图聚合 | 0.86 | 0.60 | 0.60 |

表 6：五个基准上的 F1 分数（%）比较。“QR”代表查询重写。“GPT-4o R”代表 GPT-4o 识别。

药物基准的结果。表格[3](https://arxiv.org/html/2410.12532v2#S4.T3 "表 3 ‣ 4.1 数据集和实施细节 ‣ 4 实验 ‣ MedAide：通过专门的基于 LLM 的多代理协作，迈向全能医疗助手")提供了不同模型在复合药物意图上的探索结果。(i) 由于 MedAide 中的药物知识注入和原型引导嵌入，药物咨询响应的可靠性在所有基准中得到了持续提升。(ii) 在这种情况下，百川4 和 GPT-4o 展现了最显著的整体提升。例如，GLEU 分数分别提高了 28.91% 和 28.48%。这一发现表明，MedAide 使 LLM 在大窗口上下文下理解丰富的药物信息，并提供专门的剂量推荐。(iii) 在零-shot 推理模式下，Meditron-7B 的 ROUGE-1/2/L 显著从 12.29/2.12/6.31% 提升至 34.84/29.43/32.57%，验证了我们的决策分析模块为药物禁忌理解提供了有利的事实证据。

后诊断基准的结果。(i) 在来自表格[4](https://arxiv.org/html/2410.12532v2#S4.T4 "表 4 ‣ 4.1 数据集和实施细节 ‣ 4 实验 ‣ MedAide：通过专门的基于 LLM 的多代理协作，迈向全能医疗助手")的后诊断应用中，我们观察到大多数基于 MedAide 的模型在多个方面优于原始基准。(ii) GPT-4o 在生成个性化康复建议方面表现更好，在 BLEU-1 分数上提高了 39.23%，为患者提供了更可靠的康复指导。(iii) 与此同时，百川4 在 ROUGE-L 指标上提高了 21.35%，显示出基于 MedAide 的版本在处理动态医疗数据和生成进展跟踪报告方面的优势。

### 4.4 专家医生评估

专家评估在医学模型的实际应用中起着关键作用。我们邀请了6位医生（每位支付300美元）通过多数投票法，选择不同模型生成的响应在引入MedAide前后的优胜者。响应内容通过考虑事实准确性、推荐实用性和人文关怀来进行全面评估。（i）如图[2](https://arxiv.org/html/2410.12532v2#S4.F2 "Figure 2 ‣ 4.1 Datasets and Implementation Details ‣ 4 Experiments ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")所示，所有基于MedAide的模型在不同的基准测试中展现出更高的胜率，表明所提出框架的有效性和适用性。（ii）MedAide不仅提高了不同规模的医学LLM在医疗专业化方面的能力，而且显著增强了通用模型应对复杂医疗任务的能力。

### 4.5 消融研究

我们进行系统的消融研究，以探讨表格[5](https://arxiv.org/html/2410.12532v2#S4.T5 "Table 5 ‣ 4.3 Comparision with SOTA Methods ‣ 4 Experiments ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")中不同组件的影响。选择GPT-4o作为基线来结合MedAide。

查询重写的必要性。首先，我们去除了查询重写过程，以评估其对性能的影响。（i）观察到所有指标的显著下降，表明查询重写对于确保输入信息能够被下游模块准确解读与意图相关至关重要。（ii）此外，将事实信息作为上下文纳入，增强了模型准确理解用户查询的能力，从而减少了幻觉现象并降低了模糊解释的生成。

意图识别的重要性。我们用基于提示的GPT-4o替代了基于学习的上下文编码器，探索不同意图识别策略的影响。（i）尽管GPT-4o在原始基线上的表现有所改进，但它的识别效果仍然是一个次优解。我们认为，经过显式监督训练的编码器能够更有针对性地与实际的医学意图对齐，生成比提示工程更具个性化和代表性的判断。（ii）此外，我们的默认策略具有更好的灵活性和可扩展性，能够根据具体的医疗场景动态优化。

决策分析的有效性。此外，我们直接将激活代理生成的不同输出组合在一起，作为备选候选项。表 [5](https://arxiv.org/html/2410.12532v2#S4.T5 "Table 5 ‣ 4.3 Comparision with SOTA Methods ‣ 4 Experiments ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration") 底部的结果显示，模型在不同医疗意图理解上的一致性能下降，证明了我们方法的有效性。一种合理的解释是，决策分析模块不仅能以有序的方式总结不同代理的输出，还能根据医疗指南和患者病史提供全面且准确的结论。

| 基准测试 | MedAide（我们的） | MedAgent |
| --- | --- | --- |
| 诊断前 | 15.93/17.64 | 14.33/15.26 |
| 诊断 | 27.80/23.67 | 25.46/19.54 |
| 药物 | 45.13/52.17 | 28.69/36.07 |
| 诊断后 | 26.83/18.12 | 25.45/16.69 |

表 7：MedAide 与 MedAgent 在各基准测试中的 BLEU-1/ROUGE-L 分数（%）比较。

![参见图注](img/c50f17ffd1478c66eea8510cfbfe16dc.png)

图 3：针对相同用户查询的不同模型的响应可视化分析。

### 4.6 意图检测分析

为了进一步观察意图检测的表现，表 [6](https://arxiv.org/html/2410.12532v2#S4.T6 "Table 6 ‣ 4.3 Comparision with SOTA Methods ‣ 4 Experiments ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration") 比较了在不同策略下 MedAide 和地面真值之间的 F1 分数。我们从每个基准中随机抽取了 100 个实例，组成了一个更具挑战性的意图聚合基准。核心观察结果如下：（i）完整框架达到了最佳效果，在聚合基准上达到了 86% 的性能，展示了全方位的意图语义理解能力。（ii）在“无 QR”和“有 GPT-4o R”时，性能显著下降，表明医疗查询的细粒度分解和我们定制的意图识别机制的有效性。

![参见图注](img/442bb781ca7031f28712811b21edb45a.png)

图 4：不同代理数量对四个医疗基准测试中性能的影响。

### 4.7 协作框架比较

在这里，我们比较了可复现的多代理框架MedAgent Tang等人（[2024](https://arxiv.org/html/2410.12532v2#bib.bib29)）。该医学框架主要设立了四个科室专用专家，包括儿科、心脏病学、肺病学和新生儿学。每个专家基于GPT-4o实现，协同工作，共同生成统一的报告作为最终结果。（i）从表[7](https://arxiv.org/html/2410.12532v2#S4.T7 "Table 7 ‣ 4.5 Ablation Study ‣ 4 Experiments ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")中可以看出，我们的框架在所有四个基准上都超越了MedAgent。例如，在药物基准上，MedAide以16.44/16.70%的绝对提升，在BLEU-1/ROUGE-L分数上大幅领先MedAgent。这一优势来自我们专门的药物检索，它为模型提供了更精确的药物信息上下文。（ii）此外，Pre-Diagnosis和Post-Diagnosis任务的收益反映出，所提出的MedAide能够更高效地捕捉诊断和护理需求的关键特征。

### 4.8 响应可视化分析

为了直观地比较不同模型的响应质量，我们在图[3](https://arxiv.org/html/2410.12532v2#S4.F3 "Figure 3 ‣ 4.5 Ablation Study ‣ 4 Experiments ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")中展示了MedAide和SOTA方法在相同用户查询下生成的结果。HuatuoGPT-II提供了广泛的医学知识，但缺乏针对性的推荐。虽然MedAgent框架通过多学科视角提供了详细的治疗方案，但它缺乏连贯的诊断推理。相比之下，我们的框架详细解读了患者的结果，并提供个性化的诊断和后续治疗。MedAide在电解质失衡和心血管风险管理方面显示出良好的实用性。

### 4.9 代理数量的影响

图[4](https://arxiv.org/html/2410.12532v2#S4.F4 "Figure 4 ‣ 4.6 Intent Detection Analysis ‣ 4 Experiments ‣ MedAide: Towards an Omni Medical Aide via Specialized LLM-based Multi-Agent Collaboration")展示了当MedAide框架中代理数量变化时，模型性能的变化趋势。当代理数量从2增加到4时，所有基准上的结果都有逐步改善，表明多代理协作有助于提高模型在多任务场景下的表现。相反，当代理数量超过4时，由于引入过多的信息冗余，性能提升不再显著。

## 5 结论与未来工作

本文介绍了MedAide，一个用于复杂医学场景的多代理框架。通过查询重写、意图识别和代理协作，MedAide增强了模型对医学意图的理解，并在多个基准上展现了其有效性。

未来工作。我们计划在未来结合补充性模态（例如医学影像），以提高MedAide在多模态诊断和应用中的潜力。

## 局限性

MedAide框架在将大规模医疗代理与真实临床环境集成方面取得了显著进展，但也存在一定局限性。目前，该框架包括26,684个药物样本和506个真实临床病例记录。尽管如此，这一范围仍然不够全面，特别是在罕见病和专科药物研究方面。此外，由于初版主要强调语言处理，未来的研究计划扩展到多模态能力，尤其是医疗影像数据的集成，以增强框架处理临床诊断中视觉信息的能力。依赖OpenAI的API可能带来潜在的操作挑战，这意味着未来的研究应探索使用更高效的开源模型作为可行的替代方案。尽管存在这些局限性，MedAide框架为推动人工智能在临床诊断中的应用奠定了坚实的基础。

## 伦理考虑

在将医疗代理应用于真实临床环境时，伦理考虑至关重要。我们充分意识到我们研究可能带来的影响，并已采取谨慎的措施来应对这些问题。为了提高透明度，我们承诺公开我们研究中使用的药物数据和医疗记录，以便其他研究人员验证我们的发现并在此基础上开展合作和进一步研究，促进该领域的进展。

我们深刻意识到隐私和数据保护的必要性。所有使用的数据都经过彻底的去标识化处理，所有敏感信息已被移除，并由合作的医疗机构进行了验证。我们邀请医生仅对模型的响应进行评估，不涉及任何形式的人类受试者研究。所有参与者的工作报酬为300美元，严格遵守工作执行地区的最低时薪标准。对于医疗相关数据的使用，我们严格遵循公开数据库的许可协议。对于构建的数据，我们已通过合作医疗机构的伦理委员会进行内部伦理审查，并已获得许可和批准。

## 致谢

本研究部分得到中国国家重点研发计划2021ZD0113502号资助，部分得到上海市科技重大项目2021SHZDZX0103号资助，部分得到上海市科技委员会上海市杰出学术带头人计划（编号：21XD1430300）资助。

## 参考文献

+   Bai（2023）2023。Baichuan。 [https://github.com/baichuan-inc/Baichuan-13B](https://github.com/baichuan-inc/Baichuan-13B)。

+   Pud（2024） 2024. 国家医学图书馆. [https://pubmed.ncbi.nlm.nih.gov/](https://pubmed.ncbi.nlm.nih.gov/)。

+   Achiam 等（2023） Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat 等. 2023. GPT-4 技术报告。 *arXiv 预印本 arXiv:2303.08774*。

+   AI（2024） Baichuan AI. 2024. 介绍 baichuan4. [https://platform.baichuan-ai.com/homePage](https://platform.baichuan-ai.com/homePage)。

+   Bao 等（2023） Zhijie Bao, Wei Chen, Shengze Xiao, Kuang Ren, Jiaao Wu, Cheng Zhong, Jiajie Peng, Xuanjing Huang, 和 Zhongyu Wei. 2023. Disc-medllm: Bridging general large language models and real-world medical consultation. *arXiv 预印本 arXiv:2308.14346*。

+   Cai 等（2023） Tianle Cai, Xuezhi Wang, Tengyu Ma, Xinyun Chen, 和 Denny Zhou. 2023. 大型语言模型作为工具创造者。 *arXiv 预印本 arXiv:2305.17126*。

+   Chen 等（2024） Jiawei Chen, Dingkang Yang, Tong Wu, Yue Jiang, Xiaolu Hou, Mingcheng Li, Shunli Wang, Dongling Xiao, Ke Li, 和 Lihua Zhang. 2024. 在大型视觉语言模型中检测和评估医学幻觉。 *arXiv 预印本 arXiv:2406.10185*。

+   Chen 等（2023a） Junying Chen, Xidong Wang, Anningzhe Gao, Feng Jiang, Shunian Chen, Hongbo Zhang, Dingjie Song, Wenya Xie, Chuyi Kong, Jianquan Li, Xiang Wan, Haizhou Li, 和 Benyou Wang. 2023a. [Huatuogpt-ii，一阶段训练用于大型语言模型的医学适应](https://api.semanticscholar.org/CorpusID:265221365). *ArXiv*, abs/2311.09774。

+   Chen 等（2023b） Zeming Chen, Alejandro Hernández-Cano, Angelika Romanou, Antoine Bonnet, Kyle Matoba, Francesco Salvi, Matteo Pagliardini, Simin Fan, Andreas Köpf, Amirkeivan Mohtashami, Alexandre Sallinen, Alireza Sakhaeirad, Vinitra Swamy, Igor Krawczuk, Deniz Bayazit, Axel Marmet, Syrielle Montariol, Mary-Anne Hartley, Martin Jaggi, 和 Antoine Bosselut. 2023b. [Meditron-70b: 为大型语言模型扩展医学预训练](https://arxiv.org/abs/2311.16079). *预印本*, arXiv:2311.16079。

+   Cheng 等（2023） Yi Cheng, Wenge Liu, Jian Wang, Chak Tou Leong, Yi Ouyang, Wenjie Li, Xian Wu, 和 Yefeng Zheng. 2023. [Cooper: 协调专门化代理达成复杂对话目标](https://arxiv.org/abs/2312.11792). *预印本*, arXiv:2312.11792。

+   Codd（1970） Edgar F Codd. 1970. 大型共享数据库的数据关系模型。 *ACM 通讯*, 13(6):377–387。

+   Du 等（2023） Yilun Du, Shuang Li, Antonio Torralba, Joshua B Tenenbaum, 和 Igor Mordatch. 2023. 通过多代理辩论提高语言模型的事实性和推理能力。 *arXiv 预印本 arXiv:2305.14325*。

+   Dubey 等（2024） Abhimanyu Dubey, Abhinav Jauhri, Abhinav Pandey, Abhishek Kadian, Ahmad Al-Dahle, Aiesha Letman, Akhil Mathur, Alan Schelten, Amy Yang, Angela Fan 等. 2024. Llama 3 模型群体。 *arXiv 预印本 arXiv:2407.21783*。

+   Fan 等人（2024）Zhihao Fan, Jialong Tang, Wei Chen, Siyuan Wang, Zhongyu Wei, Jun Xi, Fei Huang, 和 Jingren Zhou. 2024. [Ai hospital: 在多代理医疗互动仿真器中对大型语言模型进行基准测试](https://arxiv.org/abs/2402.09742)。*预印本*, arXiv:2402.09742。

+   He 等人（2018）He He, Derek Chen, Anusha Balakrishnan, 和 Percy Liang. 2018. 在谈判对话中解耦策略和生成。*arXiv 预印本 arXiv:1808.09637*。

+   Hong 等人（2023）Sirui Hong, Mingchen Zhuge, Jonathan Chen, Xiawu Zheng, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, 等人. 2023. Metagpt：多代理协作框架的元编程。载于 *第十二届国际学习表征会议*。

+   Jiang 等人（2024）Yue Jiang, Jiawei Chen, Dingkang Yang, Mingcheng Li, Shunli Wang, Tong Wu, Ke Li, 和 Lihua Zhang. 2024. Medthink：通过更多思考让医学大规模视觉语言模型减少幻觉。*arXiv 预印本 arXiv:2406.11451*。

+   Lee 等人（2019）Jinhyuk Lee, Wonjin Yoon, Sungdong Kim, Donghyeon Kim, Sunkyu Kim, Chan Ho So, 和 Jaewoo Kang. 2019. [Biobert: 一个预训练的生物医学语言表示模型，用于生物医学文本挖掘](https://api.semanticscholar.org/CorpusID:59291975)。*Bioinformatics*, 36:1234 – 1240。

+   Lee 等人（2024）Unggi Lee, Sanghyeok Lee, Junbo Koh, Yeil Jeong, Haewon Jung, Gyuri Byun, Yunseo Lee, Jewoong Moon, Jieun Lim, 和 Hyeoncheol Kim. 2024. 面向教师培训的生成性代理：为预服务教师设计基于大型语言模型的教育问题解决仿真。

+   Lewis 等人（2021）Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen tau Yih, Tim Rocktäschel, Sebastian Riedel, 和 Douwe Kiela. 2021. [基于检索增强生成的知识密集型 NLP 任务](https://arxiv.org/abs/2005.11401)。*预印本*, arXiv:2005.11401。

+   Li 等人（2024）Binxu Li, Tiankai Yan, Yuanting Pan, Zhe Xu, Jie Luo, Ruiyang Ji, Shilong Liu, Haoyu Dong, Zihao Lin, 和 Yixin Wang. 2024. Mmedagent：通过多模态代理学习使用医学工具。*arXiv 预印本 arXiv:2407.02483*。

+   Li 等人（2023a）Guohao Li, Hasan Hammoud, Hani Itani, Dmitrii Khizbullin, 和 Bernard Ghanem. 2023a. Camel：用于大语言模型社会“思维”探索的交互式代理。*神经信息处理系统进展*, 36:51991–52008。

+   Li 等人（2023b）Yunxiang Li, Zihan Li, Kai Zhang, Ruilong Dan, Steve Jiang, 和 You Zhang. 2023b. Chatdoctor：基于医疗领域知识在大型语言模型 Meta-AI (llama) 上微调的医学聊天模型。*Cureus*, 15(6)。

+   OpenAI（2022）OpenAI. 2022. 介绍 ChatGPT。 [https://openai.com/blog/chatgpt](https://openai.com/blog/chatgpt)。

+   OpenAI（2024）OpenAI. 2024. 介绍 o1。 [https://openai.com/index/introducing-openai-o1-preview/](https://openai.com/index/introducing-openai-o1-preview/)。

+   Park 等（2023）Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang, 和 Michael S Bernstein。2023。Generative agents: 人类行为的互动拟像。收录于*第36届ACM用户界面软件与技术年会论文集*，页面1–22。

+   Sawarkar 等（2024）Kunal Sawarkar, Abhilasha Mangal, 和 Shivam Raj Solanki。2024。[Blended rag: 通过语义搜索和混合查询检索器提高rag（检索增强生成）准确性](https://arxiv.org/abs/2404.07220)。*预印本*，arXiv:2404.07220。

+   Shi（2023）Liu Lin Ju Shi。2023。Cmlm-zhongjing-2-1-8b: 一种面向传统中医的最先进边缘计算语言模型。[https://github.com/pariskang/CMLM-ZhongJing](https://github.com/pariskang/CMLM-ZhongJing)。

+   Tang 等（2024）Xiangru Tang, Anni Zou, Zhuosheng Zhang, Ziming Li, Yilun Zhao, Xingyao Zhang, Arman Cohan, 和 Mark Gerstein。2024。[Medagents: 大型语言模型作为零-shot医学推理的合作伙伴](https://arxiv.org/abs/2311.10537)。*预印本*，arXiv:2311.10537。

+   Wang 等（2019）Xuewei Wang, Weiyan Shi, Richard Kim, Yoojung Oh, Sijia Yang, Jingwen Zhang, 和 Zhou Yu。2019。Persuasion for good: 朝着个性化的说服对话系统为社会公益而努力。收录于*第57届计算语言学协会年会论文集*，页面5635–5649。

+   Wei 等（2024）Hao Wei, Jianing Qiu, Haibao Yu, 和 Wu Yuan。2024。Medco: 基于多智能体框架的医学教育副驾驶。*arXiv预印本 arXiv:2408.12496*。

+   Wu 等（2023）Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang, 和 Chi Wang。2023。Autogen: 通过多智能体对话框架实现下一代LLM应用。*arXiv预印本 arXiv:2308.08155*。

+   Xiong 等（2023）Honglin Xiong, Sheng Wang, Yitao Zhu, Zihao Zhao, Yuxiao Liu, Linlin Huang, Qian Wang, 和 Dinggang Shen。2023。Doctorglm: 微调你的中文医生并不是一项艰巨的任务。*arXiv预印本 arXiv:2304.01097*。

+   Xu（2023）Ming Xu。2023。Medicalgpt: 训练医学gpt模型。[https://github.com/shibing624/MedicalGPT](https://github.com/shibing624/MedicalGPT)。

+   Yang 等（2023a）Aiyuan Yang, Bin Xiao, Bingning Wang, Borong Zhang, Ce Bian, Chao Yin, Chenxu Lv, Da Pan, Dian Wang, Dong Yan, 等。2023a。Baichuan 2: 开放的大规模语言模型。*arXiv预印本 arXiv:2309.10305*。

+   Yang 等（2024a）Dingkang Yang, Jinjie Wei, Dongling Xiao, Shunli Wang, Tong Wu, Gang Li, Mingcheng Li, Shuaibing Wang, Jiawei Chen, Yue Jiang, 等。2024a。Pediatricsgpt: 大型语言模型作为中国医学助手用于儿科应用。*arXiv预印本 arXiv:2405.19266*。

+   杨等人（2024b）丁康杨、东凌肖、金杰魏、名成李、兆宇陈、克李、丽华张。2024b。通过解码时的虚假和真实比较器提高大语言模型的事实准确性。*arXiv预印本 arXiv:2408.12325*。

+   杨等人（2023b）宋华杨、汉杰赵、森彬朱、光宇周、红飞徐、玉翔贾、红英赞。2023b。[Zhongjing：通过专家反馈和真实世界多轮对话提升大语言模型的中文医学能力](https://arxiv.org/abs/2308.03549)。*预印本*，arXiv:2308.03549。

+   张等人（2023）红波张、俊英陈、风江、飞于、志宏陈、建全李、桂名陈、向波吴、志毅张、清颖肖等。2023年。Huatuogpt，致力于驯服语言模型成为医生。*arXiv预印本 arXiv:2305.15075*。

## 附录 A 实现细节

### A.1 查询规范化与解析

算法 2 语法解析算法

0:  输入查询 $Q$0:  解析的语法树 $T$  步骤 1: 分词查询  设 $Q=\{w_{1},w_{2},\dots,w_{n}\}$ 为查询中的单词（标记）序列。  步骤 2: 初始化解析  初始化一个空的语法树 $T=\emptyset$。  步骤 3: 应用上下文无关文法（CFG）  对于文法 $G$ 中的每个非终结符号 $A$，执行如下操作：      找到一个规则 $A\rightarrow\alpha$，其中 $\alpha$ 是一串终结符或非终结符。      如果 $A\rightarrow\alpha$ 与查询 $Q$ 中的子串匹配，则：        将此产生式规则添加到语法树 $T$ 中。        用非终结符 $A$ 替换匹配的子串。      结束 如果  结束 对于  步骤 4: 递归  当仍有未解析的非终结符时，执行如下操作：      重复步骤 3，直到 $T$ 覆盖整个查询 $Q$。  结束 当  步骤 5: 输出语法树  返回 $T$

算法 [2](https://arxiv.org/html/2410.12532v2#alg2 "算法 2 ‣ A.1 查询规范化和解析 ‣ 附录 A 实现细节 ‣ MedAide：通过基于专门化LLM的多代理协作，朝着全能医学助手迈进") 首先对输入查询进行消歧并将查询分割成单独的单词或标记。接下来，算法初始化一个空的语法树，并通过上下文无关文法（CFG）规则匹配查询中的子串，将符合的规则添加到语法树中。算法不断递归应用规则，直到语法树覆盖整个查询，最终输出完整的语法树。

### A.2 精细化查询构造

在精炼查询构造器中，我们实施了一套规则（如图 [5](https://arxiv.org/html/2410.12532v2#A1.F5 "图 5 ‣ A.2 精炼查询构建 ‣ 附录 A 实现细节 ‣ MedAide：通过基于专门化大语言模型的多代理协作构建全方位医学助手") 所示），以确保生成的子查询在语义和形式上都具有高质量。这些规则包括子查询过滤、语义重叠移除、查询合并、语法标准化、意图优先级排序和格式标准化。这些过程共同优化输出，优化查询结构，并优先处理关键的医学意图，确保最终查询在语义上连贯且在结构上统一。

![请参见标题](img/4bc5f511b9ea16914b921d56646ac126.png)

图 5：规则集的示意图。

### A.3 BioBert 编码器的训练细节

对于医学意图分类，我们使用基于 BioBERT 的上下文编码器，该编码器来自 HuggingFace 库^*^**[https://huggingface.co/dmis-lab/biobert-v1.1](https://huggingface.co/dmis-lab/biobert-v1.1)，将输入句子映射到 768 维的嵌入空间。对于每个意图分类任务，我们在嵌入层之后添加一个全连接层，输出维度为 17，并通过 softmax 激活函数生成与 17 个意图相对应的概率分布。为了处理这些任务，我们部署了四个 V100 32GB GPU，以提供足够的计算资源和图形内存。在训练过程中使用 Adam 优化器，初始学习率为 2e-5，批量大小为 32，训练轮数为 5，并且在验证集上应用学习率调度策略进行动态调整。损失函数采用交叉熵损失函数。我们通过准确率、精确率、召回率和 F1 分数来评估模型，以确保模型在验证集上的表现。

## 附录 B 提示模板

本节概述了我们框架中四个不同代理所使用的结构化提示模板。每个提示模板旨在引导语言模型生成与特定医学任务相关且准确的回应。这些模板对于确保代理与用户之间的互动有效且具有医学信息性至关重要。此外，查询重写阶段各个组件的提示模板详见图 [6](https://arxiv.org/html/2410.12532v2#A2.F6 "图 6 ‣ 附录 B 提示模板 ‣ MedAide：通过专门的基于 LLM 的多代理协作迈向全能医学助手")。每个代理的提示模板的具体内容分别展示在图 [7](https://arxiv.org/html/2410.12532v2#A2.F7 "图 7 ‣ 附录 B 提示模板 ‣ MedAide：通过专门的基于 LLM 的多代理协作迈向全能医学助手")、[8](https://arxiv.org/html/2410.12532v2#A2.F8 "图 8 ‣ 附录 B 提示模板 ‣ MedAide：通过专门的基于 LLM 的多代理协作迈向全能医学助手")、[9](https://arxiv.org/html/2410.12532v2#A2.F9 "图 9 ‣ 附录 B 提示模板 ‣ MedAide：通过专门的基于 LLM 的多代理协作迈向全能医学助手")和 [10](https://arxiv.org/html/2410.12532v2#A2.F10 "图 10 ‣ 附录 B 提示模板 ‣ MedAide：通过专门的基于 LLM 的多代理协作迈向全能医学助手")。

![请参阅说明文字](img/70f9c0ab4e642502990a2a21047ebbec.png)

图 6：查询重写的提示模板示意图。

![请参阅说明文字](img/bce3e1e5cefd10492034fcc012d463d3.png)

图 7：预诊断的提示模板示意图。

![请参阅说明文字](img/4ebd4b02d63f8c59959a84ddfc9e7e6d.png)

图 8：诊断的提示模板示意图。

![请参阅说明文字](img/c500670dbee2c1bbe9d0db1932eeffc4.png)

图 9：药物的提示模板示意图。

![请参阅说明文字](img/7f2bf25892bc218ebcc45843ea78eb41.png)

图 10：后诊断的提示模板示意图。
