# P13：【2025版】13. 批归一化详解.zh_en - 小土堆Pytorch教程 - BV1YeknYbENz

训练鹅需要很多时间，尤其是如果你想用它们进行非常酷的应用。

![](img/ab852ff5e5a2bdae41557cea570f841c_1.png)

就像本课程讨论的那样，生成式对抗网络在训练过程中往往比较脆弱。

![](img/ab852ff5e5a2bdae41557cea570f841c_3.png)

因为它们不像分类器那样直接，有时候，生成器和判别器的技能并不总是那么一致，它们本可以更好，出于这些原因，所有可以加速并稳定训练的技巧对于这些模型都是至关重要的。



![](img/ab852ff5e5a2bdae41557cea570f841c_5.png)

并且批标准化已被证明在这方面非常有效。

![](img/ab852ff5e5a2bdae41557cea570f841c_7.png)

所以让我们开始，我会提醒你归一化对训练的影响。

![](img/ab852ff5e5a2bdae41557cea570f841c_9.png)

我会向你展示内部协变量偏移的含义以及为什么会发生。

![](img/ab852ff5e5a2bdae41557cea570f841c_11.png)

最后，你将能够将所有这些与批标准化联系起来。

![](img/ab852ff5e5a2bdae41557cea570f841c_13.png)

并且你会对这为何加速并稳定神经网络的训练产生一些直觉。

![](img/ab852ff5e5a2bdae41557cea570f841c_15.png)

所以假设你有一个非常简单的神经网络，有两个输入变量，x1表示大小，x2表示毛色，有一个激活输出，这个示例基于这些特征判断是否是猫或不是猫。



![](img/ab852ff5e5a2bdae41557cea570f841c_17.png)

并且让我们假设x1的分布在这里，你的数据集大小通常以这种方式围绕一个中等大小的示例分布，例如，非常少的，极端小或极端大的例子，而毛色分布的偏斜稍微向更高的值倾斜，在这里，具有更高的均值和更低的标准差。

所以假设这里的更高值代表更深的毛色，当然，请注意，比较毛色的相同值，与比较苹果与橘子有点类似，因为深色毛发和大体型并没有实际的相关性，然而，这些不同分布的结果会影响神经网络的学习方式。



![](img/ab852ff5e5a2bdae41557cea570f841c_19.png)

例如，如果它试图达到这个局部最小值。

![](img/ab852ff5e5a2bdae41557cea570f841c_21.png)

并且输入数据分布差异很大，这个成本函数会被拉长。

![](img/ab852ff5e5a2bdae41557cea570f841c_23.png)

因此权重的变化，与每个输入的关系会有不同的影响，对成本函数的影响各不相同，这使得训练相当困难，这会减慢训练速度，并且高度依赖于权重的初始化。



![](img/ab852ff5e5a2bdae41557cea570f841c_25.png)

此外，如果新的训练或测试数据有。

![](img/ab852ff5e5a2bdae41557cea570f841c_27.png)

假设，颜色非常浅，因此数据分布可能会改变或以某种方式变化。

![](img/ab852ff5e5a2bdae41557cea570f841c_29.png)

那么成本函数的形式也可能会改变，所以可以看到这里稍微圆一些，并且最小值的位置也可能会移动，即使决定某物是否为猫的真实情况保持不变。



![](img/ab852ff5e5a2bdae41557cea570f841c_31.png)

标签没有改变，这就是所谓的协变量偏移，这在训练集和测试集之间经常发生，如果没有采取预防措施。

![](img/ab852ff5e5a2bdae41557cea570f841c_33.png)

数据分布如何变化，但如果输入变量x1和x2被归一化。

![](img/ab852ff5e5a2bdae41557cea570f841c_35.png)

意味着新输入变量x1'和x2'的分布，将更加相似，将更加相似，说意味着等于零，标准差等于一。

![](img/ab852ff5e5a2bdae41557cea570f841c_37.png)

那么成本函数也会在这些两个维度上显得更平滑、更平衡。

![](img/ab852ff5e5a2bdae41557cea570f841c_39.png)

作为结果，训练实际上会容易得多，并且可能更快。

![](img/ab852ff5e5a2bdae41557cea570f841c_41.png)

无论原始输入变量的分布如何变化。

![](img/ab852ff5e5a2bdae41557cea570f841c_43.png)

例如，从训练到测试，归一化变量的均值和标准差会被归一到相同的位置，所以围绕均值零和标准差一。

![](img/ab852ff5e5a2bdae41557cea570f841c_45.png)

并且对于训练数据，这是通过批处理统计量来完成的，因此，在你训练每个批次时，你计算平均值和标准差。

![](img/ab852ff5e5a2bdae41557cea570f841c_47.png)

并将它们调整为零均值和一标准差，对于测试数据，你所做的是，你可以查看统计量，这些统计量是在训练过程中收集的，并用它们来使测试数据更接近训练数据，以进行归一化，这种协变量偏移的影响将显著降低。

这就是输入变量归一化的主要效果，平滑成本函数，减少协变量偏移，然而，协变量偏移不应该是一个问题。

![](img/ab852ff5e5a2bdae41557cea570f841c_49.png)

如果，你只确保，你的数据集的分布与你建模的任务相似。

![](img/ab852ff5e5a2bdae41557cea570f841c_51.png)

所以测试集与你的训练集相似，在分布方面。

![](img/ab852ff5e5a2bdae41557cea570f841c_53.png)

神经网络就像我所展示的这个，实际上是容易受到称为内部协变量偏移的东西的影响。

![](img/ab852ff5e5a2bdae41557cea570f841c_55.png)

这就意味着内部隐藏层的协变量偏移。

![](img/ab852ff5e5a2bdae41557cea570f841c_57.png)

所以，取神经网络的第一个隐藏层的激活输出的输出，看这里这个节点，在训练模型时，影响激活值的所有权重都会被更新，因此，激活中的值的分布会随着训练的变化而变化，并且会受到这种训练过程的影响。

这使得训练过程变得困难，由于与输入变量分布变化相似的偏移，例如，毛色，只对内部节点有影响，这些节点比颜色和大小的意义更抽象。



![](img/ab852ff5e5a2bdae41557cea570f841c_59.png)

现在批标准化试图解决这个问题。

![](img/ab852ff5e5a2bdae41557cea570f841c_61.png)

并基于每个输入批次计算的统计数据对所有这些内部节点进行标准化，这样做的目的是减少内部协变量偏移，这还具有平滑成本函数的额外好处，从而使神经网络更容易训练并加快整个训练过程。

批标准化中的批字意味着使用批次统计。

![](img/ab852ff5e5a2bdae41557cea570f841c_63.png)

随着您继续本周的视频，您将更熟悉这一点。

![](img/ab852ff5e5a2bdae41557cea570f841c_65.png)

![](img/ab852ff5e5a2bdae41557cea570f841c_66.png)

总之，批标准化平滑了成本函数并减少了内部协变量偏移的影响，它用于加快和稳定训练，在建立伟大的GANs时非常有用，需要注意的一点是，虽然批标准化对我们的GAN和更广泛的神经网络非常有效。

内部协变量偏移的理论背后并不具有决定性，我们仍在寻找其他原因的证据。

![](img/ab852ff5e5a2bdae41557cea570f841c_68.png)