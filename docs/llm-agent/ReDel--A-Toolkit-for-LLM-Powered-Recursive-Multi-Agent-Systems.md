<!--yml

分类：未分类

date: 2025-01-11 12:20:56

-->

# ReDel：一个基于LLM的递归多智能体系统工具包

> 来源：[https://arxiv.org/html/2408.02248/](https://arxiv.org/html/2408.02248/)

Andrew Zhu,  Liam Dugan,  Chris Callison-Burch

宾夕法尼亚大学

{andrz,ldugan,ccb}@seas.upenn.edu

###### 摘要

最近，使用大语言模型（LLM）构建复杂的多智能体系统以执行诸如编写文献综述、草拟消费者报告和规划假期等任务的兴趣日益增加。虽然目前已有许多工具和库帮助创建这类系统，但没有一个支持递归多智能体系统——即模型本身能够灵活决定何时委派任务以及如何组织其委派结构。在这项工作中，我们介绍了ReDel：一个支持自定义工具使用、委派方案、基于事件的日志记录和交互式回放的递归多智能体系统工具包，提供了一个易于使用的网页界面。我们展示了，使用ReDel，我们能够通过可视化和调试工具轻松识别潜在的改进领域。我们的代码、文档和PyPI包都是开源的¹¹1ReDel的源代码可以在[https://github.com/zhudotexe/redel](https://github.com/zhudotexe/redel)找到，并且可以在MIT许可下自由使用。

\setminted

frame=lines, framesep=2mm, baselinestretch=1.2, fontsize=

ReDel：一个基于LLM的递归多智能体系统工具包

Andrew Zhu,  Liam Dugan,  Chris Callison-Burch 宾夕法尼亚大学 {andrz,ldugan,ccb}@seas.upenn.edu

## 1 引言

多智能体系统将多个大语言模型（LLM）结合起来，以完成单个LLM无法胜任的复杂任务或回答复杂问题。在这种场景中，通常会为每个LLM提供工具（Parisi et al.（[2022](https://arxiv.org/html/2408.02248v2#bib.bib8)）；Schick et al.（[2023](https://arxiv.org/html/2408.02248v2#bib.bib13)）），这些工具可以扩展其能力，如搜索互联网获取实时数据或与网页浏览器进行交互。在大多数情况下，这些系统是手动定义的，由人工负责定义一个静态的任务分解图，并为图中的每个子任务定义一个智能体来处理（Hong et al.，[2024](https://arxiv.org/html/2408.02248v2#bib.bib2)；Wu et al.，[2023](https://arxiv.org/html/2408.02248v2#bib.bib15)；Zhang et al.，[2024](https://arxiv.org/html/2408.02248v2#bib.bib19)；Qiao et al.，[2024](https://arxiv.org/html/2408.02248v2#bib.bib12)，等）。

在递归多智能体系统中，不是由人类定义多个智能体的布局，而是给一个根智能体一个工具，用来生成额外的智能体。当面临复杂任务时，根智能体可以将任务分解成更小的子任务，然后将这些任务委派给新创建的子智能体。每个子智能体可以在任务足够小的情况下完成任务，或者递归地进一步分解并委派任务²²2这就是该工具包名称ReDel的由来：它是“递归委派”（Recursive Delegation）的缩写。Khot等人（[2023](https://arxiv.org/html/2408.02248v2#bib.bib3)）；Lee和Kim（[2023](https://arxiv.org/html/2408.02248v2#bib.bib4)）；Prasad等人（[2024](https://arxiv.org/html/2408.02248v2#bib.bib9)）（图[1](https://arxiv.org/html/2408.02248v2#S1.F1 "Figure 1 ‣ 1 Introduction ‣ ReDel: A Toolkit for LLM-Powered Recursive Multi-Agent Systems")）。

在当前的多智能体系统领域，大多数工具都集中在人工定义的静态系统上，且在处理那些在运行时将智能体添加到计算图中的动态系统时效果较差。此外，这些工具大多不适合学术用途，或者被隐藏在付费墙和专有许可之后。Zhu等人（[2023](https://arxiv.org/html/2408.02248v2#bib.bib22)）。

![参考标题](img/e25a41fb21c397e8bb81f2448c1b9855.png)

图1：ReDel允许开发者创建递归智能体系统，检查每个智能体的状态，并可视化系统的委派图（右）。递归智能体可用于解决复杂任务，例如规划前往日本的旅行（左）。

在本文中，我们介绍了ReDel，这是一个功能齐全的开源工具包，用于递归多智能体系统。ReDel通过提供一个模块化的接口来创建工具、不同的委派方法和日志，以便于后期分析，从而简化了实验过程。该系统的细粒度日志记录和中央事件驱动系统使得从系统中的任何地方监听信号变得容易，并且每个事件都会自动记录下来，便于事后数据分析。ReDel还提供了一个网页界面，允许用户直接与配置好的系统进行交互，并查看保存的运行回放，这使得研究人员和开发人员能够轻松构建、迭代和分析递归多智能体系统。在第[4](https://arxiv.org/html/2408.02248v2#S4 "4 Evaluation & Case Study ‣ ReDel: A Toolkit for LLM-Powered Recursive Multi-Agent Systems")节中，我们使用ReDel在三个不同的智能体基准上运行递归多智能体系统，而在第[5](https://arxiv.org/html/2408.02248v2#S5 "5 Using ReDel for Error Analysis ‣ ReDel: A Toolkit for LLM-Powered Recursive Multi-Agent Systems")节中，我们展示了如何使用该工具包探索这些系统的复杂行为。

## 2 相关工作

#### 递归多智能体系统。

关于递归多智能体系统的最新研究工作由Lee和Kim（[2023](https://arxiv.org/html/2408.02248v2#bib.bib4)）、Khot等人（[2023](https://arxiv.org/html/2408.02248v2#bib.bib3)）、Qi等人（[2023](https://arxiv.org/html/2408.02248v2#bib.bib10)）和Prasad等人（[2024](https://arxiv.org/html/2408.02248v2#bib.bib9)）完成。这些工作提出了微调或少样本提示LLM（大语言模型）以分解复杂任务，并使用子智能体解决每个部分的方法（通常称为递归或分层分解）。ReDel基于这些工作中介绍的方法，利用现代模型本身的工具使用能力（Schick等人，[2023](https://arxiv.org/html/2408.02248v2#bib.bib13)）进行零样本分解和任务委派（即无需人类编写的提示示例），而不是使用少样本提示或微调。作为一个框架，我们提供了一个可扩展的接口，将这些方法应用于更多的任务和领域。

其他多智能体系统方法，如智能体进化（Qian等人，[2024](https://arxiv.org/html/2408.02248v2#bib.bib11)）；Yuan等人（[2024](https://arxiv.org/html/2408.02248v2#bib.bib18)）；Zhou等人（[2024b](https://arxiv.org/html/2408.02248v2#bib.bib21)）则通过扰动人类编写的提示和工具，动态创建新的子智能体变体。在本文中，我们选择探索使用零样本提示和函数调用进行任务委派，而不进行动态适应，但我们的框架足够灵活，也能够实现这些替代的智能体委派方法。

#### 多智能体系统框架。

尽管还有其他基于LLM的多智能体系统框架，但它们各自存在各种弱点，使其不适合递归系统和/或学术用途。在表[1](https://arxiv.org/html/2408.02248v2#S2.T1 "表1 ‣ 多智能体系统框架 ‣ 2 相关工作 ‣ ReDel：基于LLM的递归多智能体系统工具包")中，我们将LangGraph Campos等人（[2023](https://arxiv.org/html/2408.02248v2#bib.bib1)）、LlamaIndex Liu等人（[2022](https://arxiv.org/html/2408.02248v2#bib.bib5)）、MetaGPT Hong等人（[2024](https://arxiv.org/html/2408.02248v2#bib.bib2)）、AutoGPT Significant Gravitas（[2023](https://arxiv.org/html/2408.02248v2#bib.bib14)）和XAgent XAgent团队（[2023](https://arxiv.org/html/2408.02248v2#bib.bib16)）与我们的系统ReDel进行对比。大多数系统都是围绕静态多智能体系统构建的，只有AutoGPT和XAgent支持单级委托。只有LangGraph和LlamaIndex允许智能体并行异步运行，而MetaGPT、AutoGPT和XAgent则以同步方式逐个运行智能体。为了记录系统深层次的事件，只有LlamaIndex为开发者提供了严格的工具包，使他们能够在系统运行时的任何时刻发出事件。大多数系统不允许开发者从日志中重放系统运行，只有LangGraph通过对系统每个状态进行快照来支持重放。大多数系统没有提供可视化接口，只有AutoGPT和XAgent提供了一个简单的基于聊天的用户界面。除非订阅付费服务，否则LangGraph的重放不能以可视化方式查看，而是以每个状态的原始数据呈现。最后，只有AutoGPT、MetaGPT和XAgent是完全开源的，而LangGraph和LlamaIndex则使用专有代码提供更多超出其开源库功能的“高级”功能。

相比之下，ReDel允许开发者定制智能体的委托策略，并构建多级动态系统，同时提供所有这些功能，并且完全免费和开源。它是唯一一个为递归多智能体系统提供一流支持的工具包，并在系统可视化和现代LLM工具使用方面提供最佳支持。

|  |  ReDel  |  LangGraph  |  LlamaIndex  |  MetaGPT  |  AutoGPT  |  XAgent  |
| --- | --- | --- | --- | --- | --- | --- |
| 动态系统 | \scalerel*![[未加说明的图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[未加说明的图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[未加说明的图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[未加说明的图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[未加说明的图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[未加说明的图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ |
| 并行代理 | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ |
| 事件驱动 | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ |
| 运行重放 | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ |
| 网络接口 | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[无标题图片]](img/9cf4324911d3b2a7cf4831b1385d179a.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ |
| 完全开源 | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[无标题图片]](img/e59b12dac6ea56abd1b13e3b4425e291.png)○ | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ | \scalerel*![[无标题图片]](img/554bf6fe44534f0edeb7ce4f6553c87a.png)○ |

表1：ReDel与竞争工具包的功能对比。ReDel是唯一一个完全开源的工具包，支持基于丰富的事件驱动和网络接口的动态多代理系统。

## 3 系统设计

ReDel由两部分组成：一个Python包，用于定义递归委托系统、记录事件和运行实验；以及一个Web接口，用于快速和交互式地迭代已定义的系统或分析实验日志。在接下来的章节中，我们将更详细地讨论这些组件。

### 3.1 工具使用

在 ReDel 中，“工具”是一个由 Python 编写的函数组，它对代理公开。代理可能会生成请求来调用此工具中的适当函数，这些函数与环境进行交互（例如，搜索互联网）。

{minted}

python class MyHTTPTool(ToolBase): @ai_function() def get(self, url: str): """获取网页内容，并返回原始 HTML。""" resp = requests.get(url) return resp.text

图 2：一个简单的 ReDel 工具示例，它向任何配备该工具的代理公开一个 HTTP GET 功能。

开发者可以在任何 Python 文件中定义工具，并且工具的方法可以由任何 Python 代码实现。ReDel 是用纯 Python 实现的，方法体不会被发送到代理的底层语言模型中，因此工具的实现复杂度和长度没有限制。同样，工具可以使用任何其他外部库中定义的功能，这允许开发者利用现有的应用代码。一个提供 HTTP 请求功能的基本工具示例如图 [2](https://arxiv.org/html/2408.02248v2#S3.F2 "图 2 ‣ 3.1 工具使用 ‣ 3 系统设计 ‣ ReDel：一个为 LLM 提供递归多代理系统的工具包") 中所示。

ReDel 配备了一个网页浏览工具和一个电子邮件工具作为示例，我们鼓励开发者实现适用于自己目的的领域特定工具。

{minted}

python prompt_toks = Counter() out_toks = Counter()

for event in read_jsonl("/path/to/events.jsonl"): if event["type"] == "tokens_used": eid = event["id"] prompt_toks[eid] += event["prompt_tokens"] out_toks[eid] += event["completion_tokens"]

图 3：ReDel 系统中的每个事件，无论是内建的还是自定义的，都会记录到一个 JSONL 文件中。开发者可以使用他们选择的数据分析工具对事件日志进行事后分析。这个示例展示了令牌计数。

{minted}

python # 定义一个自定义事件类 CustomToolEvent(BaseEvent): type: Literal["custom_event"] = "custom_event" id: str # 派发代理的 ID foo: str # 其他数据

# 定义一个派发事件的工具类 MyTool(ToolBase): @ai_function() def my_cool_function(self): self.app.dispatch( CustomToolEvent(id=self.kani.id, foo="bar") ) # 其他行为在这里 …

图 4：使用 ReDel 定义一个自定义事件并从工具中派发它。自定义事件可以用来在系统内部深度添加可观察性，并可以在事后查询以进行丰富的数据分析。

### 3.2 委托方案

委托方案是代理用来将任务分配给子代理的策略。在 ReDel 中，委托方案作为一种特殊类型的工具实现，LLM 代理（“父代理”）可以通过将任务指令作为参数来调用这些工具。这些指令会被发送到一个新的子代理（“子代理”），如果任务足够简单，子代理可以直接完成，或者将任务拆分成更小的部分并递归地再次委托。

![参见标题](img/f6ad21cb1ac4d586dc45f3f940ce84a5.png)

(a) ReDel 网页界面的首页。

![参见说明文字](img/01db034979a44a7149ebe19b1012260d.png)

(b) ReDel 的交互式视图允许用户快速迭代提示和工具设计，并测试端到端的性能。

![参见说明文字](img/c7dead40e9fc006321846d31e1da0821.png)

(c) 保存浏览器显示存储在文件系统中配置目录中的日志。它允许开发者搜索和查看以前的 ReDel 系统运行记录。

![参见说明文字](img/10823812f24d86d506259eb68bea86ff.png)

(d) ReDel 的重放视图允许开发者重放保存的 ReDel 系统运行记录，在分析或调试系统性能时，为事件提供时间上下文。

图 5：ReDel Web 界面的四种视图：主页（a）、交互式（b）、保存浏览器（c）和重放（d）。

受到操作系统中常见进程管理范式的启发，ReDel 提供了两种委托方案：

+   •

    DelegateOne：同步阻塞父代理的执行，直到子代理返回其结果（以聊天输出的形式）。

+   •

    DelegateWait³³3 方案因此得名，因为它为代理提供了两个功能：delegate()，用于向子代理发送指令并启动它，和 wait()，用于检索子代理的结果，如有必要，则等待其完成：不阻塞父代理的执行。相反，提供一个单独的函数来异步获取特定子代理的结果（聊天输出）。

DelegateOne 方案非常适合具有并行函数调用的 LLM，因为它允许 ReDel 让一组已启动的子代理并行运行，并在所有子代理完成后返回其结果。

相比之下，DelegateWait 方案非常适合没有并行函数调用的 LLM，因为它允许这些模型在决定等待某个代理的结果之前，先启动多个代理（即，获取其对话输出）。缺点是，如果父代理从未检索某个子代理的结果，就有可能造成僵尸代理。⁴⁴根据我们的测试，这种情况比较少见。据我们了解，ReDel 是第一个实现这种延迟委托方案的系统。

开发者还可以模块化实现自己的委托方案，方式类似于定义工具，这样可以启用更复杂的行为。例如，开发者可以实现一个委托方案，允许父代理向现有子代理提出后续问题，从而实现多轮委托。开发者还可以使用委托方案控制子代理如何将信息传递回父代理——例如，让每个子代理调用 set_result() 函数明确记录其对子任务的答案，而不是隐式地将其聊天输出发送给父代理。我们在附录 [A](https://arxiv.org/html/2408.02248v2#A1 "附录 A 自定义委托方案 ‣ ReDel：一个 LLM 驱动的递归多代理系统工具包") 和 GitHub 仓库中包含了如何定义委托方案的示例。

### 3.3 事件与日志

ReDel 作为一个事件驱动框架，拥有全面的内置事件，并且能够定义自定义事件。一个事件可以定义为从子代理的创建到特定工具的使用等任何事情。每当 ReDel 捕获到一个事件时，它会将该事件记录到一个 JSONL 文件中。这个文件本质上充当了系统运行的执行追踪，用户可以使用标准的数据分析工具来检查这个追踪并调试他们的运行。图 [3](https://arxiv.org/html/2408.02248v2#S3.F3 "Figure 3 ‣ 3.1 Tool Usage ‣ 3 System Design ‣ ReDel: A Toolkit for LLM-Powered Recursive Multi-Agent Systems") 显示了如何使用一个基本的 Python 脚本来统计系统在事后使用的令牌。

此外，仅使用内置事件，ReDel 就能够通过我们的 Web 界面交互式地回放任何响应，以提供额外的可视化调试辅助（请参阅第 [3.4](https://arxiv.org/html/2408.02248v2#S3.SS4 "3.4 Web Interface ‣ 3 System Design ‣ ReDel: A Toolkit for LLM-Powered Recursive Multi-Agent Systems") 节）。在第 [4](https://arxiv.org/html/2408.02248v2#S4 "4 Evaluation & Case Study ‣ ReDel: A Toolkit for LLM-Powered Recursive Multi-Agent Systems") 节中，我们展示了一个案例研究，说明如何使用这一功能来调试复杂的查询失败。我们在附录 [B](https://arxiv.org/html/2408.02248v2#A2 "Appendix B Application Events ‣ ReDel: A Toolkit for LLM-Powered Recursive Multi-Agent Systems") 中提供了内置默认事件集，并在图 [4](https://arxiv.org/html/2408.02248v2#S3.F4 "Figure 4 ‣ 3.1 Tool Usage ‣ 3 System Design ‣ ReDel: A Toolkit for LLM-Powered Recursive Multi-Agent Systems") 中展示了定义自定义事件的示例。

### 3.4 Web 界面

Web 界面由四个主要视图组成：

#### 主页。

主页（图 [5(a)](https://arxiv.org/html/2408.02248v2#S3.F5.sf1 "In Figure 5 ‣ 3.2 Delegation Schemes ‣ 3 System Design ‣ ReDel: A Toolkit for LLM-Powered Recursive Multi-Agent Systems")）是首次启动界面时的默认视图。用户可以通过在聊天栏中发送消息切换到交互式视图，或者使用提供的按钮加载已保存的回放或了解更多关于 ReDel 的信息。侧边栏让用户可以在他们已启动的交互会话之间切换，启动新的会话，或者加载已保存的回放。

#### 交互视图。

在交互视图（图 [5(b)](https://arxiv.org/html/2408.02248v2#S3.F5.sf2 "在图 5 ‣ 3.2 委托方案 ‣ 3 系统设计 ‣ ReDel：一个基于 LLM 的递归多代理系统工具包")）中，用户可以向根节点发送消息与系统进行互动。当系统运行时，右上面板会显示委托图：一个每个代理在系统中的可视化表示，展示它们的父节点、子节点以及当前状态：运行中（绿色）、等待中（黄色）或已完成（灰色）。用户可以通过点击委托图中的每个节点进一步检查它，底部右侧面板将显示该节点的完整消息历史。ReDel 支持流式处理，每个代理的 LLM 生成会实时显示。

|  | FanOutQA | TravelPlanner | WebArena |
| --- | --- | --- | --- |
| 系统 | Loose | 模型评估 | CS-Micro | H-Micro | 最终 | SR | SR（AC） | SR（UA） |
| ReDel（GPT-4o） | 0.687 | 0.494 | 67.49 | 9.52 | 2.78 | 0.203 | 0.179 | 0.643 |
| ReDel（GPT-3.5-turbo） | 0.300 | 0.087 | 54.58 | 0 | 0 | 0.092 | 0.066 | 0.571 |
| 基准（GPT-4o） | 0.650 | 0.394 | 50.83 | 18.81 | 0 | 0.162 | 0.128 | 0.786 |
| 基准（GPT-3.5-turbo） | 0.275 | 0.077 | 48.75 | 0.24 | 0 | 0.085 | 0.058 | 0.571 |
| 已发布的 SotA | 0.580 | 0.365 | 61.1 | 15.2 | 1.11 | 0.358 | — | — |

表 2：系统在 FanOutQA、TravelPlanner 和 WebArena 上的表现。SotA 模型分别是 FanOutQA 上的 GPT-4o、TravelPlanner 上的 GPT-4-turbo/Gemini Pro，以及 WebArena 上的 SteP。我们看到，ReDel 在所有基准测试中都优于对应的单一代理基准，并且在三项中的两项上超过了已发布的 SotA。

#### 保存浏览器。

保存浏览器（图 [5(c)](https://arxiv.org/html/2408.02248v2#S3.F5.sf3 "在图 5 ‣ 3.2 委托方案 ‣ 3 系统设计 ‣ ReDel：一个基于 LLM 的递归多代理系统工具包")）允许用户从之前会话的列表中选择回放进行查看。这使得研究人员可以批量运行实验，同时保存日志，并使用界面在稍后的日期回顾系统的行为。保存列表包含了 ReDel 服务器在提供的保存目录中找到的所有保存项，它们的标题、事件数以及最后编辑时间。用户可以在保存标题中搜索关键字，也可以按名称、编辑时间或事件数对保存进行排序——后者可以让用户快速一目了然地找到异常值。

#### 回放视图。

仅使用内置的默认事件（见附录[B](https://arxiv.org/html/2408.02248v2#A2 "附录B 应用事件 ‣ ReDel：一个面向LLM的递归多代理系统工具包")），ReDel能够保存足够的信息来在重放设置中完全重建一个会话。因此，重放视图（图[5(d)](https://arxiv.org/html/2408.02248v2#S3.F5.sf4 "图5 ‣ 3.2 委派方案 ‣ 3 系统设计 ‣ ReDel：一个面向LLM的递归多代理系统工具包")）允许用户逐步查看系统在特定会话期间调度的每个事件（包括内置和自定义事件），并可视化每个事件对系统的影响。

重放视图的布局几乎与交互式视图相同，唯一的不同是消息栏被重放控制按钮替换。用户可以使用这些控制按钮在根节点、委派图中的选定节点之间跳转，或者使用滑块查找事件。消息历史和委派图会实时更新，随着用户在重放中查找事件。

## 4 评估与案例研究

为了评估ReDel，我们将其性能与基准单代理系统以及已发布的最先进系统进行了比较，使用了三种不同的基准。我们在代码发布中包含了所有实验的日志和源代码。

### 4.1 实验设置

#### 基准测试。

为了正确评估ReDel，我们必须选择包含足够复杂任务的数据集。因此，我们为基准选择了以下数据集：

1.  1.

    FanOutQA：Zhu等人（[2024](https://arxiv.org/html/2408.02248v2#bib.bib23)）代理必须从多个维基百科文章中汇编数据，以回答复杂的信息查询。

1.  2.

    TravelPlanner：Xie等人（[2024](https://arxiv.org/html/2408.02248v2#bib.bib17)）代理必须使用工具创建旅行计划，搜索航班、餐馆和景点数据库。

1.  3.

    WebArena：Zhou等人（[2024a](https://arxiv.org/html/2408.02248v2#bib.bib20)）代理必须执行复杂的网络任务，如将商品添加到购物车或在GitLab上发表评论。

由于成本限制，我们将评估范围限制为每个基准大约100-300个例子（见附录[C](https://arxiv.org/html/2408.02248v2#A3 "附录C 基准比较 ‣ ReDel：一个面向LLM的递归多代理系统工具包")）。

#### 模型。

对于我们的两个主要ReDel系统，我们使用了GPT-4o OpenAI（[2024](https://arxiv.org/html/2408.02248v2#bib.bib7)）和GPT-3.5-turbo OpenAI（[2022](https://arxiv.org/html/2408.02248v2#bib.bib6)）作为底层模型。在所有设置中，根节点没有被赋予工具使用能力，并使用DelegateOne委派方案。

对于两个基准系统，我们直接使用了GPT-4o和GPT-3.5-turbo模型。所有模型均被赋予了对所有工具的平等访问权限，且未进行少量示例提示或微调。

### 4.2 结果

在表[2](https://arxiv.org/html/2408.02248v2#S3.T2 "Table 2 ‣ Interactive View. ‣ 3.4 Web Interface ‣ 3 System Design ‣ ReDel: A Toolkit for LLM-Powered Recursive Multi-Agent Systems")中，我们报告了评估的结果。我们看到，在所有基准测试中，我们的递归委派系统明显优于其对应的单一代理基线。我们甚至在FanOutQA和TravelPlanner两个方面超越了先前的最先进系统。

此外，我们还看到，随着基础模型能力的提升，ReDel与基线系统之间的差距变得更大。我们认为，这预示着此类技术在未来更强大的模型中应用的前景光明。

在ReDel失败的少数情况下，即TravelPlanner中的H-Micro和WebArena中的SR，这些问题归因于指标失败和不平衡的比较。在TravelPlanner的案例中，经过进一步检查，我们发现递归系统往往会为餐食输入更多常识性的内容（例如“在飞机上”或“打包午餐”），这会导致TravelPlanner评估脚本在硬性约束指标上给出0分。至于WebArena的结果，发布的SotA SteP模型使用了少样本链式思维提示，而我们的系统全部使用零样本提示。

## 5 使用ReDel进行错误分析

在我们的错误分析中，我们取出了每个基准的保存日志文件，并通过ReDel web界面的回放视图手动检查了成功运行和失败运行的日志。通过这次调查，我们观察到递归多代理系统中存在两种常见的失败案例。这些案例如下：

+   •

    过度承诺：代理试图自行完成一个过于复杂的任务。

+   •

    欠承诺：代理不执行任何工作，而是将其收到的任务重新委派。

我们发现，过度承诺通常发生在代理执行多个工具调用并将检索到的信息填充到其上下文窗口时。在ReDel web界面中，这表现为一个异常小的委派图，通常只有两个节点：根节点和根节点委派给的单个子节点，而该子节点随后发生过度承诺。在实践中，这通常（但不总是）导致过度承诺的模型“忘记”它本应完成的任务，因为原始任务由于上下文窗口的限制而被截断。一个过度承诺的模型可能会因为输出上下文窗口中剩余内容的总结，而不是回答原始任务，导致任务失败；而由于非过度承诺原因导致的任务失败，可能表现为一个幻觉结果或一个简单的道歉，表示无法完成任务。

相反，我们发现不足委派通常发生在模型错误地决定自己没有解决问题所需的工具时，并假设其未来的子代理会拥有解决问题所需的工具。在所有三个基准测试中，这导致了失败，因为代理进入了一个无限委派的循环，直到达到配置的深度限制或超时。在网页界面中，这表现为委派图中的一条节点链（图 [6](https://arxiv.org/html/2408.02248v2#S5.F6 "Figure 6 ‣ 5 Using ReDel for Error Analysis ‣ ReDel: A Toolkit for LLM-Powered Recursive Multi-Agent Systems")）。

在表格 [3](https://arxiv.org/html/2408.02248v2#S5.T3 "Table 3 ‣ 5 Using ReDel for Error Analysis ‣ ReDel: A Toolkit for LLM-Powered Recursive Multi-Agent Systems") 中，我们列出了ReDel在每个基准测试中与GPT-4o和GPT-3.5-turbo的过度委派和不足委派率。我们通过启发式方法完成了这一计算，任何代理数量不超过两个的委派图被视为过度委派，而任何链条中有三名或以上的代理且每个代理仅有零个或一个子代理的委派图则被视为不足委派。我们发现，随着模型的增强，它们的委派倾向更强。然而，这种委派倾向可能导致不足委派。

|  | FOQA | TP | WA |
| --- | --- | --- | --- |
| 系统   | 过度委派 | 不足委派 | 过度委派 | 不足委派 | 过度委派 | 不足委派 |
| RD (4o) | 22.7 | 11.3 | 41.1 | 0.5 | 31.3 | 44.8 |
| RD (3.5-t) | 40.8 | 1.1 | 96.7 | 0 | 54.6 | 17.7 |

表格 3：我们测试的两个递归多代理系统在各个基准测试中的过度委派（OC）和不足委派（UC）率（百分比）。

鉴于这两种问题的普遍性，我们假设递归多代理系统可能仍能通过针对这些行为的干预进一步提高性能。例如，可以通过微调或提示代理，给出特定领域的指令，详细说明模型何时应该委派任务，何时应独立完成任务。

尽管实施此类改进超出了本文的范围，但我们相信这个案例研究有助于展示ReDel系统的优势。通过委派图视图，可以轻松识别和表征递归多代理系统中的错误，我们希望通过ReDel，能够进行更多的研究，以进一步完善这些系统，以达到最大效用。

![参考说明](img/986e55e6524f75babd8294ad5017b97d.png)

图 6：递归系统表现出不足委派时，会产生长链条的代理（蓝色框），如在ReDel委派图中所见。

## 6 结论

我们提出了 ReDel，一种用于递归多智能体系统的全新工具包。ReDel 使学术开发者能够快速构建、迭代并运行涉及动态多智能体系统的实验。它提供了一个模块化接口，用于创建智能体使用的工具，一个事件框架，用于为后续分析提供实验数据记录，还有一个自由开源的 web 接口，供开发者与其定义的系统进行交互和探索。我们使用 ReDel 展示递归多智能体系统在三个不同基准测试中的表现，并将这些实验的完整日志包含在我们的演示版本中，以确保可复现性和进一步探索⁵⁵5[https://datasets.mechanus.zhu.codes/redel-dist.zip](https://datasets.mechanus.zhu.codes/redel-dist.zip)。ReDel 为递归多智能体系统开辟了新范式的大门，我们期待看到开发者未来如何利用我们的系统。

## 致谢

本研究部分由美国国家情报总监办公室（ODNI）、情报高级研究计划局（IARPA）通过 HIATUS 项目合同 #2022-22072200005 支持。此材料基于由国家科学基金会研究生研究奖学金资助的工作，资助编号为 DGE-2236662。文中观点和结论仅代表作者个人意见，不应解读为美国国家情报总监办公室、IARPA、国家科学基金会或美国政府的官方政策或观点，无论是明示还是暗示。尽管有任何版权声明，美国政府有权为政府目的复制和分发再版。

## 参考文献

+   Campos 等（2023）Nuno Campos、William FH、Vadym Barda 和 Harrison Chase. 2023. [LangGraph](https://github.com/langchain-ai/langgraph)。

+   Hong 等（2024）Sirui Hong、Mingchen Zhuge、Jonathan Chen、Xiawu Zheng、Yuheng Cheng、Jinlin Wang、Ceyao Zhang、Zili Wang、Steven Ka Shing Yau、Zijuan Lin、Liyang Zhou、Chenyu Ran、Lingfeng Xiao、Chenglin Wu 和 Jürgen Schmidhuber. 2024. [MetaGPT：面向多智能体协作框架的元编程](https://openreview.net/forum?id=VtmBAGCN7o)。发表于 *第十二届国际学习表征大会*。

+   Khot 等（2023）Tushar Khot、Harsh Trivedi、Matthew Finlayson、Yao Fu、Kyle Richardson、Peter Clark 和 Ashish Sabharwal. 2023. [分解提示：一种用于解决复杂任务的模块化方法](https://openreview.net/forum?id=_nGgzQjzaRy)。发表于 *第十一届国际学习表征大会*。

+   Lee 和 Kim（2023）Soochan Lee 和 Gunhee Kim. 2023. [思维的递归：一种基于分治的语言模型多情境推理方法](https://doi.org/10.18653/v1/2023.findings-acl.40)。发表于 *计算语言学协会发现：ACL 2023*，第623-658页，多伦多，加拿大。计算语言学协会。

+   Liu 等人 (2022) Jerry Liu, Logan 和 Simon Siu. 2022. [LlamaIndex](https://doi.org/10.5281/zenodo.1234)。

+   OpenAI (2022) OpenAI. 2022. [ChatGPT: 为对话优化语言模型](https://openai.com/blog/chatgpt)。

+   OpenAI (2024) OpenAI. 2024. [Hello GPT-4o](https://openai.com/index/hello-gpt-4o/)。

+   Parisi 等人 (2022) Aaron Parisi, Yao Zhao, 和 Noah Fiedel. 2022. [TALM: 工具增强语言模型](https://arxiv.org/abs/2205.12255)。*预印本*，arXiv:2205.12255。

+   Prasad 等人 (2024) Archiki Prasad, Alexander Koller, Mareike Hartmann, Peter Clark, Ashish Sabharwal, Mohit Bansal, 和 Tushar Khot. 2024. [ADaPT: 使用语言模型进行按需分解和规划](https://doi.org/10.18653/v1/2024.findings-naacl.264)。发表于 *计算语言学会发现：NAACL 2024*，第4226–4252页，墨西哥城，墨西哥。计算语言学会。

+   Qi 等人 (2023) Jingyuan Qi, Zhiyang Xu, Ying Shen, Minqian Liu, Di Jin, Qifan Wang, 和 Lifu Huang. 2023. [苏格拉底式提问的艺术：使用大语言模型进行递归思维](https://doi.org/10.18653/v1/2023.emnlp-main.255)。发表于 *2023年自然语言处理经验方法会议论文集*，第4177–4199页，新加坡。计算语言学会。

+   Qian 等人 (2024) Cheng Qian, Shihao Liang, Yujia Qin, Yining Ye, Xin Cong, Yankai Lin, Yesai Wu, Zhiyuan Liu, 和 Maosong Sun. 2024. [Investigate-consolidate-exploit: 一种任务间智能体自我进化的通用策略](https://arxiv.org/abs/2401.13996)。*预印本*，arXiv:2401.13996。

+   Qiao 等人 (2024) Shuofei Qiao, Ningyu Zhang, Runnan Fang, Yujie Luo, Wangchunshu Zhou, Yuchen Jiang, Chengfei Lv, 和 Huajun Chen. 2024. [AutoAct: 通过自我规划实现自动智能体从零开始的QA学习](https://doi.org/10.18653/v1/2024.acl-long.165)。发表于 *第62届计算语言学会年会（第1卷：长篇论文）论文集*，第3003–3021页，泰国曼谷。计算语言学会。

+   Schick 等人 (2023) Timo Schick, Jane Dwivedi-Yu, Roberto Dessi, Roberta Raileanu, Maria Lomeli, Eric Hambro, Luke Zettlemoyer, Nicola Cancedda, 和 Thomas Scialom. 2023. [Toolformer: 语言模型可以自学使用工具](https://openreview.net/forum?id=Yacmpz84TH)。发表于 *第三十七届神经信息处理系统会议*。

+   Significant Gravitas (2023) Significant Gravitas. 2023. [AutoGPT](https://github.com/Significant-Gravitas/AutoGPT)。

+   Wu 等人 (2023) Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Beibin Li, Erkang Zhu, Li Jiang, Xiaoyun Zhang, Shaokun Zhang, Jiale Liu, Ahmed Hassan Awadallah, Ryen W White, Doug Burger, 和 Chi Wang. 2023. [AutoGen: 通过多智能体对话实现下一代LLM应用](https://arxiv.org/abs/2308.08155)。*预印本*，arXiv:2308.08155。

+   XAgent 团队 (2023) XAgent 团队. 2023. Xagent: 一个用于复杂任务求解的自主智能体。

+   Xie 等人 (2024) 谢 Jian, Kai Zhang, Jiangjie Chen, Tinghui Zhu, Renze Lou, Yuandong Tian, Yanghua Xiao, 和 Yu Su. 2024. TravelPlanner：一种用于语言代理的现实世界规划基准。在 *第四十一届国际机器学习大会*。

+   Yuan 等人 (2024) Siyu Yuan, Kaitao Song, Jiangjie Chen, Xu Tan, Dongsheng Li, 和 Deqing Yang. 2024. [Evoagent: 通过进化算法实现自动化多代理生成](https://arxiv.org/abs/2406.14228)。 *预印本*，arXiv:2406.14228。

+   Zhang 等人 (2024) Ceyao Zhang, Kaijie Yang, Siyi Hu, Zihao Wang, Guanghe Li, Yihang Sun, Cheng Zhang, Zhaowei Zhang, Anji Liu, Song-Chun Zhu, Xiaojun Chang, Junge Zhang, Feng Yin, Yitao Liang, 和 Yaodong Yang. 2024. [Proagent: 使用大型语言模型构建主动合作代理](https://doi.org/10.1609/aaai.v38i16.29710)。 *人工智能领域的 AAAI 会议论文集*，38(16)：17591–17599。

+   Zhou 等人 (2024a) Shuyan Zhou, Frank F. Xu, Hao Zhu, Xuhui Zhou, Robert Lo, Abishek Sridhar, Xianyi Cheng, Tianyue Ou, Yonatan Bisk, Daniel Fried, Uri Alon, 和 Graham Neubig. 2024a. [Webarena: 一个用于构建自主代理的现实网络环境](https://openreview.net/forum?id=oKn9c6ytLx)。在 *第十二届国际学习表示大会*。

+   Zhou 等人 (2024b) Wangchunshu Zhou, Yixin Ou, Shengwei Ding, Long Li, Jialong Wu, Tiannan Wang, Jiamin Chen, Shuai Wang, Xiaohua Xu, Ningyu Zhang, Huajun Chen, 和 Yuchen Eleanor Jiang. 2024b. [符号学习使自我进化代理成为可能](https://arxiv.org/abs/2406.18532)。 *预印本*，arXiv:2406.18532。

+   Zhu 等人 (2023) Andrew Zhu, Liam Dugan, Alyssa Hwang, 和 Chris Callison-Burch. 2023. [Kani: 一个轻量级且高度可扩展的语言模型应用框架](https://doi.org/10.18653/v1/2023.nlposs-1.8)。在 *第三届自然语言处理开源软件研讨会 (NLP-OSS 2023)* 论文集，页码 65–77，新加坡。计算语言学协会。

+   Zhu 等人 (2024) Andrew Zhu, Alyssa Hwang, Liam Dugan, 和 Chris Callison-Burch. 2024. [FanOutQA: 一个多跳、多文档问题回答基准，适用于大型语言模型](https://doi.org/10.18653/v1/2024.acl-short.2)。在 *第62届计算语言学协会年会论文集（第二卷：短文）*，页码 18–37，泰国曼谷。计算语言学协会。

## 附录 A 自定义委托方案

以下注释代码片段展示了如何使用 ReDel Python 包定义一个委托方案——这里的委托方案是对打包的 DelegateOne 方案的再现。

{minted}

python class DelegateOne(DelegationBase): @ai_function() async def delegate(instructions: str): """(在此处插入模型的提示)"""

# 向系统请求一个新的代理实例 subagent = await self.create_delegate_kani(instructions)

# 将代理委托者的状态设置为等待代理的状态 `self.kani.run_state(RunState.WAITING)`：# 将代理的响应缓冲为字符串列表，过滤 ASSISTANT 消息 # 使用 full_round_stream，以便应用自动分派流事件 `result = []` 异步地从子代理获取流 `for stream in subagent.full_round_stream(instructions):` `msg = await stream.message()` `if msg.role == ChatRole.ASSISTANT and msg.content:` `result.append(msg.content)`

# 清理代理的临时状态并将结果返回给调用者 `await subagent.cleanup()` 返回 `"\n".join(result)`

图 7：使用 ReDel 定义自定义委托方案。委托工具负责管理它们创建的任何代理的生命周期。

## 附录 B 应用事件

下表列出了每次运行 ReDel 系统时会触发的内置默认事件。每个事件都有一个类型键，用于确定事件的类型，以及一个时间戳键。

| 事件名称 | 键 | 描述 |
| --- | --- | --- |
| 代理生成 | kani_spawn | 创建了一个新的代理。与事件相关的数据包含该代理生成时的完整状态，包括其 ID、与其他代理的关系、为其提供支持的 LLM 描述、它能访问的工具以及任何系统提示。 |
| 代理状态变化 | kani_state_change | 代理的运行状态发生了变化（例如，从 RUNNING 到 WAITING）。包含代理的 ID 以及其新状态。 |
| 使用的令牌 | tokens_used | 代理调用了为其提供支持的语言模型。包含代理的 ID、它发送的提示中令牌的数量以及 LLM 返回的完成中令牌的数量。 |
| 代理消息 | kani_message | 代理向其聊天历史中添加了一条新消息。包含代理的 ID、消息的角色（例如 USER 或 ASSISTANT）和内容。 |
| 根消息 | root_message | 类似于代理消息，但仅对根节点中的消息触发。除了触发代理消息事件外，还会触发此事件。 |
| 回合完成 | round_complete | 当根节点完成一个完整的聊天回合（即没有正在运行的子节点，并且它已生成对用户查询的响应）时触发。 |

表格 4：ReDel 工具包内置事件列表。

## 附录 C 基准比较

在这里，我们列出了在实验中测试的每个基准。

| 基准测试 | 分割 | # | 示例 | 指标 |
| --- | --- | --- | --- | --- |
| FanOutQA Zhu 等人 ([2024](https://arxiv.org/html/2408.02248v2#bib.bib23)) | dev | 310 | 世界五大银行的员工总数是多少？ | Loose: 生成的答案中找到的参考字符串的平均比例。Model Judge: 由 GPT-4 (gpt-4-0613) 判断参考答案与生成答案是否等价。 |

| TravelPlanner Xie 等人 ([2024](https://arxiv.org/html/2408.02248v2#bib.bib17)) | 验证 | 180 | 请帮我规划一趟从圣彼得堡到罗克福德的旅行，时间为 2022 年 3 月 16 日至 3 月 18 日，为期 3 天。旅行应为单人计划，预算为 1700 美元。 | CS-Micro：生成的旅行计划中没有常识性错误的元素比例（例如：同一景点重复游览）。H-Micro：生成的旅行计划中没有违反用户设置的约束或物理约束的元素比例（例如：超出预算、虚构餐厅）。

最终：在生成的旅行计划中，没有展示常识性错误且满足所有约束的比例（即有效的旅行计划）。

| WebArena Zhou 等人 ([2024a](https://arxiv.org/html/2408.02248v2#bib.bib20)) | 测试 | 271 | 给我展示评分最高的符合人体工学的椅子 | SR：任务是否成功完成或是否正确标记为不可达。SR (AC)：任务是否成功完成，仅适用于可达任务。

SR (UA)：是否正确标记任务为不可达，仅适用于不可达任务。

表 5：我们测试的每个基准数据集的划分、查询次数以及示例查询。

## 附录 D 额外设计说明

### D.1 提示

在本节中，我们提供了每个基准所使用的提示。我们为每个基准使用零-shot 提示，并提供在每个基准论文中定义的必要工具。

|  | 提示 |
| --- | --- |
| FanOutQA Zhu 等人 ([2024](https://arxiv.org/html/2408.02248v2#bib.bib23)) | 用户：{问题} |
| TravelPlanner Xie 等人 ([2024](https://arxiv.org/html/2408.02248v2#bib.bib17)) | 系统：根据用户的查询，为用户制定最佳旅行计划并保存。不要询问后续问题。用户：{问题} |

| WebArena Zhou 等人 ([2024a](https://arxiv.org/html/2408.02248v2#bib.bib20)) | 系统：你是一个自主智能代理，负责浏览网页。你将接受基于网页的任务。这些任务将通过你可以调用的特定功能来完成。以下是你将拥有的信息：

用户的目标：这是你试图完成的任务。

当前网页的可访问性树：这是网页的简化表示，提供了关键信息。

当前网页的 URL：这是你当前浏览的页面。

打开的标签：这些是你打开的标签。主页：如果你想访问其他网站，可以查看主页 http://homepage.com。它列出了你可以访问的网站。用户：浏览器状态：{观察}

URL：{url}

目标：{目标} |

表 6：我们评估中为每个基准所使用的提示。

### D.2 相同委托预防

默认情况下，ReDel 中捆绑的委托方案会阻止代理委托与其收到的指令相同的任务。如果代理尝试这样做，委托功能会返回一条消息，指示代理要么尝试自己完成任务，要么在再次委托之前将任务拆分成更小的部分。我们将其作为一种早期的应对措施来解决不充分承诺问题，但仍然会发生一些不充分承诺的情况。
