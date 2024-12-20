# P39：【2025版】39. 特征提取.zh_en - 小土堆Pytorch教程 - BV1YeknYbENz

在这个视频中，你将学习如何从你的图像中提取特征，这些特征是你在计算图像之间的特征距离时使用的。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_1.png)

具体来说，你将学习如何使用训练在大量图像上的分类器来提取特征，这些分类器通常被认为是能够建模自然图像或照片的。



![](img/3a8eaa57ca3fd822faf3874ee8b322e7_3.png)

这些大量图像通常来自著名的大型数据集imagenet，我也会在这个视频中提到imagenet。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_5.png)

为了计算真实和假图像之间的特征距离，你需要一种从那些图像中提取特征的方法，你可以通过获取预训练分类器的权重来获得一个特征提取器，通常在大量图像上，并且理想情况下在与你想要提取的特征相关的类数据上，例如。

如果你的分类器是分类狗和猫的，那么它对狗和猫的特征提取会很好，但对于办公对象可能不行，这些预训练模型的权重本质上编码了特征，因为它们必须编码大量的特征。



![](img/3a8eaa57ca3fd822faf3874ee8b322e7_7.png)

例如区分狗和植物，例如。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_9.png)

它必须弄清楚这个湿鼻子，金色的外衣，眼睛也是如此，也许还知道植物的形状看起来像什么。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_11.png)

这可能不是在最后一层。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_13.png)

这可能是一个中间层，有更多的节点在下游，他们必须在途中弄清楚一些这些特征，尽管，在现实中，提取的这些特征稍微更抽象，并不一定对应我们湿鼻子或金色外衣的概念，或者一个植物形状。

你可能在想是否需要建立一个新的分类器并训练它，每次评估时。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_15.png)

幸运的是，答案是否定的，因为有些神经网络已经预先在数百万张图片上训练过。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_17.png)

并且它们已经识别了成百上千个类别，你可以将它们插入并使用，来提取图片中的特征，这些图片通常被认为是足够广泛的，适用于大多数自然图片，现实世界的照片，因此，这些预训练的分类器可供公众免费使用。

并提供了一种可以在各种GAN中应用的方法。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_19.png)

所以，你训练的所有不同的GAN。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_21.png)

这些分类器可以分类很多，许多不同的类别，因此，它们在网络中编码了大量的相关特征。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_23.png)

好的，要使用这个预训练的分类器，实际上你不需要在这个输出上使用分类的最终任务，你不关心实际的分类器部分，但网络的其余部分是有价值的，它的权重学习了有助于最后分类任务的重要特征，因此，结果。



![](img/3a8eaa57ca3fd822faf3874ee8b322e7_25.png)

你可以砍掉这个最后的分类层，并只抓取早期层的输出。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_27.png)

那个层包含关于你输入的图像的有用信息，在这里，你看到一个通用卷积神经网络或CNN的最终层，卷积池化，全连接或线性层，激活函数紧接着输出预测。



![](img/3a8eaa57ca3fd822faf3874ee8b322e7_29.png)

你可以去掉最后一个全连接层，因为最常见的输出特征来自池化层。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_31.png)

在那最后一个用于分类的全连接层之前，因为这个池化层，你有最精细的特征信息，这个层必须编码了大量信息，所以它可以将这些信息传递给下一个全连接层。



![](img/3a8eaa57ca3fd822faf3874ee8b322e7_33.png)

只需再添加一个全连接层就可以分类这张图片。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_35.png)

所以你可以像剪掉这个部分一样截断网络。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_37.png)

使用最后一个池化层，这意味着从这一层出来的值，你取这些本质上的中间值。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_39.png)

这些值，假设有一百个值从你的。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_41.png)

假设五千五百个输入，所以这里出来的一百个值代表这个模型对这个特定输入图像提取的特征。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_43.png)

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_44.png)

你可以把这个层看作是特征层，因为它正在提取这些特征值。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_46.png)

并且，你也可以立即告诉这个特征层输出的值将远远少于你的输入。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_48.png)

所以，它压缩了输入图像中像素值到这些100个特征中。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_50.png)

使用更早的层作为特征层也是可以的。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_52.png)

所以使用最后一层只是约定俗成，因为它包含了最多信息。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_54.png)

但也可能过度拟合到你的数据集和任务上，就像这里的分类类别。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_56.png)

所以，早期层更适合获取更原始的信息。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_58.png)

随着你在网络中回溯到更早的卷积层，或者通常是池化层，对于这些早期层，在一种极端情况下。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_60.png)

如果你去最早的层次，可能只是意味着垂直边缘检测，并能够在输入的垂直边缘周围提取特征。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_62.png)

而不是这里的最后一层池化。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_64.png)

这将具有适用于特定图像类的特征。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_66.png)

例如，图像中是否有猫，如果输出预测图像中有猫。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_68.png)

因此，选择你的特征层的名称是你可以实验的事情，但我通常建议从最后一层池化开始。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_70.png)

因为它将基于大型数据集进行训练，任务广泛。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_72.png)

并且这个大型数据集通常是ImageNet，所以，一个好的开始方式通常是使用一个在ImageNet上预训练的分类器，ImageNet是一个包含1400多万张图片和2万个类别的数据集。

你们可以在这张图上看到一些类别。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_74.png)

ImageNet包括各种犬类品种，以及猫的品种和物种，几乎包含了你能想象到的任何类型的动物。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_76.png)

以及几种类型的物体，这个数据集本质上。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_78.png)

提供了大量的信息，可以被有意义地编码到一个分类器中，它是在这个数据集上训练的。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_80.png)

它提供了大量的信息，可以被有意义地编码到一个分类器中，它是在这个数据集上训练的，并且你从那个分类器中提取的特征有时被称为ImageNet嵌入。



![](img/3a8eaa57ca3fd822faf3874ee8b322e7_82.png)

因为它们将图像中的信息嵌入到一个更小的信息向量中。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_84.png)

使用从这张ImageNet数据集训练的网络中获取的权重，然后引导这些特征的嵌入过程。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_86.png)

这些嵌入只是存在的向量。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_88.png)

在更广泛被称为特征空间或嵌入空间，所以你可以想象一个像这样的图像进入。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_90.png)

输出特征实际上只是来自负向的一个向量。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_92.png)

三，二，空间中表示向量的五个不同值。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_94.png)

然后代表你的特征，这也可以被称为嵌入。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_96.png)

总结来说，你看到了如何从一个预训练的分类器中获取特征提取器，通过切断网络并使用输出层之前的层的权重。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_98.png)

最常见的是使用最后的全连接层。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_100.png)

但如果你使用更早的层，你可以提取更原始的特征，例如垂直边缘或自然外观的模式，使用训练在大型数据集上的分类器。



![](img/3a8eaa57ca3fd822faf3874ee8b322e7_102.png)

使用训练在大型数据集上的分类器，你可以提取更原始的特征，例如垂直边缘或自然外观的模式。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_104.png)

例如ImageNet，它有数百万张图片，你可以在图片上编码有意义的特征。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_106.png)

在接下来的讲座中，你将学习一种流行的分类器，它使用ImageNet数据。

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_108.png)

![](img/3a8eaa57ca3fd822faf3874ee8b322e7_109.png)