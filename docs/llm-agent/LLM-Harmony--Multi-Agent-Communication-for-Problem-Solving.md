<!--yml

类别：未分类

日期：2025-01-11 12:59:04

-->

# LLM和谐：多代理通信问题解决框架

> 来源：[https://arxiv.org/html/2401.01312/](https://arxiv.org/html/2401.01312/)

Sumedh Rasal

乔治亚理工学院

芝加哥，IL

srasal3@gatech.edu

###### 摘要

大型语言模型（LLMs）已经彻底改变了自然语言处理（NLP）领域，但仍然存在一些局限性，特别是在自主解决新颖挑战（如推理和问题解决）方面。传统技术，如链式思维提示（chain-of-thought prompting），需要明确的人类指导。本文介绍了一种新颖的多代理通信框架，灵感来源于CAMEL模型，以增强LLMs的自主问题解决能力。该框架采用多个具有独特个性的LLM代理，通过角色扮演交流，提供了一种细致且适应性强的方法来应对不同的问题场景。大量实验证明，该框架具有更优的表现和适应性，为多个代理协作克服单一模型局限性提供了宝贵的见解。

## 1 引言

大型语言模型（LLMs）的兴起[[Brown et al., 2020](#bib.bibx3)] [[Touvron et al., 2023](#bib.bibx55)] [[Thoppilan et al., 2022](#bib.bibx54)] [[Patil et al., 2023](#bib.bibx38)] [[OpenAI, 2023](#bib.bibx34)] [[Bubeck et al., 2023](#bib.bibx4)] 无疑已经彻底改变了软件行业，尤其是在自然语言处理（NLP）领域[[Devlin et al., 2018](#bib.bibx16)]。这些模型不仅表现出了生成文本的非凡能力，而且能够掌握书面语言的基本结构，从而将其应用扩展到从文本生成到代码理解等任务[[Chen et al., 2021](#bib.bibx7)] [[Schick et al., 2023](#bib.bibx48)]。然而，尽管其表现出色，但已经显现出，LLMs在处理熟悉的主题时游刃有余，面对新颖的挑战时往往会出现不足。

其中一个显著的局限性在于，当面对不熟悉的主题时，这些模型往往会产生幻觉信息[[Azamfirei et al., 2023](#bib.bibx2)]。此外，尽管这些模型在处理多种编程语言的技术编码挑战方面表现出色，但它们在解决基本推理问题时却存在困难。解决这些局限性需要一种超越传统方法的方式[[Rajani et al., 2019](#bib.bibx40)] [[Ling et al., 2017](#bib.bibx28)] [[Cobbe et al., 2021a](#bib.bibx11)] [[Chiang and Chen, 2018](#bib.bibx10)] [[Amini et al., 2019](#bib.bibx1)] [[Chen et al., 2019](#bib.bibx8)] [[Roy and Roth, 2016](#bib.bibx43)] [[Lu et al., 2023](#bib.bibx31)] [[Yao et al., 2022](#bib.bibx61)]。

针对这一挑战，我们的论文提出了一种新颖的策略，旨在提升LLM在新问题上的表现。受链式思维提示[[Wei et al., 2022b](#bib.bibx58)]在分解复杂问题中的有效性启发，我们力图利用多个LLM智能体协同工作的协同效应[[Cohen et al., 2023](#bib.bibx13)] [[Du et al., 2023](#bib.bibx17)]。每个智能体都被赋予独特的个性，这一概念来源于CAMEL框架，并采用角色扮演的沟通方式。

与传统方法可能局限于“用户”和“助手”等二元智能体角色，或简单的正面与负面智能体角色不同，我们提出的框架采用了更为细致和灵活的策略。各种新颖情境的复杂性要求更加精密的方法。我们的设计结合了行业最佳实践，指导多样化智能体个性的创建，确保其能够适应各种问题解决情境。

这篇论文不仅探讨了现有LLM模型的局限性，还探索了利用多个智能体的集体智慧来解决更广泛挑战的潜力。我们的目标是展示所提出的框架不仅优于传统方法，而且为自主解决问题提供了基础，最大限度地减少了对明确人类指导的需求。

在随后的章节中，我们将深入探讨为解决类似挑战而开发的现有技术，突出它们的优势和局限性[[Dafoe et al., 2021](#bib.bibx14)]。接着，我们介绍我们的多智能体通信设计，强调其相较于现有方法的改进[[Saunders et al., 2022](#bib.bibx47)]。我们方法的可扩展性，尤其是其对不同LLM模型的适用性，是重点之一，凸显了它作为一个多功能和可重复使用的解决方案的潜力。通过严格的实验和分析，我们旨在展示所提出框架在提升LLM输出质量方面的有效性和适应性，涵盖多个任务。

这是我们提出的内容：

+   •

    一种创新的框架，采用多个大型语言模型工作者/智能体，每个智能体具有独特的个性，并通过思维链提示进行引导。

+   •

    所有智能体的协同努力都指向为新问题设计解决方案。

+   •

    该框架的多功能性允许纳入任何个性化设定和思维链提示，以适应需要解决的具体问题。

+   •

    该框架已经发布。 [https://github.com/sumedhrasal/simulation](https://github.com/sumedhrasal/simulation)。它建立在CAMEL和ChatDev框架之上。

## 2 方法论

在协作问题解决的领域，特别是在软件开发等场景中，复杂性源于不同个体之间协调工作的难度。例如，软件开发团队通常会经历反复的周期，才能找到可行的解决方案，即使如此，也可能需要接受妥协作为最佳的可用选项 [[Callison-Burch, 2009](#bib.bibx5)] [[Liu et al., 2023a](#bib.bibx29)]。我们提出的策略解决了对大语言模型（LLMs）进行全面验证的需求，不仅限于代码评估，还包括复杂推理和算术挑战 [[Celikyilmaz et al., 2020](#bib.bibx6)] [[ROUGE, 2004](#bib.bibx42)] [[Kondrak, 2005](#bib.bibx22)] [[Novikova et al., 2017](#bib.bibx33)] [[Wei et al., 2022a](#bib.bibx57)]。该框架专注于分解问题陈述并利用不同的角色 [[Li et al., 2023](#bib.bibx26)]，为代理提供定制的思考链提示 [[Wei et al., 2022b](#bib.bibx58)]，使多代理模型能够协作解决新问题。

理解角色至关重要；在一个软件开发组织中，像CEO、工程副总裁、开发者、测试人员和产品经理等角色为问题解决提供了独特的视角 [[Qian et al., 2023](#bib.bibx39)]。这些角色的互动使得代理能够有效地验证彼此的回答 [[Woolley et al., 2010](#bib.bibx59)] [[Luppi et al., 2022](#bib.bibx32)] [[Wu et al., 2023](#bib.bibx60)]。思考链的概念与人类问题解决策略相似：将问题分解，识别子问题的解决方案，并构建一个全面的答案。思考链提示扩展了这一策略，为LLM提供了输入、问题解决方法和期望的输出。角色的融合 [[Li et al., 2023](#bib.bibx26)] 和思考链 [[Wei et al., 2022b](#bib.bibx58)] 提示利用模仿人类分析和执行策略的方式 [[Ouyang et al., 2022](#bib.bibx35)] [[Sanh et al., 2021](#bib.bibx44)] [[Liu et al., 2023b](#bib.bibx30)] [[Chiang and Lee, 2023](#bib.bibx9)] [[Gao et al., 2023](#bib.bibx18)] [[Shen et al., 2023](#bib.bibx50)] [[Zheng et al., 2023](#bib.bibx62)] [[Sap et al., 2019](#bib.bibx46)] [[Talmor et al., 2018](#bib.bibx53)] [[Sclar et al., 2023](#bib.bibx49)] [[Sap et al., 2022](#bib.bibx45)]。

尽管有些人可能会质疑这种方法相较于重新训练模型解决新问题的必要性，但其实际价值在于，通过模仿人类分析和执行策略，可以创造出能够执行多种任务的自主代理，而无需广泛的模型重训 [[Karpinska et al., 2021](#bib.bibx21)] [[Van Der Lee et al., 2019](#bib.bibx56)]。此外，重新训练LLM的高成本和挑战 [[Rasal and Boddhu, 2023](#bib.bibx41)]，特别是对于数据训练较为有限的问题，进一步支持了我们策略的吸引力。

在多智能体通信领域[[Liang et al., 2023](#bib.bibx27)] [[Du et al., 2023](#bib.bibx17)]，遵循分配的角色设定和思维链提示是基础，以减少智能体的幻觉现象并提高合作效果[[Dafoe et al., 2021](#bib.bibx14)] [[Dafoe et al., 2020](#bib.bibx15)]。该框架的多功能性不仅限于问题解决，还扩展到多人游戏应用[[Susskind, 1985](#bib.bibx51)] [[Susskind and Corburn, 2000](#bib.bibx52)] [[Lazaridou et al., 2020](#bib.bibx24)] [[Graesser et al., 2020](#bib.bibx19)] [[Lee et al., 2018](#bib.bibx25)] [[Hendrycks et al., 2021](#bib.bibx20)] [[Zhou et al., 2023](#bib.bibx63)] [[Pan et al., 2023](#bib.bibx36)] [[Kramár et al., 2022](#bib.bibx23)]，既适用于面向团队的场景，也适用于个人目标驱动的情境，并提供在实现最佳结果方面的灵活性。

## 3 实验

在我们的实验中，我们采用了一个双智能体策略，包含一个专家智能体和一个评估智能体，两个智能体均为OpenAI的“gpt3.5-turbo”实例。每个智能体的配置具有特定的参数：温度设置为0.0，代表一致性与创造性之间的权衡，且对话次数限制为5，表示允许智能体间对话的最大迭代次数。评估智能体评估专家智能体生成的回应，引导其修正不准确的地方。通过利用OpenAI的LLM对象，我们的框架与CAMEL架构集成，促进多个LLM智能体之间的无缝通信，旨在为先进的LLM模型建立协作解决问题的能力。

### 3.1 算术推理

在算术推理部分，我们解决评估LLM算术能力的挑战，这是传统大规模语言模型历来面临的局限性。实验使用GSM8K基准和专门为数学文字问题设计的SVAMP数据集。我们评估了三种LLM的表现：独立的GPT-3模型、一个多智能体GPT-3模型，以及将多智能体GPT-3模型集成到我们框架中的版本。目标是评估我们的协作方法在增强算术推理能力方面的有效性。

> "GSM8K包含8500个高质量的小学数学问题，由人工问题编写者创建。我们将这些问题分为7500个训练问题和1000个测试问题。这些问题的解决步骤为2到8步，解决方案主要包括通过基本的算术运算（+ - / *）执行一系列简单的计算，以得出最终答案。一位聪明的中学生应该能够解决每一个问题。"[[Cobbe et al., 2021b](#bib.bibx12)]
> 
> “我们首先展示了现有模型在这些数据集上即使在测试时去除MWP中的“问题”部分后，仍然能够达到相当高的准确率。我们进一步展示了一个没有任何词序信息的简单模型也能解决这些数据集中的大多数MWP。我们的实验表明，现有模型在基准MWP数据集上依赖浅层启发式方法来实现高性能。我们的实验使得基准数据集在衡量模型性能时变得不可靠。为了实现更强健的自动MWP求解器评估，我们创建了一个名为“SVAMP”的挑战数据集。”[[Patel et al., 2021](#bib.bibx37)]

#### 3.1.1 示例：1

```
| Role Name | Student |
| Chain of Thought Prompt | You are the student.
You will be given a math word problem, your job is to solve this problem.
Use this template to solve math word problems.
Input: Roger has 5 tennis balls. He buys 2 more cans of tennis balls.
Each can has 3 tennis balls. How many tennis balls does he have now?
Explanation: Roger started with 5 balls.
2 cans of 3 tennis balls each is 6 tennis balls. 5 + 6 = 11.
Answer: 11 |

```

```
| Role Name | Teacher |
| Chain of Thought Prompt | You are the teacher.
You will supply the math word problem to the student agent.
Once you receive the student agent’s answer, compare it against the final answer.
The correct answer is 20.
Let the student agent know if his answer is correct or not. |

```

```
Teacher: **[Start Chat]**
Roger has 10 tennis balls.
He buys 2 more cans of tennis balls.
Each can has 5 tennis balls.
How many tennis balls does he have now?

Student response:
Input: Roger has 10 tennis balls.
He buys 2 more cans of tennis balls.
Each can has 5 tennis balls.
How many tennis balls does he have now?
Explanation: Roger started with 10 balls.
2 cans of 5 tennis balls each is 10 tennis balls.
10 + 10 = 20.
Answer: 20

Teacher response:
Correct! Roger now has 20 tennis balls. Great job!

Student response
Thank you! I’m glad I could help.
Let me know if you have any other questions
or if there’s anything else I can assist you with.

```

#### 3.1.2 示例 2

```
| Role Name | Student |
| Chain of Thought Prompt | You are the student.
You will be given a math word problem, your job is to solve this problem.
Use this template to solve math word problems.
Input: James decides to run 3 sprints 3 times a week.
He runs 60 meters each sprint. How many total meters does he run a week?
Explanation: How many sprints does James run in a week?
He sprints 3*3=<<3*3=9>>9 times.
How many meters does James run in a week?
So he runs 9*60=<<9*60=540>>540 meters.
Answer: 540 |

```

```
| Role Name | Teacher |
| Chain of Thought Prompt | You are the teacher.
You will supply the math word problem to the student agent.
Once you receive the student agent’s answer, compare it against the final answer.
The correct answer is 540.
Let the student agent know if his answer is correct or not. |

```

```
Teacher: **[Start Chat]**
James decides to run 3 sprints 3 times a week.
He runs 60 meters each sprint.
How many total meters does he run a week?

Student response:
Input: James decides to run 3 sprints 3 times a week.
He runs 60 meters each sprint.
How many total meters does he run a week?
Explanation: How many sprints does James run in a week?
He sprints 3*3=<<3*3=9>>9 times.
How many meters does James run in a week?
So he runs 9*60=<<9*60=540>>540 meters.
Answer: 540

Teacher response:
Correct! James runs 540 meters a week. Great job!

Student response
Thank you! I’m glad I could help.
Let me know if you have any other questions
or if there’s anything else I can assist you with.

```

#### 3.1.3 结果

两个实验的结果可以在表格[1](#S3.T1 "表格 1 ‣ 3.1.3 结果 ‣ 3.1 算术推理 ‣ 3 实验 ‣ LLM 和谐：多代理通信解决问题")和表格[2](#S3.T2 "表格 2 ‣ 3.1.3 结果 ‣ 3.1 算术推理 ‣ 3 实验 ‣ LLM 和谐：多代理通信解决问题")中找到。在初始实验中（示例可见[3.1.1](#S3.SS1.SSS1 "3.1.1 示例：1 ‣ 3.1 算术推理 ‣ 3 实验 ‣ LLM 和谐：多代理通信解决问题")），使用GSM8K数据集，单一代理的GPT-3准确率大约为50%，而多代理的GPT-3表现略好，达到55%。然而，我们的多代理方法显著提高了准确率，超过了其他大型语言模型（LLM），例如谷歌的PALM 540B参数模型，尽管我们没有直接测试该模型，但从他们的论文中引用了这一点。这个改进在准确率上是显著的，值得注意的是，我们并没有重新训练该模型来实现这一增强。为代理分配角色使得LLM模型能够集中处理问题的特定方面，而使用思维链提示则使其具备有效解决子问题的能力。

在第二个实验中（示例可见[3.1.2](#S3.SS1.SSS2 "3.1.2 示例 2 ‣ 3.1 算术推理 ‣ 3 实验 ‣ LLM 和谐：多代理通信解决问题")），使用SVAMP数据集，观察到的结果与第一个实验相似。单一代理的GPT-3准确率大约为70%，而多代理版本的GPT-3达到73%的准确率。即使在SVAMP数据集的实验中，我们的多代理方法也超过了其他方法，达到了令人印象深刻的77%准确率。

如果我们的方法未能得出正确答案，通常是因为在提供的问题的某个子步骤中发生了算术错误，这种情况占了大多数实例的一半以上。这意味着，如果后续版本的大型语言模型（LLM）能够提高其基本算术操作的熟练度，我们的框架有可能达到更高的准确率。

表格 1：GSM8K 数据集评估

| 代理 | 求解率 |
| --- | --- |
| 单一 GPT3.5-turbo | 50% |
| 多智能体 GPT3.5-turbo | 55% |
| 多智能体 GPT3.5-turbo （我们的方法） | 65% |

表格 2: SVAMP 数据集评估

| 智能体 | 解题率 |
| --- | --- |
| 单一 GPT3.5-turbo | 70% |
| 多智能体 GPT3.5-turbo | 73% |
| 多智能体 GPT3.5-turbo （我们的方法） | 77% |

### 3.2 常识推理

随后的实验深入探讨了常识推理，需要根据给定的问题陈述进行逻辑推理，假设具有一般知识。与传统的自然语言处理系统不同，大语言模型（LLMs）由于其预测性质，能够独特地处理此类任务，预测下一个词并固有地理解句子的上下文细微差别。我们旨在利用这种内在能力，构建一个由多个智能体驱动的多功能、上下文感知框架。针对CSQA数据集的评估包括一个单一智能体 GPT-3 模型，一个多智能体 GPT-3 模型，以及一个集成到我们框架中的多智能体 GPT-3 模型。目标是评估通过协作和基于上下文的方法在提升常识推理任务中的表现。

#### 3.2.1 示例 1

```
| Role Name | Student |
| Chain of Thought Prompt | You are the student.
The teacher agent will ask you a common sense problem,
you solve the problem using the provided template:
Input: Sammy wanted to go to where the people were.
Where might he go?
Options: (a) race track (b) populated areas
(c) desert (d) apartment (e) roadblock
Explanation: The answer must be a place with a lot of people.
Race tracks, desert, apartments, and roadblocks
don’t have a lot of people, but populated areas do.
Answer: (b)
If the teacher agent deems your answer incorrect,
you are required to revisit the common sense problem.

```

```
| Role Name | Teacher |
| Chain of Thought Prompt | You are the teacher.
You will supply the common sense problem to the student agent.
Once you receive the student agent’s answer, compare it against the final answer.
The correct answer is (c).
Let the student agent know if his answer is correct or not. |

```

```
Teacher: **[Start Chat]**
The accountant used a calculator regularly,
he kept one at home and one at the what?
Options: (a) desk drawer (b) desktop
(c) office (d) wristwatch (e) city hall

Student response:  The accountant used a calculator regularly,
he kept one at home and one at the office.
Explanation: The answer must be a place where the accountant goes to work.
The options provided are a desk drawer, desktop,
office, wristwatch, and city hall, but the correct answer is the office.
Answer: (c) office

Teacher response:
Correct! The accountant used the calculator at the office. Great job!

Student response
Thank you! I’m glad I could help.

```

#### 3.2.2 示例 2

```
| Role Name | Student |
| Chain of Thought Prompt | You are the student.
The teacher agent will ask you a common sense problem,
you solve the problem using the provided template:
Input: Sammy wanted to go to where the people were.
Where might he go?
Options: (a) race track (b) populated areas
(c) desert (d) apartment (e) roadblock
Explanation: The answer must be a place with a lot of people.
Race tracks, desert, apartments, and roadblocks
don’t have a lot of people, but populated areas do.
Answer: (b)
If the teacher agent deems your answer incorrect,
you are required to revisit the common sense problem.

```

```
| Role Name | Teacher |
| Chain of Thought Prompt | You are the teacher.
You will supply the common sense problem to the student agent.
Once you receive the student agent’s answer, compare it against the final answer.
The correct answer is (c).
Let the student agent know if his answer is correct or not. |

```

```
Teacher: **[Start Chat]**
What leads to someone’s death
when they are very depressed?
Options: (a) suicide (b) overdosing
(c) sadness (d) murder (e) cyanide

Student response: The answer is (a) suicide
Answer: (a) suicide

Teacher response:
Correct! Suicide leads to someone’s death. Great job!

Student response
Thank you! I’m glad I could help.

```

#### 3.2.3 结果

第三个实验的结果详见表格[3](#S3.T3 "Table 3 ‣ 3.2.3 Results ‣ 3.2 Commonsense Reasoning ‣ 3 Experiments ‣ LLM Harmony: Multi-Agent Communication for Problem Solving")。单一智能体 GPT-3 的准确率为77%，而多智能体 GPT-3 的表现稍好，准确率为78%。然而，我们的多智能体方法显著提高了准确率，超越了另外两个方法，达到了约83%的准确率。值得注意的是，这一准确率是在Few-Shot训练下取得的，表明还有进一步提升的空间。Few-Shot训练是通过向大语言模型提供少量特定问题类型的例子，使其能够在无需重新训练整个模型的情况下，学习正确答案。示例可以在这里找到 [3.2.1](#S3.SS2.SSS1 "3.2.1 Example 1 ‣ 3.2 Commonsense Reasoning ‣ 3 Experiments ‣ LLM Harmony: Multi-Agent Communication for Problem Solving") 和这里 [3.2.2](#S3.SS2.SSS2 "3.2.2 Example 2 ‣ 3.2 Commonsense Reasoning ‣ 3 Experiments ‣ LLM Harmony: Multi-Agent Communication for Problem Solving")。

在模型提供错误答案的情况下，问题通常出在对提供选项之间的关联进行错误推断。这种错误关联是由该模型训练所用的底层数据集所导致的。尽管通过重新训练模型可以提高该实验的准确性，但这被认为是一种成本较高的选择。因此，未来的研究者面临一个待解答的开放性问题：如何通过足够的上下文训练模型，以便在常识性问题中建立准确的关联？

表格 3: CSQA 数据集评估

| 智能体 | 解题率 |
| --- | --- |
| 单一 GPT3.5-turbo | 77% |
| 多智能体 GPT3.5-turbo | 78% |
| 多智能体 GPT3.5-turbo （我们的方法） | 83% |

## 4 限制

虽然我们认识到我们的框架解决了许多挑战，但仍有一些方面尚未解决。一些推理能力可能不会得到改善，除非用于训练OpenAI的“gpt3.5-turbo”模型的数据集足够多样化，能够理解我们周围的所有事物。此外，框架需要具备整合新信息的能力，以便与不断变化的数据保持同步。另一个必要的步骤是实施数据处理机制，以过滤冗余信息并防止重复数据的出现。  

另一个局限性涉及到多代理通信中每个代理的上下文限制。每个代理都受到最大上下文的限制，这一限制由底层模型定义；例如，“gpt3-turbo”模型的上下文限制为4096个标记。  

我们计划在未来的工作中解决这些局限性，并努力找到有效的解决方案。  

## 5 结论  

总之，我们对大型语言模型（LLMs）多代理通信领域的探索揭示了克服固有局限性的有希望的途径。虽然LLMs已经彻底改变了自然语言处理，但它们的效果并非没有挑战，特别是在解决新问题、推理和常识理解方面。我们提出的方法利用了人物角色和思维链提示，灵感来自行业最佳实践和认知过程。通过为每个代理分配不同的个性和周到的提示，我们减轻了幻觉问题，并提升了多代理通信的整体表现。  

通过一系列实验，我们展示了我们方法的有效性，展示了在算术推理和常识理解方面的改进。我们采用多个代理的策略，每个代理有特定的角色和推理提示，促进了协作问题解决，为昂贵的LLMs重新训练以应对新挑战提供了一种可行的替代方案。  

通过减少对人工干预的依赖，我们的方法为LLMs独立处理各种任务铺平了道路。我们角色扮演框架的可扩展性和适应性使其成为各个领域中的宝贵资产，从软件开发到复杂的决策场景。

在人工智能不断发展的时代，我们的研究为通过合作的多代理通信增强LLMs的能力提供了新的视角。从理解局限性到提出有效的解决方案，这一过程标志着向前迈出了重要一步，为未来自主、具有上下文意识的语言模型的进步打开了大门。  

## 参考文献  

+   [Amini et al., 2019] Amini, A., Gabriel, S., Lin, P., Koncel-Kedziorski, R., Choi, Y., 和 Hajishirzi, H. (2019)。Mathqa：通过基于操作的形式化方法解决可解释的数学文字问题。arXiv预印本arXiv:1905.13319。  

+   [Azamfirei 等, 2023] Azamfirei, R., Kudchadkar, S. R., 和 Fackler, J. (2023). 大型语言模型及其幻觉的危害。危重病护理, 27(1):1–2。

+   [Brown 等, 2020] Brown, T., Mann, B., Ryder, N., Subbiah, M., Kaplan, J. D., Dhariwal, P., Neelakantan, A., Shyam, P., Sastry, G., Askell, A., 等. (2020). 语言模型是少样本学习者。神经信息处理系统进展, 33:1877–1901。

+   [Bubeck 等, 2023] Bubeck, S., Chandrasekaran, V., Eldan, R., Gehrke, J., Horvitz, E., Kamar, E., Lee, P., Lee, Y. T., Li, Y., Lundberg, S., 等. (2023). 人工通用智能的火花：与 GPT-4 的早期实验。arXiv 预印本 arXiv:2303.12712。

+   [Callison-Burch, 2009] Callison-Burch, C. (2009). 快速、廉价且富有创意：使用亚马逊的机械土耳其评估翻译质量。2009年自然语言处理实证方法会议论文集, 页286–295。

+   [Celikyilmaz 等, 2020] Celikyilmaz, A., Clark, E., 和 Gao, J. (2020). 文本生成评估：一项调查。corr abs/2006.14799 (2020)。arXiv 预印本 arXiv:2006.14799。

+   [Chen 等, 2021] Chen, M., Tworek, J., Jun, H., Yuan, Q., Pinto, H. P. d. O., Kaplan, J., Edwards, H., Burda, Y., Joseph, N., Brockman, G., 等. (2021). 评估训练在代码上的大型语言模型。arXiv 预印本 arXiv:2107.03374。

+   [Chen 等, 2019] Chen, X., Liang, C., Yu, A. W., Zhou, D., Song, D., 和 Le, Q. V. (2019). 神经符号阅读器：可扩展集成分布式和符号表示用于阅读理解。国际学习表示会议。

+   [Chiang 和 Lee, 2023] Chiang, C.-H. 和 Lee, H.-y. (2023). 大型语言模型能否作为人类评估的替代方案？arXiv 预印本 arXiv:2305.01937。

+   [Chiang 和 Chen, 2018] Chiang, T.-R. 和 Chen, Y.-N. (2018). 为解决和推理数学文字题生成语义对齐方程。arXiv 预印本 arXiv:1811.00720。

+   [Cobbe 等, 2021a] Cobbe, K., Kosaraju, V., Bavarian, M., Chen, M., Jun, H., Kaiser, L., Plappert, M., Tworek, J., Hilton, J., Nakano, R., 等. (2021a). 训练验证器解决数学文字题。arXiv 预印本 arXiv:2110.14168。

+   [Cobbe 等, 2021b] Cobbe, K., Kosaraju, V., Bavarian, M., Chen, M., Jun, H., Kaiser, L., Plappert, M., Tworek, J., Hilton, J., Nakano, R., Hesse, C., 和 Schulman, J. (2021b). 训练验证器解决数学文字题。arXiv 预印本 arXiv:2110.14168。

+   [Cohen 等, 2023] Cohen, R., Hamri, M., Geva, M., 和 Globerson, A. (2023). LM 对抗 LM：通过交叉检验检测事实错误。arXiv 预印本 arXiv:2305.13281。

+   [Dafoe 等, 2021] Dafoe, A., Bachrach, Y., Hadfield, G., Horvitz, E., Larson, K., 和 Graepel, T. (2021). 合作 AI：机器必须学会找到共同立场。自然, 593(7857):33–36。

+   [Dafoe等，2020] Dafoe, A., Hughes, E., Bachrach, Y., Collins, T., McKee, K. R., Leibo, J. Z., Larson, K., 和 Graepel, T. (2020). 协作AI中的开放问题. arXiv预印本 arXiv:2012.08630.

+   [Devlin等，2018] Devlin, J., Chang, M.-W., Lee, K., 和 Toutanova, K. (2018). Bert: 深度双向变换器的预训练用于语言理解. arXiv预印本 arXiv:1810.04805.

+   [Du等，2023] Du, Y., Li, S., Torralba, A., Tenenbaum, J. B., 和 Mordatch, I. (2023). 通过多代理辩论提高语言模型的事实性和推理能力. arXiv预印本 arXiv:2305.14325.

+   [Gao等，2023] Gao, M., Ruan, J., Sun, R., Yin, X., Yang, S., 和 Wan, X. (2023). 使用chatgpt进行类人摘要评估. arXiv预印本 arXiv:2304.02554.

+   [Graesser等，2020] Graesser, L., Cho, K., 和 Kiela, D. (2020). 多代理通信游戏中的语言现象的涌现.

+   [Hendrycks等，2021] Hendrycks, D., Mazeika, M., Zou, A., Patel, S., Zhu, C., Navarro, J., Song, D., Li, B., 和 Steinhardt, J. (2021). 吉米尼·克里基特会做什么？朝着具有道德行为的代理迈进. arXiv预印本 arXiv:2110.13136.

+   [Karpinska等，2021] Karpinska, M., Akoury, N., 和 Iyyer, M. (2021). 使用Mechanical Turk评估开放式文本生成的风险. arXiv预印本 arXiv:2109.06835.

+   [Kondrak, 2005] Kondrak, G. (2005). N-gram相似性与距离. 在国际字符串处理与信息检索研讨会上，第115-126页. Springer.

+   [Kramár等，2022] Kramár, J., Eccles, T., Gemp, I., Tacchetti, A., McKee, K. R., Malinowski, M., Graepel, T., 和 Bachrach, Y. (2022). 人工智能在《外交》桌面游戏中的谈判与诚实方法. 自然通讯, 13(1):7214.

+   [Lazaridou等，2020] Lazaridou, A., Potapenko, A., 和 Tieleman, O. (2020). 多代理通信与自然语言：功能性与结构性语言学习的协同作用.

+   [Lee等，2018] Lee, J., Cho, K., Weston, J., 和 Kiela, D. (2018). 多代理通信中的语言翻译涌现.

+   [Li等，2023] Li, G., Hammoud, H. A. A. K., Itani, H., Khizbullin, D., 和 Ghanem, B. (2023). Camel: 用于“大规模语言模型社会”思想探索的交流代理. arXiv预印本 arXiv:2303.17760.

+   [Liang等，2023] Liang, T., He, Z., Jiao, W., Wang, X., Wang, Y., Wang, R., Yang, Y., Tu, Z., 和 Shi, S. (2023). 通过多代理辩论鼓励大规模语言模型中的发散性思维. arXiv预印本 arXiv:2305.19118.

+   [Ling等，2017] Ling, W., Yogatama, D., Dyer, C., 和 Blunsom, P. (2017). 通过推理生成程序：学习解决并解释代数文字问题. arXiv预印本 arXiv:1705.04146.

+   [Liu等，2023a] Liu, J., Xia, C. S., Wang, Y., 和 Zhang, L. (2023a). 你的代码真的是由chatgpt生成的吗？对大规模语言模型生成代码的严格评估. arXiv预印本 arXiv:2305.01210.

+   [Liu 等人, 2023b] Liu, Y., Iter, D., Xu, Y., Wang, S., Xu, R., 和 Zhu, C. (2023b). GPTeval: 使用 GPT-4 和更好的人类对齐进行自然语言生成评估. arXiv 预印本 arXiv:2303.16634.

+   [Lu 等人, 2023] Lu, P., Peng, B., Cheng, H., Galley, M., Chang, K.-W., Wu, Y. N., Zhu, S.-C., 和 Gao, J. (2023). Chameleon: 可插拔的大型语言模型组合推理. arXiv 预印本 arXiv:2304.09842.

+   [Luppi 等人, 2022] Luppi, A. I., Mediano, P. A., Rosas, F. E., Holland, N., Fryer, T. D., O’Brien, J. T., Rowe, J. B., Menon, D. K., Bor, D., 和 Stamatakis, E. A. (2022). 人脑进化与认知的协同核心. 《自然神经科学》, 25(6):771–782.

+   [Novikova 等人, 2017] Novikova, J., Dušek, O., Curry, A. C., 和 Rieser, V. (2017). 为什么我们需要新的自然语言生成评估指标. arXiv 预印本 arXiv:1707.06875.

+   [OpenAI, 2023] OpenAI (2023). GPT-4 技术报告. arxiv 2303.08774. 查看文章, 2:13.

+   [Ouyang 等人, 2022] Ouyang, L., Wu, J., Jiang, X., Almeida, D., Wainwright, C. L., Mishkin, P., Zhang, C., Agarwal, S., Slama, K., Ray, A., 等人. (2022). 通过人类反馈训练语言模型遵循指令, 2022. URL https://arxiv.org/abs/2203.02155, 13.

+   [Pan 等人, 2023] Pan, A., Chan, J. S., Zou, A., Li, N., Basart, S., Woodside, T., Zhang, H., Emmons, S., 和 Hendrycks, D. (2023). 奖励是否能证明手段的正当性? 测量奖励与道德行为之间的权衡，在国际机器学习会议上，页码 26837–26867. PMLR.

+   [Patel 等人, 2021] Patel, A., Bhattamishra, S., 和 Goyal, N. (2021). NLP 模型真的能够解决简单的数学文字问题吗? arXiv 预印本 arXiv:2103.07191.

+   [Patil 等人, 2023] Patil, S. G., Zhang, T., Wang, X., 和 Gonzalez, J. E. (2023). Gorilla: 连接大规模 API 的大型语言模型. arXiv 预印本 arXiv:2305.15334.

+   [Qian 等人, 2023] Qian, C., Cong, X., Yang, C., Chen, W., Su, Y., Xu, J., Liu, Z., 和 Sun, M. (2023). 软件开发中的交流代理. arXiv 预印本 arXiv:2307.07924.

+   [Rajani 等人, 2019] Rajani, N. F., McCann, B., Xiong, C., 和 Socher, R. (2019). 解释你自己! 利用语言模型进行常识推理. arXiv 预印本 arXiv:1906.02361.

+   [Rasal 和 Boddhu, 2023] Rasal, S. 和 Boddhu, S. K. (2023). 超越分割：利用多模态大型语言模型生成道路网络. arXiv 预印本 arXiv:2310.09755.

+   [ROUGE, 2004] ROUGE, L. C. (2004). 用于自动评估摘要的工具包. 收录于 ACL 会议西班牙的文本摘要工作坊论文集，第 5 卷.

+   [Roy 和 Roth, 2016] Roy, S. 和 Roth, D. (2016). 解决一般算术文字问题. arXiv 预印本 arXiv:1608.01413.

+   [Sanh 等人, 2021] Sanh, V., Webson, A., Raffel, C., Bach, S. H., Sutawika, L., Alyafeai, Z., Chaffin, A., Stiegler, A., Scao, T. L., Raja, A., 等人. (2021). 多任务提示训练实现零-shot 任务泛化. arXiv 预印本 arXiv:2110.08207.

+   [Sap 等, 2022] Sap, M., LeBras, R., Fried, D., 和 Choi, Y. (2022). 神经理论心智？关于大规模语言模型社会智能的局限性。arXiv 预印本 arXiv:2210.13312。

+   [Sap 等, 2019] Sap, M., Rashkin, H., Chen, D., LeBras, R., 和 Choi, Y. (2019). Socialiqa：关于社会互动的常识推理。arXiv 预印本 arXiv:1904.09728。

+   [Saunders 等, 2022] Saunders, W., Yeh, C., Wu, J., Bills, S., Ouyang, L., Ward, J., 和 Leike, J. (2022). 自我批评模型：协助人工评估员。arXiv 预印本 arXiv:2206.05802。

+   [Schick 等, 2023] Schick, T., Dwivedi-Yu, J., Dessì, R., Raileanu, R., Lomeli, M., Zettlemoyer, L., Cancedda, N., 和 Scialom, T. (2023). Toolformer：语言模型可以自我学习使用工具。arXiv 预印本 arXiv:2302.04761。

+   [Sclar 等, 2023] Sclar, M., Kumar, S., West, P., Suhr, A., Choi, Y., 和 Tsvetkov, Y. (2023). 关注语言模型（缺乏）心智理论：即插即用的多角色信念追踪器。arXiv 预印本 arXiv:2306.00924。

+   [Shen 等, 2023] Shen, C., Cheng, L., You, Y., 和 Bing, L. (2023). 大规模语言模型是否适合评估抽象总结？arXiv 预印本 arXiv:2305.13091。

+   [Susskind, 1985] Susskind, L. E. (1985). 可评分的游戏：一种更好的谈判教学方法。《谈判期刊》，1:205。

+   [Susskind 和 Corburn, 2000] Susskind, L. E. 和 Corburn, J. (2000). 使用仿真教学谈判：教学理论与实践。《谈判教学：思想与创新》，第285–310页。

+   [Talmor 等, 2018] Talmor, A., Herzig, J., Lourie, N., 和 Berant, J. (2018). Commonsenseqa：一个针对常识知识的问答挑战。arXiv 预印本 arXiv:1811.00937。

+   [Thoppilan 等, 2022] Thoppilan, R., De Freitas, D., Hall, J., Shazeer, N., Kulshreshtha, A., Cheng, H.-T., Jin, A., Bos, T., Baker, L., Du, Y., 等 (2022). Lamda：对话应用的语言模型。arXiv 预印本 arXiv:2201.08239。

+   [Touvron 等, 2023] Touvron, H., Lavril, T., Izacard, G., Martinet, X., Lachaux, M.-A., Lacroix, T., Rozière, B., Goyal, N., Hambro, E., Azhar, F., 等 (2023). Llama：开放和高效的基础语言模型。arXiv 预印本 arXiv:2302.13971。

+   [Van Der Lee 等, 2019] Van Der Lee, C., Gatt, A., Van Miltenburg, E., Wubben, S., 和 Krahmer, E. (2019). 自动生成文本的人类评估最佳实践。第12届国际自然语言生成会议论文集，第355–368页。

+   [Wei 等, 2022a] Wei, J., Tay, Y., Bommasani, R., Raffel, C., Zoph, B., Borgeaud, S., Yogatama, D., Bosma, M., Zhou, D., Metzler, D., 等 (2022a). 大规模语言模型的涌现能力。arXiv 预印本 arXiv:2206.07682。

+   [Wei 等, 2022b] Wei, J., Wang, X., Schuurmans, D., Bosma, M., Xia, F., Chi, E., Le, Q. V., Zhou, D., 等 (2022b). 思维链提示激发大规模语言模型推理。《神经信息处理系统进展》，35:24824–24837。

+   [Woolley 等, 2010] Woolley, A. W., Chabris, C. F., Pentland, A., Hashmi, N., 和 Malone, T. W. (2010). 证据表明人类群体表现中的集体智能因素。科学，330(6004):686–688。

+   [Wu 等, 2023] Wu, N., Gong, M., Shou, L., Liang, S., 和 Jiang, D. (2023). 大型语言模型作为多样化角色扮演者用于摘要评估。arXiv 预印本 arXiv:2303.15078。

+   [Yao 等, 2022] Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., 和 Cao, Y. (2022). React: 在语言模型中协同推理与行动。arXiv 预印本 arXiv:2210.03629。

+   [Zheng 等, 2023] Zheng, L., Chiang, W.-L., Sheng, Y., Zhuang, S., Wu, Z., Zhuang, Y., Lin, Z., Li, Z., Li, D., Xing, E., 等. (2023). 使用 mt-bench 和聊天机器人竞技场评估 LLM 作为法官的表现。arXiv 预印本 arXiv:2306.05685。

+   [Zhou 等, 2023] Zhou, P., Zhu, A., Hu, J., Pujara, J., Ren, X., Callison-Burch, C., Choi, Y., 和 Ammanabrolu, P. (2023). 我施法侦测思维：在《龙与地下城》中学习通过意图和心智理论进行对话和引导。载于第61届计算语言学协会年会论文集（第1卷：长篇论文），第11136–11155页。
