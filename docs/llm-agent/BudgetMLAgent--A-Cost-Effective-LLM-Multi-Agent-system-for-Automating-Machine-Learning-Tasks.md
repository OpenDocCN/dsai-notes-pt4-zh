<!--yml
category: 未分类
date: 2025-01-11 11:57:01
-->

# BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks

> 来源：[https://arxiv.org/html/2411.07464/](https://arxiv.org/html/2411.07464/)

\useunder

\ul

Shubham Gandhi TCS ResearchPuneIndia [gandhi.shubham@tcs.com](mailto:gandhi.shubham@tcs.com) ,  Manasi Patwardhan TCS ResearchPuneIndia [manasi.patwardhan@tcs.com](mailto:manasi.patwardhan@tcs.com) ,  Lovekesh Vig TCS ResearchDelhiIndia [lovekesh.vig@tcs.com](mailto:lovekesh.vig@tcs.com)  and  Gautam Shroff TCS ResearchDelhiIndia [gautam.shroff@tcs.com](mailto:gautam.shroff@tcs.com)(2024; 2024)

###### Abstract.

Large Language Models (LLMs) excel in diverse applications including generation of code snippets, but often struggle with generating code for complex Machine Learning (ML) tasks. Although existing LLM single-agent based systems give varying performance depending on the task complexity, they purely rely on larger and expensive models such as GPT-4\. Our investigation reveals that no-cost and low-cost models such as Gemini-Pro, Mixtral and CodeLlama perform far worse than GPT-4 in a single-agent setting. With the motivation of developing a cost-efficient LLM based solution for solving ML tasks, we propose an LLM Multi-Agent based system which leverages combination of experts using profiling, efficient retrieval of past observations, LLM cascades, and ask-the-expert calls. Through empirical analysis on ML engineering tasks in the MLAgentBench benchmark, we demonstrate the effectiveness of our system, using no-cost models, namely Gemini as the base LLM, paired with GPT-4 in cascade and expert to serve occasional ask-the-expert calls for planning. With 94.2% reduction in the cost (from $0.931 per run cost averaged over all tasks for GPT-4 single agent system to $0.054), our system is able to yield better average success rate of 32.95% as compared to GPT-4 single-agent system yielding 22.72% success rate averaged over all the tasks of MLAgentBench.

Automated ML, Large Language Models, LLM Agents^†^†copyright: acmcopyright^†^†journalyear: 2024^†^†doi: 3703412.3703416^†^†isbn: 979-8-4007-1161-9^†^†journalyear: 2024^†^†copyright: rightsretained^†^†conference: 4th International Conference on AI-ML Systems; October 08–11, 2024; Baton Rouge, LA, USA^†^†booktitle: 4th International Conference on AI-ML Systems (AIMLSystems 2024), October 08–11, 2024, Baton Rouge, LA, USA^†^†doi: 10.1145/3703412.3703416^†^†isbn: 979-8-4007-1161-9/24/10^†^†ccs: Computing methodologies Multi-agent systems^†^†ccs: Computing methodologies Intelligent agents^†^†ccs: Computing methodologies Multi-agent planning

## 1\. Introduction

Although recent advances have shown that Large Language Models (LLMs) are adept at handling a vast array of applications ranging from natural language (Fang et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib9); Huang and Chang, [2023](https://arxiv.org/html/2411.07464v2#bib.bib16); Zhu et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib51); Yi et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib41)) to code-related tasks (Zheng et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib49); Zan et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib43); Zhang et al., [2024a](https://arxiv.org/html/2411.07464v2#bib.bib48)), this capability does not often translate to more complicated and nuanced tasks (Yeadon et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib40)). Most code-related efforts involving LLMs (Guo et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib11); Huang et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib15); Zhong et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib50)) are based on tasks such as HumanEval (Chen et al., [2021](https://arxiv.org/html/2411.07464v2#bib.bib7)) and MBXP (Athiwaratkun et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib4)), that have a relatively easier level of complexity that is far from what is experienced by data scientists. However, real-world engineering challenges demand nuanced problem-solving and intricate planning, often involving multiple rounds of strategizing, experimentation, and recalibration. LLM agent systems excel in simulating this iterative process, since they comprise of an environment containing code files, description files and data files and a pre-defined action space allowing interaction with the environment. This demonstrates their capability to address intricate engineering challenges effectively. (Zhang et al., [2024b](https://arxiv.org/html/2411.07464v2#bib.bib46)).

Transitioning to codifying Machine Learning (ML) applications brings its own challenges since they often involve training models on datasets, tuning hyperparameters, devising ways to improve performance, etc. These applications are not straightforward and require a deep understanding of the underlying algorithms and techniques along with specific libraries used for implementation of plans. Although there exist AutoML-based approaches for automating such tasks (He et al., [2021](https://arxiv.org/html/2411.07464v2#bib.bib12); Salehin et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib29)), these offer limited flexibility since they typically operate within predefined constraints and search spaces in the form of possible configurations of architectures and/ or hyper-parameters, which may limit their ability to explore solutions out-of-distribution of the search space. While works such as ChatDev (Qian et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib27)) and MetaGPT (Hong et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib13)) have explored the capabilities of LLM Agents in a software development environment, there is a notable scarcity of research on utilizing LLM Agents for solving ML tasks.

Recent works like MLCopilot (Zhang et al., [2024c](https://arxiv.org/html/2411.07464v2#bib.bib47)) introduce an assistant for solving ML tasks. However, these architectures are limited in the types of problems they can address and must strictly follow task description formats that do not align with real-world scenarios. Additionally, such assistants only suggest solutions, leaving the actual burden of implementation to the user. To the best of our knowledge, MLAgentBench (Huang et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib17)) is the only significant benchmark addressing ML problem solving capabilities of LLM Agents directly dealing with code.

Although they get good performance on some tasks in their benchmark, they focus on single-agent systems using expensive LLMs such as GPT-4, which costs approximately $0.52-$2.9 per run, depending on the task. For the experiments they conduct, they go for 8 runs per task for 15+ tasks, leading to very high experimental cost of approximately $200+. With such larger models becoming increasingly expensive to use, there is a natural incentive to develop no-cost or low-cost systems using smaller, open-source models and making them equally capable for niche tasks. However, existing agent creation frameworks like AutoGen (Wu et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib39)) do not prioritize cost-reduction. Replacing single-agent systems using expensive LLMs with single-agent smaller, open-source LLMs may not serve the purpose. Our initial experiments with replacing all LLM calls in Huang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib17)) for auto-generating codes for ML tasks, with no or low-cost LLMs, namely, Gemini-Pro (Team et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib32))¹¹1gemini-pro 1.0 API from [https://ai.google.dev/tutorials/python_quickstart](https://ai.google.dev/tutorials/python_quickstart). The rate limit for the free or no-cost version is sufficient for conducting our experiments, however, we also include costs for a no-cost version with pay-as-you-go pricing, CodeLlama (Rozière et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib28))²²2[https://huggingface.co/codellama/CodeLlama-34b-Instruct-hf](https://huggingface.co/codellama/CodeLlama-34b-Instruct-hf) and Mixtral(Jiang et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib19))³³3Mixtral-8x7B-v0.1 [https://huggingface.co/mistralai/Mixtral-8x7B-v0.1](https://huggingface.co/mistralai/Mixtral-8x7B-v0.1), yield very poor results for all of the tasks in a single-agent setting.

In real-world setting any complicated tasks are rarely tackled by a single individual alone, especially when all the individuals do not possess the required expertise to perform the task. Instead, teams of engineers collaborate, with each member having a unique role (persona) and contributing unique expertise and skills to achieve the target with collective efforts. Past works on LLM agents have simulated this real-world setting by designing multi-agent frameworks (Li et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib22); Shen et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib30)), combining LLM experts (Wang et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib35); Ding et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib8)) and defining cascades (Chen et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib6); Yue et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib42); Zhang et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib45)) for tasks such as code generation, reasoning, question answering, etc. Cascades refer to the chaining of LLMs in a progressive fashion, where, a weaker LLM is invoked first and if the response is not satisfactory then stronger LLMs are invoked. However, to the best of our knowledge multi-agent frameworks with open-source LLMs as agents have not be explored for engineering of ML tasks.

In this paper, we address the gap of utilizing LLMs for solving ML tasks by proposing a system that leverages - (i) Multi-LLM Agents as a combination of experts using profiling, (ii) LLM Cascades, (iii) Efficient retrieval of relevant past observations, and (iv) our novel occasional ask-the-expert calls to GPT-4 ⁴⁴4gpt-4-0125-preview [https://platform.openai.com/docs/models/gpt-4-and-gpt-4-turbo](https://platform.openai.com/docs/models/gpt-4-and-gpt-4-turbo) for planning. Our approach aims to bridge the divide between capabilities of cheaper LLMs and the requirements of complex ML tasks, offering a more cost-efficient and scalable solution. Through empirical analysis, we validate the following claims:

*   •

    Our best performing multi-agent system using no-cost or low-cost versions of Gemini-Pro as the base LLM, is able to perform tasks at a fraction of the cost (on an average average $0.054 for no-cost and $0.120 for low-cost version per run per ML task in MLAgentBench Dataset) as compared to benchmarked single-agent GPT4 system presented in Huang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib17)) (on an average $0.931 per run per task)

*   •

    With 94.2% and 87.1% cost reduction for the no-cost and low-cost Gemini-Pro versions, our best performing multi-agent system is able to yield better success rate of 32.95% averaged for all the tasks in MLAgentBench as compared to the GPT4 based single-agent system yielding 22.72% average success rate for all tasks.

*   •

    Our best performing multi-agent system is able to achieve equal or better performance for 45.45% of tasks when compared to the GPT4-based Single-Agent system in Huang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib17)), whereas it yields comparable performance for other tasks

## 2\. MLAgentBench Dataset

MLAgentBench (Huang et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib17)) is a dataset designed for evaluating LLM Agents for Machine Learning (ML) tasks. ML tasks defined within the MLAgentBench dataset are specified with clarity, providing a concise description of the desired objective, evaluation metric, and submission guidelines. These task descriptions are diverse and not restricted to a particular format, allowing for a broad range of task definitions. For example, tasks involve improving model accuracy on a given dataset or optimizing a specific performance metric. The dataset also provides necessary files containing training and testing data, along with detailed descriptions of the data and metrics. Starter code, implemented across diverse ML frameworks like PyTorch, TensorFlow, JAX, and Keras, is provided to assist agents in getting started. While some tasks (cifar10, ogbn-arxiv, etc) offer baseline implementations for comparison, others (imdb, house-price, etc) require agents to code models from scratch based on the provided specifications and dataset files.

In the MLAgentBench framework, each task represents an environment where agents interact by performing actions and receiving observations. The benchmark offers a set of primitive low-level actions, including file system operations (For example, list files, read, write, append, copy, etc), executing Python scripts, and declaring final answers. Additionally, there also exist high-level actions such as understanding a file, reflection (looking over past steps and contemplating based on the given description of what to reflect on), inspecting a segment of a file and editing a script (or a script segment). High-level actions may call some low-level actions or LLMs internally (for example understand file action might result in file contents being passed to an LLM and asking it to understand the contents.). Each action is accompanied by comprehensive documentation, specifying its name, description, usage guidelines, expected return values, and implementation. These actions enable LLM agent to manipulate files, execute scripts, and declare final outcomes within the task environment, facilitating iterative problem-solving and evaluation.

We consider a subset of MLAgentBench dataset. We do not take two tasks in to consideration, viz. fathomnet (Woodward et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib38)) and identify-contrails (Ng et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib25)), due to our compute restrictions which can not handle the extremely large size of the datasets corresponding to these tasks. We also drop the LLM Tools tasks from MLAgentBench, as it does not have any comparable results for evaluating our system. Thus, we take into consideration the following tasks: (i) Canonical tasks sich as cifar10 (Krizhevsky, [2009](https://arxiv.org/html/2411.07464v2#bib.bib20)), imdb (Maas et al., [2011](https://arxiv.org/html/2411.07464v2#bib.bib24)) and ogbn-arxiv (Hu et al., [2021](https://arxiv.org/html/2411.07464v2#bib.bib14))), (ii) Classic Kaggle tasks such as house-price (Anna Montoya, [2016](https://arxiv.org/html/2411.07464v2#bib.bib3)) and spaceship-titanic (Addison Howard, [2022](https://arxiv.org/html/2411.07464v2#bib.bib2)), (iii) Kaggle Challenges such as parkinsons-disease (Leslie Kirsch and Dardov, [2023](https://arxiv.org/html/2411.07464v2#bib.bib21)) and feedback (Franklin et al., [2022](https://arxiv.org/html/2411.07464v2#bib.bib10))) (iv) Current Research such as CLRS (Veličković et al., [2022](https://arxiv.org/html/2411.07464v2#bib.bib34)) and BabyLM (Warstadt et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib37))) and (v) Improve Code tasks such as llama-inference and vectorization.

## 3\. BudgetMLAgent

![Refer to caption](img/ff3d10b8486f4f8957bb545a4d9dca7f.png)

Figure 1\. BudgetMLAgent defined in Section [3](https://arxiv.org/html/2411.07464v2#S3 "3\. BudgetMLAgent ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks") with profiling, cascades, ask-the-expert lifelines and retrieval from logs. Note that GPT4 cascade and planning expert calls both count towards the max GPT4 calls limit. Ge: Gemini Pro (no-cost LLM Agent).

Our proposed system, BudgetMLAgent is an LLM Multi-Agent system that primarily uses no-cost LLMs and incorporates several enhancements viz. (i) LLM Profiling, (ii) LLM Cascades and (iii) Ask-the-expert lifelines.

### 3.1\. Multi-Agent LLM Profiling

Our system capitalizes on the groundwork laid by MLAgentBench (Huang et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib17)) that provides a straightforward single LLM Agent based solution for the tasks. It operates through an organized prompt-response based interaction system by an agent that uses a set of available actions to interact with the environment. Through carefully structured prompts, they aim to ensure clarity and precision in conveying task descriptions, available tools (possible set-of actions), and most recent steps taken, to enhance the agent’s decision-making process. To emphasize thoughtful decision-making during planning, the LLM is instructed to stick to a structured format for providing responses to the aforementioned structured prompts, including elements such as ‘Reflection’ on understanding the prior observations, an updatable ‘Implementation Plan’ and step-wise ‘Status’, ‘Fact check’ on if the objective statements from the Plan and Status guessed or directly confirmed and ‘Thought’ on the action to be performed with justification and reasoning. This should be followed by the proposed ‘Action’ for the next step along with the corresponding ‘Action Inputs’ in JSON format. This structured response format is aimed at enhancing the agent’s ability to engage in reflective thinking, better planning, and result verification. They also make use of a logging mechanism inspired by the memory stream paradigm (Park et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib26)), which enables efficient management of historical data rather than inundating the LLM with extensive historical context. By adopting this design, they ensure that the log file serves as a repository of relevant information that can be easily retrieved and updated by the agent using LLMs. Retrieval-enabled ($R$) runs refer to the ones having this functionality enabled. Thus, this retrieved information coupled with the recent actions and observations make up the historical context. The retrieved information acts as long-term memory whereas the recent actions and observations act as short-term memory.

 | Type | Agent | Profile |
| --- | --- | --- |
| Planner | Default planner | You are a planner for solving machine learning tasks. |
| Planning Expert | You are an expert in planning for solving machine learning tasks. |
| High-level action worker | Understand File | You are an expert in understanding files containing both code and natural language. |
| Edit Script (AI) | You are an expert in editing code files. |
| Reflection | You are an expert in reflecting on previous actions when solving a machine learning task. |
| Inspect Script Lines | NA |
| Low-level action worker | List Files | NA |
| Copy File | NA |
| Undo Edit Script | NA |
| Final Answer | NA |
| Execute Script | NA | 

Table 1\. Agents for callable actions and profiles given through system prompts to agents involving LLM calls. NA - no profile as these are programmatic agents and not LLM agents.

We extend the above framework for a multi-agent LLM system. Multi-Agent LLM Profiling refers to the technique of combining the expertise of multiple agents using LLM Profiling, i.e. assigning each of the agents distinct personas. We characterize the multi-agent nature of our system by categorizing the agents into two specific classes - (i) A Planner (P) that utilizes the aforementioned agent structure to consider historical context and ’plan’ the next action, and (ii) Workers (W[i]s) that execute the actions. In addition to a profile for the planner, we also include distinct personas for workers performing distinct actions that involve calls to LLMs such as Edit Script, Understand File, etc as seen in table [1](https://arxiv.org/html/2411.07464v2#S3.T1 "Table 1 ‣ 3.1\. Multi-Agent LLM Profiling ‣ 3\. BudgetMLAgent ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks"). Instead of having the default ”You are a helpful AI assistant” system prompt, we have distinct profiles for each action resulting in a system with agents specialized in distinct roles. These worker agents do not interact with each other and are instead invoked by the Planner P agent whenever it chooses to perform a corresponding action. Executing these actions may or may not involve an internal LLM call.

### 3.2\. LLM Cascade

LLM Cascades refer to the technique of conditional invocation of sequentially connected LLMs (L[1], L[2], … L[k]). Here, we chain LLMs in a manner wherein $cost(L_{1})<cost(L_{2})...<cost(L_{k})$. Here the cost is represented by the latest pricing information of the corresponding models. A set of protocols are enforced to decide if the response by an LLM at a particular ”cascade” is acceptable or not. If it is acceptable, then the response is used as is and if not, we move up the cascade to the next LLM. For example, the LLMs in cascade could be Gemini-Pro (a no-cost LLM) followed by GPT4 (an expensive LLM). If Gemini-Pro fails to generate an acceptable response before exhausting its maximum retries, then the system would invoke GPT4 for that step. In our study, the protocols to move up the cascade are two-fold - (i) If the current LLM fails to generate a response that adheres to the specified format, even after maximum $m$ number of tries, or (ii) If the current LLM chooses an action that has already been repeated $r$ consecutive times in the past $r$ steps.

### 3.3\. Ask-the-Expert Action

To save on expenses, we primarily employ no-cost LLMs for both planning and action-based LLM calls. Preliminary investigations reveal that mostly planning is the shortcoming of these LLMs and that their responses are of sufficient quality when it comes to other action-based LLM calls by workers. Therefore, we give the LLM with the Planner P persona, $l$ ’lifelines’, where it can choose to call a larger, more expensive LLM with higher expertise, in scenarios where it identifies that it is stuck at a step. This choice is implemented in the form of an action, Request Help from a Planning Expert, that the current ‘Planner’ LLM agent can choose to take. Moreover, in practice, this upper cap on number of larger model calls (life-lines) also counts the calls made as a result of the cascade protocol mentioned in Section [3.2](https://arxiv.org/html/2411.07464v2#S3.SS2 "3.2\. LLM Cascade ‣ 3\. BudgetMLAgent ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks"). During the course of a run, once the planner hits this upper cap, the Request Help from a Planning Expert action is no longer included in the prompt when populating the actions available to call.

## 4\. Experimentation and Results

We perform our experiments on the subset of tasks of MLAgentBench dataset explained in Section [2](https://arxiv.org/html/2411.07464v2#S2 "2\. MLAgentBench Dataset ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks").

### 4.1\. Metrics

We evaluate our results by taking two metrics under consideration.

#### 4.1.1\. Success Rate

The success rate is the percentage (%) of runs which are considered as successful. As per defined in Huang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib17)), a run is considered to be successful if it achieves more than 10% improvement at the last step over the average performance of the baseline in the starter code. Here the performance measure is task specific. For canonical tasks, classic kaggle, kaggle challenges and current research type tasks mentioned in Section [2](https://arxiv.org/html/2411.07464v2#S2 "2\. MLAgentBench Dataset ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks"), the prediction accuracy of the final submission.csv file is considered as the performance metric. For improve code type tasks, the improvement in the runtime of the code is considered as a success metric, whereas for CLRS, the saved final model checkpoints are evaluated for accuracy and improvement in accuracy is considered as success criteria.

#### 4.1.2\. Cost

If an LLM has a monetary cost associated with it, we compute the average cost in dollars ($) per run based on number of tokens used for that model⁵⁵5 The pricing information as of March 2024. For LLMs where APIs are available, this becomes $\$[($Cost_per_input_token $*$ Num_of_input_tokens$)+($Cost_per_output_token $*$ Num_of_output_tokens$)]$. Claude1 V1.0 used as a single agent in Huang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib17)) is discontinued and hence latest pricing details for this are unavailable. For approximating cost for Claude, we use pricing information for Claude-Instant ⁶⁶6[https://www.anthropic.com/api](https://www.anthropic.com/api).

### 4.2\. Models and hyperparameters

We analyze multiple no-cost LLMs such as Gemini, CodeLlama and Mixtral as single agents to test their ML problem solving capabilities. We employ two configurations for our proposed multi-agent framework: (i) Gemini, which is best performing single agent LLM, with ChatGPT ⁷⁷7gpt-3.5-turbo from [https://platform.openai.com/docs/models/gpt-3-5-turbo](https://platform.openai.com/docs/models/gpt-3-5-turbo) in cascade. (ii) Gemini with GPT4 in cascade and ‘Ask-the-Expert’ agent. For our runs, we set all hyperparameters as per the implementation of Huang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib17)). Maximum number of actions is set to 30\. Maximum number of recent actions to be included in context as the short-term memory is set to to 3\. For runs involving cascades, we set the maximum number of retries allowed ($m$) to 3 for Gemini-Pro, 3 for ChatGPT and 1 for GPT4 in the interest of cost. For ask-the-expert GPT4 calls also, we set the maximum number of retries allowed to 1\. The maximum number of times an action can be consecutively repeated ($r$) is set to 3\. For planning-related LLM calls, the temperature is set to 0.2 and for internal action-related LLM calls, the temperature is set to 0.01, since we need some amount of diversity in the output for the former and more definitive responses for latter. For runs involving ask-the-planning-expert calls, we use GPT4 as the planning expert and set the maximum number of calls to GPT4 (lifelines $l$) to 5\. Note that this also includes the calls made to GPT4 in an LLM cascade. Since the exact monetary cost for GPT4 and Claude single agent runs from Huang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib17)) is not made available, we approximate these costs for comparison. We use the average token usage from other single agent runs, namely Gemini-Pro and CodeLlama, after multiplying with a factor of 0.809 to account for 19.1% lesser token usage by GPT4 as suggested by Huang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib17)).

### 4.3\. Baselines

We use following baselines: (i) Single Agent GPT4 in retrieval setting (G + R) (ii) Single Agent GPT4 with no retrieval (G) (iii) Single Agent Claude V1.0 in retrieval setting (C + R) (iv) Single Agent Claude V1.0 with no retrieval (C) ((i) to (iv) are from Huang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib17))) (v) Single Agent Gemini Pro in retrieval setting (Ge + R) (vi) Single Agent Code Llama in retrieval setting (Co + R) (vii) Single Agent Mixtral in retrieval setting (Mx + R) ((v) to (vii) are our baselines using no-cost LLMs)

 |  | Single Agent(Huang et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib17)) | Profiling + Cascade | Profiling + Cascade + Expert |
| Task | G + R | G | C + R | C | Ge + R | Ge | Ge + Ch + R | Ge + Ch | Ge + G + R | Ge + G |
|  

&#124; cifar10 &#124;

 |  

&#124; 25 &#124;
&#124; ($0.583) &#124;

 |  

&#124; 50 &#124;
&#124; ($0.44) &#124;

 |  

&#124; 8 &#124;
&#124; ($0.058) &#124;

 |  

&#124; 48 &#124;
&#124; ($0.044) &#124;

 |  

&#124; 12.5 &#124;
&#124; ($0)* &#124;
&#124; ($0.016) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.044) &#124;

 |  

&#124; 62.5 &#124;
&#124; ($0)* &#124;
&#124; ($0.025) &#124;

 |  

&#124; 25 &#124;
&#124; ($0)* &#124;
&#124; ($0.093) &#124;

 |  

&#124; 75 &#124;
&#124; ($0.057)* &#124;
&#124; ($0.094) &#124;

 |  

&#124; 37.5 &#124;
&#124; ($0.06)* &#124;
&#124; ($0.034) &#124;

 |
|  

&#124; imdb &#124;

 |  

&#124; 12.5 &#124;
&#124; ($1.48) &#124;

 |  

&#124; 25 &#124;
&#124; ($0.919) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.147) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.091) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.006) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.038) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.022) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.023) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.014)* &#124;
&#124; ($0.060) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.071)* &#124;
&#124; ($0.124) &#124;

 |
|  

&#124; ogbn-arxiv &#124;

 |  

&#124; 50 &#124;
&#124; ($1.27) &#124;

 |  

&#124; 87.5 &#124;
&#124; ($1.112) &#124;

 |  

&#124; 40 &#124;
&#124; ($0.125) &#124;

 |  

&#124; 32 &#124;
&#124; ($0.109) &#124;

 |  

&#124; 25 &#124;
&#124; ($0)* &#124;
&#124; ($0.011) &#124;

 |  

&#124; 25 &#124;
&#124; ($0)* &#124;
&#124; ($0.016) &#124;

 |  

&#124; 50 &#124;
&#124; ($0)* &#124;
&#124; ($0.020) &#124;

 |  

&#124; 50 &#124;
&#124; ($0)* &#124;
&#124; ($0.015) &#124;

 |  

&#124; 50 &#124;
&#124; ($0.033)* &#124;
&#124; ($0.067) &#124;

 |  

&#124; 75 &#124;
&#124; ($0.026)* &#124;
&#124; ($0.045) &#124;

 |
|  

&#124; house-price &#124;

 |  

&#124; 25 &#124;
&#124; ($1.6) &#124;

 |  

&#124; 12.5 &#124;
&#124; ($0.938) &#124;

 |  

&#124; 64 &#124;
&#124; ($0.158) &#124;

 |  

&#124; 76 &#124;
&#124; ($0.093) &#124;

 |  

&#124; 25 &#124;
&#124; ($0)* &#124;
&#124; ($0.042) &#124;

 |  

&#124; 37.5 &#124;
&#124; ($0)* &#124;
&#124; ($0.070) &#124;

 |  

&#124; 62.5 &#124;
&#124; ($0)* &#124;
&#124; ($0.046) &#124;

 |  

&#124; 62.5 &#124;
&#124; ($0)* &#124;
&#124; ($0.049) &#124;

 |  

&#124; 87.5 &#124;
&#124; ($0.091)* &#124;
&#124; ($0.161) &#124;

 |  

&#124; 75 &#124;
&#124; ($0.038)* &#124;
&#124; ($0.153) &#124;

 |
|  

&#124; spaceship- &#124;
&#124; titanic &#124;

 |  

&#124; 25 &#124;
&#124; ($1.42) &#124;

 |  

&#124; 12.5 &#124;
&#124; ($0.85) &#124;

 |  

&#124; 4 &#124;
&#124; ($0.141) &#124;

 |  

&#124; 16 &#124;
&#124; ($0.088) &#124;

 |  

&#124; 37.5 &#124;
&#124; ($0)* &#124;
&#124; ($0.039) &#124;

 |  

&#124; 37.5 &#124;
&#124; ($0)* &#124;
&#124; ($0.052) &#124;

 |  

&#124; 75 &#124;
&#124; ($0)* &#124;
&#124; ($0.077) &#124;

 |  

&#124; 100 &#124;
&#124; ($0.0004)* &#124;
&#124; ($0.038) &#124;

 |  

&#124; 75 &#124;
&#124; ($0.021)* &#124;
&#124; ($0.096) &#124;

 |  

&#124; 100 &#124;
&#124; ($0.091)* &#124;
&#124; ($0.213) &#124;

 |
|  

&#124; parkinsons- &#124;
&#124; disease &#124;

 |  

&#124; 12.5 &#124;
&#124; ($2.9) &#124;

 |  

&#124; 0 &#124;
&#124; ($1.57) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.287) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.155) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.078) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.065) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.024) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.028) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.099)* &#124;
&#124; ($0.186) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.107)* &#124;
&#124; ($0.183) &#124;

 |
|  

&#124; feedback &#124;

 |  

&#124; 37.5 &#124;
&#124; ($1.25) &#124;

 |  

&#124; 12.5 &#124;
&#124; ($1.15) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.124) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.114) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.058) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.046) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.0005)* &#124;
&#124; ($0.038) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.030) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.047)* &#124;
&#124; ($0.110) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.022)* &#124;
&#124; ($0.068) &#124;

 |
|  

&#124; llama- &#124;
&#124; inference &#124;

 |  

&#124; 0 &#124;
&#124; ($1.46) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.927) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.145) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.092) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.030) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.059) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.0003)* &#124;
&#124; ($0.032) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.034) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.027)* &#124;
&#124; ($0.074) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.055)* &#124;
&#124; ($0.093) &#124;

 |
|  

&#124; vectorization &#124;

 |  

&#124; 0 &#124;
&#124; ($1.16) &#124;

 |  

&#124; 0 &#124;
&#124; ($1.23) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.114) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.121) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.062) &#124;

 |  

&#124; 12.5 &#124;
&#124; ($0)* &#124;
&#124; ($0.058) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.039) &#124;

 |  

&#124; 12.5 &#124;
&#124; ($0)* &#124;
&#124; ($0.025) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.017)* &#124;
&#124; ($0.077) &#124;

 |  

&#124; 75 &#124;
&#124; ($0.004)* &#124;
&#124; ($0.092) &#124;

 |
|  

&#124; CLRS &#124;

 |  

&#124; 12.5 &#124;
&#124; ($0.523) &#124;

 |  

&#124; 50 &#124;
&#124; ($0.43) &#124;

 |  

&#124; 52 &#124;
&#124; ($0.052) &#124;

 |  

&#124; 40 &#124;
&#124; ($0.042) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.006) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.006*) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.0003)* &#124;
&#124; ($0.033) &#124;

 |  

&#124; 12.5 &#124;
&#124; ($0)* &#124;
&#124; ($0.022) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.027)* &#124;
&#124; ($0.050) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.046)* &#124;
&#124; ($0.138) &#124;

 |
|  

&#124; babylm &#124;

 |  

&#124; 0 &#124;
&#124; ($0.818) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.637) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.081) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.063) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0.032) &#124;

 |  

&#124; 0 &#124;
&#124; ($0)* &#124;
&#124; ($0*.027) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.0025)* &#124;
&#124; ($0.046) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.001)* &#124;
&#124; ($0.024) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.086)* &#124;
&#124; ($0.147) &#124;

 |  

&#124; 0 &#124;
&#124; ($0.078)* &#124;
&#124; ($0.113) &#124;

 |
| Average |  

&#124; 18.18 &#124;
&#124; ($1.315) &#124;

 |  

&#124; 22.72 &#124;
&#124; ($0.931) &#124;

 |  

&#124; 15.27 &#124;
&#124; ($0.13) &#124;

 |  

&#124; 20 &#124;
&#124; ($0.092) &#124;

 |  

&#124; 9.09 &#124;
&#124; ($0)* &#124;
&#124; ($0.034) &#124;

 |  

&#124; 10.23 &#124;
&#124; ($0)* &#124;
&#124; ($0.044) &#124;

 |  

&#124; 22.72 &#124;
&#124; ($0.0003)* &#124;
&#124; ($0.036) &#124;

 |  

&#124; 22.72 &#124;
&#124; ($0.0001)* &#124;
&#124; ($0.032) &#124;

 |  

&#124; 26.14 &#124;
&#124; ($0.047)* &#124;
&#124; ($0.102) &#124;

 |  

&#124; 32.95 &#124;
&#124; ($0.054)* &#124;
&#124; ($0.120) &#124;

 | 

Table 2\. Each cell depicts Success Rate in %: % successful runs, ($): Average cost per run, * - considering free Gemini Pro API calls; G - GPT4 , C - Claude V1.0, Ge - Gemini Pro, Co - CodeLlama Instruct 34b, Ch - ChatGPT (GPT 3.5 Turbo), R - Retrieval: The agent can retrieve and summarize relevant information from long-term memory in research logs. In no-retrieval setting this functionality is disabled.

### 4.4\. Results and Discussion

We address key Research Questions (RQ) based on the results in table [2](https://arxiv.org/html/2411.07464v2#S4.T2 "Table 2 ‣ 4.3\. Baselines ‣ 4\. Experimentation and Results ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks").

#### 4.4.1\. RQ1 - Do no-cost and low-cost LLM single-agents sacrifice performance for cost savings?

We observe a significant drop in the performance when we use a purely no-cost or low-cost LLM (Ge, Co, and Mx) in single-agent setting across all tasks as opposed to LLM single agents using GPT4 or Claude presented in Huang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib17)). We observe CodeLlama (Co) and Mixtral (Mx) are unable to produce any successful runs leading to average 0% success rate. Thus, we omit them from the results in table [2](https://arxiv.org/html/2411.07464v2#S4.T2 "Table 2 ‣ 4.3\. Baselines ‣ 4\. Experimentation and Results ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks"). We see that Mixtral is almost never able to adhere to the required response format mentioned in section [3.1](https://arxiv.org/html/2411.07464v2#S3.SS1 "3.1\. Multi-Agent LLM Profiling ‣ 3\. BudgetMLAgent ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks") across all runs which leads to termination due to maximum retry limit being exceeded. However, we observe that Gemini-Pro (Ge + R) is able to produce successful runs in single agent setting yielding non-zero performance for some tasks (cifar10, ogbn-arxiv, house-price and spaceship-titanic) but zero for others. Visual inspection reveals that there is not much difference between the quality of responses in internal action-related LLM calls between Gemini-Pro and CodeLllama, with the former being slightly superior. However, we observe that Code Llama is unable to produce any successful runs due to its inability to plan effectively.

#### 4.4.2\. RQ2 - How do profiling and cascades affect performance and cost?

From table [2](https://arxiv.org/html/2411.07464v2#S4.T2 "Table 2 ‣ 4.3\. Baselines ‣ 4\. Experimentation and Results ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks"), it can be seen that profiling with Gemini-Pro as base LLM and ChatGPT (GPT-3.5-turbo) in cascade (Ge + Ch) both with and without retriever setting, significantly increases success rate for many tasks, namely cifar10, ogbn-arxiv, house-price, spaceship-titanic and vectorization when compared with Ge as single agent. However, for imdb, parkinsons-disease, feedback, llama-inference, CLRS and babylm tasks, the performance still remains zero due to their complex nature. Overall average success rate of profiling and cascade is comparable with GPT4 (G) performance. These improvements are obtained at almost 100% cost reduction for no-cost Gemini-Pro (From $1.315 for G + R and $0.13 for C + R to $0.0003 for Ge + Ch + R and from $0.931 for G and $0.092 for C to $0.0001 for Ge + Ch). This is because inference for GPT-3.5-turbo is much cheaper leading to minuscule costs. However, qualitative analysis shows that GPT-3.5-turbo often fails to adhere to the required response format mentioned in section [3.1](https://arxiv.org/html/2411.07464v2#S3.SS1 "3.1\. Multi-Agent LLM Profiling ‣ 3\. BudgetMLAgent ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks") leading to further retries. Thus, we shift to GPT4 for cascades for subsequent runs.

#### 4.4.3\. RQ3 - How does adding ask-the-expert lifelines to profiling and cascade affect performance and cost?

Table [2](https://arxiv.org/html/2411.07464v2#S4.T2 "Table 2 ‣ 4.3\. Baselines ‣ 4\. Experimentation and Results ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks") shows that access to GPT4 ask-the-expert lifeline calls as part of our proposed system (Ge + G and Ge + G + R) improves success rate for cifar10, ogbn-arxiv, house-price, spaceship-titanic and vectorization tasks when compared with Ge + Ch+ R and Ge + Ch. Additionally, we observe improvements in success rate when compared with G + R and G for cifar10, house-price, spaceship-titanic and vectorization tasks at a cost reduction ranging from 90-99% across tasks for no-cost Gemini-Pro. On an average the Profiling + Cascade + Expert setting and using GPT4 for cascade and expert gives 43.78% and 71.19% improvement in retrieval setting and 45.02% and 64.75% improvement in non-retrieval setting over GPT4 and Claude as single agent, respectively. This comes at 96.43% and 63.85% cost reduction for no-cost Gemini-Pro in retrieval setting and 94.2% and 41.3% reduction in cost in non-retrieval setting over GPT4 and Claude, respectively, demonstrating the efficacy of our approach. Qualitative analysis reveals that there are instances when the planner can successfully identify that it is stuck, and calls the expert to get itself ’unstuck’. For example, if a particular edit does not lead to proper execution after multiple steps, and if the expert is called, it takes a different action such as understanding some part of the file better before making further edits.

#### 4.4.4\. RQ4 - How does retrieval from logs affect performance and cost?

In accordance with the findings of Huang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib17)), our observations indicate that while access to retrieval from logs proves effective for certain tasks, for others, disabling it yields better results. One interesting case to note is of cifar10\. GPT4 (G) has a greater success rate than G + R. Huang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib17)) justify this by stating that since cifar10 is a comparitively easier task and the long-term memory context become a distraction. But the trend gets reversed in the case of Gemini-Pro with GPT4 in cascade and expert setting. Ge + G + R has a greater success rate than Ge + G. This can be due to the differences in pretraining and actual data seen by Gemini-Pro and GPT4\. Similar to success rate, cost is greater with retrieval for some tasks, whereas it is greater without retrieval for others. Average cost per run for Ge + G ($0.054) is greater than that for Ge + G + R ($0.047) for no-cost Gemini-Pro.

## 5\. Related Work

### 5.1\. LLM Agents for nuanced code-related tasks

With the rising capabilities of Large Language Models, there have been many advances in employing LLM Agents for code-related tasks. Initial works relied primarily on HumanEval (Chen et al., [2021](https://arxiv.org/html/2411.07464v2#bib.bib7)) and MBXP (Athiwaratkun et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib4)) for benchmarking their approaches. However, recent works have started moving towards more nuanced and real-world benchmarks. Wang et al. ([2024](https://arxiv.org/html/2411.07464v2#bib.bib36)) present CodeAct - a Python code database containing tasks related to API handling, library usage, etc. They also present CodeActAgent that is finetuned on Llama-2 (Touvron et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib33)) and Mistral-7b (Jiang et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib18)) using CodeActInstruct, an instruction tuning dataset consisting of multi-turn interactions. AgentBench (Liu et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib23)) also introduces three code agent environments based on operating system, database and knowledge graph related problems for evaluating LLM agents on these tasks. There has also been some amount of work done for tackling more complex problems. Zhang et al. ([2024b](https://arxiv.org/html/2411.07464v2#bib.bib46)) construct CodeAgentBench, a dataset containing real-world repo-level coding challenges. They also propose CodeAgent to tackle the tasks in this dataset utilizing external information retrieval, code implementation and testing tools at its disposal.

Huang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib17)) construct MLAgentBench, explained in Section [2](https://arxiv.org/html/2411.07464v2#S2 "2\. MLAgentBench Dataset ‣ BudgetMLAgent: A Cost-Effective LLM Multi-Agent system for Automating Machine Learning Tasks"), that consists of machine learning tasks. They also propose a single agent based system to solve these tasks using various tools such as code execution, code editing, etc. Tang et al. ([2024](https://arxiv.org/html/2411.07464v2#bib.bib31)) explores the use of LLM Agents with repository-level code. However, their approach does not involve direct code generation. Instead, they focus on creating shell scripts to utilize pre-existing code files with appropriate arguments. On a similar note, CodePlan (Bairi et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib5)) involves framing repository-level coding as a planning problem in the form of a chain of edits. Additionally, NL2Repo (Zan et al., [2024](https://arxiv.org/html/2411.07464v2#bib.bib44)) presents a system for generating codebases from the ground-up, using natural language specifications.

### 5.2\. LLM Multi-Agents

While LLM single-agents have demonstrated remarkable capabilities in various tasks, the complexity of real-world challenges often necessitates collaborative approaches. In this context, the exploration of LLM multi-agent systems has emerged as a promising avenue to enhance problem-solving capabilities and address intricate tasks more effectively. ChatDev (Qian et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib27)) and MetaGPT (Hong et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib13)) are recent multi-agent systems introduced for tackling software development tasks. AutoGen (Wu et al., [2023](https://arxiv.org/html/2411.07464v2#bib.bib39)) is a more generic framework for developing LLM multi-agents and has been evaluated on multiple tasks ranging from math problem solving to retrieval augmented chatting Li et al. ([2024](https://arxiv.org/html/2411.07464v2#bib.bib22)); Shen et al. ([2024](https://arxiv.org/html/2411.07464v2#bib.bib30)) also present frameworks for using multiple LLM agents in a collaborative manner for tasks such as reasoning and tool-learning. Wang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib35)); Ding et al. ([2024](https://arxiv.org/html/2411.07464v2#bib.bib8)) propose combining multiple LLM expert agents to solve a variety of tasks including text summarization and question answering. To save up on costs, Chen et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib6)); Yue et al. ([2024](https://arxiv.org/html/2411.07464v2#bib.bib42)); Zhang et al. ([2023](https://arxiv.org/html/2411.07464v2#bib.bib45)) propose using cascading LLMs in increasing order of cost and capability for diverse tasks such as reasoning, query answering and news prediction. Our focus is on providing cost-effective solution for ML engineering tasks using multi-agent system with no-cost open-source LLM as the base with paid LLM with higher expertise in cascade.

## 6\. Conclusion

In this work, we propose BudgetMLAgent - an LLM Multi-Agent system for solving machine learning tasks in a cost-effective manner without hampering the system performance. Our primary investigations on tasks defined in MLAgentBench dataset, with Single-Agent systems with purely no-cost models give zero success rates for CodeLlama and Mixtral and very poor success rates with Gemini-Pro (9.09% with access to the complete logs and 10.23% with short-term access), as compared to paid models such as GPT4 and ClaudeV1.0 (18.18% and 22.72%, 15.27% and 20% respectively). We subsequently propose a multi-agent framework, BudgetMLAgent, using no-cost Gemini-Pro as the base LLM, leveraging (i) profiling for a planner and multiple worker agents interacting with the ML code generation environment using distinct actions, (ii) cascades to LLMs with more expertise such as GPT-3.5-turbo and GPT4 and (iii) our novel ask-the-expert GPT4 lifelines. BudgetMLAgent results in improving the success rates for MLAgentBench tasks (26.14% and 32.95% respectively) along with significant cost reductions when compared with GPT4 and ClaudeV1.0 based Single-Agent systems (96.43% and 63.85% with access to the complete logs, 94.2% and 41.3% reduction with short-term access respectively).

## References

*   (1)
*   Addison Howard (2022) Ryan Holbrook Addison Howard, Ashley Chow. 2022. Spaceship Titanic. [https://kaggle.com/competitions/spaceship-titanic](https://kaggle.com/competitions/spaceship-titanic)
*   Anna Montoya (2016) DataCanary Anna Montoya. 2016. House Prices - Advanced Regression Techniques. [https://kaggle.com/competitions/house-prices-advanced-regression-techniques](https://kaggle.com/competitions/house-prices-advanced-regression-techniques)
*   Athiwaratkun et al. (2023) Ben Athiwaratkun, Sanjay Krishna Gouda, Zijian Wang, Xiaopeng Li, Yuchen Tian, Ming Tan, Wasi Uddin Ahmad, Shiqi Wang, Qing Sun, Mingyue Shang, Sujan Kumar Gonugondla, Hantian Ding, Varun Kumar, Nathan Fulton, Arash Farahani, Siddhartha Jain, Robert Giaquinto, Haifeng Qian, Murali Krishna Ramanathan, Ramesh Nallapati, Baishakhi Ray, Parminder Bhatia, Sudipta Sengupta, Dan Roth, and Bing Xiang. 2023. Multi-lingual Evaluation of Code Generation Models. arXiv:2210.14868 [cs.LG]
*   Bairi et al. (2023) Ramakrishna Bairi, Atharv Sonwane, Aditya Kanade, Vageesh D C, Arun Iyer, Suresh Parthasarathy, Sriram Rajamani, B. Ashok, and Shashank Shet. 2023. CodePlan: Repository-level Coding using LLMs and Planning. arXiv:2309.12499 [cs.SE] [https://arxiv.org/abs/2309.12499](https://arxiv.org/abs/2309.12499)
*   Chen et al. (2023) Lingjiao Chen, Matei Zaharia, and James Zou. 2023. FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance. arXiv:2305.05176 [cs.LG]
*   Chen et al. (2021) Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman, Alex Ray, Raul Puri, Gretchen Krueger, Michael Petrov, Heidy Khlaaf, Girish Sastry, Pamela Mishkin, Brooke Chan, Scott Gray, Nick Ryder, Mikhail Pavlov, Alethea Power, Lukasz Kaiser, Mohammad Bavarian, Clemens Winter, Philippe Tillet, Felipe Petroski Such, Dave Cummings, Matthias Plappert, Fotios Chantzis, Elizabeth Barnes, Ariel Herbert-Voss, William Hebgen Guss, Alex Nichol, Alex Paino, Nikolas Tezak, Jie Tang, Igor Babuschkin, Suchir Balaji, Shantanu Jain, William Saunders, Christopher Hesse, Andrew N. Carr, Jan Leike, Josh Achiam, Vedant Misra, Evan Morikawa, Alec Radford, Matthew Knight, Miles Brundage, Mira Murati, Katie Mayer, Peter Welinder, Bob McGrew, Dario Amodei, Sam McCandlish, Ilya Sutskever, and Wojciech Zaremba. 2021. Evaluating Large Language Models Trained on Code. arXiv:2107.03374 [cs.LG]
*   Ding et al. (2024) Dujian Ding, Ankur Mallick, Chi Wang, Robert Sim, Subhabrata Mukherjee, Victor Rühle, Laks V. S. Lakshmanan, and Ahmed Hassan Awadallah. 2024. Hybrid LLM: Cost-Efficient and Quality-Aware Query Routing. In *The Twelfth International Conference on Learning Representations*. [https://openreview.net/forum?id=02f3mUtqnM](https://openreview.net/forum?id=02f3mUtqnM)
*   Fang et al. (2024) Xi Fang, Weijie Xu, Fiona Anting Tan, Jiani Zhang, Ziqing Hu, Yanjun Qi, Scott Nickleach, Diego Socolinsky, Srinivasan Sengamedu, and Christos Faloutsos. 2024. Large Language Models(LLMs) on Tabular Data: Prediction, Generation, and Understanding – A Survey. arXiv:2402.17944 [cs.CL]
*   Franklin et al. (2022) Alex Franklin, Maggie, Meg Benner, Natalie Rambis, Perpetual Baffour, Ryan Holbrook, Scott Crossley, and Ulrich Boser. 2022. Feedback Prize - English Language Learning. [https://kaggle.com/competitions/feedback-prize-english-language-learning](https://kaggle.com/competitions/feedback-prize-english-language-learning)
*   Guo et al. (2024) Daya Guo, Qihao Zhu, Dejian Yang, Zhenda Xie, Kai Dong, Wentao Zhang, Guanting Chen, Xiao Bi, Y. Wu, Y. K. Li, Fuli Luo, Yingfei Xiong, and Wenfeng Liang. 2024. DeepSeek-Coder: When the Large Language Model Meets Programming – The Rise of Code Intelligence. arXiv:2401.14196 [cs.SE]
*   He et al. (2021) Xin He, Kaiyong Zhao, and Xiaowen Chu. 2021. AutoML: A survey of the state-of-the-art. *Knowledge-Based Systems* 212 (Jan. 2021), 106622. [https://doi.org/10.1016/j.knosys.2020.106622](https://doi.org/10.1016/j.knosys.2020.106622)
*   Hong et al. (2023) Sirui Hong, Mingchen Zhuge, Jonathan Chen, Xiawu Zheng, Yuheng Cheng, Ceyao Zhang, Jinlin Wang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, Chenyu Ran, Lingfeng Xiao, Chenglin Wu, and Jürgen Schmidhuber. 2023. MetaGPT: Meta Programming for A Multi-Agent Collaborative Framework. arXiv:2308.00352 [cs.AI]
*   Hu et al. (2021) Weihua Hu, Matthias Fey, Marinka Zitnik, Yuxiao Dong, Hongyu Ren, Bowen Liu, Michele Catasta, and Jure Leskovec. 2021. Open Graph Benchmark: Datasets for Machine Learning on Graphs. arXiv:2005.00687 [cs.LG]
*   Huang et al. (2024) Dong Huang, Qingwen Bu, Jie M. Zhang, Michael Luck, and Heming Cui. 2024. AgentCoder: Multi-Agent-based Code Generation with Iterative Testing and Optimisation. arXiv:2312.13010 [cs.CL]
*   Huang and Chang (2023) Jie Huang and Kevin Chen-Chuan Chang. 2023. Towards Reasoning in Large Language Models: A Survey. arXiv:2212.10403 [cs.CL]
*   Huang et al. (2023) Qian Huang, Jian Vora, Percy Liang, and Jure Leskovec. 2023. Benchmarking Large Language Models As AI Research Agents. arXiv:2310.03302 [cs.LG]
*   Jiang et al. (2023) Albert Q. Jiang, Alexandre Sablayrolles, Arthur Mensch, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Florian Bressand, Gianna Lengyel, Guillaume Lample, Lucile Saulnier, Lélio Renard Lavaud, Marie-Anne Lachaux, Pierre Stock, Teven Le Scao, Thibaut Lavril, Thomas Wang, Timothée Lacroix, and William El Sayed. 2023. Mistral 7B. arXiv:2310.06825 [cs.CL]
*   Jiang et al. (2024) Albert Q. Jiang, Alexandre Sablayrolles, Antoine Roux, Arthur Mensch, Blanche Savary, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Emma Bou Hanna, Florian Bressand, Gianna Lengyel, Guillaume Bour, Guillaume Lample, Lélio Renard Lavaud, Lucile Saulnier, Marie-Anne Lachaux, Pierre Stock, Sandeep Subramanian, Sophia Yang, Szymon Antoniak, Teven Le Scao, Théophile Gervet, Thibaut Lavril, Thomas Wang, Timothée Lacroix, and William El Sayed. 2024. Mixtral of Experts. arXiv:2401.04088 [cs.LG]
*   Krizhevsky (2009) Alex Krizhevsky. 2009. Learning Multiple Layers of Features from Tiny Images. [https://api.semanticscholar.org/CorpusID:18268744](https://api.semanticscholar.org/CorpusID:18268744)
*   Leslie Kirsch and Dardov (2023) Stacey Adam Leslie Kirsch, Sohier Dane and Victoria Dardov. 2023. AMP®-Parkinson’s Disease Progression Prediction. [https://kaggle.com/competitions/amp-parkinsons-disease-progression-prediction](https://kaggle.com/competitions/amp-parkinsons-disease-progression-prediction)
*   Li et al. (2024) Junyou Li, Qin Zhang, Yangbin Yu, Qiang Fu, and Deheng Ye. 2024. More Agents Is All You Need. arXiv:2402.05120 [cs.CL]
*   Liu et al. (2023) Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang, Shudan Zhang, Xiang Deng, Aohan Zeng, Zhengxiao Du, Chenhui Zhang, Sheng Shen, Tianjun Zhang, Yu Su, Huan Sun, Minlie Huang, Yuxiao Dong, and Jie Tang. 2023. AgentBench: Evaluating LLMs as Agents. arXiv:2308.03688 [cs.AI]
*   Maas et al. (2011) Andrew L. Maas, Raymond E. Daly, Peter T. Pham, Dan Huang, Andrew Y. Ng, and Christopher Potts. 2011. Learning Word Vectors for Sentiment Analysis. In *Proceedings of the 49th Annual Meeting of the Association for Computational Linguistics: Human Language Technologies*, Dekang Lin, Yuji Matsumoto, and Rada Mihalcea (Eds.). Association for Computational Linguistics, Portland, Oregon, USA, 142–150. [https://aclanthology.org/P11-1015](https://aclanthology.org/P11-1015)
*   Ng et al. (2023) Joe Ng, Carl Elkin, Aaron Sarna, Walter Reade, and Maggie Demkin. 2023. Google Research - Identify Contrails to Reduce Global Warming. [https://kaggle.com/competitions/google-research-identify-contrails-reduce-global-warming](https://kaggle.com/competitions/google-research-identify-contrails-reduce-global-warming)
*   Park et al. (2023) Joon Sung Park, Joseph C. O’Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, and Michael S. Bernstein. 2023. Generative Agents: Interactive Simulacra of Human Behavior. arXiv:2304.03442 [cs.HC]
*   Qian et al. (2023) Chen Qian, Xin Cong, Wei Liu, Cheng Yang, Weize Chen, Yusheng Su, Yufan Dang, Jiahao Li, Juyuan Xu, Dahai Li, Zhiyuan Liu, and Maosong Sun. 2023. Communicative Agents for Software Development. arXiv:2307.07924 [cs.SE]
*   Rozière et al. (2024) Baptiste Rozière, Jonas Gehring, Fabian Gloeckle, Sten Sootla, Itai Gat, Xiaoqing Ellen Tan, Yossi Adi, Jingyu Liu, Romain Sauvestre, Tal Remez, Jérémy Rapin, Artyom Kozhevnikov, Ivan Evtimov, Joanna Bitton, Manish Bhatt, Cristian Canton Ferrer, Aaron Grattafiori, Wenhan Xiong, Alexandre Défossez, Jade Copet, Faisal Azhar, Hugo Touvron, Louis Martin, Nicolas Usunier, Thomas Scialom, and Gabriel Synnaeve. 2024. Code Llama: Open Foundation Models for Code. arXiv:2308.12950 [cs.CL]
*   Salehin et al. (2024) Imrus Salehin, Md. Shamiul Islam, Pritom Saha, S.M. Noman, Azra Tuni, Md. Mehedi Hasan, and Md. Abu Baten. 2024. AutoML: A systematic review on automated machine learning with neural architecture search. *Journal of Information and Intelligence* 2, 1 (2024), 52–81. [https://doi.org/10.1016/j.jiixd.2023.10.002](https://doi.org/10.1016/j.jiixd.2023.10.002)
*   Shen et al. (2024) Weizhou Shen, Chenliang Li, Hongzhan Chen, Ming Yan, Xiaojun Quan, Hehong Chen, Ji Zhang, and Fei Huang. 2024. Small LLMs Are Weak Tool Learners: A Multi-LLM Agent. arXiv:2401.07324 [cs.AI]
*   Tang et al. (2024) Xiangru Tang, Yuliang Liu, Zefan Cai, Yanjun Shao, Junjie Lu, Yichi Zhang, Zexuan Deng, Helan Hu, Kaikai An, Ruijun Huang, Shuzheng Si, Sheng Chen, Haozhe Zhao, Liang Chen, Yan Wang, Tianyu Liu, Zhiwei Jiang, Baobao Chang, Yin Fang, Yujia Qin, Wangchunshu Zhou, Yilun Zhao, Arman Cohan, and Mark Gerstein. 2024. ML-Bench: Evaluating Large Language Models and Agents for Machine Learning Tasks on Repository-Level Code. arXiv:2311.09835 [cs.CL] [https://arxiv.org/abs/2311.09835](https://arxiv.org/abs/2311.09835)
*   Team et al. (2023) Gemini Team, Rohan Anil, Sebastian Borgeaud, et al. 2023. Gemini: A Family of Highly Capable Multimodal Models. arXiv:2312.11805 [cs.CL]
*   Touvron et al. (2023) Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, Dan Bikel, Lukas Blecher, Cristian Canton Ferrer, Moya Chen, Guillem Cucurull, David Esiobu, Jude Fernandes, Jeremy Fu, Wenyin Fu, Brian Fuller, Cynthia Gao, Vedanuj Goswami, Naman Goyal, Anthony Hartshorn, Saghar Hosseini, Rui Hou, Hakan Inan, Marcin Kardas, Viktor Kerkez, Madian Khabsa, Isabel Kloumann, Artem Korenev, Punit Singh Koura, Marie-Anne Lachaux, Thibaut Lavril, Jenya Lee, Diana Liskovich, Yinghai Lu, Yuning Mao, Xavier Martinet, Todor Mihaylov, Pushkar Mishra, Igor Molybog, Yixin Nie, Andrew Poulton, Jeremy Reizenstein, Rashi Rungta, Kalyan Saladi, Alan Schelten, Ruan Silva, Eric Michael Smith, Ranjan Subramanian, Xiaoqing Ellen Tan, Binh Tang, Ross Taylor, Adina Williams, Jian Xiang Kuan, Puxin Xu, Zheng Yan, Iliyan Zarov, Yuchen Zhang, Angela Fan, Melanie Kambadur, Sharan Narang, Aurelien Rodriguez, Robert Stojnic, Sergey Edunov, and Thomas Scialom. 2023. Llama 2: Open Foundation and Fine-Tuned Chat Models. arXiv:2307.09288 [cs.CL]
*   Veličković et al. (2022) Petar Veličković, Adrià Puigdomènech Badia, David Budden, Razvan Pascanu, Andrea Banino, Misha Dashevskiy, Raia Hadsell, and Charles Blundell. 2022. The CLRS Algorithmic Reasoning Benchmark. arXiv:2205.15659 [cs.LG]
*   Wang et al. (2023) Hongyi Wang, Felipe Maia Polo, Yuekai Sun, Souvik Kundu, Eric Xing, and Mikhail Yurochkin. 2023. Fusing Models with Complementary Expertise. arXiv:2310.01542 [cs.LG]
*   Wang et al. (2024) Xingyao Wang, Yangyi Chen, Lifan Yuan, Yizhe Zhang, Yunzhu Li, Hao Peng, and Heng Ji. 2024. Executable Code Actions Elicit Better LLM Agents. arXiv:2402.01030 [cs.CL]
*   Warstadt et al. (2023) Alex Warstadt, Leshem Choshen, Aaron Mueller, Adina Williams, Ethan Wilcox, and Chengxu Zhuang. 2023. Call for Papers – The BabyLM Challenge: Sample-efficient pretraining on a developmentally plausible corpus. arXiv:2301.11796 [cs.CL]
*   Woodward et al. (2023) Ben Woodward, eor123, Genevieve Patterson, and Lilli Carlsen. 2023. FathomNet 2023. [https://kaggle.com/competitions/fathomnet-out-of-sample-detection](https://kaggle.com/competitions/fathomnet-out-of-sample-detection)
*   Wu et al. (2023) Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Beibin Li, Erkang Zhu, Li Jiang, Xiaoyun Zhang, Shaokun Zhang, Jiale Liu, Ahmed Hassan Awadallah, Ryen W White, Doug Burger, and Chi Wang. 2023. AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation. arXiv:2308.08155 [cs.AI]
*   Yeadon et al. (2024) Will Yeadon, Alex Peach, and Craig P. Testrow. 2024. A comparison of Human, GPT-3.5, and GPT-4 Performance in a University-Level Coding Course. arXiv:2403.16977 [cs.CL]
*   Yi et al. (2024) Zihao Yi, Jiarui Ouyang, Yuwen Liu, Tianhao Liao, Zhe Xu, and Ying Shen. 2024. A Survey on Recent Advances in LLM-Based Multi-turn Dialogue Systems. arXiv:2402.18013 [cs.CL]
*   Yue et al. (2024) Murong Yue, Jie Zhao, Min Zhang, Liang Du, and Ziyu Yao. 2024. Large Language Model Cascades with Mixture of Thoughts Representations for Cost-efficient Reasoning. arXiv:2310.03094 [cs.CL]
*   Zan et al. (2023) Daoguang Zan, Bei Chen, Fengji Zhang, Dianjie Lu, Bingchao Wu, Bei Guan, Wang Yongji, and Jian-Guang Lou. 2023. Large Language Models Meet NL2Code: A Survey. In *Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, Anna Rogers, Jordan Boyd-Graber, and Naoaki Okazaki (Eds.). Association for Computational Linguistics, Toronto, Canada, 7443–7464. [https://doi.org/10.18653/v1/2023.acl-long.411](https://doi.org/10.18653/v1/2023.acl-long.411)
*   Zan et al. (2024) Daoguang Zan, Ailun Yu, Wei Liu, Dong Chen, Bo Shen, Wei Li, Yafen Yao, Yongshun Gong, Xiaolin Chen, Bei Guan, Zhiguang Yang, Yongji Wang, Qianxiang Wang, and Lizhen Cui. 2024. CodeS: Natural Language to Code Repository via Multi-Layer Sketch. arXiv:2403.16443 [cs.CL] [https://arxiv.org/abs/2403.16443](https://arxiv.org/abs/2403.16443)
*   Zhang et al. (2023) Jieyu Zhang, Ranjay Krishna, Ahmed H. Awadallah, and Chi Wang. 2023. EcoAssistant: Using LLM Assistant More Affordably and Accurately. arXiv:2310.03046 [cs.SE]
*   Zhang et al. (2024b) Kechi Zhang, Jia Li, Ge Li, Xianjie Shi, and Zhi Jin. 2024b. CodeAgent: Enhancing Code Generation with Tool-Integrated Agent Systems for Real-World Repo-level Coding Challenges. arXiv:2401.07339 [cs.SE]
*   Zhang et al. (2024c) Lei Zhang, Yuge Zhang, Kan Ren, Dongsheng Li, and Yuqing Yang. 2024c. MLCopilot: Unleashing the Power of Large Language Models in Solving Machine Learning Tasks. arXiv:2304.14979 [cs.LG] [https://arxiv.org/abs/2304.14979](https://arxiv.org/abs/2304.14979)
*   Zhang et al. (2024a) Ziyin Zhang, Chaoyu Chen, Bingchang Liu, Cong Liao, Zi Gong, Hang Yu, Jianguo Li, and Rui Wang. 2024a. Unifying the Perspectives of NLP and Software Engineering: A Survey on Language Models for Code. arXiv:2311.07989 [cs.CL]
*   Zheng et al. (2024) Zibin Zheng, Kaiwen Ning, Yanlin Wang, Jingwen Zhang, Dewu Zheng, Mingxi Ye, and Jiachi Chen. 2024. A Survey of Large Language Models for Code: Evolution, Benchmarking, and Future Trends. arXiv:2311.10372 [cs.SE]
*   Zhong et al. (2024) Lily Zhong, Zilong Wang, and Jingbo Shang. 2024. LDB: A Large Language Model Debugger via Verifying Runtime Execution Step-by-step. arXiv:2402.16906 [cs.SE]
*   Zhu et al. (2023) Wenhao Zhu, Hongyi Liu, Qingxiu Dong, Jingjing Xu, Shujian Huang, Lingpeng Kong, Jiajun Chen, and Lei Li. 2023. Multilingual Machine Translation with Large Language Models: Empirical Results and Analysis. arXiv:2304.04675 [cs.CL]