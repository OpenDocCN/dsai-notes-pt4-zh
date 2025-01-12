<!--yml

类别：未分类

日期：2025-01-11 12:46:11

-->

# LLMs 的决策能力如何？评估 LLMs 在多智能体环境中的博弈能力

> 来源：[https://arxiv.org/html/2403.11807/](https://arxiv.org/html/2403.11807/)

Jen-tse Huang^(1,2)，Eric John Li¹，Man Ho Lam¹，Tian Liang^(4,2)，Wenxuan Wang^(1,2)

Youliang Yuan^(3,2)，Wenxiang Jiao²，Xing Wang²，Zhaopeng Tu²，Michael R. Lyu¹

¹香港中文大学     ²腾讯 AI 实验室

³香港中文大学（深圳）     ⁴清华大学 {jthuang,wxwang,lyu}@cse.cuhk.edu.hk {ejli,mhlam}@link.cuhk.edu.hk liangt21@mails.tsinghua.edu.cn youliangyuan@link.cuhk.edu.cn  {joelwxjiao,brightxwang,zptu}@tencent.com Jiao Wenxiang 为通讯作者。

###### 摘要

决策是一个复杂的过程，涉及多种能力，使其成为评估大型语言模型（LLMs）的优秀框架。研究人员通过博弈论的视角研究了 LLMs 的决策能力。然而，现有的评估主要集中在两人博弈场景，其中 LLM 与另一方进行竞争。此外，之前的基准测试由于其静态设计存在测试集泄露的问题。我们提出了 GAMA($\gamma$)-Bench，这是一个新的框架，用于评估 LLMs 在多智能体环境中的博弈能力。它包括八个经典的博弈论场景，并采用专门设计的动态评分方案来定量评估 LLMs 的表现。$\gamma$-Bench 允许灵活的游戏设置，并根据不同的游戏参数调整评分系统，从而实现对 LLMs 在鲁棒性、泛化能力以及改进策略方面的全面评估。我们的结果表明，GPT-3.5 展现出强大的鲁棒性，但泛化能力有限，可以通过诸如 Chain-of-Thought 等方法得到增强。我们还评估了来自六个模型家族的十二个 LLMs，包括 GPT-3.5、GPT-4、Gemini、LLaMA-3.1、Mixtral 和 Qwen-2。Gemini-1.5-Pro 超过了其他模型，得分为 $68.1$ 分（满分 $100$），紧随其后的是 LLaMA-3.1-70B（$64.5$）和 Mixtral-8x22B（$61.4$）。所有代码和实验结果可通过 [https://github.com/CUHK-ARISE/GAMABench](https://github.com/CUHK-ARISE/GAMABench) 获得。

![请参见标题](img/537c25b3b70085cb1f42bf4e7be72ac1.png)

图 1：$\gamma$-Bench 使得多个 LLMs 和人类可以进行多回合游戏。该框架包括三类游戏，每类针对不同的 LLM 能力，并包括八个经典的博弈论游戏。

## 1 引言

我们最近见证了大型语言模型（LLMs）在人工智能领域取得的进展，这标志着该领域的重大突破。作为领先的LLM，**ChatGPT**¹¹1[https://chat.openai.com/](https://chat.openai.com/) 展示了它在多种自然语言处理任务中的高效能力，包括机器翻译**Jiao et al.**（[2023](https://arxiv.org/html/2403.11807v4#bib.bib25)）、句子修订**Wu et al.**（[2023](https://arxiv.org/html/2403.11807v4#bib.bib65)）、信息检索**Zhu et al.**（[2023](https://arxiv.org/html/2403.11807v4#bib.bib73)）和程序修复**Surameery & Shakor**（[2023](https://arxiv.org/html/2403.11807v4#bib.bib60)）。超越学术领域，LLMs已经进入我们的日常生活的各个方面，如教育**Baidoo-Anu & Ansah**（[2023](https://arxiv.org/html/2403.11807v4#bib.bib6)）、法律服务**Guha et al.**（[2023](https://arxiv.org/html/2403.11807v4#bib.bib16)）、产品设计**Lanzi & Loiacono**（[2023](https://arxiv.org/html/2403.11807v4#bib.bib34)）和医疗保健**Johnson et al.**（[2023](https://arxiv.org/html/2403.11807v4#bib.bib26)）。鉴于它们广泛的能力，评估LLMs要求的不仅仅是简单的孤立任务。一个全面且多方面的方法对于评估这些先进模型的有效性是非常必要的。

随着大型语言模型（LLMs）中编码的广泛知识、它们的智能**Liang et al.**（[2024](https://arxiv.org/html/2403.11807v4#bib.bib38)）以及在通用任务解决中的能力**Qin et al.**（[2023](https://arxiv.org/html/2403.11807v4#bib.bib54)），一个问题出现了：LLMs能否帮助日常决策？决策是一个复杂的任务，需要多种能力：(1) 感知：理解情境、环境和规则的能力，LLMs的长文本理解能力也是感知的一部分。(2) 规划：通过预测潜在结果，最大化长期利益而非眼前利益的能力。(3) 算术推理：量化现实世界场景并进行计算的能力。(4) ToM 推理：心智理论**Kosinski**（[2023](https://arxiv.org/html/2403.11807v4#bib.bib32)）；**Bubeck et al.**（[2023](https://arxiv.org/html/2403.11807v4#bib.bib8)）；**Huang et al.**（[2024a](https://arxiv.org/html/2403.11807v4#bib.bib21)）指的是推断他人意图和信念的能力。(5) 批判性思维：整合所有可用信息以做出最佳决策的能力。鉴于这些复杂性，决策对智能体来说是一项重大挑战。

许多研究借鉴了博弈论的原理 Duan 等人 ([2024](https://arxiv.org/html/2403.11807v4#bib.bib11)); Xie 等人 ([2024](https://arxiv.org/html/2403.11807v4#bib.bib67)); Xu 等人 ([2023a](https://arxiv.org/html/2403.11807v4#bib.bib68))，博弈论是一个建立完善的领域，具有若干优势：（1）范围：博弈论可以将多种现实生活场景抽象为简单的数学模型，从而促进广泛的评估。（2）可量化性：通过分析这些模型中的纳什均衡 Nash ([1950](https://arxiv.org/html/2403.11807v4#bib.bib44))，我们可以获得一个可测量的指标，用于比较大语言模型（LLMs）的决策表现。（3）可变性：这些模型的可调参数使得我们能够创建多样化的情境，增强评估的多样性和鲁棒性。然而，现有研究通常限于双人或双行动设置，如经典的囚徒困境和最终通牒博弈 Guo ([2023](https://arxiv.org/html/2403.11807v4#bib.bib17)); Phelps & Russell ([2023](https://arxiv.org/html/2403.11807v4#bib.bib50)); Akata 等人 ([2023](https://arxiv.org/html/2403.11807v4#bib.bib3)); Aher 等人 ([2023](https://arxiv.org/html/2403.11807v4#bib.bib2)); Brookins & DeBacker ([2024](https://arxiv.org/html/2403.11807v4#bib.bib7))。此外，先前的工作依赖于固定的经典博弈设置，增加了大语言模型在训练过程中已遇到这些情境的可能性，从而面临测试集泄露的风险。在本文中，我们评估了在更复杂的情境中大语言模型的表现，这些情境涉及多个玩家、行动和回合，涵盖了经典博弈论场景，并具有动态可调的博弈参数。

表 1：不同大语言模型在 $\gamma$-Bench 上的表现（分数）。

| $\gamma$-Bench 排行榜 | GPT-3.5 | GPT-4 | Gemini-Pro |
| --- | --- | --- | --- |
| 0613 | 1106 | 0125 | 0125 | 1.0 | 1.5 |
| 猜测 2/3 平均数 | $41.4_{\pm 0.5}$ | $68.5_{\pm 0.5}$ | $63.4_{\pm 3.4}$ | $91.6_{\pm 0.6}$ | $77.3_{\pm 6.2}$ | $95.4_{\pm 0.5}$ |
| El Farol Bar | $74.8_{\pm 4.5}$ | $64.3_{\pm 3.1}$ | $68.7_{\pm 2.7}$ | $23.0_{\pm 8.1}$ | $33.5_{\pm 10.3}$ | $37.2_{\pm 4.2}$ |
| 分配美元 | $42.4_{\pm 7.7}$ | $70.3_{\pm 3.3}$ | $68.6_{\pm 2.4}$ | $98.1_{\pm 1.9}$ | $77.6_{\pm 3.6}$ | $93.8_{\pm 0.3}$ |
| 公共物品博弈 | $17.8_{\pm 1.7}$ | $43.5_{\pm 12.6}$ | $38.8_{\pm 8.1}$ | $89.2_{\pm 1.8}$ | $68.5_{\pm 7.6}$ | $100.0_{\pm 0.0}$ |
| 餐厅困境 | $67.0_{\pm 4.9}$ | $1.4_{\pm 1.3}$ | $2.8_{\pm 2.8}$ | $0.9_{\pm 0.7}$ | $3.1_{\pm 1.5}$ | $35.9_{\pm 5.3}$ |
| 封闭竞标拍卖 | $4.2_{\pm 0.3}$ | $3.4_{\pm 1.0}$ | $5.5_{\pm 1.0}$ | $8.7_{\pm 0.7}$ | $14.9_{\pm 6.6}$ | $13.3_{\pm 4.6}$ |
| 战斗皇家 | $19.5_{\pm 7.7}$ | $35.7_{\pm 6.9}$ | $28.6_{\pm 11.0}$ | $86.8_{\pm 9.7}$ | $16.5_{\pm 6.9}$ | $81.3_{\pm 7.7}$ |
| 海盗博弈 | $68.4_{\pm 19.9}$ | $69.5_{\pm 14.6}$ | $71.6_{\pm 7.7}$ | $85.4_{\pm 8.7}$ | $57.4_{\pm 14.3}$ | $87.9_{\pm 5.6}$ |
| 总体 | $41.9_{\pm 2.0}$ | $44.6_{\pm 1.5}$ | $43.5_{\pm 2.1}$ | $60.5_{\pm 2.7}$ | $43.6_{\pm 3.1}$ | $68.1_{\pm 1.5}$ |

(a) 封闭源 LLMs：Gemini-1.5-Pro 在性能上领先。

| $\gamma$-基准排行榜 | LLaMA-3.1 | Mixtral | Qwen-2 |
| --- | --- | --- | --- |
| 8B | 70B | 405B | 8x7B | 8x22B | 72B |
| 猜测 2/3 的平均值 | $85.5_{\pm 3.0}$ | $84.0_{\pm 1.7}$ | $94.3_{\pm 0.6}$ | $91.8_{\pm 0.4}$ | $83.6_{\pm 4.6}$ | $93.2_{\pm 1.3}$ |
| El Farol 酒吧 | $75.7_{\pm 2.2}$ | $59.7_{\pm 3.5}$ | $20.5_{\pm 24.2}$ | $66.8_{\pm 5.8}$ | $39.3_{\pm 12.2}$ | $17.0_{\pm 25.5}$ |
| 分配美元 | $56.4_{\pm 8.4}$ | $87.0_{\pm 4.1}$ | $94.9_{\pm 1.0}$ | $1.2_{\pm 2.8}$ | $79.0_{\pm 9.6}$ | $91.9_{\pm 2.4}$ |
| 公共物品游戏 | $19.6_{\pm 1.0}$ | $90.6_{\pm 3.6}$ | $96.9_{\pm 0.8}$ | $27.7_{\pm 11.7}$ | $83.7_{\pm 3.5}$ | $81.3_{\pm 5.9}$ |
| 餐馆困境 | $59.3_{\pm 2.4}$ | $48.1_{\pm 5.7}$ | $14.4_{\pm 4.5}$ | $76.4_{\pm 7.1}$ | $79.9_{\pm 5.8}$ | $0.0_{\pm 0.0}$ |
| 密封竞标拍卖 | $16.9_{\pm 1.8}$ | $4.5_{\pm 0.7}$ | $4.2_{\pm 1.2}$ | $0.8_{\pm 0.4}$ | $5.2_{\pm 1.8}$ | $0.9_{\pm 0.2}$ |
| 大逃杀 | $35.9_{\pm 12.1}$ | $77.7_{\pm 26.0}$ | $92.7_{\pm 10.1}$ | $12.6_{\pm 9.5}$ | $36.0_{\pm 21.0}$ | $81.7_{\pm 9.6}$ |
| 海盗游戏 | $78.3_{\pm 10.0}$ | $64.0_{\pm 15.5}$ | $65.6_{\pm 22.3}$ | $67.3_{\pm 7.6}$ | $84.3_{\pm 8.8}$ | $86.1_{\pm 6.4}$ |
| 总体 | $53.4_{\pm 3.1}$ | $64.5_{\pm 3.4}$ | $60.4_{\pm 4.4}$ | $43.1_{\pm 2.3}$ | $61.4_{\pm 2.0}$ | $56.5_{\pm 3.4}$ |

(b) 开源 LLMs：LLaMA-3.1-70B 在性能上领先。

我们包括了八个游戏，并根据它们的特点将其分为三类。框架中的第一类评估 LLMs 通过理解游戏规则和识别其他玩家行为模式来做出最优决策的能力。这些游戏的一个显著特点是，个别玩家如果没有合作，是无法获得更高收益的，前提是其他参与者合作。实际上，这些游戏的纳什均衡与最大化整体社会福利一致。我们将这类游戏命名为 I. 合作游戏，包括（1）猜测 2/3 的平均值，（2）El Farol 酒吧，以及（3）分配美元。第二类评估 LLMs 更倾向于优先考虑自身利益，可能会为了更大的收益背叛他人。与第一类不同，这类游戏鼓励背叛合作伙伴的行为，从而获得更高的奖励。通常，这些游戏的纳什均衡导致社会福利的减少。该类别命名为 II. 背叛游戏，包括（4）公共物品游戏，（5）餐馆困境，（6）密封竞标拍卖。最后，我们特别关注两款具有顺序决策过程的游戏，它们与之前六款基于同时决策的游戏有所区别。III. 顺序游戏包括（7）大逃杀和（8）海盗游戏。

在本文中，我们基于 GPT-3.5 (0125) 模型指令十个智能体参与八个博弈，并对获得的结果进行分析。随后，我们评估了模型在多次运行、温度参数变化和提示模板变化下的鲁棒性。进一步的探索将考察诸如 Chain-of-Thought (CoT) [Kojima et al. (2022)](https://arxiv.org/html/2403.11807v4#bib.bib30) 等指导性提示是否能增强模型的决策能力。此外，还考察了模型在不同游戏设置下的泛化能力。最后，我们评估了包括 GPT-3.5-Turbo (0613, 1106, 0125) OpenAI ([2022](https://arxiv.org/html/2403.11807v4#bib.bib47)), GPT-4-Turbo (0125) OpenAI ([2023](https://arxiv.org/html/2403.11807v4#bib.bib48)), Gemini-1.0-Pro Pichai & Hassabis ([2023](https://arxiv.org/html/2403.11807v4#bib.bib51)), Gemini-1.5-Pro Pichai & Hassabis ([2024](https://arxiv.org/html/2403.11807v4#bib.bib52)), LLaMA-3.1 (8B, 70B, 405B) Dubey et al. ([2024](https://arxiv.org/html/2403.11807v4#bib.bib12)), Mixtral (8x7B, 8x22B) Jiang et al. ([2024](https://arxiv.org/html/2403.11807v4#bib.bib24)), 和 Qwen-2-72B Yang et al. ([2024](https://arxiv.org/html/2403.11807v4#bib.bib71)) 等十二个 LLM 的表现。我们通过从同一模型创建多个智能体来参与游戏，然后计算这些智能体的平均表现，比较不同 LLM 的表现。

本文的贡献可以总结为：

+   •

    我们回顾并比较了现有文献中使用博弈论模型评估 LLM 的研究，重点讨论我们对多玩家设定和 LLM 泛化能力的关注。

+   •

    从多玩家的设定出发，我们收集了八个经典的博弈论场景，以衡量大语言模型（LLMs）在多智能体环境中的博弈能力，并实现了我们的框架——GAMA($\gamma$)-Bench。它支持动态生成具有多样化配置的游戏场景，提供无限的场景来评估 LLM 的泛化能力，同时最大限度地减少测试集泄漏的风险。

+   •

    我们将 $\gamma$-Bench 应用到十二个 LLM 上，对它们在多智能体博弈场景中的表现进行深入分析，表明它们在决策过程中作为助手的潜力。

## 2 游戏简介

我们收集了博弈论中广泛研究的八个博弈，并提出了 $\gamma$-Bench，一个具有多玩家、多轮次和多行动设置的框架。值得注意的是，$\gamma$-Bench 允许 LLM 和人类同时参与，使我们能够评估 LLM 在与人类或固定策略对抗时的表现。本节详细介绍了每个游戏及其经典设置（参数）。

### 2.1 合作博弈

#### (1) 猜测平均数的 2/3

由Ledoux（[1981](https://arxiv.org/html/2403.11807v4#bib.bib35)）最初提出，游戏中玩家独立选择一个介于0到100之间的整数（包括0和100）。赢家是选择最接近整个群体平均值的三分之二的数字的玩家。一个典型的初始策略可能会导致玩家假设平均值为50，从而推算出一个大约为$50\times\frac{2}{3}\approx 33$的获胜数字。然而，如果所有参与者都采取这种推理，平均值会变为33，从而将获胜数字改变为大约22。该游戏有一个纯策略纳什均衡（PSNE），其中所有玩家选择零将导致集体获胜。

#### (2) El Farol 酒吧

由Arthur（[1994](https://arxiv.org/html/2403.11807v4#bib.bib4)）和Huberman（[1988](https://arxiv.org/html/2403.11807v4#bib.bib23)）提出的这个游戏要求玩家决定是去酒吧娱乐，还是待在家里不与他人交流。然而，酒吧有一个有限的容量，只能容纳一部分人。如果超过60%的人决定去酒吧，酒吧就会变得过于拥挤，从而不再那么愉快。相反，如果60%或更少的人去酒吧，体验会比待在家里更愉快。设想如果每个人都采取相同的纯策略，即要么每个人都去酒吧，要么每个人都待在家里，那么社会福利就不会得到最大化。值得注意的是，该游戏没有纯策略纳什均衡（PSNE），但存在混合策略纳什均衡（MSNE），其中最佳策略是以60%的概率去酒吧，40%的概率待在家里。

#### (3) 分割美元

首次由Shapley & Shubik（[1969](https://arxiv.org/html/2403.11807v4#bib.bib57)）提到的这个游戏涉及两个玩家独立出价，最高可达100美分来竞标一美元。Ashlock & Greenwood（[2016](https://arxiv.org/html/2403.11807v4#bib.bib5)）进一步将该游戏推广到多人情境中。如果所有玩家的出价总和不超过一美元，则每个玩家将获得他们各自的出价；如果总和超过一美元，则任何玩家都无法获得任何东西。该游戏的纳什均衡（NE）发生在每个玩家出价恰好为$\frac{100}{N}$美分时。

### 2.2 背叛游戏

#### (4) 公共物品游戏

自1950年代初期起就被研究，萨缪尔森（[1954](https://arxiv.org/html/2403.11807v4#bib.bib56)）提出的这个游戏要求$N$个玩家秘密决定将他们的私人代币贡献给公共资金池。池中的代币随后会被一个因子$R$（$1<R<N$）放大，然后产生的“公共物品”会在所有玩家之间均匀分配。玩家保留他们未贡献的代币。简单计算表明，对于每个玩家贡献的代币，他们的净收益是$\frac{R}{N}-1$，小于零。这表明每个玩家的理性策略是不贡献任何代币，从而达成该游戏的纳什均衡（NE）。该游戏作为一种工具，用于研究参与者之间的自私行为和搭便车倾向。

#### (5) 餐厅困境

这个游戏是囚徒困境的多玩家变体，参考了Glance & Huberman ([1994](https://arxiv.org/html/2403.11807v4#bib.bib14))。游戏中有$N$个玩家一起用餐，他们需要共同决定如何分摊所有费用。每个玩家需要独立选择是点贵的菜肴还是便宜的菜肴，分别定价为$x$和$y$（$x>y$）。昂贵的菜肴为每个玩家提供$a$的效用，超过了另一种选择的$b$效用（$a>b$）。该游戏满足两个假设：（1）$a-x<b-y$：虽然昂贵的菜肴提供了更高的效用，但其效益并不值得其更高的成本，这导致当单独就餐时，玩家更倾向于选择便宜的菜肴。（2）$a-\frac{x}{N}>b-\frac{y}{N}$：当费用由所有就餐者共同分担时，个体倾向于选择昂贵的菜肴。这些假设导致了一个纳什均衡（NE），即所有玩家都选择了更贵的菜肴。然而，这种纯策略纳什均衡（PSNE）会导致社会福利总量较低，只有$N(a-x)$，而如果所有人选择便宜菜肴，总社会福利将为$N(b-y)$。这个游戏评估了从长远来看，玩家能否建立持续的合作关系。

#### (6) 密封标书拍卖

密封标书拍卖（SBA）要求玩家同时并保密地提交出价，这与逐步公开出价的拍卖形式不同。我们考虑两种密封标书拍卖的变体：第一价格密封标书拍卖（FPSBA）和第二价格密封标书拍卖（SPSBA）。在FPSBA中，也叫做盲拍，如果所有玩家都出价等于他们对物品的真实估值$v_{i}$，那么获胜者的净收益为$b_{i}-v_{i}=0$，而其他玩家也没有获得任何收益，McAfee & McMillan ([1987](https://arxiv.org/html/2403.11807v4#bib.bib41))。此外，最高出价者会发现，为了赢得拍卖，只需出价略高于第二高的出价即可。受到这两个因素的驱动，FPSBA在实际场景中常被认为效率低下，因为竞标者往往倾向于提交远低于其实际估值的出价，从而导致社会福利次优。相比之下，SPSBA，通常称为维克里拍卖，要求获胜者支付第二高的出价，鼓励所有玩家进行真实出价，Vickrey ([1961](https://arxiv.org/html/2403.11807v4#bib.bib62))。可以证明，在SPSBA中真实出价代表着一个纳什均衡（NE）。该拍卖评估了在信息不完全的博弈中代理的表现，在这种博弈中，代理无法知道其他玩家的估值。

### 2.3 顺序博弈

#### (7) 大逃杀

从**Truel** Kilgour（[1975](https://arxiv.org/html/2403.11807v4#bib.bib29)）扩展而来，**Battle Royale**涉及$N$个玩家相互射击。在广泛研究的形式中，Kilgour & Brams（[1997](https://arxiv.org/html/2403.11807v4#bib.bib28)）中，玩家有不同的命中目标的概率，回合顺序由命中概率的升序决定。游戏允许无限子弹，并且有意错过射击的战术选项。每个参与者的目标是成为唯一的幸存者，游戏在只剩下一个玩家时结束。尽管已为无限顺序**truel**确定了纳什均衡（NE），Kilgour（[1977](https://arxiv.org/html/2403.11807v4#bib.bib27)）指出，随着玩家数量的增加，这些均衡的复杂性呈指数级上升。

#### (8) 海盗游戏

这个游戏是**Ultimatum Game**的多人版本，Goodin（[1998](https://arxiv.org/html/2403.11807v4#bib.bib15)）；Stewart（[1999](https://arxiv.org/html/2403.11807v4#bib.bib59)）。每个玩家被分配一个“海盗等级”，决定他们的行动顺序。游戏涉及$N$个海盗讨论他们发现的$G$金币的分配。最资深的海盗首先提出分配方法。如果提议被至少一半的海盗批准，包括提议者本人，游戏结束，金币按照提议分配。否则，最资深的海盗将被扔下船，下一位等级较高的海盗将担任提议者角色，直到游戏结束。每个海盗的目标优先级为（1）生存，（2）最大化自己的金币份额，以及（3）有机会将其他人淘汰出局。Stewart（[1999](https://arxiv.org/html/2403.11807v4#bib.bib59)）确定了最优策略，其中最资深的海盗将一金币分配给每个奇数排名的海盗，其余金币自己保留。

## 3 GAMA-Bench评分方案

本节展示了使用GPT-3.5（0125）模型在每个游戏的默认设置下进行的实验。以该模型为案例研究，我们展示了如何使用$\gamma$-Bench来评估LLM的表现。提示和设计方法可参见附录§[C](https://arxiv.org/html/2403.11807v4#A3 "附录C 关于提示的详细信息 ‣ 我们在LLM决策中的进展如何？评估LLM在多智能体环境中的游戏能力")。每个游戏包含基于GPT-3.5的十个智能体，温度参数设置为1。对于同时进行的游戏，将进行二十轮。为了提高结果的可靠性并减少方差的影响，我们每个游戏进行了五次运行。为了清晰和简洁，本节展示了五次运行中的一次，而§[4.1](https://arxiv.org/html/2403.11807v4#S4.SS1 "4.1 RQ1: 稳健性 ‣ 4 超越默认设置 ‣ 我们在LLM决策中的进展如何？评估LLM在多智能体环境中的游戏能力")详细介绍了定量结果。我们在$\gamma$-Bench上对GPT-3.5行为的研究结果包括：

<svg class="ltx_picture" height="157.16" id="S3.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,157.16) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 5.32 5.32)"><foreignobject color="#000000" height="146.52" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="589.36">Key Findings: • The model’s decisions are mainly influenced by the outcomes of the preceding round rather than deriving from the reasoning of the optimal strategy. • Although initially demonstrating suboptimal performance, the model can learn from historical data and enhance its performance over time. A larger fluctuation is observed in games that are difficult to optimize from historical data, such as the El Farol Bar game. • The model demonstrates the ability to engage in spontaneous cooperation, leading to increased social welfare beyond mere self-interest, without the necessity for explicit communication. However, this phenomenon also results in low performance in Betraying Games. • The model shows limitations in sequential games with more complicated rules. • The aggregate score of the model on $\gamma$-Bench is $44.9$.</foreignobject></g></g></svg>

### 3.1 合作游戏

#### (1) 猜测2/3的平均值

[[TO PROMPT]](https://arxiv.org/html/2403.11807v4#A3.SS2 "C.2 合作游戏 ‣ 附录C 关于提示的详细信息 ‣ 我们在LLM决策中的进展如何？评估LLM在多智能体环境中的游戏能力") 该游戏的标准设置为 $MIN=0$，$MAX=100$，以及 $R=\frac{2}{3}$。我们在图[2](https://arxiv.org/html/2403.11807v4#S3.F2 "图2 ‣ (2) El Farol Bar ‣ 3.1 合作游戏 ‣ 3 GAMA-Bench评分方案 ‣ 我们在LLM决策中的进展如何？评估LLM在多智能体环境中的游戏能力") 中展示了所有智能体的选择、平均值以及获胜数字。关键观察包括：(1) 在第一轮中，智能体一致选择$50$（或接近$50$），这与从$0$到$100$的均匀分布的均值相对应。这种行为表明模型未能识别出获胜数字是平均值的$\frac{2}{3}$。(2) 随着回合的进行，所选的平均数字显著下降，表明智能体能够根据历史结果进行适应。由于最佳策略是选择$MIN$，因此本游戏中的得分公式为 $S_{1}=\frac{1}{NK}\sum_{ij}(C_{ij}-MIN)$，其中$C_{ij}$是玩家$i$在第$j$轮选择的数字。模型得分为$65.4$，这是基于此游戏的结果计算得出的²²2为清晰起见，我们将原始得分归一化到$[0,100]$范围内，较高的值表示更好的表现。重缩放方法的详细信息请参见附录§[E](https://arxiv.org/html/2403.11807v4#A5 "附录E 原始得分重缩放方法 ‣ 我们在LLM决策中的进展如何？评估LLM在多智能体环境中的游戏能力")。

#### (2) El Farol Bar

[[提示来源]](https://arxiv.org/html/2403.11807v4#A3.SS2 "C.2 合作游戏 ‣ 附录 C 提示的详细信息 ‣ 我们在大语言模型决策能力上的进展？评估大语言模型在多智能体环境中的游戏能力")  该游戏的标准设置为 $MIN=0$, $MAX=10$, $HOME=5$, 和 $R=60\%$。为了探索不完全信息的影响，我们引入了两种设置：显式设置表示每个人都可以在每轮结束时看到结果，而隐式设置表示那些待在家中的玩家无法得知酒吧中的情况。图 [2](https://arxiv.org/html/2403.11807v4#S3.F2 "图 2 ‣ (2) El Farol 酒吧 ‣ 3.1 合作游戏 ‣ 3 GAMA-Bench 评分方案 ‣ 我们在大语言模型决策能力上的进展？评估大语言模型在多智能体环境中的游戏能力")(2) 展示了智能体决定是否去酒吧的概率和酒吧中的总玩家数。我们发现：（1）在第一轮中，智能体有倾向去酒吧。由于人满为患，玩家更倾向于待在家里，导致图 [2](https://arxiv.org/html/2403.11807v4#S3.F2 "图 2 ‣ (2) El Farol 酒吧 ‣ 3.1 合作游戏 ‣ 3 GAMA-Bench 评分方案 ‣ 我们在大语言模型决策能力上的进展？评估大语言模型在多智能体环境中的游戏能力")(2-1) 和图 [2](https://arxiv.org/html/2403.11807v4#S3.F2 "图 2 ‣ (2) El Farol 酒吧 ‣ 3.1 合作游戏 ‣ 3 GAMA-Bench 评分方案 ‣ 我们在大语言模型决策能力上的进展？评估大语言模型在多智能体环境中的游戏能力")(2-2) 中都出现了波动。在隐式设置下，由于缺乏酒吧占用情况的直接观察，智能体需要额外的轮次（第 $2$ 至第 $6$ 轮）来辨识酒吧是否有空位。（2）智能体去酒吧的概率逐渐稳定，且隐式设置中的平均概率低于显式设置。由于最优策略是以概率 $R$ 选择去酒吧，因此该游戏的原始得分³³3为了简化起见，我们仅评估了隐式设置。为 $S_{2}=\frac{1}{K}\sum_{j}\lvert\frac{1}{N}\sum_{i}D_{ij}-R\rvert$，其中 $D_{ij}=1$ 表示玩家 $i$ 在第 $j$ 轮选择去酒吧，$D_{ij}=0$ 表示玩家 $i$ 选择待在家里。该模型在该游戏中的得分为 $73.3$。

![参见说明](img/c8480c01bfef61ce47cf1329be04cfbc.png)

图 2：GPT-3.5（0125）在合作与背叛游戏中的表现。

#### (3) 分配美元

[[TO PROMPT]](https://arxiv.org/html/2403.11807v4#A3.SS2 "C.2 合作游戏 ‣ 附录 C 关于提示的细节 ‣ 我们在大语言模型的决策制定上走了多远？评估大语言模型在多智能体环境中的游戏能力")  该游戏的基础设置为$G=100$。我们在图[2](https://arxiv.org/html/2403.11807v4#S3.F2 "图2 ‣ (2) El Farol Bar ‣ 3.1 合作游戏 ‣ 3 GAMA-Bench评分方案 ‣ 我们在大语言模型的决策制定上走了多远？评估大语言模型在多智能体环境中的游戏能力")(3)中绘制了所有代理人的提案及其提案的总和。我们的分析揭示了以下见解：（1）在第一轮中，代理人的决策与游戏的纳什均衡（NE）预测一致。然而，在获得金币后，代理人表现出更强的贪婪，提出超过NE规定的分配金额。在什么都没得到后，他们倾向于提出一个“更安全”的金额。这个趋势持续并导致后续轮次的波动。（2）尽管存在这些波动，提议的金币总和趋向于约$100$。由于最佳策略是提出$G/N$，因此该游戏中的原始得分为$S_{3}=\frac{1}{K}\sum_{j}\lvert\sum_{i}B_{ij}-G\rvert$，其中$B_{ij}=1$表示玩家$i$在第$j$轮的提议金额。模型在该游戏中的得分为$68.1$。

### 3.2 背叛游戏

#### (4) 公共物品游戏

[[TO PROMPT]](https://arxiv.org/html/2403.11807v4#A3.SS3 "C.3 背叛游戏 ‣ 附录 C 关于提示的细节 ‣ 我们在大语言模型的决策制定上走了多远？评估大语言模型在多智能体环境中的游戏能力")  该游戏的基础设置为$R=2$。每个玩家每轮有$T=20$的贡献额度。图[2](https://arxiv.org/html/2403.11807v4#S3.F2 "图2 ‣ (2) El Farol Bar ‣ 3.1 合作游戏 ‣ 3 GAMA-Bench评分方案 ‣ 我们在大语言模型的决策制定上走了多远？评估大语言模型在多智能体环境中的游戏能力")(4)展示了每个代理人贡献的代币及其每轮相应的收益。观察结果揭示了以下几点：（1）尽管投资回报为$-80\%$，代理人表现出在搭便车和贡献所有代币之间交替的模式。（2）随着轮次的推进，贡献到公共资金池的代币数量显著增加，从而提高了整体社会福利。这些发现表明，LLM展现了合作行为，优先考虑集体利益而非个人利益。由于我们期望模型能够推断出最佳策略，即贡献零代币，因此该游戏中的原始得分为$S_{4}=\frac{1}{NK}\sum_{ij}C_{ij}$，其中$C_{ij}=1$表示玩家$i$在第$j$轮的贡献金额。模型在该游戏中的得分为$41.3$。

#### (5) 餐馆困境

[[提示]](https://arxiv.org/html/2403.11807v4#A3.SS3 "C.3 背叛游戏 ‣ 附录 C 提示详情 ‣ 我们在大规模语言模型的决策能力上走了多远？评估大规模语言模型在多智能体环境中的游戏能力")  这个游戏的基本设置为$P_{h}=20$，$P_{l}=10$，$U_{h}=20$，$U_{l}=15$。我们在图[2](https://arxiv.org/html/2403.11807v4#S3.F2 "图 2 ‣ (2) El Farol Bar ‣ 3.1 合作博弈 ‣ 3 GAMA-Bench 评分方案 ‣ 我们在大规模语言模型的决策能力上走了多远？评估大规模语言模型在多智能体环境中的游戏能力")中展示了智能体选择昂贵菜肴的概率、他们的最终效用以及平均账单。对该图的分析揭示了以下几点： (1) 与该游戏的纳什均衡预测相反，智能体普遍倾向于选择便宜的菜肴，这最大化了整体社会福利。 (2) 引人注目的是，出现了一种背离合作行为的现象，其中一个智能体持续选择背叛其他人，从而获得更高的效用。该智能体在随后的回合中持续保持这一背叛模式。由于我们期望模型推断出最优策略，即选择昂贵的菜肴，该游戏的原始得分由$S_{5}=\frac{1}{NK}\sum_{ij}D_{ij}$给出，其中$D_{ij}=1$表示玩家$i$在回合$j$中选择了便宜的菜肴，$D_{ij}=0$表示玩家$i$选择了昂贵的菜肴。该模型在此游戏中的得分为$4.0$。

#### (6) 密封竞标拍卖

[[提示]](https://arxiv.org/html/2403.11807v4#A3.SS3 "C.3 背叛游戏 ‣ 附录 C 提示详细信息 ‣ LLMs 的决策能力现状：评估 LLMs 在多代理环境中的游戏能力")  在该游戏的基本设置中，我们在每一回合随机为每个代理分配估值，范围从 $0$ 到 $200$。我们固定随机数生成的种子，以确保在不同设置和模型之间的公平比较。我们评估 LLMs 在第一价格拍卖和第二价格拍卖两种设置下的表现。图 [2](https://arxiv.org/html/2403.11807v4#S3.F2 "图 2 ‣ (2) El Farol 酒吧 ‣ 3.1 合作游戏 ‣ 3 GAMA-Bench 评分方案 ‣ LLMs 的决策能力现状：评估 LLMs 在多代理环境中的游戏能力") (6) 展示了每个代理的估值与出价之间的差异以及出价金额。我们的主要发现包括：（1）如 §[2.2](https://arxiv.org/html/2403.11807v4#S2.SS2.SSS0.Px3 "(6) 密封竞价拍卖 ‣ 2.2 背叛游戏 ‣ 2 游戏简介 ‣ LLMs 的决策能力现状：评估 LLMs 在多代理环境中的游戏能力") 中所述，我们注意到在第一价格拍卖中，代理通常提交的出价低于他们的估值，这一趋势通过图 [2](https://arxiv.org/html/2403.11807v4#S3.F2 "图 2 ‣ (2) El Farol 酒吧 ‣ 3.1 合作游戏 ‣ 3 GAMA-Bench 评分方案 ‣ LLMs 的决策能力现状：评估 LLMs 在多代理环境中的游戏能力") (6-1) 中估值与出价之间的正差异得以体现。（2）尽管纳什均衡（NE）建议每个人在第二价格设置中出价与其估值相同，但我们发现出价低于估值的倾向，正如图 [2](https://arxiv.org/html/2403.11807v4#S3.F2 "图 2 ‣ (2) El Farol 酒吧 ‣ 3.1 合作游戏 ‣ 3 GAMA-Bench 评分方案 ‣ LLMs 的决策能力现状：评估 LLMs 在多代理环境中的游戏能力") (6-2) 中所示。由于最佳策略是出价低于其真实估值⁴⁴4 我们根据背叛游戏的定义仅评估第一价格设置，因此该游戏的原始得分为 $S_{6}=\frac{1}{NK}\sum_{ij}(v_{ij}-b_{ij})$，其中 $v_{ij}$ 和 $b_{ij}$ 分别为玩家 $i$ 在第 $j$ 回合的估值和出价。模型在该游戏中的得分为 $6.5$。

### 3.3 顺序游戏

![参考说明](img/5698a53ee0ab027ca099ec33be7c0c2a.png)

图 3：GPT-3.5 (0125) 在“生死竞技场”中的表现。（a）：每一回合中代理的动作和结果。例如，在第 $11$ 回合，玩家 $6$ 射击玩家 $7$ 但未命中。

#### (7) 生死竞技场

[[TO PROMPT]](https://arxiv.org/html/2403.11807v4#A3.SS4 "C.4 顺序游戏 ‣ 附录C 关于提示的详细信息 ‣ 我们在LLM的决策能力上走了多远？评估LLM在多代理环境中的游戏能力")  在本游戏的基本设置中，我们为每个代理分配了不同的命中率，从 $35\%$ 到 $80\%$，以 $5\%$ 为增量。这种设置涵盖了广泛的命中率范围，避免了极端的 $0\%$ 或 $100\%$。图 [3](https://arxiv.org/html/2403.11807v4#S3.F3 "图3 ‣ 3.3 顺序游戏 ‣ 3 GAMA-Bench评分方案 ‣ 我们在LLM的决策能力上走了多远？评估LLM在多代理环境中的游戏能力") 展示了每轮的行动和结果，以及剩余参与者的统计。我们的观察发现：(1) 与我们的预期不同，代理很少会选择目标是命中率最高的玩家。(2) 代理忽略了“故意错过”这一策略。例如，在第 $19$ 轮，剩下玩家 $7$、$8$ 和 $10$，轮到玩家 $7$ 行动。玩家 $7$ 的最佳策略本应是故意错过一枪，从而诱使玩家 $8$ 消除玩家 $10$，让玩家 $7$ 能在下一轮针对玩家 $8$ 争取胜利。然而，玩家 $7$ 选择了攻击玩家 $10$，导致玩家 $8$ 自己开火。为简单起见，我们评估代理是否针对命中率最高的玩家（不包括自己）。因此，本游戏的原始得分为 $S_{7}=\frac{1}{Nk}\sum_{ij}I_{ij}$，其中 $k$ 代表已进行的轮次数量，$I_{ij}=1$ 表示玩家 $i$ 在第 $j$ 轮瞄准了命中率最高的玩家，而 $I_{ij}=0$ 则表示没有瞄准。该模型在此游戏中的得分为 $20.0$。

表2：GPT-3.5（0125）在“海盗游戏”中的表现。每一行显示了特定轮次中提出的黄金分配方案，以及每个海盗是否接受（“✓”）或拒绝（“✗”）该提案。$S_{8P}$ 显示了提案者的得分，而 $S_{8V}$ 显示了所有投票者的得分。

| 海盗排名 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | $S_{8P}$ | $S_{8V}$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 第1轮 | 100✓ | 0✗ | 0✗ | 0✗ | 0✗ | 0✗ | 0✗ | 0✗ | 0✗ | 0✗ | $8$ | $1.00$ |
| 第2轮 | - | 99✓ | 0✗ | 1✓ | 0✓ | 0✗ | 0✗ | 0✗ | 0✗ | 0✓ | $6$ | $0.75$ |
| 第3轮 | - | - | 50✓ | 1✓ | 1✓ | 1✓ | 1✓ | 1✓ | 1✓ | 44✓ | $94$ | $0.57$ |

#### (8) 海盗游戏

[[提示]](https://arxiv.org/html/2403.11807v4#A3.SS4 "C.4 顺序博弈 ‣ 附录 C 提示详情 ‣ 我们在大语言模型的决策制定上走多远？评估大语言模型在多智能体环境中的博弈能力") 该游戏的默认设置是 $G=100$。如§[2.3](https://arxiv.org/html/2403.11807v4#S2.SS3.SSS0.Px2 "(8) 海盗游戏 ‣ 2.3 顺序博弈 ‣ 2 游戏简介 ‣ 我们在大语言模型的决策制定上走多远？评估大语言模型在多智能体环境中的博弈能力") 中所介绍，首位提议者的最佳策略是将 $96$ 金币分配给自己，分别给第三、第五、第七和第九个海盗各一金币。Stewart ([1999](https://arxiv.org/html/2403.11807v4#bib.bib59)) 阐明了选民的最佳策略：（1）如果分配到两个或更多金币，则接受；（2）如果没有分配金币，则拒绝；（3）如果分配到一个金币且该金币与提议者的奇偶性相同，则接受，否则拒绝。表[2](https://arxiv.org/html/2403.11807v4#S3.T2 "表 2 ‣ (7) 生死战 ‣ 3.3 顺序博弈 ‣ 3 GAMA-Bench 评分方案 ‣ 我们在大语言模型的决策制定上走多远？评估大语言模型在多智能体环境中的博弈能力") 展示了一个示例游戏的提议和投票结果。关键结论是，代理未能提出最佳提议，并且经常投出错误的选票，这表明大语言模型在该游戏中的表现次优。为了全面评估模型的表现，考虑了两个方面：（1）提议者是否给出了合理的提议，（2）选民是否对给定的提议采取了正确的行动。对于（1），我们计算给定提议与最佳策略之间的 $L_{1}$ 范数，定义为 $S_{8P}=\frac{1}{k}\sum_{j}\lVert P_{j}-O_{j}\rVert_{1}$，其中 $P_{j}$ 代表模型的提议，$O_{j}$ 代表第 $j$ 轮的最佳提议，游戏在第 $k$ 轮结束。对于（2），我们计算选择正确行动的准确性，如上所述，公式为：$S_{8V}=\frac{2}{k(2N-k-1)}\sum_{ij}I_{ij}$，其中 $I_{ij}=1$ 如果玩家 $i$ 在第 $j$ 轮投票正确，$I_{ij}=0$ 否则，提议者不计入计算。该模型在此游戏中的得分为 $80.6$。

## 超越默认设置

本节深入探讨了以下几个研究问题（RQ）。RQ1 鲁棒性：在多次运行中是否存在显著的差异？性能是否对不同的温度和提示模板敏感？RQ2 推理策略：增强推理能力的策略是否适用于游戏场景？这包括实现链式思维（Chain-of-Thought, CoT）Kojima等人（[2022](https://arxiv.org/html/2403.11807v4#bib.bib30)）；Wei等人（[2022](https://arxiv.org/html/2403.11807v4#bib.bib64)）的推理方法，以及为LLMs分配独特的个性。RQ3 泛化能力：LLM性能如何随不同游戏设置变化？LLMs是否记得在训练阶段学到的答案？RQ4 排行榜：不同的LLMs在$\gamma$-Bench上的表现如何？除非另有说明，否则我们使用§[3](https://arxiv.org/html/2403.11807v4#S3 "3 GAMA-Bench Scoring Scheme ‣ How Far Are We on the Decision-Making of LLMs? Evaluating LLMs’ Gaming Ability in Multi-Agent Environments")中描述的默认设置。

### 4.1 RQ1: 鲁棒性

本研究问题（RQ）考察了大语言模型（LLMs）响应的稳定性，评估了三个关键因素对模型性能的影响：(1) 模型采样策略引入的随机性，(2) 温度参数设置，(3) 用于游戏指令的提示词。

#### 多次运行

首先，我们在相同设置下运行所有游戏五次。图[4](https://arxiv.org/html/2403.11807v4#A6.F4 "Figure 4 ‣ F.1 Robustness: Multiple Runs ‣ Appendix F Detailed Results ‣ How Far Are We on the Decision-Making of LLMs? Evaluating LLMs’ Gaming Ability in Multi-Agent Environments")展示了各测试的平均表现，而表[4](https://arxiv.org/html/2403.11807v4#A6.T4 "Table 4 ‣ F.1 Robustness: Multiple Runs ‣ Appendix F Detailed Results ‣ How Far Are We on the Decision-Making of LLMs? Evaluating LLMs’ Gaming Ability in Multi-Agent Environments")列出了相应的分数。分析结果显示，除了两个顺序游戏和“公共物品游戏”外，模型展示了一致的表现，这可以通过每个游戏的分数方差较低来证明。

#### 温度

正如我们在文献综述中所讨论的 §[B](https://arxiv.org/html/2403.11807v4#A2 "附录 B 文献综述：通过博弈论评估大型语言模型 ‣ 我们在大型语言模型决策能力方面的进展如何？评估大型语言模型在多智能体环境中的博弈能力")，先前的研究采用了不同的温度参数范围，从 $0$ 到 $1$，但未深入探讨其影响。本研究在原始设置下，针对一系列温度 $\{0.0,0.2,0.4,0.6,0.8,1.0\}$ 进行了跨游戏的实验。实验结果，既有视觉展示也有定量分析，分别记录在图 [5](https://arxiv.org/html/2403.11807v4#A6.F5 "图 5 ‣ F.2 鲁棒性：温度 ‣ 附录 F 详细结果 ‣ 我们在大型语言模型决策能力方面的进展如何？评估大型语言模型在多智能体环境中的博弈能力") 和表 [5](https://arxiv.org/html/2403.11807v4#A6.T5 "表 5 ‣ F.2 鲁棒性：温度 ‣ 附录 F 详细结果 ‣ 我们在大型语言模型决策能力方面的进展如何？评估大型语言模型在多智能体环境中的博弈能力") 中。结果表明，$3.4$ 的小整体方差表明，对于大多数游戏，温度调整对结果的影响可以忽略不计。一个显著的例外出现在“猜 2/3 平均值”游戏中，随着温度的增加，得分有所提升（从 $48.0$ 增至 $65.4$），这与零温度下接近随机的表现形成鲜明对比。

#### 提示模板

我们还研究了提示措辞对模型表现的影响。我们利用 GPT-4 重新编写了默认提示模板，生成了四个额外版本。我们对生成的版本进行了手动检查，确保 GPT-4 遵守游戏规则，并且没有更改关键数据。这些提示模板可以在 §[D](https://arxiv.org/html/2403.11807v4#A4 "附录 D GPT-4 重述提示示例 ‣ 我们在大型语言模型决策能力方面的进展如何？评估大型语言模型在多智能体环境中的博弈能力") 中找到。我们将使用这些模板的结果绘制在图 [6](https://arxiv.org/html/2403.11807v4#A6.F6 "图 6 ‣ F.3 鲁棒性：提示版本 ‣ 附录 F 详细结果 ‣ 我们在大型语言模型决策能力方面的进展如何？评估大型语言模型在多智能体环境中的博弈能力") 中，并在表 [6](https://arxiv.org/html/2403.11807v4#A6.T6 "表 6 ‣ F.3 鲁棒性：提示版本 ‣ 附录 F 详细结果 ‣ 我们在大型语言模型决策能力方面的进展如何？评估大型语言模型在多智能体环境中的博弈能力") 中记录了定量分数。值得注意的是，我们发现提示措辞对性能有显著影响，如“公共物品博弈”（$11.5$）、“餐厅困境”（$23.7$）和“海盗博弈”（$14.7$）中的高方差所示。

<svg class="ltx_picture" height="56.15" id="S4.SS1.SSS0.Px3.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,56.15) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 5.32 5.32)"><foreignobject color="#000000" height="45.51" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="589.36">Answer to RQ1: GPT-3.5 exhibits consistency in multiple runs and shows robustness against different temperature settings. However, inappropriate prompt designs resulting from potential misinformation during rephrasing can significantly impair performance.</foreignobject></g></g></svg>

### 4.2 RQ2: 推理策略

本研究问题（RQ）关注通过提示指令来提高模型性能。我们研究了两种策略：链式思维（CoT）提示，Kojima等人（[2022](https://arxiv.org/html/2403.11807v4#bib.bib30)）和人物设定，Kong等人（[2024](https://arxiv.org/html/2403.11807v4#bib.bib31)）。我们在图[7](https://arxiv.org/html/2403.11807v4#A6.F7 "图7 ‣ F.4 推理策略 ‣ 附录F 详细结果 ‣ 我们在大型语言模型的决策能力上走多远了？评估LLMs在多智能体环境中的游戏能力")和表[7](https://arxiv.org/html/2403.11807v4#A6.T7 "表7 ‣ F.4 推理策略 ‣ 附录F 详细结果 ‣ 我们在大型语言模型的决策能力上走多远了？评估LLMs在多智能体环境中的游戏能力")中展示了可视化和定量结果。

#### CoT

根据Kojima等人（[2022](https://arxiv.org/html/2403.11807v4#bib.bib30)）的研究，引入一个初步的短语“让我们一步步思考”，鼓励模型在提出结论之前，按顺序分析并解释其推理过程。这种方法在特定场景下证明了其有效性，比如在游戏（1）、（3）、（4）和（5）中，通过提高整体得分从$44.9$到$57.4$，增加了$12.5$。在“（3）分配美元”游戏中，结合CoT方法减少了模型建议不成比例的大额分配的倾向，得分提高了$15.3$。类似地，在“（4）公共物品游戏”和“（5）晚餐困境”中，CoT促使模型意识到当自由搭便车者是最优策略，得分分别提高了$14.8$和$51.5$。

#### 人物设定

研究表明，Kong等人（[2024](https://arxiv.org/html/2403.11807v4#bib.bib31)）；Huang等人（[2024b](https://arxiv.org/html/2403.11807v4#bib.bib22)）证明了为模型分配角色会影响其在各种下游任务中的表现。受到这一发现的启发，我们的研究开始时通过一个指定模型角色的提示，例如“你是[角色]”，其中角色可以是合作且协作的助手、利己且贪婪的助手或数学家。我们的研究结果显示，分配“合作”角色能够提高模型在游戏（1）、（2）和（3）中的表现，特别是在“（3）El Farol酒吧”游戏中，表现超越了CoT方法。相反，“自私”角色显著降低了模型在几乎所有游戏中的表现，唯一的例外是“（7）生死战”游戏。“数学家”角色将模型的整体得分提高了$0.7$，这个提升较小，并未超过CoT方法的效果。

<svg class="ltx_picture" height="57.69" id="S4.SS2.SSS0.Px2.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,57.69) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 5.32 5.32)"><foreignobject color="#000000" height="47.05" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="589.36">Answer to RQ2: It is possible to improve GPT-3.5 through simple prompt instructions. Among the methods we explore, the CoT prompting performs the best, achieving a performance close to GPT-4 ($57.4$ vs. $60.5$).</foreignobject></g></g></svg>

### 4.3 研究问题3：泛化能力

考虑到数学、经济学和计算机科学等领域对游戏的广泛探索，这些游戏的原始设置很可能已经包含在LLM的训练数据集中。为了确定我们选择的游戏中是否存在数据污染，我们对其进行了多种设置测试。每个游戏所选参数的具体细节见表[8](https://arxiv.org/html/2403.11807v4#A6.T8 "Table 8 ‣ F.5 Generalizability ‣ Appendix F Detailed Results ‣ How Far Are We on the Decision-Making of LLMs? Evaluating LLMs’ Gaming Ability in Multi-Agent Environments")，实验结果以图[8](https://arxiv.org/html/2403.11807v4#A6.F8 "Figure 8 ‣ F.5 Generalizability ‣ Appendix F Detailed Results ‣ How Far Are We on the Decision-Making of LLMs? Evaluating LLMs’ Gaming Ability in Multi-Agent Environments")的形式直观展示。我们的研究发现，不同游戏中的模型泛化能力存在差异。具体来说，在游戏(1)、(3)、(5)、(6)和(8)中，模型在不同设置下表现出正确的性能。在“(3) Divide the Dollar”游戏中，随着总金币数（$G$）的增加，模型的表现有所提升，这表明更高的金币分配能满足所有玩家的需求。相反，模型在游戏(2)和(4)中的泛化能力较差。对游戏“(2) El Farol Bar”的分析表明，模型在不同酒吧容量（$R$）下表现出一致的决策模式，始终以约50%的概率选择参与，这表明模型的行为是随机的。同样，在“(4) 公共物品游戏”中，模型即使在回报率为零的情况下，也始终贡献相似的金额，表明模型未能理解游戏规则。造成这种表现不佳的一个可能原因是，模型无法基于历史数据逐步调整其表现。

Nagel（[1995](https://arxiv.org/html/2403.11807v4#bib.bib43)）进行了实验，$15$至$18$名参与者参与了“(1) 猜2/3平均数”游戏，使用了$\frac{1}{2}$、$\frac{2}{3}$和$\frac{4}{3}$的比例。每个比例的平均数字分别为$27.05$、$36.73$和$60.12$。类似地，Rubinstein（[2007](https://arxiv.org/html/2403.11807v4#bib.bib55)）对$\frac{2}{3}$比例进行了更大规模的实验，涉及2,423名参与者，得到了相似的平均数$36.2$，与Nagel（[1995](https://arxiv.org/html/2403.11807v4#bib.bib43)）的发现一致。模型在相同的比例下产生的平均数为$34.59$、$34.59$和$74.92$，这表明其预测更接近人类行为，而非游戏的纳什均衡（NE）。

<svg class="ltx_picture" height="89.36" id="S4.SS3.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,89.36) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 5.32 5.32)"><foreignobject color="#000000" height="78.72" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="589.36">Answer to RQ3: GPT-3.5 demonstrates variable performance across different game settings, exhibiting notably lower efficacy in “(2) El Farol Bar” and “(4) Public Goods Game.” It is noteworthy that, $\gamma$-Bench provides a test bed to evaluate the ability of LLMs in complex reasoning scenarios. As model’s ability improves (e.g., achieving more than $90$ on $\gamma$-Bench), we can increase the difficulty by varying game settings.</foreignobject></g></g></svg>

### 4.4 RQ4: 排行榜

本RQ调查了不同LLMs在决策能力上的差异，使用了$\gamma$-Bench。我们首先关注开源模型，包括OpenAI的GPT-3.5（0613、1106和0125）、GPT-4（0125）和Google的Gemini Pro（1.0、1.5）。结果整理在表[1(a)](https://arxiv.org/html/2403.11807v4#S1.T1.st1 "表1 ‣ 1 引言 ‣ 我们在大规模语言模型决策能力方面的进展如何？评估LLMs在多智能体环境中的游戏能力")中，模型性能在附录的图[9](https://arxiv.org/html/2403.11807v4#A6.F9 "图9 ‣ F.6 排行榜 ‣ 附录F详细结果 ‣ 我们在大规模语言模型决策能力方面的进展如何？评估LLMs在多智能体环境中的游戏能力")中进行了可视化。Gemini-1.5-Pro得分为$68.1$，明显超过了其他模型，特别是在游戏（1）、（4）和（5）中。GPT-4紧随其后，得分为$60.5$。在“(2) El Farol Bar”游戏中的表现下降（$23.0$）源于其偏好保守策略，倾向于待在家中。在“(5) Diner’s Dilemma”游戏中的表现不佳（$0.9$）则是因为偏好个人收益而非集体利益。此外，对三个GPT-3.5更新的评估显示其表现相似。

接下来，我们关注开源模型，其性能在表[1(b)](https://arxiv.org/html/2403.11807v4#S1.T1.st2 "表1 ‣ 1 引言 ‣ 我们在大规模语言模型决策能力方面的进展如何？评估LLMs在多智能体环境中的游戏能力")中进行了详细展示，并在图[10](https://arxiv.org/html/2403.11807v4#A6.F10 "图10 ‣ F.6 排行榜 ‣ 附录F详细结果 ‣ 我们在大规模语言模型决策能力方面的进展如何？评估LLMs在多智能体环境中的游戏能力")中进行了可视化。排名前两的开源模型LLaMA-3.1-70B和Mixtral-8x22B紧随Gemini-1.5-Pro，其得分分别为$64.5$和$61.4$，超越了GPT-4。大多数开源模型，包括Qwen-2、LLaMA-3.1-405B和LLaMA-3.1-8B，都超越了GPT-3.5和Gemini-1.0-Pro。Mixtral-8x7B的表现最差，可能是因为其规模较小且推理能力较弱。有趣的是，LLaMA-3.1-405B的表现不及其较小的对应版本70B版，我们将其归因于在“(2) El Farol Bar”游戏中的过于保守策略，这一挑战与GPT-4所面临的类似。

<svg class="ltx_picture" height="39.55" id="S4.SS4.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,39.55) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 5.32 5.32)"><foreignobject color="#000000" height="28.9" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="589.36">Answer to RQ4: Currently, Gemini-1.5-Pro outperforms all other models evaluated in this study. LLaMA-3.1-70B performs closely, being in the second place.</foreignobject></g></g></svg>

## 5 相关工作

### 5.1 特定游戏

除了表格[3](https://arxiv.org/html/2403.11807v4#A2.T3 "Table 3 ‣ Appendix B Literature Review: Evaluating LLMs with Game Theory ‣ How Far Are We on the Decision-Making of LLMs? Evaluating LLMs’ Gaming Ability in Multi-Agent Environments")中列出的使用经典游戏评估LLMs的论文外，研究人员还探索了涉及更复杂游戏的多种场景。最近的研究以复杂且具有欺骗性的Avalon游戏环境为测试平台，关注长期的多方对话 Stepputtis等人（[2023](https://arxiv.org/html/2403.11807v4#bib.bib58)）、社会行为 Lan等人（[2023](https://arxiv.org/html/2403.11807v4#bib.bib33)）、社会智能 Liu等人（[2024](https://arxiv.org/html/2403.11807v4#bib.bib39)）和递归思考 Wang等人（[2023](https://arxiv.org/html/2403.11807v4#bib.bib63)）在识别欺骗性信息方面的研究。其他论文则探讨了LLMs在沟通类游戏中的应用，例如狼人杀，重点关注无需调整的框架 Xu等人（[2023b](https://arxiv.org/html/2403.11807v4#bib.bib69)）和强化学习驱动的方法 Xu等人（[2024](https://arxiv.org/html/2403.11807v4#bib.bib70)）。O’Gara（[2023](https://arxiv.org/html/2403.11807v4#bib.bib46)）发现，先进的LLMs在文本类游戏《Hoodwinked》中表现出了欺骗和识别谎言的能力。与此同时，Liang等人（[2023](https://arxiv.org/html/2403.11807v4#bib.bib37)）评估了LLMs在猜词游戏《Who Is Spy?》中的智能和战略沟通能力。在水资源分配挑战游戏中，Mao等人（[2023](https://arxiv.org/html/2403.11807v4#bib.bib40)）构建了一个突显有限资源不平等竞争的场景。

### 5.2 游戏基准

另一类研究通过收集游戏来构建更为全面的基准，以评估LLMs的人工通用智能。Tsai等人（[2023](https://arxiv.org/html/2403.11807v4#bib.bib61)）发现，尽管LLMs，如ChatGPT，在文字游戏中表现竞争力，但它们在世界建模和目标推理方面存在困难。GameEval Qiao等人（[2023](https://arxiv.org/html/2403.11807v4#bib.bib53)）引入了三种目标驱动的对话游戏（Ask-Guess、SpyFall和TofuKingdom），有效评估LLMs在合作和对抗环境中的问题解决能力。MAgIC Xu等人（[2023a](https://arxiv.org/html/2403.11807v4#bib.bib68)）提出了基于概率图模型的方法，用于在多智能体游戏环境中评估LLMs。LLM-Co Agashe等人（[2023](https://arxiv.org/html/2403.11807v4#bib.bib1)）开发了LLM-Coordination框架，以评估LLMs在多智能体协调情境中的表现，展示了它们在合作伙伴意图推理和主动协助方面的能力。[Wu et al.](https://arxiv.org/html/2403.11807v4#bib.bib66)提出了SmartPlay，这是一个评估LLMs作为智能体在六款游戏中表现的基准，强调推理、规划和学习能力。这些论文专注于设计更为复杂的游戏，而我们的研究则探讨了博弈论中的八个经典且基础的游戏。

## 6 结论

本文提出了$\gamma$-Bench，一个旨在评估LLMs在多智能体环境中游戏能力的基准。$\gamma$-Bench包含八个经典的博弈论情境，强调多回合和多动作下的多玩家互动。我们的研究结果表明，GPT-3.5（0125）在$\gamma$-Bench上表现出有限的决策能力，但它可以通过学习历史结果来改进自身。通过精心设计的评分方案，我们观察到GPT-3.5（0125）在不同温度和提示下展现出了良好的稳健性。值得注意的是，像CoT这样的策略在此情境下证明是有效的。然而，它在多种游戏环境中的泛化能力仍然有限。最后，Gemini-1.5-Pro在所有测试模型中表现最佳，取得了$\gamma$-Bench排行榜的最高排名，开源的LLaMA-3.1-70B紧随其后。

## 限制

本研究存在几个局限性。首先，由于时间和预算的限制，我们没有评估所有显著的大型语言模型（LLMs），如LLaMA-3.2、Qwen-2.5 和 Claude-3.5。然而，我们承诺在未来扩大我们的排行榜，纳入更多LLMs。其次，我们的实验没有探索不同LLMs在同一博弈中的竞争场景。相反，我们的评估使用了来自同一LLM的十个代理。我们承认，在同一博弈中包含不同的LLMs可能会带来更有趣的见解，这一方面计划作为未来研究方向。第三，我们将博弈回合数限制为20轮，并告知代理总回合数，这可能会影响“背叛”博弈中的策略，在这些博弈中，代理可能最初合作并在最后一轮背叛以获得更大的收益。我们也将这一部分留作未来研究议程。然而，我们认为20轮足以观察代理行为模式。延长回合数将超出令牌限制，而不会带来新的观察结果，因为收敛趋势保持一致。

## 伦理声明与更广泛的影响

我们的研究旨在评估和增强LLMs的推理能力，以促进它们在决策场景中的应用。一方面，用户需要注意，目前的LLMs在决策中往往表现出自利行为，这可能无法最大化社会福利。另一方面，我们的框架通过促进人类与LLM的互动，推动社会福利，通过博弈可以应用于经济学和博弈论等教育领域。最终，增强LLMs的推理能力可以使它们成为有效的决策助手。

## 致谢

本文得到了中国香港特别行政区研究资助局（香港中文大学一般研究基金编号 CUHK 14206921）的资助。

## 参考文献

+   Agashe 等人（2023）Saaket Agashe、Yue Fan 和 Xin Eric Wang。评估大型语言模型中的多代理协调能力。*arXiv 预印本 arXiv:2310.03903*，2023年。

+   Aher 等人（2023）Gati V Aher、Rosa I Arriaga 和 Adam Tauman Kalai。利用大型语言模型模拟多个人类并复制人类实验研究。发表于 *国际机器学习会议*，第337-371页，PMLR，2023年。

+   Akata 等人（2023）Elif Akata、Lion Schulz、Julian Coda-Forno、Seong Joon Oh、Matthias Bethge 和 Eric Schulz。与大型语言模型进行重复博弈。*arXiv 预印本 arXiv:2305.16867*，2023年。

+   Arthur（1994）W Brian Arthur。归纳推理与有限理性。*美国经济评论*，84(2):406-411，1994年。

+   Ashlock & Greenwood（2016）Daniel Ashlock 和 Garrison Greenwood。广义“分配美元”问题。发表于 *2016年IEEE进化计算大会（CEC）*，第343-350页，IEEE，2016年。

+   Baidoo-Anu & Ansah (2023) David Baidoo-Anu 和 Leticia Owusu Ansah. 生成性人工智能（AI）时代的教育：理解 ChatGPT 在促进教学和学习中的潜在好处。*人工智能杂志*，7(1)：52-62，2023。

+   Brookins & DeBacker (2024) Philip Brookins 和 Jason DeBacker. 与 GPT 玩游戏：我们能从经典战略游戏中学到关于大语言模型的什么？ *经济学公报*，44(1)：25-37，2024。

+   Bubeck 等人 (2023) Sébastien Bubeck, Varun Chandrasekaran, Ronen Eldan, Johannes Gehrke, Eric Horvitz, Ece Kamar, Peter Lee, Yin Tat Lee, Yuanzhi Li, Scott Lundberg 等人. 人工通用智能的火花：关于 GPT-4 的早期实验。*arXiv 预印本 arXiv:2303.12712*, 2023。

+   Capraro 等人 (2023) Valerio Capraro, Roberto Di Paolo 和 Veronica Pizziol. 评估大语言模型预测人类如何平衡自利与他人利益的能力。*arXiv 预印本 arXiv:2307.12776*, 2023。

+   Chen 等人 (2023) Jiangjie Chen, Siyu Yuan, Rong Ye, Bodhisattwa Prasad Majumder 和 Kyle Richardson. 把你的钱拿出来：评估大语言模型代理在拍卖场景中的战略规划和执行。*arXiv 预印本 arXiv:2310.05746*, 2023。

+   Duan 等人 (2024) Jinhao Duan, Renming Zhang, James Diffenderfer, Bhavya Kailkhura, Lichao Sun, Elias Stengel-Eskin, Mohit Bansal, Tianlong Chen 和 Kaidi Xu. Gtbench：通过博弈论评估揭示大语言模型的战略推理局限性。*arXiv 预印本 arXiv:2402.12348*, 2024。

+   Dubey 等人 (2024) Abhimanyu Dubey, Abhinav Jauhri, Abhinav Pandey, Abhishek Kadian, Ahmad Al-Dahle, Aiesha Letman, Akhil Mathur, Alan Schelten, Amy Yang, Angela Fan 等人. Llama 3 模型群体。*arXiv 预印本 arXiv:2407.21783*, 2024。

+   Fan 等人 (2024) Caoyun Fan, Jindou Chen, Yaohui Jin 和 Hao He. 大语言模型能否作为博弈论中的理性玩家？一项系统分析。发表于 *人工智能会议论文集*，第 38 卷第 16 号，17960-17967 页，2024。

+   Glance & Huberman (1994) Natalie S Glance 和 Bernardo A Huberman. 社会困境的动态。*科学美国人*，270(3)：76-81，1994。

+   Goodin (1998) Robert E Goodin. *制度设计理论*。剑桥大学出版社，1998。

+   Guha 等人 (2023) Neel Guha, Julian Nyarko, Daniel E Ho, Christopher Re, Adam Chilton, Aditya Narayana, Alex Chohlas-Wood, Austin Peters, Brandon Waldon, Daniel Rockmore 等人. Legalbench：一个合作构建的用于衡量大语言模型法律推理能力的基准。发表于 *第 37 届神经信息处理系统会议 数据集与基准轨道*，2023。

+   Guo (2023) Fulin Guo. 游戏理论实验中的 GPT 代理。*arXiv 预印本 arXiv:2305.05516*, 2023。

+   郭等人（2023）郭佳贤、杨博、Paul Yoo、Bill Yuchen Lin、岩泽裕介、松尾丰。怀疑代理：通过具有心智理论意识的GPT-4进行不完全信息游戏。《arXiv预印本 arXiv:2309.17277》，2023。

+   赫达里与洛雷（2023）巴巴克·赫达里与努齐奥·洛雷。大型语言模型的战略行为：博弈结构与情境框架。《情境框架（2023年9月10日）》, 2023。

+   霍顿（2023）约翰·J·霍顿。大型语言模型作为模拟经济代理：我们能从“硅人类”中学到什么？技术报告，美国国家经济研究局，2023。

+   黄等人（2024a）黄建泽、林曼豪、李俊、任树杰、王文轩、焦文祥、屠兆鹏、吕纬明。漠不关心还是富有同情心？评估大型语言模型（LLMs）与人类的情感契合度。《神经信息处理系统进展 37》，2024a。

+   黄等人（2024b）黄建泽、王文轩、李俊、林曼豪、任树杰、袁有良、焦文祥、屠兆鹏、吕纬明。会话AI的人性：评估LLMs的心理刻画。《第十二届国际学习表征大会论文集》，2024b。

+   胡伯曼（1988）Bernardo A. 胡伯曼。《计算的生态学》。North-Holland，1988。

+   蒋等人（2024）Albert Q. 蒋、Alexandre Sablayrolles、Antoine Roux、Arthur Mensch、Blanche Savary、Chris Bamford、Devendra Singh Chaplot、Diego de las Casas、Emma Bou Hanna、Florian Bressand 等。专家混合模型。《arXiv预印本 arXiv:2401.04088》，2024。

+   焦等人（2023）焦文祥、王文轩、黄建泽、王星、屠兆鹏。ChatGPT是一个好的翻译器吗？一项初步研究。《arXiv预印本 arXiv:2301.08745》，2023。

+   约翰逊等人（2023）Douglas Johnson、Rachel Goodman、J·Patrinely、Cosby Stone、Eli Zimmerman、Rebecca Donald、Sam Chang、Sean Berkowitz、Avni Finn、Eiman Jahangir 等。评估AI生成的医学响应的准确性与可靠性：对Chat-GPT模型的评估。《Research square》，2023。

+   基尔高尔（1977）D·马克·基尔高尔。无限顺序三人决斗的均衡点。《国际博弈论杂志》，6:167–180，1977。

+   基尔高尔与布拉姆斯（1997）D·马克·基尔高尔与Steven J·布拉姆斯。三人决斗。《数学杂志》，70(5):315–326，1997。

+   基尔高尔（1975）D·马克·基尔高尔。顺序三人决斗。《国际博弈论杂志》，4:151–174，1975。

+   小岛等人（2022）小岛武、谷诗翔、Machel Reid、松尾丰、岩泽裕介。大型语言模型是零-shot推理者。《神经信息处理系统进展》，35:22199–22213，2022。

+   Kong 等人（2024）Aobo Kong、Shiwan Zhao、Hao Chen、Qicheng Li、Yong Qin、Ruiqi Sun、Xin Zhou、Enzhi Wang 和 Xiaohang Dong. 通过角色扮演提示改善零样本推理. 载于 *2024年北美计算语言学会会议：人类语言技术（卷1：长篇论文）*，第4099–4113页，2024年。

+   Kosinski（2023）Michal Kosinski. 心智理论可能在大语言模型中自发出现. *arXiv预印本arXiv:2302.02083*，2023年。

+   Lan 等人（2023）Yihuai Lan、Zhiqiang Hu、Lei Wang、Yang Wang、Deheng Ye、Peilin Zhao、Ee-Peng Lim、Hui Xiong 和 Hao Wang. 基于LLM的代理社会调查：在亚瑟王游戏中的合作与对抗. *arXiv预印本arXiv:2310.14985*，2023年。

+   Lanzi & Loiacono（2023）Pier Luca Lanzi 和 Daniele Loiacono. ChatGPT 和其他大语言模型作为在线互动协作游戏设计的进化引擎. 载于 *遗传与进化计算会议论文集*，第1383–1390页，2023年。

+   Ledoux（1981）Alain Ledoux. 比赛完整结果。受害者们喜欢出14号王牌。*Jeux & Stratégie*，2(10)：10–11，1981年。

+   Li 等人（2023）Jiatong Li、Rui Li 和 Qi Liu. 超越静态数据集：一种深度互动方法评估LLM. *arXiv预印本arXiv:2309.04369*，2023年。

+   Liang 等人（2023）Tian Liang、Zhiwei He、Jen-tes Huang、Wenxuan Wang、Wenxiang Jiao、Rui Wang、Yujiu Yang、Zhaopeng Tu、Shuming Shi 和 Xing Wang. 利用猜字游戏评估大语言模型的智能. *arXiv预印本arXiv:2310.20499*，2023年。

+   Liang 等人（2024）Tian Liang、Zhiwei He、Wenxiang Jiao、Xing Wang、Yan Wang、Rui Wang、Yujiu Yang、Zhaopeng Tu 和 Shuming Shi. 通过多主体辩论激励大语言模型的发散性思维. 载于 *2024年自然语言处理实证方法会议论文集*，2024年。

+   Liu 等人（2024）Ziyi Liu、Abhishek Anand、Pei Zhou、Jen-tse Huang 和 Jieyu Zhao. Interintent：通过互动游戏情境中的意图理解研究大语言模型的社会智能. 载于 *2024年自然语言处理实证方法会议论文集*，2024年。

+   Mao 等人（2023）Shaoguang Mao、Yuzhe Cai、Yan Xia、Wenshan Wu、Xun Wang、Fengyi Wang、Tao Ge 和 Furu Wei. Alympics：语言代理与博弈论相遇. *arXiv预印本arXiv:2311.03220*，2023年。

+   McAfee & McMillan（1987）R Preston McAfee 和 John McMillan. 拍卖与竞标. *经济文献杂志*，25(2)：699–738，1987年。

+   Myerson（2013）Roger B Myerson. *博弈论*. 哈佛大学出版社，2013年。

+   Nagel（1995）Rosemarie Nagel. 在猜测游戏中的解开：一项实验研究. *美国经济评论*，85(5)：1313–1326，1995年。

+   Nash（1950）John F Nash. N人博弈中的均衡点. *美国国家科学院院刊*，36(1)：48–49，1950年。

+   Nash (1951) John F Nash. 非合作博弈. *数学年刊*，54(2):286–295，1951。

+   O’Gara (2023) Aidan O’Gara. 被蒙蔽：文本游戏中的欺骗与合作对语言模型的影响. *arXiv预印本arXiv:2308.01404*，2023。

+   OpenAI (2022) OpenAI. 推出ChatGPT. *OpenAI博客 2022年11月30日*，2022。网址 [https://openai.com/index/chatgpt/](https://openai.com/index/chatgpt/)。

+   OpenAI (2023) OpenAI. GPT-4技术报告. *arXiv预印本arXiv:2303.08774*，2023。

+   Persky (1995) Joseph Persky. 回顾：经济人种性的行为学. *经济学视角杂志*，9(2):221–231，1995。

+   Phelps & Russell (2023) Steve Phelps 和 Yvan I Russell. 使用实验经济学研究大规模语言模型中涌现的目标行为. *arXiv预印本arXiv:2305.07970*，2023。

+   Pichai & Hassabis (2023) Sundar Pichai 和 Demis Hassabis. 推出Gemini：我们最大且最强大的AI模型. *Google博客 2023年12月6日*，2023。网址 [https://blog.google/technology/ai/google-gemini-ai/](https://blog.google/technology/ai/google-gemini-ai/)。

+   Pichai & Hassabis (2024) Sundar Pichai 和 Demis Hassabis. 我们的下一代模型：Gemini 1.5. *Google博客 2024年2月15日*，2024。网址 [https://blog.google/technology/ai/google-gemini-next-generation-model-february-2024/](https://blog.google/technology/ai/google-gemini-next-generation-model-february-2024/)。

+   Qiao et al. (2023) Dan Qiao, Chenfei Wu, Yaobo Liang, Juntao Li, 和 Nan Duan. Gameeval：评估LLM在对话游戏中的表现. *arXiv预印本arXiv:2308.10032*，2023。

+   Qin et al. (2023) Chengwei Qin, Aston Zhang, Zhuosheng Zhang, Jiaao Chen, Michihiro Yasunaga, 和 Diyi Yang. ChatGPT是否是一个通用的自然语言处理任务求解器？见 *2023年自然语言处理经验方法会议论文集*，第1339–1384页，2023。

+   Rubinstein (2007) Ariel Rubinstein. 本能与认知推理：反应时间的研究. *经济学期刊*，117(523):1243–1259，2007。

+   Samuelson (1954) Paul A Samuelson. 公共支出的纯理论. *经济学与统计评论*，36(4):387–389，1954。

+   Shapley & Shubik (1969) Lloyd S Shapley 和 Martin Shubik. 纯竞争、联盟力量与公平分配. *国际经济评论*，10(3):337–362，1969。

+   Stepputtis et al. (2023) Simon Stepputtis, Joseph P Campbell, Yaqi Xie, Zhengyang Qi, Wenxin Zhang, Ruiyi Wang, Sanketh Rangreji, Charles Lewis, 和 Katia Sycara. 在《亚瓦隆》游戏中使用大规模语言模型进行角色识别的长期对话理解. 见 *计算语言学协会年会成果：EMNLP 2023*，第11193–11208页，2023。

+   Stewart (1999) Ian Stewart. 海盗谜题. *科学美国人*，280(5):98–99，1999。

+   Surameery & Shakor (2023) Nigar M Shafiq Surameery 和 Mohammed Y Shakor. 使用ChatGPT解决编程错误. *国际信息技术与计算机工程期刊 (IJITC) ISSN: 2455-5290*，3(01):17–22，2023。

+   蔡等（2023）蔡晨风，周晓晨，刘思远，李晶，俞墨，梅洪源。大型语言模型能否玩得好文本游戏？当前的最前沿技术与未解之谜。*arXiv预印本arXiv:2304.02868*，2023年。

+   维克里（1961）威廉·维克里。反推测、拍卖和竞争性密封投标。*金融学期刊*，16（1）：8–37，1961年。

+   王等（2023）王申志，刘畅，郑子龙，齐思远，陈硕，杨启森，赵安德，王超飞，宋世基，黄高。阿瓦隆的思想游戏：通过递归思考与欺骗对抗。*arXiv预印本arXiv:2310.01320*，2023年。

+   魏等（2022）魏杰森，王学志，达尔·舒尔曼，马尔滕·博斯马，夏飞，艾德·奇，雷奎·V·李，邓尼·周等。思维链提示引发大型语言模型的推理能力。*神经信息处理系统进展*，35：24824–24837，2022年。

+   吴等（2023）吴浩然，王文轩，万宇轩，焦文祥，吕俊杰。ChatGPT还是Grammarly？在语法错误修正基准上评估ChatGPT。*arXiv预印本arXiv:2303.13648*，2023年。

+   吴等（2024）吴跃，唐璇，汤姆·M·米切尔，李元志。Smartplay：作为智能体的大型语言模型基准。发表于*第十二届国际学习表示大会*，2024年。

+   谢等（2024）谢成兴，陈灿宇，贾斐然，叶子宇，赖士扬，舒凯，顾金东，阿德尔·比比，胡子牛，戴维·杰根斯，詹姆斯·埃文斯，菲利普·托尔，伯纳德·加内姆，李国豪。大型语言模型智能体能模拟人类信任行为吗？*神经信息处理系统进展*，37，2024年。

+   许等（2023a）许琳，胡志远，周大泉，任洪宇，董震，库尔特·凯茨，冯家石。Magic：大型语言模型驱动的多智能体在认知、适应性、理性和协作方面的研究。*arXiv预印本arXiv:2311.08562*，2023a年。

+   许等（2023b）许宇壮，王硕，李鹏，罗福文，王晓龙，刘伟东，刘扬。探索大型语言模型在沟通游戏中的应用：狼人游戏的实证研究。*arXiv预印本arXiv:2309.04658*，2023b年。

+   许等（2024）许泽来，余超，方飞，王瑜，吴怡。带有强化学习的语言智能体在狼人游戏中的战略玩法。发表于*第41届国际机器学习大会论文集*，2024年。

+   杨等（2024）杨安，杨宝松，惠滨源，郑博，余博文，周昌，李程鹏，李承源，刘大一，黄飞等。Qwen2技术报告。*arXiv预印本arXiv:2407.10671*，2024年。

+   张等（2024）张亚东，毛少光，葛涛，王勋，夏岩，兰曼，魏赋如。大型语言模型的K级推理。*arXiv预印本arXiv:2402.01521*，2024年。

+   朱等人（2023）朱宇涛、袁华英、王树婷、刘经南、刘文翰、邓承龙、窦志成、温纪荣。大语言模型在信息检索中的应用：一项调查。*arXiv预印本arXiv:2308.07107*，2023。

###### 目录

1.  [1 引言](https://arxiv.org/html/2403.11807v4#S1 "在《LLM的决策能力：多智能体环境中的评估》中，提供了该研究的背景和目标")

1.  [2 游戏简介](https://arxiv.org/html/2403.11807v4#S2 "在《LLM的决策能力：多智能体环境中的评估》中，我们介绍了LLM的博弈能力")

    1.  [2.1 合作博弈](https://arxiv.org/html/2403.11807v4#S2.SS1 "在《游戏简介》中，我们介绍了合作博弈的基本概念")

    1.  [2.2 背叛博弈](https://arxiv.org/html/2403.11807v4#S2.SS2 "在《游戏简介》中，我们探讨了背叛博弈的概念")

    1.  [2.3 顺序博弈](https://arxiv.org/html/2403.11807v4#S2.SS3 "在《引言：LLM的决策能力》一节中，我们评估了LLM在多智能体环境中的博弈能力")

1.  [3 GAMA-Bench评分方案](https://arxiv.org/html/2403.11807v4#S3 "在《LLM的决策能力：多智能体环境中的评估》中，介绍了GAMA-Bench评分方案")

    1.  [3.1 合作博弈](https://arxiv.org/html/2403.11807v4#S3.SS1 "在《GAMA-Bench评分方案》一节中，我们分析了合作博弈的特点")

    1.  [3.2 背叛博弈](https://arxiv.org/html/2403.11807v4#S3.SS2 "在《GAMA-Bench评分方案》一节中，我们探讨了背叛博弈的影响")

    1.  [3.3 顺序博弈](https://arxiv.org/html/2403.11807v4#S3.SS3 "在《GAMA-Bench评分方案》一节中，我们讨论了LLM在多智能体环境中的博弈能力")

1.  [4 超越默认设置](https://arxiv.org/html/2403.11807v4#S4 "在《LLM的决策能力：多智能体环境中的评估》中，探索了超越默认设置的可能性")

    1.  [4.1 RQ1：鲁棒性](https://arxiv.org/html/2403.11807v4#S4.SS1 "在《超越默认设置》中，我们探讨了LLM的鲁棒性问题")

    1.  [4.2 RQ2：推理策略](https://arxiv.org/html/2403.11807v4#S4.SS2 "在《超越默认设置》中，我们评估了LLM在推理策略方面的能力")

    1.  [4.3 RQ3：泛化能力](https://arxiv.org/html/2403.11807v4#S4.SS3 "在《超越默认设置》中，我们评估了LLM的泛化能力")

    1.  [4.4 RQ4: 排行榜](https://arxiv.org/html/2403.11807v4#S4.SS4 "第4章 默认设置之外 ‣ 我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

1.  [5 相关工作](https://arxiv.org/html/2403.11807v4#S5 "我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

    1.  [5.1 特定游戏](https://arxiv.org/html/2403.11807v4#S5.SS1 "第5章 相关工作 ‣ 我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

    1.  [5.2 游戏基准](https://arxiv.org/html/2403.11807v4#S5.SS2 "第5章 相关工作 ‣ 我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

1.  [6 结论](https://arxiv.org/html/2403.11807v4#S6 "我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

1.  [A 博弈论更多信息](https://arxiv.org/html/2403.11807v4#A1 "我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

    1.  [A.1 表述](https://arxiv.org/html/2403.11807v4#A1.SS1 "附录A 博弈论更多信息 ‣ 我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

    1.  [A.2 纳什均衡](https://arxiv.org/html/2403.11807v4#A1.SS2 "附录A 博弈论更多信息 ‣ 我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

    1.  [A.3 人类行为](https://arxiv.org/html/2403.11807v4#A1.SS3 "附录A 博弈论更多信息 ‣ 我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

1.  [B 文献综述：通过博弈论评估LLMs](https://arxiv.org/html/2403.11807v4#A2 "我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

1.  [C 促使提示的细节](https://arxiv.org/html/2403.11807v4#A3 "我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

    1.  [C.1 设计方法论](https://arxiv.org/html/2403.11807v4#A3.SS1 "附录C 促使提示的细节 ‣ 我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

    1.  [C.2 合作游戏](https://arxiv.org/html/2403.11807v4#A3.SS2 "附录C 促使提示的细节 ‣ 我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

    1.  [C.3 背叛游戏](https://arxiv.org/html/2403.11807v4#A3.SS3 "附录C 促使提示的细节 ‣ 我们在LLMs的决策能力上走多远？评估LLMs在多智能体环境中的游戏能力")

    1.  [C.4 顺序博弈](https://arxiv.org/html/2403.11807v4#A3.SS4 "在附录C 提示详情 ‣ 我们在LLMs决策能力方面的进展：评估LLMs在多智能体环境中的游戏能力")

1.  [D GPT-4改写提示的示例](https://arxiv.org/html/2403.11807v4#A4 "我们在LLMs决策能力方面的进展：评估LLMs在多智能体环境中的游戏能力")

1.  [E 原始分数的重新缩放方法](https://arxiv.org/html/2403.11807v4#A5 "我们在LLMs决策能力方面的进展：评估LLMs在多智能体环境中的游戏能力")

1.  [F 详细结果](https://arxiv.org/html/2403.11807v4#A6 "我们在LLMs决策能力方面的进展：评估LLMs在多智能体环境中的游戏能力")

    1.  [F.1 鲁棒性：多次运行](https://arxiv.org/html/2403.11807v4#A6.SS1 "在附录F 详细结果 ‣ 我们在LLMs决策能力方面的进展：评估LLMs在多智能体环境中的游戏能力")

    1.  [F.2 鲁棒性：温度](https://arxiv.org/html/2403.11807v4#A6.SS2 "在附录F 详细结果 ‣ 我们在LLMs决策能力方面的进展：评估LLMs在多智能体环境中的游戏能力")

    1.  [F.3 鲁棒性：提示版本](https://arxiv.org/html/2403.11807v4#A6.SS3 "在附录F 详细结果 ‣ 我们在LLMs决策能力方面的进展：评估LLMs在多智能体环境中的游戏能力")

    1.  [F.4 推理策略](https://arxiv.org/html/2403.11807v4#A6.SS4 "在附录F 详细结果 ‣ 我们在LLMs决策能力方面的进展：评估LLMs在多智能体环境中的游戏能力")

    1.  [F.5 泛化能力](https://arxiv.org/html/2403.11807v4#A6.SS5 "在附录F 详细结果 ‣ 我们在LLMs决策能力方面的进展：评估LLMs在多智能体环境中的游戏能力")

    1.  [F.6 排行榜](https://arxiv.org/html/2403.11807v4#A6.SS6 "在附录F 详细结果 ‣ 我们在LLMs决策能力方面的进展：评估LLMs在多智能体环境中的游戏能力")

    1.  [F.7 GPT-3.5（0125）详细玩家行为](https://arxiv.org/html/2403.11807v4#A6.SS7 "在附录F 详细结果 ‣ 我们在LLMs决策能力方面的进展：评估LLMs在多智能体环境中的游戏能力")

1.  [G LLM与特定策略](https://arxiv.org/html/2403.11807v4#A7 "我们在LLMs决策能力方面的进展：评估LLMs在多智能体环境中的游戏能力")

## 附录A 关于博弈论的更多信息

### A.1 表述

博弈论涉及分析理性主体之间战略互动的数学模型，Myerson ([2013](https://arxiv.org/html/2403.11807v4#bib.bib42))。一个博弈可以通过以下关键元素建模：

1.  1.

    玩家，记作$\mathcal{P}=\{1,2,\cdots,N\}$：一个由$N$个参与者组成的集合。

1.  2.

    行动，表示为$\mathcal{A}=\{\mathcal{A}_{i}\}$：每个玩家可以选择的$N$个行动集。例如，$\mathcal{A}=\{\mathcal{A}_{1}=\{C,D\},\mathcal{A}_{2}=\{D,F\},\cdots,\mathcal{A}_{N}=\{C,F\}\}$

1.  3.

    效用函数，表示为$\mathcal{U}=\{\mathcal{U}_{i}\colon\times_{j=1}^{N}\mathcal{A}_{j}\mapsto \mathbb{R}\}$：一组$N$个函数，用于量化每个玩家对所有可能结果的偏好。

1.  4.

    信息，表示为$\mathcal{I}=\{\mathcal{I}_{i}\}$：每个玩家可以获得的$N$个信息集，包括其他玩家的行动集、效用函数、历史行动以及其他信念。

1.  5.

    顺序，表示为$\mathcal{O}=\mathcal{O}_{1},\mathcal{O}_{2},\cdots,\mathcal{O}_{k}$：指定$k$个步骤的$k$个集合的序列，用于采取行动。例如，$\mathcal{O}=\mathcal{P}$意味着所有玩家同时采取行动。

在这项研究中，多玩家游戏被定义为$|\mathcal{P}|>2$的游戏，因为博弈论模型至少有两个玩家。类似地，多行动游戏是指$\forall_{i\in\mathcal{P}}|\mathcal{A}_{i}|>2$的游戏。同时，多回合游戏指的是同一组玩家反复参与游戏，并记录所有先前的行动。同步游戏满足$k=1$的条件，而顺序游戏有$k>1$，这意味着玩家按特定顺序做出决策。完全信息游戏的特点是$\forall_{i,j\in\mathcal{P}|i\neq j}\mathcal{I}_{i}=\mathcal{I}_{j}$。由于每个玩家都可以看到自己的行动，上述条件表明所有玩家都能看到游戏中的完整信息集。相反，不满足这一标准的游戏被归类为不完全信息游戏，其中玩家对其他玩家的行动知之甚少。

### A.2 纳什均衡

研究博弈论模型通常涉及分析其纳什均衡（NE） Nash（[1950](https://arxiv.org/html/2403.11807v4#bib.bib44)）。NE是一个特定的策略集合，其中没有人能够通过仅改变自己的策略来获得任何收益。这意味着，给定一个玩家的选择，其他玩家的策略被限制在一个特定的集合中，这反过来又限制了原始玩家的选择只能维持在初始的策略中。当每个玩家的策略只有一个行动时，均衡被识别为纯策略纳什均衡（PSNE） Nash（[1950](https://arxiv.org/html/2403.11807v4#bib.bib44)）。然而，在某些博弈中，如石头剪子布，只有当玩家对自己的行动采取概率性方法时，NE才存在。这种均衡类型被称为混合策略纳什均衡（MSNE） Nash（[1951](https://arxiv.org/html/2403.11807v4#bib.bib45)），其中PSNE是MSNE的一个子集，概率集中在单一行动上。根据下述定理[A.1](https://arxiv.org/html/2403.11807v4#A1.Thmtheorem1 "Theorem A.1 (Nash’s Existence Theorem) ‣ A.2 Nash Equilibrium ‣ Appendix A More Information on Game Theory ‣ How Far Are We on the Decision-Making of LLMs? Evaluating LLMs’ Gaming Ability in Multi-Agent Environments")，我们可以分析每个博弈的NE，并评估LLMs的选择是否与NE一致。

###### 定理A.1（纳什存在定理）

每个玩家可以从有限的行动中选择的、有限数量玩家的博弈，至少有一个混合策略纳什均衡，其中每个玩家的行动由一个概率分布决定。

### A.3 人类行为

NE的实现假设参与者为经济人（Homo Economicus），他们始终理性且狭义自利，旨在最大化个人目标 Persky（[1995](https://arxiv.org/html/2403.11807v4#bib.bib49)）。然而，人类决策常常偏离这一理想。实证研究表明，人类的选择往往与NE的预测相背离 Nagel（[1995](https://arxiv.org/html/2403.11807v4#bib.bib43)）。这种偏差归因于人类决策的复杂性，涉及的不仅是理性分析，还有个人价值观、偏好、信仰和情感。通过比较先前研究中记录的人类决策模式，并结合NE，我们可以确定LLMs是否表现出更接近经济人或实际人类决策者的倾向，从而揭示它们与类人或纯理性决策过程的对齐程度。

## 附录B 文献综述：使用博弈论评估LLMs

通过博弈论模型评估大语言模型（LLM）已经成为一个热门的研究方向。最近的研究概述总结在表[3](https://arxiv.org/html/2403.11807v4#A2.T3 "Table 3 ‣ Appendix B Literature Review: Evaluating LLMs with Game Theory ‣ How Far Are We on the Decision-Making of LLMs? Evaluating LLMs’ Gaming Ability in Multi-Agent Environments")中。通过我们的分析，出现了几个关键的观察结果：（1）这些研究大多数集中在双人设置上。（2）研究主要集中在两种行动博弈上；特别是，一半的研究考察了囚徒困境和最后通牒博弈（独裁者博弈是最后通牒博弈的变体之一）。（3）文献中存在一个显著的空白，即缺乏对多个回合中的 LLM 决策与 MSNE 预测的行动概率分布之间的比较研究。（4）研究中使用的温度变量存在差异，这使得很难得出有关温度对 LLM 表现影响的定论。

表 3：使用博弈论模型评估大语言模型（LLM）的现有研究对比。T 表示每个实验中使用的温度。MP 表示多玩家设置，MR 表示多轮互动。Role 指示是否为 LLM 分配了特定角色。

| 论文 | 模型 | T | MP | MR | 角色 | CoT | 博弈 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Horton ([2023](https://arxiv.org/html/2403.11807v4#bib.bib20)) | text-davinci-003 | - | ✗ | ✗ | ✗ | ✗ | 独裁者博弈 |
| Guo ([2023](https://arxiv.org/html/2403.11807v4#bib.bib17)) | gpt-4-1106-preview | 1 | ✗ | ✓ | ✓ | ✓ | 最后通牒博弈， |
| 囚徒困境 |
| Phelps & Russell ([2023](https://arxiv.org/html/2403.11807v4#bib.bib50)) | gpt-3.5-turbo | 0.2 | ✗ | ✓ | ✓ | ✗ | 囚徒困境 |
| Akata et al. ([2023](https://arxiv.org/html/2403.11807v4#bib.bib3)) | text-davinci-003, | 0 | ✗ | ✓ | ✗ | ✗ | 囚徒困境， |
| gpt-3.5-turbo, gpt-4 | 性别之战 |
| Aher et al. ([2023](https://arxiv.org/html/2403.11807v4#bib.bib2)) | text-ada-001, text-babbage-001, | 1 | ✗ | ✗ | ✓ | ✗ | 最后通牒博弈 |
| text-curie-001, text-davinci-001, |
| text-davinci-002, text-davinci-003, |
| gpt-3.5-turbo, gpt-4 |
| Capraro et al. ([2023](https://arxiv.org/html/2403.11807v4#bib.bib9)) | ChatGPT-4, Bard, Bing Chat | - | ✗ | ✗ | ✗ | ✓ | 独裁者博弈 |
| （三种变体） |
| Brookins & DeBacker ([2024](https://arxiv.org/html/2403.11807v4#bib.bib7)) | gpt-3.5-turbo | 1 | ✗ | ✗ | ✗ | ✗ | 独裁者博弈， |
| 囚徒困境 |
| Li et al. ([2023](https://arxiv.org/html/2403.11807v4#bib.bib36)) | gpt-3.5-turbo-0613, | - | ✓ | ✓ | ✗ | ✗ | 公共物品博弈 |
| gpt-4-0613, claude-2.0, |
| chat-bison-001 |
| Heydari & Lorè ([2023](https://arxiv.org/html/2403.11807v4#bib.bib19)) | gpt-3.5-turbo-16k, gpt-4, | 0.8 | ✗ | ✗ | ✓ | ✓ | 囚徒困境， |
| llama-2 | 鹿群狩猎、雪崩博弈， |
| 囚徒愉快 |
| Guo et al. ([2023](https://arxiv.org/html/2403.11807v4#bib.bib18)) | gpt-3.5, gpt-4 | - | ✗ | ✓ | ✗ | ✓ | 乐都持有游戏 |
| Chen et al. ([2023](https://arxiv.org/html/2403.11807v4#bib.bib10)) | gpt-3.5-turbo-0613, | 0.7 | ✓ | ✓ | ✓ | ✓ | 英式拍卖 |
| gpt-4-0613, claude-instant-1.2, |
| claude-2.0, chat-bison-001 |
| Xu et al. ([2023a](https://arxiv.org/html/2403.11807v4#bib.bib68)) | gpt-3.5-turbo, gpt-4, | - | ✓ | ✓ | ✗ | ✓ | 成本共享, |
| llama-2-70b, claude-2.0, | 囚徒困境, |
| palm-2 | 公共物品博弈 |
| Fan et al. ([2024](https://arxiv.org/html/2403.11807v4#bib.bib13)) | text-davinci-003, | 0.7 | ✗ | ✓ | ✗ | ✗ | 独裁者游戏, |
| gpt-3.5-turbo, gpt-4 | 剪刀石头布, |
| 环形网络游戏 |
| Zhang et al. ([2024](https://arxiv.org/html/2403.11807v4#bib.bib72)) | gpt-4 | 0.7 | ✓ | ✓ | ✓ | ✓ | 猜测平均值的0.8 |
| 生存拍卖游戏 |
| Duan et al. ([2024](https://arxiv.org/html/2403.11807v4#bib.bib11)) | gpt-3.5-turbo, gpt-4, | 0.2 | ✓ | ✓ | ✗ | ✓ | 十种游戏^a |
| llama-2-70b, codellama-34b, |
| mistral-7b-orca |
| Xie et al. ([2024](https://arxiv.org/html/2403.11807v4#bib.bib67)) | text-davinci-003, | - | ✗ | ✓ | ✓ | ✓ | 七种游戏^b |
| gpt-3.5-turbo-instruct, |
| gpt-3.5-turbo-0613, gpt-4, |
| llama-2-(7/13/70)b, |
| vicuna-(7/13/33)b-v1.3 |
| 本研究 | gpt-3.5-turbo, gpt-4 | 0$\sim$1 | ✓ | ✓ | ✓ | ✓ | 八种游戏^c |
| gemini-pro |

+   a

    井字棋, 四连棋, 库恩扑克, 突破游戏, 撒谎骰子, 盲拍卖, 谈判, 尼姆游戏, 猪游戏, 迭代囚徒困境。

+   b

    信任游戏, 最低可接受概率信任游戏, 重复信任游戏, 独裁者游戏, 风险独裁者游戏, 彩票人游戏, 彩票赌博游戏。

+   c

    猜测平均值的2/3, El Farol酒吧, 分割美元, 公共物品博弈, 餐厅困境, 密封竞标拍卖, 大逃杀, 海盗游戏。

[返回到RQ1]

[返回到相关工作]

## 附录C 提示的详细信息

### C.1 设计方法学

我们采用了一个连贯的方法，以确保提示设计是系统性的，而非随意的。游戏描述来自经过验证的来源，包括第§[2](https://arxiv.org/html/2403.11807v4#S2 "2 引言 ‣ 我们在决策制定方面走得多远？评估LLMs在多智能体环境中的游戏能力")节中的学术论文和维基百科条目。利用这些描述，我们指示GPT-4生成提示，以指导LLMs参与指定的游戏。这些提示结构化地包含四个基本元素：游戏规则、玩家目标、宣布游戏结果的模板（用于显示历史结果）以及格式化响应的JSON指令。我们进行了人工检查，以确保GPT-4对游戏描述的理解是正确的。这些提示在本节的其余部分中详细描述。

### C.2 合作游戏

| 猜测平均值的2/3 |
| --- |
| 系统 | 你正在参与一个由 $N$ 名玩家进行的、持续 $K$ 轮的游戏。 |
|  | 游戏规则： |
|  | 1\. 每个玩家选择一个介于 $MIN$ 和 $MAX$ 之间的整数，包括两端。 |
|  | 2\. 所有选择完成后，会计算所有选择数字的平均值。 |
|  | 3\. 目标数字是该平均值的 $R$。 |
|  | 4\. 获胜者是选择的数字最接近目标数字的玩家。 |
|  | $\cdots$ |
| 用户 | 第 $I$ 轮游戏结果： |
|  | 平均选择的数字：$M_{I}$ |
|  | 目标数字（$R$ 的平均值）：$T_{I}$ |
|  | 获胜数字：$W_{I}$ |
|  | 你选择了： |
| 助手 | {“chosen_number”: “$C_{IJ}$”} |
| 用户 | [恭喜你赢了]/[很遗憾你输了]。 |
|  | $\cdots$ |
| 用户 | 现在开始第 $I$ 轮。 |
|  | 你的目标是选择一个你认为最接近所有玩家选择数字的平均值的数字，包括你的选择。 |
|  | 请按以下 JSON 格式提供你选择的数字： |
|  | {“chosen_number”: “介于_$MIN$_和_$MAX$之间的整数”}。 |

[返回游戏描述]

[返回原始实验]

| El Farol酒吧 |
| --- |
| 系统 | 你正在参与一个由 $N$ 名玩家进行的、持续 $K$ 轮的游戏。 |
|  | 游戏规则： |
|  | 1\. 每一轮，你和其他玩家独立决定是否去酒吧。 |
|  | 2\. 如果去酒吧的玩家少于或等于 $R$，那么所有去酒吧的玩家比待在家里更开心，获得的效用是 $MAX$。 |
|  | 3\. 如果超过 $R$ 的玩家去了酒吧，所有去酒吧的玩家比待在家里更不开心，获得的效用是 $MIN$。 |
|  | 4\. 所有待在家里的玩家将获得 $HOME$ 的效用。 |
|  | $\cdots$ |
| 用户 | 第 $I$ 轮游戏结果： |
|  | （仅针对显式）$G_{I}$ 名玩家去了酒吧，而 $S_{I}$ 名玩家待在家里。$G_{I}/N$ 是 [多于]/[等于或少于] $R$ 的玩家去了酒吧。 |
|  | 这一轮去酒吧比待在家里[更少]/[更多]开心。 |
|  | 你选择了： |
| 助手 | {“decision”: “$D_{IJ}$”} |
| 用户 | 你获得了 $G_{IJ}$。 |
|  | $\cdots$ |
| 用户 | 现在开始第 $I$ 轮。 |
|  | 你的目标是最大化你的乐趣。当你预测少于 $R$ 的玩家会去酒吧时，选择去酒吧；否则选择待在家里。 |
|  | 请按以下 JSON 格式提供你的决策： |
|  | {“decision”: “去_或_留”}。 |
| 分配美元 |
| --- |
| 系统 | 你正在参与一个由 $N$ 名玩家进行的、持续 $K$ 轮的游戏。 |
|  | 游戏规则： |
|  | 1\. 你正在分配 $G$ 个金币。每个玩家独立地提出一个出价。 |
|  | 2\. 如果所有出价的总和不超过 $G$，每个玩家将获得他们的出价金额。 |
|  | 3\. 如果所有出价的总和超过 $G$，所有玩家将什么都得不到。 |
|  | $\cdots$ |
| 用户 | 第 $I$ 轮游戏结果： |
|  | 你的出价金额是： |
| 助手 | {“bid_amount”: “$B_{IJ}$”} |
| 用户 | 所有出价的总和是 $S_{I}$。 |
|  | 总和 [不超过]/[超过] $G$。 |
|  | 你收到了 [$B_{IJ}$]/[$0$] 金币。 |
|  | $\cdots$ |
| 用户 | 现在第 $I$ 轮开始。 |
|  | 你的目标是最大化个人收益，同时不让所有竞标的总和超过 $G$ 金币。 |
|  | 请以以下 JSON 格式提供你的竞标金额： |
|  | {“bid_amount”: “介于 0 和 $G$ 之间的整数”}。 |

[返回游戏说明]

[返回香草实验]

### C.3 背叛游戏

| 公共物品博弈 |
| --- |
| --- |
| 系统 | 你正在参与一个由 $N$ 位玩家进行的 $K$ 轮游戏。 |
|  | 游戏规则： |
|  | 1\. 在每一轮中，作为玩家的你必须决定要悄悄贡献多少私人代币到公共池中。 |
|  | 2\. 这个池中的总代币将乘以系数 $R$ 来产生“公共物品”回报。 |
|  | 3\. 这笔回报将平分给所有玩家，不管他们的个人贡献如何。 |
|  | 4\. 任何你没有贡献的代币将保留在你的私人收藏中。 |
|  | $\cdots$ |
| 用户 | 第 $I$ 轮游戏结果： |
|  | 每位玩家贡献的代币：$C_{I1},C_{I2},\cdots,C_{IN}$ |
|  | 你贡献了： |
| 助手 | {“tokens_contributed”: “$C_{IJ}$”} |
| 用户 | 公共池中的代币：$S_{I}$ |
|  | 你的收益：$g_{IJ}$ |
|  | 你在第 $I$ 轮后的代币数：$T_{IJ}$ |
|  | 每位玩家在第 $I$ 轮后的代币数量：$T_{I1},T_{I2},\cdots,T_{IN}$ |
|  | $\cdots$ |
| 用户 | 现在第 $I$ 轮开始。 |
|  | 你的目标是在游戏结束时最大化你的总代币数。你目前有 $T_{I-1J}$ 个代币。你需要决定贡献到公共池的代币数量。 |
|  | 请以以下 JSON 格式提供代币数量： |
|  | {“tokens_contributed”: “介于 0 和 $T_{IJ}$ 之间的整数”} |
| 餐桌困境 |
| --- |
| 系统 | 你正在参与一个由 $N$ 位玩家进行的 $K$ 轮游戏。 |
|  | 游戏规则： |
|  | 1\. 每位玩家必须选择点一个昂贵的菜肴或便宜的菜肴。 |
|  | 2\. 昂贵菜肴的价格是 $P_{h}$，便宜菜肴的价格是 $P_{l}$。 |
|  | 3\. 昂贵的菜肴为你带来 $U_{h}$ 的效用，便宜的菜肴为你带来 $U_{l}$ 的效用。 |
|  | 4\. 昂贵的菜肴比便宜的更美味，但当单独就餐时，它的价格并不足以证明其价值。 |
|  | 5\. 每轮结束时，所有订单的菜肴总成本将平等分配给所有玩家。 |
|  | $\cdots$ |
| 用户 | 第 $I$ 轮游戏结果： |
|  | $N_{h}$ 人选择了昂贵的菜肴，而 $N_{l}$ 人选择了便宜的菜肴。 |
|  | 总成本是 $S_{I}$。你需要支付 $C_{I}$。 |
|  | 你选择了： |
| 助手 | {“chosen_dish”: “$D_{IJ}$”} |
| 用户 | 你的效用是 $u_{IJ}$。 |
|  | $\cdots$ |
| 用户 | 现在第 $I$ 轮开始。 |
|  | 你的目标是最大化整体满意度，平衡菜肴的质量和共享的成本。 |
|  | 请提供你选择的菜肴，格式如下 JSON： |
|  | {“chosen_dish”: “昂贵或便宜”} |

[返回游戏描述]

[返回经典实验]

| 密封投标拍卖 |
| --- |
| 系统 | 你正在参与一个由$N$名玩家进行的$K$轮游戏。 |
|  | 游戏规则： |
|  | 1\. 每个玩家在每轮中都有对物品的私人估值。 |
|  | 2\. 每个玩家在不知道其他玩家出价和估值的情况下，提交物品的书面出价。 |
|  | 3\. 出价最高者赢得物品，并支付[最高]/[第二高]的出价。 |
|  | 4\. 如果你获胜，你的这一轮效用为你的估值减去支付的价格。如果你失败，你的效用为零。 |
|  | $\cdots$ |
| 用户 | 第$I$轮游戏结果： |
|  | 你对这一轮物品的估值为$v_{IJ}$。 |
|  | 你的出价为： |
| 助手 | {“bid”: “$b_{IJ}$”} |
| 用户 | 最高出价为：$W_{I}$。 |
|  | 支付的价格为：$P_{I}$。 |
|  | 你[获胜]/[失败]。你的效用为[$u_{IJ}$]/[0]。 |
|  | $\cdots$ |
| 用户 | 第$I$轮开始。 |
|  | 你的目标是最大化总效用。你对这一轮物品的估值为$v_{IJ}$。 |
|  | 请按以下JSON格式提供您的出价： |
|  | {“bid”: “integer_between_0_and_$v_{IJ}$”} |

[返回游戏描述]

[返回经典实验]

### C.4 顺序游戏

| 大逃杀 |
| --- |
| 系统 | 你正在参与一个由$N$玩家进行的游戏。 |
|  | 游戏规则： |
|  | 1\. 你正在参与一个生存游戏，只有一个玩家可以存活并获胜。 |
|  | 2\. 玩家根据命中率按预定顺序轮流射击，顺序从最低到最高。 |
|  | 3\. 玩家按射击顺序排列的名字和命中率为：{“$ID_{1}$”: “$HIT_{1}$”, “$ID_{2}$”: “$HIT_{2}$”, $\cdots$, “$ID_{N}$”: “$HIT_{N}$”}。你是$ID_{J}$，你的命中率是$HIT_{J}$，你是第$RANK_{J}$位射击的玩家。 |
|  | 4\. 你有无限的子弹。 |
|  | 5\. 在你的回合，你可以选择故意错过射击。 |
|  | $\cdots$ |
| 用户 | 第$I$轮游戏结果： |
|  | 你的操作： |
| 助手 | (仅限玩家本人) {“target”: “$t_{IJ}$”} |
| 用户 | $NAME_{J}$ [故意没打中]/[射击$t_{IJ}$并命中]/[射击$t_{IJ}$但没打中]。 |
|  | 现在剩余$N_{I}$名玩家。 |
|  | $\cdots$ |
| 用户 | 第$I$轮开始。 |
|  | 你的目标是消除其他玩家，直到游戏结束并获胜。剩余玩家的名字和按射击顺序排列的命中率为：{“$ID_{1}$”: “$HIT_{1}$”, “$ID_{2}$”: “$HIT_{2}$”, $\cdots$, “$ID_{N}$”: “$HIT_{N}$”}。你是$ID_{J}$，你的命中率是$HIT_{J}$，你是第$RANK_{J}$位射击的玩家。请决定是否射击某个玩家或故意错过。 |
|  | 请按以下JSON格式提供您的操作： |
|  | {“target”: “playerID_or_null”} |

[返回游戏描述]

[返回经典实验]

| 海盗游戏 |
| --- |
| 系统 | 你正在参与一个由$N$玩家进行的游戏。 |
|  | 游戏规则： |
|  | 1\. 你们是找到$G$枚金币的海盗，正在决定如何在你们之间分配这些金币。 |
|  | 2\. 海盗们将严格按资历顺序做出决策。你是$RANK_{J}$个最资深的海盗。 |
|  | 3\. 最资深的海盗提出一个分配$G$枚金币的计划。 |
|  | 4\. 所有海盗，包括提议者，在提议的分配上投票。 |
|  | 5\. 如果大多数海盗接受该计划，每个海盗将按最资深海盗提出的方式获得金币。 |
|  | 6\. 如果大多数海盗拒绝该计划，提议者将被扔下船，下一位资深海盗提出新计划。 |
|  | 7\. 游戏在计划被接受或只剩一个海盗时结束。 |
|  | $\cdots$ |
| 用户 | 第$I$个最资深的海盗提出了一个计划，内容为{“$I$”: “$g_{II}$”, “$I+1$”: “$g_{II+1}$”, $\cdots$, “$I$”: “$g_{IN}$”}。 |
|  | $A_{I}$个海盗选择接受该分配。 |
|  | 你选择了： |
| 助手 | {“decision”: “$D_{I}J$”} |
| 用户 | 少于一半的海盗接受了该计划。 |
|  | 第$I$个最资深的海盗被扔下船，退出游戏。游戏继续进行。 |
|  | $\cdots$ |
| 用户 | 现在，第$I$个最资深的海盗需要提出一个计划。 |
|  | 你的主要目标是生存。如果你能生存下来，你的下一个目标是最大化你获得的金币数量。如果不影响其他目标，你也可能更愿意把另一个海盗扔下船。 |
| \hdashline对于投票者 | 提议的计划为{“$I$”: “$g_{II}$”, “$I+1$”: “$g_{II+1}$”, $\cdots$, “$I$”: “$g_{IN}$”}。你将从该计划中获得$g_{IJ}$金币。 |
|  | 请以以下JSON格式提供你对当前提议的决策： |
|  | {“decision”: “accept_or_reject”} |
| \hdashline对于提议者 | 你需要提出一个分配$G$金币的计划。提出的数字必须是非负整数，并且总和为$G$。 |
|  | 请提供你对金币分配的提议，格式为从你到第$I$个最资深海盗的以下JSON格式： |
|  | {”proposal”: {“$I$”: “$g_{II}$”, “$I+1$”: “$g_{II+1}$”, $\cdots$, “$I$”: “$g_{IN}$”}} |

[返回到游戏描述]

[返回到原始实验]

## 附录 D GPT-4 重新措辞的提示示例

§[4.1](https://arxiv.org/html/2403.11807v4#S4.SS1 "4.1 RQ1: Robustness ‣ 4 Beyond Default Settings ‣ How Far Are We on the Decision-Making of LLMs? Evaluating LLMs’ Gaming Ability in Multi-Agent Environments")涉及测试GPT-3.5 (0125)在不同提示模板下的鲁棒性。本节介绍了本次分析中使用的提示，即V2到V4版本的提示，V1为默认版本，具体内容请见§[C](https://arxiv.org/html/2403.11807v4#A3 "附录C 提示详情 ‣ 如何评估LLMs在多代理环境中的决策能力？")。我们只包括了游戏“猜测2/3的平均值”的提示，其他七个游戏的五个提示模板可以在我们的GitHub上找到（[https://github.com/CUHK-ARISE/GAMABench](https://github.com/CUHK-ARISE/GAMABench)）。

| 猜测2/3的平均值（V2） |
| --- |
| 系统 | 你正在参与一场有$N$名参与者的游戏，共有$K$轮。 |
|  | 游戏规则如下： |
|  | 1\. 每个参与者必须选择一个整数，范围在$MIN$和$MAX$之间，包含这两个数。 |
|  | 2\. 每个人选择数字后，计算这些数字的平均值。 |
|  | 3\. 目标数字为该平均数的$R$倍。 |
|  | 4\. 选择数字最接近目标数字的人或团队将被宣布为获胜者。 |
|  | $\cdots$ |
| 用户 | 第$I$轮游戏结果如下： |
|  | 选出的平均数是$M_{I}$ |
|  | 目标数字，即平均值的$R$倍，是$T_{I}$ |
|  | 获胜的数字是$W_{I}$。 |
|  | 你选择的数字是： |
| 助手 | {“chosen_number”: “$C_{IJ}$”} |
| 用户 | [恭喜你获胜]/[很遗憾你输了]。 |
|  | $\cdots$ |
| 用户 | 从第$I$轮开始。 |
|  | 你的目标是选择一个数字，按照你的判断，该数字将与所有玩家选择数字的总平均值的$R$倍最接近，包括你的选择。 |
|  | 请以以下JSON格式提供你选择的数字： |
|  | {“chosen_number”: “integer_between_$MIN$_and_$MAX$”}。 |

[返回到RQ1]

| 猜测2/3的平均值（V3） |
| --- |
| 系统 | 你正在参与一场有$N$名参与者的游戏，共有$K$轮。 |
|  | 游戏规则： |
|  | 1\. 每位玩家在$MIN$和$MAX$之间选择一个整数，包含这两个数。 |
|  | 2\. 每位玩家选择后，将根据所有选择的数字计算平均值。 |
|  | 3\. 平均数乘以$R$得到目标数字。 |
|  | 4\. 选择的数字最接近目标数字的个人或团队将被宣布为获胜者。 |
|  | $\cdots$ |
| 用户 | 第$I$轮游戏结果： |
|  | 选择数字的平均值：$M_{I}$ |
|  | 目标百分比（平均值的$R$倍）是：$T_{I}$ |
|  | 获胜的数字是：$W_{I}$。 |
|  | 你选择了： |
| 助手 | {“chosen_number”: “$C_{IJ}$”} |
| 用户 | [恭喜您获胜]/[遗憾您失败]。 |
|  | $\cdots$ |
| 用户 | 第$I$轮现已开始。 |
|  | 目标是选择一个您认为最接近$R$倍所有玩家（包括您自己）选择的数字平均数的数字。 |
|  | 请以以下JSON格式提供您选择的数字： |
|  | {“chosen_number”: “integer_between_$MIN$_and_$MAX$”}。 |
| 猜测平均数的2/3（V4） |
| --- |
| 系统 | 您正在参与一个由$N$名玩家组成、共$K$轮的游戏。 |
|  | 游戏规则如下： |
|  | 1\. 所有玩家必须选择一个介于$MIN$和$MAX$之间的整数，包括这两个数字。 |
|  | 2\. 所有选择的数字将被收集，并计算它们的平均数。 |
|  | 3\. 目标数字，或称为目标数，是$R$倍计算出的平均数。 |
|  | 4. 获胜的玩家是那些选择的数字最接近目标数字的玩家。 |
|  | $\cdots$ |
| 用户 | 第$I$轮的结果如下： |
|  | 选择的平均数是$M_{I}$。目标数字是$R$倍的平均数，它是$T_{I}$。获胜的数字是$W_{I}$。 |
|  | 您选择的数字是： |
| 助手 | {“chosen_number”: “$C_{IJ}$”} |
| 用户 | [恭喜您获胜]/[遗憾您失败]。 |
|  | $\cdots$ |
| 用户 | 第$I$轮现已开始。 |
|  | 您的任务是选择一个数字，您认为它将最接近$R$倍所有玩家（包括您自己的选择）选择的数字平均数。 |
|  | 请以以下JSON格式提供您选择的数字： |
|  | {“chosen_number”: “integer_between_$MIN$_and_$MAX$”}。 |

[返回到RQ1]

| 猜测平均数的2/3（V5） |
| --- |
| 系统 | 您将参与一个包含$N$名玩家、共$K$轮的游戏。 |
|  | 游戏说明： |
|  | 1\. 每个玩家必须选择一个在$MIN$和$MAX$之间的整数，包括这两个数字。 |
|  | 2\. 所有玩家选择的数字的中位数将在所有选择完成后确定。 |
|  | 3\. 玩家们的目标数字是$R$倍计算出的平均数。 |
|  | 4\. 选择最接近该目标数字的玩家将被宣布为获胜者。 |
|  | $\cdots$ |
| 用户 | 第$I$轮游戏结果： |
|  | 选择的平均数字是：$M_{I}$ |
|  | 目标数字（$R$倍平均数）是：$T_{I}$ |
|  | 获胜的数字是：$W_{I}$。 |
|  | 您的选择是： |
| 助手 | {“chosen_number”: “$C_{IJ}$”} |
| 用户 | [恭喜您获胜]/[遗憾您失败]。 |
|  | $\cdots$ |
| 用户 | 第$I$轮现已开始。 |
|  | 您的任务是选择一个数字，您认为它将最接近$R$倍所有玩家选择的数字平均数（包括您自己的选择）。 |
|  | 请以以下JSON格式提供您选择的数字： |
|  | {“chosen_number”: “integer_between_$MIN$_and_$MAX$”}. |

[返回至 RQ1]

## 附录E 原始分数的重新调整方法

跨游戏的原始分数缺乏一致性。在某些游戏中，较高的分数表示更好的表现，而在其他游戏中，较低的分数更为理想。此外，分数范围因游戏而异，并且可能会随不同的游戏参数发生变化。为了在$\gamma$-Bench上标准化分数，我们将原始分数重新调整到0到100的范围，其中较高的分数始终表示更好的表现。具体的评分方案见公式[1](https://arxiv.org/html/2403.11807v4#A5.E1 "附录E 原始分数的重新调整方法 ‣ 我们在LLM决策能力方面走得多远？评估LLM在多智能体环境中的游戏能力")。

|  | $\begin{split}S_{1}&=\begin{cases}\frac{MAX-S_{1}}{MAX-MIN}*100,&R<1\\ \frac{\lvert 2S_{1}-(MAX-MIN)\rvert}{MAX-MIN}*100,&R=1\\ \frac{S_{1}}{MAX-MIN}*100,&R>1\end{cases},\\ S_{2}&=\frac{\max(R,1-R)-S_{2}}{\max(R,1-R)}*100,\\ S_{3}&=\frac{G-S_{3}}{G}*100,\\ S_{4}&=\frac{T-S_{4}}{T}*100,\\ S_{5}&=(1-S_{5})*100,\\ S_{6}&=\frac{S_{6}}{\max_{ij}v_{ij}}*100,\\ S_{7}&=S_{7}*100,\\ S_{8}&=\frac{2*G-S_{8P}}{2*G}*50+S_{8V}*50.\\ \end{split}$ |  | (1) |
| --- | --- | --- | --- |

[返回至原始实验]

## 附录F 详细结果

本节展示了§[4](https://arxiv.org/html/2403.11807v4#S4 "4 超越默认设置 ‣ 我们在LLM决策能力方面走得多远？评估LLM在多智能体环境中的游戏能力")的定量和可视化结果，并包括§[3](https://arxiv.org/html/2403.11807v4#S3 "3 GAMA-Bench评分方案 ‣ 我们在LLM决策能力方面走得多远？评估LLM在多智能体环境中的游戏能力")中GPT-3.5 (0125)实验的玩家行为图。

### F.1 鲁棒性：多次运行

表 4：在相同设置下玩游戏五次的定量结果。

| 测试 | T1 (默认) | T2 | T3 | T4 | T5 | $Avg_{\pm Std}$ |
| --- | --- | --- | --- | --- | --- | --- |
| 猜测 2/3 的平均值 | $65.4$ | $62.3$ | $63.9$ | $58.3$ | $67.3$ | $63.4_{\pm 3.4}$ |
| El Farol 酒吧 | $73.3$ | $67.5$ | $68.3$ | $67.5$ | $66.7$ | $68.7_{\pm 2.7}$ |
| 分割美元 | $68.1$ | $67.7$ | $68.7$ | $66.0$ | $72.6$ | $68.6_{\pm 2.4}$ |
| 公共物品博弈 | $41.3$ | $25.4$ | $45.7$ | $38$ | $44.0$ | $38.8_{\pm 8.1}$ |
| 餐厅困境 | $4.0$ | $3.5$ | $0.0$ | $6.5$ | $0.0$ | $2.8_{\pm 2.8}$ |
| 密封投标拍卖 | $6.5$ | $6.5$ | $4.6$ | $5.5$ | $4.4$ | $5.5_{\pm 1.0}$ |
| Battle Royale | $20.0$ | $21.4$ | $46.7$ | $23.5$ | $31.3$ | $28.6_{\pm 11.0}$ |
| 海盗游戏 | $80.6$ | $71.2$ | $72.0$ | $74.7$ | $59.5$ | $71.6_{\pm 7.7}$ |
| 总体 | $44.9$ | $40.7$ | $46.2$ | $42.5$ | $43.2$ | $43.5_{\pm 2.1}$ |

![参见图注](img/2936edec295c4a44c26bd74059751948.png)

图 4：在相同设置下玩游戏五次的结果。

[返回至 RQ1]

### F.2 鲁棒性：温度

表 5：在温度参数范围从 $0$ 到 $1$ 之间玩游戏的定量结果。

| 温度 | 0.0 | 0.2 | 0.4 | 0.6 | 0.8 | 1.0（默认） | $Avg_{\pm Std}$ |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 猜 2/3 的平均数 | $48.0$ | $50.0$ | $49.8$ | $54.7$ | $61.7$ | $65.4$ | $54.9_{\pm 7.1}$ |
| El Farol 酒吧 | $55.8$ | $71.7$ | $63.3$ | $68.3$ | $69.2$ | $73.3$ | $66.9_{\pm 6.4}$ |
| 分配美元 | $69.3$ | $67.0$ | $67.7$ | $67.9$ | $72.8$ | $68.1$ | $68.8_{\pm 2.1}$ |
| 公共物品游戏 | $15.3$ | $10.8$ | $17.9$ | $18.0$ | $36.5$ | $41.3$ | $23.3_{\pm 12.5}$ |
| 餐馆困境 | $0.0$ | $0.0$ | $0.0$ | $0.0$ | $0.0$ | $4.0$ | $0.7_{\pm 1.6}$ |
| 密封竞标拍卖 | $5.6$ | $6.0$ | $4.6$ | $4.5$ | $5.8$ | $6.5$ | $5.5_{\pm 0.8}$ |
| 大逃杀 | $28.6$ | $26.7$ | $46.7$ | $15.0$ | $33.3$ | $20.0$ | $28.4_{\pm 11.1}$ |
| 海盗游戏 | $75.0$ | $53.9$ | $77.7$ | $83.8$ | $59.5$ | $80.6$ | $71.8_{\pm 12.2}$ |

| 总体 | $37.2$ | $35.7$ | $40.9$ | $39.0$ | $42.3$ | $44.9$ | $40.0_{\pm 3.4}$ | ![参见标题](img/86c13cfd2f5c8659ed8016ced8f891e4.png)

图 5：在温度参数范围从 $0$ 到 $1$ 之间玩游戏的结果。

[返回到 RQ1]

### F.3 鲁棒性：提示版本

表 6：使用不同提示模板玩游戏的定量结果。

| 提示版本 | V1（默认） | V2 | V3 | V4 | V5 | $Avg_{\pm Std}$ |
| --- | --- | --- | --- | --- | --- | --- |
| 猜 2/3 的平均数 | $65.4$ | $66.4$ | $47.9$ | $66.9$ | $69.7$ | $63.3_{\pm 8.7}$ |
| El Farol 酒吧 | $73.3$ | $75.8$ | $65.8$ | $75.8$ | $71.7$ | $72.5_{\pm 4.1}$ |
| 分配美元 | $68.1$ | $81.0$ | $91.5$ | $75.8$ | $79.7$ | $79.2_{\pm 8.5}$ |
| 公共物品游戏 | $41.3$ | $26.6$ | $45.2$ | $50.2$ | $24.2$ | $37.5_{\pm 11.5}$ |
| 餐馆困境 | $4.0$ | $3.5$ | $0.0$ | $57.0$ | $18.5$ | $16.6_{\pm 23.7}$ |
| 密封竞标拍卖 | $6.5$ | $4.6$ | $5.7$ | $2.6$ | $5.9$ | $5.0_{\pm 1.6}$ |
| 大逃杀 | $20.0$ | $30.8$ | $15.0$ | $25.0$ | $18.8$ | $21.9_{\pm 6.1}$ |
| 海盗游戏 | $80.6$ | $87.9$ | $60.8$ | $60.5$ | $53.7$ | $68.7_{\pm 14.7}$ |

![参见标题](img/675c29a22a846bbc43b5ff63ee2f0e79.png)

图 6：使用不同提示模板玩游戏的结果。

[返回到 RQ1]

### F.4 推理策略

表 7：使用基于提示的改进方法玩游戏的定量结果。

| 改进措施 | 默认 | CoT | 合作 | 自私 | 数学家 |
| --- | --- | --- | --- | --- | --- |
| 猜 2/3 的平均数 | $65.4$ | $75.1$ | $69.0$ | $14.5$ | $71.4$ |
| El Farol 酒吧 | $73.3$ | $71.7$ | $74.2$ | $63.3$ | $60.0$ |
| 分配美元 | $68.1$ | $83.4$ | $70.7$ | $49.7$ | $69.2$ |
| 公共物品游戏 | $41.3$ | $56.1$ | $32.4$ | $37.4$ | $25.6$ |
| 餐馆困境 | $31.0$ | $82.5$ | $0.0$ | $17.5$ | $47.0$ |
| 密封竞标拍卖 | $6.5$ | $1.4$ | $7.4$ | $4.0$ | $5.8$ |
| 大逃杀 | $20.0$ | $17.6$ | $6.3$ | $33.3$ | $26.7$ |
| 海盗游戏 | $80.6$ | $71.2$ | $80.6$ | $74.7$ | $59.5$ |
| 总体 | $44.9$ | $57.4$ | $42.6$ | $36.8$ | $45.6$ |

![请参见标题](img/42388253e2c09ae28bd6544499cd5871.png)

图 7：使用基于提示的改进方法玩游戏的结果。

[返回到 RQ2]

### F.5 泛化能力

表 8：使用不同游戏设置进行游戏的定量结果。

| 猜测 2/3 的平均值 | $Avg_{\pm Std}$ |
| --- | --- |
| \hdashline$R=$ | $0$ | $1/6$ | $1/3$ | $1/2$ | $2/3$ | $5/6$ | $1$ | $7/6$ | $4/3$ | $3/2$ | $5/3$ | $11/6$ | $2$ |  |
|  | $79.1$ | $61.7$ | $66.6$ | $65.4$ | $65.4$ | $54.8$ | $62.4$ | $70.0$ | $74.9$ | $65.9$ | $67.3$ | $63.3$ | $73.6$ | $67.0_{\pm 6.3}$ |
| El Farol Bar | $Avg_{\pm Std}$ |
| --- | --- |
| \hdashline$R=$ | $0\%$ | $20\%$ | $40\%$ | $60\%$ | $80\%$ | $100\%$ |  |
|  | $53.5$ | $61.3$ | $63.3$ | $73.3$ | $68.1$ | $60.0$ | $63.3_{\pm 6.9}$ |
| 分割美元 | $Avg_{\pm Std}$ |
| --- | --- |
| \hdashline$G=$ | $50$ | $100$ | $200$ | $400$ | $800$ |  |
|  | $73.2$ | $68.1$ | $82.5$ | $82.1$ | $80.7$ | $77.3_{\pm 6.4}$ |
| 公共物品游戏 | $Avg_{\pm Std}$ |
| --- | --- |
| \hdashline$R=$ | $0.0$ | $0.5$ | $1.0$ | $2.0$ | $4.0$ |  |
|  | $42.0$ | $29.0$ | $52.5$ | $41.3$ | $25.9$ | $38.1_{\pm 10.8}$ |
| 餐厅困境 | $Avg_{\pm Std}$ |
| --- | --- |
| \hdashline$(P_{l},U_{l},P_{h},U_{h})=$ | $(10,15,20,20)$ | $(11,5,20,7)$ | $(4,19,9,20)$ | $(1,8,19,12)$ | $(4,5,17,7)$ | $(2,11,8,13)$ |  |
|  | $4.0$ | $2.5$ | $4.5$ | $13.5$ | $0.0$ | $12.0$ | $6.1_{\pm 5.4}$ |
| 封闭竞标拍卖 | $Avg_{\pm Std}$ |
| --- | --- |
| \hdashline$Range=$ | $(0,100]$ | $(0,200]$ | $(0,400]$ | $(0,800]$ |  |
|  | $6.0$ | $5.1$ | $4.7$ | $5.5$ | $5.3_{\pm 0.6}$ |
| 大逃杀 | $Avg_{\pm Std}$ |
| --- | --- |
| \hdashline$Range=$ | $[51,60]$ | $[35,80]$ | $[10,100]$ |  |
|  | $28.6$ | $20.0$ | $33.3$ | $27.3_{\pm 6.8}$ |
| 海盗游戏 | $Avg_{\pm Std}$ |
| --- | --- |
| \hdashline$G=$ | $4$ | $5$ | $100$ | $400$ |  |
|  | $73.8$ | $47.1$ | $80.6$ | $83.6$ | $71.3_{\pm 16.6}$ |

![请参见标题](img/a9b59d980ada385ef49dc82a17d1fc34.png)

图 8：使用不同游戏设置玩游戏的结果。

[返回到 RQ3]

### F.6 排行榜

![请参见标题](img/b94e75b5d07fa677e82fb425f217c75b.png)

图 9：使用不同闭源 LLM 玩游戏的结果。

![请参见标题](img/b0292e77ad13dbcaf15d2429ea125cc8.png)

图 10：使用不同开源 LLM 玩游戏的结果。

[返回到 RQ4]

### F.7 GPT-3.5（0125）详细玩家行为

![请参见标题](img/bbf176a8b5523673d89f05ab09537198.png)

图 11：合作与背叛游戏中的玩家行为。

## 附录 G LLM 与特定策略对比

![请参见标题](img/9d59ed1706a1e4594456c4c9920ad7a4.png)

图 12：GPT-3.5（0125）与两种固定策略对抗时，在“分割美元”和“公共物品游戏”中的表现。

我们的框架支持LLM和人类之间的并行互动，使我们能够研究LLM在与使用固定策略的对手互动时的行为。有许多可能的策略，在这里我们使用两个例子：首先，我们让一位玩家在“(3) 分金游戏”中始终出价$91$金，迫使所有其他参与者只出一金。其目标是确认LLM代理是否会针对主导参与者调整策略。此外，我们还研究了代理对于一个持续的搭便车者在“(4) 公共物品博弈”中的反应，目的是判断代理是否会识别并调整与搭便车者的合作关系。我们在图[12](https://arxiv.org/html/2403.11807v4#A7.F12 "Figure 12 ‣ Appendix G LLM vs. Specific Strategies ‣ How Far Are We on the Decision-Making of LLMs? Evaluating LLMs’ Gaming Ability in Multi-Agent Environments")中绘制了九个代理的平均出价和贡献的代币。我们发现，代理会在“(3) 分金游戏”中响应主导策略而降低出价。与预期相反，在“(4) 公共物品博弈”中，代理会增加贡献，以弥补搭便车者带来的不足。
