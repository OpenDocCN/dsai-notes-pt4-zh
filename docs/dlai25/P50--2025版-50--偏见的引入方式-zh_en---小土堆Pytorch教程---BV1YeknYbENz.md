# P50：【2025版】50. 偏见的引入方式.zh_en - 小土堆Pytorch教程 - BV1YeknYbENz

现在你已经了解了机器学习中的偏见，你将了解偏见如何进入模型，我将向你介绍一些它们，但重要的是要知道还有其他途径。



![](img/eccbe8f7653afad24b5776e12c1b33bc_1.png)

一些还没有被意识到的，首先你将了解偏见如何进入你的模型在任何阶段。

![](img/eccbe8f7653afad24b5776e12c1b33bc_3.png)

然后更具体地看看脉冲系统中的样子。

![](img/eccbe8f7653afad24b5776e12c1b33bc_5.png)

偏见的一种方式可以进入你的模型是在训练阶段，所以首先你应该注意你的训练数据集，这可能意味着数据中没有变化。



![](img/eccbe8f7653afad24b5776e12c1b33bc_7.png)

或者数据可能表达出在收集过程中产生的偏见。

![](img/eccbe8f7653afad24b5776e12c1b33bc_9.png)

是否对有色人种的图像比例过少，有时如果你只是从互联网上抓取图像，这可能很难被发现。

![](img/eccbe8f7653afad24b5776e12c1b33bc_11.png)

例如，抓取名人的图像。

![](img/eccbe8f7653afad24b5776e12c1b33bc_13.png)

你也需要考虑数据是如何收集的，数据是否全部从一个地点收集，一次网络抓取，它是否由一个人或一个特定的人口群体收集。



![](img/eccbe8f7653afad24b5776e12c1b33bc_15.png)

记住，无论什么被认为是所谓的。

![](img/eccbe8f7653afad24b5776e12c1b33bc_17.png)

多样化的数据集也真的非常依赖于你对公平的定义。

![](img/eccbe8f7653afad24b5776e12c1b33bc_19.png)

正如你在之前的视频中所看到的，如果你使用的是标记数据，那么标签的多样性也会影响你的数据。

![](img/eccbe8f7653afad24b5776e12c1b33bc_21.png)

这是因为不同的群体可能会以不同的方式标记事物。

![](img/eccbe8f7653afad24b5776e12c1b33bc_23.png)

这可能会导致你的数据中产生固有的偏见，例如，如果你的标签者。

![](img/eccbe8f7653afad24b5776e12c1b33bc_25.png)

大多是男性，他们标记求职者的简历是否适合软件工程职位的面试。

![](img/eccbe8f7653afad24b5776e12c1b33bc_27.png)

这些只是你应该考虑的几个方面，当你在为你模型的训练数据做准备时。

![](img/eccbe8f7653afad24b5776e12c1b33bc_29.png)

在更广泛的社会中存在的偏见也可能影响你模型的所有部分。

![](img/eccbe8f7653afad24b5776e12c1b33bc_31.png)

另一个可以引入的领域是在你评估期间，例如，是谁创造了评估方法。

![](img/eccbe8f7653afad24b5776e12c1b33bc_33.png)

你可能在使用的评价方法对图像有偏见。

![](img/eccbe8f7653afad24b5776e12c1b33bc_35.png)

通常被认为是所谓的引号，在那个社会中，是好的还是正确的，或者在一种文化中。

![](img/eccbe8f7653afad24b5776e12c1b33bc_37.png)

但在另一个地方不是，一个简单的例子是汽车是靠左行驶还是靠右行驶。

![](img/eccbe8f7653afad24b5776e12c1b33bc_39.png)

如果他们通常在某一侧行驶，那么评估可能会反映出这一点，并对车轮在错误一侧的车辆进行不良评估或打分。

![](img/eccbe8f7653afad24b5776e12c1b33bc_41.png)

引号内，引号外。

![](img/eccbe8f7653afad24b5776e12c1b33bc_43.png)

这将完全取决于当地的驾驶法规。

![](img/eccbe8f7653afad24b5776e12c1b33bc_45.png)

另一个更具体的例子与ImageNet有关，用于训练Inception V3模型的数据集，该模型我也用于评估GAN和其他评估指标，我也用于评估GAN和其他评估指标。

imagenet中超过一半的图像来自美国和大不列颠。

![](img/eccbe8f7653afad24b5776e12c1b33bc_47.png)

与世界人口密度相比，这并不真正代表大多数人的来源。

![](img/eccbe8f7653afad24b5776e12c1b33bc_49.png)

这种不平衡导致系统不准确地将图像分类到类别中，这些类别根据地理位置不同，稻帽会被分类为头发吗，还是披肩会被分类为围巾，评估计算的方式，可以强化模型中的内部偏见，这可能让你认为模型在某项任务上表现很好。

但实际上它无法执行该任务，例如，研究人员实际上从收入水平不同的家庭中提取了物品，然后他们评估了这些产品的准确性，这些模型的最顶尖对象识别模型，结果表明，来自高收入家庭的图像在这些模型上具有更高的准确性。

所以这不太好，是那些开发这些模型的人可能会得出结论，他们的模型在视觉感知方面达到了人类水平，并且说视觉感知已经解决，但实际上，只是对特定社会经济地位的物体的高准确性，这并不是我对伟大模型的愿景。



![](img/eccbe8f7653afad24b5776e12c1b33bc_51.png)

所以现在偏见也可以通过架构引入。

![](img/eccbe8f7653afad24b5776e12c1b33bc_53.png)

优化代码的程序员的多样性如何，他们对。

![](img/eccbe8f7653afad24b5776e12c1b33bc_55.png)

什么是所谓的正确或错误，以及什么是好看或不好看。

![](img/eccbe8f7653afad24b5776e12c1b33bc_57.png)

毕竟，特别是在生成模型中，评估指标不是很好。

![](img/eccbe8f7653afad24b5776e12c1b33bc_59.png)

更加关注如何，选择各种问题在这个领域。

![](img/eccbe8f7653afad24b5776e12c1b33bc_61.png)

一旦你选择了重要的问题，人们会优化那些定义的正确、错误、好或坏。

![](img/eccbe8f7653afad24b5776e12c1b33bc_63.png)

的解决方案，这将影响研究方向的选择，并且。

![](img/eccbe8f7653afad24b5776e12c1b33bc_65.png)

例如，用于生成gan面孔的损失函数可以影响模型认为正确的内容，这可能是皮肤颜色更浅或更深的区别。

![](img/eccbe8f7653afad24b5776e12c1b33bc_67.png)

这只是一些例子，偏见可以出现在任何人可能设计、工程或接触的系统中，因为每个人都有偏见，无论是否有意识。



![](img/eccbe8f7653afad24b5776e12c1b33bc_69.png)

这里有一个偏见的例子，在一个叫做pulse的系统中，使用了一种称为style gan的argan状态，来从模糊的像素化图像中创建高分辨率图像，这是一种称为上采样的过程，所以从这里这些像素化图像作为输入。

到这些上采样图像作为输出，它做得相当好，这就是研究社区在2020年看到的最好的成果之一，你看到一个男孩被很好地上采样，然后你可以看到一个视频游戏角色被带入生活的酷应用。



![](img/eccbe8f7653afad24b5776e12c1b33bc_71.png)

然而，这不是一个像素化奥巴马照片（他是双种族）的示例。

![](img/eccbe8f7653afad24b5776e12c1b33bc_73.png)

被上采样成一个明显是白人的人，此外。

![](img/eccbe8f7653afad24b5776e12c1b33bc_75.png)

政治家亚历山德里亚·奥卡西奥-科尔特斯和演员，露西·露，也被变成他们不可辨认的自己。

![](img/eccbe8f7653afad24b5776e12c1b33bc_77.png)

这些在种族上可能更被描述为所谓的白人。

![](img/eccbe8f7653afad24b5776e12c1b33bc_79.png)

更接近风格生成器生成的平均面部特征，在它所训练的数据上。

![](img/eccbe8f7653afad24b5776e12c1b33bc_81.png)

在一个白人视频游戏角色上表现更好，而不是这些人种，这是一个相当强的迹象。

![](img/eccbe8f7653afad24b5776e12c1b33bc_83.png)

模型中存在偏见问题是一个事实。

![](img/eccbe8f7653afad24b5776e12c1b33bc_85.png)

但很难说，系统失败的地方在哪里，是风格生成网络，还是建立在风格生成网络之上的系统。

![](img/eccbe8f7653afad24b5776e12c1b33bc_87.png)

或者是风格生成网络所训练的数据集。

![](img/eccbe8f7653afad24b5776e12c1b33bc_89.png)

已经进行了研究来减轻GAN中的偏见，例如，使用对抗性损失来惩罚模型为偏见。

![](img/eccbe8f7653afad24b5776e12c1b33bc_91.png)

这很复杂，并且是重要的工作领域，所以我真的很鼓励你去检查一下。

![](img/eccbe8f7653afad24b5776e12c1b33bc_93.png)

到目前为止，这类问题开始在机器学习社区中受到更多关注。

![](img/eccbe8f7653afad24b5776e12c1b33bc_95.png)

《脉冲》论文的作者已经发布了一份警告声明。

![](img/eccbe8f7653afad24b5776e12c1b33bc_97.png)

关于应用他们的模型，总之，偏见可以从多个不同的路径被引入到模型中，这是一个非常真实的问题，就像脉冲所看到的那样，同样，也与指南针从前的视频有关，我希望这将提醒你尝试保持警惕，在自己的模型中存在偏见。



![](img/eccbe8f7653afad24b5776e12c1b33bc_99.png)

甚至找到在日常学习中克服这种偏见的方法，应用和推进机器学习，机器学习和GAN的世界，需要研究人员和实践者在他们的工作中思考这个问题，所以现在你拥有了这个重要的知识。

我希望你现在能够负责任地与最先进的GAN一起工作。

![](img/eccbe8f7653afad24b5776e12c1b33bc_101.png)

这将渗入产品并影响人们的生活，拥有巨大力量的，伴随着巨大的责任。

![](img/eccbe8f7653afad24b5776e12c1b33bc_103.png)

所以善用你的力量。