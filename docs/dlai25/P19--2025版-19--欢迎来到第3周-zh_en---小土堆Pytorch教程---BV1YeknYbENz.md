# P19：【2025版】19. 欢迎来到第3周.zh_en - 小土堆Pytorch教程 - BV1YeknYbENz

上周，你给你的GAN进行了一些小的升级，以生成更好的图像，本周你将探索GAN训练中的另一个主要问题，那就是GAN每次生成相同的东西，所以再次，训练于所有各种犬种的GAN只会生成金毛寻回犬。

而这显然是一个问题，这个问题发生是因为判别器在改进，但它卡在了说一张狗的图片看起来极度虚假或极度真实之间，毕竟它是一个分类器，所以它被鼓励这样说，你知道这越来越好，一卷或零假，但在一轮训练中。

如果鉴别器认为生成器的金毛寻回犬看起来真实，即使它看起来并不真实，那么生成器会紧紧抓住那只金毛寻回犬并只产生金毛寻回犬，所以现在当鉴别器发现这只金毛寻回犬是假的，在下一轮训练中，生成器不知道去哪里。

因为它真的没有其他任何不同图像的武器，这就是学习的结束，学习的最终结果是非常，对于这些网络来说非常糟糕，因此，每次深入一层都是因为二进制交叉熵损失，其中判别器被迫生成一个介于0或1的值。

尽管在0和1之间有无限个十进制值，它会随着变得更好而接近0和1，所以这周，你将被介绍一种新的损失函数，允许判别器说出负数，四或一百任何介于负，无穷大和无穷大之间的数字，这解决了这个问题。

使两个网络都能继续学习，就像你将在你的任务中实现这个惊人的GAN升级一样。