<!--yml

类别：未分类

日期：2025-01-11 11:45:08

-->

# KG4Diagnosis：一种基于知识图谱增强的分层多代理大型语言模型框架，用于医疗诊断

> 来源：[https://arxiv.org/html/2412.16833/](https://arxiv.org/html/2412.16833/)

Kaiwen Zuo¹，Yirui Jiang^(2,4)，Fan Mo^(3, 4)，Pietro Liò³ 通讯作者。

###### 摘要

将大型语言模型（LLMs）应用于医疗诊断需要系统化的框架，能够处理复杂的医疗场景，并保持专业知识。我们提出了KG4Diagnosis，这是一个新颖的分层多代理框架，将LLMs与自动化知识图谱构建相结合，涵盖了362种常见疾病，跨越多个医学专业领域。我们的框架通过两级架构反映现实世界中的医疗系统：一个初步评估和分诊的全科医生（GP）代理，与专门的代理协作进行特定领域的深入诊断。核心创新在于我们的端到端知识图谱生成方法，包含：（1）优化医疗术语的语义驱动实体与关系提取，（2）从非结构化医疗文本中重构多维度决策关系，以及（3）由人工引导的推理来扩展知识。KG4Diagnosis作为一个可扩展的基础平台，用于构建专业的医疗诊断系统，具备整合新疾病和医疗知识的能力。框架的模块化设计使得领域特定的增强功能能够无缝集成，为开发针对性的医疗诊断系统提供了价值。我们提供了架构指南和协议，以促进在不同医疗场景中的应用。

## 引言

知识图谱（KGs）已成为许多领域中的变革性工具，展示了它们在组织复杂数据集和支持高级推理与决策中的能力。在金融领域，KGs通过将不同的金融数据集关联起来，揭示隐藏的模式和关系，起着关键作用，尤其在风险评估和欺诈检测方面。例如，KGs在检测欺诈性关联交易中的应用，使金融机构能够建模实体之间复杂的相互依赖关系，从而提高了识别欺诈活动的准确性（Zhang, Li, 和 Wang [2023](https://arxiv.org/html/2412.16833v2#bib.bib38)）。类似地，在教育领域，KGs通过构建来自海量学术资源的知识，提升了个性化学习，能够推荐量身定制的学习路径。一项显著的应用是利用KGs整合来自课程设计、学生评估和教学资源的数据，创建适应性系统，进而提升学生的参与度和学习成果。在制造业中，知识图谱（KGs）通过整合异构数据源，使过程的自动化和优化成为可能。最近的一项研究强调了KGs在可重构制造系统（RMS）中的作用，其中语义模型和KGs支持自动化资产能力匹配和重构解决方案。这种方法通过利用结构化知识支持制造系统中的动态决策，显著提高了效率、降低了成本并提升了生产力（Mo 等人 [2024](https://arxiv.org/html/2412.16833v2#bib.bib20)）。Du 等人利用多特征融合技术构建了高效的制造知识图谱，并成功应用于汽车制造（Du 等人 [2022](https://arxiv.org/html/2412.16833v2#bib.bib11)）。

在医疗领域，KGs（Abdulla, Mukherjee, 和 Ranganathan [2023](https://arxiv.org/html/2412.16833v2#bib.bib1); Alam, Giglou, 和 Malik [2023](https://arxiv.org/html/2412.16833v2#bib.bib3); Wu 等人 [2024](https://arxiv.org/html/2412.16833v2#bib.bib34)）作为组织多样化医疗数据和支持临床决策的重要基础设施发挥着关键作用。然而，构建和推理医疗KGs（Abdulla, Mukherjee, 和 Ranganathan [2023](https://arxiv.org/html/2412.16833v2#bib.bib1); Al Khatib 等人 [2024](https://arxiv.org/html/2412.16833v2#bib.bib2)），特别是从非结构化和多模态数据中构建，仍然面临着现有方法尚未完全解决的重要挑战。

当前的医学知识图谱（KG）构建方法涵盖了从传统的基于规则的系统到先进的人工智能模型。基于规则和本体驱动的方法，如使用SNOMED-CT（Chang和Mostafa [2021](https://arxiv.org/html/2412.16833v2#bib.bib10)）和UMLS（Amos等人 [2020](https://arxiv.org/html/2412.16833v2#bib.bib5)），提供了可靠性，但缺乏可扩展性，且在处理非结构化数据时存在困难。虽然像GPT这样的**大语言模型**（LLMs）（OpenAI [2022](https://arxiv.org/html/2412.16833v2#bib.bib22)，[2023](https://arxiv.org/html/2412.16833v2#bib.bib23)；Touvron等人 [2023](https://arxiv.org/html/2412.16833v2#bib.bib33)；García-Ferrero等人 [2024](https://arxiv.org/html/2412.16833v2#bib.bib12)）和MedPaLM（Qian等人 [2024](https://arxiv.org/html/2412.16833v2#bib.bib26)）在从非结构化数据生成结构化知识方面表现出潜力，但它们在幻觉和准确性方面面临挑战（Huang等人 [2023](https://arxiv.org/html/2412.16833v2#bib.bib14)；Tonmoy等人 [2024](https://arxiv.org/html/2412.16833v2#bib.bib32)；Guo等人 [2024](https://arxiv.org/html/2412.16833v2#bib.bib13)）。结合图神经网络（GNNs）的混合方法尝试平衡符号推理与深度学习，但仍然计算复杂，并依赖于良好结构化的输入（Zhang [2021](https://arxiv.org/html/2412.16833v2#bib.bib37)；Zhang等人 [2024](https://arxiv.org/html/2412.16833v2#bib.bib36)；Shuifa等人 [2023](https://arxiv.org/html/2412.16833v2#bib.bib28)）。

在诊断和治疗中，医学知识图谱为识别患者数据、医学文献和临床指南中的模式和关系提供了重要的基础（Li等人 [2020](https://arxiv.org/html/2412.16833v2#bib.bib15)）。在诊断中，知识图谱有助于将症状与潜在疾病匹配，识别相关检查，并优先考虑鉴别诊断（Tang等人 [2023](https://arxiv.org/html/2412.16833v2#bib.bib31)）。在治疗中，知识图谱有助于根据患者特定因素，如共病、药物相互作用和基因标记，推荐个性化的治疗方案（Bonner等人 [2022](https://arxiv.org/html/2412.16833v2#bib.bib7)）。这些过程通过提供结构化、基于证据的建议，增强了临床决策。

为了应对当前方法的局限性并提升整体临床工作流程，我们提出了KG4Diagnosis，这是一种新型的端到端框架，用于构建、诊断、治疗和推理自动化医学知识图谱。我们的框架独特地整合了分层的多智能体架构，模拟现实世界的医学系统：全科医生（GP）智能体进行初步评估和分诊，然后与专科智能体协调进行领域特定分析。这种方法将**大语言模型**（LLMs）的广泛能力与专业医学知识的精确性相结合，确保了准确的诊断、个性化的治疗建议和增强的临床决策。

该框架创新性地结合了语义实体提取、决策重建和可扩展知识扩展的先进技术，专门设计用于处理非结构化和多模态医疗数据。通过弥合传统知识图谱方法与现代AI能力之间的差距，KG4Diagnosis旨在实现更强大且适应性更强的医疗决策支持系统。

在本文中，我们做出了以下关键贡献：

+   •

    我们提出了KG4Diagnosis，一个新颖的分层多智能体框架，模拟现实世界的医疗系统，包含用于初步评估的全科医生代理和针对362种常见疾病的领域专用诊断代理。

+   •

    我们开发了一个创新的端到端知识图谱构建流程，融合了三个关键组件：语义驱动的实体提取、多维决策关系重建和人工引导的推理用于知识扩展。

+   •

    我们通过多智能体验证和知识图谱约束实现了应对大型语言模型（LLM）幻觉挑战的强大机制，并通过全面的基准测试进行了验证。

+   •

    我们通过现实世界的医疗场景展示了框架的实际价值。

+   •

    我们提供了一个模块化且可扩展的架构，支持新医疗领域和知识的无缝集成，并提供了详细的实施协议，以便在各种医疗环境中广泛采用。

![参见说明](img/4f7da88f62e930cef9837880869a0089.png)

图1：KG4Diagnosis框架概述。该系统包含以下组件：(1) 输入的医疗文本被分段，并通过实体提取和关系提取模块进行处理；(2) 提取的实体和关系存储在专用数据库中；(3) 这些数据库用于构建医疗知识图谱；(4) 医疗知识图谱与大型语言模型（LLMs）和多智能体系统（MAS）集成，以增强诊断推理；(5) 诊断响应通过人工引导推理提供给用户端点。该框架突出展示了医疗文本处理、知识图谱构建和协作推理的结构化方法，以实现更先进的诊断结果。

![参见说明](img/4e7cc53e0856a2ea7aeb1c255ae751ca.png)

图2：一个诊断对话示例，展示了患者、医生和AI医疗助手之间的互动。患者描述症状，医生提问以澄清，AI提供解释和建议。该对话突出了协作诊断过程，以及AI系统如何帮助提供个性化的医疗建议。

## 方法论

### 系统架构概述

KG4Diagnosis 被设计为一个层次化的多代理框架，将大型语言模型（LLMs）与自动化知识图谱构建相结合，用于医疗诊断（见图 [1](https://arxiv.org/html/2412.16833v2#Sx1.F1 "Figure 1 ‣ Introduction ‣ KG4Diagnosis: A Hierarchical Multi-Agent LLM Framework with Knowledge Graph Enhancement for Medical Diagnosis")）。系统架构包含两个主要组成部分：一个处理和结构化医疗知识的知识图谱构建管道，以及一个基于 Camel 的多代理系统，用于实现层次化的医疗决策。这一设计与现实世界的医疗实践相似，在那里，全科医生与专家协作提供全面的患者护理（见图 [2](https://arxiv.org/html/2412.16833v2#Sx1.F2 "Figure 2 ‣ Introduction ‣ KG4Diagnosis: A Hierarchical Multi-Agent LLM Framework with Knowledge Graph Enhancement for Medical Diagnosis")）。

### 知识图谱构建管道

该框架实现了一个三阶段过程，用于自动化构建医疗知识图谱。最初，医疗文档被划分为符合知识图谱上下文约束的数据块。随后，采用语义驱动的实体和关系提取模块，从这些数据块中提取实体和关系。该过程利用 BioBERT，这是一种专为生物医学领域设计的模型，确保精确提取医疗实体并识别它们之间的关系。在接下来的阶段，根据提取的实体和关系构建知识图谱，从而促进其自动生成。我们还通过使用 LLMs 来识别更广泛的、上下文感知的实体和关系，进一步增强医疗知识图谱，这些知识补充了 BioBERT 在特定领域的提取。

在最后阶段，知识图谱的扩展和验证将通过专家评审进行。医学专家将手动验证已构建的关系，经过验证的知识将用于训练大规模模型，推动未来的知识扩展。

知识图谱构建管道的各个部分的细节如下：

#### 第一阶段：数据块划分与分段

在第一阶段，医疗文档根据上下文约束进行数据块划分。设 $D=\{d_{1},d_{2},\dots,d_{n}\}$ 表示一组医疗文档。每个文档 $d_{i}$ 被划分为 $m$ 个数据块：

|  | $C_{i}=\{c_{i1},c_{i2},\dots,c_{im}\}$ |  |
| --- | --- | --- |

这些数据块 $c_{ij}$ 是通过基于上下文的分段规则生成的。分段过程可以用数学公式表示为：

|  | $f_{\text{seg}}(d_{i})\rightarrow C_{i}$ |  |
| --- | --- | --- |

其中 $f_{\text{seg}}$ 是一个函数，将文档 $d_{i}$ 映射到一组数据块 $C_{i}$。

#### 第二阶段：语义驱动的实体和关系提取

该流程利用 BioBERT 的上下文嵌入与医学本体（如 SNOMED-CT 和 UMLS）相结合，从分段的数据块中提取实体和关系。提取过程可以表示如下：

+   •

    *实体提取：* 提取的实体集合 $E$ 定义为：

    |  | $E=\{e_{1},e_{2},\dots,e_{n}\}$ |  |
    | --- | --- | --- |

    其中 $E$ 表示医学实体的集合，如疾病、药物、症状等。

+   •

    *关系提取：* 实体 $e_{i}$ 和 $e_{j}$ 之间的关系集合 $R$ 表示为：

    |  | $R=\{(e_{i},r,e_{j})\mid e_{i},e_{j}\in E\}$ |  |
    | --- | --- | --- |

    其中 $r$ 表示从医学文本中提取的实体 $e_{i}$ 和 $e_{j}$ 之间的关系。

在这一阶段，BioBERT 捕捉医学文本的语义含义，并将其映射到标准化的医学本体，确保实体和关系的准确提取。

#### 第3阶段：知识图谱构建

一旦提取了实体和关系，就会构建知识图谱。知识图谱可以表示为图 $G$，其中：

|  | $G=(V,E)$ |  |
| --- | --- | --- |

这里，节点集合 $V=\{e_{1},e_{2},\dots,e_{k}\}$ 表示医学实体，边集合 $E=\{r_{1},r_{2},\dots,r_{l}\}$ 表示这些实体之间的关系。

知识图谱的构建基于提取的实体和关系。因此，知识图谱可以表示为：

|  | $G=(V,E)\quad\text{其中}\quad V=E\text{ 且 }E=R$ |  |
| --- | --- | --- |

节点表示实体，边表示关系。

#### 第4阶段：LLM增强的知识图谱

我们利用 LLMs 来增强医学知识图谱，通过识别超出 BioBERT 提取能力的实体和关系。虽然 BioBERT 在生物医学领域内精确地进行特定领域的提取，但 LLMs 通过提供更广泛的、基于上下文的语义提取，尤其是从复杂或模糊的医学文本中，做出贡献。丰富的实体和关系存储在专用数据库中，并集成到知识图谱中，随后对知识图谱进行优化，以支持使用 LLMs 进行推理。这一增强的知识图谱通过多代理系统和 LLM 驱动的诊断推理的协同能力，支持更强大的推理和决策，推动高级诊断工作流的实施。

#### 第5阶段：人类引导推理

在最后阶段，专家验证对于确保知识图谱中构建的关系和实体的质量与准确性至关重要。专家验证过程涉及主动学习和强化学习技术，以用经过验证和可靠的信息扩展图谱。

+   •

    *专家验证关系：* 医学专家手动审查提取的实体之间的关系 $R$，以验证其临床相关性。如果某个关系 $(e_{i},r,e_{j})$ 被确认准确，则保留在知识图谱中。如果某个关系被认为无效或不确定，则会被修正或移除。

+   •

    *专家验证的图谱扩展关系：* 在验证后，知识图谱通过融入新的、经过专家验证的实体和关系进行扩展。经过验证的图谱通过这些确认的连接得到了丰富，提升了图谱的可靠性和全面性。

    |  | $G_{\text{expanded}}=G\cup\text{验证过的实体和关系}$ |  |
    | --- | --- | --- |

    其中 $G_{\text{expanded}}$ 表示扩展后的知识图谱，包含了先前提取的以及专家验证过的实体和关系。

通过这个专家指导的验证与扩展过程，知识图谱逐渐演变为一个强大且可靠的医学研究和临床决策资源。

### 医学诊断的层次化多智能体框架

为了应对医学诊断推理的复杂性，我们开发了一个层次化的多智能体框架来处理用户的诊断查询。该框架整合了一个全科医生大语言模型（GP-LLM）和多个领域特定的顾问大语言模型（Consultant-LLMs）。诊断过程在数学上建模如下：

#### GP-LLM: 主要诊断智能体

GP-LLM 作为分析用户查询的初始接口。设用户查询为 $q\in Q$，其中 $Q$ 是所有可能用户查询的集合。对于产生初步诊断 $x$ 的查询 $q$，其诊断置信度定义为：

|  | $P_{\text{GP}}(x\mid q)=f_{\text{GP}}(q)$ |  | (1) |
| --- | --- | --- | --- |

其中 $P_{\text{GP}}(x\mid q)\in[0,1]$ 是 GP-LLM 对诊断 $x$ 赋予的置信度，$f_{\text{GP}}$ 表示基于广泛知识库的概率诊断函数。

当 GP-LLM 启动转诊时：

|  | $P_{\text{GP}}(x\mid q)<\tau\quad\text{或}\quad x\in X_{s}$ |  | (2) |
| --- | --- | --- | --- |

这里：

+   •

    $\tau$ 是转诊的置信度阈值（设定为 $0.7$）。

+   •

    $X_{s}\subset X$ 是需要专门化知识的诊断子集。

GP-LLM 的输出表达为：

|  |  $\text{Output}_{\text{GP}}=\begin{cases}\text{转诊到 Consultant-LLM},&% \text{如果 }P_{\text{GP}}(x\mid q)<\tau\;\text{或}\;x\in X_{s},\\ \text{诊断: }x,&\text{否则。}\end{cases}$  |  | (3) |
| --- | --- | --- | --- |

#### Consultant-LLMs: 专业诊断智能体

每个 Consultant-LLM 都针对特定的医学领域进行优化，例如风湿病学。设 $\text{Agent}_{i}$ 表示第 $i^{\text{th}}$ 个 Consultant-LLM，其中 $i=1,2,\dots,n$ 且 $n=4$（心脏病学、神经学、内分泌学和风湿病学）。$\text{Agent}_{i}$ 在查询 $q$ 中诊断疾病 $y$ 的置信度函数定义为：

|  | $P_{\text{Agent}_{i}}(y\mid q)=f_{\text{Agent}_{i}}(q)$ |  | (4) |
| --- | --- | --- | --- |

其中 $P_{\text{Agent}_{i}}(y\mid q)\in[0,1]$ 且 $f_{\text{Agent}_{i}}$ 是基于特定领域训练数据集和临床指南的概率诊断函数。

对于需要多个代理协同推理的案例，最终的诊断置信度计算如下：

|  | $P_{\text{final}}(z\mid q)=\sum_{i=1}^{n}w_{i}P_{\text{Agent}_{i}}(z\mid q)$ |  | (5) |
| --- | --- | --- | --- |

其中 $w_{i}$ 表示分配给 $\text{Agent}_{i}$ 贡献的权重，经过归一化处理，使得 $\sum_{i=1}^{n}w_{i}=1$。

#### 代理间沟通协议

转介和沟通流程确保了案例的无缝转移和协同优化。令 $T(A,B,q)$ 表示将用户查询 $q$ 从代理 $A$ 转移到代理 $B$。转移函数建模如下：

|  | $T(A,B,q)=\phi(q),\quad\phi:Q\to Q^{\prime}$ |  | (6) |
| --- | --- | --- | --- |

其中 $\phi$ 将 $q$ 转换为与接收代理 $B$ 兼容的格式。反馈给GP-LLM以更新其知识库 $K_{\text{GP}}$，如下所示：

|  | $K_{\text{GP}}^{(t+1)}=K_{\text{GP}}^{(t)}+\Delta K$ |  | (7) |
| --- | --- | --- | --- |

其中 $\Delta K$ 是从顾问LLM中获得的增量知识。

#### 转介决策阈值

转介决策在数学上定义如下：

|  | $\text{Referral}=\begin{cases}1,&\text{如果 }P_{\text{GP}}(x\mid q)<\tau\;\text{或}\;x\in X_{s}\\ 0,&\text{否则.}\end{cases}$ |  | (8) |
| --- | --- | --- | --- |

这里：

+   •

    $\text{Referral}=1$ 表示升级到顾问LLM。

+   •

    $\text{Referral}=0$ 表示查询保持在GP-LLM中。

#### 多代理协作下的高级诊断

对于需要多个顾问LLM输入的复杂查询，最终的诊断置信度计算如下：

|  | $P_{\text{final}}(z\mid q)=\frac{1}{n}\sum_{i=1}^{n}P_{\text{Agent}_{i}}(z\mid q),\quad z\in Z$ |  | (9) |
| --- | --- | --- | --- |

其中 $Z\subset X$ 表示需要多领域专业知识的复杂诊断空间。

#### 总结

该建模形式化了分层多代理框架中的诊断推理。置信度函数 $P_{\text{GP}}(x\mid q)$、$P_{\text{Agent}_{i}}(y\mid q)$ 和 $P_{\text{final}}(z\mid q)$ 分别定义了GP-LLM、各个顾问LLM以及协作多代理系统的概率输出。置信度阈值（$\tau=0.7$）确保在必要时能够准确且高效地升级到专门的诊断代理。

### 未来培训和评估工作

该系统的训练方法涵盖了362种常见疾病，涉及多个医学专业领域，代表了医学诊断中的广泛范围。训练过程被战略性地设计为多方面的，结合了通用医学知识和专业领域的专长。对于每个疾病类别，我们为相应的专科代理人实施了有针对性的微调协议，确保深入的领域特定知识，同时在更广泛的框架内保持一致性。

![参见说明](img/0bd2714c72718f8d36bc5c61892a1c11.png)

图 3：示例 1 说明了肥胖的复杂性，突出其核心病症以及与患者状况和减肥手术等相关因素。它还展示了相关药物和 BMI 分类，强调了这些元素在理解肥胖这一多维健康问题中的相互联系。

![参见说明](img/6fca17f500088f1a19c718a9b3b8bdd5.png)

图 4：示例 2 展示了知识图在肥胖领域的专业性。该知识图突出了某些药物，如奥司美克（Ozempic），不仅有助于体重管理，还能降低心血管风险。肥胖、2型糖尿病和心血管疾病之间的关系被描绘出来，显示了它们的共同症状、治疗方法和共病情况。图表强调了药物在应对复杂健康问题中的多方面作用。

![参见说明](img/2e5f3e10b61de6e953bf5894187e4f71.png)

图 5：KG4Diagnosis 完整医学知识图的可视化。节点代表不同的医学概念，如行为、症状、类别和病情，颜色图例予以指示。边缘表示这些概念之间的关系，使得结构化表示和高级诊断推理成为可能。密集连接的中央区域突出显示了治疗、症状和诊断之间的核心交互，而外围节点提供了额外的上下文细节。这种层次结构将医学数据整合在一起，促进多代理协作和人类引导的推理。

图[3](https://arxiv.org/html/2412.16833v2#Sx2.F3 "图 3 ‣ 未来训练和评估工作 ‣ 方法论 ‣ KG4Diagnosis: 基于知识图谱增强的分层多智能体大语言模型框架用于医学诊断")和[4](https://arxiv.org/html/2412.16833v2#Sx2.F4 "图 4 ‣ 未来训练和评估工作 ‣ 方法论 ‣ KG4Diagnosis: 基于知识图谱增强的分层多智能体大语言模型框架用于医学诊断")中展示的知识图谱示例，展示了两种先进的肥胖药物（奥泽姆匹克和维戈维），并演示了我们的框架如何有效模拟真实世界的临床咨询。图[5](https://arxiv.org/html/2412.16833v2#Sx2.F5 "图 5 ‣ 未来训练和评估工作 ‣ 方法论 ‣ KG4Diagnosis: 基于知识图谱增强的分层多智能体大语言模型框架用于医学诊断")展示的知识图谱的完整结构揭示了疾病、症状和诊断模式之间复杂的相互联系。可视化结果显示了医学知识组织的层次结构，呈现了从一般诊断模式到专门医学领域的清晰路径。这一结构使得高效的知识导航成为可能，并支持系统的分层决策过程。

我们的持续学习机制通过动态的智能体互动和反馈回路来增强初始训练。这种方法使得系统能够随着时间的推移不断发展和优化其诊断能力，适应通过智能体协作识别的新医学见解和模式。该框架使用 PyTorch 实现神经网络组件，并使用 Neo4j 管理知识图谱，目前涵盖了知识库中的所有 362 种疾病，并为知识扩展提供了结构化路径。

鉴于该框架在医学诊断中的全面性和创新方法，目前正在开发一个全面的基准，用于评估多个维度的性能，包括诊断准确性、幻觉预防和多智能体协调效率。该基准将提供评估医疗 AI 系统的标准化指标，并将在完成后通过我们的 GitHub 仓库公开。即将发布的基准旨在为评估医疗应用中的分层多智能体系统建立新的标准，促进该关键领域未来的研究和发展。

## 讨论

KG4Diagnosis 的开发与评估，涵盖了多个医学专科中的 362 种常见疾病，提供了将分层多智能体系统与医学知识图谱集成应用于医疗保健的重大见解。我们的综合框架展示了既有前景广阔的能力，也有需要进一步研究的重要挑战。

### 技术成就与创新

自动化知识图谱构建与层级多代理架构的结合，在解决医学人工智能系统的关键挑战方面展示了令人鼓舞的结果。我们的框架在保持诊断准确性的同时防止幻觉，代表了传统单代理方法的重大进步。特别值得注意的是，我们的语义驱动实体提取和关系重建模块在处理复杂医学术语和关系方面的有效性，相比传统方法实现了更高的精确度。

通过MAS实现的层级多代理结构在管理复杂的医学案例方面尤为有价值。全科医生代理有效地进行病例分诊并与专业代理协调，其操作方式与现实中的医学实践相似，有可能减少每个病例都需要进行完整专业咨询时产生的计算开销。此外，我们通过多层验证机制防止幻觉的方式，利用知识图谱作为有效的约束系统，显著减少了与独立LLM实现相比的错误诊断。

### 系统适应性与扩展性

生成的知识图谱结构展示了不同疾病实体、症状和诊断模式之间的复杂相互关系。这一全面的覆盖支持了高效的知识导航和层级决策过程。我们框架的模块化设计在融入新的医学领域和知识方面展现了特别的优势，使其非常适合医学知识的动态性质。

然而，扩展性分析揭示了重要的考量因素。尽管层级结构通过其分层决策过程有效地管理计算资源，但随着医学领域的扩展，系统在协调多个专业代理时面临日益复杂的挑战。这突显了未来版本中对更复杂协调机制的需求。

### 限制与挑战

系统的表现可能会受到基础知识图谱的质量和全面性的影响，尤其是在罕见或复杂的医学条件下。在处理医学知识快速发展的边缘案例或处理训练数据中未充分表示的罕见疾病组合时，仍然存在挑战。

未来的研究将包括在最先进的MedQA数据集上进行实验，以验证我们框架的优越性。MEDQA可以用于执行基准测试，从而评估LLM完美功能的可重复性。同时，这将使我们能够将我们的框架与其他知名模型进行基准对比，如ESM-1b、Med-PaLM和BioGPT。通过在MedQA中的性能评估，我们不仅旨在展示我们系统的竞争优势，还希望发现进一步改进的领域。此外，系统在知识图谱构建和代理训练中对高质量医学数据的高度依赖，给在数据资源有限的地区部署带来了挑战。尽管我们的框架在已充分记录的医学条件下表现出色，但在处理罕见疾病或异常症状组合时，其有效性仍需要进一步研究。

## 相关工作

基于规则和本体驱动的方法：医学KG构建和推理的最新进展催生了各种方法论（Lu等人 [2024](https://arxiv.org/html/2412.16833v2#bib.bib17); Li等人 [2020](https://arxiv.org/html/2412.16833v2#bib.bib15); Peng等人 [2023](https://arxiv.org/html/2412.16833v2#bib.bib25)），每种方法都有独特的优势，同时面临着不同的挑战。传统的医学KG构建方法主要依赖基于规则的系统和本体驱动的技术。虽然这些方法在产生可解释的输出和通过既定医学本体保持结构一致性方面表现出色，但它们在扩展性和处理非结构化数据方面存在显著局限性（Abdulla、Mukherjee和Ranganathan [2023](https://arxiv.org/html/2412.16833v2#bib.bib1)）。

深度学习和预训练模型：深度学习方法的出现，特别是BERT和BioBERT等预训练语言模型（Masoumi等人 [2024](https://arxiv.org/html/2412.16833v2#bib.bib19)），显著提高了临床文本中的信息提取能力。然而，这些模型通常在处理特定领域的细微差别时存在困难，并且需要大量的计算资源（Alsentzer等人 [2019](https://arxiv.org/html/2412.16833v2#bib.bib4)）。LLM代表了处理非结构化医学数据的一个进展。尽管近期的研究展示了它们在生成结构化知识和理解复杂医学关系方面的潜力，但关于幻觉和验证的问题仍然存在（Brown等人 [2020](https://arxiv.org/html/2412.16833v2#bib.bib9)）。Med-HALT基准和对比解码技术已成为解决这些问题的有前途的方法（Liu等人 [2023](https://arxiv.org/html/2412.16833v2#bib.bib16)）。此外，将LLM与多代理系统（MAS）集成，在医学应用中展现出特别的潜力（Singhal等人 [2023b](https://arxiv.org/html/2412.16833v2#bib.bib30)）。

混合符号-神经方法：结合符号推理和神经架构的混合方法因其在可解释性和适应性之间的平衡而受到关注。这些系统将基于知识的推理与数据驱动的学习结合，尽管它们需要精心策划的输入，并面临计算可扩展性挑战（Wu, Zhang 和 Lin [2023](https://arxiv.org/html/2412.16833v2#bib.bib35)）。最近的多模态整合创新已扩展了KG的能力，能够整合多种数据类型，包括临床记录、医学影像和实验室结果，尽管标准化和融合的挑战仍然存在（Zhou 等人 [2022](https://arxiv.org/html/2412.16833v2#bib.bib39)）。

医学领域LLMs的进展：最近医学LLMs的进展显著提升了医疗保健中的自然语言理解领域。像ESM-1b（Rives 等人 [2021](https://arxiv.org/html/2412.16833v2#bib.bib27)）这样的模型最初是为了蛋白质表示而开发的，已在生物医学应用中展示了潜力，利用进化规模建模高精度地分析生物序列。另一方面，Med-PaLM（Singhal 等人 [2023a](https://arxiv.org/html/2412.16833v2#bib.bib29)）代表了针对临床使用的通用LLMs的专门化改编，专注于回答医学问题并在结构化数据集中进行推理。类似地，MediTron（Bosselut 等人 [2024](https://arxiv.org/html/2412.16833v2#bib.bib8)）和BioGPT（Luo 等人 [2022](https://arxiv.org/html/2412.16833v2#bib.bib18)）已被设计用来提取生物医学知识，其中MediTron在多模态数据整合方面表现出色，而BioGPT则专门针对生物医学文献进行了微调，以执行实体和关系抽取任务。最近开发的GPT-4-medprompt（Nori 等人 [2023](https://arxiv.org/html/2412.16833v2#bib.bib21)）通过整合特定领域的提示以引导推理，进一步推动了医学LLMs的边界，提高了上下文准确性并减少了医学应用中的幻觉现象。

分层多智能体架构：分层多智能体架构的出现代表了一个特别有前景的方向。Pandey 等人（Pandey, Amod 和 Kumar [2024](https://arxiv.org/html/2412.16833v2#bib.bib24)）展示了这种架构能够有效地模拟现实世界中的医疗系统，通用智能体负责初步评估，而专业智能体则管理领域特定的诊断。该方法不仅提高了诊断准确性，还增强了系统的可扩展性和可靠性。

尽管取得了这些进展，该领域仍面临若干关键挑战。处理非结构化医疗数据仍然是一个重大障碍，要求采用更复杂的方法以实现准确的信息提取和结构化（Avula等人 [2022](https://arxiv.org/html/2412.16833v2#bib.bib6)）。在医学情境中预防和检测LLM幻觉问题需要在验证机制和验证协议方面持续创新（Huang等人 [2023](https://arxiv.org/html/2412.16833v2#bib.bib14)）。此外，整合多模态医疗信息仍面临数据标准化和融合方面的挑战。医学系统中多个专门智能体的协调需要进一步完善通信协议和决策框架。此外，开发全面且标准化的医学知识图谱系统评估协议仍然是一个积极的研究领域，这对于确保这些系统在临床应用中的可靠性和有效性至关重要。这些相互关联的挑战为创新解决方案提供了机会，能够结合各种方法的优势，同时解决各自的局限性。

## 结论

本文提出了KG4Diagnosis，一个创新的分层多智能体框架，将自动化知识图谱构建与专门化的LLMs（大语言模型）结合用于医学诊断。我们的实现涵盖了362种常见疾病，展示了将知识图谱与分层多智能体架构相结合，在应对医学AI系统中关键挑战方面的有效性。该框架的创新之处在于其三阶段知识图谱构建管道和基于层级的智能体结构，其中语义驱动的处理和人类引导的推理为系统创建了强大的知识基础，而多层智能体架构则反映了现实世界医学实践。该系统通过多重验证层有效防止了幻觉问题，并通过定向专家咨询有效管理了计算资源。尽管我们当前的实现显示出良好的结果，我们正在开发全面的基准，以为社区提供标准化的评估指标。这项工作不仅为当前医学AI挑战提供了实际解决方案，还为未来在医疗应用中分层多智能体系统的开发奠定了基础，可能会改善医疗服务和患者的治疗效果。

## 参考文献

+   Abdulla, Mukherjee, 和 Ranganathan (2023) Abdulla, K.; Mukherjee, S.; 和 Ranganathan, P. 2023. 《提升知识图谱的多模态数据整合：当前挑战与机遇》. *大数据期刊*, 10(1): 1–15.

+   Al Khatib 等人（2024）Al Khatib, H. S.; Neupane, S.; Kumar Manchukonda, H.; Golilarz, N. A.; Mittal, S.; Amirlatifi, A.; 和 Rahimi, S. 2024. 以患者为中心的知识图谱：当前方法、挑战和应用综述。*人工智能前沿*，7: 1388479。

+   Alam, Giglou 和 Malik（2023）Alam, F.; Giglou, H. B.; 和 Malik, K. M. 2023. 面向循证医学的自动化临床知识图谱生成框架。*专家系统与应用*，233: 120964。

+   Alsentzer 等人（2019）Alsentzer, E.; Murphy, J.; Boag, W.; Weng, W.; Jin, D.; Naumann, T.; 和 McDermott, M. 2019. 公开可用的临床 BERT 嵌入。arXiv:1901.08746。

+   Amos 等人（2020）Amos, L.; Anderson, D.; Brody, S.; Ripple, A.; 和 Humphreys, B. L. 2020. UMLS 用户与用途：当前概述。*美国医学信息学会杂志*，27(10): 1606–1611。

+   Avula 等人（2022）Avula, R.; 等人。2022. 通过先进的数据挖掘技术在医疗保健中的数据驱动决策：应用与局限性调查。*国际应用机器学习与计算智能杂志*，12(4): 64–85。

+   Bonner 等人（2022）Bonner, S.; Barrett, I. P.; Ye, C.; Swiers, R.; Engkvist, O.; Bender, A.; Hoyt, C. T.; 和 Hamilton, W. L. 2022. 与药物发现相关的生物医学数据集综述：知识图谱视角。*生物信息学简报*，23(6): bbac404。

+   Bosselut 等人（2024）Bosselut, A.; Chen, Z.; Romanou, A.; Bonnet, A.; Hernández-Cano, A.; Alkhamissi, B.; Matoba, K.; Salvi, F.; Pagliardini, M.; Fan, S.; 等人。2024. MEDITRON：适应临床实践的开放医疗基础模型。

+   Brown 等人（2020）Brown, T.; Mann, B.; Ryder, N.; Subbiah, M.; Kaplan, J.; Dhariwal, P.; Neelakantan, A.; Shyam, P.; Sastry, G.; 和 Askell, A. 2020. 语言模型是少样本学习者。arXiv:2005.14165。

+   Chang 和 Mostafa（2021）Chang, E.; 和 Mostafa, J. 2021. SNOMED CT 的使用，2013-2020：文献综述。*美国医学信息学会杂志*，28(9): 2017–2026。

+   Du 等人（2022）Du, K.; Yang, B.; Wang, S.; Chang, Y.; Li, S.; 和 Yi, G. 2022. 基于注意力机制和图卷积网络特征融合的制造业知识图谱关系抽取。*基于知识的系统*，255: 109703。

+   García-Ferrero 等人（2024）García-Ferrero, I.; Agerri, R.; Salazar, A. A.; Cabrio, E.; de la Iglesia, I.; Lavelli, A.; Magnini, B.; Molinet, B.; Ramirez-Romero, J.; Rigau, G.; 等人。2024. 医学 mT5：面向医学领域的开源多语言文本到文本大语言模型。*arXiv 预印本 arXiv:2404.07613*。

+   Guo 等人（2024）Guo, T.; Chen, X.; Wang, Y.; Chang, R.; Pei, S.; Chawla, N. V.; Wiest, O.; 和 Zhang, X. 2024. 基于大语言模型的多智能体：进展与挑战综述。*arXiv 预印本 arXiv:2402.01680*。

+   黄等（2023）黄, L.; 余, W.; 马, W.; 钟, W.; 冯, Z.; 王, H.; 陈, Q.; 彭, W.; 冯, X.; 秦, B.; 等. 2023. 大型语言模型中的幻觉调查：原理、分类、挑战与开放问题. *ACM信息系统交易*.

+   李等（2020）李, L.; 王, P.; 阎, J.; 王, Y.; 李, S.; 姜, J.; 孙, Z.; 唐, B.; 常, T.-H.; 王, S.; 等. 2020. 真实世界数据医学知识图谱：构建与应用. *医学中的人工智能*, 103: 101817.

+   刘等（2023）刘, Y.; 奥特, M.; 戈亚尔, N.; 杜, J.; 乔希, M.; 陈, D.; 利维, O.; 路易斯, M.; 泽特尔莫耶, L.; 和 斯托扬诺夫, V. 2023. Med-HALT：评估医学大语言模型中的幻觉. arXiv:2410.15702.

+   陆等（2024）陆, Z.; 阿弗里迪, I.; 康, H. J.; 鲁奇金, I.; 和 郑, X. 2024. 调查神经符号方法在可靠物联网人工智能中的应用. *可靠智能环境学报*, 10(3): 257–279.

+   罗等（2022）罗, R.; 孙, L.; 夏, Y.; 秦, T.; 张, S.; 潘, H.; 和 刘, T.-Y. 2022. BioGPT：用于生物医学文本生成与挖掘的生成预训练变换器. *生物信息学简报*, 23(6): bbac409.

+   马苏米等（2024）马苏米, S.; 阿米尔哈尼, H.; 萨德吉安, N.; 和 沙赫拉兹, S. 2024. 自然语言处理（NLP）促进医学研究中的摘要评审：BioBERT在探索NLP在医学研究中20年应用中的作用. *系统综述*, 13(1): 107.

+   莫等（2024）莫, F.; 查普林, J. C.; 桑德森, D.; 马丁内斯-阿雷利亚诺, G.; 和 拉切夫, S. 2024. 语义模型和知识图谱作为制造系统重配置的推动因素. *机器人与计算机集成制造*, 86: 102625.

+   诺里等（2023）诺里, H.; 李, Y. T.; 张, S.; 卡里甘, D.; 埃德加, R.; 富西, N.; 金, N.; 拉尔森, J.; 李, Y.; 刘, W.; 等. 2023. 通用基础模型能否超越专用调优？医学领域的案例研究. *arXiv预印本 arXiv:2311.16452*.

+   OpenAI（2022）OpenAI. 2022. ChatGPT：优化对话语言模型. OpenAI技术博客.

+   OpenAI（2023）OpenAI. 2023. GPT-4技术报告. arXiv:2303.08774.

+   潘迪, 阿莫德, 和 库马尔（2024）潘迪, H. G.; 阿莫德, A.; 和 库马尔, S. 2024. 推进医疗自动化：用于医疗需求合理化的多智能体系统. 在 德梅尔-福什曼, D.; 阿纳尼亚多, S.; 美娃, M.; 罗伯茨, K.; 和 筑井, J., 主编, *第23届生物医学自然语言处理研讨会论文集*, 39–49. 曼谷, 泰国: 计算语言学协会.

+   彭等（2023）彭, C.; 夏, F.; 纳塞里帕尔萨, M.; 和 奥斯本, F. 2023. 知识图谱：机遇与挑战. *人工智能评论*, 56(11): 13071–13102.

+   Qian等（2024）Qian, J.; Jin, Z.; Zhang, Q.; Cai, G.; 和 Liu, B. 2024. 基于下一代智能和大型模型Med-PaLM 2的肝癌问答系统。*国际计算机科学与信息技术期刊*，2(1): 28–35。

+   Rives等（2021）Rives, A.; Meier, J.; Sercu, T.; Goyal, S.; Lin, Z.; Liu, J.; Guo, D.; Ott, M.; Zitnick, C. L.; Ma, J.; 等 2021. 生物结构与功能通过将无监督学习扩展到2.5亿蛋白质序列中涌现。*美国国家科学院院刊*，118(15): e2016239118。

+   Shuifa等（2023）Shuifa, S.; Xiaolong, L.; Weisheng, L.; Dajiang, L.; Sihui, L.; Liu, Y.; 和 Yirong, W. 2023. 图神经网络在知识图谱推理中的应用综述。*计算机科学与技术前沿期刊*，17(1): 27。

+   Singhal等（2023a）Singhal, K.; Azizi, S.; Tu, T.; Mahdavi, S. S.; Wei, J.; Chung, H. W.; Scales, N.; Tanwani, A.; Cole-Lewis, H.; Pfohl, S.; 等 2023a. 大语言模型编码临床知识。*自然*，620(7972): 172–180。

+   Singhal等（2023b）Singhal, K.; Tu, T.; Gottweis, J.; Sayres, R.; 和 Wulczyn, E. 2023b. 朝着专家级医疗问答迈进：基于大语言模型的研究。*自然医学*，29(1): 50–58。

+   Tang等（2023）Tang, X.; Chi, G.; Cui, L.; Ip, A. W.; Yung, K. L.; 和 Xie, X. 2023. 探索基于知识图谱的飞机故障诊断构建与应用研究。*传感器*，23(11): 5295。

+   Tonmoy等（2024）Tonmoy, S.; Zaman, S.; Jain, V.; Rani, A.; Rawte, V.; Chadha, A.; 和 Das, A. 2024. 大语言模型中的幻觉缓解技术综述。*arXiv预印本 arXiv:2401.01313*。

+   Touvron等（2023）Touvron, H.; Lavril, T.; Izacard, G.; Martinet, X.; Lachaux, M.-A.; Lacroix, T.; Rozière, B.; Goyal, N.; Hambro, E.; Azhar, F.; 等 2023. LLaMA：开放高效的基础语言模型。*arXiv预印本 arXiv:2302.13971*。

+   Wu等（2024）Wu, J.; Zhu, J.; Qi, Y.; Chen, J.; Xu, M.; Menolascina, F.; 和 Grau, V. 2024. 医疗图RAG：通过图检索增强生成推动安全的医疗大语言模型。*arXiv预印本 arXiv:2408.04187*。

+   Wu, Zhang, 和 Lin（2023）Wu, Z.; Zhang, Y.; 和 Lin, X. 2023. 医疗知识图谱构建中的可扩展性挑战。*IEEE医学信息学学报*，14(2): 120–132。

+   Zhang等（2024）Zhang, J.; Zan, H.; Wu, S.; Zhang, K.; 和 Huo, J. 2024. 带增量学习机制的自适应图神经网络用于知识图谱推理。*电子学*，13(14): 2778。

+   Zhang（2021）Zhang, Y. 2021. 基于图神经网络的知识推理。*乔治亚理工学院：美国乔治亚州亚特兰大*。

+   Zhang, Li, 和 Wang（2023）Zhang, Y.; Li, X.; 和 Wang, J. 2023. 用于欺诈检测的知识图谱：以欺诈性关联方交易为例。在*知识发现与数据挖掘进展*，182–194。Springer。

+   Zhou 等人（2022）Zhou, J.; Cui, G.; Zhang, Z.; Yang, C.; Liu, Z.; Wang, L.; Li, C.; 和 Sun, M. 2022. 图神经网络：方法与应用综述。*AI Open*, 1(1): 1–12。
