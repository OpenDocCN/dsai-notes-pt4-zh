<!--yml
category: 未分类
date: 2025-01-11 12:42:51
-->

# AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks

> 来源：[https://arxiv.org/html/2404.06411/](https://arxiv.org/html/2404.06411/)

Luca Gioacchini${}^{1,2}$, Giuseppe Siracusano${}^{1}$, Davide Sanvito${}^{1}$, Kiril Gashteovski${}^{1,3}$,
David Friede${}^{1}$, Roberto Bifulco${}^{1}$, Carolin Lawrence${}^{1}$
${}^{1}$ NEC Laboratories Europe, Heidelberg, Germany
${}^{2}$ Politecnico di Torino, Turin, Italy
${}^{3}$ CAIR, Ss. Cyril and Methodius University, Skopje, North Macedonia

# AgentQuest: A Modular Benchmark Framework
to Measure Progress and Improve LLM Agents

Luca Gioacchini${}^{1,2}$, Giuseppe Siracusano${}^{1}$, Davide Sanvito${}^{1}$, Kiril Gashteovski${}^{1,3}$,
David Friede${}^{1}$, Roberto Bifulco${}^{1}$, Carolin Lawrence${}^{1}$
${}^{1}$ NEC Laboratories Europe, Heidelberg, Germany
${}^{2}$ Politecnico di Torino, Turin, Italy
${}^{3}$ CAIR, Ss. Cyril and Methodius University, Skopje, North Macedonia

###### Abstract

The advances made by Large Language Models (LLMs) have led to the pursuit of LLM agents that can solve intricate, multi-step reasoning tasks. As with any research pursuit, benchmarking and evaluation are key corner stones to efficient and reliable progress. However, existing benchmarks are often narrow and simply compute overall task success. To face these issues, we propose AgentQuest ¹¹1Demo provided at [https://youtu.be/0JNkIfwnoak](https://youtu.be/0JNkIfwnoak). – a framework where (i) both benchmarks and metrics are modular and easily extensible through well documented and easy-to-use APIs; (ii) we offer two new evaluation metrics that can reliably track LLM agent progress while solving a task. We exemplify the utility of the metrics on two use cases wherein we identify common failure points and refine the agent architecture to obtain a significant performance increase. Together with the research community, we hope to extend AgentQuest further and therefore we make it available under [https://github.com/nec-research/agentquest](https://github.com/nec-research/agentquest).

AgentQuest: A Modular Benchmark Framework
to Measure Progress and Improve LLM Agents

Luca Gioacchini${}^{1,2}$, Giuseppe Siracusano${}^{1}$, Davide Sanvito${}^{1}$, Kiril Gashteovski${}^{1,3}$, David Friede${}^{1}$, Roberto Bifulco${}^{1}$, Carolin Lawrence${}^{1}$ ${}^{1}$ NEC Laboratories Europe, Heidelberg, Germany ${}^{2}$ Politecnico di Torino, Turin, Italy ${}^{3}$ CAIR, Ss. Cyril and Methodius University, Skopje, North Macedonia

## 1 Introduction

Generative Agents Kiela et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib5)) are software systems that leverage foundation models like Large Language Models (LLMs) to perform complex tasks, take decisions, devise multi-steps plans and use tools (API calls, coding, etc.) to build solutions in heterogeneous contexts Wang et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib19)); Weng ([2023](https://arxiv.org/html/2404.06411v1#bib.bib20)). The potential ability to solve heterogeneous tasks with high degrees of autonomy has catalysed the interest of both research and industrial communities. Nonetheless, it is still unclear to which extent current systems are successfully able to fulfil their promises. In fact, methodologies to benchmark, evaluate and advance these systems are still in their early days.

![Refer to caption](img/4d48acc3a24047d59d814cfc66df6ef6.png)

(a) Existing

![Refer to caption](img/62331c7bb1368fd2d0b43335e567228c.png)

(b) AgentQuest

Figure 1: Overview of agent-benchmark interactions in existing frameworks and in AgentQuest. AgentQuest defines a common interface to interact with the benchmarks and to compute progress metrics, easing the addition of new benchmarks and allowing researchers to evaluate and debug their agent architectures.

We identify a couple of gaps. Firstly, benchmarking agents requires combining different benchmark types Liu et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib8)); Chalamalasetti et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib1)). For example, some benchmarks focus on specific capabilities and provide gaming environments, which we refer to as “closed-box” – i.e. with a finite set of actions Liu et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib8)); Patil et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib14)); Chalamalasetti et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib1)) – whereas other benchmarks provide open-ended tasks and access to general tools, like web browsing Zhuang et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib25)); Zheng et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib24)); Mialon et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib9)). As benchmarks are developed independently, significant effort goes into custom integration of new agent architectures with each benchmark.

Secondly, and more critically, existing benchmarks mostly focus on providing a *success rate* measure, i.e. a binary success/fail evaluation for each of the proposed tasks. While success rate is helpful to measure overall advances of an agent technology, it has limited use in guiding improvements for new generative agent architectures. Here, it is important to consider that generative agents often combine foundation models with multiple other components, such as memory and tools. Developers can reason about these individual components in terms of architecture and their inter-dependence, and could actively change and evolve them using deeper insights about how an agent performs in a benchmark. That is, developers need benchmarks to both evaluate and *debug* agents.

For example, current benchmarks make it hard to answer questions like does the agent fail completely the tasks or does it partially solve them? Does the agent fail consistently at a certain step? Would extra run time lead to finding a solution? Answering these questions would require tracing and inspecting the execution of the agent. We argue that providing a more efficient approach that is consistent over multiple benchmarks is a stepping stone towards evolving generative agents.

We address these gaps introducing AgentQuest, a modular framework to support multiple diverse benchmarks and agent architectures (See Figure [1](https://arxiv.org/html/2404.06411v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks")), alongside with two new metrics – i.e. progress rate and repetition rate – to debug an agent architecture behaviour. AgentQuest defines a standard interface to connect an arbitrary agent architecture with diverse benchmarks, and to compute progress and repetition rates from them.

We showcase the framework, implementing 4 benchmarks in AgentQuest: ALFWorld Shridhar et al. ([2020](https://arxiv.org/html/2404.06411v1#bib.bib15)), Lateral Thinking Puzzles Sloane ([1992](https://arxiv.org/html/2404.06411v1#bib.bib16)), Mastermind and Sudoku. The latter two are newly introduced with AgentQuest. Additional benchmarks can be easily added, while requiring no changes to the tested agents.

Our final contribution is to present our experience leveraging the proposed metrics to debug and improve existing agent architectures as implemented in LangChain Chase ([2022](https://arxiv.org/html/2404.06411v1#bib.bib2)). In particular, we show that in the Mastermind benchmark the combination of progress rate and repetition rate identifies a limitation in the ability of the agent to explore the full space of potential solutions. Guided by this insight we could improve the success rate in this benchmark by up to $\approx$20%. In Lateral Thinking Puzzles we show that partially repeating actions is part of the agent strategy, whereas in ALFWorld, we show that monitoring the progress rate makes it possible to identify that the final success rate is limited by the allowed runtime of the agent, and that more steps lead to a better performance. Finally, in the Sudoku benchmark, we show that the low success rate is actually paired with low progress rate, making clear that the tested agent is unable to solve this type of tasks.

## 2 Generative AI Agents in a Nutshell

Generative AI agents are automated systems relying on software components integrated with LLMs pre-trained on large amount of data for language understanding and processing. When assigned a task, an agent engages in a systematic process: it iteratively formulates self-generated instructions, executes them, and observes the outcomes until the ultimate objective is achieved. Next, we showcase the basic interaction between agents and the environment in which they operate and describe the standard benchmarking techniques.

### 2.1 Agent-Environment interaction

Closely following the terminology in Reinforcement Learning (RL)²²2Unlike RL scenarios, the agent does not need a further training process. It relies on the pre-trained LLM and does not perform an action under the influence of any reward. Sutton and Barto ([2018](https://arxiv.org/html/2404.06411v1#bib.bib18)), the core elements defining the agent-environment interaction are *environment*, *state*, *observation* and *action* (see [Figure 0(a)](https://arxiv.org/html/2404.06411v1#S1.F0.sf1 "0(a) ‣ Figure 1 ‣ 1 Introduction ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks")).

#### Environment and states.

The environment refers to the external system the agent interacts with. In this context, we treat the benchmark and the environment as synonyms. It is typically described through a finite set of hidden *states*, which are not directly observable by the agent and represent the benchmark configuration.

#### Observations and actions.

The agent interacts with the environment for multiple execution steps. At each step, the environment produces an *observation* providing information about its current hidden state. The agent uses the internal LLM to process the received observation. Being pre-trained on general knowledge data, the LLM engages a reasoning process generating a *thought* on the observation (e.g. the planned strategy to follow in the current step or the usage of a tool). According to this thought, the agent provides the environment an *action* to modify the current hidden state. ³³3Unlike RL, the LLM outputs are unconstrained, and any provided action is considered valid.

The following is an example of an agent interacting with a closed-box environment to solve a code-breaking challenge, i.e. the Mastermind game Stuckman and Zhang ([2005](https://arxiv.org/html/2404.06411v1#bib.bib17)). The agent is tasked to guess a four-digit code. At each step, the agent devises a strategy and acts accordingly to guess the code. The environment provides a feedback on the number of guessed digits in correct and wrong positions. The process continues until a stopping condition is met.

[⬇](data:text/plain;base64,PEBcdGV4dGNvbG9ye2ZvcmVzdGdyZWVufXtcdGV4dGJme0VOVklST05NRU5UfTogSW5pdGlhbGl6aW5nIGEgbmV3IGVudmlyb25tZW50Li4ufUA+CjxAXHRleHRjb2xvcntpbnRlcm5hdGlvbmFsa2xlaW5ibHVlfXtcdGV4dGJme09CU0VSVkFUSU9OfTogU3RhcnQgZ3Vlc3NpbmcgdGhlIDQgZGlnaXRzIGNvZGUufUA+CjxAXHRleHRjb2xvcnttYWdlbnRhfXtcdGV4dGJme1RIT1VHSFR9OiBJJ2xsIHN0YXJ0IGJ5IGd1ZXNzaW5nIDEyMzQgdG8gZ2V0IGFuIGlkZWEgb2Ygd2hpY2h9QD4KPEBcdGV4dGNvbG9ye21hZ2VudGF9e29mIHRoZXNlIG51bWJlcnMgYXJlIGluIHRoZSBzZWNyZXQgbnVtYmVyIGFuZCBpZiBhbnkgYXJlIGlufUA+CjxAXHRleHRjb2xvcnttYWdlbnRhfXt0aGUgY29ycmVjdCBwb3NpdGlvbi59QD4KPEBcdGV4dGNvbG9ye2dvbGRlbnllbGxvd317XHRleHRiZntBQ1RJT059OiAxMjM0fUA+Cgo8QFx0ZXh0Y29sb3J7Zm9yZXN0Z3JlZW59e1x0ZXh0YmZ7RU5WSVJPTk1FTlR9OiBDb21wYXJpbmcgdGhlIGd1ZXNzZWQgY29kZSB3aXRoIHRoZSB0cnV0aC4uLn1APgo8QFx0ZXh0Y29sb3J7aW50ZXJuYXRpb25hbGtsZWluYmx1ZX17XHRleHRiZntPQlNFUlZBVElPTn06IFlvdXIgZ3Vlc3MgaGFzIDEgY29ycmVjdCBudW1iZXJzIGluIHRoZSB3cm9uZyB9QD4KPEBcdGV4dGNvbG9ye2ludGVybmF0aW9uYWxrbGVpbmJsdWV9e3Bvc2l0aW9uIGFuZCAwIGNvcnJlY3QgbnVtYmVycyBpbiB0aGUgY29ycmVjdCBwb3NpdGlvbi4gfUA+CjxAXHRleHRjb2xvcntpbnRlcm5hdGlvbmFsa2xlaW5ibHVlfXtLZWVwIGd1ZXNzaW5nLi4ufUA+CjxAXHRleHRjb2xvcnttYWdlbnRhfXtcdGV4dGJme1RIT1VHSFR9OiBPbmUgb2YgdGhlIGRpZ2l0cyAxLCAyLCAzLCBvciA0IGlzIGNvcnJlY3QgYnV0IGlufUA+CjxAXHRleHRjb2xvcnttYWdlbnRhfXt0aGUgd3JvbmcgcG9zaXRpb24uIEknbGwgdHJ5IHRvIG1vdmUgZWFjaCBkaWdpdCB0byBhfUA+CjxAXHRleHRjb2xvcnttYWdlbnRhfXtkaWZmZXJlbnQgcG9zaXRpb24gdG8gc2VlIGlmIEkgY2FuIGdldCBhIGNvcnJlY3QgcG9zaXRpb24ufUA+CjxAXHRleHRjb2xvcntnb2xkZW55ZWxsb3d9e1x0ZXh0YmZ7QUNUSU9OfTogMjE0M31APgouLi4=)ENVIRONMENT: Initializing a new environment...OBSERVATION: Start guessing the 4 digits code.THOUGHT: I’ll start by guessing 1234 to get an idea of whichof these numbers are in the secret number and if any are inthe correct position.ACTION: 1234ENVIRONMENT: Comparing the guessed code with the truth...OBSERVATION: Your guess has 1 correct numbers in the wrongposition and 0 correct numbers in the correct position.Keep guessing...THOUGHT: One of the digits 1, 2, 3, or 4 is correct but inthe wrong position. I’ll try to move each digit to adifferent position to see if I can get a correct position.ACTION: 2143...

### 2.2 Benchmarking an agent

When evaluating agents performance on a benchmark, the following two metrics are commonly used Liu et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib8)): (i) Success Rate (SR), i.e. the ratio of successful tasks to the total instances; (ii) Time to Success, i.e. the average time required to obtain a solution. While important and trending metrics Chalamalasetti et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib1)); Hessel et al. ([2022](https://arxiv.org/html/2404.06411v1#bib.bib4)); Zhang et al. ([2020a](https://arxiv.org/html/2404.06411v1#bib.bib22)), they exclusively address the final success. They cannot measure intermediate success or failure and therefore make it difficult to understand why agents might systematically fail and how they can be improved. In contrast, we want to define intermediate metrics that allow us to easily assess and compare the performance of agents across a wide range of tasks.

## 3 AgentQuest Overview

We designed AgentQuest as a separation layer between agent and environment (see [Figure 0(b)](https://arxiv.org/html/2404.06411v1#S1.F0.sf2 "0(b) ‣ Figure 1 ‣ 1 Introduction ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks")). Essentially, it offers (i) a unified interface (i.e. the *driver*) ensuring compatibility between different agent architectures and benchmarks with minimal programming efforts (Section [3.1](https://arxiv.org/html/2404.06411v1#S3.SS1 "3.1 Benchmarks common interface ‣ 3 AgentQuest Overview ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks")); (ii) the implementation of two metrics beyond task success (i.e. *progress rate* and *repetition rate*) aimed at monitoring the agent advancement toward the final goal and allowing us to understand the reasons behind failures (Section [3.2](https://arxiv.org/html/2404.06411v1#S3.SS2 "3.2 Understanding agent advancements ‣ 3 AgentQuest Overview ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks")); (iii) a unique vantage point and interface for implementing new metrics to monitoring and measuring the execution (Section [3.3](https://arxiv.org/html/2404.06411v1#S3.SS3 "3.3 Adding new metrics ‣ 3 AgentQuest Overview ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks")).

### 3.1 Benchmarks common interface

Different benchmarks require invoking distinct functions, using specific formats, and performing parsing and post-processing of observations and agent actions. To integrate different agent architectures, the common trend is hardcoding such benchmark-specific requirements directly in the framework (Liu et al. [2023](https://arxiv.org/html/2404.06411v1#bib.bib8); Chalamalasetti et al. [2023](https://arxiv.org/html/2404.06411v1#bib.bib1), inter alia). This results in many custom interfaces tailored on each environment, making it difficult to easily move to other benchmarks and agent architectures.

Instead, AgentQuest exposes a single unified Python interface, i.e. the Driver and two classes reflecting the agent-environment interaction components (i.e. Observation, Action).

#### Observations and actions.

We provide two simple classes: Observation and Action. The first has two required attributes: (i) output, a string reporting information about the environment state; (ii) done, a Boolean variable indicating if the final task is currently accomplished or not. The Action class has one required attribute, action_value. It is a string directly output by the agent. Once processed and provided to the environment, it triggers the environment change. To customise the interactions, developers can define optional attributes.

#### Driver.

We provide the Driver class with two mandatory methods: (i) the reset method initialises a new instance of the environment and returns the first observation; (ii) the step method performs one single execution step. It accepts one instance of the Action class from the agent, processes the action (e.g. parses the action_value string) and uses it to modify the environment state. It always returns an observation. The driver supports also the benchmark-specific state attribute, acting as a simple API. It exposes the environment state at step $t$, useful to compute the progress rate.

We here provide an example of the implemented interaction for Mastermind:

[⬇](data:text/plain;base64,ZnJvbSBhZ2VudHF1ZXN0LmRyaXZlcnMgaW1wb3J0IE1hc3Rlck1pbmREcml2ZXIKZnJvbSBhZ2VudHF1ZXN0LnV0aWxzIGltcG9ydCBBY3Rpb24KZnJvbSBhZ2VudHF1ZXN0Lm1ldHJpY3MgaW1wb3J0IGdldF9wcm9ncmVzcywgZ2V0X3JlcGV0aXRpb24KCmFnZW50ID0gLi4uICMgSW5pdGlhbGl6ZSB5b3VyIGFnZW50CmFjdGlvbnMsIHByb2dyZXNzLCByZXBldGl0aW9ucyA9IFtdLCBbXSwgW10KIyBJbml0aWFsaXplIHRoZSBlbnZpcm9ubWVudCBhbmQgcmVzZXQgcm91bmQKZHJpdmVyID0gTWFzdGVyTWluZERyaXZlcih0cnV0aD0nNTYxOCcpCm9icyA9IGRyaXZlci5yZXNldCgpCiMgQWdlbnQgbG9vcAp3aGlsZSBub3Qgb2JzLmRvbmU6CiAgICBndWVzcyA9IGFnZW50KG9icy5vdXRwdXQpICMgR2V0IHRoZSBhZ2VudCBvdXRwdXQKICAgIGFjdGlvbiA9IEFjdGlvbihhY3Rpb25fdmFsdWU9Z3Vlc3MpICMgQ3JlYXRlIGFjdGlvbgogICAgYWN0aW9ucy5hcHBlbmQoYWN0aW9uLmFjdGlvbl92YWx1ZSkgIyBTdG9yZSBhY3Rpb24KICAgIG9icyA9IGRyaXZlci5zdGVwKGFjdGlvbikgIyBFeGVjdXRlIHN0ZXAKICAgICMgQ29tcHV0ZSBjdXJyZW50IHByb2dyZXNzIGFuZCByZXBldGl0aW9uCiAgICBwcm9ncmVzcy5hcHBlbmQoZ2V0X3Byb2dyZXNzKGRyaXZlci5zdGF0ZSwgJzU2MTgnKSkKICAgIHJlcGV0aXRpb25zLmFwcGVuZChnZXRfcmVwZXRpdGlvbnMoYWN0aW9ucykpCiAgICAjIEV4dGVuZCB3aXRoIHlvdXIgY3VzdG9tIG1ldHJpY3MgaGVyZSAuLi4KIyBDb21wdXRlIGZpbmFsIG1ldHJpY3MKUFIgPSBbeC9sZW4oJzU2MTgnKSBmb3IgeCBpbiBwcm9ncmVzc10KUlIgPSBbeC8obGVuKGFjdGlvbnMpLTEpIGZvciB4IGluIHJlcGV0aXRpb25zXQ==)from agentquest.drivers import MasterMindDriverfrom agentquest.utils import Actionfrom agentquest.metrics import get_progress, get_repetitionagent = ... # Initialize your agentactions, progress, repetitions = [], [], []# Initialize the environment and reset rounddriver = MasterMindDriver(truth=’5618’)obs = driver.reset()# Agent loopwhile not obs.done:    guess = agent(obs.output) # Get the agent output    action = Action(action_value=guess) # Create action    actions.append(action.action_value) # Store action    obs = driver.step(action) # Execute step    # Compute current progress and repetition    progress.append(get_progress(driver.state, ’5618’))    repetitions.append(get_repetitions(actions))    # Extend with your custom metrics here ...# Compute final metricsPR = [x/len(’5618’) for x in progress]RR = [x/(len(actions)-1) for x in repetitions]

### 3.2 Understanding agent advancements

Getting insights on how they tackle a specific task is key to comprehend agent behaviours, capabilities and limitations. Furthermore, identifying systematic agent failures allows to pinpoint necessary adjustments within the architecture to effectively address the underlying issues.

AgentQuest contributes towards this direction introducing two cross-benchmark metrics, the *progress rate* and the *repetition rate*. While the first expresses *how much* the agent is advancing towards the final goal, the latter indicates *how* it is reaching it, with a specific focus on the amount of repeated (i.e. similar) actions the agent performs.

#### Milestones and progress rate.

To quantify the agent advancement towards the final goal, AgentQuest uses a set of *milestones* $\mathcal{M}$. In a nutshell, we break down the final solution into a series of environment hidden states the agent needs to reach to get the final solution of the task, hence, $\mathcal{M}\subseteq\mathcal{S}$, where $\mathcal{S}$ is the set of hidden states. The magnitude of $\mathcal{M}$ determines the level of *granularity* in the evaluation process. Specifically, when $\mathcal{M}$ aligns closely with $\mathcal{S}$, it offers a more comprehensive insight into the agent progress, resulting in finer granularity, whereas for $|\mathcal{M}|=1$ the evaluation coincides with the success rate.

We assign a score to all the states included in $\mathcal{M}$ through a scoring function $f$ and, at execution step $t$, we define the *progress rate* $\text{PR}_{t}:\mathcal{S}\to[0,1]$ dependant of such scoring function, as an indication of how far the agent is from the goal, allowing to track agent progress over time. Depending on the benchmark, the progress rate might also decrease during the execution. Milestones can either be manually annotated, or internally computed.

#### Repetition rate.

The repetition rate $\text{RR}_{t}$ is a measure of the agent tendency of repeating actions. Depending on the benchmark, we do not consider repetitions as a limitation – e.g. solving a maze requires repetitions, such as going left repeatedly. See also [Section 4](https://arxiv.org/html/2404.06411v1#S4 "4 Insights via AgentQuest ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks") for a positive and negative example of repetitions.

At execution step $t$, we consider the set of unique actions taken by the agent up to $t-1$, $\mathcal{A}_{t-1}$. Then, we compute the similarity function $g$ between the current action $a_{t}$ and all the previous ones in $\mathcal{A}_{t-1}$. As any action generated by the LLM is considered valid, we consider the action $a_{t}$ as *repeated* if it exists at least one previous action $a\in\mathcal{A}_{t-1}$ such that $g(a_{t},a)\geq\theta_{a}$, where $\theta_{a}\in[0,1]$ is the *resolution*.⁴⁴4A higher resolution demands closer matches for classification as repeated actions, while lower values broaden the spectrum of qualifying action similarities. If the action is not repeated, we update the set of unique actions as $A_{t}=A_{t-1}\cup a_{t}$.

Based on this, we define the repetition rate at step $t$ as the cumulative number of repeated actions normalised by the number of execution steps, $T$, except for the first. Formally, $\text{RR}_{t}=\frac{t-|A_{t}|}{T-1}$.

### 3.3 Adding new metrics

Table 1: Attributes exposing components of the agent-environment interaction useful to define new metrics.

| Class | Attribute | Access to |
| Driver | state | Hidden states |
| Observation | output | Observations |
| Action | action$\_$value | Agent actions |

We rely on the progress and repetition rates to show how AgentQuest can be extended with new metrics through a simple function template. We then show the implementations of the functions adapted to the considered benchmark.

#### Metric function template.

We use a Python function template to easily define the elements of the agent-environment interactions required for computing a given metric. [Table 1](https://arxiv.org/html/2404.06411v1#S3.T1 "Table 1 ‣ 3.3 Adding new metrics ‣ 3 AgentQuest Overview ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks") provides a recap of the main attributes and reference classes that can be used as input for the custom metrics. Additionally, users can provide external data, like milestones or action history.

Table 2: Overview of the benchmarks provided in AgentQuest.

| Benchmark | Description | Milestones |
| Mastermind |  

&#124; Guessing a numeric code with feedback on guessed digits and positions. &#124;

 |  

&#124; Digits of the code to guess. &#124;

 |
| LTP |  

&#124; Solving riddles by asking Yes/No questions. &#124;

 |  

&#124; Guessed riddle key aspects. &#124;

 |
| ALFWorld |  

&#124; Finding an object in a textual world and using it. &#124;

 |  

&#124; Sequence of actions. &#124;

 |
| Sudoku |  

&#124; 9x9 grid puzzle. Digits 1-9 fill each column, row, and 3x3 sub-grid &#124;
&#124; without repetition. &#124;

 |  

&#124; Total number of correct &#124;
&#124; inserted digits. &#124;

 |

#### Implement progress rate.

Depending on the benchmark, developers need to implement the custom scoring function $f$ through the get_progress function and define the set of milestones $\mathcal{M}$. Milestones can either be user-defined or internally computed within get_progress. Here, we show the definition of get_progress to quantify the achieved milestones for Mastermind. The milestones are the digits of the final solution and the progress indicates the count of correctly guessed digits in their positions:

[⬇](data:text/plain;base64,ZGVmIGdldF9wcm9ncmVzcyhzdGF0ZSwgbWlsZXN0b25lcyk6CiAgICByZWFjaGVkX21pbGVzdG9uZXMgPSAwICMgRGlnaXRzIGluIGNvcnJlY3QgcG9zaXRpb24KICAgIGZvciBpLCBqIGluIHppcChzdGF0ZSwgbWlsZXN0b25lcyk6CiAgICAgICAgaWYgaSA9PSBqOiByZWFjaGVkX21pbGVzdG9uZXMgKz0gMQogICAgcmV0dXJuIHJlYWNoZWRfbWlsZXN0b25lcwoKIyBVc2FnZSBleGFtcGxlLiBUaGUgY29kZSB0byBndWVzcyBpcyAnNTYxOCcKcHJvZ3Jlc3MgPSBnZXRfcHJvZ3Jlc3MoJzIzMTgnLCAnNTYxOCcpICMgUmVhY2hlZCBtaWxlc3RvbmVzCj4+PiAyCnByb2dyZXNzL2xlbignNTYxOCcpICMgQ29tcHV0ZSBQcm9ncmVzcyBSYXRlCj4+PiAwLjU=)def get_progress(state, milestones):    reached_milestones = 0 # Digits in correct position    for i, j in zip(state, milestones):        if i == j: reached_milestones += 1    return reached_milestones# Usage example. The code to guess is ’5618’progress = get_progress(’2318’, ’5618’) # Reached milestones>>> 2progress/len(’5618’) # Compute Progress Rate>>> 0.5

#### Implement repetition rate.

To determine if an action is repeated, the end user must define the similarity function $g$ according to the considered benchmark. We provide the get_repetitions template function to compute the number of repeated actions. Here, we illustrate its implementation in Python and provide a usage example for Mastermind, where $g$ is the Levenshtein similarity Levenshtein ([1966](https://arxiv.org/html/2404.06411v1#bib.bib6)).

[⬇](data:text/plain;base64,ZnJvbSBMZXZlbnNodGVpbiBpbXBvcnQgcmF0aW8gYXMgZwoKZGVmIGdldF9yZXBldGl0aW9ucyhhY3Rpb25zLCBUSEVUQV9BKToKICAgIHVuaXF1ZV9hY3QgPSBzZXQoKSAjIEluaXRpYWxpc2UgdW5pcXVlIGFjdGlvbnMKICAgIGZvciBpLGEgaW4gZW51bWVyYXRlKGFjdGlvbnMpOgogICAgICAgICMgQ2hlY2sgZm9yIHJlcGV0aXRpb25zCiAgICAgICAgaWYgYWxsKFtnKGEsYWN0aW9uc1t4XSk8VEhFVEFfQSBmb3IgeCBpbiByYW5nZShpKV0pOgogICAgICAgICAgICB1bmlxdWVfYWN0LmFkZChhKQogICAgcmV0dXJuIGxlbihhY3Rpb25zKS1sZW4odW5pcXVlX2FjdCkKCiMgVXNhZ2UgZXhhbXBsZS4gVGhlIGNvZGUgdG8gZ3Vlc3MgaXMgJzU2MTgnCmFjdGlvbnMgPSBbJzEyMzQnLCAnMjE0MycsICcxMjM0JywgJzU2MTgnXSAjIEFjdGlvbnMgaGlzdG9yeQpyZXBldGl0aW9ucyA9IGdldF9yZXBldGl0aW9ucyhhY3Rpb25zLCAxLjApCj4+PiAxIHJlcGVhdGVkIGFjdGlvbgojIENvbXB1dGUgUmVwZXRpdGlvbiBSYXRlCnJlcGV0aXRpb25zLyhsZW4oYWN0aW9ucyktMSkKPj4+IDAuMzM=)from Levenshtein import ratio as gdef get_repetitions(actions, THETA_A):    unique_act = set() # Initialise unique actions    for i,a in enumerate(actions):        # Check for repetitions        if all([g(a,actions[x])<THETA_A for x in range(i)]):            unique_act.add(a)    return len(actions)-len(unique_act)# Usage example. The code to guess is ’5618’actions = [’1234’, ’2143’, ’1234’, ’5618’] # Actions historyrepetitions = get_repetitions(actions, 1.0)>>> 1 repeated action# Compute Repetition Raterepetitions/(len(actions)-1)>>> 0.33

In other cases, where $a$ can be any text string, we can use standard metrics, such as BLEU Papineni et al. ([2002](https://arxiv.org/html/2404.06411v1#bib.bib12)), ROUGE Lin ([2004](https://arxiv.org/html/2404.06411v1#bib.bib7)) or BERTScore Zhang et al. ([2020b](https://arxiv.org/html/2404.06411v1#bib.bib23)).

## 4 Insights via AgentQuest

We investigate agent behaviours in different reasoning scenarios by proposing a starting set of four benchmarks. We implemented from scratch Sudoku Felgenhauer and Jarvis ([2006](https://arxiv.org/html/2404.06411v1#bib.bib3)) and Mastermind Stuckman and Zhang ([2005](https://arxiv.org/html/2404.06411v1#bib.bib17)) environments, while ALFWorld Shridhar et al. ([2020](https://arxiv.org/html/2404.06411v1#bib.bib15)) and Lateral Thinking Puzzles (LTP)Sloane ([1992](https://arxiv.org/html/2404.06411v1#bib.bib16)) are existing implementations Liu et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib8)). [Table 2](https://arxiv.org/html/2404.06411v1#S3.T2 "Table 2 ‣ Metric function template. ‣ 3.3 Adding new metrics ‣ 3 AgentQuest Overview ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks") provides an overview of the benchmarks and their respective milestones used to measure progress.

We emphasise that this evaluation is not aimed at providing a thorough evaluation and comparison of agent architectures, but rather to show how to use AgentQuest and how monitoring progress and action repetition can provide relevant insights to developers, even after a few executions.

#### Experimental setup.

We use as reference architecture the off-the-shelf chat agent provided by LangChain Chase ([2022](https://arxiv.org/html/2404.06411v1#bib.bib2)) powered by GPT-4 OpenAI ([2023b](https://arxiv.org/html/2404.06411v1#bib.bib11)) as LLM because it is intuitive, easy to extend and open source. We run 15 instances of the four benchmarks within AgentQuest, setting the maximum number of execution steps as 60⁵⁵5We limit the number of instances in our experiments for two main reasons: (i) the work primarily serves as a demonstration of the developed framework itself, rather than an extensive evaluation of the agent performance; (ii) extensive tests could have significantly impacted the ability to reproduce the experiments due to the expensive nature of API calls.. In Appendix [B](https://arxiv.org/html/2404.06411v1#A2 "Appendix B Appendix: Additional agents architectures and benchmarks ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks") we provide examples on how to use AgentQuest with two additional agent architectures and GAIA Mialon et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib9)) as open-ended environment.

Table 3: Average existing and proposed metrics for the tested benchmarks. We report the metrics, Success Rate (SR), Steps, Progress Rate at step 60 (PR${}_{60}$) and Repetition Rate at final step 60 (RR${}_{60}$). We denote with ${}^{*}$ the improved results after modifying the agent architecture.

|  | Existing Metrics | AgentQuest |
|  | SR | Steps | PR${}_{\mathbf{{60}}}$ | RR${}_{\mathbf{{60}}}$ |
| Mastermind | 0.47 | 41.87 | 0.62 | 0.32 |
| LTP | 0.20 | 52.00 | 0.46 | 0.81 |
| ALFWorld | 0.86 | 21.00 | 0.74 | 0.06 |
| Sudoku | 0.00 | 59.67 | 0.08 | 0.22 |
| Mastermind${}^{*}$ | 0.60 | 39.73 | 0.73 | 0.00 |
| ALFWorld${}^{*}$ | 0.93 | 25.86 | 0.80${}^{\dagger}$ | 0.07${}^{\dagger}$ |
|  

&#124; ${}^{\dagger}$Metrics referred to the extended runtime up to 120 &#124;
&#124; steps, hence PR${}_{120}$ and RR${}_{120}$. &#124;

 |

#### Experimental results.

For Mastermind, [Figure 1(a)](https://arxiv.org/html/2404.06411v1#S4.F1.sf1 "1(a) ‣ Figure 2 ‣ Experimental results. ‣ 4 Insights via AgentQuest ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks") shows the progress rate PR${}_{t}$ and repetition rate RR${}_{t}$. In the first 22 steps, the agent explores different solutions (RR${}_{[0,22]}<5\%$). This leads to growing progress towards the final goal, reaching half of the milestones (PR${}_{22}\approx 55\%$). Then, the agent starts performing the same actions, exhibiting a repetitive pattern (see also [Figure 2(a)](https://arxiv.org/html/2404.06411v1#S4.F2.sf1 "2(a) ‣ Figure 3 ‣ Experimental results. ‣ 4 Insights via AgentQuest ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks") rightmost part) and failing to reach the final goal within the next 38 steps. This results in a rise of the repetitions to RR${}_{60}=30\%$ and a saturation of the progress rate at PR${}_{60}=55\%$. Hence, AgentQuest offered us a crucial insights on why the current agent cannot solve the Mastermind game.

To overcome this agent limitation we incorporate a memory component Park et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib13)) into the agent architecture. The agent stores the past guesses in a local buffer. Then, at each step, if the agent outputs an action already in the buffer, it is prompted to provide a new one. [Table 3](https://arxiv.org/html/2404.06411v1#S4.T3 "Table 3 ‣ Experimental setup. ‣ 4 Insights via AgentQuest ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks") (Mastermind${}^{*}$) shows that this simple change in agent architecture has a big impact: the agent can now solve more instances, increasing the final SR from 47% to 60% and preventing repetitions (RR${}_{60}=0\%$). This highlights how studying the interplay between progress and repetition rates can allow us to improve agent architecture, sometimes even with simple remedies. We support our intuition extending the evaluation to more instances of Mastermind from 15 to 60 achieving comparable results – i.e. 43% of SR with the standard architecture and 62% with the simple memory (19% of improvement).

![Refer to caption](img/9126ab7571bd637520e6706b4ec2702f.png)

(a) Mastermind

![Refer to caption](img/bd85fbd2f263a7db12f1c6032f7bbe10.png)

(b) LTP

Figure 2: Average Progress rate PR${}_{t}$ and the repetition rate RR${}_{t}$ on Mastermind and LTP. Mastermind: It starts out with a low RR${}_{t}$ but this increases after step 22 while the progress rate also stall at 55%. LTP: at first a higher RR${}_{t}$ allows the agent to progress by making small variations that lead to success, but later this plateaus.

![Refer to caption](img/eb3e0c214e27d16b015288f8fe45534f.png)

(a) Mastermind

![Refer to caption](img/882748339fcfc0747b7ae6b22e8aaf7c.png)

(b) LTP

Figure 3: Examples of repeated actions in Mastermind and LTP. Mastermind: there is a set of unique actions at first, but then gets stuck repeating the same actions over and over. LTP: repeated actions are small variations of the same question that lead to progress.

For LTP, the AgentQuest metrics reveal a different agent behaviour, where repetitions are part of the agent reasoning strategy, enhancing the progress rate ([Figure 1(b)](https://arxiv.org/html/2404.06411v1#S4.F1.sf2 "1(b) ‣ Figure 2 ‣ Experimental results. ‣ 4 Insights via AgentQuest ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks")). From the initial steps, the agent changes aspects of the same questions until a local solution emerges. This leads to horizontal indicators in [Figure 2(b)](https://arxiv.org/html/2404.06411v1#S4.F2.sf2 "2(b) ‣ Figure 3 ‣ Experimental results. ‣ 4 Insights via AgentQuest ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks") and RR${}_{20}\approx 30\%.$ Despite solving only a few riddles (SR=0.2), these repetitions contribute to progress, achieving 46% of the milestones by the end of the execution, with a final repetition rate of RR${}_{60}=81\%$. This shows us how the interplay of progress and repetition rates provides an insight on how agents behave across the different time steps.

Consider the benchmark ALFWorld in [Table 3](https://arxiv.org/html/2404.06411v1#S4.T3 "Table 3 ‣ Experimental setup. ‣ 4 Insights via AgentQuest ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks") (we report the metrics trend in Appendix [A](https://arxiv.org/html/2404.06411v1#A1 "Appendix A Appendix: ALFWorld and Sudoku benchmarks ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks")). It requires the exploration of a textual world to locate an object. While the agent explores the solution space and limits action repetitions (RR${}_{60}=6\%$), it fails to solve all the games (PR${}_{60}=74\%$). This discrepancy may arise from the more exploration steps required to discover the object. We support this intuition extending the benchmark runtime to 120 steps resulting in a success and progress rates increase by 6% (ALFWorld${}^{*}$ in [Table 3](https://arxiv.org/html/2404.06411v1#S4.T3 "Table 3 ‣ Experimental setup. ‣ 4 Insights via AgentQuest ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks")). This confirms the usefulness of AgentQuest in understanding the agent failures. We support our intuition also extending the evaluation to more instances of ALFWorld from 15 to 60 achieving comparable results – i.e. 83% of SR with 60 steps as limit and 87% with 120 steps as limit (4% of improvement).

Finally, we look at Sudoku, known for its high level of difficulty Felgenhauer and Jarvis ([2006](https://arxiv.org/html/2404.06411v1#bib.bib3)). The low progress and repetition rates achieved after 60 steps (PR${}_{60}=8\%$ and RR${}_{60}=22\%$) indicate that the current agent architecture struggles in finding correct solutions solving this task. We report the metrics trend in Appendix [A](https://arxiv.org/html/2404.06411v1#A1 "Appendix A Appendix: ALFWorld and Sudoku benchmarks ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks").

## 5 Conclusions

AgentQuest allows the research community to keep track of agent progress in a holistic manner. Starting out with a first set of four benchmarks and two new metrics, AgentQuest is easily extendable. Furthermore, the two proposed metrics, progress and repetition rates, have the great advantage of allowing to track how agents advance toward the final goal over time. Especially studying their interplay can lead to important insights that will allow the research community to improve agent performance. Finally, we believe that promptly sharing AgentQuest with the research community will facilitate benchmarking and debugging agents, and will foster the creation and use of new benchmarks and metrics.

## Ethical Considerations

The complexity of LLM agents poses challenges in comprehending their decision-making processes. Ethical guidelines must demand transparency in such systems, ensuring that developers and end-users comprehend how decisions are reached.

We are not aware of any direct ethical impact generated by our work. However, we hope that insights into Generative AI agents’ decision-making processes will be applied to improve and promote transparency and fairness.

## Acknowledgements

This project has received funding from the European Union’s Horizon Europe research and innovation programme (SNS-JU) under the Grant Agreement No 101139285 (“NATWORK”).

## References

*   Chalamalasetti et al. (2023) Kranti Chalamalasetti, Jana Götze, Sherzod Hakimov, Brielen Madureira, Philipp Sadler, and David Schlangen. 2023. [Clembench: Using Game Play to Evaluate Chat-Optimized Language Models as Conversational Agents](http://arxiv.org/abs/2305.13455).
*   Chase (2022) Harrison Chase. 2022. [LangChain - Building applications with LLMs through composability](https://github.com/langchain-ai/langchain).
*   Felgenhauer and Jarvis (2006) Bertram Felgenhauer and Frazer Jarvis. 2006. Mathematics of Sudoku I. *Mathematical Spectrum*.
*   Hessel et al. (2022) Jack Hessel, Ari Holtzman, Maxwell Forbes, Ronan Le Bras, and Yejin Choi. 2022. [CLIPScore: A Reference-free Evaluation Metric for Image Captioning](http://arxiv.org/abs/2104.08718).
*   Kiela et al. (2023) Douwe Kiela, Tristan Thrush, Kawin Ethayarajh, and Amanpreet Singh. 2023. [Plotting Progress in AI](https://contextual.ai/plotting-progress-in-ai/). *Contextual AI Blog*.
*   Levenshtein (1966) Vladimir I. Levenshtein. 1966. Binary codes capable of correcting deletions, insertions, and reversals. In *Soviet Physics Doklady*.
*   Lin (2004) Chin-Yew Lin. 2004. [ROUGE: A Package for Automatic Evaluation of Summaries](https://aclanthology.org/W04-1013).
*   Liu et al. (2023) Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang, et al. 2023. [AgentBench: Evaluating LLMs as Agents](http://arxiv.org/abs/2308.03688).
*   Mialon et al. (2023) Grégoire Mialon, Clémentine Fourrier, Craig Swift, Thomas Wolf, Yann LeCun, and Thomas Scialom. 2023. [GAIA: a benchmark for General AI Assistants](http://arxiv.org/abs/2311.12983).
*   OpenAI (2023a) OpenAI. 2023a. [Assistants API](https://platform.openai.com/docs/assistants/overview).
*   OpenAI (2023b) OpenAI. 2023b. [GPT-4 Technical Report](http://arxiv.org/abs/2303.08774).
*   Papineni et al. (2002) Kishore Papineni, Salim Roukos, Todd Ward, and Wei-Jing Zhu. 2002. [Bleu: a Method for Automatic Evaluation of Machine Translation](https://doi.org/10.3115/1073083.1073135).
*   Park et al. (2023) Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang, and Michael S Bernstein. 2023. [Generative Agents: Interactive Simulacra of Human Behavior](https://doi.org/10.1145/3586183.3606763).
*   Patil et al. (2023) Shishir G. Patil, Tianjun Zhang, Xin Wang, and Joseph E. Gonzalez. 2023. [Gorilla: Large Language Model Connected with Massive APIs](http://arxiv.org/abs/2305.15334).
*   Shridhar et al. (2020) Mohit Shridhar, Xingdi Yuan, Marc-Alexandre Côté, Yonatan Bisk, Adam Trischler, and Matthew Hausknecht. 2020. [ALFWorld: Aligning Text and Embodied Environments for Interactive Learning](http://arxiv.org/abs/2010.03768).
*   Sloane (1992) Paul Sloane. 1992. *Lateral Thinking Puzzlers*. Sterling Publishing Company, Inc.
*   Stuckman and Zhang (2005) Jeff Stuckman and Guo-Qiang Zhang. 2005. [Mastermind is NP-complete](http://arxiv.org/abs/cs/0512049).
*   Sutton and Barto (2018) Richard S Sutton and Andrew G Barto. 2018. *Reinforcement Learning: An Introduction*. MIT press.
*   Wang et al. (2023) Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin, et al. 2023. [A Survey on Large Language Model based Autonomous Agents](http://arxiv.org/abs/2308.11432).
*   Weng (2023) Lilian Weng. 2023. [LLM-powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/).
*   Yao et al. (2022) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. 2022. [ReAct: Synergizing Reasoning and Acting in Language Models](http://arxiv.org/abs/2210.03629).
*   Zhang et al. (2020a) Tianyi Zhang, Varsha Kishore, Felix Wu, Kilian Q. Weinberger, and Yoav Artzi. 2020a. [BERTScore: Evaluating Text Generation with BERT](http://arxiv.org/abs/1904.09675).
*   Zhang et al. (2020b) Tianyi Zhang, Varsha Kishore, Felix Wu, Kilian Q. Weinberger, and Yoav Artzi. 2020b. [BERTScore: Evaluating Text Generation with BERT](https://openreview.net/forum?id=SkeHuCVFDr).
*   Zheng et al. (2023) Lianmin Zheng, Wei-Lin Chiang, Ying Sheng, Siyuan Zhuang, Zhanghao Wu, Yonghao Zhuang, Zi Lin, Zhuohan Li, Dacheng Li, Eric Xing, et al. 2023. [Judging LLM-as-a-judge with MT-Bench and Chatbot Arena](http://arxiv.org/abs/2306.05685).
*   Zhuang et al. (2023) Yuchen Zhuang, Yue Yu, Kuan Wang, Haotian Sun, and Chao Zhang. 2023. [ToolQA: A Dataset for LLM Question Answering with External Tools](http://arxiv.org/abs/2306.13304).

## Appendix A Appendix: ALFWorld and Sudoku benchmarks

In this section we report the detailed metrics for each step for the ALFWorld and Sudoku benchmarks, omitted for the sake of brevity from the main paper.

![Refer to caption](img/40ea2d9ca0914894be58198cfabb73ec.png)

(a) ALFWorld

![Refer to caption](img/ffbbbd33e822fedfb2aa37547a3f3ec9.png)

(b) Sudoku

Figure 4: Progress rate PR${}_{t}$ and the repetition rate RR${}_{t}$ on ALFWorld and Sudoku averaged over 15 runs. ALFWorld: It starts out with a low repetition rate and quick increase of the progress rate. Then a slow increase of the repetition rate enables to further increase the progress rate although less quickly. Sudoku: The progress rate quickly reaches 8%. The repetition rate then slowly increases without any positive change in the progress rate.

Figure [3(a)](https://arxiv.org/html/2404.06411v1#A1.F3.sf1 "3(a) ‣ Figure 4 ‣ Appendix A Appendix: ALFWorld and Sudoku benchmarks ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks") reports the progress rate and repetition rate for ALFWorld. The repetition rate is close to 0% for the first 20 steps, then it slowly increases up to 6% after 60 steps. The progress rate quickly reaches over 50% in 10 steps, then keeps increasing, although slowly, up to 74%. The consistent improvement of the progress rate even for steps close to 60 together with the low repetition rate suggests that higher values may be reached by increasing the maximum number of steps. We validate this hypothesis by extending the benchmark runtime to 120 steps. As previously reported in Table [3](https://arxiv.org/html/2404.06411v1#S4.T3 "Table 3 ‣ Experimental setup. ‣ 4 Insights via AgentQuest ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks"), this results in an improvement of 6 percentage points for both the success rate the progress rate, i.e. SR$=93\%$ and PR${}_{120}=80\%$.

Figure [3(b)](https://arxiv.org/html/2404.06411v1#A1.F3.sf2 "3(b) ‣ Figure 4 ‣ Appendix A Appendix: ALFWorld and Sudoku benchmarks ‣ AgentQuest: Benchmarking LLM Agents Behaviours in Multi-step Intensive Reasoning Tasks") includes the two metrics for the Sudoku benchmark. We can observe that the progress rate quickly reaches a plateau at 8% in very few steps. The repetition rate is close to 0% for the first 10 steps, then it slowly increases up to 22% after 60 steps without any improvement of the progress rate.

## Appendix B Appendix: Additional agents architectures and benchmarks

In this section we highlight the plug-and-play aspect of AgentQuest showing the implementation of Mastermind with two additional agents architectures, i.e. ReAct Yao et al. ([2022](https://arxiv.org/html/2404.06411v1#bib.bib21)) as the most used architecture in literature and OpenAI Assistant OpenAI ([2023a](https://arxiv.org/html/2404.06411v1#bib.bib10)), as the most recent proprietary architecture. Additionally, we show how to implement the open-ended benchmark GAIA Mialon et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib9)) requiring the usage of external tools. For brevity, in the following snippets we omit details, like error handling or full agent definition. The complete code is available in the [GitHub repository](https://github.com/nec-research/agentquest).

### B.1 ReAct for Closed-box Environments

We show an example of how to execute a closed-box benchmark (i.e. ALFWorld) with an agent based on the ReAct architecture Yao et al. ([2022](https://arxiv.org/html/2404.06411v1#bib.bib21)). Such architecture forces the agent decision making process to generate both textual reasoning traces and actions pertaining to a task in an interleaved manner. Common implementations Chase ([2022](https://arxiv.org/html/2404.06411v1#bib.bib2)); Yao et al. ([2022](https://arxiv.org/html/2404.06411v1#bib.bib21)) rely on external tools to perform actions. Here, we ensure compatibility with existing implementations providing a single tool (i.e. ProxyTool) that forwards the actions to the driver. In a nutshell, the agent reflects on the action to take and invokes the tool. Then, we feed the tool input to the driver to perform the interaction with the environment. At each step, we provide the agent the updated history of the actions and observations through the intermediate_steps variable.

[⬇](data:text/plain;base64,ZnJvbSBhZ2VudHF1ZXN0LmRyaXZlcnMgaW1wb3J0IE1hc3Rlck1pbmREcml2ZXIKZnJvbSBhZ2VudHF1ZXN0Lm1ldHJpY3MgaW1wb3J0IC4uLgpmcm9tIGFnZW50cXVlc3QudXRpbHMgaW1wb3J0IEFjdGlvbgouLi4KCiMgRGVmaW5lIGEgZHVtbXkgdG9vbCBmb3IgY2xvc2VkLWJveCBlbnZpcm9ubWVudHMKY2xhc3MgUHJveHlUb29sKEJhc2VUb29sKToKICAgIG5hbWUgPSAicHJveHl0b29sIgogICAgZGVzY3JpcHRpb24gPSAiUHJvdmlkZSB0aGUgYWN0aW9uIHlvdSB3YW50IHRvIHBlcmZvcm0iCiAgICBkZWYgX3J1bihzZWxmKToKICAgICAgICBwYXNzCgojIEluc3RhbnRpYXRlIGN1c3RvbSBwcm9tcHQKcHJvbXB0ID0gQ3VzdG9tUHJvbXB0VGVtcGxhdGUoCiAgICB0ZW1wbGF0ZT0uLi4sICMgTExNIHByb21wdAogICAgdG9vbHM9W1Byb3h5VG9vbCgpXSwKICAgIGlucHV0X3ZhcmlhYmxlcz1bImludGVybWVkaWF0ZV9zdGVwcyIsIC4uLl0KKQojIEluaXRpYWxpc2UgdGhlIGFnZW50CmFnZW50ID0gY3JlYXRlX3JlYWN0X2FnZW50KGxsbSwgW1Byb3h5VG9vbCgpXSwgcHJvbXB0KQppbnRlcm1lZGlhdGVfc3RlcHMgPSBbXQojIEluaXRpYWxpc2UgdGhlIGRyaXZlcgpkcml2ZXIgPSBNYXN0ZXJNaW5kRHJpdmVyKGdhbWUpCiMgR2V0IHRoZSBmaXJzdCBvYnNlcnZhdGlvbgpvYnMgPSBkcml2ZXIucmVzZXQoKQojIEFnZW50IExvb3AKd2hpbGUgbm90IG9icy5kb25lOgogICAgIyBSZXRyaWV2ZSB0aGUgYWdlbnQgb3V0cHV0CiAgICBhZ2VudF9jaG9pY2UgPSBhZ2VudC5pbnZva2UoCiAgICAgICAgeydpbnB1dCc6b2JzLm91dHB1dCwKICAgICAgICAgJ2ludGVybWVkaWF0ZV9zdGVwcyc6aW50ZXJtZWRpYXRlX3N0ZXBzfQogICAgKQogICAgYWN0aW9uID0gQWN0aW9uKGFjdGlvbl92YWx1ZT1hZ2VudF9jaG9pY2UudG9vbF9pbnB1dCkKICAgICMgUGVyZm9ybSB0aGUgc3RlcAogICAgb2JzID0gZHJpdmVyLnN0ZXAoYWN0aW9uKQogICAgIyBVcGRhdGUgaW50ZXJtZWRpYXRlIHN0ZXBzCiAgICBpbnRlcm1lZGlhdGVfc3RlcHMuYXBwZW5kKChhZ2VudF9jaG9pY2UsIG9icy5vdXRwdXQpKQogICAgIyBHZXQgY3VycmVudCBtZXRyaWNzIC4uLg==)from agentquest.drivers import MasterMindDriverfrom agentquest.metrics import ...from agentquest.utils import Action...# Define a dummy tool for closed-box environmentsclass ProxyTool(BaseTool):    name = "proxytool"    description = "Provide the action you want to perform"    def _run(self):        pass# Instantiate custom promptprompt = CustomPromptTemplate(    template=..., # LLM prompt    tools=[ProxyTool()],    input_variables=["intermediate_steps", ...])# Initialise the agentagent = create_react_agent(llm, [ProxyTool()], prompt)intermediate_steps = []# Initialise the driverdriver = MasterMindDriver(game)# Get the first observationobs = driver.reset()# Agent Loopwhile not obs.done:    # Retrieve the agent output    agent_choice = agent.invoke(        {’input’:obs.output,         ’intermediate_steps’:intermediate_steps}    )    action = Action(action_value=agent_choice.tool_input)    # Perform the step    obs = driver.step(action)    # Update intermediate steps    intermediate_steps.append((agent_choice, obs.output))    # Get current metrics ...

### B.2 OpenAI Assistant for Closed-box Environments

The OpenAI Assistant OpenAI ([2023a](https://arxiv.org/html/2404.06411v1#bib.bib10)) is a proprietary architecture. It allows users to define custom agents by specifying the tasks to accomplish and the set of tools the agent can use. While the decision-making process is not directly accessible by the end-users (the agent and the LLM are hosted on the proprietary cloud environment), the tools can be invoked both remotely or locally. In the latter, users have control on the tool invocation managing the agent loop.

Similarly to ReAct, we here rely on the ProxyTool, acting as a proxy between the agent and the environment. We invoke the remote agent with the initial task (e.g. first ALFWorld observation) and process the output of its decision making process, i.e. the action to perform provided as tool input. Then, we bypass the tool invocation, directly forwarding the action to the driver to perform the execution step and retrieve the next observation. Finally, we invoke the agent with the new observation concluding the execution step.

[⬇](data:text/plain;base64,ZnJvbSBhZ2VudHF1ZXN0LmRyaXZlcnMgaW1wb3J0IE1hc3Rlck1pbmREcml2ZXIKZnJvbSBhZ2VudHF1ZXN0Lm1ldHJpY3MgaW1wb3J0IC4uLgpmcm9tIGFnZW50cXVlc3QudXRpbHMgaW1wb3J0IEFjdGlvbgouLi4KCiMgRGVmaW5lIGEgZHVtbXkgdG9vbCBmb3IgY2xvc2VkLWJveCBlbnZpcm9ubWVudHMKY2xhc3MgUHJveHlUb29sKEJhc2VUb29sKToKICAgIG5hbWUgPSAicHJveHl0b29sIgogICAgZGVzY3JpcHRpb24gPSAiUHJvdmlkZSB0aGUgYWN0aW9uIHlvdSB3YW50IHRvIHBlcmZvcm0iCiAgICBkZWYgX3J1bihzZWxmKToKICAgICAgICBwYXNzCgojIEluaXRpYWxpc2UgdGhlIGFnZW50CmFnZW50ID0gT3BlbkFJQXNzaXN0YW50UnVubmFibGUuY3JlYXRlX2Fzc2lzdGFudCgKICAgIGluc3RydWN0aW9ucz0uLi4gIyBMTE0gcHJvbXB0CiAgICB0b29scz1bUHJveHlUb29sKCldLAogICAgbW9kZWw9Li4uICMgQ2hvc2VuIExMTQogICAgYXNfYWdlbnQ9VHJ1ZQopCiMgSW5pdGlhbGlzZSB0aGUgZHJpdmVyCmRyaXZlciA9IE1hc3Rlck1pbmREcml2ZXIoZ2FtZSkKIyBHZXQgdGhlIGZpcnN0IG9ic2VydmF0aW9uCm9icyA9IGRyaXZlci5yZXNldCgpCiMgR2V0IHRoZSBmaXJzdCBhY3Rpb24KcmVzcG9uc2UgPSBhZ2VudC5pbnZva2UoeyJjb250ZW50Ijogb2JzLm91dHB1dH0pCiMgQWdlbnQgTG9vcAp3aGlsZSBub3Qgb2JzLmRvbmU6CiAgICAjIFJldHJpZXZlIHRoZSBhZ2VudCBvdXRwdXQKICAgIGFnZW50X2d1ZXNzID0gcmVzcG9uc2VbMF0udG9vbF9pbnB1dAogICAgYWN0aW9uID0gQWN0aW9uKGFjdGlvbl92YWx1ZT1hZ2VudF9ndWVzcykKICAgICMgUGVyZm9ybSB0aGUgc3RlcAogICAgb2JzID0gZHJpdmVyLnN0ZXAoYWN0aW9uKQogICAgIyBHZXQgY3VycmVudCBtZXRyaWNzIC4uLgogICAgIyBNYW5hZ2UgUHJveHkgVG9vbCBvdXRwdXQKICAgIHRvb2xfb3V0cHV0cyA9IFsKICAgICAgICB7Im91dHB1dCI6IG9icy5vdXRwdXQsCiAgICAgICAgICJ0b29sX2NhbGxfaWQiOiByZXNwb25zZVswXS50b29sX2NhbGxfaWR9CiAgICBdCiAgICAjIEludm9rZSB0aGUgYWdlbnQgdG8gZ2V0IHRoZSBuZXh0IGFjdGlvbgogICAgcmVzcG9uc2UgPSBhZ2VudC5pbnZva2UoCiAgICAgICAgeyJ0b29sX291dHB1dHMiOiB0b29sX291dHB1dHMsCiAgICAgICAgICJydW5faWQiOiByZXNwb25zZVswXS5ydW5faWQsCiAgICAgICAgICJ0aHJlYWRfaWQiOiByZXNwb25zZVswXS50aHJlYWRfaWR9CiAgICAp)from agentquest.drivers import MasterMindDriverfrom agentquest.metrics import ...from agentquest.utils import Action...# Define a dummy tool for closed-box environmentsclass ProxyTool(BaseTool):    name = "proxytool"    description = "Provide the action you want to perform"    def _run(self):        pass# Initialise the agentagent = OpenAIAssistantRunnable.create_assistant(    instructions=... # LLM prompt    tools=[ProxyTool()],    model=... # Chosen LLM    as_agent=True)# Initialise the driverdriver = MasterMindDriver(game)# Get the first observationobs = driver.reset()# Get the first actionresponse = agent.invoke({"content": obs.output})# Agent Loopwhile not obs.done:    # Retrieve the agent output    agent_guess = response[0].tool_input    action = Action(action_value=agent_guess)    # Perform the step    obs = driver.step(action)    # Get current metrics ...    # Manage Proxy Tool output    tool_outputs = [        {"output": obs.output,         "tool_call_id": response[0].tool_call_id}    ]    # Invoke the agent to get the next action    response = agent.invoke(        {"tool_outputs": tool_outputs,         "run_id": response[0].run_id,         "thread_id": response[0].thread_id}    )

### B.3 OpenAI Assistant for Open-ended Environments

When interacting with an open-ended environment, the agent is not restricted to the pre-defined actions of the closed-box environment and it is allowed to select any user-defined tool (e.g. retrieving information online or executing code). Hence, we provide the agent the list of tools via the tool variable. The agent relies on its reasoning process to choose which tool to invoke. Omitted here for the sake of brevity, we rely of the manual annotations of the GAIA questions Mialon et al. ([2023](https://arxiv.org/html/2404.06411v1#bib.bib9)) as milestones to compute the progress rate.

[⬇](data:text/plain;base64,ZnJvbSBhZ2VudHF1ZXN0LmRyaXZlcnMgaW1wb3J0IEdhaWFEcml2ZXIKZnJvbSBhZ2VudHF1ZXN0Lm1ldHJpY3MgaW1wb3J0IC4uLgpmcm9tIGFnZW50cXVlc3QudXRpbHMgaW1wb3J0IEFjdGlvbgouLi4KCiMgRGVmaW5lIHRoZSB0b29scwp0b29scz1bCiAgICBPbmxpbmVTZWFyY2goKSwgIyBSZXRyaWV2ZSBhIHdlYiBwYWdlIGxpbmsKICAgIFdlYkNvbnRlbnRQYXJzZXIoKSwgIyBSZWFkIHRoZSB3ZWIgcGFnZQogICAgRmluYWxBbnN3ZXJSZXRyaWV2ZXIoKSwgIyBQcm92aWRlIHRoZSBmaW5hbCBhbnN3ZXIKICAgIC4uLgpdCiMgSW5pdGlhbGlzZSB0aGUgYWdlbnQKYWdlbnQgPSBPcGVuQUlBc3Npc3RhbnRSdW5uYWJsZS5jcmVhdGVfYXNzaXN0YW50KAogICAgaW5zdHJ1Y3Rpb25zPS4uLiAjIExMTSBwcm9tcHQKICAgIHRvb2xzPXRvb2xzLAogICAgbW9kZWw9Li4uICMgQ2hvc2VuIExMTQogICAgYXNfYWdlbnQ9VHJ1ZQopCiMgSW5pdGlhbGlzZSB0aGUgZHJpdmVyCmRyaXZlciA9IEdhaWFEcml2ZXIocXVlc3Rpb24sIHRvb2xzKQojIEdldCB0aGUgZmlyc3Qgb2JzZXJ2YXRpb24Kb2JzID0gZHJpdmVyLnJlc2V0KCkKIyBHZXQgdGhlIGZpcnN0IGFjdGlvbgpyZXNwb25zZSA9IGFnZW50Lmludm9rZSh7ImNvbnRlbnQiOiBvYnMub3V0cHV0fSkKIyBBZ2VudCBMb29wCndoaWxlIG5vdCBvYnMuZG9uZToKICAgICMgUmV0cmlldmUgdGhlIGFnZW50IG91dHB1dAogICAgYWN0ID0gZid7cmVzcG9uc2VbMF0udG9vbH06e3Jlc3BvbnNlWzBdLnRvb2xfaW5wdXR9JwogICAgYWN0aW9uID0gQWN0aW9uKGFjdGlvbl92YWx1ZT1hY3QpCiAgICAjIFBlcmZvcm0gdGhlIHN0ZXAgaW52b2tpbmcgdGhlIGxvY2FsIHRvb2wKICAgIG9icyA9IGRyaXZlci5zdGVwKGFjdGlvbikKICAgICMgR2V0IGN1cnJlbnQgbWV0cmljcyAuLi4KICAgICMgTWFuYWdlIHRvb2wgb3V0cHV0IGFzIG9ic2VydmF0aW9uCiAgICB0b29sX291dHB1dHMgPSBbCiAgICAgICAgeyJvdXRwdXQiOiBvYnMub3V0cHV0LAogICAgICAgICAidG9vbF9jYWxsX2lkIjogcmVzcG9uc2VbMF0udG9vbF9jYWxsX2lkfQogICAgXQogICAgIyBJbnZva2UgdGhlIGFnZW50IHRvIGdldCB0aGUgbmV4dCBhY3Rpb24KICAgIHJlc3BvbnNlID0gYWdlbnQuaW52b2tlKAogICAgICAgIHsidG9vbF9vdXRwdXRzIjogdG9vbF9vdXRwdXRzLAogICAgICAgICAicnVuX2lkIjogcmVzcG9uc2VbMF0ucnVuX2lkLAogICAgICAgICAidGhyZWFkX2lkIjogcmVzcG9uc2VbMF0udGhyZWFkX2lkfQogICAgKQ==)from agentquest.drivers import GaiaDriverfrom agentquest.metrics import ...from agentquest.utils import Action...# Define the toolstools=[    OnlineSearch(), # Retrieve a web page link    WebContentParser(), # Read the web page    FinalAnswerRetriever(), # Provide the final answer    ...]# Initialise the agentagent = OpenAIAssistantRunnable.create_assistant(    instructions=... # LLM prompt    tools=tools,    model=... # Chosen LLM    as_agent=True)# Initialise the driverdriver = GaiaDriver(question, tools)# Get the first observationobs = driver.reset()# Get the first actionresponse = agent.invoke({"content": obs.output})# Agent Loopwhile not obs.done:    # Retrieve the agent output    act = f’{response[0].tool}:{response[0].tool_input}’    action = Action(action_value=act)    # Perform the step invoking the local tool    obs = driver.step(action)    # Get current metrics ...    # Manage tool output as observation    tool_outputs = [        {"output": obs.output,         "tool_call_id": response[0].tool_call_id}    ]    # Invoke the agent to get the next action    response = agent.invoke(        {"tool_outputs": tool_outputs,         "run_id": response[0].run_id,         "thread_id": response[0].thread_id}    )

Here, the driver acts as a wrapper, executing the tool with the parameters provided by the agent (tool_input) and forwards the output to the agent in the correct format:

[⬇](data:text/plain;base64,Y2xhc3MgR2FpYURyaXZlcigpOgogICAgZGVmIF9faW5pdF9fKHNlbGYsIHF1ZXN0aW9uLCB0b29scywgLi4uKToKICAgICAgICAjIEluaXRpYWxpc2UgdGhlIHRvb2wgbG9va3VwCiAgICAgICAgc2VsZi50b29sX2xvb2t1cCA9IHt4Lm5hbWU6eCBmb3IgeCBpbiB0b29sc30KICAgIC4uLgogICAgZGVmIHN0ZXAoc2VsZiwgYWN0aW9uKToKICAgICAgICAjIFBhcnNlIHRoZSBhY3Rpb24KICAgICAgICB0b29sLCB0b29sX2lucHV0ID0gYWN0aW9uLmFjdGlvbl92YWx1ZS5zcGxpdCgnOicpCiAgICAgICAgIyBJbnZva2UgdGhlIHRvb2wKICAgICAgICB0b29sX291dCA9IHNlbGYudG9vbF9sb29rdXBbdG9vbF0uX3J1bih0b29sX2lucHV0KQogICAgICAgICMgUGFyc2UgdGhlIHRvb2wgb3V0cHV0IGhlcmUgLi4uCiAgICAgICAgcmV0dXJuIE9ic2VydmF0aW9uKG91dHB1dD10b29sX291dCk=)class GaiaDriver():    def __init__(self, question, tools, ...):        # Initialise the tool lookup        self.tool_lookup = {x.name:x for x in tools}    ...    def step(self, action):        # Parse the action        tool, tool_input = action.action_value.split(’:’)        # Invoke the tool        tool_out = self.tool_lookup[tool]._run(tool_input)        # Parse the tool output here ...        return Observation(output=tool_out)