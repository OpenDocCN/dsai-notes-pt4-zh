# P72：【2025版】72. pix2pix像素距离损失项.zh_en - 小土堆Pytorch教程 - BV1YeknYbENz

![](img/27cde80aadc413ff7229fb2a10a861bd_0.png)

像素 pix 的最后一个组成部分是你将要学习的像素距离损失项，首先我会回顾正则化以及添加额外损失项的含义。



![](img/27cde80aadc413ff7229fb2a10a861bd_2.png)

然后我会讨论在 pix2pix 中具体使用的这个像素距离损失项。

![](img/27cde80aadc413ff7229fb2a10a861bd_4.png)

它鼓励生成的输出和真实配对输出之间的像素距离，这是对特定于 pix2pix 生成器的额外损失项。

![](img/27cde80aadc413ff7229fb2a10a861bd_6.png)

记住你可以将损失形式化为对抗损失，这是你的主要损失。

![](img/27cde80aadc413ff7229fb2a10a861bd_8.png)

可能是 BCE 损失或 WASSON 损失 W 损失，你可以添加额外的损失项。

![](img/27cde80aadc413ff7229fb2a10a861bd_10.png)

例如 L1 正则化或梯度罚你第三周学过的。

![](img/27cde80aadc413ff7229fb2a10a861bd_12.png)

并且你可以添加一个 lambda 项来加权。

![](img/27cde80aadc413ff7229fb2a10a861bd_14.png)

那个损失函数通常小于一，以便它不会压倒你的主要对抗损失。

![](img/27cde80aadc413ff7229fb2a10a861bd_16.png)

对抗损失只是另一种谈论 GAN 损失的方式，在特定于 pix2pix 的情况。

![](img/27cde80aadc413ff7229fb2a10a861bd_18.png)

如果你想要你的结果看起来很好，你可以实际上添加一个额外的像素最后项在这里。

![](img/27cde80aadc413ff7229fb2a10a861bd_20.png)

给生成器一些关于真实目标图像的信息，所以我们可以尝试更紧密地匹配。

![](img/27cde80aadc413ff7229fb2a10a861bd_22.png)

所以让我们看看这意味着什么，所以像素距离损失项检查生成器生成的输出。

![](img/27cde80aadc413ff7229fb2a10a861bd_24.png)

加上那个真实目标输出，这是真实建筑。

![](img/27cde80aadc413ff7229fb2a10a861bd_26.png)

这里只是生成的。

![](img/27cde80aadc413ff7229fb2a10a861bd_28.png)

并且它计算两个像素之间的差异，试图鼓励生成的输出尽可能接近真实输出，因为如果这个像素距离真的很小。

![](img/27cde80aadc413ff7229fb2a10a861bd_30.png)

这意味着图像几乎完全相同。

![](img/27cde80aadc413ff7229fb2a10a861bd_32.png)

如果它真的很大，这意味着它们相距甚远并且生成的图像与真实目标输出相距甚远。

![](img/27cde80aadc413ff7229fb2a10a861bd_34.png)

因此像素距离损失在帮助这方面现实性方面非常有用，现在要完全接近真实的东西。

![](img/27cde80aadc413ff7229fb2a10a861bd_36.png)

但是请注意这里有一些棘手的事情。

![](img/27cde80aadc413ff7229fb2a10a861bd_38.png)

这也是监督的一层。

![](img/27cde80aadc413ff7229fb2a10a861bd_40.png)

生成器实际上看到了真实图像，它不仅仅看到真实输入图像。

![](img/27cde80aadc413ff7229fb2a10a861bd_42.png)

那只是为了条件。

![](img/27cde80aadc413ff7229fb2a10a861bd_44.png)

那是你看过的 one hot 类向量，但现在它以某种方式隐式地看到了配对输出，尽管这是以非常柔软的方式。



![](img/27cde80aadc413ff7229fb2a10a861bd_46.png)

主要是让那些样本看起来超级好。

![](img/27cde80aadc413ff7229fb2a10a861bd_48.png)

因此综合起来。

![](img/27cde80aadc413ff7229fb2a10a861bd_50.png)

pix2pix 生成器的总和是由 BCE 损失，那是你的对抗损失。

![](img/27cde80aadc413ff7229fb2a10a861bd_52.png)

加上这里的像素距离损失，更正式地说你可以说这是在多个样本之间看到时间。

![](img/27cde80aadc413ff7229fb2a10a861bd_54.png)

某种等待 lambda 你可以学习或调整或调整，并且看生成的和真实输出之间的差异。

![](img/27cde80aadc413ff7229fb2a10a861bd_56.png)

特别是 L1 或像素距离，他们。

![](img/27cde80aadc413ff7229fb2a10a861bd_58.png)

总之，选择选择。

![](img/27cde80aadc413ff7229fb2a10a861bd_60.png)

让生成器引用引号，看到轮子，通过将其损失函数的正则化项添加到其损失函数中。

![](img/27cde80aadc413ff7229fb2a10a861bd_62.png)

这取真实目标输出和假之间的像素差异。

![](img/27cde80aadc413ff7229fb2a10a861bd_64.png)

这鼓励生成器使图像类似于真实，在图像图像转换中，这是可以接受的。

![](img/27cde80aadc413ff7229fb2a10a861bd_66.png)

因为输出更直接地反映真实图像，总的来说，最后这一项鼓励生成的图像类似于真实目标。

![](img/27cde80aadc413ff7229fb2a10a861bd_68.png)

![](img/27cde80aadc413ff7229fb2a10a861bd_69.png)