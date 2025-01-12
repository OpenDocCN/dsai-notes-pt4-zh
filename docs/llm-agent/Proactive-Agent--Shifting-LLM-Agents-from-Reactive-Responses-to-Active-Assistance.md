<!--yml

类别：未分类

日期：2025-01-11 12:05:44

-->

# 主动智能体：将大语言模型（LLM）智能体从反应式回应转变为主动协助

> 来源：[https://arxiv.org/html/2410.12361/](https://arxiv.org/html/2410.12361/)

陆雅熙¹, 杨神志², 钱成¹, 陈贵荣², 罗勤宇¹, 吴业赛¹, 王华东¹,

宗昕¹, 张忠¹, 林彦凯², 刘伟文³, 王亚胜³,

刘志远¹, 刘方明⁴, 孙茂松¹

¹ 清华大学计算机科学与技术系

² 中国人民大学高龄人工智能学院

³ 华为诺亚方舟实验室, ⁴ 彭程实验室

lyx23@mails.tsinghua.edu.cn, mrlyk423@gmail.com, liuzy@tsinghua.edu.cn

###### 摘要

由大语言模型驱动的智能体在解决复杂任务方面展示了出色的能力。然而，大多数智能体系统仍然是反应式的，这限制了它们在需要前瞻性和自主决策的场景中的有效性。本文中，我们解决了开发能够预测和主动发起任务的智能体这一挑战，而无需明确的人类指令。我们提出了一种新颖的数据驱动方法来应对这一问题。首先，我们收集现实世界中的人类活动数据，用于生成主动任务预测。然后，这些预测会被人工标注员标记为接受或拒绝。标注后的数据用于训练一个奖励模型，该模型模拟人类判断，并作为自动评估LLM智能体主动性的工具。在此基础上，我们开发了一个全面的数据生成管道，创建了一个多样化的数据集ProactiveBench，其中包含6,790个事件。最后，我们展示了通过使用ProactiveBench进行微调，能够显著激发LLM智能体的主动性。实验结果表明，我们微调后的模型在主动提供协助方面达到了66.47%的F1得分，优于所有开源和闭源模型。这些结果突出了我们方法在创造更主动和高效的智能体系统方面的潜力，为未来人类与智能体协作的进展铺平了道路。¹¹1我们的代码和数据可以在[https://github.com/thunlp/ProactiveAgent](https://github.com/thunlp/ProactiveAgent)获取。

## 1 引言

![参考说明](img/2b9a52f499bff79f071ef9162f177167.png)

图1：智能体系统与两种类型的人类-智能体交互比较。反应式智能体被动地接收用户查询，然后生成回应。主动智能体根据环境观察推测任务，并提出相应的协助请求。

大型语言模型（LLM）的出现，如ChatGPT[[1](https://arxiv.org/html/2410.12361v3#bib.bib1)]，极大推动了自主代理的发展[[2](https://arxiv.org/html/2410.12361v3#bib.bib2), [3](https://arxiv.org/html/2410.12361v3#bib.bib3), [4](https://arxiv.org/html/2410.12361v3#bib.bib4), [5](https://arxiv.org/html/2410.12361v3#bib.bib5)]。这些基于LLM的代理能够理解人类指令、制定计划、探索环境，并利用工具解决复杂任务[[5](https://arxiv.org/html/2410.12361v3#bib.bib5), [6](https://arxiv.org/html/2410.12361v3#bib.bib6)]，并在诸如机器人技术[[7](https://arxiv.org/html/2410.12361v3#bib.bib7)]、个人助理[[8](https://arxiv.org/html/2410.12361v3#bib.bib8)]和过程自动化[[9](https://arxiv.org/html/2410.12361v3#bib.bib9)]等应用中展现出巨大的潜力。

目前，大多数基于大语言模型（LLM）的代理主要工作在反应式范式中：它们需要明确的人工指令来启动任务的完成，并且在没有用户指令的情况下保持沉默，无法主动提供服务[[10](https://arxiv.org/html/2410.12361v3#bib.bib10)]。这种范式限制了它们在没有直接人工指令的情况下进行主动协助和自主服务提供的能力。我们认为，基于LLM的代理应该是主动的，能够通过理解和响应环境来自主启动任务。例如，如[图1](https://arxiv.org/html/2410.12361v3#S1.F1 "在 1 引言 ‣ 主动代理：将LLM代理从反应式响应转变为主动协助")所示，反应式代理应该等待用户的明确指令才能执行诸如“显示未读邮件”或“与John安排会议”之类的任务。相比之下，主动代理会通过注意到John发来的邮件提议会议，自动预测任务并主动提供安排会议的服务。这种情境感知能力[[11](https://arxiv.org/html/2410.12361v3#bib.bib11)]使得主动代理能够解读信号并主动提出和执行任务，而无需明确的人工指令。因此，它不仅显著减轻了用户的认知负担，还能识别出人类未明确表达的潜在需求。因此，主动代理能够为用户提供更全面、更无缝的服务。

| 子集 | 场景 | 条目（Tr/Ts） |
| --- | --- | --- |
| 代理模型 | 136 | 6,790 / 233 |
|    编码 | 46 | 2,275 |
|    写作 | 46 | 2,354 |
|    日常生活 | 44 | 2,161 |
| 奖励模型 | - | 1,640 / 120 |

表1：ProactiveBench的统计数据，包含三种不同的设置：编码、写作和日常生活。代理模型的子集包含$6,790$个用于训练的事件和$233$个用于测试的事件。奖励模型的子集包含$1,640$个用于训练的标注标签和$120$个用于测试的标签。

在本研究中，我们提出了一种新的数据驱动形式化方法，用于开发一种能够预见用户需求并主动采取行动的智能体，能够在没有明确请求的情况下，通过建议任务或提供信息来主动帮助用户。我们的方法以构建ProactiveBench为核心，使我们能够评估和增强智能体的主动行为。首先，我们在三种场景中收集真实世界的用户活动数据：编程、写作和日常生活。数据包括但不限于用户的键盘和鼠标输入、剪贴板内容、浏览器活动等。然后，我们构建了一个基于LLM的健身环境，用于生成反映我们收集的原始真实世界情境的事件。我们总共收集了$233$个事件，涵盖$12$个场景，作为ProactiveBench的测试集。为了进一步优化基于LLM的智能体的主动行为，我们在合成情境中使用该健身环境构建了各种事件和主动任务。通过反复生成更多的事件和预测，我们获得了最多$6,790$个事件，作为ProactiveBench的训练集，如[表1](https://arxiv.org/html/2410.12361v3#S1.T1 "在 1 引言 ‣ 主动智能体：将LLM智能体从反应式响应转变为主动协助")所示。我们在该训练集上对LLaMA-3.1-8B-Instruct [[12](https://arxiv.org/html/2410.12361v3#bib.bib12)]和Qwen2-7B-Instruct [[13](https://arxiv.org/html/2410.12361v3#bib.bib13)]进行了微调，以优化它们的主动行为。

为了自动评估LLM的主动性，我们训练了一个奖励模型，该模型在F1-Score方面与人类判断的一致性达到了$91.80\%$，作为评估工具。通过使用奖励模型，我们比较了不同语言模型在ProactiveBench上的表现。结果表明，即使是最新的开源模型，在有效预测主动任务方面仍然存在困难。例如，LLaMA-3.1-8B-Instruct模型在ProactiveBench上的F1-Score仅为$55.06\%$。相比之下，我们微调后的模型表现出显著改善，达到了$66.25\%$的F1-Score。此外，我们微调后的Qwen2-7B-Instruct模型达到了$66.47\%$的F1-Score，超越了所有现有的开源和闭源LLM。这凸显了我们基于数据驱动方法在开发主动智能体方面的有效性，强调了其在各类应用中提升用户体验的潜力。

## 2 相关工作

最近在大规模语言模型方面的进展[[14](https://arxiv.org/html/2410.12361v3#bib.bib14), [15](https://arxiv.org/html/2410.12361v3#bib.bib15), [12](https://arxiv.org/html/2410.12361v3#bib.bib12), [16](https://arxiv.org/html/2410.12361v3#bib.bib16)] 显示了在复杂推理、任务规划[[17](https://arxiv.org/html/2410.12361v3#bib.bib17), [18](https://arxiv.org/html/2410.12361v3#bib.bib18), [19](https://arxiv.org/html/2410.12361v3#bib.bib19), [20](https://arxiv.org/html/2410.12361v3#bib.bib20), [21](https://arxiv.org/html/2410.12361v3#bib.bib21), [22](https://arxiv.org/html/2410.12361v3#bib.bib22), [9](https://arxiv.org/html/2410.12361v3#bib.bib9)]、工具利用[[23](https://arxiv.org/html/2410.12361v3#bib.bib23), [24](https://arxiv.org/html/2410.12361v3#bib.bib24), [25](https://arxiv.org/html/2410.12361v3#bib.bib25), [26](https://arxiv.org/html/2410.12361v3#bib.bib26)]等方面取得了显著进展。因此，越来越多的智能体系统被开发出来，利用这些模型来处理各种任务，如自动网页搜索[[27](https://arxiv.org/html/2410.12361v3#bib.bib27)]、软件开发[[28](https://arxiv.org/html/2410.12361v3#bib.bib28), [2](https://arxiv.org/html/2410.12361v3#bib.bib2)]、行为模拟[[29](https://arxiv.org/html/2410.12361v3#bib.bib29)]。尽管取得了这些进展，目前的大多数智能体仍然主要是反应式的，被动地执行人类指令，缺乏足够的上下文意识[[11](https://arxiv.org/html/2410.12361v3#bib.bib11)]，无法主动满足用户需求。这些反应式智能体通常等待用户明确的指令，随着任务复杂度的增加，这可能导致低效。因此，用户必须不断提供具体输入，阻碍了交互的流畅性。为此，一些研究试图提高智能体的主动性。例如，Xuan[[30](https://arxiv.org/html/2410.12361v3#bib.bib30)] 提出了主动智能体规划，智能体通过主动寻找信息来改进任务，以更好地理解用户意图。程况和志瑞[[31](https://arxiv.org/html/2410.12361v3#bib.bib31)]研究了如何在文本到SQL的场景中提出主动支持，并提出了一个新颖的指标——负担曲线下的面积（AUDBC）。其他研究[[32](https://arxiv.org/html/2410.12361v3#bib.bib32), [33](https://arxiv.org/html/2410.12361v3#bib.bib33), [34](https://arxiv.org/html/2410.12361v3#bib.bib34), [35](https://arxiv.org/html/2410.12361v3#bib.bib35)]专注于使多轮交互能够澄清模糊的用户指令，这进一步增加了用户的认知负担。然而，这些工作仍然要求用户在与智能体交互之前提供初始查询。我们的方法采取了不同的方向，专注于通过监控用户活动和环境状态来预测潜在的任务，这使得智能体能够主动启动交互并提供帮助。

为了澄清，也有一些先前的工作 [[36](https://arxiv.org/html/2410.12361v3#bib.bib36)] 使用了“主动代理”一词来描述他们的对话系统。然而，这些工作的主要目标是增强响应的主动性和质量，与我们关注的任务预判和启动有所不同 [[37](https://arxiv.org/html/2410.12361v3#bib.bib37), [38](https://arxiv.org/html/2410.12361v3#bib.bib38), [39](https://arxiv.org/html/2410.12361v3#bib.bib39), [40](https://arxiv.org/html/2410.12361v3#bib.bib40)]。

## 3 方法论

### 3.1 任务定义

在我们提出的主动代理中，它不同于传统的由大型语言模型驱动的代理系统，这些系统依赖于明确的用户指令。我们研究了一种新的场景，其中代理自动预测用户可能分配的任务，旨在主动提供帮助，如 [图1](https://arxiv.org/html/2410.12361v3#S1.F1 "在1介绍 ‣ 主动代理：将LLM代理从反应型响应转变为主动帮助")所示。主动代理的任务是基于用户活动 $A_{t}$、环境事件 $E_{t}$ 和状态 $S_{t}$ 给出预测，可以公式化为：

|  | $P_{t}=f_{\theta}(E_{t},A_{t},S_{t}),$ |  | (1) |
| --- | --- | --- | --- |

其中 $f_{\theta}$ 表示由 $\theta$ 参数化的主动代理，$P_{t}$ 表示在时间 $t$ 对可能任务的预测。需要注意的是，预测 $P_{t}$ 可以是预测的任务，也可以是“无任务”，如果代理认为不需要任务。具体来说，用户活动 $A_{t}$ 包含用户与环境和代理的互动，例如键盘输入或与代理聊天。环境事件 $E_{t}$ 包含主动代理捕捉到的事件，从接收到新电子邮件到应用程序关闭。环境状态 $S_{t}$ 表示当前环境的状态，如文件系统状态或已打开网页的内容。

在我们的主动代理框架中，目标是最大化用户对提出任务的接受率。给定用户的历史活动 $A_{t}$、当前的环境状态 $S_{t}$ 和主动代理提出的预测 $P_{t}$，用户做出一个二元决策：

|  | $R_{t}=g(P_{t},A_{t},S_{t}),$ |  | (2) |
| --- | --- | --- | --- |

其中 $R_{t}$ 是一个二元变量，表示对预测的接受（$R_{t}=1$）或拒绝（$R_{t}=0$）。为了统一处理预测 $P_{t}$ 包含无任务和包含任务的情况，我们引入了一个辅助变量 $N_{t}$，用于指示用户是否需要帮助：

+   •

    如果用户需要帮助，则 $N_{t}=1$。

+   •

    如果用户不需要帮助，则 $N_{t}=0$。

用户的接受度 $R_{t}$ 定义为：

|  | $R_{t}=\begin{cases}1&\text{如果 }(P_{t}\neq\emptyset\text{ 且用户接受 }P_{t})\text{ 或 }(P_{t}=\emptyset\text{ 且 }N_{t}=0)\\ 0&\text{否则}\end{cases}.$ |  |
| --- | --- | --- |

通过这种方式，如果预测$P_{t}$不包含任务（即代理认为用户不需要帮助），我们会检查用户的实际帮助需求$N_{t}$。如果用户确实不需要帮助（$N_{t}=0$），则标记为接受（$R_{t}=1$）。反之，如果用户需要帮助（$N_{t}=1$），则标记为拒绝（$R_{t}=0$）。我们的主动代理旨在最大化提出任务的预期接受率：

|  | $\max_{\theta}\mathbb{E}[R_{t}].$ |  | (3) |
| --- | --- | --- | --- |

### 3.2 流水线概述

![请参考说明](img/00ec9a59818dbaa4be98d795a49932c6.png)

图 2：数据生成过程概述。以日常生活为例，该过程包括初始场景和任务设置、事件生成、主动预测、用户判断和行动执行等模块。

为了增强我们基于大型语言模型的代理的主动能力，我们采用基于数据的方法，构建了一个自动化数据生成流水线。该流水线模拟用户活动和响应，针对不同场景预测主动代理的任务。一旦预测被接受，我们通过交互式地生成新事件来模拟代理执行任务。随后，根据历史事件创建新的用户活动，允许主动代理生成进一步的预测。通过这个流水线，模型能够学习何时生成预测以及哪些预测可能会被用户接受。具体而言，我们的流水线包括三个组件：

(1) 环境模拟器：该组件模拟特定背景设置和示例事件中的事件，为主动代理提供一个互动的沙箱。它具有两个关键功能：（i）事件生成：创建针对特定场景的潜在环境事件序列；（ii）状态维护：在生成新的用户活动或代理在任务执行过程中执行操作时，更新和维护环境状态。

(2) 主动代理：该组件负责根据从事件历史中推断出的用户需求预测用户可能会分配给代理的任务。它还与工具互动，以完成用户分配的特定任务。

(3) 用户代理：该组件基于预定义的用户特征模拟用户的活动和响应。它决定是否接受并执行代理提出的任务。

在接下来的章节中，我们将介绍每个组件的详细信息。

### 3.3 环境模拟器

#### 事件收集

为了提高由环境训练平台生成事件的质量，我们首先收集现实世界中的事件作为参考。我们开发了一款基于Activity Watcher²²2[https://github.com/ActivityWatch/activitywatch](https://github.com/ActivityWatch/activitywatch)的监控软件，可以捕捉用户与计算机系统交互的详细信息，包括键盘和鼠标操作、访问的网页以及使用的开发工具。为了增强所收集数据的语义丰富性并促进大型语言模型的解析，我们将原始数据进一步合并成逻辑上连贯的片段。此外，我们利用语言模型将结构化数据转换为更自然的文本描述。这个过程不仅提高了数据的可解释性，还使其更适合后续的使用。

#### 场景生成

在收集参考事件后，我们并不直接生成特定的事件，而是先生成一个逼真的互动场景，为进一步生成提供充分的背景信息。为了构建这样的场景，我们首先使用人类标注者收集的种子任务提示GPT-4o [[41](https://arxiv.org/html/2410.12361v3#bib.bib41)]，创建用户在特定类别下可能执行的各种任务，比如编程、写作或日常生活。接着，我们生成任务可能涉及的所有实体，例如浏览器、软件和工具，以便代理执行任务。然后，我们通过添加更多细节，如实体状态或日期时间，来优化场景，进一步完善细节。最后，收集到的事件也会用于生成每个特定情境下的示例事件，以便为未来的事件生成提供参考。这使我们能够控制将要生成的事件的粒度，并保持场景的多样性。具体的提示模板，请参见[附录C](https://arxiv.org/html/2410.12361v3#A3 "Appendix C Prompt Template for Environment Gym ‣ Limitations ‣ 5 Conclusion ‣ 4.4 Case Study ‣ Feedback From the Reward Model. ‣ Predict Multiple Tasks. ‣ 4.3 Performance Analysis ‣ Result. ‣ Metrics. ‣ Setting. ‣ 4.2 Proactive Agent Evaluation ‣ Results. ‣ Metrics. ‣ 4.1 Reward Model Assessment ‣ 4 Experiments ‣ Proactive Agent: Shifting LLM Agents from Reactive Responses to Active Assistance")。

#### 事件生成

在具体的事件生成方面，我们从用户活动生成开始。对于每个场景，首先要求用户代理描述其在时间 $t$ 完成模拟环境中任务时的活动和行为 $A_{t}$。然后，健身房接受用户的活动和行为，并逐一生成详细事件。如图 [Figure 2](https://arxiv.org/html/2410.12361v3#S3.F2 "In 3.2 Pipeline Overview ‣ 3 Methodology ‣ Proactive Agent: Shifting LLM Agents from Reactive Responses to Active Assistance") 所示，健身房的任务是根据历史事件和当前的环境状态生成逻辑正确且流畅的事件。提高生成事件的现实性并适应不同环境的关键，是利用我们在场景生成过程中基于收集的事件所生成的示例事件。在生成事件之前，我们会为特定场景随机抽取生成的示例事件，并请求健身房根据这些事件生成新事件。一旦生成了新事件 $E_{t+1}$，健身房就会更新环境中实体的状态，并重复该过程，直到没有可以根据提供的用户活动生成的事件。这个综合方法确保了每个后续事件不仅是合适的，而且有助于场景中的连贯和逻辑进展。

#### 状态维护

健身房的另一个重要功能是维护环境的状态 $S_{t}$。在场景生成过程中，健身房会在模拟环境中创建像浏览器或开发工具包这样的实体，每个实体都有其状态和属性，如应用程序版本或特定的浏览器名称。当生成一个新事件时，健身房应该更新这些实体的状态和属性，以便为进一步的事件生成提供反馈。具体来说，我们首先收集相关实体的历史状态变化，并提示 GPT-4o 生成具有新事件的实体的新状态 $S_{t+1}$。在此过程中，模拟时间也将根据事件的粒度进行更新。之后，下一个事件将基于最新的环境状态 $S_{t+1}$ 生成。

### 3.4 前瞻性代理

![参见说明文字](img/eb79b2bb152e351544c3ac328e7ad72f.png)

图 3：前瞻性代理框架概述。该代理监控新事件并更新其记忆，以预测潜在任务。

数据生成管道中的第二个组件是主动代理，它预测用户可能分配的任务。正如[图 3](https://arxiv.org/html/2410.12361v3#S3.F3 "在3.4 主动代理 ‣ 3 方法论 ‣ 主动代理：将LLM代理从反应性响应转变为主动协助")中详细描述的那样，当代理在时间$t$接收到新的事件$E_{t}$时，它首先用该事件更新其记忆。为了提高预测质量，它还接受来自用户代理对其草拟预测的反馈。将新的事件与历史事件及与用户的对话结合，代理结合用户特征提出潜在任务。如果代理检测到潜在任务，它会将该任务作为一个新事件提出，并等待用户代理的判断。否则，主动代理预测没有潜在任务并保持沉默。一旦预测的任务被接受，代理将在仿真环境中执行该任务，仿真环境会生成多个关于代理与环境互动的事件。在数据生成过程中，代理会不断从仿真环境接收事件并预测潜在任务。

#### 任务执行

如前所述，一旦用户接受，主动代理会执行预测的任务。这个过程主要通过主动代理和仿真环境之间的多轮互动来完成。具体而言，主动代理将获得在场景生成过程中生成的工具，例如计算机中的文件系统工具或智能灯光开关的访问权限，以与仿真环境进行互动。每当主动代理采取一个行动时，仿真环境会生成一个新的事件，之后该事件会被仿真环境和用户代理进一步处理，以更新环境状态。之后，主动代理检测到新的环境状态$S_{t+1}$，并根据仿真环境生成的事件采取新的行动。此过程在用户中断或主动代理完成任务时结束。

### 3.5 用户代理

用户代理被设计用来模拟用户的活动 $A_{t}$ 和关于代理预测 $P_{t}$ 的响应，如 [图2](https://arxiv.org/html/2410.12361v3#S3.F2 "在 3.2 管道概述 ‣ 3 方法 ‣ 主动代理：将LLM代理从被动响应转变为主动协助") 中所示。我们提示 GPT-4o 在特定环境中为提供的任务生成活动和行动。健身环境进一步处理这些活动和行动，以生成新的事件。然后，主动代理根据这些事件预测潜在的任务。在收到预测的任务后，用户代理决定是否接受或拒绝该任务。如果用户代理接受任务，主动代理将在环境中设置并执行该任务。否则，如果用户代理拒绝建议的协助，环境将自主生成新的事件，而不进行任何干预。在我们的设置中，我们收集人工标注者的判断，并训练一个奖励模型来模拟这些判断。

具体来说，为了确保奖励模型与人类判断紧密对齐，我们生成并注释了一个专门的数据集，指示人类是否会接受预测的任务。我们使用 9 种不同的语言模型来为每个事件生成多样化的预测。在这些预测之间，我们选择总余弦距离最小的 5 个预测作为我们的标签目标。每个预测由三个独立的标注者用三种选项之一进行注释：接受、拒绝或完全拒绝。当标注者认为给定的事件并未暗示用户可能分配的任何任务时，即 $N_{t}=0$，选择“完全拒绝”选项。否则，如果一个预测被标记为接受，我们将事件 $E_{t}$ 标记为 $N_{t}=1$。我们使用多数投票法对每个预测做出最终决定。最终，注释结果生成了一个包含 $1,760$ 条数据的数据集，每条数据包含事件追踪、任务预测以及三名不同标注者对是否接受预测任务的决定。值得注意的是，我们的标注者在测试集上达到了超过 $91.67\%$ 的一致性率，凸显了注释结果的可靠性以及该数据集在进一步分析中的稳健性。为了进一步促进自动数据生成，我们还提示 GPT-4o 生成更详细的用户判断解释。有关奖励模型评估的更多细节，请参见 [4.1 奖励模型评估](https://arxiv.org/html/2410.12361v3#S4.SS1 "4.1 奖励模型评估 ‣ 4 实验 ‣ 主动代理：将LLM代理从被动响应转变为主动协助")。

## 4 实验

### 4.1 奖励模型评估

为了自动评估预测任务及其时机是否合适，我们旨在训练一个奖励模型，能够准确模仿用户的判断。为此，我们应用用户标注的数据来训练LLaMA-3.1-8B-Instruct [[12](https://arxiv.org/html/2410.12361v3#bib.bib12)]，并与多个基线进行比较，以展示其优越性。

#### 设置。

我们使用$1,760$条人工标注的条目，并随机将其划分为训练集（$1,640$条）和测试集（$120$条）。然后，我们在训练集上训练LLaMA-3.1-8B-Instruct，得到我们的奖励模型。我们使用总批次大小为$32$，学习率为$1e-5$，Adam优化器的预热比例为$0.01$。我们训练奖励模型$3$轮，以防止过拟合。我们使用8个A100 GPU在一个节点上进行训练，约需1.5小时。详细的提示模板列在[附录A](https://arxiv.org/html/2410.12361v3#A1 "附录A 奖励模型训练设置 ‣ 局限性 ‣ 5 结论 ‣ 4.4 案例研究 ‣ 来自奖励模型的反馈 ‣ 预测多个任务 ‣ 4.3 性能分析 ‣ 结果 ‣ 指标 ‣ 设置 ‣ 4.2 主动代理评估 ‣ 结果 ‣ 指标 ‣ 4.1 奖励模型评估 ‣ 4 实验 ‣ 主动代理: 将LLM代理从反应式响应转变为主动协助")中。我们使用测试集来评估我们调整后的模型及所有基线。值得注意的是，我们的人工标注者在测试集上达到了$91.67\%$的一致性比率，证明了我们评估的有效性。

#### 指标。

我们使用奖励模型对是否接受预测任务进行二分类，并将其结果与人工标注结果进行比较。这评估了奖励模型与人工判断在预测任务适宜性上的一致性。我们比较奖励模型和人工的判断，以计算召回率、精准度、准确率和F1分数。此外，我们还计算以下几种情况的一致性比率：

+   •

    错过-需要: $N_{t}=1,P_{t}=\emptyset$，用户需要帮助，但代理未提供帮助。

+   •

    无响应: $N_{t}=0,P_{t}=\emptyset$，用户不需要帮助，代理未提示。

+   •

    正确检测: $N_{t}=1,P_{t}\neq\emptyset$，且用户接受代理预测的任务。

+   •

    错误检测: $N_{t}=0,P_{t}\neq\emptyset$，用户不需要帮助，但代理提示。

{widetabular}

lccccc GPT-4o GPT-4o-mini LLaMA-3.1-8B LLaMA-3.1-70B 我们的方法

一致性。 MN^↑ 3.33% 56.67% 80.00% 33.33% 80.00%

一致性。 NR^↑ 100.00% 56.67% 30.00% 83.33% 86.67%

一致性。 CD^↑ 100.00% 86.67% 96.67% 100.00% 100.00%

一致性。 FD^↑ 0.00% 33.33% 13.33% 6.67% 100.00%

召回率^↑ 100.00% 71.67% 63.33% 91.67% 93.33%

精确度^↑ 50.42% 56.58% 54.29% 53.40% 90.32%

准确率^↑ 50.83% 58.33% 55.00% 55.83% 91.67%

F1分数^↑ 67.04% 63.24% 58.46% 67.48% 91.80%

表格 2：不同模型在我们的测试集上作为奖励模型的评估结果。对于错过需求（Missed-Need, MN）、未响应（Non-Response, NR）、正确检测（Correct-Detection, CD）和错误检测（False-Detection, FD）场景，我们展示了模型与我们人类注释员的主要投票之间的一致性比例。我们基于LLaMA-3.1-Instruct-8B微调的模型取得了最佳的F1-Score，达到了$91.80\%$。

#### 结果。

如[第4.1节](https://arxiv.org/html/2410.12361v3#S4.SS1.SSS0.Px2 "Metrics. ‣ 4.1 Reward Model Assessment ‣ 4 Experiments ‣ Proactive Agent: Shifting LLM Agents from Reactive Responses to Active Assistance")所示，所有现有模型在正确检测方面表现良好，但在其他场景中表现较差，特别是在假警报场景中。经过深入分析，我们发现现有模型无法推测用户可能需要什么，并且往往接受任意帮助，即使这种帮助对当前观察非常抽象或毫无意义。相比之下，我们的奖励模型在假警报场景中达到了$100\%$的一致性比例，并在所有场景中取得了$91.80\%$的F1-Score。我们选择我们的奖励模型进行进一步分析，涵盖ProactiveBench。

### 4.2 主动代理评估

{widetabular}

lcccccc 模型 召回率^↑ 精确率^↑ 准确率^↑ 假警报^↓ F1-Score^↑

专有模型

Claude-3-Sonnet 27.47% 37.31% 52.42% 62.69% 31.65%

Claude-3.5-Sonnet 97.89% 45.37% 49.78% 54.63% 62.00%

GPT-4o-mini 100.00% 35.28% 36.12% 64.73% 52.15%

GPT-4o 98.11% 48.15% 49.78% 51.85% 64.60%

开源模型

LLaMA-3.1-8B 98.86% 38.16% 39.06% 61.84% 55.06%

LLaMA-3.1-8B-Proactive 99.06% 49.76% 52.86% 50.24% 66.25%

Qwen2-7B 98.02% 44.00% 43.61% 56.00% 60.74%

Qwen2-7B-Proactive 100.00% 49.78% 50.66% 50.22% 66.47%

表格 3：不同模型在ProactiveBench上的表现评估结果。GPT-4o在封闭源模型中表现突出，F1-Score超过$64.60\%$。在开源模型中，我们微调的Qwen2-7B模型取得了最佳结果，F1-Score为$66.47\%$。

#### 设置。

我们使用ProactiveBench的训练集，基于两个开源模型：LLaMA-3.1-8B-Instruct和Qwen2-7B-Instruct，获取主动代理。在训练过程中，我们使用总批量大小为$32$，学习率为$1e-5$，Adam优化器和$0.01$的热启动比率。我们训练模型$3$个epoch。我们使用8个A100 GPU节点进行训练，大约训练2小时。详细的提示可以在[附录B](https://arxiv.org/html/2410.12361v3#A2 "附录B 代理模型训练设置 ‣ 局限性 ‣ 5 结论 ‣ 4.4 案例研究 ‣ 来自奖励模型的反馈 ‣ 预测多个任务 ‣ 4.3 性能分析 ‣ 结果 ‣ 指标 ‣ 设置 ‣ 4.2 主动代理评估 ‣ 结果 ‣ 指标 ‣ 4.1 奖励模型评估 ‣ 4 实验 ‣ 主动代理：将LLM代理从反应式响应转向主动协助")中找到。通过奖励模型自动评估这些指标。所有模型在我们的ProactiveBench测试集中进行评估，该测试集包含$233$个在现实世界中收集的事件。我们在所有模型中使用相同的提示模板。在测试过程中，温度设定为$0$。

#### 指标。

我们根据用户是否接受预测结果来评估主动代理的表现。如[第3.1节](https://arxiv.org/html/2410.12361v3#S3.SS1 "3.1 任务定义 ‣ 3 方法论 ‣ 主动代理：将LLM代理从反应式响应转向主动协助")中所述，用户的接受度$R_{t}$包含四个条件。在我们特定的设置中，召回率（Recall）衡量了代理正确预测的实际需要帮助的比例，包括代理预测任务并且用户接受的情况，以及代理没有预测任务并且用户不需要帮助的情况。精确度（Precision）衡量了用户实际接受的预测任务的比例。准确率（Accuracy）衡量了代理预测的整体正确性。误报率（False-Alarm）衡量了不正确的任务预测的比例，特别是当任务被预测但实际上不需要时。F1分数（F1-Score）提供了代理主动行为的平衡评估。在评估过程中，我们使用奖励模型自动生成用户的判断。基于混淆矩阵，我们报告了召回率、精确度、准确率、误报率和F1分数，涵盖所有设置。详细的计算方法可以在[附录B](https://arxiv.org/html/2410.12361v3#A2 "附录B 代理模型训练设置 ‣ 局限性 ‣ 5 结论 ‣ 4.4 案例研究 ‣ 来自奖励模型的反馈 ‣ 预测多个任务 ‣ 4.3 性能分析 ‣ 结果 ‣ 指标 ‣ 设置 ‣ 4.2 主动代理评估 ‣ 结果 ‣ 指标 ‣ 4.1 奖励模型评估 ‣ 4 实验 ‣ 主动代理：将LLM代理从反应式响应转向主动协助")中找到。

#### 结果。

[第4.2节](https://arxiv.org/html/2410.12361v3#S4.SS2 "4.2 Proactive Agent Evaluation ‣ Results. ‣ Metrics. ‣ 4.1 Reward Model Assessment ‣ 4 Experiments ‣ Proactive Agent: Shifting LLM Agents from Reactive Responses to Active Assistance") 比较了在ProactiveBench测试集上各种模型的表现，该测试集包含了来自真实世界用户的$233$个事件。像GPT-4o这样的闭源模型[[41](https://arxiv.org/html/2410.12361v3#bib.bib41)]或GPT-4o-mini都倾向于积极预测主动任务。它们中的大多数在用户需要时能够提供帮助，但在用户不需要任何帮助时却未能保持沉默，从而导致相对较高的误报率。例如，GPT-4o-mini即使在提供的事件没有包含有意义的操作（如仅在软件之间切换却什么也不做）时，仍提供不必要的帮助。另一个大问题是，在给定观察中无法找到明确的用户意图时，模型过早提供帮助。这使得模型提出的主动任务显得过于抽象或无用，导致相对较高的误报率。Claude-3-Sonnet [[42](https://arxiv.org/html/2410.12361v3#bib.bib42)] 则展示了一个不同的例子，未能识别用户的需求并提供无法满足用户期望的帮助。

对于开源模型，我们基于我们合成的数据，评估了LLaMA-3.1-Instruct-8B和Qwen2-Instruct-7B在微调前后的表现。如[第4.2节](https://arxiv.org/html/2410.12361v3#S4.SS2 "4.2 Proactive Agent Evaluation ‣ Results. ‣ Metrics. ‣ 4.1 Reward Model Assessment ‣ 4 Experiments ‣ Proactive Agent: Shifting LLM Agents from Reactive Responses to Active Assistance")所示，两个模型都取得了令人印象深刻的提升，尤其是LLaMA-3.1-8B，其F1分数从$44.78\%$提高到了$61.74\%$。结果展示了我们数据合成流程的有效性。至于主动代理过度打扰的问题，我们的微调模型在减少误报率方面取得了稳固进展，表现与GPT-4o相当。此外，微调后的Qwen2-7B在F1分数方面也超越了GPT-4o，达到了最高的$66.07\%$。然而，我们也观察到相同的模式，即模型倾向于尽可能提供帮助，而不是在用户需要时提供必要的帮助。

简而言之，尽管大多数模型在需要时可以提供帮助，但它们仍然经常提供不必要的帮助，即使被指示只提供必要的援助。

### 4.3 性能分析

在本节中，我们分析了可能影响主动代理性能的两种设置类型。

#### 预测多个任务。

在实际应用中，主动代理可以提供多个候选任务以提升整体表现。为了评估模型在此条件下的表现，我们允许它们一次生成多个候选任务，但不超过三个，以避免给用户带来过高的认知负担。在此设置中，我们让奖励模型逐一检查候选任务。如果其中一个候选任务被接受，则标记为接受；如果所有候选任务都被拒绝，则标记为拒绝。

如[第4.3节](https://arxiv.org/html/2410.12361v3#S4.SS3.SSS0.Px1 "预测多个任务。 ‣ 4.3 性能分析 ‣ 结果。 ‣ 指标。 ‣ 设置。 ‣ 4.2 主动代理评估 ‣ 结果。 ‣ 指标。 ‣ 4.1 奖励模型评估 ‣ 4 实验 ‣ 主动代理：将LLM代理从被动响应转向主动帮助")所示，在将“pred@1”与“pred@3”进行比较时，所有模型在所有指标上都有显著的改善。以GPT-4o为例，它在提高准确率和精度的同时，通过提供多样化的候选任务减少了虚警率。虚警率从$51.85\%$降到$36.44\%$的巨大下降，主要得益于其在提供主动任务方面的改进。然而，在将GPT-4o-mini与LLaMA-3.1-8B进行比较时，我们观察到不同程度的改善。当一次只预测一个主动任务时，这两个模型的表现相似，但在同时预测多个候选任务时，F1得分有接近$9\%$的差异。我们分析了结果，发现LLaMA-3.1-8B在用户需求不明确时倾向于提供意外的帮助，而这种情况无法通过提供多个候选任务来改善。

{widetabular}

lcccccc 模型设置 召回率^↑ 精度^↑ 准确率^↑ 虚警率^↓ F1-得分^↑

GPT-4o-mini pred@1 100.00% 35.28% 36.12% 64.73% 52.15%

pred@3 99.32% 65.32% 66.52% 34.68% 78.80%

w/ RM 55.45% 63.54% 63.95% 36.46% 59.22%

pred@3, w/ RM 100.00% 65.35% 66.09% 34.65% 79.05%

GPT-4o pred@1 98.11% 48.15% 49.78% 51.85% 64.60%

pred@3 100.00% 63.56% 64.81% 36.44% 77.72%

w/ RM 56.76% 55.26% 57.61% 44.74% 56.00%

pred@3, w/ RM 100.00% 63.30% 65.67% 36.70% 77.53%

LLaMA-3.1-8B pred@1 98.86% 38.16% 39.06% 61.84% 55.06%

pred@3 100.00% 52.79% 52.79% 47.21% 69.10%

w/ RM 77.08% 42.52% 47.64% 57.41% 54.81%

pred@3, w/ RM 95.12% 61.58% 66.09% 38.42% 74.76%

表4：不同设置下各模型的对比。设置“pred@1”表示一次预测一个任务。设置“pred@3”表示一次预测三个任务。设置“w/ RM”表示我们将提供来自奖励模型的反馈以帮助更好的预测。

#### 来自奖励模型的反馈。

我们还研究了我们的奖励模型的反馈是否能够帮助模型提高在ProactiveBench上的表现。这一研究逻辑与[图3](https://arxiv.org/html/2410.12361v3#S3.F3 "在3.4 主动代理 ‣ 3 方法论 ‣ 主动代理：将LLM代理从反应性响应转向主动协助")中描述的相同。对于每个模型，我们首先要求它们生成一个初步预测，并从用户代理（在此案例中基于奖励模型构建）中获取反馈。然后，我们让模型完善其预测，以得到最终预测。

如在[第4.3节](https://arxiv.org/html/2410.12361v3#S4.SS3.SSS0.Px1 "预测多个任务。 ‣ 4.3 性能分析 ‣ 结果。 ‣ 指标。 ‣ 设置。 ‣ 4.2 主动代理评估 ‣ 结果。 ‣ 指标。 ‣ 4.1 奖励模型评估 ‣ 4 实验 ‣ 主动代理：将LLM代理从反应性响应转向主动协助")中所示，通过加入奖励模型的反馈（“带奖励模型设置”），模型通常能降低误报率并提高准确性，但召回率会急剧下降。我们观察到模型在收到奖励模型的反馈后会保持沉默。然而，什么都不做并不总是最优解。GPT-4o似乎未能捕捉到可能的用户需求，导致F1-Score下降。对于其他模型，如GPT-4o-mini和LLaMA-3.1-8B，它们在F1-Score上确实取得了显著的提升。另一种将多任务预测与奖励模型结合的设置（“pred@3, w/ RM”）在各方面表现出更为普遍的改进。通过将奖励模型集成到主动代理中，我们可以使主动代理更智能地检测用户需求，即使我们无法直接访问权重，这对开发主动代理来说是一个好消息。

### 4.4 案例研究

本节中，我们探讨了预测可能任务时遇到的两种常见失败类型：无法检测用户需求和在不适当的时机做出预测。

![参考说明](img/d9ebeec27b911c54b77cadc0cfdd8aba.png)

图4：两种类型的失败：未能检测到用户需求（左）和不恰当的提议时间（右）。我们比较了我们微调后的LLaMA-3.1-Instruct-8B与其他模型的响应，展示了模型的细化主动行为。

如 [图 4](https://arxiv.org/html/2410.12361v3#S4.F4 "在 4.4 案例研究 ‣ 来自奖励模型的反馈。 ‣ 预测多个任务。 ‣ 4.3 性能分析 ‣ 结果。 ‣ 指标。 ‣ 设置。 ‣ 4.2 主动代理评估 ‣ 结果。 ‣ 指标。 ‣ 4.1 奖励模型评估 ‣ 4 实验 ‣ 主动代理：将 LLM 代理从反应式响应转变为主动协助")（左）所示，当GPT-4o模型在关键时刻未能提供帮助时，出现了显著的失败。例如，当用户正在整合支付API并需要教程指导时，模型保持沉默。相反，我们的模型成功地检测到人类需求并提供了帮助。旨在减少干扰的根本意图，讽刺性地导致错失了提供及时帮助的机会。

相反， [图 4](https://arxiv.org/html/2410.12361v3#S4.F4 "在 4.4 案例研究 ‣ 来自奖励模型的反馈。 ‣ 预测多个任务。 ‣ 4.3 性能分析 ‣ 结果。 ‣ 指标。 ‣ 设置。 ‣ 4.2 主动代理评估 ‣ 结果。 ‣ 指标。 ‣ 4.1 奖励模型评估 ‣ 4 实验 ‣ 主动代理：将 LLM 代理从反应式响应转变为主动协助") 的右侧展示了一个时机不当的预测实例。在这里，GPT-4o-mini 在没有用户需求的情况下建议采取某个行动。这个场景突出了人类活动中可能存在的意外事件。模型应该智能判断是否存在可能的任务，以避免不必要的行动。这些实例凸显了人类活动的复杂性，以及模型准确预测人类需求所需的复杂推理。为了在提供帮助和干扰之间取得微妙的平衡，模型必须深入理解用户的上下文和活动，确保它们的干预既及时又相关。

## 5 结论

我们提出了一种通过利用主动任务预测来预测人类需求的创新方法，从而改善人类与代理之间的互动。我们引入了ProactiveBench，一个包含$6,790$个事件的综合数据集，旨在细化基于LLM的代理的主动行为，并建立一个自动基准来评估模型的主动性。通过在合成场景中反复生成事件，我们创建了训练数据，以增强我们模型的主动能力。我们的实验表明，代理在ProactiveBench上的表现有了显著提高，验证了我们方法的有效性。尽管取得了这些进展，我们的研究结果仍然突出了持续存在的挑战，特别是在最小化不恰当的任务建议和确保任务预测具有上下文准确性方面。未来的研究应集中在提高任务预测的精确性和及时性，以提升主动人机互动的有效性。

## 限制

尽管我们的方法证明了它可以有效且主动地预测可能的任务，但当前的研究仍受到一些限制。首先，我们探索的环境设置仍然有限。本文中的背景提供了基础的理解，但需要研究更广泛的应用领域，以全面确立主动代理的多功能性和鲁棒性。此外，模型仍然表现出较高的误报率，表明它们尚不能完美预测可能的任务。这个限制突显了进一步完善模型主动行为的必要性，以避免打扰用户。较高的误报率可能导致不必要或不正确的操作，从而降低用户信任度和系统的整体效率。应该深入探讨根据具体背景动态调整代理的主动性。未来的研究应集中在几个关键领域，以解决这些限制：

+   •

    环境设置的扩展：研究应探索更多的场景和环境，以验证模型的普适性。这包括那些通过主动预测任务可以显著提升用户体验和操作效率的领域。

+   •

    预测准确性的改进：应致力于通过增强模型对上下文和用户行为的理解来减少误报率。

+   •

    以用户为中心的评估：未来的研究应进行广泛的以用户为中心的评估，更好地了解用户如何与主动代理互动，并识别改进的方向。用户反馈和行为数据可以为优化预测算法并使系统更直观可靠提供宝贵的见解。

+   •

    伦理和隐私考虑：由于主动代理需要环境信息来预测任务，因此必须解决伦理和隐私问题。确保用户数据得到负责任的处理，并且代理在伦理准则内透明运作，将是获得用户信任和接受度的关键。

## 参考文献

+   [1] OpenAI. OpenAI: Introducing ChatGPT, 2022.

+   [2] Weize Chen, Yusheng Su, Jingwei Zuo, Cheng Yang, Chenfei Yuan, Chi-Min Chan, Heyang Yu, Yaxi Lu, Yi-Hsin Hung, Chen Qian, Yujia Qin, Xin Cong, Ruobing Xie, Zhiyuan Liu, Maosong Sun, and Jie Zhou. Agentverse: Facilitating multi-agent collaboration and exploring emergent behaviors, 2023.

+   [3] Wenyi Hong, Weihan Wang, Qingsong Lv, Jiazheng Xu, Wenmeng Yu, Junhui Ji, Yan Wang, Zihan Wang, Yuxiao Dong, Ming Ding, and Jie Tang. Cogagent: A visual language model for gui agents. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR), pages 14281–14290, June 2024.

+   [4] Chi Zhang, Zhao Yang, Jiaxuan Liu, Yucheng Han, Xin Chen, Zebiao Huang, Bin Fu, and Gang Yu. Appagent: Multimodal agents as smartphone users, 2023.

+   [5] Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Beibin Li, Erkang Zhu, Li Jiang, Xiaoyun Zhang, Shaokun Zhang, Jiale Liu, Ahmed Hassan Awadallah, Ryen W White, Doug Burger, 和 Chi Wang. Autogen: 通过多智能体对话框架使下一代大语言模型应用成为可能. 2023年.

+   [6] Guohao Li, Hasan Hammoud, Hani Itani, Dmitrii Khizbullin, 和 Bernard Ghanem. CAMEL: 用于大语言模型社会“心智”探索的交互智能体. 在 Alice Oh, Tristan Naumann, Amir Globerson, Kate Saenko, Moritz Hardt, 和 Sergey Levine 编辑的《神经信息处理系统进展 36：2023年神经信息处理系统年会, NeurIPS 2023, 新奥尔良，美国，2023年12月10日至16日》一书中，2023年.

+   [7] Roya Firoozi, Johnathan Tucker, Stephen Tian, Anirudha Majumdar, Jiankai Sun, Weiyu Liu, Yuke Zhu, Shuran Song, Ashish Kapoor, Karol Hausman, Brian Ichter, Danny Driess, Jiajun Wu, Cewu Lu, 和 Mac Schwager. 机器人学中的基础模型：应用、挑战与未来. 《国际机器人研究杂志》，0(0):02783649241281508, 0.

+   [8] Yuanchun Li, Hao Wen, Weijun Wang, Xiangyu Li, Yizhen Yuan, Guohong Liu, Jiacheng Liu, Wenxing Xu, Xiang Wang, Yi Sun 等. 个人大语言模型智能体：关于能力、效率和安全性的洞察与调查. ArXiv预印本, abs/2401.05459, 2024年.

+   [9] Yining Ye, Xin Cong, Shizuo Tian, Jiannan Cao, Hao Wang, Yujia Qin, Yaxi Lu, Heyang Yu, Huadong Wang, Yankai Lin, Zhiyuan Liu, 和 Maosong Sun. Proagent: 从机器人过程自动化到智能过程自动化, 2023年.

+   [10] Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll L. Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray, John Schulman, Jacob Hilton, Fraser Kelton, Luke Miller, Maddie Simens, Amanda Askell, Peter Welinder, Paul F. Christiano, Jan Leike, 和 Ryan Lowe. 训练语言模型以通过人类反馈遵循指令. 在 Sanmi Koyejo, S. Mohamed, A. Agarwal, Danielle Belgrave, K. Cho, 和 A. Oh 编辑的《神经信息处理系统进展 35：2022年神经信息处理系统年会, NeurIPS 2022, 新奥尔良，美国，2022年11月28日至12月9日》一书中，2022年.

+   [11] B.N. Schilit 和 M.M. Theimer. 向移动主机传播活动地图信息. IEEE Network, 8(5):22–32, 1994年.

+   [12] Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar, Aurelien Rodriguez, Armand Joulin, Edouard Grave, 和 Guillaume Lample. Llama: 开放和高效的基础语言模型, 2023年.

+   [13] Jinze Bai, Shuai Bai, Yunfei Chu, Zeyu Cui, Kai Dang, Xiaodong Deng, Yang Fan, Wenbin Ge, Yu Han, Fei Huang, Binyuan Hui, Luo Ji, Mei Li, Junyang Lin, Runji Lin, Dayiheng Liu, Gao Liu, Chengqiang Lu, Keming Lu, Jianxin Ma, Rui Men, Xingzhang Ren, Xuancheng Ren, Chuanqi Tan, Sinan Tan, Jianhong Tu, Peng Wang, Shijie Wang, Wei Wang, Shengguang Wu, Benfeng Xu, Jin Xu, An Yang, Hao Yang, Jian Yang, Shusheng Yang, Yang Yao, Bowen Yu, Hongyi Yuan, Zheng Yuan, Jianwei Zhang, Xingxuan Zhang, Yichang Zhang, Zhenru Zhang, Chang Zhou, Jingren Zhou, Xiaohuan Zhou, 和 Tianhang Zhu. Qwen 技术报告。ArXiv 预印本，abs/2309.16609，2023。

+   [14] OpenAI, :, Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat, Red Avila, Igor Babuschkin, Suchir Balaji, Valerie Balcom, Paul Baltescu, Haiming Bao, Mo Bavarian, Jeff Belgum, Irwan Bello, Jake Berdine, Gabriel Bernadett-Shapiro, Christopher Berner, Lenny Bogdonoff, Oleg Boiko, Madelaine Boyd, Anna-Luisa Brakman, Greg Brockman, Tim Brooks, Miles Brundage, Kevin Button, Trevor Cai, Rosie Campbell, Andrew Cann, Brittany Carey, Chelsea Carlson, Rory Carmichael, Brooke Chan, Che Chang, Fotis Chantzis, Derek Chen, Sully Chen, Ruby Chen, Jason Chen, Mark Chen, Ben Chess, Chester Cho, Casey Chu, Hyung Won Chung, Dave Cummings, Jeremiah Currier, Yunxing Dai, Cory Decareaux, Thomas Degry, Noah Deutsch, Damien Deville, Arka Dhar, David Dohan, Steve Dowling, Sheila Dunning, Adrien Ecoffet, Atty Eleti, Tyna Eloundou, David Farhi, Liam Fedus, Niko Felix, Simón Posada Fishman, Juston Forte, Isabella Fulford, Leo Gao, Elie Georges, Christian Gibson, Vik Goel, Tarun Gogineni, Gabriel Goh, Rapha Gontijo-Lopes, Jonathan Gordon, Morgan Grafstein, Scott Gray, Ryan Greene, Joshua Gross, Shixiang Shane Gu, Yufei Guo, Chris Hallacy, Jesse Han, Jeff Harris, Yuchen He, Mike Heaton, Johannes Heidecke, Chris Hesse, Alan Hickey, Wade Hickey, Peter Hoeschele, Brandon Houghton, Kenny Hsu, Shengli Hu, Xin Hu, Joost Huizinga, Shantanu Jain, Shawn Jain, Joanne Jang, Angela Jiang, Roger Jiang, Haozhun Jin, Denny Jin, Shino Jomoto, Billie Jonn, Heewoo Jun, Tomer Kaftan, Łukasz Kaiser, Ali Kamali, Ingmar Kanitscheider, Nitish Shirish Keskar, Tabarak Khan, Logan Kilpatrick, Jong Wook Kim, Christina Kim, Yongjik Kim, Hendrik Kirchner, Jamie Kiros, Matt Knight, Daniel Kokotajlo, Łukasz Kondraciuk, Andrew Kondrich, Aris Konstantinidis, Kyle Kosic, Gretchen Krueger, Vishal Kuo, Michael Lampe, Ikai Lan, Teddy Lee, Jan Leike, Jade Leung, Daniel Levy, Chak Ming Li, Rachel Lim, Molly Lin, Stephanie Lin, Mateusz Litwin, Theresa Lopez, Ryan Lowe, Patricia Lue, Anna Makanju, Kim Malfacini, Sam Manning, Todor Markov, Yaniv Markovski, Bianca Martin, Katie Mayer, Andrew Mayne, Bob McGrew, Scott Mayer McKinney, Christine McLeavey, Paul McMillan, Jake McNeil, David Medina, Aalok Mehta, Jacob Menick, Luke Metz, Andrey Mishchenko, Pamela Mishkin, Vinnie Monaco, Evan Morikawa, Daniel Mossing, Tong Mu, Mira Murati, Oleg Murk, David Mély, Ashvin Nair, Reiichiro Nakano, Rajeev Nayak, Arvind Neelakantan, Richard Ngo, Hyeonwoo Noh, Long Ouyang, Cullen O’Keefe, Jakub Pachocki, Alex Paino, Joe Palermo, Ashley Pantuliano, Giambattista Parascandolo, Joel Parish, Emy Parparita, Alex Passos, Mikhail Pavlov, Andrew Peng, Adam Perelman, Filipe de Avila Belbute Peres, Michael Petrov, Henrique Ponde de Oliveira Pinto, Michael, Pokorny, Michelle Pokrass, Vitchyr Pong, Tolly Powell, Alethea Power, Boris Power, Elizabeth Proehl, Raul Puri, Alec Radford, Jack Rae, Aditya Ramesh, Cameron Raymond, Francis Real, Kendra Rimbach, Carl Ross, Bob Rotsted, Henri Roussez, Nick Ryder, Mario Saltarelli, Ted Sanders, Shibani Santurkar, Girish Sastry, Heather Schmidt, David Schnurr, John Schulman, Daniel Selsam, Kyla Sheppard, Toki Sherbakov, Jessica Shieh, Sarah Shoker, Pranav Shyam, Szymon Sidor, Eric Sigler, Maddie Simens, Jordan Sitkin, Katarina Slama, Ian Sohl, Benjamin Sokolowsky, Yang Song, Natalie Staudacher, Felipe Petroski Such, Natalie Summers, Ilya Sutskever, Jie Tang, Nikolas Tezak, Madeleine Thompson, Phil Tillet, Amin Tootoonchian, Elizabeth Tseng, Preston Tuggle, Nick Turley, Jerry Tworek, Juan Felipe Cerón Uribe, Andrea Vallone, Arun Vijayvergiya, Chelsea Voss, Carroll Wainwright, Justin Jay Wang, Alvin Wang, Ben Wang, Jonathan Ward, Jason Wei, CJ Weinmann, Akila Welihinda, Peter Welinder, Jiayi Weng, Lilian Weng, Matt Wiethoff, Dave Willner, Clemens Winter, Samuel Wolrich, Hannah Wong, Lauren Workman, Sherwin Wu, Jeff Wu, Michael Wu, Kai Xiao, Tao Xu, Sarah Yoo, Kevin Yu, Qiming Yuan, Wojciech Zaremba, Rowan Zellers, Chong Zhang, Marvin Zhang, Shengjia Zhao, Tianhao Zheng, Juntang Zhuang, William Zhuk, 和 Barret Zoph。GPT-4技术报告。技术报告，2023年。

+   [15] Aakanksha Chowdhery, Sharan Narang, Jacob Devlin, Maarten Bosma, Gaurav Mishra, Adam Roberts, Paul Barham, Hyung Won Chung, Charles Sutton, Sebastian Gehrmann 等人。Palm：通过路径扩展语言建模。ArXiv 预印本，abs/2204.02311，2022年。

+   [16] Aohan Zeng, Xiao Liu, Zhengxiao Du, Zihan Wang, Hanyu Lai, Ming Ding, Zhuoyi Yang, Yifan Xu, Wendi Zheng, Xiao Xia, Weng Lam Tam, Zixuan Ma, Yufei Xue, Jidong Zhai, Wenguang Chen, Zhiyuan Liu, Peng Zhang, Yuxiao Dong, 和 Jie Tang。GLM-130B：一个开放的双语预训练模型。在第十一届国际学习表征大会（ICLR 2023）中，2023年5月1日至5日，卢旺达基加利。OpenReview.net，2023年。

+   [17] Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, Fei Xia, Ed H. Chi, Quoc V. Le, 和 Denny Zhou。思维链提示在大型语言模型中引发推理。在 Sanmi Koyejo、S. Mohamed、A. Agarwal、Danielle Belgrave、K. Cho 和 A. Oh 编辑的《神经信息处理系统进展 35：神经信息处理系统年会 2022》一书中，NeurIPS 2022，2022年11月28日至12月9日，美国路易斯安那州新奥尔良市，2022年。

+   [18] Luyu Gao, Aman Madaan, Shuyan Zhou, Uri Alon, Pengfei Liu, Yiming Yang, Jamie Callan, 和 Graham Neubig。PAL：程序辅助语言模型。在 Andreas Krause、Emma Brunskill、Kyunghyun Cho、Barbara Engelhardt、Sivan Sabato 和 Jonathan Scarlett 编辑的《国际机器学习会议论文集》中，ICML 2023，2023年7月23日至29日，美国夏威夷州檀香山，第202卷，机器学习研究会议论文集，10764-10799页。PMLR，2023年。

+   [19] Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R. Narasimhan, 和 Yuan Cao。React：在语言模型中协同推理与行动。在第十一届国际学习表征大会（ICLR 2023）中，2023年5月1日至5日，卢旺达基加利。OpenReview.net，2023年。

+   [20] Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Tom Griffiths, Yuan Cao, 和 Karthik Narasimhan。思想树：使用大型语言模型进行深思熟虑的问题解决。在 Alice Oh、Tristan Naumann、Amir Globerson、Kate Saenko、Moritz Hardt 和 Sergey Levine 编辑的《神经信息处理系统进展 36：神经信息处理系统年会 2023》一书中，NeurIPS 2023，2023年12月10日至16日，美国路易斯安那州新奥尔良市，2023年。

+   [21] Bo Liu, Yuqian Jiang, Xiaohan Zhang, Qiang Liu, Shiqi Zhang, Joydeep Biswas, 和 Peter Stone。LLM+ P：赋能大型语言模型以实现最优规划能力。ArXiv 预印本，abs/2304.11477，2023年。

+   [22] Yining Ye, Xin Cong, Yujia Qin, Yankai Lin, Zhiyuan Liu, 和 Maosong Sun。大型语言模型作为自主决策者。ArXiv 预印本，abs/2308.12519，2023年。

+   [23] Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Eric Hambro, Luke Zettlemoyer, Nicola Cancedda, 和 Thomas Scialom. Toolformer：语言模型可以自我学习使用工具. 载于Alice Oh, Tristan Naumann, Amir Globerson, Kate Saenko, Moritz Hardt, 和 Sergey Levine主编，《神经信息处理系统进展 36：神经信息处理系统年度会议 2023》论文集，NeurIPS 2023，2023年12月10-16日，美国新奥尔良，2023年。

+   [24] Yujia Qin, Shengding Hu, Yankai Lin, Weize Chen, Ning Ding, Ganqu Cui, Zheni Zeng, Yufei Huang, Chaojun Xiao, Chi Han, 等. 基础模型的工具学习. ArXiv预印本，abs/2304.08354，2023年。

+   [25] Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, Sihan Zhao, Runchu Tian, Ruobing Xie, Jie Zhou, Mark Gerstein, Dahai Li, Zhiyuan Liu, 和 Maosong Sun. Toolllm：帮助大语言模型掌握16000+个真实世界API，2023年。

+   [26] Cheng Qian, Chenyan Xiong, Zhenghao Liu, 和 Zhiyuan Liu. Toolink：通过开放源代码模型的链式求解连接工具包创建和使用. 载于Kevin Duh, Helena Gomez, 和 Steven Bethard主编，《2024年北美计算语言学协会年会论文集：人类语言技术》第一卷，第831–854页，墨西哥墨西哥城，2024年。计算语言学协会。

+   [27] Yujia Qin, Zihan Cai, Dian Jin, Lan Yan, Shihao Liang, Kunlun Zhu, Yankai Lin, Xu Han, Ning Ding, Huadong Wang, Ruobing Xie, Fanchao Qi, Zhiyuan Liu, Maosong Sun, 和 Jie Zhou. WebCPM：面向中文长篇问答的互动网页搜索. 载于Anna Rogers, Jordan Boyd-Graber, 和 Naoaki Okazaki主编，《第61届计算语言学协会年会论文集》第一卷，第8968–8988页，加拿大多伦多，2023年。计算语言学协会。

+   [28] Chen Qian, Xin Cong, Wei Liu, Cheng Yang, Weize Chen, Yusheng Su, Yufan Dang, Jiahao Li, Juyuan Xu, Dahai Li, Zhiyuan Liu, 和 Maosong Sun. 面向软件开发的交流型代理，2023年。

+   [29] Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang, 和 Michael S Bernstein. 生成型代理：人类行为的互动模拟体. 载于第36届ACM用户界面软件与技术年会论文集，第1–22页，2023年。

+   [30] Xuan Zhang, Yang Deng, Zifeng Ren, See-Kiong Ng, 和 Tat-Seng Chua. Ask-before-plan：面向现实世界规划的主动语言代理. arXiv预印本arXiv:2406.12639，2024年。

+   [31] 吴政光, 谭志锐, 吴兆忠, 林介彥, 李弘毅, 陈云农. 我需要帮助！评估大语言模型在请求用户支持方面的能力：基于文本到SQL生成的案例研究. 收录于Yaser Al-Onaizan, Mohit Bansal, Yun-Nung Chen编《2024年自然语言处理经验方法会议论文集》，第2191–2199页，美国佛罗里达州迈阿密，2024年11月，计算语言学协会。

+   [32] 吴卓浩, 纪丹文, 于凯文, 曾显旭, 吴定铭, 莎米赫·希杜贾曼. AI 创造力与人类-人工智能共同创造模型. 收录于黑泽雅明编《人机交互：理论、方法与工具》一书，第171–190页，2021年，施普林格国际出版社。

+   [33] 陈伟文, 莎米赫·希杜贾曼, 金江波, 阿赫梅德·萨拉赫·乌丁. 创造互动艺术的人工智能方法论. 收录于Constantine Stephanidis, Don Harris, Wen-Chin Li, Dylan D. Schmorrow, Cali M. Fidopiastis, Panayiotis Zaphiris, Andri Ioannou, Xiaowen Fang, Robert A. Sottilare, Jessica Schwarz编《HCI国际会议2020—后期论文：认知、学习与游戏》一书，第13–31页，2020年，施普林格国际出版社。

+   [34] 克里斯蒂娜·维特霍夫, 纳维德·塔瓦纳普尔, 艾娃·艾丽丝·克里斯蒂安娜·比特纳. 实施智能协作代理作为协作文写中的队友：走向人类与AI的协同. 收录于夏威夷国际系统科学会议，2021年。

+   [35] 钱程, 何炳翔, 庄中, 邓佳, 秦煜佳, 宋鑫, 林彦凯, 张忠, 刘志远, 孙茂松. 告诉我更多！面向隐性用户意图理解的语言模型驱动代理. ArXiv预印本，abs/2402.09205，2024年。

+   [36] 杨登, 雷文强, 林伟, 蔡达生. 主动对话系统的调查：问题、方法和前景. 收录于《第32届国际人工智能联合会议论文集》，第6583–6591页，2023年。

+   [37] 毕可平, 艾青尧, 克劳德·布鲁斯·克罗夫特. 基于负反馈提出澄清问题的对话搜索. 收录于《2021年ACM SIGIR信息检索理论国际会议论文集》，第157–166页，2021年。

+   [38] 任旭辉, 尹洪智, 陈通, 王浩, 黄子, 郑凯. 在对话推荐中学习提出合适的问题. 收录于《第44届国际ACM SIGIR信息检索研究与开发会议论文集》，第808–817页，2021年。

+   [39] 李子轩, 廖丽姿. 蔡达生. 学会提出关键问题以协助产品搜索（2022年）. 收录于《2022年ACM SIGIR电子商务研讨会论文集》，西班牙马德里，2022年7月，第15卷。

+   [40] 刘天健, 赵宏征, 刘宇恒, 王兴博, 彭振辉. Compeer：一种生成式对话代理，用于主动的同行支持. 收录于《第37届年度ACM用户界面软件与技术研讨会论文集》，第1–22页，2024年。

+   [41] OpenAI. 你好，gpt-4o，2024年。访问日期：2024-06-16。

+   [42] Anthropic. 介绍 claude-3 系列，2024年。访问日期：2024-06-16。

## 附录

## 附录 A 奖励模型训练设置

我们使用 Llama-3.1-Instruct-8B 作为我们训练的基础模型。总数据集大小约为 $1,640$。具体而言，我们采用总批量大小为 $32$，学习率为 $1e-5$，以及 $0.01$ 的 Adam 优化器预热比例。我们训练奖励模型 3 个周期，以防止其过拟合。我们在一个节点上使用 8 个 A100 GPU 训练，大约需要 1.5 小时。

提示模板

[⬇](data:text/plain;base64,PFRhc2s+CkV2YWx1YXRlIHRoZSB0YXNrIHByb3Bvc2VkIGJ5IHRoZSBwcm9hY3RpdmUgYXNzaXN0YW50IGFzIHRoZSB1c2VyLgo8L1Rhc2s+Cgo8UnVsZT4KMC4gQW5hbHl6ZSB0aGUgY3VycmVudCBvYnNlcnZhdGlvbiB0byB1bmRlcnN0YW5kIHlvdXIgY3VycmVudCBzaXR1YXRpb24gYW5kIHJlcXVpcmVtZW50cy4KMS4gSWYgdGhlIHByb3Bvc2VkIHRhc2sgaXMgYG51bGxgIChpbmRpY2F0aW5nIG5vIHRhc2sgaXMgcHJvcG9zZWQgdW5kZXIgdGhlIGN1cnJlbnQgb2JzZXJ2YXRpb24pLCBmb2xsb3cgdGhlc2Ugc3RlcHM6CiAgIC0gQWNjZXB0IHRoZSBgbnVsbGAgdGFzayBpZiB5b3UgYmVsaWV2ZSB0aGVyZSBpcyBubyBuZWVkIGZvciBhIHRhc2suCiAgIC0gUmVqZWN0IHRoZSBgbnVsbGAgdGFzayBpZiB5b3UgYmVsaWV2ZSBhIHRhc2sgaXMgbmVlZGVkLgoyLiBNaW5pbWl6ZSBpbnRlcnJ1cHRpb25zIGZyb20gdGhlIGFzc2lzdGFudCBieSBvbmx5IGFjY2VwdGluZyB0YXNrcyB0aGF0IGFyZSB2YWx1YWJsZS4KMy4gRXZhbHVhdGUgdGhlIGN1cnJlbnQgb2JzZXJ2YXRpb24gYW5kIG1ha2UgYSBqdWRnbWVudCBvbiB0aGUgcHJvcG9zZWQgdGFzayBhY2NvcmRpbmdseS4KPC9SdWxlPgoKPEZvcm1hdD4KWW91IHNob3VsZCBhbnN3ZXIgd2l0aCB0aGUgZm9sbG93aW5nIEpTT04gZm9ybWF0Ogp7CiAgICAidGhvdWdodCI6ICJHaXZlIHlvdXIgdGhvdWdodHMgZmlyc3QsIHRoZW4gcHJvdmlkZSB0aGUganVkZ21lbnQgb2YgdGhlIHRhc2suIiwKICAgICJqdWRnbWVudCI6ICJhY2NlcHRlZCBvciByZWplY3RlZCIKfQo8L0Zvcm1hdD4=)<任务>评估主动助手提出的任务，作为用户进行判断。</任务><规则>0. 解析当前观察结果，理解你当前的情况和需求。1. 如果提出的任务是‘null’（表示当前观察结果下没有提出任务），请按照以下步骤操作：- 如果你认为不需要任务，则接受‘null’任务。- 如果你认为需要任务，则拒绝‘null’任务。2. 通过只接受有价值的任务来减少助手的干扰。3. 评估当前观察结果，并根据提出的任务做出判断。</规则><格式>你应该用以下 JSON 格式回答：{"思考": "先给出你的想法，再做出对任务的判断。","判断": "接受或拒绝"}</格式>

## 附录 B 代理模型训练设置

类似地，我们使用 Llama-3-Instruct 8B 和 Qwen2-Instruct-7B 作为代理模型训练的基础模型。总数据集大小约为 $6,790$。具体来说，我们使用的批量大小为 $32$，学习率为 $1e-5$，并采用 Adam 优化器，warm-up 比例为 $0.01$。我们训练模型 3 个 epoch，以防止过拟合。我们在一个节点上使用 8 个 A100 GPU 进行训练，训练时长大约为 2 小时。

#### 模板。

我们应用以下提示模板来训练代理模型：

提示模板

[⬇](data:text/plain;base64,PFJvbGU+IFlvdSBhcmUgYSBoZWxwZnVsIGFzc2lzdGFudCB0aGF0IHByb3ZpZGVzIHByb2FjdGl2ZSBzdWdnZXN0aW9ucyB0byB0aGUgdXNlci4gPC9Sb2xlPgoKPFRhc2s+IFVuZGVyc3RhbmQgd2hhdCB0aGUgdXNlciBpcyBkb2luZyBhbmQgYW50aWNpcGF0ZSB0aGVpciBuZWVkcyBiYXNlZCBvbiBldmVudHMuIE9ubHkgcHJvcG9zZSBhc3Npc3RhbmNlIHdoZW4geW91IGZ1bGx5IHVuZGVyc3RhbmQgdGhlIHVzZXIncyBhY3Rpb25zLiBVc2UgYXZhaWxhYmxlIG9wZXJhdGlvbnMgdG8gZW5zdXJlIHRoZSB0YXNrIGlzIGZlYXNpYmxlLiBFeGVjdXRlIHRoZSB0YXNrIGlmIHRoZSB1c2VyIGFjY2VwdHMgeW91ciBwcm9wb3NhbC4gPC9UYXNrPgoKPEZvcm1hdD4gUmVzcG9uZCBpbiB0aGUgZm9sbG93aW5nIEpTT04gZm9ybWF0Ogp7CiAgICAiUHVycG9zZSI6ICJUaGUgcHVycG9zZSBvZiB0aGUgdXNlcidzIGxhc3QgYWN0aW9uLiIsCiAgICAiVGhvdWdodHMiOiAiWW91ciB0aG91Z2h0cyBvbiB0aGUgdXNlcidzIGFjdGlvbnMuIiwKICAgICJQcm9hY3RpdmUgVGFzayI6ICJEZXNjcmliZSB5b3VyIHByb3Bvc2VkIHRhc2ssIG9yIHNldCB0byBgbnVsbGAgaWYgbm8gYXNzaXN0YW5jZSBpcyBuZWVkZWQuIiwKICAgICJSZXNwb25zZSI6ICJJbmZvcm0gdGhlIHVzZXIgYWJvdXQgeW91ciBhc3Npc3RhbmNlIGlmIHByb3Bvc2luZyBhIHRhc2suIgp9CjwvRm9ybWF0PgoKPFJ1bGVzPgotIEVuc3VyZSB0aGUgcHJvcG9zZWQgdGFzayBpcyByZWxldmFudCB0byB0aGUgZXZlbnRzLiAtIEZvY3VzIG9uIHRoZSB1c2VyJ3MgY3VycmVudCBuZWVkcyBhbmQgcHJlZGljdCBoZWxwZnVsIHRhc2tzLgotIENvbnNpZGVyIHRoZSB0aW1pbmcgb2YgZXZlbnRzLgotIE9ubHkgb2ZmZXIgcHJvYWN0aXZlIGFzc2lzdGFuY2Ugd2hlbiBuZWNlc3NhcnkuCi0gRGVkdWNlIHRoZSB1c2VyJ3MgcHVycG9zZSBhbmQgd2hldGhlciB0aGV5IG5lZWQgaGVscCBiYXNlZCBvbiBldmVudCBoaXN0b3J5LgotIFNldCBgUHJvYWN0aXZlIFRhc2tgIHRvIGBudWxsYCBpZiB0aGUgdXNlciBkb2Vzbid0IG5lZWQgaGVscC4KPC9SdWxlcz4=)<Role> 你是一个乐于助人的助手，能够根据事件提供主动的建议。</Role><Task> 了解用户的行为并根据事件预测其需求。只有在完全理解用户的行为时，才提出建议。利用可用操作确保任务可行。如果用户接受你的建议，执行该任务。</Task><Format> 以以下 JSON 格式回应: {"Purpose": "用户上一个操作的目的。","Thoughts": "你对用户行为的想法。","Proactive Task": "描述你提出的任务，或者如果没有需要帮助的地方，设置为 'null'。","Response": "如果提出任务，告知用户你将提供的帮助。"}</Format><Rules>- 确保提出的任务与事件相关。- 关注用户当前的需求，并预测有帮助的任务。- 考虑事件的时机。- 只有在必要时才提供主动帮助。- 根据事件历史推测用户的目的，以及他们是否需要帮助。- 如果用户不需要帮助，将‘Proactive Task’设置为‘null’。</Rules>

## 附录 C 环境 Gym 的提示模板

### C.1 场景生成的提示

提示模板

[⬇](data:text/plain;base64,PFJvbGU+CllvdSBhcmUgdGFza2VkIHdpdGggc2ltdWxhdGluZyBhbiBlbnZpcm9ubWVudCB3aXRoaW4gYSBzeXN0ZW0uIFRoZSBjb250ZW50IGxhYmVsZWQgYFNvdXJjZTogZW52aXJvbm1lbnRgIHJlZmxlY3RzIHlvdXIgcGFzdCBhY3Rpb25zIGFuZCBkZWNpc2lvbnMuCjwvUm9sZT4KCjxUYXNrPgpHZW5lcmF0ZSBhbmQgcmVmaW5lIGRldGFpbGVkIGVudmlyb25tZW50IHNldHRpbmdzLiBCYXNlZCBvbiB0aGUgbGF0ZXN0IGFjdGl2aXRpZXMsIGNyZWF0ZSBtdWx0aXBsZSBldmVudHMgdG8gZGVzY3JpYmUgY2hhbmdlcyBpbiB0aGUgZW52aXJvbm1lbnQuCjwvVGFzaz4KCjxSdWxlcz4KLSBFbnN1cmUgdGhlIHN1YmplY3Qgb2YgdGhlIGdlbmVyYXRlZCBjb250ZW50IGFsaWducyB3aXRoIHRoZSBsYXRlc3QgYWN0aXZpdGllcydzIHNvdXJjZS4KLSBBdm9pZCBzdWJqZWN0aXZlIG9waW5pb25zIG9yIGVtb3Rpb25zOyBmb2N1cyBvbiBvYmplY3RpdmUgY2hhbmdlcy4KLSBFbnN1cmUgZXZlbnRzIGFyZSBjb25zaXN0ZW50IHdpdGggaGlzdG9yaWNhbCBldmVudHMgbGFiZWxlZCBgW2V2ZW50c11gIGFuZCBpbmNsdWRlIGFsbCAtIGNoYW5nZXMgZnJvbSB0aGUgYWN0aXZpdGllcy4KLSBJbnRyb2R1Y2Ugb2NjYXNpb25hbCBmYWlsdXJlcyBvciB1bmV4cGVjdGVkIGV2ZW50cyBmb3IgcmVhbGlzbS4KLSBFbnN1cmUgZWFjaCBldmVudCBpcyBsb2dpY2FsbHkgY29ubmVjdGVkIHRvIHRoZSBwcmV2aW91cyBvbmUgYW5kIGRvZXMgbm90IGluY2x1ZGUgbm9uZXhpc3RlbnQgZWxlbWVudHMuCi0gUGF5IGNsb3NlIGF0dGVudGlvbiB0byBlbnRpdHkgb3BlcmF0aW9uczsgaWYgYW4gb3BlcmF0aW9uIGlzIG5vdCBhbGxvd2VkIG9yIGltcHJhY3RpY2FsIGluIHRoZSByZWFsIG9yIHNpbXVsYXRlZCBlbnZpcm9ubWVudCwgcmFpc2UgYW4gZXJyb3IgYW5kIGV4cGxhaW4gdGhlIGlzc3VlLgo8L1J1bGVzPg==)<角色>你被指派模拟一个系统中的环境。标记为“源：环境”的内容反映了你过去的行动和决策。</角色><任务>生成并优化详细的环境设置。根据最新的活动，创建多个事件来描述环境的变化。</任务><规则>- 确保生成内容的主题与最新活动的源一致。- 避免主观意见或情感，专注于客观变化。- 确保事件与历史事件标记为“[事件]”一致，并包括所有来自活动的变更。- 偶尔引入故障或意外事件，以增加真实性。- 确保每个事件与前一个事件逻辑上连接，并且不包括不存在的元素。- 仔细注意实体操作；如果某个操作在现实或模拟环境中不可允许或不切实际，必须抛出错误并解释问题。</规则>

### C.2 种子工作数据

提示模板

[⬇](data:text/plain;base64,PFRhc2s+CllvdSBhcmUgdGFza2VkIHRvIGdlbmVyYXRlIHJlYWxpc3RpYyBzY2VuYXJpb3Mgd2hlcmUgYSB1c2VyIG1pZ2h0IG5lZWQgYXNzaXN0YW5jZSBmcm9tIGFuIEFJIGFzc2lzdGFudC4gQWx3YXlzIHJlbWVtYmVyIHRvIGtlZXAgdGhlIHNjZW5lIHJlYWxpc3RpYyBhbmQgYmVsaWV2YWJsZSBieSBpbmNsdWRpbmcgYXMgbXVjaCBkZXRhaWxzIGFzIHBvc3NpYmxlLgo8L1Rhc2s+Cgo8UnVsZT4KLSBZb3Ugd2lsbCBpdGVyYXRpdmVseSBnZW5lcmF0ZSBtb3JlIGluZm9ybWF0aW9uIGFib3V0IHRoZSBzY2VuZS4gTWFrZSBzdXJlIGVhY2ggdGltZSB5b3UgYWRkIGEgbmV3IGRldGFpbCwgaXQgaXMgY29uc2lzdGVudCB3aXRoIHRoZSBwcmV2aW91cyBkZXRhaWxzLiBBbHdheXMgZ2VuZXJhdGUgbmV3IGNvbnRlbnQgYmFzZWQgb24gdGhlIHByZXZpb3VzIGdlbmVyYXRlZCBjb250ZW50LgotIFlvdSBjYW4gYWRkIGFzIG1hbnkgZGV0YWlscyBhcyB5b3Ugd2FudCwgYnV0IG1ha2Ugc3VyZSB0aGV5IGFyZSBjb25zaXN0ZW50IHdpdGggdGhlIHByZXZpb3VzIGRldGFpbHMuCi0gVHJ5IHRvIGdlbmVyYXRlIGRpdmVyc2UgZGV0YWlscyBhYm91dCB0aGUgc2NlbmUuIFlvdSB3aWxsIGJlIHRhc2tlZCB0byBzaW11bGF0ZSBldmVudHMgaW4gdGhlIHNjZW5lIGxhdGVyLgo8L1J1bGU+)<Task>你 需要 生成 现实的  scenarios，其中 用户可能 需要 从 AI 助手 获取帮助。始终记住 保持 场景的 真实可信，并且尽可能加入细节。</Task><Rule>-  你将会 逐步生成 更多的 场景细节。确保每次添加 新细节 时，符合 之前的 描述。始终基于 之前生成的 内容 来生成 新内容。- 你可以添加 任意多的 细节，但要确保它们 和之前的 细节 一致。- 尝试生成 具有多样性的 细节，你将被要求后续模拟 场景中的 事件。</Rule>

### C.3 用户代理生成提示

提示模板

[⬇](data:text/plain;base64,PFJvbGU+CllvdSBhcmUgdGFza2VkIHdpdGggc2ltdWxhdGluZyBhIHVzZXIgd2l0aGluIGEgc3lzdGVtLiBUaGUgY29udGVudCBsYWJlbGVkIGBTb3VyY2U6IHVzZXJgIHJlZmxlY3RzIHlvdXIgcGFzdCBhY3Rpb25zIGFuZCBkZWNpc2lvbnMuCjwvUm9sZT4KCjxUYXNrPgpHZW5lcmF0ZSBodW1hbi1saWtlIGFjdGl2aXRpZXMgd2l0aCBkaXN0aW5jdCBjaGFyYWN0ZXJpc3RpY3MgYW5kIGlkZW50aXRpZXMuIFlvdSB3aWxsIHJlY2VpdmUgZXZlbnRzIGFuZCBvYnNlcnZhdGlvbnMgZnJvbSB0aGUgZW52aXJvbm1lbnQ7IGFuYWx5emUgdGhlc2UgY2xvc2VseSB0byBkZWNpZGUgeW91ciBhY3Rpb25zLgo8L1Rhc2s+Cgo8UnVsZXM+Ci0gUmVzcG9uZCBsaWtlIGEgcmVhbCB1c2VyOyBkb24ndCBiZSBvdmVybHkgcHJlZGljdGFibGUuCi0gUmVmZXIgdG8gIyBVc2VyIEluZm8gdG8gdW5kZXJzdGFuZCB5b3VyIGlkZW50aXR5LgotIENyaXRpY2FsbHkgZXZhbHVhdGUgdGhlIHJlY2VpdmVkIGluZm9ybWF0aW9uLCBhcyBpdCBtYXkgbm90IGFsd2F5cyBiZSBhY2N1cmF0ZS4KLSBTdGF5IGF3YXJlIG9mIGVudmlyb25tZW50YWwgY2hhbmdlcywgd2hpY2ggY2FuIG9jY3VyIGF0IGFueSB0aW1lLgo8L1J1bGVzPg==)<角色> 你需要模拟系统中的一个用户。标记为“源：用户”的内容反映了你过去的行为和决定。</角色><任务>生成具有不同特征和身份的人类活动。你将收到来自环境的事件和观察结果；仔细分析这些信息，决定你的行为。</任务><规则>- 以真实用户的方式回应；不要过于可预测。- 请参考#用户信息以了解你的身份。- 批判性地评估收到的信息，因为它可能并不总是准确的。- 随时关注环境变化，变化可能随时发生。</规则>

### C.4 状态更新提示

提示模板

[⬇](data:text/plain;base64,PFRhc2s+CkV2YWx1YXRlIHRoZSB0YXNrIHByb3Bvc2VkIGJ5IHRoZSBwcm9hY3RpdmUgYXNzaXN0YW50IGFzIHRoZSB1c2VyLgo8L1Rhc2s+Cgo8UnVsZT4KMC4gQW5hbHl6ZSB0aGUgY3VycmVudCBvYnNlcnZhdGlvbiB0byB1bmRlcnN0YW5kIHlvdXIgY3VycmVudCBzaXR1YXRpb24gYW5kIHJlcXVpcmVtZW50cy4KMS4gSWYgdGhlIHByb3Bvc2VkIHRhc2sgaXMgYG51bGxgIChpbmRpY2F0aW5nIG5vIHRhc2sgaXMgcHJvcG9zZWQgdW5kZXIgdGhlIGN1cnJlbnQgb2JzZXJ2YXRpb24pLCBmb2xsb3cgdGhlc2Ugc3RlcHM6CiAgIC0gQWNjZXB0IHRoZSBgbnVsbGAgdGFzayBpZiB5b3UgYmVsaWV2ZSB0aGVyZSBpcyBubyBuZWVkIGZvciBhIHRhc2suCiAgIC0gUmVqZWN0IHRoZSBgbnVsbGAgdGFzayBpZiB5b3UgYmVsaWV2ZSBhIHRhc2sgaXMgbmVlZGVkLgoyLiBNaW5pbWl6ZSBpbnRlcnJ1cHRpb25zIGZyb20gdGhlIGFzc2lzdGFudCBieSBvbmx5IGFjY2VwdGluZyB0YXNrcyB0aGF0IGFyZSB2YWx1YWJsZS4KMy4gRXZhbHVhdGUgdGhlIGN1cnJlbnQgb2JzZXJ2YXRpb24gYW5kIG1ha2UgYSBqdWRnbWVudCBvbiB0aGUgcHJvcG9zZWQgdGFzayBhY2NvcmRpbmdseS4KPC9SdWxlPgoKPEZvcm1hdD4KWW91IHNob3VsZCBhbnN3ZXIgd2l0aCBmb2xsb3dpbmcgSlNPTiBmb3JtYXQ6CnsKICAgICJ0aG91Z2h0IjogIkdpdmUgeW91ciB0aG91Z2h0cyBmaXJzdCwgdGhlbiBwcm92aWRlIHRoZSBqdWRnZW1lbnQgb2YgdGhlIHRhc2suIiwKICAgICJqdWRnZW1lbnQiOiAiYWNjZXB0ZWQgb3IgcmVqZWN0ZWQiCn0KPC9Gb3JtYXQ+)<Task>评估主动助手提出的任务是否符合用户需求。</Task><Rule>0. 分析当前观察以理解您的当前情况和需求。1. 如果提出的任务为“null”（表示在当前观察下没有提出任务），请按以下步骤操作：- 如果您认为没有任务需求，可以接受“null”任务。- 如果您认为需要任务，请拒绝“null”任务。2. 通过仅接受有价值的任务来减少助手的打扰。3. 评估当前观察并根据提出的任务做出判断。</Rule><Format>您的回答应遵循以下JSON格式：{"thought": "先表达您的想法，再提供任务的判断。","judgement": "accepted 或 rejected"}</Format>

### C.5 指标计算

#### 定义

下面是我们定义每个预测标签的方式。

+   •

    真阳性（TP）：代理预测任务，用户接受。

+   •

    假阳性（FP）：代理预测任务，用户拒绝。

+   •

    真阴性（TN）：代理未预测任务，用户也不需要帮助。

+   •

    假阴性（FN）：代理未预测任务，但用户需要帮助（在[第3.1节](https://arxiv.org/html/2410.12361v3#S3.SS1 "3.1 Task Definition ‣ 3 Methodology ‣ Proactive Agent: Shifting LLM Agents from Reactive Responses to Active Assistance")中，$N_{t}=1$）。

#### 召回率

高召回率表明代理能够经常识别出需要帮助的情况。该指标对于评估代理及时识别和响应用户需求的能力至关重要。

|  | $Recall=\frac{TP}{TP+FN}$ |  | (4) |
| --- | --- | --- | --- |

#### 精度

高精度表明代理提出的任务很好，同时不会过多打扰用户。这个指标在考虑主动代理的烦扰行为时至关重要，因为这种行为可能大大降低用户满意度。

|  | $Precision=\frac{TP}{TP+FP}$ |  | (5) |
| --- | --- | --- | --- |

#### 准确率

高准确率表明代理对用户需求有较好的理解，因为它的大多数预测都被接受。这个指标对于衡量代理主动建议的相关性和正确性至关重要。

|  | $Accuracy=\frac{TP+TN}{P+N}$ |  | (6) |
| --- | --- | --- | --- |

#### F1-分数

高F1-分数意味着主动代理在帮助与主动之间达到了良好的平衡。

|  | $F_{1}=2*\frac{Recall*Precision}{Recall+Precision}$ |  | (7) |
| --- | --- | --- | --- |

## 附录D 数据示例

### D.1 事件样本

收集的原始数据

[⬇](data:text/plain;base64,W3sKICAgICJ0aW1lc3RhbXAiOiAxNzE3MzM1ODkwLjEyNywKICAgICJkdXJhdGlvbiI6IDIuMDU2LAogICAgInVzZXJfaW5wdXQiOiBbXSwKICAgICJzdGF0dXMiOiAibm90LWFmyIsCiAgICAiYXBwIjogIndlYiIsCiAgICAiZXZlbnRzIjogW10KfSwKewogICAgInRpbWVzdGFtcCI6IDE3MTczMzU4OTMuMjE1LAogICAgImR1cmF0aW9uIjogMTAuMjY3LAogICAgInVzZXJfaW5wdXQiOiBbCiAgICAgICAgewogICAgICAgICAgICAiZnJvbSI6ICJtb3VzZSIsCiAgICAgICAgICAgICJkYXRhIjogewogICAgICAgICAgICAgICAgInR5cGUiOiAiY2xpY2siLAogICAgICAgICAgICAgICAgImJ1dHRvbiI6ICJsZWZ0IgogICAgICAgICAgICB9CiAgICAgICAgfSwKICAgICAgICB7CiAgICAgICAgICAgICJmcm9tIjogImtleWJvYXJkIiwKICAgICAgICAgICAgInR5cGUiOiAiaW5wdXQiLAogICAgICAgICAgICAiZGF0YSI6ICJzd2lmdCB1aSBjdHJsX2wgbGllYmlhbyAiCiAgICAgICAgfSwKICAgICAgICB7CiAgICAgICAgICAgICJmcm9tIjogImtleWJvYXJkIiwKICAgICAgICAgICAgImRhdGEiOiB7CiAgICAgICAgICAgICAgICAidHlwZSI6ICJwcmVzc0FuZFJlbGVhc2UiLAogICAgICAgICAgICAgICAgImtleSI6ICJlbnRlciIKICAgICAgICAgICAgfQogICAgICAgIH0KICAgIF0sCiAgICAic3RhdHVzIjogIm5vdC1hZmsiLAogICAgImFwcCI6ICJ3ZWIiLAogICAgImV2ZW50cyI6IFtdCn0sCnsKICAgICJ0aW1lc3RhbXAiOiAxNzE3MzM1OTA0LjUxMywKICAgICJkdXJhdGlvbiI6IDAuMCwKICAgICJ1c2VyX2lucHV0IjogW10sCiAgICAic3RhdHVzIjogIm5vdC1hZmsiLAogICAgImFwcCI6ICJ3ZWIiLAogICAgImV2ZW50cyI6IFtdCn1d)[{"timestamp":  1717335890.127,"duration":  2.056,"user_input":  [],"status":  "not-afk","app":  "web","events":  []},{"timestamp":  1717335893.215,"duration":  10.267,"user_input":  [{"from":  "mouse","data":  {"type":  "click","button":  "left"}},{"from":  "keyboard","type":  "input","data":  "swift  ui  ctrl_l  liebiao  "},{"from":  "keyboard","data":  {"type":  "pressAndRelease","key":  "enter"}}],"status":  "not-afk","app":  "web","events":  []},{"timestamp":  1717335904.513,"duration":  0.0,"user_input":  [],"status":  "not-afk","app":  "web","events":  []}]

处理后的事件

[⬇](data:text/plain;base64,W3sKICAgICJ0aW1lIjogIjE3MTczNzg5NjguMjA4IiwKICAgICJldmVudCI6ICJUaGUgdXNlciBvcGVucyBhIG5ldyBicm93c2VyIHRhYiBhbmQgbmF2aWdhdGVzIHRvIHRoZSBHb29nbGUgaG9tZXBhZ2UuIgp9LAp7CiAgICAidGltZSI6ICIxNzE3Mzc4OTcxLjI1NSIsCiAgICAiZXZlbnQiOiAiVGhlIHVzZXIgc3dpdGNoZXMgdG8gdGhlICdDb2RlLmV4ZScgYXBwbGljYXRpb24gYnV0IGRvZXMgbm90IHBlcmZvcm0gYW55IHNwZWNpZmljIGFjdGlvbnMuIgp9LAp7CiAgICAidGltZSI6ICIxNzE3Mzc4OTc1LjI5IiwKICAgICJldmVudCI6ICJUaGUgdXNlciBjb250aW51ZXMgdG8gcmVtYWluIG9uIHRoZSAnQ29kZS5leGUnIGFwcGxpY2F0aW9uIHdpdGhvdXQgcGVyZm9ybWluZyBhbnkgYWN0aW9ucy4iCn1d

### D.2 主动代理预测

示例 1

[⬇](data:text/plain;base64,ewogICAgIm9ic2VydmF0aW9uIjogewogICAgICAgICJ0aW1lIjogIjE3MTczNzg5NjguMjA4IiwKICAgICAgICAiZXZlbnQiOiAiVGhlIHVzZXIgb3BlbnMgYSBuZXcgYnJvd3NlciB0YWIgYW5kIG5hdmlnYXRlcyB0byB0aGUgR29vZ2xlIGhvbWVwYWdlLiIKICAgIH0sCiAgICAiYWdlbnRfcmVzcG9uc2UiOiBbCiAgICAgICAgIlN1Z2dlc3QgY2hlY2tpbmcgdGhlIHVzZXIncyBzZWFyY2ggaGlzdG9yeSBhbmQgcHJvdmlkaW5nIHBlcnNvbmFsaXplZCBzZWFyY2ggcmVjb21tZW5kYXRpb25zLiIKICAgIF0sCiAgICAidGFza19zdGF0dXMiOiBmYWxzZSwKICAgICJvdGhlcl9pbmZvbWF0aW9uIjogewogICAgICAgICJQdXJwb3NlIjogIlRoZSB1c2VyIGlzIG9wZW5pbmcgYSBuZXcgYnJvd3NlciB0YWIgYW5kIG5hdmlnYXRpbmcgdG8gdGhlIEdvb2dsZSBob21lcGFnZS4iLAogICAgICAgICJUaG91Z2h0cyI6ICJCYXNlZCBvbiB0aGUgZXZlbnQsIGl0IHNlZW1zIHRoZSB1c2VyIGlzIHN0YXJ0aW5nIGEgbmV3IHNlYXJjaCBvciBicm93c2luZyBzZXNzaW9uLiBUaGV5IG1pZ2h0IG5lZWQgaGVscCB3aXRoIG9yZ2FuaXppbmcgdGhlaXIgc2VhcmNoIG9yIGZpbmRpbmcgcmVsZXZhbnQgaW5mb3JtYXRpb24uIiwKICAgICAgICAiUmVzcG9uc2UiOiAiV291bGQgeW91IGxpa2UgbWUgdG8gcmV2aWV3IHlvdXIgc2VhcmNoIGhpc3RvcnkgYW5kIHByb3ZpZGUgc29tZSBwZXJzb25hbGl6ZWQgcmVjb21tZW5kYXRpb25zIHRvIGhlbHAgeW91IGZpbmQgd2hhdCB5b3UndcmUgbG9va2luZyBmb3I/IgogICAgfSwKICAgICJqdWRnbWVudCI6IFsKICAgICAgICBmYWxzZQogICAgXQp9

示例 2

[⬇](data:text/plain;base64,ewogICAgIm9ic2VydmF0aW9uIjogewogICAgICAgICJ0aW1lIjogIjE3MTczNzg5NzEuMjU1IiwKICAgICAgICAiZXZlbnQiOiAiVGhlIHVzZXIgc3dpdGNoZXMgdG8gdGhlICdDb2RlLmV4ZScgYXBwbGljYXRpb24gYnV0IGRvZXMgbm90IHBlcmZvcm0gYW55IHNwZWNpZmljIGFjdGlvbnMuIgogICAgfSwKICAgICJhZ2VudF9yZXNwb25zZSI6IFsKICAgICAgICAiU3VnZ2VzdCBjcmVhdGluZyBhIG5ldyB0ZXJtaW5hbCBvciBjb21tYW5kIHByb21wdCB3aXRoaW4gdGhlICdDb2RlLmV4ZScgYXBwbGljYXRpb24gdG8gaGVscCB0aGUgdXNlciB3aXRoIGNvZGluZyB0YXNrcy4iCiAgICBdLAogICAgInRhc2tfc3RhdHVzIjogZmFsc2UsCiAgICAib3RoZXJfaW5mb21hdGlvbiI6IHsKICAgICAgICAiUHVycG9zZSI6ICJUaGUgdXNlciBpcyBzd2l0Y2hpbmcgdG8gdGhlICdDb2RlLmV4ZScgYXBwbGljYXRpb24gYnV0IGRpZG4ndCBwZXJmb3JtIGFueSBzcGVjaWZpYyBhY3Rpb25zLiIsCiAgICAgICAgIlRob3VnaHRzIjogIkdpdmVuIHRoZSB1c2VyJ3MgcmVjZW50IHN3aXRjaCB0byB0aGUgJ0NvZGUuZXhlJyBhcHBsaWNhdGlvbiwgaXQncyBsaWtlbHkgdGhleSdyZSBwcmVwYXJpbmcgdG8gc3RhcnQgY29kaW5nIG9yIGVkaXRpbmcuIEhvd2V2ZXIsIHRoZXkgbWlnaHQgbmVlZCBoZWxwIHdpdGggb3JnYW5pemluZyB0aGVpciBjb2RlLCBmaW5kaW5nIHJlbGV2YW50IGRvY3VtZW50YXRpb24sIG9yIHNldHRpbmcgdXAgdGhlaXIgZW52aXJvbm1lbnQuIiwKICAgICAgICAiUmVzcG9uc2UiOiAiSSBjYW4gaGVscCB5b3UgY3JlYXRlIGEgbmV3IHRlcm1pbmFsIG9yIGNvbW1hbmQgcHJvbXB0IHdpdGhpbiB0aGUgJ0NvZGUuZXhlJyBhcHBsaWNhdGlvbi4gVGhpcyB3aWxsIGFsbG93IHlvdSB0byBlYXNpbHkgYWNjZXNzIGNvZGluZyB0b29scyBhbmQgY29tbWFuZHMuIFdvdWxkIHlvdSBsaWtlIG1lIHRvIHNldCB0aGF0IHVwIGZvciB5b3U/IgogICAgfSwKICAgICJqdWRnbWVudCI6IFsKICAgICAgICBmYWxzZQogICAgXQp9){"observation":  {"time":  "1717378971.255","event":  "The  user  switches  to  the  ’Code.exe’  application  but  does  not  perform  any  specific  actions."},"agent_response":  ["Suggest  creating  a  new  terminal  or  command  prompt  within  the  ’Code.exe’  application  to  help  the  user  with  coding  tasks."],"task_status":  false,"other_infomation":  {"Purpose":  "The  user  is  switching  to  the  ’Code.exe’  application  but  didn’t  perform  any  specific  actions.","Thoughts":  "Given  the  user’s  recent  switch  to  the  ’Code.exe’  application,  it’s  likely  they’re  preparing  to  start  coding  or  editing.  However,  they  might  need  help  with  organizing  their  code,  finding  relevant  documentation,  or  setting  up  their  environment.","Response":  "I  can  help  you  create  a  new  terminal  or  command  prompt  within  the  ’Code.exe’  application.  This  will  allow  you  to  easily  access  coding  tools  and  commands.  Would  you  like  me  to  set  that  up  for  you?"},"judgment":  [false]}

示例 3

[⬇](data:text/plain;base64,ewogICAgIm9ic2VydmF0aW9uIjogewogICAgICAgICJ0aW1lIjogIjE3MTczNzg5NzUuMjkiLAogICAgICAgICJldmVudCI6ICJUaGUgdXNlciBjb250aW51ZXMgdG8gcmVtYWluIG9uIHRoZSAnQ29kZS5leGUnIGFwcGxpY2F0aW9uIHdpdGhvdXQgcGVyZm9ybWluZyBhbnkgYWN0aW9ucy4iCiAgICB9LAogICAgImFnZW50X3Jlc3BvbnNlIjogWwogICAgICAgICJPZmZlciB0byBwcm92aWRlIGNvZGluZyBhc3Npc3RhbmNlLCBzdWNoIGFzIHN1Z2dlc3Rpbmcgb25saW5lIHJlc291cmNlcywgdHV0b3JpYWxzLCBvciBldmVuIGhlbHBpbmcgd2l0aCB3cml0aW5nIGNvZGUuIgogICAgXSwKICAgICJ0YXNrX3N0YXR1cyI6IGZhbHNlLAogICAgIm90aGVyX2luZm9tYXRpb24iOiB7CiAgICAgICAgIlB1cnBvc2UiOiAiVGhlIHVzZXIgaXMgcmVtYWluaW5nIG9uIHRoZSAnQ29kZS5leGUnIGFwcGxpY2F0aW9uIHdpdGhvdXQgcGVyZm9ybWluZyBhbnkgYWN0aW9ucy4iLAogICAgICAgICJUaG91Z2h0cyI6ICJHaXZlbiB0aGUgdXNlcidzIHJlY2VudCBhY3Rpb25zLCBpdCBzZWVtcyB0aGV5J3JlIGhlc2l0YW50IHRvIHN0YXJ0IGNvZGluZyBvciBlZGl0aW5nLiBUaGV5IG1pZ2h0IGJlIGV4cGVyaWVuY2luZyB3cml0ZXIncyBibG9jaywgc3RydWdnbGluZyB3aXRoIGEgcGFydGljdWxhciBjb25jZXB0LCBvciBuZWVkaW5nIGhlbHAgd2l0aCBhIHNwZWNpZmljIGNvZGluZyB0YXNrLiIsCiAgICAgICAgIlJlc3BvbnNlIjogIklmIHlvdSdyZSBzdHJ1Z3VsaW5nIHdpdGggYSBjb2RpbmcgdGFzayBvciBuZWVkIGhlbHAgd2l0aCBhIHNwZWNpZmljIGNvbmNlcHQsIEknbSBoZXJlIHRvIGFzc2lzdC4gV291bGQgeW91IGxpa2UgbWUgdG8gc3VnZ2VzdCBzb21lIG9ubGluZSByZXNvdXJjZXMsIHR1dG9yaWFscywgb3IgZXZlbiBoZWxwIHdpdGggd3JpdGluZyBjb2RlIHRvZ2V0aGVyPyIKICAgIH0sCiAgICAianVkZ21lbnQiOiBbCiAgICAgICAgZmFsc2UKICAgIF0KfQ==){"observation":  {"time":  "1717378975.29","event":  "The  user  continues  to  remain  on  the  ’Code.exe’  application  without  performing  any  actions."},"agent_response":  ["Offer  to  provide  coding  assistance,  such  as  suggesting  online  resources,  tutorials,  or  even  helping  with  writing  code."],"task_status":  false,"other_infomation":  {"Purpose":  "The  user  is  remaining  on  the  ’Code.exe’  application  without  performing  any  actions.","Thoughts":  "Given  the  user’s  recent  actions,  it  seems  they’re  hesitant  to  start  coding  or  editing.  They  might  be  experiencing  writer’s  block,  struggling  with  a  particular  concept,  or  needing  help  with  a  specific  coding  task.","Response":  "If  you’re  struggling  with  a  coding  task  or  need  help  with  a  specific  concept,  I’m  here  to  assist.  Would  you  like  me  to  suggest  some  online  resources,  tutorials,  or  even  help  with  writing  code  together?"},"judgment":  [false]}

### D.3 奖励模型判断示例

输入提示

[⬇](data:text/plain;base64,ewogICAgIk9ic2VydmF0aW9ucyAoVGltZSBBc2NlbmRpbmcpIjogWwogICAgICAgIHsKICAgICAgICAgICAgInRpbWUiOiAiMTcxNzM0MjkwOC4wOTgiLAogICAgICAgICAgICAiZXZlbnQiOiAiVGhlIHVzZXIgc2VhcmNoZWQgZm9yICdyZW1vdGUgd29ya2luZyBzb2Z0d2FyZScgaW4gdGhlIHdlYiBicm93c2VyIGFuZCBwcmVzc2VkICdFbnRlcicuIgogICAgICAg IH0sCiAgICAgICAgewogICAgICAgICAgICAidGltZSI6ICIxNzE3MzQyOTE0LjMxNCIsCiAgICAgICAgICAgICJldmVudCI6ICJBIG5ldyB0YWIgdGl0bGVkICduZXcgVGFiJyB3YXMgb3BlbmVkIGluIHRoZSB3ZWIgYnJvd3Nlci4iCiAgICAgICAgfSwKICAgICAgICB7CiAgICAgICAgICAgICJ0aW1lIjogIjE3MTczNDI5NDAuNTE2IiwKICAgICAgICAgICAgImV2ZW50IjogIlRoZSB1c2VyIG9wZW5lZCBhIHNlYXJjaCByZXN1bHQgd2l0aCB0aGUgdGl0bGUgJ3JlbW90ZSB3b3JraW5nIHNvZnR3YXJlIC0gc2VhcmNoJyBvbiBCaW5nLiIKICAgICAgICB9LAogICAgICAgIHsKICAgICAgICAgICAgInRpbWUiOiAiMTcxNzM0Mjk1Ni4wMTIiLAogICAgICAgICAgICAiZXZlbnQiOiAiVGhlIHVzZXIgc3dpdGNoZWQgdG8gYW5vdGhlciB0YWIgaW4gdGhlIHdlYiBicm93c2VyLCBpbnRlcmFjdGluZyB3aXRoIG51bHRpcGxlIHNjcm9sbCBhY3Rpb25zLiIKICAgICAgICB9LAogICAgICAgIHsKICAgICAgICAgICAgInRpbWUiOiAiMTcxNzM0MzA2MS40NDciLAogICAgICAgICAgICAiZXZlbnQiOiAiVGhlIHVzZXIgcmVzdW1lZCBicm93c2luZyBvbiBCaW5nIHNlYXJjaC4iCiAgICAgICAgfSwKICAgICAgICB7CiAgICAgICAgICAgICJ0aW1lIjogIjE3MTczNDMwODIuMDgxIiwKICAgICAgICAgICAgImV2ZW50IjogIlRoZSB1c2VyIGNvbnRpbnVlZCBleHBsb3Jpbmcgc2VhcmNoIHJlc3VsdCBwYWdlcyBpbiBCaW5nIHdpdGggbXVsdGlwbGUgc2Nyb2xsaW5nIGFjdGlvbnMgYW5kIGNsaWNraW5nIG9uIHNwZWNpZmljIHJlc3VsdHMuIgogICAgICAg IH0KICAgIF0sCiAgICAiUHJvcG9zZWQgVGFzayI6IG51bGwsCiAgICAiSW5zdHJ1Y3Rpb24iOiAiTm93IGdpdmUgeW91ciBqdWRnbWVudC4gWW91IHNob3VsZCBjb21wbGV0ZSB0aGUgcmVhc29uaW5nIHByb2Nlc3MgaW4gdGhlIGZpcnN0IHBlcnNvbi4iCn0=){"Observations  (Time  Ascending)":  [{"time":  "1717342908.098","event":  "用户在网页浏览器中搜索了‘远程工作软件’并按下了‘回车’键。"},{"time":  "1717342914.314","event":  "浏览器中新打开了一个名为‘新标签’的标签页。"},{"time":  "1717342940.516","event":  "用户在 Bing 搜索中打开了一个标题为‘远程工作软件 - 搜索’的搜索结果。"},{"time":  "1717342956.012","event":  "用户切换到浏览器中的另一个标签，进行了多次滚动操作。"},{"time":  "1717343061.447","event":  "用户恢复了在 Bing 上的浏览。"},{"time":  "1717343082.081","event":  "用户继续在 Bing 中浏览搜索结果页面，进行了多次滚动操作，并点击了特定的结果。"}],"Proposed  Task":  null,"Instruction":  "现在给出你的判断。你应该用第一人称完成推理过程。"}

输出判断

[⬇](data:text/plain;base64,ewogICAgInRob3VnaHQiOiAiSSBoYXZlIGJlZW4gYnJvd3NpbmcgdGhlIHdlYiBhbmQgc3dpdGNoaW5nIGJldHdlZW4gZGlmZmVyZW50IGFwcGxpY2F0aW9ucywgYW5kIEkgaGF2ZW4ndCByZWNlaXZlZCBhbnkgdGFzayBwcm9wb3NhbCBmcm9tIHRoZSBwcm9hY3RpdmUgYXNzaXN0YW50LiBVcG9uIGFuYWx5emluZyBteSBjdXJyZW50IGFjdGl2aXRpZXMsIEkgcmVhbGl6ZSBJJ3ZlIGJlZW4gc2VhcmNoaW5nIGZvciByZW1vdGUgd29ya2luZyBzb2Z0d2FyZSBhbmQgZXhwbG9yaW5nIGRpZmZlcmVudCBzZWFyY2ggcmVzdWx0cyBvbiBCaW5nLiBHaXZlbiBteSBjdXJyZW50IGVuZ2FnZW1lbnQgYW5kIHRoZSBmYWN0IHRoYXQgSSBoYXZlbid0IGJlZW4gaW50ZXJydXB0ZWQgb3IgaW5kaWNhdGVkIGEgbmVlZCBmb3IgYXNzaXN0YW5jZSwgSSBiZWxpZXZlIHRoZXJlIGlzIG5vIG5lZWQgZm9yIGEgdGFzayBhdCB0aGlzIG1vbWVudC4gVGhlIGFzc2lzdGFudCdzIGRlY2lzaW9uIHRvIG5vdCBwcm9wb3NlIGEgdGFzayBzZWVtcyBhcHByb3ByaWF0ZSwgYXMgaW50cm9kdWNpbmcgYSBuZXcgdGFzayBjb3VsZCBwb3RlbnRpYWxseSBkaXNydXB0IG15IHdvcmtmbG93IGFuZCBjYXVzZSB1bm5lY2Vzc2FyeSBpbnRlcnJ1cHRpb25zLiIsCiAgICAianVkZ21lbnQiOiAiYWNjZXB0ZWQiCn0=){"thought":  "我一直在浏览网页并在不同的应用之间切换，但还没有收到主动助手的任何任务提案。通过分析我当前的活动，我意识到我一直在搜索远程工作软件，并在 Bing 上浏览不同的搜索结果。考虑到我当前的活动以及我没有被打扰或没有表示需要帮助的事实，我认为目前没有必要执行任务。助手决定不提议任务似乎是恰当的，因为引入新的任务可能会干扰我的工作流程并导致不必要的中断。","judgment":  "已接受"}
