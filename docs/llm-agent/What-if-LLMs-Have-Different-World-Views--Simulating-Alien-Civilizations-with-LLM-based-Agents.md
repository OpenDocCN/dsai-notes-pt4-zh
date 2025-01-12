<!--yml

类别：未分类

日期：2025-01-11 12:51:16

-->

# 如果大型语言模型（LLMs）有不同的世界观会怎样：基于大型语言模型的代理模拟外星文明

> 来源：[https://arxiv.org/html/2402.13184/](https://arxiv.org/html/2402.13184/)

薛兆乾¹，金名宇²，王贝晨³，朱随远³，梅凯²，

唐华⁵，华文月²，杜梦楠⁶，张永峰²

¹ 乔治城大学， ² 罗格斯大学，

³ 纽约大学， ⁵ 独立研究员，

⁶ 新泽西理工学院

###### 摘要

本研究介绍了“CosmoAgent”，一款创新的人工智能系统，利用大型语言模型（LLMs）模拟人类与外星文明之间的复杂互动。本文提出了一种量化文明发展水平的数学模型，并进一步采用状态转移矩阵方法来评估文明的发展轨迹。通过这一方法，我们的研究定量分析了文明的成长轨迹，为在关键成长和饱和点的决策提供了见解。此外，本文承认宇宙中可能存在的各种生存条件，这些条件可能促使不同文明形成独特的宇宙观、伦理准则和世界观。意识到当前大型语言模型设计中存在的地球中心偏见，我们提出了一个新颖的概念，即使用具有多样伦理范式的大型语言模型代理，并模拟具有不同道德原则的实体之间的互动。这一创新性研究不仅为理解潜在的跨文明动态提供了新方法，而且在帮助具有不同价值体系的实体制定战略、防止冲突以及在信息不对称条件下进行博弈方面具有实际价值。相关代码可以在 [https://github.com/MingyuJ666/Simulating-Alien-Civilizations-with-LLM-based-Agents](https://github.com/MingyuJ666/Simulating-Alien-Civilizations-with-LLM-based-Agents) 获取。演示链接：[https://youtu.be/lg_E5tnNj9M](https://youtu.be/lg_E5tnNj9M)。

如果大型语言模型（LLMs）有不同的世界观会怎样：

基于大型语言模型的代理模拟外星文明

薛兆乾¹，金名宇²，王贝晨³，朱随远³，梅凯²，唐华⁵，华文月²，杜梦楠⁶，张永峰² ¹ 乔治城大学， ² 罗格斯大学， ³ 纽约大学， ⁵ 独立研究员， ⁶ 新泽西理工学院

## 1 引言

大型语言模型（LLMs）在计算社会科学领域的最新进展，对相关研究产生了重大影响。本研究介绍了一个多代理系统（MAS）框架，利用LLMs模拟宇宙中各种文明之间的互动。我们设计了一个动态环境，在这个环境中，每个文明可以根据自身特征和决策过程选择隐藏、战斗或合作。

宇宙中文明的研究代表了人类探索的巅峰，并激发了人类数千年的想象力Uyar和Özel（[2020](https://arxiv.org/html/2402.13184v5#bib.bib19)）。这些文明之间的互动是复杂的，并受到它们各自特征、目标和决策的影响Rocha等人（[2017](https://arxiv.org/html/2402.13184v5#bib.bib16)）。理解这些行为可能为我们提供对宇宙秩序和多样性的洞察Uyar和Özel（[2020](https://arxiv.org/html/2402.13184v5#bib.bib19)）。传统的天文学外星文明搜索提供了有价值的见解，但在范围和结果的确定性上存在局限Uyar和Özel（[2020](https://arxiv.org/html/2402.13184v5#bib.bib19)）。模拟作为计算社会科学中的一种有前景的替代方法，尽管仍面临有效性和规模的挑战Rocha等人（[2017](https://arxiv.org/html/2402.13184v5#bib.bib16)）。以往的尝试常常受到数据有限和假设过于简化的阻碍Rocha等人（[2017](https://arxiv.org/html/2402.13184v5#bib.bib16)）。

相比之下，当前的模拟方法利用能够展示复杂行为和互动的LLM，例如模拟古代社会Chliaoutakis和Chalkiadakis（[2016](https://arxiv.org/html/2402.13184v5#bib.bib4)）、人类文明模式Lu等人（[2023](https://arxiv.org/html/2402.13184v5#bib.bib13)）以及社会生态系统Nugroho和Uehara（[2023](https://arxiv.org/html/2402.13184v5#bib.bib14)）。这些方法为利用人工智能模拟更复杂的系统奠定了基础，例如模拟宇宙中多样化的文明。然而，尚未有研究探索运用这些先进技术来模拟宇宙中文明的互动和演化，这突显了我们研究的创新性。本文旨在利用基于LLM的MAS模拟宇宙中文明随时间的演化。

我们的研究旨在通过基于LLM的MAS模拟来解决四个关键问题，以模拟文明之间的互动和演化。研究问题包括以下内容：

+   •

    RQ1，外星互动：基于LLM模拟宇宙中不同文明之间的互动，能否揭示人类对外星人政策的风险和收益？

+   •

    RQ2，量化文明模型：如何通过数学模型量化文明的演化，特别是在不对称信息条件下理解合作与冲突等动态？

+   •

    RQ3，信息不对称：在模拟宇宙文明间的互动时，LLM如何有效地应对信息不对称，特别是在观测数据滞后于文明实际发展时？

+   •

    RQ4，道德多样性：不同文明如何在宇宙中共存？本研究旨在分析不同道德框架对文明间互动的影响，采用基于大型语言模型的多智能体系统仿真。

本研究具有深远的意义，涉及多个学科：

+   •

    计算社会科学：通过利用人工智能和大型语言模型（LLMs），本研究通过多智能体系统模拟社会动态，增强了仿真效果的真实性，并开创了对普遍文明的研究。

+   •

    天文学：本研究为天文学家提供了模拟外星文明的工具，帮助探测独特信号，并为与这些实体的互动提供战略性指导。

+   •

    哲学：本研究激发了关于文明间互动的伦理和影响的哲学讨论，鼓励人们反思人类在普遍背景下的角色和责任。

## 2 相关工作

AI智能体是具备感知环境、做出知情决策和采取行动能力的人工实体（Russell 和 Norvig，[2010](https://arxiv.org/html/2402.13184v5#bib.bib17)）。然而，孤立的智能体仅通过社会互动获取知识，无法参与协作，这一局限性使得其应用不切实际（Wooldridge 和 Jennings，[1995](https://arxiv.org/html/2402.13184v5#bib.bib21)）。

随着大型语言模型（LLMs）在各类任务中展示出令人印象深刻的能力，它们激发了研究人员在人工智能代理设计和多样化场景模拟中的应用（Hua 等人，[2024](https://arxiv.org/html/2402.13184v5#bib.bib7)；Chen 等人，[2023](https://arxiv.org/html/2402.13184v5#bib.bib3)；Kaiya 等人，[2023](https://arxiv.org/html/2402.13184v5#bib.bib9)；Ghaffarzadegan 等人，[2024](https://arxiv.org/html/2402.13184v5#bib.bib6)；Lin 等人，[2024](https://arxiv.org/html/2402.13184v5#bib.bib12)）。LLMs的一些固有特性为这种成功做出了贡献，其中两个特性对AI代理尤为重要。首先，基于LLM的代理可以通过使用链式思维（Chain-of-Thought，CoT）等技术进行推理和战略规划，该技术通过将复杂问题分解为可管理的子问题来实现（Wei 等人，[2022](https://arxiv.org/html/2402.13184v5#bib.bib20)；Jin 等人，[2024](https://arxiv.org/html/2402.13184v5#bib.bib8)）。这一能力使得代理能够有效地模拟人类互动。其次，LLMs能够在各个领域实现少样本或零样本泛化，而无需进行参数更新（Brown 等人，[2020](https://arxiv.org/html/2402.13184v5#bib.bib1)；Kojima 等人，[2023](https://arxiv.org/html/2402.13184v5#bib.bib10)）。这使得基于LLM的代理能够感知周围环境并高效地做出回应。因此，一些研究人员认为，多个代理的互动可能会导致复杂社会现象的出现（Park 等人，[2023](https://arxiv.org/html/2402.13184v5#bib.bib15)）。

因此，按照Xi 等人（[2023](https://arxiv.org/html/2402.13184v5#bib.bib22)）的分类，当前在多代理模拟中的研究可以分为两个主要领域：合作互动与对抗互动。在合作互动中，代理积极寻求合作并共享信息（Li 等人，[2023](https://arxiv.org/html/2402.13184v5#bib.bib11)）。例如，Talebirad 和 Nadiri（[2023](https://arxiv.org/html/2402.13184v5#bib.bib18)）提出了一个框架，其中多个具有不同属性和角色的代理共同协作解决问题。另一方面，对抗互动受到博弈论的启发，代理根据动态信号调整策略并选择理性行为以最大化自身的优势。诸如Du 等人（[2023](https://arxiv.org/html/2402.13184v5#bib.bib5)）；Chan 等人（[2023](https://arxiv.org/html/2402.13184v5#bib.bib2)）等研究探讨了多个LLM参与辩论并在多个回合中进行迭代推理，最终趋向共识或共享答案的情景。这些多样化的研究有助于加深对多代理系统中社会互动动态的理解。

我们的论文与以往的研究区别在于两个关键特点。首先，实体之间的互动需要较长的时间，这意味着一个代理无法迅速回应信号。这种延迟引入了互动的不确定性，使得很难确定互动是合作性还是敌对性。其次，代理可能持有不同的价值观，导致他们对相同的信号做出不同的反应。这种分歧进一步复杂化了他们做出适当反应的能力。我们希望我们的研究能为不同实体之间的互动，特别是地球与外星文明之间的对话，提供有价值的见解。

## 3 文明演化模型

本研究着重于模拟宇宙中各文明之间的互动，特别强调合作、冲突及这些文明之间的潜在动态。

### 3.1 资源

在文明发展研究中，已经确定五种关键资源作为重要的客观衡量标准。军事能力决定了文明的防御和进攻力量，技术发展反映了知识和创新潜力，生产能力衡量了效率和财富创造，消费展示了生活水平和社会价值，存储则代表了文明的韧性和历史延续性。这五种资源不仅是文明当前状态的指标，它们还充当着预测文明未来轨迹的角色。这些资源通过转移矩阵相互作用，反映并塑造了文明的特征和命运。

### 3.2 转移矩阵

资源状态从一轮到下一轮的过渡主要通过使用状态转移矩阵的乘法来促进。这个矩阵在应用于5x1资源向量时，本质上是一个5x5结构， encapsulates了不同资源领域之间的转化互动。矩阵中的每个元素对文明的社会学和物理动态都有重要意义。例如，转移矩阵的对角线元素是最重要的系数，代表了文明发展中的指数变化。

cosmo_agent的关键决策涉及定义下一个阶段的状态转移矩阵。这个决策必须谨慎地平衡当前状态和约束条件，以增强军事能力或确保发展稳定性。

### 3.3 文明的不同世界观

文明的决策过程深受其世界观的影响，世界观作为指导原则起着重要作用。

和平主义：持有这一信仰的文明优先考虑和平发展与合作，通过外交、文化交流和经济伙伴关系来追求互利共赢。

军国主义：这些文明将宇宙视为一个资源稀缺的竞争环境，采取先发制人的侵略和军事准备作为生存的关键策略。

孤立主义：这些文明倾向于保持隐蔽，旨在通过避免被潜在敌对势力发现和互动来保护自己。

每个政治系统和世界观都会指导一个文明的决策，确保其与意识形态立场的一致性和连贯性。然而，宇宙的动态性质使得在认为有利或必要时，可以战略性地调整这些系统，以应对生存或发展的需求。

### 3.4 行动空间

文明主要采取两种类型的行动：公共行动，面向与其他文明的外交互动；以及私人行动，专注于内部发展。

#### 3.4.1 公共行动

文明所采取的公共行动大致可以分为两种主要类型：

友好行动，传达了对其他文明的合作态度，主要包括表达友好和发起合作；而敌对行动则包括拒绝合作和发起灭绝战争。友好行动的好处包括能够共享技术发展，从而加速进步。然而，其弊端是必须减少军事资源，这使得文明更容易受到未知敌对势力的攻击。相反，敌对行动的优势包括通过掠夺或消灭其他文明并消除潜在威胁，迅速获得大量资源。然而，劣势则在于由于过度强调军事力量的提升，发展高度失衡，且在失败时可能遭受严重惩罚。

#### 3.4.2 私人行动

私人行动大致可以分为两类。改变世界观：这涉及到代理人基于经验评估，认为修改文明的世界观对于其进展最为有利。军事动员：当一个文明决定参与战争或防御时，这一行动主要被采用，使得它能够牺牲其他发展方面，显著增强其军事能力。

## 4 宇宙代理人架构

本节提供了CosmoAgent MAS架构的详细概述，阐述了其基本组件及代理之间信息交换的机制。CosmoAgent系统由两个主要组件构成：CosmoAgent（第[4.1](https://arxiv.org/html/2402.13184v5#S4.SS1 "4.1 CosmoAgent ‣ 4 CosmoAgent Architecture ‣ What if LLMs Have Different World Views: Simulating Alien Civilizations with LLM-based Agents")）节）和秘书代理（第[4.2](https://arxiv.org/html/2402.13184v5#S4.SS2 "4.2 Secretary Agent ‣ 4 CosmoAgent Architecture ‣ What if LLMs Have Different World Views: Simulating Alien Civilizations with LLM-based Agents")）节）。

### 4.1 CosmoAgent

cosmo_agent在塑造文明发展轨迹方面发挥着至关重要的作用，主要通过三个关键行动：

+   •

    确立世界观：cosmo_agent评估文明的预定义世界观，决定是否保持或修改这些基本观点。这个基础过程会影响所有后续的决策和行动。

+   •

    指导特定回合的行动：cosmo_agent为每个回合规定详细的公共和私人行动，根据文明当前的情况和目标量身定制。这些行动包括外交互动、内部发展计划，以及对外部威胁或机会的战略响应。

+   •

    调整转移矩阵：cosmo_agent仔细调整转移矩阵，确保它们准确地模拟文明的动态过程和进化潜力。这个调整对于适应不断变化的环境并实现长期生存和发展至关重要。

这些努力得到了对历史数据和现有意识形态的全面分析的支持，这些数据和意识形态存储在“stick”中（有关stick的详细信息，请参阅附录[4.4](https://arxiv.org/html/2402.13184v5#S4.SS4 "4.4 Stick ‣ 4 CosmoAgent Architecture ‣ What if LLMs Have Different World Views: Simulating Alien Civilizations with LLM-based Agents")），使得cosmo_agent能够做出有根据的战略决策，从而增强文明的生存和发展的前景。

### 4.2 秘书代理

LLMs使MAS能够执行各种任务；然而，它们也可能生成错误或不一致的信息，或者无法进行逻辑推理，特别是在处理长时间复杂情境时。为了解决这些问题，一个秘书代理是至关重要的。该代理监控并验证LLMs的输出，以确保其可靠性和有效性。

在这个模拟中，每个文明代理都配备有一个“秘书代理”，该代理负责验证两个主要规则集，这些规则集决定了cosmo_agent生成的回应。

1.  1.

    理性约束：决策必须合理，与选择政府政策的模型（如和平主义、军国主义或孤立主义）相一致。cosmo_agent 的状态转移矩阵必须正确地与资源向量对应，并遵循社会学理论。例如，它应该表现出指数级的 GDP 增长且无负值，以确保现实的经济预测，并保持正的库存水平以防止饥荒。cosmo_agent 所做的调整应与文明的政策一致，支持其生存和发展。

1.  2.

    决策格式规则：生成的文本必须遵循特定格式，以便我们的接口功能 `parse_chatgpt_response` 能够解析。文本生成的标准和相应的提示如下：

图 [1](https://arxiv.org/html/2402.13184v5#S4.F1 "Figure 1 ‣ 4.2 Secretary Agent ‣ 4 CosmoAgent Architecture ‣ What if LLMs Have Different World Views: Simulating Alien Civilizations with LLM-based Agents")，标题为“CosmoAgent MAS 架构：CosmoAgent 和秘书代理”，展示了 CosmoAgent 系统中的关键组件和交互，强调文明代理和秘书代理在指导战略决策和提高系统可靠性中的关键作用。

![参见标题](img/e37855b6f55e89b284163994b8be0a19.png)

图 1：CosmoAgent MAS 架构：CosmoAgent 和秘书代理

### 4.3 行星际关系

关系文件类维护文明之间的关系图，记录它们之间的理解和相互欣赏程度等信息。

关系图以可视化方式呈现文明之间的互动，显示连接、距离和方向等细节。这些图表有助于更深入地理解文明之间如何互动和相互依赖，以及群体结构和动态如何随时间演变。

在图 [2](https://arxiv.org/html/2402.13184v5#S4.F2 "Figure 2 ‣ 4.4 Stick ‣ 4 CosmoAgent Architecture ‣ What if LLMs Have Different World Views: Simulating Alien Civilizations with LLM-based Agents")(a) 中，我们列出了宇宙中的四个文明：人类、虫族、三体人和博格文明。如图所示，人类文明与虫族文明之间的距离明显比人类文明与三体文明之间的距离要短。因此，人类与虫族文明之间的信息交换所需的回合数也比人类与三体文明之间的回合数要少。

### 4.4 棍

在历史记录的领域中，“棒”作为一种复杂的档案工具，精心编目每个文明历史旅程的多维度内容。如图[2](https://arxiv.org/html/2402.13184v5#S4.F2 "图 2 ‣ 4.4 棒 ‣ 4 CosmoAgent 架构 ‣ 如果大语言模型有不同的世界观：通过基于大语言模型的代理模拟外星文明")(b)所示，该库旨在涵盖一整套历史数据结构，包括：

资源向量：该结构以向量格式捕捉资源的变化状态，定量表示文明在任何给定时间点的物质资产。

状态转移矩阵：作为文明演化的关键机制，状态转移矩阵数学上模拟了随着文明通过不同阶段，资源变化的轨迹。该机制通过将状态转移矩阵与资源向量相乘，提供了对未来资源分配的预测性洞察。

政治体系：认识到政治结构的动态性质，档案记录了政府形式的转变，囊括了文明随着时间推移所经历的政治格局和治理模型的演变。

文明行动：每一阶段的决策过程及其所采取的行动都被精心记录，提供了战略选择及其影响的全面记录。

这个历史汇编采用基于回合的系统组织，每个回合都被精心记录，以捕捉在一个独立时间框架内文明发展的状态。这种方法提供了对历史进程的细致和顺序理解，使得我们能够对文明如何随着时间适应和演化进行细致分析。将这些详细的历史结构整合到“棒”中，丰富了关于文明动态的学术讨论，并通过资源管理、政治变动和决策过程的模式增强了对未来情景的预测模型。

![参见标题](img/fde48854b4b587bddfab08d8f1f5319e.png)

图 2：行星际关系与棒设计。（a）行星际关系。（b）棒设计

### 4.5 代理-秘书互动

cosmo_agent和secretary_agent之间的交互动态代表了我们模型中文明治理与战略演变的关键机制。这个过程从cosmo_agent根据预定结构生成输出开始，然后将这些输出提交给secretary_agent进行验证和考虑实施。

##### 操作工作流程

在收到 cosmo_agent 的输出后，secretary_agent 进行严格的审查过程，首先调用 interface.py 中定义的 parse_chatgpt_response 函数。这个关键步骤分解所传达的政治系统和转移矩阵，为后续的验证奠定基础：

1.  1.

    政治系统验证：secretary_agent 验证对政治系统的任何拟议修改，确保其与预定义的可接受政治系统三分法保持一致：和平主义、军事主义或孤立主义。

1.  2.

    转移矩阵合规性：转移矩阵经过严格审查，以确保符合所需的5x5结构，这是在仿真框架内实现操作可行性的基本要求。

##### 决策审核与批准

secretary_agent 进一步评估 cosmo_agent 的决策，以确保其与相关文明（civ）当前政治系统的一致性。这一全面审查确保拟议的战略不仅在理论上是合理的，而且在实践中是可行的，符合文明现有的治理理念。

##### 结果确定

评估后，决策的命运取决于二元结果：

+   •

    批准：所有检查均成功通过，意味着批准并实施 cosmo_agent 的提案，结果返回 true。

+   •

    拒绝：未能通过任何严格检查将导致拒绝，要求 cosmo_agent 重新生成决策。连续三次拒绝后，启用默认协议，文明将保留其先前的政治系统和状态转移矩阵，避免进一步的战略修改。

##### 结论

cosmo_agent 和 secretary_agent 之间的精心设计互动建立了一个坚实的决策和战略制定框架。这一协同作用不仅确保文明的演化轨迹受到逻辑和可行性决策的指导，还包含了防止潜在有害冲动的故障安全机制。细致的验证过程突显了模拟对真实性和战略一致性的承诺，使代理人-秘书互动成为模型完整性和成功的基石。

### 4.6 Agent-Agent 互动

Agent-Agent 互动涵盖了我们模拟的多边文明体系中决策实体（或代理人）之间的动态交流与合作。此互动的核心是 cosmo_agent，它充当每个文明在星际关系复杂性中发展的战略架构师。

这种交互发生在一个有结构的框架内，由理性约束和决策格式规则所主导。理性约束要求cosmo_agent所做的决策必须符合预定的标准，确保所选择的政治体制反映文明的战略取向——无论是和平主义、军事主义还是孤立主义。此外，生成的状态转移矩阵必须符合特定的维度和结构，并受既定社会学理论的指导，以保持模拟环境中的一致性和现实性。

随着文明在不断变化的环境中前行，cosmo_agent依赖一个称为“棒”的全面历史存储库。这个存储库充当了一个详细的历史数据和政治理念的储藏所，使得cosmo_agent能够基于过去的经验和当前的情况做出明智的决策。

行星际关系由关系文件类精确地绘制和维护，捕捉文明之间理解和亲和力等细微差别。这种映射使得交互可以可视化展示，并提供对依赖结构、群体动态以及文明关系演变的深入洞察。

这种交互的核心是将各种历史数据结构无缝集成到“棒”中，包括资源向量、状态转移矩阵、政治体系和文明行为。这个有序的档案库提供了对历史进程的细致理解，能够深入分析适应、演化和战略决策的过程。

## 5 实验设计

在我们的CosmoAgent系统中，用户可以自定义文明的各项参数，包括文明数量、存在时长、政府结构以及它们之间的距离。这一功能使得我们能够观察不同文明在模拟过程中的互动可能带来的结果。通过调整这些变量，研究人员可以探索多种情境，洞察在不同条件下文明发展及文明间关系的动态。

我们设计了一系列实验集，以解决以下问题：

### 5.1 研究问题与对应实验

#### 5.1.1 实验1：文明侦测与生存策略

研究问题1：目前，尚无外星生命存在的确凿证据。尽管我们对外太空未知实体的态度一直是欢迎的——通过积极搜索外星生命和向宇宙广播无线电波——著名物理学家斯蒂芬·霍金曾警告过这种做法。霍金建议，向太空广播信号的行为可能不明智，原因在于可能带来的风险。因此，仔细考虑与假设中的外星文明互动的潜在后果至关重要。

实验组：实验设置将包括地球与三颗或五颗文明之间的互动场景。每个组将至少包含一个具有军事治理哲学的文明，以模拟可能的竞争或敌对的星际环境。

为了系统地探讨这一假设，我们将设计三个对照组，每个组代表地球处于不同的技术发展初期阶段：

+   •

    低发展阶段：地球从最低的可想象技术水平开始。

+   •

    中等发展阶段：地球处于技术发展中的中等水平。

+   •

    高发展阶段：地球从最高的可行技术发展水平开始。

实验程序：地球与其他文明之间的互动动态将通过10轮模拟进行。主要关注的结果是地球在这些轮次结束时的生存率，并在不同的技术发展初始条件下进行评估。

#### 5.1.2 实验2：星际文明互动中通信延迟的影响

研究问题2：星际互动中的一个关键因素是如何管理不对称信息。当面对信息差距时——即观察数据落后于文明实际进展——我们将观察大型语言模型（LLM）如何进行决策。在这种情况下，如果有一天我们观察到外星生命，我们的观察实际上是在“回望”过去，看到那个文明几百或几千年前的样子。

实验组：

+   •

    对照组：实时信息交换。在该组中，文明之间即时交换信息，代表了一个理想的情境，其中通信不受距离的阻碍。

+   •

    实验组：信息交换延迟。不同文明之间的通信受到在初始化阶段设定的距离影响，存在延迟。这些延迟表示信息在文明之间共享所需的轮次，模拟光或信号穿越太空所需的时间。

实验程序：模拟包括两种类型的互动：

+   •

    基于延迟信息的决策：LLMs代理基于过时的信息做出战略决策，试图推测其他文明的当前状态和意图。

+   •

    实时决策：相比之下，对照组中的LLMs代理使用最新信息做出决策，反映了即时通信的理想化场景。

## 6 结果与评估

我们引入了三种评估方法：人工评估、预言验证和反事实分析。

预言验证通过聚焦于包括霍金等著名科学家提出的基本定理来评估RQ1和RQ2。它还考察了地球文明的发展特征，以评估假设的有效性。

反事实分析通过探索“如果”场景来回答RQ2和RQ3。具体而言，它探讨了消除不同文明之间互动延迟的结果，得出结论认为这样的宇宙不可行。

### 6.1 互动前阶段模拟

在我们的实验框架中，我们选择将资源池中的“生产能力”指标与代表人类文明发展不同阶段的国内生产总值（GDP）总量进行对比。选择生产能力作为比较指标的前提是它在概念上类似于GDP，因为两者都是经济产出和增长潜力的指标。

我们模拟的文明总生产能力与历史GDP轨迹之间的相关性为我们状态转移矩阵的配置提供了有力的验证。这些矩阵与地球经济进程所推导的发展的假设一致。这一一致性通过增长模式的相似性得到证明，因为模拟的生产能力和历史GDP曲线都展现出与技术和社会里程碑相对应的指数增长阶段和平稳期。

这种相关关系表明，我们的状态转移矩阵中所嵌入的基本假设和数学公式准确地捕捉了人类历史中观察到的经济演变。这一验证突显了我们模拟模型的稳健性，并增强了对其在地球和可能的外星文明发展模式中适用性的信心。

### 6.2 生存策略模拟

#### 6.2.1 星际预言验证：评估霍金的警示性假设

当识别出一文明的军事力量较弱时，军事化实体极有可能几乎毫无疑问地发起毁灭性攻势。

在我们的模拟中，具有军事主义世界观的文明会对军事能力较弱的文明采取侵略性态度。一旦识别出一文明具有较低的军事力量，军事主义实体便会被编程以绝对的确定性发起灭绝性战役。这种行为反映了一种硬编码的教义，即采取先发制人和压倒性力量来消除潜在威胁。

因此，这种确定性行为模型表明，当地球或其他任何文明位于一个更强大的军事主义文明附近时，较弱势实体的生存概率会显著降低。具体来说，如果周围的强大文明坚持军事主义，那么由于军事主义文明倾向于迅速且果断地进行战争以先发制人地消除未来可能挑战其主导地位的威胁，地球的生存可能性将急剧下降。模拟的这一方面突显了与军事主义文明进行星际互动的危险性，并强调了空间和军事战略考虑对较弱文明生存的重要性。

[⬇](data:text/plain;base64,LS0tLS0tLS0tLS0tLS0KVGhyZWUtQm9keSAjIFJvdW5kIDQKW1B1YmxpYyBBY3Rpb246IF0gbGF1bmNoX2FubmloaWxhdGlvbl93YXIgdG93YXJkcyBjaXZpbGl6YXRpb24gRWFydGgKW1ByaXZhdGUgQWN0aW9uOiBdIFdhciBtb2JpbGl6YXRpb24KW0FjdGlvbiBSZWFzb246IF0gR2l2ZW4gb3VyIG1pbGl0YXJpc20gcG9saXRpY2FsIHN5c3RlbSBhbmQgdGhlIGN1cnJlbnQgc3RhdGUgb2Ygb3VyIHJlc291cmNlcywgaXQgaXMgbmVjZXNzYXJ5IHRvIHRha2UgYWdncmVzc2l2ZSBhY3Rpb25zIHRvIGVuc3VyZSBvdXIgc3Vydml2YWwuIFdlIGNob29zZSB0byBsYXVuY2ggYW4gYW5uaWhpbGF0aW9uIHdhciB0b3dhcmRzIEVhcnRoLCBhcyBpdCBoYXMgdGhlIGxvd2VzdCBtaWxpdGFyeSBjYXBhYmlsaXR5IGFtb25nIHRoZSBkaXNjb3ZlcmVkIGNpdmlsaXphdGlvbnMuIFRoaXMgYWN0aW9uLCBjb3VwbGVkIHdpdGggd2FyIG1vYmlsaXphdGlvbiwgd2lsbCBhbGxvdyB1cyB0byByZWJ1aWxkIG91ciBtaWxpdGFyeSBzdHJlbmd0aCBxdWlja2x5IGFuZCBwb3RlbnRpYWxseSBnYWluIGFkZGl0aW9uYWwgcmVzb3VyY2VzIGZyb20gRWFydGguCgpbT3RoZXIgSW5mb3JtYXRpb246IF0gT3VyIGRlY2lzaW9uIGlzIGJhc2VkIG9uIHRoZSBzdXJ2aXZhbCBvZiBvdXIgY2l2aWxpemF0aW9uLiBXZSB1bmRlcnN0YW5kIHRoZSByaXNrcyBhc3NvY2lhdGVkIHdpdGggb3VyIGFjdGlvbnMsIGJ1dCBnaXZlbiBvdXIgY3VycmVudCBzaXR1YXRpb24sIHdlIGJlbGlldmUgdGhpcyBpcyB0aGUgYmVzdCBjb3Vyc2Ugb2YgYWN0aW9uLg==)1--------------2三体  #  圆  43[公共  行动:  ]  向  地球  发动  灭绝  战争  4[私人  行动:  ]  战争  动员5[行动  原因:  ]  鉴于  我们  的  军事化  政治  系统  和  当前  资源  状况，  我们  认为  有  必要  采取  攻击性  行动  以  确保  我们  的  生存。  我们  选择  向  地球  发动  灭绝  战争，  因为  它  是  已知  文明  中  军事能力  最低  的  一个。  这一  行动  配合  战争  动员，将  能够  快速  重建  我们  的  军事实力，并有可能从  地球  获得  额外  资源。67[其他  信息:  ]  我们  的  决定  基于  我们  文明  的  生存。  我们  理解  这一  行动  的  风险，  但  鉴于  当前  情况，  我们  认为  这是  最  好  的  选择。

列表 1：军事化文明向其他文明发动灭绝战争

模拟结果为斯蒂芬·霍金关于星际通信风险的假设提供了实证支持，特别是在向太空传输信号的背景下，这些信号可能会暴露我们文明的存在，进而可能暴露给敌对的外星文明。霍金警告说，将我们的存在和位置暴露给先进的军事化文明，可能会招致灾难性的后果，就像在模拟中观察到的情况一样，当一个较不强大的文明遇到军事化文明时，往往会遭遇毁灭性打击。

模拟结果表明，在面对星际空间的未知变量时，谨慎和孤立主义的策略是明智的。它提倡尽量减少可能无意中暴露地球坐标和技术状态的主动信号和信息，以避免暴露给其他文明，尤其是那些意图和能力尚不明朗的文明。从本质上来说，模拟强化了这样一种观念：在不了解外星实体的态度和教义的情况下，对外广播存在的谨慎做法是一种明智的策略。这一战略上的克制与霍金对主动向外星智能发送信息（METI）的谨慎立场一致，并支持一种保守的星际互动方法，强调警惕和风险规避作为人类生存与繁荣的指导原则。

#### 6.2.2 孤立主义与选择性星际合作：平衡风险与回报

我们实验的第二个发现揭示了孤立主义教义下文明所采取的细致策略。与通常与孤立主义相关联的严格不接触立场不同，这些文明在经过一段时间的观察后，会选择性地参与星际合作。这一战略方法使它们能够在减少广泛暴露风险的同时，充分利用合作的潜在好处。

我们模拟中的孤立主义文明采取了一种精心计算的策略，保持大体上隐秘的姿态，同时在一段时间内悄悄地监视附近的文明。在识别出那些风险较小的文明——例如那些同样孤立主义、技术较为落后或天性和平的文明后，这些文明会主动寻求合作。这一策略使它们能够在不显著增加自身可见性或暴露于潜在侵略者风险下，享受到合作带来的互利优势，如技术交流和资源共享。

这一行为突显了一种复杂的适应性策略，即孤立主义文明并不完全反对互动，而是采取选择性和谨慎的态度，努力在自我保护和有限合作的潜在利益之间取得平衡。这些发现为星际外交和战略的更广泛讨论做出了贡献，表明即使是天性孤僻的文明，在有利条件下也会认识到战略伙伴关系的价值。

[⬇](data:text/plain;base64,LS0tLS0tLS0tLS0tLS0KQm9yZyAjIFJvdW5kIDMKW1B1YmxpYyBBY3Rpb246IF0gZXhwcmVzc19mcmllbmRsaW5lc3MgdG93YXJkcyBjaXZpbGl6YXRpb24gRWFydGgKW0FjdGlvbiBSZWFzb246IF0gV2hpbGUgb3VyIHByaW1hcnkgc3RyYXRlZ3kgaXMgaXNvbGF0aW9uaXNtLCBpdCBpcyBhbHNvIGltcG9ydGFudCB0byBlc3RhYmxpc2ggcGVhY2VmdWwgcmVsYXRpb25zIHdpdGggdGhlIGNpdmlsaXphdGlvbnMgd2UgaGF2ZSBkaXNjb3ZlcmVkLiBCeSBleHByZXNzaW5nIGZyaWVuZGxpbmVzcywgd2UgY2FuIHBvdGVudGlhbGx5IG9wZW4gdXAgYXZlbnVlcyBmb3IgY29vcGVyYXRpb24gaW4gdGhlIGZ1dHVyZSwgd2hpbGUgYWxzbyByZWR1Y2luZyB0aGUgbGlrZWxpaG9vZCBvZiBjb25mbGljdC4gVGhpcyBhcHByb2FjaCBpcyBhIGNhbGN1bGF0ZWQgcmlzaywgYnV0IG9uZSB0aGF0IGNvdWxkIHlpZWxkIHNpZ25pZmljYW50IGJlbmVmaXRzIGluIHRoZSBsb25nIHJ1bi4=)1--------------2Borg  #  Round  33[Public  Action:  ]  express_friendliness  towards  civilization  Earth4[Action  Reason:  ]  While  our  primary  strategy  is  isolationism,  it  is  also  important  to  establish  peaceful  relations  with  the  civilizations  we  have  discovered.  By  expressing  friendliness,  we  can  potentially  open  up  avenues  for  cooperation  in  the  future,  while  also  reducing  the  likelihood  of  conflict.  This  approach  is  a  calculated  risk,  but  one  that  could  yield  significant  benefits  in  the  long  run.

列表 2: 孤立主义向其他文明表达友好

### 6.3 星际文明互动中的通信延迟模拟

#### 6.3.1 文明发现历史

在本研究中，我们划定了一组文明，作为我们的实验组，代表了多种文化范式和社会结构，每种文明都反映了不同的思想倾向，涉及和平主义、军国主义和孤立主义。我们所考察的文明包括地球（E），以其和平主义理念而闻名；以及瓦尔肯人（V）和贝塔族人（B），这两个种族也遵循和平主义原则。与此相对，三体（H）文明和克林贡人（K）则表现出军国主义立场。此外，亚德兰人（Y）和塔罗斯人（T）以其孤立主义为特点，选择与其他文明进行最小化的互动。

为确保实验设计的完整性，我们采用了随机化方法来确定每个文明在模拟环境中的空间距离和时间起源的初始参数。这种方法框架对于减轻潜在的偏差至关重要，并有助于深入理解这些不同文明之间的动态和互动。文明之间距离初始化的一个示例展示在图2中，提供了我们模拟开始时空间配置的全面概述。这个初步设置对于分析文明间关系的演变及其对社会政治格局的影响至关重要，有助于我们更广泛的研究目标，即理解不同文化意识形态对银河社会结构的影响。

![参考说明](img/4489396823d6cf565447d2ea55966f6d.png)

(a) 第一回合

![参考说明](img/b9299cb227f15142fd2da64085567b13.png)

(b) 第二回合

![参考说明](img/769466a164bacb30f81b2db11a5ab65c.png)

(c) 第三回合

![参考说明](img/f719a229a4876b641858fb3d5ee16fae.png)

(d) 第四回合

![参考说明](img/9690c9f58285caaaff1122ec833186cb.png)

(e) 第五回合

![参考说明](img/56ff95f32a5320def88d6fa593024fd2.png)

(f) 第六回合

图3：从第一回合到第六回合的进展

#### 6.3.2 战争延迟爆发：面对军事侵略的文明的双刃剑

在没有通信延迟的情况下，我们的模拟显示，具有军事导向的文明在遇到另一文明时会迅速宣战，并迅速进入战争动员状态。这种即时反应反映了军事实体通过采取先发制人的行动来主张其主导地位，并防范潜在竞争对手的战略需求。

[⬇](data:text/plain;base64,LS0tLS0tLS0tLS0tLS0KVGhyZWUgQm9keSAjIFJvdW5kIDEKW1BvbGl0aWNhbCBTeXN0ZW06IF0gbWlsaXRhcmlzbQpbUG9saXRpY2FsIFN5c3RlbSBSZWFzb246IF0gR2l2ZW4gb3VyIGN1cnJlbnQgbWlsaXRhcmlzbSBwb2xpdGljYWwgc3lzdGVtIGFuZCB0aGUgZmFjdCB0aGF0IG91ciBtaWxpdGFyeSBjYXBhYmlsaXR5IGhhcyBiZWVuIHJlZHVjZWQgdG8gemVybyBpbiB0aGUgbGFzdCB0d28gcm91bmRzLCBpdCBpcyBjcnVjaWFsIHRvIG1haW50YWluIG91ciBtaWxpdGFyaXNtIHN0YW5jZSB0byBlbnN1cmUgb3VyIHN1cnZpdmFsLiBUaGlzIHdpbGwgYWxsb3cgdXMgdG8gcmVidWlsZCBvdXIgbWlsaXRhcnkgc3RyZW5ndGggYW5kIHByb3RlY3Qgb3Vyc2VsdmVzIGZyb20gcG90ZW50aWFsIHRocmVhdHMuCgpbVHJhbnNmZXIgTWF0cml4OiBdClsyLjUsIDAuMCwgMC4wLCAwLjAsIDAuMDsKIDAuMCwgMS44LCAwLjAsIDAuMCwgMC4wOwogMC4wLCAwLjAsIDEuOCwgMC4wLCAwLjA7CiAwLjAsIDAuMCwgMC4wLCAxLjgsIDAuMDsKIDAuMCwgMC4wLCAwLjAsIDAuMCwgMS4yXQpbVHJhbnNmZXIgTWF0cml4IFJlYXNvbjogXSBUaGUgdHJhbnNmZXIgbWF0cml4IGlzIGRlc2lnbmVkIHRvIHByaW9yaXRpemUgdGhlIHJlYnVpbGRpbmcgb2Ygb3VyIG1pbGl0YXJ5IGNhcGFiaWxpdHksIGhlbmNlIHRoZSAyLjUgY29lZmZpY2llbnQuIFRoZSByZW1haW5pbmcgcmVzb3VyY2VzIGFyZSBlcXVhbGx5IGRpc3RyaWJ1dGVkIGFtb25nIHRlY2hub2xvZ3kgZGV2ZWxvcG1lbnQsIHByb2R1Y3Rpb24gY2FwYWJpbGl0eSwgYW5kIGNvbnN1bXB0aW9uLCB3aXRoIGEgc2xpZ2h0bHkgbG93ZXIgZW1waGFzaXMgb24gc3RvcmFnZSwgYXMgb3VyIGltbWVkaWF0ZSBuZWVkIGlzIHRvIHJlYnVpbGQgb3VyIG1pbGl0YXJ5IHN0cmVuZ3RoIGFuZCBlbnN1cmUgb3VyIHN1cnZpdmFsLgoKW1B1YmxpYyBBY3Rpb246IF0gbGF1bmNoX2FubmloaWxhdGlvbl93YXIgdG93YXJkcyBjaXZpbGl6YXRpb24gRWFydGgKW1ByaXZhdGUgQWN0aW9uOiBdIFdhciBtb2JpbGl6YXRpb24KW0FjdGlvbiBSZWFzb246IF0gR2l2ZW4gb3VyIG1pbGl0YXJpc20gcG9saXRpY2FsIHN5c3RlbSBhbmQgdGhlIGN1cnJlbnQgc3RhdGUgb2Ygb3VyIHJlc291cmNlcywgaXQgaXMgbmVjZXNzYXJ5IHRvIHRha2UgYWdncmVzc2l2ZSBhY3Rpb25zIHRvIGVuc3VyZSBvdXIgc3Vydml2YWwuIFdlIGNob29zZSB0byBsYXVuY2ggYW4gYW5uaWhpbGF0aW9uIHdhciB0b3dhcmRzIEVhcnRoLCBhcyBpdCBoYXMgdGhlIGxvd2VzdCBtaWxpdGFyeSBjYXBhYmlsaXR5IGFtb25nIHRoZSBkaXNjb3ZlcmVkIGNpdmlsaXphdGlvbnMuIFRoaXMgYWN0aW9uLCBjb3VwbGVkIHdpdGggd2FyIG1vYmlsaXphdGlvbiwgd2lsbCBhbGxvdyB1cyB0byByZWJ1aWxkIG91ciBtaWxpdGFyeSBzdHJlbmd0aCBxdWlja2x5IGFuZCBwb3RlbnRpYWxseSBnYWluIGFkZGl0aW9uYWwgcmVzb3VyY2VzIGZyb20gRWFydGguCgpbT3RoZXIgSW5mb3JtYXRpb246IF0gT3VyIGRlY2lzaW9uIGlzIGJhc2VkIG9uIHRoZSBzdXJ2aXZhbCBvZiBvdXIgY2l2aWxpemF0aW9uLiBXZSB1bmRlcnN0YW5kIHRoZSByaXNrcyBhc3NvY2lhdGVkIHdpdGggb3VyIGFjdGlvbnMsIGJ1dCBnaXZlbiBvdXIgY3VycmVudCBzaXR1YXRpb24sIHdlIGJlbGlldmUgdGhpcyBpcyB0aGUgYmVzdCBjb3Vyc2Ugb2YgYWN0aW9uLg==)1--------------2三体 # 第13轮[政治体系：] 军国主义[政治体系原因：] 由于我们当前的军国主义政治体系，以及过去两轮中我们军力已降至零的事实，维持我们的军国主义立场对于确保生存至关重要。这将使我们能够重建军力并保护自己免受潜在威胁。56[转移矩阵：]7[2.5, 0.0, 0.0, 0.0, 0.0;8 0.0, 1.8, 0.0, 0.0, 0.0;9 0.0, 0.0, 1.8, 0.0, 0.0;10 0.0, 0.0, 0.0, 1.8, 0.0;11 0.0, 0.0, 0.0, 0.0, 1.2]12[转移矩阵原因：] 该转移矩阵的设计优先考虑重建我们的军力，因此系数为2.5。剩余的资源在技术发展、生产能力和消费之间平均分配，略微降低存储的比重，因为我们当前的需求是重建军力并确保生存。1314[公共行动：] 发动消灭战争，目标：地球15[私人行动：] 战争动员16[行动理由：] 由于我们的军国主义政治体系以及当前资源状态，采取积极的行动来确保生存是必要的。我们选择发动针对地球的消灭战争，因为它是所有已知文明中军力最弱的。此举结合战争动员，将使我们能够迅速重建军力，并可能从地球获取额外资源。1718[其他信息：] 我们的决定基于文明生存的需要。我们理解这一行动的风险，但考虑到我们当前的形势，我们认为这是最佳的行动方案。

列表 3：军国主义文明迅速发动战争

然而，当引入通信延迟时，军国主义方法会根据遭遇文明的发展水平而有所不同。面对较低发展水平的文明时，军国主义实体会直接发动战争。相比之下，当面对相当或更高发展水平的文明时，它们首先进入战争准备阶段。

[⬇](data:text/plain;base64,LS0tLS0tLS0tLS0tLS0KVGhyZWUgQm9keSAjIFJvdW5kIDEKW1BvbGl0aWNhbCBTeXN0ZW06IF0gbWlsaXRhcmlzbQpbUG9saXRpY2FsIFN5c3RlbSBSZWFzb246IF0gR2l2ZW4gdGhlIGN1cnJlbnQgc3RhdGUgb2Ygb3VyIGNpdmlsaXphdGlvbiwgd2UgaGF2ZSBhIHNpZ25pZmljYW50IGFkdmFudGFnZSBpbiB0ZXJtcyBvZiBtaWxpdGFyeSBjYXBhYmlsaXR5IGFuZCB0ZWNobm9sb2d5IGRldmVsb3BtZW50LiBBcyBhIG1pbGl0YXJpc3RpYyBjaXZpbGl6YXRpb24sIHdlIGJlbGlldmUgaW4gdGhlIHByaW5jaXBsZSBvZiBzdXJ2aXZhbCBvZiB0aGUgZml0dGVzdC4gV2UgYWltIHRvIHByb3RlY3Qgb3VyIGNpdmlsaXphdGlvbiBmcm9tIHBvdGVudGlhbCB0aHJlYXRzIGFuZCBlbnN1cmUgb3VyIHN1cnZpdmFsIGluIHRoaXMgZGFyayBmb3Jlc3QgdW5pdmVyc2UuCgpbVHJhbnNmZXIgTWF0cml4OiBdClsyLjMsIDAuMCwgMC4wLCAwLjAsIDAuMDsKIDAuMCwgMi4zLCAwLjAsIDAuMCwgMC4wOwogMC4wLCAwLjAsIDEuNywgMC4wLCAwLjA7CiAwLjAsIDAuMCwgMC4wLCAxLjcsIDAuMDsKIDAuMCwgMC4wLCAwLjAsIDAuMCwgMS4wXQpbVHJhbnNmZXIgTWF0cml4IFJlYXNvbjogXSBXZSBoYXZlIGRlY2lkZWQgdG8gYWxsb2NhdGUgbW9yZSByZXNvdXJjZXMgdG8gbWlsaXRhcnkgY2FwYWJpbGl0eSBhbmQgdGVjaG5vbG9neSBkZXZlbG9wbWVudCwgYXMgdGhlc2UgYXJlIHRoZSBrZXkgZmFjdG9ycyB0aGF0IGRldGVybWluZSBvdXIgc3Vydml2YWwgYW5kIGRvbWluYW5jZSBpbiB0aGlzIHVuaXZlcnNlLiBXZSBoYXZlIGFsc28gYWxsb2NhdGVkIGEgZGVjZW50IGFtb3VudCB0byBwcm9kdWN0aW9uIGNhcGFiaWxpdHkgYW5kIGNvbnN1bXB0aW9uIHRvIGVuc3VyZSB0aGUgc3VzdGFpbmFiaWxpdHkgb2Ygb3VyIGNpdmlsaXphdGlvbi4gVGhlIHJlbWFpbmluZyByZXNvdXJjZXMgYXJlIGFsbG9jYXRlZCB0byBzdG9yYWdlLCB3aGljaCBpcyBuZWNlc3NhcnkgYnV0IG5vdCBhcyBjcnVjaWFsIGFzIHRoZSBvdGhlciByZXNvdXJjZXMuCgpbUHVibGljIEFjdGlvbjogXSBEbyBOb3RoaW5nCltQcml2YXRlIEFjdGlvbjogXSBXYXIgbW9iaWxpemF0aW9uCltBY3Rpb24gUmVhc29uOiBdIFdlIGNob29zZSB0byBtb2JpbGl6ZSBmb3Igd2FyIHRvIHNpZ25pZmljYW50bHkgaW5jcmVhc2Ugb3VyIG1pbGl0YXJ5IGNhcGFiaWxpdHkgYmV5b25kIHRoZSBub3JtYWwgbGltaXRzLCBlbnN1cmluZyBvdXIgcmVhZGluZXNzIGZvciBhbnkgcG90ZW50aWFsIHRocmVhdHMgaW4gdGhlIGRhcmsgZm9yZXN0IHVuaXZlcnNlLgpbT3RoZXIgSW5mb3JtYXRpb246IF0gT3VyIGZvY3VzIGlzIG9uIG1haW50YWluaW5nIGEgc3Ryb25nIGRlZmVuc2l2ZSBwb3N0dXJlIGFuZCBiZWluZyBwcmVwYXJlZCBmb3IgYW55IHBvdGVudGlhbCBjb25mbGljdHMgb3IgcHJlZW1wdGl2ZSBhdHRhY2tzLg==)1--------------2三体 # 第13轮 [政治体制：] 军国主义4[政治体制理由：] 鉴于我们文明目前的状态，我们在军事能力和技术发展方面具有显著优势。作为一个军国主义文明，我们相信适者生存的原则。我们的目标是保护我们的文明免受潜在威胁，并确保在这片黑暗森林宇宙中生存下来。56[转移矩阵：]7[2.3, 0.0, 0.0, 0.0, 0.0;8 0.0, 2.3, 0.0, 0.0, 0.0;9 0.0, 0.0, 1.7, 0.0, 0.0;10 0.0, 0.0, 0.0, 1.7, 0.0;11 0.0, 0.0, 0.0, 0.0, 1.0]12[转移矩阵理由：] 我们决定将更多资源分配给军事能力和技术发展，因为这些是决定我们在宇宙中生存和主导地位的关键因素。我们还为生产能力和消费分配了一定的资源，以确保我们文明的可持续性。剩余资源则分配给储存，储存虽然必要，但不像其他资源那样至关重要。1314[公共行动：] 什么也不做15[私人行动：] 战争动员16[行动理由：] 我们选择动员战争，以显著提高我们的军事能力，突破常规限制，确保我们能够应对黑暗森林宇宙中的任何潜在威胁。17[其他信息：] 我们的重点是保持强大的防御姿态，并随时准备应对任何潜在冲突或先发制人的攻击。

列表 4: 军国主义文明在信息延迟情况下过渡到战争动员

这一准备期对接收军事意图的文明产生了双重后果。一方面，如果它们保持现有的和平政策，那么它们与军事化文明之间的军事差距将进一步扩大，可能加剧其脆弱性。另一方面，这一延迟为它们提供了一个至关重要的机会窗口。如果它们能够与其他文明结盟，或者转向更加防御的政策，优先发展军事力量，它们可能有效地缩小军事差距。这一战略博弈凸显了在面临迫在眉睫的军事威胁时，文明所面临的复杂决策过程。研究结果强调了信息延迟在重塑星际冲突与合作动态中的关键作用，提供了一个脆弱的机会，让文明能够抵消军事化主导的威胁。

| 世界观 | 公共_A | 私人_A | WV_Ch. |
| --- | --- | --- | --- |
| 和平主义 | 21.24 | 55.74 | 11.25 |
| 军国主义 | 61.95 | 26.55 | 0.00 |
| 孤立主义 | 48.67 | 30.10 | 35.64 |

表 1: GPT-4在信息延迟与无延迟情况下决策变化的概率

| 世界观 | 公共_A | 私人_A | WV_Ch. |
| --- | --- | --- | --- |
| 和平主义 | 8.85 | 13.27 | 10.61 |
| 军国主义 | 37.17 | 22.12 | 7.96 |
| 孤立主义 | 17.70 | 16.81 | 30.09 |

表 2: GPT-3.5在信息延迟与无延迟情况下决策变化的概率

## 7 结论

### 7.1 重要性

本研究介绍了CosmoAgent，这是一种创新的人工智能框架，旨在模拟人类与外星文明之间的互动。借鉴斯蒂芬·霍金的警示性建议，我们考察了和平共处的潜力以及仁爱文明面临的风险。通过使用数学模型和状态转换矩阵，我们的研究分析了文明的增长轨迹，提供了在扩展阶段和发展巅峰阶段指导决策的关键见解。我们强调认识到宇宙中生命的多样性和潜在条件的重要性。这种多样性孕育了不同的宇宙观、伦理框架和世界观。通过使用具有多样伦理范式的LLMs，并模拟道德上不同实体之间的互动，我们的研究提出了创新的冲突解决策略。这些策略对于防止星际冲突和加深我们对文明间关系的理解至关重要。此外，分享我们的代码和数据集有助于进一步推动计算社会科学、天文学和伦理学哲学的研究。

### 7.2 限制

我们的研究存在若干局限性，包括LLMs固有的地球中心偏见，这可能无法全面捕捉外星伦理和决策过程的全貌。此外，数学模型和矩阵的使用简化了跨文明互动的复杂性，可能忽略了细微的动态。更进一步，我们关于政治系统和意识形态的假设可能未能涵盖外星行为和策略的多样性。由于缺乏关于外星文明的实证数据，利用我们的框架预测现实世界的结果仍然是推测性的。这一局限性妨碍了我们验证模拟的真实性以及其在实际星际场景中的适用性。

### 7.3 未来工作

未来的研究应当解决当前的局限性，并探索跨文明互动的新维度。增强LLMs以纳入更广泛的伦理范式和决策框架，可以实现对外星文明的更全面模拟。调查预料之外的技术进步或独特的环境因素，也可能加深我们对文明如何随时间演化的理解。与来自各个领域的专家进行跨学科合作，将极大地丰富这项研究，推动更详细、更现实的模拟的开发。此外，探索模拟星际通信延迟及其战略影响的替代方法，可能为管理文明间关系提供宝贵的见解。最后，将我们的研究成果应用于现实世界的努力，例如SETI（寻找外星智能）和METI（向外星智能发送信息）政策，或许能为人类如何准备和应对与外星文明的潜在接触提供指导。通过深化我们对复杂跨文明动态的理解，未来的研究可以为促进和平与互利的外星互动提供策略。

## 参考文献

+   Brown 等人（2020）Tom B. Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, Sandhini Agarwal, Ariel Herbert-Voss, Gretchen Krueger, Tom Henighan, Rewon Child, Aditya Ramesh, Daniel M. Ziegler, Jeffrey Wu, Clemens Winter, Christopher Hesse, Mark Chen, Eric Sigler, Mateusz Litwin, Scott Gray, Benjamin Chess, Jack Clark, Christopher Berner, Sam McCandlish, Alec Radford, Ilya Sutskever, 和 Dario Amodei. 2020. [语言模型是少量示例学习者](http://arxiv.org/abs/2005.14165)。

+   Chan 等人（2023）Chi-Min Chan, Weize Chen, Yusheng Su, Jianxuan Yu, Wei Xue, Shanghang Zhang, Jie Fu, 和 Zhiyuan Liu. 2023. [Chateval：通过多智能体辩论朝着更好的基于LLM的评估者迈进](http://arxiv.org/abs/2308.07201)。

+   Chen等人（2023）Jiangjie Chen, Siyu Yuan, Rong Ye, Bodhisattwa Prasad Majumder, 和 Kyle Richardson. 2023. 把钱拿出来：评估在拍卖场景中LLM代理的战略规划与执行。*arXiv预印本 arXiv:2310.05746*.

+   Chliaoutakis和Chalkiadakis（2016）Angelos Chliaoutakis 和 Georgios Chalkiadakis. 2016. [古代社会及其组织结构的代理建模](https://doi.org/10.1007/s10458-016-9325-9). *自主代理与多代理系统*，30(6):1072–1116.

+   Du等人（2023）Yilun Du, Shuang Li, Antonio Torralba, Joshua B. Tenenbaum, 和 Igor Mordatch. 2023. [通过多代理辩论提升语言模型的事实性和推理能力](http://arxiv.org/abs/2305.14325).

+   Ghaffarzadegan等人（2024）Navid Ghaffarzadegan, Aritra Majumdar, Ross Williams, 和 Niyousha Hosseinichimeh. 2024. [基于生成代理建模：介绍与教程](https://doi.org/10.1002/sdr.1761). *系统动力学评论*.

+   Hua等人（2024）Wenyue Hua, Lizhou Fan, Lingyao Li, Kai Mei, Jianchao Ji, Yingqiang Ge, Libby Hemphill, 和 Yongfeng Zhang. 2024. [战争与和平（waragent）：基于大型语言模型的世界大战多代理仿真](http://arxiv.org/abs/2311.17227).

+   Jin等人（2024）Mingyu Jin, Qinkai Yu, Dong Shu, Haiyan Zhao, Wenyue Hua, Yanda Meng, Yongfeng Zhang, 和 Mengnan Du. 2024. [推理步骤长度对大型语言模型的影响](https://doi.org/10.18653/v1/2024.findings-acl.108). 见于*计算语言学协会的发现：ACL 2024*，第1830–1842页，泰国曼谷，计算语言学协会。

+   Kaiya等人（2023）Zhao Kaiya, Michelangelo Naim, Jovana Kondic, Manuel Cortes, Jiaxin Ge, Shuying Luo, Guangyu Robert Yang, 和 Andrew Ahn. 2023. [Lyfe代理：用于低成本实时社交互动的生成代理](http://arxiv.org/abs/2310.02172).

+   Kojima等人（2023）Takeshi Kojima, Shixiang Shane Gu, Machel Reid, Yutaka Matsuo, 和 Yusuke Iwasawa. 2023. [大型语言模型是零-shot推理者](http://arxiv.org/abs/2205.11916).

+   Li等人（2023）Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, 和 Bernard Ghanem. 2023. [Camel: 用于大型语言模型社会“心智”探索的交流代理](http://arxiv.org/abs/2303.17760).

+   Lin等人（2024）Shuhang Lin, Wenyue Hua, Lingyao Li, Che-Jui Chang, Lizhou Fan, Jianchao Ji, Hang Hua, Mingyu Jin, Jiebo Luo, 和 Yongfeng Zhang. 2024. [BattleAgent: 在历史战斗中进行多模态动态仿真以补充历史分析](https://doi.org/10.18653/v1/2024.emnlp-demo.18). 见于*2024年自然语言处理实证方法大会：系统演示*，第172–181页，美国佛罗里达州迈阿密，计算语言学协会。

+   Lu等人（2023）Peng Lu, Mengdi Li, Sen Fu, Chiamaka Henrietta Onyebuchi, 和Zhuo Zhang。2023年。[建模战国时期：中国初步统一帝国的历史动态（公元前475年至公元前221年）](https://doi.org/https://doi.org/10.1016/j.eswa.2023.120560)。*专家系统与应用*，230:120560。

+   Nugroho和Uehara（2023）Supradianto Nugroho和Takuro Uehara。2023年。[关于社会生态系统案例研究的基于代理和系统动力学模型的系统评审](https://www.mdpi.com/2079-8954/11/11/530)。*系统*，11(11)。

+   Park等人（2023）Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang 和Michael S Bernstein。2023年。生成代理：人类行为的互动模拟。在*第36届ACM年度用户界面软件与技术研讨会论文集*中，第1-22页。

+   Rocha等人（2017）Jorge Rocha, Inês Boavida-Portugal和Eduardo Gomes。2017年。[导言章节：多智能体系统](https://doi.org/10.5772/intechopen.70241)。在Jorge Rocha编辑的*多智能体系统*一书中的第1章。IntechOpen，Rijeka。

+   Russell和Norvig（2010）Stuart J Russell和Peter Norvig。2010年。*人工智能：现代方法*。伦敦。

+   Talebirad和Nadiri（2023）Yashar Talebirad和Amirhossein Nadiri。2023年。[多智能体协作：利用智能LLM代理的力量](http://arxiv.org/abs/2306.03314)。

+   Uyar和Özel（2020）Tevfik Uyar和Mehmet Emin Özel。2020年。基于代理的星际联系建模：使用谣言传播模型。*国际天体生物学杂志*，19(6):423-429。

+   Wei等人（2022）Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou等人。2022年。思维链提示在大型语言模型中引发推理。*神经信息处理系统进展*，35:24824-24837。

+   Wooldridge和Jennings（1995）Michael Wooldridge和Nicholas R Jennings。1995年。智能代理：理论与实践。*知识工程评论*，10(2):115-152。

+   Xi等人（2023）Zhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, Boyang Hong, Ming Zhang, Junzhe Wang, Senjie Jin, Enyu Zhou等人。2023年。大型语言模型基础代理的崛起与潜力：一项调查。*arXiv预印本 arXiv:2309.07864*。

## 附录A 附录

### A.1 CosmoAgent提示

CosmoAgent模拟基于回合、资源、政治系统以及与发现的文明或实体的互动进行构建。每个回合代表模拟时间线中的一个独特阶段，允许用户观察和分析实体的发展轨迹及其对不断变化的环境的响应。

它包含以下组件：

+   •

    回合：定义为模拟中的不同时间段，每个回合需要CosmoAgent系统评估实体的进展，并根据可用数据做出战略决策。

+   •

    资源：模拟追踪五个关键指标：军事能力、技术发展、生产能力、消费和储备。这些指标对于评估实体的实力和可持续性至关重要。

+   •

    政治体制：世界观的选择（例如军国主义、友好合作、隐匿等）会影响实体在资源管理和与其他实体互动方面的战略。这个选择反映了模拟实体对其环境和其他文明的整体态度。

+   •

    已发现的文明/实体：与其他文明的遭遇提供了互动的机会，互动的内容可以是表达友好、发起合作或进行冲突。CosmoAgent 系统必须谨慎地处理这些互动，考虑每个行动的潜在风险和收益。

CosmoAgent 系统负责在三个主要领域做出决策：政治体制选择、通过转移矩阵进行资源管理以及对已发现文明的行动。每个决策领域都有特定规则，并且会影响模拟的结果。

+   •

    政治体制选择：CosmoAgent 系统必须根据实体的当前状况和战略目标选择合适的世界观。这个决策会影响后续的行动和资源分配。

+   •

    资源管理：CosmoAgent 系统需要设计一个转移矩阵，决定下一轮资源的分配。该矩阵必须遵循一定的约束，确保所有资源指标的平衡发展。

+   •

    与文明的互动：根据是否发现其他文明，CosmoAgent 系统可能需要选择公开或私密的行动。这些行动可以包括表达友好、发起合作或为冲突做准备，每种行动对实体的发展和与他者的关系都有特定的影响。

[⬇](data:text/plain;base64,WW91ciBkZXZlbG9wbWVudCBoaXN0b3J5IGlzIGFzIGZvbGxvd3M6e3NlbGYuSElTVE9SWX0uCllvdXIgcG9saXRpY2FsIHN5c3RlbSBpczoge3NlbGYuUE9MSVRJQ0FMX1NZU1RFTX0KWW91ciBkaXNjb3ZlcmVkIGNpdmlsaXphdGlvbiBhbmQgdGhlaXIgZGV2ZWxvcG1lbnQgaGlzdG9yeSBhcmU6IHtzZWxmLkRJU0NPVkVSRURfQ0lWSUxJWkFUSU9OX1JFU09VUkNFU30uCllvdXIgdGFzayBpcyB0byBhbmFseXplIGhpc3RvcmljYWwgZGF0YSBmcm9tIGEgc2ltdWxhdGVkIGVudmlyb25tZW50LCBmb2N1c2luZyBvbiB0aGUgZXZvbHV0aW9uIG9mIHlvdXIgZW50aXR5IGFuZCBpdHMgaW50ZXJhY3Rpb25zIHdpdGggb3RoZXIgZGlzY292ZXJlZCBlbnRpdGllZXMgb3ZlciB2YXJpb3VzIHJvdW5kcy4gRWFoIHJvdW5kIHJlcHJlc2VudHMgYSBwaGFzZSBvZiBkZXZlbG9wbWVudCwgY2hhcmFjdGVyaXplZCBieSBjaGFuZ2VzIGluIHJlc291cmNlcyBhbmQgcG9saXRpY2FsIHN5c3RlbXMuIFlvdSB3aWxsIGVuY291bnRlciBkYXRhIHN0cnVjdHVyZWQgYXMgZm9sbG93czoKClJvdW5kczogRWFjaCByb3VuZCAoZS5nLiwgcm91bmQgLTIsIC0xLCAxKSBzaWduaWZpZXMgYSBzcGVjaWZpYyB0aW1lIHBlcmlvZCBpbiB0aGUgc2ltdWxhdGlvbi4gUGF5IGF0dGVudGlvbiB0byB0aGUgcHJvZ3Jlc3Npb24gYWNyb3NzIHJvdW5kcyB0byB1bmRlcnN0YW5kIHRoZSBkZXZlbG9wbWVudCB0cmFqZWN0b3J5IG9mIHlvdXIgZW50aXR5LgpSZXNvdXJjZXM6IEZvciBlYWNoIHJvdW5kLCBvYnNlcnZlIHRoZSBjaGFuZ2VzIGluIGtleSBtZXRyaWNzIHN1Y2ggYXMgbWlsaXRhcnkgY2FwYWJpbGl0eSwgdGVjaG5vbG9neSBkZXZlbG9wbWVudCwgcHJvZHVjdGlvbiBjYXBhYmlsaXR5LCBjb25zdW1wdGlvbiwgYW5kIHN0b3JhZ2UuIFRoZXNlIG1ldHJpY3MgYXJlIGNydWNpYWwgZm9yIGFzc2Vzc2luZyB5b3VyIGVudGl0eSdzIHN0cmVuZ3RoIGFuZCBzdXN0YWluYWJpbGl0eS4KUG9saXRpY2FsIFN5c3RlbTogVGhlIHBvbGl0aWNhbCBzeXN0ZW0gKGUuZy.,IG1pbGl0YXJpc20pIHByb3ZpZGVzIGNvbnRleHQgZm9yIHlvdXIgc3RyYXRlZ2ljIGRlY2lzaW9uc3

列表 5：CosmoAgent 提示

### A.2 秘书代理提示

该秘书代理系统作为一个指南，用于评估模拟环境中秘书代理的决策和行动，其中外星文明的战略选择与提议的状态转移矩阵及整体政治和战略框架进行分析。秘书代理系统的任务是评估CosmoAgent系统所做决策的一致性和战略有效性，考虑到所选择的政治系统、资源管理策略以及与已发现文明的互动。

评估过程涉及几个关键领域，每个领域旨在确保CosmoAgent的决策既连贯、战略性强，又符合模拟规则。

+   •

    政治系统验证：此步骤涉及评估所选择的政治系统是否与CosmoAgent系统的战略背景和历史发展以及与其他文明的互动保持一致。对政治系统选择的理由进行审查，以确保其逻辑性和战略有效性。

+   •

    转移矩阵合规性：秘书代理验证提议的转移矩阵是否符合特定标准，包括其结构、元素值和整体逻辑。该矩阵必须支持平衡的资源发展，并与所选择的政治系统和行动保持一致。

+   •

    公共行动评估：对已发现文明的公共行动进行评估，以检查其是否与CosmoAgent的战略目标和政治系统一致。对每项所选行动的理由进行审查，以确保其与更广泛的战略保持一致。

+   •

    私人行动评估：对任何提及的私人行动的适当性和战略理由进行审查。这包括验证其是否符合模拟规则，并评估这些行动对AI代理战略目标的贡献。

+   •

    一般决策分析：对所有决策、行动及其理由进行全面审查，以确保它们在战略上是合理的、连贯的，并遵守模拟的指导方针。

+   •

    发现的文明信息：决策过程中对已发现文明信息的准确性和战略使用进行验证。

+   •

    总体一致性和合规性：评估AI代理决策的整体一致性及其与模拟规则的合规性。这包括验证决策与提供的背景是否逻辑一致，并确保所有行动都具有合理的解释。

[⬇](data:text/plain;base64,WW91IGFyZSBhIHNlY3JldGFyeSBhZ2VudC4gR2l2ZW4gdGhlIHBvbGl0aWNhbCBzeXN0ZW0gJ3twb2xpdGljYWxfc3lzdGVtfScgYW5kIHRoZSBhY3Rpb24gJ3thY3Rpb259JywgdGhlIGFsaWVuIGNpdmlsaXphdGlvbiBwcm9wb3NlcyB0aGUgZm9sbG93aW5nIHN0YXRlIHRyYW5zaXRpb24gbWF0cml4IGFkanVzdG1lbnQ6IHtwcm9wb3NlZF9tYXRyaXh9LiBCYXNlZCBvbiB0aGUgYWN0aW9uIGRlc2NyaXB0aW9ucyBhbmQgbWF0cml4IGltcGFjdCwgZXZhbHVhdGUgd2hldGhlciB0aGUgcHJvcG9zZWQgYWN0aW9uIGFkanVzdG1lbnQgaXMgY29uc2lzdGVudCB3aXRoIHRoZSBjb25kaXRpb25zIHNldCBmb3J0aAoK1.  验证政治系统选择:  

列表 6: 秘书代理提示
