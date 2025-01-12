<!--yml

category: 未分类

日期: 2025-01-11 12:44:42

-->

# 结合微调和基于LLM的代理进行直观的智能合约审计与原因分析

> 来源：[https://arxiv.org/html/2403.16073/](https://arxiv.org/html/2403.16073/)

Wei Ma¹, Daoyuan Wu²¹, Yuqiang Sun¹, Tianwen Wang³, Shangqing Liu¹, Jian Zhang¹, Yue Xue⁴, Yang Liu¹ ¹ 南洋理工大学，新加坡

² 香港科技大学，中国香港特别行政区

³ 新加坡国立大学，新加坡

⁴ MetaTrust Labs，新加坡

ma_wei@ntu.edu.sg, daoyuan@cse.ust.hk, suny0056@e.ntu.edu.sg, tianwenw.vk@gmail.com,

liu.shangqing@ntu.edu.sg, jian_zhang@ntu.edu.sg, xueyue@metatrust.io, yangliu@ntu.edu.sg

###### 摘要

智能合约是建立在像以太坊这样的区块链上的去中心化应用程序。近期研究表明，大型语言模型（LLMs）在审计智能合约方面具有潜力，但当前的研究表明，即使是GPT-4也只能达到30%的精度（当决策和理由都正确时）。这很可能是因为现成的LLM主要是针对通用文本/代码语料库进行预训练的，并没有针对Solidity智能合约审计的特定领域进行微调。

本文提出了iAudit，一个结合微调和基于LLM的代理的通用框架，用于进行带有原因分析的直观智能合约审计。具体而言，iAudit的灵感来源于一个观察，即专家审计人员首先识别出可能出错的地方，然后对代码进行详细分析以确定原因。因此，iAudit采用了两阶段微调方法：首先对Detector模型进行微调，以做出决策，然后对Reasoner模型进行微调，以生成漏洞的原因。然而，仅仅通过微调面临着准确识别漏洞最优原因的挑战。因此，我们引入了两个基于LLM的代理，Ranker和Critic，它们根据微调后的Reasoner模型的输出，迭代地选择并辩论最合适的漏洞原因。为了评估iAudit，我们收集了一个平衡的数据集，其中包含1,734个正样本和1,810个负样本，用于微调iAudit。然后，我们将其与传统的微调模型（CodeBERT、GraphCodeBERT、CodeT5和UnixCoder）以及基于提示学习的LLM（GPT4、GPT-3.5和CodeLlama-13b/34b）进行了比较。在包含263个真实智能合约漏洞的数据集上，iAudit达到了91.21%的F1得分和91.11%的准确率。iAudit生成的漏洞原因与实际原因的匹配度约为38%。

¹¹脚注文本：通讯作者：吴道元。工作在 NTU 时进行。<svg class="ltx_picture" height="78.97" id="p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,78.97) matrix(1 0 0 -1 0 0) translate(0,2.77)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 15.75 11.81)"><foreignobject color="#000000" height="49.81" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="568.5">“在漏洞赏金项目中，真正难以教授的一个大技能就是直觉。我做的每一件事都在跟随我的直觉。什么看起来有趣，什么看起来不对。”——凯蒂·帕克斯顿-费尔，百万美元收入的黑客之一[[1](https://arxiv.org/html/2403.16073v3#bib.bib1)]。</foreignobject></g></g></svg>

## I 引言

自以太坊问世以来，智能合约作为基于区块链技术的关键应用已经崭露头角。由于其开放性、透明性和不可逆性，智能合约已成为去中心化金融应用（DeFi）的基础。然而，由于 DeFi 管理着大量的数字资产，识别并修复智能合约中的漏洞至关重要。目前，黑客在智能合约中利用的实际漏洞主要是由于逻辑缺陷[[2](https://arxiv.org/html/2403.16073v3#bib.bib2)]，这使得传统基于模式的程序分析[[3](https://arxiv.org/html/2403.16073v3#bib.bib3), [4](https://arxiv.org/html/2403.16073v3#bib.bib4), [5](https://arxiv.org/html/2403.16073v3#bib.bib5), [6](https://arxiv.org/html/2403.16073v3#bib.bib6), [7](https://arxiv.org/html/2403.16073v3#bib.bib7), [8](https://arxiv.org/html/2403.16073v3#bib.bib8), [9](https://arxiv.org/html/2403.16073v3#bib.bib9), [10](https://arxiv.org/html/2403.16073v3#bib.bib10)]的效果较差。根据 Defillama Hacks [[11](https://arxiv.org/html/2403.16073v3#bib.bib11)]，截至2024年3月，漏洞攻击已造成约76.9亿美元的损失。因此，迫切需要创新的方法来应对这些新兴威胁。

最近的研究[[12](https://arxiv.org/html/2403.16073v3#bib.bib12), [13](https://arxiv.org/html/2403.16073v3#bib.bib13), [14](https://arxiv.org/html/2403.16073v3#bib.bib14), [15](https://arxiv.org/html/2403.16073v3#bib.bib15)]表明，大型语言模型（LLM）在智能合约审计中具有潜力，特别是在检测逻辑漏洞方面表现出优越的性能[[2](https://arxiv.org/html/2403.16073v3#bib.bib2), [13](https://arxiv.org/html/2403.16073v3#bib.bib13)]。然而，最近的一项系统评估研究[[15](https://arxiv.org/html/2403.16073v3#bib.bib15)]显示，即使在为基于LLM的漏洞检测范式配备最先进的方法时，也仅能实现大约30%的准确率，这种方法是通过在检索增强生成（RAG）[[16](https://arxiv.org/html/2403.16073v3#bib.bib16)]的方式增强GPT-4，使其具备总结的漏洞知识。此结果仅在决策（即判断代码是否存在漏洞）和理由（即找出正确的漏洞类型）都正确时成立。这可以归因于现成的LLM（如GPT-4）主要是通过在通用文本/代码语料库上进行预训练的，尚未针对Solidity智能合约审计的特定领域进行微调。

微调[[17](https://arxiv.org/html/2403.16073v3#bib.bib17), [18](https://arxiv.org/html/2403.16073v3#bib.bib18)]相较于RAG[[19](https://arxiv.org/html/2403.16073v3#bib.bib19)]，可能是将Solidity特定的漏洞数据嵌入模型的一个有前景的方法，从而解决上述问题。具体来说，通过用易受攻击和不易受攻击的代码微调大型语言模型（LLM），它可以有效地判断一段新代码是否存在漏洞。根据前言中提到的一位赚取百万美元的黑客的见解，这种直觉在漏洞审计中至关重要。因此，我们提出了一种新颖的二阶段微调方法，而不是微调单一模型同时生成漏洞决策（即“是”或“否”）和漏洞原因（即漏洞类型或原因）。该方法首先微调一个检测器模型，仅进行决策，然后微调一个推理器模型，生成漏洞的原因。通过这种方式，微调后的LLM可以通过先进行直觉判断，然后执行后续分析，模仿人类黑客来识别漏洞的原因。

我们将这种“感知-再分析”微调方法实现为一个通用框架，称为iAudit²²2iAudit被部署为MetaTrust Labs的TrustLLM的审计模块；请参见[https://huggingface.co/MetaTrustSig](https://huggingface.co/MetaTrustSig)。用于直观的智能合约审计。在此实现中，iAudit允许Detector做出多个直观判断，每个判断代表一种感知。为了实现这一点，iAudit为相同的漏洞标签生成多个变体提示，以调节Detector，并为相同的漏洞原因生成多个变体提示，以调节Reasoner。虽然可以基于多数投票确定最优决策，但仅通过微调无法在推理阶段识别漏洞的最佳原因。为了解决这个新问题，我们将基于LLM的代理引入iAudit的微调范式。具体来说，我们引入了两个专门的LLM代理——Ranker和Critic代理——以迭代选择和辩论最合适的漏洞原因，基于微调后的Reasoner模型的输出。

为了获得用于训练和测试iAudit的高质量数据，我们提出利用有信誉的审计报告收集正样本，并采用我们自己的数据增强方法生成负样本。最终，我们收集了一个平衡的数据集，包含1,734个正样本，即来自263个智能合约审计报告的脆弱函数及其原因，以及1,810个负样本，即非脆弱的良性代码。然后，我们将iAudit与传统的全模型微调方法进行比较，包括CodeBERT、GraphCodeBERT、CodeT5和UnixCoder，以及基于提示学习的LLM（如GPT-4/GPT-3.5和CodeLlama-13b/34b）。我们的实验结果表明，iAudit实现了91.21%的F1得分，显著优于基于提示学习的LLM（其得分在60%以上）并且明显超越了使用相同训练数据的其他微调模型（其得分在80%以上）。此外，在与真实解释的对齐度方面，iAudit的输出明显优于其他模型，达到了37.99%的一致性率。相比之下，排名第二的GPT-4仅达到24%。

除了评估结果外，我们还进行了三项消融实验，以进一步验证iAudit的两阶段微调和多数投票策略，并衡量附加调用图信息对模型性能的影响。我们总结了以下关键发现：

+   •

    iAudit的两阶段方法在检测性能上优于集成模型，该模型同时输出标签和原因。我们还通过实验确认，当需要同时输出两种信息时，模型难以集中注意力在标签上。

+   •

    多数投票提高了检测性能和稳定性。使用多个提示词也能使模型比仅使用单个提示词时表现更好。

+   •

    调用图信息可能在某些情况下帮助模型做出更好的判断，但我们也观察到，在某些情况下，这些额外信息可能会使模型感到困惑，从而降低其性能。

路线图。本文的其余部分结构如下。我们首先在第[II](https://arxiv.org/html/2403.16073v3#S2 "II Background ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")节介绍相关背景，接着在第[III](https://arxiv.org/html/2403.16073v3#S3 "III Design of iAudit ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")节介绍iAudit的设计。然后在第[IV](https://arxiv.org/html/2403.16073v3#S4 "IV Evaluation ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")节展示我们的实验设置和结果。随后，在第[V](https://arxiv.org/html/2403.16073v3#S5 "V Related Work ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")节和第[VI](https://arxiv.org/html/2403.16073v3#S6 "VI Threats to Validity ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")节讨论相关工作和局限性。最后，第[VIII](https://arxiv.org/html/2403.16073v3#S8 "VIII Conclusion ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")节总结本文。

可用性。为了促进未来的研究和比较，我们已将推理代码和数据集公开，详见[[20](https://arxiv.org/html/2403.16073v3#bib.bib20)]。

## II 背景

### II-A 预训练模型与大语言模型

预训练模型是指已经在大规模数据集上进行初步训练的模型。这些模型可以通过最小的调整快速适应各种特定任务，避免了从头开始复杂的训练过程。目前，大多数预训练模型采用基于变换器（transformers）架构[[21](https://arxiv.org/html/2403.16073v3#bib.bib21)]。这种方法的创新之处在于，预训练模型利用大数据和精心设计的任务进行有效的特征学习，已被证明在多个领域中有效，如文本处理、图像识别和软件工程。标准的变换器结构由一个编码器和一个解码器组成，二者在结构上相似，但功能不同。根据使用的变换器结构，预训练模型可以分为基于编码器的、基于解码器的或编码器-解码器结合类型。例如，基于编码器的模型有BERT[[22](https://arxiv.org/html/2403.16073v3#bib.bib22)]和CodeBERT[[23](https://arxiv.org/html/2403.16073v3#bib.bib23)]，基于解码器的模型有GPT系列[[24](https://arxiv.org/html/2403.16073v3#bib.bib24), [25](https://arxiv.org/html/2403.16073v3#bib.bib25)]，而编码器-解码器模型则有BART[[26](https://arxiv.org/html/2403.16073v3#bib.bib26)]、T5[[27](https://arxiv.org/html/2403.16073v3#bib.bib27)]和CodeT5[[28](https://arxiv.org/html/2403.16073v3#bib.bib28)]。与一般的预训练模型相比，大型语言模型（LLMs）[[29](https://arxiv.org/html/2403.16073v3#bib.bib29), [30](https://arxiv.org/html/2403.16073v3#bib.bib30)] 在所使用的更大数据和模型规模上有显著差异。这些模型通过学习全球范围的知识库进行训练，通常达到数十亿的规模。随着模型大小、数据量和计算能力的增加，性能也得到了提升，这一点通过扩展法则（Scaling Laws）[[31](https://arxiv.org/html/2403.16073v3#bib.bib31)]得到了证明。像GPT-3.5、GPT-4和Gemini[[32](https://arxiv.org/html/2403.16073v3#bib.bib32)]这样的闭源大型语言模型通过API对外提供服务，而像Llama2[[33](https://arxiv.org/html/2403.16073v3#bib.bib33)]这样的开源模型经过微调后能够达到与闭源模型相当甚至更好的性能。

### II-B 参数高效微调

大语言模型（LLMs）具有极其庞大的参数量。完全微调一个大语言模型需要大量硬件资源，成本非常高。因此，相较于完全微调，轻量级参数微调[[34](https://arxiv.org/html/2403.16073v3#bib.bib34), [35](https://arxiv.org/html/2403.16073v3#bib.bib35)]目前是使用LLMs的主要方法。尽管LLMs可以通过上下文学习[[36](https://arxiv.org/html/2403.16073v3#bib.bib36)]而不需要任务特定的微调，但这通常需要精心准备的提示。此外，研究发现对具有较小参数的大语言模型进行部分微调，可以实现甚至超越大型模型的效果[[37](https://arxiv.org/html/2403.16073v3#bib.bib37), [38](https://arxiv.org/html/2403.16073v3#bib.bib38)]。这些微调方法与完全模型微调的不同之处在于，它们仅专注于微调附加参数，同时保持大模型的权重不变，这种方法统称为参数高效微调[[34](https://arxiv.org/html/2403.16073v3#bib.bib34), [35](https://arxiv.org/html/2403.16073v3#bib.bib35)]。这些方法通常可以分为四种类型：Adapter[[37](https://arxiv.org/html/2403.16073v3#bib.bib37)]、低秩适应（LoRA）[[39](https://arxiv.org/html/2403.16073v3#bib.bib39)]、前缀调优（prefix tuning）[[40](https://arxiv.org/html/2403.16073v3#bib.bib40)]和提示调优（prompt tuning）[[41](https://arxiv.org/html/2403.16073v3#bib.bib41)]。

Adapter[[37](https://arxiv.org/html/2403.16073v3#bib.bib37)]向模型的每一层添加一个轻量级的附加模块，用于捕捉特定于下游任务的信息。在优化过程中，只有附加模块的参数会被优化。由于Adapter的参数数量远小于模型本身的参数数量，因此大大减少了模型的总体参数数量和计算复杂度。

低秩适应（Low-Rank Adaptation，LoRA）[[39](https://arxiv.org/html/2403.16073v3#bib.bib39)]是一种参数高效的适应方法，旨在以更低的参数成本调整大语言模型（LLMs）以适应下游任务。LoRA的核心思想是将额外的低秩适应参数引入自注意力机制，通过最小化附加参数的方式，有效地调整模型以适应新任务。

前缀调优（Prefix tuning）[[40](https://arxiv.org/html/2403.16073v3#bib.bib40)]向模型的每一层添加一个“前缀”序列，作为额外的上下文输入。这种方法使得模型能够适应特定任务，同时保持在预训练过程中获得的大部分知识。与前缀调优不同，提示调优（prompt tuning）[[41](https://arxiv.org/html/2403.16073v3#bib.bib41)]将提示标记添加到输入中，并可以放置在任意位置。

总结来说，使用适配器会增加推理延迟[[39](https://arxiv.org/html/2403.16073v3#bib.bib39), [42](https://arxiv.org/html/2403.16073v3#bib.bib42)]。前缀或提示调优受结构约束，限制了新注意力模式的学习[[43](https://arxiv.org/html/2403.16073v3#bib.bib43)]。LoRA是一种高效且低成本的方法，其性能可以接近全量微调的方法[[39](https://arxiv.org/html/2403.16073v3#bib.bib39)]。

### II-C 智能合约及其漏洞

智能合约是实现去中心化金融[[44](https://arxiv.org/html/2403.16073v3#bib.bib44)]的关键技术之一，是区块链技术的应用层。根据DeFiLlama的数据[[45](https://arxiv.org/html/2403.16073v3#bib.bib45)]，截至2024年3月，前三大区块链平台（以太坊、波场和币安智能链）的总锁仓价值已达730亿美元。鉴于智能合约与经济利益的密切关系，其安全性已引起广泛关注。智能合约中的漏洞可能导致重大损失，例如重入攻击和访问控制攻击[[46](https://arxiv.org/html/2403.16073v3#bib.bib46)]。

在现实世界中，黑客采用了更为复杂的战术。目前，为了应对智能合约中的漏洞，已使用多种静态和动态检测工具[[3](https://arxiv.org/html/2403.16073v3#bib.bib3), [4](https://arxiv.org/html/2403.16073v3#bib.bib4), [5](https://arxiv.org/html/2403.16073v3#bib.bib5), [6](https://arxiv.org/html/2403.16073v3#bib.bib6), [7](https://arxiv.org/html/2403.16073v3#bib.bib7), [8](https://arxiv.org/html/2403.16073v3#bib.bib8), [9](https://arxiv.org/html/2403.16073v3#bib.bib9), [10](https://arxiv.org/html/2403.16073v3#bib.bib10)]来测试合约的安全性。不幸的是，一些复杂的漏洞很难通过这些检测工具被发现。例如，在三明治攻击中[[47](https://arxiv.org/html/2403.16073v3#bib.bib47)]，攻击者监控其他待处理的交易，并在发现一笔高价值但尚未完成的交易时，首先执行他们的交易。由于这种先发制人的行为，攻击交易会以更高的价格执行，从而允许攻击者立即将获取的超额利润卖出以获利。许多资金雄厚的项目团队还会在公开发布前邀请第三方审计其智能合约，以确保其安全性。

![请参见说明文字](img/743dbab3e3ba1972f39496ce76f3e6a3.png)

图1：iAudit概览，展示其四个角色：检测器、推理器、排序器和评论器。

## III iAudit的设计

如第[I](https://arxiv.org/html/2403.16073v3#S1 "I Introduction ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")小节中所述，iAudit采用了一种新颖的两阶段微调方法，并将其与基于LLM的智能体结合，进行直观的智能合约审计并提供合理的解释。如图[1](https://arxiv.org/html/2403.16073v3#S2.F1 "Figure 1 ‣ II-C Smart Contracts and Their Vulnerabilities ‣ II Background ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")所示，iAudit有以下四个角色：

+   •

    检测器（Detector）是实现直观智能合约审计的关键组件。通过微调一个包含漏洞与非漏洞代码的大型语言模型（LLM），检测器能够分辨一段代码是否存在漏洞，就像人类黑客感知潜在漏洞一样。

+   •

    推理器（Reasoner）从检测器（Detector）获取初步的漏洞感知信息，并根据检测器的决策进一步分析漏洞的潜在原因。通过在训练和推理过程中将检测器的输出与推理器的推理相连接，iAudit实现了两阶段微调。

+   •

    为了在推理阶段识别漏洞的最佳原因，我们进一步将基于LLM的智能体概念引入iAudit的微调范式中。具体而言，排序器（Ranker）评估每个潜在漏洞的原因，选择一个最优的解释，而批评者（Critic）则进一步评估排序器的输出，辩论并确定漏洞的最适当原因。

挑战。虽然图[1](https://arxiv.org/html/2403.16073v3#S2.F1 "Figure 1 ‣ II-C Smart Contracts and Their Vulnerabilities ‣ II Background ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")中展示的iAudit的四个角色具有直观性，但要有效地训练并协调它们，以便在合理的解释下进行智能合约审计，仍然是一个困难的任务。更具体来说，在iAudit的设计和实现过程中，我们遇到了以下四个挑战：

C1:

如何收集和推导高质量的训练数据？对于像iAudit这样的微调模型，获取高质量的训练数据始终至关重要。我们建议利用信誉良好的审计报告来收集正样本，并采用我们自己的数据增强方法来推导负样本。由于这一部分与iAudit的设计无关，我们将其展示推迟到本节的最后部分，在第[III-D](https://arxiv.org/html/2403.16073v3#S3.SS4 "III-D High-quality Training Data Collection and Enhancement ‣ III Design of iAudit ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")小节中。

C2:

如何做出有效的漏洞判断？尽管使用漏洞代码和非漏洞代码来微调模型比较直接，但在数据有限的情况下进行有效微调仍然是一个挑战。我们在第[III-A](https://arxiv.org/html/2403.16073v3#S3.SS1 "III-A Using Multi-prompt Tuning and Majority Voting for Effective Vulnerability Judgements in the Detector ‣ III Design of iAudit ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")节中通过选择使用多个提示进行微调而非单一提示，来努力解决这个问题。这种方法的优点有两点：(i) 通过增加数据量丰富了训练数据集，(ii) 减少了单一提示带来的偏差，从而提高了结果的可靠性[[48](https://arxiv.org/html/2403.16073v3#bib.bib48)]。因此，可以通过多数投票实现最佳的漏洞感知。

C3:

如何有效地将Detector的漏洞感知与Reasoner的漏洞推理连接起来？iAudit的微调方式独特，因为它采用了Detector和Reasoner模型的两阶段微调方法。因此，如何有效地连接这两个模型成为了一个在传统微调中没有遇到的新问题。我们在第[III-B](https://arxiv.org/html/2403.16073v3#S3.SS2 "III-B Connecting Detector for Reasoner’s Tuning & Inference ‣ III Design of iAudit ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")节中阐述了iAudit设计的这一部分内容。

C4:

如何从Reasoner的输出中获取最佳的漏洞原因？由于Reasoner也使用了多提示微调，因此需要在Reasoner输出的多个原因中识别出最佳的漏洞原因。我们在第[III-C](https://arxiv.org/html/2403.16073v3#S3.SS3 "III-C Ranking and Debating the Optimal Vulnerability Cause ‣ III Design of iAudit ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")节中介绍了两个基于LLM的组件——Ranker和Critic，用于迭代地选择并讨论最合适的漏洞原因。

工作流示例。总结来说，图[1](https://arxiv.org/html/2403.16073v3#S2.F1 "图 1 ‣ II-C 智能合约及其漏洞 ‣ II 背景 ‣ 结合微调和基于LLM的代理进行直观的智能合约审计及其解释")还展示了iAudit工作流的一个示例。最初，Detector使用五条不同的推理路径（提示）来感知代码漏洞。感知结果随后经过多数投票确定共识标签。根据投票结果，Reasoner根据不同的推理路径解释这一结果，生成十个答案（每个答案考虑代码位置的上下文与否）。接下来，Ranker选择置信度为9/10的Reason 1并解释这一选择。Critic质疑这一选择，并建议Ranker重新评估。考虑到Critic的反馈，Ranker重新排名十个理由，并选择置信度为10/10的Reason 3。Critic再次审查Ranker的选择并同意这一决定。循环完成，最终理由返回给用户。

### III-A 使用多提示调优和多数投票进行有效的漏洞判断

Detector是一个精细调优的专家模型，负责评估代码是否存在风险。它模拟人类在看到一段代码时的直觉判断，评估是否存在问题。我们采用了LoRA [[39](https://arxiv.org/html/2403.16073v3#bib.bib39)]对CodeLlama-13b [[49](https://arxiv.org/html/2403.16073v3#bib.bib49)]进行基于高质量数据集的指令式微调[[50](https://arxiv.org/html/2403.16073v3#bib.bib50)]。在训练过程中，对于相同的输入代码，我们将其包装在多个提示中。这些提示具有不同的指令格式，代表不同的推理路径，如图[1](https://arxiv.org/html/2403.16073v3#S2.F1 "图 1 ‣ II-C 智能合约及其漏洞 ‣ II 背景 ‣ 结合微调和基于LLM的代理进行直观的智能合约审计及其解释")所示。在Detector的推理阶段，基于每个提示的输出结果，我们采用多数投票方法来确定输入标签，并使用投票比例作为置信度分数。基于Detector的多数投票结果，Reasoner在第[III-B](https://arxiv.org/html/2403.16073v3#S3.SS2 "III-B 将Detector连接到Reasoner的调优与推理 ‣ III iAudit的设计 ‣ 结合微调和基于LLM的代理进行直观的智能合约审计及其解释")节中根据不同的推理路径生成不同的理由。

值得注意的是，在选择基础模型时，我们随机选择了16个实际的逻辑漏洞，来评估三种流行的开源模型：StarCoder、Llama2和CodeLlama。经过人工审核，我们发现StarCoder有时拒绝回应，Llama2给出了一个矛盾的回应，并且标注了两个标签。相比之下，CodeLlama提供了更稳定的回应，这使我们选择了CodeLlama作为进一步微调的基础模型。

<svg class="ltx_picture" height="229.68" id="S3.F2.1.1.pic1" overflow="visible" version="1.1" width="240"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,229.68) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 211.48)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="196.69">探测器的提示模板</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 49.85)"><foreignobject color="#000000" height="143.9" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="196.69">以下是描述分类任务的指令。[任务描述] ### 指令：[任务指令] ### 输入：[输入描述]：

“‘Solidiy

{code}

”’

### 响应：</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="12.45" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="196.69">标签是{标签名称}。</foreignobject></g></g></svg>

图2：探测器使用的提示模板。

检测器使用的提示模板如图[2](https://arxiv.org/html/2403.16073v3#S3.F2 "图 2 ‣ III-A 使用多提示微调和多数投票进行有效漏洞判断 ‣ III 设计 iAudit ‣ 结合微调与基于 LLM 的智能合约审计代理与其理由")所示。虚线以上是我们模型的输入 $x$。"{code}"是输入代码的占位符。虚线以下，“标签是{Label Name}”是我们的目标训练输出 $y$，其中“{Label Name}”是标签占位符，可以是“safe”或“vulnerable”。图[4](https://arxiv.org/html/2403.16073v3#S3.F4 "图 4 ‣ III-B 连接检测器与推理器的微调与推理 ‣ III 设计 iAudit ‣ 结合微调与基于 LLM 的智能合约审计代理与其理由")左侧表格（检测器的多个提示）详细列出了[任务描述]、[任务说明]和[输入描述]，分别作为符号a、b和c。我们使用LoRA对CodeLlama-13b进行生成性微调，如方程式[1](https://arxiv.org/html/2403.16073v3#S3.E1 "在 III-A 使用多提示微调和多数投票进行有效漏洞判断 ‣ III 设计 iAudit ‣ 结合微调与基于 LLM 的智能合约审计代理与其理由")所示，

|  | $L(\theta)=-\sum_{t=1}^{T}\log P_{\theta}(y_{t}&#124;x,y_{<t})$ |  | (1) |
| --- | --- | --- | --- |

其中 $\theta$ 表示 LoRA 可训练参数，$T$ 是输出序列长度，$P_{\theta}(y_{t}|x,y_{<t})$ 是模型参数 $\theta$ 在给定上下文 $x$ 和所有前一时间步的令牌 $y_{<t}$ 的条件下生成令牌 $y_{t}$ 的概率。在这种生成性方法中，时间步 $t$ 的输出仅依赖于前一个时间步（$<t$）。

微调后，在推理过程中，提出的输入遵循训练格式，我们需要通过关键字匹配从输出中提取标签，使用“safe”和“vulnerable”。由于我们使用了多个提示，我们得到多个标签预测，并使用多数投票来决定最终的预测标签。在多数投票中，每个推理路径为“safe”或“vulnerable”之一的可用标签投一票，分别用 $l_{0}$ 和 $l_{1}$ 表示。获得超过一半投票的标签将被宣布为获胜者。令 $L=\{l_{0},l_{1}\}$ 表示标签集，$V=\{v_{1},v_{2},\ldots,v_{m}\}$ 表示提示投票者集。每个提示投票者 $v_{i}$ 为某个标签投一票。获胜标签 $l_{i}$ 是 $l_{0}$ 和 $l_{1}$ 中满足以下条件的标签：$|\{v\in V:\text{vote}(v)=l_{i}\}|>\frac{m}{2}$。此条件表示，获胜标签 $o_{w}$ 必须获得超过一半的总票数 $m$。我们还将获胜标签的投票比例作为最终决策的置信度得分。

### III-B 连接检测器以进行 Reasoner 的调优与推理

Reasoner 是一个专家模型，负责推理和解释代码漏洞。它解释检测器的多数投票结果，生成多个替代解释。在 Reasoner 的 LoRA 微调过程中，我们的输入包括代码、其上下文、相应的标签，并且我们构建了零-shot 思维链（CoT）[[51](https://arxiv.org/html/2403.16073v3#bib.bib51)]提示，采用不同的命令格式进行训练。在推理阶段，Reasoner 根据检测器的多数投票标签输出多个解释。我们构建了两种类型的提示：第一种类型包括标签、代码信息及其函数调用关系；第二种类型仅包括标签和代码信息。在第[IV](https://arxiv.org/html/2403.16073v3#S4 "IV Evaluation ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")节中，我们将探讨是否包括函数调用关系的影响。对于每种类型，我们设计了五种不同的指令格式，总共有十条推理路径，如图[1](https://arxiv.org/html/2403.16073v3#S2.F1 "Figure 1 ‣ II-C Smart Contracts and Their Vulnerabilities ‣ II Background ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")所示。

<svg class="ltx_picture" height="365.28" id="S3.F3.1.1.pic1" overflow="visible" version="1.1" width="240"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,365.28) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 347.08)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="196.69">Reasoner 的提示模板</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 51.24)"><foreignobject color="#000000" height="278.12" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="196.69">以下是描述推理任务的指令。[任务描述] ### 指令：[任务指令] ### 输入：[输入描述]：

“‘Solidiy

{代码}

“‘

### 作为调用者：（可选）[调用者描述] “‘

{caller 信息}

“‘

### 作为被调用者：（可选）[被调用者描述] “‘

{callee 信息}

“‘

### 响应：[标签信息 + 零-shot-CoT 提示]</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="196.69">{目标推理}</foreignobject></g></g></svg>

图 3：Reasoner 使用的提示模板。

推理器使用的提示模板如图 [3](https://arxiv.org/html/2403.16073v3#S3.F3 "图 3 ‣ III-B 连接检测器用于推理器的调优与推断 ‣ III iAudit 设计 ‣ 结合微调与基于 LLM 的智能合约审计代理及其理由") 所示。 “code”、 “caller info”、 “callee info”和“Target Reason”分别是输入代码、调用者上下文、被调用者上下文和目标输出的占位符。图 [4](https://arxiv.org/html/2403.16073v3#S3.F4 "图 4 ‣ III-B 连接检测器用于推理器的调优与推断 ‣ III iAudit 设计 ‣ 结合微调与基于 LLM 的智能合约审计代理及其理由") 中的右侧表格（推理器的多重提示）详细介绍了第一种提示类型，包括调用上下文，其中 [Task Description] 用 a 表示， [Task Instruction] 用 b 表示， [Input Description] 用 c 表示， [Caller Description] 和 [Callee Description] 用 d 表示， [Label Information + Zero-shot-CoT Tip] 用 e 表示。对于第二种提示类型，调用上下文被省略。推理器采用与检测器相同的微调方法，如公式 [1](https://arxiv.org/html/2403.16073v3#S3.E1 "在 III-A 使用多提示微调和多数投票进行有效漏洞判断 ‣ III iAudit 设计 ‣ 结合微调与基于 LLM 的智能合约审计代理及其理由") 所示。在推断过程中，提出的输入提示遵循训练格式，推理器生成十个答案来解释检测器的评估。

![参见标题](img/fe638efcaa6cf1da916fefd510976afc.png)

图 4：检测器和推理器的详细多重提示。

### III-C 排序与辩论最优漏洞原因

Ranker 和 Critic 是两个基于 LLM 的代理，它们合作从推理器为给定代码功能返回的多个解释中选择最合适的漏洞原因。Ranker 执行两个操作：“rank”和“merge”。“rank”涉及从提供的解释中选择最佳解释，而“merge”涉及整合多个选择的解释。我们为 Ranker 定义了 10 个约束，用于选择最佳解释。Critic 根据代码功能评估 Ranker 的答案，提供三个后续操作指令：“agree”、“rerank”和“merge”。“agree”表示当前答案合理，可以返回给用户；“rerank”表示 Ranker 需要根据 Critic 的反馈和之前的答案重新选择；“merge”则表示必须整合所提供的最佳理由。

更具体地说，Ranker 在提示中采用以下 10 个约束作为选择标准：

1.  1.

    如果某个理由描述了在提供的输入中不存在的代码，那么它就是无效的。

1.  2.

    如果某个理由与代码无关，那么这个理由是无效的。

1.  3.

    如果这个理由与事实不符，那么这个理由就是不合理的。

1.  4.

    如果一个理由与决策无关，则该理由无效。

1.  5.

    如果一个理由假设了任何未提供的信息，则该理由无效。

1.  6.

    如果代码是安全的且一个理由支持该决策，请检查代码是否存在其他潜在漏洞。如果代码存在其他潜在漏洞，则该理由无效。

1.  7.

    选择的理由应该与决策最相关。

1.  8.

    选择的理由必须是最合理和准确的。

1.  9.

    选择的理由必须是事实性的、逻辑的且具有说服力。

1.  10.

    不要对给定的代码做任何假设。

Ranker和Critic都是基于Mixtral 8x7B-Instruct [[52](https://arxiv.org/html/2403.16073v3#bib.bib52)]模型实现的LLM代理，其能力接近于更大规模的LLM [[52](https://arxiv.org/html/2403.16073v3#bib.bib52)，[53](https://arxiv.org/html/2403.16073v3#bib.bib53)，[54](https://arxiv.org/html/2403.16073v3#bib.bib54)]。此外，我们观察到，专家混合模型（MoE）[[52](https://arxiv.org/html/2403.16073v3#bib.bib52)]比其他模型更有效地输出预定格式的数据，这使得我们更容易处理输出结果。

### III-D 高质量训练数据的收集与增强

训练数据的质量对于微调LLM至关重要。为了收集正面样本，即有风险的漏洞代码，我们可以使用来自知名行业公司的审计报告，如Trail of Bits、Code4rena和Immunefi。具体而言，我们爬取并解析了来自263个智能合约审计报告中的1,734个漏洞函数及其原因，这些报告来自一个叫做Solodit的流行审计网站 [[55](https://arxiv.org/html/2403.16073v3#bib.bib55)]。

然而，为了训练我们的模型，我们还需要非易受攻击的良性代码（即负样本），但在审计报告中缺少这种类型的数据。因此，我们提出了自己的数据增强方法来获取高质量的负样本。具体而言，我们采用了LLM4Vuln中描述的基于GPT-4的方法[[15](https://arxiv.org/html/2403.16073v3#bib.bib15)]，从Code4rena上的漏洞报告中提取漏洞知识。这包括漏洞函数的功能描述以及漏洞发生的代码级原因。然后，我们基于功能描述将这些原始漏洞知识使用亲和传播（Affinity Propagation）[[56](https://arxiv.org/html/2403.16073v3#bib.bib56)]进行聚类，如[[57](https://arxiv.org/html/2403.16073v3#bib.bib57)]所述，并使用GPT-4为每个组总结功能描述。通过组功能、单个功能和漏洞疏漏的层次信息，我们在GPTScan[[13](https://arxiv.org/html/2403.16073v3#bib.bib13)]中采用层次化的基于GPT的匹配方法（即先匹配组，再匹配功能和疏漏），以获取测试代码的标签信息。如果没有匹配到漏洞信息，则该函数被标记为负样本。所有使用的提示均来自LLM4Vuln和GPTScan。

最终，我们收集了一个平衡的数据集，其中包含1,734个正样本和1,810个负样本。在该数据集中，漏洞函数的中位数代码行数为49.5行，复杂度为18.5，而安全函数的中位数代码行数为35.5行，复杂度为13.5。两类函数的复杂度分布在某种程度上相似，但存在轻微差异，Kullback-Leibler（KL）散度值为0.1。该数据集被分为训练集、验证集和测试集，分别包含2,268条、567条和709条数据。在训练过程中，我们使用训练集和验证集。在测试过程中，我们使用测试集。

![参见说明](img/dc999e8e94d102b6976b4d9a4c175d0b.png)

图 5：基于GPT-3.5的漏洞解释扩展的数据增强。

在收集标注数据和未标注数据后，我们还获得了与漏洞相关的相应解释。然而，这些漏洞解释的质量差异较大。此外，一些数据包含外部链接，这可能导致模型产生幻觉并输出不存在的链接。为了提高数据集中漏洞背后原因的可解释性，我们使用了GPT-3.5来增强现有的解释，扩展漏洞的解释、概念验证（PoC）和推荐的修复方法。如图[5](https://arxiv.org/html/2403.16073v3#S3.F5 "图 5 ‣ III-D 高质量训练数据收集与增强 ‣ III iAudit设计 ‣ 结合微调和基于LLM的智能合约审计代理与解释")所示，左侧部分展示了漏洞的原始解释，这些解释简短且缺乏细节。因此，我们将这些解释作为提示，并结合代码上下文，指导GPT-3.5生成更详细的描述，包括PoC和缓解建议。

请注意，我们之所以选择GPT-3.5，主要是因为它在成本和效果之间达到了良好的平衡——比GPT-4便宜得多，但在我们手动比较时，在这个特定任务上质量相当。我们还在此过程中对提示进行了约束，以确保增强的解释与实际漏洞一致。

## IV 评估

### IV-A 实验设置

在我们的研究中，我们仔细选择了一系列基准模型，并将其分为两组：零样本学习的LLMs和基于微调的预训练代码模型，以确保全面且可靠的比较分析。对于零样本学习的LLMs，我们选择了CodeLlama-13b-Instruct、CodeLlama-34b-Instruct[[49](https://arxiv.org/html/2403.16073v3#bib.bib49)]、GPT-3.5和GPT-4作为基准，代表了当前的最先进技术。此外，我们还选择了CodeBERT[[23](https://arxiv.org/html/2403.16073v3#bib.bib23)]、GraphCodeBERT[[58](https://arxiv.org/html/2403.16073v3#bib.bib58)]、CodeT5[[28](https://arxiv.org/html/2403.16073v3#bib.bib28)]、UnixCoder[[59](https://arxiv.org/html/2403.16073v3#bib.bib59)]和CodeLlama-13b[[49](https://arxiv.org/html/2403.16073v3#bib.bib49)]来训练分类器。在这些模型中，CodeBERT、GraphCodeBERT、CodeT5和UnixCoder经历了完整的模型微调过程，以适应特定的代码分类任务。特别地，CodeLlama-13b采用LoRA进行轻量级调优，并使用最后的标记表示进行分类。需要注意的是，我们的方法不同；iAudit的检测器通过生成标签名称作为任务输出实现分类。

### IV-B 研究问题（RQ）

由于我们提出的方法包含两个核心功能：漏洞检测和原因解释，我们设计了一系列实验来评估和展示这两项任务的性能和有效性。这些实验旨在回答以下研究问题（RQs）：

#### RQ1 - 性能比较

iAudit在漏洞检测方面的表现与其他模型相比如何？这个问题旨在了解检测器在漏洞检测中的有效性与其他现有模型的对比。重点是通过准确性、精确度、召回率和F1得分等度量指标进行比较分析，以评估和对比性能。

#### RQ2 - 解释一致性

iAudit的推理器生成的解释在多大程度上与实际原因一致？RQ2关注推理器为检测器的决策提供的解释质量。它质疑iAudit给出的理由是否与漏洞背后的实际原因相符，强调模型的可解释性和可信度。

#### RQ3 - 两阶段方法 vs. 集成模型

iAudit与一个同时进行检测和推理的集成模型相比如何？我们的方法基于生成模型，分别训练了两个模型来生成标签和原因。另一种方法使用单一模型同时生成原因和标签。这个问题探讨了将检测器和推理器组件整合为一个模型的有效性和影响。

#### RQ4 - 多数投票的有效性

多数投票能提高检测器的有效性吗？RQ4调查通过采用多数投票机制是否能够提高检测器的有效性。多数投票是一种基于多个模型的多数输出作出最终决策的技术，它可能提高方法的鲁棒性和准确性。

#### RQ5 - 附加信息的影响

调用图展示了代码与项目中其他组件的交互，预计这对我们的任务有利。我们解决了两个具体的研究子问题：

+   •

    RQ5.1\. 调用图能否提高检测器性能？

+   •

    RQ5.2\. 调用图在我们生成解释的过程中，特别是在推理器-排序器-批评器管道中的作用是什么？

除了上述研究问题，我们还使用我们的模型对Code4rena上的两个赏金项目（目前匿名）进行了审计。我们邀请了审计专家验证我们的发现。最终，我们发现了6个关键漏洞，这些漏洞得到了项目团队或审计专家的认可。特别是，一个漏洞没有被任何工具发现，被标记为一个重要发现。这证明了iAudit在现实世界中的价值。由于页面限制，相关案例研究已包含在附录材料中，供感兴趣的读者参考。

### IV-C RQ1 - 性能比较

首先，我们将 iAudit 与基于零-shot 学习的 LLM 进行了比较，如表[I](https://arxiv.org/html/2403.16073v3#S4.T1 "TABLE I ‣ IV-C RQ1 - Performance Comparison ‣ IV Evaluation ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")所示。我们的方法在推理阶段也使用了零-shot 方法。我们考虑了两个专有模型（GPT-4 和 GPT-3.5）和三个开源模型（CodeLlama-13b 和 CodeLlama-34b）。对于开源模型，我们严格遵循了它们的提示格式。Huggingface Transformer [[60](https://arxiv.org/html/2403.16073v3#bib.bib60)] 已经将这些提示格式集成到其框架中。格式转换通过调用 apply_chat_template 完成。CodeLlama 需要添加 [INST] 和 [/INST] 以及特殊标签 <<SYS>> 和 <</SYS>>。如表[I](https://arxiv.org/html/2403.16073v3#S4.T1 "TABLE I ‣ IV-C RQ1 - Performance Comparison ‣ IV Evaluation ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")所示，经过微调后，我们提出的策略在零-shot 场景下在 F1、准确率和精确度方面显著超过了基准模型，分别达到了 $0.9121$、$0.8934$ 和 $0.9111$ 的高分。然而，当检查召回率时，我们注意到基准模型的表现都非常优秀。特别地，GPT-4 和 GPT-3.5 达到了召回率为 $1$ 的得分。我们检查了混淆矩阵，发现所有测试数据都由漏洞标注。对于 GPT-4 和 GPT-3.5，我们采用了由我们的行业合作伙伴 MetaTrust Labs 提供的提示，MetaTrust Labs 是一家 Web3 安全公司。

表 I：iAudit 的检测器与零-shot LLM 的性能比较。

|  | F1 | 召回率 | 精确度 | 准确率 |
| --- | --- | --- | --- | --- |
| GPT-4 | 0.6809 | 1 | 0.5162 | 0.5162 |
| GPT-3.5 | 0.6809 | 1 | 0.5162 | 0.5162 |
| CodeLlama-13b | 0.6767 | 0.9781 | 0.5173 | 0.5176 |
| CodeLlama-34b | 0.6725 | 0.9454 | 0.5219 | 0.5247 |
| iAudit | 0.9121 | 0.8934 | 0.9316 | 0.9111 |

表 II：iAudit 的检测器与其他微调模型的性能比较。

|  | F1 | 召回率 | 精确度 | 准确率 |
| --- | --- | --- | --- | --- |
| CodeBERT | 0.8221 | 0.7322 | 0.9371 | 0.8364 |
| GraphCodeBERT | 0.8841 | 0.8333 | 0.9414 | 0.8872 |
| CodeT5 | 0.8481 | 0.7705 | 0.9431 | 0.8575 |
| UnixCoder | 0.8791 | 0.8443 | 0.9169 | 0.8801 |
| CodeLlama-13b-class | 0.8936 | 0.8716 | 0.9167 | 0.8928 |
| iAudit | 0.9121 | 0.8934 | 0.9316 | 0.9111 |

其次，我们将Detector与微调后的模型在检测漏洞方面进行了比较，使用F1、召回率、精度和准确率作为评估指标，如表[II](https://arxiv.org/html/2403.16073v3#S4.T2 "TABLE II ‣ IV-C RQ1 - Performance Comparison ‣ IV Evaluation ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")所示。我们将我们的方法与CodeBERT、GraphCodeBERT、CodeT5、UnixCoder和CodeLlama-13b-class进行了比较。CodeBERT、GraphCodeBERT、CodeT5和UnixCoder都进行了完整的模型微调。这些传统的预训练模型将输入序列的第一个标记作为分类器的特征输入。CodeBERT基于变压器编码器。GraphCodeBERT与CodeBERT具有相同的架构，但在数据依赖关系上进行了额外的预训练。CodeT5利用变压器编码器和解码器，采用类似于T5的架构[[27](https://arxiv.org/html/2403.16073v3#bib.bib27)]。UnixCoder统一了编码器和解码器架构，通过前缀适配器使用掩蔽注意矩阵来控制模型行为。CodeLlama-13b-class基于LoRA进行分类。我们使用PEFT框架对CodeLlama-13b-class进行了LoRA分类微调。CodeLlama-13b-class使用输入序列最后一个标记的表示作为分类器的特征输入。

如表[II](https://arxiv.org/html/2403.16073v3#S4.T2 "TABLE II ‣ IV-C RQ1 - Performance Comparison ‣ IV Evaluation ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")所示，iAudit在所有方法中，在F1、召回率和准确率方面分别取得了最高分，分别为$0.9121$、$0.8934$和$0.9111$。CodeLlama-13b-class在漏洞检测率上仅次于我们的方法，且其性能较为接近。GraphCodeBERT和UnixCoder的表现不及CodeLlama-13b-class。虽然CodeT5在精度上达到了$0.9431$的最高值，但其其他指标低于GraphCodeBERT和UnixCoder。CodeBERT的表现最差。此外，这些模型的准确率得分相对较高（均超过$0.91$），表明预测的许多风险漏洞确实是有风险的。

在精度方面，iAudit的假阳性比CodeT5、GraphCodeBERT和CodeBERT更多。这主要是因为iAudit使用了比这些模型更长的上下文长度（16k），而这些模型的512长度上下文截断导致漏洞样本与安全样本在长度上的对齐更为紧密，从而影响了表面上的代码复杂度。然而，我们的数据集显示，漏洞代码相对于安全代码更为复杂。这使得iAudit在输出10个独特的假阳性时，涉及了更为复杂的代码。

<svg class="ltx_picture" height="74.67" id="S4.SS3.p5.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,74.67) matrix(1 0 0 -1 0 0) translate(0,2.77)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 15.75 11.81)"><foreignobject color="#000000" height="45.51" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="568.5">Answer for RQ1: The performance of iAudit’s Detector exceeds that of traditional full-model fine-tuning, LoRA fine-tuning in a classification manner, and LLMs based on in-context learning. The performance of fine-tuned models is also better than that of zero-shot learning.</foreignobject></g></g></svg>

### IV-D RQ2 - 解释对齐

![参考标题](img/75cbb088a38bb082d85b0e5bb56a54e3.png)

图6：比较与真实原因的对齐情况。

为了衡量Reasoner在解释漏洞方面的有效性，我们比较了我们生成的解释与根本原因之间的一致性。鉴于LLM在解释文本意义方面的出色表现，我们使用GPT-4验证我们生成的解释是否与根本原因一致。为了进行一致性评估，我们采用了Y. Sun等人提出的自动化标注提示[[15](https://arxiv.org/html/2403.16073v3#bib.bib15)]。我们的一致性测试结果如图[6](https://arxiv.org/html/2403.16073v3#S4.F6 "Figure 6 ‣ IV-D RQ2 - Explanation Alignment ‣ IV Evaluation ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")所示，其中y轴表示我们在测试集中与根本原因匹配的解释百分比。我们的方法显著优于基线方法，达到了37.99%的一致性率，而没有任何基线方法超过25%。在这些基线方法中，GPT-4以24.13%的一致性表现第二。此外，结果还表明CodeLlama-13b表现最弱。

<svg class="ltx_picture" height="71.97" id="S4.SS4.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,71.97) matrix(1 0 0 -1 0 0) translate(0,2.77)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 15.75 11.81)"><foreignobject color="#000000" height="42.82" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="568.5">Answer for RQ2: The rationality of Reasoner’s output is clearly superior to that of other models. On the test set, its consistency with real reasons reaches 37.99%, which is over 10% higher than the second-ranked GPT-4.</foreignobject></g></g></svg>

### IV-E RQ3 - 两阶段方法 vs. 集成模型

![参考说明](img/042e3545a603e569154ad4bffa1a3611.png)

图7：将iAudit与同时进行决策和漏洞解释的集成模型进行比较。

我们的研究方法涉及漏洞检测和解释，分为两个阶段进行。我们基于生成方法训练了两个模型，即检测器（Detector）和推理器（Reasoner），在各自的高质量数据集上执行这些任务。一个问题是，这两个任务是否可以合并并在一个模型中同时训练。对此，我们开发了一个集成模型，该模型生成漏洞的标签和解释，并将其与我们的两阶段方法进行比较。集成模型使用与Reasoner类似的提示，并增加了输出标签的要求。我们探讨了两种集成训练方法：1）先生成标签，然后解释原因；2）先解释原因，然后生成标签。

如图[7](https://arxiv.org/html/2403.16073v3#S4.F7 "Figure 7 ‣ IV-E RQ3 - Two-stage Approach vs. An Integration Model ‣ IV Evaluation ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")所示，结果表明，我们的逐步方法在四个关键性能指标上优于集成模型。尽管这三种方法的精确度相似，但在其他指标上的差异显著。通过分析这些结果，我们发现集成方法对负样本的准确率较高，但对正样本的检测率较低（即召回率较低）。这可能归因于生成损失优化，其中输出序列较长，导致与标签相关的损失占总损失的比例较小，从而防止模型充分关注标签。为了验证这一假设，我们在集成训练过程中加入了仅包含标签生成的数据，指导模型更多地关注标签。在评估阶段，我们仍要求模型同时输出标签和解释。通过这种混合训练方法，我们观察到模型在漏洞检测性能上有了显著改善，F1分数为$0.8433$，召回率为$0.8164$，精确度为$0.8723$，准确率为$0.8434$。

<svg class="ltx_picture" height="91.27" id="S4.SS5.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,91.27) matrix(1 0 0 -1 0 0) translate(0,2.77)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 15.75 11.81)"><foreignobject color="#000000" height="62.11" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="568.5">Answer for RQ3: iAudit achieved better detection performance than the integration model that outputs labels and reasons simultaneously. We confirmed that the model struggles to focus on the labels when required to output both types of information, as evidenced by our inclusion of label-only data in the verification process.</foreignobject></g></g></svg>

表 III: 多数投票与单一提示。

|  | F1 | 召回率 | 精确度 | 准确率 |
| --- | --- | --- | --- | --- |
| 单一提示 | 0.8278 | 0.8005 | 0.8567 | 0.8279 |
| Prompt-1 | 0.8988 | 0.8852 | 0.9127 | 0.8970 |
| Prompt-2 | 0.9027 | 0.8743 | 0.9329 | 0.9027 |
| Prompt-3 | 0.9063 | 0.8852 | 0.9284 | 0.9055 |
| Prompt-4 | 0.9098 | 0.8962 | 0.9239 | 0.9083 |
| Prompt-5 | 0.9096 | 0.8934 | 0.9263 | 0.9083 |
| iAudit | 0.9121 | 0.8934 | 0.9316 | 0.9111 |

### IV-F RQ4 - 多数投票的有效性

我们的研究探索了一种使用多个提示和投票机制的方法，让检测器决定最终标签。该方法旨在提高模型的精度和可信度。在评估过程中，我们继续使用了F1分数、召回率、精准度和准确率等指标。我们分别计算了每个提示的这些指标，以便进行对比分析，如表[III](https://arxiv.org/html/2403.16073v3#S4.T3 "TABLE III ‣ IV-E RQ3 - Two-stage Approach vs. An Integration Model ‣ IV Evaluation ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")所示。需要注意的是，第一行的Single-prompt表示我们仅使用了一种提示格式来训练检测器。Prompt-1、Prompt-2、Prompt-3、Prompt-4和Prompt-5代表了多提示训练后每个提示的结果。最后一行显示了通过多数投票后的结果，表明多数投票能够提高iAudit的整体性能，F1分数和准确率均为最高。同时，除了Single-prompt之外，我们注意到多个提示之间的性能差异极小。Single-prompt的表现明显逊色于其他提示。与仅使用单一提示进行训练相比，使用多个提示可以提高模型性能。

此外，我们将测试集分为两组，分别为“正确预测”组和“错误预测”组，依据预测结果是否正确，并分析这两组中的置信度得分分布。我们发现，在错误预测组中，置信度得分在$0.6$到$0.8$区间的比例显著高于正确预测组（分别为11%对2%，10%对3%），如图[8](https://arxiv.org/html/2403.16073v3#S4.F8 "Figure 8 ‣ IV-F RQ4 - Effectiveness of Majority Voting ‣ IV Evaluation ‣ Combining Fine-tuning and LLM-based Agents for Intuitive Smart Contract Auditing with Justifications")所示。置信度得分在一定程度上可以反映预测结果的可靠性。当置信度得分较低时，预测结果的可信度较低。

<svg class="ltx_picture" height="74.67" id="S4.SS6.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,74.67) matrix(1 0 0 -1 0 0) translate(0,2.77)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 15.75 11.81)"><foreignobject color="#000000" height="45.51" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="568.5">RQ4的回答：多数投票增强了检测性能和稳定性。此外，使用多个提示使得模型的表现优于仅使用单一提示时的表现，更加可靠。</foreignobject></g></g></svg>![参考图注](img/fc68295e03daabb661b80c438930635a.png)

图 8：正确预测和错误预测的投票分数分布。

### IV-G RQ5 - 附加信息的影响

我们探索了是否引入额外的调用图信息可以提升模型的表现。我们将函数调用关系作为上下文信息添加到提示中。

#### RQ5.1

通过表[IV](https://arxiv.org/html/2403.16073v3#S4.T4 "表 IV ‣ RQ5.2 ‣ IV-G RQ5 - 附加信息的影响 ‣ IV 评估 ‣ 结合微调和基于LLM的智能合约审计代理与解释")中的对比实验，我们发现这类调用上下文信息并没有改善模型的整体表现。在第二行，Call，我们使用了包含调用信息的提示，并采用多数投票决定预测结果。对于第三行，Call-OutCall，我们使用了包含和不包含调用信息的提示，并同样使用了多数投票。与iAudit的探测器相比，它们的精确度、准确率和F1值较低，召回率几乎相同。

#### RQ5.2

图[9](https://arxiv.org/html/2403.16073v3#S4.F9 "图 9 ‣ RQ5.2 ‣ IV-G RQ5 - 附加信息的影响 ‣ IV 评估 ‣ 结合微调和基于LLM的智能合约审计代理与解释")展示了Ranker-Critic选择的理由分布。我们可以看到，大多数（65%）选择的理由来自包含调用信息的提示，而仍有较高比例（35%）的选择理由来自不包含调用信息的提示。

虽然函数调用关系提供了更多的信息，但这些信息并不总是能帮助模型更好地完成当前任务。在某些情况下，这些信息可能会造成干扰，使得模型难以识别关键信息，从而导致更多的假阳性，影响性能。此外，并非所有的函数调用关系都具有实际价值。如果这些附加信息与模型试图解决的问题关系不大，它们可能无法提升模型的性能。我们的研究表明，仅仅添加函数调用信息并不能直接促进模型在漏洞检测中的有效性。在漏洞检测领域，如何构建有效的上下文信息仍然是一个具有挑战性且值得研究的问题。

<svg class="ltx_picture" height="74.67" id="S4.SS7.SSS0.Px2.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,74.67) matrix(1 0 0 -1 0 0) translate(0,2.77)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 15.75 11.81)"><foreignobject color="#000000" height="45.51" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="568.5">Answer for RQ5: Additional call graph information may enable the model to make better judgments in some cases. However, we also observed situations where this additional information could potentially confuse the model, thereby reducing its performance.</foreignobject></g></g></svg>

表 IV：附加信息有无的影响。

|  | F1 | 召回率 | 精确度 | 准确率 |
| --- | --- | --- | --- | --- |
| Call | 0.9011 | 0.8962 | 0.9061 | 0.8984 |
| Call-OutCall | 0.9083 | 0.8934 | 0.9237 | 0.9069 |

| iAudit | 0.9121 | 0.8934 | 0.9316 | 0.9111 | ![参考图注](img/c0c39f5b3bee44dcaf194abd1b8c5111.png)

图 9：Ranker-Critic的最终理由分布。

## V 相关工作

漏洞检测一直是软件生态系统健康和可持续发展的关键问题，尤其是在区块链和智能合约中。传统的漏洞检测方法，如基于预定义静态分析规则的检测方法[[3](https://arxiv.org/html/2403.16073v3#bib.bib3)]，通常缺乏强大的泛化能力，并且很难适应新类型的漏洞。此外，一些与逻辑相关的漏洞[[2](https://arxiv.org/html/2403.16073v3#bib.bib2)]也难以通过静态分析规则进行封装。为了解决这个问题，研究人员已经采用了基于深度学习的方法。例如，Zhuang等人[[61](https://arxiv.org/html/2403.16073v3#bib.bib61)]使用图神经网络来检测智能合约中的漏洞。Liu等人[[62](https://arxiv.org/html/2403.16073v3#bib.bib62)]结合了可解释的图特征和专家模式，以实现更好的结果和可解释的权重。Wu等人[[63](https://arxiv.org/html/2403.16073v3#bib.bib63)]利用预训练技术和关键数据流图来检测智能合约中的漏洞。

随着大语言模型的出现，研究人员不仅在漏洞检测中使用传统的深度学习模型，还利用大语言模型（LLMs）。例如，Ullah等人[[64](https://arxiv.org/html/2403.16073v3#bib.bib64)]评估了大语言模型在漏洞检测任务中的表现，发现它们可能表现不佳。Fu等人[[65](https://arxiv.org/html/2403.16073v3#bib.bib65)]进一步分析了大语言模型在漏洞检测中的差距。Thapa等人[[66](https://arxiv.org/html/2403.16073v3#bib.bib66)]利用大语言模型进行软件漏洞检测，David等人[[12](https://arxiv.org/html/2403.16073v3#bib.bib12)]则使用大语言模型进行智能合约漏洞任务。Alqarni等人[[67](https://arxiv.org/html/2403.16073v3#bib.bib67)]对BERT模型[[68](https://arxiv.org/html/2403.16073v3#bib.bib68)]进行了微调，用于源代码漏洞检测。Sun等人[[15](https://arxiv.org/html/2403.16073v3#bib.bib15)]提出了一个统一的评估框架LLM4Vuln，以增强大语言模型的漏洞检测能力。一些研究还将大语言模型与传统的程序分析方法结合。例如，Sun等人[[13](https://arxiv.org/html/2403.16073v3#bib.bib13)]提出了GPTScan，用于智能合约，通过静态程序分析减少大语言模型的假阳性。Li等人[[69](https://arxiv.org/html/2403.16073v3#bib.bib69)]提出了LLift，用于将大语言模型与静态分析工具集成。SmartInv[[70](https://arxiv.org/html/2403.16073v3#bib.bib70)]和PropertyGPT[[71](https://arxiv.org/html/2403.16073v3#bib.bib71)]进一步将大语言模型与形式验证方法结合，旨在不仅检测漏洞，还要证明一段代码是安全的。

然而，所有这些研究并没有将领域特定的知识融入模型本身，仅仅关注来自预训练数据集的知识或脆弱的代码片段，这不能有效地检测逻辑错误。

## VI 有效性威胁

在数据收集过程中，存在数据偏差的风险，这可能会导致在这些数据上训练和测试的模型无法准确地进行泛化。此外，数据标注的精确度对模型的表现有着显著影响。为了缓解这些问题，我们从真实且公开的审计报告中收集了经过验证的数据，并利用了最新的工具，如GPTScan [[13](https://arxiv.org/html/2403.16073v3#bib.bib13)]和LLM4Vuln [[15](https://arxiv.org/html/2403.16073v3#bib.bib15)]，来辅助数据的清理和标注。需要注意的是，数据中的外部链接可能会导致LLM生成不准确的信息；因此，我们进行了数据清理，移除了这些链接。考虑到LLM对输入数据的敏感性，我们对代码数据进行了标准化处理，包括去除不必要的空格而不改变代码语义，以增强模型的鲁棒性和可靠性。为了最大化GPT-3.5和GPT-4的零-shot学习表现，我们采用并优化了来自我们合作伙伴MetaTrust Labs的提示语。这些提示语已被集成到他们的工作流程中。对于开源模型，我们与审计专家合作，调整了这些模型的提示语。

过拟合是模型训练中的常见问题，我们通过实施早停策略来解决这一问题。不同模型的选择可能会影响ranker-critic架构的效果。我们测试了多个前沿的开源模型，包括MoE [[52](https://arxiv.org/html/2403.16073v3#bib.bib52)]、CodeLlama-70b [[49](https://arxiv.org/html/2403.16073v3#bib.bib49)]、Llama2-70b [[33](https://arxiv.org/html/2403.16073v3#bib.bib33)]以及最近推出的Gemma [[72](https://arxiv.org/html/2403.16073v3#bib.bib72)]，并比较了它们在推理基准测试中的表现。根据模型输出格式的严格性和操作速度等因素，我们选择了MoE [[52](https://arxiv.org/html/2403.16073v3#bib.bib52)]。我们的研究还表明，MoE所选择的理由与真实理由之间的一致性达到了约38%。为了控制成本，我们将ranker-critic循环中的最大迭代次数限制为五次，并采用了四位小数的精度处理。

## VII 限制

尽管我们的方法在试验中表现优异，其主要优势在于检测智能合约中的逻辑漏洞，根据近期的研究，逻辑漏洞占可利用漏洞的80%以上[[2](https://arxiv.org/html/2403.16073v3#bib.bib2)]。因此，尽管iAudit的训练数据包括了重入、溢出和下溢漏洞的实例，但处理这些漏洞并不是iAudit的主要使用场景。事实上，这些传统的合约漏洞理论上更适合通过程序分析方法来检测。也就是说，基于AI的方法和基于编程语言（PL）的方法各有其独特的优势，而由于iAudit完全基于AI，本文仅与其他基于AI的方法进行了比较。此外，尽管我们通过投票和代理人尽力减轻大型语言模型（LLMs）中的幻觉现象，但幻觉仍然可能发生。最后，由于我们的工具依赖于LLMs，因此存在一定的硬件要求。虽然这些模型可以被压缩以在较低性能的GPU上运行，但这种压缩可能会导致模型性能下降。

## VIII 结论

在本文中，我们提出了iAudit，这是第一个结合了微调和基于LLM的代理人来检测漏洞并解释结果的智能合约审计框架。我们采用了基于多重提示的策略，并应用基于LoRA的微调来训练检测器和推理器。前者基于多数投票机制生成结果，而后者则根据不同的推理路径提供多个替代解释。此外，我们引入了两个LLM代理人——Ranker和Critic，它们合作选择最合适的解释。我们的方法在零-shot场景下相比于零-shot LLM学习和传统的全模型微调方法展示了更优的性能。我们研究了多数投票策略带来的性能提升，并比较了不同的LoRA训练方法，提供了我们选择的合理性。我们还探索了额外调用上下文如何影响模型的性能。未来的工作将集中于增强模型的稳定性及其与人类偏好的对齐。

## 致谢

我们感谢所有审稿人提供的详细和建设性的意见。我们还要感谢MetaTrust Labs的所有同事在部署TrustLLM的iAudit模型方面的帮助。本研究/项目得到了新加坡国家研究基金会和网络安全局在其国家网络安全研发计划（NCRP25-P04-TAICeN）下的资助、新加坡国家研究基金会以及DSO国家实验室在AI新加坡计划（AISG奖项编号：AISG2-GC-2023-008）下的资助，以及NRF研究员资助（NRF-NRFI06-2020-0001）。本材料中表达的任何观点、发现、结论或建议均为作者个人意见，并不代表新加坡国家研究基金会和新加坡网络安全局的观点。Daoyuan Wu还部分得到了香港科技大学资助的支持。

## 参考文献

+   [1] L. Whitney, “Google在2023年向安全研究人员支付了1000万美元的漏洞赏金，” https://www.zdnet.com/article/google-paid-out-10-million-in-bug-bounties-to-security-researchers-in-2023/，2024年3月。

+   [2] Z. Zhang, B. Zhang, W. Xu, 和 Z. Lin, “揭开智能合约中可利用漏洞的面纱，”发表于 *2023 IEEE/ACM 第45届国际软件工程大会（ICSE）*，2023年5月，第615–627页。

+   [3] J. Feist, G. Grieco, 和 A. Groce, “Slither: 一个用于智能合约的静态分析框架，”发表于 *2019 IEEE/ACM 第二届区块链软件工程新兴趋势国际研讨会（WETSEB）*，2019年5月，第8–15页。

+   [4] Y. Fang, D. Wu, X. Yi, S. Wang, Y. Chen, M. Chen, Y. Liu, 和 L. Jiang, “超越“protected”和“private”：智能合约中自定义函数修饰符的经验性安全分析，”发表于 *第32届ACM SIGSOFT国际软件测试与分析研讨会论文集*，系列ISSTA 2023，美国纽约，2023年，第1157–1168页。

+   [5] L. Brent, N. Grech, S. Lagouvardos, B. Scholz, 和 Y. Smaragdakis, “Ethainter: 一个用于复合漏洞的智能合约安全分析器，”发表于 *第41届ACM SIGPLAN编程语言设计与实现大会论文集*，系列PLDI 2020，美国纽约，2020年，第454–469页。

+   [6] J. Chen, X. Xia, D. Lo, J. Grundy, X. Luo, 和 T. Chen, “Defectchecker: 通过分析EVM字节码自动检测智能合约缺陷，” *IEEE软件工程学报*，第48卷，第7期，第2189–2207页，2022年7月。

+   [7] L. Brent, A. Jurisevic, M. Kong, E. Liu, F. Gauthier, V. Gramoli, R. Holz, 和 B. Scholz, “Vandal: 一个用于智能合约的可扩展安全分析框架，”编号arXiv:1809.03981，2018年9月。

+   [8] S. Kalra, S. Goel, M. Dhawan, 和 S. Sharma, “ZEUS: 智能合约安全性分析，”发表于 *ISOC NDSS会议论文集*，2018年。

+   [9] P. Tsankov, A. Dan, D. Drachsler-Cohen, A. Gervais, F. Bünzli, 和 M. Vechev, “Securify: 智能合约的实用安全分析，”发表于 *ACM CCS会议论文集*，2018年。

+   [10] M. Mossberg, F. Manzano, E. Hennenfent, A. Groce, G. Grieco, J. Feist, T. Brunson, 和 A. Dinaburg, “Manticore：一个面向二进制文件和智能合约的用户友好的符号执行框架”，发表于 *2019年第34届IEEE/ACM自动化软件工程国际会议论文集(ASE)*，2019年11月，第1186–1189页。

+   [11] Defillama, “Defillama 漏洞事件”，2024年。[在线]. 可用：[https://defillama.com/hacks](https://defillama.com/hacks)

+   [12] I. David, L. Zhou, K. Qin, D. Song, L. Cavallaro, 和 A. Gervais, “你还需要手动审计智能合约吗？” no. arXiv:2306.12338，2023年6月。

+   [13] Y. Sun, D. Wu, Y. Xue, H. Liu, H. Wang, Z. Xu, X. Xie, 和 Y. Liu, “GPTScan：结合 GPT 和程序分析检测智能合约中的逻辑漏洞”，发表于 *第46届IEEE/ACM软件工程国际会议论文集*，2024年。

+   [14] S. Hu, T. Huang, F. Ilhan, S. Tekin, 和 L. Liu, “基于大型语言模型的智能合约漏洞检测：新视角”，发表于 *2023年第五届IEEE智能系统与应用中的信任、隐私与安全国际会议论文集(TPS-ISA)*，加利福尼亚州洛斯阿拉米托斯，2023年11月，第297–306页。

+   [15] Y. Sun, D. Wu, Y. Xue, H. Liu, W. Ma, L. Zhang, M. Shi, 和 Y. Liu, “LLM4Vuln：解耦并增强 LLMs 漏洞推理的统一评估框架”，no. arXiv:2401.16185，2024年1月。

+   [16] P. Lewis, E. Perez, A. Piktus, F. Petroni, V. Karpukhin, N. Goyal, H. Küttler, M. Lewis, W.-t. Yih, T. Rocktäschel, S. Riedel, 和 D. Kiela, “知识密集型 NLP 任务的检索增强生成”，发表于 *第34届神经信息处理系统国际会议论文集*，系列 NIPS'20。纽约州红钩：Curran Associates Inc.，2020年12月，第9459–9474页。

+   [17] K. Tian, E. Mitchell, H. Yao, C. D. Manning, 和 C. Finn, “微调语言模型以提高事实性”，no. arXiv:2311.08401，2023年11月。

+   [18] K. Lv, Y. Yang, T. Liu, Q. Gao, Q. Guo, 和 X. Qiu, “有限资源下的大型语言模型全参数微调”，no. arXiv:2306.09782，2023年6月。

+   [19] A. Balaguer, V. Benara, R. L. d. F. Cunha, R. d. M. E. Filho, T. Hendry, D. Holstein, J. Marsman, N. Mecklenburg, S. Malvar, L. O. Nunes, R. Padilha, M. Sharp, B. Silva, S. Sharma, V. Aski, 和 R. Chandra, “RAG 与微调：管道、权衡及农业案例研究”，no. arXiv:2401.08406，2024年1月。

+   [20] iAudit, “iaudit 推理代码和数据集，”2024年。[在线]. 可用：[https://sites.google.com/view/iaudittool/home](https://sites.google.com/view/iaudittool/home)

+   [21] A. Vaswani, N. Shazeer, N. Parmar, J. Uszkoreit, L. Jones, A. N. Gomez, Ł. Kaiser, 和 I. Polosukhin, “Attention is all you need,” *神经信息处理系统进展*，第30卷，2017年。

+   [22] J. Devlin, M.-W. Chang, K. Lee 和 K. Toutanova, “Bert：用于语言理解的深度双向Transformer模型的预训练，”发表于 *2019年北美计算语言学协会年会：人类语言技术论文集，第1卷（长篇与短篇论文）*，美国明尼阿波利斯，2019年6月，页码4171–4186。

+   [23] Z. Feng, D. Guo, D. Tang, N. Duan, X. Feng, M. Gong, L. Shou, B. Qin, T. Liu, D. Jiang 和 M. Zhou, “CodeBERT：一种针对编程语言和自然语言的预训练模型，”发表于 *计算语言学协会发现：EMNLP 2020*，T. Cohn、Y. He 和 Y. Liu 主编，在线发布，2020年11月，页码1536–1547。

+   [24] A. Radford, J. Wu, R. Child, D. Luan, D. Amodei 和 I. Sutskever, “语言模型是无监督多任务学习者，”2019年。

+   [25] T. Brown, B. Mann, N. Ryder, M. Subbiah, J. D. Kaplan, P. Dhariwal, A. Neelakantan, P. Shyam, G. Sastry, A. Askell *等人*，“语言模型是少量学习者，” *神经信息处理系统进展*，第33卷，页码1877–1901，2020年。

+   [26] M. Lewis, Y. Liu, N. Goyal, M. Ghazvininejad, A. Mohamed, O. Levy, V. Stoyanov 和 L. Zettlemoyer, “Bart：用于自然语言生成、翻译和理解的去噪序列到序列预训练模型，”编号arXiv:1910.13461，2019年10月。

+   [27] C. Raffel, N. Shazeer, A. Roberts, K. Lee, S. Narang, M. Matena, Y. Zhou, W. Li 和 P. J. Liu, “使用统一的文本到文本Transformer探索迁移学习的极限，” *机器学习研究期刊*，第21卷，第1期，2020年1月。

+   [28] Y. Wang, W. Wang, S. Joty 和 S. C. Hoi, “Codet5：面向代码理解和生成的标识符感知统一预训练编码器-解码器模型，”发表于 *2021年自然语言处理实证方法会议论文集*，2021年，页码8696–8708。

+   [29] W. X. Zhao, K. Zhou, J. Li, T. Tang, X. Wang *等人*，“大规模语言模型的调查，”编号arXiv:2303.18223，2023年11月。

+   [30] Y. Chang, X. Wang, J. Wang, Y. Wu, L. Yang, K. Zhu, H. Chen, X. Yi, C. Wang, Y. Wang, W. Ye, Y. Zhang, Y. Chang, P. S. Yu, Q. Yang 和 X. Xie, “关于大规模语言模型评估的调查，” *ACM智能系统技术学报*，2024年1月，已接收。

+   [31] J. Kaplan, S. McCandlish, T. Henighan, T. B. Brown, B. Chess, R. Child, S. Gray, A. Radford, J. Wu 和 D. Amodei, “神经语言模型的规模规律，”编号arXiv:2001.08361，2020年1月。

+   [32] Google, “Google Gemini AI，”2024年。[在线]. 可用链接：[https://blog.google/technology/ai/google-gemini-ai](https://blog.google/technology/ai/google-gemini-ai)

+   [33] H. Touvron, L. Martin, K. Stone *等人*， “Llama 2：开放基础和微调聊天模型，”编号arXiv:2307.09288，2023年7月，arXiv:2307.09288 [cs]。

+   [34] L. Xu, H. Xie, S.-Z. J. Qin, X. Tao 和 F. L. Wang, “针对预训练语言模型的参数高效微调方法：批判性回顾与评估，”编号arXiv:2312.12148，2023年12月。

+   [35] Z. Wan, X. Wang, C. Liu, S. Alam, Y. Zheng, J. Liu, Z. Qu, S. Yan, Y. Zhu, Q. Zhang, M. Chowdhury, 和 M. Zhang, “高效的大规模语言模型：一项调查”，编号arXiv:2312.03863，2024年1月。

+   [36] Q. Dong, L. Li, D. Dai, C. Zheng, Z. Wu, B. Chang, X. Sun, J. Xu, L. Li, 和 Z. Sui, “关于上下文学习的调查”，编号arXiv:2301.00234，2023年6月。

+   [37] Z. Hu, L. Wang, Y. Lan, W. Xu, E.-P. Lim, L. Bing, X. Xu, S. Poria, 和 R. Lee, “LLM-adapters：一种用于大规模语言模型参数高效微调的适配器系列”，发表于 *2023年自然语言处理经验方法会议论文集*，新加坡，2023年12月，页5254–5276。

+   [38] W. Song, Z. Li, L. Zhang, H. Zhao, 和 B. Du, “稀疏足以在微调预训练的大规模语言模型中”，编号arXiv:2312.11875，2023年12月。

+   [39] E. J. Hu, P. Wallis, Z. Allen-Zhu, Y. Li, S. Wang, L. Wang, W. Chen *等*，“Lora：大规模语言模型的低秩适配”，发表于 *国际学习表征会议*，2021年。

+   [40] X. L. Li 和 P. Liang, “前缀调优：优化连续提示以进行生成”，发表于 *第59届计算语言学协会年会暨第11届国际联合自然语言处理会议论文集（卷1：长篇论文）*，2021年，页4582–4597。

+   [41] B. Lester, R. Al-Rfou, 和 N. Constant, “规模的力量对参数高效的提示调优”，发表于 *2021年自然语言处理经验方法会议论文集*，在线与多米尼加共和国蓬塔卡纳，2021年11月，页3045–3059。

+   [42] N. Mundra, S. Doddapaneni, R. Dabre, A. Kunchukuttan, R. Puduppully, 和 M. M. Khapra, “适配器效率的综合分析”，发表于 *第七届数据科学与数据管理联合国际会议论文集（第11届ACM IKDD CODS与第29届COMAD）*，2024年，页136–154。

+   [43] A. Petrov, P. Torr, 和 A. Bibi, “何时提示和前缀调优有效？能力与局限性的理论”，发表于 *第十二届国际学习表征会议*，2024年。

+   [44] D. A. Zetzsche, D. W. Arner, 和 R. P. Buckley, “去中心化金融 (defi)”，*金融监管杂志*，第6卷，页172–203，2020年。

+   [45] Defillama, “Defillama链”，2024年。[在线]。可用链接：[https://defillama.com/chains](https://defillama.com/chains)

+   [46] P. Praitheeshan, L. Pan, J. Yu, J. Liu, 和 R. Doss, “以太坊智能合约漏洞的安全分析方法：一项调查”，编号arXiv:1908.08605，2020年9月。

+   [47] P. Züst, T. Nadahalli, 和 Y. W. R. Wattenhofer, “分析与防止以太坊的三明治攻击”，*苏黎世联邦理工学院*，2021年。

+   [48] C. Zhou, J. He, X. Ma, T. Berg-Kirkpatrick, 和 G. Neubig, “零-shot任务泛化的提示一致性”，发表于 *2022年计算语言学会年会论文集：EMNLP 2022*，阿布扎比，阿联酋，2022年12月，页2613–2626。

+   [49] B. Rozière, J. Gehring, F. Gloeckle, S. Sootla *等人*, “Code llama: 开放式代码基础模型，”编号arXiv:2308.12950，2024年1月。

+   [50] R. Taori, I. Gulrajani, T. Zhang, Y. Dubois, X. Li, C. Guestrin, P. Liang, 和 T. B. Hashimoto, “斯坦福Alpaca: 一种指令跟随型的Llama模型，”[https://github.com/tatsu-lab/stanford_alpaca](https://github.com/tatsu-lab/stanford_alpaca)，2023年。

+   [51] T. Kojima, S. S. Gu, M. Reid, Y. Matsuo, 和 Y. Iwasawa, “大型语言模型是零-shot推理器，” *神经信息处理系统进展*，第35卷，页码22 199–22 213，2022年。

+   [52] A. Q. Jiang, A. Sablayrolles, A. Roux, A. Mensch, 等人, “Mixtral of experts，”编号arXiv:2401.04088，2024年1月。

+   [53] A. Q. Jiang, A. Sablayrolles, A. Mensch, 等人, “Mistral 7b，”编号arXiv:2310.06825，2023年10月。

+   [54] F. Xue, Z. Zheng, Y. Fu, J. Ni, Z. Zheng, W. Zhou, 和 Y. You, “Openmoe: 一项开放专家混合语言模型的早期工作，”编号arXiv:2402.01739，2024年1月。

+   [55] Solodit, “Solodit，” 2024年。[在线]. 可用链接: [https://solodit.xyz/](https://solodit.xyz/)

+   [56] B. J. Frey 和 D. Dueck, “通过数据点之间的消息传递进行聚类，” *科学*，第315卷，第5814期，页码972–976，2007年。

+   [57] X. Yi, D. Wu, L. Jiang, Y. Fang, K. Zhang, 和 W. Zhang, “区块链系统漏洞的实证研究：模块、类型与模式，”在 *第30届ACM欧洲软件工程联合会议暨软件工程基础研讨会*，2022年，页码709–721。

+   [58] D. Guo, S. Ren, S. Lu, Z. Feng, D. Tang, L. Shujie, L. Zhou, N. Duan, A. Svyatkovskiy, S. Fu *等人*, “Graphcodebert: 利用数据流进行代码表示的预训练，”在 *国际学习表示大会*，2020年。

+   [59] D. Guo, S. Lu, N. Duan, Y. Wang, M. Zhou, 和 J. Yin, “Unixcoder: 统一跨模态预训练的代码表示，”在 *第60届计算语言学会年会论文集（第一卷：长篇论文）*，2022年，页码7212–7225。

+   [60] Huggingface, “Huggingface transformer,” 2024\. [在线]. 可用链接: [https://huggingface.co/docs/transformers/](https://huggingface.co/docs/transformers/)

+   [61] Y. Zhuang, Z. Liu, P. Qian, Q. Liu, X. Wang, 和 Q. He, “使用图神经网络进行智能合约漏洞检测，”在 *第二十九届国际人工智能联合会议*，第3卷，2020年7月，页码3283–3290。

+   [62] Z. Liu, P. Qian, X. Wang, L. Zhu, Q. He, 和 S. Ji, “智能合约漏洞检测：从纯神经网络到可解释的图特征与专家模式融合，”在 *第二十九届国际人工智能联合会议*，第3卷，2021年8月，页码2751–2759。

+   [63] H. Wu, Z. Zhang, S. Wang, Y. Lei, B. Lin, Y. Qin, H. Zhang, 和 X. Mao, “Peculiar：基于关键数据流图和预训练技术的智能合约漏洞检测，”发表于*2021 IEEE第32届国际软件可靠性工程研讨会（ISSRE）*，2021年10月，第378–389页。

+   [64] S. Ullah, M. Han, S. Pujar, H. Pearce, A. Coskun, 和 G. Stringhini, “大语言模型目前无法可靠地识别和推理安全漏洞（尚未？）：全面评估、框架和基准测试，”发表于*2024 IEEE安全与隐私研讨会（SP）*，美国加利福尼亚州洛斯阿拉米托斯，2024年5月，第199–199页。

+   [65] M. Fu, C. Tantithamthavorn, V. Nguyen, 和 T. Le, “Chatgpt在漏洞检测、分类和修复中的应用：我们还差多远？”发表于*2023年第30届亚太软件工程大会（APSEC）*，美国加利福尼亚州洛斯阿拉米托斯，2023年12月，第632–636页。

+   [66] C. Thapa, S. I. Jang, M. E. Ahmed, S. Camtepe, J. Pieprzyk, 和 S. Nepal, “基于变换器的语言模型用于软件漏洞检测，”发表于*第38届年度计算机安全应用大会论文集*，ACSAC ’22系列。美国纽约：计算机协会，2022年12月，第481–496页。

+   [67] M. Alqarni 和 A. Azim, “使用先进BERT语言模型进行低级源代码漏洞检测，”发表于*加拿大人工智能会议论文集*。加拿大人工智能协会（CAIAC），2022年5月。

+   [68] J. Devlin, M.-W. Chang, K. Lee, 和 K. Toutanova, “BERT：深度双向变换器预训练用于语言理解，”发表于*2019年北美计算语言学会会议：人类语言技术，第1卷（长短论文）*，J. Burstein, C. Doran, 和 T. Solorio编辑。美国明尼苏达州明尼阿波利斯：计算语言学会，2019年6月，第4171–4186页。

+   [69] H. Li, Y. Hao, Y. Zhai, 和 Z. Qian, “利用大语言模型辅助静态分析：一个Chatgpt实验，”发表于*第31届ACM联合欧洲软件工程会议暨软件工程基础研讨会论文集*，ESEC/FSE 2023系列，美国纽约，2023年，第2107–2111页。

+   [70] S. J. Wang, K. Pei, 和 J. Yang, “Smartinv：智能合约不变量推理的多模态学习，”发表于*2024 IEEE安全与隐私研讨会（SP）*。IEEE计算机学会，2024年，第126–126页。

+   [71] Y. Liu, Y. Xue, D. Wu, Y. Sun, Y. Li, M. Shi, 和 Y. Liu, “Propertygpt：通过检索增强的属性生成进行智能合约的LLM驱动形式化验证，”*arXiv预印本 arXiv:2405.02580*，2024年。

+   [72] G. Team, T. Mesnard, C. Hardin, R. Dadashi, S. Bhupatiraju, S. Pathak *等*，“Gemma：基于Gemini研究和技术的开放模型，”arXiv:2403.08295号，2024年3月。
