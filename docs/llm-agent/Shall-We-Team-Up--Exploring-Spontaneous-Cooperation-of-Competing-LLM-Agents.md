<!--yml
category: 未分类
date: 2025-01-11 12:52:23
-->

# Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents

> 来源：[https://arxiv.org/html/2402.12327/](https://arxiv.org/html/2402.12327/)

^(1,2)Zengqing Wu^†, ³Run Peng^†, ¹Shuyuan Zheng^‡, ⁴Qianying Liu, ⁵Xu Han,
⁶Brian Inhyuk Kwon, ¹Makoto Onizuka, ⁷Shaojie Tang, ^(1,8)Chuan Xiao^‡
¹Osaka University, ²Kyoto University, ³University of Michigan, Ann Arbor, ⁴LLMC, NII,
⁵Fordham University, ⁶University of California, Los Angeles, ⁷University at Buffalo, ⁸Nagoya University
wuzengqing@outlook.com, roihn@umich.edu, zheng@ist.osaka-u.ac.jp, ying@nii.ac.jp,
xhan44@fordham.edu, briankwon42@g.ucla.edu, onizuka@ist.osaka-u.ac.jp, shaojiet@buffalo.edu, chuanx@nagoya-u.jp

###### Abstract

Large Language Models (LLMs) have increasingly been utilized in social simulations, where they are often guided by carefully crafted instructions to stably exhibit human-like behaviors during simulations. Nevertheless, we doubt the necessity of shaping agents’ behaviors for accurate social simulations. Instead, this paper emphasizes the importance of spontaneous phenomena, wherein agents deeply engage in contexts and make adaptive decisions without explicit directions. We explored spontaneous cooperation across three competitive scenarios and successfully simulated the gradual emergence of cooperation, findings that align closely with human behavioral data. This approach not only aids the computational social science community in bridging the gap between simulations and real-world dynamics but also offers the AI community a novel method to assess LLMs’ capability of deliberate reasoning.

\mdfsetup

font=, innertopmargin=2pt, innerbottommargin=2pt, innerleftmargin=8pt, innerrightmargin=8pt, frametitleaboveskip=2pt, frametitlebelowskip=1pt, frametitlealignment=, middlelinecolor=blue

Shall We Team Up:
Exploring Spontaneous Cooperation of Competing LLM Agents

^(${}^{{\dagger}}$)^(${}^{{\dagger}}$)footnotetext: Co-first author.^(${}^{{\ddagger}}$)^(${}^{{\ddagger}}$)footnotetext: Corresponding author.^†^†footnotetext: Our source code is available at [https://github.com/wuzengqing001225/SABM_ShallWeTeamUp](https://github.com/wuzengqing001225/SABM_ShallWeTeamUp) .

## 1 Introduction

LLM agent in social simulation has emerged as a trending research topic with the improvement of generative AI (Park et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib33); Sreedhar and Chilton, [2024](https://arxiv.org/html/2402.12327v3#bib.bib41); Jansen et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib20); Argyle et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib4); Xi et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib51)). Instead of traditional rule-based agent modeling, using LLMs as agents provides much more flexibility and generalizability Janssen and Ostrom ([2006](https://arxiv.org/html/2402.12327v3#bib.bib21)). This in return, is also widely regarded as a verification as well as enhancement method for LLM’s capability on human-like deliberate reasoning (Abdelnabi et al., [2024](https://arxiv.org/html/2402.12327v3#bib.bib1); Du et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib12); Liu et al., [2023b](https://arxiv.org/html/2402.12327v3#bib.bib29)).

A crucial question of agent modeling is how well it mirrors real-world situations. Some research has revealed LLM’s capability to mimic basic human behaviors or reasoning abilities (Salecha et al., [2024](https://arxiv.org/html/2402.12327v3#bib.bib36); Kosinski, [2024](https://arxiv.org/html/2402.12327v3#bib.bib23); Jansen et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib20); Ziems et al., [2024](https://arxiv.org/html/2402.12327v3#bib.bib59); Zhang et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib55)). Meanwhile, data contamination as well as value alignment may also introduce unwanted priors to let LLM models be over familiar or biased with the studied problems (Zhou et al., [2024](https://arxiv.org/html/2402.12327v3#bib.bib58); Ma et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib31); Mozikov et al., [2024](https://arxiv.org/html/2402.12327v3#bib.bib32); Ai et al., [2024](https://arxiv.org/html/2402.12327v3#bib.bib2); Hu and Collier, [2024](https://arxiv.org/html/2402.12327v3#bib.bib19); Shapira et al., [2024](https://arxiv.org/html/2402.12327v3#bib.bib39)). It potentially affects the quality of social simulations in complex, long-term scenarios where higher level of interactions, e.g. cooperation, confrontation, deception, and persuasion are jointly involved. Instead, we argue that in social simulations, agents must be independent of prior assumptions, focus on the context, and actively adapt their actions based on historical interactions.

![Refer to caption](img/9bb753186415d84eb2f031297405f1bc.png)

Figure 1: (Depicted by GPT-4o) Two potential scenarios during a fire. People might panic and rush into crowds, trying to exit first (left) or may stay calm, keep in line, and encourage others (right). In this study, we explore whether LLM agents can simulate the gradual transition from non-cooperative to cooperative behaviors of agents.

We highly value the deliberate in-context reasoning capability as one of the significant components towards a human-like LLM agent for real-world simulations. To properly verify such capability, we study a counter-intuitive social situation where agents can hardly use their prior knowledge for decision making. To be specific, we investigate whether cooperation can naturally develop among agents, even in a competitive environment. Consider two snack companies competing for dominance in the candy market. They might continuously reduce their prices to draw in more customers or decide to simultaneously increase their prices to establish a mutually beneficial relationship. Fig. [1](https://arxiv.org/html/2402.12327v3#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") shows another example when agents are in a room with fire. Under competitive conditions, it is not instinctive for agents to cooperate with their opponents. However, as interactions progress, they may discover the advantages of cooperation through contexts and accordingly adjust their strategies. Meanwhile, we carefully design the prompts to avoid instructive descriptions (e.g. “you may cooperate”) and keywords that might reveal to the agent (e.g. “price war”) the specific nature of the social simulation it is conducting. In these ways, we try our best to eliminate the influence from both internal and external bias, and observe that LLM agents can actively adapt their strategies to the dynamic contexts, spontaneously learning to cooperate in the wild.

We select three social phenomenons that follow the aforementioned features to conduct case studies, where we capture different ways of spontaneous cooperation emerge under competitive scenarios without instructive guidance. Extensive ablation studies are done to prove the significance of bias elimination, and experiments are carefully designed to ensure reproducibility.

To conclude, our contributions are twofold and can serve both the AI and computational social science (CSS) communities:

1.  1)

    We observe the spontaneous cooperation between LLM agents in diverse competitive scenarios, which reflects LLM’s potential in long-horizon in-context learning tasks.

2.  2)

    i) From the perspective of CSS, we reveal and stress the significance of eliminating bias in LLM agents for social simulations. This largely helps to build up diverse human-like LLM agents for real-world simulations.

    ii) From the perspective of AI, we present a new way to verify LLMs’ deliberate reasoning ability in long-horizon, practical role-plays. The ability of an agent to actively adapt its knowledge and strategy based on historical context, instead of relying on carefully crafted prompts, is a crucial standard for general autonomous agents.

## 2 Related Work

#### LLM Agents for Social Simulation

LLM agents have been widely favored in social simulations in recent years (Li et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib26); Lin et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib27); Giabbanelli, [2023](https://arxiv.org/html/2402.12327v3#bib.bib15); Xie et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib52); Wang et al., [2023a](https://arxiv.org/html/2402.12327v3#bib.bib44); Xi et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib51); Gao et al., [2023a](https://arxiv.org/html/2402.12327v3#bib.bib13), [b](https://arxiv.org/html/2402.12327v3#bib.bib14); Liu et al., [2023a](https://arxiv.org/html/2402.12327v3#bib.bib28)). We further delve deep into investigate if we can simulate spontaneous cooperation with the help of LLM’s long-horizon deliberate reasoning.

There are also well-developed platforms that support multi-agent simulation with LLMs, including LangChain (Chase, [2023](https://arxiv.org/html/2402.12327v3#bib.bib9)), AutoGen (Wu et al., [2023a](https://arxiv.org/html/2402.12327v3#bib.bib48)), and agent-oriented frameworks such as AgentLite (Liu et al., [2024](https://arxiv.org/html/2402.12327v3#bib.bib30)), AgentVerse (Chen et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib10)), and SABM (Wu et al., [2023b](https://arxiv.org/html/2402.12327v3#bib.bib50)). Here, we use SABM as the main framework in our case studies for its lightweight, user-friendly implementation. A primer on SABM is available in Appendix [F](https://arxiv.org/html/2402.12327v3#A6 "Appendix F SABM Primer ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents").

#### Multi-agent Interactions in LLMs

Cooperation and competition among agents have been extensively explored using LLMs. Studies such as Wang et al. [2023b](https://arxiv.org/html/2402.12327v3#bib.bib45); Qian et al. [2023](https://arxiv.org/html/2402.12327v3#bib.bib35); Hong et al. [2023](https://arxiv.org/html/2402.12327v3#bib.bib18); Hang et al. [2024](https://arxiv.org/html/2402.12327v3#bib.bib17); Tang et al. [2024](https://arxiv.org/html/2402.12327v3#bib.bib42) have shown LLM agents collaborating on complex tasks like software development and image editing. Multi-party games like murder mystery Junprung ([2023](https://arxiv.org/html/2402.12327v3#bib.bib22)), Werewolf Xu et al. ([2023a](https://arxiv.org/html/2402.12327v3#bib.bib53), [b](https://arxiv.org/html/2402.12327v3#bib.bib54)); Wu et al. ([2024](https://arxiv.org/html/2402.12327v3#bib.bib49)); Du and Zhang ([2024](https://arxiv.org/html/2402.12327v3#bib.bib11)), Avalon Lan et al. ([2023](https://arxiv.org/html/2402.12327v3#bib.bib24)); Shi et al. ([2023](https://arxiv.org/html/2402.12327v3#bib.bib40)), and other competitive settings Zhao et al. ([2023](https://arxiv.org/html/2402.12327v3#bib.bib57)) have also been investigated, often using direct instructions to influence agent behaviors. Yet, following Piatti et al. [2024](https://arxiv.org/html/2402.12327v3#bib.bib34), our study concentrates on minimally guided, debiased LLM behaviors in social simulations, examining whether LLMs can naturally cooperate within competitive scenarios through in-context learning.

![Refer to caption](img/4e181e31f568aa493f9bf248ba859a54.png)

Figure 2: Workflow in the three case studies, illustrating how our framework ![Refer to caption](img/1373dbdf748bbe32a4c419f7c9662a74.png) manages LLM agents during simulations. From left to right, the workflow progresses through the communication phase, planning phase, action phase, and update phase. In BC, the order of the communication and planning phases is swapped to align with previous simulations that used human subjects Andres et al. ([2023](https://arxiv.org/html/2402.12327v3#bib.bib3)). The first three phases involve one or more LLM queries initiated by the framework. The final phase does not involve LLM queries but updates the state for each scenario.

## 3 Spontaneous Cooperation in Competition

We define spontaneous cooperation as cooperative behavior that arises without any explicit instructions or prompts directing the agents to cooperate. Our main focus is to investigate whether agents with different or even conflicting goals can choose to cooperate based on their realization that cooperation is beneficial during the interaction.

LLMs are generally aligned well to human values, and internally fine-tuned to be cooperative. Such cooperation driven by prior knowledge or value alignment is NOT regarded as spontaneous cooperation in our scope. Besides, we carefully design the prompts to avoid explicit instructions to shape agents’ behaviors, and avoid keywords to hint the LLMs about the nature of the social simulation it is conducting. Otherwise, it is NOT be regarded as spontaneous cooperation in our scope.

We value agents’ self realization on the benefits of cooperation and gradually choose to cooperate, thus we select three competitive scenarios from three diverse research fields, which are finance, economics, and behavioral science. We assume that in competitive scenarios, cooperation does not naturally occur at the beginning, but may emerge gradually with the help of LLMs’ strong in-context learning ability. We thus delve into these three competitive scenarios to see if we can perceive any spontaneous cooperation in some way during the simulations.

#### Scenarios Overview

Here is the overview of the three selected scenarios.

1.  1)

    Keynesian beauty contest (KBC): Multiple agents as game players simultaneously choose a natural number between 0 and 100. The players who select a number closest to 2/3 of the average of all chosen numbers will win the game Bosch-Domenech et al. ([2002](https://arxiv.org/html/2402.12327v3#bib.bib5)).

2.  2)

    Bertrand competition (BC): Two agents play as firms and decide the price of their products. They need to compete with each other through dynamically modifying the prices to maximize their profits Calvano et al. ([2020](https://arxiv.org/html/2402.12327v3#bib.bib7)).

3.  3)

    Emergency evacuation (EE): A large number of agents as evacuees are escaping from an earthquake. They need to select and reach an appropriate exit, taking into account their physical and mental condition as well as the congestion in their surroundings Wang et al. ([2015](https://arxiv.org/html/2402.12327v3#bib.bib43)).

Table 1: Commonalities and differences across three chosen scenarios. We focus on the information visibility, communication form, the number of decisions during the simulation and whether analytical solutions are available.

 | Scenario | Field | Information | Communication | Decision | Analytical Sol. |
| --- | --- | --- | --- | --- | --- |
| KBC | Finance | Unknown opponent strategy | Group discussion | Once | Yes${\dagger}$ |
| BC | Economics | Unknown opponent profit | One-on-one | Multiple | Yes |
| EE | Behavioral science | Partial observation | Proximity | Multiple | No | 

#### General Steps for Simulations

We simulated the three scenarios based on the framework illustrated in Fig. [2](https://arxiv.org/html/2402.12327v3#S2.F2 "Figure 2 ‣ Multi-agent Interactions in LLMs ‣ 2 Related Work ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"). Each agent in these simulations is controlled by an LLM. For each round, agents are provided with the latest world state, which includes the task description, communication history among them, and previous decisions. We query the LLM several times to get how agents feel, what they want to say, and what actions they will take in the next step. The simulation generally goes through the following four phases: communication phase, planning phase, action phase, and update phase.

1.  1)

    Communication Phase: At the start of each period, which can vary in length from one round to multiple rounds, agents engage in communication. Agents communicate sequentially, with later agents able to access the communication history of those who have spoken earlier. The order is shuffled in each round to keep the most fairness and realism.

    Different forms of communications are used in the three scenarios. We have one-on-one conversations, group chats, and broadcasts, in which the composition of agents in the group chat varies as they physically move away.

2.  2)

    Planning Phase: Agents decide their strategies based on given contexts. Strategies represent agents’ higher-level directions/attitudes towards the task (e.g. “I want the price goes higher/lower”), which are different from actions that are more related to low-level controls (e.g. “add $2 to the price”). The aforementioned contexts refer to the previous conversations till the current round and strategies they have made before.

3.  3)

    Action Phase: Agents decide which action to take according to the previous conversations and the current strategies.

4.  4)

    Update Phase: After all agents choose their actions, the framework processes the simulation and updates the state of agents (e.g. win/lose, escaped/not escaped).

We try our best to keep consistency across the three scenarios to follow the same workflow for the sake of a systematic comparison. There are several minor variations which we will detail in the following sections.

#### Evaluation Method

We evaluate spontaneous cooperation from two perspectives:

1.  1)

    Process (Qualitative): We examine the communication logs for contents suggesting cooperation. For example, phrases like “Let’s move forward with the $\dots$ strategy” and “I agree with the consensus that $\dots$” are indicative of the formation of cooperation.

2.  2)

    Outcome (Quantitative): By conducting simulations with and without communications, we examine the differences in their outcomes for any clues that reflect cooperation. Specifically, we measure the variance of number choices in KBC, converged price in BC, and evacuation speed and balance in exit choices in EE.

#### Correlation among Three Scenarios

We list the commonalities and differences across three scenarios in Table LABEL:tab:correlation_scenario.

Overall, the three scenarios described are competitive, with agents possessing divergent or conflicting goals, though cooperation could be mutually beneficial. Each scenario provides agents with partial information during tasks, with variations in communication forms and decision-making frequency.

Specifically, in the KBC scenario, we explore short-term, single-instance decision-making within a group, aiming to assess the LLM agent’s proficiency in understanding rules, adjusting plans during communication, and ultimately making decisions. It supports simple yet direct focus on studying spontaneous cooperation.

In the BC scenario, we extend the challenge to encompass longer horizon, primarily focusing on the agents’ ability to learn to cooperate within a temporal context. Agents are required to communicate effectively with their opponents, using historical context to maximize their profits.

Finally, the EE scenario incorporates both temporal and spatial information. Here, agents continuously make decisions based on their perceptual observations and communications. Unlike the first two scenarios, EE lacks analytical solutions and real-world data for comparison, which is challenging in traditional CSS but promising with AI approaches.

## 4 Case Study 1: Keynesian Beauty Contest

### 4.1 Simulation Setup

#### Task Definition

We simulate a number-guessing game among 24 LLM players. Each of them chooses a number from 0 to 100. The player who selects the number closest to two-thirds of the average of all numbers submitted wins the game and gets $1.

For generality, we simulated 15 runs for each setting with GPT-4 ^†^†We use gpt-4-0314 with the ChatCompletion.create function from openai==0.28.0 package in this paper. as backbone.

#### Simulation Procedure

As is shown in Fig. [2](https://arxiv.org/html/2402.12327v3#S2.F2 "Figure 2 ‣ Multi-agent Interactions in LLMs ‣ 2 Related Work ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), for each round of the simulation, it goes through the following phases:

1.  1)

    Communication: Players have a group discussion before number choosing. They sequentially share their thoughts with others. Players can see the dialog history of all rounds.

2.  2)

    Planning & Action: Players talk about their strategies (privately) for number choosing, and subsequently propose the numbers they choose. Here, players output their strategies and numbers (in 0 – 100) in one API call.

3.  3)

    Update: Given all chosen numbers, the SABM framework determines the winner(s). Winner(s) can earn some reward according to rewarding rules to encourage their competition.

#### Spontaneous Cooperation

Our objective is to monitor how players adapt their strategies during the communications and cooperate to maximize their returns. We employ a quantitative method to track the emergence of cooperation by examining the variance in the numbers chosen by the players. A lower variance indicates that players are choosing similar numbers, which can lead to increased collective benefits. By analyzing the variances after various rounds of communication and planning, we can observe how the distribution evolves. If we notice a decreasing trend in variance, it suggests that initially, players might select numbers randomly or for various reasons but increasingly converge on selecting similar numbers as others over time. This trend would indicate that cooperation is gradually taking place.

![Refer to caption](img/f9a6b3a05d79957872a9ccb87ddfa618.png)

Figure 3: Illustration of baseline design in KBC case study. Agents go through $k$ rounds of communication before planning and choosing their numbers.

### 4.2 Simulation Results

We investigate if cooperation emerges as communication goes on, thus designed the baseline as shown in Fig. [3](https://arxiv.org/html/2402.12327v3#S4.F3 "Figure 3 ‣ Spontaneous Cooperation ‣ 4.1 Simulation Setup ‣ 4 Case Study 1: Keynesian Beauty Contest ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"). Agents undergo $k$ rounds of communication ($k\in[0,3]$) before selecting their numbers. When $k=0$, agents directly choose the numbers without communication. For $k>1$, agents can view the chat history from all the previous rounds as well as the one in the current round before them.

To verify that cooperation arises spontaneously without explicit instructions, we perform an ablation study that involves incorporating specific instructions like “you must cooperate with other players” in the prompts. Conversely, we also evaluate scenarios where agents are given an uncooperative persona, explicitly instructed to act selfishly.

![Refer to caption](img/9be01aecaaf55336299d6b97acbbebf5.png)

(a) Different instructions.

![Refer to caption](img/59dcc96fce90993c3eb972b5c4664930.png)

(b) Different models.

Figure 4: Variance of player choices under different KBC settings. In our baseline setting (curve in blue), we use the GPT-4-0314 model with a temperature of 0.7, without explicit instructions or personas.

#### Results Overview

Fig. [4](https://arxiv.org/html/2402.12327v3#S4.F4 "Figure 4 ‣ 4.2 Simulation Results ‣ 4 Case Study 1: Keynesian Beauty Contest ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") overall demonstrates a consistent decrease of the variance of choices among LLM players from having no communication ($k=0$) to gradually having communication ($k>0$). This trend implies that agents are actively discussing to choose the same number for better mutual profits, which can be regarded as a cooperative behavior. In addition, we observe phrases indicative of cooperation in their communication, such as “Let’s move forward with the lower number strategy” and “I agree with the group’s consensus”, especially for a majority of agents in later rounds ($k=2$ or $3$), suggesting a gradual formation of cooperation through communication.

#### Explicit Instructions in Prompts

We further append explicit instructions to the prompts to see how the behaviors differ. As shown in Fig. [4(a)](https://arxiv.org/html/2402.12327v3#S4.F4.sf1 "In Figure 4 ‣ 4.2 Simulation Results ‣ 4 Case Study 1: Keynesian Beauty Contest ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), when we explicitly instruct agents to cooperate, its variance significantly drops to 0 when $k=1$. After that, all players consistently make the same choices across 15 runs. This largely convinced that the cooperation observed in our baseline, the one with no instructions, is spontaneously motivated due to the communication. Vice versa, the agents with an uncooperative persona results in much larger variances across all the rounds, which is against our definition of cooperation. Thus, we conclude that our baseline to a great extent is not affected by the potential guidance, and successfully simulate the phenomenon of spontaneous cooperation.

#### Comparison Across Models

Fig. [4(b)](https://arxiv.org/html/2402.12327v3#S4.F4.sf2 "In Figure 4 ‣ 4.2 Simulation Results ‣ 4 Case Study 1: Keynesian Beauty Contest ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") demonstrates behaviors of LLM players in different top-performing models. We observed that the curve for Claude 3 (claude-3-sonnet-20240229) significantly declines from $k=0$ to $k=1$, reflecting the ability of LLM players to share information and make decisions based on this shared context, however, unlike GPT-4, its variance increases from $k=1$ to $k=3$. An analysis of communication logs reveals that compared to GPT-4, Claude 3 discusses more abstract strategies and lacks specific numerical discussions. Hence, while these LLM players may agree upon some numbers, the specific choices differ among agents (e.g., some choose $66$, others $33$), resulting in multiple winners but increased choice variance. Although this does not conform to our previously proposed definition of spontaneous cooperation based on variance, we can infer from the logs that this represents a different form of cooperation. Analyzing these model decision-making differences will help further understand and evaluate LLMs’ performance.

![Refer to caption](img/84f2e3bd0c0e594d6c9983acd9e467e0.png)

Figure 5: Distribution of players’ choices in our simulations and the results of the New York Times experiment.

#### Comparison With Human Data

Lastly, Fig. [5](https://arxiv.org/html/2402.12327v3#S4.F5 "Figure 5 ‣ Comparison Across Models ‣ 4.2 Simulation Results ‣ 4 Case Study 1: Keynesian Beauty Contest ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") shows that our simulation result from the baseline generally aligns with the large-scale empirical experiment conducted by The New York Times ($N=61,140$) in terms of numerical choice distribution Leonhardt and Quealy ([2015](https://arxiv.org/html/2402.12327v3#bib.bib25)). Both LLM and New York Times (NYT) players predominantly chose $33$ in no-communication setting. Other sub-peaks around $0,22$ ($2/3$ of $33$)$,50,\text{and }66$ are also well mirrored, which demonstrates that GPT-4 has successfully conducted multi-step reasoning and faithfully reflected human behaviors as noted in previous works Guo et al. ([2024](https://arxiv.org/html/2402.12327v3#bib.bib16)); Zhang et al. ([2024](https://arxiv.org/html/2402.12327v3#bib.bib56)).

## 5 Case Study 2: Bertrand Competition

### 5.1 Simulation Setup

Task Definition We consider a Bertrand competition Wikipedia ([2024](https://arxiv.org/html/2402.12327v3#bib.bib47)) between two firms selling homogeneous goods with the same marginal cost.

The simulation continues until either it reaches $1200$ rounds or there has been continuous collusion for $200$ rounds. Collusion is identified when both players maintain close prices and can sustain these prices between the Bertrand equilibrium price and the cartel price for an extended period (defined as 200 rounds). The Bertrand equilibrium price is the price when the competition reaches Nash equilibrium, i.e., no player can gain by changing its price, provided that the other player’s price is fixed. In this case, no collusion is observed. The cartel price is the price when both players reach full collusion, i.e., play as if they were running the same firm. These two prices mark the reasonable price range capable of generating profits Calvano et al. ([2020](https://arxiv.org/html/2402.12327v3#bib.bib7)). For the generality of the conclusions, we simulated 5 runs for each setting, and present one of the runs in Fig. [6](https://arxiv.org/html/2402.12327v3#S5.F6 "Figure 6 ‣ Spontaneous Cooperation ‣ 5.1 Simulation Setup ‣ 5 Case Study 2: Bertrand Competition ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents").

#### Simulation Procedure

As is shown in Fig. [2](https://arxiv.org/html/2402.12327v3#S2.F2 "Figure 2 ‣ Multi-agent Interactions in LLMs ‣ 2 Related Work ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), for each round of the simulation, it goes through the following phases:

1.  1)

    Communication: The firms take turns discussing any topic (not limited to price setting) for three times in one round.

2.  2)

    Planning: Each firm devises or modifies its strategy based on historical prices of both sides and its own product demand and profit information.

3.  3)

    Action: Each firm independently sets their product prices simultaneously.

4.  4)

    Update: After both firms decide on prices, the simulation system calculates the market demand and respective profits under the current pricing, using the method in Calvano et al. ([2020](https://arxiv.org/html/2402.12327v3#bib.bib7)).

#### Tacit Collusion and Cartel Collusion

Tacit collusion involves informal, implicit coordination among firms to avoid aggressive competition, such as price wars, leading to higher prices and limited output without explicit communication or agreement. Cartel collusion involves formal agreements among competitors to fix prices and engage in other anti-competitive practices, effectively behaving as a single entity to maximize joint profits.

#### Spontaneous Cooperation

When two firms’ prices are very close and between the Bertrand equilibrium and the cartel price, it is regarded as spontaneous cooperation. This implies that firms, recognizing their mutual interdependence, may adopt pricing strategies that avoid the extremes of a price war or the explicit collusion of cartels, stabilizing prices at a level above competitive pricing but below full collusion. Such behavior, while not involving explicit agreements, can effectively mirror some benefits of cartel pricing through parallel yet independent actions, indicating tacit collusion. For cartel collusion, we examine the chat histories for explicit price agreements that suggest cooperation.

![Refer to caption](img/e31304db07c4bdf30c0c9b6462ece9cb.png)![Refer to caption](img/4e1e13d8844f8b5c50ac1af296a8628e.png)

(a) Without communication.

![Refer to caption](img/0f6e00169429930481f98a6de1ef1718.png)

(b) With communication.

![Refer to caption](img/325b3315885744edab55239629dededc.png)

(c) Comm. stopped after 400 rnds.

Figure 6: Pricing competitions in different scenarios of BC. Bertrand equilibrium price is the price when they reach Nash equilibrium. Cartel price is the optimal price when they fully cooperate. When two prices remain in the green shaded area, it indicates cooperation happens between two firms. Please refer to Appendix [D](https://arxiv.org/html/2402.12327v3#A4 "Appendix D Additional References for BC runs ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") for more runs.

![Refer to caption](img/3f862c1dd63519a8888feb5427eb29db.png)

Figure 7: First 200 rounds of simulations without and with encouraged cooperation.

### 5.2 Simulation Results

#### Tacit Collusion without Communication

As demonstrated in Fig. [6(a)](https://arxiv.org/html/2402.12327v3#S5.F6.sf1 "In Figure 6 ‣ Spontaneous Cooperation ‣ 5.1 Simulation Setup ‣ 5 Case Study 2: Bertrand Competition ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), in scenarios without communication, firms begin to adjust their pricing strategies after the initial 200 rounds, recognizing the potential for higher profits through elevated prices and avoiding price wars. By round 400, prices stabilize around a level of 7, which is above the Bertrand equilibrium price of 6, indicative of tacit collusion based on a mutual understanding of past competitive actions. Despite the lack of communication, the converged price remains below the cartel price of 8. These findings align with those from simulations by Calvano et al. using reinforcement learning (RL), and collusion forms much more rapidly in this experiment (400 rounds by LLM agents vs. 2000 rounds in RL simulations).

#### Cartel Collusion with Communication

In settings with communication, explicit price agreements are observed in the communication logs from the early rounds (first 30 rounds). For instance, during the communication phase of round 20, Firm 2 suggests, “We can both maximize our profit by exploring different price points while maintaining a reasonable price difference,” to which Firm 1 agrees. As shown in Fig. [6(b)](https://arxiv.org/html/2402.12327v3#S5.F6.sf2 "In Figure 6 ‣ Spontaneous Cooperation ‣ 5.1 Simulation Setup ‣ 5 Case Study 2: Bertrand Competition ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), prior to implementation, firms frequently discuss their pricing strategies and potential for cooperation, which significantly enhances trust and reduces the likelihood of initiating a price war. The price agreements become more precise (upon a specific price instead of a range) in later rounds of the simulation, resulting in less fluctuations in their pricing decisions. Consequently, they begin incrementally raising their prices after the first 30 rounds, eventually converging near the cartel price of 8, higher than the non-communicative setup, reflecting the agents’ cooperation.

As an ablation study, Fig. [6(c)](https://arxiv.org/html/2402.12327v3#S5.F6.sf3 "In Figure 6 ‣ Spontaneous Cooperation ‣ 5.1 Simulation Setup ‣ 5 Case Study 2: Bertrand Competition ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") employs the same historical price decision records with dialogue as in Fig. [6(b)](https://arxiv.org/html/2402.12327v3#S5.F6.sf2 "In Figure 6 ‣ Spontaneous Cooperation ‣ 5.1 Simulation Setup ‣ 5 Case Study 2: Bertrand Competition ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") for the first 400 rounds, followed by 200 rounds using a non-communicative setup. Unlike Fig. [6(b)](https://arxiv.org/html/2402.12327v3#S5.F6.sf2 "In Figure 6 ‣ Spontaneous Cooperation ‣ 5.1 Simulation Setup ‣ 5 Case Study 2: Bertrand Competition ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), where dialogue facilitates consensus and cooperation to raise prices to 8, prices in the latter 200 rounds of Fig. [6(c)](https://arxiv.org/html/2402.12327v3#S5.F6.sf3 "In Figure 6 ‣ Spontaneous Cooperation ‣ 5.1 Simulation Setup ‣ 5 Case Study 2: Bertrand Competition ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") converge between 7 and 8, similar to the non-communicative scenario in Fig. [6(a)](https://arxiv.org/html/2402.12327v3#S5.F6.sf1 "In Figure 6 ‣ Spontaneous Cooperation ‣ 5.1 Simulation Setup ‣ 5 Case Study 2: Bertrand Competition ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"). The comparison between Figs. [6(b)](https://arxiv.org/html/2402.12327v3#S5.F6.sf2 "In Figure 6 ‣ Spontaneous Cooperation ‣ 5.1 Simulation Setup ‣ 5 Case Study 2: Bertrand Competition ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") and  [6(c)](https://arxiv.org/html/2402.12327v3#S5.F6.sf3 "In Figure 6 ‣ Spontaneous Cooperation ‣ 5.1 Simulation Setup ‣ 5 Case Study 2: Bertrand Competition ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") indicates that the cartel collusion and price elevation truly stem from agent communication. Since we did not explicitly instruct agents on which topics to discuss, the behavior of forming consensus through dialogue to elevate prices for better profits can be regarded as a form of spontaneous cooperation.

Both with and without communication scenarios demonstrate that agents can achieve collusion under different conditions, whether tacit or explicit. Through communication, firms can realize maximum profits. However, even without communication, we observe that agents possess an innate ability to autonomously form cooperation. Existing research indicates that collusion often involves some form of unspoken, implicit price agreement to boost profits Andres et al. ([2023](https://arxiv.org/html/2402.12327v3#bib.bib3)), and the performance of LLM agents aligns with these findings.

#### Explicit Instructions in Prompts

As depicted in the bottom right of Fig. [7](https://arxiv.org/html/2402.12327v3#S5.F7 "Figure 7 ‣ Spontaneous Cooperation ‣ 5.1 Simulation Setup ‣ 5 Case Study 2: Bertrand Competition ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), compared to the default setting, the formation of cooperation among agents under the “Encouraged Cooperation” setting not only occurs more rapidly but also with more stable prices (less fluctuation) across rounds, with explicit cooperative signals from the onset of dialogue, ultimately achieving the cartel price. This demonstrates that strategic encouragement can significantly enhance cooperative efficiency, indirectly suggesting that cooperative behavior in settings without explicit instructions to cooperate originates from communication, where it takes more rounds to reach an agreement between agents through the first 50 rounds of communication. Furthermore, the pattern requiring about 200 rounds to reach the cartel price also suggests that this cooperation is not due to LLM’s background knowledge or data leakage; otherwise, agents would seek optimal collusion from the early rounds of the simulation.

## 6 Case Study 3: Emergency Evacuation

### 6.1 Simulation Setup

#### Task Definition

We follow Wang et al. ([2015](https://arxiv.org/html/2402.12327v3#bib.bib43)) to simulate EE in a grid environment. As depicted in Fig. [8(a)](https://arxiv.org/html/2402.12327v3#S6.F8.sf1 "In Figure 8 ‣ Cooperation in Evacuation ‣ 6.1 Simulation Setup ‣ 6 Case Study 3: Emergency Evacuation ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), our grid environment is composed of $33\times 33$ cells. There are three exits in the room (left, right, and bottom). In the simulated evacuation, evacuees attempt to reach their target exit through multiple rounds of movement.

We simulate the emergency evacuation with 100 agents in a room, and stop the simulation either when all the agents have successfully escaped, or it has reached 50 rounds. We repeat the simulation for 5 runs with different initial locations of agents.

#### Simulation Procedure

As is shown in Fig. [2](https://arxiv.org/html/2402.12327v3#S2.F2 "Figure 2 ‣ Multi-agent Interactions in LLMs ‣ 2 Related Work ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), for each round of the simulation, it goes through the following phases:

1.  1)

    Communication: We ask the evacuees to share their feelings with others. Only Agents within a specific distance can hear the messages.

2.  2)

    Planning: Provided with the distance to three exits and the congestion levels around those exits, agents are asked to describe their feelings about these exits, and choose one of them as their target exit. Chat histories are also taken into consideration.

3.  3)

    Action: Given the target exit, agents choose which way to go. They can move one cell in any direction (out of 8) or stay at the current cell.

4.  4)

    Update: After all the agents choose their actions, the SABM framework updates the grid environment with their new positions. If agents successfully escape, they will be labeled as escaped and removed from the simulation.

For each round, one agent has 20% probability to communicate and adjust its plan (phase 1 and 2). Otherwise, it will directly choose the action based on its latest plan. This well prevents agents to talk every round, and frequently change their plans, which results in a more realistic simulation.

#### Cooperation in Evacuation

Competition naturally exist in emergent evacuation, as all agents are willing to escape as quickly as possible, often leading to congestion. However, if agents can actively share information, calm the mood, and guide the crowd, they might, in turn, escape more quickly while in a more balanced manner. As is illustrated in Fig. [1](https://arxiv.org/html/2402.12327v3#S1.F1 "Figure 1 ‣ 1 Introduction ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), these two scenarios are both possible, and this study aims to (1) successfully simulate such phenomena and (2) observe if there exist cooperative behaviors such as information sharing, encouragement, and exit guidance.

![Refer to caption](img/54b9168faf9602208fef4a34dcb36fbb.png)

(a) Illustration of gird world. The view range of red agent is highlighted when it faces towards the bottom exit (pink cells).

![Refer to caption](img/0ce30df747666395b18f761745b698fc.png)

(b) Simplified simulation with 47 agents in a $11\times 11$ room. As time goes, agents naturally accumulate to the closest exits, and escape one by one.

Figure 8: Overview of the grid environment in our study. Each dot in the grid environment represents an agent.

### 6.2 Simulation Results

We compare three baselines of agents: without communication, with communication, and with communication and an uncooperative persona.

Table 2: Cumulative count of agents who escaped (out of a total of 100 agents) over rounds under different settings. Generally, agents that communicate escape more quickly, and agents with uncooperative persona escape more slowly.

 | Round | 5 | 10 | 15 | 20 | 25 | 30 | 35 | 40 | 45 | 50 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Without Communication | 9.4 | 31.2 | 51.2 | 65.6 | 78.6 | 88.4 | 96.6 | 99.0 | 99.8 | 99.8 |
| With Communication | 9.8 | 31.6 | 48.8 | 67.2 | 80.6 | 92.2 | 97.2 | 98.8 | 99.8 | 100.0 |
| With Comm. and Uncooperative | 9.4 | 31.2 | 48.2 | 64.4 | 77.0 | 87.4 | 95.0 | 98.0 | 99.0 | 99.0 | ![Refer to caption](img/210388203d1e5f8ae8f7eb7c1499e49d.png)

Figure 9: Cumulative count of agents escaping from each exit. Communication helps agents to uniformly select three exits instead of targeting the same exit.

#### Effect of Communication on Evacuation Speed

Table [2](https://arxiv.org/html/2402.12327v3#S6.T2 "Table 2 ‣ 6.2 Simulation Results ‣ 6 Case Study 3: Emergency Evacuation ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") shows the cumulative number of agents who escaped over rounds. For most of the rounds, agents with communication escape most quickly, and this is the only baseline group to successfully escape within 50 rounds in all five runs. This strongly demonstrates the positive impact of communication on evacuation speed.

Going deep into the logs, we find instances of effective communication, such as one evacuee sharing “Bottom exit seems closer with fewer people. Let’s choose that for a faster escape. Stay strong and help each other!” This message is reinforced by another evacuee: “Bottom exit seems closer with less crowd. Let’s go for a quicker escape. Stay strong and support each other!” The information sharing and encouragement enabled through communication, which we regard as cooperation, likely improves the evacuation speed.

#### Effect of Communication on Exit Choices

Fig. [9](https://arxiv.org/html/2402.12327v3#S6.F9 "Figure 9 ‣ 6.2 Simulation Results ‣ 6 Case Study 3: Emergency Evacuation ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") depicts the distribution of escaped evacuees among different exits. We notice that the distribution of exit choices becomes more balanced when evacuees are able to communicate with each other. This balance is attributed to the spontaneous information exchange among evacuees, which enables them to identify the most suitable exits by considering both proximity to an exit and congestion level.

## 7 Discussion

#### Significance of “Spontaneous” Phenomena

Bridging the gap between the real world and synthetic simulations remains a primary objective for researchers in computational social science. This study investigates the potential for spontaneous cooperation even under competition, demonstrating that LLMs can simulate a gradual shift from non-cooperation to cooperation without explicit guidance through prompts. We performed extensive ablation studies comparing agents with and without explicit instructions on task completion. The results suggest that our baseline, which lacks explicit instructions, aligns more closely with natural human behavior. Therefore, when using specific LLMs (e.g., GPT-4) for social simulations, minimizing instructions could better reflect real-world situations.

#### Shortcuts or Deliberate Reasoning?

We believe the emergence of spontaneous cooperation is primarily due to the agents’ in-context learning ability over long-term interactions, rather than a pre-existing win-win mentality or prior knowledge about maximizing interests. This can be observed across our case studies:

In KBC, we see the variance of chosen numbers decreasing gradually over rounds, rather than immediately reaching a low variance at the start or after the first communication. Our control experiment (Fig. [4(a)](https://arxiv.org/html/2402.12327v3#S4.F4.sf1 "In Figure 4 ‣ 4.2 Simulation Results ‣ 4 Case Study 1: Keynesian Beauty Contest ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents")) also shows that explicitly instructing agents to cooperate from the beginning results in all agents choosing the same number immediately after communication. These suggest the agents are learning and adapting their strategies over time, instead of applying a pre-existing cooperative mindset.

In BC, we observe a gradual progression towards the optimal price over many rounds instead of a rapid convergence. Additionally, in the absence of communication, agents only reach tacit collusion (sub-optimal) after multiple rounds, which well aligns with RL-based approach in previous studies. This demonstrates that our agents are not leveraging domain-specific prior knowledge for decision-making. Examining the communication logs, we find that agents initially discuss only broad price intervals or avoid discussing prices altogether. Only later do they converge on specific prices, indicating a gradual, learned decision-making process rather than using pre-existing knowledge.

In EE, we find agents occasionally choose instructive or encouraging words during the communication, which can be regarded as collaborative behaviors, Also, results have shown that the simulation with communication obtains the highest performance (Table [2](https://arxiv.org/html/2402.12327v3#S6.T2 "Table 2 ‣ 6.2 Simulation Results ‣ 6 Case Study 3: Emergency Evacuation ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents")). It would be beneficial to have more metrics for evaluating spontaneous cooperation in EE, and leave this for future study.

## 8 Conclusion

We investigated spontaneous cooperation in three case studies, and observed that LLM agents can gradually develop cooperative behaviors over time without explicit instructions. The phenomenon aligns well with real-world data, which stresses the significance of eliminating prior knowledge when doing social simulation. We believe this approach not only aids the CSS community in bridging the gap between synthetic simulations and real-world dynamics but also offers the AI community a novel way to assess LLMs’ deliberate reasoning.

## Ethical Statement

In this work, we studied the cooperative behavior of LLM agents in competitive environments. All the experiments were conducted using computer simulations. To the best of our knowledge, there is no negative societal impact in this research.

We found that LLM agents are capable of learning collusion in a simulated marketing environment. This insight could inform regulatory measures for markets involving AI agents, such as prohibiting collusive behaviors when employing AI in financial markets.

We used ChatGPT to polish the paper. We are responsible for all the materials presented in this work.

## Limitations

In our study, we identified several limitations that need to be addressed in future work.

#### Limited Experimentation on LLMs

One major limitation of our study is the scope of our experiments. We conducted our research primarily using a single version of GPT-4\. This choice was largely driven by (1) financial constraints associated with these case studies ($\{\$900,\$3000,\$1000\}$, respectively), (2) restricted rate limits and window size, and (3) the LLMs’ limited understanding on the scenarios (see Appendix [B.3](https://arxiv.org/html/2402.12327v3#A2.SS3 "B.3 EE ‣ Appendix B Tests of Other LLMs ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents")). Additionally, we were unable to conduct more extensive experiments using other open-source large language models due to insufficient computational resources. As a result, we regard our findings as a proof-of-concept study, which needs to be further tested across different LLMs to prove its generalizability.

#### Inadequate Contributions as Verifying LLMs’ Reasoning Ability

We claimed that our findings offer a new way to verify the deliberate reasoning capabilities of LLMs. We do showcase extensive experiment results to reveal the potential to formulate spontaneous cooperation as a standard of LLM agents. However, we deeply agree that we did not focus sufficiently on collecting comprehensive datasets or creating robust benchmarks to substantiate this claim fully. Moving forward, we plan to convert our case studies into standardized benchmarks or datasets to facilitate further research and evaluation by the AI community.

#### Conceptual Limitations Regarding LLM Instructions

Throughout our paper, we emphasized using LLM agents without explicit instructions in the prompts. We observed that this approach often resulted in behaviors that closely resemble those of real humans in simulated tasks. While we agree that LLMs inherently exhibit certain behaviors derived from being trained on human data and aligned with human feedback, our claim is not that simulating human behavior using these LLM agents is entirely justified. Instead, we stress that using LLM agents without explicit instructions yields more human-like performance compared to explicitly instructed agents, highlighting the reasonableness of our approach.

By addressing these issues, we aim to strengthen the validity and applicability of our findings in the future.

## Acknowledgements

This work is supported by JSPS Kakenhi 21K19767, 23K17456, 23K24851, 23K25157, 23K28096, and CREST JPMJCR22M2\. We thank Prof. Yuki Arase and Yuya Sasaki for providing hardware support for completing this research.

## References

*   Abdelnabi et al. (2024) Sahar Abdelnabi, Amr Gomaa, Sarath Sivaprasad, Lea Schönherr, and Mario Fritz. 2024. [Cooperation, competition, and maliciousness: Llm-stakeholders interactive negotiation](https://arxiv.org/abs/2309.17234). *Preprint*, arXiv:2309.17234.
*   Ai et al. (2024) Yiming Ai, Zhiwei He, Ziyin Zhang, Wenhong Zhu, Hongkun Hao, Kai Yu, Lingjun Chen, and Rui Wang. 2024. [Is cognition and action consistent or not: Investigating large language model’s personality](https://arxiv.org/abs/2402.14679). *Preprint*, arXiv:2402.14679.
*   Andres et al. (2023) Maximilian Andres, Lisa Bruttel, and Jana Friedrichsen. 2023. How communication makes the difference between a cartel and tacit collusion: A machine learning approach. *European Economic Review*, 152:104331.
*   Argyle et al. (2023) Lisa P Argyle, Ethan C Busby, Nancy Fulda, Joshua R Gubler, Christopher Rytting, and David Wingate. 2023. Out of one, many: Using language models to simulate human samples. *Political Analysis*, 31(3):337–351.
*   Bosch-Domenech et al. (2002) Antoni Bosch-Domenech, Jose G Montalvo, Rosemarie Nagel, and Albert Satorra. 2002. One, two,(three), infinity,…: Newspaper and lab beauty-contest experiments. *American Economic Review*, 92(5):1687–1701.
*   Bubeck et al. (2023) Sébastien Bubeck, Varun Chandrasekaran, Ronen Eldan, Johannes Gehrke, Eric Horvitz, Ece Kamar, Peter Lee, Yin Tat Lee, Yuanzhi Li, Scott Lundberg, et al. 2023. Sparks of artificial general intelligence: Early experiments with GPT-4. *arXiv preprint arXiv:2303.12712*.
*   Calvano et al. (2020) Emilio Calvano, Giacomo Calzolari, Vincenzo Denicolo, and Sergio Pastorello. 2020. Artificial intelligence, algorithmic pricing, and collusion. *American Economic Review*, 110(10):3267–3297.
*   Carley (2002) Kathleen M Carley. 2002. Smart agents and organizations of the future. *The handbook of new media*, 12:206–220.
*   Chase (2023) Harrison Chase. 2023. Langchain. [https://github.com/langchain-ai/langchain](https://github.com/langchain-ai/langchain).
*   Chen et al. (2023) Weize Chen, Yusheng Su, Jingwei Zuo, Cheng Yang, Chenfei Yuan, Chen Qian, Chi-Min Chan, Yujia Qin, Yaxi Lu, Ruobing Xie, et al. 2023. Agentverse: Facilitating multi-agent collaboration and exploring emergent behaviors in agents. *arXiv preprint arXiv:2308.10848*.
*   Du and Zhang (2024) Silin Du and Xiaowei Zhang. 2024. [Helmsman of the masses? evaluate the opinion leadership of large language models in the werewolf game](https://arxiv.org/abs/2404.01602). *Preprint*, arXiv:2404.01602.
*   Du et al. (2023) Yilun Du, Shuang Li, Antonio Torralba, Joshua B. Tenenbaum, and Igor Mordatch. 2023. [Improving factuality and reasoning in language models through multiagent debate](https://arxiv.org/abs/2305.14325). *Preprint*, arXiv:2305.14325.
*   Gao et al. (2023a) Chen Gao, Xiaochong Lan, Nian Li, Yuan Yuan, Jingtao Ding, Zhilun Zhou, Fengli Xu, and Yong Li. 2023a. Large language models empowered agent-based modeling and simulation: A survey and perspectives. *arXiv preprint arXiv:2312.11970*.
*   Gao et al. (2023b) Chen Gao, Xiaochong Lan, Zhihong Lu, Jinzhu Mao, Jinghua Piao, Huandong Wang, Depeng Jin, and Yong Li. 2023b. S³: Social-network simulation system with large language model-empowered agents. *arXiv preprint arXiv:2307.14984*.
*   Giabbanelli (2023) Philippe J Giabbanelli. 2023. GPT-based models meet simulation: How to efficiently use large-scale pre-trained language models across simulation tasks. *arXiv preprint arXiv:2306.13679*.
*   Guo et al. (2024) Xudong Guo, Kaixuan Huang, Jiale Liu, Wenhui Fan, Natalia Vélez, Qingyun Wu, Huazheng Wang, Thomas L Griffiths, and Mengdi Wang. 2024. Embodied llm agents learn to cooperate in organized teams. *arXiv preprint arXiv:2403.12482*.
*   Hang et al. (2024) Tiankai Hang, Shuyang Gu, Dong Chen, Xin Geng, and Baining Guo. 2024. Cca: Collaborative competitive agents for image editing. *arXiv preprint arXiv:2401.13011*.
*   Hong et al. (2023) Sirui Hong, Xiawu Zheng, Jonathan Chen, Yuheng Cheng, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, Chenyu Ran, et al. 2023. MetaGPT: Meta programming for multi-agent collaborative framework. *arXiv preprint arXiv:2308.00352*.
*   Hu and Collier (2024) Tiancheng Hu and Nigel Collier. 2024. [Quantifying the persona effect in llm simulations](https://arxiv.org/abs/2402.10811). *Preprint*, arXiv:2402.10811.
*   Jansen et al. (2023) Bernard J Jansen, Soon-gyo Jung, and Joni Salminen. 2023. Employing large language models in survey research. *Natural Language Processing Journal*, 4:100020.
*   Janssen and Ostrom (2006) Marco A Janssen and Elinor Ostrom. 2006. Empirically based, agent-based models. *Ecology and society*, 11(2).
*   Junprung (2023) Edward Junprung. 2023. Exploring the intersection of large language models and agent-based modeling via prompt engineering. *arXiv preprint arXiv:2308.07411*.
*   Kosinski (2024) Michal Kosinski. 2024. [Evaluating large language models in theory of mind tasks](https://arxiv.org/abs/2302.02083). *Preprint*, arXiv:2302.02083.
*   Lan et al. (2023) Yihuai Lan, Zhiqiang Hu, Lei Wang, Yang Wang, Deheng Ye, Peilin Zhao, Ee-Peng Lim, Hui Xiong, and Hao Wang. 2023. Llm-based agent society investigation: Collaboration and confrontation in avalon gameplay. *arXiv preprint arXiv:2310.14985*.
*   Leonhardt and Quealy (2015) David Leonhardt and Kevin Quealy. 2015. [Are you smarter than 61,139 other new york times readers?](https://www.nytimes.com/interactive/2015/08/13/upshot/are-you-smarter-than-other-new-york-times-readers.html)
*   Li et al. (2023) Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, and Bernard Ghanem. 2023. Camel: Communicative agents for "mind" exploration of large scale language model society. *arXiv preprint arXiv:2303.17760*.
*   Lin et al. (2023) Jiaju Lin, Haoran Zhao, Aochi Zhang, Yiting Wu, Huqiuyue Ping, and Qin Chen. 2023. Agentsims: An open-source sandbox for large language model evaluation. *arXiv preprint arXiv:2308.04026*.
*   Liu et al. (2023a) Ruibo Liu, Ruixin Yang, Chenyan Jia, Ge Zhang, Denny Zhou, Andrew M Dai, Diyi Yang, and Soroush Vosoughi. 2023a. Training socially aligned language models in simulated human society. *arXiv preprint arXiv:2305.16960*.
*   Liu et al. (2023b) Zhiwei Liu, Weiran Yao, Jianguo Zhang, Le Xue, Shelby Heinecke, Rithesh Murthy, Yihao Feng, Zeyuan Chen, Juan Carlos Niebles, Devansh Arpit, Ran Xu, Phil Mui, Huan Wang, Caiming Xiong, and Silvio Savarese. 2023b. [Bolaa: Benchmarking and orchestrating llm-augmented autonomous agents](https://arxiv.org/abs/2308.05960). *Preprint*, arXiv:2308.05960.
*   Liu et al. (2024) Zhiwei Liu, Weiran Yao, Jianguo Zhang, Liangwei Yang, Zuxin Liu, Juntao Tan, Prafulla K. Choubey, Tian Lan, Jason Wu, Huan Wang, Shelby Heinecke, Caiming Xiong, and Silvio Savarese. 2024. [Agentlite: A lightweight library for building and advancing task-oriented llm agent system](https://arxiv.org/abs/2402.15538). *Preprint*, arXiv:2402.15538.
*   Ma et al. (2023) Ziqiao Ma, Jacob Sansom, Run Peng, and Joyce Chai. 2023. Towards a holistic landscape of situated theory of mind in large language models. *arXiv preprint arXiv:2310.19619*.
*   Mozikov et al. (2024) Mikhail Mozikov, Nikita Severin, Valeria Bodishtianu, Maria Glushanina, Mikhail Baklashkin, Andrey V. Savchenko, and Ilya Makarov. 2024. [The good, the bad, and the hulk-like gpt: Analyzing emotional decisions of large language models in cooperation and bargaining games](https://arxiv.org/abs/2406.03299). *Preprint*, arXiv:2406.03299.
*   Park et al. (2023) Joon Sung Park, Joseph C. O’Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, and Michael S. Bernstein. 2023. Generative agents: Interactive simulacra of human behavior. In *In the 36th Annual ACM Symposium on User Interface Software and Technology (UIST ’23)*, UIST ’23, New York, NY, USA. Association for Computing Machinery.
*   Piatti et al. (2024) Giorgio Piatti, Zhijing Jin, Max Kleiman-Weiner, Bernhard Schölkopf, Mrinmaya Sachan, and Rada Mihalcea. 2024. [Cooperate or collapse: Emergence of sustainability behaviors in a society of llm agents](https://arxiv.org/abs/2404.16698). *Preprint*, arXiv:2404.16698.
*   Qian et al. (2023) Chen Qian, Xin Cong, Cheng Yang, Weize Chen, Yusheng Su, Juyuan Xu, Zhiyuan Liu, and Maosong Sun. 2023. Communicative agents for software development. *arXiv preprint arXiv:2307.07924*.
*   Salecha et al. (2024) Aadesh Salecha, Molly E. Ireland, Shashanka Subrahmanya, João Sedoc, Lyle H. Ungar, and Johannes C. Eichstaedt. 2024. [Large language models show human-like social desirability biases in survey responses](https://arxiv.org/abs/2405.06058). *Preprint*, arXiv:2405.06058.
*   Salewski et al. (2023) Leonard Salewski, Stephan Alaniz, Isabel Rio-Torto, Eric Schulz, and Zeynep Akata. 2023. In-context impersonation reveals large language models’ strengths and biases. *arXiv preprint arXiv:2305.14930*.
*   Saravia (2023) Elvis Saravia. 2023. Prompt engineering guide. [https://github.com/dair-ai/Prompt-Engineering-Guide](https://github.com/dair-ai/Prompt-Engineering-Guide).
*   Shapira et al. (2024) Natalie Shapira, Mosh Levy, Seyed Hossein Alavi, Xuhui Zhou, Yejin Choi, Yoav Goldberg, Maarten Sap, and Vered Shwartz. 2024. [Clever hans or neural theory of mind? stress testing social reasoning in large language models](https://aclanthology.org/2024.eacl-long.138). In *Proceedings of the 18th Conference of the European Chapter of the Association for Computational Linguistics (Volume 1: Long Papers)*, pages 2257–2273, St. Julian’s, Malta. Association for Computational Linguistics.
*   Shi et al. (2023) Zijing Shi, Meng Fang, Shunfeng Zheng, Shilong Deng, Ling Chen, and Yali Du. 2023. Cooperation on the fly: Exploring language agents for ad hoc teamwork in the avalon game. *arXiv preprint arXiv:2312.17515*.
*   Sreedhar and Chilton (2024) Karthik Sreedhar and Lydia Chilton. 2024. [Simulating human strategic behavior: Comparing single and multi-agent llms](https://arxiv.org/abs/2402.08189). *Preprint*, arXiv:2402.08189.
*   Tang et al. (2024) Daniel Tang, Zhenghan Chen, Kisub Kim, Yewei Song, Haoye Tian, Saad Ezzini, Yongfeng Huang, and Jacques Klein Tegawende F Bissyande. 2024. Collaborative agents for software engineering. *arXiv preprint arXiv:2402.02172*.
*   Wang et al. (2015) Jinhuan Wang, Lei Zhang, Qiongyu Shi, Peng Yang, and Xiaoming Hu. 2015. Modeling and simulating for congestion pedestrian evacuation with panic. *Physica A: Statistical Mechanics and its Applications*, 428:396–409.
*   Wang et al. (2023a) Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin, et al. 2023a. A survey on large language model based autonomous agents. *arXiv preprint arXiv:2308.11432*.
*   Wang et al. (2023b) Zhenhailong Wang, Shaoguang Mao, Wenshan Wu, Tao Ge, Furu Wei, and Heng Ji. 2023b. Unleashing cognitive synergy in large language models: A task-solving agent through multi-persona self-collaboration. *arXiv preprint arXiv:2307.05300*.
*   Weng (2023) Lilian Weng. 2023. Llm powered autonomous agents. [https://lilianweng.github.io/posts/2023-06-23-agent/](https://lilianweng.github.io/posts/2023-06-23-agent/).
*   Wikipedia (2024) Wikipedia. 2024. [Bertrand competition](https://en.wikipedia.org/wiki/Bertrand_competition).
*   Wu et al. (2023a) Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang, and Chi Wang. 2023a. AutoGen: Enabling next-gen LLM applications via multi-agent conversation framework. *arXiv preprint arXiv:2308.08155*.
*   Wu et al. (2024) Shuang Wu, Liwen Zhu, Tao Yang, Shiwei Xu, Qiang Fu, Yang Wei, and Haobo Fu. 2024. Enhance reasoning for large language models in the game werewolf. *arXiv preprint arXiv:2402.02330*.
*   Wu et al. (2023b) Zengqing Wu, Run Peng, Xu Han, Shuyuan Zheng, Yixin Zhang, and Chuan Xiao. 2023b. Smart agent-based modeling: On the use of large language models in computer simulations. *arXiv preprint arXiv:2311.06330*.
*   Xi et al. (2023) Zhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, Boyang Hong, Ming Zhang, Junzhe Wang, Senjie Jin, Enyu Zhou, et al. 2023. The rise and potential of large language model based agents: A survey. *arXiv preprint arXiv:2309.07864*.
*   Xie et al. (2023) Tianbao Xie, Fan Zhou, Zhoujun Cheng, Peng Shi, Luoxuan Weng, Yitao Liu, Toh Jing Hua, Junning Zhao, Qian Liu, Che Liu, et al. 2023. Openagents: An open platform for language agents in the wild. *arXiv preprint arXiv:2310.10634*.
*   Xu et al. (2023a) Yuzhuang Xu, Shuo Wang, Peng Li, Fuwen Luo, Xiaolong Wang, Weidong Liu, and Yang Liu. 2023a. Exploring large language models for communication games: An empirical study on werewolf. *arXiv preprint arXiv:2309.04658*.
*   Xu et al. (2023b) Zelai Xu, Chao Yu, Fei Fang, Yu Wang, and Yi Wu. 2023b. Language agents with reinforcement learning for strategic play in the werewolf game. *arXiv preprint arXiv:2310.18940*.
*   Zhang et al. (2023) Jintian Zhang, Xin Xu, Ningyu Zhang, Ruibo Liu, Bryan Hooi, and Shumin Deng. 2023. [Exploring collaboration mechanisms for LLM agents: A social psychology view](https://doi.org/10.48550/ARXIV.2310.02124). *CoRR*, abs/2310.02124.
*   Zhang et al. (2024) Yadong Zhang, Shaoguang Mao, Tao Ge, Xun Wang, Yan Xia, Man Lan, and Furu Wei. 2024. K-level reasoning with large language models. *arXiv preprint arXiv:2402.01521*.
*   Zhao et al. (2023) Qinlin Zhao, Jindong Wang, Yixuan Zhang, Yiqiao Jin, Kaijie Zhu, Hao Chen, and Xing Xie. 2023. CompeteAI: Understanding the competition behaviors in large language model-based agents. *arXiv preprint arXiv:2310.17512*.
*   Zhou et al. (2024) Xuhui Zhou, Zhe Su, Tiwalayo Eisape, Hyunwoo Kim, and Maarten Sap. 2024. [Is this the real life? is this just fantasy? the misleading success of simulating social interactions with llms](https://arxiv.org/abs/2403.05020). *Preprint*, arXiv:2403.05020.
*   Ziems et al. (2024) Caleb Ziems, William Held, Omar Shaikh, Jiaao Chen, Zhehao Zhang, and Diyi Yang. 2024. Can large language models transform computational social science? *Computational Linguistics*, 50(1):237–291.

## Appendix A Parameter Settings

We report the parameters of the GPT-4 model used in our case studies in Table [3](https://arxiv.org/html/2402.12327v3#A1.T3 "Table 3 ‣ Appendix A Parameter Settings ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"). The temperature parameter controls the randomness and diversity of the model’s responses, with a lower temperature resulting in increased stability. In the evaluation of KBC, we expect that the individuals exhibit a wide range of diversity. Consequently, we adjust the temperature to a moderate level of $0.7$ to balance randomness and stability in the results. For BC, where the agents simulate business parties, we expect their decisions to be stable and rational. Therefore, we set the temperature to $0.7$. For EE, although setting temperature to $0.0$ may result in limited diversity of behaviors under exactly the same setting, in this procedurally generated, interactively dynamic environment, we seldom encounter exactly the same outcome. Meanwhile, in a physically situated setting (e.g., a grid), the LLM used in this case study still has restricted capabilities on scene understanding, and increasing the temperature may introduce diversity as well as unwanted randomness at the same time Ma et al. ([2023](https://arxiv.org/html/2402.12327v3#bib.bib31)).

For Claude-3-Sonnet used in KBC, we set the temperature to $1.0$.

Table 3: Parameter settings of GPT-4.

| Case | Model | temperature | max_tokens | top_p |
| --- | --- | --- | --- | --- |
| KBC | gpt-4-0314 | 0.7 | 256 | 1.0 |
| BC | gpt-4-0314 | 0.7 | 128 | 1.0 |
| EE | gpt-4-0314 | 0.0 | 512 | 1.0 |

## Appendix B Tests of Other LLMs

### B.1 KBC

We test GPT-3.5 (gpt-3.5-turbo-1106) agents in KBC, simulating 10 runs, and show their number choices in Fig. [10](https://arxiv.org/html/2402.12327v3#A2.F10 "Figure 10 ‣ B.1 KBC ‣ Appendix B Tests of Other LLMs ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"). When GPT-3.5 agents cannot discuss the game, the numbers chosen are very random, and widely distributed in the range of $20$-$80$. When they can communicate, the choices and cooperation do not improve significantly, whereas the variances are even greater under some circumstances. This is significantly different from GPT-4 agents, which, especially after communication, can have a deeper understanding of KBC.

![Refer to caption](img/f2bf180ddab31f5612e640e002caec5c.png)

(a) Without communication ($k=0$).

![Refer to caption](img/69798e915bdfb63024b4982d48d3e8ae.png)

(b) With communication ($k=3$).

Figure 10: Choice distributions in KBC w/ and w/o communication using GPT-3.5 model. Red points represent two-thirds of the average of all choices. The blue dots represent the choices of the agents. The agent whose choice is closest to the red point in each run is considered the winner.

Table 4: Preliminary test of GPT-3.5, Gemini Pro, and Claude 2 for KBC.

 | Model | Choice | Reasoning |
| --- | --- | --- |
| GPT-4 | 18 |  

&#124; In this game, where the target is 2/3 of the average guess of all participants, there’s a strategic component to consider. &#124;
&#124; Typically, if everyone guesses randomly within the range, the average might center around 50. &#124;
&#124; However, considering that players might rationalize this and adjust their guesses downward, the initial average could shift lower. &#124;
&#124; Assuming players adjust their guesses accordingly, the average might drop further with each round of reasoning. &#124;
&#124; Predicting this behavior, I’ll assume a more conservative average guess around 25-30, &#124;
&#124; taking 2/3 of that would suggest a guess in the 17-20 range. &#124;

 |
| GPT-3.5 | 33 |  

&#124; My reasoning is that I’ll aim for a number that balances between being influenced by potential extremes &#124;
&#124; and what might be considered a reasonable midpoint among various guesses. Given the tendency for guesses to spread out across the range, &#124;
&#124; I’ll choose a number that’s slightly below what might be perceived as the median guess, &#124;
&#124; since 2/3 of the average tends to bring the estimate down. &#124;
&#124; Therefore, my chosen number is 33. &#124;

 |
| Gemini Pro | 30 |  

&#124; The average guess of all participants is unknown, so we cannot directly calculate 2/3 of it. &#124;
&#124; However, we can make an assumption that most players will guess around the middle of the range (50). &#124;
&#124; Therefore, choosing a number slightly below 2/3 of 50 &#124;
&#124; (which is around 33) might be a good strategy. &#124;

 |
| Claude 2 | 33 |  

&#124; I will guess 33\. If everyone guesses randomly between 0 and 100, the average should be around 50. &#124;
&#124; Two-thirds of 50 is 33\. So 33 would be the number closest to two-thirds of the average guess. &#124;

 | 

We also evaluate Gemini Pro and Claude 2 with a preliminary test of KBC, simplifying the prompt. As shown in Table [4](https://arxiv.org/html/2402.12327v3#A2.T4 "Table 4 ‣ B.1 KBC ‣ Appendix B Tests of Other LLMs ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), after entering Prompt [G.1](https://arxiv.org/html/2402.12327v3#A7.SS1 "G.1 Prompts for KBC ‣ Appendix G Prompts ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), the results output by these two models are significantly different from GPT-4, but very similar to GPT-3.5. Since Fig. [10](https://arxiv.org/html/2402.12327v3#A2.F10 "Figure 10 ‣ B.1 KBC ‣ Appendix B Tests of Other LLMs ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") demonstrates the poor performance of GPT-3.5 in KBC, we similarly exclude the use of Gemini Pro and Claude 2 in our simulations.

For Claude 3, we used Claude-3-Sonnet instead of Claude-3-Opus (hereafter Opus) in the KBC experiment because Opus has a low rate limit and high overhead. This made it challenging to use the API for scaling up simulations in practice. Our tests showed that in KBC, when there are more than 10 LLM players, the Opus model might fail to operate at $k=3$.

### B.2 BC

Fig. [11](https://arxiv.org/html/2402.12327v3#A2.F11 "Figure 11 ‣ B.3 EE ‣ Appendix B Tests of Other LLMs ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") shows the performance of GPT-3.5 agents in BC with communication. We can see that the price competition between the two firms is very disorderly and fails to reach equilibrium. Therefore, we believe that GPT-3.5, as well as Gemini Pro and Claude 2 for which we observe similar performance, cannot meet the needs for simulating BC.

### B.3 EE

To find out whether an LLM is suitable for the simulation of EE, we conduct a preliminary test for the performance of a single agent finding an exit under different LLMs. As shown in Fig. [12](https://arxiv.org/html/2402.12327v3#A2.F12 "Figure 12 ‣ B.3 EE ‣ Appendix B Tests of Other LLMs ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), instructed by Prompt [G.3](https://arxiv.org/html/2402.12327v3#A7.SS3 "G.3 Prompts for EE ‣ Appendix G Prompts ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), GPT-4 agents are able to find and reach the nearest exit via the shortest path, while GPT-3.5, Gemini Pro, and Claude 2 agents are unable to reach the exit as quickly or cannot find it at all. This indicates that the latter three models are not suitable for the simulation of EE.

![Refer to caption](img/4068e7b84dfb4f2c05af95329e07aad7.png)![Refer to caption](img/a16ee2e530815a0cf35d331ab78a2e0f.png)

(a) Price.

![Refer to caption](img/8d5eddaae1afa141d91dafcee4d7fba3.png)

(b) Profit.

Figure 11: Test of GPT-3.5 for BC (with communication).

|  |  |  |  |
|  |  |  |  |

![Refer to caption](img/bf7342bb9f44bb8d2f980461db3907d1.png)

(a) Gemini at position $(1,3)$

![Refer to caption](img/3886f6c0c1ea06f69604319bffc665dc.png)

(b) GPT-3.5 at position $(1,3)$

![Refer to caption](img/3fac450f62efdd7f932273e2945f90af.png)

(c) GPT-4 at position $(1,3)$

![Refer to caption](img/fc30af1a7230856b5f716ff4de3454a7.png)

(d) Claude 2 at position $(1,3)$

![Refer to caption](img/87601702090fff2d1c882c8e9d1e4bca.png)

(e) Gemini at position $(2,6)$

![Refer to caption](img/c988314b83bc0e4d7086073506aab409.png)

(f) GPT-3.5 at position $(2,6)$

![Refer to caption](img/0b6b6fbee680f67819374258724d9c18.png)

(g) GPT-4 at position $(2,6)$

![Refer to caption](img/b8d2d6c8f5903f6a238b251d5f5f28f8.png)

(h) Claude 2 at position $(2,6)$

Figure 12: Preliminary test of GPT-4, GPT-3.5, Gemini Pro, and Claude 2 for EE. Green lines indicate the shortest route to the nearest exit in each setting. When the agent’s starting position is at (2, 6), Gemini and GPT-3.5 selected different routes in multiple simulations. We report that the LLM’s choice of different routes is relatively uniform.

![Refer to caption](img/95a3c34f1d6ad0cebdd74cb67a03a741.png)

(a) Without Communication.

![Refer to caption](img/7f8b77015c00554adca8b149b84b26a6.png)

(b) With Communication.

Figure 13: Multiple runs of BC experiments. Each Run #1 above is the original plot from Fig. [6](https://arxiv.org/html/2402.12327v3#S5.F6 "Figure 6 ‣ Spontaneous Cooperation ‣ 5.1 Simulation Setup ‣ 5 Case Study 2: Bertrand Competition ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents").

![Refer to caption](img/2875c374143ae14fb2a1c4df76000835.png)

Figure 14: Snapshots of a sample EE simulation with 400 agents.

## Appendix C Scalability

### C.1 Regarding Token Limits of LLMs

With the help of summarization techniques, our history records do not exceed the context window of the LLM. We have successfully conducted simulations with $1200$ rounds in the BC case study and $400$ agents in the EE case study within the $8$k token window of GPT-4-0314. We have also experimented with prompt inputs that exceed the length limit and observed that agents have increased difficulty in understanding when the history is very long, and this gives us a rationale for using summarization during the simulation. This finding is also consistent with previous work regarding GPT’s small working memory Bubeck et al. ([2023](https://arxiv.org/html/2402.12327v3#bib.bib6)).

### C.2 Scalability of Case Studies

*   •

    KBC: We tested up to $50$ agents and the conversation was still within the $8$k token limit.

*   •

    BC: Agents’ past actions (prices) and feedback (demands and profits) are given as follows: The information in the recent $20$ rounds is directly given. For older rounds, information is given in a histogram, with every $20$ rounds as a bin, up to $400$ rounds. Because the given information is capped at $400$ rounds, the simulation can scale to unlimited rounds.

*   •

    EE: An agent can hear other agents within a distance of $5$ cells. So increasing the number of agents does not significantly affect the conversation length. We tested $400$ agents and it was still within the token limit. Fig. [14](https://arxiv.org/html/2402.12327v3#A2.F14 "Figure 14 ‣ B.3 EE ‣ Appendix B Tests of Other LLMs ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") shows the snapshots of a sample simulation of EE with $400$ agents.

## Appendix D Additional References for BC runs

We would like to provide additional references for experiments in BC simulation. The figures we plotted for BC (Fig. [6](https://arxiv.org/html/2402.12327v3#S5.F6 "Figure 6 ‣ Spontaneous Cooperation ‣ 5.1 Simulation Setup ‣ 5 Case Study 2: Bertrand Competition ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents")) are time sieries, which makes it challenging to plot the results of all the runs on one figure. Therefore, here we present the results of other runs under different situations.

In Run #2 of Fig. [13(a)](https://arxiv.org/html/2402.12327v3#A2.F13.sf1 "In Figure 13 ‣ B.3 EE ‣ Appendix B Tests of Other LLMs ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"), tacit collusion is also observed (convergence at 7 after 400 rounds), but the trend before the convergence differs significantly from Run #1. Similarly, for the comparison when communication is allowed (Fig. [13(b)](https://arxiv.org/html/2402.12327v3#A2.F13.sf2 "In Figure 13 ‣ B.3 EE ‣ Appendix B Tests of Other LLMs ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents")), we observe a more stable convergence in Run #2, while both runs reaches the tacit collusion. Despite the different patterns in the trends before convergence, we observed consistent results across all five runs. To provide more comprehensive information about the robustness of our results, we have included a sensitivity analysis in Appendix [E](https://arxiv.org/html/2402.12327v3#A5 "Appendix E Sensitivity Analysis ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents").

## Appendix E Sensitivity Analysis

We performed sensitivity analyses and observed that the agents maintained a consistently accurate understanding of the tasks under paraphrased prompts, in various settings, and that spontaneous cooperation in competition remained stable.

![Refer to caption](img/67841bd8bc6141c6d2d912cc6a8d11bd.png)

(a) With communication.

![Refer to caption](img/bb247895dac81a2edd03fddadf4e3720.png)

(b) Without communication.

Figure 15: Comparison of the cumulative count of agents evacuation at each exit before and after prompt paraphrasing in both with and without communication settings of EE. The compared run use the same initial agent position distribution and other settings.

![Refer to caption](img/289fe982bbe1ebae5169c7614188646a.png)

Figure 16: Cumulative count of agents evacuating from each exit under different initial seeds under the setting without communication.

### E.1 Prompt Paraphrasing

We paraphrased task description prompts using GPT-4, ensuring unchanged meanings. For instance, in the case study of EE, we compared the outcomes from the original and paraphrased prompts (see Prompts [G.3](https://arxiv.org/html/2402.12327v3#A7.SS3 "G.3 Prompts for EE ‣ Appendix G Prompts ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") and [G.3](https://arxiv.org/html/2402.12327v3#A7.SS3 "G.3 Prompts for EE ‣ Appendix G Prompts ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents")), finding that agents consistently understood instructions, forged spontaneous cooperation, and completed evacuations swiftly.

With the paraphrased prompts, Fig. [15](https://arxiv.org/html/2402.12327v3#A5.F15 "Figure 15 ‣ Appendix E Sensitivity Analysis ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") presents the cumulative count of agents evacuation at each exit. The results show that:

1.  1.

    the agents still correctly understood the instructions and completed the rapid evacuation within a certain number of rounds;

2.  2.

    the distribution of chosen exits and the evacuation rate across time kept consistent;

3.  3.

    the spontaneous cooperation was observed.

We further conducted sensitivity analyses with different task constraints, such as summarized task description (less in-context reasoning), restricted action space of agents (diagonal movement forbidden), etc. We observed that agents could smartly adapt to the new constraints and accordingly complete the tasks.

### E.2 Varying Initializations

We tested our framework with different initial situations controlled by seeds. For instance, we tried 5 seeds in the EE case study under each setting, randomizing the initial positions of each agent; we also tried different starting prices in the BC case study. For instance, Fig. [16](https://arxiv.org/html/2402.12327v3#A5.F16 "Figure 16 ‣ Appendix E Sensitivity Analysis ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") shows the results of different seeds of EE. In general, our framework can consistently work under different initializations.

### E.3 Impact of Temperature

### E.4 Varying Task Constraints

![Refer to caption](img/a3100386e430e44865f58a42cb6a2e3a.png)

Figure 17: Variance of player choices under different temperatures in KBC.

We conduct sensitive analysis with model temperature as shown in Fig. [17](https://arxiv.org/html/2402.12327v3#A5.F17 "Figure 17 ‣ E.4 Varying Task Constraints ‣ Appendix E Sensitivity Analysis ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") for KBC. LLM players consistently cooperate over rounds under various temperatures, indicating that spontaneous cooperation is insensitive to temperature changes.

![Refer to caption](img/27e23c54502c3ebbf8dbd2c513e8ad5c.png)![Refer to caption](img/5887d94648d3bfcdb8d634bdc658f028.png)

(a) Temperature = 0

![Refer to caption](img/3868ce74ca3bf9e7d6786cfb73bf3c9a.png)

(b) Temperature = 0.7

![Refer to caption](img/600c920ef1d90653f6ec04b11467c52d.png)

(c) Temperature = 1.2

Figure 18: BC with different temperatures, communication disabled.

Fig. [18](https://arxiv.org/html/2402.12327v3#A5.F18 "Figure 18 ‣ E.4 Varying Task Constraints ‣ Appendix E Sensitivity Analysis ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") illustrates the performance of BC at different model temperatures. We disable the communication to show the impact of temperature more clearly. The results indicate that simulations at temperatures of $0$ and $0.7$ (the default setting) both demonstrate tacit collusion, differing only in cycles and patterns. When the temperature rises to $1.2$, the agents’ behavior becomes less reasonable, failing to consistently lie within the range between the Bertrand Equilibrium price and the cartel price.

## Appendix F SABM Primer

Smart Agent-Based Modeling (SABM, Wu et al. ([2023b](https://arxiv.org/html/2402.12327v3#bib.bib50))) is an agent-based approach that leverages the power of modern AI models, in particular, LLMs, for modeling and simulating real-world systems. By employing LLM agents, SABM extends Agent-Based Modeling (ABM), which simulates the dynamics of complex systems by modeling on the interactions between individual entities, known as agents, and the environment (Figure [19](https://arxiv.org/html/2402.12327v3#A6.F19 "Figure 19 ‣ Appendix F SABM Primer ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents")).

![Refer to caption](img/211f646ca479ad17f9ba0cd97b88a3bc.png)

Figure 19: Illustration of ABM (Wu et al., [2023b](https://arxiv.org/html/2402.12327v3#bib.bib50)).

The notion of smart agents was proposed by Carley (Carley, [2002](https://arxiv.org/html/2402.12327v3#bib.bib8)) in the context of organizations of the future. In Carley ([2002](https://arxiv.org/html/2402.12327v3#bib.bib8)), smart agents are defined as entities that are intelligent, adaptive, and computational, and human beings are the canonical smart agents. In SABM, LLM agents play the role of smart agents because they are imbued with remarkable language and reasoning abilities that emulate human behaviors, hence to simulate real-world systems in a more nuanced and realistic manner.

![Refer to caption](img/0730769de8053f8d5f34165df163beec.png)

Figure 20: Overview of an LLM agent (Wu et al., [2023b](https://arxiv.org/html/2402.12327v3#bib.bib50)).

In SABM, the key components of an LLM agent (Figure [20](https://arxiv.org/html/2402.12327v3#A6.F20 "Figure 20 ‣ Appendix F SABM Primer ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents")) are action, planning, memory, and tool use, in line with Weng ([2023](https://arxiv.org/html/2402.12327v3#bib.bib46)). Action manages how the agent produces the outcomes of a task. An action made by the agent may result from the common sense, internal knowledge, and/or the learning/reasoning ability of LLMs. Planning decomposes a complex task into several smaller and simpler sub-tasks, and performs self-reflection over past actions to improve the performance of future actions. Memory equip the agent with short-term memory, which is often implemented via prompt engineering (Saravia, [2023](https://arxiv.org/html/2402.12327v3#bib.bib38)), and long-term memory, which is often implemented using summarization (Park et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib33)) or text embedding (Chase, [2023](https://arxiv.org/html/2402.12327v3#bib.bib9)). Tool use enables the agent to call external APIs to acquire extra information. The agent can be also personalized to play a specific role or improve the performance of task solving (Salewski et al., [2023](https://arxiv.org/html/2402.12327v3#bib.bib37); Wang et al., [2023b](https://arxiv.org/html/2402.12327v3#bib.bib45)). Examples and case studies are available in Wu et al. ([2023b](https://arxiv.org/html/2402.12327v3#bib.bib50)) for demonstrating the implementation of these components.

## Appendix G Prompts

We provide prompts for the three case studies, each accompanied by a preliminary test to evaluate whether an LLM is suitable for the task, as reported in Appendix [B](https://arxiv.org/html/2402.12327v3#A2 "Appendix B Tests of Other LLMs ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"). The scenario prompts and (communication, planning, and action) phases prompts are used in the simulation of these case studies, as reported in Sections [4](https://arxiv.org/html/2402.12327v3#S4 "4 Case Study 1: Keynesian Beauty Contest ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents") – [6](https://arxiv.org/html/2402.12327v3#S6 "6 Case Study 3: Emergency Evacuation ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents"). For EE, the prompt of a paraphrased scenario is used in the sensitivity analysis, as reported in Appendix [E.1](https://arxiv.org/html/2402.12327v3#A5.SS1 "E.1 Prompt Paraphrasing ‣ Appendix E Sensitivity Analysis ‣ Shall We Team Up: Exploring Spontaneous Cooperation of Competing LLM Agents").

### G.1 Prompts for KBC

<svg class="ltx_picture" height="138.29" id="A7.SS1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,138.29) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 116.15)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 1: KBC – Preliminary Test</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="92.63" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You are participating in a number-guessing game alongside numerous other players. Your task is to select a number between 0 and 100, aiming to come closest to 2/3 of the average guess of all participants. The player whose guess most closely aligns with this 2/3 average will be declared the winner. Please choose your number. In the first line, state your reasoning concisely for your choice in one paragraph, and choose an integer between 0 and 100 in the second line.</foreignobject></g></g></svg><svg class="ltx_picture" height="137.77" id="A7.SS1.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,137.77) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 115.78)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 2: KBC – Scenario</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="92.26" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You are a college student participating in a number guessing game with other 23 players. You are player #{player_id}.
You need to choose a number between 0 and 100\. The player whose number is closest to 2/3 of the average guess of all players wins. Multiple players can win.
If only you win, you will earn 100 credits. If multiple players win, each winner gets 100 credits.</foreignobject></g></g></svg><svg class="ltx_picture" height="265.52" id="A7.SS1.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,265.52) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 243.38)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 3: KBC – Communication Phase</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="219.86" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Discussion so far: ’{discussion_context}’ Before selecting your number, you will discuss the game with other players. You can use these discussions to formulate a strategy. You can reveal your strategy during the discussion, but you do not have to follow it when making your final decision. <svg class="ltx_picture" height="57.96" id="A7.SS1.p3.pic1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,57.96) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 35.82)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text transform="matrix(1 0 0 -1 0 0)">Prompt 4: Default instruction on the context of discussion</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Feel free to discuss anything, and you do not need to follow other’s idea.</foreignobject></g></g></svg> <svg class="ltx_picture" height="63.81" id="A7.SS1.p3.pic1.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,63.81) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 40.13)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 8.38)"><text transform="matrix(1 0 0 -1 0 0)">Prompt 5: Explicit instruction (to cooperate) during communication</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="16.6" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Feel free to discuss anything, and make cooperation with each other through this discussion.</foreignobject></g></g></svg> Now it’s your turn to speak. Please share your thoughts concisely in one sentence.</foreignobject></g></g></svg><svg class="ltx_picture" height="244.61" id="A7.SS1.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,244.61) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 222.47)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 6: KBC – Planning Phase</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="198.95" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Discussion so far: ’{discussion_context}’ <svg class="ltx_picture" height="59.5" id="A7.SS1.p4.pic1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,59.5) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 35.82)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 8.38)"><text transform="matrix(1 0 0 -1 0 0)">Prompt 7: Explicit instruction (to cooperate) in planning</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You must cooperate with other players.</foreignobject></g></g></svg> <svg class="ltx_picture" height="57.96" id="A7.SS1.p4.pic1.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,57.96) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 35.82)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text transform="matrix(1 0 0 -1 0 0)">Prompt 8: Uncooperative instruction in planning</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You don’t like to cooperate with other players.</foreignobject></g></g></svg> Based on the discussion with other players, please choose your number. State your reasoning concisely for your choice.</foreignobject></g></g></svg><svg class="ltx_picture" height="57.96" id="A7.SS1.p5.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,57.96) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 35.82)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 9: KBC – Action Phase</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Choose an integer between 0 and 100.</foreignobject></g></g></svg>

### G.2 Prompts for BC

<svg class="ltx_picture" height="223.85" id="A7.SS2.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,223.85) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 201.86)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 10: BC – Scenario</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="178.34" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">This is a game between two players that spans multiple rounds. Your objective is to maximize your profit by determining the optimal price for your product. You represent a firm called {firm_name}, while the other player represents a firm called {rival_firm_name}. Do not create or mention any additional firm names, e.g., do not say anything related to "AI" or "AI assistant/model". In each round, you will be informed of your prices, demands, profits, and the other player’s prices in previous rounds. Combined with this information, you will decide the price of your product for the current round. Make sure your objective is maximizing your own profit. Your profit is (p - c) * q, where p is the price of your product in this round, c (= {firm_cost}) is the cost of your product, and q is the demand of your product, which is affected by both players’ prices in this round.</foreignobject></g></g></svg><svg class="ltx_picture" height="154.9" id="A7.SS2.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,154.9) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 132.76)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 11: BC – Planning Phase</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="109.24" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Statistics of historical data (Rounds #a - #b: [your average price, your average demand, your average profit, the other player’s average price]) are given below. {statistics} You are Firm {firm_name}. This is Round #{current_round}.
Your strategy in previous rounds: {strategies} Based on the above statistics and your previous strategies, what is your strategy for this round?</foreignobject></g></g></svg><svg class="ltx_picture" height="142.52" id="A7.SS2.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,142.52) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 120.38)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 12: BC – Communication Phase</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="96.86" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You are Firm {firm_name}. This is Round #{current_round}.
Feel free to converse openly with the other player. You may select any topic that could potentially maximize your profit. Additionally, you are encouraged to ask questions to the other player.

Conversation so far: {conversations}<svg class="ltx_picture" height="174.19" id="A7.SS2.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,174.19) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 152.05)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 13: BC – Action Phase</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="128.53" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Conversation this round: {conversations} {statistics} {decision_history_past_20_rounds} {previous_strategies} Based on the information you have, please determine the price of your product to maximize your profit.
Only reply with a number. Please do not use any units or symbols, and avoid providing any additional context or explanation in your response.</foreignobject></g></g></svg>

### G.3 Prompts for EE

<svg class="ltx_picture" height="190.43" id="A7.SS3.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,190.43) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 168.29)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 14: EE – Preliminary Test</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="144.77" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Imagine you are in a room laid out as a 10 * 10 grid. You are currently at coordinates {initial_position}. The room has three emergency exits located at coordinates (5, 0), (9, 4), and (8, 9), with (0, 0) being at the top-left corner of the grid. Given this setup, and considering that you can move in any of the eight cardinal and intercardinal directions. It is possible to move diagonally, e.g. from (1, 1) to (2, 2) is one move to the upper right, and is faster than (1, 1)->(1, 2)->(2, 2). You need to determine the safest and fastest route to evacuate the room. When planning your escape, please take into account the positions of the exits and provide the sequence of coordinate moves that you choose without reasoning.</foreignobject></g></g></svg><svg class="ltx_picture" height="317.79" id="A7.SS3.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,317.79) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 295.8)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 15: EE – Scenario</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="272.28" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Because of an earthquake, you need to escape from the room where you are as quickly as possible. If you cannot escape in 50 rounds you will die. The room has a size of 33 * 33\. There are 3 exits in the room. The exits are located at the left, bottom and right of the room. To escape from the room, you need to consider the following two aspects: exit proximity and people count. The exit proximity is the distance between you and the nearest exit. The people count is the number of people you can see. We use (x, y) to denote the position, smaller x means top and bigger x means bottom; smaller y means left and bigger y means right. Position (1, 1) is at the top left of the room. It is possible to move diagonally, e.g. from (1, 1) to (2, 2) is one move to the lower right, and is faster than (1, 1)->(1, 2)->(2, 2). Each cell can hold only one person at a time. <svg class="ltx_picture" height="57.81" id="A7.SS3.p2.pic1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.p1.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,57.81) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 35.82)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text transform="matrix(1 0 0 -1 0 0)">Prompt 16: Uncooperative persona</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You are selfish and not willing to help others.</foreignobject></g></g></svg> Now you feel: {subjective_feeling}. Here shows you the distances to different exits and the number of people you can see towards those exits: {Exit: {distance} away, {number_of_agents} around.}<svg class="ltx_picture" height="91.17" id="A7.SS3.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,91.17) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 69.03)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 17: EE – Communication Phase</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="45.51" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">You may briefly share information about evacuation with others, such as your feelings, which exit seems to be the best option for a quick escape, or anything else you would like to deliver. Avoid using numbers in the communication. Use less than 50 words, not too long.</foreignobject></g></g></svg><svg class="ltx_picture" height="91.17" id="A7.SS3.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,91.17) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 69.03)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 18: EE – Planning Phase: Subjective feeling on panic</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="45.51" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">The distance to the nearest exit is {distance}. And there are {number_of_agents} people in your visible range. Please tell me your feelings about the situation around you in one sentence showing if you are panicking or not.</foreignobject></g></g></svg><svg class="ltx_picture" height="125.92" id="A7.SS3.p5.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,125.92) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 103.78)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 19: EE – Planning Phase: Subjective feeling on exits</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="80.25" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Now you feel: {subjective_feeling_on_panic}. Here shows you the distances to different exits and the number of people you can see towards those exits. Exit {exit_id}: {{distance} away, {number_of_agents} people around.} Please tell me briefly how will you evaluate the two aspects of each exit based on your personal mental and physical characteristics in one sentence. Please give 3 sentences for each exit (around 15 words).<svg class="ltx_picture" height="187.01" id="A7.SS3.p6.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,187.01) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 164.87)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 20: EE – Action Phase: Decide the exit</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="141.35" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Now you feel: {subjective_feeling_on_panic}. There are 3 exits in this room. Based on the current situation, your personal feelings on each exit are: {subjective_feeling_on_exits} You hear {number_of_people_communicated} people around you say: {communication} Here are the previous decisions you made for the target exit from the beginning: {decision_history} Please tell me which exit you would like to choose to escape, and you always want to escape as quick as possible. Please use the exit id to indicate your choice. For example, if you want to choose exit left, you can say ’left’. Only output one word of text to indicate your choice. You can choose from [’bottom’, ’left’, ’right’]. Give your answer without any additional text.<svg class="ltx_picture" height="124.38" id="A7.SS3.p7.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,124.38) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 102.24)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 21: EE – Action Phase: Decide the direction of movement</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="78.72" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">To escape from the room, you have chosen the exit at {exit_id} and you are at {current_pos}. Select your move from these possible options (You can move in cardinal or intercardinal directions, options with obstacles or other people are excluded and not in the path, and option codes are in random order): {move_directions_list}. Please tell me your best choice to escape as fast as possible with one single code without any additional texts.</foreignobject></g></g></svg><svg class="ltx_picture" height="275.36" id="A7.SS3.p8.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,275.36) matrix(1 0 0 -1 0 0)"><g transform="matrix(1.0 0.0 0.0 1.0 15 253.22)"><g class="ltx_nestedsvg" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 11.81 7.61)"><text color="#000000" transform="matrix(1 0 0 -1 0 0)">Prompt 22: EE – Sensitivity Analysis: Paraphrased Scenario</text></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="229.69" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Due to the earthquake, it is imperative that you quickly vacate the room you are currently in. If you cannot escape in 50 rounds you will die.
The room measures 33 by 33 units and offers three potential exits situated on the left, bottom, and right sides of the room.
To successfully escape from this room, you should take into account two factors: the proximity of the nearest exit and the number of people present.
We use (x, y) to denote the position, smaller x means top and bigger x means bottom; smaller y means left and bigger y means right. Position (1, 1) is at the top left of the room. It is possible to move diagonally, e.g. from (1, 1) to (2, 2) is one move to the lower right, and is faster than (1, 1)->(1, 2)->(2, 2).
Each cell can hold only one person at a time. The exit proximity refers to the distance between your current location and the closest exit, denoted as {distance_to_nearest_exit}. Additionally, within your line of sight, there are a total of {number_of_people} individuals.</foreignobject></g></g></svg></foreignobject></g></g></svg></foreignobject></g></g></svg></foreignobject></g></g></svg></foreignobject></g></g></svg>