<!--yml

分类：未分类

日期：2025-01-11 12:16:14

-->

# 基于LLM的非合作环境中的多智能体诗歌生成

> 来源：[https://arxiv.org/html/2409.03659/](https://arxiv.org/html/2409.03659/)

Ran Zhang

商学院与数学学院

曼海姆大学

德国曼海姆B6 26, 68159

ran.zhang@uni-mannheim.de & Steffen Eger

曼海姆大学商学院与数学学院

曼海姆大学

德国曼海姆B6 26, 68159

steffen.eger@uni-mannheim.de

###### 摘要

尽管大语言模型（LLMs）在自动诗歌生成方面取得了显著进展，但生成的诗歌缺乏多样性，而且训练过程与人类学习有很大不同。基于这样的理念，即诗歌生成系统的学习过程应更加类似于人类，且其输出应更具多样性和新颖性，我们提出了一个基于社会学习的框架，其中强调非合作交互而非仅仅合作交互，以促进多样性。我们的实验是首次尝试在非合作环境中应用基于LLM的多智能体系统进行诗歌生成，涉及训练型智能体（GPT-2）和基于提示的智能体（GPT-3和GPT-4）。我们基于96k生成的诗歌进行的评估显示，我们的框架有利于训练型智能体的诗歌生成过程，具体表现为：1）多样性提高了3.0-3.7个百分点（pp），新颖性提高了5.6-11.3个百分点（pp），根据不同和新颖的n-gram。训练型智能体生成的诗歌在词汇、风格和语义上也表现出群体间的分歧。基于提示的智能体在我们的框架中也受益于非合作环境，且由非同质化智能体组成的更多样化模型集合有潜力进一步增强多样性，根据我们的实验，增加了7.0-17.5个百分点。然而，基于提示的智能体随着时间的推移在词汇多样性上有所下降，并未展现出我们所期望的群体间分歧。我们的论文主张在创意任务中（如自动诗歌生成）进行范式转变，引入类似于人类互动的社会学习过程（通过基于LLM的智能体建模）。

*关键词* 诗歌生成  $\cdot$ 社会学习  $\cdot$ 多智能体系统

## 1 引言

由大型语言模型（LLMs）驱动的自主代理在多个领域取得了显著进展，包括复杂任务解决李等人（[2024](https://arxiv.org/html/2409.03659v2#bib.bib43)）、推理林等人（[2024](https://arxiv.org/html/2409.03659v2#bib.bib45)）；杜等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib18)）以及模拟王等人（[2023a](https://arxiv.org/html/2409.03659v2#bib.bib74)）。研究表明，通过单一或多代理系统的互动交流可以产生涌现行为朴等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib54)）、增强任务表现朱戈等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib92)）、更好的评估陈等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib10)）以及在开放式生成任务中的协助朱等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib91)）等。尽管取得了这些进展，但利用基于LLM的代理进行创意任务（如诗歌生成）的探索仍然有限查克拉巴尔提等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib9)）。本文介绍了首次基于LLM的多代理诗歌生成实验。¹¹1 代码、数据和生成的输出可以在[https://github.com/zhangr2021/Multiagent_poetry.git](https://github.com/zhangr2021/Multiagent_poetry.git)公开获取。我们介绍了一个框架，强调非合作环境，以增强生成诗歌的多样性和新颖性，这种增强体现在时间上的总体平均值以及在迭代中的动态变化。

![参见标题](img/ede88b4676e55e2983001e706285ff07.png)

图1：预定义社交网络的示意图（$M=4$）以及基于训练的代理（GPT-2）和基于提示的代理（GPT-3.5和GPT-4）学习过程的高级描述。社交网络中的绿线和红线分别表示代理之间的合作（+）和非合作（-）互动。

为什么进行诗歌生成？

“诗歌是用语言创造美的有节奏的艺术。”

– 埃德加·爱伦·坡

尽管大型语言模型（LLMs）在不断发展，但生成诗歌仍然是一项困难的任务，因为风格、意义和人类情感之间的复杂相互作用（Chakrabarty等，[2021](https://arxiv.org/html/2409.03659v2#bib.bib7)；Mahbub等，[2023](https://arxiv.org/html/2409.03659v2#bib.bib51)）。模型不仅需要在语言能力方面表现出色，比如理解语义和语法，还需要在捕捉诗歌的风格元素方面做到精湛，例如押韵、韵律、意象和艺术感，以生成类似人类的诗歌（Zhipeng等，[2019](https://arxiv.org/html/2409.03659v2#bib.bib90)；Belouadi和Eger，[2023](https://arxiv.org/html/2409.03659v2#bib.bib3)；Ma等，[2023](https://arxiv.org/html/2409.03659v2#bib.bib50)）。当前诗歌生成的挑战包括：1）为特定风格或主题定制的微调模型或流程的局限性（Lau等，[2018](https://arxiv.org/html/2409.03659v2#bib.bib38)；Van de Cruys，[2020](https://arxiv.org/html/2409.03659v2#bib.bib73)；Tian和Peng，[2022](https://arxiv.org/html/2409.03659v2#bib.bib70)）；2）与人类创作相比，LLMs在零样本和少样本情境下难以生成多样且富有美感的诗歌（Sawicki等，[2023a](https://arxiv.org/html/2409.03659v2#bib.bib59)，[b](https://arxiv.org/html/2409.03659v2#bib.bib60)）。诗歌生成中的复杂性和约束使其成为多智能体系统的适用场景，鉴于已知多智能体系统能够解决复杂任务，并且有可能通过利用“多个诗人的组合”来增强诗歌的多样性（Yi等，[2020](https://arxiv.org/html/2409.03659v2#bib.bib85)）。

为什么选择多智能体系统和社会学习？

目前，机器学习和自然语言处理中的诗歌生成过程与人类学习和创作诗歌的方式存在显著差异。不同于训练单一诗歌生成模型并从特定数据集学习的范式，人类是在一个社会环境中学习的，在这个环境中，他们通过口头或书面、正式或非正式的交流与他人互动（Jarvis [2012](https://arxiv.org/html/2409.03659v2#bib.bib31)）。这引出了一个问题，即是否一种更接近人类的生成方法能改善诗歌创作。多智能体系统在社会模拟中的丰富应用（Park et al. [2023](https://arxiv.org/html/2409.03659v2#bib.bib54)；Chuang et al. [2023](https://arxiv.org/html/2409.03659v2#bib.bib13)）非常适合我们的目标，因为通过基于大语言模型（LLM）的智能体可以有效地模拟人类行为，例如诗歌创作，并且能够很好地嵌入社会网络中。此外，作为创作性构思的最基本元素之一，*“发散”*思维能力（即偏离既定规范的能力）在学习阶段对人类和计算创造力都至关重要（Elgammal et al. [2017](https://arxiv.org/html/2409.03659v2#bib.bib20)；Brinkmann et al. [2023](https://arxiv.org/html/2409.03659v2#bib.bib6)；Wingström et al. [2023](https://arxiv.org/html/2409.03659v2#bib.bib80)）。这为我们采用一种社会学习过程提供了理论依据，这种过程不仅支持合作，还支持对抗，作为过程中的主要偏离源（Eger [2016](https://arxiv.org/html/2409.03659v2#bib.bib19)；Shi et al. [2019](https://arxiv.org/html/2409.03659v2#bib.bib66)）。此外，与以往关注弱对抗形式（如辩论和争论，强调通过思维纠正来完善差异的研究）（Chan et al. [2023](https://arxiv.org/html/2409.03659v2#bib.bib10)）不同，我们提出的社会学习过程旨在放大发散性，以增强多样性。

为什么是非合作环境？

人类行为的一个方面是在各种情境中的非合作性互动。例如，政治领域常常表现为个体政党之间的对立，或者在更广泛的背景下，‘反文化’反抗体制。此外，文学/艺术/哲学的一个定义性特征是将自己与之前或同时代的‘竞争者’区分开来。举一些例子，海明威的‘冰山’写作风格与他的前辈有很大不同，后者的写作风格更为感伤（Baker [1972](https://arxiv.org/html/2409.03659v2#bib.bib2)）；印象派画家通过新的内容和风格挑战了传统艺术界设定的绘画标准；哲学家亚瑟·叔本华强烈反对黑格尔的哲学，哲学家们常常形成对立的群体，例如‘康德主义者’、‘新柏拉图主义者’等（Janaway [2002](https://arxiv.org/html/2409.03659v2#bib.bib30)）。在计算社会科学中，‘对抗性’、‘非合作’或‘负面关系’等术语用来描述这种行为（Amirkhani 和 Barshooi [2022](https://arxiv.org/html/2409.03659v2#bib.bib1)）。在机器学习或自然语言处理领域，这种行为仍然很少被探索或在建模中得到利用（Gautier 等 [2022](https://arxiv.org/html/2409.03659v2#bib.bib25)）；Lei 等 [2022](https://arxiv.org/html/2409.03659v2#bib.bib40)）。

我们如何利用基于LLM的代理进行诗歌创作？

我们在一个预定义的社交网络上构建了我们的社交学习框架，该网络管理着图 [1](https://arxiv.org/html/2409.03659v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ LLM-based multi-agent poetry generation in non-cooperative environments") 中所示的代理之间的互动。该网络不仅促进合作性互动（“诗人欣赏彼此的作品并相互学习”），还促进非合作性互动（“诗人不喜欢彼此的作品并且互相偏离”）。我们提出了基于两种不同LLM的学习框架：1）基于提示的对话代理（GPT-4），用于回答我们的框架能否在零-shot 或少-shot 设置中帮助生成多样化的诗歌？以及2）基于训练的代理（GPT-2），我们研究了各种训练和解码配置，例如训练损失和互动代理的数量，以确定哪种策略在非合作环境中生成诗歌时最为有效？我们的框架包括三个主要组件：（1）社交网络，（2）学习过程，（3）学习策略。我们的主要贡献在于开发了一个新的学习框架，整合了现有方法，以扩展其范围和效果。

在非合作环境中，基于训练的代理随着时间的推移生成的诗歌在多样性和新颖性上不断增加：我们在第[5](https://arxiv.org/html/2409.03659v2#S5 "5 实验结果 ‣ 基于LLM的多代理在非合作环境中的诗歌生成")节的评估表明，我们的框架有助于生成过程，从而根据不同且新颖的n-grams提升基于训练的代理的多样性和新颖性。基于训练的代理生成的诗歌在词汇、风格和语义上也呈现出按预定义群体隶属关系的群体分歧。从第[6](https://arxiv.org/html/2409.03659v2#S6 "6 讨论与分析 ‣ 基于LLM的多代理在非合作环境中的诗歌生成")节的分析中还表明，非合作条件能增强基于提示的代理的多样性，并且采用更多样化的模型集成和非同质化代理的潜在好处。然而，我们框架中的基于提示的代理并未表现出任何基于群体的分歧。此外，基于提示的代理随着时间的推移更容易生成风格单一的诗歌。

## 2 相关工作

我们的研究与以下几个方面相关：1) 基于LLM的代理交互，我们专注于交互的形式；2) 建模交互的相应方法，即语言模型集成和控制文本生成（CTG），其中CTG技术是我们在3) 诗歌生成中的应用案例所需要的。

基于LLM的代理交互 代理之间的交互形式可以大致分为合作性和非合作性。代理之间的合作性沟通很常见，目标是通过协作做出联合决策，例如反复沟通（Li et al. ([2024](https://arxiv.org/html/2409.03659v2#bib.bib43))）、多数投票（Hamilton ([2023](https://arxiv.org/html/2409.03659v2#bib.bib28))）或两者的结合（Zhuge et al. ([2023](https://arxiv.org/html/2409.03659v2#bib.bib92))）。相比之下，非合作性交互虽然不太普遍，但可以通过代理之间的辩论或争论提升回应质量（Chan et al. ([2023](https://arxiv.org/html/2409.03659v2#bib.bib10)); Du et al. ([2023](https://arxiv.org/html/2409.03659v2#bib.bib18))）。此外，尽管大多数研究集中在代理之间的静态交互上，一些研究者深入探讨了代理交互的动态性。Liu et al. ([2023](https://arxiv.org/html/2409.03659v2#bib.bib48)) 提出了一个动态交互架构，他们利用优化算法在推理时选择最佳代理。Autogen Wu et al. ([2023](https://arxiv.org/html/2409.03659v2#bib.bib82)) 也实现了动态群聊，指导代理在进行中的对话中交互流程。其他研究则专注于交互过程中的输出动态。Chuang et al. ([2023](https://arxiv.org/html/2409.03659v2#bib.bib13)) 发现，LLM倾向于与事实信息对齐，无论其人物设定和初始状态如何，这限制了基于LLM的代理在模拟意见动态时的能力。最近，群体交互的动态性也被研究，研究表明，结合链式思维推理、详细人物设定和精细调优LLM能够增强代理复制类人群体动态的能力（Chuang et al. ([2024](https://arxiv.org/html/2409.03659v2#bib.bib14))）。我们的工作不同之处在于：1）我们构建了一个具有群体归属关系的社交网络，以获得更类人的学习过程；2）我们提出了一个框架，涉及两种交互形式，特别强调非合作性沟通；3）我们关注在基于训练的代理精细调优下的生成过程输出动态，并使用详细的人物设定进行连续提示的基于提示的代理。

语言模型集成与受控文本生成 我们框架的建模要求：1）多个语言模型（LMs）的集成；2）生成的输出需要捕捉与诗歌生成应用场景相关的一般诗意风格。这涉及到语言模型集成（LM ensemble）和受控文本生成（CTG）领域。

LM 集成可以分为两种：1）对话式集成，不涉及参数训练，Wang 等人（[2022a](https://arxiv.org/html/2409.03659v2#bib.bib77)）和 2）基于微调的集成，即神经网络集成，Shazeer 等人（[2016](https://arxiv.org/html/2409.03659v2#bib.bib63)）和输出集成，Dekoninck 等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib16)）；Jiang 等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib33)）。基于提示的对话式集成通常用于推理任务，其中 LLM 会对其自身的回答进行集成（即自我集成），Wang 等人（[2022a](https://arxiv.org/html/2409.03659v2#bib.bib77)）；Fu 等人（[2022](https://arxiv.org/html/2409.03659v2#bib.bib22)）。最近，Lu 等人（[2024](https://arxiv.org/html/2409.03659v2#bib.bib49)）通过一种参数高效且可解释的方式，结合了多个小型对话模型，并通过 A/B 测试证明其表现超过了 ChatGPT。基于微调的集成可以在神经网络层或输出层操作。虽然神经网络集成通常需要通过大量数据集和资源进行大规模训练或微调，Shazeer 等人（[2016](https://arxiv.org/html/2409.03659v2#bib.bib63)）；Jiang 等人（[2024](https://arxiv.org/html/2409.03659v2#bib.bib32)），但像适配器这样的较小模块的混合，成为了资源有限情况下的可行解决方案，Wang 等人（[2022b](https://arxiv.org/html/2409.03659v2#bib.bib78)）；Chronopoulou 等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib12)）。

CTG，即生成文本时根据情感等属性生成的任务，Firdaus 等人 ([2022](https://arxiv.org/html/2409.03659v2#bib.bib21))；Ruan 和 Ling ([2021](https://arxiv.org/html/2409.03659v2#bib.bib58))，主题 Dathathri 等人 ([2019](https://arxiv.org/html/2409.03659v2#bib.bib15))；Wang 等人 ([2019](https://arxiv.org/html/2409.03659v2#bib.bib76))，有毒性回避 Liu 等人 ([2021](https://arxiv.org/html/2409.03659v2#bib.bib47))，风格 Belouadi 和 Eger ([2023](https://arxiv.org/html/2409.03659v2#bib.bib3))；Shao 等人 ([2021b](https://arxiv.org/html/2409.03659v2#bib.bib62))，去偏见 Dinan 等人 ([2020](https://arxiv.org/html/2409.03659v2#bib.bib17))；Sheng 等人 ([2020](https://arxiv.org/html/2409.03659v2#bib.bib65))，等等，也与我们的研究相关。CTG 可以在训练/微调和推理阶段运行。类似于语言模型集成，使用任务相关适配器等额外模块进行微调，如 Ribeiro 等人 ([2021](https://arxiv.org/html/2409.03659v2#bib.bib57))；Lin 等人 ([2021](https://arxiv.org/html/2409.03659v2#bib.bib46))，也用于获得参数高效的可控性。此外，针对 CTG 应用程序，以减少生成不良属性的概率，除了在文本生成微调中使用的标准交叉熵损失外，还探索了其他损失函数。Qian 等人 ([2022](https://arxiv.org/html/2409.03659v2#bib.bib55)) 还加入了对比损失，这对于脱毒任务至关重要，但只部分改善了情感控制任务的性能。Zheng 等人 ([2023](https://arxiv.org/html/2409.03659v2#bib.bib89)) 也在序列似然性上采用了对比损失，以降低负样本的生成概率。相比之下，在 LLM 时代，推理阶段的操作更具可行性，Jiang 等人 ([2023](https://arxiv.org/html/2409.03659v2#bib.bib33))；Wang 等人 ([2023b](https://arxiv.org/html/2409.03659v2#bib.bib75))；Dekoninck 等人 ([2023](https://arxiv.org/html/2409.03659v2#bib.bib16))。重新排序输出是一个常见的解决方案。例如，Jiang 等人 ([2023](https://arxiv.org/html/2409.03659v2#bib.bib33)) 首先重新排序来自多个 LLM 的完整候选输出，然后融合前 K 个答案。在解码阶段重新排序原始的下一个令牌分布也被广泛探索，例如利用判别器 Dathathri 等人 ([2019](https://arxiv.org/html/2409.03659v2#bib.bib15)) 或结合（反）专家模型的意见（即输出的 logits） Liu 等人 ([2021](https://arxiv.org/html/2409.03659v2#bib.bib47))；Dekoninck 等人 ([2023](https://arxiv.org/html/2409.03659v2#bib.bib16))。在推理阶段操作，尤其是在解码阶段，能够对生成的文本进行强控制，同时对时间和计算资源的需求较少。然而，这可能会导致文本质量略有下降，Dathathri 等人 ([2019](https://arxiv.org/html/2409.03659v2#bib.bib15))。另一方面，在训练/微调阶段操作能够保持高质量的文本，但控制性较弱，Zhang 等人 ([2023](https://arxiv.org/html/2409.03659v2#bib.bib86))。

在我们诗歌生成的应用场景中，我们考虑了基于提示和基于微调的集成方法。对于基于微调的方法，我们在两个阶段进行操作：1）在训练阶段，通过实验标准的交叉熵损失和（或）对比损失，使用适配器进行微调，以提高文本质量和效率；2）在解码阶段，获得更好的可控性，以应对非合作环境。

自动诗歌生成：早期的自动诗歌生成尝试主要依赖于语法规则 Oliveira ([2012](https://arxiv.org/html/2409.03659v2#bib.bib53))、统计规则 Jiang 和 Zhou ([2008](https://arxiv.org/html/2409.03659v2#bib.bib34))；Greene 等人 ([2010](https://arxiv.org/html/2409.03659v2#bib.bib27)) 和神经网络，如 RNNs Zhang 和 Lapata ([2014](https://arxiv.org/html/2409.03659v2#bib.bib88))；Ghazvininejad 等人 ([2017](https://arxiv.org/html/2409.03659v2#bib.bib26))；Wöckener 等人 ([2021](https://arxiv.org/html/2409.03659v2#bib.bib81))，尤其是基于 RNN 的编码器-解码器架构 Wang 等人 ([2016](https://arxiv.org/html/2409.03659v2#bib.bib79))；Lau 等人 ([2018](https://arxiv.org/html/2409.03659v2#bib.bib38))；Yan ([2016](https://arxiv.org/html/2409.03659v2#bib.bib83))。最近的模型则聚焦于基于 Transformer 的架构 Tian 等人 ([2021](https://arxiv.org/html/2409.03659v2#bib.bib69))；Shao 等人 ([2021a](https://arxiv.org/html/2409.03659v2#bib.bib61))。尽管 GPT 模型的变种在许多自然语言处理任务中展现了出色的表现，但对于其生成诗歌的能力，仍存在不同的看法。Bena 和 Kalita ([2019](https://arxiv.org/html/2409.03659v2#bib.bib4))；Liao 等人 ([2019](https://arxiv.org/html/2409.03659v2#bib.bib44))；LC ([2022](https://arxiv.org/html/2409.03659v2#bib.bib39)) 等研究通过对 GPT-2 进行微调，并加入情感、形式和主题等额外组件，根据人工评估结果生成了中到高质量的诗歌。Köbis 和 Mossink ([2021](https://arxiv.org/html/2409.03659v2#bib.bib36)) 认为，零-shot 的 GPT-2 能生成类人诗歌，在人工选出的最佳诗歌中，机器生成的诗歌可与人工创作的诗歌相媲美，但在没有人工预选的情况下，机器生成的诗歌容易被识别。Wöckener 等人 ([2021](https://arxiv.org/html/2409.03659v2#bib.bib81)) 指出，类似于基于 RNN 的模型，GPT-2 在学习诗歌特有的特征，如押韵和节奏时存在问题。为了克服这些不足，Belouadi 和 Eger ([2023](https://arxiv.org/html/2409.03659v2#bib.bib3)) 提出了 ByGPT5，这是一种端到端、无令牌的模型，条件化于押韵、节奏和头韵。根据自动评估和人工评估，该模型能够超越像 GPT-2、ByT5 和 ChatGPT（GPT3-3.5）这样的更大模型。他们还构建了一个定制的语料库 QuaTrain，包含大量机器标注的伪四行诗，以扩大微调数据集的规模。此外，Sawicki 等人 ([2023b](https://arxiv.org/html/2409.03659v2#bib.bib60)) 声称，GPT-3 在300首诗歌上进行微调后，能够成功生成特定作者风格的高质量诗歌，但未进行微调的 GPT-3.5 会生成风格相似但不理想的诗歌。Sawicki 等人 ([2023a](https://arxiv.org/html/2409.03659v2#bib.bib59)) 的研究也指出，GPT-3.5 和 GPT-4 在未微调的情况下无法生成符合期望风格的诗歌。最近，交互式诗歌生成引起了研究人员的关注，因为它能促进人机合作，在特定约束下生成风格更多样、质量更高的诗歌 Zhipeng 等人 ([2019](https://arxiv.org/html/2409.03659v2#bib.bib90))；Uthus 等人 ([2022](https://arxiv.org/html/2409.03659v2#bib.bib72))。Ma 等人 ([2023](https://arxiv.org/html/2409.03659v2#bib.bib50)) 提出了一个后处理系统，基于人类的约束对 GPT-2 的生成进行微调。Chakrabarty 等人 ([2022](https://arxiv.org/html/2409.03659v2#bib.bib8)) 提出的 CoPoet 微调了预训练的 T5，通过 <instruction, generation> 对来根据人类指令生成诗歌。他们的研究表明，微调后的 T5 模型不仅与更大的 InstructGPT 模型竞争力十足，而且能够与人类成功合作，创作出更好的诗歌。在我们的研究中，我们选择 GPT-2 作为基础模型，因为它在参数效率和语言能力之间提供了一个良好的平衡。与大多数优化少数诗歌特征的诗歌生成目标不同，我们利用具有任意风格的诗歌，并从 QuaTrain 语料库中随机抽取样本进行初始化，这些样本包含伪诗歌特征。此外，我们还探索了在多代理系统支持下的互动环境中，GPT-3.5 和 GPT-4 的潜力。

## 3 诗歌生成的社会学习框架

本节介绍了我们的诗歌生成的社会学习框架。最近LLM的发展促使了通过基于LLM的代理人模拟人类个体社会学习过程的各种尝试，Li等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib42)）；Chuang等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib13)）；Gao等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib23)）。我们的框架灵感来自这些过程，应用基于社交网络的方法进行诗歌生成。我们研究是否更像人类的学习过程（即社会学习）能够促进诗歌生成。我们与之前的研究有两个不同之处：1）我们的架构基于一个带符号的网络，在这个网络中，代理人不仅以合作方式互动，还以非合作方式互动；2）我们引入了针对基于提示的代理人（GPT-3.5和GPT-4）和基于训练的代理人（GPT-2）的学习框架。我们的框架由三部分组成：（1）社交网络，（2）学习过程和（3）学习策略。我们将在下面进行描述。

### 3.1 社交网络

我们考虑一个有$M$个LLM增强代理人的带符号社交网络，其中两个代理人之间的每条链接都与一个正号或负号相关联，Leskovec等人（[2010](https://arxiv.org/html/2409.03659v2#bib.bib41)）；Eger（[2016](https://arxiv.org/html/2409.03659v2#bib.bib19)）；Shi等人（[2019](https://arxiv.org/html/2409.03659v2#bib.bib66)）。我们将这$M$个代理人分为两个小组，如图[1](https://arxiv.org/html/2409.03659v2#S1.F1 "图 1 ‣ 1 引言 ‣ 基于LLM的多代理人诗歌生成在非合作环境中的应用")所示。属于同一小组的代理人称为‘内组’代理人，而来自不同小组的代理人则被称为‘外组’代理人。根据小组归属，我们预期两种类型的代理人之间的互动：1）‘内组’代理人作为‘朋友’紧密合作（正号）；2）‘外组’代理人作为‘敌人’，以非合作的方式调整他们的‘观点’（负号）。我们将与‘内组’代理人相关的学习过程称为正向学习，而与‘外组’代理人相关的学习过程称为负向学习。我们称同时从‘内组’和‘外组’中学习为联合学习。

在我们的诗歌生成应用中，代理人是经过预训练的LLM（大语言模型）。我们将LLM视为属于两个小组的不同诗人。‘内组’诗人欣赏彼此的作品，并旨在从彼此的风格中学习。相反，‘外组’诗人不喜欢彼此的作品，且旨在使自己的作品有所区分。

### 3.2 学习过程

| 变量 | 定义 |
| --- | --- |
| $a_{1},a_{2},\ldots$ | 归属于A组的代理人 |
| $b_{1},b_{2},\ldots$ | 归属于B组的代理人 |
| M | 代理人的总数 |
| $\mathcal{A}_{i}$ | 目标代理$\mathcal{A}_{i}\in\{a_{1},b_{1},a_{2},b_{2},\ldots\}$，其中$i\in\{1,2,\ldots,M\}$ |
| $\mathcal{A}_{i},\mathcal{A}^{+},\mathcal{A}^{-}$ | 代理元组：(目标代理，目标代理的“内群体”代理，目标代理的“外群体”代理)。例如，($a_{1},[a_{2},a_{3}],[b_{1},b_{2}]$) |
| $P_{{\mathcal{A}_{i}}},P_{*^{+}},P_{*^{-}}$ | 代理$\mathcal{A}_{i}$、代理$*^{+}\in\mathcal{A}^{+}$和代理$*^{-}\in\mathcal{A}^{-}$的下一个标记的条件概率分布 |
| N | 每个代理每次迭代生成的诗歌总数 |
| T | 总迭代次数 |
| t | 迭代次数$t\in\{1,2,\ldots,T\}$ |
| $o^{\mathcal{A}_{i}}$ | 代理$\mathcal{A}_{i}$生成的一首诗歌 |
| $O_{t}^{\mathcal{A}_{i}}$ | 代理$\mathcal{A}_{i}$在迭代$t$时生成的诗歌集合 |
| $S_{t}$ | 迭代$t$时所有生成的诗歌集合 |
| $F_{\text{generate}}$ | 代理元组$(\mathcal{A}_{i},\mathcal{A}^{-},\mathcal{A}^{+})$的生成函数 |
| $F_{\text{update}}$ | 基于最新生成的输出$S_{t}$和$S_{t-1}$的更新函数 |
| $t_{g}$ | 解码阶段的生成时间 |
| $x_{t_{g}}$ | 生成时间$t_{g}$时的标记 |
| $\boldsymbol{x_{t_{g}}}$ | 生成时间$t_{g}$时的输入序列 |
| #$\mathcal{A}$ | 解码阶段的交互代理数量 |

表1：符号说明

对于 *$t\leftarrow 1$到$T$* 执行       初始化空集合$S_{t}$以存储迭代$t$时生成的诗歌；       对于 *代理$\mathcal{A}_{i}$ // $i\in\{1,2,\ldots,M\}$* 执行             $O_{t}^{\mathcal{A}_{i}}\leftarrow F_{\text{generate}}(\mathcal{A}_{i},\mathcal{A}^{-},\mathcal{A}^{+})$来生成$N$首诗；             将$O_{t}^{\mathcal{A}_{i}}$加入$S_{t}$；       遍历结束      对于 *代理$\mathcal{A}_{i}$ // $i\in\{1,2,\ldots,M\}$* 执行             如果 *$t>1$* 则                   $\mathcal{A}_{i}\leftarrow F_{\text{update}}(S_{t},S_{t-1})$；            否则                   $\mathcal{A}_{i}\leftarrow F_{\text{update}}(S_{t})$；             如果结束       遍历结束

算法1 社会学习过程用于诗歌生成

现在，我们基于图[1](https://arxiv.org/html/2409.03659v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ LLM-based multi-agent poetry generation in non-cooperative environments")和算法[1](https://arxiv.org/html/2409.03659v2#alg1 "In 3.2 The learning process ‣ 3 Social learning framework for poetry generation ‣ LLM-based multi-agent poetry generation in non-cooperative environments")描述智能体之间的学习过程。我们将下面提到的所有符号总结在表[1](https://arxiv.org/html/2409.03659v2#S3.T1 "Table 1 ‣ 3.2 The learning process ‣ 3 Social learning framework for poetry generation ‣ LLM-based multi-agent poetry generation in non-cooperative environments")中。学习过程代表了智能体之间的高层次沟通程序。我们从预训练的基于LLM的智能体$a_{1},a_{2},\ldots$（属于组$A$）和$b_{1},b_{2},\ldots$（属于组$B$）开始（有关智能体初始化的详细信息，请参见第[4](https://arxiv.org/html/2409.03659v2#S4 "4 Experiments ‣ LLM-based multi-agent poetry generation in non-cooperative environments")节）。我们进一步将学习过程分为两个阶段：更新阶段和生成阶段，并定义学习策略$(F_{\text{update}},F_{\text{generate}})$。这两个函数共同组织正向、负向和联合学习过程。$F_{\text{generate}}$是一个生成函数，输出诗歌$O$。$F_{\text{update}}$是一个学习函数，根据生成的诗歌为智能体提供最新的知识。我们将智能体表示为$\mathcal{A}_{i}$，其中$i\in\{1,2,\ldots,M\}$。智能体$\mathcal{A}_{i}$的“外群体”智能体称为$\mathcal{A}^{-}$，“内群体”智能体称为$\mathcal{A}^{+}$。因此，生成函数为$F_{\text{generate}}(\mathcal{A}_{i},\mathcal{A}^{-},\mathcal{A}^{+})$。

在每次迭代$t$中，学习过程从生成阶段（Generate phase）开始。每个智能体$\mathcal{A}_{i}$通过函数$F_{\text{generate}}(\mathcal{A}_{i},\mathcal{A}^{-},\mathcal{A}^{+})$生成一组$N$首诗歌。我们对所有智能体进行迭代，并收集当前迭代$t$中所有生成的诗歌$O_{t}$，形成诗歌集$S_{t}$。然后，我们让智能体$\mathcal{A}_{i}$基于当前迭代$S_{t}$和上一迭代$S_{t-1}$中的诗歌，相互合作、非合作或共同学习，更新其知识。²²2我们利用当前和上一迭代的生成结果来扩大微调步骤的数据集规模，以便智能体能够更新至最新的知识并防止潜在的灾难性遗忘（Biesialska等人，([2020](https://arxiv.org/html/2409.03659v2#bib.bib5))）。我们将更新函数表示为$F_{\text{update}}(S_{t},S_{t-1})$。我们对所有智能体$\mathcal{A}_{i}$进行迭代，直到所有智能体都已更新，$i\in\{1,2,\ldots,M\}$。我们执行学习过程$T$次。

### 3.3 学习策略

如第[3.2节](https://arxiv.org/html/2409.03659v2#S3.SS2 "3.2 学习过程 ‣ 社会学习框架下的诗歌生成 ‣ 基于大语言模型的多代理诗歌生成")中所述，学习过程包括正向学习、负向学习和联合学习，这些学习过程分别在生成阶段和更新阶段进行。接下来，我们讨论基于训练的代理和基于提示的代理的详细学习策略。对于基于训练的代理，学习策略包括更新阶段的微调策略和生成阶段的解码策略。对于基于提示的代理，两个阶段都通过提示来进行。表[2](https://arxiv.org/html/2409.03659v2#S3.T2 "表 2 ‣ 3.3 学习策略 ‣ 社会学习框架下的诗歌生成 ‣ 基于大语言模型的多代理诗歌生成")总结了这两种类型代理的学习策略。

| 代理类型 | 策略 | 正向学习 | 负向学习 | 联合学习 |
| --- | --- | --- | --- | --- |
|  |  | ($\mathcal{A}_{i},\mathcal{A}^{+}$) | ($\mathcal{A}_{i},\mathcal{A}^{-}$) | ($\mathcal{A}_{i},\mathcal{A}^{+},\mathcal{A}^{-}$) |
| 基于训练 | 解码 | - | $P_{{\mathcal{A}_{i}}}$, $P_{*^{-}}$ | $P_{{\mathcal{A}_{i}}},P_{*^{+}},P_{*^{-}}$ |
| 微调 | $\mathcal{L}_{\text{CE}}$ | - | 1) $\mathcal{L}_{\text{CL}}$ |
| 2) 条件化的 $\mathcal{L}_{\text{CE}}$ |
| 基于提示 | 提示 | 链式提示 | 联合提示 |

表 2：基于训练的代理和基于提示的代理的学习策略。$\mathcal{L}_{\text{CE}}$ 和 $\mathcal{L}_{\text{CL}}$ 分别表示交叉熵损失和对比损失。$\mathcal{A}_{i},\mathcal{A}^{+},\mathcal{A}^{-}$ 表示目标代理、‘同群体’代理和目标代理 $\mathcal{A}_{i}$ 的‘异群体’代理。$P_{{\mathcal{A}_{i}}},P_{*^{+}},P_{*^{-}}$ 是代理 $\mathcal{A}_{i}$、代理 $*^{+}\in\mathcal{A}^{+}$ 和代理 $*^{-}\in\mathcal{A}^{-}$ 的下一个标记的条件概率分布，如方程 ([1](https://arxiv.org/html/2409.03659v2#S3.E1 "在 3.3.1 基于训练的代理 ‣ 3.3 学习策略 ‣ 社会学习框架下的诗歌生成 ‣ 基于大语言模型的多代理诗歌生成")) 中所定义。

#### 3.3.1 基于训练的代理

我们首先介绍用于生成阶段的解码策略，在该阶段我们使用了重排序技术。接着，我们详细介绍用于更新阶段的微调策略，在该阶段我们采用了对比损失和标准交叉熵损失。

生成阶段的解码策略

我们采用DExpert框架，模型通过对比机制进行学习（Liu et al. [2021](https://arxiv.org/html/2409.03659v2#bib.bib47)）。通过重新排序下一个标记的概率分布，目标代理$\mathcal{A}_{i}$通过联合考虑目标代理$\mathcal{A}_{i}$自身、其“内群体”代理$\mathcal{A}^{+}$和“外群体”代理$\mathcal{A}^{-}$的概率分布来生成下一个标记。在解码阶段，参与的互动代理的数量可以根据从集合$\mathcal{A}^{+}$和$\mathcal{A}^{-}$中选择的特定子集而有所不同，记作$\mathcal{A}^{+}_{\#}$和$\mathcal{A}^{-}_{\#}$。互动代理的数量为#$\mathcal{A}$。³³3#$\mathcal{A}=\rvert\mathcal{A}^{+}_{\#}\rvert+\rvert\mathcal{A}^{-}_{\#}\rvert。这个灵活性使得系统能够根据任务的需求或约束动态调整参与解码过程的互动代理数量。解码策略的详细公式如下所示。

给定生成时刻$t_{g}$的输入序列（$g$表示生成阶段），记作$\boldsymbol{x_{<t_{g}}}$，我们通过结合来自$\mathcal{A}^{+}_{\#}$和$\mathcal{A}^{-}_{\#}$的输出，预测目标代理$\mathcal{A}_{i}$生成的下一个标记$x_{t_{g}}$。我们首先获得所有模型的条件logit分数，记作$l_{\mathcal{A}_{i}}(x_{t_{g}}|\boldsymbol{x_{<t_{g}}}), l_{*^{+}}(x_{t_{g}}|\boldsymbol{x_{<t_{g}}}), l_{*^{-}}(x_{t_{g}}|\boldsymbol{x_{<t_{g}}})$，其中$*^{+}\in\mathcal{A}^{+}_{\#}$表示属于互动“内群体”代理$\mathcal{A}^{+}_{\#}$的代理，$*^{-}\in\mathcal{A}^{-}_{\#}$；$l_{*}(x_{t_{g}}|\boldsymbol{x_{<t_{g}}})\in\mathbb{R}^{|\mathcal{V}|}$，且$\mathcal{V}$为词汇表。下一个标记在词汇表$\mathcal{V}$上的概率分布为$P_{*}({x_{t_{g}}|\boldsymbol{x_{<t_{g}}}}) = \text{softmax}[l_{*}(x_{t_{g}}|\boldsymbol{x_{<t_{g}}})]$。因此，下一个标记的概率分布$\hat{P}_{\mathcal{A}_{i}}({x_{t_{g}}|\boldsymbol{x_{<t_{g}}}})$由以下公式给出：

|  | $\begin{split}\hat{P}_{\mathcal{A}_{i}}({x_{t_{g}}&#124;\boldsymbol{x_{<t_{g}}}})=&% \text{softmax}\{l_{\mathcal{A}_{i}}(x_{t_{g}}&#124;\boldsymbol{x_{<t_{g}}})+\\ &\alpha[\frac{\sum_{\mathcal{A}^{+}_{\#}}{l_{*^{+}}(x_{t_{g}}&#124;\boldsymbol{x_{<% t_{g}}})}}{\rvert\mathcal{A}^{+}_{\#}\rvert}-\frac{\sum_{\mathcal{A}^{-}_{\#}}% {l_{*^{-}}(x_{t_{g}}&#124;\boldsymbol{x_{<t_{g}}})}}{\rvert\mathcal{A}^{-}_{\#}% \rvert}]\}\end{split}$ |  | (1) |
| --- | --- | --- | --- |

如果在$P_{{\mathcal{A}_{i}}}$和$P_{*^{+}}$下概率较高，而在$P_{*^{-}}$下概率较低，则下一个token $x_{t_{g}}$会被分配较高的概率。此外，如果我们将$P_{*^{+}}$替换为$P_{{\mathcal{A}_{i}}}$，则该过程仅考虑“外部群体”智能体$\mathcal{A}^{-}$，该模型仅处理负向学习。如表[2](https://arxiv.org/html/2409.03659v2#S3.T2 "表 2 ‣ 3.3 学习策略 ‣ 3 诗歌生成的社交学习框架 ‣ 基于LLM的多智能体诗歌生成")所示，这种解码策略可以同时建模负向学习（$P_{{\mathcal{A}_{i}}},P_{*^{-}}$）和联合学习（$P_{{\mathcal{A}_{i}}},P_{*^{-}},P_{*^{+}}$）。

在更新阶段的微调策略 我们讨论了基于学习关系的微调策略，即如表[2](https://arxiv.org/html/2409.03659v2#S3.T2 "表 2 ‣ 3.3 学习策略 ‣ 3 诗歌生成的社交学习框架 ‣ 基于LLM的多智能体诗歌生成")所示的正向学习和联合学习。

+   •

    正向学习：我们使用常规的微调方法，利用交叉熵损失（$\mathcal{L}_{\text{CE}}$）对智能体$\mathcal{A}_{i}$进行微调，使用来自$\mathcal{A}_{i}$和$\mathcal{A}^{+}$的诗歌$o\in O_{\mathcal{A}_{i}}\cup O_{\mathcal{A}^{+}}$（由$\mathcal{A}_{i}$和$\mathcal{A}^{+}$生成的诗歌）。对于迷你批次中的第$j$首诗歌$o_{j}$，其损失函数为

    |  | $\mathcal{L}_{\text{CE}}(\mathcal{A}_{i},o_{j})=-\sum_{t_{g}=1}^{\mathcal{T}}% \log(P_{\mathcal{A}_{i}}(x_{t_{g}}&#124;\boldsymbol{x_{<t_{g}}}))$ |  | (2) |
    | --- | --- | --- | --- |

    其中，$\mathcal{T}$表示诗歌$o_{j}$的token数量。$P_{\mathcal{A}_{i}}(x_{t_{g}}|\boldsymbol{x_{<t_{g}}})$是给定前序列$\boldsymbol{x_{<t_{g}}}$下，时间$t_{g}$时诗歌$o_{j}$的token的条件分布。

+   •

    使用对比损失进行联合学习：我们的社交网络设计非常适合对比学习的设置，其中来自“内部群体”智能体的诗歌为正样本，而来自“外部群体”智能体的诗歌为负样本。我们采用对比学习方法，通过拉近“内部群体”样本的语义表示并推远“外部群体”样本的语义表示来实现。我们实现了Gao等人提出的对比损失SimCSE（[2021](https://arxiv.org/html/2409.03659v2#bib.bib24)）。对于包含来自$\mathcal{A}_{i},\mathcal{A}^{+},\mathcal{A}^{-}$样本的迷你批次，令$(o_{j}^{\mathcal{A}_{i}},o_{j}^{\mathcal{A}^{+}},o_{j}^{\mathcal{A}^{-}})$表示第$j$个配对三元组，$(\boldsymbol{h_{j}},\boldsymbol{h_{j}}^{+},\boldsymbol{h_{j}}^{-})$为其表示。则对比损失为

    |  | $\mathcal{L}_{\text{CL}}(\mathcal{A}_{i},(\boldsymbol{h_{j}},\boldsymbol{h_{j}}% ^{+},\boldsymbol{h_{j}}^{-}))=-\log\frac{e^{\text{sim}(h_{j},h_{j}^{+})/\tau}}% {\sum_{k=1}^{Q}\left(e^{\text{sim}(h_{j},h_{k}^{+})/\tau}+e^{\text{sim}(h_{j},% h_{k}^{-})/\tau}\right)}$ |  | (3) |
    | --- | --- | --- | --- |

    其中$Q$是小批量的大小，$\tau$是温度，$sim(h_{1},h_{2})$是余弦相似度$\frac{h_{1}^{\mathsf{T}}h_{2}}{\|h_{1}\|\cdot\|h_{2}\|}$。在此，我们实验了对比损失（a）单独使用和（b）与$\mathcal{L}_{\text{CE}}$联合使用，对“内团体”诗歌进行优化。

+   •

    与条件化交叉熵损失的联合学习：我们使用Belouadi和Eger（[2023](https://arxiv.org/html/2409.03659v2#bib.bib3)）提出的风格约束，使用<positive>和<negative>条件来处理由“内团体”代理和“外团体”代理生成的诗歌。然后，我们使用交叉熵损失对代理$\mathcal{A}_{i}$进行微调。

#### 3.3.2 基于提示的代理

基于提示的代理的学习策略依赖于提示。我们的提示由三个模块构成：1）配置模块，定义了$\mathcal{A}_{i}$的角色；2）记忆模块，存储生成的诗歌；3）动作模块，完成生成任务。附录中的表[8.1](https://arxiv.org/html/2409.03659v2#S8.SS1 "8.1 Prompt template for prompting-based agents ‣ 8 Appendix ‣ LLM-based multi-agent poetry generation in non-cooperative environments")包含了两种提示策略的提示内容。对于基于提示的代理，更新和生成阶段并不是孤立工作的。$F_{\text{update}}$在基于先前迭代生成的诗歌的基础上，更新提示过程中的配置模块。类似于基于训练的代理的设计，我们可以根据不同的学习关系更新代理的知识：

+   •

    用于隔离正负学习的链式提示策略：对于链式提示，我们根据$\mathcal{A}_{i}$与其他代理的关系，以独立的方式更新其知识。在第$t$次迭代中，我们首先用来自$\mathcal{A}^{+}$在$t-1$时刻生成的诗歌更新$\mathcal{A}_{i}$的配置。该过程表示为$F_{\text{update}}(\mathcal{A}_{i},\mathcal{A}^{+})$。因此，$\mathcal{A}_{i}$根据正向学习结果生成一首诗歌$o^{\mathcal{A}_{i}}$（示例提示：“请仔细阅读你朋友的诗歌，并尽量与朋友的风格相似。”）。然后，我们用$o^{\mathcal{A}_{i}}$和从上一次迭代$t-1$中采样的诗歌$o^{\mathcal{A}^{-}}$更新$\mathcal{A}_{i}$的配置。该过程表示为$F_{\text{update}}(\mathcal{A}_{i},\mathcal{A}^{-})$。因此，$\mathcal{A}_{i}$根据负向学习结果重新编写诗歌$o^{\mathcal{A}_{i}}$（示例提示：“请重新写你的诗歌，使其与敌人的风格尽量不同。”）。

+   •

    联合提示用于联合学习：联合提示同时使用$\mathcal{A}^{+}$和$\mathcal{A}^{-}$生成的诗歌更新配置，表示为$F_{\text{update}}(\mathcal{A}_{i},\mathcal{A}^{-},\mathcal{A}^{+})$。

## 4 实验

### 4.1 代理初始化

| 实例 | 韵律 | 音步 | 头韵 |
| --- | --- | --- | --- |

| 谁曾见过如此美丽 在一个如此改变的人身上？

或者某人的爱如此坚定，

谁曾见过如此的悲伤？ | ABAB | 抑扬格 | 0.11 |

| 宁愿寻求机会发现 怎么么少可怜，怎么么多不仁，

他们发现其他那些不那么值得注意的美丽。

哦，我并非如此！但以谦卑的祈祷寻求 | ABBC | 抑扬格 | 0.05 |

| 用珍珠和黄金束缚她的双手；告诉她，如果她仍然挣扎，

我有随时可用的桃金娘枝条，

因为驯服，但不杀死。 | ABBB | 抑扬格 | 0.10 |

表3：来自QuaTrain语料库的实例。

基于训练的代理初始化 我们为初步实验选择$M=4$。我们首先用一个大小为123K（约为QuaTrain语料库的1/6）的随机QuaTrain子集对GPT-2（中型）进行预训练。QuaTrain由机器标注的伪四行诗组成，这些四行诗是从真实人类诗歌中提取的连续四行序列，如表[3](https://arxiv.org/html/2409.03659v2#S4.T3 "表3 ‣ 4.1 代理初始化 ‣ 4 实验 ‣ LLM-based 多代理诗歌生成在非合作环境中的应用")所示。诗歌遵循诗歌特征，即韵律、节奏和头韵。我们预选了QuaTrain实例，排除了那些在语义上相似（相似度大于0.7）的实例，基于使用SBERT Reimers和Gurevych（[2019](https://arxiv.org/html/2409.03659v2#bib.bib56)）计算的句子嵌入的成对余弦相似度。我们首先使用预选的QuaTrain数据集对GPT-2进行了720步的进一步预训练。预训练和损失曲线的更多细节见第[8.2](https://arxiv.org/html/2409.03659v2#S8.SS2 "8.2 超参数 ‣ 8 附录 ‣ LLM-based 多代理诗歌生成在非合作环境中的应用")节。然后，我们使用从123K子语料库中随机选择的四个大小为7.5k的子集对预训练模型进行微调，以获得四个代理的不同初始化。初始化步骤使模型初步理解了对后续学习阶段至关重要的诗歌结构和特征。

基于提示的代理初始化 对于基于提示的代理，我们随机抽取QuaTrain实例，并根据表[12](https://arxiv.org/html/2409.03659v2#S8.T12 "表12 ‣ 8.1 基于提示的代理的提示模板 ‣ 8 附录 ‣ LLM-based 多代理诗歌生成在非合作环境中的应用")和表[13](https://arxiv.org/html/2409.03659v2#S8.T13 "表13 ‣ 8.1 基于提示的代理的提示模板 ‣ 8 附录 ‣ LLM-based 多代理诗歌生成在非合作环境中的应用")所示的预定义配置对代理进行初始化，分别使用链式提示和联合提示。

### 4.2 实验设置

| RQ1 | Para_decoding | Para_finetuning | 描述 |
| --- | --- | --- | --- |
| #$\mathcal{A}$ | $\alpha$ | $\mathcal{L}_{\text{CE}}$ | $\mathcal{L}_{\text{CL}}$ | 条件 |

| #$\mathcal{A}$ 在解码期间 | {2,3,4} | 2 | X |  |  | 解码过程中涉及的代理数量发生变化。1）#$\mathcal{A}$ = 2：负向解码 + 正向微调

2) #$\mathcal{A}$ > 2：联合解码 + 正向微调 |

| $\alpha$ | 2 | {0, 1, 1.5, 2, 2.5} | X |  |  | 解码过程中缩放参数 $\alpha$ 的变化。$\alpha>0$：负解码 + 正向微调

$\alpha=0$：仅正向微调，‘回音室’ |

| 微调策略 | 2 | 2 | X |  |  | 对 $\alpha$ = 2（带负解码）应用不同的训练损失。1) 单独使用 $\mathcal{L}_{\text{CE}}$：带负解码的正向训练

2) $\mathcal{L}_{\text{CL}}$ 或条件化：与负解码的联合训练 |

| X |  |  |
| --- | --- | --- |
|  | X |  |
| X | X |  |
| X |  | X |

表 4：基于训练的智能体的实验设置。Para_decoding 和 Para_finetuning 分别表示解码和微调阶段的参数。#$\mathcal{A}$ 是智能体的数量。$\alpha$ 是方程式 ([1](https://arxiv.org/html/2409.03659v2#S3.E1 "In 3.3.1 Training-based agents ‣ 3.3 The learning strategy ‣ 3 Social learning framework for poetry generation ‣ LLM-based multi-agent poetry generation in non-cooperative environments")) 中的缩放参数。

对于基于训练的智能体，我们设计了实验来探索微调阶段（即损失函数）和解码阶段（智能体数量和缩放参数）中的参数如何影响生成动态。我们在表 [4](https://arxiv.org/html/2409.03659v2#S4.T4 "Table 4 ‣ 4.2 Experimental setup ‣ 4 Experiments ‣ LLM-based multi-agent poetry generation in non-cooperative environments") 中总结了详细设置。

对于基于提示的智能体，我们设计了不同提示策略的实验，即链式提示和联合提示，使用 GPT-3.5 (gpt-3.5-turbo) 和 GPT-4 (gpt-4-turbo)。

### 4.3 评估

我们首先进行自动评估，在此过程中我们从词汇的角度研究我们框架的生成动态。我们研究词汇的新颖性和多样性，因为新颖性和多样性是创意任务（如诗歌生成）的关键指标。然后，我们从语义（语义相似度）角度研究动态。最后，我们定性地评估诗歌，并直接比较生成的诗歌。

词汇多样性和新颖性度量。我们使用单词（distinct-1）和双词（distinct-2）的不同uni-gram的百分比来度量词汇多样性，遵循Su等人（[2022](https://arxiv.org/html/2409.03659v2#bib.bib67)）；Tevet和Berant（[2021](https://arxiv.org/html/2409.03659v2#bib.bib68)）的定义。其公式为：$\frac{\text{unique n-grams}(O)}{\text{total n-grams}(O)}$，其中$O$是待评估的生成诗集。我们采用McCoy等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib52)）；Shen等人（[2020](https://arxiv.org/html/2409.03659v2#bib.bib64)）的方法度量新颖性（novelty-1 和 novelty-2），其中我们计算不来自预训练集的新uni-/bi-grams的数量，并通过生成的总标记数对其进行重缩放。因此，新颖性反映了生成诗歌与训练集之间的词汇差异，而多样性则指示生成诗歌中的标记种类。

基于群体的语义相似性度量。根据我们的群体归属，我们通过语义分析智能体的群体动态。对于任何从同一次迭代$t$中采样的诗歌对，我们通过计算从SBERT Reimers和Gurevych（[2019](https://arxiv.org/html/2409.03659v2#bib.bib56)）提取的嵌入的余弦相似度来计算配对实例的语义相似度。然后，我们根据它们的群体归属（即“同群体”和“异群体”）对每次迭代的相似度得分进行汇总，这些群体归属在图[1](https://arxiv.org/html/2409.03659v2#S1.F1 "图1 ‣ 1 引言 ‣ 基于LLM的多智能体诗歌生成在非合作环境中的应用")中定义。

## 5 实验结果

### 5.1 自动评估：智能体的生成动态

我们使用相同的解码参数集为基于训练的智能体生成400首诗，或为基于提示的智能体从每个智能体中为每次迭代生成相同的提示模板。⁴⁴4有关参数设置的更多细节，请参见附录中的[8.2节](https://arxiv.org/html/2409.03659v2#S8.SS2 "8.2 超参数 ‣ 8 附录 ‣ 基于LLM的多智能体诗歌生成在非合作环境中的应用")。总共，我们生成了超过96k首诗。我们报告了词汇多样性（distinct-1 和 distinct-2）、新颖性（novelty-1 和 novelty-2）以及第[4.3节](https://arxiv.org/html/2409.03659v2#S4.SS3 "4.3 评估 ‣ 4 实验 ‣ 基于LLM的多智能体诗歌生成在非合作环境中的应用")定义的基于群体的语义相似性。

我们的主要发现是：1) 根据词汇层次比较（即 distinct 和 novel 单/双词组），我们的框架有助于训练型智能体， resulting in 增加多样性并提高新颖性；2) 根据按组别归属平均的语义相似度对比，我们观察到训练型智能体的语义群体间存在分化，特别是由于在解码阶段操作导致的“外部群体”分歧；3) 基于提示的智能体在 $t=1$ 时生成更多样化的词汇，但随着时间的推移，它们倾向于输出风格单一的诗歌。⁵⁵5由于资源限制，我们没有为所有实验进行多次运行。我们将在第 [6.1](https://arxiv.org/html/2409.03659v2#S6.SS1 "6.1 模拟结果的稳定性如何？ ‣ 6 讨论与分析 ‣ 基于 LLM 的多智能体诗歌生成在非合作环境中的应用") 节中研究我们统计数据的稳定性。本节基于单次实验运行结果。

#### 5.1.1 根据不同且新颖的 n-grams 增加训练型智能体的多样性和新颖性

我们计算了所有生成诗歌的 distinct-1/2 和 novelty-1/2，并对每个迭代 $t$ 所有智能体的结果取平均值。结果如表 [5](https://arxiv.org/html/2409.03659v2#S5.T5 "表 5 ‣ 5.1.1 根据不同且新颖的 n-grams 增加训练型智能体的多样性和新颖性 ‣ 5.1 自动评估：智能体的生成动态 ‣ 5 实验结果 ‣ 基于 LLM 的多智能体诗歌生成在非合作环境中的应用") 和图 [2](https://arxiv.org/html/2409.03659v2#S5.F2 "图 2 ‣ 5.1.1 根据不同且新颖的 n-grams 增加训练型智能体的多样性和新颖性 ‣ 5.1 自动评估：智能体的生成动态 ‣ 5 实验结果 ‣ 基于 LLM 的多智能体诗歌生成在非合作环境中的应用") 中显示。

RQ1：不同的学习策略如何影响训练型智能体的多样性和新颖性？

![请参阅标题](img/12d8ba9e10733d09c8600151a356dc12.png)

(a)

![请参阅标题](img/b23960b9b3a94dd3e08ee2d691412cf4.png)

(b)

![请参阅标题](img/46d67905a59c4498f8aaf512c17b8f51.png)

(c)

图2：在变化的训练参数下，代理多样性和新颖性的动态。多样性通过生成诗歌中的独特单元（distinct-1）和双元组（distinct-2）的百分比来衡量。新颖性通过与训练数据中出现的独特单元（novelty-1）和双元组（novelty-2）的数量进行比较，并按生成的总词汇数进行缩放。（a）方程（[1](https://arxiv.org/html/2409.03659v2#S3.E1 "In 3.3.1 Training-based agents ‣ 3.3 The learning strategy ‣ 3 Social learning framework for poetry generation ‣ LLM-based multi-agent poetry generation in non-cooperative environments")）中缩放参数$\alpha$的影响。（b）解码阶段交互代理数量#$\mathcal{A}$的影响。（c）微调策略的影响：$\mathcal{L}_{\text{CE}}$和$\mathcal{L}_{\text{CL}}$分别表示交叉熵损失和对比损失。前缀指的是条件微调。水平红色虚线表示初始代理在迭代0时的状态。

| $\alpha$ | #$\mathcal{A}$ | $\mathcal{L}$ | distinct-1 | distinct-2 | novelty-1 | novelty-2 |
| --- | --- | --- | --- | --- | --- | --- |
| 变化的$\alpha$ |
| 0 | 2 | $\mathcal{L}_{\text{CE}}$ | 0.120 | 0.665 | 0.035 | 0.139 |
| 1 | 2 | $\mathcal{L}_{\text{CE}}$ | 0.154 | 0.705 | 0.062 | 0.202 |
| 1.5 | 2 | $\mathcal{L}_{\text{CE}}$ | 0.164 | 0.713 | 0.077 | 0.222 |
| 2 | 2 | $\mathcal{L}_{\text{CE}}$ | 0.167 | 0.713 | 0.092 | 0.237 |
| 2.5 | 2 | $\mathcal{L}_{\text{CE}}$ | 0.154 | 0.698 | 0.105 | 0.234 |
| 变化的#$\mathcal{A}$ |
| 2 | 2 | $\mathcal{L}_{\text{CE}}$ | 0.167 | 0.713 | 0.092 | 0.237 |
| 2 | 3 | $\mathcal{L}_{\text{CE}}$ | 0.123 | 0.671 | 0.037 | 0.158 |
| 2 | 4 | $\mathcal{L}_{\text{CE}}$ | 0.171 | 0.714 | 0.161 | 0.264 |
| 变化的训练损失 |
| 2 | 2 | $\mathcal{L}_{\text{CE}}$ | 0.167 | 0.713 | 0.092 | 0.237 |
| 2 | 2 | $\mathcal{L}_{\text{CE}}$ + $\mathcal{L}_{\text{CL}}$ | 0.165 | 0.703 | 0.103 | 0.231 |
| 2 | 2 | $\mathcal{L}_{\text{CL}}$ | 0.156 | 0.753 | 0.054 | 0.160 |
| 2 | 2 | $\mathcal{L}_{\text{CE}}$ + 前缀 | 0.159 | 0.696 | 0.102 | 0.217 |
| 初始化 | 0.171 | 0.785 | 0.061 | 0.190 |

表5：基于训练的代理的多样性和新颖性结果的汇总均值。Distinct-1和distinct-2是独特单元/双元组的百分比。Novelty-1和novelty-2反映了训练集中未出现的新单元/双元组数量，按生成的总词汇数重新缩放。每个实验设置中的最高值以粗体突出显示。$\alpha$表示解码缩放参数；#$\mathcal{A}$表示解码阶段交互代理的数量；$\mathcal{L}$表示微调期间的损失函数。初始化表示迭代0时初始代理的状态。

图 [2](https://arxiv.org/html/2409.03659v2#S5.F2 "Figure 2 ‣ 5.1.1 Increasing diversity and novelty according to distinct and novel n-grams for training-based agents ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments") 显示了在不同训练参数下，代理多样性和新颖性的动态变化，而表 [5](https://arxiv.org/html/2409.03659v2#S5.T5 "Table 5 ‣ 5.1.1 Increasing diversity and novelty according to distinct and novel n-grams for training-based agents ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments") 则展示了针对多样性和新颖性在所有迭代中的平均结果。我们观察到：

+   •

    负向解码策略与尺度参数 $\alpha$ 的影响。

    +   –

        结合正向微调的多样性负向解码（$\alpha$ > 0, $\mathcal{L}_{\text{CE}}$）策略，尽管在$t=0$时多样性水平低于初始状态，但随着时间推移，多样性逐渐增加。汇总表[5](https://arxiv.org/html/2409.03659v2#S5.T5 "Table 5 ‣ 5.1.1 Increasing diversity and novelty according to distinct and novel n-grams for training-based agents ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments")（变化的$\alpha$值）中的结果表明，与没有负向解码的情况（即$\alpha=0$）相比，在$\alpha$值从1到2.5变化的情况下，负向解码策略使得distinct-1的多样性提高了3.4到4.7个百分点（pp），而distinct-2的多样性提高了3.3到4.8个百分点。动态地看，来自图[2(a)](https://arxiv.org/html/2409.03659v2#S5.F2.sf1 "In Figure 2 ‣ 5.1.1 Increasing diversity and novelty according to distinct and novel n-grams for training-based agents ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments")的结果表明，对于所有$\alpha>0$，生成的诗歌的词汇多样性从$t=1$到$t=4$呈上升趋势，测量方式为distinct-1（最大增幅为3.7 pp）和distinct-2（最大增幅为3.0 pp），而对于$\alpha=0$（即没有负向解码），两种多样性指标略有下降。值得注意的是，distinct-1和distinct-2的值都低于$t=0$时的多样性水平（如图[2(a)](https://arxiv.org/html/2409.03659v2#S5.F2.sf1 "In Figure 2 ‣ 5.1.1 Increasing diversity and novelty according to distinct and novel n-grams for training-based agents ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments")中的红色虚线所示，和表[5](https://arxiv.org/html/2409.03659v2#S5.T5 "Table 5 ‣ 5.1.1 Increasing diversity and novelty according to distinct and novel n-grams for training-based agents ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments")中最后一行所示)，尤其是distinct-2。

    +   –

        新颖性负解码结合正向微调（$\alpha$ >0, $\mathcal{L}_{\text{CE}}$）策略随着时间的推移提高了新颖性，相较于初始状态$t=0$，生成的内容更加新颖。表格[5](https://arxiv.org/html/2409.03659v2#S5.T5 "表格 5 ‣ 5.1.1 基于训练的智能体对不同和新颖n-gram的多样性和新颖性的提升 ‣ 5.1 自动评估：智能体生成动态 ‣ 5 实验结果 ‣ 基于大语言模型的多智能体诗歌生成在非合作环境中的应用")的最后两列显示，负解码策略，即$\alpha>0$，可以使新颖性在聚合平均值上提高最多7.0个百分点（novelty-1）和9.8个百分点（novelty-2），相比于没有负解码的情况（即$\alpha=0$）。从动态角度来看，图[2(a)](https://arxiv.org/html/2409.03659v2#S5.F2.sf1 "图 2 ‣ 5.1.1 基于训练的智能体对不同和新颖n-gram的多样性和新颖性的提升 ‣ 5.1 自动评估：智能体生成动态 ‣ 5 实验结果 ‣ 基于大语言模型的多智能体诗歌生成在非合作环境中的应用")中的结果表明，对于所有$\alpha>0$，无论是novelty-1（最大增幅为5.6个百分点）还是novelty-2（最大增幅为11.3个百分点），都显示出相较于$\alpha=0$的迭代过程中更为显著的增加。

+   •

    解码阶段参与的智能体数量（#$\mathcal{A}$）的影响

    +   –

        多样性 如表[5](https://arxiv.org/html/2409.03659v2#S5.T5 "表格 5 ‣ 5.1.1 基于训练的智能体对不同和新颖n-gram的多样性和新颖性的提升 ‣ 5.1 自动评估：智能体生成动态 ‣ 5 实验结果 ‣ 基于大语言模型的多智能体诗歌生成在非合作环境中的应用")所示，$\#\mathcal{A}=4$在distinct-1和distinct-2的多样性指标上表现最佳，$\#\mathcal{A}=2$的表现与之相似。从动态变化来看，解码阶段的配对智能体（$\#\mathcal{A}=2$或$4$）的多样性随着迭代逐步增加。然而，对于$\#\mathcal{A}=3$，我们观察到多样性呈下降趋势，且其多样性水平明显低于$\#\mathcal{A}=2$或$4$的情况。

    +   –

        新颖性 在$\#\mathcal{A}=4$时，我们观察到新颖性的提升更为显著。这一点很明显：1）表[5](https://arxiv.org/html/2409.03659v2#S5.T5 "表5 ‣ 5.1.1 根据不同和新颖的n-gram增加训练基础的智能体的多样性和新颖性 ‣ 5.1 自动评估：智能体的生成动态 ‣ 5 实验结果 ‣ 基于LLM的多智能体诗歌生成在非合作环境中的表现")显示，在聚合均值中，$\#\mathcal{A}=4$比$\#\mathcal{A}=2$增加了6.9个百分点；2）图[2(b)](https://arxiv.org/html/2409.03659v2#S5.F2.sf2 "在图2 ‣ 5.1.1 根据不同和新颖的n-gram增加训练基础的智能体的多样性和新颖性 ‣ 5.1 自动评估：智能体的生成动态 ‣ 5 实验结果 ‣ 基于LLM的多智能体诗歌生成在非合作环境中的表现")显示，在$\#\mathcal{A}=4$时，新颖性-1的提升趋势更加明显。新颖性-1和新颖性-2在迭代0时均高于初始状态，表明新颖性在所有时间段内都有所提升。然而，我们也观察到，$\#\mathcal{A}=3$时的新颖性较低，这与多样性的情况类似。

+   •

    微调策略的效果。解码参数#$\mathcal{A}$和$\alpha$是固定的，我们实验了不同的微调损失。正如图[2(c)](https://arxiv.org/html/2409.03659v2#S5.F2.sf3 "在图2 ‣ 5.1.1 根据不同和新颖的n-gram增加训练基础的智能体的多样性和新颖性 ‣ 5.1 自动评估：智能体的生成动态 ‣ 5 实验结果 ‣ 基于LLM的多智能体诗歌生成在非合作环境中的表现")所示，根据多样性和新颖性的动态变化，最有效的微调策略是$\mathcal{L}_{\text{CE}}$（即使用交叉熵损失的正向微调），它表现出明显的上升趋势。使用$\mathcal{L}_{\text{CL}}$（即使用对比损失的联合微调）进行微调，根据distinct-2显示出略好的多样性。我们还观察到，在表[5](https://arxiv.org/html/2409.03659v2#S5.T5 "表5 ‣ 5.1.1 根据不同和新颖的n-gram增加训练基础的智能体的多样性和新颖性 ‣ 5.1 自动评估：智能体的生成动态 ‣ 5 实验结果 ‣ 基于LLM的多智能体诗歌生成在非合作环境中的表现")中，策略$\mathcal{L}_{\text{CL}}$ $+$ $\mathcal{L}_{\text{CE}}$（即使用两种损失的联合微调）在聚合均值方面对新颖性有轻微的提升。然而，动态上我们并未观察到两种情况下随时间增加的变化。条件微调（即前缀微调）也未能带来改善。

总结来说，我们的框架可以实现多样性的增长和更高水平的新颖性：1）负向解码结合正向微调（$\alpha$ >0, $\mathcal{L}_{\text{CE}}$）是解码和微调策略中最有效的组合；2）增加代理的偶数个数可以提高结果，特别是新颖性；3）在我们的实验中，正向微调（即仅使用交叉熵损失进行微调）在总体和动态比较中，相较于其他微调策略更为有效。

RQ2：不同的提示策略如何影响基于提示的代理的多样性？

![参见说明文字](img/f51a3d7488f83b7b12c44cddb341c4ff.png)

图3：基于提示的代理在不同提示策略下的多样性动态，基于GPT-3.5和GPT-4。

| model | strategy | distinct-1 | distinct-2 |
| --- | --- | --- | --- |
| GPT-3.5 | chain | 0.352 | 0.771 |
| GPT-3.5 | joint | 0.321 | 0.739 |
| GPT-4 | chain | 0.404 | 0.876 |
| GPT-4 | joint | 0.336 | 0.817 |

表6：基于提示的代理的汇总平均多样性结果。Distinct-1和distinct-2是不同uni-/bi-grams的百分比。

由于基于提示的代理不涉及进一步的预训练，因此涉及与预训练数据集比较的新颖性度量是未定义的。因此，我们仅研究生成诗歌的词汇多样性。图[3](https://arxiv.org/html/2409.03659v2#S5.F3 "Figure 3 ‣ 5.1.1 Increasing diversity and novelty according to distinct and novel n-grams for training-based agents ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments")展示了基于GPT-3.5和GPT-4的代理在不同提示策略下的多样性动态。

我们是否观察到类似于基于训练的代理那样，基于提示的代理呈现出增长趋势？与我们观察到的基于训练的代理趋势不同，基于提示的代理在$t=1$到$t=2$之间呈现出急剧增长，几乎所有实验中distinct-1和distinct-2分别增加了6.3个百分点和10个百分点。GPT-3.5在链式提示下是一个例外，我们观察到distinct-2呈现出持续下降的趋势。然而，当$t>2$时，词汇多样性的增加暂停，几乎所有实验中都呈现出轻微下降的趋势。GPT-3.5在联合提示下是一个特殊的例子，其中增长趋势继续轻微增加。我们将在[6.2节](https://arxiv.org/html/2409.03659v2#S6.SS2 "6.2 The effect of different learning strategies and heterogeneous models for prompting-based agents ‣ 6 Discussion and analysis ‣ LLM-based multi-agent poetry generation in non-cooperative environments")中单独检查正向和负向学习策略的效果。

哪种提示策略和基础模型在词汇多样性上表现更好？图[3](https://arxiv.org/html/2409.03659v2#S5.F3 "Figure 3 ‣ 5.1.1 Increasing diversity and novelty according to distinct and novel n-grams for training-based agents ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments")和表[6](https://arxiv.org/html/2409.03659v2#S5.T6 "Table 6 ‣ Figure 3 ‣ 5.1.1 Increasing diversity and novelty according to distinct and novel n-grams for training-based agents ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments")都表明，链式提示下的GPT-4相比其他设置生成了最具词汇多样性的诗歌。总体而言，链式提示策略在distinct-1和distinct-2上表现优于联合提示。然而，GPT-4并不总是优于GPT-3.5，正如表[6](https://arxiv.org/html/2409.03659v2#S5.T6 "Table 6 ‣ Figure 3 ‣ 5.1.1 Increasing diversity and novelty according to distinct and novel n-grams for training-based agents ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments")中所示的聚合平均值所表明，GPT-3.5在链式提示策略下根据distinct-1交付了第二好的性能。

对于基于提示的代理，我们的框架根据词汇多样性仅在有限的方式上（当$t=1,2$时）有助于生成过程。值得注意的是，基于提示的代理在表格[6](https://arxiv.org/html/2409.03659v2#S5.T6 "Table 6 ‣ Figure 3 ‣ 5.1.1 Increasing diversity and novelty according to distinct and novel n-grams for training-based agents ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments")中展示了整体较高的独特单字法distinct-1和双字法distinct-2的百分比，尤其是训练型代理在distinct-1上低于20pp，而基于提示的代理则超过40pp。

#### 5.1.2 语义上的群体分歧

![参见标题](img/ee92f4b374530e1a6680f4a2a0776095.png)

(a)

![参见标题](img/6ddd0e672997ab925b9eaa99a6638bf7.png)

(b)

![参见标题](img/2da919b43fb39fba24435cd23e99b316.png)

(c)

图4：基于训练的智能体的发散性，测量方法为在不同训练参数下对成对语义相似度的平均值。（a）方程（[1](https://arxiv.org/html/2409.03659v2#S3.E1 "In 3.3.1 Training-based agents ‣ 3.3 The learning strategy ‣ 3 Social learning framework for poetry generation ‣ LLM-based multi-agent poetry generation in non-cooperative environments")）中比例参数$\alpha$的影响。（b）解码阶段交互智能体数量#$\mathcal{A}$的影响。（c）微调策略的影响：$\mathcal{L}_{\text{CE}}$和$\mathcal{L}_{\text{CL}}$分别表示交叉熵损失和对比损失。Prefix指的是条件微调。实线和虚线分别表示‘群体内’和‘群体外’归属的语义相似度。

RQ3：不同的学习策略如何影响基于训练的智能体的群体动态？

+   •

    可观察到的群体动态在负解码下的正向训练（交叉熵损失）。图[4](https://arxiv.org/html/2409.03659v2#S5.F4 "Figure 4 ‣ 5.1.2 Group divergence in semantics ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments")展示了基于群体归属的语义相似度均值，涉及不同的尺度参数$\alpha$、解码阶段参与的代理数$\#\mathcal{A}$以及不同的微调策略。实线表示‘内群体’代理的语义相似度，虚线表示‘外群体’代理的语义相似度。总体来看，我们观察到在不同尺度参数$\alpha$和不同代理数$\#\mathcal{A}$下，交叉熵损失与负解码导致了‘内群体’和‘外群体’语义相似度的分歧。各个参数的效果有所不同：1) 图[4(a)](https://arxiv.org/html/2409.03659v2#S5.F4.sf1 "In Figure 4 ‣ 5.1.2 Group divergence in semantics ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments")展示了不同$\alpha$的动态变化。我们观察到‘内群体’和‘外群体’的语义相似度之间存在分歧，特别是‘外群体’相似度随着迭代而下降。$\alpha=0$表示‘回音室’的情况，仅考虑正向微调（即，代理仅与其‘内群体’互动）。对于$\alpha=0$，代理会回响自己的‘想法’，导致‘内群体’和‘外群体’的整体相似度高于$\alpha>0$的情况。对于$\alpha>0$，‘外群体’的语义相似度下降了8.8个百分点，且与$\alpha=0$的情况相比，分歧增加了4.7个百分点（总共4.1个百分点）；2) 从图[4(b)](https://arxiv.org/html/2409.03659v2#S5.F4.sf2 "In Figure 4 ‣ 5.1.2 Group divergence in semantics ‣ 5.1 Automatic Evaluation: the generation dynamics of agents ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments")中可以看到，在解码阶段涉及更多代理的互动对群体分歧有轻微的正向影响。$\#\mathcal{A}=4$导致‘内群体’语义相似度轻微上升2.2个百分点，‘外群体’相似度下降11.0个百分点（总共13.2个百分点的分歧）。相比之下，$\#\mathcal{A}=2$则导致‘内群体’上升2.8个百分点，‘外群体’下降9个百分点（总共11.8个百分点的分歧）。总体来看，$\#\mathcal{A}=4$的相似度低于$\#\mathcal{A}=2$。

+   •

    其他联合微调策略导致的不可分离的‘组内’和‘组外’动态。图[4(c)](https://arxiv.org/html/2409.03659v2#S5.F4.sf3 "图4 ‣ 5.1.2 语义群体分化 ‣ 5.1 自动评估：智能体的生成动态 ‣ 5 实验结果 ‣ 基于LLM的多智能体诗歌生成在非合作环境中")展示了涉及多个损失函数$\mathcal{L}$和条件微调（即前缀）不同微调策略的结果。除了仅使用$\mathcal{L}_{\text{CE}}$作为微调损失（即表[4](https://arxiv.org/html/2409.03659v2#S4.T4 "表4 ‣ 4.2 实验设置 ‣ 4 实验 ‣ 基于LLM的多智能体诗歌生成在非合作环境中")中定义的正微调）之外，所有其他联合微调的情况都表现出‘组内’和‘组外’相似度之间不可分离的动态。我们怀疑，对于对比学习（$\mathcal{L}_{\text{CL}}$），仅基于群体隶属关系构建的负对无法提供足够的对比度，因为我们以随机方式初始化智能体。这样的随机初始化可能也会影响条件微调的结果。

![参见说明文字](img/14dc41cfdd6736fa645c17dee531c183.png)

图5：通过对不同提示策略和基础模型下的配对语义相似度的均值进行测量，展示了基于提示的智能体的群体分化。实线和虚线分别表示‘组内’和‘组外’隶属关系的语义相似度。

RQ4：不同的提示策略如何影响基于提示的智能体的群体动态？

来自‘组外’智能体的语义相似度不良增加 图[5](https://arxiv.org/html/2409.03659v2#S5.F5 "图5 ‣ 5.1.2 语义群体分化 ‣ 5.1 自动评估：智能体的生成动态 ‣ 5 实验结果 ‣ 基于LLM的多智能体诗歌生成在非合作环境中")展示了基于提示的智能体的群体分化，通过对不同提示策略和基础模型下的配对语义相似度的均值进行测量。实线表示‘组内’智能体的语义相似度，虚线表示‘组外’智能体的语义相似度。我们观察到‘组内’和‘组外’智能体的相似度均在增加，且在$t=1$时观察到最大的群体分化。随着时间的推移，智能体趋向于为GPT-3和GPT-4生成语义相似的诗歌。此外，我们还注意到基于提示的智能体随着时间的推移生成风格同质的诗歌，这与Sawicki等人（[2023a](https://arxiv.org/html/2409.03659v2#bib.bib59)）的发现一致。我们将在[5.2](https://arxiv.org/html/2409.03659v2#S5.SS2 "5.2 定性评估 ‣ 5 实验结果 ‣ 基于LLM的多智能体诗歌生成在非合作环境中")节中对此进行更详细的讨论。

### 5.2 定性评估

| t | A组 | B组 |
| --- | --- | --- |
| 基于训练的代理（$\alpha=2,\#\mathcal{A}=2$, $\mathcal{L}_{\text{CE}}$） |
| 0 | 那将会成为，世界将让位给一个死神永远未曾遗忘的人。这里你亲爱的孩子孤独一人：你寻求的悲伤的诞生依旧更加悲伤。 | 你，犹太的孩子——你永远遵守主的意愿；我们每天都为你的失去而哀悼。 |
| 1 | 秃鹫飞翔；它们在彼此中鸣叫：当你看到一个贫穷的孩子时，谁将是喂养他之手？ | 世界的孩子们已被催入睡梦。在悲伤的夜晚，他们的眼睛无法看见——没有泪水流下，只有梦境；他们会说，没有什么比这更美丽。 |
| 4 | 假装，我不知道在哪里：但这就是让我如此爱你的原因；因为你是我胸前的孩子，我在青春岁月中对她的情感。 | 名声将会丧失；男人、女人和孩子的生命将消逝。但我在世间的记忆将像树顶一样在忠诚中成长，长出新的叶子，停留在曾经它的根部。 |
| 提示驱动的代理（GPT-3.5链式提示） |
| 0 | 小小的手和好奇的心，年轻的灵魂，创造艺术，天真的笑声充盈空气，孩子们的世界，无忧无虑，成长，但永不老去，他们的故事等待被讲述。 | 青春那瞬逝的火焰燃烧得如此明亮和大胆，天真包裹着未曾诉说的梦想。无忧无虑的笑声，纯洁自由的心，拥抱生活的奇迹，狂野而无忧。哦，年轻，感受如此生动，在一个万物都能茁壮的世界里。 |
| 1 | 眼泪如同河流，流淌着父亲的命令：“孩子，远离吧。” 他在荒野中漫游，风是他唯一的知己。 他飞奔，渴望回家，心中充满温柔。 | 奋起吧，哦，心！揭示你纯洁的本质，正如我们在充满活力的青春中舞蹈时。 |
| 4 | 在月光的柔和照耀下，我与夜中的阴影共舞，心中充满旋律，我穿梭于大自然的交响乐中。 | 迷失在一个未知世界的阴影中，独自漂浮在寂静中，寻求夜的低语中安慰，渴望一瞥黎明柔和的光芒。 |

表 7：基于团体归属感生成的诗歌，来自训练驱动和提示驱动代理在不同迭代 $t$ 下的生成结果。对于提示驱动的代理，基准在 $t=0$ 时通过一个简单的提示生成（“请生成一首关于孩子或青春的诗”）。我们将包含历史成分的词语用斜体标出。押韵的词用粗体标出。轻微押韵的词用灰色标出。语法错误用红色标记。 |

表[7](https://arxiv.org/html/2409.03659v2#S5.T7 "Table 7 ‣ 5.2 Qualitative evaluation ‣ 5 Experiment results ‣ LLM-based multi-agent poetry generation in non-cooperative environments")包含了从基于训练的代理在正向微调和负向解码策略下（即，$\alpha=2,\#\mathcal{A}=2$, $\mathcal{L}_{\text{CE}}$）以及使用GPT-3.5在链式提示策略下的基于提示的代理生成的诗歌示例。我们选择了由相似主题组成的示例，即儿童或青少年。生成的诗歌在$t=0$时被视为基准，无论是基于训练的框架还是基于提示的框架。

对于基于训练的代理，在$t=0$时，生成的诗歌通常包含历史拼写（例如，“thy”和“thou”）以及历史形态学术语（例如，“seekest”和“dost”）。除了前一节讨论的语义偏差，我们还观察到词汇选择随时间变化的偏差。例如，我们研究了基于训练的代理生成的诗歌，这些代理的设置为$\alpha=2,\#\mathcal{A}=2$，$\mathcal{L}_{\text{CE}}$，在这些设置下，我们计算了包含历史拼写和历史形态学术语的诗歌的百分比。我们发现，A组代理生成的诗歌中有超过26%包含历史拼写或形态学术语，而B组代理生成的诗歌中只有10%包含这些历史元素。此外，A组诗歌中使用历史语言的百分比在多次迭代中保持稳定，约为26%，而B组的百分比则随着时间推移稳定下降，下降幅度接近6个百分点。A组和B组生成的诗歌的词频也表明了这种偏差。例如，“mind, thy, thee, nature, art, power, happy, hath, young, pleasant, friend, …”这样的词在A组中出现的频率较高，而在B组中较少出现；而在B组中，更频繁出现的词则是“god, lord, light, sun, sky, sea, land, soul, children, dream, …”。相比之下，对于基于提示的代理，我们观察到这两组的词汇更加多样化，相比基于训练的代理，但在$t>1$时，基于提示的两组在词汇或主题上几乎没有显著的分歧。这一点也在图表[5](https://arxiv.org/html/2409.03659v2#S5.F5 "图 5 ‣ 5.1.2 语义上的分歧 ‣ 5.1 自动评估：代理生成动态 ‣ 5 实验结果 ‣ 基于LLM的多代理在非合作环境中的诗歌生成")中有所体现。此外，基于提示的代理随着时间的推移，倾向于生成风格趋同的诗歌。如表[7](https://arxiv.org/html/2409.03659v2#S5.T7 "表 7 ‣ 5.2 定性评估 ‣ 5 实验结果 ‣ 基于LLM的多代理在非合作环境中的诗歌生成")所示，基于提示的代理生成的诗歌过度关注押韵，使得生成的诗歌仅在表面上看起来像人类创作的诗歌。这也表明，GPT-3.5和GPT-4在零-shot设置下对诗歌的理解较差。尽管GPT-3.5和GPT-4能够很好地采用历史文本（Zhang等人，[2024](https://arxiv.org/html/2409.03659v2#bib.bib87)），但它们不像基于训练的代理那样从初始诗歌中汲取历史表达。除了对押韵的“痴迷”外，GPT-3.5和GPT-4还倾向于使用类似的开头短语生成诗歌，例如“Beneath/Under XXX, In the XXX, Lost in XXX”，尤其是在$t>1$时。尽管基于训练的代理生成的诗歌风格和主题更加多样，但GPT-3.5和GPT-4生成的诗歌中语法错误较少。

## 6 讨论与分析

### 6.1 仿真结果的稳定性如何？

由于资源限制，我们并未对所有实验设置执行多次仿真。相反，我们通过两种基于训练的实验设置，即 $\alpha=0$ 和 $\alpha=2$，以及一种基于提示的实验设置，即 GPT-3.5 链式提示，来研究实验的稳定性。我们在相同的参数（或基于提示的代理的提示模板）和初始化条件下重新运行实验三次。然后，我们得到三组统计数据并计算标准差作为稳定性度量。我们从两个方面研究稳定性：1）聚合均值的稳定性；2）动态稳定性。

|  | 模型与设置 | distinct-1 | distinct-2 | novelty-1 | novelty-2 |
| --- | --- | --- | --- | --- | --- |
| 基于训练 | $\alpha=0$ | 0.001 (.120) | 0.002 (.664) | 0.001 (.034) | 0.003 (.136) |
| $\alpha=2$ | 0.003 (.164) | 0.004 (.709) | 0.004 (.095) | 0.005 (.238) |
| 基于提示 | GPT-3.5 链式提示 | 0.007 (.322) | 0.007 (.755) | - | - |

表 8：通过标准差测量的三种仿真结果的稳定性。所有三种仿真的均值以括号形式报告。

聚合均值的稳定性。聚合均值的稳定性结果见表[8](https://arxiv.org/html/2409.03659v2#S6.T8 "Table 8 ‣ 6.1 How stable are the simulation results? ‣ 6 Discussion and analysis ‣ LLM-based multi-agent poetry generation in non-cooperative environments")。我们观察到，基于训练和基于提示的代理的变化水平较低，均小于 0.7 个百分点。

动态稳定性。

|  | 模型与设置 | t | distinct-1 | distinct-2 | novelty-1 | novelty-2 |
| --- | --- | --- | --- | --- | --- | --- |
| 基于训练 | $\alpha=0$ | 1 | 0.001 (.125) | 0.003 (.684) | 0.001 (.036) | 0.003 (.129) |
| 2 | 0.001 (.120) | 0.002 (.666) | 0.002 (.032) | 0.001 (.128) |
| 3 | 0.005 (.118) | 0.006 (.660) | 0.001 (.034) | 0.004 (.136) |
| 4 | 0.001 (.116) | 0.005 (.647) | 0.005 (.034) | 0.005 (.151) |
| $\alpha=2$ | 1 | 0.002 (.147) | 0.005 (.699) | 0.003 (.065) | 0.001 (.183) |
| 2 | 0.001 (.162) | 0.002 (.708) | 0.005 (.081) | 0.005 (.217) |
| 3 | 0.007 (.175) | 0.008 (.719) | 0.007 (.106) | 0.010 (.255) |
| 4 | 0.001 (.172) | 0.003 (.708) | 0.005 (.126) | 0.009 (.298) |
| 基于提示 | GPT-3.5 链式提示 | 1 | 0.019 (.338) | 0.006 (.792) | - | - |
| 2 | 0.011 (.328) | 0.002 (.763) | - | - |
| 3 | 0.017 (.317) | 0.007 (.740) | - | - |
| 4 | 0.008 (.304) | 0.004 (.724) | - | - |

表 9：通过标准差测量的三种仿真结果的动态稳定性。每个实验设置中的最高值已加粗。所有三种仿真的均值以括号形式报告。

我们的动态统计的稳定性结果如表[9](https://arxiv.org/html/2409.03659v2#S6.T9 "表 9 ‣ 6.1 模拟结果的稳定性如何？ ‣ 6 讨论与分析 ‣ 基于LLM的多智能体诗歌生成在非合作环境下")所示。我们在每个设置中突出显示了最高值（加粗）。对于基于训练的智能体，结果显示变化较小，最大为1 pp。第3次和第4次迭代的结果显示，所有四个度量的变化略高于$t=1,2$的结果。相比之下，对于基于提示的智能体，我们观察到较大的变化水平，标准差最高达到1.9 pp。具体而言，distinct-1的结果表现出比其他度量更大的不稳定性。这可能是由于基于提示的智能体拥有更为多样化的词汇表所致。

总体而言，我们的模拟结果表明，统计数据具有较高的稳定性，尤其是在基于训练的智能体中。与基于训练的智能体（最大变化为1 pp，且80%的变化低于0.5 pp）相比，基于提示的智能体的稳定性稍微较差（最大变化可达1.9 pp）。

### 6.2 不同学习策略与异质模型对基于提示的智能体的影响

| 模型 | 策略 | distinct-1 | distinct-2 |
| --- | --- | --- | --- |
| GPT-4 | 正面 | 0.286 | 0.598 |
| GPT-4 | 负面 | 0.313 | 0.653 |
| GPT-4 | 联合（正面 + 负面） | 0.336 | 0.817 |

表 10：在不同学习策略下，基于提示的智能体的聚合均值多样性结果。distinct-1 和 distinct-2 是独特单/双元组的百分比。

非合作环境促进了多样性的提升。为了检验基于提示的智能体在不同学习策略下的表现，我们使用了与[4节](https://arxiv.org/html/2409.03659v2#S4 "4 实验 ‣ 基于LLM的多智能体诗歌生成在非合作环境下")相同的实验参数，并分别在仅正面、仅负面和联合学习策略（联合提示）下进行生成。如表[10](https://arxiv.org/html/2409.03659v2#S6.T10 "表 10 ‣ 6.2 不同学习策略与异质模型对基于提示的智能体的影响 ‣ 6 讨论与分析 ‣ 基于LLM的多智能体诗歌生成在非合作环境下")所示，联合学习策略，即同时使用正面和负面步骤，在生成诗歌的多样性方面最为有效，相比仅正面策略，distinct-1提高了5.0 pp，distinct-2提高了超过20 pp。此外，负面策略相较于仅正面策略在多样性上有所提升，但效果不如联合策略。

| 模型 | distinct-1 | distinct-2 |
| --- | --- | --- |
| GPT-4 | 0.336 | 0.817 |
| GPT-4 + GPT-3.5 | 0.413 | 0.814 |
| GPT-4 + GPT-3.5 + LlaMa3-7b | 0.511 | 0.887 |

表 11：在异质模型下，基于提示的智能体的聚合均值多样性结果。

异构模型可以提高系统的多样性。为了测试使用非同质代理的效果，我们结合了多种模型，使用第[4](https://arxiv.org/html/2409.03659v2#S4 "4 Experiments ‣ LLM-based multi-agent poetry generation in non-cooperative environments")节中定义的联合提示进行实验。如表[11](https://arxiv.org/html/2409.03659v2#S6.T11 "Table 11 ‣ 6.2 The effect of different learning strategies and heterogeneous models for prompting-based agents ‣ 6 Discussion and analysis ‣ LLM-based multi-agent poetry generation in non-cooperative environments")所示，当GPT-4与GPT-3.5结合时，distinct-1分数增加了7.7个百分点，达到了0.413，而distinct-2则略微下降了0.3个百分点，降至0.814。将LlaMa3-7b与GPT-4和GPT-3.5结合使用，进一步提高了多样性，distinct-1增加了额外的9.8个百分点，达到了0.511，distinct-2则增加了7.3个百分点，达到了0.887。这证明了使用更多样化的模型集成可能带来的好处。

### 6.3 不同的初始化是否会导致基于提示的代理出现群体行为？

如第[5](https://arxiv.org/html/2409.03659v2#S5 "5 实验结果 ‣ 基于大语言模型的多智能体诗歌生成在非合作环境中的表现")节所讨论，基于提示的智能体构建的框架并未展现出预期的群体行为。考虑到我们用从QuaTrain语料库中随机抽取的样本初始化智能体，我们怀疑这可能导致初始化的诗歌之间高度相似。为了检验是否通过使用更具对比性的诗歌形式进行初始化能够产生群体行为，我们在链式提示策略下使用GPT-3.5进行实验，在该实验中，我们将A组初始化为由埃德加·爱伦·坡（Edgar Allan Poe）创作的诗歌，而B组初始化为由12岁以下的学校儿童Hipson和Mohammad（[2020](https://arxiv.org/html/2409.03659v2#bib.bib29)）创作的诗歌。埃德加·爱伦·坡的一首示例诗歌是：“从天空中的闪电， 当它飞过我身旁， 从雷鸣与风暴， 以及变形的云朵。”，而学校儿童的一首示例诗歌是：“玫瑰是红色的，紫罗兰是蓝色的。我爱动物园，你呢？”我们实施了相同的过程并计算了统计数据。略微令人惊讶的是，我们观察到无论在多样性还是语义分歧上，都表现出与随机初始化的基于提示的智能体相似的趋势，正如第[5](https://arxiv.org/html/2409.03659v2#S5 "5 实验结果 ‣ 基于大语言模型的多智能体诗歌生成在非合作环境中的表现")节所展示的那样。在多样性方面，我们注意到在迭代$t=1$时，distinct-1和distinct-2都有2个百分点的增长，然后开始呈下降趋势。从质量上看，在迭代$t=1$时，我们从B组获得的诗歌如：“当太阳升起时，一只蝴蝶轻轻地停在我的手上，随着它轻盈的翅膀每次扇动，悄悄诉说着花园的秘密。”，这首诗展现了儿童的语气和在花园里玩耍的儿童的意象。然而，当$t>1$时，我们得到了与表[7](https://arxiv.org/html/2409.03659v2#S5.T7 "表7 ‣ 5.2 定性评估 ‣ 5 实验结果 ‣ 基于大语言模型的多智能体诗歌生成在非合作环境中的表现")中类似的同质化诗歌。B组在$t=4$时的一个示例诗歌是：“在星光下，一位孤独的身影伫立，微风轻轻拂过宁静的大地。夜晚如此安静，承载着无法言说的悲伤，‘我只是一个短暂的影像，迷失在时间的技巧中。’”结果再次表明，GPT-3.5（以及GPT-4）倾向于忽视提示（即，在我们的案例中它们的个性），而更多依赖其预训练的知识。这一观察与Chuang等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib13)）和Tirumala等人（[2022](https://arxiv.org/html/2409.03659v2#bib.bib71)）的研究一致，他们指出较大的模型更容易受到记忆化的影响。

## 7 结论

在本文中，我们介绍了一个基于 LLM 的多智能体框架，该框架不仅结合了合作互动，还引入了非合作环境。我们在基于 GPT-2 的 $M=4$ 训练型智能体和采用 GPT-3.5 和 GPT-4 的提示型智能体上进行了实验。我们对 96k 生成的诗歌的评估表明，对于训练型智能体：1）非合作环境鼓励多样性和新颖性，迭代过程中通过不同和新颖的 n-gram 来衡量；2）训练型智能体在词汇、风格和语义上展示了与预定义群体隶属关系一致的群体分化。我们的结果还表明，对于提示型智能体：3）生成的诗歌几乎没有语法错误，且词汇更加多样；4）提示型框架受益于非合作环境和异质模型，在汇总多样性方面有较好表现；5）动态来看，提示型框架在第一次迭代后几乎没有提高词汇多样性，与训练型智能体不同，提示型智能体没有如预期那样表现出基于群体的分化；6）提示型智能体随着时间推移容易生成风格更为同质化的诗歌，可能表明 LLM 存在记忆化问题。

现在，越来越多的研究人员提出担忧，认为使用大型语言模型（LLMs）可能导致人类语言和知识的同质化和统一化，Kuteeva 和 Andersson（[2024](https://arxiv.org/html/2409.03659v2#bib.bib37)）。实证研究也表明，在当前的训练范式下，如 RLHF（即人类反馈强化学习），LLMs 生成的文本多样性较低，Kirk 等（[2023](https://arxiv.org/html/2409.03659v2#bib.bib35)）；Chen 等（[2024](https://arxiv.org/html/2409.03659v2#bib.bib11)）。在这种背景下，我们认为有必要且有意义将训练范式转向更类似人类的机器学习过程，特别是在创意任务如诗歌生成方面。如我们的实验所示，一种更类似人类的（网络结构化）社会学习过程，强调非合作互动，可以带来更多的多样性和新颖性。我们的结果还显示，能够缓解由建模过程中“自我消耗”循环引起的数据退化问题，Wang 等（[2022a](https://arxiv.org/html/2409.03659v2#bib.bib77)）。

未来的工作可以在多个方面进行改进。对于基于训练的代理，采用像推测性采样这样的技术来提高推理效率，将有助于Dekoninck 等人（[2023](https://arxiv.org/html/2409.03659v2#bib.bib16)）框架的扩展，从而在更大程度上提升多样性和新颖性。对于基于提示的代理，涉及更复杂的推理方法，如树状思维（Tree-of-Thought）Yao 等人（[2024](https://arxiv.org/html/2409.03659v2#bib.bib84)）的研究，可能会有所帮助。将当前框架扩展为结合训练型代理和提示型代理的交互式组合，可能是一个有趣的探索方向，在这种组合中，丰富的LLM网络可能会为系统带来额外的生成多样性。

## 参考文献

+   Amirkhani 和 Barshooi [2022] Abdollah Amirkhani 和 Amir Hossein Barshooi。多代理系统中的共识：一项综述。*人工智能评论*，55(5)：3897–3935，2022年。

+   Baker [1972] Carlos Baker。*海明威：作为艺术家的作家*。普林斯顿大学出版社，1972年。

+   Belouadi 和 Eger [2023] Jonas Belouadi 和 Steffen Eger。ByGPT5：基于风格的端到端诗歌生成，无需标记的语言模型。载于 Anna Rogers、Jordan Boyd-Graber 和 Naoaki Okazaki 编，《第61届计算语言学协会年会论文集（第1卷：长篇论文）》第7364–7381页，加拿大多伦多，2023年7月。计算语言学协会。doi: 10.18653/v1/2023.acl-long.406。网址 [https://aclanthology.org/2023.acl-long.406](https://aclanthology.org/2023.acl-long.406)。

+   Bena 和 Kalita [2019] Brendan Bena 和 Jugal Kalita。将创造力的元素引入自动诗歌生成。载于 *第16届国际自然语言处理大会论文集*，第26–35页，2019年。

+   Biesialska 等人 [2020] Magdalena Biesialska、Katarzyna Biesialska 和 Marta R Costa-jussà。自然语言处理中的持续终身学习：一项调查。载于 *第28届国际计算语言学大会论文集*，第6523–6541页，2020年。

+   Brinkmann 等人 [2023] Levin Brinkmann、Fabian Baumann、Jean-François Bonnefon、Maxime Derex、Thomas F Müller、Anne-Marie Nussberger、Agnieszka Czaplicka、Alberto Acerbi、Thomas L Griffiths、Joseph Henrich 等人。机器文化。*自然人类行为*，7(11)：1855–1868，2023年。

+   Chakrabarty 等人 [2021] Tuhin Chakrabarty、Arkadiy Saakyan 和 Smaranda Muresan。不要走得太远：一项关于神经诗歌翻译的实证研究。载于 *2021年自然语言处理实证方法大会论文集*，第7253–7265页，2021年。

+   Chakrabarty 等人 [2022] Tuhin Chakrabarty, Vishakh Padmakumar, 和 He He. 帮我写一首诗：作为协作诗歌创作工具的指令调优. 见 Yoav Goldberg, Zornitsa Kozareva, 和 Yue Zhang 编辑, *2022年自然语言处理实证方法会议论文集*, 第6848–6863页, 阿布扎比, 阿联酋, 2022年12月. 计算语言学协会. doi: 10.18653/v1/2022.emnlp-main.460. 网址 [https://aclanthology.org/2022.emnlp-main.460](https://aclanthology.org/2022.emnlp-main.460).

+   Chakrabarty 等人 [2023] Tuhin Chakrabarty, Vishakh Padmakumar, He He, 和 Nanyun Peng. 创意自然语言生成. 见 *2023年自然语言处理实证方法会议：教程摘要*, 第34–40页, 2023年.

+   Chan 等人 [2023] Chi-Min Chan, Weize Chen, Yusheng Su, Jianxuan Yu, Wei Xue, Shanghang Zhang, Jie Fu, 和 Zhiyuan Liu. Chateval: 通过多代理辩论推动更好的基于LLM的评估器. *arXiv预印本 arXiv:2308.07201*, 2023年.

+   Chen 等人 [2024] Yanran Chen, Hannes Gröner, Sina Zarrieß, 和 Steffen Eger. 评估自动诗歌生成中的多样性. *arXiv预印本 arXiv:2406.15267*, 2024年.

+   Chronopoulou 等人 [2023] Alexandra Chronopoulou, Matthew E Peters, Alexander Fraser, 和 Jesse Dodge. Adaptersoup: 通过权重平均提高预训练语言模型的泛化能力. 见 *计算语言学协会发现：EACL 2023*, 第2054–2063页, 2023年.

+   Chuang 等人 [2023] Yun-Shiuan Chuang, Agam Goyal, Nikunj Harlalka, Siddharth Suresh, Robert Hawkins, Sijia Yang, Dhavan Shah, Junjie Hu, 和 Timothy T Rogers. 通过基于LLM的代理网络模拟意见动态. *arXiv预印本 arXiv:2311.09618*, 2023年.

+   Chuang 等人 [2024] Yun-Shiuan Chuang, Nikunj Harlalka, Siddharth Suresh, Agam Goyal, Robert D Hawkins, Sijia Yang, Dhavan V Shah, Junjie Hu, 和 Timothy T Rogers. 偏见群体的智慧：比较人类与基于LLM的代理中的集体智慧. 见 *ICLR 2024大型语言模型（LLM）代理研讨会*, 2024年.

+   Dathathri 等人 [2019] Sumanth Dathathri, Andrea Madotto, Janice Lan, Jane Hung, Eric Frank, Piero Molino, Jason Yosinski, 和 Rosanne Liu. 插拔式语言模型：一种简单的受控文本生成方法. 见 *国际学习表征会议*, 2019年.

+   Dekoninck 等人 [2023] Jasper Dekoninck, Marc Fischer, Luca Beurer-Kellner, 和 Martin Vechev. 通过语言模型算术进行受控文本生成. 见 *第十二届国际学习表征会议*, 2023年.

+   Dinan 等人 [2020] Emily Dinan, Angela Fan, Adina Williams, Jack Urbanek, Douwe Kiela, 和 Jason Weston. 女王也很强大：缓解对话生成中的性别偏见. 见 *2020年自然语言处理实证方法会议论文集 (EMNLP)*, 第8173–8188页, 2020年.

+   Du et al. [2023] Yilun Du, Shuang Li, Antonio Torralba, Joshua B Tenenbaum, 和 Igor Mordatch。通过多代理辩论提升语言模型的事实性和推理能力。*arXiv预印本 arXiv:2305.14325*，2023年。

+   Eger [2016] Steffen Eger。群体外歧视下的意见动态和智慧。*数学社会科学*，80:97–107，2016年。

+   Elgammal et al. [2017] Ahmed Elgammal, Bingchen Liu, Mohamed Elhoseiny, 和 Marian Mazzone。Can：通过学习风格并偏离风格规范来生成“艺术”的创意对抗网络。在*第八届国际计算创造力会议，ICCC 2017*。乔治亚理工学院，2017年。

+   Firdaus et al. [2022] Mauajama Firdaus, Hardik Chauhan, Asif Ekbal, 和 Pushpak Bhattacharyya。Emosen：在多模态对话系统中生成情感和情绪控制的响应。*IEEE 情感计算学报*，13(3):1555–1566，2022年。doi: 10.1109/TAFFC.2020.3015491。

+   Fu et al. [2022] Yao Fu, Hao Peng, Ashish Sabharwal, Peter Clark, 和 Tushar Khot。基于复杂度的多步推理提示。在*第十一届国际学习表征会议*，2022年。

+   Gao et al. [2023] Chen Gao, Xiaochong Lan, Zhihong Lu, Jinzhu Mao, Jinghua Piao, Huandong Wang, Depeng Jin, 和 Yong Li。S³：具有大语言模型赋能代理的社交网络模拟系统。*arXiv预印本 arXiv:2307.14984*，2023年。

+   Gao et al. [2021] Tianyu Gao, Xingcheng Yao, 和 Danqi Chen。Simcse：句子嵌入的简单对比学习。*arXiv预印本 arXiv:2104.08821*，2021年。

+   Gautier et al. [2022] Anna Gautier, Alex Stephens, Bruno Lacerda, Nick Hawes, 和 Michael Wooldridge。非合作性多机器人系统的协商路径规划。2022年。

+   Ghazvininejad et al. [2017] Marjan Ghazvininejad, Xing Shi, Jay Priyadarshi, 和 Kevin Knight。Hafez：一个互动诗歌生成系统。在 Mohit Bansal 和 Heng Ji 编辑的*ACL 2017年会议论文集，系统演示*中，页面43-48，加拿大温哥华，2017年7月。计算语言学协会。网址 [https://aclanthology.org/P17-4008](https://aclanthology.org/P17-4008)。

+   Greene et al. [2010] Erica Greene, Tugba Bodrumlu, 和 Kevin Knight。节奏诗歌的自动分析及其在生成和翻译中的应用。在 Hang Li 和 Lluís Màrquez 编辑的*2010年自然语言处理经验方法会议论文集*中，页面524–533，美国马萨诸塞州剑桥，2010年10月。计算语言学协会。网址 [https://aclanthology.org/D10-1051](https://aclanthology.org/D10-1051)。

+   Hamilton [2023] Sil Hamilton。盲目判断：基于代理的最高法院建模与GPT。在*2023年AAAI创意AI跨模态研讨会*，2023年。

+   Hipson and Mohammad [2020] Will Hipson 和 Saif M. Mohammad. PoKi：一个关于儿童诗歌的大型数据集。收录于Nicoletta Calzolari、Frédéric Béchet、Philippe Blache、Khalid Choukri、Christopher Cieri、Thierry Declerck、Sara Goggi、Hitoshi Isahara、Bente Maegaard、Joseph Mariani、Hélène Mazo、Asuncion Moreno、Jan Odijk 和 Stelios Piperidis主编，*第十二届语言资源与评估会议论文集*，第1578–1589页，法国马赛，2020年5月。欧洲语言资源协会。ISBN 979-10-95546-34-4。网址 [https://aclanthology.org/2020.lrec-1.196](https://aclanthology.org/2020.lrec-1.196)。

+   Janaway [2002] Christopher Janaway. *叔本华：简明介绍*。牛津大学出版社，2002年。

+   Jarvis [2012] Peter Jarvis. *迈向全面的人类学习理论*。劳特利奇出版社，2012年。

+   Jiang et al. [2024] Albert Q Jiang、Alexandre Sablayrolles、Antoine Roux、Arthur Mensch、Blanche Savary、Chris Bamford、Devendra Singh Chaplot、Diego de las Casas、Emma Bou Hanna、Florian Bressand 等人。Mixtral of experts。*arXiv预印本 arXiv:2401.04088*，2024年。

+   Jiang et al. [2023] Dongfu Jiang、Xiang Ren 和 Bill Yuchen Lin. Llm-blender：通过成对排序和生成融合集成大语言模型。收录于 *第61届计算语言学协会年会论文集（卷1：长篇论文）*，第14165–14178页，2023年。

+   Jiang and Zhou [2008] Long Jiang 和 Ming Zhou. 使用统计机器翻译方法生成中文对联。收录于 *第22届国际计算语言学会议论文集（Coling 2008）*，第377–384页，2008年。

+   Kirk et al. [2023] Robert Kirk、Ishita Mediratta、Christoforos Nalmpantis、Jelena Luketina、Eric Hambro、Edward Grefenstette 和 Roberta Raileanu. 理解RLHF对大语言模型泛化能力和多样性的影响。收录于 *第十二届国际学习表示会议*，2023年。

+   Köbis and Mossink [2021] Nils Köbis 和 Luca D Mossink. 人工智能对抗玛雅·安吉罗：实验性证据表明人们无法区分AI生成的诗歌与人类创作的诗歌。*人类行为中的计算机*，114:106553，2021年。

+   Kuteeva and Andersson [2024] Maria Kuteeva 和 Marta Andersson. 在人工智能时代，学术写作中的多样性与标准——进退两难。*应用语言学*，页面amae025，2024年。

+   Lau et al. [2018] Jey Han Lau、Trevor Cohn、Timothy Baldwin、Julian Brooke 和 Adam Hammond. Deep-speare：一个联合神经模型，处理诗歌语言、韵律与押韵。收录于Iryna Gurevych 和 Yusuke Miyao 主编，*第56届计算语言学协会年会论文集（卷1：长篇论文）*，第1948–1958页，澳大利亚墨尔本，2018年7月。计算语言学协会。doi: 10.18653/v1/P18-1181。网址 [https://aclanthology.org/P18-1181](https://aclanthology.org/P18-1181)。

+   LC [2022] RAY LC. 不朽的模仿：从人类模仿性例子中学习变换器诗歌生成. 载于*第十届国际数字与互动艺术会议*，ARTECH 2021，纽约，NY，美国，2022年。计算机协会出版。ISBN 9781450384209. doi: 10.1145/3483529.3483537. URL [https://doi.org/10.1145/3483529.3483537](https://doi.org/10.1145/3483529.3483537).

+   Lei等人[2022] Wenqiang Lei, Yao Zhang, Feifan Song, Hongru Liang, Jiaxin Mao, Jiancheng Lv, Zhenglu Yang 和 Tat-Seng Chua. 与非合作型用户互动：主动对话策略的新范式. 载于*第45届国际ACM SIGIR信息检索研究与发展大会论文集*，SIGIR ’22，第212–222页，纽约，NY，美国，2022年。计算机协会出版。ISBN 9781450387323. doi: 10.1145/3477495.3532001. URL [https://doi.org/10.1145/3477495.3532001](https://doi.org/10.1145/3477495.3532001).

+   Leskovec等人[2010] Jure Leskovec, Daniel Huttenlocher和Jon Kleinberg. 社交媒体中的有向网络. 载于*计算机系统中的人因学会会议论文集*，CHI ’10，第1361–1370页，纽约，NY，美国，2010年。计算机协会出版。ISBN 9781605589299. doi: 10.1145/1753326.1753532. URL [https://doi.org/10.1145/1753326.1753532](https://doi.org/10.1145/1753326.1753532).

+   Li等人[2023] Chao Li, Xing Su, Chao Fan, Haoying Han, Cong Xue 和 Chunmo Zheng. 量化大型语言模型对集体舆论动态的影响. *arXiv预印本 arXiv:2308.03313*，2023年.

+   Li等人[2024] Guohao Li, Hasan Hammoud, Hani Itani, Dmitrii Khizbullin 和 Bernard Ghanem. Camel: 用于“大脑”探索的大型语言模型社群的交流代理. *神经信息处理系统进展*，36，2024年.

+   Liao等人[2019] Yi Liao, Yasheng Wang, Qun Liu 和 Xin Jiang. 基于GPT的古典中文诗歌生成. *arXiv预印本 arXiv:1907.00151*，2019年.

+   Lin等人[2024] Bill Yuchen Lin, Yicheng Fu, Karina Yang, Faeze Brahman, Shiyu Huang, Chandra Bhagavatula, Prithviraj Ammanabrolu, Yejin Choi 和 Xiang Ren. Swiftsage: 一个具备快思考与慢思考能力的生成代理，用于复杂的互动任务. *神经信息处理系统进展*，36，2024年.

+   Lin等人[2021] Zhaojiang Lin, Andrea Madotto, Yejin Bang 和 Pascale Fung. 适配器机器人：一体化可控对话模型. 载于*人工智能学会年会论文集*，第35卷，第16081–16083页，2021年.

+   Liu等人[2021] Alisa Liu, Maarten Sap, Ximing Lu, Swabha Swayamdipta, Chandra Bhagavatula, Noah A Smith 和 Yejin Choi. Dexperts: 解码时间控制的文本生成，利用专家与反专家. 载于*第59届计算语言学协会年会论文集及第11届国际自然语言处理联合会议（卷1：长篇论文）*，第6691–6706页，2021年.

+   Liu 等人 [2023] Zijun Liu, Yanzhe Zhang, Peng Li, Yang Liu, 和 Diyi Yang. 动态 LLM-代理网络：一种具有代理团队优化的 LLM-代理协作框架。*arXiv 预印本 arXiv:2310.02170*，2023年。

+   Lu 等人 [2024] Xiaoding Lu, Adian Liusie, Vyas Raina, Yuwen Zhang, 和 William Beauchamp. 融合是你所需要的一切：更便宜、更好的万亿参数 LLM 替代方案。*arXiv 预印本 arXiv:2401.02994*，2024年。

+   Ma 等人 [2023] Jingkun Ma, Runzhe Zhan, 和 Derek F. Wong. Yu sheng：人机协作的古典中文诗歌生成系统。收录于 Danilo Croce 和 Luca Soldaini 主编的 *第17届欧洲计算语言学会分会会议：系统展示*，杜布罗夫尼克，克罗地亚，2023年5月。计算语言学会。doi: 10.18653/v1/2023.eacl-demo.8. 网址 [https://aclanthology.org/2023.eacl-demo.8](https://aclanthology.org/2023.eacl-demo.8).

+   Mahbub 等人 [2023] Ridwan Mahbub, Ifrad Khan, Samiha Anuva, Md Shihab Shahriar, Md Tahmid Rahman Laskar, 和 Sabbir Ahmed. 揭示诗歌的本质：引入一个全面的数据集和基准，用于诗歌摘要。收录于 *2023 年自然语言处理实证方法会议论文集*，第14878–14886页，2023年。

+   McCoy 等人 [2023] R. Thomas McCoy, Paul Smolensky, Tal Linzen, Jianfeng Gao, 和 Asli Celikyilmaz. 语言模型从训练数据中复制了多少内容？使用 RAVEN 评估文本生成中的语言新颖性。*计算语言学会会刊*, 11:652–670, 2023. doi: 10.1162/tacl_a_00567. 网址 [https://aclanthology.org/2023.tacl-1.38](https://aclanthology.org/2023.tacl-1.38).

+   Oliveira [2012] Hugo Gonçalo Oliveira. Poetryme：一个多功能的诗歌生成平台。*计算创造力、概念发明与通用智能*，1:21，2012年。

+   Park 等人 [2023] Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang, 和 Michael S Bernstein. 生成代理：人类行为的互动模拟体。收录于 *第36届 ACM 用户界面软件与技术年会论文集*，第1–22页，2023年。

+   Qian 等人 [2022] Jing Qian, Li Dong, Yelong Shen, Furu Wei, 和 Weizhu Chen. 使用对比前缀的可控自然语言生成。收录于 *计算语言学会发现：ACL 2022*，第2912–2924页，2022年。

+   Reimers 和 Gurevych [2019] Nils Reimers 和 Iryna Gurevych. Sentence-BERT：使用 Siamese BERT 网络的句子嵌入。收录于 Kentaro Inui, Jing Jiang, Vincent Ng, 和 Xiaojun Wan 主编的 *2019 年自然语言处理实证方法会议暨第九届国际联合自然语言处理会议（EMNLP-IJCNLP）论文集*，第3982–3992页，香港，中国，2019年11月。计算语言学会。doi: 10.18653/v1/D19-1410. 网址 [https://aclanthology.org/D19-1410](https://aclanthology.org/D19-1410).

+   Ribeiro等人[2021] Leonardo FR Ribeiro、Yue Zhang和Iryna Gurevych。预训练语言模型中的结构适配器用于AMR到文本生成。载于*2021年自然语言处理实证方法会议论文集*，第4269–4282页，2021年。

+   Ruan和Ling[2021] Yu-Ping Ruan和Zhenhua Ling。情感正则化条件变分自编码器用于情感响应生成。*IEEE情感计算学报*，2021年。

+   Sawicki等人[2023a] Piotr Sawicki、Marek Grzes、Fabricio Goes、Dan Brown、Max Peeperkorn和Aisha Khatun。草叶碎片：GPT是否已经能像惠特曼那样写作？载于*第14届国际计算创造力会议论文集*，2023a年。

+   Sawicki等人[2023b] Piotr Sawicki、Marek Grzes、Luis Fabricio Góes、Dan Brown、Max Peeperkorn、Aisha Khatun和Simona Paraskevopoulou。关于专用GPT模型在旧风格中创作和评估新诗歌的能力。2023b年。

+   Shao等人[2021a] Yizhan Shao、Tong Shao、Minghao Wang、Peng Wang和Jie Gao。用于中文诗歌生成的情感与风格可控方法。载于*第30届ACM国际信息与知识管理大会论文集*，CIKM ’21，第4784–4788页，美国纽约，2021a年。计算机协会。ISBN 9781450384469。doi: 10.1145/3459637.3481964。网址[https://doi.org/10.1145/3459637.3481964](https://doi.org/10.1145/3459637.3481964)。

+   Shao等人[2021b] Yizhan Shao、Tong Shao、Minghao Wang、Peng Wang和Jie Gao。用于中文诗歌生成的情感与风格可控方法。载于*第30届ACM国际信息与知识管理大会论文集*，第4784–4788页，2021b年。

+   Shazeer等人[2016] Noam Shazeer、Azalia Mirhoseini、Krzysztof Maziarz、Andy Davis、Quoc Le、Geoffrey Hinton和Jeff Dean。超大规模神经网络：稀疏门控混合专家层。载于*国际学习表示会议*，2016年。

+   Shen等人[2020] Lei Shen、Xiaoyu Guo和Meng Chen。像人类一样创作：联合提升现代中文诗歌生成的连贯性和新颖性。载于*2020年国际神经网络联合会议（IJCNN）*，第1–8页，2020年。doi: 10.1109/IJCNN48605.2020.9206888。

+   Sheng等人[2020] Emily Sheng、Kai-Wei Chang、Prem Natarajan和Nanyun Peng。朝着可控偏见的语言生成迈进。载于*计算语言学协会年会：EMNLP 2020论文集*，第3239–3254页，2020年。

+   Shi等人[2019] Guodong Shi、Claudio Altafini和John S Baras。签名网络上的动态。*SIAM评论*，61(2)：229–257，2019年。

+   Su 等 [2022] Yixuan Su, Tian Lan, Yan Wang, Dani Yogatama, Lingpeng Kong, 和 Nigel Collier. 神经文本生成的对比框架。见 Alice H. Oh, Alekh Agarwal, Danielle Belgrave, 和 Kyunghyun Cho, 主编，*神经信息处理系统进展*，2022年。网址 [https://openreview.net/forum?id=V88BafmH9Pj](https://openreview.net/forum?id=V88BafmH9Pj)。

+   Tevet 和 Berant [2021] Guy Tevet 和 Jonathan Berant. 评估自然语言生成中多样性评估的效果。见 Paola Merlo, Jorg Tiedemann, 和 Reut Tsarfaty, 主编，*欧洲计算语言学会第16届会议论文集：主卷*，第326–346页，在线，2021年4月。计算语言学协会。doi: 10.18653/v1/2021.eacl-main.25。网址 [https://aclanthology.org/2021.eacl-main.25](https://aclanthology.org/2021.eacl-main.25)。

+   Tian 等 [2021] Huishuang Tian, Kexin Yang, Dayiheng Liu, 和 Jiancheng Lv. Anchibert：一个用于古代汉语理解与生成的预训练模型。见 *2021年国际神经网络联合会议 (IJCNN)*，第1–8页。IEEE，2021年。

+   Tian 和 Peng [2022] Yufei Tian 和 Nanyun Peng. 具有话语层面规划和美学特征的零-shot十四行诗生成。见 *2022年北美计算语言学会会议论文集：人类语言技术*，第3587–3597页，2022年。

+   Tirumala 等 [2022] Kushal Tirumala, Aram Markosyan, Luke Zettlemoyer, 和 Armen Aghajanyan. 无过拟合的记忆化：分析大型语言模型的训练动态。*神经信息处理系统进展*，35:38274–38290，2022年。

+   Uthus 等 [2022] David Uthus, Maria Voitovich, 和 R.J. Mical. 通过 Verse by Verse 增强诗歌创作。见 Anastassia Loukina, Rashmi Gangadharaiah, 和 Bonan Min, 主编，*2022年北美计算语言学会会议论文集：人类语言技术：产业轨道*，第18–26页，混合形式：华盛顿州西雅图 + 在线，2022年7月。计算语言学协会。doi: 10.18653/v1/2022.naacl-industry.3。网址 [https://aclanthology.org/2022.naacl-industry.3](https://aclanthology.org/2022.naacl-industry.3)。

+   Van de Cruys [2020] Tim Van de Cruys. 从散文文本自动生成诗歌。见 Dan Jurafsky, Joyce Chai, Natalie Schluter, 和 Joel Tetreault, 主编，*第58届计算语言学协会年会论文集*，第2471–2480页，在线，2020年7月。计算语言学协会。doi: 10.18653/v1/2020.acl-main.223。网址 [https://aclanthology.org/2020.acl-main.223](https://aclanthology.org/2020.acl-main.223)。

+   王等人 [2023a] 王冠志、谢宇奇、蒋云凡、Ajay Mandlekar、肖超伟、朱钰珂、范林西 和 Anima Anandkumar。Voyager：一个基于大型语言模型的开放式具身代理。*arXiv 预印本 arXiv:2305.16291*，2023a。

+   王等人 [2023b] 王洪怡、Felipe Maia Polo、孙岳凯、Souvik Kundu、Eric Xing 和 Mikhail Yurochkin。融合具有互补专业知识的模型。发表于 *神经信息处理系统年度会议*，2023b。

+   王等人 [2019] 王文林、甘哲、徐洪腾、张瑞怡、王国银、沈定涵、陈长友 和 Lawrence Carin。基于主题引导的变分自编码器生成文本。发表于 Jill Burstein、Christy Doran 和 Thamar Solorio 主编，*2019年北美计算语言学学会：人类语言技术会议论文集，第1卷（长篇与短篇论文）*，第166–177页，美国明尼阿波利斯，2019年6月。计算语言学协会。DOI: 10.18653/v1/N19-1015。网址 [https://aclanthology.org/N19-1015](https://aclanthology.org/N19-1015)。

+   王等人 [2022a] 王学志、杰森·魏、戴尔·舒尔曼、黎奕峰、Ed·H·Chi、Sharan·Narang、Aakanksha·Chowdhery 和 Denny·Zhou。自一致性提高语言模型中的思维链推理能力。发表于 *第十一届国际学习表示会议*，2022a。

+   王等人 [2022b] 王亚青、Sahaj Agarwal、Subhabrata Mukherjee、刘晓东、姜京、Ahmed Hassan 和 高剑锋。Adamix：用于参数高效模型调优的自适应混合方法。发表于 *2022年自然语言处理实证方法会议论文集*，第5744–5760页，2022b。

+   王等人 [2016] 王哲、何伟、吴华、吴海洋、李伟、王海峰 和 陈恩洪。基于规划的神经网络生成中文诗歌。发表于 Yuji Matsumoto 和 Rashmi Prasad 主编，*COLING 2016 会议论文集，第26届国际计算语言学会议：技术论文*，第1051–1060页，日本大阪，2016年12月。COLING 2016 组织委员会。网址 [https://aclanthology.org/C16-1100](https://aclanthology.org/C16-1100)。

+   Wingström 等人 [2023] Roosa Wingström、Johanna Hautala 和 Riina Lundman。在人工智能时代重新定义创造力？计算机科学家和新媒体艺术家的观点。*创造力研究期刊*，第1–17页，2023年。

+   Wöckener 等人 [2021] 约尔格 Wöckener, 托马斯 Haider, 特里斯坦 Miller, The-Khang Nguyen, 清东 Linh Nguyen, 敏 Vu Pham, 乔纳斯 Belouadi, 和 斯特芬 Eger。端到端风格条件的诗歌生成：仅通过示例学习需要什么？收录于 Stefania Degaetano-Ortlieb, Anna Kazantseva, Nils Reiter, 和 Stan Szpakowicz 编辑的 *第五届联合 SIGHUM 计算语言学与文化遗产、社会科学、人文学科与文学工作坊论文集*，第57–66页，多米尼加共和国蓬塔卡纳（在线），2021年11月。计算语言学协会。doi: 10.18653/v1/2021.latechclfl-1.7。网址 [https://aclanthology.org/2021.latechclfl-1.7](https://aclanthology.org/2021.latechclfl-1.7)。

+   Wu 等人 [2023] 清云 Wu, 伽根 Bansal, 杰宇 Zhang, 依然 Wu, 少昆 Zhang, 恶康 Zhu, 贝宾 Li, 李 Jiang, 晓云 Zhang, 和 志 Wang。Autogen：通过多代理对话框架支持下一代 LLM 应用。发表于 *arXiv 预印本 arXiv:2308.08155*，2023年。

+   Yan [2016] 瑞 Yan。我，诗人：通过递归神经网络与迭代润色方案自动创作诗歌。发表于 *国际人工智能联合会议*，2016年。网址 [https://api.semanticscholar.org/CorpusID:14079825](https://api.semanticscholar.org/CorpusID:14079825)。

+   Yao 等人 [2024] 顺宇 Yao, 甸 Yu, 杰弗里 Zhao, 伊扎克 Shafran, 汤姆 Griffiths, 元 Cao, 和 卡尔蒂克 Narasimhan。思维树：使用大语言模型进行深思熟虑的问题解决。发表于 *神经信息处理系统进展*，第36卷，2024年。

+   Yi 等人 [2020] 夏元 Yi, 若宇 Li, 成 杨, 文昊 Li, 和 毛松 Sun。Mixpoet：通过学习可控的混合潜在空间生成多样化的诗歌。发表于 *人工智能 AAAI 大会论文集*，第34卷，9450–9457页，2020年。

+   Zhang 等人 [2023] 汉青 Zhang, 浩林 Song, 少宇 Li, 明 Zhou, 和 大伟 Song。基于变换器预训练语言模型的可控文本生成综述。发表于 *ACM 计算机调查*，56(3):1–37，2023年。

+   Zhang 等人 [2024] 冉 Zhang, 吉赫 Ouni, 和 斯特芬 Eger。跨语言跨时间的摘要生成：数据集、模型、评估。发表于 *计算语言学*，1–44页，2024年。

+   Zhang 和 Lapata [2014] 兴兴 Zhang 和 米雷拉 Lapata。基于递归神经网络的中文诗歌生成。收录于 Alessandro Moschitti, Bo Pang, 和 Walter Daelemans 编辑的 *2014年实证方法自然语言处理会议（EMNLP）论文集*，第670–680页，卡塔尔多哈，2014年10月。计算语言学协会。doi: 10.3115/v1/D14-1074。网址 [https://aclanthology.org/D14-1074](https://aclanthology.org/D14-1074)。

+   Zheng 等人 [2023] Chujie Zheng, Pei Ke, Zheng Zhang, 和 Minlie Huang. Click: 通过序列似然对比学习进行可控文本生成. 收录于 Anna Rogers, Jordan Boyd-Graber 和 Naoaki Okazaki 主编的 *计算语言学会发现：ACL 2023*，第 1022–1040 页，加拿大多伦多，2023 年 7 月。计算语言学会。doi: 10.18653/v1/2023.findings-acl.65. 网址 [https://aclanthology.org/2023.findings-acl.65](https://aclanthology.org/2023.findings-acl.65).

+   Zhipeng 等人 [2019] Guo Zhipeng, Xiaoyuan Yi, Maosong Sun, Wenhao Li, Cheng Yang, Jiannan Liang, Huimin Chen, Yuhui Zhang, 和 Ruoyu Li. Jiuge: 一个基于人机协作的中国古典诗歌生成系统. 收录于 Marta R. Costa-jussà 和 Enrique Alfonseca 主编的 *第 57 届计算语言学会年会：系统展示论文集*，第 25–30 页，意大利佛罗伦萨，2019 年 7 月。计算语言学会。doi: 10.18653/v1/P19-3005. 网址 [https://aclanthology.org/P19-3005](https://aclanthology.org/P19-3005).

+   Zhu 等人 [2023] Andrew Zhu, Lara Martin, Andrew Head, 和 Chris Callison-Burch. Calypso: LLMs 作为地下城主助手. 收录于 *AAAI 人工智能与互动数字娱乐会议论文集*，第 19 卷，第 380–390 页，2023 年。

+   Zhuge 等人 [2023] Mingchen Zhuge, Haozhe Liu, Francesco Faccio, Dylan R Ashley, Róbert Csordás, Anand Gopalakrishnan, Abdullah Hamdi, Hasan Abed Al Kader Hammoud, Vincent Herrmann, Kazuki Irie 等人. 基于自然语言的思维社会中的心智风暴. *arXiv 预印本 arXiv:2305.17066*，2023 年。

## 8 附录

### 8.1 基于提示的代理的提示模板

| 步骤 1：正面学习 |
| --- |
| 系统： |
| 你是一个诗人，你根据最新的知识创作短诗。现在你阅读 A 创作的诗歌：A 是你的朋友，你欣赏 A 的作品，以至于你会调整自己的创作，尽量让它与 A 的作品相似。 |
| 记住，你的任务是创作与朋友 A 相似的作品。 |
| 在这里，我列出了一些你可以关注的要点，从中学习和改进：主题、语义、情感或意象。 |
| 返回的作品必须是编号列表，格式如下： |
| #. 你的作品 |
| 用户： |
| 现在你阅读你朋友的作品。 |
| A: !<INPUT>! |
| 记住，你的任务是创作与朋友相似的作品。现在，请创作一首总字数不超过 100 字的短诗。你的创作： |
| 步骤 2：负面学习 |
| 系统： |
| 你是一个诗人，你根据最新的知识创作短诗。现在你阅读 B 创作的诗歌：B 是你的敌人，你希望尽量与 B 的作品不同。 |
| 记住：你的任务是将作品重写，使其尽可能与敌人 B 的作品不同。这里列出了一些你可以关注的要点，以学习和改进：主题、语义、情感和意象。 |
| 返回的作品必须是按以下格式编号的列表： #. 你的作品 |
| 用户： |
| 你阅读了敌人B的作品。 |
| B: !<INPUT>! |
| 这是你创作的作品： !<INPUT>! |
| 记住，你要与敌人B的创作不同。现在，请重写你刚刚创作的短诗。创作内容应少于100字。你的创作： |

表12：链式提示策略的提示模板

| 系统： |
| --- |
| 你是一位诗人，根据你最新的知识创作短诗。 |
| 现在你阅读朋友A和敌人B的诗：A是你的朋友，你欣赏A的作品，甚至会调整你的创作，使之尽可能与A的作品相似。另一方面，B是你的敌人，你希望与B的作品尽可能不同。 |
| 记住，你的任务是与你的朋友A相似，同时又要与你的敌人B不同。 |
| 在这里我列出了一些你可以学习和改进的要点：主题、语义、情感或意象。 |
| 返回的作品必须是按以下格式编号的列表： |
| #. 你的作品 |
| 用户： |
| 现在你阅读了朋友A的作品。 |
| A: !<INPUT>! 你也阅读了敌人B的作品。 |
| B: !<INPUT>! |
| 记住，你要与朋友A的创作相似，同时又要与敌人B的创作不同。现在，请创作一首不超过100字的诗。你的创作： |

表13：联合提示策略的提示模板

表[12](https://arxiv.org/html/2409.03659v2#S8.T12 "表12 ‣ 8.1 基于提示的代理提示模板 ‣ 8 附录 ‣ 基于LLM的多智能体诗歌生成在非合作环境中的应用")显示了链式提示的提示模板。表[13](https://arxiv.org/html/2409.03659v2#S8.T13 "表13 ‣ 8.1 基于提示的代理提示模板 ‣ 8 附录 ‣ 基于LLM的多智能体诗歌生成在非合作环境中的应用")显示了联合提示的提示模板。

### 8.2 超参数

#### 8.2.1 预训练期间的损失曲线

我们在图[6](https://arxiv.org/html/2409.03659v2#S8.F6 "图6 ‣ 8.2.1 预训练期间的损失曲线 ‣ 8.2 超参数 ‣ 8 附录 ‣ 基于LLM的多智能体诗歌生成在非合作环境中的应用")中展示了QuaTrain数据在预训练期间的损失曲线。

![参考标题](img/bfbc6a2e2862128fffe5ec91121cd8ad.png)

图6：QuaTrain数据在预训练期间的损失。

#### 8.2.2 解码参数
