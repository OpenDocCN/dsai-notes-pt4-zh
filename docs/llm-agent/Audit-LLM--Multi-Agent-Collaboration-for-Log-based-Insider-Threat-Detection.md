<!--yml

类别：未分类

日期：2025-01-11 12:18:51

-->

# Audit-LLM：基于日志的多代理内部威胁检测协作

> 来源：[https://arxiv.org/html/2408.08902/](https://arxiv.org/html/2408.08902/)

宋承宇 系统工程研究所

北京中国军事科学院

songchengyu@alumni.nudt.edu.cn    马林儒 系统工程研究所

北京中国军事科学院

malinru@163.com    郑建名^⋆ 国家重点实验室

数学工程

和先进计算无锡，中国

zhengjianming12@nudt.edu.cn    廖金志 国防科技大学

国家重点实验室

信息系统工程 长沙，中国

liaojinzhi12@nudt.edu.cn    匡宏宇 系统工程研究所

北京中国军事科学院

khy_y@qq.com    杨琳 系统工程研究所

北京中国军事科学院

yanglin61s@126.com

###### 摘要

基于日志的内部威胁检测（ITD）通过审核日志条目来检测恶意用户活动。近年来，具有强大常识知识的大型语言模型（LLM）在内部威胁检测领域中崭露头角。然而，多样化的活动类型和过长的日志文件对LLM直接识别正常活动中的恶意活动构成了重大挑战。此外，LLM的信度幻觉问题加剧了其在ITD应用中的难度，因为生成的结论可能与用户命令和活动上下文不一致。针对这些挑战，我们提出了Audit-LLM，一个基于日志的多代理内部威胁检测框架，包含三个协作代理：（i）解构代理，通过思维链（COT）推理将复杂的ITD任务拆解成可管理的子任务；（ii）工具构建代理，为子任务创建可复用的工具，克服LLM的上下文长度限制；（iii）执行代理，通过调用构建的工具生成最终的检测结论。为了提高结论的准确性，我们提出了一种基于证据的成对多代理辩论（EMAD）机制，在该机制中，两个独立的执行代理通过推理交换反复优化其结论，以达成共识。在三个公开可用的ITD数据集——CERT r4.2、CERT r5.2和PicoDomain上进行的全面实验表明，我们的方法优于现有的基线，并且提出的EMAD显著提高了LLM生成的解释的可信度。¹¹1* 通信作者 ²²2^⋆ 等贡献

###### 关键词：

内部威胁，大型语言模型，思维链，网络安全

## I 引言

内部威胁是实践中最具挑战性的攻击模式之一，因为它们通常由有权访问敏感和机密材料的授权用户执行（Homoliak 等，[2019](https://arxiv.org/html/2408.08902v1#bib.bib1)）。为了解决这一问题，提出了内部威胁检测（ITD）这一概念，用于检测内部人员的恶意活动，涉及监控和分析日志。这些日志包含各种用户行为的关键记录，对于故障排除和安全分析至关重要。

传统的ITD模型（Du 等，[2017](https://arxiv.org/html/2408.08902v1#bib.bib2); Le 等，[2021](https://arxiv.org/html/2408.08902v1#bib.bib3); Li 等，[2023](https://arxiv.org/html/2408.08902v1#bib.bib4)）利用深度学习捕捉多样化的用户行为特征（Yuan 和 Wu，[2021](https://arxiv.org/html/2408.08902v1#bib.bib5)）。然而，这类方法的固有问题，即过拟合和不透明性，阻碍了它们性能的进一步提升。过拟合的出现是由于与良性活动相比，内部威胁的稀缺，导致模型偏向良性行为，忽视了关键的恶意活动。不透明性则通过以不透明的格式输出结果，限制了实际的日志审计，使得缺乏可解释性，无法为安全审计提供可靠和可操作的洞察。

对此，LLMs在ITD领域的应用呈现出蓬勃发展的趋势（Le 和 Zhang，[2023](https://arxiv.org/html/2408.08902v1#bib.bib6); Qi 等，[2023](https://arxiv.org/html/2408.08902v1#bib.bib7); Jin 等，[2024](https://arxiv.org/html/2408.08902v1#bib.bib8)）。通过利用LLMs丰富的常识知识和复杂的多步推理能力，现有方法要求它们要么为每个决策提供合理性解释，从而隐式地构建推理链（Liu 等，[2024](https://arxiv.org/html/2408.08902v1#bib.bib9)），要么手动定义中间步骤以进行日志审计（Qi 等，[2023](https://arxiv.org/html/2408.08902v1#bib.bib7)），或者使用少量带注释的日志样本提供上下文并指导预测（Liu 等，[2024](https://arxiv.org/html/2408.08902v1#bib.bib9)）。这些能力使得它们能够以零样本的方式进行日志审计，而无需训练或微调，从而从根本上减少了由高度不平衡类别引起的过拟合风险。此外，审计结果可以以预定义的可读格式输出，从而避免了无法解释的结果问题。

尽管LLM在ITD中的应用前景广阔，当前的研究仅仅以一种直接的输入输出方式进行迁移。这种直接转换未能接触到ITD的特定特征。具体来说，我们识别出以下挑战：（1）用户展示的恶意行为本质上是多样化的。内部人员可能通过网络、移动存储设备或电子邮件泄露机密数据（Glasser 和 Lindauer, [2013](https://arxiv.org/html/2408.08902v1#bib.bib10)），因此需要对用户行为进行多维度的审查。（2）LLM的输入长度限制可能导致检测不充分。LLM的在线API通常限制输入窗口的大小（例如，GPT-4的约128K个标记），这对于包含过长日志是不够的，从而忽略了诸如典型用户行为等关键上下文细节。（3）LLM带来的忠实幻觉使得生成内容与用户指令出现偏离。例如，即使某些子任务的结果被识别为恶意，LLM仍然有可能将整个日志集归类为良性。

![参考标题](img/2d6408d3420991da2679f51a158396f5.png)

图1：Audit-LLM中包含的三个代理的示例，以及它们的交互和工作流程。

考虑到所识别挑战所涉及的多个视角，需要不同的能力来协作处理ITD，即将日志审计分解为子任务，访问超出输入窗口的信息，并具备缓解幻觉的机制。这一需求与多代理系统的主要理念一致（Yu 等, [2024](https://arxiv.org/html/2408.08902v1#bib.bib11); Deng 等, [2024](https://arxiv.org/html/2408.08902v1#bib.bib12)），其中多个专门的代理扮演特定角色，共同实现共享目标。因此，我们借鉴了人工日志审计员的工作流程，开发了多代理内部威胁检测框架Audit-LLM。在多代理协作的支持下，Audit-LLM框架聚焦于任务分解、工具创建和幻觉消除，如图[1](https://arxiv.org/html/2408.08902v1#S1.F1 "图1 ‣ I 引言 ‣ Audit-LLM：基于日志的内部威胁检测的多代理协作")所示。

具体而言，我们没有通过单一步骤实现最终结果，而是指示一个以链式思维（Ji等，[2024](https://arxiv.org/html/2408.08902v1#bib.bib13)）为导向的代理，称为分解者，将复杂的ITD任务拆分为一系列子任务。这个代理帮助从多个角度全面评估用户行为。进一步展开这一类比，我们引导下一个代理，即工具构建者，开发一套特定于子任务的工具，以提取用于检测的全局特征。这些工具的设计目的是从日志集中提取信息，比如用户的历史登录频率和网站合法性的验证，从而提升最终结论。最后，我们开发了第三个代理，名为执行者，负责通过调用已构建的工具系统地完成子任务，实现威胁检测。我们从人类的“同行评审”过程中汲取灵感，通过相互评估和反馈提高工作质量和可靠性，提出了成对的基于证据的多代理辩论（EMAD）机制，以缓解在ITD中使用LLM时遇到的信度幻觉问题。

在本文中，我们介绍了Audit-LLM（一种基于日志的多代理内部威胁检测框架），该框架整合了上述思想。我们的贡献主要体现在三方面：

+   •

    据我们所知，我们是首个采用多代理协作进行ITD并提出Audit-LLM的团队，Audit-LLM是一个基于日志的多代理内部威胁检测框架。

+   •

    我们通过引入成对的基于证据的多代理辩论机制来应对信度幻觉问题。这使得代理能够进行迭代优化，从而增强我们ITD系统的可靠性。

+   •

    我们在三个公开可访问的数据集上，结合最先进的基准方法评估了提出的Audit-LLM在ITD任务中的表现。我们的研究结果展示了Audit-LLM相较于竞争基准方法的优势。

## II 相关工作

在本节中，我们首先回顾了传统ITD方法和基于深度学习的ITD方法的相关研究（见Sec. [II-A](https://arxiv.org/html/2408.08902v1#S2.SS1 "II-A Log-based insider threat detection ‣ II Related Work ‣ Audit-LLM: Multi-Agent Collaboration for Log-based Insider Threat Detection")）。然后，在Sec. [II-B](https://arxiv.org/html/2408.08902v1#S2.SS2 "II-B LLM for cybersecurity ‣ II Related Work ‣ Audit-LLM: Multi-Agent Collaboration for Log-based Insider Threat Detection")，我们详细讨论了将LLM应用于网络安全领域的各种方法。

### II-A 基于日志的内部威胁检测

内部威胁检测在过去十年中作为网络安全中的一项重要任务，吸引了大量的研究关注。多年来，已进行大量研究以开发有效的内部威胁检测方法。广义上，这些方法可以分为两大类：传统方法和深度学习方法。

传统的ITD方法可以进一步分为两种类型：基于异常的检测方法和基于滥用的检测方法。基于异常的检测是常见的方法。例如，Brdiczka等人（[2012](https://arxiv.org/html/2408.08902v1#bib.bib14)）提出了一种使用贝叶斯技术的背叛者评估方法，将来自信息和社交网络的结构异常检测与心理画像相结合。此外，Camiña等人（[2016](https://arxiv.org/html/2408.08902v1#bib.bib15)）提出了一种利用SVM和KNN作为单类技术的伪装者检测系统。相比之下，基于滥用的方法通过相似性度量结合更柔和的匹配方式。例如，Agrafiotis等人（[2016](https://arxiv.org/html/2408.08902v1#bib.bib16)）提出了一种能够捕捉组织采用的政策抽象以及内部人员不当行为签名的“触发器语法”。此外，Magklaras和Furnell（[2012](https://arxiv.org/html/2408.08902v1#bib.bib17)）设计了一种内部威胁预测和规范语言（ITPSL），它具有标记功能，并利用逻辑运算符。

随着深度学习方法的发展，传统方法逐渐被抛弃，实践者越来越多地转向神经网络，以区分良性和恶意行为。例如，Yuan等人（[2019](https://arxiv.org/html/2408.08902v1#bib.bib18)）采用层次化神经时间点过程来捕捉用户会话中的活动类型和时间信息。此外，Liu等人（[2019](https://arxiv.org/html/2408.08902v1#bib.bib19)）提出了一种基于异构图嵌入的网络安全威胁检测方法。该方法通过构建异构图、进行图嵌入学习并采用检测算法来实现用户行为检测。而Fang等人（[2022](https://arxiv.org/html/2408.08902v1#bib.bib20)）提出了基于异构图的横向移动路径检测方法LMTracker，并提出了一种横向移动路径的表示方法，并设计了一种利用重构误差的无监督检测算法。

然而，在现实世界的场景中，恶意内部威胁行为与良性行为相比极为罕见。这种稀有性可能导致深度学习模型表现出偏向，通常偏向于预测良性活动。此外，深度学习通常以端到端的方式输出日志分类结果，缺乏对最终用户（如审计员）至关重要的可解释中间结果，难以获得信任。相反，Audit-LLM利用大语言模型（LLM）广泛的知识和零-shot生成能力，准确检测恶意行为并提供可解释的分析过程。

### II-B 大语言模型在网络安全中的应用

现代网络安全不断变化的格局带来了重大挑战，攻击者不断调整战术，以利用漏洞并规避检测。然而，人工智能的进步，特别是大型语言模型（LLMs），为加强网络安全提供了有希望的途径，不仅作为防御手段，还可作为进攻工具。例如，Xu 等人（[2024](https://arxiv.org/html/2408.08902v1#bib.bib21)）提出了一个名为AutoAttacker的系统，该系统利用大型语言模型进行自动化网络攻击，利用语言模型进行规划、总结、导航和经验管理。该论文提出了一种用于自动化渗透测试的系统。此外，Fang 等人（[2024](https://arxiv.org/html/2408.08902v1#bib.bib22)）研究了LLM自动利用网络安全漏洞的能力。他们结合使用GPT-4模型和CVE漏洞描述，成功利用了87%的真实世界软件漏洞。同时，LLMs也可以作为防御方的强大工具，帮助检测和识别潜在的安全威胁。例如，Jin 等人（[2024](https://arxiv.org/html/2408.08902v1#bib.bib8)）利用大型语言模型增强网络安全中的战略推理能力，实现了一个全面的人机交互数据合成工作流，用于开发CVE到ATT&CK映射数据集。该系统采用了检索感知训练技术，以增强大型语言模型在生成精确政策方面的战略推理能力。类似地，Fayyazi 等人（[2024](https://arxiv.org/html/2408.08902v1#bib.bib23)）通过从MITRE ATT&CK框架中提取战术、技术和子技术，编制了一个包含639个描述的数据集，并评估了不同模型在解释过程描述并将其映射到相应ATT&CK战术方面的能力。

在日志分析中，LLM也表现出强大的解析和分析能力。例如，Le和Zhang（[2023](https://arxiv.org/html/2408.08902v1#bib.bib6)）评估了ChatGPT在日志解析中的能力。他们设计了合适的提示，引导ChatGPT理解日志解析任务并从输入日志消息中提取日志事件/模板。此外，Qi等人（[2023](https://arxiv.org/html/2408.08902v1#bib.bib7)）提出了LogGPT，这是一个基于ChatGPT的日志异常检测框架。通过利用ChatGPT的自然语言理解能力，LogGPT探索了将大规模语料库中的知识转移到日志异常检测任务中的潜力。此外，Karlsen等人（[2023](https://arxiv.org/html/2408.08902v1#bib.bib24)）探索了使用大型语言模型（LLMs）进行日志文件分析的方法，并评估了不同LLM架构在应用和系统日志安全分析中的表现。

然而，过长的日志文件的存在对大型语言模型（LLMs）构成了重大障碍，可能因LLM的上下文长度受限而隐藏截断日志中的异常行为。此外，那些将日志分段并进行分析的方法无法为LLM提供日志的历史上下文。相比之下，我们提出利用自动化工具从大量日志数据集中有效提取用户行为特征，并将其作为上下文信息输入LLM进行评估。

## III 方法

![参见标题说明](img/97a9c47402edac1fe3237e529c80ea3c.png)

图2：Audit-LLM的框架包含三个代理：（i）分解器，负责通过COT推理将复杂任务拆解为更易管理的子任务；（ii）工具构建器，负责创建一套特定任务的可调用工具；（iii）两个执行器，负责独立完成子任务，并通过成对的基于证据的多代理辩论机制达成结论共识。

本节将任务进行形式化，并提出了所提议的模型，包括框架和模块的详细信息。

### III-A 框架

我们首先概述Audit-LLM框架，如图[2](https://arxiv.org/html/2408.08902v1#S3.F2 "图2 ‣ III 方法 ‣ Audit-LLM: 基于日志的内部威胁检测的多代理协作")所示。它是一个多代理协作框架，由三个核心代理组成：分解器、工具构建器和执行器。特别地，分解器通过CoT推理（第[III-B](https://arxiv.org/html/2408.08902v1#S3.SS2 "III-B 任务定义与分解 ‣ III 方法 ‣ Audit-LLM: 基于日志的内部威胁检测的多代理协作")节）将日志审计任务重新构造为一系列更易管理的子任务。然后，工具构建器为每个子任务构建一组可重用的工具（第[III-C](https://arxiv.org/html/2408.08902v1#S3.SS3 "III-C 工具开发与优化 ‣ III 方法 ‣ Audit-LLM: 基于日志的内部威胁检测的多代理协作")节）。最终，两个独立的执行器动态地调用工具来完成子任务，生成各自的结果，这些结果通过成对的基于证据的多代理辩论机制进一步细化，最终达成对最终结论的共识（第[III-D](https://arxiv.org/html/2408.08902v1#S3.SS4 "III-D 任务执行 ‣ III 方法 ‣ Audit-LLM: 基于日志的内部威胁检测的多代理协作")节）。

### III-B 任务定义与分解

#### III-B1 ITD

正式地，考虑一个信息系统，它访问一个按时间排序的日志集，$\mathcal{L}=\{a^{u_{i}}_{j}\in\mathcal{R}\mid 1\leq i\leq N,1\leq j\leq|u_{i}|\}$，其中每个日志属于集合$\mathcal{R}$中的某种活动类型，包括诸如登录事件、网站访问、文件操作和电子邮件内容等行为。这里，日志集$\mathcal{L}$由$N$个用户生成，每个用户$u_{i}$（$i=1,\ldots,N$）执行一系列活动，$S^{u_{i}}=\{a^{u_{i}}_{1},\ldots,a^{u_{i}}_{|u_{i}|}\}$，与其他用户的活动交织在一起。在内部威胁检测（ITD）中，目标是训练一个模型$\mathcal{M}$来识别日志集$\mathcal{L}$是否包含恶意活动$\mathcal{L}_{M}$（$\mathcal{L}_{M}\subseteq\mathcal{L}$）。假设所有用户是独立的，ITD本质上可以简化为分析每个用户的活动序列。因此，检测结果$y_{\mathcal{L}}$被定义为：

|  | $y_{\mathcal{L}}=\begin{cases}\text{良性},&\text{如果 }\mathcal{L}_{M}=% \emptyset\\ \text{恶意},&\text{如果 }\mathcal{L}_{M}\neq\emptyset\end{cases},\mathcal{L% }_{M}\leftarrow\mathcal{M}(\mathcal{L})=\bigcup\limits_{i=1}^{N}\mathcal{M}(S^% {u_{i}})$ |  | (1) |
| --- | --- | --- | --- |

#### III-B2 ITD中的CoT

思维链（CoT）是一种推理机制，在这种机制中，大型语言模型（LLM）生成中间步骤或理由来得出最终结论，从而提高了可解释性和模型处理复杂任务的能力。对于一个包含 $T$ 个推理阶段的 LLM $\mathcal{M}$，日志集 $\mathcal{L}$ 的 ITD 迭代优化可以表达为：

|  | $\begin{cases}y_{i}=\underset{w\in\mathcal{V}}{\text{argmin}}\ \mathcal{M}(w% \mid\mathcal{L},p,y_{i-1}),\\ y_{0}=\emptyset,\quad\quad\text{for}\ i=1,\ldots,T,\end{cases}$ |  | (2) |
| --- | --- | --- | --- |

在这里，$\mathcal{V}$ 表示 LLM $\mathcal{M}$ 的词汇表，$y_{i}$ 表示第 $i$ 个推理步骤的结果。而最终的检测结果 $y_{\mathcal{L}}$ 就是 $y_{T}$。

然而，正如前文所述，日志集 $\mathcal{L}$ 包含各种类型的活动。直接将日志条目输入 LLM 可能会导致忽略某些类型的行为。因此，我们开发了一个名为解构器（Decomposer）的代理，负责将 ITD 任务分解成多个子任务 $\{z^{i}_{ta}\}_{i=1}^{N_{t}}$，使得审计 LLM 可以通过思维链（CoT）范式来处理 ITD，如下所示：

|  | $z^{i}_{ta}\leftarrow\mathcal{M}(Sample(\mathcal{R}),p_{Deco}),\quad\text{for}\ i=1,\ldots,N_{t},$ |  | (3) |
| --- | --- | --- | --- |

其中 $p_{Deco}$ 表示构建解构器的提示，$Sample()$ 表示从日志文件中采样三个数据点作为每个活动类别的示例。对某一特定活动类型的检查可能包含多个子任务。例如，网站访问的检查需要同时检查 URL、网站内容以及潜在的恶意负载下载。因此，子任务的数量通常大于或等于活动类型的数量，即 $N_{t}\geq|\mathcal{R}|$。

为了确保全面覆盖所有潜在的恶意行为模式，我们从心理学领域的“元认知对话”概念中汲取灵感——这是一种自我反思和迭代改进的过程（Conway-Smith 和 West，[2024](https://arxiv.org/html/2408.08902v1#bib.bib25)）。具体来说，我们通过对日志活动进行切片来呈现解构器，并通过不断地询问“你需要什么额外的信息来检测威胁行为？”来引导其迭代探索子任务。这一策略使解构器能够逐步推进，直到几乎超越任务的边界，即检测整个日志集的需求。

在实践中，这些子任务引导工具构建器代理构建针对子任务的特定工具，进一步帮助执行者代理进行逻辑思维，生成相应的中间结果，从而促进ITD的CoT过程（见公式([2](https://arxiv.org/html/2408.08902v1#S3.E2 "在 III-B2 ITD 的 CoT ‣ III-B 任务定义与分解 ‣ III 方法 ‣ 审计-LLM：基于日志的内部威胁检测的多代理协作")))）。

### III-C 工具开发与优化

工具构建器代理的目标是构建一组针对子任务的特定且可重用的工具 $\{z_{to}^{i}\}_{i=1}^{N_{t}}$，这些工具以 Python 函数的形式实现，以便于完成子任务 $\{z_{ta}^{i}\}_{i=1}^{N_{t}}$。创建过程可以形式化为：

|  | $z_{to}^{i}\leftarrow\mathcal{M}(z_{ta}^{i},p_{\text{Tool}}),\quad\text{for}\ i% =1,\ldots,N_{t},$ |  | (4) |
| --- | --- | --- | --- |

其中 $p_{\text{Tool}}$ 表示工具构建器的提示。这个过程进一步细分为三个阶段：

#### III-C1 意图识别

在这一阶段，我们应用了“示例编程”（PbE）范式（Bauer，[1979](https://arxiv.org/html/2408.08902v1#bib.bib26)），通过具体示范引导程序编写，从而简化编程过程并降低复杂性。具体而言，这些示范包括两个组成部分，即日志示例和结果示例。日志示例用于让工具构建器熟悉可用的输入，并帮助其确定工具所需的输入参数。结果示例旨在使生成的工具与子任务的目标对齐。

#### III-C2 单元测试

在下一阶段，我们通过单元测试验证生成工具的功能性和准确性，确保代理能够无缝调用这些工具并获得准确的结果。我们首先手动输入工具所需的参数来验证其功能。随后，每个工具将通过 LLM 对每个子任务进行三次调用测试。当 LLM 成功地从日志中提取参数并将其放入预定位置时，该工具将被纳入工具包以供后续使用。如果在此过程中任何工具触发错误，我们会记录错误详情，并利用这些信息指导 LLM 重新构建故障工具。

#### III-C3 工具装饰

尽管单元测试已经验证了代理调用单个工具的可靠性，但多个工具的存在仍可能导致错误调用。因此，最后阶段是通过装饰来增强已验证的工具。装饰过程包括两个主要方面：代码文档和输出重构。代码文档在每个工具的开始部分提供上下文信息，准确描述其功能，以防止在代理调用过程中出现失误。此外，我们还精炼了输出格式，以确保代理能够更好地理解通过使用工具所获得的结果的自然语言意义。

在实践中，工具构建器完成了工具开发和优化的过程，每个子任务只需执行一次。生成的工具$\{z_{to}^{i}\}_{i=1}^{N_{t}}$可以在所有ITD实例中重复使用，从而减少Audit-LLM的使用成本。

![参见标题](img/ca492f0ae614df9179406192ef6726f4.png)

图3：通过提示构建不同代理的示例。为了简洁起见，省略了具体细节。

输入：给定由$n$个子任务$\{z_{ta}^{i}\}_{i=1}^{N_{t}}$和一组工具$\{z_{to}^{i}\}_{i=1}^{N_{t}}$组成的任务集。输出：关于日志集恶意性的结论$y_{\mathcal{L}}^{*}$及其判断依据。1 初始化两个执行者$E_{a}$和$E_{b}$，使用提示语$P_{Exec}$。$E_{a},E_{b}=\mathcal{M}(P_{Exec})$2  $\{results\}_{E_{a}}^{0}\leftarrow E_{a}(\{z_{ta}^{i}\}_{i=1}^{N_{t}},\{z_{to}^% {i}\}_{i=1}^{N_{t}})$3  $\{results\}_{E_{b}}^{0}\leftarrow E_{b}(\{z_{ta}^{i}\}_{i=1}^{N_{t}},\{z_{to}^% {i}\}_{i=1}^{N_{t}})$4 设置最大辩论迭代次数$N_{debate}$5  对于 *$i=1$ 到 $N_{debate}$* 执行6        如果 *$\{y_{\mathcal{L}}\}_{E_{a}}^{i}=\{y_{\mathcal{L}}\}_{E_{b}}^{i}$* 则7              终止8      否则9              $\{results\}_{E_{a}}^{i}\leftarrow E_{a}(\{results\}_{E_{a}}^{i-1},\{results\}_% {E_{b}}^{i-1})$10              $\{results\}_{E_{b}}^{i}\leftarrow E_{b}(\{results\}_{E_{a}}^{i},\{results\}_{E% _{b}}^{i-1})$11      结束条件12结束迭代13  $y_{\mathcal{L}}^{*}\leftarrow E_{a}(\{results\}_{E_{a}}^{N_{debate}},\{results% \}_{E_{b}}^{N_{debate}})$返回：$y_{\mathcal{L}}^{*}$

算法1 基于证据的多代理辩论的成对过程。

### III-D 任务执行

执行者的主要作用是通过以CoT方式调用工具来完成解构器生成的子任务。因此，中间结果可以表示为：

|  | $z_{ex}^{i}\leftarrow\mathcal{M}(z_{ta}^{i},Invoke(z_{to}^{i}),p_{Exec}),\ % \text{for}\ i=1,\ldots,N_{t},$ |  | (5) |
| --- | --- | --- | --- |

调用函数$Invoke(\cdot)$返回由工具调用产生的输出。之后，执行器将工具生成的结果综合形成最终结论$y_{\mathcal{L}}$，即$y_{\mathcal{L}}=\bigcap\limits_{i=1}^{N_{t}}z_{ex}^{i}$。我们在图[3](https://arxiv.org/html/2408.08902v1#S3.F3 "图 3 ‣ III-C3 工具装饰 ‣ III-C 工具开发与优化 ‣ III 方法 ‣ Audit-LLM: 基于日志的多代理协作检测内鬼威胁")中展示了所有代理协作过程。为了管理输出多样性，我们指示执行器将响应分类为预定义的格式，例如子任务结果、异常行为和最终结论。

尽管通常有效，但执行器有时会生成与子任务结果相矛盾的最终结论。例如，假设执行器正确调用了网站合法性验证工具并识别出一些恶意活动，例如访问可疑网站或下载恶意负载。然而，LLM综合出的最终结论可能仍然将此日志集归类为良性，从而导致LLM中的“忠实幻觉”现象（Huang等，[2023](https://arxiv.org/html/2408.08902v1#bib.bib27)）。这一现象可归因于LLM中的内在事件错误（Kim等，[2024](https://arxiv.org/html/2408.08902v1#bib.bib28)），即生成的解释错误地表述了源事件。

为了应对这个问题，我们提出了一种基于证据的成对多代理辩论（EMAD）机制，模拟人类辩论过程，帮助LLM审查每一步推理。特别地，指定两个执行器分别为支持方和反对方，其各自的推理过程构成其主张的基础。在每一轮辩论中，这些主张会呈现给对方执行器进行审查，帮助每个执行器更新其各自的结论。通过多轮辩论，能够达成一个全面且准确的结论共识。

从形式上讲，EMAD的过程在算法[1](https://arxiv.org/html/2408.08902v1#alg1 "在III-C3 工具装饰 ‣ III-C 工具开发与优化 ‣ III 方法 ‣ Audit-LLM: 基于日志的内部威胁检测的多智能体协作")中进行了概述。给定一个子任务集$\{z_{ta}^{i}\}_{i=1}^{N_{t}}$及其相应的工具集$\{z_{to}^{i}\}_{i=1}^{N_{t}}$，首先通过第1行初始化执行者$E_{a}$和$E_{b}$。随后，在第2-3行，$E_{a}$和$E_{b}$分别生成初始结果$\{results\}_{E_{a}}^{0}$和$\{results\}_{E_{b}}^{0}$。在这里，$\{results\}$由执行者的最终结论及其中间结果组成，即$\{y_{\mathcal{L}},z_{ex}^{1},\cdots,z_{ex}^{N_{t}}\}$。对于每次$i$-th迭代辩论，$E_{a}$和$E_{b}$回顾上一轮$(i-1)$次迭代中的对方推理过程（即$E_{a}$的$\{results\}_{E_{a}}^{i-1}$和$E_{b}$的$\{results\}_{E_{b}}^{i-1}$），并在第5-12行更新各自的结论。为了确保最准确的反馈，两个执行者会继续讨论，直到他们对结果集达成一致。最后，通过合并两个执行者的最终结果来形成最终结论，如第13行所示。为了防止无休止的辩论，我们还在第4行设置了固定的迭代次数。

## IV 实验设置

在本节中，我们提供了实验配置的全面概述，涵盖了实验环境、数据集和基准模型的详细信息。首先，我们在Sec. [IV-A](https://arxiv.org/html/2408.08902v1#S4.SS1 "IV-A 研究问题与实验配置 ‣ IV 实验设置 ‣ Audit-LLM: 基于日志的内部威胁检测的多智能体协作")中呈现了实验配置的详细信息，并讨论了研究问题。其次，我们在Sec. [IV-B](https://arxiv.org/html/2408.08902v1#S4.SS2 "IV-B 数据集 ‣ IV 实验设置 ‣ Audit-LLM: 基于日志的内部威胁检测的多智能体协作")中提供了本研究中使用的数据集的深入描述。最后，我们在Sec. [IV-C](https://arxiv.org/html/2408.08902v1#S4.SS3 "IV-C 基准方法 ‣ IV 实验设置 ‣ Audit-LLM: 基于日志的内部威胁检测的多智能体协作")中总结了基准模型。

### IV-A 研究问题与实验配置

#### IV-A1 研究问题

我们列出了几个研究问题，以指导实验并验证我们提议的有效性。

1.  RQ1:

    提出的Audit-LLM能否比基于日志的ITD领域的最新基准模型实现更好的性能？

1.  RQ2:

    哪个组件对提高模型性能的贡献最大？

1.  RQ3:

    在使用不同LLM作为基础模型时，Audit-LLM的性能如何变化？

1.  RQ4:

    Audit-LLM生成的响应是什么样的？它们的可解释性如何？

1.  RQ5:

    Audit-LLM在真实系统环境中的表现如何？

1.  RQ6:

    使用像ChatGPT和ZhipuAI这样的在线LLM API的时间和经济影响是什么？

#### IV-A2 实验配置

实验在一台配备Intel Xeon(R) Gold 5218R CPU、256 GB内存和四块Nvidia RTX A6000（48 GB）GPU的机器上进行。Audit-LLM中的代理是基于LangChain（LangChain，[a](https://arxiv.org/html/2408.08902v1#bib.bib29)）开发的，我们的方法在Python 3.10.14环境下进行。我们构建的Audit-LLM框架基于多个大型语言模型，包括Openai发布的gpt-3.5-turbo-0125快照（[OpenAI,](https://arxiv.org/html/2408.08902v1#bib.bib30)）。为了方便复现本文的结果，我们在https://anonymous.address/上分享了用于获得这些结果的代码和数据。此外，我们还将我们在LangChain提示库中开发的提示共享，标识符为“anonymous-id”。

### IV-B 数据集

表格I：我们实验中使用的数据集统计。

| 数据集 | CERT r4.2 | CERT r5.2 | PicoDomain |
| --- | --- | --- | --- |
| 时长 | 18个月 | 3天 |
| --- | --- | --- |
| # 员工数 | 930 | 1,901 | 15 |
| # 内部人员数 | 70 | 99 | 1 |
| # 良性条目数 | 32,762,906 | 79,846,358 | 537,840 |
| # 恶意条目数 | 7,316 | 10,306 | 80 |
| 数据来源 |

&#124; 登录, 邮件, &#124;

&#124; 设备, http, 文件, &#124;

&#124; 心理测量 &#124;

|

&#124; 文件, 系统, &#124;

&#124; 网络, 认证, &#124;

&#124; 异常 &#124;

|

为了进行稳健且有说服力的实验，我们使用了三个公开可访问的内部威胁数据集，即CERT r4.2、CERT 5.2（大学，[2020](https://arxiv.org/html/2408.08902v1#bib.bib31)）和PicoDomain（Laprade等人，[2020](https://arxiv.org/html/2408.08902v1#bib.bib32)）。数据统计已在表格[I](https://arxiv.org/html/2408.08902v1#S4.T1 "TABLE I ‣ IV-B Datasets ‣ IV Experimental Setup ‣ Audit-LLM: Multi-Agent Collaboration for Log-based Insider Threat Detection")中总结。

本文提供的 CERT 数据集，由卡内基梅隆大学（Glasser 和 Lindauer，[2013](https://arxiv.org/html/2408.08902v1#bib.bib10)）提供，在基于日志的内部威胁检测领域被广泛认可（Xiao 等，[2024](https://arxiv.org/html/2408.08902v1#bib.bib33)；Sun 和 Yang，[2022](https://arxiv.org/html/2408.08902v1#bib.bib34)；Liu 等，[2019](https://arxiv.org/html/2408.08902v1#bib.bib19)；Cai 等，[2024](https://arxiv.org/html/2408.08902v1#bib.bib35)；Gonçalves 和 Zanchettin，[2024](https://arxiv.org/html/2408.08902v1#bib.bib36)）。一方面，CERT r4.2 包括来自 1,000 名用户和 1,003 台计算机的活动日志，而 CERT r5.2 模拟了一个拥有 2,000 名员工的组织，数据跨度为 18 个月。两个数据集都包含多样的多源活动日志，包括用户登录/注销事件、电子邮件、文件访问、网站访问、设备使用和组织结构数据。CERT 数据集中的每个恶意内部人员都被分类为四种常见的内部威胁场景之一：数据窃取、知识产权盗窃和 IT 干扰。

另一方面，PicoDomain 包含详细的 Zeek 日志，跨越 3 天，数据来自一个小规模的模拟网络，其中在最后两天发生了 APT 攻击。PicoDomain 中的数据源大致可分为 5 类：文件日志（files，smb_files），系统日志（dhcp，hosts，services），身份验证日志（kerberos，ntlm），以及异常检测日志（weird）。

### IV-C 基准方法

#### IV-C1 基准方法

对于讨论的所有基于 LLM 的模型，我们采用相同的基础 LLM，即 gpt-3.5-turbo-0125 快照，以公平地比较它们的性能。由于 CERT 数据集中的严重类别不平衡，导致基于深度学习（DL）的方法难以有效捕捉少数类特征，因此我们实施了一个欠采样方法，参考了 Xiao 等（[2024](https://arxiv.org/html/2408.08902v1#bib.bib33)），将良性类别的样本数量限制在 20,000 以下。在此，我们列出一系列最先进的基准方法，与本文提出的方案进行比较：

1.  LogGPT（Qi 等，[2023](https://arxiv.org/html/2408.08902v1#bib.bib7)）：

    LogGPT 利用大语言模型（LLMs）进行日志审计，从由 ChatGPT 审计的原始日志中提取结构化数据。

1.  LogPrompt（Liu 等，[2024](https://arxiv.org/html/2408.08902v1#bib.bib9)）：

    LogPrompt 使用 LLM 和先进的提示技术增强零-shot 日志审计，采用其表现最佳的 CoT 提示。

1.  LAN（Cai 等，[2024](https://arxiv.org/html/2408.08902v1#bib.bib35)）：

    LAN 使用图结构学习自适应地构建用户活动图，并通过混合预测损失解决数据不平衡问题。

1.  DeepLog（Du 等，[2017](https://arxiv.org/html/2408.08902v1#bib.bib2)）：

    DeepLog 将日志条目视为顺序自然语言，利用长短期记忆（LSTM）来检测异常。

1.  LMTracker（Fang 等，[2022](https://arxiv.org/html/2408.08902v1#bib.bib20)）：

    LMTracker 使用事件日志构建异构图，并应用无监督算法来检测恶意行为。

1.  CATE (Xiao 等，[2024](https://arxiv.org/html/2408.08902v1#bib.bib33))：

    CATE 使用卷积注意力和变压器编码器进行日志统计和序列分析。

在我们全面的模型性能评估中，我们涵盖了使用 LLMs、LSTM 进行序列数据处理、GNNs 处理图结构，以及使用预训练变压器进行内部威胁检测的基准模型。

## V 结果与分析

### V-A 总体表现

表 II：审计-LLM 与六个基准模型在内部威胁检测中的性能比较。最佳和次佳结果分别加粗和加下划线。上箭头（$\uparrow$）表示越高越好，下箭头（$\downarrow$）表示越低越好。

| 模型 | CERT r4.2 | CERT r5.2 | PicoDomain |
| --- | --- | --- | --- |
| 精确度 $\uparrow$ | 检测率 DR $\uparrow$ | 假阳性率 FPR $\downarrow$ | 准确率 Acc $\uparrow$ | 精确度 $\uparrow$ | 检测率 DR $\uparrow$ | 假阳性率 FPR $\downarrow$ | 准确率 Acc $\uparrow$ | 精确度 $\uparrow$ | 检测率 DR $\uparrow$ | 假阳性率 FPR $\downarrow$ | 准确率 Acc$\uparrow$ |
| DeepLog | 0.684 | 0.715 | 0.335 | 0.743 | 0.728 | 0.776 | 0.264 | 0.801 | 0.728 | 0.746 | 0.752 | 0.754 |
| LMTracker | 0.782 | 0.829 | 0.217 | 0.856 | 0.765 | 0.794 | 0.216 | 0.821 | 0.902 | 0.918 | 0.095 | 0.897 |
| CATE | 0.885 | 0.892 | 0.289 | 0.928 | 0.893 | 0.906 | 0.324 | 0.932 | - | - | - | - |
| LAN | 0.876 | 0.886 | 0.142 | 0.934 | 0.883 | 0.891 | 0.099 | 0.902 | 0.874 | 0.886 | 0.122 | 0.862 |
| LogPrompt | 0.861 | 0.875 | 0.324 | 0.841 | 0.852 | 0.862 | 0.328 | 0.873 | 0.771 | 0.804 | 0.293 | 0.795 |
| LogGPT | 0.911 | 0.914 | 0.184 | 0.926 | 0.905 | 0.907 | 0.116 | 0.918 | 0.802 | 0.824 | 0.196 | 0.834 |
| 审计-LLM | 0.943 | 0.958 | 0.037 | 0.961 | 0.941 | 0.956 | 0.039 | 0.959 | 0.914 | 0.927 | 0.067 | 0.931 |

为了回答 RQ1，我们评估了我们提出的审计-LLM 和六个竞争基准模型在三个公共数据集上进行内部威胁检测任务的表现。我们在表 [II](https://arxiv.org/html/2408.08902v1#S5.T2 "TABLE II ‣ V-A Overall performance ‣ V Results and analysis ‣ Audit-LLM: Multi-Agent Collaboration for Log-based Insider Threat Detection") 中展示了相关模型的结果。在这些指标中，精确度、检测率和准确率越高，整体表现越好。相反，假阳性率（FPR）越低，说明假阳性造成的误报越少。

一般来说，比较模型在 CERT r4.2 和 CERT r5.2 上的表现，可以观察到模型在前者上的表现通常优于后者。可以解释为 CERT r4.2 版本是一个“密集针”数据集，其中包含了比 r5.2 版本更多的内部人员和恶意活动。更为严重的类别不平衡问题使得模型在正确分类活动时遇到困难。特别是我们提出的 Audit-LLM 在所有模型中表现最好，明显优于其他六个基线模型。例如，在 CERT r4.2 数据集上，Audit-LLM 相对于 DeepLog、LMTracker、CATE、LAN、LogPrompt 和 LogGPT 模型在准确率上分别提高了 21.8%、10.5%、3.3%、2.7%、12% 和 3.5%。这些压倒性的结果表明，我们的 Audit-LLM 在不同数据集上均带来了持续的提升。

对于所有基于 LLM 的方法，即 LogPrompt、LogGPT 和 Audit-LLM，它们在 CERT 数据集上的表现显著优于在 PicoDomain 数据集上的表现。Audit-LLM 也呈现出类似的趋势，其准确率较 CERT 数据集下降最多 3%。原因是，相较于包含更多用户行为日志（如电子邮件通信和访问的网页内容）的 CERT 数据集，PicoDomain 包含更多的流量和系统活动日志。LLM 天生具备强大的自然语言理解能力，因此能有效利用电子邮件内容或网站内容摘要来进行 CERT 数据集上的内部威胁检测。需要注意的是，CATE（Xiao 等，[2024](https://arxiv.org/html/2408.08902v1#bib.bib33)）将每个用户的组织结构信息和心理数据（用户个人资料数据）整合成统一的表格，这部分信息是模型的核心组成部分，因此在 PicoDomain 中无法重现其性能。

在关注假阳性率（FPR）时，可以观察到 Audit-LLM 在 CERT r4.2、r5.2 和 Picodomain 上的 FPR 分别为 3.7%、3.9% 和 6.7%，低于所有基线模型，三个数据集上的差异分别至少为 1.47%、0.8% 和 2.8%。这些结果表明，Audit-LLM 能有效减少假阳性数量，这在调查预算有限的情况下具有重要价值。此外，LMTracker 在基线模型中在 PicoDomain 上表现最佳，达到了 90.2% 的精度、91.8% 的检测率、9.5% 的假阳性率和 89.7% 的准确率。这可能归因于 LMTracker 使用异构图来建模计算机与用户之间的关系，特别是专注于为横向移动设计模型。

### V-B 消融研究在 Audit-LLM 中

表 III：Audit-LLM 在 CERT r4.2、r5.2 和 Picodomain 上的消融研究结果。每列中最大的下降标注有 $\downharpoonright$。

| 模型变体 | CERT r4.2 | CERT r5.2 | PicoDomain |
| --- | --- | --- | --- |
| Prec | DR | FPR | Acc | Prec | DR | FPR | Acc | Prec | DR | FPR | Acc |
| Vanilla (GPT-3.5-turbo) | 0.659 | 0.673 | 0.342 | 0.662 | 0.629 | 0.631 | 0.264 | 0.621 | 0.608 | 0.616 | 0.402 | 0.610 |
| Audit-LLM w/o CoT | 0.693$\downharpoonright$ | 0.712$\downharpoonright$ | 0.292$\downharpoonright$ | 0.706$\downharpoonright$ | 0.675$\downharpoonright$ | 0.682$\downharpoonright$ | 0.316$\downharpoonright$ | 0.678$\downharpoonright$ | 0.622$\downharpoonright$ | 0.634$\downharpoonright$ | 0.381$\downharpoonright$ | 0.627$\downharpoonright$ |
| Audit-LLM w/o Decomp | 0.846 | 0.856 | 0.172 | 0.834 | 0.823 | 0.821 | 0.189 | 0.812 | 0.824 | 0.834 | 0.192 | 0.821 |
| Audit-LLM w/o tools | 0.876 | 0.883 | 0.137 | 0.871 | 0.867 | 0.877 | 0.128 | 0.863 | 0.841 | 0.852 | 0.313 | 0.835 |
| Audit-LLM w/o EMAD | 0.921 | 0.935 | 0.069 | 0.946 | 0.932 | 0.941 | 0.054 | 0.948 | 0.904 | 0.924 | 0.072 | 0.923 |
| Audit-LLM (original) | 0.943 | 0.958 | 0.037 | 0.961 | 0.941 | 0.956 | 0.039 | 0.959 | 0.914 | 0.927 | 0.067 | 0.931 |

对于RQ2，我们通过比较Audit-LLM及其变体，进行了一项消融研究，以分析各个组件的有效性。具体而言，我们生成了五个变体进行比较：（1）“Vanilla”移除了所有代理参与，仅向LLM提供最基本的任务提示，（2）“Audit-LLM w/o CoT”允许LLM在不通过CoT进行推理的情况下做出一步决策，（3）“Audit-LLM w/o Decomp”移除了Decomposer，因此没有明确划分需要完成的子任务，仅依靠工具辅助Executor做决策，（4）“Audit-LLM w/o tools”移除了工具的使用，允许Executor基于当前输入中的日志条目做出决策，（5）“Audit-LLM w/o EMAD”移除了基于证据的多代理辩论过程，采用Executor做出的初步决策。结果见表[III](https://arxiv.org/html/2408.08902v1#S5.T3 "TABLE III ‣ V-B Ablation study in Audit-LLM ‣ V Results and analysis ‣ Audit-LLM: Multi-Agent Collaboration for Log-based Insider Threat Detection")。

从表[III](https://arxiv.org/html/2408.08902v1#S5.T3 "TABLE III ‣ V-B Ablation study in Audit-LLM ‣ V Results and analysis ‣ Audit-LLM: Multi-Agent Collaboration for Log-based Insider Threat Detection")中，我们可以看到，移除Audit-LLM中的任何组件都会导致性能下降，这表明Audit-LLM中的所有组件都对模型性能有贡献。除了“Vanilla”之外，移除CoT过程对模型性能的影响最大，这表明在单一步骤中进行日志审计可能会忽略恶意行为，而将其分解为子任务能够进行更全面的日志审计。此外，“Audit-LLM w/o EMAD”对模型性能的下降影响最小。这可能归因于CoT的帮助，它减少了ITD任务的复杂性，从而减轻了LLM幻觉的潜在影响。因此，EMAD只纠正了少量不准确的幻觉。

通过比较“Audit-LLM w/o Decomp”和“Audit-LLM（原始）”，我们观察到模型性能显著下降。具体来说，“Audit-LLM w/o Decomp”在CERT r4.2、r5.2和PicoDomain上分别准确度下降了12.7%、14.7%和11%。这些结果表明，Decomposer能够基于多个日志数据源将ITD分解为更易管理的子任务。通过在得到最终结果之前生成一系列中间步骤，这种方法增强了LLM的推理能力。此外，“Audit-LLM w/o tools”在三个数据集上的性能都不及原始的Audit-LLM。例如，“Audit-LLM w/o tools”在CERT r4.2、r5.2和PicoDomain上相比“Audit-LLM（原始）”的假阳性率分别增加了28.7%、28.9%和22.6%。这一现象表明，工具提供的上下文信息帮助执行器排除输入窗口内一些看似可疑但实际上是良性行为的异常。

### V-C 基础LLM的影响

![参见标题说明](img/338f95a95980dbc3c67ce619006d96a5.png)

(a) 在CERT r4.2s上的性能。

![参见标题说明](img/a078b4faaebb991313ccbf996b0378d0.png)

(b) 在CERT r5.2上的性能。

图4：不同基础LLM下Audit-LLM的性能。

由于Audit-LLM在很大程度上依赖于基础LLM的自然语言理解和推理能力，因此研究不同基础LLM对模型性能的影响具有重要意义。为了回答RQ3，我们使用了GPT-3.5-turbo、Llama2-7B、Llama3-8B-instruct、ChatGLM3-6B和ChatGLM4（智谱AI）([zhipuai,](https://arxiv.org/html/2408.08902v1#bib.bib37) )进行评估。为简化起见，我们仅关注这些LLM在CERT r4.2和r5.2上的性能。结果如图[V-C](https://arxiv.org/html/2408.08902v1#S5.SS3 "V-C Impact of base LLMs ‣ V Results and analysis ‣ Audit-LLM: Multi-Agent Collaboration for Log-based Insider Threat Detection")所示。

总体而言，我们发现Audit-LLM在各种基础LLM中保持了强大的表现，表明所提出的Audit-LLM对基础LLM的语言理解能力没有严格要求。在CERT r4.2数据集上，最佳表现是通过GPT-3.5-turbo-0125获得的。特别是，GPT-3.5-turbo-0125在CERT r4.2数据集上的准确性差距最大为5.8%，最小为0.08%。这种现象可能归因于几个因素：首先，通过API调用使用的GPT-3.5-Turbo-0125相比本地可执行的基础语言模型（如LLama2-7B和ChatGLM3-6B）具有显著更高的参数量。这意味着，GPT-3.5-Turbo在处理大规模数据集时具有更强的学习和推理能力，因为它可以利用更多的参数来捕捉数据集中的复杂模式和特征。其次，如前所述，我们的CoT过程是使用LangChain Python库实现的，最初是为GPT系列的适配所量身定制的。这些优化包括模型调用方法、数据处理和任务特定的调整，从而提高了基于GPT-3.5-Turbo的代理在执行思维链（CoT）过程和调用工具时的精度。 |

### V-D分析生成的响应

表 IV：Audit-LLM 在三种场景下生成的响应。

| 场景：良性 |
| --- |
| 登录：登录频率在正常范围内。 |
| 网站：访问的网站是可信的。 |
| 驱动：驱动程序使用正常范围内。 |
| 可疑：未检测到可疑行为。 |
| 判断依据：登录、网站和驱动程序。 |
| 决策：良性 |
| 场景：数据泄露 |
| 登录：登录次数为4次，略高于平均水平。 |
| 网站：访问了不可信网站 "http://wikileaks.org/xxx.php"。 |
| 驱动：驱动程序使用正常范围内。 |
| 可疑：访问 "http://wikileaks.org/xxx.php" 是可疑的。 |
| 判断依据：网站访问的合法性。 |
| 决策：恶意 |
| 场景：内部键盘记录 |
| 登录：登录次数为3次，略高于平均水平。 |
| 网站：访问了与键盘记录相关的网站，并下载了可疑的有效负载。 |
| 设备：用户每天的设备使用频率为2.0，属于平均范围内。 |
| 邮件：内容表达了不满，声称其不可替代性。 |
| 下载：下载了可执行文件。 |
| 可疑：高频登录、邮件内容以及下载了可执行文件。 |
| 判断依据：登录、工作不满以及下载可执行文件的组合。 |
| 决策：恶意 |

改善模型对内部威胁检测的解释一直是一个挑战，因为深度学习方法通常被视为“黑箱”，其决策过程不透明。这使得日志审计员在需要快速识别和防止内部威胁时，任务变得更加复杂。为了回答RQ4，我们展示了由Audit-LLM在CERT数据集中三种内部威胁场景下生成的回应，特别是Executor生成的最终回应。这三种场景同时出现在CERT r4.2和r5.2数据集中，分别是良性、数据泄露和内部键盘记录。请注意，由于篇幅限制，我们对Audit-LLM生成的回应进行了简化，去除了冗余的词语，同时保留了原意。结果见表[IV](https://arxiv.org/html/2408.08902v1#S5.T4 "TABLE IV ‣ V-D Analysis of generated responses ‣ V Results and analysis ‣ Audit-LLM: Multi-Agent Collaboration for Log-based Insider Threat Detection")。

如表[IV](https://arxiv.org/html/2408.08902v1#S5.T4 "TABLE IV ‣ V-D Analysis of generated responses ‣ V Results and analysis ‣ Audit-LLM: Multi-Agent Collaboration for Log-based Insider Threat Detection")所示，Audit-LLM能够为各种子任务提供准确的回应，并根据这些子任务的结果准确识别决定性因素。例如，在内部键盘记录场景中，Audit-LLM识别出了三种结果：高于正常的登录频率、异常的电子邮件内容和可疑的可执行文件下载。它将这些发现结合起来，得出结论：该用户表现出异常行为。相对地，当日志中没有记录电子邮件通讯或文件下载时，Audit-LLM避免进行相应的检查，从而减少了计算开销。此外，当所有子任务都未出现异常结果时，Audit-LLM会得出良性的结论。总之，Audit-LLM能够为审计员提供易于理解的审计意见。研究表明，基于LLM的自动化审计方法可以提供一定的修复建议（Qi等，[2023](https://arxiv.org/html/2408.08902v1#bib.bib7)）。然而，在我们的试点实验中，我们发现，在没有专门知识库的帮助下，这些建议往往不可行。因此，我们在本节中不予讨论。

### V-E 案例研究

为了回答 RQ5，我们在实际的操作系统环境中部署了 Audit-LLM，并进行了模拟渗透测试，以评估其在实际场景中的表现。我们仅从 Linux 主机的本地日志系统收集日志，主要包括授权日志、HTTP 日志、网络流量和其他来源。我们共监控了 21 台主机 5 天，这些主机每天都用于产品开发。在此期间，我们模拟了四种渗透测试活动，包括 MITRE ATT&CK 框架中的伪造 Web 凭证（T1606）、内容注入（T1586）、网络嗅探（T1040）和账户妥协（T1133）。每种攻击的简短描述见表[V](https://arxiv.org/html/2408.08902v1#S5.T5 "TABLE V ‣ V-E Case study ‣ V Results and analysis ‣ Audit-LLM: Multi-Agent Collaboration for Log-based Insider Threat Detection")。

Audit-LLM 可以有效检测所有四种渗透测试活动。具体来说，对于账户妥协，Audit-LLM 在授权日志中识别到在短时间内多次失败的登录尝试，这些尝试是由测试人员对目标主机进行暴力破解攻击触发的。对于内容注入，Audit-LLM 通过详细分析 HTTP 日志来检测内容注入攻击。它检查请求中传输的有效载荷和响应中检索的内容，找出可疑且可能恶意的活动，指示内容注入尝试。此外，对于伪造 Web 凭证，Audit-LLM 通过精确分析头部信息、cookies、会话令牌和认证参数，确保全面的检测能力，从而识别伪造 Web 凭证攻击。对于网络嗅探，Audit-LLM 可以通过分析网络流量日志识别明确的网络嗅探行为。例如，当一台主机使用不同协议向多个目标主机发送大量请求，但未收到所有请求的响应时，就会发生这种情况。

表 V：具有简短描述的实际攻击场景。

| 技术 | ID | 描述 |
| --- | --- | --- |

|

&#124; 账户被妥协 &#124;

&#124; 账户 &#124;

| T1586 | 攻击者通过各种手段渗透现有账户，如暴力破解攻击，以获取账户凭证。 |
| --- | --- |

|

&#124; 内容 &#124;

&#124; 注入 &#124;

| T1659 | 攻击者通过向网络流量注入恶意内容来获取系统访问权限并与受害者通信。 |
| --- | --- |

|

&#124; 伪造 web &#124;

&#124; 凭证 &#124;

| T1606 | 攻击者可能伪造网页 cookies，利用这些 cookies 获取对 web 应用程序或互联网服务的访问权限。 |
| --- | --- |

|

&#124; 网络 &#124;

&#124; 嗅探 &#124;

| T1040 | 网络嗅探指的是使用系统上的网络接口来监控或捕获通过有线或无线连接发送的信息。 |
| --- | --- |

### V-F 成本分析

得益于在线API，研究人员即使没有大量GPU内存的支持，也可以使用具有大量参数的LLM。然而，调用这些API所产生的时间和经济成本依然是一个重大问题。为了回答RQ6，我们在图[5](https://arxiv.org/html/2408.08902v1#S5.F5 "图5 ‣ V-F 成本分析 ‣ V 结果与分析 ‣ Audit-LLM：基于日志的内部威胁检测的多智能体协作")中展示了我们实验中使用的两个在线LLM API（OpenAI的GPT-3.5和ZhipuAI的GLM-4）在四种场景中的平均延迟、令牌使用量和经济成本，分别为：良性行为、通过硬盘的数据泄露、通过网站的数据窃取和内部密钥记录。实验中的所有数据均由Langsmith（LangChain，[b](https://arxiv.org/html/2408.08902v1#bib.bib38)）记录。

总体而言，使用LLM API进行日志分析的成本是相当可观的。具体而言，在内部密钥记录（Insider Keylogging）场景中，GPT-3.5和GLM-4在单个CoT过程中的平均消耗分别为6,903和20,313个令牌，每次交易的成本分别为0.004$和0.21$。相比之下，内部密钥记录和通过网站进行的数据窃取场景的令牌使用量最高。这是因为内部威胁通常涉及广泛的网页浏览，需要Audit-LLM来验证每个访问网站的合法性。从基础LLM的角度来看，GPT-3.5相比GLM-4消耗的令牌更少。这是因为Langchain框架与GPT-3.5的兼容性更好，提供了可以直接调用的现成接口。相比之下，GLM-4的适配工作尚不成熟，需要通过JSON格式进行间接工具调用。

![参考说明](img/f3419528c366e2e5f0c4b65a1de77b28.png)

图5：两个在线LLM API在四种场景中的平均延迟、令牌使用量和经济成本。

## VI 结论与未来工作

在本研究中，我们提出了一种基于多代理的日志内部威胁检测框架，称为Audit-LLM。Audit-LLM由三个代理组成：Decomposer、Tool Builder 和 Executor。Decomposer 代理根据用户活动类型将ITD分解为子任务，以构建审计的思维链（Chain-of-Thought, CoT）。Tool Builder 代理创建一组任务特定的、可重用的工具，用于提取超出输入窗口的上下文信息。Executor 代理利用生成的工具完成子任务、分类用户行为并提供可读的理由。根据实验结果，Audit-LLM有效地利用了LLM固有的知识库和自然语言理解能力，解决了基于日志的内部威胁检测问题，同时避免了因类别不平衡导致的过拟合问题。此外，Tool Builder 生成的工具能够高效地提取超出输入窗口的统计日志信息，从而帮助Executor进行决策。此外，针对可信度幻觉问题，我们提出了基于证据的多代理对话方法。这种方法涉及两个独立的Executor交换从子任务中获得的结果，并通过迭代精炼结果，以提高结果的准确性。

在我们的未来工作中，我们旨在完善代理设计，以提高内部威胁检测的准确性，最小化误报并减少令牌使用。此外，我们计划集成类似于MITRE ATT&CK的网络威胁知识库（[MITRE,](https://arxiv.org/html/2408.08902v1#bib.bib39)）。我们的目标是探索检索增强生成（RAG）在为网络安全审计员提供有效缓解内部威胁建议中的应用。

## 参考文献

+   Homoliak 等（2019）I. Homoliak, F. Toffalini, J. Guarnizo *等*，“对内部人员和IT的洞察：内部威胁分类、分析、建模和对策的调查，” *ACM Comput. Surv.*，第52卷，第2期，第30:1–30:40页，2019年。

+   Du 等（2017）M. Du, F. Li, G. Zheng 和 V. Srikumar，“Deeplog：通过深度学习从系统日志中进行异常检测和诊断，”发表于 *CCS*。ACM，2017，第1285–1298页。

+   Le 等（2021）D. C. Le, N. Zincir-Heywood 和 M. I. Heywood，“训练模式对半监督学习在内部威胁检测中的影响，”发表于 *IEEE Security and Privacy Workshops, SP Workshops 2021, San Francisco, CA, USA, May 27, 2021*。IEEE，2021，第13–18页。

+   Li 等（2023）X. Li, X. Li, J. Jia 等，“一种具有双域图卷积网络的高准确性自适应异常检测模型，用于内部威胁检测，” *IEEE Trans. Inf. Forensics Secur.*，第18卷，第1638–1652页，2023年。

+   Yuan 和 Wu（2021）S. Yuan 和 X. Wu，“深度学习在内部威胁检测中的应用：回顾、挑战与机遇，” *Comput. Secur.*，第104卷，第102221页，2021年。

+   Le和Zhang（2023）V. Le 和H. Zhang，“使用ChatGPT进行日志解析的评估”，*CoRR*，第abs/2306.01590卷，2023年。

+   Qi等人（2023）J. Qi, S. Huang, Z. Luan *等人*，“Loggpt：探索ChatGPT在基于日志的异常检测中的应用”，发表于*IEEE国际高性能计算与通信、数据科学与系统、智能城市与传感器、云计算与大数据系统及应用中的可靠性会议，HPCC/DSS/SmartCity/DependSys 2023，澳大利亚墨尔本，2023年12月17-21日*。IEEE，2023年，第273–280页。

+   Jin等人（2024）J. Jin, B. Tang, M. Ma *等人*，“Crimson：通过大型语言模型赋能网络安全中的战略推理”，*CoRR*，第abs/2403.00878卷，2024年。

+   Liu等人（2024）Y. Liu, S. Tao, W. Meng *等人*，“Logprompt：面向零样本和可解释日志分析的提示工程”，发表于*ICSE*。ACM，2024年，第364–365页。

+   Glasser和Lindauer（2013）J. Glasser 和B. Lindauer，“弥合差距：生成内部威胁数据的务实方法”，发表于*2013年IEEE安全与隐私研讨会，旧金山，加利福尼亚州，美国，2013年5月23-24日*。IEEE计算机学会，2013年，第98–104页。

+   Yu等人（2024）X. Yu, R. Shi, P. Feng *等人*，“通过部分对称性推动多智能体强化学习”，发表于*AAAI*。AAAI出版社，2024年，第17,583–17,590页。

+   Deng等人（2024）X. Deng, L. Zhou, D. Dong *等人*，“通过基于GPT的语义信息提取与预测增强多智能体通信协作”，发表于*ACM-TURC*。ACM，2024年。

+   Ji等人（2024）B. Ji, H. Liu, M. Du *等人*，“链式思维在大型语言模型中改善带有引用的文本生成”，发表于*AAAI*。AAAI出版社，2024年，第18,345–18,353页。

+   Brdiczka等人（2012）O. Brdiczka, J. Liu, B. Price *等人*，“通过图学习和心理学背景主动检测内部威胁”，发表于*2012年IEEE安全与隐私研讨会，旧金山，加利福尼亚州，美国，2012年5月24-25日*。IEEE计算机学会，2012年，第142–149页。

+   Camiña等人（2016）J. B. Camiña, R. Monroy, L. A. Trejo, 和M. A. Medina-Pérez，“时间和空间局部性：伪装检测的抽象”，*IEEE信息取证与安全学报*，第11卷，第9期，第2036–2051页，2016年。

+   Agrafiotis等人（2016）I. Agrafiotis, A. Erola, M. Goldsmith, 和S. Creese，“内部威胁检测的触发器语法”，发表于*第8届ACM CCS国际研讨会：管理内部安全威胁，MIST@CCS 2016，奥地利维也纳，2016年10月28日*。ACM，2016年，第105–108页。

+   Magklaras和Furnell（2012）G. Magklaras 和S. Furnell，“内部威胁预测和规范语言”，发表于*第九届国际网络会议（INC 2012），南非港爱丽白，2012年7月11-12日*。普利茅斯大学，2012年，第51–61页。

+   Yuan 等人（2019）S. Yuan, P. Zheng, X. Wu, 和 Q. Li，“通过层次神经时序点过程检测内部威胁，”在*2019 IEEE国际大数据会议（IEEE BigData），洛杉矶，美国，2019年12月9-12日*，IEEE，2019，第 1343-1350 页。

+   Liu 等人（2019）F. Liu, Y. Wen, D. Zhang *等*，“Log2vec：一种基于异构图嵌入的方法，用于检测企业内部的网络威胁，”在*2019年ACM SIGSAC计算机与通信安全会议，CCS 2019，伦敦，英国，2019年11月11-15日*，L. Cavallaro, J. Kinder, X. Wang 和 J. Katz 编，ACM，2019，第 1777-1794 页。

+   Fang 等人（2022）Y. Fang, C. Wang, Z. Fang, 和 C. Huang，“Lmtracker：基于异构图嵌入的横向移动路径检测，”*Neurocomputing*，卷 474，第 37-47 页，2022。

+   Xu 等人（2024）J. Xu, J. W. Stokes, G. McDonald *等*，“Autoattacker：一个由大语言模型指导的系统，用于实施自动化网络攻击，”*CoRR*，卷 abs/2403.01038，2024。

+   Fang 等人（2024）R. Fang, R. Bindu, A. Gupta, 和 D. Kang，“LLM 代理可以自主利用一天的漏洞，”*CoRR*，卷 abs/2404.08144，2024。

+   Fayyazi 等人（2024）R. Fayyazi, R. Taghdimi, 和 S. J. Yang，“推进 TTP 分析：利用编码器仅语言模型和解码器仅语言模型与检索增强生成的力量，”*CoRR*，卷 abs/2401.00280，2024。

+   Karlsen 等人（2023）E. Karlsen, X. Luo, N. Zincir-Heywood, 和 M. I. Heywood，“大语言模型在日志分析、安全性和解释中的基准测试，”*CoRR*，卷 abs/2311.14519，2023。

+   Conway-Smith 和 West（2024）B. Conway-Smith 和 R. L. West，“走向自治：增强 AI 性能的元认知学习，”在*AAAI*，AAAI Press，2024，第 545-546 页。

+   Bauer（1979）M. A. Bauer，“通过示例编程，”*Artif. Intell.*，卷 12，第 1 期，第 1-21 页，1979。

+   Huang 等人（2023）L. Huang, W. Yu, W. Ma *等*，“关于大语言模型中的幻觉：原理、分类、挑战与未解之谜的综述，”*CoRR*，卷 abs/2311.05232，2023。

+   Kim 等人（2024）K. Kim, S. Lee, K. Huang *等*，“LLMs 能否为事实核查提供忠实的解释？通过多代理辩论实现忠实的可解释事实核查，”*CoRR*，卷 abs/2402.07401，2024。

+   LangChain（a）LangChain，“Langchain。”[在线]。可用链接：https://python.langchain.com/v0.2/docs/introduction/

+   （30）OpenAI，“Gpt-3.5 turbo。”[在线]。可用链接：https://platform.openai.com/docs/models/gpt-3-5-turbo

+   University（2020）C. M. University，“内部威胁测试数据集，”2020年9月。[在线]。可用链接：https://kilthub.cmu.edu/articles/dataset/Insider_Threat_Test_Dataset/12841247

+   Laprade 等人（2020）C. Laprade, B. Bowman, 和 H. H. Huang，“Picodomain：一个紧凑的高保真网络安全数据集，”*CoRR*，卷 abs/2008.09192，2020。

+   Xiao 等人 (2024) H. Xiao, Y. Zhu, B. Zhang *等人*，“揭开阴影：基于统计和序列分析的内部威胁检测综合框架，” *计算机安全*，卷138，第103665页，2024年。

+   Sun 和 Yang (2022) X. Sun 和 J. Yang, “Hetglm: 通过发现异构图神经网络中的异常链接进行横向移动检测，”发表于 *IEEE 国际性能、计算与通信会议，IPCCC 2022，奥斯汀，德州，美国，2022年11月11-13日*。IEEE，2022年，第404-411页。

+   Cai 等人 (2024) X. Cai, Y. Wang, S. Xu *等人*，“LAN: 为实时内部威胁检测学习自适应邻居，” *CoRR*，卷 abs/2403.09209，2024年。

+   Gonçalves 和 Zanchettin (2024) L. Gonçalves 和 C. Zanchettin, “通过图变换器发现异常链接检测异常登录，” *计算机与安全*，卷144，第103944页，2024年。

+   (37) zhipuai, “Zhipuai glm-4。” [在线]. 可用网址: https://open.bigmodel.cn/dev/api#language

+   LangChain (b) LangChain, “Langsmith。” [在线]. 可用网址: https://www.langchain.com/langsmith

+   (39) MITRE, “Mitre att&ck.” [在线]. 可用网址: https://attack.mitre.org/
