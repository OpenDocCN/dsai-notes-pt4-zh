<!--yml

类别：未分类

日期：2025-01-11 13:02:39

-->

# 标题名称为希望

> 来源：[https://arxiv.org/html/2310.15065/](https://arxiv.org/html/2310.15065/)

\translatedtitle

法语：标题名称为希望 \translatedtitle德语：Der Name des Titels ist Hoffnung \translatedtitle西班牙语：El nombre del título es esperanza

Ben Trovato [trovato@corporation.com](mailto:trovato@corporation.com) [1234-5678-9012](https://orcid.org/1234-5678-9012 "ORCID标识符") ， G.K.M. Tobin [webmaster@marysville-ohio.com](mailto:webmaster@marysville-ohio.com) 文档清晰度研究所P.O. Box 1212DublinOhioUSA43017-6221 ， Lars Thørväld The Thørväld Group1 Thørväld CircleHeklaIceland [larst@affiliation.org](mailto:larst@affiliation.org) ， Valerie Béranger Inria Paris-RocquencourtRocquencourtFrance ， Aparna Patel Rajiv Gandhi UniversityRono-HillsDoimukhArunachal PradeshIndia ， Huifen Chan 清华大学30 Shuangqing RdHaidian QuBeijing ShiChina ， Charles Palmer Palmer Research Laboratories8600 Datapoint DriveSan AntonioTexasUSA78229 [cpalmer@prl.com](mailto:cpalmer@prl.com) ， John Smith The Thørväld Group1 Thørväld CircleHeklaIceland [jsmith@affiliation.org](mailto:jsmith@affiliation.org) 和 Julius P. Kumquat The Kumquat ConsortiumNew YorkUSA [jpkumquat@consortium.net](mailto:jpkumquat@consortium.net)(2018年；2007年2月20日；2009年3月12日；2009年6月5日)

###### 摘要。

一篇清晰且文档齐全的LATEX文档，作为一篇格式化文章展示，供ACM在会议论文集或期刊上发表。基于“acmart”文档类，本文介绍并解释了许多常见的变体，以及作者在准备工作文档时可能使用的许多格式化元素。

数据集，神经网络，凝视检测，文本标签^†^†版权：acmcopyright^†^†期刊年份：2018^†^†doi：XXXXXXX.XXXXXXX^†^†会议：请确保从您的版权确认邮件中输入正确的会议标题；2018年6月3日至5日；纽约州伍德斯托克^†^†价格：15.00^†^†isbn：978-1-4503-XXXX-X/18/06^†^†ccs：计算机系统组织 嵌入式系统^†^†ccs：计算机系统组织 冗余^†^†ccs：计算机系统组织 机器人技术^†^†ccs：网络 网络可靠性{translatedabstract}

法语：一篇清晰且文档齐全的LATEX文档，作为一篇格式化文章展示，供ACM在会议论文集或期刊上发表。基于“acmart”文档类，本文介绍并解释了许多常见的变体，以及作者在准备工作文档时可能使用的许多格式化元素。

{translatedabstract}

德语 一份清晰且文档齐全的LATEX文档作为一篇文章，已按ACM会议论文集或期刊出版格式进行排版。基于“acmart”文档类，本文章也介绍并解释了许多常见的变体，以及作者在准备其工作文档时可以使用的各种格式元素。

{translatedabstract}

西班牙语 一份清晰且文档齐全的LATEX文档作为一篇文章，已按ACM会议论文集或期刊出版格式进行排版。基于“acmart”文档类，本文章也介绍并解释了许多常见的变体，以及作者在准备其工作文档时可以使用的各种格式元素。

\translatedkeywords

法语 数据集、神经网络、凝视检测、文本标注 \translatedkeywordsgerman数据集、神经网络、凝视检测、文本标注 \translatedkeywordsspanish数据集、神经网络、凝视检测、文本标注

图1. 西雅图海鹰队在2010年春季训练中的表现。

![参见标题](img/b35ee202f60decd4b15eef22314259e0.png)

从三垒席位享受棒球比赛。铃木一朗准备打击。

图1. 西雅图海鹰队在2010年春季训练中的表现。

## 1. 引言

ACM的统一文章模板于2017年推出，提供了一种一致的LATEX样式，可供ACM出版物使用，并包含了未来数字图书馆项目所需的可访问性和元数据提取功能。多个ACM和SIG特定的LATEX模板已经被检查，并将它们的独特功能整合到这个全新的模板中。

如果你是第一次使用ACM出版，这份文档是准备你的工作以供出版的宝贵指南。如果你之前已经在ACM上发表过文章，本文件将为你提供有关文章模板的最新变化的见解和指导。

“`acmart`”文档类可用于为任何ACM出版物——会议或期刊——准备文章，并且适用于出版的任何阶段，从审稿到最终的“定稿”，到作者自己的版本，几乎不需要对源文件做任何修改。

## 2. 模板概述

如引言中所述，“`acmart`”文档类可用于准备许多不同种类的文档——例如一篇全长技术论文的双盲初稿、一篇两页的SIGGRAPH新兴技术摘要、一篇“定稿”的期刊文章、一篇SIGCHI扩展摘要等——只需选择适当的模板样式和模板参数。

本文档将解释文档类的主要功能。如需进一步了解，LATEX用户指南可通过[https://www.acm.org/publications/proceedings-template](https://www.acm.org/publications/proceedings-template)获取。

### 2.1\. 模板样式

“`acmart`”文档类的主要参数是模板样式，表示出版物类型或SIG发布该工作的类型。此参数包含在方括号中，是`documentclass`命令的一部分：

```
  \documentclass[STYLE]{acmart}

```

期刊使用三种模板样式中的一种。除了三个ACM期刊之外，所有期刊都使用`acmsmall`模板样式：

+   •

    acmsmall：默认的期刊模板样式。

+   •

    acmlarge：由JOCCH和TAP使用。

+   •

    acmtog：由TOG使用。

大多数会议论文集文档将使用`acmconf`模板样式。

+   •

    acmconf：默认的会议论文集模板样式。

+   •

    sigchi：用于SIGCHI会议文章。

+   •

    sigchi-a：用于SIGCHI“扩展摘要”文章。

+   •

    sigplan：用于SIGPLAN会议文章。

### 2.2\. 模板参数

除了指定要用于格式化工作的模板样式外，还有一些模板参数会修改应用模板样式的某些部分。这些参数的完整列表可以在LATEX用户手册中找到。

常用的参数或参数组合包括：

+   •

    anonymous,review：适用于“双盲”会议投稿。使工作匿名并包含行号。使用该命令可以在每一页的工作中打印提交的唯一ID。

+   •

    authorversion：生成适合作者发布的版本。

+   •

    screen：生成彩色超链接。

本文档使用以下字符串作为源文件中的第一个命令：

```
\documentclass[sigconf, language=french,
language=german, language=spanish, language=english]{acmart}

```

## 3\. 修改

修改模板 — 包括但不限于：调整边距、字体大小、行距、段落和列表定义，以及使用`\vspace`命令手动调整工作元素之间的垂直间距 — 是不允许的。

如果发现修改，您的文档将返回给您进行修订。

## 4\. 字体

“`acmart`”文档类要求使用“Libertine”字体系列。你的TEX安装应包括此字体包。请不要替换其他字体。应避免使用“`lmodern`”和“`ltimes`”包，因为它们会覆盖内建字体系列。

## 5\. 标题信息

你的工作标题应适当使用大写字母 - [https://capitalizemytitle.com/](https://capitalizemytitle.com/) 提供了有用的标题大写规则。使用`title`命令定义你的工作标题。如果你的工作有副标题，请使用`subtitle`命令定义。不要在标题中插入换行符。

如果你的标题较长，必须定义一个短版本，用于页面标题，以避免文本重叠。`title`命令具有“短标题”参数：

```
  \title[short title]{full title}

```

## 6\. 作者和所属单位

每位作者必须单独定义，以确保元数据的准确识别。作为例外，多个作者可以共享一个单位。作者的姓名不应缩写；尽可能使用完整的名字。尽量包括作者的电子邮件地址。

将作者的姓名或电子邮件地址分组，或者提供“电子邮件别名”，如下所示，是不可接受的：

```
  \author{Brooke Aster, David Mehldau}
  \email{dave,judy,steve@university.edu}
  \email{firstname.lastname@phillips.org}

```

`authornote` 和 `authornotemark` 命令允许一个注释应用于多个作者 —— 例如，如果文章的前两位作者对工作贡献相等。

如果您的作者名单很长，必须定义一个简短的作者名单，用于页面页眉中，以避免文本重叠。以下命令应放在最后一个 `\author{}` 定义之后：

```
  \renewcommand{\shortauthors}{McCartney, et al.}

```

忽略此命令将强制使用所有作者姓名的串联列表，可能导致页眉中的文本重叠。

文章模板的文档，详见 [https://www.acm.org/publications/proceedings-template](https://www.acm.org/publications/proceedings-template)，对这些命令及其有效使用方法进行了完整的说明。

请注意，期刊文章必须包含作者地址。

## 7\. 权利信息

所有由 ACM 发布的作品的作者都需要填写权利表单。根据作品的种类和作者选择的权利管理方式，可能是版权转让、许可、授权或 OA（开放获取）协议。

无论选择何种权利管理方式，作者在提交完成的权利表单后，将收到该表单的副本。此表单包含必须复制到源文档中的 LATEX 命令。编译文档源文件时，这些命令及其参数会将格式化的文本添加到最终文档的多个区域：

+   •

    第一页上的“ACM 引用格式”文本。

+   •

    第一页上的“权利管理”文本。

+   •

    页面页眉中的会议相关信息。

权利信息对于每一项作品都是独特的；如果你正在为某个活动准备多篇作品，请确保为每篇作品使用正确的命令集。

对于所有超过一页的文章，都需要使用 ACM 引用格式文本，而一页文章（摘要）则是可选的。

## 8\. CCS 概念和用户定义的关键词

“acmart”文档类的两个元素为您提供了强大的分类工具，帮助读者在网上搜索中找到您的作品。

ACM 计算分类系统 — [https://www.acm.org/publications/class-2012](https://www.acm.org/publications/class-2012) — 是一套描述计算学科的分类器和概念。作者可以通过 [https://dl.acm.org/ccs/ccs.cfm](https://dl.acm.org/ccs/ccs.cfm) 选择该分类系统中的条目，并生成 LATEX 源文件中需要包含的命令。

用户定义的关键字是作者选择的单词和短语的逗号分隔列表，提供了一种更灵活的方式来描述所展示的研究内容。

对于所有长度超过两页的文章，必须使用CCS概念和用户定义的关键字，而对于一页或两页的文章（或摘要），这些是可选的。

## 9\. 分节命令

你的工作应使用标准的LATEX分节命令：`section`、`subsection`、`subsubsection` 和 `paragraph`。它们应进行编号；不要删除命令中的编号。

不允许通过将段落的第一个词或词组设置为粗体或斜体来模拟章节命令。

## 10\. 表格

“`acmart`”文档类包含了“`booktabs`”包 — [https://ctan.org/pkg/booktabs](https://ctan.org/pkg/booktabs) — 用于制作高质量的表格。

表格标题应放在表格上方。

由于表格不能跨页显示，因此表格的最佳位置通常是在离其首次引用最近的页面顶部。为了确保表格的正确“浮动”位置，可以使用环境 `table` 来封装表格内容和表格标题。表格本身的内容必须放在 `tabular` 环境中，以便正确对齐行和列，并设置所需的水平和垂直线条。同样，关于表格内容的详细说明，请参见《LATEX 用户指南》。

紧接着此句的部分是表格[1](#S10.T1 "Table 1 ‣ 10\. Tables ‣ The Name of the Title is Hope")在输入文件中的位置；比较这里表格的位置与文档打印输出中的表格位置。

表格 1\. 特殊字符频率

| 非英语或数学 | 频率 | 备注 |
| --- | --- | --- |
| Ø | 1 in 1,000 | 用于瑞典名字 |
| $\pi$ | 1 in 5 | 数学中常见 |
| $ | 4 in 5 | 商业中使用 |
| $\Psi^{2}_{1}$ | 1 in 40,000 | 无法解释的用法 |

要设置一个更宽的表格，占据页面有效区域的整个宽度，可以使用环境 `table*` 来封装表格内容和表格标题。与单列表格类似，这种宽表格将根据需要“浮动”到更合适的位置。紧接着此句的部分是表格[2](#S10.T2 "Table 2 ‣ 10\. Tables ‣ The Name of the Title is Hope")在输入文件中的位置；再次强调，比较这里表格的位置与文档打印输出中的表格位置是很有启发性的。

表格 2\. 一些典型的命令

| 命令 | 数字 | 备注 |
| --- | --- | --- |
| \author | 100 | 作者 |
| \table | 300 | 用于表格 |
| \table* | 400 | 用于更宽的表格 |

始终使用 `midrule` 来分隔表格的表头行和数据行，并仅用于此目的。这样可以帮助辅助技术识别表格标题，并帮助用户更轻松地浏览表格。

## 11\. 数学公式

你可能想要以三种不同的样式展示数学公式：内联、编号或非编号显示。接下来的章节将讨论这三种样式。

### 11.1\. 内联（正文）公式

出现在正文中的公式称为内联公式或正文公式。它是通过数学环境产生的，可以通过通常的 \begin …\end 结构或简短形式$ …$来调用。你可以使用LATEX（Lamport:LaTeX）中任何符号和结构，从$\alpha$到$\omega$；本节将仅展示一些内联公式的示例。请注意，这个公式：$\lim_{n\rightarrow\infty}x=0$，在内联数学样式下看起来与在显示样式下稍有不同。（见下一节）。

### 11.2\. 显示公式

一个编号的显示公式——它通过垂直空间与正文分开，并水平居中——是通过equation环境生成的。一个未编号的显示公式是通过displaymath环境生成的。

同样，在这两种环境中，你可以使用LATEX中提供的任何符号和结构；本节将只给出几个显示公式的示例。首先，考虑上面作为内联公式展示的公式：

| (1) |  | $\lim_{n\rightarrow\infty}x=0$ |  |
| --- | --- | --- | --- |

请注意，它在displaymath环境中的格式有所不同。现在，我们将输入一个未编号的公式：

|  | $\sum_{i=0}^{\infty}x+1$ |  |
| --- | --- | --- |

并跟随另一个编号的公式：

| (2) |  | $\sum_{i=0}^{\infty}x_{i}=\int_{0}^{\pi+2}f$ |  |
| --- | --- | --- | --- |

只是为了演示LATEX如何处理编号。

## 12\. 图像

应使用“`figure`”环境来表示图像。一个或多个图像可以放在一个figure环境中。如果图像包含第三方材料，必须明确标识，如下面的示例所示。

图2\. 1907年富兰克林Model D跑车。由Harris & Ewing, Inc.拍摄，[公有领域]，通过Wikimedia Commons。（[https://goo.gl/VLCRBB](https://goo.gl/VLCRBB)）。

![参见图注](img/665b97e4ed16a7266c78b87532dd7269.png)

一位穿白色连衣裙的女人和一个穿白色连衣裙的女孩坐在一辆敞篷车里。

图2\. 1907年富兰克林Model D跑车。由Harris & Ewing, Inc.拍摄，[公有领域]，通过Wikimedia Commons。（[https://goo.gl/VLCRBB](https://goo.gl/VLCRBB)）。

你的图像应包含一个图注，向读者描述图像。

图注应放置在图片下方。

每张图像都应该有图注，除非它纯粹是装饰性的。这些图注向无法看到图像的人传达图像中的内容。它们还被搜索引擎爬虫用于索引图像，并且在图像无法加载时也会显示。

图形描述必须是未格式化的纯文本，长度不超过2000个字符（包括空格）。图形描述不应重复图形标题——其目的是捕捉图例或论文正文中未提供的重要信息。对于传达重要且复杂新信息的图形，简短的文字描述可能不足以说明问题。更复杂的替代描述可以放在附录中，并在简短的图形描述中引用。例如，提供一张数据表，展示柱状图中的信息，或一个结构化的列表，表示图表。有关如何编写图形描述及其重要性的更多信息，请参阅[https://www.acm.org/publications/taps/describing-figures/](https://www.acm.org/publications/taps/describing-figures/)。

### 12.1\. “预告图”

“预告图”是放置在所有作者和单位信息之后，文章正文之前，横跨整页的图像或一组图像。如果您希望在文章中包含这样的图形，请将命令放在`\maketitle`命令之前：

```
  \begin{teaserfigure}
    \includegraphics[width=\textwidth]{sampleteaser}
    \caption{figure caption}
    \Description{figure description}
  \end{teaserfigure}

```

## 13\. 引用与参考文献

强烈建议使用BibTEX来准备和格式化参考文献。作者的姓名应完整——使用全名（如“Donald E. Knuth”），而不是缩写（如“D. E. Knuth”）——并应包含参考文献的关键识别特征：标题、年份、卷号、期号、页码、文章DOI等。

参考文献包含在您的源文件中，通过以下两个命令，这两个命令应放在`\end{document}`命令之前：

```
  \bibliographystyle{ACM-Reference-Format}
  \bibliography{bibfile}

```

其中，“`bibfile`”是BibTEX文件的名称，不包括`.bib`后缀。

引用和参考文献默认按编号排序。少数ACM出版物的引用和参考文献采用“作者-年份”格式；对于这些例外情况，请在LATEX源文件的前导部分（即“`\begin{document}`”命令之前）包含此命令：

```
  \citestyle{acmauthoryear}

```

一些示例。一个分页的期刊文章（Abril07），一个编号的期刊文章（Cohen07），对整期的引用（JCohen96），一本专著（整本书）（Kosiur01），一本系列中的专著/整本书（参见规格文档中的2a）（Harel79），一本可分割的书籍，如选集或合集（Editor00），接着是相同的示例，但只有在给定卷号时才输出系列（Editor00a）（因此Editor00a的系列不应出现，因为它没有卷号），一本可分割书籍中的章节（Spector90），一本系列中可分割书籍的章节（Douglass98），一部多卷作品作为书籍（Knuth97），几篇会议论文（例如会议、研讨会、工作坊的论文）（分页的会议论文）（Andler79；Hagerup1993），一篇包含所有可能元素的会议论文（Smith10），一个编号的会议论文示例（VanGundy07），一篇非正式出版的作品（Harel78），几篇预印本（Bornmann2019；AnzarootPBM14），一篇博士论文（Clarkson85），一篇硕士论文（anisi03），一篇在线文档/全球信息网资源（Thornburg01；Ablamowicz07；Poker06），一款视频游戏（案例1）（Obama08）和（案例2）（Novak03）和（Lee05）以及（案例3）一项专利（JoeScientist001），已接受出版的工作（rous08），'YYYYb'—繁荣作者的测试（SaeediMEJ10）和（SaeediJETC10）。其他引用可能包含'重复'的DOI和网址（一些SIAM文章）（Kirschmer:2010:AEI:1958016.1958018）。Boris / Barbara Beeton：作为书籍的多卷作品（MR781536）和（MR781537）。几条带有DOI的引用：（2004:ITE:1009386.1010128；Kirschmer:2010:AEI:1958016.1958018）。在线引用：（TUGInstmem；Thornburg01；CTANacmart）。文献记录：（R）和（UMassCitations）。

## 14\. 致谢

资金来源和其他支持的识别，以及对在研究和工作准备过程中提供帮助的个人和团体的感谢，应该包含在致谢部分，该部分应放在文献引用部分之前。

该部分具有特殊的环境：

```
  \begin{acks}
  ...
  \end{acks}

```

这样，在文章元数据提取阶段，相关信息可以更容易地收集，并确保节标题的拼写一致性。

作者不应将此部分准备为编号或未编号的`\section`；请使用“`acks`”环境。

## 15\. 附录

如果你的作品需要附录，请在源文档的“`\end{document}`”命令之前添加附录。

使用“`appendix`”命令开始附录：

```
  \appendix

```

并且请注意，在附录中，章节是按字母标记的，而不是按数字编号的。本文有两个附录，展示了章节和子章节的标识方法。

## 16\. 多语言论文

论文可以用除英语以外的其他语言书写，或包括用不同语言写的标题、副标题、关键词和摘要（通常，一篇非英语论文应包括英文标题和英文摘要）。使用`language=...`来标注论文中使用的每种语言。最后指定的语言为论文的主要语言。例如，一篇法语论文如果包含英文和德语的标题和摘要，可以以如下命令开始：

```
\documentclass[sigconf, language=english, language=german,
               language=french]{acmart}

```

文章的标题、副标题、关键词和摘要将以论文的主要语言进行排版。命令`\translatedXXX`和`XXX`（用于开始标题、副标题和关键词）可以用来设置其他语言中的这些元素。环境`translatedabstract`用于设置摘要的翻译。这些命令和环境有一个强制性的第一个参数：第二个参数的语言。关于这些命令的使用示例，请参见`sample-sigconf-i13n.tex`文件。

## 17\. SIGCHI扩展摘要

“`sigchi-a`”模板样式（仅在LATEX中可用，不支持Word）会生成横向格式的文章，具有宽左边距。该模板样式提供三种环境，可以将格式化输出放置在边距处：

sidebar::

将格式化文本放置在边距处。

marginfigure::

将图形放置在边距处。

margintable::

将表格放置在边距处。

###### 致谢。

致罗伯特，感谢你提供贝果并解释了CMYK与色彩空间。

## 附录A 研究方法

### A.1\. 第一部分

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Morbi malesuada, quam in pulvinar varius, metus nunc fermentum urna, id sollicitudin purus odio sit amet enim. Aliquam ullamcorper eu ipsum vel mollis. Curabitur quis dictum nisl. Phasellus vel semper risus, et lacinia dolor. Integer ultricies commodo sem nec semper.

### A.2\. 第二部分

Etiam commodo feugiat nisl pulvinar pellentesque. Etiam auctor sodales ligula, non varius nibh pulvinar semper. Suspendisse nec lectus non ipsum convallis congue hendrerit vitae sapien. Donec at laoreet eros. Vivamus non purus placerat, scelerisque diam eu, cursus ante. Etiam aliquam tortor auctor efficitur mattis.

## 附录B 在线资源

Nam id fermentum dui. Suspendisse sagittis tortor a nulla mollis, in pulvinar ex pretium. Sed interdum orci quis metus euismod, et sagittis enim maximus. Vestibulum gravida massa ut felis suscipit congue. Quisque mattis elit a risus ultrices commodo venenatis eget dui. Etiam sagittis eleifend elementum.

Nam interdum magna at lectus dignissim, ac dignissim lorem rhoncus. Maecenas eu arcu ac neque placerat aliquam. Nunc pulvinar massa et mattis lacinia.
