<!--yml
category: 未分类
date: 2025-01-11 12:59:04
-->

# LLM Harmony: Multi-Agent Communication for Problem Solving

> 来源：[https://arxiv.org/html/2401.01312/](https://arxiv.org/html/2401.01312/)

Sumedh Rasal
Georgia Institute of Technology
Chicago, IL
srasal3@gatech.edu 

###### Abstract

Large Language Models (LLMs) have revolutionized Natural Language Processing but exhibit limitations, particularly in autonomously addressing novel challenges such as reasoning and problem-solving. Traditional techniques like chain-of-thought prompting necessitate explicit human guidance. This paper introduces a novel multi-agent communication framework, inspired by the CAMEL model, to enhance LLMs’ autonomous problem-solving capabilities. The framework employs multiple LLM agents, each with a distinct persona, engaged in role-playing communication, offering a nuanced and adaptable approach to diverse problem scenarios. Extensive experimentation demonstrates the framework’s superior performance and adaptability, providing valuable insights into the collaborative potential of multiple agents in overcoming the limitations of individual models.

## 1 Introduction

The rise of Large Language Models (LLMs) [[Brown et al., 2020](#bib.bibx3)] [[Touvron et al., 2023](#bib.bibx55)] [[Thoppilan et al., 2022](#bib.bibx54)] [[Patil et al., 2023](#bib.bibx38)] [[OpenAI, 2023](#bib.bibx34)] [[Bubeck et al., 2023](#bib.bibx4)] has undeniably revolutionized the software industry, particularly in the realm of Natural Language Processing (NLP) [[Devlin et al., 2018](#bib.bibx16)]. These models have demonstrated a remarkable ability not only to generate text but also to grasp the underlying structures of written language, extending their utility to tasks ranging from text generation to code understanding [[Chen et al., 2021](#bib.bibx7)] [[Schick et al., 2023](#bib.bibx48)]. However, despite their prowess, it has become evident that LLMs, while adept at handling familiar topics, often falter when confronted with novel challenges.

One notable limitation lies in their tendency to hallucinate information when presented with unfamiliar subjects [[Azamfirei et al., 2023](#bib.bibx2)]. Moreover, while these models exhibit commendable proficiency in addressing technical coding challenges across multiple programming languages, they struggle with fundamental reasoning questions. Addressing such limitations necessitates an approach that goes beyond conventional methodologies [[Rajani et al., 2019](#bib.bibx40)] [[Ling et al., 2017](#bib.bibx28)] [[Cobbe et al., 2021a](#bib.bibx11)] [[Chiang and Chen, 2018](#bib.bibx10)] [[Amini et al., 2019](#bib.bibx1)] [[Chen et al., 2019](#bib.bibx8)] [[Roy and Roth, 2016](#bib.bibx43)] [[Lu et al., 2023](#bib.bibx31)] [[Yao et al., 2022](#bib.bibx61)].

In response to this challenge, our paper introduces a novel strategy aimed at enhancing LLM performance on novel problems. Drawing inspiration from the effectiveness of chain-of-thought prompting [[Wei et al., 2022b](#bib.bibx58)] in breaking down complex problems, we seek to leverage the synergy of multiple LLM agents working collaboratively [[Cohen et al., 2023](#bib.bibx13)] [[Du et al., 2023](#bib.bibx17)]. Each agent is endowed with a distinct persona, a concept inspired by the CAMEL framework, and engages in role-playing communication methods.

Unlike traditional approaches that might be limited to binary agent personas, such as "user" and "assistant," or simplistic positive and negative agent roles, our proposed framework adopts a nuanced and flexible strategy. The intricacies of various novel scenarios demand a more sophisticated approach. Our design incorporates industry best practices to guide the creation of diverse agent personas, ensuring adaptability to a multitude of problem-solving contexts.

The paper not only addresses the limitations of existing LLM models but also explores the potential of harnessing the collective intelligence of multiple agents to tackle a broader range of challenges. We aim to demonstrate that the proposed framework not only outperforms traditional methodologies but also provides a foundation for autonomous problem-solving, minimizing the need for explicit human guidance.

In the subsequent sections, we delve into the existing techniques that have been developed to address similar challenges, highlighting their strengths and limitations [[Dafoe et al., 2021](#bib.bibx14)]. Following this, we present our multi-agent communication design, emphasizing its improvements over current methodologies [[Saunders et al., 2022](#bib.bibx47)]. The extendability of our approach to various LLM models is a key focus, underlining its potential as a versatile and reusable solution. Through rigorous experimentation and analysis, we aim to showcase the effectiveness and adaptability of our proposed framework in enhancing the quality of LLM outputs across a spectrum of tasks.

This is what we propose:

*   •

    An innovative framework employing multiple large language model workers/agents, each characterized by a unique persona and guided by a chain-of-thought prompt.

*   •

    The collaborative effort of all agents is directed toward devising solutions for novel problems.

*   •

    The framework’s versatility allows the incorporation of any persona and chain-of-thought prompt, aligning with the specific problem to be addressed.

*   •

    The framework is available. [https://github.com/sumedhrasal/simulation](https://github.com/sumedhrasal/simulation). It is built on top of CAMEL’s and ChatDev’s framework.

## 2 Methodology

In the realm of collaborative problem-solving, particularly evident in scenarios like software development, the complexities arise from coordinating efforts among diverse individuals. Software development teams, for instance, often undergo iterative cycles before converging on a viable solution, and even then, compromises may be accepted as the best available option [[Callison-Burch, 2009](#bib.bibx5)] [[Liu et al., 2023a](#bib.bibx29)]. Our proposed strategy addresses the need for comprehensive validation of Large Language Models (LLMs), not just in code evaluation but also in complex reasoning and arithmetic challenges [[Celikyilmaz et al., 2020](#bib.bibx6)] [[ROUGE, 2004](#bib.bibx42)] [[Kondrak, 2005](#bib.bibx22)] [[Novikova et al., 2017](#bib.bibx33)] [[Wei et al., 2022a](#bib.bibx57)]. Focused on breaking down problem statements and leveraging distinct personas [[Li et al., 2023](#bib.bibx26)], the framework provides agents with tailored chain-of-thought prompts [[Wei et al., 2022b](#bib.bibx58)], enabling multi-agent models to tackle novel problems collaboratively.

Understanding personas is crucial; in a software development organization, roles such as CEO, VP of Engineering, developers, testers, and product managers contribute unique perspectives to problem-solving [[Qian et al., 2023](#bib.bibx39)]. The interplay of these personas allows agents to validate each other’s responses effectively [[Woolley et al., 2010](#bib.bibx59)] [[Luppi et al., 2022](#bib.bibx32)] [[Wu et al., 2023](#bib.bibx60)]. The concept of chain-of-thought mirrors human problem-solving strategies: decomposing a problem, identifying solutions to sub-problems, and constructing a comprehensive answer. The chain-of-thought prompt extends this strategy, offering the LLM an input, a problem-solving approach, and the desired output. The fusion of personas [[Li et al., 2023](#bib.bibx26)] and chain-of-thought [[Wei et al., 2022b](#bib.bibx58)] prompts capitalizes on mimicking human analytical and execution strategies [[Ouyang et al., 2022](#bib.bibx35)] [[Sanh et al., 2021](#bib.bibx44)] [[Liu et al., 2023b](#bib.bibx30)] [[Chiang and Lee, 2023](#bib.bibx9)] [[Gao et al., 2023](#bib.bibx18)] [[Shen et al., 2023](#bib.bibx50)] [[Zheng et al., 2023](#bib.bibx62)] [[Sap et al., 2019](#bib.bibx46)] [[Talmor et al., 2018](#bib.bibx53)] [[Sclar et al., 2023](#bib.bibx49)] [[Sap et al., 2022](#bib.bibx45)].

While some might question the necessity of this approach over retraining the model for novel problems, the practicality lies in the belief that emulating human-like analysis and execution strategies can create autonomous agents capable of diverse tasks without extensive model retraining [[Karpinska et al., 2021](#bib.bibx21)] [[Van Der Lee et al., 2019](#bib.bibx56)]. Additionally, the high cost and challenges associated with retraining LLMs [[Rasal and Boddhu, 2023](#bib.bibx41)], particularly for problems with limited training data, support the appeal of our strategy.

In the realm of multi-agent communication [[Liang et al., 2023](#bib.bibx27)] [[Du et al., 2023](#bib.bibx17)], adherence to assigned personas and chain-of-thought prompts is foundational to mitigate agent hallucination and improving cooperation [[Dafoe et al., 2021](#bib.bibx14)] [[Dafoe et al., 2020](#bib.bibx15)]. The framework’s versatility extends beyond problem-solving to applications in multi-player games [[Susskind, 1985](#bib.bibx51)] [[Susskind and Corburn, 2000](#bib.bibx52)] [[Lazaridou et al., 2020](#bib.bibx24)] [[Graesser et al., 2020](#bib.bibx19)] [[Lee et al., 2018](#bib.bibx25)] [[Hendrycks et al., 2021](#bib.bibx20)] [[Zhou et al., 2023](#bib.bibx63)] [[Pan et al., 2023](#bib.bibx36)] [[Kramár et al., 2022](#bib.bibx23)], accommodating both team-oriented and individual goal-driven scenarios, and offering flexibility in achieving optimal outcomes.

## 3 Experiments

In our experimentation, we adopt a two-agent strategy comprising an expert agent and an evaluator agent, both instances of OpenAI’s "gpt3.5-turbo." Each agent is configured with specific parameters: temperature set to 0.0, representing the trade-off between coherence and creativity, and a conversation limit of 5, indicating the maximum iterations allowed for inter-agent conversation. The evaluator agent assesses the responses generated by the expert agent, guiding it to rectify inaccuracies. Leveraging OpenAI’s LLM object, our framework, integrated with CAMEL’s architecture, facilitates seamless communication among multiple LLM agents, aiming to establish a collaborative problem-solving capability for advanced LLM models.

### 3.1 Arithmetic Reasoning

In the arithmetic reasoning segment, we address challenges that assess the arithmetic proficiency of LLMs, an area where traditional large language models have historically faced limitations. The experiment employs the GSM8K benchmark and the SVAMP data set, specifically designed for math word problems. We evaluate the performance of three LLMs: a standalone GPT-3 model, a multi-agent GPT-3 model, and a multi-agent GPT-3 model integrated into our framework. The objective is to gauge the effectiveness of our collaborative approach in enhancing arithmetic reasoning capabilities.

> "GSM8K consists of 8.5K high-quality grade school math problems created by human problem writers. We segmented these into 7.5K training problems and 1K test problems. These problems take between 2 and 8 steps to solve, and solutions primarily involve performing a sequence of elementary calculations using basic arithmetic operations (+ - / *) to reach the final answer. A bright middle school student should be able to solve every problem."[[Cobbe et al., 2021b](#bib.bibx12)]

> "We first show that existing models achieve reasonably high accuracies on these datasets even after removing the "question" part of the MWP at test time. We further show that a simple model without any word-order information can also solve a majority of MWPs in these datasets. Our experiments indicate that existing models rely on shallow heuristics in benchmark MWP datasets for achieving high performance. Our experiments render the benchmark datasets unreliable to measure model performance. To enable more robust evaluation of automatic MWP solvers, we created a challenge set called "SVAMP"."[[Patel et al., 2021](#bib.bibx37)]

#### 3.1.1 Example: 1

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

#### 3.1.2 Example 2

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

#### 3.1.3 Results

The outcomes from both experiments can be found in Table [1](#S3.T1 "Table 1 ‣ 3.1.3 Results ‣ 3.1 Arithmetic Reasoning ‣ 3 Experiments ‣ LLM Harmony: Multi-Agent Communication for Problem Solving") and Table [2](#S3.T2 "Table 2 ‣ 3.1.3 Results ‣ 3.1 Arithmetic Reasoning ‣ 3 Experiments ‣ LLM Harmony: Multi-Agent Communication for Problem Solving"). In the initial experiment (example can be found here [3.1.1](#S3.SS1.SSS1 "3.1.1 Example: 1 ‣ 3.1 Arithmetic Reasoning ‣ 3 Experiments ‣ LLM Harmony: Multi-Agent Communication for Problem Solving")) using the GSM8K data set, the single-agent GPT-3 achieves approximately 50% accuracy, and the multi-agent GPT-3 performs slightly better at 55%. However, our multi-agent approach significantly enhances accuracy, surpassing other large language models (LLMs) such as Google’s PALM 540B parameter model, which we haven’t directly tested but are referencing from their paper. This improvement is notable in terms of accuracy, and it’s noteworthy that we haven’t retrained the model to achieve this enhancement. Assigning personas to the agents enables the LLM model to concentrate on specific aspects of the problem, and the use of chain-of-thought prompts equips it with efficient means to solve sub-problems.

In the second experiment (example can be found here [3.1.2](#S3.SS1.SSS2 "3.1.2 Example 2 ‣ 3.1 Arithmetic Reasoning ‣ 3 Experiments ‣ LLM Harmony: Multi-Agent Communication for Problem Solving")) using the SVAMP data set, similar results are observed to those in the first experiment. The single-agent GPT-3 achieves an accuracy of approximately 70%, while the multi-agent version of GPT-3 attains an accuracy of 73%. Even in the SVAMP data set experiment, our multi-agent approach surpasses others, delivering an impressive 77% accuracy.

If our approach is unsuccessful in arriving at the correct answer, it is often due to an arithmetic error occurring in one of the sub-steps of the provided problem, accounting for more than half of the instances. This implies that if subsequent iterations of large language models (LLMs) enhance their proficiency in basic arithmetic operations, we can potentially attain an even greater accuracy with our framework.

Table 1: GSM8K Data set Evaluation

| Agent | Solve Rate |
| --- | --- |
| Single GPT3.5-turbo | 50% |
| Multi-Agent GPT3.5-turbo | 55% |
| Multi-Agent GPT3.5-turbo (Our approach) | 65% |

Table 2: SVAMP Data set Evaluation

| Agent | Solve Rate |
| --- | --- |
| Single GPT3.5-turbo | 70% |
| Multi-Agent GPT3.5-turbo | 73% |
| Multi-Agent GPT3.5-turbo (Our approach) | 77% |

### 3.2 Commonsense Reasoning

The subsequent experiment delves into commonsense reasoning, requiring logical inference from a given problem statement, assuming general knowledge. Unlike traditional natural language processing systems, LLMs are uniquely equipped for such tasks due to their predictive nature, where they anticipate the next word, inherently understanding the contextual nuances of sentences. We aim to harness this inherent capability and build a versatile, context-aware framework driven by multiple agents. Evaluations on the CSQA dataset involve a one-agent GPT-3 model, a multi-agent GPT-3 model, and a multi-agent GPT-3 model integrated into our framework. The goal is to assess the performance gains achieved through collaborative, context-driven approaches in enhancing commonsense reasoning tasks.

#### 3.2.1 Example 1

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

#### 3.2.2 Example 2

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

#### 3.2.3 Results

The results of the third experiment are detailed in Table [3](#S3.T3 "Table 3 ‣ 3.2.3 Results ‣ 3.2 Commonsense Reasoning ‣ 3 Experiments ‣ LLM Harmony: Multi-Agent Communication for Problem Solving"). The single-agent GPT-3 achieves an impressive 77% accuracy, while the multi-agent GPT-3 performs slightly better at 78%. However, our multi-agent approach substantially improves accuracy, surpassing the other two and reaching approximately 83% accuracy. Notably, this accuracy is attained through Few-Shot training, indicating there is room for further enhancement. Few-shot training involves providing the LLM with a few examples of a specific problem type, enabling it to learn the correct answer without retraining the entire model for novel problems. Examples can be found here [3.2.1](#S3.SS2.SSS1 "3.2.1 Example 1 ‣ 3.2 Commonsense Reasoning ‣ 3 Experiments ‣ LLM Harmony: Multi-Agent Communication for Problem Solving") and here [3.2.2](#S3.SS2.SSS2 "3.2.2 Example 2 ‣ 3.2 Commonsense Reasoning ‣ 3 Experiments ‣ LLM Harmony: Multi-Agent Communication for Problem Solving").

In instances where the model provided incorrect answers, it often stemmed from making inaccurate correlations within the provided options. This incorrect correlation is a result of the underlying data set on which this particular model was trained. While retraining the model could improve the accuracy of this experiment, it’s considered a costly option. This poses an open question for future researchers to address: How can the model be trained with enough context to establish accurate correlations in commonsense word problems?

Table 3: CSQA Data set Evaluation

| Agent | Solve Rate |
| --- | --- |
| Single GPT3.5-turbo | 77% |
| Multi-Agent GPT3.5-turbo | 78% |
| Multi-Agent GPT3.5-turbo (Our approach) | 83% |

## 4 Limitations

While we recognize that our framework addresses numerous challenges, there are still a few aspects that remain unaddressed. Some of the reasoning capabilities may not see improvement unless the dataset used to train OpenAI’s "gpt3.5-turbo" is sufficiently diverse to comprehend the entirety of our surroundings. Additionally, the framework requires the capability to incorporate new information to stay updated with the constantly evolving data. Another necessary step involves implementing a data processing mechanism to filter out redundant information and prevent the inclusion of duplicate data.

Another limitation pertains to the context limit of each agent in multi-agent communication. Each agent is constrained by the maximum context, defined by the underlying model; for instance, the "gpt3-turbo" model has a context limit of 4096 tokens.

We intend to address these limitations in our future work and strive to find effective solutions.

## 5 Conclusion

In conclusion, our exploration into the realm of multi-agent communication for Large Language Models (LLMs) has unveiled promising avenues for overcoming inherent limitations. While LLMs have revolutionized natural language processing, their efficacy is not without challenges, particularly in addressing novel problems, reasoning, and commonsense understanding. Our proposed approach leverages personas and chain-of-thought prompts, inspired by industry best practices and cognitive processes. By assigning distinct personalities and thoughtful prompts to each agent, we mitigate issues like hallucination and enhance the overall performance of multi-agent communication.

Through a series of experiments, we demonstrated the effectiveness of our approach, showcasing improvements in arithmetic reasoning and commonsense understanding. Our strategy of employing multiple agents, each with a specific role and reasoning prompt, facilitates collaborative problem-solving, providing a feasible alternative to the costly retraining of LLMs for novel challenges.

By reducing reliance on human intervention, our approach paves the way for LLMs to tackle a myriad of tasks independently. The scalability and adaptability of our role-playing framework position it as a valuable asset in various domains, from software development to complex decision-making scenarios.

In an era where artificial intelligence continues to evolve, our research contributes a novel perspective on enhancing the capabilities of LLMs through cooperative multi-agent communication. The journey from understanding limitations to proposing effective solutions marks a significant step forward, opening doors to future advancements in autonomous, context-aware language models.

## References

*   [Amini et al., 2019] Amini, A., Gabriel, S., Lin, P., Koncel-Kedziorski, R., Choi, Y., and Hajishirzi, H. (2019). Mathqa: Towards interpretable math word problem solving with operation-based formalisms. arXiv preprint arXiv:1905.13319.
*   [Azamfirei et al., 2023] Azamfirei, R., Kudchadkar, S. R., and Fackler, J. (2023). Large language models and the perils of their hallucinations. Critical Care, 27(1):1–2.
*   [Brown et al., 2020] Brown, T., Mann, B., Ryder, N., Subbiah, M., Kaplan, J. D., Dhariwal, P., Neelakantan, A., Shyam, P., Sastry, G., Askell, A., et al. (2020). Language models are few-shot learners. Advances in neural information processing systems, 33:1877–1901.
*   [Bubeck et al., 2023] Bubeck, S., Chandrasekaran, V., Eldan, R., Gehrke, J., Horvitz, E., Kamar, E., Lee, P., Lee, Y. T., Li, Y., Lundberg, S., et al. (2023). Sparks of artificial general intelligence: Early experiments with gpt-4. arXiv preprint arXiv:2303.12712.
*   [Callison-Burch, 2009] Callison-Burch, C. (2009). Fast, cheap, and creative: Evaluating translation quality using amazon’s mechanical turk. In Proceedings of the 2009 conference on empirical methods in natural language processing, pages 286–295.
*   [Celikyilmaz et al., 2020] Celikyilmaz, A., Clark, E., and Gao, J. (2020). Evaluation of text generation: A survey. corr abs/2006.14799 (2020). arXiv preprint arXiv:2006.14799.
*   [Chen et al., 2021] Chen, M., Tworek, J., Jun, H., Yuan, Q., Pinto, H. P. d. O., Kaplan, J., Edwards, H., Burda, Y., Joseph, N., Brockman, G., et al. (2021). Evaluating large language models trained on code. arXiv preprint arXiv:2107.03374.
*   [Chen et al., 2019] Chen, X., Liang, C., Yu, A. W., Zhou, D., Song, D., and Le, Q. V. (2019). Neural symbolic reader: Scalable integration of distributed and symbolic representations for reading comprehension. In International Conference on Learning Representations.
*   [Chiang and Lee, 2023] Chiang, C.-H. and Lee, H.-y. (2023). Can large language models be an alternative to human evaluations? arXiv preprint arXiv:2305.01937.
*   [Chiang and Chen, 2018] Chiang, T.-R. and Chen, Y.-N. (2018). Semantically-aligned equation generation for solving and reasoning math word problems. arXiv preprint arXiv:1811.00720.
*   [Cobbe et al., 2021a] Cobbe, K., Kosaraju, V., Bavarian, M., Chen, M., Jun, H., Kaiser, L., Plappert, M., Tworek, J., Hilton, J., Nakano, R., et al. (2021a). Training verifiers to solve math word problems. arXiv preprint arXiv:2110.14168.
*   [Cobbe et al., 2021b] Cobbe, K., Kosaraju, V., Bavarian, M., Chen, M., Jun, H., Kaiser, L., Plappert, M., Tworek, J., Hilton, J., Nakano, R., Hesse, C., and Schulman, J. (2021b). Training verifiers to solve math word problems. arXiv preprint arXiv:2110.14168.
*   [Cohen et al., 2023] Cohen, R., Hamri, M., Geva, M., and Globerson, A. (2023). Lm vs lm: Detecting factual errors via cross examination. arXiv preprint arXiv:2305.13281.
*   [Dafoe et al., 2021] Dafoe, A., Bachrach, Y., Hadfield, G., Horvitz, E., Larson, K., and Graepel, T. (2021). Cooperative ai: machines must learn to find common ground. Nature, 593(7857):33–36.
*   [Dafoe et al., 2020] Dafoe, A., Hughes, E., Bachrach, Y., Collins, T., McKee, K. R., Leibo, J. Z., Larson, K., and Graepel, T. (2020). Open problems in cooperative ai. arXiv preprint arXiv:2012.08630.
*   [Devlin et al., 2018] Devlin, J., Chang, M.-W., Lee, K., and Toutanova, K. (2018). Bert: Pre-training of deep bidirectional transformers for language understanding. arXiv preprint arXiv:1810.04805.
*   [Du et al., 2023] Du, Y., Li, S., Torralba, A., Tenenbaum, J. B., and Mordatch, I. (2023). Improving factuality and reasoning in language models through multiagent debate. arXiv preprint arXiv:2305.14325.
*   [Gao et al., 2023] Gao, M., Ruan, J., Sun, R., Yin, X., Yang, S., and Wan, X. (2023). Human-like summarization evaluation with chatgpt. arXiv preprint arXiv:2304.02554.
*   [Graesser et al., 2020] Graesser, L., Cho, K., and Kiela, D. (2020). Emergent linguistic phenomena in multi-agent communication games.
*   [Hendrycks et al., 2021] Hendrycks, D., Mazeika, M., Zou, A., Patel, S., Zhu, C., Navarro, J., Song, D., Li, B., and Steinhardt, J. (2021). What would jiminy cricket do? towards agents that behave morally. arXiv preprint arXiv:2110.13136.
*   [Karpinska et al., 2021] Karpinska, M., Akoury, N., and Iyyer, M. (2021). The perils of using mechanical turk to evaluate open-ended text generation. arXiv preprint arXiv:2109.06835.
*   [Kondrak, 2005] Kondrak, G. (2005). N-gram similarity and distance. In International symposium on string processing and information retrieval, pages 115–126\. Springer.
*   [Kramár et al., 2022] Kramár, J., Eccles, T., Gemp, I., Tacchetti, A., McKee, K. R., Malinowski, M., Graepel, T., and Bachrach, Y. (2022). Negotiation and honesty in artificial intelligence methods for the board game of diplomacy. Nature Communications, 13(1):7214.
*   [Lazaridou et al., 2020] Lazaridou, A., Potapenko, A., and Tieleman, O. (2020). Multi-agent communication meets natural language: Synergies between functional and structural language learning.
*   [Lee et al., 2018] Lee, J., Cho, K., Weston, J., and Kiela, D. (2018). Emergent translation in multi-agent communication.
*   [Li et al., 2023] Li, G., Hammoud, H. A. A. K., Itani, H., Khizbullin, D., and Ghanem, B. (2023). Camel: Communicative agents for" mind" exploration of large scale language model society. arXiv preprint arXiv:2303.17760.
*   [Liang et al., 2023] Liang, T., He, Z., Jiao, W., Wang, X., Wang, Y., Wang, R., Yang, Y., Tu, Z., and Shi, S. (2023). Encouraging divergent thinking in large language models through multi-agent debate. arXiv preprint arXiv:2305.19118.
*   [Ling et al., 2017] Ling, W., Yogatama, D., Dyer, C., and Blunsom, P. (2017). Program induction by rationale generation: Learning to solve and explain algebraic word problems. arXiv preprint arXiv:1705.04146.
*   [Liu et al., 2023a] Liu, J., Xia, C. S., Wang, Y., and Zhang, L. (2023a). Is your code generated by chatgpt really correct? rigorous evaluation of large language models for code generation. arXiv preprint arXiv:2305.01210.
*   [Liu et al., 2023b] Liu, Y., Iter, D., Xu, Y., Wang, S., Xu, R., and Zhu, C. (2023b). Gpteval: Nlg evaluation using gpt-4 with better human alignment. arXiv preprint arXiv:2303.16634.
*   [Lu et al., 2023] Lu, P., Peng, B., Cheng, H., Galley, M., Chang, K.-W., Wu, Y. N., Zhu, S.-C., and Gao, J. (2023). Chameleon: Plug-and-play compositional reasoning with large language models. arXiv preprint arXiv:2304.09842.
*   [Luppi et al., 2022] Luppi, A. I., Mediano, P. A., Rosas, F. E., Holland, N., Fryer, T. D., O’Brien, J. T., Rowe, J. B., Menon, D. K., Bor, D., and Stamatakis, E. A. (2022). A synergistic core for human brain evolution and cognition. Nature Neuroscience, 25(6):771–782.
*   [Novikova et al., 2017] Novikova, J., Dušek, O., Curry, A. C., and Rieser, V. (2017). Why we need new evaluation metrics for nlg. arXiv preprint arXiv:1707.06875.
*   [OpenAI, 2023] OpenAI (2023). Gpt-4 technical report. arxiv 2303.08774. View in Article, 2:13.
*   [Ouyang et al., 2022] Ouyang, L., Wu, J., Jiang, X., Almeida, D., Wainwright, C. L., Mishkin, P., Zhang, C., Agarwal, S., Slama, K., Ray, A., et al. (2022). Training language models to follow instructions with human feedback, 2022. URL https://arxiv. org/abs/2203.02155, 13.
*   [Pan et al., 2023] Pan, A., Chan, J. S., Zou, A., Li, N., Basart, S., Woodside, T., Zhang, H., Emmons, S., and Hendrycks, D. (2023). Do the rewards justify the means? measuring trade-offs between rewards and ethical behavior in the machiavelli benchmark. In International Conference on Machine Learning, pages 26837–26867\. PMLR.
*   [Patel et al., 2021] Patel, A., Bhattamishra, S., and Goyal, N. (2021). Are nlp models really able to solve simple math word problems? arXiv preprint arXiv:2103.07191.
*   [Patil et al., 2023] Patil, S. G., Zhang, T., Wang, X., and Gonzalez, J. E. (2023). Gorilla: Large language model connected with massive apis. arXiv preprint arXiv:2305.15334.
*   [Qian et al., 2023] Qian, C., Cong, X., Yang, C., Chen, W., Su, Y., Xu, J., Liu, Z., and Sun, M. (2023). Communicative agents for software development. arXiv preprint arXiv:2307.07924.
*   [Rajani et al., 2019] Rajani, N. F., McCann, B., Xiong, C., and Socher, R. (2019). Explain yourself! leveraging language models for commonsense reasoning. arXiv preprint arXiv:1906.02361.
*   [Rasal and Boddhu, 2023] Rasal, S. and Boddhu, S. K. (2023). Beyond segmentation: Road network generation with multi-modal llms. arXiv preprint arXiv:2310.09755.
*   [ROUGE, 2004] ROUGE, L. C. (2004). A package for automatic evaluation of summaries. In Proceedings of Workshop on Text Summarization of ACL, Spain, volume 5.
*   [Roy and Roth, 2016] Roy, S. and Roth, D. (2016). Solving general arithmetic word problems. arXiv preprint arXiv:1608.01413.
*   [Sanh et al., 2021] Sanh, V., Webson, A., Raffel, C., Bach, S. H., Sutawika, L., Alyafeai, Z., Chaffin, A., Stiegler, A., Scao, T. L., Raja, A., et al. (2021). Multitask prompted training enables zero-shot task generalization. arXiv preprint arXiv:2110.08207.
*   [Sap et al., 2022] Sap, M., LeBras, R., Fried, D., and Choi, Y. (2022). Neural theory-of-mind? on the limits of social intelligence in large lms. arXiv preprint arXiv:2210.13312.
*   [Sap et al., 2019] Sap, M., Rashkin, H., Chen, D., LeBras, R., and Choi, Y. (2019). Socialiqa: Commonsense reasoning about social interactions. arXiv preprint arXiv:1904.09728.
*   [Saunders et al., 2022] Saunders, W., Yeh, C., Wu, J., Bills, S., Ouyang, L., Ward, J., and Leike, J. (2022). Self-critiquing models for assisting human evaluators. arXiv preprint arXiv:2206.05802.
*   [Schick et al., 2023] Schick, T., Dwivedi-Yu, J., Dessì, R., Raileanu, R., Lomeli, M., Zettlemoyer, L., Cancedda, N., and Scialom, T. (2023). Toolformer: Language models can teach themselves to use tools. arXiv preprint arXiv:2302.04761.
*   [Sclar et al., 2023] Sclar, M., Kumar, S., West, P., Suhr, A., Choi, Y., and Tsvetkov, Y. (2023). Minding language models’(lack of) theory of mind: A plug-and-play multi-character belief tracker. arXiv preprint arXiv:2306.00924.
*   [Shen et al., 2023] Shen, C., Cheng, L., You, Y., and Bing, L. (2023). Are large language models good evaluators for abstractive summarization? arXiv preprint arXiv:2305.13091.
*   [Susskind, 1985] Susskind, L. E. (1985). Scorable games: A better way to teach negotiation. Negot. J., 1:205.
*   [Susskind and Corburn, 2000] Susskind, L. E. and Corburn, J. (2000). Using simulations to teach negotiation: Pedagogical theory and practice. Teaching negotiation: Ideas and innovations, pages 285–310.
*   [Talmor et al., 2018] Talmor, A., Herzig, J., Lourie, N., and Berant, J. (2018). Commonsenseqa: A question answering challenge targeting commonsense knowledge. arXiv preprint arXiv:1811.00937.
*   [Thoppilan et al., 2022] Thoppilan, R., De Freitas, D., Hall, J., Shazeer, N., Kulshreshtha, A., Cheng, H.-T., Jin, A., Bos, T., Baker, L., Du, Y., et al. (2022). Lamda: Language models for dialog applications. arXiv preprint arXiv:2201.08239.
*   [Touvron et al., 2023] Touvron, H., Lavril, T., Izacard, G., Martinet, X., Lachaux, M.-A., Lacroix, T., Rozière, B., Goyal, N., Hambro, E., Azhar, F., et al. (2023). Llama: Open and efficient foundation language models. arXiv preprint arXiv:2302.13971.
*   [Van Der Lee et al., 2019] Van Der Lee, C., Gatt, A., Van Miltenburg, E., Wubben, S., and Krahmer, E. (2019). Best practices for the human evaluation of automatically generated text. In Proceedings of the 12th International Conference on Natural Language Generation, pages 355–368.
*   [Wei et al., 2022a] Wei, J., Tay, Y., Bommasani, R., Raffel, C., Zoph, B., Borgeaud, S., Yogatama, D., Bosma, M., Zhou, D., Metzler, D., et al. (2022a). Emergent abilities of large language models. arXiv preprint arXiv:2206.07682.
*   [Wei et al., 2022b] Wei, J., Wang, X., Schuurmans, D., Bosma, M., Xia, F., Chi, E., Le, Q. V., Zhou, D., et al. (2022b). Chain-of-thought prompting elicits reasoning in large language models. Advances in Neural Information Processing Systems, 35:24824–24837.
*   [Woolley et al., 2010] Woolley, A. W., Chabris, C. F., Pentland, A., Hashmi, N., and Malone, T. W. (2010). Evidence for a collective intelligence factor in the performance of human groups. science, 330(6004):686–688.
*   [Wu et al., 2023] Wu, N., Gong, M., Shou, L., Liang, S., and Jiang, D. (2023). Large language models are diverse role-players for summarization evaluation. arXiv preprint arXiv:2303.15078.
*   [Yao et al., 2022] Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., and Cao, Y. (2022). React: Synergizing reasoning and acting in language models. arXiv preprint arXiv:2210.03629.
*   [Zheng et al., 2023] Zheng, L., Chiang, W.-L., Sheng, Y., Zhuang, S., Wu, Z., Zhuang, Y., Lin, Z., Li, Z., Li, D., Xing, E., et al. (2023). Judging llm-as-a-judge with mt-bench and chatbot arena. arXiv preprint arXiv:2306.05685.
*   [Zhou et al., 2023] Zhou, P., Zhu, A., Hu, J., Pujara, J., Ren, X., Callison-Burch, C., Choi, Y., and Ammanabrolu, P. (2023). I cast detect thoughts: Learning to converse and guide with intents and theory-of-mind in dungeons and dragons. In Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), pages 11136–11155.