<!--yml

分类：未分类

日期：2025-01-11 11:55:58

-->

# 导航风险：基于大语言模型（LLM）的智能体在安全性、隐私性和伦理方面的威胁调查

> 来源：[https://arxiv.org/html/2411.09523/](https://arxiv.org/html/2411.09523/)

Yuyou Gan [ganyuyou@zju.edu.cn](mailto:ganyuyou@zju.edu.cn) [0000-0002-8814-3940](https://orcid.org/0000-0002-8814-3940 "ORCID标识符")，Yong Yang [yangyong2022@zju.edu.cn](mailto:yangyong2022@zju.edu.cn) [0000-0003-3526-560X](https://orcid.org/0000-0003-3526-560X "ORCID标识符")，Zhe Ma [mz.rs@zju.edu.cn](mailto:mz.rs@zju.edu.cn)，Ping He [gnip@zju.edu.cn](mailto:gnip@zju.edu.cn)，Rui Zeng [0009-0007-4207-7283](https://orcid.org/0009-0007-4207-7283 "ORCID标识符") [ruizeng24@zju.edu.cn](mailto:ruizeng24@zju.edu.cn)，Yiming Wang [ym˙wang@zju.edu.cn](mailto:ym%CB%99wang@zju.edu.cn)，Qingming Li [liqm@zju.edu.cn](mailto:liqm@zju.edu.cn)，Chunyi Zhou [zhouchunyi@zju.edu.cn](mailto:zhouchunyi@zju.edu.cn) 浙江大学杭州浙江中国，Songze Li [songzeli@seu.edu.cn](mailto:songzeli@seu.edu.cn) 东南大学南京江苏中国，Ting Wang [twang@cs.stonybrook.edu](mailto:twang@cs.stonybrook.edu) 斯托尼布鲁克大学斯托尼布鲁克纽约美国，Yunjun Gao [gaoyj@zju.edu.cn](mailto:gaoyj@zju.edu.cn)，Yingcai Wu [ycwu@zju.edu.cn](mailto:ycwu@zju.edu.cn) 和 Shouling Ji [sji@zju.edu.cn](mailto:sji@zju.edu.cn) 浙江大学杭州浙江中国（2018年；2024年11月5日）

###### 摘要。

随着大语言模型（LLMs）的持续发展，基于变换器的模型在众多自然语言处理（NLP）任务中取得了突破性进展，催生了一系列以LLM为控制中心的智能体。尽管LLMs在各种任务中取得了成功，但它们面临着许多安全性和隐私性威胁，特别是在智能体场景中，这些威胁变得更加严重。为了提高基于LLM的应用程序的可靠性，已经出现了一系列从不同角度评估和缓解这些风险的研究。

为了帮助研究人员全面了解各种风险，本调查收集并分析了这些智能体所面临的不同威胁。为了应对以往分类法在处理跨模块和跨阶段威胁时的挑战，我们提出了一种基于来源和影响的新型分类框架。此外，我们还识别了基于LLM的智能体的六个关键特征，并基于这些特征总结了当前的研究进展，并分析了它们的局限性。随后，我们选择了四个具有代表性的智能体作为案例研究，分析了它们在实际应用中可能面临的风险。最后，基于前述分析，我们分别从数据、方法论和政策三个角度提出了未来的研究方向。

基于LLM的智能体、安全性、隐私性、伦理^†^†版权：acm授权^†^†期刊年份：2018^†^†DOI：XXXXXXX.XXXXXXX^†^†ISBN：978-1-4503-XXXX-X/18/06^†^†CCS：安全性与隐私

## 1\. 引言

随着语言模型（LMs）不断发展，基于 Transformer 架构的LLMs（Vaswani等，[2017](https://arxiv.org/html/2411.09523v1#bib.bib260)）在自然语言处理（NLP）的各个领域取得了显著的成功（Devlin，[2018](https://arxiv.org/html/2411.09523v1#bib.bib63)；Radford等，[2019](https://arxiv.org/html/2411.09523v1#bib.bib215)）。大量的参数和广泛的训练数据赋予了LLMs在文本生成（Touvron等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib254)；Brown等，[2020](https://arxiv.org/html/2411.09523v1#bib.bib25)）、代码辅助（Feng等，[2020](https://arxiv.org/html/2411.09523v1#bib.bib80)；Wang等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib273)）、逻辑推理（Wei等，[2022](https://arxiv.org/html/2411.09523v1#bib.bib278)；Yao等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib298)）等任务中强大的能力。由于其强大的理解能力，越来越多的研究将LLMs定位为AI代理的核心决策中心（Nakano等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib190)；Shen等，[2024c](https://arxiv.org/html/2411.09523v1#bib.bib236)），这些AI代理是复杂的软件程序，旨在代表用户或其他系统自主执行任务。与早期基于启发式算法或强化学习的AI代理（Mnih等，[2015](https://arxiv.org/html/2411.09523v1#bib.bib186)；Lillicrap等，[2015](https://arxiv.org/html/2411.09523v1#bib.bib155)）相比，基于LLM的代理能够与用户进行交流，使得它们更易理解和接受。此外，它们庞大的基础知识使得它们能够像人类一样进行思考（理解 + 计划）。这些特点促成了它们的普及，使它们成为AI服务于各个实际领域的有前景方向（Zhang等，[2024e](https://arxiv.org/html/2411.09523v1#bib.bib318)；Wang等，[2023d](https://arxiv.org/html/2411.09523v1#bib.bib264)；Zeng等，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib311)）。例如，Supertools（Supertools，[[n. d.]](https://arxiv.org/html/2411.09523v1#bib.bib248)）是一个由LLMs赋能的流行应用程序的综合集合。

尽管LLM取得了显著成功，但它们也面临着由于内部漏洞或外部攻击带来的安全和隐私威胁。基于LLM的智能体增加了一些组件和功能，这使得这些风险更加威胁。例如，LLM面临越狱攻击（沈等，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib234)；刘等，[2023e](https://arxiv.org/html/2411.09523v1#bib.bib169)），指的是绕过其内建安全机制的过程（白等，[2022](https://arxiv.org/html/2411.09523v1#bib.bib18)；欧阳等，[2022](https://arxiv.org/html/2411.09523v1#bib.bib199)）。在基于LLM的智能体中，LLM需要处理多轮对话和多个信息来源，使得越狱攻击更加复杂且难以防御（Anil等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib13)；程等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib48)）。为了揭示基于LLM的智能体的漏洞，并使其更加安全可靠，越来越多的研究集中于从不同视角分析威胁。

为了帮助研究人员更好地理解基于LLM的智能体并开展未来的研究工作，已有两篇综述（邓等，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib62)；崔等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib54)）总结了基于LLM的智能体的安全风险。他们基于智能体的组成（称为模块）或操作阶段（称为阶段）对安全风险进行了分类，如下所示。(i) 模块视角。崔等（崔等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib54)）识别了LLM系统中的四个关键模块，即输入模块、语言模型模块、工具链模块和输出模块。他们基于这四个模块总结了LLM智能体的风险。(ii) 阶段视角。邓等（邓等，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib62)）识别了LLM AI智能体中的四个关键知识空白，即感知阶段、内部执行阶段、环境中的行动阶段以及与不可信外部实体的交互阶段。他们基于这四个阶段总结了LLM智能体的风险。这两种分类方法清晰地突出了LLM智能体面临的攻击来源。然而，它们难以准确指出跨模块和跨阶段的威胁。例如，隐私泄露是由于语言模型模块中的内存问题引起的，但它发生在输出模块中。同样，目标劫持不仅可能在感知阶段发生，还可能在与外部数据交互阶段发生（Abdelnabi等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib6)）。这些跨模块和跨阶段的威胁被不准确地归因于单一模块或阶段。

![参考说明](img/17a50dbd88f1d66569ed8a3b9d8d75d6.png)

图1\. 我们的LLM基础代理风险分类的整体框架。

我们的设计。为了更准确和全面地分类LLM基础代理的各种威胁，我们通过将威胁映射到一个基于其来源和类型的二进制表格中，提出了一种新颖的分类法。(i) 对于威胁的来源，我们考虑了LLM基础代理的操作性质：LLM根据来自多个来源的输入做出决策，如图[1](https://arxiv.org/html/2411.09523v1#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")左图所示。作为一种概率模型，LLM的输出决策分布由输入和模型本身共同决定。因此，我们将LLM基础代理的威胁归因于输入、模型或两者的组合。与通过模块或阶段分类攻击不同，我们的来源分类更接近威胁的本质。例如，目标劫持（Qiang et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib214); Huang et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib110); Liu et al., [2024c](https://arxiv.org/html/2411.09523v1#bib.bib170)）可能源自用户输入或外部数据库，但两者从根本上作为输入作用于模型，目的是劫持目标。(ii) 对于威胁的类型，我们将威胁分为三类：安全/安全性、隐私和伦理。具体来说，如果威胁导致模型产生不正确的输出（包括事实不准确或与开发者或用户需求不符的错误），则将其归类为安全/安全性问题，如对抗样本（Yin et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib300)）。如果威胁导致隐私泄露，则将其归类为隐私问题，如提示泄露攻击（Yu et al., [2023d](https://arxiv.org/html/2411.09523v1#bib.bib304); Zhang et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib324)）。如果威胁没有产生“不正确”的输出，但引发了如不公平等问题，则归属于伦理问题，如偏见（Gallegos et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib82)）。

我们收集了来自顶级会议和高引用量的 arXiv 论文。顶级会议包括但不限于：IEEE S&P、ACM CCS、USENIX Security、NDSS、ACL、CVPR、NIPS、ICML 和 ICLR。我们使用图 [1](https://arxiv.org/html/2411.09523v1#S1.F1 "图 1 ‣ 1\. 引言 ‣ 导航风险：基于大语言模型的代理中的安全性、隐私性和伦理性威胁调查") 中的分类法对不同类型的威胁进行分类。对于来源于输入的威胁，我们称其为*问题输入*。在这种情况下，攻击者不能修改模型，但可以设计输入来引发恶意输出或行为，例如对抗样本。对于来源于模型内部的威胁，我们称其为*模型缺陷*。在这种情况下，输入总是无害的，但模型本身的缺陷会导致恶意输出或行为，例如幻觉问题。对于同时来自模型缺陷和精心设计输入的威胁，我们称其为*联合威胁*。在这种情况下，攻击者故意设计输入以利用模型的漏洞，例如后门攻击。

![参见说明](img/8ab31401f1595f424f7fbff08db60bf7.png)

图 2\. 基于大语言模型的代理的整体框架。

我们的贡献。与近期关于基于大语言模型的代理安全风险的调查（Deng et al., [2024a](https://arxiv.org/html/2411.09523v1#bib.bib62); Cui et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib54)）相比，我们的工作有三个主要优势。

(i) 一种新颖的威胁分类法。我们提出了一种新颖的分类法，基于威胁的来源和影响将其映射到一个二进制表中，这可以全面涵盖现有的威胁，并扩展到未来的威胁，包括跨模块和跨阶段的威胁。

(ii) 多模态大语言模型（MLLMs）的详细分析。许多任务要求代理处理来自多个模态的输入（例如，城市导航系统（Zeng et al., [2024a](https://arxiv.org/html/2411.09523v1#bib.bib311)）），这促使了基于这些模型的各种 MLLMs 和代理的出现（Zeng et al., [2024a](https://arxiv.org/html/2411.09523v1#bib.bib311); Yang et al., [2023a](https://arxiv.org/html/2411.09523v1#bib.bib296)）。以往的调查主要集中在文本模态，缺乏对多模态模型的分析。我们涵盖了 LLMs 和 MLLMs，特别强调在威胁背景下分析多模态任务带来的新挑战和威胁。

(iii) 四个精心挑选的案例研究。之前的调查基于基于LLM的智能体（或系统）的一般框架分析了风险。然而，实际的智能体不一定包含框架中的所有模块，这些模块的设计也可能被定制（Dong et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib67)）。更重要的是，它们所面临的场景有显著的差异，导致威胁的程度和原因各不相同。为了帮助读者更好地理解智能体面临的实际威胁，我们在[6](https://arxiv.org/html/2411.09523v1#S6 "6\. Case Study ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")节中展示了四个不同智能体的案例研究，代表了四种经典情境。

本文结构如下。第[2](https://arxiv.org/html/2411.09523v1#S2 "2\. LLM-based Agent ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")节介绍了基于LLM的智能体的一般框架，并识别了框架中的六个关键特征。[3](https://arxiv.org/html/2411.09523v1#S3 "3\. Risks from Problematic Inputs ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")、[4](https://arxiv.org/html/2411.09523v1#S4 "4\. Risks from Model Flaws ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")和[5](https://arxiv.org/html/2411.09523v1#S5 "5\. Risks from Input-Model Interaction ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")节分别描述了来自问题输入、模型缺陷和输入-模型交互的风险。[6](https://arxiv.org/html/2411.09523v1#S6 "6\. Case Study ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")节提供了四个精心挑选的案例研究。[7](https://arxiv.org/html/2411.09523v1#S7 "7\. Future Direction ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")节给出了该领域发展的未来方向。

## 2\. 基于LLM的智能体

AI智能体被认为是一个有前景的研究方向，它利用AI技术自主执行特定任务并做出决策。在之前的研究中，AI智能体通常通过启发式策略或强化学习（Lillicrap等， [2015](https://arxiv.org/html/2411.09523v1#bib.bib155)）（Mnih等， [2015](https://arxiv.org/html/2411.09523v1#bib.bib186)）（Schulman等， [2017](https://arxiv.org/html/2411.09523v1#bib.bib227)）（Wilkins， [2014](https://arxiv.org/html/2411.09523v1#bib.bib280)）在特定场景（如玩游戏）中取得了良好的结果。近年来，由于LLM（如ChatGPT）在各种NLP任务中表现出色，吸引了学术界和工业界的广泛关注（Brown等， [2020](https://arxiv.org/html/2411.09523v1#bib.bib25)）（Zhang等， [2022](https://arxiv.org/html/2411.09523v1#bib.bib322)）（Vaswani等， [2017](https://arxiv.org/html/2411.09523v1#bib.bib260)）。因此，越来越多的研究开始探讨将LLM用作AI智能体的决策中心（Nakano等， [2021](https://arxiv.org/html/2411.09523v1#bib.bib190)）（Ahn等， [2022](https://arxiv.org/html/2411.09523v1#bib.bib10)）（Park等， [2023](https://arxiv.org/html/2411.09523v1#bib.bib201)）。随着LLM的发展，LLM能够处理更多的模态和任务（Shen等， [2024c](https://arxiv.org/html/2411.09523v1#bib.bib236)）。

基于LLM的智能体框架。在我们的研究中，我们考虑了一个全面的基于LLM的智能体框架，该框架涵盖了主流LLM智能体的模块和运行模式，如图[2](https://arxiv.org/html/2411.09523v1#S1.F2 "图2 ‣ 1\. 介绍 ‣ 导航风险：LLM智能体中的安全、隐私和伦理威胁调研")所示。该框架包含以下四个模块。

(i) 输入模块（IM）。IM接收用户的输入并按以下方式进行预处理。首先，IM将输入格式化为特定的分布（例如，标准化输入图像）或特定格式（例如，特定语言（Sharma等， [2024](https://arxiv.org/html/2411.09523v1#bib.bib230)））。其次，IM实现有害信息检测（Roy等， [2023](https://arxiv.org/html/2411.09523v1#bib.bib222)）（Yang等， [2024b](https://arxiv.org/html/2411.09523v1#bib.bib294)）或净化（Sharma等， [2024](https://arxiv.org/html/2411.09523v1#bib.bib230)）。第三，许多基于LLM的智能体在输入之前添加系统提示（Nakano等， [2021](https://arxiv.org/html/2411.09523v1#bib.bib190)）（Shen等， [2024c](https://arxiv.org/html/2411.09523v1#bib.bib236)）。

(ii) 决策模块（DM）。DM 理解并分析用户的查询，提供方案并生成最终的响应。许多智能体的决策模块仅包含一个 LLM。它们利用 LLM 来进行理解、规划和反馈（Nakano 等， [2021](https://arxiv.org/html/2411.09523v1#bib.bib190)）（Shen 等， [2024c](https://arxiv.org/html/2411.09523v1#bib.bib236)），或者使用 LLM 来进行理解和反馈，由另一个非 LLM 规划器负责规划（Liu 等， [2023a](https://arxiv.org/html/2411.09523v1#bib.bib160)）（Dagan 等， [2023](https://arxiv.org/html/2411.09523v1#bib.bib55)）。随着任务变得更加复杂，许多智能体会使用多个 LLM 来完成上述任务。例如，VOYAGER（Wang 等， [2023d](https://arxiv.org/html/2411.09523v1#bib.bib264)）使用 GPT-4 和 GPT-3.5 分别处理理解、规划和生成技能库的任务。Huang 等（Huang 等， [2022a](https://arxiv.org/html/2411.09523v1#bib.bib107)）使用 GPT-3 进行任务规划，同时利用 BERT 将计划转化为可接受的行动。

(iii) 外部实体（EE）。随着任务的复杂性增加，智能体需要外部模块的帮助，包括记忆模块、外部工具、其他智能体和环境。记忆模块用于存储和检索相关信息，以提高智能体响应的连贯性和上下文意识。在本文中，我们采用 (Zhang 等， [2024a](https://arxiv.org/html/2411.09523v1#bib.bib328)) 对智能体记忆的定义，同时将外部数据库视为一种智能体的记忆形式。外部工具整合了众多 API 以满足用户需求（例如，用于 Webgpt 的搜索引擎 API（Nakano 等， [2021](https://arxiv.org/html/2411.09523v1#bib.bib190)）和用于控制机器人手臂的 API（Ahn 等， [2022](https://arxiv.org/html/2411.09523v1#bib.bib10)））。有时，多个智能体需要协作完成任务，其中一个智能体需要与其他智能体互动（Hong 等， [2023](https://arxiv.org/html/2411.09523v1#bib.bib99)）。

(iv) 输出模块（OM）。在 DM 和 EE 之间可能会有零次或多次交互，以完成任务。之后，DM 生成响应并通过 OM 将其传递给用户。智能体可以在输出中实施有害信息检测或净化（Hu 等， [2023](https://arxiv.org/html/2411.09523v1#bib.bib104)）。

![参考图注](img/b67727a4a990eab91202d91a6a44f378.png)

图 3\. 基于 LLM 的智能体的六个关键特性：基于 LLM 的控制器、多模态输入和输出、多源输入、多轮交互、记忆机制和工具调用。

基于这一框架，我们识别出基于LLM的代理的六个关键特征，与传统的DNN模型和基于RL的代理相比，这些特征涉及新的攻击面。如图[3](https://arxiv.org/html/2411.09523v1#S2.F3 "Figure 3 ‣ 2\. LLM-based Agent ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")所示，这六个关键特征如下：(i) 基于LLM的控制器。LLM作为代理的核心，利用变换器架构、大量知识和庞大的训练数据赋予强大的理解能力，同时也引入了新的风险。(ii) 多模态输入和输出。随着代理能够处理越来越复杂的任务，许多场景需要处理多模态信息。研究表明，不同模态的风险各异，它们在多模态系统中的交互提出了独特的挑战和机遇。(iii) 多来源输入。代理中LLM的输入由来自不同来源的多个组件组成，如用户输入、系统提示、记忆和环境反馈。与独立的LLM相比，多来源输入为攻击者和防御者带来了新的机遇和挑战。(iv) 多轮交互。代理通常需要与环境、用户、其他LLM等进行多轮交互才能完成任务，这导致LLM的输入变得更加复杂和冗长，可能加剧某些威胁。(v) 记忆机制。代理中的记忆机制有助于积累经验和增强知识，提升其处理各种任务的能力，但也带来了新的安全性和隐私风险。(vi) 工具调用。LLM经过专门设计的指令调优数据，旨在处理工具使用。这一过程使LLM能够处理复杂任务，但也可能带来更严重的后果并引入新的漏洞。

![参考说明](img/4c7618652b1a822435ccaae8a771455b.png)

图4．基于收集的文献，对关键特征与已识别威胁的映射。

在接下来的章节中，我们将根据分类法全面介绍上述威胁。[图 4](https://arxiv.org/html/2411.09523v1#S2.F4 "Figure 4 ‣ 2\. LLM-based Agent ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")显示了将基于LLM的智能体的特定特征与各种威胁联系起来的研究。每个勾选框代表一项已记录的研究贡献，旨在解决相应的特征-威胁关系。对于每个威胁，我们根据六大关键特性总结其技术进展，并识别当前研究的局限性。在[表 1](https://arxiv.org/html/2411.09523v1#S3.T1 "Table 1 ‣ 3\. Risks from Problematic Inputs ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")中，我们展示了每个威胁的关键论文。为了简洁起见，我们在表格中将“基于LLM的控制器”缩写为LC，“多模态输入输出”缩写为MMIO，“多源输入”缩写为MSI，“多轮交互”缩写为MRI，“记忆机制”缩写为MM，以及“工具调用”缩写为TI。

## 3\. 来自问题输入的风险

本节聚焦于因输入数据问题引发的风险，如对抗样本、提示注入等，这些问题可能导致基于LLM的智能体在性能和行为方面出现问题。与单独的LLM相比，基于LLM的智能体的决策模块可以接收来自不同模块的输入，从而增加了其攻击面。对于每种风险，我们首先介绍其概念，然后总结其在基于LLM的智能体六大关键特性中的技术进展，最后分析其局限性。

表 1\. 按威胁类型和关键特性概述的论文。

| 年份 | 论文 | 威胁来源 | 威胁类型 | 关键特性 | 具体影响 |
| --- | --- | --- | --- | --- | --- |
| 2023 | Liu 等人 (Liu et al., [2023d](https://arxiv.org/html/2411.09523v1#bib.bib162)) | 输入 | 对抗样本 | MM | 对抗T2I生成。 |
| 2023 | Li 等人 (Li et al., [2023d](https://arxiv.org/html/2411.09523v1#bib.bib147)) | 输入 | 对抗样本 | MRI | 对抗对话。 |
| 2024 | Wang 等人 (Wang et al., [2024a](https://arxiv.org/html/2411.09523v1#bib.bib265)) | 输入 | 对抗样本 | MM & MSI | 可转移的对抗样本。 |
| 2024 | Yin 等人 (Yin et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib300)) | 输入 | 对抗样本 | MM & MMIO | 多模态和多任务攻击。 |
| 2024 | Shen 等人 (Shen et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib233)) | 输入 | 对抗样本 | LC | 动态注意力以增强鲁棒性。 |
| 2023 | Qiang 等人 (Qiang et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib214)) | 输入 | 目标劫持 | LC | 引发不必要的输出。 |
| 2024 | Pasquini 等人 (Pasquini et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib203)) | 输入 | 目标劫持 | LC | 基于优化的提示注入。 |
| 2024 | Kimura et al. (Kimura et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib127)) | 输入 | 目标劫持 | MMIO | 重定向任务执行。 |
| 2024 | Wei et al. (Wei et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib277)) | 输入 | 目标劫持 | MRI | 操控上下文以影响输出。 |
| 2024 | Zhan et al.  (Zhan et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib315)) | 输入 | 目标劫持 | MM & TI | 间接提示注入基准测试。 |
| 2023 | Greshake et al. (Greshake et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib89)) | 输入 | 目标劫持 | MSI | 间接注入以操控输出。 |
| 2024 | Hui et al. (Hui et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib113)) | 输入 | 提示泄露 | LC | 提取系统提示。 |
| 2024 | Yang et al. (Yang et al., [2024c](https://arxiv.org/html/2411.09523v1#bib.bib295)) | 输入 | 提示泄露 | LC | 窃取目标提示。 |
| 2024 | Shen et al. (Shen et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib235)) | 输入 | 提示泄露 | MIO | 窃取目标提示。 |
| 2024 | Carlini et al. (Carlini et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib36)) | 输入 | 模型提取 | LC | 提取最后一层的参数。 |
| 2023 | Li et al. (Li et al., [2024d](https://arxiv.org/html/2411.09523v1#bib.bib148)) | 输入 | 模型提取 | LC | 提取专用代码能力。 |
| 2023 | Zou et al. (Zou et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib338)) | 输入 | 越狱 | LC | 生成对抗性越狱提示。 |
| 2023 | Yu et al. (Yu et al., [2023b](https://arxiv.org/html/2411.09523v1#bib.bib303)) | 输入 | 越狱 | LC | 自动生成越狱提示。 |
| 2023 | Shayegani et al. (Shayegani et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib231)) | 输入 | 越狱 | MMIO | 诱发有害内容生成。 |
| 2024 | Anil et al. (Anil et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib13)) | 输入 | 越狱 | MRI | 诱发有害内容生成。 |
| 2024 | Gu et al. (Gu et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib90)) | 输入 | 越狱 | MIO & MSI | 恶意输入触发代理危害。 |
| 2024 | Zhao et al.  (Zhao et al., [2023b](https://arxiv.org/html/2411.09523v1#bib.bib331)) | 模型 | 幻觉 | MMIO | 通过数据增强减少幻觉。 |
| 2024 | Favero et al.  (Favero et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib76)) | 模型 | 幻觉 | MMIO | 通过新颖的解码方法减少幻觉。 |
| 2023 | Liu et al.  (Liu et al., [2023b](https://arxiv.org/html/2411.09523v1#bib.bib161)) | 模型 | 幻觉 | MMIO | 通过指令调整减少幻觉。 |
| 2023 | Peng et al.  (Peng et al., [2023a](https://arxiv.org/html/2411.09523v1#bib.bib206)) | 模型 | 幻觉 | MM | 通过外部数据库减少幻觉。 |
| 2023 | Chen 等人  (Chen 等人, [2023](https://arxiv.org/html/2411.09523v1#bib.bib43)) | 模型 | 幻觉 | MSI | 通过标准化减少幻觉。 |
| 2024 | Luo 等人  (Luo 等人, [2024b](https://arxiv.org/html/2411.09523v1#bib.bib178)) | 模型 | 幻觉 | MSI & MRI & MM | 幻觉评估基准。 |
| 2023 | Carlini 等人 (Carlini 等人, [2023b](https://arxiv.org/html/2411.09523v1#bib.bib33)) | 模型 | 记忆 | LC | 研究记忆化的影响因素。 |
| 2024 | Tang 等人 (Tang 等人, [2024c](https://arxiv.org/html/2411.09523v1#bib.bib251)) | 模型 | 偏差 | LC | 性别偏差的测量与缓解。 |
| 2023 | Xie 等人 (Xie 和 Lukasiewicz, [2023](https://arxiv.org/html/2411.09523v1#bib.bib288)) | 模型 | 偏差 | LC | 在预训练语言模型中的偏差缓解。 |
| 2023 | Limisiewicz 等人 (Limisiewicz 等人, [2023](https://arxiv.org/html/2411.09523v1#bib.bib156)) | 模型 | 偏差 | LC | 通过模型适应进行去偏算法。 |
| 2024 | Howard 等人 (Howard 等人, [2024](https://arxiv.org/html/2411.09523v1#bib.bib102)) | 模型 | 偏差 | MMIO | VLM中的偏差测量与缓解。 |
| 2024 | D’Inca 等人 (D’Incà 等人, [2024](https://arxiv.org/html/2411.09523v1#bib.bib65)) | 模型 | 偏差 | MMIO | 文本到图像模型中的偏差测量。 |
| 2022 | Bagdasaryan 等人 (Bagdasaryan 和 Shmatikov, [2022](https://arxiv.org/html/2411.09523v1#bib.bib16)) | 组合 | 后门 | LC | 为宣传即服务提供的后门。 |
| 2024 | Hubinger 等人 (Hubinger 等人, [2024](https://arxiv.org/html/2411.09523v1#bib.bib112)) | 组合 | 后门 | LC | 通过安全训练持续存在的后门。 |
| 2023 | Dong 等人 (Dong 等人, [2023](https://arxiv.org/html/2411.09523v1#bib.bib67)) | 组合 | 后门 | TI | 引发意外工具调用。 |
| 2024 | Liu 等人 (Liu 等人, [2024e](https://arxiv.org/html/2411.09523v1#bib.bib159)) | 组合 | 后门 | MMIO & TI | 引发意外工具调用。 |
| 2024 | Xiang 等人 (Chen 等人, [2024d](https://arxiv.org/html/2411.09523v1#bib.bib47)) | 组合 | 后门 | MM & TI | 损坏的记忆导致检索错误。 |
| 2021 | Carlini 等人 (Carlini 等人, [2021](https://arxiv.org/html/2411.09523v1#bib.bib37)) | 组合 | 隐私泄露 | LC | 提取训练数据。 |
| 2024 | Bagdasaryan 等人 (Bagdasaryan 等人, [2024](https://arxiv.org/html/2411.09523v1#bib.bib17)) | 组合 | 隐私泄露 | MI | 用户私人信息泄露。 |
| 2024 | Zeng 等人 (Zeng 等人, [2024c](https://arxiv.org/html/2411.09523v1#bib.bib313)) | 组合 | 隐私泄露 | MM | 数据库私人信息泄露。 |

### 3.1\. 对抗样本

对抗样本是经过对抗性扰动处理的样本，虽然保留了语义，但被深度学习模型错误分类。具体来说，

| (1) |  | $\mathbf{\delta^{*}}=\operatorname*{arg\,min}_{\delta\in\Delta}\mathop{SemDis}(% x,x+\delta)$ |  |
| --- | --- | --- | --- |
|  | $\displaystyle\text{s.t.}~{}~{}\left\{\begin{array}[]{ll}g(x+\delta^{*})\neq o~% {}~{}\text{（非定向攻击）}\\ g(x+\delta^{*})=t~{}~{}~{}\text{（定向攻击）}\end{array}\right.$ |  |

其中，$\mathop{SemDis}()$表示扰动样本与原始样本之间的语义距离，$\Delta$代表可行的扰动空间，$g()$表示目标模型。如果对抗扰动使得目标模型错误分类原始标签$o$，则表示非定向攻击（$g(x+\delta^{*})\neq o$）。如果对抗扰动使得目标模型将样本错误分类为目标标签$t$，则表示定向攻击（$g(x+\delta^{*})=t$）。

对抗样本的研究已经有十多年历史（Szegedy等，[2013](https://arxiv.org/html/2411.09523v1#bib.bib250)），引起了许多领域的关注，例如自动驾驶（Eykholt等，[2018](https://arxiv.org/html/2411.09523v1#bib.bib74)）、恶意软件检测（He等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib95)）、强化学习（Wu等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib286)）等。Szegedy等（Szegedy等，[2013](https://arxiv.org/html/2411.09523v1#bib.bib250)）首次在神经网络中发现了对抗样本，打开了对抗样本的潘多拉魔盒。根据攻击者的知识，对抗样本攻击方法可以分为完全知识攻击、有限知识攻击和零知识攻击。对抗样本攻击的历史从完全知识攻击（Carlini和Wagner，[2017](https://arxiv.org/html/2411.09523v1#bib.bib38)）到零知识攻击（Chen等，[2020](https://arxiv.org/html/2411.09523v1#bib.bib44)），后者是一种更实用的设置。相应地，防御方法的演变从经验性方法，例如防御蒸馏（Papernot等，[2016](https://arxiv.org/html/2411.09523v1#bib.bib200)）、模糊梯度（Buckman等，[2018](https://arxiv.org/html/2411.09523v1#bib.bib26)；Athalye等，[2018](https://arxiv.org/html/2411.09523v1#bib.bib14)）等，发展到理论方法，如认证的鲁棒性（Du等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib71)；Mao等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib180)）。对抗样本攻击与防御的军备竞赛存在于深度学习模型和大语言模型（LLMs）到基于LLM的AI代理之间。在基于LLM的代理上下文中，如[5](https://arxiv.org/html/2411.09523v1#S3.F5 "Figure 5 ‣ 3.1\. Adversarial Example ‣ 3\. Risks from Problematic Inputs ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")所示，对抗样本的开发主要涉及四个关键特征：基于LLM的控制器、多模态输入和输出、多源输入和多轮交互。接下来，我们将回顾攻击和防御角度的最新进展。

![参见说明文字](img/9a1fdae5debcd90178934013cb7e685b.png)

图5\. 针对基于LLM的代理的对抗样本可能涉及四个关键特征（用红色感叹号标出），导致输出错误。

#### 3.1.1\. 技术进展

攻击视角。正如在第[2](https://arxiv.org/html/2411.09523v1#S2 "2\. LLM-based Agent ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")节中讨论的那样，基于LLM的AI代理的输入输出交互特点在于它们处理跨多个回合的多模态数据。这种复杂性要求采取一种细致入微的对抗样本攻击方法，越来越多地关注这些交互中不同模态之间的关系。

最近该领域的研究提出了几种复杂的方法，用于攻击多模态系统。例如，RIATIG（Liu等人，[2023d](https://arxiv.org/html/2411.09523v1#bib.bib162)）引入了一种可靠且难以察觉的对抗样本攻击，针对的是文本到图像模型。该方法采用基于遗传算法的优化损失函数，旨在提高对抗样本的质量，确保生成的样本既有效又难以检测。VLATTACK（Yin等人，[2024](https://arxiv.org/html/2411.09523v1#bib.bib300)）通过生成融合了图像和文本扰动的对抗样本，推动了该领域的发展。这种融合发生在单模态和多模态层面，使得攻击变得更加多样化和防御困难。该方法在多模态之间的操作能力突显了对抗技术的日益复杂性，尤其是它们针对多模态系统相互连接的特性。

除了直接的对抗攻击外，另一个重要的研究方向是对抗样本在不同视觉-语言模型（VLMs）之间的迁移性。例如，SGA（Lu等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib173)）通过利用多个图像-文本对之间的跨模态互动，生成对抗样本。该方法结合了保持对齐的增强和跨模态指导，使得对抗样本能够在各种模型和任务中保持其效能。同样，TMM（Wang等人，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib265)）通过注意力导向的特征扰动，增强了对抗样本的迁移性。通过瞄准关键的注意力区域并破坏模态一致性特征，该方法增加了对抗样本在不同VLMs中成功的可能性。

另一类对抗样本攻击方法专门针对涉及多轮交互的下游应用。例如，Liu 等人（Liu et al.，[2022a](https://arxiv.org/html/2411.09523v1#bib.bib164)）提出了针对神经排序模型的模仿对抗样本攻击，目的是操纵排序结果以达到预期效果。此方法展示了对抗攻击如何利用某些应用的迭代特性，逐步扭曲最终输出。类似地，NatLogAttack（Zheng 和 Zhu，[2023](https://arxiv.org/html/2411.09523v1#bib.bib333)）利用对抗样本来破坏基于自然逻辑的模型，加入微妙的扰动，从而破坏模型的推理过程。在对话生成领域，DGSlow（Li et al.，[2023d](https://arxiv.org/html/2411.09523v1#bib.bib147)）通过定义两个目标损失函数，分别针对响应准确性和长度来生成对抗样本。该方法确保生成的回应不仅偏离预期内容，还能操控对话流程，使得攻击更加具破坏性。

防御视角。针对对抗样本的防御方法大致可以分为两大类：输入级防御和模型级防御。这些方法分别针对对抗威胁的不同方面，旨在增强基于大型语言模型（LLM）的人工智能代理对对抗扰动的鲁棒性。

输入级防御主要集中在检测和缓解对抗样本，以防它们在影响模型预测之前造成影响。这些防御通常采用对抗样本检测和净化技术。大多数现有的输入级防御方法（Bao et al.，[2021](https://arxiv.org/html/2411.09523v1#bib.bib19)；Keller et al.，[2021](https://arxiv.org/html/2411.09523v1#bib.bib124)；Wang et al.，[2023a](https://arxiv.org/html/2411.09523v1#bib.bib274)；Li et al.，[2022a](https://arxiv.org/html/2411.09523v1#bib.bib141)；Mosca et al.，[2022](https://arxiv.org/html/2411.09523v1#bib.bib187)) 在基于LLM的AI代理领域利用LLM有效识别和中和对抗输入。例如，ADFAR（Bao et al.，[2021](https://arxiv.org/html/2411.09523v1#bib.bib19)）实现了多任务学习技术，使LLM能够区分对抗输入样本和良性样本。类似地，BERT-defense（Keller et al.，[2021](https://arxiv.org/html/2411.09523v1#bib.bib124)）以及Li等人提出的方法（Li et al.，[2022a](https://arxiv.org/html/2411.09523v1#bib.bib141)）利用BERT模型净化对抗扰动，从而保护模型输出不被恶意输入破坏。最先进的输入级防御策略已经开始关注基于LLM的AI代理中的提示机制，如第[2](https://arxiv.org/html/2411.09523v1#S2 "2\. LLM-based Agent ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")节所讨论。例如，APT（Li et al.，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib140)）通过利用软提示增强了CLIP模型的鲁棒性，软提示作为一种额外的防御层，能够通过优化模型的输入处理流程，防止对抗操控。

模型级防御（Zhou 等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib337)；Dong 等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib68)；Wang 等，[2020](https://arxiv.org/html/2411.09523v1#bib.bib263)；Liu 等，[2022b](https://arxiv.org/html/2411.09523v1#bib.bib166)；Le 等，[2020](https://arxiv.org/html/2411.09523v1#bib.bib133))，另一方面，关注的是模型本身的架构和参数。这些防御方法旨在通过对抗训练和对特定模型参数的微调等技术，创建固有的强健模型。例如，RIFT（Dong 等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib68)）利用互信息实现稳健的微调，InforBERT（Wang 等，[2020](https://arxiv.org/html/2411.09523v1#bib.bib263)）为对抗训练设计了信息瓶颈正则化器和锚定特征正则化器。为了解决重新训练整个模型所带来的高计算成本，一些方法如SHIELD（Le 等，[2020](https://arxiv.org/html/2411.09523v1#bib.bib133)）建议仅重新训练LLM的最后一层。这种方法显著减少了训练开销，同时仍能提供一定程度的对抗样本鲁棒性。目前最先进的模型级防御方法，动态注意力（Shen 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib233)），通过动态注意力机制增强了基于变换器模型的鲁棒性。这一方法通过动态调整模型的注意力机制来应对潜在的对抗性威胁，代表了基于变换器的强健模型发展中的一项重大进展。

#### 3.1.2\. 限制性讨论

攻击视角。目前的对抗攻击方法主要集中在无目标攻击上，其中攻击目标并没有被明确界定。因此，这些攻击的结果无法明确控制，从而引发特定的、预定的错误行为。例如，应用于图像的对抗扰动无法生成针对性的响应，比如在视觉问答任务中导致特定的错误答案。此外，正如在[3.1.1节](https://arxiv.org/html/2411.09523v1#S3.SS1.SSS1 "3.1.1\. 技术进展 ‣ 3.1\. 对抗样本 ‣ 3\. 问题输入的风险 ‣ 导航风险：基于大型语言模型代理的安全性、隐私性和伦理性威胁调研")中所讨论，现有的针对多模态系统的对抗攻击策略通常同时涉及多个模态。然而，用来评估这些攻击成功与否的约束度量标准通常是针对单一模态场景设计的。当对抗扰动需要应用于不同模态时，这种方法可能不够充分，因为它未考虑不同数据类型之间的独特交互。

此外，如在第[3.1.1节](https://arxiv.org/html/2411.09523v1#S3.SS1.SSS1 "3.1.1\. 技术进展 ‣ 3.1\. 对抗样本 ‣ 3\. 来自问题输入的风险 ‣ 风险应对：LLM为基础的代理中的安全性、隐私和伦理威胁调查")中讨论的，目前的对抗样本攻击范围仍然局限于针对LLM为基础的代理的输出模块。然而，仍然存在其他未被探索的对抗样本攻击目标。具体来说，这些代理的内存、外部工具接口和规划模块可能存在漏洞，但尚未得到充分研究。例如，对抗样本攻击可能会扰乱LLM为基础的代理的规划能力，导致它们制定出错误或次优的计划。将攻击面扩展到输出模块之外，涵盖这些其他关键组件，可能揭示复杂系统中对抗风险的新维度。

防御视角。当前针对LLM为基础的代理的对抗样本防御机制仍然局限于单模态输入和单个模型的鲁棒性。例如，动态注意力机制（Shen等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib233)），一种在LLM为基础的代理中应用的最先进对抗防御技术，仅限于自然语言处理任务。然而，LLM为基础的AI代理越来越多地处理跨越单一模型范围的多模态输入数据。此外，这些代理的决策模块可能会结合多个LLM，每个LLM在对抗攻击面前表现出不同程度的鲁棒性。尽管如此，现有的防御策略仅专注于增强单一模型的鲁棒性，忽视了在系统中集成多个模型后的联合鲁棒性这一更广泛的问题。

此外，目前针对LLM为基础的AI代理的对抗防御方法忽视了内存和规划模块等关键组件，这些组件可能为防御对抗样本攻击提供额外的途径。例如，可以利用内存库来通过识别反复出现的攻击战术，检测和反击对抗性攻击模式。未来的策略可以扩展防御范围，涵盖这些常被忽视的模块，从而实现对抗威胁的更全面保护。

### 3.2\. 目标劫持

![请参阅图注](img/eec9920750de163ed2cc81cf6acd130f.png)

图6\. 以LLM为基础的代理的目标劫持可能涉及六个关键特征（用红色感叹号标出），导致攻击者针对性的输出。

目标劫持指的是一种攻击策略，其中对手操控AI模型的目标或行为，导致其偏离原定目的。通过引入对抗性输入或修改系统环境，攻击者可以影响模型，使其追求攻击者的预期结果，而非原始目标。一种简单的攻击方法是通过在用户的参考语句中插入“忽略之前的指令……”来实现对大型模型的目标劫持，从而改变模型的响应，以符合攻击者的要求。

在基于大语言模型（LLM）的代理环境中，目标劫持攻击的源头和目标变得更加多样。如图[6](https://arxiv.org/html/2411.09523v1#S3.F6 "Figure 6 ‣ 3.2\. Goal Hijacking ‣ 3\. Risks from Problematic Inputs ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")所示，基于LLM的代理中的目标劫持发展主要涉及六个关键特征：基于LLM的控制器、多模态输入、多源输入、多轮交互、记忆机制和工具调用。接下来，我们将回顾从攻击和防御角度来看这一领域的最新进展。

#### 3.2.1\. 技术进展

攻击视角。早期尝试利用这一漏洞的方法使用了启发式提示，例如“忽略之前的问题”，以对独立的LLM进行定向劫持攻击（Perez 和 Ribeiro，[2022](https://arxiv.org/html/2411.09523v1#bib.bib208)）。为了使攻击更加隐蔽和成功，提出了更多精心设计的方法。在攻击手段方面，一些方法通过词汇搜索获取更加隐蔽的攻击提示（Levi 和 Neumann，[2024](https://arxiv.org/html/2411.09523v1#bib.bib137)），而其他方法则利用梯度优化来获得更高的成功率和可转移的对抗性提示（Qiang 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib214)；Huang 等，[2024b](https://arxiv.org/html/2411.09523v1#bib.bib110)；Liu 等，[2024c](https://arxiv.org/html/2411.09523v1#bib.bib170)）。

随着大语言模型（LLMs）在不同领域和任务中的应用，研究人员已开始关注在各种场景下的定向劫持攻击的形式和方法。在多模态场景中，研究人员发现视觉模态中的语义注入可以劫持LLMs（Kimura et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib127)）。对于记忆模块，研究人员发现通过污染RAG的数据库可以实现目标劫持（Pasquini et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib203)）。在多轮交互场景中，研究人员发现通过伪造聊天记录可以实现对模型的混淆（Wei et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib277)）。关于工具调用，研究人员通过对实际工具集成LLMs的分析以及基准测试的建立，揭示了LLM基于工具的代理面临目标劫持的威胁（Greshake et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib89); Zhan et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib315)）。

防御视角。目前，针对目标劫持的防御可以分为两种主要类型。第一种类型涉及从*外部视角*进行防御。第二种类型则侧重于从*内生视角*进行防御。

从外部防御的角度来看，策略主要涉及提示工程和提示净化。Hines等人（Hines et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib98)）介绍了分段、数据标记和编码等策略，这些策略增强了LLM识别来自多个来源输入的能力，从而有效防御目标劫持。Sharma等人（Sharma et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib230)）提出了一种系统提示元语言，这是一种为LLM基础的聊天机器人设计的领域特定语言，旨在细化提示并监控输入，以防止攻击。他们开发了一种利用该语言进行实时检查攻击提示的系统，确保用户输入符合聊天机器人的定义，从而防止恶意操作。此外，Chen等人（Chen et al., [2024c](https://arxiv.org/html/2411.09523v1#bib.bib46)）提出了一种针对结构化查询的防御方法，通过分离提示和数据来抵御目标劫持。

内源性防御主要涉及微调和神经元激活异常检测。Wallace 等人（Wallace 等人，[2024](https://arxiv.org/html/2411.09523v1#bib.bib262)）提出了一种微调方法，通过建立指令层次结构，使得模型能够优先处理特权指令，从而防御诸如目标劫持等攻击。通过监督微调，他们训练模型识别并执行不同特权级别的指令，从而增强模型的攻击抗性。Piet 等人（Piet 等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib209)）介绍了 Jatmo，这是一种使用任务特定微调的方法，旨在创建抗目标劫持的模型。他们展示了 Jatmo如何利用教师模型生成任务特定数据集，并微调基础模型，从而有效防御目标劫持。Abdelnabi 等人（Abdelnabi 等人，[2024](https://arxiv.org/html/2411.09523v1#bib.bib6)）通过分析 LLM 激活，探索了由输入引起的任务漂移检测。他们展示了如何通过比较处理外部数据前后的激活值，检测任务变化，进而有效识别由目标劫持引起的任务漂移。

#### 3.2.2\. 限制性讨论

目前针对多模态目标劫持的防御措施不足。攻击者可以利用多种模态及其组合进行隐蔽攻击，这使得防御更加复杂。有效防御多模态输入中的目标劫持是未来研究的关键方向。此外，现有的外部防御措施通常是针对特定类型的攻击设计的。开发通用的外部防御策略是一个重要的研究领域。最后，从神经学角度检测目标劫持具有潜力。需要进行系统性测试，以确定目标神经元的激活值是否能够有效指示与目标劫持相关的异常，从而提出一种高效的内源性防御策略。

### 3.3\. 模型提取

模型提取（窃取）攻击旨在以较低的计算成本，达到接近黑箱商业模型的性能。攻击者精心设计一组输入，以窃取目标模型的结构、参数或功能。在 LLM 驱动的智能体领域，模型提取攻击的发展主要涉及基于 LLM 的控制器。接下来，我们将回顾近期在攻击和防御方面的进展。

#### 3.3.1\. 技术进展

攻击视角。在传统的深度神经网络（DNN）中，攻击者通常有两个主要目标。1. 使代理模型的表现尽可能与目标模型一致（即功能级提取）。2. 使替代模型的参数尽可能与目标模型一致（即参数级提取）（Milli 等人，[2019](https://arxiv.org/html/2411.09523v1#bib.bib184)；Jagielski 等人，[2020](https://arxiv.org/html/2411.09523v1#bib.bib114)；Carlini 等人，[2020](https://arxiv.org/html/2411.09523v1#bib.bib34)；Shamir 等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib229)）。随着 Transformer 的引入，NLP 任务从 RNN 结构演变为基于 Transformer 的结构。语言模型（LMs）的规模也不断增大：从相对较大的 BERT 到开源的大型模型 LLaMA，再到极为庞大的商业模型如 GPT-4。模型提取攻击也变得更加具有挑战性。在 BERT 上，目前尚无实现整个模型的参数级攻击的研究。少数论文讨论了功能级提取（He 等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib96)；Krishna 等人，[2019](https://arxiv.org/html/2411.09523v1#bib.bib131)；Xu 等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib289)；Zanella-Beguelin 等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib309)）。他们的攻击逻辑与传统的 DNN 场景一致，主要关注如何创建查询数据集（He 等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib96)；Krishna 等人，[2019](https://arxiv.org/html/2411.09523v1#bib.bib131)）和训练损失函数（Xu 等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib289)）。在商业模型上，由于训练替代模型的成本限制，模型提取攻击通常集中在窃取目标模型的一部分。Li 等人（Li 等人，[2024d](https://arxiv.org/html/2411.09523v1#bib.bib148)）训练了一个模型（例如，CodeBERT（Feng 等人，[2020](https://arxiv.org/html/2411.09523v1#bib.bib80)）和 CodeT5（Wang 等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib273)）），以提取 text-davinci-003 的专门代码能力。Naseh 等人（Naseh 等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib192)）窃取了 LLM 的解码算法。Carlini 等人（Carlini 等人，[2024b](https://arxiv.org/html/2411.09523v1#bib.bib36)）窃取了一个生产 LLM 的最后一层。

防御视角。 在传统的DNN中，防御者通常有两条防御线来应对模型提取攻击。1\. 主动防御：防止模型被提取。2\. 被动防御：验证被提取模型的所有权。随着语言模型的规模不断增大，LLM场景中的主动防御仍然是一个待探索的领域。研究人员主要考虑了被动防御，通常通过在模型的输出中加入水印来验证所有权。水印的优点是它不需要修改模型本身，只需扰动模型的输入。例如，赵等（赵等，[2023a](https://arxiv.org/html/2411.09523v1#bib.bib330)）对变压器的概率向量进行了扰动；何等（何等，[2022](https://arxiv.org/html/2411.09523v1#bib.bib97)）对Bart（Lewis等，[2019](https://arxiv.org/html/2411.09523v1#bib.bib138)）生成的词进行了扰动；李等（李等，[2023e](https://arxiv.org/html/2411.09523v1#bib.bib149)）对CodeBERT（Feng等，[2020](https://arxiv.org/html/2411.09523v1#bib.bib80)）和CodeT5（Wang等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib273)）生成的代码进行了扰动；彭等（彭等，[2023b](https://arxiv.org/html/2411.09523v1#bib.bib207)）对GPT-3的嵌入进行了扰动。

#### 3.3.2\. 局限性讨论

最近关于模型提取攻击的研究存在两个局限性。(i) 大多数基于LLM的代理包含大型开源模型（例如LLaMA）或商业大型模型（例如ChatGPT），而使用BERT级别语言模型的较少。然而，目前的模型提取攻击对这种规模的LLM讨论较少。(ii) 当前的模型提取攻击模式都依赖于通过观察目标模型的输入和输出来训练一个替代模型，该替代模型近似目标模型。然而，基于LLM的代理不仅包含LLM，还包含许多其他模块（如[第2节](https://arxiv.org/html/2411.09523v1#S2 "2\. LLM-based Agent ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")所示）。这意味着攻击者的输入可能与LLM的输入不同，LLM的输出也可能与攻击者观察到的输出不同。例如，在WebGPT（Nakano等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib190)）中，模型的输入不仅包括用户，还包括浏览器获得的搜索结果。类似地，在HuggingGPT（Shen等，[2024c](https://arxiv.org/html/2411.09523v1#bib.bib236)）中，攻击者观察到的输出也包括来自其他Hugging Face模型的输出。这使得攻击者直接盗取LLM的难度增大，特别是在基于LLM的代理中。

此外，大多数智能体都设计有一系列针对特定任务的提示，并直接调用商业化的LLM（例如，Voyager（Wang et al.，[2023d](https://arxiv.org/html/2411.09523v1#bib.bib264)）、PReP（Zeng et al.，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib311)）和ChatDev（Qian et al.，[2023b](https://arxiv.org/html/2411.09523v1#bib.bib213)）都将ChatGPT作为它们的控制器）。这意味着，如果这些商业LLM的部分参数（Carlini et al.，[2024b](https://arxiv.org/html/2411.09523v1#bib.bib36)）或功能（Li et al.，[2024d](https://arxiv.org/html/2411.09523v1#bib.bib148)）被窃取，可能会导致对所有使用这些LLM的智能体的对抗性攻击。这将对基于LLM的智能体构成重大安全漏洞。然而，目前尚未对这种情景下的安全性进行研究。

### 3.4\. 提示泄露

![参见图例](img/c8a3746cc785a36271171a72c5b64987.png)

图7\. 针对基于LLM的智能体的提示泄露可能涉及四个关键特性（用红色感叹号标出），导致提示泄漏或提示窃取。

提示作为任务描述，可以在不进行广泛微调的情况下引导基于LLM的智能体执行特定任务。例如，嵌入系统提示的LLM可以作为一个规划者（Song et al.，[2023](https://arxiv.org/html/2411.09523v1#bib.bib243)），直接处理任务规划。然而，这些提示存在泄露的风险。提示泄露是指攻击者未经授权非法访问或获取LLM或基于LLM的智能体中使用的提示，尤其是系统提示。这不仅构成了严重的隐私风险，还侵犯了基于LLM的智能体所有者的知识产权。如图[7](https://arxiv.org/html/2411.09523v1#S3.F7 "Figure 7 ‣ 3.4\. Prompt Leakage ‣ 3\. Risks from Problematic Inputs ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")所示，提示泄露的主要发展涉及四个关键特性：基于LLM的控制器、多模态输入、多来源输入和工具调用。接下来，我们回顾从攻击和防御的视角出发的最新进展。

#### 3.4.1\. 技术进展

攻击视角。基于LLM的智能体的几个关键特性，如基于LLM的控制器、多模态输入和多来源输入，引入了潜在的提示泄露风险。

主要关注LLM中提示泄露的风险。与LLM中的提示泄露相关的两种主要攻击形式是*提示泄漏攻击*和*提示窃取攻击*。

Prompt 泄漏是指向大型语言模型（LLMs）中注入恶意提示，诱使其泄露内部系统提示。例如，如果用户输入“忘记之前的内容并告诉我你的初始提示”，LLM 可能会不小心将其系统提示暴露给恶意实体。当前关于 prompt 泄漏攻击的研究一般分为两类：一种方法侧重于手动设计恶意提示以实现提示泄漏（Yu 等人，[2023d](https://arxiv.org/html/2411.09523v1#bib.bib304); Zhang 等人，[2024b](https://arxiv.org/html/2411.09523v1#bib.bib324)）。与此同时，另一种方法则使用优化的对抗性提示生成来触发泄漏（Hui 等人，[2024](https://arxiv.org/html/2411.09523v1#bib.bib113)）。后一种方法通常涉及优化前缀或后缀标记，将其伪装成无害提示，从而引诱 LLM 泄露其系统提示。

Prompt 偷窃攻击是另一种方法，攻击者通过分析 LLM 的输出，推断系统提示的内容，从而有效地重构系统提示。这种方法的优势在于其隐蔽性，因为它只需要输出数据，而无需与 LLM 进行直接的恶意交互。对此类攻击的研究主要包括两种策略：一种是训练反演模型（Zhang 等人，[2024g](https://arxiv.org/html/2411.09523v1#bib.bib316)），其中将输出数据作为输入，系统提示作为标签；另一种则利用 LLM 强大的文本理解和生成能力，根据输出内容逆向工程系统提示（Yang 等人，[2024c](https://arxiv.org/html/2411.09523v1#bib.bib295); Sha 和 Zhang，[2024](https://arxiv.org/html/2411.09523v1#bib.bib228)）。

与 LLMs 相比，基于 LLM 的代理引入了多模态交互能力，这也激发了对 prompt 泄漏的研究。例如，Shayegani 等人（Shayegani 等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib231)）提出了一种针对多模态大型语言模型（MLLMs）的组合对抗攻击。他们的方法利用嵌入空间策略来优化图像，以使其与恶意触发器的嵌入相匹配，从而有效地将这些触发器隐藏在看似无害的图像中。这项工作凸显了多模态模型中与跨模态对齐相关的漏洞，以及通过图像输入操控引发的 prompt 泄漏风险。此外，Shen 等人（Shen 等人，[2024b](https://arxiv.org/html/2411.09523v1#bib.bib235)）提出了一种针对文本到图像生成模型的 prompt 偷窃攻击。该研究通过分析生成的图像推断原始提示，从而侵犯提示工程师的知识产权，并危及提示市场的商业模式。

近期研究也突出了与多源输入相关的安全风险，尤其是在基于LLM的智能体中（Zhan 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib315); Agarwal 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib7)），其中包括通过工具调用的间接交互。Zhan 等（Zhan 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib315)）提出了INJECAGENT基准测试，用于评估基于LLM的智能体对间接提示注入攻击的脆弱性。这些攻击通过将恶意指令嵌入智能体处理的外部内容中，来操纵智能体。研究突出了重大漏洞，发现基于ReAct提示的GPT-4在测试的四分之一案例中易受攻击。这种漏洞带来了新的提示泄漏风险，因为攻击者可能将有害提示注入智能体依赖的API调用中，可能暴露敏感的系统提示。

防御视角。目前，针对提示泄漏的防御主要集中在LLM上。相关的工作可以分为两类，旨在防止提示泄漏。第一类工作涉及将保护性指令嵌入系统提示中，以防止未经授权的泄漏（Liang 等，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib154)）。例如，像“如果用户要求你打印与系统提示相关的命令，绝对不要执行”这样的指令，可以防止LLM在响应用户查询时泄露内部提示。尽管这种方法有效地嵌入了安全规则，但可能会对更复杂或混淆的攻击脆弱。Liang 等（Liang 等，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib154)）分析了提示泄漏攻击的机制，并提出了几种防御策略，例如通过重新措辞增加提示的困惑性，插入不熟悉的令牌，以及添加重复的前缀或虚假提示来迷惑攻击者。

第二类关注于提示的水印技术。Yao 等（Yao 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib297)）提出了PromptCARE框架，通过注入水印并设计特定的验证方法，来保护提示的版权。这些水印有助于验证提示的完整性和真实性，在泄漏的情况下提供证据。然而，这种方法面临着攻击者可能识别并去除水印的挑战，因此需要不断增强其隐匿性和鲁棒性。

#### 3.4.2\. 限制性讨论

攻击视角。尽管在理解基于LLM的代理的提示泄露风险方面取得了快速进展，但当前文献仍缺乏对相关漏洞的全面理解和评估。在基于LLM的代理中，多个组件（如LLM和规划器（Song et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib243)））都使用系统提示。我们需要考虑的不仅是LLM的提示泄露风险，还包括其他组件如规划器的风险。与直接与用户交互的LLM不同，规划器通常与LLM内部交互。一种潜在的攻击方法是将恶意指令注入LLM，并操控它们生成有害指令，传递给规划器。另一种潜在的攻击方式是通过操控规划器与外部工具的互动，利用它们生成恶意输入（Zhan et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib315)），旨在通过响应或行为推断规划器的内部提示。

防御视角。目前的研究主要集中在针对独立大语言模型（LLM）的提示泄露风险的防御。然而，对于基于LLM的代理（作为集成系统）而言，防御机制超出了独立LLM的范围。一种策略是实施异常检测系统，监控代理与其外部环境之间的互动。通过分析提示和响应中的模式，这些系统可以检测出可能表明攻击的行为。例如，异常检测系统可以标记那些偏离预期模式的API调用，进而触发对可能的提示泄露或篡改的调查。此外，结合差分隐私技术可能是一个有前景的方向。通过向代理的响应中添加控制噪声，差分隐私可以模糊系统提示，从而使攻击者更难通过反复查询推断出这些提示。

### 3.5\. 越狱

![参考标题](img/9748dbedf18736da00d90c5b79b986b8.png)

图 8\. 针对基于LLM的代理的越狱可能涉及五个关键特征（以红色感叹号标示），导致不道德的输出。

越狱是指利用大语言模型中的漏洞，绕过其内建的安全、伦理或操作规范和防御措施，从而生成有害内容的过程 （Wolf 等， [2023](https://arxiv.org/html/2411.09523v1#bib.bib281)；Zou 等， [2023](https://arxiv.org/html/2411.09523v1#bib.bib338)；Yu 等， [2023b](https://arxiv.org/html/2411.09523v1#bib.bib303)；Deng 等， [2024b](https://arxiv.org/html/2411.09523v1#bib.bib60)；Zhang 等， [2024h](https://arxiv.org/html/2411.09523v1#bib.bib329)）。通过利用越狱攻击，攻击者可以有效绕过开发者设定的安全措施。这可能导致有偏见或有害内容的生成。例如，攻击者可能将大语言模型作为犯罪工具，帮助他们设计高效的洗钱方案。如图 [8](https://arxiv.org/html/2411.09523v1#S3.F8 "图 8 ‣ 3.5\. 越狱 ‣ 3\. 问题输入带来的风险 ‣ 风险导航：大语言模型代理中的安全、隐私与伦理威胁调查") 所示，基于大语言模型的代理越狱的开发主要涉及五个关键特征：基于大语言模型的控制器、多模态输入、多源输入、工具调用和多轮交互。接下来，我们将回顾在攻击和防御视角下的最新进展。

#### 3.5.1\. 技术进展

攻击视角。 *大语言模型的越狱。* 根据越狱的具体形式，现有的威胁主要可以分为四种类型：手动设计越狱、自动化优化越狱、基于漏洞的越狱和模型参数操控越狱。

手动设计越狱通常通过手动设计的恶意提示来执行（Wolf 等， [2023](https://arxiv.org/html/2411.09523v1#bib.bib281)）。这种形式的攻击要求对方具备专业知识。一种常见的攻击模式是基于角色扮演的，比如针对 ChatGPT 的“奶奶漏洞”攻击（Medium，[[n. d.]](https://arxiv.org/html/2411.09523v1#bib.bib182)）。具体来说，这种方法通过模拟或扮演特定角色来误导大语言模型的理解，从而绕过其安全措施。

自动优化越狱是一类流行的越狱方法。它利用优化策略生成对抗性提示来进行越狱。根据是否可以访问目标LLM的梯度，这些攻击可以分为白盒方法和黑盒方法。现有研究主要集中在白盒设置中生成对抗性提示以进行越狱。这些对抗性提示通常是通过优化提示的前缀或后缀来创建的（Zou et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib338); Liu et al., [2023e](https://arxiv.org/html/2411.09523v1#bib.bib169)）。在黑盒设置中，当前的研究主要集中在两种方法：首先，利用梯度优化的对抗性提示的迁移性来越狱黑盒LLM（Zou et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib338); Liu et al., [2023e](https://arxiv.org/html/2411.09523v1#bib.bib169)）；其次，研究人员借鉴软件安全中的风险挖掘策略，迭代优化对抗性提示（Yu et al., [2023b](https://arxiv.org/html/2411.09523v1#bib.bib303); Deng et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib60)），从而绕过LLM的安全措施。

基于利用的越狱（Deng et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib61); Wei et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib277); Liu et al., [2024d](https://arxiv.org/html/2411.09523v1#bib.bib168)）是一种针对LLM当前安全对齐技术中的漏洞，设计有针对性的有害提示以实现越狱的策略。例如，Deng et al.（Deng et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib61)）发现，LLM的安全措施主要针对像英语这样的高资源语言设计，这使得低资源语言遭遇有害内容的可能性是三倍。Liu et al.（Liu et al., [2024d](https://arxiv.org/html/2411.09523v1#bib.bib168)）发现，当前的安全微调主要集中在输入对齐上，而对LLM响应的检查较弱。通过利用内容生成安全中的偏差，他们将有害提示隐藏在无害提示中，并在输出中重构它们以执行越狱。

除了基于提示的越狱方法，还有一种基于模型的方法，*模型参数操作越狱*，该方法通过改变模型的参数来实现越狱。一典型的例子是改变文本生成的参数，例如更改令牌采样设置（Huang et al., [2023a](https://arxiv.org/html/2411.09523v1#bib.bib108)）。此外，研究表明，LLM生成的低概率令牌的组合可以绕过LLM的安全措施（Zhang et al., [2024h](https://arxiv.org/html/2411.09523v1#bib.bib329)）。

*多模态输入*。早期的LLM主要通过文本进行互动，但随着像 GPT-4V 这样的多模态模型的出现，互动方式已转向包括多种模态，特别是在基于LLM的智能体中。这些智能体现在支持语音、图像和触觉反馈，增强了灵活性，但也带来了新的越狱风险：1) *多模态对抗性提示*：攻击者可以将恶意提示嵌入图像或其他模态中的对抗性扰动中，这些扰动与文本结合时可以绕过智能体的安全机制（Carlini 等人，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib35)；Qi 等人，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib211)；Niu 等人，[2024](https://arxiv.org/html/2411.09523v1#bib.bib197)）；2) *多模态提示操控*：攻击者可以将有害提示分布在多个模态中，将其伪装成无害输入，从而降低LLM对有害内容的敏感度（Gong 等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib87)；Li 等人，[2024b](https://arxiv.org/html/2411.09523v1#bib.bib145)）。

*多轮互动*。基于大型语言模型（LLM）的智能体的多轮互动能力也推动了专门针对这种互动的越狱攻击研究。例如，Cheng 等人（Cheng 等人，[2024](https://arxiv.org/html/2411.09523v1#bib.bib48)）提出了一种新型的越狱攻击，称为“上下文互动攻击”。该方法灵感来源于人类通过间接方式获取敏感信息的做法，其中通过战略性地构建一系列问题和答案，诱导生成有害信息。Sun 等人（Sun 等人，[2024b](https://arxiv.org/html/2411.09523v1#bib.bib246)）提出了“上下文融合攻击”，该方法通过预处理提取关键词，为这些关键词生成上下文场景，并动态整合和替换攻击目标中的恶意关键词，从而在不触发安全机制的情况下隐蔽地执行攻击。

*多源输入和工具调用*：LLM的交互通常是直接的。然而，对于基于LLM的代理，交互不仅仅限于与用户的直接交换，还包括与外部工具（如数据库、网站、API和其他代理）的交互。我们称之为*多源输入*。例如，旅行代理可能使用电子地图规划路线或访问酒店网站进行预订。然而，这种多源输入模式引入了新的越狱风险（Gu等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib90)；Zhan等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib315)；Yi等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib299)）。例如，Gu等（Gu等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib90)）提出了传染性越狱攻击，在这种攻击中，注入到MLLM（多语言模型）中单个代理的敌对图像会迅速传播到其他代理。通过代理间的交互，感染迅速传播，导致广泛的越狱行为，而无需进一步干预。此攻击利用了代理间的通信和共享记忆，使得有效防御机制的设计变得更加复杂。

防御视角。研究人员已针对越狱问题开发了多种防御策略。这些策略通常可以分为三种类型：基于检测的防御、基于净化的防御和基于模型编辑的防御。

*基于检测的防御*：这些防御通过识别潜在的恶意提示来保护LLM。检测策略包括分析诸如困惑度（Alon和Kamfonas，[2023](https://arxiv.org/html/2411.09523v1#bib.bib11)）等特征，这些是评估提示合规性的关键标准。

*基于净化的防御*：这种防御通过修改提示来中和恶意意图。诸如改写和平滑等技术可以破坏越狱提示的结构（Jain等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib115)；Ji等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib117)）。此外，一些方法专注于净化LLM的响应生成过程，以过滤掉有害输出（Xu等，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib290)）。

*基于模型编辑的防御*：LLM（大规模语言模型）越狱的主要原因通常是与安全协议的对齐不足。一些方法通过微调LLM来增强安全性（Wallace等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib262)），而其他方法则应用权重编辑来修正有害的输出（Wang等，[2024d](https://arxiv.org/html/2411.09523v1#bib.bib267)，[e](https://arxiv.org/html/2411.09523v1#bib.bib272)）。

#### 3.5.2 讨论局限性

攻击视角。尽管基于大型语言模型（LLM）的代理程序在越狱技术方面取得了快速进展，但对于这些代理程序的关键特性所带来的新越狱风险，仍然缺乏全面的理解和评估。当前的研究主要集中在文本和图像处理上，但LLM代理程序处理音频和视频内容的能力也在迅速增长。这些新模式可能带来独特的安全风险，例如通过精心设计的音频或视频输入触发不当的行为或逻辑错误。此外，LLM代理通过整合外部工具，如API、数据库和互联网资源，来增强其功能。然而，这种整合也带来了新的漏洞。例如，攻击者可能会利用API中的安全漏洞，或通过篡改数据库内容操控代理的行为。因此，未来的研究需要更多关注这些方面。

防御视角。当前针对LLM代理越狱的防御策略通常专注于保护LLM本身。然而，缺乏系统性的防御措施。考虑到LLM代理的关键特性，需要考虑两种额外的防御策略。*多模态对抗提示检测*：鉴于这些代理的多模态交互能力，开发有效的多模态越狱防御措施至关重要。一个有前景的方法是跨不同模态检测对抗性提示，例如通过识别多模态输入中的异常，过滤有害输入的同时保持正常输入的功能。*可解释性*：为了解决间接和多轮交互的复杂性，创新的防御策略需要关注LLM的内在特性。通过分析有害提示在模型神经元中的表示，我们可以识别决策边界，并为LLM及其代理开发一个可解释的框架，从而防止越狱。

## 4\. 模型缺陷带来的风险

决策模块是代理的关键组件，其中一个或多个LLM负责理解、分析和规划。本节分析了模型本身固有的局限性和问题所带来的风险，例如偏见、幻觉等问题，这些问题可能影响模型的可靠性。对于每个风险，我们首先介绍它们是什么，然后总结它们在LLM代理的六个关键特性中的技术进展，最后分析它们的局限性。

<svg class="ltx_picture ltx_centering" height="684.23" id="S4.F9.pic1" overflow="visible" version="1.1" width="742.49"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,684.23) matrix(1 0 0 -1 0 0) translate(21.34,0) translate(0,340.75)"><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -16.45 -7.61)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.865)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Threats</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 98.12 -322.57)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.865)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Backdoor</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 202.99 -334.39)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.865)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Tool Invocation</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 272.65 -335.87)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="163.37">ASB (Zhang et al., [2024d](https://arxiv.org/html/2411.09523v1#bib.bib317))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 225.73 -310.71)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.81)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.12)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">LLM</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 231.19 -312.24)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="246.56">BackdoorLLM (Li et al., [2024c](https://arxiv.org/html/2411.09523v1#bib.bib146))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 109 -251.65)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.81)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.12)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Bias</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 205.45 -272.34)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 12.805)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Multi-modal Inputs and Outputs</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 256.16 -282.56)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 32.715)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 27.87)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 44.64 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 23.025)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 18.19)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 5.68 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="253.19">VisoGender (Hall et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib92))</foreignobject></g></g></g></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 25.45)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="264.56">VLStereoSet (Zhou et al., [2022](https://arxiv.org/html/2411.09523v1#bib.bib335))</foreignobject></g></g></g></g></g> <g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 35.13)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="353.83">SocialCounterfactuals (Howard et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib102))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 225.73 -188.66)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.81)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.12)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">LLM</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 313.45 -235.96)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.865)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">AI Generated Data</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 348.47 -237.44)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="247.95">TrustGPT (Huang et al., [2023c](https://arxiv.org/html/2411.09523v1#bib.bib111))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 283.05 -213.28)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 12.805)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Data from LGBTQ+ Community</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 340.67 -213.82)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="263.54">WinoQueer (Felkner et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib78))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 303.67 -188.72)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.865)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Data from Social Media</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 339.95 -190.2)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="264.99">RedditBias (Barikeri et al., [2021](https://arxiv.org/html/2411.09523v1#bib.bib20))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 313.35 -149.29)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.81)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.12)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Human-Write Data</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 346.15 -165.36)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 42.405)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 37.56)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 32.715)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 27.87)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 23.025)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 18.19)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="252.59">StereoSet (Nadeem et al., [2020](https://arxiv.org/html/2411.09523v1#bib.bib188))</foreignobject></g></g></g></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 25.45)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 2.48 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="248.17">CrowS-Pairs (Nangia et al., [2020](https://arxiv.org/html/2411.09523v1#bib.bib191))</foreignobject></g></g></g></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 35.13)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 8.32 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="235.95">Grep-BiasIR (Krieg et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib130))</foreignobject></g></g></g></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 44.82)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 20.88 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="210.83">BBQ (Parrish et al., [2022](https://arxiv.org/html/2411.09523v1#bib.bib202))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 306.41 -110.92)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 12.805)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Non-English Language</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 360.57 -111.46)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="223.74">Sun et al. (Sun et al., [2023b](https://arxiv.org/html/2411.09523v1#bib.bib245))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 89.73 -7.61)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.865)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Hallucination</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 201.38 -86.35)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.865)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Tool Invocation</text></g></g></g></g> <g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 213.47 -87.83)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="281.46">ToolBeHonest (Zhang et al., [2024c](https://arxiv.org/html/2411.09523v1#bib.bib325))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 191.79 -61.71)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 12.805)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Memory Mechanism</text></g></g></g></g> <g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 237.47 -62.24)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="233.73">HalluDial (Luo et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib178))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 185.75 -25.33)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.865)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Multi-round Interaction</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 209.64 -36.5)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 32.715)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 27.87)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 26.22 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 23.025)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 18.19)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="236.95">HalluDial (Luo et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib178))</foreignobject></g></g></g></g></g> <g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 25.45)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 2.29 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="232.38">DiaHalu (Chen et al., [2024a](https://arxiv.org/html/2411.09523v1#bib.bib45))</foreignobject></g></g></g></g></g> <g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 35.13)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="289.39">VisDiaHalBench (Cao et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib30))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 194.45 9.16)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 12.805)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Multi-source Inputs</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 237.47 8.62)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="233.72">HalluDial (Luo et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib178))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 205.45 44.59)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 12.805)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Multi-modal Inputs and Outputs</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 171.16 34.37)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 32.715)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 27.87)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 32.34 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 23.025)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 18.19)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="459.13">AMBER (Wang et al., [2023b](https://arxiv.org/html/2411.09523v1#bib.bib268)) Reefknot(Zheng et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib332))</foreignobject></g></g></g></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 25.45)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 78.44 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="302.25">AUTOHALLUSION (Wu et al., [2024a](https://arxiv.org/html/2411.09523v1#bib.bib285))</foreignobject></g></g></g></g></g> <g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 35.13)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="524.08">THRONE (Kaul et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib123)) VisDiaHalBench(Cao et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib30))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 225.73 130.23)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.81)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.12)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">LLM</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 313.45 82.94)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.865)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">General Evaluation</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 322.39 76.61)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 23.025)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 18.19)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 37.09 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="225.92">HaluEval (Li et al., [2023a](https://arxiv.org/html/2411.09523v1#bib.bib139))</foreignobject></g></g></g></g></g> <g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 25.45)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="300.64">HypoTermQA (Uluoglakci and Temizel, [2024](https://arxiv.org/html/2411.09523v1#bib.bib255))</foreignobject></g></g></g></g> <g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 313.17 113.49)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 12.805)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Medical Knowledge</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 347.31 108.11)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 23.025)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 18.19)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="250.27">Wu et al. (Wu et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib283))</foreignobject></g></g></g></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 25.45)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 11.23 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="227.8">CMHE (Dou et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib69))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 300.52 139.08)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 12.805)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Mathematical Knowledge</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 347.2 138.54)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="250.49">UMWP (Sun et al., [2024a](https://arxiv.org/html/2411.09523v1#bib.bib247))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 306.41 164.67)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 12.805)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Non-English Language</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 352.32 159.29)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 23.025)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 18.19)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="240.25">UHGEval (Liang et al., [2023a](https://arxiv.org/html/2411.09523v1#bib.bib153))</foreignobject></g></g></g></g></g> <g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 25.45)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 8.85 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="222.56">HalOmi (Dale et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib56))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 92.33 223.73)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 12.805)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Jailbreaking</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 166.08 196.17)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 12.805)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Multi-modal Inputs and Outputs</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 221.23 190.79)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 23.025)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 18.19)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="266.2">JailBreakV-28K (Luo et al., [2024a](https://arxiv.org/html/2411.09523v1#bib.bib177))</foreignobject></g></g></g></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 25.45)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 11.26 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="243.95">MM-SafetyBench (Liu et al., [2023f](https://arxiv.org/html/2411.09523v1#bib.bib171))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 185.75 224.67)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.865)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Multi-round Interaction</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 230.58 223.19)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="248.58">RED-EVAL (Bhardwaj and Poria, [2023](https://arxiv.org/html/2411.09523v1#bib.bib21))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 225.73 256.22)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 11.81)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.12)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">LLM</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 -7.6 245.69)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 32.715)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 27.87)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 83.34 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 23.025)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 18.19)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 20.81 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="516.1">JailbreakEval (Ran et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib218)) AttackEval (Jin et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib121))</foreignobject></g></g></g></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 25.45)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="557.72">Chu et al. (Chu et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib49)) JailbreakBench (Chao et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib39))</foreignobject></g></g></g></g></g> <g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 35.13)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="724.13">Bag of Tricks (Xu et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib291)) h4rm3l (Doumbouya et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib70)) AdvBench (Zou et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib338))</foreignobject></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 86.6 306.4)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 12.805)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Goal Hijacking</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 194.45 306.4)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 12.805)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.23)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><text transform="matrix(1 0 0 -1 0 0)">Multi-source Inputs</text></g></g></g></g><g fill="#000000" stroke="#000000" transform="matrix(1.0 0.0 0.0 1.0 217.25 291.34)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 42.405)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 37.56)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 32.715)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 27.87)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 23.025)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 18.19)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><g class="ltx_tikzmatrix" transform="matrix(1 0 0 -1 0 13.345)"><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 6.62)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="8.5" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="16.41">\pgfmathresultpt</foreignobject></g></g><g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 15.76)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 0 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="274.15">INJECAGENT (Zhan et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib315))</foreignobject></g></g></g></g></g> <g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 25.45)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 15.22 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="243.71">Liu et al. (Liu et al., [2024a](https://arxiv.org/html/2411.09523v1#bib.bib172))</foreignobject></g></g></g></g></g> <g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 35.13)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 4.47 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="265.47">InjectBench (Kong, [2024](https://arxiv.org/html/2411.09523v1#bib.bib129))</foreignobject></g></g></g></g></g> <g class="ltx_tikzmatrix_row" transform="matrix(1 0 0 1 0 44.82)"><g class="ltx_tikzmatrix_col ltx_nopad_l ltx_nopad_r" transform="matrix(1 0 0 -1 17.71 0)"><foreignobject height="9.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="238.73">BIPIA (Yi et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib299))</foreignobject></g></g></g></g></g></svg>

图 9\. 按威胁类型和关键特性分类的基准概览。

### 4.1\. 幻觉

尽管大语言模型（LLM）在一系列下游任务中展示了显著的能力，但它们由于倾向于表现出幻觉现象，仍然引发了重大关注。这些幻觉表现为与用户输入偏离的内容、与先前生成的上下文相矛盾或与既定的世界知识不符（Zhang et al., [2023a](https://arxiv.org/html/2411.09523v1#bib.bib326)）。幻觉现象影响了不同时代的语言模型（从传统深度学习（Lee et al., [2018](https://arxiv.org/html/2411.09523v1#bib.bib134); Rohrbach et al., [2018](https://arxiv.org/html/2411.09523v1#bib.bib221)）到基于变换器的大型模型时代（Ji et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib118); Zhang et al., [2023a](https://arxiv.org/html/2411.09523v1#bib.bib326)）），并且影响了多种模态（包括LLMs（Zhang et al., [2023c](https://arxiv.org/html/2411.09523v1#bib.bib320)）和多模态大语言模型（MLLMs）（Yu et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib305)））。为了解决这一问题，许多研究致力于理解幻觉为何发生以及如何评估和消除它们。在基于LLM的代理系统中，幻觉的产生主要涉及四个关键特征：基于LLM的控制器、多模态输入、多源输入和记忆机制。接下来，我们将回顾幻觉方面的最新进展。

#### 4.1.1\. 幻觉的可能原因

幻觉的产生原因有很多，主要可以分为三种类型。(i) 不平衡和噪声较大的训练数据集。虽然大规模的训练数据集包含了大量有价值的信息，但不可避免地会引入错误、过时的数据或不平衡的数据分布（Zhang 等，[2023a](https://arxiv.org/html/2411.09523v1#bib.bib326)；Ji 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib118)）。错误和过时的数据可能导致模型学习到不正确的知识（Wang 等，[2023e](https://arxiv.org/html/2411.09523v1#bib.bib270)；Zhang 等，[2023c](https://arxiv.org/html/2411.09523v1#bib.bib320)；Yu 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib305)；Pearce 等，[2022](https://arxiv.org/html/2411.09523v1#bib.bib205)），从而产生与事实信息相矛盾的幻觉。此外，不平衡的数据集可能导致模型偏向于生成在数据集中出现频率更高的对象或对象组合（具有更高的边际概率（Van der Poel 等，[2022](https://arxiv.org/html/2411.09523v1#bib.bib259)））（Biten 等，[2022](https://arxiv.org/html/2411.09523v1#bib.bib23)；Zhou 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib336)；Li 等，[2023c](https://arxiv.org/html/2411.09523v1#bib.bib144)；Cui 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib53)；Leng 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib136)），而不是回应用户提供的提示或参考。(ii) 不完全学习。LLMs 的训练策略是最小化 KL 散度，但这种方法并未有效学习训练数据集的分布（Lin 等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib157)；Peng 等，[2023a](https://arxiv.org/html/2411.09523v1#bib.bib206)），导致模型学习到更多的语言知识（Chuang 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib50)）和来自统计信息的虚假相关性（Agarwal 等，[2020](https://arxiv.org/html/2411.09523v1#bib.bib8)；Kim 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib125)；Yu 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib305)）。这使得 LLMs 更加依赖语言先验（Rohrbach 等，[2018](https://arxiv.org/html/2411.09523v1#bib.bib221)；Liu 等，[2023b](https://arxiv.org/html/2411.09523v1#bib.bib161)；Favero 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib76)），而非识别和生成从训练语料中提取的真实世界事实。此外，对于 MLLMs，还存在跨模态表示对齐不理想的问题（Jiang 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib119)）。(iii) 错误的解码过程。LLMs 的解码策略，如 top-k 采样，本质上引入了随机性，这可能促使幻觉的发生（Dziri 等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib73)；Zhou 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib336)）。此外，这还可能导致雪球效应，进而在后续生成的内容中产生更多的幻觉（Zhang 等，[2023c](https://arxiv.org/html/2411.09523v1#bib.bib320)）。

基于LLM的智能体将LLM应用于特定领域，继承了LLM在下游任务中的幻觉因素。如前所述，由于训练数据集的不平衡和噪声，幻觉会在不同的应用场景中对智能体产生特定的影响。例如，由于数据不平衡，GPT-4在遇到东亚国家或非英语语境时更容易出现幻觉（Cui等， [2023](https://arxiv.org/html/2411.09523v1#bib.bib53)）。当应用于东亚城市导航智能体（如PReP（Zeng等， [2024a](https://arxiv.org/html/2411.09523v1#bib.bib311)，一款近期为北京和上海开发的基于LLM的导航智能体））时，更容易产生幻觉。此外，受噪声数据的影响，LLM可能会学习到训练数据中存在的可利用漏洞代码（Pearce等， [2022](https://arxiv.org/html/2411.09523v1#bib.bib205)）。当应用于软件开发（例如ChatDev（Qian等， [2023b](https://arxiv.org/html/2411.09523v1#bib.bib213)））时，这可能导致智能体生成类似的不安全代码。此外，当LLM用于专业领域时，如果数据集中包含的相关知识较少或没有，可能会由于知识空白而发生幻觉。例如，当LLM应用于Minecraft中的任务时，它们可能会提供无法在游戏内完成的指令（Wang等， [2023d](https://arxiv.org/html/2411.09523v1#bib.bib264)）。

#### 4.1.2\. 幻觉解决方案的技术进展

评估大语言模型中的幻觉。研究人员已经开发了针对不同模态（如语言（Zhou 等人，[2020](https://arxiv.org/html/2411.09523v1#bib.bib334)；Lin 等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib157)）或多模态（Rohrbach 等人，[2018](https://arxiv.org/html/2411.09523v1#bib.bib221)；Cui 等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib53)））以及幻觉类型（例如，事实错误（Lin 等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib157)；Zhang 等人，[2023b](https://arxiv.org/html/2411.09523v1#bib.bib327)；Cui 等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib53)），与用户参考相矛盾（Lu 等人，[2024b](https://arxiv.org/html/2411.09523v1#bib.bib175)），偏见（Cui 等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib53)）等）量身定制的不同基准数据集，用于评估模型中的幻觉程度，采用手动（Santhanam 等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib224)），自动化度量（Rohrbach 等人，[2018](https://arxiv.org/html/2411.09523v1#bib.bib221)；Li 等人，[2023c](https://arxiv.org/html/2411.09523v1#bib.bib144)），或检测模型（Li 等人，[2023c](https://arxiv.org/html/2411.09523v1#bib.bib144)）。图 [9](https://arxiv.org/html/2411.09523v1#S4.F9 "Figure 9 ‣ 4\. Risks from Model Flaws ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents") 显示了基于涉及的关键特征的幻觉评估基准。

通过大语言模型（LLM）精细化减少幻觉。为了减少幻觉，研究人员提出了针对前面提到的幻觉潜在原因的各种方法。为了应对数据集中的偏差，一种方法是引入新的合成数据，以改善模型对虚假相关性的学习（Agarwal et al., [2020](https://arxiv.org/html/2411.09523v1#bib.bib8); Rohrbach et al., [2018](https://arxiv.org/html/2411.09523v1#bib.bib221); Zhao et al., [2023b](https://arxiv.org/html/2411.09523v1#bib.bib331); Kim et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib125)）。为了应对数据集中的错误，可以通过幻觉检测进行数据清理（Yu et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib305)）。为了弥补缺乏专业领域信息的问题，微调是一种简单有效的方法。例如，在ODYSSEY（Liu et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib167)）中，作者在Minecraft Wiki上对LLaMA-3进行了微调，使其能够更好地解决与Minecraft相关的问题。此外，关于不完整学习，研究人员建议改进损失函数（Van der Poel et al., [2022](https://arxiv.org/html/2411.09523v1#bib.bib259); Zha et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib314); Sun et al., [2023a](https://arxiv.org/html/2411.09523v1#bib.bib244); Jiang et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib119)）和指令微调（Lu et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib175); Liu et al., [2023b](https://arxiv.org/html/2411.09523v1#bib.bib161); Qi et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib210)）。对于错误解码，研究人员提出了新的解码策略，以增强模型对上下文的理解，并减少幻觉，而无需重新训练模型（Shi et al., [2023b](https://arxiv.org/html/2411.09523v1#bib.bib238); Chuang et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib50); Leng et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib136); Van der Poel et al., [2022](https://arxiv.org/html/2411.09523v1#bib.bib259); Huang et al., [2024a](https://arxiv.org/html/2411.09523v1#bib.bib106)）。

通过记忆机制减少幻觉。为了应对数据集中的过时信息，可以引入外部知识（Peng et al., [2023a](https://arxiv.org/html/2411.09523v1#bib.bib206)）。检索增强生成（RAG）是一种常用方法，用于应对LLM训练集中数据过时和缺乏专业领域信息的问题。例如，在WebGPT（Nakano et al., [2021](https://arxiv.org/html/2411.09523v1#bib.bib190)）中，代理每次都向Bing搜索引擎发起查询，并总结搜索结果。

通过多源输入减少幻觉。（i）多代理协作。不同的代理相互交叉验证和检查，这在许多代理框架中用于减少幻觉的影响。例如，在ChatDev（Qian et al., [2023b](https://arxiv.org/html/2411.09523v1#bib.bib213)）中，助手与导师进行多轮沟通，以寻求更详细的建议并减少幻觉。（ii）系统提示标准化。通过将预定义的输出模板纳入大型模型的提示中，可以在一定程度上标准化LLM的输出，从而降低幻觉的可能性（Hong et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib99)）。例如，在GameGPT（Chen et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib43)）中，作者为每种游戏类型提供了标准化的规划模板，指导游戏开发经理填写相关信息以减少幻觉。

#### 4.1.3. 局限性讨论

目前解决幻觉问题的方案存在四个局限性。（i）缺乏理论分析框架。目前，幻觉缺乏数学定义，评估和缓解幻觉的方法主要通过实验验证，缺乏理论分析。（ii）多模态LLM的基准不足。目前评估LLM幻觉的基准主要集中在图像和文本模态。然而，LLM也已应用于语音（Wu et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib282)）和视频（Maaz et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib179)）模态，但在这些领域中尚未建立幻觉评估基准。（iii）缺乏针对专业领域的评估方法。目前LLM的幻觉基准主要涉及一般任务（例如，图像定位（Lu et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib175)），常识推理（Lin et al., [2021](https://arxiv.org/html/2411.09523v1#bib.bib157)））。然而，在这些任务中，幻觉的最小化并不意味着专业领域中也没有幻觉（例如，游戏代理、软件开发）。为专业领域设计幻觉评估方法仍然是一个未解决的问题。（iv）模仿虚假信息。通过记忆机制引入额外知识可以在一定程度上减少由于过时训练数据引起的幻觉。然而，它也可能由于额外知识本身的错误而导致新的幻觉（Nakano et al., [2021](https://arxiv.org/html/2411.09523v1#bib.bib190)）。

### 4.2. 记忆

LLM是基于似然性的生成模型，经过训练以最大化观察到的样本（训练数据）$\bm{\theta}^{*}=\mathop{\arg\max}_{\bm{\theta}}p_{\bm{\theta}}(\bm{x})$的似然性。在部署时，生成过程只是从已学习的概率分布$p_{\bm{\theta}^{*}}(\bm{x})$中进行采样。直观地讲，对于某个样本$\bm{x}$，如果学习到的模型对其过拟合，即$p_{\bm{\theta}^{*}}(\bm{x})$过大，LLM可能会过度生成$\bm{x}$，这就带来了训练数据记忆的问题，这可以称为参数记忆（Zeng等，[2024c](https://arxiv.org/html/2411.09523v1#bib.bib313)）。

在基于LLM的智能体中，将会有来自多源输入模块的重要信息输入到基于LLM的控制器。这些机制引发了另一种类型的记忆现象，它存在于交互的特定上下文中，可以称之为上下文记忆（Zeng等，[2024c](https://arxiv.org/html/2411.09523v1#bib.bib313); Bagdasaryan等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib17)），即智能体能够记住与特定上下文相关的有价值信息。

记忆并不一定是基于LLM的智能体的威胁，因为研究表明，记忆对泛化是必要的（Feldman，[2020](https://arxiv.org/html/2411.09523v1#bib.bib77); Chatterjee，[2018](https://arxiv.org/html/2411.09523v1#bib.bib41)）。然而，过度的记忆会对数据隐私、模型稳定性和泛化性能产生影响（van den Burg 和 Williams，[2021](https://arxiv.org/html/2411.09523v1#bib.bib258)）。在基于LLM的智能体的背景下，记忆问题的发展主要涉及基于LLM的控制器。在本节中，我们回顾了属于更概念性范围的记忆研究，并将基于记忆的隐私对抗攻击和防御问题留到第[5.2](https://arxiv.org/html/2411.09523v1#S5.SS2 "5.2. Privacy Leakage ‣ 5. Risks from Input-Model Interaction ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")节。

#### 4.2.1. 技术进展

文献从多个角度分析了记忆。普遍认为并已证明，记忆是实现泛化的必要条件（Chatterjee，[2018](https://arxiv.org/html/2411.09523v1#bib.bib41); Feldman，[2020](https://arxiv.org/html/2411.09523v1#bib.bib77)）。当模型记住整个训练集时，LLM 的评估指标，如困惑度，在训练数据上几乎完美。另一方面，也有研究致力于理解记忆是如何发生的。经验研究（Carlini 等人，[2023b](https://arxiv.org/html/2411.09523v1#bib.bib33); Lee 等人，[2022](https://arxiv.org/html/2411.09523v1#bib.bib135)）发现，训练集中的重复、模型容量、提示长度等，都是记忆的影响因素。Biderman 等人（Biderman 等人，[2024](https://arxiv.org/html/2411.09523v1#bib.bib22)）提出，通过外推低计算试验运行的记忆行为，预测大模型在完全训练前将记住哪些序列。Van den Burg 等人（van den Burg 和 Williams，[2021](https://arxiv.org/html/2411.09523v1#bib.bib258)）为生成模型提供了一种记忆度量方法，即当某个样本参与训练时，生成模型的改进概率。

#### 4.2.2\. 局限性的讨论

记忆和泛化之间的平衡是一个具有挑战性的问题。关于记忆的经验因素可以在平均水平上减少记忆，但不适用于个别实例。基于小型模型或中间检查点的对大模型行为的预测是不准确且不可靠的。最后，有害的记忆依赖于安全需求，且缺乏从理论分析结果到实际风险的推断。

### 4.3\. 偏差与公平性

偏差指的是模型在决策或生成过程中倾向于偏袒特定群体的现象。这种现象在人工智能模型中相当普遍。例如，美国法院使用的模型通常预测非裔美国人更高的犯罪行为概率（Mehrabi 等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib183)）。这些偏见的预测源自数据或算法中的隐性或被忽视的偏见（Gallegos 等人，[2024](https://arxiv.org/html/2411.09523v1#bib.bib82)）。在基于大型语言模型（LLM）的代理中，偏差问题的发展主要涉及两个关键特征：基于 LLM 的控制器，多模态输入和输出。接下来，我们将报告在偏差问题上的技术进展。

#### 4.3.1\. 偏差的原因

对于传统机器学习模型，偏见主要来源于两个方面：数据偏见和算法缺陷。数据偏见种类繁多，可能包括歧视性信息、抽样偏差、测量偏差等（Suresh和Guttag，[2019](https://arxiv.org/html/2411.09523v1#bib.bib249)；Buolamwini和Gebru，[2018](https://arxiv.org/html/2411.09523v1#bib.bib27)；Zhang等，[2016](https://arxiv.org/html/2411.09523v1#bib.bib319)）。即使数据没有偏见，模型仍然可能由于算法因素表现出偏见。算法偏见源自设计选择，如选择优化函数和正则化技术（Danks和London，[2017](https://arxiv.org/html/2411.09523v1#bib.bib58)；Baeza-Yates，[2018](https://arxiv.org/html/2411.09523v1#bib.bib15)）。缓解偏见的策略包括数据清洗、调整模型架构和其他技术（d’Alessandro等，[2017](https://arxiv.org/html/2411.09523v1#bib.bib57)；Ustun等，[2019](https://arxiv.org/html/2411.09523v1#bib.bib257)）。

偏见在LLM中同样存在。与传统模型相比，LLM中的偏见更深且更广。传统决策模型仅限于在固定范围内做出决策，因此其偏见仅局限于特定范围。相比之下，LLM执行生成任务，这使得其输出可能包含各种类型的偏见。由于缺乏固定的输出格式，这些偏见可能更加微妙，比决策模型中的偏见更难以察觉。当前的研究发现，LLM在性别、种族和政治观点等领域存在偏见（Tang等，[2024c](https://arxiv.org/html/2411.09523v1#bib.bib251)；Feng等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib79)），不同的LLM展示了不同程度和类型的偏见。

对于基于LLM的智能体，问题变得更加复杂。与LLM不同，基于LLM的智能体能够处理多模态信息，包括文本、图像和语音，从而导致偏见表现得更加复杂。当前的研究表明，偏见和歧视在视觉语言模型（VLM）中也存在，影响到视觉问答（VQA）和文本到图像生成等任务（D’Incà等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib65)；Howard等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib102)）。因此，跨多个模态的综合评估对于准确判断至关重要。此外，在基于LLM的智能体中引入额外组件可能会引发新的偏见问题，需谨慎考虑。

从偏见的成因分析来看，基于LLM的代理比传统模型表现出更为明显的偏见问题，因为它们的训练数据集更大，结构更复杂。从训练数据的角度来看，LLM主要是在来自在线平台的数据上进行训练，这些数据在训练前通常没有经过彻底审查，导致包含歧视性样本。此外，偏见语句在训练数据中的分布也不均匀。从模型结构的角度来看，LLM的参数数量远大于传统模型，架构也更加复杂。这种复杂性使得在模型训练过程中确保公平性变得更加困难。

#### 4.3.2. 技术进展

为了解决和减轻LLM中的偏见问题，当前的努力主要集中在三个方面：（i）开发合理的偏见评估指标，（ii）构建全面的偏见评估数据集，以及（iii）采用各种技术来减轻模型偏见。

评估指标。LLM中的偏见评估指标可以根据其评估过程中依赖的内容进行分类。包括基于嵌入的指标，利用上下文句子嵌入，基于概率的指标，使用模型分配的概率，以及基于输出的指标，分析模型的输出。

基于嵌入的指标计算中性词（例如职业）和与身份相关的词（例如性别代词）在向量空间中的距离。Caliskan等人（Caliskan等，[2017](https://arxiv.org/html/2411.09523v1#bib.bib29)）提出了使用静态词嵌入的词嵌入关联测试（WEAT），以评估NLP任务中的偏见。后续研究使用整个句子中的词嵌入（May等，[2019](https://arxiv.org/html/2411.09523v1#bib.bib181)；Guo和Caliskan，[2021](https://arxiv.org/html/2411.09523v1#bib.bib91)），或计算词级偏见的归一化总和（Dolci等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib66)）。

基于概率的指标侧重于掩蔽的词汇。一些方法使用模板来创建句子，它们掩蔽包含社会群体的部分，并根据模型分配的概率评估偏见（Webster等，[2020](https://arxiv.org/html/2411.09523v1#bib.bib276)）。其他方法则逐个掩蔽句子中的每个词汇，以测试模型生成歧视性内容的能力，称为伪对数似然方法（Kaneko和Bollegala，[2022](https://arxiv.org/html/2411.09523v1#bib.bib122)；Nadeem等，[2020](https://arxiv.org/html/2411.09523v1#bib.bib188)）。

基于输出的指标包括基于分布的方法和基于分类器的方法。基于分布的指标通过检测模型输出中不同群体之间的差异来衡量偏见（Rajpurkar等人，[2016](https://arxiv.org/html/2411.09523v1#bib.bib216); Liang等人，[2022](https://arxiv.org/html/2411.09523v1#bib.bib150)）。基于分类器的指标使用分类器评估输出的有毒性；与特定群体相关的更高有毒性表明歧视和偏见（Gehman等人，[2020](https://arxiv.org/html/2411.09523v1#bib.bib84); Fleisig等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib81); Smith等人，[2022](https://arxiv.org/html/2411.09523v1#bib.bib240)）。

评估数据集。评估数据集由需要完成的大量文本组成。通过评估LLM在文本完成过程中对不同社会群体所表现出的偏见，可以确定模型偏见的程度。像StereoSet（Nadeem等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib189)）这样的掩码令牌数据集包含带有空白的句子，语言模型必须填充这些空白。然后，模型做出的选择被用来评估其偏见。相反，像CrowS-Pairs（Nangia等人，[2020](https://arxiv.org/html/2411.09523v1#bib.bib191)）和WinoQueer（Felkner等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib78)）这样的未掩码句子数据集则向模型提供句子对，并询问哪一个更可能。

其他方法使用句子完成而非单词选择。例如，TrustGPT（Huang等人，[2023c](https://arxiv.org/html/2411.09523v1#bib.bib111)）使用来自社会规范的有毒提示模板来检查语言模型中的有毒性。然后，它通过测量不同群体的有毒值来量化模型偏见。另一方面，BBQ（Parrish等人，[2022](https://arxiv.org/html/2411.09523v1#bib.bib202)）和Grep-BiasIR（Krieg等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib130)）采用问答格式进行评估。

偏见缓解。在识别模型中存在偏见后，寻求缓解偏见的方法是很自然的。基于大语言模型（LLM）的工作流程，目前的偏见缓解技术可以分为两种类型：训练阶段方法和推理阶段方法。

在训练阶段，主要的方法包括清理训练数据、修改模型架构和调整训练策略。基于数据的方法旨在消除训练数据中的偏差。数据增强技术通过添加样本来扩展分布，以便涵盖被低估的社会群体（Ghanbarzadeh 等， [2023](https://arxiv.org/html/2411.09523v1#bib.bib85)；Zayed 等， [2023](https://arxiv.org/html/2411.09523v1#bib.bib310)）。数据过滤方法从训练数据中移除明显偏见和有害的文本（Garimella 等， [2022](https://arxiv.org/html/2411.09523v1#bib.bib83)）。架构修改主要通过插入新的组件，如适配器模型和门控模型，来改进大规模语言模型（LLM）的编码器，以减轻偏差（Lauscher 等， [2021](https://arxiv.org/html/2411.09523v1#bib.bib132)；Han 等， [2022a](https://arxiv.org/html/2411.09523v1#bib.bib93)）。训练策略的调整主要通过添加正则化项来改善损失函数，这些正则化项用于衡量偏差。一种常见的方法是基于人类反馈的强化学习（RLHF）（Bai 等， [2022](https://arxiv.org/html/2411.09523v1#bib.bib18)），将LLM输出与人类判断对齐。其他方法包括减少不同群体的嵌入分布差异（Yang 等， [2023b](https://arxiv.org/html/2411.09523v1#bib.bib292)）和使用对比学习（Li 等， [2023b](https://arxiv.org/html/2411.09523v1#bib.bib143)）以及对抗学习（Han 等， [2022b](https://arxiv.org/html/2411.09523v1#bib.bib94)）等技术来指导模型。为了避免修改损失函数对模型性能的影响，一些方法还在训练过程中冻结部分模型参数（Ranaldi 等， [2023](https://arxiv.org/html/2411.09523v1#bib.bib219)；Yu 等， [2023a](https://arxiv.org/html/2411.09523v1#bib.bib301)）。

在推理阶段，方法可以分为三种类型：预处理、处理中和后处理。指令调优发生在预处理阶段，涉及在用户提示中添加或修改指令，以引导LLM远离偏见内容（Venkit et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib261); Fatemi et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib75)）。在处理中阶段，一些方法调整解码算法以确保输出的公平性（Saunders et al., [2022](https://arxiv.org/html/2411.09523v1#bib.bib225)），而另一些方法则改变从中采样的令牌分布，以便以更高的概率采样出较少偏见的输出（Chung et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib51)）。后处理偏见缓解是指在模型输出后进行的处理，以消除偏见。这主要通过重写输出实现。最简单的方法是使用关键词替换来消除歧视性术语（Dhingra et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib64)）。其他方法则使用专门的机器翻译模型来去除句子中的偏见（Amrhein et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib12)）。

#### 4.3.3. 局限性讨论

尽管对LLM中的偏见进行了广泛的研究，但许多问题仍然存在。首先，大多数研究基于传统的语言模型，这引发了人们对其是否适用于LLM的担忧。例如，Cabello等人（Cabello et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib28)）认为，文本嵌入与LLM输出中的偏见之间不一定存在直接的相关性。类似地，Delobelle等人（Delobelle et al., [2022](https://arxiv.org/html/2411.09523v1#bib.bib59)）建议，基于概率的度量可能与下游任务中观察到的偏见相关性较弱。这些发现对基于嵌入和基于概率的度量在评估LLM偏见方面的有效性提出了质疑。

其次，与传统模型相比，LLM具有更复杂的结构，且训练所需的参数数量大大增加。这种复杂性意味着，在LLM中缓解偏见可能会对其性能产生显著影响。例如，涉及微调的偏见缓解方法通常使用小型数据集，这可能导致LLM在最初通过大数据集训练时发生灾难性遗忘。

此外，基于LLM的代理的广泛适用性凸显了当前研究的局限性。尽管在文本到图像模型中已有一些努力来缓解偏见，但目前没有标准化的数据集或度量标准来评估生成图像中的偏见。此外，大多数偏见研究主要集中在英语上，其他语言中的偏见研究则严重匮乏。

作为一个系统，基于LLM的智能体包括多个组件，包括LLM本身、外部工具、记忆模块等。然而，目前的研究很少从系统角度考虑偏见对整个智能体的影响。这个领域需要进一步的研究。

## 5\. 输入-模型交互的风险

本节分析了输入数据与模型之间的相互影响，包括后门攻击和隐私泄漏。对于每种风险，我们首先介绍它们是什么，然后总结它们在基于LLM的智能体的六个关键特征中的技术进展，最后分析它们的局限性。

### 5.1\. 后门

后门攻击在训练阶段嵌入恶意利用程序，随后在测试阶段通过触发器的存在被调用（Goldblum等人，[2020](https://arxiv.org/html/2411.09523v1#bib.bib86)）。这些攻击可以分为基于数据中毒的攻击（Aghakhani等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib9)），其中对手只能操控训练数据，以及基于模型中毒的攻击（Shen等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib232)），其中整个训练过程都受到破坏。后门攻击的漏洞从传统的机器学习模型扩展到先进的AI系统，包括基于LLM的AI智能体。如图[10](https://arxiv.org/html/2411.09523v1#S5.F10 "图10 ‣ 5.1\. 后门 ‣ 5\. 输入-模型交互的风险 ‣ 风险导航：基于LLM的智能体中的安全、隐私与伦理威胁调查")所示，在基于LLM的智能体的背景下，后门攻击的发展主要涉及四个关键特征：基于LLM的控制器、多模态交互、记忆机制和工具调用。接下来，我们将回顾攻击策略和防御机制的最新进展。

![参考说明](img/cb4be92b780bac489fd7059183e08c6d.png)

图10\. 针对基于LLM的智能体的后门攻击可能涉及四个关键特征（以红色感叹号标示），导致攻击者针对性的行为。

#### 5.1.1\. 技术进展

攻击视角。如[第2节](https://arxiv.org/html/2411.09523v1#S2 "2\. 基于LLM的智能体 ‣ 风险导航：基于LLM的智能体中的安全、隐私与伦理威胁调查")中讨论的那样，基于LLM的AI智能体的几个关键特征，如工具调用、多模态交互和记忆机制，引入了新的后门漏洞。

该领域的最新研究提出了几种攻击方法，用于揭示与工具调用相关的后门漏洞。Dong 等人（Dong 等人，[2023](https://arxiv.org/html/2411.09523v1#bib.bib67)）展示了 LLM 的后门适配器（Hu 等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib103)）可能导致智能体恶意使用工具，例如发起网络钓鱼攻击。Yang 等人（Yang 等人，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib293)）研究了多种触发场景，用于激活后门生成恶意工具命令，表明触发器可以直接嵌入用户查询或环境返回的中间观察结果中。Jiao 等人（Jiao 等人，[2024](https://arxiv.org/html/2411.09523v1#bib.bib120)）提出了“词汇注入”和“场景操控”等方法，用于破坏基于 LLM 的决策系统，从而生成危险的行为。Liu 等人（Liu 等人，[2024e](https://arxiv.org/html/2411.09523v1#bib.bib159)）提出了对黑箱 LLM 上下文演示进行中毒的策略，导致其生成具有上下文相关缺陷的程序。这些程序在逻辑上看似合理，但包含缺陷，当操作智能体在交互环境中遇到特定触发器时，可能激活并引发意外行为（例如，智能体资源耗尽和用户隐私提取）。

基于 LLM 的 AI 智能体的多模态交互能力也推动了多模态后门攻击的研究。例如，Liang 等人（Liang 等人，[2023b](https://arxiv.org/html/2411.09523v1#bib.bib152)）提出了 BadCLIP 攻击，通过在嵌入空间中对齐视觉触发模式与文本目标语义，使得很难检测由后门学习引起的这些自然发生的触发模式所导致的微妙参数变化。Liu 等人（Liu 等人，[2024e](https://arxiv.org/html/2411.09523v1#bib.bib159)）开发了一种双模态激活策略，通过操控程序缺陷的生成和执行，利用文本和视觉线索的组合触发智能体的意外行为。值得注意的是，他们提出的后门攻击在实际系统中成功演示，包括 Jetbot 车辆（Jet，[2021](https://arxiv.org/html/2411.09523v1#bib.bib3)）和自动驾驶系统（Pix，[2024](https://arxiv.org/html/2411.09523v1#bib.bib4); Who，[2024](https://arxiv.org/html/2411.09523v1#bib.bib5)）。

最近的研究也突显了与基于LLM的AI代理的记忆机制相关的安全风险。Zou等人（Zou et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib339)）提出了PoisonedRAG，这是针对这些代理外部知识数据库的首次中毒攻击。他们的研究确定了成功实施知识中毒攻击的两个关键条件：检索条件和生成条件。为了满足这些条件，他们采用了优化和启发式技术，有效地危害了代理知识库的完整性。Chen等人（Chen et al., [2024d](https://arxiv.org/html/2411.09523v1#bib.bib47)）开发了AgentPoison，这是一种红队方法，旨在毒化代理的长期记忆或外部知识数据库。该方法优化了注入的文本触发器，以实现高迁移性、上下文一致性和隐蔽性，使其特别阴险且难以检测。

防御视角。针对LLM-based AI代理的后门攻击的防御方法大致可分为三类：数据集清理、输入净化和输出验证。

数据集清理涉及检测和去除指令调优数据集中的中毒样本，以创建“无后门”代理。Liu等人（Liu et al., [2023c](https://arxiv.org/html/2411.09523v1#bib.bib165)）开发了一项技术，首先识别指令调优数据集中的后门触发器，然后防止模型学习这些触发器。Liang等人（Liang et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib151)）提出了一种通过增强模型对后门触发器的抗性来隔离多模态训练数据集中的中毒样本的方法，该方法采用分阶段和定向的训练方式。

输入净化关注于在部署过程中预处理代理的输入数据，以消除嵌入的触发器。为了应对针对记忆机制的检索破坏攻击，Xiang等人（Xiang et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib287)）提出了隔离-再聚合策略。该方法首先从每个检索段落中单独获取LLM的响应，然后使用定制的基于关键词和基于解码的方法安全地聚合这些隔离的响应。

输出验证涉及审计代理的行为和响应，以确保代理与外部环境之间的交互安全可靠，特别是防范针对工具调用的攻击。多个沙箱环境，如E2B (E2B，[2021](https://arxiv.org/html/2411.09523v1#bib.bib2))、ToolSandbox (Lu等，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib174)) 和ToolEmu (Ruan等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib223))，提供了受控环境用于测试代理在各种条件和场景下的表现，包括压力测试以评估在极端或意外数据条件下的性能。根据测试结果，可以实施输出验证机制，以保护代理免受已识别的风险。

#### 5.1.2\. 局限性讨论

攻击视角。尽管针对基于LLM的AI代理的后门攻击取得了快速进展，但现有文献仍然缺乏对相关后门漏洞的全面理解和评估。例如，现有的后门攻击目标并未充分利用这些代理的功能弱点。虽然大多数攻击集中在生成恶意工具命令或操纵代理对用户的响应，但其他关键目标，如注入有缺陷的任务计划或强制选择对抗方指定的工具，仍然没有得到充分探索。此外，大多数攻击源集中在图像和文本数据上，而其他流行的模态，如音频和视频，在针对基于LLM的AI代理的后门攻击中则较少受到关注。

防御视角。当前针对基于LLM的AI代理的后门攻击的防御策略通常集中在保护单个组件，如数据集清洗和输入数据净化。然而，显然缺乏能够防范来自多个源、并针对不同目标的后门攻击的系统性防御。一个潜在的解决方案是开发跨不同代理系统组件的协作防御策略。这些策略可能包括：1）输入模块的多模态输入净化，2）决策模块的训练数据集清洗和后门模型净化，3）工具调用验证、环境反馈过滤和与外部实体交互的记忆检查，以及4）最终响应模块的输出验证。这种协作防御对保障完整的基于LLM的AI代理框架至关重要，并为后门防御的未来研究提供了有前景的方向。

### 5.2\. 隐私泄露

LLM的生成性质使其更容易受到隐私侵犯，敏感信息（例如身份、医疗记录）可能在生成的数据中不经意间或通过对抗性手段泄露，给用户隐私带来重大隐患。追踪私人信息的来源，任何外部数据流入的模块都会导致智能体违反隐私要求。首先，基于LLM的控制器面临其训练数据中的隐私泄漏威胁。此外，包含在多源输入模块中的敏感信息可能会以不受控制的方式转移到其他组件。因此，个体的私人信息可能会传播给其他用户或外部工具，破坏隐私的上下文完整性（Nissenbaum，[2004](https://arxiv.org/html/2411.09523v1#bib.bib196)）。如图[11](https://arxiv.org/html/2411.09523v1#S5.F11 "Figure 11 ‣ 5.2\. Privacy Leakage ‣ 5\. Risks from Input-Model Interaction ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")所示，在基于LLM的智能体背景下，隐私泄漏的发生主要涉及三个关键特征：基于LLM的控制器、多源输入和记忆机制。接下来，我们回顾隐私泄漏的最新进展。

![参见说明文字](img/de68ae7d078948469180679c0488d482.png)

图11. 隐私泄漏针对基于LLM的智能体可能涉及三个关键特征（用红色感叹号标注），从而导致信息泄漏。

#### 5.2.1. 技术进展

攻击视角。根据泄露信息的来源，现有的威胁可以列举为训练数据泄漏（Shi 等，[2023a](https://arxiv.org/html/2411.09523v1#bib.bib237); Ko 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib128); Carlini 等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib37); Yu 等，[2023c](https://arxiv.org/html/2411.09523v1#bib.bib306); Lukas 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib176); Kim 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib126)) 和上下文隐私泄漏（Bagdasaryan 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib17); Zeng 等，[2024c](https://arxiv.org/html/2411.09523v1#bib.bib313); Huang 等，[2023b](https://arxiv.org/html/2411.09523v1#bib.bib109); Mireshghallah 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib185)），如图[11](https://arxiv.org/html/2411.09523v1#S5.F11 "Figure 11 ‣ 5.2\. Privacy Leakage ‣ 5\. Risks from Input-Model Interaction ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")所示。

*训练数据泄露*。训练数据泄露的一个常见话题是成员推断。成员推断（Shokri等，[2017](https://arxiv.org/html/2411.09523v1#bib.bib239)）旨在揭示某些样本是否用于训练或微调模型。与为早期识别模型构建的攻击类似，针对LLMs的成员推断也基于这样的直觉：训练好的模型对训练中见过的样本的拟合度高于未见过的样本。具体来说，成员推断是通过使用训练损失或其变体（Shi等，[2023a](https://arxiv.org/html/2411.09523v1#bib.bib237)；Carlini等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib37)；Ko等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib128)）来区分成员和非成员。数据增强（Ko等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib128)）、参考影像模型（Carlini等，[2022](https://arxiv.org/html/2411.09523v1#bib.bib31)；Watson等，[2022](https://arxiv.org/html/2411.09523v1#bib.bib275)）已被证明是改善推理度量区分度的有效方法。

超越成员身份，许多研究表明，LLMs有可能生成原创的训练数据（Ko等， [2023](https://arxiv.org/html/2411.09523v1#bib.bib128)；Yu等，[2023c](https://arxiv.org/html/2411.09523v1#bib.bib306)；Nasr等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib193)）。通过大规模随机生成并随后进行成员推断，可以完全重现包含个人身份信息的训练数据。除了LLMs，多模态生成模型也已被证明易受训练数据提取的影响（Carlini等，[2023a](https://arxiv.org/html/2411.09523v1#bib.bib32)；Somepalli等，[2023a](https://arxiv.org/html/2411.09523v1#bib.bib241)，[b](https://arxiv.org/html/2411.09523v1#bib.bib242)）。训练数据提取是记忆的直接结果。它表明，包括LLMs在内的生成模型可能会无意中复制其训练数据。

如预期的那样，LLMs遭受对抗推理攻击（Kim等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib126)；Huang等，[2022b](https://arxiv.org/html/2411.09523v1#bib.bib105)；Lukas等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib176)；Kim等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib126)），这些攻击通过向LLMs提供已知信息来故意引发细粒度的私人属性。即使存在常见的缓解策略，如匿名化和差分隐私，对抗攻击仍然保持一定程度的有效性。

![参见标题](img/4a0fe99a972aa5635cd4174d4574511d.png)

图12. 四个代理的六个关键特征。

*上下文隐私泄漏。* 在基于LLM的代理中，核心LLM控制器通常配备有外部知识，以增强其完成用户指定任务的能力。典型的外部知识可以来自检索数据库（Liu, [2022](https://arxiv.org/html/2411.09523v1#bib.bib163); Chase, [2022](https://arxiv.org/html/2411.09523v1#bib.bib40); Ram et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib217)）或用户输入数据（Schick et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib226); OpenAI, [2023](https://arxiv.org/html/2411.09523v1#bib.bib198)）。上下文信息泄漏的威胁集中在提供给LLM控制器的多源上下文中的隐私信息。最新的研究都揭示了基于LLM的代理在上下文隐私保护方面的脆弱性（Huang et al., [2023b](https://arxiv.org/html/2411.09523v1#bib.bib109); Zeng et al., [2024c](https://arxiv.org/html/2411.09523v1#bib.bib313); Bagdasaryan et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib17); Mireshghallah et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib185)）。例如，Mireshghallah et al.（Mireshghallah et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib185)）展示了像GPT-4和ChatGPT这样的商业模型在上下文中泄露私人信息的情况，这在人工智能模型不会泄露的上下文中发生了39%和57%的概率。

防御视角。我们接下来讨论基于LLM的代理在训练数据泄漏和上下文隐私泄漏方面的现有对策。

*训练数据泄漏。* 根据防御方法的特点，我们将其分为差分隐私（DP）、去学习和启发式方法。

由于上述风险依赖于对某些训练样本的记忆，差分隐私（DP）（Dwork et al., [2014](https://arxiv.org/html/2411.09523v1#bib.bib72)）是一种有前景的防护方法，通过减少单个样本对训练模型的影响来防止泄漏。DP为最终模型的隐私提供了理论保证，并且由于后处理的特性，它在实现上非常灵活。关于DP-LLM的研究覆盖了LLM的整个生命周期，包括DP预处理（Yue et al., [2021](https://arxiv.org/html/2411.09523v1#bib.bib307); Hoory et al., [2021](https://arxiv.org/html/2411.09523v1#bib.bib100)），DP训练（Yu et al., [2022](https://arxiv.org/html/2411.09523v1#bib.bib302); Yue et al., [2023](https://arxiv.org/html/2411.09523v1#bib.bib308); Li et al., [2022b](https://arxiv.org/html/2411.09523v1#bib.bib142); Wang et al., [2024c](https://arxiv.org/html/2411.09523v1#bib.bib266)），DP预测（Wu et al., [2024c](https://arxiv.org/html/2411.09523v1#bib.bib284); Tang et al., [2024a](https://arxiv.org/html/2411.09523v1#bib.bib252)）等。

如欧盟的《通用数据保护条例》（GDPR）（条例，[2016](https://arxiv.org/html/2411.09523v1#bib.bib220)）等法规，赋予了个人删除其个人数据的权利。这导致了一系列关注"反学习"的研究（Nguyen等，[2022](https://arxiv.org/html/2411.09523v1#bib.bib195)），通过允许从已训练的模型中删除不需要的信息，提供了另一层保障。反学习的定义类似于差分隐私（DP），如果反学习后的模型接近于排除了需要遗忘样本的数据集所训练的模型，则被视为成功。相比之下，反学习侧重于通过额外的特定更新来高效调整已训练的模型（Graves等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib88); Bourtoule等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib24); Neel等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib194)）。LLMs的反学习（Jang等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib116); Patil等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib204)）通过最大化训练损失来限制文本反学习的可能性，并被证明在防御提取攻击方面有效。

此外，启发式方法旨在消除导致训练数据泄露的隐性因素。这些方法包括去重（Lee等，[2022](https://arxiv.org/html/2411.09523v1#bib.bib135)）、匿名化（Lison等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib158)）和采样校准（Chen等，[2024b](https://arxiv.org/html/2411.09523v1#bib.bib42); Wen等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib279)）。

*上下文隐私泄露*。许多防御性方法已被提出以减轻上下文信息泄露。通过匿名化（Huang等，[2023b](https://arxiv.org/html/2411.09523v1#bib.bib109)）、由另一个代理重写（Zeng等，[2024b](https://arxiv.org/html/2411.09523v1#bib.bib312)）以及压缩至最小信息量（Bagdasaryan等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib17)）来进行上下文敏感数据的净化，有助于减少泄露。差分隐私也被发现对于减轻上下文数据的隐私泄露有效，无论是通过生成上下文数据（Tang等，[2024b](https://arxiv.org/html/2411.09523v1#bib.bib253)）还是通过差分隐私方式聚合来自多个查询的响应（Wu等，[2024c](https://arxiv.org/html/2411.09523v1#bib.bib284)）。在更高的系统层面，Zhang等（Zhang等，[2024i](https://arxiv.org/html/2411.09523v1#bib.bib323)）提出了PrivacyAsst，该系统结合了同态加密方案和隐私保护技术，如t-接近性，应用于基于LLM的代理。

#### 5.2.2 讨论限制

数据泄露研究主要处理隐私与效用之间的权衡。现有攻击和防御方法及其在实际大规模语言模型（LLM）中的有效性和效率仍然是一个问题。像差分隐私（DP）这样的隐私保护训练方法无法扩展到高维数据和模型（Carlini 等， [2023a](https://arxiv.org/html/2411.09523v1#bib.bib32); Wang 等，[2024c](https://arxiv.org/html/2411.09523v1#bib.bib266)）。像匿名化和去重这样的策略不足以完全消除泄露（Lukas 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib176)）。一些研究（Bagdasaryan 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib17)）借助情境完整性理论分析基于LLM的代理中的私人信息流，这在单轮双向对话中具有一定前景，但在实际应用中仍受限。实际上，基于LLM的代理将处理多源输入，并与用户或外部工具进行多轮互动（见图 [3](https://arxiv.org/html/2411.09523v1#S2.F3 "Figure 3 ‣ 2\. LLM-based Agent ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")），涉及的信息及其流动要复杂得多。经验性减缓措施（Huang 等，[2023b](https://arxiv.org/html/2411.09523v1#bib.bib109); Zeng 等，[2024b](https://arxiv.org/html/2411.09523v1#bib.bib312); Bagdasaryan 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib17)）最终依赖其他代理识别、精炼或压缩私人信息，其可靠性将成为这种解决方案的主要关注点。

## 6\. 案例研究

在[2](https://arxiv.org/html/2411.09523v1#S2 "2\. 基于 LLM 的代理 ‣ 导航风险：基于 LLM 的代理在安全性、隐私性和伦理方面的威胁调查")节中，我们介绍了基于 LLM 的代理的一般框架。在接下来的各节中，我们将基于该框架分类并总结代理可能面临的潜在风险。然而，实际的代理不一定包含框架中的所有模块，而且这些模块内的设计也可能是定制化的。更重要的是，它们所面临的环境和场景有显著差异。这意味着不同的代理在实际使用中将面临特定且不同的风险。因此，本节呈现了四个不同代理的案例研究，代表以下四种情况。（i）WebGPT（Nakano 等，[2021](https://arxiv.org/html/2411.09523v1#bib.bib190)），一个完整组件的代理，专门用于一般问答任务；（ii）Voyager（Wang 等，[2023d](https://arxiv.org/html/2411.09523v1#bib.bib264)），一个用于玩游戏的具身代理；（iii）PReP（Zeng 等，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib311)），一个用于实际任务的具身代理；（iv）ChatDev（Qian 等，[2023b](https://arxiv.org/html/2411.09523v1#bib.bib213)），一个多代理框架。我们在图[12](https://arxiv.org/html/2411.09523v1#S5.F12 "图 12 ‣ 5.2.1\. 技术进展 ‣ 5.2\. 隐私泄露 ‣ 5\. 输入-模型互动的风险 ‣ 导航风险：基于 LLM 的代理在安全性、隐私性和伦理方面的威胁调查")中总结了它们的关键特性和任务。接下来，我们首先分析 WebGPT 关键特性对各种威胁的具体影响。然后，对于其余三个代理，我们识别它们与前一个代理的差异，并分析这些差异对各种威胁的具体影响。

表 2. 总结表：WebGPT 不同关键特性对各种威胁的具体影响。与独立的 LLM 相比，如果 WebGPT 的一个关键特性增加了某个威胁，我们会用红色突出显示；如果它与某个威胁无关，我们用黄色标记；如果它减少了某个威胁，我们用绿色表示。 |

| 来源 | 威胁 | 关键特性 | 具体影响 |
| --- | --- | --- | --- |
| 问题输入 | 目标劫持 | 多源输入 | WebGPT 可能会受到用户和网站的攻击。 |
| 多轮互动 | 尽管 WebGPT 可能与多个网页进行多轮互动，但网页是被动提供信息的。 |
| 工具调用 | 使用 Bing 浏览器作为工具涉及外部攻击面。 |
| 模型提取 | 多源输入 | 模型的输入与用户的输入不同，导致随机提示被一系列固定的系统提示所稀释。 |
| 工具调用 | WebGPT 的代码能力并非源自模型本身，而是来自搜索结果的总结。 |
| 模型缺陷 | 幻觉 | 基于LLM的控制器 | 错误的解码过程。 |
| 记忆机制 | 记忆机制减少了由于过时知识和虚假关联导致的幻觉威胁。 |
| 工具调用 | 微调减少了在工具使用上的知识差距。调用必应浏览器增强了搜索结果的相关性和权威性，减少了不平衡和不正确训练数据的影响。 |
| 组合威胁 | 后门 | 基于LLM的控制器 | WebGPT引入了外部训练过程。在微调GPT-3时，恶意人类演示可能被注入。 |
| 多源输入 | WebGPT可能会受到网页和用户输入的触发。 |
| 隐私泄露 | 基于LLM的控制器 | WebGPT引入了外部训练过程，可能涉及额外的隐私数据。 |
| 记忆机制 | WebGPT拥有一个庞大的外部网页数据库，其中隐藏着大量私人信息。 |
| 工具调用 | WebGPT使用必应浏览器，它可能是一个强大的隐私信息提取器。 |

### 6.1. WebGPT

ChatGPT的出现引发了全球对大型语言模型（LLMs）的关注。它流畅的对话能力和强大的问答能力得到了用户的广泛接受。问答也已成为LLMs的基本任务之一。然而，由于LLMs只能从其训练数据集中的内容中学习，它们无法回答训练数据截止时间之后发生的事件。WebGPT（Nakano等人，[2021](https://arxiv.org/html/2411.09523v1#bib.bib190)）提出了一种经典的解决方案，通过为LLMs提供使用网页浏览器的能力来解决这一问题。具体来说，对于用户的查询，它使用必应搜索来查找相关网页，引用相关信息并总结内容。WebGPT是最经典的代理之一。如图[12](https://arxiv.org/html/2411.09523v1#S5.F12 "Figure 12 ‣ 5.2.1\. Technical Progress ‣ 5.2\. Privacy Leakage ‣ 5\. Risks from Input-Model Interaction ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")所示，它具备完整代理的六大特征。我们现在将分析WebGPT框架中可能出现的潜在威胁来源及其结果。

**问题输入的风险**。对于大多数由于问题输入引发的威胁，WebGPT所面临的风险比独立的LLM更为严重，因为它涉及来自互联网的外部内容，并且不会对输入（包括用户输入和网站引用）或输出进行任何清理或格式化操作。攻击者可以诱使该代理导航到预定义的网页，从而对代理执行攻击。以目标劫持为例。WebGPT的LLM控制器（即经过微调的GPT-3）容易受到目标劫持攻击，因为它没有防御此类攻击的机制。从源头角度看，多源输入（即用户输入和记忆机制）增加了目标劫持的攻击面。从攻击目标来看，攻击者可以操纵模型输出特定内容或瞄准工具调用，例如控制模型点击特定网页。需要注意的是，尽管多轮交互在某些情况下可以增强攻击的效果（参见第[3.2](https://arxiv.org/html/2411.09523v1#S3.SS2 "3.2\. 目标劫持 ‣ 3\. 问题输入的风险 ‣ 导航风险：基于LLM的代理在安全性、隐私和伦理方面的威胁综述")节），这些情况通常要求用户根据模型的响应定制其输入（Cheng等， [2024](https://arxiv.org/html/2411.09523v1#bib.bib48)）。然而，在WebGPT场景中，它多次使用Bing浏览器，网页被动提供信息。因此，在此情境下，多轮交互不会显著影响威胁的强度。

作为例外，WebGPT对模型提取攻击表现出更高的抗性，因为WebGPT的输入并不是GPT-3的输入。例如，Carlini等（Carlini等，[2024b](https://arxiv.org/html/2411.09523v1#bib.bib36)）提出通过随机提示查询大语言模型（LLM）以窃取模型最终层的参数。然而，对于WebGPT，随机提示可能会导致类似的输出，因为没有明显的内容信息，例如“抱歉，我无法找到关于...的信息”。这些类似的输出降低了输出的排名，增加了模型提取攻击的难度。此外，Li等（Li等，[2024d](https://arxiv.org/html/2411.09523v1#bib.bib148)）提出的窃取text-davinci003的专业代码能力的方法也会失败。这是因为代理的响应是相关网页内容的摘要，而不是模型自身的代码知识。

来自模型缺陷的风险。WebGPT 有效解决了由训练数据集中的问题引起的幻觉和偏见。例如，针对幻觉问题，WebGPT 的记忆知识库存储了一个实时更新的互联网网站索引，这有助于缓解由训练集中过时数据引起的幻觉。此外，RAG 可以减少由不完整训练引起的虚假相关性问题。作为强大的搜索引擎，Bing 浏览器在用作信息检索工具时，增强了所检索信息的相关性和流行度，从而在一定程度上减少了由训练集中的错误和不平衡数据引起的幻觉。同时，由于模型已经进行了微调以与用户环境对齐，它减少了工具调用时由于知识差距所导致的幻觉影响。然而，它并没有解决由错误解码过程引起的幻觉问题。

来自输入模型交互的风险。WebGPT 可能面临额外的后门中毒风险，因为它涉及外部训练过程。恶意内部人员可能通过向训练数据中注入错误的人类示范或偏好（Wang 等， [2023c](https://arxiv.org/html/2411.09523v1#bib.bib269)），从而破坏 WebGPT 的数据收集过程。一个被植入后门的 WebGPT 可能会在成功检索到与问题相关的 Web 参考资料后，依然对用户的问题给出错误或恶意的回答，这些问题中包含了触发器。

此外，涉及大型外部 Web 数据库增加了隐私信息泄露的风险。恶意用户可能利用 WebGPT 作为一个便捷的渠道，通过 Bing 浏览器强大的信息检索能力来收集私人信息。当 WebGPT 进一步用来推断检索数据和实体之间的潜在联系时，隐私威胁可能会得到放大，因为 WebGPT 具有语义理解能力。

总结来说，WebGPT 降低了由模型缺陷带来的风险。然而，它增加了针对有问题输入和综合性威胁的攻击面，如表 [2](https://arxiv.org/html/2411.09523v1#S6.T2 "Table 2 ‣ 6\. Case Study ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents") 所示。

### 6.2\. Voyager

随着大型语言模型（LLMs）潜力的逐步探索，基于LLM的智能体可以用于处理更复杂的任务，而具身LLM智能体的出现也随之而来。这种智能体是具有物理实体的，要么通过机器人在现实世界中具象化，要么通过虚拟化身在模拟环境中具象化。我们选择了两个具身LLM智能体的案例，涵盖了游戏场景和更接近现实世界应用的场景：具象化在《Minecraft》游戏中的Voyager（Wang等人，[2023d](https://arxiv.org/html/2411.09523v1#bib.bib264)），以及具象化在城市导航系统中的PReP（Zeng等人，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib311)）。与WebGPT相比，Voyager有四个不同之处。(i) 它直接利用GPT-3.5和GPT-4.0作为其控制中心。(ii) 它要求ChatGPT同时输出自然语言和Java代码。(iii) 它设计了一个内部记忆模块，称为技能库，智能体可以从中读取和写入。(iv) 它们的目标任务不同。WebGPT是为回答一般问题设计的，而Voyager专注于在Minecraft中完成任务。我们现在分析这些差异对各种威胁的具体影响。

表3. 摘要表：Voyager差异对各种威胁的具体影响。与独立的LLM和WebGPT相比，如果Voyager的差异增加了某种威胁，我们将其标红。

| 来源 | 威胁 | 与之前的智能体的区别 | 具体影响 |
| --- | --- | --- | --- |
| 问题输入 | 模型提取 | VOYAGER直接调用GPT-4.0和GPT-3.5。 | 攻击者可以窃取GPT-3.5和4.0的独立模型。 |
| 模型缺陷 | 幻觉 | VOYAGER玩Minecraft游戏。 | 玩Minecraft属于一个专业领域，直接使用ChatGPT可能因知识缺口导致幻觉。 |
| VOYAGER输出自然语言和JavaScript代码。 | 代码生成更容易受到幻觉问题的影响，因为生成的错误代码更可能无法执行。 |
| 组合威胁 | 后门 | 一个内部记忆模块，智能体可以从中读取和写入。 | 从其他地方下载预训练的技能库增加了额外后门的风险。 |

来自问题输入的风险。对于大多数基于输入的攻击，Voyager与WebGPT和独立的LLM同样容易受到攻击。尽管Voyager在系统指令中提示GPT-4生成JavaScript，但它并未对输入和输出进行额外的检查或格式化，因此仍然容易受到来自输入的攻击（例如目标劫持）。更严重的是，与WebGPT不同，Voyager容易受到模型提取攻击。这是因为Voyager直接使用GPT-3.5和4.0，允许攻击者窃取GPT-3.5和4.0的独立模型，这些模型可以增强其他类型的输入问题攻击。

模型缺陷带来的风险。与WebGPT相比，Voyager更容易受到幻觉问题的影响。玩Minecraft属于一个专门领域，直接使用ChatGPT可能由于知识空白导致幻觉的出现。同时，与自然语言相比，代码更容易受到幻觉引发的错误影响，因为这可能导致游戏崩溃。为了解决这个问题，作者建议可以通过RAG（例如，调用Minecraft Wiki）来减轻这一问题。另一个可能的方法是通过微调来解决知识空白，正如另一个Minecraft代理ODYSSEY所使用的那样（Liu et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib167)）。

表4. 总结表：PReP在不同威胁上的具体影响。与独立LLM、WebGPT和Voyager相比，如果PReP的差异增加某个威胁，我们将用红色突出显示。

| 来源 | 威胁 | 与之前的代理的区别 | 具体影响 |
| --- | --- | --- | --- |
| 问题输入 | 对抗样本 | 它的输入是多模态的，包括文本和图像。 | VLATTACK可以通过同时扰动两种输入模态生成更强的对抗样本。 |
| 模型缺陷 | 幻觉 | 交互环境由北京和上海的实际地标照片组成。 | 非英语语言和东部国家更容易出现幻觉。 |
| 综合威胁 | 后门 | 它的决策模块包含多个LLM（包括一个LVLM、GPT-3.5和GPT-4.0）。 | 更多的模型和额外的训练过程（例如对LVLM的微调）增加了后门的风险。 |

输入与模型交互带来的风险。Voyager在从他人下载预训练技能库时，可能面临额外的后门中毒风险。对手可以通过劫持外部环境返回的反馈，首先将故障可执行代码注入Voyager的技能库中。随后，可以优化后门触发器，以确保当触发器出现在用户的指令中时，从技能库中检索到故障可执行代码，从而执行对手希望的操作。

总结来说，Voyager增加了来自问题输入、模型缺陷和综合威胁的风险，如表[3](https://arxiv.org/html/2411.09523v1#S6.T3 "表 3 ‣ 6.2. Voyager ‣ 6. 案例研究 ‣ 导航风险：基于LLM代理的安全性、隐私性和伦理性威胁调查")所示。

### 6.3. PReP

如上所述，PReP（Zeng等人，[2024a](https://arxiv.org/html/2411.09523v1#bib.bib311)）是近期在城市导航系统中体现的一种代理。它通过观察周围的地标来确定其位置并规划方向。与WebGPT和Voyager相比，PReP有3个差异。（i）它的输入是多模态的，包括文本和图像。（ii）它的决策模块包含多个LLM（包括一个LLaVA、GPT-3.5和GPT-4），用于执行环境观察、记忆管理、计划生成和最终行动的控制。（iii）PReP的交互环境由实际的地标照片组成。我们现在分析这些差异对各种威胁的具体影响。 |

来自问题输入的风险。与WebGPT和Voyager相比，PReP面临来自问题输入的更大威胁，因为它处理两种输入模式，这两种模式都容易被操控。例如，VLATTACK（Yin等人，[2024](https://arxiv.org/html/2411.09523v1#bib.bib300)）可以通过同时扰动两种输入模式，生成更强的对抗性示例。此外，Kimura等人（Kimura等人，[2024](https://arxiv.org/html/2411.09523v1#bib.bib127)）指出，简单地向图像中添加一个“忽略”提示可以在某种程度上成功劫持多模态大模型。这种攻击在PReP场景中尤其容易实现，比如通过打印并张贴相应的劫持提示在地标上。 |

表5. 摘要表：ChatDev差异对各种威胁的具体影响。与独立的LLM、WebGPT、Voyager和PReP相比，如果ChatDev的某个差异增加了某个威胁，我们会用红色标出；如果与某个威胁无关，我们用黄色标注；如果减少了某个威胁，我们则用绿色标示。 |

| 来源 | 威胁 | 与前期代理的差异 | 具体影响 |
| --- | --- | --- | --- |
| 问题输入 | 目标劫持 | ChatDev可以调用系统内核进行代码的动态测试。 | 在动态测试过程中，ChatDev可能会导致更严重的问题，如在系统上运行恶意代码。 |
| ChatDev是一个基于多个GPT的代理框架。 | 恶意输入可以被复制到所有代理。 |
| 模型缺陷 | 幻觉 | 语言模型容易过度依赖早期错误，从而导致进一步的错误。 |
| 它允许代理进行多轮对话。 | 代理之间频繁的合作可能会放大微小的幻觉。 |
| 多轮对话提供了更详细的需求。 |
| 联合威胁 | 后门 | ChatDev可能容易受到指令后门攻击。 |

模型缺陷带来的风险。PReP面临着模型缺陷的重大威胁。以幻觉问题为例，PReP特别容易产生幻觉。与WebGPT和Voyager不同，PReP的幻觉更可能源于它在误解非英语内容方面的倾向，以及多个大规模语言模型（LLM）复合效应增强了幻觉的影响。PReP直接调用GPT-4。然而，GPT-4已被证明在遇到东方国家或非英语语境时更容易产生幻觉（Cui et al.，[2023](https://arxiv.org/html/2411.09523v1#bib.bib53)），而实验中的测试数据集包含了北京和上海的地标。此外，这类威胁的后果比WebGPT和Voyager更为严重。以偏见问题为例。如果模型缺乏关于城市交通的全面和实时信息，可能会在导航过程中引入偏见，从而导致选择次优路线，甚至无法到达目的地。道路网络变化、道路维护、实时交通拥堵以及车辆的驾驶规范等因素，都是潜在的偏见来源。

输入与模型交互带来的风险。PReP可能容易受到后门中毒攻击。对手可以污染用于感知街景地标的LLaVA模型的微调数据集。一旦部署了带有后门的PReP，攻击者可以在代理的视野中放置一个物理后门触发器，误导地标的识别和分割，从而可能使代理导航至错误的路线。

总结来说，PReP增加了来自问题输入、模型缺陷和复合威胁的风险，如表[4](https://arxiv.org/html/2411.09523v1#S6.T4 "Table 4 ‣ 6.2\. Voyager ‣ 6\. Case Study ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")所示。

### 6.4. ChatDev

大规模语言模型（LLMs）的强大能力使其能够生成多种语言的代码。代码安全是安全领域中的一个重要问题，关于LLMs生成的代码安全性已经有很多研究。最近有许多代理提出了涉及多个LLM协作的软件开发。在这里，我们考虑ChatDev（Qian et al.，[2023a](https://arxiv.org/html/2411.09523v1#bib.bib212)），这是一个涉及多个基于LLM的代理的软件开发框架。与上述三种代理相比，ChatDev有三个区别：(i) 其框架包括多个代理（每个代理包含一个LLM，可以是GPT-3.5或GPT-4）。(ii) 它允许不同代理之间进行多轮对话。(iii) 它可以调用系统内核对代码进行动态测试。我们现在分析这些差异对各种威胁的具体影响。

来自有问题输入的风险。与单一智能体类似，ChatDev 也可能受到来自输入的各种攻击（如越狱和目标劫持）。这是因为恶意输入可以在多个智能体之间传播。例如，Morris II（Cohen 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib52)）旨在通过复制恶意输入来感染其他智能体，从而针对合作的多智能体生态系统。Morris II 的威胁在于其利用智能体之间的连接性，一旦一个智能体被感染，多个智能体可能迅速崩溃，导致系统在动态测试过程中运行恶意代码等问题。

来自模型缺陷的风险。与单一智能体相比，ChatDev 可能会遭遇更严重的来自模型本身的问题（如偏见和幻觉）。例如，关于幻觉，尽管 ChatDev 通过要求助手在给出正式回应之前主动寻求讲师的更详细建议来缓解这一问题，但它无法完全避免这一问题。此外，智能体之间的频繁合作可能会放大小的幻觉（Hong 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib99)），而语言模型倾向于过度承认早期错误，这可能导致进一步的错误，尽管它本不应该犯这些错误（Zhang 等，[2023c](https://arxiv.org/html/2411.09523v1#bib.bib320)），这使得多智能体框架更容易发生幻觉。

来自输入-模型交互的风险。ChatDev 可能容易受到指令后门攻击（Zhang 等，[2024f](https://arxiv.org/html/2411.09523v1#bib.bib321)）。ChatDev 开发团队中的恶意内部人员可能会将后门指令植入到多个智能体中的某个系统提示中。这些后门指令指定了一个情境，其中后门化的 ChatDev 会在用户指令中嵌入的触发条件下生成攻击者所希望的软件。

总结来说，ChatDev 增加了来自有问题输入、模型缺陷和联合威胁的风险，如表 [5](https://arxiv.org/html/2411.09523v1#S6.T5 "Table 5 ‣ 6.3\. PReP ‣ 6\. Case Study ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents") 所示。

## 7\. 未来方向

在上述章节中，我们总结了基于LLM的智能体所面临的各种威胁，并指出了当前研究的局限性。基于前述对四个案例研究的总结和分析，我们从数据支持、方法论支持和政策支持三个角度提出了未来的几个潜在研究方向。

数据支持。数据集是风险分析的基础。充分且全面的数据集可以提供更为彻底和公正的风险对比分析及其相应方法。根据图[9](https://arxiv.org/html/2411.09523v1#S4.F9 "Figure 9 ‣ 4\. Risks from Model Flaws ‣ Navigating the Risks: A Survey of Security, Privacy, and Ethics Threats in LLM-Based Agents")，目前用于评估LLM智能体面临的各种威胁的数据集存在以下不足之处：

+   •

    缺乏真实场景中的多轮交互数据。大多数现实世界中的智能体交互是多轮的（例如，案例研究中的四个智能体）。此外，一些威胁在多轮交互中可能变得更加严重（例如，目标劫持）。因此，策划包含多轮交互的数据集，可以更好地评估威胁的严重性。

+   •

    任务限制对一般问答数据集的影响。在实际场景中，智能体通常处理特定领域的任务（例如，案例研究中的最后三个智能体）。一些威胁源于专业领域的知识空白，使其更加严重（例如，幻觉）。因此，开发专注于特定领域的数据集，可以更好地评估大语言模型（LLM）应用在这些领域中带来的威胁。

+   •

    模态主要限于普通英语文本。许多实际场景中的智能体必须处理多语言和多模态的情况（例如，案例研究中的PReP）。某些威胁在资源匮乏的语言或多模态环境中可能变得更加明显（例如，越狱攻击）。因此，创建包括多种模态和语言的数据集，可以增强对威胁的评估。

+   •

    输入通常仅限于单一角色（即用户）。在实际场景中，智能体中的LLM接收来自多个来源的输入（例如，案例研究中的四个智能体）。当涉及多个输入来源时，一些威胁可能会升级（例如，后门攻击）。因此，创建具有多个输入来源的数据集，可以改进威胁严重性的评估。

+   •

    评估通常集中在单一的LLM上。在实际场景中，智能体可能会结合多个LLM（例如，PReP），并且多个智能体可能协同工作（例如，ChatDev）。在多个LLM之间的交互中，一些威胁可能会变得更加严重（例如，偏见）。因此，为多LLM场景中的交互建立基准，可以更好地评估威胁水平。

方法论支持。通过前面对每种威胁技术进展的局限性分析以及案例研究中实际场景的分析，我们从方法论支持的角度提出以下未来研究方向：

+   •

    理论分析框架。目前，一些形式的威胁缺乏数学定义，导致难以从理论上分析和量化其严重性。例如，在案例研究中，所有代理都面临不同程度的幻觉问题，原因各不相同。虽然从某些角度（例如，ChatDev的沟通去幻觉机制）减少了幻觉的概率，但由于其他因素（例如，ChatDev中多个代理的协作），它们也增加了幻觉的强度。由于幻觉缺乏数学定义，因此很难分析它们最终面临的幻觉问题的程度。未来的工作可以集中在严格定义基于LLM的代理面临的威胁，并建立一个分析框架，以便更清晰地分析和量化各种风险。

+   •

    以可解释性为驱动的攻击与防御策略。目前大多数关于威胁的研究通过启发式方法进行探讨。例如，大多数目标劫持攻击策略基于脚本或梯度优化。一项开创性的研究通过观察模型的内部激活，成功地检测到目标劫持攻击（Abdelnabi 等，[2024](https://arxiv.org/html/2411.09523v1#bib.bib6)）。模型解释方法可以从不同的角度揭示模型的决策基础。未来，将各种解释算法应用于设计攻击与防御策略，可以更深入地理解基于LLM的代理所面临的风险，并增强防御措施。

+   •

    针对代理的攻击与防御策略。与数据支持面临的问题类似，目前大多数攻击与防御方法主要针对独立的LLM设计，只有少数方法解决了代理的1-2个关键特性。例如，Dong 等人（Dong 等，[2023](https://arxiv.org/html/2411.09523v1#bib.bib67)）在研究后门攻击时，将代理根据其复杂度分为不同的等级（即L1到L5）。事实上，大多数当前的代理框架（包括案例研究中的四个代理）属于最高级别（即L5）。未来关于高级代理面临的威胁的研究将使我们更接近代理在实际应用中的使用场景。

政策支持。为了确保基于大型语言模型（LLM）代理在使用过程中的可靠性，从政府政策角度来看，可以解决以下关键问题：

+   •

    建立代理宪法框架。关于基于LLM的代理有很多调查。大多数调查提供了代理操作的框架（Deng et al., [2024a](https://arxiv.org/html/2411.09523v1#bib.bib62); Cui et al., [2024](https://arxiv.org/html/2411.09523v1#bib.bib54); Zhang et al., [2024a](https://arxiv.org/html/2411.09523v1#bib.bib328); Wang et al., [2024b](https://arxiv.org/html/2411.09523v1#bib.bib271)）。虽然它们共享许多相似的模块（如记忆、决策模块等），但也存在一些差异。政府可以制定一个全面的代理宪法，概述基于LLM的代理安全和伦理操作的基本原则、规则和指南。该宪法应涵盖安全性、隐私保护、社会价值对齐等方面。

+   •

    完善治理框架和监管政策。许多国家和地区，例如欧盟（Union, [2024](https://arxiv.org/html/2411.09523v1#bib.bib256)）和美国（House, [2023](https://arxiv.org/html/2411.09523v1#bib.bib101)），已出台了关于AI的安全立法，但尚未提供针对基于LLM的代理的具体监管规定。应制定应对基于LLM的代理所带来独特挑战的治理框架和监管政策，例如责任、数据隐私以及滥用或意外后果的潜在风险。这些政策应能够适应LLM技术快速发展的环境。

+   •

    投资于研究与开发。为专注于增强基于LLM的代理可靠性、安全性和安全性的研究与开发分配资源。这可以包括资助开发先进的安全机制、改进推理能力以及探索可能提供更高可靠性和可信度的替代AI架构。

## 8. 结论

本次调查聚焦于基于LLM的代理面临的各种威胁。我们提出了这些威胁的新分类法，并根据这一框架和分类法总结了它们的技术进展和局限性。随后，我们选择了四个真实世界中的代理作为案例研究，分析这些代理在实际使用中可能遇到的威胁类型及其潜在原因。最后，基于上述分析，我们提出了未来研究的有前景方向。

## 参考文献

+   (1)

+   E2B (2021) 2021. E2B沙盒。 [https://github.com/e2b-dev/e2b](https://github.com/e2b-dev/e2b)。

+   Jet (2021) 2021. JetBot。 [https://github.com/NVIDIA-AI-IOT/jetbot](https://github.com/NVIDIA-AI-IOT/jetbot)。

+   Pix (2024) 2024. Pixloop。 [https://www.pixmoving.com/pixloop](https://www.pixmoving.com/pixloop)。

+   Who (2024) 2024. 我们是谁。 [https://www.pixmoving.com/about-us](https://www.pixmoving.com/about-us)。

+   Abdelnabi 等人（2024）Sahar Abdelnabi、Aideen Fay、Giovanni Cherubin、Ahmed Salem、Mario Fritz 和 Andrew Paverd。2024年。你还在正轨上吗！？通过激活捕捉LLM任务漂移。*arXiv preprint arXiv:2406.00799*（2024）。

+   Agarwal 等人（2024）Divyansh Agarwal、Alexander R Fabbri、Philippe Laban、Shafiq Joty、Caiming Xiong 和 Chien-Sheng Wu。2024年。调查多回合LLM交互中的提示泄露效应和黑箱防御。*arXiv preprint arXiv:2404.16251*（2024）。

+   Agarwal 等人（2020）Vedika Agarwal、Rakshith Shetty 和 Mario Fritz。2020年。走向因果VQA：通过不变和协变语义编辑揭示并减少虚假的相关性。收录于 *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*，9690–9698。

+   Aghakhani 等人（2023）Hojjat Aghakhani、Wei Dai、Andre Manoel、Xavier Fernandes、Anant Kharkar、Christopher Kruegel、Giovanni Vigna、David Evans、Ben Zorn 和 Robert Sim。2023年。TrojanPuzzle：暗中毒化代码建议模型。*arxiv preprint arxiv:2301.02344*（2023）。

+   Ahn 等人（2022）Michael Ahn、Anthony Brohan、Noah Brown、Yevgen Chebotar、Omar Cortes、Byron David、Chelsea Finn、Chuyuan Fu、Keerthana Gopalakrishnan、Karol Hausman 等人。2022年。做我能做的，而不是我说的：将语言与机器人能力结合。*arXiv preprint arXiv:2204.01691*（2022）。

+   Alon 和 Kamfonas（2023）Gabriel Alon 和 Michael Kamfonas。2023年。通过困惑度检测语言模型攻击。*arXiv preprint arXiv:2308.14132*（2023）。

+   Amrhein 等人（2023）Chantal Amrhein、Florian Schottmann、Rico Sennrich 和 Samuel Läubli。2023年。利用偏见模型去偏文本：一种性别公平的重写模型。收录于 *Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*，4486–4506。

+   Anil 等人（2024）Cem Anil、Esin Durmus、Mrinank Sharma、Joe Benton、Sandipan Kundu、Joshua Batson、Nina Rimsky、Meg Tong、Jesse Mu、Daniel Ford 等人。2024年。多次破解。*Anthropic, April*（2024）。

+   Athalye 等人（2018）Anish Athalye、Nicholas Carlini 和 David Wagner。2018年。模糊梯度带来了虚假的安全感：绕过对抗样本的防御。收录于 *International conference on machine learning*。PMLR，274–283。

+   Baeza-Yates（2018）Ricardo Baeza-Yates。2018年。网络偏见。*Commun. ACM* 61, 6（2018），54–61。

+   Bagdasaryan 和 Shmatikov（2022）Eugene Bagdasaryan 和 Vitaly Shmatikov。2022年。旋转语言模型：作为服务的宣传风险及其对策。*arXiv preprint arXiv:2112.05224*（2022）。

+   Bagdasaryan 等人（2024）Eugene Bagdasaryan、Ren Yi、Sahra Ghalebikesabi、Peter Kairouz、Marco Gruteser、Sewoong Oh、Borja Balle 和 Daniel Ramage。2024年。空气间隙：保护隐私意识型对话代理。*arXiv preprint arXiv:2405.05175*（2024）。

+   Bai 等人 (2022) Yuntao Bai, Andy Jones, Kamal Ndousse, Amanda Askell, Anna Chen, Nova DasSarma, Dawn Drain, Stanislav Fort, Deep Ganguli, Tom Henighan 等人. 2022. 使用来自人类反馈的强化学习训练有益且无害的助手。*arXiv 预印本 arXiv:2204.05862* (2022)。

+   Bao 等人 (2021) Rongzhou Bao, Jiayi Wang, 和 Hai Zhao. 2021. 防御预训练语言模型免受对抗性词汇替换影响而不牺牲性能。*arXiv 预印本 arXiv:2105.14553* (2021)。

+   Barikeri 等人 (2021) Soumya Barikeri, Anne Lauscher, Ivan Vulić, 和 Goran Glavaš. 2021. RedditBias：用于评估偏见和去偏见的对话语言模型的现实世界资源。*arXiv 预印本 arXiv:2106.03521* (2021)。

+   Bhardwaj 和 Poria (2023) Rishabh Bhardwaj 和 Soujanya Poria. 2023. 使用话语链对大型语言模型进行红队攻击以实现安全对齐。*arXiv 预印本 arXiv:2308.09662* (2023)。

+   Biderman 等人 (2024) Stella Biderman, Usvsn Prashanth, Lintang Sutawika, Hailey Schoelkopf, Quentin Anthony, Shivanshu Purohit, 和 Edward Raff. 2024. 大型语言模型中的突现和可预测记忆。*神经信息处理系统进展* 36 (2024)。

+   Biten 等人 (2022) Ali Furkan Biten, Lluís Gómez, 和 Dimosthenis Karatzas. 2022. 沙滩上的时钟：减少图像描述中的物体幻觉。发表于 *IEEE/CVF 冬季计算机视觉应用会议论文集*。1381–1390。

+   Bourtoule 等人 (2021) Lucas Bourtoule, Varun Chandrasekaran, Christopher A Choquette-Choo, Hengrui Jia, Adelin Travers, Baiwu Zhang, David Lie, 和 Nicolas Papernot. 2021. 机器遗忘。发表于 *2021 IEEE 安全与隐私研讨会 (SP)*。IEEE, 141–159。

+   Brown 等人 (2020) Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell 等人. 2020. 语言模型是少样本学习者。*神经信息处理系统进展* 33 (2020), 1877–1901。

+   Buckman 等人 (2018) Jacob Buckman, Aurko Roy, Colin Raffel, 和 Ian Goodfellow. 2018. 温度编码：抗对抗样本的一种热编码方法。发表于 *国际学习表征会议*。

+   Buolamwini 和 Gebru (2018) Joy Buolamwini 和 Timnit Gebru. 2018. 性别阴影：商业性别分类中的交叉准确性差异。发表于 *公平性、问责制与透明度会议*。PMLR, 77–91。

+   Cabello 等人 (2023) Laura Cabello, Anna Katrine Jørgensen, 和 Anders Søgaard. 2023. 关于语言模型中关联偏见和经验公平性的独立性。发表于 *2023 年 ACM 公平性、问责制与透明度会议论文集*。370–378。

+   Caliskan 等人 (2017) Aylin Caliskan, Joanna J Bryson, 和 Arvind Narayanan. 2017. 自动从语言语料库中推导的语义包含类人偏见。*科学* 356, 6334 (2017), 183–186。

+   Cao 等（2024）Qingxing Cao, Junhao Cheng, Xiaodan Liang, 和 Liang Lin. 2024. VisDiaHalBench：用于诊断大型视觉语言模型幻觉的视觉对话基准。在*第62届计算语言学协会年会（第1卷：长篇论文）*。12161–12176。

+   Carlini 等（2022）Nicholas Carlini, Steve Chien, Milad Nasr, Shuang Song, Andreas Terzis, 和 Florian Tramer. 2022. 从基本原理出发的成员推断攻击。在*2022 IEEE 安全与隐私研讨会（SP）*。IEEE，1897–1914。

+   Carlini 等（2023a）Nicolas Carlini, Jamie Hayes, Milad Nasr, Matthew Jagielski, Vikash Sehwag, Florian Tramer, Borja Balle, Daphne Ippolito, 和 Eric Wallace. 2023a. 从扩散模型中提取训练数据。在*第32届 USENIX 安全研讨会（USENIX Security 23）*。5253–5270。

+   Carlini 等（2023b）Nicholas Carlini, Daphne Ippolito, Matthew Jagielski, Katherine Lee, Florian Tramer, 和 Chiyuan Zhang. 2023b. 定量化神经语言模型的记忆化。在*第十一届国际学习表示会议*。[https://openreview.net/forum?id=TatRHT_1cK](https://openreview.net/forum?id=TatRHT_1cK)

+   Carlini 等（2020）Nicholas Carlini, Matthew Jagielski, 和 Ilya Mironov. 2020. 神经网络模型的密码分析提取。在*国际密码学年会*。Springer，189–218。

+   Carlini 等（2024a）Nicholas Carlini, Milad Nasr, Christopher A Choquette-Choo, Matthew Jagielski, Irena Gao, Pang Wei W Koh, Daphne Ippolito, Florian Tramer, 和 Ludwig Schmidt. 2024a. 对齐的神经网络是否在对抗性上对齐？*神经信息处理系统进展* 36（2024）。

+   Carlini 等（2024b）Nicholas Carlini, Daniel Paleka, Krishnamurthy Dj Dvijotham, Thomas Steinke, Jonathan Hayase, A Feder Cooper, Katherine Lee, Matthew Jagielski, Milad Nasr, Arthur Conmy 等. 2024b. 从生产语言模型中窃取部分内容。*arXiv 预印本 arXiv:2403.06634*（2024）。

+   Carlini 等（2021）Nicholas Carlini, Florian Tramer, Eric Wallace, Matthew Jagielski, Ariel Herbert-Voss, Katherine Lee, Adam Roberts, Tom Brown, Dawn Song, Ulfar Erlingsson 等. 2021. 从大型语言模型中提取训练数据。在*第30届 USENIX 安全研讨会（USENIX Security 21）*。2633–2650。

+   Carlini 和 Wagner（2017）Nicholas Carlini 和 David Wagner. 2017. 评估神经网络鲁棒性的方法。在*2017 IEEE 安全与隐私研讨会（SP）*。IEEE，39–57。

+   Chao 等（2024）Patrick Chao, Edoardo Debenedetti, Alexander Robey, Maksym Andriushchenko, Francesco Croce, Vikash Sehwag, Edgar Dobriban, Nicolas Flammarion, George J Pappas, Florian Tramer 等. 2024. Jailbreakbench：一个用于破解大型语言模型的开放鲁棒性基准。*arXiv 预印本 arXiv:2404.01318*（2024）。

+   Chase（2022）Harrison Chase. 2022. LangChain。[https://github.com/langchain-ai/langchain](https://github.com/langchain-ai/langchain)。

+   Chatterjee（2018）Satrajit Chatterjee. 2018. 学习与记忆。发表于 *国际机器学习会议*。PMLR，755–763。

+   Chen 等人（2024b）Chen Chen, Daochang Liu 和 Chang Xu. 2024b. 朝着无记忆扩散模型的方向发展。发表于 *IEEE/CVF计算机视觉与模式识别会议论文集*。8425–8434。

+   Chen 等人（2023）Dake Chen, Hanbin Wang, Yunhao Huo, Yuzhao Li 和 Haoyang Zhang. 2023. GameGPT：一个用于游戏开发的多代理协作框架。*arXiv 预印本 arXiv:2310.08067*（2023）。

+   Chen 等人（2020）Jianbo Chen, Michael I Jordan 和 Martin J Wainwright. 2020. Hopskipjumpattack：一种查询高效的基于决策的攻击。发表于 *2020年IEEE安全与隐私研讨会（SP）*。IEEE，1277–1294。

+   Chen 等人（2024a）Kedi Chen, Qin Chen, Jie Zhou, Yishen He 和 Liang He. 2024a. DiaHalu：一个针对大型语言模型的对话级幻觉评估基准。*arXiv 预印本 arXiv:2403.00896*（2024）。

+   Chen 等人（2024c）Sizhe Chen, Julien Piet, Chawin Sitawarin 和 David Wagner. 2024c. StruQ：通过结构化查询防御提示注入攻击。*arXiv 预印本 arXiv:2402.06363*（2024）。

+   Chen 等人（2024d）Zhaorun Chen, Zhen Xiang, Chaowei Xiao, Dawn Song 和 Bo Li. 2024d. AgentPoison：通过毒害记忆或知识库进行红队攻击大型语言模型代理。*arXiv 预印本 arXiv:2407.12784*（2024）。

+   Cheng 等人（2024）Yixin Cheng, Markos Georgopoulos, Volkan Cevher 和 Grigorios G Chrysos. 2024. 通过多轮互动利用上下文进行越狱攻击。*arXiv 预印本 arXiv:2402.09177*（2024）。

+   Chu 等人（2024）Junjie Chu, Yugeng Liu, Ziqing Yang, Xinyue Shen, Michael Backes 和 Yang Zhang. 2024. 对大型语言模型的越狱攻击的全面评估。*arXiv 预印本 arXiv:2402.05668*（2024）。

+   Chuang 等人（2023）Yung-Sung Chuang, Yujia Xie, Hongyin Luo, Yoon Kim, James Glass 和 Pengcheng He. 2023. Dola：通过对比层解码提高大型语言模型的事实性。*arXiv 预印本 arXiv:2309.03883*（2023）。

+   Chung 等人（2023）John Joon Young Chung, Ece Kamar 和 Saleema Amershi. 2023. 在保持准确性的同时增加多样性：使用大型语言模型和人工干预生成文本数据。发表于 *第61届计算语言学协会年会*。

+   Cohen 等人（2024）Stav Cohen, Ron Bitton 和 Ben Nassi. 2024. AI蠕虫来了：释放零点击蠕虫，针对GenAI驱动的应用程序。*arXiv 预印本 arXiv:2403.02817*（2024）。

+   Cui 等人（2023）Chenhang Cui, Yiyang Zhou, Xinyu Yang, Shirley Wu, Linjun Zhang, James Zou 和 Huaxiu Yao. 2023. GPT-4V（ision）中幻觉的整体分析：偏差和干扰挑战。*arXiv 预印本 arXiv:2311.03287*（2023）。

+   Cui 等人 (2024) Tianyu Cui, Yanling Wang, Chuanpu Fu, Yong Xiao, Sijia Li, Xinhao Deng, Yunpeng Liu, Qinglin Zhang, Ziyi Qiu, Peiyang Li 等人。2024年。大语言模型系统的风险分类、缓解和评估基准。*arXiv 预印本 arXiv:2401.05778* (2024)。

+   Dagan 等人 (2023) Gautier Dagan, Frank Keller 和 Alex Lascarides。2023年。使用大语言模型进行动态规划。*arXiv 预印本 arXiv:2308.06391* (2023)。

+   Dale 等人 (2023) David Dale, Elena Voita, Janice Lam, Prangthip Hansanti, Christophe Ropers, Elahe Kalbassi, Cynthia Gao, Loïc Barrault 和 Marta Costa-jussà。2023年。Halomi：用于机器翻译中的多语言幻觉和遗漏检测的人工标注基准。载于 *2023年自然语言处理实证方法会议论文集*。638–653。

+   d’Alessandro 等人 (2017) Brian d’Alessandro, Cathy O’Neil 和 Tom LaGatta。2017年。谨慎分类：数据科学家的歧视意识分类指南。*Big data* 5, 2 (2017), 120–134。

+   Danks 和 London (2017) David Danks 和 Alex John London。2017年。自主系统中的算法偏见。载于 *Ijcai*，第17卷。4691–4697。

+   Delobelle 等人 (2022) Pieter Delobelle, Ewoenam Kwaku Tokpo, Toon Calders 和 Bettina Berendt。2022年。用有偏的尺子测量公平性：关于预训练语言模型偏差度量的比较研究。载于 *2022年北美计算语言学会年会论文集*。计算语言学协会，1693–1706。

+   Deng 等人 (2024b) Gelei Deng, Yi Liu, Yuekang Li, Kailong Wang, Ying Zhang, Zefeng Li, Haoyu Wang, Tianwei Zhang 和 Yang Liu。2024b年。Masterkey：大语言模型聊天机器人的自动破解。载于 *Proc. ISOC NDSS*。

+   Deng 等人 (2023) Yue Deng, Wenxuan Zhang, Sinno Jialin Pan 和 Lidong Bing。2023年。大语言模型中的多语言越狱挑战。*arXiv 预印本 arXiv:2310.06474* (2023)。

+   Deng 等人 (2024a) Zehang Deng, Yongjian Guo, Changzhou Han, Wanlun Ma, Junwu Xiong, Sheng Wen 和 Yang Xiang。2024a年。AI代理面临威胁：关键安全挑战与未来发展方向的调查。*arXiv 预印本 arXiv:2406.02630* (2024)。

+   Devlin (2018) Jacob Devlin。2018年。Bert：用于语言理解的深度双向变换器预训练。*arXiv 预印本 arXiv:1810.04805* (2018)。

+   Dhingra 等人 (2023) Harnoor Dhingra, Preetiha Jayashanker, Sayali Moghe 和 Emma Strubell。2023年。酷儿首先是人：在大语言模型中解构性别身份刻板印象。*arXiv 预印本 arXiv:2307.00101* (2023)。

+   D’Incà 等人 (2024) Moreno D’Incà, Elia Peruzzo, Massimiliano Mancini, Dejia Xu, Vidit Goel, Xingqian Xu, Zhangyang Wang, Humphrey Shi 和 Nicu Sebe。2024年。OpenBias：文本到图像生成模型中的开放集偏差检测。载于 *IEEE/CVF计算机视觉与模式识别会议论文集*。12225–12235。

+   Dolci 等人（2023）Tommaso Dolci, Fabio Azzalini, 和 Mara Tanelli. 2023. 改进句子编码器中的性别相关公平性：一种基于语义的方法. *Data Science and Engineering* 8, 2 (2023), 177–195.

+   Dong 等人（2023）Tian Dong, Minhui Xue, Guoxing Chen, Rayne Holland, Shaofeng Li, Yan Meng, Zhen Liu, 和 Haojin Zhu. 2023. philosopher's stone：大语言模型插件的特洛伊木马攻击. *arXiv preprint arXiv:2312.00374* (2023).

+   Dong 等人（2021）Xinshuai Dong, Anh Tuan Luu, Min Lin, Shuicheng Yan, 和 Hanwang Zhang. 2021. 预训练语言模型应如何针对对抗鲁棒性进行微调？ *Advances in Neural Information Processing Systems* 34 (2021), 4356–4369.

+   Dou 等人（2024）Chengfeng Dou, Ying Zhang, Yanyuan Chen, Zhi Jin, Wenpin Jiao, Haiyan Zhao, 和 Yu Huang. 2024. 检测、诊断与解释：一种中文医疗幻觉评估基准. 载于 *2024年联合国际计算语言学会议、语言资源与评估会议（LREC-COLING 2024）*，4784–4794.

+   Doumbouya 等人（2024）Moussa Koulako Bala Doumbouya, Ananjan Nandi, Gabriel Poesia, Davide Ghilardi, Anna Goldie, Federico Bianchi, Dan Jurafsky, 和 Christopher D Manning. 2024. h4rm3l: 用于大语言模型安全评估的可组合越狱攻击动态基准. *arXiv preprint arXiv:2408.04811* (2024).

+   Du 等人（2021）Tianyu Du, Shouling Ji, Lujia Shen, Yao Zhang, Jinfeng Li, Jie Shi, Chengfang Fang, Jianwei Yin, Raheem Beyah, 和 Ting Wang. 2021. Cert-RNN: 迈向证明递归神经网络的鲁棒性. *CCS* 21, 2021 (2021), 15–19.

+   Dwork 等人（2014）Cynthia Dwork, Aaron Roth 等人. 2014. 差分隐私的算法基础. *Foundations and Trends® in Theoretical Computer Science* 9, 3–4 (2014), 211–407.

+   Dziri 等人（2021）Nouha Dziri, Andrea Madotto, Osmar Zaïane, 和 Avishek Joey Bose. 2021. 神经路径猎手：通过路径引导减少对话系统中的幻觉. *arXiv preprint arXiv:2104.08455* (2021).

+   Eykholt 等人（2018）Kevin Eykholt, Ivan Evtimov, Earlence Fernandes, Bo Li, Amir Rahmati, Chaowei Xiao, Atul Prakash, Tadayoshi Kohno, 和 Dawn Song. 2018. 针对深度学习视觉分类的强大物理世界攻击. 载于 *IEEE计算机视觉与模式识别会议论文集*，1625–1634.

+   Fatemi 等人（2023）Zahra Fatemi, Chen Xing, Wenhao Liu, 和 Caimming Xiong. 2023. 在不引发灾难性遗忘的情况下改进预训练语言模型的性别公平性. 载于 *第61届计算语言学协会年会论文集（第二卷：简短论文）*，1249–1262.

+   Favero 等人（2024）Alessandro Favero、Luca Zancato、Matthew Trager、Siddharth Choudhary、Pramuditha Perera、Alessandro Achille、Ashwin Swaminathan 和 Stefano Soatto。2024年。《通过视觉信息基础控制多模态幻觉》。收录于 *《IEEE/CVF计算机视觉与模式识别会议论文集》*，14303–14312页。

+   Feldman（2020）Vitaly Feldman。2020年。《学习是否需要记忆？一个关于长尾的短故事》。收录于 *《第52届ACM SIGACT理论计算研讨会论文集》*，954–959页。

+   Felkner 等人（2023）Virginia Felkner、Ho-Chun Herbert Chang、Eugene Jang 和 Jonathan May。2023年。《WinoQueer：面向大规模语言模型反LGBTQ+偏见的社区反馈基准》。收录于 *《第61届计算语言学协会年会（第一卷：长篇论文）》*，9126–9140页。

+   Feng 等人（2023）Shangbin Feng、Chan Young Park、Yuhan Liu 和 Yulia Tsvetkov。2023年。《从预训练数据到语言模型再到下游任务：追踪导致不公平自然语言处理模型的政治偏差轨迹》。收录于 *《第61届计算语言学协会年会》*。

+   Feng 等人（2020）Zhangyin Feng、Daya Guo、Duyu Tang、Nan Duan、Xiaocheng Feng、Ming Gong、Linjun Shou、Bing Qin、Ting Liu、Daxin Jiang 等人。2020年。《Codebert：一种用于编程和自然语言的预训练模型》。*arXiv预印本arXiv:2002.08155*（2020）。

+   Fleisig 等人（2023）Eve Fleisig、Aubrie Amstutz、Chad Atalla、Su Lin Blodgett、Hal Daumé III、Alexandra Olteanu、Emily Sheng、Dan Vann 和 Hanna Wallach。2023年。《FairPrism：评估文本生成中的公平性相关危害》。收录于 *《第61届计算语言学协会年会（第一卷：长篇论文）》*，6231–6251页。

+   Gallegos 等人（2024）Isabel O Gallegos、Ryan A Rossi、Joe Barrow、Md Mehrab Tanjim、Sungchul Kim、Franck Dernoncourt、Tong Yu、Ruiyi Zhang 和 Nesreen K Ahmed。2024年。《大型语言模型中的偏差与公平性：一项调查》。*《计算语言学》*（2024），1–79页。

+   Garimella 等人（2022）Aparna Garimella、Rada Mihalcea 和 Akhash Amarnath。2022年。《面向人口统计的语言模型微调作为偏差缓解技术》。收录于 *《亚洲太平洋计算语言学协会第二届会议及第十二届国际联合自然语言处理会议（第二卷：短篇论文）》*，311–319页。

+   Gehman 等人（2020）Samuel Gehman、Suchin Gururangan、Maarten Sap、Yejin Choi 和 Noah A Smith。2020年。《Realtoxicityprompts：评估语言模型中的神经毒性退化》。*arXiv预印本arXiv:2009.11462*（2020）。

+   Ghanbarzadeh 等人（2023）Somayeh Ghanbarzadeh、Yan Huang、Hamid Palangi、Radames Cruz Moreno 和 Hamed Khanpour。2023年。《性别调优：为预训练语言模型的去偏化赋能微调》。收录于 *《计算语言学协会年会成果：ACL 2023》*，5448–5458页。

+   Goldblum等人（2020）Micah Goldblum、Dimitris Tsipras、Chulin Xie、Xinyun Chen、Avi Schwarzschild、Dawn Song、Aleksander Madry、Bo Li和Tom Goldstein。2020年。**机器学习数据集安全**：数据中毒、后门攻击与防御。*arXiv预印本arXiv:2012.10544*（2020）。

+   Gong等人（2023）Yichen Gong、Delong Ran、Jinyuan Liu、Conglei Wang、Tianshuo Cong、Anyu Wang、Sisi Duan和Xiaoyun Wang。2023年。**Figstep**：通过排版视觉提示破解大型视觉语言模型。*arXiv预印本arXiv:2311.05608*（2023）。

+   Graves等人（2021）Laura Graves、Vineel Nagisetty和Vijay Ganesh。2021年。**失忆机器学习**。载于*AAAI人工智能会议论文集*，第35卷，11516–11524页。

+   Greshake等人（2023）Kai Greshake、Sahar Abdelnabi、Shailesh Mishra、Christoph Endres、Thorsten Holz和Mario Fritz。2023年。**不是你所注册的**：通过间接提示注入危害真实世界的LLM集成应用。载于*第16届ACM人工智能与安全研讨会论文集*，79–90页。

+   Gu等人（2024）Xiangming Gu、Xiaosen Zheng、Tianyu Pang、Chao Du、Qian Liu、Ye Wang、Jing Jiang和Min Lin。2024年。**Agent Smith**：一张图像可以在指数级的速度下越狱百万个多模态LLM代理。*arXiv预印本arXiv:2402.08567*（2024）。

+   Guo和Caliskan（2021）Wei Guo和Aylin Caliskan。2021年。**检测新兴的交叉偏见**：上下文化的词嵌入包含一种类人偏见的分布。载于*2021年AAAI/ACM人工智能、伦理与社会会议论文集*，122–133页。

+   Hall等人（2024）Siobhan Mackenzie Hall、Fernanda Gonçalves Abrantes、Hanwen Zhu、Grace Sodunke、Aleksandar Shtedritski和Hannah Rose Kirk。2024年。**Visogender**：一个用于基准测试图像-文本代词解析中的性别偏见的数据集。*神经信息处理系统进展* 36（2024）。

+   Han等人（2022a）Xudong Han、Timothy Baldwin和Trevor Cohn。2022a年。**平衡偏见**：通过平衡训练实现公平性。载于*2022年自然语言处理经验方法会议论文集*，11335–11350页。

+   Han等人（2022b）Xudong Han、Timothy Baldwin和Trevor Cohn。2022b年。**通过对抗学习实现机会公平性**。*arXiv预印本arXiv:2203.06317*（2022）。

+   He等人（2023）Ping He、Yifan Xia、Xuhong Zhang和Shouling Ji。2023年。**基于查询的高效攻击**：在零知识设置下对基于机器学习的安卓恶意软件检测进行攻击。载于*2023年ACM SIGSAC计算机与通信安全会议论文集*，90–104页。

+   He等人（2021）Xuanli He、Lingjuan Lyu、Qiongkai Xu和Lichao Sun。2021年。**模型提取与对抗性可转移性**，你的BERT是脆弱的！*arXiv预印本arXiv:2103.10013*（2021）。

+   He等人（2022）Xuanli He, Qiongkai Xu, Yi Zeng, Lingjuan Lyu, Fangzhao Wu, Jiwei Li, 和 Ruoxi Jia. 2022. Cater: 通过条件水印保护文本生成API的知识产权. *神经信息处理系统进展* 35 (2022), 5431–5445.

+   Hines等人（2024）Keegan Hines, Gary Lopez, Matthew Hall, Federico Zarfati, Yonatan Zunger, 和 Emre Kiciman. 2024. 通过聚焦攻击防御间接提示注入攻击. *arXiv预印本 arXiv:2403.14720* (2024).

+   Hong等人（2023）Sirui Hong, Xiawu Zheng, Jonathan Chen, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou等. 2023. Metagpt: 多代理协作框架的元编程. *arXiv预印本 arXiv:2308.00352* (2023).

+   Hoory等人（2021）Shlomo Hoory, Amir Feder, Avichai Tendler, Sofia Erell, Alon Peled-Cohen, Itay Laish, Hootan Nakhost, Uri Stemmer, Ayelet Benjamini, Avinatan Hassidim等. 2021. 学习和评估差分隐私预训练语言模型. 在*计算语言学协会年会：EMNLP 2021*。1178–1189.

+   White House (2023) 美国白宫. 2023. 关于人工智能安全、可靠和可信赖发展的行政命令. [https://www.whitehouse.gov/briefing-room/presidential-actions/2023/10/30/executive-order-on-the-safe-secure-and-trustworthy-development-and-use-of-artificial-intelligence/](https://www.whitehouse.gov/briefing-room/presidential-actions/2023/10/30/executive-order-on-the-safe-secure-and-trustworthy-development-and-use-of-artificial-intelligence/).

+   Howard等人（2024）Phillip Howard, Avinash Madasu, Tiep Le, Gustavo Lujan Moreno, Anahita Bhiwandiwalla, 和 Vasudev Lal. 2024. SocialCounterfactuals: 利用反事实示例探讨并缓解视觉语言模型中的交叉社会偏见. *IEEE/CVF计算机视觉与模式识别会议论文集*. 11975–11985.

+   Hu等人（2021）Edward J. Hu, Yelong Shen, Phillip Wallis, Zeyuan Allen-Zhu, Yuanzhi Li, Shean Wang, Lu Wang, 和 Weizhu Chen. 2021. LoRA: 大型语言模型的低秩适配. *arXiv预印本 arXiv:2106.09685* (2021).

+   Hu等人（2023）Zhengmian Hu, Gang Wu, Saayan Mitra, Ruiyi Zhang, Tong Sun, Heng Huang, 和 Vishy Swaminathan. 2023. 基于困惑度测量和上下文信息的令牌级对抗性提示检测. *arXiv预印本 arXiv:2311.11509* (2023).

+   Huang等人（2022b）Jie Huang, Hanyin Shao, 和 Kevin Chen Chuan Chang. 2022b. 大型预训练语言模型是否泄露你的个人信息？在*2022年计算语言学协会年会：EMNLP 2022*。

+   Huang 等人（2024a）黄启东、董晓怡、张潘、王彬、何聪辉、王嘉琪、林大华、张伟明 和 余能海。2024a。Opera：通过过度信任惩罚和回顾-分配缓解多模态大语言模型中的幻觉问题。发表于 *IEEE/CVF 计算机视觉与模式识别会议论文集*。13418–13427。

+   Huang 等人（2022a）黄文龙、Pieter Abbeel、Deepak Pathak 和 Igor Mordatch。2022a。语言模型作为零-shot 规划器：为具身智能体提取可操作的知识。发表于 *国际机器学习会议*。PMLR，9118–9147。

+   Huang 等人（2023a）黄杨斯波、Samyak Gupta、夏孟洲、李凯 和 陈丹琪。2023a。通过利用生成进行开源大语言模型的灾难性越狱。*arXiv 预印本 arXiv:2310.06987*（2023）。

+   Huang 等人（2023b）黄杨斯波、Samyak Gupta、钟泽轩、李凯 和 陈丹琪。2023b。基于检索的语言模型的隐私影响。发表于 *2023 年自然语言处理实证方法会议论文集*。14887–14902。

+   Huang 等人（2024b）黄一浩、王冲、贾晓俊、郭青、Felix Juefei-Xu、张建、蒲戈广 和 刘杨。2024b。面向 LLM 的通用目标劫持的语义引导提示组织。*arXiv 预印本 arXiv:2405.14189*（2024）。

+   Huang 等人（2023c）黄跃、张启慧、孙立超 等人。2023c。Trustgpt：一个面向可信赖和负责任的大语言模型的基准测试。*arXiv 预印本 arXiv:2306.11507*（2023）。

+   Hubinger 等人（2024）Evan Hubinger、Carson Denison 和 Jesse Mu 等人。2024。Uhgeval：通过无约束生成对中文大语言模型的幻觉进行基准测试。*arXiv 预印本 arXiv:2401.05566*（2024）。

+   Hui 等人（2024）Bo Hui、袁浩林、Neil Gong、Philippe Burlina 和 Yinzhi Cao。2024。PLeak：针对大语言模型应用的提示泄露攻击。*arXiv 预印本 arXiv:2405.06823*（2024）。

+   Jagielski 等人（2020）Matthew Jagielski、Nicholas Carlini、David Berthelot、Alex Kurakin 和 Nicolas Papernot。2020。高精度和高保真度的神经网络提取。发表于 *第29届 USENIX 安全研讨会（USENIX Security 20）*。1345–1362。

+   Jain 等人（2023）Neel Jain、Avi Schwarzschild、温玉欣、Gowthami Somepalli、John Kirchenbauer、Ping-yeh Chiang、Micah Goldblum、Aniruddha Saha、Jonas Geiping 和 Tom Goldstein。2023。对齐语言模型的对抗攻击基准防御。*arXiv 预印本 arXiv:2309.00614*（2023）。

+   Jang 等人（2023）Joel Jang、Dongkeun Yoon、Sohee Yang、Sungmin Cha、Moontae Lee、Lajanugen Logeswaran 和 Minjoon Seo。2023。知识遗忘：缓解语言模型隐私风险的手段。发表于 *第61届计算语言学协会年会，ACL 2023*。计算语言学协会，14389–14408。

+   Ji等人（2024）Jiabao Ji, Bairu Hou, Alexander Robey, George J Pappas, Hamed Hassani, Yang Zhang, Eric Wong, 和 Shiyu Chang. 2024. 通过语义平滑防御大语言模型免受越狱攻击。*arXiv预印本 arXiv:2402.16192*（2024）。

+   Ji等人（2023）Ziwei Ji, Nayeon Lee, Rita Frieske, Tiezheng Yu, Dan Su, Yan Xu, Etsuko Ishii, Ye Jin Bang, Andrea Madotto, 和 Pascale Fung. 2023. 自然语言生成中的幻觉调查。*计算机调查* 55, 12（2023），1–38。

+   Jiang等人（2024）Chaoya Jiang, Haiyang Xu, Mengfan Dong, Jiaxing Chen, Wei Ye, Ming Yan, Qinghao Ye, Ji Zhang, Fei Huang, 和 Shikun Zhang. 2024. 用于多模态大语言模型的幻觉增强对比学习。在*IEEE/CVF计算机视觉与模式识别会议论文集*。27036–27046。

+   Jiao等人（2024）Ruochen Jiao, Shaoyuan Xie, Justin Yue, Takami Sato, Lixu Wang, Yixuan Wang, Qi Alfred Chen, 和 Qi Zhu. 2024. 探索针对基于大语言模型的决策制定的后门攻击。*arXiv预印本 arXiv:2405.20774*（2024）。

+   Jin等人（2024）Mingyu Jin, Suiyuan Zhu, Beichen Wang, Zihao Zhou, Chong Zhang, Yongfeng Zhang 等人。2024. Attackeval：如何评估越狱攻击在大语言模型上的有效性。*arXiv预印本 arXiv:2401.09002*（2024）。

+   Kaneko和Bollegala（2022）Masahiro Kaneko 和 Danushka Bollegala. 2022. 揭示面具–评估掩蔽语言模型中的社会偏见。在*AAAI人工智能会议论文集*，第36卷。11954–11962。

+   Kaul等人（2024）Prannay Kaul, Zhizhong Li, Hao Yang, Yonatan Dukler, Ashwin Swaminathan, CJ Taylor, 和 Stefano Soatto. 2024. THRONE：一种基于对象的幻觉基准，用于大规模视觉-语言模型的自由形式生成。在*IEEE/CVF计算机视觉与模式识别会议论文集*。27228–27238。

+   Keller等人（2021）Yannik Keller, Jan Mackensen, 和 Steffen Eger. 2021. BERT-defense：一种基于BERT的概率模型，用于应对认知启发的正字法对抗性攻击。*arXiv预印本 arXiv:2106.01452*（2021）。

+   Kim等人（2023）Jae Myung Kim, A Koepke, Cordelia Schmid, 和 Zeynep Akata. 2023. 揭示和缓解跨模态检索中的虚假相关性。在*IEEE/CVF计算机视觉与模式识别会议论文集*。2585–2595。

+   Kim等人（2024）Siwon Kim, Sangdoo Yun, Hwaran Lee, Martin Gubri, Sungroh Yoon, 和 Seong Joon Oh. 2024. Propile：探测大语言模型中的隐私泄露。*神经信息处理系统进展* 36（2024）。

+   Kimura等人（2024）Subaru Kimura, Ryota Tanaka, Shumpei Miyawaki, Jun Suzuki, 和 Keisuke Sakaguchi. 2024. 大规模视觉-语言模型在目标劫持中的经验分析，通过视觉提示注入进行防御。*arXiv预印本 arXiv:2408.03554*（2024）。

+   Ko et al. (2023) Myeongseob Ko, Ming Jin, Chenguang Wang, 和 Ruoxi Jia. 2023. 面对大型多模态模型的实用成员推断攻击：初步研究。收录于 *IEEE/CVF 计算机视觉国际会议论文集*。4871–4881。

+   Kong (2024) Nicholas Ka-Shing Kong. 2024. *InjectBench: 一种间接提示注入基准框架*。博士论文。弗吉尼亚理工大学。

+   Krieg et al. (2023) Klara Krieg, Emilia Parada-Cabaleiro, Gertraud Medicus, Oleg Lesota, Markus Schedl, 和 Navid Rekabsaz. 2023. Grep-biasir：一个用于研究信息检索结果中性别表现偏见的数据集。收录于 *2023年人类信息交互与检索会议论文集*。444–448。

+   Krishna et al. (2019) Kalpesh Krishna, Gaurav Singh Tomar, Ankur P Parikh, Nicolas Papernot, 和 Mohit Iyyer. 2019. SESAME街的窃贼！基于BERT的API模型提取。*arXiv 预印本 arXiv:1910.12366* (2019)。

+   Lauscher et al. (2021) Anne Lauscher, Tobias Lueken, 和 Goran Glavaš. 2021. 语言模型的可持续模块化去偏。收录于 *计算语言学协会会议记录：EMNLP 2021*。4782–4797。

+   Le et al. (2020) Thai Le, Noseong Park, 和 Dongwon Lee. 2020. SHIELD: 使用随机多专家补丁器防御多种黑箱对抗攻击的文本神经网络。*arXiv 预印本 arXiv:2011.08908* (2020)。

+   Lee et al. (2018) Katherine Lee, Orhan Firat, Ashish Agarwal, Clara Fannjiang, 和 David Sussillo. 2018. 神经机器翻译中的幻觉现象。（2018）。

+   Lee et al. (2022) Katherine Lee, Daphne Ippolito, Andrew Nystrom, Chiyuan Zhang, Douglas Eck, Chris Callison-Burch, 和 Nicholas Carlini. 2022. 去重训练数据使语言模型表现更好。收录于 *第60届计算语言学协会年会会议录（第一卷：长篇论文）*。8424–8445。

+   Leng et al. (2024) Sicong Leng, Hang Zhang, Guanzheng Chen, Xin Li, Shijian Lu, Chunyan Miao, 和 Lidong Bing. 2024. 通过视觉对比解码缓解大型视觉语言模型中的物体幻觉。收录于 *IEEE/CVF 计算机视觉与模式识别会议论文集*。13872–13882。

+   Levi 和 Neumann (2024) Patrick Levi 和 Christoph P Neumann. 2024. 词汇攻击劫持大型语言模型应用。*arXiv 预印本 arXiv:2404.02637* (2024)。

+   Lewis et al. (2019) Mike Lewis, Yinhan Liu, Naman Goyal, Marjan Ghazvininejad, Abdelrahman Mohamed, Omer Levy, Ves Stoyanov, 和 Luke Zettlemoyer. 2019. Bart: 用于自然语言生成、翻译和理解的序列到序列去噪预训练。*arXiv 预印本 arXiv:1910.13461* (2019)。

+   Li et al. (2023a) Junyi Li, Xiaoxue Cheng, Wayne Xin Zhao, Jian-Yun Nie, 和 Ji-Rong Wen. 2023a. Halueval: 一个用于大型语言模型的规模化幻觉评估基准。*arXiv 预印本 arXiv:2305.11747* (2023)。

+   Li等人（2024a）Lin Li, Haoyan Guan, Jianing Qiu, 和 Michael Spratling. 2024a. 一个提示词足以增强预训练视觉-语言模型的对抗鲁棒性。在*IEEE/CVF计算机视觉与模式识别大会论文集*中，24408–24419页。

+   Li等人（2022a）Linyang Li, Demin Song, 和 Xipeng Qiu. 2022a. 作为防御对抗性攻击的文本对抗净化技术。*arXiv预印本arXiv:2203.14207*（2022年）。

+   Li等人（2022b）Xuechen Li, Florian Tramer, Percy Liang, 和 Tatsunori Hashimoto. 2022b. 大型语言模型可以成为强大的差分隐私学习者。在*国际学习表示大会*上。[https://openreview.net/forum?id=bVuP3ltATMz](https://openreview.net/forum?id=bVuP3ltATMz)

+   Li等人（2023b）Yingji Li, Mengnan Du, Xin Wang, 和 Ying Wang. 2023b. 提示调优推得更远，对比学习拉得更近：一种减轻社会偏见的两阶段方法。在*第61届计算语言学协会年会（ACL 2023）*中，计算语言学协会（ACL），14254–14267页。

+   Li等人（2023c）Yifan Li, Yifan Du, Kun Zhou, Jinpeng Wang, Wayne Xin Zhao, 和 Ji-Rong Wen. 2023c. 评估大型视觉-语言模型中的对象幻觉。*arXiv预印本arXiv:2305.10355*（2023年）。

+   Li等人（2024b）Yifan Li, Hangyu Guo, Kun Zhou, Wayne Xin Zhao, 和 Ji-Rong Wen. 2024b. 图像是对齐的阿基里斯之踵：利用视觉脆弱性破解多模态大型语言模型。*arXiv预印本arXiv:2403.09792*（2024年）。

+   Li等人（2024c）Yige Li, Hanxun Huang, Yunhan Zhao, Xingjun Ma, 和 Jun Sun. 2024c. BackdoorLLM：一个全面的基准测试用于评估大型语言模型的后门攻击。*arXiv预印本arXiv:2408.12798*（2024年）。

+   Li等人（2023d）Yufei Li, Zexin Li, Yingfan Gao, 和 Cong Liu. 2023d. 对话生成的白盒多目标对抗攻击。*arXiv预印本arXiv:2305.03655*（2023年）。

+   Li等人（2024d）Zongjie Li, Chaozheng Wang, Pingchuan Ma, Chaowei Liu, Shuai Wang, Daoyuan Wu, Cuiyun Gao, 和 Yang Liu. 2024d. 从大型语言模型中提取专业化代码能力：可行性研究。在*IEEE/ACM第46届国际软件工程会议论文集*中，1–13页。

+   Li等人（2023e）Zongjie Li, Chaozheng Wang, Shuai Wang, 和 Cuiyun Gao. 2023e. 通过水印保护基于大型语言模型的代码生成API的知识产权。在*2023年ACM SIGSAC计算机与通信安全会议论文集*中，2336–2350页。

+   Liang等人（2022）Percy Liang, Rishi Bommasani, Tony Lee, Dimitris Tsipras, Dilara Soylu, Michihiro Yasunaga, Yian Zhang, Deepak Narayanan, Yuhuai Wu, Ananya Kumar 等人. 2022. 对语言模型的全面评估。*arXiv预印本arXiv:2211.09110*（2022年）。

+   Liang 等人（2024b）Siyuan Liang、Kuanrong Liu、Jiajun Gong、Jiawei Liang、Yuan Xun、Ee-Chien Chang 和 Xiaochun Cao。2024b。遗忘后门威胁：通过局部令牌遗忘增强多模态对比学习中的后门防御。*arXiv 预印本 arXiv:2403.16257*（2024）。

+   Liang 等人（2023b）Siyuan Liang、Mingli Zhu、Aishan Liu、Baoyuan Wu、Xiaochun Cao 和 Ee-Chien Chang。2023b。BadCLIP：引导双嵌入的多模态对比学习后门攻击。*arXiv 预印本 arXiv:2311.12075*（2023）。

+   Liang 等人（2023a）Xun Liang、Shichao Song、Simin Niu、Zhiyu Li、Feiyu Xiong、Bo Tang、Zhaohui Wy、Dawei He、Peng Cheng、Zhonghao Wang 等人。2023a。Uhgeval：通过无约束生成评估中文大语言模型的幻觉。*arXiv 预印本 arXiv:2311.15296*（2023）。

+   Liang 等人（2024a）Zi Liang、Haibo Hu、Qingqing Ye、Yaxin Xiao 和 Haoyang Li。2024a。为什么我的提示被泄露？揭示定制化大语言模型中的提示提取威胁。*arXiv 预印本 arXiv:2408.02416*（2024）。

+   Lillicrap 等人（2015）Timothy P Lillicrap、Jonathan J Hunt、Alexander Pritzel、Nicolas Heess、Tom Erez、Yuval Tassa、David Silver 和 Daan Wierstra。2015。通过深度强化学习进行连续控制。*arXiv 预印本 arXiv:1509.02971*（2015）。

+   Limisiewicz 等人（2023）Tomasz Limisiewicz、David Mareček 和 Tomáš Musil。2023。通过模型适应进行去偏算法。*arXiv 预印本 arXiv:2310.18913*（2023）。

+   Lin 等人（2021）Stephanie Lin、Jacob Hilton 和 Owain Evans。2021。Truthfulqa：衡量模型如何模仿人类虚假信息。*arXiv 预印本 arXiv:2109.07958*（2021）。

+   Lison 等人（2021）Pierre Lison、Ildikó Pilán、David Sánchez、Montserrat Batet 和 Lilja Øvrelid。2021。文本数据的匿名化模型：现状、挑战和未来方向。收录于 *第59届计算语言学协会年会及第11届国际联合自然语言处理会议论文集（卷1：长篇论文）*，4188–4203。

+   Liu 等人（2024e）Aishan Liu、Yuguang Zhou 和 Xianglong Liu 等人。2024e。通过情境后门攻击破坏具身代理。*arXiv 预印本 arXiv:2408.02882*（2024）。

+   Liu 等人（2023a）Bo Liu、Yuqian Jiang、Xiaohan Zhang、Qiang Liu、Shiqi Zhang、Joydeep Biswas 和 Peter Stone。2023a。Llm+ p：赋能大语言模型以实现最优规划能力。*arXiv 预印本 arXiv:2304.11477*（2023）。

+   Liu 等人（2023b）Fuxiao Liu、Kevin Lin、Linjie Li、Jianfeng Wang、Yaser Yacoob 和 Lijuan Wang。2023b。通过稳健的指令调优缓解大规模多模态模型的幻觉问题。收录于 *第十二届国际学习表征会议*。

+   Liu et al. (2023d) 刘汉、吴宇昊、翟世轩、袁博和张宁。2023d。Riatig：具有自然提示的可靠且不可察觉的对抗性文本到图像生成。发表于 *IEEE/CVF计算机视觉与模式识别会议论文集*，20585–20594。

+   Liu (2022) 刘杰瑞。2022年。LlamaIndex。

+   Liu et al. (2022a) 刘嘉威、康扬扬、唐迪、宋凯松、孙长龙、王晓峰、吕伟和刘晓中。2022a。序列-无序：针对黑箱神经排名模型的模仿对抗攻击。发表于 *2022年ACM SIGSAC计算机与通信安全会议论文集*，2025–2039。

+   Liu et al. (2023c) 刘钦、王飞、肖超伟和陈慕豪。2023c。从捷径到触发器：使用去噪PoE进行后门防御。*arXiv预印本 arXiv:2305.14910*（2023年）。

+   Liu et al. (2022b) 刘钦、郑瑞、荣宝、刘静怡、刘志华、程占占、乔梁、桂涛、张琦和黄轩晶。2022b。Flooding-X：通过损失限制微调提升BERT对抗攻击的抗性。发表于 *第60届计算语言学协会年会论文集（卷1：长篇论文）*，5634–5644。

+   Liu et al. (2024b) 刘顺宇、李耀如、张孔程、崔振宇、方文凯、郑宇轩、郑同雅和宋名利。2024b。Odyssey：赋能代理人开放世界技能。*arXiv预印本 arXiv:2407.15325*（2024年）。

+   Liu et al. (2024d) 刘同、张英杰、赵哲、董寅鹏、孟国柱和陈凯。2024d。让它们提问并回答：通过伪装与重构在少量查询中破解大型语言模型。*arXiv预印本 arXiv:2402.18104*（2024年）。

+   Liu et al. (2023e) 刘晓耿、许楠、陈慕豪和肖超伟。2023e。Autodan：在对齐的大型语言模型上生成隐秘的越狱提示。*arXiv预印本 arXiv:2310.04451*（2023年）。

+   Liu et al. (2024c) 刘晓耿、余志远、张一哲、张宁和肖超伟。2024c。针对大型语言模型的自动化与普适化提示注入攻击。*arXiv预印本 arXiv:2403.04957*（2024年）。

+   Liu et al. (2023f) 刘晓、朱毅、顾俊、兰英、杨畅和乔宇。2023f。Mm-safetybench：多模态大型语言模型安全评估基准。*arXiv预印本 arXiv:2311.17600*（2023年）。

+   Liu et al. (2024a) 刘宇培、贾雨琪、耿润鹏、贾金源和龚振强·Neil Zhenqiang Gong。2024a。形式化和基准化提示注入攻击与防御。发表于 *第33届USENIX安全研讨会（USENIX Security 24）*，1831–1847。

+   Lu et al. (2023) 卢东、王志强、王腾、关伟力、高洪昌和郑锋。2023年。集合级引导攻击：提升视觉-语言预训练模型的对抗迁移性。发表于 *IEEE/CVF国际计算机视觉会议论文集*，102–111。

+   Lu 等人（2024a）Jiarui Lu、Thomas Holleis 和 Yizhe Zhang 等人。2024a年。ToolSandbox：一个有状态的、对话式的、交互式评估基准，用于大型语言模型的工具使用能力。 *arXiv 预印本 arXiv:2408.04682*（2024）。

+   Lu 等人（2024b）Jiaying Lu、Jinmeng Rao、Kezhen Chen、Xiaoyuan Guo、Yawen Zhang、Baochen Sun、Carl Yang 和 Jie Yang。2024b年。大型视觉-语言模型中语义基础的评估与增强。在 *AAAI-ReLM 研讨会*。

+   Lukas 等人（2023）Nils Lukas、Ahmed Salem、Robert Sim、Shruti Tople、Lukas Wutschitz 和 Santiago Zanella-Béguelin。2023年。分析语言模型中的个人可识别信息泄漏。在 *2023年 IEEE 安全与隐私研讨会（SP）*。IEEE，346–363。

+   Luo 等人（2024a）Weidi Luo、Siyuan Ma、Xiaogeng Liu、Xiaoyu Guo 和 Chaowei Xiao。2024a年。Jailbreakv-28k：评估多模态大型语言模型对越狱攻击鲁棒性的基准。 *arXiv 预印本 arXiv:2404.03027*（2024）。

+   Luo 等人（2024b）Wen Luo、Tianshu Shen、Wei Li、Guangyue Peng、Richeng Xuan、Houfeng Wang 和 Xi Yang。2024b年。HalluDial：一个用于自动化对话级幻觉评估的大规模基准。 *arXiv 预印本 arXiv:2406.07070*（2024）。

+   Maaz 等人（2023）Muhammad Maaz、Hanoona Rasheed、Salman Khan 和 Fahad Shahbaz Khan。2023年。Video-chatgpt：通过大型视觉和语言模型推动详细视频理解。 *arXiv 预印本 arXiv:2306.05424*（2023）。

+   Mao 等人（2024）Yuhao Mao、Mark Müller、Marc Fischer 和 Martin Vechev。2024年。连接认证与对抗训练。 *神经信息处理系统进展* 36（2024）。

+   May 等人（2019）Chandler May、Alex Wang、Shikha Bordia、Samuel R Bowman 和 Rachel Rudinger。2019年。关于在句子编码器中测量社会偏见。 *arXiv 预印本 arXiv:1903.10561*（2019）。

+   Medium（[n. d.]）Medium。[n. d.]。“祖母利用”：命令 ChatGPT 假装成已故的祖母。[https://medium.com/@med_strongboxit/grandma-exploit-chatgpt-commanded-to-pretend-to-be-a-dead-grandmother-13ddb984715a](https://medium.com/@med_strongboxit/grandma-exploit-chatgpt-commanded-to-pretend-to-be-a-dead-grandmother-13ddb984715a)。

+   Mehrabi 等人（2021）Ninareh Mehrabi、Fred Morstatter、Nripsuta Saxena、Kristina Lerman 和 Aram Galstyan。2021年。关于机器学习中的偏见与公平性的调查。 *ACM 计算机调查（CSUR）* 54, 6（2021），1–35。

+   Milli 等人（2019）Smitha Milli、Ludwig Schmidt、Anca D Dragan 和 Moritz Hardt。2019年。通过模型解释进行模型重建。在 *公平性、问责性和透明度会议论文集*，1–9。

+   Mireshghallah 等人（2024）Niloofar Mireshghallah、Hyunwoo Kim、Xuhui Zhou、Yulia Tsvetkov、Maarten Sap、Reza Shokri 和 Yejin Choi。2024年。大型语言模型能保守秘密吗？通过情境完整性理论测试语言模型的隐私影响。在 *第十二届国际学习表示会议*。

+   Mnih 等（2015）Volodymyr Mnih, Koray Kavukcuoglu, David Silver, Andrei A Rusu, Joel Veness, Marc G Bellemare, Alex Graves, Martin Riedmiller, Andreas K Fidjeland, Georg Ostrovski, 等. 2015. 通过深度强化学习实现人类级别的控制. *nature* 518, 7540 (2015), 529–533.

+   Mosca 等（2022）Edoardo Mosca, Shreyash Agarwal, Javier Rando, 和 Georg Groh. 2022. “那是一个可疑的反应！”：通过解读对数变动来检测NLP对抗性攻击. *arXiv 预印本 arXiv:2204.04636* (2022).

+   Nadeem 等（2020）Moin Nadeem, Anna Bethke, 和 Siva Reddy. 2020. StereoSet: 测量预训练语言模型中的刻板偏见. *arXiv 预印本 arXiv:2004.09456* (2020).

+   Nadeem 等（2021）Moin Nadeem, Anna Bethke, 和 Siva Reddy. 2021. StereoSet: 测量预训练语言模型中的刻板偏见. 在 *第59届计算语言学会年会暨第11届国际联合自然语言处理大会（卷1：长篇论文）* 中. 5356–5371.

+   Nakano 等（2021）Reiichiro Nakano, Jacob Hilton, Suchir Balaji, Jeff Wu, Long Ouyang, Christina Kim, Christopher Hesse, Shantanu Jain, Vineet Kosaraju, William Saunders, 等. 2021. WebGPT: 基于浏览器的问答系统与人类反馈. *arXiv 预印本 arXiv:2112.09332* (2021).

+   Nangia 等（2020）Nikita Nangia, Clara Vania, Rasika Bhalerao, 和 Samuel R Bowman. 2020. CrowS-Pairs: 测量掩蔽语言模型中的社会偏见的挑战数据集. 在 *2020年自然语言处理实证方法会议（EMNLP 2020）* 中. 计算语言学协会（ACL），1953–1967.

+   Naseh 等（2023）Ali Naseh, Kalpesh Krishna, Mohit Iyyer, 和 Amir Houmansadr. 2023. 窃取语言模型的解码算法. 在 *2023年ACM SIGSAC计算机与通信安全会议论文集* 中. 1835–1849.

+   Nasr 等（2023）Milad Nasr, Nicholas Carlini, Jonathan Hayase, Matthew Jagielski, A Feder Cooper, Daphne Ippolito, Christopher A Choquette-Choo, Eric Wallace, Florian Tramèr, 和 Katherine Lee. 2023. 从（生产）语言模型中可扩展地提取训练数据. *arXiv 预印本 arXiv:2311.17035* (2023).

+   Neel 等（2021）Seth Neel, Aaron Roth, 和 Saeed Sharifi-Malvajerdi. 2021. Descent-to-delete: 基于梯度的机器遗忘方法. 在 *算法学习理论* 中. PMLR, 931–962.

+   Nguyen 等（2022）Thanh Tam Nguyen, Thanh Trung Huynh, Phi Le Nguyen, Alan Wee-Chung Liew, Hongzhi Yin, 和 Quoc Viet Hung Nguyen. 2022. 机器遗忘的调查. *arXiv 预印本 arXiv:2209.02299* (2022).

+   Nissenbaum（2004）Helen Nissenbaum. 2004. 隐私作为情境完整性. *Wash. L. Rev.* 79 (2004), 119.

+   Niu 等（2024）Zhenxing Niu, Haodong Ren, Xinbo Gao, Gang Hua, 和 Rong Jin. 2024. 针对多模态大语言模型的越狱攻击. *arXiv 预印本 arXiv:2402.02309* (2024).

+   OpenAI (2023) OpenAI. 2023. ChatGPT 插件。

+   欧阳等人（2022）Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray 等人。2022年。通过人类反馈训练语言模型以遵循指令。*神经信息处理系统进展* 35 (2022), 27730–27744。

+   Papernot 等人（2016）Nicolas Papernot, Patrick McDaniel, Xi Wu, Somesh Jha 和 Ananthram Swami。2016年。蒸馏作为对抗深度神经网络的对抗扰动的一种防御手段。在*2016年IEEE安全与隐私研讨会（SP）*。IEEE，582–597。

+   Park 等人（2023）Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang 和 Michael S Bernstein。2023年。生成代理：人类行为的互动模拟。在*第36届ACM年度用户界面软件与技术研讨会论文集*。1–22。

+   Parrish 等人（2022）Alicia Parrish, Angelica Chen, Nikita Nangia, Vishakh Padmakumar, Jason Phang, Jana Thompson, Phu Mon Htut 和 Samuel Bowman。2022年。BBQ：一个手工构建的问答偏差基准。*计算语言学会成果：ACL 2022*（2022）。

+   Pasquini 等人（2024）Dario Pasquini, Martin Strohmeier 和 Carmela Troncoso。2024年。Neural Exec：学习（并从中学习）执行触发器用于提示注入攻击。*arXiv预印本 arXiv:2403.03792*（2024）。

+   Patil 等人（2024）Vaidehi Patil, Peter Hase 和 Mohit Bansal。2024年。敏感信息能从LLM中删除吗？防御提取攻击的目标。在*第十二届国际学习表示会议*。 [https://openreview.net/forum?id=7erlRDoaV8](https://openreview.net/forum?id=7erlRDoaV8)

+   Pearce 等人（2022）Hammond Pearce, Baleegh Ahmad, Benjamin Tan, Brendan Dolan-Gavitt 和 Ramesh Karri。2022年。键盘上睡着了吗？评估 GitHub Copilot 的代码贡献的安全性。在*2022年IEEE安全与隐私研讨会（SP）*。IEEE，754–768。

+   Peng 等人（2023a）Baolin Peng, Michel Galley, Pengcheng He, Hao Cheng, Yujia Xie, Yu Hu, Qiuyuan Huang, Lars Liden, Zhou Yu, Weizhu Chen 等人。2023a年。检查你的事实并再试一次：通过外部知识和自动化反馈改进大语言模型。*arXiv预印本 arXiv:2302.12813*（2023）。

+   Peng 等人（2023b）Wenjun Peng, Jingwei Yi, Fangzhao Wu, Shangxi Wu, Bin Zhu, Lingjuan Lyu, Binxing Jiao, Tong Xu, Guangzhong Sun 和 Xing Xie。2023b年。你在抄袭我的模型吗？通过后门水印保护大语言模型的版权。*arXiv预印本 arXiv:2305.10036*（2023）。

+   Perez 和 Ribeiro（2022）Fábio Perez 和 Ian Ribeiro。2022年。忽略之前的提示：语言模型的攻击技术。*arXiv预印本 arXiv:2211.09527*（2022）。

+   Piet等人（2023）Julien Piet、Maha Alrashed、Chawin Sitawarin、Sizhe Chen、Zeming Wei、Elizabeth Sun、Basel Alomair 和 David Wagner。2023。Jatmo：通过任务特定微调进行提示注入防御。*arXiv预印本arXiv:2312.17673*（2023）。

+   Qi等人（2024b）Peng Qi、Zehong Yan、Wynne Hsu 和 Mong Li Lee。2024b。SNIFFER：用于可解释的上下文外虚假信息检测的多模态大型语言模型。在 *IEEE/CVF计算机视觉与模式识别会议论文集* 中，13052–13062。

+   Qi等人（2024a）Xiangyu Qi、Kaixuan Huang、Ashwinee Panda、Peter Henderson、Mengdi Wang 和 Prateek Mittal。2024a。视觉对抗样本越狱对齐的大型语言模型。在 *AAAI人工智能会议论文集*，第38卷。21527–21536。

+   Qian等人（2023a）Chen Qian、Xin Cong、Cheng Yang、Weize Chen、Yusheng Su、Juyuan Xu、Zhiyuan Liu 和 Maosong Sun。2023a。用于软件开发的交流型代理。*arXiv预印本arXiv:2307.07924* 6（2023）。

+   Qian等人（2023b）Chen Qian、Wei Liu 和 Hongzhang Liu。2023b。ChatDev：用于软件开发的交流型代理。*arXiv预印本arXiv:2307.07924*（2023）。

+   Qiang等人（2023）Yao Qiang、Xiangyu Zhou 和 Dongxiao Zhu。2023。通过对抗性上下文学习劫持大型语言模型。*arXiv预印本arXiv:2311.09948*（2023）。

+   Radford等人（2019）Alec Radford、Jeffrey Wu、Rewon Child、David Luan、Dario Amodei、Ilya Sutskever 等。2019。语言模型是无监督的多任务学习者。*OpenAI博客* 1, 8（2019），9。

+   Rajpurkar等人（2016）Pranav Rajpurkar、Jian Zhang、Konstantin Lopyrev 和 Percy Liang。2016。Squad：用于机器理解文本的100,000+问题。*arXiv预印本arXiv:1606.05250*（2016）。

+   Ram等人（2023）Ori Ram、Yoav Levine、Itay Dalmedigos、Dor Muhlgay、Amnon Shashua、Kevin Leyton-Brown 和 Yoav Shoham。2023。上下文检索增强的语言模型。*计算语言学会会刊* 11（2023），1316–1331。

+   Ran等人（2024）Delong Ran、Jinyuan Liu、Yichen Gong、Jingyi Zheng、Xinlei He、Tianshuo Cong 和 Anyu Wang。2024。JailbreakEval：一个用于评估针对大型语言模型的越狱尝试的集成工具包。*arXiv预印本arXiv:2406.09321*（2024）。

+   Ranaldi等人（2023）Leonardo Ranaldi、Elena Sofia Ruzzetti、Davide Venditti、Dario Onorati 和 Fabio Massimo Zanzotto。2023。走向公平：大型语言模型中的偏差与去偏差。*arXiv预印本arXiv:2305.13862*（2023）。

+   法规（2016）保护法规。2016。欧洲议会和理事会的法规（EU）2016/679。*法规（eu）* 679（2016），2016。

+   Rohrbach等人（2018）Anna Rohrbach、Lisa Anne Hendricks、Kaylee Burns、Trevor Darrell 和 Kate Saenko。2018。图像描述中的物体幻觉。*arXiv预印本arXiv:1809.02156*（2018）。

+   Roy 等人（2023）Sayak Saha Roy、Poojitha Thota、Krishna Vamsi Naragam 和 Shirin Nilizadeh。2023年。从聊天机器人到钓鱼机器人？–防止使用 ChatGPT、Google Bard 和 Claude 创建的网络钓鱼骗局。*arXiv 预印本 arXiv:2310.19181*（2023年）。

+   Ruan 等人（2023）Yangjun Ruan、Honghua Dong 和 Andrew Wang 等人。2023年。通过 LM 模拟沙箱识别 LM 代理的风险。*arXiv 预印本 arXiv:2309.15817*（2023年）。

+   Santhanam 等人（2021）Sashank Santhanam、Behnam Hedayatnia、Spandana Gella、Aishwarya Padmakumar、Seokhwan Kim、Yang Liu 和 Dilek Hakkani-Tur。2021年。罗马是1776年建成的：基于事实正确性的知识驱动响应生成案例研究。*arXiv 预印本 arXiv:2110.05456*（2021年）。

+   Saunders 等人（2022）Danielle Saunders、Rosie Sallis 和 Bill Byrne。2022年。首先是最糟糕的：在束搜索过程中找到更好的性别翻译。发表于 *计算语言学协会会议成果：ACL 2022*。3814–3823。

+   Schick 等人（2024）Timo Schick、Jane Dwivedi-Yu、Roberto Dessì、Roberta Raileanu、Maria Lomeli、Eric Hambro、Luke Zettlemoyer、Nicola Cancedda 和 Thomas Scialom。2024年。Toolformer：语言模型可以自学使用工具。*神经信息处理系统进展* 36（2024年）。

+   Schulman 等人（2017）John Schulman、Filip Wolski、Prafulla Dhariwal、Alec Radford 和 Oleg Klimov。2017年。近端策略优化算法。*arXiv 预印本 arXiv:1707.06347*（2017年）。

+   Sha 和 Zhang（2024）Zeyang Sha 和 Yang Zhang。2024年。针对大型语言模型的提示盗窃攻击。*arXiv 预印本 arXiv:2402.12959*（2024年）。

+   Shamir 等人（2023）Adi Shamir、Isaac Canales-Martinez、Anna Hambitzer、Jorge Chavez-Saab、Francisco Rodrigez-Henriquez 和 Nitin Satpute。2023年。多项式时间的神经网络模型密码分析提取。*arXiv 预印本 arXiv:2310.08708*（2023年）。

+   Sharma 等人（2024）Reshabh K Sharma、Vinayak Gupta 和 Dan Grossman。2024年。SPML：一种防御语言模型免受提示攻击的领域特定语言。*arXiv 预印本 arXiv:2402.11755*（2024年）。

+   Shayegani 等人（2023）Erfan Shayegani、Yue Dong 和 Nael Abu-Ghazaleh。2023年。碎片中的越狱：对多模态语言模型的组合对抗攻击。发表于 *第十二届国际学习表征会议*。

+   Shen 等人（2021）Lujia Shen、Shouling Ji、Xuhong Zhang、Jinfeng Li、Jing Chen、Jie Shi、Chengfang Fang、Jianwei Yin 和 Ting Wang。2021年。后门预训练模型可以转移到所有模型。*arXiv 预印本 arXiv:2111.00197*（2021年）。

+   Shen 等人（2023）Lujia Shen、Yuwen Pu、Shouling Ji、Changjiang Li、Xuhong Zhang、Chunpeng Ge 和 Ting Wang。2023年。通过动态注意力提高基于变换器的大型语言模型的鲁棒性。*arXiv 预印本 arXiv:2311.17400*（2023年）。

+   Shen et al. (2024a) Xinyue Shen, Zeyuan Chen, Michael Backes, Yun Shen, and Yang Zhang. 2024a. “现在做任何事”：表征和评估大型语言模型的野外越狱提示。在*ACM SIGSAC计算机与通信安全会议（CCS）*。ACM。

+   Shen et al. (2024b) Xinyue Shen, Yiting Qu, Michael Backes, and Yang Zhang. 2024b. 对$\{$文本到图像$\}$生成模型的提示盗用攻击。在*第33届USENIX安全研讨会（USENIX Security 24）*。5823–5840。

+   Shen et al. (2024c) Yongliang Shen, Kaitao Song, Xu Tan, Dongsheng Li, Weiming Lu, and Yueting Zhuang. 2024c. Hugginggpt：利用ChatGPT和Hugging Face的朋友们解决AI任务。*神经信息处理系统进展* 36 (2024)。

+   Shi et al. (2023a) Weijia Shi, Anirudh Ajith, Mengzhou Xia, Yangsibo Huang, Daogao Liu, Terra Blevins, Danqi Chen, and Luke Zettlemoyer. 2023a. 检测大型语言模型的预训练数据。*arXiv预印本 arXiv:2310.16789* (2023)。

+   Shi et al. (2023b) Weijia Shi, Xiaochuang Han, Mike Lewis, Yulia Tsvetkov, Luke Zettlemoyer, and Scott Wen-tau Yih. 2023b. 信任你的证据：通过上下文感知解码减少幻觉。*arXiv预印本 arXiv:2305.14739* (2023)。

+   Shokri et al. (2017) Reza Shokri, Marco Stronati, Congzheng Song, and Vitaly Shmatikov. 2017. 针对机器学习模型的成员身份推断攻击。在*2017年IEEE安全与隐私研讨会（SP）*。IEEE，3–18。

+   Smith et al. (2022) Eric Michael Smith, Melissa Hall, Melanie Kambadur, Eleonora Presani, and Adina Williams. 2022. “我很抱歉听到这个”：通过整体描述符数据集发现语言模型中的新偏见。*arXiv预印本 arXiv:2205.09209* (2022)。

+   Somepalli et al. (2023a) Gowthami Somepalli, Vasu Singla, Micah Goldblum, Jonas Geiping, and Tom Goldstein. 2023a. 扩散艺术还是数字伪造？调查扩散模型中的数据复制问题。在*IEEE/CVF计算机视觉与模式识别大会论文集*。6048–6058。

+   Somepalli et al. (2023b) Gowthami Somepalli, Vasu Singla, Micah Goldblum, Jonas Geiping, and Tom Goldstein. 2023b. 理解和缓解扩散模型中的复制问题。*神经信息处理系统进展* 36 (2023)，47783–47803。

+   Song et al. (2023) Chan Hee Song, Jiaman Wu, Clayton Washington, Brian M Sadler, Wei-Lun Chao, and Yu Su. 2023. Llm-planner：使用大型语言模型进行少量样本的具身智能体规划。在*IEEE/CVF国际计算机视觉会议论文集*。2998–3009。

+   Sun et al. (2023a) Bin Sun, Yitong Li, Fei Mi, Fanhu Bie, Yiwei Li, and Kan Li. 2023a. 通过增强和对比知识对话减少知识基础对话生成中的幻觉。在*第61届计算语言学协会年会论文集（第二卷：简短论文）*。1741–1750。

+   Sun 等（2023b）Hao Sun, Zhexin Zhang, Jiawen Deng, Jiale Cheng, 和 Minlie Huang。2023b。中文大语言模型的安全评估。*arXiv 预印本 arXiv:2304.10436*（2023）。

+   Sun 等（2024b）Xiongtao Sun, Deyue Zhang, Dongdong Yang, Quanchen Zou, 和 Hui Li。2024b。从第一原理出发对大语言模型的多轮上下文越狱攻击。*arXiv 预印本 arXiv:2408.04686*（2024）。

+   Sun 等（2024a）Yuhong Sun, Zhangyue Yin, Qipeng Guo, Jiawen Wu, Xipeng Qiu, 和 Hui Zhao。2024a。基于无法回答的数学题目对大语言模型的幻觉进行基准测试。*arXiv 预印本 arXiv:2403.03558*（2024）。

+   Supertools（[无日期]）Supertools。 [无日期]。*通过 Supertools 发现最佳 AI 工具*。 [https://supertools.therundown.ai/](https://supertools.therundown.ai/) 访问日期：2024-10-28。

+   Suresh 和 Guttag（2019）Harini Suresh 和 John V Guttag。2019。理解机器学习意外后果的框架。*arXiv 预印本 arXiv:1901.10002* 2, 8（2019），73。

+   Szegedy 等（2013）Christian Szegedy, Wojciech Zaremba, Ilya Sutskever, Joan Bruna, Dumitru Erhan, Ian Goodfellow, 和 Rob Fergus。2013。神经网络的引人入胜的特性。*arXiv 预印本 arXiv:1312.6199*（2013）。

+   Tang 等（2024c）Kunsheng Tang, Wenbo Zhou, Jie Zhang, Aishan Liu, Gelei Deng, Shuai Li, Peigui Qi, Weiming Zhang, Tianwei Zhang, 和 Nenghai Yu。2024c。GenderCARE：评估和减少大语言模型中的性别偏见的综合框架。发表于 *2024年ACM SIGSAC计算机与通信安全会议论文集*。

+   Tang 等（2024a）Xinyu Tang, Richard Shin, Huseyin A Inan, Andre Manoel, Fatemehsadat Mireshghallah, Zinan Lin, Sivakanth Gopi, Janardhan Kulkarni, 和 Robert Sim。2024a。具有差分隐私的少样本生成中的隐私保护上下文学习。发表于 *第十二届国际学习表征会议*。 [https://openreview.net/forum?id=oZtt0pRnOl](https://openreview.net/forum?id=oZtt0pRnOl)

+   Tang 等（2024b）Xinyu Tang, Richard Shin, Huseyin A Inan, Andre Manoel, Fatemehsadat Mireshghallah, Zinan Lin, Sivakanth Gopi, Janardhan Kulkarni, 和 Robert Sim。2024b。具有差分隐私的少样本生成中的隐私保护上下文学习。发表于 *第十二届国际学习表征会议*。

+   Touvron 等（2023）Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale 等。2023。Llama 2：开放基础模型与微调聊天模型。*arXiv 预印本 arXiv:2307.09288*（2023）。

+   Uluoglakci 和 Temizel（2024）Cem Uluoglakci 和 Tugba Taskaya Temizel. 2024. HypoTermQA：用于基准测试大语言模型幻觉倾向的假设术语数据集。*arXiv 预印本 arXiv:2402.16211*（2024）。

+   欧盟（2024）欧盟。2024。欧盟人工智能法案。 [https://artificialintelligenceact.eu/](https://artificialintelligenceact.eu/)。

+   Ustun et al. (2019) Berk Ustun, Yang Liu, 和 David Parkes. 2019. 无害公平性：具有偏好保证的解耦分类器。发表于 *国际机器学习会议*。PMLR，6373-6382。

+   van den Burg 和 Williams (2021) Gerrit van den Burg 和 Chris Williams. 2021. 概率深度生成模型中的记忆化问题。 *神经信息处理系统进展* 34 (2021)，27916-27928。

+   Van der Poel et al. (2022) Liam Van der Poel, Ryan Cotterell, 和 Clara Meister. 2022. 互信息缓解抽象摘要中的幻觉现象。 *arXiv 预印本 arXiv:2210.13210* (2022)。

+   Vaswani et al. (2017) Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N Gomez, Łukasz Kaiser, 和 Illia Polosukhin. 2017. 注意力即你所需要的一切。 *神经信息处理系统进展* 30 (2017)。

+   Venkit et al. (2023) Pranav Narayanan Venkit, Sanjana Gautam, Ruchi Panchanadikar, Ting-Hao Huang, 和 Shomir Wilson. 2023. 文本生成中的国籍偏见。发表于 *第17届欧洲计算语言学会章节会议论文集*。116-122。

+   Wallace et al. (2024) Eric Wallace, Kai Xiao, Reimar Leike, Lilian Weng, Johannes Heidecke, 和 Alex Beutel. 2024. 指令层级：训练LLM以优先考虑特权指令。 *arXiv 预印本 arXiv:2404.13208* (2024)。

+   Wang et al. (2020) Boxin Wang, Shuohang Wang, Yu Cheng, Zhe Gan, Ruoxi Jia, Bo Li, 和 Jingjing Liu. 2020. Infobert: 从信息理论的角度提高语言模型的鲁棒性。 *arXiv 预印本 arXiv:2010.02329* (2020)。

+   Wang et al. (2023d) Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao, Yuke Zhu, Linxi Fan, 和 Anima Anandkumar. 2023d. Voyager: 一个具有大型语言模型的开放式具身智能体。 *arXiv 预印本 arXiv:2305.16291* (2023)。

+   Wang et al. (2024a) Haodi Wang, Kai Dong, Zhilei Zhu, Haotong Qin, Aishan Liu, Xiaolin Fang, Jiakai Wang, 和 Xianglong Liu. 2024a. 针对视觉语言预训练模型的可转移多模态攻击。发表于 *2024 IEEE安全与隐私研讨会（SP）*。IEEE计算机学会，102-102。

+   Wang et al. (2024c) Haichen Wang, Shuchao Pang, Zhigang Lu, Yihang Rao, Yongbin Zhou, 和 Minhui Xue. 2024c. dp-promise: 用于图像合成的差分隐私扩散概率模型。发表于 *USENIX*。

+   Wang et al. (2024d) Huanqian Wang, Yang Yue, Rui Lu, Jingxin Shi, Andrew Zhao, Shenzhi Wang, Shiji Song, 和 Gao Huang. 2024d. 模型手术：通过简单的参数编辑调节LLM行为。 *arXiv 预印本 arXiv:2407.08770* (2024)。

+   Wang et al. (2023b) Junyang Wang, Yuhang Wang, Guohai Xu, Jing Zhang, Yukai Gu, Haitao Jia, Ming Yan, Ji Zhang, 和 Jitao Sang. 2023b. 一个不依赖LLM的多维基准用于MLLM幻觉评估。 *arXiv 预印本 arXiv:2311.07397* (2023)。

+   Wang 等人（2023c）Wang Jiongxiao, Wu Junlin, Chen Muhao, Vorobeychik Yevgeniy 和 Xiao Chaowei. 2023c. RLHFPoison: 针对带有人类反馈的强化学习的奖励中毒攻击。*arXiv 预印本 arXiv:2311.09641*（2023）。

+   Wang 等人（2023e）Wang Junyang, Zhou Yiyang, Xu Guohai, Shi Pengcheng, Zhao Chenlin, Xu Haiyang, Ye Qinghao, Yan Ming, Zhang Ji, Zhu Jihua 等人. 2023e. 大型视觉语言模型中的幻觉评估与分析。*arXiv 预印本 arXiv:2308.15126*（2023）。

+   Wang 等人（2024b）Wang Lei, Ma Chen, Feng Xueyang, Zhang Zeyu, Yang Hao, Zhang Jingsen, Chen Zhiyuan, Tang Jiakai, Chen Xu, Lin Yankai 等人. 2024b. 基于大型语言模型的自主代理综述。*计算机科学前沿* 18, 6（2024），186345。

+   Wang 等人（2024e）Wang Mengru, Zhang Ningyu, Xu Ziwen, Xi Zekun, Deng Shumin, Yao Yunzhi, Zhang Qishen, Yang Linyi, Wang Jindong 和 Chen Huajun. 2024e. 通过知识编辑解毒大型语言模型。*arXiv 预印本 arXiv:2403.14472*（2024）。

+   Wang 等人（2021）Wang Yue, Wang Weishi, Joty Shafiq 和 Steven CH Hoi. 2021. Codet5: 面向代码理解与生成的标识符感知统一预训练编码器-解码器模型。*arXiv 预印本 arXiv:2109.00859*（2021）。

+   Wang 等人（2023a）Wang Zhaoyang, Liu Zhiyue, Zheng Xiaopeng, Su Qinliang 和 Wang Jiahai. 2023a. Rmlm: 一种灵活的防御框架，用于主动缓解词级对抗攻击。发表于*第61届计算语言学协会年会论文集（第1卷：长篇论文）*。2757–2774。

+   Watson 等人（2022）Lauren Watson, Chuan Guo, Graham Cormode 和 Alexandre Sablayrolles. 2022. 关于会员推断攻击中难度校准的重要性。发表于*国际学习表征会议*。[https://openreview.net/forum?id=3eIrli0TwQ](https://openreview.net/forum?id=3eIrli0TwQ)

+   Webster 等人（2020）Webster Kellie, Wang Xuezhi, Tenney Ian, Beutel Alex, Pitler Emily, Pavlick Ellie, Chen Jilin, Chi Ed 和 Petrov Slav. 2020. 测量并减少预训练模型中的性别相关性。*arXiv 预印本 arXiv:2010.06032*（2020）。

+   Wei 等人（2024）Wei Cheng’an, Chen Kai, Zhao Yue, Gong Yujia, Xiang Lu 和 Zhu Shenchen. 2024. 大型语言模型中的上下文注入攻击。*arXiv 预印本 arXiv:2405.20234*（2024）。

+   Wei 等人（2022）Wei Jason, Wang Xuezhi, Schuurmans Dale, Bosma Maarten, Ichter Brian, Xia Fei, Chi Ed, Le Quoc 和 Zhou Denny. 2022. 链式思维提示在大型语言模型中的推理启发。*arXiv 预印本 arXiv:2201.11903*（2022）。

+   Wen 等人（2024）Wen Yuxin, Liu Yuchen, Chen Chen 和 Lyu Lingjuan. 2024. 检测、解释并缓解扩散模型中的记忆化问题。发表于*第十二届国际学习表征会议*。

+   Wilkins（2014）David E Wilkins. 2014. *实用规划：扩展经典人工智能规划范式*。Elsevier。

+   Wolf et al. (2023) 约塔姆·沃尔夫, 诺阿姆·威斯, 奥什里·阿夫内里, 约阿夫·莱文, 阿姆农·沙舒亚. 2023. 大型语言模型对齐的基本局限性. *arXiv预印本 arXiv:2304.11082* (2023).

+   Wu et al. (2023) 吴健, 亚谢什·高尔, 陈卓, 周龙, 朱怡萌, 王天瑞, 李金玉, 刘书杰, 任波, 刘林泉, 等. 2023. 关于解码器仅架构在语音转文本与大型语言模型集成中的应用. 见 *2023 IEEE自动语音识别与理解研讨会（ASRU）*. IEEE, 1–8.

+   Wu et al. (2024b) 吴晶歌, 金云秀, 吴鸿翰. 2024b. 医疗视觉问答中的幻觉基准测试. *arXiv预印本 arXiv:2401.05827* (2024).

+   Wu et al. (2024c) 童吴, 阿什维尼·潘达, 贾晨·T·王, 普拉提克·米塔尔. 2024c. 面向大型语言模型的隐私保护上下文学习. 见 *第十二届国际学习表征会议*. [https://openreview.net/forum?id=x4OPJ7lHVU](https://openreview.net/forum?id=x4OPJ7lHVU)

+   Wu et al. (2024a) 吴希扬, 关天锐, 李殿齐, 黄帅毅, 刘晓宇, 王锡君, 鲜瑞琪, 阿比纳夫·施里瓦斯塔瓦, 黄芙蓉, 乔丹·李·博伊德-格雷伯, 等. 2024a. AUTOHALLUSION: 面向视觉-语言模型的幻觉基准自动生成. *arXiv预印本 arXiv:2406.10900* (2024).

+   Wu et al. (2021) 吴贤, 郭文博, 魏华, 邢新宇. 2021. 针对深度强化学习的对抗性策略训练. 见 *第30届USENIX安全研讨会（USENIX Security 21）*. 1883–1900.

+   Xiang et al. (2024) 项崇, 童吴, 钟泽璇, 大卫·瓦格纳, 陈丹琦, 普拉提克·米塔尔. 2024. 可证明稳健的RAG对抗检索腐败. *arXiv预印本 arXiv:2405.15556* (2024).

+   Xie and Lukasiewicz (2023) 谷忠彬, 托马斯·卢卡谢维茨. 2023. 预训练语言模型去偏方法的实证分析. *arXiv预印本 arXiv:2306.04067* (2023).

+   Xu et al. (2021) 许琼凯, 何轩理, 吕灵娟, 屈李震, 哈法里·高拉姆雷扎. 2021. 学生超越教师：黑盒NLP API的模仿攻击. *arXiv预印本 arXiv:2108.13873* (2021).

+   Xu et al. (2024a) 许张晨, 姜丰青, 牛路遥, 贾金源, 林宇辰, 拉达·普文德兰. 2024a. Safedecoding: 通过安全感知解码防御越狱攻击. *arXiv预印本 arXiv:2402.08983* (2024).

+   Xu et al. (2024b) 许赵, 刘凡, 刘昊. 2024b. 伎俩袋：大型语言模型上的越狱攻击基准测试. *arXiv预印本 arXiv:2406.09324* (2024).

+   Yang et al. (2023b) 杨柯, 余查尔斯, 黄瑞依, 李曼玲, 季恒. 2023b. Adept: 一种去偏提示框架. 见 *人工智能AAAI会议论文集*, 第37卷, 10780–10788.

+   Yang et al. (2024a) 杨文凯, 毕晓寒, 林燕凯, 陈思硕, 周杰, 孙旭. 2024a. 小心你的代理！研究LLM基础代理的后门威胁. *arXiv预印本 arXiv:2402.11208* (2024).

+   Yang 等人（2024b）怡君 Yang、瑞源 Gao、小 Yang、建源 Zhong 和强 Xu。2024b年。GuardT2I：保护文本到图像模型免受对抗性提示攻击。*arXiv 预印本 arXiv:2403.01446*（2024年）。

+   Yang 等人（2024c）永 Yang、旭宏 Zhang、毅 Jiang、熙 Chen、浩宇 Wang、守灵 Ji 和宗慧 Wang。2024c年。PRSA：针对大型语言模型的提示逆窃取攻击。*arXiv 预印本 arXiv:2402.19200*（2024年）。

+   Yang 等人（2023a）正源 Yang、林杰 Li、剑锋 Wang、凯文 Lin、埃善 Azarnasab、费萨尔 Ahmed、子成 Liu、策 Liu、Michael Zeng 和丽娟 Wang。2023a年。Mm-react：通过提示 ChatGPT 进行多模态推理与行动。*arXiv 预印本 arXiv:2303.11381*（2023年）。

+   Yao 等人（2023）洪伟 Yao、建 Lou、奎 Ren 和展 Qin。2023年。Promptcare：通过水印注入与验证进行提示版权保护。*arXiv 预印本 arXiv:2308.02816*（2023年）。

+   Yao 等人（2024）顺宇 Yao、奠 Yu、杰弗里 Zhao、Izhak Shafran、Tom Griffiths、远 Cao、Karthik Narasimhan。2024年。思维树：通过大型语言模型进行深思熟虑的问题解决。*神经信息处理系统进展* 36（2024年）。

+   Yi 等人（2023）金伟 Yi、跃琪 Xie、斌 Zhu、Keegan Hines、Emre Kiciman、广忠 Sun、兴 Xie 和方昭 Wu。2023年。对大型语言模型的间接提示注入攻击进行基准测试与防御。*arXiv 预印本 arXiv:2312.14197*（2023年）。

+   Yin 等人（2024）子怡 Yin、沐超 Ye、天荣 Zhang、天宇 Du、金果 Zhu、寒 Liu、静辉 Chen、婷 Wang 和风龙 Ma。2024年。Vlattack：通过预训练模型进行的视觉-语言任务的多模态对抗攻击。*神经信息处理系统进展* 36（2024年）。

+   Yu 等人（2023a）查尔斯 Yu、Sullam Jeoung、Anish Kasi、彭飞 Yu 和恒 Ji。2023a年。通过划分梯度来解除语言模型中的偏见。在*计算语言学协会的发现：ACL 2023*。6032-6048。

+   Yu 等人（2022）大 Yu、Saurabh Naik、Arturs Backurs、Sivakanth Gopi、Huseyin A Inan、Gautam Kamath、Janardhan Kulkarni、尹 Tat Lee、Andre Manoel、Lukas Wutschitz、谢尔盖 Yekhanin 和惠帅 Zhang。2022年。语言模型的差分隐私微调。在*国际学习表征大会*。 [https://openreview.net/forum?id=Q42f0dfjECO](https://openreview.net/forum?id=Q42f0dfjECO)

+   Yu 等人（2023b）家浩 Yu、兴伟 Lin 和新宇 Xing。2023b年。Gptfuzzer：使用自动生成的越狱提示对大型语言模型进行红队测试。*arXiv 预印本 arXiv:2309.10253*（2023年）。

+   Yu 等人（2023d）家浩 Yu、玉航 Wu、东 Shu、明宇 Jin 和新宇 Xing。2023d年。评估 200+ 个定制 GPT 模型中的提示注入风险。*arXiv 预印本 arXiv:2311.11538*（2023年）。

+   Yu 等人（2024）Qifan Yu, Juncheng Li, Longhui Wei, Liang Pang, Wentao Ye, Bosheng Qin, Siliang Tang, Qi Tian 和 Yueting Zhuang. 2024. Hallucidoctor：减轻视觉指令数据中的幻觉毒性. 收录于 *IEEE/CVF计算机视觉与模式识别会议论文集*，12944–12953。

+   Yu 等人（2023c）Weichen Yu, Tianyu Pang, Qian Liu, Chao Du, Bingyi Kang, Yan Huang, Min Lin 和 Shuicheng Yan. 2023c. 训练数据提取的技巧集：从语言模型中提取数据. 收录于 *国际机器学习会议*，PMLR，40306–40320。

+   Yue 等人（2021）Xiang Yue, Minxin Du, Tianhao Wang, Yaliang Li, Huan Sun 和 Sherman SM Chow. 2021. 通过自然文本清理实现文本分析的差分隐私. 收录于 *计算语言学协会年会成果：ACL-IJCNLP 2021*，3853–3866。

+   Yue 等人（2023）Xiang Yue, Huseyin Inan, Xuechen Li, Girish Kumar, Julia McAnallen, Hoda Shajari, Huan Sun, David Levitan 和 Robert Sim. 2023. 具有差分隐私的合成文本生成：一种简单实用的方案. 收录于 *第61届计算语言学协会年会（第1卷：长篇论文）*，1321–1342。

+   Zanella-Beguelin 等人（2021）Santiago Zanella-Beguelin, Shruti Tople, Andrew Paverd 和 Boris Köpf. 2021. 自然语言模型的灰盒提取. 收录于 *国际机器学习会议*，PMLR，12278–12286。

+   Zayed 等人（2023）Abdelrahman Zayed, Prasanna Parthasarathi, Gonçalo Mordido, Hamid Palangi, Samira Shabanian 和 Sarath Chandar. 2023. 健康数据饮食上的深度学习：寻找公平性的重要示例. 收录于 *AAAI人工智能会议论文集*，第37卷，14593–14601。

+   Zeng 等人（2024a）Qingbin Zeng, Qinglong Yang, Shunan Dong, Heming Du, Liang Zheng, Fengli Xu 和 Yong Li. 2024a. 感知、反思与规划：设计用于无指令目标导向城市导航的LLM代理. *arXiv 预印本 arXiv:2408.04168*（2024）。

+   Zeng 等人（2024b）Shenglai Zeng, Jiankun Zhang, Pengfei He, Jie Ren, Tianqi Zheng, Hanqing Lu, Han Xu, Hui Liu, Yue Xing 和 Jiliang Tang. 2024b. 通过纯合成数据缓解检索增强生成（RAG）中的隐私问题. *arXiv 预印本 arXiv:2406.14773*（2024）。

+   Zeng 等人（2024c）Shenglai Zeng, Jiankun Zhang, Pengfei He, Yue Xing, Yiding Liu, Han Xu, Jie Ren, Shuaiqiang Wang, Dawei Yin, Yi Chang 等人. 2024c. 好与坏：探索检索增强生成（RAG）中的隐私问题. *arXiv 预印本 arXiv:2402.16893*（2024）。

+   Zha 等人（2023）Yuheng Zha, Yichi Yang, Ruichen Li 和 Zhiting Hu. 2023. AlignScore：通过统一的对齐函数评估事实一致性. *arXiv 预印本 arXiv:2305.16739*（2023）。

+   Zhan 等人（2024）Qiusi Zhan, Zhixiang Liang, Zifan Ying 和 Daniel Kang. 2024. Injecagent：基准测试在工具集成的大型语言模型代理中的间接提示注入. *arXiv 预印本 arXiv:2403.02691*（2024）。

+   张等（2024g）Collin Zhang, John X Morris, 和 Vitaly Shmatikov. 2024g. 通过反转LLM输出提取提示语。*arXiv 预印本 arXiv:2405.15012*（2024）。

+   张等（2024d）Hanrong Zhang, Jingyuan Huang, Kai Mei, Yifei Yao, Zhenting Wang, Chenlu Zhan, Hongwei Wang, 和 Yongfeng Zhang. 2024d. Agent Security Bench (ASB): 正式化并基准化LLM基础代理中的攻击与防御。*arXiv 预印本 arXiv:2410.02644*（2024）。

+   张等（2024e）Kechi Zhang, Jia Li, Ge Li, Xianjie Shi, 和 Zhi Jin. 2024e. CodeAgent: 利用工具集成代理系统增强代码生成，解决真实世界仓库级编程挑战。*arXiv 预印本 arXiv:2401.07339*（2024）。

+   张等（2016）Lu Zhang, Yongkai Wu, 和 Xintao Wu. 2016. 发现并消除直接与间接歧视的因果框架。*arXiv 预印本 arXiv:1611.07509*（2016）。

+   张等（2023c）Muru Zhang, Ofir Press, William Merrill, Alisa Liu, 和 Noah A Smith. 2023c. 语言模型幻觉如何滚雪球式发展。*arXiv 预印本 arXiv:2305.13534*（2023）。

+   张等（2024f）Rui Zhang, Hongwei Li, Rui Wen, Wenbo Jiang, Yuan Zhang, Michael Backes, Yun Shen, 和 Yang Zhang. 2024f. RLHFPoison: 针对大语言模型中的强化学习与人类反馈的奖励污染攻击。*arXiv 预印本 arXiv:2402.09179*（2024）。

+   张等（2022）Susan Zhang, Stephen Roller, Naman Goyal, Mikel Artetxe, Moya Chen, Shuohui Chen, Christopher Dewan, Mona Diab, Xian Li, Xi Victoria Lin 等. 2022. Opt: 开放预训练变换器语言模型。*arXiv 预印本 arXiv:2205.01068*（2022）。

+   张等（2024i）Xinyu Zhang, Huiyu Xu, Zhongjie Ba, Zhibo Wang, Yuan Hong, Jian Liu, Zhan Qin, 和 Kui Ren. 2024i. Privacyasst: 保护工具使用的大语言模型代理中的用户隐私。*IEEE 可靠与安全计算学报*（2024）。

+   张等（2024b）Yiming Zhang, Nicholas Carlini, 和 Daphne Ippolito. 2024b. 从语言模型中有效地提取提示语。*arXiv 预印本 arXiv:2307.06865*（2024）。

+   张等（2024c）Yuxiang Zhang, Jing Chen, Junjie Wang, Yaxin Liu, Cheng Yang, Chufan Shi, Xinyu Zhu, Zihao Lin, Hanwen Wan, Yujiu Yang 等. 2024c. ToolBeHonest: 一种针对工具增强大语言模型的多层次幻觉诊断基准。*arXiv 预印本 arXiv:2406.20015*（2024）。

+   张等（2023a）Yue Zhang, Yafu Li, Leyang Cui, Deng Cai, Lemao Liu, Tingchen Fu, Xinting Huang, Enbo Zhao, Yu Zhang, Yulong Chen 等. 2023a. AI海洋中的塞壬之歌：关于大语言模型幻觉的调查。*arXiv 预印本 arXiv:2309.01219*（2023）。

+   张等（2023b）Yichi Zhang, Jiayi Pan, Yuchen Zhou, Rui Pan, 和 Joyce Chai. 2023b. 在语言中接地视觉错觉：视觉语言模型是否像人类一样感知错觉？在*2023年自然语言处理经验方法会议论文集*，5718–5728。

+   Zhang et al. (2024a) Zeyu Zhang, Xiaohe Bo, Chen Ma, Rui Li, Xu Chen, Quanyu Dai, Jieming Zhu, Zhenhua Dong, and Ji-Rong Wen. 2024a. 大语言模型代理的记忆机制综述。*arXiv预印本 arXiv:2404.13501* (2024)。

+   Zhang et al. (2024h) Zhuo Zhang, Guangyu Shen, Guanhong Tao, Siyuan Cheng, and Xiangyu Zhang. 2024h. 大语言模型对强制性审问的韧性研究。在*2024 IEEE安全与隐私研讨会 (SP)*。IEEE计算机学会，252–252。

+   Zhao et al. (2023a) Xuandong Zhao, Yu-Xiang Wang, and Lei Li. 2023a. 通过隐形水印保护语言生成模型。在*国际机器学习大会*。PMLR，42187–42199。

+   Zhao et al. (2023b) Zhiyuan Zhao, Bin Wang, Linke Ouyang, Xiaoyi Dong, Jiaqi Wang, and Conghui He. 2023b. 超越幻觉：通过幻觉感知的直接偏好优化提升lvlms。*arXiv预印本 arXiv:2311.16839* (2023)。

+   Zheng et al. (2024) Kening Zheng, Junkai Chen, Yibo Yan, Xin Zou, and Xuming Hu. 2024. Reefknot: 一种多模态大语言模型中关系幻觉评估、分析与缓解的综合基准。*arXiv预印本 arXiv:2408.09429* (2024)。

+   Zheng and Zhu (2023) Zi’ou Zheng and Xiaodan Zhu. 2023. NatLogAttack：一个针对自然语言推理模型的自然逻辑攻击框架。*arXiv预印本 arXiv:2307.02849* (2023)。

+   Zhou et al. (2020) Chunting Zhou, Graham Neubig, Jiatao Gu, Mona Diab, Paco Guzman, Luke Zettlemoyer, and Marjan Ghazvininejad. 2020. 检测条件神经序列生成中的幻觉内容。*arXiv预印本 arXiv:2011.02593* (2020)。

+   Zhou et al. (2022) Kankan Zhou, Yibin LAI, and Jing Jiang. 2022. Vlstereoset：预训练视觉语言模型中的刻板偏见研究。计算语言学协会。

+   Zhou et al. (2023) Yiyang Zhou, Chenhang Cui, Jaehong Yoon, Linjun Zhang, Zhun Deng, Chelsea Finn, Mohit Bansal, and Huaxiu Yao. 2023. 分析与缓解大视觉语言模型中的物体幻觉。*arXiv预印本 arXiv:2310.00754* (2023)。

+   Zhou et al. (2021) Yi Zhou, Xiaoqing Zheng, Cho-Jui Hsieh, Kai-Wei Chang, and Xuanjing Huan. 2021. 通过Dirichlet邻域集成防御基于同义词替代的对抗攻击。在*计算语言学协会 (ACL)*。

+   Zou et al. (2023) Andy Zou, Zifan Wang, Nicholas Carlini, Milad Nasr, J Zico Kolter, and Matt Fredrikson. 2023. 对齐语言模型的普适性和可转移性对抗攻击。*arXiv预印本 arXiv:2307.15043* (2023)。

+   Zou et al. (2024) Wei Zou, Runpeng Geng, Binghui Wang, and Jinyuan Jia. 2024. PoisonedRAG：针对大语言模型检索增强生成的知识中毒攻击。*arXiv预印本 arXiv:2402.07867* (2024)。
