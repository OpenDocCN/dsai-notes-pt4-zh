<!--yml

类别：未分类

日期：2025-01-11 11:55:00

-->

# 下一代钓鱼攻击：LLM代理如何赋能网络攻击者 ††感谢：§ § \mathsection § J. Chen 感谢 Fordham 研究办公室通过 Fordham AI Research (FAIR) 资助的支持。

> 来源：[https://arxiv.org/html/2411.13874/](https://arxiv.org/html/2411.13874/)

Khalifa Afane^*, Wenqi Wei^*, Ying Mao^*, Junaid Farooq^†, 和 Juntao Chen^(*$\mathsection$)

^*美国福坦莫大学计算机与信息科学系，纽约，纽约州，美国

^†美国密歇根大学迪尔伯恩分校电气与计算机工程系，迪尔伯恩，密歇根州，美国

{mafane, wenqiwei, ymao41, jchen504}@fordham.edu, mjfarooq@umich.edu

###### 摘要

随着大型语言模型（LLM）的崛起，电子邮件钓鱼的威胁变得越来越复杂。攻击者利用 LLM 创建更加具有说服力和规避性的钓鱼邮件，因此评估当前钓鱼防御系统的抗压能力显得尤为重要。本研究对传统的钓鱼检测器进行了全面评估，包括 Gmail 垃圾邮件过滤器、Apache SpamAssassin 和 Proofpoint，以及像 SVM、逻辑回归和朴素贝叶斯等机器学习模型，旨在识别传统和通过 LLM 改写的钓鱼邮件。我们还探讨了 LLM 作为钓鱼检测工具的新兴角色，这一方法已经被 NTT Security Holdings 和 JPMorgan Chase 等公司采纳。我们的研究结果揭示了所有检测器在识别改写后的钓鱼邮件时准确率显著下降，突显了当前钓鱼防御的关键弱点。随着威胁形势的发展，我们的研究结果强调了对 LLM 生成内容实施更强的安全控制和监管，以防止其在创造高级钓鱼攻击中被滥用。本研究通过利用 LLM 生成多样化的钓鱼变体进行数据增强，借助 LLM 提升钓鱼检测能力，为开发更有效的网络威胁情报（CTI）做出了贡献，并为构建更强大、适应性更强的威胁检测系统铺平了道路。

###### 关键词：

大型语言模型，网络安全，电子邮件钓鱼检测，语义规避。

## I 引言

钓鱼邮件依然是网络安全领域普遍且持久的威胁，常常利用人类心理学诱使收件人泄露敏感信息[[1](https://arxiv.org/html/2411.13874v1#bib.bib1)], [[2](https://arxiv.org/html/2411.13874v1#bib.bib2)], [[3](https://arxiv.org/html/2411.13874v1#bib.bib3)]。这一问题直接导致大型组织在2023年平均损失达1500万美元[[4](https://arxiv.org/html/2411.13874v1#bib.bib4)]。传统的钓鱼检测器通过识别特定的语言线索，已实现了高精度和召回率，能够有效地缓解许多钓鱼攻击。然而，大型语言模型（LLM）的迅速发展为这一领域带来了新的复杂性。这些先进的模型已经在网络安全基准测试中超越了领域专家，例如CyberMetric[[5](https://arxiv.org/html/2411.13874v1#bib.bib5)]，使得它们能够创造出更为微妙和具有说服力的钓鱼邮件，从而使得传统的钓鱼数据集和检测方法变得越来越低效。因此，LLM生成的钓鱼邮件构成了一个亟需解决的新威胁。随着提示工程技术的进步，如零-shot和少-shot提示，这一挑战进一步加剧，这些技术能够在无需训练数据的情况下有效应对新任务，使得攻击者能够以最小的努力生成高度针对性且语境准确的钓鱼邮件[[6](https://arxiv.org/html/2411.13874v1#bib.bib6)], [[7](https://arxiv.org/html/2411.13874v1#bib.bib7)]。

在大型语言模型（LLM）兴起之前，已有大量研究改进了钓鱼攻击检测方法。例如，Fette等人[[8](https://arxiv.org/html/2411.13874v1#bib.bib8)]提出了一种专注于检测欺诈特征的机器学习方法，成功识别出99.5%以上的钓鱼邮件，并且误报率非常低。Ma等人[[9](https://arxiv.org/html/2411.13874v1#bib.bib9)]在此基础上，通过使用混合特征，将基于内容的关键词和短语与表单、错误匹配的URL等属性相结合，构建了更为强大的分类器。同样，Verma等人[[10](https://arxiv.org/html/2411.13874v1#bib.bib10)]探索了自然语言处理技术，强调了语义分析在识别钓鱼行为中的有效性。其他技术也证明了非常有效，达到了近乎完美的准确率[[11](https://arxiv.org/html/2411.13874v1#bib.bib11)], [[12](https://arxiv.org/html/2411.13874v1#bib.bib12)]。

![参考标题](img/b1a6671771391162e6bd8f9e477d5902.png)

图1：评估方法学工作流，突出显示传统钓鱼邮件与LLM重写邮件在检测效果上的平均差异。

LLM的双重用途特性使它们在此背景下尤为重要，因为它们在生成有益和恶意内容方面都非常有效。Wu等人[[13](https://arxiv.org/html/2411.13874v1#bib.bib13)]和Yao等人[[14](https://arxiv.org/html/2411.13874v1#bib.bib14)]强调了LLM带来的安全隐患，指出它们能够制作复杂的钓鱼邮件，这些邮件能躲避传统的检测方法。此外，研究还表明，LLM能够大规模生成个性化的钓鱼信息，既真实又具成本效益[[15](https://arxiv.org/html/2411.13874v1#bib.bib15)]。像Llama[[16](https://arxiv.org/html/2411.13874v1#bib.bib16)]、Gemini[[17](https://arxiv.org/html/2411.13874v1#bib.bib17)]、Claude[[18](https://arxiv.org/html/2411.13874v1#bib.bib18)]和GPT[[19](https://arxiv.org/html/2411.13874v1#bib.bib19)]等LLM展示了生成类人文本的卓越能力，越来越多地应用于钓鱼检测等任务[[20](https://arxiv.org/html/2411.13874v1#bib.bib20)]，[[21](https://arxiv.org/html/2411.13874v1#bib.bib21)]。本研究在前人工作的基础上，强调了这些模型的双重特性：它们不仅能够制作复杂的钓鱼邮件，还能够通过增强数据训练来提升钓鱼检测器，从而增强网络威胁情报（CTI）。

论文的其余部分组织如下：第二节回顾了与钓鱼检测相关的工作以及由LLM生成的钓鱼邮件所带来的挑战。第三节介绍了方法论，包括所使用的数据集、实验设置和改写技术。第四节讨论了实验结果，重点介绍了钓鱼检测器和机器学习模型的表现。第五节对研究结果进行了讨论，概述了局限性，并提出了未来的研究方向。最后，第六节总结了论文的关键见解和贡献。

## II 相关工作

钓鱼邮件检测传统上依赖于非LLM基础的方法，如谷歌内置的Gmail垃圾邮件过滤器，其他知名工具如SpamAssassin [[22](https://arxiv.org/html/2411.13874v1#bib.bib22)]和Proofpoint也证明是高度有效的检测工具[[23](https://arxiv.org/html/2411.13874v1#bib.bib23)]。然而，基于LLM的钓鱼检测的整合是一个新兴现象[[25](https://arxiv.org/html/2411.13874v1#bib.bib25)]，例如JPMorgan Chase公司在这一领域处于领先地位[[24](https://arxiv.org/html/2411.13874v1#bib.bib24)]，NTT Holdings则推出了他们的框架ChatSpamDetector[[21](https://arxiv.org/html/2411.13874v1#bib.bib21)]。

然而，随着钓鱼检测技术的提升，大型语言模型（LLMs）正被用来提升钓鱼攻击的复杂性，带来了新的挑战。Hazell [[15](https://arxiv.org/html/2411.13874v1#bib.bib15)] 例如，探索了LLMs在通过生成个性化电子邮件来扩大定向钓鱼攻击规模的潜力，使用了GPT-3.5和GPT-4模型，针对600多位英国议会议员进行邮件生成。研究结果表明，这些模型能够创建逼真且具有成本效益的定向钓鱼电子邮件，每封邮件的成本仅为几分之一美分。类似地，Heiding等人[[20](https://arxiv.org/html/2411.13874v1#bib.bib20)]调查了LLMs与V-Triad结合在电子邮件钓鱼生成中的应用，发现像GPT-4这样的模型与人类知识相结合时，能够生成高度可信的钓鱼邮件，这些邮件能够避开传统的检测方法。Kang等人[[26](https://arxiv.org/html/2411.13874v1#bib.bib26)]进一步探讨了LLMs在各种恶意任务中的应用，得出结论：这些模型能够生成精心设计的内容，可以被用来进行钓鱼攻击和其他诈骗活动。

与以往的研究相比，我们的工作全面评估了最先进的钓鱼检测器的检测能力，包括谷歌的Gmail垃圾邮件过滤器、SpamAssassin和Proofpoint，以及大型语言模型（LLMs），在原始和LLM重述的钓鱼邮件上进行评估，如图[1](https://arxiv.org/html/2411.13874v1#S1.F1 "图 1 ‣ I 引言 ‣ 下一代钓鱼：LLM代理如何增强网络攻击者 § J. Chen 感谢Fordham研究办公室通过Fordham AI研究（FAIR）奖助支持")所示。我们的方法采用了多种机器学习技术，展示了我们的评估方法论的工作流程。具体来说，图[1](https://arxiv.org/html/2411.13874v1#S1.F1 "图 1 ‣ I 引言 ‣ 下一代钓鱼：LLM代理如何增强网络攻击者 § J. Chen 感谢Fordham研究办公室通过Fordham AI研究（FAIR）奖助支持")突出显示了传统钓鱼邮件和LLM重述邮件之间检测效果的差异，展示了当前钓鱼检测器在处理AI生成威胁时面临的挑战。Roy等人[[27](https://arxiv.org/html/2411.13874v1#bib.bib27)]从提示工程的角度解决了LLM生成钓鱼攻击的问题，探讨了恶意提示的制作和利用LLM设计这些提示。尽管他们的方法可以缓解LLM生成钓鱼攻击的风险，但我们认为，更有效的方法是通过更好的训练数据训练模型来检测这些威胁。我们的方法通过专注于提高钓鱼检测器的检测能力，而不仅仅依赖于提示工程和检测，来补充Roy等人的工作。我们专注于Nazario和Nigerian Fraud数据集[[28](https://arxiv.org/html/2411.13874v1#bib.bib28)]，并利用GPT-4o和Llama 3重述钓鱼邮件。我们总结了本文的贡献如下：

1.  1.

    我们对传统钓鱼邮件和LLM重述的钓鱼邮件进行了全面比较。此比较评估了各种钓鱼检测工具的性能，如谷歌的Gmail垃圾邮件过滤器、SpamAssassin、Proofpoint和其他机器学习模型，以及LLM如Llama、Gemini、Claude和GPT作为钓鱼检测器的表现。

1.  2.

    我们介绍了一种有效的数据增强框架，以创建更能反映现代威胁的钓鱼邮件数据集。使用GPT-4和Llama 3，我们对Nazario数据集中的钓鱼邮件进行了重述和增强，创建了LLM-Nazario数据集，该数据集包含了原始邮件、重述邮件以及新生成的钓鱼攻击。这种方法的有效性通过在新数据集上训练三个机器学习模型来展示，从而评估它们在检测像LLM重述钓鱼邮件这样的高级威胁时的改进能力。

## III 方法

### III-A 数据收集与构建

我们为实验使用了两个主要的数据集：Nazario数据集和尼日利亚欺诈钓鱼邮件数据集[[28](https://arxiv.org/html/2411.13874v1#bib.bib28)]。Nazario数据集在清洗后原本包含2904个实例，但在我们的实验中，我们从中均匀抽取了1200封邮件，分为两个类别：600封合法邮件和600封钓鱼邮件。合法邮件最初来自无害的在线讨论和新闻通讯，而钓鱼邮件则遵循传统格式，包括欺诈性请求和虚假的链接。这些邮件长度各异，范围从10到350个字符。

尼日利亚欺诈数据集作为我们实验的第二个数据来源。从这些数据集中，我们选择了800封邮件，合法邮件和钓鱼邮件各占一半。这些钓鱼邮件平均比Nazario数据集中的邮件更长，长度范围从10到650个字符，主要涉及财务诈骗，包括大额资金的提供或个人信息的索取。

在两个数据集中，实验中保留的特征有发件人、收件人、主题和正文。这些特征至关重要，因为它们为分类提供了宝贵的上下文，特别是对于嵌入链接（如“验证密码”或“下载文件”）的判断。下一节描述的钓鱼邮件重述对评估检测系统在应对这些修改版邮件时的表现至关重要。

### III-B 实验设置

我们的实验设置包括测试三个传统的钓鱼检测系统：Google的Gmail垃圾邮件过滤器、SpamAssassin和Proofpoint，三个机器学习模型：SVM、逻辑回归和朴素贝叶斯，以及五个著名的LLM：Llama 3、Gemini 1.5、Claude 3 Sonnet、GPT-3.5和GPT-4o。

对于Gmail垃圾邮件过滤器，我们使用了不同的电子邮件账户来模拟用户环境。钓鱼邮件和合法邮件被发送到这些账户，我们记录了每封邮件是否自动被移动到垃圾邮件文件夹或显示在用户的收件箱中。这个二元决策作为Gmail对三类邮件的检测标准：原始钓鱼邮件、零-shot重述邮件和少量-shot重述邮件。

Apache SpamAssassin使用多种基于内容和统计的技术来评估邮件是否存在潜在的钓鱼或垃圾邮件。在我们的设置中，我们跟踪了每封邮件是否根据其评估标准被标记为垃圾邮件。Proofpoint利用先进的邮件安全技术，用于对邮件进行分类，我们记录了发送给接收者的邮件以及被放入不同“隔离”文件夹中的邮件。

对于三种机器学习模型，我们应用了三种文本编码技术：词袋模型[[29](https://arxiv.org/html/2411.13874v1#bib.bib29)]、词频-逆文档频率（TF-IDF）[[30](https://arxiv.org/html/2411.13874v1#bib.bib30)]和Word2Vec[[31](https://arxiv.org/html/2411.13874v1#bib.bib31)]，这些模型在研究中使用的其他数据集子集上进行了训练。平均而言，TF-IDF编码在检测钓鱼邮件方面比词袋模型准确度高2.6%，比Word2Vec高5.4%。这些编码方法将电子邮件文本转换为适合分类的数值表示。因此，表[I](https://arxiv.org/html/2411.13874v1#S4.T1 "TABLE I ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.")和表[III](https://arxiv.org/html/2411.13874v1#S4.T3 "TABLE III ‣ IV-B Nigerian Fraud Dataset Experiments ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.")中三种机器学习模型的结果是使用TF-IDF编码获得的。在数据增强后测试三种模型在重新表述邮件上的表现时，也采用了相同的编码技术。

五种LLM在三类电子邮件上进行了测试：（1）原始钓鱼邮件，（2）使用GPT-4o和零样本提示重新表述的邮件，以及（3）使用GPT-4o和少样本提示重新表述的邮件。对于零样本提示，我们提供了一个简单的指令来重新表述邮件，同时保持相同的核心内容。对于少样本提示，我们使用了三封示例重新表述的钓鱼邮件来引导LLM。

所有实验均在受控环境下进行，LLM模型接受提示，并通过三次迭代验证结果，以减少模型中非确定性行为的影响。为了确保一致性，我们采用了多数投票法来最终确定每封电子邮件的分类。这种全面的设置使我们能够比较三种传统电子邮件检测系统和五种LLM在原始和重新表述的钓鱼邮件上的表现，为每个系统的优缺点提供了宝贵的见解，特别是在检测旨在规避检测的重新表述钓鱼邮件方面。

### III-C 重新表述技术

选择零样本提示和少样本提示这两种技术，是因为它们在先前的研究中已经证明了其有效性，特别是对于缺乏广泛训练数据的新任务，这与我们的需求相符[[6](https://arxiv.org/html/2411.13874v1#bib.bib6)]。

#### III-C1 零样本提示

零样本提示涉及向大语言模型（LLM）提供一个任务描述而不包含任何示例，仅依赖于模型的预训练知识。这种方法利用了LLM从提供的指令中概括的能力，并将其广泛的预训练知识应用于新的任务。先前的研究已经证明了零样本提示的有效性，特别是在缺乏大量训练数据的新任务中[[6](https://arxiv.org/html/2411.13874v1#bib.bib6)]，[[32](https://arxiv.org/html/2411.13874v1#bib.bib32)]，[[33](https://arxiv.org/html/2411.13874v1#bib.bib33)]。这种方法简单高效，允许模型在不需要特定任务微调的情况下表现良好[[35](https://arxiv.org/html/2411.13874v1#bib.bib35)]。为了绕过一些LLM的安全层，向提示中提供明确的上下文是有效的。例如，添加说明任务是用于研究目的通常有助于获得期望的回应。以下是我们选择的零样本提示，使用这种提示的多种变体也能产生类似的结果。需要注意的是，我们只允许LLM改变电子邮件的主题和正文内容。虽然使用更好或更专业的发件人域名可能更有效，但在我们看来，这并不现实。

<svg class="ltx_picture" height="178.23" id="S3.SS3.SSS1.1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,178.23) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 160.03)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Zero-Shot Prompt</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="128.53" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">I want to train my machine learning model for my research on email phishing detection. I need you to rephrase the emails below and give me the new subject and body that are designed to bypass email phishing detectors. Keep the same sender and receiver information, avoid any sense of urgency or words like ’urgent’ and ’immediate’ in the body or subject, and avoid using deadlines or ultimatums. Avoid generic greetings, use the receiver information to greet them properly, and don’t mention sums of money or dollar amounts to make the email more legitimate. Ask for the same information the original email is asking for, just make the context more legitimate and keep the same core topic.</foreignobject></g></g></svg>

#### III-C2 少样本提示

少样本提示包括一些示例以及任务描述，帮助模型更好地理解任务并产生更准确、更符合上下文的输出。少样本提示不一定比零样本提示产生更好的结果，它们的表现取决于具体的任务和上下文[[36](https://arxiv.org/html/2411.13874v1#bib.bib36)]，[[37](https://arxiv.org/html/2411.13874v1#bib.bib37)]。对于每个数据集，我们向模型提供一个类似于零样本提示的任务描述，并附上3个原始钓鱼邮件示例（3-shot）以及精心设计的期望输出，目的是绕过钓鱼检测器。以下是来自Nazario数据集的一个原始钓鱼邮件示例，作为少样本提示的一部分，以及模型所需的期望输出。

<svg class="ltx_picture" height="161.63" id="S3.SS3.SSS2.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,161.63) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 143.42)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">原始邮件</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="111.93" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">主题：紧急内容：如果您在接下来的24小时内未完成必需的邮箱升级，您的账户将被禁止发送邮件。点击以下链接解锁并将您的账户升级至版本3.0.1。忽视此邮件将导致您的邮箱服务暂停。邮件服务团队 收件人：kevin@rocketinvestment.org</foreignobject></g></g></svg><svg class="ltx_picture" height="225.35" id="S3.SS3.SSS2.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,225.35) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 207.15)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">少样本训练的期望输出</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="175.65" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">主题：账户升级可用内容：亲爱的Kevin，您的账户有可用的升级。点击以下链接解锁并升级您的账户。升级至版本3.0.1 谢谢，邮件服务团队</foreignobject></g></g></svg>

去除紧迫感并采用更加专业、冷静的语气可以显著提高钓鱼邮件的有效性。紧迫感通常会触发检测机制，并引起收件人的怀疑。通过将语言从危言耸听转为更加中立、信息化的语气，攻击者可以构建出感觉常规的邮件，例如通知收件人有可用的更新，而非要求立即采取行动。

这种方法利用了人们对精心编写邮件的信任。专业的语言给人一种邮件来自可靠来源的印象，从而降低了被标记为可疑的可能性。同时，它还通过避免紧迫感的压力，利用人类心理，使收件人感到更加舒适、少有防备，从而增加他们与邮件互动的可能性。这一策略的微妙性至关重要。邮件不再是明显的操控行为，而是成为了一个听起来无害的请求，融入收件人日常的通信中。

### III-D 评估指标

钓鱼检测器的表现通过以下指标进行评估：

+   •

    真阳性 (TP): 正确识别的网络钓鱼邮件。

+   •

    真阴性 (TN): 正确识别的合法邮件。

+   •

    假阳性 (FP): 被错误分类为钓鱼的合法邮件。

+   •

    假阴性 (FN): 被错误分类为合法的网络钓鱼邮件。

+   •

    准确率: $(TP+TN)/(TP+TN+FP+FN)$。

+   •

    精确度: $TP/(TP+FP)$。

+   •

    召回率: $TP/(TP+FN)$，也称为检测率。

+   •

    F1分数:

    |  | $F1=2\times\frac{\text{Precision}\times\text{Recall}}{\text{Precision}+\text{Recall}}.$ |  | (1) |
    | --- | --- | --- | --- |

例如，FP率为0.2意味着20%的合法邮件被错误标记为钓鱼邮件，而FN率为0.15则意味着15%的钓鱼邮件被漏检。这些指标对于评估钓鱼检测系统的可靠性至关重要，其中假阴性率尤为重要，因为它表示未被检测到的钓鱼威胁比例，可能会导致潜在的危害。

## IV 实验结果

实验结果表明，与传统网络钓鱼邮件相比，LLM改写的钓鱼邮件在决策边界上发生了显著变化。通过分析本研究中使用的不同数据集中的200封邮件样本，我们观察到传统的网络钓鱼邮件呈现出更为清晰的决策边界。这些边界主要由某些关键词的出现所驱动，如“皇家家庭”、“紧急”和“大额付款”等，这些词汇被网络钓鱼检测器标记为高度可疑。

![参见标题说明](img/8c1ffefef599fcbb305ee6c233ff7331.png)

图 2: 传统网络钓鱼邮件与使用LLM改写的网络钓鱼邮件在分类概率方面的决策边界变化示意图。

相比之下，使用LLM生成的改写钓鱼邮件采用了更为合法的语言，如“凭证”、“会员资格”、“确认”和“账户更新”。尽管这些词汇更常与合法邮件相关联，但“登录”等词汇仍然频繁出现，在某些情境下仍然能触发网络钓鱼检测器。

这种语言的变化导致决策边界的缩小，使得分类器更难区分网络钓鱼邮件和合法邮件。图 [2](https://arxiv.org/html/2411.13874v1#S4.F2 "Figure 2 ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.") 展示了这一变化，显示了传统网络钓鱼邮件因频繁使用可疑词汇而被标记为钓鱼的概率较高。改写邮件则使用了更中性的语言，使其更难以分类，从而将决策边界推向更接近合法邮件的方向。

为了更定量地理解这些变化，我们应用了朴素贝叶斯和逻辑回归模型来评估电子邮件对特定单词的敏感性。在朴素贝叶斯模型中，电子邮件基于特定单词被分类为钓鱼邮件的概率计算公式为：

|  | $P(\text{phishing}\mid w_{i})=\frac{P(w_{i}\mid\text{phishing})P(\text{phishing% })}{P(w_{i})}$ |  |
| --- | --- | --- |

这种方法使我们能够量化特定单词（如“紧急”或“支付”）在钓鱼邮件中与合法邮件中出现的频率。因此，频繁与钓鱼尝试相关联的单词显著增加了电子邮件被分类为钓鱼邮件的概率。

类似地，逻辑回归模型根据每个单词对电子邮件整体分类的贡献提供一个概率分数。该方程为：

|  | $P(\text{phishing}\mid w_{i})=\frac{1}{1+e^{-(\mathbf{w}^{T}\mathbf{x}_{i}+b)}}$ |  |
| --- | --- | --- |

在这种情况下，权重向量$\mathbf{w}$为特定单词分配重要性，例如“凭证”或“登录”，这些单词可能同时出现在钓鱼邮件和合法邮件中，但上下文不同。偏置项$b$调整模型将电子邮件分类为钓鱼邮件或合法邮件的整体倾向。

这些模型直接影响图[2](https://arxiv.org/html/2411.13874v1#S4.F2 "Figure 2 ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.")中显示的x轴值，这些概率反映了分类对特定单词出现的敏感度。传统的钓鱼邮件，由于其单词远离决策边界，因此更容易被检测到，而重新措辞的邮件使用更多中性词汇，通过缩小决策边界来挑战检测器。

这种决策边界的变化也可以在基于大语言模型（LLM）的钓鱼检测器中观察到。如图[3](https://arxiv.org/html/2411.13874v1#S4.F3 "Figure 3 ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.")和[4](https://arxiv.org/html/2411.13874v1#S4.F4 "Figure 4 ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.")所示，Llama3模型将前两封电子邮件分类为合法邮件，剩余的邮件则被分类为钓鱼邮件，其推理主要是由特定单词的使用驱动。包含“紧急”或“支付”等词汇的钓鱼邮件更容易被检测到，而使用“账户更新”等词汇重新措辞的邮件则引入了更多的模糊性，降低了检测准确性。

![参见说明文字](img/63d300dbf3c312ae1c54d458dd00d8b5.png)

图 3: Llama 3 对 5 封原始邮件的分类结果

![参考图注](img/e160dae82ca37e668109492e05739fcf.png)

图 4: Llama 3 对 5 封重述邮件的分类结果（少-shot 提示）

表 Ⅰ: 网络钓鱼检测器在 Nazario 数据集上的表现

| 检测器 | TP | TN | FP | FN | 准确率 | 精确度 | 召回率 | F1 分数 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 原始邮件 |
| Gmail 垃圾邮件过滤器 | 573 | 589 | 27 | 11 | 96.83% | 95.50% | 98.12% | 96.79% |
| Proofpoint | 558 | 592 | 41 | 8 | 96.16% | 93.00% | 98.59% | 95.71% |
| SpamAssassin | 574 | 576 | 26 | 24 | 95.83% | 95.67% | 95.99% | 95.83% |
| 朴素贝叶斯（Naive Bayes） | 559 | 567 | 41 | 33 | 93.80% | 93.17% | 94.43% | 93.79% |
| 支持向量机（SVM） | 542 | 555 | 58 | 45 | 91.42% | 90.33% | 92.33% | 91.32% |
| 逻辑回归 | 554 | 569 | 46 | 31 | 93.58% | 92.33% | 94.70% | 93.50% |
| 零-shot 重述邮件 |
| Gmail 垃圾邮件过滤器 | 573 | 559 | 27 | 41 | 94.33% | 95.50% | 93.32% | 94.40% |
| Proofpoint | 558 | 554 | 42 | 46 | 92.67% | 93.00% | 92.38% | 92.69% |
| SpamAssassin | 574 | 545 | 26 | 55 | 93.25% | 95.67% | 91.26% | 93.41% |
| 朴素贝叶斯（Naive Bayes） | 559 | 518 | 41 | 82 | 89.75% | 93.17% | 87.21% | 90.09% |
| 支持向量机（SVM） | 542 | 533 | 58 | 67 | 89.58% | 90.33% | 89.00% | 89.66% |
| 逻辑回归 | 554 | 515 | 46 | 85 | 89.08% | 92.33% | 86.70% | 89.43% |
| 少-shot 重述邮件 |
| Gmail 垃圾邮件过滤器 | 573 | 483 | 27 | 117 | 88.00% | 95.50% | 83.04% | 88.84% |
| Proofpoint | 558 | 505 | 42 | 95 | 88.58% | 93.00% | 85.45% | 89.07% |
| SpamAssassin | 574 | 465 | 26 | 135 | 86.50% | 95.67% | 80.96% | 87.70% |
| 朴素贝叶斯（Naive Bayes） | 559 | 418 | 41 | 182 | 81.42% | 93.17% | 75.44% | 83.37% |
| 支持向量机（SVM） | 542 | 421 | 58 | 179 | 80.25% | 90.33% | 75.17% | 82.06% |
| 逻辑回归 | 554 | 460 | 46 | 140 | 84.50% | 92.33% | 79.83% | 85.63% |

表 Ⅱ: LLMs 在尼日利亚诈骗数据集上的表现

| LLM | TP | TN | FP | FN | 准确率 | 精确度 | 召回率 | F1 分数 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 原始邮件 |
| GPT-4 | 391 | 395 | 9 | 5 | 98.25% | 97.75% | 98.74% | 98.24% |
| GPT-3.5 | 386 | 384 | 14 | 16 | 96.25% | 96.50% | 96.02% | 96.26% |
| Claude 3 Sonnet | 385 | 392 | 15 | 8 | 97.12% | 96.25% | 97.96% | 97.10% |
| Llama3 | 389 | 397 | 11 | 3 | 98.25% | 97.25% | 99.23% | 98.23% |
| Gemini | 385 | 389 | 15 | 11 | 96.75% | 96.25% | 97.22% | 96.73% |
| 零-shot 重述邮件 |
| GPT-4 | 391 | 369 | 9 | 31 | 95.00% | 97.75% | 92.65% | 95.13% |
| GPT-3.5 | 386 | 347 | 14 | 53 | 91.62% | 96.50% | 87.93% | 92.01% |
| Claude 3 Sonnet | 385 | 361 | 15 | 39 | 93.25% | 96.25% | 90.80% | 93.45% |
| Llama3 | 389 | 370 | 11 | 30 | 94.88% | 97.25% | 92.84% | 94.99% |
| Gemini | 385 | 342 | 15 | 58 | 90.88% | 96.25% | 86.91% | 91.34% |
| 少-shot 重述邮件 |
| GPT-4 | 391 | 353 | 9 | 47 | 93.00% | 97.75% | 89.27% | 93.32% |
| GPT-3.5 | 386 | 322 | 14 | 78 | 88.50% | 96.50% | 83.19% | 89.35% |
| Claude 3 Sonnet | 385 | 354 | 15 | 46 | 92.38% | 96.25% | 89.33% | 92.66% |
| Llama3 | 389 | 346 | 11 | 54 | 91.88% | 97.25% | 87.81% | 92.29% |
| Gemini | 385 | 276 | 15 | 124 | 82.62% | 96.25% | 75.64% | 84.71% |

如图[3](https://arxiv.org/html/2411.13874v1#S4.F3 "图3 ‣ IV 实验结果 ‣ 下一代网络钓鱼：LLM代理如何赋能网络攻击者 § J. Chen 感谢福坦莫大学研究办公室通过福坦莫AI研究（FAIR）资助提供的支持。")所示，Llama 3模型正确识别了前两个电子邮件为合法邮件，并将后三个识别为钓鱼邮件。这表明该模型在处理原始、未经修改的电子邮件内容时的准确性。

图[4](https://arxiv.org/html/2411.13874v1#S4.F4 "图4 ‣ IV 实验结果 ‣ 下一代网络钓鱼：LLM代理如何赋能网络攻击者 § J. Chen 感谢福坦莫大学研究办公室通过福坦莫AI研究（FAIR）资助提供的支持。")展示了使用少样本提示重述邮件的检测率。在这个样本中，两封钓鱼邮件被错误地分类为合法邮件。模型给出的推理表明，它依赖于电子邮件中的特定单词和模式，这些模式很容易通过重述来绕过。

### IV-A Nazario数据集实验

来自Nazario数据集的结果，汇总在表[I](https://arxiv.org/html/2411.13874v1#S4.T1 "表I ‣ IV 实验结果 ‣ 下一代网络钓鱼：LLM代理如何赋能网络攻击者 § J. Chen 感谢福坦莫大学研究办公室通过福坦莫AI研究（FAIR）资助提供的支持。")中，突出显示了三种钓鱼检测器——Google Gmail、SpamAssassin和Proofpoint，以及三种机器学习模型——Naive Bayes、SVM和Logistic Regression的表现。实验中使用了共1200封电子邮件（600封钓鱼邮件和600封合法邮件）。

Proofpoint在原始钓鱼邮件的召回率上表现最好，但由于其敏感性，假阳性率较高。Gmail在准确性方面表现更好，假阳性较少，但在本研究中，召回率仍然是更为重要的指标。重述后，所有模型的表现显著下降，特别是在零样本重述邮件上，Naive Bayes和SVM等模型的召回率降至低至87%。少样本重述邮件进一步降低了检测率，即使是像Gmail和Proofpoint这样的先进检测器，在这些更微妙的钓鱼尝试面前也显得力不从心。

表[II](https://arxiv.org/html/2411.13874v1#S4.T2 "表II ‣ IV 实验结果 ‣ 下一代网络钓鱼：LLM代理如何赋能网络攻击者 § J. Chen 感谢福坦莫大学研究办公室通过福坦莫AI研究（FAIR）资助提供的支持。")展示了五种LLM模型——GPT-4、GPT-3.5、Claude 3 Sonnet、Llama3和Gemini——在同一数据集上的表现。它包括原始邮件、零样本重述邮件和少样本重述邮件的真阳性（TP）、真阴性（TN）、假阳性（FP）和假阴性（FN）计数，以及准确率、精确率、召回率和F1分数。

GPT-4 在所有类型的邮件中都保持了最高的准确率和召回率。Llama3 在少-shot 重写邮件中表现出了显著的改进，而 Gemini 在重写邮件上遇到了困难，尤其是在零-shot 场景下，其准确率显著下降。

### IV-B 尼日利亚欺诈数据集实验

尼日利亚欺诈数据集结合了两个具有相似内容的先前数据集 [[28](https://arxiv.org/html/2411.13874v1#bib.bib28)]。最终数据集包含 800 封邮件（400 封钓鱼邮件和 400 封合法邮件），并提出了一个独特的挑战，因为这些邮件通常较长（10–650 字符），且涉及金融诈骗和紧急语言。这一复杂性既是挑战也是优势，具体取决于钓鱼检测器。

对于原始邮件，Proofpoint 的表现优于其他检测器，召回率接近完美（98.95%），但在重写邮件上的表现有所下降。零-shot 重写使其召回率降至 93.16%，相应的准确率也有所下降，反映出重写的钓鱼邮件更为复杂，能够绕过传统的垃圾邮件启发式检测。SpamAssassin 和 Gmail 也出现了类似的下降，Gmail 在零-shot 场景中的召回率降至 88.76%。

表 III：钓鱼检测器在尼日利亚欺诈数据集上的表现

| 检测器 | TP | TN | FP | FN | 准确率 | 精确率 | 召回率 | F1 分数 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 原始邮件 |
| Gmail 垃圾邮件过滤器 | 387 | 396 | 13 | 4 | 97.88% | 96.75% | 98.98% | 97.85% |
| SpamAssassin | 378 | 396 | 22 | 4 | 96.75% | 94.50% | 98.95% | 96.68% |
| Proofpoint | 368 | 399 | 32 | 1 | 95.88% | 92.00% | 99.73% | 95.71% |
| 朴素贝叶斯 | 375 | 388 | 25 | 12 | 95.38% | 93.75% | 96.90% | 95.30% |
| 支持向量机 (SVM) | 379 | 390 | 21 | 10 | 96.12% | 94.75% | 97.43% | 96.07% |
| 逻辑回归 | 381 | 393 | 19 | 7 | 96.75% | 95.25% | 98.20% | 96.70% |
| 零-shot 重写邮件 |
| Gmail 垃圾邮件过滤器 | 387 | 351 | 13 | 49 | 92.25% | 96.75% | 88.76% | 92.58% |
| SpamAssassin | 378 | 357 | 22 | 43 | 91.88% | 94.50% | 89.79% | 92.08% |
| Proofpoint | 368 | 373 | 32 | 27 | 92.62% | 92.00% | 93.16% | 92.58% |
| 朴素贝叶斯 | 375 | 349 | 25 | 51 | 90.50% | 93.75% | 88.03% | 90.80% |
| 支持向量机 (SVM) | 379 | 343 | 21 | 57 | 90.25% | 94.75% | 86.93% | 90.67% |
| 逻辑回归 | 381 | 335 | 19 | 65 | 89.50% | 95.25% | 85.43% | 90.07% |
| 少-shot 重写邮件 |
| Gmail 垃圾邮件过滤器 | 387 | 349 | 13 | 51 | 92.00% | 96.75% | 88.36% | 92.36% |
| SpamAssassin | 378 | 340 | 22 | 60 | 89.75% | 94.50% | 86.30% | 90.21% |
| Proofpoint | 368 | 376 | 32 | 24 | 93.00% | 92.00% | 93.88% | 92.93% |
| 朴素贝叶斯 | 375 | 336 | 25 | 64 | 88.88% | 93.75% | 85.42% | 89.39% |
| 支持向量机 (SVM) | 379 | 318 | 21 | 82 | 87.12% | 94.75% | 82.21% | 88.04% |
| 逻辑回归 | 381 | 327 | 19 | 73 | 88.50% | 95.25% | 83.92% | 89.23% |

表 IV：LLMs 在尼日利亚欺诈数据集上的表现

| LLM | TP | TN | FP | FN | 准确率 | 精确率 | 召回率 | F1 分数 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 原始邮件 |
| GPT-4 | 394 | 399 | 6 | 1 | 99.12% | 98.50% | 99.75% | 99.12% |
| GPT-3.5 | 379 | 384 | 21 | 16 | 95.38% | 94.75% | 95.95% | 95.35% |
| Claude 3 Sonnet | 394 | 390 | 6 | 10 | 98.00% | 98.50% | 97.52% | 98.01% |
| Llama3 | 392 | 396 | 8 | 4 | 98.50% | 98.00% | 98.99% | 98.49% |
| Gemini | 382 | 386 | 18 | 14 | 96.00% | 95.50% | 96.46% | 95.98% |
| Zero Shot 改写电子邮件 |
| GPT-4 | 394 | 372 | 6 | 28 | 95.75% | 98.50% | 93.36% | 95.86% |
| GPT-3.5 | 379 | 354 | 21 | 46 | 91.62% | 94.75% | 89.18% | 91.88% |
| Claude 3 Sonnet | 394 | 366 | 6 | 34 | 95.00% | 98.50% | 92.06% | 95.17% |
| Llama3 | 392 | 381 | 8 | 19 | 96.62% | 98.00% | 95.38% | 96.67% |
| Gemini | 382 | 324 | 18 | 76 | 88.25% | 95.50% | 83.41% | 89.04% |
| 少量样本改写电子邮件 |
| GPT-4 | 394 | 365 | 6 | 35 | 94.88% | 98.50% | 91.84% | 95.05% |
| GPT-3.5 | 379 | 337 | 21 | 63 | 89.50% | 94.75% | 85.75% | 90.02% |
| Claude 3 Sonnet | 394 | 363 | 6 | 37 | 94.62% | 98.50% | 91.42% | 94.83% |
| Llama3 | 392 | 367 | 8 | 33 | 94.88% | 98.00% | 92.24% | 95.03% |
| Gemini | 382 | 311 | 18 | 89 | 86.62% | 95.50% | 81.10% | 87.72% |

类似朴素贝叶斯（Naive Bayes）和支持向量机（SVM）等机器学习模型，在使用原始数据集中的1500封电子邮件进行训练后，在处理改写的钓鱼电子邮件时，表现出显著的性能下降。特别是，朴素贝叶斯的准确率下降到88%，在处理少量改写电子邮件时，其召回率下降至85%。这表明，当处理能够绕过传统基于关键词检测的复杂钓鱼邮件时，这些模型存在局限性。结果表明，对于像尼日利亚诈骗数据集这样的数据集，传统的钓鱼检测器在处理原始邮件时可能仍然保持较强的性能，但在这些邮件经过高级技术改写后，它们的有效性显著下降。

在使用少量示例提示进行重新表述时，钓鱼攻击的有效性略有提升。整体准确率保持在与零-shot重新表述邮件相同的范围，这表明相比于更简单的提示，少量示例重新表述在某些情况下可能并不十分有效。表格[III](https://arxiv.org/html/2411.13874v1#S4.T3 "TABLE III ‣ IV-B Nigerian Fraud Dataset Experiments ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.") 提供了所有钓鱼检测器性能的详细分类，而表格[IV](https://arxiv.org/html/2411.13874v1#S4.T4 "TABLE IV ‣ IV-B Nigerian Fraud Dataset Experiments ‣ IV Experimental Results ‣ Next-Generation Phishing: How LLM Agents Empower Cyber Attackers § J. Chen acknowledged the support by the Fordham Office of Research through a Fordham AI Research (FAIR) Grant.") 总结了在此数据集上测试的LLM模型的表现，主要关注这里的假阴性值（FN），因为它们表示了未被检测到的有害邮件的数量。

![参考标题](img/f47c566962c88cfb9ada41d1ae62c95a.png)

图5：SVM、朴素贝叶斯和逻辑回归在检测重新表述邮件中的准确度比较：传统数据集与LLM增强数据集。

### IV-C 使用LLM进行重新表述和数据增强

将LLM用于重新表述钓鱼邮件为增强钓鱼数据集提供了重要机会，从而开发出更强大的钓鱼检测器。具体来说，使用像GPT-4和Llama3这样的LLM生成的重新表述邮件。在本研究中，我们介绍了LLM-Nazario数据集，包含5,000封邮件。该数据集的两个版本是通过使用GPT-4和Llama3进行重新表述和数据增强构建的，未来将分享用于支持钓鱼检测模型的微调和模型训练。

图[5](https://arxiv.org/html/2411.13874v1#S4.F5 "图 5 ‣ IV-B 尼日利亚诈骗数据集实验 ‣ IV 实验结果 ‣ 下一代钓鱼攻击：大型语言模型如何助力网络攻击者 § J. Chen 感谢福特汉姆大学研究办公室通过福特汉姆 AI 研究（FAIR）基金的支持。") 展示了支持向量机（SVM）、朴素贝叶斯和逻辑回归模型在三个数据集上的检测准确性，这些数据集分别为：传统的钓鱼邮件数据集、由 GPT-4 生成的 LLM 增强数据集以及由 Llama3 生成的 LLM 增强数据集。这些数据集包含传统的钓鱼邮件和由各自的 LLM 重新措辞的钓鱼邮件，后者旨在绕过标准的钓鱼邮件检测器。然后，模型仅在一组 LLM 重新措辞的钓鱼邮件上进行了测试，这些邮件通过先进的提示工程技术进行设计，旨在逃避检测。结果表明，基于 LLM 增强数据集训练的模型在检测重新措辞的钓鱼邮件方面表现显著优于基于传统数据集训练的模型。平均而言，逻辑回归模型在基于 LLM 增强数据集训练时准确率提高了 10.90%，朴素贝叶斯提高了 10.38%。SVM 展现出了最高的提升，准确率达到了 94.40%，较传统数据集的表现提高了 12.86%。

这些统计学上的提升表明，LLM 增强数据集提供了更全面的训练数据，使钓鱼邮件检测器能够更好地适应重新措辞邮件中的微妙变化。这些变化的引入推动检测器优化其决策边界，从而使其在面对传统和 LLM 重新措辞的钓鱼邮件时都更加有效。

## V 讨论与局限性

来自 Nazario 和尼日利亚诈骗数据集的结果清晰地表明了 LLM 重新措辞的钓鱼邮件所带来的挑战。传统的钓鱼邮件检测器，如 Google Gmail 垃圾邮件检测器、SpamAssassin 和 Proofpoint，在面对原始钓鱼邮件时表现极佳，但在面对重新措辞的邮件时却力不从心，无论是在零样本还是少样本的重新措辞场景中。这些检测器在面对由先进 LLM 生成的更为微妙的钓鱼攻击时，准确率和召回率普遍下降。

我们的研究强调了需要更先进的检测机制来应对这些不断变化的威胁。大型语言模型（LLMs）凭借其生成高度逼真的网络钓鱼邮件的能力，代表了网络钓鱼攻击的新前沿。然而，它们也为通过数据增强提高网络钓鱼检测器的鲁棒性提供了机会。通过将LLM生成的邮件纳入训练过程，我们可以使网络钓鱼检测器接触到更多样的语言变体，从而更好地应对复杂的攻击。所提出的LLM-Nazario数据集是朝着这个方向迈出的重要一步，它提供了一个丰富的重新表述的网络钓鱼邮件来源，可用于训练和微调网络钓鱼检测器。使用LLM增强数据集显著提高了检测准确率。这突显了LLM在网络钓鱼检测领域的双重性质：虽然它们可以用来制作更复杂的攻击，但它们也具有增强网络威胁情报（CTI）以应对这些攻击的潜力。

本研究的主要限制在于其仅关注英语语言数据集。由于当前主要提供的大型高质量数据集都是英语的，将我们的研究扩展到其他语言具有一定挑战性。未来的研究应通过引入多语言数据集来弥补这一缺口，以增强语言模型在不同语言环境下的电子邮件检测能力。另一个限制是我们没有探讨在LLM增强数据集上微调一些小型语言模型的效果，而这可能提供有价值的见解。未来的研究可以集中在这一方面，探索微调如何影响模型的性能和适应性，从而可能带来更好的检测结果。

## VI 结论

本文全面评估了传统网络钓鱼检测器和基于LLM的检测器，重点关注LLM重新表述的网络钓鱼邮件带来的挑战。我们的研究结果表明，尽管传统的网络钓鱼检测器（如Gmail垃圾邮件检测器、SpamAssassin、Proofpoint和最先进的LLM）在检测原始网络钓鱼邮件时表现良好，但在处理LLM重新表述的同一邮件版本时，它们的准确性和召回率明显下降。

然而，通过利用LLM进行数据增强，我们已经证明网络钓鱼检测器可以变得更加鲁棒。在LLM增强数据集上训练模型显著提高了其检测率。这种方法为开发更具韧性的网络钓鱼检测器提供了一种有价值的策略，这些检测器能够适应网络攻击者不断变化的战术。未来的努力应集中在通过创新的机器学习方法不断改进网络钓鱼检测系统，并定期更新训练数据集，以融入新的网络钓鱼策略。

## 参考文献

+   [1] Almomani, A., Gupta, B.B., Atawneh, S., Meulenberg, A., Almomani, E.: 钓鱼邮件过滤技术调查。IEEE通讯调查与教程，15(4)，第2070–2090页。IEEE（2013）

+   [2] Jones, H.S., Towse, J.N., Race, N., Harrison, T.: 邮件欺诈：寻找心理易受攻击的预测因素。PloS one, 14(1), 第e0209684页。公共科学图书馆（2019）

+   [3] McAlaney, J., Hills, P.J.: 通过眼动追踪理解钓鱼邮件的处理与感知信任度。心理学前沿，11，第1756页。Frontiers Media SA（2020）

+   [4] Wang, Y.: 减轻钓鱼威胁。博士论文，圣安德鲁斯大学（2023）

+   [5] Tihanyi, N., Ferrag, M.A., Jain, R., Bisztray, T., Debbah, M.: CyberMetric：基于检索增强生成的基准数据集，用于评估LLMs在网络安全知识中的表现。在2024 IEEE国际网络安全与韧性会议（CSR）上，第296–302页。IEEE（2024）

+   [6] Sahoo, P., Singh, A.K., Saha, S., Jain, V., Mondal, S., Chadha, A.: 大型语言模型中提示工程的系统性调查：技术与应用。arXiv 预印本 arXiv:2402.07927（2024）

+   [7] A. Radford, J. Wu, R. Child, D. Luan, D. Amodei, 和 I. Sutskever，“语言模型是无监督的多任务学习者，”OpenAI博客，第1卷，第8期，第9页，2019。

+   [8] Fette, I., Sadeh, N., Tomasic, A.: 学习检测钓鱼邮件。在第16届国际万维网会议论文集中，第649–656页。ACM，Banff（2007）

+   [9] Ma, L., Ofoghi, B., Watters, P., Brown, S.: 使用混合特征检测钓鱼邮件。在2009年普适、自主与可信计算研讨会论文集中，第493–497页。IEEE（2009）

+   [10] Verma, R., Shashidhar, N., Hossain, N.: 以自然语言方式检测钓鱼邮件。在计算机安全–ESORICS 2012：第17届欧洲计算机安全研究研讨会论文集中，第824–841页。Springer，Pisa（2012）

+   [11] Sheng, S., Wardman, B., Warner, G., Cranor, L., Hong, J., Zhang, C.: 钓鱼黑名单的实证分析。卡内基梅隆大学（2009）

+   [12] Alhogail, A., Alsabih, A.: 应用机器学习和自然语言处理检测钓鱼邮件。计算机与安全，110，第102414页。Elsevier（2021）

+   [13] Wu, F., Zhang, N., Jha, S., McDaniel, P., Xiao, C.: LLM安全新时代：探索现实世界中基于LLM的系统中的安全问题。arXiv 预印本 arXiv:2402.18649（2024）

+   [14] Yao, Y., Duan, J., Xu, K., Cai, Y., Sun, Z., Zhang, Y.: 大型语言模型（LLM）安全性与隐私性调查：优点、缺点与丑陋之处。High-Confidence Computing, 第100211页。Elsevier（2024）

+   [15] Hazell, J.: 使用大型语言模型进行鱼叉式钓鱼攻击。arXiv 2305（2023）

+   [16] Touvron, H., Lavril, T., Izacard, G., Martinet, X., Lachaux, M.-A., Lacroix, T., Rozière, B., Goyal, N., Hambro, E., Azhar, F., 等: Llama：开放且高效的基础语言模型。arXiv 预印本 arXiv:2302.13971（2023）

+   [17] Team, Gemini, Anil, R., Borgeaud, S., Alayrac, J.-B., Yu, J., Soricut, R., Schalkwyk, J., Dai, A. M., Hauth, A., Millican, K., 等：Gemini：一类高效的多模态模型。arXiv预印本 arXiv:2312.11805（2023年）

+   [18] Anthropic: Claude. [在线]. 可用： [https://www.anthropic.com/news/introducing-claude](https://www.anthropic.com/news/introducing-claude)

+   [19] Achiam, J., Adler, S., Agarwal, S., Ahmad, L., Akkaya, I., Aleman, F. L., Almeida, D., Altenschmidt, J., Altman, S., Anadkat, S., 等：GPT-4技术报告。arXiv预印本 arXiv:2303.08774（2023年）

+   [20] Heiding, F., Schneier, B., Vishwanath, A., Bernstein, J., Park, P.S.：利用大型语言模型设计和检测钓鱼邮件。IEEE Access（2024年）

+   [21] Koide, T., Fukushi, N., Nakano, H., Chiba, D.：ChatSpamDetector：利用大型语言模型进行有效的钓鱼邮件检测。arXiv预印本 arXiv:2402.18093（2024年）V. S. Tida 和 S. Hsu，“使用BERT模型的迁移学习进行通用垃圾邮件检测，”arXiv预印本 arXiv:2202.03480，2022年。

+   [22] “SpamAssassin，”Apache，[在线]. 可用： [https://spamassassin.apache.org/](https://spamassassin.apache.org/)。 [访问日期：2024年9月10日]。

+   [23] J. C. Brickley, K. Thakur, 和 A. S. Kamruzzaman，“技术与非技术钓鱼防御的比较分析，”《国际网络安全与数字取证期刊》，第10卷，第1期，第28–41页，2021年。

+   [24] M. Labonne 和 S. Moran，“Spam-t5：基准评估大型语言模型的少样本邮件垃圾邮件检测，”arXiv预印本 arXiv:2304.01238，2023年。

+   [25] Chataut, R., Gyawali, P.K., Usman, Y.：AI能保护你安全吗？一项关于大型语言模型在钓鱼检测中的应用研究。收录于：2024年IEEE第14届年度计算与通信研讨会与会议（CCWC），第0548–0554页。IEEE（2024年）

+   [26] Kang, D., Li, X., Stoica, I., Guestrin, C., Zaharia, M., Hashimoto, T.：利用LLMs的程序行为：通过标准安全攻击的双重用途。arXiv预印本 arXiv:2302.05733（2023年）

+   [27] Roy, S.S., Thota, P., Naragam, K.V., Nilizadeh, S.：从聊天机器人到钓鱼机器人？：商业大型语言模型中的钓鱼诈骗生成。2024年IEEE安全与隐私研讨会（SP），第221–221页，IEEE计算机学会（2024年）

+   [28] Champa, A.I., Rabbi, M.F., Zibran, M.F.：用于钓鱼邮件检测的精心策划的数据集和特征分析，结合机器学习。收录于：第三届IEEE国际计算与机器智能会议（ICMI），第1–7页（待出版）（2024年）

+   [29] Qader, W.A., Ameen, M.M., Ahmed, B.I.：词袋模型概述；重要性、实现、应用及挑战。收录于：2019年国际工程会议（IEC），第200–204页。IEEE（2019年）

+   [30] Ramos, J.：使用tf-idf确定文档查询中的词语相关性。收录于：第一届机器学习教学会议论文集，第242（1）卷，第29–48页。Citeseer（2003年）

+   [31] Church, K.W.：Word2Vec。自然语言工程，23（1），第155–162页。剑桥大学出版社（2017年）

+   [32] Yong, G., Jeon, K., Gil, D., Lee, G.：《利用视觉语言预训练模型进行零-shot和少-shot缺陷检测与分类的提示工程》。计算机辅助土木与基础设施工程，38(11)，1536–1554（2023）

+   [33] Sanh, V., Webson, A., Raffel, C., Bach, S.H., Sutawika, L., Alyafeai, Z., Chaffin, A., Stiegler, A., Scao, T.L., Raja, A., 等：多任务提示训练促进零-shot任务泛化。arXiv预印本arXiv:2110.08207（2021）

+   [34] Kojima, T., Gu, S.S., Reid, M., Matsuo, Y., Iwasawa, Y.：《大型语言模型是零-shot推理器》。神经信息处理系统进展，35，页码22199–22213（2022）

+   [35] Kojima, T., Gu, S.S., Reid, M., Matsuo, Y., Iwasawa, Y.：《大型语言模型是零-shot推理器》。神经信息处理系统进展，35，22199–22213（2022）

+   [36] Ye, X., Durrett, G.：《少-shot推理中的解释不可靠性》。神经信息处理系统进展，35，30378–30392（2022）

+   [37] Reynolds, L., McDonell, K.：《大型语言模型的提示编程：超越少-shot范式》。载于：2021年CHI计算机系统人机交互会议扩展摘要，页码1–7（2021）

+   [38] Chen, Y., Qian, S., Tang, H., Lai, X., Liu, Z., Han, S., Jia, J.：《LongLoRA：长上下文大型语言模型的高效微调》。arXiv预印本arXiv:2309.12307（2023）
