<!--yml

类别：未分类

日期：2025-01-11 12:50:41

-->

# 大型语言模型作为城市居民：一种用于个人出行生成的LLM代理框架

> 来源：[https://arxiv.org/html/2402.14744/](https://arxiv.org/html/2402.14744/)

\mdfsetup

font=, innertopmargin=2pt, innerbottommargin=2pt, innerleftmargin=8pt, innerrightmargin=8pt, frametitleaboveskip=2pt, frametitlebelowskip=1pt, frametitlealignment=, middlelinecolor=blue

王佳伟¹ &姜仁和¹ &杨创¹ &吴泽青²

和大贯诚² &柴崎良介¹ &越智信男¹ &肖川²

和 ¹东京大学，²大阪大学

{jiawei@g.ecc, koshizuka@iii}.u-tokyo.ac.jp

{jiangrh,chuang.yang,shiba}@csis.u-tokyo.ac.jp

wuzengqing@outlook.com, {onizuka,chuanx}@ist.osaka-u.ac.jp 通讯作者。

###### 摘要

本文介绍了一种新颖的方法，使用集成到代理框架中的大型语言模型（LLM）来灵活有效地生成个人出行方式。LLM通过有效处理语义数据并提供多样化的建模任务，克服了以往模型的局限性。我们的方法解决了三个研究问题：将LLM与现实世界的城市出行数据对齐，开发可靠的活动生成策略，以及探索LLM在城市出行中的应用。关键的技术贡献是提出了一种新型的LLM代理框架，该框架考虑了个人活动模式和动机，包括一种自一致性方法来将LLM与现实世界的活动数据对齐，以及一种基于检索增强的策略用于可解释的活动生成。我们评估了LLM代理框架，并将其与最先进的个人出行生成方法进行了比较，展示了我们方法的有效性及其在城市出行中的潜在应用。总的来说，这项研究标志着基于现实世界人类活动数据设计LLM代理框架用于活动生成的开创性工作，为城市出行分析提供了一个有前景的工具。

源代码可在[https://github.com/Wangjw6/LLMob/](https://github.com/Wangjw6/LLMob/) 获取。

## 1 引言

大型语言模型（LLM）的普及促进了许多超越自然语言处理（NLP）领域的应用。值得注意的是，LLM在许多学科中广泛应用，推动了我们对人类和社会的理解，涵盖了经济学[[1](https://arxiv.org/html/2402.14744v3#bib.bib1)]、政治学[[2](https://arxiv.org/html/2402.14744v3#bib.bib2)]等多个领域，并且已被作为代理应用于多项社会科学研究[[36](https://arxiv.org/html/2402.14744v3#bib.bib36)，[35](https://arxiv.org/html/2402.14744v3#bib.bib35)，[10](https://arxiv.org/html/2402.14744v3#bib.bib10)]。本文的目标是探讨使用LLM代理来研究个人出行数据。对个人出行的建模为构建可持续社区提供了许多机会，包括主动的交通管理和全面城市发展战略的设计[[4](https://arxiv.org/html/2402.14744v3#bib.bib4)，[3](https://arxiv.org/html/2402.14744v3#bib.bib3)，[45](https://arxiv.org/html/2402.14744v3#bib.bib45)]。特别是，生成可靠的活动轨迹已成为利用个体活动数据的一种有前景和有效的方法[[13](https://arxiv.org/html/2402.14744v3#bib.bib13)，[6](https://arxiv.org/html/2402.14744v3#bib.bib6)]。一方面，学习生成活动轨迹有助于深入理解活动模式，从而灵活地模拟城市出行。另一方面，尽管由于电信技术的进步，个体活动轨迹数据非常丰富，但由于隐私问题，其实际应用往往受到限制。从这个意义上说，生成的数据可以提供一种可行的替代方案，在实用性和隐私之间找到平衡。

![请参阅说明文字](img/5d1605cc13fbeca6b92f23d44d9096e2.png)

图1：使用LLM代理生成个人出行数据。

尽管先进的基于数据驱动的学习方法为生成合成个体轨迹提供了各种解决方案[[13](https://arxiv.org/html/2402.14744v3#bib.bib13), [39](https://arxiv.org/html/2402.14744v3#bib.bib39), [9](https://arxiv.org/html/2402.14744v3#bib.bib9), [6](https://arxiv.org/html/2402.14744v3#bib.bib6), [16](https://arxiv.org/html/2402.14744v3#bib.bib16), [38](https://arxiv.org/html/2402.14744v3#bib.bib38)]，生成的数据只是从数据分布的角度模仿现实世界的数据，而非语义上的模仿，这使得它们在模拟或解释分布显著不同的新颖或未预见场景中的活动时（例如疫情期间）效果较差。因此，在本研究中，为了探索更智能和有效的活动生成方法，我们提出通过利用LLM代理的前沿智能建立轨迹生成框架，如图[1](https://arxiv.org/html/2402.14744v3#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")所示。LLMs在应用于活动轨迹生成时，相比于以往的模型，呈现出两个显著的优势：

+   •

    语义可解释性。与以前主要依赖结构化数据（例如基于GPS坐标的轨迹数据）进行校准和仿真的模型不同[[17](https://arxiv.org/html/2402.14744v3#bib.bib17), [27](https://arxiv.org/html/2402.14744v3#bib.bib27), [47](https://arxiv.org/html/2402.14744v3#bib.bib47)]，LLMs在解释语义数据（例如活动轨迹数据）方面表现出优越的能力。这一优势显著拓宽了将多样数据源纳入生成过程的范围，从而增强了模型以更加细致和有效的方式理解和与复杂、现实场景互动的能力。

+   •

    模型多样性。虽然其他基于数据驱动的方法能够学习此类动态活动模式以进行生成，但它们在未见场景下的生成能力有限。相反，LLMs在处理未见任务时表现出显著的多样性，特别是在基于现有信息进行推理和决策的能力[[26](https://arxiv.org/html/2402.14744v3#bib.bib26)]。这一能力使得LLMs能够提供多样且合理的选择，使其成为建模个人出行模式的一个有前景且灵活的方法。

尽管有这些好处，确保LLM与现实世界情境有效对齐仍然是一个重大挑战[[36](https://arxiv.org/html/2402.14744v3#bib.bib36)]。这种对齐在城市出行的背景下尤为关键，因为LLM输出的精确性和可靠性对基于这些输出的城市管理的有效性至关重要。在本研究中，我们的目标是通过探讨以下研究问题来解决这一挑战：RQ 1：如何有效地将LLM与关于日常个体活动的语义丰富数据对齐？RQ 2：使用LLM代理实现可靠和有意义的活动生成的有效策略是什么？RQ 3：LLM代理在提升城市出行分析中的潜在应用是什么？

为此，我们的研究使用LLM代理推断活动模式和动机，以进行个人活动生成任务。虽然之前的研究提到习惯性活动模式和动机是活动生成的两个关键因素[[17](https://arxiv.org/html/2402.14744v3#bib.bib17)，[43](https://arxiv.org/html/2402.14744v3#bib.bib43)]，我们提出的框架则引入了一种更具可解释性和有效性的解决方案。通过利用LLM在处理语义丰富的数据集（例如个人签到数据）方面的能力，我们能够进行细致且可解释的个人出行模拟。我们的方法论围绕两个阶段展开：（1）活动模式识别和（2）动机驱动的活动生成。在阶段1中，我们利用LLM代理的语义感知，从历史数据中提取和识别自洽的个性化习惯性活动模式。在阶段2中，我们开发了两种可解释的检索增强策略，利用阶段1中识别的模式。这些策略引导LLM代理推断潜在的日常动机，如兴趣变化或情境需求。最后，我们指示LLM代理根据获得的模式和动机，模拟城市居民的行为。通过这种方式，我们生成了他们在特定推理逻辑下的日常活动。

我们使用GPT-3.5 API，并结合东京的个人活动轨迹数据集来评估所提出的框架。结果展示了我们框架在将LLM代理与语义丰富的数据对齐，用于生成个人日常活动方面的能力。与基线方法的比较，例如基于注意力的方法 [[8](https://arxiv.org/html/2402.14744v3#bib.bib8), [22](https://arxiv.org/html/2402.14744v3#bib.bib22)]，对抗性学习方法 [[6](https://arxiv.org/html/2402.14744v3#bib.bib6), [42](https://arxiv.org/html/2402.14744v3#bib.bib42)]，以及扩散模型 [[46](https://arxiv.org/html/2402.14744v3#bib.bib46)]，强调了我们框架的先进生成性能。观察结果还表明，我们的框架在再现个人出行生成的时间性和时空特性以及可解释的活动规律方面表现出色。此外，框架在特定情境下模拟城市出行（如疫情场景）时，显示了其适应外部因素并生成真实活动模式的潜力。

据我们所知，这项研究是*在基于现实世界数据生成活动轨迹的LLM代理框架开发领域的先驱性工作之一*。我们总结了我们的贡献如下： (1) 我们引入了一种新颖的LLM代理框架，用于个人出行生成，具有语义丰富性。 (2) 我们的框架引入了自一致性评估，以确保LLM代理的输出与日常活动的现实世界数据紧密对齐。 (3) 为了生成日常活动轨迹，我们的框架将活动模式与总结的动机相结合，并采用两种可解释的检索增强策略，旨在生成可靠的活动轨迹。 (4) 通过使用现实世界的个人活动数据，我们验证了框架的有效性，并探索其在城市出行分析中的应用潜力。

## 2 相关工作

### 2.1 个人出行生成

活动轨迹生成为理解个人移动性提供了有价值的视角。基于大量的通话详细记录，[姜等人](https://arxiv.org/html/2402.14744v3#bib.bib17)建立了一个机制建模框架，以生成具有高空间时间分辨率的个体活动。[Pappalardo和Simini](https://arxiv.org/html/2402.14744v3#bib.bib27)使用马尔可夫建模来估计个体访问特定位置的概率。此外，深度学习已成为建模交通复杂动态的强大工具[[15](https://arxiv.org/html/2402.14744v3#bib.bib15), [13](https://arxiv.org/html/2402.14744v3#bib.bib13), [44](https://arxiv.org/html/2402.14744v3#bib.bib44), [9](https://arxiv.org/html/2402.14744v3#bib.bib9), [21](https://arxiv.org/html/2402.14744v3#bib.bib21)]。主要挑战在于克服与数据相关的障碍，例如随机性、稀疏性和不规则模式[[8](https://arxiv.org/html/2402.14744v3#bib.bib8), [43](https://arxiv.org/html/2402.14744v3#bib.bib43), [42](https://arxiv.org/html/2402.14744v3#bib.bib42), [20](https://arxiv.org/html/2402.14744v3#bib.bib20)]。例如，[冯等人](https://arxiv.org/html/2402.14744v3#bib.bib8)提出了注意力递归网络来处理个人偏好和过渡规律。[袁等人](https://arxiv.org/html/2402.14744v3#bib.bib42)结合深度学习与神经微分方程，解决了活动轨迹生成中不规则采样活动的随机性和稀疏性挑战。最近，[朱等人](https://arxiv.org/html/2402.14744v3#bib.bib46)提出利用扩散模型生成GPS轨迹。

### 2.2 社会科学中的LLM代理

探索如何将LLM视为特定场景中的自主代理，带来了社会科学领域多样且富有前景的应用[[32](https://arxiv.org/html/2402.14744v3#bib.bib32), [36](https://arxiv.org/html/2402.14744v3#bib.bib36), [35](https://arxiv.org/html/2402.14744v3#bib.bib35), [10](https://arxiv.org/html/2402.14744v3#bib.bib10)]。例如，[Park et al.](https://arxiv.org/html/2402.14744v3#bib.bib28)建立了一个LLM代理框架，用于模拟人类在互动场景中的行为，展示了LLM在建模复杂社会互动和决策过程中的潜力。此外，LLM代理在经济研究中的应用也已被探索，为金融市场和经济学提供了新的见解[[11](https://arxiv.org/html/2402.14744v3#bib.bib11), [19](https://arxiv.org/html/2402.14744v3#bib.bib19)]。不仅仅局限于社会科学领域，[Mao et al.](https://arxiv.org/html/2402.14744v3#bib.bib23)巧妙地利用LLM生成了驾驶轨迹，用于运动规划任务。在自然科学领域，[Williams et al.](https://arxiv.org/html/2402.14744v3#bib.bib34)将LLM与流行病模型结合，用于模拟疾病传播。这些多样化的应用突显了LLM在理解和建模各种现实世界动态方面的多功能性和潜力。

## 3 方法论

我们考虑生成单个日常活动轨迹，每条轨迹代表一个人整天的活动。此外，我们关注城市背景，在这种背景下，每个人的活动轨迹被表示为一个按时间顺序排列的位置选择序列（例如，兴趣点）[[21](https://arxiv.org/html/2402.14744v3#bib.bib21)]。这个序列表示为$\{(l_{0},t_{0}),(l_{1},t_{1}),\ldots,(l_{n},t_{n})\}$，其中每个$(l_{i},t_{i})$表示个体在时间$t_{i}$的地点$l_{i}$。

![参考说明](img/268a772e15ceebce4dbd290314c06d74.png)

图 2：LLMob，提出的用于个人出行生成的LLM代理框架。

通过将城市环境中的个体建模为LLM代理，我们提出了LLMob，一个用于个人出行生成的LLM代理框架，如图[2](https://arxiv.org/html/2402.14744v3#S3.F2 "Figure 2 ‣ 3 Methodology ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")所示。LLMob基于这样一个假设：个体的活动主要受两个主要因素的影响：习惯性活动模式和当前动机。习惯性活动模式代表了典型的运动行为和偏好，反映了常规的出行和位置选择，是推测日常活动的重要信息[[31](https://arxiv.org/html/2402.14744v3#bib.bib31), [7](https://arxiv.org/html/2402.14744v3#bib.bib7), [30](https://arxiv.org/html/2402.14744v3#bib.bib30)]。另一方面，动机涉及到动态和情境性因素，这些因素在特定时刻影响个体的选择，比如即时需求或特定时期的外部环境。这一考虑对于捕捉和预测短期内的出行模式变化至关重要[[1](https://arxiv.org/html/2402.14744v3#bib.bib1), [43](https://arxiv.org/html/2402.14744v3#bib.bib43)]。此外，通过制定假设特定关注事件的提示，该框架使我们能够在各种情境中观察LLM代理的响应。

为了构建一个活动轨迹生成的流水线，我们设计了一个具备行动、记忆和规划的LLM代理[[33](https://arxiv.org/html/2402.14744v3#bib.bib33), [32](https://arxiv.org/html/2402.14744v3#bib.bib32)]。行动指定了代理如何与环境交互并做出决策。在LLMob中，环境包含了从现实世界数据中收集的信息，而代理通过生成轨迹来执行操作。记忆包括过去的行动，这些行动需要提示给LLM以触发下一步动作。在LLMob中，记忆指的是代理输出的模式和动机。规划则是在过去的行动上制定或完善计划，以应对复杂任务，并可选地将额外信息作为反馈进行整合。在LLMob中，我们利用规划来识别模式和动机，从而处理轨迹生成这一复杂任务。计划的制定、选择、反思和完善[[14](https://arxiv.org/html/2402.14744v3#bib.bib14)]依次进行，代理会根据其观察不断更新行动计划[[41](https://arxiv.org/html/2402.14744v3#bib.bib41)]：代理首先通过从数据库中提取历史轨迹的候选模式来制定一组活动计划。然后，代理通过自一致性评估进行自我反思，从候选模式中挑选出最佳模式。随着从数据库中进一步检索历史轨迹，代理将识别的模式细化为日常活动的总结性动机，并将其与已识别的模式共同用于轨迹生成。除了上述代理组件，我们还建议代理的人物设定，这可以帮助LLM模拟现实世界中个体的多样性[[29](https://arxiv.org/html/2402.14744v3#bib.bib29)]。

### 3.1 活动模式识别

LLMob的第一阶段专注于从历史数据中识别活动模式。为了有效地利用提取的活动模式作为生成日常活动的基本先验知识，我们引入了以下两个步骤。

#### 3.1.1 从语义和历史数据中提取模式

该步骤基于活动轨迹数据（例如，个人签到数据）推导出活动模式。如图[2](https://arxiv.org/html/2402.14744v3#S3.F2 "图 2 ‣ 3 方法 ‣ 大型语言模型作为城市居民：个人出行生成的LLM代理框架")左侧面板所示，该方案包括以下几个方面：对于每个人，我们首先向LLM代理指定一个候选人物角色，为后续活动模式生成提供启发基础。这种方法还鼓励生成的活动模式的多样性，因为每个候选人物角色作为生成过程中的独特先验（例如，活动轨迹数据中用户聚类在产生有意义的区分方面的重要性已被证明[[24](https://arxiv.org/html/2402.14744v3#bib.bib24)]）。与此同时，我们进行数据预处理，从大量历史数据中提取关键信息。这包括识别常见的通勤距离、确定日常出行的典型起止时间和地点，并得出该人最常访问的地点。需要注意的是，这些信息在出行分析中被广泛认为是关键特征[[17](https://arxiv.org/html/2402.14744v3#bib.bib17)]。经过预处理后，历史数据中的语义元素与数据结合在提示中，要求LLM代理总结此人的活动模式。通过这样做，我们建立了一条流线型流程，有效地弥合了语义人物特征与具体历史活动轨迹数据之间的差距，从而实现对个人一天活动的更加个性化和可解释的表示。此外，我们建议在生成候选模式时，将候选人物角色添加到提示中，以促进结果的多样性。一般而言，对于每个人，根据历史数据和$C$（$C=10$）个候选人物角色，生成一组$C$个候选模式，记作$\mathcal{CP}$。我们在附录[C.4](https://arxiv.org/html/2402.14744v3#A3.SS4 "C.4 人物角色 ‣ 附录 C 实验设置 ‣ 大型语言模型作为城市居民：个人出行生成的LLM代理框架")中提供了这些候选人物角色的详细信息。

#### 3.1.2 基于自一致性的模式评估

这一步涉及评估候选模式的一致性，以识别最可信的模式。我们实现了一个评分机制来评估候选模式与历史数据的一致性。为了实现这个目标，我们定义了一个评分函数来衡量候选模式 $cp$ 在集合 $\mathcal{CP}$ 中的表现。该函数通过两个不同的活动轨迹集来评估 $cp$：一个是目标居民 $i$ 的特定活动轨迹 $\mathcal{T}_{i}$，另一个是来自其他居民的采样活动轨迹 $\mathcal{T}_{\sim i}$：

|  | $score_{cp}=\sum_{t\in\mathcal{T}_{i}}r_{t}-\sum_{t^{\prime}\in\mathcal{T}_{% \sim i}}r_{t^{\prime}},$ |  | (1) |
| --- | --- | --- | --- |

在这一过程中，我们设计了一个评估提示，要求LLM生成评分 $r_{t}$ 和 $r_{t^{\prime}}$。具体而言，LLM代理被提示根据候选模式评估给定轨迹的偏好程度。理想情况下，LLM代理应为来自目标居民的数据分配更高的 $r_{t}$，为来自其他居民的数据分配较低的 $r_{t^{\prime}}$。这个方案本质上识别了自洽模式：从目标用户的活动轨迹数据中得出的活动模式，在评估过程中应与该人的数据一致。我们在附录 [A](https://arxiv.org/html/2402.14744v3#A1 "附录 A 算法伪代码 ‣ 大型语言模型作为城市居民：个人出行生成的LLM代理框架") 中提供了LLMob第一阶段的算法伪代码。

### 3.2 动机驱动的活动生成

在LLMob的第二阶段，我们专注于动机的检索以及动机与活动模式的整合，以生成个体的活动轨迹。由于LLM的上下文长度有限，我们不能指望LLM能够消耗所有可用的历史信息并给出合理的输出。增强检索生成已被确定为提高LLM性能的关键因素[[37](https://arxiv.org/html/2402.14744v3#bib.bib37)]。这种增强提供了额外的信息，帮助LLM更有效地响应查询。尽管先前关于活动生成的研究主要忽视了宏观时间信息（例如，日期）或特定场景（例如，恶劣天气）的关键因素[[42](https://arxiv.org/html/2402.14744v3#bib.bib42)]，我们提出了一种更为精细的活动生成方法，通过利用LLM类人智能，考虑各种条件。例如，给定日期 $d$ 的动机和习惯性活动模式，可以推断出该日期的活动轨迹：

|  | $\mathcal{T}_{d}=LLM(\mathcal{M}otivation,\mathcal{P}attern).$ |  | (2) |
| --- | --- | --- | --- |

该生成方案指导LLM代理根据给定的活动模式模拟一个特定个体，然后根据每日动机精确地生成活动轨迹。为了获得对不同数据可用性和充分性的深刻且可靠的动机，提出了两种检索方案。值得注意的是，我们将它们视为解决现实世界应用的两个有前景的方向，而不是声称哪个更优。每种检索方案的详细信息如下所示：

![参见图注](img/d034bed8ae314b7356a7b46cd4fb94a9.png)

图3：基于演变的动机检索。

#### 3.2.1 基于演变的动机检索

该方案与一个直观的原理相关，即个体在某一天的动机受到前几天兴趣和优先事项的影响[[28](https://arxiv.org/html/2402.14744v3#bib.bib28)]。在这个理解的指导下，我们的方法利用LLM代理的智能来理解日常活动的行为及其背后的动机。如图[3](https://arxiv.org/html/2402.14744v3#S3.F3 "Figure 3 ‣ 3.2 Motivation-Driven Activity Generation ‣ 3 Methodology ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")所示，对于我们希望生成活动轨迹的特定日期$d$，我们考虑过去$k$天的活动（$k=\min(7,l)$，其中$l$是使得日期$d-l$的轨迹可以在数据库中找到的最大值），并提示LLM代理根据第[3.1](https://arxiv.org/html/2402.14744v3#S3.SS1 "3.1 Activity Pattern Identification ‣ 3 Methodology ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")节中识别的模式，作为一个城市居民来总结这些活动背后的$k$个动机。利用这些总结出的动机，LLM代理进一步被提示推断目标日期$d$的潜在动机。

#### 3.2.2 基于学习的动机检索

在此方案中，我们假设个体在日常活动中倾向于建立常规行为，这些行为是由一致的动机驱动的，即使具体位置可能有所变化。例如，如果某人常常在工作日早晨去汉堡店，这种行为可能表明其动机是为了快速吃早餐。基于这一点，可以推测同一人将来可能会选择不同的快餐店，动机是出于对早晨餐食的便利性和快速性的需求。我们引入了一种基于学习的方案，从历史数据中提取动机。对于每个新的日期，用于规划活动时，唯一可用的信息是该日期本身。为了利用这一线索进行规划，我们首先构建了一个相对时间特征 $\boldsymbol{z}_{{d_{c},d_{p}}}$，用于表示过去日期 $d_{p}$ 和当前日期 $d_{c}$ 之间的关系。该特征捕捉了多个方面，例如这两个日期之间的间隔时间以及它们是否属于同一个月。利用这一设置，我们训练了一个分数近似器 $f_{\theta}(\boldsymbol{z}_{{d_{c},d_{p}}})$ 来评估任意两天之间的相似度。值得注意的是，由于缺乏监督信号，我们采用了无监督学习来训练 $f_{\theta}(\cdot)$。特别地，基于对比学习的学习方案[[5](https://arxiv.org/html/2402.14744v3#bib.bib5)]被提出。对于一个居民的每个轨迹，我们可以扫描她的其他轨迹，并根据预定义的相似度分数识别相似（正样本）和不相似（负样本）的日期。该相似度分数是通过以下公式计算两个活动轨迹 $\mathcal{T}_{d_{a}}$ 和 $\mathcal{T}_{d_{b}}$ 之间的相似度：

|  | $sim_{d_{a},d_{b}}=\sum_{t=1}^{N_{d}}\mathbf{1}_{(\mathcal{T}_{d_{a}}(t)=% \mathcal{T}_{d_{b}}(t))}\text{ 如果 }&#124;\mathcal{T}_{d_{a}}&#124;>t\text{ 且 }&#124;% \mathcal{T}_{d_{b}}&#124;>t,$ |  | (3) |
| --- | --- | --- | --- |

其中 $N_{d}$ 是一天中的时间间隔总数（例如，10分钟）。$\mathcal{T}_{d_{a}}(t)$ 表示轨迹 $\mathcal{T}_{d_{a}}$ 中记录的第 $t$ 个访问位置。直观上，类似的轨迹对应该会有更多共享的位置。因此，正样本对的特点是相似度分数最高，表明轨迹之间有较高的相似度。相反，负样本对的相似度分数较低，反映出较少的共同点。在从这些正负样本对中获取训练数据集后，我们使用对比学习训练一个模型，用以近似任何两个日期之间的相似度分数。该过程包括以下步骤：

1.  1.

    对于每个日期 $d$，基于相似度分数生成一个正样本对 $(d,d^{+})$ 和 $k$ 个负样本对（$d,d^{-}_{1}$），…，($d,d^{-}_{k}$），并计算 $\boldsymbol{z}_{d,d^{+}}$，$\boldsymbol{z}_{d,d^{-}_{1}}$，…，$\boldsymbol{z}_{d,d^{-}_{k}}$。

1.  2.

    将正负样本对输入到 $f_{\theta}(\cdot)$ 以形成：

    |  | $\text{logits}=\left[f_{\theta}(\boldsymbol{z}_{d,d^{+}}),f_{\theta}(\boldsymbol{z}_{d,d^{-}_{1}}),...,f_{\theta}(\boldsymbol{z}_{d,d^{-}_{k}})\right].$ |  | (4) |
    | --- | --- | --- | --- |

1.  3.

    采用 InfoNCE [[25](https://arxiv.org/html/2402.14744v3#bib.bib25)] 作为对比损失函数：

    |  | $\mathcal{L}(\theta)=\sum_{n=1}^{N}-\log\left(\frac{e^{\text{logits}_{i}}}{\sum_{j=1}^{k+1}e^{\text{logits}_{j}}}\right)_{n},$ |  | (5) |
    | --- | --- | --- | --- |

    其中 $N$ 是样本的批量大小，$i$ 表示正样本对的索引。

通过训练相似度评分近似模型，可以用于评估任何给定查询日期与历史日期之间的相似度。这使我们能够检索最相似的历史数据，并将其提示给LLM代理，以生成当时流行动机的摘要。通过这种方式，我们可以推断出与查询日期相关的动机，为LLM代理生成新的活动轨迹提供依据。

## 4 实验

### 4.1 实验设置

数据集。我们在东京的个人活动轨迹数据集上进行研究和验证LLMob。该数据集通过Twitter和Foursquare API获取，涵盖了2019年1月到2022年12月的数据。该数据集的时间范围具有重要意义，因为它捕捉了COVID-19疫情前的典型日常生活（即正常时期）和疫情期间随之而来的变化（即异常时期）。为了便于对不同时间段进行成本效益高且详细的分析，我们随机选择了100个用户，根据可用的轨迹数量，以10分钟为间隔建模他们的个人活动轨迹。样本如表[1](https://arxiv.org/html/2402.14744v3#S4.T1 "Table 1 ‣ 4.1 Experimental Setup ‣ 4 Experiments ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")所示。

表 1：个人活动数据样本。

| 用户ID | 纬度 | 经度 | 位置名称 | 分类 | 时间 |
| --- | --- | --- | --- | --- | --- |
| 44673 | 35.008 | 139.015 | 便利店 | 商店与服务 | 2019-12-17 8:00 |
| 44673 | 35.009 | 139.018 | 拉面餐厅 | 食品与服务 | 2019-12-17 8:30 |
| 44673 | 35.004 | 139.060 | 意大利餐厅 | 食品与服务 | 2019-12-17 11:20 |
| 44673 | 35.009 | 139.085 | 农贸市场 | 商店与服务 | 2019-12-17 14:20 |
| 44673 | 35.005 | 139.086 | 荞麦面餐厅 | 食品与服务 | 2019-12-17 18:00 |

我们利用Foursquare中的类别分类来确定每个位置的活动类别。我们使用10个候选角色（附录[C.4](https://arxiv.org/html/2402.14744v3#A3.SS4 "C.4 Personas ‣ Appendix C Experimental Setup ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")）作为后续模式生成的先验，这些角色捕捉了本研究数据中各种不同的活动模式。在应用于其他数据集时，这种候选模式的风格可以通过LLM轻松初始化。

指标。以下与个人活动相关的特征用于考察生成情况：（1）步长（SD）[[42](https://arxiv.org/html/2402.14744v3#bib.bib42)]：收集每个连续决策步之间的行程距离。此指标通过测量轨迹中两个连续位置之间的距离来评估个人活动的空间模式。（2）步间隔（SI）[[42](https://arxiv.org/html/2402.14744v3#bib.bib42)]：记录每个连续决策步之间的时间间隔。此指标通过测量轨迹中两个连续位置之间的时间间隔来评估个人活动的时间模式。（3）每日活动例程分布（DARD）：对于每个决策步骤，创建一个元组$(t,c)$，其中$t$表示发生的时间间隔（例如，从0到144的时间范围），$c$根据该步骤访问的地点确定活动类别。随后构建一个直方图，表示收集到的元组的分布。此特征展示了由活动类型和时间（例如，活动类型、时间）所特征化的个人活动模式。它提供了活动在空间和时间上的分布情况，并反映了如习惯性行为等语义信息。（4）时空访问分布（STVD）：对于每个决策步骤，创建一个元组$(t,\text{纬度},\text{经度})$，其中$t$表示发生的时间间隔（例如，从0到144的时间范围），而$\text{纬度},\text{经度}$是该步骤访问地点的地理坐标。随后构建一个直方图，表示收集到的元组的分布。此特征通过评估每个轨迹中访问位置的时空分布，包括地理坐标和时间戳，提供了对生成活动的更细粒度视角。它使得我们能够详细分析活动发生的地点和时间。

在从生成数据和真实世界轨迹数据中提取上述特征后，采用Jensen-Shannon散度（JSD）来量化它们之间的差异。较低的JSD值是更为理想的。

方法。LLMob与以下模型进行对比评估：基于马尔可夫的机械模型（MM）[[27](https://arxiv.org/html/2402.14744v3#bib.bib27)]，基于LSTM的预测模型（LSTM）[[12](https://arxiv.org/html/2402.14744v3#bib.bib12)]，两种基于注意力的预测模型，包括DeepMove [[8](https://arxiv.org/html/2402.14744v3#bib.bib8)]和STAN [[22](https://arxiv.org/html/2402.14744v3#bib.bib22)]。在深度生成模型领域，我们选择了两个对抗学习框架，包括TrajGAIL [[6](https://arxiv.org/html/2402.14744v3#bib.bib6)]和ActSTD [[42](https://arxiv.org/html/2402.14744v3#bib.bib42)]，以及一个扩散模型，DiffTraj [[46](https://arxiv.org/html/2402.14744v3#bib.bib46)]。我们使用基准模型的源代码，由相应的作者提供，并根据我们的设置进行调整。

为了在能力与成本效率之间取得平衡，我们采用GPT-3.5-turbo-0613作为LLM核心。我们使用“LLMob-E”来表示基于演变动机检索方案的提案，使用“LLMob-L”来表示包含基于学习的动机检索方案的框架（参数设置见附录[C.3](https://arxiv.org/html/2402.14744v3#A3.SS3 "C.3 Learning-Based Motivation Retrieval ‣ Appendix C Experimental Setup ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")）。为了验证所提出的每个模块的必要性，我们进行以下配置的消融研究：“LLMob w/o $\mathcal{P}$”表示生成轨迹时不使用模式（即，直接从过去的轨迹总结动机）框架。“LLMob w/o $\mathcal{M}$”表示不使用动机的框架（即，直接生成具有已识别模式的轨迹）。“LLMob w/o $\mathcal{SC}$”表示没有自一致性评估的框架（在这种情况下，候选模式是随机选择的作为已识别模式）。此外，“LLMob w/o $\mathcal{P}$ & $\mathcal{M}$”表示不包括模式和动机的框架。

### 4.2 主要结果与分析

生成性能验证（RQ 1, RQ 2）。性能评估涉及在三种不同设置下分析生成结果：（1）基于2019年正常历史轨迹生成正常轨迹，这一时期未受大流行影响。（2）基于2020年异常历史轨迹生成异常轨迹，这一年受到大流行的影响。（3）基于2019年正常历史轨迹生成2021年（大流行）异常轨迹。

这些评估的结果详细记录在表格[2](https://arxiv.org/html/2402.14744v3#S4.T2 "Table 2 ‣ 4.2 Main Results and Analysis ‣ 4 Experiments ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")中。通过对比可以观察到，尽管LLMob在精确复制空间特征（SD）方面可能不占优势，但在处理时间方面（DI）表现优越。在考虑时空特征（DARD和STVD）时，LLMob的表现也具有竞争力。特别是，LLMob在所有三个设置中都在DI和DARD方面取得了最佳成绩，并在STVD中位居第二。像DeepMove和TrajGAIL这样的基线模型分别在SD和STVD方面表现最好，但在其他方面的评估中则竞争力大大下降。我们建议，LLMob在DARD方面明显的优势（大约比基线模型最好者低1/2到1/3的JSD）可以归因于LLM代理准确复制个体活动行为动机的倾向。例如，代理可能会识别出一个人早晨习惯吃早餐的模式，而不局限于特定的餐馆。这一现象突显了LLM代理增强的语义理解能力。

表格 2：基于历史数据的轨迹生成表现（JSD）。数值越低越好。最佳和亚军分别用粗体和下划线标出。

| 模型 | 正常轨迹，正常数据 | 异常轨迹，异常数据 | 异常轨迹，正常数据 |
| --- | --- | --- | --- |
| (# 生成的轨迹：1497) | (# 生成的轨迹：904) | (# 生成的轨迹：3555) |
| SD | SI | DARD | STVD | SD | SI | DARD | STVD | SD | SI | DARD | STVD |
| MM [[27](https://arxiv.org/html/2402.14744v3#bib.bib27)] | 0.018 | 0.276 | 0.644 | 0.681 | 0.041 | 0.300 | 0.629 | 0.682 | 0.039 | 0.307 | 0.644 | 0.681 |
| LSTM [[12](https://arxiv.org/html/2402.14744v3#bib.bib12)] | 0.017 | 0.271 | 0.585 | 0.652 | 0.016 | 0.286 | 0.563 | 0.655 | 0.035 | 0.282 | 0.585 | 0.653 |
| DeepMove [[8](https://arxiv.org/html/2402.14744v3#bib.bib8)] | 0.008 | 0.153 | 0.534 | 0.623 | 0.011 | 0.173 | 0.548 | 0.668 | 0.013 | 0.173 | 0.534 | 0.623 |
| STAN [[22](https://arxiv.org/html/2402.14744v3#bib.bib22)] | 0.152 | 0.400 | 0.692 | 0.692 | 0.115 | 0.092 | 0.693 | 0.691 | 0.142 | 0.094 | 0.692 | 0.690 |
| TrajGAIL [[6](https://arxiv.org/html/2402.14744v3#bib.bib6)] | 0.128 | 0.058 | 0.598 | 0.489 | 0.133 | 0.060 | 0.604 | 0.523 | 0.332 | 0.058 | 0.434 | 0.428 |
| ActSTD [[42](https://arxiv.org/html/2402.14744v3#bib.bib42)] | 0.034 | 0.436 | 0.693 | 0.692 | 0.071 | 0.469 | 0.692 | 0.692 | 0.022 | 0.093 | 0.468 | 0.692 |
| DiffTraj [[46](https://arxiv.org/html/2402.14744v3#bib.bib46)] | 0.052 | 0.251 | 0.318 | 0.692 | 0.008 | 0.240 | 0.339 | 0.692 | 0.101 | 0.142 | 0.218 | 0.693 |
| LLMob-E | 0.053 | 0.046 | 0.125 | 0.559 | 0.056 | 0.043 | 0.127 | 0.615 | 0.062 | 0.056 | 0.117 | 0.536 |
| LLMob-E w/o $\mathcal{P}$ | 0.055 | 0.069 | 0.223 | 0.530 | 0.059 | 0.081 | 0.252 | 0.673 | 0.065 | 0.079 | 0.209 | 0.561 |
| LLMob-E w/o $\mathcal{SC}$ | 0.058 | 0.076 | 0.295 | 0.589 | 0.068 | 0.086 | 0.225 | 0.649 | 0.072 | 0.096 | 0.301 | 0.589 |
| LLMob-L | 0.049 | 0.054 | 0.136 | 0.570 | 0.057 | 0.051 | 0.124 | 0.609 | 0.064 | 0.051 | 0.124 | 0.531 |
| LLMob-L w/o $\mathcal{P}$ | 0.061 | 0.080 | 0.270 | 0.600 | 0.072 | 0.081 | 0.286 | 0.641 | 0.073 | 0.091 | 0.248 | 0.580 |
| LLMob-L w/o $\mathcal{SC}$ | 0.057 | 0.074 | 0.236 | 0.602 | 0.071 | 0.084 | 0.236 | 0.642 | 0.073 | 0.094 | 0.286 | 0.622 |
| LLMob w/o $\mathcal{M}$ | 0.059 | 0.078 | 0.264 | 0.590 | 0.066 | 0.080 | 0.274 | 0.633 | 0.074 | 0.090 | 0.255 | 0.563 |
| LLMob w/o $\mathcal{P}$ & $\mathcal{M}$ | 0.061 | 0.081 | 0.268 | 0.606 | 0.068 | 0.086 | 0.287 | 0.635 | 0.074 | 0.095 | 0.254 | 0.573 |

探索现实世界应用中的效用（RQ 3）。我们感兴趣的是LLMob如何提升社会效益，尤其是在城市出行的背景下。为此，我们提出了一个示例，利用LLM代理的灵活性和智能来理解语义信息并模拟一个未知场景。具体来说，我们通过引入额外的提示来增强原始设置，为LLM代理提供上下文，使其能够在特定情况下规划活动。例如，一个“疫情”提示如下：现在是疫情期间，政府要求居民推迟旅行和活动，并尽可能进行远程办公。

![请参见图例](img/8ed360caa2b655dd01bb5e367a73b928.png)

图4：日常活动频率。

![请参见图例](img/b4c6b08671215f5904732a2177a70cbc.png)

(a) 艺术与娱乐。

![请参见图例](img/2b6372067b972e49c1be8c16d8d0380a.png)

(b) 专业及其他场所。

图5：疫情场景下的活动热图。

通过整合上述提示，我们可以观察到外部因素，如大流行和政府措施，如何影响城市流动性及相关的社会动态。我们使用大流行期间（2021年）的活动轨迹数据作为真实数据，并在图[4](https://arxiv.org/html/2402.14744v3#S4.F4 "Figure 4 ‣ 4.2 Main Results and Analysis ‣ 4 Experiments ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")中绘制了7个类别的每日活动频率。尽管TrajGAIL在表[2](https://arxiv.org/html/2402.14744v3#S4.T2 "Table 2 ‣ 4.2 Main Results and Analysis ‣ 4 Experiments ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")中提供了最佳的STVD，但在所有类别中表现出非常低的频率，且未能反映每个类别的趋势。相比之下，LLMob-L与通过大流行提示增强的版本之间的比较展示了外部因素的影响：带有大流行提示的版本中，活动频率显著下降，这在语义上抑制了可能传播疾病的活动（例如，食物）。

此外，从时空角度来看，选择了两个主要活动（例如，艺术与娱乐和专业与其他场所）来观察行为，如图 [5(a)](https://arxiv.org/html/2402.14744v3#S4.F5.sf1 "在图 5 ‣ 4.2 主要结果与分析 ‣ 4 实验 ‣ 大型语言模型作为城市居民：一个基于LLM的个人出行生成框架") 和 [5(b)](https://arxiv.org/html/2402.14744v3#S4.F5.sf2 "在图 5 ‣ 4.2 主要结果与分析 ‣ 4 实验 ‣ 大型语言模型作为城市居民：一个基于LLM的个人出行生成框架") 中所示。这些活动特别有意义，因为它们概括了疫情对居民工作与生活平衡及日常作息的影响。具体而言，借助疫情提示，LLMob重现了一个更加现实的时空活动模式。这种增强的生成现实性归因于整合了有关疫情影响及政府应对措施的先验知识，使得LLM代理能够以符合实际行为适应性的方式行动。例如，艺术与娱乐活动的减少反映了场所关闭和社交距离指导方针，而专业与其他场所活动的变化则表明了向远程办公转变和专业环境的转型。直观地，基于各种先验知识提示LLM代理生成活动显示出在实际应用中的巨大潜力。这种有条件的生成方法，加上可靠的生成结果，可以显著减轻城市管理者的工作负担。我们建议，这种工作流可以简化城市动态分析，并有助于评估城市政策的潜在影响。

### 4.3 消融研究

模式的影响。在表 [2](https://arxiv.org/html/2402.14744v3#S4.T2 "表 2 ‣ 4.2 主要结果与分析 ‣ 4 实验 ‣ 大型语言模型作为城市居民：一个基于LLM的个人出行生成框架") 中，通过比较使用与不使用模式的LLMob（“w/o $\mathcal{P}$”），我们观察到所识别的模式 consistently 提升了轨迹生成的表现。对DARD的改进最为显著（约减少50%的JSD），表明模式的使用是捕捉日常活动语义的关键因素。我们在附录 [D.1](https://arxiv.org/html/2402.14744v3#A4.SS1 "D.1 识别模式示例 ‣ 附录 D 额外实验结果 ‣ 大型语言模型作为城市居民：一个基于LLM的个人出行生成框架") 中提供了示例模式，展示了个人的习惯行为如何通过模式识别。

自我一致性评估的影响。通过比较有无自我一致性评估的LLMob（“w/o $\mathcal{SC}$”），我们在表格[2](https://arxiv.org/html/2402.14744v3#S4.T2 "Table 2 ‣ 4.2 Main Results and Analysis ‣ 4 Experiments ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")中发现，自我一致性在各方面都非常有用，尤其是在从正常数据生成异常轨迹时，它对DARD的影响最为显著，展现了其在处理语义方面的有效性。我们还观察到，“w/o $\mathcal{SC}$”在许多情况下的表现甚至比“w/o $\mathcal{P}$”还要差，因为在“w/o $\mathcal{SC}$”中，候选模式是随机挑选的，用于总结动机，这可能会导致个体日常活动的不一致性。

动机的影响。我们将有无动机的LLMob进行比较（“w/o $\mathcal{M}$”）。从表格[2](https://arxiv.org/html/2402.14744v3#S4.T2 "Table 2 ‣ 4.2 Main Results and Analysis ‣ 4 Experiments ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")中可以看出，动机的影响与模式的影响类似。通过与同时去除模式和动机的LLMob进行比较（“w/o $\mathcal{P}$ & $\mathcal{M}$”），我们观察到这两个因素共同作用导致了更好的性能。为了展示动机和生成的轨迹，我们在附录[D.2](https://arxiv.org/html/2402.14744v3#A4.SS2 "D.2 Examples of Retrieved Motivations and Corresponding Generated Trajectories ‣ Appendix D Additional Experimental Results ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")中提供了示例，能观察到它们之间的一致性。

动机检索策略的影响。我们比较了配备两种动机检索策略的LLMob（“-E”和“-L”）。表格[2](https://arxiv.org/html/2402.14744v3#S4.T2 "Table 2 ‣ 4.2 Main Results and Analysis ‣ 4 Experiments ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")显示，没有一种检索策略始终优于另一种，尽管基于演化的检索在更多情况下表现更好（7比5）。此外，基于演化的检索通常对去除模式或自我一致性评估的敏感度较低，表明依赖LLM处理历史轨迹比使用对比学习更具鲁棒性。

## 5 结论

贡献。本研究被认为是首个基于真实数据、由 LLM 代理驱动的个人移动仿真。我们的创新框架利用活动模式和动机，指导 LLM 代理模拟城市居民，促进了可解释和有效的个体活动轨迹生成。基于真实数据进行的大规模实验研究验证了所提出的框架，并展示了 LLM 代理在改善城市移动分析方面的潜力。

社会影响。利用人工智能增强社会福利日益成为一项有前景的事业，特别是随着高容量模型（如 LLMs）的出现。本研究探讨了使用 LLMs 作为可靠代理模拟特定场景，以评估外部因素（如疫情和政府政策）影响的潜在应用途径。所提出的框架提供了一种灵活的方法，增强了 LLM 在模拟城市移动性方面的可靠性。

局限性。在本研究中，我们专注于建模个体代理的活动，而没有考虑它们之间的互动。作为未来的工作，我们计划将这一研究扩展到多代理场景，以捕捉代理之间的互动（例如，个体可能会跟随朋友或家人的活动）。鉴于收集高质量个人移动数据的挑战——许多数据集无法完整地捕捉日常活动——我们将全面实验评估限制在了一个数据集上。此外，出于成本效益的考虑，只有 GPT-3.5 被全面评估。附录 [D.3](https://arxiv.org/html/2402.14744v3#A4.SS3 "D.3 大阪数据实验 ‣ 附录 D 其他实验结果 ‣ 大型语言模型作为城市居民：一个 LLM 代理框架用于个人移动生成") 中提供了在不同城市的附加分析，附录 [D.4](https://arxiv.org/html/2402.14744v3#A4.SS4 "D.4 不同 LLMs 实验 ‣ 附录 D 其他实验结果 ‣ 大型语言模型作为城市居民：一个 LLM 代理框架用于个人移动生成") 包括了使用其他 LLMs 的补充评估。

## 致谢

本研究得到了 JSPS KAKENHI JP22H03903、JP23H03406、JP23K17456、JP24K02996 和 JST CREST JPMJCR22M2 的资助支持。

## 参考文献

+   Aher 等人 [2022] Gati Aher, Rosa I Arriaga, 和 Adam Tauman Kalai. 使用大型语言模型模拟多个人类。*arXiv 预印本 arXiv:2208.10264*，2022年。

+   Argyle 等人 [2023] Lisa P Argyle, Ethan C Busby, Nancy Fulda, Joshua R Gubler, Christopher Rytting, 和 David Wingate. 一分为多：使用语言模型模拟人类样本。*政治分析*，31(3):337–351，2023。

+   Batty [2013] Michael Batty. *城市的新科学*。麻省理工学院出版社，2013年。

+   Batty 等 [2012] Michael Batty, Kay W Axhausen, Fosca Giannotti, Alexei Pozdnoukhov, Armando Bazzani, Monica Wachowicz, Georgios Ouzounis, 和 Yuval Portugali. 未来的智能城市. *欧洲物理杂志特别专题*, 214:481–518, 2012。

+   Chen 等 [2020] Ting Chen, Simon Kornblith, Mohammad Norouzi, 和 Geoffrey Hinton. 一种简单的框架用于对比学习视觉表示. 在 *国际机器学习会议*，页面 1597–1607\. PMLR，2020。

+   Choi 等 [2021] Seongjin Choi, Jiwon Kim, 和 Hwasoo Yeo. Trajgail：利用生成对抗模仿学习生成城市车辆轨迹. *交通研究 C部分：新兴技术*, 128:103091, 2021。

+   Diao 等 [2016] Mi Diao, Yi Zhu, Joseph Ferreira Jr, 和 Carlo Ratti. 从手机轨迹推断个体日常活动：波士顿示例. *环境与规划B：规划与设计*, 43(5):920–940, 2016。

+   Feng 等 [2018] Jie Feng, Yong Li, Chao Zhang, Funing Sun, Fanchao Meng, Ang Guo, 和 Depeng Jin. Deepmove：利用注意力递归网络预测人类移动性. 在 *2018年全球互联网会议论文集*，页面 1459–1468，2018。

+   Feng 等 [2020] Jie Feng, Zeyu Yang, Fengli Xu, Haisu Yu, Mudan Wang, 和 Yong Li. 学习模拟人类移动性. 在 *第26届 ACM SIGKDD国际知识发现与数据挖掘会议论文集*，页面 3426–3433，2020。

+   Gao 等 [2023] Chen Gao, Xiaochong Lan, Nian Li, Yuan Yuan, Jingtao Ding, Zhilun Zhou, Fengli Xu, 和 Yong Li. 大型语言模型赋能的基于代理的建模与仿真：一项调查与展望. *arXiv预印本 arXiv:2312.11970*, 2023。

+   Han 等 [2023] Xu Han, Zengqing Wu, 和 Chuan Xiao. "豚鼠试验"利用 GPT：一种基于智能代理的建模方法，用于研究公司竞争与合谋. *arXiv预印本 arXiv:2308.10974*, 2023。

+   Hochreiter 和 Schmidhuber [1997] Sepp Hochreiter 和 Jürgen Schmidhuber. 长短期记忆. *神经计算*, 9(8):1735–1780, 1997。

+   Huang 等 [2019] Dou Huang, Xuan Song, Zipei Fan, Renhe Jiang, Ryosuke Shibasaki, Yu Zhang, Haizhong Wang, 和 Yugo Kato. 基于变分自编码器的城市人类移动性生成模型. 在 *2019 IEEE多媒体信息处理与检索会议(MIPR)*，页面 425–430\. IEEE，2019。

+   Huang 等 [2024] Xu Huang, Weiwen Liu, Xiaolong Chen, Xingmei Wang, Hao Wang, Defu Lian, Yasheng Wang, Ruiming Tang, 和 Enhong Chen. 理解LLM代理的规划：一项调查. *arXiv预印本 arXiv:2402.02716*, 2024。

+   Jiang 等 [2018] Renhe Jiang, Xuan Song, Zipei Fan, Tianqi Xia, Quanjun Chen, Satoshi Miyazawa, 和 Ryosuke Shibasaki. Deepurbanmomentum：一个在线深度学习系统，用于短期城市移动性预测. 在 *AAAI人工智能会议论文集*，第32卷，2018。

+   Jiang 等人 [2021] Renhe Jiang, Xuan Song, Zipei Fan, Tianqi Xia, Zhaonan Wang, Quanjun Chen, Zekun Cai 和 Ryosuke Shibasaki. 通过POI嵌入在多个城市之间转移城市人类移动性。*ACM 数据科学学报*，2(1)：1–26，2021年。

+   Jiang 等人 [2016] Shan Jiang, Yingxiang Yang, Siddharth Gupta, Daniele Veneziano, Shounak Athavale 和 Marta C González. 无需旅行调查的城市移动时间地理建模框架。*美国国家科学院院刊*，113(37)：E5370–E5378，2016年。

+   Kingma 和 Ba [2014] Diederik P Kingma 和 Jimmy Ba. Adam：一种用于随机优化的方法。*arXiv 预印本 arXiv:1412.6980*，2014年。

+   Li 等人 [2023] Nian Li, Chen Gao, Yong Li 和 Qingmin Liao. 基于大型语言模型的代理用于模拟宏观经济活动。*arXiv 预印本 arXiv:2310.10436*，2023年。

+   Long 等人 [2023] Qingyue Long, Huandong Wang, Tong Li, Lisi Huang, Kun Wang, Qiong Wu, Guangyu Li, Yanping Liang, Li Yu 和 Yong Li. 基于变分点过程的实用合成人体轨迹生成。发表于 *第29届ACM SIGKDD知识发现与数据挖掘大会论文集*，页码4561–4571，2023年。

+   Luca 等人 [2021] Massimiliano Luca, Gianni Barlacchi, Bruno Lepri 和 Luca Pappalardo. 关于深度学习在人类移动性中的应用调查。*ACM 计算机调查 (CSUR)*，55(1)：1–44，2021年。

+   Luo 等人 [2021] Yingtao Luo, Qiang Liu 和 Zhaocheng Liu. Stan：下一位置推荐的空间-时间注意力网络。发表于 *2021年网页会议论文集*，页码2177–2185，2021年。

+   Mao 等人 [2023] Jiageng Mao, Yuxi Qian, Hang Zhao 和 Yue Wang. Gpt-driver：利用 GPT 学习驾驶。*arXiv 预印本 arXiv:2310.01415*，2023年。

+   Noulas 等人 [2011] Anastasios Noulas, Salvatore Scellato, Cecilia Mascolo 和 Massimiliano Pontil. 利用语义注释在基于位置的社交网络中聚类地理区域和用户。发表于 *国际AAAI网页与社交媒体会议论文集*，卷5，页码32–35，2011年。

+   Oord 等人 [2018] Aaron van den Oord, Yazhe Li 和 Oriol Vinyals. 使用对比预测编码的表示学习。*arXiv 预印本 arXiv:1807.03748*，2018年。

+   OpenAI [2022] OpenAI. 介绍 ChatGPT。*https://openai.com/blog/chatgpt*，2022年。

+   Pappalardo 和 Simini [2018] Luca Pappalardo 和 Filippo Simini. 基于数据驱动的空间-时间人类移动常规生成。*数据挖掘与知识发现*，32(3)：787–829，2018年。

+   Park 等人 [2023] Joon Sung Park, Joseph C O’Brien, Carrie J Cai, Meredith Ringel Morris, Percy Liang 和 Michael S Bernstein. 生成代理：人类行为的互动模拟。*arXiv 预印本 arXiv:2304.03442*，2023年。

+   Salewski 等人 [2023] Leonard Salewski, Stephan Alaniz, Isabel Rio-Torto, Eric Schulz 和 Zeynep Akata. 上下文伪装揭示了大型语言模型的优势和偏见。*arXiv 预印本 arXiv:2305.14930*，2023年。

+   宋等人 [2010] 宋朝鸣、科伦·塔尔、王普、巴拉巴希·阿尔伯特-拉斯洛。人类流动的尺度特性建模。*自然物理*，6(10):818–823，2010。

+   孙等人 [2013] 孙丽君、凯·W·阿克斯豪森、李德宏、黄贤峰。理解都市中日常遭遇的模式。*美国国家科学院院刊*，110(34):13774–13779，2013。

+   王等人 [2023] 王磊、马晨、冯学阳、张泽宇、杨浩、张敬森、陈志远、唐家凯、陈旭、林彦凯等人。基于大型语言模型的自主代理调查。*arXiv 预印本 arXiv:2308.11432*，2023。

+   翁 [2023] 翁丽莲。基于大型语言模型的自主代理。[https://lilianweng.github.io/posts/2023-06-23-agent/](https://lilianweng.github.io/posts/2023-06-23-agent/)，2023。

+   威廉姆斯等人 [2023] 罗斯·威廉姆斯、霍辛尼赫·尼尤莎、马久姆达尔·阿里特拉、加法扎根·纳维德。基于生成代理的流行病建模。*arXiv 预印本 arXiv:2307.04986*，2023。

+   吴等人 [2023] 吴增庆、彭润、韩旭、郑树源、张一新、肖传。智能代理建模：大型语言模型在计算机模拟中的应用。*arXiv 预印本 arXiv:2311.06330*，2023。

+   西等人 [2023] 西智恒、陈文翔、郭鑫、何伟、丁艺文、洪博阳、张铭、王俊哲、金森杰、周恩瑜等人。基于大型语言模型代理的崛起与潜力：一项调查。*arXiv 预印本 arXiv:2309.07864*，2023。

+   徐等人 [2023a] 徐鹏、平伟、吴贤超、劳伦斯·麦克阿菲、朱晨、刘子涵、桑迪普·苏布拉马尼安、埃芙琳娜·巴赫图里娜、穆罕默德·肖耶比、布赖恩·卡坦扎罗。信息检索与长上下文大语言模型的结合。*arXiv 预印本 arXiv:2310.03025*，2023a。

+   徐等人 [2023b] 徐晓航、铃村丰太郎、杨佳伟、花井雅俊、杨创、金广纪、姜仁赫、福岛慎太郎。重新审视基于图的流动建模：一种图变换器模型用于下一兴趣点推荐。在*第31届ACM地理信息系统国际会议论文集*，第1–10页，2023b。

+   徐等人 [2024] 徐晓航、姜仁赫、杨创、范子佩、关香一。驯服人类流动预测中的长尾问题。*arXiv 预印本 arXiv:2410.14970*，2024。

+   杨等人 [2016] 杨定琦、张大庆、屈冰清。基于位置社交网络的集体行为数据参与性文化地图绘制。*ACM智能系统与技术学报（TIST）*，7(3):1–23，2016。

+   姚等人 [2022] 姚顺宇、赵杰弗里、余淼、杜楠、沙夫兰·伊扎克、纳拉辛汉·卡尔提克、曹源。React：在语言模型中协同推理与行动。*arXiv 预印本 arXiv:2210.03629*，2022。

+   Yuan 等人 [2022] Yuan Yuan, Jingtao Ding, Huandong Wang, Depeng Jin 和 Yong Li. 通过建模时空动态生成活动轨迹。在 *第28届 ACM SIGKDD 知识发现与数据挖掘会议论文集* 中，第4752–4762页，2022年。

+   Yuan 等人 [2023] Yuan Yuan, Huandong Wang, Jingtao Ding, Depeng Jin 和 Yong Li. 通过建模动态人类需求学习模拟日常活动。在 *2023年 ACM Web会议论文集* 中，第906–916页，2023年。

+   Zhang 等人 [2020] Xin Zhang, Yanhua Li, Xun Zhou, Ziming Zhang 和 Jun Luo. Trajgail：用于长期决策分析的轨迹生成对抗模仿学习。在 *2020年 IEEE 数据挖掘国际会议（ICDM）* 中，第801–810页。IEEE，2020年。

+   Zheng [2015] Yu Zheng. 轨迹数据挖掘：概述。*ACM 智能系统与技术交易（TIST）*，6(3)：1–41，2015年。

+   Zhu 等人 [2024a] Yuanshao Zhu, Yongchao Ye, Shiyao Zhang, Xiangyu Zhao 和 James Yu. Difftraj：利用扩散概率模型生成 GPS 轨迹。*神经信息处理系统进展*，36，2024a年。

+   Zhu 等人 [2024b] Yuanshao Zhu, James Jianqiao Yu, Xiangyu Zhao, Qidong Liu, Yongchao Ye, Wei Chen, Zijian Zhang, Xuetao Wei 和 Yuxuan Liang. Controltraj：带有拓扑约束扩散模型的可控轨迹生成。在 *第30届 ACM SIGKDD 知识发现与数据挖掘会议论文集* 中，第4676–4687页，2024b年。

## 附录

## 附录 A 算法伪代码

活动模式识别算法的伪代码见算法 [1](https://arxiv.org/html/2402.14744v3#algorithm1 "在附录 A 算法伪代码 ‣ 作为城市居民的大型语言模型：用于个人出行生成的 LLM 代理框架")。

输入：人物数量 $C$，活动轨迹 $\mathcal{T}$。输出：最佳活动模式 $best\_pattern$。12 初始化：空的候选模式集合 $\mathcal{CP}$；3 使用从 $\mathcal{T}$ 中提取的先验信息制定初始模式总结提示 $p_{1}$；4 对于 $i=1$ 到 $C$，执行：5 形成包含先验信息和人物 $i$ 的提示 $p_{1}$；6 $cp \leftarrow \text{LLM}(p_{1})$；7 $\mathcal{CP} \leftarrow \mathcal{CP} \cup \{ cp \}$；8 910 初始化一个评分字典 $\mathcal{S}$ 用于存储模式评分；1112 对于每个候选模式 $cp$ 在 $\mathcal{CP}$ 中，执行：13 初始化 $score_{cp}=0$；14 对于每个活动轨迹 $t$ 在 $\mathcal{T}$ 中，执行：15 为 $cp$ 和 $t$ 制定评估提示 $p_{2}$；16 $score_{cp} \leftarrow score_{cp} + \text{LLM}(p_{2})$；17 $\mathcal{S}[cp] \leftarrow score_{cp}$；18 1920 $best\_pattern \leftarrow \mathop{\rm argmax}_{cp \in \mathcal{CP}} \mathcal{S}[cp]$；返回 $best\_pattern$

算法 1 活动模式识别

## 附录 B 提示

<svg class="ltx_picture" height="124.23" id="A2.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,124.23) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 102.24)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Pattern generation</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="78.72" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Context: Act as a $<$INPUT 0$>$ in an urban neighborhood, $<$INPUT 1$>$ $<$INPUT 2$>$. There are also locations you sometimes visit: $<$INPUT 3$>$. Instructions: Reflecting on the context given, I’d like you to summarize your daily activity patterns. Your description should be coherent, utilizing conclusive language to form a well-structured paragraph. Text to Continue: I am $<$INPUT 0$>$ in an urban neighborhood, $<$INPUT 1$>$,…</foreignobject></g></g></svg>

其中，$<$INPUT 0$>$ 和 $<$INPUT 1$>$ 被候选角色替代，$<$INPUT 2$>$ 和 $<$INPUT 3$>$ 将被从历史数据中提取的活动习惯替代。具体地，我们将 $<$INPUT 2$>$ 制定为以下格式：

<svg class="ltx_picture" height="105.63" id="A2.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,105.63) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 85.63)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 5.46)"><foreignobject color="#000000" height="10.15" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="123.23">$<$INPUT 2$>$ 格式</foreignobject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="62.11" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">在 $<$周末或工作日$>$，你通常每天旅行 $<$距离$>$ 公里，你通常在 $<$第一个日常活动的时间$>$ 开始日常旅行，在 $<$最后一个日常活动的时间$>$ 结束日常旅行。你通常在一天的开始访问 $<$第一个日常活动的地点$>$，并在返回家之前去 $<$最后一个日常活动的地点$>$。</foreignobject></g></g></svg><svg class="ltx_picture" height="105.09" id="A2.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,105.09) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 85.63)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 4.92)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">模式评估</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="62.11" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">现在你作为具有以下特征的人：$<$INPUT 0$>$。这里有一个计划给你。$<$INPUT 1$>$ 在1到10的评分范围内，其中1表示最低可能性，10表示最高可能性，请根据你的日常模式评估你遵循此计划的可能性，并解释原因。评分：$<$填写$>$ 原因：</foreignobject></g></g></svg>

其中，$<$INPUT 0$>$ 是候选模式，$<$INPUT 1$>$ 是每日活动计划，用于评估。

<svg class="ltx_picture" height="107.78" id="A2.p6.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,107.78) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 85.63)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Evolving-based motivation reterieval</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="62.11" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Context: Act as a person in an urban neighborhood and describe the motivation for your activities. $<$INPUT 0$>$ In the last $<$INPUT 1$>$ days you have the following activities: $<$INPUT 2$>$ Instructions: Describe in one sentence your future motivation today after these activities. Highlight any personal interests and needs.</foreignobject></g></g></svg>

其中，$<$INPUT 0$>$ 被选定的模式替代，$<$INPUT 1$>$ 被计划活动的天数替代，$<$INPUT 2$>$ 是对应选择日期的历史活动。

<svg class="ltx_picture" height="91.17" id="A2.p8.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,91.17) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 69.03)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Learning-based motivation reterieval</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="45.51" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Context: Act as a person in an urban neighborhood. $<$INPUT 0$>$ If you have the following plans: $<$INPUT 1$>$ Instructions: Try to summarize in one sentence what generally motivates you for these plans. Highlight any personal interests and needs.</foreignobject></g></g></svg>

其中，$<$INPUT 0$>$ 被选定的模式替代，$<$INPUT 1$>$ 被检索到的历史活动替代。

<svg class="ltx_picture" height="225.54" id="A2.p10.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,225.54) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 203.4)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">每日活动生成</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="179.88" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">上下文：作为一个城市社区中的人。$<$INPUT 0$>$ 以下是你想要达成的动机：$<$INPUT 1$>$ 指令：想一想你的日常生活。然后告诉我你今天的计划并解释它。以下是你可能访问的地点：$<$INPUT 2$>$ 对上述提示的回应格式如下：$\{$“plan”: [$<$Location$>$ 在 $<$Time$>$，$<$Location$>$ 在 $<$Time$>$，…]，“reason”:…$\}$ 示例：$\{$“plan”: [小学

#125 在 9:10，市政厅

#489 在 12:50，休息区

#585 在 13:40，海鲜餐厅

#105 在 14:20] “reason”: “我今天的计划是早上完成教学任务，然后找一些美味的食物品尝。”$\}$</foreignobject></g></g></svg>

其中 $<$INPUT 0$>$ 被替换为选择的模式，$<$INPUT 1$>$ 被替换为获取的动机，$<$INPUT 2$>$ 被替换为最常访问的地点。

<svg class="ltx_picture" height="71.88" id="A2.p12.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,71.88) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 52.43)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 4.92)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Pandemic</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="28.9" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Now it is the pandemic period. The government has asked residents to postpone travel and events and to telecommute as much as possible.</foreignobject></g></g></svg>

## 附录 C 实验设置

### C.1 数据处理

所有数据通过 Twitter 和 Foursquare API 获取，并在分析之前已经进行匿名处理，以去除任何可能的个人身份信息。详细过程如下：

1.  1.

    过滤不完整的数据

    被过滤掉的用户包括那些某一年没有签到记录的用户。

1.  2.

    排除非日本签到

    在日本以外发生的签到记录已被删除。

1.  3.

    从 GPS 坐标推断行政区

    根据签到的经纬度数据推断行政区。

1.  4.

    分配行政区

    用户根据其主要签到地点分配到一个行政区；例如，主要签到地点为东京的用户被归类为属于东京。

1.  5.

    移除突发移动签到

    显示出突兀、不现实的地点变动（例如，短时间内从东京到美国）的签到记录被删除，以避免数据漂移，按照 [[40](https://arxiv.org/html/2402.14744v3#bib.bib40)] 提出的标准进行处理。

1.  6.

    数据匿名化

    实际用户 ID 和地理位置名称已进行匿名处理。仅保留地理位置的类别信息，且经纬度坐标在输入到模型之前已转换为 ID。

### C.2 环境

我们利用 GPT API 进行生成研究。具体来说，我们使用的是 gpt-3.5-turbo-0613 版本的 API，这是 2023 年 6 月 13 日的 GPT-3.5-turbo 快照。实验是在以下配置的服务器上进行的：

+   •

    CPU：AMD EPYC 7702P 64核处理器

    +   –

        架构：x86_64

    +   –

        核心/线程：64 核心，128 线程

    +   –

        基本频率：2000.098 MHz

+   •

    内存：503 GB

+   •

    GPU：4 x NVIDIA RTX A6000

    +   –

        内存：每个 48GB

### C.3 基于学习的动机检索

对于基于学习的动机检索，评分近似器通过完全连接的神经网络进行参数化，网络架构如下：

表 3：评分近似器架构。

| 层 | 输入大小 | 输出大小 | 备注 |
| --- | --- | --- | --- |
| 输入层 | 3 | 64 | 线性 |
| 激活层 | - | - | ReLU |
| 输出层 | 64 | 1 | 线性 |

我们包括查询日期的年份中的天数、查询日期是否与参考日期属于相同的星期几，以及查询和参考日期是否在同一月内作为输入特征。学习过程的设置如下：Adam [[18](https://arxiv.org/html/2402.14744v3#bib.bib18)] 被用作优化器，批处理大小为 64，学习率为 0.002，负样本数为 2。

### C.4 人物角色

我们使用 10 个候选人物角色作为后续模式生成的先验信息，如表 [4](https://arxiv.org/html/2402.14744v3#A3.T4 "Table 4 ‣ C.4 Personas ‣ Appendix C Experimental Setup ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation") 所示。

表 4：建议的人物角色及相应描述。

| 学生：通常在相似的时间往返教育机构。 |
| --- |
| 教师：通常在相似的时间往返教育机构。 |
| 办公室工作人员：有固定的早晚通勤时间， |
| 通常前往办公室区或商业中心。 |
| 访客：全天出行， |
| 通常参观景点、餐饮区和购物区。 |
| 夜班工作者：可能在标准工作时间外出行， |
| 通常较晚在晚上或夜间出行。 |
| 远程工作者：可能有非标准的出行模式， |
| 通常在不同时间访问共享工作空间或咖啡馆。 |
| 服务行业工作者：全天出行，通常参观景点， |
| 餐饮区和购物区。 |
| 公共服务人员：通常轮班工作， |
| 导致全天候和夜间出行时间不同。 |
| 健身爱好者：通常在清晨或傍晚出行， |
| 或在周末前往健身中心或公园。 |
| 零售员工：出行模式可能包括轮班 |
| 晚上开始较晚，晚上结束。 |

## 附录 D 额外的实验结果

### D.1 识别模式示例

在我们框架的第一阶段，模式被提取和识别。我们报告了以下实验中识别到的一些模式示例，这些模式对应于表[4](https://arxiv.org/html/2402.14744v3#A3.T4 "Table 4 ‣ C.4 Personas ‣ Appendix C Experimental Setup ‣ Large Language Models as Urban Residents: An LLM Agent Framework for Personal Mobility Generation")中的10个角色。

<svg class="ltx_picture" height="324.98" id="A4.SS1.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,324.98) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 309.47)"><foreignobject color="#FFFFFF" height="9.61" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">学生的日常模式</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="277.97" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你是这个城市社区的学生。你通常在相似的时间上下学。作为城市社区的学生，你的日常生活围绕着往返教育机构的旅行。工作日，你每天要行驶超过10公里，早上12:00开始，晚上20:20结束。一天的开始，你通常会去公园#2457，然后去杂货店#648，再回家。周末的模式类似，尽管你每天要行驶的距离仍然超过10公里，但你从13:40开始，19:20结束。就像工作日一样，你开始时会去公园#2457，然后去杂货店#648，最后回家。整周，你有固定时间访问的一些特定地点，如20:00的杂货店#648、09:30的咖啡店#571、08:30的公园#2457、20:00的爱尔兰酒吧#21和16:00的书店#313。此外，还有一些你偶尔会去的地方，包括中餐馆#168、超市#802、公园#4719、茶室#530、书店#336、糕点店#240、公园#2898、折扣店#807和电子商店#530。通过在相似时间访问教育机构，你确保了你的学习准时且一致。此外，你去公园和各类商店也有放松、补充物资和享受休闲活动的作用。总体来说，你的日常生活有条不紊，能够高效地满足你教育和个人的需求。</foreignobject></g></g></svg><svg class="ltx_picture" height="357.82" id="A4.SS1.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,357.82) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 342.31)"><foreignobject color="#FFFFFF" height="9.61" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">教师的日常模式</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="310.81" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你是这个城市社区的教师。你通常在相似的时间上下学。作为城市社区的教师，你的日常生活围绕着往返教育机构的规律性出行。工作日，你每天要行驶超过60公里，早上11:50开始，晚上17:50结束。一天的开始，你通常会去休息区#1533，然后去住宅区#101，在那里花费大量时间，然后再回家。周末，你的行驶距离减少到每天50公里，早上11:20开始，晚上18:00结束。周末你遵循不同的例程，早上开始时先访问购物中心#1262，然后去摩托车商店#149，最后回家。此外，还有一些你偶尔会去的地方，如会议中心#101、美食广场#559、摩托车商店#354和运动酒吧#56等。你的日常生活受到履行教学责任的需求驱动，确保你能在教育机构尽职尽责。你访问的特定地点，无论是休息区#1533还是购物中心#1262，都能为你提供必要的资源、放松或从事个人兴趣的机会。总体而言，你的日常生活安排得当，能够平衡职业承诺与个人需求。</foreignobject></g></g></svg><svg class="ltx_picture" height="225.35" id="A4.SS1.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,225.35) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 209.84)"><foreignobject color="#FFFFFF" height="9.61" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">办公室员工的日常模式</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="178.34" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你是这个城市社区的办公室员工。你有固定的上下班时间，通常前往办公区或商业中心。作为城市社区的办公室员工，你的日常生活相当有规律。工作日，你每天要行驶超过30公里，早上06:50开始，晚上20:20结束。你的例程通常始于访问平台#212，然后前往荞麦面餐厅#955，再回家。你还有一些固定时间访问的地方，如10:30的医院#1255、06:30的平台#212、22:30的荞麦面餐厅#955、08:30的公共厕所#76和05:00的平台#1068。周末，你的日常出行距离增加到超过40公里，早上07:00开始，晚上20:10结束。但模式保持不变，通常从访问平台#212开始，然后去荞麦面餐厅#955，最后回家。</foreignobject></g></g></svg><svg class="ltx_picture" height="308.38" id="A4.SS1.p5.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,308.38) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 292.86)"><foreignobject color="#FFFFFF" height="9.61" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">访客的日常模式</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="261.37" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">你是这个城市社区的访客。你通常在一天中四处旅行，参观各类景点、餐饮区和购物区。作为这个城市社区的访客，你的日常生活包括广泛的旅行，探索各种景点、餐饮区和购物区。工作日，你每天行驶超过40公里，早上11:00开始，下午17:10结束。你通常从访问拉面餐厅#4841开始，然后前往镇#373，再回家。周末有所不同，行驶的距离缩短到约30公里。你从14:00开始，16:50结束。周末的模式通常是从访问拉面餐厅#3773开始，然后前往镇#373，最后回家。你还会访问一些固定地点，如13:30的街机厅#929、11:00的保龄球馆#306、12:00的

### D.2 检索到的动机示例及相应生成的轨迹

检索到的动机激励智能体规划与其具体需求密切相关的日常活动。以下是我们实验中检索到的一些动机示例及相应生成的日常活动。

<svg class="ltx_picture" height="179.77" id="A4.SS2.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,179.77) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 161.56)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">示例 1</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="130.07" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">动机：你的动机主要源自于对娱乐和放松的渴望，因为你喜欢在深夜去保龄球馆和丼饭餐厅，访问超市和休息区以满足日常需求并休息。   生成的日常活动轨迹：2019年12月27日的活动：印刷店（35.697, 139.404）在09:10:00，娱乐场所（35.733, 139.434）在10:10:00，丼饭餐厅（35.730, 139.435）在10:30:00，平台（35.733, 139.435）在10:50:00，便利店（35.721, 139.397）在11:20:00，平台（35.729, 139.392）在21:30:00，丼饭餐厅（35.699, 139.412）在22:10:00，便利店（35.697, 139.416）在22:40:00。</foreignobject></g></g></svg><svg class="ltx_picture" height="179.77" id="A4.SS2.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,179.77) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 161.56)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">示例 2</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="130.07" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">动机：你的动机围绕着满足基本需求，如食物、便利、放松和个人兴趣，包括喜欢日本料理、保持健康的生活方式，以及享受晚上的社交聚会。   生成的日常活动轨迹：2019年12月25日的活动：宠物店（35.699, 139.404）在08:10:00，便利店（35.697, 139.416）在13:10:00，健身房（35.697, 139.412）在13:20:00，按摩工作室（35.698, 139.413）在14:50:00，电子商店（35.699, 139.414）在15:50:00，快餐店（35.697, 139.413）在17:10:00，平台（35.698, 139.415）在18:10:00，便利店（35.701, 139.393）在19:10:00。</foreignobject></g></g></svg><svg class="ltx_picture" height="167.39" id="A4.SS2.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,167.39) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 149.19)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">示例 3</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="117.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">动机：你在城市社区中的动机主要由对食物和便利的需求驱动，确保个人安全和保障，履行工作责任，并满足个人对食物的偏好。   生成的日常活动轨迹：2019年12月30日的活动：公园（35.779, 139.491）在06:40:00，便利店（35.831, 139.519）在07:10:00，折扣商店（36.356, 139.064）在10:40:00，辅助生活（36.338, 139.048）在12:00:00，银行（36.361, 139.059）在12:10:00，家政服务（36.352, 139.050）在15:30:00，寿司餐厅（36.373, 139.061）在19:40:00，温泉（36.340, 139.052）在20:30:00。</foreignobject></g></g></svg><svg class="ltx_picture" height="163.16" id="A4.SS2.p5.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,163.16) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 144.96)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">示例 4</foreignobject></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="113.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">动机：你的动机围绕着满足基本需求，如食物、便利、放松和个人兴趣，包括喜欢日本料理、保持健康的生活方式，以及享受晚上的社交聚会。   生成的日常活动轨迹：2019年12月29日的活动：公园（35.779, 139.491）在08:20:00，活动空间（35.630, 139.790）在10:40:00，会议中心（35.629, 139.795）在12:20:00，便利店（35.647, 139.817）在13:00:00，荞麦面餐厅（35.643, 139.820）在13:10:00，运动用品商店（35.695, 139.762）在14:50:00，串炸餐厅（35.778, 139.495）在16:40:00。</foreignobject></g></g></svg><svg class="ltx_picture" height="146.56" id="A4.SS2.p6.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,146.56) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 128.35)"><foreignobject color="#FFFFFF" height="12.3" overflow="visible" transform="matrix(

### D.3 大阪数据集实验

我们基于日本大阪收集的数据进行了实验。我们基于来自30人的2102条日常活动轨迹生成了537条轨迹。结果如下，其中LLMob-L/E是我们的模型，而DiffTraj和TrajGAIL是表现最好的基准方法。

表5：基于大阪数据的各种度量模型比较

| 模型 | SD | SI | DARD | STVD |
| --- | --- | --- | --- | --- |
| LLMob-L | 0.035 | 0.021 | 0.141 | 0.391 |
| LLMob-E | 0.030 | 0.018 | 0.121 | 0.380 |
| DiffTraj | 0.080 | 0.177 | 0.406 | 0.691 |
| TrajGAIL | 0.281 | 0.063 | 0.525 | 0.483 |

### D.4 不同LLM的实验

我们进行了基于不同LLM（GPT-4o-mini和Llama 3-8B）的设置实验（1）。结果如下：

表6：使用不同LLM的实验结果

| 模型 | SD | SI | DARD | STVD |
| --- | --- | --- | --- | --- |
| LLMob-L (GPT-3.5-turbo) | 0.049 | 0.054 | 0.136 | 0.570 |
| LLMob-L (GPT-4o-mini) | 0.049 | 0.055 | 0.141 | 0.577 |
| LLMob-L (Llama 3-8B) | 0.054 | 0.063 | 0.119 | 0.566 |
| LLMob-E (GPT-3.5-turbo) | 0.053 | 0.046 | 0.125 | 0.559 |
| LLMob-E (GPT-4o-mini) | 0.041 | 0.053 | 0.211 | 0.531 |
| LLMob-E (Llama 3-8B) | 0.054 | 0.059 | 0.122 | 0.561 |

我们观察到，当使用其他LLM时，我们框架的表现仍具有竞争力。特别地，GPT-4o-mini在空间度量（SD）上表现最佳；GPT-3.5-turbo在时间度量（SI）上表现最佳。而Llama 3-8B在综合考虑空间和时间因素（DARD和STVD）时表现最佳。这些结果证明了我们框架在不同LLM中的鲁棒性。
