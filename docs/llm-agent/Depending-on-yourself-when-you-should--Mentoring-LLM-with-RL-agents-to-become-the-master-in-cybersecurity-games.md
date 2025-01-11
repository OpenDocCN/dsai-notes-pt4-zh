<!--yml
category: 未分类
date: 2025-01-11 12:44:19
-->

# Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games

> 来源：[https://arxiv.org/html/2403.17674/](https://arxiv.org/html/2403.17674/)

Yikuan Yan¹ These authors contributed equally.    Yaolun Zhang¹¹¹footnotemark: 1    Keman Huang^(1,2)
¹School of Information, Renmin University of China, Beijing, China
²Cybersecurity at MIT Sloan, MIT, Cambridge, Massachusetts, USA
{yanyikuan, zhangyaolun5, keman}@ruc.edu.cn Corresponding author.

###### Abstract

Integrating LLM and reinforcement learning (RL) agent effectively to achieve complementary performance is critical in high stake tasks like cybersecurity operations. In this study, we introduce SecurityBot, a LLM agent mentored by pre-trained RL agents, to support cybersecurity operations. In particularly, the LLM agent is supported with a profile module to generated behavior guidelines, a memory module to accumulate local experiences, a reflection module to re-evaluate choices, and an action module to reduce action space. Additionally, it adopts the collaboration mechanism to take suggestions from pre-trained RL agents, including a cursor for dynamic suggestion taken, an aggregator for multiple mentors’ suggestions ranking and a caller for proactive suggestion asking. Building on the CybORG experiment framework, our experiences show that SecurityBot demonstrates significant performance improvement compared with LLM or RL standalone, achieving the complementary performance in the cybersecurity games.

## 1 Introduction

Cybersecurity operations involve the participation of various entities such as attackers and defenders. With the advancement of artificial intelligence (AI), autonomous cyber operation (ACO) agents have emerged as a promising solution in cybersecurity operations Vyas et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib36)). These agents continually engage in adversarial learning within network environments, enhancing their strategic capabilities. The recent proliferation of large language models (LLMs) has significantly bolstered the capabilities of autonomous agents Wang et al. ([2023a](https://arxiv.org/html/2403.17674v1#bib.bib38)). In comparison to traditional machine learning agents, LLM agents possess extensive knowledge, enabling them to handle richer and more complex information, coupled with robust contextual and reasoning abilities Lin et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib18)); Wang et al. ([2023b](https://arxiv.org/html/2403.17674v1#bib.bib39), [c](https://arxiv.org/html/2403.17674v1#bib.bib40)). They not only surpass state-of-the-art methods as novel tools Xia et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib43)) but also exhibit formidable interactive capabilities as assistants or agents Sandoval et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib30)).

However, LLM agents lack the specific knowledge of the local environment, incur higher training costs Hu et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib12)) and can stuck in hallucinations Ji et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib14)); Chen and Shu ([2023](https://arxiv.org/html/2403.17674v1#bib.bib5)), while also presenting attackers with powerful weapons, making them double-edge sword for cybersecurity Chen and Shu ([2023](https://arxiv.org/html/2403.17674v1#bib.bib5)); Taddeo et al. ([2019](https://arxiv.org/html/2403.17674v1#bib.bib33)). Recent research attempts to frame ACO as partially observable Markov processes (POMDP), employing reinforcement learning (RL) methods to train autonomous agents Standen et al. ([2021](https://arxiv.org/html/2403.17674v1#bib.bib32)); Team. ([2021](https://arxiv.org/html/2403.17674v1#bib.bib35)). However, without appropriate tuning methods, RL agents tend to converge to local optima, lacking robustness and generalization capabilities despite achieving favorable results Palmer et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib25)). As prior studies have demonstrated that collaborations among multiple agents can enhance team performance Dong et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib8)); Ma et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib19)), enabling the effective collaborations between LLM agents and RL agents, which can leverage the generalization knowledge of LLMs and the specialized knowledge of RLs in cybersecurity scenarios, can be promising to achieve complementary performance beyond that of individual agent.

Hence, we introduce the SecurityBot, a collaborative framework utilizing RL agents as mentors for LLM agent to support cybersecurity operations. We integrate four effective modules – profiles, memory, reflection and action – into the LLM. Simultaneously, we propose a dynamic mechanism consisting of a cursor to dynamically incorporate RL agents’ suggestions, an aggregator to rank suggestions from different RL agents, as well as a caller to proactively request mentoring from RL agents. We conduct experiments on the open-source ACO research platform, CybORG Standen et al. ([2021](https://arxiv.org/html/2403.17674v1#bib.bib32)), comparing the red team (attacker) task and blue team (defender) task performance among: (1)independently executing RL or LLM agents (Independent), (2) collaboration between a LLM agent and a RL agent (Single-Mentor), and (3) collaboration between a LLM agent and multiple RL agents (Multi-Mentors). Our experimental results demonstrate that the developed SecurityBot can effectively improve both the red team and blue team task performance compared to independent LLM or RL approaches. Furthermore, while mentoring from multiple RL agents can be beneficial, the guidance of poorly performing RL agents may be noise to, and result into unstable performance.

*   •

    We introduce SecurityBot, a mechanism to enable the effective collaboration between LLM and RL agents, to leverage RL agents as mentors to accelerate learning for LLM agents and achieve complementary performance.

*   •

    The collaboration of LLM and RL agents demonstrates performance improvement in both red team and blue team tasks, providing a promising solution of autonomous agents for cybersecurity operations.

## 2 Related Work

### 2.1 LLMs for cybersecurity operations

Given the rapid development of LLMs and the eager to incorporate advanced AIs into cybersecurity operations Iannone et al. ([2022](https://arxiv.org/html/2403.17674v1#bib.bib13)), recent studies have started to explore using LLMs to enhance cybersecurity while several evidences also reveal abusing LLMs to bring advanced threats, making it a double-edged sword Taddeo et al. ([2019](https://arxiv.org/html/2403.17674v1#bib.bib33)); Yao et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib46))

#### 2.1.1 LLM to enhance cybersecurity

LLMs demonstrate advantages in both code security and data security Noever ([2023](https://arxiv.org/html/2403.17674v1#bib.bib24)); Ali and Kostakos ([2023](https://arxiv.org/html/2403.17674v1#bib.bib1)); Qi et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib28)). For example, Fuzz4All Xia et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib43)) utilizes LLMs as input generators and mutation engines to generate diverse inputs for various programming languages, achieving an 36.8% coverage improvement compared to previous state-of-the-art techniques.

Additionally, compared to traditional machine learning approaches, LLMs possess more powerful natural language processing and contextual understanding capabilities, allowing them to elevate cybersecurity from specific to more macroscopic tasks. For example, some researchesDeng et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib7)); Pearce et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib27)) utilized these capabilities in specific security tasks to enhance effectiveness, while McIntosh et al.McIntosh et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib20)) take a further step to compared GPT-generated Governance, Risk, and Compliance (GRC) policies with those from established security vendors and government cybersecurity agencies, recommending GPT integration into companies’ GRC policy development.

#### 2.1.2 LLMs’ double-edged sword role for cybe security

However, applying LLMs to cybersecurity is a double-edged sword Taddeo et al. ([2019](https://arxiv.org/html/2403.17674v1#bib.bib33)): being generative in nature can lead to hallucinations—the generation of misleading or incorrect content, and can not effectively discern security-related fallacies, which can be catastrophic for high-stakes security tasks Ji et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib14)). These errors can compromise sensitive operations, thereby introducing substantial risks Chen and Shu ([2023](https://arxiv.org/html/2403.17674v1#bib.bib5)). As LLMs become more integrated into security frameworks, the imperative to address and mitigate these challenges grows ever more critical.

Furthermore, LLMs present attackers with powerful weapons. Recent studies have demonstrated that LLMs can significantly enhance attacks across hardware Yaman ([2023](https://arxiv.org/html/2403.17674v1#bib.bib45)), software and network Chen and Shu ([2023](https://arxiv.org/html/2403.17674v1#bib.bib5)) levels, especially that LLMs possess human-like reasoning capabilities, making user-level attacks even more severe Yao et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib46)); Falade ([2023](https://arxiv.org/html/2403.17674v1#bib.bib10)); Botacin ([2023](https://arxiv.org/html/2403.17674v1#bib.bib2)).

### 2.2 Collaboration mechanisms to improve LLMs

Recent studies have explored different mechanisms to support LLM’s collaborations with others, either LLM-based or RL-based agents, including:

#### 2.2.1 Role-based multi-LLM-agent collaboration

Within LLM-based multi-agent systems, LLM-based agents are assigned with different roles, like decomposing complex tasks, identifying errors, and collecting multiple perspectives. Then they collaborate with each other through a series of processes to resolve complex tasks such as software developments Dong et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib8)); Qian et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib29)); Hong et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib11)), sociological investigations Park et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib26)); Wang et al. ([2023b](https://arxiv.org/html/2403.17674v1#bib.bib39)); Zhang et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib47)), simulation of multiplayer games Sandoval et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib30)); Xu et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib44)) and various challenges (such as logical reasoning, stock advice, blog composing, and more) Li et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib17)); Wu et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib42)); Talebirad and Nadiri ([2023](https://arxiv.org/html/2403.17674v1#bib.bib34)). In particularly, different role-based agents exchange ideas through conversation, enforce tools to undertake tasks, garner feedback, leading to successful collaboration Wang et al. ([2023a](https://arxiv.org/html/2403.17674v1#bib.bib38)).

#### 2.2.2 Dual-process-based LLM-RL collaboration

The dual process theory highlights that human cognition consists of two mental systems where System 1 is autonomous and characterized by rapid intuition, while System 2 controls slow, deliberate thinking Wason and Evans ([1974](https://arxiv.org/html/2403.17674v1#bib.bib41)); Kahneman ([2011](https://arxiv.org/html/2403.17674v1#bib.bib15)). Grounded on this theory, SwiftSage introduces a framework that enables a small RL model, acting as the System 1 component, to collaborate with an LLM-Based agent, acting as the System 2 component. This structure effectively solve complex problems while reducing the cost of inference Lin et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib18)).

#### 2.2.3 LLM setting guidance to support RL

Some recent studies incorporate the LLM to generate or learn the reward function for RL agents, aiming at simplifying the reward function design process Ma et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib19)); Carta et al. ([2022](https://arxiv.org/html/2403.17674v1#bib.bib4)). For example, Micheli et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib21)); Kwon et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib16)); Du et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib9)) use LLM as a proxy reward function to guide RL agents in environments without clear reward signals. Additionally, Brohan et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib3)); Dasgupta et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib6)) utilize the LLM-Based agent as a planner to guide RL agent in complex and dynamic environments.

#### 2.2.4 RL acting as expert to guide LLM’s decision

LLM demonstrate powerful generalization abilities, but under specific scenario, they perform poorly due to the lack of expert trajectories. In contrast, RL models possess expert trajectories. Hence, Hu et al. ([2023](https://arxiv.org/html/2403.17674v1#bib.bib12)); Wan et al. ([2022](https://arxiv.org/html/2403.17674v1#bib.bib37)) use RL methods assist the LLM-Based agent in comprehending the environment, mastering expert-like actions, which results in better effects and lower interaction cost instructions.

Overall, LLMs has demonstrated promising potential in enhancing cybersecurity operations while their double-edged sword role raise specific concerns. Additionally, recent studies have explored different collaborations with LLMs but it is still in its early stage, especially for cybersecurity operations. Hence, using the cybersecurity adversarial game as the research context, we design a framework with four plugin modules and three collaboration mechanisms to power LLMs for cybersecurity operations, including both acting as attackers and defenders.

## 3 Cybersecurity Adversarial Game and Pre-trained RL Agents

Before detailing our design, we start with briefly introducing our research context: the cybersecurity adversarial game. In particularly, we have constructed a cybersecurity adversarial game utilizing CybORG Standen et al. ([2021](https://arxiv.org/html/2403.17674v1#bib.bib32)), an exemplary RL-based Autonomous Cyber Operation (ACO) gym. ACO supports the creation of decision-making agents for both the blue team (defender) and the red team (attacker) in adversarial scenarios, and conveys structured and unstructured information, enabling the adaptation of both RL and LLM agents.

### 3.1 Cybersecurity Adversarial Games

The scenario adopted in this study is derived from TTCP CAGE Challenge 1 ¹¹1https://github.com/cage-challenge/cage-challenge-1, an open challenge on CybORG in 2021\. As illustrated in Figure [1](https://arxiv.org/html/2403.17674v1#S3.F1 "Figure 1 ‣ 3.1 Cybersecurity Adversarial Games ‣ 3 Cybersecurity Adversarial Game and Pre-trained RL Agents ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games"), the red and blue teams compete in a simulated network environment, which can be modeled as a partially observed Markov process (POMDP). At each step, the red team and blue team take actions sequentially in the environment, causing changes in the environmental state.

Environment & Observation. The environment comprises a network consisting of 13 hosts divided into three subnets. The red team commences from the footnode in the user subnet without knowledge of any other hosts. The blue team possesses information about all hosts but lacks knowledge regarding the red team’s access status to the hosts.

For both the red and blue team RL agents, their vector observation at each step encompasses: (1) whether the last action is success, (2) whether the adversary has operated on a specific host, and (3) the red team’s access status of a specific host. Note that the observation is not guaranteed accurate due to the presence of an adversary.

![Refer to caption](img/9260b5af0815617ca32ba170fc5eac27.png)

Figure 1: A POMDP cybersecurity adversial game. The red host in User Subnet represents the foot node of the red team. The blue host in Enterprise Subnet represents the defender host of the blue team.

Action & Reward. As shown in Figure [2](https://arxiv.org/html/2403.17674v1#S3.F2 "Figure 2 ‣ 3.1 Cybersecurity Adversarial Games ‣ 3 Cybersecurity Adversarial Game and Pre-trained RL Agents ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games"), the two teams each have three reciprocal actions that cause transitions in the host’s access status. The red team achieves lateral movement between subnets by discovering new hosts through connections from the privileged host. We set the game to be zero-sum, which means that the blue team’s reward is the opposite of the red team’s reward. The reward at each step is based on the extent of red team’s exploitation,

|  | $Reward_{t}=\sum_{i=1}^{n}V_{i,t}\times A_{i,t}$ |  | (1) |

where $V_{i,t}$ and $A_{i,t}$ represents the value and the access status of $host_{i}$ at step $t$ respectively.

![Refer to caption](img/1d5242e002a34fafbe840e7ee797fb93.png)

Figure 2: Action-Status Transition. Red text represents red team actions, blue text represents blue team actions.

### 3.2 Pre-trained RL Agents

In this study, we choose three representative RL algorithms to train red team and blue team agents²²2Our framework is flexible to use other RL algorithms.:

*   •

    A3C (Asynchronous Advantage Actor-Critic) Mnih et al. ([2016](https://arxiv.org/html/2403.17674v1#bib.bib23)) combines policy gradient and value function methods by asynchronously training multiple agents to improve efficiency.

*   •

    DQN (Deep Q-Network) Mnih et al. ([2013](https://arxiv.org/html/2403.17674v1#bib.bib22)) utilizes deep neural networks to approximate the Q-value function to guide the agent’s decisions.

*   •

    PPO (Proximal Policy Optimization) Schulman et al. ([2017](https://arxiv.org/html/2403.17674v1#bib.bib31)), a policy gradient method, ensures stability through proximal policy optimization, restricting the magnitude of policy updates.

The RL-based environment facilitates agent’s training. Red team and blue team agents are trained separately, with one agent trained at a time. For agent’s adversary, we applied the fixed-strategy agents provided in CybORG. In particular, when training a red-team RL agent, we use a blue-team agent with fixed strategy which randomly performs Remove or Restore operations when encountering suspicious hosts during each Monitor action. When training a blue-team RL agent, the red-team agent as the adversary gains access to network nodes one by one based on a breadth-first strategy. Our approach aligns with the conventional RL training paradigm, wherein the agent takes an action at each step, assimilates new observations and associated rewards, and incrementally refines its strategic framework.

## 4 SecurityBot: an LLM-based agent mentored by RL agents

As shown in [3](https://arxiv.org/html/2403.17674v1#S4.F3 "Figure 3 ‣ 4 SecurityBot: an LLM-based agent mentored by RL agents ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games"), our SecurityBot contains three main parts: a LLM-based Agent, the pre-trained RL agent pool as mentors and their collaborative mechanisms.

![Refer to caption](img/bf551776cb2bc0dbdef02f59f2ad8f6d.png)

Figure 3: The Framework of SecurityBot: LLM-based RLs-mentoring Agent for Cybersecurity Operation

### 4.1 LLM Agent Design

Building upon the LLM, GPT 3.5-turbo, our LLM agent includes four plugin modules for decision making in each step:

#### 4.1.1 Profile module

As shown in Figure [4](https://arxiv.org/html/2403.17674v1#S4.F4 "Figure 4 ‣ 4.1.1 Profile module ‣ 4.1 LLM Agent Design ‣ 4 SecurityBot: an LLM-based agent mentored by RL agents ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games"), the Profile module initializes each agent’s role, goal, and available actions depending on its role. In particular, we design a prompt including the expected format for the observed environment as the input, and the expected output which is an action sequence including a series of actions with its goal, trigger, following actions, and expected outcome. When initializing the LLM agent, we use this prompt, together with the assigned goal, action, and environment format, to ask the LLM to generate an action sequence and add it to the profile, serving as the global behavior guidance for the LLM agent.

![Refer to caption](img/7508f40a5177891e20249bc7ba3eeea6.png)

Figure 4: The illustration of profile module, including the example of roles, goals, actions, environment format and the generated behavior guidance (the bottom part) as well as the process to generate the behavior guidance (the upper part).

#### 4.1.2 Memory module

The Memory module is used to store past experiences and search the related ones for decision making in each step.

Memory Storage. The memory module stores records including the timestamp, observed environment, action taken, and the outcome including the action status (success or failure) and its reward. In particular, when storing each memory record, the LLM agent rates its Importance by prompting the LLM to score it on a scale of 0 to 10.

Memory Searching. When searching memories to support action selection in each step, the LLM agent will calculate each memory record’s Relevance and Freshness:

*   •

    Relevance: measuring its environment’s similarity with the current one. We transformed each environment into vectors, and then calculate their cosine similarity.

*   •

    Freshness: measuring its freshness, represented as the reciprocal of its timestamp gap with the current step.

Finally, we calculate the product of the importance, relevance and freshness for each memory record and select the top two as the memory input for LLM when making decision.

#### 4.1.3 Action module

The Action module plays a crucial role in guiding the LLM agent to take valid action for each step. In particularly, given the observed environment and the available actions provided by the profile, this module will generate the action space with all the potential actions that the agent could take.

#### 4.1.4 Reflection module

Given the complex and dynamic environment, as the adversary agent may change the environment but is unobservant to the LLM agent, the LLM agent may encounter dilemmas situation, reflected as repetitive actions or diminishing rewards. For example, the red agent might persist in attacking a host in the network, even when such an action has been proven futile. Hence, the reflection module is designed to monitor the dilemma status and trigger the reflection process.

![Refer to caption](img/386f657de867a68514427c1477c3f140.png)

Figure 5: The prompt for Red Agent from the reflection module to motivate the LLM to choose other attack actions.

Dilemmas Monitor. At every step, the Reflection module evaluates both the Reward List and the Action List from the previous steps. If there is no increase in rewards or if the agent repeats an action, the module will collect these suspicious actions, including the series of actions associated with those records, and then activate the reflection process.

Reflection Process. The reflection process will pass these suspicious actions to the Action module and remove them if they are included in the generated action space. Additionally, as shown in Figure [5](https://arxiv.org/html/2403.17674v1#S4.F5 "Figure 5 ‣ 4.1.4 Reflection module ‣ 4.1 LLM Agent Design ‣ 4 SecurityBot: an LLM-based agent mentored by RL agents ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games"), the process provides the LLM with a prompt, elucidating that the agent is stuck in the dilemmas and providing the possible reasons to guide the LLM to choose other actions to get out of the dilemma situation.

### 4.2 Collaboration with RL agents

Using RL agents as mentors to guide the LLM agent is critical for SecurityBot to achieve better performance. More specifically, as shown in Figure [6](https://arxiv.org/html/2403.17674v1#S4.F6 "Figure 6 ‣ 4.2 Collaboration with RL agents ‣ 4 SecurityBot: an LLM-based agent mentored by RL agents ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games"), we design three collaboration mechanisms:

![Refer to caption](img/085a62f930a78f2d1481b4e6e3409a78.png)

Figure 6: Mechanisms to collaboration with RL agents. Different color refers to suggestions of different RL mentors.

#### 4.2.1 Cursor: growing to be independent

Firstly, the RL agents are pre-trained in the same environment particularly to guarantee that they can provide knowledge to mentor the LLM agent to make better decisions, especially in the early stage when LLM agents contain no information regarding the environment. However, as time goes by, the LLM agent, with its capacity to understand complex environments and accumulated experience, can surpass the RL mentors (which we will report later). Hence, we design the mechanism Cursor to decide whether the LLM agent should take suggestions from RL agents.

In particular, for each step $t$, the Cursor module will calculate an independence value $ind_{t}$ and only when the independence value $ind_{t}$ is below the given threshold $\theta_{ind}$, the LLM agent will consider suggestions from RL agents. Otherwise, the LLM agent will make the decision by itself. Hence, the Cursor module will adjust an independence value $ind_{t}$ in a way to reflects the tendency to rely on itself and consider the mentor’s suggestion when it proves beneficial. As detailed in Equation [2](https://arxiv.org/html/2403.17674v1#S4.E2 "In 4.2.1 Cursor: growing to be independent ‣ 4.2 Collaboration with RL agents ‣ 4 SecurityBot: an LLM-based agent mentored by RL agents ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games"), we adopt the monotonically increasing function $f_{x}$ (Equation [3](https://arxiv.org/html/2403.17674v1#S4.E3 "In 4.2.1 Cursor: growing to be independent ‣ 4.2 Collaboration with RL agents ‣ 4 SecurityBot: an LLM-based agent mentored by RL agents ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games")) so that $part_{1}$ reflects the trend to rely on the LLM itself. $part_{2}$ represents the trend of gaining reward from previous actions while $part_{3}$ is the signal function (Equation [4](https://arxiv.org/html/2403.17674v1#S4.E4 "In 4.2.1 Cursor: growing to be independent ‣ 4.2 Collaboration with RL agents ‣ 4 SecurityBot: an LLM-based agent mentored by RL agents ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games")) indicating whether the action is chosen when considering suggestions from mentors. In other words, if the LLM agent achieves an increasing reward without mentoring by the RLs, we would increase the independence value to make LLM agent more independent. Note that we introduce parameter $\alpha$ to control the change race and $\theta_{lr}$ to represent the minimal reward increment that we would expect the LLM agent to gain.

|  |  $\begin{aligned} ind_{t}&=ind_{t-1}+\underbrace{(f_{t}-f_{t-1})}_{\text{part\_1% }}\\ &\quad+\underbrace{\min(\alpha\times ind_{t-1},(r_{t-1}-r_{t-2}-\theta_{lr}))}% _{\text{part\_2}}&\quad\times\underbrace{\operatorname{sgn}(ind_{t-1}-\theta_{% ind})}_{\text{part\_3}}\end{aligned}$  |  | (2) |

|  | $\leavevmode\resizebox{108.405pt}{}{$\displaystyle f_{x}=\frac{1}{1+e^{-kx}}$}.$ |  | (3) |

|  | $\leavevmode\resizebox{195.12767pt}{}{$\displaystyle sgn(x)=\{\begin{array}[]{c% }-1\ \ \ if\ x>0\\ 1\ \ \ \ \ otherwise\end{array}$}.$ |  | (4) |

#### 4.2.2 Aggregator: ranking suggestions from multiple mentors

Rather than relying on only one RL agent, the LLM can refer to multiple RL agents, as different RL agents may catch different aspects of the tasks. Hence, we further introduce the aggregator mechanism to aggregate suggestions from multiple RL agents. In particular, given the top three suggestions from all the RL mentors associated with confidence, the multi-mentor mechanism will sort them based on the confidence and the top one will be presented to the LLM and the top three actions will be provided while in dilemmas. In such a case, the LLM agent does not necessarily always get suggestions from one specific RL agent during the whole task duration.

#### 4.2.3 Caller: asking for help proactively when in dilemma

As discussed above, when the LLM agent encounters a dilemma, the reflection module will be activated. Furthermore, beyond activating the reflection process, the LLM agent can further refer to RL agents for support. Unlike referring to RL mentors’ input in normal situation where only one suggestion is provided, we will provide the top three confident suggestions from the RL mentors.

## 5 Experiments and Results

### 5.1 Experiment Setup

Environment. Following the setup of Cage Challenge 1, we set the maximum number of steps in one episode, i.e., a complete round of the game, to be 100\. As mentioned earlier, we set two reward parameters as shown in Table [1](https://arxiv.org/html/2403.17674v1#S5.T1 "Table 1 ‣ 5.1 Experiment Setup ‣ 5 Experiments and Results ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games"): (1) Host value. The hosts in different subnets have different values, and (2) Access state. The higher the access state of a host, the higher the proportion of host value obtained by the red team.

Table 1: Parameters of agent reward.

| Host Subnet(V) | Reward | Access status(A) | Reward |
| --- | --- | --- | --- |
| User Subnet | 0.1 | Unknown/Known | 0 |
| Enterprise Subnet | 1.0 | Exploited | 0.5 |
| Operational Subnet | 10.0 | Privileged | 0.89 |

RL Training. The RL training process is based on the Ray RLlib, a Python library for RL³³3We focuses on the collaboration between RL agents and LLM agents, rather than training a better RL agent. Hence, we choose the adversary using the simplest strategy and default parameters without parameter tuning for the training algorithms are used. All specific algorithm parameters can refer to https://github.com/ray-project/ray/blob/master/rllib/algorithms/. Each training process consists of a total of 100 iterations (4000 episodes in total).

LLMs Setup. We leverage OpenAI’s gpt-3.5-turbo API for building the LLM Agent. All the temperatures are set to 0 to restrict the format of LLM output.

*   •

    Reflection. If the action is repeated in the last three steps, or if there is no increase in reward values in the last five steps, the reflection mechanism will be triggered.

*   •

    Cursor. $\theta_{ind}$ is set to 0.6\. $\theta_{lr}$ is set to 0.3\. $\alpha$ is set to 0.3\. $k$ in $f(x)$ is set to 0.0135.

Measurement. We consider the following measurements.

*   •

    Step reward. The reward of each step.

*   •

    Collaboration Rate ($Col$). The rate of cooperation with RL agents.

*   •

    Dilemma Rate ($DR$). The rate of collaborating with RL agents triggered by trapping into dilemma situation.

*   •

    Accept Rate ($AR$). The rate that LLM agent takes RL mentor’s suggestion, indicating the extent to which LLM Agent relies on RL mentors.

*   •

    Accept Rate in dilemma ($AR_{d}$). The rate LLM agent take suggestions when trapping into dilemma, showing the ability of RL mentors to help LLM Agent out.

Experiment Group. We incrementally add collaboration modules and assess their performance for both the red and blue team. For each group, we run the simulation for 5 times and calculate the average.

*   •

    Independent. Each RL agent (A3C, DQN, PPO) and our designed LLM agent conduct the task independently.

*   •

    Single-mentor. The LLM agent cooperate with a single RL agent (A3C&LLM, DQN&LLM, PPO&LLM).

*   •

    Multi-mentors. LLM agent cooperate with all three different RL agents (MultiMentor).

### 5.2 Performance in Red Team Task

In the red team task, LLM agents and RL agents exhibited distinct action patterns, indicative of differing knowledge bases. While collaborative synergy can surpass individual agent performance, optimal collaboration is achieved when RL agents exhibit superior performance. However, when the LLM agents considers suggestions from multiple RL agents, it struggles to efficiently process this information, leading to a decline in collaborative performance.⁴⁴4We smoothed the data using exponential smoothing and calculated confidence intervals

Figure 7: Result of red team task. (a) Comparison between LLM and PPO. They have different performances in different stages. (b)Single RL mentor result. PPO&LLM surpasses all others. (c)Comparison between Multi and Single RL mentor. PPO&LLM still performs best

#### 5.2.1 Complement knowledge of LLM agents and RL mentors

As depicted in Figure [7](https://arxiv.org/html/2403.17674v1#S5.F7 "Figure 7 ‣ 5.2 Performance in Red Team Task ‣ 5 Experiments and Results ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games")(a)⁵⁵5The performance of the three RL agents varies, while the PPO agent demonstrating superior performance. Due to space limitation, we only report the performance for PPO agent., the reward curves of the LLM agent and the PPO agent intersect: the PPO agent rapidly accumulates rewards early on, leveling off later. This behavior arises from the PPO agent gaining environmental knowledge during training, recognizing the high value of hosts in the Operational subnet. While exhibiting depth-first characteristics, insufficient training causes it to converge to a local optimum.

Conversely, the LLM agent, despite a modest early-stage reward, achieves rapid growth, outperforming the PPO agent in the later stage. The LLM agent’s behavior follows a breadth-first pattern, accumulating more exploited hosts in the network efficiently avoiding defender blocks, resulting in a higher reward.

Taking a step further, we find that LLM agents outperform PPO agents in single-step gains occurring at step 53 on average, where we differentiate the early and later stages. In later stage, we find that RL mentors always repeat one action, while LLM agent, with the reflection module, can prevent the problem. This can be the reason why RL mentors’ performance is worse than LLM agent in the stage.

#### 5.2.2 Amplification effect of single-mentor mechanisms

A stronger RL mentor enhances collaborative performance, otherwise it may slows down the LLM agent’s process. As shown in Figure [7](https://arxiv.org/html/2403.17674v1#S5.F7 "Figure 7 ‣ 5.2 Performance in Red Team Task ‣ 5 Experiments and Results ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games")(b), PPO and A3C agents exhibit superior collaborative performance compared to LLM agents alone, and in particular, the PPO&LLM group demonstrating a synergistic $1+1>2$ effect throughout the process, as well as getting into the rapid-growth phase much earlier.

Table 2: Cooperation metric of red team task

| Metric | PPO&LLM | A3C&LLM | DQN&LLM |
| --- | --- | --- | --- |
|  | $Early\backslash Later$ | $Early\backslash Later$ | $Early\backslash Later$ |
| $Col$ | $61.5\%\backslash 33.3\%$ | $78.8\%\backslash 56.2\%$ | $53.8\%\backslash 43.7\%$ |
| $DR$ | $50.0\%\backslash 100.0\%$ | $34.1\%\backslash 55.6\%$ | $39.3\%\backslash 80.9\%$ |
| $AR$ | $50.0\%\backslash 63.6\%$ | $29.2\%\backslash 51.9\%$ | $35.7\%\backslash 57.1\%$ |
| $AR_{d}$ | $50.0\%\backslash 63.6\%$ | $28.6\%\backslash 53.3\%$ | $27.3\%\backslash 52.9\%$ |

Furthermore, the cooperation mechanism guides the LLM agent to learn from RL mentors in the early stage while seeking help in dilemmas. As shown in Table [2](https://arxiv.org/html/2403.17674v1#S5.T2 "Table 2 ‣ 5.2.2 Amplification effect of single-mentor mechanisms ‣ 5.2 Performance in Red Team Task ‣ 5 Experiments and Results ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games"), the LLM agent collaborates more with RL mentors in the early stages than later, satisfying our design goal. $DR$ are all higher in the later stage, meaning most collaborations with the RL agent are triggered by the dilemmas situation. Interestingly, $AR$ and $AR_{d}$ values are both higher in the later stage, meaning in the later stage, despite outperforming the RL mentor, the LLM agent relies more on the RL mentor’s suggestions if needed.

#### 5.2.3 Noise from multi-mentors

We explored whether the LLM agent could gain more knowledge from recommendations of multi-mentors. In our setup, assistance from multiple RL mentors is not necessary helpful. As shown in Figure [7](https://arxiv.org/html/2403.17674v1#S5.F7 "Figure 7 ‣ 5.2 Performance in Red Team Task ‣ 5 Experiments and Results ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games")(c), while the performance of multi-tutors slightly outperforms LLM alone, it falls short of the LLM&PPO group. We observed that 75.61% of suggestions from RL mentors originated from DQN, but only 5.41% were accepted. In contrast, 34.61% of PPO’s suggestions were accepted. Moreover, only 15.85% of all RL suggestions were accepted, markedly lower than the acceptance rate in a single mentor scenario. This disparity illuminates the high confidence suggestion from the low performance mentor became a noise for the LLM agent.

### 5.3 Performance in Blue Team Task

Figure 8: Result of blue team task. (a)Comparison between LLM and PPO. LLM outperform PPO in Blue Team Task. (b) Single RL mentor result. PPO&LLM perform slightly better than LLM. (c) Comparison between Multi and Single RL mentor.Multi-mentor perform best on average, but not stable enough.

#### 5.3.1 A helpful but narrower complementary knowledge

As shown in Figure [8](https://arxiv.org/html/2403.17674v1#S5.F8 "Figure 8 ‣ 5.3 Performance in Blue Team Task ‣ 5 Experiments and Results ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games") (a), the LLM agents demonstrate performance similar to the PPO agent during the early stages. But after a brief period of divergence, the LLM agent consistently outperforms the PPO agent. We observe the similar situation in the case of single mentor. As shown in Figure [8](https://arxiv.org/html/2403.17674v1#S5.F8 "Figure 8 ‣ 5.3 Performance in Blue Team Task ‣ 5 Experiments and Results ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games") (b), although PPO&LLM group demonstrates a marginally superior performance over LLM agent, this advantage is not observed in other groups. These results indicate a narrower knowledge gap between LLM and RL agents in blue team task, may due to the fact that the whole network environment is used for pre-training RL agents and provided to LLM agent.

Additionally, as reported in Table [3](https://arxiv.org/html/2403.17674v1#S5.T3 "Table 3 ‣ 5.3.1 A helpful but narrower complementary knowledge ‣ 5.3 Performance in Blue Team Task ‣ 5 Experiments and Results ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games"), the LLM agent would accept RL mentors’ suggestions in the early stages. While in the later stage, both A3C&LLM and DQN&LLM groups show little interest in RL mentor’s suggestion except trapped in dilemmas. Conversely, we can observe consistently higher $AR$ rates in the later stages for PPO&LLM. This discrepancy indicates the LLM agent’s capability in identifying the suggestion quality and the importance of providing high quality suggestion to improve the LLM agent’s effectiveness.

Table 3: Cooperation result in blue team task

| Metric | PPO&LLM | A3C&LLM | DQN&LLM |
| --- | --- | --- | --- |
|  | $Early\backslash Later$ | $Early\backslash Later$ | $Early\backslash Later$ |
| $Col$ | $48.1\%\backslash 22.9\%$ | $78.8\%\backslash 33.3\%$ | $71.2\%\backslash 16.7\%$ |
| $DR$ | $40.0\%\backslash 45.5\%$ | $43.9\%\backslash 100.0\%$ | $27.0\%\backslash 100.0\%$ |
| $AR$ | $100.0\%\backslash 81.9\%$ | $53.7\%\backslash 28.6\%$ | $91.9\%\backslash 28.6\%$ |
| $AR_{d}$ | $100.0\%\backslash 60.0\%$ | $31.3\%\backslash 28.6\%$ | $100.0\%\backslash 28.6\%$ |

#### 5.3.2 Outstanding but unstable performance of multi-mentors

In contrast to the red team task, as shown in Figure [8](https://arxiv.org/html/2403.17674v1#S5.F8 "Figure 8 ‣ 5.3 Performance in Blue Team Task ‣ 5 Experiments and Results ‣ Depending on yourself when you should: Mentoring LLM with RL agents to become the master in cybersecurity games") (c), the incorporation of multiple RL mentors enhances the average performance of the blue team task beyond that of both the LLM agents and the PPO&LLM group. However, this configuration exhibits instability demonstrated as a larger confidence intervals. While it effectively defends nearly all hosts at times, in some instances, its performance is comparable to that of a single LLM. Notably, the LLM Agent accepts less than 5% of suggestions from RL mentors, predominantly originating from DQN. One reason behind this is that the most confident RL suggestions are not consistently the most effective, especially when provided by multiple mentors.

Additionally, in the blue team task, LLM agents showcase a superior understanding of the environment, often acting independently in most situations. Particularly in scenarios where the LLM agent successfully defends almost all hosts, it appears to disregard unreliable suggestions from multiple RL mentors, opting to make critical decisions autonomously.

## 6 Conclusion and Future Work

This study presents SecurityBot, a LLM agent powered by mentoring from pre-trained RL agents for cybersecurity operations. In particular, with the designed plugin modules, including the profile, memory, reflection and action modules to enhance the LLM, and three collaboration mechanisms, including a cursor, an aggregator and a caller, to effectively collaborate with pre-trained RL agents, the LLM agent achieve significant performance improvement in both cyber attack and defense tasks. Although RL agents can learn local knowledge effectively through pre-training, the LLM agent can surpass them through learning the environments in the later stage. This confirms that our designed LLM agent can be a promising solution to support cybersecurity operations.

While RL agents’ suggestions can be helpful, especially when the LLM agent trapped in dilemmas, as observed in our result, weak RL agents may serve as a noise to distract the LLM agent. Further research can design advanced aggregating strategies to extract the essence and discard the dross from RL agents. Furthermore, while we aim at empowering LLM with plugin modules and collaborating it with RL agents, we did not fintune the LLM or optimize the RL agents. Future studies can fintune a better LLM model specific for cybersecurity operations and train optimized RL agents, which could further improve the SecurityBot’s performance.

## References

*   Ali and Kostakos [2023] Tarek Ali and Panos Kostakos. Huntgpt: Integrating machine learning-based anomaly detection and explainable ai with large language models (llms). arXiv preprint arXiv:2309.16021, 2023.
*   Botacin [2023] Marcus Botacin. Gpthreats-3: Is automatic malware generation a threat? In 2023 IEEE Security and Privacy Workshops (SPW), pages 238–254\. IEEE, 2023.
*   Brohan et al. [2023] Anthony Brohan, Yevgen Chebotar, Chelsea Finn, Karol Hausman, Alexander Herzog, Daniel Ho, Julian Ibarz, Alex Irpan, Eric Jang, Ryan Julian, et al. Do as i can, not as i say: Grounding language in robotic affordances. In Conference on Robot Learning, pages 287–318\. PMLR, 2023.
*   Carta et al. [2022] Thomas Carta, Pierre-Yves Oudeyer, Olivier Sigaud, and Sylvain Lamprier. Eager: Asking and answering questions for automatic reward shaping in language-guided rl. Advances in Neural Information Processing Systems, 35:12478–12490, 2022.
*   Chen and Shu [2023] Canyu Chen and Kai Shu. Can llm-generated misinformation be detected? arXiv preprint arXiv:2309.13788, 2023.
*   Dasgupta et al. [2023] Ishita Dasgupta, Christine Kaeser-Chen, Kenneth Marino, Arun Ahuja, Sheila Babayan, Felix Hill, and Rob Fergus. Collaborating with language models for embodied reasoning. arXiv preprint arXiv:2302.00763, 2023.
*   Deng et al. [2023] Gelei Deng, Yi Liu, Víctor Mayoral-Vilches, Peng Liu, Yuekang Li, Yuan Xu, Tianwei Zhang, Yang Liu, Martin Pinzger, and Stefan Rass. Pentestgpt: An llm-empowered automatic penetration testing tool. arXiv preprint arXiv:2308.06782, 2023.
*   Dong et al. [2023] Yihong Dong, Xue Jiang, Zhi Jin, and Ge Li. Self-collaboration code generation via chatgpt. arXiv preprint arXiv:2304.07590, 2023.
*   Du et al. [2023] Yuqing Du, Olivia Watkins, Zihan Wang, Cédric Colas, Trevor Darrell, Pieter Abbeel, Abhishek Gupta, and Jacob Andreas. Guiding pretraining in reinforcement learning with large language models. arXiv preprint arXiv:2302.06692, 2023.
*   Falade [2023] Polra Victor Falade. Decoding the threat landscape: Chatgpt, fraudgpt, and wormgpt in social engineering attacks. arXiv preprint arXiv:2310.05595, 2023.
*   Hong et al. [2023] Sirui Hong, Xiawu Zheng, Jonathan Chen, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, et al. Metagpt: Meta programming for multi-agent collaborative framework. arXiv preprint arXiv:2308.00352, 2023.
*   Hu et al. [2023] Bin Hu, Chenyang Zhao, Pu Zhang, Zihao Zhou, Yuanhang Yang, Zenglin Xu, and Bin Liu. Enabling intelligent interactions between an agent and an llm: A reinforcement learning approach. arXiv preprint arXiv:2306.03604, 2023.
*   Iannone et al. [2022] Emanuele Iannone, Roberta Guadagni, Filomena Ferrucci, Andrea De Lucia, and Fabio Palomba. The secret life of software vulnerabilities: A large-scale empirical study. IEEE Transactions on Software Engineering, 49(1):44–63, 2022.
*   Ji et al. [2023] Ziwei Ji, Nayeon Lee, Rita Frieske, Tiezheng Yu, Dan Su, Yan Xu, Etsuko Ishii, Ye Jin Bang, Andrea Madotto, and Pascale Fung. Survey of hallucination in natural language generation. ACM Computing Surveys, 55(12):1–38, 2023.
*   Kahneman [2011] Daniel Kahneman. Thinking, fast and slow. macmillan, 2011.
*   Kwon et al. [2023] Minae Kwon, Sang Michael Xie, Kalesha Bullard, and Dorsa Sadigh. Reward design with language models. In The Eleventh International Conference on Learning Representations, ICLR 2023, Kigali, Rwanda, May 1-5, 2023. OpenReview.net, 2023.
*   Li et al. [2023] Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, and Bernard Ghanem. Camel: Communicative agents for ”mind” exploration of large language model society. In Thirty-seventh Conference on Neural Information Processing Systems, 2023.
*   Lin et al. [2023] Bill Yuchen Lin, Yicheng Fu, Karina Yang, Faeze Brahman, Shiyu Huang, Chandra Bhagavatula, Prithviraj Ammanabrolu, Yejin Choi, and Xiang Ren. Swiftsage: A generative agent with fast and slow thinking for complex interactive tasks. In Thirty-seventh Conference on Neural Information Processing Systems, 2023.
*   Ma et al. [2023] Yecheng Jason Ma, William Liang, Guanzhi Wang, De-An Huang, Osbert Bastani, Dinesh Jayaraman, Yuke Zhu, Linxi Fan, and Anima Anandkumar. Eureka: Human-level reward design via coding large language models. arXiv preprint arXiv:2310.12931, 2023.
*   McIntosh et al. [2023] Timothy McIntosh, Tong Liu, Teo Susnjak, Hooman Alavizadeh, Alex Ng, Raza Nowrozy, and Paul Watters. Harnessing gpt-4 for generation of cybersecurity grc policies: A focus on ransomware attack mitigation. Computers & Security, 134:103424, 2023.
*   Micheli et al. [2023] Vincent Micheli, Eloi Alonso, and François Fleuret. Transformers are sample-efficient world models. In The Eleventh International Conference on Learning Representations, ICLR 2023, Kigali, Rwanda, May 1-5, 2023. OpenReview.net, 2023.
*   Mnih et al. [2013] Volodymyr Mnih, Koray Kavukcuoglu, David Silver, Alex Graves, Ioannis Antonoglou, Daan Wierstra, and Martin Riedmiller. Playing atari with deep reinforcement learning. arXiv preprint arXiv:1312.5602, 2013.
*   Mnih et al. [2016] Volodymyr Mnih, Adria Puigdomenech Badia, Mehdi Mirza, Alex Graves, Timothy Lillicrap, Tim Harley, David Silver, and Koray Kavukcuoglu. Asynchronous methods for deep reinforcement learning. In International conference on machine learning, pages 1928–1937\. PMLR, 2016.
*   Noever [2023] David Noever. Can large language models find and fix vulnerable software? arXiv preprint arXiv:2308.10345, 2023.
*   Palmer et al. [2023] Gregory Palmer, Chris Parry, Daniel J. B. Harrold, and Chris Willis. Deep reinforcement learning for autonomous cyber operations: A survey, 2023.
*   Park et al. [2023] Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang, and Michael S Bernstein. Generative agents: Interactive simulacra of human behavior. In Proceedings of the 36th Annual ACM Symposium on User Interface Software and Technology, pages 1–22, 2023.
*   Pearce et al. [2023] Hammond Pearce, Benjamin Tan, Baleegh Ahmad, Ramesh Karri, and Brendan Dolan-Gavitt. Examining zero-shot vulnerability repair with large language models. In 2023 IEEE Symposium on Security and Privacy (SP), pages 2339–2356\. IEEE, 2023.
*   Qi et al. [2023] Jiaxing Qi, Shaohan Huang, Zhongzhi Luan, Carol Fung, Hailong Yang, and Depei Qian. Loggpt: Exploring chatgpt for log-based anomaly detection. arXiv preprint arXiv:2309.01189, 2023.
*   Qian et al. [2023] Chen Qian, Xin Cong, Cheng Yang, Weize Chen, Yusheng Su, Juyuan Xu, Zhiyuan Liu, and Maosong Sun. Communicative agents for software development. arXiv preprint arXiv:2307.07924, 2023.
*   Sandoval et al. [2023] Gustavo Sandoval, Hammond Pearce, Teo Nys, Ramesh Karri, Siddharth Garg, and Brendan Dolan-Gavitt. Lost at c: A user study on the security implications of large language model code assistants. arXiv preprint arXiv:2208.09727, 2023.
*   Schulman et al. [2017] John Schulman, Filip Wolski, Prafulla Dhariwal, Alec Radford, and Oleg Klimov. Proximal policy optimization algorithms. arXiv preprint arXiv:1707.06347, 2017.
*   Standen et al. [2021] Maxwell Standen, Martin Lucas, David Bowman, Toby J Richer, Junae Kim, and Damian Marriott. Cyborg: A gym for the development of autonomous cyber agents. arXiv preprint arXiv:2108.09118, 2021.
*   Taddeo et al. [2019] Mariarosaria Taddeo, Tom McCutcheon, and Luciano Floridi. Trusting artificial intelligence in cybersecurity is a double-edged sword. Nature Machine Intelligence, 1(12):557–560, 2019.
*   Talebirad and Nadiri [2023] Yashar Talebirad and Amirhossein Nadiri. Multi-agent collaboration: Harnessing the power of intelligent llm agents. arXiv preprint arXiv:2306.03314, 2023.
*   Team. [2021] Microsoft Defender Research Team. Cyberbattlesim. [https://github.com/microsoft/cyberbattlesim](https://github.com/microsoft/cyberbattlesim), 2021. Created by Christian Seifert, Michael Betser, William Blum, James Bono, Kate Farris, Emily Goren, Justin Grana, Kristian Holsheimer, Brandon Marken, Joshua Neil, Nicole Nichols, Jugal Parikh, Haoran Wei.
*   Vyas et al. [2023] Sanyam Vyas, John Hannay, Andrew Bolton, and Professor Pete Burnap. Automated cyber defence: A review, 2023.
*   Wan et al. [2022] Yue Wan, Chang-Yu Hsieh, Ben Liao, and Shengyu Zhang. Retroformer: Pushing the limits of end-to-end retrosynthesis transformer. In International Conference on Machine Learning, pages 22475–22490\. PMLR, 2022.
*   Wang et al. [2023a] Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin, et al. A survey on large language model based autonomous agents. arXiv preprint arXiv:2308.11432, 2023.
*   Wang et al. [2023b] Zhilin Wang, Yu Ying Chiu, and Yu Cheung Chiu. Humanoid agents: Platform for simulating human-like generative agents. arXiv preprint arXiv:2310.05418, 2023.
*   Wang et al. [2023c] Zihao Wang, Shaofei Cai, Anji Liu, Yonggang Jin, Jinbing Hou, Bowei Zhang, Haowei Lin, Zhaofeng He, Zilong Zheng, Yaodong Yang, et al. Jarvis-1: Open-world multi-task agents with memory-augmented multimodal language models. arXiv preprint arXiv:2311.05997, 2023.
*   Wason and Evans [1974] Peter C Wason and J St BT Evans. Dual processes in reasoning? Cognition, 3(2):141–154, 1974.
*   Wu et al. [2023] Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang, and Chi Wang. Autogen: Enabling next-gen llm applications via multi-agent conversation framework. arXiv preprint arXiv:2308.08155, 2023.
*   Xia et al. [2023] Chunqiu Steven Xia, Matteo Paltenghi, Jia Le Tian, Michael Pradel, and Lingming Zhang. Universal fuzzing via large language models. arXiv preprint arXiv:2308.04748, 2023.
*   Xu et al. [2023] Yuzhuang Xu, Shuo Wang, Peng Li, Fuwen Luo, Xiaolong Wang, Weidong Liu, and Yang Liu. Exploring large language models for communication games: An empirical study on werewolf. arXiv preprint arXiv:2309.04658, 2023.
*   Yaman [2023] Ferhat Yaman. Agent SCA: Advanced Physical Side Channel Analysis Agent with LLMs. PhD thesis, North Carolina State University, 2023.
*   Yao et al. [2023] Yifan Yao, Jinhao Duan, Kaidi Xu, Yuanfang Cai, Eric Sun, and Yue Zhang. A survey on large language model (llm) security and privacy: The good, the bad, and the ugly. arXiv preprint arXiv:2312.02003, 2023.
*   Zhang et al. [2023] Jintian Zhang, Xin Xu, and Shumin Deng. Exploring collaboration mechanisms for llm agents: A social psychology view. arXiv preprint arXiv:2310.02124, 2023.