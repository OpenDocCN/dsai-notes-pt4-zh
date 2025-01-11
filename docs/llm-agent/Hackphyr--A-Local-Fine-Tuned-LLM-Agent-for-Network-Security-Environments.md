<!--yml
category: 未分类
date: 2025-01-11 12:14:51
-->

# Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments

> 来源：[https://arxiv.org/html/2409.11276/](https://arxiv.org/html/2409.11276/)

Maria Rigaki Carlos Catania Sebastian Garcia

###### Abstract

Large Language Models (LLMs) have shown remarkable potential across various domains, including cybersecurity. Using commercial cloud-based LLMs may be undesirable due to privacy concerns, costs, and network connectivity constraints. In this paper, we present Hackphyr, a locally fine-tuned LLM to be used as a red-team agent within network security environments. Our fine-tuned 7 billion parameter model can run on a single GPU card and achieves performance comparable with much larger and more powerful commercial models such as GPT-4\. Hackphyr clearly outperforms other models, including GPT-3.5-turbo, and baselines, such as Q-learning agents in complex, previously unseen scenarios. To achieve this performance, we generated a new task-specific cybersecurity dataset to enhance the base model’s capabilities. Finally, we conducted a comprehensive analysis of the agents’ behaviors that provides insights into the planning abilities and potential shortcomings of such agents, contributing to the broader understanding of LLM-based agents in cybersecurity contexts.

###### keywords:

large language models , agents , reinforcement learning , network security\affiliation

[inst1]organization=Faculty of Electrical Engineering, Czech Technical University in Prague,city=Prague, country=Czech Republic

\affiliation

[inst2]organization=Engineering School, National University of Cuyo,city=Mendoza, country=Argentina

\affiliation

[inst3]organization=National Scientific and Technical Research Council (CONICET),country=Argentina

## 1 Introduction

Autonomous agents are systems that interact with and adapt to their environments to achieve specific goals. Traditionally, human defenders have been the primary responders to security breaches. The potential deployment of Autonomous Intelligent Cyber Agents (AICA) [[1](https://arxiv.org/html/2409.11276v1#bib.bib1)] for defensive operations could transform cyber defenses. At the same time, offensive agents, also known as Red-team Agents, have the potential to test existing defenses and uncover potential problems. These agents could preemptively identify threats making defense a dynamic, continuous process.

In recent years, Reinforcement Learning (RL) techniques have emerged as the predominant method for developing autonomous agents to navigate and adapt to dynamic environments [[2](https://arxiv.org/html/2409.11276v1#bib.bib2), [3](https://arxiv.org/html/2409.11276v1#bib.bib3)]. Reinforcement learning-based agents evolve through direct interactions with their environments, learning to optimize behaviors through a trial-and-error approach supported by reward systems. Trial-and-error comes with certain shortcomings: it requires hundreds of thousands of training episodes to learn a policy, and once trained, the policy is not flexible. An agent trained in one specific environment configuration will not work without retraining in a slightly different scenario.

LLMs have achieved notable successes in recent years, demonstrating significant potential in mimicking human-like behavior [[4](https://arxiv.org/html/2409.11276v1#bib.bib4), [5](https://arxiv.org/html/2409.11276v1#bib.bib5)]. Building upon this capability, a growing research area has employed LLMs as central controllers to construct autonomous agents to obtain human-like decision-making capabilities [[6](https://arxiv.org/html/2409.11276v1#bib.bib6), [7](https://arxiv.org/html/2409.11276v1#bib.bib7)]. Compared to RL approaches, LLM-based agents encapsulate a vast spectrum of human knowledge and a richer understanding of the world that enables them to act without needing additional or specific training [[8](https://arxiv.org/html/2409.11276v1#bib.bib8)]. Secondly, pre-trained language models exhibit remarkable adaptability, as they can be easily adapted to new tasks or domains with relatively minimal additional training, offering a significant advantage over other machine learning models that may require extensive retraining [[9](https://arxiv.org/html/2409.11276v1#bib.bib9)].

Prior work has explored the intersection of LLMs, cybersecurity, and sequential decision-making [[10](https://arxiv.org/html/2409.11276v1#bib.bib10)]. The authors proposed a novel approach that uses pre-trained LLMs as agents within cybersecurity environments. However, the models used were proprietary models running on the cloud. The use of cloud-based language models suffers from several problems. These models are costly to access, limiting accessibility and reproducibility. Furthermore, the models are often unilaterally changed without explanations or updates, raising concerns about their stability and reliability.

The situation can be even more critical in the context of cyber security. Communicating possibly sensitive information about a company network to a cloud-based proprietary model for additional analysis could be a major threat to the security and privacy of the company. In contrast, open-source models are freely available, promoting accessibility, reproducibility, and privacy. Additionally, many open source models come in smaller parameter sizes that allow inference to be performed in lower-cost hardware in the company premises if required. However, smaller model sizes often mean lower performance and the need for further model fine-tuning to a specific task.

In this work, we fine-tuned a 7 billion pre-trained LLM (Zephyr-7b-$\beta$) to be used as a red team agent in the NetSecGame environment [[10](https://arxiv.org/html/2409.11276v1#bib.bib10)]. For the fine-tuning process, we created a new dataset that helped boost the base model’s understanding of the environment and its ability to propose valid and useful actions. Our agent was evaluated in three scenarios with varying levels of complexity and compared to pre-trained commercial LLMs and RL baselines such as Q-learning. Finally, we performed a detailed behavioral analysis of each agent’s actions to evaluate their planning ability and how it matches human-like behavior.

This paper makes several key contributions to the field:

*   1.

    A locally deployed model named Hackphyr, specifically fine-tuned to work in the NetSecGame environment.

*   2.

    A novel dataset, accompanied by an ablation study that analyzes the impact of different dataset components on model performance. This analysis provides valuable insights into the factors that most significantly influence the effectiveness of the fine-tuned model in network security environments.

*   3.

    A detailed behavioral analysis of three LLM-based agents (GPT-4, Zephyr-7b-$\beta$, Hackphyr). This analysis offers a unique perspective on the decision-making processes of each model, providing information on their ability to navigate and solve network security challenges autonomously.

The rest of the paper is organized as follows: Sections [2](https://arxiv.org/html/2409.11276v1#S2 "2 Background ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") and [3](https://arxiv.org/html/2409.11276v1#S3 "3 Related Work ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") present the background knowledge and related work, respectively. The NetSecGame environment is presented in Section [4](https://arxiv.org/html/2409.11276v1#S4 "4 The NetSecGame Environment ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"), and the general agent design is described in Section [5](https://arxiv.org/html/2409.11276v1#S5 "5 LLM Agent Design ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"). Section [6](https://arxiv.org/html/2409.11276v1#S6 "6 Supervised Fine-tuning Methodology ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") details the supervised fine-tuning methodology, including the dataset creation and the hyper-parameter tuning process. The design of experiments and the three different scenario configurations are presented in Section [7](https://arxiv.org/html/2409.11276v1#S7 "7 Experiment Design ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"). The experiments’ results and the agents’ behavioral analysis are described in Sections [8](https://arxiv.org/html/2409.11276v1#S8 "8 Results ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") and [9](https://arxiv.org/html/2409.11276v1#S9 "9 Behavioral Analysis ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"), respectively. The dataset ablation study details are in Section [10](https://arxiv.org/html/2409.11276v1#S10 "10 Dataset Ablation Study ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"). Finally, we discuss some findings and limitations of the work in Section [11](https://arxiv.org/html/2409.11276v1#S11 "11 Discussion ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") and conclude the paper with Section [12](https://arxiv.org/html/2409.11276v1#S12 "12 Conclusions ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments").

## 2 Background

The term Large Language Models (LLMs) typically refers to transformer-based models [[11](https://arxiv.org/html/2409.11276v1#bib.bib11)] of billions of parameters that are trained to perform tasks related to Natural Language Processing (NLP). The input text is usually split into smaller chunks (tokens), and the models are trained to predict the next token $t$ after the input sequence.

Formally, given a series of tokens $t_{1},\dots,t_{N-1}$, a language modelling aims to calculate the joint probability

|  | $P(t_{1}\dots t_{N})=\prod_{i=1}^{N}P(t_{i}&#124;t_{1},\dots,t_{N-1})$ |  |

Large language models use neural networks (transformers) to estimate this probability and sample the next token in the sequence ${t}_{N}\sim f_{\theta}(t_{N}|t_{1},\dots,t_{N-1})$, where $f_{\theta}$ is the trained model with parameters $\theta$.

There are several stages in the training of LLMs. The first stage is the unsupervised pre-training with many tokens. The resulting models can perform next-token prediction and generally perform well in varied tasks related to the data. However, these large models, often called foundational, sometimes do not perform well in certain specialized tasks. Foundational models can adapt to tasks such as classification using Supervised Fine-Tuning (SFT) [[12](https://arxiv.org/html/2409.11276v1#bib.bib12), [13](https://arxiv.org/html/2409.11276v1#bib.bib13)]. Fine-tuning requires labeled data, but these datasets are usually much smaller than those used for the pre-training phase. Sometimes, the dataset can be produced using the answers of a more capable model that is used as a teacher. This method is also called distilled SFT (dSFT) [[14](https://arxiv.org/html/2409.11276v1#bib.bib14), [15](https://arxiv.org/html/2409.11276v1#bib.bib15)]. A common fine-tuning use case is instruction fine-tuning [[16](https://arxiv.org/html/2409.11276v1#bib.bib16)], which aims to align language models with user intent and thus create useful assistants and chatbots. These days, many models release an instruction fine-tuned version along with the pre-trained model.

Fine-tuning large models with billions of parameters can be time and resource-intensive. Several Parameter-Efficient Fine-Tuning (PEFT) methods have been proposed that allow the training of fewer parameters while retaining the knowledge of the base foundational model. The method used in this work is a version of Low-Rank Adaptation (LoRA) [[17](https://arxiv.org/html/2409.11276v1#bib.bib17)]. LoRA is a supervised fine-tuning method that introduces low-rank matrices that adapt the model’s weights during training. The result of training with LoRA is the trained adapters, which can then be merged into the base model.

A further step towards alignment of the model with human goals is Reinforcement Learning from Human Feedback (RLHF), which requires the creation of a reward model that can evaluate the model’s responses. Once trained, the reward model can be used with any reinforcement learning algorithm to update the base model to align with human preferences. RLHF requires datasets that contain human feedback and ratings of language model responses.

When faced with a complex task, humans tend to deconstruct it into simpler subtasks and solve them individually. Pre-trained language models, especially earlier versions such as GPT-3, were shown to have limited abilities when it comes to logical reasoning and planning. However, providing one or more examples as input can improve the model’s ability to answer questions requiring reasoning [[18](https://arxiv.org/html/2409.11276v1#bib.bib18)]. The idea of guiding or teaching the model about the expected behavior during inference time using prompts is called In-context Learning (ICL). In contrast to fine-tuning, ICL does not require any training and places the task of teaching the model to the user that supplies the prompts.

Leveraging a few demonstrative examples to guide a language model’s behavior is known as k-shot example learning, where $k$ represents the number of examples provided. The core idea is that by exposing the model to a handful of high-quality examples illustrating the desired reasoning process and expected outputs, the model can learn to generalize and apply that knowledge to solve novel problems.

One popular technique explored to improve k-shot example learning is Chain of Thought (CoT) [[19](https://arxiv.org/html/2409.11276v1#bib.bib19)]. The CoT approach provides the model with example inputs and outputs and explicitly demonstrates the step-by-step reasoning used to arrive at the final answer. The goal of explicitly showcasing the thought process is to help the model better understand the underlying logic and gain insights into the reasoning required to solve the given problem.

## 3 Related Work

### 3.1 Planning and Exploration

Pre-trained large language models (LLMs) have shown some reasoning capabilities, using prompting techniques and ICL. However, they often face challenges in long-term planning, leading to occasional hallucinations and ineffective or irrelevant actions. Several frameworks have been developed to address these limitations that employ multi-stage prompting to enhance LLM agents’ planning abilities by integrating reasoning and self-reflection. Notable examples include ReAct [[20](https://arxiv.org/html/2409.11276v1#bib.bib20)], Reflexion [[21](https://arxiv.org/html/2409.11276v1#bib.bib21)], Describe, Explain, Plan and Select (DEPS)[[22](https://arxiv.org/html/2409.11276v1#bib.bib22)], and Reasoning via Planning (RAP)[[23](https://arxiv.org/html/2409.11276v1#bib.bib23)].

ReAct [[20](https://arxiv.org/html/2409.11276v1#bib.bib20)] combines reasoning with action. Reflexion [[21](https://arxiv.org/html/2409.11276v1#bib.bib21)] advances this concept by introducing a sequential decision-making framework that includes self-reflection and evaluation, assessing the quality of actions and trajectories within an episode. This framework utilizes short-term memory to track actions during an episode and long-term memory to inform future decisions, allowing agents to learn from past experiences.

The RAP framework [[23](https://arxiv.org/html/2409.11276v1#bib.bib23)] takes a different approach and uses an LLM as a world model to simulate actions and evaluate outcomes. RAP uses Monte Carlo Tree Search (MCTS) to explore various reasoning paths, with the LLM incrementally building a reasoning tree that considers the most promising steps. By leveraging the world model to predict potential outcomes, RAP helps the LLM refine its reasoning process through rewards derived from these outcomes. This approach allows the agent to simulate and anticipate the consequences of different actions, improving its planning and decision-making abilities.

In addition to planning tasks, LLM-based agents have been successful in exploration, as demonstrated by Du et al.[[6](https://arxiv.org/html/2409.11276v1#bib.bib6)] and Wang et al.[[7](https://arxiv.org/html/2409.11276v1#bib.bib7)]. Voyager used an automatic curriculum, a skill library, and an iterative prompting mechanism for open-ended exploration in the Minecraft game environment. Similarly, Du et al. proposed using pre-trained LLMs to provide ”intrinsic motivation” that guides the exploration and goal setting of the agent in the Crafter[[24](https://arxiv.org/html/2409.11276v1#bib.bib24)] and Housekeep[[25](https://arxiv.org/html/2409.11276v1#bib.bib25)] environments.

Other works have also explored using LLMs for planning tasks, such as Spring [[26](https://arxiv.org/html/2409.11276v1#bib.bib26)], which uses an LLM to ”study” a paper describing the Crafter game environment. Using the summarized knowledge from the paper, it employs a guided Q&A approach with the LLM to select the best action.

Earlier research has explored using LLMs for classical planning tasks [[27](https://arxiv.org/html/2409.11276v1#bib.bib27), [28](https://arxiv.org/html/2409.11276v1#bib.bib28)], particularly within Planning Domain Definition Language (PDDL) domains. These studies highlighted the challenges of generating plans across diverse domains. Silver et al. [[29](https://arxiv.org/html/2409.11276v1#bib.bib29)] demonstrated improved performance by using LLMs to generate plans as Python programs, refining them through debugging. However, classical planning approaches are limited to fully observable environments, which do not apply to cybersecurity domains’ complex and dynamic settings.

### 3.2 Red Teaming

The integration of Large Language Models (LLMs) in cybersecurity, particularly in automating penetration testing and red teaming, is a rapidly developing area of research. Previous studies underscore the diverse capabilities of LLMs within this realm and illustrate their potential and limitations.

Happe et al. (2023) propose a structured approach to penetration testing by categorizing tasks into higher and lower levels: higher-level tasks focus on planning and orchestration, while lower-level tasks involve executing specific attack tools. However, the practical application of their findings remains constrained, as the attacks were evaluated in a limited set of scenarios [[30](https://arxiv.org/html/2409.11276v1#bib.bib30)].

Moskal et al. (2023) further explore the capabilities of LLMs by creating a controlled environment with a single target and employing a heuristic planner to assess LLMs’ performance in basic penetration testing tasks [[31](https://arxiv.org/html/2409.11276v1#bib.bib31)]. Their work provides an initial framework for evaluating how LLMs might handle specific tasks in a simplified setting.

More recently, the PentestGPT framework[[32](https://arxiv.org/html/2409.11276v1#bib.bib32)] seeks to automate penetration testing more comprehensively, with evaluations conducted across various machines on platforms like HacktheBox. This approach indicates a more practical application of language models in varied environments. However, it is also focused more on the low-level tasks.

Raman et al. (2024) investigate the performance of commercial LLMs in the context of Certified Ethical Hacking (CEH) certification questions[[33](https://arxiv.org/html/2409.11276v1#bib.bib33)]. Similarly, Tann et al. (2023) analyze the application of LLMs in Cisco Networking certifications and Capture The Flag (CTF) challenges [[34](https://arxiv.org/html/2409.11276v1#bib.bib34)]. While important, these evaluations do not necessarily translate to practical automation capabilities in realistic penetration testing situations; however, they provide insight into the models’ understanding of security and networking concepts.

Overall, while findings related to certification exams and controlled environments contribute valuable insights into LLMs’ knowledge regarding security concepts, they do not fully address the complexities of automating penetration testing in dynamic and diverse setups. Thus, continued exploration in realistic scenarios remains imperative for realizing the full potential of LLMs in cybersecurity.

## 4 The NetSecGame Environment

NetSecGame is a network security simulation environment that follows a reinforcement learning (RL) architecture [[10](https://arxiv.org/html/2409.11276v1#bib.bib10), [35](https://arxiv.org/html/2409.11276v1#bib.bib35)]. The NetSecGame is designed to allow agents to play in a network by submitting high-level actions and providing back an observation of the current state of the environment for that agent. This allows RL agents to play, but it is not limited to agents. Humans can also play using an interactive interface with or without collaboration with an LLM-based assistant. Allowing humans to play is critical for evaluating attacking and defending strategies.

##### Network Configuration

It is possible to configure NetSecGame in different scenarios. Each scenario has any number of host computers, services on each host, data on each host, routers, and how they are connected. It is also possible to define a host on the internet. Each scenario also has a specific goal (can be fixed or random), starting position of the attacker agent (can be random), probabilities of detection of each action, and probabilities of success of each action.

##### Game State

The objects defined in the environment are Networks, IPs, Services, and Data. The state is defined as a list of: known networks, known hosts, controlled hosts, known services, and known data. Known networks are the networks the agent knows about, the known hosts are the hosts known to the agent because it scanned for them, the controlled hosts are the hosts that the agent exploited successfully, and now controls, the known services are the services (ports) known for the agent after scanning them, and the known data are the data known to the agent because it scans for it inside the host.

An example of the state representation as given to the agents:

 <svg class="ltx_picture" height="123.03" id="S4.SS0.SSS0.Px2.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,123.03) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="95.48" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">```
State<nets:{172.18.0.0/24, 172.18.2.0/24, 172.18.1.0/24};
known_hosts:{172.18.0.72, 172.18.0.55, 172.18.0.62, 172.18.0.9,
    172.18.1.25, 172.18.0.156, 43.82.9.214, 172.18.0.76};
controlled_hosts:{172.18.0.72, 172.18.0.55, 43.82.9.214};
konwn_services:{172.18.0.72:
    {Service(name=’openssh’, type=’passive’, version=’8.1.0’,
    is_local=False)},
172.18.0.55:
    {Service(name=’postgresql’, type=’passive’, version=’14.3’,
    is_local=False)
    };
known_data:{
    172.18.0.55: { Data(owner=’User1’, id=’DatabaseData’)}}>

```</foreignobject></g></g></svg> 

##### Goals

The goal of each scenario is flexible and can be defined as a specific state, e.g., the presence of data in a specific host or a specific host that needs to be controlled by the agent. Randomization of the goal per episode is also an option. For example, the data that needs to be present in a specific host can change in each episode. Such approach can be used to evaluate generalization capabilities of the agents.

##### Actions

The action space has five action types, each of which can receive parameters. The action types are: ScanNetwork, FindServices, FindData, ExploitService, and ExfiltrateData. ScanNetwork receives as parameters the destination network; FindServices requires the destination host; FindData requires the destination host; ExploitService receives destination service and host; ExfiltrateData receives the source host, target host to exfiltrate, and target data to exfiltrate. Due to the action parameterization, the size of the action space depends on the number of objects defined in a specific scenario. The success probability for all actions was set to $1.0$ in all scenarios selected for this work.

##### Rewards

The rewards of each action are sparse. Each action that does not lead to the goal state or detection is rewarded with $-1$; if the goal state is reached, the agent gets an additional reward of $100$. A detection by the defender agent terminates the episode and has a reward of $-50$. However, these rewards sent by the environment can be ignored by the agents, who can define their internal reward functions if needed.

##### Stochastic Defender

In this version of NetSecEnv, the agent plays the role of an attacker. However, the environment allows for an omnipresent stochastic defender. The defender is aware of all actions and blocks the attacker agent if certain conditions are met: Firstly, the defender uses a fixed-size time window to detect repeated actions. A threshold of maximum repeats is defined for each action type and upon reaching the threshold, the detection is triggered. Detection thresholds can be defined in the scenario configuration.

Secondly, for Action types FindData, ExploitService, and ExfiltrateData, there is an additional threshold for repeated actions in the complete episode. The episodic repeat detection mechanism uses both the action type and parameters for checking, which means the detection is triggered only if the same action is used multiple times by the agent.

Lastly, for action types ScanNetwok and FindServices, there is a check for consecutive actions. Inspired by port scanning detectors, this detection mechanism evaluates how many actions of type ScanNetwok or FindServices were played in a sequence and if the threshold is reached, the episode is terminated.

Each action has a different probability of detection, and all the parameters on the stochastic defender are configurable.

## 5 LLM Agent Design

![Refer to caption](img/05f7dd185ca208fb35d0ecd57cfa80e1.png)

Figure 1: LLM Agent Components

The LLM agent used for all the experiments was introduced in [[10](https://arxiv.org/html/2409.11276v1#bib.bib10)]. As part of this work, we re-used the prompts and structure of the agent to directly compare the fine-tuned LLMs introduced in this work with the results of the commercial pre-trained models introduced in  [[10](https://arxiv.org/html/2409.11276v1#bib.bib10)].

Following the taxonomy proposed by [[8](https://arxiv.org/html/2409.11276v1#bib.bib8)], the LLM agent architecture comprises the following components: Profile, Memory, Planning, and Action (Figure [1](https://arxiv.org/html/2409.11276v1#S5.F1 "Figure 1 ‣ 5 LLM Agent Design ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments")).

##### Profile

This component contains all the information provided to the models as personality information. In our case, the personality of a penetration tester was handcrafted in the prompt.

##### Memory

The memory component is a list of the previous actions that were taken by the agent and the current state of the environment: $\mathcal{M}=\{a_{t-k},...,a_{t-1},s_{t}\}$. All the memory elements are presented as part of the prompt (unified memory). The list of actions has a fixed size and contains the last $k$ actions the agent took. The agent is also provided with the current state of the environment, which is considered part of the agent’s short-term memory. The agent does not use long-term memory between the episodes; therefore, each episode starts with an initial state and no action memory.

##### Planning

The agent uses the ReAct framework for planning and selecting the next actions. ReAct involves reasoning as the first step of the process and acting as the second step. Based on the current state of the environment $s_{t}$ the agent asks the LLM to analyze the objects in the state (IPs, Networks, Services, Data) and reason about the actions that can be taken for each object. In the second step, the agent uses the LLM response and the memory elements to decide and propose an appropriate action $a_{t}$. To produce a valid action, the agent is also provided with examples of actions (In-Context Learning). The memory incorporates feedback as an intrinsic reward. The agent evaluates each action as ”helpful” or ”not helpful” based on whether or not the state $s_{t+1}$ is different than $s_{t}$. Since the environment is deterministic and no action can remove objects from the state, an action is considered ”helpful” if it allows the agent to discover more objects and reach a new state.

## 6 Supervised Fine-tuning Methodology

![Refer to caption](img/9f888d882025ab9a0e8945a4dc4c2b72.png)

Figure 2: Supervised Fine-tuning Methodology

This study was conducted through several steps (Figure [2](https://arxiv.org/html/2409.11276v1#S6.F2 "Figure 2 ‣ 6 Supervised Fine-tuning Methodology ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments")). The first step of the process was to identify the weaknesses of the base models and create a dataset for supervised fine-tuning. The details of the dataset creation are presented in Section [6.1](https://arxiv.org/html/2409.11276v1#S6.SS1 "6.1 Dataset Creation ‣ 6 Supervised Fine-tuning Methodology ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"). The base model used was Zephyr-7b-$\beta$ [[15](https://arxiv.org/html/2409.11276v1#bib.bib15)] which is an instruction-tuned model based on Mistral-7B-v0.1 [[36](https://arxiv.org/html/2409.11276v1#bib.bib36)]. The reasons the Zephyr model was selected were multiple: firstly, an already instruction-tuned model did not require time and effort to perform the instruction tuning. Secondly, the model had a good ability to produce valid JSON strings, and thirdly, in our initial experiments it showed that it has enough background knowledge of networking and security. Finally, we decided to select an open-source model with seven billion parameters so that it would be possible to fine-tune it using one GPU, which allows reproducibility of our results with minimal cost. The methodology followed in this paper is general enough and can be applied to other models of similar strength.

Supervised fine-tuning is a training process that involves many hyperparameters. The method followed in this work is based on the Quantized LoRA (QLoRa) [[37](https://arxiv.org/html/2409.11276v1#bib.bib37)]. QLoRA allows a quantized version of the base model to be used during training requiring less memory in the GPU card. The rank of the adapters $r$ and the weight $\alpha$ are the two main hyperparameters used in LoRA, and the details about their selection are presented in Section [6.2](https://arxiv.org/html/2409.11276v1#S6.SS2 "6.2 Hyper-parameter Tuning ‣ 6 Supervised Fine-tuning Methodology ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"). For the fine-tuning process, we used the HuggingFace alignment-handbook library and scripts [[38](https://arxiv.org/html/2409.11276v1#bib.bib38)].

After the hyperparameter tuning, we executed a series of experiments to evaluate the performance of our fine-tuned model. These experiments compared Hackphyr against established baselines across three distinct scenarios (Section [7.1](https://arxiv.org/html/2409.11276v1#S7.SS1 "7.1 Scenario Descriptions ‣ 7 Experiment Design ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments")). All the supervised fine-tuning and the scenario experiments were executed in a single V100 Nvidia GPU card with 32GB of NVRAM.

### 6.1 Dataset Creation

Supervised fine-tuning requires high-quality data that help the language model perform well in a narrow task. The strategy for creating the dataset addressed issues we observed using the smaller models in the NetSecGame environment. The process was automated initially by asking stronger LLMs to play the role of a teacher who generates questions and answers to teach their students about the NetSecGame environment. After the automated generation of the questions and answers, we performed a human evaluation to ensure that the answers were of good quality, edited the incorrect ones, and discarded duplicates.

The dataset comprises 1641 questions and answers generated as three separate parts (Figure [2](https://arxiv.org/html/2409.11276v1#S6.F2 "Figure 2 ‣ 6 Supervised Fine-tuning Methodology ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments")). Each part has a different focus and aims to address a specific problem observed while we were testing various open-source LLMs. The complete dataset was published on HuggingFace and is accessible for further research and experimentation [[39](https://arxiv.org/html/2409.11276v1#bib.bib39)].

#### 6.1.1 Part I - Environment Understanding

The first part of the dataset contains questions and answers that test the model’s ability to understand the current status of the environment and the provided rules. The dataset was generated automatically using three different LLMs: GPT-4, GPT-3.5-turbo from OpenAI, and Claude from Anthropic. The total number of questions and answers generated was $1080$.

Given a set of 18 states collected by previous game runs, the models were asked to generate 20 questions that test a student’s ability to comprehend the game environment and the rules:

 <svg class="ltx_picture" height="52.81" id="S6.SS1.SSS1.p3.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,52.81) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="25.25" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">```
Provide 20 questions and answers that test the students
knowledge regarding the specific status and rules.

```</foreignobject></g></g></svg> 

An example of a generated question and answer can be found in Appendix [A.1](https://arxiv.org/html/2409.11276v1#A1.SS1 "A.1 Part I ‣ Appendix A Dataset examples ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments").

#### 6.1.2 Part II - Generating Valid Actions

The second part of the dataset contains questions that test the model’s ability to generate valid actions, both in terms of syntax (JSON format) and in terms of semantics (validity in the specific state). The final number of questions and answers in the second part of the dataset is $450$. The following prompt generated this part of the dataset from 18 distinct states:

 <svg class="ltx_picture" height="52.81" id="S6.SS1.SSS2.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,52.81) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="25.25" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">```
Provide 20 questions and answers that test the student’s
ability to generate valid and well-formatted actions in the
current status.

```</foreignobject></g></g></svg> 

An example of a generated question and answer can be found in Appendix [A.2](https://arxiv.org/html/2409.11276v1#A1.SS2 "A.2 Part II ‣ Appendix A Dataset examples ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments").

#### 6.1.3 Part III - Generating Good Actions

This dataset part aims to teach the fine-tuned models how to make correct decisions given a specific environment state. It comprises 113 questions and answers generated from previous game runs using GPT-4 in the small scenario [[10](https://arxiv.org/html/2409.11276v1#bib.bib10)].

The state-action pairs were selected in a way where only actions that led to a new state of the environment were used. This choice is because actions that do not lead to a new state, i.e., the discovery of a new element, are either repetitions or not useful.

An example of a generated question and answer can be found in Appendix [A.3](https://arxiv.org/html/2409.11276v1#A1.SS3 "A.3 Part III ‣ Appendix A Dataset examples ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments").

### 6.2 Hyper-parameter Tuning

Instead of the common machine learning evaluation approach of splitting between train and test sets, the complete dataset was used for training following the SFT methodology described in section [2](https://arxiv.org/html/2409.11276v1#S2 "2 Background ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"). Since random IPs and random data exfiltration goals were set for each episode, the agent is evaluated under different conditions than the ones in the training dataset.

Then, the resulting model was tested on the small scenario [7.1](https://arxiv.org/html/2409.11276v1#S7.SS1 "7.1 Scenario Descriptions ‣ 7 Experiment Design ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") for 150 episodes using the LLM-based agent as described in section [5](https://arxiv.org/html/2409.11276v1#S5 "5 LLM Agent Design ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") with a memory of the last ten actions.

A hyperparameter grid search was conducted for selecting the LoRA $r$ and $\alpha$ hyperparameters. The following parameter values were used during the grid search:

*   1.

    LoRA r: 4, 8, 16, and 32

*   2.

    LoRA $\alpha$: 4, 8 16 and 32

The rest of the hyper-parameters were set to default values according to the HuggingFace alignment handbook [[38](https://arxiv.org/html/2409.11276v1#bib.bib38)].

![Refer to caption](img/0717fad0c9095e8000dff3b67a39d446.png)

Figure 3: Win rate(%) confidence intervals by model. The blue area indicates the desired effect size.

The performance of each agent in terms of the win rate, along with the confidence intervals (CI) over the 150 episodes, is shown in Figure [3](https://arxiv.org/html/2409.11276v1#S6.F3 "Figure 3 ‣ 6.2 Hyper-parameter Tuning ‣ 6 Supervised Fine-tuning Methodology ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"). The number of 150 episodes was chosen based on a power analysis, with significance level $\alpha=0.05$ and power $1-\beta=0.8$ aiming for an effect size between 85% and 95%. The purpose of using confidence intervals is to focus on those models whose intervals do not overlap.

From figure [3](https://arxiv.org/html/2409.11276v1#S6.F3 "Figure 3 ‣ 6.2 Hyper-parameter Tuning ‣ 6 Supervised Fine-tuning Methodology ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"), we can observe that only the last five hyperparameter combinations showed significant differences between the analyzed effect sizes. The remaining hyperparameters do not provide strong enough evidence to conclude a significant difference between the win rate values at the given confidence level. However, the considerable overlap of the CIs suggests that the sample data do not provide strong enough evidence to conclude a significant difference between the win rates at the given confidence level. Therefore, we selected the agent that used a model with LoRa parameters set to $r4\_a32$, as this configuration consistently demonstrated the highest average win rate.

The final values of the most important hyper-parameters are provided in Table [1](https://arxiv.org/html/2409.11276v1#S6.T1 "Table 1 ‣ 6.2 Hyper-parameter Tuning ‣ 6 Supervised Fine-tuning Methodology ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments").

Table 1: Final hyper-parameter values

| Parameter | Value |
| --- | --- |
| LoRA r | 4 |
| LoRA $\alpha$ | 32 |
| training epochs | 2 |
| gradient accumulation steps | 2 |
| learning_rate | 2.0e-04 |
| lr scheduler type | cosine |
| max seq length | 2048 |
| number of GPUs | 1 |
| training batch size | 1 |

## 7 Experiment Design

For evaluating the agents, we have selected three scenarios on top of the NetSecGame environment described in Section [4](https://arxiv.org/html/2409.11276v1#S4 "4 The NetSecGame Environment ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"). The goal in all the scenarios was to exfiltrate specific data to the C&C server. Agents must perform a sequence of actions to discover data on a computer in the network and then exfiltrate it.

### 7.1 Scenario Descriptions

The first two data exfiltration scenarios (denoted as small and full scenarios) are depicted in Figure [4](https://arxiv.org/html/2409.11276v1#S7.F4 "Figure 4 ‣ 7.1 Scenario Descriptions ‣ 7 Experiment Design ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"). These two scenarios share the same network topology: two subnets, one with clients and one with servers, connected by a router. The small scenario differs from the full scenario only in that there is a single client on the client subnet.

![Refer to caption](img/32bff79cc30dc6d868ec445d0694eb8a.png)

Figure 4: Small and Full network scenarios [[10](https://arxiv.org/html/2409.11276v1#bib.bib10)]. The small scenario has only one client in the client subnet, while the full scenario has all five clients.

The attacker starts the scenario in one of the client machines, playing the role of an attacker who has gained a foothold in a network. The attacker also controls a machine on the internet, where the command and control (C&C) server is hosted.

Both scenarios require a minimum of five steps to exfiltrate the data. Still, the solution is far from trivial, given that the IP addresses and goals change in every episode, and the latter presents a challenge to both LLM and traditional RL agents. A more detailed description of both scenarios is presented in [[10](https://arxiv.org/html/2409.11276v1#bib.bib10)].

The third scenario (denoted as the three-networks scenario) consists of three different subnetworks inside a fictional small-medium enterprise (Figure [5](https://arxiv.org/html/2409.11276v1#S7.F5 "Figure 5 ‣ 7.1 Scenario Descriptions ‣ 7 Experiment Design ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments")). The client subnet contains the client hosts (PCs, etc.). The first server subnet (subnet A) contains some servers that are immediately accessible by the clients, and the second server subnet (subnet B) contains two servers that are only accessible by subnet A. Firewall rules prevent the clients from accessing the servers in subnet B directly. The attacker aims to exfiltrate the data from one of the servers in the subnet B to the C&C server on the internet. The attacker begins with a foothold in the client subnet and must first gain control of one of the servers in subnet A. Then they have to perform a network scan of network B to discover and exploit the servers and gain control. Once the correct server is controlled and the data are discovered, they can be exfiltrated to the C&C server.

![Refer to caption](img/a5c30792bb1f5931b92ebc1dc4c29ef1.png)

Figure 5: Three Subnets Scenario Topology. The clients can only access subnet A directly. The goal is to exfiltrate data from subnet B by first gaining a foothold to subnet A.

The three-networks scenario requires at least three more steps to solve than the previous two. This scenario is more demanding, and it is used to test the limits of the LLMs. As in the previous two scenarios, the IP addresses were randomized in each episode, and the scenarios were tested with and without the stochastic defender with thresholds; however, in this scenario, the goal is not randomized.

A summary of the main differences between the three scenarios is presented in Table [2](https://arxiv.org/html/2409.11276v1#S7.T2 "Table 2 ‣ 7.1 Scenario Descriptions ‣ 7 Experiment Design ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments")

| Scenario | Small | Full | Three-Networks |
| Network Topology | One client and one server subnet, router | One client and one server subnet, router | One client and two server subnets (A & B), client have direct access only to subnet A, router |
| Config | Single client and multiple servers | Multiple clients and multiple servers | Multiple clients and multiple servers |
| Episode Setup | Randomized IPs and goal | Randomized IPs and goal | Randomized IPs, fixed goal on subnet B |
| Steps | Five | Five | Eight |
| Purpose | Fine-tuning and hyper-parameter tuning | Test performance under more complex conditions | Test adaptability to different network configurations |

Table 2: Comparison of Data Exfiltration Scenarios

### 7.2 Evaluation Procedure

Each scenario was designed with a specific purpose in mind for facilitating the evaluation of the agents’ performance across different level of complexity levels and conditions (See Table [2](https://arxiv.org/html/2409.11276v1#S7.T2 "Table 2 ‣ 7.1 Scenario Descriptions ‣ 7 Experiment Design ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments")).

In particular, the small scenario was mainly used for fine-tuning and hyper-parameter selection, as similar examples were used during the model’s training. We tested this scenario with and without a defender to establish a performance baseline.

The full scenario is more complex and involves more clients that can be scanned for services and finding the data to exfiltrate. Since the full scenario has more clients and was not seen during training, we used it to analyze the model’s performance degradation under slightly more complex conditions.

Finally, the three networks scenario is significantly more complex, involving additional networks and more steps required for data exfiltration. This scenario introduced a different network topology not encountered during training, and it helped analyze the model for potential overfitting. This ensures that the fine-tuned models can adapt to diverse and demanding network configurations without retraining.

The language models used for testing the scenarios were GPT-4, GPT-3.5-turbo, and Zephyr-7b-$\beta$, along with Hackphyr. Similar to Section [6.2](https://arxiv.org/html/2409.11276v1#S6.SS2 "6.2 Hyper-parameter Tuning ‣ 6 Supervised Fine-tuning Methodology ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"), we evaluated the performance of each model over 150 episodes with 100 maximum steps per episode. Each episode was run independently and IPs and goals were generated with a different seed.

In addition, we conducted some baseline comparisons using a random agent and a tabular Q-learning agent [[40](https://arxiv.org/html/2409.11276v1#bib.bib40)] similar to [[10](https://arxiv.org/html/2409.11276v1#bib.bib10)]. For the Q-learning agent, it was not possible to randomize the IP per episode because the Q-table was fixed. However, the goal was randomized in each episode. The Q-agent was trained for 50,000 episodes, and the evaluation was performed on the trained agent. The random agent was run for 2,000 episodes.

For all agents, we measured the percentage of wins (win_rate), the average steps, and the average returns. A win is any episode where the agent reaches the goal state within the 100 maximum steps.

## 8 Results

The results of the experiments without and with the stochastic defender are presented in Tables [3](https://arxiv.org/html/2409.11276v1#S8.T3 "Table 3 ‣ 8.1 Scenarios Without Defender ‣ 8 Results ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") and [4](https://arxiv.org/html/2409.11276v1#S8.T4 "Table 4 ‣ 8.2 Scenarios With Defender ‣ 8 Results ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"), respectively. The tables present the win rates and returns for the small, full, and three subnets scenarios.

For the small and full scenarios, the results were taken from [[10](https://arxiv.org/html/2409.11276v1#bib.bib10)]. For the three subnets scenario, all the LLM-based agents were run using exactly the same setup, including prompts, temperatures, and other relevant parameters. This also applied to the Q-learning agent, with settings such as the learning rate, epsilon, and the number of episodes being consistent across the experiments.

### 8.1 Scenarios Without Defender

Table 3: Average returns and win rates of all agents across different scenarios without defender with 100 maximum steps per episode.

|  | Small | Full | 3 Subnets |
| --- | --- | --- | --- |
| Agent | Win% | Return | Win% | Return | Win% | Return |
| --- | --- | --- | --- | --- | --- | --- |
| Random | 40.99 | -43.43 | 31.16 | -57.49 | 4.86 | -93.28 |
| Q-learning | 67.41 | 47.55 | 58.74 | 48.00 | 0.00 | -100.00 |
| GPT-3.5-turbo | 50.00 | -16.13 | 30.00 | -51.67 | 10.00 | -87.50 |
| GPT-4 | 100.00 | 83.10 | 100.00 | 77.13 | 82.35 | 18.78 |
| Zephyr-7b-$\beta$ (base) | 30.46 | -51.80 | 23.65 | -51.77 | 1.75 | -98.80 |
| Hackphyr (ours) | 94.00 | 72.83 | 89.10 | 61.03 | 50.34 | -14.26 |

Regarding win rate and average reward return, GPT-4 agents consistently outperform all the LLM agents in all the scenarios without defenders(Small, Full, and Three Subnets). On the other hand, the Hackphyr agents demonstrate strong performance, always ranking second to GPT-4 and significantly better when compared with the base Zephyr agent.

In the small scenario, the performance of Hackphyr agents is close to GPT-4 (94% win rate. 72.8 average return). These are expected results since the dataset used for fine-tuning Zephyr contained examples from actions taken by GPT-4-based agents on the small scenario.

The performance in the full scenario is more valuable in terms of win rate and average return. This scenario was completely unseen for the Hackphyr agent during the fine-tuning process described in section [6](https://arxiv.org/html/2409.11276v1#S6 "6 Supervised Fine-tuning Methodology ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"). Despite the decrease compared to the small scenario, the Hackphyr agent showed a win rate of 89% with an average return of 61\. These values were exceptionally better than the Zephyr-based agent and outperformed GPT-3.5-turbo, a larger and more powerful model.

Finally, the complexity of the three subnets scenario caused a decrease in the win rate performance of all language models. The Hackphyr dropped to a 50% win rate and a -14.25 average return. The GPT-4-based agents observed a decrease in win rate of 20% and a drop in average return of 18.78\. The fact that a powerful model such as GPT-4 had a performance degradation shows that this is a harder scenario and that an improved agent design may be required.

The performance of the two baseline agents, Random and Q-learning, highlights the limitations of simpler strategies in the given scenarios. The Random agent consistently underperforms across all scenarios, with win rates of 40.99%, 31.16%, and 4.86% in the Small, Full, and Three Subnets scenarios, respectively, and corresponding negative returns, indicating poor decision-making that fails to generate positive outcomes. The Q-learning agent shows a notable improvement over the Random agent, particularly in the Small and Full scenarios, achieving win rates of 67.41% and 58.74%, respectively, with positive returns in both cases (47.55 and 48.00). However, in the more complex Three Subnets scenario, the Q-learning agent struggles significantly, with a 0% win rate and a return of -100.00, reflecting the limitations of the tabular RL methods. The results underscore the limited effectiveness of these baseline methods, particularly in more challenging environments, compared to more advanced models like GPT-4 and Hackphyr.

In summary, the Hackphyr agents demonstrated remarkable performance, particularly considering their fine-tuning process and the complexity of the scenarios they faced. While GPT-4 consistently outperforms all other agents, Hackphyr stands out as a robust and adaptable model, achieving results that often come close to those of GPT-4, significantly better than the other LLM-based agents such as GPT-3.5-turbo.

### 8.2 Scenarios With Defender

Table 4: Average returns and win rates of all agents across different scenarios with defender with 100 maximum steps per episode.

|  | Small | Full | 3 Subnets |
| --- | --- | --- | --- |
| Agent | Win% | Return | Win% | Return | Win% | Return |
| --- | --- | --- | --- | --- | --- | --- |
| Random | 3.76 | -65.57 | 2.72 | -66.56 | 0.13 | -70.73 |
| Q-learning | 77.96 | 54.91 | 71.00 | 45.38 | 0.00 | -70.45 |
| GPT-3.5-turbo | 20.00 | -34.27 | 16.67 | -58.00 | 6.67 | -63.63 |
| GPT-4 | 83.33 | 58.83 | 53.33 | 8.80 | 36.36 | -21.69 |
| Zephyr-7b-$\beta$ (base) | 3.00 | -66.40 | 3.33 | -66.68 | 0.62 | -70.42 |
| Hackphyr (ours) | 59.77 | 33.00 | 44.00 | 3.56 | 23.33 | -37.67 |

Table  [4](https://arxiv.org/html/2409.11276v1#S8.T4 "Table 4 ‣ 8.2 Scenarios With Defender ‣ 8 Results ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") shows the results with the stochastic defender. In all the cases, similar to the scenarios with a defender, the Hackphyr agents outperformed the agent based on Zephyr without fine-tuning.

For the small scenario, the Hackphyr agent showed a win rate of 59.77% with a return value of 33\. The GPT-4-based agent won 83 episodes with an average return value of 58\. The agent using GPT-3.5-turbo was capable of winning in just 20% of the episodes.

Similarly to the scenario without the stochastic defender, the performance decreased in the full and the three subnets scenarios. In the full scenario, the Hackphyr agent showed a win rate value of 44% with an average return of 3.5\. GPT-4-based agents performed slightly better, with a win rate of 53% and an average return value of 8.8.

The three subnets scenario was difficult for all the agents. Despite the difficulties, the FT-Zephyr agent won in 23% of the episodes, far beyond the results of a larger model such as GPT-3.5-turbo with only a 6% win rate. GPT-4 only was able to win 36% of the time. In all the cases, the average return was negative.

When the stochastic defender was present, all the LLM-based agents’ performance decreased. However, the Hackphyr agent performed relatively well, outperforming other LLM-based agents. Moreover, in the three subnets scenario, the Hackphyr agent outperformed all LLM-based agents (except for GPT-4) by a significant margin. This performance demonstrates that the model can succeed in a completely different scenario, indicating that it did not simply memorize data from the small scenario used for fine-tuning. This suggests a versatile and adaptable capability.

Regarding the baseline agents, it is worth mentioning that Q-learning displayed considerable effectiveness in scenarios with defenders, sometimes even exceeding the performance of both GPT-4 and Hackphyr. Specifically, it achieved a win rate of 77.96% with an average return of 54.91 in the small scenario and a win rate of 71.00% with an average return of 45.38.

These results could be attributed to the defender’s presence, which helped the agent learn a policy that avoided repetitions penalized due to the detection. The resulting policy made the agent perform even better than in scenarios without a defender. However, it faced significant challenges in the Three Subnets Scenario, highlighting potential limitations in more complex environments.

Finally, it must be noted that none of the LLM prompts had instructions to avoid the defender. Agents like the GPT-4-based sometimes followed a breadth-first approach, scanning hosts for services sequentially, which can trigger the stochastic defender.

## 9 Behavioral Analysis

We define a trajectory as an agent’s sequence of actions during one episode. Formally, a trajectory $\tau$ can be expressed as:

|  | $\tau=\{a_{i}\}_{i=1}^{T}$ |  |

where $a_{i}$ denotes the action taken by the agent at time step $i$, and $T$ is the total number of time steps in the episode. Let $\mathcal{T}$ represent a set of trajectories expressed as:

|  | $\mathcal{T}=\{\tau_{j}\}_{j=1}^{N}$ |  |

where $\tau_{j}$ denotes the $j$-th trajectory in the set, and $N$ is the total number of trajectories.

In this context, the goal was to analyze the set of trajectories $\mathcal{T}$ to understand the rationality and correctness of the agent’s behavior throughout the episodes. We performed a graph analysis to understand the transitions between actions taken by an agent that helped detect invalid or incorrect action transition patterns.

![Refer to caption](img/fa370429ba780372d3e157ab68e1df0d.png)

(a) Transition Matrix

![Refer to caption](img/035ba4261cb20cd99b2a3303d5aff2fe.png)

(b) Key Action Transitions

Figure 6: GPT-4 action transitions

We considered GPT-4 the reference agent since it showed the best results in all the scenarios. Figure [6](https://arxiv.org/html/2409.11276v1#S9.F6 "Figure 6 ‣ 9 Behavioral Analysis ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") illustrates the transition pathways the GPT-4 agent takes. The heatmap in Figure [6(a)](https://arxiv.org/html/2409.11276v1#S9.F6.sf1 "In Figure 6 ‣ 9 Behavioral Analysis ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") shows the probabilities of transitioning from one action to another, with higher probabilities highlighted in warmer colors. In Figure [6(b)](https://arxiv.org/html/2409.11276v1#S9.F6.sf2 "In Figure 6 ‣ 9 Behavioral Analysis ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"), the key action transition graph visually represents the most relevant transitions (with probabilities greater or equal to 0.20) among the possible actions for GPT-4\. Actions such as Start and Invalid are distinctly marked in green and red, respectively, to highlight that they are not actions an agent can choose from. In particular, an invalid action can be caused by semantic or syntactic errors. A semantic error occurs when the action taken is not allowed given the current environment state. On the other hand, a syntactic error is caused when the agent generates an invalid string representation of the action.

The transition matrix from Figure [6(a)](https://arxiv.org/html/2409.11276v1#S9.F6.sf1 "In Figure 6 ‣ 9 Behavioral Analysis ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") shows that the GPT-4-based agent is, in general, very confident when taking the next action. Most of the actions transition to the next with high probability. The only actions showing more evenly distributed transitions are the FindData and FindServices.

When analyzing the key action transitions from the GPT-4-based agent (Figure [6(b)](https://arxiv.org/html/2409.11276v1#S9.F6.sf2 "In Figure 6 ‣ 9 Behavioral Analysis ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments")), the agent starts scanning the network with a probability of $1.0$. This action seems logical for potential targets or vulnerabilities. Then, transitioning from ScanNetwork to FindServices with a high probability of $0.93$ is expected, as identifying available services is a key step after scanning a network and identifying new hosts.

The agent’s transition from FindServices to ExploitService with probabilities of $0.5$ makes sense, as exploiting vulnerabilities in identified services is a typical next step after discovering those services. The self-loop in the FindServices can be explained by the additional FindServices actions involving different IP addresses found during the ScanNetwork action.

The high probability ($0.99$) transition from ExploitService to FindData suggests that after exploiting a service, the primary goal is to find valuable data, which is logical in network intrusion scenarios.

Transitioning from FindData to ExfiltrateData with probabilities $0.28$ is logical, as extracting valuable data follows finding it. If the agent does not find any data, then a logical action is to start scanning again, and the GPT-4 agent goes to ScanNetwork with $0.39$ probability. The self-loop in the FindData mostly indicates that the action did not discover any data, and GPT-4 decided to search again.

The absence of outgoing transitions to Invalid actions suggests that the GPT-4-based agent has a low probability of generating Invalid actions from any other action. However, when an invalid action does occur, it is always associated with the ExfiltrateData action. This pattern could be due to the fact that ExfiltrateData is the most complex action, having the most parameters. Additionally, whenever an invalid action is generated, the agent’s next action is consistently to ExfiltrateData, suggesting a strong correlation between the two.

The GPT-4-based agent shows a behavior pattern of systematic reconnaissance, service discovery, exploitation, data finding, and exfiltration that matches well-known attacker tactics and techniques like the ones in the MITRE ATT&CK framework¹¹1https://attack.mitre.org/matrices/enterprise/.

![Refer to caption](img/aa80a0d7968449c8f19d605fc57b727a.png)

(a) Transition Matrix

![Refer to caption](img/287844a719465a465e33c18b6c2c1707.png)

(b) Key Action Transitions

Figure 7: Hackphyr action transitions

When comparing the behavior of the Hackphyr-based agent with the agent using GPT-4 (Figure [7](https://arxiv.org/html/2409.11276v1#S9.F7 "Figure 7 ‣ 9 Behavioral Analysis ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments")), the transition matrix shows the agent is not as confident about taking the next actions. The next action transition distribution is more evenly distributed in all cases except the ExploitService and the ScanNetwork. However, when considering the key action transaction graph [7(b)](https://arxiv.org/html/2409.11276v1#S9.F7.sf2 "In Figure 7 ‣ 9 Behavioral Analysis ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"), we can observe the agent also follows a similar behavior pattern of systematic reconnaissance, service discovery, exploitation, data finding, and exfiltration. However, the Hackphyr agent seems to follow a slightly different strategy when scanning networks. In most episodes, the agent tried to scan all the networks before continuing to search for services. This situation is represented by the self-loop transition in the ScanNetwork node.

Another observable difference with the GPT-4-based agent is the transition to invalid actions from the Start. Once the agent has generated an invalid action, the most probable transition is to another invalid action (with a probability of 0.55). The initial environment state information could be harder for the Hackphyr agent to understand. In addition, the lack of short-term memory information during the start could also cause the transition to invalid actions.

Finally, Figure [8](https://arxiv.org/html/2409.11276v1#S9.F8 "Figure 8 ‣ 9 Behavioral Analysis ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") shows the agent’s behavior when using the base Zephyr-7b-$beta$ model. The transition matrix (Figure [8(a)](https://arxiv.org/html/2409.11276v1#S9.F8.sf1 "In Figure 8 ‣ 9 Behavioral Analysis ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments")) shows a clear difference in the distribution of the action transition from GPT-4 and Hackphyr. The main difference is the high transition probability from any other possible action to an Invalid action. In addition, it seems the agent is biased to transition to the FindServices action, no matter the previous action.

![Refer to caption](img/9e9cf1e240cd622aa73acdcffc2527e1.png)

(a) Transition Matrix

![Refer to caption](img/af33731301729781a063f19c40211472.png)

(b) Key Action Transitions

Figure 8: Zephyr-7b-$\beta$ action transitions

The key actions transition graph in Figure [8(b)](https://arxiv.org/html/2409.11276v1#S9.F8.sf2 "In Figure 8 ‣ 9 Behavioral Analysis ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") shows the lack of a clear behavior pattern for conducting the penetration testing techniques. At the Start, follows a logical ScanNetwork action, and the transition to FindServices. However, the required transitions from FindeServices to ExploitService and FindData are not shown in the graph since they have a very low probability. Similar is the case of the transition from FindData to ExfiltrateData. Therefore, beyond scanning the network and finding services, the agent cannot follow the proper action sequence for exfiltrating the data.

In addition, the graph also shows the central role of the Invalid action node. All possible actions transition to an Invalid one, with a considerable probability (between 0.25 and 0.48).

## 10 Dataset Ablation Study

This ablation study aimed to determine which part of the dataset contributes more to the performance of the Hackphyr model during the SFT process.

Since the dataset consists of three parts (see Section [6.1](https://arxiv.org/html/2409.11276v1#S6.SS1 "6.1 Dataset Creation ‣ 6 Supervised Fine-tuning Methodology ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments")), we constructed three different datasets where each one has one of the dataset parts removed:

*   1.

    Dataset UV contains parts I+II, with a focus on Understanding and Valid actions and 1526 samples

*   2.

    Dataset UG, contains parts I+III, with a focus on Understanding and Good actions and 1189 samples

*   3.

    Dataset VG contains parts II+III, with a focus on Valid and Good actions) and 563 samples

Each dataset was used to fine-tune a new model with the exact hyper-parameter settings described in Section [6.2](https://arxiv.org/html/2409.11276v1#S6.SS2 "6.2 Hyper-parameter Tuning ‣ 6 Supervised Fine-tuning Methodology ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments"). We used the ”full” scenario with and without the stochastic defender to compare their performance.

First, we analyze the effectiveness of the actions taken by the agents when using different dataset parts. The actions are categorized as good, valid, or invalid: a good action is performed by the agent that increases the current state; a valid action is appropriate and permissible in the current state, maintaining system integrity or preparing for future beneficial actions, but does not modify the state immediately; and an invalid action is not permissible in the current state and should be minimized by the agent to prevent errors and inefficiencies.

Figure [9](https://arxiv.org/html/2409.11276v1#S10.F9 "Figure 9 ‣ 10 Dataset Ablation Study ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") exhibits the percentage of each type of action each agent takes on the full scenario (with and without a defender) after 150 episodes. In addition, we provide the results for the agent using the base Zephyr-7b-$\beta$ without any fine-tuning process.

![Refer to caption](img/4aad488b78c8522ffc51e60f66a888dc.png)

Figure 9: Percentage of good, valid, and invalid actions taken by agents trained with different dataset parts.

In the scenarios without the stochastic defender, Dataset UV does not show an observable increment in the number of good actions (7%); however, the number of invalid actions is considerably reduced from 44% to 18%. Dataset UG shows a considerable increase in the number of good actions (40%), and invalid actions are reduced to 10%. Finally, in Dataset VG, good actions are slightly below those in Dataset UV (35%), but the number of invalid actions is reduced to 4%.

Similarly, in the case of the stochastic defender, all agents show improvements compared to the base Zephyr agent. Training with Dataset UV has an increase in the good action rate of 23% (up from Zephyr’s 18%), with invalid actions reduced to 16% (down from 50%) and valid actions increased to 61% (up from 33%). Both agents trained with Dataset UG and Dataset VG demonstrate significant improvements, behaving almost the same with good action rates of 55% and 57%, respectively, a drastic reduction in invalid actions to 6% and 5%, and an increase in valid actions to 38%.

In both scenarios (with and without the stochastic defender), the benefits provided by the different parts of the dataset are clear. In particular, the best results are observed on agents trained with UG and VG datasets, with a considerable increase in the number of good actions. In the case of dataset UV, despite reducing the number of invalid actions, the number of good actions remains low.

Being capable of taking valid actions is a necessary condition for an agent to be able to interact with the environment. However, that could not be enough to solve it. To illustrate the agents’ performance, Figure [10](https://arxiv.org/html/2409.11276v1#S10.F10 "Figure 10 ‣ 10 Dataset Ablation Study ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") presents the agents’ results in terms of average return and average steps per episode when using the models across different datasets, along with the win rate over the 150 episodes. The model with the best performance in each metric is highlighted in blue. Note that higher values are better for return and win rate, whereas lower values are preferable for steps.

![Refer to caption](img/4a1336cb5ef3c33cf8af7aa9c9b2423e.png)

(a) Without Stochastic Defender

![Refer to caption](img/d506059ec8f4d3d990b66e55f593d4bd.png)

(b) With Stochastic Defender

Figure 10: Comparison of ablated agents in the large scenario with and without the stochastic defender.

When the stochastic defender is not present (see Figure [10](https://arxiv.org/html/2409.11276v1#S10.F10 "Figure 10 ‣ 10 Dataset Ablation Study ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments")(a)), the performance of the agent fine-tuned with dataset UV is worse in every metric. The agent has not been able to win in any of the 150 episodes (0% Win Rate). Consequently, the average return was -100, which means that the agent has exhausted the maximum number of steps allowed in the environment. So, despite the increase in the number of valid actions, the agent cannot generate enough good actions to explore and solve the environment. On the other hand, the fine-tuned version of agents using datasets UG and VG can solve the environment. In particular, the agent fine-tuned in dataset UG has shown an 85% win rate over the 150 episodes with an average return of around 50.

Similarly, in the scenario with the stochastic defender (see subfigure [10(b)](https://arxiv.org/html/2409.11276v1#S10.F10.sf2 "In Figure 10 ‣ 10 Dataset Ablation Study ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments")), the agent fine-tuned with dataset UV has not been able to win in any of the 150 episodes (0% win rate). The fine-tuned version of agents with the other two datasets can solve the environment. In particular, agents using dataset UG have shown a 50% win rate over the 150 episodes with an average return of around 5.

When the agent uses only Zephyr pre-trained data, there are many invalid actions (see Section [8](https://arxiv.org/html/2409.11276v1#S8 "8 Results ‣ Hackphyr: A Local Fine-Tuned LLM Agent for Network Security Environments") for the results of the base Zephyr-7b-$\beta$ agent). This can be a cause for a low win rate in all scenarios.

To improve the agents’ performance, Part III is essential. Datasets using part III have shown a higher number of good actions. In addition, the combination of Part I and Part III shows the best results regarding win rate and average return. Meanwhile, the combination of Part II and Part III also performs well, though slightly less effectively. The differences in performance between these datasets might be attributed to the varying sizes of the datasets.

Finally, a high number of good actions does not necessarily imply the best agent. For instance, in the case of the scenario with the defender, the agent with Dataset VG shows the highest number of good actions, but in terms of win rate, the results are not as good as the agent using Dataset UG. Good actions are useful for exploring the environment, but sometimes it is not enough to win within the allowed maximum steps.

## 11 Discussion

Our fine-tuned model performed comparatively close to the capabilities of GPT-4, which has at least two orders of magnitude more parameters. While larger models like an 80 billion parameter model are available and may have potentially superior performance, they would require significant computational resources to train and deploy. This could limit their practical application on commercial hardware for specialized tasks.

Moreover, even a 7 billion parameter model, such as Hackphyr, requires a strong GPU to run inference. Quantization techniques hold promise as a potential solution by reducing the memory requirements of the model. However, quantization often comes at the cost of performance degradation. Further investigation is needed into alternative methods that balance computational efficiency with maintainable performance.

In this work, we used the same prompt across all models and agents to compare our agents fairly. However, different language models may respond better to tailored prompts. Future work will focus on optimizing our prompts using techniques such as DSPy [[41](https://arxiv.org/html/2409.11276v1#bib.bib41)] to adapt them to the specific strengths of each model.

Finally, we focused primarily on attacking agents within the NetSecGame environment, mainly as a means of comparison to prior work. However, the principles are highly transferable to designing and implementing attacking and defending agents within varied reinforcement learning environments. Using specialized agents collaborating in multi-agent deployments, either with or without human involvement, is an exciting area for future work. We anticipate this line of work will contribute valuable insights regarding defense strategies against malicious agents and hope to extend it further in the future.

## 12 Conclusions

While powerful commercial language models can handle many tasks, there are often reasons that prohibit their use. In the security domain, companies are hesitant to share the data of their internal networks, for example. A variety of smaller models have been released under permissive licenses. These models require much less hardware to run, and they can be used on-premises. However, they are not as capable as the much larger commercial models. In this work, we fine-tuned a 7 billion parameter model to function as an attacking agent within a network security environment. The resulting model, Hackphyr, had a performance comparable to that of the most powerful commercial model (GPT-4) and outperformed other language models (GPT-3.5-turbo, Zephyr-7b-$beta$) and baselines (Q-learning agent) even in the most complex previously unseen scenario.

To fine-tune the base model, we generated a new dataset that aimed to address the weaknesses of the base model with regard to environment understanding and the generation of valid and useful actions at a given state. Finally, we performed an extensive analysis of each agent’s actions that gave us insight into their behavior and shortcomings. This type of analysis can be used as a first step to explain the LLM agents’ behavior and can be a useful tool in the future for all types of agents in similar environments.

## 13 Acknowledgements

The authors acknowledge support by the Strategic Support for the Development of Security Research in the Czech Republic 2019–2025 (IMPAKT 1) program, by the Ministry of the Interior of the Czech Republic under No. VJ02010020 – AI-Dojo: Multi-agent testbed for the research and testing of AI-driven cyber security technologies.

## Appendix A Dataset examples

### A.1 Part I

Question:

 <svg class="ltx_picture" height="110.58" id="A1.SS1.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,110.58) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="83.02" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">```
This is the current status of a computer network during a
penetration testing procedure:
‘‘‘
Known networks are 1.1.1.0/24
Known hosts are 2.2.2.3
Controlled hosts are 2.2.2.2 and 1.1.1.2
Known data for source host 1.1.1.2: are (User1, WebData)
Known services for host 1.1.1.1 are openssh
‘‘‘
The actions you can take are ScanServices, FindData, ScanNetwork,
ExploitService, and ExfiltrateData.
The rules are:
{{rules}}

Can ScanNetwork action be used on a subnet that is not within
the known networks?

```</foreignobject></g></g></svg> 

Answer:

 <svg class="ltx_picture" height="38.63" id="A1.SS1.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,38.63) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="11.07" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">```
No, according to the rules, you can only scan networks that are
in your list of known networks.

```</foreignobject></g></g></svg> 

### A.2 Part II

Question:

 <svg class="ltx_picture" height="169.04" id="A1.SS2.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,169.04) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="141.48" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">```
You are a competent pentester and you are trying to teach others.
This is the current status of a computer network during a
penetration testing procedure:
‘‘‘
Known networks are 1.1.1.0/24 and 2.2.2.0/24
Known hosts are 2.2.2.3 and 2.2.2.4 and 1.1.1.1
Controlled hosts are 2.2.2.2 and 1.1.1.2
Known data for source host 1.1.1.2: are (User1, WebData)
Known services for host 1.1.1.1 are openssh
Know services for host 1.1.1.2 are openssh and http
Know services for host 2.2.2.3 are http and ldap
Know services for host 2.2.2.4 are openssh and http and ldap
‘‘‘
The actions you can take are ScanServices, FindData, ScanNetwork,
ExploitService, and ExfiltrateData.
The rules are:
{{rules}}

Here is an example of each action in JSON format:
...

Provide all the ExploitService actions that can be taken in
the current status in the correct format.

```</foreignobject></g></g></svg> 

Answer:

 <svg class="ltx_picture" height="106.43" id="A1.SS2.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,106.43) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="78.87" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">```
Based on the current status and the rules provided, we
can perform ScanNetwork actions on the known networks.
Here are the ScanNetwork actions in the correct JSON format for
each of the known networks:

‘‘‘json
{"action":"ScanNetwork", "parameters":
    {"target_network": "10.10.0.0/24"}}
{"action":"ScanNetwork", "parameters":
    {"target_network": "10.10.1.0/24"}}
{"action":"ScanNetwork", "parameters":
    {"target_network": "10.10.2.0/24"}}
‘‘‘

```</foreignobject></g></g></svg> 

### A.3 Part III

Question:

 <svg class="ltx_picture" height="235.46" id="A1.SS3.p2.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,235.46) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="207.9" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">```
You are a pentester and you want to exfiltrate data
to target host 42.86.112.78.
The actions you can take are ScanServices, FindData, ScanNetwork,
ExploitService, and ExfiltrateData.
The rules are:
{{rules}}

Current status:
Controlled hosts are 10.0.1.76 and 42.86.112.78
Known networks are 10.0.1.0/24 and 10.0.2.0/24 and 10.0.0.0/24
Known hosts are
Known services are none
Known data are none

Here are some examples of actions:
{{examples}}

Based on the current status provided, here’s a breakdown of
the objects and the actions that can be applied to them:

### Controlled Hosts:
- **10.0.1.76** and **42.86.112.78**
  - Actions:
    - **FindData**: You can find data in hosts you control.

### Known Networks:
- **10.0.1.0/24**, **10.0.2.0/24**, and **10.0.0.0/24**
  - Actions:
    - **ScanNetwork**: You can scan networks in your list of
    known networks to discover hosts.

### Known Hosts:
- None listed, but you can discover hosts by scanning the
known networks.

### Known Services:
- None listed, but you can discover services by scanning
the hosts you know.

### Known Data:
- None listed, but you can find data in the controlled hosts.

Provide the best next action in the correct JSON format. Action:

```</foreignobject></g></g></svg> 

Answer:

 <svg class="ltx_picture" height="40.01" id="A1.SS3.p4.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,40.01) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject class="ltx_minipage" color="#000000" height="12.45" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="402.3pt">```
{"action": "ScanNetwork","parameters":
    {"target_network": "10.0.1.0/24"}}

```</foreignobject></g></g></svg> 

## References

*   [1] A. Kott, Autonomous Intelligent Cyber-defense Agent: Introduction and Overview, in: Autonomous Intelligent Cyber Defense Agent (AICA), Vol. 87, Springer International Publishing, Cham, 2023, pp. 1–15. [doi:10.1007/978-3-031-29269-9_1](https://doi.org/10.1007/978-3-031-29269-9_1).
*   [2] V. Mnih, K. Kavukcuoglu, D. Silver, A. A. Rusu, J. Veness, M. G. Bellemare, A. Graves, M. Riedmiller, A. K. Fidjeland, G. Ostrovski, et al., Human-level control through deep reinforcement learning, Nature 518 (7540) (2015) 529–533. [doi:10.1038/nature14236](https://doi.org/10.1038/nature14236).
*   [3] M. C. Ghanem, T. M. Chen, E. G. Nepomuceno, Hierarchical reinforcement learning for efficient and effective automated penetration testing of large networks, Journal of Intelligent Information Systems 60 (2) (2023) 281–303. [doi:10.1007/s10844-022-00738-0](https://doi.org/10.1007/s10844-022-00738-0).
*   [4] J. S. Park, J. O’Brien, C. J. Cai, M. R. Morris, P. Liang, M. S. Bernstein, Generative Agents: Interactive Simulacra of Human Behavior, in: Proceedings of the 36th Annual ACM Symposium on User Interface Software and Technology, UIST ’23, Association for Computing Machinery, New York, NY, USA, 2023, pp. 1–22. [doi:10.1145/3586183.3606763](https://doi.org/10.1145/3586183.3606763).
*   [5] Z. Xi, W. Chen, X. Guo, W. He, Y. Ding, B. Hong, M. Zhang, J. Wang, S. Jin, E. Zhou, et al., The rise and potential of large language model based agents: A survey, arXiv preprint arXiv:2309.07864 (2023).
*   [6] Y. Du, O. Watkins, Z. Wang, C. Colas, T. Darrell, P. Abbeel, A. Gupta, J. Andreas, Guiding Pretraining in Reinforcement Learning with Large Language Models, in: Proceedings of the 40th International Conference on Machine Learning, Honolulu, USA, 2023.
*   [7] G. Wang, Y. Xie, Y. Jiang, A. Mandlekar, C. Xiao, Y. Zhu, L. Fan, A. Anandkumar, Voyager: An open-ended embodied agent with large language models, Transactions on Machine Learning Research (2024).
*   [8] L. Wang, C. Ma, X. Feng, Z. Zhang, H. Yang, J. Zhang, Z. Chen, J. Tang, X. Chen, Y. Lin, et al., A survey on large language model based autonomous agents, Frontiers of Computer Science 18 (6) (2024) 186345.
*   [9] L. C. Magister, J. Mallinson, J. Adamek, E. Malmi, A. Severyn, Teaching Small Language Models to Reason, arXiv:2212.08410 [cs] (2023).
*   [10] M. Rigaki., O. Lukáš., C. Catania., S. Garcia., Out of the cage: How stochastic parrots win in cyber security environments, in: Proceedings of the 16th International Conference on Agents and Artificial Intelligence - Volume 3: ICAART, INSTICC, SciTePress, Roma, Italy, 2024, pp. 774–781. [doi:10.5220/0012391800003636](https://doi.org/10.5220/0012391800003636).
*   [11] A. Vaswani, N. Shazeer, N. Parmar, J. Uszkoreit, L. Jones, A. N. Gomez, L. u. Kaiser, I. Polosukhin, Attention is all you need, in: Advances in Neural Information Processing Systems, Vol. 30, Curran Associates, Inc., 2017.
*   [12] H. W. Chung, L. Hou, S. Longpre, B. Zoph, Y. Tay, W. Fedus, Y. Li, X. Wang, M. Dehghani, S. Brahma, A. Webson, S. S. Gu, Z. Dai, M. Suzgun, X. Chen, A. Chowdhery, A. Castro-Ros, M. Pellat, K. Robinson, D. Valter, S. Narang, G. Mishra, A. Yu, V. Zhao, Y. Huang, A. Dai, H. Yu, S. Petrov, E. H. Chi, J. Dean, J. Devlin, A. Roberts, D. Zhou, Q. V. Le, J. Wei, Scaling instruction-finetuned language models, Journal of Machine Learning Research 25 (70) (2024) 1–53.
*   [13] J. Wei, M. Bosma, V. Zhao, K. Guu, A. W. Yu, B. Lester, N. Du, A. M. Dai, Q. V. Le, Finetuned language models are zero-shot learners, in: International Conference on Learning Representations, 2022.
*   [14] R. Taori, I. Gulrajani, T. Zhang, Y. Dubois, X. Li, C. Guestrin, P. Liang, T. B. Hashimoto, Alpaca: a strong, replicable instruction-following model; 2023, URL https://crfm. stanford. edu/2023/03/13/alpaca. html (2023).
*   [15] L. Tunstall, E. Beeching, N. Lambert, N. Rajani, K. Rasul, Y. Belkada, S. Huang, L. von Werra, C. Fourrier, N. Habib, N. Sarrazin, O. Sanseviero, A. M. Rush, T. Wolf, Zephyr: Direct Distillation of LM Alignment, arXiv:2310.16944 [cs] (Oct. 2023).
*   [16] L. Ouyang, J. Wu, X. Jiang, D. Almeida, C. Wainwright, P. Mishkin, C. Zhang, S. Agarwal, K. Slama, A. Ray, J. Schulman, J. Hilton, F. Kelton, L. Miller, M. Simens, A. Askell, P. Welinder, P. F. Christiano, J. Leike, R. Lowe, Training language models to follow instructions with human feedback, in: Advances in Neural Information Processing Systems, Vol. 35, Curran Associates, Inc., 2022, pp. 27730–27744.
*   [17] E. J. Hu, Y. Shen, P. Wallis, Z. Allen-Zhu, Y. Li, S. Wang, L. Wang, W. Chen, LoRA: Low-rank adaptation of large language models, in: International Conference on Learning Representations, 2022.
*   [18] T. Brown, B. Mann, N. Ryder, M. Subbiah, J. D. Kaplan, P. Dhariwal, A. Neelakantan, P. Shyam, G. Sastry, A. Askell, S. Agarwal, A. Herbert-Voss, G. Krueger, T. Henighan, R. Child, A. Ramesh, D. Ziegler, J. Wu, C. Winter, C. Hesse, M. Chen, E. Sigler, M. Litwin, S. Gray, B. Chess, J. Clark, C. Berner, S. McCandlish, A. Radford, I. Sutskever, D. Amodei, Language Models are Few-Shot Learners, in: Advances in Neural Information Processing Systems, Vol. 33, Curran Associates, Inc., 2020, pp. 1877–1901.
*   [19] J. Wei, X. Wang, D. Schuurmans, M. Bosma, B. Ichter, F. Xia, E. Chi, Q. V. Le, D. Zhou, Chain-of-Thought Prompting Elicits Reasoning in Large Language Models, Advances in Neural Information Processing Systems 35 (2022) 24824–24837.
*   [20] S. Yao, J. Zhao, D. Yu, N. Du, I. Shafran, K. R. Narasimhan, Y. Cao, React: Synergizing reasoning and acting in language models, in: The Eleventh International Conference on Learning Representations, 2023.
*   [21] N. Shinn, F. Cassano, A. Gopinath, K. Narasimhan, S. Yao, Reflexion: language agents with verbal reinforcement learning, in: Advances in Neural Information Processing Systems, Vol. 36, Curran Associates, Inc., 2023, pp. 8634–8652.
*   [22] Z. Wang, S. Cai, G. Chen, A. Liu, X. Ma, Y. Liang, T. CraftJarvis, Describe, explain, plan and select: interactive planning with large language models enables open-world multi-task agents, in: Proceedings of the 37th International Conference on Neural Information Processing Systems, NIPS ’23, Curran Associates Inc., Red Hook, NY, USA, 2024.
*   [23] S. Hao, Y. Gu, H. Ma, J. Hong, Z. Wang, D. Z. Wang, Z. Hu, Reasoning with language model is planning with world model, in: NeurIPS 2023 Workshop on Generalization in Planning, 2023.
*   [24] D. Hafner, Benchmarking the spectrum of agent capabilities, in: International Conference on Learning Representations, 2022.
*   [25] Y. Kant, A. Ramachandran, S. Yenamandra, I. Gilitschenski, D. Batra, A. Szot, H. Agrawal, Housekeep: Tidying Virtual Households Using Commonsense Reasoning, in: Computer Vision – ECCV 2022, Lecture Notes in Computer Science, Springer Nature Switzerland, Cham, 2022, pp. 355–373. [doi:10.1007/978-3-031-19842-7_21](https://doi.org/10.1007/978-3-031-19842-7_21).
*   [26] Y. Wu, S. Y. Min, S. Prabhumoye, Y. Bisk, R. R. Salakhutdinov, A. Azaria, T. M. Mitchell, Y. Li, Spring: Studying papers and reasoning to play games, in: Advances in Neural Information Processing Systems, Vol. 36, Curran Associates, Inc., 2023, pp. 22383–22687.
*   [27] K. Valmeekam, A. Olmo, S. Sreedharan, S. Kambhampati, Large language models still can’t plan (a benchmark for LLMs on planning and reasoning about change), in: NeurIPS 2022 Foundation Models for Decision Making Workshop, 2022.
*   [28] T. Silver, V. Hariprasad, R. S. Shuttleworth, N. Kumar, T. Lozano-Pérez, L. P. Kaelbling, PDDL planning with pretrained large language models, in: NeurIPS 2022 Foundation Models for Decision Making Workshop, 2022.
*   [29] T. Silver, S. Dan, K. Srinivas, J. B. Tenenbaum, L. Kaelbling, M. Katz, Generalized planning in pddl domains with pretrained large language models, Proceedings of the AAAI Conference on Artificial Intelligence 38 (18) (2024) 20256–20264. [doi:10.1609/aaai.v38i18.30006](https://doi.org/10.1609/aaai.v38i18.30006).
*   [30] A. Happe, J. Cito, Getting pwn’d by ai: Penetration testing with large language models, in: Proceedings of the 31st ACM Joint European Software Engineering Conference and Symposium on the Foundations of Software Engineering, ESEC/FSE 2023, Association for Computing Machinery, New York, NY, USA, 2023, p. 2082–2086. [doi:10.1145/3611643.3613083](https://doi.org/10.1145/3611643.3613083).
*   [31] S. Moskal, S. Laney, E. Hemberg, U.-M. O’Reilly, Llms killed the script kiddie: How agents supported by large language models change the landscape of network threat testing, arXiv preprint arXiv:2310.06936 (2023).
*   [32] G. Deng, Y. Liu, V. Mayoral-Vilches, P. Liu, Y. Li, Y. Xu, T. Zhang, Y. Liu, M. Pinzger, S. Rass, Pentestgpt: An llm-empowered automatic penetration testing tool, arXiv preprint arXiv:2308.06782 (2023).
*   [33] R. Raman, P. Calyam, K. Achuthan, Chatgpt or bard: Who is a better certified ethical hacker?, Computers & Security 140 (2024) 103804. [doi:https://doi.org/10.1016/j.cose.2024.103804](https://doi.org/https://doi.org/10.1016/j.cose.2024.103804).
*   [34] W. Tann, Y. Liu, J. H. Sim, C. M. Seah, E.-C. Chang, Using large language models for cybersecurity capture-the-flag challenges and certification questions, arXiv preprint arXiv:2308.10443 (2023).
*   [35] S. Garcia, O. Lukas, M. Rigaki, C. Catania, [NetSecGame, a RL env for training and evaluating AI agents in network security tasks.](https://github.com/stratosphereips/NetSecGame) (2023).
    URL [https://github.com/stratosphereips/NetSecGame](https://github.com/stratosphereips/NetSecGame)
*   [36] A. Q. Jiang, A. Sablayrolles, A. Mensch, C. Bamford, D. S. Chaplot, D. d. l. Casas, F. Bressand, G. Lengyel, G. Lample, L. Saulnier, et al., Mistral 7b, arXiv preprint arXiv:2310.06825 (2023).
*   [37] T. Dettmers, A. Pagnoni, A. Holtzman, L. Zettlemoyer, Qlora: Efficient finetuning of quantized llms, in: Advances in Neural Information Processing Systems, Vol. 36, Curran Associates, Inc., 2023, pp. 10088–10115.
*   [38] L. Tunstall, E. Beeching, N. Lambert, N. Rajani, S. Huang, K. Rasul, A. M. Rush, T. Wolf, The alignment handbook, [https://github.com/huggingface/alignment-handbook](https://github.com/huggingface/alignment-handbook) (2023).
*   [39] Stratosphere Research Laboratory, AIC, FEL, CTU, [Netsecdata (revision 913b966)](https://huggingface.co/datasets/stratosphere/NetSecData) (2024). [doi:10.57967/hf/3057](https://doi.org/10.57967/hf/3057).
    URL [https://huggingface.co/datasets/stratosphere/NetSecData](https://huggingface.co/datasets/stratosphere/NetSecData)
*   [40] C. J. C. H. Watkins, P. Dayan, Q-learning, Machine Learning 8 (3) (1992) 279–292. [doi:10.1007/BF00992698](https://doi.org/10.1007/BF00992698).
*   [41] O. Khattab, A. Singhvi, P. Maheshwari, Z. Zhang, K. Santhanam, S. Vardhamanan, S. Haq, A. Sharma, T. T. Joshi, H. Moazam, H. Miller, M. Zaharia, C. Potts, Dspy: Compiling declarative language model calls into self-improving pipelines, arXiv preprint arXiv:2310.03714 (2023).