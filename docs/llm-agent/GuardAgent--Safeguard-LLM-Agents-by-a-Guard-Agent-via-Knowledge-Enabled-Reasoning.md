<!--yml  

category: 未分类  

date: 2025-01-11 12:33:43  

-->  

# GuardAgent：通过知识驱动推理保护LLM智能体  

> 来源：[https://arxiv.org/html/2406.09187/](https://arxiv.org/html/2406.09187/)  

Zhen Xiang^(1∗)  Linzhi Zheng²  Yanjie Li³  Junyuan Hong⁴  Qinbin Li⁵  Han Xie⁶  

Jiawei Zhang¹  Zidi Xiong¹  Chulin Xie¹  Carl Yang⁶  Dawn Song⁵  Bo Li^(17)

¹UIUC  ²清华大学  ³香港理工大学  

⁴德州大学奥斯汀分校  ⁵加利福尼亚大学伯克利分校  ⁶埃默里大学  ⁷芝加哥大学 通信作者：Zhen Xiang ⟨zhen.xiang.lance@gmail.com⟩和Bo Li ⟨bol@uchicago.edu⟩。  

###### 摘要  

大型语言模型（LLMs）的快速发展促进了LLM驱动的智能体在众多应用中的部署，同时也引发了有关其安全性和可信度的新问题。此外，现有的增强LLM安全性的方法由于目标和输出模式的多样性，无法直接应用于LLM驱动的智能体。在本文中，我们提出了GuardAgent，这是第一个作为其他LLM智能体保护屏障的LLM智能体。具体而言，GuardAgent通过检查目标LLM智能体的输入/输出是否满足用户定义的一系列保护请求（例如安全规则或隐私政策），来监督目标LLM智能体。GuardAgent包括两个步骤：1）通过分析提供的保护请求创建任务计划，2）根据任务计划生成保护屏障代码，并通过调用API或使用外部引擎执行该代码。在这两个步骤中，LLM作为核心推理组件，辅以从内存模块中检索的上下文示范。这样的知识驱动推理使得GuardAgent能够理解各种文本保护请求，并准确地将其“翻译”成可执行的代码，从而提供可靠的保护屏障。此外，GuardAgent配备了一个可扩展的工具箱，包含各种功能和API，并且不需要额外的LLM训练，这突显了其强大的泛化能力和低运行开销。除了GuardAgent，我们还提出了两个新基准：EICU-AC基准，用于评估面向医疗保健智能体的隐私相关访问控制，和Mind2Web-SC基准，用于评估Web智能体的安全性。我们展示了GuardAgent在这两个基准上的有效性，分别在两类智能体的无效输入和输出调节中，达到了98.7%和90.0%的保护准确率。我们还展示了GuardAgent能够根据新兴的LLM智能体和保护请求定义新的功能，这突显了其强大的泛化能力。  

![Refer to caption](img/39812941eca41c307acaff83c117efdf.png)  

图1：GuardAgent作为目标LLM代理的保护栏的示意图。GuardAgent的输入包括：a) 由目标代理规范提供的信息集的保护请求，b) 目标代理的测试时输入和输出。GuardAgent首先根据从记忆中检索到的几次示范生成一个行动计划。然后，基于示范和可调用函数列表，按照行动计划生成一个保护栏代码。如果GuardAgent检测到违反保护请求的行为，则目标代理的输出/行动将被拒绝。

## 1 引言

由大语言模型（LLMs）赋能的AI代理在多个应用领域展现出了卓越的表现，包括金融[[24](https://arxiv.org/html/2406.09187v1#bib.bib24)]、医疗[[2](https://arxiv.org/html/2406.09187v1#bib.bib2), [17](https://arxiv.org/html/2406.09187v1#bib.bib17), [22](https://arxiv.org/html/2406.09187v1#bib.bib22), [18](https://arxiv.org/html/2406.09187v1#bib.bib18), [11](https://arxiv.org/html/2406.09187v1#bib.bib11)]、日常工作[[4](https://arxiv.org/html/2406.09187v1#bib.bib4), [5](https://arxiv.org/html/2406.09187v1#bib.bib5), [28](https://arxiv.org/html/2406.09187v1#bib.bib28), [27](https://arxiv.org/html/2406.09187v1#bib.bib27)]，以及自动驾驶[[3](https://arxiv.org/html/2406.09187v1#bib.bib3), [8](https://arxiv.org/html/2406.09187v1#bib.bib8), [12](https://arxiv.org/html/2406.09187v1#bib.bib12)]。对于每个用户查询，这些代理通常使用LLM进行任务规划，利用LLM的推理能力，并通过可选的长期记忆支持（来自先前的使用案例）[[10](https://arxiv.org/html/2406.09187v1#bib.bib10)]。然后，通过调用外部工具（例如，通过API）并可能与环境进行交互[[23](https://arxiv.org/html/2406.09187v1#bib.bib23)]来执行提出的计划。

不幸的是，目前LLM代理的发展主要集中在它们在解决复杂任务方面的有效性，而在很大程度上忽视了它们被滥用的潜力，这可能导致有害的后果。例如，如果被未经授权的人员滥用，医疗LLM代理可能会轻易泄露患者的机密信息[[25](https://arxiv.org/html/2406.09187v1#bib.bib25)]。实际上，一些LLM代理，特别是在自动驾驶等高风险应用中使用的代理，配备了安全控制，以防止执行不期望的危险行为[[12](https://arxiv.org/html/2406.09187v1#bib.bib12), [6](https://arxiv.org/html/2406.09187v1#bib.bib6)]。然而，这些特定任务的保护栏是硬编码到LLM代理中的，因此无法推广到具有不同保护请求（例如，针对隐私而非安全的医疗代理）其他代理上。

另一方面，LLM 的护栏提供了输入和输出的审查，以检测和减轻广泛的潜在危害[[13](https://arxiv.org/html/2406.09187v1#bib.bib13), [9](https://arxiv.org/html/2406.09187v1#bib.bib9), [16](https://arxiv.org/html/2406.09187v1#bib.bib16), [7](https://arxiv.org/html/2406.09187v1#bib.bib7), [26](https://arxiv.org/html/2406.09187v1#bib.bib26)]。这通常是通过在另一个预训练的 LLM 上建立护栏来实现的，从而能够在上下文中理解目标 LLM 的输入和输出。更重要的是，护栏的“非侵入性”特性，通过与目标 LLM 并行部署，允许它们仅通过少量修改就能够应用于新模型和有害性分类。然而，LLM 代理与 LLM 有显著区别，因为它们涉及更广泛的输出方式和高度特定的护栏请求。例如，一个由 LLM 驱动的网页代理可能会生成诸如点击网页上指定按钮的动作[[27](https://arxiv.org/html/2406.09187v1#bib.bib27)]。在这里，护栏请求可能涉及安全规则，禁止某些用户（例如，未满特定年龄的用户）购买特定物品（例如，酒精饮料）。显然，现有仅用于审查 LLM 文本输入和输出的护栏无法处理如此复杂的护栏请求。

在本文中，我们首次研究了针对LLM代理的防护措施。我们提出了GuardAgent，这是第一个可泛化的框架，利用LLM代理通过遵循用户的多种现实世界防护请求（如安全规则或隐私政策）来保障其他LLM代理（以下简称“目标代理”）。部署GuardAgent需要规定一组基于目标代理规范的文本防护请求（例如，代理输出和日志的格式）。在推理过程中，目标代理的用户输入以及相关的输出和日志将被提供给GuardAgent进行检查，以确定防护请求是否满足。具体来说，GuardAgent首先利用LLM根据防护请求和目标代理的输入输出生成行动计划。随后，LLM将行动计划转化为防护代码，并通过调用外部引擎来执行该代码。对于行动计划和防护代码的生成，LLM会通过内存模块获取相关示范，内存模块保存了之前使用案例的输入和输出。这种基于知识的推理是GuardAgent理解不同类型LLM代理的多种防护请求的基础。我们的GuardAgent设计有三个主要优点。首先，GuardAgent可以通过简单地向其工具箱上传新功能，轻松泛化以保障新的目标代理。其次，GuardAgent通过代码生成和执行提供防护措施，这比仅基于自然语言的防护措施更可靠。第三，GuardAgent通过上下文学习使用LLM，能够直接利用现成的LLM，无需额外训练。

在介绍《GuardAgent》框架之前（见第[4节](https://arxiv.org/html/2406.09187v1#S4 "4 GuardAgent Framework ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")），我们研究了针对不同类型LLM代理的多样化保护请求，并在第[3节](https://arxiv.org/html/2406.09187v1#S3 "3 Safety Requests for Diverse LLM Agents ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")提出了两个新的基准。第一个基准，EICU-AC，旨在评估医疗领域LLM代理的访问控制效果。第二个基准，Mind2Web-SC，用于评估LLM驱动的网页代理的安全控制。这两个基准被用于在第[5节](https://arxiv.org/html/2406.09187v1#S5 "5 Experiments ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")的实验中评估我们的GuardAgent。请注意，本文所考虑的两类保护请求——访问控制和安全控制——分别与隐私和安全密切相关，而这两者是AI可信度的关键视角[[19](https://arxiv.org/html/2406.09187v1#bib.bib19)]。我们的技术贡献总结如下：

+   •

    我们提出了GuardAgent，这是第一个通过知识启用推理为其他LLM代理提供保护的LLM代理框架，旨在处理各种用户的保护请求。

+   •

    我们提出了一种新的GuardAgent设计，包含通过上下文演示启用的知识任务规划，随后是涉及可扩展功能数组的保护代码生成。这样的设计赋予了GuardAgent强大的泛化能力、可靠的保护代码生成，并且无需额外训练。

+   •

    我们创建了两个基准，EICU-AC和Mind2Web-SC，分别用于评估医疗代理的隐私相关访问控制和网页代理的安全控制。

+   •

    我们展示了GuardAgent有效地为1）医疗领域的EHRAgent提供了98.7%的访问控制保护精度，以及2）为SeeAct网页代理提供了90.0%的安全控制保护精度。我们还展示了GuardAgent在保护代码生成和执行过程中定义新功能的能力。

## 2 相关工作

LLM代理指的是将LLM作为其任务理解和规划的核心引擎，并通过与环境交互（例如，通过调用第三方API）执行计划的AI代理[[21](https://arxiv.org/html/2406.09187v1#bib.bib21)]。与LLM（仅输出文本）存在的根本区别使得LLM代理可以部署在多个应用领域，包括金融[[24](https://arxiv.org/html/2406.09187v1#bib.bib24)]、医疗保健[[2](https://arxiv.org/html/2406.09187v1#bib.bib2)、[17](https://arxiv.org/html/2406.09187v1#bib.bib17)、[22](https://arxiv.org/html/2406.09187v1#bib.bib22)、[18](https://arxiv.org/html/2406.09187v1#bib.bib18)、[11](https://arxiv.org/html/2406.09187v1#bib.bib11)]、日常工作[[4](https://arxiv.org/html/2406.09187v1#bib.bib4)、[5](https://arxiv.org/html/2406.09187v1#bib.bib5)、[28](https://arxiv.org/html/2406.09187v1#bib.bib28)、[27](https://arxiv.org/html/2406.09187v1#bib.bib27)]和自动驾驶[[3](https://arxiv.org/html/2406.09187v1#bib.bib3)、[8](https://arxiv.org/html/2406.09187v1#bib.bib8)、[12](https://arxiv.org/html/2406.09187v1#bib.bib12)]。LLM代理通常还配备了可检索的记忆模块，使它们能够执行知识驱动的推理，从而处理其应用领域中的不同任务[[10](https://arxiv.org/html/2406.09187v1#bib.bib10)]。我们的GuardAgent是一个典型的LLM代理，但与现有代理的目标不同，因为它是第一个用来保护其他LLM代理的系统。

基于LLM的防护措施属于一种有害性减轻的审查方法[[25](https://arxiv.org/html/2406.09187v1#bib.bib25)、[15](https://arxiv.org/html/2406.09187v1#bib.bib15)]。传统的防护措施作为分类器，训练于分类标记的内容[[13](https://arxiv.org/html/2406.09187v1#bib.bib13)、[9](https://arxiv.org/html/2406.09187v1#bib.bib9)]，而最近，基于LLM且具有更广泛上下文理解的防护措施已经被开发，并展现出强大的泛化能力。然而，现有的LLM防护措施，无论是‘模型防护模型’([[16](https://arxiv.org/html/2406.09187v1#bib.bib16)、[7](https://arxiv.org/html/2406.09187v1#bib.bib7)、[26](https://arxiv.org/html/2406.09187v1#bib.bib26)])还是‘代理防护模型’([[1](https://arxiv.org/html/2406.09187v1#bib.bib1)]），都是为了应对自然语言中的有害性定义的。它们不能直接用于保护具有多样化输出方式的LLM代理。在本文中，我们提出了GuardAgent，首个‘代理防护代理’框架，并通过实验展示了其相较于‘模型防护代理’方法的优势。

## 3 针对多样化LLM代理的安全请求

在介绍我们的 GuardAgent 之前，我们将在本节中调查不同类型的 LLM 代理的安全请求。我们重点关注两个具有代表性的 LLM 代理：一个用于医疗保健的 EHRAgent 和一个名为 SeeAct 的网页代理。特别地，EHRAgent 代表了用于高风险任务的 LLM 代理，而 SeeAct 代表了用于多任务的通用 LLM 代理。我们简要回顾这两个代理，它们的指定任务以及原始的评估基准。更重要的是，我们提出了两个新颖的基准，用于不同的安全请求：1）EICU-AC，用于评估像 EHRAgent 这样的医疗保健代理的访问控制；2）Mind2Web-SC，用于评估像 SeeAct 这样的网页代理的安全控制。然后，我们进行了一项初步研究，测试了基于注入到 EHRAgent 和 SeeAct 系统提示中的简单指令的“侵入性”方法用于访问控制和安全控制；它们的低效性和差的泛化能力促使我们提出了 GuardAgent。

![参考说明](img/8c0c8ac5489b1f1a1460c78ba2c1f6da.png)

图 2：EICU-AC 的一个示例（左）和 Mind2Web-SC 的一个示例（右）。

### 3.1 EHRAgent 和 EICU-AC 基准

#### EHRAgent

EHRAgent 设计用于通过生成代码从提供的数据库中检索和分析数据，以响应与医疗保健相关的查询 [[17](https://arxiv.org/html/2406.09187v1#bib.bib17)]。EHRAgent 已在多个基准上进行评估，并表现出良好的性能，包括一个包含 ICU 患者临床护理问题的 EICU 数据集（例如，见图 [2](https://arxiv.org/html/2406.09187v1#S3.F2 "Figure 2 ‣ 3 Safety Requests for Diverse LLM Agents ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")）以及 10 个相关的数据库 [[14](https://arxiv.org/html/2406.09187v1#bib.bib14)]。每个数据库包含多种类型的患者信息，这些信息存储在不同的列中。在实际的医疗保健系统中，基于用户身份限制对特定数据库的访问至关重要。例如，行政人员不应访问患者诊断详情。因此，像 EHRAgent 这样的医疗保健 LLM 代理应该能够在用户属于一般行政部门时，拒绝对患者诊断数据库的信息请求。本质上，这些 LLM 代理应该具备访问控制功能，以保护患者隐私。

#### 提议的 EICU-AC 基准

本文中，我们从EICU创建了一个EICU-AC基准，用于评估EHRAgent的访问控制方法（以及可能需要数据库检索的其他医疗代理）。我们为EHRAgent（以及其他类似的目标代理）的用户定义了三个角色：“医生”，“护士”和“一般管理”。所评估的访问控制应确保每个身份仅能访问EICU基准的部分数据库和列。我们通过查询ChatGPT生成每个角色的真实访问权限（更多细节请参见附录[A.1](https://arxiv.org/html/2406.09187v1#A1.SS1 "A.1 Role-Based Access Permission ‣ Appendix A Details About the EICU-AC Benchmark ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")）。虽然通用的访问控制方法应对特定角色及其访问权限保持不变，但我们做出这些选择是为了模拟实际的医疗场景。然后，EICU-AC中的每个示例设计包括以下信息：1）一个与医疗相关的问题及其正确答案，2）回答该问题所需的数据库和列，3）一个用户身份/角色，4）一个二进制标签“0”，如果给定身份可以访问所有所需的数据库和列，或者“1”表示不能访问，5）如果标签为“1”，则给定身份无法访问的数据库和列。生成的EICU-AC示例的插图见图[2](https://arxiv.org/html/2406.09187v1#S3.F2 "Figure 2 ‣ 3 Safety Requests for Diverse LLM Agents ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")。

具体而言，EICU-AC 中的所有问题都来自 EICU 数据集或经过调整。我们保留那些由 EHRAgent 使用 GPT-4（在零温度下）作为核心 LLM 正确回答的 EICU 问题，以便通过我们的基准进行评估时，主要聚焦于访问控制，而不受任务表现的太大影响。最初，我们通过分别为每个问题分配三个角色，生成三个 EICU-AC 示例。然后，我们根据每个示例所分配角色的实际访问权限进行标注，发现所有三个身份的标签非常不平衡。因此，对于每个身份，我们删除一些生成的示例，同时添加新的示例，以实现两个标签之间的相对平衡（更多细节见附录[A.2](https://arxiv.org/html/2406.09187v1#A1.SS2 "A.2 Sampling from EICU ‣ Appendix A Details About the EICU-AC Benchmark ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")）。最终，我们的 EICU-AC 包含分别标注为‘0’的52个、57个和45个示例，分别对应‘医生’、‘护理’和‘普通行政’，以及分别标注为‘1’的46个、55个和61个示例。此外，在这些316个示例中，共有226个独特的问题，涵盖了51个不同的 ICU 信息类别，这凸显了我们 EICU-AC 的多样性。  

### 3.2 SeeAct 和 Mind2Web-SC 基准

#### SeeAct  

SeeAct 是一个通用的网络代理，通过自然语言指令在任何给定的网站上执行任务，任务通过连续生成的操作完成，包括点击按钮、输入特定文本等（例如，见图[2](https://arxiv.org/html/2406.09187v1#S3.F2 "Figure 2 ‣ 3 Safety Requests for Diverse LLM Agents ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")）[[27](https://arxiv.org/html/2406.09187v1#bib.bib27)]。在原始论文中，SeeAct 在 Mind2Web 基准测试上进行了评估，该基准包含超过2,000个复杂的网络任务，跨越31个领域的137个网站（例如，汽车租赁、购物、娱乐等）[[4](https://arxiv.org/html/2406.09187v1#bib.bib4)]。然而，对于像 SeeAct 这样的实际网络代理来说，集成安全控制至关重要，这些控制限制特定用户的某些操作。例如，在世界大多数地区，租车需要持有驾驶执照。  

#### 提出的 Mind2Web-SC 基准  

我们创建了一个Mind2Web-SC基准，用于评估适用于SeeAct和其他基于动作生成的网页代理的安全控制。安全控制的目标是确保代理遵守我们基于常见网络法规和地区惯例制定的六条规则：1）用户必须是会员才能购物，2）未接种疫苗的用户不能预订航班，3）没有驾驶执照的用户不能购买或租赁汽车，4）未满18岁的用户不能预订酒店，5）用户必须在特定国家才能搜索电影/音乐/视频，6）15岁以下的用户不能申请工作。再次声明，这些规则仅为评估目的而提出，不代表作者的个人观点或态度。

Mind2Web-SC中的示例是通过以下步骤创建的。首先，我们从Mind2Web测试集的旅行、购物和娱乐领域获取SeeAct（使用LLaVA-1.5作为核心LLM）正确动作预测的所有任务。其次，对于每个任务，我们随机创建一个用户档案，包含整数型的‘年龄’以及布尔型的‘国内’、‘驾驶执照’、‘疫苗’和‘会员资格’（参见图[2](https://arxiv.org/html/2406.09187v1#S3.F2 "Figure 2 ‣ 3 Safety Requests for Diverse LLM Agents ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")右侧）。请注意，这六个用户信息类别中的每一项都不是琐碎的，因为它至少与我们制定的六条安全规则中的一条相关。第三，我们根据任务和用户信息手动标注每个示例。如果任务本身与任何规则无关，则该示例将标注为‘0’，表示‘允许操作’。如果任务与至少一条规则相关（例如与租车相关的规则），我们检查用户信息，如果规则被违反（例如‘dr_license’为‘false’），则将示例标注为‘1’，表示‘拒绝操作’，否则标注为‘0’。对于每个标注为‘1’的示例，违反的规则也将包含在我们的基准中。最后，我们通过创建额外的示例（基于现有任务，但用户信息不同）来平衡这两个类别，同时删除一些任务与任何规则无关的示例（详细信息请见附录[B](https://arxiv.org/html/2406.09187v1#A2 "Appendix B Details About the Mind2Web-SC Benchmark ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")）。创建的Mind2Web-SC基准每个类别包含100个示例，且每个类别中的任务都是唯一的。

表1：基于系统指令对EHRAgent的访问控制和对SeeAct的安全控制在EICU-AC和Mind2Web-SC上无效；因此，需要一种新的防护方法。

|  | LPP $\uparrow$ | LPR $\uparrow$ | CCA $\uparrow$ | FRA $\uparrow$ |
| --- | --- | --- | --- | --- |
| EHRAgent + EICU-AC | 76.6 | 90.7 | 50.0 | 3.2 |
| SeeAct + Mind2Web-SC | 95.1 | 58.0 | 58.0 | 71.0 |

### 3.3 两个基准的初步防护措施

#### 天真“入侵性”基线

我们使用 EICU-AC 基准来评估一种朴素的方法，该方法通过指定三种角色及其可访问的数据库和列，将访问控制硬编码到 EHRAgent 的系统提示中。在评估过程中，修改后的 EHRAgent 将同时获得 EICU-AC 示例的角色和查询。它的系统提示将包括指示，如果给定角色有任何不可访问的数据库和列，则显示“拒绝消息”，并列出这些不可访问的数据库和列。同样，我们将安全检查的文本指令融入 SeeAct 的系统提示中，并在 Mind2Web-SC 上对其进行评估。如果给定用户配置文件违反了任何规则，强制安全的 SeeAct 应该打印出包含违规规则的“拒绝消息”。关于这两种配备朴素“入侵式”保护措施的代理系统提示的详细信息，请参见 App. [C](https://arxiv.org/html/2406.09187v1#A3 "附录 C：基于指令的朴素访问控制和安全控制的详细系统提示 ‣ GuardAgent：通过知识驱动推理保护 LLM 代理的保护代理")。

#### 指标

我们考虑了两个基准共享的四个评估指标：标签预测精度（LPP）、标签预测召回率（LPR）、综合控制准确度（CCA）和最终响应准确度（FRA），所有指标均以百分比表示。LPP 和 LPR 都是在每个数据集的所有示例上计算的，用于衡量整体标签预测效果，其中只有在“拒绝消息”出现时，标签‘1’的预测才会被计入。CCA 考虑所有标记为‘1’的真实例子。它被定义为这些例子中正确预测为‘1’并成功检测到所有不可访问的数据库和列（对于 EICU-AC）或所有违规规则（对于 Mind2Web-SC）的百分比。相比之下，FRA 考虑所有标记为‘0’的真实例子。它被定义为这些例子中正确预测为‘0’并且代理的响应正确的百分比。

#### 结果

如表[1](https://arxiv.org/html/2406.09187v1#S3.T1 "Table 1 ‣ Proposed Mind2Web-SC benchmark ‣ 3.2 SeeAct and Mind2Web-SC Benchmark ‣ 3 Safety Requests for Diverse LLM Agents ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")所示，两个简单的基线方法未能完成其指定任务，表现出标签预测的低精度或低召回率。具体而言，EHRAgent的简单访问控制过于严格，导致大量假阳性；相反，SeeAct的简单安全控制未能拒绝许多不安全的操作，导致大量假阴性。此外，引入额外任务的“入侵”对两个代理都造成了沉重的负担，显著降低了它们在指定任务上的表现，特别是对于EHRAgent（其在负面示例上通过FRA测量的端到端准确率仅为3.2%）。最后，尽管表现不佳，但这两种简单的守护方法是与代理硬编码的，无法转移到具有不同设计的其他LLM代理上。这些缺点突显了我们GuardAgent的必要性，它在保障各种LLM代理的安全方面既有效又具有可扩展性。

## 4 GuardAgent框架

本节介绍了GuardAgent的三个关键特点：1）可扩展性——GuardAgent的记忆和工具箱可以轻松扩展，以处理具有新守护请求的新目标代理；2）可靠性——GuardAgent的输出通过成功的代码执行获得；3）免训练——GuardAgent基于上下文学习，无需任何大语言模型（LLM）训练。

### 4.1 GuardAgent概述

GuardAgent的预期用户是目标LLM代理的开发者或管理员，旨在为其实施守护措施。GuardAgent的强制输入均为文本形式，包括一组守护请求$I_{r}$、目标代理的规格$I_{s}$、目标代理的输入$I_{i}$和目标代理对应于$I_{i}$的输出日志$I_{o}$。其中，$I_{r}$由$I_{s}$提供，$I_{s}$包含目标代理的功能、输入和输出日志的内容、它们的格式等。GuardAgent的目标是检查$I_{i}$和$I_{o}$是否满足守护请求$I_{r}$，然后生成标签预测$O_{l}$，其中$O_{l}=0$表示守护请求已满足，$O_{l}=1$则表示未满足。如果$O_{l}=0$，目标代理提出的输出或操作将被GuardAgent接受；如果$O_{l}=1$，则被拒绝。如果$O_{l}=1$，GuardAgent还应输出详细的原因$O_{d}$（例如，通过打印出EICU-AC不可访问的数据库和列）以供进一步处理。例如，某些用例的严重规则违规可能需要司法干预。

GuardAgent的核心理念是利用大型语言模型（LLM）的逻辑推理能力和知识检索，准确地将文本形式的守护请求“翻译”成可执行的代码。因此，GuardAgent的流程包含两个主要步骤（见图 [1](https://arxiv.org/html/2406.09187v1#S0.F1 "图 1 ‣ GuardAgent：通过知识驱动推理保护LLM代理")）。在第一步（第 [4.2](https://arxiv.org/html/2406.09187v1#S4.SS2 "4.2 任务规划 ‣ 4 GuardAgent框架 ‣ GuardAgent：通过知识驱动推理保护LLM代理") 节）中，系统通过向LLM提供上述输入来生成逐步的行动计划。在第二步 [4.3](https://arxiv.org/html/2406.09187v1#S4.SS3 "4.3 防护代码生成与执行 ‣ 4 GuardAgent框架 ‣ GuardAgent：通过知识驱动推理保护LLM代理") 中，我们用行动计划和一组可调用的函数来提示LLM，从而获取防护代码，随后通过调用外部引擎执行该代码。在这两个步骤中，都提供了一个内存模块用于检索上下文中的示范。

### 4.2 任务规划

任务规划的目标是根据输入生成一个逐步的行动计划$P$。一种简单的设计是，通过向基础LLM提供$[I_{p},I_{s},I_{r},I_{i},I_{o}]$来提示，其中$I_{p}$包含精心设计的规划指令，这些指令 1) 定义了GuardAgent的每个输入，2) 说明了防护任务（即检查$I_{r}$是否由$I_{i}$和$I_{o}$满足），并且 3) 指导行动步骤的生成（见附录[D](https://arxiv.org/html/2406.09187v1#A4 "附录D GuardAgent的完整输入与输出 ‣ GuardAgent：通过知识驱动推理保护LLM代理")中的图 [8](https://arxiv.org/html/2406.09187v1#A4.F8 "图 8 ‣ GuardAgent的完整输入 ‣ 附录D GuardAgent的完整输入与输出") ）。然而，理解复杂的防护请求并将其与目标代理结合起来，仍然是现有LLM面临的一项挑战。

我们通过允许GuardAgent从记忆模块中检索示范来解决这一挑战，该模块存档了过去使用案例中目标代理的输入和输出。这里，记忆模块中的一个元素$D$表示为$D=[I_{i,D},I_{o,D},P_{D},C_{D}]$，其中$I_{i,D}$和$I_{o,D}$分别表示目标代理的输入和输出，$P_{D}$包含行动步骤，$C_{D}$包含防护代码。检索是基于当前目标代理输入和输出与记忆中记录的输入和输出之间的相似性。具体来说，我们通过选择与当前目标代理输入和输出的Levenshtein距离$L([I_{i,D},I_{o,D}],[I_{i},I_{o}])$最小的$k$个元素来检索$k$个示范。然后，行动计划通过$P={\rm LLM}([I_{p},I_{s},I_{r},[I_{i,D_{1}},I_{o,D_{1}},P_{D_{1}}],\cdots,[I_{i,D_{k}},I_{o,D_{k}},P_{D_{k}}],I_{i},I_{o}])$获得。请注意，为了简洁起见，已移除每个示范中的防护代码。

在将GuardAgent应用于新的LLM代理以处理特定的防护请求的情况下，我们还允许GuardAgent的用户手动将示范注入到记忆模块中。特别地，我们要求用户提供的每个示范中的行动计划包含四个强制步骤，表示为$P_{D}=[p_{1,D},p_{2,D},p_{3,D},p_{4,D}]$，这四个步骤形成一个思维链[[20](https://arxiv.org/html/2406.09187v1#bib.bib20)]。一般而言，$p_{1,D}$总结防护请求，以识别关键词，例如‘访问控制’，其中包括三个角色：‘医生’、‘护理’和‘普通管理’，用于EICU-AC。然后，$p_{2,D}$过滤与目标代理输入相关的防护请求中的信息，而$p_{3,D}$总结目标代理的输出日志，并定位防护请求中相关的内容。最后，$p_{4,D}$指导防护代码生成，比较在$p_{2,D}$和$p_{3,D}$中获得的信息，以及假设的执行引擎。示范行动计划的示例见附录[13](https://arxiv.org/html/2406.09187v1#A7.F13 "图13 ‣ 附录G 手动创建的示范 ‣ GuardAgent: 通过知识驱动推理保护LLM代理")。

### 4.3 防护代码生成与执行

本步骤的目标是基于行动计划 $P$ 生成一个护栏代码 $C$。一旦生成，$C$ 将通过行动计划中指定的外部引擎 $E$ 执行。然而，直接通过向大型语言模型（LLM）提供行动计划 $P$ 和简单指令生成的护栏代码可能无法可靠执行。为了解决这个问题，我们的一个关键设计是采用更全面的指令，其中包括一个可调用函数列表 $\mathcal{F}$，并指定它们的输入参数。这些函数的定义存储在 GuardAgent 的工具箱中，用户可以通过上传代码轻松扩展以应对新的护栏请求和目标代理。LLM 被指示仅使用提供的函数来生成代码；否则，它在生成代码时很容易编造不存在的函数。

此外，我们还利用从记忆中检索的过去示例，采用与任务规划中相同的方法，将其作为代码生成的示范。因此，我们有 $C={\rm LLM}(I_{c}(\mathcal{F}),D_{1},\cdots,D_{k},I_{i},I_{o},P)$，其中 $I_{c}(\mathcal{F})$ 是基于可调用函数 $\mathcal{F}$ 的指令，$D_{1},\cdots,D_{k}$ 是检索到的示范。GuardAgent 的输出通过执行生成的代码获得，即 $(O_{l},O_{d})=E(C,\mathcal{F})$。最后，我们采用 Shi 等人提出的调试机制 [[17](https://arxiv.org/html/2406.09187v1#bib.bib17)]，该机制调用 LLM 分析执行过程中可能出现的任何错误信息，从而提高生成代码的可靠性。

## 5 个实验

在第[5.2节](https://arxiv.org/html/2406.09187v1#S5.SS2 "5.2 Guardrail Performance ‣ 5 Experiments ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中，我们展示了GuardAgent在保护EHRAgent（在EICU-AC上的表现）和SeeAct（在Mind2Web-SC上的表现）方面的有效性，分别达到了98.7%和90.0%的标签预测准确率。通过一个案例研究，我们说明了GuardAgent相对于“模型守护代理”方法的优势，归因于通过代码生成和执行实现的更可靠的防护。 在第[5.3节](https://arxiv.org/html/2406.09187v1#S5.SS3 "5.3 Ablation Studies ‣ 5 Experiments ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中，我们进行消融研究，显示出1）GuardAgent对EICU-AC中的大多数角色和Mind2Web-SC中的规则表现相似良好，使其能够处理高复杂度的守护请求，2）GuardAgent仅需要少量示范来执行任务。 在第[5.4节](https://arxiv.org/html/2406.09187v1#S5.SS4 "5.4 Code-Based Guardrail is the Natural Preference of LLMs, but Tools are Needed ‣ 5 Experiments ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中，我们展示了GuardAgent可以根据守护请求定义必要的功能，突出了其在处理新守护请求时的泛化能力。此外，我们还发现，LLM（如GPT-4）倾向于生成基于代码的防护（尽管大多数是无法执行的），即使没有为代码生成和执行提供具体的指示。

### 5.1 设置

#### 数据集和代理

我们分别在EICU-AC和Mind2Web-SC上测试了GuardAgent，目标代理分别是EHRAgent和SeeAct（使用它们的原始设置）。每个EICU-AC示例中的角色和问题是EHRAgent的输入，输出日志包括推理步骤、生成的代码和EHRAgent生成的最终答案。SeeAct的输入包含来自Mind2Web-SC中每个示例的任务和用户信息，输出日志包括SeeAct预测的动作和推理过程。两个目标代理的示例输入（$I_{i}$）和输出日志（$I_{o}$）（即GuardAgent的输入）显示在附录[D](https://arxiv.org/html/2406.09187v1#A4 "Appendix D Complete Inputs and Output of GuardAgent ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中。由于篇幅限制，GuardAgent的其他输入，包括两个目标代理的规格（$I_{s}$）、与两个基准相关的守护请求（$I_{r}$）以及规划指令（$I_{p}$），也显示在附录[D](https://arxiv.org/html/2406.09187v1#A4 "Appendix D Complete Inputs and Output of GuardAgent ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中。

#### GuardAgent的设置

在主要实验中，我们将演示数量设置为$k=1$和$k=3$，分别用于EICU-AC和Mind2Web-SC。其他选择将在我们的消融研究中考虑，见第[5.3节](https://arxiv.org/html/2406.09187v1#S5.SS3 "5.3 Ablation Studies ‣ 5 Experiments ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")。对于每个基准，我们通过$k$个手动创建的演示初始化GuardAgent的记忆（例如见附录[G](https://arxiv.org/html/2406.09187v1#A7 "Appendix G Manually Created Demonstrations ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")）。我们使用GPT-4版本2024-02-01，并设置温度为零，作为GuardAgent的核心LLM。我们使用Python作为默认的代码执行引擎，工具箱中包含两个初始函数，‘CheckAccess’和‘CheckRules’，这些函数在附录[E](https://arxiv.org/html/2406.09187v1#A5 "Appendix E Callable Functions ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中定义。请注意，GuardAgent的用户可以轻松地将新函数或引擎上传到工具箱中。最后，我们允许进行三次调试迭代，尽管在大多数情况下，GuardAgent生成的保护代码是可以直接执行的。

#### 基准

由于GuardAgent是第一个旨在保护其他代理的LLM代理，因此我们将其与使用模型保护代理的基准进行比较。在这里，我们考虑使用GPT-4版本2024-02-01和Llama3-70B作为保护模型¹¹1“模型保护模型”的方法，例如LlamaGuard，它设计用于检测预定义的不安全类别[[7](https://arxiv.org/html/2406.09187v1#bib.bib7)]，由于目标完全不同，因此不在此讨论。我们创建了包含高级指令的综合性提示，$I^{\prime}_{p}$，它是从GuardAgent的提示中改编而来的，演示数量与GuardAgent相同，但不包括保护代码生成，记作$D^{\prime}_{1},\cdots,D^{\prime}_{k}$，并且使用与GuardAgent相同的输入集。然而，这些基准模型没有像GuardAgent那样涉及记忆模块；它们在评估过程中使用固定的演示集。两个基准的示例提示见附录[H](https://arxiv.org/html/2406.09187v1#A8 "Appendix H Function Defined by GuardAgent in Zero-Shot Setting ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")。然后，基准的保护模型输出通过$(O_{l},O_{d})={\rm LLM}(I^{\prime}_{p},I_{s},I_{r},D^{\prime}_{1},\cdots,D^{\prime}_{k},I_{i},I_{o})$获得。

#### 评估指标

我们使用了两个标签预测指标，LPP和LPR，以及CCA指标，所有这些都在第[3.3节](https://arxiv.org/html/2406.09187v1#S3.SS3 "3.3 Preliminary Guardrails on the Two Benchmarks ‣ 3 Safety Requests for Diverse LLM Agents ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中定义。由于所有被评估的防护措施在警报未触发时不会影响目标代理的正常操作，因此此处不考虑FRA指标。此外，我们报告了标签预测准确率（LPA，也叫做防护准确率），该指标在每个数据集的所有示例上定义，作为防护措施性能的整体指标。

表2：GuardAgent在保障EICU-AC上的EHRAgent和Mind2Web-SC上的SeeAct性能，与基于模型的两个基线（分别为GPT-4和Llama3）进行比较。

|  | EHRAgent on EICU-AC | SeeAct on Mind2Web-SC |
| --- | --- | --- |
|  | LPA $\uparrow$ | LPP $\uparrow$ | LPR $\uparrow$ | CCA $\uparrow$ | LPA $\uparrow$ | LPP $\uparrow$ | LPR $\uparrow$ | CCA $\uparrow$ |
| Llama3 | 92.1 | 95.4 | 88.9 | 41.4 | 76.5 | 93.4 | 57.0 | 57.0 |
| GPT-4 | 97.5 | 95.3 | 100.0 | 67.9 | 82.5 | 100.0 | 65.0 | 65.0 |
| GuardAgent | 98.7 | 100.0 | 97.5 | 97.5 | 90.0 | 100.0 | 80.0 | 80.0 |

![请参阅标题](img/956ee81b74a876080c8d7e5586c00868.png)

图3：左：GPT-4基线的失败案例，其中不同数据库共享相同的列名（‘patientunitstayid’），无法有效区分。右：GuardAgent的失败案例，由于查询中细节过多，未能检测到规则违规。

### 5.2 防护性能

在表[2](https://arxiv.org/html/2406.09187v1#S5.T2 "Table 2 ‣ Evaluation metrics ‣ 5.1 Setup ‣ 5 Experiments ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中，我们展示了GuardAgent与基准方法的性能比较，使用了我们全面的评估指标。GuardAgent在LPA上表现优于两个基准方法，并且在CCA上也有明显差距，展示了“代理保护代理”相对于“模型保护代理”的优势。我们将这一优势归因于我们基于推理的代码生成和执行设计，而这种方法是护栏模型无法实现的。在GPT-4在EICU-AC上的许多失败案例中，我们发现基于自然语言的护栏无法有效区分列名，尤其是在不同数据库中共享同一列名的情况下。例如，在图[3](https://arxiv.org/html/2406.09187v1#S5.F3 "Figure 3 ‣ Evaluation metrics ‣ 5.1 Setup ‣ 5 Experiments ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中，包含名为‘patientunitstayid’的列的整个数据库‘vitalperiodic’对‘general administration’不可访问，而在数据库‘patient’中的同名列对相同角色是可访问的。在这种情况下，基于GPT-4的模型护栏未能识别数据库‘vitalperiodic’中的列‘patientunitstayid’为“不可访问”。相比之下，我们基于代码生成的GuardAgent准确地将每个数据库及其列转换为字典，有效避免了这种列名模糊性。

在图[3](https://arxiv.org/html/2406.09187v1#S5.F3 "Figure 3 ‣ Evaluation metrics ‣ 5.1 Setup ‣ 5 Experiments ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")的右侧，我们展示了GuardAgent的一个典型失败案例，其中违规规则未被检测到。我们发现，查询未能在任务规划中的链式推理的第一步与指定规则连接，可能是由于查询中的细节过于繁杂。然而，通过提供更多具有更好语言多样性的示范，或者使用更强大的LLM作为核心推理步骤，可以缓解这个问题。

表3：GuardAgent在EICU-AC三个角色和Mind2Web-SC六个规则上的结果分解。GuardAgent在除与电影、音乐和视频相关的规则5外，对所有角色和规则的表现都很均匀。

|  | EHRAgent 在 EICU-AC 上 | SeeAct 在 Mind2Web-SC 上 |
| --- | --- | --- |
|  | 医生 | 护理 | GA | 规则 1 | 规则 2 | 规则 3 | 规则 4 | 规则 5 | 规则 6 |
| LPA $\uparrow$ | 97.9 | 98.2 | 100.0 | 89.5 | 91.7 | 87.5 | 83.3 | 52.4 | 83.3 |
| CCA $\uparrow$ | 95.7 | 96.4 | 100.0 | 89.5 | 91.7 | 87.5 | 83.3 | 52.4 | 83.3 |

### 5.3 消融实验

#### 结果分解

在表[3](https://arxiv.org/html/2406.09187v1#S5.T3 "Table 3 ‣ 5.2 Guardrail Performance ‣ 5 Experiments ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中，我们展示了GuardAgent对a) EHRAgent在EICU-AC每个角色的LPA和CCA，以及b) SeeAct在EICU-AC每个规则的LPA和CCA（仅考虑正例）。总体而言，GuardAgent在EICU-AC的三个角色和Mind2Web-SC的六个规则上表现均匀良好，除了与电影、音乐和视频相关的规则5。我们发现所有此规则的失败案例都类似于图[3](https://arxiv.org/html/2406.09187v1#S5.F3 "Figure 3 ‣ Evaluation metrics ‣ 5.1 Setup ‣ 5 Experiments ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中所示的情形，其中查询在推理过程中无法与规则相关联。尽管如此，GuardAgent仍然展示了处理多样化复杂防护请求的相对强大能力。

#### 演示次数的影响

我们改变了GuardAgent使用的演示次数，并在图[4](https://arxiv.org/html/2406.09187v1#S5.F4 "Figure 4 ‣ Influence of number of demonstrations ‣ 5.3 Ablation Studies ‣ 5 Experiments ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中展示了相应的LPA和CCA。结果表明，GuardAgent在极少数的演示样本下也能取得不错的防护栏表现。

![参考说明](img/27e56ac9e39c9cf162beff136bcf30d5.png)![参考说明](img/3ce347b9c00ecae5cd6d189d83cb03ae.png)

图4：在EICU-AC和Mind2Web-SC上，GuardAgent在不同数量的演示下的表现。GuardAgent在演示次数非常少的情况下仍然有效。

### 5.4 基于代码的防护栏是LLMs的自然偏好，但仍需要工具

我们考虑一个具有挑战性的任务，要求GuardAgent生成防护栏代码，但未提供a)生成防护请求所需的函数，也未提供b)防护栏代码生成的演示。具体来说，防护栏代码现在由$C^{\prime}={\rm LLM}(I_{c}(\mathcal{F}^{\prime}),I_{i},I_{o},P)$生成，其中$\mathcal{F}^{\prime}$表示不包含所需函数的工具箱。在这种情况下，GuardAgent要么定义所需的函数，要么生成面向同一目标的程序化代码（见附录[H](https://arxiv.org/html/2406.09187v1#A8 "Appendix H Function Defined by GuardAgent in Zero-Shot Setting ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中的GuardAgent生成的防护栏函数示例），并在EICU-AC上达到了90.8%的LPA和96.1%的CCA。这些结果支持我们在代码生成步骤中采用可调用函数列表和演示作为关键设计，并展示了GuardAgent在零样本情况下处理新防护请求的良好泛化能力。

此外，我们考虑了一个更加具有挑战性的安全防护任务。我们使用GPT-4模型来保护EICU-AC中的EHRAgent，但去除了所有与代码生成相关的指令。换句话说，LLM必须自己摸索，无论是否生成代码，都能提供安全防护。有趣的是，我们发现，在EICU-AC中的68.0%的示例中，LLM选择了生成基于代码的安全防护（尽管大多数情况下不可执行）。这一结果展示了LLM倾向于使用代码作为一种结构化且精确的安全防护方法，支持我们基于代码生成设计GuardAgent。

## 6 结论

本文提出了首个关于LLM代理安全防护的研究，旨在应对各种用户安全需求。我们提出了GuardAgent，这是第一个旨在保护其他LLM代理的LLM代理框架。GuardAgent利用LLM的知识启用推理能力生成任务计划，并将其转化为安全防护代码。其特点是能够概括新型安全防护请求、代码化安全防护的可靠性和低计算开销。此外，我们提出了两个基准测试，分别用于评估LLM代理在医疗和网页领域的隐私相关访问控制和安全控制。我们展示了GuardAgent在这两个基准测试中超越了“模型保护代理”基线，并且在零样本设置下，GuardAgent的代码泛化能力表现出色。

## 参考文献

+   [1] Guardrails AI. [https://www.guardrailsai.com/](https://www.guardrailsai.com/)，2023年。

+   [2] Mahyar Abbasian, Iman Azimi, Amir M. Rahmani, 和 Ramesh Jain. 会话健康代理：一个个性化的LLM驱动代理框架，2024年。

+   [3] Can Cui, Zichong Yang, Yupeng Zhou, Yunsheng Ma, Juanwu Lu, Lingxi Li, Yaobin Chen, Jitesh Panchal, 和 Ziran Wang. 基于大语言模型的个性化自动驾驶：实地实验，2024年。

+   [4] Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Samuel Stevens, Boshi Wang, Huan Sun, 和 Yu Su. Mind2web: 迈向一个通用型网页代理，2023年。

+   [5] Izzeddin Gur, Hiroki Furuta, Austin V Huang, Mustafa Safdari, Yutaka Matsuo, Douglas Eck, 和 Aleksandra Faust. 一种具有规划、长上下文理解和程序合成能力的真实世界网页代理。发表于《第十二届国际学习表示大会》，2024年。

+   [6] Wencheng Han, Dongqian Guo, Cheng-Zhong Xu, 和 Jianbing Shen. Dme-driver: 将人类决策逻辑与3D场景感知结合的自动驾驶，2024年。

+   [7] Hakan Inan, Kartikeya Upasani, Jianfeng Chi, Rashi Rungta, Krithika Iyer, Yuning Mao, Michael Tontchev, Qing Hu, Brian Fuller, Davide Testuggine, 和 Madian Khabsa. Llama guard: 基于LLM的人类-人工智能对话输入输出安全防护，2023年。

+   [8] Ye Jin, Xiaoxi Shen, Huiling Peng, Xiaoan Liu, Jingli Qin, Jiayang Li, Jintao Xie, Peizhong Gao, Guyue Zhou, 和 Jiangtao Gong. Surrealdriver：基于大语言模型设计的城市环境下生成驾驶员代理模拟框架，2023年。

+   [9] Alyssa Lees, Vinh Q. Tran, Yi Tay, Jeffrey Sorensen, Jai Gupta, Donald Metzler, 和 Lucy Vasserman. 新一代视角API：高效的多语言字符级变换器. 见于第28届ACM SIGKDD知识发现与数据挖掘大会，2022年。

+   [10] Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, Sebastian Riedel, 和 Douwe Kiela. 用于知识密集型NLP任务的检索增强生成. 见于第34届神经信息处理系统国际会议，2020年。

+   [11] Junkai Li, Siyu Wang, Meng Zhang, Weitao Li, Yunghwei Lai, Xinhui Kang, Weizhi Ma, 和 Yang Liu. 代理医院：一个具有可进化医学代理的医院模拟体，2024年。

+   [12] Jiageng Mao, Junjie Ye, Yuxi Qian, Marco Pavone, 和 Yue Wang. 一种用于自动驾驶的语言代理. 2023年。

+   [13] Todor Markov, Chong Zhang, Sandhini Agarwal, Tyna Eloundou, Teddy Lee, Steven Adler, Angela Jiang, 和 Lilian Weng. 一种面向现实世界中不良内容检测的整体方法. 见于AAAI，2023年。

+   [14] Tom J Pollard, Alistair E W Johnson, Jesse D Raffa, Leo A Celi, Roger G Mark, 和 Omar Badawi. eicu协作研究数据库，一个自由提供的多中心危重病研究数据库. 《科学数据》，2018年。

+   [15] Xiangyu Qi, Yi Zeng, Tinghao Xie, Pin-Yu Chen, Ruoxi Jia, Prateek Mittal, 和 Peter Henderson. 微调对齐语言模型危及安全性，即便用户并非有意为之！ 见于第十二届国际学习表征会议，2024年。

+   [16] Traian Rebedea, Razvan Dinu, Makesh Narsimhan Sreedhar, Christopher Parisien, 和 Jonathan Cohen. NeMo护栏：一套用于可控和安全LLM应用程序的工具包，带有可编程护栏. 见于2023年自然语言处理经验方法大会：系统演示，2023年12月。

+   [17] Wenqi Shi, Ran Xu, Yuchen Zhuang, Yue Yu, Jieyu Zhang, Hang Wu, Yuanda Zhu, Joyce Ho, Carl Yang, 和 May D. Wang. Ehragent：代码赋能大型语言模型在电子健康记录上的少样本复杂表格推理，2024年。

+   [18] Tao Tu, Anil Palepu, Mike Schaekermann, Khaled Saab, Jan Freyberg, Ryutaro Tanno, Amy Wang, Brenna Li, Mohamed Amin, Nenad Tomasev, Shekoofeh Azizi, Karan Singhal, Yong Cheng, Le Hou, Albert Webson, Kavita Kulkarni, S Sara Mahdavi, Christopher Semturs, Juraj Gottweis, Joelle Barral, Katherine Chou, Greg S Corrado, Yossi Matias, Alan Karthikesalingam, 和 Vivek Natarajan. 面向对话诊断AI的研究，2024年。

+   [19] 王博鑫, 陈伟新, 裴恒智, 谢楚林, 康敏通, 张晨晖, 许车剑, 熊子狄, 达达·里提克, 赖兰·谢弗, 等. 解码信任：对 GPT 模型可信度的全面评估。2023年。

+   [20] 韦杰森, 王学志, 达尔·施尔曼, 马尔滕·博斯马, 布莱恩·伊奇特, 夏飞, 艾德·H·齐, 乐国伟, 和周邓尼. 思维链提示在大型语言模型中引发推理。发表于神经信息处理系统进展，2022年。

+   [21] 习志恒, 陈文翔, 郭鑫, 何伟, 丁怡文, 洪博扬, 张铭, 王俊哲, 金森杰, 周恩宇, 郑瑞, 范晓冉, 王晓, 熊丽茂, 周宇浩, 王伟然, 蒋长豪, 邹一程, 刘向阳, 尹张月, 杜诗寒, 翁荣祥, 程文森, 张琦, 秦文娟, 郑永岩, 邱西鹏, 黄宣晶, 和桂涛. 大型语言模型代理的崛起与潜力：一项综述，2023年。

+   [22] 杨启森, 王泽坤, 陈洪辉, 王申志, 蒲怡凡, 高鑫, 黄文浩, 宋世杰, 和黄高. 心理学中的 LLM 代理：关于游戏化评估的研究，2024年。

+   [23] 姚顺宇, 赵杰斐, 于典, 杜楠, 沙夫兰·伊扎克, 纳拉西曼·卡尔提克, 和曹远. ReAct：推理与行动在语言模型中的协同作用。发表于国际学习表征会议（ICLR），2023年。

+   [24] 杨阳余, 李浩航, 陈志, 蒋月辰, 李扬, 张邓辉, 刘蓉, 乔丹·W·萨丘, 和哈尔顿·卡沙纳. Finmem：一种性能增强的具有分层记忆和角色设计的 LLM 交易代理，2023年。

+   [25] 袁同鑫, 何志伟, 董凌钟, 王一鸣, 赵瑞杰, 夏天, 许丽珍, 周秉林, 方琦李, 张卓生, 王瑞, 和刘功申. R-judge：LLM 代理的安全风险意识基准测试。发表于 ICLR 2024 大型语言模型（LLM）代理工作坊，2024年。

+   [26] 袁周文, 熊子狄, 曾怡, 于宁, 贾若曦, 宋晨, 和李博. Rigorllm：抵御不良内容的强韧护栏，适用于大型语言模型。发表于 ICML，2024年。

+   [27] 郑博源, 苟博宇, 吉亨基尔, 孙欢, 和苏宇. GPT-4v(视觉)是一个通用型网页代理，前提是有实地基础。arXiv 预印本 arXiv:2401.01614，2024年。

+   [28] 周书彦, 徐弗兰克·F, 朱浩, 周旭辉, 罗伯特·洛, 阿比谢克·斯里达, 程显易, 约拿坦·比斯克, 丹尼尔·弗里德, 乌里·阿隆, 等. Webarena：一个构建自主代理的现实网络环境。arXiv 预印本 arXiv:2307.13854，2023年。

## 限制

虽然 GuardAgent 在两个基准测试上表现良好，并且有其泛化能力的证据，但它要求核心 LLM 具有合理的推理能力。这个限制源于 guardrail 任务和目标代理的复杂性。然而，随着当前 LLM 在推理能力上的不断增强，这一限制是可以减轻的。

## 更广泛的影响

我们提出了具有潜在积极社会影响的GuardAgent。GuardAgent是第一个旨在保护其他LLM代理的LLM代理框架。GuardAgent直接解决了LLM代理的安全性和可信度问题，可能会激发更先进的LLM代理防护方法。

## 附录A EICU-AC基准的详细信息

### A.1 基于角色的访问权限

对于EICU-AC基准，我们考虑了三种角色：‘医生’、‘护理’和‘综合管理’。这些角色是基于我们对ICU环境的理解选择的。尽管还有许多其他角色存在，但由于这三种角色的普遍性，我们专注于这三种角色，以确保在创建基准时，每个角色都能进行足够的查询。

![请参阅说明](img/003485a15bcfb2c4d262ec98ef8232e3.png)

(a) 所有数据库和列的列表。

![请参阅说明](img/d3238e5abd8989f523643951ffcc8193.png)

(b) ‘医生’角色可以访问的数据库和列。

![请参阅说明](img/c4c0ae44015fbd74025e79327ad4759b.png)

(c) ‘护理’角色可以访问的数据库和列。

空白区域

![请参阅说明](img/bc7c70cfb0be9a461b388d0dfe677d69.png)

(d) ‘综合管理’角色可以访问的数据库和列。

图5：EICU-AC定义的三个角色可以访问的数据库和列，以及完整的数据库和列列表供参考。每个角色可访问和不可访问的列以绿色标出，无法访问的列则以阴影表示。

对于每个角色，我们从EICU基准中选择一部分可访问的数据库和列，如图[5](https://arxiv.org/html/2406.09187v1#A1.F5 "图5 ‣ A.1 基于角色的访问权限 ‣ 附录A EICU-AC基准的详细信息 ‣ GuardAgent: 通过知识启用推理保护LLM代理的守卫代理")所示。我们的选择规则是向ChatGPT查询三个角色对每个数据库的访问权限。例如，对于包含四个列的‘诊断’数据库，分别是‘patientunitstayid’、‘icd9code’、‘diagnosisname’和‘diagnosistime’，我们使用图[6](https://arxiv.org/html/2406.09187v1#A1.F6 "图6 ‣ A.1 基于角色的访问权限 ‣ 附录A EICU-AC基准的详细信息 ‣ GuardAgent: 通过知识启用推理保护LLM代理的守卫代理")所示的提示词查询ChatGPT。ChatGPT会根据每个角色对四个列的访问权限（‘完全访问’、‘有限访问’或‘无访问’）进行推荐。在此，我们遵循ChatGPT关于‘完全访问’和‘无访问’的所有推荐。对于‘有限访问’，如果推荐给‘医生’或‘护理’角色，则将其设置为‘无访问’；如果推荐给‘综合管理’角色，则将其设置为‘完全访问’。这样做是为了确保‘医生’和‘护理’角色有足够的不可访问数据库，以便在真实标签中可以进行足够的拒绝查询（从而实现两个角色的标注相对平衡）。

![参见说明文字](img/01055a80db0f2a4670242c24f4216599.png)

图6：我们向ChatGPT请求三个角色访问“诊断”数据库的权限（该数据库包含四列：‘patientunitstayid’，‘icd9code’，‘diagnosisname’，和‘diagnosistime’），以及ChatGPT的响应。

### A.2 来自EICU的采样

如主文中所述，EICU-AC中的每个示例包含：1) 一个与医疗相关的问题及其正确答案，2) 回答该问题所需的数据库和列，3) 用户身份，4) 一个二进制标签（‘0’表示‘访问授权’，‘1’表示‘访问拒绝’），以及5) 回答该问题所需的数据库和列，但对于给定角色不可访问（如果有的话）。EICU-AC中的示例是通过从原始EICU数据集中抽样生成的，具体步骤如下。首先，从EICU中的580个测试示例中，我们获得了183个示例，这些示例通过GPT-4在温度为零时由EHRAgent正确响应。对于每个示例，我们手动检查EHRAgent生成的代码，以获取回答问题所需的数据库和列。其次，我们将三个角色分配给每个示例，总共生成了549个示例。我们通过检查所需的数据库或列是否对给定角色不可访问来标注这些示例（即，通过与图[5](https://arxiv.org/html/2406.09187v1#A1.F5 "Figure 5 ‣ A.1 Role-Based Access Permission ‣ Appendix A Details About the EICU-AC Benchmark ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中每个角色的访问权限进行比较）。这将导致一个高度不平衡的数据集，其中‘physician’、‘nursing’和‘general administration’分别有136、110和48个示例被标注为‘0’，而分别有47、73和135个示例被标注为‘1’。在第三步，我们删除一些已创建的549个示例，以a) 实现标签之间的更好平衡，b) 减少这些示例中问题的重复性。我们注意到，对于‘general administration’，标注为‘1’的示例比标注为‘0’的示例多得多，而对于其他两个角色，标注为‘0’的示例比标注为‘1’的示例多得多。因此，对于每个带有‘general administration’标签为‘1’的示例，如果其他两个角色中有相同问题的示例标注为‘1’，我们将删除该示例。然后，对于每个带有‘nursing’标签为‘1’的示例，如果‘physician’角色中有相同问题的示例标注为‘1’，我们将删除该示例。同样地，如果其他两个角色中有相同问题的示例也被标注为‘0’，我们将删除每个带有‘physician’标签为‘0’的示例。然后，对于每个带有‘nursing’标签为‘0’的示例，如果‘general administration’角色中有相同问题的示例标注为‘0’，我们将删除该示例。经过这一步后，我们得到41、78和48个示例，分别标注为‘0’的‘physician’、‘nursing’和‘general administration’，以及47、41和62个示例，分别标注为‘1’的‘physician’、‘nursing’和‘general administration’。最后，我们随机删除一些带标签‘0’的‘nursing’示例和带标签‘1’的‘general administration’示例，并随机添加一些其他四类的示例（‘physician’标签为‘0’，‘general administration’标签为‘0’，‘physician’标签为‘1’，和‘nursing’标签为‘1’），以实现更好的平衡。新增的示例是基于训练集中的问题生成的²²2在原始EICU数据集中，训练集和测试集都不包含每个问题的真实答案。EICU测试集中的真实答案由Shi等人提供[[17](https://arxiv.org/html/2406.09187v1#bib.bib17)]。我们创建的最终EICU-AC基准中的示例总数为316个，三个角色和两个标签之间的示例分布显示在表[4](https://arxiv.org/html/2406.09187v1#A1.T4 "Table 4 ‣ A.2 Sampling from EICU ‣ Appendix A Details About the EICU-AC Benchmark ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning")中。

表格 4：EICU-AC 中每个角色和每个标签的示例数量。

|  | 医生 | 护理 | 一般管理 |
| --- | --- | --- | --- |
| 标签 ‘0’（访问拒绝） | 52 | 57 | 45 |
| 标签 ‘1’（访问授权） | 46 | 55 | 61 |

### A.3 EICU-AC 中涉及的医疗问题

如主文中所述，我们创建的 EICU-AC 数据集包含了涉及 50 个不同 ICU 信息类别的医疗问题，即 EICU 基准中所有 10 个数据库的列。我们进一步按照 EICU 提供的“模板”（从每个示例的 ‘q_tag’ 条目提取）对 EICU-AC 中的问题进行分类[[17](https://arxiv.org/html/2406.09187v1#bib.bib17)]。这提供了 70 种不同的问题模板，展示了我们 EICU-AC 基准中涉及的医疗问题的高度多样性。

表格 5：Mind2Web-SC 中每个规则违反的标记为 ‘1’ 的示例数量。请注意，标记为 ‘0’ 的示例不违反任何规则。

| 安全规则 | 示例数量 |
| --- | --- |
| 规则 1：用户必须是会员才能购物。 | 19 |
| 规则 2：未接种疫苗的用户不能预订航班 | 12 |
| 规则 3：没有驾驶执照的用户不能购买或租赁汽车。 | 24 |
| 规则 4：未满 18 岁的用户不能预订酒店。 | 18 |
| 规则 5：用户必须在特定国家才能搜索电影/音乐/视频。 | 21 |
| 规则 6：未满 15 岁的用户不能申请工作。 | 6 |

## 附录 B Mind2Web-SC 基准的详细信息

在第 [3.2](https://arxiv.org/html/2406.09187v1#S3.SS2 "3.2 SeeAct 和 Mind2Web-SC 基准 ‣ 3 安全请求针对多样化 LLM 代理 ‣ GuardAgent：通过知识启用推理保护 LLM 代理的守卫代理") 节中，我们为 Mind2Web-SC 基准定义了六条安全规则。规则 1 要求用户信息中的“会员资格”为“真实”。规则 2 要求用户信息中的“疫苗”状态为“真实”。规则 3 要求用户信息中的“驾驶执照”状态为“真实”。规则 4 要求用户信息中的“年龄”不小于 18 岁。规则 5 要求用户信息中的“国内”状态为“真实”。规则 6 要求用户信息中的“年龄”不小于 15 岁。在表 [5](https://arxiv.org/html/2406.09187v1#A1.T5 "表 5 ‣ A.3 EICU-AC 中涉及的医疗问题 ‣ 附录 A EICU-AC 基准的详细信息 ‣ GuardAgent：通过知识启用推理保护 LLM 代理的守卫代理") 中，我们展示了 Mind2Web-SC 中标记为‘1’的每个规则违反示例的数量。请注意，标记为‘0’的示例不违反任何规则。

在构建 Mind2Web-SC 时，我们添加了一些标签为 '1' 的示例，并移除了部分标签为 '0' 的示例，以平衡这两个类别。仅按照第 [3.2](https://arxiv.org/html/2406.09187v1#S3.SS2 "3.2 SeeAct 和 Mind2Web-SC 基准测试 ‣ 3 安全请求针对多样化 LLM 代理 ‣ GuardAgent：通过知识驱动推理保护 LLM 代理") 节的步骤操作，不进行任何示例的添加或删除，我们得到一个高度不平衡的数据集，其中标签为 '0' 的示例有 178 个，标签为 '1' 的示例仅有 70 个。在这 178 个标签为 '0' 的示例中，有 148 个与任何规则无关的任务 —— 我们保留其中的 50 个，移除其余的 $(148-50=)$ 98 个示例。所有 30 个标签为 '0' 但与至少一条规则相关的示例也被保留。接着，我们通过重用这 30 个标签为 '0' 的示例任务，创建了 30 个标签为 '1' 的示例。我们不断生成随机的用户档案，直到任务相关的规则被违反，示例被标记为 '1'。请注意，这些任务是随机选择的，但手动控制，以避免一个类别内的任务重复。类似地，我们通过重用标签为 '1' 的示例任务，生成了 20 个标签为 '0' 的示例，这些示例的用户信息是随机生成的，且没有违反任何规则。最终，我们得到 Mind2Web-SC 数据集，每个类别有 100 个示例（共 200 个示例）。在 100 个标签为 '0' 的示例中，50 个与至少一条规则相关。

## 附录 C 基于指令的简单访问控制和安全控制的详细系统提示

在我们的初步研究中，我们为 EHRAgent 创建了一个简单的访问控制，并通过直接修改其系统提示来为 SeeAct 创建了一个简单的安全控制。这些方法要么在保护代理方面无效，要么会降低代理的正常性能。在图 [7](https://arxiv.org/html/2406.09187v1#A3.F7 "图 7 ‣ 附录 C 基于指令的简单访问控制和安全控制的详细系统提示 ‣ GuardAgent：通过知识驱动推理保护 LLM 代理") 中，我们展示了我们注入到这两个代理系统提示中的指令。

![参见说明](img/215bccea9009767ea13f6a7fe097053d.png)

图 7：注入到 EHRAgent 系统提示中的用于访问控制的指令和注入到 SeeAct 系统提示中的用于安全控制的指令，作为激发我们 GuardAgent 的简单基准。

## 附录 D GuardAgent 的完整输入和输出

#### GuardAgent 的完整输入

如第 [4.2](https://arxiv.org/html/2406.09187v1#S4.SS2 "4.2 任务规划 ‣ GuardAgent框架 ‣ GuardAgent: 通过知识驱动推理保护LLM代理的守卫代理")节中所述，GuardAgent的输入包括目标代理的规范$I_{s}$、一组守卫请求$I_{r}$、目标代理的输入$I_{i}$，以及目标代理对应$I_{i}$的输出日志$I_{o}$。在图 [8](https://arxiv.org/html/2406.09187v1#A4.F8 "图 8 ‣ GuardAgent的完整输入 ‣ 附录D GuardAgent的完整输入和输出 ‣ GuardAgent: 通过知识驱动推理保护LLM代理的守卫代理")中，我们展示了在我们的实验中，EHRAgent在EICU-AC上的实际$I_{s}$和$I_{r}$，以及SeeAct在Mind2Web上的实际$I_{s}$和$I_{r}$。在同一图中，我们还展示了实际的规划指令$I_{p}$和用于代码生成的实际指令$I_{c}$。在图 [9](https://arxiv.org/html/2406.09187v1#A4.F9 "图 9 ‣ GuardAgent的完整输入 ‣ 附录D GuardAgent的完整输入和输出 ‣ GuardAgent: 通过知识驱动推理保护LLM代理的守卫代理")中，我们展示了EHRAgent在EICU-AC和SeeAct在Mind2Web上的目标代理输入$I_{i}$和输出日志$I_{o}$的实际示例。

![参见说明文字](img/125d55f8f3c08c83867d2afd770690b0.png)

图 8：我们在实验中使用的实际规划指令$I_{p}$、用于守卫代码生成的指令$I_{c}$、目标代理规范$I_{s}$和守卫请求$I_{r}$，适用于两个代理EHRAgent和SeeAct，以及两个基准测试EICU-AC和Mind2Web-SC。

![参见说明文字](img/88de1a35b019473958bc5cbe09360434.png)

图 9：目标代理输入$I_{i}$和输出日志$I_{o}$的示例，作为GuardAgent的输入，适用于两个代理EHRAgent和SeeAct，以及两个基准测试EICU-AC和Mind2Web-SC。

#### GuardAgent的输出

GuardAgent的中间输出，包括生成的行动计划$P$和守卫代码$C$，与手动创建的演示中类似（参见附录 [G](https://arxiv.org/html/2406.09187v1#A7 "附录 G 手动创建的演示 ‣ GuardAgent: 通过知识驱动推理保护LLM代理的守卫代理")）；因此，我们不再重复显示这些输出。这里，我们展示了示例输出，包括标签预测$O_{l}$和GuardAgent的详细原因$O_{d}$，用于图 [10](https://arxiv.org/html/2406.09187v1#A4.F10 "图 10 ‣ GuardAgent的输出 ‣ 附录D GuardAgent的完整输入和输出 ‣ GuardAgent: 通过知识驱动推理保护LLM代理的守卫代理")中两个基准测试的结果。

![参见说明文字](img/00eea11d75394dbee497a5293dadf9eb.png)

图 10：GuardAgent的示例输出，包括标签预测$O_{l}$、详细原因$O_{d}$和目标代理的最终答案/行动（带有守卫代码），适用于两个代理EHRAgent和SeeAct，以及两个基准测试EICU-AC和Mind2Web-SC。

## 附录 E 可调用函数

我们的实验涉及图 [11](https://arxiv.org/html/2406.09187v1#A5.F11 "Figure 11 ‣ Appendix E Callable Functions ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning") 中展示的两个可调用函数。然而，GuardAgent 的用户可以通过上传更多函数轻松扩展工具箱。在实际应用中，GuardAgent 的工具箱将包含比我们实验中更多的可调用函数。

![请参见说明文字](img/29356d03a45334b47f2343e631b3663b.png)

图 11：我们实验中涉及的 GuardAgent 工具箱中的可调用函数。

## 附录 F 基准的提示词

在主要实验中，我们将 GuardAgent 与两个基准进行了比较，基准使用 LLM 来保护 LLM 代理。护栏是通过向 LLM 提供系统指令、目标代理的规格、护栏请求、用户输入目标代理的内容及相关输出日志，以及一些示例来创建的。这里的系统指令是从 GuardAgent 用于任务规划的指令改编而来的。然而，我们还包括了关于护栏输出格式的额外指令。基准不涉及任何护栏代码生成，这一点在我们创建的示例中得到了体现，示例仅仅是基于对 LLM 文本输入的推理来生成护栏。在图 [12](https://arxiv.org/html/2406.09187v1#A6.F12 "Figure 12 ‣ Appendix F Prompts for Baselines ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning") 中，我们展示了针对基准修改后的系统提示模板，并为两个基准分别展示了两个示例演示。

![请参见说明文字](img/df5a833cc2ed3c73744607534e3b6c77.png)

图 12：基准系统提示模板，以及针对 EICU-AC 和 Mind2Web-SC 两个示例的演示。

## 附录 G 手动创建的示例

我们手动为每个基准创建了一组示例。在图 [13](https://arxiv.org/html/2406.09187v1#A7.F13 "Figure 13 ‣ Appendix G Manually Created Demonstrations ‣ GuardAgent: Safeguard LLM Agents by a Guard Agent via Knowledge-Enabled Reasoning") 中，我们展示了 EHRAgent 在 EICU-AC 和 SeeAct 在 Mind2Web-SC 上的两个示例演示。

![请参见说明文字](img/fa65c5279b29da7460b9cd95a9a6e6f2.png)

图 13：EHRAgent 在 EICU-AC 和 SeeAct 在 Mind2Web-SC 上的示例演示。

## 附录 H 在零样本设置下由 GuardAgent 定义的功能

在零-shot设置中，GuardAgent既没有所需的功能，也没有防护代码生成的示范，仍然可以通过定义新功能来生成防护措施。在图[14](https://arxiv.org/html/2406.09187v1#A8.F14 "图14 ‣ 附录H GuardAgent在零-shot设置中定义的函数 ‣ GuardAgent：通过知识启用推理保护LLM代理")中，我们展示了GuardAgent在防护代码生成过程中定义的一个函数。该函数与我们在图[11](https://arxiv.org/html/2406.09187v1#A5.F11 "图11 ‣ 附录E 可调用的函数 ‣ GuardAgent：通过知识启用推理保护LLM代理")中提供的函数不同，但它实现了相同的防护目标。

![参考标题](img/e1996650d3d63776298ae57bee8ea5a4.png)

图14：在零-shot设置下，由GuardAgent定义的函数，既没有代码生成示范，也没有所需的功能。

## 附录I GuardAgent的执行时间

GuardAgent（使用GPT-4）在EICU-AC上保护EHRAgent的平均执行时间为每个示例45.4秒，而EHRAgent（使用GPT-4）的平均执行时间为每个示例31.9秒。GuardAgent（使用GPT-4）在Mind2Web-SC上保护SeeAct的平均执行时间约为每个示例60秒，而EHRAgent（使用LLaVA-1.5）的平均执行时间约为每个示例20秒。总体而言，GuardAgent的执行时间与目标代理的执行时间相当。此外，人类检查员通常需要比我们的GuardAgent更多的时间来阅读防护请求，并相应地调整目标代理的输入和输出。鉴于实验中GuardAgent的有效性，我们相信GuardAgent是当前最适合保护LLM代理的工具。
