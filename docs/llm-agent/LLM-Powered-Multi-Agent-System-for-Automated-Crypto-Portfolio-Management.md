<!--yml

分类：未分类

日期：2025-01-11 11:41:26

-->

# 基于LLM的多智能体系统用于自动化加密货币投资组合管理

> 来源：[https://arxiv.org/html/2501.00826/](https://arxiv.org/html/2501.00826/)

\DeclareAcronym

llm简写 = LLM，长形式 = 大型语言模型 \DeclareAcronymohlc简写 = OHLC，长形式 = 开盘价、最高价、最低价和收盘价 \DeclareAcronymmcc简写 = MCC，长形式 = 马修斯相关系数 \DeclareAcronymcapm简写 = CAPM，长形式 = 资本资产定价模型

Yichen Luo 伦敦大学学院 伦敦 英国 [yichen.luo.22@ucl.ac.uk](mailto:yichen.luo.22@ucl.ac.uk) ， Yebo Feng 南洋理工大学 新加坡 [yebo.feng@ntu.edu.sg](mailto:yebo.feng@ntu.edu.sg) ， Jiahua Xu 伦敦大学学院 指数科学基金会 伦敦 英国 [jiahua.xu@ucl.ac.uk](mailto:jiahua.xu@ucl.ac.uk) ， Paolo Tasca 伦敦大学学院 指数科学基金会 伦敦 英国 [p.tasca@ucl.ac.uk](mailto:p.tasca@ucl.ac.uk)  和 Yang Liu 南洋理工大学 新加坡 [yangliu@ntu.edu.sg](mailto:yangliu@ntu.edu.sg)

###### 摘要。

加密货币投资本质上具有难度，因为与传统资产相比，其历史较短，需要整合来自不同模态的大量数据，并且要求复杂的推理。尽管深度学习方法已被应用于解决这些挑战，但它们的“黑箱”性质引发了关于信任和可解释性的担忧。最近，\acpllm在金融应用中显示出希望，因为它们能够理解多模态数据并生成可解释的决策。然而，单一\acsllm在处理资产投资等复杂、综合性任务时面临局限。这些局限性在加密货币投资中尤为突出，因为\acspllm在其训练语料库中缺乏领域特定的知识。

为了克服这些挑战，我们提出了一个可解释的、多模态、多智能体的加密货币投资框架。我们的框架使用专门的智能体，这些智能体在团队内外进行协作，处理诸如数据分析、文献整合和投资决策等子任务，涉及市值排名前30的加密货币。专家训练模块通过多模态历史数据和专业投资文献对智能体进行微调，而多智能体投资模块则利用实时数据做出有根据的加密货币投资决策。独特的团队内部和团队间协作机制通过根据智能体团队内的信心水平调整最终预测，并促进团队间的信息共享，从而提高预测的准确性。使用2023年11月至2024年9月的数据进行的实证评估表明，我们的框架在分类、资产定价、投资组合和可解释性表现方面优于单智能体模型和市场基准。

## 1\. 引言

加密货币投资是一项具有挑战性且全面的任务，原因在于其有限的资产定价证据（Corbet et al., [2019](https://arxiv.org/html/2501.00826v2#bib.bib10); Fang et al., [2022](https://arxiv.org/html/2501.00826v2#bib.bib14)）、对来自不同模态数据的需求（Jing and Kang, [2024](https://arxiv.org/html/2501.00826v2#bib.bib21); Kapur et al., [2024](https://arxiv.org/html/2501.00826v2#bib.bib22); Liu and Tsyvinski, [2021](https://arxiv.org/html/2501.00826v2#bib.bib32); Liu et al., [2022](https://arxiv.org/html/2501.00826v2#bib.bib33)），以及对复杂推理的需求（Hackethal et al., [2022](https://arxiv.org/html/2501.00826v2#bib.bib19)）。因此，分析加密货币市场、设计策略和构建投资组合变得是一项庞大的任务，并且给金融专家带来了沉重的工作负担，这使得专业服务变得稀缺或昂贵（CFP Board, [2022](https://arxiv.org/html/2501.00826v2#bib.bib7)）。为了解决这些挑战，许多研究人员探索了使用深度学习技术（Goutte et al., [2023](https://arxiv.org/html/2501.00826v2#bib.bib17); Lahmiri and Bekiros, [2019](https://arxiv.org/html/2501.00826v2#bib.bib25)）来进行加密货币投资。然而，大多数深度学习模型的“黑箱”性质引发了关于信任和可解释性的担忧，这使得投资者在投入资本时犹豫不决，难以依赖这些技术（Li et al., [2023b](https://arxiv.org/html/2501.00826v2#bib.bib27); Carta et al., [2021](https://arxiv.org/html/2501.00826v2#bib.bib6); Biran and McKeown, [2017](https://arxiv.org/html/2501.00826v2#bib.bib5)）。

\acpllm的引入彻底改变了金融领域，为加密货币投资提供了有前景的解决方案。大量研究表明，\acpllm在理解和学习多模态数据方面具有强大的能力（Yin等人，[2024](https://arxiv.org/html/2501.00826v2#bib.bib41)），这些数据包括文本（Yang等人，[2024](https://arxiv.org/html/2501.00826v2#bib.bib40）；Li等人，[2023c](https://arxiv.org/html/2501.00826v2#bib.bib29)）和图像（Zhang等人，[2024](https://arxiv.org/html/2501.00826v2#bib.bib42)），使其非常适合学习专业的加密货币投资知识，并从不同模态的数据中分析市场。另一方面，\acpllm具有出色的自然语言生成能力（Wei等人，[2022](https://arxiv.org/html/2501.00826v2#bib.bib37)；Liu等人，[2023a](https://arxiv.org/html/2501.00826v2#bib.bib30)），这使其能够生成可解释的加密货币投资决策。然而，单一\acpllm在资产预测中的表现受限，因为这一任务需要综合性和复杂的推理（Xie等人，[2023](https://arxiv.org/html/2501.00826v2#bib.bib39)；Li等人，[2023a](https://arxiv.org/html/2501.00826v2#bib.bib26)）。在加密货币投资中，这一弱点尤为明显，因为\acpllm在其训练语料库中缺乏特定领域的知识。为了解决这一挑战，研究人员开发了将复杂任务分解为子任务的方法（Wu等人，[2023](https://arxiv.org/html/2501.00826v2#bib.bib38)；Pan等人，[2024](https://arxiv.org/html/2501.00826v2#bib.bib34)）。该方法利用多个基于\acllm的代理之间的协作来得出最终的综合解决方案，每个代理专注于整体任务的一个特定方面。受人类认知过程启发，这种方法增强了推理能力和解决综合问题的效率，为基于代理的投资解决方案提供了新的可能性。尽管一些研究探索了在股票投资中使用多代理模型（Ding等人，[2024](https://arxiv.org/html/2501.00826v2#bib.bib12)；Fatemi和Hu，[2024](https://arxiv.org/html/2501.00826v2#bib.bib15)），但在加密货币投资中采用多代理模型的研究较少，而且仅限于比特币、以太坊和索拉纳以及单一模态数据（Li等人，[2024](https://arxiv.org/html/2501.00826v2#bib.bib28)）。

为了解决上述问题并填补这一空白，我们提出了一种可解释的、多模态的、多智能体框架，该框架利用多个智能体团队，在团队内部及跨团队的协作下，促进监督学习和基于市值排名前30的加密货币的投资决策。在此框架内，涉及不同模态数据的复杂投资任务被分解为若干子任务，每个经过微调的专家智能体负责特定的子任务。受到对冲基金中沟通方式的启发，我们独特的团队内部及跨团队协作机制确保最终的投资决策能有效整合来自多模态的信息。

我们的多智能体框架由两个模块组成：专家训练模块和多智能体投资模块。专家训练模块采用来自数据团队和文献团队的智能体，分别获取历史的多模态数据和相关的投资文献。接着，解释团队的智能体处理这些数据和文献，通过整合多模态信息和专业投资知识生成高质量的提示。最后，这些提示用于微调专家投资智能体，每个智能体专注于分析单一模态的数据。多智能体投资模块利用数据团队获取实时数据，并将数据传递给市场团队和加密团队。市场团队包括两个专家智能体，他们分析新闻和市场因素，预测市场趋势并确定现金-加密货币的配置。同样，加密团队有两个专家智能体，他们分析与加密货币相关的因素和单个加密货币的蜡烛图，以做出加密货币选择决策。最后，交易团队与加密货币交易所API进行交互，执行最终的投资组合策略。团队内部协作机制结合同一小组内智能体的置信度得分，产生集成预测。跨团队协作机制则允许加密团队与市场团队共享关于市场信息的记忆，从而基于更全面的信息做出更稳健的加密货币选择决策。

为了验证我们框架的有效性，我们使用了2023年6月到2024年9月的数据，验证其在分类准确度和资产定价表现方面，相较于单一智能体模型（无论是否进行了微调）的优越性。此外，我们还展示了我们的框架在投资组合表现上超越了市场基准。本论文的主要贡献总结如下：

+   •

    我们是首个提出面向大型市值加密货币投资组合管理的多智能体框架，并将多模态数据和专业投资知识分别整合进决策和解释过程中。

+   •

    我们设计了独特的团队间和团队内协作机制，以促进沟通并减少不同智能体之间的预测误差。这些机制显著提高了我们模型在加密货币投资中的表现。

+   •

    我们为\acllm设计了独特的资产定价方法，通过使用代币概率将二元的涨跌价格趋势分类转化为一系列的信心水平。这些信心水平随后被用来根据金融学中的经验性资产定价方法构建投资组合。

+   •

    通过通过微调学习历史数据和相关学术文献，我们的多智能体模型能够生成有效解释加密货币回报变化的预测，并提供高质量的解释。

+   •

    我们的多智能体模型不仅在分类、资产定价和扩展性表现上优于单一的\acllm，而且在投资组合表现上也超越了市场基准。

## 2. 相关工作

在本节中，我们回顾了经验性加密货币定价研究的进展，并检视了那些利用单一\acllm和多智能体框架进行投资的相关研究。

经验性加密货币定价。作为一种新兴的另类资产类别，加密货币吸引了大量研究兴趣，特别是在资产定价领域。经验性加密货币定价是最初由尤金·法马（Eugene Fama）和肯尼斯·弗伦奇（Kenneth French）提出的经验性资产定价的一分支，用于解释资产回报（Fama and French，[1993](https://arxiv.org/html/2501.00826v2#bib.bib13)）。早期的加密货币经验性定价研究关注市场回报的可预测性，确定了如网络活动、动量和投资者关注度等因素作为未来加密货币市场回报的重要预测因素（Liu and Tsyvinski，[2021](https://arxiv.org/html/2501.00826v2#bib.bib32)）。此外，新闻情绪被证明对加密货币市场回报有显著影响（Anamika and Subramaniam，[2022](https://arxiv.org/html/2501.00826v2#bib.bib2)）。随后，市场、规模和动量因素被确认为加密货币横截面预期回报的关键决定因素，从而发展出了专门针对加密货币的三因子模型（Liu et al.，[2022](https://arxiv.org/html/2501.00826v2#bib.bib33)）。此外，基于趋势的技术指标，通常可以在蜡烛图中识别出来，也显示出在预测加密货币横截面回报方面的预测能力（Tan and Tao，[2023](https://arxiv.org/html/2501.00826v2#bib.bib36)）。尽管这些研究强调了跨各种数据模式（包括面板数据、文本信息和图表模式）中的强大预测信息，但在开发能够整合这些不同数据模式的统一模型方面仍然存在显著的空白。

投资领域的大型语言模型。凭借其强大的文本理解和推理能力，\acpllm 已广泛应用于不同的投资任务。早期的研究主要集中在使用单一的 \acllm 来预测资产价格和执行投资策略。有些研究尝试对其金融 \acllm 进行微调，以完成投资任务（Xie 等，[2023](https://arxiv.org/html/2501.00826v2#bib.bib39); Liu 等，[2023b](https://arxiv.org/html/2501.00826v2#bib.bib31); Li 等，[2023a](https://arxiv.org/html/2501.00826v2#bib.bib26)）。此外，一项研究特别考察了 \acllm 在交易三种加密货币（比特币、以太坊和索拉纳）中的表现（Li 等，[2024](https://arxiv.org/html/2501.00826v2#bib.bib28)）。然而，即使在微调之后，单一 \acllm 的预测能力仍然有限，其结果通常存在显著的偏差。

为了进一步提升 \acllm 在投资中的表现，近年来的研究转向了使用多智能体模型来完成投资任务。一个值得注意的例子是总结-解释-预测（SEP）框架，该框架采用反思性智能体，通过其他智能体的帮助，迭代生成股票预测和解释（Koa 等，[2024](https://arxiv.org/html/2501.00826v2#bib.bib23)）。一些研究集中于使用多个智能体分别处理数据、总结信息、反思和生成股票预测（Kou 等，[2024](https://arxiv.org/html/2501.00826v2#bib.bib24); Fatemi 和 Hu，[2024](https://arxiv.org/html/2501.00826v2#bib.bib15); Ding 等，[2024](https://arxiv.org/html/2501.00826v2#bib.bib12)）。然而，针对加密货币投资任务的多智能体、多模态模型的开发仍存在差距。为填补这一空白，我们提出了一个多智能体框架，其中专门的智能体负责处理不同类型的信息，共同投资于领先的加密货币领域。

## 3\. 方法论

在本节中，我们首先将加密货币投资过程分解为多个子任务并对其进行形式化。接下来，我们介绍了所提出的多智能体加密货币投资框架，如[图 2](https://arxiv.org/html/2501.00826v2#S3.F2 "Figure 2 ‣ 3.2\. Framework Overview ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")所示。该框架由两个主要模块组成：(1) 专家训练模块，通过多模态历史数据和专业投资文献对智能体进行微调；(2) 多智能体投资模块，利用实时数据做出明智的加密货币投资决策。

### 3.1\. 问题定义

#### 3.1.1\. 加密货币现金分配

给定一个市场特定的风险因子向量 $\bm{\beta}_{t-1}=[\beta_{i}]_{p\times 1}$ 在$t-1$周，其中 $p$ 表示因子的总数，以及新闻数据 $\mathbf{N}_{t-1}=[N_{i}]_{q\times 1}$ 在$t-1$周，其中 $q$ 表示新闻头条的总数，我们的目标是生成加密货币权重 $w_{t}$ 以最大化加权市场回报：$\displaystyle\arg\max_{w_{t}}w_{t}r^{\textnormal{mkt}}_{t}$ 和一个可读的解释 $\hat{e}_{t}^{\textnormal{mkt}}$ 在$t$周。

#### 3.1.2\. 加密货币选择

给定一组加密货币 $\mathcal{C}=\{c_{i}\}_{i=1}^{I}$，一个加密特定风险因子矩阵 $\bm{\alpha}_{t-1}=[\alpha_{i,c}]_{m\times n}$，其中 $m$ 是加密特定风险因子的总数，$n$ 是加密货币的总数，以及一个可视化数据向量 $\mathbf{v}_{t-1}=[v_{c}]_{m\times 1}$，我们的目标是生成一个子集 $\mathcal{C}^{*}\subseteq\mathcal{C}$，以最大化这些加密货币的未来7天平均回报 $\displaystyle\arg\max_{\mathcal{C}^{*}\subseteq\mathcal{C}}\frac{1}{|C^{*}|}% \textstyle\sum_{c\in C^{*}}r^{c}_{t}$ 和一个可读的解释 $\hat{e}_{t}^{c}$。

### 3.2\. 框架概述

![参考标题](img/79ea15c66726d3cc150e48f0ff1e6894.png)

(a) 30天蜡烛图，带有交易量柱状图和30天移动平均线。

![参考标题](img/45ed0281b9af6ef34d40f8d873fd6d6b.png)

(b) 真实数据。下一周加密货币和整个市场的二进制价格趋势。

![参考标题](img/849bc2a9e9887b175fd801132e5ba5b7.png)

(c) 从链上和链下数据构建的风险因子。

![参考标题](img/a0991bc3644fa324601522c60bac2aa5.png)

(d) 从Cointelegraph爬取的新闻头条。

图1. 我们的多智能体框架使用的多模态数据。详细的数据描述可以在[附录A](https://arxiv.org/html/2501.00826v2#A1 "附录A 数据描述 ‣ 基于LLM的自动化加密货币投资组合管理多智能体系统")中找到。

![参考标题](img/0564905de44fff61c70ed8cc8d03c220.png)

(a) 智能体训练。

![参考标题](img/47267618b1e434eae27fcba50f17238b.png)

(b) 多智能体投资的工作流程。

图2. 自动化加密货币投资组合管理的多智能体框架。

在本文中，我们提出了一个可解释的多代理框架，用于加密货币投资，如[图 2](https://arxiv.org/html/2501.00826v2#S3.F2 "图 2 ‣ 3.2\. 框架概述 ‣ 3\. 方法论 ‣ 基于大语言模型的多代理系统用于自动化加密货币投资组合管理")所示。我们的框架包含两个主要组件：专家训练和多代理投资。第一个组件在[§3.3](https://arxiv.org/html/2501.00826v2#S3.SS3 "3.3\. 专家训练 ‣ 3\. 方法论 ‣ 基于大语言模型的多代理系统用于自动化加密货币投资组合管理")中进行了详细描述，采用多个代理之间的协作来生成训练提示，这些提示融合了来自不同模态的数据和相应的高质量、逐案解释。随后，从不同数据模态中获得的知识通过微调整合到各自的专家代理中。第二个组件在[§3.4](https://arxiv.org/html/2501.00826v2#S3.SS4 "3.4\. 多代理投资 ‣ 3\. 方法论 ‣ 基于大语言模型的多代理系统用于自动化加密货币投资组合管理")中进行了描述，它使得专家代理能够管理加密货币投资中的相应子任务，并协同构建最终的加密货币投资组合。该框架旨在将复杂的投资挑战分解为更小、更专业化的任务，由专家代理处理，从而提高预测准确性并改善投资组合表现。

### 3.3\. 专家训练

专家训练过程涉及多个代理团队之间的协作。

#### 3.3.1\. 数据团队

在数据团队中，数据获取者负责获取和处理原始数据。这个代理使用工具从领先的加密货币提供商收集数据，包括Coingecko、Blockchain.info、Coin Metrics和Cointelegraph。一旦原始数据被获取，数据获取者会将其处理成多模态格式，包括价格趋势真实数据、30天的蜡烛图、风险因素（阿尔法值）和新闻标题，如[图1](https://arxiv.org/html/2501.00826v2#S3.F1 "图1 ‣ 3.2\. 框架概述 ‣ 3\. 方法论 ‣ 基于LLM的多代理系统用于自动化加密货币投资组合管理")所示。30天蜡烛图（[1(a)](https://arxiv.org/html/2501.00826v2#S3.F1.sf1 "1(a) ‣ 图1 ‣ 3.2\. 框架概述 ‣ 3\. 方法论 ‣ 基于LLM的多代理系统用于自动化加密货币投资组合管理")）和二进制价格趋势（[1(b)](https://arxiv.org/html/2501.00826v2#S3.F1.sf2 "1(b) ‣ 图1 ‣ 3.2\. 框架概述 ‣ 3\. 方法论 ‣ 基于LLM的多代理系统用于自动化加密货币投资组合管理")）是从Coingecko提供的\acohlc价格数据和交易量中衍生出的。风险因素（[1(c)](https://arxiv.org/html/2501.00826v2#S3.F1.sf3 "1(c) ‣ 图1 ‣ 3.2\. 框架概述 ‣ 3\. 方法论 ‣ 基于LLM的多代理系统用于自动化加密货币投资组合管理")）是利用Coingecko提供的\acohlc价格数据、交易量和市值，以及来自Blockchain.info和Coin Metrics的链上数据计算得出的。风险因素被划分为五个五分位数：“非常低”，“低”，“中等”，“高”和“非常高”。五分位数的分界点是通过使用加密货币相关因素的横截面数据和市场相关因素的前两年数据来确定的。此外，新闻标题数据（[1(d)](https://arxiv.org/html/2501.00826v2#S3.F1.sf4 "1(d) ‣ 图1 ‣ 3.2\. 框架概述 ‣ 3\. 方法论 ‣ 基于LLM的多代理系统用于自动化加密货币投资组合管理")）是通过从Cointelegraph进行网页爬取获得的。我们在[附录A](https://arxiv.org/html/2501.00826v2#A1 "附录A 数据描述 ‣ 基于LLM的多代理系统用于自动化加密货币投资组合管理")中报告了详细的数据描述。

#### 3.3.2\. 文献团队

在文献团队中，文献获取者负责从Google Scholar获取学术论文。这些学术论文的选择是基于它们与实证加密货币定价领域中具体数据模态的相关性。

#### 3.3.3\. 解释团队

[tb] <svg class="ltx_picture" height="111.81" id="S3.SS3.SSS3.1.1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,111.81) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 93.61)"><foreignobject color="#000000" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">系统说明 1：解释。</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="62.11" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你是一个专业的加密货币分析师，专门根据提供的知识和信息解释预测目标。你应该内化所提供的知识，生成一个全面的解释，而无需明确引用文献。你的输出应以一个段落呈现。</foreignobject></g></g></svg>

[tb] <svg class="ltx_picture" height="146.56" id="S3.SS3.SSS3.2.1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,146.56) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 128.35)"><foreignobject color="#000000" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">提示 1：市场因素解释代理。</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="96.86" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">学习以下加密货币投资知识。使用这些知识，根据提供的信息解释未来一周的预测目标。市场因素已根据前两年的数据分为非常高、高、中、低和非常低五个类别。预测的市场回报被分为上涨或下跌。知识：{文献}（知识结束）信息：{因素/新闻}（信息结束）市场趋势：{未来市场趋势}（市场趋势结束）</foreignobject></g></g></svg>

[tb] <svg class="ltx_picture" height="129.96" id="S3.SS3.SSS3.3.1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,129.96) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 111.75)"><foreignobject color="#000000" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">提示 2：新闻解释代理。</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="80.25" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">学习以下加密货币投资知识。利用这些知识，解释根据提供的新闻标题预测的未来一周的目标。预测的市场回报已被分类为上涨或下跌。知识：{literature}（知识结束）信息：{factors/news}（信息结束）市场趋势：{future market trend}（市场趋势结束）</foreignobject></g></g></svg>

[tb] <svg class="ltx_picture" height="163.16" id="S3.SS3.SSS3.4.1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,163.16) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 144.96)"><foreignobject color="#000000" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">提示 3：加密货币解释代理。</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="113.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">学习以下加密货币投资知识。利用这些知识，解释根据提供的信息预测的{target crypto}在未来一周的价格趋势。包括{target crypto}在内的前30种加密货币的数据，已分为非常高、高、中、低和非常低五个类别。它们各自的价格趋势已被分类为上涨或下跌。知识：{literature}（知识结束）信息：{factors}（信息结束）价格趋势：{future price trend}（价格趋势结束）</foreignobject></g></g></svg>

[tb] <svg class="ltx_picture" height="146.56" id="S3.SS3.SSS3.5.1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,146.56) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 128.35)"><foreignobject color="#000000" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">提示4：视觉解释代理。</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="96.86" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">文本：学习以下加密货币投资知识。利用这些知识，根据提供的蜡烛图，解释{目标加密货币}未来一周的预测价格趋势。该图表包含显示每日开盘价、最高价、最低价和收盘价的蜡烛线。然后叠加30日移动平均收盘价。图表底部显示每日交易量。知识：{文献}（知识结束）价格趋势：{未来价格趋势}（价格趋势结束）图片网址：{网址}</foreignobject></g></g></svg>

为了生成包含详细逐案例推理的训练提示，这些推理来源于学术论文，一个解释团队负责增强训练数据。该团队通过融入专业、经过深思熟虑的解释，将普通的训练数据对——由多模态数据和真实数据组成——转化为丰富的数据对。具体来说，市场因素分析师和新闻分析师专注于分析当前一周的市场特定风险因素和新闻数据，并与下一周的市场趋势进行对应分析。通过使用共享的系统指令[§3.3.3](https://arxiv.org/html/2501.00826v2#S3.SS3.SSS3 "3.3.3\. Explanation Team ‣ 3.3\. Expert Training ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")和提示[§3.3.3](https://arxiv.org/html/2501.00826v2#S3.SS3.SSS3 "3.3.3\. Explanation Team ‣ 3.3\. Expert Training ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")，[§3.3.3](https://arxiv.org/html/2501.00826v2#S3.SS3.SSS3 "3.3.3\. Explanation Team ‣ 3.3\. Expert Training ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")，市场因素和新闻分析师通过借鉴相关学术论文的见解，解释市场相关信息与市场趋势之间的复杂关系。同样，密码学分析师使用系统指令[§3.3.3](https://arxiv.org/html/2501.00826v2#S3.SS3.SSS3 "3.3.3\. Explanation Team ‣ 3.3\. Expert Training ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")和提示[§3.3.3](https://arxiv.org/html/2501.00826v2#S3.SS3.SSS3 "3.3.3\. Explanation Team ‣ 3.3\. Expert Training ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")分析与加密货币相关的特定风险因素和真实数据，生成详细的解释。接着，技术分析师使用系统指令[§3.3.3](https://arxiv.org/html/2501.00826v2#S3.SS3.SSS3 "3.3.3\. Explanation Team ‣ 3.3\. Expert Training ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")和提示[§3.3.3](https://arxiv.org/html/2501.00826v2#S3.SS3.SSS3 "3.3.3\. Explanation Team ‣ 3.3\. Expert Training ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")解读单个加密货币30日蜡烛图与其相应真实数据之间的关系，提供经过深思熟虑的见解。最后，使用系统指令[§3.4.1](https://arxiv.org/html/2501.00826v2#S3.SS4.SSS1 "3.4.1\. Market Team ‣ 3.4\. Multi-Agent Investment ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")和模板提示[§3.4.1](https://arxiv.org/html/2501.00826v2#S3.SS4.SSS1 "3.4.1\. Market Team ‣ 3.4\. Multi-Agent Investment ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")，将多模态数据、真实数据及相应的解释整合进训练提示。最终，经过四位解释分析师增强的提示将被输入到四个\acpllm中，用以训练市场团队和加密团队的专家。

### 3.4\. 多代理投资

多代理投资组件通过多个代理之间的协作来完成加密货币投资过程。该过程从数据团队获取和处理来自不同供应商的实时多模态数据开始，具体细节请参见[§3.3.1](https://arxiv.org/html/2501.00826v2#S3.SS3.SSS1 "3.3.1\. 数据团队 ‣ 3.3\. 专家培训 ‣ 3\. 方法论 ‣ 基于LLM的多代理系统用于自动化加密货币投资组合管理")。随后，市场团队、加密团队和交易团队接收处理后的数据，并完成各自的子任务，共同推动整体投资过程的完成。

#### 3.4.1\. 市场团队

[tb] <svg class="ltx_picture" height="96.59" id="S3.SS4.SSS1.1.1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,96.59) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 78.54)"><foreignobject color="#000000" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">系统指令 2: 微调。</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="47.05" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你是一个专业的加密货币分析师，专注于基于提供的信息预测下周的{”加密货币价格趋势”/”市场趋势”}。你的输出应为以下格式：目标: （预测目标） 解释: （你的解释）</foreignobject></g></g></svg>

[tb] <svg class="ltx_picture" height="96.59" id="S3.SS4.SSS1.2.1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,96.59) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 78.54)"><foreignobject color="#000000" height="12.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">提示 5: 微调。</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="47.05" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">用户: 分析以下加密货币信息，确定其一周内的目标走势。请回复“上涨”或“下跌”，并提供你的预测理由。: {info} (信息结束) 助手: {”价格趋势”/”市场趋势”}: {趋势} 解释: {解释}</foreignobject></g></g></svg>

为了完成在 [§3.1.1](https://arxiv.org/html/2501.00826v2#S3.SS1.SSS1 "3.1.1\. Cryptocurrency-cash allocation ‣ 3.1\. Problem Formulation ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management") 中描述的加密货币与现金配置子任务，市场团队使用了经过训练的新闻专家和市场因子专家代理 $\textnormal{A}^{\textnormal{news}}$ 来预测市场趋势。具体而言，新闻专家会获得 [§3.4.1](https://arxiv.org/html/2501.00826v2#S3.SS4.SSS1 "3.4.1\. Market Team ‣ 3.4\. Multi-Agent Investment ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management") 中的系统指令，并通过将过去一周的新闻标题数据 $\mathbf{N}_{t-1}$ 填入 [§3.4.1](https://arxiv.org/html/2501.00826v2#S3.SS4.SSS1 "3.4.1\. Market Team ‣ 3.4\. Multi-Agent Investment ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management") 中概述的模板来生成提示。使用此提示，新闻专家会生成当前一周的预测 $\hat{Y}^{\textnormal{news}}_{t}$，该预测包括两个组成部分：(1) 一个二分类结果，$\hat{y}^{\textnormal{news}}_{t}\in\{\textnormal{\leavevmode\ltxml@oqmark@open% \textquotedblleft\penalty 10000\hskip-0.0002pt\hskip 0.0002ptRise% \textquotedblright\ltxml@oqmark@close{}},\textnormal{\leavevmode% \ltxml@oqmark@open\textquotedblleft\penalty 10000\hskip-0.0002pt\hskip 0.0002% ptFall\textquotedblright\ltxml@oqmark@close{}}\}$，表示预期的下周市场趋势；(2) 一个可以供人类阅读的解释，$\hat{e}^{\textnormal{news}}_{t}$，提供预测背后的详细推理，即 $\hat{Y}^{\textnormal{news}}_{t}=(\hat{y}^{\textnormal{news}}_{t},\hat{e}^{\textnormal{news}}_{t})$。我们可以将这个过程形式化为

| (1) |  | $\hat{Y}_{t}=\textnormal{A}(X^{\textnormal{mkt}}_{t-1}),$ |  |
| --- | --- | --- | --- |

其中 $X^{mkt}_{t-1}$ 是上一周的广义市场特定数据。在这种情况下，$X^{mkt}_{t-1}=\mathbf{N}_{t-1}$。类似地，市场因子专家代理 $\textnormal{A}^{\textnormal{mf}}$ 被提供了包含市场特定风险因子的系统指令和提示，$\bm{\beta}_{t-1}$，以生成一个预测 $\hat{Y}^{\textnormal{mf}}_{t}$，包括一个二分类结果，$\hat{y}^{\textnormal{mf}}_{t}\in\{\textnormal{\leavevmode\ltxml@oqmark@open% \textquotedblleft\penalty 10000\hskip-0.0002pt\hskip 0.0002ptRise% \textquotedblright\ltxml@oqmark@close{}},\textnormal{\leavevmode% \ltxml@oqmark@open\textquotedblleft\penalty 10000\hskip-0.0002pt\hskip 0.0002% ptFall\textquotedblright\ltxml@oqmark@close{}}\}$，以及一个人类可读的解释，$\hat{e}^{\textnormal{mf}}_{t}$。

[tb] <svg class="ltx_picture" height="113.35" id="S3.SS4.SSS1.3.1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,113.35) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 95.15)"><foreignobject color="#000000" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">系统指令 3：预测。</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="63.65" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你是一名专业的加密货币分析师，专注于基于提供的蜡烛图预测下周的{“加密货币价格趋势”/“市场趋势”}。你的输出应为：{“价格趋势”/“市场趋势”}：（预测目标）解释：（你的解释）</foreignobject></g></g></svg>

[tb] <svg class="ltx_picture" height="113.35" id="S3.SS4.SSS1.4.1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,113.35) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 95.15)"><foreignobject color="#000000" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">提示 6：预测。</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="63.65" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">分析以下{“加密货币”/“市场”}信息，以确定一周内{“加密货币价格趋势”/“市场趋势”}的强度。请回应“上涨”或“下跌”，并提供你的预测理由。信息：{因素/新闻/图表}（信息结束）</foreignobject></g></g></svg>

![参考说明](img/e3e2b5762d68af57181feb1a926b6c30.png)

(a) 小组内合作。

![参考说明](img/95e9744945326cc4e1900683292b0a3d.png)

(b) 团队间合作。

图 3. 同一小组内和不同小组之间的专家代理人合作。

为了使团队内的这两位代理能够协同工作并生成最终的加密货币与现金分配子任务的解决方案，采用了在[3(a)](https://arxiv.org/html/2501.00826v2#S3.F3.sf1 "3(a) ‣ 图3 ‣ 3.4.1\. 市场团队 ‣ 3.4\. 多代理投资 ‣ 3\. 方法论 ‣ 基于LLM的多代理自动化加密投资组合管理")中展示的团队内协作方法。该方法允许两位代理根据各自的预测信心水平集成他们的预测。具体而言，由于\acllm通过选择具有最高概率的标记来生成文本，我们可以提取分类标记（“上升”或“下降”）的“上升”日志概率。这个“上升”的日志概率表示为$\log P(\hat{y}_{t}=\textnormal{\leavevmode\ltxml@oqmark@open\textquotedblleft\penalty 10000\hskip-0.0002pt\hskip 0.0002ptRise\textquotedblright\ltxml@oqmark@close{}}|\mathbf{N}_{t-1})$，并在[3(a)](https://arxiv.org/html/2501.00826v2#S3.F3.sf1 "3(a) ‣ 图3 ‣ 3.4.1\. 市场团队 ‣ 3.4\. 多代理投资 ‣ 3\. 方法论 ‣ 基于LLM的多代理自动化加密投资组合管理")中进行了可视化展示，其中绿色背景表示较高的日志概率。这一概率作为预测的附加信心度量。为了集成预测，日志概率会转化为线性概率。最终的集成上升概率通过取两位代理的线性概率的算术平均值来计算，从而有效地结合它们的见解，生成更强健的子任务预测。

| (2) |  | $\bar{P}=\frac{1}{\vert\mathcal{A}\vert}\sum_{i\in\mathcal{A}}e^{\log P(\hat{y}^{i}_{t}% =\textnormal{\leavevmode\ltxml@oqmark@open\textquotedblleft\penalty 10000% \hskip-0.0002pt\hskip 0.0002ptRise\textquotedblright\ltxml@oqmark@close{}} | \mathcal{X}_{t-1})},$ |  |
| --- | --- | --- | --- | --- |

其中，$\mathcal{A}$表示团队中代理的集合。随后，如果聚合的概率超过0.5，最终预测为“上升”；否则，预测为“下降”。根据集成分类结果，确定投资组合分配：如果预测为“上升”，则投资组合将完全由加密货币组成；否则，如果预测为“下降”，投资组合将平等分配于加密货币和现金。

#### 3.4.2\. 加密团队

为了完成[§3.1.2](https://arxiv.org/html/2501.00826v2#S3.SS1.SSS2 "3.1.2\. 加密货币选择 ‣ 3.1\. 问题定义 ‣ 3\. 方法论 ‣ 基于LLM的多代理自动化加密投资组合管理")中讨论的加密货币选择子任务，加密团队采用训练有素的加密因素专家和技术专家共同预测单个加密货币的价格趋势。

为了使加密货币团队的代理能够进行预测，不仅能够考虑加密货币特定的信息，还能涵盖更广泛的市场背景，我们采用了跨团队协作，如[3(b)](https://arxiv.org/html/2501.00826v2#S3.F3.sf2 "3(b) ‣ Figure 3 ‣ 3.4.1\. Market Team ‣ 3.4\. Multi-Agent Investment ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")所示。具体来说，加密货币团队的代理接收到系统指令[§3.4.1](https://arxiv.org/html/2501.00826v2#S3.SS4.SSS1 "3.4.1\. Market Team ‣ 3.4\. Multi-Agent Investment ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")和与相关加密货币特定数据集成的提示[§3.4.1](https://arxiv.org/html/2501.00826v2#S3.SS4.SSS1 "3.4.1\. Market Team ‣ 3.4\. Multi-Agent Investment ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")，作为他们的主要输入。此外，他们还会收到来自市场因素专家和新闻专家的输入和预测，这些信息作为上下文或共享的短期记忆，增强了他们的决策过程。我们可以将其形式化为：

| (3) |  | $\hat{Y}_{c,t}=\textnormal{A}\left(c,X_{c,t-1},X^{\textnormal{mf}}_{t-1},\hat{Y% }^{\textnormal{mf}}_{t},X^{\textnormal{news}}_{t-1},\hat{Y}^{\textnormal{news}% }_{t}\right).$ |  |
| --- | --- | --- | --- |

虽然单独的市场信息并不会直接有助于横截面加密货币价格趋势的预测，因为所有加密货币共享相同的市场层级数据，但我们预期专家代理能够学习市场信息和个别加密货币之间的交互作用。通过识别这些交互作用，加密货币团队可以提高预测的准确性。因此，加密货币因素专家提供了单个加密货币$c$及其特定的加密货币风险因子向量$\bm{\alpha}_{c,t-1}=[\alpha_{i}]_{m\times 1}$，而技术专家则接收加密货币$c$及其30天的蜡烛图数据$\mathbf{v}_{c,t-1}$。利用这些输入，专家们生成二元分类，$\hat{y}^{\textnormal{cf}}_{t}\in\{\textnormal{\leavevmode\ltxml@oqmark@open% \textquotedblleft\penalty 10000\hskip-0.0002pt\hskip 0.0002pt上涨% \textquotedblright\ltxml@oqmark@close{}},\textnormal{\leavevmode% \ltxml@oqmark@open\textquotedblleft\penalty 10000\hskip-0.0002pt\hskip 0.0002% pt下跌\textquotedblright\ltxml@oqmark@close{}}\}$和$\hat{y}^{\textnormal{chart}}_{t}\in\{\textnormal{\leavevmode\ltxml@oqmark@open% \textquotedblleft\penalty 10000\hskip-0.0002pt\hskip 0.0002pt上涨% \textquotedblright\ltxml@oqmark@close{}},\textnormal{\leavevmode% \ltxml@oqmark@open\textquotedblleft\penalty 10000\hskip-0.0002pt\hskip 0.0002% pt下跌\textquotedblright\ltxml@oqmark@close{}}\}$，表示加密货币在接下来一周的价格趋势预测。此外，他们还生成了易于理解的解释$\hat{e}^{\textnormal{cf}}_{t}$和$\hat{e}^{\textnormal{chart}}_{t}$，提供了每个预测背后的详细推理。

使用在[§3.4.1](https://arxiv.org/html/2501.00826v2#S3.SS4.SSS1 "3.4.1\. 市场团队 ‣ 3.4\. 多代理投资 ‣ 3\. 方法学 ‣ 基于LLM的自动化加密投资组合管理")中讨论的相同团队内协作方法，加密货币团队中的专家代理通过生成每个加密货币的最终集合上涨概率$\bar{P}_{c}$，达成关于单个加密货币价格趋势的共识，这个过程通过[公式 2](https://arxiv.org/html/2501.00826v2#S3.E2 "2 ‣ 3.4.1\. 市场团队 ‣ 3.4\. 多代理投资 ‣ 3\. 方法学 ‣ 基于LLM的自动化加密投资组合管理")实现。然后，我们根据$\bar{P}_{c}$对集合$\mathcal{C}$中的加密货币进行排序，分配到五等分的投资组合中。具体来说，我们形成5个不相交的等权重（1/N）投资组合，每个组合代表一个上涨概率范围，记作$\mathcal{P}_{i}$。这些投资组合的构建如下：

| (4) |  | $\mathcal{P}_{i}=\left[\bar{P}_{(\left\lfloor\frac{&#124;\mathcal{C}&#124;(i-1)}{5}\right% \rfloor)},\bar{P}_{(\left\lfloor\frac{&#124;\mathcal{C}&#124;i}{5}\right\rfloor)}\right)% \quad i=1,\cdots,5,$ |  |
| --- | --- | --- | --- |

其中，$\bar{P}_{(j)}$表示升序排列的所有加密货币在集合$\mathcal{C}$中的上涨概率集合$\{\bar{P}_{c}:1\leq c\leq|\mathcal{C}|\}$的第$j$阶静态值。$\left\lfloor\right\rfloor$为下取整操作符。投资组合$\mathcal{P}_{1},\cdots,\mathcal{P}_{5}$分别被标记为极低、低、中、高和极高。最后，标记为极高的投资组合$\mathcal{P}_{5}$被选为目标加密货币子集，$\mathcal{C}^{*}=\mathcal{P}_{5}\subseteq\mathcal{C}$，用于投资。

#### 3.4.3\. 交易团队

最终的交易团队负责通过与加密货币交易所的API交互，执行基于提供的投资组合的交易，确保整个过程完全端到端。

## 4\. 实验

在本节中，我们评估了我们的多智能体框架在收集的数据集上与相关基准方法的表现。

### 4.1\. 实验设置

在本研究中，我们采用ChatGPT-4o作为基础模型，因为它是目前最先进的多模态模型，能够实现视觉微调，且在撰写本文时是最先进的模型之一¹¹1[https://platform.openai.com/docs/guides/fine-tuning#which-models-can-be-fine-tuned](https://platform.openai.com/docs/guides/fine-tuning#which-models-can-be-fine-tuned)。我们在2023年6月到2024年9月期间收集了数据集，遵循了[§3.3.1](https://arxiv.org/html/2501.00826v2#S3.SS3.SSS1 "3.3.1\. Data Team ‣ 3.3\. Expert Training ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")中描述的方法论。我们将目标加密货币集合$\mathcal{C}$指定为根据CoinGecko按市值排名前30的加密货币。该列表每周更新，以反映市值的变化。只包括高市值加密货币的理由是，低市值的加密货币往往表现出与高流动性加密货币显著不同的定价动态，部分原因是受到例如拉高出货等风险的影响。此外，交易低流动性加密货币通常伴随较大的滑点，进一步复杂化了我们的任务。为了防止信息泄露，我们将2023年11月到2024年9月的数据设为测试集，因为ChatGPT-4o的训练数据仅截至2023年10月²²2[https://platform.openai.com/docs/models](https://platform.openai.com/docs/models)。因此，训练集包括2023年6月到2023年10月的数据。

为了展示我们多智能体框架的有效性，我们从三个不同的角度进行评估：分类表现、投资组合表现和资产定价表现：

#### 4.1.1\. 分类和资产定价表现

对于分类和资产定价表现，我们采用以下基准：

+   •

    未经过微调的单一GPT-4o：对于这两项任务，我们将包含新闻、因素和蜡烛图的相同提示输入到一个GPT-4o中。单一ChatGPT在其他加密货币预测工作中已有探索。

+   •

    单一的GPT-4o经过微调：对于这两项任务，我们对一个GPT-4o进行了全面的训练微调。然后，我们将包含新闻、因素和蜡烛图的相同提示输入到微调后的GPT-4o中。这种设置使我们能够将单一模型的加密货币定价能力与多代理模型进行比较，确保两者提供相同的信息。

+   •

    加密货币的风险因素：为了评估资产定价表现，我们使用[表6](https://arxiv.org/html/2501.00826v2#A1.T6 "表6 ‣ 附录A 数据描述 ‣ LLM驱动的多代理系统用于自动化加密货币投资组合管理")中列出的风险因素构建基于五分位数的投资组合，作为基准。这一方法遵循了[公式4](https://arxiv.org/html/2501.00826v2#S3.E4 "4 ‣ 3.4.2\. 加密团队 ‣ 3.4\. 多代理投资 ‣ 3\. 方法论 ‣ LLM驱动的多代理系统用于自动化加密货币投资组合管理")中描述的经验资产定价方法。此举旨在比较传统单因素模型与我们多代理模型在加密货币回报解释能力方面的差异。

#### 4.1.2\. 可解释性表现

为了评估我们多代理模型与基准模型的可解释性表现，我们引入了五个关键指标来评估解释质量。每个回答都根据这些指标，在代表性样本集上使用GPT-4o进行0到1的评分。这些指标定义如下：

+   •

    专业性：解释是否反映了金融领域的专业知识和职业素养？

+   •

    客观性：解释是否以公正、中立的方式呈现？

+   •

    清晰性与连贯性：解释是否易于理解，并且是否遵循一个能够有效连接不同因素的逻辑结构？

+   •

    一致性：解释是否与提供的数据一致，并避免矛盾？

+   •

    理由：解释是否提供了详细的推理过程，清晰地阐明了这些指标如何影响表现？

#### 4.1.3\. 投资组合表现

表1\. 我们的多代理框架与基准模型在准确性和MCC方面的性能比较。最佳结果为**粗体**。

| 子任务 | 专家代理 | 未经过微调的单一GPT-4o | 经微调的单一GPT-4o | 多代理框架（我们的方法） |
| --- | --- | --- | --- | --- |
| 准确性 | MCC | 准确性 | MCC | 准确性 | MCC |
| 加密预测 ([§3.1.2](https://arxiv.org/html/2501.00826v2#S3.SS1.SSS2 "3.1.2\. 加密货币选择 ‣ 3.1\. 问题定义 ‣ 3\. 方法论 ‣ 基于LLM的多代理系统用于自动化加密投资组合管理")) | 加密因素 | \FPeval\resultround(0.5145/0.5600:4)0.5145 | 0.0239 | \FPeval\resultround(0.5111/0.5600:4)0.5111 | 0.0053 | \FPeval\resultround(0.5177/0.5600:4)0.5177 | 0.0247 |
| 技术 | \FPeval\resultround(0.4637/0.5600:4)0.4637 | -0.0312 | \FPeval\resultround(0.4906/0.5600:4)0.4906 | -0.0216 | \FPeval\resultround(0.5118/0.5600:4)0.5118 | 0.0169 |
| 合作 | \FPeval\resultround(0.4834/0.5600:4)0.4834 | -0.0341 | \FPeval\resultround(0.5133/0.5600:4)0.5133 | 0.0191 | \FPeval\resultround(0.5248/0.5600:4)0.5248 | 0.0428 |
| 市场预测 ([§3.1.1](https://arxiv.org/html/2501.00826v2#S3.SS1.SSS1 "3.1.1\. 加密货币现金分配 ‣ 3.1\. 问题定义 ‣ 3\. 方法论 ‣ 基于LLM的多代理系统用于自动化加密投资组合管理")) | 市场因素 | \FPeval\resultround(0.5814/0.5600:4)0.5814 | 0.1612 | \FPeval\resultround(0.5581/0.5600:4)0.5581 | 0.1141 | \FPeval\resultround(0.5814/0.5600:4)0.5814 | 0.1649 |
| 新闻 | \FPeval\resultround(0.4651/0.5600:4)0.4651 | -0.0831 | \FPeval\resultround(0.5814/0.5600:4)0.5814 | 0.1993 | \FPeval\resultround(0.5581/0.5600:4)0.5581 | 0.1306 |
| 合作 | \FPeval\resultround(0.5116/0.5600:4)0.5116 | 0.0217 | \FPeval\resultround(0.5581/0.5600:4)0.5581 | 0.1307 | \FPeval\resultround(0.5814/0.5600:4)0.5814 | 0.1612 |

对于投资组合表现，我们有以下基准：

+   •

    1/N 投资组合 (DeMiguel 等人, [2009](https://arxiv.org/html/2501.00826v2#bib.bib11)): 我们构建了一个投资组合，其中篮子中的前30种加密货币等权重分配，每种加密货币获得相同的份额。

+   •

    市场投资组合 (Li 等人, [2024](https://arxiv.org/html/2501.00826v2#bib.bib28)): 我们使用纳斯达克加密指数³³3[https://www.nasdaq.com/market-activity/index/nci](https://www.nasdaq.com/market-activity/index/nci)，该指数由纳斯达克加密指数监督委员会选择的合格加密货币组成。

+   •

    BTC 投资组合 (Li 等人, [2024](https://arxiv.org/html/2501.00826v2#bib.bib28)): 我们构建了一个由100%比特币组成的买入并持有的投资组合。

为了便于我们的分析，我们根据纳斯达克加密指数定义了加密市场的繁荣和萧条时期。我们借用了(Aramonte 等人, [2022](https://arxiv.org/html/2501.00826v2#bib.bib3))的定义方法。具体而言，我们将繁荣期定义为价格低谷和峰值之间，且价格上涨超过15%的时期；将萧条期定义为价格峰值和低谷之间，且价格下跌超过15%的时期。也存在价格变化小于15%的时期，这些时期既不是繁荣期，也不是萧条期。

我们使用以下在实证资产定价中广为人知的指标来量化投资组合和资产定价表现：

+   •

    累计回报（Cumulative）(Asness et al., [2013](https://arxiv.org/html/2501.00826v2#bib.bib4)) 衡量在交易期间，组合价格的总变化，计算公式为 $\prod_{t=1}^{T}(1+r_{t})-1$，其中 $T$ 表示交易期内的总周数，$r_{t}$ 表示周回报。

+   •

    周回报均值（Mean）(Gu et al., [2020](https://arxiv.org/html/2501.00826v2#bib.bib18)) 衡量交易期间周回报的平均值，反映组合的典型周表现。

+   •

    周回报标准差（Std）(Gu et al., [2020](https://arxiv.org/html/2501.00826v2#bib.bib18)) 衡量交易期间周回报的标准差，代表组合的波动性。

+   •

    夏普比率（Sharpe）(Sharpe, [1994](https://arxiv.org/html/2501.00826v2#bib.bib35))衡量风险调整后的回报，计算公式为 $\frac{r_{t}-r_{f}}{\sigma_{t}}$，其中 $r_{t}$ 表示周回报均值，$r_{f}$ 是无风险利率，$\sigma$ 表示周回报的标准差。

### 4.2\. 性能比较

![参考说明](img/62407e6adc100ddc10a21ca554158d06.png)

(a) 加密团队的上涨概率。

![参考说明](img/e238fc326a103cf08bf9d8a43ae68578.png)

(b) 加密团队的平均分歧。

![参考说明](img/a46ee5e95e2cdb37ba0a2cf684b16a37.png)

(c) 市场团队的上涨概率。

![参考说明](img/599ca3c319a7b93514a5bf655eb08f63.png)

(d) 市场团队的平均分歧。

图 4. 不同模型中，同一组专家智能体的上涨概率预测分布和分歧。

在本小节中，我们通过与相关基准模型的定量和定性比较，评估了我们的多智能体模型在分类、组合、资产定价和解释方面的表现。

#### 4.2.1\. 分类准确率

表 2\. 我们的多智能体模型中，市场团队预测为“上涨”和“下跌”的周平均市场回报，与基准模型进行比较。“Diff”表示“上涨”和“下跌”回报之间的平均差异。

|  | 单一GPT-4o无微调 | 单一GPT-4o有微调 | 多智能体框架（我们的） |
| --- | --- | --- | --- |
| 上涨 | \FPeval\resultround(0.0210/0.0135:4)0.0210 | \FPeval\resultround(0.0186/0.0135:4)0.0186 | \FPeval\resultround(0.0264/0.0135:4)0.0264 |
| 下跌 | \FPeval\resultround(0.0104/0.0135:4)0.0104 | \FPeval\resultround(0.0051/0.0135:4)0.0051 | \FPeval\resultround(0.0032/0.0135:4)0.0032 |
| Diff | \FPeval\resultround(0.0106/0.0135:4)0.0106 | \FPeval\resultround(0.0135/0.0135:4)0.0135 | \FPeval\resultround(0.0232/0.0135:4)0.0232 |

[表 1](https://arxiv.org/html/2501.00826v2#S4.T1 "表 1 ‣ 4.1.3\. 投资组合表现 ‣ 4.1\. 实验设置 ‣ 4\. 实验 ‣ 基于大型语言模型的多智能体系统用于自动化加密货币投资组合管理") 展示了加密货币价格和市场趋势预测的定量结果。在预测准确性方面，我们的多智能体框架在加密货币预测子任务中的分类问题上取得了最佳表现，具体内容详见[§3.1.2](https://arxiv.org/html/2501.00826v2#S3.SS1.SSS2 "3.1.2\. 加密货币选择 ‣ 3.1\. 问题表述 ‣ 3\. 方法论 ‣ 基于大型语言模型的多智能体系统用于自动化加密货币投资组合管理")。该多智能体框架的表现超过了经过微调的单一 GPT-4o 模型，表明由多个专家代理组成的系统（每个代理都经过领域特定知识的训练）在加密货币价格趋势预测方面优于一个使用综合数据集训练的单一代理。此外，微调后的 GPT-4o 模型超越了未经微调的 GPT-4o 模型，表明微调使得 \acllms 能够有效地从历史数据中学习，并将这些知识应用于未来的预测。在市场预测子任务中，具体内容详见[§3.1.1](https://arxiv.org/html/2501.00826v2#S3.SS1.SSS1 "3.1.1\. 加密货币现金分配 ‣ 3.1\. 问题表述 ‣ 3\. 方法论 ‣ 基于大型语言模型的多智能体系统用于自动化加密货币投资组合管理")，当输入包含加密货币特定因素并且结果通过集成方式处理时，多智能体框架取得了最佳表现。然而，当输入为包含新闻头条数据的提示时，单一 GPT-4o 模型的表现超过了多智能体框架。一种可能的解释是，经过综合数据集训练的单一 \acllm 可能在从新闻内容中提取隐含的市场因素相关见解方面表现更好，例如加密货币的关注度增加或网络效应。

对于此任务，更具信息量的指标是\acmcc，因为它考虑了预测中的真正例、真反例、假正例和假反例的比率（Chicco 和 Jurman，[2020](https://arxiv.org/html/2501.00826v2#bib.bib8)；Chicco 等，[2021](https://arxiv.org/html/2501.00826v2#bib.bib9)）。由于并非所有加密货币价格和市场趋势都能通过现有数据完全解释，单纯的准确度可能无法充分反映多智能体模型的分类能力。考虑到并非所有加密货币价格和市场趋势都能通过现有数据完全解释，准确度结果可能并不能充分显示多智能体模型的分类能力，因为它包含了一些对噪声的随机猜测。在\acmcc指标下，我们的多智能体框架在所有任务中均优于其他模型，除非输入由包含新闻标题的提示组成。这证明了模型的真正分类能力，因为\acmcc考虑了随机猜测的影响，并提供了更为稳健的性能评估。

![参见说明](img/c26a03ae1f4fff07b2b1c4309495a6a1.png)

(a) 以美元计的累积回报。

![参见说明](img/7847589998dd3994a4217d84b78d5322.png)

(b) 以比特币计的累积回报。

图 5. 我们的多智能体模型投资组合与基准模型在样本外累积回报的表现比较。绿色区域表示繁荣期，而红色区域表示萧条期，这由纳斯达克加密货币指数（市场）确定。

我们的多智能体模型在预测中的优异表现可以部分归因于微调过程。为了验证这一点，我们可视化了从单个未经过微调的GPT-4o、单个经过微调的GPT-4o以及我们的多智能体模型输出中提取的上升概率分布，如[4(a)](https://arxiv.org/html/2501.00826v2#S4.F4.sf1 "4(a) ‣ Figure 4 ‣ 4.2\. Performance Comparison ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")和[4(c)](https://arxiv.org/html/2501.00826v2#S4.F4.sf3 "4(c) ‣ Figure 4 ‣ 4.2\. Performance Comparison ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")所示。我们观察到，微调前的上升概率分布呈U型，而微调后的分布则更为集中，且与正态分布或对数正态分布更加吻合。鉴于个别加密货币回报和市场回报的分布通常更接近正态分布或对数正态分布，我们得出结论，微调过程使得\acpllm能够更好地学习并反映加密回报的经验分布。为了评估同一组内不同智能体的预测差异程度，我们还计算了线性“上升”概率的标准差，作为分歧的指标。[4(b)](https://arxiv.org/html/2501.00826v2#S4.F4.sf2 "4(b) ‣ Figure 4 ‣ 4.2\. Performance Comparison ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")和[4(d)](https://arxiv.org/html/2501.00826v2#S4.F4.sf4 "4(d) ‣ Figure 4 ‣ 4.2\. Performance Comparison ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")展示了我们多智能体模型与基准模型之间的平均分歧水平。我们观察到，经过微调的模型在市场团队和加密团队中展现出较低的平均分歧。这表明，微调使得专家智能体能够更好地从历史数据中学习，避免了随机猜测。

此外，由于现金-加密货币配置策略直接源自市场团队的分类结果，因此有必要评估其财务意义。[表 2](https://arxiv.org/html/2501.00826v2#S4.T2 "Table 2 ‣ 4.2.1\. Classification Accuracy ‣ 4.2\. Performance Comparison ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")报告了我们多智能体模型中市场团队预测为“上涨”和“下跌”的周平均市场回报，并与基准模型进行比较。我们观察到，我们的多智能体模型预测为“上涨”的周的平均回报是所有模型中最高的，而预测为“下跌”的周的回报则是最低的。这表明我们多智能体模型中的市场团队具有最强的能力来区分市场的繁荣与衰退。

#### 4.2.2\. 投资组合表现

表 3\. 完整、繁荣和衰退期的投资组合结果比较。“平均”、“标准差”和“夏普比率”列分别提供了每周实现的平均回报、其标准差和年化夏普比率。

| 期间 | 投资组合 | 平均 | 标准差 | 夏普比率 |
| --- | --- | --- | --- | --- |
| 所有 | 我们的模型 | 0.0172 | $0.0805$ | $1.5425$ |
| 市场 | 0.0131 | $0.0683$ | $1.3781$ |
| 1/N | 0.0082 | $0.0834$ | $0.7070$ |
| 比特币 | 0.0144 | $0.0677$ | $1.5340$ |
| 繁荣期 | 我们的模型 | 0.0428 | $0.0802$ | $3.8430$ |
| 市场 | 0.0422 | $0.0630$ | $4.8279$ |
| 1/N | 0.0391 | $0.0758$ | $3.7195$ |
| 比特币 | 0.0404 | $0.0634$ | $4.5977$ |
| 衰退期 | 我们的模型 | -0.0269 | $0.0715$ | $-2.7125$ |
| 市场 | -0.0359 | $0.0529$ | $-4.8944$ |
| 1/N | -0.0479 | $0.0723$ | $-4.7748$ |
| 比特币 | -0.0299 | $0.0519$ | $-4.1570$ |

除了分类准确性，我们的多智能体模型实现的投资组合表现也至关重要。[图 5](https://arxiv.org/html/2501.00826v2#S4.F5 "Figure 5 ‣ 4.2.1\. Classification Accuracy ‣ 4.2\. Performance Comparison ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management") 展示了我们的多智能体模型与市场指数和前30大加密货币的等权重投资组合（1/N）相比的样本外累积收益。从[5(a)](https://arxiv.org/html/2501.00826v2#S4.F5.sf1 "5(a) ‣ Figure 5 ‣ 4.2.1\. Classification Accuracy ‣ 4.2\. Performance Comparison ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")可以观察到，除2024年2月外，我们的模型生成的投资组合在整个样本期间均优于两种构建的指数。

100% 持有比特币策略在投资者中非常流行。为了评估我们多智能体模型相对于该策略的表现，我们比较了它们的累计回报。[5(b)](https://arxiv.org/html/2501.00826v2#S4.F5.sf2 "5(b) ‣ Figure 5 ‣ 4.2.1\. Classification Accuracy ‣ 4.2\. Performance Comparison ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management") 展示了我们模型的累计回报与100%持有比特币策略的累计回报之比。结果表明，除了2024年2月外，我们模型的累计回报始终超过了持有比特币策略的回报。

[Tab. 3](https://arxiv.org/html/2501.00826v2#S4.T3 "Table 3 ‣ 4.2.2\. Portfolio Performance ‣ 4.2\. Performance Comparison ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management") 展示了在完全、繁荣和萧条期的投资组合结果对比。表格显示，我们的多智能体模型在所有时期的投资组合指标中均优于其他方法，包括繁荣和萧条阶段，同时在标准差方面保持了相当的表现。值得注意的是，我们的模型在萧条期表现出强大的抗跌能力，进一步凸显了其有效性。

#### 4.2.3\. 资产定价表现

表4\. 我们的多智能体模型与基准之间的样本外五分位投资组合表现对比。根据预测的“上涨”概率（见上图面板）和下一周的顶级风险因子因素值（见下图面板），将前30种加密货币按五分位排序。 “HML”表示一种策略，即做多“非常高”投资组合，做空“非常低”投资组合。列“平均值”、“标准差”和“Sharpe”分别提供了平均的每周实现回报、标准差和Sharpe比率。所有投资组合均为等权重。^∗、^(∗∗) 和 ^(∗∗∗) 分别表示在10%、5%和1%显著性水平下的统计意义。我们只选择根据其“HML”投资组合回报的前三个风险因子。

(a) \acllm 基础模型。

| 专家代理 | 投资组合 | 单个未经调优的 GPT-4o | 单个经过调优的 GPT-4o | 我们的多智能体框架 |
| --- | --- | --- | --- | --- |
| 平均值 | 标准差 | Sharpe 比率 | 平均值 | 标准差 | Sharpe 比率 | 平均值 | 标准差 | Sharpe 比率 |
| 加密因子 | 非常低 | \FPeval\resultround(0.0066/0.0130:4)0.0066 | $0.0919$ | $0.0713$ | \FPeval\resultround(0.0086/0.0130:4)0.0086 | $0.0772$ | $0.1112$ | \FPeval\resultround(0.0036/0.0130:4)0.0036 | $0.0785$ | $0.0464$ |
| 低 | \FPeval\resultround(0.0066/0.0130:4)0.0066 | $0.1038$ | $0.0633$ | \FPeval\resultround(0.0115/0.0130:4)0.0115 | $0.0925$ | $0.1239$ | \FPeval\resultround(0.0093/0.0130:4)0.0093 | $0.0944$ | $0.0983$ |
| 中 | \FPeval\resultround(0.0019/0.0130:4)0.0019 | $0.0898$ | $0.0210$ | \FPeval\resultround(0.0090/0.0130:4)0.0090 | $0.0963$ | $0.0933$ | \FPeval\resultround(0.0072/0.0130:4)0.0072 | $0.0850$ | $0.0851$ |
| 高 | \FPeval\resultround(0.0110/0.0130:4)0.0110 | $0.0807$ | $0.1360$ | \FPeval\resultround(-0.0001/0.0130:4)-0.0001 | $0.0865$ | $-0.0012$ | \FPeval\resultround(0.0067/0.0130:4)0.0067 | $0.0934$ | $0.0719$ |
| 非常高 | \FPeval\resultround(0.0153/0.0130:4)0.0153 | $0.0843$ | $0.1810$ | \FPeval\resultround(0.0122/0.0130:4)0.0122 | $0.0947$ | $0.1283$ | \FPeval\resultround(0.0144/0.0130:4)0.0144 | $0.0956$ | $0.1510$ |
| HML | \FPeval\resultround(0.0087/0.0130:4)0.0087 | $0.0630$ | $0.1382$ | \FPeval\resultround(0.0036/0.0130:4)0.0036 | $0.0589$ | $0.0606$ | \FPeval\resultround(0.0108/0.0130:4)0.0108 | $0.0611$ | $0.1766$ |
|  |  | 均值 | 标准差 | 夏普比率 | 均值 | 标准差 | 夏普比率 | 均值 | 标准差 | 夏普比率 |
| 技术 | 非常低 | \FPeval\resultround(0.0057/0.0130:4)0.0057 | $0.0774$ | $0.0740$ | \FPeval\resultround(0.0108/0.0130:4)0.0108 | $0.0787$ | $0.1369$ | \FPeval\resultround(0.0038/0.0130:4)0.0038 | $0.0791$ | $0.0475$ |
| 低 | \FPeval\resultround(0.0048/0.0130:4)0.0048 | $0.0939$ | $0.0507$ | \FPeval\resultround(0.0069/0.0130:4)0.0069 | $0.1026$ | $0.0674$ | \FPeval\resultround(0.0055/0.0130:4)0.0055 | $0.0995$ | $0.0553$ |
| 中 | \FPeval\resultround(0.0015/0.0130:4)0.0015 | $0.0975$ | $0.0157$ | \FPeval\resultround(0.0049/0.0130:4)0.0049 | $0.0913$ | $0.0535$ | \FPeval\resultround(0.0082/0.0130:4)0.0082 | $0.0850$ | $0.0960$ |
| 高 | \FPeval\resultround(0.0172/0.0130:4)0.0172 | $0.0925$ | $0.1860$ | \FPeval\resultround(0.0032/0.0130:4)0.0032 | $0.0829$ | $0.0384$ | \FPeval\resultround(0.0132/0.0130:4)0.0132 | $0.0903$ | $0.1461$ |
| 非常高 | \FPeval\resultround(0.0119/0.0130:4)0.0119 | $0.0847$ | $0.1407$ | \FPeval\resultround(0.0152/0.0130:4)0.0152 | $0.0870$ | $0.1749$ | \FPeval\resultround(0.0103/0.0130:4)0.0103 | $0.0869$ | $0.1187$ |
| HML | \FPeval\resultround(0.0062/0.0130:4)0.0062 | $0.0586$ | $0.1057$ | \FPeval\resultround(0.0044/0.0130:4)0.0044 | $0.0590$ | $0.0752$ | \FPeval\resultround(0.0066/0.0130:4)0.0066 | $0.0524$ | $0.1250$ |
|  |  | 均值 | 标准差 | 夏普比率 | 均值 | 标准差 | 夏普比率 | 均值 | 标准差 | 夏普比率 |
| 合作 | 非常低 | \FPeval\resultround(0.0058/0.0130:4)0.0058 | $0.0900$ | $0.0640$ | \FPeval\resultround(0.0096/0.0130:4)0.0096 | $0.0776$ | $0.1235$ | \FPeval\resultround(-0.0009/0.0130:4)-0.0009 | $0.0792$ | $-0.0116$ |
| 低 | \FPeval\resultround(0.0070/0.0130:4)0.0070 | $0.1016$ | $0.0688$ | \FPeval\resultround(0.0086/0.0130:4)0.0086 | $0.1046$ | $0.0827$ | \FPeval\resultround(0.0140/0.0130:4)0.0140 | $0.0958$ | $0.1462$ |
| 中 | \FPeval\resultround(0.0101/0.0130:4)0.0101 | $0.0974$ | $0.1039$ | \FPeval\resultround(0.0089/0.0130:4)0.0089 | $0.0915$ | $0.0971$ | \FPeval\resultround(0.0041/0.0130:4)0.0041 | $0.0838$ | $0.0492$ |
| 高 | \FPeval\resultround(0.0091/0.0130:4)0.0091 | $0.0801$ | $0.1131$ | \FPeval\resultround(0.0046/0.0130:4)0.0046 | $0.0972$ | $0.0472$ | \FPeval\resultround(0.0079/0.0130:4)0.0079 | $0.0951$ | $0.0832$ |
| 非常高 | \FPeval\resultround(0.0094/0.0130:4)0.0094 | $0.0849$ | $0.1107$ | \FPeval\resultround(0.0100/0.0130:4)0.0100 | $0.0813$ | $0.1230$ | \FPeval\resultround(0.0160/0.0130:4)0.0160 | $0.0899$ | $0.1779$ |
| HML | \FPeval\resultround(0.0036/0.0130:4)0.0036 | $0.0695$ | $0.0523$ | \FPeval\resultround(0.0004/0.0130:4)0.0004 | $0.0512$ | $0.0081$ | \FPeval\resultround(0.0169/0.0130:4)0.0169^(**)** | $0.0558$ | $0.3030$ |

(b) 加密货币中的最佳风险因素。

| 因素 | 投资组合 | MOM 1,0 | MOM 4,0 | MOM 4,1 |
| --- | --- | --- | --- | --- |
| 平均值 | 标准差 | 夏普比率 | 平均值 | 标准差 | 夏普比率 | 平均值 | 标准差 | 夏普比率 |
| 顶级因素 | 非常低 | \FPeval\resultround(-0.0043/0.0140:4)-0.0043 | $0.0853$ | $-0.0499$ | \FPeval\resultround(-0.0014/0.0140:4)-0.0014 | $0.0888$ | $-0.0157$ | \FPeval\resultround(0.0008/0.0140:4)0.0008 | $0.0838$ | $0.0101$ |
| 低 | \FPeval\resultround(0.0082/0.0140:4)0.0082 | $0.0850$ | $0.0961$ | \FPeval\resultround(0.0111/0.0140:4)0.0111 | $0.1012$ | $0.1098$ | \FPeval\resultround(0.0136/0.0140:4)0.0136 | $0.1055$ | $0.1291$ |
| 中等 | \FPeval\resultround(0.0152/0.0140:4)0.0152 | $0.1071$ | $0.1421$ | \FPeval\resultround(0.0148/0.0140:4)0.0148 | $0.0952$ | $0.1558$ | \FPeval\resultround(0.0119/0.0140:4)0.0119 | $0.0959$ | $0.1242$ |
| 高 | \FPeval\resultround(0.0117/0.0140:4)0.0117 | $0.0862$ | $0.1359$ | \FPeval\resultround(0.0117/0.0140:4)0.0117 | $0.0826$ | $0.1414$ | \FPeval\resultround(0.0123/0.0140:4)0.0123 | $0.0915$ | $0.1340$ |
| 非常高 | \FPeval\resultround(0.0103/0.0140:4)0.0103 | $0.0872$ | $0.1180$ | \FPeval\resultround(0.0051/0.0140:4)0.0051 | $0.0877$ | $0.0578$ | \FPeval\resultround(0.0029/0.0140:4)0.0029 | $0.0810$ | $0.0359$ |
| HML | \FPeval\resultround(0.0146/0.0140:4)0.0146 | $0.0677$ | $0.2156$ | \FPeval\resultround(0.0073/0.0140:4)0.0073 | $0.0618$ | $0.1180$ | \FPeval\resultround(0.0027/0.0140:4)0.0027 | $0.0516$ | $0.0524$ |

本节中，我们评估了我们多智能体框架在加密货币定价中的表现。在加密市场中，单一加密货币的趋势不仅限于二元结果（即上涨或下跌），而是存在于一个连续的范围内。因此，模型还应该具备有效解释跨期加密货币收益变化的能力。[4(b)](https://arxiv.org/html/2501.00826v2#S4.T4.st2 "4(b) ‣ 表4 ‣ 4.2.3\. 资产定价表现 ‣ 4.2\. 性能比较 ‣ 4\. 实验 ‣ 基于LLM的自动化加密货币投资组合管理多智能体系统")报告了我们的多智能体模型与基准模型的样本外五分位数投资组合的性能比较。前30种加密货币根据其预测的“上涨”概率被排序为五个分位数（见[4(a)](https://arxiv.org/html/2501.00826v2#S4.T4.st1 "4(a) ‣ 表4 ‣ 4.2.3\. 资产定价表现 ‣ 4.2\. 性能比较 ‣ 4\. 实验 ‣ 基于LLM的自动化加密货币投资组合管理多智能体系统")）以及根据[4(b)](https://arxiv.org/html/2501.00826v2#S4.T4.st2 "4(b) ‣ 表4 ‣ 4.2.3\. 资产定价表现 ‣ 4.2\. 性能比较 ‣ 4\. 实验 ‣ 基于LLM的自动化加密货币投资组合管理多智能体系统")中的风险因子因子值，按照[公式4](https://arxiv.org/html/2501.00826v2#S3.E4 "4 ‣ 3.4.2\. 加密团队 ‣ 3.4\. 多智能体投资 ‣ 3\. 方法论 ‣ 基于LLM的自动化加密货币投资组合管理多智能体系统")预测的下周表现。我们报告了平均实现的每周回报、其标准差以及夏普比率。所有投资组合均为等权重。从[4(b)](https://arxiv.org/html/2501.00826v2#S4.T4.st2 "4(b) ‣ 表4 ‣ 4.2.3\. 资产定价表现 ‣ 4.2\. 性能比较 ‣ 4\. 实验 ‣ 基于LLM的自动化加密货币投资组合管理多智能体系统")中，我们做出以下观察：

+   •

    通过我们多智能体框架中不同代理的协作识别出的“非常高”（Very High）和“HML”投资组合，表现优于未经微调的单一GPT-4o模型生成的投资组合。这一改进可归因于微调过程，使得代理能够从历史加密货币趋势数据中学习，正如在[§4.2.1](https://arxiv.org/html/2501.00826v2#S4.SS2.SSS1 "4.2.1\. 分类准确率 ‣ 4.2\. 性能比较 ‣ 4\. 实验 ‣ 基于LLM的自动化加密货币投资组合管理多智能体系统")中讨论的那样。

+   •

    在我们框架下通过合作生成的最终“非常高”（Very High）和“HML”投资组合，优于通过单一的GPT-4o模型微调后生成的投资组合。这种卓越的表现可以归因于单独训练专门化智能体并采用团队内部合作机制的优势，正如[3(a)](https://arxiv.org/html/2501.00826v2#S3.F3.sf1 "3(a) ‣ Figure 3 ‣ 3.4.1\. Market Team ‣ 3.4\. Multi-Agent Investment ‣ 3\. Methodology ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")中所示。不同领域的专家智能体之间的合作有助于减少预测误差，同时提高正确预测的准确性。相比之下，通过单一智能体生成的多个结果的集成预测只能带来有限的改进，因为同一智能体的输出往往存在一定的矛盾。

+   •

    我们的多智能体模型的表现超过了三种最有效的风险因素。虽然只有少数风险因素表现出强大的预测能力，但我们的加密因子专家智能体能够横向学习这些风险因素与加密货币趋势之间的关系。这个能力使它能够有效识别未来回报非常高的加密货币，从而为整体多智能体框架作出重要贡献。

+   •

    由我们多智能体模型合作生成的“非常高”（Very High）和“HML”投资组合的表现，超过了模型内各个独立专家智能体生成的投资组合。这进一步验证了团队内部合作机制在提升我们多智能体框架资产定价能力方面的有效性。

+   •

    由我们多智能体框架合作生成的“HML”投资组合的回报在5%水平上具有显著性。这表明，从金融学中经验性资产定价的角度来看，我们的多智能体模型在解释横截面加密货币回报的变化方面具有强大的能力。

#### 4.2.4. 解释性表现

![请参阅图例](img/6a838a65c8abf6ca571dc25977c9c5d3.png)

图6. 我们的加密因子专家智能体经过微调后的示例输出与未经过微调的GPT-4o模型的比较。我们突出了模型从(Liu et al., [2022](https://arxiv.org/html/2501.00826v2#bib.bib33))中学到的资产定价术语。

使用\acpllm进行预测相较于传统深度学习方法的一个关键优势是能够生成自然语言的解释（Koa等，[2024](https://arxiv.org/html/2501.00826v2#bib.bib23)）。在加密货币投资组合管理的背景下，我们将模型可解释性定义为能够为加密货币和市场趋势预测生成基于金融领域专业资产定价知识的理由。[图6](https://arxiv.org/html/2501.00826v2#S4.F6 "Figure 6 ‣ 4.2.4\. Explanation Performance ‣ 4.2\. Performance Comparison ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")对比了我们的加密因子专家代理与经过微调的GPT-4o和未微调的GPT-4o的示例输出。我们观察到，在微调后生成的专家代理的解释包含了显著更多来自提供文献的资产定价术语。这表明，由解释团队标注的训练提示与微调过程相结合，显著提升了专家代理的可解释性能力。

此外，[图7](https://arxiv.org/html/2501.00826v2#S4.F7 "Figure 7 ‣ 4.2.4\. Explanation Performance ‣ 4.2\. Performance Comparison ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")报告了[§4.1.2](https://arxiv.org/html/2501.00826v2#S4.SS1.SSS2 "4.1.2\. Explainability Performance ‣ 4.1\. Experiment Settings ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")中概述的每个指标的平均得分。从[图7](https://arxiv.org/html/2501.00826v2#S4.F7 "Figure 7 ‣ 4.2.4\. Explanation Performance ‣ 4.2\. Performance Comparison ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")中，我们观察到，微调后的模型在除一致性外的所有指标上都优于未微调的模型。这表明，微调过程显著增强了大多数方面的可解释性。然而，在一致性方面，单一的经过微调的GPT-4o模型并未优于未微调的GPT-4o模型。这表明，使用所有历史数据对单一代理进行微调可能会在分析过程中引入矛盾，从而削弱了解释的一致性。此外，我们的多代理模型在所有指标上均优于单一的经过微调的GPT-4o模型。这突显了多代理框架的优势，在该框架中，每个经过训练的领域特定专家代理能够比单一的通用代理生成更准确和专业的解释。

![请参阅图注](img/51912d42854bb7282c52cdddba873b2c.png)

图7. 我们的多代理模型与基准模型的解释质量比较

### 4.3\. 消融研究

表 5\. 我们的多代理模型的消融研究。我们通过逐个去除每个模块并观察各项指标的变化，评估各个模块的贡献。$\CIRCLE$ 表示保留的组件，而 $\Circle$ 表示去除的组件。“Mean”、“Std”和“Sharpe”分别表示平均实现的每周回报、标准差和年化夏普比率。最佳结果用粗体显示。

| 模型中的代理和协作机制 | 累计 | 平均 | 标准差 | 夏普比率 |
| --- | --- | --- | --- | --- |
| 团队内部协作 | 团队间协作 |
| 加密因子 | 技术 | 市场因子 | 新闻 |
| ● | ● | ● | ● | ● | 0.8347 | 0.0172 | $0.0805$ | 1.5425 |
| ○ | ● | ● | ● | ● | $\pagecolor[HTML]{ffffff}0.4707$ | $0.0115$ | 0.0729 | $1.1395$ |
| ● | ○ | ● | ● | ● | $\pagecolor[HTML]{f7f9f5}0.5003$ | $0.0123$ | $0.0784$ | $1.1354$ |
| ● | ● | ○ | ● | ● | $\pagecolor[HTML]{bbd3af}0.7168$ | $0.0160$ | $0.0826$ | $1.3968$ |
| ● | ● | ● | ○ | ● | $\pagecolor[HTML]{bfd6b4}0.7024$ | $0.0157$ | $0.0834$ | $1.3576$ |
| ● | ● | ● | ● | ○ | $\pagecolor[HTML]{a0c290}0.8132$ | $0.0166$ | $0.0802$ | $1.4926$ |

在本节中，我们评估了多代理模型中各个组件对投资组合表现的贡献。[表 5](https://arxiv.org/html/2501.00826v2#S4.T5 "Table 5 ‣ 4.3\. Ablation Study ‣ 4\. Experiment ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")展示了消融研究的结果，在该研究中，我们系统地去除了关键组件或机制，以评估它们对整体投资组合表现的影响。我们从结果中提炼出以下见解：

团队内部协作的优势：我们观察到，去除任何一个代理都会导致累计收益、平均收益和夏普比率的下降，突显了每个代理通过团队内部协作机制对整体投资组合表现的重要贡献。在没有团队内部协作的情况下，同一团队内部不同的意见无法有效统一。因此，这一结果证明了团队内部协作机制在提高预测准确性和投资决策中的有效性。

团队间协作的优势：我们观察到，禁用团队间协作机制会导致累计回报、平均回报和夏普比率的降低。这一发现表明，团队间协作机制通过将市场信息融入加密团队内部代理的决策过程，增强了我们多代理模型的整体投资组合表现。

## 5\. 结论

在这项工作中，我们探讨了可解释的加密货币投资任务，这是一个具有挑战性的问题，因为与传统资产相比，加密货币的历史较短，信息来源多样，而且市场波动性高。虽然\acpllm的引入已经彻底改变了金融领域，但单一\acllms的表现仍然有限。为了解决这些挑战，我们提出了一个可解释的、多模态的、多代理框架，该框架通过多个代理团队的协作，既在团队内部又跨团队协作，实现监督学习和在市值排名前30的加密货币中的投资决策。我们的实验结果表明，我们的模型在分类准确性和资产定价表现方面，优于单代理模型（无论是否进行微调）。此外，我们还证明，我们的框架在投资组合表现上超越了市场基准。

###### 致谢

本材料部分基于Ripple公司在大学区块链研究计划（UBRI）下的资助工作（Feng等，[2022](https://arxiv.org/html/2501.00826v2#bib.bib16)）。本材料中表达的任何观点、发现、结论或建议仅代表作者个人意见，并不一定反映Ripple的观点。

## 参考文献

+   （1）

+   Anamika和Subramaniam（2022）Anamika Anamika和Sowmya Subramaniam。2022年。新闻标题在加密货币市场中重要吗？*应用经济学*（2022年11月）。[https://doi.org/10.1080/00036846.2022.2061904](https://doi.org/10.1080/00036846.2022.2061904)

+   Aramonte等（2022）Sirio Aramonte，Sebastian Doerr，Wenqian Huang和Andreas Schrimpf。2022年。DeFi借贷：没有信息的中介？*BIS公报*（2022年6月）。[https://ideas.repec.org/p/bis/bisblt/57.htmlhttps://ideas.repec.org//p/bis/bisblt/57.html](https://ideas.repec.org/p/bis/bisblt/57.htmlhttps://ideas.repec.org//p/bis/bisblt/57.html)

+   Asness等（2013）Clifford S. Asness，Tobias J. Moskowitz和Lasse Heje Pedersen。2013年。价值与动量无处不在。*金融杂志* 68，3（2013年6月），929–985。[https://doi.org/10.1111/JOFI.12021](https://doi.org/10.1111/JOFI.12021)

+   Biran和McKeown（2017）Or Biran和Kathleen McKeown。2017年。以人为本的机器学习预测解释。*IJCAI国际人工智能联合会议* 0（2017），1461–1467。[https://doi.org/10.24963/IJCAI.2017/202](https://doi.org/10.24963/IJCAI.2017/202)

+   Carta等（2021）Salvatore M. Carta，Sergio Consoli，Luca Piras，Alessandro Sebastian Podda和Diego Reforgiato Recupero。2021年。利用新闻和领域特定词汇进行可解释的机器学习以预测股市。*IEEE Access* 9（2021），30193–30205。[https://doi.org/10.1109/ACCESS.2021.3059960](https://doi.org/10.1109/ACCESS.2021.3059960)

+   CFP 理事会（2022）CFP 理事会。2022年。CFP 理事会发布加密货币指南。 [https://www.cfp.net/news/2022/12/cfp-board-issues-crypto-guidelines](https://www.cfp.net/news/2022/12/cfp-board-issues-crypto-guidelines)

+   Chicco 和 Jurman（2020）Davide Chicco 和 Giuseppe Jurman。2020年。马修斯相关系数（MCC）在二分类评估中优于F1得分和准确率的优势。*BMC 基因组学* 21, 1 (2020年1月)，1–13。 [https://doi.org/10.1186/S12864-019-6413-7/TABLES/5](https://doi.org/10.1186/S12864-019-6413-7/TABLES/5)

+   Chicco 等人（2021）Davide Chicco, Niklas Tötsch 和 Giuseppe Jurman。2021年。马修斯相关系数（MCC）在二分类混淆矩阵评估中比平衡准确率、博彩信息度和标记度更可靠。 *生物数据挖掘* 14, 1 (2021年2月)，1–22。 [https://doi.org/10.1186/S13040-021-00244-Z/TABLES/5](https://doi.org/10.1186/S13040-021-00244-Z/TABLES/5)

+   Corbet 等人（2019）Shaen Corbet, Brian Lucey, Andrew Urquhart 和 Larisa Yarovaya。2019年。加密货币作为金融资产：一个系统分析。*国际金融分析评论* 62 (3 2019)，182–199。 [https://doi.org/10.1016/J.IRFA.2018.09.003](https://doi.org/10.1016/J.IRFA.2018.09.003)

+   DeMiguel 等人（2009）Victor DeMiguel, Lorenzo Garlappi 和 Raman Uppal。2009年。最佳与天真分散化：1/N 投资组合策略有多低效？ *金融学研究评论* 22, 5 (2009年5月)，1915–1953。 [https://doi.org/10.1093/RFS/HHM075](https://doi.org/10.1093/RFS/HHM075)

+   Ding 等人（2024）Qianggang Ding, Haochen Shi 和 Bang Liu。2024年。TradExpert：用专家混合大模型革命化交易。（2024年10月）。 [https://arxiv.org/abs/2411.00782v1](https://arxiv.org/abs/2411.00782v1)

+   Fama 和 French（1993）Eugene F. Fama 和 Kenneth R. French。1993年。股票和债券回报中的共同风险因子。 *金融经济学杂志* 33, 1 (1993年2月)，3–56。 [https://doi.org/10.1016/0304-405X(93)90023-5](https://doi.org/10.1016/0304-405X(93)90023-5)

+   Fang 等人（2022）Fan Fang, Carmine Ventre, Michail Basios, Leslie Kanthan, David Martinez-Rego, Fan Wu 和 Lingbo Li。2022年。加密货币交易：一项综合调查。 *金融创新* 8, 1 (2022年12月)，1–59。 [https://doi.org/10.1186/S40854-021-00321-6/TABLES/11](https://doi.org/10.1186/S40854-021-00321-6/TABLES/11)

+   Fatemi 和 Hu（2024）Sorouralsadat Fatemi 和 Yuheng Hu。2024年。FinVision：一种多智能体框架用于股市预测。 *第五届 ACM 国际金融人工智能会议论文集* （2024年10月），582–590。 [https://doi.org/10.1145/3677052.3698688](https://doi.org/10.1145/3677052.3698688)

+   Feng 等人（2022）Yebo Feng, Jiahua Xu 和 Lauren Weymouth。2022年。大学区块链研究计划（UBRI）：推动区块链教育与研究。 *IEEE 潜力* 41, 6 (2022年)，19–25。

+   Goutte等（2023）Stéphane Goutte、Hoang Viet Le、Fei Liu和Hans Jörg von Mettenheim。2023年。深度学习与技术分析在加密货币市场中的应用。*金融研究快报* 54（2023年6月），103809。 [https://doi.org/10.1016/J.FRL.2023.103809](https://doi.org/10.1016/J.FRL.2023.103809)

+   Gu等（2020）Shihao Gu、Bryan Kelly和Dacheng Xiu。2020年。通过机器学习的经验资产定价。*金融研究评论* 33, 5（2020年5月），2223–2273。 [https://doi.org/10.1093/RFS/HHAA009](https://doi.org/10.1093/RFS/HHAA009)

+   Hackethal等（2022）Andreas Hackethal、Tobin Hanspal、Dominique M. Lammer和Kevin Rink。2022年。比特币投资者的特征和投资组合行为：来自间接加密货币投资的证据。*金融评论* 26, 4（2022年7月），855–898。 [https://doi.org/10.1093/ROF/RFAB034](https://doi.org/10.1093/ROF/RFAB034)

+   Hudson和Urquhart（2021）Robert Hudson和Andrew Urquhart。2021年。技术交易与加密货币。*运筹学年刊* 297, 1-2（2021年2月），191–220。 [https://doi.org/10.1007/S10479-019-03357-1/TABLES/9](https://doi.org/10.1007/S10479-019-03357-1/TABLES/9)

+   Jing和Kang（2024）Liu Jing和Yuncheol Kang。2024年。利用集成深度强化学习的自动化加密货币交易方法：学习理解蜡烛图。*专家系统与应用* 237（2024年3月），121373。 [https://doi.org/10.1016/J.ESWA.2023.121373](https://doi.org/10.1016/J.ESWA.2023.121373)

+   Kapur等（2024）Geeta Kapur、Sridhar Manohar、Amit Mittal、Vishal Jain和Sonal Trivedi。2024年。通过比特币和以太坊的蜡烛图模式与机器学习进行加密货币价格波动和时间序列分析。*国际质量与可靠性管理杂志* 41, 8（2024年9月），2055–2074。 [https://doi.org/10.1108/IJQRM-12-2022-0363/FULL/PDF](https://doi.org/10.1108/IJQRM-12-2022-0363/FULL/PDF)

+   Koa等（2024）Kelvin J.L. Koa、Yunshan Ma、Ritchie Ng和Tat Seng Chua。2024年。利用自反大语言模型生成可解释的股票预测。*WWW 2024 - ACM Web会议论文集* 12（2024年5月），4304–4315。 [https://doi.org/10.1145/3589334.3645611/SUPPL{_}FILE/RFP1792.MP4](https://doi.org/10.1145/3589334.3645611/SUPPL%7B_%7DFILE/RFP1792.MP4)

+   Kou等（2024）Zhizhuo Kou、Holam Yu、Jingshu Peng、香港和中国Lei Chen。2024年。利用LLM自动化量化投资策略寻找。（2024年9月）。 [https://arxiv.org/abs/2409.06289v1](https://arxiv.org/abs/2409.06289v1)

+   Lahmiri和Bekiros（2019）Salim Lahmiri和Stelios Bekiros。2019年。利用深度学习混沌神经网络进行加密货币预测。*混沌、孤立子与分形* 118（2019年1月），35–40。 [https://doi.org/10.1016/J.CHAOS.2018.11.014](https://doi.org/10.1016/J.CHAOS.2018.11.014)

+   Li et al. (2023a) Lezhi Li, Ting-Yu Chang 和 Hai Wang. 2023a. 基于多模态生成AI的基本面投资研究。(2023年12月). [https://arxiv.org/abs/2401.06164v1](https://arxiv.org/abs/2401.06164v1)

+   Li et al. (2023b) Shuqi Li, Weiheng Liao, Yuhan Chen 和 Rui Yan. 2023b. PEN: 用于预测股票价格波动并提供更好解释性的预测-解释网络。*美国人工智能大会论文集* 37, 4 (2023年6月), 5187–5194. [https://doi.org/10.1609/AAAI.V37I4.25648](https://doi.org/10.1609/AAAI.V37I4.25648)

+   Li et al. (2024) Yuan Li, Bingqiao Luo, Qian Wang, Nuo Chen, Xu Liu 和 Bingsheng He. 2024. 基于反思的LLM代理指导零-shot加密货币交易。(2024年6月). [https://arxiv.org/abs/2407.09546v1](https://arxiv.org/abs/2407.09546v1)

+   Li et al. (2023c) Yinheng Li, Shaofei Wang, Han Ding 和 Hang Chen. 2023c. 金融领域中的大型语言模型：一项调查。(2023年). [https://doi.org/10.1145/3604237.3626869](https://doi.org/10.1145/3604237.3626869)

+   Liu et al. (2023a) Hao Liu, Carmelo Sferrazza 和 Pieter Abbeel. 2023a. 回溯链使语言模型与反馈对齐。*第12届国际学习表征会议, ICLR 2024* (2023年2月). [https://arxiv.org/abs/2302.02676v8](https://arxiv.org/abs/2302.02676v8)

+   Liu et al. (2023b) Xiao-Yang Liu, Guoxuan Wang, Hongyang Yang 和 Daochen Zha. 2023b. FinGPT: 为金融领域的大型语言模型普及互联网规模的数据。(2023年7月). [https://arxiv.org/abs/2307.10485v2](https://arxiv.org/abs/2307.10485v2)

+   Liu and Tsyvinski (2021) Yukun Liu 和 Aleh Tsyvinski. 2021. 加密货币的风险与回报。*金融研究评论* 34, 6 (2021年5月), 2689–2727. [https://doi.org/10.1093/RFS/HHAA113](https://doi.org/10.1093/RFS/HHAA113)

+   Liu et al. (2022) Yukun Liu, Aleh Tsyvinski 和 Xi Wu. 2022. 加密货币中的常见风险因素。*金融学杂志* 77, 2 (2022年4月), 1133–1177. [https://doi.org/10.1111/jofi.13119](https://doi.org/10.1111/jofi.13119)

+   Pan et al. (2024) Bo Pan, Jiaying Lu, Ke Wang, Li Zheng, Zhen Wen, Yingchaojie Feng, Minfeng Zhu 和 Wei Chen. 2024. AgentCoord: 可视化探索基于LLM的多代理协作的协调策略。(2024年4月). [https://arxiv.org/abs/2404.11943v1](https://arxiv.org/abs/2404.11943v1)

+   Sharpe (1994) William F. Sharpe. 1994. Sharpe比率。*投资组合管理杂志* 21, 1 (1994年10月), 49–58. [https://doi.org/10.3905/JPM.1994.409501](https://doi.org/10.3905/JPM.1994.409501)

+   Tan and Tao (2023) Xilong Tan 和 Yubo Tao. 2023. 基于趋势的加密货币回报预测。*经济建模* 124 (2023年7月), 106323. [https://doi.org/10.1016/J.ECONMOD.2023.106323](https://doi.org/10.1016/J.ECONMOD.2023.106323)

+   Wei et al. (2022) Jason Wei、Xuezhi Wang、Dale Schuurmans、Maarten Bosma、Brian Ichter、Fei Xia、Ed H. Chi、Quoc V. Le和Denny Zhou。2022年。《思维链提示在大型语言模型中引发推理》。*神经信息处理系统进展* 35（2022年1月）。[https://arxiv.org/abs/2201.11903v6](https://arxiv.org/abs/2201.11903v6)

+   Wu et al. (2023) 吴清云、甘根·班萨尔、张杰宇、吴怡然、李北斌、朱尔康、江丽、张晓云、张绍坤、刘佳乐、艾哈迈德·哈桑·阿瓦达拉、瑞恩·W·怀特、道格·伯格和王驰。2023年。《AutoGen：通过多代理对话支持下一代LLM应用》。2023年8月。[https://arxiv.org/abs/2308.08155v2](https://arxiv.org/abs/2308.08155v2)

+   Xie et al. (2023) 费千千、张晓、韩伟光、赖彦召、彭敏、阿莱杭德罗·洛佩兹-利拉和黄季敏。2023年。《PIXIU：一个大型语言模型、指令数据和金融评估基准》。*神经信息处理系统进展* 36（2023年6月）。[https://arxiv.org/abs/2306.05443v1](https://arxiv.org/abs/2306.05443v1)

+   Yang et al. (2024) 杨京峰、金鸿烨、唐瑞翔、韩晓天、冯齐章、姜浩铭、钟绍晨、尹冰、胡霞。2024年。《在实践中驾驭LLM的力量：关于ChatGPT及其以外的综述》。*ACM数据知识发现汇刊* 18，6（2024年4月）。[https://doi.org/10.1145/3649506/ASSET/D2038F72-9808-4957-BF9D-FB4E05F82081/ASSETS/GRAPHIC/TKDD-2023-06-0236-F02.JPG](https://doi.org/10.1145/3649506/ASSET/D2038F72-9808-4957-BF9D-FB4E05F82081/ASSETS/GRAPHIC/TKDD-2023-06-0236-F02.JPG)

+   Yin et al. (2024) 尹书康、符朝佑、赵思睿、李可、孙星、许彤和陈恩宏。2024年。《多模态大型语言模型综述》。*国家科学评论*（2024年11月）。[https://doi.org/10.1093/NSR/NWAE403](https://doi.org/10.1093/NSR/NWAE403)

+   Zhang et al. (2024) 张静怡、黄佳兴、金生、卢世建。2024年。《面向视觉任务的视觉-语言模型：综述》。*IEEE模式分析与机器智能汇刊* 46，8（2024年），5625-5644。[https://doi.org/10.1109/TPAMI.2024.3369699](https://doi.org/10.1109/TPAMI.2024.3369699)

## 附录

## 附录A 数据描述

[表6](https://arxiv.org/html/2501.00826v2#A1.T6 "Table 6 ‣ Appendix A Data Description ‣ LLM-Powered Multi-Agent System for Automated Crypto Portfolio Management")展示了数据描述、相关代理和相关文献。该表提供了我们的多模态数据概览，指定了负责分析每种数据类型的代理，以及文献团队获取的文献，以增强代理的可解释性。

表6. 数据描述、对应的代理和相关文献。

| 数据类型 | 名称 | 描述 | 代理 | 文献 |
| --- | --- | --- | --- | --- |
| 图表 ([1(a)](https://arxiv.org/html/2501.00826v2#S3.F1.sf1 "1(a) ‣ 图 1 ‣ 3.2\. 框架概述 ‣ 3\. 方法论 ‣ 基于LLM的多代理系统用于自动化加密货币投资组合管理")) | 蜡烛图 | 包含交易量条和30日移动平均线的30天蜡烛图。 | 技术 |  (Hudson和Urquhart, [2021](https://arxiv.org/html/2501.00826v2#bib.bib20)) |
| 加密因子 ([1(c)](https://arxiv.org/html/2501.00826v2#S3.F1.sf3 "1(c) ‣ 图 1 ‣ 3.2\. 框架概述 ‣ 3\. 方法论 ‣ 基于LLM的多代理系统用于自动化加密货币投资组合管理")) | MCAP | 投资组合形成周的最后一天市场资本化的对数。 | 加密 |  (刘等人, [2022](https://arxiv.org/html/2501.00826v2#bib.bib33)) |
| PRC | 投资组合形成周的最后一天价格的对数。 |  |  |
|  | MAXDPRC | 投资组合形成周的最大价格。 |  |  |
|  | MOM 1,0 | 过去一周的回报。 |  |  |
|  | MOM 2,0 | 过去两周的回报。 |  |  |
|  | MOM 3,0 | 过去三周的回报。 |  |  |
|  | MOM 4,0 | 过去四周的回报。 |  |  |
|  | MOM 4,1 | 过去一至四周的回报。 |  |  |
|  | PRCVOL | 投资组合形成周的日均成交量对数与价格的乘积。 |  |  |
|  | STDPRCVOL | 投资组合形成周价格成交量标准差的对数。 |  |  |
| 市场因子 ([1(c)](https://arxiv.org/html/2501.00826v2#S3.F1.sf3 "1(c) ‣ 图 1 ‣ 3.2\. 框架概述 ‣ 3\. 方法论 ‣ 基于LLM的多代理系统用于自动化加密货币投资组合管理")) | ATTN BTC | 比特币相关的Google搜索数据，减去过去四周的平均值，然后进行标准化处理，使其均值为零，标准差为一 | 市场 |  (刘和Tsyvinski, [2021](https://arxiv.org/html/2501.00826v2#bib.bib32)) |
| ATTN CRYPTO | 与加密货币相关的Google搜索数据，减去过去四周的平均值，然后进行标准化处理，使其均值为零，标准差为一 |  |  |
|  | UNI ADDR | 比特币钱包增长 |  |  |
|  | ACT ADDR | 活跃比特币地址增长 |  |  |
|  | TXN | 比特币交易增长 |  |  |
|  | PAY | 比特币支付增长 |  |  |
| 文本 ([1(d)](https://arxiv.org/html/2501.00826v2#S3.F1.sf4 "1(d) ‣ 图 1 ‣ 3.2\. 框架概述 ‣ 3\. 方法论 ‣ 基于LLM的多代理系统用于自动化加密货币投资组合管理")) | 新闻头条 | 来自Cointelegraph的每周新闻头条 | 新闻 |  (Anamika和Subramaniam, [2022](https://arxiv.org/html/2501.00826v2#bib.bib2)) |
