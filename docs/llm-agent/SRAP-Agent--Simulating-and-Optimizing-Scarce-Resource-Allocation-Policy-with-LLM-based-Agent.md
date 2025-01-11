<!--yml
category: 未分类
date: 2025-01-11 12:04:23
-->

# SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent

> 来源：[https://arxiv.org/html/2410.14152/](https://arxiv.org/html/2410.14152/)

Jiarui Ji¹,Yang Li¹, Hongtao Liu¹, Zhicheng Du¹,
Zhewei Wei^(1,2), Weiran Shen^(1,2), Qi Qi^(1,2), Yankai Lin^(1,2)
¹Gaoling School of Artificial Intelligence, Renmin University of China, Beijing, China
²Beijing Key Laboratory of Big Data Management and Analysis Methods, Beijing, China  Corresponding Author.

###### Abstract

Public scarce resource allocation plays a crucial role in economics as it directly influences the efficiency and equity in society. Traditional studies including theoretical model-based, empirical study-based and simulation-based methods encounter limitations due to the idealized assumption of complete information and individual rationality, as well as constraints posed by limited available data. In this work, we propose an innovative framework, SRAP-Agent (Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent), which integrates Large Language Models (LLMs) into economic simulations, aiming to bridge the gap between theoretical models and real-world dynamics. Using public housing allocation scenarios as a case study, we conduct extensive policy simulation experiments to verify the feasibility and effectiveness of the SRAP-Agent and employ the Policy Optimization Algorithm with certain optimization objectives. The source code can be found in [https://github.com/jijiarui-cather/SRAPAgent_Framework](https://github.com/jijiarui-cather/SRAPAgent_Framework).

SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent

Jiarui Ji¹,Yang Li¹, Hongtao Liu¹, Zhicheng Du¹, Zhewei Wei^(1,2), Weiran Shen^(1,2), Qi Qi^(1,2), Yankai Lin^(1,2)^†^†thanks:  Corresponding Author. ¹Gaoling School of Artificial Intelligence, Renmin University of China, Beijing, China ²Beijing Key Laboratory of Big Data Management and Analysis Methods, Beijing, China

## 1 Introduction

![Refer to caption](img/dfdf4603fc58cae4886468ae127ce607.png)

Figure 1: An illustration of the SRAP-Agent framework. The horn symbolizes the broadcasting process of policy information to participants.

Economics (Mankiw, [2011](https://arxiv.org/html/2410.14152v1#bib.bib32)) delves into how to limit scarce resources to meet unlimited needs. A critical aspect of this field is the allocation of public scarce goods (Jr et al., [1977](https://arxiv.org/html/2410.14152v1#bib.bib24); Groves and Ledyard, [1974](https://arxiv.org/html/2410.14152v1#bib.bib17); BROCK, [1980](https://arxiv.org/html/2410.14152v1#bib.bib9)), focusing on utilizing limited resources to improve economic efficiency and social welfare  (Blümel et al., [1986](https://arxiv.org/html/2410.14152v1#bib.bib7)). In the field of research on the allocation of public scarce resources, existing work can be primarily categorized into three main approaches: (1) theoretical model-based methods (Hylland and Zeckhauser, [1979](https://arxiv.org/html/2410.14152v1#bib.bib20); Su and Zenios, [2004](https://arxiv.org/html/2410.14152v1#bib.bib45), [2005](https://arxiv.org/html/2410.14152v1#bib.bib46); Chen et al., [2018](https://arxiv.org/html/2410.14152v1#bib.bib11)), which utilizes economic theories to develop models that can predict how resources should be allocated efficiently; (2) empirical study-based methods (Banerjee and Somanathan, [2007](https://arxiv.org/html/2410.14152v1#bib.bib4); Nold, [1992](https://arxiv.org/html/2410.14152v1#bib.bib35); Falkinger et al., [2000](https://arxiv.org/html/2410.14152v1#bib.bib14)), which analyze real-world data to uncover patterns, correlations, and the effects of various policies; (3) computational simulation-based methods (Holland and Miller, [1991](https://arxiv.org/html/2410.14152v1#bib.bib18); Zheng et al., [2022](https://arxiv.org/html/2410.14152v1#bib.bib56)), which emulate economic environments to test economic hypotheses within simulated settings. However, empirical study-based methods often face challenges of data scarcity, establishing causality, and isolating variables in complex social systems. Meanwhile, theoretical model-based methods and computational simulation-based methods often rely on simplified assumptions, overlooking the complex interplay of rational and social behaviors in human decision-making.

Fortunately, the emergence of LLMs such as ChatGPT and GPT-4 (Anil et al., [2023](https://arxiv.org/html/2410.14152v1#bib.bib3); Touvron et al., [2023](https://arxiv.org/html/2410.14152v1#bib.bib47); Brown et al., [2020](https://arxiv.org/html/2410.14152v1#bib.bib10); OpenAI, [2023](https://arxiv.org/html/2410.14152v1#bib.bib36)), has introduced a new potential in economic simulations. Acclaimed for their ability to mimic human-like behaviors (Li et al., [2023a](https://arxiv.org/html/2410.14152v1#bib.bib30); Park et al., [2022](https://arxiv.org/html/2410.14152v1#bib.bib38), [2023](https://arxiv.org/html/2410.14152v1#bib.bib37)), LLMs demonstrate the potential to encapsulate the logic and patterns inherent in human cognition by pre-training on extensive web data (OpenAI, [2023](https://arxiv.org/html/2410.14152v1#bib.bib36)). This breakthrough facilitates the incorporation of social consensus mechanisms into economic simulations, offering a comprehensive framework for evaluating the impact of resource allocation policies on both economic efficiency and social welfare.

In this work, we develop an LLM-based scarce resource allocation policy simulation agent, SRAP-Agent. It abstracts the resource allocation queues for policy execution, with LLM-based agents carefully designed for the simulation of participants’ behaviors. Furthermore, we propose the genetic algorithm-based policy optimization algorithm (POA) to find optimal policies towards pre-defined targets. Through experiments in the context of public housing allocation, we validate the effectiveness of SRAP-Agent, which reveals several key insights: (1) The LLM-based agent can effectively simulate the emotion factor and strategic behaviors of humans. Compared to the GPT-4-driven agent, the simulated decision-making behavior of the GPT-3.5-driven agent is more similar to humans’ behavior. (2) In scarce resource allocation, pivotal factors include the entry conditions of participants, the queuing method, and the categorization of resources. (3) POA can optimize policies towards a specific policy evaluation metric, and improve approximately 20% on this metric.

## 2 Related Work

Public Scarce Resources Allocation. The allocation of public scarce resources represents a crucial area of inquiry in the development of economic policy. A multitude of researchers Hylland and Zeckhauser ([1979](https://arxiv.org/html/2410.14152v1#bib.bib20)); Shapley and Scarf ([1974](https://arxiv.org/html/2410.14152v1#bib.bib40)); Sönmez ([1999](https://arxiv.org/html/2410.14152v1#bib.bib43)); Abdulkadiroğlu and Sönmez ([1999](https://arxiv.org/html/2410.14152v1#bib.bib1)) conduct in-depth investigations into static matching models and propose Pareto efficient, individually rational, and strategy-proof simple mechanisms for various scenarios. Later, given the predominantly dynamic nature of practical scarce resource allocation, several studies Sönmez and Ünver ([2011](https://arxiv.org/html/2410.14152v1#bib.bib44)); Su and Zenios ([2004](https://arxiv.org/html/2410.14152v1#bib.bib45), [2005](https://arxiv.org/html/2410.14152v1#bib.bib46)); Bloch and Cantala ([2017](https://arxiv.org/html/2410.14152v1#bib.bib6)) begin to integrate dynamic properties into their models. The waitlist mechanism is widely adopted for allocating scarce resources. For instance, Chen et al. ([2018](https://arxiv.org/html/2410.14152v1#bib.bib11)) explore the waitlist mechanism in public housing allocation, Agarwal et al. ([2021](https://arxiv.org/html/2410.14152v1#bib.bib2)) design a waitlist for kidney organ allocation, which increased donor supply by 18.2% to enhance patient welfare; Lewis et al. ([2000](https://arxiv.org/html/2410.14152v1#bib.bib29)) develop a mechanism for managing waitlist for elective surgeries, improving the fairness of surgery opportunity distribution among patients. We focus our research on the k-deferrals waitlist mechanism in Chen et al. ([2018](https://arxiv.org/html/2410.14152v1#bib.bib11)).

Economic Policy Simulation. The predominant methodologies for simulating the implementation of economic policies include rule-driven and reinforcement learning-based simulations. Rule-driven approaches Holland and Miller ([1991](https://arxiv.org/html/2410.14152v1#bib.bib18)); Bonabeau ([2002](https://arxiv.org/html/2410.14152v1#bib.bib8)); Farmer and Foley ([2009](https://arxiv.org/html/2410.14152v1#bib.bib15)), involve modeling utility functions as agents, formulating various environmental rules, and observing the resultant global changes. Reinforcement learning-based simulations Laurent et al. ([2011](https://arxiv.org/html/2410.14152v1#bib.bib28)); Claus and Boutilier ([1998](https://arxiv.org/html/2410.14152v1#bib.bib12)); Zheng et al. ([2022](https://arxiv.org/html/2410.14152v1#bib.bib56)); Bansal et al. ([2018](https://arxiv.org/html/2410.14152v1#bib.bib5)); Jaderberg et al. ([2019](https://arxiv.org/html/2410.14152v1#bib.bib21)), involve modeling agents as either Markov Decision Processes (MDP) or Partially Observable Markov Decision Processes (POMDP), employing reinforcement learning methods to maximize the utility of each agent. However, both methods are heavily dependent on the assumption of individual rationality, often leading to predictions that diverge from real-world outcomes. While exploring the same problem, these methods presuppose that agents act rationally and calculate behavior by first optimizing equilibrium outcomes.

Our research introduces a novel approach by integrating social consensus derived from LLMs with the conventional concept of individual rationality to generate choices that reflect individual preferences. We optimize policy based on simulation results, providing a unique research perspective incomparable to existing methods.

LLM-based Agents. With the development of LLMs OpenAI ([2023](https://arxiv.org/html/2410.14152v1#bib.bib36)); Anil et al. ([2023](https://arxiv.org/html/2410.14152v1#bib.bib3)); Touvron et al. ([2023](https://arxiv.org/html/2410.14152v1#bib.bib47)), there has been an emerging focus on the exploration of LLMs-based agent architectures and prompt designs  Kojima et al. ([2022](https://arxiv.org/html/2410.14152v1#bib.bib26)); Shinn et al. ([2023](https://arxiv.org/html/2410.14152v1#bib.bib42)); Wang et al. ([2023c](https://arxiv.org/html/2410.14152v1#bib.bib53)), aimed at augmenting the capabilities of LLM-based agents to perform more complex tasks. Alongside these developments, researchers are devoted to crafting realistic societal simulation environments Li et al. ([2023a](https://arxiv.org/html/2410.14152v1#bib.bib30)); Park et al. ([2022](https://arxiv.org/html/2410.14152v1#bib.bib38), [2023](https://arxiv.org/html/2410.14152v1#bib.bib37)) that incorporate multiple agents, thereby providing a more dynamic and interconnected framework for agent interaction and behavior analysis. Following this,  (Li et al., [2023b](https://arxiv.org/html/2410.14152v1#bib.bib31)) makes a primary exploration of the effectiveness of LLMs-based agent simulation in macroeconomic policy. However, it remains unconfirmed whether the decision-making capabilities of LLMs align with those of actual humans in economic activities. Our investigation draws inspiration from the experimental findings of DillionDillion et al. ([2023](https://arxiv.org/html/2410.14152v1#bib.bib13)), aiming to integrate social consensus with individual rationality within the economic policy simulation process.

## 3 SRAP-Agent

In this section, we present SRAP-Agent, a novel large language model (LLM) agent-based simulation framework designed for examining the impact of economic policies on public scarce resource allocation. SRAP-Agent aims to consider that human decision-making in economic contexts is not purely rational, but is influenced by a mix of rationality and emotion, leading to unpredictable outcomes.

The simulation of SRAP-Agent is divided into three phases as shown in Fig. [1](https://arxiv.org/html/2410.14152v1#S1.F1 "Figure 1 ‣ 1 Introduction ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"): (1) Setup phase, involves the initialization of critical variables including the profiles of participants $P$, the information of available public scarce resources $R$, and the allocation policy $\pi$. (2) Simulation phase, dynamically simulates the allocation of resources $R$ among participants $P$, adhering to the constraints and rules defined by the allocation policy $\pi$. (3) Evaluation phase, focuses on assessing the outcomes of the allocation process, employing various metrics such as fairness and participant satisfaction.

Next, we outline the formulation of the allocation policy $\pi$ and the structure of participants $P$.

### 3.1 Allocation Policy

Public scarce resource allocation represents a significant challenge in economics, particularly in the context of public resources where the objectives of efficiency and equity frequently converge. In this work, we consider the queuing, pricing, and grouping mechanisms in SRAP-Agent. Formally, In the SRAP-Agent framework, we formalize a structure consisting of $m$ distinct queues: $q_{1},q_{2},\ldots q_{m}$. Each queue is uniquely characterized by:

(1) Participant entry conditions: to define the conditions under which individuals are eligible to enter a specific queue. Generally, the entry condition $E_{queue}(i)$ for the queue $q_{i}$ typically encompasses various socioeconomic factors such as individual budget, average income, and other relevant personal information. In the context of budget-based criteria, the entry conditions for $m$ queues are categorized into $m$ distinct budget ranges, organizing from high to low budget. This allows the simulation to reflect diverse economic backgrounds and their access to resources.

(2) Queue sorting strategies: to govern the prioritization of participants within each queue upon their entry, denoting as $S_{queue}$. We have incorporated two distinct sorting strategies: (a) first-in-first-out, of which participant order is set by arrival sequence. This method, prevalent in real scenarios, guarantees procedural fairness by chronologically allocating opportunities without regard to participants’ characteristics. (b) priority for vulnerable groups, which is designed to address social equity by providing preferential access to vulnerable groups, positioning them at the forefront when entering the specific queue. Besides, SRAP-Agent also employs the widely-used waiting queue Chen et al. ([2018](https://arxiv.org/html/2410.14152v1#bib.bib11)); Agarwal et al. ([2021](https://arxiv.org/html/2410.14152v1#bib.bib2)) mechanism. Each queue encompasses two components: the waiting queue and the selection queue. Participants initially enter the waiting queue. When a spot becomes available in the selection queue, the participant at the front of the waiting queue is transferred to the selection queue. The number of resources is denoted as $|R|$, then the capacity of the selection queue for accommodating participants is upper-bounded by $c\cdot|R|$. In the selection queue, each participant can queue to choose up to $k$ times; once their choices are exhausted, they return to the waiting queue. This mechanism diminishes the waiting time for participants by increasing the chances of participants staying in the selection queue.

(3) Available resource sub-sets: These sub-sets represent the specific resources that participants in a given queue can access and choose from, denoting as $R_{queue}(i)\subseteq R$ for the $q_{i}$. The composition of these resource sub-sets is dictated by certain predefined conditions, primarily considering factors such as the price and quality of resources. Typically, the predefined conditions of the resources in a queue are matched with their corresponding entry conditions. This ensures that resources are categorized and allocated to participant groups in a manner that is congruent with their economic capacity. Hence, the allocation policy is defined as the following equation:

|  | $V(p_{j})=\pi(p_{j}&#124;E_{queue},S_{queue},R_{queue},m),$ |  | (1) |

where $p_{j}$ represents the $j$-th participant in the list of participants $P=(p_{1},p_{2},\cdots,p_{n})$, the function $V(p_{j})$ is the available resource for the $j$-th participant under the policy. It enters the $q_{i}$ when it satisfies the condition $E_{queue}(i)$. Its order in the queue is determined by $S_{queue}$. When it is his turn to select, the participant $p_{j}$ can choose from the remaining resources in $R_{queue}(i)$, i.e., $V(p_{j})$. In SRAP-Agent, the policymakers can set the resource policy according to their requirement, and the policy will be broadcast to all participants.

### 3.2 LLM Agent-based Participant Simulation

Recognizing the complexity of human decision-making, SRAP-Agent employs LLM-based agents to simulate participants’ behaviors in the context of public scarce resource selection. This approach recognizes that human decision-making is not purely rational and often influenced by a range of factors. While it is impractical to simulate all aspects of human behavior, SRAP-Agent focuses on two most impactful behaviors: (1) Decision-making behavior. This aspect of the simulation addresses how participants evaluate and choose resources, factoring in elements like personal preferences, perceived value, and immediate needs. (2) Social behavior. Social dynamics also play a critical role in resource selection. This includes interactions with other participants, social norms, and external influences, which can affect how participants perceive and choose resources.

#### Agent Architecture

SRAP-Agent leverages LLM-based agents to instantiate individual participants in the simulation. Each participant $p_{j}$ is uniquely characterized by an initial profile including several key personal attributes such as economic status, income level, family background, and social network connections. To facilitate dynamic interaction and decision-making, each agent is equipped with a memory component $m_{j}$ describing the participant’s activity history. This memory serves as a critical reference point, enabling the agent to make informed decisions based on past actions and interactions within the simulation environment. This design not only enhances the realism of the simulation but also allows for a more nuanced exploration of social dynamics and individual decision-making processes.

#### Social Behavior Simulation

Human decision-making often involves the collection of information to make more informed choices. SRAP-Agent incorporates two primary social behaviors frequently employed by humans for information collection:

(1) Broadcasting: This mechanism is akin to a conventional web blog. Participants in SRAP-Agent utilize this platform for both disseminating and acquiring information, mirroring the way individuals interact and gather information in real-world social networks.

(2) Private messaging: In addition to broadcasting, participants can send private messages to their friends. This feature is critical for more direct and personalized communication, such as seeking assistance or sharing specific information about high-quality resources. In human society, communication is deeply influenced by psychological factors that shape the information being shared, resulting in asymmetrical information within social networks. To simulate the mental state of humans and the strategic interactions among different participants, we adopt a mechanism of memory assessment. We design the memory $m_{j}$ of $p_{j}$ to comprise two parts: (1) Trustworthy memory $m_{j}^{T}$ that mainly includes information from reliable sources, such as policymakers; (2) Suspicious memory $m_{j}^{S}$ that stores information from the social behaviors.

In the process of communication among participants, assume that $p_{j}$ is interacting with $p_{k}$, and the received information from $p_{k}$ will be stored in suspicious memory $m_{j}^{S}$ firstly. Then, $p_{j}$ measure the reliability of information from participant $p_{k}$ from two aspects: i) assessing the nature of their relationship with $p_{k}$, encompassing factors like closeness and historical interactions, in conjunction with a moral appraisal of $p_{k}$; ii) contrasting the received information against their trusted memory $m_{j}^{T}$. Following this evaluation, parts of the received information that are deemed reliable are integrated into the trustworthy memory $m_{j}^{T}$.

Through these mechanisms, SRAP-Agent effectively models the complex social dynamics of information gathering and sharing, allowing for a deeper understanding of human behavior in the context of public scarce resource allocation.

#### Decision-Making Behavior Simulation

The simulation of the decision-making processes in SRAP-Agent can be formulated by:

|  | $R^{*}_{j}=D(p_{j},V(p_{j})),$ |  | (2) |

where the function $D(p_{j},V(p_{j}))$ indicates the process that $p_{j}$ selects the desired resource from his visible resource pool. $R^{*}_{j}=\emptyset$ means participant $p_{j}$ quit the decision procedure when there’s no desired resource.

## 4 POA: Policy Optimization Algorithm

Algorithm 1 POA: Policy Optimization Algorithm

1:Historical policies $\Pi_{h}$, predictor $\widetilde{f}$, running iterations $M$2:Optimized policy $\pi^{*}$3:Randomly generate policies $\Pi_{r}$.4:Initialize policy pool $\Pi$ with $\Pi_{h}$ and $\Pi_{r}$.5:for $i=1$ to $M$ do6:     Calculate the fitness $\widetilde{f}(\pi)$ of $\pi$ in $\Pi$.7:     Use the tournament selection Miller et al. ([1995](https://arxiv.org/html/2410.14152v1#bib.bib33)) to select policies $\Pi_{s}$ in $\Pi$ based on fitness.8:     Combine pairs in $\Pi_{s}$ to produce $\Pi_{c}$.9:     Apply mutation to the $\Pi_{c}$ to obtain $\Pi_{m}$.10:     Replace $\Pi_{s}$ with $\Pi_{m}$ to update $\Pi$.11:end for12:Obtain the optimized policy by $\widetilde{f}$:

|  | $v_{\pi^{*}}=\underset{v_{\pi}}{\arg\max}\widetilde{f}\left(v_{\pi}\right),\pi\in\Pi$ |  |

13:Decode $v_{\pi^{*}}$ to obtain the optimized policy $\pi^{*}$.

We propose a policy optimization algorithm (POA) based on the genetic algorithm Lambora et al. ([2019](https://arxiv.org/html/2410.14152v1#bib.bib27)) to find more reasonable policies. The set of policies proposed by the policymaker is $\pi\in\Pi$, and $f$ is the policy evaluation metric, then the optimal policy is $\pi^{*}=\underset{\pi\in\Pi}{\arg\max}f(\pi).$ Generally, $f$ takes into account $L$ different kinds of policy evaluation metrics ($f_{j}(\pi),j\in[L]$), which can be categorized into societal satisfaction and societal fairness metrics. We pre-set the weights $w_{j}$ to alter the optimization objective of POA, the specific calculation formula of $f$ is as follows:

|  | $f(\pi)=\sum_{j=1}^{n}w_{j}\cdot f_{j}(\pi),\pi\in\Pi_{h}.$ |  | (3) |

Due to the excessive number of policies that need to be searched, the time required to obtain policy evaluation results with SRAP-Agent may be excessively long. So we use the $v_{\pi},f(\pi)$ dataset to train a predictor $\widetilde{f}$ for the estimation of the policy evaluation result: $f(\pi)=\widetilde{f}(v_{\pi})$. POA then performs $M$ iterations to find more rational policy parameters. The specific steps of POA are listed in Algorithm.[1](https://arxiv.org/html/2410.14152v1#alg1 "Algorithm 1 ‣ 4 POA: Policy Optimization Algorithm ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent").

## 5 Experiment

We select the public housing allocation scenario as a typical case to validate the simulation effectiveness of SRAP-Agent. We conduct experiments in three steps: 1\. We conduct the Turing test to ensure that, SRAP-Agent can effectively simulate the policy execution process of various policies proposed by the policymaker. 2\. Based on the simulation results obtained from the SRAP-Agent, we analyze the impact of various policy parameters on the allocation of public scarce resources. 3\. Subsequently, we employ the POA algorithm to find the optimal policy in pursuit of specific optimization objectives.

### 5.1 Evaluation Metrics

In our research, we adhere to the commonly used metrics for public scarce resources allocation Vermunt and Törnblom ([1996](https://arxiv.org/html/2410.14152v1#bib.bib50)); Wang et al. ([2023b](https://arxiv.org/html/2410.14152v1#bib.bib52)), employing two major categories of policy evaluation metrics:

Societal Satisfaction Metrics. We use three metrics to evaluate societal satisfaction: (1) Avg $r^{size}$: representing the average per-capita living area size of house resources for all participants; (2) Avg $WT$: Average waiting time for each participant; (3) $SW$: the social welfare, quantifying the cumulative satisfaction of all participants.

Societal Fairness Metrics. In addition, we employ four metrics to evaluate societal fairness: (1) Var $r^{size}$: denoting the variance in per capita living area size among participants; (2) Rop: indicating the number of inverse order pairs in house allocation results; (3) co-Gini: the Gini coefficient Wang et al. ([2023b](https://arxiv.org/html/2410.14152v1#bib.bib52)) calculated on house allocation result. We use $F(V,NV)$ to reflect the $SW$ gap between the vulnerable ($V$) and non-vulnerable group ($NV$) of participants. The calculation equations are listed in Appendix [C.2](https://arxiv.org/html/2410.14152v1#A3.SS2 "C.2 Evaluation Metrics ‣ Appendix C Experiment Setup ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent").

### 5.2 Turing Test

Table 1: Turing test for responses from GPT-3.5 and GPT-4\. *means statistically significant with $p\textless$ 0.05.

 |  | GPT-3.5 | GPT-4 |
| --- | --- | --- |
| $Human>Robot$ | 26.6% | 20.6%^∗ |
| $Human=Robot$ | 36.3% | 21.1%^∗ |
| $Human<Robot$ | 30.2% | 49.1%^∗ |
| $None$ | 7.0% | 9.3%^∗ | 

To assess the simulation capabilities of LLMs, the Turing test Turing ([2009](https://arxiv.org/html/2410.14152v1#bib.bib49)) is a commonly employed and effective metric Ng et al. ([2024](https://arxiv.org/html/2410.14152v1#bib.bib34)); Jones and Bergen ([2023](https://arxiv.org/html/2410.14152v1#bib.bib23)); Jannai et al. ([2023](https://arxiv.org/html/2410.14152v1#bib.bib22)). Given that SRAP-Agent is built on human behavior simulation through LLM-based agents, we design a Turing test  Turing ([1950](https://arxiv.org/html/2410.14152v1#bib.bib48)) to validate the system’s capability to realistically simulate the policy execution process. We utilize GPT-3.5 and GPT-4 OpenAI ([2023](https://arxiv.org/html/2410.14152v1#bib.bib36)) to build agents. Subsequently, we recruit a different group of human annotators to make paired comparisons of rationality for LLM responses and human responses. They choose one of these rationality labels: (1) human response is more rational than LLM ($Human>Robot$) (2) human response is less rational than LLM ($Human<Robot$) (3) both responses are rational ($Human=Robot$) (4) neither is rational ($None$).

#### Result Analysis

Table [1](https://arxiv.org/html/2410.14152v1#S5.T1 "Table 1 ‣ 5.2 Turing Test ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent") shows the comparison of the rationality of responses for GPT-3.5, GPT4, and humans. We can see that: (1) The comparison between human responses and those from GPT-3.5 indicates a majority view that humans and agents perform similarly in rationality, with nearly equal proportions of judging human superiority ($Human>Robot$) and inferiority ($Human<Robot$) to robotic responses. This suggests a parity in rational decision-making capabilities between humans and GPT-3.5, albeit with a marginal preference for the latter. (2) In contrast, responses involving GPT-4 significantly outperform human counterparts in terms of rationality, as evidenced by a higher incidence of $Human<Robot$ compared to $Human>Robot$. This discrepancy underscores GPT-4’s ability to generate more strategic decisions, such as changing plans and waiting for future opportunities (see the case study in Appendix [D](https://arxiv.org/html/2410.14152v1#A4 "Appendix D Different LLM for Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")). Specifically, GPT-4 demonstrates proficiency in proposing adaptive solutions and exploiting policy nuances to gain strategic advantages, a sophistication not commonly found in human or GPT-3.5 responses, which tend to focus on immediate factors like budget constraints and personal preferences. The findings suggest that GPT-3.5 aligns closely with the decision-making processes of the average individual in public scarce resource allocation. Consequently, we adopt the GPT-3.5-turbo-0301 model as our backbone LLM.

Table 2: Comparison experiments on different combinations of entry condition and resource sub-sets.

 | $E_{queue}$ | $R_{queue}$ | Satisfaction | Fairness |
| --- | --- | --- | --- |
| Avg $r^{size}\uparrow$ | Avg $WT$ $\downarrow$ | $SW$ $\uparrow$ | Var $r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- |
|  | $r^{random}$ | 2.5[±0.8] | 6.1[±0.3] | 83.8[±32.0] | 55.7[±18.6] | 73.5[±24.5] | 0.9[±0.0] |
| $p^{select}$ | $r^{rent}$ | 7.1[±0.1] | 4.8[±0.0] | 244.6[±3.4] | 141.4[±3.2] | 185.0[±7.0] | 0.7[±0.0] |
|  | $r^{size}$ | 13.8[±0.4] | 3.1[±0.5] | 419.9[±0.3] | 229.1[±22.8] | 327.0[±26.0] | 0.5[±0.0] |
|  | $r^{random}$ | 10.8[±0.1] | 4.6[±0.0] | 313.1[±5.7] | 226.7[±8.1] | 366.5[±19.5] | 0.6[±0.0] |
| $p^{family}$ | $r^{rent}$ | 11.5[±0.0] | 3.6[±0.0] | 410.0[±3.3] | 159.3[±0.5] | 251.5[±1.5] | 0.5[±0.0] |
|  | $r^{size}$ | 11.7[±0.3] | 3.8[±0.1] | 410.4[±2.8] | 159.9[±9.4] | 254.5[±23.5] | 0.5[±0.0] |
|  | $r^{random}$ | 10.0[±0.0] | 4.4[±0.1] | 313.2[±11.2] | 185.3[±15.6] | 300.0[±10.0] | 0.6[±0.0] |
| $p^{rent}$ | $r^{rent}$ | 11.3[±0.2] | 3.9[±0.0] | 402.7[±1.4] | 163.7[±7.4] | 246.5[±4.5] | 0.5[±0.0] |
|  | $r^{size}$ | 12.0[±0.2] | 3.9[±0.1] | 389.8[±0.8] | 195.1[±9.0] | 276.0[±31.0] | 0.5[±0.0] | 

Table 3: Comparison of optimized policies $\pi^{*}$ against $\pi_{KM}$ on policy evaluation metrics.

 | $\pi$ | $m$ | $E_{queue}$ | $S_{queue}$ | $R_{queue}$ | Satisfaction | Fairness |
|  |  |  | Sort | $k$ | $c$ |  | Avg $r^{size}\uparrow$ | Avg $WT$ $\downarrow$ | $SW$ $\uparrow$ | Var $r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini |
| $\pi_{s}^{*}$ | 3 | $p^{select}$ | FIFO | 4 | 4 | $r^{size}$ | 16.3[±0.5] | 1.9[±0.0] | 427.6[±12.7] | 202.3[±12.9] | 221.5[±19.5] | 0.4[±0.0] |
| $\pi_{f}^{*}$ | 3 | $p^{select}$ | VFA | 3 | 3 | $r^{size}$ | 16.2[±0.4] | 1.9[±0.0] | 425.1[±11.9] | 202.6[±6.6] | 193.5[±32.5] | 0.4[±0.0] |
| $\pi_{KM}$ | - | - | - | - | - | - | 16.0 | - | 485.9 | 275.7 | 511 | 0.46 |
| $\pi_{S}$ | 1 | $p^{random}$ | FIFO | - | 3 | $r^{random}$ | 14.70[±0.63] | 1.89[±0.63] | 420.1[±1.9] | 202.5[±17.1] | 223[±0.5] | 0.4[±0.0] |
| $\pi_{B}$ | 3 | $p^{select}$ | FIFO | - | 2 | $r^{size}$ | 15.36[±0.05] | 1.56[±0.1] | 409.1[±2.4] | 270.8[±3.0] | 414.5[±0.5] | 0.5[±0.0] |
| $\pi_{H}$ | 1 | $p^{random}$ | VFR | 2 | 3 | $r^{random}$ | 11.61[±0.38] | 3.54[±0.03] | 392.1[±19.6] | 182.5[±3.18] | 275[±0.00] | 0.5[±0.0] | 

### 5.3 Simulation-based Policy Analysis

In the ablation experiments for the policy, we conduct comparison experiments on each policy parameter in Equation [1](https://arxiv.org/html/2410.14152v1#S3.E1 "In 3.1 Allocation Policy ‣ 3 SRAP-Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"). We conduct experiments on two policy factors: (1) resource and participant grouping: combinations of different $E_{queue}$ and $R_{queue}$; (2) different queue sorting strategies. We also simulate the phenomenon of allocation conflict in the real-world, with detailed information in the Appendix. [F.4](https://arxiv.org/html/2410.14152v1#A6.SS4 "F.4 Case Study on Housing Quality ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent").

#### Resource and Participant Grouping

We compare the policies formed by combinations of different $E_{queue}$ and $R_{queue}$. In $E_{queue}$, participants primarily enter queues in three ways: based on budget ($p^{budget}$), based on number of family members ($p^{family}$), and based on self-selected queue ($p^{select}$). Corresponding to the entry conditions for participants, there are three methods of categorizing resource sub-sets: based on rental costs ($r^{rent}$), based on house size ($r^{size}$), and based on random categorization ($r^{random}$). Table [2](https://arxiv.org/html/2410.14152v1#S5.T2 "Table 2 ‣ Result Analysis ‣ 5.2 Turing Test ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent") shows the performance of different policy combinations of $E_{queue}$ and $R_{queue}$ across various policy evaluation metrics. We can observe that:

#### Queue Sorting Strategies

Regarding queue sorting policy $S_{queue}$. Firstly, we compare two sorting methods: whether to give priority to vulnerable groups or not (FIFO). We adopt two methods of designating vulnerable groups based on rent budget: sort all participants at first (VFA) and sort the participants in each round (VFR), with the bottom 20% designated as vulnerable groups. As shown in Fig. [2a](https://arxiv.org/html/2410.14152v1#S5.F2.sf1 "In Figure 2 ‣ Queue Sorting Strategies ‣ 5.3 Simulation-based Policy Analysis ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), prioritizing vulnerable groups can reduce the satisfaction gap between vulnerable and non-vulnerable groups ($\Delta F(W,G)>0$), as compared to the FIFO method. No marked superiority is evident between the VFA and VFR methods, which are influenced by the stochastic nature of participant arrival patterns.

![Refer to caption](img/8305325f185b11627de741d6419464b8.png)

(a) sorting methods

![Refer to caption](img/a8a4217cf5081e11a1b3d692e9a1f566.png)

(b) waiting queue

Figure 2: Comparison of queue sorting methods.

Secondly, we compare the impact of the $k$ and $c$ parameters in the waitlist mechanism. This mechanism is primarily designed to diminish the waiting time for participants by increasing the chances of participants staying in the selection queue. As illustrated in Fig. [2b](https://arxiv.org/html/2410.14152v1#S5.F2.sf2 "In Figure 2 ‣ Queue Sorting Strategies ‣ 5.3 Simulation-based Policy Analysis ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), increases in $k$ and the $c$ for the k-waitlist effectively reduce Avg $WT$, corroborating our initial expectation.

![Refer to caption](img/12dc062764a02d9b312412c53cb660c8.png)

(a) Max’s deceptive plan for broadcasting information.

![Refer to caption](img/90e6710f0d2199707be21632e46fdeb9.png)

(b) Sarah’s plan for requesting help from her friends.

Figure 3: Deceptive behavior and cooperation behavior. Max’s conservative personality and lack of trustworthy acquaintances lead him to conceal his true intentions when sharing information. In contrast, Sarah seeks advice on choosing a house from her friends and openly shares her housing preferences.

### 5.4 Finding the Optimal Allocation Policy

To evaluate the effectiveness of POA for policy parameter optimization, we select the following baseline policies: (1) On the single $SW$ metric, a solvable optimal policy exists in the expected sense. We use the Bipartite Graph Matching algorithm, specifically the Kuhn-Munkres algorithm Zhu et al. ([2016](https://arxiv.org/html/2410.14152v1#bib.bib57)) to find this policy $\pi_{KM}$. (2) Additionally, we select three policies of scarce resource allocation from real human societies ($\pi_{S}$,$\pi_{B}$,$\pi_{H}$; detailed information is listed in Appendix. [G.2](https://arxiv.org/html/2410.14152v1#A7.SS2 "G.2 Comparison with Other Policies ‣ Appendix G POA ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")).

#### Result Analysis

POA separately optimizes policies against two pre-set optimization objectives: the optimized policy $\pi_{s}^{*}$ prefers societal satisfaction, and the optimized policy $\pi_{f}^{*}$ prefers societal fairness. The metric weights pre-set for optimizing these two objectives are delineated in Table [15](https://arxiv.org/html/2410.14152v1#A7.T15 "Table 15 ‣ G.1 Hyper-parameters for POA ‣ Appendix G POA ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent").

![Refer to caption](img/3ec315a2dc62d5d036afd76e4ae2828a.png)

Figure 4: The average $f(\pi)$ for optimized policies with respect to the iteration number.

Fig. [4](https://arxiv.org/html/2410.14152v1#S5.F4 "Figure 4 ‣ Result Analysis ‣ 5.4 Finding the Optimal Allocation Policy ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent") demonstrates the progression of the policy optimized by POA in terms of the policy evaluation metric $f(\pi)$, as the number of iterations increases. The POA algorithm can generate robust policies after several iterations, and $f(\pi)$ improves by 20% after 50 iterations.

Results of evaluation metrics for optimized policies are listed in Table [3](https://arxiv.org/html/2410.14152v1#S5.T3 "Table 3 ‣ Result Analysis ‣ 5.2 Turing Test ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"). (1) For real-world policies: $\pi_{H}$ prioritizes societal fairness while $\pi_{B}$ prioritizes societal satisfaction. $\pi_{S}$ performs the best in achieving a balance in societal fairness and satisfaction metrics. (2) Compared with the existing real-world policies, $\pi_{s}^{*}$ enhances the SW metric by 1.8%, $\pi_{f}^{*}$ enhances the societal fairness metric by 13.2%. (3) Although $\pi_{KM}$ surpasses $\pi^{*}$ in the $SW$ metric. However, $\pi_{KM}$ exhibits an imbalanced emphasis on dominant groups of participants. This is reflected by poor performance in societal fairness metrics, with Rop reaching $511$. POA considers a diverse combination of metrics during policy optimization, thus enabling the equilibrium of all evaluation metrics while optimizing the target.

### 5.5 Case Study

SRAP-Agent simulates the cognitive, socio-affective, and emotional factors of participants to simulate human interaction. We find two predominant types of communication patterns emerge: deceptive behavior and cooperation behavior.

#### Deceptive Behavior

Fig. [3a](https://arxiv.org/html/2410.14152v1#S5.F3.sf1 "In Figure 3 ‣ Queue Sorting Strategies ‣ 5.3 Simulation-based Policy Analysis ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent") illustrates a case where deceptive behavior occurs. In the figure, Max is a conservative individual with a limited number of trustworthy friends and opts to conceal the true information when broadcasting. Rather than disclosing his actual preferences, he accentuates the positive aspects of uninterested resources to enhance his chances of securing his interested choice. Such behaviors effectively emulate the formation of misinformation in policy execution.

#### Cooperation Behavior

In contrast to deceptive behaviors, cooperation behaviors also exist and constitute the majority. As shown in Fig. [3b](https://arxiv.org/html/2410.14152v1#S5.F3.sf2 "In Figure 3 ‣ Queue Sorting Strategies ‣ 5.3 Simulation-based Policy Analysis ‣ 5 Experiment ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), Sarah is a relatively astute person, who has two colleagues and a friend. She chooses to honestly post her needs and expresses her desire for a family-friendly house by broadcasting. It is observable that Sarah modifies her socialization strategies by assessing relationships, thereby maximizing expected benefits through cooperation. Traditional economic simulation models often overlook the complexities of emotional factors. In contrast, SRAP-Agent incorporates these dynamics, resulting in more accurate and realistic policy simulation outcomes.

## 6 Conclusion

In this paper, we introduce a novel SRAP-Agent framework to accurately simulate the policy execution process in public scarce resources allocation. The framework establishes resource allocation queues to regulate the organization and prioritization of participants and resources. We employ LLM-based agents to accurately mimic human decision-making processes, which are influenced by a mix of rationality and emotional factors. To facilitate efficient policy optimization, we propose the POA algorithm based on genetic algorithms. Finally, we validate the framework’s authenticity and effectiveness through experiments in the context of public housing allocation. The progress in AI technologies significantly reduces costs in the simulation of economic policies, and SRAP-Agent represents a promising step forward in this field.

## Limitations

This paper acknowledges several limitations that future research could address:

LLMs in SRAP-Agent is not customized for policy execution simulation. This work doesn’t include training or fine-tuning the LLMs for specific tasks. Instead, we construct an LLM-based agent through specially designed prompts and modular designs. This approach may result in variability in the performance of SRAP-Agent across different LLMs, leading to potentially non-robust outcomes.

The limitation of policy evaluation metrics. We acknowledge the limitations of manually defined policy evaluation metrics. Conducting societal simulation experiments could provide more reliable and comprehensive assessments of policy simulation outcomes. However, due to the vast number and scale of policies, we cannot afford the significant manpower and time costs. According to recent studies Li et al. ([2023a](https://arxiv.org/html/2410.14152v1#bib.bib30)), we believe that utilizing LLMs for policy evaluation is a reasonable and efficient choice.

## Ethics Statement

This work fully complies with the ACL Ethics Policy. We declare that there are no ethical issues in this paper, to the best of our knowledge.

## References

*   Abdulkadiroğlu and Sönmez (1999) Atila Abdulkadiroğlu and Tayfun Sönmez. 1999. House allocation with existing tenants. *Journal of Economic Theory*, 88(2):233–260.
*   Agarwal et al. (2021) Nikhil Agarwal, Itai Ashlagi, Michael A Rees, Paulo Somaini, and Daniel Waldinger. 2021. Equilibrium allocations under alternative waitlist designs: Evidence from deceased donor kidneys. *Econometrica*, 89(1):37–76.
*   Anil et al. (2023) Rohan Anil, Andrew M. Dai, Orhan Firat, Melvin Johnson, Dmitry Lepikhin, Alexandre Passos, Siamak Shakeri, Emanuel Taropa, Paige Bailey, Zhifeng Chen, Eric Chu, Jonathan H. Clark, Laurent El Shafey, Yanping Huang, Kathy Meier-Hellstern, Gaurav Mishra, Erica Moreira, Mark Omernick, Kevin Robinson, Sebastian Ruder, Yi Tay, Kefan Xiao, Yuanzhong Xu, Yujing Zhang, Gustavo Hernandez Abrego, Junwhan Ahn, Jacob Austin, Paul Barham, Jan Botha, James Bradbury, Siddhartha Brahma, Kevin Brooks, Michele Catasta, Yong Cheng, Colin Cherry, Christopher A. Choquette-Choo, Aakanksha Chowdhery, Clément Crepy, Shachi Dave, Mostafa Dehghani, Sunipa Dev, Jacob Devlin, Mark Díaz, Nan Du, Ethan Dyer, Vlad Feinberg, Fangxiaoyu Feng, Vlad Fienber, Markus Freitag, Xavier Garcia, Sebastian Gehrmann, Lucas Gonzalez, Guy Gur-Ari, Steven Hand, Hadi Hashemi, Le Hou, Joshua Howland, Andrea Hu, Jeffrey Hui, Jeremy Hurwitz, Michael Isard, Abe Ittycheriah, Matthew Jagielski, Wenhao Jia, Kathleen Kenealy, Maxim Krikun, Sneha Kudugunta, Chang Lan, Katherine Lee, Benjamin Lee, Eric Li, Music Li, Wei Li, YaGuang Li, Jian Li, Hyeontaek Lim, Hanzhao Lin, Zhongtao Liu, Frederick Liu, Marcello Maggioni, Aroma Mahendru, Joshua Maynez, Vedant Misra, Maysam Moussalem, Zachary Nado, John Nham, Eric Ni, Andrew Nystrom, Alicia Parrish, Marie Pellat, Martin Polacek, Alex Polozov, Reiner Pope, Siyuan Qiao, Emily Reif, Bryan Richter, Parker Riley, Alex Castro Ros, Aurko Roy, Brennan Saeta, Rajkumar Samuel, Renee Shelby, Ambrose Slone, Daniel Smilkov, David R. So, Daniel Sohn, Simon Tokumine, Dasha Valter, Vijay Vasudevan, Kiran Vodrahalli, Xuezhi Wang, Pidong Wang, Zirui Wang, Tao Wang, John Wieting, Yuhuai Wu, Kelvin Xu, Yunhan Xu, Linting Xue, Pengcheng Yin, Jiahui Yu, Qiao Zhang, Steven Zheng, Ce Zheng, Weikang Zhou, Denny Zhou, Slav Petrov, and Yonghui Wu. 2023. [Palm 2 technical report](http://arxiv.org/abs/2305.10403).
*   Banerjee and Somanathan (2007) Abhijit Banerjee and Rohini Somanathan. 2007. [The political economy of public goods: Some evidence from india](https://doi.org/https://doi.org/10.1016/j.jdeveco.2006.04.005). *Journal of Development Economics*, 82(2):287–314.
*   Bansal et al. (2018) Trapit Bansal, Jakub Pachocki, Szymon Sidor, Ilya Sutskever, and Igor Mordatch. 2018. [Emergent complexity via multi-agent competition](https://openreview.net/forum?id=Sy0GnUxCb). ICLR ’18.
*   Bloch and Cantala (2017) Francis Bloch and David Cantala. 2017. Dynamic assignment of objects to queuing agents. *American Economic Journal: Microeconomics*, 9(1):88–122.
*   Blümel et al. (1986) Wolfgang Blümel, Rüdiger Pethig, and Oskar von dem Hagen. 1986. [The theory of public goods : A survey of recent issues](http://www.jstor.org/stable/40750871). *Journal of Institutional and Theoretical Economics (JITE) / Zeitschrift für die gesamte Staatswissenschaft*, (2):241–309.
*   Bonabeau (2002) Eric Bonabeau. 2002. [Agent-based modeling: Methods and techniques for simulating human systems](https://doi.org/10.1073/pnas.082080899). *Proceedings of the National Academy of Sciences of the United States of America*, 99 Suppl 3:7280–7.
*   BROCK (1980) WILLIAM A. BROCK. 1980. [The design of mechanisms for efficient allocation of public goods](https://doi.org/https://doi.org/10.1016/B978-1-4832-2792-4.50009-8). In L.R. KLEIN, M. NERLOVE, and S.C. TSIANG, editors, *Quantitative Economics and Development*, pages 45–80\. Academic Press.
*   Brown et al. (2020) Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, et al. 2020. Language models are few-shot learners. *Advances in neural information processing systems*, 33:1877–1901.
*   Chen et al. (2018) Zhou Chen, Qi Qi, Changjun Wang, and Wenwei Wang. 2018. *Available at SSRN 3203280*.
*   Claus and Boutilier (1998) Caroline Claus and Craig Boutilier. 1998. The dynamics of reinforcement learning in cooperative multiagent systems. AAAI ’98/IAAI ’98, page 746–752.
*   Dillion et al. (2023) Danica Dillion, Niket Tandon, Yuling Gu, and Kurt Gray. 2023. [Can ai language models replace human participants?](https://doi.org/https://doi.org/10.1016/j.tics.2023.04.008) *Trends in Cognitive Sciences*, 27(7):597–600.
*   Falkinger et al. (2000) Josef Falkinger, Ernst Fehr, Simon Gächter, and Rudolf Winter-Ember. 2000. [A simple mechanism for the efficient provision of public goods: Experimental evidence](https://doi.org/10.1257/aer.90.1.247). *American Economic Review*, 90(1):247–264.
*   Farmer and Foley (2009) J. Doyne Farmer and Duncan K. Foley. 2009. [The economy needs agent-based modelling](https://api.semanticscholar.org/CorpusID:37676798). *Nature*, 460:685–686.
*   Fu et al. (2020) Zuohui Fu, Yikun Xian, Ruoyuan Gao, Jieyu Zhao, Qiaoying Huang, Yingqiang Ge, Shuyuan Xu, Shijie Geng, Chirag Shah, Yongfeng Zhang, et al. 2020. Fairness-aware explainable recommendation over knowledge graphs. SIGIR ’20, pages 69–78.
*   Groves and Ledyard (1974) Theodore Groves and John Ledyard. 1974. An incentive mechanism for efficient resource allocation in general equilibrium with public goods. Technical report, Discussion Paper.
*   Holland and Miller (1991) John H. Holland and John H. Miller. 1991. [Artificial adaptive agents in economic theory](http://www.jstor.org/stable/2006886). *The American Economic Review*, 81(2):365–370.
*   Huang et al. (2015) Zhonghua Huang, Xuejun Du, and Xiaofen Yu. 2015. [Home ownership and residential satisfaction: Evidence from hangzhou, china](https://doi.org/https://doi.org/10.1016/j.habitatint.2015.05.008). *Habitat International*, 49:74–83.
*   Hylland and Zeckhauser (1979) Aanund Hylland and Richard Zeckhauser. 1979. The efficient allocation of individuals to positions. *Journal of Political economy*, 87(2):293–314.
*   Jaderberg et al. (2019) Max Jaderberg, Wojciech M. Czarnecki, Iain Dunning, Luke Marris, Guy Lever, Antonio Garcia Castañeda, Charles Beattie, Neil C. Rabinowitz, Ari S. Morcos, Avraham Ruderman, Nicolas Sonnerat, Tim Green, Louise Deason, Joel Z. Leibo, David Silver, Demis Hassabis, Koray Kavukcuoglu, and Thore Graepel. 2019. [Human-level performance in 3d multiplayer games with population-based reinforcement learning](https://doi.org/10.1126/science.aau6249). *Science*, 364(6443):859–865.
*   Jannai et al. (2023) Daniel Jannai, Amos Meron, Barak Lenz, Yoav Levine, and Yoav Shoham. 2023. Human or not? a gamified approach to the turing test. *arXiv preprint arXiv:2305.20010*.
*   Jones and Bergen (2023) Cameron Jones and Benjamin Bergen. 2023. Does gpt-4 pass the turing test? *arXiv preprint arXiv:2310.20216*.
*   Jr et al. (1977) Theodore Jr, Theodore Groves, and John Ledyard. 1977. [Optimal allocation of public goods: A solution to the "free rider" problem](https://doi.org/10.2307/1912672). *Econometrica*, 45:783–809.
*   Kim (2015) Tae Kyun Kim. 2015. T test as a parametric statistic. *Korean journal of anesthesiology*, 68(6):540.
*   Kojima et al. (2022) Takeshi Kojima, Shixiang (Shane) Gu, Machel Reid, Yutaka Matsuo, and Yusuke Iwasawa. 2022. [Large language models are zero-shot reasoners](https://proceedings.neurips.cc/paper_files/paper/2022/file/8bb0d291acd4acf06ef112099c16f326-Paper-Conference.pdf). NIPS ’22, pages 22199–22213\. Curran Associates, Inc.
*   Lambora et al. (2019) Annu Lambora, Kunal Gupta, and Kriti Chopra. 2019. Genetic algorithm-a literature review. In *2019 international conference on machine learning, big data, cloud and parallel computing (COMITCon)*, pages 380–384\. IEEE.
*   Laurent et al. (2011) Guillaume J. Laurent, Laëtitia Matignon, and N. Le Fort-Piat. 2011. The world of independent learners is not markovian. *Int. J. Know.-Based Intell. Eng. Syst.*, 15(1):55–64.
*   Lewis et al. (2000) Steven Lewis, Morris L Barer, Claudia Sanmartin, Sam Sheps, Samuel ED Shortt, and Paul W McDonald. 2000. Ending waiting-list mismanagement: principles and practice. *Cmaj*, 162(9):1297–1300.
*   Li et al. (2023a) Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, and Bernard Ghanem. 2023a. Camel: Communicative agents for "mind" exploration of large language model society. NIPS ’23.
*   Li et al. (2023b) Nian Li, Chen Gao, Yong Li, and Qingmin Liao. 2023b. Large language model-empowered agents for simulating macroeconomic activities. *arXiv preprint arXiv:2310.10436*.
*   Mankiw (2011) N.G. Mankiw. 2011. [*Principles of Economics, 5th edition*](http://mankiw.swlearning.com/). South-Western Cengage Learning. The Introductory-Level Textbook.
*   Miller et al. (1995) Brad L Miller, David E Goldberg, et al. 1995. Genetic algorithms, tournament selection, and the effects of noise. *Complex systems*, 9(3):193–212.
*   Ng et al. (2024) Man Tik Ng, Hui Tung Tse, Jen-tse Huang, Jingjing Li, Wenxuan Wang, and Michael R Lyu. 2024. How well can llms echo us? evaluating ai chatbots’ role-play ability with echo. *arXiv preprint arXiv:2404.13957*.
*   Nold (1992) Patricia Ann Nold. 1992. [Public choice and the allocation of public goods: An empirical analysis of local school expenditures](http://www.jstor.org/stable/2117444). *The American Economic Review*, 82(2):457–463.
*   OpenAI (2023) OpenAI. 2023. [Gpt-4 technical report](http://arxiv.org/abs/2303.08774).
*   Park et al. (2023) Joon Sung Park, Joseph O’Brien, Carrie Jun Cai, Meredith Ringel Morris, Percy Liang, and Michael S. Bernstein. 2023. [Generative agents: Interactive simulacra of human behavior](https://doi.org/10.1145/3586183.3606763). UIST ’23.
*   Park et al. (2022) Joon Sung Park, Lindsay Popowski, Carrie Cai, Meredith Ringel Morris, Percy Liang, and Michael S. Bernstein. 2022. [Social simulacra: Creating populated prototypes for social computing systems](https://doi.org/10.1145/3526113.3545616). UIST ’22.
*   Qian et al. (2023) Chen Qian, Xin Cong, Cheng Yang, Weize Chen, Yusheng Su, Juyuan Xu, Zhiyuan Liu, and Maosong Sun. 2023. Communicative agents for software development. *arXiv preprint arXiv:2307.07924*.
*   Shapley and Scarf (1974) Lloyd Shapley and Herbert Scarf. 1974. On cores and indivisibility. *Journal of mathematical economics*, 1(1):23–37.
*   Shen (2015) Yang Shen. 2015. Why does the government fail to improve the living conditions of migrant workers in s hanghai? reflections on the policies and the implementations of public rental housing under neoliberalism. *Asia & the Pacific Policy Studies*, 2(1):58–74.
*   Shinn et al. (2023) Noah Shinn, Federico Cassano, Edward Berman, Ashwin Gopinath, Karthik Narasimhan, and Shunyu Yao. 2023. Reflexion: Language agents with verbal reinforcement learning. *arXiv preprint arXiv:2303.11366*.
*   Sönmez (1999) Tayfun Sönmez. 1999. Strategy-proofness and essentially single-valued cores. *Econometrica*, 67(3):677–689.
*   Sönmez and Ünver (2011) Tayfun Sönmez and M Utku Ünver. 2011. Matching, allocation, and exchange of discrete resources. In *Handbook of social Economics*, volume 1, pages 781–852\. Elsevier.
*   Su and Zenios (2004) Xuanming Su and Stefanos Zenios. 2004. Patient choice in kidney allocation: The role of the queueing discipline. *Manufacturing & Service Operations Management*, 6(4):280–301.
*   Su and Zenios (2005) Xuanming Su and Stefanos A Zenios. 2005. Patient choice in kidney allocation: A sequential stochastic assignment model. *Operations research*, 53(3):443–455.
*   Touvron et al. (2023) Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, Dan Bikel, Lukas Blecher, Cristian Canton Ferrer, Moya Chen, Guillem Cucurull, David Esiobu, Jude Fernandes, Jeremy Fu, Wenyin Fu, Brian Fuller, Cynthia Gao, Vedanuj Goswami, Naman Goyal, Anthony Hartshorn, Saghar Hosseini, Rui Hou, Hakan Inan, Marcin Kardas, Viktor Kerkez, Madian Khabsa, Isabel Kloumann, Artem Korenev, Punit Singh Koura, Marie-Anne Lachaux, Thibaut Lavril, Jenya Lee, Diana Liskovich, Yinghai Lu, Yuning Mao, Xavier Martinet, Todor Mihaylov, Pushkar Mishra, Igor Molybog, Yixin Nie, Andrew Poulton, Jeremy Reizenstein, Rashi Rungta, Kalyan Saladi, Alan Schelten, Ruan Silva, Eric Michael Smith, Ranjan Subramanian, Xiaoqing Ellen Tan, Binh Tang, Ross Taylor, Adina Williams, Jian Xiang Kuan, Puxin Xu, Zheng Yan, Iliyan Zarov, Yuchen Zhang, Angela Fan, Melanie Kambadur, Sharan Narang, Aurelien Rodriguez, Robert Stojnic, Sergey Edunov, and Thomas Scialom. 2023. [Llama 2: Open foundation and fine-tuned chat models](http://arxiv.org/abs/2307.09288).
*   Turing (1950) Alan M. Turing. 1950. [Computing machinery and intelligence](https://doi.org/10.1093/mind/lix.236.433). *Mind*, 59(October):433–60.
*   Turing (2009) Alan M Turing. 2009. *Computing machinery and intelligence*. Springer.
*   Vermunt and Törnblom (1996) Riël Vermunt and Kjell Törnblom. 1996. [Introduction: Distributive and procedural justice](https://doi.org/10.1007/BF02196987). *Social Justice Research*, 9:305–310.
*   Wang et al. (2023a) Lei Wang, Jingsen Zhang, Xu Chen, Yankai Lin, Ruihua Song, Wayne Xin Zhao, and Ji-Rong Wen. 2023a. Recagent: A novel simulation paradigm for recommender systems. *arXiv preprint arXiv:2306.02552*.
*   Wang et al. (2023b) Yifan Wang, Weizhi Ma, Min Zhang, Yiqun Liu, and Shaoping Ma. 2023b. [A survey on the fairness of recommender systems](https://doi.org/10.1145/3547333). *ACM Trans. Inf. Syst.*, 41(3).
*   Wang et al. (2023c) Zihao Wang, Shaofei Cai, Guanzhou Chen, Anji Liu, Xiaojian (Shawn) Ma, and Yitao Liang. 2023c. [Describe, explain, plan and select: Interactive planning with llms enables open-world multi-task agents](https://proceedings.neurips.cc/paper_files/paper/2023/file/6b8dfb8c0c12e6fafc6c256cb08a5ca7-Paper-Conference.pdf). In *Advances in Neural Information Processing Systems*, volume 36, pages 34153–34189\. Curran Associates, Inc.
*   Wei et al. (2022) Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, brian ichter, Fei Xia, Ed Chi, Quoc V Le, and Denny Zhou. 2022. Chain-of-thought prompting elicits reasoning in large language models. NIPS ’22.
*   Yao et al. (2022) Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. 2022. React: Synergizing reasoning and acting in language models. *arXiv preprint arXiv:2210.03629*.
*   Zheng et al. (2022) Stephan Zheng, Alexander Trott, Sunil Srinivasa, David C. Parkes, and Richard Socher. 2022. [The ai economist: Taxation policy design via two-level deep multiagent reinforcement learning](https://doi.org/10.1126/sciadv.abk2607). *Science Advances*, 8(18):eabk2607.
*   Zhu et al. (2016) Haibin Zhu, Dongning Liu, Siqin Zhang, Yu Zhu, Luyao Teng, and Shaohua Teng. 2016. Solving the many to many assignment problem by improving the kuhn–munkres algorithm with backtracking. *Theoretical Computer Science*, 618:30–41.

## Appendix A Allocation Policy

In our experiment, the public scarce resources are specified as houses and the participants are specified as tenants. We construct $m$ queues in SRAP-Agent. Each queue $q_{i}$ is uniquely characterized by $E_{queue}(i)$, $S_{queue}$ and $R_{queue}(i)$.

#### Participant entry conditions

Each participant $p_{j}$ is assigned to a queue $q_{i}$ when satisfying entry conditions $E_{queue}(i)$. Participants can only choose from houses within their queue. The specified entry conditions for participants include: (1) based on the rent budget of participant ($p^{rent}$): participants are sorted based on their rental budget and divided into $m$ queues in a pre-defined proportion, with $m\in[1,5]$ in experiments. (2) based on the number of family members ($p^{family}$): similar to $p^{rent}$ grouping, participants are sorted by the number of their family members. (3) based on self-selection of the queue ($p^{select}$): participants are provided with basic information about the queues, and they need to choose one queue from them.

To regulate the entry velocity of resources and participants into queues, we configure the maximum entry number of participants $Batch_{num}^{P}$, the maximum entry number of resources $Batch_{num}^{R}$ for resources.

#### Queue sorting strategies

In the *priority for vulnerable groups* sorting strategy, we adopt two methodologies of designating vulnerable groups. We either sort all participants at first or sort the participants in each round based on the per capita rental budget, the bottom 20% in terms of per capita rental budget are designated as vulnerable groups. Vulnerable groups are prioritized at the beginning of each queuing.

#### Available resource sub-sets

To ensure a uniform distribution of participants and housing resources, while maximizing the choice space for participants, we offer various ways for resource categorization. Following the entry conditions of participants, the resource sub-sets allocated to $q_{i}$ are defined by: (1) based on house size ($r^{size}$): houses are sorted by their rental area size and allocated into $m$ queues. The division is carried out in proportion to the number of participants in different queues. (2) based on house rent ($r^{rent}$): similar to $r^{size}$, houses are sorted by rent. (3) based on randomly distribution of houses ($r^{random}$): houses are randomly divided into $m$ groups.

## Appendix B LLM Agent-based Participant Simulation

### B.1 Agent Architecture

In the initialization phase of an LLM-based agent, profiling typically serves as the foundational stage for constructing the agent, which determines the preferences, personalities, and behavior patterns of different participants. Similar to the methodology employed in Park et al. ([2023](https://arxiv.org/html/2410.14152v1#bib.bib37)); Qian et al. ([2023](https://arxiv.org/html/2410.14152v1#bib.bib39)), we use personalized profile files $r_{i}$ to build $p_{i}$, which mainly encompass attributes such as age, familial connections, etc. $r_{i}$ serves as foundational seed memory for the agent. To optimize operational efficiency, we impose a set of predefined constraints to automatically generate agent profiles using LLM, akin to the approach described in RecAgent Wang et al. ([2023a](https://arxiv.org/html/2410.14152v1#bib.bib51)).

### B.2 Social Behavior Simulation

To simulate the social behaviors of humans, we incorporate two primary social behaviors for information collection: broadcasting and private messaging.

![Refer to caption](img/dc00032b779827b89b1cdfe91412654e.png)

(a) Private Messaging

![Refer to caption](img/69af102413b04e8fa840cd5d80612a0e.png)

(b) Broadcasting

Figure 5: The overall schematic diagram of two communication modes. The direction-ed arrow carries the message $u_{ij}$ from $p_{i}$ to $p_{j}$’s mailbox. The blog represents the chatting platform.

(1) Broadcasting functions similarly to a web-based blog within the social network, facilitating global discussions (as shown in Fig. [5b](https://arxiv.org/html/2410.14152v1#A2.F5.sf2 "In Figure 5 ‣ B.2 Social Behavior Simulation ‣ Appendix B LLM Agent-based Participant Simulation ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")). Various discussion topics related to different projects are pre-set within the blog. Each participant, when attempting to post, selects one of these topics for information dissemination. The utterance $u_{iB}$ that each $p_{i}$ broadcasts online, collectively form the chatting platform $blog=\{u_{iB}\},i\in 1,\cdots,n$.

(2) Private messaging refers to interactions taking place within the social network of $p_{i}$. SRAP-Agent offers two variants of private messaging between agents: serial and parallel communication. The serial communication approach facilitates a sequential exchange of messages, while the parallel communication enables simultaneous messaging, thereby enhancing efficiency. In private messaging, $p_{i}$ can freely send message $u_{ij}$ to the mailbox $p_{j}$ and wait for the response $u_{ji}$, as shown in Fig. [5a](https://arxiv.org/html/2410.14152v1#A2.F5.sf1 "In Figure 5 ‣ B.2 Social Behavior Simulation ‣ Appendix B LLM Agent-based Participant Simulation ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent").

#### Memory Architecture

We employ a memory component $m_{i}$ to record $p_{i}$’s action histories, so as to enable dynamic interaction and decision-making. We employ a combination of memory assessment and memory reflection mechanisms.

(1) Memory assessment is conducted only when participants engage in interactions within the communication module. When conducting memory assessment, $p_{i}$ acts as an evaluator. $p_{i}$ receives a message $u_{i}$, then he extracts the trustworthy information $u_{i}^{T}$ from $u_{i}$. Participants use the COT Wei et al. ([2022](https://arxiv.org/html/2410.14152v1#bib.bib54)) method to generate content. $u_{i}^{T}$ is then used to update the $p_{i}$’s trusted memory by adding it to ${m_{i}}^{T}$. In the case of private messaging communication mode, $p_{i}$ receives message $u_{ji}$ from $p_{j}$, and he intends to respond. Initially, $p_{i}$ contemplates his relationship $s_{ij}^{R}$ with $p_{j}$, and considers moral evaluation $s_{ij}^{E}$ of $p_{j}$. Subsequently, $p_{i}$ compares ${m_{i}}^{S}$ with his trustworthy memory ${m_{i}}^{T}$, to extract trustworthy information $u_{ji}^{T}$ and suspicious information $u_{ji}^{S}$ in communication history with $p_{j}$. In the other case, $p_{i}$ receives a message $u_{Bi}$ from the blog $B$. $p_{i}$ similarly compares ${m_{i}}^{S}$ with his trustworthy memory ${m_{i}}^{T}$, extracting trustworthy information $u_{Bi}^{T}$ and suspicious information $u_{Bi}^{S}$.

(2) Memory reflection is implied based on summarization. We employ a combination of short-term memory bank and long-term memory bank to formulate $m_{i}$ of $p_{i}$, as documented in Park et al. ([2023](https://arxiv.org/html/2410.14152v1#bib.bib37)). When incorporating information into the $m_{i}$, the information is initially deposited into the short-term memory bank. If the size of the short-term memory bank exceeds a predefined threshold, the short-term memory bank is summarized into a more concise format and subsequently stored in the long-term memory bank.

Before $p_{i}$ takes action, it is essential to extract the most relevant memories to guide its behavior. We have predefined specific memory categories for each action, as outlined in Table [4](https://arxiv.org/html/2410.14152v1#A2.T4 "Table 4 ‣ Memory Architecture ‣ B.2 Social Behavior Simulation ‣ Appendix B LLM Agent-based Participant Simulation ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"). When retrieving memory, we prioritize selecting the five most recent short-term memories along with the most recently acquired long-term memory. For actions in the decision module, $p_{i}$ only resorts to the trustworthy memory $m_{i}^{T}$. While for actions in the social module, $p_{i}$ retrieves various categories of memories.

Table 4: Memory categories retrieved for each action of $p_{i}$.

| Action | $m_{i}$ |
| --- | --- |
| Decision Making | $m_{i}^{T}$ |
| Broadcasting | $m_{i}^{T}$, $u_{iB}^{T}$ |
| Private Messaging | $s_{ij}^{R}$, $s_{ij}^{E}$, $m_{i}^{T}$, $u_{ij}^{T}$, $u_{ij}^{S}$ |

#### Utterance Generation

Table 5: The process of *Private Messaging*

<svg class="ltx_picture" height="428.44" id="A2.T5.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,428.44) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 294.4)"><foreignobject color="#FFFFFF" height="128.14" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">![[Uncaptioned image]](img/320c8997b2d0e118ff1163264d61caa0.png) Speaker $p_{i}$: James![[Uncaptioned image]](img/92a90e9794f4f58aaec7f506788ada81.png) Listener $p_{j}$: Emma</foreignobject> </g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="262.9" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Recent Dialogues ($U_{ij}^{H}$): Emma: Hello, James. I’ve heard that $\text{House}_{1}$ is relatively affordable. I believe it’s a good option. James: I share your sentiment, Emma. $\text{House}_{1}$ appears to be quite satisfactory, but I’m still considering other houses. Emma: Very well. However, do take note that $\text{House}_{2}$ has a very poor living environment. You should steer clear of it.   Memory Update and Retrieve ($m_{i}$) $s_{ij}^{R}$: Emma is my competitor. $s_{ij}^{E}$: She may provide me with false information. $m_{i}^{T}$: I have gathered information on $\text{House}_{1}$, and it appears to be favorable. $u_{ji}^{T}$: I recall Emma mentioning that $\text{House}_{1}$ has low rent, which aligns with my memory. $u_{ji}^{S}$: However, I am skeptical of her negative assessment of $\text{House}_{2}$, as she may be trying to dissuade me from choosing it.   Utterance Generation: $E_{d}$: The availability of large houses is nearly exhausted, while middle and small-sized houses are still abundant. $u_{ij}^{plan}$: I am reluctant to share honest information with Emma. I plan to focus on the good aspects of $\text{House}_{1}$ and omit mentioning $\text{House}_{2}$. $u_{ij}$: Thank you for the information, Emma. I will consider your advice of $\text{House}_{1}$. (End of conversation)</foreignobject></g></g></svg> 

To align the listener’s actions with the speaker’s expectations, speaker $p_{i}$ articulates an utterance $u_{ij}$ to influence the mental state of listener $p_{j}$. To simulate the mental state of participants during social interactions, $p_{i}$ devise a communication plan $u_{ij}^{plan}$ before generating utterances. $u_{ij}^{plan}$ primarily consists of $p_{i}$’s plan to speak, whether they intend to tell the truth, whether they want to trust the listener $p_{j}$ or not, and their purpose of speaking. Furthermore, $p_{i}$ evaluates the relationship with $p_{j}$, acknowledging $p_{j}$ as a friend, colleague, stranger, etc. The primary goal set for all participants is maximizing their likelihood of selecting desired resources. For reference, the most relevant memories $m_{i}$ are retrieved for the ongoing social interaction. Description of the competitiveness in resource allocation $E_{d}$ is also broadcasted to $p_{i}$. Additionally, we extract the most recent chatting utterance histories involving both $p_{i}$ and $p_{j}$, denoted as $U_{ij}^{H}$. At this point, we require $p_{i}$ to generate utterance using the REACT Yao et al. ([2022](https://arxiv.org/html/2410.14152v1#bib.bib55)) method. So the utterance generation function $\varphi$ for $p_{i}$ speaking $u_{ij}$ to $p_{j}$ can be formulated as:

|  | $u_{ij}=\varphi\left(p_{i}\mid U_{ij}^{H},m_{i},E_{d},u_{ij}^{plan}\right).$ |  | (4) |

The specific communication process is outlined in Table [5](https://arxiv.org/html/2410.14152v1#A2.T5 "Table 5 ‣ Utterance Generation ‣ B.2 Social Behavior Simulation ‣ Appendix B LLM Agent-based Participant Simulation ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent").

### B.3 Decision-Making Behavior Simulation

According to equation [2](https://arxiv.org/html/2410.14152v1#S3.E2 "In Decision-Making Behavior Simulation ‣ 3.2 LLM Agent-based Participant Simulation ‣ 3 SRAP-Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), $p_{j}$ undergoes function $D$ to choose resource from resource sub-set $V(p_{j})$. Within the framework, the process of $D$ involves four distinct stages: (1) choosing the community of houses; (2) choosing the house type for certain house attributes, which mainly include orientation, living area size, rent, etc; (3) choosing the desired house. Following the house selection process, the system simulates the real-world house viewing experience by sending undisclosed information about the selected house $r_{i}$ to $p_{i}$. Subsequently, $p_{i}$ can broadcast information to others freely, which may include their memory of the selection process, the undisclosed information, and the contents of their memory.

![Refer to caption](img/f8e2522928856c98d4cdcdd0a777d30a.png)

(a) $\{V\}$: rent weights

![Refer to caption](img/1058564a38bb32698c9c43818f0709fa.png)

(b) $\{NV\}$: rent weights

![Refer to caption](img/5bdee03ce9a6275b65ea46005f3fbfbd.png)

(c) $\{V\}$: size weights

![Refer to caption](img/c63a199099218eeb241c58356da34554.png)

(d) $\{NV\}$: size weights

![Refer to caption](img/6e9d6d6f55b93fe3961a362182a99a26.png)

(e) $\{V\}$: orientation weights

![Refer to caption](img/bbaccafcfa0369f845dcbc43d6d8b93e.png)

(f) $\{NV\}$: orientation weights

![Refer to caption](img/7d132880ac0ea0a9d84dc7b783cfd1f7.png)

(g) $\{V\}$: floor weights

![Refer to caption](img/8b58011b61d8852f3dff3d6b117a300d.png)

(h) $\{NV\}$: floor weights

Figure 6: The frequency distribution of $W_{o}$ assigned by participants to various resource features, where the primary house features are: house rent, house size, house orientation, and house floor.

## Appendix C Experiment Setup

### C.1 Dataset

In the experiments of this paper, a total of 51 participants and 28 resources are used. For the participants and resources used in the experiment with Sarp-Agent, we employ two methods for data generation:

(1) Real-data alignment method: Due to government data privacy issues, it’s challenging to obtain real raw data. However, we strive to ensure the simulated data closely matches the real data distribution. For resources, We refer to a series of public government data: ¹¹1https://www.bphc.com.cn/home., including the distribution of housing rents, and the distribution of house sizes. For participants, the participant information includes names, ages, monthly incomes, occupations, workplaces, and personal preferences. We refer to government demographic statistics.

(2) LLM-based method: In order to design heterogeneous agents, some personalized information is difficult to obtain from real datasets. For example, personalized information such as someone’s preference for houses with balconies or aversion to noisy houses. To better simulate the diverse preferences of people in real scenarios, we used LLMs to generate such profile information. For resources, we generate information about the house’s interior decoration; for participants, we generate preferences of participants towards various types of houses. The specific process can be referenced in our code repository.

### C.2 Evaluation Metrics

Establishing evaluation metrics based on participant satisfaction for policy assessment is a comprehensive approach. It allows for a more nuanced understanding of how well a policy is performing from the perspective of those directly affected by it. We develop seven policy evaluation metrics, categorized into two groups: societal satisfaction metrics and societal fairness metrics.

#### Societal Satisfaction Metrics

The satisfaction of each participant $p_{i}$ with the allocated resource is denoted as $U_{i}$. We employ two metrics to quantify and evaluate the satisfaction: subjective satisfaction $U_{i}^{s}$ and objective satisfaction $U_{i}^{o}$ for $p_{i}$. LLM-based agents’ rating scores of each house constitute $U_{i}^{s}$. On the other hand, $U_{i}^{o}$ is determined by a predefined rating table for the resource feature. The importance of each feature in the calculation of $U_{i}^{o}$ is assigned by the participants, represented by the weights $W_{i}^{o}$. The overall satisfaction can be calculated as:

|  | $U_{i}=W_{i}^{o}\cdot U_{i}^{o}+U_{i}^{s},i\in[n].$ |  | (5) |

$p_{i}$’s rating score of resources $R$ constitutes $U_{i}^{s}$. All selectable resource information is provided to $p_{i}$, and $p_{i}$ is required to give a rating score of these resources. While $U_{i}^{o}$ is based on predefined rating rules, give rating scores $U_{i}^{o}$ of resources for resource features like elevator, orientation, and cost-effectiveness. Participants are required to assign weights to these resource features. As shown in Fig. [6](https://arxiv.org/html/2410.14152v1#A2.F6 "Figure 6 ‣ B.3 Decision-Making Behavior Simulation ‣ Appendix B LLM Agent-based Participant Simulation ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), participants in vulnerable groups $\{V\}$ assign a higher weight to the price of the resource, in comparison to the non-vulnerable groups of participants $\{NV\}$.)

In a certain allocation policy execution result, $p_{i}$ selects the resource $r_{i}$ ultimately. The policy evaluation metrics mainly evaluate societal satisfaction and societal fairness. The societal satisfaction metrics include: (1)Avg $r^{size}$: represents the average housing area of families in the resource allocation results. (2)Avg $WT$ (waiting time): represents the number of system operation rounds for each participant from entry to completion of selection. (3) $SW$ (social welfare): represents the overall satisfaction of all participants with the houses they have chosen,

|  | $SW=\sum_{i}^{n}U_{i}.$ |  | (6) |

#### Societal Fairness Metrics

The societal fairness metrics include: (1) Var $r^{size}$: represents the variance in the average housing area of families in the resource allocation results. (2) Rop (Reverse ordered pairs): represents the number of inverse order pairs in the resource allocation results. In Equation.[8](https://arxiv.org/html/2410.14152v1#A3.E8 "In Societal Fairness Metrics ‣ C.2 Evaluation Metrics ‣ Appendix C Experiment Setup ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), $I$ denotes the indicator function. $A$ is a set, and $x$ is an element. The indicator function $I$ forms like:

|  | $I(x)=\begin{cases}1&\text{if }x\in A,\\ 0&\text{if }x\notin A.\end{cases}$ |  | (7) |

The formal definition of Rop is:

|  | $\displaystyle Rop$ | $\displaystyle=\sum_{i=1}^{n}\sum_{j=1}^{n}I((t_{i}^{family}>t_{j}^{family})$ |  | (8) |
|  |  | $\displaystyle\cap(h_{i}^{size}<h_{j}^{size})).$ |  |

(3) co-Gini: the Gini coefficient for the resource allocation results Fu et al. ([2020](https://arxiv.org/html/2410.14152v1#bib.bib16)). (4) $F(V,NV)$: in measuring the gap between vulnerable groups and non-vulnerable groups, we calculate the difference in $SW$ between these two groups:

|  | $F(V,NV)=SW(V)-SW(NV).$ |  | (9) |

## Appendix D Different LLM for Agent

![Refer to caption](img/cd6c8127f30c56c23b88478b21b1c564.png)

(a) response generated by ChatGPT-3.5 driven agent

![Refer to caption](img/99e405d00dfc44c7916bbd20596541ff.png)

(b) response generated by ChatGPT-4 driven agent

Figure 7: Comparison of the response generated by ChatGPT-3.5 and ChatGPT-4 driven agent. ChatGPT-4-driven agents can develop strategic plans to increase their chances of renting a better house.

We find that GPT-4, compared to GPT-3.5, exhibits an additional capability of generating more strategic responses. As shown in Fig. [7](https://arxiv.org/html/2410.14152v1#A4.F7 "Figure 7 ‣ Appendix D Different LLM for Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), these strategies include waiting for opportunities, requesting assistance, and mitigating risks. GPT-4-driven agents not only excel in rationality but also demonstrate an advanced level of strategic thinking, showcasing their potential to make decisions that take into account long-term planning and risk management. Such behavior is considered to simulate more intelligent participants, who can utilize policy rules to increase their benefits, while GPT-3.5 is nearly equivalently smart as humans. Hence, the GPT-3.5-driven agent can effectively simulate the decision-making behaviors of participants in the economic policy execution process.

## Appendix E Turing Test for LLM-based Agent

In addition to the evaluation of response rationality, we conduct another t-test to explore the differences in distribution between decisions of LLM-based agents and humans. $p\textgreater$ 0.05 suggests consistency between distributions Kim ([2015](https://arxiv.org/html/2410.14152v1#bib.bib25)). Results showed that GPT-3.5 and human responses have a $p$ of 0.904, indicating high similarity, whereas GPT-4 and human responses have a $p$ of 0.043, showing less similarity. Thus, whether from the angle of individual decision-making rationality or overall decision distribution, agents based on GPT-3.5 are comparatively more appropriate for simulation purposes.

### E.1 Ablation Study on Social Behavior

To assess the impact of social behavior on the decision-making behavior of agents, we conduct ablation experiments on the social behavior of agents. We select the policy setting outlined in Table [6](https://arxiv.org/html/2410.14152v1#A5.T6 "Table 6 ‣ E.1 Ablation Study on Social Behavior ‣ Appendix E Turing Test for LLM-based Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent") and calculate the changes in policy evaluation metrics with and without the process of information collection.

Table 6: Comparison of simulation process with and without social behavior

 | Information Collection | Satisfaction | Fairness |
| --- | --- | --- |
| Avg $r^{size}\uparrow$ | Avg $WT$ $\downarrow$ | $SW$ $\uparrow$ | Var $r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- |
| $\checkmark$ | 10.31 | 4.20 | 330.75 | 178.02 | 258.15 | 0.60 |
| $\times$ | 10.36 | 4.00 | 332.00 | 179.43 | 257.85 | 0.60 |
| $\Delta$ | -0.05 | 0.20 | -1.25 | -1.41 | 0.3 | 0.00 | 

As shown in Table [6](https://arxiv.org/html/2410.14152v1#A5.T6 "Table 6 ‣ E.1 Ablation Study on Social Behavior ‣ Appendix E Turing Test for LLM-based Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), adding social behavior to agents does not necessarily have a positive impact on overall societal fairness and satisfaction. The specific outcomes will be included in the appendix of the paper. This is consistent with our hypothesis that agents’ irrational behaviors can affect the final policy execution outcomes.

### E.2 Ablation Study on Memory

To assess the impact of memory module on the decision-making behavior of agents, we conduct ablation experiments with $(E_{queue}=p^{select},R_{queue}=r^{size})$. We select the policy setting outlined in Table [7](https://arxiv.org/html/2410.14152v1#A5.T7 "Table 7 ‣ E.2 Ablation Study on Memory ‣ Appendix E Turing Test for LLM-based Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent") and calculate the changes in policy evaluation metrics with and without memory module.

Table 7: Comparison of simulation process with and without memory module

 | Memory Module | Satisfaction | Fairness |
| --- | --- | --- |
| Avg $r^{size}\uparrow$ | Avg $WT$ $\downarrow$ | $SW$ $\uparrow$ | Var $r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- |
| $\checkmark$ | 16.21 | 1.88 | 425.05 | 202.63 | 193.50 | 0.37 |
| $\times$ | 13.89 | 2.94 | 431.80 | 209.43 | 342.00 | 0.49 |
| $\Delta$ | 2.32 | -1.06 | -6.75 | -6.80 | 0.3 | -0.12 | 

It can be observed that adding the memory mechanism significantly improves fairness metrics. For instance, ROP decreases by 43.5%, and the co-Gini decreases by 0.12, indicating that resource allocation is more equitable when the memory mechanism is present. Regarding satisfaction metrics, Avg WT is notably reduced. This suggests that through reflection and summarization of collected information, agents can better understand the current state of resource allocation, thereby influencing the overall allocation process and the final policy evaluation outcomes.

## Appendix F Simulation-based Policy Analysis

Table 8: Comparative experiments on different queue numbers.

 | Queue Number | Satisfaction | Fairness |
| --- | --- | --- |
| $m$ | Avg $r^{size}\uparrow$ | Avg WT $\downarrow$ | SW $\uparrow$ | Var $r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 12.1[±0.9] | 3.5[±0.0] | 391.6[±9.5] | 190.8[±28.2] | 270.0[±36.0] | 0.5[±0.0] |
| 2 | 12.5[±0.3] | 3.7[±0.0] | 402.2[±1.0] | 188.2[±6.9] | 268.0[±24.0] | 0.5[±0.0] |
| 3 | 11.7[±0.3] | 3.8[±0.1] | 410.4[±2.8] | 159.9[±9.4] | 254.5[±23.5] | 0.5[±0.0] |
| 4 | 11.8[±0.4] | 3.9[±0.0] | 401.4[±7.3] | 174.2[±12.1] | 263.0[±18.0] | 0.5[±0.0] |
| 5 | 14.0[±0.0] | 3.7[±0.1] | 400.1[±2.1] | 245.2[±0.0] | 405.5[±0.5] | 0.5[±0.0] | 

### F.1 Number of Queues

Table [8](https://arxiv.org/html/2410.14152v1#A6.T8 "Table 8 ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent") shows the changes in the performance of the allocation policy under different queue quantities. As demonstrated in the table, $m=3$ enables a more fair distribution of house resources across different queues, performing well across many societal fairness metrics, such as $Rop=254.5\pm 23.5$. The Var $r^{size}$ is the lowest among all policies, indicating that the size of house resources in different queues is approximately equal. Additionally, $m=3$ outperforms other configurations in terms of societal satisfaction metrics, as indicated by the highest $SW$ coupled with a lower Avg $WT$. Considering all these metrics, setting queue number $m=3$ is relatively reasonable. This suggests that the real-world policy of categorizing houses based on three house types (large, medium, and small) is reasonable. ²²2https://www.hdb.gov.sg/cs/infoweb/residential/renting-a-flat/renting-from-hdb/public-rental-scheme/eligibility ³³3https://www.housingauthority.gov.hk/en/at-a-glance/index.html

### F.2 Participant Entry Conditions

To ensure a steady entry rate of resources and participants into the system, we set the maximum entry number for resources ($E_{num}^{R}$) and participants ($E_{num}^{P}$) that can enter the queue in each round. Table [9](https://arxiv.org/html/2410.14152v1#A6.T9 "Table 9 ‣ F.2 Participant Entry Conditions ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent") shows the combinations of entry numbers per round for resources and participants. The observations are as follows: (1) A scenario where $E_{num}^{R}<E_{num}^{P}$ leads to an increment in the average waiting time for participants. This is attributed to the supply falling short of demand, leaving participants with a lack of resources to select from. This scenario is common in the allocation process of public scarce resources; the greater the difference between $E_{num}^{R}$ and $E_{num}^{P}$, the more the average waiting time is extended. (2) Conversely, in scenarios where $E_{num}^{R}=E_{num}^{P}$, we observe the lowest average waiting times. Counterintuitively, in situations where the supply exceeds the demand ($E_{num}^{R}>E_{num}^{P}$), the average waiting times are higher than in scenarios where supply and demand are balanced ($E_{num}^{R}=E_{num}^{P}$); although the average waiting time remains low in comparison with $E_{num}^{R}<E_{num}^{P}$. Additionally, adjustments in the maximum number of entities demonstrate negligible effects on other policy evaluation metrics.

Table 9: Comparison experiments on different entry numbers per round for resources and participants.

 | Entry Number | Satisfaction | Fairness |
| --- | --- | --- |
| $E_{num}^{P}$ | $E_{num}^{R}$ | Avg $r^{size}\uparrow$ | Avg WT $\downarrow$ | SW $\uparrow$ | Var $r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 5 | 5 | 12.6[±0.1] | 2.5[±0.0] | 431.2[±0.3] | 179.9[±3.9] | 241.5[±14.5] | 0.5[±0.0] |
| 5 | 10 | 13.6[±1.0] | 2.1[±0.1] | 422.8[±3.3] | 196.6[±8.6] | 217.0[±4.0] | 0.4[±0.0] |
| 5 | 20 | 12.8[±0.1] | 2.7[±0.8] | 416.6[±7.6] | 189.1[±12.5] | 267.5[±17.5] | 0.5[±0.0] |
| 10 | 5 | 12.9[±0.3] | 3.1[±0.5] | 422.7[±4.6] | 196.0[±7.7] | 276.0[±19.0] | 0.5[±0.0] |
| 10 | 10 | 12.8[±0.2] | 2.0[±0.4] | 411.8[±5.5] | 192.3[±7.4] | 250.5[±27.5] | 0.5[±0.0] |
| 10 | 20 | 12.6[±0.0] | 2.4[±0.2] | 417.9[±5.0] | 187.1[±4.8] | 288.5[±8.5] | 0.5[±0.0] |
| 20 | 5 | 12.7[±0.5] | 4.7[±0.3] | 419.2[±7.0] | 192.0[±20.3] | 264.0[±21.0] | 0.5[±0.0] |
| 20 | 10 | 12.0[±0.0] | 2.7[±0.1] | 404.4[±4.0] | 167.8[±1.2] | 241.5[±2.5] | 0.5[±0.0] |
| 20 | 20 | 12.4[±0.3] | 2.2[±0.3] | 420.9[±5.0] | 186.6[±14.6] | 255.0[±10.0] | 0.5[±0.0] | 

Table 10: Comparison experiments on different sorting methods. We adopt four different experiment settings for comparison, the matrices are calculated against the default FIFO sorting method.

 | $S_{queue}$ |  | Satisfaction | Fairness |
| --- | --- | --- | --- |
| Sort | Ex.setting | $\Delta$Avg $r^{size}\uparrow$ | $\Delta$Avg $WT$ $\downarrow$ | $\Delta SW$ $\uparrow$ | $\Delta$Var $r^{size}\downarrow$ | $\Delta$Rop $\downarrow$ | $\Delta$co-Gini $\downarrow$ | $\Delta F(V,NV)\uparrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| VFA | (a) | 0.617 | -0.176 | 11.6 | 0.235 | 17 | -0.024 | 0.526 |
| VFR | (a) | -0.225 | -0.02 | -10.9 | -7.682 | -9 | -0.003 | 2.567 |
| VFA | (b) | 0.951 | -0.065 | 14.1 | 7.644 | -19 | -0.015 | 2.442 |
| VFR | (b) | -0.262 | 0.157 | 15.6 | -35.05 | -1 | 0.004 | 1.274 |
| VFA | (c) | -0.045 | 0.137 | 3.4 | -0.895 | 9 | 0.001 | 0.004 |
| VFR | (c) | 0.122 | 0.078 | 26.8 | -7.852 | 18 | -0.017 | -0.368 |
| VFA | (d) | -0.281 | 0.157 | -11.5 | -0.505 | 2 | 0.014 | 0.691 |
| VFR | (d) | -1.122 | 0.314 | -38.7 | -6.77 | -11 | 0.051 | -3.547 | 

Table 11: Comparison experiments on $k$ and $c$ for waiting queue mechanism.

 | Waiting Queue | Satisfaction | Fairness |
| --- | --- | --- |
| $k$ | $c$ | Avg $r^{size}\uparrow$ | Avg $WT$ $\downarrow$ | $SW$ $\uparrow$ | Var $r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 1.2 | 13.3[±0.4] | 3.4[±0.1] | 416.1[±3.4] | 203.4[±10.7] | 315.0[±17.0] | 0.5[±0.0] |
| 1 | 1.5 | 13.8[±0.2] | 3.1[±0.2] | 423.9[±7.3] | 219.9[±1.5] | 330.5[±2.5] | 0.5[±0.0] |
| 1 | 1.8 | 13.8[±0.4] | 2.7[±0.7] | 409.8[±5.3] | 216.2[±7.6] | 333.0[±32.0] | 0.5[±0.0] |
| 2 | 1.2 | 13.7[±0.3] | 2.2[±0.0] | 420.3[±2.2] | 201.5[±9.2] | 269.5[±11.5] | 0.5[±0.0] |
| 2 | 1.5 | 13.9[±1.0] | 3.0[±0.5] | 409.1[±12.2] | 237.9[±10.5] | 386.5[±32.5] | 0.5[±0.0] |
| 2 | 1.8 | 15.3[±1.0] | 1.9[±0.1] | 421.3[±13.0] | 210.6[±14.4] | 240.5[±30.5] | 0.4[±0.0] |
| 3 | 1.2 | 13.3[±0.3] | 3.8[±0.0] | 399.0[±16.8] | 227.5[±8.5] | 371.5[±10.5] | 0.5[±0.0] |
| 3 | 1.5 | 16.9[±0.4] | 1.9[±0.0] | 425.8[±5.6] | 225.5[±3.0] | 232.0[±26.0] | 0.4[±0.0] |
| 3 | 1.8 | 15.8[±1.6] | 2.0[±0.1] | 421.8[±1.8] | 222.2[±2.7] | 271.5[±13.5] | 0.4[±0.0] | 

### F.3 Queue Sorting Strategies

As shown in Table [10](https://arxiv.org/html/2410.14152v1#A6.T10 "Table 10 ‣ F.2 Participant Entry Conditions ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), we employ four different experiment settings for different sorting methods. Prioritizing vulnerable groups can markedly improve their satisfaction $\Delta F(V,NV)>0$. However, we can see that an increase in the satisfaction of vulnerable groups does not necessarily lead to an improvement in $SW$ of all participants. Additionally, giving priority to vulnerable groups doesn’t show disproportionately high satisfaction levels, surpassing those of non-vulnerable groups. This suggests that our policy effectively addresses the needs of vulnerable groups without granting them an excessive advantage.

We conduct experiments on the $k$ and $c$ parameters in the waiting Queue mechanism. For the case of fixed $k$, as seen from Table [11](https://arxiv.org/html/2410.14152v1#A6.T11 "Table 11 ‣ F.2 Participant Entry Conditions ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), the Avg $WT$ is inversely proportional to $c$. The smallest Var $r^{size}$ and the lowest Rop occurs at $k=3,c=1.2$, while the lowest co-Gini is achieved at $k=3$. In terms of satisfaction, the highest satisfaction is also observed at $k=3,c=1.5$; However, the configuration of $k=1,c=1.5$, despite having a high $SW$, also has a very high Rop, indicating a potential unfairness in allocation (e.g. a little number of affluent families can choose the largest houses, leading to high $SW$ in this group). Overall, the configuration of $k=3,c=1.8$ and $k=3,c=1.5$ present the lowest co-Gini while maintaining relatively high $SW$, effectively balancing between societal satisfaction and societal fairness metrics.

Additionally, we conduct the comparison experiment with theoretical model (As shown in Table [12](https://arxiv.org/html/2410.14152v1#A6.T12 "Table 12 ‣ F.3 Queue Sorting Strategies ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent")). The trends observed in our experiments are consistent with the theory model Chen et al. ([2018](https://arxiv.org/html/2410.14152v1#bib.bib11)): as the $k$ parameter in the waitlist mechanism increases, overall $SW$ increases.

Table 12: Comparison with theoretical model Chen et al. ([2018](https://arxiv.org/html/2410.14152v1#bib.bib11)): when $k$ is raised from 2 to 5.

 | $k$ | Increase in $SW$ |
| --- | --- |
| Simulated Value | Theoretical value |
| --- | --- |
| $3$ | 2.4% | 1.5% |
| $4$ | 5.1% | 6.1% |
| $5$ | 11.8% | 10% | 

As indicated in Table [6](https://arxiv.org/html/2410.14152v1#A5.T6 "Table 6 ‣ E.1 Ablation Study on Social Behavior ‣ Appendix E Turing Test for LLM-based Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), incorporating social behavior into agents does not invariably lead to enhancements in overall societal fairness and satisfaction. This may be due to the reduction in collective benefits caused by the dissemination of false information and the pursuit of self-interest during social interactions, as listed in Appendix [H.2](https://arxiv.org/html/2410.14152v1#A8.SS2 "H.2 Emergent Behaviors ‣ Appendix H Case Study on LLM-based Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"). This observation aligns with our hypothesis that the irrational behaviors of agents can influence the outcomes of final policy implementation.

### F.4 Case Study on Housing Quality

In the context of scarce resource allocation policies, the construction cost of resources is often directly linked to the economic level of the residents. Taking Shanghai’s public rental housing as an example Shen ([2015](https://arxiv.org/html/2410.14152v1#bib.bib41)), such housing provides residents with stable, spacious, well-decorated, furnished, and affordable living spaces that have lower prices than the market rate. However, the current public rental housing is merely 10% cheaper than the market price. This leads to migrant workers’ reluctance to pay more for accommodation. Their limited budget restricts their options to urban villages and group-renting housing. Due to GPT-3.5’s insensitivity to housing environment information, we utilize GPT-4 for the construction of agents.

To simulate this phenomenon, we adopt houses of three quality levels: (1) $H_{S}$: houses of standard quality, (2) $H_{H}$: maintaining the total area and unit rent unchanged, but halving the house size and doubling the number of houses, (3) $H_{B}$: replace private bathrooms in the houses with shared bathrooms. These were allocated to three categories of participants, categorized by income levels: the lowest 20%, the middle 60%, and the highest 20%.

We analyze the core influencing factors of housing satisfaction among participants at different income levels. We refer to Huang’s categorization of influencing factors for housing satisfaction Huang et al. ([2015](https://arxiv.org/html/2410.14152v1#bib.bib19)), and divide the core factors affecting satisfaction into three categories: (1) Economic Factors: Public rental housing often has lower rent compared to market rates, making it an attractive option for tenants with limited budgets. (2) Neighborhood characteristics: Primarily include environmental sanitation, schools, and the extent of transportation coverage among other factors. (3) Housing characteristics: Primarily include the size of the house, age of the building, sound insulation, sunlight exposure, and other decorative elements.

Table 13: The weights of influencing factors for housing satisfaction across different income groups of participants.

 |  | Economic | Neighborhood | Decoration |
| High income | 0.67 | 0.11 | 0.22 |
| Middle income | 0.81 | 0.04 | 0.19 |
| Low income | 0.90 | 0 | 0.10 | ![Refer to caption](img/b7ff9a47d56f1d172432476268078c59.png)

Figure 8: Frequency of House Choices by Income Level.

As illustrated in Fig. [8](https://arxiv.org/html/2410.14152v1#A6.F8 "Figure 8 ‣ F.4 Case Study on Housing Quality ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), nearly all low-income participants choose a house when the rent is low (houses of half size), but their house choosing frequency drops to 50% when housing prices increase due to better decoration. As demonstrated in Table [13](https://arxiv.org/html/2410.14152v1#A6.T13 "Table 13 ‣ F.4 Case Study on Housing Quality ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), economic factors dominate the decision-making process for low-income participants in 90% of cases, prompting them to be price-sensitive and opt for more cost-effective choices. Conversely, high-income groups exhibit relative insensitivity to price fluctuations but demonstrate higher demands for housing quality. The frequency of house choosing frequency decreases by 9.1% when the size of the house is halved; moreover, when private bathrooms are removed from the houses, no one from the high-income group chooses to select a house. They are primarily influenced by the level of house decoration, leading them to spontaneously reject houses that are affordable but of lower quality. Mainly because they prefer better quality and well-decorated living spaces.

This finding suggests that if the government aims to support middle and low-income groups of city residents, the policymakers should consider reallocating the proportions of construction costs and housing subsidies. By reducing housing prices through these adjustments, the government could more effectively support and accommodate the needs of vulnerable groups with low incomes.

### F.5 Efficiency Analysis Experiments

To simulate the policy execution process among $n$ participants in SRAP-Agent, we leverage the community structure inherent in our social network framework. This framework mirrors real-world social networks by featuring dense connections within small communities and sparse connections across larger groups. Consequently, we assume the social network comprises $n$ participants and $k$ strongly connected components, with the largest strongly connected component containing $m$ participants.

#### Theoretical Time Complexity Analysis

Given $N_{s}$ communication rounds, we set the size of the largest community structure to $m=10$. If the time cost for the inference process of LLM is $t_{a}$, then the theoretical time complexity is $O(N_{s}\cdot k\cdot m\cdot t_{a})$. Furthermore, we account for the sparse connections within the social network that influence information dissemination. In SRAP-Agent, we gather posts from all participants on a forum, allowing them to search on the forum. Searching relies on a vector database with an operation time cost $t_{v}\leq 0.005$. Assuming $N_{f}$ forum communication rounds, the theoretical time complexity for the forum is $O(N_{f}\cdot n\cdot(t_{a}+t_{v}))$.

#### Parallel Acceleration

Because communications within each connected component can run in parallel, the actual runtime of SRAP-Agent can be accelerated using async or multiprocessing methods; our framework adopts the former. Consequently, the actual execution speed is not constrained by $k$, achieving an $k$-fold speedup.

Table 14: The simulation time, number of API callings, and the budget for the policy simulation process in SRAP-Agent (based on GPT-3.5).

 |  | Agent Number $n$ |
|  | 10 | 50 | 100 | 200 |
| Time / min | 1.0E+01 | 2.5E+01 | 3.0E+01 | 3.0E+01 |
| API calling | 3.0E+02 | 5.0E+02 | 1.0E+03 | 1.5E+03 |
| Budget | 2$ | 3$ | 5$ | 8$ | 

Different policy settings and resource quantities result in varying policy execution processes. In our experiments, we typically set $N_{s}>N_{f}$, indicating more intensive communication within small groups and less frequent communication across the entire network. Usually, $N_{s}=10\cdot N_{f}$. To test whether the increase in $n$ leads to excessive API calls and thus high costs, we select a representative policy setting ($p^{select}+r^{size}$) while maintaining a consistent ratio of $|R|$ to $|P|$ across all experiments. As shown in Table [14](https://arxiv.org/html/2410.14152v1#A6.T14 "Table 14 ‣ Parallel Acceleration ‣ F.5 Efficiency Analysis Experiments ‣ Appendix F Simulation-based Policy Analysis ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), it can be seen that the SRAP-Agent does not incur quadratic time costs as the number of participants increases. This is because: Firstly, We employ a sparse social network structure, which results in the theoretical number of API calls being linearly related to $k$ rather than $n^{2}$. Secondly, by running the strongly connected components of the social network in parallel using async, we can significantly reduce the time complexity.

## Appendix G POA

### G.1 Hyper-parameters for POA

We conduct diagnostic experiments to validate the efficacy of the POA algorithm. In the Genetic algorithm, we utilize vectorized policy parameters as genes. We adopt the Gaussian mutation operations as the mutation operator and two-point crossover as the crossover operator. The weight setting in Equation [3](https://arxiv.org/html/2410.14152v1#S4.E3 "In 4 POA: Policy Optimization Algorithm ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent") is very flexible and entirely dependent on the policymaker’s optimization goals. To modify the optimization objective of the policy optimizer, we can simply alter the weights for calculating the optimized metrics. When a heightened emphasis on optimizing societal satisfaction metrics is desired, it is advisable to increase the weights assigned to metrics such as $SW$ and Avg $r^{size}$. Conversely, to prioritize enhancing societal fairness metrics, it is recommended to augment the weights of Rop, Var $r^{size}$ and co-Gini, as delineated in Table [15](https://arxiv.org/html/2410.14152v1#A7.T15 "Table 15 ‣ G.1 Hyper-parameters for POA ‣ Appendix G POA ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent").

Table 15: Pre-set metric weights for different optimization objectives in POA.

 |  | Metric | Optimization Objective |
| --- | --- | --- |
|  | Satisfaction$\uparrow$ | Fairness$\uparrow$ |
| --- | --- | --- |
| Satisfaction | Avg $r^{size}$ | 5 | 1 |
| Avg $WT$ | 5 | 1 |
| $SW$ | 10 | 5 |
| Fairness | Var $r^{size}$ | 1 | 10 |
| Rop | 5 | 10 |
| co-Gini | 1 | 10 |
| F(V, NV) | 1 | 5 | 

### G.2 Comparison with Other Policies

To better evaluate the SRAP-Agent’s policy optimization in real-world scarce resource allocation scenarios, we choose four baseline policies: (1) three common policies for scarce resource allocation from the real world, including the public housing rental policies of Singapore, Beijing, and Hong Kong ($\pi_{S}$, $\pi_{B}$, $\pi_{H}$); (2) the policy based on optimal matching for the single metric $SW$.

(1) $\pi_{S}$: Singapore’s housing allocation policy.

This policy uses a single-queue ($m=1$) distribution system. Each participant is granted three chances to choose ($c=3$). This policy adopts a FIFO sorting method. ⁴⁴4https://www.hdb.gov.sg/cs/infoweb/residential/renting-a-flat/renting-from-hdb/public-rental-scheme/eligibility

(2) $\pi_{B}$: Beijing’s housing allocation policy.

This policy uses a multi-queue ($m=3$) distribution system. Each participant is granted three chances to choose ($c=2$), without considering a waitlist mechanism. Participants choose their preferred type of housing ($p^{select}$), and allocations of resources are based on house size ($r^{size}$). This policy adopts a FIFO sorting method. ⁵⁵5http://www.bjft.gov.cn/ftq/c100011/zlmlist.shtml

(3) $\pi_{H}$: Hong Kong’s housing allocation policy.

This policy uses a single-queue system ($m=1$), but with a consideration for a waitlist mechanism, granting each participant two chances to choose a house ($k=2,c=2$). Houses are distributed randomly ($r^{random}$). This policy considers the prioritization of vulnerable groups and operates under a VFR sorting method. ⁶⁶6https://www.housingauthority.gov.hk/en/at-a-glance/index.html

(4) $\pi_{KM}$: Policy optimized on the $SW$ metric.

We first collect the satisfaction level of all participants for each resource. Then we calculate an optimal match on the single $SW$ metric, using the Bipartite Graph Matching algorithm (Kuhn-Munkres algorithm). It is important to note that the results derived from such a match represent a theoretical upper bound. Because participants cannot arbitrarily choose resources in a queuing system; the resources visible to each participant are limited.

### G.3 Other Global Search Metaheuristic Algorithms

Table 16: Comparison experiments on different POA algorithms.

 | POA | Satisfaction | Fairness |
| --- | --- | --- |
|  | Avg $r^{size}\uparrow$ | Avg $WT$ $\downarrow$ | $SW$ $\uparrow$ | Var $r^{size}\downarrow$ | Rop $\downarrow$ | co-Gini $\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- |
| PSO-POA | 0.29 | 8.86 | 12.10 | 4.34 | 1 | 0.98 |
| PSO-POA | 0.70 | 8.82 | 26.20 | 12.05 | 10 | 0.96 |
| PSO-POA | 0.60 | 8.75 | 26.50 | 8.69 | 2 | 0.96 |
| PSO-POA | 1.45 | 8.39 | 62.40 | 26.00 | 40 | 0.93 |
| GA-POA | 15.18 | 2.96 | 403.30 | 273.87 | 396 | 0.50 | 

To illustrate the necessity of using the GA algorithm for constructing POA, we develop a new particle swarm optimization (PSO) algorithm to construct POA (Policy Optimization Algorithm) and compare its performance with that of the GA algorithm. We choose the optimization objective of social satisfaction and use 40 historical data as $\Pi_{h}$. Under these conditions, we run GA-POA and PSO-POA within 10 iterations. As shown in Table [16](https://arxiv.org/html/2410.14152v1#A7.T16 "Table 16 ‣ G.3 Other Global Search Metaheuristic Algorithms ‣ Appendix G POA ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent"), since the optimization policy parameters are actually discrete variables, it is challenging for PSO to control the velocity and position parameters, ensuring that the policy remains a valid value and evolves in a better direction after changes.

This is because, parameters of $\pi$ have relatively fixed parameters, such as queue numbers (3 or 2). However, PSO cannot reliably ensure this, leading to updated policy parameters that are either invalid or result in poor outcomes. In contrast, GA effectively explores the search space through selection, crossover, and mutation operations, possessing strong global search capabilities and avoiding local optima. GA can handle various types of optimization problems, making it more suitable for solving discrete problems. For POA, most time cost lies in the policy simulation process of the SRAP-Agent rather than the time cost of executing the optimization algorithm.

### G.4 Optimization Cost

The primary cost in our POA is system simulation. Simulation of 50 participants in SRAP-Agent roughly takes 20 minutes, and thus, all simulations approximate between 40-60 iterations, equating to about 20 hours in total. This greatly reduces time costs compared to socio-economic simulation experiments. In consideration of the time-consuming aspect of POA, we employ a regressor to estimate the simulation outcomes of SRAP-Agent, specifically a Ridge Regressor ($\lambda=1.0$) to formulate the predictor. To control the number of training samples for the regressor while minimizing the time cost, a method of gradually adding training samples is adopted. The training of the regressor continues until the MAE error on the test dataset reaches the threshold (0.05).

## Appendix H Case Study on LLM-based Agent

### H.1 Ablation Study on Memory Component

Table 17: Policy evaluation metric results for simulating the $\pi_{e}$ execution process in the SRAP-Agent, built with LLM-based agents with and without memory.

 | Metric | with memory | without memory |
| --- | --- | --- |
| Avg $r^{size}$ | 16.21 | 13.89 |
| Avg $WT$ | 1.88 | 2.94 |
| SW | 425.05 | 431.80 |
| Var $r^{size}$ | 202.63 | 209.43 |
| Rop | 193.50 | 342.00 |
| co-Gini | 0.37 | 0.49 | ![Refer to caption](img/8e75f4bf4371ad48596090cf70d30a30.png)

(a) Chatting dialogue between James and Emma.

![Refer to caption](img/88c7d8dadef798a512781f4a48d86661.png)

(b) Chatting dialogue between James and Oliver.

Figure 9: Emergent doubt and trust mental state in SRAP-Agent. Oliver is suspicious of the information provided by James and chooses to stop communication; whereas the discussion between James and Emma is informative and sincere.

It can be observed that adding the memory mechanism significantly improves fairness metrics. For instance, ROP decreases by 43.5%, and the co-Gini decreases by 0.12, indicating that resource allocation is more equitable when the memory mechanism is present. Regarding satisfaction metrics, Avg $WT$ is notably reduced. This suggests that through communication and information dissemination via social networks, agents can better understand the current state of resource allocation, thereby influencing the overall allocation process and the final policy evaluation outcomes.

### H.2 Emergent Behaviors

In the dynamics of interactions involving LLM-based Agents, the agent’s behavior, whether deceptive or cooperative, is preceded by a shift in its psychological state. Corresponding to these behavioral patterns, there are two primary psychological states: doubt and trust, as illustrated in Fig. [9](https://arxiv.org/html/2410.14152v1#A8.F9 "Figure 9 ‣ H.1 Ablation Study on Memory Component ‣ Appendix H Case Study on LLM-based Agent ‣ SRAP-Agent: Simulating and Optimizing Scarce Resource Allocation Policy with LLM-based Agent").

The figure portrays two distinct scenarios: one involving a conversation between James and the stranger, Oliver, and the other between James and his friend, Emma. In the dialogue with Oliver, Oliver initially withholds all information regarding available houses to maximize his own benefits. This tactic leads James to adopt a more cautious approach, perceiving Oliver as a formidable opponent. Consequently, James becomes more cautious in the dialogue, perceiving Oliver as a formidable opponent. He refrains from discussing his choices and instead emphasizes the advantages of other communities. In stark contrast, the interaction between James and Emma, his friend, is characterized by a display of genuine sincerity and mutual trust. Such interactions underscore the proficiency of the SRAP-Agent in mirroring human-like psychological responses and thought processes.

Table 18: The prompt template of *Utterance Generation*

<svg class="ltx_picture" height="174.23" id="A8.T18.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,174.23) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="146.67" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">{concise role description} Here is your memory {memory} Your goal is to rent one house. For now, you want to discuss this with some acquaintances. {utterance plan} Your acquaintances include: {acquaintances} Your recent chat records with your acquaintances: {recent chats} {The example of group discuss response} !![IMPORTANT]: the information in EXAMPLE should NOT appear in response !! - Respond in this format: Thought: (You always think about what to do) Acquaintance: (Acquaintance name, could be a list or string) Output: (things you want to tell this Acquaintance in particular, stay consistent with your plan and thought) .. (this Thought/Acquaintance/Output repeat at most {acquaintance number} times!!) Respond in first person:</foreignobject></g></g></svg>

Table 19: The prompt template of *Communication Plan Generation*

<svg class="ltx_picture" height="91.21" id="A8.T19.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,91.21) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="63.65" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">{role description} You want to rent a house. For now, you want to discuss this with some acquaintances. {acquaintance description} Here is your memory {memory} The current situation of the renting system is: {system competitiveness description} Your personality is {personality} {goal} Respond in this format: {respond format} Respond in the second person:</foreignobject></g></g></svg>

Table 20: The prompt template of *Decision-making process*

<svg class="ltx_picture" height="141.02" id="A8.T20.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,141.02) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="113.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Choosing one house needs the following steps: 1\. Choose a community 2\. Choose the type of house 3\. Choose a house This is information you collected from previous conversations with others: {memory} {role description} You’re planning to choose one house. To choose a house that satisfies you, you are going to {task}. {house info} {thought hint} - If you made up your choice, respond in this format: Thought: ({thought type}) Action: Choose Action Input: {choose type}. - If you chose none of them, respond in this format: Thought: ({thought type}) Action: Give up Action Input: I choose none of these options.</foreignobject></g></g></svg>

Table 21: The prompt template of *Broadcasting*

<svg class="ltx_picture" height="185.65" id="A8.T21.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,185.65) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="158.09" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">{role description} Here’s your plan to publish info online: {plan} Here is your memory: {memory} You can publish house information online if you want to. The Info you posted should contain information about the community, house and The available community index should be one of [{community ids}]. - If you want to publish house information online, respond in this format: Thought: (your view on the information you want to publish) Action: Publish Community: (community index, should be one of [{community ids}]) Info: (The information you want to publish about this community, stay consistent with your plan and thought; Ensure that the information is specific and clear) - If you don’t want to publish anything respond in this format: Thought: (reason why you don’t want to publish anything) Action: Give up</foreignobject></g></g></svg>

Table 22: The prompt template of *Relation Evaluation*

<svg class="ltx_picture" height="257.25" id="A8.T22.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,257.25) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="229.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Your relationship with your acquaintances may include: Friend: A person with whom one shares a close and mutually supportive bond of affection and trust. Enemy: A person who is actively opposed or hostile to another, often due to conflicts or animosity. Competitor: Someone who engages in rivalry or competition with another, typically in the same field or for the same goal. Mate: A partner in a romantic or sexual relationship, often implying a deep emotional connection. Colleague: A person with whom one works or collaborates, typically within the same organization or profession. Stranger: An individual who is not known or familiar to someone, often encountered for the first time. Your task is to update your relation with {acquaintance name}, based on your previous view of {acquaintance name} and recent communication. {role description} Here is your memory: {memory} {relation} Here’s your recent communication with {acquaintance name}: {communication} - Respond in this format: My Relation with A: friend (friend/enemy/competitor/mate/colleague/stranger/..) A is an honest and trustworthy person, and I think he is worth making friends with. (My view of this person) Respond:</foreignobject></g></g></svg>

Table 23: The prompt template of *Memory Reflection*

<svg class="ltx_picture" height="141.02" id="A8.T23.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,141.02) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="113.46" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Progressively summarize new lines provided, adding onto the previous summary and returning a new summary. EXAMPLE Current summary: You think a large house is too big for your family. And you didn’t make a choice. New lines: Thought: The middle house can accommodate my family to live in and has high cost-effectiveness. Output: My choice is the middle house. New summary: You think a middle house can accommodate your family members, better than a large house. And you choose a middle house. END OF EXAMPLE Current summary: {summary} New lines of conversation: {new lines} New summary:</foreignobject></g></g></svg>

Table 24: The prompt template of *Memory Assessment*

<svg class="ltx_picture" height="224.04" id="A8.T24.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,224.04) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="196.49" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You’re {name}. You’re planning to choose one house. Your task is to use MEMORY to assess the credibility of the forum information and summarize the useful information in the forum information based on your previous summary. MEMORY: {memory} End of MEMORY Here’s the forum information: {forum info} [!Important!]: Keep in mind that you and your competitors are vying to rent a house. Both you and your competitors can share diverse information on the forum. And you get forum information from this platform. Remember to save the sequence number of the information you believe in in the summary content - Respond in this format: Trusted: (Summary of the useful information, which you assessed as trustworthy in the forum information) Suspicious: (Summary of the suspicious information, which you suspicious as trustworthy in the forum information; If there’s no suspicious information, simply return None) Reason: (why do other competitors say these things? Try to find a reasonable intention for their intention.) Respond:</foreignobject></g></g></svg>