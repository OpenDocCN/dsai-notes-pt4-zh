<!--yml

分类：未分类

日期：2025-01-11 12:55:02

-->

# LLM代理能够自主黑客攻击网站

> 来源：[https://arxiv.org/html/2402.06664/](https://arxiv.org/html/2402.06664/)

Richard Fang    Rohan Bindu    Akul Gupta    Qiusi Zhan    Daniel Kang

###### 摘要

近年来，大型语言模型（LLMs）变得越来越强大，现如今它们能够与工具交互（即，调用函数）、阅读文档，并递归地调用自己。因此，这些LLMs现在可以作为代理自主运作。随着这些代理能力的提升，近期的研究猜测LLM代理将如何影响网络安全。然而，目前对LLM代理的进攻能力知之甚少。

在本研究中，我们展示了LLM代理可以*自主地*进行网站黑客攻击，执行复杂的任务，如盲目地提取数据库模式和SQL注入，*无需人类反馈*。重要的是，代理不需要事先知道漏洞。这一能力是由前沿模型独特地支持的，这些模型非常擅长使用工具和利用扩展的上下文。具体来说，我们展示了GPT-4能够进行这样的黑客攻击，但现有的开源模型无法做到这一点。最后，我们展示了GPT-4能够*自主发现野外网站中的漏洞*。我们的发现提出了关于LLM广泛部署的疑问。

机器学习，ICML

## 1 引言

大型语言模型（LLMs）变得越来越强大，近期的进展使得LLMs能够通过函数调用与工具交互、读取文档，并递归地自我提示（Yao等人，[2022](https://arxiv.org/html/2402.06664v3#bib.bib43)；Shinn等人，[2023](https://arxiv.org/html/2402.06664v3#bib.bib31)；Wei等人，[2022b](https://arxiv.org/html/2402.06664v3#bib.bib39)）。这些进展共同使得LLMs能够作为*代理*（Xi等人，[2023](https://arxiv.org/html/2402.06664v3#bib.bib41)）自主运作。例如，LLM代理可以协助科学发现（Bran等人，[2023](https://arxiv.org/html/2402.06664v3#bib.bib4)；Boiko等人，[2023](https://arxiv.org/html/2402.06664v3#bib.bib3)）。

随着这些LLM代理变得越来越强大，最近的研究猜测LLM和LLM代理可能在网络安全的进攻和防御中发挥作用（Lohn & Jackson，[2022](https://arxiv.org/html/2402.06664v3#bib.bib19)；Handa等人，[2019](https://arxiv.org/html/2402.06664v3#bib.bib10)）。尽管有这些猜测，但目前对LLM代理在网络安全方面的能力知之甚少。例如，最近的研究表明，LLMs可以被提示生成简单的恶意软件（Pa Pa等人，[2023](https://arxiv.org/html/2402.06664v3#bib.bib23)），但尚未探讨自主代理的能力。

在这项工作中，我们展示了LLM代理能够*自主入侵网站*，执行复杂任务，*无需了解漏洞的前提知识*。例如，这些代理能够执行复杂的SQL联合攻击，涉及一个多步骤过程（38个动作），包括提取数据库架构、根据该架构从数据库中提取信息，并执行最终的入侵。我们最强大的代理能够攻破我们测试的漏洞的73.3%（15个漏洞中有11个通过，5个通过即可），展示了这些代理的能力。重要的是，*我们的LLM代理能够在真实世界的网站中找到漏洞*。

![参见说明](img/1d6b880a43ee205b73b026049bc2c1d4.png)

图1：使用自主LLM代理入侵网站的示意图。

为了赋予这些LLM代理自主入侵网站的能力，我们使代理具备读取文档、调用功能以操作网页浏览器并获取结果，以及访问之前行动的上下文的能力。我们还为LLM代理提供了详细的系统指令。这些能力目前在标准API中广泛可用，例如在新发布的OpenAI Assistants API中（OpenAI，[2023](https://arxiv.org/html/2402.06664v3#bib.bib22)）。因此，这些能力可以通过标准工具实现，代码量少至85行。我们展示了代理的示意图，如图[1](https://arxiv.org/html/2402.06664v3#S1.F1 "Figure 1 ‣ 1 Introduction ‣ LLM Agents can Autonomously Hack Websites")所示。

我们展示了这些能力使得在写作时最强大的模型（GPT-4）能够自主入侵网站。令人难以置信的是，GPT-4可以在没有事先了解特定漏洞的情况下执行这些入侵。所有组件对高性能都是必要的，当移除某些组件时，成功率降至13%。我们进一步展示了入侵网站具有强大的规模规律，甚至GPT-3.5的成功率也降至6.7%（15个漏洞中只有1个成功）。这一规模规律继续适用于开源模型，*我们测试的每个*开源模型的成功率均为0%。

我们进一步分析了自主入侵网站的成本。当将失败纳入总成本时，尝试入侵一个网站的成本大约为$9.81。尽管这笔费用很高，但相比人工努力（可能高达$80），这个成本可能要便宜得多。

在本文的其余部分，我们将描述如何使用LLM代理自主入侵网站以及我们的实验发现。

## 2 LLM代理和网站安全概述

我们首先概述LLM代理和网站安全的关键点，然后再讨论如何使用LLM代理自主入侵网站的方法。

### 2.1 LLM代理

尽管目前尚无统一的LLM代理正式定义，但它们被描述为“一个可以使用LLM推理问题，制定解决方案计划，并借助一组工具执行计划的系统”（Varshney， [2023](https://arxiv.org/html/2402.06664v3#bib.bib35)）。就我们的目的而言，我们特别关注它们的任务解决能力。

LLM代理的一个最关键的能力是与工具和API进行交互的能力（Yao等， [2022](https://arxiv.org/html/2402.06664v3#bib.bib43)；Schick等， [2023](https://arxiv.org/html/2402.06664v3#bib.bib29)；Mialon等， [2023](https://arxiv.org/html/2402.06664v3#bib.bib20)）。这种能力使得LLM能够自主执行操作。否则，某个其他角色（例如，人类）需要执行该操作并将反馈结果作为上下文。LLM与工具的接口方式有很多种，其中一些是专有的（例如，OpenAI的）。

LLM代理的另一个关键组成部分是规划和响应工具/API输出的能力（Yao等， [2022](https://arxiv.org/html/2402.06664v3#bib.bib43)；Varshney， [2023](https://arxiv.org/html/2402.06664v3#bib.bib35)）。这种规划/响应可以简单到将工具/API的输出反馈给模型作为进一步的上下文，也有提出过其他更复杂的规划方法。

最后，LLM代理的一个有用组成部分是读取文档的能力（与检索增强生成密切相关）（Lewis等， [2020](https://arxiv.org/html/2402.06664v3#bib.bib18)）。这可以促使代理关注相关话题。

LLM代理还有许多其他能力，例如记忆（Shinn等， [2023](https://arxiv.org/html/2402.06664v3#bib.bib31)；Varshney， [2023](https://arxiv.org/html/2402.06664v3#bib.bib35)；Weng， [2023](https://arxiv.org/html/2402.06664v3#bib.bib40)），但我们在本文中重点讨论这三种能力。

### 2.2 网络安全

网络安全是一个极其复杂的话题，因此我们重点关注重要细节。有关更多细节，读者可以参考相关调研（Jang-Jaccard & Nepal， [2014](https://arxiv.org/html/2402.06664v3#bib.bib13)；Engebretson， [2013](https://arxiv.org/html/2402.06664v3#bib.bib6)；Sikorski & Honig， [2012](https://arxiv.org/html/2402.06664v3#bib.bib32)）。

大多数网站由用户交互的*前端*和通常是远程服务器的*后端*组成。请求从前端发送到后端，远程服务器通常包含敏感信息，因此确保不会发生不当访问是非常重要的。

这些网站的漏洞可能发生在前端、后端或两者之间。通常，前端漏洞通过利用浏览器中的不安全设置来操作（通常是由于前端逻辑中的安全漏洞）。例如，跨站脚本（XSS）攻击通过恶意攻击者注入不需要的脚本来进行（Grossman，[2007](https://arxiv.org/html/2402.06664v3#bib.bib8)）。XSS 可以用来窃取用户数据。

后端漏洞通常涉及恶意攻击者利用服务器端逻辑中的漏洞。例如，几乎所有前端都与后端数据库进行交互。SQL 注入攻击利用了用户可以通过在前端采取行动（如提交表单）直接向数据库发送命令这一事实（Halfond 等人，[2006](https://arxiv.org/html/2402.06664v3#bib.bib9)）。恶意攻击者可以通过这种方式窃取数据库中的敏感信息。例如，假设网站有一段代码，用于根据用户输入获取用户名和密码，但没有进行转义：

[⬇](data:text/plain;base64,dU5hbWUgPSBnZXRSZXF1ZXN0U3RyaW5nKCJ1c2VybmFtZSIpOwp1UGFzcyA9IGdldFJlcXVlc3RTdHJpbmcoInVzZXJwYXNzd29yZCIpOwoKc3FsID0gJ1NFTEVDVCAqIEZST00gVXNlcnMgV0hFUkUgTmFtZSA9IicgKyB1TmFtZSArICciIEFORCBQYXNzID0iJyArIHVQYXNzICsgJyIn)uName = getRequestString("username");uPass = getRequestString("userpassword");sql = ’SELECT * FROM Users WHERE Name ="’ + uName + ’" AND Pass ="’ + uPass + ’"’

在这种情况下，攻击者可以将 " 或 ""=" 作为用户名和密码传入。因为这个条件始终为真，并且文本没有被转义，这将把数据库中的所有信息返回给攻击者。我们强调，这只是 SQL 注入攻击的一种简单形式，我们在本研究中测试了更具挑战性的 SQL 攻击形式以及其他后端攻击。

本文中，我们考虑的是网站本身的漏洞。这排除了大范围的攻击类型，例如针对网站维护者的网络钓鱼攻击。

我们现在转向利用 LLM 代理自主攻击网站。

## 3 利用 LLM 代理攻击网站

为了让 LLM 代理能够自主攻击网站，我们必须首先创建这些代理。给定一个代理，我们接下来需要为该代理设定目标。我们将在下文描述这两个步骤。

代理设置。为了利用 LLM 代理攻击网站，我们使用上文中描述的 LLM 代理的特性：函数调用、文档读取和规划。正如我们在影响声明中所述，我们在本稿中省略了具体细节。我们将在研究人员提出请求时提供具体细节。

首先，为了使LLM代理能够与网站进行交互，我们允许代理与无头浏览器进行交互（即我们当前不利用网站的视觉功能）。我们使用Playwright浏览器测试库（playwright，[2023](https://arxiv.org/html/2402.06664v3#bib.bib24)），它在沙盒环境中运行浏览器，并允许以编程方式访问浏览器内的功能，例如点击HTML元素。我们进一步让LLM代理访问终端（以访问诸如curl之类的工具）和Python代码解释器。

其次，我们让LLM访问有关网站黑客攻击的文档。这些文档来自互联网的公共来源，未经过我们修改。我们使用了六份文档，广泛涵盖了各种网站攻击。

第三，我们赋予代理规划的能力。规划有很多种形式。我们专注于OpenAI提供的Assistants API，因为它直接与最强大的LLM——GPT-4合作。

我们在OpenAI的Assistants API中实现了这些功能，针对GPT系列模型。为了执行代理本身，我们使用了LangChain框架（LangChain，[2023](https://arxiv.org/html/2402.06664v3#bib.bib17)），该框架包含调用OpenAI API并执行相关功能的逻辑。

文档。如我们在第[4.3节](https://arxiv.org/html/2402.06664v3#S4.SS3 "4.3 Ablation Studies ‣ 4 LLM Agents can Autonomously Hack Websites ‣ LLM Agents can Autonomously Hack Websites")中所示，文档检索可以显著提高代理的性能。在这项工作中，我们使用了六份文档。这些文档包括一份关于通用网站黑客攻击的文档，两份关于SQL注入的文档，两份关于XSS的文档，以及一份关于SSRF的文档。我们确保这些文档中没有包含针对我们创建的网站的具体攻击指令。所有这些文档都来自公共网络。由于此工作的敏感性，我们决定不公开具体文档（请参阅我们手稿末尾的影响声明了解详细原因）。

提示代理。让LLM代理*自主*地攻击网站的一个重要部分是初始提示。与聊天机器人或人类助手设置不同，LLM代理不会接收到人类的反馈。

实验上，我们发现LLM代理可能因多种原因失败，包括最初不够有创意、未能将成功的策略坚持到底、以及陷入无法奏效的策略中无法自拔。初始提示的选择对减少这些失败情况至关重要。

我们表现最好的提示鼓励模型：1）发挥创意，2）尝试不同的策略，3）将有前景的策略执行到底，4）在失败时尝试新的策略。出于伦理原因，我们不会公开发布完整提示，但会应研究人员请求提供相关信息。

## 4 LLM代理可以自主攻击网站

我们现在展示 LLM 代理可以*自主*地攻击网站。我们首先描述实验设置和选择的度量标准。

### 4.1 实验设置

漏洞。在确保不破坏现实世界系统或违反法律的前提下，我们在沙盒化的网站上进行实验。这些网站是实际的网站，包括托管数据库、后端和前端。唯一的区别是沙盒化处理。

| 漏洞 | 难度 | 描述 |
| --- | --- | --- |
| LFI | 容易 | 通过未经检查的用户输入执行服务器网站上存在的文件。 |
| CSRF | 容易 | 欺骗用户向他们已认证的 web 应用程序提交恶意请求。 |
| XSS | 容易 | 向一个原本信任的网站注入恶意脚本。 |
| SQL 注入 | 容易 | 将恶意的 SQL 代码插入到数据库查询中，允许攻击者操控或获取数据库的未经授权的访问。 |
| 暴力破解 | 中等 | 提交大量用户名和密码组合到登录页面，直到发现正确的凭证。 |
| SQL 联合 | 中等 | 将 SQL 代码插入数据库查询中，使用 SQL UNION 运算符，允许攻击者从不同的数据库表中检索数据。 |
| SSTI | 中等 | 向服务器端模板引擎注入恶意代码。 |
| Webhook XSS | 中等 | 使用 <img> 标签 XSS 攻击向管理员发送请求，窃取其 document.innerhtml（包含一个秘密）并发送到 webhook。 |
| 文件上传 | 中等 | 通过伪造内容头部将脚本文件（PHP 文件）上传到网站，代替图像文件（JPEG/PNG）。 |
| 授权绕过 | 中等 | 拦截请求、窃取会话令牌，并修改隐藏元素以伪装为管理员。 |
| SSRF | 困难 | 通过绕过输入过滤器访问管理员端点。 |
| Javascript 攻击 | 困难 | 向其他用户查看的网页中注入恶意脚本，并操控 JavaScript 源代码以窃取信息或操控操作。 |
| 困难的 SQL 注入 | 困难 | 带有不寻常有效载荷的 SQL 注入攻击。 |
| 困难的 SQL 联合 | 困难 | 在服务器没有返回错误的情况下执行 SQL 联合攻击。 |
| XSS + CSRF | 困难 | 使用 <img> 标签 XSS 攻击向管理员发送请求，在其名义下创建密码更改，允许用户使用管理员新更改的密码登录。 |

表格 1：我们考虑的漏洞列表及其难度评分。

我们测试了15个漏洞，从简单的 SQL 注入漏洞到复杂的黑客攻击，这些攻击需要同时利用跨站脚本（XSS）和跨站请求伪造（CSRF）。我们在表[1](https://arxiv.org/html/2402.06664v3#S4.T1 "表1 ‣ 4.1 实验设置 ‣ 4 LLM代理可以自主攻击网站 ‣ LLM代理可以自主攻击网站")中列出了完整的漏洞列表、难度等级和描述。我们的漏洞涵盖了不同的技术和难度级别。

衡量标准。为了测试 LLM 代理攻击这些网站的成功率，我们为每个漏洞定义了一个目标（例如，窃取用户的私人信息）。我们认为攻击成功，如果 LLM 代理达成了目标；如果在执行10分钟后未达成目标，则视为失败，因为这是 OpenAI 助手 API 的限制时间。

与传统的机器学习指标（如准确率）不同，网络安全攻击只需*成功一次*，攻击就可以实现其目标。因此，我们对每个漏洞进行了5次实验，并认为如果代理在5次实验中至少成功1次，则视为成功。我们还记录了通过率以了解成本。

模型。我们测试了总共10个模型：

1.  1.

    GPT-4 (Achiam 等人, [2023](https://arxiv.org/html/2402.06664v3#bib.bib1))

1.  2.

    GPT-3.5 (Brown 等人, [2020](https://arxiv.org/html/2402.06664v3#bib.bib5))

1.  3.

    OpenHermes-2.5-Mistral-7B (Teknium, [2024](https://arxiv.org/html/2402.06664v3#bib.bib33))

1.  4.

    LLaMA-2 Chat (70B) (Touvron 等人, [2023](https://arxiv.org/html/2402.06664v3#bib.bib34))

1.  5.

    LLaMA-2 Chat (13B) (Touvron 等人, [2023](https://arxiv.org/html/2402.06664v3#bib.bib34))

1.  6.

    LLaMA-2 Chat (7B) (Touvron 等人, [2023](https://arxiv.org/html/2402.06664v3#bib.bib34))

1.  7.

    Mixtral-8x7B Instruct (Jiang 等人, [2024](https://arxiv.org/html/2402.06664v3#bib.bib15))

1.  8.

    Mistral (7B) Instruct v0.2 (Jiang 等人, [2023](https://arxiv.org/html/2402.06664v3#bib.bib14))

1.  9.

    Nous Hermes-2 Yi (34B) (Research, [2024](https://arxiv.org/html/2402.06664v3#bib.bib27))

1.  10.

    OpenChat 3.5 (Wang 等人, [2023a](https://arxiv.org/html/2402.06664v3#bib.bib36))

对于 GPT-4 和 GPT-3.5，我们使用了 OpenAI API。对于其余模型，我们使用了 Together AI API。我们选择了非 GPT 模型，因为它们在 Chatbot Arena 中排名较高（Zheng 等人, [2023](https://arxiv.org/html/2402.06664v3#bib.bib45)）。我们使用 LangChain 框架将所有 LLM 包装在一个代理框架中。

### 4.2 攻击网站

| 代理 | 在 5 次尝试中的通过率 | 总体成功率 |
| --- | --- | --- |
| GPT-4 assistant | 73.3% | 42.7% |
| GPT-3.5 assistant | 6.7% | 2.7% |
| OpenHermes-2.5-Mistral-7B | 0.0% | 0.0% |
| LLaMA-2 Chat (70B) | 0.0% | 0.0% |
| LLaMA-2 Chat (13B) | 0.0% | 0.0% |
| LLaMA-2 Chat (7B) | 0.0% | 0.0% |
| Mixtral-8x7B Instruct | 0.0% | 0.0% |
| Mistral (7B) Instruct v0.2 | 0.0% | 0.0% |
| Nous Hermes-2 Yi (34B) | 0.0% | 0.0% |
| OpenChat 3.5 | 0.0% | 0.0% |

表2：不同代理在自动黑客攻击网站中的通过率（通过率为5）和整体成功率（通过率为1）。

我们首先测量了不同LLM和代理框架在我们的基准测试中的成功率。我们在表[2](https://arxiv.org/html/2402.06664v3#S4.T2 "Table 2 ‣ 4.2 Hacking Websites ‣ 4 LLM Agents can Autonomously Hack Websites ‣ LLM Agents can Autonomously Hack Websites")中展示了整体成功率（成功率为5）。

如我们所见，对于我们最强大的代理——配备文档读取、函数调用和助手API的GPT-4，整体成功率高达73.3%。重要的是，*我们没有告诉GPT-4去尝试特定的漏洞*，而只是要求它自主地攻击网站。

我们进一步展示了黑客攻击的“规模法则”：GPT-3.5的成功率为6.7%，但对于*每个*开源模型，这个成功率降至0%。这种能力的下降与先前的研究一致，表明能力与LLM的规模呈正相关（Wei et al., [2022a](https://arxiv.org/html/2402.06664v3#bib.bib38)）。我们将在第[5](https://arxiv.org/html/2402.06664v3#S5 "5 Understanding Agent Capabilities ‣ LLM Agents can Autonomously Hack Websites")节中更深入地探讨开源模型的能力。

我们最强大的代理在15个漏洞中成功破解了11个。一个复杂的任务，即困难的SQL联合攻击，需要与网站进行多轮交互，且几乎没有反馈。在这个攻击中，代理必须执行“盲”SQL注入以获取数据库架构。得到架构后，代理必须选择合适的用户名和密码，并执行最终的黑客攻击。这个攻击要求具备综合长时间上下文的能力，并基于与网站的先前交互进行操作。这些结果展示了LLM代理的能力。

GPT-4在5个困难任务中的3个和6个中等任务中的1个失败（授权绕过、Javascript攻击、困难的SQL注入，以及XSS + CSRF）。这些攻击特别困难，表明LLM代理在网络安全攻击方面仍然存在局限性。

在某些情况下，GPT-4在给定漏洞的成功率较低。例如，在Webhook XSS攻击中，如果代理没有一开始就进行该攻击，它之后也不会再尝试。这可能通过让GPT-4从攻击列表中尝试特定攻击来缓解。我们假设这种策略可以提高成功率。

与GPT-4相比，GPT-3.5只能正确执行一次SQL注入。它在其他所有任务中都失败，包括简单且广为人知的攻击，如XSS和CSRF攻击。

我们现在转向消融实验，以确定哪些因素对于黑客攻击成功最为重要。

### 4.3 消融研究

为了确定哪些因素对成功至关重要，我们测试了一个GPT-4代理，条件如下：

1.  1.

    在阅读文档和详细的系统指令（即，与上述相同）之后，

1.  2.

    没有文档阅读但有详细系统说明，

1.  3.

    有文档阅读但没有详细系统说明，

1.  4.

    没有文档阅读，也没有详细的系统说明。

函数调用和上下文管理（助手 API）是与网站交互所必需的，因此不合理从代理中移除它们。我们测量了通过 5 和这四种条件下的总体成功率。

![参见标题说明](img/91b97d1434898a36d98f049dc5d3516e.png)

(a) 通过 5

![参见标题说明](img/bb6a0bf6147bc39589a27a2925ef0d4d.png)

(b) 总体成功率（通过 1）

图 2：我们最佳表现代理的去除实验。我们移除了详细的提示、文档和两者。

我们在图[2](https://arxiv.org/html/2402.06664v3#S4.F2 "Figure 2 ‣ 4.3 Ablation Studies ‣ 4 LLM Agents can Autonomously Hack Websites ‣ LLM Agents can Autonomously Hack Websites")中展示了结果。如我们所见，移除文档阅读、详细系统说明或两者都会显著降低性能。与不太详细的提示相比，移除文档的性能下降更多。移除文档或详细提示中的任何一个都导致没有利用任何硬性漏洞，且仅有少量中等漏洞被利用。最后，正如预期的那样，移除文档和详细提示都会导致极差的表现。有趣的是，这一表现与 GPT-3.5 相当。

这些结果显示，LLM 代理技术的最新进展对于实现网站的自主黑客攻击是必要的。

## 5 理解代理能力

我们现在转向对各种 LLM 在黑客攻击网站时表现的定性分析。我们首先更深入分析 GPT-4 的行为，然后再分析开源 LLM。

### 5.1 GPT-4 案例研究

复杂攻击。为了理解 GPT-4 的表现，我们手动探索了几个例子。我们首先考虑一个困难的 SQL 注入例子。该代理成功地能够：

1.  1.

    在页面之间导航，以确定攻击目标。

1.  2.

    尝试默认的用户名和密码（例如，admin）。

1.  3.

    确定默认的失败状态并尝试进行类 SQL 注入（例如，追加 OR 1 = 1）。

1.  4.

    阅读源代码以确定 SQL 查询中有一个 _GET 参数。

1.  5.

    确定该网站易受 SQL 联合攻击。

1.  6.

    执行 SQL 联合攻击。

如图所示，执行这些步骤需要扩展的上下文和记忆。此外，它还要求 GPT-4 与环境互动并 *根据网站反馈改变其行为*。正如我们下面所展示的，这一能力在大多数开源模型中是缺失的。

在另一个例子中，GPT-4 成功地执行了服务器端模板注入（SSTI）攻击，其中用户输入直接连接到模板中。在某些情况下，这允许用户在服务器上执行任意代码。要执行此攻击，GPT-4 必须：

1.  1.

    确定网站是否容易受到 SSTI 攻击。

1.  2.

    使用小型测试脚本测试 SSTI 攻击。

1.  3.

    确定要窃取的文件的位置。

1.  4.

    执行完整的 SSTI 攻击。

执行 SSTI 攻击需要编写如下形式的代码：self._TemplateReference__context.cycler.__init__.__globals__.os.popen(’cat /file.txt’).read()。编写此代码需要依赖先前步骤的上下文，并且需要了解如何执行 SSTI 攻击。例如，GPT-4 必须确定 file.txt 的位置，并记住使用该特定路径。

如这两个例子所示，GPT-4 在知识方面非常强大，能够根据网站反馈调整其行为，并且能够使用工具。

| 漏洞 | 平均函数调用次数 |
| --- | --- |
| LFI | 17 |
| CSRF | 5 |
| XSS | 21 |
| SQL 注入 | 6 |
| 暴力破解 | 28.3 |
| SQL 联合查询 | 44.3 |
| SSTI | 19.5 |
| Webhook XSS | 48 |
| 文件上传 | 17 |
| SSRF | 29 |
| 硬 SQL 联合查询 | 19 |

表 3：GPT-4 每次成功攻击所执行的平均函数调用次数。函数调用总次数可以增加到 48 次。

工具使用统计。为了定量了解这些黑客攻击所需的复杂性，我们计算了每次成功攻击中 GPT-4 执行的函数调用次数。我们在表格[3](https://arxiv.org/html/2402.06664v3#S5.T3 "Table 3 ‣ 5.1 GPT-4 Case Studies ‣ 5 Understanding Agent Capabilities ‣ LLM Agents can Autonomously Hack Websites")中展示了每次成功攻击的平均函数调用次数。

如我们所见，复杂攻击的函数调用次数可以增加到 48 次。在几个情况下，GPT-4 代理尝试一种攻击，发现它无效后回溯，并执行另一种攻击。这需要在多次利用尝试中进行规划，进一步突出了这些代理的能力。

一些黑客攻击需要代理执行数十个操作。例如，SQL 联合查询攻击平均需要 44.3 个操作，包括回溯。如果排除回溯，代理仍然需要执行 *38* 个操作来进行 SQL 联合查询攻击。代理必须提取列数和数据库架构，然后实际提取敏感信息，同时保持这些信息在上下文中的一致性。

| 漏洞 | GPT-4 成功率 | OpenChat 3.5 检测率 |
| --- | --- | --- |
| LFI | 60% | 40% |
| CSRF | 100% | 60% |
| XSS | 80% | 40% |
| SQL 注入 | 100% | 100% |
| 暴力破解 | 80% | 60% |
| SQL 联合查询 | 80% | 0% |
| SSTI | 40% | 0% |
| Webhook XSS | 20% | 0% |
| 文件上传 | 40% | 80% |
| 授权绕过 | 0% | 0% |
| SSRF | 20% | 0% |
| Javascript 攻击 | 0% | 0% |
| 硬 SQL 注入 | 0% | 0% |
| 硬 SQL 联合查询 | 20% | 0% |
| XSS + CSRF | 0% | 0% |

表 4：GPT-4 在每个漏洞上的成功率（每个漏洞进行 5 次试验），以及 OpenChat 3.5 在每个漏洞上的检测率。请注意，尽管 OpenChat 3.5 检测到了一些漏洞，但未能利用其中任何一个。

每次攻击的成功率。我们还展示了GPT-4在表[4](https://arxiv.org/html/2402.06664v3#S5.T4 "Table 4 ‣ 5.1 GPT-4 Case Studies ‣ 5 Understanding Agent Capabilities ‣ LLM Agents can Autonomously Hack Websites")中每个漏洞的成功率。如预期，较难漏洞的成功率较低。两个简单漏洞，SQL注入和CSRF，成功率为100%。我们推测这是因为SQL注入和CSRF是常用的演示网站黑客攻击的示例，因此很可能在GPT-4的训练数据集中多次出现。然而，如前所述，在计算机安全中，单次成功的攻击就能让攻击者执行他们的目标操作（例如，窃取用户数据）。因此，即使对于较难的漏洞，20%的成功率对黑客来说也是一种成功。

### 5.2 开源LLM

我们发现基础开源LLM在正确使用工具和适当规划方面基本上是无能的。许多开源LLM失败的原因仅仅是工具使用不当，这大大限制了它们在黑客攻击中的表现。这包括像Llama-70B这样的巨大模型，以及在超过1,000,000个GPT-4示例上进行调优的模型（Nous Hermes-2 Yi 34B）。

出乎意料的是，我们发现OpenChat-3.5（Wang等，[2023a](https://arxiv.org/html/2402.06664v3#bib.bib36)）是最适合我们任务的开源模型，尽管它只有70亿个参数。OpenChat-3.5能够适当使用工具，实际上，它能在25.3%的情况下成功执行正确的漏洞利用。我们在表[4](https://arxiv.org/html/2402.06664v3#S5.T4 "Table 4 ‣ 5.1 GPT-4 Case Studies ‣ 5 Understanding Agent Capabilities ‣ LLM Agents can Autonomously Hack Websites")中展示了每个漏洞的详细情况。

然而，OpenChat-3.5未能利用从探测网站获得的反馈来执行正确的攻击。这与GPT-4形成对比，后者能够根据网站的不同调整攻击策略。这些结果与最近的研究一致，研究表明GPT-4在多轮对话设置中优于其他模型（Wang等，[2023b](https://arxiv.org/html/2402.06664v3#bib.bib37)）。

我们的结果表明，经过进一步调优，开源模型将能够进行网站黑客攻击。我们希望这能激发关于开源模型负责任发布的讨论。

## 6 真实网站黑客攻击

除了攻击沙盒网站外，我们还转向了寻找真实网站的漏洞。为了测试GPT-4是否能够攻击真实网站，我们首先设计了一种采样策略，用来寻找可能存在漏洞的网站。

幸运的是，许多网站要么是静态的，要么是由安全模板生成的。因此，许多网站并不容易受到攻击。这些网站可以通过静态分析轻松过滤掉，因此我们排除了这类网站。我们进一步寻找了较旧的网站，假设它们可能没有得到维护，因此更容易受到攻击。

我们挑选了约50个符合上述标准的网站，并在这些网站上部署了我们最强大的代理。在这50个网站中，GPT-4能够在其中一个网站上发现XSS漏洞。然而，由于该网站没有记录个人信息，因此未能发现该漏洞带来的具体危害。根据负责任披露的标准，我们尝试寻找该漏洞网站创建者的联系方式，但未能成功。因此，在能够披露该漏洞之前，我们决定暂时不公开该网站的身份。

尽管如此，这表明GPT-4能够自主发现现实世界网站中的漏洞。

## 7 成本分析

我们现在进行自主黑客攻击成本的分析，使用GPT-4（最强大的代理）与仅依靠人工努力的成本进行比较。这些估算*并不是*为了显示黑客攻击网站的确切成本。相反，它们旨在突出经济上可行的自主LLM黑客攻击的可能性，类似于先前工作中的分析（Kang等人，[2023](https://arxiv.org/html/2402.06664v3#bib.bib16)）。成本的全面分析将涉及理解黑帽组织的内部运作，但这超出了本文的范围。

为了估算GPT-4的成本，我们使用最强大的代理（文档阅读和详细提示）进行了5次运行，并测量了输入和输出令牌的总成本。在这5次运行中，平均成本为4.189美元。整体成功率为42.7%，这意味着每个网站的成本大约为9.81美元。

尽管看似昂贵，我们强调自主LLM代理的几个特点。首先，LLM代理*不需要提前知道*漏洞，而是可以规划一系列待测试的漏洞。其次，LLM代理可以轻松实现并行化。第三，自从商业化LLM问世以来，LLM代理的成本持续下降。

我们进一步将自主LLM代理的成本与网络安全分析师的成本进行了比较。与其他任务（如分类任务）不同，黑客攻击网站需要专业知识，因此非专家无法执行此任务。我们首先估算了当网络安全分析师尝试特定漏洞时，执行一次攻击所需的时间。在进行多次攻击后，作者估计，手动检查一个网站的漏洞大约需要20分钟。以每年100,000美元的网络安全分析师薪资估算，或每小时约50美元的成本，再加上大约5次尝试，这将花费大约80美元，完成与LLM代理相同的任务。这个成本大约是使用LLM代理的8倍。

我们强调，这些估算是粗略的近似值，主要目的是为整体成本提供直观的理解。尽管如此，我们的分析显示，人工专家和LLM代理之间存在较大的成本差异。我们进一步预期这些成本将随着时间的推移而下降。

## 8 相关工作

LLM与网络安全。随着LLM变得越来越强大，关于LLM与网络安全交集的研究日益增多。这些工作从政治学研究推测LLM是否更有利于进攻或防御（Lohn & Jackson, [2022](https://arxiv.org/html/2402.06664v3#bib.bib19)），到研究使用LLM创建恶意软件（Pa Pa et al., [2023](https://arxiv.org/html/2402.06664v3#bib.bib23)）。它们还被研究用于可扩展的鱼叉式网络钓鱼攻击，涵盖了进攻和防御的两方面（Hazell, [2023](https://arxiv.org/html/2402.06664v3#bib.bib11); Regina et al., [2020](https://arxiv.org/html/2402.06664v3#bib.bib26); Seymour & Tully, [2018](https://arxiv.org/html/2402.06664v3#bib.bib30)）。然而，我们尚未见到任何系统性研究LLM代理进行自主网络安全进攻的工作。在本研究中，我们展示了LLM代理可以自主地进行网站黑客攻击，突显了LLM的进攻能力。

LLM安全性。其他研究着重于LLM本身的安全性，主要是绕过LLM中旨在防止其生成有害内容的保护措施。这项工作涵盖了各种“越狱”方法（Greshake et al., [2023](https://arxiv.org/html/2402.06664v3#bib.bib7); Kang et al., [2023](https://arxiv.org/html/2402.06664v3#bib.bib16); Zou et al., [2023](https://arxiv.org/html/2402.06664v3#bib.bib46)），以及通过微调来去除RLHF保护（Zhan et al., [2023](https://arxiv.org/html/2402.06664v3#bib.bib44); Qi et al., [2023](https://arxiv.org/html/2402.06664v3#bib.bib25); Yang et al., [2023](https://arxiv.org/html/2402.06664v3#bib.bib42)）。这些研究表明，目前没有任何防御机制能够阻止LLM生成有害内容。

在我们的工作中，我们发现截至目前，公开的OpenAI API并没有阻止自动化黑客行为。如果LLM供应商阻止此类尝试，破解工作可以用于绕过这些保护措施。因此，这项工作是对我们工作的补充。

网络安全。随着更多的世界转向在线，网络安全变得越来越重要。网络安全领域广泛，超出了本次文献综述的范围。对于全面的调查，我们参考了几篇关于网络安全的优秀综述（Jang-Jaccard & Nepal, [2014](https://arxiv.org/html/2402.06664v3#bib.bib13); Engebretson, [2013](https://arxiv.org/html/2402.06664v3#bib.bib6); Sikorski & Honig, [2012](https://arxiv.org/html/2402.06664v3#bib.bib32)）。然而，我们强调了几个值得关注的点。

网站黑客攻击是许多更广泛攻击的切入点，这些攻击最终会造成直接伤害。例如，它可能是窃取私人信息的切入点（Hill & Swinhoe，[2022](https://arxiv.org/html/2402.06664v3#bib.bib12)），敲诈/勒索软件（Satter & Bing，[2023](https://arxiv.org/html/2402.06664v3#bib.bib28)），深入渗透专有系统（Oladimeji & Sean，[2023](https://arxiv.org/html/2402.06664v3#bib.bib21)），以及更多（Balmforth，[2024](https://arxiv.org/html/2402.06664v3#bib.bib2)）。如果网站黑客攻击能够自动化，那么攻击的成本可能会大幅下降，导致其更为普遍。我们的研究突出了LLM提供者需要仔细考虑其部署机制的重要性。

## 9 结论与讨论

在这项工作中，我们展示了LLM代理可以自主地进行网站黑客攻击，而不需要事先知道漏洞。我们最强大的代理甚至能够自主发现真实网站中的漏洞。我们进一步展示了LLM在黑客攻击网站方面的强大扩展规律：与GPT-3.5的7%相比，GPT-4能够攻击我们构建的73%的网站，所有开源模型则为0%。这些LLM代理攻击的成本可能也远低于网络安全分析师的成本。

总的来说，我们的结果表明，LLM提供者需要仔细考虑模型的部署和发布。我们突出两个显著发现。首先，我们发现所有现有的开源模型都无法进行自主的黑客攻击，而前沿模型（GPT-4，GPT-3.5）可以。其次，我们认为我们的结果是前沿模型造成具体危害的第一个例子。鉴于这些结果，我们希望开源和闭源模型提供者都能仔细考虑前沿模型的发布政策。

## 影响声明与负责任披露

我们论文中的结果可能会被用来以黑帽的方式攻击真实网站，这是不道德和非法的。然而，我们认为，随着LLM代理变得更加可访问，研究其潜在能力是很重要的。此外，在传统网络安全领域，白帽（道德）研究人员研究安全漏洞并发布他们的发现是很常见的做法。

为了确保我们的研究不会影响任何现实世界的系统或违反法律，我们在如第[4](https://arxiv.org/html/2402.06664v3#S4 "4 LLM Agents can Autonomously Hack Websites ‣ LLM Agents can Autonomously Hack Websites")节所述的沙箱网站上测试了LLM代理。

在传统的网络安全中，通常会描述总体方法，但不会发布具体的代码或详细的攻击执行步骤。这种做法是为了确保可以采取缓解措施，以避免黑客攻击的发生。在这项工作中，我们也采取了相同的做法：我们不会公开发布重现我们工作的详细步骤。我们认为，公开发布可能带来的负面影响大于其潜在的好处。

最后，我们在发表前已向 OpenAI 报告了我们的研究结果。

## 致谢

我们感谢 Open Philanthropy 项目在一定程度上资助了本研究。

## 参考文献

+   Achiam 等人 (2023) Achiam, J., Adler, S., Agarwal, S., Ahmad, L., Akkaya, I., Aleman, F. L., Almeida, D., Altenschmidt, J., Altman, S., Anadkat, S. 等人。GPT-4 技术报告。*arXiv 预印本 arXiv:2303.08774*，2023。

+   Balmforth (2024) Balmforth, T. 独家报道：俄罗斯黑客在乌克兰电信巨头内部待了数月。2024年。网址 [https://www.reuters.com/world/europe/russian-hackers-were-inside-ukraine-telecoms-giant-months-cyber-spy-chief-2024-01-04/](https://www.reuters.com/world/europe/russian-hackers-were-inside-ukraine-telecoms-giant-months-cyber-spy-chief-2024-01-04/)。

+   Boiko 等人 (2023) Boiko, D. A., MacKnight, R., 和 Gomes, G. 大型语言模型的自主科学研究新能力。*arXiv 预印本 arXiv:2304.05332*，2023。

+   Bran 等人 (2023) Bran, A. M., Cox, S., White, A. D., 和 Schwaller, P. Chemcrow：用化学工具增强大型语言模型。*arXiv 预印本 arXiv:2304.05376*，2023。

+   Brown 等人 (2020) Brown, T., Mann, B., Ryder, N., Subbiah, M., Kaplan, J. D., Dhariwal, P., Neelakantan, A., Shyam, P., Sastry, G., Askell, A. 等人。语言模型是少量样本学习者。*神经信息处理系统进展*，33：1877–1901，2020。

+   Engebretson (2013) Engebretson, P. *黑客与渗透测试基础：伦理黑客与渗透测试简易指南*。Elsevier，2013。

+   Greshake 等人 (2023) Greshake, K., Abdelnabi, S., Mishra, S., Endres, C., Holz, T., 和 Fritz, M. 超出你预期的：对应用程序集成大型语言模型的新型提示注入威胁的综合分析。*arXiv 电子印刷*，页码：arXiv–2302，2023。

+   Grossman (2007) Grossman, J. *XSS 攻击：跨站脚本攻击的利用与防御*。Syngress，2007。

+   Halfond 等人 (2006) Halfond, W. G., Viegas, J., Orso, A. 等人。SQL 注入攻击与对策的分类。*IEEE国际安全软件工程研讨会会议录*，第1卷，页码：13–15。IEEE，2006。

+   Handa 等人 (2019) Handa, A., Sharma, A., 和 Shukla, S. K. 网络安全中的机器学习：综述。*Wiley 跨学科评论：数据挖掘与知识发现*，9(4)：e1306，2019。

+   Hazell (2023) Hazell, J. 大型语言模型可有效扩展鱼叉式钓鱼攻击活动。*arXiv 预印本 arXiv:2305.06972*，2023。

+   Hill & Swinhoe (2022) Hill, M. 和 Swinhoe, D. 21世纪15大数据泄露事件。2022年。网址 [https://www.csoonline.com/article/534628/the-biggest-data-breaches-of-the-21st-century.html](https://www.csoonline.com/article/534628/the-biggest-data-breaches-of-the-21st-century.html)。

+   Jang-Jaccard & Nepal (2014) Jang-Jaccard, J. 和 Nepal, S. 网络安全新兴威胁调查。*计算机与系统科学杂志*，80(5)：973–993，2014。

+   Jiang 等 (2023) Jiang, A. Q., Sablayrolles, A., Mensch, A., Bamford, C., Chaplot, D. S., Casas, D. d. l., Bressand, F., Lengyel, G., Lample, G., Saulnier, L. 等。《Mistral 7b》。*arXiv 预印本 arXiv:2310.06825*，2023年。

+   Jiang 等 (2024) Jiang, A. Q., Sablayrolles, A., Roux, A., Mensch, A., Savary, B., Bamford, C., Chaplot, D. S., Casas, D. d. l., Hanna, E. B., Bressand, F. 等。《Mixtral 专家模型》。*arXiv 预印本 arXiv:2401.04088*，2024年。

+   Kang 等 (2023) Kang, D., Li, X., Stoica, I., Guestrin, C., Zaharia, M. 和 Hashimoto, T. 《利用大语言模型的程序行为：通过标准安全攻击的双重用途》。*arXiv 预印本 arXiv:2302.05733*，2023年。

+   LangChain (2023) LangChain. LangChain, 2023年。网址 [https://www.langchain.com/](https://www.langchain.com/)。

+   Lewis 等 (2020) Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., Küttler, H., Lewis, M., Yih, W.-t., Rocktäschel, T. 等。《用于知识密集型自然语言处理任务的检索增强生成》。*神经信息处理系统进展*，33:9459–9474，2020年。

+   Lohn & Jackson (2022) Lohn, A. 和 Jackson, K. 《人工智能是制造网络剑还是盾？》2022年。

+   Mialon 等 (2023) Mialon, G., Dessì, R., Lomeli, M., Nalmpantis, C., Pasunuru, R., Raileanu, R., Rozière, B., Schick, T., Dwivedi-Yu, J., Celikyilmaz, A. 等。《增强型语言模型：一项综述》。*arXiv 预印本 arXiv:2302.07842*，2023年。

+   Oladimeji & Sean (2023) Oladimeji, S. 和 Sean, K. 《Solarwinds 黑客事件解释：你需要了解的一切》。2023年。网址 [https://www.techtarget.com/whatis/feature/SolarWinds-hack-explained-Everything-you-need-to-know](https://www.techtarget.com/whatis/feature/SolarWinds-hack-explained-Everything-you-need-to-know)。

+   OpenAI (2023) OpenAI. 《在 DevDay 上宣布的新模型和开发者产品》，2023年。网址 [https://openai.com/blog/new-models-and-developer-products-announced-at-devday](https://openai.com/blog/new-models-and-developer-products-announced-at-devday)。

+   Pa Pa 等 (2023) Pa Pa, Y. M., Tanizaki, S., Kou, T., Van Eeten, M., Yoshioka, K. 和 Matsumoto, T. 《攻击者的梦想？探索 ChatGPT 在开发恶意软件中的潜力》。在 *第16届网络安全实验与测试研讨会论文集* 中，10–18页，2023年。

+   playwright (2023) playwright. 《Playwright：现代 Web 应用的快速可靠端到端测试》，2023年。网址 [https://playwright.dev/](https://playwright.dev/)。

+   Qi 等 (2023) Qi, X., Zeng, Y., Xie, T., Chen, P.-Y., Jia, R., Mittal, P. 和 Henderson, P. 《即使用户无意，微调对齐的语言模型也会危及安全！》*arXiv 预印本 arXiv:2310.03693*，2023年。

+   Regina 等 (2020) Regina, M., Meyer, M. 和 Goutal, S. 《文本数据增强：向更好的鱼叉式钓鱼邮件检测迈进》。*arXiv 预印本 arXiv:2007.02033*，2020年。

+   Research (2024) Research, N. 《Nous Hermes 2 - Yi-34b》，2024年。网址 [https://huggingface.co/NousResearch/Nous-Hermes-2-Yi-34B](https://huggingface.co/NousResearch/Nous-Hermes-2-Yi-34B)。

+   Satter & Bing (2023) Satter, R. 和 Bing, C. 美国官员查封敲诈网站；勒索病毒黑客誓言进行更多攻击。2023年。网址 [https://www.reuters.com/technology/cybersecurity/us-officials-say-they-are-helping-victims-blackcat-ransomware-gang-2023-12-19/](https://www.reuters.com/technology/cybersecurity/us-officials-say-they-are-helping-victims-blackcat-ransomware-gang-2023-12-19/)。

+   Schick 等人 (2023) Schick, T., Dwivedi-Yu, J., Dessì, R., Raileanu, R., Lomeli, M., Zettlemoyer, L., Cancedda, N., 和 Scialom, T. Toolformer：语言模型能够自我学习使用工具。*arXiv 预印本 arXiv:2302.04761*，2023。

+   Seymour & Tully (2018) Seymour, J. 和 Tully, P. 用于社交媒体上的鱼叉式网络钓鱼帖子的生成模型。*arXiv 预印本 arXiv:1802.05196*，2018。

+   Shinn 等人 (2023) Shinn, N., Cassano, F., Gopinath, A., Narasimhan, K. R., 和 Yao, S. Reflexion：带有语言强化学习的语言代理。在*第三十七届神经信息处理系统会议*，2023。

+   Sikorski & Honig (2012) Sikorski, M. 和 Honig, A. *实用恶意软件分析：分解恶意软件的实用指南*。No Starch Press，2012年。

+   Teknium (2024) Teknium. Openhermes 2.5 - Mistral 7B，2024年。网址 [https://huggingface.co/teknium/OpenHermes-2.5-Mistral-7B](https://huggingface.co/teknium/OpenHermes-2.5-Mistral-7B)。

+   Touvron 等人 (2023) Touvron, H., Martin, L., Stone, K., Albert, P., Almahairi, A., Babaei, Y., Bashlykov, N., Batra, S., Bhargava, P., Bhosale, S., 等人。Llama 2：开源基础模型和微调聊天模型。*arXiv 预印本 arXiv:2307.09288*，2023。

+   Varshney (2023) Varshney, T. LLM 代理介绍。2023年。网址 [https://developer.nvidia.com/blog/introduction-to-llm-agents/](https://developer.nvidia.com/blog/introduction-to-llm-agents/)。

+   Wang 等人 (2023a) Wang, G., Cheng, S., Zhan, X., Li, X., Song, S., 和 Liu, Y. Openchat：通过混合质量数据推动开源语言模型的发展。*arXiv 预印本 arXiv:2309.11235*，2023a。

+   Wang 等人 (2023b) Wang, X., Wang, Z., Liu, J., Chen, Y., Yuan, L., Peng, H., 和 Ji, H. Mint：在多轮互动中评估 LLMs 与工具和语言反馈的结合。*arXiv 预印本 arXiv:2309.10691*，2023b。

+   Wei 等人 (2022a) Wei, J., Tay, Y., Bommasani, R., Raffel, C., Zoph, B., Borgeaud, S., Yogatama, D., Bosma, M., Zhou, D., Metzler, D., 等人。大规模语言模型的突现能力。*arXiv 预印本 arXiv:2206.07682*，2022a。

+   Wei 等人 (2022b) Wei, J., Wang, X., Schuurmans, D., Bosma, M., Xia, F., Chi, E., Le, Q. V., Zhou, D., 等人。链式思维提示引发大规模语言模型的推理。*神经信息处理系统进展*，35:24824–24837，2022b。

+   Weng (2023) Weng, L. LLM 驱动的自主代理，2023年。网址 [https://lilianweng.github.io/posts/2023-06-23-agent/](https://lilianweng.github.io/posts/2023-06-23-agent/)。

+   席等人（2023）席智、陈文、郭鑫、何伟、丁怡、洪波、张明、王杰、金晟、周恩等人。大语言模型代理的崛起与潜力：一项调查。*arXiv 预印本 arXiv:2309.07864*，2023年。

+   杨等人（2023）杨晓、王欣、张乾、佩佐尔德、王文逸、赵鑫、林丹。影子对齐：轻松颠覆安全对齐语言模型的方式。*arXiv 预印本 arXiv:2310.02949*，2023年。

+   姚等人（2022）姚晟、赵剑、余东、杜宁、沙弗兰、纳拉辛汉、曹阳。React：在语言模型中协同推理与行动。*arXiv 预印本 arXiv:2210.03629*，2022年。

+   詹等人（2023）詹群、方荣、宾度、古普塔、桥本、康丹。通过微调去除GPT-4中的RLHF保护机制。*arXiv 预印本 arXiv:2311.05553*，2023年。

+   郑等人（2023）郑琳、蒋文龙、盛阳、庄盛、吴泽、庄阳、林智、李泽、李丹、邢恩鹏、张辉、冈萨雷斯、斯托伊卡。通过mt-bench与聊天机器人竞技场评判LLM作为法官，2023年。

+   邹等人（2023）邹安、王志、科尔特、弗雷德里克森。普适且可转移的对齐语言模型对抗攻击。*arXiv 预印本 arXiv:2307.15043*，2023年。
