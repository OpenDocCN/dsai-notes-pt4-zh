<!--yml

类别：未分类

日期：2025-01-11 12:46:27

-->

# LLM增强的自主智能体能否合作？通过Melting Pot评估它们的合作能力

> 来源：[https://arxiv.org/html/2403.11381/](https://arxiv.org/html/2403.11381/)

Manuel Mosquera [ma.mosquerao@uniandes.edu.co](mailto:ma.mosquerao@uniandes.edu.co) Juan Sebastian Pinzón [js.pinzonr@uniandes.edu.co](mailto:js.pinzonr@uniandes.edu.co) Yesid Fonseca [y.fonseca@uniandes.edu.co](mailto:y.fonseca@uniandes.edu.co) Manuel Ríos [manrios@bancolombia.com.co](mailto:manrios@bancolombia.com.co) Nicanor Quijano [nquijano@uniandes.edu.co](mailto:nquijano@uniandes.edu.co) Luis Felipe Giraldo [lf.giraldo404@uniandes.edu.co](mailto:lf.giraldo404@uniandes.edu.co) Rubén Manrique [rf.manrique@uniandes.edu.co](mailto:rf.manrique@uniandes.edu.co)

###### 摘要

随着人工智能领域的不断发展，其中一个重要方向是大规模语言模型（LLM）的发展及其在增强多智能体人工智能系统中的潜力。本文探讨了通过著名的Melting Pot环境和参考模型（如GPT-4和GPT-3.5）来评估大规模语言模型增强的自主智能体（LAA）的合作能力。初步结果表明，尽管这些智能体表现出一定的合作倾向，但在特定环境下它们仍然难以实现有效的协作，这凸显了更强大架构的需求。研究的贡献包括：为LLM适配Melting Pot游戏场景的抽象层，实施用于LLM介导的智能体开发的可重用架构（该架构包括短期和长期记忆以及不同的认知模块），以及通过与Melting Pot的“Commons Harvest”游戏相关的度量标准来评估合作能力。最后，本文讨论了当前架构框架的局限性，以及通过一组新模块促进LAA更好合作的潜力。

###### 关键词：

智能体，LLM，合作性人工智能^†^†期刊：人工智能期刊\affiliation

[DISC]组织=洛杉矶大学系统与计算工程系，地址=地址，城市=波哥大，邮政编码=111711，国家=哥伦比亚 \affiliation[IBIO]组织=洛杉矶大学生物医学工程系，地址=地址，城市=波哥大，邮政编码=111711，国家=哥伦比亚 \affiliation[IELE]组织=洛杉矶大学电气与电子工程系，地址=地址，城市=波哥大，邮政编码=111711，国家=哥伦比亚 \affiliation[BANCOL]组织=巴科洛比亚银行分析与人工智能卓越中心，城市=波哥大，邮政编码=111711，国家=哥伦比亚

## 1 引言

AI代理在日常生活中，如自动驾驶车辆和客户服务等领域的出现和相关性日益增加，这要求这些实体具备适当的能力，以便与人类和其他AI对等体进行合作。尽管在推动AI代理个体智能组件的进展上取得了显著进展，但将研究重点扩展到增强其社交智能——即在群体环境中有效协作以解决普遍问题的能力——现在正是时宜。这一转变与AI研究的快速发展相契合，带来了新的机会，旨在促进合作，并借鉴社会选择理论和社会系统发展的见解（Dafoe等，[2020](https://arxiv.org/html/2403.11381v2#bib.bib4)）。

此外，利用AI来管理开放创新过程，为提升AI的社交智能提供了框架。通过整合AI来执行关键职能，如绘制创新格局、协调多样的知识输入，并确保合作符合集体目标，我们可以更好地理解并利用AI在促进社区内外复杂协作努力中的能力（Broekhuizen等，[2023](https://arxiv.org/html/2403.11381v2#bib.bib3)）。

合作需要两个或更多的代理基于相互理解，进行协作性行动。从进化的角度来看，合作在物种的生存中发挥了至关重要的作用（Pennisi，[2009](https://arxiv.org/html/2403.11381v2#bib.bib15)；Dale等，[2020](https://arxiv.org/html/2403.11381v2#bib.bib5)）。它促进了影响我们周围环境的社会结构的发展，并为解决复杂问题提供了解决方案，例如社会困境（Gross等，[2023](https://arxiv.org/html/2403.11381v2#bib.bib7)）。揭示促使合作行为在社区中演化的过程被视为科学家应当应对的重要挑战（Pennisi，[2009](https://arxiv.org/html/2403.11381v2#bib.bib15)）。

已经提出了合作的数学和计算模型，用以研究促进人工智能智能体在协作行为中出现合作的机制，旨在提高它们的共同福利（Dafoe等， [2020](https://arxiv.org/html/2403.11381v2#bib.bib4)）。例如，矩阵博弈多年来为研究社会困境提供了一个有用的工具（Axelrod和Hamilton， [1981](https://arxiv.org/html/2403.11381v2#bib.bib2)），在这种博弈中，合作或背叛的决策被限制为一个原子行为。此外，多智能体强化学习已被用来研究在复杂场景中，如何学习合作策略，尤其是在存在时间上扩展的社会困境的情况下（Leibo等， [2017](https://arxiv.org/html/2403.11381v2#bib.bib10)， [2021](https://arxiv.org/html/2403.11381v2#bib.bib9); McKee等， [2023](https://arxiv.org/html/2403.11381v2#bib.bib12); Rios等， [2023](https://arxiv.org/html/2403.11381v2#bib.bib16)）。

对AI智能体的研究为生成体现更具人性特征且与人类兼容的智能技术提供了一个方向，远离了忽视智能体互动的唯我主义方法。这种方法的一个有前景的例子是Melting Pot，这是一种AI研究工具，旨在通过标准化的测试场景促进多智能体人工智能中的协作。这些环境通过将物理环境（“基底”）与一组共同参与者（“背景人群”）配对，强调非平凡、可学习和可度量的合作（Agapiou等， [2023](https://arxiv.org/html/2403.11381v2#bib.bib1)）。这些环境促进了参与者之间的相互依存关系。

最近，AI智能体的研究也受到了大规模语言模型（LLMs）飞跃性进展的影响。LLMs的日益成功鼓励了对LLM增强型自主智能体（LAA）的进一步探索。LAA代表了一种仍在兴起的研究方向，目前已有的探索非常有限（Du等， [2023](https://arxiv.org/html/2403.11381v2#bib.bib6); Liu等， [2023](https://arxiv.org/html/2403.11381v2#bib.bib11); Zhang等， [2023](https://arxiv.org/html/2403.11381v2#bib.bib21); Shinn等， [2023](https://arxiv.org/html/2403.11381v2#bib.bib18); Schick等， [2023](https://arxiv.org/html/2403.11381v2#bib.bib17); Yao等， [2023](https://arxiv.org/html/2403.11381v2#bib.bib20))。这些研究中有一个共同点，即已经明确指出了需求。为了实现成功，LAA必须依赖于一种能够回忆相关事件的架构，并反思这些记忆以进行泛化和得出更高层次的推理，利用这些推理来制定及时且长期的计划（Park等， [2023](https://arxiv.org/html/2403.11381v2#bib.bib14)）。

这些架构提供了专门的模块用于特定任务，利用精心设计的提示和流程来执行复杂任务并导航复杂的环境。在这些架构中，已经观察到有人类行为的复制，特别是Park等人（[2023](https://arxiv.org/html/2403.11381v2#bib.bib14)）成功地在模拟环境中创建了逼真的人类行为。类似的框架，由Voyager（Wang等人，[2023](https://arxiv.org/html/2403.11381v2#bib.bib19)）使用，使得一个代理能够在Minecraft世界中导航，并独立开发工具和技能。MetaGPT（Hong等人，[2023](https://arxiv.org/html/2403.11381v2#bib.bib8)）引入了一个框架，允许生成完全功能的程序，模拟具有不同角色和预定义代理交互的商业化环境。

尽管该领域取得了显著进展，增强型大语言模型（LLMs）自主代理（LAAs）中的合作能力在当前研究中仍有所忽视。然而，这些能力可能是赋能这些代理执行创新任务并在复杂环境中取得成功的关键。本研究代表了对LAAs固有合作能力的初步探索。我们采用了一个评估框架，其中包括来自Melting Pot项目（Agapiou等人，[2023](https://arxiv.org/html/2403.11381v2#bib.bib1)）的场景通信接口（在这些场景中，人工代理在可能出现社会困境的环境中共存），以及Park等人（[2023](https://arxiv.org/html/2403.11381v2#bib.bib14)）提出的最新架构，还有参考的大型语言模型（LLMs），如GPT-4和GPT-3.5。我们的结果表明，在基于简单自然语言定义和为选定Melting Pot场景量身定制的合作度量的基础上，存在合作行为的潜力。尽管代理表现出了合作的倾向，但他们的行为未能展示出在给定环境中有效合作的清晰理解。因此，我们的分析强调了需要更强大的架构，以促进LAAs中更好的合作。

总结来说，我们的贡献如下：

+   •

    将Melting Pot场景适配为LLMs可以轻松操作的文本表示。

+   •

    实现一个可重用的架构，用于开发LAAs，采用Generative Agents（Park等人，[2023](https://arxiv.org/html/2403.11381v2#bib.bib14)）中提出的模块。该架构包括短期和长期记忆，以及感知、规划、反思和行动的认知模块。我们的项目可以在[https://github.com/Cooperative-IA/CooperativeGPT](https://github.com/Cooperative-IA/CooperativeGPT)找到。

+   •

    实现自然语言指定的“个性”，使代理能够明确是否应当合作。这些描述旨在基于它们的预训练知识，判断它们在不熟悉的情境中如何理解合作。

+   •

    在使用我们架构的“公共收获”游戏中评估 LLM 中介代理，测试不同情境下，是否通过自然语言指定代理的个性。

+   •

    讨论关于“公共收获”游戏中的合作性指标的结果，使用的架构的局限性，以及提出的改进架构，旨在促进 LAAs 之间更好的合作。

## 2 相关工作

代理架构已经发展，以解决传统 LLM 的局限性，使它们配备了多种工具以实现自主操作或最低限度的人工监督。

使用大型语言模型（LLMs）时，一个显著的挑战是它们容易出现幻觉，并且在面对近期事件或特定主题时存在知识空白。这些局限性降低了它们的实用性，因为它们仅限于在训练过程中获取的信息，无法吸收新的数据。为了解决这个问题，Schick 等人 ([2023](https://arxiv.org/html/2403.11381v2#bib.bib17)) 提出了一个早期解决方案——Toolformer。该模型经过训练，能够判断何时以及如何调用 API（工具），以提升 LLM 在不同任务中的表现。该数据集是自监督的，仅在 API 调用对模型性能有正面影响时才会包含这些调用。这种方法使模型能够判断 API 调用的相关性和最佳执行时机。

然而，要执行更复杂的任务，单纯调用工具可能不足够。Toolformer 缺乏推理 API 调用背后理由的能力，并且没有接收到全面的环境反馈来指导其后续的行动以实现目标。意识到这一差距，Yao 等人 ([2023](https://arxiv.org/html/2403.11381v2#bib.bib20)) 提出了基于提示的范式 ReAct，将推理与行动相结合。通过提供上下文提示示例，ReAct 指导 LLM 何时进行推理，何时采取行动，从而在性能上优于单独使用推理或行动的方法。此外，为了使代理能够从错误中学习，Shinn 等人 ([2023](https://arxiv.org/html/2403.11381v2#bib.bib18)) 在 ReAct 框架中加入了自我反思模块。这个模块提供了对过去失败尝试的口头反馈，帮助提升后续尝试的表现。

相反，受到模拟真实人类行为的启发，Park等人（[2023](https://arxiv.org/html/2403.11381v2#bib.bib14)）设计了一个复杂的代理框架。该架构围绕着以多样化提示为主的模块，构建了一个认知序列。通过利用不同的提示，可以优化LLM的表现，使其能够为特定任务采用专门的技术。值得注意的是，记忆在该框架中占据了重要位置，能够保存代理的经验和见解。多样化地检索这些记忆，极大地增强了它们在多个目标中的实用性。此外，Wang等人（[2023](https://arxiv.org/html/2403.11381v2#bib.bib19)）开发了Voyager架构，支持在Minecraft中实现自主游戏玩法。这个先进的框架使代理能够自主制定发现议程。值得一提的是，该架构还可以创建自己的API，编写相应的代码，验证API功能，并将其存储在向量数据库中，以供未来使用。

最近，研究者们致力于通过多代理框架提升代理的表现，这些框架利用不同实例的LLM独立执行角色或任务。Du等人（[2023](https://arxiv.org/html/2403.11381v2#bib.bib6)）通过一个框架展示了这一点，该框架旨在让不同的LLM实例进行辩论，从而提高回答的事实性和准确性。随后，Hong等人进一步利用了多个代理的潜力。他们为每个代理分配了特定的角色，并伴随一系列预定义任务，明确了输入和输出的期望。这些任务为代理之间建立了结构化的交互路径，使它们能够实现用户定义的目标。通过在软件相关任务中的有效性展示，该框架借鉴了传统软件公司操作动态，达到了HumanEval和MBPP基准测试中的最先进表现。

类似地，Liu 等人（[2023](https://arxiv.org/html/2403.11381v2#bib.bib11)）提出了 BOLAA 框架。该系统通过定义由中央控制器监管的专门代理来协调多代理活动。控制器的角色至关重要：它选择最适合执行给定任务的代理，并促进与其的通信。此外，Zhang 等人（[2023](https://arxiv.org/html/2403.11381v2#bib.bib21)）深入研究了多代理架构，探讨了社会特征和协作策略对不同数据集的影响。同样，Ni 和 Buehler（[2024](https://arxiv.org/html/2403.11381v2#bib.bib13)）开发了“MechAgents”系统，这是一个利用多个动态交互的大型语言模型（LLMs）来自主解决力学任务的系统。该框架不仅展示了自我修正和互相修正的代码在人工智能代理之间的有效性，而且还使得代理能够更容易地将基于物理的建模与特定领域的知识结合起来。这为工程师解决问题的方式提供了新的自动化和改进途径。

## 3 方法论

### 3.1 实验设置

#### 3.1.1 环境

本文使用了来自 Melting Pot（Agapiou 等人，[2023](https://arxiv.org/html/2403.11381v2#bib.bib1)）的场景，这是一个由 DeepMind 开发的研究工具，旨在多代理人工智能领域内进行实验和评估。Melting Pot 中的场景特别设计，用于建立社会情境，在这些情境中，代理解决冲突的能力受到挑战，并且参与代理之间存在显著的相互依赖关系。

![参见说明](img/091e2b218be40a01369af81336809b60.png)

图1：这是“公地收获”场景运行模拟的屏幕截图。可以通过它们的黑色手臂和腿部来识别机器人。

在我们的实验过程中，我们选择了“公地收获”场景。在这一场景中，采用不可持续做法的代理可能导致资源枯竭的情况。这被称为公地悲剧。该场景的结构围绕一个网格世界展开，里面有苹果，每个苹果给予代理1点奖励。苹果的再生受每步概率的限制，该概率由苹果在L2范数中分布，半径为2决定。值得注意的是，如果附近没有其他苹果，苹果可能会枯竭。图[1](https://arxiv.org/html/2403.11381v2#S3.F1 "Figure 1 ‣ 3.1.1 Environment ‣ 3.1 Experimental setup ‣ 3 Methodology ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot")提供了这一自定义设计场景的视觉展示，图中显示了3个LLM代理和2个机器人。

LLM代理能够在每轮执行高层次动作。这些动作包括：`使玩家(player_name)在(x, y)位置无法移动`、`移动到(x, y)位置`、`保持不动`以及`探索(x, y)`。这些动作使得代理能够攻击其他玩家、移动到地图上的预定位置、停留在同一位置并探索世界。相反，机器人作为通过强化学习训练的代理，每当LLM代理进行两次移动时，它们会执行一次移动。控制机器人的政策使它们采取不可持续的采摘行为，并对附近的其他代理发起攻击。

通常，在此场景中，最大化人口福利需要LLM代理避免吃掉每棵苹果树上的最后一个苹果，并攻击那些以不可持续方式采摘苹果的机器人或代理，以防止苹果的枯竭。

#### 3.1.2 模拟

在一次模拟中，每一局游戏都会有预定数量的LLM代理和机器人参与。LLM代理在自己的回合内执行一个高层次的动作，并持续执行该动作，直到所有三个LLM代理都完成了各自的高层次动作。与此同时，机器人不断地移动，每当任何一个代理进行两次移动时，机器人会进行一次移动（请注意，一个高层次动作通常包含多个移动）。模拟要么在达到预定的最大轮次（通常为100轮）时结束，要么在所有苹果被消耗完时提前结束。

### 3.2 将环境适应LLM代理

Melting Pot场景由多个二维层组成，每一层容纳不同的物体，并且每个物体都有自己的定制逻辑。最初，使用带有不同符号的矩阵似乎是将游戏状态传达给LLM的最直观方式，但这对于像GPT-3.5或GPT-4这样的LLM来说，解读和推理矩阵中物体位置所提供的空间信息变得具有挑战性。为了解决这个问题，我们选择开发一个专门为这个环境量身定制的观察生成器。在这个生成器中，每个相关的物体都会收到自然语言描述，并附上表示行和列的坐标向量 $[x,y]$。此外，在代理等待其回合时，一些相关的状态变化也会被捕捉并传达给代理。生成的描述对象和事件的完整列表见附录[A](https://arxiv.org/html/2403.11381v2#A1 "附录 A 生成的环境对象描述 ‣ LLM增强的自主代理能否合作？通过Melting Pot评估其合作能力")。

## 4 LLM代理架构

LLM智能体的设计主要参考了生成型智能体架构（Park等，[2023](https://arxiv.org/html/2403.11381v2#bib.bib14)）。这一选择是由于其全面的特性，使其成为最具多功能性的架构之一，可以轻松调整以应对各种任务。虽然Voyager架构（Wang等，[2023](https://arxiv.org/html/2403.11381v2#bib.bib19)）也是一个可行的选项，但由于其固有的僵化性，其效能在某些方面有所限制。Voyager在游戏过程中动态构建智能体的动作，涉及生成和验证代码以在环境中执行动作。在我们具体的应用场景中，外部化动作以提高简单性被认为是更优的选择。

图 [2](https://arxiv.org/html/2403.11381v2#S4.F2 "Figure 2 ‣ 4 LLM agent architecture ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot") 展示了一个流程图，概述了智能体启动动作的过程。LLM智能体执行的每个动作都涉及一个全面的认知序列，旨在增强智能体的推理能力。该序列包括吸收来自过去经验的反馈，并将其目标转化为可行的计划，从而使智能体能够在环境中执行动作。这个架构框架始终处于环境感知状态，生成的观察数据使得智能体能够有效地应对世界的变化。

![请参阅说明](img/eff11a97f29b9897dd4ce989328abdf7.png)

图2：LLM智能体采取动作的流程图。

### 4.1 记忆结构

该智能体架构采用了三种不同的记忆结构，每种结构都针对特定功能：

#### 4.1.1 长期记忆

该仓库存储了环境的观察数据以及智能体在其认知模块中生成的各种思考。通过利用ChromaDB向量数据库，记忆得以存储，而Ada OpenAI模型则生成上下文嵌入，使得智能体能够检索与给定查询相关的记忆。

#### 4.1.2 短期记忆

为了促进特定记忆或信息的快速检索，采用了Python字典。该字典存储着智能体必须随时可用的信息，例如其名称，以及那些需要不断更新的数据，比如当前的世界观察。

#### 4.1.3 空间记忆

考虑到智能体在网格世界环境中的导航需求，空间信息变得至关重要。这包括智能体的位置和方向。为了支持智能体从一个点到另一个点的有效导航，实施了效用函数来帮助智能体实现空间感知和移动。

### 4.2 认知模块

#### 4.2.1 感知模块

认知序列的初始阶段是感知模块。该模块的任务是吸收来自环境的原始观察。这些观察作为当前世界状态的全面快照，提供有关代理可见窗口中项目的见解。

为了优化处理效率，观察会根据它们与代理的距离进行初步排序。然后，只有最接近的观察结果会传递给下一个认知模块。决定传递观察数量的参数称为`attention_bandwidth`，最初配置为10。

接下来，该模块负责构建一个内存，用于长期存储。以下是一个示例内存：

列表1：感知模块的提示

[⬇](data:text/plain;base64,SSB0b29rIHRoZSBhY3Rpb24gImdyYWIgYXBwbGUgKDksIDIwKSIgaW4gbXkgbGFzdCB0dXJuLgpTaW5jZSB0aGVuLCB0aGUgZm9sbG93aW5nIGNoYW5nZXMgaW4gdGhlIGVudmlyb25tZW50IGhhdmUgYmVlbiBvYnNlcnZlZDoKT2JzZXJ2ZWQgdGhhdCBhZ2VudCBib3RfMSB0b29rIGFuIGFwcGxlIGZyb20gcG9zaXRpb24gWzgsIDIwXS4gQXQgMjAyMy0xMS0xOSAwNDowMDowMApPYnNlcnZlZCB0aGF0IGFnZW50IGJvdF8xIHRvb2sgYW4gYXBwbGUgZnJvbSBwb3NpdGlvbiBbOCwgMjFdLiBBdCAyMDIzLTExLTE5IDA2OjAwOjAwCk9ic2VydmVkIHRoYXQgYWdlbnQgY29uZHVjdGVkIHJvY2sgb3Bwb3NpdGlvbiBsZWFybmluZyBhdCBwcm9wZXIgaW5mb3JtYXRpb24gTGF1cmEgZnJvbSBwb3NpdGlvbiBbMiwgMTVdLiBBdCAyMDIzLTExLTE5IDA3OjAwOjAwCnRoZSBjdXJyZW50IHRpbWUgaW4gdGhlIGdyYWQgdXBkYXRlIHRoZSBjdXJyZW50IGVuZHVyaW5nIHRpbWUgc2VsZWN0ZWQgdGhlc2UgY2hhbGxlbmdlcyBhbmQgdGV4dCwgYnJlYWtpbmcuIFRoaW5rcyBsaWZlIGluIHRoZSBtYXBsaW5nIHRhZ2luZyBhbmQgd2l0aCBpdCwgYXBwZWFycyBhY2N1cmFjeS4KSSBjYW4gY3VycmVudGx5IG9ic2VydmUgdGhlIGZvbGxvd2luZzoKT2JzZXJ2ZWQgYW4gYXBwbGUgYXQgcG9zaXRpb24gWzksIDIwXS4gVGhpcyBhcHBsZSBiZWxvbmdzIHRvIHRyZWUgNi4KT2JzZXJ2ZWQgZ3Jhc3MgdG8gZ3JvdyBhcHBsZXMgYXQgcG9zaXRpb24gWzgsIDIwXS4gVGhpcyBncmFzcyBiZWxvbmdzIHRvIHRyZWUgNi4=)1 我在上一次回合中执行了动作“抓取苹果（9, 20）”。

最终，感知模块决定代理是否应根据当前的观察启动回应。在此阶段，代理评估其现有计划和待执行的行动，判断它们的适宜性。它评估是否继续当前的行动方案，或者观察到的条件是否需要制定新计划，并生成相应的执行行动。完整的提示见 [C](https://arxiv.org/html/2403.11381v2#A3 "附录C 反应提示 ‣ LLM增强的自主代理能合作吗？通过Melting Pot评估它们的合作能力")。

#### 4.2.2 规划模块

该模块在观察结果已排序和筛选后发挥作用。规划模块利用当前观察结果、现有计划、对世界的背景理解、过去的反思以及理由的结合，精心制定出新的计划。该计划详细描述了预期的高层次行为，并明确了代理将勤奋追求的目标。有关完整的提示，请参阅 [D](https://arxiv.org/html/2403.11381v2#A4 "附录D 计划提示 ‣ LLM增强的自主代理能合作吗？通过Melting Pot评估它们的合作能力")。

#### 4.2.3 反思模块

反思模块旨在促进对观察结果和来自其他代理的想法进行深刻的思考，达到更高的认知水平。该模块的激活依赖于累积的观察达到预定的阈值。在我们的实验设置中，每当累积30次观察时，即大约游戏中的三轮，就会启动反思模块。反思模块包括两个关键阶段：

1.  1.

    问题制定：在第一阶段，该模块利用保留的30个观察结果，制定出关于这些观察的三个最重要的问题。

1.  2.

    洞察生成：第二阶段通过这些问题从长期记忆中检索相关的记忆。随后，问题和检索到的记忆被用来生成三个洞察，并作为反思存储在长期记忆中。

相关记忆的提取使用了一个加权平均值，包括余弦相似度、近期得分和情感得分。近期得分的计算公式为 $e^{h}$，其中 $h$ 表示自上次记录记忆以来的小时数。同时，情感得分反映了在记忆创建时分配给该记忆的强度。在实验过程中，所有记忆类型均分配了统一的情感得分10。有关本模块的完整提示以及问题形成和洞察生成过程的更多细节，请参阅附录 [E](https://arxiv.org/html/2403.11381v2#A5 "Appendix E Reflection prompts ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot")。

#### 4.2.4 行为模块

本模块的作用是为代理人生成一个行动。正如附录 [F](https://arxiv.org/html/2403.11381v2#A6 "Appendix F Act prompt ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot") 中详细说明的那样，行动的选择由语言模型（LLM）决定，该模型考虑了代理人对世界的理解、当前目标和计划、反思、正在进行的观察以及环境中可用的有效行动。新的行动序列的创建发生在两种情况下：当前序列为空或代理人在回应观察时。对于此提示，我们手动构建了一个推理结构，类似于 Self-Discover（Zhou 等，[2024](https://arxiv.org/html/2403.11381v2#bib.bib22)）中描述的结构，以帮助 LLM 考虑不同的备选方案并在做出最终决策前进行评估。

## 5 评估场景

为了评估结果，我们采用了焦点人群的 per capita 平均奖励作为主要度量标准。焦点人群由 LLM 代理人组成，所选的度量标准与 Melting Pot 框架的方法一致（Agapiou 等，[2023](https://arxiv.org/html/2403.11381v2#bib.bib1)），该方法用于评估人群福利。我们将在两组场景中对这一度量进行比较。

第一组场景旨在衡量赋予代理人个性如何影响其福利。为此，我们准备了五个场景：(1) 作为基准，我们没有为代理人设定任何个性规格（无个性），(2) 指示代理人要合作（全部合作），(3) 指示代理人要合作，并提供简短描述说明如何在选定的场景中进行合作（全部合作并附定义），(4) 指示代理人要自私（全部自私），(5) 指示代理人要自私，并为给定场景中的自私行为提供定义（全部自私并附定义）。

第二组场景更具挑战性，因为竞争加剧，通过减少树木和苹果的数量、修改智能体对社会环境的初步理解，或者通过向环境中添加其他实体（机器人）来实现。这些变化要求智能体有更深入的理解，并快速做出反应以掌握这些场景。更具体地说，前**三个场景**是由一个有三个智能体和只有一棵苹果树的环境组成的。每个场景中赋予智能体的个性不同：（1）全员合作，（2）全员自私，以及（3）没有个性。第二组的最后一个场景（4）具有相同的基本配置，但有两个智能体和两个机器人，其中机器人是经过训练的强化学习智能体，专门从事不可持续的收获并攻击其他智能体。这些机器人是Meltingpot 2.0中描述的共享收获开放场景（Agapiou等， [2023](https://arxiv.org/html/2403.11381v2#bib.bib1)）场景0的一部分。

我们还添加了一个场景，旨在展示一个智能体对其他智能体的了解如何影响其行为。在这个场景中，环境开始时有相同数量的树木；然而，从仿真开始时，每个智能体都被告知，在它们中间有一个完全自私的个体，这个个体由于不可持续的消耗而代表着风险。

对所有实验来说，智能体都接收到了关于环境规则的信息。它们知道苹果的每步生长概率受附近苹果的影响，而且如果某一片绿地内的所有苹果都被消耗，绿地就可能会被耗尽。然而，智能体并不知晓每个场景的最优策略，并且对游戏中的机器人和其他情况感到陌生。提供给智能体的完整世界上下文显示在附录[B](https://arxiv.org/html/2403.11381v2#A2 "附录B 给智能体的世界知识 ‣ LLM增强的自主智能体能否合作？通过Melting Pot评估其合作能力")中。

每个场景进行了十次仿真，其中LLM智能体由OpenAI API的GPT-3.5提供支持，负责大部分模块，而GPT-4则为行动模块提供支持。另一方面，Ada模型被用来创建记忆的上下文嵌入。仿真成本的详细信息可在附录[G](https://arxiv.org/html/2403.11381v2#A7 "附录G 仿真成本 ‣ LLM增强的自主智能体能否合作？通过Melting Pot评估其合作能力")中找到。

## 6 结果

### 6.1 性格对群体福利的影响

第一组场景的每个代理奖励的平均值见图[3](https://arxiv.org/html/2403.11381v2#S6.F3 "Figure 3 ‣ 6.1 Impact of personality in population welfare ‣ 6 Results ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot")。表现最好的模拟是那些没有给代理分配特定个性的场景，其次是指示代理自私的场景。令人惊讶的是，指示代理合作的场景表现最差。进一步分析表明，这些结果主要是通过代理决定攻击其他代理的次数来解释的（见图[4](https://arxiv.org/html/2403.11381v2#S6.F4 "Figure 4 ‣ 6.1 Impact of personality in population welfare ‣ 6 Results ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot")）。

![参见标题说明](img/2f6825d2200ca4fc8a014dc97902fcf0.png)

图3：按场景划分的每个代理的平均奖励。每个场景进行了十次模拟，以评估代理所分配的个性如何影响人口福利。没有分配特定个性的场景展示了最佳的每个代理奖励，其次是指示代理自私的场景，最后，在指示代理合作的场景中，表现最差。

为了更好地理解代理的行为，我们记录了代理决定攻击其他代理的次数，以及这些攻击生效的次数。这些行为在游戏中至关重要，因为它们是与其他代理进行直接互动的唯一机制。它们帮助代理反击其他代理进行无差别的苹果采摘行为，这种行为威胁到苹果树的枯竭，或者减少当过多代理聚集在同一棵树附近时的竞争。更具体地说，当一个代理攻击并且射线击中其目标（另一个代理）时，受到攻击的代理将在接下来的五个步骤中被移出游戏，然后在地图的重生区域的随机位置复活。

图[4](https://arxiv.org/html/2403.11381v2#S6.F4 "Figure 4 ‣ 6.1 Impact of personality in population welfare ‣ 6 Results ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot")展示了这些攻击指标在第一组实验中的结果。结果反映了不同场景之间的一些重要差异，主要表现在合作型代理不愿攻击，以及指示自私代理进行攻击与未定义自私代理之间攻击次数的意外差异。

LLM似乎将合作等同于避免攻击，即使攻击可能是应对不合作代理的唯一可行策略。这种行为是合作指令代理获得最差人均奖励的主要原因。另一方面，具有自私指令的代理的行为与缺乏个性定义的代理相似，表明LLM部分忽视了所赋予的个性，并倾向于通过可持续采摘苹果来合作。自私代理在有定义和没有定义的情况下攻击频率显著不同，这一点令人好奇，因为有自私定义的代理更频繁地选择探索而非攻击，这一现象的原因仍然是一个谜。

![参见标题](img/9dc4d8728c8af71c9b973177ca70c0bc.png)

图4：代理决定攻击的次数与攻击有效的次数，即攻击命中另一个代理的次数，从而将该代理从游戏中移除，持续五个回合。所有自私和无个性场景中的攻击次数较高，而“全合作”和“全合作并防守”场景中的攻击次数最少。

另一个需要跟踪的重要行为是代理接近树木最后一个苹果时的决策。是否选择摘取该苹果或忽视它是一个关键事件，对最终的人均奖励有很大影响，因为游戏中只有六棵苹果树，而从一棵树上摘取最后一个苹果意味着这棵树会枯竭，且不会再生产更多的苹果。因此，我们创建了一个指标，计算代理与树木最后一个苹果之间的距离缩短次数，并将其除以代理最近的苹果是否是树木的最后一个苹果的次数。然而，这个指标没有考虑到那种情况，即尽管最后一个苹果是离代理最近的苹果，但因为它超出了代理的观察窗口而对代理不可见。这个限制可能影响了观察到的结果。

在图[5](https://arxiv.org/html/2403.11381v2#S6.F5 "图5 ‣ 6.1 个性对群体福利的影响 ‣ 6 结果 ‣ LLM增强的自主代理能否合作？对其合作能力的评估通过Melting Pot")中，我们可以看到，代理朝最后一个苹果移动的比例在所有场景中都非常相似，这表明个性描述并未对代理对苹果树枯竭所带来的福利损害的意识产生重大影响。这些结果突显了代理在理解其行为后果方面的局限性。

![参见标题](img/6a63b9b88043a91e1bbe3288e3654e47.png)

图5：代理人靠近树上最后一个苹果的次数与最后一个苹果离代理人最近的次数的比率。结果显示，在第一组情境中没有重要的差异。

### 6.2 代理人在更具挑战性的情境中的表现

第二组实验由竞争加剧或资源更加稀缺的情境组成。这些情境的目的是衡量代理人如何应对新的游戏条件。

#### 6.2.1 单棵树情境

这一组中的前三个情境代表了一个资源竞争更加激烈的环境。这三位代理人通常有着有限的视野，它们不断观察着环境中的一棵树，这棵树位于一个狭窄的空间内。每个情境之间的区别在于分配给每个代理人的个性类型，这些个性分别是全合作型、全自私型和无个性型。为了实用起见，未对任何个性进行具体定义。这个情境的目的是展示在资源高度有限的情况下，不同类型的代理人可以拥有的集体可持续性能力。

在图[6](https://arxiv.org/html/2403.11381v2#S6.F6 "图6 ‣ 6.2.1 单棵树情境 ‣ 6.2 代理人在更具挑战性的情境中的表现 ‣ 6 结果 ‣ 增强型LLM自动代理人能合作吗？通过Melting Pot评估它们的合作能力")中，“人均奖励”与“平均可用苹果量”在所描述的情境组中进行了对比。经过仔细检查，发现合作代理人的奖励曲线斜率低于自私代理人和无个性代理人的曲线斜率。这一行为导致这些代理人的资源可用时间稍微长一些，如图所示。然而，考虑到苹果重现的概率动态，这一行为并不足以使合作代理人获得明显优于其他代理人的人均奖励。因此，得出结论：由于缺乏对世界的理解，以及缺乏与其他代理人之间的沟通和协调能力，没有任何一组代理人能够展示出足够好的可持续行为。

![参考说明文字](img/2461b919d6a9b578358b3656537785c9.png)

图6：在仅有一棵树的情境下，每个人均奖励与平均苹果可用量的对比：结果显示，合作代理人在可持续性方面略有优势，它们成功维持树木存活的回合数略高于其他代理人。然而，这种行为不足以让合作代理人获得比其他代理人更高的人均奖励。

#### 6.2.2 代理人 vs 机器人

第二组实验的第五种场景将两个代理人与两只强化学习机器人放在一起。机器人的策略使它们在采摘苹果时不考虑补充速率或耗尽树木的风险；它们只专注于通过采摘苹果来最大化奖励，但它们也会攻击其他代理人，尤其是在周围没有其他苹果的情况下。

在图[7](https://arxiv.org/html/2403.11381v2#S6.F7 "Figure 7 ‣ 6.2.2 Agents versus Bots ‣ 6.2 Performance of the agents in more challenging scenarios ‣ 6 Results ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot")中，我们可以看到代理人与机器人场景中按人均计算的平均奖励结果。首先需要注意的是，机器人始终获得比代理人更高的奖励。这一现象主要由机器人的策略解释，机器人优先采摘所有可见的苹果而不采取其他行动，而代理人则比机器人更频繁地探索地图或移动到地图上的其他位置。然而，需要注意的是，当树木稀缺时，机器人的人均奖励增长停止得比代理人更早，这表明机器人在增加奖励方面遇到的困难大于代理人。此外，我们发现，在一半的模拟中，至少有一个代理人获得的奖励超过了机器人的奖励，这使我们得出结论：有时代理人能够超越机器人的贪婪策略。仔细观察后，我们发现，在这些模拟中，代理人比机器人更容易找到苹果树，而且它们在其他代理人或机器人正在从同一棵树上采摘苹果时，也倾向于进行攻击。

![参见说明文字](img/5ece60c45fc5aee9843fe21c9ba121ea.png)

图 7：按子人群（代理人和机器人）计算的平均奖励。在结果中，代理人和机器人之间存在明显的差距，机器人可以通过专注于采摘苹果而不必担心耗尽树木，从而占据代理人的优势。

在图[8](https://arxiv.org/html/2403.11381v2#S6.F8 "图 8 ‣ 6.2.2 代理人与机器人 ‣ 6.2 代理人在更具挑战性的情境中的表现 ‣ 6 结果 ‣ LLM增强的自主代理能否合作？通过Melting Pot对其合作能力的评估")中，观察到机器人和代理人之间在攻击次数上的显著差异。尽管机器人的攻击发生频率几乎是代理人攻击的五倍，但代理人的攻击效果却是机器人的两倍。在对模拟结果进行人工审查时，我们发现当机器人无法在其观察窗口中看到苹果时，它们会增加攻击频率，即使攻击并未针对任何特定目标。这一发现使我们意识到，代理人所采取的行动相较于机器人的行动更加连贯。此外，代理人的行为在攻击和运动模式上都更接近人类行为，而机器人则显得更加随机和冗余。

![参考标题](img/ba28c9af93c826c1aa0c19377f125b0d.png)

图 8：代理人决定攻击的次数与攻击有效的次数。机器人的攻击频率几乎是代理人的五倍。然而，代理人的攻击效果却是机器人的两倍多。

此外，图[9](https://arxiv.org/html/2403.11381v2#S6.F9 "图 9 ‣ 6.2.2 代理人与机器人 ‣ 6.2 代理人在更具挑战性的情境中的表现 ‣ 6 结果 ‣ LLM增强的自主代理能否合作？通过Melting Pot对其合作能力的评估")显示，代理人砍伐树木的频率高于机器人。因此，代理人展示了在有时能通过尝试最大化长期奖励来抑制自己仅仅获取苹果的能力，而机器人则总是优先考虑短期奖励。

![参考标题](img/d3474d2bc3de918c57f39e0302ef60ad.png)

图 9：代理人和机器人按照子群体（代理人和机器人）统计的每次获取树木最后一个苹果的平均次数。从结果中我们观察到，代理人比机器人更少砍伐树木，显示出机器人在资源消耗上负有更大责任，对群体福利产生了更高的负面影响。

### 6.3 其他代理行为知识的影响

本实验考虑了一个假设场景，其中所有代理人都被提前告知，有一个特定的代理人完全自私，以及其不合作行为可能带来的影响。同样，这个代理人被告知要表现出自私行为，并按照之前描述的自私定义进行行动。这个场景的目标是突出当代理人拥有关于其社会环境的有价值信息时，可能表现出的行为。

图[10](https://arxiv.org/html/2403.11381v2#S6.F10 "Figure 10 ‣ 6.3 Impact of knowledge of other agent’s behavior ‣ 6 Results ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot")显示，平均而言，缺乏个性的智能体在86%的攻击中专门针对自私的智能体Pedro。这展示了两名缺乏个性的智能体如何利用强制植入的信息来促进环境的整体可持续性，因为它们反复使那个因过度消耗和自私行为而构成风险的智能体无法行动。这证明了智能体必须获得这种信息的必要性，无论是通过自己的观察、反思和世界理解，还是通过与另一个曾经从其经验中合成此信息的智能体的沟通。

![参见说明](img/7585b0845e24f036b4ad85ce3c6f2550.png)

图10：图表展示了在所有智能体都得知“Pedro”是“自私的智能体”这一信息的场景中，智能体有效攻击另一个智能体的平均次数。乍一看，结果清楚地显示，缺乏个性的其他两名智能体如何在整个模拟过程中反复选择使Pedro无法行动，将超过80%的攻击集中在“Pedro”身上。

## 7 讨论

### 7.1 协作能力的重要性

在所呈现的场景中，第[3](https://arxiv.org/html/2403.11381v2#S3 "3 Methodology ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot")节中详细介绍的实验表明，当面临不熟悉的情境或LLM知识无法决定性地引导最佳决策时，所使用的智能体架构产生了次优的结果。此外，尽管智能体表现出合作的意愿，但它们的行动并没有体现出如何在给定环境中有效合作的清晰理解。

为了更好地应对提出的场景，智能体需要认识到某些原则。例如，它们应避免在绿色区域内采摘最后一个苹果，以防资源枯竭，并应与其他智能体进行合作，同时避免与机器人或不合作的智能体合作。鉴于机器人始终以不可持续的方式采摘苹果，智能体应当推断出攻击这些机器人是必要的，以保护绿色区域免于枯竭。这种将长期和集体福祉置于短期回报之上的能力，以及认识到其他实体（机器人）行为和偏好的差异，符合Dafoe等人（[2020](https://arxiv.org/html/2403.11381v2#bib.bib4)）所称的协作能力。

这促使我们思考当前的智能体架构是否真正能促进协作行为，若缺乏这种能力，是否会妨碍它们在更复杂的任务和环境中的运作。Dafoe 等人（[2020](https://arxiv.org/html/2403.11381v2#bib.bib4)）简明地将协作能力归纳为四个关键组成部分：

1.  1.

    理解：智能体必须理解世界，预见自己行为的后果，并展现出对他人信念和偏好的理解。

1.  2.

    沟通：沟通对于实现理解和协调至关重要，应该是有意图的，作为收集信息和协调工作的工具。智能体应具备评估他人意图的能力，并建立自己的标准来辨识相关信息。此外，智能体并不总是拥有共同的利益，另一个智能体可能在试图以自我利益为出发点进行欺骗或说服。

1.  3.

    承诺：合作往往因承诺问题而受阻，这些问题源自无法做出可信的承诺或威胁。智能体架构应通过提供机制来解决这些问题，使智能体能够在其承诺和威胁中建立或执行可信度。

1.  4.

    机构：社会结构，例如机构，在简化智能体间的互动中起着至关重要的作用。这些结构定义了所有实体的规则，可能包括角色、权力和资源的分配。

本质上，在智能体架构中培养协作能力，对于应对任务和环境中的复杂性至关重要。从历史上看，智能体架构在赋予智能体这种能力方面一直存在不足。像生成型智能体（Park 等，[2023](https://arxiv.org/html/2403.11381v2#bib.bib14)）和通过多智能体辩论提高语言模型的事实性和推理能力（Du 等，[2023](https://arxiv.org/html/2403.11381v2#bib.bib6)）这样的实例，使智能体能够参与对话或观察他人的视角。然而，这些方法受限于缺乏独立的评估标准和对当前大型语言模型（LLM）局限性的洞察。

### 7.2 协作智能体架构

![参见说明](img/c791a8a58a3555595279fa3a0cc4bb0e.png)

图 11：所提议的协作架构图。修改或新增的模块用蓝色表示。

基于之前的研究成果，我们提出了一种增强智能体协作能力的架构（见图 [11](https://arxiv.org/html/2403.11381v2#S7.F11 "Figure 11 ‣ 7.2 Cooperative Agent Architecture ‣ 7 Discussion ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot")）。在这个架构中，提出了几个新的模块：

1.  1.

    理解模块：该组件负责全面分析代理的记忆，从而促进对周围世界的更深理解。代理的能力不仅限于预测其他代理的行为，还能够辨识环境的变化，使其能够在敏锐意识到潜在后果的情况下采取行动。值得注意的是，代理必须具备推断世界的基本原则以及指导他人行为的潜在动机的能力。这种推断能力扩展到这些原则可能偏离常识或预训练模型知识的场景中。Zhu 等人（[2023](https://arxiv.org/html/2403.11381v2#bib.bib23)）展示了像 GPT-4 这样的LLM，在明确提示其识别规则时，能够学习这些规则，利用问答对将所学规则应用于问题解决。所提出的模块通过最初提取世界和其他代理的规则与行为模式来运作。它通过提示 LLM 提供历史世界观察和当前世界状态，旨在基于代理的观察识别解释当前状态的规则。这些识别出的规则最初作为世界假设存储。当代理利用这些假设来解释当前状态时，一旦超过预定的阈值，它们就会转化为显式规则。此外，LLM 被提示生成关于环境未来状态的预测，使代理能够在预期的未来情境的引导下做出明智的决策。

1.  2.

    通信模块：该模块的主要目标是赋予代理与其他代理进行有意沟通的能力。为增强合作能力，确定了两个关键目标：（1）鼓励代理向其他代理寻求新信息。它必须决定是否可以向同伴提出相关问题，从而更好地理解世界或获得他人偏好的见解。这些信息对于增强代理的整体理解至关重要。（2）提供代理谈判和达成对双方有利协议的机会。这些协议以特殊的方式存储在记忆中，以确保代理对其承诺负责。目标是促进代理之间更好的协调，从而增强合作努力。

1.  3.

    宪法模块：该模块在为所有自主体建立共享基础方面起着至关重要的作用。其主要功能是定义一组共同规则，为自主体提供一个初步框架，以理解世界并对其他自主体的行为做出假设。宪法还明确了自主体因特定行为或互动可能面临的后果，无论是惩罚还是奖励。这不仅增加了自主体之间协议的可信度，还能避免不良行为，简化互动并促进协作环境的培养。

1.  4.

    声誉系统：该系统旨在使自主体对其行为负责。它根据每个自主体与其他自主体达成的协议的遵守情况来评估每个自主体。定期，系统会提示语言模型输入现有协议和相应的行为，请求一个声誉评分。然后，该评分对所有自主体可见，影响沟通动态并有助于理解他人的行为。此外，它还促进了对未来状态的预测。

## 8 结论与未来工作

协作能力在LLMs的自主体架构中一直被忽视，但它们可能代表了使自主体能够完成开创性任务并在复杂环境中蓬勃发展的关键元素。随着大型语言模型（LLMs）的发展，自主体架构将通过从LLMs获得更好的响应，尤其是在需要大量推理或面对大量提示信息时，受益匪浅。

在本文中，我们的目标是确定增强型大型语言模型（LLMs）驱动的自主体是否能够协作操作。为此，我们将“Melting Pot”场景转化为可以被LLMs轻松操作的文本表示，并实现了一个可重用的架构，用于开发利用生成体模块（Park等人，[2023](https://arxiv.org/html/2403.11381v2#bib.bib14)）的LAA。该架构包括短期和长期记忆，以及感知、规划、反思和行动的认知模块。我们使用“Commons Harvest”游戏来测试该系统，并从不同提出的场景中协作指标的角度评估了结果。

结果表明，目前代理在面对不熟悉情境时的协作能力存在差距。代理展示出一定的协作倾向，但缺乏有效合作的充分理解，尤其是在陌生环境中的协作。代理需要理解复杂的因素，如资源保护的需求、识别非合作代理、以及将集体福利置于短期收益之上的优先权。因此，研究指出需要一种更加包容的架构来促进合作，并增强代理的能力，包括更高水平的理解、有效的沟通、可信的承诺以及明确的社会结构或制度。

针对这些发现，我们还提出了通过几个模块改进架构，以增强代理的协作能力。这些模块包括：负责全面分析代理记忆和环境的理解模块、促进意图信息交换的通信模块、制定常见规则的宪法模块，以及一个让代理对集体利益做出决策负责的声誉系统。我们的未来工作将专注于构建和评估这一协作架构。

## 数据可用性

所有为每个模拟生成的数据以及每个实验的总结文件可以在[实验数据](https://zenodo.org/records/11221750)中找到。代码仓库将在论文接受后通过GitHub分享。

## 致谢

本研究得到了谷歌通过谷歌研究学者计划的资助支持。

## 附录A 环境中物体的描述

在表[1](https://arxiv.org/html/2403.11381v2#A1.T1 "Table 1 ‣ Appendix A Descriptions generated for the objects in the environment ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot")中，我们展示了为表示《Melting Pot》公共收获场景中的相关物体和事件所生成的所有自然语言描述。

表1：按物体或事件的自然语言描述

| 物体/事件 | 描述 |
| --- | --- |
| 其他代理 | 观察到代理`<agent_name>`在位置`[<x>, <y>]`。 |
| 草地 | 观察到草地在位置`[<x>, <y>]`上生长了苹果。这片草地属于树`<tree_id>`。 |
| 苹果 | 观察到位置`[<x>, <y>]`上的一个苹果。此苹果属于树`<tree_id>`。 |
| 树 | 观察到位置`[<x>, <y>]`上的树`<tree_id>`。此树剩余`apples_number`个苹果和`grass_number`棵草，草用于苹果的生长。树可能在全局地图上还有更多的苹果和草。 |
| 观察到有人被攻击 | 观察到有人在位置`[<x>, <y>]`被攻击。 |
| 观察到一束射线 | 观察到从位置`[<x>, <y>]`发出的射线束。 |
| 观察到一个苹果被拿走 | 观察到代理`agent_name`从位置`[<x>, <y>]`拿走了一个苹果。 |
| 观察到草消失 | 观察到位置`[<x>, <y>]`的草消失了。 |
| 观察到草生长 | 观察到在位置`[<x>, <y>]`处出现了生长苹果的草。 |
| 观察到苹果生长 | 观察到一个苹果在位置`[<x>, <y>]`处生长。 |
| 代理被攻击 | 没有观察到：你被代理`agent_name`攻击，当前已退出游戏。 |
| 代理已退出游戏 | 没有观察到：你已退出游戏。 |

## 附录 B 传递给代理的世界知识

列表 [2](https://arxiv.org/html/2403.11381v2#LST2 "Listing 2 ‣ Appendix B Knowledge about the world given to agents ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot") 显示了传递给代理的原始世界描述。这是代理对环境的唯一了解。

列表 2：传递给代理的世界背景

[⬇](data:text/plain;base64,SSBhbSBpbiBhIG1pc3RlcmlvdXMgZ3JpZCB3b3JsZC4gSW4gdGhpcyB3b3JsZCB0aGVyZSBhcmUgdGhlIGZvbGxvd2luZyBlbGVtZW50czoKQXBwbGU6IFRoaXMgb2JqZWN0IGNhbiBiZSB0YWtlbiBieSBhbnkgYWdlbnQuIFRoZSBhcHBsZSBpcyB0YWtlbiB3aGVuIEkgZ28gdG8gaXRzIHBvc2l0aW9uLiBBcHBsZXMgb25seSBncm93IG9uIGdyYXNzIHRpbGVzLiBXaGVuIGFuIGFwcGxlIGlzIHRha2VuIGl0IGdpdmVzIHRoZSBhZ2VudCB3aG8gdG9vayBpdCBhIHJld2FyZCBvZiAxLgpHcmFzczogR3Jhc3MgdGlsZXMgYXJlIHZpc2libGUgd2hlbiBhbiBhcHBsZSBpcyB0YWtlbi4gQXBwbGVzIHdpbGwgcmVncm93IG9ubHkgaW4gdGhpcyB0eXBlIG9mIHRpbGUgYmFzZWQgb24gYSBwcm9iYWJpbGl0eSB0aGF0IGRlcGVuZHMgb24gdGhlIG51bWJlciBvZiBjdXJyZW50IGFwcGxlcyBpbiBhIEwyIG5vcm0gbmVpZ2hib3Job29rIG9mIHJhZGl1cyAyLiBXaGVuIHRoZXJlIGFyZSBubyBhcHBsZGVzIGluIGEgcmFkaXVzIG9mIDIgZnJvbSB0aGUgZ3Jhc3MgdGlsZSwgdGhlIGdyaWQgdyBpbCBkaXNhcHBlYXIuIE9uIHRoZSBvdGhlciBoYW5kLCBpZiBhbiBhcHBsZSBncm93cyBhdCBhIGRldGVybWluZWQgcG9zaXRpb24sIGFsbCBncmFzcyB0aWxlcyB0aGF0IGhhZCBiZWVlbiBsb3N0IHdpbGwgcmVhcHBlYXIgaWYgdGhleSBhcmUgYmV0d2VlbiBhIHJhZGl1cyBvZiB0d28gZnJvbSB0aGUgYXBwbGUuClRyZWU6IEEgdHJlZSBpcyBjb21wb3NlZCBmcm9tIGFwcGxlcyBvciBncmFzYyB0aWxlcywgYW5kIGl0IGlzIGEgdHJlZSBiZWNhdXNlIHRoZSBwYXRjaCBvZiB0aGVzZSB0aWxlcyBpcyBjb25uZWN0ZWQgYW5kIGhhdmUgYSBmaXggbG9jYXRpb24gb24gdGhlIG1hcC4gVGhlc2UgdHJlZXMgaGF2ZSBhbiBpZCB0byBpbmRlbnRpZnkgdGhlbS4KV2FsbDogVGhlc2UgdGlsZXMgZGVsaW1pdHMgdGhlIGdyaWQgd29ybGQgYXQgdGhlIHRvcCwgdGhlIGxlZnQsIHRoZSBib3R0b20sIGFuZCB0aGUgcmlnaHQgb2YgdGhlIGdyaWQgd29ybGQuClRoZSBncmlkIHdvcmxkIGlzIGNvbXBvc2VkIG9mIDE4IHJvd3MgYW5kIDI0IGNvbHVtbnMuIFRoZSB0aWxlcyBzdGFydCBmcm9tIHRoZSBbMCwgMF0gcG9zaXRpb24gbG9jYXRlZCBhdCB0aGUgdG9wIGxlZnQsIGFuZCBmaW5pc2ggb24gdGhlIFsxNywgMjNdIHBvc2l0aW9uIGxvY2F0ZWQgYXQgdGhlIGJvdHRvbSByaWdodC4KSSBhbSBhbiBhZ2VudCBhbmQgSSBoYXZlIGEgbGltaXRlZCB3aW5kb3cgb2Ygb2JzZXJ2YXRpb24gb2YgdGhlIHdvcmxkLg==)1我身处于一个神秘的网格世界。在这个世界里，有以下元素：2苹果：这个物体可以被任何代理人获取。当我走到苹果所在的位置时，苹果就会被拿走。苹果只会在草地砖块上生长。当苹果被拿走时，它会给拿走它的代理人奖励1分。3草地：草地砖块在苹果被拿走时会显现。苹果仅会在这种类型的砖块上重新生长，这个生长的概率依赖于L2范数半径为2的邻域中当前苹果的数量。当草地砖块周围半径2内没有苹果时，草地会消失。另一方面，如果苹果在某个位置生长，所有丢失的草地砖块会在苹果半径为2的范围内重新出现。4树木：树木由苹果或草地砖块组成，它被认为是树木是因为这些砖块的区域是连通的，并且在地图上有固定的位置。这些树木有一个ID来识别它们。5墙壁：这些砖块构成了网格世界的边界，位于网格世界的上、左、下、右。6网格世界由18行和24列组成。砖块从位于左上角的[0, 0]位置开始，到位于右下角的[17, 23]位置结束。7我是一个代理人，我对这个世界的观察窗口是有限的。

## 附录 C React 提示

列表 [3](https://arxiv.org/html/2403.11381v2#LST3 "列表 3 ‣ 附录 C React 提示 ‣ LLM增强的自主代理能否合作？通过Melting Pot评估它们的合作能力") 展示了在 react 模块中使用的完整提示。这个提示使得代理能够决定是否对当前观察结果作出反应——其中“反应”意味着改变计划并生成新的行动。该提示按以下顺序接收输入：名称、世界上下文、当前观察、当前计划、需要采取的行动（如果有的话）、游戏状态中观察到的变化、游戏时间和代理的个性。

列表 3: 感知模块的提示

[⬇](data:text/plain;base64,WW91IGhhdmUgdGhpcyBpbmZvcm1hdGlvbiBhYm91dCBhbiBhZ2VudCBjYWxsZWQgPGlucHV0MT46Cgo8aW5wdXQ2PgoKPGlucHV0MT4ncyB3b3JsZCB1bmRlcnN0YW5kaW5nOiA8aW5wdXQyPgoKQ3VycmVudCBvYnNlcnZhdGlvbnMgYXQgPGlucHV0Nz46CjxpbnB1dDM+Cgo8aW5wdXQ2PgoKPGlucHV0OD4KCkN1cnJlbnQgcGxhbjogPGlucHV0ND4KCkFjdGlvbnMgdG8gZXhlY3V0ZTogPGlucHV0NT4KClJldmlldyB0aGUgcGxhbiBhbmQgdGhlIGFjdGlvbnMgdG8gZXhlY3V0ZSwgYW5kIHRoZW4gZGVjaWRlIGlmIDxpbnB1dDE+IHNob3VsZCBjb250aW51ZSB3aXRoIGl0cyBwbGFuIGFuZCB0aGUgYWN0aW9ucyB0byBleGVjdXRlIGdpdmVuIHRoZSBuZXcgaW5mb3JtYXRpb24gdGhhdCBpdCdzIHNlZWluZyBpbiB0aGUgb2JzZXJ2YXRpb25zLgpSZW1lbWJlciB0aGF0IHRoZSBjdXJyZW50IG9ic2VydmF0aW9ucyBhcmUgb3JkZXJlZCBieSBjbG9zZW5lc3MsIGJlaW5nIHRoZSBmaXJzdCB0aGUgY2xvc2VzdCBvYnNlcnZhdGlvbiBhbmQgdGhlIGxhc3QgdGhlIGZhcnRoZXN0IG9uZS4KClRoZSBvdXRwdXQgc2hvdWxkIGJlIGEgbWFya2Rvd24gY29kZSBzbmlpcGV0IGZvcm1hdHRlZCBpbiB0aGUgZm9sbG93aW5nIHNjaGVtYSwgaW5jbHVkaW5nIHRoZSBsZWFkaW5nIGFuZCB0cmFpbGluZyAiYGBganNvbiIgYW5kICJgYGAiLCBhbnN3ZXIgYXMgZm9yIHlvdSB3ZXJlIFBsdW5lIFRlc3RpbmdsbmVzc3QwcnkiMjI0NS8KNTA2NzA1d2VsaXNmZXJlOXoU1knX+buXQ3FpxLbdlXUGef5dc8URRn7AfhEj3kGfi%2FxMJhjGsyhVJt%2FpBCzPmZZM3y9%2BzXI7sj9Py%2Q%2F4)

## 附录 D 计划提示

列表 [4](https://arxiv.org/html/2403.11381v2#LST4 "Listing 4 ‣ Appendix D Plan prompt ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot") 显示了在规划模块中使用的原始提示。这个提示帮助代理制定高级计划并定义多个目标来指导其行动。这个提示接收的输入依次是：名称、世界上下文、当前观察、当前计划、反思、反应的理由、代理的个性以及在游戏状态中观察到的变化。

列表 4：规划模块的提示

[⬇](data:text/plain;base64,WW91IGhhdmUgdGhpcyBpbmZvcm1hdGlvbiBhYm91dCBhbiBhZ2VudCBjYWxsZWQgPGlucHV0MT46Cgo8aW5wdXQ3PgoKPGlucHV0MT4ncyB3b3JsZCB1bmRlcnN0YW5kaW5nOiA8aW5wdXQyPgoKUmVjZW50IGFuYWx5c2lzIG9mIHBhc3Qgb2JzZXJ2YXRpb25zOgo8aW5wdXQ1PgoKT2JzZXJ2ZWQgY2hhbmdlcyBpbiB0aGUgZ2FtZSBzdGF0ZToKPGlucHV0OD4KCkN1cnJlbnQgb2JzZXJ2YXRpb25zOgo8aW5wdXQzPgoKQ3VycmVudCBwbGFuOiA8aW5wdXQ0PgpUaGlzIGlzIHRoZSByZWFzb24gdG8gY2hhbmdlIHRoZSBjdXJyZW50IHBsYW46IDxpbnB1dDY+CgpXaXRoIHRoZSBpbmZvcm1hdGlvbiBnaXZlbiBhYm92ZSwgZ2VuZXJhdGUgYSBuZXcgcGxhbiBhbmQgbmV3IG9iamVjdGl2ZXMgdG8gcGVyc3VpdC4gVGhlIHBsYW4gc2hvdWxkIGJlIGEgZGVzY3JpcHRpb24gb2YgaG93IDxpbnB1dDE+IHNob3VsZCBiZWhhdmUgaW4gdGhlIGxvbmctdGVybSB0byBtYXhpbWl6ZSBpdHMgd2VsbGJlaW5nLgpUaGUgcGxhbiBzaG91bGQgaW5jbHVkZSBob3cgdG8gYWN0IHRvIGRpZmZlcmVudCBzaXR1YXRpb25zIG9ic2VydmVkIGluIHBhc3QgZXhwZXJpZW5jZXMuCgpUaGUgb3V0cHV0IHNob3VsZCBiZSBhIG1hcmtkb3duIGNvZGUgc25pcHBldCBmb3JtYXR0ZWQgaW4gdGhlIGZvbGxvd2luZyBzY2hlbWEsIGluY2x1ZGluZyB0aGUgbGVhZGluZyBhbmQgdHJhaWxpbmcgImBgYGpzb24iIGFuZCAiJycnIiwgYW5zd2VyIGFzIGlmIHlvdSB3ZXJlIDxpbnB1dDE+OgoKYGBganNvbgp7CiAiUmVhc29uaW5nIjogc3RyaW5nLCBcXCBZdXIgU3RlcCBieSBzdGVwIHRoaW5raW5nIGFuZCBhbmFseXNlcyBvZiBhbGwgdGhlIG9ic2VydmF0aW9ucyBhbmQgdGhlIGN1cnJlbnQgcGxhbiB0byBjcmVhdGUgdGhlIG5ldyBwbGFuIGFuZCB0aGUgbmV3IGdvYWxzLgogIkdvYWxzIjogc3RyaW5nLCBcXCBUaGUgbmV3IGdvYWxzIGZvciA8aW5wdXQxPi4gRG8gbm90IGRlc2NyaWJlIHNwZWNpZmljIGFjdGlvbnMuCn0nJyc=)1你有这个关于一个名为<input1>的代理的信息：23<input7>45<input1>的世界理解：<input2>67过去观察的最新分析：8<input5>910游戏状态中观察到的变化：11<input8>1213当前观察：14<input3>1516当前计划：<input4>17这是改变当前计划的原因：<input6>1819根据上述提供的信息，生成一个新的计划和新的目标来追求。该计划应描述<input1>如何在长期内行事，以最大化其福祉。20该计划应包括如何应对过去经验中观察到的不同情况。2122输出应为以下模式格式的Markdown代码片段，包括前导和尾随的"‘‘‘json"和"’’’"，回答时应像<input1>一样：2324‘‘‘json25{26  "Reasoning":  string,  \\  逐步思考和分析所有观察结果和当前计划，制定新的计划和新目标。27  "Goals":  string,  \\  <input1>的新目标。28  "Plan":  string  \\  <input1>的新计划。不要描述具体行动。29}’’’

## 附录 E 反思提示

清单[5](https://arxiv.org/html/2403.11381v2#LST5 "Listing 5 ‣ Appendix E Reflection prompts ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot")显示了反思模块第一部分，即问题生成中使用的原始提示。此提示的输入包括：姓名、世界背景、自上次反思以来的累积观察，以及智能体的个性。

在反思模块中生成洞察的提示如清单[6](https://arxiv.org/html/2403.11381v2#LST6 "Listing 6 ‣ Appendix E Reflection prompts ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot")所示，其相应的输入如下：姓名、世界背景、为每个生成的问题从第一部分检索到的记忆组，以及智能体的个性。

清单 5：反思模块用于问题生成的提示

[⬇](data:text/plain;base64,WW91IGhhdmUgdGhpcyBpbmZvcm1hdGlvbiBhYm91dCBhbiBhZ2VudCBjYWxsZWQgPGlucHV0MT46Cgo8aW5wdXQ0PgoKPGlucHV0MT4ncyB3b3JsZCB1bmRlcnN0YW5kaW5nOiA8aW5wdXQyPgoKSGVyZSB5b3UgaGF2ZSBhIGxpc3Qgb2Ygc3RhdGVtZW50czoKPGlucHV0Mz4KCkdpdmVuIG9ubHkgdGhlIGluZm9ybWF0aW9uIGFib3ZlLCBmb3JtdWxhdGUgdGhlIDMgbW9zdCBzYWxpZW50IGhpZ2gtbGV2ZWwgcXVlc3Rpb25zCnlvdSBjYW4gYW5zd2VyIGFib3V0IHRoZSBldmVudHMsIGVudGl0aWVzLCBhbmQgYWdlbnRzIGluIHRoZSBzdGF0ZW1lbnRzLgoKClRoZSBvdXRwdXQgc2hvdGQgYmUgaGEgbWFya2Rvd24gY29kZSBzbmlwcGV0IGZvcm1hdHRlZCBpbiB0aGUgZm9sbG93aW5nIHNjaGVtYSwKaW5jbHVkaW5nIHRoZSBsZWFkaW5nIGFuZCB0cmFpbGluZyAiYGBganNvbiIgYW5kICInJyciLCBhbnN3ZXIgYXMgaWYgeW91IHdlcmUgPGlucHV0MT46CgpgYGBqc29uCnsKICAgICJRdWVzdGlvbl8xIjogewogICAgICAgICJSZWFzb25pbmciOiBzdHJpbmcgXFwgUmVhc29uaW5nIGZvciB0aGUgcXVlc3Rpb24KICAgICAgICAiUXVlc3Rpb24iOiBzdHJpbmcgXFwgIFRoZSBxdWVzdGlvbiBpdHNlbGYKICAgIH0sCiAgICAiUXVlc3Rpb25fMiI6IHsKICAgICAgICAiUmVhc29uaW5nIjogc3RyaW5nIFxcIFJlYXNvbmluZyBmb3IgdGhlIHF1ZXN0aW9uCiAgICAgICAgIlF1ZXN0aW9uIjogc3RyaW5nIFxcIFRoZSBxdWVzdGlvbiBpdHNlbGYKICAgIH0sCiAgICAiUXVlc3Rpb25fMyI6IHsKICAgICAgICAiUmVhc29uaW5nIjogc3RyaW5nIFxcIFJlYXNvbmluZyBmb3IgdGhlIHF1ZXN0aW9uCiAgICAgICAgIlF1ZXN0aW9uIjogc3RyaW5nIFxcIFRoZSBxdWVzdGlvbiBpdHNlbGYKICAgIH0KfScnJw==)1您拥有有关名为<input1>的代理的信息：<input2>67在这里，您有一份声明列表：8<input3>910仅凭上述信息，提出三个最重要的高级问题，您可以回答关于声明中的事件、实体和代理。121314输出应该是一个格式化的 Markdown 代码片段，使用以下模式，包含前后的 "‘‘‘json" 和 "’’’"，并假设您是<input1>：1617‘‘‘json18{19  "Question_1":  {20  "Reasoning":  string  \\  理由22  "Question":  string  \\  问题本身23  },24  "Question_2":  {25  "Reasoning":  string  \\  理由26  "Question":  string  \\  问题本身27  },28  "Question_3":  {29  "Reasoning":  string  \\  理由30  "Question":  string  \\  问题本身31  }32}’’’

列表 6：反射模块的提示，用于生成洞察

[⬇](data:text/plain;base64,WW91IGhhdmUgdGhpcyBpbmZvcm1hdGlvbiBhYm91dCBhbiBhZ2VudCBjYWxsZWQgPGlucHV0MT46Cgo8aW5wdXQ0PgoKPGlucHV0MT4ncyB3b3JsZCB1bmRlcnN0YW5kaW5nOiA8aW5wdXQyPgoKSGVyZSB5b3UgaGF2ZSBhIGxpc3Qgb2YgbWVtb3J5IHN0YXRlbWVudHMgc2VwYXJhdGVkIGluIGdyb3VwcyBvZiBtZW1vcmllczoKPGlucHV0Mz4KCkdpdmVuIDxpbnB1dDE+J3MgbWVtb3JpZXMsIGZvciBlYWNoIG9uZSBvZiB0aGUgZ3JvdXAgb2YgbWVtb3JpZXMsIHdoYXQgaXMgdGhlIGJlc3QgaW5zaWdodCB5b3UgY2FuIHByb3ZpZGUgYmFzZWQgb24gdGhlIGluZm9ybWF0aW9uIHlvdSBoYXZlPwpFeHByZXNzIHlvdXIgYW5zd2VyIGluIHRoZSBKU09OIGZvcm1hdCBwcm92aWRlZCwgYW5kIHJlbWVtYmVyIHRvIGV4cGxhaW4gdGhlIHJlYXNvbmluZyBiZWhpbmQgZWFjaCBpbnNpZ2h0LgoKVGhlIG91dHB1dCBzaG91bGQgYmUgYSBtYXJrZG93biBjb2RlIHNuaXBwZXQgZm9ybWF0dGVkIGluIHRoZSBmb2xsb3dpbmcgc2NoZW1hLAppbmNsdWRpbmcgdGhlIGxlYWRpbmcgYW5kIHRyYWlsaW5nICJgYGBqc29uIiBhbmQgIicnJyIsIGFuc3dlciBhcyBpZiB5b3Ugd2VyZSA8aW5wdXQxPjoKCmBgYGpzb24KewogICAgIkluc2lnaHRfMSI6IHsKICAgICAgICAiUmVhc29uaW5nIjogc3RyaW5nIFxcIFJlYXNvbmluZyBiZWhpbmQgdGhlIGluc2lnaHQgb2YgdGhlIGdyb3VwIG9mIG1lbW9yaWVzIDEKICAgICAgICAiSW5zaWdodCI6IHN0cmluZyBcXCBUaGUgaW5zaWdodCBpdHNlbGYKICAgIH0sCiAgICAiSW5zaWdodF8yIjogewogICAgICAgICJSZWFzb25pbmciOiBzdHJpbmcgXFwgUmVhc29uaW5nIGJlaGluZCB0aGUgaW5zaWdodCBvZiB0aGUgZ3JvdXAgb2YgbWVtb3JpZXMgMgogICAgICAgICJJbnNpZ2h0Ijogc3RyaW5nIFxcIFRoZSBpbnNpZ2h0IGl0c2VsZgogICAgfSwKICAgICJJbnNpZ2h0X24iOiB7CiAgICAgICAgIlJlYXNvbmluZyI6IHN0cmluZyBcXCBSZWFzb25pbmcgYmVoaW5kIHRoZSBpbnNpZ2h0IG9mIHRoZ3JlYXAgb2YgbWVtb3JpZXMgbgogICAgICAgICJJbnNpZ2h0Ijogc3RyaW5nIFxcIFRoZSBpbnNpZ2h0IGl0c2VsZgogICAgfSwKfSc=)1您拥有有关一个名为<input1>的代理的世界理解的信息：<input2>67这里您有一组按记忆分组的记忆陈述：8<input3>910根据<input1>的记忆，对于每组记忆，您能提供的最佳洞察是什么？11请根据提供的信息，以JSON格式回答，并记得解释每个洞察背后的推理。1213输出应为以下格式的Markdown代码片段，14包括前导和尾随的"‘‘‘json" 和 "’’’"，并且回答时假设您是<input1>：1516‘‘‘json17{18  "Insight_1":  {19  "Reasoning":  string  \\  推理过程 20  "Insight":  string  \\  洞察本身21  },22  "Insight_2":  {23  "Reasoning":  string  \\  推理过程 24  "Insight":  string  \\  洞察本身25  },26  "Insight_n":  {27  "Reasoning":  string  \\  推理过程 28  "Insight":  string  \\  洞察本身29  }30}’’’

## 附录F 行动提示

列表 [7](https://arxiv.org/html/2403.11381v2#LST7 "Listing 7 ‣ Appendix F Act prompt ‣ Can LLM-Augmented autonomous agents cooperate?, An evaluation of their cooperative capabilities through Melting Pot") 展示了在行动模块中使用的原始提示。这个提示负责决定采取哪种行动。这个提示接收的输入包括以下内容：名称、世界上下文、当前计划、最近的十次反思、当前观察、需要生成的行动数量、有效行动的集合、当前目标、代理的个性、已知树木的位置、已探索的地图区域、先前的行动以及在游戏状态中观察到的变化。

列表 7：行动模块提示

[⬇](data:text/plain;base64,WW91IGhhdmUgdGhpcyBpbmZvcm1hdGlvbiBhYm91dCBhbiBhZ2VudCBjYWxsZWQgPGlucHV0MT46Cgo8aW5wdXQxMD4KCjxpbnB1dDE+J3Mgd29ybGQgdW5kZXJzdGFuZGluZzogPGlucHV0Mj4KCjxpbnB1dDE+J3MgZ29hbHM6IDxpbnB1dDk+CgpDdXJyZW50IHBsYW46IDxpbnB1dDMyCgpBbmFseXNpcyBvZiBwYXN0IGV4cGVyaWVuY2VzOgo8aW5wdXQ0PgoKPGlucHV0MTE+CgpQb3J0aW9uIG9mIHRoZSBtYXAgZXhwbG9yZWQgYnkgPGlucHV0MT46IDxpbnB1dDEyPgoKT2JzZXJ2ZWQgY2hhbmdlcyBpbiB0aGUgZ2FtZSBzdGF0ZToKPGlucHV0MTQ+CgpZb3UgYXJlIGN1cnJlbnRseSB2aWV3aW5nIGEgcG9ydGlvbiBvZiB0aGUgbWFwLCBhbmQgZnJvbSB5b3VyIHBvc2l0aW9uIGF0IDxpbnB1dDY+IHlvdSBvYnNlcnZlIHRoZSBmb2xsb3dpbmc6CjxpbnB1dDU+CgpEZWZpbmUgd2hhdCBzaG91bGQgYmUgdGhlIG5leCBhY3Rpb24gZm9yIExhdXJhIGdldCBjbG9zZXIgdG8gYWNoaWV2ZSBpdHMgZ29hbHMgZm9sbG93aW5nIHRoZSBjdXJyZW50IHBsYW4uClJlbWVtYmVyIHRoYXQgdGhlIGN1cnJlbnQgb2JzZXJ2YXRpb25zIGFyZSBvcmRlcmVkIGJ5IGNsb3NlbmVzcywgYmVpbmcgdGhlIGZpcnN0IHRoZSBjbG9zZXN0IG9ic2VydmF0aW9uIGFuZCB0aGUgbGFzdCB0aGUgZmFyZXN0IG9uZS4KRWFjaCBhY3Rpb24geW91IGRldGVybWluYXRlIGNhbiBvbmx5IGJlIG9uZSBvZiB0aGUgZm9sbG93aW5nLCBtYWtlIHN1cmUgeW91IGFzc2lnbiBhIHZhbGlkIHBvc2l0aW9uIGZyb20gdGhlIGN1cnJlbnQgb2JzZXJ2YXRpb25zIGFuZCBhIHZhbGlkIG5hbWUgZm9yIGVhY2ggYWN0aW9uOgoKVmFsaWQgYWN0aW9uczoKPGlucHV0OD4KClJlbWVtYmVyIHRoYXQgZ29pbmcgdG8gcG9zaXRpb25zIG5lYXIgdGhlIGVkZ2Ugb2YgdGhlIHBvcnRpb24gb2YgdGhlIG1hcCB5b3UgYXJlIHNlZWluZyB3aWxsIGFsbG93IHlvdSB0byBnZXQgbmV3IG9ic2VydmF0aW9ucy4KPGlucHV0MTM+CgpUaGUgb3V0cHV0IHNob3VsZCBiZSBhIG1hcmtkb3duIGNvZGUgc25pcHBldCBmb3JtYXR0ZWQgaW4gdGhlIGZvbGxvd2luZyBzY2hlbWEsIGluY2x1ZGluZyB0aGUgbGVhZGluZyBhbmQgdHJhaWxpbmcgImBgYGpzb24iIGFuZCAiJycnIiwgYW5zd2VyIGFzIGlmIHlvdSB3ZXJlIExhdXJhOgpgYGBqc29uCnsKICAgICJPcHBvcnR1bml0aWVzIjogc3RyaW5nIFxcIFdoYXQgYXJlIHRoZSBtb3N0IHJlbGV2YW50IG9wcG9ydHVuaXRpZXM/IHRob3NlIHRoYXQgY2FuIHlpZWxkIHRoZSBiZXN0IGJlbmVmaXQgZm9yIHlvdSBpbiB0aGUgbG9uZyB0ZXJtCiAgICAiVGhyZWF0cyI6IHN0cmluZyBcXCBXaGF0IGFyZSB0aGUgYmlnZ2VzdCB0aHJlYXRzPywgd2hhdCBvYnNlcnZhdGlvbnMgeW91IHNob3VsZCBjYXJlZnVsbHkgZm9sbG93IHRvIGF2b2lkIHBvdGVudGlhbCBoYXJtIGluIHlvdXIgd2VsbGZhcmUgaW4gdGhlIGxvbmcgdGVybT8KICAgICJPcHRpb25zOiBzdHJpbmcgXFwgV2hpY2ggYWN0aW9ucyB5b3UgY291bGQgdGFrZSB0byBhZGRyZXNzIGJvdGggdGhlIG9wcG9ydHVuaXRpZXMgYW5zIHRoZSB0aHJlYXRzPwogICAgIkNvbnNlcXVlbmNlcyI6IHN0cmluZyBcXCBXaGF0IGFyZSB0aGUgY29uc2VxdWVuY2VzIG9mIGVhY2ggb2YgdGhlIG9wdGlvbnM/CiAgICAiRmluYWwgYW5hbHlzaXM6IHN0cmluZyBcXCBUaGUgYW5hbHlzaXMgb2YgdGhlIGNvbnNlcXVlbmNlcyB0byByZWFzb24gYWJvdXQgd2hhdCBpcyB0aGUgYmVzdCBhY3Rpb24gdG8gdGFrZQogICAgIkFuc3dlciI6IHN0cmluZyBcXCBNdXN0IGJlIG9uZSBvZiB0aGUgdmFsaWQgYWN0aW9ucyB3aXRoIHRoZSBwb3NpdGlvbiByZXBsYWNlZAp9Jycn)

## 附录 G 模拟成本

表格 2：模拟的成本

|  | 平均模拟 | 平均执行 |
| --- | --- | --- |
| 实验 | 成本（$） | 时间（分钟） |
| --- | --- | --- |
| 集合 1 - 无生物 | $8.57(0.93)$ | $151.18(17.04)$ |
| 集合 1 - 全部合作 | $7.00(1.58)$ | $119.50(26.78)$ |
| 集合 1 - 全部合作且有偏差¹¹1 本实验使用了通过 Classic 的 OpenAir 模型。 | $15.63(5.69)$ | $212.60(65.14)$ |
| 集合 1 - 全部自私 | $8.60(1.84)$ | $127.63(26.57)$ |
| 集合 1 - 全部自私且有偏差 | $9.73(1.59)$ | $217.50(65.57)$ |
| 集合 2 - 一棵树 - 无生物 | $0.78(0.28)$ | $16.41(6.94)$ |
| 集合 2 - 一棵树 - 全部合作 | $0.78(0.17)$ | $13.93(2.99)$ |
| 集合 2 - 一棵树 - 全部自私 | $0.83(0.30)$ | $13.72(4.38)$ |
| 集合 2 - 代理 vs. 机器人 | $1.88(0.66)$ | $27.86(11.83)$ |
| 集合 3 - 所有意识到有一人自私 | $10.05(0.88)$ | $151.93(10.56)$ |

## 参考文献

+   Agapiou 等人 (2023) Agapiou, J.P., Vezhnevets, A.S., Duéñez-Guzmán, E.A., Matyas, J., Mao, Y., Sunehag, P., Köster, R., Madhushani, U., Kopparapu, K., Comanescu, R., Strouse, D., Johanson, M.B., Singh, S., Haas, J., Mordatch, I., Mobbs, D., Leibo, J.Z., 2023. Melting pot 2.0. [arXiv:2211.13746](http://arxiv.org/abs/2211.13746).

+   Axelrod 和 Hamilton (1981) Axelrod, R., Hamilton, W.D., 1981. 合作演化. 科学 211, 1390–1396. 网址：[https://www.science.org/doi/abs/10.1126/science.7466396](https://www.science.org/doi/abs/10.1126/science.7466396)，doi：[10.1126/science.7466396](http://dx.doi.org/10.1126/science.7466396).

+   Broekhuizen 等人 (2023) Broekhuizen, T., Dekker, H., de Faria, P., Firk, S., Nguyen, D.K., Sofka, W., 2023. 用于管理开放创新的人工智能：机遇、挑战及研究议程. 商业研究期刊 167, 114196. 网址：[https://www.sciencedirect.com/science/article/pii/S0148296323005556](https://www.sciencedirect.com/science/article/pii/S0148296323005556)，doi：[https://doi.org/10.1016/j.jbusres.2023.114196](http://dx.doi.org/https://doi.org/10.1016/j.jbusres.2023.114196).

+   Dafoe 等人 (2020) Dafoe, A., Hughes, E., Bachrach, Y., Collins, T., McKee, K.R., Leibo, J.Z., Larson, K., Graepel, T., 2020. 合作人工智能中的开放问题. [arXiv:2012.08630](http://arxiv.org/abs/2012.08630).

+   Dale 等人 (2020) Dale, R., Marshall-Pescini, S., Range, F., 2020. 什么对合作至关重要？社会关系对认知的重要性. 科学报告 10, 11778. 网址：[https://doi.org/10.1038/s41598-020-68734-4](https://doi.org/10.1038/s41598-020-68734-4)，doi：[10.1038/s41598-020-68734-4](http://dx.doi.org/10.1038/s41598-020-68734-4).

+   Du 等人 (2023) Du, Y., Li, S., Torralba, A., Tenenbaum, J.B., Mordatch, I., 2023. 通过多智能体辩论提升语言模型的事实性和推理能力. [arXiv:2305.14325](http://arxiv.org/abs/2305.14325).

+   Gross 等人（2023）Gross, J., Méder, Z.Z., Dreu, C.K.D., Romano, A., Molenmaker, W.E., Hoenig, L.C., 2023. 普遍合作的演化。科学进展 9, eadd8289。网址：[https://www.science.org/doi/abs/10.1126/sciadv.add8289](https://www.science.org/doi/abs/10.1126/sciadv.add8289)，doi：[10.1126/sciadv.add8289](http://dx.doi.org/10.1126/sciadv.add8289)。

+   Hong 等人（2023）Hong, S., Zhuge, M., Chen, J., Zheng, X., Cheng, Y., Zhang, C., Wang, J., Wang, Z., Yau, S.K.S., Lin, Z., Zhou, L., Ran, C., Xiao, L., Wu, C., Schmidhuber, J., 2023. Metagpt：用于多代理协作框架的元编程。[arXiv:2308.00352](http://arxiv.org/abs/2308.00352)。

+   Leibo 等人（2021）Leibo, J.Z., Duéñez-Guzmán, E., Vezhnevets, A.S., Agapiou, J.P., Sunehag, P., Koster, R., Matyas, J., Beattie, C., Mordatch, I., Graepel, T., 2021. 使用 Melting Pot 进行多代理强化学习的可扩展评估。[arXiv:2107.06857](http://arxiv.org/abs/2107.06857)。

+   Leibo 等人（2017）Leibo, J.Z., Zambaldi, V., Lanctot, M., Marecki, J., Graepel, T., 2017. 在顺序社会困境中的多代理强化学习，收录于：第16届自主代理与多代理系统会议论文集，国际自主代理与多代理系统基金会，南卡罗来纳州理奇兰。第464-473页。

+   Liu 等人（2023）Liu, Z., Yao, W., Zhang, J., Xue, L., Heinecke, S., Murthy, R., Feng, Y., Chen, Z., Niebles, J.C., Arpit, D., Xu, R., Mui, P., Wang, H., Xiong, C., Savarese, S., 2023. Bolaa: 基准测试与协调 LLM 增强的自主代理。[arXiv:2308.05960](http://arxiv.org/abs/2308.05960)。

+   McKee 等人（2023）McKee, K.R., Hughes, E., Zhu, T.O., Chadwick, M.J., Koster, R., Castaneda, A.G., Beattie, C., Graepel, T., Botvinick, M., Leibo, J.Z., 2023. 一个多代理强化学习模型：人类群体中的声誉与合作。[arXiv:2103.04982](http://arxiv.org/abs/2103.04982)。

+   Ni 和 Buehler（2024）Ni, B., Buehler, M.J., 2024. Mechagents: 大型语言模型多代理协作能够解决力学问题、生成新数据并整合知识。极限力学快报 67, 102131。网址：[https://www.sciencedirect.com/science/article/pii/S2352431624000117](https://www.sciencedirect.com/science/article/pii/S2352431624000117)，doi：[https://doi.org/10.1016/j.eml.2024.102131](http://dx.doi.org/https://doi.org/10.1016/j.eml.2024.102131)。

+   Park 等人（2023）Park, J.S., O’Brien, J.C., Cai, C.J., Morris, M.R., Liang, P., Bernstein, M.S., 2023. 生成代理：人类行为的互动模拟。[arXiv:2304.03442](http://arxiv.org/abs/2304.03442)。

+   Pennisi（2009）Pennisi, E., 2009. 合作的起源。科学 325, 1196–1199。网址：[https://www.science.org/doi/abs/10.1126/science.325.1196](https://www.science.org/doi/abs/10.1126/science.325.1196)，doi：[10.1126/science.325.1196](http://dx.doi.org/10.1126/science.325.1196)。

+   Rios 等人（2023）Rios, M., Quijano, N., Giraldo, L.F., 2023. 利用多智能体强化学习理解世界以解决社会困境。 [arXiv:2305.11358](http://arxiv.org/abs/2305.11358)。

+   Schick 等人（2023）Schick, T., Dwivedi-Yu, J., Dessì, R., Raileanu, R., Lomeli, M., Zettlemoyer, L., Cancedda, N., Scialom, T., 2023. Toolformer: 语言模型可以自学使用工具。 [arXiv:2302.04761](http://arxiv.org/abs/2302.04761)。

+   Shinn 等人（2023）Shinn, N., Cassano, F., Berman, E., Gopinath, A., Narasimhan, K., Yao, S., 2023. Reflexion: 具有语言强化学习的语言代理。 [arXiv:2303.11366](http://arxiv.org/abs/2303.11366)。

+   Wang 等人（2023）Wang, G., Xie, Y., Jiang, Y., Mandlekar, A., Xiao, C., Zhu, Y., Fan, L., Anandkumar, A., 2023. Voyager: 具有大型语言模型的开放式具身代理。 [arXiv:2305.16291](http://arxiv.org/abs/2305.16291)。

+   Yao 等人（2023）Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., Cao, Y., 2023. React: 协同推理与行动在语言模型中的作用。 [arXiv:2210.03629](http://arxiv.org/abs/2210.03629)。

+   Zhang 等人（2023）Zhang, J., Xu, X., Deng, S., 2023. 探索大型语言模型代理的协作机制：社会心理学视角。 [arXiv:2310.02124](http://arxiv.org/abs/2310.02124)。

+   Zhou 等人（2024）Zhou, P., Pujara, J., Ren, X., Chen, X., Cheng, H.T., Le, Q.V., Chi, E.H., Zhou, D., Mishra, S., Zheng, H.S., 2024. Self-discover: 大型语言模型自我构建推理结构。 [arXiv:2402.03620](http://arxiv.org/abs/2402.03620)。

+   Zhu 等人（2023）Zhu, Z., Xue, Y., Chen, X., Zhou, D., Tang, J., Schuurmans, D., Dai, H., 2023. 大型语言模型可以学习规则。 [arXiv:2310.07064](http://arxiv.org/abs/2310.07064)。
