# P15：15.机器人与模仿学习 - life_code - BV1X84y1Q7wV

![](img/dde5df4844de2a3e55cc92f364263e12_0.png)

![](img/dde5df4844de2a3e55cc92f364263e12_1.png)

嗨，大家，谢谢你们的耐心等待，我很高兴能在这里。我来简单自我介绍一下，我叫 Ted Shao，是谷歌 Bra 团队的高级研究工程师。我在机器人领域工作了五年，涉及了一些主题，包括多任务学习、强化学习，最近则在广泛思考如何扩展机器人，以确保它们能够在现实世界中实际工作。

😊，我想今天我会谈论一些不同的话题。首先需要说明的是，我们的团队现在规模相当庞大，这些项目都是大型合作，有些项目已经有超过 40 人参与多年，所以这些都是大规模的努力。我很幸运能与一群非常聪明的人一起工作。其次，我的一些看法可能比其他观点更激烈或更具争议，因此所有这些意见绝对只是我个人的，不代表谷歌或团队中的其他人。

话不多说，欢迎来到我的 TEDx 演讲。![](img/dde5df4844de2a3e55cc92f364263e12_3.png)

嗯。所以我想，也许你们中的一些人已经看过很多关于酷炫机器人学习的视频。这些天真是太棒了。但是我比以往更加兴奋，这不仅仅是炒作，我认为在过去的两年中，研究人员和机器人领域对学习的看法发生了根本性的变化，而这种变化与更广泛的趋势有很大关系，包括大型互联网模型在不同领域（如语言、音频等）中的基础建模。我今天的目标是向你们传达，为什么我特别兴奋于现在这个时刻，以及为什么在机器人学习领域发生了一个非常根本的 18 度范式转变。如果你在这次演讲中只记住一件事，那就是你对机器人科技的兴奋程度稍微提高了一些，或者相信现在是这些机器人真正开始指数级扩展并做一些很酷的事情的时刻，那么我的演讲就成功了。

嗯。银河。演讲将分为几个部分，我们将从一个非常高的层面开始，谈谈为什么要为机器人建立基础模型。它可能是什么样子，以及我们可能如何达到的成分和配方，然后我们将深入探讨我团队在过去一年或两年中非常自豪的一些工作，最后我们将回到高层，思考机器人学习的下一步。

那么，为什么机器人需要基础模型呢？等一下，让我试着隐藏这个东西。不，这没关系，我暂时会保留那个条，但顶部的条说“为什么机器人需要基础模型”，你知道这个概念是在斯坦福提出的，我将用“互联网规模模型”、“基础模型”和“大型语言模型”来互换使用，希望这能清楚表达，但通常我谈论的是这些在大量数据上训练的大型单体模型。

它们有两个我认为相当不错的重要特性，一个是“涌现”，当非常简单的事物在小规模上工作时，当你只是扩大规模，增加数据、计算和更大的模型时，它们的表现会好得多，而我们在这里看到的是，当这些模型变得足够优秀时，它们擅长和能够做的领域空间开始变得组合更大。对此，我想推荐两篇博客文章，我强烈建议阅读，一篇是雅各布·斯坦哈特的《更多就是不同：人工智能》，这篇文章链接了这个现象。

在物理或生物等其他领域中，例如，单个水分子的行为会非常不同，它们的电静力学作用也会有很大差异，然后它们开始聚集并共同表现出液体的特性，我们在动物群体和聚集模式中看到了这一点，我们在经济学中的人类行为中也看到了这一点，这种现象在不同领域普遍存在，如今甚至在人工智能中，我们看到模型在做一些在更小规模下甚至不可能的事情。

但当它们达到某个临界规模时，它们开始表现得非常好。这在杰森的博客文章《LOMs 中的涌现》中有记录，你可以在左下角看到这个图，成功率跨越了许多不同任务，无论是基本算术还是抽取式问答，成功率基本是平稳的，直到这些模型变得足够大、足够好，然后成功率就开始飙升，这就是我认为这些模型特别令人兴奋的原因，我很好奇你对此的看法。

身体基础模型现在展示了扩展性。好问题，我很高兴你问这个。我们有一些方向，我很兴奋能在大约 10 分钟内解答这个问题。但我认为这是我们所有人，包括我自己都在思考的问题。所以，我认为在我们甚至讨论任何机器人基础模型的可行性或存在性之前，我们需要问这是否真的必要，我认为一个并不显而易见的论点是，依赖于这些新兴能力可能对机器人实际工作是不可或缺的，过去几十年的机器人研究大多是在一个箱子、一个房间、一个桌子里进行的。

一个机器人一个建筑，然而这些与人类每天所处的真实世界情况相比差异巨大。我认为，要实现这个巨大的飞跃，我们将不得不依赖这种新兴能力的规模曲线，其中某些东西可以正常运作，也许你有一个人形机器人经过数百次试验成功地编程了后空翻，但从这个到混乱的真实世界，我认为我们将不得不依靠这种涌现现象。

![](img/dde5df4844de2a3e55cc92f364263e12_5.png)

我认为，甚至从智力或学术的角度思考为什么或为什么不一个机器人基础模型可能会有效是很有趣的，它在许多其他领域都取得了成功。在音频、音乐、编码语言等领域，每天似乎都有存在的证明，甚至在 3D 模型及其他方面。但如果机器人有一些特别之处，无论是具身性、因果关系还是物理基础，这成为了使这个在其他领域都有效的简单配方无法适用的障碍。如果机器人有特殊之处导致这个配方失效，我认为研究原因是非常有趣的。就我个人而言，我是个乐观主义者，我不认为会有某种神秘的“秘密酱”阻止机器人使用在其他地方有效的相同公式和配方。

不过，我认为这是一个我想要找到答案的概念。因此，也许与其从哲学上激励我们需要基础模型，基础模型是伟大的，我们如何为机器人建立它们，实际上该怎么做？我认为我们可以通过借鉴其他领域的巨人的肩膀，利用一些成分。首先是关注其他领域的机器学习扩展的不同设计原则，让我们先看看高容量架构。

今天这堂课的话题是自注意力等概念，以及变换器中包含的所有不同思想，正如安德烈·卡帕斯基著名所说的。这就像一个神奇的通用可微计算机，非常通用、非常稳健，在许多不同维度上极具可扩展性。我们应当利用这些，同时也要利用可扩展性法则的更多指导原则。这里的趋势是，我们不仅需要扩大模型大小，还需要扩展计算能力，也要扩大我们训练的大型数据集中唯一标记的数量。

![](img/dde5df4844de2a3e55cc92f364263e12_7.png)

但如果我们将三者结合，这已被证明在任何领域都有相当不错的成功机会。因此，最终这意味着，我认为这将在后面提到的数据集大小似乎比质量更重要。

即使维基百科上有一些拼写错误的句子，或者一些你知道的不实信息，或者一些不太理想的内容，只要你的数据集在总体上足够多样化和有趣，这些问题希望会在混合中被洗净。这是第二个成分。

互联网规模模型本身的激增，不仅仅是原则。令人兴奋的是，这无疑对专家和普通人都造成了震惊，因为许多不同模态的生成模型正在经历新兴能力，并一次又一次地超越我们所有的期望。即使我们认为已经耗尽，认为这些内容太多、不可能工作时，总会出现一些东西彻底颠覆我的认知。我认为这种趋势肯定会持续下去，而且不仅会继续加速，它们的发生将不受我们是否采取行动的影响。作为一个机器人研究人员，或者你在任何其他子领域中，机器学习的某些部分可能在短期内你根本不会接触到，而这些部分将每周都在见证巨大的突破、扩展和新的能力上线。

你可以不仅从模型的令人印象深刻程度来看待这一点，还可以从进展的加速、发布新模型的时间尺度来看，许多团体在大规模合作下共同努力，然后这些模型又可以被所有人访问和使用，进行构建。

最终的成分是一个更具机器人特定趋势的内容。这是一个巨大的转变，从在线机器人学习中，机器人在线收集经验，进行操作，通过试错学习，转变为离线环境，在这里我们将数据生成过程与数据消费过程分开，正如我们在其他基础建模领域所见的那样。

这些大型互联网规模数据集是如此多样化而静态，我们只需抓取一次或多次连续抓取，但我们聚合出一个不断增长的持续数据堆。在这里，我们看到来自 Luther 或 Lyon 5B 的数据集，这些图像配对文本的数据集相当庞大，数量级上远超我们以前见过的，这无疑是其他领域能够如此成功训练这些大型基础模型的关键成分。

然后，这回到机器人技术上，我想简要讲一下转变是如何发生的，因为说“机器人技术更多的是离线而非在线”这一句很容易，对许多来自其他领域的人来说，这似乎是显而易见的方式，但在机器人技术中，这是一个非常大的转变，我认为对于很多人来说，机器人技术与强化学习是同义的，但我认为这种情况正变得越来越不成立，因此我想带你们回顾一下我团队的简史，关于 Google 机器人技术的简要历史，当然，谢谢，我认为这不仅仅是为了戏剧性表述，而是真正想指导你们了解我们团队的思维是如何在这些年中发生了剧烈变化，这将如何影响我们在具体项目中所做的设计决策，以及我们所承担的风险和研究方向。谢谢。

所以在 2016 年，你们中的一些人可能见过这个，我们有一个我们称之为臂农场的项目，7 个 Cua 机器人在一个房间里进行 247 的采集和拾取数据，这是在现实世界中进行的在线策略学习，我们是第一个团队说“嘿，我们能不能这样做，目标是能否进行端到端的机器人学习，并在现实世界中取得成果”，这在当时是有一定风险的，并不是一个常见的观点，从中我们发展出了几条有趣的研究方向，我们开始探索，比如 QTO，这是一种在处理连续控制动作时使用视觉输入的 Q 学习方法，我们还使用 CycleGAN 将基于仿真的图像转换为真实感图像，以实现从仿真到现实的转变，我们研究了如何让机器人在现实世界中更快更高效地工作。对不起，你有问题吗？是的，很好的问题，我认为基本上是这样的。

如果臂从箱子里拿东西时出现错误掉了出来，我们第二天早上会回来，发现房间里散落着物体。所以没有重置，但如果它们稍微错过，物体会掉回箱子，希望能够重新处于可以再捡起的位置。

😡，哦，当然，聊天谢谢你，我会在未来这样做，这个具体问题是关于这个 247 臂农场的，我们是如何进行重置的，答案是我们没有，我们设计了一个箱子，使得物体稍微错过时会掉回箱子中，自我再生，也许会通过训练数据增加更多多样性，但这是在使用 Q 学习进行离策略在线强化学习，同时我们又将其与 SIim 数据重新部署。

接下来，我们大约在 2020 年经历了一个整合阶段，觉得“这真不错”，但我们想要脱离简单的框架，如何在更实际的环境中完成更复杂的任务，以更接近人类日常使用的方式。在这里，我们最终确定了一个办公室微型厨房的环境，如果你听说过著名的谷歌微型厨房，我想这是我们决定操作的环境，我们开始收集数据，扩大我们的实际操作，并探索一些不同的方法。在右下角，可以说这是机械化的重置版本，我们有一个可以折叠的箱子，它可以进行现实世界中的多任务强化学习，箱子会在两侧翻转，从一个侧面倾倒物体到另一个侧面，这样我们可以进行更有趣的任务，而之前的手臂农场只能任意拾取，现在我们可以说，“嘿，拿起胡萝卜，把番茄放到盘子上”，然后箱子会翻转并重置。

在多任务模仿学习方面，还有其他一些工作，这就是 BC0。我们还研究了将强化学习与模仿学习相结合的内容。但在 2020 年，我们再次意识到我们在许多不同的方向上工作，想要整合。我认为当时真正困扰我们的两大问题是，我们在所有这些方法中遇到了两个主要的瓶颈，一些方法在现实世界中的表现停滞在 50%到 70%左右，而其他方法则需要非常特定的数据分布，它们必须在政策上，或者只能使用演示，等等。

blah，blah，这些不同的方法都有许多不同的细微差别和问题，我们提出的问题是我们对任何方法开放，任何能够让我们在现实世界中以非常高效的方式解决任务的策略，准确率超过 90%，并且能够根据我们收集的一些数据进行扩展。你知道，这可能比在资源更加有限的学术环境中要宽松一些，但归根结底，我们团队也并没有无限的资金，我们仍然有一定数量的资源。

某些操作员受限于物理法则，因此我们需要某种方式来获取更多数据，以便我们可以从中学习。因此，在 2022 年春季，我们都在思考这个问题几个月。我们决定采用多任务模仿学习，这与 247 臂农场大相径庭，这是我们解决问题方法的巨大演变。我们发现，只要有足够的温柔关爱，多任务模仿学习能够达到 90%的表现，并且随着更多演示而变得更好，这些演示并不是最便宜的，但它能够随着额外演示而扩展，这正是我们寻找的生命迹象。所以这让我们回到了不到一年前，我们团队决定这是近期未来的前进方向。

也许我们可以考虑我们在这里采取的方法未来如何扩展，我们可能能够重新引入这些其他线程。例如，现在我们将演示数据的收集与如何从中学习通过多任务模仿学习政策解耦。

也许未来我们会做一些像离线强化学习的事情。但我认为在高层次上，我在短短几分钟内压缩了我们团队六年非常痛苦的教训，从今天的角度回顾，甚至仅仅两年前的情况。

如果你告诉我今天的策略能够以这种方式扩展，我可能不会相信你。😡，啊，有几。但无论他们做什么，而不是试图弄清楚怎么做。这是个很好的问题，我认为当时任务条件化确实还是一个开放的问题。但我认为通过这项工作 BC0，我们发现语言，至少在模板化语言的表示中，足够好，我们可以指导，我认为 BC0 涉及超过 80 个任务，它们非常模板化，比如采摘葡萄，或将葡萄移到盘子上，或者把布拖过桌子。我认为这种表示仍然足够，让你学习到相当数量的技能，你本质上是将一个独热编码 ID 输入到你的策略网络中，它将学习模仿，而对于这 80 个任务中的每一个，能够收集到数百或数千个演示。

我稍后会详细谈谈这些具体内容。嗯。所以是的，今天，或者至少在 2022 年。让我们做离线方法，让我们将数据生成与数据消费解耦。让我们利用这三条我们提到的教训，结合机器学习扩展的设计原则，找出未来在机器人学习和基础模型的配方中可以应用的教训。

我认为第一个教训非常重要的是这些高容量架构，比如注意力机制，第二个我稍后会提到的是数据互操作性、标记化、离散化，而第二个要素是这些模型本身的普及，我们能否利用它们，因为它们会随着时间的推移变得更好，我想在这里提到我的同事卡罗尔·赫尔斯曼的苦涩教训 2。

0 这是来自理查德·萨顿的第一个苦涩教训，你应该利用能够随着更多计算而扩展的方法，也许在当今时代，教训是我们应该利用能够利用基础模型改进的方法，因为它们会变得更好，是的，这就是教训 1。

0 和。0 一件让我比较的事情是拥有一套方法。我想选择那些能随着更多计算机扩展，或者在这种情况下，随着更好的基础模型扩展的方法，问题是我到底如何决定哪些方法符合这些标准？

是的，问题很好，我认为，也许这是一个很好的问题，我对此没有好的答案，哦，抱歉，问题是关于苦涩教训 1。0 和苦涩教训 2。0，问题是，好的，这就是教训，但我们到底如何决定哪些方法符合这些标准，我认为我的答案是，这并不总是显而易见，实际上有时相当棘手，但也许有时你知道你可以非常自信地说。

哦，是的，这肯定会随着更多的数据和计算而扩展，某些方面是这样的，但基本上你越是硬编码，假设就越多，启发式就越多，你就越依赖于特定基础模型的特定实现和特定算法类，也许这将比一种假设某种非常抽象输入和输出，并假设如何从输入和输出改进的方式的方法更不可靠，也许算法本身甚至会彻底改变，所以我认为这就是我对苦涩教训 2 的看法。

0 但这仍然是，我认为陪审团尚未对此做出决定。😊，对。我基本上想提出的一个观点是，语言是我们利用苦涩教训 2。0 的方法，如果你把语言作为所有这些基础相互交流的通用表示，无论是你知道的图文描述、生成或其他。

我认为这是一种利用苦涩教训 2。0 的方法。最后，第三个要素是离线机器人学习，将数据生成与数据消费解耦。将这些结合在一起，我对现代体现智能的尝试的配方是将这些大型离线数据集与高容量架构结合，通过使用语言作为通用的媒介，在我即将呈现的不同项目中，我认为在某种程度上都受到了这一哲学的启发。

哦。现在我们似乎理解了动机和可能的一种方法。抱歉，底层常常是使用语言的架构，我好奇这些中是否有任何当前的。Waternck 这个词不太合适，原始的内容是有限的。明白了，因为看起来我们已经有了架构，似乎我们已经有很多这些成分，那么为什么机器人技术还没有解决呢？我认为实际上这里的观点，也许现在对于这个听众来说不太合适。

但我认为这在机器人领域并不明显，许多人并不同意所有这些，甚至对于其中的两个或任何这些观点也持怀疑态度。并且我认为，机器人技术中每个组件的成熟度的规模存在很大差异，我们可以稍后讨论，例如数据规模或已经通过其他机器学习领域渗透到机器人中的架构，但我认为我们在各自如何接受这些教训并对其进行投资方面仍处于非常不同的阶段。

你没有参与这个，抱歉，我很好奇想问你一些名字。那些可能不会深入挖掘这些内容的人。1。是的，我可能也不想在这里惹上太多麻烦，我可能会在几张幻灯片中陷入一些麻烦，所以我会进一步扩展这个问题。是的，我个人而言，当然不代表我的团队，但我团队中的很多人可能在学习、扩展、数据驱动的基础模型方面处于极端的边缘，让我们大胆尝试，我认为很多人并不相信这一点，之后乐意讨论原因。

也许在 Zoom 之后也是如此，所以好的，那我们就开始深入探讨，看看这个食谱如何真正渗透到特定领域。😊 第一个是 RT1，这是我们小组最近的工作，研究如何扩展模仿学习，看看我们如何可以实际应用这些基本原则。

所以第一个要考虑的是我们实际上要做什么，让我们把自己放在 2020 年春季的思维模式中，我们已经收集了演示一段时间。这是大量的演示，大约 100000 个，花了一年半的时间在许多机器人上执行许多任务。这非常昂贵，随着时间的推移，这不会以疯狂的速度增加，我们不会每天获得 100000 个高质量的新演示，这将随着时间的推移增长，但不会免费增长，采用自主方式做这件事非常困难，正如你之前看到的 MT 使用的 bin 重置机制，或者 DeepM 有一项名为 RGB 堆叠的工作，他们试图进行自主重置，而我们现在的做法，或者至少在这篇论文中，是由 BC0 开创的人工远程操控，这同样非常昂贵，因此这将受到限制，最后 BC0 使用了基于 ResNet 的主干网络，效果相当不错，但发现它对训练分布非常敏感，例如，当他们从一些遥控操作中移除数据以使数据更加均匀时，性能反而有所改善。

性能变得更好，但这并不是我们喜欢的特性，我们希望更多的数据，即使它们不完全相同。因此，这里的教训是模型需要强健，并且需要泛化。好吧，我们知道模型需要强健和泛化，还有什么呢？现成的模型相当慢，如果我们采用这些巨大的视觉变压器，在其他领域中，它们无法在真实机器人上运行，我们需要能够以相当高的频率运行，它们需要具有反应性，推理时间需要短，因为我们所有的模型都是基于视觉的。

最后，我们希望我们的数据能够理解语言，正如我提到的。如果语言是普遍的粘合剂，我们的数据集已经包含了一些语言，我们希望最终的模型能够非常多模态。这是我们需要考虑的首要原则。这意味着我们不能仅仅采用现有的东西。

我们可能需要从头开始设计或至少修改一些东西，让我们借鉴在其他领域中看到的最佳实践。因此，我们工作了一段时间，并再次提出了这个 RT1 的架构。这是一个大型团队，参与了许多不同的贡献，我将在这里简要介绍其中一些。从高层次来看，RT1 是一个机器人变压器，它以三赫兹的频率运行，接收来自机器人 RGB 相机的视觉输入以及自然语言指令，图像被切片并输入到一个高效的网络分词器中，接着传递给即将讨论的 token learn，同时语言指令也被分词，并被放入同一个变压器中，最后我们输出离散化的动作作为 tokens，并以三赫兹的频率将其发送到现实世界中，形成闭环。

这个变换器是一个解码器，我们使用稀疏分类熵目标进行动作预测，通过应用因果掩码。我们使用预训练的高效网络骨干，并且还使用标记学习器以加快推理。深入一点，哦，对不起，有个问题。对的。非常好的问题。当图像标记从每个图像中进入时，你知道来自相机的高保真 RGB 图像，我们将其分割成 81 个独立补丁，所以每个补丁在空间上就像一个正方形，但有趣的是，标记学习器在这里做的是，这是我们组的一个先前工作，它接收一组可能的图像补丁，并动态选择哪些图像补丁标记在给定的上下文中更相关，因此从这 81 个图像补丁标记中，我们抽样出八个用于推理，这个过程在每次都发生。

这个过程已经学习到在任何给定时刻哪八个补丁是相关的，否则我们发送的标记太多，上下文长度会爆炸，我们将无法在机器人上进行推理，我们还传入六个图像的序列长度，历史在进行时间一致的现实世界任务时非常重要，因为物理等因素以及你知道对象之间的细微细节对你的机器人来说非常重要，这些细节确实很重要。

总的来说，模型的大小是 3500 万个参数，这比许多其他大型互联网规模模型要小得多。最后一个主要区别是动作离散化，在我们进行的许多产品中，我们是在进行连续控制。如果你仔细想想，我们的机器人确实有末端执行器姿态控制、位置控制，现实世界是一个连续状态空间，但为了做到这一点，我们必须提出许多算法新颖性。

一个 CM 演员基本上对这些连续动作空间进行采样，以提出那些将由 Q 函数评估的最高动作，我们这样飞来飞去。但是这非常敏感，但我们需要这样做才能让事情运转。

但是现在我们决定让我们的行为知道，只需 256 个离散动作，并将其预测为标记。有什么问题吗？我提到的是。你有关于速度和延迟反应的设计或工程要求。

我是说，当你说这需要相对较小的模型时，这是合理的。但是当我们谈论基础模型的扩展时，一个重要的信息是，它可能会被数据、计算或参数的瓶颈限制，所以我想知道的是，你如何平衡这些，因为你希望拥有大量参数以构建一个强大的模型，或者另一方面你希望输入非常时尚。

是的，问题很棒，简单重复一下，问题是我们设置了一个 100 毫秒推理时间的严格限制，但基础建模中的许多经验教训是你不应该在任何维度上限制自己，无论是状态集大小、计算能力还是模型容量。我认为我最初的回答是这是一个非常好的观点，我认为这将在未来成为一个严重的瓶颈，但就我们的初步案例而言，我认为这更多是探索这些原则，甚至超出我们目前所考虑的规模是否可以有效，这 3500 万与许多之前的工作相比是巨大的，例如使用 Resnet 34 等，因此这已经比许多其他选项大得多，或许至少在短期内这是我们能够达到的最大规模，而不必考虑更多技巧。

是的，我们可以稍后再谈这个，或许我也想听听你的想法，因为如何突破这些瓶颈并不明显。是的，问题很棒，我们对模型大小进行了一些消融实验，我可能会在几张幻灯片中提到。

不过也许我们可以稍后再谈这个，如果没有，那也是个很好的问题。所以是的，这就是架构，我稍后会讨论一些趋势的消融实验。但也许你知道这是一个机器人讲座，我应该给你展示一些很漂亮的视觉效果，所以让我们看看我们做的一些评估，我们与一些基线进行比较，其中一个是你可能熟悉的 gotto，还有其他基于 reite 的 species0。我们发现我们在可见任务与不可见任务上进行评估，并且在各种物体上我们的正常数据收集看起来是这样的：左上角的图片是三罐在一个灰色的桌子上，但我们进一步推进，引入了更多物体，使得桌面混乱到即使是人类有时也很难找到你实际上在寻找的物体。我们增加了桌子类别，以使纹理非常不同，我们引入了新的微型厨房环境和这些表面在一起，我们发现 Rt1 比其他不同的方法更稳健。

![](img/dde5df4844de2a3e55cc92f364263e12_9.png)

这个数据的好问题是，gotto 模型是用我们的数据训练的，还是已经包含在 gotto 中？答案是这个数据没有包含在 gotto 中，所以我们仅用我们的数据重新训练了 gotto 模型。是的，这里是机器人在我们的微型厨房中执行各种有趣任务的不同可视化，你可以看到它是在一个设置上训练的。

但随后它进入了全新的厨房，新的台面，新物体，并且能够相当稳健地处理所有这些。我们还把它放入了一个长期的设置中，使用我们稍后会谈到的 Saan 框架。在这些设置中，许多都是混合了所有这些泛化能力，左侧的图中我们使用了受 VeEMA 论文启发的所谓泛化水平，基本上是同时越来越多地改变更多的变异因素。在这里我们发现 RT1 是最稳健的。

就是这样，你可以同样地去看前面的内容。是的，好问题，稍后我们会更详细地讨论，但我认为从高层次来看，远程操作员会获得一个结构化的模板命令，例如动词名词动词，或者像“捡起可乐罐”或“将苹果移近海绵”，我们大约设置了 700 个这样的任务，他们会继续收集数据，完成测试，然后我们确保成功的确是成功的，并且我们会丢弃那些不安全的内容，例如。

哦，是的，明白了，对于这篇论文，我们利用了 130,000 个演示。它们都被写下来了。啊这个。有什么问题吗？是的，好的问题，我认为许多之前的工作也注意到，当你有例如，问题是你发现数据中的轨迹非常多模态吗？我认为你的意思是，从 A 点到 B 点我可以向左、向右或直行，而我认为这种多样性基本上对于单一图像状态来说，但我的数据有三个可能的标签，这在某些时候会产生非常糟糕的影响。我们认为，因为我们使用远程操作员的演示，数据比例如“玩数据”更为同质，后者让操作员随意操作，我们事后进行标记。我认为我们的数据在这方面更为同质，但我们没有发现很多在之前项目中遇到的问题，一个潜在的答案可能是架构本身，但我们稍后也可以讨论这个问题。

回到叉市。对。我好的。好问题，我们实际上有一个终止动作，所以策略本身，问题是如何判断一个回合是否完成，策略能够预测终止，因为在 H teley 操作会话结束时，操作员可以点击一个按钮，标记为该回合已完成。

我想。是的，我认为在这些评估中我们非常严格，但我确实认为在某些情况下，你知道，也许如果我们只是为自己做实验，我们会有一个密集的奖励尺度，比如抓取物体并移动更近，抓取物体并几乎成功，如果我在最后搞砸了，我们就会有一个评分曲线。

但对于我在这里展示的所有这些统计数据，结果是零或一，一个完全完成，零。并没有完全完成。![](img/dde5df4844de2a3e55cc92f364263e12_11.png)

哦。我认为让人期待的或许是关于多模态方面的讨论，然后我们进一步推动极限，我们决定在非常多样化的数据分布上进行训练。是的，你现在看到的是基于这个日常机器人专有移动操控器的 1301000 个演示。

但我们也在寻找在非常不同的数据分布上进行训练的机会，包括非常不同的动作分布、非常不同的轨迹，甚至非常不同的视觉对象任务。为此，我们包含了两个其他数据源，一个是模拟数据，这就像是我们的机器人在 SIim 中，但看起来非常不同，这些数据也是通过强化学习收集的，而不是通过过去提到的所有 IL 和 RL 工作中的遥控演示。我们发现结合这两种数据类型将会非常困难，因为 RL 数据的动作非常短促、迅速，并且针对特定的奖励函数进行了优化。

人工收集的遥控数据更加“人性化”，可以这么说，最后我们恢复了几年前的 2018 年的数据，如果你还记得 Ka 项目。那时，Ar farm 在那种状态下已经多年未运行。但我们仍然拥有这些数据，因此我们希望看看是否可以将一个不同动作空间的不同机器人与不同视觉的物体和不同建筑的数据与我们最初训练的宏观厨房数据集结合起来。令我感到非常惊讶的是，RT1 能够从所有这些非常多样化的数据分布中学习，我从未见过这样的结果，任何其他架构也如此。

一个 ResNet 或甚至像强化学习这样的学习方法可以在如此不同的数据分布上成功学习得如此稳健，我们评估了例如概念的结合，因此我们会让原始的日常机器人拾取在 Ka 项目中仅见的物体，或者我们会放置在模拟中仅见的物体。

并观察我们的策略是否能理解，所以看起来它确实能够在之前的数据集看到的物体之间进行泛化，以及在现在的真实微型环境中看到的概念，这是日常机器人的一个非常重要的结果。

很好的问题，是的，我们进行了标记化，并确保标记化方案是可互操作的，我想这就是要点，我稍后可以深入探讨一下。请注意，这并不意味着我们可以将一个机器人的确切动作发送给另一个机器人并让其执行，更像是在数据集中，甚至通过人工检查，你可以看出这些数据来自两个不同的机器人。

😡，所以是的，让我们来看看关于我们现在在这里讨论的缩放法则的一些消融实验。我们发现减少数据量确实会降低性能，但更有趣的是任务的多样性在这里非常重要，我们有两个不同的趋势。绿色线是减少每个任务的总剧集数量时发生的情况，紫色曲线是减少总任务数量时发生的情况。我们发现，拥有更多的任务在相对上比为每个任务拥有更多的数据更为重要。

我觉得这是一课，我想这可能会建议我们应该进一步扩大机器人技术的方式，不仅仅是收集同一任务在相同设置下的更多数据，而是要走出实验室，获取更丰富的行为。

问题是我没有定义多样性数据。很好的问题，问题是你如何定义这个案例中的数据多样性，它仅仅是遥控操作接收到的独特结构化模板命令的数量，所以当你开始减少这 700 个模板命令，仅训练 500 个或 300 个时，性能下降的速度比我们对总体方法进行相同比例的削减时要快得多，所以我想我很好奇。

没有头绪。它看起来似乎有些混乱，真的很难抓住要点。是的，我觉得问题在于似乎数据大小和成功率之间几乎存在线性相关性，我想你知道我们可以应用一些复杂的，比如说缩放法则，尝试曲线拟合，但我们没有深入研究这一点，因为这是我们预期的趋势，我们只是不确定它会对我们造成多大的影响，我认为。

除了我们在经验上看到了这种现象，我没有什么特别好的见解。嗯，问题是，也许这会无限继续下去，2023 年 1 月有什么神奇之处。我觉得也许这就是我们开始将算法探索与现实操作的缩放实际考虑混淆的时候，正是在这时我们获得了数据，我们的策略接近 100%的饱和度，我们就想“好吧，收集另一个数据集”，所以我们基本上收集到 100%后就切换到其他任务，但有趣的是，当我们在这个 Rt1 架构上大下注时，我们已经收集了演示一段时间，因此我们有可能收集了超过所需的数据，在某些情况下，实际上可以在不损失太多性能的情况下削减任务，这非常有趣。

是的。他说如果这与不同任务轨迹数据在不同任务之间可能更加多样化有关，但在多个轨迹中，如果你只是对所有任务使用相同的作者，或者说没有给定的领域的话。

朱类喺度要发过俾猪呢啊呢啊。这个问题很好，问题在于不同任务在能力和熵方面是否相等，确实有些任务要简单得多，比如“捡起这个物体”的任务，它所能挤出有趣内容的潜力远不如“把东西放进抽屉然后关上抽屉”，所以这个问题很不错。

很好，现在，消融实验，我们也在没有大型模型的情况下进行训练。我们没有预训练，使用了一系列连续的离散动作和自回归动作，没有历史，没有变压器。我认为所有这些设计选择似乎对稳健性能是必要的。哦，当然。是的。分正家成。是的，我认为所有的意思是，对于论文写作来说，这就是我们可以经验性找到的最佳方法，然后我们会弄清楚这些方法为何重要，所以我认为这里一个令人惊讶的事情也许是自回归动作会产生负面影响，你可能会认为提供更多信息总是比提供更少信息更好，但在这种情况下，也许对你先前的动作进行条件化就像进行上下文学习一样，进行在线系统识别以确定。

这些数据来自哪里，以及你如何会过拟合到那一特定的动作历史，因此去除这些实际上更好。有趣的小插曲。没。很好，可能出于时间考虑，我会试着更快地完成其他的，然后我们也许可以在最后做一些问题，如果可能的话，这样我们就有时间处理所有内容。

这里的下一个工作稍微偏离技能学习，实际上进入了规划层面。我认为第一个项目借鉴了其他领域的设计原则以及这种离线机器人学习范式，并将其应用于技能学习，因此我们实际上将其推广到机器人系统的其他部分，第一项工作是 Sacan，如果你还记得 2022 年的这个时间点，我们开始思考如何扩展这种多任务学习，同时大型语言模型和其他类型的基础模型也在迅速发展，无论是 Iogen 还是 Do2，我们确实想找出如何利用它们。我们想出了这个 RT21 设计，我们对其寄予厚望，但从这里开始我们开始探索所有的更好的教训。

我们可以开始在我们的全栈系统中利用基础模型。这样做的一个问题是，语言模型并不完全适合机器人。例如，如果你是一台厨房里的机器人，你问语言模型“我洒了饮料，我该怎么办”，语言模型会给出一些不太相关的建议，比如让你吸尘，或者叫清洁工，或者道歉，而这些都是机器人在厨房无法帮助你的事情。

所以这有两个部分，一个问题是我们的机器人是有限的。它们在能做的事情上受到很大限制，不能做所有事情，但可以做某些事情。第二个问题是语言模型也受到限制，它们不知道机器人看到的东西，不理解自己是在微型厨房的机器人身体中，需在物理世界中完成实际任务。因此，我们需要让机器人讲语言模型的语言，让语言模型讲机器人语言。为此，我们在相同的设置中提出，比如“请把一个苹果放在桌子上”，我们对语言模型在一组受限任务上的预测进行评分，这些任务是我们知道机器人经过训练能够完成的，同时我们也会从机器人中提取能力函数，能力函数是对给定状态下机器人能做什么的估计，以及在该状态下它成功完成该任务的信心。在我们的案例中，我们使用强化学习中的某种价值函数。

这两种评分涵盖了这个特性。我们有来自语言模型的信心值和来自机器人的信心值，我们可以将这两个值结合起来，希望组合后的预测既能对高层次指令（比如“找到一个苹果是第一步，请把苹果放在桌子上”）有很大的语义相关性，又是机器人能做到的事情，因为机器人在框架中并不知道，但它知道自己被训练过去找到苹果，因此可以导航去寻找。希望你可以这样进行闭环，然后持续从语言模型预测出高层次的计划，并与机器人理解的能力函数相结合。

![](img/dde5df4844de2a3e55cc92f364263e12_13.png)

这里有一个视频，展示了机器手在做不同的事情。但我很乐意稍后线下分享，这非常酷，相信我，这是自切面包以来最伟大的东西。😊！[](img/dde5df4844de2a3e55cc92f364263e12_15.png)

好。没有异议。是的，我们在非常长的指令测试中，这些指令包含超过 10 种单独的导航和操作技能，在右下角的微型厨房中。我们在这方面评估了数百个不同的评估，并测试了许多不同的概念，包括通过使用单一的禁止词重新表述，通过从同事和朋友那里得到的指令进行绘制。

我们发现，在语言模型规划方面确实存在失败，模型会预测当前情况的错误路径，同时在策略执行方面，即便得到了一个好的计划，机器人有时也会搞砸，不过总体上它表现得还是相当不错。现在让我们回到这个教训，我认为这是一个很好的例子，展示了我们如何利用互联网规模的基础模型随着它们的改进而变得更好。当我们开始这个项目时，我们使用了谷歌的一个语言模型 Flan，在我们实施期间，Palm 这个半语言模型上线了，当时我们能够进行热插拔，性能在没有我们做任何事情的情况下就自然而然地提升了，前提是语言就是 API，计划只需任何字符串，来源可以是人类，也可以是语言模型，当我们改进那个语言模型时。

系统总体上变得更好，你可以看到随着模型 LOM 规模的增加，我们的计划性能也变得更出色。这里有一些很酷的技巧来使其运作良好，我们是如何生成这个计划的？其实就是通过提示，正如如今的思维链那样，通过更好的提示，给出一些优秀的机器人计划的例子，现在给我一个新的计划，从这个高层指令开始。我们发现机器人能够完成所有事情，从理解不同语言到要求它们进行非常复杂的推理，比如“嘿，给我一些含咖啡因的东西”或者“我不再喝咖啡了”或“给我一些更好的东西”或者“带给我一个健康的小吃”对比“带给我一个不健康的小吃”，所以凯文能够在这些方面进行推理。

我想这可能是我们团队首次将机器人技术与语言模型接触的机会，也是对这两个领域如何重叠的首次探索，确实仍有改进空间，因此在我们的独白中，我们尝试通过引入视觉语言模型进一步改善这些。

这里的想法是，我们的计划成功率很高，但遗憾的是，它并不能真正从失败中恢复。我的意思是，语言模型并不能实时更新世界的情况，因此如果这是提出的计划，比如去桌子那边。

拿起一罐可乐带给你，但你搞砸了，把可乐掉在地上，它仍然会继续试图把它带给你，把它放在一边，这一切都不再重要，因为你掉了可乐。因此，在你的独白中，我们真的希望弄清楚如何将环境的闭环动态反馈添加到这个规划过程中。

让我们现在拿那个确切的例子，而不是直接纠正每个指令。也许我们从场景中添加一些反馈，也通过语言作为通用 API 来传达。场景可以告诉你里面实际有什么，也许机器人现在会问一个问题，这里机器人是一个语言模型在询问澄清问题，也许人类会回答，或者另一个语言模型，然后你可以预测下一步要执行的任务，一旦语言模型有了足够的上下文，也许你甚至添加成功检测等内容。

![](img/dde5df4844de2a3e55cc92f364263e12_17.png)

那我们怎么做到这一点呢？首先，我们实现的是所谓的被动场景描述，仅仅使用现成的工程启发式方法，利用物体检测模型。像 vile 这样的东西，你可以用文本描述场景，并将所有上下文传达给语言模型。

对于主动场景描述，这可能类似于视觉问答，如果你熟悉这个领域的话。语言模型实际上可以提出它对场景中的积极查询，也许是为了确保它有足够的上下文继续前进。在这里，或者一个人可以提供答案，或者未来随着 VQA 模型的改进可以提供答案。最后，关于成功检测，这对于允许语言模型规划者知道何时尝试重新开始某件事非常重要。在这里，我们获取第一和最后一张图像，微调一个剪辑成功检测器，并使用它向我们的语言模型提供二进制成功与失败的信息。

嗯。![](img/dde5df4844de2a3e55cc92f364263e12_19.png)

在结果方面，我们可以看到非常相似的长期评估。但是这里有趣的是，我们实际上能够基本上在机器人上实现所有这些不同的自动反馈机制，因此它能够推理并从这里的事情中恢复。你会看到它试图去桌子，但人类实际上在说嘿。

我改变主意，然后人类又改变主意，再次请求来回移动，而机器人能够做到这一点，你知道，也许我们此刻有点触及语言模型，但语言模型也能够重新规划，并确保人类的意图得到满足。

我们也尝试了，我不确定这个视频是否展示了，但在我走来走去，敲打机器人的手，强迫成功检测器告诉它“嘿，你搞砸了，知道吗，重试一下。”时的对抗输入情况。啊。我们也在几个不同领域进行了测试，包括一个模拟的桌面操作领域和一个现实世界的操作领域。

我们发现这比仅仅使用视觉特征（如 clipport）要好得多。我认为这确实反映了一个趋势，我在 2018 年时非常欣赏过。一位机器人教授曾说，当他们查看所有阻碍机器人学习大规模扩展的不同因素时。

它认为瓶颈在于高层次的语义规划，即关于常识推理的思考。我认为在 2022 年和 2023 年，语言模型可以提供一种路径，至少在过渡期间可以进行卸载。如果语言模型是 API，那么你可以将这些视觉语言模型作为对象检测器引入，随着成功检测器、VQA 和语言模型的提升。

你可以将它们全部纳入其中，这个行为有点像一个生命救助器，如果你的机器人当前没有常识推理能力。这些其他模型可以作为一个支架，帮助你赶上它们当前的能力，或许在未来你能超越语言模型的知识，但在短期内，我们似乎可以利用它们来加速我们在现实世界中的操作。

现在转到下一步，我们看到语言模型如何进行规划，看到视觉语言模型如何帮助规划。现在我们要稍微调整一下思路，考虑视觉语言模型如何帮助解决机器人学习面临的其他瓶颈。其中之一是数据收集非常昂贵，正如我们之前提到的，我们确实有这个 130。

000 集演示数据集，但它是在一年半的时间内收集的，成本很高，包括资源、时间和金钱，涉及许多机器人，当然这些任务也有些限制。我们使用了 700 条非常模板化的命令指令给遥控操作员，因为我们知道如果我们为每个模板任务收集足够的数据，这样可以扩展。

我们可以完成这个特定任务，下面是之前有人提到的流程：我们给出这个 pick coA 指令，操作员在现实世界中控制机器人，完成任务，标记这一集为结束，然后将其划分到这个大型橙色数据集中，而这个大型橙色数据集是我们在所有之前项目中训练控制策略所使用的。

我们最初考虑的是增加一些众包的回顾注释，如果你熟悉的话，包括回顾经验重放和强化学习，结合目标条件。也许机器人做了一些不仅仅是高层模板指令的事情，我们可以让人类更详细地描述机器人所做的事情，也许它拿起了桌子右侧的可乐，也许它把可乐拿起来后打翻了，也许它很慢地把可乐移动到中间。这一演示包含了大量语义多样性，而这些是高层模板“拿可乐”指令所无法完全捕捉到的。因此，我们用这些详细描述标记了这个大橙色数据集的 3%。

然后我们应用了一种伪标签策略，这种策略在其他领域（如视频预训练和其逆动力学模型）中已经被看到。但我们将其应用于数据集中所包含的语义说明，所以第一步。

我们在你的小标签数据集（占你主要数据集的 3%）上预训练了一个 CLIP 模型。然后，你可以使用该训练后的 BLM 数据来标记之前所有的模板指令演示，也就是那 130,000 个 episode 的数据集。现在你有了一个重新标记的数据，里面有大量有趣的语义指令，然后我们将所有这些数据集输入到 RT1 中，并像往常一样训练一个语言条件行为克隆策略。

但尽管我们通常只使用数据集 B，即橙色数据集，现在我们使用所有三个数据集。最后，我们在完全新的、未见过的指令上进行评估。在之前的工作中，我们主要是在 700 条模板指令上进行评估。但在这项工作中，我们实际上超越了这一点，我们可以输入几乎任何你认为可能成功的内容，并且可以用各种方式表达，你可以添加拼写错误，甚至可以通过参考语义概念来表达空间概念，我们会看看效果如何。

这可能有效的原因是，这里是左侧的 TN 嵌入，右侧是相同的嵌入，但左侧是根据用于收集该情节的原始模板指令着色的，右侧是视觉语言模型认为如果允许它为该情节添加一个自由形式的自然语言标题，它会是什么。你会看到，左侧有这些大的“可口可乐”聚类，像是数百或数千个情节，但我们都称之为“可口可乐”。而在右侧，我们可以扩展这些概念，实际上这个情节是在选择红色的可口可乐，这个情节是在选择皱巴巴的可口可乐，这个是在选择靠近薯片袋的可口可乐。因此，你可以通过使用语言作为多样性机制，充分利用同一基础数据集，扩展你所考虑的概念。例如，在中间，你会看到“打开上面的抽屉”变成“抓住并拉出上面的抽屉”。

我们有像“左中”的内容，对于底部的情节，“从白色碗中挑选绿色薯片”变成“从碗中抬起薯片袋并将其放在桌子的左下角”。所以你有很多这样的语义空间概念，现在将进入你的目标监督标签中。

我有一个问题。那时我站在他们的角度。我听到的语言似乎是你必须要的。是的，伟大的问题，所以我想如果我能稍微重新表述一下，问题在于，实际上这是一个非常困难甚至可能是不可解决的问题，如何将你在现实中看到的所有语言概念映射到一些具体的体现类型的情节上。

这里我想说的是，我们无疑在引入很多我们的先验和偏见，比如我们称之为“左边”时，你是指左边 10 厘米还是 2 厘米，这些词语的定义对我们来说意味着什么，生成这些字幕的云计算评级者对它们的理解又是什么，它们对机器人意味着什么，对语言模型又意味着什么，可能这些都是略有不同的，但希望至少如果它们大致相似，我们就能得到方向上的正确改进。因此，我认为这些具体定义的细微差别，以及这些词语的实际语义，可能现在超出了范围，但或许在更高的层面上，我们会进一步探讨。不过，我觉得基本的标准实在是太低了，我们有 700 个模板指令，基本上是独热编码，我们只想让这些更接近自然语言，即使只是一点点，我认为我们至少在朝着这个方向努力，借助这些视觉语言模型。

希望这能回答你的问题。我们还在左上角与几个基线进行了比较。我们考虑如果只在这 3%的高端人类评定标签上进行训练会怎样？如果我们只在原始的 RT1 数据集上训练会怎样？如果我们同时在这两个上进行训练又会怎样？再加上我们的 VLN 提供的所有预测会怎样？有趣的是，你知道重新标记似乎是普遍有益的，我们只在这个项目中新颖的指令上进行了评估，这是机器人项目中第一次只在我可以输入的内容上进行测试，我输入的任何想法都成为了测试集，我们必须确保这些内容从未包含在训练覆盖中，你可以在右边看到这些有趣的例子，比如“将孤独的物体移到其他地方”，我完全不知道这怎么工作，像是提起黄色矩形，谈论颜色，以及“将右侧的苹果移到左边”，实际上场景中有两个苹果，并且在我们的训练演示中。

我们从未收集过包含重复物体的场景，因为我们考虑到这种低支付模态问题，如果你说“挑选可乐”，而这两个可乐就会很难确定该做哪个，但通过语言标签，看起来也许你现在可以做到，所以即使我们从未在有两个苹果的场景中进行训练，现在你可以在这些场景上进行评估，并仅通过语言指定你想选择哪个苹果，这实际上也运作得相当合理。

最后一个例子我觉得很有趣，我们尝试的单个可乐做了一个新颖的行为，推动到左侧，这不是一个模板化的指令，我们只是让可乐靠近其他物体，比如说可乐靠近苹果，靠近海绵，所以推动可乐进入空气的这个动作其实从未包含过，但也许在某个标签中有，比如说如果你看到“可乐靠近苹果”，而苹果在左边，再看到“可乐靠近海绵”，海绵也在左边，你可能会推测模型可以推广，理解“左”意味着桌子的一侧，而不是特定物体，所以这可能是发生的情况，但这并不明确。我只是想到了一些东西，打字就变成了实际情况，我们确实希望在未来更定量地探索这一点，左下角当然是，我认为与非视觉增强进行比较，所以也许你可以仅通过语言获得这些有趣的概念。

在这里，我们添加了随机噪声，或者用 Madlib 风格仅仅替换单词。我们甚至可以在这种情况下使用 LOM GT3 来提出现有指令的改写。但是我认为我的收获是，你确实需要视觉基础，以便视觉语言模型能够在某一时刻确实说出这个说明在事实上的准确性，而这可能是对机器人来说有趣的内容。

这个微调过程同时提供了这两点。是的。不。喂。是的，确实这些只是五个评估指令的某些子集，但我们有超过 60 个，我们没有进行完整的定量消融。例如，正如我们在 RT1 中所做的那样，我们有一个未见任务集的场景，这个场景是组合性的，你会看到，比如将可乐移动到苹果附近，或者将苹果移动到海绵附近，但我们会保留将可乐移动到海绵附近，并对此进行测试。但是在这种情况下，我认为我们可以更进一步，因为我们的语言是完全自由形式的，组合空间会更大，因此我们确实尝试了一些组合评估，但我们可以做得更深入。

我在时间上怎么样？好吧，可能 10 分钟，我会尽量快点结束。那么，结论是两个部分，对吧？第一课是利用基础模型，第二课是将它们作为数据增强，第三课是确保我们的离线数据集足够强大，以便这些不同的行为存在，并且你可以用语言描述它们。如果你没有足够多样化的行为，无论你的标注多么好，你可能无法引导出所有有趣的概念。

对我来说，也许最令人兴奋的是，实际上一些标签噪声是可以接受的。在监督学习和模仿学习中，通常需要非常干净的标签，总是 100%真实的。你不想从一些大比例不准确的噪声数据中学习，但在我们的案例中，似乎一些标签噪声是可以的，视觉语言模型并不总是准确预测场景的描述，我认为当噪声过高时确实会造成伤害，但在较低水平上，它似乎仍然可以处理得很好且足够稳健。

所以这是对一些使用这种大型语言配方的个别作品的深入探讨。基础模型，来自机器人系统不同部分的数据集。这是开始时的那种介绍，我希望你们至少能看到我们的团队如何尝试将这些原则应用于加速现实世界中的机器人学习。![](img/dde5df4844de2a3e55cc92f364263e12_21.png)

随着我们看到这些不同类型的成分和课程如何映射到机器人系统的不同部分，以实现技能学习，这就是我们之前讨论的 RQ1，关于规划的部分。他们可以通过与视觉语言模型的紧密反馈来实现，这在低级控制的独白中没有涉及，但我们团队的一项令人兴奋的工作实际上是使用语言模型来预测在机器人上直接执行的代码。也许这些低级控制器的语言模型，他们阅读教科书，阅读过 Ross 等人的文献，阅读过您所提到的文档代码，并能够为这些机器人编写代码，我们可以执行这些代码。

对于数据文档，我们看到了与视觉语言模型的对话，尽管我在这里没有讨论。但对于以对象为中心的表示，比如特定对象的特征激活图，我们可以将其用作任务表示来映射场景。在 NL 地图中，他们为我们所研究的微型导航进行了对象中心的导航。我希望在接下来的几周和几个月中。

我们还有更多的行和条目可以添加。但我认为这种心态是一种非常令人兴奋的研究方向，关于如何应用这些关于基础模型和离线数据集的高层概念，看看目前机器人系统中存在的内容，发现许多仍然可用的空白和机会，我们可以从探索性试点到更广泛的评估，真正构建出强大的系统。

我认为这两者都具有很高的价值。所以我想总结一下，探索所有这些互补的方向非常有趣。但仍然有一些重大问题，关于我们如何能将这些概念进一步发展，以及随着基础模型的改善、在线数据的可用性增加、更多数据的同质化和标记化、互操作性提高，这些趋势和想法可能如何演变。我认为来自其他领域的许多概念，例如语言学和视觉，以及所有大规模扩展相关的问题，都在语言基础模型中得到开创。

希望这些想法能够渗透到机器人技术中。也许机器人技术甚至可以通过提供具体的行动来回馈，提供因果数据，可能会改善一些没有具体体现的这些大型语言模型的推理质量。对此，我想感谢大家的时间，感谢 David 提供的支持，并对论文或任何高层次的问题持开放态度。

非常感谢。嗯。嗯。8 是。我。等一下啊。诶。但。你买打。是的，问得很好，所以我想这个问题是关于需要更多语义推理的任务，比如说以某种速度操作，或者可能是在提示本身的问题中涉及的数字推理。我会说，对于很多常识推理，比如你知道连续丢弃三瓶可乐，我认为语言模型现在对此非常擅长。因此，对于 saycan 规划器，它会预测连续丢弃三次可乐。而对于低级技能策略学习，我认为这更具高方差。我会说，目前我们并没有真正根据速度或具体操作进行条件设置，但这可能是我能做到的，如果你能重新标记，比如慢慢捡起可乐与快速捡起可乐，这可能是视觉语言模型能够识别的。

检查。我今天不太认识那些人。评估中有多少个教程，如果我们有两个任务的数据？

提到某件事，因为那只有在我正确谈论的情况下才会想到，我想拾起政府法律。我想做到这一点。很好的问题，问题是我们在什么规模上看到组合泛化开始发生，可能是在你看到一个方块的颜色，然后想要在新颜色上进行评估。我认为这是个很好的问题，不幸的是我的回答会非常模糊，这取决于你如何定义你的任务，取决于你数据的规模，也取决于你试图跨越的概念。我认为已经有很多尝试基本上在学习和机器人领域中正式化泛化的意义，即使在我们考虑的特定设置中，我认为没有明确的趋势，比如你可以说哦，是的。

这是我需要达到的数字，你知道我可以在 X Y Z 维度上进行泛化，比如你可以评估所有这些，但我认为这并不会帮助你预测新趋势，至少在现在。我认为我们可能还差一个数量级，才能开始进行非常广泛的泛化陈述。

泛化能力，我认为你知道，将我们的数据集规模增加一到两个零，我们就可以开始在任务对象技能方面谈论这一点。是的。嗯。你怎么了。嗯。所以请你指定检查的内容。并注意疫苗。所以我们发个你系。是的。

正在扩展。对。是的，非常敏锐的观察，问题是，在 Sacan 中，预测这些右侧的价值函数所存储的功能只限于某些特定的任务，因此这是否是瓶颈，我会说是的，100%你系统能够执行的任务数量的扩展，能够提供给规划者作为选择的自助餐确实是瓶颈。不管你的规划者多么优秀，如果你只能完成三项任务，那么只能对这三项任务的某些组合进行映射以符合高级指令。因此，随着更多任务的增加，机器人低级技能能力的提升，你实际上是在为机器人能够尝试执行的高级指令的覆盖添加精确度。这是我今天看到的主要瓶颈之一。

对。放到英文点吗。很好的问题，我们是否尝试过用 RLHF 或 RL 的 RT1。我认为简短的答案是我们有一些正在进行的项目，但目前所有项目仍然使用这种实现学习损失。我认为我把这个 Mohead 赌注视为一种存在证明，它确实有效，虽然不便宜，但确实有效并且可以扩展，这至少是一个良好的起点。我们在接下来的几个月和几年里的主要希望是，能否在此基础上有所改善，是否能将离线改进加入方程中，我是个 R 的人，所以我真的希望如此。

好啊。听到你的意见。让我重复一下。好。那个生日己看得不太好。我们会有不同的事件吗？走吧。是的，好的问题，关于任务平衡以及仅用文本数据是否足够帮助运动控制学习，我的希望是，当模型在机器人领域经历涌现时，我们已经在语言领域看到涌现，或许这些推理概念会在两者之间开始转移。我会提到一篇有趣的论文，我认为《维基百科能否帮助强化学习》，由 Shane 和其他人撰写，他们在维基百科上进行了大规模的策略网络预训练，仅使用文本，并用其初始化对 Atari 游戏的控制，这确实有效。所以，也许这是哲学上的问题，但也许在文本与行动数据之间有某种决策推理的转移。

对。好好。这是关于过去图像的问题。确实是个好问题，我绝对同意，你知道，传递六幅图像在执行任务时是不够的，比如清理我的整个房子，而你只能在最后的两秒钟内传递，真是让人失望。所以我认为这肯定会成为一个限制，尤其是当我们的任务变得更加复杂和长期时。我觉得这里还有另一个悬而未决的问题是上下文长度。我们有高维图像，即使通过标记学习来减少我们传递的补丁数量，它仍然是非常高维的，我们很快就会达到上下文长度的上限。我们能否在此基础上进行改进，或许是检索变换器或其他机制。

哦，这就是啊，审查的。这边现在有这个。真是个好问题，我认为我们希望在未来探索这个。但是由于上下文长度的限制，我们已经接近上下文长度的容量，仅仅是六幅图像，更不用说传递整个零样本行为或少样本行为的轨迹了，所以是的。

好的，对。好的，听到这个。很酷，谢谢大家。![](img/dde5df4844de2a3e55cc92f364263e12_23.png)
