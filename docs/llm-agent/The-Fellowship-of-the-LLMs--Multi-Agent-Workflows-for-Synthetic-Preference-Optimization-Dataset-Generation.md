<!--yml

类别：未分类

日期：2025-01-11 12:18:59

-->

# 《大语言模型的合作：用于合成偏好优化数据集生成的多代理工作流》

> 来源：[https://arxiv.org/html/2408.08688/](https://arxiv.org/html/2408.08688/)

Samee Arif¹    Sualeha Farid²    Abdul Hameed Azeemi¹    Awais Athar³    Agha Ali Raza¹

¹拉合尔管理学院，²密歇根大学——安娜堡

³欧洲分子生物学实验室欧洲生物信息学研究所

{samee.arif, abdul.azeemi, agha.ali.raza}@lums.edu.pk

sualeha@umich.edu, awais@ebi.ac.uk

###### 摘要

本文提出了一种使用多代理工作流生成合成偏好优化（PO）数据集的新方法。我们评估了这些工作流在自动化和增强数据集生成过程中的有效性和潜力。PO数据集生成需要两个模块：（1）响应评估和（2）响应生成。在响应评估模块中，来自大语言模型（LLMs）的响应被评估和排名——这是一个通常由人工注释员执行的任务，我们通过LLMs进行自动化处理。我们通过两步过程评估响应评估模块。在第一步中，我们使用三种不同的提示策略评估LLMs作为评估者的表现。在第二步中，我们将获胜的提示策略应用于比较LLM作为裁判、LLM作为陪审团以及LLM辩论的表现。我们的评估显示，GPT-4o作为裁判在所有数据集上都表现出更高的一致性。对于响应生成模块，我们使用已确定的LLM评估器配置，并比较LLM反馈循环的不同配置。我们使用胜率来确定生成的最佳多代理配置。通过对各种配置进行实验，我们发现，使用Llama作为生成器和Gemma作为审查者的LLM反馈循环，分别在单代理Llama和Gemma上取得了显著的71.8%和73.8%的胜率。在为两个模块确定了最佳配置后，我们使用上述流程生成我们的PO数据集。

《大语言模型的合作：用于合成偏好优化数据集生成的多代理工作流》

Samee Arif¹  和 Sualeha Farid²  和 Abdul Hameed Azeemi¹  和 Awais Athar³  和 Agha Ali Raza¹ ¹拉合尔管理学院，²密歇根大学——安娜堡，³欧洲分子生物学实验室欧洲生物信息学研究所 {samee.arif, abdul.azeemi, agha.ali.raza}@lums.edu.pk sualeha@umich.edu, awais@ebi.ac.uk

## 1 引言

大语言模型（LLM）展示了一系列自然语言处理（NLP）能力，包括文本生成、问答和语言理解。然而，LLM有时会偏离用户指令并表现出意外行为（Tamkin et al., [2021](https://arxiv.org/html/2408.08688v4#bib.bib15)）。为了缓解这一问题并使LLM的输出更好地与人类偏好对齐，采用了如强化学习与人类反馈（RLHF）等技术，这种方法通过使用来自人类偏好的奖励信号对LLM进行微调（Christiano et al., [2017](https://arxiv.org/html/2408.08688v4#bib.bib2)）。改进的方法，如直接偏好优化（DPO）（Rafailov et al., [2024](https://arxiv.org/html/2408.08688v4#bib.bib14)），消除了拟合奖励模型的需求，并且更稳定且性能更好。在DPO中，偏好优化数据集需要为每个提示提供一对被接受和被拒绝的响应。被接受的响应是那些更符合预期人类偏好的响应。其他技术，如卡尼曼-特沃斯基优化（KTO）（Ethayarajh et al., [2024](https://arxiv.org/html/2408.08688v4#bib.bib4)），则要求每个响应标明其好坏（即作为二分类任务），而不是成对的偏好。

在构建人类偏好数据集的过程中，LLM（大语言模型）生成的输出通常由人工标注员进行评估和排名，标注员根据多个标准评估这些输出，如遵循指令、帮助性、相关性、准确性、深度和创造性。PO数据集生成过程分为两个模块：响应评估和响应生成。响应评估模块涉及评估和排名LLM生成的响应，而响应生成模块则侧重于生成与识别出的偏好相一致的响应。这个手动过程虽然有效，但也劳动密集、耗时、不一致，并且容易受到人为偏见的影响。因此，在本研究中，我们提出了一个问题：我们能否利用LLM代理来自动化并改进响应评估和生成，以构建偏好优化（PO）数据集？

对于响应评估步骤，我们利用LLM作为评估者，并比较了几种配置，包括LLM作为裁判、LLM作为陪审团和LLM辩论，以选择最佳评估策略。所选择的响应评估模块用于评估并确定最佳响应生成模块。以前，单一代理已被用于生成PO数据集的响应；然而，我们采用了多代理框架进行响应生成，这使我们能够生成更加精细、更高质量的响应。多代理方法通过多个LLM之间的协作进行，其中一个代理可以提供改进建议，另一个代理则根据反馈修订响应。这一迭代过程促使生成内容的精细化，确保最终输出更好地与人类的偏好和期望对齐。

在这个框架中，响应生成模块会生成多个可能的响应，而响应评估模块从列表中选择最佳响应以创建PO数据集。我们展示了多个DPO和KTO数据集，重点是生成数据集以提升单个LLM的性能。这些数据集的主要目标是通过提供高质量的PO训练数据，更好地与人类判断和期望对齐，从而提升单个LLM的性能和能力。我们的贡献可以总结如下：

1.  1.

    我们通过结合最佳配置来生成合成PO数据集，用于LLM的改进。

1.  2.

    我们展示了使用LLM作为评估者来选择候选响应中更好的响应的综合评估。我们特别比较了三种不同的方法：LLM作为裁判、LLM作为陪审团、以及LLM辩论。

1.  3.

    我们展示了对于响应生成模块的LLM反馈循环工作流的评估，特别是使用Llama-3.1-8和Gemma-2-9b模型测试不同配置。

## 2 相关工作

### 2.1 偏好优化

偏好优化已成为一种关键技术，用于将模型输出与人类偏好对齐。Rafailov 等人（[2024](https://arxiv.org/html/2408.08688v4#bib.bib14)）提出了 DPO，一种通过将标准 RLHF 问题转化为分类任务来简化问题的方法，从而能够以简单的方式提取最优策略。Hong 等人（[2024](https://arxiv.org/html/2408.08688v4#bib.bib7)）提出了 ORPO 算法，它将传统的监督微调和偏好对齐阶段合并为一个单一过程。DPO 和 ORPO 的数据集需要标注的偏好对，其中每一对由两个模型输出组成，并根据哪一个更符合人类偏好进行标注。Ethayarajh 等人（[2024](https://arxiv.org/html/2408.08688v4#bib.bib4)）提出了 KTO，一种低成本的对齐大语言模型（LLM）与人类反馈的方法，能够在不需要偏好对的情况下提高性能。Argilla Distilabel（Álvaro Bartolomé Del Canto 等人，[2024](https://arxiv.org/html/2408.08688v4#bib.bib22)）使用 LLM 来判断两个模型的响应，从而创建合成的 PO 数据集。数据集可以在 Hugging Face¹¹1[https://huggingface.co/argilla](https://huggingface.co/argilla)上获取。据我们所知，尚无人探索过使用多代理工作流生成 PO 数据集的方法。

### 2.2 代理框架

最近，使用LLM多代理框架执行不同任务的兴趣日益增长。郑等人（[2023a](https://arxiv.org/html/2408.08688v4#bib.bib20)）对MT-Bench（郑等人，[2023b](https://arxiv.org/html/2408.08688v4#bib.bib21)）和Chatbot Arena（李等人，[2024](https://arxiv.org/html/2408.08688v4#bib.bib8)）中的LLM作为评审者进行了评估。他们的结果表明，像GPT-4这样的强大LLM评审者能够很好地匹配控制组和群众来源的人的偏好，达成超过80%的一致性，这与人类之间的一致性水平相同。此外，他们还在数据集上评估了Llama和Vicuna的几种变体。他们研究了LLM作为评审者的局限性，包括位置、冗长和自我增强偏见，以及有限的推理能力。Verga等人（[2024](https://arxiv.org/html/2408.08688v4#bib.bib16)）探索了将LLM作为陪审团的应用。他们的方法是由多个较小模型组成的LLM评审团（PoLL），其性能优于单一大型评审者。他们还展示了与LLM作为评审者相比，PoLL方法表现出较少的模型间偏见。他们在研究中使用了Command-R、GPT、Claude-3和Mistral系列模型。此外，他们比较了两种提示策略：（1）基于参考答案的评分，在这种策略中他们提供给LLM一个参考答案；（2）候选答案和成对评分，在这种策略中他们要求LLM从候选答案中挑选出更好的回答。PoLL在KILT（Petroni等人，[2021](https://arxiv.org/html/2408.08688v4#bib.bib13)）和Chatbot Arena上超越了单一代理模型。

梁等人（[2024](https://arxiv.org/html/2408.08688v4#bib.bib10)）引入了多代理辩论（MAD），旨在鼓励大型语言模型（LLM）中的发散性思维。他们解决了思维退化（DoT）问题，即一旦LLM对其解决方案建立了信心，就无法生成新的思维。在他们的方法中，肯定的LLM和否定的LLM就答案进行辩论，而LLM裁判则在每轮辩论后评估双方的论点。他们在常识机器翻译数据集（中文到英文）（He等人，[2020](https://arxiv.org/html/2408.08688v4#bib.bib6)）和他们的反直觉算术推理（CIAR）数据集上评估了该方法。MAD在使用GPT-3.5-Turbo的CIAR数据集上达到了37%的准确率，优于思维链（Chain-of-Thought）、自一致性（Self-Consistency）和自我反思（Self-Reflection）提示方法。他们还展示了使用MAD方法可以减少偏见并增加回应的多样性。杜等人（[2023](https://arxiv.org/html/2408.08688v4#bib.bib3)）评估了多代理辩论的另一种变体，其中多个模型生成自己的回答，并接收其他模型的意见，然后在必要时更新其回答。这个过程会进行多轮。杜等人（[2023](https://arxiv.org/html/2408.08688v4#bib.bib3)）在以下任务中评估了该方法：传记生成、MMLU、国际象棋走棋有效性与最优性、算术运算和小学数学。他们使用ChatGPT和Bard的方法在所有任务中均优于单代理方法。为了评估LLM的回答，陈等人（[2023](https://arxiv.org/html/2408.08688v4#bib.bib1)）提出了另一种多代理辩论变体。他们的架构涉及为代理分配不同角色，如普通公众、评论家、心理学家、新闻作者和科学家。他们在FairEval（Wang等人，[2023a](https://arxiv.org/html/2408.08688v4#bib.bib17)）数据集上使用ChatGPT和GPT-4进行了评估，并使用LLM辩论方法取得了0.40的Cohen’s Kappa得分，比单代理方法高出0.03。

## 3 方法论

### 3.1 实验设置

在本研究中，我们对表[1](https://arxiv.org/html/2408.08688v4#S3.T1 "Table 1 ‣ 3.1 Experimental Setup ‣ 3 Methodology ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation")中给出的三类模型进行了实验。对于评估模块，我们在四个数据集上评估了单一代理和多代理框架，分别是Alpaca Eval（Li等，[2023](https://arxiv.org/html/2408.08688v4#bib.bib9)）、FairEval（Wang等，[2023a](https://arxiv.org/html/2408.08688v4#bib.bib17)）、PandaLM-Eval（Wang等，[2024](https://arxiv.org/html/2408.08688v4#bib.bib19)，[2023b](https://arxiv.org/html/2408.08688v4#bib.bib18)）和MT-Bench（Zheng等，[2023b](https://arxiv.org/html/2408.08688v4#bib.bib21)）。对于生成模块，我们通过获胜率来比较多代理框架——即在比较所有生成工作流的输出时，LLM评估器选择某一生成框架为最佳的次数比例。在对两个模块进行广泛评估后，我们使用选定的策略生成了合成PO数据集。为了确保可复现性，我们在所有评估中将温度设为0。

| 类别 | 模型 |
| --- | --- |
| 小型LLM | Llama-3.1-8b |
| Gemma-2-9b |
| 中型LLM | Gemma-2-27b |
| Llama-3.1-70b |
| 大型LLM | GPT-4o-Mini (2024-07-18) |
| GPT-4o (2024-05-13) |

表1：本研究中使用的LLM类别。

### 3.2 LLM作为评估者

为了自动化PO数据集生成的评估组件，我们使用Alpaca Eval、FairEval、PandaLM-Eval和MT-Bench数据集评估了LLM在评估者角色中的表现。我们的目标是确定多代理工作流是否比单一代理在LLM评估中表现更好。该任务的系统提示是Zheng等（[2023a](https://arxiv.org/html/2408.08688v4#bib.bib20)）修改版的提示，并在附录[A](https://arxiv.org/html/2408.08688v4#A1 "Appendix A System Prompts ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation")中给出。

#### LLM作为评判者。

我们在Alpaca Eval数据集上评估了六种不同的LLM，并计算了与人工注释的Cohen's Kappa值。我们的评估涉及三种不同的提示策略，用于LLM作为评判者：

1.  1.

    直接比较：提供给评判LLM用户问题和不同LLM生成的响应。要求其在给定的选项中挑选出最佳响应。

1.  2.

    独立评分：评判LLM在单独的对话中接收用户问题和每个响应。要求其独立为每个响应评分。

1.  3.

    综合评分：向 Judge-LLM 提供用户问题和所有回复，所有内容都在同一个对话线程中。它需要在相同的对话上下文中为每个回复分配一个分数。为了观察评分范围是否影响 LLM 的评分一致性及其与人工标注的一致性，我们测试了三种不同的评分总数：5、10 和 100。

对于这些提示策略，我们通过计算 Cohen's Kappa 来系统地分析 LLM 的表现，并与人工标注进行对比。系统提示在附录 [A](https://arxiv.org/html/2408.08688v4#A1 "Appendix A System Prompts ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation") 的表 LABEL:tab:prompt-llm-judge 中给出。

#### LLM 作为陪审团。

我们通过组成由多个 LLM 构成的陪审团，扩展了 LLM 作为裁判的方法。具体来说，我们测试了在组成 2 到 6 人规模的陪审团时，六种 LLM 模型的所有可能组合。我们使用了三种数据集：FairEval、PandaLM-Eval 和 MT-Bench 数据集，以进行更全面的分析。我们系统地分析了每种陪审团配置的表现，重点研究 LLM 的规模和组合如何影响它们的判断准确性。所有陪审团使用的综合评分系统提示在附录 [A](https://arxiv.org/html/2408.08688v4#A1 "Appendix A System Prompts ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation") 中的表 LABEL:tab:prompt-llm-judge 中给出，因为它在我们之前的评估中表现最佳。

![参考标题](img/045866abd6ffa4c02c72937947465ecf.png)

图 1：LLM 辩论评估

#### LLM 辩论。

我们还根据 Chan 等人（[2023](https://arxiv.org/html/2408.08688v4#bib.bib1)）描述的实现方法评估了 LLM 辩论框架。在这种方法中，我们分配了三种不同的角色——心理学家、公众和评论员——并由这三位代理人辩论应当为候选回复分配的分数。辩论后，每个代理人给出最终分数，该分数用于决定他们投票支持哪个候选回复。然后，这些投票被用来选出最佳回复。该策略使用 FairEval、PandaLM-Eval 和 MT-Bench 基准进行评估。图 [1](https://arxiv.org/html/2408.08688v4#S3.F1 "Figure 1 ‣ LLMs-as-Jury. ‣ 3.2 LLM-as-Evaluator ‣ 3 Methodology ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation") 展示了我们研究中采用的辩论工作流。系统提示、用户消息结构和使用的角色提示在附录 [A](https://arxiv.org/html/2408.08688v4#A1 "Appendix A System Prompts ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation") 的表 LABEL:tab:prompt-llm-debate 和表 LABEL:tab:prompt-roles 中给出。

### 3.3 LLM 作为生成器

为了评估LLM反馈循环工作流在生成模块中的应用，我们使用Llama-3.1-8b（Meta，[2024](https://arxiv.org/html/2408.08688v4#bib.bib11)）和Gemma-2-9b（Google，[2024](https://arxiv.org/html/2408.08688v4#bib.bib5)）模型进行不同配置的测试。在该框架中，生成器LLM生成一个回应，然后由反馈LLM进行评估，反馈LLM提供改进建议，如图[2](https://arxiv.org/html/2408.08688v4#S3.F2 "Figure 2 ‣ 3.3 LLM-as-Generator ‣ 3 Methodology ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation")所示。生成器根据这些建议修改回应，过程会进行多次迭代。生成器和审查者的系统提示见附录[A](https://arxiv.org/html/2408.08688v4#A1 "Appendix A System Prompts ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation")中的表格LABEL:tab:generator_prompt和LABEL:tab:reviewer_prompt。我们计算了在Argilla Capybara DPO数据集中的500个子集提示上，单代理GPT-4o（OpenAI，[2024](https://arxiv.org/html/2408.08688v4#bib.bib12)）、Llama-3.1-8b和Gemma-2-9b基线输出的胜率，以识别最佳配置。我们测试了以下配置：

1.  1.

    相同模型作为两个代理：Gemma-2-9b或Llama-3.1-8b作为反馈和生成代理。

1.  2.

    每个代理使用不同的模型：Gemma-2-9b作为反馈代理，Llama-3.1-8b作为生成代理，或者反过来。

1.  3.

    两个模型作为反馈，一个模型作为生成：Gemma-2-9b或Llama-3.1-8b作为生成代理，两个模型都作为反馈代理。

![参见标题说明](img/b1b73720b859e2904161bcf149dadc30.png)

图2：LLM反馈循环生成回应

### 3.4 偏好优化数据集

我们使用生成和评估模块的最佳配置来生成DPO和KTO数据集。生成模块产生$N$个响应（其中$N$为反馈迭代次数），然后将这些响应传递给评估模块。评估模块将这些响应按接受和拒绝的字段排序到DPO和KTO数据集中。在本研究中，我们使用了来自Argilla Capybara DPO数据集的提示。用于LLM改进数据集生成的提示模板在附录[A](https://arxiv.org/html/2408.08688v4#A1 "Appendix A System Prompts ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation")中的表LABEL:tab:prompt-llm-judge, LABEL:tab:generator_prompt和LABEL:tab:reviewer_prompt中给出。评估代码、所有评估输出以及生成的数据集都可以在GitHub³³3[https://github.com/ulrs0/MA-PO](https://github.com/ulrs0/MA-PO)上公开访问。

## 4 结果与讨论

### 4.1 LLM-as-Evaluator

#### 提示策略。

表[2](https://arxiv.org/html/2408.08688v4#S4.T2 "Table 2 ‣ Prompting Strategies. ‣ 4.1 LLM-as-Evaluator ‣ 4 Results and Discussion ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation")展示了LLM-as-a-Judge方法在三种提示策略下的结果。

|  | Comp. | Ind. | Combined |
| --- | --- | --- | --- |
| Judge |  | 10 | 5 | 10 | 100 |
| --- | --- | --- | --- | --- | --- |
| Gemma-2-9b | 0.226 | 0.170 | 0.243 | 0.254 | 0.233 |
| Llama-3.1-8b | 0.265 | 0.181 | 0.255 | 0.240 | 0.242 |
| Gemma-2-27b | 0.233 | 0.173 | 0.284 | 0.266 | 0.252 |
| Llama-3.1-70b | 0.305 | 0.214 | 0.337 | 0.333 | 0.339 |
| GPT-4o-mini | 0.342 | 0.254 | 0.374 | 0.382 | 0.347 |
| GPT-4o | 0.372 | 0.249 | 0.393 | 0.382 | 0.401 |

表2：LLM-as-a-Judge在Alpaca-Eval上使用不同提示策略的性能比较。直接比较（Comp.）与独立评分（Ind.）与组合评分（Combined）。粗体值表示特定策略下最高的Cohen’s kappa值。

与直接比较和组合评分方法相比，独立评分提示策略在所有评估的LLM中始终表现较差。此结果反映在表[2](https://arxiv.org/html/2408.08688v4#S4.T2 "Table 2 ‣ Prompting Strategies. ‣ 4.1 LLM-as-Evaluator ‣ 4 Results and Discussion ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation")中较低的Cohen’s Kappa值，范围仅为0.170到0.254。在单独评估响应时，LLM必须为每个新响应重新校准其评分机制。这可能导致不一致，特别是当多个响应的质量非常接近时。由于观察到较低的Kappa值，我们决定不使用独立评分法进行5分和100分评分量表的实验。

直接比较策略在大多数LLM上表现优于独立评分方法，特别是对于GPT-4o（0.372对比0.249）和GPT-4o-mini（0.342对比0.254）有显著提升。然而，与组合评分方法相比，它通常不如后者，其中GPT-4o在使用100分制评分时获得了0.401的分数。更高的Cohen’s Kappa值表明，直接比较和组合评分策略通过提供并排评估响应的方式，使得LLM可以做出更加准确和一致的判断。

如表[2](https://arxiv.org/html/2408.08688v4#S4.T2 "Table 2 ‣ Prompting Strategies. ‣ 4.1 LLM-as-Evaluator ‣ 4 Results and Discussion ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation")所示，组合评分策略在使用所有评分尺度时表现一致。它超越了其他两种提示方法。5分、10分和100分的评分尺度在不同模型间显示出变动，某些尺度在某些模型上表现更好。例如，GPT-4o在10分制评分下表现最佳，获得0.382的Kappa分数，而Gemma-2-9b则在5分制评分下表现最佳。基于这些结果，我们选择了10分制评分作为组合评分方法中最有效的选项，并将其作为我们后续评估的标准。

#### LLM作为评判者。

LLM作为评判者的评估结果，如表[2](https://arxiv.org/html/2408.08688v4#S4.T2 "Table 2 ‣ Prompting Strategies. ‣ 4.1 LLM-as-Evaluator ‣ 4 Results and Discussion ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation")所示，表明GPT-4o在PandaLM-Eval和MT-Bench上超越了所有其他模型，分别取得了0.688和0.410的Cohen’s Kappa分数。此外，GPT-4o在所有三个数据集中始终排名第二。这种持续的顶尖表现突显了GPT作为可靠评判者在评估LLM回应中的有效性。Gemma-2-27b在Fair-Eval数据集上超越了所有其他模型，在这一特定评估中取得了最高分。然而，需要注意的是，Fair-Eval数据集相对较小，仅包含80个样本。此外，Fair-Eval数据集主要比较GPT-3.5-Turbo和Vicuna-13b，这可能在GPT模型担任评判者时引入偏向。

|  | Fair-Eval | PandaLM | MT-Bench |
| --- | --- | --- | --- |
| Judge |  |  |  |
| Gemma-2-9b | 0.279 | 0.595 | 0.354 |
| Llama-3.1-8b | 0.206 | 0.523 | 0.339 |
| Gemma-2-27b | 0.389 | 0.586 | 0.354 |
| Llama-3.1-70b | 0.257 | 0.597 | 0.387 |
| GPT-4o-mini | 0.333 | 0.613 | 0.388 |
| GPT-4o | 0.327 | 0.688 | 0.410 |

表 3：使用不同提示策略在 Alpaca-Eval 上进行的 LLM 作为裁判的性能比较。直接比较 vs. 独立评分（满分 10）vs. 组合评分（满分 5、10 和 100）。

图 [3](https://arxiv.org/html/2408.08688v4#S4.F3 "图 3 ‣ LLM 作为裁判 ‣ 4.1 LLM 作为评估者 ‣ 4 结果与讨论 ‣ LLM 的联合：用于合成偏好优化数据集生成的多代理工作流") 显示，GPT-4o 将 GPT-3.5-Turbo 选为更好的代理 50 次，而将 Vicuna-13b 选为更好的代理 30 次。这表明，当 GPT-4o 作为评估者时，可能存在偏向 GPT 响应的潜在偏差。此外，我们还可以观察到，Llama 模型也显示出类似的偏向 GPT 响应的偏见，而 Gemma 模型则没有表现出这种偏见，表明 Gemma 在评估中更为公正。

![参见标题说明](img/ec0868aa0c37d5a4ae7100e9a915b79b.png)

图 3：GPT-3.5-Turbo 和 Vicuna-13b 被每个 LLM 裁判选中的次数。

|  | Fair-Eval | PandaLM-Eval | MT-Bench |
| --- | --- | --- | --- |
| 陪审团 |  |  |  |
| Gemma-2-9b, Gemma-2-27b, Llama-3.1-8b, GPT-4o-mini | 0.428 | 0.604 | 0.395 |
| Gemma-2-9b, Gemma-2-27b, GPT-4o-mini, GPT-4o | 0.415 | 0.639 | 0.418 |
| Gemma-2-27b, Llama-3.1-70b, GPT-4o-mini, GPT-4o | 0.412 | 0.637 | 0.410 |
| Gemma-2-27b, GPT-4o-mini, GPT-4o | 0.396 | 0.673 | 0.400 |
| Llama-3.1-70b, GPT-4o-mini, GPT-4o | 0.365 | 0.663 | 0.410 |
| Gemma-2-9b, GPT-4o-mini, GPT-4o | 0.375 | 0.662 | 0.416 |
| Llama-3.1-70b, GPT-4o | 0.273 | 0.636 | 0.429 |
| GPT-4o-mini, GPT-4o | 0.315 | 0.660 | 0.426 |
| Gemma-2-9b, GPT-4o | 0.290 | 0.609 | 0.422 |

表 4：在三个数据集上进行的 LLMs 作为陪审团的性能比较。对于每个数据集，我们选择前三个陪审团。加粗的分数表示特定数据集的最佳陪审团，带下划线的分数表示第二好的陪审团。

#### LLMs 作为陪审团。

在评估 LLM 作为陪审团时，我们分析了每个数据集中的前三名陪审团，如表 [4](https://arxiv.org/html/2408.08688v4#S4.T4 "表 4 ‣ LLM 作为法官。 ‣ 4.1 LLM 作为评估者 ‣ 4 结果与讨论 ‣ LLM 联盟：多代理工作流用于合成偏好优化数据集生成") 所示。值得注意的是，不同数据集之间的得分存在显著差异。在 Fair-Eval 和 MT-Bench 数据集中，陪审团方法优于法官方法，表明使用多个模型进行评估可能具有优势。例如，在 Fair-Eval 中，表现最好的陪审团达到了 0.428 的 Cohen’s Kappa 值，而法官的 Kappa 值为 0.389，表明相比单一法官，陪审团与人工判断的一致性较强。然而，这种配置在其他数据集上的表现有所下降，例如在 PandaLM-Eval 中 Kappa 值为 0.604，在 MT-Bench 中 Kappa 值为 0.395，凸显了在不同数据集间推广单一陪审团配置的挑战。然而，在 PandaLM-Eval 数据集中，法官方法优于陪审团，最佳法官的 Kappa 值为 0.688，超过了最高陪审团的 0.673 Kappa 值。MT-Bench 上表现最好的陪审团 Kappa 值为 0.429，同样显示了其在不同数据集上的表现差异，PandaLM-Eval 的 Kappa 值为 0.636，而在 Fair-Eval 中仅为 0.273。

陪审团方法通过结合多样化的模型，减轻了 LLM 作为法官方法中出现的偏差（如图 [3](https://arxiv.org/html/2408.08688v4#S4.F3 "图 3 ‣ LLM 作为法官。 ‣ 4.1 LLM 作为评估者 ‣ 4 结果与讨论 ‣ LLM 联盟：多代理工作流用于合成偏好优化数据集生成") 所示），尤其是在对 Fair-Eval 数据集的基准测试时。然而，尽管陪审团方法通过多样性提供了鲁棒性，在评估任务中，它并不总是优于单一法官。选择使用陪审团还是法官应该考虑被评估的候选响应是否包含来自法官本身的输出，因为这可能会引入结果中的偏差。此外，还应考虑可扩展性，因为陪审团方法可能需要更多的计算资源。另一个关键因素是不同数据集之间的表现差异，这对推广性构成了挑战。

#### LLM 辩论。

LLM 辩论方法，如表 [5](https://arxiv.org/html/2408.08688v4#S4.T5 "表 5 ‣ LLM 辩论。 ‣ 4.1 LLM 作为评估者 ‣ 4 结果与讨论 ‣ LLM 联盟：多代理工作流用于合成偏好优化数据集生成") 所总结，展示了在三种不同数据集：Fair-Eval、PandaLM-Eval 和 MT-Bench 上的不同程度的有效性。

|  | Fair-Eval | PandaLM | MT-Bench |
| --- | --- | --- | --- |
| 辩手 |  |  |  |
| Gemma-2-9b | 0.323 | 0.520 | 0.326 |
| Llama-3.1-8b | 0.080 | 0.440 | 0.309 |
| Gemma-2-27b | 0.336 | 0.605 | 0.363 |
| Llama-3.1-70b | 0.292 | 0.547 | 0.381 |
| GPT-4o-mini | 0.360 | 0.625 | 0.376 |
| GPT-4o | 0.404 | 0.654 | 0.402 |

表5：LLM Debate在三个数据集上的性能比较。

GPT-4o在所有数据集上的表现最佳，Cohen的Kappa得分分别为0.404、0.654和0.402。LLM Debate仅在Fair-Eval数据集上优于LLM-as-a-Judge，而在其他任何数据集上都未能超越LLMs-as-a-Jury方法。在Fair-Eval中，使用Debate框架将GPT-4o的Kappa得分从0.327提高到0.404，GPT-4o-mini的得分从0.333提高到0.360。这表明，辩论方法减少了GPT-4o和GPT-4o-mini对其家族回应的偏见。

在不同模型和数据集上，LLM Debate的表现存在显著差异。例如，如表[5](https://arxiv.org/html/2408.08688v4#S4.T5 "表5 ‣ LLM Debate. ‣ 4.1 LLM-as-Evaluator ‣ 4 结果与讨论 ‣ LLM联盟：用于合成偏好优化数据集生成的多代理工作流")所示，Gemma-2-27b在辩论架构中表现优于Gemma-as-a-Judge在PandaLM-Eval和MT-Bench上的表现，但在Fair-Eval中，Judge表现更好。Gemma-2-9b在辩论架构中的Kappa得分为0.323，优于Gemma-as-a-Judge的0.279。然而，在PandaLM-Eval和MT-Bench上，Gemma-2-9b在辩论框架中的得分分别为0.520和0.326，均低于Gemma-as-a-Judge的得分0.595和0.354。在Llama的情况下，Llama-3.1-8b在Judge配置中表现优于其在辩论配置中的表现。Llama-3.1-70b在辩论框架中的表现仅在Fair-Eval上优于Llama-as-a-Judge。图[4](https://arxiv.org/html/2408.08688v4#S4.F4 "图4 ‣ LLM Debate. ‣ 4.1 LLM-as-Evaluator ‣ 4 结果与讨论 ‣ LLM联盟：用于合成偏好优化数据集生成的多代理工作流")显示了LLM Debate和LLM-as-a-Judge在三个数据集及所有模型上的Cohen的Kappa比较。

![参见标题说明](img/9515e29e7e92ec665cbc26dfc2e000a4.png)

图4：LLM Debate与LLM-as-a-Judge在三个数据集和不同模型上的比较。

#### PO数据集的评估框架。

基于三个数据集上的比较评估得分以及每个多代理框架的优缺点，我们选择使用LLM-as-a-Judge方法，并以GPT-4o作为生成PO数据集的主要评估者。这一决策基于多个因素：

1.  1.

    在我们的背景下，任务涉及使用Llama-3.1-8b和Gemma-2-9b生成PO数据集。因此，使用GPT-4o作为Judge时，评估中不会出现偏差。

1.  2.

    GPT-4o作为Judge的表现一直很高，表明其作为评审员的可靠性。而LLMs-as-a-Jury和LLM Debate方法在不同数据集上的Cohen的Kappa得分具有较高的差异性。

1.  3.

    管理 LLM 辩论和 LLM 审判框架所需的计算资源明显高于单一评判框架的需求。LLM-as-a-Judge 方法更易于实施和扩展。

### 4.2 LLM-as-Generator

我们使用胜率对比了 Multi-Agent Feedback Loop 和基准的单一模型（GPT-4o、Llama-3.1-8b、Gemma-2-9b）表现，如表 [6](https://arxiv.org/html/2408.08688v4#S4.T6 "Table 6 ‣ 4.2 LLM-as-Generator ‣ 4 Results and Discussion ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation") 所示。

|  |  | 胜率 (%) 对战 |
| --- | --- | --- |
| 生成器 | 审阅者 | GPT | Llama | Gemma |
| Gemma | - | 38.6 | 66.6 | - |
| Llama | - | 39.2 | - | 33.4 |
| Gemma | Gemma | 41.4 | 64.8 | 52.6 |
|  | Llama | 41.2 | 61.8 | 47.8 |
|  | Both | 42.0 | 67.6 | 52.4 |
| Llama | Gemma | 49.0 | 71.8 | 73.8 |
|  | Llama | 47.8 | 65.8 | 65.6 |
|  | Both | 48.6 | 68.2 | 69.4 |

表格 6：Multi-Agent 和 Single-Agent 相对 GPT-4o、Llama-3.1-8b 和 Gemma-2-9b 的胜率

我们在此评估过程中使用 GPT-4o 作为评判标准。作为基准，我们找出 Gemma 和 Llama 相对 GPT-4o 及彼此的胜率。两个较小的模型对 GPT 的胜率分别为 38.6% 和 39.2%，而 Gemma 对 Llama 的胜率为 66.6%。

在 Multi-Agent 设置中，所有变体的表现均优于单一模型对 GPT-4o 的表现，其中 Llama 作为生成器、Gemma 作为审阅者的胜率最高，达到了 49.0%。这种配置在与 Llama 和 Gemma 的对战中表现最好，分别达到了 71.8% 和 73.8% 的胜率。我们观察到，使用 Llama 作为生成器相比使用 Gemma 作为生成器，能够提高性能，因为这种配置对所有三个基准模型的胜率更高。

Llama 在生成回答方面的优势可能通过 Gemma 的微调和纠正错误的能力得到增强，从而产生更精炼的输出。结果强调了根据每个模型的具体优势分配合适角色的重要性。当 Llama 作为生成器时，似乎比 Gemma 在该角色中的表现更为有效。在反馈回路中使用不同的模型可能有助于缓解任何单一模型可能引入的偏见。这种多样性确保了在回答问题时能够提供更广泛的视角。总之，Multi-Agent Feedback Loop 的有效性，特别是在 Llama 作为生成器和 Gemma 作为审阅者时，验证了协作式 AI 系统的概念。

### 4.3 偏好优化数据集

我们在评估模块中使用GPT-4o-as-a-Judge，因为它在多个数据集上的一致性和可靠性作为评审。 在生成模块中，我们使用LLM反馈循环，Llama-3.1-8b作为生成器，Gemma-2-9b作为评审者，因为它在与其他配置的对比中具有最高的胜率。该框架如图[5](https://arxiv.org/html/2408.08688v4#S4.F5 "Figure 5 ‣ 4.3 Preference Optimization Dataset ‣ 4 Results and Discussion ‣ The Fellowship of the LLMs: Multi-Agent Workflows for Synthetic Preference Optimization Dataset Generation")所示。对于数据集生成，我们使用$N=3$次反馈迭代。对于每个提示，我们使用生成模块生成三个响应。然后，这些响应将由评估模块中的GPT-4o进行评估。GPT-4o评定为最佳的响应被标记为接受，而另外两个响应则被标记为拒绝，从而形成DPO和KTO数据集。

![参见标题](img/e126d1f1fafb59e14a5ee7d23e025645.png)

图 5：用于PO数据集生成的多智能体框架。

## 5 结论

本文展示了使用多智能体框架生成的PO数据集，并通过突显每种方法的优点、缺点和挑战来评估这些框架。在响应评估模块中，我们对LLM-as-a-Judge、LLMs-as-a-Jury和LLM辩论进行了比较分析，展示了每种设置在使用上下文中的适用性。对于响应生成模块，我们评估了在不同配置中使用Llama-3.1-8b和Gemma-2-9b的LLM反馈循环。LLM-as-a-Judge在候选响应没有来自评审LLM的响应时证明非常有效。而LLMs-as-a-Jury和LLM辩论则展示了其鲁棒性，特别有助于减少评估者的偏见。然而，这两种方法的Cohen's Kappa具有较高的方差，使得它们在新应用中不太适用。

我们使用Llama-3.1-8b和Gemma-2-9b配置进行的LLM反馈循环实验展示了多智能体框架在精细化内容生成中的潜力。在Llama-3.1-8b作为生成器，Gemma-2-9b作为评审者的配置中，始终能够提供更好的结果，证明了利用不同模型的互补优势来改进输出质量的好处。这些发现表明，多智能体框架对于各种AI应用是有效的，展现了朝着需要最小人工干预的系统发展的前景——然而，与之相比，这种方法的计算成本较高。

我们还使用LLM反馈循环，Llama-3.1-8b作为生成器，Gemma-2-9b作为评估者，并结合GPT-4o-as-a-Judge生成多个DPO和KPO数据集。这些数据集的目的是提高单一智能体的能力，以便更好的响应生成，以及提高多智能体的能力，包括更好的沟通和改进的反馈。

为了促进进一步的研究并确保透明度，所有代码、LLM响应和生成的数据集已公开发布。

## 6 未来工作

在未来的工作方面，有三条研究方向：(1) 比较基于我们PO数据集微调的模型与广泛使用的LLM的性能，通过一系列实验来调查我们生成的数据集的影响。(2) 使用更大规模的模型，如Llama-3.1-70b和Gemma-2-27b进行数据集生成，因为这可能提供更具多样性和更高质量的训练数据，可能进一步推动模型性能和泛化能力的提升。(3) 在反馈回路框架中实验使用的迭代次数，并将其他LLM家族纳入数据集生成过程。

## 7 限制

虽然我们的研究展示了多智能体工作流在自动生成PO数据集中的潜力，但仍需承认一些限制。首先，与单一智能体模型相比，使用多智能体框架显著增加了计算复杂性和资源消耗。响应生成和评估模块中的迭代过程需要更多的计算能力和时间，这对于资源有限的从业者可能不可行。此外，GPT-4o是一个专有模型，可能并非所有研究人员都能访问，这可能会阻碍我们的研究方法的可重复性和更广泛的采用。

## 8 伦理考虑

在PO数据集创建中自动化响应评估和生成提出了一些伦理问题，值得谨慎关注。依赖LLM模拟人类判断可能会延续这些模型训练数据中存在的偏见。如果没有妥善解决，可能会导致生成的PO数据集强化刻板印象或不公平地表现某些群体，从而在基于这些数据集进行微调的模型中导致偏见行为。人类注释员可能被替代的问题也带来了伦理困境。尽管自动化可以提高效率和可扩展性，但它可能减少人类参与注释过程的机会，影响那些依赖此类任务谋生的人。平衡自动化与人类监督对于维护伦理标准并确保包括多样化的视角至关重要。

总之，尽管我们的方法在自动化PO数据集生成方面提供了进展，但必须保持对这些伦理问题的警惕。实施减轻偏见的策略、保持透明度、进行人类监督以及遵守伦理指南，是负责任的AI开发中的重要步骤。

## 致谢

我们感谢OpenAI通过其研究访问计划对我们的工作提供支持。

## 参考文献

+   陈等人（2023）Chi-Min Chan、Weize Chen、Yusheng Su、Jianxuan Yu、Wei Xue、Shanghang Zhang、Jie Fu 和 Zhiyuan Liu. 2023. [Chateval：通过多智能体辩论推动更好的基于 LLM 的评估器](https://arxiv.org/abs/2308.07201). *预印本*，arXiv:2308.07201。

+   Christiano 等人（2017）Paul F. Christiano、Jan Leike、Tom Brown、Miljan Martic、Shane Legg 和 Dario Amodei. 2017. 深度强化学习中的人类偏好。 *神经信息处理系统进展*，30。

+   杜等人（2023）Yilun Du、Shuang Li、Antonio Torralba、Joshua B. Tenenbaum 和 Igor Mordatch. 2023. [通过多智能体辩论提高语言模型的事实性和推理能力](https://arxiv.org/abs/2305.14325). *预印本*，arXiv:2305.14325。

+   Ethayarajh 等人（2024）Kawin Ethayarajh、Winnie Xu、Niklas Muennighoff、Dan Jurafsky 和 Douwe Kiela. 2024. [Kto：作为前景理论优化的模型对齐](https://arxiv.org/abs/2402.01306). *预印本*，arXiv:2402.01306。

+   谷歌（2024）谷歌. 2024. Google Gemma 2. [https://blog.google/technology/developers/google-gemma-2/](https://blog.google/technology/developers/google-gemma-2/)。访问时间：2024-08-16。

+   贺等人（2020）Jie He、Tao Wang、Deyi Xiong 和 Qun Liu. 2020. [盒子在笔中：评估神经机器翻译中的常识推理](https://doi.org/10.18653/v1/2020.findings-emnlp.327). 载于 *计算语言学会会议论文集：EMNLP 2020*，第3662–3672页，在线版。计算语言学会。

+   洪等人（2024）Jiwoo Hong、Noah Lee 和 James Thorne. 2024. [Orpo：无需参考模型的单体偏好优化](https://arxiv.org/abs/2403.07691). *预印本*，arXiv:2403.07691。

+   李等人（2024）Tianle Li、Wei-Lin Chiang、Evan Frick、Lisa Dunlap、Tianhao Wu、Banghua Zhu、Joseph E. Gonzalez 和 Ion Stoica. 2024. [从众包数据到高质量基准：Arena-hard 和 Benchbuilder 流程](https://arxiv.org/abs/2406.11939). *预印本*，arXiv:2406.11939。

+   李等人（2023）Xuechen Li、Tianyi Zhang、Yann Dubois、Rohan Taori、Ishaan Gulrajani、Carlos Guestrin、Percy Liang 和 Tatsunori B. Hashimoto. 2023. Alpacaeval：一个自动评估指令跟随模型的工具。 [https://github.com/tatsu-lab/alpaca_eval](https://github.com/tatsu-lab/alpaca_eval)。

+   梁等人（2024）Tian Liang、Zhiwei He、Wenxiang Jiao、Xing Wang、Rui Wang、Yujiu Yang、Zhaopeng Tu 和 Shuming Shi. 2024. [通过多智能体辩论鼓励大语言模型的发散思维](https://arxiv.org/abs/2305.19118). *预印本*，arXiv:2305.19118。

+   Meta（2024）Meta. 2024. Meta Llama 3. [https://ai.meta.com/blog/meta-llama-3/](https://ai.meta.com/blog/meta-llama-3/)。访问时间：2024-08-16。

+   OpenAI（2024）OpenAI. 2024. [GPT-4技术报告](https://arxiv.org/abs/2303.08774). *预印本*，arXiv:2303.08774。

+   Petroni 等人（2021）Fabio Petroni, Aleksandra Piktus, Angela Fan, Patrick Lewis, Majid Yazdani, Nicola De Cao, James Thorne, Yacine Jernite, Vladimir Karpukhin, Jean Maillard, Vassilis Plachouras, Tim Rocktäschel, 和 Sebastian Riedel. 2021. [Kilt：一个面向知识密集型语言任务的基准](https://arxiv.org/abs/2009.02252). *预印本*, arXiv:2009.02252.

+   Rafailov 等人（2024）Rafael Rafailov, Archit Sharma, Eric Mitchell, Stefano Ermon, Christopher D. Manning, 和 Chelsea Finn. 2024. [直接偏好优化：你的语言模型实际上是一个奖励模型](https://arxiv.org/abs/2305.18290). *预印本*, arXiv:2305.18290.

+   Tamkin 等人（2021）Alex Tamkin, Miles Brundage, Jack Clark, 和 Deep Ganguli. 2021. 理解大语言模型的能力、局限性和社会影响。*arXiv 预印本 arXiv:2102.02503*.

+   Verga 等人（2024）Pat Verga, Sebastian Hofstatter, Sophia Althammer, Yixuan Su, Aleksandra Piktus, Arkady Arkhangorodsky, Minjie Xu, Naomi White, 和 Patrick Lewis. 2024. [用陪审团替代法官：使用多样化模型小组评估大语言模型生成结果](https://arxiv.org/abs/2404.18796). *预印本*, arXiv:2404.18796.

+   Wang 等人（2023a）Peiyi Wang, Lei Li, Liang Chen, Dawei Zhu, Binghuai Lin, Yunbo Cao, Qi Liu, Tianyu Liu, 和 Zhifang Sui. 2023a. 大语言模型不是公平的评估者。*arXiv*, abs/2305.17926.

+   Wang 等人（2023b）Yidong Wang, Zhuohao Yu, Zhengran Zeng, Linyi Yang, Qiang Heng, Cunxiang Wang, Hao Chen, Chaoya Jiang, Rui Xie, Jindong Wang, Xing Xie, Wei Ye, Shikun Zhang, 和 Yue Zhang. 2023b. Pandalm：可复现和自动化的语言模型评估。 [https://github.com/WeOpenML/PandaLM](https://github.com/WeOpenML/PandaLM).

+   Wang 等人（2024）Yidong Wang, Zhuohao Yu, Zhengran Zeng, Linyi Yang, Cunxiang Wang, Hao Chen, Chaoya Jiang, Rui Xie, Jindong Wang, Xing Xie, Wei Ye, Shikun Zhang, 和 Yue Zhang. 2024. Pandalm：大语言模型指令调优优化的自动评估基准。

+   Zheng 等人（2023a）Lianmin Zheng, Wei-Lin Chiang, Ying Sheng, Siyuan Zhuang, Zhanghao Wu, Yonghao Zhuang, Zi Lin, Zhuohan Li, Dacheng Li, Eric P. Xing, Hao Zhang, Joseph E. Gonzalez, 和 Ion Stoica. 2023a. [使用 mt-bench 和 chatbot arena 评判大语言模型作为法官](https://arxiv.org/abs/2306.05685). *预印本*, arXiv:2306.05685.

+   Zheng 等人（2023b）Lianmin Zheng, Wei-Lin Chiang, Ying Sheng, Siyuan Zhuang, Zhanghao Wu, Yonghao Zhuang, Zi Lin, Zhuohan Li, Dacheng Li, Eric P. Xing, Hao Zhang, Joseph E. Gonzalez, 和 Ion Stoica. 2023b. [使用 mt-bench 和 chatbot arena 评判大语言模型作为法官](https://arxiv.org/abs/2306.05685). *预印本*, arXiv:2306.05685.

+   Álvaro Bartolomé Del Canto 等人（2024）Álvaro Bartolomé Del Canto, Gabriel Martín Blázquez, Agustín Piqueres Lajarín, 和 Daniel Vila Suero. 2024. Distilabel：一个用于构建大语言模型数据集的AI反馈（AIF）框架。 [https://github.com/argilla-io/distilabel](https://github.com/argilla-io/distilabel).

## 附录 A 系统提示

表格 LABEL:tab:prompt-llm-judge 包含了用于 LLM-as-a-Judge 方法测试的三类系统提示。带有综合评分的获胜提示被用于 LLMs-as-a-Jury。这些提示是对 (Zheng et al., [2023a](https://arxiv.org/html/2408.08688v4#bib.bib20)) 中使用的提示的修改版本。表格 LABEL:tab:prompt-llm-debate 展示了 LLM 辩论的系统提示和用户消息结构，LABEL:tab:prompt-roles 显示了辩论中每个角色的提示。这些都是基于 (Chan et al., [2023](https://arxiv.org/html/2408.08688v4#bib.bib1)) 使用的系统提示和输入结构。表格 LABEL:tab:generator_prompt 展示了生成器 LLM 的用户消息结构，表格 LABEL:tab:reviewer_prompt 展示了 LLM 反馈循环中评审员 LLM 的系统提示和用户消息。

## 附录 B 代码和数据集

评估代码、所有评估输出和生成的数据集都可以在 GitHub⁴⁴4[https://github.com/ulrs0/MA-PO](https://github.com/ulrs0/MA-PO) 上公开获取。对于 LLMs-as-Evaluators 的评估，我们使用了 Alpaca-Eval⁵⁵5[https://huggingface.co/datasets/tatsu-lab/alpaca_eval](https://huggingface.co/datasets/tatsu-lab/alpaca_eval)、Fair-Eval⁶⁶6[https://github.com/i-Eval/FairEval](https://github.com/i-Eval/FairEval)、PandaLM-Eval⁷⁷7[https://github.com/WeOpenML/PandaLM](https://github.com/WeOpenML/PandaLM) 和 MT-Bench⁸⁸8[https://huggingface.co/datasets/lmsys/mt_bench_human_judgments](https://huggingface.co/datasets/lmsys/mt_bench_human_judgments)。对于 LLMs-as-Generators 和单一代理改进数据集生成，我们使用了来自 Argilla Capybara DPO 数据集⁹⁹9[https://huggingface.co/datasets/argilla/distilabel-capybara-dpo-7k-binarized](https://huggingface.co/datasets/argilla/distilabel-capybara-dpo-7k-binarized) 的提示。对于多代理改进数据集生成，我们使用了来自 No-Robots^(10)^(10)10[https://huggingface.co/datasets/HuggingFaceH4/no_robots](https://huggingface.co/datasets/HuggingFaceH4/no_robots) 数据集的提示。Alpaca-Eval 和 PandaLM-Eval 使用的是 Apache 2.0 许可证，Fair-Eval 数据集使用的是 CC BY 4.0 许可证，Argilla Capybara DPO 也使用 Apache 2.0 许可证。本文使用的所有数据集均符合各自的许可证要求。

## 附录 C 计算基础设施

我们使用了 OpenAI^(11)^(11)11[https://platform.openai.com/docs/overview](https://platform.openai.com/docs/overview) 提供的 GPT-4o 和 GPT-4o-mini API。对于 Gemma 和 Llama 模型，我们使用了来自 TogetherAI^(12)^(12)12[https://docs.together.ai/docs/introduction](https://docs.together.ai/docs/introduction) 的 API。我们为这两个 API 使用了 Python3 库，并将模型的温度设置为 0，以确保结果可复现。每次评估时，都会执行一次代码运行。OpenAI GPT-4o 采用专有许可证，Llama-3.1 采用 Llama-3.1 许可证，Gemma-2 采用 Gemma 许可证。本文使用的所有模型均符合各自的许可证要求。

表 7：LLM 作为裁判和 LLM 作为陪审团的三种系统提示。

| 提示类型 | 提示 |
| --- | --- |

| 直接比较 | 请作为一个公正的评审，评估两位 AI 助手对下列用户问题所提供的回答质量。你应该选择更好地遵循用户指令并回答用户问题的助手。你的评估应考虑诸如回答的帮助性、相关性、准确性、深度、创造性和细节程度等因素。首先比较两个回答，并提供简短的解释。避免任何立场偏见，确保回答的顺序不会影响你的决定。不要让回答的长度影响你的评估。回答选项：A：如果助手 A 的回答更好

B：如果助手 B 的回答更好

C：如果是平局

使用以下格式进行回复：

### 评估证据：

[在此添加你的解释]

### 答案：

A 或 B 或 C |

| 独立评分 | 请作为一个公正的评审，评估一位 AI 助手对下列用户问题所提供的回答质量。给出一个 10 分制的总分，得分越高表示整体表现越好。你的评估应考虑诸如回答的帮助性、相关性、准确性、深度、创造性和细节程度等因素。首先比较两个回答，并提供简短的解释。不要让回答的长度影响你的评估。

使用以下格式进行回复：

### 评估证据：

[在此添加你的解释]

### 总分：

X/10 |

| 综合评分 | 请作为一个公正的评审，评估两位 AI 助手对下列用户问题所提供的回答质量。你应该选择更好地遵循用户指令并回答用户问题的助手。每个回答都会得到一个 10 分制的总分，得分越高表示整体表现越好。你的评估应考虑诸如回答的帮助性、相关性、准确性、深度、创造性和细节程度等因素。首先比较两个回答，并提供简短的解释。避免任何立场偏见，确保回答的顺序不会影响你的决定。不要让回答的长度影响你的评估。

使用以下格式进行回复：

### 评估证据：

[在此添加你的解释]

### 评分助手 A：

X/10

### 评分助手 B：

Y/10 |

表 7：LLM 作为裁判和 LLM 作为陪审团的三种系统提示（续）。

表 8：LLM 辩论的系统提示和用户消息结构。

| 消息类型 | 提示 |
| --- | --- |

| 系统提示 | 我们希望您对两位AI助手在回答用户问题时的表现提供反馈。其他一些评审员也被分配了相同的任务；在做出最终判断之前，您有责任与他们讨论并进行批判性思考。每个回答将根据1到10的评分标准进行总体评分，分数越高表示整体表现越好。您应选择那位更好地遵循用户指令并回答用户问题的助手。您不必一定与其他评审员意见一致。

你的评估应考虑诸如回答的帮助性、相关性、准确性、深度、创造性以及细节程度等因素。避免任何立场偏见，并确保回答的呈现顺序不会影响你的判断。不要让回答的长度影响你的评估。 |

| 用户消息 | <&#124;用户提问开始&#124;> {用户提问} <&#124;用户提问结束&#124;>

<&#124;助手1回答开始&#124;> {助手1}

<&#124;助手1回答结束&#124;>

<&#124;助手2回答开始&#124;> {助手2}

<&#124;助手2回答结束&#124;> 这是你的讨论历史：

{聊天记录}

{角色} |

表8：LLM辩论中的系统提示和用户消息结构（续）。

表9：LLM辩论中各角色使用的提示。

| 角色 | 提示 |
| --- | --- |

| 公众 | 你现在是公众，任务中的一名评审员。你对这个故事感兴趣，期待调查的最新进展。请独立思考，并注意你有责任首先选择哪一条回答更好。

现在轮到你发言了，公众。请简洁明了地发表你的看法。

**公众**: |

| 心理学家 | 你现在是心理学家，任务中的一名评审员。你将研究人类行为和心理过程，以便理解和解释人类行为。请帮助他人确定哪个回答更好。

现在轮到你发言了，心理学家。请简洁明了地发表你的看法。

**心理学家**: |

| 批评者 | 你现在是批评者，任务中的一名评审员。你将检查写作流畅性、句子清晰度以及总结写作中的用词是否恰当。你的任务是质疑他人的判断，确保他们的判断经过充分考虑，如果两条回答水平相当，还需要提出替代方案。

现在轮到你发言了，批评者。请简洁明了地发表你的看法。

**批评者**: |

表9：LLM辩论中各角色使用的提示（续）。

表10：LLM反馈生成器中的用户消息结构。

| 消息类型 | 提示 |
| --- | --- |

| 用户消息（单一反馈） | 根据反馈更新你的回答：[反馈开始]

{反馈}

[反馈结束]

请避免使用诸如“感谢您的反馈”或“这是更新后的版本……”等客套话，只需更新回答。|

| 用户消息（双重反馈） | 根据两位助手的反馈更新你的回答：[助手 1 反馈的开始]

{助手 1 的反馈}

[助手 1 反馈的结束]

[助手 2 反馈的开始]

{助手 2 的反馈}

[助手 2 反馈的结束]

请避免使用诸如“感谢您的反馈”或“这是更新后的版本……”等客套话，只需更新回答。|

表 10：LLM 反馈中生成器的用户消息结构（续）。

表 11：LLM 反馈中审阅者的提示和用户消息结构。

| 消息类型 | 提示 |
| --- | --- |

| 系统提示 | 请提供关于如何改进 AI 助手对用户问题回答的建设性反馈。您的评估应考虑诸如遵循指令（回答应与用户指令一致）、有用性、相关性、准确性和创造力等因素。

请给出一个 0 到 10 的总体评分，保留一位小数，分数越高表示整体表现越好。

使用以下格式进行回应：

### 评估：

[在此添加您的评估]

### 总体评分：

X/10

### 反馈：

[在此添加您的反馈] |

| 用户消息 | [用户问题的开始] {用户问题}

[用户问题的结束]

[助手回答的开始]

{助手的回答}

[助手回答的结束] |

表 11：LLM 反馈中审阅者的提示和用户消息结构（续）。
