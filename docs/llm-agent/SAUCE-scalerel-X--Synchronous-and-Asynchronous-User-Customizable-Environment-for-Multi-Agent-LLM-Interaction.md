<!--yml

分类: 未分类

日期: 2025-01-11 11:58:57

-->

# SAUCE\scalerel*X：用于多智能体LLM交互的同步与异步用户定制环境

> 来源：[https://arxiv.org/html/2411.03397/](https://arxiv.org/html/2411.03397/)

Shlomo Neuberger¹, Niv Eckhaus², Uri Berger^(2,3),

Amir Taubenfeld^(2,5), Gabriel Stanovsky², Ariel Goldstein^(1,4)

¹耶路撒冷希伯来大学商学院，以色列

²耶路撒冷希伯来大学计算机科学与工程学院

³墨尔本大学计算与信息系统学院

⁴耶路撒冷希伯来大学认知与脑科学系

⁵谷歌研究

通信地址：[shlomo.neuberger@gmail.com](mailto:shlomo.neuberger@gmail.com)

###### 摘要

许多人类互动，例如政治辩论，通常是在群体环境中进行的，在这种环境下有许多参与者，每个参与者都有不同的观点和议程。为了探索这种复杂的社会环境，我们提出了SAUCE\scalerel*![[未标注的图片]](img/004278962984f7af810d2f1319f98b73.png)X：一个可定制的Python平台，允许研究人员即插即用多种大型语言模型（LLM），让它们参与用户选择的任何主题的讨论。我们的平台负责实例化模型、安排响应、管理讨论历史，并生成全面的输出日志，所有这些都可以通过配置文件进行定制，几乎不需要任何编码技能。SAUCE\scalerel*![[未标注的图片]](img/004278962984f7af810d2f1319f98b73.png)X的一个创新特性是我们的异步通信功能，模型不仅可以决定说什么，还能决定何时发言，从而模拟人类沟通的一个重要方面。我们通过两个初步实验展示了SAUCE\scalerel*![[未标注的图片]](img/004278962984f7af810d2f1319f98b73.png)X的吸引力，并邀请社区使用它进行各种小组模拟。¹¹1 SAUCE\scalerel*![[未标注的图片]](img/004278962984f7af810d2f1319f98b73.png)X已在[github.com/Deep-Cognition-Lab/SAUCE](https://github.com/Deep-Cognition-Lab/SAUCE)上公开发布。短视频演示可在[此YouTube链接](https://youtu.be/Y4VIY-Qfneg?si=Hs6BpXK_f2Ox_P09)观看。

SAUCE\scalerel*![[未标注的图片]](img/004278962984f7af810d2f1319f98b73.png)X：用于多智能体LLM交互的同步与异步用户定制环境

Shlomo Neuberger¹, Niv Eckhaus², Uri Berger^(2,3), Amir Taubenfeld^(2,5), Gabriel Stanovsky², Ariel Goldstein^(1,4) ¹耶路撒冷希伯来大学商学院，以色列 ²耶路撒冷希伯来大学计算机科学与工程学院 ³墨尔本大学计算与信息系统学院 ⁴耶路撒冷希伯来大学认知与脑科学系 ⁵谷歌研究 通信地址：[shlomo.neuberger@gmail.com](mailto:shlomo.neuberger@gmail.com)

## 1 引言

近年来，大型语言模型（LLMs）随着聊天能力的提升而崛起，Köpf 等人（[2024](https://arxiv.org/html/2411.03397v1#bib.bib6)）。这些模型通常在两个基本假设下进行训练和评估。首先，与 LLM 的互动通常是假设为*二元*互动。也就是说，通常有一个人类用户发出自然语言指令，然后单个 LLM 尝试执行这些指令。此外，互动是*同步的*，即 LLM 用自己的单一回应来回答人类用户的每个请求，之后用户可以基于模型的回应发出单一的后续请求，依此类推，其中没有外部时间流逝的概念。这种框架足够表达，能够处理各种领域的各种任务。

![参见标题](img/a3952790442f58fbad2cbdc47aa5c3a5.png)

图 1：不同代理之间讨论的示意图，运行在 SAUCE\scalerel*![参见标题](img/004278962984f7af810d2f1319f98b73.png)X 上。我们的框架允许设置讨论主题，然后通过实例化模型并调度它们的回应来管理小组讨论。

然而，许多现实世界的人类互动并不遵循这些共同的基本假设，因此无法在标准的 LLM 应用中捕捉到。首先，人类互动往往是在任意数量的参与者之间进行的，每个参与者可能有不同的观点，并且对于互动结果有不同的目标。这种多方讨论的目标通常是通过参与者之间的达成一致或妥协来找到共同点。例如，在政治辩论中，理想情况下就是这种情况。其次，在许多现实场景中，人类互动是异步的，除了决定*说什么*，还需要决定*何时*发言，这常常是一个显著的挑战。例如，在许多战略谈判场景中，如金融讨论或更有结构的社交游戏中，选择保持沉默往往能够传达重要的信息。

在这项工作中，我们介绍了SAUCE\scalerel*![[未标注图像]](img/004278962984f7af810d2f1319f98b73.png)X，一个模块化且用户友好的Python平台，用于多代理、异步LLM实验。SAUCE\scalerel*![[未标注图像]](img/004278962984f7af810d2f1319f98b73.png)X建立了一个讨论室，用户可以在其中实例化不同的模型，让它们围绕共享讨论主题相互互动（参见图 [1](https://arxiv.org/html/2411.03397v1#S1.F1 "图1 ‣ 1 引言 ‣ SAUCE\scalerel*X：多代理LLM交互的同步与异步用户自定义环境") 和 [4](https://arxiv.org/html/2411.03397v1#S3.F4 "图4 ‣ 3.5.1 EndTypeNumMsgs ‣ 3.5 结束类型 ‣ 3 SAUCE\scalerel*X：架构 ‣ SAUCE\scalerel*X：多代理LLM交互的同步与异步用户自定义环境")），提供同步调度，在这种调度下LLMs按预定义方式被提示，以及异步调度，在这种调度下SAUCE\scalerel*![[未标注图像]](img/004278962984f7af810d2f1319f98b73.png)X会跟踪一个模拟的外部时钟，允许模型根据外部时间和讨论历史“跳过”自己的回合。

我们展示了实验结果，表明SAUCE\scalerel*![[未标注图像]](img/004278962984f7af810d2f1319f98b73.png)X有效地促进了在同步和异步环境中对多代理LLM交互的研究。Taubenfeld等人（[2024](https://arxiv.org/html/2411.03397v1#bib.bib10)）最近使用我们的平台模拟了代表不同政治意识形态的代理之间的政治辩论，揭示了LLM代理倾向于服从模型固有的社会偏见，即使在被指示从特定政治视角进行辩论时。这种行为显著地与人类中观察到的社会动态相背离。在另一个实验中，我们模拟了对电车难题（Thomson [1984](https://arxiv.org/html/2411.03397v1#bib.bib11)）的异步哲学辩论，展示了代理如何根据情境和时间限制调整其参与度。这个设置展示了异步沟通的灵活性，揭示了多样化的发言策略，如频繁发言、选择等待并倾听，以及根据不断变化的情境调整参与方式。

SAUCE\scalerel*![[未标注图像]](img/004278962984f7af810d2f1319f98b73.png)X可以促进两个互补方向的研究。首先，对多参与者或异步环境中实际场景感兴趣的模型开发者，可以轻松地插拔自己的模型，评估它们如何相互作用。其次，SAUCE\scalerel*![[未标注图像]](img/004278962984f7af810d2f1319f98b73.png)X使得用户研究能够将人类参与者与LLM（大语言模型）结合，在这些环境中进行互动。

## 2 SAUCE\scalerel*![[未标注图像]](img/004278962984f7af810d2f1319f98b73.png)X：亮点

我们的平台旨在满足一些特定的需求：

+   •

    与各种模型类型的集成。考虑到大语言模型（LLMs）的动态发展，SAUCE\scalerel*![[无标题图像]](img/004278962984f7af810d2f1319f98b73.png)X 便于与不同的 LLM 源进行集成，包括 HuggingFace、API 或本地设置。用户可以通过配置文件管理每个参与者的模型类型。

+   •

    异步通信。该框架支持异步通信，使得智能体可以根据上下文选择性地参与并跳过轮次。

+   •

    可复现性。通过使用相同的配置文件，实验可以始终如一地重复。系统还具有批处理模式，支持同一实验的多个迭代。详细的日志有助于结果分析。

## 3 SAUCE\scalerel*![[无标题图像]](img/004278962984f7af810d2f1319f98b73.png)X：架构

[⬇](data:text/plain;base64,ewogICJleHBlcmltZW50IjogewogICJzY2VuYXJpbyI6ICJZb3UncmUgZGlzY3Vzc2luZyBzb2NpYWwgd2VsZmFyZSIKICB9LAogICJob3N0IjogewogICAgImNsYXNzIjogIlJvdW5kIFJvYmluIEhvc3QiLAogICAgInN0YXJ0X3BlcnNvbl9pbmRleCI6IDAKICB9LAogICJwZXJzb25zIjogWwogICAgewogICAgICJjbGFzcyI6ICJwZXJzb25faHVnZ2luZ19mYWNlIiwKICAgICAibmFtZSI6ICJLYXR5YSIsCiAgICAgImJhY2tncm91bmRfc3RvcnkiOiAiWW91IGFyZSB2ZXJ5IGtpbmQgYW5kIHlvdSBhbHdheXMgdHJ5IHRvIGhlbHAgb3RoZXJzLiIsCiAgICAgIm1vZGVsX3BhdGgiOiAibWlzdHJhbGFpL01peHRyYWwtOHg3Qi12MC4xIgogICAgfSwKICAgIHsKICAgICAiY2xhc3MiOiAiaHVtYW4iLAogICAgICJuYW1lIjogIlZpY3RvciIsCiAgICAgImJhY2tncm91bmRfc3RvcnkiOiAiWW91IGRvbid0IGNhcmUgbXVjaCBhYm91dCBvdGhlciBwZW9wbGUncyBmZWVsaW5ncy4iCiAgICB9LAogICAgewogICAgICJjbGFzcyI6ICJhc3luY19ncm91cF9kaXNjdXNzYW50IiwKICAgICAibmFtZSI6ICJKdWxpZXQiLAogICAgICJiYWNrZ3JvdW5kX3N0b3J5IjogIllvdSdyZSBhbiB1bmRlY2lzaXZlIHBlcnNvbiIsCiAgICAgImdlbmVyYXRpb25fbW9kZWxfbmFtZSI6ICJtZXRhLWxsYW1hL01ldGEtTGxhbWEtMy4xLThCIgogICAgICJzY2hlZHVsaW5nX21vZGVsX25hbWUiOiAibWljcm9zb2Z0L1BoaS0zLW1pbmktNGstaW5zdHJ1Y3QiCiAgICB9CiAgXSwKICAiZW5kVHlwZSI6IHsKICAgImNsYXNzIjogIml0ZXJhdGlvbiIsCiAgICJtYXhfbnVtX21zZ3MiOiAyMAogIH0KfQ==){"experiment":  {"scenario":  "You’re  discussing  social  welfare"},"host":  {"class":  "Round  Robin  Host","start_person_index":  0},"persons":  [{"class":  "person_hugging_face","name":  "Katya","background_story":  "You  are  very  kind  and  you  always  try  to  help  others.","model_path":  "mistralai/Mixtral-8x7B-v0.1"},{"class":  "human","name":  "Victor","background_story":  "You  don’t  care  much  about  other  people’s  feelings."},{"class":  "async_group_discussant","name":  "Juliet","background_story":  "You’re  an  undecisive  person","generation_model_name":  "meta-llama/Meta-Llama-3.1-8B""scheduling_model_name":  "microsoft/Phi-3-mini-4k-instruct"}],"endType":  {"class":  "iteration","max_num_msgs":  20}}

图 2：示例 JSON 配置文件，设置所有所需对象以进行多智能体讨论。

![参见标题](img/cf58e1568cbd16f16a80e4565f54117c.png)

图 3：SAUCE\scalerel*X 中类的层次结构，包括带有详细描述的章节编号。所有继承自 AsynchronousPerson 的类在其对应章节中有描述。

SAUCE\scalerel*![[未标注的图片]](img/004278962984f7af810d2f1319f98b73.png)X 平台中的基础对象是实验（Experiment），它在图 [2](https://arxiv.org/html/2411.03397v1#S3.F2 "图 2 ‣ 3 SAUCE\scalerel*X: 架构 ‣ SAUCE\scalerel*X: 面向多代理 LLM 互动的同步和异步用户自定义环境") 所示的 JSON 文件中配置，指定了参与者列表（Person 对象）、管理讨论的主持人（Host）、结束实验的标准（EndType）以及其他可选字段，如参与者的实验后调查问题。

用户可以通过调用其 run 方法来执行实验。此方法启动实验的 SessionRoom，该会话室进入一个 while 循环，不断检查实验的 EndType 标准是否满足。只要未满足，该会话室将使用实验的 Host 确定下一个应该发言的 Person，并调用其 generate_answer 方法（如果异步 Person 选择不发言，则该方法可以返回 None）。一旦 EndType 标准满足，SessionRoom 将提示每个参与的 Person 回答预定的调查问题，最后返回完整的实验输出。

接下来的各节将详细介绍这些类的内容。请参见图 [3](https://arxiv.org/html/2411.03397v1#S3.F3 "图 3 ‣ 3 SAUCE\scalerel*X: 架构 ‣ SAUCE\scalerel*X: 面向多代理 LLM 互动的同步和异步用户自定义环境") 中的类层次结构。

### 3.1 实验

该类作为实验相关信息的容器。类的属性包括 Persons 对象的列表、SessionRoom、EndType、Host、场景描述（一个字符串，概述正在进行的具体实验，参见图 [2](https://arxiv.org/html/2411.03397v1#S3.F2 "图 2 ‣ 3 SAUCE\scalerel*X: 架构 ‣ SAUCE\scalerel*X: 面向多代理 LLM 互动的同步和异步用户自定义环境") 中的配置示例）和实验后的调查问题。其主要方法是 load_from_file，用于基于 JSON 配置初始化实例，以及 run，执行实验的方法。

### 3.2 会话室

SessionRoom代表实验发生的空间。该类的属性包括实验对象和更新中的当前聊天消息列表。在执行过程中，SessionRoom对象检查实验的EndType标准是否满足，并继续调用其`iterate`方法，直到满足为止。`iterate`方法使用实验的Host来确定哪个参与者应当发言，然后调用该参与者的`generate_answer`。如果参与者选择发言，聊天消息列表会相应更新。当EndType标准满足时，SessionRoom对象通过调查问题查询所有参与者。

### 3.3 主办方

这是一个抽象类，代表实验的指挥者，负责指导和管理参与者之间的互动。它确定实验的规则和条件，并管理互动的时序和顺序。

该类的属性包括参与者列表（Person对象）和当前轮到发言的参与者索引。它唯一的方法是`get_curr_person_and_move_to_next`，该方法返回下一个将发言的Person，并相应更新索引属性。

#### 3.3.1 HostRoundRobin

一个按照轮流方式循环遍历参与者的主办方。它可以接受一个参数，用于指定从哪个参与者开始。

#### 3.3.2 HostRandom

一个主办方，每次回合随机选择一个参与者，方法是从所有参与者中均匀且独立地进行抽样。

### 3.4 参与者

Person是一个抽象类，代表实验中的一个参与者。它封装了参与者的共同属性和行为，包括他们的名字、背景故事以及用于生成消息的模型。该类的接口仅包括`generate_answer`方法。此方法以实验的场景（例如，见图[2](https://arxiv.org/html/2411.03397v1#S3.F2 "图 2 ‣ 3 SAUCE\scalerel*X：架构 ‣ SAUCE\scalerel*X：同步和异步可定制的多代理LLM互动环境")）和当前的聊天历史为输入，并输出一个相应的消息，供对话添加。

#### 3.4.1 实现的人员类

我们在SAUCE\scalerel*![[无标题图片]](img/004278962984f7af810d2f1319f98b73.png)X中实现了几个现成的人员类。这些类使得与现有的LLM API进行交互成为可能，例如PersonOpenAiCompletion和PersonHuggingFace，它们分别被定制用于使用OpenAI的Completion平台和HuggingFace的transformers包生成消息。

我们还实现了Human类，它允许在实验中配置一个人类参与者。使用这个子类时，系统会交互式地提示用户输入，并将该输入作为生成的消息。

#### 3.4.2 异步通信

我们使用不同的Person对象来模拟异步通信。这种方法使我们能够独立于所选择的Host，并支持不同类型的Person，具有不同形式的同步和异步方法，如图[2](https://arxiv.org/html/2411.03397v1#S3.F2 "图 2 ‣ 3 SAUCE\scalerel*X: 架构 ‣ SAUCE\scalerel*X: 同步和异步的用户自定义多代理LLM互动环境")中的配置示例所示，其中两个人使用同步通信，另一个使用异步通信。为了实现这一点，当Host授予机会时，Person选择是否发言或跳过轮次。

为了模拟异步通信，抽象的AsynchronousPerson类使得`generate_answer`方法能够返回None作为潜在输出，表示参与者选择不发言。

该类还包括布尔方法`should_generate_answer`，该方法必须在子类中实现。当将此功能与Host结合时，结果是一个消息采样过程，Host不断地询问所有参与者，允许他们决定何时发言。因此，通过频繁采样，我们可以模拟连续的互动。

该平台允许与参与者共享实验的开始时间（作为人物配置中的一个参数），并在提示和生成消息时使用它和当前时间。这使得在考虑时间的情况下进行操作，特别是当有时间限制时，参与者可以基于此来决定是否在某一时刻发言。[4.2](https://arxiv.org/html/2411.03397v1#S4.SS2 "4.2 哲学小组讨论中的异步设置 ‣ 4 评估：案例研究 ‣ SAUCE\scalerel*X: 同步和异步的用户自定义多代理LLM互动环境")部分演示了考虑时间可能会影响发言策略。

##### 已实现异步参与者。

与同步情况下一样，我们实现了一个由人控制的类——AsynchronousHuman。它继承了常规的同步Human类，并实现了AsynchronousPerson接口。它允许将人类参与者包括在由Host运行的实验中，从而使其能够与人工代理互动，同时保持在Host授予发言权时自由选择是否发言。`should_generate_answer`方法通过询问用户是否发送消息来实现。

此外，我们为LLM生成的异步通信提供了两个抽象子类。FineTunedAsynchronousPerson代表一个抽象类，用于表示经过微调的LLM模型，它执行两项任务：(1) 决定是否发言，(2) 决定说什么。这类模型可以被微调为输出一个新的特殊标记（例如<pass>），以表示它们选择跳过轮次。

InnerSchedulerAsynchronousPerson持有两个独立的模型实例——一个用于消息生成（生成器——可以通过生成模型名称字段进行配置，如图[2](https://arxiv.org/html/2411.03397v1#S3.F2 "Figure 2 ‣ 3 SAUCE\scalerel*X: Architecture ‣ SAUCE\scalerel*X: Synchronous and Asynchronous User-Customizable Environment for Multi-Agent LLM Interaction")所示），另一个用于决定是否发布消息（调度器——可以通过调度模型名称字段进行配置，如图[2](https://arxiv.org/html/2411.03397v1#S3.F2 "Figure 2 ‣ 3 SAUCE\scalerel*X: Architecture ‣ SAUCE\scalerel*X: Synchronous and Asynchronous User-Customizable Environment for Multi-Agent LLM Interaction")所示）。为了模拟不同的决策过程，我们实现了两个子类。第一个（FirstDecidesThenGenerates）模拟基于上下文的决策过程，其中调度器首先根据聊天历史判断是否生成消息，如果批准，则生成器创建该消息。第二个（FirstGeneratesThenDecides）在决策是否发言时，会考虑生成器在聊天历史中的预测输出。

最后，我们实现了一个任务特定的类——AsynchronousGroupDiscussant，用于模拟小组讨论。该类继承自InnerSchedulerAsynchronousPerson，并添加了个人观点等属性。该类使得参与者能够就预设的话题进行讨论。有关我们使用这种类型的参与者模拟哲学讨论的详细示例实验，请参见[4.2](https://arxiv.org/html/2411.03397v1#S4.SS2 "4.2 Philosophical Group Discussion in An Asynchronous Setting ‣ 4 Evaluation: Case Studies ‣ SAUCE\scalerel*X: Synchronous and Asynchronous User-Customizable Environment for Multi-Agent LLM Interaction")节。

### 3.5 结束类型（EndType）

结束类型（EndType）是决定实验结束条件的抽象表示。它可以通过整体标准来决定，例如参与者之间的整体一致性，或者通过对话的技术特征来决定，例如达到消息数量限制。它用于标志实验的结束，并启动任何需要的操作或数据收集。

该类包含一个布尔方法——did_end，该方法以SessionRoom作为参数。通过使用SessionRoom的信息，如当前的聊天历史，该方法决定实验是否应结束。

#### 3.5.1 结束类型——消息数量（EndTypeNumMsgs）

基于小组对话中发送的消息数量的结束类型标准。当SessionRoom的聊天历史达到预设的消息数量时，触发实验的结束。

![参见说明文字](img/cbfc7f2931682c29b533241a4302981a.png)

图4：展示了作为SAUCE\scalerel*![参见说明](img/004278962984f7af810d2f1319f98b73.png)X上的异步小组讨论一部分发送的几条消息。本实验在确认当前时间并设置讨论时间限制的情况下进行。粗体：两位参与者（Joseph和Jennifer）选择不发言，这促使第三位参与者（Amanda）请求他们发表意见（粗体）；第四位参与者（Robert）注意到其他人保持沉默后，再次选择发言。

## 4 评估：案例研究

我们描述了使用SAUCE\scalerel*![[无标题图片]](img/004278962984f7af810d2f1319f98b73.png)X进行的两个初步实验，旨在展示其在解决各种研究问题中的实用性。

### 4.1 同步设置中的政治辩论模拟

使用我们的框架，Taubenfeld等人（[2024](https://arxiv.org/html/2411.03397v1#bib.bib10)）探讨了LLM代理的态度变化和偏见。他们为共和党和民主党代理设计了叙事，并在关于有争议问题的跨党派辩论中监控他们的观点。图[1](https://arxiv.org/html/2411.03397v1#S1.F1 "图1 ‣ 1 引言 ‣ SAUCE\scalerel*X: 用于多代理LLM交互的同步和异步用户可定制环境")呈现了与该研究中的对话相似的情形。

为了追踪参与者的态度，我们通过一项退出调查向模型提问，这是我们框架的一个功能。在每次辩论之前和每个循环之后，代理被要求使用0-10的量表评价他们对话题严重性的立场。为了防止调查影响辩论或未来的评分，框架被配置为将调查问题从对话历史中排除，确保代理无法得知自己或他人先前的回答。

该研究使用了多种模型，包括HuggingFace的本地模型和通过OpenAI API访问的其他模型。为了获得具有统计学意义的结果，每个实验都进行了多次重复，SAUCE\scalerel*![[无标题图片]](img/004278962984f7af810d2f1319f98b73.png)X简化了这一过程，支持在本地GPU上并行执行或通过批量请求访问OpenAI端点。

### 4.2 异步设置中的哲学小组讨论

我们使用SAUCE\scalerel*![[无标题图片]](img/004278962984f7af810d2f1319f98b73.png)X通过哲学小组讨论来研究多代理环境中的伦理决策动态。我们进行了一场经典电车难题的辩论实验，目标是在严格的10分钟时间限制内达成一致决定。该情境要求模型决定是否拉动一个杠杆，将一辆失控的电车引到另一条轨道，那里会杀死一个人而不是五个。在这个实验中，我们使用了我们框架的异步通信功能。

我们使用了四个参与者，每个参与者都有一个独特的背景故事（见图[4](https://arxiv.org/html/2411.03397v1#S3.F4 "图 4 ‣ 3.5.1 结束类型与消息数 ‣ 3.5 结束类型 ‣ 3 SAUCE\scalerel*X: 架构 ‣ SAUCE\scalerel*X: 用于多智能体LLM互动的同步与异步用户可定制环境")），他们对话题的初步看法以及他们意见的强度。参与者通过AsynchronousGroupDiscussant类进行实现。

我们尝试了多种提示方法来决定何时发言。我们发现，在提示中指示剩余讨论时间会导致智能体选择发言的频率有所变化（见图[4](https://arxiv.org/html/2411.03397v1#S3.F4 "图 4 ‣ 3.5.1 结束类型与消息数 ‣ 3.5 结束类型 ‣ 3 SAUCE\scalerel*X: 架构 ‣ SAUCE\scalerel*X: 用于多智能体LLM互动的同步与异步用户可定制环境")）。两位性格较为内敛的角色（Joseph和Jennifer）在大多数回合中选择不发言。相比之下，具有明确观点的角色（Amanda）则积极鼓励那些较为安静的参与者加入讨论。第四个角色（Robert）被设计为中立者，在意见明确的角色尝试吸引安静的参与者时，他最初选择等待。经过几次未果的尝试后，他最终变得更加积极地参与讨论。

## 5 相关工作

最近的几项研究提出了多智能体互动的框架。有些研究聚焦于任务导向的对话 Li et al. ([2023](https://arxiv.org/html/2411.03397v1#bib.bib7)); Chen et al. ([2023](https://arxiv.org/html/2411.03397v1#bib.bib2))，防止用户选择自己的讨论话题。其他研究则探索了限制为两个智能体进行对话的互动 Park et al. ([2023](https://arxiv.org/html/2411.03397v1#bib.bib9)); Wang et al. ([2023](https://arxiv.org/html/2411.03397v1#bib.bib12))，并且仅允许同步通信 Gu et al. ([2024](https://arxiv.org/html/2411.03397v1#bib.bib3))。相比之下，SAUCE\scalerel*![[未标注的图片]](img/004278962984f7af810d2f1319f98b73.png)X支持不限数量的智能体就任何话题进行讨论，且支持同步和异步通信。

## 6 结论与未来工作

SAUCE\scalerel*![[未标注的图片]](img/004278962984f7af810d2f1319f98b73.png)X使得使用人工多智能体模拟来理解人类社会互动和复杂社会行为成为可能。这种方法对于研究复杂现象，特别是在经济学等各种社会领域中的合作、竞争和群体沟通等问题尤为重要，最近在这些领域中，大型语言模型（LLMs）被发现具有实用性 Horton ([2023](https://arxiv.org/html/2411.03397v1#bib.bib4)); Argyle et al. ([2023](https://arxiv.org/html/2411.03397v1#bib.bib1)); Kazinnik ([2023](https://arxiv.org/html/2411.03397v1#bib.bib5)); Manning et al. ([2024](https://arxiv.org/html/2411.03397v1#bib.bib8))。

此外，该平台还允许在异步环境中建模 LLM 之间的互动，例如群体互动和社交游戏。目前，我们正在利用异步建模来模拟社交 Mafia 游戏，在该游戏中，代理需要合作以发现谁在撒谎并应被群体淘汰。这种互动要求参与者特别关注说话时机，因为说得太多或太少可能会显得可疑。

## 参考文献

+   Argyle 等人（2023）Lisa P. Argyle, Ethan C. Busby, Nancy Fulda, Joshua R. Gubler, Christopher Rytting 和 David Wingate。2023。由一变多：使用语言模型模拟人类样本。*政治分析*，31(3):337–351。

+   Chen 等人（2023）Weize Chen, Yusheng Su, Jingwei Zuo, Cheng Yang, Chenfei Yuan, Chen Qian, Chi-Min Chan, Yujia Qin, Yaxi Lu, Ruobing Xie 等人。2023。Agentverse：促进多智能体协作并探索智能体中的涌现行为。*arXiv 预印本 arXiv:2308.10848*。

+   Gu 等人（2024）Zhouhong Gu, Xiaoxuan Zhu, Haoran Guo, Lin Zhang, Yin Cai, Hao Shen, Jiangjie Chen, Zheyu Ye, Yifei Dai, Yan Gao 等人。2024。Agent group chat：一种互动式群聊模拟体，用于更好地引发集体涌现行为。*arXiv 预印本 arXiv:2403.13433*。

+   Horton（2023）John J. Horton。2023。大型语言模型作为模拟经济代理：我们能从 Homo Silicus 中学到什么？技术报告，美国国家经济研究局。

+   Kazinnik（2023）Sophia Kazinnik。2023。[银行挤兑中断：利用生成式 AI 模拟存款撤回](https://api.semanticscholar.org/CorpusID:266454509)。*SSRN 电子期刊*。

+   Köpf 等人（2024）Andreas Köpf, Yannic Kilcher, Dimitri von Rütte, Sotiris Anagnostidis, Zhi Rui Tam, Keith Stevens, Abdullah Barhoum, Duc Nguyen, Oliver Stanley, Richárd Nagyfi 等人。2024。Openassistant conversations——民主化大规模语言模型的对齐。*神经信息处理系统进展*，36。

+   Li 等人（2023）Guohao Li, Hasan Hammoud, Hani Itani, Dmitrii Khizbullin 和 Bernard Ghanem。2023。Camel：用于“大型语言模型社会”思想探索的交互式代理。*神经信息处理系统进展*，36:51991–52008。

+   Manning 等人（2024）Benjamin S. Manning, Kehang Zhu 和 John J. Horton. 2024. [自动化社会科学：语言模型作为科学家和受试者](https://arxiv.org/abs/2404.11794)。*预印本*，arXiv:2404.11794。

+   Park 等人（2023）Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang 和 Michael S. Bernstein。2023。生成代理：人类行为的互动式模拟体。在 *第36届年度 ACM 用户界面软件与技术研讨会论文集*，第1-22页。

+   Taubenfeld 等人（2024）Amir Taubenfeld, Yaniv Dover, Roi Reichart 和 Ariel Goldstein。2024。LLM 辩论模拟中的系统性偏差。*arXiv 预印本 arXiv:2402.04049*。

+   托马森（1984）朱迪思·贾维斯·托马森。1984年。电车难题。*耶鲁法学评论*，94:1395。

+   王等人（2023）赵琳·王、余英·邱和余昶·邱。2023年。[类人代理：用于模拟类人生成代理的平台](https://doi.org/10.18653/v1/2023.emnlp-demo.15)。收录于《2023年自然语言处理实证方法会议：系统演示论文集》，第167-176页，新加坡。计算语言学会。
