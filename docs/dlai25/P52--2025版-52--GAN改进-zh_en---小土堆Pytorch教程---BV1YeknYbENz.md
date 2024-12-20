# P52：【2025版】52. GAN改进.zh_en - 小土堆Pytorch教程 - BV1YeknYbENz

![](img/6f6817cfd87724af6556f5ae1fc4904b_0.png)

在这个视频中你将了解GAN的改进，你将看到GAN如何随着时间的推移而进步以及哪些方法有助于提高它们。

![](img/6f6817cfd87724af6556f5ae1fc4904b_2.png)

你可能记得在课程的开始时看到过这个，这是一条来自Ian Goodfellow的推文，他被认为是GAN的创造者，这是一个可视化，展示了多年来GAN的进步速度。



![](img/6f6817cfd87724af6556f5ae1fc4904b_4.png)

它们基本上从几乎无法辨认的面部黑白图像。

![](img/6f6817cfd87724af6556f5ae1fc4904b_6.png)

进步到到2018年你可以相信的高质量图像，看起来是真实的人。

![](img/6f6817cfd87724af6556f5ae1fc4904b_8.png)

它们自从GAN训练稳定性的第一次主要改进以来已经变得更好。

![](img/6f6817cfd87724af6556f5ae1fc4904b_10.png)

你可能记得在模式崩溃视频中提到过，不稳定的GAN通常意味着训练它更长时间。

![](img/6f6817cfd87724af6556f5ae1fc4904b_12.png)

不会对它有帮助，在这个例子中，生成器在这里卡在某种局部最小值。

![](img/6f6817cfd87724af6556f5ae1fc4904b_14.png)

并且只生成单一的东西，这里只生成4。

![](img/6f6817cfd87724af6556f5ae1fc4904b_16.png)

而当它应该生成多种东西的时候。

![](img/6f6817cfd87724af6556f5ae1fc4904b_18.png)

你已经看到了许多防止模式崩溃的方法，但一个有趣的事情是看看这两个样本。

![](img/6f6817cfd87724af6556f5ae1fc4904b_20.png)

你想要左边的这个有很多变化的东西，这里有很高的样本标准差。

![](img/6f6817cfd87724af6556f5ae1fc4904b_22.png)

但在这里又有模式崩溃，当你看一个小批次，比如三样本。

![](img/6f6817cfd87724af6556f5ae1fc4904b_24.png)

与这里左边你想要的比较，你会发现标准差非常低。

![](img/6f6817cfd87724af6556f5ae1fc4904b_26.png)

这意味着这三个样本之间的变化很小。

![](img/6f6817cfd87724af6556f5ae1fc4904b_28.png)

因此这个小批次中的每个样本，可以表明GAN固有的变化程度，结果。

![](img/6f6817cfd87724af6556f5ae1fc4904b_30.png)

这些图片的标准差可以显示你的生成器是否接近模式崩溃，因为标准差会很低。

![](img/6f6817cfd87724af6556f5ae1fc4904b_32.png)

为了缓解这个问题，也许你可以将这个信息传递给判别器。

![](img/6f6817cfd87724af6556f5ae1fc4904b_34.png)

然后判别器可以惩罚生成器，或者给生成器反馈，试图，防止它崩溃。

![](img/6f6817cfd87724af6556f5ae1fc4904b_36.png)

因为如果将这个信息传递给判别器而不是只给一个样本。

![](img/6f6817cfd87724af6556f5ae1fc4904b_38.png)

判别器现在可以看到所有这些样本，并且意识到可能这里存在一些模式崩溃，当然它不会看到这些所有样本。

![](img/6f6817cfd87724af6556f5ae1fc4904b_40.png)

它会通过感知小批次的标准差来感知它们，另一种研究人员改进GAN稳定性的方法是强制执行一Lipschitz连续性。



![](img/6f6817cfd87724af6556f5ae1fc4904b_42.png)

在使用Wasserstein损失或W损失时。

![](img/6f6817cfd87724af6556f5ae1fc4904b_44.png)

这可以防止模型学习得过快，并且保持有效的W距离，你在这个专业内周三的视频中已经看到过，你想要确保函数的梯度在所有这个点上保持在这些绿色三角形中，所以这是在观察这个点，并且观察到这个点将产生不同的梯形。



![](img/6f6817cfd87724af6556f5ae1fc4904b_46.png)

观察这里将产生不同的梯形。

![](img/6f6817cfd87724af6556f5ae1fc4904b_48.png)

所以这是在观察这个点，并且观察到这个点将产生不同的梯形。

![](img/6f6817cfd87724af6556f5ae1fc4904b_50.png)

观察这里将产生不同的梯形。

![](img/6f6817cfd87724af6556f5ae1fc4904b_52.png)

你想要确保你的功能保持在这些范围内，这意味着它不会超过线性增长，你已经在wgn gp中看到这一点。

![](img/6f6817cfd87724af6556f5ae1fc4904b_54.png)

它使用的是wasserson损失，这可以简化为w。

![](img/6f6817cfd87724af6556f5ae1fc4904b_56.png)

它通过使用梯度罚函数或这里的gp来保持一阶李普希茨连续性。

![](img/6f6817cfd87724af6556f5ae1fc4904b_58.png)

另一种方法是称为光谱归一化，在某种意义上，它非常类似于批量归一化，它通过归一化权重来实现这一点。

![](img/6f6817cfd87724af6556f5ae1fc4904b_60.png)

它通常以某种层的形式实现。

![](img/6f6817cfd87724af6556f5ae1fc4904b_62.png)

光谱范数层，这是另一种权重归一化方法，类似于批量归一化，它能通过保持值在一定范围内来稳定训练，我不会深入细节，因为关于光谱范数的数学细节会很复杂，但它保持了 Lipschitz 连续性。



![](img/6f6817cfd87724af6556f5ae1fc4904b_64.png)

所以不用太担心，这种方法有开源实现。

![](img/6f6817cfd87724af6556f5ae1fc4904b_66.png)

最后一种方式，研究人员尝试通过使用两种技术来稳定GANs，你将在随后的视频中了解更多细节。

![](img/6f6817cfd87724af6556f5ae1fc4904b_68.png)

这些技术被称为风格GAN。第一种技术是使用移动平均，它通过几个检查点计算加权平均值。

![](img/6f6817cfd87724af6556f5ae1fc4904b_70.png)

这意味着不再只选择一个生成器的权重。通常我们使用θi来表示生成器的权重。其中i等于，例如，第200步。无论你在哪里找到最佳的生成器检查点，你都想使用平均的θ。我将在这里用一个横线来表示平均值。

这将等于j到n的theta j的总和，所以你想要的是所有这些参数的平均值。

![](img/6f6817cfd87724af6556f5ae1fc4904b_72.png)

这会给你带来更平滑的结果，并且在训练过程中进行。

![](img/6f6817cfd87724af6556f5ae1fc4904b_74.png)

但这也可以在保存模型时进行采样。

![](img/6f6817cfd87724af6556f5ae1fc4904b_76.png)

在这里你可以看到平均和不平均之间的差异，在没有平均的情况下。

![](img/6f6817cfd87724af6556f5ae1fc4904b_78.png)

你可以看到也许那个生成器的检查点是在某种局部最小值，或者它只是使用某种平均在这里不是很平滑。

![](img/6f6817cfd87724af6556f5ae1fc4904b_80.png)

指数平均可以产生更平滑的结果。

![](img/6f6817cfd87724af6556f5ae1fc4904b_82.png)

这里的差异非常明显，第二种技术叫做渐进增长。

![](img/6f6817cfd87724af6556f5ae1fc4904b_84.png)

它逐渐在增加的图像分辨率上训练你的生成器。

![](img/6f6817cfd87724af6556f5ae1fc4904b_86.png)

这样它可以生成高质量图像，你将在即将到来的风格视频中了解更多关于这一点，所以现在你已经学会了一些方法，研究人员已经提高了GAN的训练稳定性，所以他们可以训练更长时间并得到改进。

你也会看到GAN在其他方面的进步。

![](img/6f6817cfd87724af6556f5ae1fc4904b_88.png)

GAN的另一个重大改进是他们的容量，生成器和鉴别器已经变得非常宽和深。

![](img/6f6817cfd87724af6556f5ae1fc4904b_90.png)

例如深度卷积生成对抗网络，你之前在第二周学过的dc gans。

![](img/6f6817cfd87724af6556f5ae1fc4904b_92.png)

这些模型可以更大，因为有了更好的硬件，如更好的gpu。

![](img/6f6817cfd87724af6556f5ae1fc4904b_94.png)

以及高分辨率的数据集，如hq数据集。

![](img/6f6817cfd87724af6556f5ae1fc4904b_96.png)

这是一个来自flickr的高分辨率人脸图像集，用于训练风格gan，最先进的gan，一些架构也开始适应这些高分辨率数据集。



![](img/6f6817cfd87724af6556f5ae1fc4904b_98.png)

gan的最后一个重大改进也是他们输出多样性的增加，他们的生成的图像有了越来越多的变化，这太棒了，因为我们想要模拟所有可能的人类面孔，不仅仅是在脸上生成，所以记住这也是一个关键标准，你现在评估你的GAN。

并且多样性的一种改进方式是通过拥有更多覆盖范围的更大数据集，因为训练图像的多样性增加也会导致输出结果的多样性增加，这些面孔实际上是由风格GAN生成的，斯大林有几种架构上的改变来增加多样性。

我想要知道你之前看到的使用小批量标准差方法，或许有助于训练的稳定性也可以通过避免模式崩溃来增加和改善多样性，并且趋向于多样性的变化，真实的图像在总结中有所改进，GANs从改进的训练中取得了很大进展。



![](img/6f6817cfd87724af6556f5ae1fc4904b_100.png)

稳定性、容量和多样性，一个稳定的GAN，较少的模式崩溃风险，可以让你训练模型更长时间，并且支持更大的模型。



![](img/6f6817cfd87724af6556f5ae1fc4904b_102.png)

更高分辨率的图像将提高您输出质量的，一些更改会增加您生成的图像的多样性。

![](img/6f6817cfd87724af6556f5ae1fc4904b_104.png)

你将在下一集的风格视频中再次看到这些。

![](img/6f6817cfd87724af6556f5ae1fc4904b_106.png)