<!--yml

类别：未分类

日期：2025-01-11 12:24:23

-->

# 基于Minecraft建造者对话代理任务的LLM基准

> 来源：[https://arxiv.org/html/2407.12734/](https://arxiv.org/html/2407.12734/)

Chris Madge

伦敦玛丽女王大学

c.j.madge@qmul.ac.uk

\AndMassimo Poesio

伦敦玛丽女王大学

m.poesio@qmul.ac.uk

###### 摘要

在本研究中，我们提出将Minecraft建造任务改编为适合评估LLM在空间导向任务中能力的基准，并为建造代理设计提供参考。以往的研究提出了具有不同复杂结构和人类编写指令的语料库。我们则尝试提供一个全面的合成基准，用于在一系列不同的任务中测试建造代理，这些任务包括常见的建筑操作。我们认为这种方法使我们能够探究不同代理的特定优缺点，并测试LLM在空间推理和基于向量的数学这一挑战性领域的能力。

## 1 引言

能够在虚拟世界环境中操作的对话代理的开发，长期以来一直是人工智能领域的研究兴趣所在，Winograd（[1972](https://arxiv.org/html/2407.12734v1#bib.bib20)）。近年来，这项研究的重点是开发能够在游戏环境中操作的代理。游戏环境为研究任务导向的对话代理提供了一个理想的沙箱，Szlam 等人（[2019](https://arxiv.org/html/2407.12734v1#bib.bib16)）指出，这激发了多个平台的开发，以便进行此类研究，Johnson 等人（[2016](https://arxiv.org/html/2407.12734v1#bib.bib5)）；Urbanek 等人（[2019](https://arxiv.org/html/2407.12734v1#bib.bib17)）；Callison-Burch 等人（[2022](https://arxiv.org/html/2407.12734v1#bib.bib2)）；Gray 等人（[2019](https://arxiv.org/html/2407.12734v1#bib.bib3)）；Ogawa 等人（[2020](https://arxiv.org/html/2407.12734v1#bib.bib13)）；Köhn 等人（[2020](https://arxiv.org/html/2407.12734v1#bib.bib7)），数据收集练习Narayan-Chen 等人（[2019](https://arxiv.org/html/2407.12734v1#bib.bib12)）；Jayannavar 等人（[2020](https://arxiv.org/html/2407.12734v1#bib.bib4)）；Mohanty 等人（[2022](https://arxiv.org/html/2407.12734v1#bib.bib11)）以及竞赛Kiseleva 等人（[2022](https://arxiv.org/html/2407.12734v1#bib.bib6)）。

本研究的目标是提出一个类似的数据集合成基准，用于测试LLM在基于文本的空间推理和向量数学上的表现。现有的研究设计了一系列基准测试，旨在评估LLM在超出普通标记预测范围的任务中的表现，Srivastava等人([2022](https://arxiv.org/html/2407.12734v1#bib.bib15))；Wu等人([2023](https://arxiv.org/html/2407.12734v1#bib.bib21))。然而，尽我们所知，空间推理的要求并不常见，而且不涉及3D结构的构建。在LLM基准测试之前，已有其他任务被提议用于测试基于文本的空间推理，然而这些任务不太可能激发出本任务所需的组合向量数学、消歧义或结构要求，Weston等人([2015](https://arxiv.org/html/2407.12734v1#bib.bib19))；Shi等人([2022](https://arxiv.org/html/2407.12734v1#bib.bib14))；Mirzaee和Kordjamshidi ([2022](https://arxiv.org/html/2407.12734v1#bib.bib10))。

我们的基准测试灵感来源于Jayannavar等人提出的虚拟世界环境“Minecraft Builder Task”([2020](https://arxiv.org/html/2407.12734v1#bib.bib4))，在该环境中，建筑工人需要根据建筑师提供的基于文本的指令进行操作，以完成一座结构，而无法看到目标结构。之前的研究曾探讨了在这一设置中使用LLM，Madge和Poesio ([2024](https://arxiv.org/html/2407.12734v1#bib.bib9))；Kranti等人([2024](https://arxiv.org/html/2407.12734v1#bib.bib8))，尽管表现看起来很有前景，但空间推理和向量数学仍然是LLM面临的挑战性任务，Bang等人([2023](https://arxiv.org/html/2407.12734v1#bib.bib1))。

除了作为一个有趣的基准，评估LLM在超越文本任务的不断进化能力外，我们希望这也能为建筑工人代理设计者提供有关他们方法的具体优缺点的启示。在浏览这些数据集时，我们发现了一些常见的模式，并根据这些模式产生了相应的测试场景。

除了提出这个基准，我们还提供了对测试这些基准时的初步讨论，涉及到*Llama-3-70b-Instruct*的使用、我们解决这些挑战的方法以及对这些方法的评估。

## 2 我们的方法

之前的语料库通常表现为表示物体的形状。然而，似乎结构所代表的物体的最终描述在传达所需结构方面几乎没有作用。我们识别了用于提供指令的常见模式，并采取基于规则的方法，为建筑工人生成建筑师指令，指引其围绕这些模式的不同区块排列进行操作。

为了验证我们的基准测试，我们将其应用于几种不同的提示方法。我们采用了零-shot方法、少-shot方法，最后是链式思维（Chain of Thought），Wei等人([2022](https://arxiv.org/html/2407.12734v1#bib.bib18))。

在本节进一步描述我们的方法时，我们通过之前的一个语料库中的现有示例来进行说明，参考了Narayan-Chen等人（[2019](https://arxiv.org/html/2407.12734v1#bib.bib12)）的研究。自然地，表示一个物体的体素形式有多种方式，并且由于它是体素形式的表示，这种表示在某种程度上是抽象的，可能不会显现出双方所指的对象是什么（例如[A.1](https://arxiv.org/html/2407.12734v1#A1.SS1 "A.1 B1-A3-C8-1522432497234 ‣ 附录 A 附录 ‣ 基于Minecraft建造者对话代理任务的LLM基准测试")）。当最终标签被使用时，通常是建造者在对话结束时验证架构师指令，而不是架构师作为指令的一部分使用（例如[A.2](https://arxiv.org/html/2407.12734v1#A1.SS2 "A.2 B1-A3-C4-1522432009099 ‣ 附录 A 附录 ‣ 基于Minecraft建造者对话代理任务的LLM基准测试")）。当结构被类比为一个物体时，几乎总是伴随有具体的逐块指令，而不是孤立使用（例如[A.3](https://arxiv.org/html/2407.12734v1#A1.SS3 "A.3 B1-A3-C1-1522435497386 ‣ 附录 A 附录 ‣ 基于Minecraft建造者对话代理任务的LLM基准测试")）。

我们发现更常见的情况是，指令通常采取以下三种形式之一，我们将在后续小节中进行讨论。

### 2.1 绝对寻址

在任务对话的开始阶段，或者在创建新的独立子结构时，架构师需要在网格中引用一个空间，而不使用现有的参考点，因此引用的范围仅限于网格本身，例如[A.4](https://arxiv.org/html/2407.12734v1#A1.SS4 "A.4 B3-A2-C12-1522445699382 ‣ 附录 A 附录 ‣ 基于Minecraft建造者对话代理任务的LLM基准测试")。我们称之为绝对寻址。为了对这一能力进行基准测试，我们设计了一个测试，要求代理在网格的前三个层级的每个位置放置一个方块。

### 2.2 相对寻址

相对地址可能是最常见的一种类型，通常在对话中根据现有的方块位置给出（例如[A.5](https://arxiv.org/html/2407.12734v1#A1.SS5 "A.5 B3-A2-C23-1522447244858 ‣ 附录 A 附录 ‣ 基于Minecraft建造者对话代理任务的LLM基准测试")）。为了测试这一点，我们要求建造者在每个与现有方块相邻的方向上放置一个方块（如图[1](https://arxiv.org/html/2407.12734v1#S2.F1 "图 1 ‣ 2.2 相对寻址 ‣ 我们的方法 ‣ 基于Minecraft建造者对话代理任务的LLM基准测试")所示）。另外，始终有三个不同颜色的方块作为干扰项。我们将此测试与删除操作进行重复，而不是添加操作。

![参见标题](img/41d8444372e1a29ce94d7631040dc7ba.png)

图1：相对定位任务，将一个绿色方块放置在一个现有的蓝色方块上方

### 2.3 原始形状

当给出构建包含多个方块的结构的命令时，这些结构通常是原始形状，例如一排方块或塔楼，例如 [A.6](https://arxiv.org/html/2407.12734v1#A1.SS6 "A.6 B1-A3-C3-1522431780184 ‣ 附录A 附录 ‣ 基于Minecraft构建者对话代理任务的LLM基准")。我们测试了四种不同的原始形状：一排、一座塔/堆、一块立方体和一个矩形。

## 3 结果

表 [1](https://arxiv.org/html/2407.12734v1#S3.T1 "表 1 ‣ 3 结果 ‣ 基于Minecraft构建者对话代理任务的LLM基准测试")展示了不同方法之间的评分范围，代表了应用不同提示技术时可能获得的结果。

|  | 零样本 | 思维链 |
| --- | --- | --- |
| 绝对寻址 | 42.98 | 76.5 |
| 相对寻址 | 82.02 | 95.8 |
| 原始形状 | 59.02 | 60.3 |

表 1：结果

我们认为该方法可能有助于发现代理的弱点，并为解决这些问题提供方法。例如，识别出的一个主要问题是，在没有使用思维链方法的情况下，LLM（大语言模型）往往忽略计算某一轴的值。此外，尽管已指示LLM应用右手坐标系（3D坐标系），其中$Z$轴为南方，南方却经常与负值（左手坐标系）关联。通过几个示例强化这一概念，可以避免这个问题。

## 4 结论

在本研究中，我们提出了一个基于Minecraft类任务的新LLM基准。通过应用一些基本策略来验证该基准的有效性，以观察它如何挑战当前的LLM。

## 致谢

本研究由ARCIDUCA资助，EPSRC EP/W001632/1

## 参考文献

+   Bang 等（2023）Yejin Bang, Samuel Cahyawijaya, Nayeon Lee, Wenliang Dai, Dan Su, Bryan Wilie, Holy Lovenia, Ziwei Ji, Tiezheng Yu, Willy Chung 等。2023. 《ChatGPT在推理、幻觉和互动方面的多任务、多语言、多模态评估》。*arXiv预印本 arXiv:2302.04023*。

+   Callison-Burch 等（2022）Chris Callison-Burch, Gaurav Singh Tomar, Lara J Martin, Daphne Ippolito, Suma Bailis 和 David Reitter. 2022. 《地下城与龙：人工智能对话挑战》。*arXiv预印本 arXiv:2210.07109*。

+   Gray 等（2019）Jonathan Gray, Kavya Srinet, Yacine Jernite, Haonan Yu, Zhuoyuan Chen, Demi Guo, Siddharth Goyal, C Lawrence Zitnick 和 Arthur Szlam. 2019. 《Craftassist：一个对话驱动的交互代理框架》。*arXiv预印本 arXiv:1907.08584*。

+   Jayannavar 等（2020）Prashant Jayannavar, Anjali Narayan-Chen 和 Julia Hockenmaier. 2020. 《在Minecraft对话中学习执行指令》。发表于*第58届计算语言学协会年会论文集*，第2589-2602页。

+   Johnson 等（2016）Matthew Johnson, Katja Hofmann, Tim Hutton 和 David Bignell. 2016. 《Malmo平台：人工智能实验平台》。发表于*Ijcai*，第4246-4247页。

+   Kiseleva 等人（2022）Julia Kiseleva、Ziming Li、Mohammad Aliannejadi、Shrestha Mohanty、Maartje ter Hoeve、Mikhail Burtsev、Alexey Skrynnik、Artem Zholus、Aleksandr Panov、Kavya Srinet 等人。2022年。在协作环境中的互动基础语言理解：Iglu 2021。发表于 *NeurIPS 2021 竞赛与展示专辑*，页码146–161。PMLR。

+   Köhn 等人（2020）Arne Köhn、Julia Wichlacz、Christine Schäfer、Alvaro Torralba、Jörg Hoffmann 和 Alexander Koller。2020年。Mc-saar-instruct：一个为 Minecraft 指令生成代理提供的平台。发表于 *第21届话语与对话特别兴趣小组年会论文集*，页码53–56。

+   Kranti 等人（2024）Chalamalasetti Kranti、Sherzod Hakimov 和 David Schlangen。2024年。[检索增强代码生成用于情境动作生成：以 Minecraft 为例](http://arxiv.org/abs/2406.17553)。

+   Madge 和 Poesio（2024）Chris Madge 和 Massimo Poesio。2024年。[大型语言模型作为 Minecraft 代理](http://arxiv.org/abs/2402.08392)。

+   Mirzaee 和 Kordjamshidi（2022）Roshanak Mirzaee 和 Parisa Kordjamshidi。2022年。利用合成语料进行迁移学习，用于空间角色标注和推理。*arXiv 预印本 arXiv:2210.16952*。

+   Mohanty 等人（2022）Shrestha Mohanty、Negar Arabzadeh、Milagro Teruel、Yuxuan Sun、Artem Zholus、Alexey Skrynnik、Mikhail Burtsev、Kavya Srinet、Aleksandr Panov、Arthur Szlam 等人。2022年。收集互动多模态数据集以支持基础语言理解。*arXiv 预印本 arXiv:2211.06552*。

+   Narayan-Chen 等人（2019）Anjali Narayan-Chen、Prashant Jayannavar 和 Julia Hockenmaier。2019年。在 Minecraft 中的协作对话。发表于 *第57届计算语言学会年会论文集*，页码5405–5415。

+   Ogawa 等人（2020）Haruna Ogawa、Hitoshi Nishikawa、Takenobu Tokunaga 和 Hikaru Yokono。2020年。用于收集任务导向对话数据的游戏化平台。发表于 *第十二届语言资源与评估会议论文集*，页码7084–7093。

+   Shi 等人（2022）郑向 Shi、强 张和 Aldo Lipani。2022年。Stepgame：一个新的基准测试，用于文本中的鲁棒多跳空间推理。发表于 *人工智能大会论文集*，第36卷，页码11321–11329。

+   Srivastava 等人（2022）Aarohi Srivastava、Abhinav Rastogi、Abhishek Rao、Abu Awal Md Shoeb、Abubakar Abid、Adam Fisch、Adam R Brown、Adam Santoro、Aditya Gupta、Adrià Garriga-Alonso 等人。2022年。超越模仿游戏：量化并推断语言模型的能力。*arXiv 预印本 arXiv:2206.04615*。

+   Szlam 等人（2019）Arthur Szlam、Jonathan Gray、Kavya Srinet、Yacine Jernite、Armand Joulin、Gabriel Synnaeve、Douwe Kiela、Haonan Yu、Zhuoyuan Chen、Siddharth Goyal 等人。2019年。为什么要在 Minecraft 中构建助手？ *arXiv 预印本 arXiv:1907.09273*。

+   Urbanek 等（2019）Jack Urbanek, Angela Fan, Siddharth Karamcheti, Emily Dinan, Saachi Jain, Samuel Humeau, Tim Rocktäschel, Douwe Kiela, Arthur Szlam, 和 Jason Weston. 2019. 在幻想文本冒险游戏中学习说话和行动。

+   Wei 等（2022）Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou 等. 2022. 连锁思维提示引发大语言模型的推理。*神经信息处理系统进展*，35:24824–24837。

+   Weston 等（2015）Jason Weston, Antoine Bordes, Sumit Chopra, Alexander M Rush, Bart Van Merriënboer, Armand Joulin, 和 Tomas Mikolov. 2015. 朝着 AI 完全问题回答迈进：一组必要的玩具任务。*arXiv 预印本 arXiv:1502.05698*。

+   Winograd（1972）Terry Winograd. 1972. 理解自然语言。*认知心理学*，3(1):1–191。

+   Wu 等（2023）Yue Wu, Xuan Tang, Tom M Mitchell, 和 Yuanzhi Li. 2023. Smartplay: 大型语言模型作为智能体的基准。*arXiv 预印本 arXiv:2310.01557*。

## 附录 A 附录

### A.1 B1-A3-C8-1522432497234

> 建造者
> 
> 这是一个表格吗？
> 
> 建筑师
> 
> 我不知道那是什么

### A.2 B1-A3-C4-1522432009099

> 建造者
> 
> 这是一朵花！
> 
> 建筑师
> 
> 是的，你非常敏锐，观察力极强的建造者

### A.3 B1-A3-C1-1522435497386

> 建筑师
> 
> 现在我们必须创建铃铛。请从中间的紫色块向下延伸 4 个橙色块，就像它正在悬挂一样

### A.4 B3-A2-C12-1522445699382

> 建筑师
> 
> 在左上角放置一个紫色块

### A.5 B3-A2-C23-1522447244858

> 建筑师
> 
> 在你添加的每个红色块下方再加一个绿色块

### A.6 B1-A3-C3-1522431780184

建筑师

建造一个蓝色的 2x1 结构
