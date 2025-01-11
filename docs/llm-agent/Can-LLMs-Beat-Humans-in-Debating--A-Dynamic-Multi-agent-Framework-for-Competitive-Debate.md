<!--yml
category: 未分类
date: 2025-01-11 12:19:53
-->

# Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate

> 来源：[https://arxiv.org/html/2408.04472/](https://arxiv.org/html/2408.04472/)

Yiqun Zhang¹, Xiaocui Yang¹, Shi Feng¹, Daling Wang¹, Yifei Zhang¹, Kaisong Song² Corresponding author.

###### Abstract

Competitive debate is a complex task of computational argumentation. Large Language Models (LLMs) suffer from hallucinations and lack competitiveness in this field. To address these challenges, we introduce Agent for Debate (Agent4Debate), a dynamic multi-agent framework based on LLMs designed to enhance their capabilities in competitive debate. Drawing inspiration from human behavior in debate preparation and execution, Agent4Debate employs a collaborative architecture where four specialized agents, involving Searcher, Analyzer, Writer, and Reviewer, dynamically interact and cooperate. These agents work throughout the debate process, covering multiple stages from initial research and argument formulation to rebuttal and summary. To comprehensively evaluate framework performance, we construct the Competitive Debate Arena, comprising 66 carefully selected Chinese debate motions. We recruit ten experienced human debaters and collect records of 200 debates involving Agent4Debate, baseline models, and humans. The evaluation employs the Debatrix automatic scoring system and professional human reviewers based on the established Debatrix-Elo and Human-Elo ranking. Experimental results indicate that the state-of-the-art Agent4Debate exhibits capabilities comparable to those of humans. Furthermore, ablation studies demonstrate the effectiveness of each component in the agent structure.

Code — https://github.com/ZhangYiqun018/agent-for-debate

## Introduction

Competitive debate, as a structured and competitive form of communication (Nichols [1936](https://arxiv.org/html/2408.04472v2#bib.bib21); Thueblood [1926](https://arxiv.org/html/2408.04472v2#bib.bib30)), plays a crucial role in fields such as education, law, and politics. It challenges the comprehensive ability of participants, including logical thinking, expression skills, rapid analysis, argument construction, and rebuttal techniques, ultimately aiming to persuade a third party. With the advancement of artificial intelligence technologies, computational argumentation has emerged, and it is dedicated to simulating and understanding human argumentation processes through computational methods (Atkinson et al. [2017](https://arxiv.org/html/2408.04472v2#bib.bib3); Eger, Daxenberger, and Gurevych [2017](https://arxiv.org/html/2408.04472v2#bib.bib10)). However, existing research is largely confined to specific tasks on particular datasets, such as argument mining (Lawrence and Reed [2019](https://arxiv.org/html/2408.04472v2#bib.bib16)), argument quality assessment (Wachsmuth et al. [2017a](https://arxiv.org/html/2408.04472v2#bib.bib35)), and argument generation (Li, Ji, and Han [2021](https://arxiv.org/html/2408.04472v2#bib.bib18)). While these methods excel at specific tasks, they struggle to handle the complexity of competitive debate characterized by its openness, intense competition, and the need for decision-making and comprehensive skills.

In recent years, Large Language Models (LLMs) (OpenAI [2023](https://arxiv.org/html/2408.04472v2#bib.bib22); Touvron et al. [2023b](https://arxiv.org/html/2408.04472v2#bib.bib32)) have demonstrated remarkable capabilities in various natural language processing tasks, offering new possibilities for constructing high-performance debate systems. Competitive debate, characterized by multi-turn document-level text generation with inter-turn logical dependencies, presents a unique challenge for LLMs, particularly in two significant areas. First, LLMs often face hallucination problems (Ji et al. [2023](https://arxiv.org/html/2408.04472v2#bib.bib14)), where models may generate plausible information that is inaccurate or fabricated. Second, due to limitations in safety alignment during training (Ouyang et al. [2022](https://arxiv.org/html/2408.04472v2#bib.bib23)) and constraints in handling long contexts (Liu et al. [2024](https://arxiv.org/html/2408.04472v2#bib.bib20)), models often need to improve in adversarial and sustained debate scenarios (shown in Figure [1](https://arxiv.org/html/2408.04472v2#Sx1.F1 "Figure 1 ‣ Introduction ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate")), struggling to maintain competitiveness and argumentative consistency.

![Refer to caption](img/43497dc20ba71e19777ac0afe3474e5b.png)

Figure 1: Before and After: Agent4Debate’s impact on LLMs competitive debating skills.

To address these challenges, we propose a multi-agent framework based on LLMs, Agent for Debate (Agent4Debate). Agent4Debate features a dynamic, multi-agent collaborative architecture, leveraging the cooperation of multiple specialized LLMs to enable the framework to participate in multi-stage competitive debates. Inspired by human debate preparation processes, our framework incorporates four key agents, including Searcher, Analyzer, Writer, and Reviewer. To comprehensively evaluate the competitive debate capabilities of Agent4Debate, we establish the Competitive Debate Arena, employing an Elo ranking system widely used in competitive sports, ensuring fairness and scalability. This arena comprises 66 carefully selected Chinese debate motions, covering three categories (Abell [2018](https://arxiv.org/html/2408.04472v2#bib.bib1)), such as Policy, Value, and Fact, thoroughly testing the performance of participants across different types of debates. Participants include Agent4Debate with different foundation models, two baselines, and ten experienced human debaters. All participants engage in pairwise matches, with each debate assessed through two independent evaluation methods, including an automatic debate judging system based on the Debatrix (Liang et al. [2024](https://arxiv.org/html/2408.04472v2#bib.bib19)) metrics, and an expert judging system consisting of three professional human reviewers. Based on these two sets of independent evaluation results, we construct two separate Elo (Elo [1967](https://arxiv.org/html/2408.04472v2#bib.bib11); Zheng et al. [2023](https://arxiv.org/html/2408.04472v2#bib.bib44)) ranking lists, providing a multi-faceted quantitative assessment of participants’ performance across various debate motions. The experimental results from the arena demonstrate that Agent4Debate can achieve human-level performance in various types of competitive debates.

In conclusion, the main contributions of this work are as follows:

*   •

    We propose the Agent4Debate, which enhances the performance of LLMs in competitive debates through dynamic multi-agent collaboration. This framework mimics human debate team interactions, with agents adapting roles and strategies. Specifically, it employs the Searcher for information gathering, the Analyzer for strategic assessment, the Writer for argument formulation, and the Reviewer for critical evaluation.

*   •

    We construct the Competitive Debate Arena, a public resource comprising 66 Chinese debate motions and 200 debate matches across Policy, Value, and Fact categories. Human debaters are incorporated, and we establish Debatrix-Elo and Human-Elo rankings using Debatrix metrics and professional human judges, respectively.

*   •

    Our experimental results indicate that Agent4Debate’s performance in competitive debates is comparable to that of humans. Ablation studies validate the effectiveness of each component.

## Related Work

### Computational Argumentation

Argumentation research has deep historical roots (Walton, Reed, and Macagno [2008](https://arxiv.org/html/2408.04472v2#bib.bib37)), with its core objective being to achieve persuasion through logical reasoning and promote consensus among parties. In recent years, computational argumentation has emerged as an increasingly important field in natural language processing, with its main research directions encompassing argument mining (Lawrence and Reed [2019](https://arxiv.org/html/2408.04472v2#bib.bib16); Chen et al. [2024](https://arxiv.org/html/2408.04472v2#bib.bib8)), argument generation (Hua, Hu, and Wang [2019](https://arxiv.org/html/2408.04472v2#bib.bib12)), argument persuasiveness (Carlile et al. [2018](https://arxiv.org/html/2408.04472v2#bib.bib6)), and argument quality assessment (Wachsmuth et al. [2017b](https://arxiv.org/html/2408.04472v2#bib.bib36); Liang et al. [2024](https://arxiv.org/html/2408.04472v2#bib.bib19); Wachsmuth et al. [2024](https://arxiv.org/html/2408.04472v2#bib.bib34)). Project Debater (Slonim et al. [2021](https://arxiv.org/html/2408.04472v2#bib.bib29)), a debate system that integrates multiple modules, relies on retrieval-based methods rather than generative approaches for its argumentation. With the rise of Large Language Models research utilizing adversarial methods such as debate to enhance model capabilities (Du et al. [2023](https://arxiv.org/html/2408.04472v2#bib.bib9); Chang [2024](https://arxiv.org/html/2408.04472v2#bib.bib7)) has gradually attracted academic attention. Against this backdrop, our study focuses on competitive debate, a complex computational argumentation task that integrates multiple sub-tasks.

### LLM-based Agents

LLMs, such as ChatGPT (OpenAI [2023](https://arxiv.org/html/2408.04472v2#bib.bib22)), LLaMA (Touvron et al. [2023b](https://arxiv.org/html/2408.04472v2#bib.bib32), [a](https://arxiv.org/html/2408.04472v2#bib.bib31)), demonstrate powerful capabilities in instruction following and reasoning tasks. Harnessing these advanced capabilities, researchers have developed LLM-based agents, which mark a significant step forward in the field. These agents leverage the language understanding and generation abilities of models for more sophisticated tasks like multi-step reasoning and interactive problem-solving, as shown in recent studies (Wang et al. [2023b](https://arxiv.org/html/2408.04472v2#bib.bib39); Li et al. [2023](https://arxiv.org/html/2408.04472v2#bib.bib17)). They have various uses across different domains, such as software engineering (Qian et al. [2023](https://arxiv.org/html/2408.04472v2#bib.bib25)) and scientific inquiry (Boiko, MacKnight, and Gomes [2023](https://arxiv.org/html/2408.04472v2#bib.bib5)), highlighting their versatility. These agents can imitate complex human actions, partake in social interactions (Park et al. [2023](https://arxiv.org/html/2408.04472v2#bib.bib24); Tu et al. [2023](https://arxiv.org/html/2408.04472v2#bib.bib33)), and replicate intricate scenarios like elections (Argyle et al. [2022](https://arxiv.org/html/2408.04472v2#bib.bib2)), debates (Wang et al. [2023a](https://arxiv.org/html/2408.04472v2#bib.bib38); Du et al. [2023](https://arxiv.org/html/2408.04472v2#bib.bib9)), and consumer patterns (Wang et al. [2023c](https://arxiv.org/html/2408.04472v2#bib.bib40)), illustrating their capacity to emulate human social dynamics. While these agents demonstrate impressive capabilities in emulating human social dynamics, current research predominantly explores collaborative scenarios. However, competitive settings, though equally crucial in human interactions, remain comparatively underexplored.

## Task Definition

Competitive debate is a structured multi-turn interactive task. Each turn of statement can be regarded as a document-level text generation task, with a temporal and logical progression relationship between multiple turns. A typical debate has two opposing sides: the Pro side and the Con side. We represent the competitive debate as a sequence:

|  | $D=\{(s_{1},r_{1}),(s_{2},r_{2}),\cdots,(s_{n},r_{n})\}$ |  | (1) |

where $(s_{i},r_{i})$ denotes the $i$-th statement and its corresponding role, $s_{i}$ is the statement, and $r_{i}\in\{\textit{Pro},\textit{Con}\}$ represents the role of speaker. Each statement can be defined as:

|  | $s_{i}=\mathcal{G}(m,r_{i},D_{i-1})$ |  | (2) |

where $m$ is the motion of debate, $D_{i-1}$ represents the history of the first $i-1$ statements, and $\mathcal{G}(\cdot)$ is the generation function that produces each statement.

Typical competitive debate structure usually comprises three distinct stages, namely constructive arguments, rebuttals, and summary statements. To ensure fairness and simulate actual competitive debate conditions (Whitman [2005](https://arxiv.org/html/2408.04472v2#bib.bib41)), we establish specific rules for each stage:

![Refer to caption](img/d23698c30494fff47ec908f96eae1780.png)

Figure 2: Agent for Debate (Agent4Debate) Workflow: A dynamic framework simulating human debate team collaboration. From searching to reviewing, it showcases how four key roles (Searcher, Analyzer, Writer, Reviewer) interact and work iteratively. The right side illustrates the cyclical process from information gathering to argument formation using Stage 1 as an example, highlighting the framework’s multi-steps progression and recursive refinement.

*   •

    In Stage 1 (Constructive Arguments), both sides work independently, with the Con side unable to view the Pro’s constructive argument, ensuring initial viewpoints are uninfluenced.

*   •

    Stages 2 and 3 (Rebuttal and Summary) employ a progressive disclosure mechanism, where participants access all previous content to construct targeted statements.

*   •

    We alternate the sequence across stages to balance the advantages of speaking order. The Pro side speaks first in Stage 2, while the Con side leads in Stage 3.

## Agent for Debate

To address the challenges of hallucination and the difficulties in maintaining competitiveness and argumentative consistency in sustained debate scenarios, we propose the Agent for Debate (Agent4Debate) framework to enable LLMs to participate in competitive debates, as shown in Figure [2](https://arxiv.org/html/2408.04472v2#Sx3.F2 "Figure 2 ‣ Task Definition ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate"). This framework dynamically simulates human debate preparation through dialogue-based collaboration (Wu et al. [2023](https://arxiv.org/html/2408.04472v2#bib.bib42)) among four LLM-based agents, each mirroring key roles in a human debate team. The Searcher acts as a research assistant, gathering relevant information, while the Analyzer functions like an executive coach, strategizing and analyzing arguments. The Writer performs as a debater, crafting and articulating arguments, and the Reviewer serves as a debate coach, providing feedback and quality control. These agents interact flexibly throughout the debate process, adapting their roles and contributions based on the current stage and needs, much like a well-coordinated human debate team.

The collaboration in Agent4Debate is not just a simple sequence of steps, but rather a dynamic interaction between multiple agents, based on the debate stage and context. All the agents are equipped with customized prompts for different debate stages, enabling them to better adapt to and execute the specific tasks of the current stage. In the following sections, we introduce the functions of each agent in detail.

### Searcher Agent

The Searcher is a tool agent in the Agent4Debate framework, designed to effectively mitigate hallucination issues and address information timeliness problems that LLMs may encounter during debates. It achieves this by accessing and organizing information from external knowledge bases. The workflow of Searcher primarily involves decomposing search questions into more refined queries, then utilizing external tools (such as search engines or specialized knowledge bases) to retrieve relevant information, and finally systematically compiling and organizing the obtained answers. The information compiled by the Searcher forms a motion knowledge base, which is fixed and accessible to all agents for reference throughout the entire debate process. This approach ensures consistency and reliability of the information used in the debate. Note that, the Searcher plays different roles at various stages of the debate. In Stage 1, the Searcher uses the motion as the search question for information gathering. However, in Stage 2 and Stage 3, the Searcher switches to a passive mode, waiting for specific instructions from the Writer before conducting targeted searches.

### Analyzer Agent

The Analyzer is a core agent in the Agent4Debate framework, integrating real-time information from the debate and providing structured guidance for subsequent content output. Its primary function is to systematically analyze and plan the debate content based on the given motion, current stage, and historical context, thus bridging different phases of the debate. The workflow of Analyzer primarily involves breaking down the debate content step-by-step, drafting detailed outlines, and providing targeted strategic advice to other agents. This approach ensures coherence in debate reasoning and comprehensiveness in argumentation. Notably, the Analyzer plays different roles at various stages:

*   •

    In Stage 1, the Analyzer receives the debate topic and compiled materials from the Searcher. It then summarizes the motion and formulates definitions, judgment criteria, main arguments, and supporting evidence from its own perspective.

*   •

    In Stage 2, the Analyzer analyzes all content from previous phases, summarizing the differences in viewpoints between both sides, such as the opponent’s definitions and judgment criteria. It then suggests rebuttal techniques that can be used to address these differences.

*   •

    In Stage 3, in addition to continuing to summarize points of disagreement and provide rebuttal techniques, the Analyzer also offers suggestions from a value-based perspective, further enhancing the depth and persuasiveness of the debate.

### Writer Agent

The Writer is the executive agent in the Agent4Debate framework, responsible for transforming analysis and planning into actual debate content. Its primary function is to compose complete debate drafts based on the instructions and outlines provided by the Analyzer and to revise these drafts according to feedback from the Reviewer, ensuring the quality and persuasiveness of the debate. Workflow of the Writer primarily encompasses the following aspects:

*   •

    Content Creation: Based on the outline provided by the Analyzer, the Writer expands it into a detailed debate script, ensuring the logic of arguments and the sufficiency of supporting evidence.

*   •

    Revision and Refinement: Upon receiving modification suggestions from the Reviewer, the Writer makes corresponding adjustments and optimizations to the script to enhance its overall quality.

*   •

    Resource Assessment: The Writer evaluates whether the information in the current knowledge base is sufficient to support the requirements of the outline and script revisions. If information is found to need to be improved, the Writer proactively initiates requests to the Searcher, clearly specifying the additional materials needed.

### Reviewer Agent

The Reviewer is the quality control agent in the Agent4Debate framework, responsible for reviewing and evaluating debate scripts generated by the Writer. Its primary function is to provide targeted modification suggestions based on the current debate stage and historical context, ensuring the debate content’s quality, logic, and persuasiveness. The Reviewer’s workflow focuses on different aspects at various stages of the debate:

*   •

    In Stage 1, the Reviewer primarily concentrates on the completeness of the argument structure, the comprehensiveness of content (including definitions, criteria, and main points), the sufficiency of supporting evidence, and the fluency of expression.

*   •

    In Stage 2, building upon the previous stage, the Reviewer additionally focuses on the appropriate application of rebuttal techniques and ensures that rebuttals to the opponent’s arguments do not lead to self-contradiction in one’s stance.

*   •

    In Stage 3, besides addressing the content from the previous two stages, the Reviewer also assesses the depth of the debate content and makes a judgment based on the context, providing detailed reasons for this assessment.

The Reviewer maintains argumentative coherence by continuously assessing consistency with previously presented information across all debate stages. This process involves providing feedback and modification suggestions to the Writer, and facilitating targeted revisions. The review-revision cycle persists iteratively until the script meets the Reviewer’s quality standards.

## Experimental Setup

### Experimental Subjects

Our experimental design involves three types of participants, including the baseline framework, Agent4Debate based on different LLMs, and human participants. For all models, we set temperature to 0.2 and Top P to 0.75, with no other parameters adjusted.

#### Baseline

We adopt the benchmark framework of AI-Debater 2024 competition¹¹1http://www.fudan-disc.com/sharedtask/AIDebater24, incorporating Tavily²²2https://tavily.com as the search engine and stage-specific prompts. We uses Claude-3.5-sonnet and Deepseek-Chat (Bi et al. [2024](https://arxiv.org/html/2408.04472v2#bib.bib4)) as the foundation models.

#### Agent4Debate

To comprehensively evaluate the generalization capability of Agent4Debate and conduct more in-depth comparative experiments, we select a variety of advanced LLMs as the foundation for Agent4Debate. These models include Claude-3.5-sonnet, GPT-4o (OpenAI [2023](https://arxiv.org/html/2408.04472v2#bib.bib22)), and Gemini-1.5-Pro/Flash (Reid et al. [2024](https://arxiv.org/html/2408.04472v2#bib.bib28)), all of which have demonstrated excellent performance in various evaluations (Zheng et al. [2023](https://arxiv.org/html/2408.04472v2#bib.bib44)). Considering that our study focuses on Chinese competitive debate, we specifically incorporate several LLMs that excel in Chinese language processing, including Qwen2-72b-Instruct (Yang et al. [2024](https://arxiv.org/html/2408.04472v2#bib.bib43)), Deepseek-Chat , and GLM-4-Air. Switching models in Agent4Debate experiments updates all components accordingly. In all experiments, the Searcher used Tavily as the search engine.

#### Humans

We recruit ten experienced debaters for our experiment to validate the performance of Agent4Debate against humans in competitive debate. Each debater has with 2-4 years of debate team training and at least one year of Chinese competitive debate experience. They are informed that they will be debating against artificial intelligence and are given 2 days of preparation time for each motion. To ensure effective communication, we use the Whisper model (Radford et al. [2023](https://arxiv.org/html/2408.04472v2#bib.bib26)) to transcribe human speeches into text while the human debaters read the model’s output directly. This design ensures accurate information transfer and provides human debaters ample time for reflection and response. These debaters participate only in the debates, not in other research activities.

### Metrics

#### Debatrix

Debatrix (Liang et al. [2024](https://arxiv.org/html/2408.04472v2#bib.bib19)) is a multi-turn debate evaluation method based on LLMs. It comprehensively assesses debates by considering the chronological order of statements and evaluating them along three dimensions, each described in natural language: Argument (A), Source (S), and Language (L). These natural language evaluations are then integrated to form an Overall (O) assessment, ultimately determining the winner. In our implementation, we convert each dimension’s descriptive result into a ternary outcome (win, lose, or tie). This evaluation approach is particularly well-suited for our multi-turn, document-level competitive debate scenarios. In our experiments, we employ GPT-4o-mini as the foundational model for Debatrix. To ensure the reliability of the assessment, we conduct three independent evaluations using Debatrix for each debate, ultimately deriving the final scores.

#### Human

We invite three experienced Chinese competitive debate judges to participate in this study. Each judge possesses 3-5 years of experience in Chinese competitive debates and has coached university debate teams. The judges independently assess each debate, casting a vote for win, lose, or tie, with the outcome determined by majority rule. To maintain impartiality, judges are only informed that both sides have an equal burden of proof without receiving any additional context. It is important to note that all judges are external to the research development process and do not have backgrounds in computer science, thereby minimizing potential biases.

### Competitive Debate Arena

To comprehensively assess the abilities of Agent4Debate, Baseline, and Humans in competitive debate, we establish the Competitive Debate Arena. This arena is designed to provide a fair and extensible evaluation environment, covering various types of debate motions and assessment methods. We carefully select 66 debate motions from major Chinese debate competitions over the past decade, including Chinese Debate World Cup, The World Mandarin Debating Championship, and International Chinese Debating Competition. These motions cover three main categories (Abell [2018](https://arxiv.org/html/2408.04472v2#bib.bib1)), including Value, Fact, and Policy. Fact makes statements or comparisons about testable aspects of the natural world, Value assigns value or judgment to certain things or concepts, while Policy typically suggests action plans through proposed changes.

In terms of evaluation methods, we adopt two independent review approaches, where one uses the Debatrix based on LLMs for assessment, and the other involves judgments by experienced human reviewers. These review methods are completely independent, each producing separate results. Based on these review methods, we construct two ranking systems, including Debatrix-Elo and Human-Elo. To build these ranking systems, we draw inspiration from the Chatbot Arena (Zheng et al. [2023](https://arxiv.org/html/2408.04472v2#bib.bib44)) approach and adopt an improved version of the Bradley-Terry (BT) model (Hunter [2004](https://arxiv.org/html/2408.04472v2#bib.bib13); Rafailov et al. [2024](https://arxiv.org/html/2408.04472v2#bib.bib27)) to calculate Elo scores. The traditional BT model uses the following formula to calculate the probability of Participant A winning over Participant B:

|  | $P(A>B)=\frac{e^{\gamma_{A}}}{e^{\gamma_{A}}+e^{\gamma_{B}}}$ |  | (3) |

where $\gamma_{A}$ and $\gamma_{B}$ represent the ability parameters of A and B, respectively.

However, considering that our review system (whether Debatrix or human reviewers) independently provides three scores, we improve the traditional model by introducing a weight function based on score differences:

|  | $w_{i}=\frac{1}{1+e^{-&#124;\text{score}_{A_{i}}-\text{score}_{B_{i}}&#124;}}$ |  | (4) |

where $\text{score}\in[0,3]$. This weight function adjusts the importance of each match in the final ranking, making the ranking calculation more precise. Based on this weight function, our likelihood function becomes:

|  | $\mathcal{L}=\prod_{i=1}^{n}P(A_{i}>B_{i})^{w_{i}}$ |  | (5) |

By maximizing this likelihood function, we can obtain more accurate ability parameter estimates, thus constructing a more precise ranking system.

Our improved Elo system not only effectively reflects participants’ overall performance in multiple matchups but also allows for more nuanced adjustments based on the specifics of each match. Using two independent review methods and ranking systems, we can better understand the performance of participants and compare potential differences between Debatrix and human reviews. Furthermore, this Elo system is scalable, efficiently incorporating new frameworks or models for ongoing comparative analysis.

## Experimental Results

### Agent4Debate vs. Baseline

We conduct a comparative performance evaluation of Agent4Debate against the baselines. Each framework participates in 20 debate matches, including five different motions. To ensure fairness, the number of times each framework argued for the Pro and Con sides is balanced. Debatrix is employed as the evaluation criteria. Debatrix scoring is applied three times for each debate, with 1 point awarded for each win in the dimensions of Argument (A), Language (L), Source (S), and Overall (O) performance. In the case of a tie, both sides are awarded 0.5 points.

| Model | Framework | Debatrix |
| --- | --- | --- |
| S | L | A | O |
| --- | --- | --- | --- |
| Claude-3.5-sonnet | Agent4Debate | 2.83 | 1.76 | 2.52 | 2.62 |
| Baseline | 0.17 | 1.24 | 0.48 | 0.38 |
| Deepseek-Chat | Agent4Debate | 2.73 | 1.88 | 2.31 | 2.77 |
| Baseline | 0.27 | 1.12 | 0.69 | 0.23 |

Table 1: Comparison of Agent4Debate and Baseline.

As shown in Table [1](https://arxiv.org/html/2408.04472v2#Sx6.T1 "Table 1 ‣ Agent4Debate vs. Baseline ‣ Experimental Results ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate"), Agent4Debate enhances the competitive debating performance across both models. For Claude-3.5-sonnet, the Overall score improves from 0.38 to 2.62, while for Deepseek-Chat, it increases from 0.23 to 2.77\. These results demonstrate that the Agent4Debate framework effectively enhances the performance of language models of varying scales and types in competitive debate tasks. Among all metrics, Source shows improvement. This can be attributed to the Searcher Agent and Analyzer Agent within Agent4Debate, which conducts an in-depth analysis of debate motions and systematic organization of materials, utilizing external knowledge more effectively than the simple search approach from baseline. The Language shows relatively modest improvement, reflecting robust generation capabilities of LLMs, leaving limited room for enhancement.

Comparing the results between Claude-3.5-sonnet and Deepseek-Chat, it is observed that Agent4Debate yields more pronounced performance improvements for the more powerful model, particularly in the Argument and Overall metrics. This may be due to more advanced models possessing stronger reasoning abilities and better instruction-following capabilities (Kaplan et al. [2020](https://arxiv.org/html/2408.04472v2#bib.bib15)), thus exhibiting superior adaptability to complex frameworks.

### Ablation Study

To evaluate the contribution of each agent within Agent4Debate, we conduct a series of ablation studies. The experimental setup remains consistent with the previous comparative experiments. Each ablation configuration engages in 20 debates across five motions, with a balanced distribution of the Pro and Con sides. The evaluation continues to employ Debatrix, with the scoring method identical to that of the comparative experiments. We do not perform an ablation experiment on the Writer Agent, as it is responsible for the text generation at every stage. The foundation model for the ablation study is Claude-3.5-sonnet.

| Framework | Debatrix |
| --- | --- |
| S | L | A | O |
| --- | --- | --- | --- |
| Agent4Debate | 2.79 | 1.54 | 2.01 | 2.12 |
| w/o Searcher | 0.21 | 1.46 | 0.99 | 0.88 |
| Agent4Debate | 1.83 | 1.50 | 1.79 | 1.76 |
| w/o Analyzer | 1.17 | 1.50 | 1.21 | 1.24 |
| Agent4Debate | 1.74 | 1.67 | 2.13 | 1.93 |
| w/o Reviewer | 1.26 | 1.33 | 0.87 | 1.07 |

Table 2: The results of ablation study. The foundation model for the ablation study is Claude-3.5-sonnet.

Table [2](https://arxiv.org/html/2408.04472v2#Sx6.T2 "Table 2 ‣ Ablation Study ‣ Experimental Results ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate") presents the detailed results of our ablation study, clearly illustrating the impact of removing each agent. The experimental results demonstrate that each agent in the Agent4Debate framework contributes to the overall performance. When we remove any agent, the Overall score decreases, confirming the necessity of each component. Specifically, removing the Analyzer reduces the Overall score from 2.12 to 1.76\. Its impact on the Source and Argument metrics is particularly notable, with the Source score dropping from 2.79 to 1.83 and the Argument score from 2.01 to 1.79\. This indicates the Analyzer’s crucial role in the formulation of material analysis, argument refinement, and rebuttal strategy. The absence of the Searcher results in a dramatic drop in the Source score from 2.79 to 0.21, while the Overall score falls from 2.12 to 0.88\. This highlights the importance of appropriately searching and organizing external knowledge to enhance debate performance. The removal of the Reviewer has a smaller impact on overall performance (Overall score decreases from 2.12 to 1.93). However, its primary function of reviewing drafts, suggesting revisions, and improving the output quality of Agent4Debate aligns with the framework’s design expectations.

### Results of Competitive Debate Arena

We collect records of 200 debate matches (excluding those from comparison experiments and ablation studies), covering 66 debate motions across three categories, including Fact, Value, and Policy. Participants included in Agent4Debate using different foundation models, two baselines, and ten human debaters, all of which are engaged in randomly paired competitions. Each debate is independently assessed using both the Debatrix and human judges. Utilizing the improved BT model in Eq. [5](https://arxiv.org/html/2408.04472v2#Sx5.E5 "In Competitive Debate Arena ‣ Experimental Setup ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate"), we calculate Elo scores for all 200 matches and sub-Elo scores for each of the three debate categories. The experimental results are presented in two independent ranking systems, consisting of Debatrix-Elo (Table [3](https://arxiv.org/html/2408.04472v2#Sx6.T3 "Table 3 ‣ Results of Competitive Debate Arena ‣ Experimental Results ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate")) and Human-Elo (Table [4](https://arxiv.org/html/2408.04472v2#Sx6.T4 "Table 4 ‣ Results of Competitive Debate Arena ‣ Experimental Results ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate")).

| Model | Full | Fact | Policy | Value |
| --- | --- | --- | --- | --- |
| Gemini-1.5-Pro | 1034.15 | 1154.93 | 1231.98 | 1075.30 |
| Claude-3.5-sonnet | 1032.51 | 1159.18 | 1224.19 | 1074.33 |
| Qwen2-72b-Instruct | 1023.31 | 1130.83 | 1179.62 | 1081.75 |
| GPT-4o | 1022.21 | 1150.14 | 1137.49 | 1069.55 |
| Gemini-1.5-Flash | 1012.45 | 1136.21 | 1156.50 | 1057.73 |
| GLM-4-Air | 1011.72 | 1155.07 | 1148.53 | 1048.42 |
| Deepseek-chat | 1004.00 | 1118.98 | 1131.16 | 1054.89 |
| Claude-3.5-sonnet^∗ | 982.07 | 479.50 | 956.21 | 1021.44 |
| Human | 978.35 | 1109.73 | 515.57 | 953.05 |
| Deepseek-Chat^∗ | 954.34 | 491.13 | 478.78 | 983.99 |

Table 3: Debatrix-Elo Ranking. ^∗ denotes baseline models, unmarked models are Agent4Debate foundation models.

| Model | Full | Fact | Policy | Value |
| --- | --- | --- | --- | --- |
| Gemini-1.5-Pro | 1040.64 | 1110.23 | 1104.79 | 1048.10 |
| Claude-3.5-sonnet | 1031.15 | 1093.87 | 1104.44 | 1020.05 |
| GPT-4o | 1028.84 | 1086.78 | 1099.63 | 1033.09 |
| Human | 1006.46 | 1055.82 | 1030.32 | 1006.57 |
| Gemini-1.5-Flash | 1000.00 | 1037.45 | 997.66 | 1003.29 |
| Qwen2-72b-Instruct | 999.70 | 1041.10 | 976.16 | 1005.56 |
| Claude-3.5-sonnet^∗ | 991.38 | 1023.29 | 968.34 | 997.47 |
| GLM-4-Air | 972.48 | 940.00 | 948.31 | 996.67 |
| Deepseek-chat | 971.94 | 963.05 | 946.30 | 986.79 |
| Deepseek-Chat^∗ | 962.61 | 786.44 | 911.33 | 979.29 |

Table 4: Human-Elo Ranking. ^∗ denotes baseline models, unmarked models are Agent4Debate foundation models.

Drawing from the experimental results presented in Tables [3](https://arxiv.org/html/2408.04472v2#Sx6.T3 "Table 3 ‣ Results of Competitive Debate Arena ‣ Experimental Results ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate") and [4](https://arxiv.org/html/2408.04472v2#Sx6.T4 "Table 4 ‣ Results of Competitive Debate Arena ‣ Experimental Results ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate"), we can derive the following insights. (1) Agent4Debate, especially those using advanced foundation models such as Gemini-1.5-Pro and Claude-3.5-sonnet, demonstrate performance comparable to or surpassing human debaters in Debatrix-Elo and Human-Elo rankings. The top-performing Agent4Debate (Gemini-1.5-Pro) consistently ranks first, scoring 1044.18 in Debatrix-Elo and 1040.64 in Human-Elo. Experimental results indicate that models with more robust reasoning and instruction-following capabilities perform better within the Agent4Debate framework. (2) In Debatrix-Elo, most models show score variations across the Fact, Policy, and Value categories. In contrast, Human-Elo displays more consistent scores for each model across categories. This disparity may arise because Debatrix considers Source, Language, and Argument dimensions, while human judges likely focus more on logic and rebuttal techniques. Debatrix-Elo and Human-Elo show high consistency in model rankings, particularly for top-performing models. However, human performance is ranked differently in the two rankings. In Debatrix-Elo, humans rank 8th with a score of 978.35, while in Human-Elo, they rank 4th with a score of 1006.46\. This suggests that Debatrix-Elo may underestimate human performance. This underestimation is partly due to the different evaluation tendencies between Debatrix and human judges, and partly because human speech quality deteriorates when transcribed to text. (3) In Debatrix-Elo, certain models excel in specific categories. This is due to differences in the argumentation processes for the three types of debate motions: Policy debates typically require extensive evidence to demonstrate policy necessity and effectiveness; Value debates often demand more substantial logical reasoning and expressive skills; Fact debates combine characteristics of both. These distinctions, reflected in Debatrix’s multi-dimensional evaluation, yield varying results.

#### Agent4Debate vs. Humans

| Model | Debatrix | Human |
| --- | --- | --- |
| S | L | A | O |
| --- | --- | --- | --- |
| Human | 0.52 | 0.30 | 0.6 | 0.42 | 1.22 |
| Agent4Debate | 2.48 | 2.70 | 2.40 | 2.58 | 1.78 |

Table 5: Comparison of Human and Agent4Debate

We conduct a separate analysis of 30 debates between Agent4Debate and human debaters. In these debates, to ensure comprehensive experimentation, all foundation models of Agent4Debate participate. The scoring results from the Debatrix system and human judges are presented in Table [5](https://arxiv.org/html/2408.04472v2#Sx6.T5 "Table 5 ‣ Agent4Debate vs. Humans ‣ Results of Competitive Debate Arena ‣ Experimental Results ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate").

Debatrix for human performance is lower than human judges across three dimensions. This discrepancy may stem from several factors. Regarding Source, human debaters use voice input, which is then transcribed into text. People typically do not directly cite references in oral debates, leading to lower scores. The Language score is the lowest, possibly due to oral expressions often containing verbal tics and informal language, coupled with imperfect voice-to-text transcription accuracy, affecting language quality assessment. The low Argument score may be a cascading effect of the previous two low scores, thus impacting Debatrix’s overall understanding and evaluation of human input. In contrast, human judges employ different criteria when evaluating competitive debates. They usually prioritize core factors such as logical reasoning and debating skills, only considering other aspects when these primary elements are challenging to distinguish. This approach to judgment differs from the Debatrix.

#### Consistency

To analyze the differences between Debatrix and human evaluations, we conduct a consistency analysis. Consistency is calculated by comparing the result between human and Debatrix, with tie considered consistent outcomes. Table [6](https://arxiv.org/html/2408.04472v2#Sx6.T6 "Table 6 ‣ Consistency ‣ Results of Competitive Debate Arena ‣ Experimental Results ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate") presents the results, showing that internal consistency among human reviewers remains stable across all matches, while the consistency between Debatrix and human reviewers varies when including or excluding human debaters. These findings further corroborate the above observations. These findings suggest that while Debatrix shows differences from human reviewers in evaluating debates between humans and models, it still provides valuable insights, particularly in assessing model-to-model debates. In these cases, Debatrix offers multi-faceted analytical results that contribute to our understanding of models’ comprehensive capabilities in competitive debates.

| Consistency | Excluding Human Debates | All Debates |
| --- | --- | --- |
| Debatrix vs. Human | 0.66 | 0.56 |
| Among Human | 0.74 | 0.73 |

Table 6: Consistency between Debatrix and Human Judges

We further analyze the Elo rankings and Agent4Debate in the Appendix.

## Conclusion

We propose a dynamic multi-agent framework, Agent for Debate (Agent4Debate), to enable LLMs to participate in competitive debates. To evaluate the performance of Agent4Debate, we construct the Competitive Debate Arena, comprising 66 classic Chinese debate motions. We recruit ten human debaters and collect 200 debate matches involving Agent4Debate, baselines, and human debaters. Using the Debatrix and human judges for evaluation, we propose Debatrix-Elo and Human-Elo rankings. Experimental results show that our state-of-the-art Agent4Debate exhibits capabilities comparable to those of humans in competitive debates. Ablation studies prove the effectiveness of each component in the agent structure.

## References

*   Abell (2018) Abell, J. 2018. Value, Fact, and Policy Resolutions.
*   Argyle et al. (2022) Argyle, L. P.; Busby, E. C.; Fulda, N.; Gubler, J.; Rytting, C.; and Wingate, D. 2022. Out of One, Many: Using Language Models to Simulate Human Samples. In *Proceedings of the 60th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, 819–862.
*   Atkinson et al. (2017) Atkinson, K.; Baroni, P.; Giacomin, M.; Hunter, A.; Prakken, H.; Reed, C.; Simari, G. R.; Thimm, M.; and Villata, S. 2017. Towards Artificial Argumentation. *AI Mag.*, 38: 25–36.
*   Bi et al. (2024) Bi, X.; Chen, D.; Chen, G.; Chen, S.; Dai, D.; Deng, C.; Ding, H.; Dong, K.; Du, Q.; Fu, Z.; et al. 2024. Deepseek llm: Scaling open-source language models with longtermism. *arXiv preprint arXiv:2401.02954*.
*   Boiko, MacKnight, and Gomes (2023) Boiko, D. A.; MacKnight, R.; and Gomes, G. 2023. Emergent Autonomous Scientific Research Capabilities of Large Language Models. *arXiv preprint arXiv:2304.05332*.
*   Carlile et al. (2018) Carlile, W.; Gurrapadi, N.; Ke, Z.; and Ng, V. 2018. Give Me More Feedback: Annotating Argument Persuasiveness and Related Attributes in Student Essays. In Gurevych, I.; and Miyao, Y., eds., *Proceedings of the 56th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, 621–631\. Melbourne, Australia: Association for Computational Linguistics.
*   Chang (2024) Chang, E. Y. 2024. SocraSynth: Multi-LLM Reasoning with Conditional Statistics. arXiv:2402.06634.
*   Chen et al. (2024) Chen, G.; Cheng, L.; Tuan, L. A.; and Bing, L. 2024. Exploring the Potential of Large Language Models in Computational Argumentation. arXiv:2311.09022.
*   Du et al. (2023) Du, Y.; Li, S.; Torralba, A.; Tenenbaum, J. B.; and Mordatch, I. 2023. Improving Factuality and Reasoning in Language Models through Multiagent Debate. arXiv:2305.14325.
*   Eger, Daxenberger, and Gurevych (2017) Eger, S.; Daxenberger, J.; and Gurevych, I. 2017. Neural end-to-end learning for computational argumentation mining. *arXiv preprint arXiv:1704.06104*.
*   Elo (1967) Elo, A. E. 1967. The proposed uscf rating system, its development, theory, and applications. *Chess life*, 22(8): 242–247.
*   Hua, Hu, and Wang (2019) Hua, X.; Hu, Z.; and Wang, L. 2019. Argument Generation with Retrieval, Planning, and Realization. arXiv:1906.03717.
*   Hunter (2004) Hunter, D. R. 2004. MM algorithms for generalized Bradley-Terry models. *The annals of statistics*, 32(1): 384–406.
*   Ji et al. (2023) Ji, Z.; Lee, N.; Frieske, R.; Yu, T.; Su, D.; Xu, Y.; Ishii, E.; Bang, Y. J.; Madotto, A.; and Fung, P. 2023. Survey of hallucination in natural language generation. *ACM Computing Surveys*, 55(12): 1–38.
*   Kaplan et al. (2020) Kaplan, J.; McCandlish, S.; Henighan, T.; Brown, T. B.; Chess, B.; Child, R.; Gray, S.; Radford, A.; Wu, J.; and Amodei, D. 2020. Scaling Laws for Neural Language Models. arXiv:2001.08361.
*   Lawrence and Reed (2019) Lawrence, J.; and Reed, C. 2019. Argument Mining: A Survey. *Computational Linguistics*, 45(4): 765–818.
*   Li et al. (2023) Li, G.; Hammoud, H. A. A. K.; Itani, H.; Khizbullin, D.; and Ghanem, B. 2023. CAMEL: Communicative Agents for ”Mind” Exploration of Large Scale Language Model Society. *arXiv preprint arXiv:2303.17760*.
*   Li, Ji, and Han (2021) Li, S.; Ji, H.; and Han, J. 2021. Document-level event argument extraction by conditional generation. *arXiv preprint arXiv:2104.05919*.
*   Liang et al. (2024) Liang, J.; Ye, R.; Han, M.; Lai, R.; Zhang, X.; Huang, X.; and Wei, Z. 2024. Debatrix: Multi-dimensinal Debate Judge with Iterative Chronological Analysis Based on LLM. *arXiv preprint arXiv:2403.08010*.
*   Liu et al. (2024) Liu, N. F.; Lin, K.; Hewitt, J.; Paranjape, A.; Bevilacqua, M.; Petroni, F.; and Liang, P. 2024. Lost in the middle: How language models use long contexts. *Transactions of the Association for Computational Linguistics*, 12: 157–173.
*   Nichols (1936) Nichols, E. R. 1936. A historical sketch of intercollegiate debating: I. *Quarterly Journal of Speech*, 22(2): 213–220.
*   OpenAI (2023) OpenAI, R. 2023. GPT-4 technical report. *arXiv*, arXiv preprint arXiv:2303.08774.
*   Ouyang et al. (2022) Ouyang, L.; Wu, J.; Jiang, X.; Almeida, D.; Wainwright, C.; Mishkin, P.; Zhang, C.; Agarwal, S.; Slama, K.; Ray, A.; et al. 2022. Training language models to follow instructions with human feedback. *Advances in neural information processing systems*, 35: 27730–27744.
*   Park et al. (2023) Park, J. S.; O’Brien, J. C.; Cai, C. J.; Morris, M. R.; Liang, P.; and Bernstein, M. S. 2023. Generative Agents: Interactive Simulacra of Human Behavior. *arXiv preprint arXiv:2304.03442*.
*   Qian et al. (2023) Qian, C.; Cong, X.; Yang, C.; Chen, W.; Su, Y.; Xu, J.; Liu, Z.; and Sun, M. 2023. Communicative Agents for Software Development. *arXiv preprint arXiv:2207.07924*.
*   Radford et al. (2023) Radford, A.; Kim, J. W.; Xu, T.; Brockman, G.; McLeavey, C.; and Sutskever, I. 2023. Robust speech recognition via large-scale weak supervision. In *International conference on machine learning*, 28492–28518\. PMLR.
*   Rafailov et al. (2024) Rafailov, R.; Sharma, A.; Mitchell, E.; Ermon, S.; Manning, C. D.; and Finn, C. 2024. Direct Preference Optimization: Your Language Model is Secretly a Reward Model. arXiv:2305.18290.
*   Reid et al. (2024) Reid, M.; Savinov, N.; Teplyashin, D.; Lepikhin, D.; Lillicrap, T.; Alayrac, J.-b.; Soricut, R.; Lazaridou, A.; Firat, O.; Schrittwieser, J.; et al. 2024. Gemini 1.5: Unlocking multimodal understanding across millions of tokens of context. *arXiv preprint arXiv:2403.05530*.
*   Slonim et al. (2021) Slonim, N.; Bilu, Y.; Alzate, C.; Bar-Haim, R.; Bogin, B.; Bonin, F.; Choshen, L.; Cohen-Karlik, E.; Dankin, L.; Edelstein, L.; et al. 2021. An autonomous debating system. *Nature*, 591(7850): 379–384.
*   Thueblood (1926) Thueblood, T. C. 1926. A chapter on the organization of college courses in public speaking. *Quarterly Journal of Speech*, 12(1): 1–11.
*   Touvron et al. (2023a) Touvron, H.; Martin, L.; Stone, K.; Albert, P.; Almahairi, A.; Babaei, Y.; Bashlykov, N.; Batra, S.; Bhargava, P.; Bhosale, S.; Bikel, D.; Blecher, L.; Ferrer, C. C.; Chen, M.; Cucurull, G.; Esiobu, D.; Fernandes, J.; Fu, J.; Fu, W.; Fuller, B.; Gao, C.; Goswami, V.; Goyal, N.; Hartshorn, A.; Hosseini, S.; Hou, R.; Inan, H.; Kardas, M.; Kerkez, V.; Khabsa, M.; Kloumann, I.; Korenev, A.; Koura, P. S.; Lachaux, M.-A.; Lavril, T.; Lee, J.; Liskovich, D.; Lu, Y.; Mao, Y.; Martinet, X.; Mihaylov, T.; Mishra, P.; Molybog, I.; Nie, Y.; Poulton, A.; Reizenstein, J.; Rungta, R.; Saladi, K.; Schelten, A.; Silva, R.; Smith, E. M.; Subramanian, R.; Tan, X. E.; Tang, B.; Taylor, R.; Williams, A.; Kuan, J. X.; Xu, P.; Yan, Z.; Zarov, I.; Zhang, Y.; Fan, A.; Kambadur, M.; Narang, S.; Rodriguez, A.; Stojnic, R.; Edunov, S.; and Scialom, T. 2023a. Llama 2: Open Foundation and Fine-Tuned Chat Models. arXiv:2307.09288.
*   Touvron et al. (2023b) Touvron, H.; Martin, L.; Stone, K.; Albert, P.; Almahairi, A.; Babaei, Y.; Bashlykov, N.; Batra, S.; Bhargava, P.; Bhosale, S.; et al. 2023b. Llama 2: Open foundation and fine-tuned chat models. *arXiv preprint arXiv:2307.09288*.
*   Tu et al. (2023) Tu, Q.; Chen, C.; Li, J.; Li, Y.; Shang, S.; Zhao, D.; Wang, R.; and Yan, R. 2023. CharacterChat: Learning towards Conversational AI with Personalized Social Support. *arXiv preprint arXiv:2308.10278*.
*   Wachsmuth et al. (2024) Wachsmuth, H.; Lapesa, G.; Cabrio, E.; Lauscher, A.; Park, J.; Vecchi, E. M.; Villata, S.; and Ziegenbein, T. 2024. Argument Quality Assessment in the Age of Instruction-Following Large Language Models. arXiv:2403.16084.
*   Wachsmuth et al. (2017a) Wachsmuth, H.; Naderi, N.; Hou, Y.; Bilu, Y.; Prabhakaran, V.; Thijm, T. A.; Hirst, G.; and Stein, B. 2017a. Computational argumentation quality assessment in natural language. In *Proceedings of the 15th Conference of the European Chapter of the Association for Computational Linguistics: Volume 1, Long Papers*, 176–187.
*   Wachsmuth et al. (2017b) Wachsmuth, H.; Naderi, N.; Hou, Y.; Bilu, Y.; Prabhakaran, V.; Thijm, T. A.; Hirst, G.; and Stein, B. 2017b. Computational Argumentation Quality Assessment in Natural Language. In Lapata, M.; Blunsom, P.; and Koller, A., eds., *Proceedings of the 15th Conference of the European Chapter of the Association for Computational Linguistics: Volume 1, Long Papers*, 176–187\. Valencia, Spain: Association for Computational Linguistics.
*   Walton, Reed, and Macagno (2008) Walton, D.; Reed, C.; and Macagno, F. 2008. Argumentation Schemes. In *Computer Science, Psychology*.
*   Wang et al. (2023a) Wang, H.; Du, X.; Yu, W.; Chen, Q.; Zhu, K.; Chu, Z.; Yan, L.; and Guan, Y. 2023a. Apollo’s Oracle: Retrieval-Augmented Reasoning in Multi-Agent Debates. *arXiv preprint arXiv:2312.04854*.
*   Wang et al. (2023b) Wang, L.; Ma, C.; Feng, X.; Zhang, Z.; Yang, H.; Zhang, J.; Chen, Z.; Tang, J.; Chen, X.; Lin, Y.; Zhao, W. X.; Wei, Z.; and Wen, J.-R. 2023b. A Survey on Large Language Model Based Autonomous Agents. *arXiv preprint arXiv:2308.11432*.
*   Wang et al. (2023c) Wang, L.; Zhang, J.; Yang, H.; Chen, Z.; Tang, J.; Zhang, Z.; Chen, X.; Lin, Y.; Song, R.; Zhao, W. X.; Xu, J.; Dou, Z.; Wang, J.; and Wen, J.-R. 2023c. When Large Language Model Based Agent Meets User Behavior Analysis: A Novel User Simulation Paradigm. *arXiv preprint arXiv:2306.02552*.
*   Whitman (2005) Whitman, G. 2005. Formats of Debate. https://www.csun.edu/~dgw61315/debformats.html. Accessed on August 04, 2024.
*   Wu et al. (2023) Wu, Q.; Bansal, G.; Zhang, J.; Wu, Y.; Li, B.; Zhu, E.; Jiang, L.; Zhang, X.; Zhang, S.; Liu, J.; Awadallah, A. H.; White, R. W.; Burger, D.; and Wang, C. 2023. AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation. arXiv:2308.08155.
*   Yang et al. (2024) Yang, A.; Yang, B.; Hui, B.; Zheng, B.; Yu, B.; Zhou, C.; Li, C.; Li, C.; Liu, D.; Huang, F.; et al. 2024. Qwen2 technical report. *arXiv preprint arXiv:2407.10671*.
*   Zheng et al. (2023) Zheng, L.; Chiang, W.-L.; Sheng, Y.; Zhuang, S.; Wu, Z.; Zhuang, Y.; Lin, Z.; Li, Z.; Li, D.; Xing, E. P.; Zhang, H.; Gonzalez, J. E.; and Stoica, I. 2023. Judging LLM-as-a-judge with MT-Bench and Chatbot Arena. arXiv:2306.05685.

## Appendix

### Further Analysis of Elo Rankings

Competitive Debate Arena currently includes only Chinese debate records. This choice is based on the availability of professional Chinese debate judges, ensuring the reliability of our Elo ranking system. However, Agent4Debate is designed to support competitive debates in other languages, including English, using the same structural framework. Expanding to multilingual debates would require only minor adjustments to the language constraints in the prompts. Future research may explore the implementation of debates in various languages.

| Model | Debatrix-CI | Human-CI |
| --- | --- | --- |
| Gemini-1.5-Pro | + 69/- 18 | + 57/- 14 |
| Claude-3.5-sonnet | + 67/- 18 | + 54/- 14 |
| Qwen2-72b-instruct | + 66/- 20 | + 54/- 15 |
| GPT-4o | + 66/- 20 | + 58/- 14 |
| Gemini-1.5-flash | + 66/- 18 | + 53/- 15 |
| GLM-4-Air | + 67/- 16 | + 48/- 19 |
| Deepseek-chat | + 64/- 22 | + 44/- 19 |
| baseline (Claude-3.5-sonnet) | + 69/- 36 | + 51/- 31 |
| Human | + 64/- 550 | + 50/- 22 |
| baseline (Deepseek-chat) | + 62/- 530 | + 32/- 520 |

Table 7: 95% Confidence Intervals

Table [7](https://arxiv.org/html/2408.04472v2#Ax1.T7 "Table 7 ‣ Further Analysis of Elo Rankings ‣ Appendix ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate") presents the 95% confidence intervals (CI) for various models and human performance, as evaluated by both the Debatrix-Elo and Human-Elo rankings. The CIs are expressed as upper and lower bounds relative to the median scores. Based on experimental results, we estimate that new debate models or frameworks can achieve a relatively stable ranking after about 15 debates. This allows for quick assessment of their competitive debate performance. Models with lower win rates show wider CIs (like baseline (Deepseek-Chat)), especially in the lower bound. However, this does not significantly affect the evaluation of their debate performance. The wider CIs mainly reflect the increased uncertainty in precise ranking for these models.

The Elo system can be used to estimate the winning rate of competitive debates between models. Figure [3](https://arxiv.org/html/2408.04472v2#Ax1.F3 "Figure 3 ‣ Further Analysis of Elo Rankings ‣ Appendix ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate") presents the win rate calculated using the Debatrix-elo and Human-elo rankings, respectively.

![Refer to caption](img/98f6f6c353c88ffc52e513779289f268.png)

Figure 3: Predicted Win Rates Using Elo Rankings for Model A in A vs. B Battles.

### Further Analysis of Agent4Debate

As shown in Table [8](https://arxiv.org/html/2408.04472v2#Ax1.T8 "Table 8 ‣ Further Analysis of Agent4Debate ‣ Appendix ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate"), the interaction frequency data reveals distinct patterns across debate stages. The Searcher role shows peak activity during the Argument stage (3.4431), while the Writer’s interactions are highest in the Summary stage (3.0932). The Reviewer maintains consistently high engagement throughout all stages. In contrast, the Analyzer exhibits a stable, low interaction frequency (approximately 1.0) across all phases. Notably, the Searcher’s high activity in the Stage 1 is due to its proactive information-seeking role, whereas in subsequent stages (Rebuttal and Summary), it assumes a more passive stance, resulting in lower interaction frequencies. These patterns suggest varied role importance and engagement levels at different debate stages, with the Analyzer serving as an efficient, low-interaction information processor throughout the process.

| Stage | Searcher | Analyzer | Writer | Reviewer |
| --- | --- | --- | --- | --- |
| Overall | 2.1862 | 1.0015 | 2.7243 | 2.4888 |
| Stage 1 | 3.4431 | 1.0036 | 2.4313 | 2.1682 |
| Stage 2 | 1.3435 | 1.0000 | 2.7939 | 2.5817 |
| Stage 3 | 1.2559 | 1.0000 | 3.0932 | 2.8720 |

Table 8: Interaction Frequencies of Roles Across Debate Stages in the Agent4Debate

Table [9](https://arxiv.org/html/2408.04472v2#Ax1.T9 "Table 9 ‣ Further Analysis of Agent4Debate ‣ Appendix ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate") illustrates the average token usage (only calculated completion token) per utterance for each agent. The Searcher, responsible for collecting and organizing information, shows the highest token consumption (2182.90). The Analyzer and Writer exhibit similar token usage (795.06 and 811.94 respectively). The Reviewer, tasked with providing feedback, has the lowest token consumption (415.57), consistent with its role in offering concise critiques. These token consumption patterns align well with the intended functions of each role in our multi-agent framework design, with the Searcher’s high token usage emphasizing its critical role in information processing.

| Token | Searcher | Analyzer | Writer | Reviewer |
| Completion | 2182.90 | 795.06 | 811.94 | 415.57 |

Table 9: Token completion counts for different agents

### Prompts

This section outlines the prompt design principles for Agent4Debate. Due to space constraints, we provide only key examples here, with the complete set of prompts available in the Github Repository³³3https://github.com/ZhangYiqun018/agent-for-debate.

Agent4Debate employs a conversational multi-agent collaborative structure without implicit long-term memory. All information is stored within the dialogue context, accessible to each agent during their turn. The prompt design for each agent consists of five components: profile, knowledge, workflow, rules, and output format.

The profile contains basic agent information. For instance, the Analyzer’s profile might state: ”You are an experienced debate coach tasked with analyzing debate motions, stances, and relevant materials.” The knowledge component stores debate-related information and techniques specific to the task of agent. For example, the Writer agent for the constructive argument (Stage 1) would have prompts on proof techniques, while Stage 2 Writer would have prompts on logical fallacies and rebuttal strategies. The workflow component outlines the specific steps for task completion, ensuring each agent follows a chain-of-thought (CoT) process. The rules component lists guidelines to prevent low-quality responses. Finally, the output format defines the structure of the response of agent.

Each agent is guided by prompts with similar structures but varying content across different debate stages, ensuring task completion at each stage.

### Case Study

Figures [4](https://arxiv.org/html/2408.04472v2#Ax1.F4 "Figure 4 ‣ Case Study ‣ Appendix ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate") to [7](https://arxiv.org/html/2408.04472v2#Ax1.F7 "Figure 7 ‣ Case Study ‣ Appendix ‣ Can LLMs Beat Humans in Debating? A Dynamic Multi-agent Framework for Competitive Debate") show case studies of two debate motions: one on the correlation between justice and interest and another on implementing a fat tax in developed countries. Each motion is shown in English and Chinese, with the English versions translated from Chinese using Claude-3.5-sonnet. These case studies demonstrate the application of Agent4Debate using different foundation models, including GPT-4o, Claude-3.5-sonnet, and Gemini-1.5-Pro. Due to space limitations, references for the case studies have been omitted.

![Refer to caption](img/a4be78e77ddc8b7cf84e1f2ad70e97ad.png)

Figure 4: English (translated by Claude-3.5-sonnet from Chinese) case study of the debate motion ”Justice is nothing but interest. (Pro side) / Justice is nothing more than interest (Con side)”. Pro side is Agent4Debate (GPT-4o), Con side is Agent4Debate (Claude-3.5-sonnet).

![Refer to caption](img/d3eed3d6f53d4630c05431943cceffb2.png)

Figure 5: Chinese case study of the debate motion ”Justice is nothing but interest. (Pro side) / Justice is nothing more than interest (Con side)”. Pro side is Agent4Debate (GPT-4o), Con side is Agent4Debate (Claude-3.5-sonnet).

![Refer to caption](img/5287f57d844534bc164e163d73b74e80.png)

Figure 6: English (translated by Claude-3.5-sonnet from Chinese) case study of the debate motion ”Developed countries should (Pro side) / should not (Con side) impose a fat tax.”. Pro side is Agent4Debate (Gemini-1.5-Pro), Con side is Agent4Debate (Claude-3.5-sonnet).

![Refer to caption](img/bd7d990e2b3aaa12f6f02b620b14c831.png)

Figure 7: Chinese case study of the debate motion ”Developed countries should (Pro side) / should not (Con side) impose a fat tax.”. Pro side is Agent4Debate (Gemini-1.5-Pro), Con side is Agent4Debate (Claude-3.5-sonnet).