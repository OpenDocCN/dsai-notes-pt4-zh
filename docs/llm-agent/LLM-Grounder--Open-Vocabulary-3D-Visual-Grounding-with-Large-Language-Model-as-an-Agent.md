<!--yml

category: 未分类

日期：2025-01-11 13:06:23

-->

# LLM-Grounder：基于大语言模型作为代理的开放词汇3D视觉定位

> 来源：[https://arxiv.org/html/2309.12311/](https://arxiv.org/html/2309.12311/)

\tcbuselibrary

listings

Jianing Yang${}^{*,1}$  Xuweiyi Chen${}^{*,1}$  Shengyi Qian${}^{1}$  Nikhil Madaan${}^{2}$  Madhavan Iyengar${}^{1}$

David F. Fouhey${}^{1,3}$  Joyce Chai${}^{1}$ *平等贡献.${}^{1}$美国密歇根大学计算机科学与工程系，安娜堡，密歇根州，48109\. 联系方式：Jianing Yang jianingy@umich.edu.${}^{2}$Nikhil Madaan是独立研究人员.${}^{3}$纽约大学。本研究得到了NSF IIS-1949634、NSF SES-2128623和微软学术项目计算信用的大力支持。项目网站：[https://chat-with-nerf.github.io/](https://chat-with-nerf.github.io/)

###### 摘要

3D视觉定位是家庭机器人一项至关重要的技能，使它们能够导航、操作物体并根据环境回答问题。现有的方法通常依赖于大量的标注数据或在处理复杂语言查询时存在局限性，我们提出了LLM-Grounder，一种新颖的零-shot、开放词汇的基于大语言模型（LLM）的3D视觉定位管道。LLM-Grounder利用LLM将复杂的自然语言查询分解为语义成分，并使用视觉定位工具，如OpenScene或LERF，来识别3D场景中的物体。LLM随后评估所提议物体之间的空间和常识关系，做出最终的定位决策。我们的方法不需要任何标注的训练数据，并且能够推广到新的3D场景和任意文本查询。我们在ScanRefer基准上评估了LLM-Grounder，并展示了最先进的零-shot定位准确率。我们的研究表明，LLM显著提高了定位能力，尤其是在复杂语言查询的处理上，使得LLM-Grounder成为机器人领域中3D视觉语言任务的有效方法。

\AtBeginShipoutNext\AtBeginShipoutAddToBox{tikzpicture}

[记住图片，覆盖] \node[text=black, anchor=south, align=center, text width=0.8] at ([yshift=0.5cm]current page.south) 本研究已提交给IEEE，待可能的发表。

版权可能会在未经通知的情况下转移，转移后此版本可能无法访问。

{strip}![[无标题图片]](img/29bfa68d00682963427074c4af2e402e.png)

图 1：在开放词汇三维视觉定位任务中，基于 CLIP 的模型倾向于将文本输入视为*“词袋”*，忽略了组成文本输入的语义结构，例如物体之间复杂的空间关系。在右上方展示了使用 OpenScene [[1](#bib.bib1)]（一种基于 CLIP 的三维定位方法）作为视觉定位器时的这种行为。当被要求定位空间信息的文本查询“餐桌和窗户之间的椅子”时，它错误地高亮了餐桌和窗户，这些并非目标，而是参考性地标（红色边框）。我们提出通过利用大语言模型（LLM）来解决这个问题，1\. 有意生成一个计划，将复杂的视觉定位查询分解为子任务；2\. 协调和与工具（如目标查找器和地标查找器）互动以收集信息；3\. 利用空间和常识知识来反思从工具收集的反馈。

## I 引言

想象一下，你被放置在一个三维场景中，并被要求找到*“餐桌和窗户之间的椅子”*（图[1](#S0.F1 "图 1 ‣ LLM-Grounder: 基于大语言模型的开放词汇三维视觉定位")）。对于人类来说，找出答案是很容易的。这种技能被称为三维视觉定位，我们通常在日常任务中依赖它，从寻找物体到操作工具。掌握这种能力对于构建任何辅助人类的家庭机器人至关重要，因为它作为进行复杂导航（知道去哪里）、操作（知道抓取什么/哪里）和问答的基本技能。

为了赋予机器人这种能力，研究人员已经开发了多种方法。一种方向是训练一个3D与文本端到端的神经架构，提出围绕物体的边界框并共同建模文本-边界框匹配[[2](#bib.bib2), [3](#bib.bib3), [4](#bib.bib4), [5](#bib.bib5), [6](#bib.bib6), [7](#bib.bib7), [8](#bib.bib8), [9](#bib.bib9), [10](#bib.bib10), [11](#bib.bib11)]。然而，这类模型通常需要大量的3D-文本配对作为训练数据，而这些数据很难获得[[12](#bib.bib12), [13](#bib.bib13)]。因此，这些训练方法在新场景中往往无法获得良好的表现。最近，研究者们尝试解决开放词汇的3D视觉定位问题[[14](#bib.bib14), [15](#bib.bib15), [16](#bib.bib16), [17](#bib.bib17), [18](#bib.bib18), [19](#bib.bib19), [20](#bib.bib20), [1](#bib.bib1), [21](#bib.bib21), [22](#bib.bib22), [23](#bib.bib23)]，通常基于CLIP的优势[[24](#bib.bib24)]。然而，依赖于CLIP使得这些方法表现出“词袋”行为，即能够很好地建模无序内容，但在处理文本和视觉信息时，忽略了属性、关系和顺序[[25](#bib.bib25)]。例如，如图[1](#S0.F1 "Figure 1 ‣ LLM-Grounder: Open-Vocabulary 3D Visual Grounding with Large Language Model as an Agent")所示，当向OpenScene[[1](#bib.bib1)]提供文本查询“餐桌和窗户之间的椅子”时，模型会将房间内所有的椅子、窗户和餐桌进行定位，忽略了窗户和餐桌仅作为提供空间关系的标记物，与目标椅子无关。

与此同时，像ChatGPT和GPT-4这样的**大型语言模型**（LLMs）[[26](#bib.bib26)]已经展现出了令人印象深刻的语言理解能力，包括计划和工具使用。这些能力使得LLM能够作为代理，解决复杂任务，通过将任务拆解成更小的部分，并知道何时、如何以及使用何种工具来完成子任务[[27](#bib.bib27), [28](#bib.bib28), [29](#bib.bib29), [30](#bib.bib30), [31](#bib.bib31), [32](#bib.bib32), [33](#bib.bib33), [34](#bib.bib34)]。这正是处理复杂自然语言查询的3D视觉定位所需要的：将组成性语言解析为更小的语义单元，使用工具和与环境互动以收集反馈，并通过空间和常识推理来迭代性地将语言定位到目标物体。基于这些观察，我们提出了这个问题，

> 我们能否使用基于LLM的代理来改善零-shot开放词汇的3D视觉定位？

在本研究中，我们提出了*LLM-Grounder*，一种新颖的开放词汇、零-shot、基于LLM代理的3D视觉定位管道。我们的直觉是，LLM可以通过将语言分解、空间和常识推理任务交给LLM本身来缓解基于CLIP的视觉定位器的“词袋”弱点，同时利用视觉定位器的优势来定位简单的名词短语。如在第[III](#S3 "III Method ‣ LLM-Grounder: Open-Vocabulary 3D Visual Grounding with Large Language Model as an Agent")节所述，LLM-Grounder将LLM作为核心来协调定位过程。LLM首先将组成的自然语言查询解析为语义概念，如物体类别、物体属性（颜色、形状、材质）、地标和空间关系。这些子查询被传递给一个由OpenScene [[1](#bib.bib1)] 或 LERF [[35](#bib.bib35)]支持的视觉定位*工具*，这些是基于CLIP [[24](#bib.bib24)]的开放词汇3D视觉定位方法，用于在场景中定位每个概念。视觉定位器会在场景中最相关的候选区域周围提出一些边界框。对于每个候选区域，视觉定位器工具计算并提供空间信息，如物体体积和到地标的距离，返回给LLM代理，以便代理能够从空间关系和常识的角度全面评估情况，并选择最符合原始查询所有标准的候选区域。这个过程会不断重复，直到LLM代理决定它已经得出结论。值得注意的是，我们的方法通过向代理提供环境反馈并使代理的推理过程闭环，扩展了之前的神经符号方法[[36](#bib.bib36)]。

值得注意的是，我们的方法不需要任何标注数据的训练。它是开放词汇的，并且能够在零-shot情况下推广到新的3D场景和任意文本查询，这在3D场景的语义多样性和3D-文本标注数据有限的情况下尤为重要。在我们的实验中（第[IV](#S4 "IV Experiments ‣ LLM-Grounder: Open-Vocabulary 3D Visual Grounding with Large Language Model as an Agent)节"），我们在ScanRefer基准上评估了LLM-Grounder[[12](#bib.bib12)]。该基准主要评估需要理解组合视觉参考表达式的3D视觉-语言定位能力。我们的方法提升了零-shot开放词汇方法（如OpenScene和LERF）的定位能力，并在ScanRefer上展示了SOTA的零-shot定位准确性，且未使用任何标注数据。我们的消融研究表明，随着语言查询变得更加复杂，LLM能显著提高定位能力。这些发现凸显了LLM-Grounder作为一种有效的3D视觉-语言任务方法的潜力，特别适用于机器人应用，在这些应用中，理解复杂环境并响应动态查询至关重要。

总结来说，本文的贡献如下：

+   •

    我们发现，使用大语言模型（LLM）作为代理可以提升零-shot开放词汇方法在3D视觉定位任务中的基础能力。

+   •

    在零-shot设置下，我们在ScanRefer上实现了SOTA（最先进的技术），无需使用标注数据。

+   •

    我们发现，当定位查询文本变得更加复杂时，LLM更加有效。

![参考标题](img/02e9d0cc2da6c29c88dd26133ec40854.png)

图2：LLM-Grounder概览。在接收到定位对象的查询后，我们的方法通过一个LLM代理进行推理，根据用户的请求生成一个使用工具定位对象的计划。该代理与工具进行交互，例如目标查找器和地标查找器，以收集信息，如对象的边界框、对象体积以及从工具到地标的距离。然后，这些信息被返回给代理，进行进一步的空间和常识推理，以对候选项进行排名、过滤并选择最匹配的对象。

## II 相关工作

自然语言的3D视觉定位。在无结构的3D场景中将自然语言查询与场景结合，对于各种机器人任务至关重要。开创性的基准测试，如ScanRefer [[12](#bib.bib12)]和ReferIt3D [[13](#bib.bib13)]，推动了这一领域的发展。如这些基准测试所提到的，3D和文本中的指代任务需要对语言的组合语义以及3D场景的结构、几何和语义有深入的理解。为推动性能发展，已经提出了许多在3D和语言上联合训练的方法 [[2](#bib.bib2), [3](#bib.bib3), [4](#bib.bib4), [5](#bib.bib5), [6](#bib.bib6), [7](#bib.bib7), [8](#bib.bib8), [9](#bib.bib9), [10](#bib.bib10), [11](#bib.bib11)]。然而，由于这些基准测试基于原始的ScanNet [[37](#bib.bib37)]，这些方法仅限于封闭词汇设置，原因在于原始ScanNet中呈现的特定对象类别。受2D开放词汇分割 [[38](#bib.bib38), [39](#bib.bib39), [40](#bib.bib40)] 发展推动，研究人员开始探索3D开放词汇定位 [[1](#bib.bib1), [14](#bib.bib14), [15](#bib.bib15), [16](#bib.bib16), [17](#bib.bib17), [18](#bib.bib18), [19](#bib.bib19), [20](#bib.bib20), [21](#bib.bib21), [41](#bib.bib41), [35](#bib.bib35)]。然而，这些方法大多依赖于CLIP [[24](#bib.bib24)]作为基础的视觉-语言桥梁。当定位文本查询是一个简单的名词短语（例如，“一个红色的苹果”）时，这种方法效果良好；然而，研究表明，CLIP表现出“词袋”行为，缺乏组合理解，如文本或视觉的关系、属性和顺序 [[25](#bib.bib25)]，这是ScanRefer和ReferIt3D所呈现挑战中的一个关键方面。认识到这一点，Semantic Abstraction [[22](#bib.bib22)]和3D-CLR [[23](#bib.bib23)]使用空间信息的文本和3D数据，训练一个专门的神经网络，在进行定位之前解析和定位文本查询的组合语义。相比之下，我们的方法探索了使用LLM代理在无需训练（零-shot）的情况下完成相同任务的可能性。NS3D [[36](#bib.bib36)]使用基于LLM的代码生成生成程序来解决这个问题，虽然与我们的方法较为相似，但它仍然使用真实的对象分割和类别来简化视觉定位，因此缺乏开放词汇和零-shot能力。

LLM代理 最近，大型语言模型（LLM）[[42](#bib.bib42), [43](#bib.bib43), [44](#bib.bib44), [26](#bib.bib26), [45](#bib.bib45), [46](#bib.bib46)]的最新进展展示了其令人惊讶的出现能力。在这里，我们列举了一些使LLM能够作为代理使用的能力。

#### 规划

规划涉及将复杂的目标分解为子目标，并基于已执行的动作和环境反馈进行自我反思。Chain-of-thought [[27](#bib.bib27)] 表明，当被指示“逐步思考”时，LLM 能通过将复杂任务分解为更小的任务来展现更好的规划能力。Tree-of-thoughts [[28](#bib.bib28)] 通过探索每个步骤中的多种思路扩展了这一方法，将链条转变为树形结构。[[47](#bib.bib47), [48](#bib.bib48), [49](#bib.bib49), [50](#bib.bib50)] 展示了当 LLM 被指示对其输出和环境反馈进行自我反思时，它能够生成更好的输出。

#### 工具使用

使用工具的能力是人类智能的独特特征。认识到当前的 LLM 在所有任务上都不擅长（例如数学和事实问答问题），研究人员已探索让 LLM 协调工具使用以完成任务的可能性。从本质上讲，工具使用问题是决定使用哪个工具以及何时使用它们。苏格拉底模型 [[29](#bib.bib29)] 使用自然语言作为媒介，促使 LLM 代理与其他多模态语言模型（如视觉语言模型和音频语言模型）进行引导讨论，共同完成任务。MRKL [[51](#bib.bib51)] 和 TAML [[52](#bib.bib52)] 为 LLM 配备了计算器，并展示了它在解决数学问题方面的增强能力。基于这些发现，像 LangChain [[53](#bib.bib53)] 这样的软件库已被开发出来，以简化 LLM 工具使用的开发者流程。ToolFormer [[30](#bib.bib30)]、HuggingGPT [[54](#bib.bib54)] 和 API-Bank [[55](#bib.bib55)] 通过开放更多 API 和机器学习模型作为 LLM 使用的工具，进一步推动了工具使用的进展。

在机器人技术中，SayCan [[31](#bib.bib31)]、InnerMonologue [[32](#bib.bib32)]、Code as Policies [[33](#bib.bib33)] 和 LM-Nav [[34](#bib.bib34)] 利用 LLM 的规划和工具使用能力，使其作为真实机器人在长时程复杂任务中的高级控制器。这些任务的成功促使我们使用 LLM 作为代理，帮助解决 3D 视觉引导中提出的组合语言-视觉理解挑战。

| 训练规模 | 开放词汇 | 方法 | 视觉引导器 | + LLM 代理 | Acc@0.25 $\uparrow$ | Acc@0.5 $\uparrow$ |
| --- | --- | --- | --- | --- | --- | --- |
| 36k 标注的 3D 文本数据 | 闭集词汇 | ScanRefer[[12](#bib.bib12)] | – | – | 34.4 | 20.1 |
| 3DVG-Trans[[2](#bib.bib2)] | – | – | 41.5 | 28.2 |
| 零-shot | 开放词汇 | LERF[[35](#bib.bib35)] | LERF | ✗ | 4.4 | 0.3 |
| 我们的 | LERF | ✓ GPT-4 | 6.9 (+2.5) | 1.6 (+1.3) |
| 零-shot | 开放词汇 | OpenScene[[1](#bib.bib1)] | OpenScene | ✗ | 13.0 | 5.1 |
| 我们的 | OpenScene | ✓ GPT-3.5 | 14.3 (+1.3) | 4.7 (-0.4) |
| 我们的 | OpenScene | ✓ GPT-4 | 17.1 (+4.1) | 5.3 (+0.2) |

表I：ScanRefer实验结果。LLM（GPT-4）代理显著提高了零样本开放词汇3D定位器（如LERF和OpenScene）的3D定位能力。我们通过Accuracy@0.25和@0.5来衡量定位能力，这些是边界框预测的准确度，前提是其与真实框的交并比（IoU）分别超过0.25和0.5。括号中的数字表示添加LLM代理后的性能增益或损失。结果还显示，较不强大的LLM（如GPT-3.5）无法显著提高定位能力。最后，尽管与我们的方法（*零样本开放词汇*）不可直接比较，但为了完整性，列出了那些在*ScanRefer上训练并采用封闭词汇*的方法的性能。

|  |  | LERF | OpenScene |
| --- | --- | --- | --- |
| 低视觉难度 | 不使用LLM | 10.8 | 27.6 |
| 使用LLM | 15.1 (+4.3) | 33.6 (+6.0) |
| 高视觉难度 | 不使用LLM | 2.5 | 8.6 |
| 使用LLM | 4.4 (1.9) | 12.1 (+3.5) |

表II：视觉复杂度的消融研究。LLM代理在低视觉难度设置中对于3D定位更加有效。数字表示Acc@.25。

## III 方法

最近，来自Auto-GPT [[56](#bib.bib56)]、GPT-Engineer [[57](#bib.bib57)] 和 ToolFormer [[30](#bib.bib30)] 的成功案例展示了LLM作为代理的早期成功迹象。与传统机器学习模型不同，代理具备行动能力：它是一个由目标驱动的实体，能够推理其目标、制定计划、检查和使用工具，并与环境互动并收集反馈。在3D视觉定位任务中，代理可能是解决现有模型表现出的“词袋”行为的有前景的解决方案。在LLM-Grounder中，我们使用GPT-4作为代理，并提示其完成三个任务：1\. 将复杂的文本查询拆解成可以更好地由下游工具（如基于CLIP的3D视觉定位器，如OpenScene和LERF）处理的子任务；2\. 协调并使用这些工具来解决其提出的子任务；3\. 通过结合空间理解和常识来推理来自环境的反馈，以做出定位决策。

规划。LLM的第一个优势是它们的规划能力。研究表明，链式思维推理[[27](#bib.bib27)]，即明确地提示LLM将复杂目标分解为更小的子任务（“一步一步思考”），有助于算术、常识和符号推理任务。受这些发现的启发，我们设计了我们的代理，正如图[2](#S1.F2 "Figure 2 ‣ I Introduction ‣ LLM-Grounder: Open-Vocabulary 3D Visual Grounding with Large Language Model as an Agent")所示。具体来说，我们首先让代理描述其观察结果，这样代理就有机会总结当前的情况。上下文可以包括人类的文本查询和从工具返回的信息（在下文中描述）。然后，代理开始一个名为推理的部分，作为代理进行高层次规划的思维草稿。接着，在计划部分，代理必须列出更具体的步骤，以实现高层次计划，包括任何工具使用、比较或计算。代理可以在自我批评部分反思生成的计划，并进行任何最终的修正[[50](#bib.bib50)]。

工具使用。LLM的第二个优势来自于它们使用工具的能力。我们指示LLM代理使用工具来解决“词袋”行为（见第[II](#S2 "II Related Work ‣ LLM-Grounder: Open-Vocabulary 3D Visual Grounding with Large Language Model as an Agent)节"）。如图[2](#S1.F2 "Figure 2 ‣ I Introduction ‣ LLM-Grounder: Open-Vocabulary 3D Visual Grounding with Large Language Model as an Agent")所示，我们告知LLM预期的输入和输出格式，即我们为视觉定位和反馈设计的两个工具的API，并要求LLM代理按照给定格式与它们交互。这些工具包括目标查找器和地标查找器。

目标查找器和地标查找器。目标查找器和地标查找器接受文本查询输入，找到可能位置的聚类的边界框，并返回以质心和大小（$C_{x},C_{y},C_{z},\Delta X,\Delta Y,\Delta Z$）形式表示的候选边界框列表。目标是用户在查询中提到的主要物体（例如在“餐桌和窗户之间的椅子”中，目标是“椅子”）；地标是用来空间性地指代目标的物体（例如“餐桌”和“窗户”）。目标查找器还计算每个候选物体的体积，而地标查找器则计算每个目标候选物体的质心与地标质心之间的欧几里得距离。体积、距离和边界框一起为LLM代理提供反馈，以进行空间推理和常识推理。例如，体积为$0.01m^{3}$的候选“椅子”可能是一个假阳性，应当被过滤掉；而距离地标的距离与查询中提到的空间关系不符的候选应当被拒绝。目标查找器和地标查找器由开放词汇的基于CLIP的3D视觉定位工具LERF [[35](#bib.bib35)] 和 OpenScene [[1](#bib.bib1)] 实现。这些工具在给定复杂文本查询时表现出“词袋”行为（参见 Sec. [I](#S1 "I Introduction ‣ LLM-Grounder: Open-Vocabulary 3D Visual Grounding with Large Language Model as an Agent)")；然而，当给定更简单的文本查询，如简单的名词短语（“一把椅子”）时，这些工具通常能表现良好。LLM代理利用这种名词短语定位的能力，同时通过分解复杂的定位查询，逐一定位物体，并在之后推理它们的空间关系，从而弥补了这些3D视觉定位器在语言理解和空间推理方面的不足。为了使用目标查找器，我们指示LLM代理从原始自然语言查询中解析出名词短语（例如“木椅”）；为了使用地标查找器，我们指示LLM代理解析出原始查询中提到的任何地标物体及其与目标物体的空间关系。

## IV 实验

在实验中，我们首先希望评估基于LLM的代理在零-shot开放词汇3D视觉定位方面的表现，并与基于CLIP的3D视觉定位方法进行比较。然后，我们在封闭词汇设置中评估我们的方法，并与封闭词汇和训练过的方法进行比较。最后，我们展示一些在实际场景中的定性例子，以展示我们方法的泛化能力。

### IV-A 数据集

ScanRefer。ScanRefer [[12](#bib.bib12)] 是一个基准数据集，用于在室内3D场景中使用自然语言进行3D物体定位。它包含了51,583个由人类编写的描述，涉及18个语义类别的11,046个物体，数据来源于800个ScanNet [[37](#bib.bib37)] 3D场景，其中训练/验证/测试集分别包含36,665、9,508和5,410个描述。我们使用验证集中的前14个场景进行表格[I](#S2.T1 "TABLE I ‣ Tool-Using ‣ II Related Work ‣ LLM-Grounder: Open-Vocabulary 3D Visual Grounding with Large Language Model as an Agent")中展示的实验，该表格包含998个文本与3D物体的配对。我们还报告了ScanRefer的两个标准度量：Accuracy@0.25 和 Accuracy@0.5。0.25和0.5是3D边界框IoU的不同阈值。

### IV-B 基准方法

![参见说明](img/74f82f104c4ac0a0eff9ce6bd08ae865.png)

图3：定性示例。LLM代理使用空间推理成功区分出正确的物体实例。

ScanRefer。ScanRefer [[12](#bib.bib12)] 使用端到端的3D-文本神经架构，通过自然语言输入来定位物体。具体来说，它将3D点云处理为PointNet++ [[58](#bib.bib58)] 特征，然后对这些点进行聚类，并提出物体的边界框。接着，语言特征与聚类和框进行融合，以确定哪些框是由语言所指代的。该流程使用文本与边界框配对、场景中所有物体的真实边界框和语义类别进行监督。我们将此基准作为当前已训练管道性能的展示，与我们在零-shot设置下的实验进行对比，后者没有使用任何监督。

3DVG-Transformer。3DVG-Transformer [[2](#bib.bib2)] 基于ScanRefer的端到端神经架构，提出了一种新的神经模块，用于在提出边界框之前聚合邻近的聚类。与ScanRefer类似，3DVG-Transformer同样使用真实物体边界框、语义类别和人类注释描述的监督。

OpenScene 和 LERF。OpenScene [[1](#bib.bib1)] 和 LERF [[35](#bib.bib35)] 是零-shot 开放词汇 3D 场景理解方法。OpenScene 将 2D CLIP 特征提取并转化为 3D 点云，并通过计算查询的 CLIP 文本嵌入与 3D 点云中每个点之间的余弦相似度来实现文本查询的定位。LERF 通过将 CLIP 嵌入编码到神经辐射场中来实现相同的功能。当单独使用时，这些方法表现出“词袋模型”行为，如图 [1](#S0.F1 "Figure 1 ‣ LLM-Grounder: Open-Vocabulary 3D Visual Grounding with Large Language Model as an Agent") 所示，这是我们希望通过 LLM 代理的深思熟虑推理来解决的问题。为了使用 OpenScene 和 LERF 在 3D 视觉定位基准 ScanRefer 上生成边界框，我们对具有高余弦相似度的点应用 DBSCAN 聚类 [[59](#bib.bib59)]，并在其周围绘制边界框。

### IV-C 结果

我们首先展示了 LLM-Grounder 在图 [3](#S4.F3 "Figure 3 ‣ IV-B Baseline Methods ‣ IV Experiments ‣ LLM-Grounder: Open-Vocabulary 3D Visual Grounding with Large Language Model as an Agent") 中的定性结果。更多结果和演示可以在项目网站¹¹1[https://chat-with-nerf.github.io/](https://chat-with-nerf.github.io/)上找到，包括野外场景。

与基准方法相比，我们发现*LLM 代理可以提高零-shot 开放词汇定位*。如表 [I](#S2.T1 "TABLE I ‣ Tool-Using ‣ II Related Work ‣ LLM-Grounder: Open-Vocabulary 3D Visual Grounding with Large Language Model as an Agent") 所示，添加 LLM 代理后，LERF 和 OpenScene 的定位性能分别在 Accuracy$@$0.25 上分别提高了 5.0% 和 17.1%。我们将 LERF 性能提升较小的原因归因于 LERF 本身较弱的整体定位能力。性能提升较小表明，当工具向 LLM 代理提供过于嘈杂的反馈时，LLM 代理很难基于嘈杂的输入进行推理并提高性能。我们还注意到 Accuracy$@$0.5 上的性能提升较低，该指标要求预测的边界框与真实边界框的重叠度超过 50%。我们将这一现象归因于底层定位器缺乏实例分割能力。我们观察到，定位器通常会为正确定位的物体预测出过大或过小的边界框，这种预测无法被 LLM 修正，因此导致精确视觉定位的困难和性能提升较低。此外，我们发现当使用 GPT-3.5 作为 OpenScene 的代理时，性能相较于不使用 GPT 时有所下降。我们将此归因于 GPT-3.5 在工具使用、空间推理和常识推理能力方面较弱。

![Refer to caption](img/b5683635dd7263019d32ce06e0f6fe98.png)

图4：性能差异（有LLM - 无LLM）与查询文本复杂度的对比。当文本查询更复杂时，LLM的帮助作用更大，但在更高复杂度的情况下，帮助作用并不显著。

![请参阅说明文字](img/96f273b196ad31fee5c1d43452954995.png)

图5：不同模型与查询文本复杂度的性能对比。所有模型在处理更复杂的句子时都表现不佳，但带有LLM代理的模型表现更好，尤其是在这些较高复杂度的情况下。

### IV-D 消融研究

我们接下来评估LLM代理主要改进的方面。我们测试了两种不同的设置：（1）LLM代理在更困难的视觉上下文中是否能提供更多帮助？（2）LLM代理在更困难的文本查询中是否能提供更多帮助？

视觉上下文的难度。我们在表格[II](#S2.T2 "TABLE II ‣ Tool-Using ‣ II Related Work ‣ LLM-Grounder: Open-Vocabulary 3D Visual Grounding with Large Language Model as an Agent")中按视觉难度对结果进行了分类，发现LLM代理对于低视觉难度的查询更为有效，表现为更高的定位性能提升。具体来说，我们将定位查询分为*低视觉难度*和*高视觉难度*类别。如果文本查询中提到的物体是该场景中该类别的唯一物体（0个干扰物体），则该查询具有低视觉难度；如果场景中有多个同类干扰物体，则该查询具有高视觉难度。在我们评估的998个查询中，232个查询具有低视觉难度，766个查询具有高视觉难度。表格[II](#S2.T2 "TABLE II ‣ Tool-Using ‣ II Related Work ‣ LLM-Grounder: Open-Vocabulary 3D Visual Grounding with Large Language Model as an Agent")中的结果显示，LLM在低视觉难度查询中带来了更多的性能提升。这种行为可以通过低视觉难度和高视觉难度设置中呈现的不同挑战来解释。在低视觉难度设置中，开放词汇3D定位器面临的主要挑战是“词袋”行为。例如，如果文本查询是“厨房里的水槽”，且场景中只有一个水槽，词袋定位器将突出整个厨房，导致定位精度较低。而LLM代理特别擅长解决这个问题，它通过解析出目标物体“水槽”并仅将这个单一名词传递给定位器，从而避免了词袋行为。然而，在高视觉难度设置中，还有一个额外的挑战：*实例歧义*。由于场景中有多个同类物体，视觉定位器会返回多个候选对象给LLM代理。LLM代理可以利用其空间和常识推理能力，借助体积和与地标的距离信息筛选出一些实例，但更复杂的实例歧义通常需要更细致的视觉线索，而这正是LLM代理所不具备的，因为它是盲目的。

文本查询的难度。随着查询的复杂度增加，LLM代理有助于提高性能，但仅限于一定的范围。我们可以通过计算句子中名词的数量来衡量查询的复杂性：描述中名词越多，定位特定物体就越困难。从图[5](#S4.F5 "图5 ‣ IV-C 结果 ‣ IV 实验 ‣ LLM-Grounder：作为代理的开放词汇3D视觉定位")可以看到，无论是否使用LLM代理，随着句子复杂性的增加，性能都会下降。然而，从分析使用LLM代理与不使用LLM代理的性能差异来看，我们发现查询复杂度与性能呈二次方依赖关系（图[4](#S4.F4 "图4 ‣ IV-C 结果 ‣ IV 实验 ‣ LLM-Grounder：作为代理的开放词汇3D视觉定位")）。这表明，当面对更高复杂度的查询时，LLM能在定位中提供优势，但当复杂度超过某一阈值后，这一优势会减弱。当查询复杂度较低时，没有LLM的模型也能有效定位物体，因此LLM的优势不明显。随着复杂度的增加，基准模型的表现变差，而LLM提供了更显著的优势。然而，随着指称表达的复杂性增加，LLM的空间推理能力可能无法超越无LLM基准模型的表现。我们可能需要更强大的LLM才能在这些高复杂度范围内获得优势。

## V 结论与局限性

我们介绍了LLM-Grounder，这是一种新颖的3D视觉定位方法，利用大型语言模型（LLMs）作为协调定位过程的核心代理。我们的实证评估表明，LLM-Grounder在处理复杂文本查询方面表现尤为突出，从而为3D视觉定位任务提供了一种强大、零-shot、开放词汇的解决方案。然而，仍有一些限制需要考虑。成本：使用基于GPT的模型作为核心推理代理可能会非常耗费计算资源，这可能会限制其在资源有限的环境中的应用。延迟：由于GPT模型固有的延迟，推理过程可能较慢。这种延迟可能成为实时机器人应用中的一个重要瓶颈，在这些应用中，快速决策至关重要。尽管存在这些限制，LLM-Grounder仍然为3D视觉定位设立了一个新的基准，并为未来将LLMs与机器人系统结合的研究开辟了新途径。

## 参考文献

+   [1] S. Peng, K. Genova, C. M. Jiang, A. Tagliasacchi, M. Pollefeys, 和 T. Funkhouser, “Openscene: 使用开放词汇的3D场景理解”，发表于*IEEE/CVF计算机视觉与模式识别会议（CVPR）论文集*，2023年。

+   [2] L. Zhao, D. Cai, L. Sheng, 和 D. Xu, “3DVG-Transformer：基于点云的视觉定位关系建模”，发表于*ICCV*，2021年，第2928-2937页。

+   [3] J. Roh, K. Desingh, A. Farhadi 和 D. Fox， “Languagerefer：3D视觉定位的空间-语言模型，”发表于 *机器人学习会议*。PMLR，2022年，第1046–1056页。

+   [4] D. Cai, L. Zhao, J. Zhang, L. Sheng 和 D. Xu， “3DJCG：用于3D点云上的联合密集标注和视觉定位的统一框架，”发表于 *IEEE/CVF计算机视觉与模式识别会议论文集*，2022年，第16 464–16 473页。

+   [5] J. Chen, W. Luo, R. Song, X. Wei, L. Ma 和 W. Zhang， “学习点-语言层次对齐用于3D视觉定位，” 2022年。

+   [6] D. Z. Chen, Q. Wu, M. Nießner 和 A. X. Chang， “D3net：用于RGB-D扫描中的半监督密集标注和视觉定位的说话者-听众架构，” 2021年。

+   [7] Z. Yuan, X. Yan, Y. Liao, R. Zhang, S. Wang, Z. Li 和 S. Cui， “Instancerefer：通过实例多级上下文参考进行点云上的视觉定位的协同整体理解，”发表于 *IEEE/CVF国际计算机视觉会议论文集*，2021年，第1791–1800页。

+   [8] E. Bakr, Y. Alsaedy 和 M. Elhoseiny， “环顾四周并参考：2D合成语义知识蒸馏用于3D视觉定位，” *神经信息处理系统进展*，第35卷，第37 146–37 158页，2022年。

+   [9] H. Liu, A. Lin, X. Han, L. Yang, Y. Yu 和 S. Cui， “Refer-it-in-rgbd：一种自下而上的方法用于RGBD图像中的3D视觉定位，”发表于 *IEEE/CVF计算机视觉与模式识别会议论文集*，2021年，第6032–6041页。

+   [10] A. Jain, N. Gkanatsios, I. Mediratta 和 K. Fragkiadaki， “自下而上的检测变换器用于图像和点云中的语言定位，”发表于 *欧洲计算机视觉会议*。Springer，2022年，第417–433页。

+   [11] S. Huang, Y. Chen, J. Jia 和 L. Wang， “多视角变换器用于3D视觉定位，”发表于 *IEEE/CVF计算机视觉与模式识别会议论文集*，2022年，第15 524–15 533页。

+   [12] D. Z. Chen, A. X. Chang 和 M. Nießner， “Scanrefer：使用自然语言进行RGB-D扫描中的3D物体定位，” *第16届欧洲计算机视觉会议（ECCV）*，2020年。

+   [13] P. Achlioptas, A. Abdelreheem, F. Xia, M. Elhoseiny 和 L. J. Guibas， “ReferIt3D：用于现实世界场景中细粒度3D物体识别的神经听众，”发表于 *第16届欧洲计算机视觉会议（ECCV）*，2020年。

+   [14] B. Chen, F. Xia, B. Ichter, K. Rao, K. Gopalakrishnan, M. S. Ryoo, A. Stone, 和 D. Kappler， “开放词汇可查询场景表示用于现实世界规划，”发表于 *2023 IEEE国际机器人与自动化会议（ICRA）*。IEEE，2023，第11 509–11 522页。

+   [15] R. Ding, J. Yang, C. Xue, W. Zhang, S. Bai 和 X. Qi， “PLA：语言驱动的开放词汇3D场景理解，”发表于 *IEEE/CVF计算机视觉与模式识别会议论文集*，2023年，第7010–7019页。

+   [16] S. Y. Gadre, M. Wortsman, G. Ilharco, L. Schmidt, 和 S. Song, “牧场上的牛：语言驱动的零-shot物体导航基准线和基准测试，” *CVPR*，2023年。

+   [17] C. Huang, O. Mees, A. Zeng, 和 W. Burgard, “用于机器人导航的视觉语言地图，” *2023 IEEE国际机器人与自动化会议（ICRA）*。IEEE，2023年，pp. 10 608–10 615。

+   [18] K. M. Jatavallabhula, A. Kuwajerwala, Q. Gu, M. Omama, T. Chen, A. Maalouf, S. Li, G. S. Iyer, S. Saryazdi, N. V. Keetha *等*，“Conceptfusion：开放集多模态3D映射，” *ICRA2023机器人预训练研讨会（PT4R）*，2023年。

+   [19] K. Mazur, E. Sucar, 和 A. J. Davison, “特征逼真的神经融合用于实时、开放集场景理解，” *2023 IEEE国际机器人与自动化会议（ICRA）*。IEEE，2023年，pp. 8201–8207。

+   [20] N. M. M. Shafiullah, C. Paxton, L. Pinto, S. Chintala, 和 A. Szlam, “Clip-fields：弱监督语义场用于机器人记忆，” *ICRA2023机器人预训练研讨会（PT4R）*，2023年。

+   [21] A. Takmaz, E. Fedele, R. W. Sumner, M. Pollefeys, F. Tombari, 和 F. Engelmann, “OpenMask3D：开放词汇3D实例分割，” *arXiv预印本arXiv:2306.13631*，2023年。

+   [22] H. Ha 和 S. Song, “语义抽象：来自2D视觉-语言模型的开放世界3D场景理解，” *第六届机器人学习年会*，2022年。[在线]。可访问：[https://openreview.net/forum?id=lV-rNbXVSaO](https://openreview.net/forum?id=lV-rNbXVSaO)

+   [23] Y. Hong, C. Lin, Y. Du, Z. Chen, J. B. Tenenbaum, 和 C. Gan, “从多视角图像中学习和推理3D概念，” *IEEE/CVF计算机视觉与模式识别会议论文集*，2023年。

+   [24] A. Radford, J. W. Kim, C. Hallacy, A. Ramesh, G. Goh, S. Agarwal, G. Sastry, A. Askell, P. Mishkin, J. Clark *等*，“从自然语言监督中学习可转移的视觉模型，” *国际机器学习会议*。PMLR，2021年，pp. 8748–8763。

+   [25] M. Yuksekgonul, F. Bianchi, P. Kalluri, D. Jurafsky, 和 J. Zou, “视觉-语言模型何时以及为什么像词袋模型一样工作，以及如何应对？” *第十一届国际学习表征会议*，2022年。

+   [26] OpenAI, “GPT-4技术报告，” 2023年。

+   [27] J. Wei, X. Wang, D. Schuurmans, M. Bosma, F. Xia, E. Chi, Q. V. Le, D. Zhou *等*，“链式思维提示激发大型语言模型的推理，” *神经信息处理系统进展*，第35卷，pp. 24 824–24 837，2022年。

+   [28] S. Yao, D. Yu, J. Zhao, I. Shafran, T. L. Griffiths, Y. Cao, 和 K. Narasimhan, “思维树：使用大型语言模型进行深思熟虑的问题解决，” *arXiv预印本arXiv:2305.10601*，2023年。

+   [29] A. Zeng, M. Attarian, K. M. Choromanski, A. Wong, S. Welker, F. Tombari, A. Purohit, M. S. Ryoo, V. Sindhwani, J. Lee *等*，“苏格拉底模型：通过语言进行零样本多模态推理的组合，” 载于 *第十一届国际学习表征大会*，2022年。

+   [30] T. Schick, J. Dwivedi-Yu, R. Dessì, R. Raileanu, M. Lomeli, L. Zettlemoyer, N. Cancedda 和 T. Scialom, “Toolformer：语言模型可以自学使用工具，” 2023年。

+   [31] M. Ahn, A. Brohan, N. Brown, Y. Chebotar, O. Cortes, B. David, C. Finn, C. Fu, K. Gopalakrishnan, K. Hausman *等*，“做我能做的，而不是我说的：将语言与机器人能力联系起来，” *arXiv预印本arXiv:2204.01691*，2022年。

+   [32] W. Huang, F. Xia, T. Xiao, H. Chan, J. Liang, P. Florence, A. Zeng, J. Tompson, I. Mordatch, Y. Chebotar *等*，“内心独白：通过与语言模型的规划进行具身推理，” 载于 *机器人学习大会*。PMLR，2023年，页1769–1782。

+   [33] J. 梁, W. 黄, F. 夏, P. 徐, K. Hausman, B. Ichter, P. Florence 和 A. Zeng, “代码作为策略：用于具身控制的语言模型程序,” 载于 *2023年IEEE国际机器人与自动化会议（ICRA）*。IEEE，2023年，页9493–9500。

+   [34] D. Shah, B. Osinski, B. Ichter 和 S. Levine, “LM-nav：基于大型预训练语言、视觉和行动模型的机器人导航，” 载于 *第六届机器人学习年会*，2022年。[在线]. 可用链接：[https://openreview.net/forum?id=UW5A3SweAH](https://openreview.net/forum?id=UW5A3SweAH)

+   [35] J. Kerr, C. M. Kim, K. Goldberg, A. Kanazawa 和 M. Tancik, “Lerf：嵌入语言的辐射场,” 载于 *国际计算机视觉大会（ICCV）*，2023年。

+   [36] J. Hsu, J. Mao 和 J. Wu, “Ns3d：3D物体和关系的神经符号基础，” *2023年IEEE/CVF计算机视觉与模式识别大会（CVPR）*，页2614–2623，2023年。[在线]. 可用链接：[https://api.semanticscholar.org/CorpusID:257687234](https://api.semanticscholar.org/CorpusID:257687234)

+   [37] A. Dai, A. X. Chang, M. Savva, M. Halber, T. Funkhouser 和 M. Nießner, “ScanNet：丰富标注的室内场景三维重建,” 载于 *计算机视觉与模式识别会议（CVPR）论文集，IEEE*，2017年。

+   [38] B. 李, K. Q. Weinberger, S. Belongie, V. Koltun 和 R. Ranftl, “语言驱动的语义分割，” 载于 *国际学习表征大会*，2022年。[在线]. 可用链接：[https://openreview.net/forum?id=RriDjddCLN](https://openreview.net/forum?id=RriDjddCLN)

+   [39] G. Ghiasi, X. Gu, Y. Cui 和 T.-Y. Lin, “使用图像级标签扩展开放词汇图像分割,” 载于 *欧洲计算机视觉大会*。Springer，2022年，页540–557。

+   [40] F. 梁, B. 吴, X. 戴, K. 李, Y. 赵, H. 张, P. 张, P. Vajda 和 D. Marculescu, “基于掩膜适应的 CLIP 进行开放词汇语义分割,” 载于 *IEEE/CVF计算机视觉与模式识别大会论文集*，2023年，页7061–7070。

+   [41] S. Kobayashi, E. Matsumoto, 和 V. Sitzmann, “通过特征场蒸馏分解 Nerf 以进行编辑，” 收录于 *神经信息处理系统进展*，第35卷，2022年。[在线]。可访问：[https://arxiv.org/pdf/2205.15585.pdf](https://arxiv.org/pdf/2205.15585.pdf)

+   [42] T. Brown, B. Mann, N. Ryder, M. Subbiah, J. D. Kaplan, P. Dhariwal, A. Neelakantan, P. Shyam, G. Sastry, A. Askell *等*， “语言模型是少样本学习者，” *神经信息处理系统进展*，第33卷，第1877–1901页，2020年。

+   [43] J. Wei, M. Bosma, V. Zhao, K. Guu, A. W. Yu, B. Lester, N. Du, A. M. Dai, 和 Q. V. Le, “微调的语言模型是零样本学习者，” 收录于 *国际学习表征会议*，2021年。

+   [44] L. Ouyang, J. Wu, X. Jiang, D. Almeida, C. Wainwright, P. Mishkin, C. Zhang, S. Agarwal, K. Slama, A. Ray *等*， “训练语言模型以遵循带有人类反馈的指令，” *神经信息处理系统进展*，第35卷，第27,730–27,744页，2022年。

+   [45] C. Raffel, N. Shazeer, A. Roberts, K. Lee, S. Narang, M. Matena, Y. Zhou, W. Li, 和 P. J. Liu, “探索通过统一的文本到文本变换器进行迁移学习的极限，” *机器学习研究杂志*，第21卷，第1期，第5485–5551页，2020年。

+   [46] H. Touvron, T. Lavril, G. Izacard, X. Martinet, M.-A. Lachaux, T. Lacroix, B. Rozière, N. Goyal, E. Hambro, F. Azhar *等*， “Llama: 开放且高效的基础语言模型，” *arXiv 预印本 arXiv:2302.13971*，2023年。

+   [47] S. Yao, J. Zhao, D. Yu, N. Du, I. Shafran, K. Narasimhan, 和 Y. Cao, “React: 在语言模型中协同推理与行动，” *arXiv 预印本 arXiv:2210.03629*，2022年。

+   [48] N. Shinn, F. Cassano, B. Labash, A. Gopinath, K. Narasimhan, 和 S. Yao, “Reflexion: 带有语言强化学习的语言代理，” *arXiv 预印本 arXiv:2303.11366*，2023年。

+   [49] H. Liu, C. Sferrazza, 和 P. Abbeel, “事后链条使语言模型与反馈对齐，” *arXiv 预印本 arXiv:2302.02676*，第3卷，2023年。

+   [50] E. Jang, “大型语言模型能否对自己的输出进行批判和迭代？” *evjang.com*，2023年3月。[在线]。可访问：[https://evjang.com/2023/03/26/self-reflection.html](https://evjang.com/2023/03/26/self-reflection.html)

+   [51] E. Karpas, O. Abend, Y. Belinkov, B. Lenz, O. Lieber, N. Ratner, Y. Shoham, H. Bata, Y. Levine, K. Leyton-Brown *等*， “Mrkl 系统：一种模块化的神经符号架构，结合了大型语言模型、外部知识源和离散推理，” *arXiv 预印本 arXiv:2205.00445*，2022年。

+   [52] A. Parisi, Y. Zhao, 和 N. Fiedel, “Talm: 工具增强的语言模型，” *arXiv 预印本 arXiv:2205.12255*，2022年。

+   [53] H. Chase, “LangChain，” 2022年10月。[在线]。可访问：[https://github.com/hwchase17/langchain](https://github.com/hwchase17/langchain)

+   [54] Y. Shen, K. Song, X. Tan, D. Li, W. Lu, 和 Y. Zhuang, “Hugginggpt: 利用ChatGPT及其在Huggingface上的朋友解决AI任务，” *arXiv预印本 arXiv:2303.17580*，2023年。

+   [55] M. Li, F. Song, B. Yu, H. Yu, Z. Li, F. Huang, 和 Y. Li, “Api-bank: 用于工具增强的LLM基准测试，” *arXiv预印本 arXiv:2304.08244*，2023年。

+   [56] “Auto-gpt,” [https://github.com/Significant-Gravitas/Auto-GPT](https://github.com/Significant-Gravitas/Auto-GPT)，2013年。

+   [57] “Gpt-engineer,” [https://github.com/AntonOsika/gpt-engineer](https://github.com/AntonOsika/gpt-engineer)，2013年。

+   [58] C. R. Qi, L. Yi, H. Su, 和 L. J. Guibas, “Pointnet++: 在度量空间中对点集进行深度层次特征学习，” *神经信息处理系统进展*，第30卷，2017年。

+   [59] M. Ester, H.-P. Kriegel, J. Sander, X. Xu *等*，“一种基于密度的算法，用于在含噪声的大型空间数据库中发现聚类，” 在 *kdd*，第96卷，第34期，1996年，第226-231页。
