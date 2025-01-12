<!--yml

类别：未分类

日期：2025-01-11 12:43:09

-->

# AI2Apps：一种构建基于LLM的AI代理应用的可视化IDE

> 来源：[https://arxiv.org/html/2404.04902/](https://arxiv.org/html/2404.04902/)

Pang Xin^(1†)，Zhucong Li^(2,4†)，Jiaxiang Chen^(2,4)，Yuan Cheng^(2,3)，Yinghui Xu²，Yuan Qi^(2,3)

${}^{1}$ ContinuAI ${}^{2}$ 人工智能创新与孵化研究所，

复旦大学，上海，中国

${}^{3}$ 上海人工智能科学院，上海，中国

${}^{4}$ 复旦大学计算机科学与技术学院，上海，中国

pxavdpro@gmail.com，{zcli22, jiaxiangchen23}@m.fudan.edu.cn

{cheng_yuan, xuyinghui, qiyuan}@fudan.edu.cn

###### 摘要

我们介绍了AI2Apps，一种具有全周期能力的可视化集成开发环境（Visual IDE），旨在加速开发者构建可部署的基于LLM的AI代理应用。该可视化IDE优先考虑其开发工具的完整性和组件的可视化性，确保顺畅高效的构建体验。一方面，AI2Apps集成了一个全面的开发工具包——从原型画布、AI辅助的代码编辑器，到代理调试器、管理系统和部署工具——所有这些都在基于网页的图形用户界面中提供。另一方面，AI2Apps将可重用的前端和后端代码可视化为直观的拖放组件。此外，一个名为AI2Apps扩展（AAE）的插件系统被设计用于扩展性，展示了一个包含20个组件的新插件如何使Web代理模拟类人浏览行为。我们的案例研究展示了显著的效率提升，AI2Apps在调试一个特定的复杂多模态代理时，分别减少了约90%和80%的令牌消耗和API调用。AI2Apps，包括一个在线演示¹¹1[https://www.ai2apps.com](https://www.ai2apps.com)，开源代码²²2[https://github.com/Avdpro/ai2apps](https://github.com/Avdpro/ai2apps)，以及一个屏幕录像视频³³3[https://youtu.be/tQTqxk1LzzU](https://youtu.be/tQTqxk1LzzU)，现在公开访问。

²²脚注：通讯作者。

## 1 引言

大型语言模型（LLMs）的出现显著推动了AI代理技术的发展，为下一代应用铺平了道路。然而，开发人员迫切需要一个全面的全栈解决方案，以减轻他们面临的重复性任务和不断增加的成本。尽管取得了显著成功，但现有的基于LLM的AI代理应用开发仍面临各自的局限性。LLM操作（LLMOps）平台通常缺乏与面向专业开发人员的工程级工具的集成，从而限制了编程和调试的灵活性（Openai, [2023](https://arxiv.org/html/2404.04902v1#bib.bib27); LangChain, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib15); Microsoft, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib19); Baidubce, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib2); ByteDance, [2023](https://arxiv.org/html/2404.04902v1#bib.bib7); Gao et al., [2024](https://arxiv.org/html/2404.04902v1#bib.bib11); Xie et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib34); LangGenius, [2023](https://arxiv.org/html/2404.04902v1#bib.bib16); Logspace, [2023](https://arxiv.org/html/2404.04902v1#bib.bib18); FlowiseAI, [2023](https://arxiv.org/html/2404.04902v1#bib.bib10); Dataelement, [2023](https://arxiv.org/html/2404.04902v1#bib.bib9))。集成开发环境（IDEs）在提供足够的可重用可视化组件方面表现不足，且在整个开发过程中仍然笨重且耗时。（Microsoft, [2023c](https://arxiv.org/html/2404.04902v1#bib.bib21), [e](https://arxiv.org/html/2404.04902v1#bib.bib23)）。软件开发工具包（SDKs），作为代理框架的基础，通常集成在LLMOps中或通过IDEs使用（Microsoft, [2023d](https://arxiv.org/html/2404.04902v1#bib.bib22); AutoGPT, [2023](https://arxiv.org/html/2404.04902v1#bib.bib1); LangChain, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib14); Wu et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib32); Li et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib17); Chen et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib8); Baidubce, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib3); Nakajima, [2023](https://arxiv.org/html/2404.04902v1#bib.bib25); Microsoft, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib20); Hong et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib12))。

![请参阅说明](img/781258f598c811d77131829ac8ed97b6.png)

图1：概述了AI2Apps与现有基于LLM的AI代理应用构建工作的比较，纵轴表示开发工具的完整性，横轴表示组件的可视化程度。

针对上述不足，我们推出了AI2Apps，一款具备全周期功能的视觉集成开发环境（Visual IDE），它赋能开发者高效构建可部署的基于LLM的AI代理应用。根据我们所知，AI2Apps是第一个实现工程级完整性和全栈可视化的基于LLM的AI代理应用开发环境，如图[1](https://arxiv.org/html/2404.04902v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AI2Apps: A Visual IDE for Building LLM-based AI Agent Applications")所示。它的优势体现在：

1.  1.

    完整性。AI2Apps通过在基于网页的图形用户界面（GUI）中提供无缝集成的开发工具包，达成了工程级的完整性，工具包涵盖从原型画布、AI辅助代码编辑器到代理调试器、管理系统和部署工具等一系列工具。在设计模式下，开发者可以通过拖放组件在原型画布中快速设计代理。在代码模式下，AI2Apps配备了AI辅助代码编辑器，旨在帮助开发者更快、更一致地编写代理应用代码。管理系统会自动保持原型画布和代码编辑器之间的双向同步，极大提高了编程效率。然后，代理调试器提供基于拓扑的调试功能，包括断点、单步运行、跟踪和GPT模拟等。这些功能帮助开发者迅速定位问题并优化代理性能。完成的AI代理可以通过部署工具一键打包为可部署的网页/移动应用，也可以通过几行代码将其作为AI扩展集成到现有的网站/应用中。

1.  2.

    可视化。AI2Apps通过将多维可重用的前端和后端代码表示为拖放式可视化组件，达成了全栈可视化。AI2Apps提供了面向三个维度的可重用组件：用户交互、链和流程控制。用户交互代表前端GUI小部件，便于用户与代理的交互方式。仅依赖聊天往往不足以有效地与用户互动，因此，AI2Apps提供了超过50个GUI小部件来支持代理开发，包括菜单、按钮、图表等。这些小部件使得代理能够以类似现实世界应用的方式与用户互动。链表示包括LLM、提示、代码、代理和其他工具等组件的后端顺序流。流程控制使开发者能够表达应用的逻辑，包含连接器、分支、数组循环、汇总和错误处理等组件。它提供了传统文本程序代码的可视化表示。

1.  3.

    扩展性。AI2Apps 扩展（AAE）是专为 AI2Apps 设计的插件扩展系统。AAE 为开发者提供了广泛的机会，可以通过插件和可拖动组件利用开源技术来增强应用。在我们的演示视频⁴⁴4[https://youtu.be/tQTqxk1LzzU](https://youtu.be/tQTqxk1LzzU)中，我们展示了一个包含 20 个组件的新插件，如何使 Web 代理应用模拟类似人类的浏览行为。

我们进行了一项案例研究，发现效率有了显著提升。在代理调试器的帮助下，AI2Apps 能够在调试一个故事创作多模态代理应用时，分别将令牌消耗和 API 调用减少约 90% 和 80%。

我们的贡献。（1）我们设计了一个具有全周期功能的可视化 IDE，帮助开发者加速构建可部署的基于大语言模型（LLM）的 AI 代理应用。（2）我们为这个可视化 IDE 实现了一个插件扩展系统，为开发者提供了广泛的机会，通过自由利用开源技术来增强应用。

![参考说明](img/65afb33c5a49242b72b1f1ca367409e6.png)

图 2：AI2Apps 的架构。左右两侧显示的是截图。（a）原型画布利用内置组件设计拓扑图。（b）代码编辑器利用 AI 辅助，实时生成并继续编程原型画布中生成的代码。（c）代理调试器定位问题并优化代理性能。（d）部署工具发布可部署的应用。（e）插件扩展系统引入新的组件。（f）管理系统支持操作环境和资源调度。

## 2 初步研究

### 2.1 基于 LLM 的 AI 代理

AI代理是能够通过传感器感知周围环境、做出决策并通过执行器采取相应行动的人工实体。基于LLM的AI代理指的是将LLM作为这些代理的大脑或控制器的主要组件，并通过多模态感知和工具利用等策略扩展其感知和行动空间（Xi等， [2023](https://arxiv.org/html/2404.04902v1#bib.bib33)；Nakano等， [2021](https://arxiv.org/html/2404.04902v1#bib.bib26)；Yao等， [2022](https://arxiv.org/html/2404.04902v1#bib.bib35)；Schick等， [2024](https://arxiv.org/html/2404.04902v1#bib.bib29)；Wei等， [2022](https://arxiv.org/html/2404.04902v1#bib.bib31)；Kojima等， [2022](https://arxiv.org/html/2404.04902v1#bib.bib13)；Wang等， [2023](https://arxiv.org/html/2404.04902v1#bib.bib30)）他们已经被应用于各种现实场景，例如软件开发（Li等， [2023](https://arxiv.org/html/2404.04902v1#bib.bib17)；Qian等， [2023](https://arxiv.org/html/2404.04902v1#bib.bib28)；Hong等， [2023](https://arxiv.org/html/2404.04902v1#bib.bib12)）和科学研究（Boiko等， [2023a](https://arxiv.org/html/2404.04902v1#bib.bib4)；Bran等， [2023](https://arxiv.org/html/2404.04902v1#bib.bib6)；Boiko等， [2023b](https://arxiv.org/html/2404.04902v1#bib.bib5)）。

### 2.2 可视化IDE

IDE是通过源代码编辑器、构建自动化工具和调试器等功能来增强软件开发的软件套件。如图[1](https://arxiv.org/html/2404.04902v1#S1.F1 "图 1 ‣ 1 介绍 ‣ AI2Apps: 一种用于构建基于LLM的AI代理应用程序的可视化IDE")所示，VScode（Microsoft， [2023f](https://arxiv.org/html/2404.04902v1#bib.bib24)）确实是一款广泛使用的IDE软件。

可视化IDE超越了传统的基于文本的IDE，通过增强效率、改善代码理解和促进协作，提供了用户友好的GUI、快速原型制作和集成开发资源。之前提到的另外两个容易混淆的概念是：SDK通常表现为应用程序编程接口（API）或软件框架，包含设备上的库和可重用的函数，用于与特定编程语言进行接口。LLMOps平台是专门设计的工具，用于简化LLM的部署、管理和扩展。这些平台对于将LLM用于聊天机器人、内容生成、数据分析等应用的企业和开发者至关重要。

## 3 AI2Apps框架

AI2Apps的设计兼顾了其开发工具的完整性和组件的可视化，确保了顺畅高效的构建体验。它可以分为六个模块，如图[2](https://arxiv.org/html/2404.04902v1#S1.F2 "图 2 ‣ 1 介绍 ‣ AI2Apps: 一种用于构建基于LLM的AI代理应用程序的可视化IDE")所示。

### 3.1 原型画布

原型设计在应用开发中起着至关重要的作用，它通过创建交互式模型来验证功能和用户体验，从而降低成本，确保交付满足用户需求的成功应用。在设计模式下，开发者可以通过拖放组件在原型画布上快速设计代理逻辑。其特点包括：

基于拓扑。原型画布以拓扑图的形式表示应用逻辑，从而将所有耦合的代码拆解为清晰可靠的单元。与逐行阅读复杂代码相比，包含拓扑图的AI代理代码在后期维护中的优势巨大。它不仅提供了对原始代码设计理念的清晰理解，还能更快地识别代码问题。

视觉组件。原型画布将面向三个维度的组件可视化：用户交互、链条和流程控制。用户交互代表前端GUI小部件，便于用户与代理的交互方式。单靠聊天往往无法高效地与用户互动。因此，AI2Apps提供了超过50种GUI小部件来支持代理开发，包括菜单、按钮、图表等。这些小部件使代理能够以类似于现实应用的方式与用户进行交互。链条代表包含LLM、提示、代码、代理和其他工具的后端顺序流，作为视觉组件展示。流程控制使开发者能够表达应用的逻辑，包含连接器、分支、数组循环、汇总和错误处理等组件。开发者可以轻松地添加和配置这些组件，而无需手动编写大量代码。这不仅提高了开发效率，还减少了出错的可能性。

代码同步。传统开发方法难以保持初始设计和代码实现之间的实时同步，这使得设计文档在维护阶段作为参考的效果大打折扣。AI2Apps的创新“设计与编码同步”模式确保设计和实现之间始终保持一致，有效消除了设计愿景和实际代码之间的任何差异。

原型设计画布代码编辑器代理调试器部署工具名称拓扑组件代码同步AI助手多语言断点步骤运行追踪代码应用可视化集成开发环境：AI2Apps（我们的）${}^{*}$ ✓ ✓ 双向 ✓ ✓ ✓ ✓ ✓ ✓ 开放集成开发环境：提示流与VScode（微软，[2023c](https://arxiv.org/html/2404.04902v1#bib.bib21)） ✓ ✗ 双向 ✗ ✗ ✓ ✓ ✓ ✓ 开放语义内核与VScode（微软，[2023e](https://arxiv.org/html/2404.04902v1#bib.bib23)） ✗ ✗ ✗ ✗ ✓ ✓ ✓ ✗ ✓ 开放LLMOps平台：GPTs（Openai，[2023](https://arxiv.org/html/2404.04902v1#bib.bib27)） ✗ ✓ ✗ ✗ ✗ ✗ ✗ ✗ ✗ 闭源LangSmith（LangChain，[2023b](https://arxiv.org/html/2404.04902v1#bib.bib15)） ✗ ✓ ✗ ✗ ✓ ✓ ✓ ✓ ✓ 闭源Autogen Studio（微软，[2023a](https://arxiv.org/html/2404.04902v1#bib.bib19)） ✗ ✓ ✗ ✗ ✗ ✗ ✗ ✗ ✓ 闭源Appbuilder（Baidubce，[2023a](https://arxiv.org/html/2404.04902v1#bib.bib2)） ✗ ✓ ✗ ✗ ✓ ✗ ✗ ✗ ✓ 闭源Coze（字节跳动，[2023](https://arxiv.org/html/2404.04902v1#bib.bib7)） ✓ ✓ ✗ ✗ ✗ ✗ ✗ ✗ ✗ 闭源AgentScope${}^{*}$（高等，[2024](https://arxiv.org/html/2404.04902v1#bib.bib11)） ✗ ✓ ✗ ✗ ✗ ✗ ✗ ✗ ✓ 闭源OpenAgent${}^{*}$（谢等，[2023](https://arxiv.org/html/2404.04902v1#bib.bib34)） ✗ ✓ ✗ ✗ ✗ ✗ ✗ ✗ ✗ 闭源Dify${}^{*}$（LangGenius，[2023](https://arxiv.org/html/2404.04902v1#bib.bib16)） ✗ ✓ 单向 ✗ ✓ ✗ ✗ ✗ ✓ 闭源Langflow${}^{*}$（Logspace，[2023](https://arxiv.org/html/2404.04902v1#bib.bib18)） ✓ ✓ 单向 ✗ ✗ ✗ ✗ ✗ ✓ 闭源Flowise${}^{*}$（FlowiseAI，[2023](https://arxiv.org/html/2404.04902v1#bib.bib10)） ✓ ✓ 单向 ✗ ✗ ✗ ✗ ✗ ✓ 闭源Bisheng${}^{*}$（Dataelement，[2023](https://arxiv.org/html/2404.04902v1#bib.bib9)） ✓ ✓ 单向 ✗ ✗ ✗ ✗ ✗ ✓ 闭源SDK：提示流${}^{*}$（微软，[2023b](https://arxiv.org/html/2404.04902v1#bib.bib20)） ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ 语义内核${}^{*}$（微软，[2023d](https://arxiv.org/html/2404.04902v1#bib.bib22)） ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ AutoGPT${}^{*}$（AutoGPT，[2023](https://arxiv.org/html/2404.04902v1#bib.bib1)） ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ BabyAGI${}^{*}$（中岛，[2023](https://arxiv.org/html/2404.04902v1#bib.bib25)） ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ LangChain${}^{*}$（LangChain，[2023a](https://arxiv.org/html/2404.04902v1#bib.bib14)） ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ Autogen${}^{*}$（吴等，[2023](https://arxiv.org/html/2404.04902v1#bib.bib32)） ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ MetaGPT${}^{*}$（洪等，[2023](https://arxiv.org/html/2404.04902v1#bib.bib12)） ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ ✗ AppBuilder-SDK${}^{*}$（Baidubce，[2023b](https://arxiv.org/html/2404.04902v1#bib.bib3)） ✗ ✓ ✗ ✗ ✓ ✗ ✗ ✗ ✓ ✗ Camel${}^{*}$（李等，[2023](https://arxiv.org/html/2404.04902v1#bib.bib17)） ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ 闭源AgentVerse${}^{*}$（陈等，[2023](https://arxiv.org/html/2404.04902v1#bib.bib8)） ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✗ ✓ 闭源

表 1：通过开发工具的完整性，比较AI2Apps与当前现有工作的差异。 “${}^{*}$”表示该项目已开源。 “双向”表示原型画布与代码编辑器之间的实时双向同步。 “单向”表示只有单向同步。 “开放”表示生成的应用程序属于用户个人版权，可以自由使用。 “封闭”表示生成的应用程序只能在平台内部署或封装为API。所有表格中的统计数据截至2024年3月收集。

| 名称 | 用户互动 | 链接 | 流程控制 |
| --- | --- | --- | --- |
| 视觉IDE: |  |  |  |
| AI2Apps (我们的)${}^{*}$ | ✓ | ✓ | ✓ |
| IDE: |  |  |  |
| PromptFlow w/ VScode (微软, [2023c](https://arxiv.org/html/2404.04902v1#bib.bib21)) | ✗ | ✓ | ✗ |
| Semantic Kernel w/ VScode (微软, [2023e](https://arxiv.org/html/2404.04902v1#bib.bib23)) | ✗ | ✗ | ✗ |
| LLMOps 平台: |  |  |  |
| GPTs (OpenAI, [2023](https://arxiv.org/html/2404.04902v1#bib.bib27)) | ✗ | 有限 | ✗ |
| LangSmith (LangChain, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib15)) | ✗ | 有限 | ✗ |
| Autogen Studio (微软, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib19)) | ✗ | 有限 | ✗ |
| Appbuilder (百度云, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib2)) | ✗ | 有限 | ✗ |
| Coze (字节跳动, [2023](https://arxiv.org/html/2404.04902v1#bib.bib7)) | ✗ | ✓ | ✗ |
| AgentScope${}^{*}$ (Gao 等人, [2024](https://arxiv.org/html/2404.04902v1#bib.bib11)) | ✗ | 有限 | ✗ |
| OpenAgent${}^{*}$ (Xie 等人, [2023](https://arxiv.org/html/2404.04902v1#bib.bib34)) | ✗ | 有限 | ✗ |
| Dify${}^{*}$ (LangGenius, [2023](https://arxiv.org/html/2404.04902v1#bib.bib16)) | ✗ | ✓ | ✗ |
| Langflow${}^{*}$ (Logspace, [2023](https://arxiv.org/html/2404.04902v1#bib.bib18)) | ✗ | ✓ | ✗ |
| Flowise${}^{*}$ (FlowiseAI, [2023](https://arxiv.org/html/2404.04902v1#bib.bib10)) | ✗ | ✓ | ✗ |
| Bisheng${}^{*}$ (Dataelement, [2023](https://arxiv.org/html/2404.04902v1#bib.bib9)) | ✗ | ✓ | ✗ |
| SDK: |  |  |  |
| Prompt flow${}^{*}$ (微软, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib20)) | ✗ | ✗ | ✗ |
| Semantic Kernel${}^{*}$ (微软, [2023d](https://arxiv.org/html/2404.04902v1#bib.bib22)) | ✗ | ✗ | ✗ |
| AutoGPT${}^{*}$ (AutoGPT, [2023](https://arxiv.org/html/2404.04902v1#bib.bib1)) | ✗ | ✗ | ✗ |
| BabyAGI${}^{*}$ (Nakajima, [2023](https://arxiv.org/html/2404.04902v1#bib.bib25)) | ✗ | ✗ | ✗ |
| LangChain${}^{*}$ (LangChain, [2023a](https://arxiv.org/html/2404.04902v1#bib.bib14)) | ✗ | ✗ | ✗ |
| Autogen${}^{*}$ (Wu 等人, [2023](https://arxiv.org/html/2404.04902v1#bib.bib32)) | ✗ | ✗ | ✗ |
| MetaGPT${}^{*}$ (Hong 等人, [2023](https://arxiv.org/html/2404.04902v1#bib.bib12)) | ✗ | ✗ | ✗ |
| AppBuilder-SDK${}^{*}$ (百度云, [2023b](https://arxiv.org/html/2404.04902v1#bib.bib3)) | ✗ | ✗ | ✗ |
| Camel${}^{*}$ (Li et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib17)) | ✗ | ✗ | ✗ |
| AgentVerse${}^{*}$ (Chen et al., [2023](https://arxiv.org/html/2404.04902v1#bib.bib8)) | ✗ | ✗ | ✗ |

表2：通过组件的可视化比较AI2Apps与当前现有工作。“${}^{*}$”表示该项目已开源。“Limited”表示链条无法在直观的拓扑图中可视化。表中的所有统计数据均由2024年3月收集。

### 3.2 代码编辑器

代码编辑器是开发者用于编写和编辑软件开发项目代码的软件工具。在代码模式下，AI2Apps具备AI辅助代码编辑器，旨在帮助开发者编写高质量的应用代码。其特点包括：

AI Copilot。我们的代码编辑器配备了AI助手，能够根据上下文在光标位置生成后续代码，或直接重写整个文档。我们允许用户自由修改AI助手的拓扑结构以增强其功能，或者用户可以通过API集成外部AI助手。

AI UI Creator。AI UI创建器通过对话界面生成用户所需的标准化UI代码，利用50多个GUI小部件，并在原型画布中显示出来。

多语言。代码编辑器支持任何形式的编程语言，包括JavaScript、Python等，允许来自不同编程背景的开发者自由地集成自己的代码。

### 3.3 代理调试器

代理调试器的设计理念灵感来源于通用集成开发环境（IDE）中的调试功能。与传统的IDE专注于逐行调试代码不同，代理调试器特别针对跟踪拓扑图的轨迹进行定制。其特点包括：

断点。设置断点可以在拓扑图中的特定位置暂停执行。这样，开发者可以检查程序在特定时间点的状态，包括变量值、内存状态以及程序的执行路径。设置断点是一种有效的故障排除方法，尤其在处理复杂错误和性能问题时。

步骤运行。步骤运行功能允许开发者逐节点执行拓扑图，或跳入特定代理的子图，便于更详细地检查执行流和各时间点的状态。这种方法特别有助于理解拓扑图的执行路径、识别逻辑错误以及检查变量变化。

跟踪功能。跟踪功能在拓扑图上直观地呈现数据变量的流动轨迹。用户可以保存并下载以JSON格式记录的跟踪日志，供后续分析代理使用。

GPT 模拟。在调用外部LLM API的阶段，用户可以设置GPT模拟。当满足预定义条件时，API调用将不再通过网络，而是直接返回由GPT模拟设置的结果。这种方式可以显著降低token成本并提高调试效率。

### 3.4 部署工具

大多数现有主流开发平台开发的应用程序严重依赖平台本身的运行时环境，这阻碍了AI代理应用程序的开发。然而，作为一个可视化IDE，AI2Apps集成了多个部署工具，并支持用户直接构建外部可部署的应用程序。完成的AI代理可以打包为独立的Web/移动应用程序。它们也可以作为AI扩展集成到现有的网站/应用程序中，只需几行代码。

### 3.5 插件扩展系统

AI2Apps扩展（AAE）是一个专门为AI2Apps设计的插件扩展系统。AAE为开发者提供了丰富的机会，通过利用开放技术作为插件和可拖拽组件来增强应用程序。AAE具有可重用的组件，允许开发者将现有组件组合或创建新组件。开发者可以通过将自定义组件发布为包来共享，从而通过AAE系统拓展AI2Apps的功能。

在我们的录像视频⁵⁵5[https://youtu.be/tQTqxk1LzzU](https://youtu.be/tQTqxk1LzzU)中，我们展示了一个包含20个组件的新插件如何使Web代理应用程序模拟类似人类的浏览行为。通过Web扩展组件，AI2Apps对网页进行全面控制，能够执行如打开/切换页面、读取页面内容、填写/修改页面内容、模拟用户行为等操作。

### 3.6 管理系统

管理系统为AI2Apps提供了各种底层功能支持，包括：

Web-OS系统。基于Web API技术，它为AI2Apps提供了一整套桌面操作系统功能。例如文件系统、多任务支持、网络调用等。它为用户提供了一种与原生操作系统相当的开发体验，同时更加方便和安全。

运行时管理器。它为面向AI的应用程序提供专门的运行时基础功能，包括：应用程序管理/调度、插件扩展/控制、AI功能组件的集成等。

包管理器。它通过包扩展了AI2Apps的功能，这些包包括构建、发布、共享包的服务，以及安装和升级服务。

## 4 案例研究与应用

![参见说明](img/ec5c11fd04649c7f46ec32a0fa4557d5.png)

图3：我们通过AI2Apps构建的使用助手截图。

### 4.1 案例研究

我们进行了案例研究，并发现了显著的效率提升。在这个案例中，我们最初使用AI2Apps构建了一个复杂的代理应用程序，用于创作短篇小说。它允许指定绘画风格、精确的无限长度写作，支持包括封面和插图的功能，并具有强大的文章编辑能力。它支持逐段修改，包括内容和图片，并提供选择是否在编辑过程中使用AI辅助的选项。我们通过清除代理的所有提示，进行了四组对照实验，邀请了八名志愿者尝试恢复原始代理的功能，其中一半使用代理调试器，另一半没有使用。借助代理调试器，AI2Apps在调试一个故事写作多模态代理应用时，分别将令牌消耗和API调用减少了大约90%和80%。代理调试器的用户界面如图[3](https://arxiv.org/html/2404.04902v1#S4.F3 "Figure 3 ‣ 4 Case Study and Usage ‣ AI2Apps: A Visual IDE for Building LLM-based AI Agent Applications")所示。该代理项目可以通过[https://github.com/Avdpro/StoryWriter](https://github.com/Avdpro/StoryWriter)访问。

### 4.2 使用方法

鉴于AI2Apps的广泛功能，本文在其有限的篇幅内无法提供详细的使用指南。如需了解更多关于我们使用示例的信息，请参考[https://github.com/Avdpro/ai2apps](https://github.com/Avdpro/ai2apps)。我们已经使用AI2Apps构建了一个使用助手，如图[3](https://arxiv.org/html/2404.04902v1#S4.F3 "Figure 3 ‣ 4 Case Study and Usage ‣ AI2Apps: A Visual IDE for Building LLM-based AI Agent Applications")所示，允许用户通过在[https://www.ai2apps.com](https://www.ai2apps.com)运行它即可快速上手AI2Apps。

## 5 与相关工作对比

在图[1](https://arxiv.org/html/2404.04902v1#S1.F1 "图 1 ‣ 1 引言 ‣ AI2Apps：构建基于LLM的AI代理应用的可视化IDE")、表[1](https://arxiv.org/html/2404.04902v1#S3.T1 "表 1 ‣ 3.1 原型设计画布 ‣ 3 AI2Apps框架 ‣ AI2Apps：构建基于LLM的AI代理应用的可视化IDE")和表[2](https://arxiv.org/html/2404.04902v1#S3.T2 "表 2 ‣ 3.1 原型设计画布 ‣ 3 AI2Apps框架 ‣ AI2Apps：构建基于LLM的AI代理应用的可视化IDE")中，我们从开发工具的完整性和组件的可视化两个角度，展示了现有工作的一项详细对比。据我们所知，AI2Apps是第一个实现了工程级完整性和可视化IDE全栈功能的基于LLM的AI代理应用开发环境。可视化IDE可以像传统IDE一样集成SDK，也可以集成到LLMOps平台中。因此，AI2Apps的开源发布有效填补了当前开发技术的空白，促进了AI代理应用的发展。

## 6 结论

我们介绍了AI2Apps，这是第一个具有全周期功能的可视化IDE，能够帮助开发者高效构建可部署的基于LLM的AI代理应用。AI2Apps提供了工程级开发工具和全栈可视化组件。它具有基于Web的界面，提供如原型设计画布、AI驱动的代码编辑器、代理调试器、管理系统和部署工具等工具，同时还包括直观的拖放组件，用于代码可视化。此外，我们为这个可视化IDE实现了插件扩展系统，为开发者提供了广泛的机会，通过自由利用开放技术来增强应用功能。我们进行了一项案例研究，发现显著的效率提升。在调试一个故事写作的多模态代理应用时，借助代理调试器，AI2Apps能够分别减少约90%和80%的token消耗和API调用。

## 限制

作为一款可视化IDE，AI2Apps仍无法完全匹配传统IDE在应用开发中的灵活性，也无法提供LLMOps平台对于已部署应用的操作能力。然而，它在开发便捷性方面的优势是值得注意的。我们热烈欢迎LLMOps平台集成这项工作，以增强他们的产品，从而共同促进相关学术研究。

## 伦理声明

(1) 该材料是作者们的原创作品，至今在项目开发阶段未曾在其他地方发表过。 (2) 该论文目前未在其他地方考虑发表。 (3) 论文真实、完整地反映了作者们的研究和分析。 (4) 我们的工作不包含身份特征，不会伤害任何人。案例研究部分的八名参与者是从工程专业的学生中招募的志愿者。在案例研究实验之前，所有参与者都已获得书面和口头的详细指导。唯一记录的用户相关信息是用户名，这些信息被匿名化并作为标识符用于标记不同的参与者。 (5) AI2Apps 旨在帮助用户构建可部署的 AI 智能体应用。 (6) 我们的工作不涉及 LLM 的训练或微调；我们仅使用经过研究许可的公开可用 API。因此，我们的方法不存在与数据相关的风险。

## 参考文献

+   AutoGPT (2023) AutoGPT. 2023. Autogpt. [https://github.com/Significant-Gravitas/AutoGPT](https://github.com/Significant-Gravitas/AutoGPT)。

+   Baidubce (2023a) Baidubce. 2023a. Appbuilder. [https://cloud.baidu.com/product/AppBuilder](https://cloud.baidu.com/product/AppBuilder)。

+   Baidubce (2023b) Baidubce. 2023b. Appbuilder-sdk. [https://github.com/baidubce/app-builder](https://github.com/baidubce/app-builder)。

+   Boiko 等人 (2023a) Daniil A Boiko, Robert MacKnight, 和 Gabe Gomes. 2023a. 大型语言模型的涌现自主科学研究能力。*arXiv 预印本 arXiv:2304.05332*。

+   Boiko 等人 (2023b) Daniil A Boiko, Robert MacKnight, Ben Kline, 和 Gabe Gomes. 2023b. 使用大型语言模型进行自主化学研究。*Nature*, 624(7992):570–578。

+   Bran 等人 (2023) Andres M Bran, Sam Cox, Oliver Schilter, Carlo Baldassari, Andrew White, 和 Philippe Schwaller. 2023. 用化学工具增强大型语言模型。在 *NeurIPS 2023 AI for Science Workshop*。

+   ByteDance (2023) ByteDance. 2023. Coze：下一代 AI 聊天机器人开发平台。 [https://www.coze.com/](https://www.coze.com/)。

+   Chen 等人 (2023) Weize Chen, Yusheng Su, Jingwei Zuo, Cheng Yang, Chenfei Yuan, Chen Qian, Chi-Min Chan, Yujia Qin, Yaxi Lu, Ruobing Xie 等人. 2023. Agentverse：促进多智能体协作并探索智能体中的涌现行为。*arXiv 预印本 arXiv:2308.10848*。

+   Dataelement (2023) Dataelement. 2023. Bisheng. [https://github.com/dataelement/bisheng](https://github.com/dataelement/bisheng)。

+   FlowiseAI (2023) FlowiseAI. 2023. Flowise. [https://github.com/FlowiseAI/Flowise](https://github.com/FlowiseAI/Flowise)。

+   Gao 等人 (2024) Dawei Gao, Zitao Li, Weirui Kuang, Xuchen Pan, Daoyuan Chen, Zhijian Ma, Bingchen Qian, Liuyi Yao, Lin Zhu, Chen Cheng 等人. 2024. Agentscope：一个灵活且稳健的多智能体平台。*arXiv 预印本 arXiv:2402.14034*。

+   Hong et al. (2023) 司睿·洪、明晨·祝戈、乔纳森·陈、晓武·郑、宇恒·程、金麟·王、策耀·张、子力·王、史蒂文·卡·兴·邱、子娟·林等. 2023. Metagpt: 用于多智能体协作框架的元编程. 载于 *第十二届国际学习表征大会*。

+   Kojima et al. (2022) 小岛武、石向·香根·顾、马切尔·里德、松尾裕、高田伊和佐. 2022. 大型语言模型是零-shot 推理器. *神经信息处理系统进展*, 35:22199–22213.

+   LangChain (2023a) LangChain. 2023a. Langchain. [https://github.com/langchain-ai/langchain](https://github.com/langchain-ai/langchain).

+   LangChain (2023b) LangChain. 2023b. Langsmith. [https://www.langchain.com/langsmith](https://www.langchain.com/langsmith).

+   LangGenius (2023) LangGenius. 2023. Dify. [https://github.com/langgenius/dify](https://github.com/langgenius/dify).

+   Li et al. (2023) 李国豪、哈桑·阿贝德·卡德尔·哈姆穆德、哈尼·伊塔尼、德米特里·赫兹布林、伯纳德·甘恩. 2023. Camel: 用于“大型语言模型社会”心智探索的交互式代理. 载于 *第37届神经信息处理系统会议*。

+   Logspace (2023) Logspace. 2023. Langflow. [https://github.com/logspace-ai/langflow](https://github.com/logspace-ai/langflow).

+   Microsoft (2023a) Microsoft. 2023a. Autogen Studio 2.0: 革新 AI 代理. [https://autogen-studio.com/](https://autogen-studio.com/).

+   Microsoft (2023b) Microsoft. 2023b. 提示流. [https://github.com/microsoft/promptflow](https://github.com/microsoft/promptflow).

+   Microsoft (2023c) Microsoft. 2023c. Visual Studio Code 的提示流. [https://marketplace.visualstudio.com/items?itemName=prompt-flow.prompt-flow](https://marketplace.visualstudio.com/items?itemName=prompt-flow.prompt-flow).

+   Microsoft (2023d) Microsoft. 2023d. 语义内核. [https://github.com/microsoft/semantic-kernel](https://github.com/microsoft/semantic-kernel).

+   Microsoft (2023e) Microsoft. 2023e. Visual Studio Code 的语义内核. [https://learn.microsoft.com/en-us/semantic-kernel/vs-code-tools/](https://learn.microsoft.com/en-us/semantic-kernel/vs-code-tools/).

+   Microsoft (2023f) Microsoft. 2023f. Visual Studio Code - 开源. [https://github.com/microsoft/vscode](https://github.com/microsoft/vscode).

+   Nakajima (2023) 中岛阳平. 2023. Babyagi. [https://github.com/yoheinakajima/babyagi](https://github.com/yoheinakajima/babyagi).

+   Nakano et al. (2021) 中野礼一郎、雅各布·希尔顿、苏奇尔·巴拉吉、杰夫·吴、龙·欧阳、克里斯蒂娜·金、克里斯托弗·赫塞、尚塔努·简、维尼特·科萨拉朱、威廉·桑德斯等. 2021. Webgpt: 通过浏览器辅助的问答与人类反馈. *arXiv 预印本 arXiv:2112.09332*.

+   Openai (2023) Openai. 2023. 探索 GPTs. [https://chat.openai.com/gpts](https://chat.openai.com/gpts).

+   Qian 等人（2023）Chen Qian, Xin Cong, Cheng Yang, Weize Chen, Yusheng Su, Juyuan Xu, Zhiyuan Liu, 和 Maosong Sun。2023年。《软件开发的沟通代理》*arXiv 预印本 arXiv:2307.07924*。

+   Schick 等人（2024）Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Eric Hambro, Luke Zettlemoyer, Nicola Cancedda, 和 Thomas Scialom。2024年。《Toolformer：语言模型可以自我学习使用工具》*《神经信息处理系统进展》*，36。

+   Wang 等人（2023）Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin 等人。2023年。《基于大语言模型的自主代理调查》*arXiv 预印本 arXiv:2308.11432*。

+   Wei 等人（2022）Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou 等人。2022年。《思维链提示引发大语言模型推理》*《神经信息处理系统进展》*，35:24824–24837。

+   Wu 等人（2023）Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang, 和 Chi Wang。2023年。《Autogen：通过多代理对话框架实现下一代 LLM 应用》*arXiv 预印本 arXiv:2308.08155*。

+   Xi 等人（2023）Zhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, Boyang Hong, Ming Zhang, Junzhe Wang, Senjie Jin, Enyu Zhou 等人。2023年。《基于大语言模型的代理的崛起与潜力：一项调查》*arXiv 预印本 arXiv:2309.07864*。

+   Xie 等人（2023）Tianbao Xie, Fan Zhou, Zhoujun Cheng, Peng Shi, Luoxuan Weng, Yitao Liu, Toh Jing Hua, Junning Zhao, Qian Liu, Che Liu 等人。2023年。《Openagents：一个开放的平台，用于野外语言代理》*arXiv 预印本 arXiv:2310.10634*。

+   Yao 等人（2022）Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R Narasimhan, 和 Yuan Cao。2022年。《React：在语言模型中协同推理与行动》收录于*第十一届国际学习表示会议*。
